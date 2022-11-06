# 

* workload 运行应用程序，对外提供服务：Pod、ReplicaSet、Deployment、StatefulSet、DaemonSet、Job、Cronjob,...
* 服务发现及负载均衡：service、Ingress
* 配置与存储：Volume，CSI
    * ConfigiMap、Secret
    * DownwardAPI
* 集群级资源
    * Namespace、Node、Role、ClusterRole、RoleBinding, ClusterRoleBinding
* 元数据型资源
    * HPA、PodTemplate、LimitRange（资源限制）




在创建资源时，除了命令式创建方式，还可以使用&lt;font color=&#34;#f8070d&#34; size=2&gt;配置清单&lt;/font&gt;进行创建。配置清单由很多属性或字段所组成。

apiVersion: 说明对应的对象属于Kubernetes的哪一个API群组名称和版本。给定apiVersion时由两部分组成&lt;font color=&#34;#f8070d&#34; size=3&gt;`group/version`&lt;/font&gt;，group如果省略表示core（核心组）之意。使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl api-versions`&lt;/font&gt;获得当前系统所支持的apiserver版本。

* kind: 资源类别，用来指明哪种资源用来初始化成资源对象时使用。

* metadata: 元数据，内部嵌套很多2级、3级字段。主要提供以下几个字段。
    * name，在同一类别当中name必须是唯一的。
    * namespace 对应的对象属于哪个名称空间，name受限于namespace，不同的namespace中name可以重名。
    * lables key-value数据，对于key名称，最多为43个字符，value，可为空。填写时只能使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`字母`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;数字&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`_`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`-`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`.`&lt;/font&gt;，只能以字母或数字开头及结尾。
    * annotations 资源注解

* spec: specification，定义接下来创建的资源对象应该满足的规范（&lt;font color=&#34;#f8070d&#34; size=2&gt;期望的状态 disired state&lt;/font&gt;）。spec是用户定义的。不同的资源类型，其所需要嵌套的字段各不相同。如果某一字段属性标记为required表示为必选字段，剩余的都为可选字段，系统会赋予其默认值。如果某一字段标记为Cannot be updated，则表示为对象一旦创建后不能改变字段值。可使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl explain pods.spec`&lt;/font&gt;查看详情。
  * containers [required]object list
    * \- &lt;font color=&#34;#f8070d&#34; size=2&gt;name&lt;/font&gt;  [string] 定义容器名称
    * &lt;font color=&#34;#f8070d&#34; size=2&gt;image&lt;/font&gt; [string] 启动Pod内嵌容器时所使用的镜像。可是顶级、私有、第三方仓库镜像。
    * &lt;font color=&#34;#f8070d&#34; size=2&gt;imagePulLPolicy&lt;/font&gt; [string] 镜像获取的策略，可选参数Always（总是从仓库下载，无论本地有无此镜像）、Never（从不下载，无论本地有无此镜像）、IfNotPresent（本地存在则使用，不存在则从仓库拉去镜像）。如果tag设置为latest，默认值则为&lt;font color=&#34;#f8070d&#34; size=3&gt;`Always`&lt;/font&gt;，非latest标签，默认值都为&lt;font color=&#34;#f8070d&#34; size=3&gt;`IfNotPresent`&lt;/font&gt;。
    * ports []object 定义容器内要暴露的端口时，可以暴露多个端口，每个端口应该由多个属性来定义（端口名称、端口号、协议）。&lt;font color=&#34;#f8070d&#34; size=2&gt;注意暴露端口仅仅是提供额外信息的，并不能限制系统是否真能暴露&lt;/font&gt;。
    * command Entrypoint array，运行的程序
    * args 向Entrypoint传递参数，官方对command和args的对比说明[Define a Command and Arguments for a Container - Kubernetes](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
    * nodeSelector map[string]string 节点标签选择器，确定Pod只运行在哪个或哪类节点上

* status: 资源的当前状态 &lt;font color=&#34;#f8070d&#34; size=2&gt;current state&lt;/font&gt;。Kubernetes用于确保每一个资源定义完后，让其当前状态无限向目标状态转移，从而满足用户期望。从此角度来看，status是只读的，有Kubernets集群自行维护。
&lt;font color=&#34;#f8070d&#34; size=2&gt;&lt;/font&gt;

Kubernetes内嵌格式说明

kubectl explain

```yaml
$  kubectl get pod clients -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: &#34;2019-01-03T08:06:03Z&#34;
  labels:
    run: clients
  name: clients
  namespace: default
  resourceVersion: &#34;38704&#34;
  selfLink: /api/v1/namespaces/default/pods/clients
  uid: 6a59ae8e-0f2e-11e9-b848-000c2984f329
spec:
  containers:
  - image: busybox
    imagePullPolicy: Always
    name: clients
    resources: {}
    stdin: true
    stdinOnce: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    tty: true
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-rlkwc
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node03.k8s.test
  priority: 0
  restartPolicy: Never
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-rlkwc
    secret:
      defaultMode: 420
      secretName: default-token-rlkwc
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: &#34;2019-01-03T08:06:03Z&#34;
    status: &#34;True&#34;
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: &#34;2019-01-03T08:06:10Z&#34;
    status: &#34;True&#34;
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: &#34;2019-01-03T08:06:10Z&#34;
    status: &#34;True&#34;
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: &#34;2019-01-03T08:06:03Z&#34;
    status: &#34;True&#34;
    type: PodScheduled
  containerStatuses:
  - containerID: docker://8346b505f471c3bbc8a008216f3c63dc1e2a13a8b1237ef94252436cda37a652
    image: busybox:latest
    imageID: docker-pullable://busybox@sha256:7964ad52e396a6e045c39b5a44438424ac52e12e4d5a25d94895f2058cb863a0
    lastState: {}
    name: clients
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: &#34;2019-01-03T08:06:09Z&#34;
  hostIP: 10.0.0.17
  phase: Running
  podIP: 10.244.1.6
  qosClass: BestEffort
  startTime: &#34;2019-01-03T08:06:03Z&#34;
```

在创建资源时，apiserver仅接收JSON格式的资源定义。在使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl run`&lt;/font&gt;命令时，自动将给定内容转换成JSON格式。

yaml格式提供配置清单，apiserver可自动将其转为JSON格式，（yaml可无损转为json），而后再提交。

大部分资源的配置清单都由五个一级字段组成。


配置清单可带来复用效果



  

alpha 内测版
beta 公测版
stable 稳定版




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
    - &#34;/bin/sh&#34;
    - &#34;-c&#34;
    - &#34;sleep 3600&#34;

```


-l 

-L 获取显示指定类别的资源对象时，对每个资源对象显示其标签值。


使用复杂格式标签选择器，

标签选择器

Kubernetes的标签选择器支持两类
，支持等值关系的标签选择器 =,==,!= 等值关系判断
，集合关系的标签选择器：
* key in(value1..valueN)
* key notin(value1..valueN) 
* key
* !key


```sh
$  kubectl get pods -l app!=redus --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
test-pod   2/2     Running   2          2d23h   app=myapp-redis,tier=frontend

$  kubectl get pods -l app!=redis --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
test-pod   2/2     Running   2          2d23h   app=myapp-redis,tier=frontend

$  kubectl get pods -l app=redis --show-labels
No resources found.


$  kubectl get pods -l &#34;app in (test,aaa,myapp-redis)&#34; --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
test-pod   2/2     Running   2          2d23h   app=myapp-redis,tier=frontend

$  kubectl get pods -l &#34;app notin (test,aaa,bbb)&#34; --show-labels
NAME       READY   STATUS    RESTARTS   AGE     LABELS
test-pod   2/2     Running   2          2d23h   app=myapp-redis,tier=frontend

```

许多资源支持内嵌字段定义其使用的标签选择器： 
matchLabels：直接给定key value service只支持此类
matchExpressions：基于给定的表达式定义使用的标签选择器`{key:&#34;KEY&#34;,operator:&#34;OPERATOR&#34;,values:[VAL1,VAL2,...]`。

操作符：
In，Notin：values必须为非空列表
Exists，NotExists：values必须为空列表。

使用标签的不止是Pod，各种对象都可以打标签，包括Node。当节点有标签后，在添加资源时，就可以让资源对节点有倾向性。
```sh
$ kubectl get nodes --show-labels
NAME              STATUS   ROLES    AGE    VERSION   LABELS
node02.k8s.test   Ready    [none]   3d1h   v1.13.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node02.k8s.test
node03.k8s.test   Ready    [none]   3d1h   v1.13.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=node03.k8s.test
```

annotations：
与label不同的地方在于，它不能用于挑选资源对象，仅用于为对象提供“元数据”。对键值长度没有要求。在构建大型镜像时通常会用其标记对应的资源对象的元数据
