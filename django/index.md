# 

[toc]

## 路由匹配

### django中默认匹配页

```
url(r&#39;^$&#39;, views.login),
```

### django中404匹配

```
url(r&#39;^$&#39;, views.login), # 需要放置最后，不过一般不推荐，都是通过异常捕获处理
```

### named group 名称组

https://docs.djangoproject.com/en/1.11/topics/http/urls/#named-groups

````
url(r&#39;^test[0-9]{4}&#39;,views.login) 
````



## 反向解析

&gt; 别名不能出现冲突

```
from django.shortcuts import reverse
reverse(xxx)
```

### 名称组反向解析

无名分组

```python
# 路由部分
url(r&#39;^index/(\d&#43;)/&#39;, views.home, name=&#39;xxx&#39;) 
# 前端
 &lt;a href=&#34;{% url &#39;id&#39; obj.id %}&#34; class=&#34;btn btn-primary btn-xs&#34;&gt;remove&lt;/a&gt;s
# 后端
print reverse(&#39;id&#39;, args=(id,))
```

有名称分组

```python
# 路由部分
  url(r&#34;^userdel/(?P&lt;id&gt;\d&#43;)/&#34;,views.UserDelete, name=&#39;id&#39;),
# 前端
  &lt;a href=&#34;{% url &#39;id&#39; obj.id %}&#34; class=&#34;btn btn-primary btn-xs&#34;&gt;remove&lt;/a&gt;
# or
  &lt;a href=&#34;{% url &#39;id&#39; id=obj.id %}&#34; class=&#34;btn btn-primary btn-xs&#34;&gt;remove&lt;/a&gt;s
# 后端
  print reverse(&#39;id&#39;, kwargs={&#34;id&#34;:id})
```

## 路由分发

路由分发中，并不能识别出，名称分组并不能准确识别出对应的分组，这里需要增加namespace概念

```python
from django.conf.urls import url, include
from django.contrib import admin
from memberserver import urls as member_urls

urlpatterns = [
    url(r&#39;member&#39;, include(member_urls)),
]
```

路由分发中，并不能识别出，名称分组并不能准确识别出对应的分组，这里需要增加namespace概念

```python
from django.conf.urls import url, include
from django.contrib import admin
from memberserver import urls as member_urls

urlpatterns = [
    url(r&#39;member&#39;, include(member_urls, namespace=&#34;member&#34;)),
]
# 在后端映射可以使用
reverse(&#34;member:id&#34;) ## 来获得对应的路由

# 在前端可以使用 来获得对应的路由
{% url &#39;id&#39; id=obj.id %}
```

伪静态

虚拟环境

