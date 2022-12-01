# ch2 iptables命令帮助信息


## 2 iptables命令帮助信息

有问题查帮助，下面是很全的帮助信息（必须拿下它）

```bash
$ iptables -h
```

```bash
$ iptables -nL
# INPUT链 	ACCEPT默认允许决策              
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination  
```

### 2.1 启动和查看iptables状态

```bash
/etc/init.d/iptables start
systemctl start iptables
```

> **实例演示1：**

```bash
$ /etc/init.d/iptables status
表格：filter
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
2ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
num  target     prot opt source               destination         
$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
```

***
**提示：如果遇到下面的无法启动IPTABLES的情况**
***

```bash
$ /etc/init.d/iptables start
$ /etc/init.d/iptables start
$ /etc/init.d/iptables status
Firewall is stopped
```

***
**解决：setup -> Firewall Configuration -> enable**
***

**Iptables默认加载的内核模块**
```bash
$ lsmod|egrep "nat|filter"   
iptable_filter          2793  1 
ip_tables              17831  1 iptable_filter
```

**加载如下模块到linux内核**
```bash
modprobe ip_tables \
modprobe iptable_filter \
modprobe iptable_nat \
modprobe ip_conntrack \
modprobe ip_conntrack_ftp \
modprobe ip_nat_ftp \
modprobe ipt_state
```

### 2.2 iptables参数

| **参数选项**    | **注释说明**                                 |
| --------------- | -------------------------------------------- |
| -n *num*        | 数字                                         |
| -L              | 列表                                         |
| -F              | 清除所有规则，不会处理默认的规则             |
| -X              | 删除用户自定义的链                           |
| -Z              | 链的计数器清零                               |
| -t              | 指定表                                       |
| -A              | 添加协议                                     |
| -p              | 协议（all tcp udp Icmp）默认为all            |
| --dport         | 目的端口                                     |
| --sport         | 源端口                                       |
| -j              | 处理的行为（ACCEPT接受 DROP丢弃 REJECT拒绝） |
| -D              | 删除规则                                     |
| -A              | 添加规则到指定链的结尾                       |
| -I              | 添加规则到指定链的开头                       |
| -s              | 指定源地址                                   |
| --line-numbers  | 显示序号                                     |
| -i *<网络接口>* | 指定数据包进入本机的网络接口。               |
| -o *<网络接口>* | 指定数据包要离开本机所使用的网络接口。       |

### 2.3 清除默认规则

- `iptables -F` 清除所有规则，不会处理默认的规则
- `iptables -X` 删除用户自定义的链
- `iptables -Z` 链的计数器清零

**实例演示2：**

```bash
iptables -F == iptables --flush
iptables -X == iptables --delete-chain
iptables -Z
```
> **禁止规则**

**1.禁止ssh端口**

(1) 找出当前机器SSH端口

```bash
$ netstat -lntup|grep ssh
tcp     0      0 192.168.1.5:52113       0.0.0.0:*         LISTEN      1053/sshd 
```

(2) 禁止掉当前SSH端口，这里是52113

```bash
iptables -t [table] -[AD] chain rule-specification [options]
```

**具体命令**

```bash
iptables -A INPUT -p tcp --dport 52113 -j DROP
iptables -tfilter -A INPUT -p tcp --dport 52113 -j DROP	
```
注：

```txt
1.  iptables默认用的就是filter表，因此，以上两条命令等价。
2.  其中INPUT DROP要大写
3.  --jump  -j target target for rule（may load target extension）基本处理行为：ACCEPT（接受）、DROP（丢弃）、REJECT（拒绝）。比较：DROP好于REJECT（不要给reject，拒绝会给对方信息，透漏信息了）
```

命令行执行的规则，仅仅在内存里临时生效。

```bash
$ iptables -A INPUT -p tcp --dport 52113 -j DROP 
$ 
ÐÅºÅµÆ³¬Ê±Ê
```

打台球：如果对方告诉你不去，REJECT（拒绝），如果对方没反应，DROP（丢弃）。

(3) 恢复刚才断掉的SSH连接

```txt
1.  去机房重启系统或者登陆服务器删除刚才的禁止规则。
2.  让机房人员重启服务器或让机房人员拿用户密码登陆进去。
3.  通过服务器的远程管理卡管理（推荐）。
4.  先写一个定时任务，每5分钟就停止防火墙。
5.  测试环境测试号，写成脚本，批量执行。
```

我们恢复的办法，登陆虚拟终端页面删除掉刚才的规则。当然也可执行iptables -F， iptables stop等。
练习：禁止用户访问80端口或3306端口：

```bash
iptables -t filter -A INPUT -p tcp --dport 80 -j DROP
$ telnet 192.168.1.5 80
Trying 192.168.1.5...
Connected to 192.168.1.5.
Escape character is '^]'.
^CConnection closed by foreign host.
$ telnet 192.168.1.5 80
Trying 192.168.1.5...

$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:80 
```

使用-I和-A的顺序，防火墙的过滤根据规则顺序的。

- `-A` 是添加规则到指定链的结尾，最后一条。

- `-I` 是添加规则到指定链的结尾，第一条。

```bash
$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination   
```


```bash
$ iptables -A INPUT -p tcp -s 192.168.1.1 --dport 80 -j DROP
$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  192.168.1.1          0.0.0.0/0           tcp dpt:80 
```
**查看规则序号：**

```bash
iptables -L -n --line-numbers
$ iptables -L -n --line-numbers                             
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  192.168.1.1          0.0.0.0/0           tcp dpt:80 
2    DROP       tcp  --  192.168.1.2          0.0.0.0/0           tcp dpt:80 
Chain FORWARD (policy ACCEPT)

```
指定位置插入规则：插入到第二行

```bash
$ iptables -I INPUT 2 -p tcp -s 192.168.1.3 --dport 80 -j DROP 
$ iptables -L -n --line-numbers                               
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  192.168.1.1          0.0.0.0/0           tcp dpt:80 
2    DROP       tcp  --  192.168.1.3          0.0.0.0/0           tcp dpt:80 
3    DROP       tcp  --  192.168.1.2          0.0.0.0/0           tcp dpt:80 
Chain FORWARD (policy ACCEPT)
```

通过序号删除规则，删除上述第2条规则

```bash
$ iptables -D INPUT 2 == delete from iptables where id=2
$ iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  192.168.1.1          0.0.0.0/0           tcp dpt:80 
2    DROP       tcp  --  192.168.1.2          0.0.0.0/0           tcp dpt:80 
Chain FORWARD (policy ACCEPT)
```

**小结：总结删除规则的方法：**

```
1.   iptables -D INPUT -P tcp --dport 8080 -J DROP
2.   iptables -F 删除所有规则
3.   /etc/init.d/iptables restart （用iptables命令行配置的命令都是临时生效）
4.   iptables -D INPUT 序列号
```

**基于客户端源地址网段控制，禁止10.0.0.0网段连入**

```bash
iptables -t filter -A INPUT -i eth0 -s 10.0.0.0/24 -J DROP
iptables -A INPUT -i eth0 -s 10.0.0.0/24 -J DROP
```

注：iptables默认用的就是filter表，因此以上两条命令等价。
执行以上命令可以发现，我这里已经无法远程连接了。
登陆虚拟机，删除刚才禁止的来源地址为10网段的命令。
`iptables -D INPUT -i eth0 -s 10.0.0.0/24 -J DROP` (完整策略规则删除)
`iptables -D INPUT 1`（根据策略在链中的序号删，每条链都是各自从1编号）。

```bash
$ iptables -nL --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  192.168.1.5          0.0.0.0/0           tcp dpt:80 
2    DROP       tcp  --  192.168.1.4          0.0.0.0/0           tcp dpt:80 
3    DROP       tcp  --  192.168.1.2          0.0.0.0/0           tcp dpt:80 

Chain FORWARD (policy ACCEPT)
n      
$ iptables -D INPUT 1
$ iptables -nL --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  192.168.1.4          0.0.0.0/0           tcp dpt:80 
2    DROP       tcp  --  192.168.1.2          0.0.0.0/0           tcp dpt:80 

Chain FORWARD (policy ACCEPT)   
```

> **还可以通过“！”来取反**

```bash
# 只有 192.168.1.1才可访问80端口 ！放在选项的前面而不是参数的前面
iptables -I INPUT -p tcp ! -s 192.168.1.1 --dport 80 -j DROP
```

测试配置拒绝规则也是匹配：下面的测试有两个要点：非的作用，匹配拒绝也是匹配。

centos5版本

```bash
iptables -I INPUT -p tcp -s ! 192.168.1.1 --dport 80 -j DROP
```

centos6.4高版本：

```bash
$ iptables -t filter -A INPUT -i eth0 -s ! 10.0.0.115 -j DROP
Using intrapositioned negation (`--option ! this`) is deprecated in favor of extrapositioned (`! --option this`).
# 解决方案： iptables -t filter -A INPUT -i eth0 -s ! 10.0.0.115 -j DROP
```

> **测试非”！“**

1.源地址不是10.0.0.101单个IP的禁止链接

```
iptables -t filter -I INPUT -i eth0 ! -s 10.0.0.101 -j DROP
iptables -A INPUT -p all -i eth0 ! -s 10.0.0.106 -j DROP # p(udp tcp icmp all)
# 不让主机ping通
iptables -t filter -I INPUT -p icmp --icmp-type 8 -i eth0 -s ! 192.168.2.83 -j DROP

# ssh 断开链接
$ iptables -t filter -I INPUT -i eth0 ! -s 192.168.2.83 -j DROP
$ 
ÐÅºÅµÆ³¬Ê±Ê
# ping 不通
C:\Users\Company>ping 192.168.2.83
正在 Ping 192.168.2.83 具有 32 字节的数据:
请求超时。
请求超时。
请求超时。
192.168.2.83 的 Ping 统计信息:
数据包: 已发送 = 4，已接收 = 0，丢失 = 4 (100% 丢失)，
```
2.原地址不是192.168.2.0/24的网段禁止连接

```bash
 iptables -t filter -I INPUT -i eth0 -s ! 192.168.2.0/24 -j DROP  ==  
 iptables -t filter -I INPUT -i eth0 -s 192.168.2.0/24 -j ACCECT # 工作场景
```
第一节讲了linux优化，更改root和和ssh端口

```bash
iptables -A INPUT -p tcp --dport 52113 ! -s 192.168.2.0/24 -J DROP
```

在默认规则为允许的情况下，上述可以封堵ssh访问。

> **企业工作中解决这个问题：**

```
1.  vpn服务（拨号拨到VPN上，然后以VPN的内网地址区访问内部的机器地址）。
2.  前端对外提供服务器的机器SSH端口都做禁止外部IP访问限制，可以开启后端或者不对外提供服务的机器，保留SSH服务（更改root和SSH端口）。然后，我们平时就先连接此机器没在去连其他机器。
3.  流量特别大的外网机器不要开防火墙，会影响性能，购买硬件防火墙。
```

封掉3306端口

```bash
iptables -A INPUT -p tcp --dorp 3306 -j DROP
```

匹配指定的协议

```bash
iptables -A INPUT -P tcp 		# 如果不指定-p，默认就是all
```

匹配指定协议外的所有协议

```bash
iptables -A INPUT -p ! tcp
iptables -I INPUT ! -p tcp -s 192.168.2.83 -j DROP
```

匹配网段

```bash
iptables -A INPUT -s 10.0.0.0/24
iptables -A INPUT ! -s 10.0.0.0/24
```

匹配单一端口

```bash
iptables -A INPUT -p tcp ! --sport 22
iptables -A INPUT -p tcp ! --dport 22 -s 10.0.0.20 -j DROP
```

匹配端口范围

```bash
iptables -A INPUT -p tcp --sport 22:80
iptables -I INPUT -p tcp --dport 21,22,23,24 # 错误语法
iptables -I INPUT -p tcp -m multiport --dport 18:80 -j DROP
iptables -I INPUT -p tcp --dport 21:23 -j DROP # 最佳
```

> **实例1：测试匹配端口范围**

```bash
iptables -F
$ iptables -t filter -A INPUT -p tcp --dport 20:100 -j DROP
C:\Users\Company>telnet 192.168.2.83 80
正在连接192.168.2.83...无法打开到主机的连接。 在端口 80: 连接失败
$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpts:20:100 
```

测试结果
1.  ssh52113端口终端直接断掉
2.  telnet连接80不通

```bash
$ iptables -t filter -A INPUT -p tcp --dport 50000:60000 -j DROP     
$ 
ÐÅºÅµÆ³¬Ê±Ê
```

> **实例2:列举端口**

```bash
$ iptables -t filter -A INPUT -p tcp -m multiport --dport 80,90,100 -j DROP
$ iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0           multiport dports 80,90,100 
```

测试结果：telnet连接80不通

```bash
$ telnet 192.168.1.5 80
正在连接192.168.1.5...无法打开到主机的连接。 在端口 80: 连接失败
```

> **匹配ICMC类型**

`iptables -A INPUT -p icmp --icmp-type 8`

例：`iptables -A INPUT -p icmp --icmp-type 8 -j DROP`

```bash
iptables -A INPUT -p icmp -m icmp --icmp-type any -j ACCEPT
 	iptables -A FORWARD -s 192.168.1.0/24 -p icmp -m icmp --icmp-type any -j ACCEPT
# 在工作中默认是拒绝状态，用什么开什么，只有内网允许ping
```

> **匹配指定的网络接口**

```bash
iptables -A INPUT -i eth0
iptables -A FORWARD -o eth0
```

**记忆方法：**

in-interface -i input name

in-interface -o output name

**匹配网络状态**

- -m state --state

- NEW：已经或将启动洗呢连接

- ESTABLISHED：已经建立的连接

- RELATED：正在启动的新连接

- INVALID：非法或无法识别的 

- FTP服务是特殊的，需要配状态连接。

允许关联的状态包通过（web服务不要使用ftp）

```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

比喻：看电影出去WC或者接个电话，回来也得允许进去。

限制指定时间包的允许通过数量及并发数 `-m limit --limit n/{second/minute/hour/day}`

**指定时间内的请求速率“n”为速率，后面为时间分别为：秒时分**

`--limit-burst [n]`: 在同一时间内允许通过的请求“n”为数字，不指定默认为5
