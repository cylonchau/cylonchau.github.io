# envoy流量管理

[toc]

## 灰度发布

### 概述

新版本上线时，无论是出于产品稳定性还是用户接受程度等方面因素的考虑，直接以新代旧都充满风险；于是，通行做法是新老版本同时在线，且一开始仅分出较小比例的流量至新版本，待确认新版本没问题后再逐级加大流量切换。

灰度发布是迭代的软件产品在生产环境安全上线的一种重要手段，对于Envoy来说，灰度发布仅是流量治理的一种典型应用；以下是几种常见的场景口金丝雀发布

	- 蓝绿发布
	- A/B测试
	- 流量镜像

### 灰度策略
需要在生产环境发布一个新的待上线版本时，需要事先添加一个灰度版本，而后将原有的生产环境的默认版本的流量引流一部分至此灰度版本，配置的引流机制即为灰度策略，经过评估稳定后，即可配置此灰度版本接管所有流量，并下线老版本。

常用的策略类型大体可分为<font color="#0215cd" size=3> **“基于请求内容发布”**</font>和<font color="#0215cd" size=3>**“基于流量比例发布”**</font>两种类型

<font color="#f8070d" size=2>**基于请求内容发布：**</font>配置相应的请求内容规则，满足相应规则服务流量会路由到灰度版本；例如对于http请求，通过匹配特定的Cookie标头值完成引流

Cookie内容：

- 完全匹配：当且仅当表达式完全符合此情况时，流量才会走到这个版本；

- 正则匹配：此处需要您使用正则表达式来匹配相应的规则；

自定义Header：

- 完全匹配：当且仅当表达式完全符合此情况时，流量才会走到这个版本；

- 正则匹配：此处需要您使用正则表达式来匹配相应的规则；可以自定义请求头的key和value，value支持完全匹配和正则匹配；

<font color="#f8070d" size=2>**基于流量比例发布：**</font>对灰度版本配置期望的流量权重，将服务流量以指定权重比例引流到灰度版本；例如10%的流量分配为新版本，90%的流量保持在老版本。这种灰度策略也可以称为AB测试；

<font color="#0215cd" size=2>**注：所有版本的权重之和为100；**</font>

### 灰度发布的实现方式

<font color="#f8070d" size=3>**基于负载均衡器进行灰度发布**</font>

在服务入口的支持流量策略的负载均衡器上配置流量分布机制

仅支持对入口服务进行灰度，无法支撑后端服务需求

<font color="#f8070d" size=3>**基于Kubernetes进行灰度发布**</font>

根据新旧版本应用所在的Pod数量比例进行流量分配，不断滚动更新旧版本Pod到新版本（先增后减、先减后增、又增又减）；服务入口通常是Service或Ingress。

<font color="#f8070d" size=3>**基于服务网格进行灰度发布**</font>

对于Envoy或lstio来说，灰度发布仅是流量治理机制的一种典型应用。通过控制平面，将流量配置策略分发至对目标服务的请求发起方的envoy sidecar上即可。

支持基于请求内容的流量分配机制，例如浏览器类型、cookie等。

服务访问入口通常是一个单独部署的Envoy Gateway。

### Envoy中流量转移

新版本上线时，为兼顾到产品的稳定性及用户的接受程度，让新老版本同时在线，将流量按需要分派至不同的版本；

- 蓝绿发布
- A/B测试
- 金丝雀发布
- 流量镜像
- ....

HTTP路由器能够将流量按比例分成两个或多个上游集群中虚拟主机中的路由，从而产生两种常见用例：

- 版本升级：路由时将流量逐渐从一个集群迁移至另一个集群，实现灰度发布；

- 通过在路由中定义路由相关流量的百分比进行；

A/B测试或多变量测试：同时测试多个相同的服务，路由的流量必须在相同服务的不同版本的集群之间分配；通过在路由中使用基于权重的集群路由完成；另外，匹配条件中，结合指定标头或也能够完成基于内容的流量管理；

流量迁移是指，通过在路由中配置运行时对象选择特定路由以及相应集群的概率的变动，从而实现将虚拟主机中特定路由的流量逐渐从一个集群迁移到另一个集群。

[**runtime_fraction**](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-routematch)

```yaml
route:
- match:
  prefix|path|regex:
  runtime_faction:	    # 额外匹配指定的运行时键值，每次评估匹配路径时，必须低于此字段指示的百分比，支持渐进式修改。
    default_value:      # 运行时键值不可用时，则使用此默认值。 
      numerator:        # 指定分子，默认0
      denominator:      # 指定分母，小于分母时，最终百分比为1， 分母可使用 HUNDRED（默认） TEN_THOUSAND和MILLION，
    runtime_key: routing.traffic_shift.KEY # 要使用运行时的建，值需要用户自行定义。
  route:
    cluster: app_v1
- match:
    prefix|path|regex:
  route:
    cluster: app_v2
```

Envoy基于首次匹配以<font color="#f8070d" size=2>短路机制 </font>工作，若相应的路由存在runtime_fraction对象，则根据其值（或默认值）另外匹配百分比之外的其它请求；这意味着上述示例中，如果两个路由条目match条件一样，则 `runtime_fraction` 对象定义的百分比之外的流量将由第二个路由条目精确捕获。

用户可以通过不断地修改 `runtime_fraction` 对象的值完成流量迁移；<font color="#f8070d" size=3>`curl envoy_administration_api:9901/runtime_modify?k1=v1&k2=v2`</font>

```yaml
admin:
  access_log_path: /dev/null
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy_http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: local_route
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: base_domain
              domains: [ "*" ]
              routes:
              - match:
                  prefix: "/"
                  runtime_fraction:
                    default_value:
                      numerator: 0
                      denominator: HUNDRED
                    runtime_key: routing.traffic_shift.sv
                route:
                  cluster: app_v2
              - match: { prefix: / }
                route: { cluster: app_v1 }
          http_filters:
          - name: envoy.filters.http.router

  clusters:
  - name: app_v1
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: app_v1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: version1, port_value: 90 }
    health_checks:
      timeout: 3s
      interval: 30s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses:
          start: 200
          end: 201
  - name: app_v2
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: app_v2
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: version2, port_value: 90 }
    health_checks:
      timeout: 3s
      interval: 30s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses:
          start: 200
          end: 201
```

docker-compose

```yaml
version: '3'
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
    - webserver3
    - webserver4
  
  webserver1:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - version1
    environment: 
    - VERSION=v1
    - COLORFUL=blue
    expose:
    - 90
  webserver2:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - version1
    environment: 
    - VERSION=v1
    - COLORFUL=blue
    expose:
    - 90
  webserver3:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - version2
    environment: 
    - VERSION=v2
    - COLORFUL=red
    expose:
    - 90
  webserver4:
    image: sealloong/envoy-end:latest
    environment: 
    - VERSION=v2
    - COLORFUL=red
    networks:
      envoymesh:
        aliases:
        - version2
    expose:
    - 90
networks:
  envoymesh: {}
```

测试使用的脚本

```sh
#!/bin/bash

declare -i v1=0
declare -i v2=0
declare -r inverval="0"
declare -i count=$2

declare -i i=0

while [ ${i} -lt ${count} ]
do
	if curl -s http://$1/version | grep "version£ºv2" &> /dev/null; then
		let v2++
	else
		let v1++
	fi
	echo -e "\033[31m[Version1:v2]\033[0m==${v1}:${v2}"
	sleep $inverval
	let i++
done
```

### Envoy 流量分割 traffic split

HTTP router过滤器支持在一个路由中指定多个上游具有权重属性的集群，而后将流量基于权重调度至此些集群其中之一。

分配给每个集群的权重也可以使用运行时参数进行调整；

``` yaml
routes
- match:
  route:
    weight_clusters: 
      cluster: [] # 与当前路由关联的一个或多个集群，required
      - name: ..  # 集群的名称，是指定已存在cluster的名称，非自定义标志。
        weight: .. # 集群权重，取值范围为0~total_weight 
        metadata_match: .. # 子集负载均衡器使用的端点元数据匹配条件，可选参数，仅用于上游及群众具有此字段中设置的元数据匹配的元数据端点以进行流量分配。
      total_weight: .. # 总权重值，默认100
      runtime_key_prefix: ... # 可选参数，用于设定键前缀，从而每个集群以 runtime_key_prefix+.cluster[i].name 为其键名，名能够运行时键值的方式为每个集群提供权重，其中，cluster[i].name表示列表中第i个集群
```

#### 流量分流配置示例

假设存在某微服务应用，其v1和v2两个版本分别对应于appv1和appv2两个集群；

初始权重比例为appv1承载100%，而appv2不承载任何请求；随后可通过运行时参数，将所有流量切往appv2；

各自的权重比例可通过运行时参数动态调整：<font color="#f8070d" size=2>`curl -XPOST http://host:admin_port/runtime_modify?routing.traffic_split.prefix.k1=0&routing.traffic_splic.prefix.k2=100`</font> 

与 `taffic_shift` 不同，`Traffic splitting` 只需要单个路由条目。 路由中的 `weighted_clusters` 配置块可用于指定多个上游群集以及指示将发送到每个上游群集的流量**百分比**的权重。

[配置示例](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/traffic_splitting.html?highlight=traffic_shift#)

```yaml
admin:
  access_log_path: /dev/null
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_80
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy_http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          http_filters:
          - name: envoy.filters.http.router
          stat_prefix: local_route
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: split_traffic
              domains: [ "*" ]
              routes:
              - match: 
                  prefix: "/"
                route:
                  weighted_clusters:
                    total_weight: 100
                    runtime_key_prefix: routing.traffic_split.version
                    clusters:
                    - name: version_v1
                      weight: 100
                    - name: version_v2
                      weight: 0     
  clusters:
  - name: version_v1
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: version_v1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: version1, port_value: 90 }
    health_checks:
      timeout: 3s
      interval: 30s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses:
          start: 200
          end: 201
  - name: version_v2
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: version_v2
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: version2, port_value: 90 }
    health_checks:
      timeout: 3s
      interval: 30s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses:
          start: 200
          end: 201
```

docker-compose

```yaml
version: '3'
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
    - webserver3
    - webserver4
  
  webserver1:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - version1
    environment: 
    - VERSION=v1
    - COLORFUL=blue
    expose:
    - 90
  webserver2:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - version1
    environment: 
    - VERSION=v1
    - COLORFUL=blue
    expose:
    - 90
  webserver3:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - version2
    environment: 
    - VERSION=v2
    - COLORFUL=red
    expose:
    - 90
  webserver4:
    image: sealloong/envoy-end:latest
    environment: 
    - VERSION=v2
    - COLORFUL=red
    networks:
      envoymesh:
        aliases:
        - version2
    expose:
    - 90
networks:
  envoymesh: {}
```

### Envoy 影子镜像 Shadow mirroring



**关于影子镜像**

流量镜像，也称为流量复制或影子镜像

流量镜像功能通常用于在生产环境进行测试，通过将生产流量镜像拷贝到测试集群或者新版本集群，实现新版本接近真实环境的测试，旨在有效地降低新版本上线的风险；

流量镜像可用于以下场景

- 验证新版本：实时对比镜像流量与生产流量的输出结果，完成新版本目标验证
- 测试：用生产实例的真实流量进行模拟测试
- 隔离测试数据库：与数据处理相关的业务，可使用空的数据存储并加载测试数据，针对该数据进行镜像流量操作，实现测试数据的隔离

假设我们要配置以下设置，其中25％的流量被影子镜像到另一个上游，可通过访问 `myservice-test.mycompany.com`。

![å¸¦æ25ï¼æµééåçEnvoyè®¾ç½®ã](../../../images/Envoy%E6%B5%81%E9%87%8F%E7%AE%A1%E7%90%86/envoy-mirror-setup.png)

将流量转发至一个集群（主集群）的同时再转发到另一个集群（影子集群）

无须等待影子集群返回响应。

支持收集影子集群的常规统计信息，常用于测试具有实际生产流量的服务，而不会以任何方式影响最终客户。

```yaml
route:
 cluster|weighted_cluster:
 ..
 request_mirror_policies:
   cluster: ..
   trace_sampled: # 确定是否应采样跟踪范围。默认为true。
   runtime_faction: {}
     default_value:  # 运行时key不可用时，则使用此默认值，。
       numerator:    # 指定分子，默认0
       denominator:  # 指定坟墓，小鱼分子时，最终百分比为1，分母可固定使用HUNDRED TEN_RHOUSAND MILLION
     runtime_key: routing.request_mirror.Key # 运行时键，值需自行定义
```

默认情况下，路由器会镜像所有请求；也可使用如下两个参数配置转发的流量比例

	- `runtime_key` ：运行时键，用于明确定义向影子集群转发的流量的百分比，取值范围为0-10000，每个数字表示0.01%的请求比例；定义了此键却未指定其值时，默认为0；此参数即将废弃，并下面的参数所取代；
	- `runtime_fraction`：转发的流量比例小于N/D，优先级高于单独指定的前一个参数runtime_key;
	- `runtime_key` 定义运行时进行动态调整：<font color="#f8070d" size=2>`curl -XPOST http://host:admin_port/runtime_modify?routing.request_mirror.version=100` </font>

envoy.yaml

```yaml
admin:
  access_log_path: /dev/null
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_80
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy_http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          http_filters:
          - name: envoy.filters.http.router
          stat_prefix: local_route
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: split_traffic
              domains: [ "*" ]
              routes:
              - match: 
                  prefix: "/"
                route:
                  cluster: version_v1
                  request_mirror_policies:
                    cluster: version_v2
                    runtime_fraction:
                      default_value:
                        numerator: 0
                        denominator: HUNDRED
                      runtime_key: routing.request_mirror.version
  clusters:
  - name: version_v1
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: version_v1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: version1, port_value: 90 }
    health_checks:
      timeout: 3s
      interval: 30s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses:
          start: 200
          end: 201
  - name: version_v2
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: version_v2
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: version2, port_value: 90 }
    health_checks:
      timeout: 3s
      interval: 30s
      unhealthy_threshold: 2
      healthy_threshold: 2
      http_health_check:
        path: /ping
        expected_statuses:
          start: 200
          end: 201
```

[参考](https://blog.markvincze.com/shadow-mirroring-with-envoy/)

## 金丝雀发布 Canary Deployment

金丝雀部署通过在生产环境运行的服务中引一部分实际流量对一个新版本进行测试，测试新版本的性能和表现，然后从这部分的新版本中快速获取用户反馈

特点：通过在线上运行的服务中，新加入少量的新版本的服务，然后从这少量的新版本中快速获得反馈，根据反馈决定最后的交付形态

## 蓝绿部署 Blue-Green Deployment

蓝绿发布提供了一种零宕机的部署方式，不停用老版本的同时部署新版本进行测试，确认没问题后将流量切到新版本。特点：

	- 在保留旧版本的同时部署新版本，将两个版本同时在线，新版本和旧版本互相热备。
	- 通过切换路由权重（weight）的方式（非0即100）实现应用的不同版本上线或者下线，如果有问题可以快速地回滚到老版本。

## AB测试 A/B Testing

从本质上讲，AB测试是一种实验，通过向用户随机显示页面的两个或多个变体，并使用统计分析来确定哪种变体对于给定的转化目标效果更好；

- A/B测试可用于测试各种营销元素，例如设计/视觉元素、导航、表单和宣传用语等；

- A/B测试可以用于测试、比较和分析几乎所有内容

最常用于网站以及移动应用程序；将Web或App界面或流程的两个或多个版本，在同一时间维度，分别让两个或多个属性或组成成分相同（相似）的访客群组访问，收集各群组的用户体验数据和业务数据，最后分析评估出最好版本以正式采用

主要用于转换率优化，一般在线业务会定期通过A/B测试来优化其目标网页并提高ROI

A/B测试需要同时在线上部署A和B两个对等版本同时接收用户流量，按一定的目标选择策略将一部分用户导向A版本，让另一部分用户使用B版本；分别收集两部分用户的反馈，并根据分析结果确定最终使用的版本；

A/B测试中分流的设计直接决定了测试结果是否有效

AB测试是对线上生产环境的测试，在对改进版本所产生效果的好坏不能十分确定时对测试版本的导入流量通常不宜过大，尤其对于那些影响范围较大的改版（如主流程页面的重大调整），影响用户决策的新产品上线和其他具有风险性的功能上线通常采用先从小流量测试开始，然后逐步放大测试流量的方法。但是，测试版本的流量如果太小又可能造成随机结果的引入，试验结果失去统计意义
