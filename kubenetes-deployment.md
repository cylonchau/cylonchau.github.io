# Kubenetes Deployment


Deployment当中借助于ReplicaSet进行更新的策略反映在Deployment的对象定义所需字段可使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl explain deploy`&lt;/font&gt;，Deployment属于extension群组。在1.10版本中它被移至到apps群组。他与ReplicaSet相比增加了几个字段。

stratgy 重要字段，定义更新策略，它支持两种策略 重建式更新 &lt;font color=&#34;#f8070d&#34; size=3&gt;`Recreate`&lt;/font&gt;与滚动更新&lt;font color=&#34;#f8070d&#34; size=3&gt;`RollingUpdate`&lt;/font&gt;，如果type为RollingUpdate，那么RollingUpdate的策略还可以使用RollingUpdate来定义，如果type为Recreate，那么RollingUpdate字段无效。 默认值为&lt;font color=&#34;#f8070d&#34; size=3&gt;`RollingUpdate`&lt;/font&gt;

stratgy.RollingUpdate控制RollingUpdate更新力度
 - maxSurge 对应的更新过程当中，最多能超出目标副本数几个。有两种取值方式，为直接指定数量和百分比。在使用百分比时，在计算数据时如果不足1会补位1个。
 - maxUnavailable 最多有几个副本不可用。

revisionHistoryLimit 滚动更新后，在历史当中最多保留几个历史版本，默认10。




&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;在使用Deployment创建Pod时，Deployment会自动创建ReplicaSet，而且Deployment名称是使用Pod模板的hash值，此值是固定的。

&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;Deployment在实现更新应用时，可以通过编辑配置文件来实现，使用kubectl apply -f更改每次变化。每次的变化通过吧变化同步至apiserver中，apiserver发现其状态与etcd不同，从而改变etcd值来实现修改其期望状态，来实现现有状态去逼近期望状态。

kubectl explain deploy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deploy
      release: canary
  template:
    metadata:
      labels:
        app: deploy
        release: canary
    spec:
      containers:
      - name: my-deploy
        image: node01:5000/busybox:v1
        ports:
        - name: http
          containerPort: 80
        command: [&#34;/bin/sh&#34;,&#34;-c&#34;,&#34;/bin/httpd -f -h /tmp&#34;]
```

&gt; **使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl apply`&lt;/font&gt; 声明式更新、创建资源对象。**

将上述资源配置清单的replicaSet数量改为3个后，可以看到数量增加为3，而对应的hash值没变化。 

```sh
$ kubectl apply -f deploy.yaml 
deployment.apps/app-deploy configured
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
app-deploy-5b8db6bc7d-bkldv   1/1     Running   0          4s
app-deploy-5b8db6bc7d-r2pcv   1/1     Running   0          5h5m
app-deploy-5b8db6bc7d-wgbbg   1/1     Running   0          5h5m
```

&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;由于Deployment是构建在ReplicaSet之上，对Pod做扩展、缩容是很方便的。处理动态修改资源配置清单外，还可以使用&lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl patch`&lt;/font&gt;（打补丁）进行操作。

&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;patch操作是对对象的JSON内容进行打补丁，-p选项值为JSON格式，其建值需以引号引起。

&gt; **语法**

```
kubectl patch
```

-p 提供补丁

```
$ kubectl patch deployment app-deploy -p &#39;{&#34;spec&#34;:{&#34;replicas&#34;:4}}&#39;
deployment.extensions/app-deploy patched
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
app-deploy-58d74fd87f-555jm   1/1     Running   0          15m
app-deploy-58d74fd87f-kkwz9   1/1     Running   0          15m
app-deploy-58d74fd87f-vzthx   1/1     Running   0          15m
app-deploy-58d74fd87f-zwdn5   1/1     Running   0          4s
```

```
kubectl patch deployment app-deploy -p &#39;{&#34;spec&#34;: {&#34;strategy&#34;: {&#34;rollingUpdate&#34;: {&#34;maxSurge&#34;:1,&#34;maxUnavailable&#34;:0 }}}}&#39;
```

&gt; **更新版本**

&amp;nbsp;&amp;nbsp;&amp;nbsp;&amp;nbsp;在更新完成后使用 kubectl get rs 查看可看到有两个 rs版本，所不同的是，镜像版本不同和可用的数量为0，但这个对应的模板会保留，随时等待回滚。

```
$ kubectl get rs -o wide
NAME                    DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                   SELECTOR
app-deploy-58d74fd87f   3         3         3       76s     my-deploy    node01:5000/busybox:v2   app=deploy,pod-template-hash=58d74fd87f,release=canary
app-deploy-5b8db6bc7d   0         0         0       5h10m   my-deploy    node01:5000/busybox:v1   app=deploy,pod-template-hash=5b8db6bc7d,release=canary
```


更新版本可以使用可使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl set image`&lt;/font&gt;进行更新

语法

```
kubectl set image [-f filename | type name] container=version
```

```
kubectl set image deployment app-deploy my-deploy=node01:5000/busybox:v4 &amp;&amp; kubectl rollout pause deployment app-deploy
```



此时可以看到rs保留多个版本

```
$ kubectl get rs -o wide
NAME                    DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES                   SELECTOR
app-deploy-58d74fd87f   0         0         0       34m    my-deploy    node01:5000/busybox:v2   app=deploy,pod-template-hash=58d74fd87f,release=canary
app-deploy-7fbc8b6df    3         3         3       25m    my-deploy    node01:5000/busybox:v4   app=deploy,pod-template-hash=7fbc8b6df,release=canary
```

使用 kubectl rollout pause可以暂停更新。

```
kubectl rollout pause type typename
```

使用resume可恢复暂停操作

```
kubectl rollout resume type typename
```

### Deployment版本滚动的历史保留

可使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl rollout history`&lt;/font&gt; 查看滚动历史

```
$ kubectl rollout history deployment app-deploy
deployment.extensions/app-deploy 
REVISION  CHANGE-CAUSE
1         [none]
2         [none]
```


#### 版本回滚

使用 &lt;font color=&#34;#f8070d&#34; size=3&gt;`kubectl rollout undo`&lt;/font&gt;默认是回滚至上一个版本


&gt; **语法**

```
kubectl rollout undo type typename 
```

&lt;font color=&#34;#f8070d&#34; size=3&gt;`--to-revision=n`&lt;/font&gt; 回滚至指定版本

```
kubectl rollout undo deployment app-deploy --to-revision=1
```

回滚后查询当前rs版本

```
$ kubectl get rs -o wide
NAME                    DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                   SELECTOR
app-deploy-58d74fd87f   3         3         3       108m    my-deploy    node01:5000/busybox:v2   app=deploy,pod-template-hash=58d74fd87f,release=canary
app-deploy-7fbc8b6df    0         0         0       99m     my-deploy    node01:5000/busybox:v4   app=deploy,pod-template-hash=7fbc8b6df,release=canary
```

Deployment回滚演示

![b1fc4e91.gif](https://note.youdao.com/yws/public/resource/db979da305f5f19c7dbef51e4b9f32bf/xmlnote/WEBRESOURCE5695ba7b8d4db604cc2ac74a5a8f8eb6/732)

