---
title: 7. 自定义模型管理器
date: 2023-11-7 02:18:11 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

模型管理器是在调用某个模型的`objects`方法时返回的对象, 如:

```python
manager = Product.objects
```

在 [6. 通用关系查询.md](6. 通用关系查询.md) 中, 封装了通过content-type查询被标记的数据的代码

```python
from django.shortcuts import render
from django.contrib.contenttypes.models import ContentType
from store.models import Product
from tags.models import TaggedItem

def index(request):
    content_type = ContentType.objects.get_for_model(Product)
    query_set = TaggedItem.objects\
        .select_related('tag')\
        .filter(content_type=content_type, object_id=1)

    return render(request, 'tags.html')

```

但如果每次添加一个查询功能都要封装这些代码显然并不合适, 理想状态下的代码应该是

```python
TaggedItem.objects.get_tags_from(Product, 1)
```

此时便需要设置自定义模型管理器

```python
class TaggedItemManager(models.Manager):
    def get_tags_for(self, obj_type, obj_id):
        content_type = ContentType.objects.get_for_model(obj_type)

        return TaggedItem.objects \
            .select_related('tag') \
            .filter(
                content_type=content_type,
                object_id=obj_id
            )

class TaggedItem(models.Model):
    # 修改模型的管理器对象
    objects = TaggedItemManager()
    tag = models.ForeignKey(Tag, on_delete=models.CASCADE)
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey()
```
