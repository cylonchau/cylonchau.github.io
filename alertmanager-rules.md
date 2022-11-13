# 

```yaml
groups:
- name: node-rules
  rules:
  - alert: NodeDown # 告警名称
    expr: up{job=&#34;node&#34;} == 0 # 告警的判定条件,使用promsql
    for: 10s # 满足告警条件持续时间多久后，才会发送告警
    annotations: # 解析项，详细解释告警信息
      summary: &#34;{{$labels.job}}: has been down&#34;
      description: &#34;{{$labels.instance}}: job {{$labels.job}} has been down vlaue=={{$value}}&#34;
  - alert: &#39;cpu use percent gt 85%&#39;
    expr: ceil(100 - avg(irate(node_cpu_seconds_total{mode=&#34;idle&#34;}[7m])) by(instance)*100) &gt; 85
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: cpu use percent gt 85%&#34;
      description: &#34;{{$labels.instance}}: job {{$labels.job}} cpu current use = {{$value}}&#34;
  - alert: &#39;Free disk space is less than 20%&#39;
    expr: floor((1-(node_filesystem_free_bytes{fstype=~&#34;ext4|xfs&#34;} / node_filesystem_size_bytes{fstype=~&#34;ext4|xfs&#34;})) * 100) &gt; 85
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Free disk space is less than 20%&#34;
      description: &#34;{{$labels.instance}}: job {{$labels.job}} Current Used disk space = {{$value}}&#34;
  - alert: &#39;Free Member space is less than 20%&#39;
    expr: ceil((ceil(node_memory_MemTotal_bytes /1024/1024/1024) - ceil((node_memory_Cached_bytes &#43; node_memory_Buffers_bytes &#43; node_memory_MemFree_bytes) /1024/1024/1024)) / ceil((node_memory_Cached_bytes &#43; node_memory_Buffers_bytes &#43; node_memory_MemFree_bytes) /1024/1024/1024) * 100) &gt;= 80
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Free disk space is less than 20%&#34;
      description: &#34;{{$labels.instance}}: job {{$labels.job}} Current Used Member space = {{$value}}G&#34;
- name: php-rules
  rules:
  - alert: PHP-experter Down
    expr: up{job=&#34;php-fpm&#34;} == 0
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: has been down&#34;
      description: &#34;{{$labels.instance}}: job {{$labels.job}} has been down vlaue=={{$value}}&#34;
  - alert: &#39;Slow Requests过多&#39;
    expr: floor(sum(increase(phpfpm_slow_requests_total{job=&#34;php-fpm&#34;}[1m])) by(instance)) &gt; 10
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: PHP-FPM slow requests过多&#34;
      description: &#34;1分钟内{{$labels.instance}}: {{$labels.job}} slow requests vlaue: {{$value}}&#34;
  - alert: PHP-FPMDown
    expr: phpfpm_up{job=&#34;php-fpm&#34;} == 0
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: has been down&#34;
      description: &#34;{{$labels.instance}}: job {{$labels.job}} has been down vlaue=={{$value}}&#34;
- name: nginx-rules
  rules:
  - alert: nginx499
    expr: floor(increase(nginx_vts_filter_requests_total{direction=&#34;4xx&#34;,filter_name=&#34;499&#34;,job=&#34;nginx&#34;}[5m])) &gt; 10
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Too many&#34;
      description: &#34;1分钟内 {{$labels.instance}}: {{$labels.job}} Current value: {{$value}}&#34;
  - alert: nginx502
    expr: floor(increase(nginx_vts_filter_requests_total{direction=&#34;5xx&#34;,filter_name=&#34;502&#34;,job=&#34;nginx&#34;}[5m])) &gt; 10
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Too many&#34;
      description: &#34;1分钟内 {{$labels.instance}}: {{$labels.job}} Current value: {{$value}}&#34;
  - alert: nginx403
    expr: floor(increase(nginx_vts_filter_requests_total{direction=&#34;4xx&#34;,filter_name=&#34;403&#34;,job=&#34;nginx&#34;}[5m])) &gt; 200
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Too many&#34;
      description: &#34;1分钟内 {{$labels.instance}}: {{$labels.job}} Current value: {{$value}}&#34;
  - alert: nginx504
    expr: floor(increase(nginx_vts_filter_requests_total{direction=&#34;5xx&#34;,filter_name=&#34;504&#34;,job=&#34;nginx&#34;}[5m])) &gt; 5
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Too many&#34;
      description: &#34;1分钟内 {{$labels.instance}}: {{$labels.job}} Current value: {{$value}}&#34;
  - alert: nginx500
    expr: floor(increase(nginx_vts_filter_requests_total{direction=&#34;5xx&#34;,filter_name=&#34;500&#34;,job=&#34;nginx&#34;}[5m])) &gt; 10
    for: 10s
    annotations:
      summary: &#34;{{$labels.job}}: Too many&#34;
      description: &#34;1分钟内 {{$labels.instance}}: {{$labels.job}} Current value: {{$value}}&#34;
```

5、告警信息生命周期的3中状态

- inactive：表示当前报警信息即不是firing状态也不是pending状态
- pending：表示在设置的阈值时间范围内被激活的
- firing：表示超过设置的阈值时间被激活的
- resolved 告警恢复
