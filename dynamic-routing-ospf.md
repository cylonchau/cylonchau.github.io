# Dynamic Routing - OSPF


Open Shortest Path First `OSPF`，开放的最短路径优先协议，是IETF组织开发的一个基于链路状态的内部网关协议，它的使用不受任何厂商限制，所有人都可以使用，所以称为开放的，而最短路径优先（SPF）只是OSPF的核心思想，其使用的算法是Dijkstra算法，最短路径优先并没有太多特殊的含义，并没有任何一个路由协议是最长路径优先的，所有协议，都会选最短的。

OSPF针对IPv4协议使用的是OSPF Version 2（RFC2328）；针对IPv6协议使用OSPF Version 3（RFC2740）

目的：

在OSPF出现前，网络上广泛使用RIP（Routing Information Protocol）作为内部网关协议。

由于RIP是基于距离矢量算法的路由协议，存在着收敛慢、路由环路、可扩展性差等问题，所以逐渐被OSPF取代。

OSPF作为基于链路状态的协议，能够解决RIP所面临的诸多问题。此外，OSPF还有以下优点：

- OSPF采用组播形式收发报文，这样可以减少对其它不运行OSPF路由器的影响。
- OSPF支持无类型域间选路（CIDR）。
- OSPF支持对等价路由进行负载分担。
- OSPF支持报文加密。

由于OSPF具有以上优势，使得OSPF作为优秀的内部网关协议被快速接收并广泛使用。

OSPF协议特点：

- OSPF把自治系统AS（Autonomous System）划分成逻辑意义上的一个或多个区域；
- OSPF通过LSA（Link State Advertisement）的形式发布路由；
- OSPF依靠在OSPF区域内各设备间交互OSPF报文来达到路由信息的统一；
- OSPF报文封装在IP报文内，可以采用单播或组播的形式发送。

## OSPF工作流程

**寻找邻居**

OSPF协议运行后，先寻找网络中可与自己交互链路状态信息的周边路由器，可以交互链路状态信息的路由器互为邻居

**建立邻居关系**

邻接关系可以想象为一条点到点的虚链路，他是在一些邻居路由器之间构成的。只有建立了可靠邻接关系的路由器才相互传递链路状态信息。

**链路状态信息传递**

OSPF路由器将建立描述网络链路状态的LSA Link State Advertisement，链路状态公告，建立邻接关系的OSPF路由器之间将交互LSA，最终形成包含网络完整链路状态的配置信息。

**计算路由**

获得了完整的LSBD后，OSPF区域内的每个路由器将会对该区域的网络结构有相同的认识，随后各路由器将依据LSDB的信息用SPF算法独立计算出路由。

## Router ID

OSPF Router-ID用于在OSPF domain中唯一地表示一台OSPF路由器，从OSPF网络设计的角度，我们要求全OSPF域内，**禁止出现两台路由器拥有相同的Router-ID**。

OSPF Router-ID的设定可以通过手工配置的方式，或者通过协议自动选取的方式。当然，在实际网络部署中，强烈建议手工配置OSPF的Router-ID，因为这关系到协议的稳定。

## 实验：单区域OSPF配置

![image-20210125194222939](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210125194222939.png)

配置两台路由器

```bash
[Huawei]sysname R2

[R2]interface lo0
[R2-LoopBack0]ip add 2.2.2.2 32
[R2-LoopBack0]dis this
[V200R003C00]
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255 
#
return
[R2-LoopBack0]quit

[R2]interface g0/0/0
[R2-GigabitEthernet0/0/0]ip a 20.0.0.1 24
Jan 24 2021 22:01:22-08:00 R2 %%01IFNET/4/LINK_STATE(l)[0]:The line protocol IP 
on the interface GigabitEthernet0/0/0 has entered the UP state. 
[R2-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 20.0.0.1 255.255.255.0 
#
return

[R2]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 8        Routes : 8        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        2.2.2.2/32  Direct  0    0           D   127.0.0.1       LoopBack0
       20.0.0.0/24  Direct  0    0           D   20.0.0.1        GigabitEthernet
0/0/0
       20.0.0.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet
0/0/0
     20.0.0.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet
0/0/0
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0
```

整理配置

```
sysname R2
interface lo0
ip add 2.2.2.2 32
[R2-LoopBack0]dis this
quit
interface g0/0/0
ip a 20.0.0.1 24
dis this
dis ip routing-table 
```

配置ospf协议

```bash
# 配置route-id
ospf router-id 2.2.2.2
# 选择区域
# OSPF实施了两层的分层：
# 1.骨干区域（也就是area0）
# 2.连接到骨干的区域（area1~65535）
area 0
# 声明一个路由，子网掩码为反向的
network 2.2.2.2 0.0.0.0
network 10.0.0.0 0.0.0.255
# 打印ospf对的简要信息
dis ospf peer brief 
# 显示ospf路由表
dis ospf routing
```

可以看到对应的已经学习到动态的路由

```
[R1]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 9        Routes : 9        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        1.1.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0
        2.2.2.2/32  OSPF    10   1           D   10.0.0.2        GigabitEthernet
0/0/0
       10.0.0.0/24  Direct  0    0           D   10.0.0.1        GigabitEthernet
0/0/0
       10.0.0.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet
0/0/0
     10.0.0.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet
0/0/0
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0

```

配置完成后可以看到对应的报文与状态

```
[R1-ospf-1-area-0.0.0.0]
Jan 24 2021 22:12:19-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[0]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.0.0.10, NeighborE
vent=HelloReceived, NeighborPreviousState=Down, NeighborCurrentState=Init) 
[R1-ospf-1-area-0.0.0.0]
Jan 24 2021 22:12:19-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[1]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.0.0.10, NeighborE
vent=2WayReceived, NeighborPreviousState=Init, NeighborCurrentState=ExStart) 
[R1-ospf-1-area-0.0.0.0]
Jan 24 2021 22:12:19-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[2]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.0.0.10, NeighborE
vent=NegotiationDone, NeighborPreviousState=ExStart, NeighborCurrentState=Exchan
ge) 
[R1-ospf-1-area-0.0.0.0]
Jan 24 2021 22:12:19-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[3]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.0.0.10, NeighborE
vent=ExchangeDone, NeighborPreviousState=Exchange, NeighborCurrentState=Loading)
 
[R1-ospf-1-area-0.0.0.0]
Jan 24 2021 22:12:20-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[4]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.0.0.10, NeighborE
vent=LoadingDone, NeighborPreviousState=Loading, NeighborCurrentState=Full) 
[R1-ospf-1-area-0.0.0.0]
```

## ospf的八种状态

在OSPF网络中，为了交换路由信息，邻居设备之间首先要建立邻接关系，邻居（Neighbors）关系和邻接（Adjacencies）关系是两个不同的概念。

邻居关系：OSPF设备启动后，会通过OSPF接口向外发送Hello报文，收到Hello报文的OSPF设备会检查报文中所定义的参数，如果双方一致就会形成邻居关系，两端设备互为邻居。

邻接关系：形成邻居关系后，如果两端设备成功交换DD报文和LSA，才建立邻接关系。

OSPF共有8种状态机，分别是：Down、Attempt、Init、2-way、Exstart、Exchange、Loading、Full。

- **Down**：邻居会话的初始阶段，表明没有在邻居失效时间间隔内收到来自邻居路由器的Hello数据包。
- **Attempt**：该状态仅发生在NBMA网络中，表明对端在邻居失效时间间隔（dead interval）超时前仍然没有回复Hello报文。此时路由器依然每发送轮询Hello报文的时间间隔（poll interval）向对端发送Hello报文。
- **Init**：收到Hello报文后状态为Init。
- **2-way**：收到的Hello报文中包含有自己的Router ID，则状态为2-way；如果不需要形成邻接关系则邻居状态机就停留在此状态，否则进入Exstart状态。
- **Exstart**：开始协商主从关系，并确定DD的序列号，此时状态为Exstart。
- **Exchange**：主从关系协商完毕后开始交换DD报文，此时状态为Exchange。
- **Loading**：DD报文交换完成即Exchange done，此时状态为Loading。
- **Full**：LSR重传列表为空，此时状态为Full

## 实验 多播网络ospf关系

在广播多路访问网络（Multi Access）中，所有的路由器的接口都是相同网段，这些接口两两建立OSPF邻居关系，这就意味着，网络中共有：`n(n-1)/2`。维护如此多的邻居关系不仅额外消耗资源，更增加了网络中LSA的泛洪数量。

为减小多路访问网络中的 OSPF 流量，OSPF 会在每一个MA网络（多路访问网络）选举一个指定路由器 (DR)和一个备用指定路由器 (BDR)。

DR选举规则：最高OSPF接口优先级拥有者被选作DR，如果优先级相等（默认为1），具有最高的OSPF Router-ID的路由器被选举成DR，并且DR具有非抢占性。

指定路由器 (DR)：DR 负责使用该变化信息更新其它所有 OSPF 路由器（DR Rother）。备用指定路由器 (BDR)：BDR 会监控 DR 的状态，并在当前 DR 发生故障时接替其角色。 注意OSPF为“接口敏感型协议”，DR及BDR的身份状态是基于OSPF接口的。

MA网络中，所有的DRother路由器均只与DR和BDR建立邻接关系，DRother间不建立全毗邻邻接关系。如此一来，该多路访问网络中设备需要维护的OSPF邻居关系大幅减小：M= (n-2)×2+1，LSA的泛洪问题也可以得到一定的缓解。

![image-20210125203616619](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210125203616619.png)

可以查看到对应两种状态的ospf中的角色

```bash
[R3]dis ospf peer brief

	 OSPF Process 1 with Router ID 10.10.10.10
		  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State    
 0.0.0.0          GigabitEthernet0/0/0             30.30.30.30      Full        
 0.0.0.0          GigabitEthernet0/0/0             5.5.5.5          Full        
 0.0.0.0          GigabitEthernet0/0/0             6.6.6.6          Full        
 0.0.0.0          GigabitEthernet0/0/0             7.7.7.7          Full        
 ----------------------------------------------------------------------------

[R7-ospf-1-area-0.0.0.0]dis ospf peer brief 

	 OSPF Process 1 with Router ID 7.7.7.7
		  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State    
 0.0.0.0          GigabitEthernet0/0/0             10.10.10.10      Full        
 0.0.0.0          GigabitEthernet0/0/0             30.30.30.30      Full        
 0.0.0.0          GigabitEthernet0/0/0             5.5.5.5          2-Way       
 0.0.0.0          GigabitEthernet0/0/0             6.6.6.6          2-Way       
 ----------------------------------------------------------------------------
 
 # 除了dr 与 bdr 任何机器值只与dr和bdr形成关系
 [R3]dis ospf peer 

	 OSPF Process 1 with Router ID 10.10.10.10
		 Neighbors 

 Area 0.0.0.0 interface 192.168.0.10(GigabitEthernet0/0/0)'s neighbors
 Router ID: 30.30.30.30      Address: 192.168.0.11    
   State: Full  Mode:Nbr is  Master  Priority: 1
   DR: 192.168.0.10  BDR: 192.168.0.11  MTU: 0    
   Dead timer due in 40  sec 
   Retrans timer interval: 5 
   Neighbor is up for 00:57:22     
   Authentication Sequence: [ 0 ] 

 Router ID: 5.5.5.5          Address: 192.168.0.12    
   State: Full  Mode:Nbr is  Slave  Priority: 1
   DR: 192.168.0.10  BDR: 192.168.0.11  MTU: 0    
   Dead timer due in 40  sec 
   Retrans timer interval: 5 
   Neighbor is up for 00:56:31     
   Authentication Sequence: [ 0 ] 

 Router ID: 6.6.6.6          Address: 192.168.0.13    
   State: Full  Mode:Nbr is  Slave  Priority: 1
   DR: 192.168.0.10  BDR: 192.168.0.11  MTU: 0    
   Dead timer due in 40  sec 
   Retrans timer interval: 5 
   Neighbor is up for 00:56:08     
   Authentication Sequence: [ 0 ]
```

## 实验：多区域ospf

![image-20210125221613058](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210125221613058.png)

对两个区域的路由器设置对应的配置

```bash
system-view 
sysname R10
ip address 10.0.0.1 24
in l0
ip address 1.1.1.1 32


ospf router-id 4.4.4.4
area 0
network 0.0.0.0 255.255.255.255
[R10]dis ospf peer brief 

	 OSPF Process 1 with Router ID 4.4.4.4
		  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State    
 0.0.0.0          GigabitEthernet0/0/0             5.5.5.5          Full        
 ----------------------------------------------------------------------------
```

对边界路由设置双区域

```bash
system-view 
sysname R11
ip address 10.0.0.2 24
in l0
ip address 2.2.2.2 32

interface g0/0/1
ip address 10.1.0.1 255.255.255.0 

ospf router-id 5.5.5.5
area 0
network 10.0.0.0 0.0.0.255
network 2.2.2.2 0.0.0.0


area 1
network 10.1.0.0 0.0.0.255

[R11-ospf-1-area-0.0.0.1]dis ospf peer brief 

	 OSPF Process 1 with Router ID 5.5.5.5
		  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State    
 0.0.0.0          GigabitEthernet0/0/0             4.4.4.4          Full        
 0.0.0.1          GigabitEthernet0/0/1             3.3.3.3          Full        
 ----------------------------------------------------------------------------
```

可以看到已经学习到对应的路由了

```bash
[R10]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 11       Routes : 11       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

        1.1.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0
        2.2.2.2/32  OSPF    10   1           D   10.0.0.2        GigabitEthernet
0/0/0
        3.3.3.3/32  OSPF    10   2           D   10.0.0.2        GigabitEthernet
0/0/0
       10.0.0.0/24  Direct  0    0           D   10.0.0.1        GigabitEthernet
0/0/0
       10.0.0.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet
0/0/0
     10.0.0.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet
0/0/0
       10.1.0.0/24  OSPF    10   2           D   10.0.0.2        GigabitEthernet
0/0/0
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
127.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0
255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0
```

 [ospf-test.zip](..\..\..\images\Dynamic Routing - OSPF\ospf-test.zip)
