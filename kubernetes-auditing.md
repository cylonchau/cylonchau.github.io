# Understanding kubernetes 4A - Auditing


## Overview 

> 本文是关于Kubernetes 4A解析的第四章
>
> - [深入理解Kubernetes 4A - Authentication源码解析](https://cylonchau.github.io/kubernetes-authentication.html)
> - [深入理解Kubernetes 4A - Authorization源码解析](https://cylonchau.github.io/kubernetes-authorization.html)
> - [深入理解Kubernetes 4A - Admission Control源码解析](https://cylonchau.github.io/ch3.7-admission-webhook.html)
> - 深入理解Kubernetes 4A - Audit源码解析
>
> 所有关于Kubernetes 4A四部分代码上传至仓库 github.com/cylonchau/hello-k8s-4A

审计是信息系统中非常重要的一部分，Kubernetes 1.11中也增加了审计 (***Auditing***) 功能，通过审计功能获得 deployment, ns,等资源操作的事件。

**objective**：

- 从设计角度了解Auditing在kubernets中是如何实现的
- 了解kubernetes auditing webhook
- 完成实验，通过webhook来收集审计日志

如有错别字或理解错误地方请多多担待，代码是以1.24进行整理，实验是以1.19环境进行，差别不大。

## Kubernetes Auditing

根据Kubernetes官方描述审计在kubernetes中是有控制平面 *kube-apiserver* 中产生的一个事件，记录了集群中所操作的资源，审计围绕下列几个维度来记录事件的：

- 发生了什么
- 发生的事件
- 谁触发的
- 发生动作的对象
- 在哪里检查到动作的
- 从哪触发的
- 处理行为是什么

审计生命周期开始于组件 *kube-apiserver* 准入控制阶段，在每个阶段内都会产生审计事件并经过预处理后写入后端，目前后端包含webhook与日志文件。

> 审计日志功能增加了 *kube-apiserver* 的内存消耗，因为会为每个请求存储了审计所需的上下文。内存的消耗取决于审计日志配置 <sup><a href="#1">[1]</a></sup>。

## 审计事件设计

审计的schema不同于资源API的设计，没有 `metav1.ObjectMeta` 属性，Event是一个事件的结构体，Policy是事件配置，属于kubernetes资源，在代码 [k8s.io/apiserver/pkg/apis/audit/types.go](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/apis/audit/types.go#L79-L148) 可以看到

```go
type Event struct {
	metav1.TypeMeta `json:",inline"`
	Level Level `json:"level" protobuf:"bytes,1,opt,name=level,casttype=Level"
	AuditID types.UID `json:"auditID" protobuf:"bytes,2,opt,name=auditID,casttype=k8s.io/apimachinery/pkg/types.UID"`
	
	Stage Stage `json:"stage" protobuf:"bytes,3,opt,name=stage,casttype=Stage"`
	RequestURI string `json:"requestURI" protobuf:"bytes,4,opt,name=requestURI"`
	Verb string `json:"verb" protobuf:"bytes,5,opt,name=verb"`
	User authnv1.UserInfo `json:"user" protobuf:"bytes,6,opt,name=user"`
	ImpersonatedUser *authnv1.UserInfo `json:"impersonatedUser,omitempty" protobuf:"bytes,7,opt,name=impersonatedUser"`
	SourceIPs []string `json:"sourceIPs,omitempty" protobuf:"bytes,8,rep,name=sourceIPs"`
	UserAgent string `json:"userAgent,omitempty" protobuf:"bytes,16,opt,name=userAgent"`
	ObjectRef *ObjectReference `json:"objectRef,omitempty" protobuf:"bytes,9,opt,name=objectRef"`
	// +optional
	ResponseStatus *metav1.Status `json:"responseStatus,omitempty" protobuf:"bytes,10,opt,name=responseStatus"`

...
	
}
```

对于记录的认证事件来说，会根据请求阶段记录审计的阶段，主要分为下属集中情况，每个请求会记录其中一个验证阶段，如代码所示 <sup><a href="#1">[1]</a></sup>

```go
const (
    // 这个阶段是audit handler收到请求后立即生成事件的阶段，然后委托handler chain处理。
	StageRequestReceived Stage = "RequestReceived"
    // 这个阶段阶段仅对长时间运行的请求如 watch
	// 将在发送响应标头后，响应正文之前生成的阶段
	StageResponseStarted Stage = "ResponseStarted"
    // 这个阶段是发送相应体后的事件。
	StageResponseComplete Stage = "ResponseComplete"
	// 如果程序出现panic，则触发这个阶段
	StagePanic Stage = "Panic"
)
```

## 审计工作流程

审计真正工作的地方在 [k8s.io/apiserver/pkg/endpoints/filters/audit.go.WithAudit](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/endpoints/filters/audit.go#L42-L119) 函数，下面对与官方文档说明与这个实际代码进行结合

```go
func WithAudit(handler http.Handler, sink audit.Sink, policy audit.PolicyRuleEvaluator, longRunningCheck request.LongRunningRequestCheck) http.Handler {
    // sink是一个backend（webhook 或 日志），policy则是自定义的事件配置
    // 如果两者之一未配置，则不会使用审计功能
	if sink == nil || policy == nil {
		return handler
	}
	return http.HandlerFunc(func(w http.ResponseWriter, req *http.Request) {
        // 通过给定的配置与请求构建出一个事件 context
        // 这里可以看到
		auditContext, err := evaluatePolicyAndCreateAuditEvent(req, policy)
		if err != nil {
			utilruntime.HandleError(fmt.Errorf("failed to create audit event: %v", err))
			responsewriters.InternalError(w, req, errors.New("failed to create audit event"))
			return
		}
		// 下面代码可以看出是对事件context进行构建，与拿到来自请求Context
		ev := auditContext.Event
		if ev == nil || req.Context() == nil {
			handler.ServeHTTP(w, req)
			return
		}

		req = req.WithContext(audit.WithAuditContext(req.Context(), auditContext))

		ctx := req.Context()
		omitStages := auditContext.RequestAuditConfig.OmitStages
		
        // 这里到StageRequestReceived阶段，如果是收到请求阶段则通过注入的后端进行处理
		ev.Stage = auditinternal.StageRequestReceived
		if processed := processAuditEvent(ctx, sink, ev, omitStages); !processed {
            audit.ApiserverAuditDroppedCounter.WithContext(ctx).Inc()
			responsewriters.InternalError(w, req, errors.New("failed to store audit event"))
			return
		}

		// 拦截watch类长请求的状态码
		var longRunningSink audit.Sink
		if longRunningCheck != nil {
			ri, _ := request.RequestInfoFrom(ctx)
			if longRunningCheck(req, ri) {
				longRunningSink = sink
			}
		}
		respWriter := decorateResponseWriter(ctx, w, ev, longRunningSink, omitStages)

		// send audit event when we leave this func, either via a panic or cleanly. In the case of long
		// running requests, this will be the second audit event.
        // 在离开函数前会处理 ResponseStarted、ResponseComplete、Panic这三个阶段
		defer func() {
			if r := recover(); r != nil {
				defer panic(r)
                // 当前发生panic的请求
				ev.Stage = auditinternal.StagePanic
				ev.ResponseStatus = &metav1.Status{
					Code:    http.StatusInternalServerError,
					Status:  metav1.StatusFailure,
					Reason:  metav1.StatusReasonInternalError,
					Message: fmt.Sprintf("APIServer panic'd: %v", r),
				}
				processAuditEvent(ctx, sink, ev, omitStages)
				return
			}

			// if no StageResponseStarted event was sent b/c neither a status code nor a body was sent, fake it here
			// But Audit-Id http header will only be sent when http.ResponseWriter.WriteHeader is called.
			fakedSuccessStatus := &metav1.Status{
				Code:    http.StatusOK,
				Status:  metav1.StatusSuccess,
				Message: "Connection closed early",
			}
			if ev.ResponseStatus == nil && longRunningSink != nil {
				ev.ResponseStatus = fakedSuccessStatus
				ev.Stage = auditinternal.StageResponseStarted
				processAuditEvent(ctx, longRunningSink, ev, omitStages)
			}
			// ResponseStarted 在响应头发送后，响应体发送前的事件。watch会触发他
            
			ev.Stage = auditinternal.StageResponseComplete
			if ev.ResponseStatus == nil {
                // 没有相应状态 正是上面构造的fakedSuccessStatus
				ev.ResponseStatus = fakedSuccessStatus
			}
            // 将事件发送到后端
			processAuditEvent(ctx, sink, ev, omitStages)
		}()
		handler.ServeHTTP(respWriter, req)
	})
}
```

- 在评估请求时，会调用 `GetAuthorizerAttributes(ctx)` ，这里通过授权记录然后来通过给定的审计配置来

- 当在将事件发送到后端时，使用 `processAuditEvent()` 函数，最终修改时间后会转交至后端函数，例如webhook，会请求后端配置的webhook url的客户端，最终被执行 `return sink.ProcessEvents(ev)`

    ```go
    func (b *backend) processEvents(ev ...*auditinternal.Event) error {
    	var list auditinternal.EventList
    	for _, e := range ev {
    		list.Items = append(list.Items, *e)
    	}
    	return b.w.WithExponentialBackoff(context.Background(), func() rest.Result {
    		trace := utiltrace.New("Call Audit Events webhook",
    			utiltrace.Field{"name", b.name},
    			utiltrace.Field{"event-count", len(list.Items)})
    		// Only log audit webhook traces that exceed a 25ms per object limit plus a 50ms request overhead allowance. The high per object limit used here is primarily to allow enough time for the serialization/deserialization of audit events, which contain nested request and response objects plus additional event fields.
    		defer trace.LogIfLong(time.Duration(50+25*len(list.Items)) * time.Millisecond)
    		return b.w.RestClient.Post().Body(&list).Do(context.TODO())
    	}).Error()
    }
    ```

在 [k8s.io/apiserver/pkg/endpoints/filters/audit.go.evaluatePolicyAndCreateAuditEvent](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/endpoints/filters/audit.go#L125-L155) 会评估请求的级别和规则，而 [k8s.io/apiserver/pkg/audit/policy/checker.go](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/audit/policy/checker.go#L64-L84)

```go
func (p *policyRuleEvaluator) EvaluatePolicyRule(attrs authorizer.Attributes) auditinternal.RequestAuditConfigWithLevel {
	for _, rule := range p.Rules {
        // 评估则是评估用户与用户组，verb，ns,非API资源 /metrics /healthz
		if ruleMatches(&rule, attrs) {
            // 通过后，则将这条规则与配置返回
			return auditinternal.RequestAuditConfigWithLevel{
				Level: rule.Level,
				RequestAuditConfig: auditinternal.RequestAuditConfig{
					OmitStages:        rule.OmitStages,
					OmitManagedFields: isOmitManagedFields(&rule, p.OmitManagedFields),
				},
			}
		}
	}
	// 如果条件都不满足，则构建一个
	return auditinternal.RequestAuditConfigWithLevel{
		Level: DefaultAuditLevel,
		RequestAuditConfig: auditinternal.RequestAuditConfig{
			OmitStages:        p.OmitStages,
			OmitManagedFields: p.OmitManagedFields,
		},
	}
}
```

[k8s.io/apiserver/pkg/audit/request.go.NewEventFromRequest](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/audit/request.go#L48-L93) 创建出审计事件对象被上面 [evaluatePolicyAndCreateAuditEvent](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/endpoints/filters/audit.go#L125-L155) 返回

```go
// evaluatePolicyAndCreateAuditEvent is responsible for evaluating the audit
// policy configuration applicable to the request and create a new audit
// event that will be written to the API audit log.
// - error if anything bad happened
func evaluatePolicyAndCreateAuditEvent(req *http.Request, policy audit.PolicyRuleEvaluator) (*audit.AuditContext, error) {
	ctx := req.Context()

	attribs, err := GetAuthorizerAttributes(ctx)
	if err != nil {
		return nil, fmt.Errorf("failed to GetAuthorizerAttributes: %v", err)
	}

	ls := policy.EvaluatePolicyRule(attribs)
	audit.ObservePolicyLevel(ctx, ls.Level)
	if ls.Level == auditinternal.LevelNone {
		// Don't audit.
		return &audit.AuditContext{
			RequestAuditConfig: ls.RequestAuditConfig,
		}, nil
	}

	requestReceivedTimestamp, ok := request.ReceivedTimestampFrom(ctx)
	if !ok {
		requestReceivedTimestamp = time.Now()
	}
	ev, err := audit.NewEventFromRequest(req, requestReceivedTimestamp, ls.Level, attribs)
	if err != nil {
		return nil, fmt.Errorf("failed to complete audit event from request: %v", err)
	}

	return &audit.AuditContext{
		RequestAuditConfig: ls.RequestAuditConfig,
		Event:              ev,
	}, nil
}
```

到这里，已经清楚的了解到，Kubernetes审计工作与什么位置了，而对于Kubernetes准入给出的登录（***Authentication***），授权 (***Authorization***) 与 准入控制 (***Admission control***) 三个阶段来说，Audition 位于授权之后，正如下图所示，而这个真正的流程在kubernetes中有个属于叫 ***handler chain*** 整个链条中，准入与审计只是其中一部分。

![image-20221128001225515](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221128001225515.png)

<center>图：Kubernetes 4A 的 handler chain</center><br>

由图再结合代码可以看出，所有的客户端访问API都需要经过完整经由整个链条，而 Auditing 事件的构建是需要获取经由验证过的用户等资源构建出的事件，首次发生的为 `StageRequestReceived` ，这将在收到请求后执行，而由代码又可知，因为在最终结束掉整个请求时会执行 `WithAudit` 函数，这就为 `StageResponseComplete` 与 `StageResponseStarted` 这两个阶段被执行，而这个将发生在被注册的 handler 完成后，也就是  ***Admission control*** 后因为 AC 是在每个真实REST中被执行。TODO

## 审计策略级别 <sup><a href="#2">[2]</a></sup>

审计策略级别是控制审计记录将记录那些对象的数据内容，当事件被处理时，会按照配置的审计规则进行比较。而使用该功能需要 *kube-apiserver* 开启参数 `--audit-policy-file` 指定对应的配置，如果未指定则默认不记录任何事件，可供定义的级别有四个，被定义在 k8s.io/apiserver/pkg/apis/audit/v1/types.go 中

```go
const (
	// LevelNone disables auditing
	LevelNone Level = "None"
	// LevelMetadata provides the basic level of auditing.
	LevelMetadata Level = "Metadata"
	// LevelRequest provides Metadata level of auditing, and additionally
	// logs the request object (does not apply for non-resource requests).
	LevelRequest Level = "Request"
	// LevelRequestResponse provides Request level of auditing, and additionally
	// logs the response object (does not apply for non-resource requests).
	LevelRequestResponse Level = "RequestResponse"
)
```

- **None**： 不记录符合该规则的事件
- **Metadata**：只记录请求元数据（如User, timestamp, resources, verb），不记录请求和响应体。
- **Request**：记录事件元数据和请求体，不记录响应体。
- **RequestResponse**： 记录事件元数据，请求和响应体

下面是Kubernetes官网给出的 Policy 的配置 <sup><a href="#2">[2]</a></sup>

```yaml
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
# omitStages 代表忽略该阶段所有请求事件
# RequestReceived 这里配置的指在RequestReceived阶段忽略所有请求事件
omitStages:
  - "RequestReceived"
rules:
  # 记录将以RequestResponse级别的格式记录pod更改
  - level: RequestResponse
    resources:
    - group: ""
      # 这里资源的配置必须与RBAC配置的一致，pods将不支持pods/log这类子资源
      resources: ["pods"]
  # 如果需要配置子资源按照下列方式
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # 不记录的资源为controller-leader的configmaps资源的请求
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # 不记录用户为 "system:kube-proxy" 发起的对 endpoints与services资源的watch请求事件
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # 每个登录成功的用户，都会被追加一个用户组为 "system:authenticated"
  # 下述规则为不记录包含非资源类型的URL的已认证请求
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # 记录kube-system名称空间configmap更改事件的请求体与元数据
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # 事件将记录所有名称空间内对于configmap与secret资源改变的元数据
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # 记录对group为core与extensions下的资源类型请求的 请求体与元数据（request级别）
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # 这种属于泛规则，会记录所有上述其他之外的所有类型请求的元数据
  # 类似于授权，小权限在前，* 最后
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"
```

## Backend <sup><a href="#3">[3]</a></sup>

kubernetes目前为Auditing 提供了两个后端，日志方式与webhook方式，kubernetes审计事件会遵循 `audit.k8s.io` 结构写入到后端。

### 日志模式配置

启用日志模式只需要配置几个参数 <sup><a href="#4">[4]</a></sup>

- `--audit-log-path` 写入审计事件的日志路径。这个是必须配置的否则默认输出到STDOUT
- `--audit-log-maxage`  审计日志文件保留的最大天数
- `--audit-log-maxbackup` 审计日志保留的的最大数量
- `--audit-log-maxsize` 审计日志文件最大大小（单位M）大于会切割

例如配置

```bash
--audit-policy-file=/etc/kubernetes/audit-policy.yaml \
--audit-log-path=/var/log/kubernetes/audit.log \
--audit-log-maxsize=20M
```

### webhook <sup><a href="#5">[5]</a></sup>

webhook是指审计事件将由 *kube-apiserver* 发送到webhook服务中记录，开启webhook只需要配置 `--audit-webhook-config-file` 与 `--audit-policy-file` 两个参数，而其他的则是对该模式的辅助

- `--audit-webhook-config-file` ：webhook的配置文件，格式是kubeconfig类型，所有的信息不是kubernetes api配置，而是webhook相关信息
- `--audit-webhook-initial-backoff ` ：第一次失败后重试事件，随后仍失败后将以指数方式退避重试

- `--audit-webhook-mode` ：发送至webhook的模式。 *batch*, *blocking*, *blocking-strict* 。

例如配置

```yaml
--audit-policy-file=/etc/kubernetes/audit-policy.yaml \
--audit-webhook-config-file=/etc/kubernetes/auth/audit-webhook.yaml \
--audit-webhook-mode=batch \
```

对于initialBackoff 的退避重试则如代码所示 [k8s.io/apiserver/pkg/server/options/audit.go](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/server/options/audit.go#L584-L594)

```go
func (o *AuditWebhookOptions) newUntruncatedBackend(customDial utilnet.DialFunc) (audit.Backend, error) {
	groupVersion, _ := schema.ParseGroupVersion(o.GroupVersionString)
	webhook, err := pluginwebhook.NewBackend(o.ConfigFile, groupVersion, webhook.DefaultRetryBackoffWithInitialDelay(o.InitialBackoff), customDial)
	if err != nil {
		return nil, fmt.Errorf("initializing audit webhook: %v", err)
	}
	webhook = o.BatchOptions.wrapBackend(webhook)
	return webhook, nil
}
```

在函数 [k8s.io/apiserver/pkg/util/webhook/webhook.go.DefaultRetryBackoffWithInitialDelay](https://github.com/kubernetes/kubernetes/blob/e72eea92396503521f1acaab60a2975c13ffc618/staging/src/k8s.io/apiserver/pkg/util/webhook/webhook.go#L42-L49) 中看到 通过 wait.Backoff 进行的

```go
return wait.Backoff{
    // 时间间隔，用于调用 Step 方法时返回的时间间隔
    Duration: initialBackoffDelay,  
   	
    // 用于计算下次的时间间隔，不能为负数
    // Factor 大于 0 时，Backoff 在计算下次的时间间隔时都会根据 
    // Duration * Factor，Factor * Duration 不能大于 Cap
    Factor:   1.5,
    
    // 抖动，Jitter > 0 时，每次迭代的时间间隔都会额外加上 0 - Duration * Jitter 的随机时间,
    // 并且抖动出的时间不会设置为 Duration，而且不受 Caps 的限制
    Jitter:   0.2,
    
    // 进行指数回退(*Factor) 操作的次数
    // 当 Factor * Duration > Cap 时 Steps 会被设置为 0, Duration 设置为 Cap
    // 也就是说后续的迭代时间间隔都会返回 Duration
    Steps:    5,
    
    // 还有一个cap（Cap time.Duration），是最大的时间间隔
}
```

### 批处理

日志后端与webhook后端都支持批处理模式，默认值为webhook默认开启batch，而log则被禁用

- `--audit-log-mode/--audit-webhook-mode` ：参数通过将webhook替换为log则为对应的 batch 模式的参数，可以通过 `kube-apiserver --help|grep "audit"|grep batch` 查看
    - `batch` 默认值，缓冲事件进行异步批量处理
        - `--audit-webhook-batch-buffer-size`：批处理之前要缓冲的事件数。如果传入事件的溢出，则被丢弃。
        - `--audit-webhook-batch-max-size`：定义每一批中的最大事件数
        - `--audit-webhook-batch-max-wait`：批处理队列未满时等待的事件，到时强制写入一次
        - `--audit-webhook-batch-throttle-qps`：定义每秒最大平均批次
        - `--audit-webhook-batch-throttle-burst`：如果之前还没使用throttle-qps之前，发送的最大批数，通常情况下为第一次启动时生效的参数
    - `blocking` 阻止 apiserver 处理每个单独事件
    - `blocking-strict`：与 *blocking* 相同，但当 *RequestReceived* 阶段的审计日志记录失败时，对 kube-apiserver 的整个请求都将失败

## 参数调整

适当的调整参数与策略可以有效适应 *kuber-apiserver* 的负载，如在记录日志时应只记录所需的事件，而不是所有的事件，这样可以避免 APIServer不必要开销，例如：

- 每个请求存在多个阶段，而审计时其实不关心响应等信息，可以只记录 `RequestReceived` 的 `metadata` 级别。
- "pods/log", "pods/status" 在记录时应该区分子资源类型，而不要直接写 *`pods`* 或 *`pods/*`*
- kubernetes系统组件内的事件如果没有特殊要求可以不记录
- 对于资源类型，如configmap的请求其实没必要记录
- 审计记录应严格按照外部用户记录，而不是所有请求

如何适配APIServer的负载能力，正如官方给的示例一样，如果 *kube-apiserver* 每秒收到100个请求，而记录事件为 `ResponseStarted` 和`ResponseComplete` 阶段，此时会记录的条数约 200/s ，如果batch缓冲区为100，那么需要配置的参数至少2Qps/s。再假设后端处理能力为5秒，那么缓冲区需要配置的大小至少为5秒的事件，即1000条evnet，10个batch。正如下图所示：

![img](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/624219-20220802143739996-2062420228.png)



<center>图：审计批处理参数调优结构图</center>
<center><em>Source：</em>https://www.cnblogs.com/zhangmingcheng/p/16539514.html</center><br>

而kube-apiserver提供了两个Prometheus指标可以用于监控审计子系统的状态

- `apiserver_audit_event_total` 审计事件的总数
- `apiserver_audit_error_total` 由于错误而被丢弃的审计事件总数，例如panic类型事件

## 实验：Audit Webhook

编写一个webhook，用于处理接收到的日志，这里直接打印

```go
func serveAudit(w http.ResponseWriter, r *http.Request) {
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		httpError(w, err)
		return
	}

	var eventList audit.EventList
	err = json.Unmarshal(b, &eventList)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}
	for _, event := range eventList.Items {

		// here is your logic

		fmt.Printf("审计ID %s: 用户<%s>, 请求对象<%s>, 操作<%s>, 请求阶段<%s>\n",
			event.AuditID,
			event.User.UID,
			event.RequestURI,
			event.Verb,
			event.Stage,
		)
	}
	w.WriteHeader(http.StatusOK)
}
```

当使用命令执行查看Pod的操作时，会看到webhook收到的下述审计日志

操作命令

```bash
for n in `seq 1 100`; do kubectl get pod --user=admin; done
```

审计日志

```log
审计ID c0313416-f950-4361-9823-7c4792b143fd: 用户<admin>, 请求对象</api/v1/namespaces/default/pods?limit=500>, 操作<list
>, 请求阶段<ResponseComplete>
审计ID db2390c1-83cf-42e7-b589-70cd04003d0e: 用户<admin>, 请求对象</api/v1/namespaces/default/pods?limit=500>, 操作<list
>, 请求阶段<ResponseComplete>
审计ID a8fc2ff9-d0c5-4263-901c-b5974fd58026: 用户<admin>, 请求对象</api/v1/namespaces/default/pods?limit=500>, 操作<list
>, 请求阶段<ResponseComplete>
```

## 总结

kubernetes通过插件的方式提供了提供了一个IT系统 4A模型为集群提供了安全保障，与传统的4A (***Authentication***, ***Authorization***, ***Accounting***, ***Auditing***) 不同的是，对于 *Accounting* 与 *Authentication* 在kubernetes中设计来说 Kubernetes没有用户的实现而是一个抽象，这使得Kubernetes可以更灵活使用任意的用户系统完成登录（OID, X.509, webhook, proxy, SA....），而对于授权来说，Kubernetes 通过多种授权模型(RBAC, ABAC, Node, Webhook)，为集群提供了灵活的权限；而不同的是，通过 ***Admission Control*** 可以为集群提供更多的安全策略，例如镜像策略，通过三方提供的控制器来自定义更多的安全策略，如OPA。而这种设计为Kubernetes集群提供了一种更灵活的安全。

> **Reference**
>
> <sup id="1">[1]</sup> [***Auditing***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
>
> <sup id="2">[2]</sup> [***Audit policy***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#audit-policy)
>
> <sup id="3">[3]</sup> [***Audit backends***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#audit-backends)
>
> <sup id="4">[4]</sup> [***Log backend***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#log-backend)
>
> <sup id="5">[5]</sup> [***Webhook backend***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#webhook-backend)
>
> <sup id="6">[6]</sup> [***Event batching***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#batching) 
>
> <sup id="7">[7]</sup> [***Parameter tuning***](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#parameter-tuning)
>
> <sup id="8">[8]</sup> [***Privilege Management Infrastructure***](https://ldapwiki.com/wiki/Privilege%20Management%20Infrastructure)
>
> <sup id="9">[9]</sup> [***Kubernetes 审计（Auditing）功能详解***](https://www.cnblogs.com/zhangmingcheng/p/16539514.html)
>
> <sup id="10">[10]</sup> [***kubernetes 审计日志功能***](https://blog.tianfeiyu.com/2019/01/30/k8s-audit-webhook/)
