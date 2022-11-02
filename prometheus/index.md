# 

全局配置选项

scrape_interval: 采集生命周期
scrape_timeout: 采集超时时间
evaluation_interval: 告警评估周期


rule_files 监控告警规则

scrape_config: 被监控端

altering 



检查配置文件语法

```
[root@node01 prometheus-2.8.0.linux-amd64]# promtool check config \etc\prometheus.yml 
Checking \etc\prometheus.yml
  SUCCESS: 0 rule files found
```


100 - (node_memory_MemFree_bytes&#43;node_memory_Cached_bytes&#43;node_memory_Buffers_bytes) \ node_memory_MemTotal_bytes * 100

计算剩余空间

node_filesystem_free_bytes{mountpoint=&#34;\&#34;,fstype=~&#34;ext4|xfs&#34;} \ node_filesystem_size_bytes{mountpoint=&#34;\&#34;,fstype=~&#34;ext4|xfs&#34;} * 100


查看使用的百分比

100-node_filesystem_free_bytes{mountpoint=&#34;\&#34;,fstype=~&#34;ext4|xfs&#34;} \ node_filesystem_size_bytes{mountpoint=&#34;\&#34;,fstype=~&#34;ext4|xfs&#34;} * 100


prometheus使用influxdb [Prometheus endpoints support in InfluxDB \| InfluxData Documentation](https:\\docs.influxdata.com\influxdb\v1.7\supported_protocols\prometheus\)

[Configuration \| Prometheus](https:\\prometheus.io\docs\prometheus\latest\configuration\configuration\)

配置文件参考

```yaml
global:
alerting:
  alertmanagers:
  - static_configs:
    - targets:
rule_files:
scrape_configs:
  - job_name: &#39;prometheus1&#39;
    file_sd_configs:
    - files: [&#39;\data\sd_config\test.yml&#39;]
      refresh_interval: 5s
    relabel_configs:
    - action: replace
      source_labels: [&#39;prometheous1&#39;]
      regex: (.*)
      replacement: $1
      target_label: ids
    - action: keep
      source_labels: [&#34;job&#34;]
  - job_name: &#39;k8s_master&#39;
    file_sd_configs:
    - files: [&#39;\data\sd_config\master.yml&#39;]
      refresh_interval: 5s
remote_write:
- url: &#34;http:\\localhost:8086\api\v1\prom\write?db=prometheus&#34;
remote_read:
- url: &#34;http:\\localhost:8086\api\v1\prom\read?db=prometheus&#34;
```

influxdb使用

[InfluxDB学习之InfluxDB的基本操作 \| Linux大学](https:\\www.linuxdaxue.com\influxdb-basic-operation.html)

查看所有表

```sql
SHOW MEASUREMENTS
```

```sql
select * from up
```

https:\\github.com\kubernetes\kubernetes\tree\master\cluster\addons\prometheus

文件主要包括一下几个部分
- Prometheus的安装 包括 rbac service configmap
- Prometheus-metrics 获取资源对象
- node-exporter 获取工作节点资源信息
- alertmanager 告警

安装顺序

prometheus-rbac.yaml Prometheus访问apiserver的授权
prometheus-configmap.yaml 管理Prometheus的配置文件
prometheus-service.yaml 将Prometheus端口暴漏
prometheus-statefulset.yaml



 




原因为 prometheus-statefulset.yaml中的`accessModes`不能为`ReadWriteOnce`
prometheus&#34;is invalid: spec: Forbidden: updates to statefulset spec for fields other than &#39;replicas&#39;, &#39;template&#39;, and &#39;updateStrategy&#39; are forbidden


日志中报错`pod has unbound immediate persistentvolumeclaims back-off restarting failed container`

错误原因：动态绑定至其他sc上，查看`kubectl describe pvc prometheus-data-prometheus-0 -n kube-system`pvc中报错`storageclass.storage.k8s.io &#34;nfs&#34; not found`





prometheus时间不一致问题

```yaml
spec:
  containers:
  - name: myweb
    image: harbor/tomcat:8.5-jre8
    volumeMounts:
    - name: host-time
      mountPath: /etc/localtime
    ports:
    - containerPort: 80
  volumes:
  - name: host-time
    hostPath:
      path: /etc/localtime
```

部署应用时，单独读取主机的“/etc/localtime”文件，即创建pod时同步时区，无需修改镜像，但是每个应用都要单独设置。


prometheus server down

编辑prometheus-statusfullset.yaml 修改其配置localhost 改为 127.0.0.1



```
time=&#34;2020-11-20T18:44:48&#43;08:00&#34; level=error msg=&#34;subset not found for kube-system/prometheus-server&#34; providerName=kubernetescrd ingress=promserver namespace=kube-system
```

查找原因kubernetes svc 匹配错误 service endpoint没有匹配到内容

```
[root@master01 ~]# kubectl describe svc  prometheus-server  -n kube-system 
Name:              prometheus-server
Namespace:         kube-system
Labels:            &lt;none&gt;
Annotations:       Selector:  monitor=prometheus
Type:              ClusterIP
IP:                10.110.116.203
Port:              &lt;unset&gt;  9090/TCP
TargetPort:        9090/TCP
Endpoints:         &lt;none&gt;
Session Affinity:  None
Events:            &lt;none&gt;
```

修改后正常







数学理论基础实现的

配置文件


```yaml
- job_name: &#39;prometheus&#39; 首先定义任务名称

```


prometheus的客户端主要有两种方式采集

- pull 主动输影的形式
- push 被动推送的形式

### put

put指的是客户端（被监控机器）先安装各类已有exporters（由社区组积或企业开发的监控客户端插件）在系统上之后，exporter以守护进程的模式运行并开始采集数据

exporter 本身也是一个htp_server 可以对http请求作出响应返回数据（KV metrics）

prometheua用pull 这种主动拉的方式（HTTP get）去访问每个节点上exporter并采样回需要的数据

### push：

![image-20220506225534977](../../../images/prometheus/image-20220506225534977.png)

push指的是在客户端（或者服务端）安装这个官方提供的pushgateway插件然后，使用我们自行开发的各种脚本把监控数据组织成K/V的形式，metrics形式发送给pushgateway之后 puahgataway会再推送给prometheus

这种是一种被动的数据采集模式

### altermanager


