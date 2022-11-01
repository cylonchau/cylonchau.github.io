# 

[toc]

## 1 键/值介绍

Redis key值是二进制安全的，这意味着可以用任何二进制序列作为key值，从形如“0foo”的简单字符串到一个JPG文件的内容都可以。空字符串也是有效key值。

***关于key的几条规则:***

太长的键值，例如1024字节的键值，不仅因为消耗内存，而且在数据中查找这类键值的计算成本很高。

太短的键值，如果你要用 <font color="#f8070d" size=3>`u:1000:pwd`</font>来代替<font color="#f8070d" size=3>`user:1000:password`</font>，这没有什么问题，但后者更易阅读，并且由此增加的空间消耗相对于key object和value object本身来说很小.当然，没人阻止您一定要用更短的键值节省一丁点空间。

<font style="background:#bafe01;" size=2>最好坚持一种模式;</font>。例如：<font color="#f8070d" size=2>`object-type:id:field`</font>就是个不错的注意，像这样<font color="#f8070d" size=2>`user:1000:password`</font>。我喜欢对多单词的字段名中加上一个点，就像这样：<font color="#f8070d" size=2>`comment.1234.renlv_to`</font>

key建议：<font color="#f8070d" size=3>`object-type:id:field`</font> <font style="background:#bafe01;" size=2>长度10-20</font>

value建议：<font style="background:#bafe01;" size=2>string不要超过2K set sortedset元素不要超过5000</font>

```bash
[root@redis ~]# redis-cli set user_list:user_id:5 zhangsan
OK
[root@redis ~]# redis-cli get user_list:user_id:5
"zhangsan"
```

## 2 通用操作

### 2.1 找到全部给定模式的匹配到的key

```bash
127.0.0.1:6379> keys * #<==打印全部key
1) "name"
2) "site"
127.0.0.1:6379> keys na[ma]e #<==返回正则匹配到的key
1) "name"
127.0.0.1:6379> keys nam?
1) "name"
```

### 2.2 randomkey返回随机key

```bash
127.0.0.1:6379> randomkey
"name"
127.0.0.1:6379> randomkey
"site"
```

### 2.3 exists检查key是否存在

```bash
127.0.0.1:6379> exists site
(integer) 1
127.0.0.1:6379> exists sites
(integer) 0
```

### 2.4 type 查看key类型

```bash
127.0.0.1:6379> type set
set
127.0.0.1:6379> type site
string
```

### 2.5 修改key名称

```bash
127.0.0.1:6379> rename site web 
OK
127.0.0.1:6379> keys *
1) "set"
2) "web"
3) "name"
4) "test"
```
> **renamnx rename not exists,当要修改的新名称存在,会用改名后的覆盖存在的那个key**

```bash
127.0.0.1:6379> set rename_test test
OK
127.0.0.1:6379> set rename_test1 test1
OK
127.0.0.1:6379> set tmp tmp
OK
127.0.0.1:6379> rename rename_test tmp
OK
127.0.0.1:6379> keys *
1) "tmp"
2) "rename_test1"
127.0.0.1:6379> get tmp
"test" #<==可以看到tmp把rename_test给覆盖了
```

### 2.6 将当前数据库的 key 移动到给定的数据库db当中

```bash
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> keys *
(empty list or set)
127.0.0.1:6379[2]> select 0
OK
127.0.0.1:6379> move web 2
(integer) 1
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> keys *
1)"web"
```

### 2.7 dbsize返回当前选择的数据库的key数量

```bash
127.0.0.1:6379> dbsize
(integer) 1
```

### 2.8 shutdown 同步保存数据到磁盘，并结束Redis进程

语法：shutdown [NOSAVE|SAVE]

```bash
redis-cli shutdown save
12632:M 22 Apr 16:42:23.492 * Saving the final RDB snapshot before exiting.
12632:M 22 Apr 16:42:23.528 * DB saved on disk
12632:M 22 Apr 16:42:23.528 * Removing the pid file.
12632:M 22 Apr 16:42:23.529 # Redis is now ready to exit, bye bye...
```

### 2.9 异步操作Redis

bgrewriteaof (异步重写aof日志文件)
作用减少aof文件大小
BGSAVE(异步保存数据到磁盘)


## 3 String字符串类型

这是最简单的Redis类型。如果你只用这种类型,Redis就是一个可以持久化的memcached服务器(注:memcache的数据仅保存在内存中,服务器重启后,数据将丢失)。

### 3.1 常规字符串操作

在Redis中，常用set设置一对key/value键值，然后用get来获取字符串的值。value值可以是任何类型的字符串(包括二进制数据)，例如你可以在一个键下保存一个jpg图片。但值的长度不能超过1GB。

>  **set：设置一个建的字符串值**

语法：set key value [EX seconds] [PX milliseconds] [NX|XX]

```bash
127.0.0.1:6379> set k1 test:key:123
OK  
127.0.0.1:6379> ttl k1
(integer) -1 #<==set默认生命周期为永久有效
127.0.0.1:6379> set k1 test_key NX #<==设置一个key,如果不存在就创建。
(nil) #<==因k1存在故没有创建
127.0.0.1:6379> set k2 test_key NX
OK	#<==名为k2的key不存在成功创建
127.0.0.1:6379> set k2 test EX 100 #<==设置key的生命周期 px毫秒 ex秒
OK 
127.0.0.1:6379> ttl k2
(integer) 97
```

>  **mset：设置多个key value**

```bash
127.0.0.1:6379> mset key zhangsan ex 100 key1 lisi 
OK
127.0.0.1:6379> keys *
1) "key1"
2) "ex"
3) "key" #<==通过结果可得知 mset不支持判断是否存在与设置生命周期
```

>  **setrange：从指定偏移量开始重写字符串的一部分**

```bash
127.0.0.1:6379> set key test:123
OK
127.0.0.1:6379> setrange key 5 abc
(integer) 8
127.0.0.1:6379> get key
"test:abc"  #<==不存在的偏移量用\x00填充
```

>  **append：将值附加到key中**

```bash
127.0.0.1:6379> set test a
OK
127.0.0.1:6379> APPEND test bc
(integer) 3
127.0.0.1:6379> get test
"abc"
```

>  **getrange：Get a substring of the string stored at a key**

```bash
127.0.0.1:6379> set k test:getrange
OK
127.0.0.1:6379> getrange k 0 3
"test"
127.0.0.1:6379> getrange k 0 100
"test:getrange" #<==如果指定值大于字符串长度返回到字符串结尾处
127.0.0.1:6379> getrange k 100 1000
"" #<==如果开始位置大于字符串总长度,返回空字符串
```

### 3.2 String类型可以用来存储数字
虽然字符串是Redis的基本值类型,但你仍然能通过它完成一些有趣的操作。例如：原子递增。

INCR命令将字符串值解析成整型,将其加1,最后将结果保存为新的字符串值,类似的命令有INCRBY, DECR and DECRBYC实际上他们在内部就是同一个命令,只是看上去有点儿不同。

incr是原子操作意味着什么呢?就是说即使多个客户端对同一个key发出INCR命令,也决不会导致竟争的情况.例如如下情况永远不可能发生:「客户端1和客户端2同时读出“10",他们俩都对其加到11,然后将新值设置为11。最终的值一定是12,read-increment-set操作完成时,其他客户端不会在同一时间执行任何命令。

> **incr:将键的整数值递增1**

```bash
127.0.0.1:6379> set num 1
OK
127.0.0.1:6379> incr num
(integer) 2
```

> **decr:将键的整数值递减1**

```bash
127.0.0.1:6379> decr num
(integer) 0
127.0.0.1:6379> decr num
(integer) -1 #<==与memcached不同的是,redis会被减少为负数
```

> **incrby:将键增加一个指定定量**

```bash
127.0.0.1:6379> incrby num 10
(integer) 9
```

> **incrbyfloat: 将键增加一个指定定量的浮点值**

```bash
127.0.0.1:6379> incrbyfloat num 0.1 
"9.1" #<==操作后值变为浮点型
127.0.0.1:6379> incrby num 11 
(error) ERR value is not an integer or out of range #<==不可以在进行整型原子递增
```

### 3.3 为key设置新值并且返回原值

对字符串,另一个操作是GETSET命令,行如其名:他为key设置新值并且返回原值。这有什么用处呢?例如:你的系统每当有新用户访问时就用INCR命令操作一个Redis key。你希望每小时对这个信息收集一次。你就可以GETSET这个key并给其赋值0并读取原值。

> **getset：设置key的字符串值，并返回旧值**

```bash
127.0.0.1:6379> getset k1 test:getset:demo
"test"
127.0.0.1:6379> get k1
"test:getset:demo"
```

### 3.4 setbit getbit bitcount binop

> **这是偏移位上的二进制值**

```bash
(error) ERR bit offset is not an integer or out of range
127.0.0.1:6379> setbit s 4294967296 1
(error) ERR bit offset is not an integer or out of range
127.0.0.1:6379> setbit s 4294967295 1 #<==最大范围为232-1 512M key的最大值
(integer) 0
(1.46s)
```

## 4 List类型

要说清楚列表数据类型,最好先讲一点儿理论背景,在信息技术界List这个词常常被使用不当.例如"Python Lists"就名不副实(名为Linked Lists),<font style="background:#bafe01;" size=2>但他们实际上是数组</font>(同样的数据类型在Ruby中叫数组)。

列表就是有<font style="background:#bafe01;" size=2>序元素的序列</font>:10,20,1,2,3就是一个列表。但用数组实现的List和用Linked List实现的List,在属性方面大不相同。

Redis lists基于Linked Lists实现。这意味着即使在一个list中有数百万个元素,在头部或尾部添加一个元素的操作,其时间复杂度也是常数级别的。用LPUSH命令在十个元素的list头部添加新元素,和在千万元素list头部添加新元素的速度相同。

那么,坏消息是什么?在数组实现的list中利用索引访问元素的速度极快,而同样的操作在linked list实现的list上没有那么快。

Redis Lists用linked list实现的原因是:对于数据库系统来说,至关重要的特性是:能非常快的在很大的列表上添加元素.另一个重要因素是,正如你将要看到的:Redis lists能在常数时何取得常数长度。


### 4.1 创建list并插入数据

LPUSH命令可向list的左边(头部)添加一个新元素,而RPUSH命令可向list的右边(尾部)添加一个新元素。最后LRANGE命令可从list中取出一定范围的元素。

> **lpush:向list头部插入数据，如果list不存在则自动创建**

```bash
127.0.0.1:6379> lpush list lisi #<== 头部插入一个数据
(integer) 1
127.0.0.1:6379> lpush list zhangsan
(integer) 2
```

> **llen:查看list列表元素个数**

```bash
127.0.0.1:6379> llen list	#<==获得列表元素个数
(integer) 2
```
> **lrange:查看列表所有元素**

```bash
127.0.0.1:6379> lrange list 0 3 #<== 打印列表的1,3元素
 1) "zhangsan"
 2) "lisi"
 3) "test1"
127.0.0.1:6379> lrange list 0 -1#<== 打印列表的所有元素
```

***
**<font color="#0215cd" size=2> <font color="#f8070d" size=2>⚠</font> 注意：RANGE带有两个索引,范围内第一个和最后一个元素。这两个索引都可以负来告知redis从尾部开始计数,因此-1表示最后一个元素,-2表示list中的倒数第二个元素,以此类推。
</font>**
***

> **lindex:返回指定元素在列表中的下标**

```bash
127.0.0.1:6379> rpush a 1 2 3 4 5
(integer) 5
127.0.0.1:6379> lindex a 1
"2"
127.0.0.1:6379> lindex a 1
```

> **linsert在指定元素前后插入数据**

```bash
127.0.0.1:6379> rpush arr 1 2 3
(integer) 3
127.0.0.1:6379> linsert arr after 2 5
(integer) 4
127.0.0.1:6379> lrange arr 0 -1
1) "1"
2) "2"
3) "5"
4) "3"
```

### 4.2 list应用场景

正如你可以从上面的例子中猜到的,list可被用来实现聊天系统。还可以作为不同进程间传递消息的队列。关键是,你可以每次都以原先添加的顺序访问数据。这不需要任何SQL ORDER BY操作,将会非常快,也会很容易扩展到百万级别元素的规模。

例如在评级系统中,比如社会化新闻网站reddit.com,你可以把每个新提交的链接添加到一个list,用LRANGE可简单的对结果分页。

在博客引擎实现中,你可为每篇日志设置一个list,在该list中推入进博客评论,等等。

向Redis list压入ID而不是实际的数据,

在上面的例子里,我们将“对象”(此例中是简单消息)直接压入Redis list,但通常不应这么做,由于对象可能被多次引4:例如在一个lis:中维护其时间顺序,在一个集合中保存它的类别,只要有必要,它还会出现在其他list中,等等。

让我们回到reddit.com的例子,将用户提交的链接(新闻)添加到list中,有更可靠的方法如下所示

```bash
127.0.0.1:6379> incr next.news.id
(integer) 1
127.0.0.1:6379> set new:1:title "redis is simple"
OK
127.0.0.1:6379> set news:1:url "http://www.baidu.com"
OK
127.0.0.1:6379> lpush submitted.news 1
(integer) 1
```
OK我们自增一个key,很容易得到一个独一无二的自增ID,然后通过此ID创建对象入为对象的每个字段设置一个key。最后将新对象的submitted.news lists。

>> rpop 从尾部删除一个元素
>> rpush 向尾部插入一个数据
>> lpop 从头部删除一个元素

### 4.3 删除list内元素

> **lrem:删除count的绝对值个value后结束<font style="background:#bafe01;" size=2> count > 0从头开始删,<0从尾部开始删</font>**

```bash
127.0.0.1:6379> rpush aa a a b c a a
(integer) 6
127.0.0.1:6379> lrem aa 1 a
(integer) 1
127.0.0.1:6379> LRANGE aa 0 -1
1) "a"
2) "b"
3) "c"
4) "a"
5) "a"
127.0.0.1:6379> lrem aa -2 a
(integer) 2
127.0.0.1:6379> LRANGE aa 0 -1
1) "a"
2) "b"
3) "c"
```

> **ltrim:剪切key对应的链接,切[start,stop]一段,并把该段重新赋给key**

```bash
127.0.0.1:6379> rpush a 1 2 3 4 5
(integer) 5
127.0.0.1:6379> ltrim a 2 3
OK
127.0.0.1:6379> lrange a 0 -1
1) "3"
2) "4"
```
> **rpoplpush:将arr尾部的元素弹出到tmp头部**

作用：接收返回值,并做业务处理。如果成功,rpop tmp清除任务；如不成功,下次从tmp表里取任务

```bash
127.0.0.1:6379> rpoplpush arr tmp
"3"
127.0.0.1:6379> lrange arr 0 -1
1) "1"
2) "2"
3) "5"
127.0.0.1:6379> lrange tmp 0 -1
1) "3"
```

## 5 set集合

Redis集合是未排序的集合,其元素是二进制安全的字符串。sadd命令可以向集合添加一个新元素。和sets相关的操作也有许多,比如检测某个元素是否存在,以及实现交集,并集,差集等等。

### 5.1 创建并向集合插入新元素

> **sadd：向集合内添加一个或多个元素**

```bash
127.0.0.1:6379> sadd set a b c d #<==向集合key中增加元素
(integer) 4 
```

> **smembers：查看集合内所有元素**

```bash
127.0.0.1:6379> smembers set #<==返回集合中的所有元素
1) "d"
2) "c"
3) "b"
4) "a"
```

### 5.2 删除集合中元素

> **sadd：从集合内删除一个或多个指定元素** 

```bash
127.0.0.1:6379> srem set a #<==删除集合中名为a的元素
(integer) 1
127.0.0.1:6379> smembers set
1) "d"
2) "c"
3) "b"
```

### 5.3 查看集合内元素

> **sismember：检查集合中是否存在某个元素**

```bash
127.0.0.1:6379> sismember set a
(integer) 0
127.0.0.1:6379> sismember set b
(integer) 1
```
> **srandmember：返回元素中指定个数的随机元素**

语法 srandmember key [count]

```bash
127.0.0.1:6379> srandmember set  2
1) "d"
2) "b"
```

>  **scard返回元素中的个数**

```bash
127.0.0.1:6379> scard set
(integer) 3
```

### 5.4 移动集合内元素

> **smove：将source里的member移动到destination集合中**

语法：SMOVE source destination member

```bash
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> sadd set a b c d e
(integer) 5
127.0.0.1:6379> smove set tmp e
(integer) 1
127.0.0.1:6379> keys *
1) "set"
2) "tmp"
127.0.0.1:6379> smembers tmp
1) "e"
127.0.0.1:6379> smembers set
1) "c"
2) "b"
3) "a"
4) "d"
```
“b”是这个集合的成员,而“b”不是。集合特别适合表现对象之间的关系。例如<font style="background:#bafe01;" size=2>用Redis集合可以很容易实现标签功能</font>。

下面是一个简单的方案:对每个想加标签的对象,用一个标签ID集合与之关联,并且对每个已有的标签,一组对象ID与之关联。

例如假设我们的新闻ID=1000被加了三个tag 1,2,5就可以设置下面两个集合

```bash
127.0.0.1:6379> sadd news:1000:tags 1
(integer) 1
127.0.0.1:6379> sadd news:1000:tags 2
(integer) 1
127.0.0.1:6379> sadd news:1000:tags 5
(integer) 1
127.0.0.1:6379> smembers news:1000:tags
1) "1"
2) "2"
3) "5"
```
而有些看上去并不简单的操作仍然能使用相应的redis命令轻松实现。例如我们也许想获得一份同时拥有标签1,2,10和27的对象列表。这可以用SINTER命令来做,他可以在不同集合之间取出交集。

tags:1:obj tags:2:obj tags:5:obj tags:27:obj

```bash
127.0.0.1:6379> smembers test1
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> smembers test4
1) "1"
2) "2"
3) "9"
127.0.0.1:6379> sinter test1 test4
1) "1"
2) "2"
```

### 5.5 集合计算

***创建一个集合***

```bash
127.0.0.1:6379> sadd A 1 2 3 4 5
(integer) 5
127.0.0.1:6379> sadd B 1 3 4 5
(integer) 4
127.0.0.1:6379> sadd C 1 2 6 7 8 9
(integer) 6
```
> **sinter：取出指定集合内的交集**

```bash
127.0.0.1:6379> sinter A B  #<==取出A B两个集合中的交集
1) "1" 
2) "3"
3) "4"
4) "5"
127.0.0.1:6379> sinter A B C
1)"1"
```

> **sinterstore：将多个集合的交集放入另一个集合内**

```bash
127.0.0.1:6379> sinterstore tmp A B  #<== 将A B集合中的交集放置在tmp集合中
2)(integer) 4
3)127.0.0.1:6379> smembers tmp
4)1) "1"
5)2) "3"
6)3) "4"
7)4) "5"
```

> **sunion获得多个集合的并集**

```bash
127.0.0.1:6379> sunion A B
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

> **sdiff获得多个集合的差**

```bash
127.0.0.1:6379> sdiff A B
1) "2"
```

## 6 有序集合

集合是使用频率很高的数据类型,但是…对许多问题来说他们也有点儿太不讲顺序了；因此Redis1.2引入了有序集合。他和集合非常相似,也是二进制安全的字符串集合,但是这次带有关联的score,以及一个类似lrange的操作可以返回有序元素,此操作只能作用于有序集合,它就是,zrange命令。

基本上有序集合从某种程度上说是SQL世界的索引在Redis中的等价物。例如在上面提到的reddit.com例子中,并祥有提到如何根据用户投票和时间因素将新闻组合生成首页。我们将看到有序集合如何解决这个问题,但最好先从更简单的事情开始,阐明这个高级数据类型是如何工作的.让我们添加几个黑客,并将他们的生日作为"score"。

### 6.1 添加并查看有序集合

> **添加一个或多个成员到一个集合,如果这个成员存在,则更新其排序分数**

语法：ZADD key [NX|XX] [CH] [INCR] score member [score member ...]

***选项详解***

| 参数 | 解释                                 |
| :--: | :----------------------------------- |
|  XX  | 仅更新已经存在的元素。不要添加元素。 |
|NX|	不要更新现有的元素。始终添加新元素。|
|CH|	从添加的新元素的数量修改返回值,改变元素的总数（CH是更改的缩写）。已更改的元素是添加的新元素,已经存在的元素已更新。所以在命令行中指定的具有与过去相同的分数的元素不计算在内。注意：通常,ZADD的返回值仅计算添加的新元素的数量。|
|INCR|	当指定此选项时,ZADD的行为就像ZINCRBY。在此模式下只能指定一个记分元素对。|

**创建一个集合**

```bash
127.0.0.1:6379> zadd hacker 1953 zhangfei 1963 zhaoyun 1989 wangping 1992 zhugeliang 1945 liubei
(integer) 5
127.0.0.1:6379> zadd hacker 100 zhugeliang
(integer) 0
127.0.0.1:6379> zrange hacker 0 -1
1) "zhugeliang"
2) "liubei"
3) "zhangfei"
4) "zhaoyun"
5) "wangping"
```

> **查看某个元素在集合内的位置**

```bash
127.0.0.1:6379> zscore code_lag py
"4"
```

对有序集合采说,按生日排序返回这些数据易如反掌,因为他们已经是有序的。有序集合是通过一个dual-ported数据结构实现的,它包含一个精简的有序列表和一个hash table,因此添加一个元素的时间复杂度是O(log(N))。这还行,但当我们需要访问有序的元素时,Redis不必再做任何事情,它已经是有序的了。

### 6.2 对集合的值排序

> **通过索引查看一个有序集合的范围**

```bash
127.0.0.1:6379> zrange hacker 0 -1 withscores  #<==打印score
 1) "liubei"
 2) "1945"
 3) "zhangfei"
 4) "1953"
 5) "zhaoyun"
 6) "1988"
 7) "wangping"
 8) "1989"
 9) "zhugeliang"
10) "1992"
```

> **通过指定索引范围查看有序集合列表（倒序）**

```bash
127.0.0.1:6379> zrevrange hacker 0 -1 withscores
 1) "zhugeliang"
 2) "1992"
 3) "wangping"
 4) "1989"
 5) "zhaoyun"
 6) "1988"
 7) "zhangfei"
 8) "1953"
 9) "liubei"
10)"1945"
```

### 6.3 查看集合中的元素个数

> **返回有序集合中元素的个数**

```bash
127.0.0.1:6379> zcard hacker   
(integer) 8
```
> **返回min,max包括极值**

```bash
127.0.0.1:6379> zcount hacker 1950 2000
(integer) 7
127.0.0.1:6379> zcount hacker (1950 (2000 #<==不包括极值的用法
(integer) 4
```

### 6.4 查找集合区间的元素

zrangebysocre：返回有序集合key的分数min与max之间的所有元素（等于min或max）。这些元素被认为是从低到高的顺序排列。min和max可以是-inf(无穷)和+inf(正无穷)，可以不需要知道的有序集合最高或最低score来获得元素。

> **获取1950年之前（两个极值也包含）出生的人**

```bash
127.0.0.1:6379> zrangebyscore hacker -inf 1950 withscores
1) "liubei"
2) "1945"
3) "jiangwie"
4) "1950"
```

> **返回1950,1970区间内的值**

```bash
127.0.0.1:6379> zrangebyscore hacker 1950 1970 withscores
1) "jiangwie"
2) "1950" 
3) "zhangfei"
4) "1953" 
```

> **使用)返回 1950,2000区间内的值**

```bash
127.0.0.1:6379> zrangebyscore hacker (1950 (2000 withscores
1) "zhangfei"
2) "1953"
3) "zhaoyun"
4) "1988"
5) "wangping"
6) "1989"
7) "zhugeliang"
8) "1992"
```

> **使用limit返回指定区间内的值**

```bash
127.0.0.1:6379> zrangebyscore hacker -inf +inf withscores limit 0 3
1) "liubei"
2) "1945"
3) "jiangwie"
4) "1950"
5) "zhangfei"
6) "1953"
```

zremrangebyscore这个名字虽然不算好,但他却非常有用,还会返回已删除的元素数量。

回到Reddit的例子，现在我们有个基于有序集合的像样方案来生成首页。用一个有序集合来包含最近几天的新闻(用zremrangebyscore不时的删除旧新闻).用一个后台任务从有序集合中获取所有元素,根据用户投票和新闻时间计算score,然后用新闻ID和scores关联生成reddit.home.page有序集合.要显示首页,我们只需闪电般的调用ZRANGE不时的从reddit.home.page有序集合中删除过旧的新闻也是为了让我们的系统总是工作在有限的新闻集合之上。
更新有序集合的scores.

结束这篇指南之前还有最后一个小贴士.有序集合scores可以在任何时候更新。只要用ZADD对有序集合内的元素操作就会更新它的score(和位置),时间复杂度是O(log(N)),因此即使大量更新,有序集合也是合适的。<font style="background:#bafe01;" size=2>其中N是排序集合中的元素数，M是返回元素的数量。如果M是常数（例如，总是用LIMIT来询问前10个元素)，可以考虑O(log=(N))。</font>

### 6.5 计算交并集

> **计算由numkeys指定个数的key的的交集，并将结果存储在其中destination**

```bash
127.0.0.1:6379> zadd k1 1 a 2 b 3 c
(integer) 3  #<==a b c相当于数组的key 1 2 3相当于数组的值

127.0.0.1:6379> zadd k2 10 a 20 b 30 c
(integer) 3 

127.0.0.1:6379> zinterstore tmp 1 k1 k2
(error) ERR syntax error #<==指定numkeys的数量和传参数量不一致会提示语法错误
127.0.0.1:6379> zinterstore tmp 2 k1 k2
(integer) 3

127.0.0.1:6379> zrange tmp 0 -1 withscores
1) "a" #<==可以看出默认的聚合方法是sum
2) "11"
3) "b"
4) "22"
5) "c"
6) "33"

127.0.0.1:6379> zinterstore tmp 2 k1 k2 weights 2 1 aggregate sum
(integer) 3  #<==指定权重计算两个之间key之间的交集

127.0.0.1:6379> zrange tmp 0 -1 withscores
1) "a" 
2) "12" #<==k1 a=1权重为2 就等于2x1+10=12 
3) "b"
4) "24"
5) "c"
6) "36"
```


## 7 hash

hash能够<font style="background:#bafe01;" size=2>存储一个key对多个属性的数据</font> 如：<font color="#f8070d" size=3>`user.username user.password`</font>

### 7.1 设置key中的域与值

> **hset：将key中的field域设置为value，如果field域存在则覆盖原value，不存在则添加**

```bash
127.0.0.1:6379> hset user1 name zhangsan
(integer) 1
127.0.0.1:6379> hset user1 age 12
(integer) 1
127.0.0.1:6379> hset user1 gender male
(integer) 1
```

> **hmset：给key设置多个field域与值**

```bash
127.0.0.1:6379> hmset user2 name lisi age 11 gender female
OK
```
### 7.2 获得key中的域和值

> **hget：返回key中一个域的值**

```bash
127.0.0.1:6379> hget user1 name
"zhangsan"
```

> **hgetall：返回key中所有域和值**

```bash
127.0.0.1:6379> hgetall user1
1) "name"
2) "zhangsan"
3) "age"
4) "12"
5) "gender"
6) "male"
```

> **hmget：返回key中多个值**

```bash
127.0.0.1:6379> hmget user1 name age
1) "zhangsan"
2) "12"
```
> **hkeys：返回key中的所有域**

```bash
127.0.0.1:6379> hkeys user1
1) "age"
2) "gender"
3) 
```

> **hvals：返回key中的所有域的值**

```bash
127.0.0.1:6379> hvals user1
1) "12"
2) "male"
```

### 7.3 删除key

> **hdel：删除key中一个或多个域**

```bash
127.0.0.1:6379> hdel user1 name
(integer) 1
127.0.0.1:6379> hgetall user1
1) "age"
2) "12"
3) "gender"
4) "male"

```

### 7.4 查看域信息

> **hlen：返回key中域的个数**

```bash
127.0.0.1:6379> hlen user1
(integer) 2
```

> **hexists：查看key中是否存在某个域**

```bash
127.0.0.1:6379> hexists user1 name
(integer) 0
127.0.0.1:6379> hexists user1 age
(integer) 1
```

> **hstrlen：返回key中域的值的长度**

```bash
127.0.0.1:6379> HSTRLEN user2 gender
(integer) 6
```

### 7.5 hash的原子操作

> **hincrby：对域中的值增长value个(单位整型)**
>
> > 语法：hincrby key field increment

```bash
127.0.0.1:6379> hget user1 age
"14"
127.0.0.1:6379> hincrby user1 age 2
(integer) 16
127.0.0.1:6379> hget user1 age
"16"
```

> **hincrbyfloat：对域中的值增长value(单位浮点型)**

```bash
127.0.0.1:6379> hincrbyfloat user1 age 0.3
"16.3"
127.0.0.1:6379> hget user1 age
"16.3"
127.0.0.1:6379> hincrbyfloat user1 age -2 #<==减少用负数即可
"14.3"
```

bitMap 可实现用很小的内存实现高效的存储。

HyperLogLog 超小内存唯一值计数。 12k来实现唯一值得计数

GEO 基于地理信息位置定位
