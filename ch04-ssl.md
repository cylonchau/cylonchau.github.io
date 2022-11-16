# ch4 understanding openldap configuration - TLS/SSL


## OpenLDAP TLS/SSL 配置

对于 TLS/SSL 方向的内容不过多阐述了，这里只阐述openldap TLS/SSL 配置方向的内容

openldap提供了两种方式进行 TLS/SSL 认证

- 自动模式：客户端通过 **ldaps://hostname/** 形式的URL访问slapd，slapd默认为636端口提供 TLS 会话
- 主动定义：slapd标准端口389支持 TLS/SSL ，客户端通过显式配置 TLS/SSL 也可以使用 URL格式**ldap://hostname/** 进行访问，需要注意的是，在同步时如果使用 **ldap://** 格式URL需要指定参数 **starttls=yes** 或者 **starttls=critical** 使用 **ldaps://** 则不需要指定该参数

## 生成自签名证书

创建CA证书 

```bash
openssl genrsa -out cakey.key 2048

openssl req -new -x509 \
	-key cakey.key \
	-out cacert.crt \
	-days 3650 \
	-subj "/C=HK/ST=HK/O=TVB/OU=bigbigchannl/CN=tvb-ca"
	
touch index.txt
echo "01" > serial
```

生成证书请求

```bash
openssl genrsa -out openldap.key 2048

openssl req -new \
	-key openldap.key \
	-out openldap.csr \
	-days 3650 \
	-subj "/C=HK/ST=HK/O=TVB/OU=bigbigchannl/CN=10.0.0.4"

openssl ca \
	-in openldap.csr \
	-cert cacert.crt \
	-keyfile cakey.key  \
	-out openldap.crt \
	-days 3650
```

## 启用openldap TLS认证

slapd TLS 部分配置是在全局部分

### 修改服务端参数

```
# cn=config base（全局部分）

dn: cn=config
changetype: modify
# Security - TLS section
add: olcTLSCertificateFile
# cafile
olcTLSCertificateFile: /certs/ldapscert.pem
-
add: olcTLSCertificateKeyFile
# ca key
olcTLSCertificateKeyFile: /certs/keys/ldapskey.pem
-
# 证书
olcTLSCertificateFile: "/etc/openldap/certs/openldap.crt"
olcTLSCertificateKeyFile: "/etc/openldap/certs/openldap.key"
-
# the following directive is the default but 
# is explicitly included for visibility reasons
add: olcTLSVerifyClient
olcTLSVerifyClient: never
```

### 修改客户端参数

### 命令行指令相关配置

客户端时指 `ladpadd`，`ladpsearch` 相关配置文件，默认为  `/etc/openldap/ldap.conf`

```
URI	ldap://ldap.example.com ldaps://10.0.0.4:636

#SIZELIMIT	12
#TIMELIMIT	15
#DEREF		never

TLS_CACERTDIR	/etc/openldap/certs
TLS_CACERT /etc/openldap/certs/cacert.crt
TLS_CERT /etc/openldap/certs/openldap.csr
TLS_KEY  /etc/openldap/certs/openldap.key
```

修改服务要监听的地址

```
SLAPD_URLS="ldapi:/// ldaps:///"
```

### 同步时相关配置

在同步时仅仅需要修改slave部分配置即可

```ldif
syncrepl
 rid=000
 type=RefreshandPersist
 provider=ldaps://ldap-master.example.com
 bindmethod=simple
 searchbase="dc=example,dc=com"
 retry="5 3 300 +"
 attrs="*,+"
 binddn="...."
 credentials=....
```

## Troubleshooing

错误 `Re: OpenLDAP/TLS main: TLS init def ctx failed: -207`

- [OpenLDAP/TLS main: TLS init def ctx failed: -207](https://www.openldap.org/lists/openldap-software/200901/msg00134.html)
- [how to configure tls and ldap](https://www.openldap.org/lists/openldap-software/200812/msg00100.html)

错误 `Re: slapadd: could not parse entry (line=13)`

- 可能原因，上下文存在空行

错误:  `openssl slapd tls init def ctx failed: -1` ；证书的路径，文件名，权限是否正确 

验证证书 `openssl verify -CAfile {ca.crt} {.crt}`

- [RE: main: TLS init def ctx failed: -1](https://www.openldap.org/lists/openldap-technical/201502/msg00168.html)

> Reference：
>
> [Chapter 15. LDAP Security](https://www.zytrax.com/books/ldap/ch15/#tls)










