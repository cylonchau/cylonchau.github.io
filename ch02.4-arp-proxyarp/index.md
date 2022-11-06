# arp proxy&amp;arp


## 什么是arp

地址解析协议，`Address Resolution Protoco`，使用ARP协议可实现通过IP地址获得对应主机的的物理地址（MAC）

在TCP/IP的网络环境下，每个互联网的主机都会被分配一个32位的IP地址，这种互联网地址是在网际范围标识主机的一种逻辑地址。为了让报文在物理网路上传输，还补习要知道对方目的主机的物理地址才行。这样就存在把IP地址变换成物理地址的地址转换问题。

在以太网环境，为了正确地向目的主机传送报文，必须把目的主机的32为IP地址转换成为目的主机48位以太网地址(MAC),这个就需要在互联层有一个服务或功能将IP地址转换为相应的物理地址(MAC)，这个服务就是ARP协议.

所谓的&#34;地址解析&#34;，就是主机在发送帧之前将目标IP地址转换成目标MAC地址的过程。ARP协议的基本功能就是通过目标设备的IP地址，查询目标设备的MAC地址，以保证主机间互相通信的顺利进行.

ARP协议和DNS有相像之处。不同点是：DNS实在域名和IP之间解析，另外ARP协议不需要配置服务，而DNS要配置服务才行。

### ARP缓存表

在每台安装有TCP/IP协议的设备都会有一个ARP缓存表（windows命令提示符里输入`arp -a`即可）， 表里的IP地址与MAC地址是一一对应的。

![image-20221025172742176](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025172742176.png)

```bash
C:\Users\CM&gt;arp -a
接口: 192.168.1.103 --- 0x3
  Internet 地址         	物理地址              	类型
  192.168.1.1           	3c-46-d8-5d-53-87     	动态
  192.168.1.255         	ff-ff-ff-ff-ff-ff    	静态
  224.0.0.22            	01-00-5e-00-00-16     	静态
  224.0.0.251           	01-00-5e-00-00-fb    	静态
  224.0.0.252           	01-00-5e-00-00-fc     	静态
  239.11.20.1           	01-00-5e-0b-14-01     	静态
  239.255.255.250       	01-00-5e-7f-ff-fa     	静态
```

### arp常用命令

`arp -a` 查看所有记录

`arp -d` 清除arp表

`arp -s $ip $mac`   将绑定IP和MAC

`arp -n` 不解析名称打印arp表

### ARP缓存是把双刃剑

主机有了arp缓存表，可以加快arp的解析速度，减少局域网内广播风暴。

正是有了arp缓存表，给恶意黑客带来了攻击服务器主机的风险，这个就是arp欺骗攻击

切换路由器，负载均衡器等设备时，可能会导致短时网络中断（发送广播）。

### 为什么要使用ARP协议

OSI模型把网络工作分为七层，彼此不直接打交道，只通过接口(layer interface)。IP地址工作在第三层，MAC地址工作在第二层。在局域网中，当主机或其它三层网络设备有数据要发送给另一台主机或三层网络设备时，它需要知道对方的网络层地址（即IP地址）。但是仅有IP地址是不够的，因为IP报文必须封装成帧才能通过物理网络发送，因此发送方还需要知道接收方的物理地址（即MAC地址），但又不能跨第二、三层，所以需要用ARP协议服务，来帮助获取到目的节点的MAC地址。ARP可以实现将IP地址解析为MAC地址。主机或三层网络设备上会维护一张ARP表，用于存储IP地址和MAC地址的关系。一般ARP表项包括动态ARP表项和静态ARP表项。

## 模拟一个环境抓包分析arp数据包的内容

```
[Huawei]         系统视图
&lt;Huawei&gt;         用户视图，开机命令行进入的就是用户视图
system-view      用户视图切换系统视图
interface g0/0/0 选择接口
display arp all  查看arp表
arp-porxy enable 开启代理ARP功能
```

```bash
&lt;Huawei&gt;dis this 
#
return

# 设置g0/0/0接口ip
[Huawei-GigabitEthernet0/0/0]ip a 192.168.0.1 24
Jan 19 2021 17:56:33-08:00 Huawei %%01IFNET/4/LINK_STATE(l)[0]:The line protocol
 IP on the interface GigabitEthernet0/0/0 has entered the UP state. 
[Huawei-GigabitEthernet0/0/0]
[Huawei-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 192.168.0.1 255.255.255.0 
#
return

# 设置g0/0/1 ip
[Huawei-GigabitEthernet0/0/0]int g0/0/1
[Huawei-GigabitEthernet0/0/1]ip a 192.168.1.1 24
Jan 19 2021 17:57:02-08:00 Huawei %%01IFNET/4/LINK_STATE(l)[1]:The line protocol
 IP on the interface GigabitEthernet0/0/1 has entered the UP state. 
```

此时查看pc与路由器的arp表

```
$ arp -a

Internet Address    Physical Address    Type

$ 
```

ping另外一台设备 192.168.2.2

```
$ ping 192.168.1.2

Ping 192.168.1.2: 32 data bytes, Press Ctrl_C to break
Request timeout!
From 192.168.1.2: bytes=32 seq=2 ttl=127 time=15 ms
From 192.168.1.2: bytes=32 seq=3 ttl=127 time=16 ms

--- 192.168.1.2 ping statistics ---
  3 packet(s) transmitted
  2 packet(s) received
  33.33% packet loss
  round-trip min/avg/max = 0/15/16 ms
```

![image-20210119185418021](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210119185418021.png)

另外一段抓包可以看到对应收到的广播

![image-20210119185433046](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210119185433046.png)

以太网arp数据包段说明

| 目的mac地址                           | 源mac地址                             | 帧类型 | 硬件类型 | 上层协议类型 | mac地址长度 | ip地址长度 | 操作类型 | 源mac地址                             | 源ip地址    | 目的mac地址                           | 目的ip地址  |
| ------------------------------------- | ------------------------------------- | ------ | -------- | ------------ | ----------- | ---------- | -------- | ------------------------------------- | ----------- | ------------------------------------- | ----------- |
|                                       |                                       |        |          |              |             |            |          |                                       |             |                                       |             |
| HuaweiTe_c7:73:db (54:89:98:c7:73:db) | HuaweiTe_58:37:e8 (00:e0:fc:58:37:e8) | ARP    | Ethernet | IPv4         | 6           | 4          | 2        | HuaweiTe_58:37:e8 (00:e0:fc:58:37:e8) | 192.168.0.1 | HuaweiTe_c7:73:db (54:89:98:c7:73:db) | 192.168.0.2 |

![image-20210119191143883](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210119191143883.png)

arp广播是通过网关进行传递的，本机arp表缓存的为网关的mac地址

![image-20210119191157808](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210119191157808.png)

```
$ arp -a

Internet Address    Physical Address    Type
192.168.1.1         00-E0-FC-58-37-E9   dynamic
```

op操作类型说明

| 代码 | 说明     |
| ---- | -------- |
| 1    | arp请求  |
| 2    | arp应答  |
| 3    | rarp请求 |
| 4    | rarp应答 |

### 动态ARP表项

动态ARP表项由ARP协议通过ARP报文自动生成和维护，可以被老化，可以被新的ARP报文更新，也可以被静态ARP表项所覆盖。当到达老化时间或接口关闭时会删除相应的动态ARP表项。

### 静态ARP表项

静态ARP表项通过手工配置（通过对应设备的IP地址与MAC地址绑定命定进行）和维护。不会被老化，也不会被动态ARP表项覆盖。配置静态ARP表项可以增加通信的安全性，因为静态ARP可以限定和指定IP地址的设备通信时只使用指定的MAC地址（也就是我们通常所说的IP地址和MAC地址的绑定），此时攻击报文无法修改此表项的IP地址和MAC地址的映射关系，从而保护了本设备和指定设备间正常通信。静态ARP表项又分为短静态ARP表项和长静态ARP表项

### 短静态ARP表项

在配置短静态ARP表项时，只需要配置IP地址和MAC地址项。如果出接口是三层以太网接口，短静态ARP表项可以直接用于报文转发；如果出接口是VLAN虚接口，短静态ARP表项不能直接用于报文转发，当要发送IP数据包时，先发送ARP请求报文，如果收到的相应报文中的源IP地址和源MAC地址与所配置的IP地址和MAC地址相同，则将接受ARP响应报文的接口加入该静态表项中，之后就可以用于IP数据包的转发了。

### 长静态ARP表项

在配置长静态ARP表项时，除了配置IP地址和MAC地址项外，还必须配置该ARP表所对应的VLAN（虚拟局域网）和出接口。也就是长静态ARP表项同事绑定了IP地址、MAC地址、VLAN和端口，可以直接用于报文转发。

### apr欺骗

ARP病毒，ARP欺骗

高可用服务器对之间切换时要考虑ARP缓存问题

路由器等设备无缝迁移时要考虑ARP缓存的问题，例如：更换办公室的路由器.

### ARP欺骗原理

ARP攻击就是通过伪造IP地址和MAC地址对实现ARP欺骗的，如果一台主机中了ARP病毒，那么它就能够在网络中产生大量的ARP通信量（它会以很快的频率进行广播），以至于使网络阻塞，攻击者只要持续不断的发出伪造ARP响应包就能更改局域网中目标主机ARP缓存中的IP-MAC条目，造成网络中断或中间人攻击。

ARP攻击主要是存在于局域网网络中，局域网中若有一个人感染ARP木马，则感染该ARP木马的系统将会试图通过“ARP欺骗”手段截获所在网络内其他计算机的通信故障。

服务器切换ARP问题

当网络中一台提供服务的机器宕机后，当在其他运行正常的机器添加宕机的机器的IP时，会因为客户端的ARP table cache的地址解析还是宕机的机器的MAC地址。从而导致，即使在其他运行正常的机器添加宕机的机器的IP，也会发生客户依然无法访问的情况。

解决方法是：当宕机时，IP地址迁移到其他机器上时，需要通过arping命令来通知所有网络内机器清除其本地的ARP table cache，从而使得客户机访问时重新广播获取MAC地址.

几乎所有的高可用软件都会考虑这个问题。

ARP广播而进行新的地址解析。

linux下具体命令：

```
arping -I eth0 -c 3 -s 10.0.0.162 10.0.0.253
arping -U -I eth0 10.0.0.162
```

## proxy arp

代理ARP的原理就是当出现跨网段的ARP请求时，路由器将自己的MAC返回给发送ARP广播请求发送者，实现MAC地址代理（善意的欺骗），最终使得主机能够通信。

如一个网络中设备D1 (`192.168.0.2/24`) 与设备D2 (`192.168.1.2/24`)，D1和D2在相互通信时，D1先发送了ARP广播，请求D2的mac地址，但是由于两个设备处于不同网，也就是说D1的ARP请求会被R1拦截到，然后R1会封装自己的mac地址为目的地址发送一个ARP回应数据报给R1（善意的欺骗），然后R1就会代替D1去访问D2。

如上述arp抓包图，首先广播收到回复为R1`192.168.0.1`

![image-20210119192807856](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210119192807856.png)

如果R1关闭了arp的代理功能，那么R1再访问R3的时候，R2并不会把自己的mac地址给R1，那么R1和R3之间就无法通信。默认情况下，思科的设备是开启了arp代理功能，也就是说，R2会作为中间代理实现R1和R3之间跨网段通信。

### 实验：通过命名空间模拟容器网络的代理arp数据包

实验前所需掌握的知识**接口作用域 interface scope**与**链路本地地址（Link-local address)**

#### 接口作用域

路由的接口作用域，这个配置可以解释为路由的范围会影响源数据（源地址）请求的选择。当主机存在多个网络接口和地址时，route scope控制ip数据寻址和广播的范围。

- global：如果来自不同的端口（可以理解为网卡等）可以转发。
- link：仅在此设备有效，即只有来自这个网络接口设备的流量才走这条路由（发送和接收为同一端口）

- host：本地回环，仅用于在主机内部进行通信。

- site：ipv6独有。

  reference http://linux-ip.net/html/tools-ip-address.html

#### 链路本地地址

 `169.254.0.0/16` 保留地址块，在`169.254.1.0` ~ `169.254.254.255` 中随机选择一个地址进行ARP广播，如果收到回复，表示IP地址已经使用。

在Kubernetes Calico网络中，当一个数据包的目的地址不是本网络时，会先发起ARP广播，网关设置即`169.254.1.1`收到会将自己的mac地址返回给发送端，后续的请求由这个veth对进行完成，使用代理arp做了arp欺骗。这样做抑制了arp广播攻击，并且通过代理arp也可以进行跨网络的访问。

**实验目的**：模拟calico网络，使用代理arp欺骗完成网络的跨网段通信

![image-20210120214626798](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210120214626798.png)

在准备的两个主机进行相应的设置

```bash
# 添加网络名称空间
ip netns add net1
# 创建一个虚拟以太网对
ip link add veth1 type veth peer name eth1
# 将一端关联至网络名称空间内
ip link set eth1 netns net1
# 设置一个IP地址
ip netns exec net1 ip addr add 192.168.1.10/24 dev eth1
# 启动这个网卡
ip netns exec net1 ip link set eth1 up

# 添加一个自动寻址IP，作为网关
ip netns exec net1 ip route add 169.254.1.1 dev eth1 scope link
# 所有的流量都通过这个网关进行进出
ip netns exec net1 ip route add default via 169.254.1.1 dev eth1
ip netns exec net1 ip route 
ip netns exec net1 ip link set eth1 up
# 设置主机端的网络对
ip link set veth1 up
# 所有通过192.168.1.10的数据包进出都走veth1
ip route add 192.168.1.10 dev veth1 scope link
# 通往192.168.2.10的数据的下一挑是10.0.0.3（对端主机IP）
ip route add 192.168.2.10 via 10.0.0.3 dev eth0


## 设置另外一台设备
ip netns add net1
ip link add veth1 type veth peer name eth1
ip link set eth1 netns net1

ip netns exec net1 ip addr add 192.168.2.10/24 dev eth1
ip netns exec net1 ip link set eth1 up

ip netns exec net1 ip route add 169.254.1.1 dev eth1 scope link
ip netns exec net1 ip route add default via 169.254.1.1 dev eth1
ip netns exec net1 ip route 

ip link set veth1 up
ip route add 192.168.2.10 dev veth1 scope link
ip route add 192.168.1.10 via 10.0.0.4 dev eth0
```

开启对应内核设置

```bash
# 代理arp
echo 1 &gt; /proc/sys/net/ipv4/conf/veth1/proxy_arp
# 专用VLAN代理arp。基本上允许代理arp回复到同一接口
echo 1 &gt; /proc/sys/net/ipv4/conf/veth1/proxy_arp_pvlan
# 内核转发
echo 1 &gt; /proc/sys/net/ipv4/ip_forward
```

![image-20210120215702370](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210120215702370.png)

![image-20210120215729933](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210120215729933.png)



reference

[linux proxy arp](http://linux-ip.net/html/ether-arp-proxy.html)

[169.254-ip-address](https://nova.moe/note-on-169-254-ip-addresses/)

​	
