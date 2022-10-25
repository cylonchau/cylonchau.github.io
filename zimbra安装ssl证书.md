# zimbra安装ssl证书


zimbra在后台安装证书签发机构签发证书出现时候出现错误：{RemoteManager: mail.domain.com->zimbra@mail.domain.com:22}

```
com.zimbra.common.service.ServiceException: system failure: exception during auth {RemoteManager: mail.domain.com->zimbra@mail.domain.com:22}
ExceptionId:qtp1068934215-357:https:https ://mail.domain.com:7071/service/admin/soap/GetMailQueueRequest:
Code:service.FAILURE
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801141357701-503029310.png)

```
com.zimbra.common.service.ServiceException: system failure: exception during auth {RemoteManager: mail.domain.com->zimbra@mail.domain.com:22}
ExceptionId:qtp1068934215-357:https:https ://mail.domain.com:7071/service/admin/soap/GetMailQueueRequest:
Code:service.FAILURE
       at com.zimbra.common.service.ServiceException.FAILURE(ServiceException.java:286)
       at com.zimbra.cs.rmgmt.RemoteManager.getSession(RemoteManager.java:209)
       at com.zimbra.cs.rmgmt.RemoteManager.execute(RemoteManager.java:139)
       at com.zimbra.cert.GetCert.addCertsOnServer(GetCert.java:112)
       at com.zimbra.cert.GetCert.handle(GetCert.java:75)
 
       Caused by: java.io.IOException: There was a problem while connecting to mail.domain.com:22
               at ch.ethz.ssh2.Connection.connect(Connection.java:699)
               at ch.ethz.ssh2.Connection.connect(Connection.java:490)
               at com.zimbra.cs.rmgmt.RemoteManager.getSession(RemoteManager.java:200)
               ... 59 more
```

检查ssh的配置与zimbra的配置

```
$  zmprov gacf | grep Remote
zimbraRemoteImapBindPort: 8143
zimbraRemoteImapSSLBindPort: 8993
zimbraRemoteImapSSLServerEnabled: TRUE
zimbraRemoteImapServerEnabled: TRUE
zimbraRemoteManagementCommand: /opt/zimbra/libexec/zmrcd
zimbraRemoteManagementPort: 22
zimbraRemoteManagementPrivateKeyPath: /opt/zimbra/.ssh/zimbra_identity
zimbraRemoteManagementUser: zimbra
```

看到ssh端口为22，修改为当前系统使用的ssh端口

```
zmprov mcf zimbraRemoteManagementPort {sshport}
```

 再次查看

```
zmprov gacf | grep Remote
zimbraRemoteImapBindPort: 8143
zimbraRemoteImapSSLBindPort: 8993
zimbraRemoteImapSSLServerEnabled: TRUE
zimbraRemoteImapServerEnabled: TRUE
zimbraRemoteManagementCommand: /opt/zimbra/libexec/zmrcd
zimbraRemoteManagementPort: 9955
zimbraRemoteManagementPrivateKeyPath: /opt/zimbra/.ssh/zimbra_identity
zimbraRemoteManagementUser: zimbra
```

修改zimbra用户可以被允许登陆ssh

```
AllowUsers root zimbra
```

此时可以正确提交

> **Reference**
> http://thaiserv.blogspot.com/2015/05/get-message-failure-exception-during.html
> https://wiki.zimbra.com/wiki/RemoteManager_exception
