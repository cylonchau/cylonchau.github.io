# ch2 understanding openldap - command line




## ldapsearch

## 查询api ldapsearch

### ldapsearch命令参数说明

语法

```
ldapsearch  [options]  filter [attributes]
```

| 参数 | 说 明                                                |
| :--: | ---------------------------------------------------- |
|  -W  | 指定密码，交互式，不需要在命令上写密码               |
|  -w  | 指定密码，需要命令上指定密码                         |
|  -H  | ldapapi                                              |
|  -D  | 所绑定的服务器的DN，如`cn=admin,dc=etiantian,dc=org` |
|  -f  | -f: filename.ldif文件                                |
|  -b  | -b 指定作为查询节点而不是默认的                      |
| -LLL | 以LDIF格式打印响应，不带注释                         |
|  -x  | 简单的认证                                           |

### 简单的搜索

最简单的在查询ldap条目的最简单方法是使用带有 “`-x`” 选项进行简单身份验证，并使用 “`-b`”  指定搜索域。

```bash
$ ldapsearch -x -b &lt;search_base&gt; -H &lt;ldap_host&gt;
```

例如向 10.0.0.3 上openldap服务查询，该命令需要服务器接受匿名身份验证，这将可以查询而无需绑定管理员帐户

```bash
$ ldapsearch -x -b &#34;dc=test,dc=com&#34; -H ldap://10.0.0.3
```

### 使用管理员账户进行搜索

使用管理员帐户进行搜索，必须使用backend配置的 RootDN 并在命令行使用 “`-D`” 选项 和 “`-W`” ，如果要使用非交互式认证，使用选项 “`-w`”

```bash
$ ldapsearch -x -b &lt;search_base&gt; -H &lt;ldap_host&gt; -D &lt;bind_dn&gt; -W
```

例如，上章在安装时配置了RootDN：“ *cn=admin,dc=test,dc=com* ”。如果要使用此帐户执行搜索，可以使用命令

```bash
$ ldapsearch -x -b &#34;dc=test,dc=com&#34; -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -W 
```

### 使用过滤器进行搜索

上述讲到的查询方法，是进行全局查询，会输出所有的条目，这样浪费了时间和资源，在大多数情况下，都希望查询以在特定的目录树中查找特定对象。而 ”过滤器“ filter 就是为了解决这个问题的。

要使用过滤器搜索，需要在 `ldapsearch` 命令的末尾附加 filter 公式：`&lt;object_class&gt;=&lt;object_value&gt;`

```bash
$ ldapsearch &lt;previous_options&gt; &#34;(object_type)=(object_value)&#34; &lt;optional_attributes&gt;
```

例如查找所有对象，可以使用过滤器 `objectclass` 并且值为 ”`*`“ ，

```bash
$ ldapsearch -x -b &lt;search_base&gt; -H &lt;ldap_host&gt; -D &lt;bind_dn&gt; -W &#34;objectclass=*&#34;
```

### 查找特定的用户

如果查询目录树上的所有用户账户，如上一章初始化数据时，初始化的用户账户类为 ”posixAccount“ ，可以通过 ”posixAccount“ 来缩小查询范围

```bash
ldapsearch -x -b &lt;search_base&gt; -H &lt;ldap_host&gt; -D &lt;bind_dn&gt; -W &#34;objectclass=posixAccount&#34;
```

如果每个条目输出的内容多的情况下，也可以输出的属性值来进一步缩小搜索范围，例如只需要 `uid` 与 `gidNumber`

```bash
$ ldapsearch -x -b &#34;dc=test,dc=com&#34; -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111 &#34;objectclass=posixAccount&#34; uid gidNumber
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: objectclass=posixAccount
# requesting: uid gidNumber 
#

# user01, group, test.com
dn: uid=user01,ou=group,dc=test,dc=com
uid: user01
gidNumber: 10001

# cylon, group, test.com
dn: uid=cylon,ou=group,dc=test,dc=com
uid: cylon
gidNumber: 10001

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

### 使用运算符进行筛选搜索

`ldapsearch` 可以使用多个过滤器，使用多个过滤器用运算符 “`AND`” 进行分割，需要注意的一点是，多个过滤器必须将所有条件括在括号中，并在查询的开头加一个 “`&amp;`”  字。

```bash
$ ldapsearch &lt;previous_options&gt; &#34;(&amp;(&lt;condition_1&gt;)(&lt;condition_2&gt;)...)&#34;
```

例如，查找具有 “`objectclass=posixAccount`” 与 “`uid=cylon`” 的条目，您将运行以下查询

```bash
$ ldapsearch -x -b &#34;dc=test,dc=com&#34; -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111 &#34;(&amp;(objectclass=posixAccount)(uid=cylon))&#34;
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: (&amp;(objectclass=posixAccount)(uid=cylon))
# requesting: ALL
#

# cylon, group, test.com
dn: uid=cylon,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/cylon
loginShell: /bin/bash
uid: cylon
cn: cylon
userPassword:: e1NTSEF9MnB2RTRDNnh5OGRrbVcyYUQvZUVvY1Zhamc4QnVqV1c=
uidNumber: 10005
gidNumber: 10001
sn: cylon

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

也可以使用或运算符 ” `OR` “，使多个 “`OR`” 运算符，需要所有条件括在括号内，并使用符号 “ `|` ”  写在过滤条件开头。

```bash
$ ldapsearch &lt;previous_options&gt; &#34;(|(&lt;condition_1&gt;)(&lt;condition_2&gt;)...)&#34;
```

例如，搜索 `uid=user01` 或者 `uid=zhangsan` 的用户

```bash
$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34; &#39;(|(uid=user01)(uid=zhangsan))&#39;
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: (|(uid=user01)(uid=zhangsan))
# requesting: ALL
#

# user01, group, test.com
dn: uid=user01,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/user01
loginShell: /bin/bash
uid: user01
cn: user01
uidNumber: 10004
gidNumber: 10001
userPassword:: e1NTSEF9aEpwSUlWeGoxcVM5ZzA1cVVsZ0crbzdNTzE0RVhiRlE=
sn: user01
givenName: user01

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

也可以使用 “非” 运算符 ” `!` “，使多个 “`!`” 运算符，需要所有条件括在括号内，并使用符号 “ `!` ”  写在过滤条件开头。

```bash
$ ldapsearch &lt;previous_options&gt; &#34;(!(&lt;condition_1&gt;)(&lt;condition_2&gt;)...)&#34;
```

例如，匹配 `uid` 不为 `zhangsan` 的用户

```bash
$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34; &#39;(!(uid=zhangsan))&#39;
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: (!(uid=user01))
# requesting: ALL
#

# test.com
dn: dc=test,dc=com
objectClass: top
objectClass: organizationalUnit
objectClass: extensibleObject
description: US Organization
ou: people
dc: test

# group, test.com
dn: ou=group,dc=test,dc=com
objectClass: organizationalUnit
ou: group

# tech, group, test.com
dn: cn=tech,ou=group,dc=test,dc=com
objectClass: posixGroup
gidNumber: 10001
cn: tech
memberUid: user01
memberUid: cylon

# cylon, group, test.com
dn: uid=cylon,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/cylon
loginShell: /bin/bash
uid: cylon
cn: cylon
userPassword:: e1NTSEF9MnB2RTRDNnh5OGRrbVcyYUQvZUVvY1Zhamc4QnVqV1c=
uidNumber: 10005
gidNumber: 10001
sn: cylon

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 4
```

### 使用多个过滤条件

多个过滤条件可以使用 `()` 括起来所有的过滤条件

```bash
$ ldapsearch &lt;previous_options&gt; &#34;(|(&lt;condition_1&gt;)(&lt;condition_2&gt;)...)&#34;
```

### 使用通配符进行搜索

除了运算符，还有一种高校过滤器就是通配符 “`*`” 这使得 `ldapsearch` 过滤器拥有一些正则表达式功能

使用通配符，只需要对应的查询条件字符串结尾或开头加上 “`*`” 即可

```bash
$ ldapsearch &lt;previous_options&gt; &#34;(object_type)=*(object_value)&#34;

$ ldapsearch &lt;previous_options&gt; &#34;(object_type)=(object_value)*&#34;
```

例如查询 `uid` 以 “`c`” 开头的用户

```bash
$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34; &#39;(uid=c*)&#39;
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: (uid=c*)
# requesting: ALL
#

# cylon, group, test.com
dn: uid=cylon,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/cylon
loginShell: /bin/bash
uid: cylon
cn: cylon
userPassword:: e1NTSEF9MnB2RTRDNnh5OGRrbVcyYUQvZUVvY1Zhamc4QnVqV1c=
uidNumber: 10005
gidNumber: 10001
sn: cylon

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

### 扩展过滤器

可以由符合 “ `:` ”隔的其他过滤器，例如区分大小写与不区分大小写

- `CaseIgnoreMatch ` 不区分大小写
- `cn:caseExactMatch` 区分大小写
- 

```bash
$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34;  &#34;cn:CaseIgnoreMatch:=USER01&#34;
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: cn:CaseIgnoreMatch:=USER01
# requesting: ALL
#

# user01, group, test.com
dn: uid=user01,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/user01
loginShell: /bin/bash
uid: user01
cn: user01
uidNumber: 10004
gidNumber: 10001
userPassword:: e1NTSEF9aEpwSUlWeGoxcVM5ZzA1cVVsZ0crbzdNTzE0RVhiRlE=
sn: user01
givenName: user01

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

例如类似值搜索

```bash
$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34;  &#34;givenName~=usr01&#34;
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: givenName~=usr011
# requesting: ALL
#

# user01, group, test.com
dn: uid=user01,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/user01
loginShell: /bin/bash
uid: user01
cn: user01
uidNumber: 10004
gidNumber: 10001
userPassword:: e1NTSEF9aEpwSUlWeGoxcVM5ZzA1cVVsZ0crbzdNTzE0RVhiRlE=
sn: user01
givenName: user01

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

### 查找openldap服务配置

`ldapsearch` 命令有一种高级用法是查询 `slapd` 服务的配置。要进行这种搜索，必须使用选项 “`-Y`” 并指定 关键字 “`EXTERNAL`” 作为身份验证机制。

```bash
$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config 
```

例如

```bash
$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config
SASL/EXTERNAL authentication started
SASL username: gidNumber=0&#43;uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
# extended LDIF
#
# LDAPv3
# base &lt;cn=config&gt; with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# config
dn: cn=config
objectClass: olcGlobal
cn: config
olcArgsFile: /var/run/openldap/slapd.args
olcDisallows: bind_anon
olcLogLevel: stats
olcPidFile: /var/run/openldap/slapd.pid
olcTLSCACertificatePath: /etc/openldap/certs
olcTLSCertificateFile: &#34;OpenLDAP Server&#34;
olcTLSCertificateKeyFile: /etc/openldap/certs/password

# schema, config
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema
olcObjectIdentifier: OLcfg 1.3.6.1.4.1.4203.1.12.2

...

# {1}collective, schema, config
dn: cn={1}collective,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: {1}collective

...

# {-1}frontend, config
dn: olcDatabase={-1}frontend,cn=config
objectClass: olcDatabaseConfig
objectClass: olcFrontendConfig
olcDatabase: {-1}frontend

# {0}config, config
dn: olcDatabase={0}config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: {0}config
olcAccess: {0}to attrs=userPassword,shadowLastChange   by dn.children=&#34;cn=admi
 n,dc=test,dc=com&#34; write   by anonymous auth   by self write   by * none
olcAccess: {1}to * by dn.base=&#34;gidNumber=0&#43;uidNumber=0,cn=peercred,cn=external
 ,cn=auth&#34; manage   by group.exact=&#34;cn=configadmin,ou=admin,dc=seal,dc=com&#34; wr
 ite   by * none

# {1}monitor, config
dn: olcDatabase={1}monitor,cn=config
objectClass: olcDatabaseConfig
olcDatabase: {1}monitor
olcAccess: {0}to * by dn.base=&#34;gidNumber=0&#43;uidNumber=0,cn=peercred,cn=external
 ,cn=auth&#34; read by dn.base=&#34;cn=Manager,dc=my-domain,dc=com&#34; read by * none

# {2}mdb, config
dn: olcDatabase={2}mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: {2}mdb
olcDbDirectory: /var/lib/ldap
olcSuffix: dc=test,dc=com
olcRootDN: cn=admin,dc=test,dc=com
olcRootPW: {SSHA}xU9xFym/s7rawpmzpsYE&#43;Q1qPsVPOwDw
olcDbCheckpoint: 1024 10
olcDbIndex: objectClass eq,pres
olcDbIndex: uid,ou,cn,mail,surname,givenname eq,pres,sub

# search result
search: 2
result: 0 Success

# numResponses: 20
# numEntries: 19
```

&gt; Notes：这种查询命令需要直接在slapd服务对应节点上运行。

这类搜索也可以使用过滤器，例如指定搜索数据库的配置

```bash
$ ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config &#34;(objectclass=olcDatabaseConfig)&#34;
```

## 更新API ldapmodify

`ldapmodify` 有两个参数来指定如何修改数据：

- `replace` 要修改的字段
- `changetype: modify` 模式为修改模式
- `dn` 对那个条目进行搜索，RootDN的后缀，对于每个条目例如，user 为 `uid=&lt;&gt;,ou=&lt;&gt;,dc=test,dc=com`

例如下面命令是将 cylon用户的uid更改为10010

```bash
$ cat &lt;&lt; EOF | ldapmodify -r  -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=cylon,ou=Group,dc=test,dc=com
changetype: modify
replace: uidNumber
uidNumber: 10010
EOF
```

## 删除API ldapdelete

删除API与更新API类似，不过内容只需要一个 `dn` （这个dn是隐式的，不用单独声明字段），例如删除用户cylon

```bash
cat &lt;&lt; EOF | ldapdelete -r  -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
uid=cylon,ou=Group,dc=test,dc=com
EOF
```

使用 `ldapmodify` 删除条目，只要吧 `changetype: delete` 在加上显式声明的 `dn` 也可以删除条目，例如

```bash
$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34; uid=cylon  
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: uid=cylon
# requesting: ALL
#

# cylon, group, test.com
dn: uid=cylon,ou=group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/cylon
loginShell: /bin/bash
uid: cylon
cn: cylon
userPassword:: e1NTSEF9MnB2RTRDNnh5OGRrbVcyYUQvZUVvY1Zhamc4QnVqV1c=
uidNumber: 10005
gidNumber: 10001
sn: cylon

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1



$ cat &lt;&lt; EOF | ldapmodify -r  -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=cylon,ou=Group,dc=test,dc=com
changetype: delete
EOF

deleting entry &#34;uid=cylon,ou=Group,dc=test,dc=com&#34;

$ ldapsearch -x -H ldap://10.0.0.3 -w 111 -D &#34;cn=admin,dc=test,dc=com&#34; -b &#34;dc=test,dc=com&#34; uid=cylon  
# extended LDIF
#
# LDAPv3
# base &lt;dc=test,dc=com&gt; with scope subtree
# filter: uid=cylon
# requesting: ALL
#

# search result
search: 2
result: 0 Success

# numResponses: 1
```

## 插入API ldapadd

`ldapapp` 使用起来比较复杂，在添加时，区分与RootDN，子条目，并且属性相关都需要配置对

下列时增加一个用户

```bash
cat &lt;&lt; EOF | ldapadd -x -H ldap://10.0.0.3 -D &#34;cn=admin,dc=test,dc=com&#34; -w 111
dn: uid=cylon,ou=Group,dc=test,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
homeDirectory: /home/cylon
loginShell: /bin/bash
uid: cylon
cn: cylon
userPassword: {SSHA}2pvE4C6xy8dkmW2aD/eEocVajg8BujWW
uidNumber: 10005
gidNumber: 10001
sn: cylon
EOF
```



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [***Managing Entries ldapmodify and ldapdelete***](https://docs.oracle.com/cd/E19693-01/819-0995/6n3cq3apv/index.html)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [***How To Search LDAP using ldapsearch***](https://devconnected.com/how-to-search-ldap-using-ldapsearch-examples/)


