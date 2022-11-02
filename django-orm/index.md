# 

https://www.cnblogs.com/Dominic-Ji/p/11516152.html

对象关系映射（Object Relational Mapping，简称ORM）模式是一种为了解决面向对象与关系数据库存在的互不匹配的现象的技术。

简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系数据库中。

ORM在业务逻辑层和数据库层之间充当了桥梁的作用。

## django中仅测试ORM

导入model，然后直接使用对应对象进行ORM操作。

```
import os
if __name__ == &#34;__main__&#34;:
    os.environ.setdefault(&#34;DJANGO_SETTINGS_MODULE&#34;, &#34;app.settings&#34;)
    import django
    django.setup()
    from xxx import models
    models.User.objects.all()
```



## 连接数据库

django配置数据库

```python
DATABASES = {
    &#39;default&#39;: {
        &#39;ENGINE&#39;: &#39;django.db.backends.mysql&#39;,
        &#39;USER&#39;: &#39;root&#39;,
        &#39;PASSWORD&#39;: &#39;111&#39;,
        &#39;HOST&#39;:&#39;127.0.0.1&#39;,
        &#39;NAME&#39;: &#39;book&#39;,
        &#39;CHARSET&#39;: &#39;utf8&#39;
    }
}
```

**可选**：`pymysql` 使用模块连接MySQL数据库:：在项目中`__init__.py` 文件中添加配置：

```python
import pymysql
pymysql.install_as_MySQLdb()
```

## 创建（表）对象

ORM中，`O` (Object)  代表&#34;对象&#34;，而`R`(Relational) 则代表&#34;关系&#34;。所以创建表即创建一个类，字段则是类的属性。类的每个实例则对应表中的一条记录。

在Django中model就是你数据来源。通常，一个model映射到一个数据库表，一般情况下基本满足：

- 每个model（表）都是一个Python类，它是`django.db.models.Model`的子类即继承`models.Model`。

- 类的每个属性都代表一个字段（字段）。

- 实例化出的对象，代表表中的记录。

例如

```python
class User(models.Model):
name = models.CharField(max_length=32)
age = models.IntegerField()
registration_time = models.DateField()
```

### 字段类型

&gt; **Reference**
&gt;
&gt; [字段类型说明](https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/#field-types)
&gt;
&gt; [字段选项说明](https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/#field-options)

&gt;  **必需掌握字段类型说明：**

- `BigIntegerField` or `BigAutoField`: `1~9223372036854775807` 的64位自增int
- `CharField`: 用于存储字符串，对应MySQL中varchar
- `DateField`:  python中的`datetime.date.today()` 即 `Y-m-d`
- `DateTimeField`：`timezone.now` - `django.utils.timezone.now()`为`YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]`，相当于Python中的`datetime.datetime()`

- `DecimalField`：小数，使用：`models.DecimalField(..., max_digits=5, decimal_places=2)`
- `BooleanField`: 代表一个true/false的布尔值。
- `AutoField`: int类型的自增列，必须填入参数`primary_key=True` 。当model中如果没有自增列，则自动会创建一个列名为id的列。

### 字段参数

&gt;**Reference**
&gt;
&gt;[字段选项说明](https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/#field-options)

**必须掌握的字段选项**：

- `null`：表示该字段是否允许空值，如果为true，django将在数据库中将空值存储为null。默认为false。使用：`models.CharField(null=True)`
- `db_index`：是否对字段创建索引，如果为true，将为此字段创建数据库索引。
- `default`：字段的默认值。这可以是值或可调用的对象。如果可调用它将每次创建新对象时调用它。
- `primary_key`：`primary_key=True`，该字段为主键。

时间字段特殊参数：

- `auto_now_add`:  `auto_now_add=True`，仅在创建对象时将当前时间插入到数据库中。如果不设置
- `auto_now`: `auto_now=True`，每次更新数据记录的时候会更新该字段。

##  增

### save()

Python对象中表示数据库表数据，模型类表示数据库表，该类的实例表示数据库表中的记录。

要创建一个对象，然后调用`save()`将其保存到数据库中。

`save()` 直到调用时，才操作数据库，并且没有返回值。

`save()` 必须实例化后调用

```python
b = models.User(name=&#34;zhangsan&#34;,age=18)
b.save()
```

### create()

创建对象并且保存

```python
models.User.objects.create(name=&#34;lisi&#34;, age=19)
```

## 删

### delete()

`delete()` 在查询集中的所有行上执行SQL DELETE，并返回删除的对象数量和每个对象类型的删除次数的字典。

delete()无法对`QuerySet` 上调用`delete()`

```python
models.User.objects.filter(pk=1).delete()
```

## 查

### 查询必会的方法

**返回值为QuerySet对象的方法有**

`all()` 查询所有数据
`filter()` 带有过滤条件的查询
`exclude()` 排除数据，`exclude(&#39;xxx=xxx&#39;)`
`order_by()` 排序 降序 `models.User.objects.order_by(&#39;-age&#39;)`
`reverse()` 反转，反转的数据必须是 `order_by()`后的数据
`distinct()` 去重，主键是唯一值，需要过滤主键

**返回值为特殊的QuerySet**

`values()`    返回一个可迭代的字典序列。（列表套字典）
`values_list()` 返回一个可迭代的元祖序列。（列表套`QuerySet`）

**返回值为具体对象**

`get()`
`first()`
`last()`

**返回值布尔值：**

`exists()`

**返回值为数字**

`count()` 统计当前数据个数

#### django 查看原生SQL的方法

- ``QuerySet` 可以使用`models.User.objects.values_list().query`

- 终端打印，在`setting.py`中配置下列

  ```python
  LOGGING = {
      &#39;version&#39;: 1,
      &#39;disable_existing_loggers&#39;: False,
      &#39;handlers&#39;: {
          &#39;console&#39;:{
              &#39;level&#39;:&#39;DEBUG&#39;,
              &#39;class&#39;:&#39;logging.StreamHandler&#39;,
          },
      },
      &#39;loggers&#39;: {
          &#39;django.db.backends&#39;: {
              &#39;handlers&#39;: [&#39;console&#39;],
              &#39;propagate&#39;: True,
              &#39;level&#39;:&#39;DEBUG&#39;,
          },
      }
  }
  ```

  ###  条件查询

  #### 基于双下划线的查询

  查询大于的18的用户 `models.User.objects.filter(age__gt=12)`

  查询年龄为18,19,20岁的用户 `models.User.objects.filter(age__in=[18,19,20])`

  查询90后用户 `models.User.objects.filter(age__range=[22,31])`

  模糊查询：查询名字包含`l` 的用户：`models.User.objects.filter(name__contains=&#39;li&#39;)`

  模糊查询：忽略大小写查询：`models.User.objects.filter(name__icontains=&#39;li&#39;) ` 

  查询注册时间为 2020 7月份数据：`models.User.objects.filter(registration_time__month=7)`

## 多表操作

在django中外键的存在使得`ORM`框架在处理表关系的时候异常的强大。在Django中，外键类定义为：`class ForeignKey(to,on_delete,**options)` 。可以看到外键的参数大致分为：

- to：引用那个model（表）。
- on_delete：当使用了外键引用model（表）的数据被删除后的操作。

**定义一个外键**：

在关系数据库中外键的作用是在于将表彼此关联起来。Django提供了定义三种最常见的数据库关系类型的方法：多对一、多对多和一对一。

而关系型字段分为：

| 关系型字段                                                   | 对应关系 |
| ------------------------------------------------------------ | -------- |
| [ForeignKey](https://docs.djangoproject.com/zh-hans/2.0/topics/db/models/#many-to-one-relationships) | 多对一   |
| [ManyToManyField](https://docs.djangoproject.com/zh-hans/2.0/topics/db/models/#many-to-many-relationships) | 多对多   |
| [OneToOneField](https://docs.djangoproject.com/zh-hans/2.0/topics/db/models/#one-to-one-relationships) | 一对一   |

如下：

- 一个作者可以写多本书，但一本书只能由一个出版社出版，使用 `ForeignKey` 可以直接使用Book实例中通过 `Press` 属性来操作对应的`Press`模型。
- 一本书 可以由多个 `Author` 编写，也可以由一个作者  `Author` 编写，但一个作者( `Author`)也可以编写多本书 `Book`。
-  一般情况下，出版社仅记录`Author`的一个联系方式，也就是 `Author` 与 `AuthorDetail` 为一对一关系。

```python
class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    publishData = models.DateField(auto_now_add=True)

    press = models.ForeignKey(to=&#34;Press&#34;)
    author = models.ManyToManyField(to=&#34;Author&#34;)
    
class Press(models.Model):
    name = models.CharField(max_length=32,null=True)
    address = models.CharField(max_length=32)
    email = models.EmailField()

class Author(models.Model):
    name = models.CharField(max_length=32,null=True)
    age = models.IntegerField()
    authorDetail = models.OneToOneField(to=&#34;AuthorDetail&#34;)

class AuthorDetail(models.Model):
    phoneNumber = models.BigIntegerField()
    address = models.CharField(max_length=32)
```

### 外键的基本操作

添加外键关系：

```python
bookobj = models.Book.objects.filter(pk=1).first()
bookobj.author.add(1) # 给主键为1的书籍绑定一个主键1的作者
bookobj.author.add([1,2,3]) # 给主键为1的书籍绑定多个作者
```

移除关系：`bookobj.author.remove(1)`

修改关系：`bookobj.author.set(2)`

清空该关系：`bookobj.author.clear() # 清除所有这个作者的书`

### 正反向概念

正向查询：在子表中，查询父表（外键所在表）的信息
反向查询：通过父表，查询子表的信息

### 多表查询

查询口诀：正向查询按外键字段，反向查询按表名（model）

`all()` 当结果为多个时，需要使用`.all()` 如多对多，一对多

**查询书籍1的出版社** 正向查询

```
book = models.Book.objects.filter(pk=1).first()
print book.press.name
```

**查询书籍1的作者** 正向查询

```
book = models.Book.objects.filter(pk=1).first()
print book.author.all()
```

**查询作者1的电话**

```
auther = models.Author.objects.filter(pk=1).first()
print auther.authorDetail.phoneNumber
```

**查询出版社拥有的书** 反向查询

```python
press = models.Press.objects.filter(pk=1).first()
books = press.book_set.all()
```

**查询作者Phoenix写的书**

```python
auther = models.Author.objects.filter(name=&#34;Phoenix&#34;).first()
print auther.book_set.all()
```

**根据手机号查询作者**

```python
phone = models.AuthorDetail.objects.filter(phoneNumber=1511111111).first()
print phone.author.name
```

### 基于下划线的查询

**根据名称查询手机号**

```python
models.Author.objects.filter(name=&#34;Phoenix&#34;).values(&#34;authorDetail__phoneNumber&#34;)
# 获取两个表中的字段
models.Author.objects.filter(name=&#34;Phoenix&#34;).values(&#34;authorDetail__phoneNumber&#34;,&#34;name&#34;)
```

**查询书籍1的作者名称**

```python
# 正向
models.Book.objects.filter(pk=1).values(&#34;author__name&#34;)
# 反向
models.AuthorDetail.objects.filter(author__name=&#34;Phoenix&#34;).values(&#34;phoneNumber&#34;,&#34;author__name&#34;)
```

查询书籍1的作者手机号

```
models.Book.objects.filter(pk=1).values(&#34;author__authorDetail__phoneNumber&#34;)
```

## 聚合查询 (aggregate)

`aggregate()` 是 `QuerySet` 的一个终止子句，意思是说，它返回一个包含一些键值对的字典。键的名称是聚合值的标识符，值是计算出来的聚合值。键的名称是按照字段和聚合函数的名称自动生成出来的。



使用聚合查询需要引入具体的类 `from django.db.models import Avg,Max,Min,Count,Sum`

获取书的总数 `Book.objects.count()`

对数据进行聚合查询：`aggregate(别名 = 聚合函数名[avg,max..](&#34;属性名称&#34;))`

## 分组查询（annotate）

分组查询一般会与聚合函数一起使用，使用前也许引入具体类：`from django.db.models import Avg,Max,Min,Count,Sum`

返回值：

- 分组后，用 values 取值，则返回值是 QuerySet 数据类型里面为一个个字典；
- 分组后，用 values_list 取值，则返回值是 QuerySet 数据类型里面为一个个元组。

分组位置 `annotate`：

- **values  or values_list 在 annotate 前：**values 或者 values_list 是声明以什么字段分组，annotate 执行分组。
- **values or values_list 在annotate后：** annotate 表示直接以当前表的pk执行分组，values 或者 values_list 表示查询哪些字段， 并且要将 annotate 里的聚合函数起别名，在 values 或者 values_list 里写其别名。

**统计每本书的作者有几个**

```python
models.Book.objects.annotate(autherNum=Count(&#39;author__id&#39;)).values(&#39;autherNum&#39;,&#39;title&#39;)
```

**统计出版社最便宜书的价格**

```python
 models.Press.objects.annotate(minPrice=Min(&#39;book__price&#39;)).values(&#34;name&#34;, &#34;minPrice&#34;)
```

**统计不止一个作者的书**

```python
models.Book.objects.annotate(autherCount=Count(&#34;author__id&#34;)).filter(autherCount__gt=1).values(&#34;title&#34;,&#34;autherCount&#34;)
```

**统计作者出书的总价**

```python
models.Author.objects.annotate(bookPrice=Sum(&#34;book__price&#34;)).values(&#34;name&#34;,&#34;bookPrice&#34;)
```

**根据指定字段分组**

## F&amp;Q查询

### F查询

F 可以在对Model字段值的转换时，无需从数据库中将值加载到内存中，进行操作后再`save()`。

例如。通常情况下，在更新数据时需先从数据库里将原数据加载到内存里，编辑后最后提交。

```python
order = Order.objects.get(orderid=&#39;1&#39;)
order.amount &#43;= 1
order.save()
```

而F 可以直接对值进行运行而不必将数据从库中拉到内存中。例如

**卖出大于库存的书籍**

```python
models.Book.objects.filter(sell__gt=F(&#39;stock&#39;))
```

**对所有书籍价格增加100**

```python
models.Book.objects.update(price=F(&#39;price&#39;)&#43;100)
```

### Q查询

ORM filter() 等方法中的关键字参数查询都是一起进行 `AND`  的。 如需要执行更复杂的查询（例如OR语句），你可以使用Q对象。

Q是对查询条件进行字符串拼接，故可以组合 `&amp;`  和`|`  等操作符以及使用括号进行分组来编写任意复杂的Q对象。同时，Q 对象可以使用`~` 操作符取反。

Q 对象允许组合正常的查询和取反(`NOT`) 查询。



如：**查询作者是的Radamandis和Phoenix**

```python
models.Book.objects.filter(Q(authors__name=&#34;Phoenix&#34;)|Q(authors__name=&#34;Radamandis&#34;))
```

**查询作者不是Phoenix的书**

```python
models.Book.objects.filter(~Q(author__name=&#34;Phoenix&#34;))
```

**也可以进行组合查询**: 查询作者不是Phoenix 并且价格大于700

```
models.Book.objects.filter(~Q(author__name=&#34;Phoenix&#34;) &amp; Q(publishData__gt=&#34;2021-08-04&#34;))
```

Q的第二种使用方法

```python
query = Q()
query.connector = &#39;OR&#39; #默认为and
query.children.append((&#39;id&#39;, 1))
query.children.append((&#39;id&#39;, 2))
query.children.append((&#39;id&#39;, 3))

models.Book.objects.filter(query)
```

## 事务



&gt; Reference
&gt;
&gt; [transactions](https://docs.djangoproject.com/zh-hans/3.2/topics/db/transactions/)

在操作多表，或多次变更数据时，这些数据的修改应该是一个整体事务，即要么一起成功，要么一起失败。Django 默认的事务行为是自动提交，即每执行一次则会自动提交到数据库。



在django中事务的使用是通过`django.db.transaction`模块提供的`atomic`来定义事务。所以使用事务需要先引入`from django.db import transaction`

事务的使用可以通过`装饰器` 或 `with`语句。



通过装饰器方式（全局事务），在整个函数内为一个事务，要么一起成功，要么一起失败。

```python
@transaction.atomic
def test():
    models.Press.objects.create(name=&#34;Yasgot&#34;,address=&#34;Nordische Botschaften&#34;, email=&#34;yasgot.com@gamil.com&#34;)
    print &#34;insert ok.&#34;
    time.sleep(10)
    models.Press.objects.create(name=&#34;Yasgot&#34;,address=&#34;Nordische Botschaften&#34;, email=&#34;yasgot.com@gamil.com&#34;)
    book = models.Press.objects.all()
    print book
```

通过with方式（局部事务），在函数中，使用 `with transaction.atomic():` 代码块内的为一个事务。

```python
def viewfunc(request):
	with transaction.atomic():
	# 这部分代码会在事务中执行
```

### 事务的异常处理

#### 保存点

保存点（`savepoint`），在事务中可以做到部分回滚，而不是整个事务。

`atomic()` 为开启一个事务，而回滚是通过，`transaction.rollback()` 执行的完全回滚。而django也推荐仅使用`atomic()`。

- [`savepoint(*using=None*)`](https://docs.djangoproject.com/zh-hans/3.2/topics/db/transactions/#django.db.transaction.savepoint)：创建新的保存点，返回保存点ID (`sid`) 。

- [`savepoint_commit(*sid*, *using=None*)`](https://docs.djangoproject.com/zh-hans/3.2/topics/db/transactions/#django.db.transaction.savepoint_commit)：释放保存点 `sid` 。如回滚等将不在保证之前的保存点的数据而是整个事务。

- [`savepoint_rollback(*sid*, *using=None*)`](https://docs.djangoproject.com/zh-hans/3.2/topics/db/transactions/#django.db.transaction.savepoint_rollback):回滚事务 `sid` 。

下面是官方的一个例子：[example-to-savepoint](https://docs.djangoproject.com/zh-hans/3.2/topics/db/transactions/#django.db.transaction.clean_savepoints)

```python
from django.db import transaction

# open a transaction
@transaction.atomic
def viewfunc(request):

    a.save()
    # transaction now contains a.save()

    sid = transaction.savepoint()

    b.save()
    # transaction now contains a.save() and b.save()

    if want_to_keep_b:
        transaction.savepoint_commit(sid)
        # open transaction still contains a.save() and b.save()
    else:
        transaction.savepoint_rollback(sid)
        # open transaction now contains only a.save()
```

## 执行原生SQL

&gt; **Reference**
&gt;
&gt; [raw-sql](https://docs.djangoproject.com/zh-hans/3.2/topics/db/sql/)

**`raw()`** :执行原生语句 `django.db.models.query.RawQuerySet`

**` django.db.connection()`**：；连接多个库 `from django.db import connection`

## 自定义字段类

&gt; **Reference**
&gt;
&gt; [custom-filed](https://docs.djangoproject.com/zh-hans/3.2/howto/custom-model-fields/)

```
class FixedCharField(models.Field):
    &#34;&#34;&#34;
    自定义的 char 类型的字段类
    &#34;&#34;&#34;
    def __init__(self, max_length, *args, **kwargs):
        self.max_length = max_length
        super(FixedCharField, self).__init__(max_length=max_length, *args, **kwargs)

    def db_type(self, connection):
        &#34;&#34;&#34;
        限定生成数据库表的字段类型为 char，长度为 max_length 指定的值
        &#34;&#34;&#34;
        return &#39;char(%s)&#39; % self.max_length
```


