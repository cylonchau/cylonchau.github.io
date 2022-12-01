# 

rabbitmq-plugins list

### 添加账号

```
rabbitmqctl set_user_tages luke luke
```


### 设置tag

```
rabbitmqctl set_user_tags luke 111
```



```
curl -X PUT   http://localhost:9200/store/books/1 -H "Content-type: application/json" -d '{
  "title": "Elasticsearch :The Definitive Guide",
  "name": {
    "first":"zachary",
    "last": "tong"
  },
  "publish_date":"2015-02-06",
  "price":"49.99"
}'

curl localhost:9200/store/books/1

{"_index":"store","_type":"books","_id":"1","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":{
"title": "Elasticsearch :The Definitive Guide",
"name": {
  "first":"zachary",
  "last": "tong"
 },
"publish_date":"2015-02-06",
"price":"49.99"
}}
```



```
curl http://172.18.3.106:9200/store/books/1?_source=title

{"_index":"store","_type":"books","_id":"1","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":{"title":"Elasticsearch :The Definitive Guide"}}
```





指定_source不指定字段，表示查询所有

curl http://172.18.3.106:9200/store/books/1?_source
{"_index":"store","_type":"books","_id":"1","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":{
"title": "Elasticsearch :The Definitive Guide",
"name": {
  "first":"zachary",
  "last": "tong"
 },
"publish_date":"2015-02-06",
"price":"49.99"
}}





curl http://172.18.3.106:9200/store/books/1?_source=title,publish_date
{"_index":"store","_type":"books","_id":"1","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":{"title":"Elasticsearch :The Definitive Guide","publish_date":"2015-02-06"}}





通过覆盖方式修改



```
curl -X PUT   http://172.18.3.106:9200/store/books/1 -H "Content-type: application/json" -d '{         
  "title": "Elasticsearch :The Definitive Guide",
  "name": {
    "first":"zachary",
    "last": "tong"
  },
  "publish_date":"2015-02-06",
  "price":"99"
}'
{"_index":"store","_type":"books","_id":"1","_version":2,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term":2}
```

通过_update API的方式单独更新某个字段

```
curl -X POST  'http://172.18.3.106:9200/store/books/1/_update' -H "Content-type: application/json" -d '{
    "doc":{
        "price":"199"

    }
}'

{"_index":"store","_type":"books","_id":"1","_version":3,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":2,"_primary_term":2}
```



#### 删除文档

```
curl -XDELETE 'http://172.18.3.106:9200/store/books/1'
```







复杂查询



query ： 相当于查询

- bool： 组合条件查询
  - must： 条件必须满足，相当于and  match_all 查询所有
  - should 条件可以满足也可以不满足，相当于or
  - must_not 条件不需要满足，相当于not

filter 过滤

term ： 在他下面的price是不能分词的

```
curl http://172.18.3.106:9200/store/books/_search -H "Content-type: application/json" -d '{
    "query":{
        "bool" :{
            "must":{
                "match_all" :{}
            },
            "filter": {
                "term" :{
                    "price" :39.99
                }
            }
        }
    }
}'

{"took":73,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"store","_type":"books","_id":"1","_score":1.0,"_source":{
  "title": "Elasticsearch :The Definitive Guide",
  "name": {
    "first":"li ",
    "last": "si "
  },
  "publish_date":"2015-02-06",
  "price":"39.99"
}}]}}


curl http://172.18.3.106:9200/store/books/_search -H "Content-type: application/json" -d '{
    "query":{
        "bool" :{
            "must":[
            	{"term" :{"price" :59.99}},
            	{"term" :{"price" :39.99}}
            ],
            "must_not": {
            	"term" : {"publish_date" :"2016-02-06"}
            }
        }
    }
}'

{"took":29,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":2,"max_score":0.2876821,"hits":[{"_index":"store","_type":"books","_id":"2","_score":0.2876821,"_source":{
  "title": "Elasticsearch :The Definitive Guide",
  "name": {
    "first":"zhang",
    "last": "san"
  },
  "publish_date":"2015-02-06",
  "price":"59.99"
}},{"_index":"store","_type":"books","_id":"1","_score":0.2876821,"_source":{
  "title": "Elasticsearch :The Definitive Guide",
  "name": {
    "first":"li ",
    "last": "si "
  },
  "publish_date":"2015-02-06",
  "price":"39.99"
}}]}}
```



```
curl http://172.18.3.106:9200/store/books/_search -H "Content-type: application/json" -d '{
    "query": {
        "constant_score": {
	        "filter": {
                "term" : {
                    "price":39.99
                }
            }
        }
    }
}'

{"took":3,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":1.0,"hits":[{"_index":"store","_type":"books","_id":"1","_score":1.0,"_source":{
  "title": "Elasticsearch :The Definitive Guide",
  "name": {
    "first":"li ",
    "last": "si "
  },
  "publish_date":"2015-02-06",
  "price":"39.99"
}}]}}
```

指定多个值



```
curl http://172.18.3.106:9200/store/books/_search -H "Content-type: application/json" -d '{
    "query": {
        "constant_score": {
	        "filter": {
                "terms" : {
                    "price":[39.99,99]
                }
            }
        }
    }
}'
```

range 范围过滤



- gt > 大于
- lt < 小于
- gte >= 大于等于
- lte <= 小于等于

```
curl http://172.18.3.106:9200/store/books/_search -H "Content-type: application/json" -d '{
    "query": {
        "range" : {
            "price":{
                "gte" :10,
                "lt" : 90
            }
        }
    }
}'
```

如果没有定义类型，比对的是字符串。



> 例：name和athor都必须包含Guide，并且价钱等于33.99或者188.99



```
curl http://172.18.3.106:9200/store/books/_search -H "Content-type: application/json" -d '{
    "query": {
        "bool": {
            "must": {
                "multi_match" : {
                    "operator": "and",
                    "fields": [
                        "name",
                        "author"
                    ],
                    "query": "Guide"
                }
            },
            "filter":{
                "terms" : {
                    "price": [
                        35.99,
                        188.99
                    ]
                }
            }
        }
    }
}'
```





查看setting

```
[root@node03 ~]# curl -XGET http://172.18.3.106:9200/news/_settings?pretty
{
  "news" : {
    "settings" : {
      "index" : {
        "creation_date" : "1580568630458",
        "number_of_shards" : "5",
        "number_of_replicas" : "1",
        "uuid" : "BLnOUQT5QPqBjCmEcJGoUA",
        "version" : {
          "created" : "6080699"
        },
        "provided_name" : "news"
      }
    }
  }
}
```

创建索引时设置setting

```
curl -XPUT  http://172.18.3.106:9200/store/ -H 'Content-Type:application/json' -d  '{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 2
    }
}'
```

动态修改已经存在索引的setting

---

**注**：此处只能修改副本，分片一旦创建就无法修改

---

```
curl -H 'Content-Type:application/json' -XPUT  http://172.18.3.106:9200/store1/_settings  -d  '{
    "index": {
        "number_of_replicas": 1
    }
}'
```



es核心概念



mapping


