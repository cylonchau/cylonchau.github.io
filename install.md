# 

[toc]

## 1 生产服务器硬件配置需求

ldap服务对系统环境的要求不高，一般在生产场景，ldap服务应该最少是两台，这样某一台物理服务器岩机才不会因单点问题影响生产业务故障，下面是在工作中单台的硬件配置参考：

| 设备名称 | 配置描述                                                     |
| -------- | ------------------------------------------------------------ |
| CPU      | 一颗四核至强处理器 Intel（R）Xeon（R）CPU E5606@2.13GHz      |
| MEM      | 8GB（2×4GB）667MHz                                           |
| Raid     | SAS Raid 10/Raid0 SAS6I Raid卡（支持1/0/5/6/10）。           |
| DISK     | <font color="#f8070d" size=2> 146GBx1  3.5英寸SAS硬盘  15k</font> |

操作系统：Centos7/8 64bit。

| 操&nbsp;作&nbsp;系&nbsp;统 | 其&nbsp;它                    |
| -------------------------- | ----------------------------- |
| CentOS-7.6                 | 当前很稳定且免费的Linux版本。 |

网卡及IP资源

| 名&nbsp;称     | 接 口 | IP        | 用途                            |
| -------------- | ----- | --------- | ------------------------------- |
| ldap主服务器01 | eth0  | 10.0.0.17 | 外部管理IP，用于WAN数据转发。   |
|                | eth1  | 10.0.0.17 | 备用管理IP，用于LAN内数据转发。 |
| ldap从服务器02 | eth0  | 10.0.0.8  | 管理IP，用于LAN数据转发。       |
|                | eth1  | 10.0.0.18 | 外部管理IP，用于WAN数据转发。   |


***

> Tips：内外网IP分配可采用最后8位相同的方式，这样使于管理。

***

## 2 openldap master服务安装

yum 安装OpenLDAP组件

```
yum install -y \
	openldap \
	openldap-servers \
	openldap-clients \
	openldap-devel \
	compat-openldap
```

默认OpenLdap服务所使用的端口为389，此端口采用明文传输数据，数据信息得不到保障。所以可以通过配置CA及结合TLS/SASL实现数据加密传输，所使用端口为636。

编译安装

```
yum install -y \
	libtool-ltdl \
	libtool-ltdl-devel \
	gcc \
	openssl \
	openssl-devel
```

重新生成配置文件

```
slapadd -n 0 -F /etc/openldap/slapd.d -l /etc/openldap/slapd.ldif
```

Openldap依赖的相关软件：

- http://www.openldap.org/doc/admin24/install.html

## 3 ldap参数配置优化。

### 3.1 配置ldap管理员密码参数


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
[root@node06 openldap]# slappasswd -s 123 
{SSHA}OHmKWcXD6YUDEPhCK9W5ut8EK2esIaNF
```

```ldif
# 指定要搜索的后缀
olcSuffix: dc=cylon,dc=org
# rootdn，使用这个dn可以登录服务器
olcRootDN: cn=admin,dc=cylon,dc=org
olcDbDirectory: /var/lib/ldap
# 指定ldapserver管理员密码==
olcRootPW: {SSHA}OHmKWcXD6YUDEPhCK9W5ut8EK2esIaNF
```

###  3.2 日志及缓存参数。

```
# 设置日志级别，记录日志信息方便调试 stats 256（日志连接/操作/结果） 
olcLogLevel: stats

# 设置ldap可以缓存的记录数
olcDbCacheSize: 1000

# checkpoint 可以吧内存中的数据写会数据文件的操作，此设置表示每达到2048kb或者10分钟执行一次
olcDbCheckpoint: 1024 10
```

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

###3.3 授权及安全参数配置

```ldif
access to dn="cn=subschema" by * read
access to 
    by self write 
    by dn subtree="ou=sysusers,dc=intra,dc=qq,dc=com" read
    by anonymous auth
```

**权限管理的说明**

- http://www.openldap.org/doc/admin24/access-control.html

***

> **提示**: 
>
> 参数在文件中的先后位置不能随意动。
>
> 空行和以“#”开头的注释行将被忽略。如果一行以空格开头，它将被认为是接着前一行的（即使前一行是注释）。

***

**官方配置文件详解：**

- http://www.openldap.org/doc/admin24/guide.html#Configuring%20slapd
- http://www.openldap.org/software/man.cgi?query=slapd-config&sektion=5&apropos=0&manpath=OpenLDAP+2.4-Release

### 3.4 配置syslog记录ldap服务日志配置syslog

记录ldap服务日志，默认级别为256；

```sh
echo 'local4.*          /var/log/slapd.log' >> /etc/rsyslog.conf

local4.*                /var/log/slapd.log
```

### 3.5 配置LDAP数据库路径

***

> **注意**：
>
> slapd.conf中设定了LDAP数据库格式为hdb，存储路径`/var/1ib/ldap`

***

```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

chown ldap.ldap /var/lib/ldap/DB_CONFIG
chmod 700 /var/lib/ldap/DB_CONFIG
```

### 3.6 查看OpenLDAP 数据库

此时要查看OpenLDAP数据库，会发现没有内容

```
ldapsearch -LLL -x -W -H ldap://10.0.0.4 -D "cn=admin,dc=cylon,dc=com" -b "dc=cylon,dc=com" "(uid=*)"

Enter LDAP Password: 
No such object (32)
```



> **Reference**
>
> - http://www.openldap.org/faq/data/cache/1.html
> - http://www.openldap.org/doc/admin24/appendix-common-errors.html

## 4 为OpenLDAP初始化数据

通过OpenLDAP客户端工具来初始化数据经过长期的使用、教学比对，第三种方法比较适合初学者学习掌握，不过本文以最简单的不好理解的方法讲解。

为ldap master 初始化基础用户数据。

通过事先准备好的文件初始化数据。

```bash
cat << EOF | ldapadd -x -H ldaps://10.0.0.4 -D "cn=admin,dc=cylon,dc=com" -W 
dn: dc=cylon,dc=com
objectClass: organization
objectClass: dcObject
dc: cylon
o: cylon

dn: ou=People,dc=cylon,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=group,dc=cylon,dc=com
objectClass: organizationalUnit
ou: group

dn: cn=tech,ou=group,dc=cylon,dc=com
objectClass: posixGroup
gidNumber: 10001
cn: tech
EOF

cat << EOF | ldapadd -x -H ldaps://10.0.0.4 -D "cn=admin,dc=cylon,dc=com" -W 
dn: cn=test_admin,ou=Group,dc=cylon,dc=com
objectClass: groupOfUniqueNames
cn: test_admin
uniqueMember: cn=test_admin,ou=Group,dc=cylon,dc=com
EOF
```

一个条目的完整信息

```bash
cat << EOF | ldapadd -x -H ldaps://10.0.0.4 -D "cn=admin,dc=cylon,dc=com" -W 
dn: uid=user01,ou=People,dc=cylon,dc=com
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
userPassword: {SSHA}hJpIIVxj1qS9g05qUlgG+o7MO14EXbFQ
sn: user01
givenName: user01

dn: uid=oldgirl,ou=People,dc=cylon,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/oldgirl
loginShell: /bin/bash
uid: oldgirl
cn: oldgirl
userPassword: {SSHA}2pvE4C6xy8dkmW2aD/eEocVajg8BujWW
uidNumber: 10005
gidNumber: 10001
sn: oldgirl
EOF
```

***

> **提示:**

- 以上信息中ldap用户1：user01密码`111`，用户2：oldgirl 密码`123456`
- 这些原始信息是如何获得的？

***

3.8.3检查初始化的ldap测试数据。

查询导入的结果：

```
ldapsearch -LLL -w 123 -x -H ldap://10.0.0.4 -D "cn=admin,dc=cylon,dc=com" -b "dc=cylon,dc=com"
```

**ldapsearch命令参数说明**

| 参数 | 说 明                                            |
| :--: | ------------------------------------------------ |
|  -W  | 指定密码，交互式，不需要在命令上写密码           |
|  -w  | 指定密码，需要命令上指定密码                     |
|  -H  | ldapapi                                          |
|  -D  | 所绑定的服务器的DN，如`cn=admin,dc=cylon,dc=org` |
|  -f  | -f: filename.ldif文件                            |
|  -b  | -b 指定查询的相对节点                            |
| -LLL | 以LDIF格式打印响应，不带注释                     |
|  -x  | 简单的认证                                       |


备份ldap数据库数据


```bash
ldapadd -x -H ldap://cylon.org -D "cn=admin,dc=cylon,dc=org" -W -f base.ldif >base.ldif
ldapadd -x -H ldap://cylon.org -D "cn=admin,dc=cylon,dc=org" -W -f test.ldif >test.ldif
```

