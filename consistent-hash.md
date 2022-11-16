# consistent hash


[toc]

## Memcached应用场景

### 基本场景

比如有 N 台 cache 服务器（后面简称 cache），那么如何将一个对象 object 映射到 N 个 cache 上呢，你很可能会采用类似下面的通用方法计算 object 的 hash 值，然后均匀的映射到到N个cache; <font color="#f8070d" size=3>`hash(object)%N`</font>

如下图：

![image-20221024235352996](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235352996.png)

> **这时，一切都运行正常，再考虑如下的两种情况**：

一个 cache服务器m down掉了（在实际应用中必须要考虑这种情况），这样所有映射到cache m的对象都会失效，怎么办，需要把cache m从cache 中移除，这时候 cache 是 $N-1$ 台，映射公式变成了 <font color="#f8070d" size=3>`hash(object)%(N-1)`</font> 。此时数据 $3\%3-1=3\%2=1$ 此时，3应该在S3上，但是由于S3down机导致到S1去取，这时会未命中。如下图

![image-20221024235406831](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235406831.png)

由于访问加重，需要添加 cache ，这时候 cache 是 $N+1$ 台，映射公式变成了 <font color="#f8070d" size=3>`hash(object)%(N+1)`</font> 。1和2意味着突然之间几乎所有的 cache 都失效了。对于服务器而言，这是一场灾难，洪水般的访问都会直接冲向后台服务器。$\frac{N-1} { N\times (N-1)}$

**即**：


有N台服务器，变为 $N-1$ 台，即每 $N \times (N-1)$个数中，求余相同的只有 <font color="#f8070d" size=3>`N-1`</font> 个。命中率为：$\frac{1}{3}$

再来考虑第三个问题，由于硬件能力越来越强，你可能想让后面添加的节点多做点活，显然上面的 hash 算法也做不到。

有什么方法可以改变这个状况呢，这就是 <font color="#f8070d" size=3>consistent hashing</font>...

但现在一致性hash算法在分布式系统中也得到了广泛应用，研究过memcached缓存数据库的人都知道，memcached服务器端本身不提供分布式cache的一致性，而是由客户端来提供，具体在计算一致性hash时采用如下步骤：

1. 首先求出memcached服务器（节点）的哈希值，并将其配置到 0～2<sup>32 </sup>的圆（continuum）上。

2. 然后采用同样的方法求出存储数据的键的哈希值，并映射到相同的圆上。

3. 然后从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器上。如果超过2<sup>32</sup>仍然找不到服务器，就会保存到第一台memcached服务器上。

![image-20221024235424248](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235424248.png)

从上图的状态中添加一台memcached服务器。余数分布式算法由于保存键的服务器会发生巨大变化而影响缓存的命中率，但Consistent Hashing中，只有在圆（continuum）上增加服务器的地点逆时针方向的第一台服务器上的键会受到影响，如下图所示：

![image-20221024235424248](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235424248.png)

接下来使用如下算法定位数据访问到相应服务器：将数据key使用相同的函数Hash计算出哈希值，并确定此数据在环上的位置，从此位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器。

## consistent hash原理

### 基本概念

一致性哈希算法（Consistent Hashing）最早在论文《Consistent Hashing and Random Trees: Distributed Caching Protocols for Relieving Hot Spots on the World Wide Web》中被提出。简单来说，一致性哈希将整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数H的值空间为 <font color="#f8070d" size=3>0-2<sup>32</sup>-1</font>（即哈希值是一个32位无符号整形），整个哈希空间环如下：

![image-20221024235550927](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235550927.png)

整个空间按顺时针方向组织。0和2<sup>32</sup>-1在零点中方向重合。

下一步将各个服务器使用Hash进行一个哈希，具体可以选择服务器的ip或主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置，这里假设将上文中四台服务器使用ip地址哈希后在环空间的位置如下：

![image-20221024235605237](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235605237.png)

接下来使用如下算法定位数据访问到相应服务器：将数据key使用相同的函数Hash计算出哈希值，并确定此数据在环上的位置，从此位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器。

例如我们有Object A、Object B、Object C、Object D四个数据对象，经过哈希计算后，在环空间上的位置如下：

![image-20221024235630275](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235630275.png)


根据一致性哈希算法，数据A会被定为到Node A上，B被定为到Node B上，C被定为到Node C上，D被定为到Node D上。

下面分析一致性哈希算法的容错性和可扩展性。现假设Node C不幸宕机，可以看到此时对象A、B、D不会受到影响，只有C对象被重定位到Node D。一般的，在一致性哈希算法中，如果一台服务器不可用，则受影响的数据仅仅是此服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响。如下图所示：

下面考虑另外一种情况，如果在系统中增加一台服务器Node X，如下图所示：

![image-20221024235642951](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235642951.png)

此时对象Object A、B、D不受影响，只有对象C需要重定位到新的Node X 。一般的，在一致性哈希算法中，如果增加一台服务器，则受影响的数据仅仅是新服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据，其它数据也不会受到影响。

综上所述，一致性哈希算法对于节点的增减都只需重定位环空间中的一小部分数据，具有较好的容错性和可扩展性。

另外，一致性哈希算法在服务节点太少时，容易因为节点分部不均匀而造成数据倾斜问题。例如系统中只有两台服务器，其环分布如下：

![image-20221024235702564](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235702564.png)

此时必然造成大量数据集中到Node A上，而只有极少量会定位到Node B上。为了解决这种数据倾斜问题，一致性哈希算法引入了<font style="background:#ffc104;" size=2>虚拟节点机制</font>，即对每一个服务节点计算多个哈希，每个计算结果位置都放置一个此服务节点，称为虚拟节点。具体做法可以在服务器ip或主机名的后面增加编号来实现。例如上面的情况，可以为每台服务器计算三个虚拟节点，于是可以分别计算 <font color="#f8070d" size=3>`Node A#1`</font>  <font color="#f8070d" size=3>`Node A#2`</font>  <font color="#f8070d" size=3>`Node A#3`</font>  <font color="#f8070d" size=3>`Node B#1`</font>  <font color="#f8070d" size=3>`Node B#2`</font>  <font color="#f8070d" size=3>`Node B#3`</font> 的哈希值，于是形成六个虚拟节点：

![image-20221024235718095](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235718095.png)

同时数据定位算法不变，只是多了一步虚拟节点到实际节点的映射，例如定位到<font color="#f8070d" size=3>`Node A#1`</font>  <font color="#f8070d" size=3>`Node A#2`</font>  <font color="#f8070d" size=3>`Node A#3`</font> 三个虚拟节点的数据均定位到Node A上。这样就解决了服务节点少时数据倾斜的问题。在实际应用中，通常将虚拟节点数设置为**==32==**甚至更大，因此即使很少的服务节点也能做到相对均匀的数据分布。

参考: http://www.cnblogs.com/haippy/archive/2011/12/10/2282943.html

## 一致性hash（consistent hash）在PHP中使用

```php
<?php 
	class ConsistentHash
	{
		public $nodes = array();
		
		public function __construct(){
			
		}
  
		public function generateHash($str){
			return sprintf('%u',crc32($str));
		}
		
		public function findNode(){
			
		}
  
		public function lookup($key){
			$tmp = $this->generateHash($key);
			$node = current($this->nodes);
			foreach($this->nodes as $key=>$val){
				if( $tmp <= $key ){
					$node = $val;
					break;
				}
			}
			return $node;
		}
		
		public function getNode(){
			var_dump($this->nodes);
		}
		
		public function addNode($node){
			$this->nodes[$this->generateHash($node)] = $node;
			ksort($this->nodes);
		}	
	}
	
	$hash = new ConsistentHash;
	$hash->addNode('192.168.2.80:11211');
	$hash->addNode('192.168.2.80:11212');
	$hash->addNode('192.168.2.80:11213');
	echo '<hr><br>';
	$hash->getNode();
	echo '<br>';
	echo $hash->generateHash('zhangsan'),'<br>';
	echo $hash->lookup('zhangsan'),'<br>';
?>
```

执行结果

![image-20221024235759078](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235759078.png)



![image-20221024235812368](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235812368.png)



这时可以看出，数据倾斜问题。

创建虚拟节点，解决数据倾斜问题

```php
<?php 
	class ConsistentHash{
		public $nodes = array();
		protected $num = 0;
		protected $priNode = array();
		
		public function __construct($nodeNum){
			$this->num = $nodeNum;
		}
		
    public function generateHash($str){
			return sprintf('%u',crc32($str));
		}
		
		public function selectNode($key){
			$tmp = $this->generateHash($key);
			$node = current($this->nodes);  # 选择最小的节点作为默认值
			foreach($this->priNode as $key=>$val){
				if( $tmp <= $key ){
					$node = $val;
					break;
				}
			}
			return $node;
		}
		
		public function getNode(){
			var_dump($this->nodes);
			var_dump($this->priNode);
		}
		
		public function addNode($node){	
			for($n=0;$n<$this->num;$n++){
				$this->priNode[$this->generateHash($node.'_'.$n)] = $node;
			}
			$this->nodes[$this->generateHash($node)] = $node;
			ksort($this->priNode);
		}	
	}
	
	$hash = new ConsistentHash(32);
	$hash->addNode('192.168.2.80:11211');
	$hash->addNode('192.168.2.80:11212');
	$hash->addNode('192.168.2.80:11213');
	$hash->getNode();
	echo '<br>';
	echo $hash->generateHash('zhangsan'),'<br>';
	echo $hash->generateHash('lisi'),'<br>';
	echo $hash->lookup('zhangsan'),'<br>';
	echo $hash->lookup('lisi'),'<br>';
	echo $hash->generateHash('wangwu'),'<br>';
	echo $hash->lookup('wangwu'),'<br>';
?>
```

此时自动分配的节点为

![image-20221024235829164](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235829164.png)

可看出6E-8E存在11211上，大于。35.7E-35.9E存在11212上

![image-20221024235847363](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235847363.png)



![image-20221024235855512](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235855512.png)





## 一致性hash与取模命中率的对比实验

 [dring.rar](../../../images\dring.rar) 

### 实验目的

测试Memcached缓存服务器有N台变为N-台时，取模和consistent hasing算法的命中率

### 实验原理

相同的硬件环境、操作系统、数据缓存环境，5个memcached节点，用两种分布式算法建立缓存，缓存命中率稳定后，减少1个节点，观察命中率的变化，知道命中率在次稳定。

### 前端软件架构

```sh
config.php 		#←配置memcached节点信息
hash.php 	 	#←分布式算法
init.php 		#←初始化数据
exec.php 		#←减少节点后请求数据
stat.php		#←统计平均命中率
index.html 		#←生成动态图表
```

### 取模算法的实验

当5台缓存服务器全部正常的情况下，此时的命中率统计图如下：

![image-20221024235917083](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235917083.png)

> **这是查看5台缓存服务器的查询与命中次数如下**：

```sh
$ for n in {1..5};do printf "stats\r\n"|nc 127.0.0.1 1121$n|egrep 'get_hits|cmd_get'; done
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
$ for n in {1..5};do printf "stats\r\n"|nc 127.0.0.1 1121$n|grep item; done
STAT curr_items 2044
STAT total_items 2044
STAT curr_items 1983
STAT total_items 1983
STAT curr_items 1993
STAT total_items 1993
STAT curr_items 2001
STAT total_items 2001
STAT curr_items 1979
STAT total_items 1979
```

这时断掉一台缓存服务器，此时的命中率从100%瞬间降至8%。

![image-20221024235932558](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235932558.png)

运行一段时间后，可见命中率保持20%左右，在预热完毕后，逐步上升。

![image-20221024235950143](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221024235950143.png)

此时查看5台缓存服务器的查询次数与命中次数，发现已经很均匀了。

```sh
$ for n in {1..5};do printf "stats\r\n"|nc 127.0.0.1 1121$n|egrep 'get_hits|cmd_get'; done
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 8481
STAT get_hits 6465
STAT cmd_get 8482
STAT get_hits 6476
STAT cmd_get 8482
STAT get_hits 6504
STAT cmd_get 8483
STAT get_hits 6478
```

经过较长时间后，可以看到命中率已经很平稳了

![image-20221025000013472](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025000013472.png)

### 一致性hash算法命中率实验

模拟出正常情况下，5台缓存服务器的命中率

```sh
$ for n in {1..5};do printf "stats\r\n"|nc 127.0.0.1 1121$n|egrep 'get_hits|cmd_get'; done
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0
STAT cmd_get 0
STAT get_hits 0

$ for n in {1..5};do printf "stats\r\n"|nc 127.0.0.1 1121$n|grep item; done
STAT curr_items 999
STAT total_items 999
STAT curr_items 1005
STAT total_items 1005
STAT curr_items 6005
STAT total_items 6005
STAT curr_items 998
STAT total_items 998
STAT curr_items 993
STAT total_items 993
```

此时断开1台服务器，可以见到命中率下降到73%就稳定了。

![image-20221025000031883](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025000031883.png)

观察一段时间后命中率逐步上升到95%

![image-20221025000050675](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025000050675.png)

## 在实战中会存在的问题

缓存雪崩的现象

一般是由于某个节点生效，导致其他节点的缓存命中率下降，缓存中缺失的数据去数据可查询。短时间内造成数据库服务器崩溃。或，由于缓存周期性的输小，如：6小时失效一次，那么每6小时，将有一个请求““峰值”，严重情况下会导致数据库宕机。

建议解决方案：

- 将缓存的生命周期设置为随机的时间短（如4-10）小时，这样缓存不同时失效，把工作分担到各个时间点上。
- 可在夜间缓慢建立一部分缓存
- 可建立多个缓存交叉使用，做好镜像，将多个缓存失效时间错开。

 


