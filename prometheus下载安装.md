# 

安装prometheus之前，必须先安装ntp时间同步，prometheus T_S对系统时间的准确性要求很高，必须保证本机时间实时同步。

## prometheus下载

prometheus解压安装之后，默认自带一个基本的配置文件。

./prometheus 启动参数

```yaml
--config.file=&#34;prometheus.yml&#34; # 指定配置文件，默认当前目录下prometheus.yml
--web.listen-address=&#34;0.0.0.0:9090&#34;  # 指定监听的端口，默认9090
```

默认配置文件



```yaml
global:  # 全局定义
  scrape_interval:     15s  # prometheus自定义数据采集频率，抓取采集数据的时间间隔，默认每15秒去被监控主机上采样一次。
  evaluation_interval: 15s # 监控数据规则的评估频率
alerting: 
  alertmanagers:
  - static_configs:
    - targets:
rule_files:
# prometheus 自身配置
scrape_configs: # 抓取数据的配置
  - job_name: &#39;prometheus&#39; 
    static_configs:
    - targets: [&#39;localhost:9090&#39;,&#34;api01:9100&#34;,&#39;10.0.0.3:9100&#39;] 
    # 定义监控项，targets可以并列写多个节点。
    # 节点机器名称必须在prometheus服务器hosts文件中可以解析到
```

### 搭建被监控节点exporter

node_exporter是一个以http_server方式运行在后台，并且持续不断采集Linux系统中各种操作系统本身相关的监控插件的程序，其采集量是很大很全的，往往默认的采集项目超过实际需求


### prometheus启动参数

- `--web.read-timeout=5m` 请求链接的最大等待时间。防止太多空闲链接。
- `--web.max-connections=512` 最大连接数
- `--storage.tsdv.retention=15d` 在存储中保存多久的数据。
- `--storage.tsdb.path=data` 存储数据的路径
- `--query.timeout=2m`
- `-query.max-concurrency=20` 







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
