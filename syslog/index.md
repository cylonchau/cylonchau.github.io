# syslog

[TOC]

## 1 rsyslog介绍
syslog守护进程，内部有两个进程，syslogd主要负责用户空间的用户进程记录日志；klog负责内核所发生的各种时间记录日志。两者合并后形成syslog。

rsyslog是syslog下一代升级产品，依然有syslogd klogd提供服务。

rsyslog可以开通远程机制监听在某个套接字上，其他任何主机所产生的日志信息由本机的rsyslog收集起来，收集完后不负责记录，而是建立一个tcp或udp连接发送给专门的日志服务器，由专门的日志服务器负责记录。默认情况下是明文的。
### 1.1 rsyslog特性
1. 多线程。
2. 支持UDP,TCP协议，基于ssl tls加密完成远程日志传输。支持RELP协议
3. 实现将日志存储到MySQL PGSQL等关系型数据库中。
4. 强大的过滤器，可实现过滤日志信息中任何部分，支持自定义输出格式。

## 2 日志格式
事件产生的事件  主机  进程pid  事件

```bash
Jun  6 23:36:58 Lamp-02 NET[1838]: /etc/sysconfig/network-scripts/ifup-post: updated /etc/resolv.conf
Jun  6 23:46:15 Lamp-02 yum[1963]: Updated: mysql-libs-5.1.73-8.el6_8.x86_64
Jun  6 23:46:16 Lamp-02 yum[1963]: Installed: mysql-5.1.73-8.el6_8.x86_64
```

有些日志记录二进制日志/var/log/wtmp /var/log/btmp

last：`/var/log/wtmp` 当前系统中成功登陆的日志
lastb：`/var/log/btmp` 当前系统中失败的登陆尝试
lastlog：显示当前系统每一个用户最近一次登陆时间

### 2.1 日志等级
日志级别：事件的关键性程度

lev	|说明

| lev          | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| none         | 不记录                                                       |
| debug        | 调试信息                                                     |
| info         | 正常信息，仅是一些基本信息说明                               |
| notice       | 比info还需要注意的一些信息内容                               |
| warning,warn | 警告信息，可能有些问题，但是还不至于影响到某个服务运作的信息 |
| err,error    | 一些重大的错误信息                                           |
| crit         | 临界状态，比error还要严重的错误信息，橙色警报                |
| alert        | 红色警报，已经很有问题的等级，比crit还要严重                 |
| emerg,panic  | 疼痛等级，意指系统已经要宕机的状态！很严重的错误信息         |

### 2.2 设施类型
facility：把某一类具有相同特性的由各个应用程序所产生的日志数据流归类到用一个数据收集管道中，这个收集管道称之为facility。



| 类型           | 说明                                             |
| -------------- | ------------------------------------------------ |
| auth(authpriv) | 与认证有关的机制，例如login ssh su等需要账号密码 |
| cron           | 例行性工作调度cron/at等生成信息日志的地方        |
| daemon         | 与各个daemon有关的信息。                         |
| kern           | 内核产生信息地方                                 |
| lpr            | 打印相关信息                                     |
| mail           | 邮件收发有关的信息                               |
| news           | 新闻组服务器有关信息                             |
| syslog         | 自身产生日志                                     |
| user           | 用户                                             |
| security       | 与安全相关信息                                   |
| local0-local7  | 用户自定义8个设置                                |

### 2.3 通配机制
| 通配符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| .      | 代表【比后面还要高的等级（含该等级）都被记录下来】 例如：mail.info代表只要是mail信息，而且该信息等级高于info（含info）时，就会被记录下来。 |
| .=     | 代表所需要的等级就是后面接的等级而已，其他的不要。例如：main.=info代表的只要是mail信息，而且该信息等于info级别，就会被记录下来。 |
| .!     | 代表不等于（取反），亦是除了该等级外的其他等级都记录。       |
| *      | 所有；例如：*.info代表所有设施的info级别                     |
| none   | 不记录                                                       |

### 2.4 日志的输出位置
| 位置             | 说明                                                  |
| ---------------- | ----------------------------------------------------- |
| 文件             | /var/log/messages                                     |
| 打印机或其他设备 | /dev/lp0这个打印机装置                                |
| 使用者名称       | 显示给用户，*代表目前在线所有的人                     |
| 远程主机         | @10.0.0.2 远程日志服务器，@代表udp协议，@@代表tcp协议 |
| 管道             | \|command                                             |

```bash
$ rpm -ql rsyslog
/etc/logrotate.d/syslog
/etc/pki/rsyslog
/etc/rsyslog.conf &lt;==配置文件
/etc/rsyslog.d
/etc/sysconfig/rsyslog
/usr/bin/rsyslog-recover-qi.pl
/usr/lib/systemd/system/rsyslog.service &lt;==单元文件
/usr/lib64/rsyslog
/usr/lib64/rsyslog/imdiag.so &lt;==收集日志接受日志流输入时的输入过滤工具
/usr/lib64/rsyslog/omjournal.so &lt;==
/var/lib/rsyslog
```

## 3 rsyslog配置文件详解

rsyslog配置文件分为3个模块，&lt;font style=&#34;background:#fee904;&#34; size=3&gt; 每段的配置必须严格写在#### xxx ####位置内&lt;/font&gt;
```bash
$ grep &#39;##&#39; /etc/rsyslog.conf
#### MODULES ####   #&lt;==加载的模块
#### GLOBAL DIRECTIVES ####  #&lt;==定义日志格式默认模板
#### RULES ####  #&lt;==转发规则
# ### begin forwarding rule ###
# ### end of the forwarding rule ###
```

### 3.1 常用参数
```bash
$ModLoad imklog #&lt;==加载模块
mail.*  -/var/log/maillog #&lt;==将mail所有类型的日志异步写入maillog文件中
```

## 4 rsyslog案例
### 4.1 设置ssh日志为其他设施
#### 4.1.1 修改ssh配置文件
```bash
#SyslogFacility AUTHPRIV
SyslogFacility local1
```
#### 4.1.2 修改rsyslog配置文件
将ssh日志写入到ssh.log中
```bash
local1.*    -/var/log/ssh.log
```
使用ssh登陆查看日志生成结果
```bash
$ cat /var/log/ssh.log
Jun  7 00:15:17 Lamp-02 sshd[2966]: Accepted password for root from 192.168.2.1 port 57670 ssh2
```
### 4.2 将日志记录到远程服务器
在客户端配置
```bash
*.info;mail.none;authpriv.none;cron.none;local0.none;          @192.168.2.82
```
服务端开启配置
```bash
$ModLoad imudp
$UDPServerRun 514
```
查看服务端的日志记录情况
```bash
$ cat /var/log/messages
Jun  2 05:47:59 lnmp yum[128171]: Updated: httpd-tools-2.4.6-45.el7.centos.4.x86_64
Jun  2 05:48:03 lnmp yum[128171]: Updated: httpd-2.4.6-45.el7.centos.4.x86_64
Jun  2 05:48:03 lnmp systemd: Reloading.
Jun  2 05:48:04 lnmp systemd: Configuration file /usr/lib/systemd/system/auditd.service is marked world-inaccessible. This has no effect as configuration data is accessible via APIs without restrictions. Proceeding anyway.
```

### 4.3 基于MySQL存储日志
将日志存储到数据库中需要加载相对应的模块
```bash
rsyslog.x86_64                5.8.10-10.el6_6 base
rsyslog-gnutls.x86_64        	5.8.10-10.el6_6 base
rsyslog-gssapi.x86_64        	5.8.10-10.el6_6 base
rsyslog-mysql.x86_64         	5.8.10-10.el6_6 base
rsyslog-pgsql.x86_64          5.8.10-10.el6_6 base
rsyslog-relp.x86_64           5.8.10-10.el6_6 base
rsyslog-snmp.x86_64           5.8.10-10.el6_6 base
...
```

#### 4.3.1 安装rsyslog程序包
```bash
$ rpm -ql rsyslog-mysql
/lib64/rsyslog/ommysql.so
/usr/share/doc/rsyslog-mysql-5.8.10
/usr/share/doc/rsyslog-mysql-5.8.10/createDB.sql
```
#### 4.3.2 配置服务器端参数
```bash
$Modload ommysql
*.info;mail.none;authpriv.none;cron.none :ommysql:localhost,Syslog,syslog,111
```
#### 4.3.3 导入数据库并授权
```sql
mysql &lt; /usr/share/doc/rsyslog-mysql-5.8.10/createDB.sql
grant all privileges on syslog.* to syslog@&#39;localhost&#39; identified by &#39;111&#39;;
```

查看结果
```sql
mysql&gt; select count(*) from systemevents;
&#43;----------&#43;
| count(*) |
&#43;----------&#43;
|        3 |
&#43;----------&#43;
1 row in set (0.00 sec)
```

#### 4.3.4 安装loganalyzer
下载loganalyzer：http://loganalyzer.adiscon.com/downloads/
```bash
sh configure.sh
sh secure.sh
chmod 666 config.php
```
