# etcd


[toc]

##  概述

etcd 是兼具一致性和高可用性的键值数据库，为云原生架构中重要的基础组件，由`CNCF `孵化托管。etcd 在微服务和 Kubernates 集群中不仅可以作为服务注册与发现，还可以作为 key-value 存储的中间件。

### 先决条件

- 运行的 etcd 集群个数成员为奇数。
- etcd 是一个 leader-based 分布式系统。确保主节点定期向所有从节点发送心跳，以保持集群稳定。
- 保持稳定的 etcd 集群对 Kubernetes 集群的稳定性至关重要。因此，请在专用机器或隔离环境上运行 etcd 集群，以满足所需资源需求]。
- 确保不发生资源不足。&lt;br&gt;集群的性能和稳定性对网络和磁盘 IO 非常敏感。任何资源匮乏都会导致心跳超时，从而导致集群的不稳定。不稳定的情况表明没有选出任何主节点。在这种情况下，集群不能对其当前状态进行任何更改，这意味着不能调度新的 pod。

## 相关术语

- `Raft`：etcd所采用的保证分布式系统强一致性的算法。
- `Node`：节点 ，Raft状态机的一个实例，具有唯一标识。
- `Member`：  成员，一个etcd实例。承载一个Node，且可为客户端请求提供服务。
- `Cluster`：集群，由多个Member构成可以协同工作的etcd集群。
- `Peer`：同伴，`Cluster`中其他成员。
- `Proposal `：提议，一个需要完成 raft 协议的请求(例如写请求，配置修改请求)。
- `Client`： 向etcd集群发送HTTP请求的客户端。
- `WAL`：预写式日志，etcd用于持久化存储的日志格式。
- `snapshot`：etcd防止WAL文件过多而设置的快照，存储etcd数据状态。
- `Proxy`：etcd的一种模式，为etcd集群提供反向代理服务。
- `Leader`：Raft算法中通过竞选而产生的处理所有数据提交的节点。
- `Follower`：竞选失败的节点作为Raft中的从属节点，为算法提供强一致性保证。
- `Candidate`：当Follower超过一定时间接收不到Leader的心跳时转变为Candidate开始竞选。
- `Term`：某个节点成为Leader到下一次竞选时间，称为Ubuntu一个Term。
- `Index`：数据项编号。Raft中通过Term和Index来定位数据。

## ETCD 部署

### 源码安装

基于master分支构建etcd

```bash
git clone https://github.com/etcd-io/etcd.git
cd etcd
./build # 如脚本格式为dos的，需要将其格式修改为unix，否则报错。
```

启动命令

`--listen-client-urls` 于 `--listen-peer-urls` 不能为域名

`--listen-client-urls` 于 `--advertise-client-urls` 

```sh
 ./etcd --name=etcd \
 --data-dir=/var/lib/etcd/ \
 --listen-client-urls=https://10.0.0.1:2379 \
 --listen-peer-urls=https://10.0.0.1:2380 \
 --advertise-client-urls=https://hketcd:2379 \
 --initial-advertise-peer-urls=https://hketcd:2380 \
 --cert-file=&#34;/etc/etcd/pki/server.crt&#34; \
 --key-file=&#34;/etc/etcd/pki/server.key&#34; \
 --client-cert-auth=true \
 --trusted-ca-file=&#34;/etc/etcd/pki/ca.crt&#34; \
 --auto-tls=false \
 --peer-cert-file=&#34;/etc/etcd/pki/peer.crt&#34; \
 --peer-key-file=&#34;/etc/etcd/pki/peer.key&#34; \
 --peer-client-cert-auth=true \
 --peer-trusted-ca-file=&#34;/etc/etcd/pki/ca.crt&#34; \
 --peer-auto-tls=false
```

### 其他方式

- CentOS 可以使用 `yum install etcd -y`
- Ubuntu 可以预构建的[二进制文件](https://github.com/etcd-io/etcd/releases/)

### 安装报错

&gt; certificate: x509: certificate specifies an incompatible key usage
&gt;
&gt; 原因：此处证书用于`serverAuth` 与`clientAuth`，缺少`clientAuth`导致
&gt;
&gt; 解决：`extendedKeyUsage=serverAuth， clientAuth` 

```bash
WARNING: 2020/11/12 14:11:42 grpc: addrConn.createTransport failed to connect to {0.0.0.0:2379  &lt;nil&gt; 0 &lt;nil&gt;}. Err: connection error: desc = &#34;transport: authentication handshake failed: remote error: tls: bad certificate&#34;. Reconnecting...
{&#34;level&#34;:&#34;warn&#34;,&#34;ts&#34;:&#34;2020-11-12T14:11:46.415&#43;0800&#34;,&#34;caller&#34;:&#34;embed/config_logging.go:198&#34;,&#34;msg&#34;:&#34;rejected connection&#34;,&#34;remote-addr&#34;:&#34;127.0.0.1:52597&#34;,&#34;server-name&#34;:&#34;&#34;,&#34;error&#34;:&#34;tls: failed to verify client certificate: x509: certificate specifies an incompatible key usage&#34;}
```

&gt;原因：证书使用的者不对。
&gt;
&gt;解决：查看`subjectAltName` 是否与请求地址一致。

```sh
error &#34;tls: failed to verify client&#39;s certificate: x509: certificate specifies an incompatible key usage&#34;, ServerName &#34;&#34;
```

&gt;原因：`ETCD_LISTEN_PEER_URLS` 与 `ETCD_LISTEN_CLIENT_URLS` 不能用域名

```
error verifying flags, expected IP in URL for binding (https://hketcd:2380). See &#39;etcd --help&#39;
```

&gt;error #0: x509: certificate has expired or is not yet valid
&gt;
&gt;原因：证书还未生效
&gt;
&gt;解决：因服务器时间不对导致，校对时间后正常

```
etcd: rejected connection from &#34;x.x.x.x:1126&#34; (error &#34;tls: failed to verify client&#39;s certificate: x509: certificate signed by unknown authority (possibly because of \&#34;crypto/rsa: verification error\&#34; while trying to verify candidate authority certificate \&#34;etcd-ca\&#34;)&#34;, ServerName &#34;&#34;)
```

## 配置文件详解

[config](https://etcd.io/docs/v3.3.13/op-guide/configuration/)

## etcdctl 使用

```sh
etcdctl --key-file=/etc/etcd/pki/client.key \
--cert-file=/etc/etcd/pki/client.crt \
--ca-file=/etc/etcd/pki/ca.crt  \
--endpoint=&#34;https://node01.k8s.test:2379&#34; \
cluster-health

member 288506ee270a7733 is healthy: got healthy result from https://node03.k8s.test:2379
member 863156df9b1575d1 is healthy: got healthy result from https://node02.k8s.test:2379
member ff386de9dc0b3c40 is healthy: got healthy result from https://node01.k8s.test:2379
```

v3 版本客户端使用

```sh
export ETCDCTL_API=3

etcdctl --key=/etc/etcd/pki/client.key \
--cert=/etc/etcd/pki/client.crt \
--cacert=/etc/etcd/pki/ca.crt \
--endpoints=&#34;https://master.k8s:2379&#34; \
endpoint health


etcdctl \
--key=/etc/etcd/pki/client.key \
--cert=/etc/etcd/pki/client.crt \
--cacert=/etc/etcd/pki/ca.crt \
--endpoints=&#34;https://master.k8stx.com:2379&#34; \
endpoint status
```

### 日志独立

etcd日志默认输出到 `/var/log/message` 如果想独立日志为一个文件，可以使用rsyslogd过滤器功能，使etcd的日志输出到单独的文件内。

1. **新建`/etc/rsyslog.d/xx.conf`文件**。
2. **在新建文件内写入内容如下**

```
if $programname == &#39;etcd&#39; then /var/log/etcd.log
# 停止往其他文件内写入，如果不加此句，会继续往/var/log/message写入。
if $programname == &#39;etcd&#39; then stop  
```

也可以

```
if ($programname == &#39;etcd&#39;) then {
   action(type=&#34;omfile&#34; file=&#34;/var/log/etcd.log&#34;)
   stop
}
```

