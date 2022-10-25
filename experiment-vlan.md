# 实验 - VLAN


虚拟局域网VLAN（Virtual Local Area Network），是将一个物理的LAN在逻辑上划分成多个广播域的通信技术。



VLAN内的主机间可以直接通信，而VLAN间不能直接通信，从而将广播报文限制在一个VLAN内。

以太网是一种基于CSMA/CD（Carrier Sense Multiple Access/Collision Detection）的共享通讯介质的数据网络通讯技术。当主机数目较多时会导致冲突严重、广播泛滥、性能显著下降甚至造成网络不可用等问题。通过交换机实现LAN互连虽然可以解决冲突严重的问题，但仍然不能隔离广播报文和提升网络质量。

在这种情况下出现了VLAN技术，这种技术可以把一个LAN划分成多个逻辑的VLAN，每个VLAN是一个广播域，VLAN内的主机间通信就和在一个LAN内一样，而VLAN间则不能直接互通，这样，广播报文就被限制在一个VLAN内。

## VLAN的作用

- 限制广播域：广播域被限制在一个VLAN内，节省了带宽，提高了网络处理能力。

- 增强局域网的安全性：不同VLAN内的报文在传输时是相互隔离的，即一个VLAN内的用户不能和其它VLAN内的用户直接通信。

- 提高了网络的健壮性：故障被限制在一个VLAN内，本VLAN内的故障不会影响其他VLAN的正常工作。

- 灵活构建虚拟工作组：用VLAN可以划分不同的用户到不同的工作组，同一工作组的用户也不必局限于某一固定的物理范围，网络构建和维护更方便灵活。



## VLAN Tag

要使交换机能够分辨不同VLAN的报文，需要在报文中添加标识VLAN信息的字段。IEEE 802.1Q协议规定，在以太网数据帧的目的MAC地址和源MAC地址字段之后、协议类型字段之前加入4个字节的VLAN标签（又称VLAN Tag，简称Tag），用以标识VLAN信息。

![image-20220506224238271](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506224238271.png)

VLAN Tag各字段含义：

| 字段 | 长度  | 含义                                                         | 取值                                                         |
| ---- | ----- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| TPID | 2Byte | Tag Protocol Identifier（标签协议标识符），表示数据帧类型。  | 表示帧类型，取值为0x8100时表示IEEE 802.1Q的VLAN数据帧。如果不支持802.1Q的设备收到这样的帧，会将其丢弃。<br/><br/>各设备厂商可以自定义该字段的值。当邻居设备将TPID值配置为非0x8100时， 为了能够识别这样的报文，实现互通，必须在本设备上修改TPID值，确保和邻居设备的TPID值配置一致。 |
| PRI  | 3bit  | Priority，表示数据帧的802.1p优先级。                         | 取值范围为0～7，值越大优先级越高。当网络阻塞时，设备优先发送优先级高的数据帧。 |
| CFI  | 1bit  | Canonical Format Indicator（标准格式指示位），表示MAC地址在不同的传输介质中是否以标准格式进行封装，用于兼容以太网和令牌环网。 | CFI取值为0表示MAC地址以标准格式进行封装，为1表示以非标准格式封装。在以太网中，CFI的值为0。 |
| VID  | 12bit | VLAN ID，表示该数据帧所属VLAN的编号。                        | VLAN ID取值范围是0～4095。由于0和4095为协议保留取值，所以VLAN ID的有效取值范围是1～4094。 |



 

其中，数据帧中的VID（VLAN ID）字段标识了该数据帧所属的VLAN，数据帧只能在其所属VLAN内进行传输。

对于交换机来说，其内部处理的数据帧都带有VLAN标签，而现网中交换机连接的设备有些只会收发Untagged帧，要与这些设备交互，就需要接口能够识别Untagged帧并在收发时给帧添加、剥除VLAN标签。同时，现网中属于同一个VLAN的用户可能会被连接在不同的交换机上，且跨越交换机的VLAN可能不止一个，如果需要用户间的互通，就需要交换机间的接口能够同时识别和发送多个VLAN的数据帧。

## VLAN PVID

缺省VLAN又称PVID（Port Default VLAN ID）。设备处理的数据帧都带Tag，当设备收到UNTagged帧时，就需要给该帧添加Tag，添加什么Tag，就由接口上的缺省VLAN决定。一个物理端口只能拥有一个PVID，当一个物理端口拥有了一个PVID的时候，必定会拥有和PVID相等的VID，而且在这个VID上，这个物理端口必定是Untagged Port。

因此，根据接口连接对象以及对收发数据帧处理的不同，华为定义了4种接口的链路类型：Access、Trunk、Hybrid和QinQ，以适应不同的连接和组网：

- Access接口：一般用于和不能识别Tag的用户终端（如用户主机、服务器等）相连，或者不需要区分不同VLAN成员时使用。Access接口大部分情况只能收发Untagged帧，且只能为Untagged帧添加唯一的VLAN Tag。
- Trunk接口：一般用于连接交换机、路由器、AP以及可同时收发Tagged帧和Untagged帧的语音终端。它可以允许多个VLAN的帧带Tag通过，但只允许一个VLAN的帧从该类接口上发出时不带Tag（即剥除Tag）。
- Hybrid接口：既可以用于连接不能识别Tag的用户终端（如用户主机、服务器等）和网络设备（如Hub、傻瓜交换机），也可以用于连接交换机、路由器以及可同时收发Tagged帧和Untagged帧的语音终端、AP。它可以允许多个VLAN的帧带Tag通过，且允许从该类接口发出的帧根据需要配置某些VLAN的帧带Tag（即不剥除Tag）、某些VLAN的帧不带Tag（即剥除Tag）。
- 使用QinQ（802.1Q-in-802.1Q）协议，一般用于私网与公网之间的连接，也被称为Dot1q-tunnel接口。它可以给帧加上双层Tag，即在原来Tag的基础上，给帧加上一个新的Tag，从而可以支持多达4094×4094个VLAN。

| 接口类型   | 对接收不带Tag的报文处理                                      | 对接收带Tag的报文处理                                        | 发送帧处理过程                                               |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Access接口 | 接收该报文，并打上缺省的VLAN ID。                            | 当VLAN ID与缺省VLAN ID相同时，接收该报文。<br/>当VLAN ID与缺省VLAN ID不同时，丢弃该报文 | 先剥离帧的PVID Tag，然后再发送。                             |
| Trunk接口  | 打上缺省的VLAN ID，当缺省VLAN ID在允许通过的VLAN ID列表里时，接收该报文。<br/>打上缺省的VLAN ID，当缺省VLAN ID不在允许通过的VLAN ID列表里时，丢弃该报文。 | 当VLAN ID在接口允许通过的VLAN ID列表里时，接收该报文。<br/>当VLAN ID不在接口允许通过的VLAN ID列表里时，丢弃该报文 | 当VLAN ID与缺省VLAN ID相同，且是该接口允许通过的VLAN ID时，去掉Tag，发送该报文。<br/>当VLAN ID与缺省VLAN ID不同，且是该接口允许通过的VLAN ID时，保持原有Tag，发送该报文。 |
| Hybrid接口 | 打上缺省的VLAN ID，当缺省VLAN ID在允许<br/>通过的VLAN ID列表里时，接收该报文。<br/><br/>打上缺省的VLAN ID，当缺省VLAN ID不在允许通过的VLAN ID列表里时，丢弃该报文。 | 当VLAN ID在接口允许通过的VLAN ID列表里时，接收该报文。<br/>当VLAN ID不在接口允许通过的VLAN ID列表里时，丢弃该报文。 | 当VLAN ID是该接口允许通过的VLAN ID时，发送该报文。可以通过命令设置发送时是否携带Tag。 |

由上面各类接口添加或剥除VLAN标签的处理过程可见：

当接收到不带VLAN标签的数据帧时，Access接口、Trunk接口、Hybrid接口都会给数据帧打上VLAN标签，但Trunk接口、Hybrid接口会根据数据帧的VID是否为其允许通过的VLAN来判断是否接收，而Access接口则无条件接收。

当接收到带VLAN标签的数据帧时，Access接口、Trunk接口、Hybrid接口都会根据数据帧的VID是否为其允许通过的VLAN（Access接口允许通过的VLAN就是缺省VLAN）来判断是否接收。

当发送数据帧时：

1. Access接口直接剥离数据帧中的VLAN标签。
2. Trunk接口只有在数据帧中的VID与接口的PVID相等时才会剥离数据帧中的VLAN标签。
3. Hybrid接口会根据接口上的配置判断是否剥离数据帧中的VLAN标签。

因此，Access接口发出的数据帧肯定不带Tag，Trunk接口发出的数据帧只有一个VLAN的数据帧不带Tag，其他都带VLAN标签，Hybrid接口发出的数据帧可根据需要设置某些VLAN的数据帧带Tag，某些VLAN的数据帧不带Tag。



##  VLAN-Access Port

Access接口一般用于和不能识别Tag的用户终端（如用户主机、服务器等）相连，或者不需要区分不同VLAN成员时使用。它只能收发Untagged帧，且只能为Untagged帧添加唯一VLAN的Tag。

![image-20220506225142143](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506225142143.png)

配置VLAN access

```
interface GigabitEthernet0/0/1
port link-type access
port default vlan 10
```

可以看到收到的包和回来的包并添加VLAN Tag

![image-20220506225154629](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506225154629.png)



### VLAN-Hybrid Port

Hybrid接口既可以用于连接不能识别Tag的用户终端（如用户主机、服务器）和网络设备（如Hub），也可用于连接换机、路由器以及可同时收发Tagged帧和Untagged帧的语音终端、AP。它可允许多个VLAN的帧带Tag通过，且允许从该类接口发出的帧根据需要配置某些VLAN的帧带Tag（即不剥除Tag）某些VLAN的帧不带Tag（即剥除Tag）。

![image-20210131144114120](../../../../images/vlan interface/image-20210131144114120.png)

配置Hybrid vlan

```
interface GigabitEthernet0/0/1
port hybrid pvid vlan 10
port hybrid tagged vlan 10
```

抓包可以看到对应的Hybrid port收到的

![image-20210131132701528](../../../../images/vlan interface/image-20210131132701528.png)

![image-20210131132642594](../../../../images/vlan interface/image-20210131132642594.png)



实现pc1与pc2都可与pc3互通，pc1与pc2之间不能互通。

![image-20210131195302130](../../../../images/vlan interface/image-20210131195302130.png)

```
system-view
sysname SW7

interface GigabitEthernet0/0/1
# 进来时给包打标签
port hybrid pvid vlan 10
# 出去时去掉标签
port hybrid untagged vlan 10 30

interface GigabitEthernet0/0/2
port hybrid pvid vlan 20
port hybrid untagged vlan 20 30

interface GigabitEthernet0/0/3
port hybrid pvid vlan 30
port hybrid untagged vlan 10 20 30
```

流程分析：当pc1流量进入SW时会被添加vlan10的tag，在通过vlan30口出去时会进行 `port hybrid untagged|tag` == `port trunk access xxx` 这个操作是“是否允许这些tag通过，通过时进行对tag的操作 `tag` | `untag` ”

**实验**

![image-20210131201307039](../../../../images/vlan interface/image-20210131201307039.png)



```
system-view 
sysname SW8

interface GigabitEthernet0/0/1
port hybrid pvid vlan 10
port hybrid untagged vlan 20

interface GigabitEthernet0/0/2
port hybrid pvid vlan 20
port hybrid untagged vlan 10
```

这个实验可以证实，在接口G0/0/1到G0/0/2分配配置了untapped对方vlan的id发现无法ping通。

在默认情况下hybrid只允许vlan1通过，而G0/0/1会被打上vlan10的tag，而到了G0/0/2，此时的标签设置的是双向都设置的为只允许vlan10的通过而不允许vlan20通过，而G0/0/1则相反。所以双方都需要`port hybrid untagged vlan 10 20`



## VLAN-Trunk Port

Trunk接口一般用于连接交换机、路由器、AP以及可同时收发Tagged帧和Untagged帧的语音终端。它可以允许多个VLAN的帧带Tag通过，但只允许一个VLAN的帧从该类接口上发出时不带Tag（即剥除Tag）。

![image-20210131152952129](../../../../images/vlan interface/image-20210131152952129.png)



配置交换机 4 实现图2 4的论点

![image-20210131172254100](../../../../images/vlan interface/image-20210131172254100.png)

```
system-view
sysname SW3
vlan batch 10 20

interface GigabitEthernet0/0/1
port link-type access
port default vlan 10

interface GigabitEthernet0/0/2
port link-type access
port default vlan 20

interface GigabitEthernet0/0/3
port link-type trunk
port trunk allow-pass vlan 10 20
```

SW4

```
system-view
sysname SW4
vlan batch 10 20

interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 10 20

interface GigabitEthernet0/0/2
port link-type access
port default vlan 10

interface GigabitEthernet0/0/3
port link-type access
port default vlan 20
```

![image-20210131193456826](../../../../images/vlan interface/image-20210131193456826.png)

此图完成的是图 1 3步骤的论点

SW5

```
interface GigabitEthernet0/0/1
port link-type access
port default vlan 10

interface GigabitEthernet0/0/2
port link-type access
port default vlan 10
```

SW6

```
interface GigabitEthernet0/0/1
port link-type trunk
port trunk pvid vlan 10
port trunk allow-pass vlan 10 20

interface GigabitEthernet0/0/2
port link-type access
port default vlan 10
```

 [vlan.zip](..\..\..\images\vlan interface\vlan.zip) 
