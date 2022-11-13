# python django-restframework view set




## What is Views

drf提供了两个基类，五个视图扩展类，9个视图集

drf提供了一个Django中view的子类`APIView `,主要变动大概为以下：

- 重新封装了`Request` 与 `Response`实例。
  - 使用了独有的Request与Response对象，并且提供了专有的解析器 `Parser` 可以根据HTTP `Content-Type` 指明的请求数据进行解析。
- 增加了自有的鉴权/节流
  - 在django中`dispatch()` 分发前，会对请求进行身份认证、权限检查、流量控制。
- 异常捕获 `APIException`。

APIView implement

```python
@classmethod
def as_view(cls, **initkwargs):
    ....
	# 调用父类的方法，Python 3 可以使用直接使用 super().xxx 代替 super(Class, self).xxx
    view = super(APIView, cls).as_view(**initkwargs)
    view.cls = cls
	# 并且生成一个新的request
    view.initkwargs = initkwargs

    # Note: session based authentication is explicitly CSRF validated,
    # all other authentication is CSRF exempt.
    return csrf_exempt(view)

## 父类的view会执行dispatch分配为对应的handle memory，通过method获得对应的方法处理请求
    def dispatch(self, request, *args, **kwargs):
        # Try to dispatch to the right method; if a method doesn&#39;t exist,
        # defer to the error handler. Also defer to the error handler if the
        # request method isn&#39;t on the approved list.
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)
```

## What is GenericAPIView

`GenericAPIView` 是继承与 `APIView`的子类，在 `APIView` 的基础上增加了对于视图的通用支持方法，用来简化用户代码的编写。主要增加了 `QuerySet` 与 `Serializers`

**GenericAPIView implement**



```python
class GenericAPIView(views.APIView):
  
    queryset = None
    serializer_class = None

    lookup_url_kwarg = None

 
    def get_queryset(self):
      ...
        assert self.queryset is not None, (
            &#34;&#39;%s&#39; should either include a `queryset` attribute, &#34;
            &#34;or override the `get_queryset()` method.&#34;
            % self.__class__.__name__
        )

        queryset = self.queryset
        if isinstance(queryset, QuerySet):
            # Ensure queryset is re-evaluated on each request.
            queryset = queryset.all()
        return queryset

```

## How to Use

&gt; **Reference**
&gt;
&gt; [APIView](https://www.django-rest-framework.org/api-guide/views/)
&gt;
&gt; [GenericAPIView](https://www.django-rest-framework.org/api-guide/generic-views/)

使用`APIView`与使用`View`类似，像往常一样，请求会根据不同的方法被`dispatch`到对应的处理逻辑方法，例如`.get()`or `.post()`

引入

```python
from rest_framework.views import APIView
from rest_framework.response import Response
```

使用`GenericAPIView` 是 `APIView` 的子类，是实现了`APIView` 的常用行为的一个类。一般情况下会与引入

- `queryset`：对象查询集，使用`GenericAPIView` 必须设置该属性，或者重写 `get_queryset()` 方法
- `serializer_class`: 序列化器类，必须设置该属性或重写`get_serializer_class()`方法。
- `lookup_field`: 查库时使用的条件字段，一般为传入的值，默认为pk
- `pagination_class` ：分页

```python
from rest_framework import generics
class BookViewSet(generics.GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookModelSerializer

    def get(self, request):
        book_list = self.get_queryset()
        book_serializers = self.get_serializer(book_list, many=True)
        return Response(book_serializers.data)
    def delete(self, reques, pk):
        book = self.get_object().delete()
        return Response({&#34;message&#34;:&#34;success&#34;, &#34;status&#34;:100})
```

### 五个视图扩展

Mixin类：DRF提供的通用的增删改查行为，Mixin一般与`generics.GenericAPI` 混用，可以组成灵活的视图。

- `CreateModelMixin`: 保存新对象实例
  - 创建成功返回201与序列化后的列表，失败则返回400与错误的详细信息
- `UpdateModelMixin` ：对现有对象实例进行更新
  - 与创建相同，成功返回200，失败返回400
- `DestroyModelMixin`：删除对象实例
  - 成功删除返回204 错误将返回一个404
- `ListModelMixin`：列出实例列表
  - 查询成功返回200，需要设置queryset，相应数据可以设置分页
- `RetrieveModelMixin`: 只读操作单个对象

### 九个视图集

&gt; 在路由确定用于请求的控制器之后，您的控制器负责理解请求并产生适当的输出。
&gt;
&gt; — [Ruby on Rails 文档](https://guides.rubyonrails.org/action_controller_overview.html)Django REST 框架允许您将一组相关视图的逻辑组

视图集 `ViewSet` 是DRF基于view使用视图集ViewSet，可以将一系列逻辑相关的动作放到一个类中，如`.get()`或`.post()`则不在提供了，换为`.list()`和`.create()`的具体逻辑动作。


