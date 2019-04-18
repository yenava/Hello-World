# QuerySet API：

## 模型.objects：
这个对象是`django.db.models.manager.Manager`的对象，这个类是一个空壳类，他上面的所有方法都是从`QuerySet`这个类上面拷贝过来的。因此我们只要学会了`QuerySet`，这个`objects`也就知道该如何使用了。
`Manager`源码解析：
```python
class_name = "BaseManagerFromQuerySet"

class_dict = {
    '_queryset_class': QuerySet
}

class_dict.update(cls._get_queryset_methods(QuerySet))

# type动态的时候创建类
# 第一个参数是用来指定创建的类的名字。创建的类名是：BaseManagerFromQuerySet
# 第二个参数是用来指定这个类的父类。
# 第三个参数是用来指定这个类的一些属性和方法
return type(class_name,(cls,),class_dict)

_get_queryset_methods：这个方法就是将QuerySet中的一些方法拷贝出来
```

## filter/exclude/annotate：过滤/排除满足条件的/给模型添加新的字段。
> 返回对象为QuerySet，可以链式调用

### Django objects.all()、objects.get()与objects.filter()区别

示例代码:

> ret=UserInfo.objects.all()  

all返回的是QuerySet对象，程序并没有真的在数据库中执行SQL语句查询数据，但支持迭代，使用for循环可以获取数据。

> ret=UserInfo.objects.get(id='1')  

get返回的是Model对象，类型为列表，说明使用get方法会直接执行sql语句获取数据

> ret=UserInfo.objects.filter()  

filter和get类似，但支持更强大的查询功能

### 补充：
条件选取querySet的时候，filter表示=，exclude表示!=。 
querySet.distinct() 去重复

> __exact 精确等于 like ‘aaa’ __iexact 精确等于 忽略大小写 ilike ‘aaa’  
__contains 包含 like ‘%aaa%’ __icontains 包含 忽略大小写 ilike   ‘%aaa%’，但是对于sqlite来说，contains的作用效果等同于icontains。  
__gt 大于  
__gte 大于等于  
__lt 小于  
__lte 小于等于  
__in 存在于一个list范围内  
__startswith 以…开头  
__istartswith 以…开头 忽略大小写  
__endswith 以…结尾  
__iendswith 以…结尾，忽略大小写  
__range 在…范围内  
__year 日期字段的年份  
__month 日期字段的月份  
__day 日期字段的日  
__isnull=True/False  


## order_by：
```python
# 根据创建的时间正序排序
articles = Article.objects.order_by("create_time")
# 根据创建的时间倒序排序
articles = Article.objects.order_by("-create_time")
# 根据作者的名字进行排序
articles = Article.objects.order_by("author__name")
# 首先根据创建的时间进行排序，如果时间相同，则根据作者的名字进行排序
articles = Article.objects.order_by("create_time",'author__name')
```

一定要注意的一点是，多个`order_by`，会把前面排序的规则给打乱，而使用后面的排序方式。比如以下代码：

```python
articles = Article.objects.order_by("create_time").order_by("author__name")
```

他会根据作者的名字进行排序，而不是使用文章的创建时间。
当然，也可以在模型定义的在`Meta`类中定义`ordering`来指定默认的排序方式。示例代码如下：
```python
    class Meta:
        db_table = 'book_order'
        ordering = ['create_time','-price']
```

还可以根据`annotate`定义的字段进行排序。比如要实现图书的销量进行排序，那么示例代码如下：
```python
books = Book.objects.annotate(order_nums=Count("bookorder")).order_by("-order_nums")
    for book in books:
        print('%s/%s'%(book.name,book.order_nums))
```