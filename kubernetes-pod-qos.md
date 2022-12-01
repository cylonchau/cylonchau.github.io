# 理解Kubernetes驱逐核心 - Pod QoS


服务质量 Quality of Service (QoS)，在Kubernetes是用于解决资源抢占，延迟等方向的一种技术，是服务于调度与抢占之间的条件。

## QoS 级别

QoS 与 资源限制紧密相关，正如下属展示，是一个Pod资源限制部分的配置

```yaml
resources:
  limits:
    cpu: 200m
    memory: 1G
  requests:
    cpu: 500m
    memory: 1G
```

而Kubernetes 将Pod QoS 根据 CPU 与 内存的配置，将QoS分为三个等级：

- **Guaranteed**：确保的，只设置 `limits` 或者 `requests` 与 `limits` 为相同时则为该等级
- **Burstable**：可突发的，只设置 `requests` 或 `requests` 低于  `limits` 的场景 
- **Best-effort**： 默认值，如果不设置则为这个等级

## 为什么要关心Pod QoS级别

在Kubernetes中，将资源分为两类：可压缩性资源 “CPU”，不可压缩性资源 “内存”。当可压缩性资源用尽时，不会被终止与驱逐，而不可压缩性资源用尽时，即Pod内存不足，此时会被OOMKiller杀掉，也就是被驱逐等操作，而了解Pod 的QoS级别可以有效避免关键Pod被驱逐。

![image-20221201203746111](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221201203746111.png)

<center>图：Pod QoS分类</center>
<center><em>Source：</em>https://doc.kaas.thalesdigital.io/docs/BestPractices/QOS</center><br>

有上图可知，**BestEffort** 级别的 Pod 能够使用节点上所有资源，浙江导致其他 Pod 出现资源问题。所以这类 Pod 优先级**最低**，如果系统没有内存，将首先被杀死。

## Pod是如何被驱逐的

当节点的计算资源不足时，*kubelet* 会发起驱逐，这个操作是为了避免系统OOM事件，而QoS的等级决定了驱逐的优先级，没有限制资源的 **BestEffort** 类型的Pod最先被驱逐，接下来资源使用率低于 `Requests` 的 **Guaranteed** 与 **Burstable** 将不会被其他Pod的资源使用量而驱逐，其次对于此类Pod而言，如果Pod使用了比配置（`Requests`）更多的资源时，会根据这两个级别Pod的优先级进行驱逐。 **BestEffort** 与  **Burstable **将按照先优先级，后资源使用率顺序进行驱逐

对于磁盘压力来讲，驱逐顺序根据 **BestEffort** ==》**Burstable** ==》**Guaranteed** 进行驱逐

## 如何查看Pod的QoS等级

Pod资源清单中 Status 字段代表Pod QoS等级

```bash
kubectl get pod <pod_name> -o jsonpath='{.status.qosClass}'
```

## 如何配置QoS默认级别

如果不想对每个Pod都配置资源限制，Kubernetes提供了一个API `LimitRange` 可以指定默认的QoS，为Pod提供默认的资源限制，然后准入控制器会增加默认的资源限制 k8s.io/kubernetes/plugin/pkg/admission/limitranger，正如官方给出的[实例](https://kubernetes.io/docs/concepts/policy/limit-range/)一样

> Notes：准入控制器与Pod控制器概念不同，准入控制器是 *kube-apiserver* 请求时的hander chain

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
```

准入控制器会进行检查，而 `LimitRange` 也是一个标准，限制所有Pod的资源限制标准（`Request` 与 `limits` ）必须小于等于 `LimitRange` 配置的格式，例如下列配置将不会被准入

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
```

由于该Pod没有配置 **Limits** ，不符合规范，该Pod不会被调度，错误如下

```bash
Pod "example-conflict-with-limitrange-cpu" is invalid: spec.containers[0].resources.requests: Invalid value: "700m": must be less than or equal to cpu limit
```

如果同时设置 `request` 和 `limit` ，即使大于`LimitRange` 的配置，新的 Pod 也会被成功调度：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-no-conflict-with-limitrange-cpu
spec:
  containers:
  - name: demo
    image: registry.k8s.io/pause:2.0
    resources:
      requests:
        cpu: 700m
      limits:
        cpu: 700m
```


