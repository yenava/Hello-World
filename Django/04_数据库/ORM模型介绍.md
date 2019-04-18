### ORM模型
采用写原生sql语句会在代码中产生一些问题
> sql重复利用率不高  
很多sql是在业务逻辑中拼写出来的，如果有数据库要修改，就要修改这些逻辑，就容易产生很多bug  
sql容易忽略web安全问题，给未来带来隐患  

ORM全称Objects Relational Mapping，对象关系映射，通过ORM可以用类的方式去操作数据库，不用再写原生SQL语句。原理是把表映射成类，把行做实例，字段做属性，最终执行对象时会把对应的操作转化为SQL语句。有以下几个优点：  
> 易用性，减少原生SQL的使用，直观清晰  
性能损耗小  
设计灵活  
可移植性

需要将app添加进settings里，在app下的model文件中配置需要的映射。  
使用makemigrations生成脚本文件  
>python manage.py makemigrations  
把你写在models中的代码翻译成增、删、改的 SQL 语句

使用migrate将新生成的迁移脚本文件映射到数据库中  
>python manage.py migrate
将python manage.py makemigrations翻译的SQL语句去数据库执行
python manage.py migrate --fake；假设 migrate 把所有SQL语句执行成功了

出现报错的原因：
>无非就是   makemigrations翻译的 SQL，跟数据库里面真实的表结构 相互冲突,增加、删除不了表、外键关系；  
所以遇到报错 你应该先把model中的代码注释掉-----》python manage.py migrate --fake------》python manage.py makemigrations  
去数据库把已经存在的表、外键删掉（确保数据库目前的状态，可以让makemigrations翻译出来的SQL语句在数据库里执行成功；然后migrate）--------》 python manage.py migrate；

在Django中操作数据库有两种方式。第一种方式时使用原生**sql**语句操作，第二种就是使用__ORM__模型来操作。  
在Django中使用原生sql语句操作其实就是使用python db api的接口来操作。如果你的mysql驱动使用的时pymysql，那么你就是使用pymysql来操作的，只不过Django将数据库连接的这一部分封装好了，我们只要在settings中配置好了数据库的连接信息后直接使用Django封装好的api就可以操作了。

在连接数据库中如果遇到  
**django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.  
Did you install mysqlclient?**   
需要在init文件中加入如下代码
```python
import pymysql

pymysql.install_as_MySQLdb()

```

## ORM增删改查
### 使用ORM添加一条数据到数据库中
```python
book = Book(name='西游记',author='吴承恩',price=30)
book.save()
```
### 查询操作
### 1.主键查找
```python
book = Book.objects.get(pk=1) # 或id=1
print(book)
```
### 2.根据其他条件查找
```python
Book.objects.filter(name='西游记') 会返回所有name为西游记的行，返回值是个列表
Book.objects.filter(name='西游记').first() 返回匹配的第一个
```
### 删除数据    
```python    
book = Book.objects.get(pk=1)
book.delete()
```
### 修改数据    
```python
book = Book.objects.get(pk=2)
book.price = 200
book.save()
```

