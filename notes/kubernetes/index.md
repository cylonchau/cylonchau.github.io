# kubernetes面试题

## Kubernetes概念

### Pod 的生命周期

Pod 状态始终处于一下几个状态之一:

- Pending: 部署 Pod 事务已被集群受理，但当前容器镜像还未下载完或现有资源无法满足 Pod 的资源需求
- Running: 所有容器已被创建，并被部署到节点上
- Successed: Pod 成功退出，并不会被重启
- Failed: Pod 中有容器被终止
- Unknown: 未知原因，如 kube-apiserver 无法与 Pod 进行通讯

### Kubernetes有哪些不同类型的服务？

- cluster ip
- Node Port
- Load Balancer
- Extrenal Name

### 什么是ETCD？

Etcd是用Go编程语言编写的，是一个分布式键值存储，用于协调分布式工作。因此，Etcd存储Kubernetes集群的配置数据，表示在任何给定时间点的集群状态。



### 什么是Ingress网络，它是如何工作的？

[Ingress网络](https://link.zhihu.com/?target=https%3A//www.edureka.co/blog/kubernetes-networking%3Futm_source%3Dmedium%26utm_medium%3Dcontent-link%26utm_campaign%3Dkubernetes-interview-questions%26source%3Dpost_page---------------------------)是一组规则，充当Kubernetes集群的入口点。这允许入站连接，可以将其配置为通过可访问的URL，负载平衡流量或通过提供基于名称的虚拟主机从外部提供服务。因此，Ingress是一个API对象，通常通过HTTP管理集群中服务的外部访问，是暴露服务的最有效方式。

### 什么是Headless Service？

Headless Service类似于“普通”服务，但没有群集IP。此服务使您可以直接访问pod，而无需通过代理访问它。



### 什么是集群联邦？

在联邦集群的帮助下，可以将多个Kubernetes集群作为单个集群进行管理。因此，您可以在数据中心/云中创建多个Kubernetes集群，并使用联邦来在一个位置控制/管理它们。

联合集群可以通过执行以下两项操作来实现此目的。请参考下图。

### kube-proxy的作用

kube-proxy运行在所有节点上，它监听apiserver中service和endpoint的变化情况，创建路由规则以提供服务IP和负载均衡功能。简单理解此进程是Service的透明代理兼负载均衡器，其核心功能是将到某个Service的访问请求转发到后端的多个Pod实例上。

### kube-proxy iptables的原理

Kubernetes从1.2版本开始，将iptables作为kube-proxy的默认模式。iptables模式下的kube-proxy不再起到Proxy的作用，其核心功能：通过API  Server的Watch接口实时跟踪Service与Endpoint的变更信息，并更新对应的iptables规则，Client的请求流量则通过iptables的NAT机制“直接路由”到目标Pod。



### kube-proxy ipvs的原理

IPVS在Kubernetes1.11中升级为GA稳定版。IPVS则专门用于高性能负载均衡，并使用更高效的数据结构（Hash表），允许几乎无限的规模扩张，因此被kube-proxy采纳为最新模式。

在IPVS模式下，使用iptables的扩展ipset，而不是直接调用iptables来生成规则链。iptables规则链是一个线性的数据结构，ipset则引入了带索引的数据结构，因此当规则很多时，也可以很高效地查找和匹配。

可以将ipset简单理解为一个IP（段）的集合，这个集合的内容可以是IP地址、IP网段、端口等，iptables可以直接添加规则对这个“可变的集合”进行操作，这样做的好处在于可以大大减少iptables规则的数量，从而减少性能损耗。



### kube-proxy ipvs和iptables的异同

iptables与IPVS都是基于Netfilter实现的，但因为定位不同，二者有着本质的差别：iptables是为防火墙而设计的；IPVS则专门用于高性能负载均衡，并使用更高效的数据结构（Hash表），允许几乎无限的规模扩张。

与iptables相比，IPVS拥有以下明显优势：

- 为大型集群提供了更好的可扩展性和性能；
- 支持比iptables更复杂的复制均衡算法（最小负载、最少连接、加权等）；
- 支持服务器健康检查和连接重试等功能；
- 可以动态修改ipset的集合，即使iptables的规则正在使用这个集合。

### Kubernetes镜像的下载策略

Kubernetes的镜像下载策略有三种：Always、Never、IFNotPresent。

- Always：镜像标签为latest时，总是从指定的仓库中获取镜像。
- Never：禁止从仓库中下载镜像，也就是说只能使用本地镜像。
- IfNotPresent：仅当本地没有对应镜像时，才从目标仓库中下载。默认的镜像下载策略是：当镜像标签是latest时，默认策略是Always；当镜像标签是自定义时（也就是标签不是latest），那么默认策略是IfNotPresent。

### 简述Kubernetes Scheduler使用哪两种算法将Pod绑定到worker节点

Kubernetes Scheduler根据如下两种调度算法将 Pod 绑定到最合适的工作节点：

- 预选（Predicates）：输入是所有节点，输出是满足预选条件的节点。kube-scheduler根据预选策略过滤掉不满足策略的Nodes。如果某节点的资源不足或者不满足预选策略的条件则无法通过预选。如“Node的label必须与Pod的Selector一致”。
- 优选（Priorities）：输入是预选阶段筛选出的节点，优选会根据优先策略为通过预选的Nodes进行打分排名，选择得分最高的Node。例如，资源越富裕、负载越小的Node可能具有越高的排名。

## Kubernetes 开发



### 资源和类型

**Kind**：实体的类型

**resources**：resources**是**，restful中的资源，标识一组HTTP端点（paths），可以理解为kind的实例化。



例如：Pod是etcd中的数据，而resources对应的 path上的resources

#### Resources和kinds区别

![img](../../images/kubernetes/d14b874da710)

- Resources与HTTP paths关联，
- Resources始终是API Group和Version的一部分。
- kind是这些endpoint返回并接收的objects的类型，并持久存在于etcd中。

| Kubernetes | OOP    |
| :--------- | :----- |
| Kind       | Class  |
| Resource   | Object |

Kind 其实就是一个类，用于描述对象的；而 Resource 就是具体的 Kind，可以理解成类已经实例化成对象了。

#### GVR与GVK有什么区别？

- GVR = Group Version Resources
- GVK = Group Version Kind 

每个kind都属于一个Group和Version中，通过GVK标识，GVR是GVK对外提供服务的入口，GVK与GVR之间的映射过程交 REST mapping





### client-go 客户端类型有哪些？


-  RestClient：是最基础的客户端，其作用是将http client进行封装成rest api格式。位于rest目录
-  ClientSet：基于RestClient进行封装对 Resource 与 version 管理集合，
-  DiscoverySet：RestClient进行封装，可动态发现kube-apiserver所支持的GVR（Group Version Resource）。
-  DynamicClient：基于RestClient，包含动态的客户端，可以对Kubernetes所支持的 API对象进行操作，包括CRD。



---

> Author: cylon  
> URL: http://example.org/notes/kubernetes/  

