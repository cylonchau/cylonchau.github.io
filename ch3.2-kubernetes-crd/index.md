# Kubernetes CRD


Kubernetes的主节点或控制面板当中主要有三个组件，其中apiserver是整个系统的数据库，借助于Cluster Store（etcd）服务，来实现所有的包括用户所期望状态的定义，以及集群上资源当前状态的实时记录等。

etcd是分布式通用的K/V系统 `KV Store` ，可存储用户所定义的任何由KV Store所支持的可持久化的数据。它不仅仅被apiserver所使用，如flannel、calico二者也需要以etcd来保存当前应用程序对应的存储数据。 任何一个分布式应用程序几乎都会用到一个高可用的存储系统。

apiserver将etcd所提供的存储接口做了高度抽象，使用户通过apiserver来完成数据存取时，只能使用apiserver中所内建支持的数据范式。在某种情况之下，我们所期望管理的资源或存储对象在现有的Kubernetes资源无法满足需求时。

 Operator本身是建构在StatefulSet以及本身的基本Kubernetes资源之上，由开发者自定义的更高级的、更抽象的自定义资源类型。他可借助于底层的Pod、Service功能，再次抽象出新资源类型。更重要的是，整个集群本身可抽象成一个单一资源。

为了实现更高级的资源管理，需要利用已有的基础资源类型，做一个更高级的抽象，来定义成更能符合用户所需要的、可单一管理的资源类型，而无需去分别管理每一个资源。

在Kubernetes之上自定义资源一般被称为扩展Kubernetes所支持的资源类型， 

- 自定义资源类型 CRD Custom Resource Definition
- 自定义apiserver
- 修改APIServer源代码，改动内部的资源类型定义

CRD是kubernetes内建的资源类型，从而使得用户可以定义的不是具体的资源，而是资源类型，也是扩展Kubernetes最简单的方式。



## Intorduction CRD

### 什么是CRD

在 Kubernetes API 中，resources 是存储 API 对象集合的endpoint。例如，内置 Pod resource 包含 Pod 对象的集合。当我们想扩展API，原生的Kubernetes就不能满足我们的需求了，这时 **CRD**  (`CustomResourceDefinition`) 就出现了。在 Kubernetes 中创建了 CRD 后，就可以像使用任何其他原生 Kubernetes 对象一样使用它，从而利用 Kubernetes 的所有功能、如安全性、API 服务、RBAC 等。

Kubernetes 1.7 之后增加了对 CRD 自定义资源二次开发能力来扩展 Kubernetes API，通过 CRD 我们可以向 Kubernetes API 中增加新资源类型，而不需要修改 Kubernetes 源码来创建自定义的 API server，该功能大大提高了 Kubernetes 的扩展能力。

## 创建 CRD

**前提条件**： Kubernetes 服务器版本必须不低于版本 1.16

再创建新的 `CustomResourceDefinition`（CRD）时，Kubernetes API 服务器会为指定的每一个版本生成一个 `RESTful` 的资源路径。（即定义一个Restful API）。CRD 可以是`namespace`作用域的，也可以是`cluster`作用域的，取决于 CRD 的 `scope` 字段设置。和其他现有的内置对象一样，删除一个`namespace`时，该`namespace`下的所有定制对象也会被删除。CustomResourceDefinition 本身是不受名字空间限制的，对所有名字空间可用。

例如，编写一个firewall port 规则：

```yaml
# 1.16版本后固定格式
apiVersion: apiextensions.k8s.io/v1
# 类型crd
kind: CustomResourceDefinition
metadata:
  # 必须为name=spec.names.plural &#43; spec.group
  name: ports.firewalld.fedoraproject.org
spec:
  # api中的group
  # /apis/&lt;group&gt;/&lt;version&gt;/&lt;plural&gt;
  group: firewalld.fedoraproject.org
  # 此crd作用于 可选Namespaced|Cluster
  scope: Namespaced
  names:
    # 名字的复数形式，用于api
    plural: ports
    # 名称的单数形式。用于命令行
    singular: port
    # 种类，资源清单类型
    kind: PortRule
    # 名字简写，类似允许 CLI 上较短的字符串匹配的资源
    shortNames: 
    - fp
  versions:
  # 定义版本的类型
  - name: v1
  # 通过 served 标志来启用或禁止
    served: true
  # 其中一个且只有一个版本必需被标记为存储版本
    storage: true
  # 自定义资源的默认认证的模式
    schema:
  # 使用的版本
      openAPIV3Schema:
        # 定义一个参数为对象类型
        type: object
        # 这个参数的类型
        properties:
        # 参数属性spec
          spec:
            # spec属性的类型为对象
            type: object
            # 对象属性
            properties:
              # spec属性name
              name:
              # 类型为string
                type: string
              port:
                type: integer
              isPermanent:
                type: boolean
```

需要注意的是v1.16版本以后已经 `GA`了，使用的是`v1`版本，之前都是`vlbeta1`，定义规范有部分变化，所以要注意版本变化。

这个地方的定义和我们定义普通的资源对象比较类似，我们说我们可以随意定义一个自定义的资源对象，但是在创建资源的时候，肯定不是任由我们随意去编写`YAML`文件的，当我们把上面的`CRD`文件提交给Kubernetes之后，Kubernetes会对我们提交的声明文件进行校验，从定义可以看出CRD是基于 [OpenAPIv3 schem](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.1.0.md) 进行规范的。当然这种校验只是**对于字段的类型进行校验**，比较初级，如果想要更加复杂的校验，这个时候就需要通过Kubernetes的admission webhook来实现了。关于校验的更多用法，可以前往官方文档查看。

创建一个crd类型资源

```yaml
apiVersion: &#34;firewalld.fedoraproject.org/v1&#34;
kind: PortRule
metadata:
  name: http-port
spec:
  name: &#34;nginx&#34;
  port: 80
  isPermanent: false
```

查看创建的crd

```
$ kubectl get t
NAME                                CREATED AT
firewallds.port.fedoraproject.org   2022-06-19T09:27:09Z
```



&gt; Reference
&gt;
&gt; [CRD](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
&gt;
&gt; [CRD Definition](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#customresourcedefinition-v1-apiextensions-k8s-io)


