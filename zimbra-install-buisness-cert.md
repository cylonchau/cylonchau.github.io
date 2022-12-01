# zimbra安装三方颁发的证书


## 步骤1：取得SSL凭证

证书需要取的从根证书每一级的证书

## 步骤2：合成SSL证书

将中级、根证书合成为一个证书

顺序：按照从后到前合成为一个证书  如，三级 ==》二级 ==》 根

合成后的格式如下
```

-----BEGIN CERTIFICATE----- 
你的crt內容
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE----- 
MIIDSjCCAjKgAwIBAgIQRK+wgNajJ7qJMDmGLvhAazANBgkqhkiG9w0BAQUFADA/
MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
DkRTVCBSb290IENBIFgzMB4XDTAwMDkzMDIxMTIxOVoXDTIxMDkzMDE0MDExNVow
PzEkMCIGA1UEChMbRGlnaXRhbCBTaWduYXR1cmUgVHJ1c3QgQ28uMRcwFQYDVQQD
Ew5EU1QgUm9vdCBDQSBYMzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
AN+v6ZdQCINXtMxiZfaQguzH0yxrMMpb7NnDfcdAwRgUi+DoM3ZJKuM/IUmTrE4O
rz5Iy2Xu/NMhD2XSKtkyj4zl93ewEnu1lcCJo6m67XMuegwGMoOifooUMM0RoOEq
OLl5CjH9UL2AZd+3UWODyOKIYepLYYHsUmu5ouJLGiifSKOeDNoJjj4XLh7dIN9b
xiqKqy69cK3FCxolkHRyxXtqqzTWMIn/5WgTe1QLyNau7Fqckh49ZLOMxt+/yUFw
7BZy1SbsOFU5Q9D8/RhcQPGX69Wam40dutolucbY38EVAjqr2m7xPi71XAicPNaD
aeQQmxkqtilX4+U9m5/wAl0CAwEAAaNCMEAwDwYDVR0TAQH/BAUwAwEB/zAOBgNV
HQ8BAf8EBAMCAQYwHQYDVR0OBBYEFMSnsaR7LHH62+FLkHX/xBVghYkQMA0GCSqG
SIb3DQEBBQUAA4IBAQCjGiybFwBcqR7uKGY3Or+Dxz9LwwmglSBd49lZRNI+DT69
ikugdB/OEIKcdBodfpga3csTS7MgROSR6cz8faXbauX+5v3gTt23ADq1cEmv8uXr
AvHRAosZy5Q6XkjEGB5YGV8eAlrwDPGxrancWYaLbumR9YbK+rlmM6pZW87ipxZz
R8srzJmwN0jP41ZL9c8PDHIyh8bwRLtTcm1D9SZImlJnt1ir/md2cXjbDaJWFBM5
JDGFoqgCWjBH4d1QB7wCCZAA62RjYJsWvIjJEubSfZGL+T0yjWW06XyxV3bqxbYo
Ob8VZRzI9neWagqNdwvYkQsEjgfbKbYK7p2CNTUQ
-----END CERTIFICATE-----
```

## 步骤3：验证你的商业证书

复制生成的所有证书到目录 `/opt/zimbra/ssl/zimbra/commercial` 下，（合成后的根证书、证书、与秘钥）

切換到 zimbra 用戶

```
$ zmcertmgr verifycrt comm  {{privkey.key}} {{cert.crt}} {{ca.crt}}


$ zmcertmgr verifycrt  comm commercial.key commercial.crt  commercial_ca.crt
** Verifying 'commercial.crt' against 'commercial.key'
Certificate 'commercial.crt' and private key 'commercial.key' match.
** Verifying 'commercial.crt' against 'commercial_ca.crt'
Valid certificate chain: commercial.crt: OK

```

## 步骤4：部署证书

**注意：以下所有命令应以zimbra用户**


```
$ /opt/zimbra/bin/zmcertmgr deploycrt comm {{cert.crt}} {{ca.crt}}
```


```
$ zmcertmgr deploycrt  comm commercial.crt  commercial_ca.crt


** Fixing newlines in 'commercial.crt'
** Fixing newlines in 'commercial_ca.crt'
** Verifying 'commercial.crt' against '/opt/zimbra/ssl/zimbra/commercial/commercial.key'
Certificate 'commercial.crt' and private key '/opt/zimbra/ssl/zimbra/commercial/commercial.key' match.
** Verifying 'commercial.crt' against 'commercial_ca.crt'
Valid certificate chain: commercial.crt: OK
** Copying 'commercial.crt' to '/opt/zimbra/ssl/zimbra/commercial/commercial.crt'
'commercial.crt' and '/opt/zimbra/ssl/zimbra/commercial/commercial.crt' are identical (not copied) at /opt/zimbra/bin/zmcertmgr line 1278.
** Copying 'commercial_ca.crt' to '/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt'
'commercial_ca.crt' and '/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt' are identical (not copied) at /opt/zimbra/bin/zmcertmgr line 1278.
** Appending ca chain 'commercial_ca.crt' to '/opt/zimbra/ssl/zimbra/commercial/commercial.crt'
** Importing cert '/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt' as 'zcs-user-commercial_ca' into cacerts '/opt/zimbra/common/lib/jvm/java/lib/security/cacerts'
** NOTE: restart mailboxd to use the imported certificate.
** Saving config key 'zimbraSSLCertificate' via zmprov modifyServer mail.com...ok
** Saving config key 'zimbraSSLPrivateKey' via zmprov modifyServer mail.com...ok
** Installing imapd certificate '/opt/zimbra/conf/imapd.crt' and key '/opt/zimbra/conf/imapd.key'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.crt' to '/opt/zimbra/conf/imapd.crt'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.key' to '/opt/zimbra/conf/imapd.key'
** Creating file '/opt/zimbra/ssl/zimbra/jetty.pkcs12'
** Creating keystore '/opt/zimbra/conf/imapd.keystore'
** Installing ldap certificate '/opt/zimbra/conf/slapd.crt' and key '/opt/zimbra/conf/slapd.key'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.crt' to '/opt/zimbra/conf/slapd.crt'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.key' to '/opt/zimbra/conf/slapd.key'
** Creating file '/opt/zimbra/ssl/zimbra/jetty.pkcs12'
** Creating keystore '/opt/zimbra/mailboxd/etc/keystore'
** Installing mta certificate '/opt/zimbra/conf/smtpd.crt' and key '/opt/zimbra/conf/smtpd.key'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.crt' to '/opt/zimbra/conf/smtpd.crt'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.key' to '/opt/zimbra/conf/smtpd.key'
** Installing proxy certificate '/opt/zimbra/conf/nginx.crt' and key '/opt/zimbra/conf/nginx.key'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.crt' to '/opt/zimbra/conf/nginx.crt'
** Copying '/opt/zimbra/ssl/zimbra/commercial/commercial.key' to '/opt/zimbra/conf/nginx.key'
** NOTE: restart services to use the new certificates.
** Cleaning up 3 files from '/opt/zimbra/conf/ca'
** Removing /opt/zimbra/conf/ca/629b96f7.0
** Removing /opt/zimbra/conf/ca/ca.key
** Removing /opt/zimbra/conf/ca/ca.pem
** Copying CA to /opt/zimbra/conf/ca
** Copying '/opt/zimbra/ssl/zimbra/ca/ca.key' to '/opt/zimbra/conf/ca/ca.key'
** Copying '/opt/zimbra/ssl/zimbra/ca/ca.pem' to '/opt/zimbra/conf/ca/ca.pem'
** Creating CA hash symlink '629b96f7.0' -> 'ca.pem'
** Creating /opt/zimbra/conf/ca/commercial_ca_1.crt
** Creating CA hash symlink '65ff7287.0' -> 'commercial_ca_1.crt'
** Creating /opt/zimbra/conf/ca/commercial_ca_2.crt
** Creating CA hash symlink 'fc5a8f99.0' -> 'commercial_ca_2.crt'
** Creating /opt/zimbra/conf/ca/commercial_ca_3.crt
** Creating CA hash symlink '157753a5.0' -> 'commercial_ca_3.crt'
```

4. 重启zimbra服务

```
zmcontrol restart
```

瀏覽器訪問地址

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801141804014-521206040.png)
