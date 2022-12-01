# python django-restframework serializers




## What is serializers？

serializers主要作用是将原生的Python数据类型（如 `model` `querysets `）转换为web中通用的`JSON`，`XML`或其他内容类型。

`DRF` 提供了一个`Serializer`类，它为您提供了种强大的通用方法来控制响应的输出，以及一个`ModelSerializer `类，它为创建处理 `model instance` 和 `serializers` 提供了一个序列化的快捷方式。

> **Reference**
>
> [drf serializers manual](https://www.django-rest-framework.org/api-guide/serializers/)

## How to Declaring Serializers?

序列化一个django model

```python
class Comment:
    def __init__(self, email, content, created=None):
        self.email = email
        self.content = content
        self.created = created or datetime.now()

comment = Comment(email='leila@example.com', content='foo bar')
```

声明Serializers，可以用来序列化与反序列化对象 `Comment`的属性及值。

```python
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField() # 属性名称与类Comment名校相同
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```

## 序列化及反序列化

## 序列化

```python
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
    
# 上面类似于如下python中的操作
from rest_framework.renderers import JSONRenderer

json = JSONRenderer().render(serializer.data)
json
# b'{"email":"leila@example.com","content":"foo bar","created":"2016-01-27T15:17:10.375877"}'
```

### 反序列化

反序列化是将json数据流解析为python的数据类型，后映射至对象

```python
import io
from rest_framework.parsers import JSONParser

stream = io.BytesIO(json)
data = JSONParser().parse(stream)

serializer = CommentSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# {'content': 'foo bar', 'email': 'leila@example.com', 'created': datetime.datetime(2012, 08, 22, 16, 20, 09, 822243)}
```

## 数据的落地

如果需要对经过认证的数据进行保存入库，需要实现对应 serializer的 `create()` 和 `update()` 方法

```python
class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()

    def create(self, validated_data): # validate_data 实际与 Comment一致，打散后为
        return Comment(**validated_data)

    def update(self, instance, validated_data): # drf serializer实现了对应的实例，instance是该serializer，vilidated是对应的属性
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        return instance
```

```
serializer = CommentSerializer(data={'email': 'foobar', 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'email': ['Enter a valid e-mail address.'], 'created': ['This field is required.']}
```

### save()

`save()` 可以创建或更新一个实例（实例是值库中的行）。

```python
# .save() will create a new instance.
serializer = CommentSerializer(data=data)

# .save() will update the existing `comment` instance.
serializer = CommentSerializer(comment, data=data)
```

## How to Use validate?

validate是值在反序列化数据时，需要对数据进行验证（如，长度，值，类型），即在数据落地前，对其制定的规则进行验证。

```python
serializer = CommentSerializer(data={'email': 'foobar', 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'email': ['Enter a valid e-mail address.'], 'created': ['This field is required.']}
```

`.is_valid()` 是对数据的验证。`raise_exception` 是一个可选参数，如果 `serializers.ValidationError`如果存在验证错误，将引发异常。异常由 REST framework 提供的默认异常处理程序自动处理，并`HTTP 400 Bad Request`默认返回响应。

```python
# Return a 400 response if the data was invalid.
serializer.is_valid(raise_exception=True)
```

### 字段的验证

#### 自定义验证

**单字段验证**

通过子类 `.validate_<field_name>` 方法进行自定义验证方式，该方法需要返回验证的值或触发`serializers.ValidationError`. 例如：

```python
from rest_framework import serializers

class BlogPostSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=100)
    content = serializers.CharField()

    def validate_title(self, value):
        """
        Check that the blog post is about Django.
        """
        if 'django' not in value.lower():
            raise serializers.ValidationError("Blog post is not about Django")
        return value
```

**类级别验证**



如果需要对多个字段进行验证验证，需要在类中实现`validate()` 方法。该方法仅单个参数 `data`, 为验证的字段的字典。例如

```python
from rest_framework import serializers

class EventSerializer(serializers.Serializer):
    description = serializers.CharField(max_length=100)
    start = serializers.DateTimeField()
    finish = serializers.DateTimeField()

    def validate(self, data):
        """
        Check that start is before finish.
        """
        if data['start'] > data['finish']:
            raise serializers.ValidationError("finish must occur after start")
        return data
```

#### 忽略验证

**注意：**如在Serializer 的`<field_name>` 声明了参数`required=False` 则该字段不会进行验证。

#### 指定验证器

Serializer 的`<field_name>` 还可以声明 validator，例如，

```python
def multiple_of_ten(value):
    if value % 10 != 0:
        raise serializers.ValidationError('Not a multiple of ten')

class GameRecord(serializers.Serializer):
    score = IntegerField(validators=[multiple_of_ten])
    ...
```

> validator Reference
>
> [validator](https://www.django-rest-framework.org/api-guide/validators/)

## ModelSerializer

`ModelSerializer`，是drf为了方便实现好的可以直接用的Serializer。实现为：

- 将根据模型自动为您生成一组字段。
- 将自动为Serializer程序生成validator，例如 unique_together 验证器。
- 包括简单的实现默认的`.create()`和`.update()`。

### ModelSerializer的声明

```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ['id', 'account_name', 'users', 'created']
```

#### Meta的说明

`Meta` 类，如名称可知，这是设置Serializer的一些元数据。包含`Model`,`Filed`, `Validator`等信息，例如声明一个Meta类。

```python
class EventSerializer(serializers.Serializer):
    name = serializers.CharField()
    room_number = serializers.IntegerField(choices=[101, 102, 103, 201])
    date = serializers.DateField()

    class Meta:
        # 通过在内部Meta类中声明validators来包含，如下所示：
        validators = [
            UniqueTogetherValidator(
                queryset=Event.objects.all(),
                fields=['room_number', 'date']
            )
        ]
        # 通过在内部Meta类中声明model来包含对应使用model，如下所示：
        model = User
        # fields 可以指定要序列化的字段，'__all__'为model中的所有字段
        fields = ['username', 'email', 'profile']
        exclude=['username'] # exclude是要排除的字段
```

> 注：从 3.3.0 版开始，**必须**提供以下属性之一`fields`或`exclude`.

Serializer会在Meta中拿取自己对应的属性进行使用，例如

```python
meta = getattr(self, 'Meta', None)
validators = getattr(meta, 'validators', None)

# assert <condition>,(..error message)
# 可以看到Meta和Meta.model必须要设置
assert hasattr(self, 'Meta'), (
    'Class {serializer_class} missing "Meta" attribute'.format(
        serializer_class=self.__class__.__name__
    )
)
assert hasattr(self.Meta, 'model'), (
    'Class {serializer_class} missing "Meta.model" attribute'.format(
        serializer_class=self.__class__.__name__
    )
)
if model_meta.is_abstract_model(self.Meta.model):
    raise ValueError(
        'Cannot use ModelSerializer with Abstract Models.'
    )
```



### 其他用法

设置只读字段：字段属性中添加 `read_only=True`, 或者在Meta类中添加属性 中指定字段 `read_only_fields` 为列表。

[readonly-field](https://www.django-rest-framework.org/api-guide/serializers/#specifying-read-only-fields)



## Serializer的字段与字段属性属性

> Reference
>
> [fields](https://www.django-rest-framework.org/api-guide/fields)

### 字段属性

- `read_only` 在创建或更新时改属性True字段都被忽略
- `write_only` 仅为创建或更新时使用，序列化时不操作该字段
- `required` 默认情况下，在反序列化时未提供字段会引发错误，如果不需要可以设置为`False`
- `source`：
  - 用于序列化时，填充替代对应字段名称的作用 `URLField(source='get_absolute_url')`
  - 可以跨表
  - 可以执行对象内方法。
- `Many`: 可以返回多个对象，而非一个，在objects.all时使用

### 字段类型

- `BooleanField()` 
- `CharField(max_length=None, min_length=None, allow_blank=False, trim_whitespace=True)` 文本字段
-  `EmailField(max_length=None, min_length=None, allow_blank=False)` email字段
- `RegexField(regex, max_length=None, min_length=None, allow_blank=False)` 正则表达式
- `IPAddressField(protocol='both', unpack_ipv4=False, **options)` IP地址
- `SerializerMethodField(method_name=None)` 通过方法序列化，只读字段 
  - `method_name` 序列化时通过方法的名称。默认为`get_<field_name>`.

