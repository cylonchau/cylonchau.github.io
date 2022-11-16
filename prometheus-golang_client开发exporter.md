# prometheus golang_client开发Exporter


exporter是一个独立运行的采集程序，其中的功能需要有这三部分

- 自身是HTTP服务器，可以相应从外部发过来的HTTP GET请求。
- 自身需要运行在后台，并可以定期触发抓取本地的监控数据。
- 返回给prometheus_server的内容是要符合prometheus规定的metrics类型的key-Value




promethes监控中对于采集过来的数据统一称为metrice数据

**Metrics**，为某个系统某个服务做监控、做统计，就需要用到Metrics.

metrics是一种对采样数掘的总称（metrics并不代表某一种具体的数据格式是一种对于度星计算单位的抽象）


metrics的几种主要的类型


## METRIC TYPES

prometheus客户端库提供4中metric类型

- counter 计数器，累加指标
- gauge 测量指标
- summary 概略
- histogram 直方图

### counter 




Counter计数器，累加的指标数据，随时间逐步增加，如程序运行次数、运行错误发生总数。如网卡流量，代表持续增加的数据包或者传输字节的累加值

比如对用户访问量的采样数据

我们的产品被用户访问一次就是1过了10分钟后积累到100

过一天后累积到20000

一周后积累到100000-150000

```go
test = prometheus.NewSummaryVec(
		prometheus.SummaryOpts{
			Name:       "zhangsan",
			Help:       "username",
			Objectives: map[float64]float64{0.5: 0.05, 0.9: 0.01, 0.99: 0.001},
		},
		[]string{"service"},
	)
```



```
# HELP zhangsan username
# TYPE zhangsan summary
zhangsan{service="aaa",quantile="0.5"} 0.4933459627210085
zhangsan{service="aaa",quantile="0.9"} 0.884503005897664
zhangsan{service="aaa",quantile="0.99"} 0.9939536606026
zhangsan_sum{service="aaa"} 3145.830341777395
zhangsan_count{service="aaa"} 6308
```
在每次执行后值会累加


```
# HELP zhangsan username
# TYPE zhangsan summary
zhangsan{service="aaa",quantile="0.5"} 0.4933459627210085
zhangsan{service="aaa",quantile="0.9"} 0.884503005897664
zhangsan{service="aaa",quantile="0.99"} 0.9945251964629818
zhangsan_sum{service="aaa"} 3165.232975252529
zhangsan_count{service="aaa"} 6347
```


### gauge


最简单的度量指标，只有一个简单的返回值，或者叫瞬时状态，例如，我们想衡量一个特处理队列中任务的个数。

用更简单的方式举个例子

例如：如果我要监控覆盘容量或者内存的使用量，那么就应该使用Gauges的metrics格式来度量，因为硬盘的容量或者内存的使用量是随着时间的推移不断的瞬时，没有规则的变化。这种变化没有规律，当前是多少，采集回来的就是多少，既不能肯定是一直持续增长，也不能肯定是一直降低，是多少就是多少这种就是Gauges使用类型的代表。

如图所示CPU的上下厚动就是采集使用Gauge形式的metrics数据，没有规律是多少就得到多少，然后显示出来。


gauge代表采集的是一个单一的数据，此数据可增可减，如内存使用情况，cpu使用情况等。


```go
gaugetest = prometheus.NewGauge(prometheus.GaugeOpts{
		Name: "lisi",
		Help: "password",
})
```


```
# HELP lisi password
# TYPE lisi gauge
lisi 0.4578056215001356
```

```
# HELP lisi password
# TYPE lisi gauge
lisi 0.44668242347036363
```

https://github.com/lwhile/workDoc

### summary 概略



### histogram 比例型的估算数值

Histogram 统计数据的分布情况。比如最小值，最大值，中间值，还有中位数，75百分位，90百分位，95百分位，98百分位，99百分位，和99.9百分位的值（percentiles）。

**这是一种特殊的metrics数据类型，代表的是一种**近似的百分比估算数值

比如我们在企业工作中经常接触这种数据

Http.response_time HTTP响应时间

代表的是一次用户HTTP请求在系统传输和执行过程中总共花费的时间

nginx中的也会记录这一项数值在日志中

那么问题来了

我们做一个假设

如果我们想通过监控的方式抓取当天的nginx accoss.log，并且想监控用户的访问时间我们应该怎么做呢？同学们肯定很容易想到
简单制把日志每行的`http_response_time` 数值统统采集下来期然后计算一下总的平均值即可。那么假如我们采集到今天一天的访问量是100万次然后把这100万次的`http_response_time` 全都加一超然后除以100万最后得出来一个值

0.05秒=50毫秒

这个数据的意文大么？

假如今天中午1：00的时候发生了一次线上故障系统整体的访问变得非常缓慢大部分的用户请求时间都达到了0.5~1秒作用但是这一段时间只持续了5分钟，总的一天的平均值并不能表现得出来我们如何在 1:00 ~ 1:05 的时候实现报警呢？

在举个例子：

就算我们一天下来线上没有发生故障大部分用户的响应时间都在0.05秒（通过总时间/总次数得出）但是我们不要忘了任何系统中都一定存在慢请求就是有一少部分的用户请求时间会比总的平均值大很多甚至接近5秒10秒的也有（这种情况很普遍因为各种因素可能是软件本身的bug也可能是系统的原因更有可能是少部分用户的使用途径中出现了问题）



那么我们的监控需要发现和报警这种少部分的特殊状况，用总平均能获得吗？

如果采用总平均的方式，那不管发生什么特殊情况，因为大部分的用户响应都是正常的你永远也发现不了少部分的问题所以Histogram的metrics类型在这种时候就派上用场了通过histogram类型（prometheus中其实提供了一个基于histogram算法的函数可以直接使用）可以分别统计出全部用户的单应时间中
~=0.05秒的量有多少0 ~ 0.05秒的有多少，>2秒的有多少>10秒的有多少

我们就可以很清晰的看到当前我们的系统中必于基本正常状态的有多少百分比的用户（或者是请求）多少处于速度极快的用户，多少处于慢请求或者有问题的请求

metrics的类型其实还有另外的类型

但是在我们大米运维的课程中我们最主要使用的就是counter ganga 和 histogram

#### kv的数据形式

对于采集回来的数据类型再往细了说必须要以一种具体的数据格式供我们查看和使用那么我们来看一下一个exporter 给我们果集来的服务器上的k/v形式metrics数据当一个exporter被安装和运行在被监控的服务器上后，使用简单的curl命令就可以看到exporer帮我们采集到metrics数据的样子，以k/v的形式展现和保存

```bash
curl localhost:9100/metrics
```
