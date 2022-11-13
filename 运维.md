# 运维类面试题收集


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
cat /dev/urandom | head -1 | md5sum | head -c 32 &gt;&gt; /pass
```

将生成的32位随机数 保存到/pass文件里了

### 统计出nginx的access.log中访问量最多的5个IP

```
cat access_log | awk  &#39;{print $1}&#39; | sort | uniq -c | sort -n -r | head -5
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
- 工作稳定，自身有完整的双机热备方案，如LVS&#43;Keepalived和LVS&#43;Heartbeat；
- 应用范围比较广，可以对所有应用做负载均衡；
- 配置性比较低，这是一个缺点也是一个优点，因为没有可太多配置的东西，所以并不需要太多接触，大大减少了人为出错的几率。

LVS的缺点：

- 软件本身不支持正则处理，不能做动静分离，这就凸显了Nginx/HAProxy&#43;Keepalived的优势。
- 如果网站应用比较庞大，LVS/DR&#43;Keepalived就比较复杂了，特别是后面有Windows Server应用的机器，实施及配置还有维护过程就比较麻烦，相对而言，Nginx/HAProxy&#43;Keepalived就简单一点


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
`apachectl -l|sed -n &#39;/worker\|prefork/p&#39;`

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

### Kubernetes有哪些不同类型的服务？

- cluster ip
- Node Port
- Load Balancer
- Extrenal Name

### 什么是ETCD？

Etcd是用Go编程语言编写的，是一个分布式键值存储，用于协调分布式工作。因此，Etcd存储Kubernetes集群的配置数据，表示在任何给定时间点的集群状态。



### 什么是Ingress网络，它是如何工作的？

[Ingress网络](https://link.zhihu.com/?target=https%3A//www.edureka.co/blog/kubernetes-networking%3Futm_source%3Dmedium%26utm_medium%3Dcontent-link%26utm_campaign%3Dkubernetes-interview-questions%26source%3Dpost_page---------------------------)是一组规则，充当Kubernetes集群的入口点。这允许入站连接，可以将其配置为通过可访问的URL，负载平衡流量或通过提供基于名称的虚拟主机从外部提供服务。因此，Ingress是一个API对象，通常通过HTTP管理集群中服务的外部访问，是暴露服务的最有效方式。

### 什么是Headless Service？

Headless Service类似于“普通”服务，但没有群集IP。此服务使您可以直接访问pod，而无需通过代理访问它。



### 什么是集群联邦？

在联邦集群的帮助下，可以将多个Kubernetes集群作为单个集群进行管理。因此，您可以在数据中心/云中创建多个Kubernetes集群，并使用联邦来在一个位置控制/管理它们。

联合集群可以通过执行以下两项操作来实现此目的。请参考下图。

### kube-proxy的作用

kube-proxy运行在所有节点上，它监听apiserver中service和endpoint的变化情况，创建路由规则以提供服务IP和负载均衡功能。简单理解此进程是Service的透明代理兼负载均衡器，其核心功能是将到某个Service的访问请求转发到后端的多个Pod实例上。

### kube-proxy iptables的原理

Kubernetes从1.2版本开始，将iptables作为kube-proxy的默认模式。iptables模式下的kube-proxy不再起到Proxy的作用，其核心功能：通过API  Server的Watch接口实时跟踪Service与Endpoint的变更信息，并更新对应的iptables规则，Client的请求流量则通过iptables的NAT机制“直接路由”到目标Pod。



### kube-proxy ipvs的原理

IPVS在Kubernetes1.11中升级为GA稳定版。IPVS则专门用于高性能负载均衡，并使用更高效的数据结构（Hash表），允许几乎无限的规模扩张，因此被kube-proxy采纳为最新模式。

在IPVS模式下，使用iptables的扩展ipset，而不是直接调用iptables来生成规则链。iptables规则链是一个线性的数据结构，ipset则引入了带索引的数据结构，因此当规则很多时，也可以很高效地查找和匹配。

可以将ipset简单理解为一个IP（段）的集合，这个集合的内容可以是IP地址、IP网段、端口等，iptables可以直接添加规则对这个“可变的集合”进行操作，这样做的好处在于可以大大减少iptables规则的数量，从而减少性能损耗。



### kube-proxy ipvs和iptables的异同

iptables与IPVS都是基于Netfilter实现的，但因为定位不同，二者有着本质的差别：iptables是为防火墙而设计的；IPVS则专门用于高性能负载均衡，并使用更高效的数据结构（Hash表），允许几乎无限的规模扩张。

与iptables相比，IPVS拥有以下明显优势：

- 为大型集群提供了更好的可扩展性和性能；
- 支持比iptables更复杂的复制均衡算法（最小负载、最少连接、加权等）；
- 支持服务器健康检查和连接重试等功能；
- 可以动态修改ipset的集合，即使iptables的规则正在使用这个集合。

### Kubernetes镜像的下载策略

Kubernetes的镜像下载策略有三种：Always、Never、IFNotPresent。

- Always：镜像标签为latest时，总是从指定的仓库中获取镜像。
- Never：禁止从仓库中下载镜像，也就是说只能使用本地镜像。
- IfNotPresent：仅当本地没有对应镜像时，才从目标仓库中下载。默认的镜像下载策略是：当镜像标签是latest时，默认策略是Always；当镜像标签是自定义时（也就是标签不是latest），那么默认策略是IfNotPresent。

### 简述Kubernetes Scheduler使用哪两种算法将Pod绑定到worker节点

Kubernetes Scheduler根据如下两种调度算法将 Pod 绑定到最合适的工作节点：

- 预选（Predicates）：输入是所有节点，输出是满足预选条件的节点。kube-scheduler根据预选策略过滤掉不满足策略的Nodes。如果某节点的资源不足或者不满足预选策略的条件则无法通过预选。如“Node的label必须与Pod的Selector一致”。
- 优选（Priorities）：输入是预选阶段筛选出的节点，优选会根据优先策略为通过预选的Nodes进行打分排名，选择得分最高的Node。例如，资源越富裕、负载越小的Node可能具有越高的排名。

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
- 2.当监控主机超过300&#43; ,建议使用主动模式

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

