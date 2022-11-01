# 

## 1 Replication的工作原理

设置一个Slave，无论是第一次还是重连到Master，它都会发出一个sync命令。当Master收到sync命令之后，会做两件事：

1.  Master执行BGSAVE，即在后台保存数据到磁盘（rdb快照文件）。
2.  Master同时将新收到的写入和修改数据集的命令存入缓冲区（非查询类）。

当Master在后台把数据保存到快照文件完成之后，把这个快照传送给Slave，而Slave则把内存清空后，加载该文件到内存中。而Master也会把此前收集到缓冲区中的命令，通过Reids命令协议形式转发给Slave，Slave执行这些命令，实现和Master的同步。Master/Slave此后会不断通过<font style="background:#edfb05;" size=2>异步方式进行命令的同步</font>。

![img](../../../images/replication/wps4.png)

***
<font color=#360c9f size=2>**注：在redis2.8之前，主从之间一旦发生重连都会引发全量同步操作。但在2.8之后版本，也可能是部分同步操作。**</font>
***

### 1.1 部分复制

2.8后，当主从之间的连接断开之后，他们之间可以采用持续复制处理方式代替采用全量同步。Master端为复制流维护一个内存缓冲区（in-memory backlog），记录最近发送的复制流命令；同时，Master和Slave之间都维护一个复制偏移量(replication offset)和当前Master服务器ID（Master run id）。当网络断开，Slave尝试重连时：


1.  如果MasterID相同（即仍是断网前的Master服务器），并且从断开时到当前时刻的历史命令依然在Master的内存缓冲区中存在，则Master会将缺失的这段时间的所有命令发送给Slave执行，然后复制工作就可以继续执行了

2.  否则，依然需要全量复制操作。

Redis 2.8 的这个部分重同步特性会用到一个新增的PSYNC内部命令， 而 Redis 2.8以前的旧版本只有SYNC命令，不过，只要从服务器是Redis 2.8或以上的版本，它就会根据主服务器的版本来决定到底是使用 PSYNC还是SYNC。
如果主服务器是 Redis 2.8 或以上版本，那么从服务器使用 PSYNC 命令来进行同步。
如果主服务器是 Redis 2.8 之前的版本，那么从服务器使用 SYNC 命令来进行同步。

## 2 redis主从同步特点

1. 一个Master可以有多个Slave。
2. Redis使用异步复制。从2.8开始，Slave会周期性（每秒一次）发起一个ack确认复制流（replication stream）被处理进度；
3. 不仅主服务器可以有从服务器， 从服务器也可以有自己的从服务器， 多个从服务器之间可以构成一个图状结构；
4. 复制在Master端是非阻塞模式的，这意味着即便是多个Slave执行首次同步时，Master依然可以提供查询服务；
5. 复制在Slave端也是非阻塞模式的：如果你在redis.conf做了设置，Slave在执行首次同步的时候仍可以使用旧数据集提供查询；你也可以配置为当Master与Slave失去联系时，让Slave返回客户端一个错误提示；
6. 当Slave要删掉旧的数据集，并重新加载新版数据时，Slave会阻塞连接请求（一般发生在与Master断开重连后的恢复阶段）；
7. 复制功能可以单纯地用于数据冗余（data redundancy），也可以通过让多个从服务器处理只读命令请求来提升扩展性（scalability）： 比如说， 繁重的 SORT 命令可以交给附属节点去运行。
8. 可以通过修改Master端的redis.config来避免在Master端执行持久化操作（Save），由Slave端来执行持久化。

## 3 redis replication配置文件详解

```bash
slaveof [masterip] [masterport] #←该redis为slave ip和port是master的ip和port

masterauth <master-password> #←如果master设置了安全密码，此处为master的安全密码

slave-serve-stale-data yes#←当slave丢失master或同步正在进行时，如果发生对slave的服务请求：
slave-serve-stale-data no #←slave返回client错误:"SYNC with master in progress"
slave-serve-stale-data yes #←slave依然正常提供服务

slave-read-only yes #←设置slave不可以写数据，只能用于同步

repl-ping-slave-period 10 #←发送ping到master的时间间隔
repl-timeout 60 #←IO超时时间
repl-backlog-size 1mb #←backlog的大小，当从库连接不到主库时，backlog的队列能放多少
repl-backlog-ttl 3600 #←backlog的生命周期
min-slaves-max-lag 10 #←延迟小于min-slaves-max-lag秒的slave才认为是健康的slave
# 当master不可用,Sentinel会根据slave的优先级选举一个master。
# 最低的优先级的slave,当选master.而配置成0,永远不会被选举
slave-priority 100 
```

## 4 配置Replication

***当前生效：在命令行输入以下命令***

```bash
redis-cli slaveof 127.0.0.1 6379
````

***永久生效：修改配置文件"slaveof"选项***

```bash
slaveof [masterip] [masterport]
slaveof 127.0.0.1 6379
```

这样就可以保证Redis_6380服务程序在每次启动后都会主动建立与Redis_6379的Replication连接了。

***查看从库的同步情况***

```bash
127.0.0.1:6380> MONITOR   #←监听服务器实时收到的所有请求
OK
1492710733.401532 [0 127.0.0.1:6379] "PING" #← 从库会ping主库
1492710743.510808 [0 127.0.0.1:6379] "PING"
1492711152.901232 [0 127.0.0.1:6379] "sadd" "web_site" "www.baidu.com" "www.google.com" "www.qq.com"
```

***获取有关服务器的信息和统计信息***

```bash
127.0.0.1:6380> info Replication
Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:8999
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

***当前主从同步存在的问题***
由于master和slave服务器不是Redis自动选举产生，需要人工参与，因此主从倒换无法自动完成。这样就存在一个问题，什么时候以及由谁来触发倒换。redis2.8的master和slave服务器是Redis自动选举产生了。

redis高可用 http://www.cnblogs.com/Xrinehart/p/3501372.html
