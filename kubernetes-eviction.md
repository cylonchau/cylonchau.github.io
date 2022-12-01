# Understanding Kubernetes - 理解Kubernetes的驱逐机制


驱逐 (***eviction***) 是指终止在Node上运行的Pod，保证workload的可用性，对于使用Kubernetes，了解驱逐机制是很有必要性的，因为通常情况下，Pod被驱逐是需要解决驱逐背后导致的问题，而想要快速定位就需要对驱逐机制进行了解。

## Pod被驱逐原因

Kubernetes官方给出了下属Pod被驱逐的原因：

- 抢占驱逐 (***Preemption and Eviction***) <sup><a href="#1">[1]</a></sup>
- 节点压力驱逐 (***Node-pressure***) <sup><a href="#2">[2]</a></sup>
- 污点驱逐 (***Taints***) <sup><a href="#3">[3]</a></sup>
- 使用API发起驱逐 (***API-initiated***) <sup><a href="#4">[4]</a></sup>
- 排出Node上的Pod (***drain***) <sup><a href="#5">[5]</a></sup>
- 被 controller-manager 驱逐

### 抢占和优先级

抢占是指当节点资源不足以运行新添加的Pod时，*kube-scheduler* 会检查低优先级Pod而后驱逐掉这些Pod以将资源分配给优先级高的Pod。这个过程称为 “抢占” 例如这个实例是 [*kube-proxy* 被驱逐的场景](https://cylonchau.github.io/ch00.0-pod-network-troubleshooting.html#%E9%9B%86%E7%BE%A4pod%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A8%E8%B6%85%E6%97%B6)

### 节点压力驱逐

节点压力驱逐是指，Pod所在节点的资源，如CPU, 内存, inode等，这些资源被分为可压缩资源CPU (***compressible resources***) 与不可压缩资源 (***incompressible resources***) 磁盘IO, 内存等，当不可压缩资源不足时，Pod会被驱逐。对于此类问题的驱逐 是每个计算节点的 kubelet 通过捕获 cAdvisor 指标来监控节点的资源使用情况。

### 被 controller-manager 驱逐

***kube-controller-manager*** 会定期检查节点的状态，如节点处于 `NotReady`  超过一定时间，或Pod部署长时间失败，这些Pod由控制平面 ***controller-manager*** 创建新的Pod已替换存在问题的Pod

### 通过API发起驱逐

Kubernetes为用户提供了驱逐的API，用户可以通过调用API来实现自定义的驱逐。

对于 1.22 以上版本，可以通过API `policy/v1` 进行驱逐

```bash
curl -v \
	-H 'Content-type: application/json' \
	https://your-cluster-api-endpoint.example/api/v1/namespaces/default/pods/quux/eviction -d '\
	{
        "apiVersion": "policy/v1",
        "kind": "Eviction",
        "metadata": {
            "name": "quux",
            "namespace": "default"
        }
    }'
```

例如，要驱逐Pod `netbox-85865d5556-hfg6v`，可以通过下述命令

```bash
# 1.22+
$ curl -v 'https://10.0.0.4:6443/api/v1/namespaces/default/pods/netbox-85865d5556-hfg6v/eviction' \
--header 'Content-Type: application/json' \
--cert /etc/kubernetes/pki/apiserver-kubelet-client.crt \
--key /etc/kubernetes/pki/apiserver-kubelet-client.key \
--cacert /etc/kubernetes/pki/ca.crt \
-d '{
    "apiVersion": "policy/v1",
    "kind": "Eviction",
    "metadata": {
        "name": "netbox-85865d5556-hfg6v",
        "namespace": "default"
    }
}'

# 1.22-
curl -v 'https://10.0.0.4:6443/api/v1/namespaces/default/pods/netbox-85865d5556-hfg6v/eviction' \
--header 'Content-Type: application/json' \
--cert /etc/kubernetes/pki/apiserver-kubelet-client.crt \
--key /etc/kubernetes/pki/apiserver-kubelet-client.key \
--cacert /etc/kubernetes/pki/ca.crt \
-d '{
    "apiVersion": "policy/v1beta1",
    "kind": "Eviction",
    "metadata": {
        "name": "netbox-85865d5556-hfg6v",
        "namespace": "default"
    }
}'
```

可以看到结果，旧Pod被驱逐，而新Pod被创建，在这里实验环境节点较少，所以体现为没有更换节点

```bash
$ kubectl get pods -o wide
NAME                      READY   STATUS        RESTARTS   AGE    IP              NODE             NOMINATED NODE   READINESS GATES
netbox-85865d5556-hfg6v   1/1     Terminating   0          101d   192.168.1.213   master-machine   <none>           <none>
netbox-85865d5556-vlgr4   1/1     Running       0          101d   192.168.0.4     node01           <none>           <none>
netbox-85865d5556-z6vqx   1/1     Running       0          11s    192.168.1.220   master-machine   <none>           <none>
```

#### 通过API驱逐返回状态

- **200 OK|201 Success**：允许驱逐，`Eviction` 类似于向Pod URL发送 `DELETE` 请求
- **429 Too Many Requests**：由于API限速可能会看到该相应，另外也为配置原因，不允许驱逐 `poddisruptionbudget` (PDB是一种保护机制，将总是确保一定数量或百分比的Pod 被自愿驱逐)
- **500 Internal Server Error**：不允许驱逐，存在错误配置，如多个PDB引用一个 Pod

### 排出Node上的Pod

***drain*** 是kubernetes 1.5+之后提供给用户维护命令，通过这个命令 (`kubectl drain <node_name>`) 可以驱逐该节点上运行的所有Pod，已用来对节点主机进行操作（如内核升级，重启）

> Notes：`kubectl drain <node_name>` 一次只能接一个nodename <sup><a href="#6">[6]</a></sup>

### 污点驱逐

污点通常与容忍度同时使用，拥有污点的node，Pod将不会被调度至该节点，而容忍度将允许一定的污点来调度 pod。

在Kubernetes 1.18+后，允许基于污点的驱逐机制，即kubelet在某些情况下会自动添加节点从而进行驱逐：

Kubernetes内置了一些污点，此时 *Controller* 会自动污染节点：

- `node.kubernetes.io/not-ready`: Node故障。对应 NodeCondition 的`Ready` =  `False`。
- `node.kubernetes.io/unreachable`：Node控制器无法访问节点。对应 NodeCondition `Ready`= `Unknown`。
- `node.kubernetes.io/memory-pressure`：Node内存压力。
- `node.kubernetes.io/disk-pressure`：Node磁盘压力。
- `node.kubernetes.io/pid-pressure`：Node有PID压力。
- `node.kubernetes.io/network-unavailable`：Node网络不可用。
- `node.kubernetes.io/unschedulable`：Node不可调度。

## 【转】实例：Pod被驱逐故障排除过程 <sup><a href="#7">[7]</a></sup>

设想一个场景：，有三个工作节点的Kubernetes 集群，版本为 v1.19.0。发现在 worker 1 上运行的一些 pod 被驱逐了

![image-20221129225911143](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221129225911143.png)

<center>图：Pod被驱逐的日志</center>
<center><em>Source：</em>https://www.containiq.com/post/kubernetes-pod-evictions</center><br>

从上图可以看出有很多pod被驱逐了，报错信息也很清楚。由于节点上存储资源不足，导致kubelet触发驱逐过程。

### 方法1：启用auto-scaler

- 向集群添加工作节点，要么部署cluster-autoscaler以根据配置的条件自动扩缩容。

- 只增加worker的本地存储空间，这涉及到虚拟机的扩容，会导致worker节点暂时不可用。

### 方法2：保护关键Pod

在资源清单中指定资源请求和限制，配置QoS (***Quality of Service***)。当kubelet触发驱逐时，将至少保证这些 pod 不受影响。

这种施在一定程度上保证了一些关键Pod的可用性。如果节点出现问题时 Pod 没有被驱逐，这将需要执行更多步骤来查找故障。

运行命令 `kubectl get pods` 结果显示很多 pod 处于 evicted 状态。检查结果将保存在节点的kubelet日志中。查找对应日志使用 `cat /var/paas/sys/log/kubernetes/kubelet.log | grep -i Evicted -C3`。

### 检查思路

#### 查看Pod容忍度

当Pod故障无法连接或节点无法响应时，可以使用 `tolerationSeconds` 配置对应时长长短

```yaml
tolerations:
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 6000
```

#### 查看防止 Pod 驱逐的条件

如果**集群中的节点数小于50，并且故障节点数超过总节点数的55%**，则暂停 Pod 驱逐。在这种情况下，Kubernetes 将尝试驱逐故障节点的工作负载（运行在kubernetes中的APP）。

下属json描述了一个健康的节点

```json
"conditions": [
    {
        "type": "Ready",
        "status": "True",
        "reason": "KubeletReady",
        "message": "kubelet is posting ready status",
        "lastHearbeatTime": "2019-06-05T18:38:35Z",
        "lastTransitionTime": "2019-06-05T11:41:27Z"
    }
]
```

如果就绪条件为 `Unknown ` 或 `False` 的时间超过了 pod-eviction-timeout，node controller 将对分配给该节点上的所有 Pod 执行 `API-initiated` 类型驱逐。

#### 检查Pod的已分配资源

Pod会根据节点的资源使用情况被逐出。被逐出的Pod将会根据分配给Pod的节点资源进行调度。管理驱逐”和“调度”的条件由不同的规则组成。这种结果会导致，被逐出的容器可能会被重新安排到原始节点。因此，要合理分配资源给每个容器。

#### 检查Pod 是否定期失败

Pod 可以被驱逐多次。即如果在 Pod 被驱逐并调度到新节点后该节点中的 Pod 也被驱逐，则该 Pod 将再次被驱逐。

如果驱逐动作是由 *kube-controller-manager* 触发的，则保留处于 *Terminating* 状态的 Pod 。在节点恢复后，Pod将被 自动销毁。如果节点已经被删除或者其他原因无法恢复，可以强制删除Pod。

如果是由 *kubelet* 触发的驱逐，Pod 状态将保留为 Evicted 状态。仅用于后期故障定位，可直接删除。

删除被逐出的 Pod 命令为：

```bash
kubectl get pods | grep Evicted | awk ‘{print $1}’ | xargs kubectl delete pod
```

> Notes：
>
> - 被Kubernetes驱逐的Pod，不会被自动重新创建 pod。如果要重新创建Pod，需要使用replicationcontroller、replicaset和 deployment 机制，这也是上述提到的Kubernetes的工作负载。
> - Pod控制器是协调一组Pod始终为理想状态的控制器，所以会删除后重建，也是Kubernetes 声明式API的特点

## 如何监控被驱逐的Pod

### 使用Prometheus

```bash
kube_pod_status_reason{reason="Evicted"} > 0
```

### 使用 ContainIQ

ContainIQ 是为Kubernetes设计的可观测性工具，其中包含Kubernetes 事件仪表板，这就包括 Pod 驱逐事件

> **Reference**
>
> <sup id="1">[1]</sup> [***Scheduling, Preemption and Eviction***](https://kubernetes.io/docs/concepts/scheduling-eviction/)
>
> <sup id="2">[2]</sup> [***Node-pressure Eviction***](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/)
>
> <sup id="3">[3]</sup> [***Taints and Tolerations***](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
>
> <sup id="4">[4]</sup> [***API-initiated Eviction***](https://kubernetes.io/docs/concepts/scheduling-eviction/api-eviction/)
>
> <sup id="5">[5]</sup> [***Safely Drain a Node***](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
>
> <sup id="6">[6]</sup> [***Draining multiple nodes in parallel***](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/#draining-multiple-nodes-in-parallel) 
>
> <sup id="7">[7]</sup> [***Kubernetes Pod Evictions | Troubleshooting and Examples***](https://www.containiq.com/post/kubernetes-pod-evictions)
>
> <sup id="8">[8]</sup> [***kubernetes pod evicted***](https://sysdig.com/blog/kubernetes-pod-evicted/)
