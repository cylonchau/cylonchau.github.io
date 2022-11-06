# 脱管与kubernetes上的leader election应用


## Backgroud

前一章中，对kubernetes的选举原理进行了深度剖析，下面就通过一个example来实现一个，利用kubernetes提供的选举机制完成的高可用应用。

对于此章需要提前对一些概念有所了解后才可以继续看下去

- leader election mechanism
- RBCA
- Pod runtime mechanism

## Implementation

### 代码实现

如果仅仅是使用Kubernetes中的锁，实现的代码也只有几行而已。

```go
package main

import (
	&#34;context&#34;
	&#34;flag&#34;
	&#34;fmt&#34;
	&#34;os&#34;
	&#34;os/signal&#34;
	&#34;syscall&#34;
	&#34;time&#34;

	metav1 &#34;k8s.io/apimachinery/pkg/apis/meta/v1&#34;
	clientset &#34;k8s.io/client-go/kubernetes&#34;
	&#34;k8s.io/client-go/rest&#34;
	&#34;k8s.io/client-go/tools/clientcmd&#34;
	&#34;k8s.io/client-go/tools/leaderelection&#34;
	&#34;k8s.io/client-go/tools/leaderelection/resourcelock&#34;
	&#34;k8s.io/klog/v2&#34;
)

func buildConfig(kubeconfig string) (*rest.Config, error) {
	if kubeconfig != &#34;&#34; {
		cfg, err := clientcmd.BuildConfigFromFlags(&#34;&#34;, kubeconfig)
		if err != nil {
			return nil, err
		}
		return cfg, nil
	}

	cfg, err := rest.InClusterConfig()
	if err != nil {
		return nil, err
	}
	return cfg, nil
}

func main() {
	klog.InitFlags(nil)

	var kubeconfig string
	var leaseLockName string
	var leaseLockNamespace string
	var id string
	// 初始化客户端的部分
	flag.StringVar(&amp;kubeconfig, &#34;kubeconfig&#34;, &#34;&#34;, &#34;absolute path to the kubeconfig file&#34;)
	flag.StringVar(&amp;id, &#34;id&#34;, &#34;&#34;, &#34;the holder identity name&#34;)
	flag.StringVar(&amp;leaseLockName, &#34;lease-lock-name&#34;, &#34;&#34;, &#34;the lease lock resource name&#34;)
	flag.StringVar(&amp;leaseLockNamespace, &#34;lease-lock-namespace&#34;, &#34;&#34;, &#34;the lease lock resource namespace&#34;)
	flag.Parse()

	if leaseLockName == &#34;&#34; {
		klog.Fatal(&#34;unable to get lease lock resource name (missing lease-lock-name flag).&#34;)
	}
	if leaseLockNamespace == &#34;&#34; {
		klog.Fatal(&#34;unable to get lease lock resource namespace (missing lease-lock-namespace flag).&#34;)
	}
	config, err := buildConfig(kubeconfig)
	if err != nil {
		klog.Fatal(err)
	}
	client := clientset.NewForConfigOrDie(config)

	run := func(ctx context.Context) {
		// 实现的业务逻辑，这里仅仅为实验，就直接打印了
		klog.Info(&#34;Controller loop...&#34;)

		for {
			fmt.Println(&#34;I am leader, I was working.&#34;)
			time.Sleep(time.Second * 5)
		}
	}

	// use a Go context so we can tell the leaderelection code when we
	// want to step down
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// 监听系统中断
	ch := make(chan os.Signal, 1)
	signal.Notify(ch, os.Interrupt, syscall.SIGTERM)
	go func() {
		&lt;-ch
		klog.Info(&#34;Received termination, signaling shutdown&#34;)
		cancel()
	}()

	// 创建一个资源锁
	lock := &amp;resourcelock.LeaseLock{
		LeaseMeta: metav1.ObjectMeta{
			Name:      leaseLockName,
			Namespace: leaseLockNamespace,
		},
		Client: client.CoordinationV1(),
		LockConfig: resourcelock.ResourceLockConfig{
			Identity: id,
		},
	}

	// 开启一个选举的循环
	leaderelection.RunOrDie(ctx, leaderelection.LeaderElectionConfig{
		Lock:            lock,
		ReleaseOnCancel: true,
		LeaseDuration:   60 * time.Second,
		RenewDeadline:   15 * time.Second,
		RetryPeriod:     5 * time.Second,
		Callbacks: leaderelection.LeaderCallbacks{
			OnStartedLeading: func(ctx context.Context) {
				// 当选举为leader后所运行的业务逻辑
				run(ctx)
			},
			OnStoppedLeading: func() {
				// we can do cleanup here
				klog.Infof(&#34;leader lost: %s&#34;, id)
				os.Exit(0)
			},
			OnNewLeader: func(identity string) { // 申请一个选举时的动作
				if identity == id {
					return
				}
				klog.Infof(&#34;new leader elected: %s&#34;, identity)
			},
		},
	})
}
```

&gt; 注：这种lease锁只能在in-cluster模式下运行，如果需要类似二进制部署的程序，可以选择endpoint类型的资源锁。

### 生成镜像

这里已经制作好了镜像并上传到dockerhub（`cylonchau/leaderelection:v0.0.2`）上了，如果只要学习运行原理，则忽略此步骤

```docker
FROM golang:alpine AS builder
MAINTAINER cylon
WORKDIR /election
COPY . /election
ENV GOPROXY https://goproxy.cn,direct
RUN GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o elector main.go

FROM alpine AS runner
WORKDIR /go/elector
COPY --from=builder /election/elector .
VOLUME [&#34;/election&#34;]
ENTRYPOINT [&#34;./elector&#34;]
```

### 准备资源清单

默认情况下，Kubernetes运行的pod在请求Kubernetes集群内资源时，默认的账户是没有权限的，默认服务帐户无权访问协调 API，因此我们需要创建另一个serviceaccount并相应地设置 对应的RBAC权限绑定；在清单中配置上这个sa，此时所有的pod就会有协调锁的权限了

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-leaderelection
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: leaderelection
rules:
  - apiGroups:
      - coordination.k8s.io
    resources:
      - leases
    verbs:
      - &#39;*&#39;
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: leaderelection
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: leaderelection
subjects:
  - kind: ServiceAccount
    name: sa-leaderelection
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: leaderelection
  name: leaderelection
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: leaderelection
  template:
    metadata:
      labels:
        app: leaderelection
    spec:
      containers:
        - image: cylonchau/leaderelection:v0.0.2
          imagePullPolicy: IfNotPresent
          command: [&#34;./elector&#34;]
          args:
          - &#34;-id=$(POD_NAME)&#34;
          - &#34;-lease-lock-name=test&#34;
          - &#34;-lease-lock-namespace=default&#34;
          env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.name
          name: elector
      serviceAccountName: sa-leaderelection
```

### 集群中运行

执行完清单后，当pod启动后，可以看到会创建出一个 lease

```bash
$ kubectl get lease
NAME   HOLDER                            AGE
test   leaderelection-5644c5f84f-frs5n   1s


$ kubectl describe lease
Name:         test
Namespace:    default
Labels:       &lt;none&gt;
Annotations:  &lt;none&gt;
API Version:  coordination.k8s.io/v1
Kind:         Lease
Metadata:
  Creation Timestamp:  2022-06-28T16:39:45Z
  Managed Fields:
    API Version:  coordination.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        f:acquireTime:
        f:holderIdentity:
        f:leaseDurationSeconds:
        f:leaseTransitions:
        f:renewTime:
    Manager:         elector
    Operation:       Update
    Time:            2022-06-28T16:39:45Z
  Resource Version:  131693
  Self Link:         /apis/coordination.k8s.io/v1/namespaces/default/leases/test
  UID:               bef2b164-a117-44bd-bad3-3e651c94c97b
Spec:
  Acquire Time:            2022-06-28T16:39:45.931873Z
  Holder Identity:         leaderelection-5644c5f84f-frs5n
  Lease Duration Seconds:  60
  Lease Transitions:       0
  Renew Time:              2022-06-28T16:39:55.963537Z
Events:                    &lt;none&gt;
```

通过其持有者的信息查看对应pod（因为程序中对holder Identity设置的是pod的名称），实际上是工作的pod。

如上实例所述，这是利用Kubernetes集群完成的leader选举的方案，虽然这不是最完美解决方案，但这是一种简单的方法，因为可以无需在集群上部署更多东西或者进行大量的代码工作就可以利用Kubernetes集群来完成一个高可用的HA应用。


