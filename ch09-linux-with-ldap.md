# ch9 understanding openldap applicaiton - linux authentication with ldap


[toc]

## Overview

如果要使Linux账号通过LDAP进行身份认证，就需要配置Linux的 身份验证模块 (***Pluggable Authentication Modules***) 与 名称服务交换系统 (***Name Service Switch***) 与LDAP交互。

## PAM 和 NSS <sup><a href="#3">[3]</a></sup>

NSS (***name service switch***) 通俗理解为是一个数据库系统，他作用是用于如何将操作系统与各种名称的解析机制关联起来，例如主机名，用户名，组名等内容的查找；例如UID查找使用 `passwd` 库，GID的查找使用 `group` 库，并且还可以告知查找的来源，如文件，LDAP等

PAM (***Pluggable Authentication Modules***) 全称是可插拔的认证模块，PAM在Linux中是位于用户数据库与应用之间的认证模块，它本身并不工作，并且本身也不提供或扩展现有数据库系统，当登陆shell时，依赖于由NSS提供的密码库与组库等信息，完成对应的查询

例如下列两张图完整的阐述了PAM与NSS之间，在用户登陆时做了些什么

![image-20221112160527824](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221112160527824.png)

<center>图：pam和nss工作示意图1</center>
<center><em>Source：</em>https://medium.com/@fengliplatform/understanding-nss-and-pam-using-a-ssh-example-80512eb0f39e</center><br>

由图可以看出，当在进行 `ping` , `id` 等操作时，会通过nss找到 `passwd` 库找到用户id，以及通过nss确定是 hosts解析还是dns服务解析对应的域名

如果这张图不明白可以看下一张图

![image-20221112160836827](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221112160836827.png)

<center>图：pam和nss工作示意图2-1</center>
<center><em>Source：</em>https://medium.com/@fengliplatform/understanding-nss-and-pam-using-a-ssh-example-80512eb0f39e</center><br>

图2-1 中使用了tom用户去登录pecan主机，此时在节点 `yam` 上，将寻找 pecan主机的IP，这是通过 `/etc/nsswitch.conf` 来确定是通过 `hosts` 还是 dns服务进行查找。

接下来找到pecan的IP，这里会输入用户名与密码，这里将会被sshd服务接管，此时 pecan 主机的sshd接收到用户端请求连接后，将用户名通过nss进行识别，确定是否为合法用户，如果用户有效，则通过PAM进行认证。认证的源也将由 `/etc/nsswitch.conf`  中配置的对应 `passwd` 库来找到，例如ldap,file等。正如下图2-2所示

![image-20221112161605611](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221112161605611.png)

<center>图：pam和nss工作示意图2-2</center>
<center><em>Source：</em>https://medium.com/@fengliplatform/understanding-nss-and-pam-using-a-ssh-example-80512eb0f39e</center><br>

## Linux with LDAP <sup><a href="#1">[1]</a></sup>

在大致了解了Linux登录认证的原理后，知道了要使Linux使用LDAP需要配置两个部分，NSS与PAM，通常有下述几种方案：

- NSS + PAM
- SSSD (***System Security Services Daemon***)，SSSD是提供严重的一种工具，可以包含多种源例如LDAP，AD，Kerberos 等，并且提供了缓存功能（当ldap不可用时提供服务）

### 配置NSS

安装 `nss-pam-ldapd`

- CentOS/Redhat：`	yum install -y nss-pam-ldapd`
- Debian/Ubuntu：`apt-get install libnss-ldapd`

配置 ***/etc/nsswitch.conf*** ，该文件保存了各数据库，需要对 `group` , `passwd ` , `shadow ` 库开启 ldap

```conf
passwd:     files sss ldap
shadow:     files sss ldap
group:      files sss ldap
```

### 配置PAM

- CentOS/Redhat：`	yum install -y pam_ldap`
- Debian/Ubuntu：`apt-get install libpam-ldapd`

要想通过本地 Sudo 实现 OpenLDAP 用户提权，可按以下步骤操作。

```sh
authconfig \
--enableldap \
--enableldapauth \
--ldapserver=ldap://10.0.0.10:389 \
--ldapbasedn="dc=test,dc=org" \
--enablemkhomedir \
--ldapbasedn="ou=tvb,dc=test,dc=org" \
--enableshadow \
--update \
```

> Notes：
>
> - CentOS/RedHat 也可以使用 SSSD替代，`yum install -y sssd` ，上面提供的方案是NSS+PAM
>
> - 通常情况下Ubuntu会使用 SSSD 服务替代，SSSD包含了 PAM模块 和 NSS模块
>
> - Ubuntu中，没有被 `authconfig` 没有被打包在 SSSD 中，通常需要安装 `sudo apt install ldap-auth-config`

## Linux用户权限控制

部署此功能的原因是能够在所有基础结构服务器上仅使用一个用户，并且无需每次在每台服务器上手动更新 <font color="#f8070d" size=3>`/etc/sudoers`</font> 文件，即可为此用户提供sudo权限。现在，这些天您可以使用像ansible这样的工具来执行此操作，但是并不是说OpenLDAP用法必须仅用于posixGroup用户访问，因此OpenLDAP只擅长它。OpenLDAP集成应扩展到您部署的每个集中式系统，并且您唯一的“管理员”用户可以访问基础架构范围内的所有系统。

### sudoers在slapd部分配置

要使 `OpenLDAP` 服务端实现用户权限控制，具体的实施步骤可以大致分为如下几步：

- 创建 ldap 中 sudoers容器，并创建默认的搜索域

- 为 `slapd` 导入sudo schema

- 定义sudo规则条目及sudo组

    - 通过手动定义用户加入sudo组，集成sudo权限
        - 命令添加及修改

    - 通过转换 本地 sudoers 配置文件 为LDAP ldif格式文件
        - OpenLDAP 提供的 perl脚本  `sudoers2ldif` ，通常会看到，如果没有可以从 [Github](https://github.com/lbt/sudo/blob/master/plugins/sudoers/sudoers2ldif) 中下载 
        - sudo 提供的转换命令 `cvtsudoers` 

- 图形化管理界面配置

- 客户端配置加入OpenLDAP服务端

- 客户端识别sudo策略及验证用户权限


### OpenLDAP服务端导入sudo schema

```sh
# 找到sudo的openldap schema
rpm -ql sudo | grep schema

# 将其复制到openldap schema目录
cp /usr/share/doc/sudo-1.8.23/schema.OpenLDAP /etc/openldap/schema/sudo.schema  

# 生成include sudo.ldif
echo "include   /etc/openldap/schema/sudo.schema" >  /tmp/sudo.conf
slapcat -f /tmp/sudo.conf -F /tmp/ -n 0 -s "cn={0}sudo,cn=schema,cn=config" > /tmp/sudo.ldif

# 最后需要删除后面几行
sed -i "s@{0}sudo@sudo@g" /tmp/sudo.ldif
head -n -8 /tmp/sudo.ldif > /etc/openldap/schema/sudo.ldif
```

生成的对应schema在：<font color="#f8070d" size=3>`/etc/openldap/slapd.d/cn=config/cn=schema/`</font> 

### 为 OpenLDAP 创建 suers 容器

创建ldap的sudoers容器，官网有给出提示，sudoers如果在ldap中使用必须放在 `ou=SUDOers` 中，其中 `cn=default` 为最先被查找的条目

> The *sudoers* configuration is contained in the ‘`ou=SUDOers`’ LDAP container. <sup><a href="#2">[2]</a></sup>

默认情况下，sudo检索的域为  `cn=default,ou=SUDOers,dc=xx,dc=xx` 如果找到，那么该条目中所有 `sudoOption` 属性都会被解析为全局默认值，这类似于服务端中查询一个 sudo 用户权限时一般有两到三次查询。

- 第一次查询解析全局配置
- 第二次查询匹配用户名或者用户所在的组（特殊标签 ALL 也在此次查询中匹配），如果没有找到相关匹配项，则发出第三次查询，此次查询返回所有包含用户组的条目并检查该用户是否存在于这些组中

下面命令是创建一个 sudoers 在 ldap中的根容器 `ou=SUDOers` ，这个步骤总是需要手动执行，因为默认通过 `/etc/sudoers` 转换过来的 ldif 是不带这个域的

```bash
$ cat << EOF | ldapadd -D "cn=admin,dc=test,dc=com" -w 111 -H ldap://10.0.0.10:389
dn: ou=SUDOers,dc=test,dc=com
objectclass: organizationalunit
ou: SUDOers
EOF
```

下面步骤是将 `/etc/sudoers` 转换为 ldap `ldif` 

创建一个环境变量 `SUDOERS_BASE` ，这个在perl脚本执行的必须条件。

```bash
export SUDOERS_BASE="ou=SUDOers,dc=test,dc=com"
```

接下来，使用sudo包中提供了一个命令 `cvtsudoers ` 可以将  sudoers的配置文件 `/etc/sudoers` 转换为LDAP ldif格式文件

```bash
cvtsudoers /etc/sudoers -f ldif -o sudoers_defaults.ldif
```

另外，通过脚本将 sudoers的配置文件 `/etc/sudoers` 转换为LDAP ldif格式的文件，将用这个来创建默认的`cn=default` 的sudoers

```bash
perl sudoers2ldif /etc/sudoers > sudoers_defaults.ldif
```

> Notes：openldap还有一种操作为 *ldif to schema*，通过下述命令可以完成 <sup><a href="#4">[4]</a></sup>
>
> ```bash
> sed '/^dn: /d;/^objectClass: /d;/^cn: /d;s/olcAttributeTypes:/attributetype/g;s/olcObjectClasses:/objectclass/g' file.ldif > file.schema
> ```

接下来将这些配置导入到ldap中

```bash
ldapadd -D "cn=admin,dc=test,dc=com" -w 111 -H ldap://10.0.0.10:389 -f sudoers_defaults.ldif
```

### 配置客户端主机支持sudo over ldap

在客户端，需要两个额外步骤。

**需要添加**：<font color="#f8070d" size=3>` /etc/nsswitch.conf`</font>

```sh
 ## 增加
 sudoers:    ldap files
 
 echo 'sudoers:    ldap files' >> /etc/nsswitch.conf
```

**需要提供**：<font color="#f8070d" size=3>`/etc/sudo-ldap.conf`</font>

```sh
binddn cn=clientsearch,ou=admin,dc=cylon,dc=org
bindpw 111
uri ldap://10.0.0.20
sudoers_base ou=sudoers,dc=cylon,dc=org
```

## Troubleshooting

错误：`invalid structural object class chain`

```bash
ldap_add: Object class violation (65)
	additional info: invalid structural object class chain (groupOfUniqueNames/posixGroup)
```

原因：在ldap中 `groupOfUniqueNames` / `posixGroup` / `inetOrgPerson` 实际上是同一种类型 <sup><a href="#6">[6]</a></sup> ，级别的对象组，其 `objectClass` 只能选择包含一个，要 `gidNumber` 就不能有 `memberOf` 了。

**问题出于**：对于筛选来说，`posixGroup` 组并不支持  `memberOf`  属性，这种情况下可能无法做到权限的筛选与用户的筛选等，但是对于Linux 使用ldap管理用户权限时，普通的 `groupOfUniqueNames` 并不带有 `gidNumber` 属性，使得用户没有组，这时配置的组的权限，sudo实际不生效。

**解决**：

- 为 `posixGroup`  生成一个 `memberOf` 属性 <sup><a href="#5">[5]</a></sup>，这里在尝试导入 ldif 文件时 slapd 生成配置失败无报错
- 直接使用 `posixGroup` 替换组  `groupOfUniqueNames` 
    - `posixGroup` 使用的是 `memberUid` 进行关联，在检索时，可以使用 `uid=memberUid` 进行过滤
    - 本质上 `posixGroup` 与  `groupOfUniqueNames` 只是对组，没有对用户
    - 需要配置 refint 解决引用关系一致性问题，正如下列给出配置一样
        ```ldif
        dn: olcOverlay=refint,olcDatabase={1}mdb,cn=config
        objectClass: olcConfig
        objectClass: olcOverlayConfig
        objectClass: olcRefintConfig
        objectClass: top
        olcOverlay: refint
        olcRefintAttribute: memberUid uid
        olcRefintNothing: cn=default,dc=test,dc=com
        ```


> **Reference**
>
> <sup id="1">[1]</sup> [***Configure OpenLDAP login for CentOS 7***](https://joeho.xyz/blog-posts/configure-openldap-login-for-centos7/)
>
> <sup id="2">[2]</sup> [***sudoers.ldap.man***](https://www.sudo.ws/docs/man/sudoers.ldap.man/#SUDOers_LDAP_container)
>
> <sup id="3">[3]</sup> [***Understanding nss and pam using a ssh example***](https://medium.com/@fengliplatform/understanding-nss-and-pam-using-a-ssh-example-80512eb0f39e)
>
> <sup id="4">[4]</sup> [***Convert AD-Schema from \*.ldif to \*.schema***](https://serverfault.com/questions/818393/openldap-convert-ad-schema-from-ldif-to-schema)
>
> <sup id="5">[5]</sup> [***GENERATING A MEMBEROF ATTRIBUTE FOR POSIXGROUPS***](http://blog.oddbit.com/post/2013-07-22-generating-a-membero/)
>
> <sup id="6">[6]</sup> [***how to fix (65) invalid structural object class chain (posixGroup/groupOfNames)?***](https://ldap.umich.narkive.com/4c1rn2Qz/how-to-fix-65-invalid-structural-object-class-chain-posixgroup-groupofnames) 
>
> <sup id="7">[7]</sup> [***Configure OpenLDAP login for CentOS 7***](https://joeho.xyz/blog-posts/configure-openldap-login-for-centos7/)
>
> <sup id="8">[8]</sup> [***Linux LDAP Authentication***](https://linuxhint.com/linux-ldap-authentication-2/)
>
> <sup id="9">[9]</sup> [***Linux user management with LDAP***](https://www.vennedey.net/resources/1-Linux-user-management-with-LDAP)
>
> <sup id="10">[10]</sup> [***2. LDAP authentication using pam_ldap and nss_ldap***](https://tldp.org/HOWTO/archived/LDAP-Implementation-HOWTO/pamnss.html)
>
> <sup id="11">[11]</sup> [***How to Configure SUDO via OpenLDAP Server***](https://kifarunix.com/how-to-configure-sudo-via-openldap-server/)
>
> <sup id="12">[12]</sup> [***SUDOers from OpenLDAP***](http://pig.made-it.com/ldap-sudoers.html)
>
> <sup id="13">[13]</sup> [***OpenLDAPSudo权限讲解-OpenLDAPSudo权限讲解***](https://wiki.shileizcc.com/confluence/pages/viewpage.action?pageId=40566794#OpenLDAPSudo权限讲解-OpenLDAPSudo权限讲解)
>
> <sup id="14">[14]</sup> [***authconfig***](https://linux.die.net/man/8/authconfig)
>
> <sup id="15">[15]</sup> [***Configuring PAM Authentication and User Mapping with LDAP Authentication***](https://mariadb.com/kb/en/configuring-pam-authentication-and-user-mapping-with-ldap-authentication/)

