# 

## Linux 基础



### BASH和DOS控制台之间的主要区别在于3个方面：

**答案：**

- BASH命令区分大小写，而DOS命令则不区分;
- 在BASH下，/ character是目录分隔符，\ 作为转义字符。在DOS下，/ 用作命令参数分隔符，\ 是目录分隔符
- DOS遵循命名文件中的约定，即8个字符的文件名后跟一个点，扩展名为3个字符。BASH没有遵循这样的惯例。
  

### Linux 中进程有哪几种状态？在 ps 显示出来的信息中，分别用什么符号表示的？

**答案：**

- R runnable (on run queue) 运行 (正在运行或在运行队列中等待)

- S 中断 sleeping(休眠中, 受阻, 在等待某个条件的形成或接受到信号)

- D 不可中断 uninterruptible sleep (usually IO)(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)

-  Z 僵尸 a defunct (”zombie”) process (进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)

-  T 停止 traced or stopped (进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行) 

  

 

### 系统目前有许多正在运行的任务，在不重启机器的条件下，有什么方法可以把所有正在运行的进程移除呢？

答案：使用linux命令 ’disown -r ’可以将所有正在运行的进程移除。

### Linux 粘滞位作用

**答案：**

粘滞位(sticky bit)权限是针对目录的，对文件无效，设置了sticky位表示这个目录里的文件只能被owner和root删除。



### 什么是inode？

**答案：**

inode是Linux(Unix)操作系统中文件系统的一个概念。inode的全称为index node，也就是索引节点。那么inode是用来索引什么的呢？其实inode表示的是一个文件，它是用来索引文件数据的。



### 磁盘报错”No space left on device”，但是通过命令df –h查看磁盘空间没有满，请问为什么？

**答案：**

该磁盘的inode数量被用尽，无法再写入文件。
企业工作中邮件临时队列 `/var/spool/clientmquene`或`/var/spool/postfix/maildrop`这里很容易被大量小文件占满导致`No space left on device`的错误。`clientmquene`目录只有安装了`sendmail`服务，才会有，是`sendmail`的临时队列。



### 一个100M的磁盘分区，分别写入1K的文件，及写入1M的文件，分别可以写多少个？

**答案：**

在linux文件系统中，iNode用来存放文件的属性信息，而Block用来存放文件实际内容，默认大小1K(boot)或4K(非系统分区默认为4k)。

写入1M文件的数量为100/1，且不会存在磁盘浪费情况（这也说明了一般情况下，inode和block的数量都是足够的）；

而写入1K文件时，inode和block同时被消耗，但一般block数量远大于inode的数量，因此写入的数量就是inode的数量，并且这样会浪费3/4的磁盘容量。







### 系统调用与库函数的区别？

系统调⽤：操作系统为⽤户程序与硬件设备进⾏交互提供的⼀组接⼝，发⽣在内核地址空间。

库函数：把⼀些常⽤的函数编写完放到⼀个⽂件⾥，编写应⽤程序时调⽤，这是由第三⽅提供的，发⽣在⽤户地址空间。

在移植性⽅⾯，不同操作系统的系统调⽤⼀般是不同的，移植性差；⽽在所有的ANSIC编译器版本中，C库函数是相同的。

在调⽤开销⽅⾯，系统调⽤需要在⽤户空间和内核环境间切换，开销较⼤；⽽库函数调⽤属于“过程调⽤”，开销较⼩。

***简单点说，库函数就是系统调⽤的上层封装，为了让应⽤程序在使⽤时更加⽅便。***





### 阻塞IO、⾮阻塞IO、同步IO、异步IO的区别？

在Richard Stevens的《UNIX Network Programming Volume 1, Third Edition: The Sockets Networking》的第6.2节介绍了五种IO Model，并说明了各种IO的特点和区别。

- blocking IO
- non-blocking IO
- IO multiplexing
- signal driven IO（不讨论）
- asynchronous IO

【答】：

当⼀个⽹络IO的 read 操作发⽣时，它会经历两个阶段：

- 等待数据准备 (Waiting for the data to be ready)
- 将数据从内核拷⻉到进程中 (Copying the data from the kernel to the process)

这些IO Model的区别就是在两个阶段上各有不同的情况:

- 阻塞IO（blocking IO）：线程阻塞以等待数据，然后将数据从内核拷⻉到进程，返回结果之后才解除阻塞状态。也就是说两个阶段都被block了。
- ⾮阻塞IO（non-blocking IO）：当对⼀个⾮阻塞socket执⾏读操作时，如果kernel中的数据还没有准备好，那么它并不会block⽤户进程，⽽是⽴刻返回⼀个error。⽤户进程需要不断地主动进⾏read操作，⼀旦数据准备好了，就会把数据拷⻉到⽤户内存。也就是说，第⼀阶段并不会阻塞线程，但第⼆阶段拷⻉数据还是会阻塞线程。
- IO复⽤（IO multiplexing）：这种IO⽅式也称为event driven IO. 通过使⽤select/poll/epoll在单个进程中同时处理多个⽹络连接的IO。例如，当⽤户进程调⽤了select，那么整个进程会被block，通过不断地轮询所负责的所有socket，当某个socket的数据准备好了，select就会返回。这个时候⽤户进程再调⽤read操作，将数据从kernel拷⻉到⽤户进程。在IO复⽤模型中，实际上对于每⼀个socket，⼀般都设置成为non-blocking，但是，整个⽤户进程其实是⼀直被block的，先是被select函数block，再是被socket IO第⼆阶段block。
- 同步IO（synchronous IO）：POSIX中的同步IO定义是—— A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes。也就是说同步IO在IO操作完成之前会阻塞线程，按照这个定义，之前所述的blocking IO，non-blocking IO，IO multiplexing都属于synchronous IO。（non-blocking IO也属于同步IO是因为它在真正拷⻉数据时也会阻塞线程）。
- 异步IO（asynchronous IO）：POSIX中的异步IO定义是—— An asynchronous I/O operation does not cause the requesting process to be blocked。在linux异步IO中，⽤户进程发起read操作之后，直接返回，去做其它的事。⽽另⼀⽅⾯，从kernel的⻆度，当它受到⼀个asynchronous read之后，kernel会等待数据准备完成，然后将数据拷⻉到⽤户内存，当这⼀切都完成之后，kernel会给⽤户进程发送⼀个signal，告诉它read操作完成了。也就是说两个阶段都不会阻塞线程。它就像是⽤户进程将整个IO操作交给了他⼈（kernel）完成，然后他⼈做完后发信号通知。在此期间，⽤户进程不需要去检查IO操作的状态，也不需要主动的去拷⻉数据。



### 分时系统与实时系统的区别？

【答】：

- 分时系统：系统把CPU时间分成很短的时间⽚，轮流地分配给多个作业。优点：对多个⽤户的多个作业都能保证⾜够快的响应时间，并且有效提⾼了资源的利⽤率。
- 实时系统：系统对外部输⼊的信息，能够在规定的时间内（截⽌期限）处理完毕并做出反应。优点：能够集中地及时地处理并作出反应，⾼可靠性，安全性。

### 除了sleep之外，usleep()也是linux系统调⽤，它号称⾃⼰是微秒级的，你相信它真的有这么快吗？为什么？



【答】：usleep实际上达不到微秒级，主要是因为linux是分时系统，由于进程调度（上下⽂切换）和系统时间中断精度的原因。













### 死锁产⽣的四个条件，死锁发⽣后怎么检测和恢复？
【答】

死锁的四个必要条件：

- 互斥条件：⼀个资源每次只能被⼀个进程使⽤。
- 请求与保持条件：⼀个进程在申请新的资源的同时保持对原有资源的占有。不剥夺条件:进程已获得的资源，在未使⽤完之前，不能强⾏剥夺。
- 循环等待条件:若⼲进程之间形成⼀种头尾相接的循环等待资源关系。

死锁发⽣后：
  - 检测死锁：⾸先为每个进程和每个资源指定⼀个唯⼀的号码，然后建⽴资源分配表和进程等待表。
  - 解除死锁：当发现有进程死锁后，可以直接撤消死锁的进程或撤消代价最⼩的进程，直⾄有⾜够的资源可⽤，死锁状态消除为⽌。



### 内存泄漏怎么产⽣的？如何避免？

【答】

我们所说的内存泄漏⼀般是指堆内存的泄漏，也就是程序在运⾏过程中动态申请的内存空间不再使⽤后没有及时释放，导致那块内存不能被再次使⽤。

更⼴义的内存泄漏还包括未对系统资源的及时释放，⽐如句柄、socket等没有使⽤相应的函数释放掉，导致系统资源的浪费。

解决⽅法：

- 养成良好的编码习惯和规范，记得及时释放掉内存或系统资源。
- 重载new和delete，以链表的形式⾃动管理分配的内存。
- 使⽤智能指针。



### 如何在⼀个不安全的环境中实现安全的数据通信？

要实现数据的安全传输，当然就要对数据进⾏加密了。



如果使⽤对称加密算法，加解密使⽤同⼀个密钥，除了⾃⼰保存外，对⽅也要知道这个密钥，才能对数据进⾏解密。如果你把密钥也⼀起传过去，就存在密码泄漏的可能。所以我们使⽤⾮对称算法，过程如下：

- ⾸先 接收⽅ ⽣成⼀对密钥，即私钥和公钥；
- 然后，接收⽅ 将公钥发送给 发送⽅；
- 发送⽅⽤收到的公钥对数据加密，再发送给接收⽅；
- 接收⽅收到数据后，使⽤⾃⼰的私钥解密。

由于在⾮对称算法中，公钥加密的数据必须⽤对应的私钥才能解密，⽽私钥⼜只有接收⽅⾃⼰知道 ，这样就保证了数据传输的安全性。



### 堆和栈的区别？

【答】

栈由编译器⾃动分配释放，存放函数参数、局部变量等。⽽堆由程序员⼿动分配和释放；

栈是向低地址扩展的数据结构，是⼀块连续的内存的区域。⽽堆是向⾼地址扩展的数据结构，是不连续的内存区域；

栈的默认⼤⼩为1M左右，⽽堆的⼤⼩可以达到⼏G，仅受限于计算机系统中有效的虚拟内存。



### 数据库的三范式？

【答】

- 1NF：字段不可分（原⼦性）

- 2NF：有主键，⾮主键字段依赖主键（唯⼀性）
- 3NF：⾮主键字段不能相互依赖（每列都与主键有直接关系，不存在传递依赖）



## 如何优化 Linux系统（笼统）

- 不用root，添加普通用户，通过sudo授权管理
- 更改默认的远程连接SSH服务端口及禁止root用户远程连接
- 定时自动更新服务器时间
- 配置国内yum源
- 关闭selinux及iptables（iptables工作场景如果有外网IP一定要打开，高并发除外）
- 调整文件描述符的数量
- 精简开机启动服务（crond rsyslog network sshd）
- 内核参数优化（/etc/sysctl.conf）
- 更改字符集，支持中文，但建议还是用英文字符集，防止乱码
- 锁定关键系统文件
- 清空/etc/issue，去除系统及内核版本登录前的屏幕显示

### 基础命令

### ps aux 中的VSZ代表什么意思，RSS代表什么意思
- VSZ:虚拟内存集,进程占用的虚拟内存空间
- RSS:物理内存集,进程战用实际物理内存空间

### shell下32位随机密码生成

```
cat /dev/urandom | head -1 | md5sum | head -c 32 >> /pass
```
将生成的32位随机数 保存到/pass文件里了

### 统计出nginx的access.log中访问量最多的5个IP

```
cat access_log | awk  '{print $1}' | sort | uniq -c | sort -n -r | head -5
```

## web与lb

## 讲述一下LVS三种模式的工作过程

### LVS负载的原理，和Nginx负载有啥区别

- **VS/NAT：（Virtual Server via Network Address Translation）**

  也就是网络地址翻译技术实现虚拟服务器，当用户请求到达调度器时，调度器将请求报文的目标地址（即虚拟IP地址）改写成选定的Real Server地址，同时报文的目标端口也改成选定的Real Server的相应端口，最后将报文请求发送到选定的Real Server。在服务器端得到数据后，Real Server返回数据给用户时，需要再次经过负载调度器将报文的源地址和源端口改成虚拟IP地址和相应端口，然后把数据发送给用户，完成整个负载调度过程。

  可以看出，在NAT方式下，用户请求和响应报文都必须经过Director Server地址重写，当用户请求越来越多时，调度器的处理能力将称为瓶颈。

- **VS/TUN ：即（Virtual Server via IP Tunneling）**

  也就是IP隧道技术实现虚拟服务器。它的连接调度和管理与VS/NAT方式一样，只是它的报文转发方法不同，VS/TUN方式中，调度器采用IP隧道技术将用户请求转发到某个Real Server，而这个Real Server将直接响应用户的请求，不再经过前端调度器，此外，对Real Server的地域位置没有要求，可以和Director Server位于同一个网段，也可以是独立的一个网络。因此，在TUN方式中，调度器将只处理用户的报文请求，集群系统的吞吐量大大提高。

- **VS/DR： 即（Virtual Server via Direct Routing）**

  也就是用直接路由技术实现虚拟服务器。它的连接调度和管理与VS/NAT和VS/TUN中的一样，但它的报文转发方法又有不同，VS/DR通过改写请求报文的MAC地址，将请求发送到Real Server，而Real Server将响应直接返回给客户，免去了VS/TUN中的IP隧道开销。这种方式是三种负载调度机制中性能最高最好的，但是必须要求Director Server与Real Server都有一块网卡连在同一物理网段上。

回答负载调度算法，IPVS实现在八种负载调度算法，我们常用的有四种调度算法（轮叫调度、加权轮叫调度、最少链接调度、加权最少链接调度）。一般说了这四种就够了，也不会需要你详细解释这四种算法的。你只要把上面3种负载均衡技术讲明白面试官就对这道问题很满意了。

### lvs与nginx的区别：

LVS的优点：

- 抗负载能力强、工作在第4层仅作分发之用，没有流量的产生，这个特点也决定了它在负载均衡软件里的性能最强的；无流量，同时保证了均衡器IO的性能不会受到大流量的影响；
- 工作稳定，自身有完整的双机热备方案，如LVS+Keepalived和LVS+Heartbeat；
- 应用范围比较广，可以对所有应用做负载均衡；
- 配置性比较低，这是一个缺点也是一个优点，因为没有可太多配置的东西，所以并不需要太多接触，大大减少了人为出错的几率。

LVS的缺点：

- 软件本身不支持正则处理，不能做动静分离，这就凸显了Nginx/HAProxy+Keepalived的优势。
- 如果网站应用比较庞大，LVS/DR+Keepalived就比较复杂了，特别是后面有Windows Server应用的机器，实施及配置还有维护过程就比较麻烦，相对而言，Nginx/HAProxy+Keepalived就简单一点


Nginx的优点：

- 工作在OSI第7层，可以针对http应用做一些分流的策略。比如针对域名、目录结构。它的正则比HAProxy更为强大和灵活；
- Nginx对网络的依赖非常小，理论上能ping通就就能进行负载功能，这个也是它的优势所在；
- Nginx安装和配置比较简单，测试起来比较方便；
- 可以承担高的负载压力且稳定，一般能支撑超过几万次的并发量；
- Nginx可以通过端口检测到服务器内部的故障，比如根据服务器处理网页返回的状态码、超时等等，并且会把返回错误的请求重新提交到另一个节点；
- Nginx不仅仅是一款优秀的负载均衡器/反向代理软件，它同时也是功能强大的Web应用服务器。LNMP现在也是非常流行的web环境，大有和LAMP环境分庭抗礼之势，Nginx在处理静态页面、特别是抗高并发方面相对apache有优势；


Nginx的缺点：

- Nginx不支持url来检测。
- Nginx仅能支持http和Email，这个它的弱势。
- Nginx的Session的保持，Cookie的引导能力相对欠缺。

### apache工作模式

查看apache工作模式
`apachectl -l|sed -n '/worker\|prefork/p'`

- prefork使用的是多个子进程，而每个子进程只有一个线程，每个进程在某个确定的时间只能维持一个连接.

- worker模式是Apache2.X新引进来的模式，是线程与进程的结合，在worker模式下会有多个子进程，每个进程又会有多个线程。每个线程在某个确定的时间只能维持一个连接。

- event模式：event和 worker模式很像，最大的区别在于，它解决了`keep-alive`场景下 ，长期被占用的线程的资源浪费问题。

  `event` MPM中，会有一个专门的线程来管理这些`keep-alive`类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放。这样，一个线程就能处理几个请求了，实现了异步非阻塞。
  
  `event` MPM在遇到某些不兼容的模块时，会失效，将会回退到worker模式，一个工作线程处理一个请求。官方自带的模块，全部是支持`event`MPM的。

优缺点

- 优点：内存占用比prefork模式低，适合高并发高流量HTTP服务。

- 缺点：假如一个线程崩溃，整个进程就会连同其任何线程一起“死掉”.由于线程贡献内存空间，所以一个程序在运行时必须被系统识别为“每个线程都是安全的”服务稳定性不如prefork模式。

https://github.com/PlutoaCharon/LiunxNotes/blob/master/Liunx%E9%9D%A2%E8%AF%95%E7%AC%94%E8%AE%B0/Linux%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86%E7%B1%BB/Apache.md

### kubernetes

- 

## zabbix

###  简述Zabbix-proxy使用场景

- 监控远程位置，解决跨机房
- 监控主机多，性能跟不上，延迟大
- 解决网络不稳定

### zabbix 是怎么实施监控的

agentd需要安装到被监控的主机上，它负责定期收集各项数据，并发送到zabbix server端，zabbix server将数据存储到数据库中，zabbix web根据数据在前端进行展现和绘图。这里agentd收集数据分为主动和被动两种模式：

主动：agent请求server获取主动的监控项列表，并主动将监控项内需要检测的数据提交给server/proxy

被动：server向agent请求获取监控项的数据，agent返回数据。

- 主动模式被动模式：默认为zabbix-agent被动模式

- 1.被动模式（zabbix-server轮询检测zabbix-agent）
- 2.主动模式（zabbix-agent主动上报给zabbix-server）优

- 1.当（Queue）队列中有大量的延迟监控项
- 2.当监控主机超过300+ ,建议使用主动模式

### zabbix 怎么开启自定义监控

1、写一个脚本用于获取待监控服务的一些状态信息。

2、在zabbix客户端的配置文件zabbix_agentd.conf中添加上自定义的“UserParameter”，目的是方便zabbix调用我们上面写的那个脚本去获取待监控服务的信息。

3、在zabbix服务端使用zabbix_get测试是否能够通过第二步定义的参数去获取zabbix客户端收集的数据。

4、在zabbix服务端的web界面中新建模板，同时第一步的脚本能够获取什么信息就添加上什么监控项，“键值”设置成前面配置的“UserParameter”的值。

5、数据显示图表，直接新建图形并选择上一步的监控项来生成动态图表即可。





## iptables



### iptables四表五链（必问题）



表：

- nat 用于网络地址解析
- mangle mangle包， 特定数据包的更改，好比head,content
- filter default表，用于过滤包
- raw 优先级最高，用于pretouting和output ,使用raw表，能够跳过NAT表和ip_conntrack处理,即再也不作地址转换和数据包的连接跟踪处理了

链：

- prerouting,

- input 
- forword
- output
- postrouting

### iptables和firewalld的基本区别是什么呢？

在linux中，防火墙的概念是 Linux 内核网络堆栈中的网络数据包进行操作的规则集合，而iptables则是这些用来操作这些规则的**用户空间**命令行工具，通过将规则格式化为内核数据结构转发为内核。



firewalld是fedora的一个开源项目，是iptables/nftables命令的封装，实现了更多动态防火墙的概念，并且加入更多用户鉴权，GUI管理等。



类似于http是tcp的wapper，firewalld则是iptables的wapper。



### 什么是iptables中的目标值（能被指定为目标），他们有什么用

下面是在iptables中可以指定为目标的值：

- ACCEPT : 接受包
- QUEUE : 将包传递到用户空间 (应用程序和驱动所在的地方)
- DROP : 丢弃包
- RETURN : 将控制权交回调用的链并且为当前链中的包停止执行下一调用规则


