# lvs &#43; keepalived集群架构


## 1 LVS负载均衡集群介绍

### 1.1 搭建负载均衡服务的需求

负载均衡(Load Balance)集群提供了一种廉价、有效、透明的方法，来扩展网络设备和服务器的负载、带宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。

&gt; **搭建负载均衡服务的需求**

1. 把单台计算机无法承受的大规模的并发访问或数据流量分担到多台节点设备上分别处理，减少用户等待响应的时间，提升用户体验.
2. 单个重负载的运算分担到多台节点设备上做并发处理，每个节点设备处理结束后，将结果汇总，返回给用户，系统处理能力得到大幅度提高。
3. 7*24小时服务保证，任意一个或多个有限后面节点设备宕机，要求不能影响业务。

在负载均衡集群中，所有计算机节点都应该提供相同的服务。集群负载均衡器所截获所有对该服务的入站请求。然后将这些请求尽可能的平均分配在所有集群节点上。

### 1.2 LVS (Linux Virtual Server)介绍

LVS是Linux Virtual Server的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统，可在UNIX、Linux平台下实现负载均衡集群功能。该项目在1998年5月由章文嵩博士组织成立，是中国国内最早出现的自由软件项目之一

LVS项目介绍 http://www.linuxvirtualserver.org/zh/lvs1.html

LVS集群的体系结构 http://www.linuxvirtualserver.org/zh/lvs2.html

LVS集群中的IP负载均衡技术 http://www.linuxvirtualserver.org/zh/lvs3.html

LVS集群的负载调度 http://www.linuxvirtualserver.org/zh/lvs4.html

### 1.3 IPVS（LVS）发展史

早在2.2内核时，IPVS就已经以内核补丁的形式出现。

从2.4.23版本开始，IPVS软件就是合并到Linux内核的常用版本的内核补丁的集合。

从2.4.24以后IPVS已经成为Linux官方标准内核的一部分。

### 1.4 IPVS软件工作层次图



&lt;img src=&#34;https://raw.githubusercontent.com/CylonChau/imgbed/main/img/4af27131.png&#34; alt=&#34;4af27131&#34; style=&#34;zoom:80%;&#34; /&gt;



从上图可以看出，LVS负载均衡调度技术是在Linux内核中实现的，因此，被称之为Linux虚拟服务器（Linux virtual Server）。我们使用该软件配置LVS时候，不能直接配置内核中的ipvs，而需要使用ipvs的管理工具ipsadm进行管理.

&gt; **LVS技术点小结：**

* 真正实现调度的工具是IPVS， 工作在Linux内核层面
* LVS自导IPVS管理工具是ipvsadm
* keepalived实现管理IPVS及负载均衡器的高可用。
* RedHat工具Piranha WEB管理实现调度的工具IPVS。

### 1.5 LVS体系结构与工作原理简单描述

LVS集群负载均衡器接受服务的所有入站客户端计算机请求，并根据调度算法决定那个集群几点应该处理回复请求。负载均衡器简称(LB)有时也被成为LVS Director简称Director

LVS虚拟服务器的体系结构如下图所示，一组服务器通过告诉的局域网或者地理分布的广域网互相连接，在他们的前端有一个负载调度器（Load Balancer）。负载调度器能无缝地将网络请求调度到真实服务器上，从而使得服务器集群的结构对客户是透明的，客户访问集群系统提供的网络服务就像访问一台高性能、高可用的服务器一样。客户程序不收服务器集群的影响不需作任何修改。胸的伸缩性通过在服务集群中透明的加入和删除一各节点来达到，通过检测节点或服务进程故障和正确地重置系统达到高可用性。由于我们的负载调度技术是在Linux内核中实现的，我们称之为Linux虚拟服务器（Linux Virtual Server）。

&lt;img src=&#34;https://raw.githubusercontent.com/CylonChau/imgbed/main/img/d8e6dbb7.png&#34; alt=&#34;d8e6dbb7&#34; style=&#34;zoom:80%;&#34; /&gt;

### 1.6 LVS基本工作过程图**

&gt; **LVS基本工作过程图1：带颜色的小方块代表不同的客户端请求

![9b7099eb](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/9b7099eb.png)

&gt; **LVS基本工作过程图2：**

不同的客户端请求小方块经过负载均衡器，通过指定的分配策略被分发到后面的机器上

![0d677548](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/0d677548.png)

&gt; **LVS基本工作过程图3：**

![1a42b889](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1a42b889.png)

&gt; **LVS基本工作过程图4：**

![001c6c47](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/001c6c47.png)


### 1.7 LVS相关术语命名约定
| 名称                           | 缩写            | 说明                                                         |
| ------------------------------ | --------------- | ------------------------------------------------------------ |
| 虚拟IP地址(Virtual IP Address) | VIP&amp;nbsp;&amp;nbsp; | VIP为Direcort用于向客户端计算机提供IP地址.如www.baidu.com域名就要解析到VIP上提供服务 |
|真实IP地址(Real Server IP Address)|RIP|在集群下面节点上使用的IP地址，物理IP地址|
|Director的IP地址(Director IP Address)|DIP|Director用于连接内外网络的IP地址，物理网卡上的IP地址，是负载均衡器上的IP|
|客户端主机IP地址(Client IP Address)|CIP|客户端用户计算机请求集群服务器的IP地址，该地址用作发送给集群的请求的源IP地址|

LVS集群内部的节点称为真实服务器(Real Server)，也叫做集群节点。请求集群服务的计算机称为客户端计算机。

与计算机通常在网上交换数据包的方式相同，客户端计算机、Director和真实服务器使用IP地址彼此进行通信。


### 1.8 LVS集群的4种工作模式介绍与原理讲解

IP虚拟服务器软件IPVS

在调度器的实现技术中，IP负载金恒技术是效率最高的。在已有的IP负载均衡技术中有通过网络地址转换(Network Address Translation)将一组服务器构成一个高性能的、高可用的虚拟服务器，我们称之为VS/NAT技术(Virtual Server via Network Address Translation)，大多数商业化的IP负载均衡调度器产品都是使用NAT方法，如Cisco的LocalDirector、F5、Netscaler的Big/IP和Alteon的ACEDirector。

在分析VS/NAT的缺点和网络服务的非对称性的基础上，我们提出通过IP隧道实现虚拟服务器方法VS/TUN(Virtual Server via IP Tunneling ) 和通过直接路由实现虚拟服务器的方法VS/DR(Virtual Server via Direct Routing )，它们可以极大地提高系统的伸缩性。所以，IPVS软件实现了这三种IP负载均衡技术，淘宝开源的模式FULLNAT。

#### 1.8.1 NAT模式==&gt;网络地址转换&lt;==收费站模式

&gt; **Virtual Server via Network Address Translation (VS/NAT)**

通过网络地址转换，调度器LB重写请求报文的目标地址，根据预设的调度算法，将请求分派给后端的真实服务器；真实服务器的响应报文处理之后，返回时必须要通过调度器，经过调度器时报文的源地址被重写，再返回给客户，完成整个负载调度过程。


***
**&lt;font color=&#34;#0215cd&#34; size=2&gt; &lt;font color=&#34;#f8070d&#34; size=2&gt;⚠&lt;/font&gt; 提示：VS/NAT模式，很类似公路上的收费站，来去都要经过LB负载均衡器，通过修改目的地址，端口或源端口。&lt;/font&gt;**
***

**原理描述**

![556081af](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/556081af.png)

客户端通过Virtual IP Address（虚拟服务的IP地址）访问网络服务时，请求的报文到达调度器LB时，调度器根据连接调度算法从一组真实服务器中选出一台服务器，将报文的目标地址VIP改写成选的服务器的地址RIP1，请求报文的目标端口改写成选定服务器的相应端口（RS）提供服务端口，最后将修改后的报文发送给选出服务器RS1。同时，调度器LB在连接的Hash表中记录这个连接，当这个连接的下一个报文到达时，从连接的Hash表中可以得到原选定服务器的地址和端口，进行同样的改写操作，并将报文传给原选定的服务器RS1。当来自真实服务器RS1的相应报文返回调度器时，调度器将返回报文的源地址和源端口改为VIP和相应端口，然后调度器再把报文发给请求用户。in DNAT out SNAT。

![46a03c4c](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/46a03c4c.png)

&gt; **NAT模式小结**

1. NAT技术将请求的报文（通过DNAT方式改写）和响应的报文（通过SNAT方式改写），通过调度器地址重写然后在转发给内部的服务器，报文返回时在改写成原来的用户请求的地址。

2. 只需要在调度器LB上配置WAN公网IP即可，调度器也要有私有LAN IP和内部RS节点通信。
3. 每台内部RS节点的网关地址，必须要配成调度器LB的私有LAN内物理网卡地址 (LD1P)，这样才能确保数据报文返回时仍然经过调度器LB。
4. 由于清求与响应的数据报文都经过调度器LB.因此，网站访问早大时调度器LB有较大瓶颈，一般要求最多10-20台节点。
5. NAT模式支持对IP及端口的转换，即用户请求10.0.0.1:80,可以通过调度器转换到RS节点的10.0.0.2:8080 (DR和TUN模式不具备的）。-
6. 所有NAT内部RS节点只需配置私有LAN IP即可。
7. 由于数据包来回都需要经过调度器，因此，要开启内核转发&lt;font color=&#34;#f8070d&#34; size=3&gt;`net.ipv4.ip_forward=1`&lt;/font&gt;,当然也包括iptables防火枪的forward功能（DR和TUX模式不需要）。

#### 1.8.2 DR模式-直连路由模式

**Virtual Server via Direct Routing (VS/DR)**

VS/DR模式是通过改写请求报文的目标MAC地址，将请求发给真实服务器的，而真实服务器将相应后的处理结果直接返回给客户端用户。同VS/TUN技术一样，VS/DR技术可极大的提高集群系统的伸缩性。而且，这种DR模式没有IP隧道的开销，对集群中的真实服务器也没有必须支持IP隧道协议的要求，但是要求调度器LB与真实服务器RS都有一块物理网卡连在同一物理网段上，即必须在同一个局域网环境。

![4d7565ce](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/4d7565ce.png)

在LVS-DR配置中，Director将所有入站请求转发给集群内部节点，但集群内部的节点直接将他们的回复发给客户端计算机（没有通过Director回来）如图所示：

![c895a20d](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/c895a20d.png)

**&lt;font color=&#34;#0215cd&#34; size=2&gt; &lt;font color=&#34;#f8070d&#34; size=2&gt;⚠&lt;/font&gt; 特别提示：(VS/DR)模式是互联网使用的最多的一种模式。
&lt;/font&gt;**

***

VS/DR模式的工作流程如下图所示：它的连接调度和管理与VS/NAT和VS/TUN中的一样，它的报文转发方法和前两种又有不同，DR模式将报文直接路由给目标服务器，在VS/DR模式中，调度器根据各个真实服务器的负载情况，连接数多少等，动态地选择一台服务器，不修改目的IP地址和目的端口，也不封装IP报文，而是将请求的数据帧的MAC地址改为选出服务器的MAC地址，然后再将修改后的数据帧在与服务器组的局域网上发送。因为请求的数据帧的MAC地址是选出的真实服务器，所以真实服务器肯定可以收到这个改写了目标MAC地址的数据帧，从中可以获得该请求的IP报文。当真实服务器发现 报文的目标地址VIP是在本地的网络设备上，真实服务器处理这个报文，然后根据路由表 将响应报文直接返回给客户。

![31be91fa](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/31be91fa.png)

在VS/DR中，根据缺省的TCP/IP协议栈处理，请求报文的目标地址为VIP，响应报文的源地址肯定也为VIP，所以响应报文不需要作任何修改，可以直接返回给客户，客户认为得到正常的服务，而不会知道是哪一台服务器处理的。

VS/DR负载调度器跟VS/TUN—样只处于从客户到服务器的半连接中，按照半连接的TCP有限状态机进行状态迁移。

&gt; **原理图：IN更改目的MAC/OUT null**

![cf02c772](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/cf02c772.png)

&gt; **DR模式小结：**

1. 通过在调度器LB上修改数据包的目的MAC地址实现转发。注意，源IP地址仍然是 CIP，目的IP地址仍然是VIP。
2. 请求的报文经过调度器，而RS响应处理后的报文无需经过调度器LB，因此，并发访问量大时使用效率很高（和NAT模式比）。
3. 因DR模式是通过MAC地址的改写机制实现的转发，因此，所有RS节点和调度器LB 只能在一个局域网LAN中（小缺点）。
4. 需要注意RS节点的VIP的绑定（lo:vip/32，lo1:vip/32)和ARP抑制问题。
5. 强调下：RS节点的默认网关不需要是调度器LB的DIP而直接是IDC机房分配的上级路由器的IP (这是RS带有外网IP地址的情况），理论讲：只要RS可以出网即可，不是必须要配置外网IP。
6. 由于DR模式的调度器仅进行了目的MAC地址的改写，因此，调度器LB无法改变请求的报文的目的端口（和NAT要区别）。
7. 当前，调度器LB支持几乎所有的UNIX, LINUX系统，但目前不支持WINDOWS系统。真实服务器RS节点可以是WINDOWS系统。
8. 总的来说DR模式效率很高，但是配置也较麻烦，因此，访问量不是特别大的公司可以用haproxy/nginx取代。这符合运维的原则：简单、易用、高效。日1000-2000W PV或并发请求小于1W以下都可以考虑用haproxy/nginx（LVS NAT）模式
9. 直接对外的业务访问，例如web服务做RS节点，RS最好用公网IP地址。如果不直接对外的业务，例如：MySQL，存储系统RS节点，最好只用内部IP地址。

### 1.8.3 Virtual Server via IP Tunneling (VS/TUN)

采用NAT技术时，由于请求和响应的报文都必须经过调度器地址重写，当客户请求越来越多时，调度器的处理能力将成为瓶颈。为了解决这个问题，调度器把请求的报文通过IP隧道（相当于ipip或ipsec ）转发至真实服务器，而真实服务器将响应处理后直接返回给客户端用户，这样调度器就只处理请求的入站报文。由于一般网络服务应答数据比请求报文大很多，采用VS/TUN技术后，集群系统的最大吞吐量可以提高10倍。

它的连接调度和管理与VS/NAT中的一样，只是它的报文转发方法不同。调度器根据各个服务器的负载情况，连接数多少，动态地选择一台服务器，将原请求的报文封装在另一个IP报文中，再将封装后的IP报文转发给选出的真实服务器；真实服务器收到报文后，先将收到的报文解封获得原来目标地址为VIP地址的报文，服务器发现VIP地址被配置在本地的IP隧道设备上(此处要人为配置)，所以就处理这个请求，然后根据路由表将响应报文直接返回给客户。

![1b499670](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1b499670.png)

TUN模式
1. 负载均衡器通过把请求的报文通过IP隧道(ipip隧道)的方式(请求的报文不经过原目的地址的改写(包括MAC)，而是直接封装成另外的IP报文)，转发至真实服务器，而真实服务器将响应处理后直接返回给客户端用户。
2. 由于真实服务器将响应处理后的报文直接返回给客户端用户，因此。最好RS有一个外网IP地址，这样效率才会更高。理论上：只要能出网即可，无需外网IP地址。
3. 由于调度器LB只处理入站请求的报文。因此，此集群系统的吞吐量可以提高10倍以上，但隧道模式也会带来一定的系统开销。TUN模式适合LAN/WAN.
4. TUN模式的LAN环境转发不如DR模式效率高，而且还要考虑系统对IP隧道的支持问题。
5. 所有的RS服务器都要绑定VIP，抑制ARP，配置复杂.
6. LAN环境一般多采用DR模式，WAN环境可以用TUN模式，但是当前在，WAN环境下，请求转发更多的被haproxy/nginx/DNS调度等代理取代。因此，TUN模式在国内公司实际应用的已经很少。跨机房应用要么拉光纤成局域网，要么DNS调度，底层数据还得同步.
7. 直接对外的访问业务，例如:web服务做RS节点，最好用公网IP地址。不直接对外的业务，例如:MySQL,存储系统RS节点，最好用内部IP地址。

#### 1.8.4 FULLNAT模式 淘宝网最新开源的
&gt; **背景**

LVS当前应用主要采用DR和NAT模式，但这2种模式要求RealServer和LVS在同 一个vlan中，导致部署成本过髙：TUNNEL模式虽然可以跨vlan，但RealServer上需要部署ipip隧道模块等，网络拓扑上需要连通外网，较复杂，不易运维。
为了解决上述问题，我们在LVS上添加了一种新的转发模式：FULLNAT，该模式和NAT模式的区别是：Packet IN时，除了做DNAT,还做SNAT(用户ip-&gt;内网ip),从而实现LVS-RealServer之间可以跨vlan通讯，RealServer只需要连接到内网;

&gt; **目标**

FULLNAT将作为一种新工作镆式（同DR/NAT/TUNNEL),实现如下功能：
1. Packet IN时，目标IP更换为realserver ip，源IP更换为内网local IP；
2. Packet OUT时，目标IP更换为client IP 注：Local IP为一组内网ip地址;性能要求，和NAT比，正常转发性能下降&lt;10%。

### 1.9 ARP协议

#### 1.9.1 什么是ARP协议

ARP协议，全称“Address Resolution Protocol”，中文名称是地址解析协议，使用ARP协议可实现通过IP地址获得对应主机的的物理地址（MAC）。

在TCP/IP的网络环境下，每个互联网的主机都会被分配一个32位的IP地址，这种互联网地址是在网际范围标识主机的一种逻辑地址。为了让报文在物理网路上传输，还补习要知道对方目的主机的物理地址才行。这样就存在把IP地址变换成物理地址的地址转换问题。

在以太网环境，为了正确地向目的主机传送报文，必须把目的主机的32为IP地址转换成为目的主机48位以太网地址(MAC),这个就需要在互联层有一个服务或功能将IP地址转换为相应的物理地址(MAC)，这个服务就是ARP协议。

所谓的地址解析&#34;地址解析&#34;，就是主机在发送帧之前将目标IP地址转换成目标MAC地址的过程。ARP协议的基本功能就是通过目标设备的IP地址，查询目标设备的MAC地址，以保证主机间互相通信的顺利进行.

ARP协议和DNS有相像之处。不同点是：DNS实在域名和IP之间解析，另外ARP协议不需要配置服务，而DNS要配置服务才行。

#### 1.9.2 ARP缓存表

在每台安装有TCP/IP协议的电脑里都会有一个ARP缓存表（windows命令提示符里输入&lt;font color=&#34;#f8070d&#34; size=3&gt;`arp -a`&lt;/font&gt;即可）， 表里的IP地址与MAC地址是一一对应的。

![a4e30825](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/a4e30825.png)

```
C:\Users\CM&gt;arp -a
接口: 192.168.1.103 --- 0x3
  Internet 地址           物理地址                类型
  192.168.1.1             3c-46-d8-5d-53-87       动态
  192.168.1.255           ff-ff-ff-ff-ff-ff       静态
  224.0.0.22              01-00-5e-00-00-16       静态
  224.0.0.251             01-00-5e-00-00-fb       静态
  224.0.0.252             01-00-5e-00-00-fc       静态
  239.11.20.1             01-00-5e-0b-14-01       静态
  239.255.255.250         01-00-5e-7f-ff-fa       静态
```

arp常用命令

arp -a 查看所有记录

arp -d 清除

arp -s ip mac  绑定IP和MAC


#### 1.9.3 ARP缓存是把双刃剑
主机有了arp缓存表，可以加快arp的解析速度，减少局域网内广播风暴。
正是有了arp缓存表，给恶意黑客带来了攻击服务器主机的风险，这个就是arp欺骗攻击
切换路由器，负载均衡器等设备时，可能会导致短时网络中断.

#### 1.9.4为什么要使用ARP协议

OSI模型把网络工作分为7层，彼此不直接打交道，只通过接口(layer interface)。IP地址工作在第三层，MAC地址工作在第二层。当协议在发送数据包时，需要先封装第三层IP地址，第二层MAC地址的报头，但协议只知道目的的节点的IP地址，不知道目的节点的MAC地址，又不能跨第二、三层，所以得用ARP协议服务，来帮助获取到目的节点的MAC地址.

osi7层模型协议 包封装解封装详解 http://www.tudou.com/programs/view/sP9JY_KranA

tcp三次握手四次断开原理过程详解 http://www.tudou.com/programs/view/XjHCDedZQa8

#### 1.9.5 ARP在生产环境产生的问题及解决办法

ARP病毒，ARP欺骗
高可用服务器对之间切换时要考虑ARP缓存问题
路由器等设备无缝迁移时要考虑ARP缓存的问题，例如：更换办公室的路由器.

&gt; **ARP欺骗原理**

ARP攻击就是通过伪造IP地址和MAC地址对实现ARP欺骗的，如果一台主机中了ARP病毒，那么它就能够在网络中产生大量的ARP通信量（它会以很快的频率进行广播），以至于使网络阻塞，攻击者只要持续不断的发出伪造ARP响应包就能更改局域网中目标主机ARP缓存中的IP-MAC条目，造成网络中断或中间人攻击。

ARP攻击主要是存在于局域网网络中，局域网中若有一个人感染ARP木马，则感染该ARP木马的系统将会试图通过“ARP欺骗”手段截获所在网络内其他计算机的通信故障。

&gt; **服务器切换ARP问题**

当网络中一台提供服务的机器宕机后，当在其他运行正常的机器添加宕机的机器的IP时，会因为客户端的ARP table cache的地址解析还是宕机的机器的MAC地址。从而导致，即使在其他运行正常的机器添加宕机的机器的IP，也会发生客户依然无法访问的情况。

解决方法是：当宕机时，IP地址迁移到其他机器上时，需要通过arping命令来通知所有网络内机器清除其本地的ARP table cache，从而使得客户机访问时重新广播获取MAC地址。几乎所有的高可用软件都会考虑这个问题。

ARP广播而进行新的地址解析。

linux下具体命令：

```sh
arping -I eth0 -c 3 -s 10.0.0.162 10.0.0.253
arping -U -I eth0 10.0.0.162
```

## 2 LVS的调度算法

LVS的调度算法决定了如何在集群节点之间分布工作负荷。

当Director调度器收到来自客户端计算机访问它的VIP上的集群服务的入站请求时，Director调度器必须决定哪个集群节点应该处理请求。Director调度器可用于做出该决定的调度方法分成两个基本类别:

固定调度方法：rr wrr dh sh

动态调度算法：wlc lc lblc lblcr SED NQ(后两种官方站点没提到，编译LVS, make过程可以看到召10种调度算法见如下表格：

| 算法  | 说明                                                         |      |
| ----- | ------------------------------------------------------------ | ---- |
| rr    | 轮循调度(Round-Robin)，它将请求依次分配不同的RS节点，也就是在RS节点中均摊请求。这种算法简单，但是只适合于RS节点处理性能相差不大的情况。 |      |
| wrr   | 加权轮循调度(Weighted Round-Robin)，它将依据不同RS节点的权值分配任务。权值较高的RS将优先获得任务，并且分配到的连接数将比权值较低的RS节点更多。相同权值的RS得到相同数目的连接数。 |      |
| dh    | 目的地址哈希调度(Destination Hashing)以目的地址为关键字查找一个静态hash表来获得需要的RS。 |      |
| sh    | 源地址哈希调度(Source Hashing)以源地址为关键字查找一个静态hash表来获得需要的RS。 |      |
| wlc   | 加权最小连接数调度(Weighted Least-Connection) 假设各台RS的权值依次为Wi(I=1..n)，当前的TCP连接数依次为Ti ( I= 1..n )，依次选取Ti/Wi为最小的RS作为     下一个分配的RS。 |      |
| lc    | 最小连接数调度压(Least-Connection)，IPVS表存储了所有的活动的连接。把新的连接请求发送到当前连接数最小的RS。 |      |
| lblc  | 基于地址的最小连接数调度(Locality-Based Least-Connection)，将来自同一目的地址的请求分配给同一台RS节点，如果这台服务器尚未满负荷，分配给连接数最小的RS，并以它为下一次分配的首先考虑。 |      |
| lblcr | 基于地址带重复最小连接数调度(Locality-Based Least-Connection with Replication)对于某一目的地址，对应有一个RS子集。对此地址请求，为它分配子集中连接数最小RS;如果子集中所有服务器均已满负荷，则从集群中选择一个连接数较小服务器，将它加入到此子集并分配连接;若一定时间内，未被做任何修改，则将子集中负载最大的节点从子集删除。 |      |
| SED   | 最短的期望的延迟(Shortest Expected Delay Scheduling SED) (SED)&lt;br&gt;基于wlc算法。这个必须举例来说了，&lt;br&gt;ABC三台机器分别权重123，连接数也分别是123。那么如果使用WLC算法的话一个新请求进入时它可能会分给ABC中的任意一个。使用sed算法后会进行这样一个运算。&lt;br&gt;A(1&#43;1)/1,&lt;br&gt;B(1&#43;2)/2,&lt;br&gt;C(1&#43;3)/3,&lt;br&gt;根据运算结果，把连接交给C |      |
| NQ    | 最少队列调度(Never Queue Scheduling NQ) (NQ)&lt;br&gt;无需队列。如果有台realserver的连接数=0就直接分配过去，不需要在进行sed运算 |      |

### 2.1 LVS的调度算法的生产环境选型

(1) 一般的网络服务，如HTTP、Mail、MySQL等，常用的LVS调度算法为：

* 基本轮叫调度rr算法
* 加权最小连接调度wlc
* 加权轮训调度wrr算法

(2) 基于局部性的最少链接LBLC和带复制的基于局部性最少链接LBLCR主要适用于Web Cache和Db Cache集群，但是我们很少这样用。

(3) 源地址散列调度SH和目标地址散列调度DH可以结合使用在防火墙集群中，它们可以保证整个系统的唯一出入口。

(4) 最短预期延时调度SED和不排队调度NQ主要是对处理时间相对比较长的网络服务。
实际使用中，这些算法的适用范围不限于这些。我们最好参考内核中的连接调度算法的实现原理，根据具体的业务需求合理的选型。

## 3 安装LVS

下载地址：http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.26.tar.gz

安装前准备

```sh
# ipvs为lvs调度器，工作在内核层，先查看是否安装
lsmod|grep ip_vs 

# 以uname -r结果为准工作中如果做安装虚拟化可能有多个内核，lvs是基于内核的
ln -s /usr/src/kernels/`uname -r`/ /usr/src/linux 
```

安装依赖包

```sh
yum install libnl-devel popt-devel popt-static  #&lt;==centos7未发现问题
make &amp;&amp; make install
/sbin/ipvsadm ||  modprobe ip_vs
[root@lb_02 ipvsadm-1.26]# lsmod|grep ip_vs
ip_vs                   136798  0
nf_conntrack            105702  1 ip_vs
libcrc32c              12644  2 xfs,ip_vs
```
&gt; **VS小结**

1、Centos5.X安装lvs，使用1.2.4版本。不要用1.2.6。

2、Centos6.4安装lvs，使用1.2.6版本。并且需要先安装yum install libnl* popt*-y。

3、安装lvs后，要执行ipvsadm把ip_vs模块加载到内核。

## 4 手动配置LVS负载均衡服务

### 4.1 手工添加LVS转发

用户访问www.lb.com然后被DNS解析到vip 10.0.0.10，这个步骤是在DNS里配置的。
如果是自建DNS lb域的DNS记录设置如下

```sh
www IN A 10.0.0.10
```

如果未自建dns，需要在购买DNS域名商提供的DNS管理界面增加类似上面的DNS记录一条。这里的IP地址一定外网地址，才能正式使用，我们假设192.168.1.0/24段为外网段。修改结果类似下图(必须做真正环境才能做下面修改)

### 4.2 配置LVS虚拟IP (VIP)

```sh
ifconfig eth0:0 10.0.0.10/24 up     
route  add  -host  10.0.0.10  dev  eth0   #←添加主机路由，也可不加此行
```

因虚拟网卡网段和VIP不在一个网段，需要设置路由，windows设置路由方法如下：

```sh
route -p add 10.0.0.0/24 192.168.2.23
route print
```

到这里说明VIP

```sh
C:\Users\CM&gt;ping 10.0.0.10

正在 Ping 10.0.0.10 具有 32 字节的数据:
来自 10.0.0.10 的回复: 字节=32 时间&lt;1ms TTL=64
来自 10.0.0.10 的回复: 字节=32 时间&lt;1ms TTL=64
来自 10.0.0.10 的回复: 字节=32 时间&lt;1ms TTL=64
```

***
**&lt;font color=&#34;#0215cd&#34; size=2&gt; &lt;font color=&#34;#f8070d&#34; size=2&gt;⚠&lt;/font&gt; 提示:到这里说明VIP地址己经配好，并可以使用了。&lt;/font&gt;**
***

### 4.3 手工执行配置添加LVS服务并增加两台RS

```sh
ipvsadm -C
ipvsadm --set 30 5 60
ipvsadm -A -t 10.0.0.10:80 -s wrr
ipvsadm -A -t 10.0.0.10:80 -s wrr -p 20
ipvsadm -a -t 10.0.0.10:80 -r 192.168.2.82:80 -g -w 1
ipvsadm -a -t 10.0.0.10:80 -r 192.168.2.21 -g -w 1
ipvsadm -D -t 10.0.0.10:80 -s wrr
ipvsadm -d -t 10.0.0.10:80 -r 192.168.2.21
```

&gt; **相关参数说明**

```sh
#&lt;==清除内核虚拟服务器表中的所有记录
Either long or short options are allowed.
  --add-service       -A        add virtual service with options
  --edit-service      -E        edit virtual service with options
  --delete-service    -D        delete virtual service
  --clear             -C        clear the whole table
  --restore           -R        restore rules from stdin
  --save              -S        save rules to stdout
  --add-server        -a        add real server with options
  --edit-server       -e        edit real server with options
  --delete-server     -d        delete real server
  --list              -L|-l     list the table
  --set tcp tcpfin udp            set connection timeout values
  --tcp-service  -t service-address   service-address is host[:port]
  --udp-service  -u service-address   service-address is host[:port]
  --scheduler    -s scheduler         one of rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,
                                      the default scheduler is wlc.
  --persistent    -p [timeout]          persistent service
  --netmask       -M netmask            persistent granularity mask
  --real-server   -r server-address     server-address is host (and port)
  --gatewaying    -g                    gatewaying (direct routing) (default)
  --ipip          -i                    ipip encapsulation (tunneling)
  --masquerading  -m                    masquerading (NAT)
  --weight        -w weight             capacity of real server
  --mcast-interface interface           multicast interface for connection sync
  --connection    -c                    output of current IPVS connections
  --timeout                               output of timeout (tcp tcpfin udp)
  --stats                                 output of statistics information
```

此时，在浏览器访问10.0.0.10结果是无法访问的：因为根据LVS原理，RS在接收到包后发现不是自己IP后就丢弃了。  [IPVS(也叫LVS)的源码分析之persistent参数 - 沧海一粟 - CSDN博客](http://blog.csdn.net/raintungli/article/details/39051435)

### 4.4 手工在RS端绑定

```sh
ifconfig lo:0 10.0.0.10/32 up    #&lt;==注意，子网掩码特殊

```

每个集群节点上的环回接口 (lo) 设备上被绑定VIP地址(其广播地址是其本身，子网掩码是255.255.255.255，采取可变长掩码方式把网段划分成只含一个主机地址的目的避免ip地址冲突)允许LVS-DR集群中的集群节点接受发向该VIP地址的数据包，这会有一个非常严重的问题发生，集群内部的真实服务器将尝试回复来自正在请求VIP客户端ARP广播，这样所有的真实服务器都将声称自己拥有该VIP地址，这时客户端将有可能接发送请求数据包到某台真实服务器上，从而破坏了DR集群的负载均衡策略。因此，必须要抑制所有真实服务器响应目标地址为VIP的ARP广播，而把客户端ARP广播的响应交给负载均衡调度器。

![06e08a6d](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/06e08a6d.png)

抑制ARP响应方法如下

```sh
echo &#34;1&#34; &gt;/proc/sys/net/ipv4/conf/lo/arp_ignore
echo &#34;2&#34; &gt;/proc/sys/net/ipv4/conf/lo/arp_announce    
echo &#34;1&#34; &gt;/proc/sys/net/ipv4/conf/all/arp_ignore
echo &#34;2&#34; &gt;/proc/sys/net/ipv4/conf/all/arp_announce
```

#### 4.5.1 arp抑制技术参数说明
中文说明:，
&gt; **arp_ignore- INTEGER**

定义对目标地址为本地IP的ARP询问不同的应答模式。

0 (默认值)：回应任何网络接口上对任何本地IP地址的arp查询请求。

1：只回答目标IP地址是来访问网络接口本地地址的ARP查询请求

2：只回答目标IP地址是来访网络接口本地地址的ARP查询请求，且来访IP必须在该网络接口的子网段内。

3：不回应该网络界面的arp请求，而只对设置的唯一和连接地址做出回应。，

4-7：保留未使用。‘

8：不回应所有(本地地址)的arp查询。

&gt; **arp_announce一INTEGER**

对网络接口上，本地IP地址的发出的，ARP回应，作出相应级别的限制：确定不同程度的限制，宣布对来自本地源lP地址发出Ail，请求的接口
    

0 (默认)在任意网络接口(eth0,eth1, lo)上的任何本地地址

1：尽量避免不在该网络接口子网段的本地地址做出arp回应.当发起ARP请求的源IP地址是被设置应该经由路由达到此网络接口的时候很有用。此时会检查来访IP是否为所有接口上的子网段内ip之一。如果该来访IP不属于各个网络接口上的子网段内，那么将采用级别2的方式来进行处理。

2：一对查询目标使用最适当的本地地址，在此模式下将忽略这个IP数据包的源地址并尝试能与该地址通信的本地地址，首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址。如果没有合适的地址被发现，将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接来进发送。限制了使用本地VIP地址作为优先的网络接口。

#### 4.5.2 检查手工添加LVS转发成果

测试LVS服务的转发：

首先在客户端浏览器访问RS  http://192.168.1.6 及 RS http://192.168.1.7 确认是否RS端正常然后访问DR的VIP http://10.0.0.10， 如果经过多次测试能分别出现RS1, RS2的 不同结果内容就是表示配置成功。

***
**&lt;font color=&#34;#0215cd&#34; size=2&gt; &lt;font color=&#34;#f8070d&#34; size=2&gt;⚠&lt;/font&gt; 提示:负载均衡的算法倾向于一个客户端IP定向到一个后端服务器，以保持会话连贯性，如果用两三台机器去测试也许就不一样。&lt;/font&gt;**
***

#### 4.5.3 在访问的同时可以用命令查看状态信息

```sh
[root@lc ~]# ipvsadm -L -n --stats
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -&gt; RemoteAddress:Port
TCP  10.0.0.10:80                       88      674        0    88293        0
  -&gt; 192.168.2.21:80                  44      355        0    47851        0
  -&gt; 192.168.2.82:80                  44      319        0    40442        0
```

### 4.6 使用脚本配置lvs负载均衡服务

```sh
#!/bin/sh
VIP=&#39;10.0.0.10&#39;
. /etc/init.d/functions
Port=80

RIP=(
  192.168.2.21
  192.168.2.82
)

clean_all(){
    ipvsadm -C
    return $?
}

set_virtual_server(){
    ipvsadm --set 30 5 60
    ipvsadm -A -t $VIP:$Port -s wrr
    return $?
}

start(){
    clean_all
    [ $? -eq 0 ] &amp;&amp; set_virtual_server || return 1
    for n in ${RIP[@]}
    do
        ipvsadm -a -t $VIP:$Port -r $n:$Port -g -w 1
    done
}
stop(){
    clean_all
}

usage(){
  echo &#39;USAGE:&#39;$0 &#39;{start|stop|restart}&#39;
}

case &#34;$1&#34; in
    start|START)
    start
    ;;
    stop|STOP)
    stop
    ;;
    restart|RESTART)
    stop
    start
    ;;
    *)
    usage
esac
```

```sh
#!/bin/sh
. /etc/init.d/functions

master_ip=192.168.2.23
VIP=&#39;10.0.0.10&#39;
Port=80
RIP=(
  192.168.2.21
  192.168.2.82
)

set_virtual_server(){
  ipvsadm --set 30 5 60
  ipvsadm -A -t $VIP:$Port -s wrr
  return $?
}

check_master(){ 
  /usr/bin/ping -c 2 $master_ip &amp;&gt;/dev/null
  return $?
}

clean_all(){
  ipvsadm -C
}

start(){
  clean_all &amp;&amp; /sbin/ifconfig eth0:0 $VIP/24 up
  [ $? -eq 0 ] &amp;&amp; set_virtual_server || return 10
  for n in ${RIP[@]}
  do
    /sbin/ipvsadm -a -t $VIP:$Port -r $n:$Port -g -w 1
  done
}

stop(){
  clean_all
  /sbin/ifconfig eth0:0 $VIP/24 down
}

check_isnode(){
  num=`ipvsadm -L -n|grep 192|wc -l`
  if [ $num -lt ${#RIP[@]} ];then
    return 0
  fi
  return 11
}

check_isup(){
  num=`ifconfig eth0:0|grep $VIP|wc -l`
  if [ $num -eq 1 ];then
    return 0
  fi
  return 12
}

while true
do
  check_master
  if [ $? -ne 0 ];then
    check_isnode
    if [ $? -eq 0 ];then
      start
    fi
  else
    check_isup
    if [ $? -eq 0 ];then 
      stop
    fi
  fi
  sleep 3
done
```

## 5 常见的LVS负载均衡高可用解决方案

(1) 通过开发上面的脚本来解决，如果负载均衡器硬件坏了。几分钟或秒级别内在其它备机上完成新的部署，如果做的细的，还可以写脚本来做调度器之间的切换和接管功能。早起的方法，还是比较笨重的，目前已经不推荐使用。

(2) heartbeart&#43;lvs&#43;ldirectord脚本配置方案，这个方案同学们自己可以去搜索，这个方案中heartbeat负责VIP的切换以及资源的启动停止，ldirectord负责RS节点的健康检查，用于比较复杂，不易控制，属于早期的解决方案，现在已经很少使用了。

&gt; **heartbeat and ldirectord方案资料:**

http://www.linuxvirtualserver.org/docs/ha/heartbeat_ldirectord.html

http://www.linuxvirtualserver.org/docs/ha/ultramonkey.html

(3) 通过Redhat提供的工具piranha来配置LVS
 Piranha是REDHAT提供的一个基于Web的LVS配置软件，可以省去手工配置LVS的繁琐工作，同时，也可单独提供。cluster功能，例如，可以通过Piranh。激活Director Server的后备主机，也就是配置Director Server的双机热备功能。

(4) keepalived&#43;LVS方案，当前最优方案，因为这个方案符合简单、易用、高效的运维原则。
The Keepalived Solution http://www.linuxvirtualserver.org/docs/ha/keepalived

(5) 其他

http://bbs.linuxtone.org/thread-1402-1-1.html

LVS Documentation

http://www.linuxvirtualserver.org/zh/index.html

http://www.linuxvirtualserver.org/Documents.html#performance

http://zh.linuxvirtualserver.org/node/2230

## 6 LVS集群分发请求RS不均衡生产环境实战解决

生产环境中 `ipvsadm -L -n` 发现两台RS的负载不均衡，一台有很多请求，一台没有请求，并且没有请求的那台RS经测试服务正常，lo:VIP也有。但是就是没有请求。

```sh
TCP 172.168.1.50:3307 wrr persistent 10
  一&gt;172.168.1.51:3307          Route   1      0         0
  一&gt;172.168.1.52:3307          Route   1      8         12758
```
&gt; **问题原因：**

persistent 10的原因，persistent会话保持，当clientA访问网站的时候，LVS把请求分发给了52,那么以后clientA再点击的其他操作其他请求，也会发送给52这台机器。

&gt; **解决办法:**

到keepalived中注释掉persistent 10然后/etc/init.d/keepalived reload，然后可以看到以后负载均衡两边都请求都均衡了。

&lt;font style=&#34;background:#ffc104;&#34; size=2&gt;其它导致负载不均的原因可能有：&lt;/font&gt;

1. LVS自身的会话保持参数设置((-p 300, persistent 300)。优化:大公司尽量用cookie替代session
2. LVS调度算法设置，例如：rr, wrr, wld,lc算法。
3. 后端RS节点的会话保持参数，例如:apache的keealive参数。
4. 访问量较少的情况不均衡的现象更加明显。
5. 用户发送的请求时间长短，和请求资源多少大小。

实现会话保持的方案：

http://oldboy.blog.5lcto.com/2561410/1331316

http://oldboy.blog.51cto.com/2561410/1323468

## 7 LVS故障排错理论及实战讲讲解

排查的大的思路就是，要熟悉LVS的工作原理过程，然后根据原理过程来排查。

1. 调度器上LVS调度规则及IP的正确性。
2. RS节点上VIP绑定和arp抑制的检查。

生产处理思路：

1. 对绑定的vip做实时监控，出问题报警或者自动处理后报警。
2. 把绑定的vip做成配置文件，例如:vi /etc/sysconfig/network-scripts/lo:0

ARP抑制的配置思路：

1. 如果是单个VIP，那么可以用stop传参设置0。
2. 如果RS端有多个、VIP绑定，此时，即使是停止VIP绑定也一定不要置0。

```sh
if [ ${#VIP[@]} ];then
echo &#34;0&#34; &gt;/proc/sys/net/ipv4/conf/lo/arp_ignore
echo &#34;0&#34; &gt;/proc/sys/net/ipv4/conf/lo/arp_announce
echo &#34;0&#34; &gt;/proc/sys/net/ipv4/conf/all/arp_ignore
echo &#34;0&#34; &gt;/proc/sys/net/ipv4/conf/all/arp_announce
if
```

3. RS节点上自身提供月够的检查(DR不能端口转换)

4. 辅助排除工具有tcpdump, ping等。

5. 负载均衡和反向代理集群的三角形排查理论。

![960fc67a](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/960fc67a.png)

## 8 keepalived&#43;lvs负载均衡配置

```conf
virtual_server 10.0.0.10 80 { 
  delay_loop 3 #&lt;== 健康检查时间，单位是秒  
  lb_algo wrr #&lt;== 负载调度的算法为wlc
  lb_kind DR  #&lt;==LVS实现负载的机制，NAT/DR/TUN/FULLNAT  
  nat_mask 255.255.255.0  
  persistence_timeout 20 #&lt;==会话保持 -p的功能
  protocol TCP #&lt;==lvs4层负载均衡 tcp udp
# ipvsadm -A -t 10.0.0.10:80 -s wrr -p 20

  real_server 192.168.1.5 80 {
    weight 1 #&lt;==配置节点权值，数字越大权重越高  
    TCP_CHECK { #&lt;==健康检查
      connect_timeout 10 #&lt;==超时时间
      nb_get_retry 3 #&lt;==延迟重试次数
      delay_before_retry 3 #&lt;==重试次数
      connect_port 80 #&lt;==检查端口
    }
  }
}
# ipvsadm -a -t 10.0.0.10:80 -r 192.168.1.5 -g -w 1
```
查看负载结果

![b403805d](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/b403805d.png)

![188bf29c](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/188bf29c.png)

当master宕机后，可看到近1分钟时间进行切换

![36ac7562](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/36ac7562.png)

LVS负载均衡代码平滑上线发布思路

发布代码：

开发人员本地测试-一&gt;办公室内部测试（开发个人，测试人员）一（配置管理员）--&gt;IDC机房测试环境（测试人员）--&gt;正是服务器--&gt;运维上线--&gt;100台


一台一台下，测试完ok挂上去，此时，机器上代码不一致，用户体验就不同，此时需要下线一半，测试，测试完再下线另一半。

下线方法：

准备两套配置文件，一套配置文件含有前两台配置文件的配置，一套配置文件含有后两台配置文件的配置。最后用完整的配置文件替换。

方法二：用ipvsadm来控制节点的增加和删除。keepalived不重启节点就不会改变

生产场景测试步骤
