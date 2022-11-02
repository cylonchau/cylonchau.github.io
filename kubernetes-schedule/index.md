# kubernetes调度

[toc]

## Overview

`kube-scheduler` 是kubernetes控制平面的核心组件，其默认行为是将 pod 分配给节点，同时平衡Pod与Node间的资源利用率。通俗来讲就是 `kube-scheduler` 在运行在控制平面，并将工作负载分配给 Kubernetes 集群。

本文将深入 Kubernetes 调度的使用，包含：”一般调度”，”亲和度“，“污点与容忍的调度驱逐”。最后会分析下 **Scheduler Performance Tuning**，即微调scheduler的参数来适应集群。

## 简单的调度

### NodeName &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt; 

最简单的调度可以指定一个 **NodeName** 字段，使Pod可以运行在对应的节点上。如下列资源清单所示

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: netpod
spec:
  containers:
  - name: netbox
    image: cylonchau/netbox
  nodeName: node01  
```

通过上面的资源清单Pod最终会在 node01上运行。这种情况下也会存在很多的弊端，如资源节点不足，未知的nodename都会影响到Pod的正常工作，通常情况下，这种方式是不推荐的。

```bash
$ kubectl describe pods netpod 
Name:         netpod
Namespace:    default

	...

QoS Class:       BestEffort
Node-Selectors:  &lt;none&gt;
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  86s   kubelet  Pulling image &#34;cylonchau/netbox&#34;
  Normal  Pulled   17s   kubelet  Successfully pulled image &#34;cylonchau/netbox&#34;
  Normal  Created  17s   kubelet  Created container netbox
  Normal  Started  17s   kubelet  Started container netbox



$ kubectl get pods netpod  -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
netpod   1/1     Running   0          48m   192.168.0.3   node01   &lt;none&gt;           &lt;none&gt;
```

通过上面的输出可以确定，通过 `NodeName` 方式是不经过 *scheduler* 调度的

### nodeSelector  &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt; 

`label` 是 kubernetes中一个很重要的概念，通常情况下，每一个工作节点都被赋予多组 *label* ,可以通过命令查看对应的 *label* 。

```bash
$ kubectl get node node01 --show-labels
NAME     STATUS   ROLES    AGE   VERSION    LABELS
node01   Ready    &lt;none&gt;   15h   v1.18.20   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux
```

而 `nodeSelector` 就是根据这些 *label* ，来选择具有特定一个或多个标签的节点。例如，如果需要在一组特定的节点上运行pod，可以设置在 “*PodSpec*” 中定义`nodeSelector` 为一组键值对：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netpod-nodeselector
spec:
  selector:
    matchLabels:
      app: netpod
  replicas: 2 
  template:
    metadata:
      labels:
        app: netpod
    spec:
      containers:
      - name: netbox
        image: cylonchau/netbox
      nodeSelector: 
        beta.kubernetes.io/os: linux
```

对于上面的pod来讲，Kubernetes Scheduler 会找到带有 `beta.kubernetes.io/os: linux`标签的节点。对于更多kubernetes内置的标签，可以参考  &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt; 

对于标签选择器来说，最终会分布在具有标签的节点上

```bash
kubectl describe pod netpod-nodeselector-69fdb567d8-lcnv6 
Name:         netpod-nodeselector-69fdb567d8-lcnv6
Namespace:    default

	...

QoS Class:       BestEffort
Node-Selectors:  beta.kubernetes.io/os=linux
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  8m18s  default-scheduler  Successfully assigned default/netpod-nodeselector-69fdb567d8-lcnv6 to node01
  Normal  Pulling    8m17s  kubelet            Pulling image &#34;cylonchau/netbox&#34;
  Normal  Pulled     7m25s  kubelet            Successfully pulled image &#34;cylonchau/netbox&#34;
  Normal  Created    7m25s  kubelet            Created container netbox
  Normal  Started    7m25s  kubelet            Started container netbox
```

## 节点亲和性 &lt;sup&gt;&lt;a href=&#34;#4&#34;&gt;[4]&lt;/a&gt;&lt;/sup&gt; 

对于使用了调度功能的系统来说，亲和度 （`Affinity`）是个很常见的概念，通常亲和度发生在并行（`parallel `）环境中；在这种环境下，亲和度提供了在一个节点上运行pod可能比在其他节点上运行更有效，而计算亲和度通常由多种条件组成。一般情况下，亲和度分为“**软亲和**与**硬亲和**

- 软亲和，**Soft Affinity**，是调度器尽可能将任务保持在同一个节点上。这只是一种尝试；如果不可行，则将进程迁移到另一个节点
- 硬亲和，**Hard affinity**，硬亲和度是强行将任务绑定到指定的节点上

而在kubernetes中也支持亲和度的概念，而亲和度是与 *nodeSelector* 配合形成的一个算法。其中硬亲和被定义为`requiredDuringSchedulingIgnoredDuringExecution`；软亲和被定义为 `preferredDuringSchedulingIgnoredDuringExecution`

- 硬亲和性（`requiredDuringSchedulingIgnoredDuringExecution`）：必须满足条件，否则调度程序无法调度 Pod。
- 软亲和性 （`preferredDuringSchedulingIgnoredDuringExecution`）：*scheduler* 将查找符合条件的节点。如果没有满足要求的节点将忽略这条规则，*scheduler* 将仍会调度 Pod。

### Node Affinity

#### Node Affinity参数说明

调度程序会更倾向于将 pod 调度到满足该字段指定的亲和性表达式的节点，但它可能会选择违反一个或多个表达式的节点。最优选的节点是权重总和最大的节点，即对于满足所有调度要求（资源请求、requiredDuringScheduling 亲和表达式等）的每个节点，通过迭代该字段的元素来计算总和如果节点匹配相应的matchExpressions，则将“权重”添加到总和中；具有最高和的节点是最优选的。

如果在调度时不满足该字段指定的亲和性要求，则不会将 Pod 调度到该节点上。如果在 pod 执行期间的某个时间点不再满足此字段指定的亲和性要求（例如，由于更新），系统可能会或可能不会尝试最终将 pod 从其节点中逐出。

affinity 范围应用于 `Pod.Spec` 下，参数如下：

- **`nodeAffinity`**：node亲和度相关根配置
    - **`preferredDuringSchedulingIgnoredDuringExecution`**：软亲和
        - **`preference`** (*required*)：选择器
            - **`matchExpressions`**：匹配表达式，标签可以指定部分
                - **`key`** (*\&lt;string&gt; \-required\-*)：
                - **`operator`** (*\&lt;string&gt; -required-*)：# 与一组 key-values的运算方式。 
                    - In, NotIn, Exists, DoesNotExist, Gt, Lt。
                - **`values`** (*&lt;[]string&gt;*)：
            - **`matchFields`**：  匹配字段
                - **`key`** (*\&lt;string&gt; \-required\-*)：
                - **`operator`** (*\&lt;string&gt; -required-*)：# 与一组 key-values的运算方式。 
                    - In, NotIn, Exists, DoesNotExist, Gt, Lt。
                - **`values`** (*&lt;[]string&gt;*)：
        - **`weight`** (*required*)：范围为 1-100，具有最高和的节点是最优选的
    - **`requiredDuringSchedulingIgnoredDuringExecution`**：硬亲和
        - **`nodeSelectorTerms`**
            - **`matchExpressions`**：
                - **`key`** (*\&lt;string&gt; \-required\-*)：
                - **`operator`** (*\&lt;string&gt; -required-*)：# 与一组 key-values的运算方式。 
                    - In, NotIn, Exists, DoesNotExist, Gt, Lt。
                - **`values`** (*&lt;[]string&gt;*)：
            - **`matchFields`**：
                - **`key`** (*\&lt;string&gt; \-required\-*)：
                - **`operator`** (*\&lt;string&gt; -required-*)：一组 key-values 的运算方式。 
                    - In, NotIn, Exists, DoesNotExist, Gt, Lt。
                - **`values`** (*&lt;[]string&gt;*)：

&gt; Notes: matchFields使用的是资源清单的字段（kubectl get node -o yaml），而matchExpressions匹配的是标签

#### Node Affinity示例

上面的介绍了解到了Kubernetes中相对与 `nodeSelector`可以更好表达复杂的调度需求：**节点亲和性**，使用PodSpec中的字段 `.spec.affinity.nodeAffinity`  指定相关 *affinity* 配置。

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netpod-nodeselector
spec:
  selector:
    matchLabels:
      app: netpod
  replicas: 2 
  template:
    metadata:
      labels:
        app: netpod
    spec:
      containers:
      - name: netbox
        image: cylonchau/netbox
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: app
                operator: In
                values:
                - test
```

上面的清单表明，当节点存在 `app: test` 标签时，会调度到对应的Node上，如果没有节点匹配这些条件也不要紧，会根据普通匹配进行调度。

当硬策略和软策略同时存在时的情况，根据设置的不同，硬策略优先级会高于软策略，哪怕软策略权重设置为100

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netpod-nodeselector
spec:
  selector:
    matchLabels:
      app: netpod
  replicas: 2 
  template:
    metadata:
      labels:
        app: netpod
    spec:
      containers:
      - name: netbox
        image: cylonchau/netbox
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: app
                operator: In 
                values:
                - test
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                - antarctica-east1
                - antarctica-west1
```

下面是报错信息

```bash
Warning  FailedScheduling  4s (x3 over 24s)  default-scheduler  0/2 nodes are available: 2 node(s) didn&#39;t match node selector.
```

## Pod亲和性 &lt;sup&gt;&lt;a href=&#34;#4&#34;&gt;[4]&lt;/a&gt;&lt;/sup&gt; 

pod亲和性和反亲和性是指根据节点上已运行的Pod的标签而不是Node标签来限制Pod可以在哪些节点上调度。例如：*X* 满足一个或多个运行 *Y* 的条件，这个时候 Pod满足在X中运行。其中 *X* 为拓扑域，*Y* 则是规则。

&gt; Notes：官方文档中不推荐pod亲和度在超过百个节点的集群中使用该功能 &lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt; 

### Pod亲和性配置

Pod亲和性和反亲和性与Node亲和性类似，affinity 范围应用于 `Pod.Spec.podAffinity`  下，这里不做重复复述，可以参考Node亲和性参数说明部分。

topologyKey，==不允许是空值==，该值将影响Pod部署的位置，影响范围为，与亲和性条件匹配的对应的节点中的什么拓扑，topologyKey的拓扑域由label标签决定。

除了 `topologyKey` 之外，还有标签选择器 `labelSelector`  与 名称空间 `namespaces` 可以作为同级的替代选项

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netpod-podaffinity
spec:
  selector:
    matchLabels:
      app: podaffinity
  replicas: 1 
  template:
    metadata:
      labels:
        app: podaffinity
    spec:
      containers:
      - name: podaffinity
        image: cylonchau/netbox
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - netpod
            topologyKey: zone
```

如果没有Pod匹配到规则，则pending状态

```bash
Warning  FailedScheduling  59s (x2 over 59s)  default-scheduler  0/2 nodes are available: 2 node(s) didn&#39;t match pod affinity rules, 2 node(s) didn&#39;t match pod affinity/anti-affinity.
```

### Pod Anti-Affinity

在某些场景下，部分节点不应该有很多资源，即某些节点不想被调度。例如监控运行节点由于其性质，不希望该节点上有很多资源，或者因节点配置的不同，配置较低节点不希望调度很多资源；在这种情况下，如果将符合预期之外的Pod调度过来会降低其托管业务的性能。这种情况下就需要 ***反亲和度***（`Anti-Affinity`）来使Pod远离这组节点

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netpod-podaffinity
spec:
  selector:
    matchLabels:
      app: podaffinity
  replicas: 1 
  template:
    metadata:
      labels:
        app: podaffinity
    spec:
      containers:
      - name: podaffinity
        image: cylonchau/netbox
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - netpod
              topologyKey: zone
```

## Taints And Tolerations &lt;sup&gt;&lt;a href=&#34;#6&#34;&gt;[6]&lt;/a&gt;&lt;/sup&gt; 

### Taints 

亲和度和反亲和度虽然可以阻止Pod在特定节点上运行，但还存在一个问题，就是亲和度和反亲和度需要声明运行的节点或者是不想运行的节点，如果忘记声明，还是会被调度到对应的Node上。Kubernetes还提供了一种驱逐Pod的方法，就是污点（`Taints`）与容忍（`Tolerations`）。

创建一个污点

```bash
kubectl taint nodes node1 key1=value1:NoSchedule

$ kubectl taint nodes mon01 role=monitoring:NoSchedule
```

删除一个污点，

```bash
kubectl taint nodes node1 key1=value1:NoSchedule-
```

除了 `NoSchedule` ，还有 `PreferNoSchedule` 与 NoExecute

- `PreferNoSchedule` ：类似于软亲和性的属性，尽量去避免污点，但不是强制的。
- `NoExecute` 表示，当Pod还没在节点上运行时，并且存在至少一个污点时生效，此时Pod不会被调度到该节点；当Pod已经运行在节点上时，并且存在至少一个污点时生效，Pod将会从节点上被驱逐。

### Tolerations 

当Node有污点时，在调度时会自动被排除。当调度在受污染的节点上执行Predicate部分时将失败，而容忍度则是使 pod 具有对该节点上的污点进行容忍，即拥有容忍度的Pod可以调度到有污点的节点之上。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netpod-podaffinity
spec:
  selector:
    matchLabels:
      app: podaffinity
  replicas: 1 
  template:
    metadata:
      labels:
        app: podaffinity
    spec:
      containers:
      - name: podaffinity
        image: cylonchau/netbox
      tolerations:
      - key: &#34;role&#34;
        operator: &#34;Equal&#34;
        value: &#34;monitoring&#34;
        effect: &#34;NoSchedule&#34;
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - netpod
              topologyKey: zone
```

容忍度中存在一个特殊字段 `TolerationSeconds` ，表示容忍的时间，默认不会设置，即永远容忍污点。设置成0或者负数表示理解驱逐。==仅在污点为 `NoExecute` 时生效==

`operator` 属性有两个值 `Exists` 和 `Equal`

- 如果 operator 为 `Exists`，则无需 value 属性，因为判断的是有污点的情况下。
- 如果 operator 为 `Equal`，则表示 key 与 value 之间的关系是 $key=value$ 

- 空 key，并且operator为 `Exists`，将匹配到所有，即容忍所有污点
- 空 `effect` 匹配所有  `effect` ，即匹配所有污点；这种情况下加上条件的话，可以容忍所有类型的污点

## 驱逐 &lt;sup&gt;&lt;a href=&#34;#7&#34;&gt;[7]&lt;/a&gt;&lt;/sup&gt; 

当污点设置为 `NoExecute`这种情况下会驱逐Pod，驱逐条件又如下几个：

- 不容忍污点的 pod 会立即被驱逐
- 容忍污点但未配置 `tolerationSeconds` 属性的会保持不变，即该节点与Pod保持绑定
- 容忍指定污点的 pod 并且配置了`tolerationSeconds` 属性，节点与Pod绑定状态仅在配置的时间内。

Kubernetes内置了一些污点，此时 *Controller* 会自动污染节点：

- `node.kubernetes.io/not-ready`: Node故障。对应 NodeCondition 的`Ready` =  `False`。

- `node.kubernetes.io/unreachable`：Node控制器无法访问节点。对应 NodeCondition `Ready`= `Unknown`。
- `node.kubernetes.io/memory-pressure`：Node内存压力。
- `node.kubernetes.io/disk-pressure`：Node磁盘压力。
- `node.kubernetes.io/pid-pressure`：Node有PID压力。
- `node.kubernetes.io/network-unavailable`：Node网络不可用。
- `node.kubernetes.io/unschedulable`：Node不可调度。

&gt; Notes：Kubernetes  `node.kubernetes.io/not-ready` 属性和 `node.kubernetes.io/unreachable` 属性添加容差时效 `tolerationSeconds=300`。即在检测到其中问题后，Pod 将保持绑定5分钟。

## 优先级和抢占

kubernetes中也为Pod提供了优先级的机制，有了优先级机制就可以在并行系统中提供抢占机制，有了抢占机制后，当还未调度时，高优先级Pod会比低优先级Pod先被调度，在资源不足时，低优先级Pod可以被高优先级Pod驱逐。

优先级功能由 [PriorityClasses](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass) 提供。`PriorityClasses` 是作为集群级别资源而不是命名空间级别资源，只是用来声明优先级级别。

`value` 作为优先级级别，数字越大优先级级别越高。而 `name` 是这个优先级的名称，与其他资源name值相似，值的内容需要符合DNS域名约束。

`globalDefault` 是集群内默认的优先级级别，仅只有一个 `PriorityClass` 可以设置为 `true`

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: &#34;This priority class should be used for XYZ service pods only.&#34;
```

&gt; Notes:
&gt;
&gt; - 如果集群内不存在任何 `PriorityClass` ，则存在的Pod的优先级都为0
&gt; - 当对集群设置了 `globalDefault=true` 后，不会改变已经存在的 Pod 的优先级。仅对于 `PriorityClass`   `globalDefault=true` 后创建的 Pod。
&gt; - 如果删除了 `PriorityClass` ，存在还是使用的这个 `PriorityClass` 的Pod保持不变，新创建的Pod无法使用这个 `PriorityClass` 。

### 非抢占

当 `preemptionPolicy: Never` 时，Pod不会抢占其他Pod，但不可调度时，会一直在调度队列中等待调度，直到满足要求才会被调度。==非抢占式pod仍可能被其他高优先级的pod抢占==

&gt; Notes：preemptionPolicy在Kubernetes v1.24 [stable]

`preemptionPolicy` 是作为非抢占的配置，默认参数为 `PreemptLowerPriority`；表示了允许高优先级Pod抢占低优先级Pod。如果 `preemptionPolicy: Never`，代表Pod是非抢占式的。

下列是一个非抢占式的配置样例

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 1000000
preemptionPolicy: Never
globalDefault: false
description: &#34;This priority class will not cause other pods to be preempted.&#34;
```

当配置了优先级后，优先级准入控制器会使用 `priorityClassName` 中配置的对应的 `PriorityClass` 的 value值来填充当前Pod的优先级，如果没有找到对应抢占策略，则拒绝。

下面是在Pod中配置优先级的示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

### 抢占

当创建Pod之后，Pod会进入队列并等待被调度。*scheduler* 从队列中选择一个 Pod 并尝试将其调度到一个节点上。如果没有找到满足 Pod 的所有指定要求的 Node，则为 `Pending` 的 Pod 触发抢占。当Pod在找合适的节点时，即试图抢占一个节点，会在这个Node中删除一个或多个优先级低于当前Pod的Pod，使当前Pod能够被调度到对应的Node上。当低优先级Pod被驱逐后，当前Pod可以被调度到该Node，这个过程被称为抢占 `preemption`。

而提供可驱逐资源的Node成为被提名Node（`nominated Node `），在当Pod抢占到一个Node时，其 `nominatedNodeName` 会被标注为这个 Node的名称，当然标注后也不一定，一定是被抢占到这个Node之上，例如，当前Pod在等待驱逐低优先级Pod的过程中，有其他节点变成可用节点 *FN* 时，这个时候Pod会被抢占到这个节点。



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [创建一个会被调度到特定节点上的 Pod](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/assign-pods-nodes/#%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E4%BC%9A%E8%A2%AB%E8%B0%83%E5%BA%A6%E5%88%B0%E7%89%B9%E5%AE%9A%E8%8A%82%E7%82%B9%E4%B8%8A%E7%9A%84-pod)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [nodeSelector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [labels annotations taints](https://kubernetes.io/docs/reference/labels-annotations-taints/)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
&gt;
&gt; &lt;sup id=&#34;5&#34;&gt;[5]&lt;/sup&gt; [inter pod affinity and anti affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity)
&gt;
&gt; &lt;sup id=&#34;6&#34;&gt;[6]&lt;/sup&gt; [taint and toleration](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
&gt;
&gt; &lt;sup id=&#34;7&#34;&gt;[7]&lt;/sup&gt; [evictions](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#taint-based-evictions)
&gt;
&gt; &lt;sup id=&#34;8&#34;&gt;[8]&lt;/sup&gt; [pod priority preemption](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)


