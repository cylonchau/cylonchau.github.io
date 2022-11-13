# 

Pod的资源限制



**requests**：资源需求，要想运行容器硬件的最低保障。 request &lt;= limits  需求用于在调度时以确保要拥有这么多资源，满足Pod在运行某些功能时的基本需求

**limits**：用于限制容器无论怎么运行绝对不能超过的资源阈值。

**CPU计量单位**：

​	对应的一颗逻辑CPU。（2核双线程CPU被虚拟成4颗CPU）。

​	一个CPU还能被单独划分子单位，1CPU=1000m ==millicores==  500m=0.5CPU

**内存计量单位**:

​	E P T G M K

​	Ei Pi Ti

一个Pod内容器最多可以使用0.5个CPU `cpu.limit=500m`；requests，要想确保容器正常运行，至少确保`cpu.requests=200m`,为运行环境的最低保证（要确保被调用的目标节点上有`cpu.requests`的空闲资源可用）。每一个节点有多少资源已经被分配出去，是根据节点上所有运行的容器的`cpu.requests`之和。

requests定义能够被调度器作为基本衡量条件的调度值使用的。对每一个节点来讲，会将所有容器的资源需求量之和作为已分配出去的配额

资源限制及资源需求是定义在容器级别的 https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

kubectl explain 

pods.spec.containers.resources

- limits 
- requests



在定义时，需求和维度，可同时也可指定一个。可只定义需求不定义限制。也可只定义限制不定义需求。需求和限制在定义时，可单独指定cpu或memory，或全部指定。一般而言，通常都需要做一个完整的限制

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pods-test
  labels:
    app: myapp
    tir: frontend
spec:
  containers:
  - name: myapp
    image: ikubernetes/stress-ng
    command: [&#34;usr/bin/stress-ng&#34;, &#34;-c 1&#34;, &#34;--metrics-brief&#34;]
    resources:
      requests:
        cpu: &#34;200m&#34;
        memory: &#34;128Mi&#34;
      limits:
        cpu: &#34;500m&#34;
        memory: &#34;400Mi&#34;
```

容器内运行命令查看，此处显示的效果只是根据其limits有关，而不是requests

```bash
kubectl exec pods-test -- top
Mem: 2029896K used, 5979304K free, 386232K shrd, 2108K buff, 1528808K cached
CPU:  13% usr   0% sys   0% nic  86% idle   0% io   0% irq   0% sirq
Load average: 0.33 0.29 0.23 2/325 11
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    6     1 root     R     6884   0%   2  13% {stress-ng-cpu} usr/bin/stress-ng 
    7     0 root     R     1504   0%   3   0% top
    1     0 root     S     6244   0%   0   0% usr/bin/stress-ng -c 1 --metrics-b
```

物理机所使用CPU资源，物理机为4C，限制为500m 可看到使用1/8 CPU。

```bash
top - 20:27:35 up 4 days,  2:47,  1 user,  load average: 0.64, 0.37, 0.26
Tasks: 198 total,   3 running, 195 sleeping,   0 stopped,   0 zombie
%Cpu0  :   50.5/0.3    51[||||||||||||||||||||||||||||||||||||||||||||||||||                                                  ]
%Cpu1  :   0.3/1.0     1[|                                                                                                   ]
%Cpu2  :   0.7/0.7     1[||                                                                                                  ]
%Cpu3  :   0.7/0.7     1[||                                                                                                  ]
KiB Mem :  8009200 total,  5978284 free,   398248 used,  1632668 buff/cache
KiB Swap:  4194300 total,  4193532 free,      768 used.  6931260 avail Mem 

PID   USER      PR  NI     VIRT    RES    SHR S   %CPU  %MEM   TIME&#43;     COMMAND
5188  root      20   0     6884   2136    252 R   50.0  0.0    2:21.13   stress-ng-cpu
7065  root      20   0  1113176  81524  36032 S   1.3   1.0    92:09.68  kubelet
4112  root      20   0   162024   2368   1596 R   0.3   0.0    0:00.97   top
```

---

说明：容器内可见资源量与可用资源量并不相同。使用资源限制后，所看到的为整个节点上的系统资源，而不仅仅是当前一个容器内的资源量。

---

在容器内限制了资源后，会自动分配Qos类别，默认QoS类别有三个：

- Guranteed 确保，此类Pod有最高优先级，每个容器同时设置CPU和内存的requests和limits，并且requests等于limits时，会自动归类为此类别。
- Burstable 至少有一个容器设置了CPU或内存资源的requests属性。会自动归类为此类别。
- BestEffort 没有任何一个容器设置了requests或limits属性

已占用量与需求量大的比例大的将优先被kill掉

​	

cAdvisor 负责收集当前节点上各pod上的各容器，以及节点级的系统资源指标占用量
