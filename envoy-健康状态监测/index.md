# Envoy 健康状态监测


### 官方文档

[healthy_check](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/health_check.proto)	

Envoy的服务发现并未采用完全一致的机制，而是假设主机以最终一致的方式加入或离开网格，它结合主动健康状态检查机制来判定集群的健康状态；

- 健康与否的决策机制以完全分布式的方式进行，因此可以很好地应对网络分区；
- 为集群启用主机健康状态检查机制后，Envoy基于如下方式判定是否路由请求到一个主机；
- 发现失败，单健康检查正常，此时，无法添加新主机，但现有主机将继续正常运行。

| Discovery Status | Health Check OK | Health Check Failed  |
| ---------------- | --------------- | -------------------- |
| Discovered       | Route           | Dont&#39;t Route         |
| Absent           | Route           | Don&#39;t Route / Delete |

### 故障处理机制

Envoy提供了一系列开箱即用的故障处理机制：

- 超时（timeout）
- 有限次数的重试，并支持可变的重试延迟
- 主动健康检查与异常探测
- 连接池
- 断路器

所有这些特性，都可以在运行时动态配置；结合流量管理机制，用户可为每个服务/版本定制所需的故障恢复机制；

### 健康状态监测

健康状态检测用于确保代理服务器不会将下游客户端的请求代理至工作异常的上游主机；

Envoy支持两种类型的健康状态检测，二者均基于集群进行定义：

**主动检测**（Active Health Checking）：Envoy周期性地发送探测报文至上游主机，并根据其响应判断其健康状态；Envoy目前支持三种类型的主动检测：

- HTTP：向上游主机发送HTTP请求报文
- L3/L4：向上游主机发送L3/L4请求报文，基于响应的结果判定其健康状态，或仅通过连接状态进行判定；
- Redis：向上游的redis服务器发送Redis PING；

**被动检测**（Passive Health Checking）：Envoy通过异常检测（Outlier Detection）机制进行被动模式的健康状态检测；

目前，仅htp router、tcp proxy和redis proxy三个过滤器支持异常值检测；

Envoy支持以下类型的异常检测：

- 连续5XX：意指所有类型的错误，非htp router过滤器生成的错误也会在内部映射为5xx错误代码；
- 连续网关故障：连续5XX的子集，单纯用于http的502、503或504错误，即网关故障；
- 连续的本地原因故障：Envoy无法连接到上游主机或与上游主机的通信被反复中断；
- 成功率：主机的聚合成功率数据阀值；



#### 主动健康状态监测

集群的主机健康状态检测机制需要显式定义，否则，发现的所有上游主机即被视为可用；

定义语法

```yaml
clusters:
- name:
  ..
  load_assignment:
  endpoints:
  - lb_endpoints:
    - endpoint:
      health_check_config:
        port_value: .. # 自定义健康状态监测是使用的端口。
  ...
  health_checks:
  - timeout: ... # 超时时长
    interval: ... # 时间间隔
    initial_jitter: ...  # 初始监测时间点散开量，以毫秒为单位
    interval_jitter: ..  # 间隔监测时间点散开量，以毫秒为单位
    unhealthy_threshold: ... # 将主机标记为不建行状态监测阈值，即至少多少次不健康的监测后才将其标记为不可用
    healty_threshold: ...    # 将主机标位健康状态的监测阈值，单初始监测成功一次视为健康
    http_helth_check: {} 	 # HTTP类型的监测，包括此种类型在内的一下四种检测类型必须设置一种。
    tcp_health_check: {}     # TCP类型的监测。
    grpc_health_check: {}	# GRPC专用的监测
    custom_health_check: {} # 自定义监测
    reuse_connection: ...   # 布尔型值，是否在多次监测之间重用连接，默认值true
    no_traffic_interval: ... # 定义未曾调度任何流量值集群时，其端点健康监测时间间隔，一旦其接受流量即转为正常的时间间隔
    unhealthy_interval: ...  # 标记为 unhealthy 状态的端点的健康监测时间间隔，一但重新标记为 “health”，即转为正常时间间隔。
    unhealthy_edge_interval: ... # 端点刚被标记为 unhealthy 状态时的健康监测时间间隔，随后转为unhealthy_interval的定义。
    healthy_edge_interval: ... #  端点刚被标记为 healthy 状态时的健康监测时间间隔，随后即转为interval的定义。
```

##### 主动健康监测:TCP

tcp类型监测

```yaml
clusters:
- name: local_service
  connect_timeout: 0.25s
  lb_policy: ROUND_ROBIN
  type: EDS
  eds_cluster_config:
    eds_config:
      api_config_source:
        api_type: GRPC
        grpc_services:
        - envoy_grpc:
            cluster_name: xds_cluster
  health_checks:
  - timeout: 5s
    interval: 10s
    unhealthy_threshold: 2
    healthy_threshod: 2
    tcp_health_check: {}
```

空负载的tcp检测意味着仅通过连接状态判定其检测结果;

非空负载的cp检测可以使用send和receive来分别指定请求负荷及于响应报文中期望模糊匹配的结果;

##### 主动健康监测:HTTP

http类型的检测可以自定义使用的 `path`、`host`和期望的响应码等，并能够在必要时修改（添加/删除）请求报文的标头

具体配置语法如下

```yaml
healthy_checks: []
- ...
  http_health_check:
    &#34;host&#34;: .. # 检测时使用的主机头，默认为空，此时使用集群名称。
    &#34;path&#34;: .. # 检测时使用的路径，例如 /healthz。必选参数
    &#34;server_name&#34;: # 用于验证监测目标集群的服务名称参数，可选；
    &#34;request_headers_to_add&#34;: [] # 想监测报文添加自定义标头列表；
    &#34;request_headers_to_add&#34;: [] # 从监测报文中移除标头列表。
    &#34;use_http2&#34;: # 是否使用http2协议
    &#34;expected_statuses&#34;: [] # 期望的响应码列表
```

配置示例

```yaml
clusters:
- name: local_service
  connect_timeout: 0.25s
  lb_policy: ROUND_ROBIN
  type: EDS
  eds_cluster_config:
    eds_config：
      api_config_source:
        api_type: GRPC
        grpc_services:
        - envoy_grpc:
          cluster_name: xds_cluster
  healthy_checks:
  - timeout: 5s
    interval: 10s
    unhealthy_threshold: 2
    healthy_threshold: 2
    http_health_check:
      host: .. # 默认为空值，并且自动使用集群为其值；
      path: .. # 监测针对的路径，例如/healthz;
      expected_statuses: .. # 期望的响应码，默认200
```



```yaml
version: &#39;3&#39;
services:
  envoy:
    image: envoyproxy/envoy-alpine:v1.15-latest
    environment: 
    - ENVOY_UID=0
    - HEALTHY=ok
    ports:
    - 80:80
    - 443:443
    - 82:9901
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    - ./certs:/etc/envoy/certs
    networks:
      envoymesh:
        aliases:
        - envoy
    depends_on:
    - webserver1
    - webserver2
  
  webserver1:
    image: sealloong/envoy-end:latest
    environment: 
    - COLORFUL=blue
    - HEALTHY=ok
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90
   webserver2:
    image: sealloong/envoy-end:latest
    environment: 
    - COLORFUL=blue
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90 
networks:
  envoymesh: {}
```



```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy_http_connection_manager
        typed_config:
          &#34;@type&#34;: type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: [ &#34;*&#34; ]
              routes:
              - match: { prefix: &#34;/&#34; }
                route: { cluster: local_service }
          http_filters:
          - name: envoy.filters.http.router

  clusters:
  - name: local_service
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: local_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: webservice, port_value: 90 }
    health_checks:
      timeout: 1s
      interval: 2s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses: 200
```

### 异常主机驱逐机制

确定主机异常一&gt;若尚未驱逐主机，且已驱逐的数量低于允许的阀值，则已经驱逐主机一&gt;主机处于驱逐状态一定时长一&gt;超出时长后自动恢复服务。

异常探测通过outlier_dection字段定义在集群上下文中

[](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/cluster/v3/outlier_detection.proto)

```yaml
clusters:
- name: ...
  ...
  outlier_detection:
    consecutive_5xx: .. # 因连续5xx错误而弹出主机之前，允许出现的连续5xx相应或本地原始错误数量，默认5
    interval: # 弹射分析臊面之间的时间间隔，默认为 10000ms或10s
    base_ejection_time: # 主机被弹出的基准时长，实际时长=基准时长*主机已弹出的次数；默认为30s
    max_ejection_percent: # 因一场探测而允许弹出的上游集群中的主机数量百分比，默认为10%，无论如何至少弹出一个主机。
    enforcing_consecutive_5xx: # 基于连续的5xx检查到主机异常时主机将被弹出的几率，可用于禁止弹出或缓慢弹出。默认100
    success_rate_minimum_hosts: # 对集群启动成功率异常监测的最少主机数。默认5
    success_rate_stdev_volume: # 在监测的一次时间间各种，必须收集的请求总数的最小值，默认100
    success_rate_stdev_factor: # 用确定成功率异常值弹出的弹射阈值的因子： 弹射阈值=均值-(因子*平均成功率标准差)；不过，此处设置的值需要除以1000以得到因子，例如使用1.3 时需要将该参数值设置为1300。
    consecutive_gateway_failure: # 因连续网关故障而天厨主机的最少连续故障数，默认5
    enforcing_consecutive_gateway_failure: # 基于连续网关故障检测时而弹出主机的几率的百分比，默认0
    split_external_local_origin_error: # 是否区分本地原因而导致的故障和外部故障，默认为false，设置为true时，一下三项生效：
    consective_local_origin_failure: # 本地原因故障而弹出主机的最少故障次数，默认5
    enforcing_consecutive_local_ogrgin_failure: # 基于连续的本地故障检测到异常状态而弹出主机的几率百分比。默认100
#    enforcing_local_origin_success_rate: # 基于本地故障检测地成功率统计检测到异常状态而弹出主机的几率，默认100
```

同主动健康检查一样，异常检测也要配置在集群级别；下面的示例用于配置在返回3个连续5xx错误时将主机弹出30秒;

```yaml
consecutive_5xx: &#34;3&#34;
base_ejection_time: &#34;30s&#34;
```

在新服务上启用异常检测时应该从不太严格的规则集开始，以便仅弹出具有网关连接错误的主机（HTTP503），并且仅在10%的时间内弹出它们;

```yaml
consecutive_gateway_failure: 3
base_ejection_time: &#34;30s&#34;
enforcing_consecutive_gateway_failure: &#34;10&#34;
```

同时，高流量、稳定的服务可以使用统计信息来弹出频繁异常容的主机；下面的配置示例将弹出错误率低于群集平均值1个标准差的任何端点，统计信息每10秒进行一次评估，并且算法不会针对任何在10秒内少于500个请求的主机运行

```yaml
interval: &#34;10s&#34;
base_ejection_time: &#34;30s&#34;
success_rate_minimum_hosts: &#34;10&#34;
success_rate_request_volume: &#34;500&#34;
success_rate_stdev_factor: &#34;1000&#34;
```



### 离群值检测:HTTP

docker-compose

```yaml
version: &#39;3&#39;
services:
  envoy:
    image: envoyproxy/envoy-alpine:v1.15-latest
    environment: 
    - ENVOY_UID=0
    ports:
    - 80:80
    - 443:443
    - 82:9901
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    networks:
      envoymesh:
        aliases:
        - envoy
    depends_on:
    - webserver1
    - webserver2
  
  webserver1:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90
  webserver2:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90
  webserver3:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90
  webserver4:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90
  webserver5:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - myservice
        - webservice
    expose:
    - 90
networks:
  envoymesh: {}
```


