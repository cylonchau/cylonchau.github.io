# Ingress


IngressController比较独特，它与DaemonSet、Deployment、Repliacaset不同，DaemonSet、Deployment等控制器是作为ControllerManager的子组件存在的。Ingress Controller是独立运行的一组Pod资源，通常是拥有七层代理、调度能力的应用程序。

通常在使用IngressController时有三种选择Nginx、Traefik、Envoy。


IngressController nginx运行在Pod中，其配置文件是在Pod中。后端代理的Pod随时会发生变动，IngressController需要watch API当中的后端Pod资源的改变。IngressController自身无法识别目前符合自己关联的（条件的）被代理的Pod资源有哪些，IngressController需借助service来实现。

因此要想定义一个对应的调度功能，还需要创建service，此service通过label selector关联至每一个upstream服务器组，通过此service资源关联至后端的Pod。此service不会被当做被代理时的中间节点，它仅仅是为Pod做分类的。此service关联的Pod，就将其写入upstream中。

在Kubernetes中有一种特殊资源叫做Ingress，当Pod发生改变时，其servcie对应的资源也会发生改变， 依赖于IngressResource将变化结果反应至配置文件中。

Ingress定义期望IngressController如何创建前段代理资源（虚拟主机、Url路由映射），同时定义后端池（upstream）。upstream中的列表数量，是通过service获得。

Ingress可以通过编辑注入到IngressController中，并保存为配置文件，且Ingress发现service选定的后端Pod资源发生改变，此改变会及时反映至Ingress中，Ingress将其注入到前端调度器Pod中，并触发Pod中的container主进程（nginx）重载配置文件。

要想使用Ingress功能，需要有service对某些后端资源进行分类，而后Ingress通过分类识别出Pod的数量和IP地址信息，并将反映结果生成配置信息注入到upstream中。


IngressController根据自身需求方式来定义前端，而后根据servcie收集到的后端Pod IP定义成upstream server，将这些信息反映在Ingress server当中，由Ingress动态注入到IngressController当中。

Ingress也是标准的Kubernetes资源，定义Ingress时同样类似于Pod方式来定义。使用`kubectl explain Ingress`查看帮助。

- spec
 - rules 规则，对象列表
     - host 主机调度 虚拟主机而非url映射
     - http 
        - paths 路径调度
          - backend
          - path 
 - backend 定义被调度的后端主机，靠service定义，找到后端相关联的Pod资源。
   - serviceName 后端servcie名称，即用来关联Pod资源的service。
   - servicePort 


IngressController部署

namespace.yaml 创建名称空间
configmap.yaml 为nginx从外部注入配置的
rbac.yaml 定义集群角色、授权。必要时让IngressController拥有访问他本身到达不了的名称空间的权限




ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: defualt # 与deployment和要发布的service处在同一名称空间内
  annotations:
    kubernetes.io/ingress.class: &#34;nginx&#34;
spec:
  rules:
  - host: test.baidu.com # 这个是外部访问域名，service映射到主机节点地址上
    http:
      paths:
      - pach: 
        backend:
          serviceName: myapp 
          servicePort: 80
```

证书是不能直接贴入ingress中的，需要将其转为特殊格式 secret， secret是标准的Kubernetes对象，c可以直接注入到Pod中被Pod所引用。


[Kubernetes的负载均衡问题(Nginx Ingress) - ericnie - 博客园](https://www.cnblogs.com/ericnie/p/6965091.html) 

[Installation Guide - NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/#verify-installation)
