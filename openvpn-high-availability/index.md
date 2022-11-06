# openvpn high-availability


## 方案1：在Vpn 客户端使用多个配置文件实现（由用户选择拨号）类

方案1：需求分析与操作过程讲解。

基本说明：

生产场景中比较规范的做法是让所有的VPN SERVER尽可能共享同一套 server，ca证书或者连接同一个认证系统（即使是跨机房）。这样只需要一份客户端认证和文件和多份指定不同vpn client的配置文件即可实现vpn的负载均衡了。

总结结论：

1）该负载均衡方案操作简单，不引入多余服务（后面的方案都会引入服务），因此不会增加多余的单点故障，当用户连接的vpn不能使用时，用户就可以人工选择拨号其他的VPN服务器。

2）如果使用者为公司内部工作人员，此种方案是值得推荐的。老男孩老师推荐。

3）从广义上讲这是在用户端实现的负载均衡方案，类似早期的华军下载站一样，由用户选择下载站点，而不是用什么智能DNS等复杂的业务模式。

缺点：当一个vpnserver不能使用时，不能自动连上别的vpn server。

## 方案2：通过在客户端配置文件实现负载均衡（客户端文件里随机连接服务器）

提示：同方案1，所有VPN SERVER 需要共享同一套 server，ca证书。openvpn 服务器一套keys的多份拷贝方式式。

```
remote 10.0.0.28 52115
remote 10.0.0.552115
remote-random
resolv-retry 20
```

[implementing-a-load-balancing-failover-configuration](https://openvpn.net/community-resources/how-to/#implementing-a-load-balancing-failover-configuration)

总结结论：

1）该负载均衡方案操作简单，不引入多余服务（后面的方案都会引入服务），因此不会增加多余的单点故障，，当用户连接的vpn不能使用时，电脑可以重新再次自动拨号连接VPN服务器。

2）如果使用者为公司内部工作人员，此种方案是值得推荐的。-老男孩老师推荐。如果是使用者为外部人员，那么这个方案依然是可以的。

3）本方案是比较标准的在VPN用户端，由客户端配置参数实现的负载均衡的方案，是非常值得推荐的方案。

4）和方案1对比，方案2的配置更简单，仅需一个配置文件多个remote参数，拨号时客户端会随机自动选择拨号，方案1则需要手动选择不同的配置文件拨号。当正在连接的VPN服务端右机时，那么此时方案2不需要人工干预，客户端的VPN会自动判断并且自动重新连接其他的可用vpn服务器。.

## 方案3：通过域名加DNS轮询的方式实现负载均衡（由DNS自动分配vpn）

![SOLVE)OVPN Load Balance Review | Netgate Forum](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1539219005583-ovpn-load-balance.png)



总结结论：

1）通过dns轮询实现VPN负载均衡方案操作比较复杂，引入了DNS服务，因此增加了单点故障及维护成本，当用户连接的vpn不能使用时，用户也需要重新人工再次拨号。

2）如果使用者为公司内部工作人员，此种方案是不推荐的。如果是外部的用户可以考虑用这种方式，但是复杂度比方案1大了很多（如果存在DNS服务器加配置还可以）。

3）当机房多，配置文件多时，无需用户选择服务器，只需拨号即可。如果多个VPN在一个机房还好一些，如果多个VPN服务器不在一个机房，还需要通过IPSEC进行连接。
总之，此法很麻烦，中小型公司老男孩老师极不推荐。

4）DNS轮询会遭遇到客户端DNS缓存问题，从而导致服务切换失效。。

[one-network-interface-on-a-public-network](https://openvpn.net/access-server-manual/typical-network-configurations/#one-network-interface-on-a-public-network)


