# telegram接收altermanager消息


```
curl -XPOST https://api.telegram.org/bot977657989:AAF0QE88WhxRIdpFLOYO_9ldLun5VtpfCWw/getUpdates

curl -X POST \
     -H &#39;Content-Type: application/json&#39; \
     -d &#39;{&#34;chat_id&#34;: &#34;850233746&#34;, &#34;text&#34;: &#34;This is a test from curl&#34;, &#34;disable_notification&#34;: true}&#39; \
     https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage
     
curl -X POST \
     -H &#39;Content-Type: application/json&#39; \
     -d &#39;{&#34;chat_id&#34;: &#34;850233746&#34;, &#34;text&#34;: &#34;This is a test from curl&#34;, &#34;disable_notification&#34;: true}&#39; \
     https://api.telegram.org/bot1009139816:AAGTmFsJDkH9H3E0OVoFi4GyvYp0uMctvcE/sendMessage
```

https://api.telegram.org/bot721202655:AAG_kN1IHP93Wmnd90RRaJC-dK9tKQHddRA/sendMessage

```json
{
	&#34;chat_id&#34;: &#34;-383641009&#34;,
	&#34;text&#34;: &#34;This is a test from curl&#34;, 
	&#34;disable_notification&#34;: true
}
```

alertmanager发送的消息类型如下：


```json
{
	&#34;receiver&#34;: &#34;webhook&#34;,
	&#34;status&#34;: &#34;firing&#34;,
	&#34;alerts&#34;: [{
		&#34;status&#34;: &#34;firing&#34;,
		&#34;labels&#34;: {
			&#34;alertname&#34;: &#34;InstanceDown&#34;,
			&#34;instance&#34;: &#34;192.168.8.32:9110&#34;,
			&#34;job&#34;: &#34;node&#34;,
			&#34;role&#34;: &#34;api&#34;,
			&#34;service&#34;: &#34;node&#34;
		},
		&#34;annotations&#34;: {
			&#34;description&#34;: &#34;192.168.8.32:9110: job node has been down vlaue==0&#34;,
			&#34;summary&#34;: &#34;192.168.8.32:9110: has been down&#34;
		},
		&#34;startsAt&#34;: &#34;2019-07-25T21:55:13.352482605&#43;08:00&#34;,
		&#34;endsAt&#34;: &#34;0001-01-01T00:00:00Z&#34;,
		&#34;generatorURL&#34;: &#34;http://zy-tw-tianxia-prod-prometheus01:19090/graph?g0.expr=up&#43;%3D%3D&#43;0\u0026g0.tab=1&#34;
	}, {
		&#34;status&#34;: &#34;firing&#34;,
		&#34;labels&#34;: {
			&#34;alertname&#34;: &#34;InstanceDown&#34;,
			&#34;instance&#34;: &#34;192.168.8.34:9110&#34;,
			&#34;job&#34;: &#34;node&#34;,
			&#34;role&#34;: &#34;api&#34;,
			&#34;service&#34;: &#34;node&#34;
		},
		&#34;annotations&#34;: {
			&#34;description&#34;: &#34;192.168.8.34:9110: job node has been down vlaue==0&#34;,
			&#34;summary&#34;: &#34;192.168.8.34:9110: has been down&#34;
		},
		&#34;startsAt&#34;: &#34;2019-07-25T21:53:13.352482605&#43;08:00&#34;,
		&#34;endsAt&#34;: &#34;0001-01-01T00:00:00Z&#34;,
		&#34;generatorURL&#34;: &#34;http://zy-tw-tianxia-prod-prometheus01:19090/graph?g0.expr=up&#43;%3D%3D&#43;0\u0026g0.tab=1&#34;
	}],
	&#34;groupLabels&#34;: {
		&#34;alertname&#34;: &#34;InstanceDown&#34;
	},
	&#34;commonLabels&#34;: {
		&#34;alertname&#34;: &#34;InstanceDown&#34;,
		&#34;job&#34;: &#34;node&#34;,
		&#34;role&#34;: &#34;api&#34;,
		&#34;service&#34;: &#34;node&#34;
	},
	&#34;commonAnnotations&#34;: {},
	&#34;externalURL&#34;: &#34;http://zy-tw-tianxia-prod-prometheus01:9093&#34;,
	&#34;version&#34;: &#34;4&#34;,
	&#34;groupKey&#34;: &#34;{}:{alertname=\&#34;InstanceDown\&#34;}&#34;
}
```

https://www.qikqiak.com/post/prometheus-operator-custom-alert/


### telegram在文本消息中插入换行符


尝试&#39;％0D&#39;或&#39;％0A&#39;  参考地址：https://stackoverrun.com/cn/q/8787508



telegram api 使用

https://stackoverrun.com/cn/q/8787508
