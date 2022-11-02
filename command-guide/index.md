# 

ldapsearch:搜索 OpenLDAP 目录树条目

**ldapsearch命令参数说明**

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

查询uid=user1的用户 

`ldapsearch -x -H ldaps://10.0.0.4 -W -D &#34;cn=admin,dc=cylon,dc=com&#34; -b &#34;dc=cylon,dc=com&#34; &#39;(uid=user1)&#39;`

搜索uid=user01 或者 uid=zhangsan的用户

`ldapsearch -x -H ldaps://10.0.0.4 -W -D &#34;cn=admin,dc=cylon,dc=com&#34; -b &#34;dc=cylon,dc=com&#34; &#39;(|(uid=user1)(uid=zhangsan))&#39;`

How do I match 3 attributes?

`(&amp;(objectClass=user)(objectClass=top)(objectClass=person))`

`(|(objectClass=user)(objectClass=top)(objectClass=person))`

匹配uid不为zhangsan的用户

`&#39;(!(uid=zhangsan))&#39;`

查询属性

`&#39;(objectClass=groupOfUniqueNames)&#39;`

ldapapp

-f 文件添加

-S 将因错误调过的条目写入文件

备份ldap配置

```
ldapadd -x -H ldap://etiantian.org -D &#34;cn=admin,dc=etiantian,dc=org&#34; -W -f test.ldif &gt;test.ldif
```

增加配置

```
ldapadd -x -H ldaps://10.0.0.4 -D &#34;cn=admin,dc=cylon,dc=com&#34; -W
```


