# 编译安装PHP

[toc]

###  下载PHP

- 台湾镜像站：http://ftp.ntu.edu.tw/php/distributions/
- 搜狐镜像站：http://mirrors.sohu.com/php/
- 阿里镜像：http://mirrors.aliyun.com/
- 官网：http://php.net/downloads.php

### 检查PHP所需的lib库

```bash
rpm -qa \
zlib-devel \
libxml2-devel \
libjpeg-devel \
libjpeg-turbo-devel \
libiconv-devel \
freetype-devel \
libpng-devel \
gd-devel \
libcurl-devel \
libxslt-devel
```
***
**&lt;font color=#0215cd size=2&gt;提示：libjpeg-turbo-devel是早期libjpeg-devel的新名字，libcurl-devel是早期curl的新名字。&lt;/font&gt;**
***


每个lib一般都会存在对应的以“*-devel*”命名的包，安装lib对应的-devel包后，对应的lib包就会自动安装好，例如安装gd-devel时就会安装gd。

这些lib库不是必须安装的，但是目前的企业环境下一般都需要安装。否则，PHP程序运行时会出现问题，例如验证码无法显示等。

执行下面命令安装相关的lib软件包：
```bash
yum install -y \
zlib-devel \
libxml2-devel \
libjpeg-devel \
libjpeg-turbo-devel \
freetype-devel \
libpng-devel \
gd-devel \
curl-devel \
libxslt-devel \
bzip2-devel \
gmp-devel \
readline-devel
```
***
**&lt;font color=#0215cd size=2&gt; 提示：从安装上看，仅有libiconv-devel这个包没有安装，因为默认的yum源没有此包。可以一个一个地yum安装或通过源文件手工编译安装（这样效率慢）&lt;/font&gt;**
***

#### 安装libiconv-devel

libiconv下载地址：http://ftp.gnu.org/pub/gnu/libiconv/

可以将libiconv制作成rpm包，批量安装时，可放至本地yum源内

```bash
./configure --prefix=/usr/local/libiconv
make &amp; make install
```
#### 安装epel源

可以安装redhat官方yum源里没有的软件，epel源和官方源不冲突

阿里镜像 http://mirrors.aliyun.com/ 

```bash
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
```
Centos7
```bash
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```
### 安装libmcrypt-devel

这是一个使用动态加载的模块化的libmcrypt。libmcrypt对于在程序运行时添加/移除算法是有用的。limbcrypt-nm目前不再被官方支持，其软件地址为`http://mcrypt.hellug.gr/lib/`，编译PHP的过程中，libmcrypt库不是必须要安装的包。
```bash
yum install libmcrypt-devel -y
# 安装成功后
rpm -qa libmcrypt-devel
libmcrypt-devel-5.8-9.el6.x86_64
```
### 安装mhash加密扩展库

mhash是基于离散数学原理不可逆向的PHP加密方式扩展库，其在默认情况下不会开启。mhash可以用于创建校验数值、消息摘要、消息认证码，以及无需原文的关键信息保存（如密码）等。它为PHP提供了多种散列算法，如MD5、SHA1、GOST等。可以通过MHASH_hashname()查看其支持的算法有哪些。
```bash
yum install mhash -y
```
### 安装mcrypt

PHP程序员在编写代码程序时，除了要保证代码的高性能之外，还有一点是非常重要的，那就是程序的安全性保障。PHP除了自带的几种加密函数外，还有功能更全面的PHP加密扩展库mcrypt和mhash。
其中，mcrypt扩展库可以实现加密解密功能，就是既能将明文加密，也可以将密文还原。
可以说，mcrypt是PHP里面重要的加密支持扩展库，该库在默认情况下不开启。
```bash
yum install mcrypt -y
```
```bash
yum install mcrypt mhash libmcrypt-devel -y
```

问：如果在不能联网的状态下怎么配置PHP环境？
答：在yum时，可以在yum配置文件中设置安装后不删除包
vi /etc/yum.conf

### 编译PHP

#### 编译PHP参数详解

```bash
./configure 
--prefix=/app/php-7.  # &lt;==指定PHP的安装目录
--with-curl 	 			    # &lt;==打开PHP curl扩展
--with-curlwrappers 		# &lt;==curl工具打开url流的测试版，不建议开启
--with-freetype-dir 		# &lt;== 打开freetype字体库支持
--with-gd 					    # &lt;==打开PHP GD库依赖
--with-png-dir 			    # &lt;==
--with-jpeg-dir 			  # &lt;==
--enable-gd-native-ttf  # &lt;==打开PHP GD库ttf
--with-gettext 			    # &lt;==打开PHP gettext库，多国语言时需要
--with-iconv-dir=/usr/local/lib 	# &lt;==打开PHP iconv字符集转换格式时需要
--with-kerberos 			  # &lt;==PHP的一种加密方式 DES
--with-libdir=lib64 		# &lt;==默认找/usr/lib下。64位需指定路径
--with-libxml-dir 			# &lt;==打开libxml2库的支持
--enable-xml
--enable-safe-mode 		  # &lt;==打开安全模式
# 需要指定mysql的安装路径,安装PHP需要的MySQL相关内容。
# 当然如果没有MySQL软件包，也可以不单独安装，
# 这样的情况可使用--with-mysql=mysqlnd替代--with-mysql=/app/mysql
# 因为PHP软件里面已经自带连接MySQL的客户端工具。PHP7遗弃
--with-mysql=/app/mysql/
--with-mysqli=/app/mysql/bin/mysql_config 		# &lt;==使用PHP mysqli扩展
--with-pdo-mysql 			  # &lt;==使用pdo mysql扩展 
--with-pdo-sqlite 			# &lt;==使用pdo sqlite扩展
--with-openssl 			    # &lt;==https需要
--with-pcre-regex 			# &lt;==使用pcre正则	
--with-pear 				    # &lt;==安装pear，一般没啥用	
--with-xmlrpc 				  # &lt;==打开xml-rpc的c语言
--enable-libxml 
--disable-rpath 			  # &lt;==关闭额外的运行库文件
--with-xsl 					    # &lt;==打开XSLT文件支持,扩展libXML2库,需要libxslt软件
--with-zlib 				    # &lt;==打开zlib库的支持,用于http压缩传输
--enable-zip 				    # &lt;==打开对zip的支持
--enable-bcmath 			  # &lt;==打开图片大小调整,用zabbix监控时会用到该模块
--enable-inline-optimization 	# &lt;==优化线程
--enable-mbregex 			  # &lt;==
--enable-mbstring 			# &lt;==支持mbstring
--with-mcrypt 				  # &lt;==编码函数库
--with-mhash 				    # &lt;==mhash算法的扩展
--enable-opcache 			  # &lt;==php自带的加速模块php5.5
--enable-pcntl 			    # &lt;==freeTDS需要用到,可能是链接mssql
--enable-shmop 			    # &lt;==
--enable-soap 				  # &lt;==soap模块的扩展
--enable-sockets 		 	  # &lt;==打开sockets支持
--enable-sysvsem 		 	  # &lt;==使用sysv信号机制,则打开此选项
--enable-short-tags 		# &lt;==开启短标签
--with-fpm-user=www 
--with-fpm-group=www 
--enable-fpm 				    # &lt;==表示激活PHP-FPM方式服务,即FactCGI方式运行PHP服务
--with-apxs2=/app/apache/bin/apxs # &lt;==使httpd支持PHP
--enable-json
--with-bz2
--enable-filter
--enable-session
```
#### 公共编译参数

```bash
./configure \
--prefix=/app/php-5.5.38 \
--with-curl \
--with-freetype-dir \
--with-gd \
--with-png-dir \
--with-jpeg-dir \
--enable-gd-native-ttf \
--with-gettext \
--with-iconv-dir \
--with-kerberos \
--with-libxml-dir \
--enable-xml \
--enable-safe-mode \
--with-mysql=/app/mysql/ \
--with-mysqli=/app/mysql/bin/mysql_config \
--with-pdo-mysql \
--with-pdo-sqlite \
--with-openssl \
--with-pcre-regex \
--with-pear \
--with-xmlrpc \
--enable-libxml \
--disable-rpath \
--with-xsl \
--with-zlib \
--enable-zip \
--enable-bcmath \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--with-mcrypt \
--with-mhash \
--enable-opcache \
--enable-pcntl \
--enable-shmop \
--enable-soap \
--enable-sockets \
--enable-sysvsem \
--enable-short-tags \
--enable-json \
--with-bz2 \
--enable-filter \
--enable-session \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-fpm \
--with-apxs2=/app/apache/bin/apxs
```
```bash
configure: error: Cannot find libmysqlclient under /app/mysql/.
```
解决：
```bash
ln -s /app/mysql/lib /app/mysql/lib64
ln -s /app/mysql/lib/libmysqlclient.so.15./app/mysql/lib64/libmysqlclient_r.so
```
#### nginx 5.3.27

```bash
./configure \
--prefix=/app/php-5.3.27 \
--with-mysql=/app/mysql \
--with-iconv-dir=/usr/local/libiconv \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir \
--with-libxml-dir \
--enable-xml \
--disable-rpath \
--enable-safe-mode \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--with-curl \
--enable-mbregex \
--enable-fpm \
--enable-mbstring \
--with-mcrypt \
--with-gd \
--enable-gd-native-ttf \
--with-openssl \
--with-mhash \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--enable-short-tags \
--enable-zend-multibyte \
--enable-static \
--with-xsl \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--enable-ftp
```
***
&lt;font color=#0215cd size=2&gt;注：上述每行结尾的换行符反斜线（\）之后不能再有任何字符包括空格&lt;/font&gt;
***
#### apache 5.3

```bash
./configure \
--prefix=/app/php-5.3.27 \
--with-mysql=/app/mysql \
--with-iconv-dir=/usr/local/libiconv \
--with-apxs2=/app/apache/bin/apxs \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir \
--with-libxml-dir \
--enable-xml \
--disable-rpath \
--enable-safe-mode \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--with-curl \
--with-curlwrappers \
--enable-mbregex \
--enable-mbstring \
--with-mcrypt \
--with-gd \
--enable-gd-native-ttf \
--with-openssl \
--with-mhash \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--enable-short-tags \
--enable-zend-multibyte \
--enable-static \
--with-xsl \
--enable-ftp
```
#### 配置php.ini

```bash
/home/tools/php-5.3.27/php.ini-production
/home/tools/php-5.3.27/php.ini-development
```

```bash
cp /home/tools/php-5.3.27/php.ini-production /app/php/lib/php.ini
```
开发环境更多的是开启日志、调试信息，而生产环境都是关闭状态。

#### 配置PHP（FastCGI）的配置文件php-fpm.conf

* PHP5位置：/app/php/etc/
* PHP7位置：/app/php/etc/和 php-fpm.d
```bash
cp php-fpm.conf.default php-fpm.conf # &lt;==PHP5只需要改它即可
php-fpm.d/www.conf.default # &lt;==PHP7还需要改这个文件
```

## 3 配置Nginx支持PHP程序

```nginx
# 这里如果配置不好，很容易出现404错误，
# 此处也可以吧两个localtion里的root html/blog合成一个 
 location  ~.*\.(php|php5)?$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
  	index  index.html index.htm;
 }  # &lt;==可将所有location里的root提出到外面。
```

![image-20221025225413522](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-2022102522541352png)

&lt;center&gt;原因：local中没有路径，要么加路径，要么提出最外面&lt;/center&gt;

![image-20221025225422603](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025225422603.png)



## 4 配置apache支持PHP

默认生成
```bash
$ grep libphp5 /app/apache/conf/httpd.conf
LoadModule php5_module        modules/libphp5.so
$ ll /app/apache/modules/
总用量 29620
-rw-r--r--. 1 root root     94月   8 03:02 httpd.exp
-rwxr-xr-x. 1 root root 303164月  21 01:46 libphp5.so
```
修改311行
```bash
AddType application/x-httpd-php .php .phtml
AddType application/x-httpd-php-source .phps
```
更改daemon，更改用户是为了安全考虑

![image-20221025225445542](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-2022102522544554png)

打不开解决方法：

```bash
$ lsof -i:80
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd    5  root    4u  IPv6  69      TCP *:http (LISTEN)
httpd   60apache    4u  IPv6  69      TCP *:http (LISTEN)
```

```bash
$ /app/apache/bin/apachectl restart
```

![image-20221025225459373](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025225459373.png)
