# 

```
lc@lc-virtual-machine:~/data/src/cron/cronexpr/cronsingle$ go run main.go 
2019-09-28 10:04:52.830934665 +0800 CST m=+0.000258385 2019-09-28 10:05:00 +0800 CST

lc@lc-virtual-machine:~/data/src/cron/cronexpr/cronsingle$ go run main.go 
2019-09-28 23:39:59.583964716 +0800 CST m=+0.000297279 2019-09-28 23:40:00 +0800 CST

lc@lc-virtual-machine:~/data/src/cron/cronexpr/cronsingle$ go run main.go 
2019-09-28 23:40:04.225305549 +0800 CST m=+0.000261585 2019-09-28 23:45:00 +0800 CST

lc@lc-virtual-machine:~/data/src/cron/cronexpr/cronsingle$ go run main.go 
2019-09-29 00:24:51.678504279 +0800 CST m=+0.000228322 2019-09-29 00:25:00 +0800 CST

lc@lc-virtual-machine:~/data/src/cron/cronexpr/cronsingle$ go run main.go 
2019-09-29 00:25:06.171242203 +0800 CST m=+0.000266800 2019-09-29 00:30:00 +0800 CST
```

https://github.com/gorhill/cronexpr

cronexpr计算表达式，其计算调度时间是根据自己的小时分钟线， 如 `"*/5 * * * *"` 为 0,5,10,15,20,25,30..,60。不是根据启动cronexpr时间来计算的。


