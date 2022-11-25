# 使用ldap客户端创建zimbra ldap用户的格式


命令如下

```ldif
cat << EOF | ldapadd -x -W -H ldap://:389 -D "uid=zimbra,cn=admins,cn=zimbra"
dn: uid=jak1,ou=people,dc=mail,dc=xxxxx2021,dc=com
zimbraAccountStatus: active
displayName: jak1
givenName: jak1
sn: jak1
zimbraMailStatus: enabled
objectClass: inetOrgPerson
objectClass: zimbraAccount
objectClass: amavisAccount
zimbraId: e2214a66-3ga2-4241-9223-44f222ce0522
zimbraCreateTimestamp: 20191102062818.876Z
zimbraMailHost: mail.xxxx2021.com
zimbraMailTransport: lmtp:mail.xxxx2021.com:7025
zimbraMailDeliveryAddress: scott8@mail.xxxx2021.com
mail: jak1@mail.xxxx2021.com
cn: jak1
uid: jak1
userPassword:: e1NTSEE1MTJ9ZzBzZGlXRlBjbDQxa2xmZ200YXc1ZkJzSGQzVXNBdVBydUlKRnZ
 LTExYby9HWXBoUkNTMzZYMEx5VnpCZUJPMGJNTCtTV2IwSnhkaHdudTViR0c1bTJabFVhU3R1N1J3
EOF

```

- zimbraAccountStatus 为账户设置中的状态
- zimbraId 唯一的值
- givenName 姓
- displayName 显示名字

![image-20221125182349029](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221125182349029.png)



```bash
ldapsearch -LLL -w 99tJFkhVfn -H ldap://172.31.110.115:389 -D "uid=zimbra,cn=admins,cn=zimbra"|less
```



