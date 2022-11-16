# Kubernetes Pod控制器


[toc]

## Kubernetes资源清单

| 类别                       | 名称                                                         |
| -------------------------- | ------------------------------------------------------------ |
| 工作负载型资源（workload） | 运行应用程序，对外提供服务：Pod、ReplicaSet、Deployment、StatefulSet、DaemonSet、Job、Cronjob （ReplicationController在v1.11版本被废弃） |
| 服务发现及负载均衡         | service、Ingress                                             |
| 配置与存储                 | Volume、CSI（容器存储接口                                    |
| 特殊类型存储卷             | ConfigMap（当配置中心来使用的资源类型）、Secret（保存敏感数据）、DownwardAPI（把外部环境中的信息输出给容器） |
| 集群级资源                 | Namespace、Node、Role、ClusterRole、RoleBinding（角色绑定）、ClusterRoleBinding（集群角色绑定） |
| 元数据型资源               | HPA、PodTemplate（Pod模板，用于让控制器创建Pod时使用的模板）、LimitRange（用来定义硬件资源限制的） |


## Kubernetes配置清单使用说明

在Kubernetes中创建资源时，除了命令式创建方式，还可以使用yaml格式的文件来创建符合我们预期期望的pod，这样的yaml文件我们一般称为<font color="#f8070d" size=2>资源清单</font>。资源清单由很多属性或字段所组成。

以yawl格式输出pod的详细信息。


### 资源清单格式

```yaml
kubectl get pod clients -o yaml
```

### Pod资源清单常用字段讲解

在创建资源时，apiserver仅接收JSON格式的资源定义。在使用<font color="#f8070d" size=3>`kubectl run`</font>命令时，自动将给定内容转换成JSON格式。yaml格式提供配置清单，apiserver可自动将其转为JSON格式，（yaml可无损转为json），而后再提交。使用资源配置请清单可带来<font color="#f8070d" size=2>复用效果</font>。


Pod资源配置清单由五个一级字段组成，通过<font color="#f8070d" size=3>`kubectl create -f yamlfile`</font>就可以创建一个Pod


- **<font color="#f8070d" size=3>apiVersion</font>**: 说明对应的对象属于Kubernetes的哪一个API群组名称和版本。给定apiVersion时由两部分组成<font color="#f8070d" size=3>`group/version`</font>，group如果省略表示core（核心组）之意。使用<font color="#f8070d" size=3>`kubectl api-versions`</font>获得当前系统所支持的apiserver版本。alpha 内测版、beta 公测版、stable 稳定版

- **<font color="#f8070d" size=3>kind</font>**: 资源类别，用来指明哪种资源用来初始化成资源对象时使用。

- **<font color="#f8070d" size=3>metadata</font>**: 元数据，内部嵌套很多2级、3级字段。主要提供以下几个字段。
  
    - **<font color="#0215cd" size=3>name</font>**，在同一类别当中name必须是唯一的。
    
    - **<font color="#0215cd" size=3>namespace</font>** 对应的对象属于哪个名称空间，name受限于namespace，不同的namespace中name可以重名。
    
    - **<font color="#0215cd" size=3>lables</font>** key-value数据，对于key名称及value，最多为63个字符，value，可为空。填写时只能使用<font color="#f8070d" size=2>`字母`</font>、<font color="#f8070d" size=2>数字</font>、<font color="#f8070d" size=3>`_`</font>、<font color="#f8070d" size=3>`-`</font>、<font color="#f8070d" size=3>`.`</font>，只能以字母或数字开头及结尾。
    
    - **<font color="#0215cd" size=3>annotations</font>** 资源注解。与label不同的地方在于，它不能用于挑选资源对象，仅用于为对象提供“元数据”。对键值长度没有要求。在构建大型镜像时通常会用其标记对应的资源对象的元数据

- **<font color="#f8070d" size=3>spec</font>**: specification，定义接下来创建的资源对象应该满足的规范（<font color="#f8070d" size=2>期望的状态 disired state</font>）。spec是用户定义的。不同的资源类型，其所需要嵌套的字段各不相同。如果某一字段属性标记为required表示为必选字段，剩余的都为可选字段，系统会赋予其默认值。如果某一字段标记为Cannot be updated，则表示为对象一旦创建后不能改变字段值。可使用<font color="#f8070d" size=3>`kubectl explain pods.spec`</font>查看详情。
  
  - **<font color="#0215cd" size=3>containers</font>** [required]object list
    
    - **<font color="#ffc104" size=3>name</font>**  [string] 定义容器名称
    
    - **<font color="#ffc104" size=3>image</font>** [string] 启动Pod内嵌容器时所使用的镜像。可是顶级、私有、第三方仓库镜像。
    - **<font color="#ffc104" size=3>imagePulLPolicy</font>** [string] 镜像获取的策略，可选参数Always（总是从仓库下载，无论本地有无此镜像）、Never（从不下载，无论本地有无此镜像）、IfNotPresent（本地存在则使用，不存在则从仓库拉去镜像）。如果tag设置为latest，默认值则为<font color="#f8070d" size=3>`Always`</font>，非latest标签，默认值都为<font color="#f8070d" size=3>`IfNotPresent`</font>。
    
    * **<font color="#ffc104" size=3>ports</font>** []object 定义容器内要暴露的端口时，可以暴露多个端口，每个端口应该由多个属性来定义（端口名称、端口号、协议）。<font color="#f8070d" size=2>注意暴露端口仅仅是提供额外信息的，并不能限制系统是否真能暴露</font>。
    
    - **<font color="#ffc104" size=3>command</font>** Entrypoint array，运行的程序
    
    - **<font color="#ffc104" size=3>args</font>** 向Entrypoint传递参数，官方对command和args的对比说明[Define a Command and Arguments for a Container - Kubernetes](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
  * **<font color="#0215cd" size=3>nodeSelector</font>** map[string]string 节点标签选择器，确定Pod只运行在哪个或哪类节点上。
  * **<font color="#0215cd" size=3>livenessProbe</font>** [object] 存活性验证
    * **<font color="#ffc104" size=3>exec</font>** 执行容器中存在的用户自定义命令。
        * command []string 运行命令来探测是否执行成功。 
    - **<font color="#ffc104" size=3>httpGet</font>**
    - **<font color="#ffc104" size=3>tcpSocket</font>**
    - **<font color="#ffc104" size=3>failureThreshold</font>** 确定失败的探测的失败次数，默认值3，最小值1。
    - **<font color="#ffc104" size=3>periodSeconds</font>** 周期间隔时长。默认10秒。
    - **<font color="#ffc104" size=3>timeoutSeconds</font>** 超时时长，默认1秒。
    - **<font color="#ffc104" size=3>initialDelaySeconds</font>** [integer] 初始化延迟探测时间，默认容器启动时立刻探测。

  * **<font color="#0215cd" size=3>readinessProbe</font>** 就绪性探测


* **<font color="#f8070d" size=3>status</font>**: 资源的当前状态 <font color="#f8070d" size=2>current state</font>。Kubernetes用于确保每一个资源定义完后，让其当前状态无限向目标状态转移，从而满足用户期望。从此角度来看，status是只读的，有Kubernets集群自行维护。

Kubernetes内嵌格式说明

### 获取pod资源的配置清单帮助

> **语法格式**

```sh
kubectl explain pod.lev1.lev2...
```

> **示例**

```sh
$ kubectl explain pod.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec {Object}

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds	{integer}
     Optional duration in seconds the pod may be active on the node relative to
     StartTime before the system will actively try to mark it failed and kill
     associated containers. Value must be a positive integer.

   affinity	{Object}
     If specified, the pod's scheduling constraints

   automountServiceAccountToken	{boolean}
     AutomountServiceAccountToken indicates whether a service account token
     should be automatically mounted.

   containers	{[]Object} -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.
     
   hostname	{string}
     Specifies the hostname of the Pod If not specified, the pod's hostname will
     be set to a system-defined value.

   imagePullSecrets	{[]Object}
    
    ....
    ....
    ....    

   volumes	{[]Object}
     List of volumes that can be mounted by containers belonging to the pod.
     More info: https://kubernetes.io/docs/concepts/storage/volumes

```


## 资源清单定义

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: default
  labels:
    app: myapp-redis
    tier: frontend
spec:
  containers:
  - name: redis-app
    image: redis
    imagePullPolicy: IfNotPresent
    ports:
    - name: redis
      containerPort: 6379
  - name: busybox
    image: busybox
    command:
    - "/bin/sh"
    - "-c"
    - "sleep 3600"
```

> **从yaml文件加载创建资源**

```sh
kubectl create -f pod.yaml 
```

> **从yaml文件加载删除资源**

```sh
kubectl delete -f pod.yaml 
```


## 标签选择器的使用

Label是Kubernetes中极具特色的功能之一，是附加在对象之上的<font color="#f8070d" size=2>键值对</font>，每一个资源可存在多个标签，每一个标签都是一组键值对。每一个标签都可以被标签选择器进行匹配度检查，从而完成资源挑选。Label既可以在对象创建时指定，可以在资源创建啊之后使用命令来管理（添加、修改、删除）。

Label可基于简单且直接的标准将Pod多个较小的分组。而service也需要识别标签对其识别并管控，或关联到的资源。最资源设定标签后，还可以使用标签来查看、删除等对其执行相应管理操作。

***
**<font color="#0215cd" size=2><font color="#f8070d" size=2>⚠</font> 注意：在定义标签时，资源标签其标签名称key及value的值必须小于等于63个字符，value，可为空。填写时只能使用<font color="#f8070d" size=2>`字母`</font>、<font color="#f8070d" size=2>数字</font>、<font color="#f8070d" size=3>`_`</font>、<font color="#f8070d" size=3>`-`</font>、<font color="#f8070d" size=3>`.`</font>，只能以字母或数字开头及结尾。
</font>**

***

在定义键名时也可以使用键前缀(DNS域名)，加前缀总长度不能超过253个字符。

### 对标签进行过滤

`kubectl get pods`常用参数说明

| 选项                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| -l                   | 大S                                                          |
| -L，label-columns=[] | 接受以逗号分隔的标签列表，这些标签将作为列显示。名字区分大小写。 |
| --show-labels        | 在最后一列打印标签。                                         |

> **--show-labels 显示Pod标签**

```sh
$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-7cdbd8cdc9-8blzc   1/1     Running   0          39s   pod-template-hash=7cdbd8cdc9,run=nginx
test-pod                 2/2     Running   0          17h   app=myapp-redis,tier=frontend
```


> **-L 获取显示指定类别的资源对象时，对每个资源对象显示其标签值。**

```sh
$ kubectl get pods --show-labels -L=run,app
NAME                     READY   STATUS    RESTARTS   AGE   RUN     APP           LABELS
nginx-7cdbd8cdc9-8blzc   1/1     Running   0          11m   nginx                 pod-template-hash=7cdbd8cdc9,run=nginx
test-pod                 2/2     Running   0          17h           myapp-redis   app=myapp-redis,tier=frontend
```

> **-l 标签过滤**

```sh
$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-7cdbd8cdc9-8blzc   1/1     Running   0          14m   pod-template-hash=7cdbd8cdc9,run=nginx
test-pod                 2/2     Running   0          17h   app=myapp-redis,tier=frontend

$ kubectl get pods --show-labels -l app
NAME       READY   STATUS    RESTARTS   AGE   LABELS
test-pod   2/2     Running   0          17h   app=myapp-redis,tier=frontend
```

### 资源对象打标签

> **kubectl label语法**

```sh
kubectl label -f filename|typename key1=value1 ... keyn valuen
```

对已有Pod添加标签

```sh
$ kubectl label pod nginx-7cdbd8cdc9-8blzc app=nginx-demo
pod/nginx-7cdbd8cdc9-8blzc labeled
```


修改已有Label值得Pod，需要使用 <font color="#f8070d" size=3>`--overwrite`</font>

```sh
$ kubectl label pod nginx-7cdbd8cdc9-8blzc app=nginx-demo
error: 'app' already has a value (nginx-demo), and --overwrite is false

$ kubectl label pod nginx-7cdbd8cdc9-8blzc app=nginx-demo1 --overwrite
pod/nginx-7cdbd8cdc9-8blzc labeled
```

### 使用复杂格式标签选择器

标签选择器

Kubernetes支持的标签选择器有两类，第一类是<font color="#f8070d" size=2>基于等值关系</font>的标签选择器，另一类是<font color="#f8070d" size=2>基于集合关系</font>的标签选择器。

> **基于等值关系的选择器**

基于等值关系的标签选择器的操作费无非就是<font color="#f8070d" size=2>等值关系</font>判断的的符号，如：<font color="#f8070d" size=3>`=`</font>  <font color="#f807	0d" size=3>`==`</font>  <font color="#f8070d" size=3>`!=`</font>


```sh
$ kubectl get pods --show-labels -l app=nginx-demo1
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-7cdbd8cdc9-8blzc   1/1     Running   0          6h    app=nginx-demo1,pod-template-hash=7cdbd8cdc9,run=nginx

$ kubectl get pods --show-labels -l app,tier
NAME       READY   STATUS    RESTARTS   AGE   LABELS
test-pod   2/2     Running   5          23h   app=myapp-redis,tier=frontend

$ kubectl get pods --show-labels -l app=myapp-redis,tier=frontend
NAME       READY   STATUS    RESTARTS   AGE   LABELS
test-pod   2/2     Running   5          23h   app=myapp-redis,tier=frontend
```

> **基于集合关系的标签选择器。于集合关系的标签选择器。**

基于集合关系就是如下几种类型来进行判断

* key in(value1,value2..,valueN)
* key notin(value1,value2,..valueN) 不具有此键也表示符合条件。
* key
* !key 不存在此键的资源


```sh
$ kubectl get pods --show-labels -l "app in (nginx-demo1,redus)"
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
nginx-7cdbd8cdc9-8blzc   1/1     Running   0          6h15m   app=nginx-demo1,pod-template-hash=7cdbd8cdc9,run=nginx
$ kubectl get pods --show-labels -l "app notin (nginx-demo1,redus)"
NAME       READY   STATUS    RESTARTS   AGE   LABELS
test-pod   2/2     Running   6          23h   app=myapp-redis,tier=frontend

```

可以使用标签的不止是Pod，各种对象都可以打标签，包括Node。当节点有标签后，在添加资源时，就可以让资源对节点有倾向性。

```sh
$ kubectl get nodes --show-labels
NAME              STATUS   ROLES    AGE    VERSION   LABELS
node02.k8s.test   Ready    [none]   3d1h   v1.13.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node02.k8s.test
node03.k8s.test   Ready    [none]   3d1h   v1.13.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node03.k8s.test
```

## Pod的生命周期

常见的Pod状态：

Pending 挂起，请求创建Pod时，发现条件不满足，调度尚未完成
running
Failed
Success
Unkown，未知状态。Pod的状态是Apiserver与运行Pod的节点上的Kubelet通信获取状态信息的。当Node节点上Kubelet进程发生故障。Apiserver无法获取Pod信息。


Pod的创建过程：

用户创建Pod时，将请求提交给Apiserver，Apiserver将创建请求的目标状态保存在etcd中，而后apiserver请求scheduler进行调度（负责挑选出合适的节点来运行Pod）。并将调度结果保存至etcd的Pod资源信息中。随后目标节点kubelet通过apiserver的状态变化，拿到用户所提交的创建清单。根据清单在当前节点上创建并运行Pod，并将当前结果状态发送给apiserver，由apiserver将此状态信息存至etcd中。


restartPolicy: 重启策略
Always 总是重启, OnFailure 只有其状态为错误时重启，正常终止时不重启, Never 从不重启. Default to Always.

容器的重启策略

Pod在被调度至某一节点之上时，只要此节点存在，Pod不会被重新调度，只会重启。除非Pod删除或Pod存在节点故障才会被重新调度。

Pod的终止过程

在kubenetes集群中，Pod代表在Kubernetes集群节点上运行的程序或进程，是向用户提供服务的主要单位。当在提交删除一个Pod时，不会直接kill删除的，而是向Pod内的每一个容器发送TEAM终止信号，使Pod中容器正常终止。终止默认有30秒宽限期。宽限期结束，依然无法终止，会重新发送kill信号，强行进行终止。


## kubernetes中的探测方式

所谓的容器探测无非就是，在容器中设置一些探针或传感器来获取相应的数据。作为其存活与否、就绪与否的标准。目前来讲Kubernetes所支持的存货性探测方式和就绪行探测方式都是一样的。

Kubernetes中的探针类型有三种<font color="#f8070d" size=3>`ExecAction`</font>，TCP套接字探针 <font color="#f8070d" size=3>`TCPSocketAction`</font> ，<font color="#f8070d" size=3>`HTTPGetAction`</font>，使用<font color="#f8070d" size=3>`kubectl explain pod.containers.xxx`</font>获取帮助信息。


### livenessProbe实例

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: liveness-pod
  namespace: default
spec:
  containers:
  - name: liveness-container
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","touch /tmp/healthy; sleep 20; rm -fr /tmp/healthy; sleep 3600"]
    livenessProbe:
      exec:
        command: ["test","-e","/tmp/healthy"]
      initialDelaySeconds: 3
      periodSeconds: 2
      successThreshold: 1
```
在创建后使用describe查看错误
```
$ kubectl describe pod liveness-pod
Name:               liveness-pod
Namespace:          default
Priority:           0
PriorityClassName:  [none]
Node:               node03.k8s.test/10.0.0.17
Start Time:         Thu, 10 Jan 2019 18:36:27 +0800
Labels:             [none]
Annotations:        [none]
Status:             Running
IP:                 10.244.1.9
Containers:
  liveness-container:
    Container ID:  docker://10062f9026968e664fbb128ddb33a14e30efc848e914fe12d769bab9180ab21a
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:7964ad52e396a6e045c39b5a44438424ac52e12e4d5a25d94895f2058cb863a0
    Port:          [none]
    Host Port:     [none]
    Command:
      /bin/sh
      -c
      touch /tmp/healthy; sleep 20; rm -fr /tmp/healthy; sleep 3600
    State:          Running
      Started:      Thu, 10 Jan 2019 18:39:17 +0800
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Thu, 10 Jan 2019 18:38:22 +0800
      Finished:     Thu, 10 Jan 2019 18:39:17 +0800
    Ready:          True
    Restart Count:  3
    Liveness:       exec [test -e /tmp/healthy] delay=3s timeout=1s period=2s #success=1 #failure=3
    Environment:    [none]
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ml2gd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-ml2gd:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ml2gd
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  [none]
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
```
可以看到根据定义的检测规则，Pod在不停的重启。
```
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
liveness-pod             1/1     Running   8          17m
```

### readinessProbe，就绪性探测实例

> **编写yaml文件，使用HTTPAction探针进行就绪性探测**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-pod
  namespace: default
spec:
  containers:
  - name: http-container
    image: httpd
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    readinessProbe:
      httpGet:
        port: http
        path: /index.html
      initialDelaySeconds: 2
      periodSeconds: 3
```

> **使用资源配置清单创建Pod，并查看其状态**

```
$ kubectl create -f http.yaml 
pod/http-pod created


$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
http-pod                 1/1     Running   0          6s
```

查看此Pod的就绪性。

```
$ kubectl describe pod http-pod
Name:               http-pod
Namespace:          default
Priority:           0
PriorityClassName:  [none]
Node:               node02.k8s.test/10.0.0.16
Start Time:         Fri, 11 Jan 2019 11:34:36 +0800
Labels:             [none]
Annotations:        [none]
Status:             Running
IP:                 10.244.0.18
Containers:
  http-container:
    Container ID:   docker://562525c9498153ed0285d6fdaa03b822efca470194341f12e5f510ae9e93f570
    Image:          httpd
    Image ID:       docker-pullable://httpd@sha256:a613d8f1dbb35b18cdf5a756d2ea0e621aee1c25a6321b4a05e6414fdd3c1ac1
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 11 Jan 2019 11:34:38 +0800
    Ready:          True
    Restart Count:  0
    Readiness:      http-get http://:http/index.html delay=2s timeout=1s period=3s #success=1 #failure=3
    Environment:    [none]
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ml2gd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-ml2gd:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ml2gd
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  [none]
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                      Message
  ----    ------     ----  ----                      -------
  Normal  Scheduled  11s   default-scheduler         Successfully assigned default/http-pod to node02.k8s.test
  Normal  Pulled     9s    kubelet, node02.k8s.test  Container image "httpd" already present on machine
  Normal  Created    9s    kubelet, node02.k8s.test  Created container
  Normal  Started    9s    kubelet, node02.k8s.test  Started container
```

手动接入Pod内，将探测的文件删除。此时查看Pod的就绪性如下。提示404

```sh
kubectl exec -it http-pod -- /bin/bash
```

此时，Pod就绪的容器量为0个，也就是说，容器中httpd进程正常，但是web页面不存在。

```
$ kubectl get pods 
NAME                     READY   STATUS    RESTARTS   AGE
http-pod                 0/1     Running   0          3h39m
```

```
$ kubectl describe pod http-pod
Name:               http-pod
Namespace:          default
Priority:           0
PriorityClassName:  [none]
Node:               node02.k8s.test/10.0.0.16
Start Time:         Fri, 11 Jan 2019 11:34:36 +0800
Labels:             [none]
Annotations:        [none]
Status:             Running
IP:                 10.244.0.18
Containers:
  http-container:
    Container ID:   docker://562525c9498153ed0285d6fdaa03b822efca470194341f12e5f510ae9e93f570
    Image:          httpd
    Image ID:       docker-pullable://httpd@sha256:a613d8f1dbb35b18cdf5a756d2ea0e621aee1c25a6321b4a05e6414fdd3c1ac1
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 11 Jan 2019 11:34:38 +0800
    Ready:          False
    Restart Count:  0
    Readiness:      http-get http://:http/index.html delay=2s timeout=1s period=3s #success=1 #failure=3
    Environment:    [none]
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ml2gd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-ml2gd:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ml2gd
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  [none]
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From                      Message
  ----     ------     ----               ----                      -------
  Normal   Scheduled  3h39m              default-scheduler         Successfully assigned default/http-pod to node02.k8s.test
  Normal   Pulled     3h39m              kubelet, node02.k8s.test  Container image "httpd" already present on machine
  Normal   Created    3h39m              kubelet, node02.k8s.test  Created container
  Normal   Started    3h39m              kubelet, node02.k8s.test  Started container
  Warning  Unhealthy  0s (x10 over 27s)  kubelet, node02.k8s.test  Readiness probe failed: HTTP probe failed with statuscode: 404
```

lifecycle 生命周期，用来定义启动后和终止前钩子

lifecycle

postStart Pod在创建启动之后立即执行的操作。如果执行失败，容器会终止，被重启，重启与否取决于重启策略。

preSTop Pod在终止之前立即被执行的命令。命令执行完毕后，Pod才会被终止。

***
**<font color="#0215cd" size=2> <font color="#f8070d" size=2>⚠</font> 注意：注意两个command的执行顺序<font color="#f8070d" size=3>`containers.command`</font>是定义容器的command，<font color="#f8070d" size=3>`containers.postStart.exec.command`</font>是定义容器启动后初始化操作
</font>**
***

许多资源支持内嵌字段定义其使用的标签选择器： 
matchLabels：直接给定key value service只支持此类
matchExpressions：基于给定的表达式定义使用的标签选择器`{key:"KEY",operator:"OPERATOR",values:[VAL1,VAL2,...]`。

操作符：
In，Notin：values必须为非空列表
Exists，NotExists：values必须为空列表。




Pod控制器都是内嵌Pod模板，

Pod控制器去管理Pod中间层，并确保每一个Pod资源始终处于定义、或所期望的目标状态。当Pod状态出现故障，首先尝试重启容器，


Pod控制器有多种类型

* ReplicaSet：ReplicaSet被称为新一代的ReplicationController，它的核心作用在于代用户创建指定数量的副本，并确保Pod副本一直处于满足用户期望数量的状态，多退少补。还支持自动扩缩容机制。

主要有三个组件组成

用户期望Pod副本数量
标签选择器，以便选定由自己管理控制的Pod副本。
Pod资源模板  通过标签选择器选到的标签副本数量低于指定数量，会使用Pod资源模板完成Pod资源的新建。


帮助用户管理无状态的Pod资源，并确保精确反应用户所定义的目标数量，但是Kubernetes不建议直接使用ReplicaSet



Deployment

Deployment工作于ReplicaSet之上，一个Deployment可以管理多个rs，但是存活的（），通常保留历史版本中的10个。Deployment通过控制ReplicaSet来控制Pod。Deployment除了支持ReplicaSet所支持的功能，还支持滚动更新、回滚等机制，而且还提供了声明式配置的功能。是用来管理无状态应用的目前最佳的Pod控制器。

- Deployment能提供滚动式自定义、自控制的更新。
- Deployment在更新时可控制更新节奏和更新逻辑。

声明式配置在创建资源时，可以基于声明逻辑来定义，所有更新的资源可以随时重新进行声明，只要资源支持动态运行时修改，就可以随时改变在apiserver上定义的目标期望状态。

- Pod副本数量是有可能大于节点数的，并且数量本身彼此间没有任何精确对应关系。


DaemonSet 

用于确保集群的每一个节点或指定条件的节点上只运行一个特定的Pod副本，通常用于实现系统级的后台任务。

Job 只能执行一次性的作业
CronJob 周期性运行作业，每次运行都有正常退出的时间 不需要持续后台运行。如果前一次任务没有完成，下一次时间点又到了，CronJob还需要处理此类问题

区别

Job和CronJob与DaemonSet和Deployment显著区别就在于，Job和CronJob不需要持续后台运行。

Deployment只能用于管控无状态应用。常用于只关注群体，而不必关注个体的场景。

StatefulSet能够实现管理有状态应用，而且每一个应用，每一个Pod副本都是被单独管理的。他拥有自己的独有标识和独有的数据集，一旦节点发生故障，在加进来之前需要进行初始化操作。

StatefulSet提供了封装控制器，将需要人为手动做的操作、复杂的执行逻辑，定义成脚本，放置在StatefulSet Pod模板的定义当中。每次节点故障，通过脚本可自动恢复状态。



ReplicaSet的使用

ReplicaSet定义方式可以使用<font color="#f8070d" size=3>`kubectl explain rs`</font>查看帮助，ReplicaSet使用的apps组中的v1，而不在是core组。内嵌字段与Pod类似，而spec中定义时最核心的只有3个 relicas副本数量、selector 标签选择器 templates Pod模板，对于templates而言，其内部就是Pod模板。

使用ReplicaSet创建Pod



labels中的标签，必须符合selector中的选择标准。否则创建的Pod是无用的，创建Pod都不够relicas定义的数量，它将会永久创建下去。

在Pod模板中起的Pod名称是没有用的，它会自动以控制器的名称后跟一串随机串，来作为Pod名称来创建。


> **当Pod控制器数量超出用户期望数量，会随机删除其中一个Pod**



```
$ kubectl get pods --show-labels
NAME                     READY   STATUS        RESTARTS   AGE    LABELS
rs-5xw7l                 1/1     Terminating   0          40m    app=myapp,release=canary,test=1a
rs-6zx6b                 1/1     Running       0          40m    app=myapp,release=canary,test=1a
test-pod                 2/2     Running       0          46m    app=myapp,release=canary

$ kubectl label pods test-pod release=canary
pod/test-pod labeled

$ kubectl get pods --show-labels
NAME                     READY   STATUS        RESTARTS   AGE    LABELS
rs-5xw7l                 1/1     Terminating   0          40m    app=myapp,release=canary,test=1a
rs-6zx6b                 1/1     Running       0          40m    app=myapp,release=canary,test=1a
test-pod                 2/2     Running       0          46m    app=myapp,release=canary

$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE    LABELS
rs-6zx6b                 1/1     Running   0          42m    app=myapp,release=canary,test=1a
test-pod                 2/2     Running   0          47m    app=myapp,release=canary
```


ReplicaSet动态规模的扩容、缩容


修改了控制器后，Pod资源并不会随之更改，因为Pod资源足额就不会被重建，只有重建的Pod资源的版本才是新版本的。





## Pod Preset

`Pod Preset` 是一种 API 资源，在 Pod 创建时，用户可以用它将额外的将运行时需求信息注入 Pod内。 使用标签选择算符来指定 Pod Preset 所适用的 Pod。

### 在集群中启动Pod Preset

在集群中使用 Pod Preset，必须确保以下几点：

- 需要确保你使用的是`kubernetes 1.8`版本以上
- 已启用 API 类型 `settings.k8s.io/v1alpha1/podpreset`
- 已启用准入控制器 `PodPreset`

apiserver添加参数 `--enable-admission-plugins`  `--runtime-config `

```bash
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodPreset \
--runtime-config=settings.k8s.io/v1alpha1=true
```

创建一个PodPreset

```yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: time-preset
  namespace: default
spec:
  selector:
    matchLabels:
  volumeMounts:
    - mountPath: /etc/localtime
      name: time
  volumes:
    - name: time
      hostPath:
        path: /etc/localtime
  env:
    - name: ENVOY_END
      value: envoy-1.15
```

### 特定Pod禁用Pod Preset

在 Pod 的 `.spec` 中添加形如 `podpreset.admission.kubernetes.io/exclude: "true"` 的注解



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podpreset-deply
  labels:
    app: podpreset-deply

spec:
  replicas: 1
  selector:
    matchLabels:
      app: podpreset-deply
  template:
    metadata:
      name: podpreset-deply
      labels:
        app: podpreset-deply
      annotations:
        podpreset.admission.kubernetes.io/exclude: "true"
    spec:
      containers:
        - name: envoy-end
          image: sealloong/envoy-end
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
```

> Reference
>
> [openshift](https://access.redhat.com/documentation/en-us/openshift_container_platform/3.7/html/developer_guide/dev-guide-pod-presets#sample-pod-spec-exclude-preset)
>
> [kubernetes-pod preset](https://v1-18.docs.kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/)


