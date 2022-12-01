# Understand Kubernetes - configMap


`secret`、`configMap`特殊类型的存储卷，多数情况下不是为Pod提供存储空间来用的，而是给管理员或用户提供了从集群外部向Pod内部应用注入配置信息的方式。

工作实现

### configMap 

在集群内部存在一个名称空间，在名称空间当中拥有一个可正常运行的Pod，当镜像启动时使用的配置文件在做镜像之前就确定了，并且做完镜像就不能修改了。除非在做镜像时使用entryPoint脚本去接受用户启动容器时传入环境变量进来，将环境变量的数据替换到配置文件中去，从而使应用程序在启动之前就能获得一个新的配置文件而后得到新的配置。当需要修改配置文件时是很麻烦的。而配置中心只需将集中的配置文件修改，并通知给相应进程，让其重载配置文件。而Kubernetes的应用也存在此类问题，当配置文件修改后就需要更新整个镜像。因此无需将配置信息写死在镜像中。而是引入一个新的资源，这个资源甚至是整个Kubernetes集群上的一等公民（标准的K8S资源）。这个资源被叫做configMap

configMap当中存放的配置信息，随后启动每一个Pod时，Pod可以共享使用同一个configMap资源，这个资源对象可以当存储卷来使用，也可以从中基于环境变量方式从中获取到一些数据传递给环境变量，注入到容器中去使用。 因此configMap扮演了Kubernetes中的配置中心的功能。但是configMap是明文存储数据的。因此和configMap拥有同样功能的标准资源`secret`就诞生了。与configMap所不同之处在于，secret中存放的数据是用过编码机制进行存放的。

**核心作用**：让配置信息从镜像中解耦，从而增强应用的可移植性与复用性。使一个镜像文件可以为应用程序运行不同配置的环境而工作。简单来讲，一个configMap就是一系列配置数据的集合。这些数据可以注入到Pod对象中的容器所使用。

在configMap中，所有的配置信息都保存为`key value`格式。V只是代表了一段配置信息，可能是一个配置参数，或整个配置文件信息都是没有问题的。


> 配置容器化应用的方式

- 自定义命令行参数  `args []`
- 把配置文件直接陪进镜像；
- 环境变量
    - Cloud Native的应用程序一般可直接通过环境变量加载配置
    - 通过`entrypoint`脚本来预处理变量为配置文件中的配置信息。
- 存储卷


> 配置文件注入方式：

- 将configMap做存储卷
- 使用env




docker config
***
- **contioners**
    - **env**
        - `name` 变量名
        - `value` 变量值
        - `valueFrom` 数据不是一个字符串，而是引用另外一个对象将其传递给这个变量。
            - `configMapKeyRef` configMap中的某个键
            - `fieldRef` 某个字段。此资源可以是Pod自身的字段。如`metadata.labels` `status.hostIP` `status.podIP`
            - `resourceFieldRef` 资源需求和资源限制。
            - `secreKeyRef` 引用secre
***

configMap无需复杂描述，因此没有spec字段


```
apiVersion
kind
data
binaryData 一般情况下data与binaryData只使用其中一种。
```

创建简单的configMap还可以使用  `kubectl create configMap`来创建。如需要长期使用，可以定义为配置清单文件。

```
kubectl create configmap nginx-config \
--from-literal=nginx_port=80 \
--from-literal=servername=test.com
```

使用文件创建

```
kubectl create configmap nginx-conf --from-file=www.conf
```

将configMap注入到Pod中。

方式1：使用环境变量方式注入

```yaml
env:
- name: NGINX_SERVER_PORT
  valueFrom:
    configMapKeyRef:
      name: nginx-config
      key: nginx_port
      optional # 如为true表示必须拥有此key
- name: NGINX_SERVER_NAME
  valueFrom:
    configMapKeyRef:
      name: nginx-config
      key: server_name
```

当使用环境变量方式注入时，只在系统启动时有效，如使用存储卷方式定义的，是可以实时更新的。


方式2：

```yaml
containers:
- name: nginx-configMap
  volumeMounts:
  - name: nginxconfig
    mountPaht: /etc/nginx/con.d/
    readOnly: true # 不需要容器去修改他的内容
volumes:
- name: nginxconfig
  configMap: # volume类型
    name: nginx-conf
```

k8s  节点为了运行Pod，而获取镜像，镜像如果托管在必须认证才能获取的私有仓库上时。node节点上的kubelet需能自动完成认证。



docker-registry docker私有仓库认证信息使用
generic
tls 证书私钥

spec
- imagePullSecrets Pod在创建时，如果要连到私有仓库需要做认证，此处的secret包含了让kubelet去连接私有仓库的账号和密码。此账号密码是secret对象提供的， 必须是专用对象。


创建方法

```
kubectl create secret type(docker-registry|generic|tls) Name 
```

查询
```
kubectl get secret passwd -o yaml
```

![427f4e23.png](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/427f4e23.png)


env方式载入secret

```yaml
env:
- name: MYSQL_ROOT_PASSWD
  valueFrom:
    secretKeyRef:
      name: root-pwd
      key: passwd
      optional # 如为true表示必须拥有此key
```
