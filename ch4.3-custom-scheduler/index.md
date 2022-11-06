# 基于Prometheus的Kubernetes调度器

[toc]

## Overview

本文将深入讲解 如何扩展 Kubernetes scheduler 中各个扩展点如何使用，与扩展scheduler的原理，这些是作为扩展 *scheduler* 的所需的知识点。最后会完成一个实验，基于网络流量的调度器。

## kubernetes调度配置 

kubernetes集群中允许运行多个不同的 *scheduler*  ，也可以为Pod指定不同的调度器进行调度。在一般的Kubernetes调度教程中并没有提到这点，这也就是说，对于亲和性，污点等策略实际上并没有完全的使用kubernetes调度功能，在之前的文章中提到的一些调度插件，如基于端口占用的调度 `NodePorts` 等策略一般情况下是没有使用到的，本章节就是对这部分内容进行讲解，这也是作为扩展调度器的一个基础。

### Scheduler Configuration &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt; 

*kube-scheduler* 提供了配置文件的资源，作为给 *kube-scheduler* 的配置文件，启动时通过 `--onfig=` 来指定文件。目前各个kubernetes版本中使用的 `KubeSchedulerConfiguration` 为，

- 1.21 之前版本使用 `v1beta1`
- 1.22 版本使用 `v1beta2` ，但保留了 `v1beta1`
- 1.23, 1.24, 1.25 版本使用 `v1beta3` ，但保留了  `v1beta2`，删除了 `v1beta1`

下面是一个简单的 *kubeSchedulerConfiguration* 示例，其中 `kubeconfig` 与启动参数 `--kubeconfig` 是相同的功效。而 *kubeSchedulerConfiguration* 与其他组件的配置文件类似，如 *kubeletConfiguration* 都是作为服务启动的配置文件。

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /etc/srv/kubernetes/kube-scheduler/kubeconfig
```

&gt; Notes: `--kubeconfig` 与 `--config` 是不可以同时指定的，指定了 `--config` 则其他参数自然失效 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

### kubeSchedulerConfiguration使用

通过配置文件，用户可以自定义多个调度器，以及配置每个阶段的扩展点。而插件就是通过这些扩展点来提供在整个调度上下文中的调度行为。

下面配置是对于配置扩展点的部分的一个示例，关于扩展点的讲解可以参考kubernetes官方文档调度上下文部分

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
profiles:
  - plugins:
      score:
        disabled:
        - name: PodTopologySpread
        enabled:
        - name: MyCustomPluginA
          weight: 2
        - name: MyCustomPluginB
          weight: 1
```

&gt; Notes: 如果name=&#34;*&#34; 的话，这种情况下将禁用/启用对应扩展点的所有插件

既然kubernetes提供了多调度器，那么对于配置文件来说自然支持多个配置文件，profile也是列表形式，只要指定多个配置列表即可，下面是多配置文件示例，其中，如果存在多个扩展点，也可以为每个调度器配置多个扩展点。

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
  	plugins:
      preScore:
        disabled:
        - name: &#39;*&#39;
      score:
        disabled:
        - name: &#39;*&#39;
  - schedulerName: no-scoring-scheduler
    plugins:
      preScore:
        disabled:
        - name: &#39;*&#39;
      score:
        disabled:
        - name: &#39;*&#39;
```

### scheduler调度插件 &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt; 

*kube-scheduler* 默认提供了很多插件作为调度方法，默认不配置的情况下会启用这些插件，如：

- ***ImageLocality***：调度将更偏向于Node存在容器镜像的节点。扩展点：`score`.
- ***TaintToleration***：实现污点与容忍度功能。扩展点：`filter`, `preScore`, `score`.
- ***NodeName***：实现调度策略中最简单的调度方法 `NodeName` 的实现。扩展点：`filter`.
- ***NodePorts***：调度将检查Node端口是否已占用。扩展点：`preFilter`, `filter`.
- ***NodeAffinity***：提供节点亲和性相关功能。扩展点：`filter`, `score`.
- ***PodTopologySpread***：实现Pod拓扑域的功能。扩展点：`preFilter`, `filter`, `preScore`, `score`.
- ***NodeResourcesFit***：该插件将检查节点是否拥有 Pod 请求的所有资源。使用以下三种策略之一：`LeastAllocated` （默认）`MostAllocated` 和 `RequestedToCapacityRatio`。扩展点：`preFilter`, `filter`, `score`.
- ***VolumeBinding***：检查节点是否有或是否可以绑定请求的 [卷](https://kubernetes.io/docs/concepts/storage/volumes/). 扩展点：`preFilter`, `filter`, `reserve`, `preBind`, `score`.
- ***VolumeRestrictions***：检查安装在节点中的卷是否满足特定于卷提供程序的限制。扩展点：`filter`.
- ***VolumeZone***：检查请求的卷是否满足它们可能具有的任何区域要求。扩展点：`filter`.
- ***InterPodAffinity***： 实现Pod 间的亲和性与反亲和性的功能。扩展点：`preFilter`, `filter`, `preScore`, `score`.
- ***PrioritySort***：提供基于默认优先级的排序。扩展点：`queueSort`.

对于更多配置文件使用案例可以参考官方给出的文档

## 如何扩展kube-scheduler &lt;sup&gt;&lt;a href=&#34;#4&#34;&gt;[4]&lt;/a&gt;&lt;/sup&gt; 

当在第一次考虑编写调度程序时，通常会认为扩展 *kube-scheduler* 是一件非常困难的事情，其实这些事情 kubernetes 官方早就想到了，kubernetes为此在 1.15 版本引入了framework的概念，framework旨在使 *scheduler* 更具有扩展性。

*framework* 通过重新定义 各扩展点，将其作为 *plugins* 来使用，并且支持用户注册 `out of tree` 的扩展，使其可以被注册到 *kube-scheduler* 中，下面将对这些步骤进行分析。

### 定义入口

*scheduler* 允许进行自定义，但是对于只需要引用对应的 [NewSchedulerCommand](https://github.com/kubernetes/kubernetes/blob/e37e4ab4cc8dcda84f1344dda47a97bb1927d074/cmd/kube-scheduler/app/server.go#L64-L117)，并且实现自己的 *plugins* 的逻辑即可。

```go
import (
    scheduler &#34;k8s.io/kubernetes/cmd/kube-scheduler/app&#34;
)

func main() {
    command := scheduler.NewSchedulerCommand(
            scheduler.WithPlugin(&#34;example-plugin1&#34;, ExamplePlugin1),
            scheduler.WithPlugin(&#34;example-plugin2&#34;, ExamplePlugin2))
    if err := command.Execute(); err != nil {
        fmt.Fprintf(os.Stderr, &#34;%v\n&#34;, err)
        os.Exit(1)
    }
}
```

而 **NewSchedulerCommand** 允许注入 out of tree plugins，也就是注入外部的自定义 plugins，这种情况下就无需通过修改源码方式去定义一个调度器，而仅仅通过自行实现即可完成一个自定义调度器。

```go
// WithPlugin 用于注入out of tree plugins 因此scheduler代码中没有其引用。
func WithPlugin(name string, factory runtime.PluginFactory) Option {
	return func(registry runtime.Registry) error {
		return registry.Register(name, factory)
	}
}
```

### 插件实现

对于插件的实现仅仅需要实现对应的扩展点接口。下面通过内置插件进行分析

对于内置插件 `NodeAffinity` ,我们通过观察他的结构可以发现，实现插件就是实现对应的扩展点抽象 *interface* 即可。

![image-20220807212221684](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220807212221684.png)

#### 定义插件结构体

其中 [framework.FrameworkHandle](https://github.com/kubernetes/kubernetes/blob/e37e4ab4cc8dcda84f1344dda47a97bb1927d074/pkg/scheduler/framework/v1alpha1/interface.go#L495-L524) 是提供了Kubernetes API与 *scheduler* 之间调用使用的，通过结构可以看出包含 lister，informer等等，这个参数也是必须要实现的。

```go
type NodeAffinity struct {
	handle framework.FrameworkHandle
}
```

#### 实现对应的扩展点

```go
func (pl *NodeAffinity) Score(ctx context.Context, state *framework.CycleState, pod *v1.Pod, nodeName string) (int64, *framework.Status) {
	nodeInfo, err := pl.handle.SnapshotSharedLister().NodeInfos().Get(nodeName)
	if err != nil {
		return 0, framework.NewStatus(framework.Error, fmt.Sprintf(&#34;getting node %q from Snapshot: %v&#34;, nodeName, err))
	}

	node := nodeInfo.Node()
	if node == nil {
		return 0, framework.NewStatus(framework.Error, fmt.Sprintf(&#34;getting node %q from Snapshot: %v&#34;, nodeName, err))
	}

	affinity := pod.Spec.Affinity

	var count int64
	// A nil element of PreferredDuringSchedulingIgnoredDuringExecution matches no objects.
	// An element of PreferredDuringSchedulingIgnoredDuringExecution that refers to an
	// empty PreferredSchedulingTerm matches all objects.
	if affinity != nil &amp;&amp; affinity.NodeAffinity != nil &amp;&amp; affinity.NodeAffinity.PreferredDuringSchedulingIgnoredDuringExecution != nil {
		// Match PreferredDuringSchedulingIgnoredDuringExecution term by term.
		for i := range affinity.NodeAffinity.PreferredDuringSchedulingIgnoredDuringExecution {
			preferredSchedulingTerm := &amp;affinity.NodeAffinity.PreferredDuringSchedulingIgnoredDuringExecution[i]
			if preferredSchedulingTerm.Weight == 0 {
				continue
			}

			// TODO: Avoid computing it for all nodes if this becomes a performance problem.
			nodeSelector, err := v1helper.NodeSelectorRequirementsAsSelector(preferredSchedulingTerm.Preference.MatchExpressions)
			if err != nil {
				return 0, framework.NewStatus(framework.Error, err.Error())
			}

			if nodeSelector.Matches(labels.Set(node.Labels)) {
				count &#43;= int64(preferredSchedulingTerm.Weight)
			}
		}
	}

	return count, nil
}
```

最后在通过实现一个 [New](https://github.com/kubernetes/kubernetes/blob/e37e4ab4cc8dcda84f1344dda47a97bb1927d074/pkg/scheduler/framework/plugins/nodeaffinity/node_affinity.go#L116-L118) 函数来提供注册这个扩展的方法。通过这个 *New* 函数可以在 `main.go` 中将其作为 out of tree plugins 注入到 *scheduler* 中即可

```go
// New initializes a new plugin and returns it.
func New(_ runtime.Object, h framework.FrameworkHandle) (framework.Plugin, error) {
	return &amp;NodeAffinity{handle: h}, nil
}
```

## 实验：基于网络流量的调度 &lt;sup&gt;&lt;a href=&#34;#7&#34;&gt;[7]&lt;/a&gt;&lt;/sup&gt; 

通过上面阅读了解到了如何扩展 *scheduler* 插件，下面实验将完成一个基于流量的调度，通常情况下，网络一个Node在一段时间内使用的网络流量也是作为生产环境中很常见的情况。例如在配置均衡的多个主机中，主机A作为业务拉单脚本运行，主机B作为计算服务运行。通常来说计算服务会使用更多的系统资源，而拉单需要更多的是网络流量，此时在调度时，默认调度器有限选择的是系统空闲资源多的节点，这种情况下如果有Pod被调度到该节点上，那么可能双方业务都会收到影响（前端代理觉得这个节点连接数少会被大量调度，而拉单脚本因为网络带宽的占用降低了效能）。

### 实验环境

- 一个kubernetes集群，至少保证有两个节点。
- 提供的kubernetes集群都需要安装prometheus node_exporter，可以是集群内部的，也可以是集群外部的，这里使用的是集群外部的。
- 对 [promQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) 与 [client_golang](https://github.com/prometheus/client_golang) 有所了解

**实验大致分为以下几个步骤**：

- 定义插件API
    - 插件命名为 `NetworkTraffic`
- 定义扩展点
    - 这里使用了 Score 扩展点，并且定义评分的算法
- 定义分数获取途径（从prometheus指标中拿到对应的数据）
- 定义对自定义调度器的参数传入
- 将项目部署到集群中（集群内部署与集群外部署）
- 实验的结果验证

实验将仿照内置插件 [nodeaffinity](https://github.com/kubernetes/kubernetes/blob/e37e4ab4cc8dcda84f1344dda47a97bb1927d074/pkg/scheduler/framework/plugins/nodeaffinity/node_affinity.go) 完成代码编写，为什么选择这个插件，只是因为这个插件相对比较简单，并且与我们实验目的基本相同，其实其他插件也是同样的效果。

整个实验的代码上传至 github.com/CylonChau/customScheduler

### 实验开始

#### 错误处理

在初始化项目时，`go mod tidy` 等操作时，会遇到大量下面的错误

```bash
go: github.com/GoogleCloudPlatform/spark-on-k8s-operator@v0.0.0-20210307184338-1947244ce5f4 requires
        k8s.io/apiextensions-apiserver@v0.0.0: reading k8s.io/apiextensions-apiserver/go.mod at revision v0.0.0: unknown revision v0.0.0
```

kubernetes issue #79384 &lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt; 中有提到这个问题，粗略浏览下没有说明为什么会出现这个问题，在最下方有个大佬提供了一个脚本，出现上述问题无法解决时直接运行该脚本后正常。

```bash
#!/bin/sh
set -euo pipefail

VERSION=${1#&#34;v&#34;}
if [ -z &#34;$VERSION&#34; ]; then
    echo &#34;Must specify version!&#34;
    exit 1
fi
MODS=($(
    curl -sS https://raw.githubusercontent.com/kubernetes/kubernetes/v${VERSION}/go.mod |
    sed -n &#39;s|.*k8s.io/\(.*\) =&gt; ./staging/src/k8s.io/.*|k8s.io/\1|p&#39;
))
for MOD in &#34;${MODS[@]}&#34;; do
    V=$(
        go mod download -json &#34;${MOD}@kubernetes-${VERSION}&#34; |
        sed -n &#39;s|.*&#34;Version&#34;: &#34;\(.*\)&#34;.*|\1|p&#39;
    )
    go mod edit &#34;-replace=${MOD}=${MOD}@${V}&#34;
done
go get &#34;k8s.io/kubernetes@v${VERSION}&#34;
```

#### 定义插件API

通过上面内容描述了解到了定义插件只需要实现对应的扩展点抽象 *interface* ，那么可以初始化项目目录 `pkg/networtraffic/networktraffice.go`。

定义插件名称与变量

```go
const Name = &#34;NetworkTraffic&#34;
var _ = framework.ScorePlugin(&amp;NetworkTraffic{})
```

定义插件的结构体

```go
type NetworkTraffic struct {
    // 这个作为后面获取node网络流量使用
	prometheus *PrometheusHandle
	// FrameworkHandle 提供插件可以使用的数据和一些工具。
	// 它在插件初始化时传递给 plugin 工厂类。
	// plugin 必须存储和使用这个handle来调用framework函数。
	handle framework.FrameworkHandle
}
```

#### 定义扩展点

因为选用 Score 扩展点，需要定义对应的方法，来实现对应的抽象

```go
func (n *NetworkTraffic) Score(ctx context.Context, state *framework.CycleState, p *corev1.Pod, nodeName string) (int64, *framework.Status) {
    // 通过promethes拿到一段时间的node的网络使用情况
	nodeBandwidth, err := n.prometheus.GetGauge(nodeName)
	if err != nil {
		return 0, framework.NewStatus(framework.Error, fmt.Sprintf(&#34;error getting node bandwidth measure: %s&#34;, err))
	}
	bandWidth := int64(nodeBandwidth.Value)
	klog.Infof(&#34;[NetworkTraffic] node &#39;%s&#39; bandwidth: %s&#34;, nodeName, bandWidth)
	return bandWidth, nil // 这里直接返回就行
}
```

接下来需要对结果归一化，这里就回到了调度框架中扩展点的执行问题上了，通过源码可以看出，Score 扩展点需要实现的并不只是这单一的方法。

```go
// Run NormalizeScore method for each ScorePlugin in parallel.
parallelize.Until(ctx, len(f.scorePlugins), func(index int) {
    pl := f.scorePlugins[index]
    nodeScoreList := pluginToNodeScores[pl.Name()]
    if pl.ScoreExtensions() == nil {
        return
    }
    status := f.runScoreExtension(ctx, pl, state, pod, nodeScoreList)
    if !status.IsSuccess() {
        err := fmt.Errorf(&#34;normalize score plugin %q failed with error %v&#34;, pl.Name(), status.Message())
        errCh.SendErrorWithCancel(err, cancel)
        return
    }
})
```

通过上面代码了解到，实现 `Score ` 就必须实现 `ScoreExtensions`，如果没有实现则直接返回。而根据 `nodeaffinity` 中示例发现这个方法仅仅返回的是这个扩展点对象本身，而具体的归一化也就是真正进行打分的操作在 `NormalizeScore` 中。

```go
// NormalizeScore invoked after scoring all nodes.
func (pl *NodeAffinity) NormalizeScore(ctx context.Context, state *framework.CycleState, pod *v1.Pod, scores framework.NodeScoreList) *framework.Status {
	return pluginhelper.DefaultNormalizeScore(framework.MaxNodeScore, false, scores)
}

// ScoreExtensions of the Score plugin.
func (pl *NodeAffinity) ScoreExtensions() framework.ScoreExtensions {
	return pl
}
```

而在 [framework](https://github.com/kubernetes/kubernetes/blob/e37e4ab4cc8dcda84f1344dda47a97bb1927d074/pkg/scheduler/framework/runtime/framework.go#L692-L700) 中，真正执行的操作的方法也是 `NormalizeScore()`

```go
func (f *frameworkImpl) runScoreExtension(ctx context.Context, pl framework.ScorePlugin, state *framework.CycleState, pod *v1.Pod, nodeScoreList framework.NodeScoreList) *framework.Status {
	if !state.ShouldRecordPluginMetrics() {
		return pl.ScoreExtensions().NormalizeScore(ctx, state, pod, nodeScoreList)
	}
	startTime := time.Now()
	status := pl.ScoreExtensions().NormalizeScore(ctx, state, pod, nodeScoreList)
	f.metricsRecorder.observePluginDurationAsync(scoreExtensionNormalize, pl.Name(), status, metrics.SinceInSeconds(startTime))
	return status
}
```

下面来实现对应的方法

在 *NormalizeScore* 中需要实现具体的选择node的算法，因为对node打分结果的区间为 $[0,100]$ ，所以这里实现的算法公式将为 $最高分 - (当前带宽 / 最高最高带宽 * 100)$，这样就保证了，带宽占用越大的机器，分数越低。

例如，最高带宽为200000，而当前Node带宽为140000，那么这个Node分数为：$max - \frac{140000}{200000}\times 100 = 100 - (0.7\times100)=30$ 

```go
// 如果返回framework.ScoreExtensions 就需要实现framework.ScoreExtensions
func (n *NetworkTraffic) ScoreExtensions() framework.ScoreExtensions {
	return n
}

// NormalizeScore与ScoreExtensions是固定格式
func (n *NetworkTraffic) NormalizeScore(ctx context.Context, state *framework.CycleState, pod *corev1.Pod, scores framework.NodeScoreList) *framework.Status {
	var higherScore int64
	for _, node := range scores {
		if higherScore &lt; node.Score {
			higherScore = node.Score
		}
	}
	// 计算公式为，满分 - (当前带宽 / 最高最高带宽 * 100)
	// 公式的计算结果为，带宽占用越大的机器，分数越低
	for i, node := range scores {
		scores[i].Score = framework.MaxNodeScore - (node.Score * 100 / higherScore)
		klog.Infof(&#34;[NetworkTraffic] Nodes final score: %v&#34;, scores)
	}

	klog.Infof(&#34;[NetworkTraffic] Nodes final score: %v&#34;, scores)
	return nil
}
```

&gt; Notes：在kubernetes中最大的node数支持5000个，岂不是在获取最大分数时循环就占用了大量的性能，其实不必担心。*scheduler* 提供了一个参数 `percentageOfNodesToScore`。这个参数决定了这里要循环的数量。更多的细节可以参考官方文档对这部分的说明 &lt;sup&gt;&lt;a href=&#34;#6&#34;&gt;[6]&lt;/a&gt;&lt;/sup&gt; 

**配置插件名称**

为了使插件注册时候使用，还需要为其配置一个名称

```go
// Name returns name of the plugin. It is used in logs, etc.
func (n *NetworkTraffic) Name() string {
	return Name
}
```

#### 定义PrometheusHandle

网络插件的扩展中还存在一个 `prometheusHandle`，这个就是操作prometheus-server拿去指标的动作。

首先需要定义一个 *PrometheusHandle* 的结构体

```go
type PrometheusHandle struct {
	deviceName string // 网络接口名称
	timeRange  time.Duration // 抓取的时间段
	ip         string // prometheus server的连接地址
	client     v1.API // 操作prometheus的客户端
}
```

有了结构就需要查询的动作和指标，对于指标来说，这里使用了 `node_network_receive_bytes_total` 作为获取Node的网络流量的计算方式。由于环境是部署在集群之外的，没有node的主机名，通过 `promQL` 获取，整个语句如下：

```bash
sum_over_time(node_network_receive_bytes_total{device=&#34;eth0&#34;}[1s]) * on(instance) group_left(nodename) (node_uname_info{nodename=&#34;node01&#34;})
```

整个 *Prometheus* 部分如下：

```go
type PrometheusHandle struct {
	deviceName string
	timeRange  time.Duration
	ip         string
	client     v1.API
}

func NewProme(ip, deviceName string, timeRace time.Duration) *PrometheusHandle {
	client, err := api.NewClient(api.Config{Address: ip})
	if err != nil {
		klog.Fatalf(&#34;[NetworkTraffic] FatalError creating prometheus client: %s&#34;, err.Error())
	}
	return &amp;PrometheusHandle{
		deviceName: deviceName,
		ip:         ip,
		timeRange:  timeRace,
		client:     v1.NewAPI(client),
	}
}

func (p *PrometheusHandle) GetGauge(node string) (*model.Sample, error) {
	value, err := p.query(fmt.Sprintf(nodeMeasureQueryTemplate, node, p.deviceName, p.timeRange))
	fmt.Println(fmt.Sprintf(nodeMeasureQueryTemplate, p.deviceName, p.timeRange, node))
	if err != nil {
		return nil, fmt.Errorf(&#34;[NetworkTraffic] Error querying prometheus: %w&#34;, err)
	}

	nodeMeasure := value.(model.Vector)
	if len(nodeMeasure) != 1 {
		return nil, fmt.Errorf(&#34;[NetworkTraffic] Invalid response, expected 1 value, got %d&#34;, len(nodeMeasure))
	}
	return nodeMeasure[0], nil
}

func (p *PrometheusHandle) query(promQL string) (model.Value, error) {
    // 通过promQL查询并返回结果
	results, warnings, err := p.client.Query(context.Background(), promQL, time.Now())
	if len(warnings) &gt; 0 {
		klog.Warningf(&#34;[NetworkTraffic Plugin] Warnings: %v\n&#34;, warnings)
	}

	return results, err
}
```

#### 定义调度器传入的参数

因为需要指定 *prometheus* 的地址，网卡名称，和获取数据的大小，故整个结构体如下，另外，参数结构必须遵循`&lt;Plugin Name&gt;Args` 格式的名称。

```go
type NetworkTrafficArgs struct {
	IP         string `json:&#34;ip&#34;`
	DeviceName string `json:&#34;deviceName&#34;`
	TimeRange  int    `json:&#34;timeRange&#34;`
}
```

为了使这个类型的数据作为 `KubeSchedulerConfiguration` 可以解析的结构，还需要做一步操作，就是在扩展APIServer时扩展对应的资源类型。在这里kubernetes中提供两种方法来扩展 `KubeSchedulerConfiguration` 的资源类型。

一种是旧版中提供了 [framework.DecodeInto](https://github.com/kubernetes/kubernetes/blob/7a98bb2b7c9112935387825f2fce1b7d40b76236/pkg/scheduler/framework/plugins/nodelabel/node_label.go#L65-L80) 函数可以做这个操作

```go
func New(plArgs *runtime.Unknown, handle framework.FrameworkHandle) (framework.Plugin, error) {
	args := Args{}
	if err := framework.DecodeInto(plArgs, &amp;args); err != nil {
		return nil, err
	}
	...
}
```

另外一种方式是必须实现对应的深拷贝方法，例如 [NodeLabel](https://github.com/kubernetes/kubernetes/blob/e37e4ab4cc8dcda84f1344dda47a97bb1927d074/pkg/scheduler/apis/config/types_pluginargs.go#L37-L49) 中的

```go
// &#43;k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object

// NodeLabelArgs holds arguments used to configure the NodeLabel plugin.
type NodeLabelArgs struct {
	metav1.TypeMeta

	// PresentLabels should be present for the node to be considered a fit for hosting the pod
	PresentLabels []string
	// AbsentLabels should be absent for the node to be considered a fit for hosting the pod
	AbsentLabels []string
	// Nodes that have labels in the list will get a higher score.
	PresentLabelsPreference []string
	// Nodes that don&#39;t have labels in the list will get a higher score.
	AbsentLabelsPreference []string
}
```

最后将其注册到register中，整个行为与扩展APIServer是类似的

```go
// addKnownTypes registers known types to the given scheme
func addKnownTypes(scheme *runtime.Scheme) error {
	scheme.AddKnownTypes(SchemeGroupVersion,
		&amp;KubeSchedulerConfiguration{},
		&amp;Policy{},
		&amp;InterPodAffinityArgs{},
		&amp;NodeLabelArgs{},
		&amp;NodeResourcesFitArgs{},
		&amp;PodTopologySpreadArgs{},
		&amp;RequestedToCapacityRatioArgs{},
		&amp;ServiceAffinityArgs{},
		&amp;VolumeBindingArgs{},
		&amp;NodeResourcesLeastAllocatedArgs{},
		&amp;NodeResourcesMostAllocatedArgs{},
	)
	scheme.AddKnownTypes(schema.GroupVersion{Group: &#34;&#34;, Version: runtime.APIVersionInternal}, &amp;Policy{})
	return nil
}
```

&gt; Notes：对于生成深拷贝函数及其他文件，可以使用 kubernetes 代码库中的脚本 [kubernetes/hack/update-codegen.sh](https://github.com/kubernetes/kubernetes/blob/v1.24.3/hack/update-codegen.sh) 

这里为了方便使用了 *framework.DecodeInto* 的方式。

#### 项目部署

准备 scheduler 的 profile，可以看到，我们自定义的参数，就可以被识别为 *KubeSchedulerConfiguration* 的资源类型了。

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /mnt/d/src/go_work/customScheduler/scheduler.conf
profiles:
- schedulerName: custom-scheduler
  plugins:
    score:
      enabled:
      - name: &#34;NetworkTraffic&#34;
      disabled:
      - name: &#34;*&#34;
  pluginConfig:
    - name: &#34;NetworkTraffic&#34;
      args:
        ip: &#34;http://10.0.0.4:9090&#34;
        deviceName: &#34;eth0&#34;
        timeRange: 60
```

如果需要部署到集群内部，可以打包成镜像

```dockerfile
FROM golang:alpine AS builder
MAINTAINER cylon
WORKDIR /scheduler
COPY ./ /scheduler
ENV GOPROXY https://goproxy.cn,direct
RUN \
    sed -i &#39;s/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g&#39; /etc/apk/repositories &amp;&amp; \
    apk add upx  &amp;&amp; \
    GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -ldflags &#34;-s -w&#34; -o scheduler main.go &amp;&amp; \
    upx -1 scheduler &amp;&amp; \
    chmod &#43;x scheduler

FROM alpine AS runner
WORKDIR /go/scheduler
COPY --from=builder /scheduler/scheduler .
COPY --from=builder /scheduler/scheduler.yaml /etc/
VOLUME [&#34;./scheduler&#34;]
```

部署在集群内部所需的资源清单

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: scheduler-sa
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: scheduler
subjects:
  - kind: ServiceAccount
    name: scheduler-sa
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-scheduler
  namespace: kube-system
  labels:
    component: custom-scheduler
spec:
  selector:
    matchLabels:
      component: custom-scheduler
  template:
    metadata:
      labels:
        component: custom-scheduler
    spec:
      serviceAccountName: scheduler-sa
      priorityClassName: system-cluster-critical
      containers:
        - name: scheduler
          image: cylonchau/custom-scheduler:v0.0.1
          imagePullPolicy: IfNotPresent
          command:
            - ./scheduler
            - --config=/etc/scheduler.yaml
            - --v=3
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10251
            initialDelaySeconds: 15
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10251
```

启动自定义 *scheduler*，这里通过简单的二进制方式启动，所以需要一个kubeconfig做认证文件

```bash
./main --logtostderr=true \
	--address=127.0.0.1 \
	--v=3 \
	--config=`pwd`/scheduler.yaml \
	--kubeconfig=`pwd`/scheduler.conf
```

启动后为了验证方便性，关闭了原来的 *kube-scheduler* 服务，因为原来的  *kube-scheduler* 已经作为HA中的master，所以不会使用自定义的 *scheduler* 导致pod pending。

#### 验证结果

准备一个需要部署的Pod，指定使用的调度器名称

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      schedulerName: custom-scheduler
```

这里实验环境为2个节点的kubernetes集群，master与node01，因为master的服务比node01要多，这种情况下不管怎样，调度结果永远会被调度到node01上。

```bash
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
nginx-deployment-69f76b454c-lpwbl   1/1     Running   0          43s   192.168.0.17   node01   &lt;none&gt;           &lt;none&gt;
nginx-deployment-69f76b454c-vsb7k   1/1     Running   0          43s   192.168.0.16   node01   &lt;none&gt;           &lt;none&gt;
```

而调度器的日志如下

```log
I0808 01:56:31.098189   27131 networktraffic.go:83] [NetworkTraffic] node &#39;node01&#39; bandwidth: %!s(int64=12541068340)
I0808 01:56:31.098461   27131 networktraffic.go:70] [NetworkTraffic] Nodes final score: [{master-machine 0} {node01 12541068340}]
I0808 01:56:31.098651   27131 networktraffic.go:70] [NetworkTraffic] Nodes final score: [{master-machine 0} {node01 71}]
I0808 01:56:31.098911   27131 networktraffic.go:73] [NetworkTraffic] Nodes final score: [{master-machine 0} {node01 71}]
I0808 01:56:31.099275   27131 default_binder.go:51] Attempting to bind default/nginx-deployment-69f76b454c-vsb7k to node01
I0808 01:56:31.101414   27131 eventhandlers.go:225] add event for scheduled pod default/nginx-deployment-69f76b454c-lpwbl
I0808 01:56:31.101414   27131 eventhandlers.go:205] delete event for unscheduled pod default/nginx-deployment-69f76b454c-lpwbl
I0808 01:56:31.103604   27131 scheduler.go:609] &#34;Successfully bound pod to node&#34; pod=&#34;default/nginx-deployment-69f76b454c-lpwbl&#34; node=&#34;no
de01&#34; evaluatedNodes=2 feasibleNodes=2
I0808 01:56:31.104540   27131 scheduler.go:609] &#34;Successfully bound pod to node&#34; pod=&#34;default/nginx-deployment-69f76b454c-vsb7k&#34; node=&#34;no
de01&#34; evaluatedNodes=2 feasibleNodes=2
```



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [scheduling config](https://kubernetes.io/docs/reference/scheduling/config/)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [scheduling-plugins](https://kubernetes.io/docs/reference/scheduling/config/#scheduling-plugins)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [custom scheduler plugins](https://github.com/kubernetes/enhancements/blob/master/keps/sig-scheduling/624-scheduling-framework/README.md#custom-scheduler-plugins-out-of-tree)
&gt;
&gt; &lt;sup id=&#34;5&#34;&gt;[5]&lt;/sup&gt; [ssues #79384](https://github.com/kubernetes/kubernetes/issues/79384)
&gt;
&gt; &lt;sup id=&#34;6&#34;&gt;[6]&lt;/sup&gt; [scheduler perf tuning](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/scheduler-perf-tuning/)
&gt;
&gt; &lt;sup id=&#34;7&#34;&gt;[7]&lt;/sup&gt; [creating a kube-scheduler plugin](https://medium.com/@juliorenner123/k8s-creating-a-kube-scheduler-plugin-8a826c486a1)
