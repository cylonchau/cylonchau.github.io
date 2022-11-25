# zimbra用户管理员脚本




## 启动故障：`zimbra postsuper: fatal: scan_dir_push: open directory defer: Permission denied`

```bash
Host mail.domain.com
        Starting ldap...Done.
        Starting zmconfigd...Done.
        Starting dnscache...Done.
        Starting logger...Done.
        Starting mailbox...Done.
        Starting memcached...Done.
        Starting proxy...Done.
        Starting amavis...Done.
        Starting antispam...Done.
        Starting antivirus...Done.
        Starting opendkim...Done.
        Starting snmp...Done.
        Starting spell...Done.
        Starting mta...Failed.
Starting saslauthd...done.
postsuper: fatal: scan_dir_push: open directory defer: Permission denied
postfix failed to start
        Starting stats...Done.
        Starting service webapp...Done.
        Starting zimbra webapp...Done.
        Starting zimbraAdmin webapp...Done.
        Starting zimlet webapp...Done.
```

查看服务器状态：
```bash
mta   Stopped
postfix is not running
```

经查看mta服务是由postfix启动。

- 查看系统是否已经对自带的sendmail和postfix进行关闭，端口25是否被占用，如果是请关闭并重启zimbra

- 如果不是则执行/opt/zimbra/libexec/zmfixperms (run as root)

> Refer：[Zimbra 启动时mta无法启动  postsuper: fatal: scan_dir_push: open directory defer: Permission denied](https://www.chenxie.net/archives/635.html)

## 错误：

```bash
opendkim: /opt/zimbra/conf/opendkim.conf: ldap://xxxx333.com:389/?DKIMSelector?sub?(DKIMIdentity=$d): dkimf_db_open(): Connect error
Failed to start opendkim: 0
```

原因：无法连接至ldap服务，检查ldap服务是否正常

> Refer：[ZCS 8.0 mta error with zmopendkimctl error ](https://forums.zimbra.org/viewtopic.php?t=13946)

## 错误：Error: Queue report unavailable

```
zmcontrol status
Host mail.ttdconline.com
amavis                  Running
antispam                Running
antivirus               Running
ldap                    Running
logger                  Running
mailbox                 Running
memcached               Running
mta                     Running
opendkim                Running
proxy                   Running
service webapp          Running
snmp                    Running
spell                   Running
stats                   Running
zimbra webapp           Running
zimbraAdmin webapp      Running
zimlet webapp           Running
zmconfigd               Running

We reviewed logs and services and we see that the MTA is down:

$ tail -f /var/log/mail.log
Jan 22 11:08:00 zcs postfix/postqueue[19195]: fatal: Queue report unavailable – mail system is down
```

> Refer: [Error: Queue report unavailable – mail system is down					](https://dilliganesh.wordpress.com/2016/10/12/error-queue-report-unavailable-mail-system-is-down/)

## 错误 Logswatch Failed

zimbra logswatch failed

. /opt/zimbra/.bashrc

```
Starting logger...Failed.
[b]Starting logswatch...failed.[/b]
```

> Refer: [8.8.15 Starting Logswatch Failed](https://forums.zimbra.org/viewtopic.php?f=13&t=66900)





## 修改管理员账户密码



server.domain.com (`https://server.domain.com:7071`) 是当前运行的zimbra的域名或者IP地址，默认的http监听端口为7071
 输入用户名： [admin@domain.com](https://link.jianshu.com?t=mailto:admin@domain.com) 和密码，完成登录

在zimbra安装配置时创建了管理员账户，可以在web端的账户工具栏任何时候进行账户密码修改，选择administrator 用户并选择密码修改
 也可以在命令行中运行zmprov进行管理员账户密码的修改：

```bash
zmprov sp admin@domain.com <password>
zmprov gaaa //列出所有管理员
zmprov sp admin q1w2e3r4 或 zmprov sp admin@wish.com q12e3r4  # 修改管理员账号密码
```

## 清除队列

查看发送队列数量:

```
/opt/zimbra/libexec/zmqstat
```

查看队列内容

```
mailq
```

删除队列

```
/opt/zimbra/postfix/sbin/postqueue -f
```

查看邮件队列

```
/opt/zimbra/postfix/sbin/postcat -qv EC753D0D00
```

> Refer：
>
> [Managing The Postfix Queues](https://wiki.zimbra.com/wiki/Managing-The-Postfix-Queues)
>
> [Zimbra – deleting all email in queue by sender](Zimbra – deleting all email in queue by sender)

## 被攻击状态

查看邮件状态

```
$ /opt/zimbra/libexec/zmqstat
hold=0
corrupt=0
deferred=563344
active=19992
incoming=45830
```

```
postmap /opt/zimbra/conf/restricted_senders
postmap /opt/zimbra/conf/local_domains 
postmap ../common/conf/main.cf
```

问题

```
Feb 23 00:36:56 ${domainname} postfix/postscreen[7614]: PASS OLD [193.26.3.10]:63396
Feb 23 00:36:56 ${domainname} postfix/smtpd[7615]: connect from mail.health.kiev.ua[193.26.3.10]
Feb 23 00:36:57 ${domainname} postfix/smtpd[7615]: Anonymous TLS connection established from mail.health.kiev.ua[193.26.3.10]: TLSv1 with cipher DHE-RSA-AES256-SHA (256/256 bits)
Feb 23 00:36:58 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:36:58 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:36:58 ${domainname} postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@${domainname}.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:36:59 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:36:59 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:36:59 ${domainname} postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@${domainname}.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:00 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:00 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:37:00 ${domainname} postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@${domainname}.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:00 ${domainname} postfix/postqueue[13129]: fatal: Queue report unavailable - mail system is down
Feb 23 00:37:01 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:01 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:37:01 ${domainname} postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@${domainname}.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:01 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:01 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:37:01 ${domainname} postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@${domainname}.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:05 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:05 ${domainname} postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
```

解决：找其他服务器处理这个问题并修改配置

```
local_only = check_recipient_access lmdb:/opt/zimbra/conf/local_domains, reject
```

正常

```
Feb 23 00:49:31 ${domainname} postfix/smtpd[24306]: connect from iZj6c4jc5vsy383um5cgomZ[172.31.108.227]
Feb 23 00:49:31 ${domainname} postfix/smtpd[24306]: NOQUEUE: reject: RCPT from iZj6c4jc5vsy383um5cgomZ[172.31.108.227]: 554 5.7.1 <test1@${domainname}.com>: Sender address rejected: Access denied; from=<test1@${domainname}.com> to=<test@163.com> proto=ESMTP helo=<${domainname}.com>
Feb 23 00:49:31 ${domainname} postfix/smtpd[24306]: disconnect from iZj6c4jc5vsy383um5cgomZ[172.31.108.227] ehlo=1 mail=1 rcpt=0/1 quit=1 commands=3/4
Feb 23 00:49:31 ${domainname} zmconfigd[17300]: Tracking service snmp
Feb 23 00:49:32 ${domainname} zmconfigd[17300]: Watchdog: service antivirus status is OK.
Feb 23 00:49:32 ${domainname} zmconfigd[17300]: All rewrite threads completed in 0.00 se
```

配置安全策略

> [Zimbra 8.7.11规则：只能发送内部邮件](https://blog.csdn.net/qq_38209584/article/details/73867569)
>
> [Domain level blocking of users](https://wiki.zimbra.com/wiki/Domain_level_blocking_of_users#ZCS_8.7_and_later)
>
> [Rejecting Emails at SMTP Level](https://wiki.zimbra.com/wiki/Rejecting_Emails_at_SMTP_Level)

配置监控策略

> [配置磁盘空间的告警](https://zimbra.github.io/adminguide/latest/#_configuring_disk_space_notifications)


