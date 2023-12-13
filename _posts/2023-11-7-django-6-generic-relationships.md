---
title: 6. 通用关系查询
date: 2023-11-7 02:17:16 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

当编写一个tags模块, 用于记录某一项数据是否被标记了

没有必要专门为每一种类型的记录专门定义一个tags模块, 而是可以通过一个通用的tags模块来完成

即视频, 文章, 产品, 等等全部由该模块来进行标记记录, 即便这些数据分属于不同的子app

此时便可以通过`generic relationships`来完成

这一功能基于django的`ContentType`模块

```python
INSTALLED_APPS = [
    # ...
    'django.contrib.contenttypes',
    # ...
]
```

django会为所有的model生成一个唯一的content_type, 储存在`django_content_type`表中

要获取特定model的content_type

```python
from django.shortcuts import render
from django.contrib.contenttypes.models import ContentType

def index(request):
    # 基于模型类直接获取其content type
    content_type = ContentType.objects.get_for_model(Product)
    return render(request, 'tags.html')

```

对应的SQL

```sql
SELECT `django_content_type`.`id`,
       `django_content_type`.`app_label`,
       `django_content_type`.`model`
  FROM `django_content_type`
 WHERE (`django_content_type`.`app_label` = 'store' AND `django_content_type`.`model` = 'product')
 LIMIT 21
```

通过`content_type`进行数据查询

```python
query_set = TaggedItem.objects\
        .select_related('tag')\
        .filter(content_type=content_type, object_id=1)
```
