# ch2.6 协议数据单元：帧 包 段 报文间的区别


## Overview <sup><a href="#1">[1]</a></sup>

协议数据单元 ***Protocol Data Unit*** (PDU) 是应用于OSI模型中的数据结构，在OSI模型中每一层都会被添加一个header，tailer进行封装，header, tailer加原始报文的组合就是PDU。

在每层中，PDU的名称都是不同的，这也是很多人的疑问，一会数据报文称为数据包，一会数据报文成为数据帧，该文介绍网络中的单元，以了解之间的区别

## 物理层 

物理层数据的呈现方式是以 “位” (***bit***) 为单位的，即0 1，在该层中数据以二进制形式进行传输

## 数据链路层 <sup><a href="#2">[2]</a></sup>

到达数据链路层，实际上可以说进入了TCP/IP栈对底层，而该层的单位为 ”帧“ (***frame***)，该层中，MAC地址会被封装到数据包中，比如以太网帧，PPP帧都是指该层的数据包

该层中数据帧包含：

- 源MAC
- 目的MAC
- 数据，由网络层给出的
- 数据的总长度
- 校验序列

## 网络层 <sup><a href="#3">[3]</a></sup>

在网络层中协议数据单元被称为数据 “包" (***package***) ，是网络间节点通讯的基本单位。该层中IP地址会被封装到数据包内。

该层中数据包包含：

- 标头：源IP，目的IP，协议，数据包编号，帮助数据包匹配网络的位
- playload：数据包的主体
- 标尾：包含几个位，用于告知已到达数据包的末尾与错误检查（循环冗余检查 (***CRC***)）

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/eyJidWNrZXQiOiJjb250ZW50Lmhzd3N0YXRpYy5jb20iLCJrZXkiOiJnaWZcL3F1ZXN0aW9uNTI1LXBhY2tldC5naWYiLCJlZGl0cyI6eyJyZXNpemUiOnsid2lkdGgiOjI5MH0sInRvRm9ybWF0IjoiYXZpZiJ9fQ%3D%3D)

<center>图：数据包组成</center>
<center><em>Source：</em>https://computer.howstuffworks.com/question525.htm</center><br>

例如一个电子邮件，假设电子邮件大小尾3500bit，发送时使用1024的固定大小数据包进行发送，那么每个数据包标头为 96bits，标尾为 32bit，剩余 896bits 将用于实际的数据大小。这里为3500bits，会被分为4个数据包，前三个数据包为 896bits，最后一个数据包大小为 812bits。接收端会根据包编号进行解包重组

## 传输层

### Segment

在传输层TCP协议的协议数据单元被称为 ”段“  (***Segment***) ，上面讲到，IP数据包会以固定大小的数据包进行发送，如果超出大小的会被划分为多个数据包，每个数据包的碎片就被称之为***Segment***。

数据包分割通常会发生在该层，当发生下列场景时会需要分段

- 数据包大于网络支持的最大传输单元 (***MTU***)
- 网络不可靠，将数据包分为更小的包

### datagram <sup><a href="#4">[4]</a></sup>

在传输层UDP协议的协议数据单元被称为 ”数据报“ (***datagram***) ，datagram是一种逐层增加的设计，用于无连接通讯

下图是一个UDP数据报被封装位一个IP数据包：IPv4字段值位17 表示udp协议

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/figure_10-1_600.png)

<center>图：udp的IP包</center>
<center><em>Source：</em>https://notes.shichao.io/tcpv1/ch10</center><br>

对于udp数据报的组成包含header与playload，udp的header大小为固定的8字节

- **源端口**：可选
- **目的端口**：识别接收信息的进程
- **Length**：udp header + udp playload的长度，最小值为8
- **checksum**：与lenght一样其实是多余的，因为第三层包含了这两个信息

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/figure_10-2_600.png)

<center>图：udp数据报组成</center>
<center><em>Source：</em>https://notes.shichao.io/tcpv1/ch10</center><br>

> Notes：使用UDP时应注意避免分段，例如帧中MTU为 1500 字节，假设 IPv4 header为 20 字节，UDP header 为 8 字节，则应用程序最多为数据留下 1472 字节以避免碎片

## 数据

对于传输层之上，协议数据单元没有特定的名词，可以统称为协议数据单元或者数据，整个PDU分层结构图如下图所示，其中 T 表示 Trailer，H 表示 Header，通常H包含源地址和目的地址及一些用于管理通信的控制信息。T含错误检查之类的信息或标志 PDU 结束的字段。

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/page108.gif)

<center>图：PDU分层结构图</center>
<center><em>Source：</em>http://www.telecomworld101.com/Intro2dcRev2/page108.html</center><br>

> Notes：每层的数据字段都由上一层PDU组成，通常情况下，每层只知道自己该层的信息，如网络层仅知道对端网络层，而不知道数据链路层或传输层



> **Reference**
>
> <sup id="1">[1]</sup> [***Protocol Data Unit***](http://www.telecomworld101.com/Intro2dcRev2/page108.html)
>
> <sup id="2">[2]</sup> [***difference between segments packets and frames***](https://www.slashroot.in/difference-between-segments-packets-and-frames)
>
> <sup id="3">[3]</sup> [***What is a packet?***](https://computer.howstuffworks.com/question525.htm)
>
> <sup id="4">[4]</sup> [***User Datagram Protocol (UDP) and IP Fragmentation***](https://notes.shichao.io/tcpv1/ch10/)
