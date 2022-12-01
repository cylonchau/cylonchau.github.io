# Understand Kubernetes - kubectl


kubectl是apiserver的客户端程序，是通过连接master节点上的apiserver。kubectl是整个kubernetes集群管理入口的客户端工具。他可以连接至apiserver上，实现各种k8s相关对象资源的增删改查等基本操作。

快速获取并允许此镜像为容器

查看kubectl和apiserver的版本号

```sh
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.1", GitCommit:"eec55b9ba98609a46fee712359c7b5b365bdd920", GitTreeState:"clean", BuildDate:"2018-12-13T10:39:04Z", GoVersion:"go1.11.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.1", GitCommit:"eec55b9ba98609a46fee712359c7b5b365bdd920", GitTreeState:"clean", BuildDate:"2018-12-13T10:31:33Z", GoVersion:"go1.11.2", Compiler:"gc", Platform:"linux/amd64"}
```

查看集群信息

```sh
$ kubectl cluster-info
Kubernetes master is running at https://node01.test.k8s:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


desicribe 描述资源的详细信息

```sh
$ kubectl describe node node02.k8s.test
Name:               node02.k8s.test
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=node02.k8s.test
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"8e:c7:43:db:3c:08"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 10.0.0.16
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 25 Dec 2018 18:34:57 +0800
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Wed, 02 Jan 2019 16:16:32 +0800   Tue, 25 Dec 2018 18:34:57 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Wed, 02 Jan 2019 16:16:32 +0800   Tue, 25 Dec 2018 18:34:57 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Wed, 02 Jan 2019 16:16:32 +0800   Tue, 25 Dec 2018 18:34:57 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Wed, 02 Jan 2019 16:16:32 +0800   Tue, 01 Jan 2019 23:13:19 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  10.0.0.16
  Hostname:    node02.k8s.test
Capacity:
 cpu:                2
 ephemeral-storage:  9517Mi
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             1867048Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  8981367998
 hugepages-1Gi:      0
 hugepages-2Mi:      0
 memory:             1764648Ki
 pods:               110
System Info:
 Machine ID:                 886a31bc590e4c7cb2ee454a15a07de3
 System UUID:                FB914D56-C297-1F61-3F74-5806C0299A09
 Boot ID:                    d8e85197-a972-4601-a2b1-fdf896fe31da
 Kernel Version:             3.10.0-693.el7.x86_64
 OS Image:                   CentOS Linux 7 (Core)
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.9.0
 Kubelet Version:            v1.13.1
 Kube-Proxy Version:         v1.13.1
PodCIDR:                     10.244.0.0/24
Non-terminated Pods:         (7 in total)
  Namespace                  Name                           CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                           ------------  ----------  ---------------  -------------  ---
  default                    client-f5cdb799f-vjglq         0 (0%)        0 (0%)      0 (0%)           0 (0%)         7d19h
  default                    httpd-5d8cbbcd67-cp5p7         0 (0%)        0 (0%)      0 (0%)           0 (0%)         16h
  default                    nginx-2-c988474bf-c4ds9        0 (0%)        0 (0%)      0 (0%)           0 (0%)         7d20h
  default                    nginx-2-c988474bf-ghgkh        0 (0%)        0 (0%)      0 (0%)           0 (0%)         7d20h
  default                    nginx-7cdbd8cdc9-bsrf2         0 (0%)        0 (0%)      0 (0%)           0 (0%)         7d21h
  default                    redis-785f9d6bfb-w74cr         0 (0%)        0 (0%)      0 (0%)           0 (0%)         7d20h
  kube-system                kube-flannel-ds-amd64-lvnp8    100m (5%)     100m (5%)   50Mi (2%)        50Mi (2%)      7d20h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (5%)  100m (5%)
  memory             50Mi (2%)  50Mi (2%)
  ephemeral-storage  0 (0%)     0 (0%)
Events:
  Type     Reason                   Age    From                         Message
  ----     ------                   ----   ----                         -------
  Normal   NodeReady                7d20h  kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeReady
  Normal   Starting                 17h    kubelet, node02.k8s.test     Starting kubelet.
  Normal   NodeHasSufficientMemory  17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeHasSufficientPID
  Warning  Rebooted                 17h    kubelet, node02.k8s.test     Node node02.k8s.test has been rebooted, boot id: d8e85197-a972-4601-a2b1-fdf896fe31da
  Normal   NodeNotReady             17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeNotReady
  Normal   NodeAllocatableEnforced  17h    kubelet, node02.k8s.test     Updated Node Allocatable limit across pods
  Normal   NodeReady                17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeReady
  Normal   Starting                 17h    kube-proxy, node02.k8s.test  Starting kube-proxy.
  Normal   Starting                 17h    kube-proxy, node02.k8s.test  Starting kube-proxy.
  Normal   Starting                 17h    kubelet, node02.k8s.test     Starting kubelet.
  Normal   NodeHasSufficientMemory  17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     17h    kubelet, node02.k8s.test     Node node02.k8s.test status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  17h    kubelet, node02.k8s.test     Updated Node Allocatable limit across pods
```


lable 设置、获取分类资源。
annotate 注解，给资源额外增加其他键值数据，与lable不同之处在于，长度不受限制，可打任何key value数据


kubectl run :创建并运行一个特定的镜像，最好以带副本方式运行。创建一个基于deployment或job控制器中的某一种来管理创建容易。

```sh
kubectl run nginx --image=nginx --relicas=5
```
基于nginx镜像的创建nginx控制器，启动5个nginx资源

--restart=Nerver 表示容器结束后不补上去

-- `<cmd>` `<age1>` ... `<age2>` ... `<ageN>` 基于某容器启动时不运行镜像中默认运行的命令。

--schedule 创建的控制器不再是deployment而是cron job


查看被创建的deployment

```sh
$ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
client    1/1     1            1           8d
httpd     2/2     2            2           39h
nginx     1/1     1            1           8d
nginx-2   2/2     2            2           8d
redis     1/1     1            1           8d
```

UP-TO-DATE 当前处于最新状态
AVAILABLE 可用的有几个，pod创建完成后，会对其做就绪性探测。
READY 内部一共有几个pod/有几个pod已经就绪



$  kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
client-f5cdb799f-vjglq    1/1     Running   1          8d
httpd-5d8cbbcd67-brp4w    1/1     Running   0          39h
httpd-5d8cbbcd67-cp5p7    1/1     Running   0          39h
nginx-2-c988474bf-c4ds9   1/1     Running   1          8d
nginx-2-c988474bf-ghgkh   1/1     Running   1          8d
nginx-7cdbd8cdc9-bsrf2    1/1     Running   1          8d
redis-785f9d6bfb-w74cr    1/1     Running   1          8d

显示更多的信息`-o wide`

```bash
$  kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
client-f5cdb799f-vjglq    1/1     Running   1          8d    10.244.0.11   node02.k8s.test   [node]           [node]
httpd-5d8cbbcd67-brp4w    1/1     Running   0          39h   10.244.1.2    node03.k8s.test   [node]           [node]
httpd-5d8cbbcd67-cp5p7    1/1     Running   0          39h   10.244.0.12   node02.k8s.test   [node]           [node]
nginx-2-c988474bf-c4ds9   1/1     Running   1          8d    10.244.0.7    node02.k8s.test   [node]           [node]
nginx-2-c988474bf-ghgkh   1/1     Running   1          8d    10.244.0.8    node02.k8s.test   [node]           [node]
nginx-7cdbd8cdc9-bsrf2    1/1     Running   1          8d    10.244.0.10   node02.k8s.test   [node]           [node]
redis-785f9d6bfb-w74cr    1/1     Running   1          8d    10.244.0.9    node02.k8s.test   [node]           [node]
```

Pod的地址只能在kubernetes中的node节点上使用，客户端一般分为两类，一类为其他Pod，另一类为集群外部客户端

```
$ kubectl expose deployment httpd --name=httpd-svc --port=80 --target-port=80
service/httpd-svc exposed

$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
httpd-svc    ClusterIP   10.98.233.10    <none>        80/TCP    19s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   9d
n            ClusterIP   10.101.107.86   <none>        80/TCP    8d
```


一般是集群内部节点访问的地址，更多情况下svc地址是被Pod客户端访问的。Pod客户端在访问时，完全可以实现基于service自己的名称来访问。只不过在Pod客户端中需要能解析此名称才可以，而解析时依赖coreDNS服务

查看coreDNS地址为10.244.0.13和10.244.1.3
```sh
$ kubectl get pods -n kube-system -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
coredns-7748f7f6df-h8tbp      1/1     Running   0          23h   10.244.0.13   node02.k8s.test   [node]           [node]
coredns-7748f7f6df-xqqks      1/1     Running   0          23h   10.244.1.3    node03.k8s.test   [node]           [node]
kube-flannel-ds-amd64-24d25   1/1     Running   0          40h   10.0.0.17     node03.k8s.test   [node]           [node]
kube-flannel-ds-amd64-lvnp8   1/1     Running   1          8d    10.0.0.16     node02.k8s.test   [node]           [node]
```
一般情况下，我们不直接使用地址访问，coredns自身也有服务名，我们使用10.96.0.10来解析。
```sh
$ kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   [node]        53/UDP,53/TCP,9153/TCP   23h

```

查看资源详细信息

```sh
$ kubectl describe svc httpd-svc
Name:              httpd-svc
Namespace:         default
Labels:            run=httpd
Annotations:       [node]
Selector:          run=httpd
Type:              ClusterIP
IP:                10.98.233.10
Port:              [unset]  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.0.12:80,10.244.1.2:80
Session Affinity:  None
Events:            [node]
```

对于Service、Pod来讲这些资源都是可以被改变的，使用`kubectl edit`改变相关信息
