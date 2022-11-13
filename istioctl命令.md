# istio命令



[toc]

显示配置文件中的差异 `istioctl profile diff default demo`

显示对应配置的profile `istioctl profile dump demo`

显示可用的配置 `istioctl profile list`

安装指定配置的istio `istioctl install --set profile=demo`

生成配置清单 `istioctl manifest generate`

istioctl验证安装: `istioctl manifest generate --set profile=demo |istioctl verify-install -`



istio的非侵入式流量治理



流量治理是一个非常宽泛的话题，例如：

◎ 动态修改服务间访问的负载均衡策略，比如根据某个请求特征做会话保持；

◎ 同一个服务有两个版本在线，将一部分流量切到某个版本上；

◎ 对服务进行保护，例如限制并发连接数、限制请求数、隔离故障服务实例等；

◎ 动态修改服务中的内容，或者模拟一个服务运行故障等。

在Istio中实现这些服务治理功能时无须修改任何应用的代码。较之微服务的SDK方式，Istio以一种更轻便、透明的方式向用户提供了这些功能。用户可以用自己喜欢的任意语言和框架进行开发，专注于自己的业务，完全不用嵌入任何治理逻辑。只要应用运行在Istio的基础设施上，就可以使用这些治理能力。
一句话总结 Istio 流量治理的目标：以基础设施的方式提供给用户非侵入的流量治理能力，用户只需
关注自己的业务逻辑开发，无须关注服务访问管理。



## istio服务架构

在istio1.8中，istio的分为 `envoy` （数据平面） 、`istiod` （控制平面） 、`addons`（管理插件） 及 `istioctl` （命令行工具，用于安装、配置、诊断分析等操作）组成。

### Pilot

Pilot是Istio控制平面流量管理的核心组件，管理和配置部署在Istio服务网格中的所有Envoy代理实例。

pilot-discovery为envoy sidecar提供服务发现，用于路由及流量的管理。通过kubernetes CRD资源获取网格的配置信息将其转换为xDS接口的标准数据格式后，通过gRPC分发至相关的envoy sidecar

Pilot组件包含工作在控制平面中的 `pilot-discovery` 和工作与数据平面的`pilot-agent ` 与Envoy(istio-proxy)

pilot-discovery主要完成如下功能：

- 从service registry中获取服务信息
- 从apiserver中获取配置信息。
- 将服务信息与配置信息适配为xDS接口的标准数据格式，通过xDS api完成配置分发。

pilot-agent 主要完成如下功能

- 基于kubernetes apiserver为envoy初始化可用的boostrap配置文件并启动envoy。

- 管理监控envoy的云兄状态及配置重载。

envoy

- 每个sidecar中的envoy是由pilot-agent基于生产的bootstrap配置进行启动，并根据指定的pilot地址，通过xDS api动态获取配置。
- sidecar形式的envoy通过流量拦截机制为应用程序实现入站和出站的代理功能。

在istio中的管理策略都是基于Kubernetes CRD的实现，其中有关于流量管理的CRD资源包括 `VirtualService` `EnvoyFilter`  `Gateway` `ServiceEntry`  `Sidecar` `DestinationRule` `WorkloadEntry` `WorkloadGroup`。[reference istio-networking-crd-resouces](https://istio.io/latest/docs/reference/config/networking/)

- VirtualServices：用于定义路由，可以理解为envoy的 `listener` =&gt; `filter` =&gt; `route_config`

- DestinationRule：用于定义集群，可以理解为envoy 的 cluster
- Gateway：用于定义作用于`istio-ingress-gateway`
- ServiceEntry：用于定义出站的路由，作用于`istio-egress-gateway`
- EnvoyFilter：为envoy添加过滤器或过滤器链。
- Sidecar：用于定义运行在sidecar之上的envoy配置。



Virtual Services和 Destination Rules是Istio流量路由功能的核心组件



Istio基于ServiceEntry资源对象将外部服务注册到网格内，从而像将外部服务以类
同内部服务一样的方式进行访问治理；
 对于外部服务，网格内Sidecar方式运行的Envoy即能执行治理；
 若需要将外出流量收束于特定几个节点时则需要使用专用的Egress Gateway完成，并基
于此Egress Gateway执行相应的流量治理；

## 网格流量管理



### 配置istio Virtual Services

`VirtualServices`是istio用于在其运行平台Kubernetes定义的配置，用来影响流量的路由规则；其本质就是为集群中envoy提供路由配置的。

####  VirtualServices名词解释

VirtualServices中一些流量路由定义的关键术语。

- Services：服务的唯一应用名称的单位，在Kubernetes之上 Services通常为Kubernetes Services资源。

- Source：在上文中，下游发起请求的客户端服务。

- Host：客户端请求服务时使用的地址

- Service versions：service允许的不同版本的子集（通常为流量管理中的概念，如AB等）每个Service都有一个包含所有实例的默认版本。


#### VirtualServices资源说明

VirtualServices中主要有这些配置用于配置流量的路由定义。 [reference virtual services](https://istio.io/latest/docs/reference/config/networking/virtual-service/)

- hosts：string[] 目标主机，可以是带有统配符的DNS Name或IP
- gateways：string[]，这些资源生效的网关和sidecar的名称。默认为名称空间级别，跨名称空间使用 `&lt;gateway namespace&gt;/&lt;gateway name&gt;`

  - `mesh` 默认值，表示生效与网格内所有sidecar

  - 仅应用于Gateway，该字段设置为Gateway的名称。

  - 忽略此字段：将应用于网格内部所有的sidecar
- [http](https://istio.io/latest/docs/reference/config/networking/virtual-service/)： HTTP协议流量的路由规则表。
  - [match](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPMatchRequest)：[] 匹配的条件。一个列表内单项内容的条件具有AND，整个列表的条件为OR。
    - name：
    - uri：匹配值区分大小写 
      - `exact:` 精确匹配。
      - `prefix`：用于前缀匹配。
      - `regex`：基于正则表达式匹配。
    - method：HTTP方法，参数与uri相同。
    - ...
  - [route](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPRouteDestination)：[] 设置的http流量的转发规则
    - destination：请求转发到的唯一标识符。
      - host：允许平台及ServiceEntry的服务名称，Kubernetes中为短名称`reviews.default.svc.cluster.local`
      - subset，在DestinationRule中定义的子集
      - port：可选，公开服务的端口
    - weight：转发流量的比例0-100 ，各目标的和应为100。
    - headers：操作头规则。
  - [redirect](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPRedirect)：重定向规则
  - delegate：只能在`Route`和`Redirect`为空时设置，委托的`VirtualServices` 名称
  - rewrite：重写HTTP URI。
  - timeout：HTTP请求超时，默认禁用。
  - retries：HTTP请求重试策略。
  - fault：故障注入
  - mirror：流量镜像
  - mirrorPercentage：对应mirror的比例
  - headers：操作http头的规则
  - ...
- tcp：TCP流量的路由规则的有序列表
- exportTo：允许 `VirtualServices` 其他名称空间的sidecar与gateway使用。

### VirtualServices配置实例

基于HTTP header的请求，将请求为`/ratings/v2/` 路径，并且请求头包含 `end-user`  值为`jason` 。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings-route
spec:
  hosts:
  - ratings.prod.svc.cluster.local
  http:
  - match:
    - headers:
        end-user:
          exact: jason
      uri:
        prefix: &#34;/ratings/v2/&#34;
      ignoreUriCase: true # 是否区分大小写，仅exact和prefix生效。
    route:
    - destination:
        host: ratings.prod.svc.cluster.local
```

委托其他virtualServices处理

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - &#34;bookinfo.com&#34;
  gateways:
  - mygateway
  http:
  - match:
    - uri:
        prefix: &#34;/productpage&#34;
    delegate:
       name: productpage
       namespace: nsA
  - match:
    - uri:
        prefix: &#34;/reviews&#34;
    delegate:
        name: reviews
        namespace: nsB
```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
  namespace: nsA
spec:
  http:
  - match:
     - uri:
        prefix: &#34;/productpage/v1/&#34;
    route:
    - destination:
        host: productpage-v1.nsA.svc.cluster.local
  - route:
    - destination:
        host: productpage.nsA.svc.cluster.local
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
  namespace: nsB
spec:
  http:
  - route:
    - destination:
        host: reviews.nsB.svc.cluster.local
```



### Istio目标规则配置：DestinationRule

[DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/)定义在完成路由配置后应用于服务流量的策略，即如何将流量调度至集群内，可以理解为DestinationRule定义的是envoy中的cluster。应用的内容也是envoy中cluster段的配置，如负载均衡配置，sidecar连接值及离群检测。

#### DestinationRule字段说明

- host: 注册表中的服务名称，kubernetes平台中使用短名称
- [trafficPolicy](https://istio.io/latest/docs/reference/config/networking/destination-rule/#TrafficPolicy)：应用的流量策略。
  - loadBalancer：使用的负载均衡算法，
    - simple
      - ROUND_ROBIN
      - LEAST_CONN
      - RANDOM
      - PASSTHROUGH
  - connectionPool：一致性hash
  - [outlierDetection](https://istio.io/latest/docs/reference/config/networking/destination-rule/#OutlierDetection)：离群值检测
    - consecutiveGatewayErrors：满足502 503 504 错误数弹出。
    - consecutive5xxErrors： 满足5xx错误数弹出。
    - interval：探测时间间隔
    - baseEjectionTime：最小逐出时间。主机被驱逐的时间等于`baseEjectionTime` * 退出次数。
    - maxEjectionPercent：最大驱逐比例，默认10%。
    - minHealthPercent：最少健康比例，默认为0%
  - tls
  - portLevelSettings
- subsets：[] 服务各个版本命名集。
  - name：子集的名称
  - labels：标签过滤器
  - trafficPolicy：子集流量策略，继承DestinationRule级别流量策略。
- exportTo：跨名称空间使用。

### DestinationRule配置实例

基于服务子集的配置

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
  subsets:
  - name: testversionv3
    labels:
      version: v3
  - name: testversionv2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
```

配置离群值

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-cb-policy
spec:
  host: reviews.prod.svc.cluster.local
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http2MaxRequests: 1000
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutiveErrors: 7
      interval: 5m
      baseEjectionTime: 15m
```


### 使用istio gateway配置服务入口

Istio还提供了一种配置模型 [Istio Gateway](https://istio.io/latest/docs/reference/config/networking/gateway/)。`Gateway` 与` KubernetesIngress` 相比，Gateway有高度的定制化与灵活性，并且允许将Istio功能应用于集群流量入口。

Gateway中运行的程序为envoy，它从控制平面接收相应的配置，并完成相关流量的传输；Gateway资源只负责网络入口点的相关功能，具体的路由实现则由VirtualService完成。

### Gateway CRD资源说明

Gateway定义了一个集群入口的负载均衡器，该负载均衡为运行在网格的边缘代理，负责将外部流量引入集群的内部。

Gateway资源生效于`Ingress` | `Egress` Envoy Pod的标签选择器，使用selector定义：`selector: app=istio-ingressgateway`。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: study-gateway
  namespace: default
spec:
  selector: # 基于名称空间中匹配pod的标签从而生效的应用
    app: istio-ingressgateway # 标签可以是一个或多个
  servers: # 描述对应的envoy的lintener的配置。
  - port:  # 设置envoy lintener
      number: 90 #  端口号 (Required)
      targetPort: # 可选 (Optional)
      name: envoy_end # 分配给端口的标签。
      protocol: HTTP # 端口服务协议，HTTP|HTTPS|GRPC|HTTP2|MONGO|TCP|TLS
    hosts: [ &#34;*&#34; , &#34;text.studyenvoy.com&#34; ] # 设置dnsName 可选的名称空间，*|. 
    tls: # 与TLS相关的选项集 (Optional)
    name: # 服务器的可选名称，必须唯一 (Optional)
```

### Gateway配置实例

基于istio Bookinfo示例的[Gateway](https://istio.io/latest/docs/examples/bookinfo/)资源清单。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - &#34;*&#34;
```

这里可以看到istio-ingress-gateway的pod的标签 `app=istio-ingressgateway`

```
[root@master01 ~]# kubectl get pods -n istio-system --show-labels
NAME                                    READY   STATUS    RESTARTS   AGE   LABELS
istio-ingressgateway-78b47bc88b-xqqpn   1/1     Running   0          22d   app=istio-ingressgateway, 
chart=gateways, 
heritage=Tiller, install.operator.istio.io/owning-resource=unknown,
istio.io/rev=default,istio=ingressgateway,
operator.istio.io/component=IngressGateways,
pod-template-hash=78b47bc88b,
release=istio,service.istio.io/canonical-name=istio-ingressgateway,
service.istio.io/canonical-revision=latest
```

## Istio外部服务配置：ServiceEntry

在Istio中提供了[ServiceEntry](https://istio.io/latest/docs/reference/config/networking/service-entry/)，可将网格外的服务加入网格中，像网格内的服务一样进行管理。

在实现上就是把外部服务加入 Istio 的服务发现，这些外部服务因为各种原因不能被直接注册到网格中。

### ServiceEntry字段说明

- host：与ServiceEntry关联的主机
- addresses：与服务关联的虚拟IP地址。
- ports：
  - number：服务的端口。
  - protocol：服务公开的协议。HTTP|HTTPS|GRPC|HTTP2|MONGO|TCP| TLS之一。
  - targetPort：目标端口号。
- location：`MESH_EXTERNAL ` | `MESH_INTERNAL`，决定是网格内部还是外部。
- resolution：服务发现机制。
  - NONE：
  - STATIC：指定静态IP地址。
  - DNS：通过DNS发现。
- endpoints：服务关联的端点，`workloadSelector` 与 `endpoints` 二选一。
- exportTo：共享其他名称空间
- subjectAltNames：如指定，将验证服务器证书的使用者备用名称是否与指定值之一匹配。

## 使用istio ingress gateway 配置一个网格外部的应用

### 部署应用程序

准备一个后端的应用

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpend-deply
  namespace: kube-system
  labels:
    app: httpend-deply
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpend-deply
  template:
    metadata:
      namespace: kube-system
      name: httpend-deply
      labels:
        app: httpend-deply
    spec:
      containers:
        - name: envoy-end
          image: sealloong/envoy-end
          imagePullPolicy: IfNotPresent
          livenessProbe:
            initialDelaySeconds: 3 # 首次探测延迟时间
            periodSeconds: 2 # 定期重试
            failureThreshold: 1 # 失败重试次数
            httpGet:
              port: 90
              path: ping
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: envoy-end
  labels:
    app: envoy-end
  namespace: kube-system
spec:
  type: NodePort # nodeport是为了验证服务是否正常
  ports:
    - port: 90
      name: envoy-end
      targetPort: 90
      nodePort: 30102
  selector:
    app: httpend-deply
```

应用Gateway和VirtualServices

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: envoyend-gateway
  namespace: kube-system
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 90
        name: http
        protocol: HTTP
      hosts:
        - &#34;*&#34;
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: envoy-end
  namespace: kube-system
spec:
  hosts:
    - &#34;*&#34;
  gateways:
    - envoyend-gateway
  http:
    - match:
        - uri:
            exact: /
      route:
        - destination:
            host: envoy-end
            port:
              number: 90
```

应用DestinationRule

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: envoyend
  namespace: kube-system
spec:
  host: envoy-end
  subsets:
    - name: end
      labels:
        app: httpend-deply
```


