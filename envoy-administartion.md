# envoy 管理


envoy内建了一个管理接口，它支持查询和修改操作，甚至有可能暴露私有数据（例如统计数据、集群名称和证书信息等）因此非常有必要精心编排期访问控制机制，以避免非授权访问。

`administration interface` 属于 `bootstrap` 配置文件中的==顶级配置字段==使用。

[administration interface offical document](https://www.envoyproxy.io/docs/envoy/v1.15.0/api-v3/admin/admin)


```yaml
admin:
  access_log_path: .. # 管理接口的访问日志文件路径，无须记录访问日志使用/dev/null
  profile_path: ...   # cpu_profile的输出路径，默认为/var/log/envoy/envoy.prof;
  address:
    socket_address:   # 监听的套接字
      protocol: ..
      address: ...
      prot_value: ...
```

下面是一个简单的配置示例

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }
```

admin接口内置了多个 `/path` ，不同的path可能会分别接受不同的GET或POST请求

- GET `/help` ：<font color="#f8070d;" size=2>打印所有可用选项；</font>
- `/` : Admin home page
- `/certs` : <font color="#f8070d;" size=2>GET 列出在的所有TLS相关的信息；</font>
- `/clusters` : upstream cluster status <font color="#f8070d;" size=2>GET，格外支持使用 `/clusters?format=json` ；</font>
- `/config_dump` : dump current Envoy configs (experimental) <font color="#f8070d;" size=2>GET，打印Envoy 加载的各类配置信息；</font>
- `/contention` : dump current Envoy mutex contention stats (if enabled) <font color="#f8070d;" size=2>GET 互斥跟踪；</font>
- `/cpuprofiler` : enable/disable the CPU profiler POST <font color="#f8070d;" size=2>启用/禁用cupprofiler。</font>
- `/drain_listeners` : drain listeners 
- `/healthcheck/fail` : cause the server to fail health checks <font color="#f8070d;" size=2>POST 强制设定HTTP健康状态检查为失败；</font>
- `/healthcheck/ok` : cause the server to pass health checks <font color="#f8070d;" size=2>POST 强制设定HTTP健康状态检查为成功；</font>
- `/heapprofiler` : enable/disable the heap profiler <font color="#f8070d;" size=2>POST 启用或禁用heapprofiler；</font>
- `/help` : print out list of admin commands 
- `/hot_restart_version` : print the hot restart compatibility version <font color="#f8070d;" size=2>GET 热重启相关信息；</font>
- `/listeners` : print listener info <font color="#f8070d;" size=2>GET 列出所有侦听器，支持使用 `GET /listeners?format=json` </font>
- `/logging` : query/change logging levels <font color="#f8070d;" size=2>POST，弃用或禁用不同子组件上的不同日志记录级别；</font>
- `/memory` : print current allocation/heap usage <font color="#f8070d;" size=2>POST，打印当前内在分配信息，以字节为单位；；</font>
- `/quitquitquit` : exit the server <font color="#f8070d;" size=2>POST，干净退出服务器；</font>
- `/ready` : print server state, return 200 if LIVE, otherwise return 503 
- `/reopen_logs` : reopen access logs
- `/reset_counters` : reset all counters to zero <font color="#f8070d;" size=2>POST，重置所有计数器；</font>
- `/runtime` : print runtime values <font color="#f8070d;" size=2>GET，以json格式输出所有运行时相关值；</font>
- `/runtime_modify` : modify runtime values <font color="#f8070d;" size=2>POST `/runtime_modify?k1=v1&k2=v2` 添加或修改在查询参数中传递的运行时值；</font>
- `/server_info` : print server version/status information  <font color="#f8070d;" size=2>GET，打印当前Envoy Server相关信息；</font>
- `/stats` : print server stats <font color="#f8070d;" size=2>按需输出统计数据，如：`GET /stats?filter=reger`，另外还支持json和promotheus两种格式输出；</font>
- `/stats/prometheus` : print server stats in prometheus format
- `/stats/recentlookups` : Show recent stat-name lookups
- `/stats/recentlookups/clear` : clear list of stat-name lookups and counter
- `/stats/recentlookups/enable` : enable recording of reset stat-name lookup names



**集群统计信息中主机状态的说明**

| Name            | Type    | Description                                                  |
| --------------- | ------- | :----------------------------------------------------------- |
| cx_total        | Counter | Total connecions                                             |
| cx_active       | Gauge   | Total active coinnections                                    |
| cx_connect_fail | Counter | Total connection failures                                    |
| rq_total        | Counter | Total requests                                               |
| rq_timeout      | Counter | Total timed out requests                                     |
| rq_success      | Counter | Total requests with non-5xx responses                        |
| rq_error        | Counter | Total requests with 5xx responses                            |
| rq_active       | Gauge   | Total active requests                                        |
| healthy         | String  | The health status of the host. See below                     |
| weight          | Integer | Load balancing weight(1-100)                                 |
| zone            | String  | Service zone                                                 |
| canary          | Boolean | Whether the host is a canary                                 |
| success_rate    | Double  | Request success rate (0-100). -1 if there was not enough request volume in the interval to calculate it |

 示例总结

- GET `/clusters` ：列出所有已配置的集群，包括每个集群中发现的所有上游主机以及每个主机的统计信息；支持输出为json格式；
  - 集群管理器信息：`version_info` string，无CDS时，则显示为 `version_info::static`
  - 集群相关的信息：断路器、异常点检测和用于表示是否通过CDS添加的标识 `add_via_api`
  - 每个主机的统计信息：包括总连接数、活动连接数、总请求数和主机的健康状态等；不健康的原因通常有以下三种：
    - √ filed_activehc：未通过主动健康状态检测；
    - √ failed_edshelth：被EDS标记为不健康；
    - √ failed_outlier_check：未通过异常检测机制的检查；
- GET `/listeners` ：列出所有已配置的侦听器，包括侦听器的名称以及监听的地址；支持输出为json格式；
- POST `/reset_counters` ：将所有计数器重围为0；不过，它只会影响Server本地的输出，对于已经发送到外部存储系统的统计数据无效；
- GET `/config_dump` ：以json格式打印当前从Envoy的各种组件加载的配置信息；
- GET `/ready` ：获取Server就绪与否的状态，LIVE状态为200，否则为503；
