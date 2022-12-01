# telegram接收altermanager消息


```
curl -XPOST https://api.telegram.org/bot977657989:AAF0QE88WhxRIdpFLOYO_9ldLun5VtpfCWw/getUpdates

curl -X POST \
     -H 'Content-Type: application/json' \
     -d '{"chat_id": "850233746", "text": "This is a test from curl", "disable_notification": true}' \
     https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage
     
curl -X POST \
     -H 'Content-Type: application/json' \
     -d '{"chat_id": "850233746", "text": "This is a test from curl", "disable_notification": true}' \
     https://api.telegram.org/bot1009139816:AAGTmFsJDkH9H3E0OVoFi4GyvYp0uMctvcE/sendMessage
```

https://api.telegram.org/bot721202655:AAG_kN1IHP93Wmnd90RRaJC-dK9tKQHddRA/sendMessage

```json
{
	"chat_id": "-383641009",
	"text": "This is a test from curl", 
	"disable_notification": true
}
```

alertmanager发送的消息类型如下：


```json
{
	"receiver": "webhook",
	"status": "firing",
	"alerts": [{
		"status": "firing",
		"labels": {
			"alertname": "InstanceDown",
			"instance": "192.168.8.32:9110",
			"job": "node",
			"role": "api",
			"service": "node"
		},
		"annotations": {
			"description": "192.168.8.32:9110: job node has been down vlaue==0",
			"summary": "192.168.8.32:9110: has been down"
		},
		"startsAt": "2019-07-25T21:55:13.352482605+08:00",
		"endsAt": "0001-01-01T00:00:00Z",
		"generatorURL": "http://zy-tw-tianxia-prod-prometheus01:19090/graph?g0.expr=up+%3D%3D+0\u0026g0.tab=1"
	}, {
		"status": "firing",
		"labels": {
			"alertname": "InstanceDown",
			"instance": "192.168.8.34:9110",
			"job": "node",
			"role": "api",
			"service": "node"
		},
		"annotations": {
			"description": "192.168.8.34:9110: job node has been down vlaue==0",
			"summary": "192.168.8.34:9110: has been down"
		},
		"startsAt": "2019-07-25T21:53:13.352482605+08:00",
		"endsAt": "0001-01-01T00:00:00Z",
		"generatorURL": "http://zy-tw-tianxia-prod-prometheus01:19090/graph?g0.expr=up+%3D%3D+0\u0026g0.tab=1"
	}],
	"groupLabels": {
		"alertname": "InstanceDown"
	},
	"commonLabels": {
		"alertname": "InstanceDown",
		"job": "node",
		"role": "api",
		"service": "node"
	},
	"commonAnnotations": {},
	"externalURL": "http://zy-tw-tianxia-prod-prometheus01:9093",
	"version": "4",
	"groupKey": "{}:{alertname=\"InstanceDown\"}"
}
```

https://www.qikqiak.com/post/prometheus-operator-custom-alert/


### telegram在文本消息中插入换行符


尝试'％0D'或'％0A'  参考地址：https://stackoverrun.com/cn/q/8787508



telegram api 使用

https://stackoverrun.com/cn/q/8787508
