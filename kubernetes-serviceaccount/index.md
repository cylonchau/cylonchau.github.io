# Kubernetes serviceaccount


在整个Kubernetes集群来讲 apiserver是访问控制的唯一入口。如通过service或ingress暴露之后，是可以不通过apiserver接入的，只需要通过节点的nodePort或者ingress controller daemonset共享宿主机节点网络名称空间监听的宿主机网络地址（节点地址），直接接入。

当请求到达APIServer时，会经历几个阶段，如图所示

![image-20221025003822017](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221025003822017.png)

&lt;center&gt;图：Kubernetes API 请求的请求处理步骤图&lt;/center&gt;

&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://kubevious.io/blog/post/securing-kubernetes-using-pod-security-policy-admission-controller&lt;/center&gt;

**任何用户（sa与人类用户）在通过任何方式试图操作API资源时，必须要经历下列的操作**：

- **Authentication**，这个步骤在建立TLS连接后，验证包含，证书、密码，Token；可以指定多种认证，依次尝试每一个，直到其中一个认证成功。如果认证失败，此时客户端收到的是401。
- **Authorization**，此步骤是在完成 *Authentication* 后确定了来源用户，此时用户的请求动作必须被授权。如bob用户对pod资源有 `get` , `list` 权限操作。如果
- **Admission Control**：此步骤为图3，与 *Authorization* 不同的时，这里只要有任意准入控制器拒绝，则拒绝；多个准入控制器会按顺序执行

Refer to [controlling access](https://kubernetes.io/docs/concepts/security/controlling-access/)

### 认证

Kubernetes是高度模块化设计的，因此其认证授权与准入控制是各自都通过插件的方式，可由用户自定义选择经由什么样的插件来完成何种控制逻辑。如**对称秘钥认证方式**、**令牌认证**。由于Kubernetes提供的是resetful方式的接口，其服务都是通过HTTP协议提供的，因此认证信息只能经由HTTP协议的认证首部进行传递，此认证首部通常被称作认证令牌(token)。

**ssl认证**，对于Kubernetes访问来讲，ssl证书能让客户端去确认服务器的身份，（要求服务端发送服务端证书，确认证书是否为认可的CA签署的。）在Kubernetes通信过程当中，重要的是服务器还需认证客户端的身份，因此==Kubectl也应有一个证书，并且此证书为server端所认可的CA所签署的证书==。并且客户端身份也要与证书当中标识的身份保持一致。双方需互相做双向证书认证。认证之后双方基于SSL会话实现加密通讯。

***

注：kubernetes认证无需执行串行检查，用户经过任何一个认证插件通过后，即表示认证通过，无需再经由其他插件进行检查。

***

### 授权

kubernetes的授权也支持多种授权插件来完成用户的权限检查，kubernetes 1.6之后开始支持基于RBAC的认证。除此只外还有基于节点的认证、webhook基于http回调机制，通过web的rest服务来实现认证的检查机制。最重要的是RBAC的授权检查机制。基于角色的访问控制，通常只有许可授权，没有拒绝授权。默认都是拒绝。

在默认情况下，使用kubeadm部署Kubernetes集群是强制启用了RBAC认证的。


### 准入控制

一般而言，准入控制本身只是用来定义对应授权检查完成之后的后续其他安全检查操作的。

### 用户账号

一般而言用户账号大体上应具有以下信息

- **user** 用户，一般而言由`username`与`userid`组成。

- **group** 用户组

- **extra** 用来提供额外信息

- **API资源** k8sapiserver是分组的，向哪个组，哪个版本的哪个api资源对象发出请求必须进行标识，所有的请求资源通过url path进行标识的。如 **`/apis/apps/v1/`**，所有名称空间级别的资源在访问时一般都需指名namespaces关键词，并给出namespaces名称来获取 **`/apis/apps/v1/namespaces/default/`**  **`/apis/apps/v1/namespaces/default/nginx`** 。

  一个完整意义上的url 对象引用url格式 ==`/apis/&lt;GROUPS&gt;/&lt;VERSION&gt;/namespaces/&lt;NameSpace_name&gt;/&lt;Kind&gt;/[/object_id]`==

```bash
$ kubectl api-versions
admissionregistration.k8s.io/v1beta1
...
```

Kubernetes中，所有的api都取决于一个根 `/apis`

```
$ curl -k --cert /etc/k8s/pki/apiserver-kubelet-client.crt --key /etc/k8s/pki/apiserver-kubelet-client.key  https://localhost:6443/apis/apps/v1/namespaces/typay/deployments/nginx-ingress-controller
{
  &#34;kind&#34;: &#34;Deployment&#34;,
  &#34;apiVersion&#34;: &#34;apps/v1&#34;,
  &#34;metadata&#34;: {
    &#34;name&#34;: &#34;nginx-ingress-controller&#34;,
    &#34;namespace&#34;: &#34;houtu&#34;,
    &#34;selfLink&#34;: &#34;/apis/apps/v1/namespaces/houtu/deployments/nginx-ingress-controller&#34;,
    &#34;uid&#34;: &#34;dc8cbca7-fcab-49c5-a4f4-b44858bbf603&#34;,
    &#34;resourceVersion&#34;: &#34;225525&#34;,
    &#34;generation&#34;: 4,
    &#34;creationTimestamp&#34;: &#34;2019-11-21T12:47:33Z&#34;,
    &#34;labels&#34;: {
      &#34;k8s-app&#34;: &#34;nginx-ingress-controller&#34;
    },
    &#34;annotations&#34;: {
      &#34;deployment.kubernetes.io/revision&#34;: &#34;4&#34;,
      &#34;kubectl.kubernetes.io/last-applied-configuration&#34;: &#34;{\&#34;apiVersion\&#34;:\&#34;extensions/v1beta1\&#34;,\&#34;kind\&#34;:\&#34;Deployment\&#34;,\&#34;metadata\&#34;:{\&#34;annotations\&#34;:{},\&#34;labels\&#34;:{\&#34;k8s-app\&#34;:\&#34;nginx-ingress-controller\&#34;},\&#34;name\&#34;:\&#34;nginx-ingress-controller\&#34;,\&#34;namespace\&#34;:\&#34;houtu\&#34;},\&#34;spec\&#34;:{\&#34;replicas\&#34;:1,\&#34;template\&#34;:{\&#34;metadata\&#34;:{\&#34;labels\&#34;:{\&#34;k8s-app\&#34;:\&#34;nginx-ingress-controller\&#34;}},\&#34;spec\&#34;:{\&#34;containers\&#34;:[{\&#34;args\&#34;:[\&#34;/nginx-ingress-controller\&#34;,\&#34;--default-backend-service=houtu/push-front\&#34;],\&#34;env\&#34;:[{\&#34;name\&#34;:\&#34;POD_NAME\&#34;,\&#34;valueFrom\&#34;:{\&#34;fieldRef\&#34;:{\&#34;fieldPath\&#34;:\&#34;metadata.name\&#34;}}},{\&#34;name\&#34;:\&#34;POD_NAMESPACE\&#34;,\&#34;valueFrom\&#34;:{\&#34;fieldRef\&#34;:{\&#34;fieldPath\&#34;:\&#34;metadata.namespace\&#34;}}}],\&#34;image\&#34;:\&#34;quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1\&#34;,\&#34;livenessProbe\&#34;:{\&#34;httpGet\&#34;:{\&#34;path\&#34;:\&#34;/healthz\&#34;,\&#34;port\&#34;:10254,\&#34;scheme\&#34;:\&#34;HTTP\&#34;},\&#34;initialDelaySeconds\&#34;:10,\&#34;timeoutSeconds\&#34;:1},\&#34;name\&#34;:\&#34;nginx-ingress-controller\&#34;,\&#34;ports\&#34;:[{\&#34;containerPort\&#34;:80,\&#34;hostPort\&#34;:80},{\&#34;containerPort\&#34;:443,\&#34;hostPort\&#34;:443}],\&#34;readinessProbe\&#34;:{\&#34;httpGet\&#34;:{\&#34;path\&#34;:\&#34;/healthz\&#34;,\&#34;port\&#34;:10254,\&#34;scheme\&#34;:\&#34;HTTP\&#34;}}}],\&#34;hostNetwork\&#34;:true,\&#34;serviceAccountName\&#34;:\&#34;nginx-ingress-serviceaccount\&#34;,\&#34;terminationGracePeriodSeconds\&#34;:60}}}}\n&#34;
    }
  },
  &#34;spec&#34;: {
    &#34;replicas&#34;: 1,
    &#34;selector&#34;: {
      &#34;matchLabels&#34;: {
        &#34;k8s-app&#34;: &#34;nginx-ingress-controller&#34;
      }
    },
    &#34;template&#34;: {
      &#34;metadata&#34;: {
        &#34;creationTimestamp&#34;: null,
        &#34;labels&#34;: {
          &#34;k8s-app&#34;: &#34;nginx-ingress-controller&#34;
        }
      },
      &#34;spec&#34;: {
        &#34;containers&#34;: [
          {
            &#34;name&#34;: &#34;nginx-ingress-controller&#34;,
            &#34;image&#34;: &#34;quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1&#34;,
            &#34;args&#34;: [
              &#34;/nginx-ingress-controller&#34;,
              &#34;--default-backend-service=houtu/push-front&#34;
            ],
            &#34;ports&#34;: [
              {
                &#34;hostPort&#34;: 80,
                &#34;containerPort&#34;: 80,
                &#34;protocol&#34;: &#34;TCP&#34;
              },
              {
                &#34;hostPort&#34;: 443,
                &#34;containerPort&#34;: 443,
                &#34;protocol&#34;: &#34;TCP&#34;
              }
            ],
            &#34;env&#34;: [
              {
                &#34;name&#34;: &#34;POD_NAME&#34;,
                &#34;valueFrom&#34;: {
                  &#34;fieldRef&#34;: {
                    &#34;apiVersion&#34;: &#34;v1&#34;,
                    &#34;fieldPath&#34;: &#34;metadata.name&#34;
                  }
                }
              },
              {
                &#34;name&#34;: &#34;POD_NAMESPACE&#34;,
                &#34;valueFrom&#34;: {
                  &#34;fieldRef&#34;: {
                    &#34;apiVersion&#34;: &#34;v1&#34;,
                    &#34;fieldPath&#34;: &#34;metadata.namespace&#34;
                  }
                }
              }
            ],
            &#34;resources&#34;: {
              
            },
            &#34;livenessProbe&#34;: {
              &#34;httpGet&#34;: {
                &#34;path&#34;: &#34;/healthz&#34;,
                &#34;port&#34;: 10254,
                &#34;scheme&#34;: &#34;HTTP&#34;
              },
              &#34;initialDelaySeconds&#34;: 10,
              &#34;timeoutSeconds&#34;: 1,
              &#34;periodSeconds&#34;: 10,
              &#34;successThreshold&#34;: 1,
              &#34;failureThreshold&#34;: 3
            },
            &#34;readinessProbe&#34;: {
              &#34;httpGet&#34;: {
                &#34;path&#34;: &#34;/healthz&#34;,
                &#34;port&#34;: 10254,
                &#34;scheme&#34;: &#34;HTTP&#34;
              },
              &#34;timeoutSeconds&#34;: 1,
              &#34;periodSeconds&#34;: 10,
              &#34;successThreshold&#34;: 1,
              &#34;failureThreshold&#34;: 3
            },
            &#34;terminationMessagePath&#34;: &#34;/dev/termination-log&#34;,
            &#34;terminationMessagePolicy&#34;: &#34;File&#34;,
            &#34;imagePullPolicy&#34;: &#34;IfNotPresent&#34;
          }
        ],
        &#34;restartPolicy&#34;: &#34;Always&#34;,
        &#34;terminationGracePeriodSeconds&#34;: 60,
        &#34;dnsPolicy&#34;: &#34;ClusterFirst&#34;,
        &#34;serviceAccountName&#34;: &#34;nginx-ingress-serviceaccount&#34;,
        &#34;serviceAccount&#34;: &#34;nginx-ingress-serviceaccount&#34;,
        &#34;hostNetwork&#34;: true,
        &#34;securityContext&#34;: {
          
        },
        &#34;schedulerName&#34;: &#34;default-scheduler&#34;
      }
    },
    &#34;strategy&#34;: {
      &#34;type&#34;: &#34;RollingUpdate&#34;,
      &#34;rollingUpdate&#34;: {
        &#34;maxUnavailable&#34;: 1,
        &#34;maxSurge&#34;: 1
      }
    },
    &#34;revisionHistoryLimit&#34;: 2147483647,
    &#34;progressDeadlineSeconds&#34;: 2147483647
  },
  &#34;status&#34;: {
    &#34;observedGeneration&#34;: 4,
    &#34;replicas&#34;: 1,
    &#34;updatedReplicas&#34;: 1,
    &#34;unavailableReplicas&#34;: 1,
    &#34;conditions&#34;: [
      {
        &#34;type&#34;: &#34;Available&#34;,
        &#34;status&#34;: &#34;True&#34;,
        &#34;lastUpdateTime&#34;: &#34;2019-11-21T12:47:33Z&#34;,
        &#34;lastTransitionTime&#34;: &#34;2019-11-21T12:47:33Z&#34;,
        &#34;reason&#34;: &#34;MinimumReplicasAvailable&#34;,
        &#34;message&#34;: &#34;Deployment has minimum availability.&#34;
      }
    ]
  }
}
```



```bash
$ curl -k --cert /etc/k8s/pki/apiserver-kubelet-client.crt --key /etc/k8s/pki/apiserver-kubelet-client.key  https://localhost:6443/api/v1/namespaces/houtu
{
  &#34;kind&#34;: &#34;Namespace&#34;,
  &#34;apiVersion&#34;: &#34;v1&#34;,
  &#34;metadata&#34;: {
    &#34;name&#34;: &#34;houtu&#34;,
    &#34;selfLink&#34;: &#34;/api/v1/namespaces/houtu&#34;,
    &#34;uid&#34;: &#34;da736612-d112-4e38-8546-0f2b9169b92f&#34;,
    &#34;resourceVersion&#34;: &#34;201510&#34;,
    &#34;creationTimestamp&#34;: &#34;2019-11-21T08:33:38Z&#34;
  },
  &#34;spec&#34;: {
    &#34;finalizers&#34;: [
      &#34;kubernetes&#34;
    ]
  },
  &#34;status&#34;: {
    &#34;phase&#34;: &#34;Active&#34;
  }
}
```

删除操作

```
$ curl X DELETE -k --cert /etc/k8s/pki/apiserver-kubelet-client.crt --key /etc/k8s/pki/apiserver-kubelet-client.key  https://localhost:6443/apis/apps/v1/namespaces/default/deployments/nginx-test/
{
  &#34;kind&#34;: &#34;Status&#34;,
  &#34;apiVersion&#34;: &#34;v1&#34;,
  &#34;metadata&#34;: {
    
  },
  &#34;status&#34;: &#34;Success&#34;,
  &#34;details&#34;: {
    &#34;name&#34;: &#34;nginx-test&#34;,
    &#34;group&#34;: &#34;apps&#34;,
    &#34;kind&#34;: &#34;deployments&#34;,
    &#34;uid&#34;: &#34;12c5184c-82bc-4f7d-8ec8-38a2439983cf&#34;
  }
}

$ kubectl get deploy
No resources found.
```



在Kubernetes之上，来自于那些地方的客户端需要和apiserver打交道

- 集群外部客户端，通过apiserver对外通信的监听地址
- 集群之上的客户端，apiserver拥有一个在集群内工作地址，``kubectl get svc` 查看，kubernetes是将apiserver以service方式引入到集群内部，从而使得Pod直接请求集群上的apiserver的服务了。

Pod在请求apiserver上的服务是通过10.96.0.1来进行的。但是apiserver请求是需要做认证的。首先apiserver将自己证书传递给客户端，客户端去校验服务端(apiserver)身份。 服务器发给每个Pod客户端的时候，证书所标明的身份的地址为10.96.0.1，所以在apiserver上手动创建证书，必须要确保证书持有者名称能够解析到两条IP记录才可以。

```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    &lt;none&gt;        443/TCP   47h

$ kubectl describe svc kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       &lt;none&gt;
Selector:          &lt;none&gt;
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         172.31.71.50:6443
Session Affinity:  None
Events:            &lt;none&gt;
```



类型分类


- 对象类 deployment、namespace都属于对象类。
- 同一类型下的所有对象的集合在reset风格API下叫集合，在Kubernetes中被称之为列表（list）。
- xx

Kubernetes api账户有两类，真实的账户（人用的账户）userAccount与==Pod客户端serviceAccount==（Pod连接apiserver使用的账户）。 

每个Pod无论你定义与否，都会挂载一个存储卷，这就是Podservice认证时的认证信息，通过secret定义存储卷的方式关联到Pod上，从而使Pod内运行的应用，通次secret保存的认证信息，来连接apiserver并完成认证。  

在每一个名称空间当中都存在一个默认的secret，`default-token-xxx`，这是让当前名称空间当中所有的Pod资源试图去连接apiserver时，隐藏的、预制的一个认证信息。所以所有的Pod都能直接连接apiserver。此secret所包含的认证信息仅仅是获取当前Pod自身的属性。

### 给Pod增加自定义服务账号。

serviceAccount也属于标准的Kubernetes资源，可以自行创建serviceAccount，由自定义Pod使用serviceAccountName去加载自定义serviceAccount。serviceAccount是一个可以使用命令行创建的简单资源对象，可以使用`kubectl create serviceaccount`，创建也可以使用资源清单进行创建。

语法

```
kubectl create serviceaccount {Name} -o yaml --dry-run
```

**serveraccount本身不具备权限，可以使用rbac对serviceaccount授予权限**

创建完之后会自动生成一个token信息，用于让sa连接至当前系统认证的信息。注：认证不代表权限，可以登录、认证到Kubernetes但是做不了其余事情。所有的的操作权限靠授权实现的。


在创建Pod中使用自定义的sa
```yaml
spec:
  containers:
  serviceAccountName: admin
```

```
Volumes:
  default-token-4pj85:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-4pj85
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  &lt;none&gt;
```




在定义好secret后，在定义Pod时，使用 `imagePullSecrets` 指明用哪个secret对象，secret对象中包含了认证私有regsi的账号和密码。

在Pod当中也可以不使用imagePullSecrets来告诉Pod如何去下载镜像文件。而可以直接使用serviceAccountName。serviceAccountName相当于指定一个sa账号，而sa账号是可以附带认证到私有regsiry的secret信息的。Pod通过sa的`Image pull secrets`也能完成资源镜像下载时的认证。这样就不会在Pod资源清单中泄露出去secret使用的什么信息。

使用kubectl describe sa admin


&gt; **Pod获取私有镜像时的两种认证方式**

- 在Pod上直接使用`imagePullSecrets`字段指定认证使用的secret对象。
- 在Pod自定义serviceAccount，在serviceAccount附加此Pod获取镜像认证时使用的secret对象。

### kubectl创建配置文件



&gt; Reference
&gt;
&gt; [controlling access](https://kubernetes.io/docs/concepts/security/controlling-access/)

