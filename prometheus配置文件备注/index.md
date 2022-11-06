# 

备注信息参考 https://www.cnblogs.com/longcnblogs/p/9620733.html

```yaml
global:
  scrape_interval: 1m # 设定抓取数据的周期，默认为1min
  scrape_timeout: 30s # 设定更新rules文件的周期，默认为1min

rule_files: # alertmanager设置的报警规则
- &#34;/usr/share/prometheus/rules.yml&#34;

alerting:
  alertmanagers:
  - static_configs:
    - targets: [&#34;localhost:9093&#34;]

scrape_configs:
  - job_name: &#39;prometheus-server&#39; 
    static_configs:
    - targets: [&#39;127.0.0.1:19090&#39;]
  - job_name: &#39;php-fpm&#39;
    file_sd_configs: # 动态文件加载配置
    - files: 
      - &#34;/data/prometheus/sd_config/php-fpm.yml&#34;
      refresh_interval: 30s # 刷新时间
  - job_name: &#39;nginx&#39;
    metrics_path: &#39;/status/format/prometheus&#39; # 设置metric路由
    file_sd_configs:
    - files:
      - &#34;/data/prometheus/sd_config/nginx.yml&#34;
  - job_name: &#39;node&#39;
    file_sd_configs:
    - files:
      - &#34;/data/prometheus/sd_config/node.yml&#34;
remote_write:
- url: &#34;http://localhost:18086/api/v1/prom/write?db=prometheus&#34;
remote_read:
- url: &#34;http://localhost:18086/api/v1/prom/read?db=prometheus&#34;
```



nginx-config.yaml

```json
[
  {
    &#34;targets&#34;: [ &#39;192.168.8.31:8082&#39;,&#39;192.168.8.41:8082&#39;,&#39;192.168.8.51:8082&#39; ],
    &#34;labels&#34;: {
      &#34;service&#34;: &#34;nginx&#34;,
      &#34;role&#34;: &#34;admin&#34;
    }
  },

  {
    &#34;targets&#34;: [ &#39;192.168.8.32:8082&#39;,&#39;192.168.8.42:8082&#39;,&#39;192.168.8.52:8082&#39; ],
    &#34;labels&#34;: {
      &#34;service&#34;: &#34;nginx&#34;,
      &#34;role&#34;: &#34;api&#34;
    }
  },
  {
    &#34;targets&#34;: [ &#39;192.168.8.34:8082&#39;,&#39;192.168.8.44:8082&#39;,&#39;192.168.8.54:8082&#39; ],
    &#34;labels&#34;: {
      &#34;service&#34;: &#34;nginx&#34;,
      &#34;role&#34;: &#34;bank&#34;
    }
  },
  
  {
    &#34;targets&#34;: [ &#39;192.168.8.33:8082&#39;,&#39;192.168.8.43:8082&#39;,&#39;192.168.8.53:8082&#39; ],
    &#34;labels&#34;: {
      &#34;service&#34;: &#34;nginx&#34;,
      &#34;role&#34;: &#34;center&#34;
    }
  },

   {
    &#34;targets&#34;: [ &#39;192.168.8.35:8082&#39;,&#39;192.168.8.45:8082&#39;,&#39;192.168.8.55:8082&#39; ],
    &#34;labels&#34;: {
      &#34;service&#34;: &#34;nginx&#34;,
      &#34;role&#34;: &#34;thirdchannel&#34;
    }
  },
  {
    &#34;targets&#34;: [ &#39;192.168.8.30:8082&#39;,&#39;192.168.8.40:8082&#39;,&#39;192.168.8.50:8082&#39; ],
    &#34;labels&#34;: {
      &#34;service&#34;: &#34;nginx&#34;,
      &#34;role&#34;: &#34;proxy&#34;
    }
  },

]
```


