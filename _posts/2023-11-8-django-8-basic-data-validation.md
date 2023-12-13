---
title: 8. Django 数据校验
date: 2023-11-8 02:18:51 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

[Django 数据校验的文档](https://docs.djangoproject.com/en/4.2/ref/validators/)

在定义ORM模型时便可直接为字段添加数据校验

```python
from django.core.validators import MinValueValidator
class Product(models.Model):
    title = models.CharField(max_length=255)
    slug = models.SlugField()
    # null表示在数据库中可以为空, 而表单依然要求输入内容
    # blank则表示表单输入可以为空
    description = models.TextField(null=True, blank=True)
    unit_price = models.DecimalField(max_digits=6,
                                     decimal_places=2,
                                     validators=[MinValueValidator(1, message='最小值为1')])
```
