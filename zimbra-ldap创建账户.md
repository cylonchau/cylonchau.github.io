# 

```ldif
cat << EOF | ldapadd -x -W -H ldap://172.31.110.115:389 -D "uid=zimbra,cn=admins,cn=zimbra"
dn: uid=scott8,ou=people,dc=mail,dc=ironhide2021,dc=com
zimbraAccountStatus: active
displayName: scott8
givenName: scott8
sn: scott8
zimbraMailStatus: enabled
objectClass: inetOrgPerson
objectClass: zimbraAccount
objectClass: amavisAccount
zimbraId: e2214a66-3ga2-4241-9223-44f222ce0522
zimbraCreateTimestamp: 20191102062818.876Z
zimbraMailHost: mail.ironhide2021.com
zimbraMailTransport: lmtp:mail.ironhide2021.com:7025
zimbraMailDeliveryAddress: scott8@mail.ironhide2021.com
mail: scott8@mail.ironhide2021.com
cn: scott8
uid: scott8
userPassword:: e1NTSEE1MTJ9ZzBzZGlXRlBjbDQxa2xmZ200YXc1ZkJzSGQzVXNBdVBydUlKRnZ
 LTExYby9HWXBoUkNTMzZYMEx5VnpCZUJPMGJNTCtTV2IwSnhkaHdudTViR0c1bTJabFVhU3R1N1J3
EOF
99tJFkhVfn
```



```bash
ldapsearch -LLL -w 99tJFkhVfn -H ldap://172.31.110.115:389 -D "uid=zimbra,cn=admins,cn=zimbra"|less
```



zimbra postsuper: fatal: scan_dir_push: open directory defer: Permission denied

https://www.chenxie.net/archives/635.html



opendkim: /opt/zimbra/conf/opendkim.conf: ldap://itcom333.com:389/?DKIMSelector?sub?(DKIMIdentity=$d): dkimf_db_open(): Connect error
Failed to start opendkim: 0

https://forums.zimbra.org/viewtopic.php?t=13946



zimbra logswatch failed

. /opt/zimbra/.bashrc



```bash
	https://server.domain.com:7071
```

server.domain.com 是当前运行的zimbra的域名或者IP地址，默认的http监听端口为7071
 输入用户名： [admin@domain.com](https://link.jianshu.com?t=mailto:admin@domain.com) 和密码，完成登录

#### 修改管理员账户密码：

在zimbra安装配置时创建了管理员账户，可以在web端的账户工具栏任何时候进行账户密码修改，选择administrator 用户并选择密码修改
 也可以在命令行中运行zmprov进行管理员账户密码的修改：



```bash
zmprov sp admin@domain.com <password>
zmprov gaaa //列出所有管理员
zmprov sp admin q1w2e3r4 或 zmprov sp admin@wish.com q12e3r4  # 修改管理员账号密码
```



查看发送队列数量:

```
/opt/zimbra/libexec/zmqstat
```

查看队列内容

```
mailq
```

删除队列

https://wiki.zimbra.com/wiki/Managing-The-Postfix-Queues



```
[root@itcom333 ~]#  /opt/zimbra/libexec/zmqstat
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
Feb 23 00:36:56 itcom333 postfix/postscreen[7614]: PASS OLD [193.26.3.10]:63396
Feb 23 00:36:56 itcom333 postfix/smtpd[7615]: connect from mail.health.kiev.ua[193.26.3.10]
Feb 23 00:36:57 itcom333 postfix/smtpd[7615]: Anonymous TLS connection established from mail.health.kiev.ua[193.26.3.10]: TLSv1 with cipher DHE-RSA-AES256-SHA (256/256 bits)
Feb 23 00:36:58 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:36:58 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:36:58 itcom333 postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@itcom333.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:36:59 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:36:59 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:36:59 itcom333 postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@itcom333.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:00 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:00 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:37:00 itcom333 postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@itcom333.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:00 itcom333 postfix/postqueue[13129]: fatal: Queue report unavailable - mail system is down
Feb 23 00:37:01 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:01 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:37:01 itcom333 postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@itcom333.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:01 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:01 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
Feb 23 00:37:01 itcom333 postfix/smtpd[7615]: NOQUEUE: reject: RCPT from mail.health.kiev.ua[193.26.3.10]: 451 4.3.5 <>: Sender address rejected: Server configuration error; from=<> to=<vivian@itcom333.com> proto=ESMTP helo=<mail.health.kiev.ua>
Feb 23 00:37:05 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders is unavailable. open database /opt/zimbra/conf/restricted_senders.lmdb: MDB_INVALID: File is not an LMDB file
Feb 23 00:37:05 itcom333 postfix/smtpd[7615]: warning: lmdb:/opt/zimbra/conf/restricted_senders: table lookup problem
```

解决：找其他服务器处理这个问题并修改配置

```
local_only = check_recipient_access lmdb:/opt/zimbra/conf/local_domains, reject
```

正常

```
Feb 23 00:49:31 itcom333 postfix/smtpd[24306]: connect from iZj6c4jc5vsy383um5cgomZ[172.31.108.227]
Feb 23 00:49:31 itcom333 postfix/smtpd[24306]: NOQUEUE: reject: RCPT from iZj6c4jc5vsy383um5cgomZ[172.31.108.227]: 554 5.7.1 <test1@itcom333.com>: Sender address rejected: Access denied; from=<test1@itcom333.com> to=<test@163.com> proto=ESMTP helo=<itcom333.com>
Feb 23 00:49:31 itcom333 postfix/smtpd[24306]: disconnect from iZj6c4jc5vsy383um5cgomZ[172.31.108.227] ehlo=1 mail=1 rcpt=0/1 quit=1 commands=3/4
Feb 23 00:49:31 itcom333 zmconfigd[17300]: Tracking service snmp
Feb 23 00:49:32 itcom333 zmconfigd[17300]: Watchdog: service antivirus status is OK.
Feb 23 00:49:32 itcom333 zmconfigd[17300]: All rewrite threads completed in 0.00 se
```

http://www.yanglajiao.com/article/qq_38209584/73867569

https://blog.csdn.net/qq_38209584/article/details/73867569



白名单

https://wiki.zimbra.com/wiki/Domain_level_blocking_of_users#ZCS_8.7_and_later

https://wiki.zimbra.com/wiki/Rejecting_Emails_at_SMTP_Level

队列

https://dilliganesh.wordpress.com/2016/10/12/error-queue-report-unavailable-mail-system-is-down/

https://tweenpath.net/zimbra-deleting-email-queue-sender/



手册

https://zimbra.github.io/adminguide/latest/#_configuring_disk_space_notifications
