# ipset性能测试


## 测试方法
基于使用场景，最后⽣成的规则会是按照 ip 或者 `ip:port` 来进行过滤，测试时将使用10万条 `iptables` 规则来模拟对性能的压力；为了最大化测试压力情况，10万条 `iptables` 规则将都是==不会匹配==机房流量，通俗来讲，就是链式匹配会进行所有匹配并最后以无匹配告终。

网络负载的模拟将使用同机房 `scp` 来模拟，并按照下述条件进行匹配：

* 查看正常的拷贝速度，cpu负载等
* 我们建⽴10万条的普通 `iptables` 规则，查看规则建立速度，拷贝速度，CPU负载，CPU主要耗时操作等
* 我们建⽴10万的 `ipset` ，并把普通的 `iptables` 规则转为结合 `ipset` 的规则，查看规则建立速度，拷贝速度，CPU负载，CPU主要耗时等。

## 实验开始

**步骤一**：在同机房的⼀个机器构造⼀个大文件

![image-20221104233138861](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104233138861.png)

同机房拷贝

![image-20221104233643372](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104233643372.png)

观察网卡速度，CPU，系统主要耗时操作的等，此场景将在`iptables` 规则为空的情况下进行观察

![image-20221104234017089](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104234017089.png)

![image-20221104234322789](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104234322789.png)

使用 `sar` 观测网卡速度

![image-20221104233834434](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104233834434.png)

使用 `top` 观察CPU负载

![image-20221104234452841](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104234452841.png)

使用 `perf top -G` 观察CPU占用

![image-20221104234724668](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104234724668.png)

**步骤二**：创建10万条iptables，观察⽹卡速度、cpu、系统主要耗时操作的等，会发现cpu利⽤率⼤部分被ipt占⽤，拷⻉速度下降到不到⼗分之⼀

```bash
#!/bin/bash

echo *filter
for ((i=1;i<=$1;i++))
do
	echo -I INPUT -S $i -j ACCEPT
done
echo COMMIT
```

执行脚本

```bash
$ time ./mkrule.sh 100000 | sudo iptables-restore
```

观察添加规则后的⽹卡速度，CPU，系统主要耗时操作的等

使用 `sar` 观测网卡速度

![image-20221104235306056](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104235306056.png)

使用 `top` 观察CPU负载

![image-20221104235441887](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104235441887.png)

使用 `perf top -G` 观察CPU占用

![image-20221104235624488](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221104235624488.png)

**步骤三**：使用ipset替换iptables

此时改为使⽤ `ipset` ⽅式观察网卡卡速度，CPU，系统主要耗时操作的等，会发现跟没有规则没有明显变化。ipset的内存量不到2M。初步估计内存使⽤量 = $hashsize \times 16 + 存⼊数 \times (4～32之间)$

```bash
#!/bin/bash

#ipset create whitelist hash:ip maxelem 1000000 -exist
#ipset flush whitelist
echo ' creae whitelist hash:ip family inet hashsize 65536 maxelem 100000000'

for ((i=1;i<=$1;i++))
do	
	#ipset add whitelist $i
	echo add whitelist $i
done
# iptables -I INPUT -m set --match-set whitelisti src -j ACCEPT
```

执行脚本

```bash
$ time ./mkset.sh 100000 | sudo ipset restore
$ sudo iptables -Ln
$ sudo iptables -I INPUT -m set --match-set whitelisti src -j ACCEPT
```

观察添加规则后的⽹卡速度，CPU，系统主要耗时操作的等

使用 `sar` 观测网卡速度

![image-20221105000318616](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221105000318616.png)

使用 `top` 观察CPU负载

![image-20221105000454342](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221105000454342.png)

使用 `perf top -G` 观察CPU占用

![image-20221105000630330](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221105000630330.png)

```bash
$ sudo ipset list | head
Name: whitelist
Type: hash:ip
Header: family inet hashsize 65536 maxelem 100000000
Size in memory: 1891208
References: 1
```

总结：ipset对于CPU和内存的影响很小，在大量规则场景下符合预期
