# ch7 understanding openldap configuration - Backup and Restore


## Overview

本章基于openldap 2.4&#43;版本进行，主要讲解 openldap 的两种备份方法：备份openldap `backend-database` 文件，另一种方式为导出 `LDIF` 目录方式

## Backup

备份部分将分为两种方式：使用基于 `slapcat` 导出目录文件方式，与直接备份数据库文件方式。

`slapcat` 是可用于导出 slapd 数据库中数据为LDAP交换格式的命令行工具，它可以导出 slapd 的配置也可以导出 slapd的数据。

`slapcat` 使用起来很简单，参数也是与 openldap 其他命令参数类似，

|      参数       | 说 明                                                        |
| :-------------: | ------------------------------------------------------------ |
|   -a *filter*   | 只导出与过滤器声明条件相匹配的数据&lt;br&gt;例如：`slapcat -a &#34;(!(entryDN:dnSubtreeMatch:=ou=People,dc=example,dc=com))&#34;` |
|   -b *suffix*   | 将只导出-b指定DN域内数据，-b 不能与 -n 同时使用              |
|       -c        | 忽略错误                                                     |
|       -f        | -f 后接的文件将替代默认的配置文件，通常情况下备份在slapd本机执行可以不使用该参数 |
|       -F        | 指定配置目录，-F比-f优先级高，同时指定生效为-F，也就是导出的目录 |
|       -g        | 导出时不使用从属关系，仅仅为指定的数据库才会被导出           |
|       -H        | 连接 slapd 服务的地址                                        |
|       -l        | 输出的文件，默认slapcat是将内容输出到标准输出stdout中        |
| -s *subtree-dn* | 仅导出符合dn子树的条目                                       |

例如下列命令用于备份配置文件的

```bash
$ slapcat -n 0 -l config.ldif
```

&gt; Notes：slapd中，配置（0）永远是第一个数据库，跟着的就是在配置中指定的数据库，例如 {0}hdb 将表示1，{1}hdb 则是2

下列命令用于导出为LDIF格式数据

```bash
$ slapcat -n 1 -l data.ldif
```

## Restore

恢复配置时，建议直接删除原有配置，而后重新生成新的即可，下表为常见的操作系统openldap配置目录

| OS                     | Configuration Directory         |
| ---------------------- | ------------------------------- |
| Debian/Ubuntu          | /etc/ldap/slapd.d               |
| RHEL/CentOS            | /etc/openldap/slapd.d           |
| SuSE (Uses slapd.conf) | /etc/openldap/slapd.d           |
| FreeBSD                | /usr/local/etc/openldap/slapd.d |

下列命令是恢复配置的命令，`-l` 用户指定导出的ldif文件，`-F` 为配置目录，必须存在

```bash
$ slapadd -n 0 -F /etc/openldap/slapd.d -l /backups/config.ldif
```

因为运行 slapd 进程的用户为 ldap，所以新配置目录的必须拥有权限

```bash
$ chown -R ldap:ldap /etc/openldap/slapd.d
```

下列命令是恢复数据的命令。其实就是重新添加回去

```bash
$ slapadd -n 1 -F /config/directory/slapd.d -l /backups/data.ldif -w
```

