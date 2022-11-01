# 

[toc]

### 1.1 为Redis客户端外部设置连接密码

因为redis速度相当快，所以在一台比较好的服务器下，一个外部的用户可在一秒钟进行上万次的密码尝试，这意味着你需要指定非常非常强大的密码来防止暴力破解。

#### 1.1.1 修改配置文件

```bash
requirepass 123@1
```

**重启服务后登录客户端提示没有验证**

```bash
[root@redis ~]# redis-cli
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
```

**验证成功后，可以正常操作**

```bash
127.0.0.1:6379> auth 123@1
OK
127.0.0.1:6379> keys *
1) "test-durable-1"
2) "test-durable"
```

#### 1.1.2 命令行临时生效

***在命令行设置后，redis在下次重启前，每次登录都需要验证密码***

```bash
127.0.0.1:6379> CONFIG set requirepass 123@1
OK
127.0.0.1:6379> quit
[root@redis ~]# redis-cli
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
```

***
<font color=#360c9f size=2>**注意：配置Redis复制的时候如果主数据库设置了密码，需要在从数据库的配置文件中通过masterauth参数设置主数据库的密码，以使从数据库连接主数据库时自动使用AUTH命令认证。**</font>
***

**通过mysql命令行指定密码方式登录Redis客户端**

```bash
[root@redis ~]# redis-cli -a 123@1
127.0.0.1:6379> keys *
1) "test-durable-1"
2) "test-durable"
```

### 1.2 危险命令重命名

```bash
rename-command flushall abc
rename-command get eee
rename-command FLUSHALL ""  #←禁用FLUSHALL命令
```

```bash
127.0.0.1:6379> get test-durable
(error) ERR unknown command 'get'
127.0.0.1:6379> eee test-durable
"test1"
```
### 1.3 绑定只能本机连接

Redis的默认配置会接受来自任何地址发送来的请求，即在任何一个拥有公网IP的服务器上启动Redis服务器，都可以被外界直接访问到。要更改这一设置，在配置文件中修改bind参数，如只允许本机应用连接Redis.

```sh
bind 127.0.0.1
```
