# zimbra启用SMTP认证


```
zmprov modifyServer {{ you domain }} zimbraMtaTlsAuthOnly FALSE
zmcontrol  restart 
```

查看对应配置

```
zmprov getServer {{ you domain }} |  grep  Auth
```

 查看SMTP是否开启成功

```
$ telnet localhost 25
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 xxxx ESMTP Postfix
ehlo xxxx      
250-xxxxx
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH LOGIN PLAIN      #SMTP认证相关参数
250-AUTH=LOGIN PLAIN     #SMTP认证相关参数
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
```
