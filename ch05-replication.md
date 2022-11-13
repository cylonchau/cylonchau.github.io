# ch5 understanding openldap configuration - replication mechnism


## LDAP复制概述

openldap的复制 ( ***replication***) 是可以将 LDAP `DIT` (***Directory Information Tree***) 同步更新复制到一个或多个LDAP (“ `slapd` ”) 系统，主要是用于实现备份或提升性能场景。在 openldap中，需要注意的一点是 “复制” 级别属于DIT级别而非LDAP服务级别运行。因此，在运行的一个服务 (`slapd`) 中的多个DIT，每一个DIT都可以被复制到不同的其他服务中 (`slapd`) 。本章节只讲述 `openldap 2.4&#43;` 的四种复制模式。

***

&gt; **注意**：在 `openldap 2.4-` 提供的复制功能，属于一个额外的守护进程 `slurpd`。仅适用于（2.3之前版本）。

***

## openldap的复制模式

 在`openldap 2.4&#43;` 中，提供了四种复制模式：

- `Delta-syncrepl` ：2.3&#43;
- `N-Way Multi-Provider`：2.4&#43;
- `MirrorMode`：2.4&#43;
- `Syncrepl Proxy Mode`
- `slurpd`：2.3-，这种模式将不再本章节中阐述

下面将由简到易来阐述四种复制模式

## Delta-syncrepl

`Delta-syncrepl ` 模式是基于日志模式 `syncrepl ` 的一种变种模式，主要是为了解决openldap同步机制中的一些缺点。由于传统的同步机制是基于对象的同步机制，即当对象上的任何一个属性发生改变，每一个 `comsumer` 都会触发获取一次完整的对象（例如对象存在100个属性），这种模式存在以下特点：

- 优点：对象发生改变时无需注意改变次数，仅需要结果即可，类似于kubernetes list-watch 机制
- 缺点：开支过大，例如存在102,400个对象，每个对象大小1KB，当跑脚本批量更改所有对象的其中一个属性时（2Byte），每个comsumer将要触发的同步数据将为100MB数据，来更改200KB数据，外加TCP/IP协议的开销

`Delta-syncrepl` 的诞生就是为了解决 `syncrepl` 机制的缺点

&gt; Note: 
&gt;
&gt; - syncrepl 就是传统的 povider-comsumer/master-slave 模型
&gt; - Delta-syncrepl 需要注意的一点就是，当两边数据完全不同（或为空）将使用 syncrepl 同步完成后切换为 Delta-syncrepl 模式

对于 `Delta-syncrepl` 与  `syncrepl` 模式来说可以产生多种变种模型，`push` 与 `pull`

### 主从拉取模式

master-slave 拉取模式将按照下图所示的步骤进行同步

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/replication-refresh-only.png)

&lt;center&gt;图：master-slave拉取模式&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.zytrax.com/books/ldap/ch7/&lt;/center&gt;&lt;br&gt;

如图所示

- (1) 与 (3) 为 slapd服务的 master 与 slave 角色 （编号与角色按顺序对应）

- (4) 是master上的对象条目， (5) 为slave要拉去的数据
- (2) 为同步的过程
- (6) 与 (7) 分别为 master 与 slave 上需要配置的配置选项
- (A-E) 为同步的步骤

在图中可以看出，同步时提供者master会分配一个cookie (**SyncCookie**) 给消费者slave，本质上来说，cookie中包含了更改序列号，指发送给这个消费者的最后一个修改（时间戳），通过对比检查点，即同步会话发生开启时，将slave最后收到的cookie发送给master，以获得同步的限制，第一次时将会全量同步所有记录。

### 主从推送模式

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/replication-refresh-persist.png)

&lt;center&gt;图：master-slave推送模式&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.zytrax.com/books/ldap/ch7/&lt;/center&gt;&lt;br&gt;

如图所示

- (1) 与 (3) 为 slapd服务的 master 与 slave 角色 （编号与角色按顺序对应）

- (4) 是master上的对象条目， (5) 为slave要拉去的数据
- (2) 为同步的过程
- (6) 与 (7) 分别为 master 与 slave 上需要配置的配置选项
- (A-E) 为同步的步骤

在该模式下，slave可以为多个，提供者master没有维护slave的信息，即任意slave都可以申请同步，只需要满足安全限制即可，同步请求本质时一个搜索，因为配置中会定义 DN，范围，过滤条件等信息。

并且在这种模式下，cookie的维护有提供者master维护，master定期发送一个同步cookie。同步的机制类似于拉去模式，并且该模式下只要申请了同步，那么这个链接会永久被维护，也就是图中 `persist` 代表的意义，除非master, slave网络发生故障。

**主从配置的缺点**：

- 多节点访问：如果所有或大多数客户端都需要更新DIT，则所有客户端必须访问同一个服务 (slave) 进行正常读访问，而另一个服务（master）进行更新。或者，客户端始终访问主服务。后一种情况下，slave仅提供备份功能
- 弹性。由于只存在一个服务为更新的节点，这意味着存在单点故障问题

### 主从模式的配置

两种模式配置起来非常相似，只是所遇到场景的不同而需要不同的应用方法

#### 关停服务进行配置

在安装openldap部分，ldif文件中有配置database部分，这将代表了生成的配置文件，例如下面database部分的配置

```ldif
#
# Backend database definitions
#
# 这里是数据库的参数配置
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
# 使用的数据库引擎是mdb
objectClass: olcMdbConfig
olcDatabase: mdb
# Suffix 为数据库的后缀，每个数据库至少一个，在搜索时-D 后面的域后缀为dc=test,dc=com将被pass到这里
olcSuffix: dc=test,dc=com
# 指不收前面配置的权限控制的管理员账户，拥有最最高权限
olcRootDN: cn=admin,dc=test,dc=com
# 特权账户的登录密码
olcRootPW: {SSHA}xU9xFym/s7rawpmzpsYE&#43;Q1qPsVPOwDw
olcDbDirectory:	/var/lib/ldap
# 这是索引属性，下面是默认的属性
# 下列注释行意思为
#        olcDbIndex: default pres,eq
#        olcDbIndex: uid
#        olcDbIndex: cn,sn pres,eq,sub
#        olcDbIndex: objectClass eq
# pres,eq 为 present equality
# 第二行意思为，为uid属性类型维护默认索引集
# 第三行意思为，为cn,sn属性维护pres,eq,sub索引集
# 索引集类型有 pres,eq,approx,sub,none
olcDbIndex: objectClass eq,pres
olcDbIndex: uid,ou,cn,mail,surname,givenname eq,pres,sub
# 配置从缓冲区写入磁盘的，两个参数分别为多少kbyte大小的数据自上次（第二个参数）分钟则发生一次写入
olcDbCheckpoint: 1024 10
```

而在openldap中所有的同步模式的机制都是上面所示的database的子集，即需要拥有上述的配置，例如下列配置

&gt; Notes：openldap中数据库可以有多个，因为openldap是基于对象而不是基于服务

```ldif
#
# Backend database definitions
#

dn: olcDatabase={1}mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
olcSuffix: dc=test,dc=com
olcRootDN: cn=admin,dc=test,dc=com
olcDbDirectory:	/var/lib/ldap
olcDbIndex: objectClass eq,pres
olcDbCheckpoint: 1024 10
olcDbIndex: uid,ou,cn,mail,surname,givenname eq,pres,sub
olcRootPW: {SSHA}xU9xFym/s7rawpmzpsYE&#43;Q1qPsVPOwDw

# 为dn为dn: olcDatabase={1}mdb,cn=config的数据库配置同步

dn: olcOverlay=syncprov,olcDatabase={1}mdb,cn=config 
objectclass: olcSyncProvConfig
objectClass: olcOverlayConfig
olcOverlay: syncprov
olcSpCheckpoint: 100 10

dn: olcDatabase={2}mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
olcSuffix: dc=test1,dc=com
olcRootDN: cn=admin,dc=test1,dc=com
olcDbDirectory: /var/lib/ldap
olcDbIndex: objectClass eq,pres
olcDbCheckpoint: 1024 10
olcDbIndex: uid,ou,cn,mail,surname,givenname eq,pres,sub
olcRootPW: {SSHA}xU9xFym/s7rawpmzpsYE&#43;Q1qPsVPOwDw
```

配置说明：

- **olcOverlay** 属性是必须值
- **olcSyncProvConfig** 属性是基于 **olcOverlay** 后添加的属性
- 必须开启module `syncprov ` 才可以使用

上面的是master的配置，slave配置如下：

```ldif
#
# Backend database definitions
#

dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
olcSuffix: dc=test,dc=com
olcRootDN: cn=admin,dc=test,dc=com
olcRootPW: {SSHA}xU9xFym/s7rawpmzpsYE&#43;Q1qPsVPOwDw
olcDbDirectory:	/var/lib/ldap
olcDbIndex: objectClass eq,pres
olcDbIndex: uid,ou,cn,mail,surname,givenname eq,pres,sub
olcDbCheckpoint: 1024 10
olcSyncRepl: rid=002
  provider=ldap://10.0.0.10:389/
  bindmethod=simple
  binddn=&#34;cn=admin,dc=test,dc=com&#34;
  credentials=111
  searchbase=&#34;dc=test,dc=com&#34;
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry=&#34;30 5 300 3&#34;
  interval=00:00:05:00
```

当配置完成后，可以看到master上已经收到slave申请的同步请求了

```log
Nov  8 21:28:15 ldap slapd[65362]: conn=1001 fd=11 ACCEPT from IP=10.0.0.3:45860 (IP=0.0.0.0:389)
Nov  8 21:28:15 ldap slapd[65362]: conn=1001 op=0 BIND dn=&#34;cn=admin,dc=test,dc=com&#34; method=128
Nov  8 21:28:15 ldap slapd[65362]: conn=1001 op=0 BIND dn=&#34;cn=admin,dc=test,dc=com&#34; mech=SIMPLE ssf=0
Nov  8 21:28:15 ldap slapd[65362]: conn=1001 op=0 RESULT tag=97 err=0 text=
Nov  8 21:28:15 ldap slapd[65362]: conn=1001 op=1 SRCH base=&#34;dc=test,dc=com&#34; scope=2 deref=0 filter=&#34;(objectClass=*)&#34;
Nov  8 21:28:15 ldap slapd[65362]: conn=1001 op=1 SRCH attr=* &#43;
```

#### 不停服务进行配置

openldap提供了一个api `ldapi:///` 可以在外部进行更改正在运行的服务，下面配置为master不停服务进行配置

首先找到对应的db配置看是属于哪个块，例如这里为 `olcDatabase={2}mdb`

- `ls /etc/openloda/slapd.d/cn\=config/olcDatabase\=\{2\}mdb.ldif`

那么接下来的dn就需要指定这个db

```bash
cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
EOF
```

同理接下来配置slave，由于slave配置的dn为已经存在的，需要进行修改，同理这里也需要找到对应的 dn `olcDatabase={2}mdb,cn=config`

&gt; Notes: ldif也可以向yaml那样属于另外一个定义可以使用符号 “ `-` ” 分割

```bash
cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001
  provider=ldap://10.0.0.10:389/
  bindmethod=simple
  binddn=&#34;cn=admin,dc=test,dc=com&#34;
  credentials=111
  searchbase=&#34;dc=test,dc=com&#34;
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry=&#34;30 5 300 3&#34;
  interval=00:00:05:00
```

## MirrorMode

镜像模式实际上属于master-slave的一个变种，是为master-slave模式提供了两端一致性的保障，就是两端互为porvider与comsumer，即一个HA)解决方案

![image-20221108214329442](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221108214329442.png)

&lt;center&gt;图：镜像模式在多数据中心的应用架构图&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.openldap.org/doc/admin24/replication.html#MirrorMode&lt;/center&gt;&lt;br&gt;


### 镜像模式的配置

镜像模式配置与主从模式完全相同，并且工作模式也是相同的，下面仅提供热配置的方式，这种配置完后重启服务也是生效的

首先提供porvider部分的配置，因为两端是相同的，只需要修改ServerID即可

```bash
cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
-
dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 0
EOF
```

下面是comsumer部分的配置，因为两端是相同的，注意修改 `rid` , `credentials` , `provider` 值为正确的值， `provider` 由多个组成

```bash
cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001
  provider=ldap://10.0.0.10:389/
  bindmethod=simple
  binddn=&#34;cn=admin,dc=test,dc=com&#34;
  credentials=111
  searchbase=&#34;dc=test,dc=com&#34;
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry=&#34;30 5 300 3&#34;
  interval=00:00:05:00
-
add: olcMirrorMode
olcMirrorMode: TRUE
EOF
```

## N-way Multi-Master

在 `openldap 2.4&#43;` 引入了 多路多主 (&lt;font color=&#34;#f8070d&#34; size=3&gt;***N-way Multi-Master***&lt;/font&gt; ) 模式。该模式中任何数量的主机都可以彼此同步。由于 多路多主模式也是属于 `Syncrepl ` 模式的变种，这里不过多阐述同步原理了。

&gt; Notes：该模式中，所有master上的时钟同步是必须的

下图为 &lt;font color=&#34;#f8070d&#34; size=3&gt;***N-way Multi-Master***&lt;/font&gt; 模式的架构图，在此模式下所有的节点即为master又为slave，所有的节点的slave都为集群数量减一个。

![image-20200730181558472](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20200730181558472.png)

&lt;center&gt;图：N-way Multi-Master模式&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.openldap.org/doc/admin24/replication.html#MirrorMode&lt;/center&gt;&lt;br&gt;


如图所示

- (1)  (2)  (3) 为 slapd服务，因为使用 `syncrepl` 复制技术需要为每个服务提供一个唯一的 `olcServerID/ServerID` 在整个集群中。
- (5)  (6)  (7)  为 (1)  (2)  (3)  服务 `syncprov overlay` 的配置
- (4) 是为 (1)  (2)  (3)  提供了 `provider syncprov overlay` 的配置
- 此模式中每个节点发生了写入动作后，都会同步至集群中另外的所有节点上

&gt; Notes：这种模式下不支持增量同步

### syncrepl N-Way Multi-Master配置

这种模式与最基础的 `syncrepl` 是类似的，不在过多阐述

首先提供porvider部分的配置，注意修改ServerID即可，多个节点之间必须唯一

```bash
cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
EOF

cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 0
EOF
```

下面是comsumer部分的配置，因为两端是相同的，注意修改 `rid` , `credentials` , `provider` 值为正确的值

```bash
cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: {0}rid=001
  provider=ldap://10.0.0.10:389/
  bindmethod=simple
  binddn=&#34;cn=admin,dc=test,dc=com&#34;
  credentials=111
  searchbase=&#34;dc=test,dc=com&#34;
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry=&#34;30 5 300 3&#34;
  interval=00:00:05:00
olcSyncRepl: {1}rid=002
  provider=ldap://10.0.0.10:389/
  bindmethod=simple
  binddn=&#34;cn=admin,dc=test,dc=com&#34;
  credentials=111
  searchbase=&#34;dc=test,dc=com&#34;
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry=&#34;30 5 300 3&#34;
  interval=00:00:05:00
...
-
add: olcMirrorMode
olcMirrorMode: TRUE
EOF
```

## Syncrepl Proxy

Syncrepl Proxy 模式就是 Syncrepl ，只不过通过acl控制slave为只读模式，因为配置相同，将不重复阐述

![image-20221108222307430](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221108222307430.png)

&lt;center&gt;图：Syncrepl Proxy模式&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.openldap.org/doc/admin24/replication.html#Syncrepl%20Proxy&lt;/center&gt;&lt;br&gt;


## 创建指定用户同步

```bash
cat &lt;&lt; EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=admin,dc=cylon,dc=org
dn: cn=123123,ou=group,dc=cylon,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 10011
cn: mygroup
EOF

cat &lt;&lt; EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=admin,dc=cylon,dc=org
dn: uid=rpuser,dc=cylon,dc=org
objectClass: simpleSecurityObject
objectclass: account
uid: rpuser
description: Replication User
userPassword: root1234
EOF
```



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [***replication***](https://www.openldap.org/doc/admin24/replication.html)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [***Chapter 7 Replication &amp; Referral***](https://www.zytrax.com/books/ldap/ch7/#ol-syncrepl-mm)

