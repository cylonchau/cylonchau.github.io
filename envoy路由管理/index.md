# Envoy路由管理

[toc]

## HTTP 链接管理

envoy处理用户请求逻辑

![image-20200919165020447](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20200919165020447.png)

① 用户请求报文到达七层过滤器链处理机制后，首先根据请求HTTP报文中的 `Host` 首部来完成虚拟主机的选择；② 由此虚拟机主机内部的 **`Route`** 表项进行处理。③ 后由 `match` 进行匹配; `Match` 是与请求URL的 `PATH` 组成部分或请求报文的标头部分 `header` 。④ 最后才可到达 `Route` 进行 `direct_response` 、`redirect` 等。

[官方手册](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/http/http_routing)

Envoy通过内置的L4过滤器HTTP连接管理器将原始字节转换为HTTP应用层协议级别的消息和事件，例如接收到的标头和主体等，以及处理所有HTTP连接和请求共有的功能，包括访问日志、生成和跟踪请求ID，请求/响应头处理、路由表管理和统计信息等；

- 支持HTTP/1.1、WebSockets和HTTP/2，但不支持SPDY;
- 关联的路由表可静态配置，亦可经由 `xDS API` 中的 `RDS` 动态生成；
- 内建重试插件，可用于配置重试行为;
  - Host Predicates
  - Priority Predicates

内建支持302重定向，它可以捕获302重定向响应，合成新请求后将其发送到新的路由匹配（match）所指定的上游端点，并将收到的响应作为对原始请求的响应返回客户端

支持适用于HTTP连接及其组成流（constituent streams）的多种可配置超时机制

- 连接级别：空闲超时和排空超时（GOAWAY）；
- 流级别：空闲超时、每路由相关的上游端点超时和每路由相关的 `gRPC` 最大超时时长；
- 基于自定义集群的动态转发代理；

HTTP协议相关的功能通过各HTTP过滤器实现，这些过滤器大体可分为编码器、解码器和编/解码器三类；

router `envoy.router` 是最常的过滤器之一，它基于路由表完成请求的转发或重定向，以及处理重试操作和生成统计信息等；

### HTTP 路由

Envoy基于HTTP router过滤器基于路由表完成多种高级路由机制，例如:

- 将域名映射到虚拟主机；
- path的前级 `prefix` 匹配、精确匹配或正则表达式匹配；
- 虚拟主机级别的TLS重定向；
- path级别的 `path/host` 重定向；
- **`direct_response`** ，由Envoy直接生成响应报文 ；
- 显式 `host rewrite`；
- `prefix rewrite`；
- 基于HTTP标头或路由配置的**请求重试**与**请求超时**；
- 基于运行时参数的流量迁移；
- 基于权重或百分比的跨集群流量分割；
- 基于任意标头匹配路由规则；
- 基于优先级的路由； 
- 基于hash策略的路由；

#### 路由配置和虚拟主机

路由配置中的顶级元素是虚拟主机

每个虚拟主机都有一个逻辑名称以及一组域名，请求报文中的主机头将根据此处的域名进行路由；单个侦听器可以服务于多个顶级域。

基于域名选择虚拟主机后，将基于配置的路由机制完成请求路由或进行重定向；

```json
{
    &#34;name&#34; : &#34;...&#34;,
    &#34;domains&#34; : [],
    &#34;routes&#34; : [],
    &#34;require_tls&#34; : &#34;&#34;,
    &#34;virtual_clusters&#34;: [],
    &#34;rate_limits&#34;: [],
    &#34;request_headers_to_add&#34;: [],
    &#34;request_headers_to_remote&#34;: [],
    &#34;response_headers_to_add&#34;: [],
    &#34;response_headers_to_remove&#34;: [],
    &#34;cors&#34;: &#34;{...}&#34;,
    &#34;pre_filter_config&#34;: &#34;{}&#34;,
    &#34;typed_per_filter_config&#34;: &#34;{}&#34;,
    &#34;include_request_attempt_count&#34;:  &#34;..&#34;,
    &#34;retry_policy&#34;: &#34;{...}&#34;,
    &#34;hedge_policy&#34;: &#34;{...}&#34;
}
```

虚拟主机级别的路由策略用于为相关的路由属性提供默认配置，用户也可在路由配置上自定义用到的路由属性，例如**限流**、**CORS**和**重试**机制等；

#### Route及配置框架

Envoy匹配路由时，它基于如下工作过程进行：

- **Host**；检测HTTP请求的host标头或；authority，并将其同路由配置中定义的虚拟主机作匹配检查；

  - **将域名映射到虚拟主机**
    - 将请求报文中的host标头值依次与路由表中定义的各Virtualhost的domain属性值进行比较，并与第一次匹配时终止搜索
    - Domain search order
      - **Exact domain names**: `www.ilinux.io`.
      - **Suffix domain wildcards**: `*.ilinux.io` or `*-envoy.ilinux.io`.
      - **Prefix domain wildcards**: `ilinux.*` or  `ilinux-*`.
      - **Special wildcard** `*` matching any domain.

- **Match**；在匹配到的虚拟主机配置中按顺序检查虚拟主机中的每个路由条目中的匹配条件，直到第一个匹配的为止（**短路**）；

  - 基于 `prefix` `path` `regex` 三者其中任何一个进行URL匹配。
  - 可根据 `headers` 和 `query_parameters` 完成报文额外匹配。
  - 匹配得到的报文可有三种路由机制：
    - redirect
    - direct_response
    - route

- **route**；如果定义了虚拟集群，按顺序检查虚拟主机中的每个虚拟集群，直到第一个匹配的为止；

  - 支持 `cluster` 、`weighted_cluster` `cluster_header` 三者之一定义目标路由。
- 转发期间可根据 `prefix_rewrite` 和 `host_rewrite` 完成URL重写。
  - 可额外配置流量管理机制，例如：`timeout` `retry_policy` `cors` `request_mirror_policy` `rate_limits` 等

```
listeners:
- name:
  address: {...}
  filter_chians: []
  - filters:
    - name: envoy.http_connection_manager
      config:
      ...
        route_config:
          name: ...
          virutal_hosts: []
          - name: 
            domains: [] # 虚拟主机的域名，路由匹配时，将请求报文中的host标头值与此处列表项进行匹配检测
            routes: [] # 路由条目，匹配到当前虚拟主机的请求中的path匹配检测将针对各route中有match定义条件进行;
            - name: ...
              match: {...}
                prefix|path|regex: ... # 记录路径前缀、路由或正则表达式三者之一定义匹配条件；
              route: {...}
                cluster|cluster_header|weighted_cluster: .. # 基于集群、请求报文中的集群标头或加权集群（流量分割）定义路由目标；
            virtual_clusters: [] # 为此虚拟主机定义的用于收集统计信息的虚拟集群列表；
```


&gt; **路由配置框架**

符合匹配条件的请求要由如下三种方式之一处理

- **route**：路由到指定位置。
- **redirect**：重定向到指定位置。
- **direct_response**：直接以给定的内容进行响应。

路由中也可按需在请求及响应报文中添加或删除响应标头

[config.route.v3.Route](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-route)

```json
{
  &#34;name&#34;: &#34;...&#34;,
  &#34;match&#34;: &#34;{...}&#34;,
  &#34;route&#34;: &#34;{...}&#34;,
  &#34;redirect&#34;: &#34;{...}&#34;,
  &#34;direct_response&#34;: &#34;{...}&#34;,
  &#34;metadata&#34;: &#34;{...}&#34;,
  &#34;decorator&#34;: &#34;{...}&#34;,
  &#34;typed_per_filter_config&#34;: &#34;{...}&#34;,
  &#34;request_headers_to_add&#34;: [],
  &#34;request_headers_to_remove&#34;: [],
  &#34;response_headers_to_add&#34;: [],
  &#34;response_headers_to_remove&#34;: [],
  &#34;tracing&#34;: &#34;{...}&#34;,
  &#34;per_request_buffer_limit_bytes&#34;: &#34;{...}&#34;
}
```

&gt; **RouteMatch**

匹配条件是定义的检测机制，用于过滤出符合条件的请求并对其作出所需的处理，例如路由、重定向或直接响应等；

必须要定义**`prefix`** 、**`path`** 和 **`regex`** 三种匹配条件中的一种形式

```json
{
  &#34;prefix&#34;: &#34;...&#34;, 			/* path前缀匹配条件 */  
  &#34;path&#34;: &#34;...&#34;, 			/* path精确匹配条件 */
  &#34;safe_regex&#34;: &#34;{...}&#34;, 	/* 整个path（不包含query子串）必须制定的正则表达式匹配 */
  &#34;connect_matcher&#34;: &#34;{...}&#34;,
  &#34;case_sensitive&#34;: &#34;{...}&#34;,
  &#34;runtime_fraction&#34;: &#34;{...}&#34;,
  &#34;headers&#34;: [],
  &#34;query_parameters&#34;: [],
  &#34;grpc&#34;: &#34;{...}&#34;,
  &#34;tls_context&#34;: &#34;{...}&#34;
}
```

除了必须设置上述三者其中之一外，还可额外完成如下限定

- 区分字符大小写（case_sensitive）
- 匹配指定的运行键值表示的比例进行 &lt;font color=&#34;#f8070d&#34; size=2&gt;**流量迁移**&lt;/font&gt; `runtime_fraction`；
  - 不断地修改运行时键值完成流量迁移
- &lt;font color=&#34;#f8070d&#34; size=2&gt;基于标头的路由：&lt;/font&gt;匹配指定的一组标头（headers）；
- &lt;font color=&#34;#f8070d&#34; size=2&gt;基于参数的路由：&lt;/font&gt;匹配指定的一组URL查询参数（query_parameters）；
- 仅匹配grpc流量（grpc）；

&gt; **Route.HeaderMatcher**

[config.route.v3.HeaderMatcher](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-headermatcher)

指定的路由需要额外匹配指定的一组标头

路由器将根据路由配置中的所有指定标头检查请求的标头

若路由中指定的所有标头都存在于请求中且具有相同值，则匹配

若配置中未指定标头值，则基于标头的存在性进行判断

```json
{
  &#34;name&#34;: &#34;...&#34;,
  &#34;exact_match&#34;: &#34;...&#34;,
  &#34;safe_regex_match&#34;: &#34;{...}&#34;,
  &#34;range_match&#34;: &#34;{...}&#34;,
  &#34;present_match&#34;: &#34;...&#34;,
  &#34;prefix_match&#34;: &#34;...&#34;,  /* 默认配置，即根据制定标头的存在性进行判断 */
  &#34;suffix_match&#34;: &#34;...&#34;,  
  &#34;contains_match&#34;: &#34;...&#34;,
  &#34;invert_match&#34;: &#34;...&#34;   /* 布尔型值，用于制定是否为上面的匹配条件取反 */
}
```

标头及其值的上述检查机制仅能定义一个：

- exact_match：精确值匹配
- regex-match：整个值与正则表达式匹配
- range_mtch：值范围匹配
- present_match：标头存在性匹配
- prefix_match：值前缀匹配
- sufix_match：值后缀匹配
- invert_match：将匹配结果取反，默认为==false==

&gt; **route.QueryParameterMatcher**

[route.QueryParameterMatcher](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-queryparametermatcher)

指定的路由需要额外匹配的一组URL查询参数

路由器将根据路由配置中指定的所有查询参数检查路径头中的查询字符串

- 查询参数匹配将请求的URL中查询字符串视为以 &lt;font color=&#34;#f8070d&#34; size=3&gt;**`&amp;`**&lt;/font&gt; 符号分隔的 &lt;font color=&#34;#f8070d&#34; size=3&gt;`key`&lt;/font&gt; 或 &lt;font color=&#34;#f8070d&#34; size=3&gt;`key=value`&lt;/font&gt; 元素列表

- 若存在指定的查询参数，则所有参数都必须与URL中的查询字符串匹配

- 匹配条件指定为 `string_match` 或 `present_match` 其中之一。

```json
{
  &#34;name&#34;: &#34;...&#34;,
  &#34;string_match&#34;: &#34;{...}&#34;,  /* 指定查询参数值是否应与字符串匹配。 */
  &#34;present_match&#34;: &#34;...&#34;    /* 指定是否应存在查询参数。 */
}
```

### 路由目标

#### 路由目标：重定向（redirect）

[config.route.v3.RedirectAction](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-redirectaction)

为请求响应一个301应答，从而将请求从一个URL永久重定向至另一个URL

**Envoy支持如下重定向行为:**

- 协议重定向：https redirect或scheme_redirect二者只能使用其一；

- 主机重定向：host_redirect口端口重定向：port_redirect

- 路径重定向：path_redirect口路径前级重定向：prefix_redirect

- 重设响应码：response_code，默认为301；

- strip_query：是否在重定向期间删除URL中的查询参数，默认为false；

```json
{
  &#34;https_redirect&#34;: &#34;...&#34;,
  &#34;scheme_redirect&#34;: &#34;...&#34;,
  &#34;host_redirect&#34;: &#34;...&#34;,
  &#34;port_redirect&#34;: &#34;...&#34;,
  &#34;path_redirect&#34;: &#34;...&#34;,
  &#34;prefix_rewrite&#34;: &#34;...&#34;,
  &#34;response_code&#34;: &#34;...&#34;,
  &#34;strip_query&#34;: &#34;...&#34;
}
```

#### 路由目标：直接相应请求（direct_response）

[config.route.v3.DirectResponseAction](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-directresponseaction)

Envoy可以直接相应请求，而不将请求代理至上游主机

```json
{
  &#34;status&#34;: &#34;...&#34;,
  &#34;body&#34;: &#34;{...}&#34;
}
```

- 响应码可由status直接给出
- 响应正文可省略，默认为空；需要指定时应该由body通过如下三种方式之一给出数据源:
  - filename: 本地文件数据源
  - inline_bytes: 配置中给出内嵌的byte类型 （unsigned char）
  - inline_string：配置中内嵌的字符串。

#### 路由目标：路由到指定集群（route）

匹配到的流量可路由至如下三种目标之一

- cluster：指定的上游集群；
- cluster_header：在请求标头中设置cluster_header的值指定的上游集群；
- weighted_clusters：基于权重将请求路由至多个上游集群，进行流量分割；

```json
{
  &#34;cluster&#34;: &#34;...&#34;,  /* 路由到的目标集群 */
  &#34;cluster_header&#34;: &#34;...&#34;,
  &#34;weighted_clusters&#34;: &#34;{...}&#34;,   /* 将流量路由并按权重分配到多个上游集群 */
  &#34;cluster_not_found_response_code&#34;: &#34;...&#34;,
  &#34;metadata_match&#34;: &#34;{...}&#34;,
  &#34;prefix_rewrite&#34;: &#34;...&#34;,
  &#34;regex_rewrite&#34;: &#34;{...}&#34;,
  &#34;host_rewrite_literal&#34;: &#34;...&#34;,
  &#34;auto_host_rewrite&#34;: &#34;{...}&#34;,
  &#34;host_rewrite_header&#34;: &#34;...&#34;,
  &#34;host_rewrite_path_regex&#34;: &#34;{...}&#34;,
  &#34;timeout&#34;: &#34;{...}&#34;,
  &#34;idle_timeout&#34;: &#34;{...}&#34;,
  &#34;retry_policy&#34;: &#34;{...}&#34;,
  &#34;request_mirror_policies&#34;: [],
  &#34;priority&#34;: &#34;...&#34;,
  &#34;rate_limits&#34;: [],
  &#34;include_vh_rate_limits&#34;: &#34;{...}&#34;,
  &#34;hash_policy&#34;: [],
  &#34;cors&#34;: &#34;{...}&#34;,
  &#34;max_grpc_timeout&#34;: &#34;{...}&#34;,
  &#34;grpc_timeout_offset&#34;: &#34;{...}&#34;,
  &#34;upgrade_configs&#34;: [],
  &#34;internal_redirect_policy&#34;: &#34;{...}&#34;,
  &#34;internal_redirect_action&#34;: &#34;...&#34;,
  &#34;max_internal_redirects&#34;: &#34;{...}&#34;,
  &#34;hedge_policy&#34;: &#34;{...}&#34;
}
```

### 常用路由策略

**基础路由配置**

- 在match中简单通过 &lt;font color=&#34;#f8070d&#34; size=3&gt;`prefix`&lt;/font&gt; 、&lt;font color=&#34;#f8070d&#34; size=3&gt;`path`&lt;/font&gt; 或 &lt;font color=&#34;#f8070d&#34; size=3&gt;`regex`&lt;/font&gt; 指定匹配条件；

- 将匹配到的请求进行重定向、直接响应或路由到指定目标集群

**高级路由策略**

- 在match中通过 &lt;font color=&#34;#f8070d&#34; size=3&gt;`prefix`&lt;/font&gt;、&lt;font color=&#34;#f8070d&#34; size=3&gt;`path`&lt;/font&gt; 或 &lt;font color=&#34;#f8070d&#34; size=3&gt;`regex`&lt;/font&gt; 指定匹配条件，并使用高级匹配机制;

- 结合 &lt;font color=&#34;#f8070d&#34; size=3&gt;`runtime_fraction`&lt;/font&gt; 按比例切割流量;

- 结合 &lt;font color=&#34;#f8070d&#34; size=3&gt;`headers`&lt;/font&gt; 按指定的标头路由，例如基于cookie进行，将其值分组后路由到不同目标；

- 结合 &lt;font color=&#34;#f8070d&#34; size=3&gt;`query_parameters`&lt;/font&gt; 按指定的参数路由，例如基于参数group进行，将其值分组后路由到不同的目标；

  ---

  **提示**：可灵活组合多种条件构建复杂的匹配机制

  ---

**复杂路由目标**

- 结合请求报文标头中cluster header的值进行定向路由

- &lt;font color=&#34;#f8070d&#34; size=3&gt;`weighted_clusters`&lt;/font&gt;：将请求根据目标集群权重进行流量分割

- 配置&lt;font color=&#34;#f8070d&#34; size=2&gt;高级路由属性&lt;/font&gt;，例如重试、超时、CORS、限途等；

### 配置示例

示例用于演示match的基本匹配机制及不同的路由方式

```yaml
routes:
- match:
    path: &#34;/service/color&#34;
  route:
    cluster: color
- match:
    regex: &#34;^/service/.*color$&#34;
  redirect:
    path_redirect: &#34;/service/color&#34;
- match:
    prefix: &#34;/service/version&#34;
  direct_response:
    status: 200
    boby:
      inline_string: &#34;this page is envoy reponsed&#34;
- match:
    prefix: &#34;/&#34;
  route:
    cluster: webserver
  
```

示例用于演示基于标头和查询参数的匹配；

```yaml
routes:
- match:
    prefix: &#34;/&#34;
    headers:
    - name: environment
      exact_match: &#34;2&#34;
    route:
      cluster: v2
- match:
  prefix: &#34;/&#34;
  query_parmeters:
  - name: &#34;user&#34;
    string_match:
      prefix: &#34;root&#34;
  route:
    cluster: default
- match:
    prefix: &#34;/&#34;
  route:
    cluster: v1
```

docker-compose文件

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
        - service_gray
        - front_envoy
    environment: 
    - VERSION=v1
    - COLORFUL=gray
    expose:
    - 90
  webserver2:
    image: sealloong/envoy-end:latest
    networks:
      envoymesh:
        aliases:
        - service_red
        - front_envoy
    environment: 
    - VERSION=v1
    - COLORFUL=blue
    expose:
    - 90

  service_gray:
    image: envoyproxy/envoy-alpine:v1.15-latest
    environment: 
    - ENVOY_UID=0
    volumes:
    - ./envoy.yaml:/etc/envoy/envoy.yaml
    networks:
      envoymesh:
        aliases:
        - envoy
    depends_on:
    - webserver1
    - webserver2

networks:
  envoymesh: {}
```

envoy.yaml

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
          &#34;@type&#34;: type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: base_domain
              domains: [ &#34;studyenvoy.eu&#34;, &#34;*.studyenvoy.eu&#34;, &#34;*studyenvoy.*&#34; ]
              routes:
              - match:
                  path: &#34;/service/colorful&#34;
                route:
                  prefix_rewrite: &#34;/colorful&#34;
                  cluster: color
              - match:
                  safe_regex: 
                    google_re2: {}
                    regex: &#34;^/service/.*colorful$&#34;
                redirect:
                  path_redirect: &#34;/service/colorful&#34;
              - match:
                  prefix: &#34;/service/version&#34;
                direct_response:
                  status: 200
                  body:
                    inline_string: &#34;this page is envoy reponsed&#34;
              - match: { prefix: &#34;/&#34; }
                route: { cluster: default }
            - name: &#34;base_parameter&#34;
              domains: [ &#34;*&#34; ]
              routes:
              - match:
                  prefix: &#34;/&#34;
                  headers:
                  - name: &#34;version&#34;
                    exact_match: &#34;v2&#34;
                route:
                  cluster: &#34;v2&#34;
              - match:
                  prefix: &#34;/&#34;
                  query_parameters:
                  - name: &#34;user&#34;
                    string_match:
                      prefix: &#34;root&#34;
                route:
                  cluster: &#34;default&#34;
              - match:
                  prefix: &#34;/&#34;
                route:
                  cluster: v1
          http_filters:
          - name: envoy.filters.http.router

  clusters:
  - name: default
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: default
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: default_server, port_value: 90 }
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
  - name: color
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: color
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: color_server, port_value: 90 }
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
  - name: v2
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: v2
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: v2_server, port_value: 90 }
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
  - name: v1
    connect_timeout: 0.25s
    type: STRICT_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: v1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address: { address: v1_server, port_value: 90 }
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


