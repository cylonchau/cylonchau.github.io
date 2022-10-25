# docker 网络


## 1. docker的四种网络模式

### 1.1 Bridge模式（默认）

当Docker进程启动时，会在宿主机上创建一个<font style="background:#fee904;" size=2>名为docker0的虚拟网桥</font>，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

默认ip段172.17.0.1/16；<font style="background:#fee904;" size=2>从docker0子网中分配一个IP给容器使用，并设置docker0的IP为容器的默认网关</font>。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看。

使用 `docker run -p` 时，docker实际是在iptables做了DNAT规则，实现端口转发功能。可以使用 `iptables -t nat -nL` 查看。

### 1.2 host模式

启动容器的时候使用host模式，那么这个容器将<font style="background:#fee904;" size=2>不会获得一个独立的Network Namespace</font>，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

### 1.3 none模式

使用none模式，Docker容器拥有自己的Network Namespace，但是，并不为Docker容器进行任何网络配置。也就是说，这个Docker容器没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等。

### 1.4 container模式
这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。

## 2. 容器外部访问原理

```bash
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:a5ff:fe59:2034  prefixlen 64  scopeid 0x20<link>
        ether 02:42:a5:59:20:34  txqueuelen 0  (Ethernet)
        RX packets 56986  bytes 2746876 (2.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 64106  bytes 503304169 (479.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

```bash
docker run -itd --name test_network -p 80:80 centos:6

$ iptables -t nat -nL
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0           
MASQUERADE  tcp  --  172.17.0.2           172.17.0.2           tcp dpt:80

Chain DOCKER (2 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80

```

## 3. 配置桥接网络

#### 3.1 下载网桥管理工具
```bash
yum install bridge-utils -y
```

#### 3.2 停止docker并删除docker0网桥
```bash
ip link set dev docker0 down
brctl delbr docker0
```
#### 3.3 创建新桥接物理网络虚拟网桥test

```bash
brctl addbr test
ip link set dev test up
ip addr add 10.10.10.0/24 dev test     #为br0分配物理网络中的ip地址
ip addr del 10.0.0.0/24 dev eth0 #将宿主机网卡的IP清空
brctl addif test eth0
```


```bash
$ docker run -itd --name centos centos:6
26ea6dde6564006f148e4977d131e671b578c2c5df313b1d872940a9e48f0309

$ docker attach centos
[root@26ea6dde6564 /]# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:00:02  
          inet addr:192.168.0.2  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::42:c0ff:fea8:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:578 (578.0 b)  TX bytes:578 (578.0 b)
```

[Docker Centos7 下建立 Docker 桥接网络 - weifengCorp - 博客园](https://www.cnblogs.com/weifeng1463/p/7468497.html)
[centos7 docker宿主机配置桥接物理网络终极实战-zhaoyfcomeon-成长之路-51CTO博客](http://blog.51cto.com/zhaoyfcomeon/1968886)
[docker自定义网桥 - CSDN博客](https://blog.csdn.net/jackliu16/article/details/79360581)

将docker0加入网桥中
```
$ brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024206e55aa2	no		
```



  ```bash
  $ docker run -itd --net host centos
  5237ae95c660a0354046f0e5ff839c9a4babda7c9743ca8f4d2338e6a8445e55

              "Networks": {
                  "host": {
                      "IPAMConfig": null,
                      "Links": null,
                      "Aliases": null,
                      "NetworkID": "b2768c9e7cb0de16cde0626abbca8414ca80c96101531aa7842dfed7ee9fc884",
                      "EndpointID": "645627ef6384f1f36e3cfeddd5d9346cfdf1e8997b597c2ec1edd8577e17d6f5",
                      "Gateway": "",
                      "IPAddress": "",
                      "IPPrefixLen": 0,
                      "IPv6Gateway": "",
                      "GlobalIPv6Address": "",
                      "GlobalIPv6PrefixLen": 0,
                      "MacAddress": "",
                      "DriverOpts": null
                  }
              }
          }
      }
  ]
  ```
> **查看网络模式**

```bash
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
a39cc87800d6        bridge              bridge              local
b2768c9e7cb0        host                host                local
329e3d9d1c3f        none                null                local
```

---
tags: []
isStarred: false
isTrashed: false

修改docker0默认ip

```bash
$ cat /etc/docker/daemon.json 
{
    "bip": "192.168.100.1/24"
}
```
