# Ingress Controller Traefik


## Kubernetes Ingress

Kubernetes Ingress是路由规则的集合，这些规则控制外部用户如何访问Kubernetes集群中运行的服务。

在Kubernetes中，有三种方式可以使内部Pod公开访问。

- NodePort：使用Kubernetes Pod的`NodePort`，将Pod内应用程序公开到每个节点上的端口上。
- Service LoadBalancer：使用Kubernetes Service，改功能会创建一个外部负载均衡器，使流量转向集群中的Kubernetes Pod。
- Ingress Controller：

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/d4c3ff03d767b1495101096ae3cdd76cd6246b99-661x418.png)

Node Port是在Kubernetes集群中每个节点（Node）上开放端口，Kubernetes直接将流量转向集群中Pod。Kubernetes集群中使用NodePort，则需要编辑防火墙规则，但是NodePort是范围在Kubernetes集群中默认设置的范围为 30000–32767，最终导致流量端口暴露在非标准端口之上。

LoadBalancer一般应用于云厂商提供的Kubernetes服务，如果自行在机器上部署Kubernetes集群，则需要自行配置LoadBalancer的实现，

Kubernetes Ingress，为Kubernetes中的抽象概念，实现为第三方代理实现，这种三方实现集合统称为Ingress Controller。Ingress Controller负责引入外部流量并将流量处理并转向对应的服务。

## Kubernetes IngressController功能实现

上面只是说道，在Kubernetes集群中，如何将外部流量引入到Kubernetes集群服务中。

### 负载均衡

无论在Kubernetes集群中，无论采用什么方式进行流量引入，都需要在外部负载均衡完成，而后负载均衡将流量引入Kubernetes集群入口或内部中，

通常情况下，NodePort方式管理繁琐，一般不用于生产环境。

### 服务的Ingress选择

Kubernetes Ingress是选择正确的方法来管理引入外部流量到服务内部。一般选择也是具有多样性的。

- Nginx Ingress Controller，Kubernetes默认推荐的Ingress，弊端①最终配置加载依赖`config reload`，②定制化开发较难，配置基本来源于config file。
- Envoy & traefik api网关，支持tcp/udp/grpc/ws等多协议，支持流量控制，可观测性，多配置提供者。
- 云厂商提供的Ingress。AWS ALB，GCP GLBG/GCE，Azure AGIC

## Traefik介绍

traefik-现代反向代理，也可称为现代边缘路由；traefik原声兼容主流集群，Kubernetes，Docker，AWS等。官方的定位traefik是一个让开发人员将时间花费在系统研发与部署功能上，而非配置和维护。并且traefik官方也提供自己的服务网格解决方案

作为一个 modern edge router ，traefik拥有与envoy相似的特性

- 基于go语言研发，目的是为了简化开发人员的配置和维护
- tcp/udp支持
- http L7支持
- GRPC支持
- 服务发现和动态配置
- front/ edge prory支持
- 可观测性
- 流量管理
- ...

### traefik 术语

要了解trafik，首先需要先了解一下 有关trafik中的一些术语。

- EntryPoints 入口点，是可以被下游客户端连接的命名网络位置，类似于envoy 的listener和nginx的listen
- services 服务，负载均衡，上游主机接收来自traefik的连接和请求并返回响应。 类似于nginx upstream envoy的clusters
- Providers 提供者，提供配置文件的后端，如file，kubernetes，consul，redis，etcd等，可使traefik自动更新
- routers 路由器，承上启下，分析请求，将下游主机的请求处理转入到services
- middlewares: 中间件，在将下游主机的请求转入到services时进行的流量调整

## 在Kubernetes中使用traefik网关作为Ingress

Traefik于2019年9月发布2.0 GA版，增加了很多新特性，包括IngressRoute Kubernetes CRD，TCP，最新版增加UDP等。

### 安装traefik

Traefik 支持两种方式创建路由规则，一是Traefik 自定义 `Kubernetes CRD` ，还有一种是 `Kubernetes Ingress` 。

### 创建traefik使用的Kubernetes CRD 资源

traefik官网提供了创建时所需要的 [yaml](https://doc.traefik.io/traefik/reference/dynamic-configuration/kubernetes-crd/)文件，这里仅需要使用官网提供的Definitions与RBAC即可。

在官网提供的yaml文件缺少 ServiceAccount，需要自行创建。

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: default
  name: traefik-ingress-controller
```

### 创建控制器

官方文件中，暂未找到所需运行traefik的控制器，需要自己创建一个。

traefik配置一般分为静态和动态配置，此处的静态是指，大部分时间内，不改变的配置（如nginx.conf），动态配置指，经常情况下改变的配置（可以理解为 nginx中 virtual host的每个独立配置文件）。

traefik提供配置的提供者也有很多种，此处使用命令行方式设置不长改变静态配置，也可以使用配置文件方式进行配置提供。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik
spec:
  selector:
    app: traefik-ingress
  type: NodePort
  ports:
    - name: web
      port: 80
      targetPort: 80
    - name: ssl
      port: 443
      targetPort: 443
    - name: dashboard
      port: 8080
      targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: traefik-ingress-controller-deployment
  labels:
    app: traefik-ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik-ingress
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      name: traefik-ingress
      labels:
        app: traefik-ingress
    spec:
      serviceAccountName: traefik-ingress-controller
      volumes:
        - name: ssl
          hostPath:
            path: /etc/kubernetes/pki
            type: DirectoryOrCreate
      containers:
        - image: traefik:v2.3.3
          name: traefik-ingress-lb
          imagePullPolicy: IfNotPresent
          volumeMounts:
          - name: ssl
            mountPath: /usr/local/pki
          ports:
            - name: web
              containerPort: 80
              hostPort: 1880
            - name: ssl
              containerPort: 443
              hostPort: 18443
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
          args:
            - --entrypoints.web.address=:80
            - --entrypoints.ssl.address=:443
            - --providers.kubernetescrd
            - --providers.kubernetesingress
            - --api=true
            - --ping=true
            - --api.dashboard=true
            - --serverstransport.insecureskipverify=true
            - --serverstransport.rootcas=/usr/local/pki/ca.crt
            - --accesslog
          livenessProbe:
            httpGet:
              path: /ping
              port: 8080
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /ping
              port: 8080
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
```

配置文件方式，仅需参考官网，把--agrs中的参数转换为配置文件并引入到pod后。替换启动参数

```yaml
args:
- --configfile=/config/traefik.yaml
```

## 使用CRD配置Traefik的流量管理

官网提供的基于Kubernetes CRD方式配置不是很多，可以参考动态配置中Kubernetes CRD Resources小结。

### 基于traefik dashboard方式配置增加HTTP router

前面使用官方提供CRD文件注册了Kubernetes CRD资源，所以traefik 中的资源类型，可作为kubernetes中资源使用。如 `kubectl get Middleware `

dashboard是traefik自己提供的服务所需的services为自己，traefik的entryPoints，当开启`--api.dashboard=true` 会增加一个8080端口作为traefik dashboard使用。而  `api@internal` 是traefik自己提供的services资源。

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-dashboard-route
spec:
  entryPoints:
    - traefik
  routes:
    - match: Host(`10.0.0.5`)
      kind: Rule
      services:
        - name: api@internal
          port: 8080
          kind: TraefikService
```

### 为Kubernetes dashboard增加HTTP路由

基于kubernetes crd作为提供者运行的traefik中，后端service可以是traefik的services也可以是kubernetes资源中的service。

基于kubernetes sevices

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: kubernetes-dashboard-route
  namespace: kubernetes-dashboard
  annotations:
    traefik.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  entryPoints:
    - ssl
  tls:
    secretName: k8s-ca

  routes:
    - match: PathPrefix(`/ui`)
      kind: Rule
      middlewares:
        - name: strip-ui
      services:
        - name: kubernetes-dashboard # kubernetes中的service对应的名称
          kind: Service # kubernetes中的service
          port: 443
          namespace: kubernetes-dashboard # kubernetes中的service对应的名称空间。
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: strip-ui
  namespace: kubernetes-dashboard
spec:
  stripPrefix:
    prefixes:
      - "/ui"
      - "/ui/"
```

此处使用了TLS的，开启了TLS，默认traefik是进行双向认证的，而kubernetes的dashboard的证书的ca并不知道，在访问时会出现`Internal Server Error`，目前没有找到有效的双向认证方法，普遍使用的方法都是调过认证`--serverstransport.insecureskipverify=true`

**reference**

https://stackoverflow.com/questions/49412376/internal-server-error-with-traefik-https-backend-on-port-443

https://github.com/traefik/traefik/issues/6821

### 负载均衡

负载均衡使用到的是traefik的services部分。

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: TraefikService
metadata:
  name: LoadBalancer
spec:
  weighted:
    services:
    - name: prod-v1.2
      port: 443
      weight: 1
      kind: Service
    - name: prod-v1
      port: 443
      weight: 2
      kind: Service
---
```

### 流量镜像

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: TraefikService
metadata:
  name: mirror1
spec:
  mirroring:
    name: s1
    port: 80
    mirrors:
      - name: s3
        percent: 20
        port: 80
      - name: mirror2
        kind: TraefikService
        percent: 20
```
