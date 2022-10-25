# 


[TOC]

## 1 ldap目录服务介绍

### 1.1 什么是目录服务？

==目录是一类为了浏览和搜索数据而设计的特殊的数据库==，例如：为人所熟知的微软公
司的活动目录（active directory）就是目录数据库的一种，目录服务是按照==树状形式==存储信息的，目录包含基于属性的描述性信息，并且支持高级的过滤功能。

- http://www.openldap.org/doc/admin24/intro.html

一般来说，目录不支持大多数事务型数据库所支持的**高吞吐量**和复杂的更新操作。目录进行更新操作，可以说是要么全部，要么都不的原子操作。目录服务适合的业务应用在于提供大量的**查询和搜索**操作，而不是大量的写入操作。Ldap可以说是活动目录在linux系统上的一个开源实现。

为了保证目录数据的可用性和可靠性，在确保使用目录服务提供快速查询和搜索操作的同时，目录服务还提供了==主从服务器同步目录数据信息的能力==，这相当于传统的MySQL数据库的主从同步功能一样，可以最大限度的确保基于目录业务的服务持续可用性与提供并发查询能力，微软公司的活动目录（active directory）就有主域和备份域的说法。

广义的目录服务概念，可以用多种不同的方式来提供目录服务。不同的目录所允许存储的信息是不同的，在信息如何被引用、查询、更新以及防止未经授权的访问等问题上，不同的目录服务的处理方式也有诸多的不同。

例如：一些目录服务是本地的，只提供受限的服务（比如，单机上的finger服务）。另一些服务是大范围的（global），提供广阔得多的服务（比如面向整个因特网），大范围的服务通常是分布式的，这也就意味着数据是分布在多台机器上的，这些机器一起来提供目录服务，典型的大范围服务定义一个统一的名称空间（namespace）来给出一个相同的数据视图（data view），而不管你相对于数据所在的位置。DNS是一个典型的大范围分布式目录服务的例子。

- http://www.openldap.org/faq/data/cache/595.html


### 1.2 什么是ldap？

目录服务有两个国际标准，分别是`X.500`和`LDAP`。X.500是ITU定义的目录标准，而LDAP是基于TCP/IP的目录访问协议，是Intemet上目录服务的通用访问协议。

LDAP是 `Lightweight Directory Access Protocol`（轻量级目录访问协议）的缩写。正如它的名字所表明的那样，它是一个轻量级的目录访问协议，特指基于X.500的目录访问协议的简化版本。LDAP运行在 `TCP/IP` 或者其他的面向连接的传输服务之上。LDAP完整的技术规范由RFC2251“The Lightweight Directory Access Protocol（v3）”和其他几个在RFC3377中定义的文档组成。

- LDAP是轻量目录访问协议（Lielhtweiglht Directory Access Protocol）的缩写。
- LDAP标准实际上是在X.500标准基础上产生的一个简化版本。

### 1.3 什么是X.500？

X.500由ITU-T和ISO定义，它实际上不是一个协议，而是由一个==协议族==组成，包括了从X.501到X.525等一系列非常完整的目录服务协议。

从技术上来说，LDAP是一个到X.500目录服务的目录访问协议，X.500是一个OSI目录服务。最初，LDAP客户端通过网关访问X.500目录服务。网关在客户端和网关之间运行LDAP，而X.500目录访问协议（Directory Access Protocol，DAP），位于这个网关和X.500服务器之间。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/8778)

DAP是一个重量级的协议，在整个0SI协议栈上进行操作，而且需要占用大量的计算资源。而LDAP被设计为在TCP/IP层上操作，以小得多的代价实现了大多数DAP的功能。

虽然LDAP仍旧可以通过网关访问X.500目录服务器，但是现在通常都是在X.500服务器上直接实现LDAP。

单独的LDAP守护程序openldap slapd，可以被看做是一个轻量级的X.500目录服务器。也就是说，它没有实现X.500完整的DAP协议。作为一个轻量级的目录服务器，slapd实现的仅仅是X.500模型的一个子集。


1.6 LDAP中的常用名词缩写及含义。

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

### 1.4 LDAP目录服务的特点

LDAP（openldap）目录服务具有下列特点：

- LDAP是一个跨平台的、标准的协议，近几年来得到了业界广泛的认可。
- LDAP的结构用树型结构来表示，而不是用表格。因此不用SQL语句维护了。
- LDAP提供了静态数据的快速查询方式，但在更新数据方面并不擅长。
- LDAP服务可以使用基于“推”或“拉”的复制信息技术，用简单的或基于安全证书的安全认证，复制部分或全部数据，既保证了数据的安全性，又提高了数据的访问效率：
- LDAP是一个安全的协议，LDAPv3支持SASL（Simple Authentication and Security Layer）、SSL（Secure Socket Layer）和TLS（Transport Layer Security），使用认证来确保事务的安全，另外，LDAP提供了不同层次的访问控制，以限制不同用户的访问权限。
- LDAP支持异类数据存储，LDAP存储的数据可以是文本资料、二进制图片等。
- Client/Server 模型：Server端用于存储树，Client端提供操作目录信息树的工具，通过这些工具，可以将数据库的内容以文本格式（LDAP数据交换格式，LDIF）呈现在我们的面前。
- LDAP是一种开放Internet 标准，LDAP协议是跨平台的的Interent协议，它是基于X.500标准的，与X.500不同，LDAP支持TCP/IP（即可以分布式部署）。

### 1.5 LDAP的目录结构

LDAP目录服务是通过目录数据库来存储网络信息来提供目录服务的。为了方便用户迅速查找和定位信息，目录数据库是以目录信息树（Directory nfornation Tree，缩写为DIT）为存储方式的树型存储结构，目录信息树及其相关概念构成了LDAP协议的信息模型。

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

```ldif
a:uld=oldgirl,ou=People,dc=etiantia,dc=org      
    obcjectClass:posixAccount 
    objectClass:inetOrgPerscn objectClass；organi zationalPerson 
    cbjectClass；persen heseDirectory:/hese/eldgirl lezinshell:/ein/eath uid:oldeirl 
    cn:oldgirl userPssoword:：elNTs2P9TBOVT g3e jhoT jUl-R6TVpMe/B SewRto7excj=Ve 
    uidkaber：10005
    sidkumber：10001
    m：0ldgirl。
```

LDAP允许你通过使用一种叫做 `objectClass`的特殊属性来控制哪些属性是条目所必须的，哪些属性是条目可选的，`objectClass` 属性的值是由条目所必须遵从的方案（schema）
来建义的。

通过下面命令可以取出上面的内容。

```sh
ldapsearch -LLL -w oldboy -x -H ldap://10.0.0.20 -D "cn=admin,dc=cylon,dc=org" -b "dc=cylon,dc=org" uid="*"
```

信息在目录中是如何组织的？

在LDAP中，条目是按树状的层次结构组织的，传统上，这个结构往往是地理界限或者组织界限的反映。代表国家的条目位于整个目录树的顶层。之下的条目则代表各个州以及国家性的组织，再下面的条目则代表着组织单位、个人打印机、文件，或者你所能想到的其他东西，图1.1显示了按照传统命名方式组织的LDAP目录信息树。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/intro_tree.png)

<center>图1.1：LDAP目录树（传统命名方式）</center>


目录树也可以按照因特网域名结构组织。因为它允许按照DNS对目录服务进行定位，这种命名方式正变得越来越受欢迎。图1.2显示了按照域名进行组织的一个LDAP目录树的例子。


![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/intro_dctree.png)



- http://www.openldap.org/doc/admin24/intro.html



root -> 顶级域 -> 二级 -> ou -> uid 


![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567525992883.png)

<center>图1-2：LDAP目录树（域名命名方式）</center>


例：`dn:cn=testlb,ou=People,ou=accounts,dc=cylon,dc=org`

另外，LDAP允许你通过使用一种叫做objectClass的特殊属性来控制哪些属性是条目所必须的，哪些属性是条目可选的。objectClass属性的值是由条目所必须遵从的方案
（schema）来定义的。.

（3）信息是如何械引用的？

一个条目是通过它的标识名来引用的。而标识名是由相对标识名（Relative Distinguished Name或者RDN）和它的父条目名连在一起构成的。比如，在因特网命名的例子中，Barbara Jensen条目有相对标识名 `uid=babs` 和标识名为：`uid=babs,ou=People,de=example,dc=com` 。

LDIF数据文件介绍

目录数据文件内容讲解  

LDIF（LDAP Data Interchangep-omat）轻量级目录交换格式）一种ASCII文件格式，用来交换数据并使得在LDAP服务器间交换数据成为可能。

LDIF文件最常用的功能就是向目录导入或修改信息，这些信息的格式需要按照LDAP中架构（schema）格式组织，如果不符合其要求的格式就会出现错误。

#### LDIF文件的特点：

- 通过空行来分割一个条目或定义。
- 以`#`开始的行为注释。
- 所有属性的赋值方法为：`"属性: 属性值"` 例如，dn:`dn=cylon,dc=org`
- 属性可以被重复赋值。例如objectclass就可以有多个，每个属性独立一行。
- 每行的结尾不允许有空格。

在LDAP的每条记录中必须包含一个objectclass属性，且其需要赋子至少一个值。objectclass属性有等级之分，属性相当于变量，它可以被自定义赋值，值不能相同。

目录数据展现的树形结构圆。以上ldif数据文件表现个内容为如下树形结构：ldap C/S工具呈现的。


### 1.6 LDAP是想样工作的？

LDAP目录服务是基于 C/S模式的。一个或者多个LDAP服务器包含着组成整个目录信息树（DIT）的数据，客户端连接到服务器并且发出一个请求（request），然后服务器要么以一个回答（answer）子以回应，要么给出一个指针，客户可以通过此指针获取到所需的数据（通常，该指针是指向另一个LDAP服务器），无论客户端连到哪个LDAP服务器，它看到的都是同一个目录视图（view）.这是LDAP这类全局目录服务的一个重要特征。


### 1.7 LDAP的几个重要配置模式

LDAP服务的几个重要功能：

- ★★★基本的目录查询服务。
- 目录查询代理服务。
- 异机复制数据（即主从同步）。

#### 1.7.1 本地基本的目录查询服务。

在这种配置模式下，你的slapd只为你的本地域提供目录服务。它不会以任何方式与别的目录服务器交互。这种配置模式如图1所示。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567525992658.png)

<center>本地配置模式</center>

#### 1.7.2 带有指针（Referrals）的本地目录服务。

即目录查询代理服务，类似DNS的转发服务器。

在这种配置模式下，你为你的本地域运行一个LDAP服务器，并且将它配置成为当客户的请求超出你的本地域的处理能力的时候能够返回一个指针，该指针指向一个具备处理客户请求能力的更高级的服务器的地址。你可以自己运行这一服务，也可以使用别人已提供给你的一个。这种配置模式如图2所示。：

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567525992983.png)

如果你想运行本地目录服务并且参与全属的目录，那么运行这种模式。例如：openldap作为微软活动目录的代理查询服务。

referral <URI>.

该指令指定了一个指针，当服务器的slapd找不到一个本地的数据库来处理一个请求的时候，它把该指针回传给客户。

示例：

referral ldap://root.openldap.org

这将把非本地的请求“推”到OpenLDAP的根服务器上。“聪明的” LDAP客户会向反馈回来的指针所指的服务器重新发出请求。但是得注意大多数客户仅仅知道怎么样处理简单的LDAP的URL，其中包含主机部分和可选的DN部分。


1.10 同多复制的目录服务。

slarpd守护程序是用来将主slapd上的改变传播到一个或多个从属的slapd上。一个master-slave 类型的配置示例如图3所示。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/9100)

这种配置模式可以和前面的两种配置模式之一合起来使用，在前面的两种情况中，单独的slapd不能提供足够的可用性和可靠性。

10.4 简单易用的同步复制目录方案。

利用inotify+ldap客户端命令方案，或者通过定时任务加上ldap客户端命令方案！举MYSQL同步的例子说明！


![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567525992939.png)


1.10.5分布式的目录服务。

在这种配置模式下，本地的服务被分制成为多个更小的服务，每一个都可能被复制，并且通过上级（superior）或者下级（subordinate）指针（referral）粘合起来。在实际工作中，使用主从同步集群比较多一些，跨机房就是通过openvpn同步。

ldap企业架构逻辑图案例：ldap+haproxy/nginx/hearbeat集群高可用，验证的时候不跨机房。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567525993061.png)

1.11 LDAP服务的应用领域

LDAP目录服务，适合那些需要从不同的地点读取信息，但是不需要经常更新的业务信息最为有用。

LDAP的应用主要涉及以下几种类型。I
- 信息安全类：数字证书管理、授权管理、单点登录。
- 科学计算类：DCE（Distributed Computing Environment，分布式计算环境）、UDDI（Universal Description，Discovery and Integration，统一描述、发现和集成协议）。
- 网络资源管理类：MAIL系统、DNS系统、网络用户管理、电话号码簿。
- 电子政务资源管理类：内网组织信息服务、电子政务目录体系、人口基础库、法人基础库。



在工作中，常用ldap作为**公司入职后的所有员工账号等的基础信息库**，例如：邮件账号，电脑登陆账号、办公平台账号、共享服务账号、SVN账号、VPN账号、服务器的账号，无线网登陆账号等的公共账号登陆信息库。可以理解为企业的活动目录一样，另外，LDAP也可以和微软活动目录打通。

画一个内网AD域，和机房的LDAP服务打通的架构图！


### 1.8 LDAP服务的常见开源产品

Openldap是LDAP最好的开源实现，在其Openldap许可证下发行，并且已经被包含在众多流行的linux发行版中，官方网站为 [openldap.org](http://www.openldap.org)

- http://www.openldap.org/doc


官方man手册

- http://www.openldap.org/software/man.cgi

```
[root@node06 openldap-2.4.47]# ldapsearch -x -b "" -s base "(objectclass=*)" namingCointexts
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: namingCointexts 
#

#
dn:

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```


## openldap管理工具

### 3.1 为ldap master 配置web管理接口

ldap的客户端管理接口有很多，有b/s结构的web的，也有c/s结构的，我们以b/s结构的 `ldap-account-manager-3.7.tar.gz` 软件为例进行讲解。

### 3.2 安装lamp服务环境。

```bash
yum install nginx php php-ldap php-gd php-fpm -y

rpm -qa nginx php php-ldap php-gd php-fpm
```

### 3.3 下载解压配置ldap客户端软件

下载地址

- https://www.ldap-account-manager.org/lamcms/releases

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
***

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


步骤1：

打开浏览器或者在前面的访问基础上刷新：http:/10.0.0.20/ldap，正常会出现界面如下图所示：

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251834.png)

步骤2

点击右上角的“LAMconfiguration”链接进行配置。

![image-20221023180610798](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221023180610798.png)

步骤3：

点击上图中的“Edit general settings”链接进行配置。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251647.png)

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251610.png)

由于系统的密码为字母“lam”，I太简单了，我们先把密码改掉，其他的配置，我们根据需求以后可以再改，注意：这里是系统设置的权限密码，非页面上登录的密码。

步骤5：上文点OK后就会到重新登录的界面进行登录，如下图：.

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251845.png)

***
> **提示**：
>
> 请牢记，这是==ldap管理员用户admin及密码==，非前面设置的系统内部配置密码，1dap 管理员用户admin及密码是在配置LDAP时生成的。
***


步骤6：初始化1dap数据库的域。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251713.png)

步骤7：命令查看下刚才的操作帮我们建立了什么？。

```
[root@node06 ~]# ldapsearch -LLL -w 123 -x -H ldap://10.0.0.20 -D "cn=admin,dc=cylon,dc=org" -b "dc=cylon,dc=org" "(uid=chow)" 
dn: uid=chow,ou=People,dc=cylon,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/schow
loginShell: /bin/bash
cn: scott chow
uidNumber: 10006
gidNumber: 10000
userPassword:: e1NTSEF9c0YzZzBSbnFBU1NoWEVBWEFSS1lPdG41WjZRR1pxNCs=
sn: chow
givenName: scott
uid: chow
```

步骤8：继续添加1dap账号和组。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251790.png)


```
dn: cn=tech,ou=group,dc=cylon,dc=org
objectClass: posixGroup
description:: 5oqA5pyv6Y0o
gidNumber: 10001
cn: tech
```

添加用户

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527251960.png)

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527252022.png)

```
[root@node06 ~]# ldapsearch -LLL -w 123 -x -H ldap://10.0.0.20 -D "cn=admin,dc=cylon,dc=org" -b "dc=cylon,dc=org" "(uid=scott)"
dn: uid=scott,ou=People,dc=cylon,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/schow
loginShell: /bin/bash
cn: scott chow
uidNumber: 10006
userPassword:: e1NTSEF9c0YzZzBSbnFBU1NoWEVBWEFSS1lPdG41WjZRR1pxNCs=
sn: chow
givenName: scott
uid: scott
gidNumber: 10001
```

通过web接口管理ldap的配置完成！也已经初始化了用户组及用户。



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

https://yq.aliyun.com/articles/476585

https://www.i-programmer.info/programming/perl/9649-with-the-rise-of-devops-perl-shows-its-muscle.html

https://www.openldap.org/doc/admin24/replication.html

https://gihyo.jp/admin/serial/01/ldap/0020

[https://www.ilanni.com/?p=14349#%E4%B8%89%E3%80%81master%E7%9A%84%E4%B8%8A%E4%B8%BB%E4%BB%8E%E9%85%8D%E7%BD%AE](https://www.ilanni.com/?p=14349#三、master的上主从配置)

## 4 openldap配置ssl认证

创建CA证书 

```
openssl genrsa -out cakey.key 2048

openssl req -new -x509 -key cakey.key -out cacert.crt -days 3650 -subj "/C=HK/ST=HK/O=TVB/OU=bigbigchannl/CN=tvb-ca"

touch index.txt
echo "01" > serial
```

ldap创建证书请求

```bash
openssl genrsa -out openldap.key 2048
openssl req -new -key openldap.key -out openldap.csr -days 3650 -subj "/C=HK/ST=HK/O=TVB/OU=bigbigchannl/CN=10.0.0.4"

openssl ca -in openldap.csr -cert cacert.crt -keyfile cakey.key  -out openldap.crt -days 3650
```

错误 `Re: OpenLDAP/TLS main: TLS init def ctx failed: -207`

- https://www.openldap.org/lists/openldap-software/200901/msg00134.html
- https://www.openldap.org/lists/openldap-software/200812/msg00100.html

错误 `Re: slapadd: could not parse entry (line=13)`

- 可能原因，上下文存在空行

[错误](https://www.openldap.org/lists/openldap-technical/201502/msg00168.html):  `openssl slapd tls init def ctx failed: -1` ；证书的路径，文件名，权限是否正确 

验证证书 `openssl verify -CAfile {ca.crt} {.crt}`



修改服务端参数

```
olcTLSCACertificateFile: "/etc/openldap/certs/cacert.crt"
olcTLSCertificateFile: "/etc/openldap/certs/openldap.crt"
olcTLSCertificateKeyFile: "/etc/openldap/certs/openldap.key"
olcTLSVerifyClient: never
```

修改cli客户端参数 `ldap.conf`

```
URI	ldap://ldap.example.com ldaps://10.0.0.4:636

#SIZELIMIT	12
#TIMELIMIT	15
#DEREF		never

TLS_CACERTDIR	/etc/openldap/certs
TLS_CACERT /etc/openldap/certs/cacert.crt
TLS_CERT /etc/openldap/certs/openldap.csr
TLS_KEY  /etc/openldap/certs/openldap.key
```

修改服务要监听的地址

```
SLAPD_URLS="ldapi:/// ldaps:///"
```

Reference：[TLS option](https://www.openldap.org/software/man.cgi?query=slapd-config&apropos=0&sektion=0&manpath=OpenLDAP+2.4-Release&arch=default&format=html)

## 5 访问控制acl


控制对目录树中的信息的访问。谁应该能够访问记录在什么条件下他们应该能看到多少这样的记录呢 这些是我们将在本节中讨论的问题 。

OpenLDAP控制目录数据访问的主要方法是 通过访问控制列表acl 。当slapd 服务器处理来自客户机的请求时，它将评估客户机是否具有访问所请求的信息的权限。要执行此计算，slapd将依次计算confguration中的每个acl，并将适当的规则应用于传入请求。

***
> **提示：** acl策略由上而下依次进行匹配
***

所有客户端都允许读取默认的访问控制策略。不管定义了什么访问控制策略，rootdn 总是允许对所有和任何东西拥有完全的权限（即身份验证、搜索、比较、读和写 ）。

**参考地址：**
- https://www.openldap.org/software/man.cgi?query=slapd.access&sektion=5&apropos=0&manpath=OpenLDAP+2.4-Release

>  访问控制 ACL 介绍

访问控制主要定义三大方面：
- `what` 定义对那些地方的访问，部分选择应用访问的条目和/或属性
- `who` 定义人员，部分指定授予哪些实体访问
- `access` 定义权限，部分指定授予的访问。
```
access to [resources]
    by [who] [type of access granted] [control]
    by [who] [type of access granted] [control]
```

### 5.1 what控制访问规范

防问规范的`what`部分确定了应用访问控制的条目和属性。条目通常有三种选择方式：

- 通过DN
- 通过过滤器。
- 通过属性

#### 5.1.1 通过DN

**例：**

```
to *   # 选择所有
to dn[.<basic-style>]=<regex> # 使用正则表达式
to dn.<scope-style>=<DN> # 使用范围
```

***
> **注**：`to *` 通常放置在ACL策略中最下层
***

**表达式实例**

```
access to dn.regex="uid[^,]+,ou=Users,dc=example,dc=com"
```

#### 5.1.2 范围

范围分为：
- base
- one
- subtree
- children

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344410.png)





#### 5.1.3 使用filter选择条目

```
to filter=<ldap filter>
```

**例：**

```
access to filter=(objectClass=person)
access to filter="(|(|(givenName=Matt)(givenName=Barbara))(sn=Kant))"
access to dn.subtree="ou=Users,dc=example,dc=com" filter="(employeeNumber=*)"
```

#### 5.1.4 通过属性值

```
attrs=<attribute list>
attrs=<attribute> val[.<style>]=<regex>
```

**例**

```
to attrs=userPassword,shadowLastChange
```





### 5.2 who(向谁授予访问权限)


`<who>`部分标识被授予访问权限的实体。

***
> 注意，访问被授予“实体”而不是“条目”。
***

**下表总结了实体说明符**


| 说明符                       | 实体                               |
| ---------------------------- | ---------------------------------- |
| *                            | 所有，包括匿名和经过身份验证的用户 |
| anonymous                    | 匿名用户（未验证）                 |
| users                        | 通过身份验证的用户                 |
| self                         | 与目标条目关联的用户               |
| `dn[.<basic style>]=<regex>` | 匹配正则表达式的用户               |
| `dn.<scope-style>=<DN>`      | DN范围内的用户                     |


### 5.3 access（权限定义）

授予的`access`类型可以是以下类型之一
- `w`：对记录或属性的写访问。
- `r`：对记录或属性的读访问。
- `s`：对记录或属性的搜索访问。
- `c`：访问对记录或属性运行比较操作。
- `x`：访问对记录或属性执行服务器端身份验证操作。
- `d`：访问记录或属性是否存在的信息 (d 代表“披露” 。
- `0`：不允许访问记录或属性。这相当于 wrscxd 。
- `m`：管理权限

| 级别      |    权限 | 描述               |
| --------- | ------: | ------------------ |
| none=     |       0 | 禁止访问           |
| disclose= |       d | 检测信息是否存在   |
| auth=     |      dx | 需要验证（绑定）   |
| compare=  |     cdx | 需要比较           |
| search=   |    scdx | 需要应用搜索过滤器 |
| read=     |   rscdx | 需要读搜索结果     |
| write=    |  wrscdx | 需要修改/重命名    |
| manage=   | mwrscdx | 需要管理           |



### 5.4 访问控制示例


- 关闭匿名访问
- 创建一个目录管理员组，属于该组的人员都具有管理目录的权限
- 创建一个配置管理员组，属于该组的人员都具有配置 openldap 的权限
- 创建一个复制权限用户 ，该账号用于高可用之间的复制
- 创建一个搜索用户，用于 linux 集中账号认证时的，搜索
- 普通人员只具有修改自己密码的权限

***
> 提示：
- 目录管理员组：操作目录树的管理员都属于目录管理员组。
- 配置管理员组：可以操作服务器配置的（`slapd.ldif`）属于配置管理员组
***

> **关闭匿名访问**

config段

```
olcDisallows: bind_anon
```

> **普通人员修改自己的密码**

database frontend段

```
olcAccess: to attrs=userPassword,shadowLastChange
    by dn.children="cn=admin,dc=cylon,dc=com" write
    by anonymous auth
    by self write
    by * none
```

> **创建用户组**

```
olcAccess: to dn.subtree="dc=cylon,dc=com"
    by dn="cn=syncuser,ou=admin,dc=cylon,dc=com" read
    by dn="cn=clientsearch,ou=admin,dc=cylon,dc=com" read
    by group.exact="cn=ldapadmin,ou=admin,dc=cylon,dc=com" write
    by users read
```

> **配置管理员权限**

```
dn: olcDatabase=config,cn=config
....
olcAccess: to *
    by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage
# 在这句下添加
    by group.exact="cn=configadmin,ou=admin,dc=cylon,dc=org" write 
    by * none
```




默认情况下，获取schema是没有权限的，此时需要添加权限使其可以读取schema权限。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1596103387323.png)

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344473.png)

***
> **注**：在创建用户和组是需要查看schema权限
***

`slapd.ldif` 文件中添加：

```
dn: olcDatabase=frontend,cn=config
...
olcAccess: to dn.subtree=""
    by * read
```

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344427.png)

#### 错误解决

填写配置后重新生成配置文件报类似如下错误

```
# slapadd  -n0 -F /etc/openldap/slapd.d -l kerberos.ldif
SYNTAX 1.3.6.1.4.1.1466.115.121.1.26)): empty AttributeDescription
slapadd: could not parse entry (line=1)
_# 6.36% eta none elapsed none spd 18.6 M/s
Closing DB...
```

问题原因：如上配置权限换行方式不可，官网给出的解决方法：

- https://www.openldap.org/lists/openldap-technical/201705/msg00054.html

### 5.5 访问控制ACL实例


- 创建 ou (Admin)。
- 创建一个目录管理员组和配置管理员组。
- 创建一个复制用户。
- 创建 一个搜索用户。
- 在`ou=people`里面创建一个t2用户，密码`111`。
- 测试，先用t2尝试创建一个mygroup组。
- 测试，把t2属于目录管理员组，尝试创建一个group组。
- 测试，把t2属于配置管理员组，尝试修改配置文件中的日志级别。

**使用ldif 文件创建用户组**

```ldif
cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=admin,dc=cylon,dc=com
dn: ou=admin,dc=cylon,dc=com
objectClass :top
objectClass :organizationalUnit
ou: admin

dn: cn=ldapadmin,ou=admin,dc=cylon,dc=com
cn: ldapadmin
objectClass: groupOfNames
member: uid=mike,ou=People,dc=cylon,dc=com

dn: cn=clientsearch,ou=admin,dc=cylon,dc=com
cn: ldapadmin
objectClass: groupOfNames
member: uid=mike,ou=People,dc=cylon,dc=com
EOF
```

***
> **提示：** `groupOfNames` 组拥有 `member` 的属性，而 `posixGroup` 无 `member` 属性
***

**手动创建配置管理员组**

![image-20221025002951917](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025002951917.png)

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344733.png)

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344585.png)


**使用t2尝试创建一个mygroup组**

```ldif
cat << EOF | ldapadd -x -W -H ldap://10.0.0.4 -D cn=t2,ou=People,dc=cylon,dc=com
dn: cn=mygroup,ou=group,dc=cylon,dc=com
objectClass: top
objectClass: posixGroup
gidNumber: 10010
cn: mygroup
EOF
Enter LDAP Password: 
adding new entry "cn=mygroup,ou=group,dc=cylon,dc=com"
ldap_add: Insufficient access (50)
        additional info: no write access to parent
```

**将t2加入目录管理员组中**

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344582.png)

```ldif
[root@node06 openldap]# cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=t2,ou=People,dc=cylon,dc=org
dn: cn=mygroup,ou=group,dc=cylon,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 10010
cn: mygroup
EOF
Enter LDAP Password: 
adding new entry "cn=mygroup,ou=group,dc=cylon,dc=org"
```

可以看到是成功添加

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/clipboard-1567527344586.png)

使用其他用户去创建组提示是没有权限的。

```
[root@node06 openldap]# cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D uid=user01,ou=People,dc=cylon,dc=org
dn: cn=mygroup1,ou=group,dc=cylon,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 10011
cn: mygroup
EOF

Enter LDAP Password: 
adding new entry "cn=mygroup1,ou=group,dc=cylon,dc=org"
ldap_add: Insufficient access (50)
        additional info: no write access to parent
```

**使用t2尝试修改配置**

配置管理员组默认情况下，t2也没有配置权限的。

```
[root@node06 openldap]# cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=t2,ou=people,dc=cylon,dc=org
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: any
EOF
Enter LDAP Password: 
modifying entry "cn=config"
ldap_modify: Insufficient access (50)
```


### ldapACL示例参考

- http://www.uvm.edu/~fcs/tools/ACLs.html

## 6 OpenLDAP Replication

### 6.1 LDAP复制概述

复制功能允许将LDAP DIT(**Directory Information Tree**)更新复制到一个或多个LDAP系统，以实现备份和/或性能原因。在此上下文中，值得强调的是，复制在DIT级别而非LDAP服务器级别运行。因此，在运行多个DIT的单个服务器中，每个DIT可以被复制到不同的服务器。在本指南调用复制周期时间内，会定期进行复制。OpenLDAP 2.3版引入了强大的新复制功能（通常称为syncrepl），2.4版本进一步增强，以提供多主机功能。每种配置类型有两种可能的复制配置和多种变体。

***
> **注意**：2.4版之前的OpenLDAP提供了一个复制功能，它使用一个单独的守护进程，称为slurpd。此功能现在具有历史唯一意义，其描述仅适用于仍支持传统（2.3之前版本）OpenLDAP系统的功能。slurpd没有另外引用。

***

主从：在主从配置中，单个（主）DIT能够被更新，并且这些更新被复制或复制到运行从DIT的一个或多个指定服务器。从属服务器使用主DIT的只读副本进行操作。只读用户可以访问包含从DIT的服务器，但需要更新目录的用户只能通过访问包含主DIT的服务器来访问。为了使其糟糕的用户更加困惑，OpenLDAP还引入了术语提供者和消费者使用syncrepl复制功能。提供者可以被视为复制服务源（the provider）和复制服务的目的节点（或consumer）。

**主从（或provider-consumer）配置有两个明显的缺点**：

- 多位置。如果所有或大多数客户端都需要更新DIT，则他们必须访问一个服务器（从DIT）进行正常读访问，而另一个服务器（主DIT）进行更新。或者，客户端可以始终访问主DIT的服务器。在后一种情况下，复制仅提供备份功能。
- 弹性。由于只有一个服务器包含主DIT，因此它意味着存在单点故障问题。

多主：在多主机配置中，可以更新的一个或多个主DIT服务器，并将得到的更新传播到对等主机。

从历史观点上说，OpenLDAP不支持多主机操作，但2.4版本引入了多主功能。在这种情况下，OpenLDAP多主的两个通用更新争用问题，并且适用于所有LDAP系统多主实现。

- 值争用，如果在同一时间（在复制周期时间内）使用不同的值执行两个属性更新，则根据属性类型（SINGLE或MULTI-VALUED），生成的条目可能处于不正确或不可用状态。
- 删除争用，如果一个用户同时添加子条目（在复制周期时间内），则另一个用户删除原始（父）条目，则删除的条目将重新出现。

图显示了许多可能的复制配置。

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/ldap-replication.png)


- RO =只读
- RW =读写
- LDAP1面向客户端的系统是Slave并且是只读的。 客户必须向Master发出Writer。
- LDAP2面向客户端的系统是主服务器，它被复制到两个从服务器。
- LDAP3是Multi-Master，客户端可以向任一系统发出读取（搜索）和/或写入（修改）。 此配置中的每个主可以具有一个或多个从DIT。


### 6.2 Replication

复制发生在DIT级别，描述了将更新从一个LDAP服务器上的DIT复制到一个或多个其他服务器上的同一个DIT的过程。 复制配置可以是OpenLDAP特殊术语中的MASTER-SLAVE或Producer-Consumer（副本始终是只读的）或MULTI-MASTER。 复制是一个配置（操作）问题，但内容同步协议（由syncrepl使用）由RFC4533定义。

#### 6.2.1 openldap主从搭建与配置

从OpenLDAP 2.4版开始，在Consumer属性（OLC = olcSyncrepl）或指令（slapd.conf = syncrepl）调用复制之后，只有一种复制方法通常称为syncrepl。 传统版本的OpenLDAP（2.4版之前版本）提供了slurpd样式复制，现在已经过时了。

OpenLDAP2.3版引入了对新的LDAP内容同步协议的支持，从版本2.4开始，这已成为唯一支持的复制功能。 LDAP内容同步协议由RFC4533定义，消费者的功能名称olcSyncrepl/syncrepl用于调用复制。 Syncrepl功能提供经典的主从复制，因为2.4版允许多主复制。该协议使用术语提供程序（而不是主服务器）来定义复制更新的源，并使用术语使用者（而不是从服务器）来定义更新的目标。

在syncrepl样式复制中，使用者始终启动更新过程。消费者可以被配置为周期性地从提供者（refreshOnly）提取更新，或者它可以请求提供者推送更新（refreshAndPersist）。在所有复制情况下，为了明确地引用条目，服务器必须为DIT中的每个条目维护一个通用唯一编号（entryUUID）。同步过程如图

#### refreshOnly类型复制 Consumer Pull

***

<center font="bold">refreshOnly和refreshAndPersist所示：</center>

![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/replication-refresh-only.png)

使用olcSyncrepl（olc）或syncrepl（slapd.conf）配置要将DIT复制到从属副本的slapd服务器。 olcSyncrepl / syncrepl定义包含DIT主副本的slapd服务器的位置。 使用syncprov overlay配置提供程序。



#### refreshAndPersist类型(provider Push)

***



![image](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/replication-refresh-persist.png)



slapd服务器 (1) 使用olcSyncrepl(olc)或syncrepl (slave.conf (7)) 配置，将 DIT (4) 复制到从属副本 (5)。olcSyncrepl/syncrepl定义slapd服务器 (3) 的LDAP URI的位置，该服务器包含DIT (4)  的主副本。 使用syncprov overlay配置提供程序。

在refreshAndPersist类型的复制中，使用者(1)启动与提供者(3)的连接(2) ，DIT立即发生同步，并且在此过程结束时，连接保持不变。对provider (3)的后续更改(E)立即传播到consumer(1)。

**同步细节，图解**：



提供者（provider）不需要维护每个消费者的状态信息。相反，提供者定期发送同步cookie（SyncCookie），该cookie包含变更序列号（contextCSN），其实质上是一个时间戳，指示发送给该消费者的最后一个变化，可以将其视为变更或同步检查点。当一个refreshAndPersist消费者
 打开与提供者的会话，他们必须首先同步他们的DIT（或DIT片段）的状态。根据消费者的初始化方式，当它最初打开与提供者的会话时，它可能没有SyncCookie，因此更改的范围是整个DIT（或DIT片段）。此过程的快乐副产品允许消费者以空复制DIT开始。当消费者随后连接到提供者时，它将具有SyncCookie（D）。在refreshAndPersist类型的复制的情况下，在提供者，消费者或网络发生故障之后将发生重新连接，其中每个连接将终止连接，否则该连接将永久保持。

##### Master (Provider) 配置

配置refreshOnly和refreshAndPersist之间的区别是如此微不足道，它们与明确识别的单个差异相结合。

**master配置**：需要将syncprov覆盖添加为我们复制所需的DIT 的olcDatabase定义的子条目。如果我们假设为了简单起见，我们希望复制的DIT由olcDatabase={1}bdb,cn=config定义，则以下LDIF将syncprov覆盖定义添加为子条目：

```
dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpCheckpoint: 100 10
olcSpSessionLog: 1024

dn: olcDatabase={2}hdb,cn=config
olcDbIndex: entryCSN,entryUUID eq
```

- 必须只创建olcOverlay属性才能创建此条目。
- 当创建子条目时，它将具有（假设这是第一个叠加）一个olcOverlay = {0} syncprov，olcDatabase = {1} bdb，cn = config的dn 。添加条目时分配了{0}。如果你真的，真的想知道那些{}（大括号）是什么意思。
- 使用LDIF是添加到OLC DIT的众多方法之一。
- 如果构建syncprov overlay以动态加载（Linux发行版打包策略在此问题上存在很大差异），则必须在添加syncprov overlay子条目之前在`cn=module{0},cn=config`中创建相应的条目。使用OLC添加/删除模块。

```ldif
add module
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
config syncprov

dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpCheckpoint: 100 10
olcSpSessionLog: 1024

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcDbIndex
olcDbIndex: entryCSN,entryUUID eq
```

#####  Slave (Consumer)配置

Slave (Consumer)配置：需要将olcSyncrepl属性添加到包含副本的DIT的olcDatabase定义中。显然，我们将要复制的<font color="#f8070d" size=2>**olcDatabase条目必须已经存在**</font>。如果不存在，则第一个LDIF创建olcDatabase条目。第二个LDIF只是将olcSynrepl属性添加到现有条目中。


```ldif
dn: olcDatabase=hdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: hdb
olcDbDirectory: /var/lib/ldap
olcSuffix: dc=cylon,dc=org
```

***

> **说明**

- 该olcSuffix属性不是强制性的。
- <font color="#f8070d" size=2>`olcDatabase`</font>和<font color="#f8070d" size=2>`olcDbDirectory`</font>都是必须的的属性，在运行此LDIF之前，olcDbDirectory的目录必须存在且具有适当的权限。
- 创建条目时可以添加其他属性，或者随后可以在任何时候添加其他属性，包括`olcRootDN`和`olcRootPW`等明显的属性。

***

我们假设复制DIT由条目olcDatabase={1} bdb定义。 添加olcSyncrepl属性显示在以下LDIF中：

```ldif
dn: olcDatabase={2}hdb,cn=config
olcSyncRepl:
  rid=001
  provider=ldap://10.0.0.20:389/
  bindmethod=simple
  binddn="cn=admin,dc=cylon,dc=org"
  credentials=123
  searchbase="dc=cylon,dc=org"
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry="5 5 300 +"
  attrs="*,+"
  interval=00:00:00:10
```



#### 6.2.2 OpenLDAP syncrepl（N-Way）多主

OpenLDAP2.4引入了 <font color="#f8070d" size=3>`N-way Multi-Master`</font> 支持。在N路多主机配置中，任何数量的主机可以彼此同步。之前已经为[refreshOnly](http://www.zytrax.com/books/ldap/ch7/#ol-syncrepl-ro)和[refreshAndPersist](http://www.zytrax.com/books/ldap/ch7/#ol-syncrepl-rap)描述了复制功能，此处不再重复。以下注释和配置示例是N-Way Multi-Mastering独有的。

**注意：**运行N-Way Multi-Mastering时，所有master(providers)上的时钟同步到同一时间源至关重要，例如，它们都应运行NTP（网络时间协议）。

在N-Way Multi-Mastering中，每个同步服务提供者也是同步服务的消费者，如图所示：

![image-20200730181558472](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200730181558472.png)



如图所示了3路多主（1,2,3）配置。每个Master都配置为（5,6,7）作为提供者（provider）（使用syncprov overlay）和所有其他master（使用[olcSyncrepl / syncrepl](http://www.zytrax.com/books/ldap/ch6/#syncrepl)）的[使用者](http://www.zytrax.com/books/ldap/ch6/#syncrepl)。必须使用[olcServerID / ServerID](http://www.zytrax.com/books/ldap/ch6/#serverid)唯一标识每个提供程序。如上所述，每个提供商应该与公共时钟源同步。因此，DIT（4）的提供者（1）包含配置的syncprov覆盖（提供者覆盖）和两个[refreshAndPersist](http://www.zytrax.com/books/ldap/ch7/#ol-syncrepl-rap)输入olcSyncrepl / syncrepl定义，其中一个用于其他每个提供者（2,3），如蓝色通信链接所示。类似地，每个其他提供程序都具有等效配置 - 单个提供程序功能和其他两个主服务器（提供程序）的refreshAndPersist olcSyncrepl / syncrepl定义。

N-Way Multi-Master复制版本2.4目前不支持增量同步。

http://www.zytrax.com/books/ldap/ch7/#ol-syncrepl-rap



http://www.zytrax.com/books/ldap/ch3/#attribute-x-order


```
ldapsearch -x -D 'cn=admin,dc=cylon,dc=org' -W -b 'dc=cylon,dc=org' -LLL -H ldap://10.0.0.20
```


```
cat << EOF | ldapadd -x -W -H ldap://10.0.0.21 -D cn=admin,dc=cylon,dc=org
dn: dc=cylon,dc=org
objectClass: organization
objectClass: dcObject
dc: cylon
o: cylon

dn: ou=People,dc=cylon,dc=org
objectClass: organizationalUnit
ou: People

dn: ou=group,dc=cylon,dc=org
objectClass: organizationalUnit
ou: group

dn: cn=tech,ou=group,dc=cylon,dc=org
objectClass: posixGroup
description: 5oqA5pyv6YOo
gidNumber: 10001
cn: tech

dn: uid=user01,ou=People,dc=cylon,dc=org
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
sn: user01
givenName: user01
userPassword:: e1NTSEF9aEpwSUlWeGoxcVM5ZzA1cVVsZ0crbzdNTzE0RVhiRlE=

dn: uid=oldgirl,ou=People,dc=cylon,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/oldgirl
loginShell: /bin/bash
uid: oldgirl
cn: oldgirl
uidNumber: 10005
gidNumber: 10001
sn: oldgirl
userPassword:: e1NTSEF9MnB2RTRDNnh5OGRrbVcyYUQvZUVvY1Zhamc4QnVqV1c=

dn: cn=t2,ou=People,dc=cylon,dc=org
givenName: t
sn: 2
cn: t2
uid: t2
userPassword:: MTIz
uidNumber: 1000
gidNumber: 10001
homeDirectory: /home/users/t2
loginShell: /bin/sh
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top

dn: cn=jack,ou=group,dc=cylon,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 10011
cn: mygroup
cn: jack

dn: ou=admin,dc=cylon,dc=org
objectClass: top
objectClass: organizationalUnit
ou: admin

dn: cn=ldapadmin,ou=admin,dc=cylon,dc=org
cn: ldapadmin
objectClass: groupOfNames
member: uid=mike,ou=People,dc=cylon,dc=org
member: cn=t2,ou=people,dc=cylon,dc=org

dn: ou=admin1,dc=cylon,dc=org
objectClass: top
objectClass: organizationalUnit
ou: admin1

dn: cn=ldap,ou=admin1,dc=cylon,dc=org
member: cn=jack,ou=group,dc=cylon,dc=org
cn: ldap
objectClass: groupOfNames

dn: cn=configadmin,ou=admin,dc=cylon,dc=org
member: cn=mike,ou=people,dc=cylon,dc=org
cn: configadmin
objectClass: groupOfNames

dn: cn=mygroup,ou=group,dc=cylon,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 10010
cn: mygroup

dn: uid=syncuser,ou=admin,dc=cylon,dc=org
objectClass: posixAccount
objectClass: top
objectClass: inetOrgPerson
gidNumber: 0
givenName: sync
sn: user
displayName: syncuser
uid: syncuser
homeDirectory: /home/user/syncuser
loginShell: /bin/bash
cn: syncuser
uidNumber: 18809

dn: uid=rpuser,dc=cylon,dc=org
objectClass: simpleSecurityObject
objectClass: account
uid: rpuser
description: Replication User
userPassword:: cm9vdDEyMzQ=
EOF
```

```bash
cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=admin,dc=cylon,dc=org
dn: cn=123123,ou=group,dc=cylon,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 10011
cn: mygroup
EOF

cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=admin,dc=cylon,dc=org
dn: uid=rpuser,dc=cylon,dc=org
objectClass: simpleSecurityObject
objectclass: account
uid: rpuser
description: Replication User
userPassword: root1234
EOF
```

```bash
cat << EOF | ldapadd -x -W -H ldap://10.0.0.20 -D cn=admin,dc=cylon,dc=org
dn: uid=ldaprptest,ou=People,dc=cylon,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: ldaprptest
uid: ldaprptest
uidNumber: 9988
gidNumber: 100
homeDirectory: /home/ldaprptest
loginShell: /bin/bash
gecos: LDAP Replication Test User
userPassword: {crypt}x
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
EOF
```




```
ldapdelete -x -D "cn=admin,dc=cylon,dc=org" -W -r "dc=cylon,dc=org" -H ldap://10.0.0.20
```

ls |grep -v DB_CONFIG|xargs rm -f



## 7 Linux账号集中管理

### 7.1 通过本地Sudo规则实现OpenLDAP用户提权匹配

要想通过本地 Sudo 实现 OpenLDAP 用户提权，可按以下步骤操作。

```sh
authconfig \
--enableldap \
--enableldapauth \
--ldapserver=ldap://10.0.0.20:389 \
--ldapbasedn="dc=cylon,dc=org" \
--enablemkhomedir \
--ldapbasedn="ou=sudoers,dc=cylon,dc=org" \
--enableshadow \
--update \

yum install pam_ldap nss-pam-ldapd -y
```

authconfig命令详解 https://blog.51cto.com/lajifeiwomoshu/1982378

默认sudo提权使用本地sudoers文件进行查找匹配，如果指定其他获取方式，则需要添加sudoers属性并指定验证类型。当OpenLDAP用户没有权限创建用户及切换到权限，因为普通用户不具备套在管理员指令的权限。

### 7.2 在OpenLDAP服务端实现用户权限控制

部署此功能的原因是能够在所有基础结构服务器上仅使用一个用户，并且无需每次在每台服务器上手动更新 <font color="#f8070d" size=3>`/etc/sudoers`</font> 文件，即可为此用户提供sudo权限。现在，这些天您可以使用像ansible这样的工具来执行此操作，但是并不是说OpenLDAP用法必须仅用于posixGroup用户访问，因此OpenLDAP只擅长它。OpenLDAP集成应扩展到您部署的每个集中式系统，并且您唯一的“管理员”用户可以访问基础架构范围内的所有系统。

#### 7.3 为sudoers集成OpenLDAP服务器准备

要在 `OpenLDAP` 服务端实现用户权限控制，实施步骤如下。

- 导入sudo schema
- 定义sudo规则条目及sudo组
- 用户加入sudo组，集成sudo权限
- 命令添加及修改
- 图形化管理界面配置
- 客户端配置加入OpenLDAP服务端
- 客户端识别sudo策略及验证用户权限

> **OpenLDAP服务端导入sudo schema**

```sh
rpm -ql sudo | grep schema  
cp /usr/share/doc/sudo-1.8.23/schema.OpenLDAP /etc/openldap/schema/sudo.schema  
echo "include   /etc/openldap/schema/sudo.schema" >  /tmp/sudo.conf
slapcat -f /tmp/sudo.conf -F /tmp/ -n 0 -s "cn={0}sudo,cn=schema,cn=config" > /tmp/sudo.ldif

sed -i "s@{0}sudo@sudo@g" /tmp/sudo.ldif
head -n -8 /tmp/sudo.ldif > /etc/openldap/schema/sudo.ldif
```

再次检查 OpenLDAP 数据库目录中 schema 产生的文件，命令如下：<font color="#f8070d" size=3>`/etc/openldap/slapd.d/cn=config/cn=schema/`</font> 不难发现此指令成功添加了一个新条目 <font color="#f8070d" size=3>`cn={13}sudo.ldif`</font>

> **在 OpenLDAP 目录树中创建 suers 条目**

sudoers 的配置信息存放在 ou=suders 的子树中，默认 OpenLDAP 用户没有指定 sudo 规则，OpenLDAP 首先在目录树中寻找条目 cn=default，如果找到，那么所有 sudoOption 属性都会被解析为全局默认值，这类似于服务端中查询一个 sudo 用户权限时一般有两到三次查询。第一次查询解析全局配置，第二次查询匹配用户名或者用户所在的组（特殊标签 ALL 也在此次查询中匹配），如果没有找到相关匹配项，则发出第三次查询，此次查询返回所有包含用户组的条目并检查该用户是否存在于这些组中。接下来创建 OpenLDAP 的 suders 子树。具体命令如下：

```ldif
$ cat << EOF | ldapadd -D "cn=admin,dc=shileizcc,dc=com" -w 123456 -H ldap://shileizcc.com

dn: ou=sudoers,dc=shileizcc,dc=com
objectCLass: top
objectClass: organizationalUnit
ou: sudoers

dn: cn=defaults,ou=sudoers,dc=shileizcc,dc=com
objectClass: sudoRole
objectCLass: top
cn: defaults
description: Default sudoOption's go here
sudoOption: requiretty
sudoOption: !visiblepw
sudoOption: always_set_home
sudoOption: env_reset
sudoOption: env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
sudoOption: env_keep="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
sudoOption: env_keep="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
sudoOption: env_keep="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
sudoOption: env_keep="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
sudoOption: secure_path=/sbin:/bin:/usr/sbin:/usr/bin

dn: cn=%dba,ou=sudoers,dc=shileizcc,dc=com
objectClass: sudoRole
objectCLass: top
cn: %dba
sudoUser: %dba
sudoHost: ALL
sudoRunAsUser: oracle
sudoRunAsUser: grid
sudoOption: !authenticate
sudoCommand: /bin/bash

dn: cn=%app,ou=sudoers,dc=shileizcc,dc=com
objectClass: sudoRole
objectCLass: top
cn: %app
sudoUser: %app
sudoHost: ALL
sudoRunAsUser: appman
sudoOption: !authenticate
sudoCommand: /bin/bash

dn: cn=%admin,ou=sudoers,dc=shileizcc,dc=com
objectClass: sudoRole
objectCLass: top
cn: %admin
sudoUser: %admin
sudoHost: ALL
sudoRunAsUser: oracle
sudoRunAsUser: grid
sudoOption: authenticate
sudoCommand: /bin/rm
sudoCommand: /bin/rmdir
sudoCommand: /bin/chmod
sudoCommand: /bin/chown
sudoCommand: /bin/dd
sudoCommand: /bin/mv
sudoCommand: /bin/cp
sudoCommand: /sbin/fsck*
sudoCommand: /sbin/*remove
sudoCommand: /usr/bin/chattr
sudoCommand: /sbin/mkfs*
sudoCommand: !/usr/bin/passwd
sudoOrder: 0

dn: cn=%limit,ou=sudoers,dc=shileizcc,dc=com
objectClass: sudoRole
objectClass: top
cn: %limit
sudoUser: %limit
sudoHost: limit.shileizcc.com
sudoCommand: /usr/bin/chattr
sudoRunAsUser: ALL
sudoOption: !authenticate

dn: cn=%manager,ou=sudoers,dc=shileizcc,dc=com
objectClass: sudoRole
objectClass: top
cn: %manager
sudoUser: ALL
sudoHost: ALL
sudoCommand: /bin/bash
sudoRunAsUser: ALL
sudoOption: !authenticate
EOF
```

![1567835839019](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1567835839019.png)



![1567836129731](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1567836129731.png)



### 集成具有Sudoers和PAM的OpenLDAP客户端。

在客户端，您需要执行两个额外步骤。

**需要添加**：<font color="#f8070d" size=3>` /etc/nsswitch.conf`</font>

```sh
 ## 增加
 sudoers:    ldap files
 
 echo 'sudoers:    ldap files' >> /etc/nsswitch.conf
```

**需要提供**：<font color="#f8070d" size=3>`/etc/sudo-ldap.conf`</font>

```sh
binddn cn=clientsearch,ou=admin,dc=cylon,dc=org
bindpw 111
uri ldap://10.0.0.20
sudoers_base ou=sudoers,dc=cylon,dc=org
```

**参考文档**: 

- [OpenLDAPSudo权限讲解-OpenLDAPSudo权限讲解](https://wiki.shileizcc.com/confluence/pages/viewpage.action?pageId=40566794#OpenLDAPSudo权限讲解-OpenLDAPSudo权限讲解)
- https://linux.die.net/man/8/authconfig
