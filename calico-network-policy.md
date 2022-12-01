# calico network policy


## 什么是网络策略

在Kubernetes平台中，要实现零信任网络的安全架构，Calico与istio是在Kubernetes集群中构建零信任网络必不可少的组件。

而建立和维护整个集群中的“零信任网络”中，网络策略的功能在操作上大致可以总结为**使用资源配置模板来管理控制平面数据流**。说白了讲网络策略就是用来控制Pod间流量的规则。

## 在Calico中如何编写网络策略

要使用网络策略就需要先了解Calico功能：**NetworkPolicy**和**GlobalNetworkPolicy**。

`NetworkPolicy`资源，简称`np`；是命名空间级别资源。规则应用于与标签选择器匹配的endpoint的集合。

`GlobalNetworkPolicy`资源，简称 `gnp`/`gnps`与`NetworkPolicy`功能一样，是整个集群级别的资源。

`GlobalNetworkPolicy` 与 `NetworkPolicy`资源的管理也与calico的部署方式有关，使用etcd作为存储时，资源的管理只能使用 `calicoctl`进行管理

###  NetworkPolicy与GlobalNetworkPolicy的构成

```yaml
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-tcp-90
spec:
  selector: app == 'envoy' # 应用此策略的endpoint
  types: # 应用策略的流量方向
    - Ingress 
    - Egress
  ingress: # 入口的流量规则
    - action: Allow # 流量的行为
      protocol: ICMP # 流量的协议
      notProtocol: TCP # 匹配流量协议不为 值 的流量 
      source: # 流量的来源 src与dst的匹配关系为 与，所有的都生效即生效
        nets: # 有效的来源IP
        selector: # 标签选择器
        namespaceSelector: # 名称空间选择器
        ports: # 端口
        - 80 # 单独端口
        - 6040:6050	# 端口范围
      destination: # 流量的目标
  egress: # 出口的流量规则
    - action: Allow
  serviceAccountSelector: # 使用与此规则的serviceAccount
```

### NetworkPolicy使用

实例：允许6379流量可以被 `role=frontend`的pod访问

```yaml
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-tcp-6379
  namespace: production
spec:
  selector: role == 'database'
  types:
  - Ingress
  - Egress
  ingress:
  - action: Allow
    metadata:
      annotations:
        from: frontend
        to: database
    protocol: TCP
    source:
      selector: role == 'frontend'
    destination:
      ports:
      - 6379
  egress:
  - action: Allow
```

实例：禁止ICMP流量

```yaml
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-tcp-90
spec:
  selector: app == 'netbox'
  types:
    - Ingress
    - Egress
  ingress:
    - action: Deny
      protocol: ICMP
  egress:
    - action: Deny
      protocol: ICMP
```

实例：禁止访问指定服务

```yaml
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-tcp-90
spec:
  selector: app == 'netbox'
  types:
    - Ingress
    - Egress
  ingress:
    - action: Allow
  egress:
    - action: Deny
      destination:
        selector: app == 'envoy'
```

## GlobalNetworkPolicy

GlobalNetworkPolicy与NetworkPolicy使用方法基本一致，只是作用域的不同，并且可以应用很多高级的网络策略：

- [转发流量](https://docs.projectcalico.org/security/host-forwarded-traffic)
- [防御DoS](https://docs.projectcalico.org/security/defend-dos-attack)
- ....

**GlobalNetworkPolicy** 中提供了一个preDNAT的功能，是kube-proxy对Node port的端口和IP的流量DNAT到所对应的Pod中的时候，为了既允许正常的ingress流量，又拒绝其他的ingress流量，这个时候必须要在DNAT前生效，这种情况需要使用**preDNAT**。

**preDNAT** 适用的条件是，流量仅为ingress并且在DNAT之前。



> **reference**
>
> [NetworkPolicy.spec](https://docs.projectcalico.org/reference/resources/networkpolicy#spec)
>
> [NetworkPolicy.spec.ingress|egress](https://docs.projectcalico.org/reference/resources/networkpolicy#rule)
>
> [NetworkPolicy.spec.ingress.src|dst](https://docs.projectcalico.org/reference/resources/networkpolicy#entityrule)
>
> [globalnetworkpolicy](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy)


