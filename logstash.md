# Logstash 使用

[TOC]

## 1 logstash概述

logstash是基于Jruby语言研发的server/agent结构的日志收集工具，可以在任何一个能产生日志的服务器上部署其agent。agent能同时监控多个产生日志的服务，并把每个服务所产生的日志信息收集并发送给logstash server端。由于收集到的日志信息是分散的，logstash有专门的节点用来将所有的节点收集到的日志信息按时间序列合并在一起形成一个序列，基于这一个序列请求写入并存储在elasticsearch集群中。

kibana是基于node.js开发的elasticsearch的图形用户接口。它可以将用户的搜索语句发送给ES，有ES完成搜索，并且将结果返回。kibana可以将返回的数据，用非常直观的方式（画图）来做趋势展示的。

logstash基于多种数据获取机制，TCP/UDP协议、文件、syslog、windows EventLogs及STDIN等。同时也支持多种数据输出机制。获取到数据后，它支持对数据执行过滤、修改等操作。

logstash基于JRuby语言研发，需要运行在JVM虚拟机之上。工作于agent/server模型。

### 1.1 logstash工作机制

在每一个产生日志的节点之上部署一个agent，这个agent通常称为&lt;font color=&#34;#f8070d&#34; size=3&gt;`shipper`&lt;/font&gt;。shipper负责收集日志，并且发送给server端。但是在发送给server端时，为了避免server端可能无法应付大量节点同时发来的信息，会在server端和agent端之间放置一个消息队列，通常称之为&lt;font color=&#34;#f8070d&#34; size=3&gt;`broker`&lt;/font&gt;。这个消息队列比较常见的使用redis。各agent将数据发送至broker，logstash服务器会从broker中依次取出日志拿来在本地做过滤、修改等操作。在完修改后将其发送至ES集群。

对于logstash而言，server端也是插件式的工作的模式，与ES的不同在于，ES的插件是可有可无的。logstash的除核心外的所有功能都是基于插件来完成的。

### 1.2 logstash工作流程
logstash的工作模式类似于管道，从指定位置读入数据，而后过滤数据，最后将其输出到指定位置。事实上logstash的工作流程为：&lt;font color=&#34;#f8070d&#34; size=3&gt;`input | filter | output`&lt;/font&gt;。如无需对数据额外处理，filter可省略。

### 1.3 logstash插件分类
* input plugin：收集数据
* codec plugin：编码插件
* filter plugin：过滤
* output plugin：输出插件

## 2 logstash安装下载

官网地址：https://www.elastic.co/cn/products../../images/Logstash

![image](../../images/Logstash/da565791.png)

## 3 logstash的配置使用
&lt;font color=&#34;#f8070d&#34; size=3&gt;`/etc../../images/Logstash/conf.d/`&lt;/font&gt;路径下所有以&lt;font color=&#34;#f8070d&#34; size=3&gt;`.conf`&lt;/font&gt;结尾的文件都被当做配置文件使用。对于logstash的配置文件定义，就是定义插件。从哪得到数据。向哪里输出数据。

### 3.1 logstash的基本配置框架

```sh
input{
 	....
}

filter{
 	....
}

output{
 	....
}
```

简单的

```sh
input{
 	stdin {}
}
output { 
 	stdout{
 		codec =&gt;rubydebug
 	}
}
```
检查配置文件语法是否正确&lt;font color=&#34;#f8070d&#34; size=2&gt;`logstash -f /etc../../images/Logstash/conf.d/sample.conf --configtest`&lt;/font&gt;

```sh
[root@test conf.d]# logstash -f /etc../../images/Logstash/conf.d/sample.conf 
hellow owrd
Settings: Default pipeline workers: 1
Pipeline main started
{
       &#34;message&#34; =&gt; &#34;hellow owrd&#34;, #←信息的完整内容。logstash的信息是在input → filter → output之间流动的。
      &#34;@version&#34; =&gt; &#34;1&#34;, #←版本号。
    &#34;@timestamp&#34; =&gt; &#34;2018-06-04T15:02:47.912Z&#34;, #←事件发生时的时间戳。
          &#34;host&#34; =&gt; &#34;test&#34; #←时间发生的主机。
}
```

### 3.2 logstash支持的数据类型

Array：[iteml,item2,...]
Boolean：true,false
Bytes
Codec：编码器。
Hash: key =&gt;value 
Number
Password：
Path：文件系统路径。
String：字符串。
logstash还支持字段引用，使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`[]`&lt;/font&gt;进行引用

3.3 支持的条件判断
==，!=，&lt;，&lt;=，&gt;，&gt;= 正则匹配=~ !~
in,not in and,or


## 4 Logstash常用的插件

logstash是高度插件化的，主要分为&lt;font color=&#34;#f8070d&#34; size=3&gt;`input`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`codec`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`filter`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`output`&lt;/font&gt;插件：


### 4.1 input插件

#### 4.1.1 File

从指定的文件中读取事件流。其工作特性类似于&lt;font color=&#34;#f8070d&#34; size=3&gt;`tail -1f`&lt;/font&gt;，能够将日志文件尾部的一行不断的读取出来。

logstash使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`FileWatch`&lt;/font&gt;（ruby gem库）机制来监听文件变化。filewatch是linux内核中提供的一种功能。filewatch库支持以glob方式展开文件名。因此可以监听多个文件。并且可以将每个文件读取的位置及状态信息保存在&lt;font color=&#34;#f8070d&#34; size=3&gt;`.sincedb`&lt;/font&gt;数据库中。即使重启logstash后也不会从头读取文件的。.sincedb记录了每个被监听的文件的&lt;font color=&#34;#f8070d&#34; size=3&gt;`inode`&lt;/font&gt;,&lt;font color=&#34;#f8070d&#34; size=3&gt;`major number`&lt;/font&gt;,&lt;font color=&#34;#f8070d&#34; size=3&gt;`minor nubmer`&lt;/font&gt;,&lt;font color=&#34;#f8070d&#34; size=3&gt;`pos`&lt;/font&gt;。默认情况下位于启动logstash服务的用户家目录下。也可以自行定义位置。另外，file插件还可以自动识别日志的滚动（日志分割）操作。

```sh
input{
    file{
        path =&gt; [&#39;/var/log/messages&#39;]
        type =&gt; &#34;system&#34;
        start_postition =&gt; &#34;beginning&#34;
    }
}

output{
    stdout{
        codec =&gt; rubydebug
    }
}
```

[Logstash](https://www.elastic.co/guide/en../../images/Logstash/2.4/index.html)
[file](https://www.elastic.co/guide/en../../images/Logstash/2.4/plugins-inputs-file.html)


#### 4.1.2 UDP

logstash支持通过一个简单的协议来读取信息，如UDP、TCP，说明对方只要通过TCP协议发送时间也是支持的。或者哪怕通过syslog收集事件也是可以的。

logstash通过UDP协议通过网络连接来读取信息，其唯一必须制定的参数为&lt;font color=&#34;#f8070d&#34; size=3&gt;`port`&lt;/font&gt;，用于指定自己监听的端口，agent可以向服务器端端口发送日志信息。而每个被收集数据的主机使用守护进程向server端发送事件。

```sh
input{
    udp{
        prot =&gt; 25826
        queue_size =&gt; 2000  #←队列大小
        workers =&gt; 2        #←启动的工作线程数来接收发送的事件。
        buffer_size =&gt;      #←接收缓冲区的大小
    }
}

```

&gt; **UDP插件实现**


collectd：基于C语言开发的一款高度插件式的主机性能监控程序，以守护进程方式运行，能够收集系统性能相关的各种数据；并且能够基于各种存储机制将收集的结果存储下来。collectd本身所收集到的数据可以通过自身的network插件，将自己在本机所收集到的数据发送至其他主机。

collectd在epel源中

```sh
yum install collectd -y
```
collectd的大量.so文件为其模块（插件），每一个模块分别实现每一种相应的功能。其配置文件 &lt;font color=&#34;#f8070d&#34; size=3&gt;`/etc/collectd.conf`&lt;/font&gt;


```sh
Hostname &#34;node3.magedu.com&#34;
LoadPlugin syslog 
LoadPlugin cpu  # cpu性能数据
LoadPlugin df  # 磁盘空间使用情况
LoadPlugin interface # 网络接口
LoadPlugin 1oad  # 服务器负载
LoadPlugin memory  # 内存
LoadPlugin network 
&lt;Plugin network&gt;
    &lt;server &#34;10.0.0.19&#34; &#34;25826&#34;&gt; #10.0.0.19是logstash主机的地址，25826是其监听的udp端口
    &lt;/server&gt;
&lt;/Plugin
```

```sh
input udp {
    port =&gt; &#34;25826&#34;
    codec =&gt; collectd {}
    type =&gt; &#34;collectd&#34;
}

output{
    stdout{
        codec =&gt; rubydebug
    }
}

```

#### 4.1.3 redis

redis插件主要作用在于从redis中获取数据。它支持redis中的两种方式；支持redis channel（发布频道）以及lists两种方式。logstash从redis读数据，可以基于列表方式进行。也可以基于channel方式进行。也就是说redis可以仅工作在正常模式下，不用启用channel机制，也一样能够使logstash从中获取数据。

对于logstash而言，一般要求redis版本在&lt;font color=&#34;#f8070d&#34; size=3&gt;`2.6.0`&lt;/font&gt;之后的版本。

### 4.2 filter插件

filter插件主要用于将event通过output发出之前对其实现某些处理功能。对所收集到的数据当中，期望能够基于某种特定格式进行拆分、组合、过滤其中某些信息，都可以使用filter插件实现。在过滤前后如果有需要，都可以调用codec插件，事先基于某种形式对消息做编码。将处理过数据存往任何可存储位置，需要用到特定于存储位置的输出插件。logstash有众多filter过滤器。


#### 4.2.1 grok

gork是logstash中最重要的插件之一，主要用于分析并结构化文本数据。从web服务器日志读取的日志事件是遵循同种格式的，如：我们只想统计有多少访问IP，但是获取的是一行信息。为了能够方便随后的统计操作，需要将每读到的任何一行日志实现切好。使得对日志分析能够得以进行。目前是logstash中将非结构化日志数据转化为结构化的可查询数据的不二之选。

目前支持处理 &lt;font color=&#34;#f8070d&#34; size=3&gt;`syslog`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`httpd`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`nginx`&lt;/font&gt; 等。默认情况下，logstash提供了120种默认grok模式。这些模式定义在&lt;font color=&#34;#f8070d&#34; size=3&gt;`patterns`&lt;/font&gt;目录下，每个模式都有默认的名称。

```sh
rpm -ql logstash|grep &#34;patterns$&#34;;
```

#### 4.2.2 gork语法格式

logstash中grok的语法格式是使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`%{}`&lt;/font&gt;括起的一组信息使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`:`&lt;/font&gt;隔开，其中左半段称之为&lt;font color=&#34;#f8070d&#34; size=3&gt;`SYNTAX`&lt;/font&gt;由半段称之为&lt;font color=&#34;#f8070d&#34; size=3&gt;`SEMANTIC`&lt;/font&gt;。其中SYNTAX表示grok模板预定义模式中已有的模式名称，SEMANTIC，匹配到的文本的自定义的标识符。
```sh
%{SYNTAX:SEMANTIC}
```

例:

```sh
10.0.0.1 GET /index.html 30 0.23

%{IP:client_ip} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}
```

10.0.0.1 GET /index.html 30 0.23
{
       &#34;message&#34; =&gt; &#34;10.0.0.1 GET /index.html 30 0.23&#34;,
      &#34;@version&#34; =&gt; &#34;1&#34;,
    &#34;@timestamp&#34; =&gt; &#34;2018-05-31T10:34:37.150Z&#34;,
          &#34;host&#34; =&gt; &#34;mysql&#34;,
     &#34;client_ip&#34; =&gt; &#34;10.0.0.1&#34;,
        &#34;method&#34; =&gt; &#34;GET&#34;,
       &#34;request&#34; =&gt; &#34;/index.html&#34;,
         &#34;bytes&#34; =&gt; &#34;30&#34;,
      &#34;duration&#34; =&gt; &#34;0.23&#34;
}


```sh
input {
		stdin {}
}

filter {
  grok {
    match =&gt; { 
      &#34;message&#34; =&gt; &#34;%{IP:client_ip} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}&#34; 
    } 
  }
}

output {
		stdout { codec =&gt; rubydebug }
}
```

自定义grok模式

grok模式是基于正则表达式的模式编写，其元字符与其他用到正则表达式的工具awk\sed\grep\pcre差别不大


nginx log匹配方式

```sh
NGUSERNAME [a-zA-Z\.\@\-\&#43;_%]&#43;
NGUSER %{NGUSERNAME}
NGINXACCESS %{IPORHOST:clientip} - %{NOTSPACE:remote_user} \[%{HTTPDATE:timestamp}\] \&#34;(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})\&#34; %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent} %{NOTSPACE:http_x_forwarded_for}
```
[LOGSTASH&#43;ELASTICSEARCH&#43;KIBANA处理NGINX访问日志 - CSDN博客](https://blog.csdn.net/joeyon1985/article/details/46639511)


```sh
input {
		file {
				path =&gt; [&#39;/var/log/nginx/access.log&#39;]
				type =&gt; &#34;nginxlog&#34;
				start_position =&gt; &#34;beginning&#34;
		}
}

filter {
		grok {
				match =&gt; { &#34;message&#34; =&gt; &#34;%{NGINXACCESS}&#34; }
		}
}

output {
		stdout	{
				codec =&gt; rubydebug
		}
}

```

action 将数据输出到至elasticsearch中后基于什么方式做操作
hosts es主机集群列表。数组
index 存储在es的哪个索引当中



logstash在使用redis做输入或输出插件时，有两种数据类型，用来保存logstash输出的数据list channel


[ES5.4.0 记录 - CSDN博客](https://blog.csdn.net/aaronhadoop/article/details/73163025)

[18-elasticsearch集群健康为黄色 - CSDN博客](https://blog.csdn.net/qq_21383435/article/details/79323739)



curl -X PUT 172.16.16.18:9200/gongsufanghua-work_record/_settings -d &#39;{
	&#34;index&#34; : {
		&#34;number_of_replicas&#34; : 0
	}
}&#39;


[logstash-input-jdbc同步mysql数据到elasticsearch - 运维学习之路 - SegmentFault 思否](https://segmentfault.com/a/1190000011784259)

[Elasticsearch yellow unassigned_shards 恢复 replicas 节点恢复 - CSDN博客](https://blog.csdn.net/u012546526/article/details/78598608)


[logstash mysql 准实时同步到 elasticsearch - 简书](https://www.jianshu.com/p/62433b9c5c96)


## 5 conditionals

当需要在特定条件下过滤或输出事件。可以使用条件条件判断语句。Logstash中的条件查看和行为与编程语言中的条件相同。条件语句支持if，else if以及else和可以被嵌套。

[conditionals](https://www.elastic.co/guide/en../../images/Logstash/6.4/event-dependent-configuration.html#conditionals)

### 5.1 语法

```sh
if EXPRESSION {
  ...
} else if EXPRESSION {
  ...
} else {
  ...
}
```

logstash支持以下比较运算符：

* &lt;font color=&#34;#f8070d&#34; size=3&gt;`==`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`!=`&lt;/font&gt;、 &lt;font color=&#34;#f8070d&#34; size=3&gt;`&lt;`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`&gt;`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`&lt;=`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`&gt;=`&lt;/font&gt;
* regexp：&lt;font color=&#34;#f8070d&#34; size=3&gt;`=~`&lt;/font&gt;&lt;font color=&#34;#f8070d&#34; size=3&gt;`!~`&lt;/font&gt;（检查右边的模式与左边的字符串值）
* 包含：&lt;font color=&#34;#f8070d&#34; size=3&gt;`in`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`not in`&lt;/font&gt;
* 布尔运算符是： &lt;font color=&#34;#f8070d&#34; size=3&gt;`and`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`or`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`nand`&lt;/font&gt; &lt;font color=&#34;#f8070d&#34; size=3&gt;`xor（异或）`&lt;/font&gt;
* 支持的一元运算符是：&lt;font color=&#34;#f8070d&#34; size=3&gt;`!`&lt;/font&gt;

***
注：条件判断语句不包括数学运算符，如下写法logstash会报错
***

```sh
if [seg_num]&lt;[total_seg]-1 {
  ...
} 
```

### 5.2 logstash常用条件判断语句
为了避免多个input的数据提交到Elasticsearch后共存于一个索引中。使用index参数来为每个不同的type建立单独的索引。

```sh
input{
  file{
    path=&gt; &#34;/usr/local/elasticsearch-2.3.2/logs/elasticsearch.log.*&#34;
    type=&gt; &#34;elasticsearch&#34;
    start_position=&gt; &#34;beginning&#34;
  }
  
  file{
    path=&gt; &#34;/var/log/secure&#34;
    type=&gt; &#34;secure&#34;
    start_position=&gt; &#34;beginning&#34;
  }
}

output{
  if [type] == &#34;elasticsearch&#34; {  
    elasticsearch {
    hosts=&gt; [&#34;192.168.44.129:9200&#34;]
    index=&gt; &#34;elasticsearch-%{&#43;YYYY-MM}&#34;
    }
  }
  if [type] == &#34;secure&#34; {  
    elasticsearch {
    hosts=&gt; [&#34;192.168.44.129:9200&#34;]
    index=&gt; &#34;secure-%{&#43;YYYY-MM}&#34;
    }
  }
}  
```

使用in运算符来测试字段是否包含特定字符串，键或（对于列表）元素：

```sh
filter {
  if [foo] in [foobar] {
    mutate { add_tag =&gt; &#34;field in field&#34; }
  }
  if [foo] in &#34;foo&#34; {
    mutate { add_tag =&gt; &#34;field in string&#34; }
  }
  if &#34;hello&#34; in [greeting] {
    mutate { add_tag =&gt; &#34;string in field&#34; }
  }
  if [foo] in [&#34;hello&#34;, &#34;world&#34;, &#34;foo&#34;] {
    mutate { add_tag =&gt; &#34;field in list&#34; }
  }
  if [missing] in [alsomissing] {
    mutate { add_tag =&gt; &#34;shouldnotexist&#34; }
  }
  if !(&#34;foo&#34; in [&#34;hello&#34;, &#34;world&#34;]) {
    mutate { add_tag =&gt; &#34;shouldexist&#34; }
  }
}  
```

在单个条件中指定多个表达式：

```sh
if [loglevel] == &#34;ERROR&#34; and [deployment] == &#34;production&#34; {
    pagerduty {
    ...
    }
}
```
