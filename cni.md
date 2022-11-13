# 

任何Pod在访问出去之前，需做源地址转换，确保可使用物理机IP出口。每个Pod为了能确保可使别人访问，须在宿主机上的物理接口做DNAT，将服务暴漏出去。如C1与C2通信，此时为两级nat转换（出SNAT，入DNAT）

![9f2007ba.png](../images/CNI/9f2007ba.png)

![6efa9d20.png](../images/CNI/6efa9d20.png)

k8s之上网络通信模型

- 容器间通信：同一个Pod内的多个容器间的通信，通过lo接口
- Pod间通信：k8s集群中要求为，Pod与Pod之间通信要使用自己的IP与对端的IP能直接进行通信，不能进行地址转换（直达）。通信双方所间的地址，必须为直接通信双方的地址。
- Pod与service进行通信：Pod IP与Cluster IP之间使用iptables或IPVS规则实现通信。1.11&#43;的kube-proxy也支持IPVS类型的service。使用kubeadm部署的集群直接修改kube-proxy文件即可实现`kubectl get configmap -n kube-system`查看，修改`kube-proxy`，中定义了kube-proxy使用了那种模式在工作。
- service与集群外部客户端的通信。


k8s之上网络实现是靠CNI插件标准接入其他插件来实现的。

解决方案
- 虚拟网桥 brige，纯软件方式实现虚拟网卡接入到网桥中去。保证每个容器、Pod都有一个自己专用的网络接口，每一对虚拟网卡，一半留在Pod之上，一半留在宿主机之上并接入到网桥中去使用。能够使用物理桥接的方式。
- 多路复用，MacVLAN，使用mac的方式去创建VLAN。为每一个虚拟接口配置独有的mac地址，使得一个物理网卡能承载多个容器去使用。直接使用物理网卡，并基于物理网卡的macvlan机制进行跨节点之间通信。
- 硬件交换：SR-IOV，一个网卡直接在物理级虚拟出多个接口。单根IO序列化 singleroot。


如果期望使用CNI插件，在k8s来说很容易实现。在kubelet启动时指定目录路径来加载配置文件`/etc/cni/net.d/`，因此只要将配置文件放置此目录就会识别并加载为网络插件使用。kubelet调用配置文件指定的网络插件，由网络插件代为实现地址分配、接口创建、网络创建等各种功能。

```
[root@node03 k8s]# ls /etc/cni/net.d/10-flannel.conflist 
/etc/cni/net.d/10-flannel.conflist
```


CNI本身只是规范，定义容器网络插件应该怎么定义。目前CNI插件分为三大类，不同的网络插件在实现地址管理时略有不同。

在k8s之上，网络插件不但要实现网络地址分配、网络管理，还必须确保网络插件能够实现辅助、设置Pod与Pod之间是否能够互相访问网络策略（network policy）。flannle不支持网络策略。

flannle默认使用vxlan作为后端网络传输机制的。

组成一个叠加网络，使得两端主机之上，各自有个专门叠加报文封装的隧道，通常称之为`flannel.0`或`flannle.1`之类的接口。其拥有特定网络地址，是用来专门封装隧道协议报文的。mtu为1450留存了一部分。用来在做叠加隧道会有额外开销使用。额外空间不会被占用，只是留给隧道使用而已。

CNI接口，在flannle.1之上，被当前主机作为这隧道协议上作为本地通信时使用的地址接口。只有在创建Pod后，有容器运行时，此接口才会存在。在当前节点上不存在Pod运行时，默认情况下只存在flannle.1。

flannel，支持多种后端承载方式:

- vxlan
- host-gw(host gateway)，主机网关。每个节点上的Pod各自拥有专用网段，将主机自己的网络接口当作网关来使用。从而给Pod可以各自配置一个IP地址，并通过这个网关对外进行传递。
- UDP 使用普通UDP报文进行转发，因此 性能比vxlan与host-gw要低很多。仅在前两种都不支持的情况下使用。flannle在刚研发出时，linux内核不支持VXlan，而hostgw的技术门槛很高，故早期flannle使用的为UDP。

flannle的使用

任何一个部署了kubelet节点之上，都需要部署网络插件，因为kubelet需要借助网络插件为Pod设置网络接口、添加网络、以及激活网络等功能的。

flannle支持部署的方式

- 系统级的守护进程
- 部署在K8S之上的Pod。对此方式来讲，必须将flannle配置为“共享他所运行节点的网络名称空间的Pod”。故flannle作为Pod部署的方式，一定是一个damonset，在每一个节点上只运行一个Pod副本。而此Pod直接共享宿主机的网络名称空间，只有这样，Pod才能设置宿主机的网络名称空间。当把flannle托管运行在K8s之上作为Pod运行的话，其模拟运行为系统级的网络进程。

flannle的配置参数：
- network：flannle使用的CIDR格式的网络地址，用于为Pod分配网络功能。
- Subnetlen：将network切分子网供各节点使用时，使用多长的掩码进行切分，默认为24为。
- SubnetMin：用于分配给节点使用的起始的第一个子网从哪开始。如10.244.10.0/24
- SubnetMax：10.244.100.0/24
- Backend：指明各个Pod之间通信的方式：vxlan，host-gw，udp


[Kubernetes&#43;Docker&#43;Calico集群安装配置 - 不倒翁的博客 - CSDN博客](https://blog.csdn.net/mario_hao/article/details/80559354)

calico

使用calico需要满足一下几个条件

- kubelet必须配置使用network网络扩展 `--network-plugin=cni`
- kube0-proxy必须使用iptables模式
- kube-proxy必须不能使用 


部署过程

Ensure that the Kubernetes controller manager has the following flags set:  
`--cluster-cidr=&lt;your-pod-cidr&gt;`and`--allocate-node-cidrs=true`.

--allow_privileged=true


网络策略定义

`kubectl explain networkpolicy`

- apiVersion

- metadata

- spec
  
- - egress 出站规则 []obj
  
  - ports []object 目标端口（远程是目标）
	  		- port
	    		- protocol
  	- to   []object 目标地址，三者不同时使用。
	    		- ipBlock 目标地址是一个IP地址块，IP地址内的所有端点（Pod、主机）
	      		- namespaceSelector 名称空间选择器，控制Pod能到达其他名称空间的。（源Pod访问此名称空间内的Pod或某一个Pod）
	        		- podSelector 目标地址是另外一组Pod。
  
  -  ingress 入站规则
  	- from []object
  	- ports 目标端口（自己是目标）
  -  podSelector required 应用无论是出站还是入站，应用在哪个或哪些Pod上。
  -  policyType 策略类型，在当前的策略当中，即定义了ingress又定义了egress，可以指定在某一时刻两者只又一个生效。如不指定此字段，存在的egress与ingress都会生效（默认策略）。如果要指定某一个，需要明确指定进来。


```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: dev
spec:
  podSelector: {} # 没有任何显式定义规则。意味着默认拒绝所有。
  policyTypes: # 没有使用policyTypes定义，默认都是允许的。
  - Ingress

```

指定名称空间，标识这个规则对指定namespace生效的。

```
kubectl apply -f network.yml -n dev
```

查看名称空间网络规则

```
kubectl get netpol -n dev
```

[Kubernetes 解决spec.template.spec.containers[0].securityContext.privileged: Forbid - 看看我都干了些啥！ - ITeye博客](https://crabdave.iteye.com/blog/2367356)
