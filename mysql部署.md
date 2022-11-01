# 

[toc]

## 1 MySQL数据库简介

编程语言排名：http://www.tiobe.com/tiobe-index

数据库排名：http://db-engines.com/en/ranking

### 1.1 MySQL数据库分类与版本升级
MySQL数据库官网为http://www.mysql.com，其发布的MySQL版本采用双授权政策，和大多数开源产品的路线一样，分别为社区版和商业版，而这两个版本又各自分四个版本依次发布，这四个版本为Alpha版、Beta版、RC版和GA版（GA正式发布版）

### 1.2 MySQL数据库商业版和社区版的区别

在前面的内容已经阐述过了，MySQL的版本发布采用双授权政策，即分为社区版和商业版，而这两个版本又各自分四个版本依次发布：Alpha版、Beta版、RC版和GA版（GA正式发布版）
### 1.3 Alpha版

Alpha版一般只在开发的公司内部运行，不对外公开。主要死开发者自己对产品进行测试，检查产品是否存在缺陷、错误，验证产品功能与说明书、用户手册是否一致。MySQL是属于开放源代码的开源产品，因此需要世界各地开发者、爱好者和用户参与软件的开发测试和手册编写等工作。所以会对外公布此版本的源码和产品，方便任何人可以参与开发测试工作，甚至编写与修改用户手册。

### 1.4 Beta版

Beta版一般是完成功能的开发和所有的测试工作时候的产品，不会存在较大的功能或性能BUG，并且邀请或提供给公户体验与测试，以便更全面地测试软件的不足之处或存在的问题。
### 1.5 RC版

RC版属于生产环境发布之前的一个小版本或称候选版，是根据Beta测试结果，收集到的BUG或缺陷之处等收集到信息，进行修复和完善之后的新一版本

### 1.6 GA版

GA版是软件产品正式发布的版本，也称生产版本的产品。一般情况下，企业生产环境都会选择GA版本的MySQL软件，用于真实的生产环境中。偶尔有个别的大型企业会追求新功能驱动而牺牲稳定性使用其他版本，但这个是个例。

### 1.7 MySQL四中发布版本选择说明

MySQL AB官方网站会把五种数据库版本都提供下载，主要是MySQL数据库属于开发源代码的数据库产品，鼓励全球的技术爱好者参与研发、测试、文档编写和经验分享，甚至包过产品发展规划，对于Development版本、Alpha版本和Beta版本是绝对不允许使用在任何生产环境，毕竟这是一个GA版本之前，也即生产版本发布之前的一个小版本。另外，对MySQL数据库GA版本，也是需要慎重选择，开源社区产品毕竟不是经过严格的测试工序完成的产品，是全球开源技术人员的资源完成的，会存在比商业产品稳定性弱的缺陷。更严格的选择见后文。


### 1.8 MySQL产品路线


#### 1.8.1 MySQL产品路线变更历史背景

早起MySQL也是遵循版本号逐渐增加的方式发展的，格式例如：mysql-x.xx.xx.tar.gz，例如DBA都非常熟悉的生产场景版本：4.1.7、5.0.56等。

近几年，为了提高MySQL产品的竞争优势、以及提高性能、降低开发维护成本等原因，同时，更方便企业用户更精准的选择适合的版本产品用于自己的企业生产环境中。
MySQL在发展到5.1系列版本之后，重新规划为3条产品线

#### 1.8.2 5.0.xx到5.1.xx产品线介绍

第一条产品线：5.0.xx及升级到5.1.xx的产品系列，这条产品线继续完善与改进其用户体验和性能，同时增加新功能，这条路线可以说是MySQL早起产品的延续系列，这一系列的产品发布情况及历史版本如下：
MySQL 5.1是当前稳定（产品质量）发布系列。只针对漏洞修复重新发布；没有增加会影响稳定性的新功能。
  * MySQL 5.1:Previous stable(production-quality) release
MySQL 5.0是前一稳定（产品质量）发布系列。只针对严重漏洞修复和安全修复重新发布；没有增加会影响该系列的重要功能。
  * MySQL 5.0:Old stable release nearing the end of the product lifecycle
MySQL 4.0和3.23是旧的稳定(产品质量)发布系列。该版本不再使用，新的发布只用来修复特别严重的漏洞(以前的安全问题)。

#### 1.8.3 5.4.xx开始到了5.7.xx产品线系列介绍

为了更好的整合MySQL AB公司社区和第三方公司开发的新存储引擎，以及吸收新的实现算法等，从而更好的支持SMP架构，提高性能而做了大量的代码重构。版本号为从5.4.xx开始，目前发展到了5.6.x
主流：互联网公司用mysql5.5，逐步过渡到5.6。

#### 1.8.4 6.0.xx-7.1.xx产品线系列介绍

第三条产品线：为了更好的推广MySQL Cluster版本，以及提高MySQL Cluster的性能和稳定性，以及功能改进和增加，以及改动mysql基础功能，使其对Cluster存储引擎提供更有效地支持与优化。版本号为6.0.xx开发，目前发展到7.1.xx

### 1.9 MySQL数据库软件命名介绍

MySQL数据库软件的名字是由3个数字和一个后缀组成的版本号。例如，像mysql-5.0.56.tar.gz的版本号这样解释：
第一个数字（5）为主版本号，描述了文件格式。所有版本5发行都有相同文件格式。
第二个数字（0）为发行级别，主版本号和发行级别组合到一起便构成了发行序列号。
第三个数字（56 为在此发行系列的版本号，随每个新分发版本递增。通常你需要已经选择的发行(release)的最新版本。

每次更新后，版本字符串的最后一个数字递增。如果相对于前一个版本增加了新功能或有微小的不兼容性，字符串的第二个数字递增。如果文件格式改变，第一个数字递增。
后缀显示发现的稳定性级别。通过一系列后缀显示如何改进稳定性。可能的后缀有：

alpha表明发行包含大量未被彻底测试的新代码。已知的缺陷应该在新闻小节被记录。请参见附录D：MySQL变更史。在大多数alpha版本中也有新的命令和扩展。alpha版本也可能有主要代码更改等开发。但我们在发布前一定对其进行测试。

beta意味着该版本功能是完整的，并且所有的新代码被测试了，没有增加重要的新特征，应该没有已知的缺陷。当alpha版本至少一个月没有出现报导的致命漏洞，并且没有计划增加导致已经实施的功能不稳定的新功能时，版本则从alpha版变为beta版。在以后的beta版、发布版或产品发布中，所有API、外部可视结构和SQL命令列均不再更改。

rc是发布代表；是一个发行了一段时间的beta版本，看起来应该运行正常。只增加了很小的修复。(发布代表即以前所称的gamma 版)
如果没有后缀，这意味着该版本已经在很多地方运行一段时间了，而且没有非平台特定的缺陷报告。只增加了关键漏洞修复修复。这就是我们称为一个产品（稳定）或“通用”版本的东西。

MySQL的命名机制于其它产品稍有不同。一般情况，我们可以很放心地使用已经投放市场两周而没有被相同发布系列的新版本所代替的版本。

#### 1.9 MySQL产品版本最终选择建议


*  稳定版：选择开源的社区版的稳定GA版本。
*  产品线：可以选择5.1或5.5，互联网公司主流5.5，其次是5.1和5.6
*  选择MySQL数据库GA版本发布后6个月以上的GA版本
*  选择前后几个月没有打的BUG修复的版本，二不是大量修复BUG的集中版本
*  最好向后较长时间没有更新发布的版本
*  考虑开发人员开发程序使用的版本是否兼容你选择的版本
*  作为内部开发测试数据库环境，跑大概3-6个月的时间
*  优先企业非核心业务采用新版本的数据库GA版本软件
*  向DBA高手请教，使用真正的高手们使用过得好用的GA版本产品
*  经过上述工序后，若是没有重要功能BUG或性能瓶颈，则可以开始开率作为任何业务数据服务的后端数据库软件。





## 2 安装MySQL




最正宗的产品线5.1及以前：常规的编译方式安装MySQL

所谓常规方式编译安装MySQL就是延续早起MySQL的3部曲安装方式，即 ./configure;make;make install。

此种方式适合所有MySQL5.0.xx-5.1.xx产品系列，是最常规的编译方式。



### 2.1 采用cmake方式编译安装MySQL

由于MySQL5.5.xx-5.6.xx产品系列特殊性，所以编译方式也和早期的产品安装方式不同，采用cmake或gmake方式编译安装。即./cmake;make;make install，生产场景具体命令及参数为

#### 2.1.2 采用二进制方式免编译安装MySQL

采用二进制方式免编译安装MySQL，这种方式和yum/rpm包安装方式类似，适合类MySQL产品系列，不需要负载的编译设置及编译时间等待，直接解压下载的软件包，初始化可完成mysql的安装启动。

#### 2.1.3 如何正确选择MySQL的安装方式

yum/rpm安装适合对数据库要求不太高的场合，例如并发不大，公司内部，企业内部的一些应用场景。二进制免安装比较简单方便，适合5.0-5.1和5.5-5.6系列，是很多专业DBA的选择，普通linux运维人员多采用编译的方式，5.0-5.1采用的常规方式，5.5-5.6采用cmake方式。

建议，安装机器较少，推荐cmake方式，数量多了就用二进制免安装。

### 2.2 MySQL下载

http://www.mysql.com

* enterprise：企业版 商业版
* community：社区版
* yum repository：yum仓库（centos redhat fedora）
* apt repository：apt-get仓库（debian Ubuntu）
* SUSE repository：suse仓库
* window：windows版

#### 2.2.1 企业场景MySQL安装方式

|序号|安装方式|特点说明|
|:-:|-|-|
|1&nbsp;&nbsp;&nbsp;|yum/rpm包安装|简单、速度快，但是不能定制安装，入门新手常用这种方式|
|2 |二进制安装| <font style="background:#fee904;" size=2> 解压软件</font>，简单配置后就能使用，<font style="background:#fee904;" size=2>不用安装</font>，速度较快，专业DBA喜欢这种方式。软件名如：<br><font style="background:#fee904;" size=2>linux-5.5.32-linux2.6-x86_64.tar.gz</font><br><font style="background:#fee904;" size=2>mysql-5.6.33-linux-glibc2.5-x86_64.tar</font>|
|3 |源码编译安装| 可以定制安装，但是安装时间长，例如：字符集安装路径等，软件名：<font style="background:#fee904;" size=2>linux-5.5.32.tar.gz</font>。<br>针对mysql5.0 5.1；<br>mysql5.5以上 `./cmake` ; `gmake;gmake instal` ; |
|4 |源码软件结合yum/rpm安装  |把源码软件制作成符合要求的rpm，放到yum仓库里，然后通过yum来安装，结合上面1和3的优点，即安装快速，可任意定制参数，但是安装者需要具备更深能力。|

默认路径 usr/local/mysql

大型门户把源码根据企业的需求制作成rpm，搭建yum仓库，yum install xxx -y

http://oldboy.blog.51cto.com/2561410/1722136

http://oldboy.blog.51cto.com/2561410/1121725

http://oldboy.blog.51cto.com/2561410/1121745

http://dreamway.blog.51cto.com/1281816/1110874

http://dreamway.blog.51cto.com/1281816/1110874

### 2.3 二进制包安装


#### 2.3.1 创建MySQL用的账号

```bash
useradd -s /sbin/nologin -M mysql
```
#### 2.3.2 二进制方式安装mysql

解压软件包

```bash
tar -zxf mysql-5.5.32-linux2.6-x86_64.tar.gz
```

移动目录

```bash
mv mysql-5.5.32-linux2.6-x86_64 /app/mysql-5.5.32
```

创建软连接，生成去掉版本号的路径方便访问

```bash
ln -s /app/mysql-5.5.32/ /app/mysql
```

***
<font color="#0215cd" size=2>提示：操作到此部，相当于编译安装make install之后</font>
***

#### 2.3.3 初始化MySQL

```bash
[root@centos tools]# /app/mysql/scripts/mysql_install_db \
--basedir=/app/mysql/ \
--datadir=/app/mysql/data \
--user=mysql
WARNING: The host 'centos' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !
Installing MySQL system tables...
OK
Filling help tables...
OK
 
To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system
.....
.....
You can start the MySQL daemon with:
cd /app/mysql/ ; /app/mysql//bin/mysqld_safe &
 
You can test the MySQL daemon with mysql-test-run.pl
cd /app/mysql//mysql-test ; perl mysql-test-run.pl
 
Please report any problems with the /app/mysql//scripts/mysqlbug script!
```

#### 2.3.4 初始化MySQL配置文件

```bash
[root@centos mysql]# ls /app/mysql/support-files/*.cnf
my-huge.cnf
my-innodb-heavy-4G.cnf
my-large.cnf
my-medium.cnf
my-small.cnf
 # my-medium.cnf < my-small.cnf < my-small.cnf < my-huge.cnf < my-innodb-heavy-4G.cnf
```

虚拟机测试环境下选择my-small.cnf配置模板。如果是生产环境可以根据硬件配置选择高级的配置文件

```bash
cp /app/mysql/support-files/my-small.cnf /etc/my.cnf
```

#### 2.3.5 配置启动MySql数据库

设置启动脚本
二进制默认安装路径是/usr/local/mysql，启动脚本里是/usr/local/mysql都需要替换

```bash
# 启动脚本
/app/mysql/bin/mysqld_safe

# 替换命令
sed -i 's#/usr/local/mysql#/app/mysql#g' /app/mysql/bin/mysqld_safe
```

> **启动数据库**

1. 传统方式

```bash
cp mysql.server /etc/init.d/mysqld
sed -i 's#/usr/local/mysql#/app/mysql#g' /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
killall mysqld
[root@centos support-files]# /etc/init.d/mysqld start
Starting MySQL.. SUCCESS!
[root@centos support-files]# /etc/init.d/mysqld stop
Shutting down MySQL. SUCCESS!
 
[root@centos bin]# /app/mysql/bin/mysqld_safe & #<==“&”作用是在后台执行MySQL服务
```

> **mysql启动错误**

```bash
[root@multi 3306]# ./mysql start
Starting MySQL...
[root@multi 3306]# 170521 22:59:38 mysqld_safe error: log-error set to '/data/3306/logs/mysql_3306.err', however file don't exists. Create writable for user 'mysql'.
```

mysqlbug https://bugs.mysql.com/bug.php?id=84427

> **检查MySQL数据库是否启动**
```bash
[root@centos bin]# lsof -i :3306
COMMAND   PID    USER    FD   TYPE  DEVICE  SIZE/OFF  NODE  NAME
mysqld    38313  mysql   10u  IPv4  73384   0t0       TCP   *:mysql (LISTEN)
```

#### 2.3.6 配置mysql命令全局使用

> **配置全局路径的意义**

如果不为MySQL的命令配置全局路径，就无法直接在命令行输入mysql这样的命令，只能用全局命令  
方法1：

```bash
vi /etc/profile
PATH="/app/mysql/bin:$PATH"
source /etc/profile
echo 'PATH="/app/mysql/bin:$PATH"' >>/etc/profile && . /etc/profile
```

方法2：

```bash
# 尽量往前面拷，要不会和系统yum安装的mysql冲突
cp /app/mysql/bin/* /usr/local/sbin 
[root@nfs tools]# which mysql

# 如果找到的是 /usr/bin是系统yum安装的，不要让我们安装的和yum安装的冲突
/app/mysql/bin/mysql  

```

方法3：

```bash
ln -s /app/mysql/bin/* /usr/local/sbin/
```

***
**<font color="#0215cd" size=2>特别强调：必须把MySQL命令路径放在PATH路径中的前面，否则可能会导使用了mysql等命令和编译安装的不是一个，进而产生错误。</font>**
***

yum安装MySQL命令访问编译安装的服务器而出来问题：http://oldboy.blog.51cto/25614110/011

#### 2.3.7 MySQL-5.6二进制包安装
安装成功提示：与MySQL 5.5.x一样看到两个OK即安装完毕

```bash
[root@lamp_server app]#/app/mysql/scripts/mysql_install_db \
--basedir=/app/mysql/ \
--datadir=/app/mysql/data \
--user=mysql

WARNING: The host 'lamp_server' could not be looked up with /app/mysql//bin/resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !
 
Installing MySQL system tables...2016-09-28 17:45:23 0 [Warning] TIMESTAMP with implicit DEFAULT
....
....
OK
 
Filling help tables...2016-09-28 17:45:28 0 
[Warning] TIMESTAMP with implicit DEFAULT value is deprecated. 
Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
....

```
将启动脚本复制到/etc/ini.d下
```bash
cp support-files/mysql.server /etc/init.d/mysqld
cp my.cnf /etc/my.cnf #←修改配置文件
vi /etc/my.cnf
```

[mysqld]中添加：

```bash
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
server_id = 1
```

将全局启动命令做个链接

```bash
ln -s /app/mysql/bin/mysql /usr/bin
echo "PATH=/app/mysql/bin:$PATH" >>/etc/profile
```

mysql5.6二进制后登陆

```bash
[root@lamp_server mysql]# mysql -uroot -p
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.33 MySQL Community Server (GPL)
 
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
mysql>
```


## 3 MySQL多实例


### 3.1 什么是MySQL多实例？

简单的说，MySQL多实例就是一台服务器上同时开启多个不同的服务器端口（如3306、3307），同时运行多个MySQL服务器进程，这些服务进程通过不同socket监听不同的服务端口来提供服务。

这些MySQL多实例共用一套MySQL安装程序，使用不同的my.cnf（也可相同）配置文件、启动程序（也可以相同）的数据文件。在提供服务时，多实例MySQL在逻辑上看来是各自独立的，他们根据配置文件的对应设定值，获得服务器相应数量的硬件资源。

MySQL多实例就相当于房子的多个我是，每个实例可以看做一件我是，整个服务器就是一套房子，服务器的硬件资源（cpu,mem,disk）、软件资源（centos）可以看做房子的卫生间、厨房、客厅，是房子的公用资源。


nginx apache Haproxy memcached redis等都可配置多实例。

### 3.2 MySQL多实例的作用与问题

#### 3.2.1 有效利用服务器资源

当单个服务器资源有剩余时，可以充分利用剩余的资源提供更多的服务，实现资源的逻辑隔离。

#### 3.2.2 节约服务器资源

当公司自己紧张，但是数据库又需要各自尽量独立地提供服务，而且需要主从复制等技术，多实例就再好不过了。

当某个数据库实例并发很高或者有SQL慢查询时，整个实例会消耗大量的系统CPU、磁盘IO等资源，导致服务器上的其他数据库实例提供服务的质量一起下降。会存在资源互抢占问题。不同实例获取的资源是相对立的，无法像虚拟化一样完全隔离。

### 3.3 MySQL多实例的生产应用场景

#### 3.3.1 资金紧张型公司的选择

若公司资金紧张，公司业务访问量又不太大，但又希望不同业务的数据库服务各自尽量独立的提供服务而互相不收影响，同时，还需要主从复制等技术提供备份或读写分离服务，那么，多实例就再好不过了。比如：可以通过3台服务器部署9-15个实例，交叉做主从复制、数据备份及读写分离，这样就可达到9-15太服务器每个只装一个数据库才有的效果。这里要强调的是，所谓的尽量独立是相对的。

#### 3.3.2 并发不是特别大的业务

当公司业务访问量不太大的时候，服务器的资源基本都是浪费的，这时就很适合多实例的应用，如果对SQL语句的优化做的比较好，MySQL多实例会是一个很值得使用的技术，及时并发很大，合理分配好系统资源以及搭配好系统服务，也不会有太大问题

#### 3.3.3 门户网站应用MySQL多实例场景

门户网站通常都会使用多实例，因为配置硬件好的服务器，可节省IDC机柜空间，同时，跑多实例也会减少硬件资源跑不满的浪费。一般是丛库多实例，例如某部门中使用的IBM服务器为48核，96GB内存，一台服务器跑3-4个实例。(高配多实例，节省机柜空间，虚拟化也是这样)

### 3.4 MySQL多实例常见的配置方案

#### 3.4.1 单一配置文件、单一启动程序多实例部署方案

下面是MySQL官方文档提到的单一配置文件、单一启动程序多实例部署方案，不推荐此方案，这里仅作为知识点提及，不在涉及此方案说明。

```bash
[mysqld_multi]
mysqld=/usr/bin/mysqld_safe
mysqladmin=/usr/bin/mysqladmin
user=mysql
[mysql1]
socket=/var/lib/mysql/mysql.sock
port=3306
pid-file=/var/lib/mysql/mysql.pid
datadir=/var/lib/mysql
user=mysql
[mysql1]
socket=/var/lib/mysql/mysql.sock
port=3306
pid-file=/var/lib/mysql/mysql.pid
datadir=/var/lib/mysql
user=mysql
skip-name-resolve
server-id=10
```

```bash
mysql_multi --config-file=/data/mysql/my_multi.cnf start 1,2 #<==启动命令
```

对于该方案，缺点是耦合度太高，一个配置文件，不好管理。工作开发和运维的统一。原则：降低耦合度。

#### 3.4.2 多配置文件多启动程序部署方案

以下是已经部署好的MySQL-5.5双实例的目录信息及文件注释说明

```bash
data
├── 3306
│   ├── my.cnf   #<==配置文件
│ ├── data     #<==数据文件
│   └── mysql #<==多实例启动脚本
└── 3307
    ├── my.cnf
└── mysql
```

### 3.5 安装MySQL多实例

#### 3.5.1 安装MySQL需要的依赖包和编译软件

安装MySQL之前需要安装MySQL依赖包

```bash
yum install -y libaio-devel ncurses-devel
```

> **5.1.1 安装编译MySQL需要的软件**

因MySQL5.5系列采用的cmake编译，需要先下载安装cmake

```bash
yum install cmake -y
```

#### 3.5.2 采用编译方式安装MySQL

```bash
tar zxf mysql-5.5.52.tar.gz
cd mysql-5.5.52
```

编译参数

```bash
cmake . -DCMAKE_INSTALL_PREFIX=/app/mysql-5.5.54 \
-DMSQL_DATADIR=/app/mysql-5.5.54/data \
-DMYSQL_UNIX_ADDR=/app/mysql-5.5.54/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DEXTRA_CHARSETS=gbk,gb2312,utf8,ascii \
-DENABLED_LOCAL_INFILE=ON \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_EXAMPLE_STORAGE_ENGINE=1 \
-DWITHOUT_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FAST_MUTEXES=1 \
-DWITH_ZLIB=bundled \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_READLINE=1 \
-DWITH_EMBEDDED_SERVER=1 \
-DWITH_DEBUG=0
```

创建软连接

```bash
ln -s /app/mysql-5.5.54/ /app/mysql
```

#### 3.5.3 创建MySQL多实例的数据文件目录

在企业中，通常以 `/data` 目录作为MySQL多实例总的根目录，然后规划不同的数字（即MySQL实例端口号）作为/data下面的二级目录，不同的二级目录对应的数字就作为MySQL实例的端口号，以区别不同的实例，数字对应的二级目录下包含MySQL的数据文件、配置文件及启动文件等。
创建数据目录

```bash
mkdir -p /data/{3306,3307}/data 
# 多个可以依次累加，生产环境中一般3-4个实例为最佳。
```
#### 3.5.4 创建MySQL多实例的配置文件

MySQL数据库默认为用户提供了多个配置文件模板，用户可以根据服务器硬件配置的大小来选择。

```bash
[root@multi /]# ls /app/mysql/support-files/my*.cnf
my-huge.cnf             
my-medium.cnf
my-innodb-heavy-4G.cnf  
my-small.cnf
my-large.cnf
```

上面是单实例的默认配置文件模板，如果配置多实例，和单实例会有不同。为了让MySQL多实例之间彼此独立，因此要为每一个实例建立一个my.cnf配置文件和一个启动文件mysql，让他们分别对应自己的数据文件目录data
创建配置文件及目录略，一般都是用配置好的配置文件与脚本

#### 3.5.5 配置权限

```bash
[root@multi /]# find /data -type f -name 'mysql' #<==启动脚本中存在mysql密码要注意权限
/data/3306/mysql
/data/3307/mysql
find /data -type f -name 'mysql'|xargs chmod 700
```

#### 3.5.6 MySQL相关命令加入全局路径的配置

```bash
echo "PATH=/app/mysql/bin:$PATH"
ln -s /app/mysql/bin/* /usr/sbin/ 
```

### 3.6 初始化多实例数据库文件

上述步骤全部配置完毕后，就可以初始化数据库文件了，这个步骤其实也可以在编译安装MySQL之后就操作，只不过放到这里更合理，
#### 3.6.1 初始化MySQL数据库

初始化数据库会有很多提示，如果没有error级别的错误，有两个ok字样表示初始化成功，否则就解决初始化问题

```bash
cd /app/mysql/script/
./mysql_install_db \
--basedir=/app/mysql/ \
--datedir=/data/3306/data \
--user=mysql
 
./mysql_install_db \
--basedir=/app/mysql/ \
--datadir=/data/3306/data \
--user=mysql \
--defaults-file=/data/3306/my.cnf
```
#### 3.6.2 初始化数据可的原理及结果说明

初始化数据库的实质就是创建基础的数据库系统的库文件，例如：生成MySQL库表等。
初始化数据可偶查看对应实例的数据目录，可以看到多了一些文件
启动MySQL多实例数据库

```bash
/data/3306/mysql start
/data/3307/mysql start
 
[root@multi ~]# netstat -nltup       
tcp      0      0 0.0.0.0:3306    0.0.0.0:*    LISTEN     38980/mysqld        
tcp      0      0 0.0.0.0:3307    0.0.0.0:*    LISTEN     22942/mysqld        
tcp      0      0 :::52113             :::*    LISTEN     1123/sshd  
```

### 3.7 MySQL多实例启动故障排错说明

如果MySQL多实例有服务没有被启动，排查办法如下：

如果发现没有显示MySQL对应实例的端口，请稍微等待几秒再检查，MySQL服务的启动比web服务慢一些

如果还是不行，请查看MySQL服务对应实例的错误日志，错误日志路径在my.cnf配置的最下面定义。

例如3306的错误日志为：

```bash
[mysqld_safe]
log-error=/data/3306/mysql_3306.err
pid-file=/data/3306/mysqld.pid
```

1. 细看所有执行命令返回屏幕输出，不要忽略关键的输出内容。
2. 辅助查看系统日志/var/log/message
3. 如果是MySQL关联了其他服务。要同时查看相关服务的日志


### 3.8 配置及管理MySQL多实例数据库

#### 3.8.1 配置MySQL多实例数据库开机自启动
服务的开机自启动很关键，MySQL多实例的启动也不例外。

> **登陆MySQL测试**

```
[root@multi 3306]# mysql -S /data/3306/mysql.sock  #←socket用于区别不同的实例
Welcome to the MySQL monitor.  Commands end with ; or \g.
.....
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

#### 3.8.2 MySQL多实例数据库的管理方法

MySQL安装完成后，默认root账户是没有密码的，登陆不同的实例需要指定不同实例的socket路径及文件，在my.cnf中指定的。注意：不同的socket虽然名字相同，但是路径是不同的，因此是不同的文件。

```bash
mysql -S /data/3306/mysql.sock
mysql -S /data/3307/mysql.sock
```

#### 3.8.3 重启多实例

若要重启多实例数据库也需要进行相应的配置。在重启数据库前，需要调整不同实例启动文件里对应的数据库密码：

```bash
sed -n '13p' mysql
sed -i 's#mysql_pwd="oldboy"#mysql_pwd="111"#g' mysql ../3307/mysql
sed -i 's#mysql_pwd="oldboy"#mysql_pwd="111"#g' mysql ../3307/mysql
sed -n '13p' mysql
```

由于选择了`mysqladmin shutdown`的停止方式，所以停止数据库时，需要在启动文件里配置数据库的密码。上面由于密码不对，顾提示密码不对的错误

```bash
[root@multi 3306]# /data/3306/mysql stop
Stoping MySQL...
/app/mysql/bin/mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: YES)'
[root@multi 3306]# /data/3306/mysql stop -S /data/3306/mysql.sock   
Stoping MySQL...
```

***
提示：禁止使用pkill、kill -9 killall -9等命令强制杀死数据库，这回引起数据库无法启动等故障发生。
***

企业血的教训案例请看 http://oldboy.blog.51cto.com/2561410.1431161

### 3.9 多实例MySQL登陆问题

### 3.9.1 多实例本地登陆MySQL

多实例本地登陆一般是通过socket文件来指定具体登陆到那个实例的，此文件的具体位置是在mysql编译过程或my.cnf文件里指定的。在本地登陆数据库时，登陆程序会通过socket文件来判断登陆的是哪个数据库实例。

例如：通过mysql -uroot -p111 -S /data/3307/mysql.sock可知，登陆的是3307实例。mysql.sock是MySQL服务器端与本地MySQL客户端进行通信的Unix套接字文件。

#### 3.9.2 远程连接登陆mysql

远程登陆MySQL多实例中的一个实例时，通过TCP端口(port)来指定所要登陆的MySQL实例，此端口的配置是在MySQL配置文件my.cnf指定的

例如：在`mysql -uroot -p111 -h192.168.1.2 -P3307 -P`为端口参数，后面接具体的实例端口，端口是一种 “逻辑连接位置” ，是客户端程序被分派到计算机上特殊服务程序的一种方式，强调提前在192.168.1.2上对该用户做授权。

```sql
DROP USER 'monitor'@'%';

mysql> select user,host from mysql.user;
+----------+------------+
| user     | host    |
+----------+------------+
| root     | 127.0.0.1  |
| root     | localhost  |
+----------+------------+

--MySQL的用户加主机名组成一个用户

mysql> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
```


## 4 常见错误


### 4.1 安装报错解决


#### 4.1.1 `Access denied for user 'root'@'localhost' (using password: NO)`
```sql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```

问题原因：密码不对哦

#### 4.1.2 `Error: Can't create/write to file`
```sql
Error: Can't create/write to file '/tmp/#sql_4f4_0.MYD' (Errcode: 17)
```

问题原因： 权限问题 

解决方法：`chmod -R 1777 /tmp`

#### 4.1.3 `ERROR 2002 (HY000): Can't connect to local`

```sql
ERROR 2002 (HY000): 
Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

问题原因：⑴ 服务没启动 ⑵ 配置文件中socket文件与编译时指定路径不一致


解决方法：⑴ 启动服务 ⑵ 多实例启动指定socket文件

#### 4.1.4 `error while loading shared libraries:`

```bash
[root@centos tools]# /app/mysql/scripts/mysql_install_db \
--basedir=/app/mysql/ \
--datadir=/app/mysql/data \
--user=mysql
Installing MySQL system tables...
/app/mysql//bin/mysqld:error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
```

问题原因：缺少libaio包支持

解决方法：安装后重新初始化即可

```bash
yum -install libaio* -y
```

#### 4.1.5 `WARNING: The host 'xxx'`

```bash
WARNING: The host 'centos' could not be looked up with resolveip.
```

问题原因：需要修改主机名的解析，使其和uname -n一样

#### 4.1.6 `ERROR: 1004 Can’t create file`

```bash
ERROR: 1004 Can’t create file '/tmp/#sql300e_1_0.frm' (error: 13)
```

解决：原因是/tmp权限有问题(不解决，后面可能无法登陆数据库)


### 4.2 使用错误解决

#### 4.2.1 ERROR 2006

在大批量导入数据库时，出现如下错误

```bash
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
```

排查思路：后来检查了没有导入成功的几篇文章，其大小都在8MB以上，会不会是单条记录太大了导致出现`ERROR 2006 (HY000): MySQL server has gone away`的呢？

**查看允许的最大值**

登陆MySQL后，使用如下命令查询：

```sql
mysql> show global variables like 'max_allowed_packet';   
+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 8388608 |
+--------------------+---------+
```

上限是刚好8MB，怪不得报错。

> 即时生效方法

```sql
set global max_allowed_packet=1024*1024*16;
```

可在不重启MySQL的情况下立即生效，但是重启后就会恢复原样。

> 永久生效方法

编辑 `/etc/my.cnf` ，将

```sql
max_allowed_packet = 1M
```

修改为

```sql
max_allowed_packet = 16M
```

之后重新导入，就不会产生ERROR 2006 (HY000): MySQL server has gone away错误了。
5.6 Unknown/unsupported storage engine: Innodb
Plugin 'InnoDB' init function returned error.


```sql
mysql> show warnings;
+---------+------+----------------------------------------+
| Level   | Code | Message                                |
+---------+------+----------------------------------------+
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
| Warning | 1292 | Truncated incorrect DOUBLE value: 'Y ' |
+---------+------+----------------------------------------+
64 rows in set (0.00 sec)
```


