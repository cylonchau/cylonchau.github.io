# what is eBPF


## eBPF介绍

eBPF是 ***Extended Berkeley Packet Filter***，主要是用于包过滤的。为什么叫*** Berkeley Packet Filter*** 是因为论文出自 ***Lawrence Berkeley Laboratory***（相对的论文可以[参考](https://www.tcpdump.org/papers/bpf-usenix93.pdf) <sup><a href="#1">[1]</a></sup>）。“***E***" 是使BPF不仅仅是包过滤。

***eBPF*** 目前提供的功能不仅仅是包过滤，它是一个允许用户在操作系统内核加载自定义程序的框架，来自于 ”What Is eBPF?“ 

> eBPF is a framework that allows users to load and run custom programs within the kernel of the operating system. That means it can extend or even modify the way the kernel behaves. <sup><a href="#2">[2]</a></sup>

eBPF验证器

对于如果想改变Linux内核功能需要合并代码到内核或者编写内核模块。前者需要被社区接受，这需要很长一个周期；而后者可以很好的扩展内核功能，但都存在一个问题 ”==安全运行==“

”安全运行“ 问题包含”漏洞“和“崩溃”，考虑到这些，eBPF为安全运行提供了一个非常不同的方法：***eBPF verifier*** ，***eBPF verifier*** 将确保应用只能够在安全情况下被运行。

***eBPF verifier*** 保证了 eBPF 程序运行的 ”安全“ 和 ”验证“ 

”验证“ (Verification) 是指对程序进行分析，确保无论输入时什么，都会在有限的之阵内终止。例如在解除指针时，确保指针不是空置，解除对指针引用意味着将会 “查找这个地址的值”，解引用空置，会程序崩溃，而在内核中空指针会引起整个机器崩溃。

”安全“ (Security)是指将确保eBPF程序运行时安全的，这种场景将限制eBPF访问的内存为只能被访问的内存。例如，有一个 eBPF 程序出发在一个网络stack上，并通过内核的**socket buffer**，包括正在传输的数据。这里有一些特殊的辅助函数，例如 `bpf_skb_load_bytes()`，这个 eBPF 程序可以从套接字缓冲区中读取字节数据。与此同时，另一个由系统调用触发的 eBPF 程序，没有可用的套接字缓冲区，将不允许使用这个辅助函数（what is eBPF chapter2 <sup><a href="#2">[2]</a></sup>）。

eBPF的动态加载

上面也提到了eBPF是一个允许用户自定义程序加载内核功能的框架，也就意味着使用内核无需对内核代码的改变，*what is eBPF chapter2* 中提到的eBPF动态加载，可以理解为触发器与事件，eBFP可以使程序可以动态地加载到内核中和从内核中删除。当附加到事件上的程序遇到对应事件时就会被触发。

eBPF程序

eBPF是事件驱动型程序，它允许在内核中，并挂在到对应的挂载点上，当发生系统调用，函数的进入/退出，内核tracepoints，网络事件等发生，将触发对应程序的操作。

eBPF包含两部分，eBPF程序，eBPF工具

eBPF程序，只运行在内核空间内的代码，这部分仅可以用C或者Rust编写，目前eBPF使用的C语言编写

eBPF工具，是指运行在用户空间内的代码，这部分可以使用任意编程语言编写，例如Python, Go, C 等。

探针

探针 (***probes***) 是指内核定义的扩展点，可以通过eBPF程序将其附加到对应的扩展点上。常用的有`kprobe` 与 `uprobe`

- `kprobe` 是跟踪内核函数调用的探针（k是kernel的前缀），例如 `execve()`
- `kprobe` 是跟踪用户空间程序调用的探针，例如对运行程序 `nginx` 状态的跟踪

程序的加载

用户空间的程序在 eBPF验证器的允许下，可以使用 `bpf()` 系统调用从 ELF 文件中加载 eBPF程序到内核中；一旦将程序加载到内核时，就必须绑定到对应的 ”事件“ 上，每当事件发生时，对应的eBPF程序就会被触发。下面宝海一些常用的事件：

- 函数的进入进出
- Tracepoints，是Linux内核中定义的一些hook可以将 eBPF 程序附加到内核内定义的 `tracepoints` 中
- Perf 是一个收集性能数据的子系统。可以将 eBPF 程序挂到所有收集 perf 数据的地方
- Linux的安全模块 ***LSM*** 例如 ***SELinux*** 和 ***APPAmor*** 使用他们的接口，通过eBPF，可以将程序附加到对应的检查点上。
- 网络接口，***eXpress Data Path*** (XDP) 允许将eBPF程序附加到网络接口上，当收到包时，会触发对应的事件，事件可以检查或者改变一个数据包。
- 套接字和网络钩子，当应用程序在网络套接字上打开或执行其他操作时，以及发出或收到数据包时，你可以附加运行eBPF程序。在内核网络stack中也被叫做 ***traffic control*** (TC)。

eBPF MAP

在一些情况下，我们希望eBPF是从用户空间的应用来接收信息，或者将数据传递给用户空间的应用，允许eBPF程序与用户空间之间传递数据，或者不同的eBPF程序间传递数据的机制被称为 ***MAP***

***MAP*** 是数据类型，本质上是一个 `key-value` 的存储，用于存储不同类型数据的通用数据结构。允许不同eBPF程序之间以及内核和用户空间应用程序之间共享数据。

MAP用途：

- 存储数据，供多eBPF程序信息协调
- eBPF写入事件或指标供用户空间程序检索
- 用户空间配置，供eBPF程序读取并作出相应行为

## eBPF在云原生

在kubernetes中，运行在机器上的Pod共享一个内核，可以通过eBPF检测该机器上运行的应用程序，将eBPF加载到内核并附加到事件之上，就会触发相关事件，而不需要考虑进程与事件的关系。

eBPF 与 sidecar

传统的可观测性应用都使用了 ***sidecar*** 方式进行部署的，这种模式是单独部署一个与应用相同的程序到pod中。

***sidecar***  模式的两大缺点：

- 浪费资源：***sidecar*** 容器都会消耗大量资源，取决于注入的数量
- 安全运行：不能确保每个运行的Pod都被注入

eBPF的隔离

eBPF 检查器可以确保 eBPF 程序只能访问它有权限的内存。检查器检查程序时不可能超出其职权范围，包括确保内存为当前进程所拥有或为当前网络包的一部分。这意味着 eBPF 代码比它周围的内核代码受到更严格的控制，内核代码不需要通过任何类型的检查器。当遇到攻击者通过容器化的应用程序部署到节点上，并且可以提权，那么该攻击程序就可以危害到同一节点上的其他应用程序。当然eBPF检查器可以避免这个问题。

eBPF的应用

eBPF 官网 ebpf.io 中介绍，在通常情况下eBPF不会被单独使用而是通过其他项目在eBPF之上提供一个用户空间工具进行使用，例如 Cilium, bcc等。

eBPF工具

通常情况下eBPF工具都是围绕 网络(***networking***) ，可观测性(***observability***) ，安全 (***security***） 这三个重要方面使用eBPF功能的。

eBPF程序可以连接到网络接口和内核的网络堆栈的各个点（可以通过tracepoint实现）。在每个追踪点上，eBPF程序都可以选择接收/丢弃/操作数据包。基于这个条件下eBPF就可以实现很强大的网络功能，常见的eBPF 实现的网络功能有 负载均衡，分布式防火墙，cni。

- Katran L4 LoadBalancer, facebook开源的4层负载均衡器，使用的eBPF与C++结合的技术
- Cilium eBPF Kubernetes 网络插件

eBPF Tools

BCC

BCC (***BPF Compiler Collection***)  是一组基于eBPF技术用于分析操作系统和网络性能的一组工具。主要提供了以下功能

- 用于观测/追踪 在运行的Linux系统状态工具
- 对于



## 如何编写eBPF程序

eBPF 程序可以用受限的 C 来编写，使用 `clang` 编译器编译成 eBPF 字节码。

> Notes: 受限 C 语言是指省略了一些特性，例如循环，全局变量，可变参数函数，浮点数以及作为函数参数传递的结构。



Go使用 `libbpfgo` 会包装 `libbpf` （一个C语言实现的系统调用库） 的系统调用 `BPF()` 函数，通过加载BPF对象（是通过C语言编写的BPF函数）然后绑定到对应的事件上。

eBPF程序必须使用C编写，通过clang编译为BPF code，然后Go/Python等代码去读取这个文件 `xxx.o` 将其插入到内核中。所以通常情况下，我们会看到是一个Go/Python或者其他语言来包裹C代码来运行的。

一个eBPF程序的结构如下图所示，包含两部分，在用户空间运行的应用程序与运行在内核空间的eBPF程序，用户空间的应用通过系统调用 `BPF()` 来调用运行在内核空间内的eBPF程序的函数。

![image-20220905214855458](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220905214855458.png)

<center>图：eBPF program structure</center>
<center><em>Source：</em>https://files.gotocon.com/uploads/slides/conference_39/1688/original/Beginners%20guide%20to%20eBPF%20with%20Go.pdf</center><br>

eBPF的加载过程如下图所示

![image-20220905214046687](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220905214046687.png)

<center>图：eBPF program loading flow</center>
<center><em>Source：</em>https://files.gotocon.com/uploads/slides/conference_39/1688/original/Beginners%20guide%20to%20eBPF%20with%20Go.pdf</center><br>

一个 eBPF obj 代码在用户空间通过 `bpf()` 系统调用加载到内核中；在内核中首先需要验证器确保eBPF程序是可以安全运行的，如果可以安全运行，将开始在BPF 虚拟机中开始运行

![image-20220905214108075](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220905214108075.png)

<center>图：eBPF program lifetime</center>
<center><em>Source：</em>https://files.gotocon.com/uploads/slides/conference_39/1688/original/Beginners%20guide%20to%20eBPF%20with%20Go.pdf</center><br>

https://github.com/iovisor/bcc/blob/master/docs/reference_guide.md#data

https://man7.org/linux/man-pages/man7/bpf-helpers.7.html

https://gitlab.epfl.ch/debeule/bpf/-/blob/master/LOG.md

https://github.com/lizrice/learning-ebpf/blob/main/chapter5/hello.bpf.c

https://medium.com/@phylake/bottom-up-ebpf-d7ca9cbe8321


> **Reference**
>
> <sup id="1">[1]</sup> [The BSD Packet Filter](https://www.tcpdump.org/papers/bpf-usenix93.pdf)
>
> <sup id="2">[2]</sup> [What Is eBPF?](https://isovalent.com/data/liz-rice-what-is-ebpf.pdf)
>
> <sup id="3">[3]</sup> [how are ebpf programs written](https://ebpf.io/what-is-ebpf/#how-are-ebpf-programs-written)

https://www.youtube.com/watch?v=uBqRv8bDroc

