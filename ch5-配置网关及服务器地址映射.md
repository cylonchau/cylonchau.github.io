# ch5 配置网关及服务器地址映射


### 1 办公室路由网关架构图

![image-20221025002730248](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025002730248.png)

对应实际企业办公上网场景逻辑图

![image-20221025002851351](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025002851351.png)

### 2.实验环境配置需求前期准备

####  2.1 服务器网关B需要准备如下条件

>> 1. 物理条件是具备上网卡，建议eth0外网地址（这里是192.168.1.5,gw 192.168.1.2），ech1内网地址（这里是172.168.1.10，<font style="background:#bafe01;" size=2>内网卡不配GW</font >。
>> 2.  确保服务器网关B要可以上网（B上网才能代理别的机器上网）。可以通过ping baidu.com或外网IP测试。
>> 3.  内核文件/etc/sysctl.conf里开启转发功能。在服务器网关B192.168.1.5机器上开启路由转发功能。编辑/etc/sysctl.conf修改内容为net.ipv4.ip_forward = 1，然后执行sysctl -p使修改生效
>> 4.  iptables的filter表的FORWARD链允许转发
>> 5.  不要filter防火墙功能，共享上网，因此最好暂停防火墙测试/etc/init.d/tables stop

#### 2.2 加载iptables内核模块

配置网关需要iptables的nat表，PREROUTING，POSTROUTING。

***(1)载入iptables内核模块，执行并放入rc.local***

```bash
modprobe ip_tables \
modprobe iptable_filter \
modprobe iptable_nat \
modprobe ip_conntrack \
modprobe ip_conntrack_ftp \
modprobe ip_nat_ftp \
modprobe ipt_state
```

```bash
[root@Lnmp_01 ~]# lsmod|egrep ^ip 
iptable_nat             6051  0 
iptable_filter          2793  0 
```
#### 2.3 局域网的机器：

>> 1.  局域网的机器有一块网卡即可，确保局域网的机器C，默认网关这只了网关服务器B的eth1内网卡IP（172.168l.1.10）。把主机C的gateway设置为B的内网卡192的网卡ip即172.168l.1.10。
>> 2. 检查手段：
>> 3. 分别ping网关服务器B的内外网卡IP，都应该是通的就对了.
>> 4. 出公网检查除了PING网站域名外，也要ping下外网ip，排除DNS故障。不通
>> 5. ping 10.0.0.254网关也是不通的。

如上，请准备两台虚拟机B和C，其中B要有双网卡。B的内网卡的网段和C的网段一样。

***网关B：假设192.168.1.0/24为外部IP，172.168.1.0/24为内部IP***

eth0:192.168.1.5   IPADDR=192.168.1.5 

gw:192.168.1.2	GATEWAY=192.168.1.2

eth1 	eth1:172.168.1.10 

<font style="background:#bafe01;" size=2>gw：不配</font>

内部服务器C：

eth0：172.168.1.11 	IPADDR=172.168.1.11 

gw：172.168.1.10（网关B的内网卡IP）		GATEWAY=172.168.1.10

准备结果：

**B网关服务器配置**

```bash
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:31:E5:AF  
          inet addr:192.168.1.4  Bcast:192.168.1.255  Mask:255.255.255.0
 		......
eth1      Link encap:Ethernet  HWaddr 00:0C:29:31:E5:B9  
          inet addr:172.168.1.10  Bcast:172.168.1.255  Mask:255.255.255.0
# 路由
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
172.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
0.0.0.0         192.168.1.2     0.0.0.0         UG    0      0        0 eth0
```

**C为内网PC或者服务器**

```bash
[root@Lnmp-01 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:BE:2D:75  
          inet addr:172.168.1.11  Bcast:172.168.1.255  Mask:255.255.255.0
....
[root@Lnmp-01 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
0.0.0.0         172.168.1.10    0.0.0.0         UG    0      0        0 eth0
```

### 3 先做一些测试记录

1.登陆C主机172.168.1.11看是否能访问外部页面（配好DNS）。如ping www.baidu.com。正确结果：当前情况不通 lamp为10 lnmp为11

```bash
[root@Lnmp-01 ~]# ping www.baidu.com
^C
[root@Lnmp-01 ~]# ping 61.105.221.1
PING 61.105.221.1 (61.105.221.1) 56(84) bytes of data.
```

&nbsp;&nbsp;&nbsp;2.在笔记本上分别测试telnet 172.168.1.10 22看是否能连通、结果:当前情况通。

```bash
$ telnet 172.168.1.10 80
Trying 172.168.1.10...
Connected to 172.168.1.10.
Escape character is '^]'.	
[root@Lnmp-01 ~]#  telnet 172.168.1.10 80
Trying 172.168.1.10...
Connected to 172.168.1.10.
Escape character is '^]'
```

3. 在笔记本上分别测试ping 172.168.1.11看是否能连通。结果当前情况通。
4. 测试登陆172.168.1.10看是否能访问外部页面。如ping www.baidu.com结果当前情况通。
5. 在笔记本上分别测试telnet C主机 172.168.1.11 22 结果当前情况不通。
6. 在笔记本上分别测试ping 172.168.1.11看是否能连通。结果当前情况不通。

### 4 根据逻辑图实现如下要求

#### 4.1 局域网共享上网项目案例

1.实现c可经过b，通过A上因特网。
解答：提示2:注意主机防火墙功能的影响，可以尝试在GW上先/etc/init.d/iptables stop后在加命令

2.实际处理的局域网共享上网NAT命令
局域网共享的两条命令方法：

***方法1：适合于有固定ip外网地址的：***

```bash
iptables -t nat -A POSTROUTING -s 172.168.1.0/24 -o eth0 -j SNAT --to-source 192.168.1.4
```

> 1.  -s 172.168.1.0/24办公室或IDC、内网网段。
> 2.  -o eth0 为网关的外网卡接口。	
> 3.  -j SNAT --to-source 192.168.1.4是网关外网卡IP地址。

**方法2：适合变化外网地址（ADSL）**

```bash
iptables -t nat -A POSTROUTING -s 172.168.1.0/24 -j MASQUERADE
```
测试结果

```bash
[root@Lnmp-01 ~]# ping www.baidu.com
PING www.a.shifen.com (58.217.200.112) 56(84) bytes of data.
64 bytes from 58.217.200.112: icmp_seq=1 ttl=127 time=32.1 ms

$ iptables -t nat -nL
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
SNAT       all  --  172.168.1.0/24       0.0.0.0/0           to:192.168.1.4 
```

**为什么要用POSTROUTING？**

企业共享上网：

1.办公网共享上网（网关要有外网IP，否则用路由（zebra））

2.IDC内网机器上网

***企业上网到底需要不需要linux网关？***

解答：
1. 如果企业里有企业级路由器的情况下，可以不需要上网网关。使用网关只是解决路由器无法解决的需求（例如：上网行为，IP及端口的映射，网关杀毒）。
2. IDC机房，大厦有固定IP的宽带，直接用网关解决上网及控制问题。

#### 4.2 把外部IP地址及端口映射到内部服务器地址及端口（和贡献上网环境一样）


在10段主机可以通过访问192.168.1.4:80，即可访问到192.168.1.8:9000 提供的web服务。也可SSH（192.168.1.4:222 --> 192.168.1.8:52113）<== PREROUTING

**C配置WEB服务器**

解答：

⑴ 在172.168.1.10开启http服务监听9000端口，然后在网关服务器B可以访问

⑵ 具体转换命令：

```bash
iptables -t nat -A PREROUTING -d 192.168.1.5 -p tcp --dport 9000 -j DNAT --to-destination 172.168.1.11:80
# DNAT：目的地址转换，将将本地内部的地址映射到互联网地址
```

**测试结果**

*清空NAT表的规则。*

```bash
iptables -t nat -F
```

这个时候访问83的9000端口是不能访问的

```bash
iptables -t nat -A PREROUTING -d 192.168.2.83 -p tcp --dport 9000 -j DNAT --to-destination 172.168.1.11:80
```

这里看到访问192.168.1.5的9000端口就会映射到内网172.168.1.11的80上。

#### ssh转发实验

```bash
# 网关A IP 192.168.2.83 内网IP 172.168.1.10
[root@Lnmp_01 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:AF:21:4F  
          inet addr:192.168.2.83  Bcast:192.168.2.255  Mask:255.255.255.0
eth1      Link encap:Ethernet  HWaddr 00:50:56:20:37:C2  
          inet addr:172.168.1.10  Bcast:172.168.1.255  Mask:255.255.255.0
# 在网关上设置转发
iptables -t nat -I PREROUTING  -p tcp -m tcp --dport 9020 -j DNAT --to-destination 172.168.1.11:52113
# 用外网访问网关外网IP
[root@MariaDB-Test ~]# ssh -p 9020 192.168.2.83
.....
root@192.168.2.83's password: 
Last login: Mon Dec 12 22:24:58 2016 from 192.168.2.84
[root@Lamp_01 ~]# ifconfig
          inet addr:172.168.1.11  Bcast:172.168.1.255  Mask:255.255.255.0
```

强调：有个网友说网关服务需要开启80服务，但不需要对外服务？

测试结果：网关开启httpd:80后。

此时，来自80端口的请求转发依然会转发到后端的服务器。但是iptables nat规则删除后，此时就到达了http服务的80端口所以显示的是默认页面。

 **企业应用场景：**

1.   把访问外网IP及读研口的请求映射到内网某个服务器及端口（企业内部）。
2.   硬件防火墙，把访问LVS/nginx外网VIP及80端口的请求映射到IDC负载均衡器内部IP及端口上（IDC机房的操作）

**iptables企业常用案例：**
1.  lnux主机防火墙（表FILTER INPUT链）
2.  局域网机器共享上网（表：NAT POSTROUTING链）

```bash
iptables -t nat -A POSTROUTING -s 172.168.1.0/24 -o eth0 -j SNAT --to-source 192.168.1.5
```

3.  外部地址和端口，映射为内部地址和端口（表：NAT PREROUTING）

```bash
iptables -t nat -A PREROUTING -d 192.168.1.5 -p tcp --dport 80 -j DNAT --to-destination 172.168.1.11:9000
```

#### 4.3 实现192段外网IP和172段内网IP一对一映射

网关IP：eth0:192.168.1.5 ech1:172.168.1.10

首先在路由网关上绑定接口外网ip，可以是别名的方式。

```bash
# 访问外网IP就映射到0.8
-A PREROUTING -d 124.42.34.112 -j DNAT --to-destination 10.0.0.8
# 出网时候改回去
-A POSTROUTING -s 10.0.0.8 -j SNAT --to-destination 124.42.34.112
# 当局域网使用外网IP访问这台机器，会出现问题，只要是局域网访问这个地址，冲定向到网关，防止可能环路
-A POSTROUTING -s 10.0.0.0/255.255.240.0 -d 124.42.34.112 -j SNAT --to-source 10.0.0.254
```
#### 4.4 实现192段机器和10段机器互相访问

http://v.youku.com/v_show/id_XNTAyMjAwMzI0.html

