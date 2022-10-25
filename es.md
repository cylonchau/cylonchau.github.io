# 安装elasticsearch

[TOC]

## 1 elasticsearch介绍
ES是一个基于Lucene实现的开源、分布式、基于Restful风格的全文本搜索引擎；此外，它还是一个分布式实时文档存储，其中每个文档的每个field均是被索引的数据，且可被搜索：也是一个带实时分析功能的分布式搜索引擎，能够扩展至数以百计的节点实时处理PB级的数据。

### 1.1 elasticsearch基本组件

索引（index）：文档容器，换句话说，索引是具有类似属性的文档的集合。类似于表。索引名必须使用小写字母：
类型（type）：类型是索引内部的逻辑分区，其意义完全取决于用户需求。一个索引内部可定义一个或多个类型。一般来说，类型就是拥有相同的域的文档的预定义。

文档（document）：文档是Lucene索引和搜索的原子单位，它包含了一个或多个域。是域的容器：基于J50N格式表示。每个域的组成部分：一个名字，一个或多个值；拥有多个值的域，通常称为多值域：

映射（mapping）：原始内容存储为文档之前需要事先进行分析，例如切词、过滤掉某些词等：映射用于定义此分析机制该如何实现；除此之外，ES还为映射提供了诸如将域中的内容排序等功能。

### 1.2 es的集群组件
cluster: es的集群标识为集群名称：默认“elasticsearch”。节点就是靠此名字来决定加入到哪个集群中。一个节点只能属性于一个集群。

Node：运行了单个es实例的主机即为节点。用于存储数据、参与集群索引及搜索操作。节点的标识靠节点名。

Shard：将索引切割成为的物理存储组件：但每一个shard都是一个独立且完整的索引；创建索引时，ES默认将其分割为5个shard，用户也可以按需自定义，创建完成之后不可修改。

shard有两种类型：primary shard和replica。replica用于数据冗余及查询时的负载均衡。每个主shard的副本数量可自定义，且可动态修改。


### 1.3 es的工作过程
es节点启动时默认通过多播方式或单播方式在TCP协议的9300端口查找同一集群中的其他节点，并与之建立通讯。判断是否为同一集群的标准为集群名称，集群中的所有节点会选举出一个主节点负责管理整个集群状态，以及在集群范围内决定各shared的分布方式。站在用户使用中角度而言，每个节点都可以接收并相应各类请求，无需区分哪个为主节点。

当集群中某一节点为down状态时，主节点会读取集群状态信息，并启动修复过程。此过程中主节点会检查所有可用shared的主shared，并确定主shared是否存在。各种shared及对应副本shared是否对数。此时集群状态会转换为yellow。

es集群有三种状态：green、red（不可用）、yellow（修复状态）。

当某状态为down的节点上存在主shared，此时需要从各副本shared中找一个提升为主shared。在yellow状态时，各副本shared均处于未分配模式。此时各个副本都不可用，只能使用主shared。集群虽可基于各组shared进行查询，但此时吞吐能力有限。yellows属于残破不全状态。

接下来的过程中，主节点将会查找所有的冗余shard，并将其配置为主shared。如果某shared的副本数量少于配置参数的数量，会自动启动复制过程，并为其建立副本shared。直至所有条件都满足从yellow转换至green。

es工作过程中主节点会周期性检查各节点是否处于可用状态，任意节点不可用时，修复模式都会启动，此时集群将会进入重新均衡过程。

## 2 安装elasticsearch

下载地址：http://www.elastic.co/cn/downloads/elasticsearch

![image](../../images/ES/203)



![image](../../images/ES/206)



官方文档中写明对es每个版本对jdk的要求。

https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html




### 2.1 常用es配置文件说明

修改配置文件：<font color="#f8070d" size=3>`/etc/elasticsearch/elasticsearch.yml`</font>

```sh
cluster.name: test1 #←集群名称，同一网段下可以有多个集群，通过名称区分不同的集群。

node.name: node-1  #←节点名

node.master: true #←是否有资格被选举成为node
node.data: true

# es5中已废弃
index.number_of_shards: 5  #←默认索引分片个数，默认为5片。
index.number_of_replicas: 1	 #←默认索引副本个数，默认为1个副本

path.conf: /path/to/conf   #←文件的存储路径
path.data: /path/to/data   #←索引数据的存储路径，多个存储路径，用逗号隔开
path.work: /path/to/work   #←日志文件的存储路径
path.plugins: /path/to/plugins   #←插件的存放路径
network.bind_host: 192.168.0.1   #←绑定的ip地址 
network.publish_host: 192.168.0.1  #←设置其它节点和该节点交互的ip地址
network.host: 192.168.0.1  #←同时设置bind_host和publish_host
transport.tcp.port: 9300  #←节点之间交互的tcp端口，默认是9300。
http.port: 9200   #←设置对外服务的http端口，默认为9200。
transport.tcp.compress: true   #←设置是否压缩tcp传输时的数据，默认false不压缩
http.max_content_length: 100mb  #←内容的最大容量，默认100mb
http.enabled: false   #←使用http协议对外提供服务，默认为true开启。

#←默认为local即为本地文件系统，可以设置为本地文件系统，
# 分布式文件系统，hadoop的HDFS，和amazon的s3服务器等。
gateway.type: local 

gateway.expected_nodes: 2   #←集群中节点的数量，默认为2。
discovery.zen.ping.timeout: 3s #←自动发现其它节点时ping连接超时时间，默认为3秒	
discovery.zen.ping.multicast.enabled: false  #←打开多播发现节点，默认是true。

# 设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。
discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]

# 当你无法关闭系统的swap的时候，建议把这个参数设为true。
# 防止在内存不够用的时候，elasticsearch的内存被交换至交换区，导致性能骤降
bootstrap.mlockal1: true
```

参考：https://www.cnblogs.com/hanyouchun/p/5163183.html
修改启动脚本设置启动参数： <font color="#f8070d" size=3>`/usr/share/elasticsearch/bin/elasticsearch`</font>
```sh
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$PATH
```
es2.x支持方式
```sh
export ES_HEAP_SIZE=256m
```
es5.x支持方式
```sh
[root@test ~]# cat /etc/elasticsearch/jvm.options 
-Xms128m
-Xmx128m
```

### 2.2 安装head插件
官方网站：https://github.com/mobz/elasticsearch-head#running-with-built-in-server

对于elasticsearch2.x
```sh
elasticsearch/bin/plugin install file:///root/elasticsearch-head.zip
```
对于Elasticsearch 5.x，head不支持网站插件。作为独立服务器运行。
https://www.cnblogs.com/alice626/p/6206722.html

https://www.cnblogs.com/xing901022/p/6030296.html

https://www.cnblogs.com/valor-xh/p/6293689.html

https://github.com/mobz/elasticsearch-head，github上下载压缩包。

因为head为独立启动的，所以随便放置任意目录即可。

#### 2.2.1 安装node.js

https://nodejs.org/en/download/

解压后设置环境变量
```sh
export NODE_HOME=/usr/local/node-v6.9.1-linux-x64 
export PATH=$PATH:$NODE_HOME/bin
```
检查环境变量
```sh
[root@redis ~]# node -v
v8.11.2
[root@redis ~]# npm -v
5.6.0
```
#### 2.2.2 安装grunt
```sh
npm install grunt-cli
[root@redis ~]# grunt -version
grunt-cli v1.2.0
```

#### 2.2.3 修改head配置
head/Gruntfile.js，增加hostname属性，设置为*
```js
connect: { 
 	server: { 
 		options: { 
 			port: 9100, 
  			hostname: '*', 
 			base: '.', 
 			keepalive: true 
 		} 
 	} 
}
```

修改连接地址
head/_site/app.js
```js
this.base_uri = 
this.config.base_uri || 
this.prefs.get("app-base_uri") || 
"http://10.0.0.12:19200";
```

#### 2.2.4 运行head
在head目录下，执行npm install 下载以来的包：
```sh
nmp install
```

出现如下报错
```sh
Please report this full log at https://github.com/Medium/phantomjs
npm ERR! Darwin 15.0.0
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "install"
npm ERR! node v4.4.3
npm ERR! npm  v3.10.9
npm ERR! code ELIFECYCLE
 
npm ERR! phantomjs-prebuilt@2.1.14 install: `node install.js`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the phantomjs-prebuilt@2.1.14 install script 'node install.js
```
解决方法
```sh
npm install phantomjs-prebuilt@2.1.14 --ignore-scripts
```
运行head
```sh
grunt server &
```
设置http对外提供服务



### 2.3 安装bigdesk
https://github.com/hlstudio/bigdesk

下载bigdesk

```sh
[root@test ~]# cd bigdesk-master
[root@test bigdesk-master]# ll

bigdesk_es2.png
LICENSE
NOTICE
plugin-descriptor.properties
README.md
【_site】
```
进入_site目录

```sh
python -m SimpleHTTPServer port 8000
```

![image](../../images/ES/205)


marven、kopf

2.x plugin 访问 <font color="#f8070d" size=3>`9200/_plugin/head`</font>

## 3 ES使用

### 3.1 es api介绍
elasticsearch支持的是Restful风格的API接口

restuful介绍：
http://www.ruanyifeng.com/blog/2014/05/restful_api.html?bsh_bid=516759003

四类API：
* 检查集群、节点、索引等健康与否，以及获取其相应状态。
* 管理集群、节点、索引及元数据。
* 执行CRUD操作。
* 执行高级操作，例如paging,filtering等。

### 3.2 es api使用
E5访问接口：tcp/9200
```sh
curl -X [VERB] [PROTOCOL]://HOST:PORT/[PATH]?[QUERY_STR] -d '<BODY>'
```
* VERB：GET,PUT,DELETE等。
* PROTOCOL：http,https。
* QUERY_STRING：查询参数，例如？pretty表示用易读的JSON格式输出。
* BODY：请求的主体。

#### 3.2.1 查看es工作状态

```sh
[root@redis ~]# curl http://10.0.0.12:19200/?pretty
{
  "name" : "node-1",
  "cluster_name" : "test1",
  "version" : {
    "number" : "2.0.0",
    "build_hash" : "de54438d6af8f9340d50c5c786151783ce7d6be5",
    "build_timestamp" : "2015-10-22T08:09:48Z",
    "build_snapshot" : false,
    "lucene_version" : "5.2.1"
  },
  "tagline" : "You Know, for Search"
}
```
查看集群工作状态，?v查看详细信息。

```sh
[root@test ~]# curl 127.0.0.1:19200/_cat/nodes?v
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.0.0.9            27          90   1    0.07    0.03     0.05 mdi       -      node-1
10.0.0.12           46          91   0    0.00    0.01     0.05 mdi       *      node-2
```
health：查看当前集群健康状态的相关信息
```sh
[root@test ~]# curl 127.0.0.1:19200/_cat/health?pretty
1526839934 02:12:14 test green 2 2 0 0 0 0 0 0 - 100.0%

[root@test ~]# curl 127.0.0.1:19200/_cat/health?v
epoch      timestamp cluster status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1526841115 02:31:55  test    green           2         2      0   0    0    0        0             0                  -                100.0%
```


#### 3.2.2 自定义查询
```sh
[root@test ~]# curl 127.0.0.1:19200/_cat/nodes?h=ip,name,port,uptime,heap.current
10.0.0.9  node-1 19300 40.7m 107.2mb mdi
10.0.0.12 node-2 19300   57m  89.6mb mdi
```

```sh
[root@test ~]# curl 127.0.0.1:19200/_cat/master?v 
id                     host      ip        node
CorsR-_VSN2YHj5DCRXiKg 10.0.0.12 10.0.0.12 node-2
```
官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/6.2/index.html

#### 3.2.3 state api
```json
curl 127.0.0.1:19200/_cluster/state/nodes?pretty

{
  "cluster_name" : "test",
  "compressed_size_in_bytes" : 276,
  "nodes" : {
    "cplyIE3WS3a4sKRhjYQPoQ" : {
      "name" : "node-1",
      "ephemeral_id" : "axJ2wLcZRxyMMfLP7J54HA",
      "transport_address" : "10.0.0.9:19300",
      "attributes" : { }
    },
    "CorsR-_VSN2YHj5DCRXiKg" : {
      "name" : "node-2",
      "ephemeral_id" : "UTGxkPRYSRO_wU_URX_taw",
      "transport_address" : "10.0.0.12:19300",
      "attributes" : { }
    }
  }
}
```

curl 127.0.0.1:19200/_cluster/state/nodes?pretty
curl 127.0.0.1:19200/_nodes/state?pretty
https://www.elastic.co/guide/en/elasticsearch/reference/6.2/cluster-state.html

### 3.3 crud api
#### 3.3.1 创建索引
```json
curl -X PUT -H "Content-Type: application/json" http://127.0.0.1:19200/students/NO/1?pretty -d '

{
 "first_name": "jing",
 "last_name" : "guo",
 "gender" : "male",
 "age" :"25",
 "courses": "xianglong shiba zhang"
}'
{
  "_index" : "students",
  "_type" : "NO",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

```json
'{
 "first_name": "rong",
 "last_name" : "huang",
 "gender" : "female",
 "age" :23,
 "courses": "luoying shenjian zhang"
}'
```

```sh
curl -XGET http://127.0.0.1:19200/students/NO/1?pretty
{
  "_index" : "students",
  "_type" : "NO",
  "_id" : "1",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "first_name" : "rong",
    "last_name" : "huang",
    "gender" : "female",
    "age" : 23,
    "courses" : "luoying shenjian zhang"
  }
}
```


#### 3.3.2 更新文档
```sh
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:19200/students/NO/1/_update?pretty -d 
'{
  "doc": { "age": 18 }
}'
{
  "_index" : "students",
  "_type" : "NO",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

#### 3.3.3 删除文档

```sh
curl  -X DELETE http://10.0.0.12:19200/students/NO/1?pretty
{
  "_index" : "students",
  "_type" : "NO",
  "_id" : "1",
  "_version" : 4,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 2
}
```

#### 3.3.4 删除索引

```sh
curl  http://10.0.0.12:19200/_cat/indices
green open test1    cBL6BqIqStyr2a8O2n8Bdw 5 1 0 0  2.5kb  1.2kb
green open students g04VDxTUTEOLmdooHuuRpA 5 1 1 0 26.5kb 13.2kb

curl  -X DELETE http://10.0.0.12:19200/test1
{"acknowledged":true}

curl  http://10.0.0.12:19200/_cat/indices
green open students g04VDxTUTEOLmdooHuuRpA 5 1 1 0 26.5kb 13.2kb
```

### 3.4 查询数据

Query API Query DSL:JS0N based language for building complex queries。

用户实现诸多类型的查询操作，比如，simple term query,phrase,range boolean,fuzzy等：

ES的查询操作执行分为两个阶段：
* 分散阶段：
* 合并阶段：

列出一个索引的所有文档，较小级别时的演示说明

```json
curl  http://10.0.0.12:19200/_search?pretty

{
  "took" : 15,  #←执行时长
  "timed_out" : false,  #←是否超时
  "_shards" : {   #←操作设计到多少个分片
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {   #←查询结果
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "students",
        "_type" : "NO",
        "_id" : "2",
        "_score" : 1.0,  #←文档平分，没有加权
        "_source" : {
          "first_name" : "jing",
          "last_name" : "guo",
          "gender" : "male",
          "age" : "25",
          "courses" : "xianglong shiba zhang"
        }
      },
      {
        "_index" : "students",
        "_type" : "NO",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "rong",
          "last_name" : "huang",
          "gender" : "female",
          "age" : 23,
          "courses" : "luoying shenjian zhang"
        }
      }
    ]
  }
}
```

默认情况下，只返回前十个。

```json
curl -H 'content-type: application/json' http://10.0.0.12:19200/_search?pretty -d '{
    "query": {"match_all": {}}
}'
```

### 3.5 多索引、多类型查询
/_search：所有索引：
/INDEX_NAME/_search：单索引：
/INDEX1,INDEX2/_search：多索引：
/s*,t*/_search：
/students/class1/_search：单类型搜索
/students/class1,class2/_search：多类型搜索


### 3.6 插入测试数据
```sh
curl -H 'content-type: application/json' \
http://10.0.0.12:19200/test1/NO/1?pretty -d '{
   "name":"zhou yu",
   "age": "25",
   "weapon": "duan jian"
}'

curl -H 'content-type: application/json' \
http://10.0.0.12:19200/test1/NO/2?pretty -d '{
   "name":"zhangliang",
   "age": 31,
   "weapon": "fu zhou"
}'

curl -H 'content-type: application/json' \
http://10.0.0.12:19200/test1/NO/3?pretty -d '{
   "name":"zhang liao",
   "age": "31",
   "weapon": "dao"
}'

curl -H 'content-type: application/json' \
http://10.0.0.12:19200/test1/NO/4?pretty -d '{
   "name":"zhang fei",
   "age": "30",
   "weapon": "zhang ba she mao"
}'

curl -H 'content-type: application/json' \
http://10.0.0.12:19200/test1/NO/5?pretty -d '{
  "name": "zhu ge liang",
  "age": "18",
  "weapon": "yu shan",
  "intro": "31 years old"
}'
```

可以看出，默认按照字符串匹配。文档中的每个域存储为特定类型而非字符串。

```json
{ 
"hits": 
 	{ "total": 3, "max_score": 0.87546873, "hits": 
 	[ { 
 			"_index": "test1", 
 		"_type": "NO", 
 		"_id": "3", 
 		"_score": 0.87546873, 
 		"_source": 
 		  { 	
 					"name": "zhang liao", 
           		"age": "31", 
 				"weapon": "dao" 
 		  } 
 		}, 
 	  { 
 		"_index": "test1", 
 		"_type": "NO", 
 			"_id": "2", 
 		"_score": 0.87546873, 
 		"_source": 	
 			{ 	
 			  "name": "zhangliang", 
 				  "age": 31, 
 			  "weapon": "fu zhou"
 			} 
 		} ,
 		{ 
 			"_index": "test1", 
 			"_type": "NO", 
 			"_id": "5", 
 			"_score": 0.2876821, 
 			"_source": 
 			{ 	
 				"name": "zhu ge liang", 
 				"age": "18", 
 				"weapon": "yu shan", 
 				"intro": "31 years old" 
 			} 
 		} 
 	] 
} 
```

指定域搜索做精确匹配

```json
"hits": [ 
 	{ 
 			"_index": "test1", 
 			"_type": "NO", 
 			"_id": "3", 
 			"_score": 0.87546873, 
 			"_source": 
 			{ 
 				"name": "zhang liao", 
 				"age": "31", 
 				"weapon": "dao" 
 		 	} 
 		} , 
 		{ 
 		 	"_index": "test1", 
 		 	"_type": "NO", 
 		 	"_id": "4", 
 		 	"_score": 0.87546873,
 		 	"_source": 
 		 	{ 
 		 		"name": "zhang fei", 
 		 		"age": "30", 
 		 		"weapon": "zhang ba she mao" 
 		 	} 
 	} 
]
```


查看指定类型的mapping示例
curl "http://10.0.0.12:19200/test1/_mapping/NO?pretty"
mapping定义了整个文档中的数据是被当做何种类型对待的。

```json
{
	  "test1" : {
		"mappings" : {
		  "NO" : {
			"properties" : {
			  "age" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  },
			  "intro" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  },
			  "name" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  },
			  "weapon" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  }
			}
		  }
		}
	  }
	}
```
ES中搜索的数据广义上可被理解为两类：
* types:exact 
* full-text 

精确值：指未经加工的原始值；在搜索时进行精确匹配。NOTEBOOK≠notebook

full-text：用于引用文本中数据；判断文档在多大程序上匹配查询请求；即评估文档与用户请求查询的相关度。

为了完成ful1-text搜素，ES必须首先分析文本，并创建出倒排素引：倒排素引中的数据还需进行“正规化”为标准格式：如全部小写、负数变为单数、trees改为tree等。

创建倒排索引需要先分词再normalization。分词加正规化的操作及为分析。

分析需要由分析器(analyzer)进行。分析器由三个组件构成：字符过滤器、分词器、分词过滤器。

ES内置的分析器：
* Standard analyzer：ES的默认分析器，适用于多种语言，基于Unicode方式进行分析。
* Simple analyzer：简单分析器，根据所有的字母进行分析。
* Wirtespace analyzer：只把空白字符当做单词分割。
* Language analyzer：为多种语言分别提供各种各样的分析器。

分析器不仅在创建素引时用到；在构建查询时也会用到。构建索引时的分析器，分析用户查询并构建成查询语句的分词方式。两种方式需要保持一致。




[2018-09-21 05:24:03,381][INFO ][cluster.routing.allocation.decider] [node-1] low disk watermark [85%] exceeded on [JWxQxMkeTj6BkmIx-6DsXg][node-1][/var/lib/elasticsearch/my-application/nodes/0] free: 1gb[11.2%], replicas will not be assigned to this node

## 4 ES错误

### ES索引yellow状态

> **查看片状态**

```sh
curl 172.16.16.18:9200/_cat/shards
```

```sh
logstash-webapi-2018.10.30 0 p STARTED        1   6.1kb 127.0.0.1 6wPQIP5
logstash-webapi-2018.10.30 0 p STARTED        3  16.4kb 127.0.0.1 6wPQIP5
logstash-webapi-2018.10.30 0 r UNASSIGNED                         
logstash-webapi-2018.10.30 0 p STARTED     1824     1mb 127.0.0.1 6wPQIP5
logstash-webapi-2018.10.30 0 p UNASSIGNED
logstash-webapi-2018.10.30 0 p STARTED      272 363.4kb 127.0.0.1 6wPQIP5
logstash-webapi-2018.10.30 0 p UNASSIGNED
logstash-webapi-2018.10.30 0 p STARTED        0   5.1kb 127.0.0.1 6wPQIP5
logstash-webapi-2018.10.30 0 p UNASSIGNED
logstash-webapi-2018.10.30 0 p STARTED     4888   2.4mb 127.0.0.1 6wPQIP5
logstash-webapi-2018.10.30 0 p UNASSIGNED 
```

yellow表示所有主分片可用，但不是所有副本分片都可用，最常见的情景是单节点时，由于es默认是有1个副本，主分片和副本不能在同一个节点上，所以副本就是未分配unassigned

```sh
PUT 172.16.16.18:9200/logstash -d '
{
  "settings":{
           "number_of_shards":1,     
           "number_of_replicas":0
  }
}'
```
参考网址：[elasticsearch集群健康为黄色](https://blog.csdn.net/qq_21383435/article/details/79323739)
