# 

https://www.cnblogs.com/leeSmall/p/9220535.html

https://blog.csdn.net/zhibiyus/article/details/107907450

https://www.cnblogs.com/kebibuluan/p/13750293.html

[toc]





##  索引管理



### 创建索引

直接创建索引 `PUT newindex1`，创建索引可以通过 `number_of_shares` 和  `number_of_replicas` 数量来修饰分片和副本的数量。

```
PUT newindex
{
  &#34;settings&#34;: {
    &#34;index&#34; : {
      &#34;number_of_shares&#34; : 2,
      &#34;number_of_replicas&#34;: 1
    }
  }
}
```

`number_of_shares` 分片数在创建索引后不能修改

`number_of_replicas`  副本数可以随时完成修改

### 删除索引

`DEL index_name`

### 打开/关闭索引

`POST {index_name}/_close`

`POST {index_name}/_open`

关闭的索引无法进行【增删改查】操作

```
{
  &#34;error&#34; : {
    &#34;root_cause&#34; : [
      {
        &#34;type&#34; : &#34;index_closed_exception&#34;,
        &#34;reason&#34; : &#34;closed&#34;,
        &#34;index_uuid&#34; : &#34;3eCslZZ3Q9amlUyDtqTXWA&#34;,
        &#34;index&#34; : &#34;newindex&#34;
      }
    ],
    &#34;type&#34; : &#34;index_closed_exception&#34;,
    &#34;reason&#34; : &#34;closed&#34;,
    &#34;index_uuid&#34; : &#34;3eCslZZ3Q9amlUyDtqTXWA&#34;,
    &#34;index&#34; : &#34;newindex&#34;
  },
  &#34;status&#34; : 400
}
```

### 索引的映射 mapping

mapping是定义文档及包含字段的存储与索引方式。可以理解为是elasticsearch的表结构，定义mapping，即在创建index时，自行判断每个字段的类型，而不是有ES自动自动判断每个纬度的类型。这种更贴合业务场景，如分词、存储。

每个索引仅有一个映射类型（elasticsearch6.x&#43;，之前版本一个索引下有多个类型），它决定了文档将如何被索引。而映射类型分为两部分 `meta-fields` 与 `field of properties`

 `meta-fields` ：为文档的源数据，如 `_index` （索引的名称）、`_type` （文档的类型，7.0&#43;弃用）、`_id` （索引的ID）和 `_source`（用于关联文档源数据）字段

`field of properties`：字段属性，包含文档的字段或属性列表。

#### 字段的数据类型

##### 常见类型

- `binary`：二进制或Base64字符串。
- `boolean`: 布尔值`true`和`false`。
- `keywords`：`keyword`, `constant_keyword`, 和 `wildcard`.
- `Numbers`：数值类型，例如`long`和`double`
- `dates`：日期类型，`date` 和 `date_nanos`。
- `alias`： 为以有字段定义别名。

##### 对象嵌套类型

- `object` ：JSON对象。
- `flattened`：整个JSON对象作为单个字段值。
- `nested`：嵌套，与子字段之间保留关系的json对象。
- `join`：为同一索引中的文档定义父/子关系。

##### 结构化类型

- `range`：，`long_range`，`double_range`， `date_range`，和`ip_range`。
- `ip`：IPv4和IPv6地址。
- `version` ：软件版本号。支持 [语义化版本号](https://semver.org/) 优先规则。
- `murmur3`：计算并存储值的散列。

##### 汇总数据类型

- `aggregate_metric_double`：预汇总的指标值。
- `histogram`：以直方图形式预汇总的数值。

##### 文字搜寻类型

- `text`：分析的非结构化文本。
- `annotated-text`：包含特殊标记的文本。用于标识命名实体。
- `completion`：用于自动补全建议。
- `search_as_you_type` `text`类似类型，用于按需输入完成。
- `token_count`：计数令牌。

等等 [reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html#object-types)。

#### 常用字段数据类型参数

更多参数可以参考官方文档：[mapping-params](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

| 字段           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| analyzer       | 仅`text`字段支持 `analyzer` 映射参数。                       |
| index          | 选项控制是否对字段值建立索引。默认为`true`。未索引的字段不可查询。 |
| index_prefixes | 启用术语前缀的索引，以加快前缀搜索的速度                     |
| ignore_above   | 长度大于 ignore_above设置的字符串将不会被索引或存储          |

#### 映射的元字段

文档的元字段是保证系统正常运转的内置字段，如 `_index` 索引的字段  `_type`映射类型（7.0后取消），`_id_` 文档主键。

#### 动态映射 dynamic

在关系型数据库中，需要事先创建好数据库，并在库中插入表。而ES中需要事先创建好索引结构（Mapping），在插入文档到索引中，系统会根据文档内容进行索引的动态映射。自动检测添加新类型和字段，被成为动态映射。

禁用动态映射：`{index_name}/_mapping `  `dynamic` 为false

```
POST student/_mapping
{
  &#34;dynamic&#34;:&#34;false&#34;
}

GET student/_mappings
```

#### 显式映射

实现准备好映射关系，包含文档各字段类型，关系等，这种称之为显式映射 `Explicit mapping`

#### 动态模板

动态映射会自动推断数据类型，但这种并不完全符合所有的业务需求，而动态模板可以再动态映射之外更好的控制ES如何映射的数据类型。

- [`match_mapping_type`](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html#match-mapping-type) 对 Elasticsearch 检测到的数据类型进行操作
- `match`   `unmatch `  ： `match`使用pattern匹配字段名称，`unmatch` 使用正则排除字段
- `path_match`   `path_unmatch` 使用与match一样，这里为全名称，path指的是多层json的路径。

如，需要将所有数字字段映射为integer而不是long，将所有字符串都映射为text与keyword:

```
{
  &#34;mappings&#34;: {
    &#34;dynamic_templates&#34;: [
      {
        &#34;integers&#34;: {
          &#34;match_mapping_type&#34;: &#34;long&#34;,
          &#34;mapping&#34;: {
            &#34;type&#34;: &#34;integer&#34;
          }
        }
      },
      {
        &#34;strings&#34;: {
          &#34;match_mapping_type&#34;: &#34;string&#34;,
          &#34;mapping&#34;: {
            &#34;type&#34;: &#34;text&#34;,
            &#34;fields&#34;: {
              &#34;raw&#34;: {
                &#34;type&#34;:  &#34;keyword&#34;,
                &#34;ignore_above&#34;: 256
              }
            }
          }
        }
      }
    ]
  }
}
```

可以使用正则表达式来匹配字段

```
PUT student12345
{
  &#34;mappings&#34;: {
    &#34;dynamic_templates&#34;: [
      {
        &#34;longs_as_strings&#34;: {
          &#34;match_mapping_type&#34;: &#34;string&#34;,  // 捕获的类型
          &#34;match&#34;:   &#34;pass*&#34;, // 将以pass开头的string类型的映射为integer
          &#34;unmatch&#34;: &#34;user*&#34;, // 忽略以user开头的
          &#34;mapping&#34;: {
            &#34;type&#34;: &#34;integer&#34;  // 需要映射的类型
          }
        }
      }
    ]
  }
}

PUT student12345/_doc/1
{
  &#34;username&#34;: &#34;zhangsan&#34;, 
  &#34;password&#34;: &#34;123456&#34; 
}
```

查看映射后的类型

```
{
  &#34;student12345&#34; : {
    &#34;mappings&#34; : {
      &#34;dynamic_templates&#34; : [
        {
          &#34;longs_as_strings&#34; : {
            &#34;match&#34; : &#34;pass*&#34;,
            &#34;unmatch&#34; : &#34;user*&#34;,
            &#34;match_mapping_type&#34; : &#34;string&#34;,
            &#34;mapping&#34; : {
              &#34;type&#34; : &#34;integer&#34;
            }
          }
        }
      ],
      &#34;properties&#34; : {
        &#34;password&#34; : {
          &#34;type&#34; : &#34;integer&#34;
        },
        &#34;username&#34; : {
          &#34;type&#34; : &#34;text&#34;,
          &#34;fields&#34; : {
            &#34;keyword&#34; : {
              &#34;type&#34; : &#34;keyword&#34;,
              &#34;ignore_above&#34; : 256
            }
          }
        }
      }
    }
  }
}
```

`path_match`   `path_unmatch` 

```
PUT test1
{
  &#34;mappings&#34;: {
    &#34;dynamic_templates&#34;: [
      {
        &#34;full_name&#34;: { 
          &#34;path_match&#34;:   &#34;document.*&#34;, // 将name下的所有对象复制到外部顶级字段
          &#34;path_unmatch&#34;: &#34;*.middlename&#34;, // 这个字段除外
          &#34;mapping&#34;: {
            &#34;type&#34;:       &#34;text&#34;,
            &#34;copy_to&#34;:    &#34;full_name&#34;
          }
        }
      }
    ]
  }
}

PUT test1/_doc/1
{
  &#34;document&#34;: {
    &#34;firstname&#34;:  &#34;John&#34;,
    &#34;middlename&#34;: &#34;Winston&#34;,
    &#34;lastname&#34;:   &#34;Lennon&#34;
  }
}
## 当插入下列时，不成功，这里映射类型为text，与address中的对象类型不匹配所以不成功

PUT test1/_doc/1
{
  &#34;document&#34;: {
    &#34;firstname&#34;:  &#34;John&#34;,
    &#34;middlename&#34;: &#34;Winston&#34;,
    &#34;address&#34;: {
    	&#34;city&#34;: &#34;beijing&#34;,
    	&#34;province&#34;: &#34;beijing&#34;,
 			&#34;District&#34;: &#34;haidian&#34;
    }
  }
}
```





##### 模板变量

{name} {dynamic_type} 占位符，在映射中会替换为字段名和检测到的动态类型

```json
PUT student1
{
  &#34;mappings&#34;: {
    &#34;dynamic_templates&#34;: [
      {
        &#34;named_analyzers&#34;: {
          &#34;match_mapping_type&#34;: &#34;string&#34;,
          &#34;match&#34;: &#34;*&#34;,
          &#34;mapping&#34;: {
            &#34;type&#34;: &#34;text&#34;,
            &#34;analyzer&#34;: &#34;{name}&#34;
          }
        }
      },
      {
        &#34;no_doc_values&#34;: {
          &#34;match_mapping_type&#34;:&#34;*&#34;,
          &#34;mapping&#34;: {
            &#34;type&#34;: &#34;{dynamic_type}&#34;,
            &#34;doc_values&#34;: false
          }
        }
      }
    ]
  }
}

PUT student1/_doc/1
{
  &#34;name&#34;: &#34;alex chow&#34;, 
  &#34;age&#34;: 30
}
  PUT student1/_doc/1
  {
    &#34;english&#34;: &#34;Some English text&#34;, 
    &#34;count&#34;:   5 
  }
```



**reference**  [template-variables](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html#template-variables)



#### 索引的类型

Elasticsearch7.x中，不在需要document的 type

[schedule_for_removal_of_mapping_types](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html#_schedule_for_removal_of_mapping_types)

ElasticSearch在7.x版本之前 `index` 类似于数据库中的 `database`，而 `type` 等同于数据库中的 `table`

如：6.x API

```
PUT student # index（库）
{
  &#34;mappings&#34;: {
    &#34;_doc&#34;: { # type （表）
      &#34;properties&#34;: {
        &#34;type&#34;: { # filed (字段)
          &#34;type&#34;: &#34;keyword&#34;
        },
        &#34;name&#34;: {
          &#34;type&#34;: &#34;text&#34;
        },
        &#34;user_name&#34;: {
          &#34;type&#34;: &#34;keyword&#34;
        },
        &#34;email&#34;: {
          &#34;type&#34;: &#34;keyword&#34;
        },
        &#34;content&#34;: {
          &#34;type&#34;: &#34;text&#34;
        },
        &#34;tweeted_at&#34;: {
          &#34;type&#34;: &#34;date&#34;
        }
      }
    }
  }
}
```

而在7.x之后版本可以不需要

```

PUT student
{
  &#34;mappings&#34;: {
      &#34;properties&#34;: {
        &#34;type&#34;: {
          &#34;type&#34;: &#34;keyword&#34;
        },
        &#34;name&#34;: {
          &#34;type&#34;: &#34;text&#34;
        },
        &#34;user_name&#34;: {
          &#34;type&#34;: &#34;keyword&#34;
        },
        &#34;email&#34;: {
          &#34;type&#34;: &#34;keyword&#34;
        },
        &#34;content&#34;: {
          &#34;type&#34;: &#34;text&#34;
        },
        &#34;tweeted_at&#34;: {
          &#34;type&#34;: &#34;date&#34;
        }
      }
    }
}
```

在7.0以上版本使用时，必须使用 `/{index_name}/_doc/{_id}` 进行调用。

`_doc` 为路由永久组成，可以理解为 `_doc` 替换之前的 `type`

```
PUT /student/_doc/1
{
  &#34;name&#34;: &#34;zhangsan&#34;,
  &#34;user_name&#34;: &#34;zhangsan&#34;,
  &#34;email&#34;: &#34;1@gmail.com&#34;,
  &#34;content&#34;: &#34;北京分行、天津分行、河北分行、山西分行、辽宁分行&#34;
}

GET /student/_doc/1
```

对于7.0之前的`/{index}/{type}/{action}` ，的操作如`_update` 、`_search` 将紧跟`{action}`后面。

```
POST /student/_update/1
{
  &#34;doc&#34;: {
    &#34;user_name&#34;: &#34;lisi&#34;
  }
}

GET /student/_search
```

















查询doc的version，`_search?version=true` 返回结果中就会返回 version 。

```
GET doc_detail/_search?version=true
```

外部版本控制

外部版本控制：elasticsearch在处理外部版本号时会对内部版本号的处理有些不同。他不在是检查version是否与请求中指定的数值相同，而是检查当前的_version是否比指定的数值小。如果请求成功，那么外部的版本号就会被存储到文档中的_version中。

```
index/type/4?version=10&amp;version_type=external
```



插入数据

```
PUT index/type/id2
{
  &#34;name&#34;: &#34;谭嗣同&#34;
  
}
```



```
GET _search
{
  &#34;query&#34;: {
    &#34;match_all&#34;: {}
  }
}

PUT index/type/id2
{
  &#34;name&#34;: &#34;谭嗣同&#34;
  
}

PUT index/type/1
{
  &#34;name&#34;: &#34;王迪&#34;,
  &#34;age&#34;: 20,
  &#34;tag&#34;: &#34;闷骚&#34;,
  &#34;brithday&#34;: 19970521
}

PUT index/type/2
{
  &#34;name&#34;: &#34;路晨&#34;,
  &#34;age&#34;: 31,
  &#34;tag&#34;: &#34;美&#34;,
  &#34;brithday&#34;: 19900521
}

PUT index/type/3
{
  &#34;name&#34;: &#34;金沙&#34;,
  &#34;age&#34;: 25,
  &#34;tag&#34;: &#34;没烧&#34;,
  &#34;brithday&#34;: 198606521
}


PUT index/type/4?version=10&amp;version_type=external
{
  &#34;name&#34;: &#34;唐艳&#34;,
  &#34;age&#34;: 25,
  &#34;tag&#34;: &#34;骚货&#34;,
  &#34;brithday&#34;: 19960101
}


GET index/type/_search?version=2


POST index/type/4/_update
{
  &#34;doc&#34;: {
    &#34;tag&#34;: &#34;candy&#34;
  }
}

# 更新/插入
PUT index/type/3
{
  &#34;name&#34;: &#34;韦小宝&#34;
  
}
# 获取
GET index/type/1

GET index/type/_search

GET index/type/_search

## 获取全部

GET index/type/_search

DELETE index/type/id
DELETE index

POST index/type/_delete_by_query?q=name:谭嗣同
# 结构化查询

GET index/type/_search
{
  &#34;query&#34;: {
    &#34;match&#34;: {
      &#34;name&#34;: &#34;唐艳&#34;
    }
  }
}

GET index/type/_search
{
  &#34;query&#34;: {
    &#34;match&#34;: {
      &#34;tag&#34;: &#34;烧,骚&#34;
    }
  }
}

GET index/type/_search
{
  &#34;query&#34;: {
    &#34;match_all&#34;: {}
  },
  &#34;sort&#34;: {
    &#34;age&#34;: {
      &#34;order&#34;:&#34;desc&#34;
    }
  }
}

## 分页

GET index/type/_search
{
  &#34;query&#34;: {
    &#34;match_all&#34;: {}
  },
  &#34;sort&#34;: {
    &#34;_id&#34;: {
      &#34;order&#34;:&#34;desc&#34;
    }
  },
  &#34;from&#34;: 1,
  &#34;size&#34;:2
}

### 布尔值查询

GET index/type/_search
{
  &#34;query&#34;: {
    &#34;bool&#34;: {
      &#34;should&#34;: [
        {
          &#34;match&#34;: {
            &#34;name&#34;: &#34;唐嫣&#34;
          }
        },
        {
          &#34;match&#34;: {
            &#34;tag&#34;: &#34;美&#34;
          }
        }
      ]
    }
  }
}

## 查看类型
GET index/_mapping
## 查看配置
GET index/_settings

GET index

GET bid/BID/1

PUT bid/BID/1
{
	&#34;latestExecInfo&#34;: &#34;&#34;,
	&#34;uid&#34;: &#34;BID_test-all_heimdall-acquirer_97809-CREATE&#34;,
	&#34;locale&#34;: &#34;zh-CN&#34;,
	&#34;strategyDescription&#34;: [{
		&#34;task_result&#34;: {
			&#34;detail&#34;: [{
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:instance-94sr85v8-24.bcc-bjdd:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;instance-94sr85v8-24.bcc-bjdd&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:k8s-monitor01:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;k8s-monitor01&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:k8s-monitor02:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;k8s-monitor02&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:k8s-monitor04:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;k8s-monitor04&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:test-all-Y32-3.BD-HB.bcc-bdbl:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;test-all-Y32-3.BD-HB.bcc-bdbl&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:test-all-Y32-1.BD-HB.bcc-bdbl:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;test-all-Y32-1.BD-HB.bcc-bdbl&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:test-all-Y32-2.BD-HB.bcc-bdbl:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;test-all-Y32-2.BD-HB.bcc-bdbl&#34;
			}, {
				&#34;taskId&#34;: &#34;HOST:30fa7bc9-255a-437a-9ae7-81f342c15b41:instance-6534xfcz-44.bcc-bjdd:1&#34;,
				&#34;status&#34;: &#34;FINISHED&#34;,
				&#34;exitcode&#34;: 0,
				&#34;target&#34;: &#34;instance-6534xfcz-44.bcc-bjdd&#34;
			}],
			&#34;errno&#34;: 5000000,
			&#34;info&#34;: &#34;success.&#34;
		},
		&#34;status&#34;: &#34;FINISHED&#34;,
		&#34;type&#34;: &#34;deploy_task&#34;,
		&#34;stage_id&#34;: &#34;BID_test-all_heimdall-acquirer_97809_stage_1&#34;,
		&#34;deploy_params&#34;: {
			&#34;target&#34;: [&#34;k8s-monitor01&#34;, &#34;k8s-monitor04&#34;, &#34;test-all-Y32-1.BD-HB.bcc-bdbl&#34;, &#34;test-all-Y32-2.BD-HB.bcc-bdbl&#34;, &#34;test-all-Y32-3.BD-HB.bcc-bdbl&#34;, &#34;instance-94sr85v8-24.bcc-bjdd&#34;, &#34;instance-6534xfcz-44.bcc-bjdd&#34;, &#34;k8s-monitor02&#34;],
			&#34;concurrency&#34;: 3,
			&#34;failure_tolerance&#34;: 0
		}
	}, {
		&#34;check_params&#34;: {
			&#34;check_type&#34;: &#34;manual&#34;,
			&#34;check_url&#34;: &#34;&#34;
		},
		&#34;task_result&#34;: {
			&#34;detail&#34;: {},
			&#34;errno&#34;: 0,
			&#34;info&#34;: &#34;the last stage is automatically set to finished&#34;
		},
		&#34;status&#34;: &#34;FINISHED&#34;,
		&#34;type&#34;: &#34;check_task&#34;,
		&#34;stage_id&#34;: &#34;BID_test-all_heimdall-acquirer_97809_stage_2&#34;
	}],
	&#34;cancelCause&#34;: &#34;&#34;,
	&#34;couldRollback&#34;: true,
	&#34;jobStatus&#34;: &#34;FINISHED&#34;,
	&#34;type&#34;: &#34;BID&#34;,
	&#34;product&#34;: {
		&#34;type&#34;: &#34;product&#34;,
		&#34;name&#34;: &#34;test-all&#34;,
		&#34;refer&#34;: &#34;&#34;
	},
	&#34;hostnameDict&#34;: {
		&#34;k8s-monitor01&#34;: &#34;0.heimdall-acquirer.test-all&#34;,
		&#34;test-all-Y32-2.BD-HB.bcc-bdbl&#34;: &#34;5.heimdall-acquirer.test-all&#34;,
		&#34;k8s-monitor02&#34;: &#34;1.heimdall-acquirer.test-all&#34;,
		&#34;instance-6534xfcz-44.bcc-bjdd&#34;: &#34;8.heimdall-acquirer.test-all&#34;,
		&#34;instance-94sr85v8-24.bcc-bjdd&#34;: &#34;7.heimdall-acquirer.test-all&#34;,
		&#34;test-all-Y32-1.BD-HB.bcc-bdbl&#34;: &#34;4.heimdall-acquirer.test-all&#34;,
		&#34;k8s-monitor04&#34;: &#34;3.heimdall-acquirer.test-all&#34;,
		&#34;test-all-Y32-3.BD-HB.bcc-bdbl&#34;: &#34;6.heimdall-acquirer.test-all&#34;
	},
	&#34;jobContent&#34;: {
		&#34;source_info&#34;: {
			&#34;module_type&#34;: &#34;ARCHER&#34;,
			&#34;module_source&#34;: &#34;http://noahproxy.duxiaoman-int.com:80/proxy/download/dxm/dxmqa-monitor/heimdall/1.0.0.1034/&#34;,
			&#34;pre_cmd&#34;: &#34;sh /home/work/opScripts/op_before_heimdall_acquirer.sh&#34;,
			&#34;limit_rate&#34;: 10,
			&#34;module_id&#34;: 1,
			&#34;deploy_path&#34;: &#34;/home/work/heimdall/heimdall-acquirer&#34;,
			&#34;post_cmd&#34;: &#34;sh /home/work/opScripts/op_after_heimdall_acquirer.sh&#34;,
			&#34;MD5&#34;: &#34;&#34;
		},
		&#34;cmd_info&#34;: {
			&#34;exec_path&#34;: &#34;&#34;,
			&#34;cmd&#34;: &#34;&#34;
		}
	},
	&#34;timeZone&#34;: &#34;UTC&#43;08:00&#34;,
	&#34;operateTime&#34;: 0,
	&#34;reviewInfo&#34;: {
		&#34;reviewer&#34;: &#34;caiqinwei_dxm&#34;,
		&#34;reviewTime&#34;: 1621218756
	},
	&#34;startTime&#34;: 1621218751,
	&#34;impactedOpObject&#34;: [{
		&#34;type&#34;: &#34;app&#34;,
		&#34;name&#34;: &#34;heimdall-acquirer&#34;,
		&#34;refer&#34;: &#34;&#34;
	}],
	&#34;opObject&#34;: [],
	&#34;lastInterveneId&#34;: &#34;&#34;,
	&#34;intervene&#34;: &#34;None&#34;,
	&#34;jobDescription&#34;: {
		&#34;app_path&#34;: &#34;/home/work/heimdall&#34;,
		&#34;subsystem_id&#34;: &#34;monitor&#34;,
		&#34;template_name&#34;: &#34;heimdall-acquirer&#34;,
		&#34;derive_tag&#34;: &#34;&#34;,
		&#34;job_type&#34;: &#34;CREATE&#34;,
		&#34;app_id&#34;: &#34;heimdall-acquirer&#34;,
		&#34;alarm_receiver&#34;: &#34;caiqinwei_dxm&#34;,
		&#34;node_info&#34;: &#34;测试公共产品线-monitor-heimdall-acquirer&#34;,
		&#34;is_derive&#34;: &#34;False&#34;,
		&#34;machine_timeout&#34;: 600,
		&#34;module_version&#34;: &#34;1.0.0.1034&#34;,
		&#34;job_id&#34;: 97809,
		&#34;email_recv_person&#34;: &#34;&#34;,
		&#34;is_public_cmd&#34;: false,
		&#34;creator&#34;: &#34;caiqinwei_dxm&#34;,
		&#34;close_alarm&#34;: false,
		&#34;is_skip_check&#34;: false,
		&#34;productId&#34;: &#34;test-all&#34;,
		&#34;email_receivers&#34;: [{
			&#34;receiverAlias&#34;: &#34;&#34;,
			&#34;receiverType&#34;: &#34;person&#34;,
			&#34;receiverName&#34;: &#34;caiqinwei_dxm&#34;
		}],
		&#34;relative_path&#34;: &#34;heimdall-acquirer&#34;,
		&#34;failure_tolerance&#34;: 0,
		&#34;user&#34;: &#34;work&#34;,
		&#34;email_recv_group&#34;: &#34;&#34;,
		&#34;name&#34;: &#34;caiqinwei_dxm操作dxmqa-monitor/heimdall1个模块&#34;,
		&#34;sms_recv_person&#34;: &#34;&#34;,
		&#34;package_type&#34;: &#34;manager&#34;,
		&#34;sms_recv_group&#34;: &#34;&#34;,
		&#34;be_rollbacked_id&#34;: null,
		&#34;alias&#34;: &#34;&#34;,
		&#34;module_name&#34;: &#34;dxmqa-monitor/heimdall&#34;,
		&#34;sms_receivers&#34;: [{
			&#34;receiverAlias&#34;: &#34;&#34;,
			&#34;receiverType&#34;: &#34;person&#34;,
			&#34;receiverName&#34;: &#34;caiqinwei_dxm&#34;
		}]
	},
	&#34;operation_id&#34;: &#34;BID_test-all_heimdall-acquirer_97809&#34;,
	&#34;endTime&#34;: 0
}




```


