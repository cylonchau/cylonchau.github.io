# ELK 收集java日志


Java日志多由多行行组成，初始行之后的每一行以空格开头，如下例所示：在这种情况下，需要在将事件数据output之前处理多行事件。

```log
[ERROR] - 2018-09-21 04:19:57.685 (SocketTransfer.java:73) - Socket交互遇到错误
   Exception in thread "main" java.lang.NullPointerException
   at com.example.myproject.Book.getTitle(Book.java:16)
   at com.example.myproject.Author.getBookTitles(Author.java:25)
   at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
```
配置multiline参数如下：[Multiline codec plugin](https://www.elastic.co/guide/en/logstash/6.4/plugins-codecs-multiline.html)



| 参数    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| pattern | 指定正则表达式。与指定正则表达式匹配的行被视为前一行的延续或新多行事件的开始。 |
| what    | previous或next。该previous值指定与pattern选项中的值匹配的行是上一行的一部分。该next值指定与pattern选项中的值匹配的行是以下行的一部分 |
| negate  | true或false（默认为false）。如果true，与pattern模式不匹配的消息将构成多行过滤器的匹配what并将应用。 |


```sh
input {
		file {
				path => "/root/apache-tomcat-8.5.34/logs/catalina.out"
				type => "sk"
				start_position => "beginning"
				codec => multiline {
						pattern => "^\["
						negate => "true"
						what => "previous"
				}
		}
}
```

如上列日志格式为

```sh
[日志级别] - 日期 java类 - 错误报错
```

一般情况下，我们不需要将对其grok，因grok会大量影响性能。如上列格式需要对每一列进行grok

```sh
# 日期 
TESTTIME %{YEAR}\-%{MONTHNUM}\-%{MONTHDAY} %{HOUR}:%{MINUTE}:%{SECOND}
```
根据以上格式写出如下的正则
```logstash
filter {
  grok {
    match => {
      'message' => "\[\s?%{LOGLEVEL:lev}\] - %{TESTTIME:date}\s+\((?<\class>.*\.java\:\d+)\)\s\-\s(?<\msg>.*)"
    }
  
  }
  
  mutate {  
    remove_field => "message"
  }
}
```

匹配到的日志格式

```sh
{
    "@timestamp" => "2018-10-05T13:16:27.263Z",
      "@version" => "1",
          "path" => "/root/apache-tomcat-8.5.34/logs/catalina.out",
          "host" => "node02",
          "type" => "sk",
           "lev" => "DEBUG",
          "date" => "2018-10-04 12:48:56.142",
         "class" => "ObjectDaoImpl.java:219",
           "msg" => "queryPage with hql:select t.fplxdm,t.fpdm,t.fphm,t.fpcbh,f.ipdz,f.dkh,f.jqbh from ( select '026' as fplxdm,fpdm,fphm,kprq,fpcbh,jqbh from pj_zzspdz_fpmx where kprq>'20180801000000' and jqbh='499099915361' and qmbz='N') t,dj_fwqxx f where t.jqbh=f.jqbh order by t.kprq"
}
```
