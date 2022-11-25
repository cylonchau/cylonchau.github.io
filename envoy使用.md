# Envoy部署

[toc]

### 常用的构建方法

https://skyao.io/learning-envoy/

- 手动配置构建环境，手动构建Envoy。

- 使用Envoy提供的预制于Docker中的构建环境进行手动构建二进制Envoy

- 使用Envoy提供的预制于Docker中的构建环境进行手动构建二进制Envoy，并直接将Envoy程序打包成Docker镜像

- 提供Bootstrap配置文件，存在使用xDS的动态资源时，还需要基于文件或管理服务器向其提供必须的配置信息
  
  - 也可使用Envoy的配置生成器生成示例性配置
  
- 基于Bootstrap配置文件启动Envoy实例

  - 直接启动二进制Envoy
  - 于Kubernetes平台基于Sidecar的形式运行Envoy，或运行单独的Envoy Pod（Edge Proxy）

  ### 启动Envoy

启动Envoy时，需要通过（`-c` 选项为其指定初始配置文件以提供引导配置（Bootstrap configuration），这也是使用v2APl的必然要求；

```
envoy -c  <path to config>.{json,yaml,pb,pb_text}
```

扩展名代表了配置信息的组织格式；



引导配置是Envoy配置信息的基点，用于承载Envoy的初始配置，它可能包括静态资源和动态资源的定义

- 静态资源（static_resources）于启动直接加载

- 动态资源（dynamic_resources）则需要通过配置的xDS服务获取并生成

通常，`Listener` 和 `Cluster` 是Envoy得以运行的基础，而二者的配置可以全部为静态格式，也可以混合使用动态及静态方式提供，或者全部配置为动态；

一个yaml格式纯静态的基础配置框架类似如下所示：

```yaml
static_resources:
  linteners:
  - name: ...
    address: {}
    filter_chains: []
  clusters:
  - name: ...
    type: ...
    connect_timeout: {}
    lb_policy: ...
    load_assignment: {}
```

### Listener 简易静态配置

侦听器主要用于定义Envoy监听的用于接收Down streams请求的套接字、用于处理请求时调用的过滤器链及相关的其它配置属性;目前envoy仅支持tcp的协议

```yaml
listeners:
- name:
  address:
    socket_address: { address: ..., port_value: ..., protocol: ...}
```

L4过滤器echo主要用于演示网络过滤器APl的功能，它会回显接收到的所有数据至下游的请求者；在配置文件中调用时其名称为 `envoy.echo` ；

下面是一个最简单的静态侦听器配置示例：

[echo文档](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/echo_filter)

```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 15001
    filter_chains:
    - filters:
      - name: envoy.echo
```

![image-20220506225303515](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220506225303515.png)

### Cluster 静态配置

通常，集群代表了一组提供相同服务的上游服务器（端点）的组合，它可由用户静态配置，也能够通过CDS动态获取；

集群需要在“预热”环节完成之后方能转为可用状态，这意味着集群管理器通过DNS解析或EDS服务完成端点初始化，以及健康状态检测成功之后才可用；

[cluster 官方参数](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/cluster/v3/cluster.proto)

```yaml
clusters:
- name: ... # 集群的唯一名称，且为提供alt_stat_name时会被用于统计信息中
  alt_state_name: ... # 统计信息中使用集群带名称。
# 用于解析集群（生成集群端点）时使用的服务发现类型，可用值有 STATIC STRJCT_DNS LOGICAL_DNS GRIGINAL_DST和EDS等
  type: .. 
# 负载均衡算法，支持 ROUND_ROBIN LEAST_REQUEST RING_HASH RANDOM MAGLEV CLUSTER_PROVIDED
  lb_policy: 
# 为 STATIC STRJCT_DNS LOGICAL_DNS 类型的集群指定成员获取方式，EDS类型继承要使用eds_cluster_config字段配置。
  loda_assignment: 
    cluser_name: .. # 集群名称
    endpoints: #端点列表
    - locality: {} #表示上游主机所处的位置，通常以 region、none等进行标识。
      lb_endpoints: # 属于指定位置的端点列表
      - endpoint_name: #端点的名称
        endpoint: # 端点的定义
          socket_address: #端点地址标识
            address:
            port_value:
            portocol:
```

静态Cluster的各端点可以在配置中直接给出，也可借助DNS服务进行动态发现；

下面的示例直接给出了两个端点地址

```yaml
clusters:
- name: test_cluster
  connect_timeout: 0.25
  type: STATIC
  lb_policy: ROUND_ROBIN
  load_assignment:
    cluster_name: test_cluster
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address: { address: 10.0.0.1, port_value: 80 }
      - endpoint:
          address:
            socket_address: { address: 10.0.0.2, port_value: 80 }
```

### filter

#### tcp_proxy

[官方文档](https://www.envoyproxy.io/docs/envoy/v1.15.0/api-v2/config/filter/network/tcp_proxy/v2/tcp_proxy.proto)

TCP代理过滤器在下游客户端及上游集群之间执行1:1网络连接代理

- 它可以单独用作隧道替换，也可以同其他过滤器（如MongoDB过滤器或速率限制过滤器）结合使用；
- TCP代理过滤器严格执行由全局资源管理于为每个上游集群的全局资源管理器设定的连接限制
  - TCP代理过滤器检查上游集群的资源管理器是否可以在不超过该集群的最大连接数的情况下创建连接；
- TCP代理过滤器可直接将请求路由至指定的集群，也能够在多个目标集群间基于权重进行调度转发；

配置示例：

```yaml
listeners:
  name: listener_0
  address:
    socket_address: { address:0.0.0.0, port_value: 80 }
  filter_chains:
  - filters:
    - name: envoy.tcp_proxy
      typed_config:
        "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
        stat_prefix: tcp_01 # 统计数据的前缀
        cluster: test_cluster
```

下面的示例基于TCP代理将下游用户（本机）请求代理至外部的（egress）两个web服务器

```yaml  
static_resources:
  listeners:
    name: listener_0
    address: 
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.tcp_proxy.v3.TcpProxy
          stat_prefix: tcp_01
          cluster: test_tcp
  clusters:
  - name: test_tcp
    connect_timeout: 0.5s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: test_tcp
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: 127.0.0.1, port_value: 90 }
```

```dockerfile
version: '3'
services:
  envoy:
    image: envoyproxy/envoy-alpine:v1.15-latest
    ports: 
    - 81:80
    environment: 
    - ENVOY_UID=0
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    network_mode: "service:mainserver" 
    depends_on:
    - mainserver
  mainserver:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - webserver
        - httpserver
        - envoy_end
networks:
  envoymesh: {}
```

#### http_connection_manager

`http_connection_manager` 通过引入L7过滤器链实现了对http协议的操纵，其中router过滤器用于配置路由转发；

[官方文档](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto)

```yaml
listeners:
- name:
  address: 
    socket_address: { address: ..., port_value: ...., protocol:... }
  filter_chains:
  - filters:
    - name: envoy.http_connection_manager
      config:
        condec_type: .. # 链接管理器使用的编解码器类型，可用值有 AUTO HTTP1 HTTP2
        stat_prefix: ... # 统计信息中使用的易读性的信息前缀。
        route_config: ... # 静态路由配置，动态配置应该使用rds字段进行指定。
          name: # 路由配置的名称
          virtual_hosts: # 虚拟主机的路基名称，用于构成路由表
          - name: ... # 虚拟主机的逻辑名称，用于统计信息，与路由无关
            domains: [] # 虚拟主机匹配的域名列表，支持 * 通配符，匹配搜索次序为精确匹配、前缀匹配、后缀匹配及完全匹配。
            routes: [] # 路由列表，按照顺序搜索，第一个匹配到的路由信息。
        http_filters: # 定义http过滤器链
        - name: envoy.router # 调用过滤器为envoy.router
```

处理请求时，Envoy首先根据下游客户端请求的 `host` 来搜索虚拟主机列表中各 `virtual_host` 中的`domains` 列表中的定义，第一个匹配到的 `Domain` 的定义所属的 `virtual_host` 即可处理请求的虚拟主机;

而后搜索当前虚拟生机中的 `routes` 列表中的路由列表中各路由条目的 `match` 的定义，第一个匹配到的 `match` 后的路由机制（route.redirect或direct_response）即生效；

`route_config.virtual_hosts.routes` 配置的路由信息用于将下游的客户端请求路由至合适的上游集群中某Server上；

- 其路由方式是将url匹配match字段的定义
  - `match` 字段可通过  `prefix`（前级）、`path`（路径）或 `regex`（正则表达式）三者之一来表示匹配模式；
- 与match相关的请求将由 `route` 、`redirect` 或 `direct_response` 三个字段其中之一完成路由；
- 由route定义的路由目标必须是 `cluster`（上游集群名称）、`cluster_header`（根据请求标头中的 `cluster_header` 的值确定目标集群）或 `weighted_clusters`（路由目标有多个集群，每个集群拥有一定的权重）其中之一；

```yaml
routes: []
- name: ...     # 路由条目的名称
  match:
    prefix: ... # 请求的URL的前缀
  route:        # 路由条目
    cluster:    # 目标下游集群。

```

##### Egress代理配置示例

下面是一个egree类型的Envoy配置示例，定义了两个virtual_host，不过，发往第二个virtual_host的请求将被重定向至第一个。

```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: egress_http
          codec_type: AUTO
          route_config:
            name: study_route
            virtual_hosts:
            - name: web_service_1
              domains: [ "*.ik8s.io", "ik8s.io" ]
              routes:
              - match: { prefix: "/" }
                route: { cluster: web_cluster_1 }
            - name: web_service_2
              domains: [ "*.k8scast.cn", "k8scast.cn" ]
              routes:
              - match: { prefix: "/" }
                redirect:
                  host_redirect: "www.ik8s.io"
          http_filters:
          - name: envoy.filters.http.router

  clusters:
  - name: web_cluster_1
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: web_cluster_1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: myservice, port_value: 90 }
  - name: web_cluster_2
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: web_cluster_2
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: webserver2, port_value: 90 }
```

```yaml
version: '3'
services:
  envoy:
    image: envoyproxy/envoy-alpine:v1.15-latest
    environment: 
    - ENVOY_UID=0
    ports:
    - 81:80
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
    environment: 
    - COLORFUL=red
    networks:
      envoymesh:
        aliases:
        - webservice
        - myservice
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
    expose:
    - 90
     
networks:
  envoymesh: {}
```

**测试结果**

```sh
$ curl -H 'Host: www.ik8s.io' 127.0.0.1:81/ping
{"message":"pong"}

$ curl -H 'Host: www.ik8s.io' 127.0.0.1:81/colorful
{"message":"hello with red."}$ 
```

##### ingress 配置示例

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
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: [ "*" ]
              routes:
              - match: { prefix: "/" }
                route: { cluster: local_service }
          http_filters:
          - name: envoy.filters.http.router

  clusters:
  - name: local_service
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: local_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: 127.0.0.1, port_value: 90 }
```

```yaml
version: "3"
services:
  envoy:
    image: envoyproxy/envoy-alpine:v1.15-latest
    environment: 
    - ENVOY_UID=0
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    network_mode: "service:webserver"
    depends_on:
    - webserver
    
  webserver:
    image: sealloong/envoy-end:latest
    environment: 
    - COLORFUL=red
    networks:
      envoymesh:
        aliases:
        - webservice
        - myservice
    expose:
    - 90
 
networks:
  envoymesh: {}
```


