# ch8 understanding openldap configuration - advanced application


## memberOf

默认情况下，openldap提供的Posixgroup组，实际上并不能很有效的区分组与用户之间的关系。而 `memberOf` 则可以有效地检索用户与组的关系

### 在OpenLDAP配置MemberOf模块

**步骤一**：可以检查在允许的slapd服务是否已经启用该模块

```bash
$ slapcat -n 0 | grep olcModuleLoad
```

对于新部署的服务，可以按照如下方式添加

```
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModuleload: memberof.la
```

可以在线更改一个正在运行的slapd服务，使其加载 `memberOf` 模块，需要主义对应的 `module{0}` 是否正确

```bash
cat &lt;&lt; EOF | ldapmodify -Q -Y EXTERNAL -H ldapi:///
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: memberof.la
EOF
```

**步骤二**：配置overlay

在官方指南中看到`olcOverlay` 必须要配置到特定数据库的子条目。即此配置段需要在database配置后面。

&gt; Overlays must be configured as child entries of  a  specific  database. &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;
&gt;

```
dn: olcOverlay={0}memberof,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcMemberOf
objectClass: olcOverlayConfig
objectClass: top
olcOverlay: memberof
olcMemberOfDangling: ignore
olcMemberOfRefInt: TRUE
olcMemberOfGroupOC: groupOfUniqueNames
olcMemberOfMemberAD: uniqueMember
olcMemberOfMemberOfAD: memberOf
```

**步骤三**：配置refint

refint为 (***referential integrity***)，所引用完整性。是保持属性与所引用的条目保持一致。在使用 `memberOf` 时，当一个用户被归为 `memberOf` 组时，会存在 `memberOf` 的属性。当这个用户组重命名或删除时，`refintOverlay` 将自动修改或删除对应的属性。

```ldif
dn: olcOverlay=refint,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: refint
olcRefintAttribute: memberOf uniqueMember uid cn ## 哪些属性被改动时修改其他所属关系。
```

**步骤四**：配置memberOf

新建一个groupOfUniqueNames用户组，将用户放入组内

```
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
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

创建测试用户

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=syncUser,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
uid: syncUser
cn: syncUser
uidNumber: 10006
gidNumber: 10002
userPassword: {SSHA}QnB7dO98&#43;hoCUgiaAYaiJWnDzlhn2Tn6
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
userPassword: {SSHA}QnB7dO98&#43;hoCUgiaAYaiJWnDzlhn2Tn6
sn: searchUser
givenName: searchUser
memberOf: cn=dirGroup,ou=tvb,dc=test,dc=com

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
userPassword: {SSHA}QnB7dO98&#43;hoCUgiaAYaiJWnDzlhn2Tn6
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
userPassword: {SSHA}QnB7dO98&#43;hoCUgiaAYaiJWnDzlhn2Tn6
sn: admin
givenName: admin
memberOf: cn=adminGroup,ou=tvb,dc=test,dc=com
EOF
```

查看用户的memberof属性

```
$ ldapsearch -x -H ldaps://10.0.0.4 -b dc=test,dc=com -D &#34;cn=admin,dc=test,dc=com&#34; -w 111 memberOf
```

## Referential Integrity

引用完整 (***Referential Integrity***) 模块是指在两个属性互相引用时，如果自动修改或删除了一端的值，将保持另一端也能够保持一致性，删除或更新。

例如下面示例，`uid=eve,ou=users,dc=tvb,dc=com` 是一个管理员用户，`uid=mallory,ou=users,dc=tvb,dc=com` 是一个普通用户，普通用户mallory的管理员是eve用户，当eve用户修改名称或者被删除时，mallory对应的 `manager` 也应该保持更新或删除

```ldif
dn: uid=eve,ou=users,dc=tvb,dc=com
objectClass: posixAccount
objectClass: shadowAccount
objectClass: inetOrgPerson
cn: Eve
sn: Eavesdropper
uid: eve
uidNumber: 5000
gidNumber: 5000
homeDirectory: /home/eve
loginShell: /bin/sh
gecos: Eve Eavesdropper

dn: uid=mallory,ou=users,dc=tvb,dc=com
objectClass: posixAccount
objectClass: shadowAccount
objectClass: inetOrgPerson
cn: Mallory
sn: Malicious
uid: eve
uidNumber: 5001
gidNumber: 5000
homeDirectory: /home/mallory
loginShell: /bin/sh
gecos: Mallory Malicious
manager: uid=eve,ou=users,dc=tvb,dc=com
```

### 在OpenLDAP配置Referential Integrity模块

**步骤一**：可以检查在允许的slapd服务是否已经启用该模块

```bash
$ slapcat -n 0 | grep olcModuleLoad
```

可以在线更改一个正在运行的slapd服务，使其加载 `refint` 模块，需要主义对应的 `module{0}` 是否正确

```bash
cat &lt;&lt; EOF | ldapmodify -Q -Y EXTERNAL -H ldapi:///
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: refint.la
EOF
```

另外如果 模块目录不为标准的路径，也需要配置 `olcModulePath` ，当然这个参数只能指定一次

```ldif
dn: cn=module,cn=config
cn: module 
objectClass: olcModuleList
olcModulePath: /opt/openldap-current/libexec/openldap
olcModuleLoad: refint.la
```

下表是关于各操作系统内置 openldap 服务模块的标准路径

| OS               | PATH                        |
| ---------------- | --------------------------- |
| CentOS 7         | /usr/lib64/openldap         |
| openSUSE         | /usr/lib64/openldap         |
| Debian (Stretch) | /usr/lib/ldap               |
| Source (Default) | /usr/local/libexec/openldap |

由于overlay的特性，需要指定为database的子模块

```bash
$ cat &lt;&lt; EOF | ldapadd -Y EXTERNAL -H ldapi:///
dn: olcOverlay=refint,olcDatabase={1}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
olcOverlay: refint
olcRefintAttribute: manager secretary
olcRefintNothing: cn=config
EOF
```

这里需要注意的属性为：

- `olcDatabase={1}mdb` 数据库这里需要指定要配置的数据库
- `olcRefintAttribute` 这个属性指明了两个条目间关联的属性
- `olcRefintNothing` 可选参数，因为两端需要关联时，例如memberOf，组对象至少保留一个 `memberOf` 属性，当删除最后一个关联用户时，会提示非法，这个参数就是在删除最后一个用户时，指定一个占位符的该属性

## Password Policy

openldap中 Password Policy 是指用户的密码策略，如过期时间，密码安全最低要求等

### 在OpenLDAP配置ppolicy模块

可以在线更改一个正在运行的slapd服务，使其加载 `ppolicy` 模块，需要主义对应的 `module{0}` 是否正确

```bash
cat &lt;&lt; EOF | ldapmodify -Q -Y EXTERNAL -H ldapi:///
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: ppolicy.la
EOF
```

在加载完成后，需要叠加到database上才可以生效，如下列配置

```ldif
dn: olcOverlay=ppolicy,olcDatabase={1}mdb,cn=config
objectClass: olcPPolicyConfig
olcOverlay: ppolicy
olcPPolicyDefault: cn=default,ou=tvb,dc=test,dc=com
olcPPolicyUseLockout: FALSE
olcPPolicyHashCleartext: TRUE
```

这里需要注意的一点是，`olcPPolicyDefault` 是指定的一个默认密码策略，即没有为用户配置密码策略时，将使用这个默认策略

### 为用户配置默认策略

如果需要使用模块 `ppolicy` ，必须做下述三个步骤其一才可满足为用户密码提供一个管理策略

- 需要创建一个默认策略
- 为用户 ldift 添加属性  `pwdPolicySubentry` ，这可以为不同的用户设置不同的密码策略
- 属性 `pwdPolicy` 可以作为用户 ldif 中被多次引用，这代表可以设置多种不同的策略，这种比较繁琐 

**场景一**：创建一个默认策略

例如在配置ppolicy的overlay时，默认策略名称填写了 `cn=default,ou=tvb,dc=test,dc=com` 那么需要用这个DN创建一个默认策略，而创建DN需要按层级创建，顾需要创建 `ou=plicies` 

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: ou=tvb,dc=test,dc=com
objectClass: organizationalUnit
ou: policies
EOF
```

接下来创建默认的密码策略

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: cn=default,ou=tvb,dc=test,dc=com
objectClass: pwdPolicy
objectClass: organizationalRole
cn: default
pwdAttribute: userPassword
pwdMinLength: 3
pwdCheckQuality: 2
EOF
```

下表是对于一些密码策略的属性说明

| 属性               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| pwdAttribute       | 指定那个字段是密码                                           |
| pwdMinAge          | 多少秒之间必须经过更改密码，默认0                            |
| pwdMaxAge          | 密码最大期限，如果密码过期用户将被拒绝登录。 默认的是密码永不过期。 |
| pwdInHistory       | 存储多少个历史旧密码，例如更改密码时提示与历史使用密码类似。 默认为0，即可以重复使用旧密码 |
| pwdMinLength       | 密码的最小的长度。 默认没有长度的要求。 这个属性将与 `pwdCheckQuality` 同时生效，当  `pwdCheckQuality` 为0，则长度限制不生效 |
| pwdCheckQuality    | 如何对用户长度进行检查。 默认0不检查，&lt;br&gt;2 ：总是强制执行的质量检查；如果它不能检查它，密码就会被拒绝。&lt;br&gt;1 ：可被接受的密码 |
| pwdMaxFailure      | 密码错误几次用户被锁定，默认0 代表不会被锁定，他们是锁着的。 为了对此采取影响，  必须关联属性 `pwdLockout=TRUE` |
| pwdLockout         | `pwdLockout=TRUE` 时  pwdMaxFailure 会生效，否则忽略  `pwdMaxFailure` 配置 |
| pwdLockoutDuration | 账户自动解锁的时间                                           |
| pwdMustChange      | 这是当管理员创建用户后，第一次登陆时需要必须修改密码         |
| pwdPolicySubentry  | 表示设置密码的策略。 DN对应适用密码政策的条目。 默认策略由  `olcPPolicyDefault`  属性的配置。如果不存在，会使用 `olcPPolicyDefault` ，当存在时会覆盖 `olcPPolicyDefault` |

如果需要上述的策略，需要引入对象 `objectClass: pwdPolicy` 例如修改一个用户的密码策略

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=searchUser,ou=tvb,dc=test,dc=com
changetype: modify
add: pwdMinLength
pwdMinLength: 4
-
add: pwdCheckQuality
pwdCheckQuality: 2
-
replace: pwdPolicySubentry
pwdPolicySubentry: cn=example2,ou=tvb,dc=tvb,dc=com
EOF
```

而上述属性  `pwdPolicySubentry`  代表那个策略适用于那个条目。 如果不存在则默认策略生效。如果需要引入存在的密码策略可以使用该属性进行应用。

例如，下述是创建两天密码策略

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: cn=example1,ou=tvb,dc=test,dc=com
cn: example1
objectClass: organizationalRole
objectClass: pwdPolicy
pwdAttribute: userPassword
pwdMinLength: 5
pwdCheckQuality: 2

dn: cn=example2,ou=tvb,dc=test,dc=com
cn: example2
objectClass: organizationalRole
objectClass: pwdPolicy
pwdAttribute: userPassword
pwdMinLength: 6
pwdCheckQuality: 2
pwdMaxFailure: 3
pwdLockout: TRUE
pwdLockoutDuration: 10
pwdInHistory: 5
EOF
```

当需要创建一个用户引用现有的一组密码策略时，可以对用户添加属性 ` pwdPolicySubentry`，而不是重复的设置每条策略，例如下面示例

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.10 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=admin1,ou=tvb,dc=test,dc=com
changetype: modify
add: pwdPolicySubentry
pwdPolicySubentry: cn=example1,ou=tvb,dc=tvb,dc=com

dn: uid=searchUser,ou=tvb,dc=test,dc=com
changetype: modify
add: pwdPolicySubentry
pwdPolicySubentry: cn=example2,ou=tvb,dc=tvb,dc=com
EOF
```



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [***slapd-config***](https://www.openldap.org/software/man.cgi?query=slapd-config&amp;sektion=5&amp;apropos=0&amp;manpath=OpenLDAP&#43;2.4-Release#OVERLAYS)

