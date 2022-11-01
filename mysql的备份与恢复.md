# 

[TOC]

### 8.1 备份数据库的意义

运维工作到底是什么工作，到底是做什么？

运维工作简单的概括就两件事：

一是保护公司的数据；二是网站7*24小时提供服务。

那么对数据丢失一部分和网站7*24小时提供服务那个更重要呢？

都很重要，只是说相比哪个更为重要？这个具体要看业务个公司。例如：银行、金融行业，数据是最重要的，一条都不能丢，可能宕机停机影响就没那么大。百度搜索，腾讯qq聊天记录丢失了几万条数据，都不算啥。

对于数据来讲，数据最核心的就是数据库数据。

http://oldboy.blog.51cto.com/2561410/830451

### 8.2 备份单个数据库练习多种参数的使用

MySQL数据库自带了一个很好用的备份命令，就是mysqldump，它的基本使用如下：

```sql
mysqldump -u UserName -p PassWord dbName > backName.sql
```

#### 8.2.1 备份库

```sql
mysqldump -S /data/3306/mysql.sock -uroot -p test>mysql.sql   
```

检查备份结果 

```sql
[root@multi 3306]# egrep -v  "#|\*|--|^$" ./mysql.sql
DROP TABLE IF EXISTS `test1`;
CREATE TABLE `test1` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `num1` varchar(20) NOT NULL,
  `num2` varchar(20) NOT NULL,
  `num3` varchar(20) NOT NULL,
  `num4` int(11) NOT NULL DEFAULT '0' COMMENT 'test1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2000001 DEFAULT CHARSET=utf8;
LOCK TABLES `test1` WRITE;
INSERT INTO `test1` VALUES (1,'1455577','9779520','4530868',0),
```

***

<font color="#0215cd" size=2> 注：因为导出时的格式没有加字符集，一般恢复到数据库里会正常，只是系统外查看不正常而已。另外，insert是批量插入的方式，这样在恢复时效率很高。</font>

***

根据查看的结果，我们看到了已备份的表结构语句及插入的数据整合的sql语句，但是中文数据乱码了。

#### 8.2.2 设置字符集参数备份解决乱码问题

查看备份前数据库客户端及服务器端的字符集设置

```sql
mysql> show variables like "character%"
+---------------------------+-----------------------------------+
| Variable_name             | Value                     		|
+---------------------------+-----------------------------------+
| character_set_client   	| utf8                              |
| character_set_connection	| utf8                            	|
| character_set_database   	| utf8                            	|
| character_set_filesystem	| binary                          	|
| character_set_results    	| utf8                            	|
| character_set_server     	| utf8                            	|
| character_set_system     	| utf8                            	|
| character_sets_dir       	| /app/mysql-5.5.54/share/charsets/ |
+---------------------------+-----------------------------------+
```

指定对应的字符集备份，这里为<font color="#f8070d" size=3>`--default-character-set=utf8`</font>

```
mysqldump -uroot -p111 --default-character-set=utf8 t1 > t.sql -S /data/3306/mysql.sock
```

#### 8.2.3 备份时加`-B`参数，增加创建库与选择库

```sql
 -- Current Database: `t1`
 CREATE DATABASE /*!32312 IF NOT EXISTS*/ `t1` /*!40100|  
 USE `t1`;
```

#### 8.2.4 优化配置文件大小减少输出注释（debug调试）

利用<font color="#f8070d" size=3>`mysqldump`</font>的<font color="#f8070d" size=3>`--compact`</font>参数优化下备份结果

```bash
[root@multi 3306]# mysqldump -uroot -p111 --default-character-set=utf8 --compact -B t1 > t_b.sql
[root@multi 3306]# cat t_b.sql 
```

```sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `t1` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `t1`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `test1` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `num1` varchar(20) NOT NULL,
  `num2` varchar(20) NOT NULL,
  `num3` varchar(20) NOT NULL,
  `num4` int(11) NOT NULL DEFAULT '0' COMMENT 'test1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2000031 DEFAULT CHARSET=utf8;

/*!40101 SET character_set_client = @saved_cs_client */;
INSERT INTO `test1` VALUES (2000001,'690938','794482','1899596',0),(2000002,'7114536','9873892','8025852',0),(2000003,'507584','8460367','779079',0),(2000004,'8514298','234250','5628359',0),(2000005,'7439042','310137','9233557',0),(2000006,'5237373','8486196','6718861',0),(2000007,'8135719','522012','8202901',0),(2000008,'9448472','2633656','4822864',0),(2000009,'6213355','6598180','4350835',0),(2000010,'1959635','6745673','7849459',0),(2000011,'9010275','1503001','484179',0),(2000012,'7911894','8106930','6798971',0),(2000013,'9674065','7973412','844865',0),(2000014,'304088','8985848','4016974',0),(2000015,'3127327','3585711','8546578',0),(2000016,'1975754','4239037','5267926',0),(2000017,'3622517','2308806','676480',0),(2000018,'6455983','250474','1884422',0),(2000019,'8670686','7700168','2488781',0),(2000020,'9343400','9250657','8223086',0),(2000021,'3363461','2148048','649856',0),(2000022,'6805137','2076115','9965166',0),(2000023,'3597485','8091927','9667180',0),(2000024,'4060121','1299065','4314962',0),(2000025,'7677614','5443186','4183087',0),(2000026,'4585879','380131','8143022',0),(2000027,'9574716','3444513','8498418',0),(2000028,'2158552','5297508','11882',0),(2000029,'4166888','798795','1493311',0),(2000030,'5070170','870919','9144083',0);
```

> **--compact参数说明：**

（测试时用的比较多）可以优化输出内容的大小，让容量更少，适合调试。

```sql
--compact		Give less verbose output (useful for debugging).		Disables
 			structure comments and header/footer contructs. 	Enables
options  --skip-add-drop-table --no-set-names
 	 --skip-disable-keys --skip-add-locks
```

参数说明：该选项使得输出内容更简介，不包括默认选项中各种注释。有如下几个参数的功能。

<font color="#f8070d" size=3>`--skip-add-drop-table`</font>

<font color="#f8070d" size=3>`--no-set-names`</font>

<font color="#f8070d" size=3>`--skip-disable-keys`</font>

<font color="#f8070d" size=3>`--skip-add-locks`</font>

#### 8.2.5 压缩备份的数据

```sql
--- 输出时用管道进行备份，压缩效率将近3倍
--- 这个保存为一个压缩文件

mysqldump -S /data/3306/mysql.sock -uroot -p111 -B t1|gzip >t1.sql.gz
1360  t1.sql.g
3365  t2.sql
```

小结：

1. 备份数据使用-B参数，会在备份数据中增加建库及use库的语句
2. 备份数据使用-B参数，后面可以直接接多个库名。
3. 共gzip对备份的数据亚索
4. debug时可以用--compact减少输出，但不用于生产
5. 指定字符集备份用--default-character-set=utf8（一般不用）

### 8.3 mysqldump的工作原理

利用mysqldump命令备份数据的过程，实际上就是把数据从mysql库里以逻辑的sql语句的形式直接输出或者生成备份的文件的过程。

提示：使用mysqldump是把数据库的数据导出通过sql语句的形式存储，这种备份方式称之为逻辑备份，效率不是很高，一般50G以内的数据。

其他备份方式：物理备份：cp tar（停库），xtrabackup物理热备份。
备份多个库及多个参数

```sql
mysqldump -uroot -p111 -S /data/3306/mysql.sock --compact \
-B test t1|gzip>/data/test1.sql
```

**-B参数说明**

```bash
-B，参数是关键，表示接多个库并且增加use db和create database db的信息
-B, --databases     Dump several databases. Note the difference in usage; in
                      this case no tables are given. All name arguments are
                      regarded as database names. 'USE db_name;' will be
                      included in the output.
                      
# 用于导出多个数据库，注意这种情况系没有表。所有的名称参数都被认作为是数据库。
# 每个数据库名以空格隔开，将使用的数据库名称包含到输出里
# 当-B后的数据库列全时，同-A参数。
```

### 8.4 分库备份

分库备份实际上就是执行一个备份语句备份一个库，如果数据库里有多个库，就执行多条相同的备份单个库的备份语句就可以备份多个库了，注意每个库都可以用对应备份的库作为库名，结尾加.sql。备份多个库的命令如下：

mysqldump -uroot -p111 -B oldboy;

mysqldump -uroot -p111 -B test;

....

....

> **分库备份法1：**

```sql
mysql -uroot -p111 -S /data/3306/mysql.sock \
-e 'show databases;'|\
egrep -v "Database|_schema|mysql"|\
sed -r 's#^(.*)#mysqldump -uroot -p111 -S /data/3306/mysql.sock \1#g'

mysql -uroot -p111 -S /data/3306/mysql.sock bingbing
mysql -uroot -p111 -S /data/3306/mysql.sock t1
mysql -uroot -p111 -S /data/3306/mysql.sock test

-- 将结果交给bash
mysql -uroot -p111 -S /data/3306/mysql.sock \
-e 'show databases;'|egrep -v "Database|_schema|mysql"|\
sed -r 's#^(.*)#mysqldump -uroot -p111 -S /data/3306/mysql.sock -B \1|gzip>\1.sql.gz#g' \
|bash
```

> 法2：

见分库分表备份视频：http://edu.51cto.com/course/course_id-808.html

> **分库备份的意义何在？**

有时一个企业的数据库里会有多个库，例如（www.bbs，blog），但是出问题的时候很可能是某一个库，如果在备份时把所有的库都备份成了一个数据文件的话，回复某一个库的数据是就比较麻烦了。

### 8.5 备份单个表

语法：

```bash
mysqldump -uuserName -ppassWord dbName tableName >/data.sql
mysqldump -uuserName -ppassWord dbName tableName1 tableName2.. >/data.sql
```

***

**<font color="#0215cd" size=2>提示：不能加<font color="#f8070d" size=3>`-B`</font>参数了，因为库后面就是表了。</font>**

***

企业需求：一个库里有大表有小标，有时可能需要只回复某一个小表，上述的多表备份很难拆开，就想没有分库那样导致恢复某一个小表很麻烦。

那么又如何进行分表备份呢？如下，和分库的思想一样，每执行一条语句备份一个表，生成不同的数据文件即可。如下：

```bash
mysql -uroot -p111 -e 'use test; show tables;' \
|egrep -v 'Tables_in_test' \
|sed -r 's#^(.*)#mysqldump -uroot -p111 --compact test \1>\1.sql#g'|bash
```

<font color="#f8070d" size=2>分表备份缺点：文件多，很碎</font>

      备一个完整全备，在做一个分库分表备份
      脚本批量备份恢复多个SQL文件

> **面试题：多个库或者多个表备份到一块了，如何恢复单个库或者表？**

解答：

1. 第三方测试库，导入到库里，然后把需要的备份出来，恢复到正式库里。
2. 单表：grep表名 bak.sql>tab_name。
3. 实现分库分表备份

### 8.6 备份数据表结构

利用<font color="#f8070d" size=2>`mysqldump -d`</font>参数只备份表的结构，例：备份oldboy库的所有表的结构

```sql
mysqldump -uroot -p111 -d --compact -B test > t.sql
```

如果只导出数据则用<font color="#f8070d" size=2>`-t`</font>

```sql
mysqldump -uroot -p111 -S /data/3306/mysql.sock -t --compact -B test > 1t.sql 
```

<font color="#f8070d" size=2>`-T --tab=path`</font>：语句与数据分离，数据为文本。

```sql
mysqldump -uroot -p111 -S /data/3306/mysql.sock t1 test1 -T /data
```

注意只能对表进行分离，对数据库进行分离提示如下：

```bash
[root@multi data]# mysqldump -uroot -p111 -S /data/3306/mysql.sock -B t1 -T /data         
mysqldump: --databases or --all-databases can't be used with --tab.
```

小结：

* <font color="#f8070d" size=3>`-B`</font>备份多个库（并添加create和use语句）
* <font color="#f8070d" size=3>`-d`</font>只备份库表结构
* <font color="#f8070d" size=3>`-t`</font>只备份数据（sql语句形式）
* <font color="#f8070d" size=3>`-T`</font>分离表和数据成不同的文件，数据是文本，非SQL语句

### 8.7 刷新binlog参数

mysqldump用于定时对某一时刻的数据的全备份，例如：00点进行备份bak.sql.gz

增量备份：当有数据写入到数据库时，还会同时把更新的SQL语句写入到对应的文件里，这个文件就叫做binlog。

比如说晚上0点做备份， 10点宕机了，0-10点的数据就丢失了

**10点前丢失数据需要恢复的数据：**

1. 00点时刻备份的bak.sql.gz数据还原到数据库，这个时候数据恢复到了00点
2. 00-10点数据，就要从binlog里恢复。

#### 8.7.1 binlog作用

记录数据库更新的sql语句，不记录show select等，只是记录对数据库记录变更的二进制文件。

在mysql数据库当中，当你做一个全备之后到出问题的时刻，要想恢复，就是全备+全备之后的所有binlog。

#### 8.7.2 定界binlog

**问题：怎么界定备份之后和binlog文件之间连接的很紧密不多也不少。**

通过文件的日志点。可以通过<font color="#f8070d" size=3>`-F`</font>做一个区分。

只要我们做了备份，然后就刷新binlog，将来恢复的就是130以下的。130以上的包里面就包含了
binlog日志切割：确定全备和增量的结界点-F刷新binlog日志，生成新日志文件，将来增量恢复从这个新日志文件开始。

binglog文件生效需要一个参数：<font color="#f8070d" size=3>`log-bin log-bin=/data/3306/mysql-bin`</font>

```bash
ll #←备份前，查看目录binlog文件
mysql-bin.000316
mysql-bin.000317
mysql-bin.index

mysqldump -uroot -p111 -S /data/3306/mysql.sock --compact -B t1 > t1.sql -F

ll #←在刷新之后可以看到binlog文件增加了
mysql-bin.000316
mysql-bin.000317
mysql-bin.000318
mysql-bin.index
```

<font color="#f8070d" size=2>`--master-data`</font> 在备份语句里添加 <font color="#f8070d" size=3>`CHANGE MASTER`</font> 语句及 <font color="#f8070d" size=3>`binlog`</font> 文件及位置点信息

&nbsp;&nbsp;值1，为可执行的CHANGE MASTER语句

&nbsp;&nbsp;值2，为注释的`--CHANGE MASTER`语句

***

<font color="#0215cd" size=3> 注：\-\-master-data除了增量恢复确定临界点外，做主从复制时作用更大</font>

***

```bash
# 不加--master-data语句
mysqldump -uroot -p111 --compact -B t1 > t1.sql
cat t1.sql 

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `t1` /*!40100 DEFAULT CHARACTER SET utf8 */;
```

```sql
-- 加上master-data语句后
mysqldump -uroot -p111 --compact -B t1 > t1.sql --master-data=1
cat t1.sql 

CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000318', MASTER_LOG_POS=107;

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `t1` /*!40100 DEFAULT CHARACTER SET utf8 */;
```

```bash
[root@multi 3306]# mysqldump -uroot -p111 --compact -B t1 > t1.sql --master-data=2
[root@multi 3306]# cat t1.sql 
```

```sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000318', MASTER_LOG_POS=107;
```

### 8.8 mysqldump关键参数说明

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| -B                             | 指定多个库，增加建库语句和use语句。                          |
| --compact                      | 去掉注释，适合调试输出，生产不适用                           |
| -A                             | 备份所有库                                                   |
| -F                             | 刷新binlog日志，生成新文件，将来增量恢复从这个文件开始。     |
| --master-data                  | 增加binlog日志文件名及对应的位置点（即CHANGE MASTER语句）。--mater-data=1不注释 2注释 |
| -x --lock-all-tables           |                                                              |
| -l --lock-tables               | lock all tables for read                                     |
| -d                             | 只备份数据，无表结构，SQL语句形式。                          |
| -t                             | 只备份数据，无库表结构，SQL语句形式。                        |
| -T                             | 库表和数据分离不同文件，数据是文本形式。                     |
| --single-transaction           | 适合innodb事务数据库备份                                     |
| -q \-\-quick don't bufferquery | dump directly to stdout (Defaults to on; use --skip-quick to disable) |

***

**<font color="#0215cd" size=2> 说明：innodb表在备份时，通常启用选线--single-transaction来保证备份的一致性，实际上它的工作原理是设定本次会话的隔离级别为REPEATABLE READ，确保本次会话（dump）时，不会看到其他会话已经提交了的数据。</font>**

***



### 8.9 生产场景不同引擎mysqldump备份命令



#### 8.9.1 myisam引擎企业生产备份，命令（适合所有引擎或混合引擎）：

```sh
mysqldump -uroot -p111 -A -B -F -R --master-data=2 -x --events|gzip > /data/all.sql.gz
```

提示：-F也可以不用，与--，master-data有写重复

#### 8.9.2 innodb引擎企业生产备份命令：推荐使用

```sh
mysqldump -uroot -p111 -A -B -F -R --master-data=2 --events --single-transaction|gzip >/data/all.sql.gz
```

#### 8.9.3 --master-data作用：

1. 使用--master-data=2进行备份文件会增加如下内容：适合普通备份增量恢复

```sh
--CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.00020', MASTER_LOGZ_POS=1191
```

2. 使用--maste-data=1进行备份文件会增加如下内容：更适合主从复制

```sh
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.00020', MASTER_LOGZ_POS=1191
```

锁表的原理：
![img](../../../images/MySQL的备份与恢复/wps1.png)

innodb有acid的特性

dump命令是看不到后面生成的数据，只能看到执行这个命令时所有数据之前的这个数据 

事务引擎不锁表备份原理：--single-transaction 会话隔离

额外补充：

1. mysqldump逻辑备份说明

缺点：效率不是特别高

优点：简单、方便、可靠、迁移。

2. 超过50G可选方案

恢复数据到数据库的时候，认为通过SQL语句将数据删除的时候，做主从复制的时候

### 8.10 恢复数据库实战

数据库恢复事项

1. 数据恢复和字符集关联很大，如果字符集不正确会导致恢复的数据乱码。

mysql命令以及source命令恢复数据可的原理就是把文件的SQL语句，在数据库里重新执行过程。

2. 利用source命令恢复数据库。

进入mysql数据库控制台，mysql -uroot -p登陆后

use database;

然后使用source xx.sql，后面参数为脚本文件（如这里用到的sql）

source 2.sql 这个文件是系统路径，默认是登陆mysql前的系统路径

提示：

      source数据恢复和字符集关联很大，如果字符集不正确会导致恢复的数据乱码。
      utf8数据库，那么恢复的文件格式需要为utf8无bom头。

> **利用mysql命令恢复（标准）**

```sh
mysql -uroot -p111 -e 'use test;drop tables test1;show tables;'
mysql -uroot -p111 test < /2.sql
```

假设开发人员让我们插入数据到数据库（可能是邮件发给我们的，内容可能是字符串或是文件）sql文件里没有use db这样的字符，在导入时就要指定数据库名了。

```sql
[root@Multi2 3306]# mysql -uroot -p111 < /data/3306/2.sql
ERROR 1046 (3D000) at line 22: No database selected

[root@Multi2 3306]# mysql -uroot -p111 -e'use test';
ERROR 1049 (42000) at line 1: Unknown database 'test'

[root@Multi2 3306]# mysql -uroot -p111 test</data/3306/2.sql
```

如果在导出时指定-B参数，恢复时无需指定库恢复，为什么？

因为-B参数带了<font color="#f8070d" size=3>`use test;`</font>还会有<font color="#f8070d" size=3>`create database test;`</font>，而恢复时指定库就类似与<font color="#f8070d" size=3>`use test`</font>。

如果mysqldump备份时指定了-B，则恢复可以用如下方法：

```sh
mysql -uroot -p111 < back.sql
mysql -uroot -p111 DbName < back.sql # 条件是dbname库必须存在
```

***

<font color="#0215cd" size=3>提示：此处DbName相当于use Dbname</font>

***

> **问题：分库分表备份的数据如何快速恢复呢？**

还是通过脚本读指定的库和表，调用mysql命令恢复

```sh
for name in `ls /back/*.sql|sed -r 's#.back.sql##g'`;do mysql -uroot -p111 test<${name}.back.sql; done;
```

> **针对压缩的备份数据恢复**

**方法1：**

```sh
gzip -d /back/mysql_back.sql.gz
mysql -uroot -p111 dbname < /back/mysql_back.sql

gzip -cd mysql_back.sql.gz > mysql.sql #←不删除源备份文件
```

**方法2：**

```sh
gunzip < b.sql.gz > /opt/mysql.sql
mysql -uroot -p111 < /opt/mysql.sql
```

或者

```sh
gunzip -c back.sql.gz|mysql -uroot -p111 test
```

```sh
mysql -uroot -p111 -e \
'SELECT CONCAT("drop table ",table_name,";") FROM information_schema.`TABLES` WHERE table_schema="test";' \
|grep -Ev 'CONCAT("drop table ",table_name,";")'|sed -r "s#^(.*)#mysql -uroot -p111 -e 'use test;\1'#g" \
|bash
```

> **分表分库备份脚本**

```sh
#!/bin/sh
. /etc/init.d/functions
# define variable
BackDir=~/back
User=root
PassWD=111
Socket=/data/3306/mysql.sock
# define comment
Login="mysql -u${User} -p${PassWD} -S ${Socket}"
Dump="mysqldump -u${User} -p${PassWD} -S${Socket} -x --master-data=2 --compact"
DataBase=`$Login -e 'show databases;'|egrep -v  '*chema|mysql'|sed '1d'`

[ ! -d $BackDir ] && mkdir -p $BackDir
for list in $DataBase
do
	ST=`$Login -e "use $list;show tables;"|sed '1d'`
	[ ! -d $BackDir/$list ] && mkdir -p $BackDir/$list
	for table in $ST
	do
		$Dump $list $table|gzip>$BackDir/$list/$table.`date +%F`.sql.gz
		[ $? -eq 0 ] && action "$list > $table is ok" /bin/true || "$list > $table dump is fail"
	done
done
```


