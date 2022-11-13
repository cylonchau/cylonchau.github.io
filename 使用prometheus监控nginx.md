# 

prometheus 监控nginx的模块 nginx-module-vts 

https://github.com/vozlt/nginx-module-vts




下载后配置

```conf
http{
    vhost_traffic_status_zone; 
    vhost_traffic_status_zone shared:vhost_traffic_status:32m; #  设置共享内存大小
    server {
    
        vhost_traffic_status_filter_by_set_key  $status $server_name; # 计算详细的http状态代码的流量
    
        location /status {
            vhost_traffic_status_display; # 设置了该指令，则可以访问如下：
            vhost_traffic_status_display_format html;
            vhost_traffic_status off; ## 启用或禁用模块工作
        }
        
        
    }
}
```

不想统计流量的server区域禁用`vhost_traffic_statu off`

例： 计算upstream后端相应时间 nginx_upstream_responseMsec{upstream=“group1”}


