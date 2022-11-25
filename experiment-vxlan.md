# 实验 - VxLAN


[toc]

## vXlan概念 实验



### 什么是VxLAN

RFC定义了虚拟扩展局域网 VXLAN （`Virtual eXtensible Local Area Network`，）扩展方案，是对传统VLAN协议的一种扩展。VXLAN采用 （`MAC
in UDP（User Datagram Protocol`）封装方式，是NVO3（`Network Virtualization over Layer 3`）中的一种网络虚拟化技术。VXLAN的特点是将L2的以太帧封装到UDP报文（即L2 over  L4）中，并在L3网络中传输。

VXLAN本质上是一种隧道技术，在源网络设备与目的网络设备之间的IP网络上，建立一条逻辑隧道，将用户报文经过特定的封装后通过这条隧道转发。从用户的角度来看，接入网络的服务器就像是连接到了一个虚拟的二层交换机的不同端口上（可把蓝色虚框表示的数据中心VXLAN网络看成一个二层虚拟交换机），可以方便地通信。

![image-20220506225057747](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506225057747.png)

### 为什么需要VxLAN

**虚拟机规模受网络设备表项规格的限制**

在传统二层网络环境下，数据报文是通过查询MAC地址表进行二层转发。服务器虚拟化后，VM的数量比原有的物理机发生了数量级的增长，伴随而来的便是VM网卡MAC地址数量的空前增加。而接入侧二层设备的MAC地址表规格较小，无法满足快速增长的VM数量。

**网络隔离能力有限**

VLAN作为当前主流的网络隔离技术，在标准定义中只有12比特，因此可用的VLAN数量仅4096个。对于公有云或其它大型虚拟化云计算服务这种动辄上万甚至更多租户的场景而言，VLAN的隔离能力无法满足。

**虚拟机迁移范围受限**

由于服务器资源等问题（如CPU过高，内存不够等），虚拟机迁移已经成为了一个常态性业务。

**什么是虚拟机动态迁移？**

所谓虚拟机动态迁移，是指在保证虚拟机上服务正常运行的同时，将一个虚拟机系统从一个物理服务器移动到另一个物理服务器的过程。该过程对于最终用户来说是无感知的，从而使得管理员能够在不影响用户正常使用的情况下，灵活调配服务器资源，或者对物理服务器进行维修和升级。

在服务器虚拟化后，虚拟机动态迁移变得常态化，为了保证迁移时业务不中断，就要求在虚拟机迁移时，不仅虚拟机的IP地址、MAC地址等参数保持不变，而且虚拟机的运行状态也必须保持原状（例如TCP会话状态），所以虚拟机的动态迁移只能在同一个二层域中进行，而不能跨二层域迁移。

### VxLAN方案

为了应对传统数据中心网络对服务器虚拟化技术的限制，VXLAN技术应运而生，其能够很好的解决上述问题。

**针对虚拟机规模受设备表项规格限制**

VXLAN将管理员规划的同一区域内的VM发出的原始报文封装成新的UDP报文，并使用物理网络的IP和MAC地址作为外层头，这样报文对网络中的其他设备只表现为封装后的参数。因此，极大降低了大二层网络对MAC地址规格的需求。

**针对网络隔离能力限制**

在传统的VLAN网络中，标准定义所支持的可用VLAN数量只有4000个左右。VXLAN引入了类似VLAN ID的用户标识，称为VXLAN网络标识VNI（VXLAN Network Identifier），由24比特组成，支持多达16M的VXLAN段，有效得解决了云计算中海量租户隔离的问题。

**针对虚拟机迁移范围受限**

VXLAN将VM发出的原始报文进行封装后通过VXLAN隧道进行传输，隧道两端的VM不需感知传输网络的物理架构。这样，对于具有同一网段IP地址的VM而言，即使其物理位置不在同一个二层网络中，但从逻辑上看，相当于处于同一个二层域。即VXLAN技术在三层网络之上，构建出了一个虚拟的大二层网络，只要虚拟机路由可达，就可以将其规划到同一个大二层网络中。这就解决了虚拟机迁移范围受限问题。

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/5448c00ff2b14df88b112f3ee7923c6c.jpg)

### VxLAN与VLAN之间的区别

VLAN是传统的网络隔离技术，在标准定义中VLAN的数量只有4096，无法满足大型数据中心的租户间隔离需求。另外，VLAN的二层范围一般较小且固定，无法支持虚拟机大范围的动态迁移。

VXLAN完美地弥补了VLAN的上述不足，一方面通过VXLAN中的24比特VNI字段，提供多达16M租户的标识能力，远大于VLAN的4096；另一方面，VXLAN本质上在两台交换机之间构建了一条穿越数据中心基础IP网络的虚拟隧道，将数据中心网络虚拟成一个巨型“**二层交换机**”，满足虚拟机大范围动态迁移的需求。

![image-20210202234615267](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210202234615267.png)

- VXLAN Header

  增加VXLAN头（8字节），其中包含24比特的**VNI**字段，用来定义VXLAN网络中不同的租户。此外，还包含**VXLAN Flags**（8比特，取值为00001000）和两个保留字段（分别为24比特和8比特）。

- UDP Header

  VXLAN头和原始以太帧一起作为UDP的数据。UDP头中，目的端口号（VXLAN Port）固定为4789，源端口号（UDP Src. Port）是原始以太帧通过哈希算法计算后的值。

- Outer IP Header

  封装外层IP头。其中，源IP地址（Outer Src. IP）为源VM所属VTEP的IP地址，目的IP地址（Outer Dst. IP）为目的VM所属VTEP的IP地址。

- Outer MAC Header

  封装外层以太头。其中，源MAC地址（Src. MAC Addr.）为源VM所属VTEP的MAC地址，目的MAC地址（Dst. MAC Addr.）为到达目的VTEP的路径中下一跳设备的MAC地址。



## 实验：创建一个VxLAN网络

实现网络拓扑图，使用VXLAN在两台TOR交换机之间建立了一条隧道，将服务器发出的原始数据帧加以“包装”，好让原始报文可以在承载网络（比如IP网络）上传输。当到达目的服务器所连接的TOR交换机后，离开VXLAN隧道，并将原始数据帧恢复出来，继续转发给目的服务器。

![https://download.huawei.com/mdl/image/download?uuid=9aae9ee2cc864c6793e6a93792426a98](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/9aae9ee2cc864c6793e6a93792426a98.jpg)

### 什么是VXLAN VTEP

VXLAN隧道端点，VTEP（VXLAN Tunnel Endpoints）是VXLAN网络的边缘设备，是VXLAN隧道的起点和终点，VXLAN对用户原始数据帧的封装和解封装均在VTEP上进行。

VTEP是VXLAN网络中绝对的主角，VTEP既可以是一台独立的网络设备（比如华为的CloudEngine系列交换机），也可以是在服务器中的虚拟交换机。源服务器发出的原始数据帧，在VTEP上被封装成VXLAN格式的报文，并在IP网络中传递到另外一个VTEP上，并经过解封转还原出原始的数据帧，最后转发给目的服务器。

### 什么是VXLAN VNI

VNI（VXLAN Network Identifier，VXLAN 网络标识符），VNI是一种类似于VLAN ID的用户标识，一个VNI代表了一个租户，属于不同VNI的虚拟机之间不能直接进行二层通信。在VXLAN报文封装时，给VNI分配了24比特的长度空间，使其可以支持海量租户的隔离。

- 二层VNI是普通的VNI，以1：1方式映射到广播域BD，实现VXLAN报文同子网的转发。
- 三层VNI和VPN实例进行关联，用于VXLAN报文跨子网的转发（三层VNI的工作详情将在另外一篇EVPN相关的文档中展开描述）。

VNI的出现，就是专门解决以太网数据帧中VLAN只占了12比特的空间，这使得VLAN的隔离能力在数据中心网络中力不从心的问题

### 完成VxLAN网络架构

使用VxLAN完成192.168.100.1 和 192.168.100.2之间的互联互通。模拟器使用eNSP、Underlay网络使用OSPFv2、Overlay使用VxLAN。

![image-20210203000139766](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210203000139766.png)

#### eNSP中配置VxLAN

在eNSP中只有华为的CE设备（CloudEngine系列交换机）支持VxLAN

**SW1**

```
system-view
sysname SW1
vlan 10
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan all

interface GigabitEthernet0/0/2
port link-type access
port default vlan 10

dis ip interface brief 
```

**SW2**

```
system-view
sysname SW2

vlan 10
interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan all

interface GigabitEthernet0/0/2
port link-type access
port default vlan 10

dis ip interface brief 
```

**CE1**

```bash
system-view
sysname CE1

bridge-domain 10

interface GE1/0/0
# 因为CE设备默认关闭了端口
undo shutdown

# 将接口模式设置为2层模式
interface GE1/0/0.10 mode l2

# 数据包的封装
# 路由器上配置trunk的封装协议的命令
# dot1q中继封装，10指的是vlan 10
encapsulation dot1q vid 10
commit
quit

# 桥接域，连接两个不同的网段使用
bridge-domain 10
vxlan vni 10
commit

# 将bg桥与端口关联
interface GE1/0/0.10
bridge-domain 10
commit

### 
interface GE1/0/1
# 关闭默认交换口，二层设备无法配置IP地址
undo portswitch
undo shutdown
ip address 172.16.0.1 255.255.255.0
commit


# 配置ospf
interface LoopBack0
ip address 1.1.1.1 255.255.255.255
quit
commit
ospf router-id 1.1.1.1
area 0.0.0.0
network 0.0.0.0 255.255.255.255
commit

# 创建VxLAN隧道
interface Nve1   # 创建逻辑接口NVE 1
source 1.1.1.1   # 配置源VTEP的IP地址（推荐使用Loopback接口的IP地址）
## vni 10的头端复制列表为对端
vni 10 head-end peer-list 2.2.2.2
commit
```

CE2

```bash
system-view
sysname CE2


interface GE1/0/0
# 因为CE设备默认关闭了端口
undo shutdown

# 将接口模式设置为2层模式
interface GE1/0/0.10 mode l2

# 数据包的封装
# 路由器上配置trunk的封装协议的命令
# dot1q中继封装，10指的是vlan 10
encapsulation dot1q vid 10
commit
quit

# 桥接域，连接两个不同的网段使用
bridge-domain 10
vxlan vni 10
commit

# 将bg桥与端口关联
interface GE1/0/0.10
bridge-domain 10
commit

### 
interface GE1/0/1
# 关闭默认交换口，二层设备无法配置IP地址
undo portswitch
undo shutdown
ip address 172.16.0.2 255.255.255.0
commit


# 配置ospf
interface LoopBack0
ip address 2.2.2.2 255.255.255.255
quit
commit
ospf router-id 2.2.2.2
area 0.0.0.0
network 0.0.0.0 255.255.255.255
commit

# 创建VxLAN隧道
interface Nve1   # 创建逻辑接口NVE 1
source 2.2.2.2   # 配置源VTEP的IP地址（推荐使用Loopback接口的IP地址）
## vni 10的头端复制列表为对端
vni 10 head-end peer-list 1.1.1.1
commit
```

实验文件

[vxlan.zip](..\..\..\images\vxlan\vxlan.zip) 

 [vtep_g1_0_0.pcapng](..\..\..\images\vxlan\vtep_g1_0_0.pcapng) 

 [vtep_g1_0_1.pcapng](..\..\..\images\vxlan\vtep_g1_0_1.pcapng) 

## reference

[huawei_vxlan_guide](https://support.huawei.com/enterprise/zh/doc/EDOC1100087027#ZH-CN_TOPIC_0254803584)




