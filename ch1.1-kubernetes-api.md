# kubernetes API


[toc]

## APIServer

在kubernetes架构概念层面上，Kubernetes由一些具有不同角色的服务节点组成。而master的控制平面由 `Apiserver`  `Controller-manager` 和 `Scheduler` 组成。

`Apiserver` 从概念上理解可以分为 `api` 和 `object` 的集合，`api` 可以理解为，处理读写请求来修改相应 `object` 的组件；而 `object` 可以表示为 kubernetes 对象，如 `Pod`， `Deployment` 等 。

## 基于声明式的API

在命令式 API 中，会直接发送要执行的命令，例如：*运行*、*停止* 等命令。在声明式API 中，将声明希望系统执行的操作，系统将不断将自身状态朝希望状态改变。

### 为什么使用声明式

在分布式系统中，任何组件随时都可能发生故障，当组件故障恢复时，需要明白自己需要做什么。在使用命令式时，出现故障的组件可能在异常时错过调用，并且在恢复时需要其他外部组件进行干预。而声明式仅需要在恢复时确定当前状态以确定他需要做什么。

### External APIs

在kubernetes中，控制平面是透明的，及没有internal APIs。这就意味着Kubernetes组件间使用相同的API交互。这里通过一个例子来说明外部APIs与声明式的关系。

例如，创建一个Pod对象，`Scheduler` 会监听 API来完成创建，创建完成后，调度程序不会命令被分配节点启动Pod。而在kubelet端，发现pod具有与自己相同的一些信息时，会监听pod状态。如改变kubelet则修改状态，如果删除掉Pod（对象资源不存在与API中），那么kubelet则将终止他。

### 为什么不使用Internal API

使用External API可以使kubernetes组件都使用相同的API，使得kubernetes具有可扩展性和可组合性。对于kubernetes中任何默认组件，如不足满足需求时，都可以更换为使用相同API的组件。

另外，外部API还可轻松的使用公共API来扩展kubernetes的功能

## API资源

从广义上讲，kubernetes对象可以用任何数据结构来表示，如：资源实例、配置（审计策略）或持久化实体（Pod）；在使用中，常见到的就是对应YAML的资源清单。转换出来就是RESTful地址，那么应该怎么理解这个呢？即，对资源的动作（操作）如图所示。但如果需要了解Kubernetes API需要掌握一些概念才可继续。

![image-20220513221304830](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220513221304830.png)

### Group

出于对kubernetes扩展性的原因，将资源类型分为了API组进行独立管理，可以通过 `kubectl api-resources`查看。在代码部分为 `vendor/k8s.io/api`

![image-20220513215038910](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220513215038910.png)

也可以通过 `kubectl xxx -v 6` 来查看 `kubectl` 命令进行了那些API调用

```
$ kubectl get pods -v 6
I0513 21:54:33.250752   38661 round_trippers.go:444] GET http://localhost:8080/api?timeout=32s 200 OK in 1 milliseconds
I0513 21:54:33.293831   38661 round_trippers.go:444] GET http://localhost:8080/apis?timeout=32s 200 OK in 0 milliseconds
I0513 21:54:33.299741   38661 round_trippers.go:444] GET http://localhost:8080/apis/discovery.k8s.io/v1beta1?timeout=32s 200 OK in 3 milliseconds
I0513 21:54:33.301097   38661 round_trippers.go:444] GET http://localhost:8080/apis/autoscaling/v2beta1?timeout=32s 200 OK in 4 milliseconds
I0513 21:54:33.301128   38661 round_trippers.go:444] GET http://localhost:8080/apis/authorization.k8s.io/v1beta1?timeout=32s 200 OK in 3 milliseconds
I0513 21:54:33.301222   38661 round_trippers.go:444] GET http://localhost:8080/apis/rbac.authorization.k8s.io/v1beta1?timeout=32s 200 OK in 1 milliseconds
I0513 21:54:33.301238   38661 round_trippers.go:444] GET http://localhost:8080/apis/authentication.k8s.io/v1beta1?timeout=32s 200 OK in 4 milliseconds
I0513 21:54:33.301280   38661 round_trippers.go:444] GET http://localhost:8080/apis/certificates.k8s.io/v1beta1?timeout=32s 200 OK in 4 milliseconds
....
No resources found in default namespace.
```

### Kind

在`kubectl api-resources` 中可以看到，有Kind字段，大部分人通常会盲目的 `kubectl apply` ,这导致了，很多人以为 `kind` 实际上为资源名称，`Pod` ，`Deployment` 等。

根据 [api-conventions.md](https://github.com/kubernetes/community/blob/7f3f3205448a8acfdff4f1ddad81364709ae9b71/contributors/devel/sig-architecture/api-conventions.md#types-kinds) 的说明，**Kind** 是对象模式，包含三种类型：

- Object，代表系统中持久化数据资源，如，`Service`, `Namespace`, `Pod`等
- List，是一个或多个资源的集合，通常以List结尾，如 `DeploymentList`
- 对Object的操作和和非持久化实体，如，当发生错误时会返回“status”类型，并不会持久化该数据。

### Object

对象是Kubernetes中持久化的实体，也就是保存在etcd中的数据；如：`Replicaset` , `Configmap` 等。这个对象代表的了集群期望状态和实际状态。

&gt; 例如：创建了Pod，kubernetes集群会调整状态，直到相应的容器在运行

Kubernetes资源又代表了对象，对象必须定义一些[字段](https://github.com/kubernetes/community/blob/7f3f3205448a8acfdff4f1ddad81364709ae9b71/contributors/devel/sig-architecture/api-conventions.md#resources)：

- 所有对象必须具有以下字段：
    - Kind
    - apiVersion
- metadata
- spec：期望的状态
- status：实际的状态

### API Link

前面讲到的 `kubectl api-resources` 展示的列表不能完整称为API资源，而是已知类型的kubernetes对象，要对展示这个API对象，需要了解其完整的周期。以 `kubectl get --raw /` 可以递归查询每个路径。

```
kubectl get --raw /
{
  &#34;paths&#34;: [
    &#34;/api&#34;,
    &#34;/api/v1&#34;,
    &#34;/apis&#34;,
    &#34;/apis/&#34;,
    &#34;/apis/admissionregistration.k8s.io&#34;,
    &#34;/apis/admissionregistration.k8s.io/v1&#34;,
    &#34;/apis/admissionregistration.k8s.io/v1beta1&#34;,
    &#34;/apis/apiextensions.k8s.io&#34;,
    &#34;/apis/apiextensions.k8s.io/v1&#34;,
...
```

对于一个Pod来说，其查询路径就为 `/api/v1/namespaces/kube-system/pods/coredns-f9bbb4898-7zkbf`

```
kubectl get --raw /api/v1/namespaces/kube-system/pods/coredns-f9bbb4898-7zkbf|jq
	
kind: Pod
apiVersion: v1
metadata: {}
spec:{}
status: {}
```

但有一些资源对象也并非这种结构，如 `configMap` ，因其只是存储的数据，所以没有 `spec` 和 `status`

```
kubectl get --raw /api/v1/namespaces/kube-system/configmaps/coredns|jq

kind
apiVersion
metadata
data
```

### API组成

一个API的组成为 一个 API 组`Group` , 一个版本 `Version` , 和一个资源 `Resource` ; 简称为 `GVR` 

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/API-server-gvr.png)

转换为实际的http路径为：

-  `/api/{version}/namespaces/{namespace_name}/resourcesPlural/{actual_resources_name}`
- `/api/v1/namespaces/default/pods/pods123`

而GVR中的R代表的是RESTful中的资源，转换为Kubernetes中资源应为 `Kind`，简称为 `GVK`，K在URI中表示在：

-  `/apis/{GROUP}/{VERSION}/namespaces/{namespace}/{KIND}`

## 请求和处理

这里讨论API请求和处理，API的一些数据结构位于 `k8s.io/api` ，并处理集群内部与外部的请求，而Apiserver 位于 `k8s.io/apiserver/pkg/server` 提供了http服务。

那么，当 HTTP 请求到达 Kubernetes API 时，实际上会发生什么？

- 首先HTTP请求在 `DefaultBuildHandlerChain` （可以参考`k8s.io/apiserver/pkg/server/config.go`）中注册filter chain，过滤器允许并将相应的信息附加至 `ctx.RequestInfo`; 如身份验证的相应
- `k8s.io/apiserver/pkg/server/mux` 将其分配到对应的应用
- `k8s.io/apiserver/pkg/server/routes` 定义了REST与对应应用相关联
- `k8s.io/apiserver/pkg/endpoints/groupversion.go.InstallREST()` 接收上下文，从存储中传递请求的对象。

![image-20220516171908228](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220516171908228.png)

&gt; Reference
&gt;
&gt; [Kubernetes API 基础——资源、种类和对象](https://iximiuz.com/en/posts/kubernetes-api-structure-and-terminology/)
&gt;
&gt; [Kubernetes - 一个 API 统治](https://networktechstudy.com/home/kubernetes-the-one-api)
&gt;
&gt; [Design and implementation](https://flugel.it/infrastructure-as-code/building-kubernetes-operators-part-2/)
&gt;
&gt; [Extending-the-Kubernetes-API](https://www.codeproject.com/Articles/5252640/Extending-the-Kubernetes-API)
&gt;
&gt; [Kubernetes 深入探讨](https://cloud.redhat.com/blog/kubernetes-deep-dive-api-server-part-1)
&gt;
&gt; 

