# 网络名称空间


## linux namespace

**namespace**是Linux内核的一项功能，该功能对内核资源进行分区，以使一组[进程](https://en.wikipedia.org/wiki/Process_(computing))看到一组资源，而另一组进程看到另一组资源。该功能通过为一组资源和进程具有相同的名称空间而起作用，但是这些名称空间引用了不同的资源。资源可能存在于多个空间中。

Linux namespaces 是对全局系统资源的一种封装隔离，使得处于不同 namespace 的进程拥有独立的全局系统资源，改变一个namespace中的系统资源只会影响当前 namespace 里的进程，对其他 namespace 中的进程没有影响。

每一个进程在其对应的 `/proc/[pid]/ns` 下都有其 namespace 信息

查看当前进程的namespace `ll /proc/$$/ns`

```bash
cgroup -> cgroup:[4026531835] cgroup的根目录
ipc -> ipc:[4026531839] 信号量
mnt -> mnt:[4026531840] 文件系统挂载点
net -> net:[4026531992] 网络设备、网络栈、端口
pid -> pid:[4026531836] 进程编号
pid_for_children -> pid:[4026531836]
time -> time:[4026531834]
time_for_children -> time:[4026531834]
user -> user:[4026531837] 用户和用户组
uts -> uts:[4026531838] 主机名和NIS域名
```



##  linux的网络名称空间

linux network namespace 是network namespace 是 linux 内核提供的功能，在实现网络虚拟化中起重要作用，每个网络名称空间中有独立的网络协议栈，如路由表、iptables、端口等。在network namespace可以创建多个隔离的网络空间，它们有独自的网络栈信息。在运行的时候仿佛自己就在独立的网络中。

### linux网络管理命令ip

ip命令是Linux管理网络的工具，他对路由、地址、链路、namespace等的管理。

ip命令的使用说明：

- `ip netns list` 查看网络命名空间

- `ip netns add net2` 创建一个网络命名空间

- `ip link add $name link eth0 type ipvlan mode l2` 在当前名称空间创建一个类型为ipvlan l2模式的接口，将该接口关联至父接口eth0上。 

- `ip link set $name netns $nsName` 将接口加入到对应网络名称空间内

- `ip netns exec $nsName $cmd`  在对应的网络名称空间内运行命令

- `ip link add $p1-name type veth peer name $p2-name` 创建一个网卡对

- `ethtool -S $interface_name` 查看网卡对对应关系，通过index获得另外一端

## 虚拟以太网对 VETH Pair

VETH virtual-ethernet 虚拟以太网对，是网络交换机中的虚拟接口，在linux名称空间中，vEth可以充当名称空间之间的隧道，通过建立一个网桥链接到另一个名称空间中的物理设备，所有设备始终以互连对的形式创建。

![image-20221025174025232](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025174025232.png)

### 实验：使在不同网络名称空间下的vEth互通

实验实现：创建两个名称空间`net1`和`net2`，以及一对VETH设备，并将其分配`veth1`给namespace`net1` 和`veth2`namespace `net2`。这两个名称空间与此VETH对相连。分配一对IP地址，使两个名称空间之间ping通。

开启linuxip转发功能

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

创建两个名称空间

```
ip netns add net1
ip netns add net2
```

创建两个虚拟网卡对

```
ip link a veth0 type veth peer name br-veth0
ip link a veth1 type veth peer name br-veth1
```

创建一个网桥

```
ip link add bridge1 type bridge
# 启动设备
ip link set bridge1 up
```

将虚拟网卡对的一段链接至网桥上

```
ip link set br-veth0 master bridge1
ip link set br-veth1 master bridge1
 
ip link set br-veth1 up
ip link set br-veth0 up
```

虚拟网桥另外一端关联至对应名称空间内

```
ip link set veth0 netns net1
ip link set veth1 netns net2

ip netns exec net1 ip addr add 192.168.0.1/24 dev veth0
ip netns exec net2 ip addr add 192.168.0.2/24 dev veth1

ip netns exec net1 ip link set veth0 up
ip netns exec net2 ip link set veth1 up
```

此时可以通过网桥对这两个网络进行互相ping通，不同网段的需要手动添加相应的路由表规则。而ping自己的地址不通，原因是因为lo设备未启动。

```
ip netns exec net2 ip link set lo up
```

## OVS

## ipip mode

IPIP是[RFC 2003中](https://tools.ietf.org/html/rfc2003)定义的IP over IP隧道。IPIP隧道标头如下所示：

![image-20221025174038429](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025174038429.png)

## 实验：实现一个IPIP隧道

`ipip` 模式需要内核模块 `ipip.ko`  使用命令查看内核是否加载IPIP模块`lsmod | grep ipip` ；使用命令`modprobe ipip` 加载

实验效果，如下图，在两个名称空间内创建一个 tun 设备，然后将该 tun 设备绑定为一个 `ipip` 隧道。

![image-20221025174049463](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025174049463.png)

创建名称空间及虚拟网卡对

```
ip netns add net1
ip netns add net2

ip link a veth0 type veth peer name br-veth0
ip link a veth1 type veth peer name br-veth1

ip link set veth0 netns net1
ip link set veth1 netns net2
```

给虚拟网卡对的一端配置地址

```
ip addr add 192.168.10.1/24 dev br-veth0
ip addr add 192.168.20.1/24 dev br-veth1

ip netns exec net1 ip link set veth0 up
ip netns exec net2 ip link set veth1 up
```

在两个名称空间内分别创建一个ipip tunnel	

```
netns exec net2 ip tunnel add tun1 mode ipip remote 192.168.10.2 local 192.168.20.2
netns exec net1 ip tunnel add tun1 mode ipip remote 192.168.20.2 local 192.168.10.2

ip netns exec net1 ip link set tun1 up
ip netns exec net2 ip link set tun1 up

ip netns exec net1 ip addr add 192.168.100.10 peer 192.168.200.10 dev tun1
ip netns exec net2 ip addr add 192.168.200.10 peer 192.168.100.10 dev tun1
```

这个命令是在对应网络名称空间内创建隧道设备tun1，并设置隧道模式为 `ipip`，然后还需要设置隧道端点，用 `remote` 和 `local` 表示，这是**隧道外层 IP**，对应设置 **隧道内层 IP**，用 `ip addr xx peer xx` 配置。



分析ipip tunnel过程。

1. 首先 ping 命令构建一个 ICMP 请求包，ICMP 包封装在 IP 包中，源目的 IP 地址分别为 `tun1(192.168.200.10)` 和 `tun2(192.168.100.10)` 的地址。
2. 由于 tun1 和 tun2 不在同一网段，所以会查路由表，当通过 `ip tunnel` 命令建立 `ipip` 隧道之后，会自动生成一条路由，如下，表明去往目的地 `192.168.100.10` 的路由直接从 tun1 出去。

```bash
$ ip netns exec net2 route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.10.0    192.168.20.1    255.255.255.0   UG    0      0        0 veth1
192.168.20.0    0.0.0.0         255.255.255.0   U     0      0        0 veth1
192.168.100.10  0.0.0.0         255.255.255.255 UH    0      0        0 tun1
```

3. 由于配置了隧道端点，数据包出了 tun1，到达 veth1，根据 `ipip` 隧道的配置，会封装上一层新的 IP 报头，源目的 IP 地址分别为 `veth2(192.168.20.2)` 和 `veth1(192.168.10.2)`。
4. veth2和 veth1不在一个网段，因为手动添加的路由表，发现去往 `192.168.10.0` 网段从 `192.168.20.1` veth2虚拟网卡对另外一端 br-veth2出。
5. 因为Linux 打开了 `ip_forward`，类似于路由器功能，`192.168.10.0` 和 `192.168.20.1` 为直连路由，直接有路由器转发。完成了net2到net1的过程。
6. 数据包到达 net1 的 veth1，解封装数据包，发现内层 IP 报文的目的 IP 地址是 `192.168.100.10`，这正是自己配置的 `ipip` 隧道的 `tun1(192.168.100.10)` 地址，于是将报文转交 `tun1(192.168.100.10)`。
7. ICMP报文是双向的，故，相通的步骤还要返回到`tun1(192.168.200.10)` 。

> **通过抓包工具查看数据包的详细内容**
>
> ![image-20221025174102350](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025174102350.png)
> 
> ![image-20221025174126074](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025174126074.png)




