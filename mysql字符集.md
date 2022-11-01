# 

[toc]

## 1 MySQL数据库字符集介绍

简单来说，字符集就是一套文字符号及其编码、比较规则的集合，第一个计算机字符集ASCII！

MySQL数据库字符集包括字符集`(character)`和校对规则`(collation)`两个概念。其中，字符集是用来定义MySQL数据字符串的存储方式。而校对规则则是定义比较字符串的方式。

上面命令查看已建立的test数据库语句中 CHARACTER SET latin1即为数据库字符集，而COLLATE latin1_swedish_ci为校对规则，更多内容 见mysql手册第10章。

编译MySQL时，指定字符集了，这样以后建库的时候就直接create database test;

<font style="background:#ffff00;" size=2>二进制安装MySQL，并没有指定字符集，这时字符集默认latin1，此时，需要建立UTF8字符集的库，就需要指定UTF8字符集建库。</font>

```sql
create database test1 default character set utf8 default collate=utf8_general_ci; 
```

### 1.1 MySQL常见字符集介绍

在互联网环境中，使用MySQL时常用的字符集有：

| 常用字符集 | 一个汉字长度（字节） | 说明                                             |
| ---------- | -------------------- | ------------------------------------------------ |
| GBK        | 2                    | 不是国际标准，对中文环境支持很好。               |
| UTF8       | 3                    | 中英文混合环境，建议使用此字符集，用的比较多的。 |
| latin1     | 1                    | MySQL的默认字符集                                |
| utf8mb4    | 4                    | UTF8 Unicode，移动互联网                         |

### 1.2 MySQL如何选择合适的字符集？

1. 如果处理各种各样的文字，发布到不同语言的国家地区，应选Unicode字符集，对MySQL来说就是utf-8（每个汉字三个字节），如果应用需处理英文，仅有少量汉字的utf-8更好。

2. 如果只需支持中文，并且数据两很大，性能要求也高，可选GBK（定长 每个汉字占双字节，英文也占双字节），如果需大量运算，比较排序等，定长字符集更快，性能

3. 处理移动互联网业务，可能需要使用utf8mb4字符集。

**如无特别需求，选择UTF8**

### 1.3 查看MySQL字符集

> **查看当前MySQL系统支持的字符集**

MySQL可支持多种字符集，同一台机器，库或表的不同字段都可以指定不同的字符集。

```sql
mysql> show character set;
+----------+-------------------------+---------------------+--------+
| Charset  | Description             | Default collation   | Maxlen |
+----------+-------------------------+---------------------+--------+
| latin1   | cp1252 West European    | latin1_swedish_ci   |      1 |
| gbk      | GBK Simplified Chinese  | gbk_chinese_ci      |      2 |
| utf8     | UTF-8 Unicode           | utf8_general_ci     |      3 |
| utf8mb4  | UTF-8 Unicode           | utf8mb4_general_ci  |      4 |
+----------+-------------------------+---------------------+--------+
```

> **查看MySQL当前的字符集设置情况**

```sql
mysql> show global variables like '%character_set%';
+---------------------------+-----------------------------------+
| Variable_name             | Value                             |
+---------------------------+-----------------------------------+
| character_set_client      | utf8                              |
| character_set_connection  | utf8                              |
| character_set_database    | utf8                              |
| character_set_filesystem  | binary                            |
| character_set_results     | utf8                              |
| character_set_server      | utf8                              |
| character_set_system      | utf8                              |
| character_sets_dir        | /app/mysql-5.5.54/share/charsets/ |
+---------------------------+-----------------------------------+
```

提示：默认情况下<font color="#f8070d" size=3>`character_set_client`</font><font color="#f8070d" size=3>`character_set_connection`</font><font color="#f8070d" size=3>`character_set_results`</font>三者字符集和系统的字符集一致。即为

```bash
# CentOS 6
[root@Lamp ~]# cat /etc/sysconfig/i18n 
LANG="zh_CN.UTF-8"
SYSFONT="latarcyrheb-sun16"
[root@Lamp ~]# echo $LANG
zh_CN.UTF-8
# CentOS 7
[root@Lamp ~]# echo $LANG
en_US.UTF-8
[root@Lamp ~]# cat /etc/locale.conf 
LANG="en_US.UTF-8"
```


## 2 MySQL插入中文数据乱码深度剖析


### 2.1 MySQL数据库默认设置的字符集是什么？

> **1.查看MySQL默认情况下设置的字符集**

```sql
mysql> show global variables like '%character_set%';
+-----------------------------+---------+
| Variable_name               | Value   |
+-----------------------------+---------+
| character_set_client        | utf8    |   客户端字符集
| character_set_connection    | utf8    |   客户端连接字符集
| character_set_database      | utf8    |   数据库字符集，配置文件指定或建库建表指定
| character_set_filesystem    | binary  |   文件系统字符集
| character_set_results       | utf8    |   返回结果字符集
| character_set_server        | utf8    |   服务器字符集，配置文件指定或建库建表指定
| character_set_system        | utf8    |   系统字符集
+-----------------------------+---------+
```

### 2.2 执行set names gbk到底做了什么

无论Linux系统的字符集是gb2312还是utf8，默认情况插入的数据都是乱码

```sql
mysql> show global variables like '%character_set%';
+---------------------------+--------+
| Variable_name             | Value  |
+---------------------------+--------+
| character_set_client      | gbk    |  
| character_set_connection  | gbk    |  
| character_set_database    | utf8   |  
| character_set_filesystem  | binary |  
| character_set_results     | gbk    |  
| character_set_server      | utf8   |  
| character_set_system      | utf8   |  
+---------------------------+--------+

|  5 |  2 |    3 | 0000-00-00 00:00:00 | 小中僠（国） | 大中僠（国）  
```

执行完set对应的字符集操作，再次插入数据乱码就不乱了<font color="#f8070d" size=2>（注：原有乱码数据不可恢复）</font>

```sql
mysql> select id,title,content from documents;
+----+----------+--------------------------------------------------------+
| id | title    | content                                                |
+----+----------+--------------------------------------------------------+
|  5 | 灏忎腑鍏	| 澶т腑鍏                                                |
|  6 | 小中共	|                                                        |
|  7 | 巨龙中国 | 大気汚染　超大国の苦闘                                 |
|  8 | 巨龙中国 | 大気汚染　超大国の苦闘 ～ＰＭ２.５　沈黙を破る人々～   |  <=字符集不对查询也乱码
+----+----------+--------------------------------------------------------+
```

### 2.3 set names 改变了如下字符串

```bash
character_set_client  
character_set_connection
character_set_results
```

***
**<font color="#0215cd" size=2>提示：<font color="#f8070d"  size=2>`set names gbk`</font> 就是把上面3个桉树改成了 <font color="#f8070d"  size=2>`latin1`</font> 。也就是说 <font color="#f8070d"  size=2>`character_set_client`</font> <font color="#f8070d"  size=2>`character_set_connection`</font>  <font color="#f8070d"  size=2>`character_set_results`</font> 三者的字符集和默认会和linux系统的字符集一致，但是当在mysql中执行 <font color="#f8070d"  size=2>`set names charset`</font> 操作后，这三者都会改变为设置的字符集，但是<font style="background:#ffff00;" size=2>命令修改是临时生效的</font></font>**

***

set names gbk也可以用下面三个命令替代

```sql
set character_set_client=gbk;
set character_set_results=gbk;
set character_set_connection=gbk;
```

此文下回看到要了解下：http://blog.sina.com.cn/s/blog_7c35df9b010122ir.html

## 3 MySQL命令参数 `--default-character-set=latin1` 在做什么？

### 3.1 先查看MySQL的字符集

```sql
mysql -uroot -p111 -S /data/3306/mysql.sock -e 'show variables like "character_set%"'
+---------------------------+---------+
| Variable_name             | Value   |
+---------------------------+---------+
| character_set_client      | utf8    | 
| character_set_connection  | utf8    | 
| character_set_database    | utf8    | 
| character_set_filesystem  | binary  | 
| character_set_results     | utf8    | 
| character_set_server      | utf8    | 
| character_set_system      | utf8    | 
+---------------------------+---------+
```

### 3.2 带参数--default-character-set=latin1登录到MySQL中查看字符集

```sql
[root@multi ~]# mysql -uroot -p111 -S /data/3306/mysql.sock --default-character=latin1

mysql> show variables like '%character_set%';
+---------------------------+---------+
| Variable_name             | Value   |
+---------------------------+---------+
| character_set_client      | latin1  | 
| character_set_connection  | latin1  | 
| character_set_database    | utf8    | 
| character_set_filesystem  | binary  | 
| character_set_results     | latin1  | 
| character_set_server      | utf8    | 
| character_set_system      | utf8    | 
+---------------------------+---------+
```

***
**<font color="#0215cd" size=2>提示：和 <font color="#f8070d" size=3>`set names latin1`</font> 作用一样，MySQL命令后面加字符集也是把上面3个参数改成了 <font color="#f8070d" size=3>`latin1`</font> 。即 <font color="#f8070d" size=3>`character_set_client`</font> ，<font color="#f8070d" size=3>`character_set_connection`</font>，<font color="#f8070d" size=3>`character_set_results`</font> 三者字符集，但是这个登录命令修改字符集也是<font style="background:#ffff00;" size=2>临时生效的</font>。</font>**

***

## 4 确保MySQL数据库插入数据不乱码解决方案

### 4.1统一MySQL数据库客户及服务端字符集

通常MySQL数据库下面几个字符集（客户端和服务端）统一成一个字符集，才能确保插入的中文数据可以正确输出。即 <font color="#f8070d" size=2>`show variables like 'character_set%';`</font> 结果中的字符集设置尽量统一。当然，linux系统的字符集也要尽可能和数据库字符集统一。

> **show variables like 'character_set%';**

其中 <font color="#f8070d" size=2>`character_set_client`</font>，<font color="#f8070d" size=2>`character_set_connection`</font>，<font color="#f8070d" size=2>`character_set_results`</font> 默认情况下采用Linux系统字符集设置，人工登录数据库执行 <font color="#f8070d" size=2>`set names latin1;`</font> 以及MySQL指定字符集登录操作，都是改变了MySQL客户端的<font color="#f8070d" size=2>`client connection results`</font> 三个参数的字符集为 <font color="#f8070d" size=2>`latin`</font>，从而解决了插入中文乱码的问题，这个操作可以通过改变my.cnf配置文件客户端模块的参数来改变，并且永久生效。

> **通过修改my.cnf实现修改MySQL客户端的字符集，配置方法如下**

```bash
[client]
default-character-set=latin1
# 提示无需重启服务，退出重新登录生效，此参数相当于，登录后执行 set names latin1;
# 特别注意：多实例的MySQL客户端默认读/etc/my.cnf，所以指定客户端字符集就在/etc/my.cnf
```

## 5 更改MySQL服务端字符集参数

### 5.1 按如下要求更改my.cnf

```bash
[mysqld]
default-character-set=latin1  # 5.1
character-set-server=latin1   # 5.5
```

强调：以上在[mysqld]下设置的参数会更改下面两个参数的字符集设置

```bash
character_set_database 
character_set_server
```

### 5.2 编译时指定服务端字符集

```bash
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf*_general_ci \
-DEXTRA_CHARSETS=gbk,gb2312,utf8,ascii \
```

## 6 统一MySQL数据库客户服务端字符集总结

1. **客户端字符集设置为 <font color="#f8070d" size=2>`"set names utf8;"`</font>，这样可以确保插入后的中文，不会出现乱码，但是对执行 <font color="#f8070d" size=2>`set names utf8;`</font> 前插入的中文无效，此命令临时生效。**

2. **和设置客户端字符集 <font color="#f8070d" size=2>`"set names utf8"`</font> 命令有相同作用的方法还有，MySQL命令指定utf8字符集参数登录，以及在my.cnf里更改参数实现。**

3. **在MySQL的 <font color="#f8070d" size=2>`my.cnf`</font> 配置文件里 <font color="#f8070d" size=2>`[client]`</font> 模块下添加字符集配置，生效后，相当于命令行 <font color="#f8070d" size=2>`"set names utf8;"` </font>的效果，由于更改的是客户端、连接和返回结果3个字符集，因此无需重启服务就生效。**

4. **在MySQL的 <font color="#f8070d" size=2>`my.cnf`</font> 配置文件里 <font color="#f8070d" size=2>`[mysqld]`</font> 模块下添加字符集配置，生效后，创建数据库和表默认都是这个设置的字符集MySQL5.5和5.1的服务端字符集参数有变化，具体为 <font color="#f8070d" size=2>`character-set-server=utf8`</font> 参数适合5.5，<font color="#f8070d" size=2>`default-character-set=utf8`</font> 参数适合5.1及以前版本。**

## 7 彻底解决MySQL数据库插入中文乱码方案

<font style="background:#ffff00;" size=3>切记：字符集的不一致是数据库乱码的罪魁祸首。</font>


1. 确保以下（客户端和服务端）字符集是一致的，当然，字符集的选择可以有多种。

2. 修改字客户端字符集

```bash
set names gbk;

[mysql]
/app/mysql/bin/mysqlsafe --default-character
```

3. 建库建表的时候要指定和上述设置的字符集相同的字符集，以GBK字符集为例：

```bash
create database test default character set gbk collate gbk_chinese_ci;
```

4. 在数据库中执行sql语句方法
  * 尽量不在MySQL命令行直接插入数据。
  * 可在MySQL中source执行sql文件。
  * sql文件用utf8没有签名。

5. 开发程序的字符集


## 8 生产中如何更改MySQL数据库库表的字符集

### 8.1数据库字符集修改步骤
对于已有的数据库想修改字符集不能直接通过 **<font color="#f8070d" size=2>`"alter database character set *"`</font>** 或 **<font color="#f8070d" size=2>`"alter TableName character set *"`</font>**，这两个命令都没有更新已有数据的字符集，而只是对新创建的表或者数据生效。

已有数据的字符集调整，必须先将数据导出，经过修改字符集后重新导入后才可完成。

步骤如下：


> **1. 导出表结构**

```sql
mysqldump -uroot -p --default-character-set=latin1 -d dbname>table.sql
```

> **2. 编辑表结构语句alltable.sql将所有latin1字符串改成utf8;**

```sql
mysqldump -uroot -p111 -S /data/3306/mysql.sock --default-character-set=utf8 --compact -d test>table.sql
sed -i 's#utf8#gbk#g' table.sql
```

> **3.确保数据不在更新，导出所有数据（不带表结构）**

```bash
mysqldump -uroot -p111 \
--S/data/3306/mysql.sock \
--quick --no-create-info \
--extended-insert \
--default-character-set=utf8 
```

参数说明

**`--quick`**：用于转储大的表，强制mysqldump从服务器一次一行的检索数据而不是检索所有的行，并输出前cache到内存中。

**`--no-create-info`**：不创建create table语句

**`--extended-insert`**：使用包括几个values列表的多行insert语句，这样文件更小,IO也小，导入数据时会非常快。

**`--default-character-set=latin1`** ：按照原有字符集导出数据，这样导出的文件中，所有中文都是可见的，不会保存成乱码

> **4. 修改my.cnf配置调整客户端及服务端字符集，重启生效**

```bash
[client]
default-character-set=latin1
# 提示无需重启服务，退出重新登录生效，此参数相当于，登录后执行 set names latin1;
# 特别注意：多实例的MySQL客户端默认读/etc/my.cnf，所以指定客户端字符集就在/etc/my.cnf
[mysqld]
default-character-set=latin1  # 5.1
character-set-server=latin1   # 5.5
```

> **5. 通过utf8建库**

```sql
create database test default character set utf8;
```

> **6.导入表结构（更改过字符集的表结构）**

```sql
mysql -uroot -p111 -S /data/3306/mysql.sock dbname<table.sql
```

> **7.导入数据**

```sql
mysql -uroot -p -S /data/3306/mysql.sock db<data.sql
```

更改字符集思想
1. 数据库不要更新，导出所有数据。
2. 把导出的数据进行字符集更换（替换表和库）。
3. 修改my.cnf，更改MySQL客户端服务端字符集，重启生效
4. 导入更改过字符集的数据，包括表结构语句，提供服务。
5. SSH客户端，以及程序更改为对应字符集

## 9 校对集collate

collate指的是 字符之间的比较关系！

校对集，依赖于字符集！

校对集，指的是，在某个字符集下，字符的排序关系应该是什么，称之为校对集！

**<font color="#f8070d" size=3>a B c or B a c</font>**

此时，使用 order by对结果排序，看结果：顺序为 a-B-c 忽略了大小写！

```sql
MariaDB [t_t]> select * from test order by name;
+-----------+
|name       |
+-----------+
| a         |
| B         |
| c         |
+-----------+
```

可以被 校对集改变：

利用 show collation; 查看到所有的校对集！

```sql
mysql> show collation;
+---------------------+-----------+------+----------+-----------+---------+
| Collation           | Charset   | Id   | Default  | Compiled  | Sortlen |
+---------------------+-----------+------+----------+-----------+---------+
| latin1_bin          | latin1    |  47  |          | Yes       |      1  |
| latin1_general_ci   | latin1    |  48  |          | Yes       |      1  |
| gbk_chinese_ci      | gbk       |  28  | Yes      | Yes       |      1  |
| utf8_general_ci     | utf8      |  33  | Yes      | Yes       |      1  |
+---------------------+-----------+------+----------+-----------+---------+
```

ci大小写不敏感的

一个字符集下可以存在多个校对集，且有一个是默认的。

再创建一个 utt8_bin的校对集表，在排序：

```sql
create table t( name varchar(2) )engine innodb default charset=utf8 collate=utf8_bin;
insert into t values ('a'),('B'),('c');

MariaDB [t_t]> select * from t order by name;
+------+
| name |
+------+
| B    |
| a    |
| c    |
+------+
```

我们典型的选择：utf8_genreal_ci utf8_unicode_ci

### 校验规则后缀说明：

**<font color="#f8070d" size=2>_bin 二进制编码层面直接比较</font>**

**<font color="#f8070d" size=2>_ci 忽略大小写（大小写不敏感）比较</font>**

**<font color="#f8070d" size=2>_cs 大小写敏感比较</font>**
