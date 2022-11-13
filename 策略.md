# 

终端用户在运行Pod时只需将请求提交给master即可，甚至不用关心Pod运行在哪个节点之上。工作节点被master组织成为一个庞大的虚拟资源池，将CPU、存储卷等资源整合为统一的视角向客户端进行提供。

Kubernetes的调度器也可以自定义的，如没有定义，也可以称之为使用的default Scheduler。

当用户请求向apiserver创建一个Pod时，apiserver检查权限无问题后，apiserver会将请求交由Scheduler，由Scheduler从众多节点中。（Scheduler为守护进程，内部有很多调度算法，默认使用default Scheduler）。因此default Scheduler会尝试从众多节点当中选择一个适用、匹配的节点来作为运行此资源Pod对象节点。其返回结果会告知apiserver，并且会记录在etcd中（在一段时间内成为持久的状态，如节点不发生故障，或资源不发生紧缺而被kill的，此节点会一直在其Node上运行）。 其后由apiserver指挥被选定节点的kubelet。而后，kube-proxy会监控着和service相关的变动，而后将其创作成当前节点上的iptables/ipvs规则。（`kube-proxy`是管理service资源的重要组件，需连接apiserver获取某些资源定义）。

Scheduler如何决策由哪一个节点来运行Pod对象的。

Scheduler在调度时，当用户创建某一个Pod时，我们有一组Pod可以使用，Scheduler需要把Pod从Nodelist（节点对象列表当中），找出一个最佳的适合运行节点的组件。如何评测谁为最佳，就是Scheduler的工作。

default Scheduler默认实现此调度决策时，是通过三个步骤来完成的，（主要工作在二级中完成）
- 预选 (Predieate)，先排除完全不能符合此Pod运行基本要求的节点。 
- 优选 (Priorites)，计算每一个节点的优先级，基于优先级找出最佳匹配的节点。
- 选定 (Select)

在Kubernetes当中，Pod当中运行容器可以定义两个纬度的资源限制
- 起始资源基本要求，满足其基本要求才可运行。
- 资源限额（上限），超出限额后不再分配任何内存。


当前占用
资源需求
资源限额


Pod中的NodeSeletor就是一种调度条件的限定。此时预选过程中范围就会缩小很多。我们所设定的很多Pod属性都有可能影响预选甚至优选步骤。

在Kubernetes之上能够使用的特殊的调度方式有以下几类：
- 节点倾向性调度 （nodeAffinity）
- PodAffinity podAntiAffinity，期望某些Pod运行在同一节点，或相邻节点。 
- 污点和污点容忍，可以给某些节点打污点标识。

常用预选策略：
- CheckNodeCondtion：检查是否节点挂载磁盘、网络或为准备好的前提下能够将Pod调度至Pod
- GeneralPredicates：通用预选策略，其并不是单独的预选策略。
    - HostName: 检查Pod对象是否定义了`pod.spec.hostname`属性的值。如Pod定义了hostname属性，则检查节点名称与此hostname是否相匹配，但此处并不是定义Pod必须运行在这个节点之上。
    - PodFitsPorts: pod.spec.containers.ports.hostPort 如果Pod中容器定义了hostPort表示定义指定绑定在Node节点的那个端口上，如节点端口被占用，则不符合条件。
    - MatchNodeSelector：pod.spec.nodeSelector是否定义，节点标签是否可以适配到此Pod的节点选择器的。
    - PodFitsResources：检查Pod的资源需求是否能被节点的空闲资源所满足。
- NoDiskConflict：检查Pod对象依赖的存储卷是否满足条件需求。目标节点能否满足Pod存储卷的使用，就表示其可用（默认未启用）。
- PodToleratesNodeTaints: 检查Pod上的`spec.tolerations`可容忍污点是否完全包含节点上的污点。
- PodToleratesNodeExecuteTaints: 不能执行的污点（默认未启用）。
- CheckNodeLabelPresence：检查节点上指定标签的存在性。
- CheckServiceAffinity：根据当前Pod对象所属的service，已有其他Pod对象，此service对象已调度至此节点，将相同service的Pod对象尽可能放置在所属的service已经调度完成的其他Pod所有的节点上去。
- CheckVolumeBinding：检查pvc是否被其他Pod绑定。
- NoVolumeZoneConflict：在当前区域中检查此节点Pod对象是否存在存储卷冲突。
- CheckNodeMemoryPressure：检查节点内存资源是否存在处于压力过大的状态。
- CheckNodePIDPressure：检查节点上PID数量资源压力过大。
- CheckNodeDiskPressure: 检查节点磁盘IO压力是否过大。
- MatchInterPodAffity：检查节点是否满足Pod倾和性或反倾和性条件。


42.32
