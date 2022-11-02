# 


高可用分布式key value存储，可以用于配置共享和服务发现

类似项目 zookeeper consul


### raft

raft 强一致性的集群日志同步算法，etcd利用raft算法在集群中同步key-value

基于raft算法的强一致性、高可用的服务存储目录。

raft是强一致的集群日志同步算法，通过将数据写入日志，再将日志复制给其余节点。

### quorum 模型


etcd有调用者、leader节点，由多个节点组成。遵守raft协议，集群中至少有2N&#43;1个节点，1个leader 2个 follower。

调用者的写入请求发给leader，leader将日志向follower做实时复制（第一阶段，日志的复制），当日志复制给N&#43;1（N&#43;1指的是大多数）个节点后，本地提交，返回客户端。leader会周期性一部通知follower完成其本地提交

- replication：日志在leader生成，向follower复制，达到各个节点的日志序列最终一致
- term：任期，重新选举产生的leader，其term单调递增
- log index，日志行在日志序列的下标



- 选举leader需要半数以上节点参与
- 节点commit日志最多的允许选举为leader
- commit日志同样多，则term、index越大的允许选举为leader

![1569720641355](../images/etcd/1569720641355.png)

raft保证

- 提交成功的请求，一定不会丢
- 各个节点的数据将最终一致

[etcd Admin Guide \| etcd Configuration | CoreOS](https://coreos.com/etcd/docs/latest/v2/admin_guide.html)

```
nohup /usr/local/bin/etcd --listen-client-urls &#39;http://0.0.0.0:2379&#39; --advertise-client-urls &#39;0.0.0.0:2379&#39; &amp;
```


```sh
ETCDCTL_API=3 ./etcdctl
```

```sh
root@lc-virtual-machine:~# etcdctl put &#34;name&#34; &#34;zhangsan&#34;
OK

root@lc-virtual-machine:~# etcdctl get &#34;name&#34;
name
zhangsan

root@lc-virtual-machine:~# etcdctl del name
1
```

etcd不理解目录结构`/cron/jobs/job2` 认为其就是一个字符串 , etcd中key是有序的，相同前缀的可以排列在一起


```sh
lc@lc-virtual-machine:~$ etcdctl put &#34;/cron/jobs/job2&#34; &#34;{....aaa}&#34;
OK
lc@lc-virtual-machine:~$ etcdctl put &#34;/cron/jobs/job2&#34; &#34;{....bbb}&#34;
OK

lc@lc-virtual-machine:~$ etcdctl get &#34;/cron/jobs/job2&#34;
/cron/jobs/job2
{....bbb}
```

--from-key 从哪个key开始扫描

--prefix 获取满足key前缀的所有key







watch



window1 

```sh
lc@lc-virtual-machine:~$ etcdctl watch &#34;/cron/jobs&#34; --prefix
PUT
/cron/jobs/job3
{....ccc}
DELETE
/cron/jobs/job3
```

window2

```sh
lc@lc-virtual-machine:~$ etcdctl put &#34;/cron/jobs/job3&#34; &#34;{....ccc}&#34;
OK
lc@lc-virtual-machine:~$ etcdctl del &#34;/cron/jobs/job3&#34; &#34;{....ccc}&#34;
1
lc@lc-virtual-machine:~$ 
```



create_revision 创建版本

mod_revision 修改版本

version 自从创建后修改的次数

```sh
lc@lc-virtual-machine:~/data/src/cron/cronexpr/etcd-crud$ go run main.go 
[key:&#34;/cron/jobs/jobn&#34; create_revision:9 mod_revision:13 version:5 value:&#34;bye&#34; ]
lc@lc-virtual-machine:~/data/src/cron/cronexpr/etcd-crud$ go run main.go 
[key:&#34;/cron/jobs/jobn&#34; create_revision:9 mod_revision:13 version:5 value:&#34;bye&#34; ]
```





etcd restful
