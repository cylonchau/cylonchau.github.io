# Understand Kubernetes access control - RBAC



## Kubernetes API Object

在Kubernetes线群中，Kubernetes对象是持久化的实体（最终存入etcd 中的数据），集群中通过这些实体来表示整个集群的状态。前面通过`kubectl`来提交的资源清单文件，将我们的YAML文件转换成集群中的一个API对象的，然后创建的对应的资源对象。

Kubernetes API是一个以`JSON`为主要序列化方式的`HTTP`服务，除此之外支持`Protocol Buffers`序列化方式（主要用干集群内年件间的通信）。为了api的可扩展性，Kubemetes在不同的API路径（`/api/v1`或`/apis/batch`）下面支持了多个API版本，不同的API版本就味不同级别稳定性和支持。

- Alpha ：例如`v1Alpha`：默认情况下是禁用的，可以随时删除对功能的支持。
- Beta：例如 `v2beta1` 默认是启用的，表示代码已经经过了很好的测试，但是对象的语义可能会在施后的版本中以不兼咨的方式更改
- Stable：例如：`v1` 表示已经是稳定版本，也会出现在后续的很多版本中。

在Kubernetes集群中，一个API对象在Etcd 里的完整资源路径，是由：`group` （API组）、 `version` （API版本） 和 `Resource` API资源类型）三个部分组成。通过这种的结构，整个Kubernetes 中所有API对象，就可以用如下的树形结构表示出来：

![kube-api-1](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/kube-api-1.png)

### Kubernetes API Object的使用

API对象组成查看：`kubectl get --raw /`

通常，`KubernetesAPI`支持通过标准HTTP `P0ST`、`PUT`、`DELETE` 和 `GET` 在指定PATH路径上创建、更新、删除和检索操作，并使用JSON作为默认的数据交互格式。

如要创建一个Deployment对象，那YAML文件的声明就需：

```yaml
apiVersion: apps/v1 # 
kind: Deployment
```

`Deployment`就是这个API对象的资源类型（Resource），`apps`就是它的组（Group），`v1`就是它的版本（Version）。API Group、Version 和资源满唯一定义了一个HTTP路径，然后在`kube-apiserver ` 对这个url进行了监听，然后把对应的请求传递给了对应的控制器进行处理。

[API对象参考文档](https://k8s.mybatis.io)

## 授权插件分类

- Node 由节点来认证。

- ABAC 基于属性的访问控制，RBAC之前的授权控制的插件算法
- RBAC Role-based Access Control。 
- Webhook 基于http的回调机制来实现访问控制。

## RBAC

基于角色的访问控制可以理解为，角色（role）反而是授权的机制，完成了权限的授予、分配等。角色是指一个组织或者任务工作中的位置，通常代表一种权利、资格、责任等。在基于角色的访问控制中还有一种术语叫做 ==许可==（permission）。

简单来讲就如同上图描述，使用户去扮演这个角色，而角色拥有这个权限，所以用户拥有这个角色的权限。所以授权不授予用户而授予角色。

`RBAC` 使用 `rbac.authorization.k8s.io`API组来驱动鉴权操作，允许管理员通过 Kubernetes API 动态配置策略。

在 1.8 版本中，RBAC 模式是稳定的并通过 rbac.authorization.k8s.io/v1 API 提供支持。

### 启用RBAC

要使用用RBAC，需要在启动kube-apiserver时添加`--authorization-mode=RBAC` 参数。

### API概述

- ==`/apis/[group]/[version]/namespaces/[namespaces_name]/[kind][/object_id]`==

在Kubernetes当中的RBAC在实现授权时，无非就是定义标准的角色，在角色上绑定权限。使用户扮演角色。将这些概念体现为：
- **role** 标准的Kubernetes资源
    - operations  允许那些对象..
    - objects  执行那些操作..
- **rolebinding** 角色绑定
    - user 将那个用户(user account OR service account)...
    
    - role 绑定在那个角色上...
    
- **Rule**：规则是一组属于不同`API Group`资源上的一组操作的集合
- **Group**：用来关联多个账户，集群中有一些默认建的组，比如cluster-admin

- **Subject**：主题，对应集群中尝试操作的对象，集群中定义了3种类型的主题资源：
  - user account：用户，Kubernetes真正意义上User，而是使用证书的CN与O，加上kubeconfig上下文实现用户与组，这个用户是由外部独立服务进行管理的，对于用户的管理集群内部没有一个关联的资源对象，所以用户不能通过集群内部的API来进行管理。
  - service account：通过`KubernetesAPI`来管理的一些用户帐号，和namespace进行关联的，适用于集群内部运行的应用程序，需要通过API来完成权限认证，所以在集群内部进行权限操作。

> 在Kubern1etes之上资源分属于两种级别

- **cluster**

- **namespaces**

所以role和rolebinding是在名称空间级别，授予此名称空间范围内的许可权限的。除了role和rolebinding之外，集群还有另外两个组件：

- **cluster role**  集群角色。

- **cluster rolebinding** 集群角色绑定。

cluster role当中定义的权限是相对于多个名称空间共有，如果使用rolebinding绑定，这个权限被限制为 `用户仅能获取rolebinding所属名称空间上的所有权限`。

### 使用 kubeconfig 文件组织集群访问

​	在使用kubectl命令时是有使用配置文件的，配置文件是kubectl连接服务器认证文件。`kubectl config` 是专门用来管理kubectl的配置文件的。所有连接apiserver的客户端在认证时，如果基于配置文件来保存客户端的认证信息就应该将其配置配置为配置文件。

​	kubernetes组件除了apiserver都可以被称之问apiserver客户端。每个组件为了能够连接正确的集群。apiserver需提供正确的私钥、证书等认证时使用的信息需要将这些信息保存为一个配置文件，此配置文件被叫做`kubeconfig`。

​	`kubeconfig `文件可以用来组织有关集群、用户、命名空间和身份认证机制的信息。`kubectl` 命令行工具使用 kubeconfig 文件来与集群的 API 服务器进行通信。

​	默认情况下 `kubeconfig` 在 `$HOME/.kube` 目录下查找名为 `config` 的文件。可以设置 环境变量`KUBECONFIG` 或者设置 `kubectl --kubeconfig` 来选择指定的kubeconfig

#### context

​	kubeconfig 的 *context* ，定义对访问参数进行分组。每个context都有三个参数：`cluster`、`namespace `和 `user`。默认情况下，`kubectl` 命令行工具使用 `namespace` 参数设置的值与集群进行通信。默认为defaul。

  -   `kubectl config get-contexts` 查看拥有上下文	
  - `kubectl config use-context` 选择上下文
  - `kubectl config set-credentials` 设置一个用户项

  reference

  [set-credentials](http://kubernetes.kansea.com/docs/user-guide/kubectl/kubectl_config_set-credentials/)

  [kubeconfig](https://kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

### 角色的创建与管理

​	Kubernetes的访问权限主要可以梳理为几个步骤

- 创建用户：使用CA签发用户认证证书作为用户名称。
- 创建权限组：创建role或clusterrole确定操作权限。
- 绑定用户和权限组：创建rolebinding或clusterrolebinding将权限绑定在用户上。
- 使用用户访问验证：切换kubeconfig。

  `role` `cluster role` `rolebinding` `cluster rolebinding`都是标准的Kubernetes资源，可以通过`kubectl explain`查看，或`kubectl create role` 创建

  **role在限制资源范围时有三种方式：**

  - resources 资源类别，允许对这些类所有资源支持授权 操作。
  - resource Names 资源名称，表示对此类别当中，某个或某些特定资源执行操作。
  - Non-Resource URLs 非资源url，是一些不能定义为对象的资源，在Kubernetes中通常表示对某些资源所执行的一种操作，或某种特殊操作。

```bash
ROLENAME=default-admin
NS=default
kubectl create clusterrole ${ROLENAME} \
--verb=get,list \
--resource=pods,deployments \
-o yaml \
-n default \
--dry-run=client 
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-list
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - deployments
  verbs:
  - get
  - list
- apiGroups: # 表示对哪些api群组内的资源做操作。
  - apps
```

将权限与角色绑定

```bash
USERNAME=scott
ROLENAME=default-admin
NS=default
kubectl create clusterrolebinding ${ROLENAME} \
--clusterrole=${ROLENAME} \
--group=${ROLENAME} \
-n ${NS} \
-o yaml \
--dry-run=client
```


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: podlist:testrbac
  namespace: kube-system
roleRef: # 引用那个role
  apiGroup: rbac.authorization.k8s.io # 那个api之内的
  kind: Role # 哪一类
  name: pod-list # role名称，为了避免引用的是cluster role必须使用 此方式来定义
subjects: # 动作的执行主题
# 对于user group是 rbac.auth... 对于serviceaccount是""
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: testrbac # user并不是单独存在的用户资源
```

创建用户

```
USERNAME=cylon
openssl genrsa -out ${USERNAME}.key 2048
o=default-admin

openssl req -new \
-key ${USERNAME}.key \
-out ${USERNAME}.csr \
-subj "/CN=${USERNAME}/O=${o}" \
-days 3650

openssl x509 -req \
-in ${USERNAME}.csr \
-CA ca.crt \
-CAkey ca.key \
-out ${USERNAME}.crt \
-days 360
```

配置kubeconfig

```bash
USERNAME=cylon
kubectl config set-credentials ${USERNAME} \
--client-certificate=/etc/kubernetes/pki/${USERNAME}.crt \
--client-key=/etc/kubernetes/pki/${USERNAME}.key \
--embed-certs=true  \
--kubeconfig=${USERNAME} # 输出到指定的配置文件，若不指定写入KUBECONFIG环境变量指定的路径
```

设置上下文

```bash
USERNAME=cylon
kubectl config set-context ${USERNAME}@kubernetes \
--cluster=kubernetes \
--user=${USERNAME}
--kubeconfig=${USERNAME}
```

设置集群

```
kubectl config set-cluster k8s \
--certificate-authority=/etc/kubernetes/pki/ca.crt \
--embed-certs=true  \
--server=https://127.0.0.1:6443 \
--kubeconfig=${USERNAME}
```

在RBAC上进行授权时，允许我们存在**3类**组件 useraccount group serviceaccount

- user 授权绑定时，**`cluster rolebinding`** 或 **`rolebinding`** 都可以绑定在user上，也可以绑定在group上，还可以绑定在service account上。

绑定在用户上，表示只授权一个用户扮演相关角色。

绑定在在一个组上表示授权组内所有用户都在一个角色。所以想一次授权多个用户在一个名称空间中拥有一个权限可以定义为组。授权时做组授权。


如果任何一个Pod在启动时以serviceaccount name作为使用的serviceAccount，Pod中的应用程序就拥有了它所授予的权限。

创建Pod时可以给Pod指明一个属性。serviceAccount

