# Admission webhook


[toc]

## BACKGROUND

**admission controllers的特点**：

- 可定制性：准入功能可针对不同的场景进行调整。
- 可预防性：审计则是为了检测问题，而准入控制器可以预防问题发生
- 可扩展性：在kubernetes自有的验证机制外，增加了另外的防线，弥补了RBAC仅能对资源提供安全保证。

下图，显示了用户操作资源的流程，可以看出 *admission controllers* 作用是在通过身份验证资源持久化之前起到拦截作用。在准入控制器的加入会使kubernetes增加了更高级的安全功能。

![准入控制器阶段](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/admission-controller-phases.png)

&lt;center&gt;图：Kubernetes API 请求的请求处理步骤图&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/&lt;/center&gt;

这里找到一个大佬博客画的图，通过两张图可以很清晰的了解到admission webhook流程，与官方给出的不一样的地方在于，这里清楚地定位了kubernetes admission webhook 处于准入控制中，RBAC之后，push 之前。

![img](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/vhesGDFN3dLdzXwS7vzPdXkI3aglQYZgGhjc-Cx_boaV6URKFFoe8mFRZZUuJyGHywa_bOkeUlIkm-nJkCVMHPk9dr2dXFwNzAQJKzft2phsTcEDjdObjmugBcYtpdPLpLIYuIGzeFYvtsR2Lw.jpeg)
&lt;center&gt;图：Kubernetes API 请求的请求处理步骤图（详细）&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://www.armosec.io/blog/kubernetes-admission-controller/&lt;/center&gt;

### 两种控制器有什么区别？

根据官方提供的说法是

&gt; Mutating controllers may modify related objects to the requests they admit; validating controllers may not

从结构图中也可以看出，`validating ` 是在持久化之前，而 `Mutating ` 是在结构验证前，根据这些特性我们可以使用 `Mutating` 修改这个资源对象内容（如增加验证的信息），在 `validating` 中验证是否合法。

### composition of admission controllers

kubernetes中的  *admission controllers* 由两部分组成：

- 内置在APIServer中的准入控制器 [build-in li.st](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do)
- 特殊的控制器；也是内置在APIServer中，但提供一些自定义的功能
    - MutatingAdmission
    - ValidatingAdmission

Mutating 控制器可以修改他们处理的资源对象，Validating 控制器不会。当在任何一个阶段中的任何控制器拒绝这个了请求，则会立即拒绝整个请求，并将错误返回。

### admission webhook

由于准入控制器是内置在 `kube-apiserver` 中的，这种情况下就限制了admission controller的可扩展性。在这种背景下，kubernetes提供了一种可扩展的准入控制器 `extensible admission controllers`，这种行为叫做动态准入控制 `Dynamic Admission Control`，而提供这个功能的就是 `admission webhook` 。

`admission webhook`  通俗来讲就是 HTTP 回调，通过定义一个http server，接收准入请求并处理。用户可以通过kubernetes提供的两种类型的 `admission webhook`，*validating admission webhook* 和 *mutating admission webhook*。来完成自定义的准入策略的处理。

webhook 就是

&gt; 注：从上面的流程图也可以看出，admission webhook 也是有顺序的。首先调用mutating webhook，然后会调用validating webhook。

## 如何使用准入控制器

**使用条件**：kubernetes v1.16 使用 `admissionregistration.k8s.io/v1` ；kubernetes v1.9 使用 `admissionregistration.k8s.io/v1beta1`。

**如何在集群中开启准入控制器?** ：查看kube-apiserver 的启动参数 `--enable-admission-plugins` ；通过该参数来配置要启动的准入控制器，如 `--enable-admission-plugins=NodeRestriction` 多个准入控制器以 `,` 分割，顺序无关紧要。 反之使用 `--disable-admission-plugins` 参数可以关闭相应的准入控制器（Refer to [apiserver opts](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/#options)）。

通过 `kubectl` 命令可以看到，当前kubernetes集群所支持准入控制器的版本

```bash
$ kubectl api-versions | grep admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
```

## webhook工作原理

通过上面的学习，已经了解到了两种webhook的工作原理如下所示：

&gt; mutating webhook，会在持久化前拦截在 MutatingWebhookConfiguration 中定义的规则匹配的请求。MutatingAdmissionWebhook 通过向 mutating webhook 服务器发送准入请求来执行验证。
&gt;
&gt; validaing webhook，会在持久化前拦截在 `ValidatingWebhookConfiguration` 中定义的规则匹配的请求。ValidatingAdmissionWebhook 通过将准入请求发送到 validating webhook server来执行验证。

那么接下来将从源码中看这个在这个工作流程中，究竟做了些什么？

### 资源类型

对于 1.9 版本之后，也就是 `v1` 版本 ，admission 被定义在 [k8s.io\api\admissionregistration\v1\types.go](https://github.com/kubernetes/kubernetes/blob/v1.18.20/staging/src/k8s.io/api/admissionregistration/v1/types.go) ，大同小异，因为本地只有1.18集群，所以以这个讲解。

对于 `Validating Webhook` 来讲实现主要都在webhook中

```go
type ValidatingWebhookConfiguration struct {
    // 每个api必须包含下列的metadata，这个是kubernetes规范，可以在注释中的url看到相关文档
	metav1.TypeMeta `json:&#34;,inline&#34;`
	metav1.ObjectMeta `json:&#34;metadata,omitempty&#34; protobuf:&#34;bytes,1,opt,name=metadata&#34;`
	// Webhooks在这里被表示为[]ValidatingWebhook，表示我们可以注册多个
	// &#43;optional
	// &#43;patchMergeKey=name
	// &#43;patchStrategy=merge
	Webhooks []ValidatingWebhook `json:&#34;webhooks,omitempty&#34; patchStrategy:&#34;merge&#34; patchMergeKey:&#34;name&#34; protobuf:&#34;bytes,2,rep,name=Webhooks&#34;`
}
```

webhook，则是对这种类型的webhook提供的操作、资源等。对于这部分不做过多的注释了，因为这里本身为kubernetes API资源，官网有很详细的例子与说明。这里更多字段的意思的可以参考官方 [doc](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#webhook-configuration)

```go
type ValidatingWebhook struct {
	//  admission webhook的名词，Required
	Name string `json:&#34;name&#34; protobuf:&#34;bytes,1,opt,name=name&#34;`

	// ClientConfig 定义了与webhook通讯的方式 Required
	ClientConfig WebhookClientConfig `json:&#34;clientConfig&#34; protobuf:&#34;bytes,2,opt,name=clientConfig&#34;`

	// rule表示了webhook对于哪些资源及子资源的操作进行关注
	Rules []RuleWithOperations `json:&#34;rules,omitempty&#34; protobuf:&#34;bytes,3,rep,name=rules&#34;`

	// FailurePolicy 对于无法识别的value将如何处理，allowed/Ignore optional
	FailurePolicy *FailurePolicyType `json:&#34;failurePolicy,omitempty&#34; protobuf:&#34;bytes,4,opt,name=failurePolicy,casttype=FailurePolicyType&#34;`

	// matchPolicy 定义了如何使用“rules”列表来匹配传入的请求。
	MatchPolicy *MatchPolicyType `json:&#34;matchPolicy,omitempty&#34; protobuf:&#34;bytes,9,opt,name=matchPolicy,casttype=MatchPolicyType&#34;`
	NamespaceSelector *metav1.LabelSelector `json:&#34;namespaceSelector,omitempty&#34; protobuf:&#34;bytes,5,opt,name=namespaceSelector&#34;`
	SideEffects *SideEffectClass `json:&#34;sideEffects&#34; protobuf:&#34;bytes,6,opt,name=sideEffects,casttype=SideEffectClass&#34;`
	AdmissionReviewVersions []string `json:&#34;admissionReviewVersions&#34; protobuf:&#34;bytes,8,rep,name=admissionReviewVersions&#34;`
}
```

到这里了解了一个webhook资源的定义，那么这个如何使用呢？通过 `Find Usages` 找到一个 [k8s.io/apiserver/pkg/admission/plugin/webhook/accessors.go](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/admission/plugin/webhook/accessors.go) 在使用它。这里没有注释，但在结构上可以看出，包含客户端与一系列选择器组成

```go
type mutatingWebhookAccessor struct {
	*v1.MutatingWebhook
	uid               string
	configurationName string

	initObjectSelector sync.Once
	objectSelector     labels.Selector
	objectSelectorErr  error

	initNamespaceSelector sync.Once
	namespaceSelector     labels.Selector
	namespaceSelectorErr  error

	initClient sync.Once
	client     *rest.RESTClient
	clientErr  error
}
```

`accessor` 因为包含了整个webhookconfig定义的一些动作（这里个人这么觉得）。

`accessor.go` 下面 有一个 `GetRESTClient` 方法 ，通过这里可以看出，这里做的就是使用根据 `accessor` 构造一个客户端。

```go
func (m *mutatingWebhookAccessor) GetRESTClient(clientManager *webhookutil.ClientManager) (*rest.RESTClient, error) {
	m.initClient.Do(func() {
		m.client, m.clientErr = clientManager.HookClient(hookClientConfigForWebhook(m))
	})
	return m.client, m.clientErr
}
```

到这步骤已经没必要往下看了，因已经知道这里是请求webhook前的步骤了，下面就是何时请求了。

[k8s.io\apiserver\pkg\admission\plugin\webhook\validating\dispatcher.go](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/admission/plugin/webhook/validating/dispatcher.go) 下面有两个方法，Dispatch去请求我们自己定义的webhook

```go
func (d *validatingDispatcher) Dispatch(ctx context.Context, attr admission.Attributes, o admission.ObjectInterfaces, hooks []webhook.WebhookAccessor) error {
	var relevantHooks []*generic.WebhookInvocation
	// Construct all the versions we need to call our webhooks
	versionedAttrs := map[schema.GroupVersionKind]*generic.VersionedAttributes{}
	for _, hook := range hooks {
		invocation, statusError := d.plugin.ShouldCallHook(hook, attr, o)
		if statusError != nil {
			return statusError
		}
		if invocation == nil {
			continue
		}
		relevantHooks = append(relevantHooks, invocation)
		// If we already have this version, continue
		if _, ok := versionedAttrs[invocation.Kind]; ok {
			continue
		}
		versionedAttr, err := generic.NewVersionedAttributes(attr, invocation.Kind, o)
		if err != nil {
			return apierrors.NewInternalError(err)
		}
		versionedAttrs[invocation.Kind] = versionedAttr
	}

	if len(relevantHooks) == 0 {
		// no matching hooks
		return nil
	}

	// Check if the request has already timed out before spawning remote calls
	select {
	case &lt;-ctx.Done():
		// parent context is canceled or timed out, no point in continuing
		return apierrors.NewTimeoutError(&#34;request did not complete within requested timeout&#34;, 0)
	default:
	}

	wg := sync.WaitGroup{}
	errCh := make(chan error, len(relevantHooks))
	wg.Add(len(relevantHooks))
    // 循环所有相关的注册的hook
	for i := range relevantHooks {
		go func(invocation *generic.WebhookInvocation) {
			defer wg.Done()
            // invacation 中有一个 Accessor,Accessor注册了一个相关的webhookconfig
            // 也就是我们 kubectl -f 注册进来的那个webhook的相关配置
			hook, ok := invocation.Webhook.GetValidatingWebhook()
			if !ok {
				utilruntime.HandleError(fmt.Errorf(&#34;validating webhook dispatch requires v1.ValidatingWebhook, but got %T&#34;, hook))
				return
			}
			versionedAttr := versionedAttrs[invocation.Kind]
			t := time.Now()
            // 调用了callHook去请求我们自定义的webhook
			err := d.callHook(ctx, hook, invocation, versionedAttr)
			ignoreClientCallFailures := hook.FailurePolicy != nil &amp;&amp; *hook.FailurePolicy == v1.Ignore
			rejected := false
			if err != nil {
				switch err := err.(type) {
				case *webhookutil.ErrCallingWebhook:
					if !ignoreClientCallFailures {
						rejected = true
						admissionmetrics.Metrics.ObserveWebhookRejection(hook.Name, &#34;validating&#34;, string(versionedAttr.Attributes.GetOperation()), admissionmetrics.WebhookRejectionCallingWebhookError, 0)
					}
				case *webhookutil.ErrWebhookRejection:
					rejected = true
					admissionmetrics.Metrics.ObserveWebhookRejection(hook.Name, &#34;validating&#34;, string(versionedAttr.Attributes.GetOperation()), admissionmetrics.WebhookRejectionNoError, int(err.Status.ErrStatus.Code))
				default:
					rejected = true
					admissionmetrics.Metrics.ObserveWebhookRejection(hook.Name, &#34;validating&#34;, string(versionedAttr.Attributes.GetOperation()), admissionmetrics.WebhookRejectionAPIServerInternalError, 0)
				}
			}
			admissionmetrics.Metrics.ObserveWebhook(time.Since(t), rejected, versionedAttr.Attributes, &#34;validating&#34;, hook.Name)
			if err == nil {
				return
			}

			if callErr, ok := err.(*webhookutil.ErrCallingWebhook); ok {
				if ignoreClientCallFailures {
					klog.Warningf(&#34;Failed calling webhook, failing open %v: %v&#34;, hook.Name, callErr)
					utilruntime.HandleError(callErr)
					return
				}

				klog.Warningf(&#34;Failed calling webhook, failing closed %v: %v&#34;, hook.Name, err)
				errCh &lt;- apierrors.NewInternalError(err)
				return
			}

			if rejectionErr, ok := err.(*webhookutil.ErrWebhookRejection); ok {
				err = rejectionErr.Status
			}
			klog.Warningf(&#34;rejected by webhook %q: %#v&#34;, hook.Name, err)
			errCh &lt;- err
		}(relevantHooks[i])
	}
	wg.Wait()
	close(errCh)

	var errs []error
	for e := range errCh {
		errs = append(errs, e)
	}
	if len(errs) == 0 {
		return nil
	}
	if len(errs) &gt; 1 {
		for i := 1; i &lt; len(errs); i&#43;&#43; {
			// TODO: merge status errors; until then, just return the first one.
			utilruntime.HandleError(errs[i])
		}
	}
	return errs[0]
}

```

[callHook](https://github.com/kubernetes/kubernetes/blob/6adee9d4fb7a0cf3eec148448792e2ab091e1720/staging/src/k8s.io/apiserver/pkg/admission/plugin/webhook/validating/dispatcher.go#L216-L301) 可以理解为真正去请求我们自定义的webhook服务的动作

```go
func (d *validatingDispatcher) callHook(ctx context.Context, h *v1.ValidatingWebhook, invocation *generic.WebhookInvocation, attr *generic.VersionedAttributes) error {
   if attr.Attributes.IsDryRun() {
      if h.SideEffects == nil {
         return &amp;webhookutil.ErrCallingWebhook{WebhookName: h.Name, Reason: fmt.Errorf(&#34;Webhook SideEffects is nil&#34;)}
      }
      if !(*h.SideEffects == v1.SideEffectClassNone || *h.SideEffects == v1.SideEffectClassNoneOnDryRun) {
         return webhookerrors.NewDryRunUnsupportedErr(h.Name)
      }
   }

   uid, request, response, err := webhookrequest.CreateAdmissionObjects(attr, invocation)
   if err != nil {
      return &amp;webhookutil.ErrCallingWebhook{WebhookName: h.Name, Reason: err}
   }
   // 发生请求，可以看到，这里从上面的讲到的地方获取了一个客户端
   client, err := invocation.Webhook.GetRESTClient(d.cm)
   if err != nil {
      return &amp;webhookutil.ErrCallingWebhook{WebhookName: h.Name, Reason: err}
   }
   trace := utiltrace.New(&#34;Call validating webhook&#34;,
      utiltrace.Field{&#34;configuration&#34;, invocation.Webhook.GetConfigurationName()},
      utiltrace.Field{&#34;webhook&#34;, h.Name},
      utiltrace.Field{&#34;resource&#34;, attr.GetResource()},
      utiltrace.Field{&#34;subresource&#34;, attr.GetSubresource()},
      utiltrace.Field{&#34;operation&#34;, attr.GetOperation()},
      utiltrace.Field{&#34;UID&#34;, uid})
   defer trace.LogIfLong(500 * time.Millisecond)

   // 这里设置超时，超时时长就是在yaml资源清单中设置的那个值
   if h.TimeoutSeconds != nil {
      var cancel context.CancelFunc
      ctx, cancel = context.WithTimeout(ctx, time.Duration(*h.TimeoutSeconds)*time.Second)
      defer cancel()
   }
   // 直接用post请求我们自己定义的webhook接口
   r := client.Post().Body(request)

   // if the context has a deadline, set it as a parameter to inform the backend
   if deadline, hasDeadline := ctx.Deadline(); hasDeadline {
      // compute the timeout
      if timeout := time.Until(deadline); timeout &gt; 0 {
         // if it&#39;s not an even number of seconds, round up to the nearest second
         if truncated := timeout.Truncate(time.Second); truncated != timeout {
            timeout = truncated &#43; time.Second
         }
         // set the timeout
         r.Timeout(timeout)
      }
   }

   if err := r.Do(ctx).Into(response); err != nil {
      return &amp;webhookutil.ErrCallingWebhook{WebhookName: h.Name, Reason: err}
   }
   trace.Step(&#34;Request completed&#34;)

   result, err := webhookrequest.VerifyAdmissionResponse(uid, false, response)
   if err != nil {
      return &amp;webhookutil.ErrCallingWebhook{WebhookName: h.Name, Reason: err}
   }

   for k, v := range result.AuditAnnotations {
      key := h.Name &#43; &#34;/&#34; &#43; k
      if err := attr.Attributes.AddAnnotation(key, v); err != nil {
         klog.Warningf(&#34;Failed to set admission audit annotation %s to %s for validating webhook %s: %v&#34;, key, v, h.Name, err)
      }
   }
   if result.Allowed {
      return nil
   }
   return &amp;webhookutil.ErrWebhookRejection{Status: webhookerrors.ToStatusErr(h.Name, result.Result)}
}
```

走到这里基本上对 `admission webhook` 有了大致的了解，可以知道这个操作是由 apiserver 完成的。下面就实际操作下自定义一个webhook。

这里还有两个概念，就是请求参数 [AdmissionRequest](https://github.com/kubernetes/kubernetes/blob/6adee9d4fb7a0cf3eec148448792e2ab091e1720/staging/src/k8s.io/api/admission/v1/types.go#L40-L113) 和相应参数 [AdmissionResponse](https://github.com/kubernetes/kubernetes/blob/6adee9d4fb7a0cf3eec148448792e2ab091e1720/staging/src/k8s.io/api/admission/v1/types.go#L116-L150)，这些可以在 `callHook` 中看到，这两个参数被定义在 [k8s.io\api\admission\v1\types.go](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/api/admission/v1/types.go#L29-L37) ；这两个参数也就是我们在自定义 webhook 时需要处理接收到的body的结构，以及我们响应内容数据结构。

## 如何编写一个自定义的admission webhook

通过上面的学习了解到了，自定义的webhook就是做为kubernetes提供给用户两种admission controller来验证自定义业务的一个中间件 admission webhook。本质上他是一个HTTP Server，用户可以使用任何语言来完成这部分功能。当然，如果涉及到需要对kubernetes集群资源操作的话，还是建议使用kubernetes官方提供了SDK的编程语言来完成自定义的webhook。

那么完成一个自定义admission webhook需要两个步骤：

- 将相关的webhook config注册给kubernetes，也就是让kubernetes知道你的webhook
- 准备一个http server来处理 apiserver发过来验证的信息

&gt; 注：这里使用go net/http包，本身不区分方法处理HTTP的何种请求，如果用其他框架实现的，如django，需要指定对应方法需要为POST

### 向kubernetes注册webhook对象

kubernetes提供的两种类型可自定义的准入控制器，和其他资源一样，可以利用资源清单，动态配置那些资源要被adminssion webhook处理。 kubernetes将这种形式抽象为两种资源：

- ValidatingWebhookConfiguration

- MutatingWebhookConfiguration

####  ValidatingAdmission

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: &#34;pod-policy.example.com&#34;
webhooks:
- name: &#34;pod-policy.example.com&#34;
  rules:
  - apiGroups:   [&#34;&#34;] # 拦截资源的Group &#34;&#34; 表示 core。&#34;*&#34; 表示所有。
    apiVersions: [&#34;v1&#34;] # 拦截资源的版本
    operations:  [&#34;CREATE&#34;] # 什么请求下拦截
    resources:   [&#34;pods&#34;]  # 拦截什么资源
    scope:       &#34;Namespaced&#34; # 生效的范围，cluster还是namespace &#34;*&#34;表示没有范围限制。
  clientConfig: # 我们部署的webhook服务，
    service: # service是在cluster-in模式下
      namespace: &#34;example-namespace&#34;
      name: &#34;example-service&#34;
      port: 443 # 服务的端口
      path: &#34;/validate&#34; # path是对应用于验证的接口
    # caBundle是提供给 admission webhook CA证书  
    caBundle: &#34;Ci0tLS0tQk...&lt;base64-encoded PEM bundle containing the CA that signed the webhook&#39;s serving certificate&gt;...tLS0K&#34;
  admissionReviewVersions: [&#34;v1&#34;, &#34;v1beta1&#34;]
  sideEffects: None
  timeoutSeconds: 5 # 1-30s直接，表示请求api的超时时间
```

#### MutatingAdmission

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: &#34;valipod-policy.example.com&#34;
webhooks:
- name: &#34;valipod-policy.example.com&#34;
  rules:
    - apiGroups:   [&#34;apps&#34;] # 拦截资源的Group &#34;&#34; 表示 core。&#34;*&#34; 表示所有。
      apiVersions: [&#34;v1&#34;] # 拦截资源的版本
      operations:  [&#34;CREATE&#34;] # 什么请求下拦截
      resources:   [&#34;deployments&#34;]  # 拦截什么资源
      scope:       &#34;Namespaced&#34; # 生效的范围，cluster还是namespace &#34;*&#34;表示没有范围限制。
  clientConfig: # 我们部署的webhook服务，
    url: &#34;https://10.0.0.1:81/validate&#34; # 这里是外部模式
    #      service: # service是在cluster-in模式下
    #        namespace: &#34;default&#34;
    #        name: &#34;admission-webhook&#34;
    #        port: 81 # 服务的端口
    #        path: &#34;/mutate&#34; # path是对应用于验证的接口
    # caBundle是提供给 admission webhook CA证书
    caBundle: &#34;Ci0tLS0tQk...&lt;base64-encoded PEM bundle containing the CA that signed the webhook&#39;s serving certificate&gt;...tLS0K&#34;
  admissionReviewVersions: [&#34;v1&#34;]
  sideEffects: None
  timeoutSeconds: 5 # 1-30s直接，表示请求api的超时时间
```

&gt; 注：对于webhook，也可以引入外部的服务，并非必须部署到集群内部

对于外部服务来讲，需要 `clientConfig` 中的 `service` , 更换为 `url` ; 通过 `url` 参数可以将一个外部的服务引入

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
...
webhooks:
- name: my-webhook.example.com
  clientConfig:
    url: &#34;https://my-webhook.example.com:9443/my-webhook-path&#34;
  ...
```

&gt; 注：这里的url规则必须准守下列形式：
&gt;
&gt; -  `scheme://host:port/path` 
&gt; - 使用了url 时，这里不应填写集群内的服务
&gt; - `scheme` 必须是 https，不能为http，这就意味着，引入外部时也需要
&gt; - 配置时使用了，`?xx=xx` 的参数也是不被允许的（官方说法是这样的，通过源码学习了解到因为会发送特定的请求体，所以无需管参数）

更多的配置可以参考kubernetes官方提供的 [doc](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#webhook-configuration)

### 准备一个webhook

让我们编写我们的 webhook  server。将创建两个钩子，`/mutate` 与 `/validate`；

- `/mutate` 将在创建deployment资源时，基于版本，给资源加上注释 `webhook.example.com/allow: true`
-  `/validate` 将对 `/mutate`  增加的 `allow:true` 那么则继续，否则拒绝。

这里为了方便，全部写在一起了，实际上不符合软件的设计。在kubernetes代码库中也提供了一个[webhook server](https://github.com/kubernetes/kubernetes/blob/release-1.21/test/images/agnhost/webhook/main.go)，可以参考他这个webhook server来学习具体要做什么

```go
package main

import (
	&#34;context&#34;
	&#34;crypto/tls&#34;
	&#34;encoding/json&#34;
	&#34;fmt&#34;
	&#34;io/ioutil&#34;
	&#34;net/http&#34;
	&#34;os&#34;
	&#34;os/signal&#34;
	&#34;strings&#34;
	&#34;syscall&#34;

	v1admission &#34;k8s.io/api/admission/v1&#34;
	&#34;k8s.io/apimachinery/pkg/runtime&#34;
	&#34;k8s.io/apimachinery/pkg/runtime/serializer&#34;

	appv1 &#34;k8s.io/api/apps/v1&#34;
	metav1 &#34;k8s.io/apimachinery/pkg/apis/meta/v1&#34;
	&#34;k8s.io/klog&#34;
)

type patch struct {
	Op    string            `json:&#34;op&#34;`
	Path  string            `json:&#34;path&#34;`
	Value map[string]string `json:&#34;value&#34;`
}

func serve(w http.ResponseWriter, r *http.Request) {

	var body []byte
	if data, err := ioutil.ReadAll(r.Body); err == nil {
		body = data
	}
	klog.Infof(fmt.Sprintf(&#34;receive request: %v....&#34;, string(body)[:130]))
	if len(body) == 0 {
		klog.Error(fmt.Sprintf(&#34;admission request body is empty&#34;))
		http.Error(w, fmt.Errorf(&#34;admission request body is empty&#34;).Error(), http.StatusBadRequest)
		return
	}
	var admission v1admission.AdmissionReview
	codefc := serializer.NewCodecFactory(runtime.NewScheme())
	decoder := codefc.UniversalDeserializer()
	_, _, err := decoder.Decode(body, nil, &amp;admission)

	if err != nil {
		msg := fmt.Sprintf(&#34;Request could not be decoded: %v&#34;, err)
		klog.Error(msg)
		http.Error(w, msg, http.StatusBadRequest)
		return
	}

	if admission.Request == nil {
		klog.Error(fmt.Sprintf(&#34;admission review can&#39;t be used: Request field is nil&#34;))
		http.Error(w, fmt.Errorf(&#34;admission review can&#39;t be used: Request field is nil&#34;).Error(), http.StatusBadRequest)
		return
	}

	switch strings.Split(r.RequestURI, &#34;?&#34;)[0] {
	case &#34;/mutate&#34;:
		req := admission.Request
		var admissionResp v1admission.AdmissionReview
		admissionResp.APIVersion = admission.APIVersion
		admissionResp.Kind = admission.Kind
		klog.Infof(&#34;AdmissionReview for Kind=%v, Namespace=%v Name=%v UID=%v Operation=%v&#34;,
			req.Kind.Kind, req.Namespace, req.Name, req.UID, req.Operation)
		switch req.Kind.Kind {
		case &#34;Deployment&#34;:
			var (
				respstr []byte
				err     error
				deploy  appv1.Deployment
			)
			if err = json.Unmarshal(req.Object.Raw, &amp;deploy); err != nil {
				respStructure := v1admission.AdmissionResponse{Result: &amp;metav1.Status{
					Message: fmt.Sprintf(&#34;could not unmarshal resouces review request: %v&#34;, err),
					Code:    http.StatusInternalServerError,
				}}
				klog.Error(fmt.Sprintf(&#34;could not unmarshal resouces review request: %v&#34;, err))
				if respstr, err = json.Marshal(respStructure); err != nil {
					klog.Error(fmt.Errorf(&#34;could not unmarshal resouces review response: %v&#34;, err))
					http.Error(w, fmt.Errorf(&#34;could not unmarshal resouces review response: %v&#34;, err).Error(), http.StatusInternalServerError)
					return
				}
				http.Error(w, string(respstr), http.StatusBadRequest)
				return
			}

			current_annotations := deploy.GetAnnotations()
			pl := []patch{}
			for k, v := range current_annotations {
				pl = append(pl, patch{
					Op:   &#34;add&#34;,
					Path: &#34;/metadata/annotations&#34;,
					Value: map[string]string{
						k: v,
					},
				})
			}
			pl = append(pl, patch{
				Op:   &#34;add&#34;,
				Path: &#34;/metadata/annotations&#34;,
				Value: map[string]string{
					deploy.Name &#43; &#34;/Allow&#34;: &#34;true&#34;,
				},
			})

			annotationbyte, err := json.Marshal(pl)

			if err != nil {
				http.Error(w, err.Error(), http.StatusInternalServerError)
				return
			}
			respStructure := &amp;v1admission.AdmissionResponse{
				UID:     req.UID,
				Allowed: true,
				Patch:   annotationbyte,
				PatchType: func() *v1admission.PatchType {
					t := v1admission.PatchTypeJSONPatch
					return &amp;t
				}(),
				Result: &amp;metav1.Status{
					Message: fmt.Sprintf(&#34;could not unmarshal resouces review request: %v&#34;, err),
					Code:    http.StatusOK,
				},
			}
			admissionResp.Response = respStructure

			klog.Infof(&#34;sending response: %s....&#34;, admissionResp.Response.String()[:130])
			respByte, err := json.Marshal(admissionResp)
			if err != nil {
				klog.Errorf(&#34;Can&#39;t encode response messages: %v&#34;, err)
				http.Error(w, err.Error(), http.StatusInternalServerError)
			}
			klog.Infof(&#34;prepare to write response...&#34;)
			w.Header().Set(&#34;Content-Type&#34;, &#34;application/json&#34;)
			if _, err := w.Write(respByte); err != nil {
				klog.Errorf(&#34;Can&#39;t write response: %v&#34;, err)
				http.Error(w, fmt.Sprintf(&#34;could not write response: %v&#34;, err), http.StatusInternalServerError)
			}

		default:
			klog.Error(fmt.Sprintf(&#34;unsupport resouces review request type&#34;))
			http.Error(w, &#34;unsupport resouces review request type&#34;, http.StatusBadRequest)
		}

	case &#34;/validate&#34;:
		req := admission.Request
		var admissionResp v1admission.AdmissionReview
		admissionResp.APIVersion = admission.APIVersion
		admissionResp.Kind = admission.Kind
		klog.Infof(&#34;AdmissionReview for Kind=%v, Namespace=%v Name=%v UID=%v Operation=%v&#34;,
			req.Kind.Kind, req.Namespace, req.Name, req.UID, req.Operation)
		var (
			deploy  appv1.Deployment
			respstr []byte
		)
		switch req.Kind.Kind {
		case &#34;Deployment&#34;:
			if err = json.Unmarshal(req.Object.Raw, &amp;deploy); err != nil {
				respStructure := v1admission.AdmissionResponse{Result: &amp;metav1.Status{
					Message: fmt.Sprintf(&#34;could not unmarshal resouces review request: %v&#34;, err),
					Code:    http.StatusInternalServerError,
				}}
				klog.Error(fmt.Sprintf(&#34;could not unmarshal resouces review request: %v&#34;, err))
				if respstr, err = json.Marshal(respStructure); err != nil {
					klog.Error(fmt.Errorf(&#34;could not unmarshal resouces review response: %v&#34;, err))
					http.Error(w, fmt.Errorf(&#34;could not unmarshal resouces review response: %v&#34;, err).Error(), http.StatusInternalServerError)
					return
				}
				http.Error(w, string(respstr), http.StatusBadRequest)
				return
			}
		}
		al := deploy.GetAnnotations()
		respStructure := v1admission.AdmissionResponse{
			UID: req.UID,
		}
		if al[fmt.Sprintf(&#34;%s/Allow&#34;, deploy.Name)] == &#34;true&#34; {
			respStructure.Allowed = true
			respStructure.Result = &amp;metav1.Status{
				Code: http.StatusOK,
			}
		} else {
			respStructure.Allowed = false
			respStructure.Result = &amp;metav1.Status{
				Code: http.StatusForbidden,
				Reason: func() metav1.StatusReason {
					return metav1.StatusReasonForbidden
				}(),
				Message: fmt.Sprintf(&#34;the resource %s couldn&#39;t to allow entry.&#34;, deploy.Kind),
			}
		}

		admissionResp.Response = &amp;respStructure

		klog.Infof(&#34;sending response: %s....&#34;, admissionResp.Response.String()[:130])
		respByte, err := json.Marshal(admissionResp)
		if err != nil {
			klog.Errorf(&#34;Can&#39;t encode response messages: %v&#34;, err)
			http.Error(w, err.Error(), http.StatusInternalServerError)
		}
		klog.Infof(&#34;prepare to write response...&#34;)
		w.Header().Set(&#34;Content-Type&#34;, &#34;application/json&#34;)
		if _, err := w.Write(respByte); err != nil {
			klog.Errorf(&#34;Can&#39;t write response: %v&#34;, err)
			http.Error(w, fmt.Sprintf(&#34;could not write response: %v&#34;, err), http.StatusInternalServerError)
		}
	}
}

func main() {
	var (
		cert, key string
	)

	if cert = os.Getenv(&#34;TLS_CERT&#34;); len(cert) == 0 {
		cert = &#34;./tls/tls.crt&#34;
	}

	if key = os.Getenv(&#34;TLS_KEY&#34;); len(key) == 0 {
		key = &#34;./tls/tls.key&#34;
	}

	ca, err := tls.LoadX509KeyPair(cert, key)
	if err != nil {
		klog.Error(err.Error())
		return
	}

	server := &amp;http.Server{
		Addr: &#34;:81&#34;,
		TLSConfig: &amp;tls.Config{
			Certificates: []tls.Certificate{
				ca,
			},
		},
	}

	httpserver := http.NewServeMux()

	httpserver.HandleFunc(&#34;/validate&#34;, serve)
	httpserver.HandleFunc(&#34;/mutate&#34;, serve)
	httpserver.HandleFunc(&#34;/ping&#34;, func(w http.ResponseWriter, r *http.Request) {
		klog.Info(fmt.Sprintf(&#34;%s %s&#34;, r.RequestURI, &#34;pong&#34;))
		fmt.Fprint(w, &#34;pong&#34;)
	})
	server.Handler = httpserver

	go func() {
		if err := server.ListenAndServeTLS(&#34;&#34;, &#34;&#34;); err != nil {
			klog.Errorf(&#34;Failed to listen and serve webhook server: %v&#34;, err)
		}
	}()

	klog.Info(&#34;starting serve.&#34;)
	signalChan := make(chan os.Signal, 1)
	signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)
	&lt;-signalChan

	klog.Infof(&#34;Got shut signal, shutting...&#34;)
	if err := server.Shutdown(context.Background()); err != nil {
		klog.Errorf(&#34;HTTP server Shutdown: %v&#34;, err)
	}
}
```

对应的Dockerfile

```docker
FROM golang:alpine AS builder
MAINTAINER cylon
WORKDIR /admission
COPY ./ /admission
ENV GOPROXY https://goproxy.cn,direct
RUN \
    sed -i &#39;s/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g&#39; /etc/apk/repositories &amp;&amp; \
    apk add upx  &amp;&amp; \
    GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -ldflags &#34;-s -w&#34; -o webhook main.go &amp;&amp; \
    upx -1 webhook &amp;&amp; \
    chmod &#43;x webhook

FROM alpine AS runner
WORKDIR /go/admission
COPY --from=builder /admission/webhook .
VOLUME [&#34;/admission&#34;]
```

集群内部部署所需的资源清单

```yaml
apiVersion: v1
kind: Service
metadata:
  name: admission-webhook
  labels:
    app: admission-webhook
spec:
  ports:
    - port: 81
      targetPort: 81
  selector:
    app: simple-webhook
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: simple-webhook
  name: simple-webhook
spec:
  replicas: 1
  selector:
    matchLabels:
      app: simple-webhook
  template:
    metadata:
      labels:
        app: simple-webhook
    spec:
      containers:
        - image: cylonchau/simple-webhook:v0.0.2
          imagePullPolicy: IfNotPresent
          name: webhook
          command: [&#34;./webhook&#34;]
          env:
            - name: &#34;TLS_CERT&#34;
              value: &#34;./tls/tls.crt&#34;
            - name: &#34;TLS_KEY&#34;
              value: &#34;./tls/tls.key&#34;
            - name: NS_NAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
          ports:
            - containerPort: 81
          volumeMounts:
            - name: tlsdir
              mountPath: /go/admission/tls
              readOnly: true
      volumes:
        - name: tlsdir
          secret:
            secretName: webhook
---
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: &#34;pod-policy.example.com&#34;
webhooks:
  - name: &#34;pod-policy.example.com&#34;
    rules:
      - apiGroups:   [&#34;apps&#34;] # 拦截资源的Group &#34;&#34; 表示 core。&#34;*&#34; 表示所有。
        apiVersions: [&#34;v1&#34;] # 拦截资源的版本
        operations:  [&#34;CREATE&#34;] # 什么请求下拦截
        resources:   [&#34;deployments&#34;]  # 拦截什么资源
        scope:       &#34;Namespaced&#34; # 生效的范围，cluster还是namespace &#34;*&#34;表示没有范围限制。
    clientConfig: # 我们部署的webhook服务，
      url: &#34;https://10.0.0.1:81/mutate&#34;
#      service: # service是在cluster-in模式下
#        namespace: &#34;default&#34;
#        name: &#34;admission-webhook&#34;
#        port: 81 # 服务的端口
#        path: &#34;/mutate&#34; # path是对应用于验证的接口
      # caBundle是提供给 admission webhook CA证书
      caBundle: Put you CA (base64 encode) in here
    admissionReviewVersions: [&#34;v1&#34;]
    sideEffects: None
    timeoutSeconds: 5 # 1-30s直接，表示请求api的超时时间
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: &#34;valipod-policy.example.com&#34;
webhooks:
- name: &#34;valipod-policy.example.com&#34;
  rules:
    - apiGroups:   [&#34;apps&#34;] # 拦截资源的Group &#34;&#34; 表示 core。&#34;*&#34; 表示所有。
      apiVersions: [&#34;v1&#34;] # 拦截资源的版本
      operations:  [&#34;CREATE&#34;] # 什么请求下拦截
      resources:   [&#34;deployments&#34;]  # 拦截什么资源
      scope:       &#34;Namespaced&#34; # 生效的范围，cluster还是namespace &#34;*&#34;表示没有范围限制。
  clientConfig: # 我们部署的webhook服务，
    #      service: # service是在cluster-in模式下
    #        namespace: &#34;default&#34;
    #        name: &#34;admission-webhook&#34;
    #        port: 81 # 服务的端口
    #        path: &#34;/mutate&#34; # path是对应用于验证的接口
    # caBundle是提供给 admission webhook CA证书
    caBundle: Put you CA (base64 encode) in here
  admissionReviewVersions: [&#34;v1&#34;]
  sideEffects: None
  timeoutSeconds: 5 # 1-30s直接，表示请求api的超时时间
```

#### 这里需要主义的问题

**证书问题**

如果需要 `cluster-in` ，那么则需要对对应webhookconfig资源配置 `service` ；如果使用的是外部部署，则需要配置对应访问地址，如：*&#34;https://xxxx:port/method&#34;*

这两种方式的证书均需要对应的 `subjectAltName` ，`cluster-in` 模式 需要对应service名称，如，至少包含`serviceName.NS.svc` 这一个域名。

下面就是证书类问题的错误

```
Failed calling webhook, failing closed pod-policy.example.com: failed calling webhook &#34;pod-policy.example.com&#34;: Post https://admission-webhook.default.svc:81/mutate?timeout=5s: x509: certificate signed by unknown authority (possibly because of &#34;crypto/rsa: verification error&#34; while trying to verify candidate authority certificate &#34;admission-webhook-ca&#34;)
```

**相应信息问题**

上面我们了解到的APIServer是去发出 `v1admission.AdmissionReview` 也就是 Request 和 Response类型的，所以，为了更清晰的表示出问题所在，需要对响应格式中的 `Reason` 与 `Message`  配置，这也就是我们在客户端看到的报错信息。

```go
&amp;metav1.Status{
    Code: http.StatusForbidden,
    Reason: func() metav1.StatusReason {
        return metav1.StatusReasonForbidden
    }(),
    Message: fmt.Sprintf(&#34;the resource %s couldn&#39;t to allow entry.&#34;, deploy.Kind),
}
```

通过上面的设置用户可以看到下列错误

```bash
$ kubectl apply -f nginx.yaml 
Error from server (Forbidden): error when creating &#34;nginx.yaml&#34;: admission webhook &#34;valipod-policy.example.com&#34; denied the request: the resource Deployment couldn&#39;t to allow entry.
```

&gt; 注：必须的参数还包含，UID，allowed，这两个是必须的，上面阐述的只是对用户友好的提示信息

下面的报错就是对相应格式设置错误

```go
Error from server (InternalError): error when creating &#34;nginx.yaml&#34;: Internal error occurred: failed calling webhook &#34;pod-policy.example.com&#34;: the server rejected our request for an unknown reason
```

**相应信息版本问题**

相应信息也需要指定一个版本，这个与请求来的结构中拿即可

```go
admissionResp.APIVersion = admission.APIVersion
admissionResp.Kind = admission.Kind
```

下面是没有为对应相应信息配置对应KV的值出现的报错

```
Error from server (InternalError): error when creating &#34;nginx.yaml&#34;: Internal error occurred: failed calling webhook &#34;pod-policy.example.com&#34;: expected webhook response of admission.k8s.io/v1, Kind=AdmissionReview, got /, Kind=
```

**关于patch**

kubernetes中patch使用的是特定的规范，如 `jsonpatch`

&gt; kubernetes当前唯一支持的 `patchType` 是 `JSONPatch`。 有关更多详细信息，请参见 [JSON patch](https://jsonpatch.com/)
&gt;
&gt; 对于 `jsonpatch` 是一个固定的类型，在go中必须定义其结构体
&gt;
&gt; ```json
&gt; {
&gt; 	&#34;op&#34;: &#34;add&#34;, // 做什么操作
&gt; 	&#34;path&#34;: &#34;/spec/replicas&#34;, // 操作的路径
&gt; 	&#34;value&#34;: 3 // 对应添加的key value
&gt; }
&gt; ```

下面就是字符串类型设置为布尔型产生的报错

```bash
Error from server (InternalError): error when creating &#34;nginx.yaml&#34;: Internal error occurred: v1.Deployment.ObjectMeta: v1.ObjectMeta.Annotations: ReadString: expects &#34; or n, but found t, error found in #10 byte of ...|t/Allow&#34;:true},&#34;crea|..., bigger context ...|tadata&#34;:{&#34;annotations&#34;:{&#34;nginx-deployment/Allow&#34;:true},&#34;creationTimestamp&#34;:null,&#34;managedFields&#34;:[{&#34;m|..
```

### 准备证书

Ubuntu

```bash
touch ./demoCAindex.txt
touch ./demoCA/serial 
touch ./demoCA/crlnumber
echo 01 &gt; ./demoCA/serial
mkdir ./demoCA/newcerts

openssl genrsa -out cakey.pem 2048

openssl req -new \
	-x509 \
	-key cakey.pem \
	-out cacert.pem \
	-days 3650 \
	-subj &#34;/CN=admission webhook ca&#34;

openssl genrsa -out tls.key 2048

openssl req -new \
	-key tls.key \
	-subj &#34;/CN=admission webhook client&#34; \
	-reqexts webhook \
	-config &lt;(cat /etc/ssl/openssl.cnf \
	&lt;(printf &#34;[webhook]\nsubjectAltName=DNS: admission-webhook, DNS: admission-webhook.default.svc, DNS: admission-webhook.default.svc.cluster.local, IP:10.0.0.1,  IP:10.0.0.4&#34;)) \
	-out tls.csr

sed -i &#39;s/= match/= optional/g&#39; /etc/ssl/openssl.cnf

openssl ca \
	-in tls.csr \
	-cert cacert.pem \
	-keyfile cakey.pem \
	-out tls.crt \
	-days 300 \
	-extensions webhook \
	-extfile &lt;(cat /etc/ssl/openssl.cnf \
    &lt;(printf &#34;[webhook]\nsubjectAltName=DNS: admission-webhook, DNS: admission-webhook.default.svc, DNS: admission-webhook.default.svc.cluster.local, IP:10.0.0.1,  IP:10.0.0.4&#34;))
```

CentOS

```bash
touch /etc/pki/CA/index.txt
touch /etc/pki/CA/serial # 下一个要颁发的编号 16进制
touch /etc/pki/CA/crlnumber
echo 01 &gt; /etc/pki/CA/serial

openssl req -new \
	-x509 \
	-key cakey.pem \
	-out cacert.pem \
	-days 3650 \
	-subj &#34;/CN=admission webhook ca&#34;

openssl genrsa -out tls.key 2048

openssl req -new \
	-key tls.key \
	-subj &#34;/CN=admission webhook client&#34; \
	-reqexts webhook \
	-config &lt;(cat /etc/pki/tls/openssl.cnf \
	&lt;(printf &#34;[webhook]\nsubjectAltName=DNS: admission-webhook, DNS: admission-webhook.default.svc, DNS: admission-webhook.default.svc.cluster.local, IP:10.0.0.1,  IP:10.0.0.4&#34;)) \
	-out tls.csr

sed -i &#39;s/= match/= optional/g&#39; /etc/ssl/openssl.cnf

openssl ca \
	-in tls.csr \
	-cert cacert.pem \
	-keyfile cakey.pem \
	-out tls.crt \
	-days 300 \
	-extensions webhook \
	-extfile &lt;(cat /etc/pki/tls/openssl.cnf \
    &lt;(printf &#34;[webhook]\nsubjectAltName=DNS: admission-webhook, DNS: admission-webhook.default.svc, DNS: admission-webhook.default.svc.cluster.local, IP:10.0.0.1,  IP:10.0.0.4&#34;))
```

### 通过部署测试结果

可以看到我们自己注入的 annotation `nginx-deployment/Allow: true`，在该示例中，仅为演示过程，而不是真的策略，实际环境中可以根据情况进行定制自己的策略。

结果可以看出，当在 `mutating` 中不通过，即缺少对应的 annotation 标签 , 则 `validating` 会不允许准入

```bash
$ kubectl describe deploy nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Mon, 11 Jul 2022 20:25:16 &#43;0800
Labels:                 &lt;none&gt;
Annotations:            deployment.kubernetes.io/revision: 1
                        nginx-deployment/Allow: true
Selector:               app=nginx
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.14.2
```




&gt; Reference
&gt;
&gt; [extensible admission controllers](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
&gt;
&gt; [K8S client-go Patch example](https://developer.aliyun.com/article/703438)
&gt;
&gt; [admission controllers response](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#response)
&gt;
&gt; [a guide to kubernetes admission controllers](https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/)


