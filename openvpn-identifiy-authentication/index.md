# openvpn identifiy authentication

[toc]

## openvpn的统一身份验证解决方案



### openvpn的统一身份验证解决方案介绍

OpenVPN 2.0与更高版本允许OpenVPN服务器从客户端安全地获取用户名和密码，并将该信息用作认证基础。

### 方法1：通过本地证书密钥认证。

默认不配置，openvpn即使用证书进行身份认证。

![image-20221024220430016](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024220430016.png)

（1）编辑主服务器配置文件/etc/openldap/slapd.conf，取消如下行的注释：

### 方法2：本地文件认证

在使用身份验证时，需要将 `auth-user-pass` 指令添加到客户端配置文件中，设置后OpenVPN客户端向用户索要用户名/密码，并将其通过安全的TLS通道传递给服务器进行验证。

服务端配置文件需要增加配置指令 `auth-user-pass-verify auth-pam.pl via-file` 使用脚本插件。`auth-pam.pl` 在源码包 `sample/sample-script` 路径下。

```
plugin /usr/share/openvpn/plugin/lib/openvpn-auth-pam.so login
```

生产环境下官方推荐使用 `openvpn-auth-pam` 插件进行验证，相比于 **auth-pam.pl**，`openvpn-auth-pam ` 插件具有多个优点：

- `openvpn-auth-pam` 使用拆分权限执行模型来提高安全性。
- C编译的插件比脚本运行速度更快。
- OpenVPN可以通过虚拟内存（而不是通过文件或环境）将用户名/密码传递给插件，这对于服务器上的本地安全性更好。

####  获取**openvpn-auth-pam**插件

**openvpn-auth-pam**插件在openvpn代码目录`src/plugins/auth-pam` 下，运行 `make &amp;&amp; make install` 进行安装，会自动复制到openvpn安装好的 `lib/openvpn/plugins` 目录下。

#### 开启密码认证

默认情况下， 在服务器上使用 `auth-user-pass-verify` 或用户名/密码 **插件** 将启用**双重身份验证**，要求客户端证书和用户名/密码身份验证都必须成功，才能对客户端进行身份验证。可以选择关闭客户端证书认证。

```
client-cert-not-required
username-as-common-name # 用户名作为通用名称
```

开启后需要在客户端注释 **cert** 和 **key**的配置

&gt;Reference
&gt;
&gt;[authentication methods](https://openvpn.net/community-resources/how-to/#using-alternative-authentication-methods)

### 方法3：数据库认证

法2：利用的脚本程序（shell，php等）本地文件去读数据库。

法1：用pam_mysql

### 方法:4：ldap统一用户认证

openvpn-auth-ldap

http://hi.baidu.com/adriannet/item/8d488889b3d7c859850fab5f.

方法2：利用第一个文件认证的思路，去LDAP查询，还可以和本地文件比较。

![绘图1](../../../images/openvpn/绘图1.png)

&lt;center&gt;ldap认证原理图&lt;/center&gt;



####  配置openvpn服务端通过ldap进行身份验证



配置OpenVPN基 LDAP的身份验证，需要安装用于LDAP身份验证的OpenVPN插件。`openvpn-auth-ldap`，它通过LDAP为OpenVPN实现身份认证。

CentOS中 `openvpn-auth-ldap` 插件在EPEL中 ubuntu与Centos都可以通过对应的包管理工具进行插件安装。



安装完成后配置 OpenVPN LDAP身份验证 `examples/auth-ldap.conf`

```
&lt;LDAP&gt;
	URL		ldaps://ip
	BindDN	dc=kifarunix-demo,dc=com
	Password	P@ssW0rd
	Timeout	  15
	TLSEnable	yes|no
	FollowReferrals no
&lt;/LDAP&gt;
&lt;Authorization&gt;
# 搜索的域
	BaseDN		&#34;ou=people,dc=ldapmaster,dc=kifarunix-demo,dc=com&#34;
# 搜索的条件，这里使用的UID，如其他名称为用户名可以选择其他
	SearchFilter	&#34;(uid=%u)&#34;
	RequireGroup	false
&lt;/Authorization&gt;
```

还可以基于组管理

```
&lt;LDAP&gt;
	URL		ldaps://ip
	BindDN	dc=kifarunix-demo,dc=com
	Password	P@ssW0rd
	Timeout	  15
	TLSEnable	yes|no
	FollowReferrals no
&lt;/LDAP&gt;
&lt;Authorization&gt;
	BaseDN		&#34;ou=people,dc=ldapmaster,dc=kifarunix-demo,dc=com&#34;
	SearchFilter	&#34;(uid=%u)&#34;
	RequireGroup	true # 这里设置为true
	&lt;Group&gt;
		BaseDN		&#34;cn=admin,dc=kifarunix-demo,dc=com&#34;
		SearchFilter	&#34;memberOf=ou=people,dc=seal,dc=com&#34;
		MemberAttribute	uniqueMember # 需要ldap安装memberof，这时memeberof组的属性
	&lt;/Group&gt;
&lt;/Authorization&gt;
```

修改openvpn配置

```
plugin /usr/lib64/openvpn/plugin/lib/openvpn-auth-ldap.so &#34;/etc/openvpn/auth/ldap.conf&#34;
verify-client-cert none
```

完整的服务端配置

```
local 10.0.0.4
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
tls-auth ta.key 0
server 192.168.100.128 255.255.255.128
ifconfig-pool-persist ipp.txt
push &#34;route 192.168.100.0 255.255.255.0&#34;
client-to-client
duplicate-cn
keepalive 10 120
max-clients 100
persist-key
persist-tun
log-append  /var/log/openvpn.log
verb 3
compress lz4-v2
mute 20
explicit-exit-notify 1
reneg-sec 360
plugin /usr/lib64/openvpn/plugin/lib/openvpn-auth-ldap.so &#34;/etc/openvpn/auth/ldap.conf&#34;
verify-client-cert none
```

#### 启动客户端配置

```
auth-user-pass
remote-cert-tls server
```

完整的客户端配置

```
client
dev tun
proto udp
remote 10.0.0.4 1194
nobind
resolv-retry infinite
persist-key
persist-tun
mute-replay-warnings
cipher AES-256-CBC
#comp-lzo
verb 3
ca ca.crt
tls-auth ta.key 1
compress lz4-v2
#cert client.crt
#key client.key 
auth-user-pass
remote-cert-tls server
```

服务端报错 `TLS Error: reading acknowledgement record from packet`

```
TLS: Initial packet from [AF_INET]10.0.0.1:56531, sid=50a1c0bb 07a548a5
TLS Error: reading acknowledgement record from packet
```

原因，客户端开启了安全配置`tls-auth ta.key 1` 而 服务端没有对应配置

&gt; Reference
&gt;
&gt; [auth error](https://www.digitalocean.com/community/questions/help-with-the-following-error-tls-error-cannot-locate-hmac-in-incoming-packet-from-af_inet)
&gt;
&gt; [how to configure](https://kifarunix.com/configure-openvpn-ldap-based-authentication/)
&gt;
&gt; [openldap howto](https://openvpn.net/community-resources/how-to/#examples)

### 方法5：配置配置RADIUS认证。

RADIUS:Remote Authentication Dial In User Service，远程用户拨号认证系统由RFC2865，RFC2866定义，是目前应用最广泛的AAA协议。可实现验证、授权、记账等服务的协议。



### 方法6：结合U盾等设备进行双重认证。








