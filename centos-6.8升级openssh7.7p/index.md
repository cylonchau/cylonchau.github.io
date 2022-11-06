# CentOS 6.8升级OpenSSH7.7p




近期因centos 6.x 默认openssh扫描存在大量漏洞，基于安全考虑，需要将openssh_5.3p1升级为最新版，网上查了很多教程，发现openssh存在大量依赖，不解决依赖问题很难保证其他服务政策。而openssl又被大量程序依赖。实在是头疼。最后发现一个不破坏各种依赖又可以完美升级的方案

注：`curl` `wget` `yum`等依赖openssl   gitlab依赖openssh因卸载openssh与openssl编译安装导致各种依赖程序被破坏，虽然最后升级成功，但是wget curl 和代码库被破坏。

## 下载openssh7.7p源码包

http://www.openssh.com/portable.html

下载之后解压看readme和install

```
1. Prerequisites
----------------

A C compiler.  Any C89 or better compiler should work.  Where supported,
configure will attempt to enable the compiler&#39;s run-time integrity checking
options.  Some notes about specific compilers:
 - clang: -ftrapv and -sanitize=integer require the compiler-rt runtime
  (CC=clang LDFLAGS=--rtlib=compiler-rt ./configure)

You will need working installations of Zlib and libcrypto (LibreSSL /
OpenSSL)

Zlib 1.1.4 or 1.2.1.2 or greater (earlier 1.2.x versions have problems):
http://www.gzip.org/zlib/

libcrypto (LibreSSL or OpenSSL &gt;= 1.0.1 &lt; 1.1.0)
LibreSSL http://www.libressl.org/ ; or
OpenSSL http://www.openssl.org/

LibreSSL/OpenSSL should be compiled as a position-independent library
(i.e. with -fPIC) otherwise OpenSSH will not be able to link with it.
If you must use a non-position-independent libcrypto, then you may need
to configure OpenSSH --without-pie.  Note that because of API changes,
OpenSSL 1.1.x is not currently supported.

The remaining items are optional.
```

官方给出的文档中提到的先决条件openssh安装依赖 `zlib1.1.4` 并且 `openssl&gt;=1.0.1` 版本就可以了。那么直接看当前系统的openssl版本是多少

```
$ openssl version
OpenSSL 1.0.1e-fips 11 Feb 2013
$ rpm -q zlib
zlib-1.2.3-29.el6.x86_64
$ rpm -q zlib-devel
zlib-devel-1.2.3-29.el6.x86_64
```

发现自带的openssl版本符合openssh7.7p的安装条件，自带的zlib也符合OpenSSH7.7P的依赖。那么就直接安装吧。

## 打包OpenSSH

```
mkdir -p /usr/src/redhat/{SOURCES,SPECS}
cd /usr/src/redhat/SOURCES/
wget http://ftp.riken.jp/Linux/momonga/6/Everything/SOURCES/x11-ssh-askpass-1.2.4.1.tar.gz
tar xf openssh-7.7p1.tar.gz
cp openssh-7.7p1/contrib/redhat/openssh.spec /usr/src/redhat/SPECS/
chown sshd:sshd /usr/src/redhat/SPECS/ -R
sed -i &#39;s@%define no_gnome_askpass 0@%define no_gnome_askpass 1@g&#39; /usr/src/redhat/SPECS/openssh.spec
sed -i &#39;s@%define no_x11_askpass 0@%define no_x11_askpass 1@g&#39; /usr/src/redhat/SPECS/openssh.spec
cp /usr/src/redhat/SOURCES/openssh-7.7p1.tar.gz ~/rpmbuild/SOURCES/
cd /usr/src/redhat/SPECS/
rpmbuild -ba openssh.spec
```

可以看到rpm包和yum安装的是一样的。

```
├── RPMS
│   └── x86_64
│   ├── openssh-7.7p1-1.el6.x86_64.rpm
│   ├── openssh-clients-7.7p1-1.el6.x86_64.rpm
│   ├── openssh-debuginfo-7.7p1-1.el6.x86_64.rpm
│   └── openssh-server-7.7p1-1.el6.x86_64.rpm

[root@zabbix-serv SPECS]# rpm -qa|grep openssh
openssh-clients-5.3p1-117.el6.x86_64
openssh-5.3p1-117.el6.x86_64
openssh-server-5.3p1-117.el6.x86_64
```

直接替换安装rpm包

```
$ rpm -Uvh *
Preparing...                ########################################### [100%]
   1:openssh                ########################################### [ 25%]
   2:openssh-clients        ########################################### [ 50%]
   3:openssh-server         warning: /etc/ssh/sshd_config created as /etc/ssh/sshd_config.rpmnew
########################################### [ 75%]
   4:openssh-debuginfo      ########################################### [100%]

```

安装后查看各项依赖openssl的匀使用正常。这么安装比编译安装要好很多。

```
$ sshd -V
unknown option -- V
OpenSSH_7.7p1, OpenSSL 1.0.1e-fips 11 Feb 2013
usage: sshd [-46DdeiqTt] [-C connection_spec] [-c host_cert_file]
            [-E log_file] [-f config_file] [-g login_grace_time]
            [-h host_key_file] [-o option] [-p port] [-u len]
$ ssh -V
OpenSSH_7.7p1, OpenSSL 1.0.1e-fips 11 Feb 2013
$ curl baidu.com -I
HTTP/1.1 200 OK
Date: Wed, 25 Apr 2018 16:37:49 GMT
Server: Apache
Last-Modified: Tue, 12 Jan 2010 13:48:00 GMT
ETag: &#34;51-47cf7e6ee8400&#34;
Accept-Ranges: bytes
Content-Length: 81
Cache-Control: max-age=86400
Expires: Thu, 26 Apr 2018 16:37:49 GMT
Connection: Keep-Alive
Content-Type: text/html

$ wget -q baidu.com
$ yum list &gt;&gt;/dev/null
```

测试yum安装，依赖openssh的是否会将7.7p替换为5.3p

```
$ yum install openssh*
Loaded plugins: fastestmirror, security
Setting up Install Process
Examining openssh-7.7p1-1.el6.x86_64.rpm: openssh-7.7p1-1.el6.x86_64
openssh-7.7p1-1.el6.x86_64.rpm: does not update installed package.
Examining openssh-clients-7.7p1-1.el6.x86_64.rpm: openssh-clients-7.7p1-1.el6.x86_64
openssh-clients-7.7p1-1.el6.x86_64.rpm: does not update installed package.
Examining openssh-debuginfo-7.7p1-1.el6.x86_64.rpm: openssh-debuginfo-7.7p1-1.el6.x86_64
openssh-debuginfo-7.7p1-1.el6.x86_64.rpm: does not update installed package.
Examining openssh-server-7.7p1-1.el6.x86_64.rpm: openssh-server-7.7p1-1.el6.x86_64
openssh-server-7.7p1-1.el6.x86_64.rpm: does not update installed package.
Error: Nothing to do
```
