# 打洞技术


## P2P 概述

相比于 `C/S` `B/S` 架构来说， `P2P` 是由 Peer 组成。每个Peer同时是客户端也是服务器。这意味着，P2P网络中的peer点为每个其他的peer提供服务，所有节点直接相互通信，没有中心节点，并共享资源源相互联系。

P2P有结构化的P2P网络和非结构化P2P网络。如`TomP2P` (Java的一个框架，一个分布式哈希表，提供去中心化的键/值基础设施）。而`Gnutella `(第一个分散式P2P文件共享网络)，是非结构化 （unstructured）P2P网络的。另外还有两种类型的P2P网络，即集中式（centralized）P2P网络（Napster）和混合（hybrid p2p）peer网络（如Skype）。

## NAT网络

由于NAT的网络模型，破坏了主机 Peer之间的端到端连接，因此P2P网络需要穿过NAT网络，而穿过NAT网络是目前为P2P技术面临的一个很大的挑战。



网络地址转换，NAT （`Network Address Translation`）是一种模糊指明的机制，可以将两个IP连接在一起，一个NAT设备总是拥有至少两个IP地址，（公网IP，私网IP），NAT就是将数据包上的IP地址在传输时，将私网IP转换为公网IP。每个NAT设备会维护一个NAT表，该表存储了所有活动的连接。



在创建网络映射后，NAT将源IP地址和端口更改为外部源IP地址和端口。NAT保留端口，不将新端口分配给外部源。一旦创建了映射，只要映射存在，与之联系的设备就能够发回消息。不存在NAT映射的所有来自外部的通信请求都是无法穿越NAT。因此，在P2P环境中，如果两个peer位于在NAT之后，两者都无法直接联系，因为它们之间不知道外部IP地址和源端口，NAT表中并没有其所维护的映射信息。所以NAT穿越中的主要问题之一是网络地址转换问题。

![image-20211111234731102](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211111234731102.png)

### NAT网络类型

**一般来讲， NAT网络可以分为四种类型:**  [nat type](https://www.ietf.org/rfc/rfc3489.txt)

- 全锥型(Full Cone)
- 受限锥型(Restricted Cone)， 或者说是IP受限锥型
- 端口受限锥型(Port Restricted Cone), 或者说是IP &#43; PORT受限锥型
- 对称型(Symmetric)

#### Full Cone NAT 全锥形

全锥形网络（Full Cone NAT） 的工作原理类似于 IP 地址一对一映射。 内部 IP 和端口映射到相同的外部地址和端口。 之后，任何外部源都可以通过向外部地址发送数据包来访问内部主机。 这意味着，一旦创建了映射，任何外部主机都可以联系内部主机。如下图

![image-20211111235318310](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211111235318310.png)

#### 地址受限形的锥形NAT  Address Restricted Cone NAT



地址受限形锥形 NAT是，仅当内部主机先联系外部主机时，受限锥形 NAT 才会为相应的内部主机分配外部IP。 外部主机然后能够通过分配的外部地址联系内部主机。 

![image-20211111235606437](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211111235606437.png)

#### 对称型 Symmetric nat

对称 NAT 是最难穿过的NAT，因为对称将随机端口分配给映射。 如果外部主机首先与内部主机连接，则外部主机只能知道外部映射。 在 P2P 场景中， 如果两个Peer（主机）位于 NAT 后面，则它们无法互相通信以让另一个对等体知道所使用的映射。 此外，对称 NAT 几乎不可能知道分配的端口以通过打孔（hole Punching）建立连接。 

![image-20211112000834349](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211112000834349.png)

#### 反向连接 Reverse Connection



反向连接（Reverse Connection）是一种通讯机制，它允许不使用防火墙或 NAT 设备的Peer使用中继（或服务器）连接到另一个使用防火墙（或不带 UPnP、NAT-PmP 或类似机制的 NAT 设备）的Peer建立直接连接。该机制的工作原理如下。设 A 和 B 为网络设备。 A 没有使用任何类型的防火墙，而 B 使用防火墙。假设也是服务器（或中继）S



假设 `A` 和` B` 为网络设备。 A 没有使用任何类型的防火墙，而 B 使用防火墙（或也是服务器、中继）S。`A`要连接到`B`。`A`首先向`S`发送连接建立请求，然后`S`使用与`A`已经建立的连接将此消息转发给 `B`, 一旦 `B` 与 `S` 建立连接，`S` 开始帮助`B`连接到 `A`。在 `A` 和 `B` 都完成连接后，尽管 `B` 使用防火墙（或 NAT 设备），但 `A` 已经能够与` B `形成了 Peer to Peer直接通信。

反向连接设置的优点是两个网络设备即使使用了防火墙（或 NAT 设备）也能够进行通信。但是这种机制有各种限制。首先，仅允许两个设备中的一个使用防火墙（或 NAT 设备）。其次，防火墙后面的设备（或 NAT 设备）必须像中继一样连接到外部主机 或服务器。第三，不使用任何防火墙的设备（或NAT设备）需要知道防火墙后面设备的地址，并且需要能够连接到已经连接到防火墙后面设备（或NAT设备）的主机）。

仅当所有前面提到的限制都适用时，反向连接才可用。但是，由于当今许多家庭宽带都在使用 NAT 和防火墙，因此这不是连接位于 NAT 设备或防火墙后面的 P2P 网络的两个Peer的可靠的选择。

![image-20211112002041307](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211112002041307.png)

#### UPnP

在当今的家庭宽带网络中，在很多情况下，两端的Peer都使用了 NAT 设备，无法建立连接。 而 `UPnP` **通用即插即用**（Universal Plug and Play），即使为了解决此网络环境下的一种网络架构，从而可以使用P2P网络。简而言之，`UPnP` 技术的工作过程如下：

- 寻址 （Addressing）：使每个设备都获得一个 IP 地址。
- 发现 （Discovery）：UPnP Control Point，通知所有设备或其他所有设备通知其他们的存在。
- 描述（Description）：UPnP Control Point 学习其他设备的能力。
- 控制（Control）：UPnP Control Point 向设备发送命令。
- 事件（Eventing）：UPnP Control Point 监听设备的状态的变化。
- 展示（Presentation）：UPnP Control Point 显示设备的用户界面。

UPnP特点：

- 能够在任何类型的 NAT 或防火墙阻止的情况下进行通信
- 可能无法在路由器和服务器上启用
- 不提供身份验证机制
- 不支持多层 NAT

#### NAT端口映射协议

**NAT端口映射协议**（NAT Port Mapping Protocol，简写**`NAT-PMP`**）是苹果公司于 2005 年开发的 NAT 。该协议是 NAT 设备的扩展。



#### 打洞 Hole Punching

打洞（Hole Punching）是可以使NAT 后面的两个网络设备通过让另一个设备知道它们的私有断定与公共端点之间的映射信息来相互联系。 因此，它属于基于的NAT打洞机制。 必要条件是需要存在一个第三方网络设备，该设备可以获得两设备间的信息，并进行信息交换的服务器。 Hole Puching的工作原理如下:

- Host A 和 Host B 都向Host  C(中继器) 发送 UDP 数据包。当数据包通过它们的 NAT 时，NAT 将源 IP 地址重写为其公网可访问的 IP 地址。它也可能重写源端口号，在这种情况下，UDP 打孔几乎是不可能的。
- C 记录来自 Host A 和Host B 的传请求的 IP 与端口。
- C 将两者（Host A、B）信息交换 （告诉A B的IP端口，告诉B A的IP端口）
- Host A 和Host  B 的第一个数据包在进入彼此的 NAT设备 时被拒绝。然而，当数据包从 A 的 NAT 在端口X传递到 B 的 NAT 时，NAT A 注意到了它，因此在其防火墙上打了一个洞，以允许来自 B 的 NAT 的 IP 的传入数据包，从端口 X . B 的 NAT 也会发生同样的情况，它制定了一个规则，允许来自 A 的 NAT 的 IP 地址的数据包从端口 Y传入。
- 完成时，当 Host A 和 Host B 相互发送数据包时，此时会被接受，即完成了P2P网络。

![image-20211115000012830](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211115000012830.png)

![image-20211115000245628](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20211115000245628.png)



&gt; **Reference**
&gt;
&gt; [打洞](https://blog.csdn.net/yhc166188/article/details/107966193)
&gt;
&gt; [P2P技术详解(三)：P2P中的NAT穿越(打洞)方案详解(进阶分析篇)](http://www.52im.net/thread-2872-1-1.html)
