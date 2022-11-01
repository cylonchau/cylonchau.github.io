# 

[toc]

Remote Dictonary Server(Redis)是一个基于key-value键值对的持久化数据库存储系统。redis和大名鼎鼎的Memcached缓存服务很像,但是redis支持的数据存储类型更丰富,包括<font style="background:#bafe01;" size=2>string(字符串)、list(链表)、set(集合)和zset(有序集合)、Hash等</font>。

这些数据类型都支持push/pop,add/remove及取交集、并集和差集及更丰富的操作,而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached缓存服务一样，为了保证效率数据都是缓存在内存中提供服务。和memcached不同的是，<font style="background:#bafe01;" size=2>redis持久化缓存服务</font>还会周期性的把更新的数据写入到磁盘以及把修改的操作记录追加到文件里记录下来，比memcached更有优势的是，<font style="background:#bafe01;" size=2>redis还支持master-slave(主从)同步</font>,这点很类似关系型数据库MySQL。

Redis是一个开源的、使用C语言编写、３万多行代码、支持网络、可基于内存亦可久化的日志型、Key-Value数据库，并提供多种语言的API从2010年3月15日起，Redis开发工作由VMware主持。

Redis的出现，再一定程度上弥补了memcached这类key-value内存缓存服务的不足,在部分场合可以对关系数据库起到很好的补充作用.redis提供了Python, Ruby, Erlang, PHP客户端，使用很方便。redis官方文档如下：http://www.redis.io/documentation

### 1.1 Redis的优点

* 与memcached不用，redis可以持久化存储数据。
* 性能很高：Redis能支持超过10w/秒的读写频率。
* 丰富的数据类型：redis支持二进制的Strings, Lists, Hashes, Sets及sorted sets等数据类型操作。
* 原子：Redis的所有操作都是原子性的,同时Redis还支持对几个操作全并后的原子性执行。
* 丰富的特性：Redis还支持publish/subscribe(发布/订阅)，通知，key过期等等特性。
* redis支持异步主从复制。

### 1.2 Redis的应用场景

**传统的MySQL+Memcached的网站架构遇到的问题：**

MySQL数据库实际上是适合进行海量数据存储的,加上通过Memcached将热点数据  存放到到内存cache里,达到加速数据访问的目的,绝大部分公司都曾经使用过这样的架构,但随着业务数据量的不断增加,和访问量的增长,很多问题就会暴漏出来：

1.  需要不断的对MySQL进行拆库拆表Memcached也需不断跟着扩容,扩容和维护工作占据大量开发运维时间。
2.  Memcached与MySQL数据库数据一致性问题是个老大难。
3.  Memcached数据命中率低或down机,会导致大量访问直接穿透到数据库,导致MySQL无法支撑访问。
4.  跨机房cache同步一致性问题。

#### 1.2.1 redis在微博中的应用

1. 计数器：微博（评论、转发、阅读、赞等）
2. 用户（粉丝、关注、收藏、双向关注等）

#### 1.2.2 redis在短信中的应用

发送短信后存入redis中60秒过期。

### 1.3 redis的最佳应用场景

1.  Redis最佳试用场景是全部数据in-memory。
2.  Redis更多场景是作为Memcached的替代品来使用。
3.  当需要除key/value之外的更多数据类型支持时,使用Redis更合适。
4.  数据比较重要，对数据一致性有一定要求的业务。
5.  当存储的数据不能被剔除时,使用Redis更合适。
6.  更多

* <font color=#2b01fe size=2>Redis作者谈Redis应用场景</font>
http://blog.nosglfan.com/html/2235.html
* <font color=#2b01fe size=2>使用redis bitmap进行活跃用户统计</font>
http://blog.nosqlfun.com/html/3501.html

计数、cache服务、展示最近、最热、点击率最高、活跃度最高等等条件的top list、用户最近访问记录表、relation list/Message Queue、粉丝列表

Key-Value Store更加注重对海量数据存取的性能、分布式、扩展性支持上，并不需要传统关系数据库的一些特征。例如：Schema事务、完整SQL查询支持等等，因此在布式环境下的性能相对于传统的关系数据库有较大的提升。

### 1.4 redis的生产经验教训

1.  要进行Master-slave主从同步配置，在出现服务故障时可以切换。
2.  在master禁用数据据持久化只需在slave上配置数据持久化。
3.  物理内存+虚拟内存不足，这个时候dump一直死着，时间久了机器挂掉。这个情就是灾难。
4.  当Redis物理内存使用超过内存总容量的3/5时就会开始比较危险了，就开始做swap，内存碎片大！
5.  当达到最大内存时，会清空带有过期时间的如
6.  redis与DB同步写的问题，先写DB，后写redis，因为写内存基本上没有问题。

### 1.5 业务场景

1.  提高了DB的可扩展性,只需要将新加的数据放到新加的服务器上就可以了。
2.  提高了DB的可用性,只影响到需要访问的shard服务器上的数据的用户。
3.  提高了DB的可维护性,对系统的升级和配里可以按shard一个个来搞,对服务产生的影响小。
4.  小的数据库存的查询压力小,查询更快,性能更好。

> **使用过程中的一些经验与教训,做个小结：**

1. 要进行Master-slave配置,出现服务故障时可以支持切换。
2. 在master侧禁用数据持久化,只需在slave上配置数据持久化。
3. 物理内存+虚拟内存不足时,这个时候dump已知死着,时间久了机器挂掉。这个情况就是灾难。
4. 当Redis物理内存使用超过内存总容量的3/5时就会开始比较危险了,就开始做swap,内存碎片大。
5. 当达到最大内存时,会清空带有过期时间的key,即使key未到过期时间。
6. redis与DB同步写的问题,先写DB,后写redis,因为写内存基本上没有问题。



## 2 安装配置Redis



### 2.1 下载安装Redis

redis官方网站：www.redis.io

```bash
make
make PREFIX=/app/redis-3.2.8 install 
```

命令执行完成后,会在/app/redis-3.2.8/bin/目录下生成5个可执行文件

```bash
/app/redis-3.2.8/
└── bin
   ├── redis-benchmark #← redis性能测试工具,测试redis在你的系统及你的配置下读写性能
   ├── redis-check-aof #← 更新日志检查
   ├── redis-check-rdb 
   ├── redis-cli      #← redis命令行操作工具。当然也可用telnet根据其纯文本协议来操作
   └── redis-server   #←redis服务器的daemon启动服务
```
### 2.2 配置并启动Redis服务

#### 2.2.1 文件头部分

操作命令：

```bash
echo 'PATH="/app/redis/bin:$PATH"' >>/etc/profile
. /etc/profile
```

#### 2.2.2 查看命令帮助

```bash
Usage: ./redis-server [/path/to/redis.conf] [options]
       ./redis-server - (read config from stdin)
       ./redis-server -v or --version
       ./redis-server -h or --help
       ./redis-server --test-memory <megabytes> 

Examples:
       ./redis-server (run the server with default conf)
       ./redis-server /etc/redis/6379.conf
       ./redis-server --port 7777 #<==指定端口
       ./redis-server --port 7777 --slaveof 127.0.0.1 8888 
       ./redis-server /etc/myredis.conf --loglevel verbose #<==指定配置文件

Sentinel mode:
       ./redis-server /etc/sentinel.conf --sentinel
```

### 2.3 启动redis服务

创建配置文件,redis的配置文件在二进制安装包解压目录下的 redis.conf

```bash
mkdir /app/redis-3.2.8/conf
cp /root/tools/redis-3.2.8/redis.conf /app/redis-3.2.8/conf/
```

启动命令

```bash
redis-server /app/redis-3.2.8/conf/redis.conf &
```

### 2.4 启动错误

```bash
WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
```

原因：此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度, 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值,默认是511,而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候,可以将这二个参数一起参考设定。

暂时解决：redis.conf中 tcp backlog=128

```bash
WARNING overcommit_memory is set to 0! Background save may fail under low memory condition.To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
```

原因：

**overcommit_memory参数说明：**

设置内存分配策略（可选,根据服务器的实际情况进行设置）/proc/sys/vm/overcommit_memory可选值：0、1、2。

0：当用户空间请求更多的内存时，内核尝试估算出剩余可用的内存。

1：设这个参数值为1时，内核允许超量舒勇内存直到用完为止,主要用于科学计算。(最大限度的使用内存)

2：当设这个参数值为2时，内核会使用一个决不过量使用内存的算法，即系统整个内存地址空间不能超过swap+50%的RAM值，50%参数的设定是在overcommit_ratio中设定。

**解决：sysctl vm.overcommit_memory=1***

```bash
WARNING you have Transparent Huge Pages (THP) support enabled in your kernel.
This will create latency and memory usage issues with Redis. 
To fix this issue run the command 
'echo never > /sys/kernel/mm/transparent_hugepage/enabled' 
as root, 
and add it to your /etc/rc.local in order to retain the setting after a reboot.
Redis must be restarted after THP is disabled.
```

原因

临时解决方法：

```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

参考文档：http://blog.csdn.net/a491857321/article/details/52006376

### 2.5 客户端测试redis服务

#### 2.5.1 redis-cli客户端帮助

```bash
redis-cli -help
```

#### 2.5.2 使用redis-cli客户端连接redis

**指定密码 ip 端口登陆**

```bash
redis-cli -h 10.0.0.10 -p 3307 -a '111'
Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -s <socket>        Server socket (overrides hostname and port).
```

**在Linux命令行操作redis**

```bash
[root@redis ~]# redis-cli set zhangsan 10-10
OK
[root@redis ~]# redis-cli get zhangsan
"10-10"
```

#### 2.5.3 使用telnet连接redis

```bash
[root@redis ~]# telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get name
$2
45
```

#### 2.5.4 通过NC连接redis

```bash
[root@redis ~]# echo "set three-world 刘备"|nc 127.0.0.1 6379
+OK
[root@redis ~]# echo "get three-world"|nc 127.0.0.1 6379
$6
刘备
```

### 2.6 关闭redis服务

```bash
redis-cli shutdown
```
通过客户端测试redis服务

### 2.7 PHP安装Redis扩展

**源码地址：**

https://github.com/phpredis/phpredis

http://pecl.php.net/package/redis

**安装**

```bash
/app/php/bin/phpize
./configure --with-php-config=/app/php/bin/php-config
```

**修改php.ini设置**

```bash
extension=redis.so
```

**查看结果**

![img](../../../images/redis/wps1.jpg)

## 3 Redis配置文件注解

```bash
# 是否在后台执行,yes：后台运行；no：不是后台运行（老版本默认）
daemonize yes

# 此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度, 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值,默认是511,而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候,可以将这二个参数一起参考设定。该内核参数默认值一般是128,对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。在/etc/sysctl.conf中添加:net.core.somaxconn = 2048,然后在终端中执行sysctl -p。
tcp-backlog 511

# 指定 redis 只接收来自于该 IP 地址的请求,如果不进行设置,那么将处理所有请求
bind 127.0.0.1

# 此参数为设置客户端空闲超过timeout,服务端会断开连接,为0则服务端不会主动断开连接,不能小于0。
timeout 0

# 指定了服务端日志的级别。级别包括：debug（很多信息,方便开发、测试）,verbose（许多有用的信息,但是没有debug级别信息多）,notice（适当的日志级别,适合生产环境）,warn（只有非常重要的信息）	
loglevel notice
# 指定了记录日志的文件。空字符串的话,日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null。
logfile /var/log/redis/redis-server.log

# 注释掉"save"这一行配置项就可以让保存数据库功能失效
# 设置sedis进行数据库镜像的频率。
# 900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化） 
# 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化） 
# 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）
save 900 1
save 300 10
save 60 10000
```
更多配置详情：http://www.cnblogs.com/zhang-ke/p/5981108.html
