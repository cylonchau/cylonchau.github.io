# ch7 关于iptables的内核参数


调整内核参数文件/etc/sysctl.conf，以下是我们生产环境的某个服务器的配置：

```bash
# 表示如果套接字由本端要求关闭，这个檀树决定了他保持在FIN-WAIT-2状态的时间。
net.ipv4.tcp_fin_timeout = 2
# 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭。
net.ipv4.tcp_tw_reuse = 1
# 表示开启TCP连接中TIME-WAIT socket的快速收回，默认为0，表示关闭
net.ipv4.tcp_tw_recycle = 1
提示：以上两个参数为了防止生产环境下 time_wait过多设置的。
############################################################
# 表示开启SYN Cookie。当出现SYN等带队列溢出时，启动cookie来处理，可防范少量SYN攻击，默认为0表示关闭
net.ipv4.tcp_syncookies = 1
# 表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是两小时，改为20分钟 单位秒
net.ipv4.tcp_keepalive_time = 1200
# 表示对用向外连接的端口范围。缺省情况下很小。
net.ipv4.ip_local_port_range = 4000  65000
# 表示SYN队列的长度，默认为1024，加大队列长度为8192，可容纳更过等待连接的网络连接数。
net.ipv4.tcp_max_syn_backlog = 16384
# 表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清楚并打印警告信息。默认为180000，对于Apache Nginx等服务器来说可以调低一点，如：改为5000-30000，不同业务的服务器也可以给大一点，比如LVS，squid
以上几行的参数可以很好的减少TIME_WAIT套接字数量，但对于squid效果却不大。
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.route.gc_timeout = 100
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
# 以下参数是对iptables防火墙的优化，防火墙不会开提示，可以忽略不理。
net.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_tcp_timeout_established = 180
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
```

dmesg里面显示 `ip_contrack:table full`，``dropping packet.` 的错误提示，如何解决?

这有两个可能，一个是打开的端口太少至不够用，修改ip_conntrack文件为1024 65535。

还有一个原因是nat链接真的达到65535了。此时就把NAT映射表保持时间设置短一些。

**强调**：如果并发比较大，或者日PV多的情况下，开启防火墙要注意，很可能导致网站访问缓慢。

大并发（并发1万，PV日3000万）要么购买硬件防火墙，要么不开iptables防火墙。


