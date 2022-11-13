# ch1 understanding openldap - install


[toc]

## 生产服务器硬件配置需求

ldap服务对系统环境的要求不高，一般在生产场景，ldap服务应该最少是两台，这样某一台物理服务器岩机才不会因单点问题影响生产业务故障，对于硬件要求，本质上openldap使用硬件资源并不大，网上有两个帖子提出了openldap的硬件需求：

- 2003年openldap官网留言，我想安装一个 LDAP 服务器来验证邮件服务器的用户，目前有200个用户需要多少内存和CPU？&lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;
    - 1GHZ PIII/512MB 足以 
- 运行于Ubuntu LXC 之上的openldap，用户150,000，sladp进程常驻内存为200-300MB，mdb数据库文件大小为377MB，10 并发平均响应时间为 9-11 毫秒 &lt;sup&gt;&lt;a href=&#34;#13&#34;&gt;[13]&lt;/a&gt;&lt;/sup&gt;

操作系统：Centos7/8 64bit。

| 操&amp;nbsp;作&amp;nbsp;系&amp;nbsp;统 | 其&amp;nbsp;它                    |
| -------------------------- | ----------------------------- |
| CentOS-7.6                 | 当前很稳定且免费的Linux版本。 |

网卡及IP资源

| 名&amp;nbsp;称     | 接 口 | IP        | 用途                            |
| -------------- | ----- | --------- | ------------------------------- |
| ldap主服务器01 | eth0  | 10.0.0.17 | 外部管理IP，用于WAN数据转发。   |
|                | eth1  | 10.0.0.17 | 备用管理IP，用于LAN内数据转发。 |
| ldap从服务器02 | eth0  | 10.0.0.8  | 管理IP，用于LAN数据转发。       |
|                | eth1  | 10.0.0.18 | 外部管理IP，用于WAN数据转发。   |


***

&gt; Tips：内外网IP分配可采用最后8位相同的方式，这样使于管理。

***

## openldap master服务安装

CentOS/Redhat 安装OpenLDAP组件

```bash
yum install -y \
	openldap \
	openldap-servers \
	openldap-clients \
	openldap-devel \
	compat-openldap
```

Ubuntu18.04/22.04/20.04

```bash
sudo apt -y install slapd ldap-utils
```

默认OpenLDAP服务所使用的端口为389，此端口采用明文传输数据，数据信息得不到保障。所以可以通过配置CA及结合TLS/SASL实现数据加密传输，所使用端口为636。

编译安装

```
yum install -y \
	libtool-ltdl \
	libtool-ltdl-devel \
	gcc \
	openssl \
	openssl-devel
```

Openldap依赖的相关软件：

- http://www.openldap.org/doc/admin24/install.html

## openldap参数配置优化

openldap配置文件分为五部分

- sladp进程配置部分
- frontend：是一个特殊的 `olcDatabaseConfig` 配置提供权限认证
- database：存储的真实引擎
- backend：backend在slapd中不是真是的数据库，而是提供的一种转发方式

### openldap目录布局


- `/etc/openldap/slapd.d/*`： `/etc/openldap/slapd.ldif`配置信息生成的文件，每修改一次配置信息，这里的东西就要重新生成。
- `/var/lib/ldap/*`：OpenLDAP的数据文件。
- `/usr/share/openldap-servers/DB_CONFIG.example`：模板数据库配置文件。
- `/usr/share/openldap-servers/slapd.ldif`：默认模板配置文件。

默认OpenLdap服务所使用的端口为**389**，此端口采用明文传输数据，数据信息得不到保障。所以可以通过配置CA及结合TLS/SSL实现数据加密传输，所使用端口为**636**。

```
cp /usr/share/openldap-servers/slapd.ldif /etc/openldap/
```

指定密码，不提示

```
$ slappasswd -s 111
{SSHA}QnB7dO98&#43;hoCUgiaAYaiJWnDzlhn2Tn6
```

```ldif
# 指定要搜索的后缀
olcSuffix: dc=cylon,dc=org
# rootdn，使用这个dn可以登录服务器
olcRootDN: cn=admin,dc=cylon,dc=org
olcDbDirectory: /var/lib/ldap
# 指定ldapserver管理员密码==
olcRootPW: {SSHA}QnB7dO98&#43;hoCUgiaAYaiJWnDzlhn2Tn6
```

###  日志及缓存参数。

```
# 设置日志级别，记录日志信息方便调试 stats 256（日志连接/操作/结果） 
olcLogLevel: stats

# 设置ldap可以缓存的记录数
olcDbCacheSize: 1000

# checkpoint 可以吧内存中的数据写会数据文件的操作，此设置表示每达到2048kb或者10分钟执行一次
olcDbCheckpoint: 1024 10
```

### 开启扩展schema

openldap自带一些ldif文件 &lt;sup&gt;&lt;a href=&#34;#10&#34;&gt;[10]&lt;/a&gt;&lt;/sup&gt;，LDIF 是 ***LDAP Data Interchange Format***  的缩写，是作为存储与LDAP中 ” **文本格式** “ 的数据交换格式，每个条目代表的存入LDAP中记录的属性，记录之间用空行分隔，每行都是 “属性:值” 的格式，例如我们存入LDAP中一个记录，其属性有

- dn (***distinguished name***) 用于标识目录中名称的唯一标识符
- dc (***domain component***) 表示域组成，例如域名 `www.mydomain.com` 那么dc 应该配置为`DC=www,DC=mydomain,DC=com`
- ou (***organizational unit***) 这是指用户的组织，这里也可以代表用户组，例如 `OU=Lawyer,OU=Developer`
- cn (***common name***) 表示查询的个体对象的名称，这里可以代表用户名，服务名等，例如 `cn=cylon`

具有多个属性条目，在ldap中代表一条记录，例如

```
 dn: cn=The Postmaster,dc=example,dc=com
 objectClass: organizationalRole
 cn: The Postmaster
```

而ldif文件就是定义这些属性的文件，下面是openldap安装后默认的ldif文件说明：

- `collective.ldif `**：*Collective Attribute*** 组成LDAP条目的共享属性

- `corba.ldif`**：*Common Object Request Broker Architecture*** 的缩写，旨在促进部署在不同平台上的系统的通信 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

- `cosine.ldif`**：*Cooperation for Open Systems Interconnection Networking in Europe***的缩写，用于给 `cosine` 与 `Internet X.500` 模式项目提供LDAP中的属性格式 &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt;
- `duaconf.ldif`**：*Directory User Agents*** 的缩写，DUA是协议客户端，是向ldap或`Internet X.500` DSA发起请求的一端，这是为DUA客户端提供了通用配置 &lt;sup&gt;&lt;a href=&#34;#4&#34;&gt;[4]&lt;/a&gt;&lt;/sup&gt;
- `dyngroup.ldif`**：*Dynamic Group***的缩写，动态组是LDAP中的一种组，与静态组相反，DG是以URL形式为搜索条件来隐式定义一组用户。&lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt;
    - 静态组 ***Static Groups*** 使用一组DN显式定义一组用户
- `inetorgperson.ldif`**：inetOrgPerson**类是 `RFC2798中` 定义的类。在LDAP中为 **user** 的父类，比如说用户密码，登录时间等属性，更多可以参考 &lt;sup&gt;&lt;a href=&#34;#6&#34;&gt;[6]&lt;/a&gt;&lt;/sup&gt;
- `java.ldif`：用于java的一些属性
- `misc.ldif`**：*Miscellaneous*** 的简写，这里主要是一些电子邮件相关属性
- `nis.ldif`**：*Network Information Service*** 的缩写，NIS是一种发现机制，这里是一种server-client的目录属性，例如网络中的主机名，用户名之类 &lt;sup&gt;&lt;a href=&#34;#7&#34;&gt;[7]&lt;/a&gt;&lt;/sup&gt;
- `openldap.ldif`：
- `pmi.ldif`**：*Privilege Management Infrastructure*** 的缩写，是基于x.509的授权访问控制模型，这里是提供了基于PMI的一些属性
- `ppolicy.ldif`**：*password policy*** 的缩写，提供了增强的密码管理功能，例如账户到期时间，锁定等

```
include: file:///etc/openldap/schema/collective.ldif   # OpenLDAP的核心schema必须
include: file:///etc/openldap/schema/corba.ldif # 
include: file:///etc/openldap/schema/cosine.ldif
include: file:///etc/openldap/schema/duaconf.ldif
include: file:///etc/openldap/schema/dyngroup.ldif
include: file:///etc/openldap/schema/inetorgperson.ldif
include: file:///etc/openldap/schema/java.ldif
include: file:///etc/openldap/schema/misc.ldif
include: file:///etc/openldap/schema/nis.ldif
include: file:///etc/openldap/schema/openldap.ldif
include: file:///etc/openldap/schema/pmi.ldif
include: file:///etc/openldap/schema/ppolicy.ldif
```

### 授权及安全参数配置

```ldif
access to dn=&#34;cn=subschema&#34; by * read
access to 
    by self write 
    by dn subtree=&#34;ou=sysusers,dc=test,dc=com&#34; read
    by anonymous auth
```

**关于更多权限管理的说明，可以参考官方手册第八章** &lt;sup&gt;&lt;a href=&#34;#12&#34;&gt;[12]&lt;/a&gt;&lt;/sup&gt;

***

&gt; **提示**: 
&gt;
&gt; 参数在文件中的先后位置不能随意动。
&gt;
&gt; 空行和以“#”开头的注释行将被忽略。如果一行以空格开头，它将被认为是接着前一行的（即使前一行是注释）。

***

### 配置syslog记录ldap服务日志配置syslog

记录ldap服务日志，默认级别为256；

```sh
echo &#39;local4.*          /var/log/slapd.log&#39; &gt;&gt; /etc/rsyslog.conf
local4.*                /var/log/slapd.log
```

### 配置LDAP数据库存放路径

***

&gt; **注意**：
&gt;
&gt; slapd.conf中设定了LDAP数据库格式为hdb，存储路径`/var/1ib/ldap`

***

```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

chown ldap.ldap /var/lib/ldap/DB_CONFIG
chmod 700 /var/lib/ldap/DB_CONFIG
```

### 整合的配置

```ldif
#
# See slapd-config(5) for details on configuration options.
# This file should NOT be world readable.
#

dn: cn=config
objectClass: olcGlobal
cn: config
olcArgsFile: /var/run/openldap/slapd.args
olcPidFile: /var/run/openldap/slapd.pid
olcLogLevel: stats
olcDisallows: bind_anon
#
# TLS settings
#
olcTLSCACertificatePath: /etc/openldap/certs
olcTLSCertificateFile: &#34;OpenLDAP Server&#34;
olcTLSCertificateKeyFile: /etc/openldap/certs/password
#
# Do not enable referrals until AFTER you have a working directory
# service AND an understanding of referrals.
#
#olcReferral: ldap://root.openldap.org
#
# Sample security restrictions
#	Require integrity protection (prevent hijacking)
#	Require 112-bit (3DES or better) encryption for updates
#	Require 64-bit encryption for simple bind
#
#olcSecurity: ssf=1 update_ssf=112 simple_bind=64

#
# Load dynamic backend modules:
# - modulepath is architecture dependent value (32/64-bit system)
# - back_sql.la backend requires openldap-servers-sql package
# - dyngroup.la and dynlist.la cannot be used at the same time
#

#dn: cn=module,cn=config
#objectClass: olcModuleList
#cn: module
#olcModulepath:	/usr/lib/openldap
#olcModulepath:	/usr/lib64/openldap
#olcModuleload: accesslog.la
#olcModuleload: auditlog.la
#olcModuleload: back_dnssrv.la
#olcModuleload: back_ldap.la
#olcModuleload: back_mdb.la
#olcModuleload: back_meta.la
#olcModuleload: back_null.la
#olcModuleload: back_passwd.la
#olcModuleload: back_relay.la
#olcModuleload: back_shell.la
#olcModuleload: back_sock.la
#olcModuleload: collect.la
#olcModuleload: constraint.la
#olcModuleload: dds.la
#olcModuleload: deref.la
#olcModuleload: dyngroup.la
#olcModuleload: dynlist.la
#olcModuleload: memberof.la
#olcModuleload: pcache.la
#olcModuleload: ppolicy.la
#olcModuleload: refint.la
#olcModuleload: retcode.la
#olcModuleload: rwm.la
#olcModuleload: seqmod.la
#olcModuleload: smbk5pwd.la
#olcModuleload: sssvlv.la
#olcModuleload: syncprov.la
#olcModuleload: translucent.la
#olcModuleload: unique.la
#olcModuleload: valsort.la


#
# Schema settings
#

dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema

include: file:///etc/openldap/schema/core.ldif
include: file:///etc/openldap/schema/collective.ldif
include: file:///etc/openldap/schema/corba.ldif
include: file:///etc/openldap/schema/cosine.ldif
include: file:///etc/openldap/schema/duaconf.ldif
include: file:///etc/openldap/schema/dyngroup.ldif
include: file:///etc/openldap/schema/inetorgperson.ldif
include: file:///etc/openldap/schema/java.ldif
include: file:///etc/openldap/schema/misc.ldif
include: file:///etc/openldap/schema/nis.ldif
include: file:///etc/openldap/schema/openldap.ldif
include: file:///etc/openldap/schema/pmi.ldif
include: file:///etc/openldap/schema/ppolicy.ldif

#
# Frontend settings
#
# 这里是对前端权限的配置，通常默认，不添加权限
dn: olcDatabase=frontend,cn=config
objectClass: olcDatabaseConfig
objectClass: olcFrontendConfig
olcDatabase: frontend
#
# Sample global access control policy:
#	Root DSE: allow anyone to read it
#	Subschema (sub)entry DSE: allow anyone to read it
#	Other DSEs:
#		Allow self write access
#		Allow authenticated users read access
#		Allow anonymous users to authenticate
#
#olcAccess: to dn.base=&#34;&#34; by * read
#olcAccess: to dn.base=&#34;cn=Subschema&#34; by * read
#olcAccess: to *
#	by self write
#	by users read
#	by anonymous auth
#
# if no access controls are present, the default policy
# allows anyone and everyone to read anything but restricts
# updates to rootdn.  (e.g., &#34;access to * by * read&#34;)
#
# rootdn can always read and write EVERYTHING!
#

#
# Configuration database
#
# 这里是对后端数据库权限的配置
dn: olcDatabase=config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: config
olcAccess: to attrs=userPassword,shadowLastChange
    by dn.children=&#34;cn=admin,dc=test,dc=com&#34; write
    by anonymous auth
    by self write
    by * none
olcAccess: to * by dn.base=&#34;gidNumber=0&#43;uidNumber=0,cn=peercred,cn=external,cn=auth&#34; manage
    by group.exact=&#34;cn=configadmin,ou=admin,dc=seal,dc=com&#34; write
    by * none

#
# Server status monitoring
#

dn: olcDatabase=monitor,cn=config
objectClass: olcDatabaseConfig
olcDatabase: monitor
olcAccess: to * by dn.base=&#34;gidNumber=0&#43;uidNumber=0,cn=peercred,cn=external,c
 n=auth&#34; read by dn.base=&#34;cn=Manager,dc=my-domain,dc=com&#34; read by * none

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

更多配置文件选项说明可以参考 &lt;sup&gt;&lt;a href=&#34;#14&#34;&gt;[14]&lt;/a&gt;&lt;/sup&gt;

### 生成配置文件

修改配置文件后需要重新生成配置文件

```
slapadd -n 0 -F /etc/openldap/slapd.d -l /etc/openldap/slapd.ldif
```

生成配置文件时的错误

```bash
$ slapadd -n 0 -F /etc/openldap/slapd.d -l /etc/openldap/slapd.ldif
63678619 str2entry: entry -1 has no dn
slapadd: could not parse entry (line=31)
_########              43.66% eta   none elapsed            none spd   7.0 M/s 
```

对于 `slapd.ldif` 需要注意下列事项

- 行首 `#` 为注释
- 行尾不能有任何空白，这里也是很难排查的一个点
- 对于不同的属性需要放对位置快否则会报错，例如 `olcDbCacheSize: 1000` 是hdb配置，mdb添加会报错，有明显提示
- 如果生产配置失败后，修改配置文件后再次生成需要删除 ` rm -fr slapd.d/*`

例如下面就是一个有提示的典型例子

```bash
63678513 Entry (cn=config), attribute &#39;olcDbCacheSize&#39; not allowed
slapadd: dn=&#34;cn=config&#34; (line=1): (65) attribute &#39;olcDbCacheSize&#39; not allowed
```

下面的报错比较不明显，通常删除 ` rm -fr slapd.d/*` 后重试

```bash
slapadd: could not add entry dn=&#34;cn=config&#34; (line=1): 
_###                   17.92% eta   none elapsed            none spd   6.3 M/s 
```

## 为LDAP初始化数据

部署完成后就是访问，ldap了，此时向OpenLDAP 搜索会发现没有内容

```
$ ldapsearch -LLL -x -W -H ldap://10.0.0.4 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34; &#34;(uid=*)&#34;

Enter LDAP Password: 
No such object (32)
```

&gt; **Reference**
&gt;
&gt; - http://www.openldap.org/faq/data/cache/1.html
&gt; - http://www.openldap.org/doc/admin24/appendix-common-errors.html

### 创建Root条目

在第一次部署好openldap中，实际上是没有任何条目的，此时是无法存入数据，有人说在 database setting中配置了 `olcRootDN` ，这里是指标识用哪个数据库的（即那个root存入哪里），而不是一个具体的root dn，所以需要手动创建一个

例如下列，是创建一个RootDN

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: dc=test,dc=com
objectclass: top
objectClass: organizationalUnit
objectclass: extensibleObject
description: US Organization
ou: people
EOF
```

这里需要注意几点：

- `dn: dc=test,dc=com` 是指定存储的地方，如果在database配置中为配置 `olcRootDN` 则报错不会被存储
- 由于创建的的是root，所以ou的 `objectClass` 会报错，需要用一个 `extensibleObject` 才可以创建 &lt;sup&gt;&lt;a href=&#34;#15&#34;&gt;[15]&lt;/a&gt;&lt;/sup&gt;

### 创建子域

此时因为有了RootDN，可以指定dn为子域了，并且 `objectClass: organizationalUnit` 可以单独使用

这里子域其实可以理解为二级域名了，在公司中也可以为子公司

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: ou=group,dc=test,dc=com
objectClass: organizationalUnit
ou: group
EOF
```

### 创建组

这里使用 `posixGroup` ***Portable Operating System Interface of UNIX*** 的简写，这里可以理解为Linux用户管理的标准，包含一些标准的属性，类似于 `/etc/group`

下面创建一个gid为10001的组，组名为tech

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: cn=tech,ou=group,dc=test,dc=com
objectClass: posixGroup
gidNumber: 10001
cn: tech
EOF
```

### 创建用户

创建用户user01

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=user01,ou=Group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/user01
loginShell: /bin/bash
uid: user01
cn: user01
uidNumber: 10004
gidNumber: 10001
userPassword: {SSHA}hJpIIVxj1qS9g05qUlgG&#43;o7MO14EXbFQ
sn: user01
givenName: user01
```

创建用户cylon

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=cylon,ou=Group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/cylon
loginShell: /bin/bash
uid: cylon
cn: cylon
userPassword: {SSHA}2pvE4C6xy8dkmW2aD/eEocVajg8BujWW
uidNumber: 10005
gidNumber: 10001
sn: cylon
EOF
```

如上面所示，`objectClass` &lt;sup&gt;&lt;a href=&#34;#11&#34;&gt;[11]&lt;/a&gt;&lt;/sup&gt; 表示这个ldap条目拥有的属性，可以看到 `posixAccount`，`inetOrgPerson` 等都是导入的schema文件，其中 `uidNumber` ，`userPassword` 都是 NIS 定义的，`cn` 则是 core定义的

***

&gt; **提示:**
&gt;
&gt; - 以上信息中ldap
&gt;     - 用户1：user01密码 `111`
&gt;     - 用户2：cylon 密码 `123456`
&gt; - 这些原始信息是如何获得的？

***

### 检查初始化测试数据。

查询导入的结果，默认查的是所有数据

```
ldapsearch -LLL -w 111 -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34;
```

### 备份ldap数据库数据


```bash
ldapadd -x -H ldap://cylon.org -D &#34;cn=admin,dc=cylon,dc=org&#34; -W -f base.ldif &gt;base.ldif
ldapadd -x -H ldap://cylon.org -D &#34;cn=admin,dc=cylon,dc=org&#34; -W -f test.ldif &gt;test.ldif
```



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [***msg00281***](https://www.openldap.org/lists/openldap-software/200306/msg00281.html)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [***Object Request Broker Architecture***](https://ldapwiki.com/wiki/Common%20Object%20Request%20Broker%20Architecture)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [***Cooperation for Open Systems Interconnection Networking in Europe***](https://ldapwiki.com/wiki/Cooperation%20for%20Open%20Systems%20Interconnection%20Networking%20in%20Europe)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [***DUA***](https://ldapwiki.com/wiki/DUA)
&gt;
&gt; &lt;sup id=&#34;5&#34;&gt;[5]&lt;/sup&gt; [***DynamicGroup***](https://ldapwiki.com/wiki/DynamicGroup)
&gt;
&gt; &lt;sup id=&#34;6&#34;&gt;[6]&lt;/sup&gt; [***InetOrgPerson***](https://ldapwiki.com/wiki/InetOrgPerson) 
&gt;
&gt; &lt;sup id=&#34;7&#34;&gt;[7]&lt;/sup&gt; [***Network Information Service***](https://ldapwiki.com/wiki/Network%20Information%20Service)
&gt;
&gt; &lt;sup id=&#34;8&#34;&gt;[8]&lt;/sup&gt; [***Privilege Management Infrastructure***](https://ldapwiki.com/wiki/Privilege%20Management%20Infrastructure)
&gt;
&gt; &lt;sup id=&#34;9&#34;&gt;[9]&lt;/sup&gt; [***Password Policy***](https://ldapwiki.com/wiki/Password%20Policy)
&gt;
&gt; &lt;sup id=&#34;10&#34;&gt;[10]&lt;/sup&gt; [***LDAP Data Interchange Format***](https://ldapwiki.com/wiki/LDAP%20Data%20Interchange%20Format)
&gt;
&gt; &lt;sup id=&#34;11&#34;&gt;[11]&lt;/sup&gt; [***Object Class***](https://ldapwiki.com/wiki/ObjectClass)
&gt;
&gt; &lt;sup id=&#34;12&#34;&gt;[12]&lt;/sup&gt; [***8. Access Control***](https://www.openldap.org/doc/admin24/access-control.html)
&gt;
&gt; &lt;sup id=&#34;13&#34;&gt;[13]&lt;/sup&gt; [***openldap system requirements for virtualization***](https://stackoverflow.com/questions/40080921/openldap-system-requirements-for-virtualization)
&gt;
&gt; &lt;sup id=&#34;14&#34;&gt;[14]&lt;/sup&gt; [***slapdconf2***](https://www.openldap.org/doc/admin24/slapdconf2.html)
&gt;
&gt; &lt;sup id=&#34;15&#34;&gt;[15]&lt;/sup&gt; [***attribute dc is not allowed***](https://community.oracle.com/tech/apps-infra/discussion/2013873/attribute-dc-is-not-allowed)


