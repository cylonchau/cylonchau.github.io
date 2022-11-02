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
curl -X PUT   http://localhost:9200/store/books/1 -H &#34;Content-type: application/json&#34; -d &#39;{
  &#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
  &#34;name&#34;: {
    &#34;first&#34;:&#34;zachary&#34;,
    &#34;last&#34;: &#34;tong&#34;
  },
  &#34;publish_date&#34;:&#34;2015-02-06&#34;,
  &#34;price&#34;:&#34;49.99&#34;
}&#39;

curl localhost:9200/store/books/1

{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_version&#34;:1,&#34;_seq_no&#34;:0,&#34;_primary_term&#34;:1,&#34;found&#34;:true,&#34;_source&#34;:{
&#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
&#34;name&#34;: {
  &#34;first&#34;:&#34;zachary&#34;,
  &#34;last&#34;: &#34;tong&#34;
 },
&#34;publish_date&#34;:&#34;2015-02-06&#34;,
&#34;price&#34;:&#34;49.99&#34;
}}
```



```
curl http://172.18.3.106:9200/store/books/1?_source=title

{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_version&#34;:1,&#34;_seq_no&#34;:0,&#34;_primary_term&#34;:1,&#34;found&#34;:true,&#34;_source&#34;:{&#34;title&#34;:&#34;Elasticsearch :The Definitive Guide&#34;}}
```





指定_source不指定字段，表示查询所有

curl http://172.18.3.106:9200/store/books/1?_source
{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_version&#34;:1,&#34;_seq_no&#34;:0,&#34;_primary_term&#34;:1,&#34;found&#34;:true,&#34;_source&#34;:{
&#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
&#34;name&#34;: {
  &#34;first&#34;:&#34;zachary&#34;,
  &#34;last&#34;: &#34;tong&#34;
 },
&#34;publish_date&#34;:&#34;2015-02-06&#34;,
&#34;price&#34;:&#34;49.99&#34;
}}





curl http://172.18.3.106:9200/store/books/1?_source=title,publish_date
{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_version&#34;:1,&#34;_seq_no&#34;:0,&#34;_primary_term&#34;:1,&#34;found&#34;:true,&#34;_source&#34;:{&#34;title&#34;:&#34;Elasticsearch :The Definitive Guide&#34;,&#34;publish_date&#34;:&#34;2015-02-06&#34;}}





通过覆盖方式修改



```
curl -X PUT   http://172.18.3.106:9200/store/books/1 -H &#34;Content-type: application/json&#34; -d &#39;{         
  &#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
  &#34;name&#34;: {
    &#34;first&#34;:&#34;zachary&#34;,
    &#34;last&#34;: &#34;tong&#34;
  },
  &#34;publish_date&#34;:&#34;2015-02-06&#34;,
  &#34;price&#34;:&#34;99&#34;
}&#39;
{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_version&#34;:2,&#34;result&#34;:&#34;updated&#34;,&#34;_shards&#34;:{&#34;total&#34;:2,&#34;successful&#34;:1,&#34;failed&#34;:0},&#34;_seq_no&#34;:1,&#34;_primary_term&#34;:2}
```

通过_update API的方式单独更新某个字段

```
curl -X POST  &#39;http://172.18.3.106:9200/store/books/1/_update&#39; -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;doc&#34;:{
        &#34;price&#34;:&#34;199&#34;

    }
}&#39;

{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_version&#34;:3,&#34;result&#34;:&#34;updated&#34;,&#34;_shards&#34;:{&#34;total&#34;:2,&#34;successful&#34;:1,&#34;failed&#34;:0},&#34;_seq_no&#34;:2,&#34;_primary_term&#34;:2}
```



#### 删除文档

```
curl -XDELETE &#39;http://172.18.3.106:9200/store/books/1&#39;
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
curl http://172.18.3.106:9200/store/books/_search -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;query&#34;:{
        &#34;bool&#34; :{
            &#34;must&#34;:{
                &#34;match_all&#34; :{}
            },
            &#34;filter&#34;: {
                &#34;term&#34; :{
                    &#34;price&#34; :39.99
                }
            }
        }
    }
}&#39;

{&#34;took&#34;:73,&#34;timed_out&#34;:false,&#34;_shards&#34;:{&#34;total&#34;:5,&#34;successful&#34;:5,&#34;skipped&#34;:0,&#34;failed&#34;:0},&#34;hits&#34;:{&#34;total&#34;:1,&#34;max_score&#34;:1.0,&#34;hits&#34;:[{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_score&#34;:1.0,&#34;_source&#34;:{
  &#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
  &#34;name&#34;: {
    &#34;first&#34;:&#34;li &#34;,
    &#34;last&#34;: &#34;si &#34;
  },
  &#34;publish_date&#34;:&#34;2015-02-06&#34;,
  &#34;price&#34;:&#34;39.99&#34;
}}]}}


curl http://172.18.3.106:9200/store/books/_search -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;query&#34;:{
        &#34;bool&#34; :{
            &#34;must&#34;:[
            	{&#34;term&#34; :{&#34;price&#34; :59.99}},
            	{&#34;term&#34; :{&#34;price&#34; :39.99}}
            ],
            &#34;must_not&#34;: {
            	&#34;term&#34; : {&#34;publish_date&#34; :&#34;2016-02-06&#34;}
            }
        }
    }
}&#39;

{&#34;took&#34;:29,&#34;timed_out&#34;:false,&#34;_shards&#34;:{&#34;total&#34;:5,&#34;successful&#34;:5,&#34;skipped&#34;:0,&#34;failed&#34;:0},&#34;hits&#34;:{&#34;total&#34;:2,&#34;max_score&#34;:0.2876821,&#34;hits&#34;:[{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;2&#34;,&#34;_score&#34;:0.2876821,&#34;_source&#34;:{
  &#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
  &#34;name&#34;: {
    &#34;first&#34;:&#34;zhang&#34;,
    &#34;last&#34;: &#34;san&#34;
  },
  &#34;publish_date&#34;:&#34;2015-02-06&#34;,
  &#34;price&#34;:&#34;59.99&#34;
}},{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_score&#34;:0.2876821,&#34;_source&#34;:{
  &#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
  &#34;name&#34;: {
    &#34;first&#34;:&#34;li &#34;,
    &#34;last&#34;: &#34;si &#34;
  },
  &#34;publish_date&#34;:&#34;2015-02-06&#34;,
  &#34;price&#34;:&#34;39.99&#34;
}}]}}
```



```
curl http://172.18.3.106:9200/store/books/_search -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;query&#34;: {
        &#34;constant_score&#34;: {
	        &#34;filter&#34;: {
                &#34;term&#34; : {
                    &#34;price&#34;:39.99
                }
            }
        }
    }
}&#39;

{&#34;took&#34;:3,&#34;timed_out&#34;:false,&#34;_shards&#34;:{&#34;total&#34;:5,&#34;successful&#34;:5,&#34;skipped&#34;:0,&#34;failed&#34;:0},&#34;hits&#34;:{&#34;total&#34;:1,&#34;max_score&#34;:1.0,&#34;hits&#34;:[{&#34;_index&#34;:&#34;store&#34;,&#34;_type&#34;:&#34;books&#34;,&#34;_id&#34;:&#34;1&#34;,&#34;_score&#34;:1.0,&#34;_source&#34;:{
  &#34;title&#34;: &#34;Elasticsearch :The Definitive Guide&#34;,
  &#34;name&#34;: {
    &#34;first&#34;:&#34;li &#34;,
    &#34;last&#34;: &#34;si &#34;
  },
  &#34;publish_date&#34;:&#34;2015-02-06&#34;,
  &#34;price&#34;:&#34;39.99&#34;
}}]}}
```

指定多个值



```
curl http://172.18.3.106:9200/store/books/_search -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;query&#34;: {
        &#34;constant_score&#34;: {
	        &#34;filter&#34;: {
                &#34;terms&#34; : {
                    &#34;price&#34;:[39.99,99]
                }
            }
        }
    }
}&#39;
```

range 范围过滤



- gt &gt; 大于
- lt &lt; 小于
- gte &gt;= 大于等于
- lte &lt;= 小于等于

```
curl http://172.18.3.106:9200/store/books/_search -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;query&#34;: {
        &#34;range&#34; : {
            &#34;price&#34;:{
                &#34;gte&#34; :10,
                &#34;lt&#34; : 90
            }
        }
    }
}&#39;
```

如果没有定义类型，比对的是字符串。



&gt; 例：name和athor都必须包含Guide，并且价钱等于33.99或者188.99



```
curl http://172.18.3.106:9200/store/books/_search -H &#34;Content-type: application/json&#34; -d &#39;{
    &#34;query&#34;: {
        &#34;bool&#34;: {
            &#34;must&#34;: {
                &#34;multi_match&#34; : {
                    &#34;operator&#34;: &#34;and&#34;,
                    &#34;fields&#34;: [
                        &#34;name&#34;,
                        &#34;author&#34;
                    ],
                    &#34;query&#34;: &#34;Guide&#34;
                }
            },
            &#34;filter&#34;:{
                &#34;terms&#34; : {
                    &#34;price&#34;: [
                        35.99,
                        188.99
                    ]
                }
            }
        }
    }
}&#39;
```





查看setting

```
[root@node03 ~]# curl -XGET http://172.18.3.106:9200/news/_settings?pretty
{
  &#34;news&#34; : {
    &#34;settings&#34; : {
      &#34;index&#34; : {
        &#34;creation_date&#34; : &#34;1580568630458&#34;,
        &#34;number_of_shards&#34; : &#34;5&#34;,
        &#34;number_of_replicas&#34; : &#34;1&#34;,
        &#34;uuid&#34; : &#34;BLnOUQT5QPqBjCmEcJGoUA&#34;,
        &#34;version&#34; : {
          &#34;created&#34; : &#34;6080699&#34;
        },
        &#34;provided_name&#34; : &#34;news&#34;
      }
    }
  }
}
```

创建索引时设置setting

```
curl -XPUT  http://172.18.3.106:9200/store/ -H &#39;Content-Type:application/json&#39; -d  &#39;{
    &#34;settings&#34;: {
        &#34;number_of_shards&#34;: 3,
        &#34;number_of_replicas&#34;: 2
    }
}&#39;
```

动态修改已经存在索引的setting

---

**注**：此处只能修改副本，分片一旦创建就无法修改

---

```
curl -H &#39;Content-Type:application/json&#39; -XPUT  http://172.18.3.106:9200/store1/_settings  -d  &#39;{
    &#34;index&#34;: {
        &#34;number_of_replicas&#34;: 1
    }
}&#39;
```



es核心概念



mapping


