# ch0 understanding openldap - introduction


## 什么是目录服务？

==目录是一类为了浏览和搜索数据而设计的特殊的数据库==，例如：为人所熟知的微软公
司的活动目录（active directory）就是目录数据库的一种，目录服务是按照==树状形式==存储信息的，目录包含基于属性的描述性信息，并且支持高级的过滤功能。

- http://www.openldap.org/doc/admin24/intro.html

一般来说，目录不支持大多数事务型数据库所支持的**高吞吐量**和复杂的更新操作。目录进行更新操作，可以说是要么全部，要么都不的原子操作。目录服务适合的业务应用在于提供大量的**查询和搜索**操作，而不是大量的写入操作。Ldap可以说是活动目录在linux系统上的一个开源实现。

为了保证目录数据的可用性和可靠性，在确保使用目录服务提供快速查询和搜索操作的同时，目录服务还提供了==主从服务器同步目录数据信息的能力==，这相当于传统的MySQL数据库的主从同步功能一样，可以最大限度的确保基于目录业务的服务持续可用性与提供并发查询能力，微软公司的活动目录（active directory）就有主域和备份域的说法。

广义的目录服务概念，可以用多种不同的方式来提供目录服务。不同的目录所允许存储的信息是不同的，在信息如何被引用、查询、更新以及防止未经授权的访问等问题上，不同的目录服务的处理方式也有诸多的不同。

例如：一些目录服务是本地的，只提供受限的服务（比如，单机上的finger服务）。另一些服务是大范围的（global），提供广阔得多的服务（比如面向整个因特网），大范围的服务通常是分布式的，这也就意味着数据是分布在多台机器上的，这些机器一起来提供目录服务，典型的大范围服务定义一个统一的名称空间（namespace）来给出一个相同的数据视图（data view），而不管你相对于数据所在的位置。DNS是一个典型的大范围分布式目录服务的例子。

- http://www.openldap.org/faq/data/cache/595.html


## 什么是ldap？

目录服务有两个国际标准，分别是`X.500`和`LDAP`。X.500是ITU定义的目录标准，而LDAP是基于TCP/IP的目录访问协议，是Intemet上目录服务的通用访问协议。

LDAP是 `Lightweight Directory Access Protocol`（轻量级目录访问协议）的缩写。正如它的名字所表明的那样，它是一个轻量级的目录访问协议，特指基于X.500的目录访问协议的简化版本。LDAP运行在 `TCP/IP` 或者其他的面向连接的传输服务之上。LDAP完整的技术规范由RFC2251“The Lightweight Directory Access Protocol（v3）”和其他几个在RFC3377中定义的文档组成。

- LDAP是轻量目录访问协议（Lielhtweiglht Directory Access Protocol）的缩写。
- LDAP标准实际上是在X.500标准基础上产生的一个简化版本。

## 什么是X.500？

X.500由ITU-T和ISO定义，它实际上不是一个协议，而是由一个==协议族==组成，包括了从X.501到X.525等一系列非常完整的目录服务协议。

从技术上来说，LDAP是一个到X.500目录服务的目录访问协议，X.500是一个OSI目录服务。最初，LDAP客户端通过网关访问X.500目录服务。网关在客户端和网关之间运行LDAP，而X.500目录访问协议（Directory Access Protocol，DAP），位于这个网关和X.500服务器之间。

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/8778)

LDAP是一个重量级的协议，在整个OSI协议栈上进行操作，而且需要占用大量的计算资源。而LDAP被设计为在TCP/IP层上操作，以小得多的代价实现了大多数LDAP的功能。

虽然LDAP仍旧可以通过网关访问X.500目录服务器，但是现在通常都是在X.500服务器上直接实现LDAP。

单独的LDAP守护程序openldap slapd，可以被看做是一个轻量级的X.500目录服务器。也就是说，它没有实现X.500完整的DAP协议。作为一个轻量级的目录服务器，slapd实现的仅仅是X.500模型的一个子集。


LDAP中的常用名词缩写及含义。

LDAP基本概念中的常用名词缩写及含义

| 关&nbsp;键字 | &nbsp;&nbsp;英文全称 | 含&nbsp;义                                                   |
| ------------ | -------------------- | ------------------------------------------------------------ |
| dc           | Domain Component     | 城名的部分，其格式是将完整的城名分成几部分，如域名为exaple.com变成`dc=exaple,dc=com` |
| uid          | User Id              | 用户ID，如 “oldboy”。                                        |
| ou           | Organization Unit    | 组织单位，类似于Limux文件系统中的子目录，它是一个容器对象，组织单位可以包含其他各种对象（包括其他组织单元），如“tech，rongjunfeng，bingge” |
| cn           | Common Name          | 公共名称，如 “Thowas Johansson”                              |
| sn           | Surnase              | 姓，如 “Johansson”                                           |
| dn           | Distinguished Name   | 唯一辨别名，类似于Linux文件系统中的绝对路径，每个对象都有一个唯一的名称，如`uid=tos,ou=market,dc=example,dc=com`，在一个目录树中 总是唯一的。 |
| rdn          | Relative  dn         | 相对辨别名，类似于文件系统中的相对路径，它是与目录树结构无关的部分，如`uid=tom`或`cn=Thoeas Johansscn`。 |
| c            | Country              | 国家，如“CN”或“US”等                                         |
| o            | organizatione        | 组织名，如“Example，Inc.”。                                  |

## LDAP目录服务的特点

LDAP（openldap）目录服务具有下列特点：

- LDAP是一个跨平台的、标准的协议，近几年来得到了业界广泛的认可。
- LDAP的结构用树型结构来表示，而不是用表格。因此不用SQL语句维护了。
- LDAP提供了静态数据的快速查询方式，但在更新数据方面并不擅长。
- LDAP服务可以使用基于“推”或“拉”的复制信息技术，用简单的或基于安全证书的安全认证，复制部分或全部数据，既保证了数据的安全性，又提高了数据的访问效率：
- LDAP是一个安全的协议，LDAPv3支持SASL（Simple Authentication and Security Layer）、SSL（Secure Socket Layer）和TLS（Transport Layer Security），使用认证来确保事务的安全，另外，LDAP提供了不同层次的访问控制，以限制不同用户的访问权限。
- LDAP支持异类数据存储，LDAP存储的数据可以是文本资料、二进制图片等。
- Client/Server 模型：Server端用于存储树，Client端提供操作目录信息树的工具，通过这些工具，可以将数据库的内容以文本格式（LDAP数据交换格式，LDIF）呈现在我们的面前。
- LDAP是一种开放Internet 标准，LDAP协议是跨平台的的Interent协议，它是基于X.500标准的，与X.500不同，LDAP支持TCP/IP（即可以分布式部署）。

## LDAP的目录结构

LDAP目录服务是通过目录数据库来存储网络信息来提供目录服务的。为了方便用户迅速查找和定位信息，目录数据库是以目录信息树（Directory Infornation Tree，缩写为DIT）为存储方式的树型存储结构，目录信息树及其相关概念构成了LDAP协议的信息模型。

- 在LDAP中，目录是按照树型结构组织一一目录信息树（Directory Information Tree简写DIT），DIT是一个主要进行读操作的数据库。
- DIT由条目（Entry）组成，条目相当于关系数据库中的表的记录：

条目是具有分辨名DN（Distinguished Name）的属性-值对（Attribute-value，简称AV）
的集合。

在UNIX文件系统中，最项层是根目录（root），LDAP目录通常也用ROOT做根，通常称为`BaseDN`。

因为历史（X.500）的原因，LDAP目录用OU（Organization Unit）从逻辑上把数据分开来。Ou也是一种条目--==容器条目==。Ou下面即是真正的用户条目。


> **什么是dn?**

DN，Distinguished Name，即分辨名。

在LDAP中，一个条目的分辨名叫做“DN”，DN是该条目在整个树中的唯一名称标识，DN相当于关系数据席表中的关键字（Primary Key）；它是一个识别属性，通常用于检索。


>  **DN的两种设置**

基于cn（姓名），`cn=test,ou=auth,dc=cylon,dc=org`，最常见的cn是从 `/etc/group` 转来的条目。.

基于uid（User ID），`uid=test,ou=auth,dc=cylon,dc=org` 最常见的uid是 `/etc/passwd` 转来的条目。

>  **Base DN**

LDAP目录树的最顶部就是根，也就是Base DN.

>  **LDIF格式**

LDIF格式是用于LDAP数据导入、导出的格式。LDIF是LDAP数据库信息的一种文本格式。BDB。


>  **什么样的信息可以存储在目录当中？**

LDAP的信息模型是基于条目的（entry）。一个条目就是一些具有全局唯一的标识名（Distinguished Name，简写做==DN==）的属性的集合。DN用于无二义性的指代一个唯一的条目，条目的每一个属性都有一个类型（type），一个或者多个值（value），类型（type）往往是特定字符串的简写，比如用 `cn` 指代 `commpn name`，或者 `mail` 指代电子邮件地址。值（value）的语法依赖于类型（type），比如，类型为cn的属性可能包含值Babs Jensen。
类型为mail的属性可能包含值 `babs@example.com` 。类型为jpegPhoto的属性可能包含二进制格式的JPEG图象。

下面是LDAP中条目信息的例子；就相当于数据库表中的两行记录：

LDAP允许你通过使用一种叫做 `objectClass`的特殊属性来控制哪些属性是条目所必须的，哪些属性是条目可选的，`objectClass` 属性的值是由条目所必须遵从的方案（schema）
来建义的。

通过下面命令可以取出上面的内容。

```sh
ldapsearch -LLL -w oldboy -x -H ldap://10.0.0.20 -D "cn=admin,dc=cylon,dc=org" -b "dc=cylon,dc=org" uid="*"
```

信息在目录中是如何组织的？

在LDAP中，条目是按树状的层次结构组织的，传统上，这个结构往往是地理界限或者组织界限的反映。代表国家的条目位于整个目录树的顶层。之下的条目则代表各个州以及国家性的组织，再下面的条目则代表着组织单位、个人打印机、文件，或者你所能想到的其他东西，图1.1显示了按照传统命名方式组织的LDAP目录信息树。

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/intro_tree.png)

<center>图1.1：LDAP目录树（传统命名方式）</center>


目录树也可以按照因特网域名结构组织。因为它允许按照DNS对目录服务进行定位，这种命名方式正变得越来越受欢迎。图1.2显示了按照域名进行组织的一个LDAP目录树的例子。


![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/intro_dctree.png)



- http://www.openldap.org/doc/admin24/intro.html



root -> 顶级域 -> 二级 -> ou -> uid 


![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567525992883.png)

<center>图1-2：LDAP目录树（域名命名方式）</center>


例：`dn:cn=testlb,ou=People,ou=accounts,dc=cylon,dc=org`

另外，LDAP允许你通过使用一种叫做objectClass的特殊属性来控制哪些属性是条目所必须的，哪些属性是条目可选的。objectClass属性的值是由条目所必须遵从的方案
（schema）来定义的。.

（3）信息是如何械引用的？

一个条目是通过它的标识名来引用的。而标识名是由相对标识名（Relative Distinguished Name或者RDN）和它的父条目名连在一起构成的。比如，在因特网命名的例子中，Barbara Jensen条目有相对标识名 `uid=babs` 和标识名为：`uid=babs,ou=People,de=example,dc=com` 。

### LDIF数据文件介绍

目录数据文件内容讲解  

LDIF（LDAP Data Interchangep-omat）轻量级目录交换格式）一种ASCII文件格式，用来交换数据并使得在LDAP服务器间交换数据成为可能。

LDIF文件最常用的功能就是向目录导入或修改信息，这些信息的格式需要按照LDAP中架构（schema）格式组织，如果不符合其要求的格式就会出现错误。

LDIF文件的特点：

- 通过空行来分割一个条目或定义。
- 以`#`开始的行为注释。
- 所有属性的赋值方法为：`"属性: 属性值"` 例如，dn:`dn=cylon,dc=org`
- 属性可以被重复赋值。例如objectclass就可以有多个，每个属性独立一行。
- 每行的结尾不允许有空格。

在LDAP的每条记录中必须包含一个objectclass属性，且其需要赋子至少一个值。objectclass属性有等级之分，属性相当于变量，它可以被自定义赋值，值不能相同。

目录数据展现的树形结构圆。以上ldif数据文件表现个内容为如下树形结构：ldap C/S工具呈现的。


## LDAP是想样工作的？

LDAP目录服务是基于 C/S模式的。一个或者多个LDAP服务器包含着组成整个目录信息树（DIT）的数据，客户端连接到服务器并且发出一个请求（request），然后服务器要么以一个回答（answer）子以回应，要么给出一个指针，客户可以通过此指针获取到所需的数据（通常，该指针是指向另一个LDAP服务器），无论客户端连到哪个LDAP服务器，它看到的都是同一个目录视图（view）.这是LDAP这类全局目录服务的一个重要特征。


## LDAP的几个重要配置模式

LDAP服务的几个重要功能：

- ★★★基本的目录查询服务。
- 目录查询代理服务。
- 异机复制数据（即主从同步）。

#### 本地基本的目录查询服务。

在这种配置模式下，你的slapd只为你的本地域提供目录服务。它不会以任何方式与别的目录服务器交互。这种配置模式如图1所示。

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567525992658.png)

<center>本地配置模式</center>

### 带有指针（Referrals）的本地目录服务。

即目录查询代理服务，类似DNS的转发服务器。

在这种配置模式下，你为你的本地域运行一个LDAP服务器，并且将它配置成为当客户的请求超出你的本地域的处理能力的时候能够返回一个指针，该指针指向一个具备处理客户请求能力的更高级的服务器的地址。你可以自己运行这一服务，也可以使用别人已提供给你的一个。这种配置模式如图2所示。：

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567525992983.png)

如果你想运行本地目录服务并且参与全属的目录，那么运行这种模式。例如：openldap作为微软活动目录的代理查询服务。

```
referral <URI>
```

该指令指定了一个指针，当服务器的slapd找不到一个本地的数据库来处理一个请求的时候，它把该指针回传给客户。

示例：

```
referral ldap://root.openldap.org
```

这将把非本地的请求“推”到OpenLDAP的根服务器上。“聪明的” LDAP客户会向反馈回来的指针所指的服务器重新发出请求。但是得注意大多数客户仅仅知道怎么样处理简单的LDAP的URL，其中包含主机部分和可选的DN部分。

### 同多复制的目录服务。

slarpd守护程序是用来将主slapd上的改变传播到一个或多个从属的slapd上。一个master-slave 类型的配置示例如图3所示。

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/9100)

这种配置模式可以和前面的两种配置模式之一合起来使用，在前面的两种情况中，单独的slapd不能提供足够的可用性和可靠性。

### 简单易用的同步复制目录方案。

利用inotify+ldap客户端命令方案，或者通过定时任务加上ldap客户端命令方案！举MYSQL同步的例子说明！


![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567525992939.png)


### 分布式的目录服务。

在这种配置模式下，本地的服务被分制成为多个更小的服务，每一个都可能被复制，并且通过上级（superior）或者下级（subordinate）指针（referral）粘合起来。在实际工作中，使用主从同步集群比较多一些，跨机房就是通过openvpn同步。

ldap企业架构逻辑图案例：ldap+haproxy/nginx/hearbeat集群高可用，验证的时候不跨机房。

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567525993061.png)

## LDAP服务的应用领域

LDAP目录服务，适合那些需要从不同的地点读取信息，但是不需要经常更新的业务信息最为有用。

LDAP的应用主要涉及以下几种类型。I

- 信息安全类：数字证书管理、授权管理、单点登录。
- 科学计算类：DCE（Distributed Computing Environment，分布式计算环境）、UDDI（Universal Description，Discovery and Integration，统一描述、发现和集成协议）。
- 网络资源管理类：MAIL系统、DNS系统、网络用户管理、电话号码簿。
- 电子政务资源管理类：内网组织信息服务、电子政务目录体系、人口基础库、法人基础库。

在工作中，常用ldap作为**公司入职后的所有员工账号等的基础信息库**，例如：邮件账号，电脑登陆账号、办公平台账号、共享服务账号、SVN账号、VPN账号、服务器的账号，无线网登陆账号等的公共账号登陆信息库。可以理解为企业的活动目录一样，另外，LDAP也可以和微软活动目录打通。

画一个内网AD域，和机房的LDAP服务打通的架构图！


## LDAP服务的常见开源产品

Openldap是LDAP最好的开源实现，在其Openldap许可证下发行，并且已经被包含在众多流行的linux发行版中

官方网站为 [openldap.org](http://www.openldap.org)

官方man手册 http://www.openldap.org/software/man.cgi

## openldap UI

ldap的客户端管理接口有很多，有b/s结构的web的，也有c/s结构的，我们以b/s结构的 `ldap-account-manager-3.7.tar.gz` 软件为例进行讲解

- B/S 架构的 [LDAP Account Manager](https://www.ldap-account-manager.org/lamcms/)
- C/S 架构的 [LDAP Admin](http://www.ldapadmin.org/index.html)

### LDAP Account Manager

LAM是php服务需要安装lamp环境

```bash
yum install -y\
	nginx \
	php \
	php-ldap \
	php-gd \
	php-fpm

rpm -qa \
	nginx \
	php \
	php-ldap \
	php-gd \
	php-fpm
```

### 下载解压配置ldap客户端软件

下载地址：https://www.ldap-account-manager.org/lamcms/releases

```
cp config.cfg_sample config.cfg
cp lam.conf_sample lam.conf

sed -i 's@cn=Manager@cn=admin@g' lam.conf
sed -i 's@dc=my-domain@dc=cylon@g' lam.conf
sed -i 's@dc=com@dc=org@g' lam.conf

diff lam.conf_sample lam.conf
```

***

> **提示**： 此处改的地方很多

---

### 配置nginx

```
chown nginx.nginx /var/www/html/ldap -R 
```

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   /var/www/html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
		
		location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
		}
    }
}
```

### LAM使用

**步骤1**：打开浏览器或者在前面的访问基础上刷新：http:/10.0.0.20/ldap，正常会出现界面如下图所示：

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251834.png" alt="image" style="zoom: 80%;" />

**步骤2**：点击右上角的“LAMconfiguration”链接进行配置。

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221023180610798.png" alt="image-20221023180610798" style="zoom:67%;" />

**步骤3**：点击上图中的 “Edit general settings” 链接进行配置

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251647.png" alt="image" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251610.png" alt="image"  />

由于系统的密码为字母 “lam”，太简单了，我们先把密码改掉，其他的配置，我们根据需求以后可以再改，注意：这里是系统设置的权限密码，非页面上登录的密码。

**步骤5**：上文点OK后就会到重新登录的界面进行登录，如下图：

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251845.png" alt="image" style="zoom:80%;" />

***

> **提示**：
>
> 请牢记，这是 ==ldap管理员用户admin及密码==，非前面设置的系统内部配置密码，ldap 管理员用户admin及密码是在配置openldap中生成的

***

**步骤6**：初始化ldap数据库的域。

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251713.png" alt="image" style="zoom:80%;" />

**步骤7**：命令查看下刚才的操作帮我们建立了什么？。

```
$ ldapsearch -LLL -w 123 -x -H ldap://10.0.0.20 -D "cn=admin,dc=cylon,dc=org" -b "dc=cylon,dc=org" "(uid=chau)" 
dn: uid=chau,ou=People,dc=cylon,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/chau
loginShell: /bin/bash
cn: cylon chau
uidNumber: 10006
gidNumber: 10000
userPassword:: e1NTSEF9c0YzZzBSbnFBU1NoWEVBWEFSS1lPdG41WjZRR1pxNCs=
sn: chau
givenName: cylon
uid: chau
```

**步骤8**：继续添加ldap账号和组。

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251790.png" alt="image" style="zoom:80%;" />


```
dn: cn=tech,ou=group,dc=cylon,dc=org
objectClass: posixGroup
description:: 5oqA5pyv6Y0o
gidNumber: 10001
cn: tech
```

添加用户

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527251960.png" alt="image" style="zoom:80%;" />

<img src="https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527252022.png" alt="image" style="zoom:80%;" />

```
$ ldapsearch -LLL -w 123 -x -H ldap://10.0.0.20 -D "cn=admin,dc=cylon,dc=org" -b "dc=cylon,dc=org" "(uid=cylon)"
dn: uid=cylon,ou=People,dc=cylon,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/chau
loginShell: /bin/bash
cn: cylon chau
uidNumber: 10006
userPassword:: e1NTSEF9c0YzZzBSbnFBU1NoWEVBWEFSS1lPdG41WjZRR1pxNCs=
sn: chau
givenName: cylon
uid: cylon
gidNumber: 10001
```

通过web接口管理ldap的配置完成！也已经初始化了用户组及用户

### phpldapadmin

```sh
yum install -y \
    php \
    php-fpm \
    php-ldap \
    php-gd \
    php-mbstring \
    php-pear \
    php-bcmath \
    php-xml \
    phpldapadmin \
    nginx
```

```nginx
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        root   /usr/share/phpldapadmin/htdocs;
        location / {
            index  index.html index.htm index.php;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }
}
```

