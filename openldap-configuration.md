# 

默认情况下，openldap提供的Posixgroup组，实际上并不能很有效的区分组与用户之间的关系。而memberof则可以有效地进行查询用户属于哪些组。

在OpenLDAP启用MemberOf

```
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModuleload: memberof.la
```

在官方指南中看到`olcOverlay` 必须要配置到特定数据库的子条目。即此配置段需要在database配置后面。

> Overlays must be configured as child entries of  a  specific  database.
>
> Reference：[slapd-config](https://www.openldap.org/software/man.cgi?query=slapd-config&sektion=5&apropos=0&manpath=OpenLDAP+2.4-Release#end)

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

配置refint

refint为referential integrity，所引用完整性。是保持属性与所引用的条目保持一致。在使用memberof时，当一个用户被归为memberof组时，会存在memberof的属性。当这个用户组重命名或删除时，refintOverlay将自动修改或删除对应的属性。

```ldif
dn: olcOverlay=refint,olcDatabase={2}hdb,cn=config
objectClass: olcConfig
objectClass: olcOverlayConfig
objectClass: olcRefintConfig
objectClass: top
olcOverlay: refint
olcRefintAttribute: memberof uniqueMember uid cn ## 哪些属性被改动时修改其他所属关系。
```

>Reference
>
>[OpenLDAP Referential Integrity Overlay](https://tylersguides.com/guides/openldap-referential-integrity-overlay/)
>
>[slapo-refint](https://www.openldap.org/software/man.cgi?query=slapo-refint&sektion=5&apropos=0&manpath=OpenLDAP+2.4-Release)
>
>[slapd.overlays](https://www.openldap.org/software/man.cgi?query=slapd.overlays&sektion=5&apropos=0&manpath=OpenLDAP+2.4-Release)

新建一个groupOfUniqueNames用户组，将用户放入组内

```
dn: cn=test_admin,ou=Group,dc=cylon,dc=com
objectClass: groupOfUniqueNames
cn: admin
uniqueMember: uid=user1,ou=People,dc=cylon,dc=com
```

查看用户的memberof属性

```
ldapsearch -x -H ldaps://10.0.0.4 -b dc=cylon,dc=com -D "cn=admin,dc=cylon,dc=com" -W memberOf
```


