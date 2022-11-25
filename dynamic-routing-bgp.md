# Dynamic Route BGP


### 概述

BGP `Border Gateway Protocol` 边界网关协议,，是一种运行于TCP上的自治系统AS（Autonomous System）之间的路由可达，并选择最佳路由的距离矢量路由协议。

早期发布的三个版本分别是BGP-1、BGP-2和BGP-3，1994年开始使用BGP-4，2006年之后单播IPv4网络使用的版本是BGP-4，其他网络（如IPv6等）使用的版本是MP-BGP。

MP-BGP是对BGP-4进行了扩展，来达到在不同网络中应用的目的，BGP-4原有的消息机制和路由机制并没有改变。MP-BGP在IPv6单播网络上的应用称为BGP4+，在IPv4组播网络上的应用称为MBGP（Multicast BGP）。

### 历史

为方便管理规模不断扩大的网络，网络被分成了不同的自治系统。1982年，外部网关协议EGP（Exterior Gateway Protocol）被用于实现在AS之间动态交换路由信息。但是EGP设计得比较简单，只发布网络可达的路由信息，而不对路由信息进行优选，同时也没有考虑环路避免等问题，很快就无法满足网络管理的要求。

BGP是为取代最初的EGP而设计的另一种外部网关协议。不同于最初的EGP，BGP能够进行路由优选、避免路由环路、更高效率的传递路由和维护大量的路由信息。

虽然BGP用在AS之间传递路由信息，但并非所有AS之间传递路由信息都要运行BGP。如数据中心上行到Internet的出口上，为了避免Internet海量路由对数据中心内部网络影响，设备采用静态路由代替BGP与外部网络通信。

受益

BGP从多方面保证了网络的安全性、灵活性、稳定性、可靠性和高效性：

BGP采用认证和GTSM的方式，保证了网络的安全性。

BGP提供了丰富的路由策略，能够灵活的进行路由选路，并且能指导邻居按策略发布路由。

BGP提供了路由聚合和路由衰减功能用于防止路由振荡，有效提高了网络的稳定性。

BGP使用TCP作为其传输层协议（端口号为179），并支持BGP与BFD联动、BGP Tracking和BGP GR和NSR，提高了网络的可靠性。

在邻居数目多、路由量大且大多邻居有相同出口策略场景下，BGP用按组打包技术极大提高了BGP打包发包性能。

## BGP相关名词说明

| 名词                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| BGP                 | 边界网关协议（Border Gateway Protocol）是互联网上一个核心的去中心化自治路由协议，它通过维护IP路由表或前缀表来实现自治系统（AS）之间的可达性<br/>大多数ISP使用BGP来与其他ISP创建路由连接，特大型的私有IP网络也可以使用BGP<br/>BGP的通信对端（对等实体，Peer）通过TCP（端口179）会话交换数据，BGP路由器会周期地发送19字节的保活消息来维护连接。在路由协议中，只有BGP使用TCP作为传输层协议 |
| IBGP                | 内部边界网关协议。同一个AS内部的两个或多个对等实体之间运行的BGP被称为IBGP |
| IGP                 | 内部网关协议。同一AS内部的对等实体（路由器）之间使用的协议，它存在可扩容性问题：<br/>1. 一个IGP内部应该仅有数十（最多小几百）个对等实体<br/>2. 对于端点数，也存在限制，一般在数百（最多上千）个Endpoint级别<br/>IBGP和IGP都是处理AS内部路由的，仍然需要IGP的原因是：<br/>1. IBGP之间是TCP连接，也就意味着IBGP邻居采用的是逻辑连接的方式，两个IBGP连接不一定存在实际的物理链路。所以需要有IGP来提供路由，以完成BGP路由的递归查找<br/>2. BGP协议本身实际上并不发现路由，BGP将路由发现的工作全部移交给了IGP协议，它本身着重于路由的控制 |
| EBGP                | 外部边界网关协议。归属不同的AS的对等实体之间运行的BGP称为EBGP |
| AS                  | 自治系统（Autonomous system），一个组织（例如ISP）管辖下的所有IP网络和路由器的整体<br/><br/>参与BGP路由的每个AS都被分配一个唯一的自治系统编号（ASN）。对BGP来说ASN是区别整个相互连接的网络中的各个网络的唯一标识。64512到65535之间的ASN编号保留给专用网络使用 |
| Route<br/>Reflector | 同一AS内如果有多个路由器参与BGP路由，则它们之间必须配置成全连通的网状结构——任意两个路由器之间都必须配置成对等实体。由于所需要TCP连接数是路由器数量的平方，这就导致了巨大的TCP连接数<br/><br/>为了缓解这种问题，BGP支持两种方案：Route Reflector、Confederations<br/><br/>路由反射器（Route Reflector）是AS内的一台路由器，其它所有路由器都和RR直接连接，作为RR的客户机。RR和客户机之间建立BGP连接，而客户机之间则<br/>不需要相互通信<br/><br/>RR的工作步骤如下：<br/>1. 从非客户机IBGP对等实体学到的路由，发布给此RR的所有客户机<br/>2. 从客户机学到的路由，发布给此RR的所有非客户机和客户机<br/>3. 从EBGP对等实体学到的路由，发布给所有的非客户机和客户机<br/>RR的一个优势是配置方便，因为只需要在反射器上配置 |
| 工作负载            | Workload，即运行在Calico节点上的虚机或容器                   |
| 全互联              | 全互联网络（Full node-to-node Mesh）是指任何两个Calico节点都进行配对的L3连接模式 |


### BGP 应用

国内IDC机房需要在 `CNNIC` (中国互联网信息中心)或 `APNIC` (亚太网络信息中心)申请自己的IP地址段和AS号，然后将自己的IP地址广播到其它网络运营商的AS中，并**通过BGP协议将多个AS进行连接，从而实现可自动跨网访问**。此时，当用户发出访问请求后，将根据BGP协议的机制自动在已建立连接的多个AS之间为用户提供最佳路由，从而实现不同网络运营商用户的高速访问同一机房资源。

![image-20221025173224813](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025173224813.png)

### BGP的运行

BGP使用**TCP**为传输层协议，TCP端口号**179**。路由器之间的BGP会话**基于TCP连接而建立**。运行BGP的路由器被称为BGP发言者（BGP Speaker），或BGP路由器。两个建立BGP会话的路由器互为对等体（或称通信对端/对等实体，peer）。BGP对等体之间交换BGP路由表。

BGP路由器只发送增量的BGP路由更新，或进行触发更新（不会周期性更新）。

BGP具有丰富的路径属性和强大的路由策略工具。

BGP能够承载大批量的路由前缀，用于大规模的网络中。

### IBGP And EBGP

**同一个AS自治系统中的两个或多个对等实体之间运行的BGP被称为iBGP**（Internal/Interior BGP）。归属**不同的AS的对等实体之间运行的BGP称为eBGP**（External/Exterior BGP）。在AS边界上与其他AS交换信息的路由器被称作边界路由器（border/edge router），边界路由器之间互为eBGP对端。

iBGP和eBGP的区别主要在于转发路由信息的行为。例如，从eBGP peer获得的路由信息会分发给所有iBGP peer和eBGP peer，但从iBGP peer获得的路由信息仅会分发给所有eBGP peer。所有的iBGP peer之间需要全互联。

**总结**

IBGP（Internal BGP）：位于相同自治系统的BGP路由器之间的BGP邻接关系。

- 两台路由器之间要建立IBGP对等体关系，必须满足两个条件：

- 两个路由器所属AS需相同（也即AS号相同）。

- 在配置BGP时，Peer命令所指定的对等体IP地址要求路由可达，并且TCP连接能够正确建立

EBGP（External BGP）：位于不同自治系统的BGP路由器之间的BGP邻接关系。

- 两台路由器之间要建立EBGP对等体关系，必须满足两个条件：

- 两个路由器所属AS不同（也即AS号不同）。

- 在配置BGP时，Peer命令所指定的对等体IP地址要求路由可达，并且TCP连接能够正确建立.

### 实验：配置BGP

R1、R2及R3属于AS 123，R4属于AS 400；

AS123内的R1、R2及R3运行OSPF，通告各自直连接口（包括三台设备的Loopback0接口），注
意OSPF域的工作范围；所有设备的Loopback0接口地址为 `x.x.x.x/32`，其中x为设备编号（如R1的接口地址为 `1.1.1.1/32`）。

R3与R4之间建立EBGP对等体关系，R2暂时不运行BGP，R1与R3之间建立IBGP对等体关系，
所有的BGP对等体关系基于直连接口建立；R4将直连路由`4.4.4.4/32`通告到BGP，要求R1能学习到
BGP路由`4.4.4.4/32`；

修改BGP配置，使得R1与R3基于Loopback0接口建立IBGP对等体关系

![image-20221025173238524](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025173238524.png)

[eNSP BGP实验](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/basic bgp.zip) 

**配置ospf**

```bash
ospf router-id 1.1.1.1
area 0
# 这里声明的路由为单独的，否则声明全部的会使用ospf学到对应的路由
network 10.0.0.0 255.255.255.0
# 这里通过的L0接口进行bgp链接的，所以需要声明该路由。否则ospf学习不到无法链接
network 1.1.1.1 255.255.255.255
[R1-ospf-1-area-0.0.0.0]dis this
[V200R003C00]
#
 area 0.0.0.0 
  network 0.0.0.0 255.255.255.255 
```

配置bgp

```bash
bgp 123
# 设置route-id
router-id 1.1.1.1
# 自治系统号码为123
peer 3.3.3.3 as-number 123
# 建立链接的接口使用的L0，如不指定，默认使用出接口做连接。
peer 3.3.3.3 connect-interface LoopBack0

# 设置另外一个路由器
bgp 123
router-id 3.3.3.3
peer 1.1.1.1 as-number 123 
peer 1.1.1.1 connect-interface LoopBack0
# 声明一个bgp路由
network 33.33.33.33 255.255.255.255
```

此时可以看到对应的路由已经学习到了

```bash
[R1]dis bgp routing-table 

 BGP Local router ID is 1.1.1.1 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete


 Total Number of Routes: 2
Network NextHop     MED        LocPrf    PrefVal     Path/Ogn

i   4.4.4.4/32     10.2.0.2     0         100        0      400i
*>i 33.33.33.33/32 3.3.3.3      0         100        0      i
```

配置一个ebgp

```bash
# 这是两个路由器间的bgp配置。因为L0没有对应的互通接口所以使用默认出接口进行链接。
peer 10.2.0.1 as-number 123 # 与123 as里的bgp形成一个对等实体
peer 10.2.0.2 as-number 400 # 与400 as里的bgp形成一个对等实体
# 在R4上声明一个路由
network 4.4.4.4 255.255.255.255

dis bgp ip routing-table	
# 在R3上，可以看到通过eBGP学习到的到4.4.4.4的路由
[R3-bgp]dis ip routing-table 
4.4.4.4/32  EBGP    255  0    D   10.2.0.2    GigabitEthernet0/0/1
```

`refresh bgp external import ` 刷新BGP



## BGP的RR

由于IBGP水平分割的存在，为了保证所有的BGP路由器都能学习到完整的BGP路由，就必须在AS内实现IBGP全互联，这就导致AS内部需要维护大量的BGP连接，从而影响网络性能，路由反射器（Route Reflector，RR）可以“放宽”水平分割原则，解决该问题。

为保证IBGP对等体之间的连通性，需要在IBGP对等体之间建立全连接关系。假设在一个AS内部有n台设备，那么建立的IBGP连接数就为`n(n-1)/2`。当设备数目很多时，设备配置将十分复杂，而且配置后网络资源和CPU资源的消耗都很大。在IBGP对等体间使用路由反射器可以解决以上问题。

![image-20221025173307247](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025173307247.png)

> BGP反射规则
>
> BGP RR在接收到BGP路由时
>
> 如果该路由学习自非Client IBGP对等体，则反射给自己所有的Client；
>
> 如果路由学习自Client，则反射给所有非Client IBGP对等体和除了该Client之外的所有Client（华为设备可通过命令关闭RR在Client之间的路由反射行为）；
>
> 如果路由学习自EBGP对等体，则发送给所有Client和非Client IBGP对等体。
>
> ![image-20221025173318988](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025173318988.png)

BGP RR的配置

 [BGP RouteReflector.zip](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/BGP RouteReflector.zip) 

![image-20221025173332401](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025173332401.png)

```bash
# 进入系统视图
system-view 
# 设置路由器的名称
sysname R2
# 进入g0/0/0接口
in g0/0/0
ip address 10.0.0.2 24
# 配置l0口
in l0
ip address 2.2.2.2 32
# 只有R1需要配置次步骤
in g0/0/1
ip address 10.1.0.1 24
# 查看接口的信息
dis ip in b	
```

配置ospf

```
ospf router-id 1.1.1.1
area 0
network 1.1.1.1 0.0.0.0
network 10.0.0.0 0.0.0.255
network 10.1.0.0 0.0.0.255
```



```bash
bgp 123
 router-id 1.1.1.1
 peer 2.2.2.2 as-number 123 
 peer 2.2.2.2 connect-interface LoopBack0
 peer 3.3.3.3 as-number 123 
 peer 3.3.3.3 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 2.2.2.2 enable
  peer 2.2.2.2 reflect-client
  peer 3.3.3.3 enable
  peer 3.3.3.3 reflect-client
```

配置路由反射器反射客户端

```bash
# 此处是1.1.1.1的配置
# 以1.1.1.1 为中心 指定2.2.2.2为反射客户端 
peer 2.2.2.2 reflect-client
peer 3.3.3.3 reflect-client
```


