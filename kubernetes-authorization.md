# Understanding kubernetes 4A - Authorization


## Overview

> 本文是关于Kubernetes 4A解析的第二章
>
> - [深入理解Kubernetes 4A - Authentication源码解析](https://cylonchau.github.io/kubernetes-authentication.html)
> - 深入理解Kubernetes 4A - Authorization源码解析
> - [深入理解Kubernetes 4A - Admission Control源码解析](https://cylonchau.github.io/ch3.7-admission-webhook.html)
> - [深入理解Kubernetes 4A - Audit源码解析](https://cylonchau.github.io/kubernetes-auditing.html)
>
> 所有关于Kubernetes 4A部分代码上传至仓库 github.com/cylonchau/hello-k8s-4A

在 Kubernetes 中，当一个访问请求通过了登录阶段（***Authentication***），必须还需要请求拥有该对象的访问权限，而授权部分也是Kubernetes API 访问控制中的第二个部分 ***Authorization*** .

***Authorization*** 在 Kubernetes中是以评估发起请求的用户，根据其身份特性评估这次请求是被 ”拒绝“ 还是 “允许”，同访问控制三部曲中其他两个插件 (***Authentication***, ***Adminssion Control***) 一样，***Authorization*** 也可以同时配置多个，当收到用户的请求时，会依次检查这个阶段配置的所有模块，如果任何一个模块对该请求授予权限（拒绝或允许），那么该阶段会直接返回，当所有模块都没有该用户所属的权限时，默认是拒绝，在Kubernetes中，被该插件拒绝的用户显示为HTTP 403。

如有错别字或理解错误地方请多多担待，代码是以1.24进行整理，实验是以1.19环境进行，差别不大

**objective**：

- 了解kubernetes Authorization机制
- 了解授权系统的设计
- 完成实验，使用 OPA 作为 Kubernetes 外部用户，权限认证模型 *RBAC* 的替代品

## Kubernetes是如何对用户授权的

kubernetes对用户授权需要遵守的shema必须拥有下列属性，代码位于[pkg\apis\authorization\types.go](https://github.com/kubernetes/kubernetes/blob/57eb5d631ccd615cd161b6da36afc759af004b93/pkg/apis/authorization/types.go#L27-L36)

```go
type SubjectAccessReview struct {
    // API必须实现的部分
	metav1.TypeMeta
	metav1.ObjectMeta
	// 请求需要遵守的属性
	Spec SubjectAccessReviewSpec
	// 请求被授权的状态
	Status SubjectAccessReviewStatus
}
```

这里可以看到数据模型是

```go
type SubjectAccessReviewSpec struct {
	// ResourceAttributes describes information for a resource access request
	ResourceAttributes *ResourceAttributes
	// NonResourceAttributes describes information for a non-resource access request
	NonResourceAttributes *NonResourceAttributes

	// 请求的用户，必填
    // 如果只传递 User，而没有Group，那么权限必须与用户对应，例如rolebinding/clusterrolebing
    // 如果传递了User与Group，那么rolebinding/clusterrolebing权限最大为Group，最小为User
	User string
	// Groups是用户所属组，可以有多个
	Groups []string
	// Extra corresponds to the user.Info.GetExtra() method from the authenticator.  Since that is input to the authorizer
	// 这里通常对于验证和授权阶段，没有特别的需求
	Extra map[string]ExtraValue
	// UID 请求用户的UID，通常来说与User相同，Authentication中也是这么做的
	UID string
}
```

由此可得知，在授权部分，kubernetes要求请求必须存在

- **用户类属性**：**user**，**group** ，**extra**  由 ***Authentication*** 提供的用户信息
- **请求类属性**：
    - **API资源**： `curl $API_SERVER_URL/api/v1/namespaces`
    - **请求路径**： 非API资源格式的路径，`/api`，`/healthz`
    - **verb**：HTTP请求方法，GET，POST..
- 资源类属性：
    - 访问的资源的名称或ID，如Pod名
    - 要访问的名称空间
    - 资源所属组，Kubernetes资源有GVR组成

 那么，`SubjectAccessReview.Spec` 为要审查的对象，`SubjectAccessReview.Status` 为审查结果，通常在每个请求到来时，入库前必定被审查

## Kubernetes中的授权模式

知道授权的对象，就需要知道如何对该对象进行授权，Kubernetes authorizer 提供了下列授权模式

[pkg/kubeapiserver/authorizer/modes/modes.go](pkg/kubeapiserver/authorizer/modes/modes.go)

```go
const (
	// ModeAlwaysAllow is the mode to set all requests as authorized
	ModeAlwaysAllow string = "AlwaysAllow"
	// ModeAlwaysDeny is the mode to set no requests as authorized
	ModeAlwaysDeny string = "AlwaysDeny"
	// ModeABAC is the mode to use Attribute Based Access Control to authorize
	ModeABAC string = "ABAC"
	// ModeWebhook is the mode to make an external webhook call to authorize
	ModeWebhook string = "Webhook"
	// ModeRBAC is the mode to use Role Based Access Control to authorize
	ModeRBAC string = "RBAC"
	// ModeNode is an authorization mode that authorizes API requests made by kubelets.
	ModeNode string = "Node"
)
```

可以看出，大致遵循模式进行授权

- ModeABAC (***Attribute-based access control***)：是一种将属性分组，而后属性组分配给用户的模型，通常情况下这种模型很少使用
- ModeRBAC (***Role Based Access Control***) ：是kubernetes主流的授权模型，是将用户分组，将属性分配给用户组的一种模型
- ModeNode：对kubelet授权的方式
- ModeWebhook：用户注入给Kubernetes 授权插件进行回调的一种授权模式

## Kubernetes 授权生命周期

在启动 `kube-apiserver` 是都会初始化被注入一个 `Authorizer` 而这个被上面模式进行实现，如 `RBACAuthorizer` ,  `WebhookAuthorizer` [k8s.io/apiserver/pkg/authorization/authorizer/interfaces.go](https://github.com/kubernetes/kubernetes/blob/release-1.24/staging/src/k8s.io/apiserver/pkg/authorization/authorizer/interfaces.go)

```go
type Authorizer interface {
	Authorize(ctx context.Context, a Attributes) (authorized Decision, reason string, err error)
}
```

在 Run 中会创建一个CreateServerChain，这里面可以看到对应注册进来的  `Authorizer `  [k8s.io\kubernetes\cmd\kube-apiserver\app\server.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/cmd/kube-apiserver/app/server.go#L155-L173)

```go
// Run runs the specified APIServer.  This should never exit.
func Run(completeOptions completedServerRunOptions, stopCh <-chan struct{}) error {
	// To help debugging, immediately log version
	klog.Infof("Version: %+v", version.Get())

	klog.InfoS("Golang settings", "GOGC", os.Getenv("GOGC"), "GOMAXPROCS", os.Getenv("GOMAXPROCS"), "GOTRACEBACK", os.Getenv("GOTRACEBACK"))

	server, err := CreateServerChain(completeOptions)
	if err != nil {
		return err
	}

	prepared, err := server.PrepareRun()
	if err != nil {
		return err
	}

	return prepared.Run(stopCh)
}
```

可以看到在创建这个 `Authorizer `  时会调用一个 `BuildAuthorizer` 构建这个 `Authorizer `  

[k8s.io/kubernetes/cmd/kube-apiserver/app/server.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/cmd/kube-apiserver/app/server.go#L448)

```go
func buildGenericConfig(
	s *options.ServerRunOptions,
	proxyTransport *http.Transport,
) (
	genericConfig *genericapiserver.Config,
	versionedInformers clientgoinformers.SharedInformerFactory,
	serviceResolver aggregatorapiserver.ServiceResolver,
	pluginInitializers []admission.PluginInitializer,
	admissionPostStartHook genericapiserver.PostStartHookFunc,
	storageFactory *serverstorage.DefaultStorageFactory,
	lastErr error,
) {
    
	...

	genericConfig.Authorization.Authorizer, genericConfig.RuleResolver, err = BuildAuthorizer(s, genericConfig.EgressSelector, versionedInformers)
	if err != nil {
		lastErr = fmt.Errorf("invalid authorization config: %v", err)
		return
	}
	...
}
```

在代码 `BuildAuthorizer` 中构建了这个 `Authorizer` 其中可以看到 s 为 `kube-apiserver` 对于授权阶段的参数，例如参数，使用哪些模式 `--authorization-mode`，使用的webhook的配置 `--authentication-token-webhook-config-file` 等，通过传入的参数来决定这些

```go
// BuildAuthorizer constructs the authorizer
func BuildAuthorizer(s *options.ServerRunOptions, EgressSelector *egressselector.EgressSelector, versionedInformers clientgoinformers.SharedInformerFactory) (authorizer.Authorizer, authorizer.RuleResolver, error) {
   // 这里构建出  authorizer.Config
	authorizationConfig := s.Authorization.ToAuthorizationConfig(versionedInformers)

	if EgressSelector != nil {
		egressDialer, err := EgressSelector.Lookup(egressselector.ControlPlane.AsNetworkContext())
		if err != nil {
			return nil, nil, err
		}
		authorizationConfig.CustomDial = egressDialer
	}
	
    // 然后返回你开启的每一个webhook的模式的 authorizer
	return authorizationConfig.New()
}
```

而对应这部分的数据结构如下所示  [k8s.io/pkg/kubeapiserver/options/authorization.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/pkg/kubeapiserver/options/authorization.go#L34-L46)

```go
// BuiltInAuthorizationOptions contains all build-in authorization options for API Server
type BuiltInAuthorizationOptions struct {
	Modes                       []string
	PolicyFile                  string
	WebhookConfigFile           string
	WebhookVersion              string
	WebhookCacheAuthorizedTTL   time.Duration
	WebhookCacheUnauthorizedTTL time.Duration
	// WebhookRetryBackoff specifies the backoff parameters for the authorization webhook retry logic.
	// This allows us to configure the sleep time at each iteration and the maximum number of retries allowed
	// before we fail the webhook call in order to limit the fan out that ensues when the system is degraded.
	WebhookRetryBackoff *wait.Backoff
}
```

例如在客户端部分，如果需要授权，都会使用该操作，可以在代码 [k8s.io/pkg/registry/authorization/subjectaccessreview/rest.go](pkg/registry/authorization/subjectaccessreview/rest.go)  中可以看到REST中会 authorizer.Authorize 去验证是否有权限操作

```go
func (r *REST) Create(ctx context.Context, obj runtime.Object, createValidation rest.ValidateObjectFunc, options *metav1.CreateOptions) (runtime.Object, error) {
	subjectAccessReview, ok := obj.(*authorizationapi.SubjectAccessReview)
	if !ok {
		return nil, apierrors.NewBadRequest(fmt.Sprintf("not a SubjectAccessReview: %#v", obj))
	}
	if errs := authorizationvalidation.ValidateSubjectAccessReview(subjectAccessReview); len(errs) > 0 {
		return nil, apierrors.NewInvalid(authorizationapi.Kind(subjectAccessReview.Kind), "", errs)
	}

	if createValidation != nil {
		if err := createValidation(ctx, obj.DeepCopyObject()); err != nil {
			return nil, err
		}
	}

	authorizationAttributes := authorizationutil.AuthorizationAttributesFrom(subjectAccessReview.Spec)
	decision, reason, evaluationErr := r.authorizer.Authorize(ctx, authorizationAttributes)

	subjectAccessReview.Status = authorizationapi.SubjectAccessReviewStatus{
		Allowed: (decision == authorizer.DecisionAllow),
		Denied:  (decision == authorizer.DecisionDeny),
		Reason:  reason,
	}
	if evaluationErr != nil {
		subjectAccessReview.Status.EvaluationError = evaluationErr.Error()
	}

	return subjectAccessReview, nil
}
```

authorizer.Authorize 会被实现在每一个该阶段的模式下，在 withAuthentication 构建了一个授权的 http.Handler 函数

[k8s.io/apiserver/pkg/endpoints/filters/authorization.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/staging/src/k8s.io/apiserver/pkg/endpoints/filters/authorization.go#L45-L79)

```go
func WithAuthorization(handler http.Handler, a authorizer.Authorizer, s runtime.NegotiatedSerializer) http.Handler {
	if a == nil {
		klog.Warning("Authorization is disabled")
		return handler
	}
	return http.HandlerFunc(func(w http.ResponseWriter, req *http.Request) {
		ctx := req.Context()

		attributes, err := GetAuthorizerAttributes(ctx)
		if err != nil {
			responsewriters.InternalError(w, req, err)
			return
		}
        // 这里调用了authorizer.Authorizer传入的authorizer来进行鉴权
		authorized, reason, err := a.Authorize(ctx, attributes)
		// an authorizer like RBAC could encounter evaluation errors and still allow the request, so authorizer decision is checked before error here.
		if authorized == authorizer.DecisionAllow {
			audit.AddAuditAnnotations(ctx,
				decisionAnnotationKey, decisionAllow,
				reasonAnnotationKey, reason)
			handler.ServeHTTP(w, req)
			return
		}
		if err != nil {
			audit.AddAuditAnnotation(ctx, reasonAnnotationKey, reasonError)
			responsewriters.InternalError(w, req, err)
			return
		}

		klog.V(4).InfoS("Forbidden", "URI", req.RequestURI, "Reason", reason)
		audit.AddAuditAnnotations(ctx,
			decisionAnnotationKey, decisionForbid,
			reasonAnnotationKey, reason)
		responsewriters.Forbidden(ctx, attributes, w, req, reason, s)
	})
}
```

接下来在 createAggregatorConfig 调用了 BuildHandlerChainWithStorageVersionPrecondition 而又调用了

[cmd/kube-apiserver/app/aggregator.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/cmd/kube-apiserver/app/aggregator.go#L56-L78)

```go
func createAggregatorConfig(
	kubeAPIServerConfig genericapiserver.Config,
	commandOptions *options.ServerRunOptions,
	externalInformers kubeexternalinformers.SharedInformerFactory,
	serviceResolver aggregatorapiserver.ServiceResolver,
	proxyTransport *http.Transport,
	pluginInitializers []admission.PluginInitializer,
) (*aggregatorapiserver.Config, error) {
	// make a shallow copy to let us twiddle a few things
	// most of the config actually remains the same.  We only need to mess with a couple items related to the particulars of the aggregator
	genericConfig := kubeAPIServerConfig
	genericConfig.PostStartHooks = map[string]genericapiserver.PostStartHookConfigEntry{}
	genericConfig.RESTOptionsGetter = nil
	// prevent generic API server from installing the OpenAPI handler. Aggregator server
	// has its own customized OpenAPI handler.
	genericConfig.SkipOpenAPIInstallation = true

	if utilfeature.DefaultFeatureGate.Enabled(genericfeatures.StorageVersionAPI) &&
		utilfeature.DefaultFeatureGate.Enabled(genericfeatures.APIServerIdentity) {
		// Add StorageVersionPrecondition handler to aggregator-apiserver.
		// The handler will block write requests to built-in resources until the
		// target resources' storage versions are up-to-date.
		genericConfig.BuildHandlerChainFunc = genericapiserver.BuildHandlerChainWithStorageVersionPrecondition
	}
```

而 [k8s.io/apiserver/pkg/server/config.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/staging/src/k8s.io/apiserver/pkg/server/config.go#L601-L655) 返回这个函数 BuildHandlerChainWithStorageVersionPrecondition  

```go
handlerChainBuilder := func(handler http.Handler) http.Handler {
    return c.BuildHandlerChainFunc(handler, c.Config)
}

apiServerHandler := NewAPIServerHandler(name, c.Serializer, handlerChainBuilder, delegationTarget.UnprotectedHandler())

s := &GenericAPIServer{
		discoveryAddresses:         c.DiscoveryAddresses,
		LoopbackClientConfig:       c.LoopbackClientConfig,
		legacyAPIGroupPrefixes:     c.LegacyAPIGroupPrefixes,
		admissionControl:           c.AdmissionControl,
		Serializer:                 c.Serializer,
		AuditBackend:               c.AuditBackend,
		Authorizer:                 c.Authorization.Authorizer,
		delegationTarget:           delegationTarget,
		EquivalentResourceRegistry: c.EquivalentResourceRegistry,
		HandlerChainWaitGroup:      c.HandlerChainWaitGroup,
		Handler:                    apiServerHandler,

		listedPathProvider: apiServerHandler,
```

只要知道哪里调用了 handlerChainBuilder 就知道了鉴权步骤在哪里了，可以看到 handlerChainBuilder 被传入了 apiServerHandler，而后被作为参数返回给 `listedPathProvider: &GenericAPIServer{}`

listedPathProvider在 [k8s.io/apiserver/pkg/server/genericapiserver.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/staging/src/k8s.io/apiserver/pkg/server/genericapiserver.go#L282-L284) 

```go
func (s *GenericAPIServer) ListedPaths() []string {
	return s.listedPathProvider.ListedPaths()
}
```

ListedPaths() 又在代码  [k8s.io/apiserver/pkg/server/routes/index.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/staging/src/k8s.io/apiserver/pkg/server/routes/index.go#L38-L47) 中被 构建成这个http服务

```go
// ListedPaths returns the paths that should be shown under /
func (a *APIServerHandler) ListedPaths() []string {
	var handledPaths []string
	// Extract the paths handled using restful.WebService
	for _, ws := range a.GoRestfulContainer.RegisteredWebServices() {
		handledPaths = append(handledPaths, ws.RootPath())
	}
	handledPaths = append(handledPaths, a.NonGoRestfulMux.ListedPaths()...)
	sort.Strings(handledPaths)

	return handledPaths
}


// ServeHTTP serves the available paths.
func (i IndexLister) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	responsewriters.WriteRawJSON(i.StatusCode, metav1.RootPaths{Paths: i.PathProvider.ListedPaths()}, w)
}
```

至此，可以知道，每次请求时，我们在配置 *kube-apiserver* 配置的授权插件 `.authorizer.Authorize` ，而这个参数会被带至 `subjectAccessReview` 向下传递，其中 User,Group,Extra,UID 为 authentication 部分提供

## Authorization webhook

Authorization webhook 位于 [k8s.io/apiserver/plugin/pkg/authorizer/webhook/webhook.go](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/staging/src/k8s.io/apiserver/plugin/pkg/authorizer/webhook/webhook.go#L166-L247)，是通过 *kube-apiserver* 注入进来的配置，就是上面讲到的如果提供了配置就会加入这种类型的 Authorization 插件来认证。当配置此类型的授权插件，Authorize 会被调用，通过向注入的 URL 发起 REST 请求进行授权，请求对象是 `v1beta1.SubjectAccessReview`

下面是请求的实例

```
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "spec": {
	"resourceAttributes": {
	  "namespace": "kittensandponies",
	  "verb": "GET",
	  "group": "group3",
	  "resource": "pods"
	},
	"user": "jane",
	"group": [
	  "group1",
	  "group2"
	]
  }
}
```

webhook 返回的格式

```json
// 如果允许这个用户访问则返回这个格式
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "status": {
	"allowed": true
  }
}

// 如果拒绝这个用户访问则返回这个格式
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "status": {
	"allowed": false,
	"reason": "user does not have read access to the namespace"
  }
}
```

对于webhook来讲，只要接受请求保持上面格式，而返回格式为下属格式，就可以很好的将Kubernetes 权限体系接入到三方系统中，例如 ***open policy agent***。

同样 webhook 也提供了  `Authorize` 函数，如同上面一样会被注入到每个handler中被执行

```go
func (w *WebhookAuthorizer) Authorize(ctx context.Context, attr authorizer.Attributes) (decision authorizer.Decision, reason string, err error) {
	r := &authorizationv1.SubjectAccessReview{}
	if user := attr.GetUser(); user != nil {
		r.Spec = authorizationv1.SubjectAccessReviewSpec{
			User:   user.GetName(),
			UID:    user.GetUID(),
			Groups: user.GetGroups(),
			Extra:  convertToSARExtra(user.GetExtra()),
		}
	}

	if attr.IsResourceRequest() {
		r.Spec.ResourceAttributes = &authorizationv1.ResourceAttributes{
			Namespace:   attr.GetNamespace(),
			Verb:        attr.GetVerb(),
			Group:       attr.GetAPIGroup(),
			Version:     attr.GetAPIVersion(),
			Resource:    attr.GetResource(),
			Subresource: attr.GetSubresource(),
			Name:        attr.GetName(),
		}
	} else {
		r.Spec.NonResourceAttributes = &authorizationv1.NonResourceAttributes{
			Path: attr.GetPath(),
			Verb: attr.GetVerb(),
		}
	}
	key, err := json.Marshal(r.Spec)
	if err != nil {
		return w.decisionOnError, "", err
	}
	if entry, ok := w.responseCache.Get(string(key)); ok {
		r.Status = entry.(authorizationv1.SubjectAccessReviewStatus)
	} else {
		var result *authorizationv1.SubjectAccessReview
		// WithExponentialBackoff will return SAR create error (sarErr) if any.
		if err := webhook.WithExponentialBackoff(ctx, w.retryBackoff, func() error {
			var sarErr error
			var statusCode int

			start := time.Now()
			result, statusCode, sarErr = w.subjectAccessReview.Create(ctx, r, metav1.CreateOptions{})
			latency := time.Since(start)

			if statusCode != 0 {
				w.metrics.RecordRequestTotal(ctx, strconv.Itoa(statusCode))
				w.metrics.RecordRequestLatency(ctx, strconv.Itoa(statusCode), latency.Seconds())
				return sarErr
			}

			if sarErr != nil {
				w.metrics.RecordRequestTotal(ctx, "<error>")
				w.metrics.RecordRequestLatency(ctx, "<error>", latency.Seconds())
			}

			return sarErr
		}, webhook.DefaultShouldRetry); err != nil {
			klog.Errorf("Failed to make webhook authorizer request: %v", err)
			return w.decisionOnError, "", err
		}

		r.Status = result.Status
		if shouldCache(attr) {
			if r.Status.Allowed {
				w.responseCache.Add(string(key), r.Status, w.authorizedTTL)
			} else {
				w.responseCache.Add(string(key), r.Status, w.unauthorizedTTL)
			}
		}
	}
	switch {
	case r.Status.Denied && r.Status.Allowed:
		return authorizer.DecisionDeny, r.Status.Reason, fmt.Errorf("webhook subject access review returned both allow and deny response")
	case r.Status.Denied:
		return authorizer.DecisionDeny, r.Status.Reason, nil
	case r.Status.Allowed:
		return authorizer.DecisionAllow, r.Status.Reason, nil
	default:
		return authorizer.DecisionNoOpinion, r.Status.Reason, nil
	}

}
```

执行 `webhook.Authorize()` 会执行 `w.subjectAccessReview.Create()` 在这里可以看到会发起一个POST请求将 `v1beta1.SubjectAccessReview` 传入给webhook

[k8s.io/apiserver/plugin/pkg/authorizer/webhook/webhook.go.Create](https://github.com/kubernetes/kubernetes/blob/d818028a1851891cdf934a543bb7ff959ec23d50/staging/src/k8s.io/apiserver/plugin/pkg/authorizer/webhook/webhook.go#L317-L329)

```go
func (t *subjectAccessReviewV1Client) Create(ctx context.Context, subjectAccessReview *authorizationv1.SubjectAccessReview, opts metav1.CreateOptions) (result *authorizationv1.SubjectAccessReview, statusCode int, err error) {
	result = &authorizationv1.SubjectAccessReview{}

	restResult := t.client.Post().
		Resource("subjectaccessreviews").
		VersionedParams(&opts, scheme.ParameterCodec).
		Body(subjectAccessReview).
		Do(ctx)

	restResult.StatusCode(&statusCode)
	err = restResult.Into(result)
	return
}
```

## 实验：基于OPA的RBAC模型

通过上面阐述，大致了解到kubernetes认证框架中的用户的分类以及认证的策略由哪些，实验的目的也是为了阐述一个结果，就是使用OIDC/webhook 是比其他方式更好的保护，管理kubernetes集群。首先在安全上，假设网络环境是不安全的，那么任意node节点遗漏 bootstrap  token文件，就意味着拥有了集群中最高权限；其次在管理上，越大的团队，人数越多，不可能每个用户都提供单独的证书或者token，要知道传统教程中讲到token在kubernetes集群中是永久有效的，除非你删除了这个secret/sa；而Kubernetes提供的插件就很好的解决了这些问题。

### 实验环境

- 一个kubernetes集群
- 了解OPA相关技术

**实验大致分为以下几个步骤**：

- 建立一个HTTP服务器用于返回给kubernetes Authorization服务
- 查询用户操作是否有权限

### 实验开始

#### 编写webhook Authorization

这里做的就是接收 `subjectAccessReview` ，将授权结果赋予 `subjectAccessReview.Status.Allowed` ，true/false，然后返回 `subjectAccessReview`  即可

```go
func serveAuthorization(w http.ResponseWriter, r *http.Request) {
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		httpError(w, err)
		return
	}
	klog.V(4).Info("Receied: ", string(b))

	var subjectAccessReview authoV1.SubjectAccessReview
	err = json.Unmarshal(b, &subjectAccessReview)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}
	subjectAccessReview.Status.Allowed = rbac.RBACChek(&subjectAccessReview)
	b, err = json.Marshal(subjectAccessReview)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}
	w.Write(b)
	klog.V(3).Info("Returning: ", string(b))
}
```

#### 编写rego

这里简单配置了两个权限，*admin* 组拥有所有操作权限，不包含 `watch` ，而 *conf* 组只能 *list*，在访问控制三部曲中，已授权的会增加一个组，例如 `system:authenticated` 代表被 *Authentication* 授予通过的用户，所以 Groups 为一个数组格式，这里检查为两个数组的交集 > 1，则肯定代表这个用户拥有该组的权限。

实验中 *rego* 部分可以在 [playground](https://play.openpolicyagent.org/p/JFdryx8eqW) 中测试 

```go
var module = `package k8s
import future.keywords.in

default allow = false
admin_verbs := {"create", "list", "delete", "update"}
admin_groups := {"admin"}
conf_groups := {"conf"}
conf_verbs := {"list"}
allow  {
	groups := {v | v := input.spec.groups[_]}
	count(admin_groups & groups) > 0
	input.spec.resourceAttributes.verb in admin_verbs
}

allow  {
	groups := {v | v := input.spec.groups[_]}
	count(conf_groups & groups) > 0
	input.spec.resourceAttributes.verb in conf_verbs
}
`
```

下面编写 RBACChek 函数，由于go1.16提供了embed功能，就可以直接将 rego embed go中，最后`result.Allowed()`  如果 *input* 通过评估则为 `true` ，反之亦然

```go
func RBACChek(req *authoV1.SubjectAccessReview) bool {
	fmt.Printf("\n%+v\n", req)
	query, err := rego.New(
        // query是要检查的模块，data是固定格式，这与playground中不一样，需要.allow
        // k8s是package
		rego.Query("data.k8s.allow"),
		rego.Module("k8s.allow", module),
	).PrepareForEval(context.TODO())

	if err != nil {
		klog.V(4).Info(err)
		return false
	}
	result, err := query.Eval(context.TODO(), rego.EvalInput(req))

	if err != nil {
		klog.V(4).Info("evaluation error:", err)
		return false
	} else if len(result) == 0 {
		klog.V(4).Info("undefined result", err)
		return false
	}
	return result.Allowed()
}
```

#### 配置kube-apiserver

Authorization webhook 与其他 webhook 一样，启用的方法也是修改 *kube-apiserver* 参数，并指定 `kubeconfig` 类型的配置文件，其中对于 Kubernetes 集群来说 `kubeconfig` 是 kubernetes 客户端访问的信息，而 webhook 这里的 `kubeconfig` 配置文件要填写的则是 webhook的信息，其中 user,cluster,contexts 属性均为 webhook的配置信息 <sup><a href="#1">[1]</a></sup>。

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: http://10.0.0.1:88/authorization
    insecure-skip-tls-verify: true
  name: authorizator
users:
- name: webhook-authorizator
current-context: webhook-authorizator@authorizator
contexts:
- context:
    cluster: authorizator
    user: webhook-authorizator
  name: webhook-authorizator@authorizator
```

修改 *kube-apiserver* 参数

```yaml
--authorization-webhook-config-file=/etc/kubernetes/auth/authorization-webhook.conf \
# 1s 是为了在测试时减少等待的时间，否则缓存太长不会走webhook
--authorization-webhook-cache-authorized-ttl=1s \
--authorization-webhook-cache-unauthorized-ttl=1s \
# api版本建议还是指定下，因为v1与v1beta1的 subjectAccessReview 内容不同rego因为格式问题会为空从而false
# 代码中schema v1与v1beta1相同，测试时收到的请求的格式不一样，没找到原因 TODO
--authorization-webhook-version=v1 \
```

### 验证结果

准备三个外部用户，admin,admin1,searchUser，admin,admin1 为 admin 组，拥有所有权限，searchUser 为 conf 组，仅能 list 操作

```yaml
- name: admin
  user: 
    token: admin@111
- name: admin1
  user:
    token: admin1@111
- name: searchUser
  user:
    token: searchUser@111
```

测试用户 searchUser ，可以看到只能list操作

```bash
$ kubectl get pod --user=searchUser
NAME                      READY   STATUS             RESTARTS   AGE
netbox-85865d5556-hfg6v   1/1     Running            0          96d
netbox-85865d5556-vlgr4   1/1     Running            0          96d
pod                       0/1     CrashLoopBackOff   95         22h

$ kubectl delete pod pod --user=searchUser
Error from server (Forbidden): pods "pod" is forbidden: User "searchUser" cannot delete resource "pods" in API group "" in the namespace "default"
```

测试用户 admin，可以看出可以进行写与查看的操作

```bash
$ kubectl delete pod pod --user=admin
pod "pod" deleted

$ kubectl get pod --user=admin
NAME                      READY   STATUS    RESTARTS   AGE
netbox-85865d5556-hfg6v   1/1     Running   0          96d
netbox-85865d5556-vlgr4   1/1     Running   0          96d
```

## 总结

kubernetes 提供了 Authentication,Authorization,Adminsion Control,Audit 几种webhook，可以自行在Kubernetes之上实现一个4A的标准，Authorization部分提供了一个并行与，但脱离Kubernetes的授权系统，使得外部用户可以很灵活的被授权，而不是手动管理多个clusterrolebinding,rolebingding 之类的资源。

实验中使用了OPA，这里是将rego静态文件embed入go中，在正常情况下OPA给出的架构如下图所示，存在一个 ***OPA Service***，来进行验证，而实验中是直接嵌入到go中，OPA本身也提供了 ***HTTP Service***，可以直接编译运行为 HTTP服务  <sup><a href="#2">[2]</a></sup>。 TODO

![image-20221124224618920](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/image-20221124224618920.png)

<center>图：OPA 架构</center>
<center><em>Source：</em>https://www.openpolicyagent.org/docs/latest/</center><br>

OPA本身提供了 Gatekeeper ，可以作为Kubernetes 资源使用，官方示例是作为为一个kubernetes准入网关，也提供了ingress浏览的验证 <sup><a href="#3">[3]</a></sup>

> Notes：实验中还需要注意的一点则是，如果RBAC与webhook同时验证时，需要合理的规划权限，例如集群组件的账户，coreDNS，flannel等，也会被拒绝（在OPA设置的 `default allow = false` ）。



> **Reference**
>
> <sup id="1">[1]</sup> [***Webhook Mode***](https://kubernetes.io/docs/reference/access-authn-authz/webhook/)
>
> <sup id="2">[2]</sup> [***HTTP APIs***](https://www.openpolicyagent.org/docs/latest/http-api-authorization/)
>
> <sup id="3">[3]</sup> [***What is OPA Gatekeeper?***](https://www.openpolicyagent.org/docs/latest/kubernetes-introduction/#what-is-opa-gatekeeper)
>
> <sup id="4">[4]</sup> [***用 Goalng 开发 OPA 策略***](https://blog.haohtml.com/archives/31514)
>
> <sup id="5">[5]</sup> [***初探 Open Policy Agent 實作 RBAC (Role-based access control) 權限控管***](https://blog.wu-boy.com/2021/04/setup-rbac-role-based-access-control-using-open-policy-agent/)


