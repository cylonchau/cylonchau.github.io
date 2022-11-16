# ch6 understanding openldap configuration - Access Control


## Overview

访问控制 (***Access Control***) 是对目录树中的IDT访问的权限控制。主要指 “谁” 应该能够 “访问记录” 在 “什么条件下” 他们应该能看到多少这样的记录，这些将是本节中阐述的问题 。

OpenLDAP控制目录数据访问的主要方法是 通过访问控制列表 (***Access Control List***)。使 slapd 服务端在处理来自客户端的请求时，会评估客户端是否具有访问所请求的 DIT 的权限。要执行此计算，slapd 将依次计算***LDIF*** 中配置的每个ACL策略，以检查客户端是否有权限访问该 DIT。

***

> **Note**： 
>
> - ACL策略由上而下依次进行匹配
> - 默认的访问控制策略是对所有客户端都允许读取，不管定义了什么ACL策略，`rootdn ` （databases部分设置的）总是允许对所有和任何东西拥有完全的权限（即身份验证、搜索、比较、读和写 ）

***

## ACL介绍

访问控制主要定义三大方面：

- `what` 定义对那些地方的访问，部分选择应用访问的条目和/或属性
- `who` 定义人员，部分指定授予哪些实体访问
- `access` 定义权限，部分指定授予的访问。

```
access to [what]
    by [who] [control]
    by [who] [control]
```



```
access to [resources]
    by [who] [type of access granted] [control]
    by [who] [type of access granted] [control]
```

对于完整的ACL语法，如下面所示 <sup><a href="#1">[1]</a></sup>

```
<access directive> ::= access to <what>
	[by <who> [<access>] [<control>] ]+
<what> ::= * |
	[dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
	[filter=<ldapfilter>] [attrs=<attrlist>]
<basic-style> ::= regex | exact
<scope-style> ::= base | one | subtree | children
<attrlist> ::= <attr> [val[.<basic-style>]=<regex>] | <attr> , <attrlist>
<attr> ::= <attrname> | entry | children
<who> ::= * | [anonymous | users | self
			| dn[.<basic-style>]=<regex> | dn.<scope-style>=<DN>]
	[dnattr=<attrname>]
	[group[/<objectclass>[/<attrname>][.<basic-style>]]=<regex>]
	[peername[.<basic-style>]=<regex>]
	[sockname[.<basic-style>]=<regex>]
	[domain[.<basic-style>]=<regex>]
	[sockurl[.<basic-style>]=<regex>]
	[set=<setspec>]
	[aci=<attrname>]
<access> ::= [self]{<level>|<priv>}
<level> ::= none | disclose | auth | compare | search | read | write | manage
<priv> ::= {=|+|-}{m|w|r|s|c|x|d|0}+
<control> ::= [stop | continue | break]
```

- `what` 部分将确定如何控制客户端访问的条目，通常为三种方式：
    - 通过DN
    - 通过过滤器 (***filter***)
    - 通过属性 (***attributes***)
- `who` 部分将确定被授予访问权限的用户实体
    - `<who>` ，  `<access>` ， `<control>`
- `access` 部分确定了实体的访问权限

## 控制访问内容

访问内容也是 ACL 中 ***what*** 部分，例如下属为简单表示的语法格式

```
to * 
to dn[.<basic-style>]=<regex> 
to dn.<scope-style>=<DN>
```

- 其中 `to *` 表示选择所有条目
- `to dn[.<basic-style>]=<regex> ` ：通过正则表达式与DIT规范的DN来选择内容
- `to dn.<scope-style>=<DN>` ：内容将被限制在DN范围内

其中范围包含下述几项：

- base：仅匹配提供DN的条目
- one：父条目是指定DN的条目
- subtree：匹配指定DN下的所有子树中的条目，包含RootDN
- children：匹配指定的DN下的所有子条目，不包含RootDN

例如下列实例

```
 0: o=suffix 
 1: cn=Manager,o=suffix 
 2: ou=people,o=suffix 
 3: uid=kdz,ou=people,o=suffix 
 4: cn=addresses,uid=kdz,ou=people, o=suffix 
 5：uid=hyc,ou=people,o=suffix
```

- 如果请求为 `dn.base="ou=people,o=suffix"` 将匹配2
- 如果请求为 `dn.one="ou=people,o=suffix"` 将匹配 3, 5
- 如果请求为 `dn.subtree="ou=people,o=suffix"` 将匹配 2, 3, 4, 5
- 如果请求为 `dn.children="ou=people,o=suffix"` 将匹配 3, 4, 5

---

> **Notes**：在配置ACL时，`to *` 通常放置在ACL策略中最下层，即代表不匹配所有的权限时给的权限

---

### 通过DN匹配示例

```
access to dn.regex="uid[^,]+,ou=Users,dc=example,dc=com"
```

这表示 所有 来自任意 uid 为逗号开头的 `ou=Users,dc=example,dc=com` 域的权限

### 使用filter匹配

```
to filter=<ldap filter>
```

**例** ：DN与filter可以同时包含在 `<what> `子句中

```
access to filter=(objectClass=person)
access to filter="(|(|(givenName=Matt)(givenName=Barbara))(sn=Kant))"
access to dn.subtree="ou=Users,dc=example,dc=com" filter="(employeeNumber=*)"
```

**例** ： `<what>` 子句中也可以包含属性来进行过滤，多个属性可以通过符号 “`,`” 分割

```
attrs=<attribute list>
access to attrs=userPassword,shadowLastChange
```

**例**：通过使用单个 `attribute` 与使用value选择器来过滤出特定的属性

```
attrs=<attribute> val[.<style>]=<regex>

access to dn.children="cn=people,dc=stanford,dc=edu" attrs=suPrivilegeGroup val.regex="^stanford:.+"
```

> Notes： “ `*` ”  代表 `dn=*`

### WHO

`<who>` 部分标识被授权访问的角色，例如被授权访问的用户

***

> Notes：who 代表的是一个实体，例如用户，组，域，二不是访问的条目

***

**下表总结了who拥有的角色**


| 标识符                       | 访问实体                           |
| ---------------------------- | ---------------------------------- |
| *                            | 所有，包括匿名和经过身份验证的用户 |
| anonymous                    | 匿名用户（未验证）                 |
| users                        | 通过身份验证的用户                 |
| self                         | 与目标条目关联的用户               |
| `dn[.<basic style>]=<regex>` | 匹配正则表达式的用户               |
| `dn.<scope-style>=<DN>`      | DN范围内的用户                     |

**例**：who子句中的 `dnattr` 仅适用于符合ldap语法的DN的属性，而不是其他属性

```
dnattr=<dn-valued attribute name>
by *dnattr*=orgaAdmin
```

### ACCESS

`access` 类型可以是下列之一：

| 级别     |    权限 | 描述               |
| -------- | ------: | ------------------ |
| none     |       0 | 禁止访问           |
| disclose |       d | 检测信息是否存在   |
| auth     |      dx | 需要验证           |
| compare  |     cdx | 需要比较           |
| search   |    scdx | 需要应用搜索过滤器 |
| read     |   rscdx | 需要读搜索结果     |
| write    |  wrscdx | 需要修改/重命名    |
| manage   | mwrscdx | 需要管理           |

其中权限字段字母意思为：

- `w`：对记录或属性的写访问。
- `r`：对记录或属性的读访问。
- `s`：对记录或属性的搜索访问。
- `c`：访问对记录或属性运行比较操作。
- `x`：访问对记录或属性执行服务器端身份验证操作。
- `d`：访问记录或属性是否存在的信息 (d 代表“disclose” )。
- `0`：不允许访问记录或属性。这相当于 wrscxd 。
- `m`：管理权限

**例** ：最简单的access策略，限制所有人都可以读

```
access to * by * read
```

**例** ：下面示例为，对于条目所属者可以修改自己，匿名用户需要登陆，所有人可以读，对于 `by anonymous auth` 仅仅是声明了匿名用户可以登录，而读的属性是下面那条声明的。

```
access to *
    by self write
    by anonymous auth
    by * read
```

**例** ：使用 SSF `Security Strength Factor` 进行认证，SSF表示为openldap中保护的强度，与密钥长度有关

下列带包了，如果ssf强度大于128的将允许被修改自己，大于64的匿名用户可以读操作

```
access to *
    by ssf=128 self write
    by ssf=64 anonymous auth
    by ssf=64 users read
```

**例** ：下面示例为 DN示例，这将代表了人员必须有所有用户搜索 `dc=example,dc=com` 子目录，并且对  `dn.children="dc=com` 的子目录有读权限，而这里 `dc=example,dc=com` 为 `dc=com` 子目录，所以说如果要读 目录 `dc=example,dc=com` 此时没有权限，因为ACL策略是按照顺序进行的，匹配到后就结束了

```
access to dn.children="dc=example,dc=com"
	by * search
access to dn.children="dc=com"
	by * read
```

**例** ：还需要注意的一点是，所有列出的策略结尾都会隐式增加一条 `access to * by * none` 即，当没有通过who子句，也会被拒绝。

例如该策略将允许 `dc=example,dc=com` 子目录，并且属性 `homePhone` 的属于自己条目的可以写入，可以进行搜索，并包含 `dn=dc=example,dc=com` 下的所有子条目，并且网络客户端为10开头的ip都可以读，其他的将被隐式策略拒绝

```
access to dn.subtree="dc=example,dc=com" attrs=homePhone
    by self write
    by dn.children="dc=example,dc=com" search
    by peername.regex=IP=10\..+ read
access to dn.subtree="dc=example,dc=com"
    by self write
    by dn.children="dc=example,dc=com" search
    by anonymous auth
```

**例** ：有些场景下需要只有组内成员才能删除自己租的条目对象，这个时候就需要用到 `dnattr ` 作为 who 子句的选择器；下面的策略表示只有符合这个域中的成员才能删除自己的DN，这里attrs是必须的

```
access to attrs=member,entry
	by dnattr=member selfwrite
```

### ACL的排序

ACL的顺序也非常重要，如果不配置，将按照默认的固定顺序进行，但是也可以控制顺序的先后，通过为每个值添加 `{x}` x 为数字，这样可以做到ACL的排序

例如下列策略，在执行时是由 slapd 服务自动维护的

```
olcAccess: to attrs=member,entry 
	by dnattr=member selfwrite 
olcAccess: to dn.children="dc=example,dc=com" 
	by * search 
olcAccess: to dn.children="dc=com" 
	by * read
# 在没指定顺序时，将按照下列顺序执行
olcAccess: {0}to attrs=member,entry 
	by dnattr=member selfwrite 
olcAccess: {1}to dn.children="dc=example,dc=com" 
	by * search 
olcAccess: {2}to dn.children="dc=com" 
	by * read
```

比如说在生成好的策略进行动态修改时，例如下列所示，此时时修改顺序将是乱的

```
changetype: modify
delete: olcAccess
olcAccess: to dn.children="dc=example,dc=com" by * search
-
add: olcAccess
olcAccess: to dn.children="dc=example,dc=com" by * write
```

为了确保可以修改到指定的条目，需要指定对应条目的index，如果并不确定index是多少，可以通过 `/etc/openldap/slapd.d/` 对应的文件查看

```
changetype: modify
delete: olcAccess
olcAccess: {1}
-
add: olcAccess
olcAccess: {1}to dn.children="dc=example,dc=com" by * write
```

### 访问控制示例


- 关闭匿名访问
- 创建一个目录管理员组，属于该组的人员都具有管理目录的权限
- 创建一个配置管理员组，属于该组的人员都具有配置 openldap 的权限
- 创建一个复制权限用户 ，该账号用于高可用之间的复制
- 创建一个搜索用户，用于 linux 集中账号认证时的，搜索

> 提示：
>
> - 目录管理员组：操作目录树的管理员都属于目录管理员组。
> - 配置管理员组：可以操作服务器配置的（`slapd.ldif`）属于配置管理员组

### 关闭匿名访问

首先slapd默认是允许匿名用户登录的，现在关闭匿名用户登录 在全局部分配置如下（通常全局在配置文件前几行）

```
olcDisallows: bind_anon
```

### 初始化一些组与用户

创建RootDN与一些组，这里创建了三个组 ：目录管理员组 `dirGroup`，配置管理员组 `confGroup` ，与高级管理员组 `adminGroup` 。

> 注：
>
> - 这些是在初始化安装好 openldap 环境进行的，而不是存在数据的环境
> - 这里创建的组使用的 `groupOfUniqueNames` 

```bash
cat << EOF | ldapadd -x -H ldap://10.0.0.10 -D "cn=admin,dc=test,dc=com" -w 111
dn: ou=tvb,dc=test,dc=com
objectClass: organizationalUnit
ou: tvb

dn: cn=adminGroup,ou=tvb,dc=test,dc=com
objectClass: groupOfUniqueNames
cn: adminGroup
uniqueMember: uid=admin,ou=tvb,dc=test,dc=com

dn: cn=dirGroup,ou=tvb,dc=test,dc=com
objectClass: groupOfUniqueNames
cn: dirGroup
uniqueMember: uid=searchUser,ou=tvb,dc=test,dc=com

dn: cn=confGroup,ou=tvb,dc=test,dc=com
objectClass: groupOfUniqueNames
cn: confGroup
uniqueMember: uid=syncUser,ou=tvb,dc=test,dc=com
EOF
```

初始化目录管理员用户 `syncUser` ，配置管理员用户 `searchUser` ，高级管理员用户 `admin` ，密码均为111

```bash

cat << EOF | ldapadd -x -H ldap://10.0.0.10 -D "cn=admin,dc=test,dc=com" -w 111
dn: uid=syncUser,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
uid: syncUser
cn: syncUser
uidNumber: 10006
gidNumber: 10002
userPassword: {SSHA}QnB7dO98+hoCUgiaAYaiJWnDzlhn2Tn6
homeDirectory: /home/syncUser
loginShell: /bin/bash
sn: syncUser
givenName: syncUser
memberOf: cn=confGroup,ou=tvb,dc=test,dc=com

dn: uid=searchUser,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
uid: searchUser
cn: searchUser
uidNumber: 10005
gidNumber: 10001
homeDirectory: /home/searchUser
loginShell: /bin/bash
userPassword: {SSHA}QnB7dO98+hoCUgiaAYaiJWnDzlhn2Tn6
sn: searchUser
givenName: searchUser
memberOf: cn=dirGroup,ou=tvb,dc=test,dc=com

#cat << EOF | ldapadd -x -H ldap://10.0.0.10 -D "cn=admin,dc=test,dc=com" -w 111
dn: uid=admin,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
uid: admin
cn: admin
uidNumber: 0
gidNumber: 0
homeDirectory: /home/admin
loginShell: /bin/bash
userPassword: {SSHA}QnB7dO98+hoCUgiaAYaiJWnDzlhn2Tn6
sn: admin
givenName: admin
memberOf: cn=adminGroup,ou=tvb,dc=test,dc=com

dn: uid=admin1,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
uid: admin1
cn: admin
uidNumber: 0
gidNumber: 0
homeDirectory: /home/admin
loginShell: /bin/bash
userPassword: {SSHA}QnB7dO98+hoCUgiaAYaiJWnDzlhn2Tn6
sn: admin
givenName: admin
memberOf: cn=adminGroup,ou=tvb,dc=test,dc=com
EOF
```

### 创建权限：管理员用户组有所有权限

这句意思是筛选出dn下的所有不包含uniqueMember的用户，即找到`dn=cn=adminGroup,ou=tvb,dc=test,dc=com`  在上面我们把起规划为一个组，然后检查他的属性 `uniqueMember` ，`*` 表示递归，这里不用填，所有的 `uniqueMember` 均为最底层 <sup><a href="#2">[2]</a></sup>

```
 by set="[cn=adminGroup,ou=tvb,dc=test,dc=com]/uniqueMember* & user" manage
```

### 创建权限：普通用户仅允许修改自己的密码

这里使用属性选择器进行属性筛选

```bash
olcAccess: to attrs="userPassword"
    by set="[cn=adminGroup,ou=tvb,dc=test,dc=com]/uniqueMember & user" manage
# 自己可以修改
    by self write
# 登录时候需要读，必须设置该选项
    by * read
```

### 配置数据库的权限

对于客户端请求过来的权限，一般通过Frontend settings段来配置，这也可以理解是全局的ACL，对于后面的ACL会覆盖全局的ACL

```ldif
#
# Configuration database
#
dn: olcDatabase=config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: config
olcAccess: to *
    by set="[cn=adminGroup,ou=tvb,dc=test,dc=com]/uniqueMember & user" manage
    by * none
```

完整的ACL配置部分

> Notes：slapd会根据配置的ACL顺序进行检测，匹配到后就不在继续了，所以在配置时，`what` 部分（选择器范围）要从小到大，这样可以控制范围就正确，正如上面例子，用户只能修改自己密码，属性选择器一旦写在DN选择器下面，将用远不被执行

```ldif
olcAccess: to attrs="userPassword"
    by set="[cn=adminGroup,ou=tvb,dc=test,dc=com]/uniqueMember & user" manage
    by self write
    by * read
olcAccess: to dn.subtree="dc=test,dc=com"
    by set="[cn=adminGroup,ou=tvb,dc=test,dc=com]/uniqueMember & user" manage
    by dn.children="dc=test,dc=com" read
    by anonymous auth
    by * read
```

## Troubleshooting

默认情况下，获取schema是没有权限的，此时需要添加权限使其可以读取schema权限。

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1596103387323.png)

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527344473.png)

***

> **注**：在创建用户和组是需要查看schema权限

***

`slapd.ldif` 文件中添加：

```
dn: olcDatabase=frontend,cn=config
...
olcAccess: to dn.subtree=""
    by * read
or

olcAccess: to dn.subtree="dn: cn=schema,cn=config"
	by * read
```

![image](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/clipboard-1567527344427.png)



**填写配置后重新生成配置文件报类似如下错误**

```
$ slapadd  -n0 -F /etc/openldap/slapd.d -l kerberos.ldif
SYNTAX 1.3.6.1.4.1.1466.115.121.1.26)): empty AttributeDescription
slapadd: could not parse entry (line=1)
_# 6.36% eta none elapsed none spd 18.6 M/s
Closing DB...
```

问题原因：如上配置权限换行方式不可，官网给出的解决方法 <sup><a href="#1">[1]</a></sup>

**Insufficient access (50)**

问题原因：没有权限



> **Reference**
>
> <sup id="1">[1]</sup> [***msg00054***](https://www.openldap.org/lists/openldap-technical/201705/msg00054.html)
>
> <sup id="2">[2]</sup> [***Groups of Groups***](https://www.openldap.org/doc/admin24/access-control.html#8.5.1.%20Groups%20of%20Groups)
>
> <sup id="3">[3]</sup> [***ACLs***](http://www.uvm.edu/~fcs/tools/ACLs.html)
>
> <sup id="4">[4]</sup> [***Controls or what to do after a match***](https://www.openldap.org/faq/data/cache/454.html)
>
> <sup id="5">[5]</sup> [***LDAP Access Control***](https://ubuntu.com/server/docs/service-ldap-access-control)


