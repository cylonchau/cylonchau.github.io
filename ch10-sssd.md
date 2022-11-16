# ch10 understanding openldap applicaiton - Migrating to SSSD


## Overview

SSSD (***System Security Services Daemon***) 是一套用于远程身份验证的套件服务，为使用SSSD服务的客户端提供了远程访问身份认证服务来获取权限，其后端包括AD, LDAP等，本文将围绕下列方向来阐述SSSD：

- 为什么需要SSSD，以及使用SSSD来解决什么
- 使用SSSD的好处
- SSSD服务工作原理及架构
- 如何在Linux上配置SSSD+LDAP

## 为什么需要SSSD

SSSD设计主要是为了传统使用身份认证服务，例如PAM+NSS架构中存在的一些问题：

- PAM+NSS扩展性差，并配置较为复杂，尽管提供了 `authconfig` ，通常在大多数教程中以及不同的系统中配置都不相同
- PAM+NSS不是真正意义上的离线身份认证，如果当 `nslcd` 或者 `slapd` 等服务异常时，无法完成用户认证
- 以及越来越多的后端，例如LDAP, AD, IPA, IdM,Kerberos等无法做到很好的适配

SSSD就是为了解决上述的问题，对于Linux平台中，SSSD拥有比传统PAM+NSS更好的优势：

- 符合现代Linux基础架构设计需求，可以适配更多的后端，并降低了操作配置的复杂性
- 增加了缓存功能，有效的减少了对于后端服务器的负载
- 因为有了缓存功能，实现了真正的离线认证功能，即使后端服务异常，例如LDAP服务down

## 了解SSSD架构

了解SSSD架构，其实就是了解前两章的内容，要做到真正的多后端，真脱机，那么服务就有多个组件组成：

- ***Monitor***：所有SSSD的父进程，即用于管理 Providers 与 Responders
- ***Providers***：用于感知验证后端的模块，后端就是提供目录树的一端
- ***Responders***：为Linux提供与后端交互的功能，这部分通常为 NSS PAM sudo等

![image-20221114162326295](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221114162326295.png)

<center>图：SSSD架构图</center>
<center><em>Source：</em>https://sssd.io/docs/architecture.html</center><br>

### Providers

- **Local**：保存在本地缓存中的账户信息
- **LDAP**, **Kerberos**, **AD**, 
- **IPA** ：用于 Linux/UNIX 网络环境中集成身份和身份验证解决方案。
- **IdM**：一种使用本地 Linux 工具在 Linux 系统上创建身份存储、集中身份验证、Kerberos 和 DNS 服务的域控制以及授权策略的目录树后端
- **sudo**，**autofs** 与LDAP集成的功能

### Responders

- nss：名称解析服务，用于解析组与用户信息
- pam：用于用户验证的模块
- autofs：自动挂载模块，通常用于与LDAP集成，用于映射LDAP目录树
- sudo：linux中用户权限控制，通常也是与LDAP集成
- ssh：
- sssd_be：SSSD的后端进程：其中每一种后端都代表都作为一个sssd_be进程启动

### monitor

monitor是SSSD的进程，是用于管理（启动，停止，监控服务状态）Provider与Responders的功能

### SSSD工作流程

当每次用户登录，使用 `id`, `getent`, `su` ,  `sudo` 等命令时，都会触发一次查询，下图是整个查询的流程

![image-20221114213534770](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221114213534770.png)

<center>图：SSSD查询流程</center>
<center><em>Source：</em>https://sssd.io/docs/architecture.html</center><br>

可以看到图中描述了对用户 *Alice* 进行查询，从调用函数`getpwnam` 开始，首先会在memcache中检索用户数据，如果检索不到，此时 `sssd_nss`（sssd内置的模块）将从本地cache检索，如果此时还是检索不到，那么 `sssd_be` (上面提到过一个后端会有一个 sssd_be 进程) 将从远程后端检索。

通过这种模式，增加了缓存功能，有效的减少了对于后端服务器的负载，以及完整的脱机查询功能（即使LDAP服务短暂不可用），而存在的问题则是可能会造成本地资源负载过高，例如这个例子：由于服务器忙时，并且`sssd_nss` 造成与其他进程争抢资源 <sup><a href="#2">[2]</a></sup>

## 迁移nslcd到sssd

### 安装sssd

- CentOS 6/CentOS 7：`yum install sssd sssd-tools` <sup><a href="#3">[3]</a></sup>
- Ubuntu/Debian：`apt install sssd-ldap ldap-utils ` <sup><a href="#4">[4]</a></sup>

### 配置SSSD

**先决条件**：建议使用SSL模式与openldap进行通信，此时数据是加密传输的

默认安装好sssd后，不存在配置文件，需要手动创建配置文件  **`/etc/sssd/sssd.conf`** 

```bash
chown root:root /etc/sssd/sssd.conf
chmod 600 /etc/sssd/sssd.conf
```

复制下列配置到  **`/etc/sssd/sssd.conf`** 

```conf
[sssd]
# sssd全局配置，service为需要使用的模块，这里将会启动一个子进程
# 例如传统的nss+pam作为linux认证的基础，这里开启就为nss,pam
services = nss, pam 
config_file_version = 2
# domains作为给后端配置提供的一个名称
domains = default 

# 如果需要对每个模块定义的配置可以[<module_name>]进行配置
[pam]
# 成功登录后的用户在sssd中缓存的天数。 如果为0将意味着永久保存。
offline_credentials_expiration = 60

[domain/default]
# 启用tls通讯
ldap_id_use_start_tls = True

# 这个与offline_credentials_expiration进行配合的参数
# 如果true 将在offline_credentials_expiration天后是否查找缓存
# 如果为false，或不填写该参数将不查找
cache_credentials = True

# 搜索的跟域
ldap_search_base = dc=ldapmaster,dc=kifarunix-demo,dc=com

# 下面是一系列provider
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
access_provider = ldap

# ldap相关配置
ldap_uri = ldap://ldapmaster.kifarunix-demo.com

# 搜索使用的ldap用户
ldap_default_bind_dn = cn=readonly,ou=system,dc=ldapmaster,dc=kifarunix-demo,dc=com

# 搜索使用的ldap用户的密码，仅支持明文
ldap_default_authtok = P@ssWOrd

# tls相关参数

# 这个参数是指定TLS绘画是否对服务器证书进行检查
# never 客户端不检查服务器证书
# allow 请求验证服务端证书，如果没有证书则会话正常进行，如果证书错误，将被忽略
# demand 请求验证服务端证书，如果证书错误或者没有证书，终止会话
# try  请求验证服务端证书，如果没有证书则会话正常进行，如果证书错误，终止会话
ldap_tls_reqcert = demand
ldap_tls_cacert = /etc/openldap/certs/cacert.pem

# 与ldap服务端通信超时相关配置
ldap_search_timeout = 50
ldap_network_timeout = 60

# 搜索用户的参数，下列是默认条件，这是强制参数，sssd在ldap上搜索用户的搜索条件，如果目录树是特别的名称需要更改
ldap_access_order = filter
ldap_access_filter = (objectClass=posixAccount)
# 例如 ldap_access_filter = memberOf=cn=allowedusers,ou=Groups,dc=example,dc=com
```

配置完成后可以启动服务，该服务与使用 `nlscd` 一样，需要开机自启，否则远端用户将不能完成认证

```bash
systemctl start sssd
systemctl enable sssd
```

完成后需要配置下 nss 与 pam 的配置

- CentOS 7：`authconfig --enablesssd --enablesssdauth --enablemkhomedir --update`
- CentOS 8：`authselect apply-changes -b --backup=ldap-configuration-backup`
- Ubuntu：`pam-auth-update --enable mkhomedir`

> Notes：参数根据平台不同命令也不同，可以man查看下具体需要配置什么

### sudo over sssd

对于sudo方面，配置没有使用nss+pam架构那么复杂只需要加几个参数即可使用sssd作为sudo认证

```conf
[sssd]
..
# service 增加 sudo
services = nss, pam, sudo, ssh
domains = default
debug_level = 6

[sudo]
# 枚举授信域
subdomain_enumerate = true
debug_level = 9

[domain/default]
...
# provider 增加 sudo_provider
sudo_provider = ldap
# 配置sudo默认搜索域，也就是sudoers的跟容器，这个必须设置
ldap_sudo_search_base = ou=SUDOers,dc=test,dc=com

# sssd在下载ldap服务端的sudo规则间隔秒数
# 默认21600
ldap_sudo_full_refresh_interval=86400

# 智能刷新，默认900秒，可以设置为0禁止只能刷新
# 该参数是指，下载条目为服务端USD高于当前SSSD的USN最高值的所有规则
# USD Update Sequence Number 代表数据变化的序列
ldap_sudo_smart_refresh_interval=3600
```

下面为sudo 与 NSS+PAM 迁移至SSSD的完整配置

```conf
[sssd]
config_file_version = 2
services = nss, pam, sudo, ssh
domains = default
debug_level = 6

[pam]
offline_credentials_expiration = 60

[sudo]
subdomain_enumerate = true
debug_level = 9

[domain/default]
id_provider = ldap
auth_provider = ldap
sudo_provider = ldap
ldap_uri = ldaps://10.0.0.10/
ldap_search_base = dc=test,dc=com
ldap_sudo_search_base = ou=SUDOers,dc=test,dc=com
ldap_default_bind_dn = uid=searchUser,ou=tvb,dc=test,dc=com
ldap_default_authtok_type = password
ldap_default_authtok = 1
cache_credentials = True
ldap_search_timeout = 50
ldap_network_timeout = 60
ldap_access_order = filter
ldap_access_filter = (objectClass=posixAccount)
ldap_tls_cacert = /etc/ssl/certs/cacert.crt
ldap_id_use_start_tls = true
ldap_tls_reqcert = allow
ldap_sudo_full_refresh_interval=86400
ldap_sudo_smart_refresh_interval=3600
```

此时验证用户登录与sudo是使用SSSD缓存还是通过每次请求slapd

> Notes：对于更多的配置参数的说明，可以使用 ` man sssd` , `man sssd-ldap` .. 进行查询 ，也可以通过 [linux man](https://linux.die.net/man/5/sssd-ldap) 手册进行查询

## Troubleshooting

ldap 日志报错 `TLS established tls_ssf=256 ssf=256` <sup><a href="#5">[5]</a></sup>

```log
Sep 19 12:16:40 centos6 slapd[16620]: conn=228 fd=14 ACCEPT from IP=client-IP:client-Port (IP=0.0.0.0:636)
Sep 19 12:16:40 centos6 slapd[16620]: conn=228 fd=14 TLS established tls_ssf=256 ssf=256
```

这里原因是，如果你使用TLS进行通讯，只有基于 `ldap://[port_389]` 才是TLS，如果使用 `ldaps://[port_636]` 那么是通过SSL隧道进行的

解决：对于SSSD端需要开启对应的TLS配置，如下

```conf
ldap_tls_cacert = /etc/ssl/certs/cacert.crt
ldap_id_use_start_tls = true
# 对服务器提供的证书执行的检查,因为服务端配置了要验证客户端证书
ldap_tls_reqcert = allow
```



> **Reference**
>
> <sup id="1">[1]</sup> [***sssd architecture***](https://sssd.io/docs/architecture.html)
>
> <sup id="2">[2]</sup> [***[High CPU usage by sssd_nss during heavy disk IO](https://stackoverflow.com/questions/49618032/high-cpu-usage-by-sssd-nss-during-heavy-disk-io)***](https://stackoverflow.com/questions/49618032/high-cpu-usage-by-sssd-nss-during-heavy-disk-io)
>
> <sup id="3">[3]</sup> [***SSSD and LDAP***](https://ubuntu.com/server/docs/service-sssd-ldap)
>
> <sup id="4">[4]</sup> [***Chapter 10. Migrating authentication from nslcd to SSSD***](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_authentication_and_authorization_in_rhel/assembly_migrating-authentication-from-nslcd-to-sssd_restricting-domains-for-pam-services-using-sssd#doc-wrapper)
>
> <sup id="5">[5]</sup> [***OpenLDAP Client 2.4.23: TLS negotiation failure***](https://www.linuxquestions.org/questions/linux-desktop-74/openldap-client-2-4-23-tls-negotiation-failure-903809/)
>
> <sup id="6">[6]</sup> [***Chapter 10. Migrating authentication from nslcd to SSSD***](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_authentication_and_authorization_in_rhel/assembly_migrating-authentication-from-nslcd-to-sssd_restricting-domains-for-pam-services-using-sssd#doc-wrapper) 
>
> <sup id="7">[7]</sup> [***Configure SSSD***](https://www.ibm.com/docs/en/cloud-paks/cp-management/2.2.x?topic=SSFC4F_2.2.0/Infra_mgmt/auth/ldap.htm#configure-sssd)
>
> <sup id="8">[8]</sup> [***Configure OpenLDAP SSSD client on CentOS 6/7***](https://kifarunix.com/configure-openldap-sssd-client-on-centos-6-7/)

