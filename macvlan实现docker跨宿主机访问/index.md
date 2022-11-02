# macvlan实现docker跨宿主机访问

[TOC]

## 一、关于vlan说明

Macvlan和ipvlan是Linux网络驱动程序，它们将底层或主机接口直接暴露给在主机中运行的VM或容器。

Macvlan允许单个物理接口使用macvlan子接口具有多个mac和ip地址。这与使用vlan在物理接口上创建子接口不同。使用vlan子接口，每个子接口使用vlan属于不同的L2域，所有子接口都具有相同的mac地址。使用macvlan，每个子接口将获得唯一的mac和ip地址，并将直接暴露在底层网络中。Macvlan接口通常用于虚拟化应用程序，每个macvlan接口都连接到Container或VM。每个容器或VM可以直接从公共服务器获取dhcp地址，就像主机一样。这将有助于希望Container成为传统网络的客户使用他们已有的IP寻址方案。Macvlan有4种类型(Private, VEPA, Bridge, Passthru)。常用的类型是Macvlan网桥，它允许单个主机中的端点能够在没有数据包离开主机的情况下相互通信。对于外部连接，使用底层网络。下图显示了两个使用macvlan网桥相互通信以及外部世界的容器。两个容器将使用Macvlan子接口直接暴露在底层网络中。


![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/24ffba73.png)


&lt;h2 id=&#34;2&#34;&gt;二、使用mavvlan构建docker网络&lt;/h2&gt;
Macvlan，MACVLAN或MAC-VLAN允许您在单个物理接口上配置多个第2层（即以太网MAC）地址。 Macvlan允许您配置父物理以太网接口（也称为上层设备）的子接口（也称为从设备），&lt;font style=&#34;background:#ffff00;&#34; size=3&gt;每个接口都有自己唯一的（随机生成的）MAC地址，因此也有自己的IP地址&lt;/font&gt;。然后，应用程序、VM和容器可以绑定到特定的子接口，以使用自己的MAC和IP地址直接连接到物理网络。

&lt;font style=&#34;background:#ffff00;&#34; size=2&gt;Mavlan子接口不能直接与父接口通信&lt;/font&gt;，即VM不能直接与主机通信。如果需要VM主机通信，则应添加另一个macvlan子接口并将其分配给主机。

Macvlan子接口使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`eth0.20@eth0`&lt;/font&gt; 表示法来清楚地识别子接口及其父接口。子接口状态绑定到其父级状态。如果eth0关闭，则 &lt;font color=&#34;#f8070d&#34; size=3&gt;`eth0.20@eth0`&lt;/font&gt; 也会关闭。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/49435814.png)

### 2.1 配置macvlan先决条件

* 至少需要Linux内核版本3.9以上，建议使用4.0或更高版本。

### 2.2 环境准备

| 主机名 | IP地址    | 地位                 | 软件环境                 |
| ------ | --------- | -------------------- | ------------------------ |
| 物理机 | 10.0.0.1  | 物理机               | windows10                |
| 网关   | 10.0.0.2  | 宿主机网关           | vmvare网关               |
| c1     | 10.0.0.3  | 容器01               | docker                   |
| c2     | 10.0.0.4  | 容器02               | docker                   |
| node01 | 10.0.0.15 | 宿主机01（vm虚拟机） | centos 7.3/docker-ce1806 |
| node02 | 10.0.0.16 | 宿主机02（vm虚拟机） | centos 7.3/docker-ce1806 |
&lt;h3 id=&#34;2.3&#34;&gt;2.3 启动网卡混合模式&lt;/h3&gt;
两台主机网卡使用桥接模式,网卡混杂模式开启全部允许。

主机上配置的eth0网卡和创建的vlan网卡,均需要开启混杂模式。如果不开启混杂模式会导致macvlan网络无法访问外界,具体在不使用vlan时,表现为无法ping通路由,无法ping通同一网络内其他主机。

```sh
ip link set eth0 promisc on
ip link set eth0  promisc off
```
开启后查看网卡状态

```sh
[root@node01 ~]# ip addr
2: eth0: &lt;BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP&gt; mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:84:f3:29 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.15/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe84:f329/64 scope link 
       valid_lft forever preferred_lft forever
```

其中&lt;font color=&#34;#f8070d&#34; size=3&gt;`BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP`&lt;/font&gt;的&lt;font color=&#34;#f8070d&#34; size=3&gt;`PROMISC`&lt;/font&gt;说明网卡eth0已开启成混杂模式。

***
注：以上设置临时生效
***

### 2.4 基于macvlan构建docker跨宿主机通讯

```sh
docker network create \
-d macvlan \
--subnet=10.10.0.0/24 \
--gateway=10.10.0.254 \
-o parent=eth0 mvl1
```

***
&lt;font color=&#34;#0215cd&#34; size=3&gt; 说明：容器默认使用主机的DNS设置，因此无需配置DNS服务器。&lt;/font&gt;
***

查看创建结果

```sh
[root@node01 ~]# docker network ls 
NETWORK ID          NAME                DRIVER              SCOPE
3d2449dfe4b1        bridge              bridge              local
7110f9183457        host                host                local
9852fc2a7109        mvl1                macvlan             local
```

在node01上运行容器

```sh
docker run -tid --name c1 --net mvl1 --ip 10.10.0.1 busybox
```

在node02上运行容器

```sh
docker run -tid --name c2 --net mvl1 --ip 10.10.0.2 busybox
```

在C1上平C2 检查结果

```sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0A:0A:00:01  
          inet addr:10.10.0.1  Bcast:10.10.0.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # ping 10.10.0.2
PING 10.10.0.2 (10.10.0.2): 56 data bytes
64 bytes from 10.10.0.2: seq=0 ttl=64 time=0.397 ms
64 bytes from 10.10.0.2: seq=1 ttl=64 time=0.278 ms
```

&lt;h3 id=&#34;2.5&#34;&gt;2.5 构建macvlan与宿主机同网段docker网络&lt;/h3&gt;
&gt; **在两台主机上分别创建docker网络**

```sh
docker network create -d macvlan --subnet=10.0.0.0/24 --gateway=10.0.0.2 -o parent=eth0 mvl1
```

说明：
* &lt;font color=&#34;#f8070d&#34; size=3&gt;`--gateway`&lt;/font&gt;为宿主机的网关，如宿主机为物理机则设置路由器的ip。
* &lt;font color=&#34;#f8070d&#34; size=3&gt;`--subnet`&lt;/font&gt;为宿主机所在网段。

&gt; **在两台主机上分别创建容器**
```sh
docker run -ti --net mvl1 --ip 10.0.0.4 busybox
docker run -ti --net mvl1 --ip 10.0.0.3 busybox
```

&gt; **测试网络连通情况**

ping网关，结论：通。

```sh
/ # ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2): 56 data bytes
64 bytes from 10.0.0.2: seq=0 ttl=128 time=0.330 ms
```
ping宿主机，结论：不通。
```sh
/ # ping 10.0.0.15
PING 10.0.0.15 (10.0.0.15): 56 data bytes
```
ping其他宿主机，结论：通。
```sh
/ # ping 10.0.0.16
PING 10.0.0.16 (10.0.0.16): 56 data bytes
64 bytes from 10.0.0.16: seq=0 ttl=64 time=0.530 ms
```
ping其他容器，结论：通。
```sh
/ # ping 10.0.0.3
PING 10.0.0.3 (10.0.0.3): 56 data bytes
64 bytes from 10.0.0.3: seq=0 ttl=64 time=0.435 ms
```

## 三、带有VLAN的macvlan


### 3.1 说明

单个Docker主机网络接口只能作为一个macvlan或ipvlan网络的父接口。然而，一个macvlan，一个第2层域和每个物理接口一个子网是现代虚拟化解决方案中相当严重的限制。幸运的是，Docker主机子接口可以作为macvlan网络的父接口。这与VLAN的Linux实现完全一致，其中802.1Q中继连接上的每个VLAN都在物理接口的子接口上。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/36a2cb7d.png)

### 3.2 vlan介绍

VLAN(Virtual Local Area Network)又称虚拟局域网，是指在局域网的基础上，采用网络管理软件构建的可跨越不同网段、不同网络的端到端的逻辑网络。

一个VLAN组成一个逻辑子网，即一个逻辑广播域，它可以覆盖多个网络设备，允许处于不同地理位置的网络用户加入到一个逻辑子网中。使用VLAN功能后，能够将网络分割成多个广播域。

Linux支持在物理网卡上创建vlan子接口。每个vlan子接口属于不同的二层域，所有的vlan子接口拥有相同的MAC地址。这点是和Macvlan子接口不同的地方。

&gt; **vlan范围说明**

|范围|说明|
|-|-|
|0，4095| 保留 仅限系统使用 用户不能查看和使用这些VLAN|
|1 正常 |Cisco默认VLAN 用户能够使用该VLAN，但不能删除它|
|2-1001 |正常 用于以太网的VLAN 用户可以创建、使用和删除这些VLAN|
|1002-1005| 正常 用于FDDI和令牌环的Cisco默认VLAN 用户不能删除这些VLAN|
|1006-1024 |保留 仅限系统使用 用户不能查看和使用这些VLAN|
|1025-4094 |扩展 仅用于以太网VLAN|

### 3.3 环境准备

|主机名|IP地址|地位|软件环境|
|-|-|-|-|
|c1|10.10.0.1|容器01-02|docker|
|c2|10.10.0.2|容器01-02|docker|
|c3|10.10.0.3|容器02-01|docker|
|c4|10.10.0.4|容器02-02|docker|
|gateway01|10.0.0.253|容器01网关| |
|gateway01|10.0.0.254|容器01网关| |
|node01|10.0.0.15|宿主机01（vm虚拟机）|centos 7.3/docker-ce1806|
|node02|10.0.0.16|宿主机02（vm虚拟机）|centos 7.3/docker-ce1806|

### 3.4 创建VLAN

&gt; **为node01物理网卡创建macvlan子接口**



```sh
ip link add link eth0 name eth0.100 type vlan id 100
ip link add link eth0 name eth0.200 type vlan id 200
```

&gt; **启用macvlan**

```sh
ip link set eth0.100 up
ip link set eth0.200 up
```

&gt; **设置macvlan的ip和网关**

```
ip addr add 10.10.0.254/24 dev eth0.100
ip addr add 10.20.0.254/24 dev eth0.200

ip route add default via 10.10.0.254 dev eth0.100
ip route add default via 10.20.0.254 dev eth0.200
```

参考网址：

[Exploring Docker Networking – Host, None, and MACVLAN | raid-zero.com | Page 3](https://raid-zero.com/2017/08/02/exploring-docker-networking-host-none-and-macvlan/3/)

[Docker Networking: macvlans with VLANs – HiCube](http://hicu.be/docker-networking-macvlan-vlan-configuration)


