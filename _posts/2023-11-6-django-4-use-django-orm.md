---
title: 4. Django orm的基本使用
date: 2023-11-6 02:01:22 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

django orm的基本用法:

```python
from characters.models import Character
query = Character.objects.all()
```

通过直接调用模型类的objects成员的对应方法即可创建查询对象

所创建的查询对象是延迟加载的,  以此来方便进行符合查询,如:

```python
Character.objects.all().order_by('level')
```

只有当确定读取数据的时候才会真正进行数据库查询, 如

```python
list(query)
query[0:5]
```

即可以减少和服务器交互的次数, 也可以减少内存占用.

## 查询对象

### get()

```python
character = Character.objects.get(pk=1)
```

如果id不存在, 则会引发异常, 因此需要为其进行异常处理

```python
from django.core.exceptions import ObjectDoesNotExist

try:
    obj = Character.objects.get(pk=pk)
except ObjectDoesNotExist:
    pass
```

### filter()

```python
# 作用于get()想通, 区别在于不需要异常处理
Character.objects.filter(pk=pk).first()
```

filter()方法可以接收模型类所包含字段的过滤器, 其格式为:

```python
filedname__lookup=value
```

比如`name`字段包含`woo`的行,就可以写成

```python
query_set = Character.objects.filter(name__contains="woo")

# __contains代表包含, __icontains代表忽略大小写包含
```

#### 常用过滤器lookup

* `range`: 查询范围, __range=(10,20)
* `contains`,`icontains`:包含关键字
* `in`,`gt`,`gte`,`lt`,`lte`: 在范围内, >,>=,<,<=
* `start/endwith`:以xxx开始或结尾的,同样可以使用`i`前缀忽略大小写
* `isnull`: 空值判断
* `regex`:正则

等等, 详细文档连线:[QuerySet API](https://docs.djangoproject.com/en/4.2/ref/models/querysets/#field-lookups)

#### 复合查询

可以向filter()传递多个关键字参数,以实现符合查询,如:

```python
# health > 50k and ap > 10k
query_set = Character.objects.filter(health__gt=50000,
                                         attack_power__gt=10000)

# 也可以写成
query_set = Character.objects.filter(health__gt=50000).filter(attack_power__gt=100000)
```

对应的SQL:

```sql
SELECT *
  FROM `characters_character`
 WHERE (`characters_character`.`attack_power` > 10000 AND `characters_character`.`health` > 50000)
```

#### 使用`Q`进行复合逻辑查询

django提供一个专用来处理查询条件的类`Q`, 每一个`Q`对象对应一个查询条件, 在多个`Q`对象之间可以使用逻辑运算符进行链接, 通常用来处理`OR`和`NOT` 关键字的查询, 如

```python
query_set = Character.objects.filter(Q(health__gt=50000) &
                                         Q(health__lt=70000))
```

对应的SQL

```sql
SELECT *
  FROM `characters_character`
 WHERE (`characters_character`.`health` > 50000 AND `characters_character`.`health` < 70000)
```

或者:

```python
query_set = Character.objects.filter(Q(attack_power__lt=50000) |
                                         ~Q(health__gt=50000))
```

对应的SQL:

```sql
SELECT *
  FROM `characters_character`
 WHERE (`characters_character`.`attack_power` < 50000 OR NOT (`characters_character`.`health` > 50000))
```

#### 使用F对象进行查询

F代表字段引用, 在查询条件为表中某个字段的值时使用, 如:

```python
query_set = Character.objects.filter(attack_power__gt=F('defense'))
```

对应的SQL:

```sql
SELECT *
  FROM `characters_character`
 WHERE `characters_character`.`attack_power` > (`characters_character`.`defense`)
```

在关联字段查询时经常用到`F`对象

### 排序

通过`order_by`方法来对结果集进行排序, 直接输入字段名为正序, 添加`-`为倒序, 并且可以依据多个字段进行排序, 如:

```python
# 基于等级倒序, 按名称正序排列
query_set = Character.objects.order_by('-level', 'name')
```

对应的SQL

```sql
SELECT *
  FROM `characters_character`
 ORDER BY `characters_character`.`level` DESC,
          `characters_character`.`name` ASC
```

`reverse()` 反转排序条件

### 分页

django中的分页十分方便, 直接使用python的切片语法即可, 如:

```python
def characters_ap_ranking_list(page, page_count=10):
    query_set = Character.objects.order_by('-attack_power')
    return query_set[page_count*(page-1):page_count * page]
```

对应的SQL:

```sql
SELECT *
  FROM `characters_character`
 ORDER BY `characters_character`.`attack_power` DESC
 LIMIT 10
OFFSET 20
```

### 指定数据与关联查询

通过`values()`方法可以指定要进行查询的字段, 而非获取所有字段的数据以提升效率, 如

```python
query_set = Character.objects.filter(id__lt=10).values('name', 'level')
```

对应SQL

```sql
SELECT `characters_character`.`name`,
       `characters_character`.`level`
  FROM `characters_character`
 WHERE `characters_character`.`id` < 10
```

同时可以进行关联查询, 在model1中存在外键关联到model2, 则可以直接通过改外键查询到关联表的字段值,如:

```python
# 其中 Character 模型存在一个外键, 对应 Club 类
# club = models.ForeignKey(Club, on_delete=models.SET_NULL, null=True)

query_set = Character.objects.filter(id__lt=10)
query_set = query_set.values('name', 'level', 'club__name')
```

对应SQL

```sql
SELECT `characters_character`.`name`,
       `characters_character`.`level`,
       `characters_club`.`name`
  FROM `characters_character`
  LEFT OUTER JOIN `characters_club`
    ON (`characters_character`.`club_id` = `characters_club`.`id`)
 WHERE `characters_character`.`id` < 10
```

`values()`指定的字段名将直接作为查询结果ORM对象的字段名, 查询打印结果如下:

```zsh
<QuerySet [
{'name': 'Alec Rois', 'level': 95, 'club__name': 'Harris-Towne'}, 
{'name': 'Fairlie Champagne', 'level': 219, 'club__name': 'Lesch-Jacobson'},
{'name': 'Gard Scargill', 'level': 218, 'club__name': 'Goldner-Kris'}, 
{'name': 'Bordie Domingues', 'level': 6, 'club__name': 'Harris-Towne'}, 
{'name': 'Renate Lamlin', 'level': 54, 'club__name': 'Huels, Wintheiser and Wisozk'}, 
{'name': 'Marsha Pellatt', 'level': 173, 'club__name': 'Skiles, Heathcote and Leannon'}, 
{'name': 'Luella Jakeway', 'level': 200, 'club__name': 'Skiles, Heathcote and Leannon'}, 
{'name': 'Gustavus Colliber', 'level': 80, 'club__name': 'Reinger LLC'}, 
{'name': 'Araldo Prosh', 'level': 193, 'club__name': 'Lesch-Jacobson'}]>
```

`values()` 方法返回的是词典集合, 而不是orm对象集合, 参数列表便是词典的`key`

`values_list()`: 方法则是返回包含全部值的元组, 即没有字段名, 仅有数据

** `only()`: 方法的效果与values()类似, 区别在于返回的不是词典, 而是模型对象, 也就具备后续操作的能力, 如果通过模型对象获取一个没有在only查询中列出的字段, 则会自动进行额外的数据库查询.

### distinct去重

通过`distinct()`方法可以去除重复的数据行, 如

```python
query_set = Character.objects.values('level').distinct().order_by('level')
```

对应的SQL

```sql
SELECT DISTINCT `characters_character`.`level`
  FROM `characters_character`
 ORDER BY `characters_character`.`level` ASC
```

去重操作可以用来确定一个范围, 比如:

```python
# 在人物表字段`club_id`上进行去重查询
query_set = Character.objects.values('club_id').distinct()
# 查询club表, 并以上述查询作为范围, 以获取包含任务的club
club_set = Club.objects.filter(id__in=query_set)
```

对应的SQL

```sql
SELECT *
  FROM `characters_club`
 WHERE `characters_club`.`id` IN (
        SELECT DISTINCT U0.`club_id`
          FROM `characters_character` U0
       )
```

### 预加载数据

django在获取orm对象时, 默认只加载当前表的数据, 对于外链表的数据并不会载入, 但是依然可以通过数据模型直接进行关联查询, 如

```python
# views.py
query_set = Character.objects.all()
```

在模板页面中通过orm对象获取关联表数据:

```html
# template.html
<td>{{obj.club.name}}</td>
```

此时, 代码不会出现任何问题, 但是会产生大量额外用来查询关联表的SQL

```sql
SELECT *
  FROM `characters_character`
        --由于关联查询而产生的额外sql
SELECT `characters_club`.`id`,
        --fields ...
  FROM `characters_club`
 WHERE `characters_club`.`id` = 26
 LIMIT 21
 
SELECT `characters_club`.`id`,
        --fields ...
  FROM `characters_club`
 WHERE `characters_club`.`id` = 5
 LIMIT 21
 
 --...
```

为了解决这一问题, 可以使用预加载

```python
query_set = Character.objects.select_related('club').all()
```

对应的SQL:

```sql
SELECT -- fields ...
       `characters_club`.`id`,
       `characters_club`.`name`,
        -- fields ...
  FROM `characters_character`
  LEFT OUTER JOIN `characters_club`
    ON (`characters_character`.`club_id` = `characters_club`.`id`)
```

如此渲染同样的模板便不会产生额外的sql查询了

`select_related`: 用于加载多一或者一对一的关系

`prefetch_related`:用于加载多对多

### 数据统计

通过`aggreate`方法来进行数据统计

```python
from django.db.models.aggregates import Count, Max, Min, Avg

result = Character.objects.aggregate(count=Count('name'),
                                         avg_level=Avg('level'),
                                         max_ap=Max('attack_power'),
                                         min_health=Min('health'))
```

对应的SQL

```sql
SELECT COUNT(`characters_character`.`name`) AS `count`,
       AVG(`characters_character`.`level`) AS `avg_level`,
       MAX(`characters_character`.`attack_power`) AS `max_ap`,
       MIN(`characters_character`.`health`) AS `min_health`
  FROM `characters_character`
```

### 添加注解字段

可以通过`annotate()`方法在结果集中加入自定义的字段,如

```python
from django.db.models import F, Value

# 注解字段的值不能使基本类型, 必须是表达式对象
# 可以是`Value`, `F`, `Func`, `Aggregate`
temp = Character.objects.filter(level=254).values(
    'id', 'name', 'level'
).annotate(
    max_level=Value(True)
).annotate(
    test=Value("test")
).annotate(
    club_owner_id=F('club__owner')
).annotate(
    club_owner_name=F('club__owner__name')
).annotate(
    ap_avg_dev=F('attack_power') - avg_attack_power['avg_ap']
)
```

对应的sql

```sql
SELECT `characters_character`.`id`,
       `characters_character`.`name`,
       `characters_character`.`level`,
       1 AS `max_level`,
       'test' AS `test`,
       `characters_club`.`owner_id` AS `club_owner_id`,
       T3.`name` AS `club_owner_name`,
       (`characters_character`.`attack_power` - 50049.574e0) AS `ap_avg_dev`
  FROM `characters_character`
  LEFT OUTER JOIN `characters_club`
    ON (`characters_character`.`club_id` = `characters_club`.`id`)
  LEFT OUTER JOIN `characters_character` T3
    ON (`characters_club`.`owner_id` = T3.`id`)
 WHERE `characters_character`.`level` = 254
```

## 添加数据

django的数据添加可以直接通过模型对象完成, 如:

```python
  p = Product()
  p.name = "some product"
  p.description = "description of this product..."
  p.price = 0.99
  p.save()
```

也可以通过模型类的构造函数或者使用create语句:

```python
Product.objects.create(name="p1",
                       description="description of this product",
                       price=0.99)
```

但是通过关键字参数创建数据会存在一些问题:

* 在编码时没有代码提示,需要纯手动键入, 容易出现错误
* 在通过重命名进行重构字段时, 关键字参数不会被重命名, 导致引发异常

因此建议在任何时候都实用对象赋值的方式进行数据添加

## 更新数据

与添加数据的操作类似, 仅需要通过`id`或者其他字段确定需要更新的数据, 然后修改其对应的数据并`save()`即可, 如:

```python
# 更新id位2的数据的name字段
p = Product(pk=2)
p.name = "update product name"
p.save()
```

但在实际运行中会出现问题, 因为p对象的其他字段的值皆为`None`, 如果表存在非空约束的字段, 那么会直接报错, 如果没有非空约束, 那么原始数据会被更新为空.

因此, 正确的更新方式应该是优先通过ORM获取原始数据, 以保证所有的字段数据都在ORM对象之中, 然后更新其中的数据, 代码如下:

```python
p = Product.objects.get(pk=2)
p.name = "update product name"
p.save()
```

对应的SQL

```sql
SELECT `playground_product`.`id`,
       `playground_product`.`name`,
       `playground_product`.`description`,
       `playground_product`.`price`,
       `playground_product`.`create_at`
  FROM `playground_product`
 WHERE `playground_product`.`id` = 2
 LIMIT 21

UPDATE `playground_product`
   SET `name` = 'update product name',
        -- 以下为数据库中的原始数据
       `description` = 'description of this product',
       `price` = 0.99,
       `create_at` = '2023-11-08 12:09:37.122549'
 WHERE `playground_product`.`id` = 2
```

为了避免额外进行一次数据库读取的操作,可以通过`update`方法直接进行更新

```python
Product.objects.filter(pk=2).update(name="new name by update")
```

对应的SQL

```sql
UPDATE `playground_product`
   SET `name` = 'new name by update'
 WHERE `playground_product`.`id` = 2
```

没有前置的数据读取, 也没有额外的数据写入, 但是仅有一个问题. 就是在对字段名进行重命名重构时关键字参数不会被重构, 从而存在出现bug的风险.

## 删除数据

直接通过ORM对象删除或者通过查询删除皆可, 如:

```python
Product(pk=1).delete()

Product.objects.filter(pk__lt=100).delete()
```

## 事务(Transection)管理

多条数据库操作一并执行, 要么全部成功, 要么全部失败

```python
from django.db import transaction

# 装饰函数的所有内容放入同一个事务中进行管理
@transaction.atomic()
def test_transaction():
    p1 = Product()
    p1.name = "new product"
    p1.description = "insert in transaction"
    p1.price = 1.2
    p1.save()

    p2 = Product(pk=3)
    p2.name = "new product"
    p2.description = 'already in database'
    p2.price = 5
    p2.save()
```

由于p2已经存在于数据库中, 相当于执行了update操作, 但是存在非空字段, 故而更新失败.

所以同属一个事务的p1也不会保存到数据库中

也可以对方法的部分内容进行事务管理, 如下:

```python
def test_transaction():
    # some code...
    with transaction.atomic():
        p1 = Product()
        p1.name = "new product"
        p1.description = "insert in transaction"
        p1.price = 1.2
        p1.save()

        p2 = Product(pk=3)
        p2.name = "new product"
        p2.description = 'already in database'
        p2.price = 5
        p2.save()
```

## 执行原生SQL

可以通过ORM对象直接调用原生SQL语句

```python
query_set = Product.objects.raw("select * from playground_product")
```

对应的SQL

```sql
select *
  from playground_product
```

对于简单的SQL语句完全没有必要, Django orm可以很好的生成这些语句

但是某些情况下, 需要处理的查询比较复杂, 通过orm编写较为困难,或者容易产生性能瓶颈, 便可以直接通过原生SQL语句进行优化.

对于执行原生SQL的情况, ORM模型并非是必须的环节, 可以直接通过数据库直连来执行, 如下:

```python
from django.db import connection

cursor = connection.cursor()
cursor.execute("select * from playground_product")
cursor.close()

# 由于cursor必须手动关闭, 通常结合with一同使用
with connection.cursor() as cursor:
    # 执行sql语句
    cursor.execute("select * from playground_product")
    # 执行存储过程
    cursor.callproc('get_product', [1, 2, 3])
```
