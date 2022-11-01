# 

[toc]


Redis的所有数据都存储在内存中，但是他也提供对这些数据的持久化。

Redis是一个支持持久化的内存数据库，Redis需要经常将内存中的数据同步到磁盘来保证持久化。Redis支持四种持久化方式，一种是<font style="background:#bafe01;" size=2>`Snapshotting`(快照)也是默认方式</font>，另一种是<font style="background:#bafe01;" size=2>`Append-only file`(aof)的方式</font>。

## 1 RDB持久化方式

Snapshotting方式是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为dump.rdb。可以通过配置设置自动做快照持久化。例如可以配置redis在n秒内如果超过m个key被修改就自动做快照。

### 1.1 实现机制

1.  Redis调用fork子进程。

2.  父进程继续处理client请求，子进程负责将内存内容写入到临时文件。由于os的实时复制机制(copy on write)父子进程会共享相同的物理页面，当父进程处理写请求时os会为父进程要修改的页面创建副本，而不是写共享的页面。所以子进程地址空间内的数据是fork时刻整个数据库的一个快照。

3.  当子进程将快照写入临时文件完毕后，用临时文件替换原来的快照文件，然后子进程退出。client也可以使用save或者bgsave命令通知redis做一次快照持久化。save操作是在主线程中保存快照的，由于redis是用一个主线程来处理所有client的请求，这种方式会阻塞所有clien:请求。所以不推荐使用。另一点需要注意的是，每次快照持久化都是完整写入到磁盘一次并不是增量的只同步变更数据。如果数据量大的话，而且写操作比较多，必然会引起大量的磁盘IO操作，可能会严重影响性能。



![img](../../../images/redis数据持久化/wps2.png)



**缺点：**

快照方式是在一定间隔时间做一次的，所以如果<font style="background:#bafe01;" size=2>redis意外down掉的话，就会丢失最后一次快照后的所有修改</font>。如果应用要求不能丢失任何修改的话，可以采用aof持久化方式。

### 1.2 相关配置

```bash
save 900 1 	#←900秒内至少有1个key被改变
save 300 10 	#←300秒内至少有10个key被改变
save 60 10000 	#←60秒内至少有10000个key被改变
stop-writes-on-bgsave-error yes #←后台存储错误后停止写。如：磁盘空间不足
rdbcompression yes #←使用LZF压缩rdb文件
rdbchecksum yes 	  #←存储和加载rdb文件时校验
dbfilename dump.rdb #←存储rdb文件名
dir /app/redis/db/  #←rdb文件路径
```

### 1.3 持久化测试

```bash
11512:M 22 Apr 01:04:47.028 * 5 changes in 60 seconds. Saving...
11512:M 22 Apr 01:04:47.029 * Backgroundsaving started by pid 11520
11520:C 22 Apr 01:04:47.032 * DB saved on disk
11520:C 22 Apr 01:04:47.033 * RDB: 0 MB of memory used by copy-on-write
11512:M 22 Apr 01:04:47.129 * Background saving terminated with success
```
**生成dump.rdb文件**

```bash
[root@redis ~]# ll /app/redis/db
dump.rdb
```

**关闭服务删除dump.rdb文件重新启动redis服务**

```bash
127.0.0.1:6379> keys *
(empty list or set)
```

**将dump.rdb.bk文件恢复成dump.rdb。再启动服务器。**

```bash
127.0.0.1:6379> keys *
1) "test-durable"
2) "test-durable-1"
```


### 1.4 手工强制刷新

```bash
127.0.0.1:6379> save
OK
127.0.0.1:6379> bgsave
Background saving started
```

**此时服务器端消息**

```bash
4330:M 30 Mar 02:42:14.496 * DB saved on disk
[root@Lnmp redis]# 4330:M 30 Mar 02:42:28.700 * Background saving started by pid 18345
18345:C 30 Mar 02:42:28.718 * DB saved on disk
18345:C 30 Mar 02:42:28.718 * RDB: 0 MB of memory used by copy-on-write
4330:M 30 Mar 02:42:28.796 * Background saving terminated with success
```

## 2 AOF存储介绍

Append-Only-File(追加式的操作日志记录)

由于快照方式是在一定间隔时间做一次的，所以如果<font style="background:#bafe01;" size=2>redis意外down掉的话，就会丢失最后一次快照后的所有修改</font>。如果应用要求不能丢失任何修改的话，可以采用aof持久化方式。

aof比快照方式有更好的持久化性，是由于在使用aof持久化方式时,redis会将每一个收到的写命令都通过write函数追加到文件中(默认是appendonly.aof)。当redis重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。当然由于os会在内核中缓存write做的修改，所以可能不是立即写到磁盘上。这样aof方式的持久化也还是有可能会丢失部分修改.不过我们可以通过配置文件告诉redis我们想要通过fsync函数强制os写入到磁盘的时机。

### 2.1 实现机制

以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来<font style="background:#bafe01;" size=2>(不记录读操作)</font>，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据。<font style="background:#bafe01;" size=2>当 Redis 重启时，它会优先使用AOF文件来还原数据</font>，因为AOF文件保存的数据集通常比RDB文件所保存的数据集更完整。你甚至可以关闭持久化功能，让数据只在服务器运行时存在。在redis中这种存储方式默认是关闭的，需要在redis.conf文件中开启。

![img](../../../images/redis数据持久化/wps3.png)

<center>redis AOF重写原理图</center>

### 2.2 相关配置

```bash
appendonly no/yes 		#←是否打开 aof日志功能
appendfsync always  #←每次收到写命令就立即强制写入磁盘，最慢,保证完全的持久化，不推荐
appendfsync everysec #←每秒钟强制写入磁盘一次，在性能和持久化方面折中，推荐
appendfsync no #←完全依赖os，性能最好,持久化没保证
no-appendfsync-on-rewrite  yes: #←导出rdb快照的过程中,是否停止同步aof
auto-aof-rewrite-percentage 100#←aof文件大小比起上次重写时的大小,增长率100%时,重写
auto-aof-rewrite-min-size 64mb #←aof文件,至少超过64M时,重写
```

### 2.3 测试

> **使用for循环批量更改一个key的值**

```bash
for n in {1..8888};do redis-cli set name $n;done
```

> **此时redis服务端的信息**

```bash
4330:M 30 Mar 02:15:23.581 * Starting automatic rewriting of AOF on 101% growth
4330:M 30 Mar 02:15:23.581 * Background append only file rewriting started by pid 11921
4330:M 30 Mar 02:15:24.598 * AOF rewrite child asks to stop sending diffs.
11921:C 30 Mar 02:15:24.598 * Parent agreed to stop sending diffs. Finalizing AOF...
11921:C 30 Mar 02:15:24.598 * Concatenating 0.02 MB of AOF diff received from parent.
11921:C 30 Mar 02:15:24.606 * SYNC append only file rewrite performed
11921:C 30 Mar 02:15:24.606 * AOF rewrite: 0 MB of memory used by copy-on-write
4330:M 30 Mar 02:15:24.685 * Background AOF rewrite terminated with success
4330:M 30 Mar 02:15:24.685 * Residual parent diff successfully flushed to the rewritten AOF (0.00 MB)
```

> **此时日志文件大小**

```bash
[root@Lnmp redis]# ll -h
total 132K
-rw-r--r--. 1 root root  59K Mar 30 02:15 appendonly.aof
-rw-r--r--. 1 root root   56 Mar 30 02:15 temp-rewriteaof-17659.aof
[root@Lnmp redis]# ll -h
-rw-r--r--. 1 root root  22K Mar 30 02:15 appendonly.aof
```
### 2.4 手动生成新的AOF文件

```bash
bgrewriteaof
```

## 3 日志重写

aof的方式也同时带来了另一个问题。持久化文件会变的越来越大。例如我们调用<font style="background:#bafe01;" size=2>incr test命令100次</font>，文保中必须保存全部的100条命令，<font style="background:#bafe01;" size=2>其实有99条都是多余的</font>。因为要恢复数据库的状态其实文件中保存一条set test 100就够了。为了压缩aof的持久化文件，Redis提供了bgrewriteaof命令.收到此命令Redis将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件。

自动bgrewriteaof参数设置

```sh
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64m
```

## 4 RDB和AOF ,应该用哪一个？

一般来说,如果想达到足以媲美 PostgreSQL 的数据安全性，你应该同时使用两种持久化功能。如果你非常关心你的数据,但仍然可以承受数分钟以内的数据丢失，那么你可以只使用 RDB 持久化。有很多用户都只使用 AOF 持久化，并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且RDB恢复数据集的速度也要比AOF恢复的速度要快， 除此之外， 使用RDB还可以避免之前提到的AOF程序的bug 。因为以上提到的种种原因， <font style="background:#bafe01;" size=2>未来Redis可能会将AOF和RDB整合成单个持久化模型。（这是一个长期计划）</font>。
