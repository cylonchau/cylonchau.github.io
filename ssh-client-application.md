# SSH客户端应用场景


## 1 SSH命令详解

|                    参数                    | 说明                                                         |
| :----------------------------------------: | ------------------------------------------------------------ |
|                  -1&nbsp;                  | 强制使用ssh协议版本1；                                       |
|                     -2                     | 强制使用ssh协议版本2；                                       |
|                     -4                     | 强制使用IPv4地址；                                           |
|                     -6                     | 强制使用IPv6地址；                                           |
|                     -A                     | 开启认证代理连接转发功能；                                   |
|                     -a                     | 关闭认证代理连接转发功能；                                   |
|                     -b                     | 使用本机指定地址作为对应连接的源ip地址；                     |
| **<font color="#f8070d" size=3>-C</font>** | 请求压缩所有数据；                                           |
|                     -F                     | 指定ssh指令的配置文件；                                      |
| **<font color="#f8070d" size=3>-f</font>** | 后台执行ssh指令；                                            |
|                     -g                     | 允许远程主机连接主机的转发端口；                             |
|                     -i                     | 指定身份文件；                                               |
|                     -l                     | 指定连接远程服务器登录用户名；                               |
| **<font color="#f8070d" size=3>-N</font>** | 不执行远程指令；                                             |
|                     -o                     | 指定配置选项；                                               |
|                     -p                     | 指定远程服务器上的端口；                                     |
|                     -q                     | 静默模式；                                                   |
|                     -X                     | 开启X11转发功能；                                            |
|                     -x                     | 关闭X11转发功能；                                            |
|                     -y                     | 开启信任X11转发功能。                                        |
| **<font color="#f8070d" size=3>-D</font>** | 指定本地 “动态” 应用程序级端口转发。这通过分配套接字来侦听本地端口，可选地绑定到指定的bind_address。只要与此端口建立连接，就会通过安全通道转发连接，然后使用应用程序协议确定从远程计算机连接的位置。 |

## 2 SSH端口转发实战

### 2.1 概述

在通常情况下，网络防火墙会阻碍你进行某些必要的网络传输。公司的安全策略可能不允许你把SSH密钥存储在某些主机上。或者，你也可能需要在原本安全的环境中运行一些不安全的网络应用程序。

SSH提供了一个重要功能，称为转发<font color="#f8070d" size=3>`forwarding`</font>或者称为隧道传输<font color="#f8070d" size=3>`tunneling`</font>，它能够将其他 TCP 端口的网络数据通过 SSH 链接来转发，并且自动提供了相应的加密及解密服务。这一过程有时也被叫做<font color="#f8070d" size=3>`tunneling`</font>，这是因为 SSH 为其他 TCP 链接提供了一个安全的通道来进行传输而得名。

**SSH端口转发主要用来解决如下两方面问题：**
1. 突破防火墙的限制完成无法建立的 TCP 连接。
2. 加密 Client 端至 Server 端之间的通讯数据。

### 2.2 开启ssh的端口转发功能

ssh端口转发功能默认是打开的。如需修改的话，修改后需要重启sshd服务才会生效。
```bash
AllowTcpForwarding yes
```
### 2.3 常用SSH转发类型

在ssh连接的基础上，指定`ssh client`或`ssh server`的某个端口作为源地址，所有发至该端口的数据包都会透过ssh连接被转发出去；至于转发的目标地址，目标地址既可以指定，也可以不指定，如果指定了目标地址，称为定向转发，如果不指定目标地址则称为动态转发：

**定向转发**

定向转发把数据包转发到指定的目标地址。目标地址不限定是ssh client 或 ssh server，既可以是二者之一，也可以是二者以外的其他机器。

**动态转发**

动态转发不指定目标地址，数据包转发的目的地是动态决定的。

#### 2.3.1 本地端口转发

本地转发中的本地是指将本地的某个端口(1024-65535)通过SSH隧道转发至其他主机的套接字，这样当我们的程序连接本地的这个端口时，其实间接连上了其他主机的某个端口，当我们发数据包到这个端口时数据包就自动转发到了那个远程端口上了

![image-20200730182255288](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730182255288.png)

> **语法**

```sh
ssh -L [bind_address:]port:host:port ssh_host
```

```sh
ssh -fNC -L 3333:100.109.9.249:22 111.44.246.139
```

> **命令说明**

| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| bind_address | 表示客户端主机的ip，这是针对系统有多块网卡，不指定默认是127.0.0.1 |
| port         | 本地主机指定监听的端口                                       |
| host         | 远程主机的ip                                                 |
| hostport     | 指定远程主机的端口，如果远程主机是HTTP，就是80，FTP（21）。  |
| ssh_host     | 远程主机的ip，也可以是能够访问到远程主机的另一个ip           |

<font color="#f8070d" size=3>`-L`</font>表明是本地转发，此时<font color="#f8070d" size=2>`TCP客户端`</font>与<font color="#f8070d" size=2>`SSH客户端`</font>同在本地主机上。后面接着三个值，由冒号分开，分别表示：需要监听的本地端口，远程主机名或IP地址，及远程的转发目标端口号。



#### 2.3.2 远程端口转发
远程转发和本地很相似，原理也差不多，但是不同的是，本地转发是在本地主机指定的一个端口，而远程转发是由SSH服务器经由SSH客户端转发，连接至目标服务器上。本质一样，区别在于需要转发的端口是在远程主机上还是在本地主机上

现在SSH就可以把连接从（39.104.112.253:80）转发到（10.0.0.10:85）。

![image-20200730182352895](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730182352895.png)

> **语法**

```sh
ssh -R [bind_ip]:bind_port:host_ip:host_port sshserver
```

> **参数说明**

| 参数      | 说明                             |
| --------- | -------------------------------- |
| -R        | 表示远程转发。                   |
| bind_ip   | 表示绑定的远程主机iP             |
| bind_port | 远程主机监听端口                 |
| host_ip   | 被访问的主机IP                   |
| host_port | 被访问主机的端口                 |
| sshserver | 使用sshclient建立隧道的sshserver |

#### 2.3.3 动态端口转发

定向转发（包括本地转发和远程转发）的局限性是必须指定某个目标地址，如果需要借助一台中间服务器访问很多目标地址，一个一个地定向转发显然不是好办法，这时就要用的是ssh动态端口转发，它相当于建立一个SOCKS服务器。各种应用经由SSH客户端转发，经过SSH服务器，到达目标服务器，不固定端口。

> **语法**

```sh
ssh -fN -D [ssh client port] ssh server
```
> **参数说明**

| 参数            | 说明                           |
| --------------- | ------------------------------ |
| -D              | 动态转发                       |
| -f              | 后台执行                       |
| -N              | 不执行任何命令                 |
| ssh client port | 监听的端口                     |
| ssh server      | 当作sock5代理服务器(127.0.0.1) |

## 3 SSH端口转发实战

### 3.1 实验1：本地端口转发应用

#### 实验环境

| 主机   | 公网IP         | 内网IP        | 地位         |
| ------ | -------------- | ------------- | ------------ |
| node01 | 39.104.116.253 | 172.24.104.10 | client       |
| node02 | 112.35.76.212  | 172.16.16.18  | mysql server |

实现在生产内网里有一台mysql服务器（mysql Server），但是限制了只有本机上部署的应用才能直接连接此 mysql服务器。现在我们想临时从本地机器（mysql Client）直接连接到这个mysql服务器，只需要在mysql Client上运用本地端口转发。

#### 查看node01系统状态

```sh
$ ps -ef|grep mysql
root     21764 21724  0 23:15 pts/0    00:00:00 grep --color=auto mysql

$ ss -lnt
State      Recv-Q    Send-Q           Local Address:Port        Peer Address:Port              
LISTEN     0         128              *:22                      *:*    
```

查看node02系统状态

```sh
$ ps -ef|grep mysql
mysql    2042     1   0 Aug30 ?      /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
root     18250 18194  0 23:15 pts/0  grep --color=auto mysql
```

在主机A上执行命令：**<font color="#f8070d" size=3>`ssh -Nf -L *:7000:127.0.0.1:3306 112.35.76.212`</font>**

需要注意的是，在选择端口号时要注意非管理员帐号是无权绑定 1-1023 端口的，所以一般是选用一个 1024-65535 之间的并且尚未使用的端口号即可。

```sh
[root@iZ8o31djppqzqeZ ~]# ss -lnt
State      Recv-Q Send-Q          Local Address:Port          Peer Address:Port              
LISTEN     0      128              *:22                        *:*                  
LISTEN     0      128              *:7000                      *:*                  
LISTEN     0      128             :::7000                     :::*                  
```

可以发现，node01的7000端口已被监听了，是被ssh（ssh就是客服端）监听的，和node02建立了一个SSH隧道，并且我们知道node01和node02是可以通信的，我们可以通过主机A的端口访问时，其实就是间接用主机B在访问。可以看到可以正常登陆mysql。

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/babde631.gif)

***
<font color="#0215cd" size=2>注：mysql经过转发实际登陆mysql的用户组成的ip为sshserver登陆的ip。而不是sshclient的ip</font>

***

### 3.2 实验2：远程端口转发应用

内网通过路由器（SNAT）可以访问外网，而外网是无法访问到内网的，我们利用ssh来实现访问内网，要搭建这种环境，首先想到的是VMware的NAT模式，我将node03设置NAT模式。

| 主机   | 公网IP        | 内网IP        | 地位      |
| ------ | ------------- | ------------- | --------- |
| node01 | 112.35.26.104 | 172.24.104.10 | sshserver |
| node02 | 112.35.76.212 | 172.16.16.18  | sshclient |
| node03 |               | 172.16.16.11  | mysql     |
| node04 |               |               | 客户端    |

在node02执行命令：`ssh -Nf -R *:7706:172.16.16.11:3306 112.35.26.104`，sshclient与sshserver建立隧道连接。

```sh
[root@jekins ~]# netstat -nt|grep 104
tcp        0      0 172.16.16.18:50676      112.35.26.104:22        ESTABLISHED
```
sshserver上可见到监听的端口。
```sh
root@dmz01 ~]# ss -lnt|grep 7706
LISTEN     0      128         127.0.0.1:7706              *:*     
LISTEN     0      128               ::1:7706             :::*     
```

登陆环境进行测试

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/13acf536.gif)



翻墙命令 `ssh -TfnN -D {Port} {User}@{Host}`

```
ssh -TfnN -D 7070 root@195.133.11.43
```

