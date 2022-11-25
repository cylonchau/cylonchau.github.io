# mysql5.6 innodb_large_prefix引起的一个异常


> phenomenon： Specified key was too long; max key length is 3072 bytes

**在修改一个数据库字段时，字段容量被限制为了表前缀的大小而不是本身的容量大小**

查了一下`innodb_large_prefix`究竟是什么？

动态行格式`DYNAMIC row format `支持最大的索引前缀(3072)。由变量`innodb_large_prefix`进行控制。 

> By default, the index key prefix length limit is 767 bytes. See Section 13.1.13, “CREATE INDEX Statement”. For example, you might hit this limit with a column prefix index of more than 255 characters on a TEXT or VARCHAR column, assuming a utf8mb3 character set and the maximum of 3 bytes for each character. When the innodb_large_prefix configuration option is enabled, the index key prefix length limit is raised to 3072 bytes for InnoDB tables that use the DYNAMIC or COMPRESSED row format.

官方上说在 `utf8mb3`（`most bytes n`）如果设置为 `varchar(255)`时，索引前缀将大于767，可以扩展为3072，但是实际上 varchar的size可以为65535，`这个就限制了整个alter table 的操作`

因为建表是时索引没设置大小，默认是超过255的，后面开启了前缀限制，大小会为3072，此时无法做表修改




>**Reference**
>
>
> [Utf8mb4 / ERROR 1709 (HY000): Index column size too large. The maximum column size is 767 bytes ](https://forums.percona.com/t/utf8mb4-error-1709-hy000-index-column-size-too-large-the-maximum-column-size-is-767-bytes/8336)
>
> [MySQL8 innodb large prefix](https://support.cpanel.net/hc/en-us/articles/4403725847959-MySQL-8-innodb-large-prefix)
>
> [innodb_file_format settings backwards compatible](https://dba.stackexchange.com/questions/233751/are-innodb-large-prefix-and-innodb-file-format-settings-backwards-compatible)
>
> [innodb limits](https://dev.mysql.com/doc/refman/5.7/en/innodb-limits.html)
