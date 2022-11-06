# 使用weave实现docker跨宿主机通讯


[toc]

项目地址：https://github.com/weaveworks/weave



## 一、weaves说明

Weave是由weaveworks公司开发的解决Docker跨主机网络的解决方案，它能够创建一个虚拟网络，用于连接部署在多台主机上的Docker容器，这样容器就像被接入了同一个网络交换机，那些使用网络的应用程序不必去配置端口映射和链接等信息。

外部设备能够访问Weave网络上的应用程序容器所提供的服务，同时已有的内部系统也能够暴露到应用程序容器上。Weave能够穿透防火墙并运行在部分连接的网络上，另外，Weave的通信支持加密，所以用户可以从一个不受信任的网络连接到主机。

### 1.1 weaves实现原理


`weave launch`初始化时会自动下载三个docker容器来辅助运行，并且创建linux网桥与docker网络

weave 运行了三个容器：
* weave 是主程序，负责建立`weave`网络，收发数据，提供 DNS 服务等。
* weavevolumes容器提供卷存储
* weavedb容器提供数据存储
```sh
$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
weaveworks/weavedb     latest              15c78a9b1895        4 weeks ago         698B
weaveworks/weaveexec   2.4.0               bf0c403ea58d        4 weeks ago         151MB
weaveworks/weave       2.4.0               7aa67bc6bc43        4 weeks ago         96.7MB
   
```
自动创建网桥

```sh
$ brctl show
bridge name	      bridge id		       STP enabled	interfaces
docker0		        8000.02426cf29450	 no		
docker_gwbridge		8000.02420cb2e439	 no	 
weave		          8000.a2ec14f583ef	 no	 vethwe-bridge
```

* datapath：是一个openvswitch
* vethwe-datapath@vethwe-bridge：是veth pair
* vethwe-datapath：父设备是datapath
* vxlan-6784：是vxlan interface，其maste也是datapath，weave主机之间通过Vxlan节能型通信

```sh
$ ifconfig
datapath: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1376
        inet6 fe80::e45d:12ff:fee2:9d69  prefixlen 64  scopeid 0x20&lt;link&gt;
        ether e6:5d:12:e2:9d:69  txqueuelen 1000  (Ethernet)
        RX packets 19  bytes 1060 (1.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099&lt;UP,BROADCAST,MULTICAST&gt;  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:24:0d:54:06  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker_gwbridge: flags=4099&lt;UP,BROADCAST,MULTICAST&gt;  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        inet6 fe80::42:52ff:fe25:3b18  prefixlen 64  scopeid 0x20&lt;link&gt;
        ether 02:42:52:25:3b:18  txqueuelen 0  (Ethernet)
        RX packets 1032  bytes 89148 (87.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1032  bytes 89148 (87.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4419&lt;UP,BROADCAST,RUNNING,PROMISC,MULTICAST&gt;  mtu 1500
        inet 10.0.0.15  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe80::20c:29ff:fe84:f329  prefixlen 64  scopeid 0x20&lt;link&gt;
        ether 00:0c:29:84:f3:29  txqueuelen 1000  (Ethernet)
        RX packets 97077  bytes 109615069 (104.5 MiB)
        RX errors 0  dropped 244  overruns 0  frame 0
        TX packets 21805  bytes 3174138 (3.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73&lt;UP,LOOPBACK,RUNNING&gt;  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10&lt;host&gt;
        loop  txqueuelen 1  (Local Loopback)
        RX packets 1032  bytes 89148 (87.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1032  bytes 89148 (87.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethwe-bridge: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1376
        inet6 fe80::f056:b7ff:fe0f:c146  prefixlen 64  scopeid 0x20&lt;link&gt;
        ether f2:56:b7:0f:c1:46  txqueuelen 0  (Ethernet)
        RX packets 272  bytes 25496 (24.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 275  bytes 25670 (25.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethwe-datapath: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1376
        inet6 fe80::c495:98ff:fec0:508d  prefixlen 64  scopeid 0x20&lt;link&gt;
        ether c6:95:98:c0:50:8d  txqueuelen 0  (Ethernet)
        RX packets 1032  bytes 89148 (87.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1032  bytes 89148 (87.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vxlan-6784: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 65470
        ether 7a:a1:d9:e9:f7:39  txqueuelen 1000  (Ethernet)
        RX packets 513  bytes 372948 (364.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 520  bytes 379884 (370.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

weave: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1376
        inet6 fe80::469:deff:fe6b:f186  prefixlen 64  scopeid 0x20&lt;link&gt;
        ether 06:69:de:6b:f1:86  txqueuelen 1000  (Ethernet)
        RX packets 19  bytes 1060 (1.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```sh
$  docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
0ca046b6232c        bridge              bridge              local
776a38c5868e        docker_gwbridge     bridge              local
51bfcaafee94        weave               weavemesh           local

```
自动创建docker网络`weave`
```sh
$ brctl show
bridge name	      bridge id		        STP enabled	  interfaces
docker0		        8000.0242240d5406	  no		
docker_gwbridge		8000.024252253b18	  no		         vethcb0a2e3
weave		          8000.0669de6bf186	  no		         vethwe-bridge
							                                       vethwl95e206ea7
```
查看`weave`网络的信息dirver为`&#34;Driver&#34;: &#34;weavemesh&#34;`

```sh
$ docker network inspect weave
[
    {
        &#34;Name&#34;: &#34;weave&#34;,
        &#34;Id&#34;: &#34;522dd1c8152750aa5862bdcc3c025bb07b9d66410f267503ae9c4305363d5a82&#34;,
        &#34;Created&#34;: &#34;2018-08-27T17:27:37.265691267&#43;08:00&#34;,
        &#34;Scope&#34;: &#34;local&#34;,
        &#34;Driver&#34;: &#34;weavemesh&#34;,
        &#34;EnableIPv6&#34;: false,
        &#34;IPAM&#34;: {
            &#34;Driver&#34;: &#34;weavemesh&#34;,
            &#34;Options&#34;: null,
            &#34;Config&#34;: [
                {
                    &#34;Subnet&#34;: &#34;10.32.0.0/12&#34;
                }
            ]
        },
        &#34;Internal&#34;: false,
        &#34;Attachable&#34;: false,
        &#34;Ingress&#34;: false,
        &#34;ConfigFrom&#34;: {
            &#34;Network&#34;: &#34;&#34;
        },
        &#34;ConfigOnly&#34;: false,
        &#34;Containers&#34;: {},
        &#34;Options&#34;: {
            &#34;works.weave.multicast&#34;: &#34;true&#34;
        },
        &#34;Labels&#34;: {}
    }
]
```
Weave网络会在每个宿主机上创建一个网桥，每个容器通过veth pair连接到这个Weave 网桥。容器里面的veth网卡会获取到Weave网络分配给的IP地址和子网掩码。每当容器启动时，会创建两个网络接口。`eth0if51` 与`docker_gwbridge` 同属于一个网段。

```sh
1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
48: ethwe0@if49: &lt;BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN&gt; mtu 1376 qdisc noqueue 
    link/ether 3e:78:8b:2e:c9:4b brd ff:ff:ff:ff:ff:ff
    inet 10.40.0.0/12 brd 10.47.255.255 scope global ethwe0
       valid_lft forever preferred_lft forever
50: eth0@if51: &lt;BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN&gt; mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
其中`ethwe0@if49`，从名称上看出与weave相关，其对应的编号是48。我们从宿主机上面ip link进行查看，`ethwe0@if49`与`vethwle9c9e24ce@if48`是一对veth pair，而且被挂在了weave网桥上
```sh
49: vethwle9c9e24ce@if48: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1376 qdisc noqueue master weave state UP 
    link/ether 1a:c5:52:37:66:72 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::18c5:52ff:fe37:6672/64 scope link 
       valid_lft forever preferred_lft forever
51: veth9c86c85@if50: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc noqueue master docker_gwbridge state UP 
    link/ether da:57:cc:0c:7d:32 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::d857:ccff:fe0c:7d32/64 scope link 
       valid_lft forever preferred_lft forever
```

```sh
weave		8000.a2ec14f583ef	no		vethwe-bridge
							                  vethwle9c9e24ce

```



## 二、weave安装配置

项目地址：https://github.com/weaveworks/weave

### 2.1 环境准备

环境要求：
* linux内核版本为3.8以上
* dockers版本为1.10.0或更高

| 主机名 | IP地址    | 软件环境          |
| ------ | --------- | ----------------- |
| node01 | 10.0.0.15 | docker-1806 weare |
| node02 | 10.0.0.16 | docker-1806 weare |

### 2.2 下载安装weave


Weave不需要集中式的key-value存储，所以安装和运行都很简单。直接把Weave二进制文件下载到系统中就可以了。主从节点都需要安装。

```sh
wget -O /usr/local/bin/weave \
https://github.com/weaveworks/weave/releases/download/v2.4.0/weave &amp;&amp; \
chmod &#43;x /usr/local/bin/weave
```

[Docker网络 Weave - Bigberg - 博客园](https://www.cnblogs.com/bigberg/p/8694971.html)
