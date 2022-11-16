# kubernetes基础概念

[TOC]

Kubernetes集群，集群的角色都是对等的，分为主节点和工作节点。主节点之上，有三个组件分别运行为守护进程<font color="#f8070d" size=3>`API Server`</font> <font color="#f8070d" size=3>`Scheduler`</font> <font color="#f8070d" size=3>`Contorller-Manager`</font>，分别担负一定的任务。

在node节点上也有三个组件：<font color="#f8070d" size=3>`kubelet`</font>与API Server进行交互的集群角色的核心组件，<font color="#f8070d" size=3>`docker`</font>作为容器引擎来运行pod中的容器而创建的。<font color="#f8070d" size=3>`kube-proxy`</font>负责与apiserver进行通讯，当Pod内容发生改变后会生成事件，事件被kube-proxy接收，一旦发现某一service背后的Pod发生该改变，由kube-proxy负责在本地将地址反应在IPVS规则中。

### 1 lable 

为了能够统一管理一个集群当中大量的pod资源，需要给pod添加元数据信息，元数据表现为<font color="#f8070d" size=3>`Lable`</font>。格式为<font color="#f8070d" size=3>` KEY=VLAUE`</font>类型数据。在定义完成后就可使用标签选择器<font color="#f8070d" size=3>`Lable Selecter`</font>。通过定义条件，来挑选出符合条件的pod资源。

### 2 pod

在Kubernetes的集群设计中，Pod可以完全称之为有生命周期的对象，由调度器将其调度至集群中的

在k8s中，同一类pod可能不止一个。当用户的请求达到时，如何去接入请求给同类pod的那一个pod负责处理和相应。

Pod本身是需要有控制器来管理，尽量不要人手工去管理。

在k8s中，Pod有两类（非k8s官方提出，是强性分类）：

* **自主式pod**：创建后仍然需要提交给apiserver，由apiserver借助于调度器将其调度至指定的node节点，由node启动此pod。如果pod中容器出现故障需要重启容器，是由kubelet完成

* **控制器控制管理的pod**
  * ReplicationController 副本控制器，同一时间运行指定数量的pod副本。
  * ReplicaSet 副本集控制器，不直接使用。
  * Deployment 负责管理无状态应用，支持2级控制器HPA（HorizontalPodAutoscaler）
  * StatefulSet 负责管理有状态副本集
  * Daemonset 在每一个node上运行一个副本，而不是随意运行。
  * Job 运行作业
  * Ctonjob 周期性任务计划作业

### 3 service

Pod是有生命周期的，因此Pod随时都有可能被销毁。在多个Pod都提供同一服务时，客户端没法通过固定的手段访问Pod。因此为了尽可能的降低二者之间的协调的复杂度，kubernetes为每一组提供同类服务的pod，和客户端之间添加了一个<font color="#f8070d" size=2>固定</font>的中间层<font color="#f8070d" size=3>`service`</font>。 service只要不删除，他的地址和服务名称都是固定的。

service不但能提供一个稳定的访问入口，还能起到调度功能。当客户端需要访问某个服务时，只需指明service的地址或名称即可。service再把请求代理到后端的pod之上。一旦pod因故障消失，新的pod会立即被service关联进来，作为service后端的可用pod对象之一。pod上有一个固定的元数据<font color="#f8070d" size=3>`lable`</font>，service是靠标签选择器来关联pod对象的。service在将pod关联后，再动态探测pod的IP地址和端口，并作为自己可调度的后端可用服务器。主机对象。

在kubernetes中，service不是应用程序、实体组件，它只是iptables dnat规则。service作为kubernetes的对象，有service名称，相当于是服务的名称。名称可以被解析。

### 4 AddOns，附件

AddOns不作为本身一部分存在，是另外的应用程序，两者可以很紧密的结合起来，提供工作，AddOns通常被称为附加组件。

kubernetes的附件，

kubernetes的DNS可以动态改变、创建、删除、变动。如更改service名称会动态触发更改DNS中的解析记录名称。客户端在访问某一服务时，可直接访问服务的名称，由集群中DNS服务来负责解析，一般而言解析的是Service地址，而不是pod地址。因此访问是由Service代为代理实现，这种代理是由端口代理DNat来实现。但是Service背后的同一类Pod有很多使用DNAT来实现，在调度效果上不太尽如人意。因此在1.11中将iptables规则，改为ipvs规则。支持用户指定各种调度算法。


HPA水平Pod自动伸缩控制器


K8s网络模型

在k8s模型中要求拥有三种网络模型
1、pod网络：各Pod运行在一个网络中。
2、集群网络：service网络，service和pod地址是处在不同网段的。service网络不同于Pod的网络，service网络地址是虚拟的。只存在于IPVS规则中。
3、节点网络：各node节点的网络地址。

在Kubernetes集群中还存在三位通信
1、同一Pod内的多个容器间的通信：lo
2、各Pod之间的通信
3、Pod与service之间的通信


共享存储

在master主机来讲，数据并不是存在master本地，而是放在共享存储中。共享存称之为K8S的DB，这个DB是<font color="#f8070d" size=3>`etcd`</font>。etcd是键值对存储的数据库系统，和redis很相似，但是etcd还拥有很多协调功能是redis所不具备的。etcd本身是restful风格的集群。

一般情况下etcd内部通讯需要一套CA和签署证书，etcd向客户端apiserver需要CA和证书，apiserver向客户端提供服务需要另外一套CA和证书。apiserver与kubelet、kube-proxy需要证书进行认证。


CNI 

K8S通过<font color="#f8070d" size=3>`CNI`</font>（容器网络接口）插件体系来接入外部网络服务解决方案，只要遵循CNI开发服务，就能够作为kubernetes网络解决方案来使用。网络解决方案可以以附件的方式托管运行在集群之上。<font style="background:#ffc104;" size=2>此Pod为特殊Pod，虽然托管运行在集群之上，但需要共享节点的网络命名空间。</font>

目前能作为附件运行的常见CNI插件的网络功能以两个纬度，一是提供网络功能。如给Pod、Service提供IP地址。其次Kubernetes之上的网络解决方案还要求能够提供<font color="#f8070d" size=3>`Network Policy`</font>（网络策略）功能。

网络策略是允许自定义名称空间与名称空间之间，甚至同一个名称空间和各Pod之间，通过iptables规则来隔离彼此之间的互相访问。

对于K8S来讲，网络策略和网络功能是两个纬度的概念，

* flannel 实现网络配置，不支持网络策略
* calico 网络策略 网络配置
* canel


![2754a9a2.png](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/2754a9a2.png)
