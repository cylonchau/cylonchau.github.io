# networking stack


[toc]

## Linux 架构概述 &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;

本章节简单阐述Linux系统的结构，并讨论子系统中的模块之间以及与其他子系统之间的关系。

Linux内核本身鼓励无用，是作为一个操作系统的一部分参与的，只有为一个整体时他才是一个有用的实体，下图展示了Linux操作系统的分层

![image-20221028214224508](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221028214224508.png)

&lt;center&gt;图：Linux子系统分层图&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://docs.huihoo.com/linux/kernel/a1/index.html&lt;/center&gt;&lt;br&gt;

由图可以看出Linux操作系统由四部分组成：

- 用户应用
- OS服务，操作系统的一部分（例如shell）内核编程接口等
- 内核
- 硬件控制器，CPU、内存硬件、硬盘和NIC等都数据这部分

## Linux内核阐述

Linux内核将所有硬件抽象为一致的接口，为用户进程提供了一个虚拟接口，使用户无需知道计算机上安装了哪些物理硬件即可编写进程，并且Linux支持用户进程的多任务处理，每个进程都可以视作为操作系统的唯一进程独享硬件资源。内核负责维护多个用户进程，并协调其对硬件资源的访问，使得每个进程都可以公平的访问资源，并保证进程间安全。

Linux内核主要为五个子系统组成：

- 进程调度器(***SCHED***)， 控制进程对 CPU 的访问。调度程序执行策略，确保进程可以公平地访问 CPU。
- 内存管理器 (***MM***)， 允许多个进程安全地共享操作系统的内存
- 虚拟文件系统 (***VFS***)，向所有设备提供通用文件接口来抽象出各种硬件设备
- 网络接口 (***NET***)，提供对多种网络标准与各种网络硬件的访问
- 进程间通信 (***IPC***)，在单个操作系统上的多种机制进程间通信机制

## 网络子系统架构 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

网络子系统功能主要是允许 Linux 系统通过网络连接到其他系统。支持多种硬件设备，以及可以使用的多种网络协议。网络子系统抽象了这两个实现细节，以便用户进程和其他内核子系统可以访问网络，而不必知道使用什么物理设备或协议。

子系统模块包含

- 网络设备驱动层 (***Network device drivers***)，网络设备驱动程序与硬件设备通信。每个硬件设备都有对应的设备驱动程序模块。
- 独立设备接口层(***device independent interface***)，设备独立接口提供了所有硬件设备的统一视图，因此在网络子系统之上的级别无需了解硬件信息
- 网络协议层 (*** network protocol***)，网络协议实现了网络传输的协议
- 协议独立/无关接口层 (***protocol independent interface***)，提供了独立于硬件设备的网络接口，为内核内其他子系统访问网络时不依赖特定的协议和硬件接口。
- 系统调用层 (***system call***) 用于限制用户进程导出资源的访问

网络子系统的结构图如下图所示，

![image-20221029170129050](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221029170129050.png)

&lt;center&gt;图：网络子系统中的上下文&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://docs.huihoo.com/linux/kernel/a1/index.html&lt;/center&gt;&lt;br&gt;

当网络子系统转换为网络栈时，如下图所示

![img](https://miro.medium.com/max/700/1*LcGaDm_ZOCbrIerM2UDj0g.png)

&lt;center&gt;图：ISO Stack与TCP/IP Stack&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.washington.edu/R870/Networking.html&lt;/center&gt;&lt;br&gt;

当然Linux网络子系统是类似于TCP/IP栈的一种结构，当发生一个网络传输时，数据包会按照所经过的层进行封装。例如应用层应用提供了REST API，那么应用将要传输的数据封装为HTTP协议，然后传递给向下的传输层。传输层是TCP协议就会被添加对应的TCP包头。整个封装过程原始包保持不变，会根据所经过层的不同增加固定格式的包头。

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/understandlni_1304.jpg)

&lt;center&gt;图：数据包传输在每层被封装的过程&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;http://www.embeddedlinux.org.cn/linux_net/0596002556/understandlni-CHP-13-SECT-1.html&lt;/center&gt;&lt;br&gt;

对于Linux来说TCP/IP 的五层结构则是构成网络子系统的的核心组件，下图是Linux网络栈结构图

![image-20221030005922405](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221030005922405.png)

&lt;center&gt;图：Linux网络栈的结构图&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://medium.com/geekculture/linux-networking-deep-dive-731848d791c0&lt;/center&gt;&lt;br&gt;

- 图中橙色部分是位于TCP/IP的五层结构中的应用层，应用层向下通讯通过 `system call` 与 socket接口进行交互
- 蓝色部分是位于内核空间，socket向下则是传输层与网络层
- 最底层是物理层包含网卡驱动与NIC

通过图可以看出，NIC是发送与接收数据包的基本单位，当系统启动时内核通过驱动程序向操作系统注册网卡，当数据包到达网卡时，被放入队列中。内核通过硬中断，运行中断处理程序，为网络帧分配内核数据结构(sk_buff)，并将其拷贝到缓冲区中，此为内核与网卡交互的过程。

网卡硬中断只处理网卡核心数据的读取或发送，网络协议栈中的大部分处理都在软中断中进行处理。内核协议栈将从缓冲区中取出网络帧，通过网络协议栈，从下到上的根据网络栈结构逐层处理这个网络帧。

## Socket &lt;sup&gt;&lt;a href=&#34;#4&#34;&gt;[4]&lt;/a&gt;&lt;/sup&gt;

Unix Socket是一种使用了Unix文件描述符的IPC机制，在网络栈中是位于内核空间网络栈的一层，是一个用户空间与传输层之间的一个接口，可以为网络连接, 文本文件, 终端或其他；他的行为很像一个文件描述符，因为信息的读写，`read()`, `write()`与文件的方式很相似。下图是socket通信模型。

![image-20221031203257214](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221031203257214.png)

&lt;center&gt;图：socket通信模型&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://slideplayer.com/slide/10740698/&lt;/center&gt;&lt;br&gt;

作为用户空间到内核空间的第一层，Socket位于两层之间，由于IPC机制支持不同的通讯协议以及需要对不同的网络协议进行访问，故这些协议实现为位于socket的层，这种情况下，用户空间仅通过系统调用socket接口，而内核空间负责一些其他工作，例如，缓冲区管理，标准协议接口，网络接口与各种不同的网络协议。

&gt; Notes:
&gt;
&gt; - `/etc/protocols` 定义的协议号
&gt; - `/etc/services` 定义的服务的端口号

## 网络栈的工作原理

当网络包到达时，网卡（硬中断&#43;DMA）通过DMA将网络数据包放入队列中，告知中断程序硬中断已收到网络数据包。

### 数据包的发送

用户程序发送网络包时，通过网络栈模型自上而下逐层处理帧：

- 应用层：通过系统调用，调用socket API发送网络包，会被限制在内核空间的socket层，socket层将数据包放入到缓冲区内。
- 传输层：网络栈从socket取出数据包，传输层添加TCP标头
- 网络层：将IP添加到数据标头，根据MTU大小分片
- 数据链路层：MAC地址寻址，并添加到帧头尾，将帧放入发送队列，触发软中断通知
- 物理层：网卡驱动通过DMA从发送队列读取网络帧，通过网卡发送出

### 数据包的接收

内核网络栈从缓冲区读取帧，通过网络栈模型自下而上逐层处理帧：

- 数据链路层：
    - 检查数据包的有效性
    - 确定网络协议类型 IPV4 or IPV6
    - 去除帧 头, 尾
- 网络层：
    - 取出IP头，确定网络流量的方向（转发或者本机流量）
    - 删除标头，传递给传输层
- 传输层：取出TCP/UDP协议头，根据源IP, 目的IP, 源端口, 目的端口作为标识找到socket，将数据报文放置socket缓冲区
- 应用层：应用程序通过socket来读数据

下图为网络栈收/发数据的结构图

![image-20221031210517384](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221031210517384.png)

&lt;center&gt;图：Linux网络进程接收网络数据包流程图&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://slideplayer.com/slide/10740698/&lt;/center&gt;&lt;br&gt;

## 网络子系统分层结构

在了解了网络接受网络数据包的流程后，还需要对网络子系统中分层结构进行了解，在该结构中将需要基础掌握一些对于工作与网络子系统中的API的命令是如何调用的。

下图是结合 《深入理解Linux网络技术内幕》第13章 &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt; 中插图13-2与 托马斯格拉夫发表于2019年的文章 &#34;How to Make Linux Microservice-Aware with Cilium and eBPF&#34; &lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt; 的结合旨在让零基础同学可以更好的了解到各API的分层调用

![image-20221104153434745](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104153434745.png)

&lt;center&gt;图：Linux网络子系统分层调用&lt;/center&gt;&lt;br&gt;

图中可以看出，是一个基于TCP/IP栈的调用模型，其中应用层包含了常用的工具：

- 配置IP路由：`ip`
- ip防火墙（包过滤）：`iptables`
- 流量整形：`tc`
- 网络抓包：`tcpdump`
- 网卡信息：`ethtool`

对于云原生网络中，了解完整的分层是非常重要的，这将有利于开发基于eBPF服务。下面就简单的论证下该图

正如图中所示，所有的网络命令都是提供给用户的用户空间API，当发生网络动作时是需要通过内核将数据导入/出，这里使用了系统调用，调用内核提供的导入到用户空间的接口，例如 `socket`，`sysctl` 等，更多的接口介绍可以详见《深入理解Linux网络技术内幕》第3章 &lt;sup&gt;&lt;a href=&#34;#6&#34;&gt;[6]&lt;/a&gt;&lt;/sup&gt; 

到达socket后，继续向下通信时，socket提供了几种级别的接口，这些可以在常见编程语言包中被提供

- `AE_PACKAGE` / `PE_PACKAGE`：提供设备级别的API，通俗来讲，就是在网络层之下发送/接受消息的接口，工作于2层，这将允许用户在用户空间实现物理层数据包发送和接收
- `AF_INET` / `PE_INET`：是基于网络层Socket类型，`AF_INET`是指IPv4，`AF_INET6` 是IPv6，这里就是IP 地址和端口号。

如图所示，对于 `PE_PACKAGE` 套接字类型而言，Linux在链路层捕捉帧并将其注入至链路层的方式，这样跳过了所有的中间层，例如 `tcpdump` 与 `ethtool`， `PE_PACKAGE` 套接字通过将帧直接交给 `dev_queue_xmit`。

- `dev_queue_xmit` 是传输 buffer (`sk_buff`) 到网络设备中的函数，将封包传递给TC或QoS层，L3封包时调用

接下来是iptables，netfilter，是工作与多层协议栈中一系列hook，用户端由命令行工具iptables/nftables控制，可以在数据包经由的数据点上被调用对应的hook函数来改变包的行为。所有的数据包都独立存在于对应的协议栈，经过的数据包会便利所有对应的hook，因为iptables(etables)支持工作于L2的ARP协议。所有的hook都存在与每个网络名称空间内，并且每个网络设备都拥有ingress hook，这也是云原生网络中提到的为什么使用eBPF 跳过netfilter框架可以提升网络性能。

![img](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/understandlni_1801.jpg)

&lt;center&gt;图：Linux 栈中经由netfilter框架示意图&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;http://www.embeddedlinux.org.cn/linux_net/0596002556/understandlni-CHP-18-SECT-1.html&lt;/center&gt;&lt;br&gt;

接下来就是传统的一些应用，例如telnet，ping都是使用了`AE_PACKAGE` / `PE_PACKAGE` 传统联网模式

最后一个点就是 ***traffic control*** TC，是工作与L2的一组队列与其机制组成的，通常情况下是一个队列，上面也提到，所有的设备都是使用队列来调度底层设备进入的数据包，liunx中默认的队列是 `qdisc ` 。



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [***Conceptual Architecture of the Linux Kernel***](https://docs.huihoo.com/linux/kernel/a1/index.html)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [***Linux — Networking Deep Dive***](https://medium.com/geekculture/linux-networking-deep-dive-731848d791c0)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [***Network Stack Chapter13***](http://www.embeddedlinux.org.cn/linux_net/0596002556/understandlni-CHP-13-SECT-1.html)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [***User Datagram Protocol (UDP) and IP Fragmentation***](https://notes.shichao.io/tcpv1/ch10/)
&gt;
&gt; &lt;sup id=&#34;5&#34;&gt;[5]&lt;/sup&gt; [***How to Make Linux Microservice-Aware with Cilium and eBPF***](https://www.infoq.com/presentations/linux-cilium-ebpf/)
&gt;
&gt; &lt;sup id=&#34;6&#34;&gt;[6]&lt;/sup&gt; [***Network Stack Chapter13***](http://www.embeddedlinux.org.cn/linux_net/0596002556/understandlni-CHP-3-SECT-1.html)




