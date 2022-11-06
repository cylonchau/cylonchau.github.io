# istio流量治理


[toc]

## sidecar 介绍

在istio的流量管理等功能，都需要通过下发的配置应用到应用运行环境执行后生效，负责执行配置规则的组件在service mesh中承载应用代理的实体被称为`side-car`

Istio对流量策略管理等功能无须对应用程序做变动，

这种对应用服务完全非侵入的方式完全依赖于Side-car，应用的流量有Sidecar拦截并进行认证、鉴权、策略执行等治理功能。在Kubernetes平台中，Side-car容器于应用容器在同一个Pod中并共享网络名词控件，因此Side-car容易可以通过iptables规则拦截进出流量进行管理。

## sidecar的注入

sidecar是service mesh无侵入式架构的应用模式，在使用sidecar部署服务网格时，无需再每个应用节点运行服务代理，但是需要在每个应用程序中部署一个sidecar容器，来接管所有进出流量。

Sidecar会将额外容器注入到 Pod 模板中。Istio中的数据平面组件所需的容器有：

- `istio-init` 用于设置容器的iptables规则，目的是为了接管进出流量。在应用容器前启动并运行完成其生命周期，多个init容器按顺序依次完成。
- `istio-proxy` 基于envoy的sidecar的代理。

### sidecar被注入的方式

- 手动注入，使用 [`istioctl`](https://istio.io/latest/zh/docs/reference/commands/istioctl) 修改容器模板并添加前面提到的两个容器的配置。不论是手动注入还是自动注入，Istio 都从 `istio-sidecar-injector` 和的 `istio` 两个 Configmap 对象中获取配置。 [refer-istio-sidecar-injector](https://istio.io/latest/zh/blog/2019/data-plane-setup/#manual-injection)

- 自动注入，在部署应用是，istio自动将sidecar注入到pod。这是istio推荐的方法，Istio在基于Kubernetes平台之上，需要在部署应用之前，对要标记部署应用程序的名称空间标记 `kubectl label namespace default istio-injection=enabled` **这个操作是对名称空间级别生效的**。而后所部署的Pod中会注入sidecar执行上面sidecar容器的操作。

## istio injector 注入原理

Sidecar  Injector是Istio中实现自动注入Sidecar的组件，它是以Kubernetes准入控制器  AdmissionController 的形式运行的。Admission Controller 的基本工作原理是拦截Kube-apiserver的请求，在对象持久化之前、认证鉴权之后进行拦截。

Admission Controller有两种：一种是内置的，另一种是用户自定义的。

Kubernetes允许用户以Webhook的方式自定义准入控制器，Sidecar  Injector就是这样一种特殊的MutatingAdmissionWebhook。

Sidecar  Injector只在创建Pod时进行Sidecar容器注入，在Pod的创建请求到达  Kube-apiserver  后，首先进行认证鉴权，然后在准入控制阶段，Kube-apiserver以REST的方式同步调用Sidecar Injector Webhook服务进行init与istio-proxy容器的注入，最后将Pod对象持久化存储到etcd中。

## sidecar容器

sidecar容器内部运行着 `pilot-agent` 与 `envoy`

Pilot-agent：基于kubernetesAPI资源对象为envoy初始化可用的bootstrap配置进行启动，在运行后管理envoy运行状态，如配置变更，出错重启等。

envoy：数据平面的执行层，由 `pilot-agent` 所启动的，从xDS API动态获取配置信息。Envoy并通过流量拦截机制处理入栈及出栈的流量。

## envoy的listener

在运行在Kubernetes平台之上的istio，Envoy是通过Pilot将Kubernetes CRD资源 `DestnationRule` `VirtualService` `Gateway`等资源提供的配置，生成对应的Envoy配置。

每个sidecar中的envoy都会生成众多的配置，这些配置在每一个网格中会对对应的流量进行拦截管理。

通过envoy admin interface查看对应生产的envoy资源配置信息

```bash
$ kubectl exec reviews-v1-6549ddccc5-k995p -c istio-proxy -- curl -s 127.0.0.1:15000/listeners
97591f7a-0cbf-469a-a5f2-c9a76a3d0ced::0.0.0.0:15090
e523c9a4-da71-439f-885f-020f70349f21::0.0.0.0:15021
10.96.56.243_10251::10.96.56.243:10251
10.0.0.6_10250::10.0.0.6:10250
10.244.0.51_8443::10.244.0.51:8443
10.0.0.5_10250::10.0.0.5:10250
10.97.190.96_10252::10.97.190.96:10252
10.96.0.1_443::10.96.0.1:443
10.96.0.10_53::10.96.0.10:53
10.105.226.65_80::10.105.226.65:80
10.105.226.65_82::10.105.226.65:82
0.0.0.0_80::0.0.0.0:80
10.0.0.6_4194::10.0.0.6:4194
10.105.226.65_443::10.105.226.65:443
10.105.226.65_81::10.105.226.65:81
10.0.0.5_4194::10.0.0.5:4194
10.102.134.81_14250::10.102.134.81:14250
10.107.39.113_9090::10.107.39.113:9090
0.0.0.0_10255::0.0.0.0:10255
0.0.0.0_20001::0.0.0.0:20001
0.0.0.0_9090::0.0.0.0:9090
10.96.0.10_9153::10.96.0.10:9153
10.102.134.81_14268::10.102.134.81:14268
0.0.0.0_9411::0.0.0.0:9411
10.101.93.145_8000::10.101.93.145:8000
10.99.79.173_443::10.99.79.173:443
10.105.226.65_8080::10.105.226.65:8080
0.0.0.0_9080::0.0.0.0:9080
10.108.161.174_3000::10.108.161.174:3000
virtualOutbound::0.0.0.0:15001
virtualInbound::0.0.0.0:15006
..
```

[istio使用的端口](https://istio.io/latest/docs/ops/deployment/requirements/)

| Port  | Protocol | Description                                 | Pod-internal only |
| ----- | -------- | ------------------------------------------- | ----------------- |
| 15000 | TCP      | envoy管理端口                               | Yes               |
| 15001 | TCP      | envoy出站端口                               | No                |
| 15006 | TCP      | envoy入站端口                               | No                |
| 15008 | TCP      | envoy隧道端口                               | No                |
| 15020 | HTTP     | 从istio-agent envoy 应用 合并prometheus遥测 | No                |
| 15021 | HTTP     | 健康检查端口                                | No                |
| 15090 | HTTP     | Envoy Prometheus telemetry                  | No                |

envoy的listener通过绑定与IP Socket或者Unix Domain Socket，接收转发来的数据。

`VirtualOutboundListener` | `VirtualIntboundListener `通过一个端口接收所有出/入站流量，并根据目标端口来区分使用哪个Listener进行处理。

iptables通过对应规则拦截发往所在Pod的流量并转发到对应的Envoy接收端口（如入站流量为15006），该Listener通过配置将请求转发到请求原目标地址关联的Listener之上。[envoy原始目标地址](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/original_dst)

若不存在对应的listener则根据全局配置选项进行处理

- ALLOW_ANY：允许外发至任何服务的请求，没有匹配到目标Listener的流量则由tcp_proxy过滤器指向的PassthroughCluster。
- REGISTRY_ONLY：仅允许外发请求至注册过的服务，没有匹配到目标Listener的流量，则由tcp_proxy通过BlackHoleCluster丢弃。

监听端口配置参数`bind_to_port`的值为false，意味着该Listener并未真正绑定套接字上，而是通过对应的入站/出站Linstener接收兵转发流量。

## sidecar 流量的拦截

sidecar通过注入到Pod中的init，执行在 pod 命名空间中设置 `iptable` 规则来完成流量捕获并管理。

通过查看`nsenter` 命令可以查看对应实现的内容。

iptables在NAT表中新建了4条链，ISTIO_INBOUND、ISTIO_IN_REDIRECT、ISTIO_OUTPUT
和 ISTIO_REDIRECT

- 当进入Pod的流量会被`PREROUTING` 拦截处理。根据规则将数据包转发到`ISOTIO_INBOUND`。`-A PREROUTING -p tcp -j ISTIO_INBOUND`
- `ISTIO_INBOUND`处理对应的规则，当遇到`15008` , `22`  `15090`  `15021`  `15020` 不做处理，return给上一个链。 [关于iptables return](https://bbs.csdn.net/topics/340237926)
- 剩余流量从`ISTIO_INBOUND` 转交给 `ISTIO_IN_REDIRECT` `-A ISTIO_INBOUND -p tcp -j ISTIO_IN_REDIRECT`
- `ISTIO_IN_REDIRECT` 将流量交给15006端口应用处理。
- Envoy根据数据包的目的地址查看  Inbound方向的监听器配置，根据监听器及路由、Cluster、
  Endpoint等配置，决定是否将数据包转发到应用。
- OUTPUT的流量由规则 `-A OUTPUT -p tcp -j ISTIO_OUTPUT` 由 `ISTIO_OUTPUT` ` 链处理。

```bash
$ nsenter -t `ps -ef|grep details|grep envoy|awk &#39;{print $2}&#39;` -n iptables -t nat -S 
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-N ISTIO_INBOUND
-N ISTIO_IN_REDIRECT
-N ISTIO_OUTPUT
-N ISTIO_REDIRECT
-A PREROUTING -p tcp -j ISTIO_INBOUND
-A OUTPUT -p tcp -j ISTIO_OUTPUT
-A ISTIO_INBOUND -p tcp -m tcp --dport 15008 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 22 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 15090 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 15021 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 15020 -j RETURN
-A ISTIO_INBOUND -p tcp -j ISTIO_IN_REDIRECT 
# 到此入栈流量处理完成
-A ISTIO_IN_REDIRECT -p tcp -j REDIRECT --to-ports 15006

# 这里开始执行出站流量
# 原地址为127.0.0.6/32 不做处理
-A ISTIO_OUTPUT -s 127.0.0.6/32 -o lo -j RETURN
# 默认目标127.0.0.1/32--uid-owner 1337的由ISTIO_IN_REDIRECT处理
-A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -m owner --uid-owner 1337 -j ISTIO_IN_REDIRECT
-A ISTIO_OUTPUT -o lo -m owner ! --uid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN
-A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -m owner --gid-owner 1337 -j ISTIO_IN_REDIRECT
# 这些流量都不做处理。继续默认操作
-A ISTIO_OUTPUT -o lo -m owner ! --gid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -m owner --gid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -d 127.0.0.1/32 -j RETURN
# 默认所有ISTIO_OUTPUT链的流量都由ISTIO_REDIRECT
-A ISTIO_OUTPUT -j ISTIO_REDIRECT
# ISTIO_REDIRECT 的流量 默认有envoy进行处理转发到对应的应用
-A ISTIO_REDIRECT -p tcp -j REDIRECT --to-ports 15001
```

Istio中目前有两种流量拦截模式：REDIRECT模式和TPROXY模式。

TPROXY transparent proxy 透明代理，操作的是mangle表，同时需要原始客户端socket设置
IP_TRANSPARENT选项，此模式同时保留源IP地址和目标IP地址和端口。

REDIRECT ：端口重定向，将流量NAT至envoy，此模式会丢失源IP。

## sidecar资源配置

`Sidecar`这个资源清单描述了Sidecar代理的配置，从而管理他所连接的workload实例的流量。

- `workloadSelector`：选择sidecar应用此配置的Pod，如省略则为该名称空间的所有workload
  - lables: 配置pod的标签
- [ingress](https://istio.io/latest/docs/reference/config/networking/sidecar/#IstioIngressListener)：用于指定连接workload的入站流量的sidecar配置，如省略，则从默认获取。
  - port：Listener关联的端口
  - bind：Listener绑定的IP，ingress不允许unix套接字
  - [captureMode](https://istio.io/latest/docs/reference/config/networking/sidecar/#CaptureMode)： 流量捕获模式，仅Listener绑定到IP时适用。
    - `DEFAULT` 环境定义默认配置
    - `IPTABLES` ：使用IPtables重定向流量
    - `NONE` 不捕获。
- `egress`:  用于指定workload的出站流量的sidecar配置，如省略，则继承默认值
  - [port](https://istio.io/latest/docs/reference/config/networking/gateway/#Port)：Listener关联的端口
  - bind：Listener绑定的IP或套接字
  - [captureMode](https://istio.io/latest/docs/reference/config/networking/sidecar/#CaptureMode)： 流量捕获模式，仅Listener绑定到IP时适用。
    - `DEFAULT` 环境定义默认配置
    - `IPTABLES` ：使用IPtables重定向流量
    - `NONE` 不捕获。
  - hosts：目标地址，`namespace/dnsName`格式显示的一台或多台服务主机
- [outboundTrafficPolicy](https://istio.io/latest/docs/reference/config/networking/sidecar/#OutboundTrafficPolicy)：出站流量策略的配置，不指定则继承默认值。
  - [mode](https://istio.io/latest/docs/reference/config/networking/sidecar/#OutboundTrafficPolicy-Mode)：
    - REGISTRY_ONLY：仅允许注册到pilot的服务通过。
    - ALLOW_ANY：没有配置的，允许流量的出站。

### 声明一个出站流量

下面的示例在`Sidecar`名为的根命名空间中声明了全局默认配置`istio-config`，该配置在所有命名空间中配置了sidecars，以仅允许将流量发送到同一命名空间中的其他`workload`以及`istio-system`命名空间中的服务 。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: default
  namespace: istio-config
spec:
  egress:
  - hosts:
    - &#34;./*&#34;
    - &#34;istio-system/*&#34;
```

以下示例配置一个应用的出站与入站规则；在名称空间`prod-us1`中声明所有`app: ratings` 属于该`prod-us1/ratings` 带有标签的Pod 。workload在端口9080上接受入站HTTP通信。然后，将通信转发到侦听Unix套接字的附加workload实例。在出口流量，除了允许发给`istio-system` 名称空间任何端口任何Pod的流量，如果是9080端口，允许发送给`prod-us1`名称空间下的所有的服务。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: ratings
  namespace: prod-us1
spec:
  workloadSelector:
    labels:
      app: ratings
  ingress:
  - port:
      number: 9080
      protocol: HTTP
      name: somename
    defaultEndpoint: unix:///var/run/someuds.sock
  egress:
  - port:
      number: 9080
      protocol: HTTP
      name: egresshttp
    hosts:
    - &#34;prod-us1/*&#34;
  - hosts:
    - &#34;istio-system/*&#34;
```


