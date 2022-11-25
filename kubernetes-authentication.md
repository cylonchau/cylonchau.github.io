# Understanding kubernetes 4A - Authentication and External Authentication




## Overview

本章主要简单阐述kubernetes 认证相关原理，最后以实验来阐述kubernetes用户系统的思路

**objective**：

- 了解kubernetes 各种认证机制的原理
- 了解kubernetes 用户的概念
- 了解kubernetes authentication webhook 
- 完成实验，如何将其他用户系统接入到kubernetes中的一个思路

## Kubernetes 认证

在Kubernetes apiserver对于认证部分所描述的，对于所有用户访问Kubernetes API（通过任何客户端，客户端库，`kubectl` 等）时都会经历 验证 (***Authentication***) , 授权 (***Authorization***), 和准入控制 (***Admission control***) 三个阶段来完成对 “用户” 进行授权，整个流程正如下图所示

![image-20221025003822017](https://cdn.jsdelivr.net/gh/CylonChau/imgbed//img/vhesGDFN3dLdzXwS7vzPdXkI3aglQYZgGhjc-Cx_boaV6URKFFoe8mFRZZUuJyGHywa_bOkeUlIkm-nJkCVMHPk9dr2dXFwNzAQJKzft2phsTcEDjdObjmugBcYtpdPLpLIYuIGzeFYvtsR2Lw.jpeg)
<center>图：Kubernetes API 请求的请求处理步骤图</center>
<center><em>Source：</em>https://www.armosec.io/blog/kubernetes-admission-controller/</center><br>

其中在大多数教程中，在对这三个阶段所做的工作大致上为：

- ***Authentication*** 阶段所指用于确认请求访问Kubernetes API 用户是否为合法用户

- ***Authorization*** 阶段所指的将是这个用户是否有对操作的资源的权限

- ***Admission control*** 阶段所指控制对请求资源进行控制，通俗来说，就是一票否决权，即使前两个步骤完成

到这里了解到了Kubernetes API实际上做的工作就是 “人类用户” 与 “kubernetes service account" <sup><a href="#2">[2]</a></sup>；那么就引出了一个重要概念就是 “用户” 在Kubernetes中是什么，以及用户在认证中的也是本章节的中心。

在Kubernetes官方手册中给出了 ”用户“ 的概念，Kubernetes集群中存在的用户包括 ”普通用户“ 与 “service account” 但是 Kubernetes 没有普通用户的管理方式，只是将使用集群的证书CA签署的有效证书的用户都被视为合法用户 <sup><a href="#3">[3]</a></sup>

那么对于使得Kubernetes集群有一个真正的用户系统，就可以根据上面给出的概念将Kubernetes用户分为 ”外部用户“ 与 ”内部用户“。如何理解外部与内部用户呢？实际上就是有Kubernetes管理的用户，即在kubernetes定义用户的数据模型这种为 “内部用户” ，正如 service account；反之，非Kubernetes托管的用户则为 ”外部用户“ 这中概念也更好的对kubernetes用户的阐述。

对于外部用户来说，实际上Kubernetes给出了多种用户概念 <sup><a href="#3">[3]</a></sup>，例如：

- 拥有kubernetes集群证书的用户
- 拥有Kubernetes集群token的用户（`--token-auth-file` 指定的静态token）
- 用户来自外部用户系统，例如 *OpenID*，*LDAP*，*QQ connect*, *google identity platform* 等

## 向外部用户授权集群访问的示例

### 场景1：通过证书请求k8s

该场景中kubernetes将使用证书中的cn作为用户，ou作为组，如果对应 `rolebinding/clusterrolebinding` 给予该用户权限，那么请求为合法

```bash
$ curl https://hostname:6443/api/v1/pods \
	--cert ./client.pem \
	--key ./client-key.pem \
	--cacert ./ca.pem 
```

接下来浅析下在代码中做的事情

确认用户是 ***apiserver*** 在 ***Authentication*** 阶段 做的事情，而对应代码在 [pkg/kubeapiserver/authenticator](pkg/kubeapiserver/authenticator) 下，整个文件就是构建了一系列的认证器，而x.509证书指是其中一个

```go
// 创建一个认证器，返回请求或一个k8s认证机制的标准错误
func (config Config) New() (authenticator.Request, *spec.SecurityDefinitions, error) {
    
...

	// X509 methods
    // 可以看到这里就是将x509证书解析为user
	if config.ClientCAContentProvider != nil {
		certAuth := x509.NewDynamic(config.ClientCAContentProvider.VerifyOptions, x509.CommonNameUserConversion)
		authenticators = append(authenticators, certAuth)
	}
...
```

接下来看实现原理，NewDynamic函数位于代码 [k8s.io/apiserver/pkg/authentication/request/x509/x509.go](https://github.com/kubernetes/kubernetes/blob/fdc77503e954d1ee641c0e350481f7528e8d068b/staging/src/k8s.io/apiserver/pkg/authentication/request/x509/x509.go#L126-L130)

通过代码可以看出，是通过一个验证函数与用户来解析为一个 *Authenticator*

```go
// NewDynamic returns a request.Authenticator that verifies client certificates using the provided
// VerifyOptionFunc (which may be dynamic), and converts valid certificate chains into user.Info using the provided UserConversion
func NewDynamic(verifyOptionsFn VerifyOptionFunc, user UserConversion) *Authenticator {
	return &Authenticator{verifyOptionsFn, user}
}
```

验证函数为 CAContentProvider 的方法，而x509部分实现为 [k8s.io/apiserver/pkg/server/dynamiccertificates/dynamic_cafile_content.go.VerifyOptions](https://github.com/kubernetes/kubernetes/blob/fdc77503e954d1ee641c0e350481f7528e8d068b/staging/src/k8s.io/apiserver/pkg/server/dynamiccertificates/dynamic_cafile_content.go#L253-L261)；可以看出返回是一个 `x509.VerifyOptions` + 与认证的状态

```go
// VerifyOptions provides verifyoptions compatible with authenticators
func (c *DynamicFileCAContent) VerifyOptions() (x509.VerifyOptions, bool) {
	uncastObj := c.caBundle.Load()
	if uncastObj == nil {
		return x509.VerifyOptions{}, false
	}

	return uncastObj.(*caBundleAndVerifier).verifyOptions, true
}
```

而用户的获取则位于  [k8s.io/apiserver/pkg/authentication/request/x509/x509.go](https://github.com/kubernetes/kubernetes/blob/fdc77503e954d1ee641c0e350481f7528e8d068b/staging/src/k8s.io/apiserver/pkg/authentication/request/x509/x509.go#L248-L258)；可以看出，用户正是拿的证书的CN，而组则是为证书的OU

```go
// CommonNameUserConversion builds user info from a certificate chain using the subject's CommonName
var CommonNameUserConversion = UserConversionFunc(func(chain []*x509.Certificate) (*authenticator.Response, bool, error) {
	if len(chain[0].Subject.CommonName) == 0 {
		return nil, false, nil
	}
	return &authenticator.Response{
		User: &user.DefaultInfo{
			Name:   chain[0].Subject.CommonName,
			Groups: chain[0].Subject.Organization,
		},
	}, true, nil
})
```

由于授权不在本章范围内，直接忽略至入库阶段，入库阶段由 [RESTStorageProvider](https://github.com/kubernetes/kubernetes/blob/fdc77503e954d1ee641c0e350481f7528e8d068b/pkg/controlplane/instance.go#L561) 实现 这里，每一个Provider都提供了 `Authenticator` 这里包含了已经允许的请求，将会被对应的REST客户端写入到库中

```go
type RESTStorageProvider struct {
	Authenticator authenticator.Request
	APIAudiences  authenticator.Audiences
}
// RESTStorageProvider is a factory type for REST storage.
type RESTStorageProvider interface {
	GroupName() string
	NewRESTStorage(apiResourceConfigSource serverstorage.APIResourceConfigSource, restOptionsGetter generic.RESTOptionsGetter) (genericapiserver.APIGroupInfo, error)
}
```

### 场景2：通过token

该场景中，当 ***kube-apiserver*** 开启了 `--enable-bootstrap-token-auth` 时，就可以使用 Bootstrap Token 进行认证，通常如下列命令，在请求头中增加 `Authorization: Bearer <token>` 标识

```bash
$ curl https://hostname:6443/api/v1/pods \
  --cacert ${CACERT} \
  --header "Authorization: Bearer <token>" \
```

接下来浅析下在代码中做的事情

可以看到，在代码 [pkg/kubeapiserver/authenticator.New()](pkg/kubeapiserver/authenticator) 中当 ***kube-apiserver*** 指定了参数 `--token-auth-file=/etc/kubernetes/token.csv"` 这种认证会被激活

```go
if len(config.TokenAuthFile) > 0 {
    tokenAuth, err := newAuthenticatorFromTokenFile(config.TokenAuthFile)
    if err != nil {
        return nil, nil, err
    }
    tokenAuthenticators = append(tokenAuthenticators, authenticator.WrapAudienceAgnosticToken(config.APIAudiences, tokenAuth))
}
```

此时打开 token.csv 查看下token长什么样

```bash
$ cat /etc/kubernetes/token.csv
12ba4f.d82a57a4433b2359,"system:bootstrapper",10001,"system:bootstrappers"
```

这里回到代码 [k8s.io/apiserver/pkg/authentication/token/tokenfile/tokenfile.go.NewCSV](https://github.com/kubernetes/kubernetes/blob/fdc77503e954d1ee641c0e350481f7528e8d068b/staging/src/k8s.io/apiserver/pkg/authentication/token/tokenfile/tokenfile.go#L45-L91) ，这里可以看出，就是读取 `--token-auth-file=` 参数指定的tokenfile，然后解析为用户，`record[1]` 作为用户名，`record[2]` 作为UID

```go
// NewCSV returns a TokenAuthenticator, populated from a CSV file.
// The CSV file must contain records in the format "token,username,useruid"
func NewCSV(path string) (*TokenAuthenticator, error) {
	file, err := os.Open(path)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	recordNum := 0
	tokens := make(map[string]*user.DefaultInfo)
	reader := csv.NewReader(file)
	reader.FieldsPerRecord = -1
	for {
		record, err := reader.Read()
		if err == io.EOF {
			break
		}
		if err != nil {
			return nil, err
		}
		if len(record) < 3 {
			return nil, fmt.Errorf("token file '%s' must have at least 3 columns (token, user name, user uid), found %d", path, len(record))
		}

		recordNum++
		if record[0] == "" {
			klog.Warningf("empty token has been found in token file '%s', record number '%d'", path, recordNum)
			continue
		}

		obj := &user.DefaultInfo{
			Name: record[1],
			UID:  record[2],
		}
		if _, exist := tokens[record[0]]; exist {
			klog.Warningf("duplicate token has been found in token file '%s', record number '%d'", path, recordNum)
		}
		tokens[record[0]] = obj

		if len(record) >= 4 {
			obj.Groups = strings.Split(record[3], ",")
		}
	}

	return &TokenAuthenticator{
		tokens: tokens,
	}, nil
}
```

而token file中配置的格式正是以逗号分隔的一组字符串，

```go
type DefaultInfo struct {
	Name   string
	UID    string
	Groups []string
	Extra  map[string][]string
}
```

这种用户最常见的方式就是 ***kubelet*** 通常会以此类用户向控制平面进行身份认证，例如下列配置

```bash
KUBELET_ARGS="--v=0 \
    --logtostderr=true \
    --config=/etc/kubernetes/kubelet-config.yaml \
    --kubeconfig=/etc/kubernetes/auth/kubelet.conf \
    --network-plugin=cni \
    --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 \
    --bootstrap-kubeconfig=/etc/kubernetes/auth/bootstrap.conf"
```

`/etc/kubernetes/auth/bootstrap.conf` 内容，这里就用到了 ***kube-apiserver*** 配置的 `--token-auth-file=` 用户名，组必须为 `system:bootstrappers` 

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ......
    server: https://10.0.0.4:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: system:bootstrapper
  name: system:bootstrapper@kubernetes
current-context: system:bootstrapper@kubernetes
kind: Config
preferences: {}
users:
- name: system:bootstrapper
```

而通常在二进制部署时会出现的问题，例如下列错误

```log
Unable to register node "hostname" with API server: nodes is forbidden: User "system:anonymous" cannot create resource "nodes" in API group "" at the cluster scope
```

而通常解决方法是执行下列命令，这里就是将 ***kubelet*** 与 ***kube-apiserver*** 通讯时的用户授权，因为kubernetes官方给出的条件是，用户组必须为 `system:bootstrappers`  <sup><a href="#4">[4]</a></sup>

```bash
$ kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
```

生成的clusterrolebinding 如下

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2022-08-14T22:26:51Z"
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
   ...
    time: "2022-08-14T22:26:51Z"
  name: kubelet-bootstrap
  resourceVersion: "158"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/kubelet-bootstrap
  uid: b4d70f4f-4ae0-468f-86b7-55e9351e4719
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node-bootstrapper
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers
```

上述就是 bootstrap token，翻译后就是引导token，因为其做的工作就是将节点载入Kubernetes系统过程提供认证机制的用户。

> Notes：这种用户不存在与kubernetes内，可以算属于一个外部用户，但认证机制中存在并绑定了最高权限，也可以用来做其他访问时的认证

### 场景3：serviceaccount

serviceaccount通常为API自动创建的，但在用户中，实际上认证存在两个方向，一个是 `--service-account-key-file` 这个参数可以指定多个，指定对应的证书文件公钥或私钥，用以办法sa的token

首先会根据指定的公钥或私钥文件生成token

```go
if len(config.ServiceAccountKeyFiles) > 0 {
    serviceAccountAuth, err := newLegacyServiceAccountAuthenticator(config.ServiceAccountKeyFiles, config.ServiceAccountLookup, config.APIAudiences, config.ServiceAccountTokenGetter)
    if err != nil {
        return nil, nil, err
    }
    tokenAuthenticators = append(tokenAuthenticators, serviceAccountAuth)
}
if len(config.ServiceAccountIssuers) > 0 {
    serviceAccountAuth, err := newServiceAccountAuthenticator(config.ServiceAccountIssuers, config.ServiceAccountKeyFiles, config.APIAudiences, config.ServiceAccountTokenGetter)
    if err != nil {
        return nil, nil, err
    }
    tokenAuthenticators = append(tokenAuthenticators, serviceAccountAuth)
}
```

对于  `--service-account-key-file`  他生成的用户都是 “kubernetes/serviceaccount”  , 而对于 `--service-account-issuer` 只是对sa颁发者提供了一个称号标识是谁，而不是统一的 “kubernetes/serviceaccount” ，这里可以从代码中看到，两者是完全相同的，只是称号不同罢了

```go
// newLegacyServiceAccountAuthenticator returns an authenticator.Token or an error
func newLegacyServiceAccountAuthenticator(keyfiles []string, lookup bool, apiAudiences authenticator.Audiences, serviceAccountGetter serviceaccount.ServiceAccountTokenGetter) (authenticator.Token, error) {
	allPublicKeys := []interface{}{}
	for _, keyfile := range keyfiles {
		publicKeys, err := keyutil.PublicKeysFromFile(keyfile)
		if err != nil {
			return nil, err
		}
		allPublicKeys = append(allPublicKeys, publicKeys...)
	}
// 唯一的区别 这里使用了常量 serviceaccount.LegacyIssuer
	tokenAuthenticator := serviceaccount.JWTTokenAuthenticator([]string{serviceaccount.LegacyIssuer}, allPublicKeys, apiAudiences, serviceaccount.NewLegacyValidator(lookup, serviceAccountGetter))
	return tokenAuthenticator, nil
}

// newServiceAccountAuthenticator returns an authenticator.Token or an error
func newServiceAccountAuthenticator(issuers []string, keyfiles []string, apiAudiences authenticator.Audiences, serviceAccountGetter serviceaccount.ServiceAccountTokenGetter) (authenticator.Token, error) {
	allPublicKeys := []interface{}{}
	for _, keyfile := range keyfiles {
		publicKeys, err := keyutil.PublicKeysFromFile(keyfile)
		if err != nil {
			return nil, err
		}
		allPublicKeys = append(allPublicKeys, publicKeys...)
	}
// 唯一的区别 这里根据kube-apiserver提供的称号指定名称
	tokenAuthenticator := serviceaccount.JWTTokenAuthenticator(issuers, allPublicKeys, apiAudiences, serviceaccount.NewValidator(serviceAccountGetter))
	return tokenAuthenticator, nil
}
```

最后根据ServiceAccounts，Secrets等值签发一个token，也就是通过下列命令获取的值

```go
$ kubectl get secret multus-token-v6bfg -n kube-system -o jsonpath={".data.token"}
```

### 场景4：openid

OpenID Connect是 OAuth2 风格，允许用户授权三方网站访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方网站或分享他们数据的所有内容，下面是一张kubernetes 使用 OID 认证的逻辑图

![img](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/kube-login-oidc-ad4caf57f124e622897e0781fe1e3d6e1ecb5c6099776e6677ca800c4458f1de.jpg)

<center>图：Kubernetes OID认证</center>
<center><em>Source：</em>https://developer.okta.com/blog/2021/11/08/k8s-api-server-oidc</center><br>

### 场景5：webhook

webhook是kubernetes提供自定义认证的其中一种，主要是用于认证 “***不记名 token***“ 的钩子，“***不记名 token***“ 将 由身份验证服务创建。当用户对kubernetes访问时，会触发准入控制，当对kubernetes集群注册了 authenticaion webhook时，将会使用该webhook提供的方式进行身份验证时，此时会为您生成一个 token 。

如代码 [pkg/kubeapiserver/authenticator.New()](pkg/kubeapiserver/authenticator)  中所示 newWebhookTokenAuthenticator 会通过提供的config (`--authentication-token-webhook-config-file`) 来创建出一个 WebhookTokenAuthenticator

```go
if len(config.WebhookTokenAuthnConfigFile) > 0 {
    webhookTokenAuth, err := newWebhookTokenAuthenticator(config)
    if err != nil {
        return nil, nil, err
    }

    tokenAuthenticators = append(tokenAuthenticators, webhookTokenAuth)
}
```

下图是kubernetes 中 WebhookToken 验证的工作原理

![Webhook 令牌认证插件](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/25d075712ff343ce492a5db30733cd93.svg)

<center>图：kubernetes WebhookToken验证原理</center>
<center><em>Source：</em>https://learnk8s.io/kubernetes-custom-authentication</center><br>

最后由token中的authHandler，循环所有的Handlers在运行 `AuthenticateToken` 去进行获取用户的信息

```go
func (authHandler *unionAuthTokenHandler) AuthenticateToken(ctx context.Context, token string) (*authenticator.Response, bool, error) {
   var errlist []error
   for _, currAuthRequestHandler := range authHandler.Handlers {
      info, ok, err := currAuthRequestHandler.AuthenticateToken(ctx, token)
      if err != nil {
         if authHandler.FailOnError {
            return info, ok, err
         }
         errlist = append(errlist, err)
         continue
      }

      if ok {
         return info, ok, err
      }
   }

   return nil, false, utilerrors.NewAggregate(errlist)
}
```

而webhook插件也实现了这个方法 `AuthenticateToken` ,这里会通过POST请求，调用注入的webhook，该请求携带一个JSON 格式的 `TokenReview` 对象，其中包含要验证的令牌

```go
func (w *WebhookTokenAuthenticator) AuthenticateToken(ctx context.Context, token string) (*authenticator.Response, bool, error) {

    ....

		start := time.Now()
		result, statusCode, tokenReviewErr = w.tokenReview.Create(ctx, r, metav1.CreateOptions{})
		latency := time.Since(start)
...
}
```

webhook token认证服务要返回[用户的身份信息](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#userinfo-v1beta1-authentication-k8s-io)，就是上面token部分提到的数据结构（webhook来决定接受还是拒绝该用户）

```go
type DefaultInfo struct {
	Name   string
	UID    string
	Groups []string
	Extra  map[string][]string
}
```

### 场景6：代理认证



## 实验：基于LDAP的身份认证

通过上面阐述，大致了解到kubernetes认证框架中的用户的分类以及认证的策略由哪些，实验的目的也是为了阐述一个结果，就是使用OIDC/webhook 是比其他方式更好的保护，管理kubernetes集群。首先在安全上，假设网络环境是不安全的，那么任意node节点遗漏 bootstrap token文件，就意味着拥有了集群中最高权限；其次在管理上，越大的团队，人数越多，不可能每个用户都提供单独的证书或者token，要知道传统教程中讲到token在kubernetes集群中是永久有效的，除非你删除了这个secret/sa；而Kubernetes提供的插件就很好的解决了这些问题。

### 实验环境

- 一个kubernetes集群
- 一个openldap服务，建议可以是集群外部的，因为webhook不像SSSD有缓存机制，并且集群不可用，那么认证不可用，当认证不可用时会导致集群不可用，这样事故影响的范围可以得到控制，也叫最小化半径
- 了解ldap相关技术，并了解go ldap客户端

**实验大致分为以下几个步骤**：

- 建立一个HTTP服务器用于返回给kubernetes Authenticaion服务
- 查询ldap该用户是否合法
    - 查询用户是否合法
    - 查询用户所属组是否拥有权限

### 实验开始

#### 初始化用户数据

首先准备openldap初始化数据，创建三个 posixGroup 组，与5个用户 admin, admin1, admin11, searchUser, syncUser 密码均为111，组与用户关联使用的 `memberUid` 

```bash
cat << EOF | ldapdelete -r  -H ldap://10.0.0.3 -D "cn=admin,dc=test,dc=com" -w 111
dn: dc=test,dc=com
objectClass: top
objectClass: organizationalUnit
objectClass: extensibleObject
description: US Organization
ou: people

dn: ou=tvb,dc=test,dc=com
objectClass: organizationalUnit
description: Television Broadcasts Limited
ou: tvb

dn: cn=admin,ou=tvb,dc=test,dc=com
objectClass: posixGroup
gidNumber: 10000
cn: admin

dn: cn=conf,ou=tvb,dc=test,dc=com
objectClass: posixGroup
gidNumber: 10001
cn: conf

dn: cn=dir,ou=tvb,dc=test,dc=com
objectClass: posixGroup
gidNumber: 10002
cn: dir

dn: uid=syncUser,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: pwdPolicy
pwdAttribute: userPassword
uid: syncUser
cn: syncUser
uidNumber: 10006
gidNumber: 10002
homeDirectory: /home/syncUser
loginShell: /bin/bash
sn: syncUser
givenName: syncUser
memberOf: cn=confGroup,ou=tvb,dc=test,dc=com

dn: uid=searchUser,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: pwdPolicy
pwdAttribute: userPassword
uid: searchUser
cn: searchUser
uidNumber: 10005
gidNumber: 10001
homeDirectory: /home/searchUser
loginShell: /bin/bash
sn: searchUser
givenName: searchUser
memberOf: cn=dirGroup,ou=tvb,dc=test,dc=com

dn: uid=admin1,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: pwdPolicy
pwdAttribute: userPassword
uid: admin1
sn: admin1
cn: admin
uidNumber: 10010
gidNumber: 10000
homeDirectory: /home/admin
loginShell: /bin/bash
givenName: admin
memberOf: cn=adminGroup,ou=tvb,dc=test,dc=com

dn: uid=admin11,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: pwdPolicy
sn: admin11
pwdAttribute: userPassword
uid: admin11
cn: admin11
uidNumber: 10011
gidNumber: 10000
homeDirectory: /home/admin
loginShell: /bin/bash
givenName: admin11
memberOf: cn=adminGroup,ou=tvb,dc=test,dc=com

dn: uid=admin,ou=tvb,dc=test,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: pwdPolicy
pwdAttribute: userPassword
uid: admin
cn: admin
uidNumber: 10009
gidNumber: 10000
homeDirectory: /home/admin
loginShell: /bin/bash
sn: admin
givenName: admin
memberOf: cn=adminGroup,ou=tvb,dc=test,dc=com
EOF
```

接下来需要确定如何为认证成功的用户，上面讲到对于kubernetes中用户格式为 `v1.UserInfo` 的格式，即要获得用户，即用户组，假设需要查找的用户为，admin，那么在openldap中查询filter如下：

```bash
"(|(&(objectClass=posixAccount)(uid=admin))(&(objectClass=posixGroup)(memberUid=admin)))"
```

上面语句意思是，找到 `objectClass=posixAccount` 并且 `uid=admin` 或者 `objectClass=posixGroup` 并且 `memberUid=admin` 的条目信息，这里使用 ”|“ 与 ”&“ 是为了要拿到这两个结果。

#### 编写webhook查询用户部分

这里由于openldap配置密码保存格式不是明文的，如果直接使用 ”=“ 来验证是查询不到内容的，故直接多用了一次登录来验证用户是否合法

```go
func ldapSearch(username, password string) (*v1.UserInfo, error) {
	ldapconn, err := ldap.DialURL(ldapURL)
	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}
	defer ldapconn.Close()

	// Authenticate as LDAP admin user
	err = ldapconn.Bind("uid=searchUser,ou=tvb,dc=test,dc=com", "111")
	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}

	// Execute LDAP Search request
	result, err := ldapconn.Search(ldap.NewSearchRequest(
		"ou=tvb,dc=test,dc=com",
		ldap.ScopeWholeSubtree,
		ldap.NeverDerefAliases,
		0,
		0,
		false,
		fmt.Sprintf("(&(objectClass=posixGroup)(memberUid=%s))", username), // Filter
		nil,
		nil,
	))

	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}

	userResult, err := ldapconn.Search(ldap.NewSearchRequest(
		"ou=tvb,dc=test,dc=com",
		ldap.ScopeWholeSubtree,
		ldap.NeverDerefAliases,
		0,
		0,
		false,
		fmt.Sprintf("(&(objectClass=posixAccount)(uid=%s))", username), // Filter
		nil,
		nil,
	))

	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}

	if len(result.Entries) == 0 {
		klog.V(3).Info("User does not exist")
		return nil, errors.New("User does not exist")
	} else {
		// 验证用户名密码是否正确
		if err := ldapconn.Bind(userResult.Entries[0].DN, password); err != nil {
			e := fmt.Sprintf("Failed to auth. %s\n", err)
			klog.V(3).Info(e)
			return nil, errors.New(e)
		} else {
			klog.V(3).Info(fmt.Sprintf("User %s Authenticated successfuly!", username))
		}
		// 拼接为kubernetes authentication 的用户格式
		user := new(v1.UserInfo)
		for _, v := range result.Entries {
			attrubute := v.GetAttributeValue("objectClass")
			if strings.Contains(attrubute, "posixGroup") {
				user.Groups = append(user.Groups, v.GetAttributeValue("cn"))
			}
		}

		u := userResult.Entries[0].GetAttributeValue("uid")
		user.UID = u
		user.Username = u
		return user, nil
	}
}
```

#### 编写HTTP部分

这里有几个需要注意的部分，即用户或者理解为要认证的token的定义，此处使用了 ”username@password“ 格式作为用户的辨别，即登录kubernetes时需要直接输入 ”username@password“ 来作为登录的凭据。

第二个部分为返回值，返回给Kubernetes的格式必须为 `api/authentication/v1.TokenReview` 格式，`Status.Authenticated` 表示用户身份验证结果，如果该用户合法，则设置 `tokenReview.Status.Authenticated = true` 反之亦然。如果验证成功还需要 `Status.User` 这就是在`ldapSearch` 

```go
func serve(w http.ResponseWriter, r *http.Request) {
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		httpError(w, err)
		return
	}
	klog.V(4).Info("Receiving: %s\n", string(b))

	var tokenReview v1.TokenReview
	err = json.Unmarshal(b, &tokenReview)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}

	// 提取用户名与密码
	s := strings.SplitN(tokenReview.Spec.Token, "@", 2)
	if len(s) != 2 {
		klog.V(3).Info(fmt.Errorf("badly formatted token: %s", tokenReview.Spec.Token))
		httpError(w, fmt.Errorf("badly formatted token: %s", tokenReview.Spec.Token))
		return
	}
	username, password := s[0], s[1]
	// 查询ldap，验证用户是否合法
	userInfo, err := ldapSearch(username, password)
	if err != nil {
		// 这里不打印日志的原因是 ldapSearch 中打印过了
		return
	}

	// 设置返回的tokenReview
	if userInfo == nil {
		tokenReview.Status.Authenticated = false
	} else {
		tokenReview.Status.Authenticated = true
		tokenReview.Status.User = *userInfo
	}

	b, err = json.Marshal(tokenReview)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}
	w.Write(b)
	klog.V(3).Info("Returning: ", string(b))
}

func httpError(w http.ResponseWriter, err error) {
	err = fmt.Errorf("Error: %v", err)
	w.WriteHeader(http.StatusInternalServerError) // 500
	fmt.Fprintln(w, err)
	klog.V(4).Info("httpcode 500: ", err)
}
```

下面是完整的代码

```go
package main

import (
	"encoding/json"
	"errors"
	"flag"
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"

	"github.com/go-ldap/ldap"
	"k8s.io/api/authentication/v1"
	"k8s.io/klog/v2"
)

var ldapURL string

func main() {
	klog.InitFlags(nil)
	flag.Parse()
	http.HandleFunc("/authenticate", serve)
	klog.V(4).Info("Listening on port 443 waiting for requests...")
	klog.V(4).Info(http.ListenAndServe(":443", nil))
	ldapURL = "ldap://10.0.0.10:389"
	ldapSearch("admin", "1111")
}

func serve(w http.ResponseWriter, r *http.Request) {
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		httpError(w, err)
		return
	}
	klog.V(4).Info("Receiving: %s\n", string(b))

	var tokenReview v1.TokenReview
	err = json.Unmarshal(b, &tokenReview)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}

	// 提取用户名与密码
	s := strings.SplitN(tokenReview.Spec.Token, "@", 2)
	if len(s) != 2 {
		klog.V(3).Info(fmt.Errorf("badly formatted token: %s", tokenReview.Spec.Token))
		httpError(w, fmt.Errorf("badly formatted token: %s", tokenReview.Spec.Token))
		return
	}
	username, password := s[0], s[1]
	// 查询ldap，验证用户是否合法
	userInfo, err := ldapSearch(username, password)
	if err != nil {
		// 这里不打印日志的原因是 ldapSearch 中打印过了
		return
	}

	// 设置返回的tokenReview
	if userInfo == nil {
		tokenReview.Status.Authenticated = false
	} else {
		tokenReview.Status.Authenticated = true
		tokenReview.Status.User = *userInfo
	}

	b, err = json.Marshal(tokenReview)
	if err != nil {
		klog.V(3).Info("Json convert err: ", err)
		httpError(w, err)
		return
	}
	w.Write(b)
	klog.V(3).Info("Returning: ", string(b))
}

func httpError(w http.ResponseWriter, err error) {
	err = fmt.Errorf("Error: %v", err)
	w.WriteHeader(http.StatusInternalServerError) // 500
	fmt.Fprintln(w, err)
	klog.V(4).Info("httpcode 500: ", err)
}

func ldapSearch(username, password string) (*v1.UserInfo, error) {

	ldapconn, err := ldap.DialURL(ldapURL)
	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}
	defer ldapconn.Close()

	// Authenticate as LDAP admin user
	err = ldapconn.Bind("cn=admin,dc=test,dc=com", "111")
	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}

	// Execute LDAP Search request
	result, err := ldapconn.Search(ldap.NewSearchRequest(
		"ou=tvb,dc=test,dc=com",
		ldap.ScopeWholeSubtree,
		ldap.NeverDerefAliases,
		0,
		0,
		false,
		fmt.Sprintf("(&(objectClass=posixGroup)(memberUid=%s))", username), // Filter
		nil,
		nil,
	))

	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}

	userResult, err := ldapconn.Search(ldap.NewSearchRequest(
		"ou=tvb,dc=test,dc=com",
		ldap.ScopeWholeSubtree,
		ldap.NeverDerefAliases,
		0,
		0,
		false,
		fmt.Sprintf("(&(objectClass=posixAccount)(uid=%s))", username), // Filter
		nil,
		nil,
	))

	if err != nil {
		klog.V(3).Info(err)
		return nil, err
	}

	if len(result.Entries) == 0 {
		klog.V(3).Info("User does not exist")
		return nil, errors.New("User does not exist")
	} else {
		// 验证用户名密码是否正确
		if err := ldapconn.Bind(userResult.Entries[0].DN, password); err != nil {
			e := fmt.Sprintf("Failed to auth. %s\n", err)
			klog.V(3).Info(e)
			return nil, errors.New(e)
		} else {
			klog.V(3).Info(fmt.Sprintf("User %s Authenticated successfuly!", username))
		}
		// 拼接为kubernetes authentication 的用户格式
		user := new(v1.UserInfo)
		for _, v := range result.Entries {
			attrubute := v.GetAttributeValue("objectClass")
			if strings.Contains(attrubute, "posixGroup") {
				user.Groups = append(user.Groups, v.GetAttributeValue("cn"))
			}
		}

		u := userResult.Entries[0].GetAttributeValue("uid")
		user.UID = u
		user.Username = u
		return user, nil
	}
}
```

### 部署webhook

kubernetes官方手册中指出，启用webhook认证的标记是在 ***kube-apiserver*** 指定参数 `--authentication-token-webhook-config-file` 。而这个配置文件是一个 *kubeconfig* 类型的文件格式 <sup><a href="#5">[5]</a></sup>

下列是部署在kubernetes集群外部的配置

创建一个给 *kube-apiserver* 使用的配置文件 `/etc/kubernetes/auth/authentication-webhook.conf`

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: http://10.0.0.1:88/authenticate
  name: authenticator
users:
- name: webhook-authenticator
current-context: webhook-authenticator@authenticator
contexts:
- context:
    cluster: authenticator
    user: webhook-authenticator
  name: webhook-authenticator@authenticator
```

修改 *kube-apiserver* 参数

```bash
# 指向对应的配置文件
--authentication-token-webhook-config-file=/etc/kubernetes/auth/authentication-webhook.conf
# 这个是token缓存时间，指的是用户在访问API时验证通过后在一定时间内无需在请求webhook进行认证了
--authentication-token-webhook-cache-ttl=30m
# 版本指定为API使用哪个版本？authentication.k8s.io/v1或v1beta1
--authentication-token-webhook-version=v1
```

 启动服务后，创建一个 kubeconfig 中的用户用于验证结果

```conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: 
    server: https://10.0.0.4:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: k8s-admin
  name: k8s-admin@kubernetes
current-context: k8s-admin@kubernetes
kind: Config
preferences: {}
users:
- name: admin
  user: 
    token: admin@111
```

### 验证结果

**当密码不正确时，使用用户admin请求集群**

```bash
$ kubectl get pods --user=admin
error: You must be logged in to the server (Unauthorized)
```

**当密码正确时，使用用户admin请求集群**

```bash
$ kubectl get pods --user=admin
Error from server (Forbidden): pods is forbidden: User "admin" cannot list resource "pods" in API group "" in the namespace "default"
```

可以看到admin用户是一个不存在与集群中的用户，并且提示没有权限操作对应资源，此时将admin用户与集群中的cluster-admin绑定，测试结果

```bash
$ kubectl create clusterrolebinding admin \
	--clusterrole=cluster-admin \
	--group=admin
```

此时再尝试使用admin用户访问集群

```bash
$ kubectl get pods --user=admin
NAME                      READY   STATUS    RESTARTS   AGE
netbox-85865d5556-hfg6v   1/1     Running   0          91d
netbox-85865d5556-vlgr4   1/1     Running   0          91d
```

## 总结

kubernetes authentication 插件提供的功能可以注入一个认证系统，这样可以完美解决了kubernetes中用户的问题，而这些用户并不存在与kubernetes中，并且也无需为多个用户准备大量serviceaccount或者证书，也可以完成鉴权操作。首先返回值标准如下所示，如果kubernetes集群有对在其他用户系统中获得的 `Groups` 并建立了 `clusterrolebinding` 或 `rolebinding` 那么这个组的所有用户都将有这些权限。管理员只需要维护与公司用户系统中组同样多的 clusterrole 与 clusterrolebinding 即可

```go
type DefaultInfo struct {
	Name   string
	UID    string
	Groups []string
	Extra  map[string][]string
}
```

对于如何将 kubernetes 与其他平台进行融合可以参考 [文章](https://cylonchau.github.io/kubernetes-dashborad-based.html)

> Notes：Kubernetes原生就支持OID，完全不用自己开发webhook从而实现接入其他系统，这里展示的只是一个思路



> **Reference**
>
> <sup id="1">[1]</sup> [***Implementing a custom Kubernetes authentication method***](https://learnk8s.io/kubernetes-custom-authentication)
>
> <sup id="2">[2]</sup> [***Controlling Access to the Kubernetes API***](https://kubernetes.io/docs/concepts/security/controlling-access/)
>
> <sup id="3">[3]</sup> [***Users in Kubernetes***](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#users-in-kubernetes)
>
> <sup id="4">[4]</sup> [***bootstrap tokens***](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#bootstrap-tokens)
>
> <sup id="5">[5]</sup> [***Webhook Token Authentication***](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication)

