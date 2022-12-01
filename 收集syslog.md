# ELK 收集syslog


Logstash通过网络将syslog消息作为事件读取。使用用 UDPSocket, TCPServer 和 

LogStash::Filters::Grok 来实现 


### 配置示例

```sh
input {                                            
    syslog{
        port => "514"
    }
}
```

配置客户端，在修改linux主机`/etc/rsyslog.conf`

```sh
*.* @@host:port
```

## 配置说明

| 参数 | 说明               |
| ---- | ------------------ |
| *    | 类型               |
| *    | 级别               |
| @    | udp                |
| @@   | tcp                |
| host | 可为主机名或ip地址 |

***
> **<font color="#0215cd" size=2>注：收集到的数据，本身就以及是rsyslog格式了，无需再进行grok</font>**
***

```sh
{
           "message" => "(root) CMD (/bin/echo 1111 >>/root/1.txt)\n",
          "@version" => "1",
        "@timestamp" => "2018-10-05T12:13:01.000Z",
              "host" => "10.0.0.16",
          "priority" => 78,
         "timestamp" => "Oct  5 20:13:01",
         "logsource" => "node02",
           "program" => "CROND",
               "pid" => "92923",
          "severity" => 6,
          "facility" => 9,
    "facility_label" => "clock",
    "severity_label" => "Informational"
}
```

收集到的数据需要对时间进行格式化

官方date插件说明：[Date filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html#plugins-filters-date-timezone)

```sh
filter {
		date {
				match => ["timestamp" , "MMM  dd HH:mm:ss"]
				target => "timestamp"
				"timezone" => "Asia/Shanghai"
		}
}
```

格式化完后的时间如下示例

```sh
{
           "message" => "(root) CMD (/bin/echo 1111 >>/root/1.txt)\n",
          "@version" => "1",
        "@timestamp" => "2018-10-05T12:09:01.000Z",
              "host" => "10.0.0.16",
          "priority" => 78,
         "timestamp" => "2018-10-05T12:09:01.000Z",
         "logsource" => "node02",
           "program" => "CROND",
               "pid" => "92825",
          "severity" => 6,
          "facility" => 9,
    "facility_label" => "clock",
    "severity_label" => "Informational"
}
```

***
**<font color="#0215cd" size=2>说明：此处格式化完后时间与当前时区不符，相差8小时。这里不影响，在kibana中显示的为当前时区</font>**

***

![image](../../images/收集syslog/dbefd2aa.png)
