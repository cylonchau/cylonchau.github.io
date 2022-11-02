# ELK 收集java日志


Java日志多由多行行组成，初始行之后的每一行以空格开头，如下例所示：在这种情况下，需要在将事件数据output之前处理多行事件。

```log
[ERROR] - 2018-09-21 04:19:57.685 (SocketTransfer.java:73) - Socket交互遇到错误
   Exception in thread &#34;main&#34; java.lang.NullPointerException
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
				path =&gt; &#34;/root/apache-tomcat-8.5.34/logs/catalina.out&#34;
				type =&gt; &#34;sk&#34;
				start_position =&gt; &#34;beginning&#34;
				codec =&gt; multiline {
						pattern =&gt; &#34;^\[&#34;
						negate =&gt; &#34;true&#34;
						what =&gt; &#34;previous&#34;
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
    match =&gt; {
      &#39;message&#39; =&gt; &#34;\[\s?%{LOGLEVEL:lev}\] - %{TESTTIME:date}\s&#43;\((?&lt;\class&gt;.*\.java\:\d&#43;)\)\s\-\s(?&lt;\msg&gt;.*)&#34;
    }
  
  }
  
  mutate {  
    remove_field =&gt; &#34;message&#34;
  }
}
```

匹配到的日志格式

```sh
{
    &#34;@timestamp&#34; =&gt; &#34;2018-10-05T13:16:27.263Z&#34;,
      &#34;@version&#34; =&gt; &#34;1&#34;,
          &#34;path&#34; =&gt; &#34;/root/apache-tomcat-8.5.34/logs/catalina.out&#34;,
          &#34;host&#34; =&gt; &#34;node02&#34;,
          &#34;type&#34; =&gt; &#34;sk&#34;,
           &#34;lev&#34; =&gt; &#34;DEBUG&#34;,
          &#34;date&#34; =&gt; &#34;2018-10-04 12:48:56.142&#34;,
         &#34;class&#34; =&gt; &#34;ObjectDaoImpl.java:219&#34;,
           &#34;msg&#34; =&gt; &#34;queryPage with hql:select t.fplxdm,t.fpdm,t.fphm,t.fpcbh,f.ipdz,f.dkh,f.jqbh from ( select &#39;026&#39; as fplxdm,fpdm,fphm,kprq,fpcbh,jqbh from pj_zzspdz_fpmx where kprq&gt;&#39;20180801000000&#39; and jqbh=&#39;499099915361&#39; and qmbz=&#39;N&#39;) t,dj_fwqxx f where t.jqbh=f.jqbh order by t.kprq&#34;
}
```
