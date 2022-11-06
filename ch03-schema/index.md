# ch3 understand schema in openldap


## 什么是schema

schema又称为数据模型，是openldap中用于来指定一个条目所包含的对象类(objectClass)以及每个对象类所包含的属性值(attribute)，其中属性值又包含必要属性和可选属性。

&gt; Notes：拥有schema的数据代表该数据是结构化数据，无论他是什么格式，甚至于是一个连续的字符串

## 如何理解schema

不管是在学习OpenLDAP时还是学习数据库时，都会遇到一个很迷糊的Schema的概念。

在数据库中，对数据库的设计可以称之为schema。即schema约束了数据库的设计结构，并提供了整个数据库的描述。schema仅展示数据库的设计，如表字段的类型，表与表之间的关联。

![image-20210226195759348](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20210226195759348.png)

在ldap中schema与database中的schema一样，如列出的schema中，这些代表了对应的ldap结构的设计。

[what-is-a-schema](https://afteracademy.com/blog/what-is-a-schema)

[schema overview](https://docs.oracle.com/cd/B14099_19/idmanage.1012/b15883/schema_overview001.htm)

```
olcObjectClasses: ( 0.9.2342.19200300.100.4.19 NAME &#39;simpleSecurityObject&#39;
  DESC &#39;RFC1274: simple security object&#39;
  SUP top AUXILIARY
  MUST userPassword ) # 必须包含的属性
#
# RFC 1274 &#43; RFC 2247
olcAttributeTypes: ( 0.9.2342.19200300.100.1.25
  NAME ( &#39;dc&#39; &#39;domainComponent&#39; ) # 表示属性名称
  DESC &#39;RFC1274/2247: domain component&#39;
  EQUALITY caseIgnoreIA5Match # 相等性匹配
  SUBSTR caseIgnoreIA5SubstringsMatch  # 字符串匹配
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE ) # 表示数据类型
```

## schema包含什么

在openldap的schema中，有四个重要的元素：

- Objectclass：Objectclass定义了一个类别，这个类别会被不同的目录（在LDAP中就是一个Entry）用到，它说明了该目录应该有哪些属性，哪些属性是必须的，哪些又是可选的。一个objectclass的定义包括名称（NAME），说明（DESC），类型（STRUCTURAL或AUXILARY），表示是结构型的还是辅助型的），必须属性（MUST），可选属性（MAY）等信息。

- Attribute：Attribute就是一个上面Objectclass中可能包含的属性，对其的定义包括名称，数据类型，单值还是多值以及匹配规则等。后面用具体的例子来说明。

- Syntax：syntax是LDAP中的 “语法”，其实就是LDAP中会用到的数据类型和数据约束，这个语法是遵从X.500中数据约束的定义的。其定义需要有一个ID（遵从X.500）以及说明（DESP）

- Matching Rules：是用来指定某属性的匹配规则，实际上就是定义一个特殊的Syntax的别名，让LDAP服务器可以识别，并对定义的属性进行匹配。

###  Schema Files

OpenLDAP随附了一些schema，在`/usr/local/etc/openldap/ chema` 目录中

| 文件                 | 说明                                                      |
| -------------------- | --------------------------------------------------------- |
| core.schema          | OpenLDAP *core* (required)                                |
| cosine.schema        | Cosine 和 Internet X.500                                  |
| inetorgperson.schema | 在添加账号时,额外的objectClass.                           |
| misc.schema          |                                                           |
| nis.schema           | 网络信息服务(FYI),也是一种集中账号管理实现.               |
| openldap.schema      | OpenLDAP Project(experimental)                            |
| dyngroup.schema      | 定义动态组时使用的schema,包括要使用sudo.schema时,也需要它 |
| ppolicy.schema       | 需要做密码策略时,需要导入此schema                         |
| sudo.schema          | 定义sudo规则                                              |
