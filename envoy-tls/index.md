# Envoy TLS 配置


[official front proxy](https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/front_proxy)

在一个Services Mash内部中，都会存在一到多个微服务，为了将南北向流量统一引入网格内部管理，通常存在单独运行的envoy实例。

Envoy的listener支持面向下游客户端一侧的TLS会话，并可选地支持验正客户端证书；

listener中用到的数字证书可于配置中静态提供，也可借助于SDS动态获取;

`tls_context` 是 `listeners` 当中某一个 `filter_chains` 中上线文中的配置，`tls_context` 配置在哪个 listeners当中就隶属于哪个listeners。

```yaml
listeners:
...
  filter_chains:
  - filters:
    ..
    tls_context:
      common_tls_context: {} # 常规证书相关设置；
        tls_params: {}       # TLS协议版本，加密套件等；
        tls_certifcates: []  # 用到的证书和私钥文件；
        - certifcate_chain: {}
            filename:        # 证书文件路径;
          private_key: {}    # 私钥;
            filename:        # 私钥文件路径;
          password: {}       # 私钥口令；
            filename:        # 口令文件路径；
        tls_certifcate_sds_secret_configs: [] # 要基于SDS API获取TLS会话的相关信息时的配置；
      require_client_certifcate: # 是否验证客户端证书；
```

例如，下面示例将前面的Ingress示例中的Envoy配置为通过TLS提供服务，并将所有基于http协议的请求重定向至https；

```yaml
admin:
  access_log_path: /dev/null
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listeners_http
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 }
    filter_chains:
    - filters:
      - name: envoy.http_connenttion_manager
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
                redirect:
                  path_redirect: &#34;/&#34;
                  https_redirect: true
          http_filters:
          - name: envoy.router
  - name: listener_https
    address:
      socket_address: { address: 0.0.0.0, port_value: 443 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
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
          - name: envoy.router
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          &#34;@type&#34;: type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
          common_tls_context:
            tls_certificates:
              certificate_chain:
                filename: &#34;/etc/envoy/certs/server.crt&#34;
              private_key:
                filename: &#34;/etc/envoy/certs/server.key&#34;

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
```

将自定义配置组织于专用目录（例如ingres.htps_proxy/）下的envoy.yaml文件中，并创建Dockerfile文件构建自定义镜像；

- 生成测试使用的数字证书；

```
mkdir -p certs

openssl req -nodes -new -x509 -keyout certs/server.key -out certs/server.crt -days 3650 -subj &#39;/CN=ik8s.io/O=MageEdu LTD./C=CN&#39;

openssl req -nodes -new -key certs/server.key -out certs/server.crt -days 365 -subj &#34;/C=CN/ST=Guangdong/L=Guangzhou/O=xdevops/OU=xdevops/CN=*.xdevops.cn&#34;
```

docker-compose文件示例

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
    - ./certs:/etc/envoy/certs
    networks:
      envoymesh:
        aliases:
        - envoy
    depends_on:
    - webserver
  
  webserver:
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






