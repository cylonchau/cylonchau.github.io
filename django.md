# 

[toc]

## 路由匹配

### django中默认匹配页

```
url(r'^$', views.login),
```

### django中404匹配

```
url(r'^$', views.login), # 需要放置最后，不过一般不推荐，都是通过异常捕获处理
```

### named group 名称组

https://docs.djangoproject.com/en/1.11/topics/http/urls/#named-groups

````
url(r'^test[0-9]{4}',views.login) 
````



## 反向解析

> 别名不能出现冲突

```
from django.shortcuts import reverse
reverse(xxx)
```

### 名称组反向解析

无名分组

```python
# 路由部分
url(r'^index/(\d+)/', views.home, name='xxx') 
# 前端
 <a href="{% url 'id' obj.id %}" class="btn btn-primary btn-xs">remove</a>s
# 后端
print reverse('id', args=(id,))
```

有名称分组

```python
# 路由部分
  url(r"^userdel/(?P<id>\d+)/",views.UserDelete, name='id'),
# 前端
  <a href="{% url 'id' obj.id %}" class="btn btn-primary btn-xs">remove</a>
# or
  <a href="{% url 'id' id=obj.id %}" class="btn btn-primary btn-xs">remove</a>s
# 后端
  print reverse('id', kwargs={"id":id})
```

## 路由分发

路由分发中，并不能识别出，名称分组并不能准确识别出对应的分组，这里需要增加namespace概念

```python
from django.conf.urls import url, include
from django.contrib import admin
from memberserver import urls as member_urls

urlpatterns = [
    url(r'member', include(member_urls)),
]
```

路由分发中，并不能识别出，名称分组并不能准确识别出对应的分组，这里需要增加namespace概念

```python
from django.conf.urls import url, include
from django.contrib import admin
from memberserver import urls as member_urls

urlpatterns = [
    url(r'member', include(member_urls, namespace="member")),
]
# 在后端映射可以使用
reverse("member:id") ## 来获得对应的路由

# 在前端可以使用 来获得对应的路由
{% url 'id' id=obj.id %}
```

伪静态

虚拟环境

