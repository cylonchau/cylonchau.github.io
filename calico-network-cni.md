# calico network cni


Calico针对容器、虚拟机的开源网络和网络安全解决方案。是纯三层的数据中心网络方案。

Calico在每一个计算节点利用Linux Kernel实现了一个高效的虚拟路由器`vRouter`来负责数据转发，而每个`vRouter`通过BGP协议负责把自己上运行的workload的路由信息向整个Calico网络内传播。（小规模部署可以直接互联 `BGP full mesh`，大规模下可通过指定的`BGP route reflector`来完成）。 这样保证最终所有的`workload`之间的数据流量都是通过IP路由的方式完成互联的。Calico节点组网可以直接利用数据中心的网络结构（无论是L2或者L3），不需要额外的`NAT`，隧道或者`Overlay Network`。

Calico还基于`iptables`还提供了丰富而灵活的网络`Policy`，保证通过各个节点上的`ACLs`来提供Workload的多租户隔离、安全组以及其他可达性限制等功能。

### calico组件

在Kubernetes平台之上`calico/node`容器会通过DaemonSet部署到每个节点，并运行三个守护程序：

- Felix：用于管理路由规则，负责状态上报。
- BIRD：BGP的客户端，用于将Felix的路由信息加载到内核中，同时负责路由信息在集群中的分发。
- confd：用于监视Calico存储（etcd）中的配置变更并更新`BIRD`的配置文件。







calicoctl使用问题

```
Failed to create Calico API client: invalid configuration: no configuration has been provided
```

默认情况下，`calicoctl` 将使用位于的默认`KUBECONFIG`从 Kubernetes APIServer 读取`$(HOME)/.kube/config` 。

如果默认的 `KUBECONFIG` 不存在，或者想从指定的存储访问信息，则需要单独配置。

```bash
export DATASTORE_TYPE=kubernetes
export DATASTORE_TYPE=etcdv3
export KUBECONFIG=~/.kube/config
```

[reference for](https://docs.projectcalico.org/getting-started/clis/calicoctl/configure/)

## calico 安装配置

开始前准备



确定calico数据存储

Calico同时支持kubernetes api和etcd数据存储。官方给出的建议是在本地部署中使用**K8S API**，仅支持Kubernetes模式。而官方给出的etcd则是混合部署（Calico作为Kubernetes和OpenStack的网络插件运行）的最佳数据存储。

使用kubernetes api作为数据存储的安装

```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

修改Pod CIDR

Calico默认的Pod CIDR使用的是`192.168.0.0/16`，这里一般使用与controller-manager中的`--cluster-cidr` 保持一,取消资源清单内的 `CALICO_IPV4POOL_CIDR`变量的注释，并将其设置为与所选Pod CIDR相同的值。

calico的IP分配范围

Calico IPAM从`ipPool`分配IP地址。修改Pod的默认IP范围则修改清单`calico.yaml`中的 `CALICO_IPV4POOL_CIDR`

配置Calico的 `IP in IP`

默认情况下，Calico中的IPIP已经禁用，这里使用的v3.17.2 低版本默认会使用IPIP

要开启IPIP mode则需要修改配置清单内的 `CALICO_IPV4POOL_IPIP` 环境变量改为 `always`



修改secret

```yaml
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat &lt;file&gt; | base64 -w 0
  # etcd的ca
etcd-ca: # 填写上面命令编码后的值
# etcd客户端key
etcd-key: # 填写上面命令编码后的值
# etcd客户端访问证书
etcd-cert: # 填写上面命令编码后的值
```

修改configMap

```yaml
  etcd_endpoints: &#34;https://10.0.0.6:2379&#34;
  # If you&#39;re using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  etcd_ca: &#34;/calico-secrets/etcd-ca&#34;
  etcd_cert: &#34;/calico-secrets/etcd-cert&#34;
  etcd_key: &#34;/calico-secrets/etcd-key&#34;
```

在卷装载中设置440将解决此问题

`/calico-secrets/etcd-cert: permission denied`

```
2021-02-08 02:15:10.485 [INFO][1] main.go 88: Loaded configuration from environment config=&amp;config.Config{LogLevel:&#34;info&#34;, WorkloadEndpointWorkers:1, ProfileWorkers:1, PolicyWorkers:1, NodeWorkers:1, Kubeconfig:&#34;&#34;, DatastoreType:&#34;etcdv3&#34;}
2021-02-08 02:15:10.485 [FATAL][1] main.go 101: Failed to start error=failed to build Calico client: could not initialize etcdv3 client: open /calico-secrets/etcd-cert: permission denied
```

找到资源清单内的对应容器（`calico-kube-controllers`）的配置。

```yaml
volumes:
# Mount in the etcd TLS secrets with mode 400.
# See https://kubernetes.io/docs/concepts/configuration/secret/
- name: etcd-certs
  secret:
  secretName: calico-etcd-secrets
  defaultMode: 0400 # 改为0440
```

使用单独的etcd作为calico数据存储还需要修改calicoctl数据存储访问配置

`calicoctl` 在默认情况下，查找配置文件的路径为`/etc/calico/calicoctl.cfg`上。可以使用`--config`覆盖此选项默认配置。

如果`calicoctl`无法获得配置文件，将检查环境变量。

```yaml
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: etcdv3
  etcdEndpoints: &#34;https://10.0.0.6:2379&#34;
  etcdCACert: |
    # 这里填写etcd ca证书文件的内容，无需转码base64
  etcdCert: |
    # 这里填写etcd client证书文件的内容，无需转码base64
  etcdKey: |
    # 这里填写etcd client秘钥文件的内容，无需转码base64
```

&gt; reference：
&gt;
&gt; [Secret permission denied](https://github.com/kubernetes/website/issues/25587)
&gt;
&gt; [configuration calicoctl](https://docs.projectcalico.org/getting-started/clis/calicoctl/configure/overview)





```
eb  7 21:25:13 master01 etcd: recognized environment variable ETCD_NAME, but unused: shadowed by corresponding flag
Feb  7 21:25:13 master01 etcd: unrecognized environment variable ETCD_SERVER_NAME=hk.etcd
Feb  7 21:25:13 master01 etcd: recognized environment variable ETCD_DATA_DIR, but unused: shadowed by corresponding flag
Feb  7 21:25:13 master01 etcd: recognized environment variable ETCD_LISTEN_CLIENT_URLS, but unused: shadowed by corresponding flag
Feb  7 21:25:13 master01 etcd: etcd Version: 3.3.11
Feb  7 21:25:13 master01 etcd: Git SHA: 2cf9e51
Feb  7 21:25:13 master01 etcd: Go Version: go1.10.3
Feb  7 21:25:13 master01 etcd: Go OS/Arch: linux/amd64
Feb  7 21:25:13 master01 etcd: setting maximum number of CPUs to 2, total number of available CPUs is 2
Feb  7 21:25:13 master01 etcd: the server is already initialized as member before, starting as etcd member...
Feb  7 21:25:13 master01 etcd: peerTLS: cert = /etc/etcd/pki/peer.crt, key = /etc/etcd/pki/peer.key, ca = , trusted-ca = /etc/etcd/pki/ca.crt,
 client-cert-auth = true, crl-file =
Feb  7 21:25:13 master01 etcd: listening for peers on https://10.0.0.5:2380
Feb  7 21:25:13 master01 etcd: listening for client requests on 10.0.0.5:2379
Feb  7 21:25:13 master01 etcd: panic: freepages: failed to get all reachable pages (page 3471766746605708656: out of bounds: 1633)
```

集群节点损坏

```
panic: freepages: failed to get all reachable pages (page 3471766746605708656: out of bounds: 1633)
```

这是k8s不支持当前calico版本的原因, calico版本与k8s版本支持关系可到calico官网查看:

```bash
error: unable to recognize &#34;calico.yaml&#34;: no matches for kind &#34;PodDisruptionBudget&#34; in version &#34;policy/v1&#34;
```

配置SW

```
system-view
sysname SW1
vlan batch 10 20 30

interface GigabitEthernet0/0/1
port link-type trunk
port trunk allow-pass vlan 10 20 30

interface GigabitEthernet0/0/2
port link-type trunk
port trunk allow-pass vlan 10 20 30

interface GigabitEthernet0/0/3
port link-type trunk
port trunk allow-pass vlan 10 20 30
```

配置路由器间的ospf

```
interface l0
ip address 1.1.1.1 32
quit
ospf router-id 1.1.1.1
area 0
network 1.1.1.1 0.0.0.0
network 10.0.0.253 0.0.0.0
dis this

interface l0
ip address 2.2.2.2 32
quit
ospf router-id 2.2.2.2
area 0
network 2.2.2.2 0.0.0.0
network 10.0.0.254 0.0.0.0
dis this
```

配置两个k8s节点与路由器之间的bgp

```
system-view
sysname R1

interface GigabitEthernet0/0/0
ip address 10.0.0.253 24
dis this
quit

bgp 64512
router-id 10.0.0.253
peer 10.0.0.5 as-number 64512
peer 10.0.0.5 reflect-client
dis ip interface brief



system-view
sysname R2

interface GigabitEthernet0/0/0
ip address 10.0.0.254 24
dis this
quit

bgp 63400
router-id 10.0.0.254
peer 10.0.0.6 as-number 63400
peer 10.0.0.6 reflect-client
dis ip interface brief

bgp 64512
router-id 10.0.0.253 
peer 2.2.2.2 as-number 63400


bgp 63400
router-id 10.0.0.254
peer 1.1.1.1 as-number 64512
```


