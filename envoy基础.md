# envoy概念


服务网格的基本功能



- 控制服务间通信：熔断、重试、超时、故障注入、负载均衡和故障转移等。
- 服务发现：通过专用的服务总监发现服务端点。
- 可观测：指标数据采集、监控、分布式日志记录和分布式追踪。
- 安全性：TLS/SSL通信和秘钥管理。
- 身份认证和授权检查：身份认证，以及基于黑白名单或RBAC的访问控制功能。
- 部署：对容器技术的原生支持，例如Docker和Kubernetes等。
- 服务间的通信协议：HTTP1.1 HTTP2.0和gRPC等。
- 健康状态监测：监测上游服务的健康状态。
- ....





服务网格的部署模式有两种：主机共享代理和Sidecar容器

- 主机共享代理
  - 适用于同一主机存在许多容器的场景，并且还可以利用连接池来提高吞吐量。
  - 带一个代理进程故障将终止其所在主机上的整个容器队列，受影响的不仅仅是单个服务。
  - 实现方式中，常见的是允许为Kubernetes之上的 `DaemonSet`。
- Sidecar容器
  - 代理进程注入每个Pod定义以与住容器一同运行。
  - Sidecar进程应该尽可能轻量且功能完善。
  - 实现方案：Linkerd、Envoy和Conduit。



## What IS Enovy



Enovy是工作与OSI模型的7层代理

在实现上，数据平面的主流解决方案有Linkerd、Nginx、Envoy、HAProxy和Traefik等，而控制平面的实现主要有Istio、Nelson和SmartStack等几种口Linkerd
●由Buoyant公司于2016年率先创建的开源高性能网络代理程序（数据平面），是业界第一款Service Mesh产品，引领并促进了相关技术的快速发展
·Linkerd使用Namerd提供控制平面，实现中心化管理和存储路由规则、服务发现配置、支持运行时动态路由等功能

> **Envoy**

核心功能在于数据平面，于2016年由Lyft公司创建并开源，目标是成为通用的数据平面

云原生应用，既可用作前端代理，亦可实现Service Mesh中的服务间通信

Envoy常被用于实现APIGateway（如Ambassador）以及Kubernetes的Ingress Controller（例如gloo等），不过，基于Envoy实现的Service Mesh产品Istio有着更广泛的用户基础



Istio
·相比前两者来说，lstio发布时间稍晚，它于2017年5月方才面世，但却是目前最火热的Service Mesh解决方案，得到了Google、IBM、Lyt及Redhat等公司的大力推广及支持
·目前仅支持部署在Kubernetes之上，其数据平而由Envoy实现



envoy适用于现代大型面向服务架构的动态组织应用程序的7层代理应用程序，其典型特性：

- 运行在架构进程之外
- 支持3/4层过滤器
- 支持HTTP协议7层过滤器
- 支持HTTP协议7层高级路由功能





![1569226040915](../images/Untitled/1569226040915.png)

envoy在现代服务体系架构当中的适用位置既可以为一组服务提供代理，作为整个服务统一的api网关来进行接入，同时也可以对每一个微服务单独实现作为代理，此时需要以sidecar的形式运行在应用程序前端。进而与最前端的apigateway组织成所谓的服务网格（Sevice Mesh）。而在每一个Envoy实例内部都要接受请求。这个请求可能是来自互联网或服务网格之外的客户端，称之为南北流量；也可能是来自网格当中的其他服务的请求，称之为东西流量。

![image-20200827181417724](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200827181417724.png)



在Envoy当中类似于Nginx或者haproxy的功能术语：

- **`listeners`** ：面向客户端一侧提供监听并接受客户端请求的组件。类似于nginx的server或haproxy的frontend 。
- **`cluster`**：面向后端测，将多个被代理的实例分成组，统一进行负载均衡调度的组。
- **`cluster definltions`**：cluster的归类。
- `filter chains`：过滤器链，可以将多个链以流水线方式依次进行处理。 





面向客户端提供服务/监听的套接字，lintener内部包含一到多个filter组成`filter chains`，称之为过滤器链。过滤器是lintener内部的子组件。envoy支持4层过滤器和7层过滤器。



![image-20200827182244618](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200827182244618.png)



envoy 术语



**主机（Host）**：能够进行网络通信的实体（如手机，服务器等上的应用程序）。在这个文档中，主机是一个逻辑网络应用程序。一个物理硬件可能有多个主机上运行，只要他们可以独立寻址。

**下游（Downstream）**：下游主机连接到Envoy，发送请求并接收响应。

**上游（Upstream）**：上游主机接收来自Envoy的连接和请求并返回响应。

- **监听器（Listener）**：侦听器是可以被下游客户端连接的命名网络位置（例如，端口，unix域套接字等）。Envoy公开一个或多个下游主机连接的侦听器。
  - filters chains，过滤器链L4/L7
  - route：完成对客户请求进行分类

**群集（Cluster）**：群集是指Envoy连接到的一组逻辑上相似的上游主机。Envoy通过服务发现发现一个集群的成员。它可以通过主动健康检查来确定集群成员的健康度，从而Envoy通过负载均衡策略将请求路由到相应的集群成员。

**网格（Mesh）**：协调一致以提供一致的网络拓扑的一组主机。在本文档中，“Envoy mesh”是一组Envoy代理，它们构成了由多种不同服务和应用程序平台组成的分布式系统的消息传递基础。

**运行时配置（Runtime configuration）**：与Envoy一起部署的外置实时配置系统。可以更改配置设置，可以无需重启Envoy或更改主要配置。



## enovy的部署类型

![image-20200828113756518](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200828113756518.png)

仅用于service到service之间的通讯，对应的Enovy工作为

- egress listener
- ingress listener
- optional exteral service egress listeners

![image-20200828114252599](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200828114252599.png)



![image-20200828115046632](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200828115046632.png)



![image-20200828120238768](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200828120238768.png)
