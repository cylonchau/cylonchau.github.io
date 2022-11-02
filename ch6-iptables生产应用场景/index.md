# ch6 iptables生产应用场景



&gt; 1、局域网共享上网（适合做企业内部局域网上网网关，以及IDC机房内网的上网网关 nat POSTROUTING）


&gt; 2、服务器防火墙功能（适合IDC机房具有外网IP服务器，主要是filter INPUT的控制）


&gt; 3、把外部IP及端口映射到局域网内部（可以一对一IP映射，也可针对某一个端口映射。）
&gt;
&gt; 也可能是IDC把网站的外网VIP级网站端口映射到负载均衡器上（硬件防火墙）（NAT PREROUTING）


&gt; 4、办公路由器&#43;网关功能（zebra路由&#43;iptables过滤及NAT&#43;squid正向透明代理80&#43;ntop/iftop/iptaf流量查看&#43;tc/cbq流量控制限速）。



&gt; 5、邮件的网关。

&lt;font style=&#34;background:#bafe01;&#34; size=2&gt;问题2：的生产环境应用：用于没有外网地址的内网服务器，映射为公网IP后对外提供服务，也包括端口的映射&lt;/font&gt;

&lt;font style=&#34;background:#bafe01;&#34; size=2&gt;问题3：IP一对一映射&lt;/font&gt;
用于没有外网地址的内网服务器，映射为公网IP后对外提供服务，例如：ftp服务要一对一IP映射。

共享上网封IP的方法：

```bash
/sbin/iptables -I FROWAED -s 10.0.0.26 -j　DROP
/sbin/iptables ${deal} FROWARD -m mac --mac -source ${strIpMac} -j DROP
```

#### 映射多个外网IP上网

```bash
iptables -t nat -A POSTROUTING -s 10.0.1.0/255.255.240.0 -o eth0 -j SNAT --to-source 124.42.60.11-124.42.60.16
iptables -t nat -A POSTROUTING -s 172.168.1.0/255.255.255.0 -o eth0 -j SNAT --to=source 124.42.60.60-124.42.60.63
```

问题：公司内网主机多的时候，访问网站容易被封。


