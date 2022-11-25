# 静态路由


在因特网中，网络连接设备用来控制网络流量和保证网络数据传输质量。常见的网络连接设备有集线器（Hub）、网桥（Bridge）、交换机（Switch）和路由器（Router）。

路由器是一种典型的网络连接设备，用来进行路由选择和报文转发。路由器根据收到报文的目的地址选择一条合适的路径（包含一个或多个路由器的网络），然后将报文传送到下一个路由器，路径终端的路由器负责将报文送交目的主机。

**路由**（**routing**）就是报文从源地址传输到目的地址的活动。路由发生在OSI网络参考模型中的第三层即**网络层**。当报文从路由器到目的网段有多条路由可达时，路由器可以根据路由表中最佳路由进行转发。最佳路由的选取与发现此路由的路由协议的优先级、路由的度量有关。当多条路由的协议优先级。

路由是数据通信网络中最基本的要素。路由信息就是指导报文发送的路径信息，路由的过程就是报文转发的过程。

根据路由目的地的不同，路由可划分为：

- 网段路由：目的地为网段，IPv4地址子网掩码长度小于32位或IPv6地址前缀长度小于128位。

-  主机路由：目的地为主机，IPv4地址子网掩码长度为32位或IPv6地址前缀长度为128位。

根据目的地与该路由器是否直接相连，路由又可划分为：

- 直连路由：目的地所在网络与路由器直接相连。

- 间接路由：目的地所在网络与路由器非直接相连。

- 根据目的地址类型的不同，路由还可以分为：

- 单播路由：表示将报文转发的目的地址是一个单播地址。

- 组播路由：表示将报文转发的目的地址是一个组播地址。

![IP 路由图](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/routing-diagram-16615240071381.svg)

### 路由的优先级

对于相同的目的地，不同的路由协议（包括静态路由）可能会发现不同的路由，但这些路由并不都是最优的。事实上，在某一时刻，到某一目的地的当前路由仅能由唯一的路由协议来决定。为了判断最优路由，各路由协议（包括静态路由）都被赋予了一个优先级，当存在多个路由信息源时，具有较高优先级（取值较小）的路由协议发现的路由将成为最优路由，并将最优路由放入本地路由表中。

| 路由协议                  | 优先级                                                       |
| ------------------------- | ------------------------------------------------------------ |
| DIRECT                    | 0                                                            |
| OSPF                      | 10                                                           |
| IS-IS                     | IS-IS Level1 15<br>IS-IS Level 2                             |
| 由网关加入的路由          | 50                                                           |
| 路由器发现的路由          | 55                                                           |
| 静态路由                  | 60                                                           |
| UNR（User Network Route） | DHCP（Dynamic Host Configuration Protocol）：60<br/>AAA-Download：60<br/>IP Pool：61<br/>Frame：62<br/>Host：63<br/>NAT（Network Address Translation）：64<br/>IPSec（IP Security）：65<br/>NHRP（Next Hop Resolution Protocol）：65<br/>PPPoE（Point-to-Point Protocol over Ethernet）：65 |
| Berkeley RIP              | 100                                                          |
| 点对点接口聚集的路由      | 110                                                          |
| OSPF的扩展路由            | 140                                                          |
| OSPF ASE                  | 150                                                          |
| OSPF NSSA                 | 150                                                          |
| BGP                       | 170                                                          |
| EGP                       | 200                                                          |
| IBGP                      | 255                                                          |
| EBGP                      | 255                                                          |

1. 其中，0表示直接连接的路由，255表示任何来自不可信源端的路由；数值越小表明优先级越高。
2. 除直连路由（DIRECT）外，各种路由协议的优先级都可由用户手工进行配置。另外，每条静态路由的优先级都可以不相同。

路由器根据路由转发数据包，路由可通过手动配置和使用动态路由算法计算产生，其中**手动配置的路由就是静态路由**。

静态路由比动态路由使用更少的带宽，并且不占用CPU资源来计算和分析路由更新。但是当网络发生故障或者拓扑发生变化后，静态路由不会自动更新，必须手动重新配置。静态路由有5个主要的参数：目的地址、掩码、出接口、下一跳、优先级。

- 目的地址和掩码:

IPv4的目的地址为点分十进制格式，掩码可以用点分十进制表示，也可用掩码长度（即掩码中连续‘1’的位数）表示。当目的地址和掩码都为零时，表示静态缺省路由。

- 出接口和下一跳地址:

在配置静态路由时，根据不同的出接口类型，指定出接口和下一跳地址。

对于点到点类型的接口，只需指定出接口。因为指定发送接口即隐含指定了下一跳地址，这时认为与该接口相连的对端接口地址就是路由的下一跳地址。

对于NBMA（Non Broadcast Multiple Access）类型的接口（如ATM接口），配置下一跳IP地址。因这类接口支持点到多点网络，除了配置静态路由外，还需在链路层建立IP地址到链路层地址的映射，这种情况下，不需要指定
出接口。

对于广播类型的接口（如以太网接口）和VT（Virtual-template）接口，必须指定通过该接口发送时对应的下一跳地址。因为以太网接口是广播类型的接口，而VT接口下可以关联多个虚拟访问接口（Virtual Access Interface），这都会导致出现多个下一跳，无法唯一确定下一跳。

- 静态路由优先级

对于不同的静态路由，可以为它们配置不同的优先级，优先级数字越小优先级越高。配置到达相同目的地的多条静态
  路由，如果指定相同优先级，则可实现负载分担；如果指定不同优先级，则可实现路由备份。



### 实验： 在eNsp实现静态路由配置

通信是双向的，因此要留意往返流量（的路由）。

路由的行为是逐跳的，因此需保证沿途的每一台路由器都有路由

```bash
<Huawei>system-view 
[Huawei]sysname ar1
[ar1]interface g0/0/0
[ar1-GigabitEthernet0/0/0]ip address 192.168.10.1 24
Jan 23 2021 20:52:30-08:00 ar1 %%01IFNET/4/LINK_STATE(l)[0]:The line protocol IP
 on the interface GigabitEthernet0/0/0 has entered the UP state. 

[ar1-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 192.168.10.1 255.255.255.0 
#
return

[Huawei-GigabitEthernet0/0/1]display ip interface brief 
*down: administratively down
^down: standby
(l): loopback
(s): spoofing
The number of interface that is UP in Physical is 3
The number of interface that is DOWN in Physical is 0
The number of interface that is UP in Protocol is 3
The number of interface that is DOWN in Protocol is 0

Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              192.168.20.1/24      up         up        
GigabitEthernet0/0/1              192.168.30.1/24      up         up        
NULL0                             unassigned           up         up(s) 

# 查看所有路由
dis ip routing-table
# 删除命令
undo ip address 192.168.10.1 255.255.255.0 
# 查看bgp协议路由
dis ip routing-table
# 设置静态路由
ip routing-static 192.168.0.0 24 192.168.1.1
# 保存配置命令 <用户模式> save即可保存
```

eNSP实验拓扑：[静态路由.zip](../../../images/静态路由/静态路由.zip) 

![image-20210124001745422](../../../images/静态路由/image-20210124001745422.png)

## Linux中的路由

Linux系统的`route` 与`ip route` 命令用于显示和操作IP路由表（show/manipulate the IP routing table)。要实现两个不同的子网之间的通信，需要一台连接两个网络的路由器，或者同时位于两个网络的网关来实现。在Linux系统中，设置路由通常是为了解决以下问题：该Linux系统在一个局域网中，局域网中有一个网关，能够让机器访问internet，那么就需要将网关地址设置为该Linux机器的默认路由。需要注意的是，**命令行执行的route操作不会持久化**，当网卡重启或者机器重启之后，该路由就失效了；可以在/etc/rc.local中添加route命令来保证该路由设置永久有效。

`route -n` 命令显示的字段说明

| 字段        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| Destination | 目标网段或是主机                                             |
| Gateway     | 网关地址 [`*`标识目标是本机所属的网络，不需要路由]           |
| Genmask     | 网络掩码                                                     |
| flags       | 标记[可选如下]<br/>U - 路由是活动的<br/>H - 目标是一个主机<br/>G - 路由指向网关<br/>R - 恢复动态路由产生的表项<br/>D - 由动态路由后台程序动态的安装<br/>M - 由路由的后台程序修改<br/>! - 拒绝路由 |
| Metric      | 路由距离，到达指定网络需要的中转数[Linux内核中没有引用]      |
| Ref         | 路由项引用册数[Linux内核中没有引用]                          |
| Use         | 次路由项被路由软件查找的次数                                 |
| Iface       | 该路由表项输出的路由接口                                     |

Linux开启IP转发功能，Linux主机可以是一个路由器 `sysctl -w net.ipv4.ip_forward=1`
