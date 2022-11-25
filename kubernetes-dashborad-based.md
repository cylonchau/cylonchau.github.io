# 转：基于Kubernetes的PaaS平台提供dashboard支持的一种方案


**本文转自博客**： [我的小米粥分你一半](https://corvo.myseu.cn/2020/12/05/2020-12-05-%E5%9F%BA%E4%BA%8EKubernetes%E7%9A%84PaaS%E5%B9%B3%E5%8F%B0%E6%8F%90%E4%BE%9Bdashboard%E6%94%AF%E6%8C%81%E7%9A%84%E4%B8%80%E7%A7%8D%E6%96%B9%E6%A1%88/)

> 我一直在负责维护的PaaS平台引入了Kubernetes作为底层支持, 可以借助Kubernetes的生态做更多的事情, 这篇博客主要介绍如何为用户提供dashboard功能, 以及一些可以扩展的想法. 希望读者有一定的kubernetes使用经验, 并且了解rbac的功能。

## Dashboard功能

Kubernetes原生提供了Web界面, 也就是Dashboard, 具体的参考可以见[官方文档](https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/):

![image-20220516174111629](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220516174111629.png)

​	安装完成后, 我们一般是通过token来使用的, 不同的token有着不同的权限.

​	![image-20220516174123726](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220516174123726.png)

上面所说的`token`是`Bearer Token`, 除了在界面上输入之外, 你可以这么来用, 通过添加header即可.

```
curl -H "Authorization: Bearer ${TOKEN}" https://{dashboard}/api/myresource
```

## PaaS平台使用Dashboard简要讨论

### 需求分析

Dashboard本身的功能是十分强大的, 但是给所有人admin权限显然是不现实的. 对于一个普通用户来讲, PaaS平台的将他的应用(代码)部署好并运行, 他所需要关注的就只有属于他自己的项目, 平台也需要做好权限控制, 避免一个用户操作了另一个用户的应用.

### 权限系统设计

基于以上的需求讨论, 平台需要做的操作就是为每个用户创建属于自己的权限提供, 并限制可以访问到的资源. 考虑这样的情况:

我们有一个用户A, 他拥有自己的一个应用群组(G), 群组中部署了一系列应用程序(a1, a2…). 在Kubernetes中, 这样的群组概念我们将其映射为namespace, `群组(G) <=> 用户空间(NS)`, 我们需要控制的权限控制策略就变成了用户A在用户空间NS的权限控制.

### token分发策略

拥有了权限控制后, 所需要打就是将token分发给用户, 当然这是一种极度不安全的做法, Kubernetes中的token创建之后一般是不会改变的, 分发这样的token会有很大的安全风险, 有两个方面:



	1. 用户A将token保存了下来, 那么他就能不经过平台登录Dashboard, 这样不利于审计工作,
	2. token一旦泄露, PaaS平台很难做到反应(因为token脱离了平台的控制, 无法判断究竟是什么时候发生了泄露, 也无法马上吊销这个token), 安全风险比较高.

因此, 最好的做法就是不把token交给用户, 用户每次想要登录dashboard, 从平台进行跳转, 跳转时携带安全信息, 在dashboard登录时, 由平台自己的程序请求token, 避免经手用户.

------

> 如果到这里, 你没有理解上面的内容, 建议回去再看一次需求, 如果还是理解不了, 就不要往下看了, 下面只是介绍具体的实现方案.

------

## Kubernetes权限限制

Kubernetes本身有着比较复杂的权限控制系统, 设计时没必要纠结过多, 按照可以给用户和不能给用户的权限进行区分就好了. 我直接贴一下我的权限控制策略吧, 并不一定适合每个人, 只是可以做个参考.

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: xxx:xxx-group-yyy
  namespace: xxx-group-yyy
rules:
  # 可以查看当前NS下面的service, pod, logs, events
  - apiGroups: [""]
    resources: ["services", "pods", "pods/log", "events"]
    verbs: ["get", "list", "watch"]

  # 可以使用exec命令进入容器
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create", "list", "watch"]

  # 可以查看deployments和replicasets
  - apiGroups: ["extensions", "apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch"]

  # 可以查看job, cronjob以及ingress
  - apiGroups: ["batch", "extensions"]
    resources: ["cronjobs", "jobs", "ingresses"]
    verbs: ["get", "list", "watch"]
```

正如上面的注释一样, 尽可能只给用户只读的权限, 也许你已经发现了, 甚至不需要给用户namespace的查看权限, 这也是为了安全, 避免用户得知其他人的namespace.

## 分发Token以及安全性保证

这是本篇博客的核心内容: 如何使得用户可以无感知的登录到dashboard(对用户隐藏token).

该方案用到的方法是: 添加一层访问控制的网关, 用于处理token获取的操作, 具体的流程图如下.

![image-20220516174135751](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220516174135751.png)

需要注意的有几点:

1. PaaS给出的`secret_code`是有时效性的, 不允许用户一直用同一个`secret_code`进行访问
2. 网关与PaaS平台间的通信应该加密, 网关必须是PaaS平台可信的
3. 网关不应该长期保存`token`
4. 网关的访问最好添加OpenID校验, 确保网关可以精确定位到每个用户的每次访问

### 体验优化

1. 首先, 第2步到第3步, `secret_code`获取之后, 可以以302重定向的方式跳转至网关入口
2. 网关可以临时性的保存`secret_code`与`token`的映射关系, 既能够提升用户体验, 也能有效减缓PaaS平台的压力
3. dashbaord的webshell功能是基于websocket支持的, 所以请确保你的网关可以通过websocket请求, 否则终端连接后几分钟就断了, websocket可以持续几个小时那么久
4. 跳转到网关时, 可以携带更多的信息, 比如携带某个pod的id, 网关就可以直接跳转到对应的pod, 用户打开webshell就很方便了

网关的实现我不做过多的说明了, 只有一点建议, `secret_code`在跳转到网关后, 马上进行校验. 由于`dashboard`的前端路由实现问题, `secret_code`最好在校验后加密放置到cookie中, 实现方面的问题其他可以发邮件与我讨论.

## 总结

这篇博客主要介绍了一种允许普通用户使用dashboard的功能. 在实现策略上, 利用了kubernetes的权限限制, token隐藏的方案, 该方案目前我已经加入到了我负责的PaaS平台中, 稳定性方面可以满足工作需求, 安全性正如博客中介绍, 大家可以自行斟酌.


