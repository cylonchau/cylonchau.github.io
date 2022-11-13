# prometheus operator


[toc]

## 什么是 Operator？

`Operator`是由`CoreOS`公司开发的，用来扩展`kubernetes APi`的特定的应用程序控制器，`Operator`基于Kubernetes的资源和控制器概念之上构建，但同时又包含了对相应应用程序特定的一些专业知识。创建`operator`的关键是 `CRD（CustomResourceDefinition）`的设计。

### Prometheus Operator

`Prometheus Operator` 是CoreOS公司提供的基于Prometheus及其相关监视组件对Kubernetes集群组件的管理，该Operator目的是简化和自动化针对Kubernetes集群的基于Prometheus的管理及配置。

### Prometheus Operator架构组件

- **Operator**：作为Prometheus Operator的核心组件，也即是自定义的控制器，用来监视和部署管理Prometheus Operator CRD资源对象，监控并维持CRD资源状态。
- **Prometheus Server**：Operator 根据自定义资源 Prometheus 类型中定义的内容而部署的Prometheus Cluster
- **Prometheus Operator CRD**：
  - `Prometheus`：以CRD资源提供给Operator的类似于Pod资源清单定位的资源。
  - `ServiceMonitor`：声明定义对Kubernetes Services资源进行监控，使用标签选择器来选择所需配置的监控，后端是Service的Endpoint，通过Service标签选择器获取EndPoint对象。
  - `PodMonitor`：使用标签选择器，选择对匹配Pod进行监控
  - `Alertmanager`：声明定义了Alertmanager在Kubernetes中运行所提供的配置。
  - `PrometheusRule`: 声明定义了Prometheus在Kubernetes中运行所需的Rule配置。

reference

[Prometheus-Operator-design](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/design.md)

## Prometheus Operator监控二进制kubernetes

查看[兼容性列表](https://github.com/prometheus-operator/kube-prometheus#kubernetes-compatibility-matrix)选择对应的版本来下载，此处kubernetes集群为1.8.10 。

对应地址为 `https://github.com/prometheus-operator/kube-prometheus.git` ，可以在域名后添加`.cnpmjs.org` 访问中国的github加速。

```bash
git clone https://github.com.cnpmjs.org/prometheus-operator/kube-prometheus.git
```

资源清单在项目目录 `manifests` CRD在 `manifests/setup` **需要先安装CRD 和 Operator 对象**

#### kube-controller-manager 和 kube-scheduler 无监控数据

二进制部署的Kubernetes集群中部署Prometheus Operator，会发现在`prometheus server`的页面上发现`kube-controller`和`kube-schedule`的target为0/0。匹配不到节点信息，这是因为`serviceMonitor`是根据`label`去选取`svc`的。此处svc并没有`kube-controller`和`kube-schedule` 需要手动创建。

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-controller-manager
  labels:
    k8s-app: kube-controller-manager
    component: kube-controller-manager
spec:
  selector:
    k8s-app: kube-controller-manager
    component: kube-controller-manager
  ports:
    - name: https-metrics
      port: 10252
      targetPort: 10252
---
apiVersion: v1
kind: Endpoints
metadata:
  namespace: kube-system
  name: kube-controller-manager
  labels:
    k8s-app: kube-controller-manager
    component: kube-controller-manager
subsets:
- addresses:
  - ip: &#34;10.0.0.5&#34;
    nodeName: &#34;master01&#34;
  ports:
  - port: 10252
    name: https-metrics
---
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-scheduler
  labels:
    k8s-app: kube-scheduler
    component: kube-scheduler
spec:
  selector:
    k8s-app: kube-scheduler
    component: kube-scheduler
  ports:
    - name: https-metrics
      port: 10251
      targetPort: 10251
---
apiVersion: v1
kind: Endpoints
metadata:
  namespace: kube-system
  name: kube-scheduler
  labels:
    k8s-app: kube-scheduler
    component: kube-scheduler
subsets:
  - addresses:
    - ip: &#34;10.0.0.5&#34;
      nodeName: &#34;master01&#34;
    ports:
    - port: 10251
      name: https-metrics
```

此处需要注意的是：需要修改对应的Prometheus Operator资源清单的值一直才能获取到目标

`Service.spec.ports.name`要和`ServiceMonitor.spec.endpoints.port`的名称对应。

`Service.metadata.namespace`要和`ServiceMonitor.namespaceSelector.matchNames`对应

`Service.metadata.labels`的key要和`ServiceMonitor.JobLabel`对应

`Service.metadata.labels` 要和 `ServiceMonitor.selector.matchLabels`对应



## 监控第三方的服务及自定义servicemonitor

### 一、查看 Etcd 信息

这里的etcd采用二进制方法安装，可以直接访问 `host:2379/metrics` 获得。

### 二、将证书存入 Kubernetes

创建secret

```
kubectl create secret generic hketcd \
--from-file=&#39;/etc/etcd/pki/client.crt&#39; \
--from-file=&#39;/etc/etcd/pki/client.key&#39; \
--from-file=&#39;/etc/etcd/pki/ca.crt&#39; \
-n monitoring
```

### 三、将证书挂入 PrometheusServer

方法1： `kubectl edit prometheus k8s -n monitoring`

方法2：修改 `prometheus-prometheus.yaml` 文件

增加内容：

```yaml
  replicas: 2
  secrets:
    - hketcd
```

挂入后的证书保存在目录 `/etc/prometheus/secrets/{secret_name}/` 下。

### 四、创建 Etcd Service &amp; Endpoints

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: etcdmaster
  labels:
    k8s-app: etcd
    component: etcd
spec:
  selector:
    k8s-app: etcd
    component: etcd
  type: ClusterIP
  clusterIP: None # 设置为None，不分配Service IP
  ports:
    - name: https-metrics
      port: 2379
---
apiVersion: v1
kind: Endpoints
metadata:
  namespace: kube-system
  name: etcdmaster
  labels:
    k8s-app: etcd
    component: etcd
subsets:
  - addresses:
      - ip: &#34;10.0.0.5&#34;
        nodeName: &#34;master01&#34;
    ports:
      - port: 2379
        name: https-metrics
---
```

### 五、创建 ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: etcd
  namespace: monitoring
  labels:
    k8s-app: etcd
spec:
  jobLabel: k8s-app # 匹配工作的标签，这里是servicemonitor即 为service的标签
  endpoints: # 此ServiceMonitor的节点
    - port: https-metrics # k8s service endports的设置的端口的名称
      interval: 30s
      scheme: https
      tlsConfig:
        serverName: hketcd # 访问etcd的名称，因为证书原因需要认证访问的名称
        caFile: &#34;/etc/prometheus/secrets/hketcd/ca.crt&#34; # 这个是在prometheus容器内挂载的证书
        certFile: &#34;/etc/prometheus/secrets/hketcd/client.crt&#34;
        keyFile: &#34;/etc/prometheus/secrets/hketcd/client.key&#34;
        insecureSkipVerify: true
  selector: # 选择匹配到的endpoints
    matchLabels:
      k8s-app: etcd
      component: etcd
    # 与matchExpressions: #进行后端的匹配
  namespaceSelector: # 选择所在资源的名称控件
    matchNames:
      - kube-system
```


