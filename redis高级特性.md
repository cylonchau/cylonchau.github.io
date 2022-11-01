# 

[toc]

## 1 发布与订阅

### 1.1 Publish/Subscribe

发布订阅(pub/sub)是一种消息通信模式，主要的目的是解耦消息发布者和消息订阅者之间的藕合，这点和设计模式中的观察者模式比较相似。pub/sub不仅仅解决发布者和订阅者直接代码级别耦合也解决两者在物理部署上的耦合。Redis作为一个pub/sub的server,在订阅者和发布者之间起到了消息路由的功能。订阅者可以通过subscribe和psubscribe命令向Redis server订阅自己感兴趣的消息类型，Redis将消息类型称为通道(channel)。当发布者通过publish命令向Redis server发送特定类型的消息时。订阅该消息类型的全部client

都会收到此消息。这里消息的传递是多对多的。一个client可以订阅多个channel,也可以向多个channel发送消息。

Redis支持这样一种特性，你可以将数据推到某个信息管道中，然后其它人可以通过订阅这些管道来获取推送过来的信息。

### 1.2 用一个客户端订阅频道

```bash
psubscribe new
#### 1.1.3 批量订阅
127.0.0.1:6379> publish news news-test
(integer) 1
127.0.0.1:6379> publish video video-test
(integer) 1
```

**此时可以见到接受的信息**

```bash
127.0.0.1:6379> psubscribe news video
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "news"
3) (integer) 1
1) "psubscribe"
2) "video"
3) (integer) 2
1) "pmessage"
2) "news"
3) "news"
4) "news-test"
1) "pmessage"
2) "video"
3) "video"
4) "video-test"
```

## 2 数据过期设置及机制

### 2.1 Redis key的过期机制

Redis对过期键采用了lazy expiration：在访间key的时候判定key是否过期，如果过期，则进行过期处理<font style="background:#bafe01;" size=2>（过期的key没有被访间可能不会被删除）</font>。其次，每秒对volatile keys进行抽样测试，如果有过期键，那么对所有过期key进行处理。

### 2.2 expire设置key的生命周期

**使用ttl命令可以获得某个key的过期时间**

```bash
127.0.0.1:6379> ttl test 
(integer) -1  #<== -1代表永久
127.0.0.1:6379> ttl test1
(integer) -2  #<==redis2.8后不存在的key返回-2
```

**expire：设置一个key的生命周期，单位秒**

```bash
127.0.0.1:6379> expire site 10240000  #<==设置key多长时间后过期,单位秒
(integer) 1
127.0.0.1:6379> ttl site
(integer) 10239996
```

**pexpire：以毫秒数设置key的生命周期**

```bash
127.0.0.1:6379> pexpire a 10000
(integer) 1
127.0.0.1:6379> ttl a
(integer) 8
```

**pttl：以毫秒返回生命周期**

```bash
127.0.0.1:6379> pttl a
(integer) 5432
```

**persist：将key设置为永久有效**

```bash
127.0.0.1:6379> pexpire a 10000
(integer) 1
127.0.0.1:6379> persist a
(integer) 1
127.0.0.1:6379> ttl a
(integer) -1
```

### 2.3 设置指定时间过期
unix时间戳转换：http://tool.chinaz.com/Tools/unixtime.aspx

linux下获得unix时间戳

```bash
[root@redis ~]# date +%s  #<==获得当前时间的时间戳
1492845155  
[root@redis ~]# date +%s -d '2017-4-27 18:10' #<==获得指定时间的时间戳
1493287800
```

### 2.4 创建key时设置生命周期

```bash
127.0.0.1:6379> set test-expire test-tmp ex 10  #<==ex/px 秒/毫秒
OK
127.0.0.1:6379> ttl test-expire
(integer) 4 
```

## 3 事务

redis对事务的支持目前还比较简单。redis只能保证一个client发起的事务中的命令可以连续的执行，而中间不会插入其他client的命令。 由于redis是单线程来处理所有client的请求的，所以做到这点是很容易的。一般情况下redis在接受到一个client发来的命令后会立即处理并返回处理结果，但是当一个client在一个连接中发出multi命令有，这个连接会进入一个事务上下文，该连接后续的命令并不是立即执行，而是先放到一个队列中。当从此连接受到exec命令后，redis会顺序的执行队列中的所有命令。并将所有命令的运行结果打包到一起返回给client，然后此连接就结束事务上下文。
### 3.1 用法


***选项详解***
|命令 | 描述|
|---|---|
|MULTI|于开启一个事务，它总是返回 OK |
|DISCARD	|清空事务队列， 并放弃执行事务。|
|EXEC	|负责触发并执行事务中的所有命令|
|WATCH	 |为Redis 事务提供 check-and-set （CAS）行为；<br>被WATCH的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监执行之前被修改了， 那么整个事务都会被取消。|

```bash
127.0.0.1:6379> set num 1000
OK
127.0.0.1:6379> set num2 2000
OK
127.0.0.1:6379> multi  #<==开启队列
OK
127.0.0.1:6379> decrby num2 1000 #<==将num2的值减少1000
QUEUED #<==当客户端处于事务状态时,所有传入的命令都会返回一个内容为QUEUED的状态回复
127.0.0.1:6379> get num2
"1000" #<==这时在当前查看num2的值，发现已经被更改了
127.0.0.1:6379> get num2 
"2000" #<==此时在其他客户端查看num2的值
127.0.0.1:6379> exec #<==执行队列中的语句
1)(integer) 1000 
127.0.0.1:6379> get num2 #<==这时在其他客户端查看num2
"1000"
```

***
注：即使事务中有某条/某些命令执行失败了， 事务队列中的其他命令仍然会继续执行，Redis 不会停止执行事务中的命令。
***

```bash
127.0.0.1:6379> decrby num1 5
QUEUED
127.0.0.1:6379> decrby num2 aaa
QUEUED
127.0.0.1:6379> exec
1) (integer) -5
2) (error) ERR value is not an integer or out of range
```
### 3.2 放弃事务

执行 DISCARD 命令，事务会被放弃，事务队列会被清空，并且客户端会从事务状态中退出

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> keys *
QUEUED
127.0.0.1:6379> discard
OK
127.0.0.1:6379> keys *
1) "num1"
2) "num2"
```

> ***使用 check-and-set 操作实现乐观锁***

被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消， EXEC 返回空多条批量回复（null multi-bulk reply）来表示事务已经失败。

```bash
127.0.0.1:6379> watch num1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr num1 #<==在其它客户端修改num1
(integer) 101
127.0.0.1:6379> incr num1
QUEUED
127.0.0.1:6379> exec
(nil)
```

参考资料http://doc.redisfans.com/topic/transaction.html
