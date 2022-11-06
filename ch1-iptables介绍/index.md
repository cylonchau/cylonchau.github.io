# ch1 iptables介绍


### 1 防火墙实战

**关闭两项功能：**
1. selinux（生产中也是关闭的），ids入侵检测，MD5指纹将。系统所有核心文件全部做指纹识别，将指纹留下，将来出问题，一看就知道那个文件被改过。
2. &amp;nbsp;iptables（生产中看情况，内网关闭，外网打开），大并发的情况，不能开iptables，影响性能。
使用防火墙就不如不使用防火墙，不使用防火墙的前提是不给外网ip，工作中要少给外网服务器ip，这样防火墙使用率较低，防火墙使用也很消耗资源

**安全优化：**

1. 尽可能不给服务器配置外网IP。可以通过代理转发或者通过防火墙映射。
2. 并发不是特别大情况再外网IP的环境，要开启iptables防火墙

http://edu.51cto.com/course/course_id-772.html

**学好iptables基础：**

1. &amp;nbsp;OSI7层模型以及不同层对应那些协议？
2. &amp;nbsp;TCP/IP三次握手，四次断开的过程，TCP HEADER。
3. &amp;nbsp;常用的服务端口要了如指掌。

#### 1.1 iptables防火墙简介

Netfilter/iptables(以下简称iptables)是unix/linux自带的一款优秀且开放源代码的完全自由的基于包过滤的防火墙工具，它的功能十分强大，使用非常灵活，可以对流入和流出的服务器数据包进行很精细的控制。特别是他可以在一台非常低的硬件配置下跑的非常好（赛扬500MHZ 64M内存的情况部署网关防火墙）提供400人的上网服务四号==不逊色企业级专业路由器防火墙==。iptables&#43;zebra&#43;squid

iptables是linux2.4及2.6内核中集成的服务。其功能与安全性比其老一辈ipwadin ipchains强大的多（长江水后浪推前浪），iptables主要工作在OSI七层的二、三、四层，如果重新编译内核，iptables也可以支持7层控制（squid代理&#43;iptables）。

#### 1.2 iptables名词和术语

容器：包含或者说属于关系

**什么是容器？**

​	在iptables里，就是用老描述这种包含或者说属于的关系

**什么是Netfilter/iptables?**

​	Netfilter是表（tables）的容器

**什么是表（tables）？**

​	表是链的容器，所有的链（chains）都属于其对应的表。

**什么是链（chains）？**

​	链（chains）是规则的容器

**什么是规则（policy）**

​	iptables一系列过滤信息的规范和具体方法条款

&lt;center&gt;&lt;b&gt;iptables抽象和实际比喻对比表&lt;/b&gt;&lt;/center&gt;
| Netfilter | tables     | chains       | policy               |
| --------- | ---------- | ------------ | -------------------- |
| 一栋楼    | 楼里的房子 | 房子里的柜子 | 柜子里衣服的摆放规则 |

#### 1.3 iptables工作流程

iptables是采用数据包过滤机制工作的，所以他会对请求的数据包包头数据进行分析，并根据我们预先设定的规则进行匹配来决定是否可以进入主机。

数据包的流向是从左向右的!

![image-20200728221029808](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200728221029808.png)

**iptables工作小结：**

```
防火墙是一层层过滤的。实际是按照配置规则的顺序从上到下，从前到后进行过滤的。
如果匹配上规则，即明确表明是阻止还是通过，此时数据包就不在向下匹配新规则了。
如果所有规则中没有明确表明是阻止还是通过这个数据包，也就是没有匹配上规则，向下进行匹配，知道匹配默认规则得到明确的阻止还是通过。
防火墙的默认规则是对应链的所有的规则执行完才会执行的。
```
***
**提示**：

iptables防火墙规则的执行顺序默认从前到后（从上到下）依次执行，遇到匹配的规则就不在继续向下检查，只有遇到不匹配的规则才会继续向下进行匹配。

***
重点：&lt;font color=&#34;#f8070d&#34; size=2&gt;匹配上了拒绝规则也是匹配&lt;/font&gt;，这点要多注意。
```bash
iptables -A INPUT -p tcp --dport 3306 -j DROP
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
[root@Lnmp_01 ~]# iptables -nL
Chain INPUT (policy ACCEPT) 
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:3306 
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:3306 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination   
```
此时 ``telnet10.0.0.148 3306` 是不通的，原因就是telnet请求已匹配上了拒绝规则 `iptables -A INPUT -p tcp --dport 3306 -j DROP` ,因此，不会在找下面的规则匹配了。如果希望 `telnet 10.0.0.148 3306 `连通，可以吧ACCEPT规则中的-A改为-I，即 `iptables -I INPUT -p tcp --dport 3306 -j ACCEPT` 把允许规则放于INPUT链第一行生效

==默认规则是所有的规则执行完才会执行的==。

#### 1.4  iptables表（tables）和链（chains）

默认情况下，iptables根据功能和表的定义划分包含三个表，filter nat mangle，其每个表有包含不同的操作链（chains）。

**下面的表格展示了表和链的对应关系**

&lt;div style=&#34;text-align:center&#34;&gt;
&lt;table&gt;
       &lt;tr   &gt;
        &lt;td style=&#34;text-align:center; &#34; rowspan=&#34;2&#34; bgcolor=&#39;#aaa9a9&#39;&gt;&lt;strong&gt;表&lt;/strong&gt;&lt;/td&gt;
        &lt;td  style=&#34;text-align:center; &#34; colspan=&#34;5&#34; bgcolor=&#39;#aaa9a9&#39; &gt;&lt;strong&gt;链（chains）&lt;/strong&gt;&lt;/td&gt;
    &lt;/tr&gt;
     &lt;tr&gt;
        &lt;td height=&#34;30px&#34; width=&#34;100px&#34;&gt;INPUT&lt;/td&gt;
       &lt;td height=&#34;30px&#34; width=&#34;130px&#34;&gt;FORWARD&lt;/td&gt;
       &lt;td height=&#34;30px&#34; width=&#34;130px&#34;&gt;OUTPUT&lt;/td&gt;
       &lt;td height=&#34;30px&#34; width=&#34;130px&#34;&gt;PREROUTING&lt;/td&gt;
       &lt;td height=&#34;30px&#34; width=&#34;130px&#34;&gt;POSTROUTING&lt;/td&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
      &lt;td height=&#34;30px&#34; width=&#34;80px&#34;&gt;Filter&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34;&gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34;&gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34;&gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#A9A9A9&#34;&gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#A9A9A9&#34;&gt;&lt;/td&gt;
  &lt;/tr&gt;
    &lt;tr&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34;&gt;NAT&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#A9A9A9&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#A9A9A9&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
    &lt;/tr&gt;
      &lt;tr&gt;
       &lt;td height=&#34;30px&#34; width=&#34;50px&#34;&gt;Mangle&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
      &lt;td height=&#34;30px&#34; width=&#34;50px&#34; style=&#34;background:#2E8B57&#34; &gt;&lt;/td&gt;
    &lt;/tr&gt;
      &lt;tr&gt;
       &lt;td height=&#34;30px&#34; width=&#34;30px&#34; colspan=&#34;6&#34; &gt;注：绿色表示有， 灰色表示无&lt;/td&gt;
    &lt;/tr&gt;
&lt;/table&gt;  
&lt;/div&gt;  
**提示：所有链名要大写**

&lt;table align=&#39;center&#39; style=&#34;border:1px solid #fff;&#34;&gt;
   &lt;tr  align=&#39;center&#39; style=&#34;border:1px solid #fff;&#34; &gt;
       &lt;td height=&#34;50px&#34; width=&#34;100px&#34; style=&#34;text-align:center;&#34; &gt;&lt;strong&gt;表名&lt;/strong&gt;&lt;/td&gt;
        &lt;td  style=&#34;text-align:center; &#34;&gt;&lt;strong&gt;作用&lt;/strong&gt;&lt;/td&gt;
    &lt;/tr&gt;
     &lt;tr&gt;
       &lt;tr  &gt;
        &lt;td style=&#34;text-align:center; &#34;&gt;filter&lt;/td&gt;
        &lt;td  &gt;强调主要和主机自身有关，真正负责主机防火墙功能的（过滤流入流出主机的数据包）。filter表是iptables默认使用的表。这个表定义了三个链（chains）。&lt;strong&gt;企业工作场景：主机防火墙。&lt;/strong&gt;&lt;/td&gt;
    &lt;/tr&gt;
     &lt;tr&gt;
        &lt;td height=&#34;50px&#34; width=&#34;50px&#34; style=&#34;text-align:center; &#34; &gt;nat&lt;/td&gt;
       &lt;td  height=&#34;50px&#34;&gt;负责网络地址转换，即来源与目的IP地址和port的转换。应用：和主机本身无关。&lt;strong&gt;一般用于局域网共享上网或者特殊的端口转换服务相关。&lt;/strong&gt;&lt;br&gt;&lt;br&gt;
NAT功能一般企业工作场景&lt;br&gt;
1. 用于做企业路由（zebra）或网关（iptables），共享上网（POSTROUTING）&lt;br&gt;
2. 做内部外部IP地址一对一映射（dmz），硬件防火墙映射IP到内部服务器，ftp服务，（PREROUTING）&lt;br&gt;
3. web，单个端口的映射，直接映射80端口（PREROUTING）。这个表示定义了三个链（chains），nat功能就相当于网络的acl控制。和网络交换机acl类似。&lt;/td&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;td height=&#34;50px&#34; width=&#34;50px&#34; style=&#34;text-align:center; &#34;&gt;Mangle&lt;/td&gt;
       &lt;td  height=&#34;50px&#34;&gt;主要负责修改数据包中特殊的路由标记，如TTL，TOS，NARK等。这个表定义了5个链&lt;/td&gt;
    &lt;/tr&gt;
&lt;/table&gt;  


&lt;table align=&#39;center&#39; style=&#34;border:1px solid #fff;&#34;&gt;
   &lt;tr  align=&#39;center&#39; style=&#34;border:1px solid #fff;&#34; &gt;
       &lt;td height=&#34;50px&#34; width=&#34;150px&#34; style=&#34;text-align:center; &#34; &gt;&lt;strong&gt;链名&lt;/strong&gt;&lt;/td&gt;
        &lt;td  style=&#34;text-align:center; &#34;&gt;&lt;strong&gt;作用&lt;/strong&gt;&lt;/td&gt;
    &lt;/tr&gt;
     &lt;tr&gt;
       &lt;tr  &gt;
        &lt;td style=&#34;text-align:center; &#34; height=&#34;50px&#34; width=&#34;50px&#34; &gt;INPUT&lt;/td&gt;
        &lt;td  &gt;负责过滤所有目标地址是本机地址的数据包。通俗的讲，就是过滤进入主机的数据包&lt;/td&gt;
    &lt;/tr&gt;
     &lt;tr&gt;
        &lt;td  style=&#34;text-align:center; &#34; &gt;FORWAED&lt;/td&gt;
       &lt;td &gt;负责转发流经主机的数据包。起转发的作用，和NAT关系很大，后面会详细讲LVS NAT模式。&lt;/td&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;td style=&#34;text-align:center; &#34;&gt;PREROUTING&lt;/td&gt;
       &lt;td &gt;在&lt;strong&gt;数据包到达防火墙时进行路由判断之前执行的规则&lt;/strong&gt;，作用是改变数据包的目的地址、目的端口等。（通俗比喻，就是收信时，根据规则重写收件人的地址，这看上去很不地道啊！）
例如：把公网IP：124.42.60.113映射到居于玩分10.0.0.19服务器上。如果是web服务，可以把80端转为局域网的服务器上9000端口。&lt;/td&gt;
    &lt;/tr&gt;
   &lt;tr&gt;
        &lt;td style=&#34;text-align:center; &#34;&gt;POSTROUTING&lt;/td&gt;
       &lt;td &gt;在数据包离开防火墙时进行路由判断之后执行的规则，作用改变数据包的源地址、源端口等。（通俗比喻，就是寄信时，写好发件人的地址，要让人家回信时能后有地址可回。）例如：我们在现在的笔记本和虚拟机都是10.0.0.0/24，就是楚王的时候被我们企业路由器把源地址改成了公网地址了。&lt;strong&gt;生产应用：局域网共享上网。&lt;/strong&gt;&lt;/td&gt;
    &lt;/tr&gt;
&lt;/table&gt;  

由于这个表与特殊标记相关，一般情况下，我们用不到这个mangle表，这里就不做详细介绍了。给初学者的建议：新手学习时最好抓住一个主线向前学，能够跑通路就好，不一定要面面俱到，不然很容易陷进去，而苦恼，甚至失去学习的兴趣

#### 1.5  iptables表和链的流程图

下面这张图清晰的描绘了netfilter对包的处理流程


![image-20221025001126693](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025001126693.png)

**为了更好的学习将上图简化为如下**

![image-20221025002458322](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025002458322.png)

&lt;center&gt;&lt;strong&gt;图1-1 iptables简化流程图&lt;/strong&gt;&lt;/center&gt;
强调：上图可以看作地铁1 2号线来

1号线：主要是NAT功能，

​	**企业案例：**

​		1. 局域网上网共享（路由和网关），使用NAT的POSTROUTING链

​		2. 外部IP和端口映射为内部IP和端口（DMZ功能），使用NAT的PREROUTING链。

2号线：主要是FILTER功能，即防火墙功能 FILTER INPUT FORWARD

​	**企业案例：**

​		主要应用就是主机服务器防火墙，使用FILTER的INPUT链

