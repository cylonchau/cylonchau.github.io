# 

备注信息参考 https://www.cnblogs.com/longcnblogs/p/9620733.html

```yaml
global:
  scrape_interval: 1m # 设定抓取数据的周期，默认为1min
  scrape_timeout: 30s # 设定更新rules文件的周期，默认为1min

rule_files: # alertmanager设置的报警规则
- "/usr/share/prometheus/rules.yml"

alerting:
  alertmanagers:
  - static_configs:
    - targets: ["localhost:9093"]

scrape_configs:
  - job_name: 'prometheus-server' 
    static_configs:
    - targets: ['127.0.0.1:19090']
  - job_name: 'php-fpm'
    file_sd_configs: # 动态文件加载配置
    - files: 
      - "/data/prometheus/sd_config/php-fpm.yml"
      refresh_interval: 30s # 刷新时间
  - job_name: 'nginx'
    metrics_path: '/status/format/prometheus' # 设置metric路由
    file_sd_configs:
    - files:
      - "/data/prometheus/sd_config/nginx.yml"
  - job_name: 'node'
    file_sd_configs:
    - files:
      - "/data/prometheus/sd_config/node.yml"
remote_write:
- url: "http://localhost:18086/api/v1/prom/write?db=prometheus"
remote_read:
- url: "http://localhost:18086/api/v1/prom/read?db=prometheus"
```



nginx-config.yaml

```json
[
  {
    "targets": [ '192.168.8.31:8082','192.168.8.41:8082','192.168.8.51:8082' ],
    "labels": {
      "service": "nginx",
      "role": "admin"
    }
  },

  {
    "targets": [ '192.168.8.32:8082','192.168.8.42:8082','192.168.8.52:8082' ],
    "labels": {
      "service": "nginx",
      "role": "api"
    }
  },
  {
    "targets": [ '192.168.8.34:8082','192.168.8.44:8082','192.168.8.54:8082' ],
    "labels": {
      "service": "nginx",
      "role": "bank"
    }
  },
  
  {
    "targets": [ '192.168.8.33:8082','192.168.8.43:8082','192.168.8.53:8082' ],
    "labels": {
      "service": "nginx",
      "role": "center"
    }
  },

   {
    "targets": [ '192.168.8.35:8082','192.168.8.45:8082','192.168.8.55:8082' ],
    "labels": {
      "service": "nginx",
      "role": "thirdchannel"
    }
  },
  {
    "targets": [ '192.168.8.30:8082','192.168.8.40:8082','192.168.8.50:8082' ],
    "labels": {
      "service": "nginx",
      "role": "proxy"
    }
  },

]
```


