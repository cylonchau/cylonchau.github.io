# client-go controller


[toc]

## Overview

根据Kuberneter文档对[Controller](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/controllers.md)的描述，Controller在kubernetes中是负责协调的组件，根据设计模式可知，controller会不断的你的对象（如Pod）从当前状态与期望状态同步的一个过程。当然Controller会监听你的实际状态与期望状态。

## Writing Controllers

```go
package main

import (
	&#34;flag&#34;
	&#34;fmt&#34;
	&#34;os&#34;
	&#34;time&#34;

	v1 &#34;k8s.io/api/core/v1&#34;
	&#34;k8s.io/apimachinery/pkg/fields&#34;
	utilruntime &#34;k8s.io/apimachinery/pkg/util/runtime&#34;
	&#34;k8s.io/apimachinery/pkg/util/wait&#34;
	&#34;k8s.io/client-go/kubernetes&#34;
	&#34;k8s.io/client-go/rest&#34;
	&#34;k8s.io/client-go/tools/cache&#34;
	&#34;k8s.io/client-go/tools/clientcmd&#34;
	&#34;k8s.io/client-go/util/homedir&#34;
	&#34;k8s.io/client-go/util/workqueue&#34;
	&#34;k8s.io/klog&#34;
)

type Controller struct {
	lister     cache.Indexer
	controller cache.Controller
	queue      workqueue.RateLimitingInterface
}

func NewController(lister cache.Indexer, controller cache.Controller, queue workqueue.RateLimitingInterface) *Controller {
	return &amp;Controller{
		lister:     lister,
		controller: controller,
		queue:      queue,
	}
}

func (c *Controller) processItem() bool {
	item, quit := c.queue.Get()
	if quit {
		return false
	}
	defer c.queue.Done(item)
	fmt.Println(item)
	err := c.processWrapper(item.(string))
	if err != nil {
		c.handleError(item.(string))
	}
	return true
}

func (c *Controller) handleError(key string) {

	if c.queue.NumRequeues(key) &lt; 3 {
		c.queue.AddRateLimited(key)
		return
	}
	c.queue.Forget(key)
	klog.Infof(&#34;Drop Object %s in queue&#34;, key)
}

func (c *Controller) processWrapper(key string) error {
	item, exists, err := c.lister.GetByKey(key)
	if err != nil {
		klog.Error(err)
		return err
	}
	if !exists {
		klog.Info(fmt.Sprintf(&#34;item %v not exists in cache.\n&#34;, item))
	} else {
		fmt.Println(item.(*v1.Pod).GetName())
	}
	return err
}

func (c *Controller) Run(threadiness int, stopCh chan struct{}) {
	defer utilruntime.HandleCrash()
	defer c.queue.ShutDown()
	klog.Infof(&#34;Starting custom controller&#34;)

	go c.controller.Run(stopCh)

	if !cache.WaitForCacheSync(stopCh, c.controller.HasSynced) {
		utilruntime.HandleError(fmt.Errorf(&#34;sync failed.&#34;))
		return
	}

	for i := 0; i &lt; threadiness; i&#43;&#43; {
		go wait.Until(func() {
			for c.processItem() {
			}
		}, time.Second, stopCh)
	}
	&lt;-stopCh
	klog.Info(&#34;Stopping custom controller&#34;)
}

func main() {
	var (
		k8sconfig  *string //使用kubeconfig配置文件进行集群权限认证
		restConfig *rest.Config
		err        error
	)
	if home := homedir.HomeDir(); home != &#34;&#34; {
		k8sconfig = flag.String(&#34;kubeconfig&#34;, fmt.Sprintf(&#34;%s/.kube/config&#34;, home), &#34;kubernetes auth config&#34;)
	}
	k8sconfig = k8sconfig
	flag.Parse()
	if _, err := os.Stat(*k8sconfig); err != nil {
		panic(err)
	}

	if restConfig, err = rest.InClusterConfig(); err != nil {
		// 这里是从masterUrl 或者 kubeconfig传入集群的信息，两者选一
		restConfig, err = clientcmd.BuildConfigFromFlags(&#34;&#34;, *k8sconfig)
		if err != nil {
			panic(err)
		}
	}
	restset, err := kubernetes.NewForConfig(restConfig)
	lister := cache.NewListWatchFromClient(restset.CoreV1().RESTClient(), &#34;pods&#34;, &#34;default&#34;, fields.Everything())
	queue := workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())
	indexer, controller := cache.NewIndexerInformer(lister, &amp;v1.Pod{}, 0, cache.ResourceEventHandlerFuncs{
		AddFunc: func(obj interface{}) {
			fmt.Println(&#34;add &#34;, obj.(*v1.Pod).GetName())
			key, err := cache.MetaNamespaceKeyFunc(obj)
			if err == nil {
				queue.Add(key)
			}

		},
		UpdateFunc: func(oldObj, newObj interface{}) {
			fmt.Println(&#34;update&#34;, newObj.(*v1.Pod).GetName())
			if newObj.(*v1.Pod).Status.Conditions[0].Status == &#34;True&#34; {
				fmt.Println(&#34;update: the Initialized Status&#34;, newObj.(*v1.Pod).Status.Conditions[0].Status)
			} else {
				fmt.Println(&#34;update: the Initialized Status &#34;, newObj.(*v1.Pod).Status.Conditions[0].Status)
				fmt.Println(&#34;update: the Initialized Reason &#34;, newObj.(*v1.Pod).Status.Conditions[0].Reason)
			}

			if len(newObj.(*v1.Pod).Status.Conditions) &gt; 1 {
				if newObj.(*v1.Pod).Status.Conditions[1].Status == &#34;True&#34; {
					fmt.Println(&#34;update: the Ready Status&#34;, newObj.(*v1.Pod).Status.Conditions[1].Status)
				} else {
					fmt.Println(&#34;update: the Ready Status &#34;, newObj.(*v1.Pod).Status.Conditions[1].Status)
					fmt.Println(&#34;update: the Ready Reason &#34;, newObj.(*v1.Pod).Status.Conditions[1].Reason)
				}

				if newObj.(*v1.Pod).Status.Conditions[2].Status == &#34;True&#34; {
					fmt.Println(&#34;update: the PodCondition Status&#34;, newObj.(*v1.Pod).Status.Conditions[2].Status)
				} else {
					fmt.Println(&#34;update: the PodCondition Status &#34;, newObj.(*v1.Pod).Status.Conditions[2].Status)
					fmt.Println(&#34;update: the PodCondition Reason &#34;, newObj.(*v1.Pod).Status.Conditions[2].Reason)
				}

				if newObj.(*v1.Pod).Status.Conditions[3].Status == &#34;True&#34; {
					fmt.Println(&#34;update: the PodScheduled Status&#34;, newObj.(*v1.Pod).Status.Conditions[3].Status)
				} else {
					fmt.Println(&#34;update: the PodScheduled Status &#34;, newObj.(*v1.Pod).Status.Conditions[3].Status)
					fmt.Println(&#34;update: the PodScheduled Reason &#34;, newObj.(*v1.Pod).Status.Conditions[3].Reason)
				}
			}

		},
		DeleteFunc: func(obj interface{}) {
			fmt.Println(&#34;delete &#34;, obj.(*v1.Pod).GetName(), &#34;Status &#34;, obj.(*v1.Pod).Status.Phase)
			// 上面是事件函数的处理，下面是对workqueue的操作
			key, err := cache.MetaNamespaceKeyFunc(obj)
			if err == nil {
				queue.Add(key)
			}
		},
	}, cache.Indexers{})

	c := NewController(indexer, controller, queue)
	stopCh := make(chan struct{})
	stopCh1 := make(chan struct{})
	c.Run(1, stopCh)
	defer close(stopCh)
	&lt;-stopCh1
}

```

通过日志可以看出，Pod create后的步骤大概为4步：

- Initialized：初始化好后状态为Pending
- PodScheduled：然后调度
- PodCondition
- Ready

```
add  netbox
default/netbox
netbox
update netbox status Pending to Pending
update: the Initialized Status True
update netbox status Pending to Pending
update: the Initialized Status True
update: the Ready Status  False
update: the Ready Reason  ContainersNotReady
update: the PodCondition Status  False
update: the PodCondition Reason  ContainersNotReady
update: the PodScheduled Status True


update netbox status Pending to Running
update: the Initialized Status True
update: the Ready Status True
update: the PodCondition Status True
update: the PodScheduled Status True
```

大致上与 `kubectl describe pod` 看到的内容页相似

```
default-scheduler  Successfully assigned default/netbox to master-machine
  Normal  Pulling    85s   kubelet            Pulling image &#34;cylonchau/netbox&#34;
  Normal  Pulled     30s   kubelet            Successfully pulled image &#34;cylonchau/netbox&#34;
  Normal  Created    30s   kubelet            Created container netbox
  Normal  Started    30s   kubelet            Started container netbox
```

&gt; **Reference**
&gt;
&gt; [controllers.md](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/controllers.md)
