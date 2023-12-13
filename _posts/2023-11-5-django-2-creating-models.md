---
title: 2. 创建基础的模型类
date: 2023-11-5 01:57:59 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

一个类就是一个表

django会自动添加主键, 多数情况下不需要自己设置主键

```python
from django.db import models

# Create your models here.


class Product(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
    price = models.DecimalField(max_digits=6, decimal_places=2)
    inventory = models.IntegerField()
    last_updated = models.DateTimeField(auto_now=True)


class Customer(models.Model):
    first_name = models.CharField(max_length=255)
    last_name = models.CharField(max_length=255)
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=255)
    birth_date = models.DateField(null=True)

```

## 选项字段

django通过选项字段来处理枚举类型, 对该选项进行定义

```python
class Customer(models.Model):

    MEMBERSHIP_BRONZE = 'B'
    MEMBERSHIP_SILVER = 'S'
    MEMBERSHIP_GOLD = 'G'

    MEMBERSHIP_CHOICES = [
        (MEMBERSHIP_BRONZE, 'Bronze'),
        (MEMBERSHIP_BRONZE, 'Silver'),
        (MEMBERSHIP_GOLD, 'Gold')
    ]
    first_name = models.CharField(max_length=255)
    last_name = models.CharField(max_length=255)
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=255)
    birth_date = models.DateField(null=True)
    membership = models.CharField(max_length=1,
                                  choices=MEMBERSHIP_CHOICES,
                                  default=MEMBERSHIP_BRONZE)

```

## 解决循环引用

![image-20231109160342949](/assets/img/image-20231109160342949.png)

在某些需求中, 可能存在两个表之间的相互外键引用, 如上图所示:

* 多个Product可以隶属于一个Collection
* 每个Collection最多只能有一款主打产品,

于是就产生了Product与Collection之间相互引用的问题

* Product (m) -- (1) Collection
* Product (0,1) -- (1) Collection

在models定义的过程中, 由于代码是从上到下编译的, 所以就会出现一个引用了未定义类型的问题.

```python
class Collection(models.Model):
    title = models.CharField(max_length=255)
    # 由于Product尚未定义, 此时代码无法编译
    #   featured_product = models.ForeignKey(
    #   Product, on_delete=models.SET_NULL, null=True)
    # 通过字符串来传递模型类的名称, 便可以解决这个问题
    featured_product = models.ForeignKey(
        'Product', on_delete=models.SET_NULL, null=True, related_name='+')
    # other fields
    
    # !!!但是通过该方法, 虽然能够解决数据模型的相互依赖问题, 但是如果在后续维护时对Product进行重名经操作, 上面的字符串是不会同时修改的, 也就有可能引起其他异常
    
class Product(models.Model):
    title = models.CharField(max_length=255)
    collection = models.ForeignKey(Collection, on_delete=models.PROTECT)
    # other fields
```

> 总的来说, 循环引用的问题并非无法解决, 但是在进行数据库设计的时候应当尽力避免
{: .prompt-warning}
