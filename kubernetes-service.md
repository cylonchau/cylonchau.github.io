# Kubernetes Service


在Kubernetes集群中，Pod是有生命周期的，为了能够给对应的客户端提供一个固定访问端点，因此在客户端与服务端（Pod之间）添加了一个固定中间层，这个中间层被称之为Service。Service的工作严重依赖于在Kubernetes集群之上，部署的附件Kubernetes DNS服务。较新版本使用的coreDNS，1.11之前使用的KubeDNS。 


- service的名称解析是强依赖于DNS附件的。因此在部署完Kubernetes后，需要部署CoreDNS或KubeDNS。
- Kubernetes要想向客户端提供网络功能，依赖于第三方方案，在较新版本中，可通过CNI容器网络插件标准接口，来接入任何遵循插件标准的第三方方案。

Service从一定程度上来说，在每个节点之上都工作有一个组件Kube-proxy，Kube-proxy将始终监视apiserver当中，有关service资源的变动状态。此过程是通过Kubernetes中固有的请求方法watch来实现的。一旦有service资源的内容发生变动，kube-proxy都将其转换为当前节点之上的能够实现service资源调度至特定Pod之上的规则。


### service实现方式

在Kubernetes中service的实现方式有三种模型。

- **userspace** 用户空间，可以理解为，用户的请求。 1.1之前包括1.1使用此模型。

用户的请求到达当前节点的内核空间的iptables规则（service规则），由service转发至本地监听的某个套接字上的用户空间的kube-proxy，kube-proxy在处理完再转发给service，最终代理至service相关联的各个Pod，实现调度。

![image-20221025004009491](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025004009491.png)

- **iptables** 1.10-

客户端IP请求时，直接请求serviceIP，IP为本地内核空间中的service规则所截取，并直接调度至相关Pod。service工作在内核空间，由iptables直接调度。

![image-20221025004018408](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025004018408.png)

- **ipvs**  1.11默认使用，如IPVS没有激活，默认降级为iptables

客户端请求到达内核空间后，直接由ipvs规则直接调度至Pod网络地址范围内的相关Pod资源。

![image-20221025004051034](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025004051034.png)

### 使用清单创建service资源

SVC中的kubernetes service是集群中各Pod需要与Kubernetes集群apiserver联系时需要通过此svc地址联系。这个地址是集群内的apiserver服务地址。


#### service类型

- **ClusterIP** 默认值，表示分配集群IP地址，仅用于集群内通信。自动分配地址，如需固定，需要指定相应地址，在创建后无法修改。当使用ClusterIP时，只有两个端口有用，`port`与`targetPort`

- **NodePort**  接入集群外部流量，默认分配的端口是30000~32767

- **LoadBalancer**  表示将Kubernetes部署在虚拟机上，虚拟机是工作在云环境中，云环境支持lbaas（负载均衡及服务的一键调用）。

- **ExternaName** 表示将集群外部服务引用到集群内部中来，在集群内部直接使用。

```yaml
spec:
  ports: # 将哪个端口与后端容器端口建立关联关系。
  - port # service对外提供服务的端口
    name 指明port的名称
    targetPort # 容器的端口
    nodePort # 只有类型为NodePort时，才有必要用节点端口，否则此选项是无用的。
    protocol 协议，默认TCP
  seletcor 关联到哪些Pod资源上
    app: redis
    run: redis
  clusterIP: # clusterIP可以动态分贝可以不配置
  type: ClusterIP
```



```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: default
spec:
  selector: 
    run: redis
  clusterIP: 10.96.100.0
  type: ClusterIP
  ports: 
  - port: 6379
    targetPort: 6379
```


service到Pod是有一个中间层的，service会先到endpoints资源(==标准的Kubernetes对象==)， 地址加端口，而后由endpoints关联至后端Pod。

```sh
$ kubectl describe svc redis
Name:              redis
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"redis","namespace":"default"},"spec":{"clusterIP":"10.96.100.0",...
Selector:          run=redis
Type:              ClusterIP
IP:                10.96.100.0
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         10.244.0.5:6379,10.244.1.4:6379
Session Affinity:  None
Events:            <none>
```

service创建完成后，只要kubernetes集群中的DNS是存在的，就可以直接解析其服务名每一个service创建完后，都会在集群DNS中动态添加一个资源记录（不止一个）。

资源记录的默认格式为，==`SVC_NAME.NS_NAME.DOMAIN.LTD.`==，==`DOMAIN.LTD`== 默认是 ==`svc.cluster.local`==。故redis-svc的资源记录为`redis.default.svc.cluster.local.`

#### nodePort SVC

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis1
  namespace: default
spec:
  selector: 
    run: redis
  clusterIP: 10.96.100.1
  type: NodePort
  ports: 
  - port: 6380
    targetPort: 6379
    nodePort: 30001
```

#### ExternalName

在本地局域网环境中，但是在Kubernetes集群之外，或者在互联网之上的服务，我们==期望此服务让集群内的服务可访问到==，集群内部使用的都是私网地址，就算可以将请求路由出去，离开本地网络到外部，外部的相应报文也无法回到Kubernetes集群内网中。此时无法正常通信。

ExternalName用于实现，在集群中创建service， 此service端点不是本地Pod，而是service关联至外部服务上。当集群内部客户端区访问service时，由service通过层级转换，请求到外部的服务，外部报文响应给NodeIP，再由NodeIP转交至service，再由service转发至Pod，从而使Pod可访问集群外部服务。 

```sh
$ kubectl explain svc.spec.externalName
KIND:     Service
VERSION:  v1

FIELD:    externalName <string>
```

Service在实现负载均衡时，还支持`sessionAffinity`会话粘性，默认情况下基于源IP做粘性的，`ClientIP`、`None`(默认)。

sessionAffinity在service内部实现session保持。支持两种模式 `ClusterIP` `None`。设置`ClusterIP`为，将同一个客户端IP调度到同一个后端Pod。

#### 无头service (headless)

在访问service时，解析的应为service名称，每个service有其响应的service名称，解析至其ClusterIP，由service调度（dnat）至后端Pod，因此名称解析结果只会有一个ClusterIP。

所谓headless service，即在解析Service IP时，==无Service ClusterIP，此时，解析服务名时会解析至后端Pod IP之上==，IP数量取决于Pod的数量。这种service被称为`headless service`。

![image-20221025004104717](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/image-20221025004104717.png)

设置方式

```yaml
ClusterIP: none
```

```sh
$ kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP          15h
nginx        ClusterIP   None          <none>        80/TCP           5s
```


```sh
$ dig -t A nginx.default.svc.cluster.local. @10.96.0.10

; <<>> DiG 9.9.4-RedHat-9.9.4-74.el7_6.1 <<>> -t A nginx.default.svc.cluster.local. @10.96.0.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59245
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;nginx.default.svc.cluster.local. IN    A

;; ANSWER SECTION:
nginx.default.svc.cluster.local. 5 IN   A       10.244.0.6
nginx.default.svc.cluster.local. 5 IN   A       10.244.1.5

;; Query time: 1 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: 一 7月 08 13:24:47 CST 2019
;; MSG SIZE  rcvd: 154


$ kubectl get pods -o wide -l run=nginx
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx-7db9fccd9b-h72bb   1/1     Running   0          14m   10.244.1.5   node01   <none>           <none>
nginx-7db9fccd9b-mrv95   1/1     Running   0          14m   10.244.0.6   node02   <none>           <none>
```
