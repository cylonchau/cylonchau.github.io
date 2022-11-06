# ELK 收集syslog


Logstash通过网络将syslog消息作为事件读取。使用用 UDPSocket, TCPServer 和 

LogStash::Filters::Grok 来实现 


### 配置示例

```sh
input {                                            
    syslog{
        port =&gt; &#34;514&#34;
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
&gt; **&lt;font color=&#34;#0215cd&#34; size=2&gt;注：收集到的数据，本身就以及是rsyslog格式了，无需再进行grok&lt;/font&gt;**
***

```sh
{
           &#34;message&#34; =&gt; &#34;(root) CMD (/bin/echo 1111 &gt;&gt;/root/1.txt)\n&#34;,
          &#34;@version&#34; =&gt; &#34;1&#34;,
        &#34;@timestamp&#34; =&gt; &#34;2018-10-05T12:13:01.000Z&#34;,
              &#34;host&#34; =&gt; &#34;10.0.0.16&#34;,
          &#34;priority&#34; =&gt; 78,
         &#34;timestamp&#34; =&gt; &#34;Oct  5 20:13:01&#34;,
         &#34;logsource&#34; =&gt; &#34;node02&#34;,
           &#34;program&#34; =&gt; &#34;CROND&#34;,
               &#34;pid&#34; =&gt; &#34;92923&#34;,
          &#34;severity&#34; =&gt; 6,
          &#34;facility&#34; =&gt; 9,
    &#34;facility_label&#34; =&gt; &#34;clock&#34;,
    &#34;severity_label&#34; =&gt; &#34;Informational&#34;
}
```

收集到的数据需要对时间进行格式化

官方date插件说明：[Date filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html#plugins-filters-date-timezone)

```sh
filter {
		date {
				match =&gt; [&#34;timestamp&#34; , &#34;MMM  dd HH:mm:ss&#34;]
				target =&gt; &#34;timestamp&#34;
				&#34;timezone&#34; =&gt; &#34;Asia/Shanghai&#34;
		}
}
```

格式化完后的时间如下示例

```sh
{
           &#34;message&#34; =&gt; &#34;(root) CMD (/bin/echo 1111 &gt;&gt;/root/1.txt)\n&#34;,
          &#34;@version&#34; =&gt; &#34;1&#34;,
        &#34;@timestamp&#34; =&gt; &#34;2018-10-05T12:09:01.000Z&#34;,
              &#34;host&#34; =&gt; &#34;10.0.0.16&#34;,
          &#34;priority&#34; =&gt; 78,
         &#34;timestamp&#34; =&gt; &#34;2018-10-05T12:09:01.000Z&#34;,
         &#34;logsource&#34; =&gt; &#34;node02&#34;,
           &#34;program&#34; =&gt; &#34;CROND&#34;,
               &#34;pid&#34; =&gt; &#34;92825&#34;,
          &#34;severity&#34; =&gt; 6,
          &#34;facility&#34; =&gt; 9,
    &#34;facility_label&#34; =&gt; &#34;clock&#34;,
    &#34;severity_label&#34; =&gt; &#34;Informational&#34;
}
```

***
**&lt;font color=&#34;#0215cd&#34; size=2&gt;说明：此处格式化完后时间与当前时区不符，相差8小时。这里不影响，在kibana中显示的为当前时区&lt;/font&gt;**

***

![image](../../images/收集syslog/dbefd2aa.png)
