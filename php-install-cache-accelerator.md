# 配置PHP插件


[toc]

### PHP缓存加速器介绍

#### 操作码介绍及缓存原理

当客户端请求一个PHP程序时，服务器的PHP引擎会解析该PHP程序，并将其编译为特定的操作码（Operate Code，简称opcode）文件，该文件是执行PHP代码后的一种二进制表示形式。默认情况下，这个编译好的操作码文件由PHP引擎执行后丢弃。而操作码缓存（Opcode Cache）的原理就是将编译后的操作码保存下来，并放到共享内存里，以便在下一次调用该PHP页面时重用它，避免了相同代码的重复编译，节省了PHP引擎重复编译的时间，降低了服务器负载，同时减少了CPU和内存开销。

#### PHP缓存加速软件介绍

为了提高PHP引擎的高并发访问及执行速度，产生了一系列PHP缓存加速软件。这些软件设计的目的就是缓存前文提到的PHP引擎解析过的操作码文件，以便在指定时间内有相同的PHP程序请求访问时，不再需要重复解析编译，而是直接调用缓存中的PHP操作码文件，这样就提高了动态Web服务的处理速度，从而提升了用户访问企业网站的整体体验。

#### LAMP环境PHP缓存加速器的原理

下面简单介绍Apache环境的PHP缓存加速器原理。

在LAMP环境中，Apache服务是使用libphp5.so响应处理PHP程序请求的，整个流程大概如下：

1. Apache接收客户的PHP程序请求，并根据规则过滤之。

2. Apache将PHP程序请求传递给PHP处理模块libphp5.so。

3. PHP引擎定位磁盘上的PHP文件，并将其加载到内存中解析。

4. PHP处理模块libphp5.so将PHP源代码编译成为opcode。

5. PHP处理模块libphp5.so执行opcode，然后把opcode缓存起来。

6. Apache接收客户端新的PHP程序请求，PHP引擎直接读取缓存执行opcode文件，并将结果返回。在这一次任务中，就无第4步的编译解
   析了，从而提升了PHP编译解析效率。

   

PHP缓存加速器解决的是上述第5步的问题，默认情况下PHP会将opcode内容执行后丢弃，这里却通过PHP缓存加速软件，将opcode内容缓存了下来，目的是当有重复请求时，不需要再重复编译解析PHP程序代码，因为在高并发高访问量的网站上，大量的重复编译会消耗很多的系统资源和时间，而这也就会成为瓶颈，既影响了处理速度，又加重了服务器的负载，为了解决此问题，PHP缓存加速器就这样诞生了。

![image-20221025224840040](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025224840040.png)

<center>图4-1是LAMP环境下PHP请求及操作码缓存过程的原理示意图</center>

#### LNMP环境PHP缓存加速器的原理详解

在LNMP环境中，PHP引擎不再使用libphp5.so模块了，而是启动了独立的FCGI即php-fpm进程，由它监听来自Nginx的PHP程序请求，并交给PHP引擎解析处理，整个执行流程大概如下：

#### PHP缓存加速器软件种类及选择建议



PHP缓存加速器软件常见的种类有XCache、eAccelerator、APC（Alternative PHP Cache），ZendOpcache等，那么，在企业环境我们要如何选择PHP缓存加速器软件呢？



事实上，任选其一即可，没必要都安装上，都安装也可能会发生冲突。总的建议就是根据企业的业务需求及选择前的压力测试结果，或者根据个人的经验偏好选择。不过，老男孩建议首选XCache，其次是eAccelerator，如果想尝新，可以选择ZendOpcache。

> **首选XCache的原因如下：**

* 经过测试，XCache效率更高、速度更快。
* XCache软件开发社区更活跃，最新版2014年底发布。
* 支持更高版本的PHP，例如PHP 5.5、PHP 5.6。

> **次选eAccelerator的原因如下：**

* 安装及配置参数更简单，加速效果也不错。
* 文档资料较多，但官方对软件的更新很慢，社区不活跃。
* 仅适合PHP版本5.4以下的程序。
> **选择ZendOpcache的原因如下：**

* 是PHP官方研发的新一代缓存加速软件，以后的发展潜力可能会很好，PHP 5.5以前的版本可以通过ZendOpcache软件以插件扩展的方式安装，从PHP 5.5版本开始已经整合到PHP软件里了，编译时只需指定一个参数即可，例如：--enable-opcache。
* ZendOpcache可能是未来的缓存加速首选，现在的稳定性还有待检验，小规模环境下PHP 5以前的版本可以通过插件式安装使用，PHP 5以上的版本可以直接指定参数编译使用。若可以忍受ZendOpcache的各种未知问题的话，也可以尝试使用。

### 安装PHP缓存加速器扩展

#### 安装PHP eAccelerator缓存加速模块

> **eAccelerator缓存加速插件说明**

eAccelerator是一个免费的、开放源代码的PHP加速、优化及缓存的扩展插件软件，它可以缓存PHP程序编译后的中间代码文件（opcode）、session数据等，降低PHP程序在编译解析时对服务器的性能开销。eAccelerator还可以加快PHP程序的执行速度，降低服务器负载压力，使PHP程序代码执行效率提高1~10倍。

eAccelerator会把编译好的PHP程序存放在共享内存里，然后每次从内存里调用执行，可以设定把一些不适合放在内存里缓存的编译结果存储到磁盘上，默认情况下，磁盘和内存缓存都会被eAccelerator使用。

eAccelerator的最新版为0.9.6.1，支持的PHP最新版本为PHP 5.3及以前5系列的版本。
早期的0.9.5版本支持PHP 4和PHP 5.2以前的版本。

eAccelerator下载地址为：https://github.com/eaccelerator/eaccelerator/downloads。

> **eAccelerator插件安装过程**

具体的安装命令集如下：
```bash
/app/php/bin/phpize
./configure --enable-eaccelerator=shared --with-php-config=/app/php/bin/php-config
```

#### 安装PHP XCache缓存加速模块
> **XCache缓存加速插件说明**



XCache是一个开源的、又快又稳定的PHP opcode缓存器/优化器，其项目leader曾经是Lighttpd（和Nginx类似的高速Web服务软件）的开发成员之一。XCache把PHP程序编译后的数据（opcode）缓存到共享内存里，避免相同的程序重复编译。用户请求相同的PHP程序时，可以直接使用缓存中已编译好的数据，从而提高PHP的访问速度，通常可以提升2~5倍，并大幅降低服务器负载开销。

很多公司使用XCache，它已经能在大流量/高负载的生产环境中稳定运行，与同类型的opcode缓存器相比在各个方面都更胜一筹，例如：社区活跃、快速开发、能够快速跟进PHP的版本更新等。
当前稳定版本为3.1.x（全面支持PHP 5.1~5.5）和3.2.x（2014年底发布，全面支持PHP 5.1~5.6）。

>XCache软件详情请参考：
>
>http://xcache.lighttpd.net
>
>http://xcache.lighttpd.net/wiki/Introduction。

**XCache插件的安装过程**

```bash
/app/php/bin/phpize
./configure --enable-xcache   --with-php-config=/application/php/bin/php-config
./configure --enable-xcache=shared --with-php-config=/app/php/bin/php-config
```

#### PHP官方加速插件ZendOpcache

> **ZendOpcache插件说明**

从PHP 5.5开始，官方已经集成了新一代的缓存加速插件，其名字为ZendOpcache，功能和前三者相似但又有少许不同，据官方说，ZendOpcache缓存速度更快。



这几个PHP加速插件的主要原理基本相同，就是把PHP执行后的数据缓存到内存中从而避免重复的编译过程，使其能够直接使用缓存中已编译的代码，从而提高速度，降低服务器负载。它们的效率是显而易见的，	一些大型的CMS，每次打开一个页面要调用数十个PHP文件，执行数万行代码，效率可想而知，安装上述加速器后，打开页面的速度明显加快。



PHP 5.5以上版本，支持ZendOpcache很简单，只需在编译PHP 5.5时加上--enable-opcache就行了。其实，在PHP5.5版本以前，ZendOpcache也有独立的软件，并且也支持低版本的`PHP 5.2.*`、`PHP5.3.*`、`PHP 5.4.*`。



官方下载地址为：http://pecl.php.net/package/ZendOpcache。

> **ZendOpcache插件安装过程**

在PHP源码包ext目录下有一些扩展库，可直接安装，也可去官网下载扩展后安装
```bash
/application/php/bin/phpize
./configure --enable-opcache --with-php-config=/application/php/bin/php-config
```
最后需要在php.ini里开启opcache模块才可使用
```bash
[Zend Opcache]
zend_extension = /app/php5.6.26/lib/php/extensions/no-debug-non-zts-20131226/opcache.so
opcache.memory_consumption=64
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.force_restart_timeout=180
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable_cli=1
```

![image-20221025224933301](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025224933301.png)



### 安装数据库缓存及其他PHP扩展插件

#### 安装PHP Memcached扩展插件

<center>图4-3是Memcached缓存架构逻辑图。</center>


![image-20221025224945336](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025224945336.png)

PHP的Memcached扩展插件下载地址为：http://pecl.php.net/package/memcache。

#### 安装PDO_MYSQL扩展模块

> **PDO_MYSQL扩展插件说明**

PDO扩展为PHP访问数据库定义了一个轻量级一致性的接口，它提供了一个数据访问抽象层，这样，无论使用的是什么数据库，都可以通过一致的函数执行查询并获取数据。



PDO_MYSQL扩展插件下载地址为：http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz。



技巧：php源码包有或使用谷歌搜索关键字“PDO_MYSQL-1.0.2.tgz download”。

> **PDO_MYSQL扩展插件的安装过程**

PDO_MYSQL的安装有两种方法，一种是插件方式安装，另一种是编译PHP时加入PDO_MYSQL支持，直接指定PHP的对应PDO_MYSQL编译参数安装，例如：`--with-pdo-mysql=mysqlnd`，同时PHP的环境也可以不装MySQL软件，直接指定如下参数`--with-mysql=mysqlnd`，即可让PHP支持连接MySQL数据库。

### 安装其他的PHP扩展插件模块

#### 安装ImageMagick图像软件

ImageMagick是一套功能强大、稳定而且免费的工具集和开发包，可以用来读、写和处理超过89种基本格式的图片文件，包括流行的tiff、jpeg、gif、png、pdf，以及PhotoCD等。利用ImageMagick，可以根据Web应用程序的需要动态生成图片，还可以对一个（或一组）图片进行改变大小、旋转、锐化、减色或增加特效等操作，并将操作的结果以相同格式或其他格式保存。对图片的操作，即可以通过命令行进行，也可以用C/C++、Perl、Java、PHP、Python或Ruby编程来完成。同时ImageMagick提供了一个高质量的2D工具包，部分支持SVG。现在，ImageMagic的主要精力集中在加强性能、减少bug，以及提供稳定的API和ABI上。

ImageMagick的常见功能如下：

* 将图片从一个格式转换成另一个格式，包括直接转换成图标。
* 可以改变图片尺寸，旋转、锐化（sharpen）、减色，设置图片特效。
* 对图片设置各种尺寸缩略图。
* 将图片设置为可以适应于Web背景的透明图片。
* 将一组图片做成gif动画，直接convert。
* 将几张图片做成一张组合图片。
* 在一个图片上写字或画图形，带文字阴影和边框渲染。
* 给图片加边框或框架。
* 取得一些图片的特性信息。

它几乎包括了gimp可以实现的所有常规插件功能，甚至包括各种曲线参数的渲染功能。ImageMagick的下载地址为：

http://pecl.php.net/package/imagick

https://www.imagemagick.org/download/

#### ImageMagick安装报错及解决方法。
以下错误是书上写的。我在两天centos6 与centos7共计4台测试并未发现任何错误

问题1：make步骤出错。
示例如下：

```bash
cd PerlMagick && /usr/bin/perl Makefile.PL
Can't locate ExtUtils/MakeMaker.pm in @INC(@INC contains:/usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .)
at Makefile.PL line 24.
BEGIN failed--compilation aborted at Makefile.PL line 24.
make[1]:*** [PerlMagick/Makefile] Error 2
make[1]:Leaving directory `/home/oldboy/tools/ImageMagick-6.5.1-2'
make:*** [all] Error 2
```
可以看到，上述内容中有Makefile.PL、/usr/lib64/perl5/vendor_perl和Perl语言的字样，因此可以试着使用 `yum install perl-devel -y` 命令安装相关包，看看是否可以解决问题。

#### 安装imagick PHP扩展插件

imagick插件工作需要ImageMagick软件的支持，所以，必须要先安装ImageMagick，否则会报错。

imagick插件是一个可以供PHP调用ImageMagick功能的扩展模块。使用这个扩展可以使PHP具备和ImageMagick相同的功能。

安装了ImageMagick图像程序后，再安装PHP的扩展imagick插件，才能使用ImageMagick提供的api进行图片的创建与修改、压缩等操作，因为它们都集成在imagick这个PHP扩展中。

```bash
./configure \
--with-php-config=/app/php/bin/php-config \
--with-imagick=/app/imagemag/	#←如果编译安装的ImageMagick需要指定安装路径
```

### 配置PHP加速与缓存相关的扩展插件模块

#### 配置xcache/PDO_MYSQL/imagick模块生效

> **修改PHP的配置文件php.ini**

修改php.ini的配置文件过程如下：

查找`extension_dir="./"`参数，修改为`extension_dir="/app/php5.3.27/lib/php/extensions/no-debug-non-zts-20090626/"`，这个`extension_dir`对应的路径就是前文编译的模块所在的路径。

***
**<font color="#0215cd" size=2> <font color="#f8070d" size=2>⚠</font> 提示：默认的PHP配置文件路径为 `/app/php/lib/php.ini` ，可以通过在编译PHP时添加参数指定php.ini的配置路径<font color="#f8070d" size=2> `--with-config-file-path=/application/php5.3.27/lib/etc` </font>，如果不指定编译路径，默认为<font color="#f8070d" size=2>` /application/php/lib/ `</font>。</font>**

***
> **在vim命令模式下按Shift+G键跳到文件结尾，增加如下几行，然后保存：**

```bash
extension = memcache.so
extension = pdo_mysql.so
..
extension = imagick.so
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025225007761.png)



![image-20221025225016393](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025225016393.png)



#### 配置eAccelerator插件生效并优化参数

> **1 配置eAccelerator缓存目录**

配置命令1：配置eAccelerator缓存目录：
```bash
mkdir -p /tmp/eaccelerator 	#←此目录可以用tmpfs内存文件系统或SSD固态硬盘来存储。
chown -R nginx.nginx /tmp/eaccelerator #←chown后的用户是nginx的用户
```
配置命令2：配置eAccelerator参数，命令如下：
```bash
[eaccelerator]
extension=eaccelerator.so
eaccelerator.shm_size="64"
eaccelerator.cache_dir="/tmp/eaccelerator"
eaccelerator.enable="1"
eaccelerator.optimizer="1"
eaccelerator.check_mtime="1"
eaccelerator.debug="0"
eaccelerator.filter=""
eaccelerator.shm_max="0"
eaccelerator.shm_ttl="3600"
eaccelerator.shm_prune_period="3600"
eaccelerator.shm_only="0"
eaccelerator.compress="1"
eaccelerator.compress_level="9"
```
> **2 eAccelerator配置参数的详细说明。**

| **eaccellerator**                          | **解释说明**                                                 |
| :----------------------------------------- | :----------------------------------------------------------- |
| [eaccelerator]                             | 开始eAccelerator加速模块配置                                 |
| extension                                  | 加载eaccelerator加速模块，路径相对于extension_dir的配置      |
| eaccelerator.shm_size                      | 存储缓存数据的共享内存大小，如果为0，则最大值看内核配置/proc/sys/kernel/shmmax |
| eaccelerator.cache_dir="/tmp/eaccelerator" | 磁盘缓存存储路径，缓存内容为precompiled code、session、data、content和user entries。默认路径为"tmp/eaccelerator" |
| eaccelerator.enable="1"                    | 加速PHP代码执行速度，1为默认值，表示激活；0为不激活。用于缓存前的代码加速 |
| eaccelerator.check_mtime="1"               | 检查缓存修改时间，决定代码是否需要重新编译，1为激活，是默认值 |
| eaccelerator.debug="0"                     | 缓存加速调试，0为关闭，1为打开，打开后可以看到缓存命中信息   |
| eaccelerator.filter=""                     | 设定对象是否缓存规则，空表示不设定                           |
| eaccelerator.shm_max="0"                   | 可以被放置的缓存最大值，0是不限制                            |
| eaccelerator.shm_ttl="3600"                | 缓存文件的生存期                                             |
| eaccelerator.shm_prune_period=3600"        | 当共享内存空间不够时，从共享内存中移除老数据的时间周期       |
| eaccelerator.shm_only="0"                  | 是否允许缓存数据到磁盘，0为允许，但是对于session data and content caching无影响 |
| eaccelerator.compress="1"                  | 是否开启压缩，1为开启                                        |
| eaccelerator.compress_leve="9"             | 压缩级别，9为最高   |

根据内容指定是否缓存到共享内存或磁盘的其他参数：

```bash
eaccelerator.keys="shm_and_disk" #←控制keys缓存位置
eaccelerator.sessions="shm_and_disk" #←控制sessions缓存位置
eaccelerator.content="shm_and_disk" #←控制内容缓存位置上述3个参数可选的值为：
shm_and_disk：cache data in shared memory and on disk(default value)
shm:cache data in shared memory or on disk if shared memory is full or data size greater then "eaccelerator.shm_max"
shm_only:cache data in shared memory 
disk_only:cache data on disk
none:don't cache data
```

更多信息请参考 https://github.com/eaccelerator/eaccelerator/wiki/Settings。

> **3 访问PHP页面测试检查eAccelerator加速情况**
```bash
[root@lamp-02 htdocs]# find /tmp/eaccelerator/ -type f
/tmp/eaccelerator/48/0/1/eaccelerator-01aa58b81ae5ff3ed966dbbac55535a8
/tmp/eaccelerator/48/0/2/eaccelerator-02399225c2489318da660dc2213a940e
...
/tmp/eaccelerator/48/0/3/eaccelerator-03621af70cbe37e82c125a39bdb8c0b9
/tmp/eaccelerator/48/0/3/eaccelerator-03ad092ef38eaae48de869a58a893a16
```

> **4 使用tmpfs优化eAccelerator缓存目录**

tmpfs是一种基于内存的文件系统，通常使用tmpfs作为数据临时存储，比本地磁盘存储快很多，此方法适用于临时使用的各类缓存场景。例如：上传图片时很多软件默认在/tmp下临时缓存切图、存放session数据，则可以让/tmp使用tmpfs文件系统来加快访问效率。
```bash
mount -t tmpfs  -o size=16m tmpfs /tmp/eaccelerator
```

### 配置XCache插件加速

**XCache配置文件参数**

| 参数    | 说明    |
| ---- | ---- |
| [xcache-common] <br/>extension=xcacheso | 加载xcache.so,路径相对于extension_dir的配置。<br/>自3.0版本开始不再支持使用zend_extension加载XCache的方式 |
| [xcache.admin]<br/>xcache.admin.enable_auth=On | 激活管理员认证 |
| xcache.admin.user="mOo"<br/>xcache.admin.pass="md5<br/>encrypted password" | 指定XCache管理员用户名和密码.密码根据http://xcache. lighttpd.net/demo/cacher/mkpassword php 地址产生，留空表示禁止管理页面 |
| [xcache] | 开始XCache缓存参数配置段.下面所有的初始值即为默认值，除非明确说明 |
| xcache.shm_scheme="mmap" | 设置XCache如何从系统分配共享内存 |
| xcache.size = 60M | 0为禁止缓存，非0则启动缓存。需要注意系统所允许的mmap最大值 |
| xcachc.count = 1 | 指定将Cache切分成多少块，官方方推荐设置为服务器CPU的数量<br/>grep -c processor /proc/cpuinfo 1 |
| xcache.slols=8K | hash槽个数的参考值.缓冲超过此数值时也没有任何问题（you can always store count(items) > slots) |
| xcache.gc_interval = 0 | 回收器扫描过期的对象回收内存空间的间隔.0为不扫描，其他值的单位是秒 |
| xcache.var_size = 4M<br>xcache.var_count = 1 <br>xcache.var_slots = 8K<br>xcache.var_ttl = 0 <br>xcache.var_gc_interval =300 | 这几个值和上面的几个类似，只不过用于变量缓存，而不是 opcode缓存 |
| ;N/A for /dev/zero xcache.readonly_proteciion = off | 如果启用了该参数.将略微降低性能，但会提高一定的安全系数。 这个选项对于 xcache.mmap_path = /dev/zero 无效 |
| xcache.mmap_path="/dev/zero" | 对于*nix, xcache.mmap_path是一个文件路径而非目录。如果要启 用该参数，请使用"/tmp/xcache"这样的路径，而不是"/dev/*"。如 果开启了 xcache.readonly_protection参数，不同进程组的PHP将不会共享同一个/tmp/xcache路径 |
| xcache.coredump_directory="" | 当XCache crash后，适否把数据保存到指定路径 |
| xcachc.disable_on_crash =off | 当XCache发生crash时，自动动关闭XCache缓存 |


更多参数请参考官方文档：http://xcache.lighttpd.net/wiki/XcacheIni

**编辑xcache.ini，修改XCache的配置参数**

```bash
xcache.size=256M
xcache.count=2
xcache.ttl=86# 24小时
xcache.gc_interval=3600
xcache.var_size=64M
```

**将修改后的xcache.ini合并到php.ini结尾。**
```bash 
cat /home/oldboy/tools/xcache-3.2.0/xcache.ini >>php.ini
```
**检查XCache加速情况配置**

```bash
Warning: Module 'XCache' already loaded in Unknown on line 0
# 出现这样的提示，应该是加载xcache原因 ，取消后提示消失
/app/php/bin/php -v
PHP 5.5.20 (cli) (built: Oct 16 2016 21:28:19)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
    with XCache v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Cacher v3.2.0, Copyright (c) 2005-2014, by mOo
```
**复制XCache软件下面的缓存加速管理PHP程序到站点目录下**
```bash
cp -r ~/tools/xcache-3.2.0/htdocs /var/html/www/xadmin
chown -R www.www /var/html/www/xadmin
```

**编辑php.ini文件，xcache.admin模块配置如下**

```bash
[xcache.admin]
xcache.admin.enable_auth = On
xcache.admin.user = "admin"
xcache.admin.pass = "111"
```

**MD5加密可用如下方法生成**

```bash
[root@Lamp ~]# echo 111|md5sum
1181c1834012245d785120e3505ed169  - #←这里生成的并不是md5生成的111，因为echo默认有换行
[root@Lamp ~]# echo -n 111|md5sum
698d51a19d8a121ce581499d7b701668  -
```
![image-20221025225045277](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025225045277.png)

#### 配置ZendOpcache插件加速

> **1 配置ZendOpcache参数**

在php.ini里添加如下配置：
```bash
[opcache]
extension=opcache.so    #←这种方法不能用了
zend_extension=/application/php5.3.27/lib/php/extensions/no-debug-non-zts-
20090626/opcache.so;
extension=opcache.so
opcache.memory_consumption=32
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=1000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable_cli=1
```
![image-20221025225121076](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025225121076.png)

> **2 ZendOpcache配置参数说明**

**下表Opcache的部分重要参数进行了说明。**

| OPcache参数解释说明                |                                                              |
| ---------------------------------- | ------------------------------------------------------------ |
| opcache.memoiy_consumption=128     | OPcache共享内存空间大小，用于存放precompiled PHP code，默认为64，单位为Mbytes |
| opcache.interned_strings_buffer=8  | 默认依为4,interned strings内存的数量,单位是M                 |
| opcache.max_accelerated_files=4000 | 默认值为2000, OPcache散列表的key的最大数量                   |
| opcache.revalidate_freq=60         | 默认值为2,检查文件时间戳的频率，用于共享内存分配的变化       |
| opcache.fast_shutdown=l            | 默认值为0,如果激活,一个快速的关闭队列将被用来加速代码        |
| opcache.enable_cli=l               | 默认值为0,激活PHP CLI的OPcache, 于测试和调试                 |                                                            |

**<font style="color:blue;" size=2>更多的OPcache参数可以查看安装目录下的README </font>**

说明：ZendOpcache是PHP官方的新一代的缓存加速软件，PHP5.5以前可以通过ZendOpcache软件以插件扩展的方式安装，从 ==PHP5.5== 版本开始已经整合到PHP软件里，编译时只需指定一个参数即可，例如： `--enable-opcache`。

ZendOpcache可能是未来的首选，现在的稳定性还有待检验。在小规模环境下，PHP 5以上的版本可以使用。如果可以忍受其未知的问题也可以使用。

#### 生产环境PHP扩展插件的安装建议

> **1 PHP的安装插件表格列表**

| PHP EXT module      | 说明                                                  |
| ------------------- | ----------------------------------------------------- |
| eaccelerator0.9.5.2 | 适合PHP5.3以前的版本，PHP缓存加速                     |
| eaccelerator-0.9.6  | 适合PHP5.3版本，PHP缓存加速                           |
| ImageMagick         | 常用图像处理程序，属功能应用                          |
| imagick             | 需要先装图像处理程序，属功能应用                      |
| memcache            | memcache客户端数据库缓存优化应用                      |
| memcached           | 基于libmemcache客户端，性能较memcache更好，高并发首选 |
| PDO_MYSQL           | PHP数据库访问插件，属功能应用                         |
| xcache              | 支持PHP5.1-5.6，PHP缓存加速                           |
| zend opcache        | zend官方PHP缓存加速插件                               |

> **2 生产环境插件的安装建议**

对于功能性插件，如果业务产品不需要使用，可以暂时不考虑安装，例如

PDO_MYSQL\memcache\imagick 等。如果不清楚是否需要，最好还是装上，有备无患。

对于性能优化插件，eAccelerator、XCache、ZendOpcache、APC可以安装任一种，具体情况看实际业务需求，在选择时最好能搭建相关环境进行压力测试，然后根据实际测试结果来选择，用数据说话很重要。
> **3 PHP加速插件的测试对比**

eAccelerator不支持5.3以上版本

xcache支持7.0以下版本。

#### PHP缓存加速压力测试练习
分别安装ZendOpcache、eacc、XCache缓存加速插件，通过压测软件对比三者的缓存效率。
测试方法：
1. 不装任何加速插件和分别安装某一个缓存加速软件。
2. 可用压力测试软件webbench、loadruner。
3. 用压力测试方法，通过数据看看到底哪个加速器好
