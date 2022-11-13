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
MIIDSjCCAjKgAwIBAgIQRK&#43;wgNajJ7qJMDmGLvhAazANBgkqhkiG9w0BAQUFADA/
MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
DkRTVCBSb290IENBIFgzMB4XDTAwMDkzMDIxMTIxOVoXDTIxMDkzMDE0MDExNVow
PzEkMCIGA1UEChMbRGlnaXRhbCBTaWduYXR1cmUgVHJ1c3QgQ28uMRcwFQYDVQQD
Ew5EU1QgUm9vdCBDQSBYMzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
AN&#43;v6ZdQCINXtMxiZfaQguzH0yxrMMpb7NnDfcdAwRgUi&#43;DoM3ZJKuM/IUmTrE4O
rz5Iy2Xu/NMhD2XSKtkyj4zl93ewEnu1lcCJo6m67XMuegwGMoOifooUMM0RoOEq
OLl5CjH9UL2AZd&#43;3UWODyOKIYepLYYHsUmu5ouJLGiifSKOeDNoJjj4XLh7dIN9b
xiqKqy69cK3FCxolkHRyxXtqqzTWMIn/5WgTe1QLyNau7Fqckh49ZLOMxt&#43;/yUFw
7BZy1SbsOFU5Q9D8/RhcQPGX69Wam40dutolucbY38EVAjqr2m7xPi71XAicPNaD
aeQQmxkqtilX4&#43;U9m5/wAl0CAwEAAaNCMEAwDwYDVR0TAQH/BAUwAwEB/zAOBgNV
HQ8BAf8EBAMCAQYwHQYDVR0OBBYEFMSnsaR7LHH62&#43;FLkHX/xBVghYkQMA0GCSqG
SIb3DQEBBQUAA4IBAQCjGiybFwBcqR7uKGY3Or&#43;Dxz9LwwmglSBd49lZRNI&#43;DT69
ikugdB/OEIKcdBodfpga3csTS7MgROSR6cz8faXbauX&#43;5v3gTt23ADq1cEmv8uXr
AvHRAosZy5Q6XkjEGB5YGV8eAlrwDPGxrancWYaLbumR9YbK&#43;rlmM6pZW87ipxZz
R8srzJmwN0jP41ZL9c8PDHIyh8bwRLtTcm1D9SZImlJnt1ir/md2cXjbDaJWFBM5
JDGFoqgCWjBH4d1QB7wCCZAA62RjYJsWvIjJEubSfZGL&#43;T0yjWW06XyxV3bqxbYo
Ob8VZRzI9neWagqNdwvYkQsEjgfbKbYK7p2CNTUQ
-----END CERTIFICATE-----
```


### 步骤3：验证你的商业证书

复制生成的所有证书到目录 `/opt/zimbra/ssl/zimbra/commercial` 下，（合成后的根证书、证书、与秘钥）

切換到 zimbra 用戶

```
zmcertmgr verifycrt comm  {{privkey.key}} {{cert.crt}} {{ca.crt}}


[zimbra@xxxxx commercial]$ zmcertmgr verifycrt  comm commercial.key commercial.crt  commercial_ca.crt
** Verifying &#39;commercial.crt&#39; against &#39;commercial.key&#39;
Certificate &#39;commercial.crt&#39; and private key &#39;commercial.key&#39; match.
** Verifying &#39;commercial.crt&#39; against &#39;commercial_ca.crt&#39;
Valid certificate chain: commercial.crt: OK

```

## 步骤4：部署证书

**注意：以下所有命令应以zimbra用户**


```
/opt/zimbra/bin/zmcertmgr deploycrt comm {{cert.crt}} {{ca.crt}}
```


```
[zimbra@xxxxx commercial]$ zmcertmgr deploycrt  comm commercial.crt  commercial_ca.crt


** Fixing newlines in &#39;commercial.crt&#39;
** Fixing newlines in &#39;commercial_ca.crt&#39;
** Verifying &#39;commercial.crt&#39; against &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.key&#39;
Certificate &#39;commercial.crt&#39; and private key &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.key&#39; match.
** Verifying &#39;commercial.crt&#39; against &#39;commercial_ca.crt&#39;
Valid certificate chain: commercial.crt: OK
** Copying &#39;commercial.crt&#39; to &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39;
&#39;commercial.crt&#39; and &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39; are identical (not copied) at /opt/zimbra/bin/zmcertmgr line 1278.
** Copying &#39;commercial_ca.crt&#39; to &#39;/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt&#39;
&#39;commercial_ca.crt&#39; and &#39;/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt&#39; are identical (not copied) at /opt/zimbra/bin/zmcertmgr line 1278.
** Appending ca chain &#39;commercial_ca.crt&#39; to &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39;
** Importing cert &#39;/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt&#39; as &#39;zcs-user-commercial_ca&#39; into cacerts &#39;/opt/zimbra/common/lib/jvm/java/lib/security/cacerts&#39;
** NOTE: restart mailboxd to use the imported certificate.
** Saving config key &#39;zimbraSSLCertificate&#39; via zmprov modifyServer mail.com...ok
** Saving config key &#39;zimbraSSLPrivateKey&#39; via zmprov modifyServer mail.com...ok
** Installing imapd certificate &#39;/opt/zimbra/conf/imapd.crt&#39; and key &#39;/opt/zimbra/conf/imapd.key&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39; to &#39;/opt/zimbra/conf/imapd.crt&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.key&#39; to &#39;/opt/zimbra/conf/imapd.key&#39;
** Creating file &#39;/opt/zimbra/ssl/zimbra/jetty.pkcs12&#39;
** Creating keystore &#39;/opt/zimbra/conf/imapd.keystore&#39;
** Installing ldap certificate &#39;/opt/zimbra/conf/slapd.crt&#39; and key &#39;/opt/zimbra/conf/slapd.key&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39; to &#39;/opt/zimbra/conf/slapd.crt&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.key&#39; to &#39;/opt/zimbra/conf/slapd.key&#39;
** Creating file &#39;/opt/zimbra/ssl/zimbra/jetty.pkcs12&#39;
** Creating keystore &#39;/opt/zimbra/mailboxd/etc/keystore&#39;
** Installing mta certificate &#39;/opt/zimbra/conf/smtpd.crt&#39; and key &#39;/opt/zimbra/conf/smtpd.key&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39; to &#39;/opt/zimbra/conf/smtpd.crt&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.key&#39; to &#39;/opt/zimbra/conf/smtpd.key&#39;
** Installing proxy certificate &#39;/opt/zimbra/conf/nginx.crt&#39; and key &#39;/opt/zimbra/conf/nginx.key&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.crt&#39; to &#39;/opt/zimbra/conf/nginx.crt&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/commercial/commercial.key&#39; to &#39;/opt/zimbra/conf/nginx.key&#39;
** NOTE: restart services to use the new certificates.
** Cleaning up 3 files from &#39;/opt/zimbra/conf/ca&#39;
** Removing /opt/zimbra/conf/ca/629b96f7.0
** Removing /opt/zimbra/conf/ca/ca.key
** Removing /opt/zimbra/conf/ca/ca.pem
** Copying CA to /opt/zimbra/conf/ca
** Copying &#39;/opt/zimbra/ssl/zimbra/ca/ca.key&#39; to &#39;/opt/zimbra/conf/ca/ca.key&#39;
** Copying &#39;/opt/zimbra/ssl/zimbra/ca/ca.pem&#39; to &#39;/opt/zimbra/conf/ca/ca.pem&#39;
** Creating CA hash symlink &#39;629b96f7.0&#39; -&gt; &#39;ca.pem&#39;
** Creating /opt/zimbra/conf/ca/commercial_ca_1.crt
** Creating CA hash symlink &#39;65ff7287.0&#39; -&gt; &#39;commercial_ca_1.crt&#39;
** Creating /opt/zimbra/conf/ca/commercial_ca_2.crt
** Creating CA hash symlink &#39;fc5a8f99.0&#39; -&gt; &#39;commercial_ca_2.crt&#39;
** Creating /opt/zimbra/conf/ca/commercial_ca_3.crt
** Creating CA hash symlink &#39;157753a5.0&#39; -&gt; &#39;commercial_ca_3.crt&#39;
```

4. 重启zimbra服务

```
zmcontrol restart
```

瀏覽器訪問地址。

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801141804014-521206040.png)
