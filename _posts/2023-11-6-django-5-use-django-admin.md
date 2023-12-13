---
title: 5. Django-admin后台管理
date: 2023-11-6 02:05:37 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

有django提供的一个数据管理后台, 功能强大, 熟练使用可以节省大量的编码时间
[Django Admin完整官方文档](https://docs.djangoproject.com/en/dev/ref/contrib/admin/)

## 基本使用

可以通过`admin/`进行访问, 如

```url
http://127.0.0.1:8000/admin
```

在使用admin之前, 需要创建admin用户, 在终端执行:

```zsh
python manage.py createsuperuser
```

输入用户名密码即可.

忘记密码时可以通过终端命令直接修改

```zsh
python manage.py changepassword [username]
```

### 修改后台页面标题

```python
from django.contrib import admin

# 可以直接在项目的urls.py中添加, 但不推荐
admin.site.site_header = 'My site to learn django'
admin.site.index_title = "Site Apps"
```

![image-20231108212229095](/assets/img/image-20231108212229095.png){: .shadow}

## 注册模块

要通过admin后台管理某个app的数据, 需要先对其进行注册, 如`playground`模块的`Product`对象

```python
# `playground/admin.py`
from django.contrib import admin
from . import models

admin.site.register(models.Product)
```

## 修改显示字段和表排序方式

```python
class Product(models.Model):
    title = models.CharField(max_length=255)
    # other fields ...

    def __str__(self) -> str:
        return str(self.title)

    class Meta:
        ordering = ['title']
```

## 自定义管理后台

通过对ORM对象的设置完成自定义的后台管理

```python
# `playground/admin.py`
from django.contrib import admin
from . import models

# 如果不使用装饰器可以使用代码进行注册
# admin.site.register(models.Product, ProductAdmin)
# 只能注册一次, 即装饰器和代码注册只能二选一
@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ["name", 'price', 'description']
    list_editable = ['price']
    list_per_page = 20
    
    # ---添加计算字段---
    # 装饰器定义字段的排序方式
    @admin.display(ordering='price')
    def price_level(self, product):
        if product.price < 10:
            return "cheap"
        return 'expensive'
    
    # 添加一个计算库存总价值的字段, 该字段同样是计算字段, 但是其数据基于当前数据行
    # 因此便需要通过重写父类的`get_queryset`方法来修改数据集
    @admin.display(ordering=F('price')*F('inventory'))
    def total_price(self, product):
        return product.total_price

    # admin模块载入orm对象时的查询方法
    # 与`Model.objects`中获取的数据集操作方式完全一致
    # 可以通过annotate添加计算字段, 外链字段等一系列操作
    def get_queryset(self, request: HttpRequest) -> QuerySet[Any]:
        return super().get_queryset(request) \
            .annotate(total_price=F('price') * F('inventory'))
```

### 外键字段的显示

在Product模型中, 存在一个`collection`外键

```python
class Product(models.Model):
    # ...
    collection = models.ForeignKey(Collection, on_delete=models.PROTECT)
```

而`Collection`模型中设置了该模型的`__str__`方法

```python
class Collection(models.Model):
    title = models.CharField(max_length=255)
    # ...
    def __str__(self) -> str:
        return str(self.title)
```

那么此时,  如果在ProductAdmin中设置显示该外键字段, 那么便会直接显示其`title`值, 对应的SQL如下:

```sql
SELECT `store_product`.`id`,
       `store_product`.`title`,
       `store_product`.`slug`,
       `store_product`.`description`,
       `store_product`.`unit_price`,
       `store_product`.`inventory`,
       `store_product`.`last_update`,
       `store_product`.`collection_id`,
       `store_collection`.`id`,
       `store_collection`.`title`,
       `store_collection`.`featured_product_id`
  FROM `store_product`
 INNER JOIN `store_collection`
    ON (`store_product`.`collection_id` = `store_collection`.`id`)
 ORDER BY `store_product`.`title` ASC,
          `store_product`.`id` DESC
```

此时对于外键查询并不会产生额外的sql查询

但如果查询Collection的其他字段, 即通过orm对象进行关联查询, 此处依然用title举例:

```python
class ProductAdmin(admin.ModelAdmin):
    list_display = ['title', 'unit_price',
                        'inventory_status', 'collection_title']

    def collection_title(self, product):
        return product.collection.title
```

便会因为关联查询而产生额外的sql:

```sql
SELECT `store_collection`.`id`,
       `store_collection`.`title`,
       `store_collection`.`featured_product_id`
  FROM `store_collection`
 WHERE `store_collection`.`id` = 5
 LIMIT 21
 
 SELECT `store_collection`.`id`,
       `store_collection`.`title`,
       `store_collection`.`featured_product_id`
  FROM `store_collection`
 WHERE `store_collection`.`id` = 4
 LIMIT 21
 
 -- ...
```

具体产生的sql数量与当前页的数据条目相关

为了避免额外的sql查询, 可以直接使用关联查询来进行处理, 只需要在`list_select_related`成员中设置需要关联查询的表, 便可以在显示字段中添加任意关联表的字段.

```python
class ProductAdmin(admin.ModelAdmin):
    list_display = ['title', 'unit_price',
                    'inventory_status', 'collection_title']
    list_editable = ['unit_price']
    list_per_page = 10
    
    # 设置关联查询表
    list_select_related = ['collection']
```

对应的SQL:

```sql
SELECT `store_product`.`id`,
       `store_product`.`title`,
       `store_product`.`slug`,
       `store_product`.`description`,
       `store_product`.`unit_price`,
       `store_product`.`inventory`,
       `store_product`.`last_update`,
       `store_product`.`collection_id`,
       `store_collection`.`id`,
       `store_collection`.`title`,
       `store_collection`.`featured_product_id`
  FROM `store_product`
 INNER JOIN `store_collection`
    ON (`store_product`.`collection_id` = `store_collection`.`id`)
 ORDER BY `store_product`.`title` ASC,
          `store_product`.`id` DESC
```

### 添加超链接

在admin中位某一个字段的值添加超链接

```python
from django.utils.html import format_html, urlencode
from django.urls import reverse

@admin.register(models.Collection)
class CollectionAdmin(admin.ModelAdmin):
    list_display = ['title', 'products_count']

    @admin.display(ordering='products_count')
    def products_count(self, collecton):
        url = (reverse('admin:store_product_changelist')
               + '?'
               + urlencode({
                   'collection__id': str(collecton.id)
               }))

        return format_html('<a href="{}">{}</a>',
                           url,
                           collecton.products_count)

    def get_queryset(self, request: HttpRequest) -> QuerySet[Any]:
        return super().get_queryset(request)\
            .annotate(products_count=Count('product'))
```

💡 reverse函数用于动态获取页面的路由地址, 之后再说...

> 在上面的代码里, `get_queryset`方法中通过`Count`统计了`product`字段
>
> ```python
>     def get_queryset(self, request: HttpRequest) -> QuerySet[Any]:
>         return super().get_queryset(request)\
>             .annotate(products_count=Count('product'))
> ```
>
> 而`Collection`模型的定义如下:
>
> ```python
> class Collection(models.Model):
>     title = models.CharField(max_length=255)
>     featured_product = models.ForeignKey(
>         'Product', on_delete=models.SET_NULL, null=True, related_name='+')
> 
>     def __str__(self) -> str:
>         return str(self.title)
> 
>     class Meta:
>         ordering = ['title']
> ```
>
> 显然, 在Collection中并没有product字段, 而代码运行没有问题.
>
> 这是由于django在定义数据模型时, 会位外键字段反向创建一个关系字段, 这一字段应该是创建在python对象中, 所以数据库中并不存在, 来看Product类的定义:
>
> ```python
> class Product(models.Model):
>     title = models.CharField(max_length=255)
>     slug = models.SlugField()
>     description = models.TextField()
>     unit_price = models.DecimalField(max_digits=6, decimal_places=2)
>     inventory = models.IntegerField()
>     last_update = models.DateTimeField(auto_now=True)
>     ## 这里
>     collection = models.ForeignKey(Collection, on_delete=models.PROTECT)
>     promotions = models.ManyToManyField(Promotion)
> 
>     def __str__(self) -> str:
>         return str(self.title)
> ```
>
> 正是因为`collection`字段是到`Collection`模型的外键字段, 因此在Collection中也就自动添加了`product`字段,
>
> 于是也就能够直接通过django orm直接在`Collection`对象中使用`Count("product")`操作了
>
> 再看`Collection`对象中的代码
>
> ```python
>   featured_product = models.ForeignKey(
>       'Product', on_delete=models.SET_NULL, null=True, related_name='+')
> ```
>
> 这里用了一个关键字参数, `related_name='+'`
>
> 其意思便是在Product类中不创建反向字段, 事实上, 在本例中如果没有这个关键字参数会触发编译器错误
>
> 因为Product在Collection中创创建了product关系字段,
>
> 同时Collection也在Product中创建了collection关系字段
>
> 而Product自己已经有了一个同名的字段, 也就出现了命名冲突, 只要改掉其中一个名字即可.
>
> 同样的, 如果修改`Product`类的代码为:
>
> ```python
> collection = models.ForeignKey(
>         Collection, on_delete=models.PROTECT, related_name='+')
> ```
>
> 也就是不在Collection对象中创建关系字段, 在访问页面时就会直接出错
>
> ```zsh
> django.core.exceptions.FieldError: Cannot resolve keyword 'product' into field. Choices are: featured_product, featured_product_id, id, title
> ```

### 添加搜索

在django admin中添加基础的搜索条功能

```python
search_fields = ['first_name__istartswith', 'last_name__istartswith']
```

搜索条会在指定的字段中, 基于指定的lookup方式进行数据搜索

### 添加过滤器

基于当前表的某个字段进行过滤, 可以直接调用django提供的默认过滤器

```python
list_filter = ['collection', 'last_update']
```

也可以使用自定义过滤器, 过滤器本质类似于html中的单选框, 有一个id和一个value, id用于唯一标识, value用于UI显示

```python
class InventoryFilter(admin.SimpleListFilter):
    # 显示在过滤器列表中的名称
    title = 'Inventory'
    # 通过URL传递的参数名称
    parameter_name = 'inven'
    # 需要返回一个tupule list, 每一个tuple是过滤器的一个显示项, tuple的第一个值会作为过滤器的value, 用于判断过滤器选中状态, 第二个值用于UI显示
    def lookups(self, request: Any, model_admin: Any) -> list[tuple[Any, str]]:
        return [
            ('<10', 'Low')
        ]

    def queryset(self, request: Any, queryset: QuerySet[Any]) -> QuerySet[Any] | None:
        # 如果当前过滤器选项是 `<10`
        if self.value() == '<10':
            # 对当前页面的queryset进行过滤, 与ORM语法完全一样
            return queryset.filter(inventory__lt=10)
        return None
```

并在过滤器列表中添加自定义的过滤器即可

```python
list_filter = ['collection', 'last_update', InventoryFilter]
```

### 自定义用户操作

在django admin中添加对数据的批量操作方式

```python
from django.contrib import admin, messages

from . import models

@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):
    # ...code...
    actions = ['clear_inventory']

    @admin.action(description='Clear inventory')
    def clear_inventory(self, request, queryset):
        updated__count = queryset.update(inventory=0)
        self.message_user(
            request,
            f'{updated__count} products were successfully updated.',
            messages.SUCCESS # 可选
        )
```

### 自定义操作表单

对于数据的创建或修改的详情页面表单:

![image-20231109181130563](/assets/img/image-20231109181130563.png)

可以在对应的模型Admin类中对其进行定义, 如:

* `fields = ['title', 'slug']`: 仅显示列表中的字段

* `exclude = ['promotions']`: 不显示的字段

* `readonly_fields = ['slug']`: 只读字段

* `prepopulated_field = {'slug': ['title']}`:  自动填充字段, 代表slug在空白状态下会根据title的内容进行自动填充, 如果手动更改过则不会变更,  实测没什么效果, 感觉意义不大

* `autocomplete_fields = ['collection']`: 自动完成字段, 下拉列表的变种形式, 对于数量太多的下拉列表, 常规形式会严重影响用户操作体验, 自动完成字段则是在下拉列表上方增加一个文本输入框, 实现类似编码自动补全的效果.

  > 需要在`Collection`模型中设置`search_filds`成员以配合使用, 否则会出现编译错误
  >
  > ```bash
  > django.core.management.base.SystemCheckError: SystemCheckError: System check identified some issues:
  > 
  > ERRORS:
  > <class 'store.admin.ProductAdmin'>: (admin.E040) CollectionAdmin must define "search_fields", because it's referenced by ProductAdmin.autocomplete_fields.
  > ```

### 内联表

在多对多的关系中通常会使用一些链接表, 比如一个`Order`可以包含多个`Product`,一个`Product`可以出现在多个`Order`中, 所以每一个订单记录都需要知道它是哪一个订单的哪一个产品的信息

这里定义了一个`OrderItem`表

```python
class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.PROTECT)
    product = models.ForeignKey(Product, on_delete=models.PROTECT)
    quantity = models.PositiveSmallIntegerField()
    unit_price = models.DecimalField(max_digits=6, decimal_places=2)
```

但是在django admin中去单独设计该表的admin页面没有任何实际意义, 因为他必须依托于`Order`来进行呈现, 所以通常会以内联表的形式进行操作

```python
class OrderItemInline(admin.TabularInline):
    model = models.OrderItem
    autocomplete_fields = ['product']
    # 内联表额外的数据行
    extra = 0
    # 最小数据行
    min_num = 1
    # 实际数据行 = 最小 + 额外
    
@admin.register(models.Order)
class OrderAdmin(admin.ModelAdmin):
    list_display = ['id', 'placed_at', 'customer']
    ordering = ['id']
    list_per_page = LIST_PER_PAGE
    inlines = [OrderItemInline]
    autocomplete_fields = ['customer']
```

内联样式有两种:

* admin.StackedInline: 以数据表单的形式呈现
* admin.TabularInline: 以数据表的形式呈现, 每条数据呈一行, 比较常用
