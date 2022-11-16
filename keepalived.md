# Keepalived高可用集群应用实践




## 1 Keepalived介绍

Keepalived软件起初是专为LVS负载均衡软件设计的，用来管理并监控LVS集群系统中各个服务节点的状态，后来又加入了可以实现高可用的VRRP功能。因此，Keepalived除了能够管理LVS软件外，还可以作为其他服务（例如：Nginx、Haproxy、MySQL等）的高可用解决方案软件。

Keepalived软件主要是通过VRRP协议实现高可用功能的。VRRP是Virtual Router Redundancy Protocol（虚拟路由器冗余协议）的缩写，VRRP出现的目的就是为了解决静态路由单点故障问题的，它能够保证当个别节点宕机时，整个网络可以不间断地运行。所以，Keepalived一方面具有配置管理LVS的功能，同时还具有对LVS下面节点进行健康检查的功能，另一方面也可实现系统网络服务的高可用功能。

### 1.1 Keepalived服务的三个重要功能

#### 1.1.1 管理LVS负载均衡软件

早期的LVS软件，需要通过命令行或脚本实现管理，并且没有针对LVS节点的健康检查功能。为了解决LVS的这些使用不便的问题，Keepalived就诞生了，可以说，Keepalived软件起初是专为解决LVS的问题而诞生的。因此，Keepalived和LVS的感情很深，它们的关系如同夫妻一样，可以紧密地结合，愉快地工作。Keepalived可以通过读取自身的配置文件，实现通过更底层的接口直接管理LVS的配置以及控制服务的启动、停止等功能，这使得LVS的应用更加简单方便了。LVS和Keepalived的组合应用不是本章的内容范围。

#### 1.1.2 实现对LVS集群节点健康检查功能（healthcheck）

前文已讲过，Keepalived可以通过在自身的keepalived.conf文件里配置LVS的节点IP和相关参数实现对LVS的直接管理；除此之外，当LVS集群中的某一个甚至是几个节点服务器同时发生故障无法提供服务时，Keepalived服务会自动将失效的节点服务器从LVS的正常转发队列中清除出去，并将请求调度到别的正常节点服务器上，从而保证最终用户的访问不受影响；当故障的节点服务器被修复以后，Keepalived服务又会自动地把它们加入到正常转发队列中，对客户提供服务。

### 1.2 作为系统网络服务的高可用功能（failover）

Keepalived可以实现任意两台主机之间，例如Master和Backup主机之间的故障转移和自动切换，这个主机可以是普通的不能停机的业务服务器，也可以是LVS负载均衡、Nginx反向代理这样的服务器。

Keepalived高可用功能实现的简单原理为，两台主机同时安装好Keepalived软件并启动服务，开始正常工作时，由角色为Master的主机获得所有资源并对用户提供服务，角色为Backup的主机作为Master主机的热备；当角色为Master的主机失效或出现故障时，角色为Backup的主机将自动接管Master主机的所有工作，包括接管VIP资源及相应资源服务；而当角色为Master的主机故障修复后，又会自动接管回它原来处理的工作，角色为Backup的主机则同时释放Master主机失效时它接管的工作，此时，两台主机将恢复到最初启动时各自的原始角色及工作状态。

***
**<font color="#0215cd" size=2>说明：Keepalived的高可用功能是本章的重点，后面除了讲解Keepalived高可用的功能外，还会讲解Keepalived配合Nginx反向代理负载均衡的高可用的实战案例。
</font>**
***



### 1.3 Keepalived高可用故障切换转移原理

Keepalived高可用服务对之间的故障切换转移，是通过VRRP（Virtual Router Redundancy Protocol，虚拟路由器冗余协议）来实现的。

在Keepalived服务正常工作时，主Master节点会不断地向备节点发送（多播的方式）心跳消息，用以告诉备Backup节点自己还活着，当主Master节点发生故障时，就无法发送心跳消息，备节点也就因此无法继续检测到来自主Master节点的心跳了，于是调用自身的接管程序，接管主Master节点的IP资源及服务。而当主Master节点恢复时，备Backup节点又会释放主节点故障时自身接管的IP资源及服务，恢复到原来的备用角色。

#### 1.3.1 什么是VRRP

VRRP，全称Virtual Router Redundancy Protocol，中文名为虚拟路由冗余协议，VRRP的出现就是为了解决静态路由的单点故障问题，VRRP是通过一种竞选机制来将路由的任务交给某台VRRP路由器的。

VRRP早期是用来解决交换机、路由器等设备单点故障的，下面是交换、路由的Master和Backup切换原理描述，同样适用于Keepalived的工作原理。

在一组VRRP路由器集群中，有多台物理VRRP路由器，但是这多台物理的机器并不是同时工作的，而是由一台称为Master的机器负责路由工作，其他的机器都是Backup。Master角色并非一成不变的，VRRP会让每个VRRP路由参与竞选，最终获胜的就是Master。获胜的Master有一些特权，比如拥有虚拟路由器的IP地址等，拥有系统资源的Master负责转发发送给网关地址的包和响应ARP请求。

VRRP通过竞选机制来实现虚拟路由器的功能，所有的协议报文都是通过IP多播（Multicast）包（默认的多播地址224.0.0.18）形式发送的。虚拟路由器由VRID（范围0-255）和一组IP地址组成，对外表现为一个周知的MAC地址：00-00-5E-00-01-{VRID}。所以，在一个虚拟路由器中，不管谁是Master，对外都是相同的MAC和IP（称之为VIP）。客户端主机并不需要因Master的改变而修改自己的路由配置。对它们来说，这种切换是透明的。

在一组虚拟路由器中，只有作为Master的VRRP路由器会一直发送VRRP广播包（VRRP Advertisement messages），此时Backup不会抢占Master。当Master不可用时，Backup就收不到来自Master的广播包了，此时多台Backup中优先级最高的路由器会抢占为Master。这种抢占是非常快速的（可能只有1秒甚至更少），以保证服务的连续性。出于安全性考虑，VRRP数据包使用了加密协议进行了加密。

如果你在面试时，要你解答Keepalived的工作原理，建议用自己的话回答如下内容，以下为对面试官的表述：

Keepalived高可用对之间是通过VRRP通信的：
1. VRRP，全称Virtual Router Redundancy Protocol，中文名为虚拟路由冗余协议，VRRP的出现是为了解决静态路由的单点故障。
2. VRRP是通过一种竞选协议机制来将路由任务交给某台VRRP路由器的。
3. VRRP用IP多播的方式（默认多播地址（224.0.0.18））实现高可用对之间通信。
4. 工作时主节点发包，备节点接包，当备节点接收不到主节点发的数据包的时候，就启动接管程序接管主节点的资源。备节点可以有多个，通过优先级竞选，但一般Keepalived系统运维工作中都是一对。
5. VRRP使用了加密协议加密数据，但Keepalived官方目前还是推荐用明文的方式配置认证类型和密码。

### 1.4 Keepalived服务的工作原理

介绍完了VRRP，接下来我再介绍一下Keepalived服务的工作原理：

Keepalived高可用对之间是通过VRRP进行通信的，VRRP是通过竞选机制来确定主备的，主的优先级高于备，因此，工作时主会优先获得所有的资源，备节点处于等待状态，当主挂了的时候，备节点就会接管主节点的资源，然后顶替主节点对外提供服务。
在Keepalived服务对之间，只有作为主的服务器会一直发送VRRP广播包，告诉备它还活着，此时备不会抢占主，当主不可用时，即备监听不到主发送的广播包时，就会启动相关服务接管资源，保证业务的连续性。接管速度最快可以小于1秒。

## 2 Keepalived高可用服务搭建

### 2.1 安装Keepalived

通过官方地址获取Keepalived源码软件包编译安装
```sh
./configure \
--prefix=/app/keepalived-1.3.5 \
--mandir=/usr/local/share/man
make && make install
```

复制命令到/usr/sbin下

```sh
ln -s /app/keepalived-1.3.5/sbin/keepalived /usr/sbin/
```

keepalived默认会读取/etc/keepalived/keepalived.conf配置文件

```sh
mkdir /etc/keepalived && \
cp /app/keepalived-1.3.5/etc/keepalived/keepalived.conf /etc/keepalived/
```

复制sysconfig文件到/etc/sysconfig下

```sh
cp /app/keepalived-1.3.5/etc/sysconfig/keepalived /etc/sysconfig/
```

复制启动脚本到/etc/init.d下

```sh
cp ./keepalived/etc/init.d/keepalived /etc/init.d/
chmod 700 /etc/init.d/keepalived
```

### 2.2 编译错误

```sh
configure: error: libnfnetlink headers missing
yum install -y libnfnetlink-devel
```

### 2.3 启动错误

在centos7中，执行上面的步骤安装完毕后，/etc/init.d/start启动keepalived服务会报如下错误.这是因为centos7 编译安装keepalived会自动生成/usr/lib/systemd/system/keepalived.service单元文件。在单元文件中的启动命令调用的脚本没有执行权限，并且我们复制的启动脚本是复制到/etc/init.d目录下导致的

```sh
[root@mb7 keepalived-1.3.5]# systemctl status keepalived
● keepalived.service - LVS and VRRP High Availability Monitor
   Loaded: loaded (/usr/lib/systemd/system/keepalived.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since 五 2017-04-07 21:22:25 CST; 5s ago
  Process: 24459 ExecStart=/app/keepalived-1.3.5/sbin/keepalived $KEEPALIVED_OPTIONS (code=exited, status=203/EXEC)

4月 07 21:22:25 mb7 systemd[1]: Starting LVS and VRRP High Availability Monitor...
4月 07 21:22:25 mb7 systemd[1]: keepalived.service: control process exited, code=exited status=203
4月 07 21:22:25 mb7 systemd[1]: Failed to start LVS and VRRP High Availability Monitor.
4月 07 21:22:25 mb7 systemd[1]: Unit keepalived.service entered failed state.
4月 07 21:22:25 mb7 systemd[1]: keepalived.service failed.
Warning: keepalived.service changed on disk. Run 'systemctl daemon-reload' to reload units.
```

解决方法：不使用/etc/init.d/keepalived脚本启动了，修改单元文件 删掉单元文件



## 3 keepalived配置文件说明

这里的具备高可用功能的keepalived.conf配置文件包含了两个重要区块.

### 3.1 全局定义(Global Definitions)部分

这部分主要用来设置Keepalived的故障通知机制和Router ID标识。示例代码如下：

```sh
! Configuration File for keepalived
global_defs {
   notification_email { #←定义服务故障报警的Email地址。作用是当服务发生切换或RS节点等有故障时，发报警邮件。
   #←这几行是可选配置，notification_email指定在keepalived发生事件时，需要发送的Email地址，可以有多个，每行一个.
      171575158@qq.com
   }
   # 发送邮件的发送人，即发件人地址，可选.
   notification_email_from Alexandre.Cassen@firewall.loc 
   
   # 发送邮件的smtp服务器，如果本机开启了sendmail或postfix，就可以使用上面默认配置实现邮件发送，可选.
   smtp_server 192.168.200.1
   
    # 连接smtp的超时时间，可选.
   smtp_connect_timeout 30
   
   # Keepalived服务器的路由标识（router_id）。在一个局域网内，router_id应该是唯一的.
   router_id LVS_01 
}
```

***
**<font color="#f8070d" size=2>注意：第4~11行所有和邮件报警相关的参数均可以不配，在实际工作中会将监控的任务交给更加擅长监控报警的Nagios或Zabbix软件。</font>**
***


### 3.2 VRRP实例定义区块(VRRP instance(s)部分

```sh
# 定义一个vrrp_instance实例,名字是VI_1,每个vrrp_instance实例可以认为是Keepalived服务的一个实例或者作为一个业务服务，
# 在Keepalived服务配置中,这样的vrrp_instance实例可以有多个.注意,存在于主节点中的vrrp_instance实例在备节点中也要存在，
# 这样才能实现故障切换接管.
vrrp_instance VI_1 {
    
    # state MASTER表示当前实例VI_1的角色状态,当前角色为MASTER,这个状态只能有MASTER和BACKUP两种状态,并且需要大写这些字符。
    # 其中MASTER为正式工作的状态,BACKUP为备用的状态.当MASTER所在的服务器故障或失效时，
    # BACKUP所在的服务器会接管故障的MASTER继续提供服务.
    state MASTER
    
    # interface为网络通信接口.为对外提供服务的网络接口,如eth0、eth1。当前主流的服务器都有2~4个网络接口。
    interface eth0 
    
    # 虚拟路由ID标识,这个标识最好是一个数字,并且要在一个keepalived.conf配置中是唯一的。
    # 但是MASTER和BACKUP配置中相同实例的virtual_router_id又必须是一致的,否则将出现脑裂问题.
    virtual_router_id 51 
    
    # 优先级,其后面的数值也是一个数字,数字越大,表示实例优先级越高.在同一个vrrp_instance实例里，
    # MASTER的优先级配置要高于BACKUP的.若MASTER的priority值为150,那么BACKUP的priority必须小于150，
    # 一般建议间隔50以上为佳,例如：设置BACKUP的priority为100或更小的数值。
    priority 150 
    
    # 同步通知间隔.MASTER与BACKUP之间通信检查的时间间隔,单位为秒,默认为1。
    advert_int 1
    
    # 权限认证配置.包含认证类型（auth_type）和认证密码（auth_pass）；
    # 认证类型有PASS(Simple Passwd(suggested))、AH(IPSEC(not recommended))两种。
    # 官方推荐使用的类型为PASS.验证密码为明文方式，最好长度不要超过8个字符，建议用4位的数字，
    # 同一vrrp实例的MASTER与BACKUP使用相同的密码才能正常通信。
    
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    
    # 虚拟IP地址.可以配置多个IP地址,每个地址占一行,配置时最好明确指定子网掩码以及虚拟IP绑定的网络接口。
    # 否则,子网掩码默认是32位,绑定的接口和前面的interface参数配置的一致。
    # 注意,这里的虚拟IP就是在工作中需要和域名绑定的IP,即和配置的高可用服务监听的IP要保持一致.
    virtual_ipaddress { 
 		    192.168.2.120
    }
}
```

## 4 Keepalived高可用服务单实例

当没有配置高可用服务时，如果服务器宕机了怎么解决呢？无非就是找一个新服务器，配好域名解析的那个原IP，然后搭好相应的网络服务罢了，只不过手工去实现这个过程会比较漫长，相比而言，自动化切换效率更高，效果更好，而且还可以有更多的功能，例如：发送ARP广播，触发执行相关脚本动作等。

实际上也可以将高可用对的两台机器应用服务同时开启，但是只让有VIP一端的服务器提供服务，若主的服务器宕机，VIP会自动漂移到备用服务器上，此时用户的请求直接发送到备用服务器上，而无需临时启动对应服务（事先开启应用服务）。

### 4.1 实战配置Keepalived主服务器lb01 master

首先，配置lb01 MASTER的keepalived.conf配置文件，操作步骤如下：

删掉已有的所有默认配置：

```sh
vim /etc/keepalived/keepalived.conf
```

```sh
! Configuration File for keepalived

global_defs {
   notification_email {
		  171575158@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_01   #<==id为lb01，不同的keepalived.conf此ID要唯一
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
vrrp_instance VI_1 {  #←实例名字为VI_1,相同实例的备节点名字要和这个相同
    state MASTER	 #←状态为MASTER,备节点状态需要为BACKUP
    interface eth0   #←通信接口为eth0，此参数备节点设置和主节点相同
    virtual_router_id 51 #←实例ID为51，keepalived.conf里唯一
    priority 150	 #←优先级为150，备节点的优先级必须比此数字低
    advert_int 1	 #←通信检查间隔时间1秒

    authentication {	#←PASS认证类型，此参数备节点设置和主节点相同
        auth_type PASS
        auth_pass 1111
    }
    
    #←虚拟IP，即VIP为192.168.2.120,子网掩码为24位，绑定接口为eth0，别名为eth0:1，此参数备节点设置和主节点相同
    virtual_ipaddress {
        192.168.2.120 
    }
}
```

配置完毕后，启动Keepalived服务，然后检查配置结果，查看是否有虚拟IP 192.168.2.120。
```sh
[root@lb_01 ~]# ip addr|grep 192.168.2.120
    inet 192.168.2.120/24 scope global secondary eth0
```

### 4.2 配置Keepalived备服务器lb02 backup

```sh
! Configuration File for keepalived

global_defs {
   notification_email {
		  171575158@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_02   
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {  
    state BACKUP
    interface eth0   
    virtual_router_id 51 
    priority 100	 
    advert_int 1	 
    
    authentication {	
        auth_type PASS
        auth_pass 1111
    }
    
    virtual_ipaddress {
        192.168.2.120 
    }
}
```

配置完毕后重启服务，检查配置结果，查看是否有虚拟IP 192.168.2.120。

```sh
[root@lb_02 ~]# ip addr|grep 192.168.2.120
#←没有返回任何结果就对了,因为lb02为BACKUP,当主节点活着的时候,它不会接管VIP 192.168.2.120
```

出现无任何结果的现象，表示lb02的Keepalived服务单实例配置成功。如果有192.168.2.120的IP，则表示Keepalived工作不正常，说明高可用裂脑了，裂脑是两台服务器争抢同一资源导致的，同一个IP地址同一时刻应该只能出现一台服务器。

出现上述两台服务器争抢同一IP资源问题，先考虑排查两个地方：

* 主备两台服务器之间是否通信正常，如果不正常是否有iptables防火墙阻挡？
* 主备两台服务器对应的keepalived.conf配置文件是否有错误？例如，是否同一实例的virtual_router_id配置不一致。


### 4.3 高可用主备服务器切换

停掉主服务器上的Keealived服务或关闭主服务器

```sh
[root@lb_01 ~]# ip addr|grep 192.168.2.120
    inet 192.168.2.120/24 scope global secondary eth0
[root@lb_01 ~]# /etc/init.d/keepalived stop
Stopping keepalived (via systemctl):          [  确定  ]
[root@lb_01 ~]# ip addr|grep 192.168.2.120    #←查看VIP消失了
[root@lb_01 ~]#
```

此时查看BACKUP备服务器，看是否会有VIP 出现

```sh
[root@lb_02 ~]# ip addr|grep 192.168.2.120
    inet 192.168.2.120/24 scope global secondary eth0
```

可以看到备节点lb02已经接管绑定了VIP，这期间备节点还会发送ARP广播，让所有的客户端更新本地的ARP表，以便客户端访问新接管VIP服务的节点。

此时如果再启动主服务器的Keealived服务，主服务器就会接管回VIP 10.0.0.12，启动后可以观察下主备的IP漂移情况，备服务器是否释放了IP？主服务器是否又接管了IP？

```sh
[root@lb_01 ~]# /etc/init.d/keepalived start
Starting keepalived (via systemctl):              [  确定  ]
[root@lb_01 ~]# ip addr|grep 192.168.2.120
    inet 192.168.2.120/24 scope global secondary eth0
[root@lb_02 ~]# ip addr|grep 192.168.2.120	  #←备节点上的VIP则被释放了
```

### 4.4 检查nginx + keepalived工作

当主节点工作是，web页面如下：

![7cc2732f](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/7cc2732f.png) ![96c56887](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/96c56887.png)

此时模拟主节点宕机后查看nginx是否正常工作

<img src="https://raw.githubusercontent.com/CylonChau/imgbed/main/img/7d862050.png" alt="7d862050"  />

![b01b07d8](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/b01b07d8.png) ![ee56a4c8](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/ee56a4c8.png)

### 4.5 多实例

keepalive01.conf

```conf
! Configuration File for keepalived

global_defs {
   notification_email {
	171575158@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_02
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 140
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.10/24
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface eth0
    virtual_router_id 52
    priority 141
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.11/24
    }
}

```
keepalived04.conf
```conf
! Configuration File for keepalived

global_defs {
   notification_email {
				171575158@qq.com
	 }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_01
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.10/24
		}
}

vrrp_instance VI_2 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 151
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
				10.0.0.11/24
		}
}

```


```sh
[root@lb_01 ~]# ip addr|grep 10.0.0
    inet 10.0.0.10/24 scope global eth0
[root@lb_02 ~]# ip addr|grep 10.0.0
inet 10.0.0.11/24 scope global eth0
[root@lb_02 ~]# /etc/init.d/keepalived stop
Stopping keepalived (via systemctl):                       [  确定  ]
[root@lb_02 ~]# ip addr|grep 10.0.0
[root@lb_02 ~]#
[root@lb_01 ~]# ip addr|grep 10.0.0
    inet 10.0.0.10/24 scope global eth0
    inet 10.0.0.11/24 scope global secondary eth0
```



## 5 Keepalived高可用服务器的“裂脑”问题

### 5.1 什么是裂脑

由于某些原因，导致两台高可用服务器对在指定时间内，无法检测到对方的心跳消息，各自取得资源及服务的所有权，而此时的两台高可用服务器对都还活着并在正常运行，这样就会导致同一个IP或服务在两端同时存在而发生冲突，最严重的是两台主机占用同一个VIP地址，当用户写入数据时可能会分别写入到两端，这可能会导致服务器两端的数据不一致或造成数据丢失，这种情况就被称为裂脑。

### 5.2 导致裂脑发生的原因

一般来说，裂脑的发生，有以下几种原因：
* 高可用服务器对之间心跳线链路发生故障，导致无法正常通信。
* 心跳线坏了（包括断了，老化）。
* 网卡及相关驱动坏了，IP配置及冲突问题（网卡直连）。
* 心跳线间连接的设备故障（网卡及交换机）。
* 仲裁的机器出问题（采用仲裁的方案）。
* 高可用服务器上开启了iptables防火墙阻挡了心跳消息传输。
* 高可用服务器上心跳网卡地址等信息配置不正确，导致发送心跳失败。
* 其他服务配置不当等原因，如心跳方式不同，心跳广播冲突、软件Bug等。

***
<font color="#0215cd" size=2> 提示：Keepalived配置里同一VRRP实例如果virtual_router_id两端参数配置不一致，也会导致裂脑问题发生。</font>
***

### 5.3 解决裂脑的常见方案

在实际生产环境中，我们可以从以下几个方面来防止裂脑问题的发生：

同时使用串行电缆和以太网电缆连接，同时用两条心跳线路，这样一条线路坏了，另一个还是好的，依然能传送心跳消息。

当检测到裂脑时强行关闭一个心跳节点（这个功能需特殊设备支持，如Stonith、fence）。相当于备节点接收不到心跳消息，通过单独的线路发送关机命令关闭主节点的电源。

做好对裂脑的监控报警（如邮件及手机短信等或值班），在问题发生时人为第一时间介入仲裁，降低损失。例如，百度的监控报警短信就有上行和下行的区别。报警信息发送到管理员手机上，管理员可以通过手机回复对应数字或简单的字符串操作返回给服务器，让服务器根据指令自动处理相应故障，这样解决故障的时间更短。

当然，在实施高可用方案时，要根据业务实际需求确定是否能容忍这样的损失。对于一般的网站常规业务，这个损失是可容忍的。

### 5.4 解决Keepalived裂脑的常见方案

作为互联网应用服务器的高可用，特别是前端Web负载均衡器的高可用，裂脑的问题对普通业务的影响是可以忍受的，如果是数据库或者存储的业务，一般出现裂脑问题就非常严重了。因此，可以通过增加冗余心跳线路来避免裂脑问题的发生，同时加强对系统的监控，以便裂脑发生时人为快速介入解决问题。
* 如果开启防火墙，一定要让心跳消息通过，一般通过允许IP段的形式解决。
* 可以拉一条以太网网线或者串口线作为主被节点心跳线路的冗余。
* 开发监测程序通过监控软件（例如Nagios）监测裂脑。
下面是生产场景检测裂脑故障的一些思路：

1. 简单判断的思想：只要备节点出现VIP就报警，这个报警有两种情况，一是主机宕机了备机接管了；二是主机没宕，裂脑了。不管属于哪个情况，都进行报警，然后由人工查看判断及解决。
2. 比较严谨的判断：备节点出现对应VIP，并且主节点及对应服务（如果能远程连接主节点看是否有VIP就更好了）还活着，就说明发生裂脑了。

### 5.5 检测裂脑脚本

```sh
#!/bin/sh
vip=192.168.2.120
ip=192.168.2.24

function check()
{
  ping -c 2 -W 3 $ip &>/dev/null
  if [ $? -eq 0 -a `ip addr|grep $vip|wc -l` -eq 1 ]
  then
    echo 'fail'
  else
    echo ok
  fi
}
while true
do
  check
done
```

## 6 解决服务监听的网卡上不存在IP地址问题

如果配置使用<font color="#f8070d" size=3>`listen 10.0.0.12：80;`</font>的方式指定IP监听服务，而本地的网卡上没有10.0.0.12这个IP，Nginx就会报错：
```sh
root@lb01 ~]# /app/nginx/sbin/nginx
nginx [emerg] bind(to 10.0.0.12 80 failed 99 Cannot assign requested address)
```
如果要实施双主即主备同时跑不同的业务，配置文件里指定了IP监听，备节点则会因为网卡实际不存在VIP也报错。

出现上面问题的原因就是在物理网卡上没有与配置文件里监听的IP相对应的IP，解决办法是在<font color="#f8070d" size=3>`/etc/sysctl.conf`</font>中加入如下内核参数配置：

```sh
net.ipv4.ip_nonlocal_bind = 1
```
也可通过如下命令快速追加：

```sh
echo 'net.ipv4.ip_nonlocal_bind = 1' >> /etc/sysctl.conf
```

***
注：net.ipv4.ip_nonlocal_bind = 1 #此项表示启动nginx而忽略配置中监听的VIP是否存在，它同样适合Haproxy.
***
最后执行sysctl-p使上述修改生效。

## 7 解决高可用服务只针对物理服务器的问题

默认情况下Keepalived软件仅仅在对方机器宕机或Keepalived停掉的时候才会接管业务。但在实际工作中，有业务服务停止而Keepalived服务还在工作的情况，这就会导致用户访问的VIP无法找到对应的服务，那么，如何解决业务服务宕机可以将IP漂移到备节点使之接管提供服务？

第一个方法：可以写守护进程脚本来处理。当Nginx业务有问题时，就停掉本地的Keepalived服务，实现IP漂移到对端继续提供服务。实际工作中部署及开发的示例脚本如下：

```sh
#!/bin/sh
while true
do
	if [ `ps -ef|grep nginx|grep -v grep|wc -l` -lt 2 ]; then
		/etc/init.d/keepalived stop &>/dev/null
	fi
	sleep 15
done
```

后台运行脚本后停止nginx服务，查看进程状态，发现脚本会自动执行命令停止keepalived服务.

```sh
[root@lb_01 ~]# ps -ef|grep keep
root       3735      1  	0 00:48 ?       00:00:00 keepalived -D
root       3738   3735  0 00:48 ?      	00:00:00 keepalived -D
root       3782   3686  0 00:48 pts/2  	00:00:00 /bin/sh /etc/init.d/keepalived stop
root       3791   3782  	0 00:48 pts/2  	00:00:00 /bin/systemctl stop keepalived.service
root       3797      1  	0 00:48 ?       00:00:00 /bin/sh /etc/rc.d/init.d/keepalived stop
```

此时查看备节点状态，可以看出，备节点已经接管服务.

```sh
[root@lb_01 ~]# ip addr|grep 192.168.2.120
[root@lb_02 ~]# ip addr|grep 192.168.2.120
    inet 192.168.2.120/24 scope global secondary eth0
```

## 8 配置指定文件接收Keepalived服务日志

默认情况下Keepalived服务日志会输出到系统日志/var/log/messages，和其他日志信息混合在一起，很不方便，可以将其调整成由独立的文件记录Keepalived服务日志。操作步骤如下：

**1. 编辑配置文件/etc/sysconfig/keepalived**

```sh
KEEPALIVED_OPTIONS="-D"
KEEPALIVED_OPTIONS="-D-d-S 0"
```
说明：可以查看/etc/sysconfig/keepalived里注释获得上述参数的说明

```sh
 --vrrp               	-P    #←只运行VRRP子系统.
 --check              	-C    #←只运行健康检查子系统.
 --dont-release-vrrp  	-V    #←不要在守护程序停止时删除VRRP VIP和VROUTE.
 --dont-release-ipvs  	-I    #←不要在守护程序停止时删除IPVS拓扑.
 --dump-conf          	-d    #←导出备份配置数据.
 --log-detail         	-D    #←详细日志.
 --log-facility       	-S    #←设置本地的syslog设备，编号0-7(default=LOG_DAEMON)0表示指定为local0设备
```

**2. 修改rsyslog的配置文件/etc/rsyslog.conf**

```sh
74 local0.*	/var/log/keepalived.log

```
上述配置表示来自local0设备的所有日志信息都记录到/var/log/keepalived.log文件。

然后在约第42行如下信息的第一列结尾加入“；local0.none”：

```sh
 52 # Log anything (except mail) of level info or higher.
 53 # Don't log private authentication messages!
 54 *.info;mail.none;authpriv.none;cron.none;local0.none     /var/log/messages
```
上述配置表示来自local0设备的所有日志信息不再记录于/var/log/messages里。

**3. 配置完成后，重启rsyslog服务**

```sh
systemctl restart rsyslog
```

测试Keepalived日志记录结果。在重启Keepalived服务后，就会把日志信息输出到rsyslog定义的/var/log/keepalived.log文件

```sh
[root@lb_02 ~]# head -3 /var/log/keepalived.log
Apr  9 18:58:54 lb_02 Keepalived[32024]: Starting Keepalived v1.3.5 (03/19,2017), git commit v1.3.5-6-g6fa32f2
Apr  9 18:58:54 lb_02 Keepalived[32024]: Unable to resolve default script username 'keepalived_script' - ignoring
Apr  9 18:58:54 lb_02 Keepalived[32024]: Opening file '/etc/keepalived/keepalived.conf'.
```
