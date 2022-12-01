# docker跨宿主机网络通信


[toc]

## Docker Overlay Network

&nbsp;&nbsp;&nbsp;&nbsp;Overlay网络是指在不改变现有网络基础设施的前提下，通过某种约定通信协议，把二层报文封装在IP报文之上的新的数据格式。这样不但能够充分利用成熟的IP路由协议进程数据分发；而且在Overlay技术中采用扩展的隔离标识位数，能够突破VLAN的4000数量限制支持高达16M的用户，并在必要时可将广播流量转化为组播流量，避免广播数据泛滥。

&nbsp;&nbsp;&nbsp;&nbsp;因此，Overlay网络实际上是目前最主流的容器跨节点数据传输和路由方案。

**要想使用Docker原生Overlay网络，需要满足下列任意条件**

* **Docker 运行在Swarm**
* **使用键值存储的Docker主机集群**

## 使用键值存储搭建Docker主机集群

使用键值存储的Docker主机集群，需满足下列条件：

* 集群中主机连接到键值存储，Docker支持 Consul、Etcd和Zookeeper
* 集群中主机运行一个Docker守护进程
* 集群中主机必须具有唯一的主机名，因为键值存储使用主机名来标识集群成员
* 集群中linux主机内核版本在3.12+,支持VXLAN数据包处理，否则可能无法通行

## 部署docker内置的OverLAY网络

### 环境准备说明


| host   | ip-       |
| ------ | --------- |
| node01 | 10.0.0.15 |
| node02 | 10.0.0.16 |
### 安装Consul

下载地址：[Download Consul](https://www.consul.io/downloads.html)

启动命令

```bash
consul agent -server -bootstrap -ui -data-dir /data/docker/consul \
-client=10.0.0.16 -bind=10.0.0.16 


docker run -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h consul progrium/consul -server -bootstrap -ui-dir /ui
 
#-ui : consul 的管理界面
#-data-dir : 数据存储
```

### 配置docker链接consul

```bash
ExecStart=/usr/bin/dockerd  \
-H tcp://0.0.0.0:2375 \
-H unix:///var/run/docker.sock \
--cluster-store consul://10.0.0.16:8500 \
--cluster-advertise 10.0.0.16:2375
```

### 创建 overlay网络
```bash
docker network create -d overlay --subnet=10.0.2.1/24 overlay-net 
```
这边自动回进行通步，因为使用的是同一个服务器发件。
```bash
[root@node01 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5f3ff8aceaa8        bridge              bridge              local
adb97c875132        docker_gwbridge     bridge              local
497fb0d5ea2f        host                host                local
65e001b471fe        none                null                local
f72e6fcf1082        overlay-net         overlay             global
```

### 创建使用overlay网络的容器

```bash
docker run -tid --name test2 --net=overlay-net centos
docker run -tid --name test3 --net=overlay-net centos
```
进入查看ip信息。

node01
```bash
sh-4.2# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.0.2.3  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 02:42:0a:00:02:03  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.2  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:ac:12:00:02  txqueuelen 0  (Ethernet)
        RX packets 3192  bytes 12218706 (11.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2569  bytes 142308 (138.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 76  bytes 6960 (6.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 76  bytes 6960 (6.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
node02

```bash
sh-4.2# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.0.2.2  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 02:42:0a:00:02:02  txqueuelen 0  (Ethernet)
        RX packets 4  bytes 336 (336.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4  bytes 336 (336.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.2  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:ac:12:00:02  txqueuelen 0  (Ethernet)
        RX packets 2753  bytes 12193675 (11.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2345  bytes 130189 (127.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 78  bytes 6884 (6.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 78  bytes 6884 (6.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

测试网络

```bash
[root@8046b6f6e4aa /]# ping 10.0.2.3
PING 10.0.2.3 (10.0.2.3) 56(84) bytes of data.
64 bytes from 10.0.2.3: icmp_seq=1 ttl=64 time=0.349 ms


[root@8046b6f6e4aa /]# ping 10.0.2.2
PING 10.0.2.2 (10.0.2.2) 56(84) bytes of data.
64 bytes from 10.0.2.2: icmp_seq=1 ttl=64 time=0.023 ms
```
