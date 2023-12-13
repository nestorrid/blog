---
title: 9. 为已有的Django app制作插件
date: 2023-11-8 02:19:28 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

可以通过类似插件的方式将彼此时间相互独立的app通过contenttype进行结合使用

1. 创建插件app

   ```python
   python manage.py startapp store_plugin 
   ```

2. 编辑`store_plugin/admin.py`

   ```python
   from django.contrib import admin
   from django.contrib.contenttypes.admin import GenericTabularInline
   
   from store.admin import ProductAdmin
   from store.models import Product
   from tags.admin import TagAdmin
   from tags.models import TaggedItem
   
   
   class TagInlin(GenericTabularInline):
       """
       创建`TaggedItem`模型的内联插件
       """
       autocomplete_fields = ['tag']
       model = TaggedItem
       extra = 0
   
   
   class ProdcutPluginAdmin(ProductAdmin):
       """
       继承`store`模块的ProductAdmin, 为其添加内联插件
       """
       inlines = [TagInlin]
   
   # 注销原本的Product
   admin.site.unregister(Product)
   # 通过插件重新注册
   admin.site.register(Product, ProdcutPluginAdmin)
   
   ```

3. 载入插件

   ```python
   INSTALLED_APPS = [
       'store',
       'store_plugin',
       'tags',
   ]
   # 如此一来可以随时通过Installed_apps打开或者关闭Tag功能
   ```
