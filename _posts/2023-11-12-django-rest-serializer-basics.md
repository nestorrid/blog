---
title: 11. Django Rest Framework 序列化
date: 2023-11-12 00:02:12 +08000
categories: [笔记, Django]
tags: [python, django, backend, REST]
---

REST Framework提供了一个`JSONRenderer`对象, 可以将python的词典对象序列化为json对象.

同时提供了一个`Serializer`对象, 用于将models转化为python词典.

在`store`app中创建一个`serializer.py`文件

```python
from rest_framework import serializers


class ProductSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    title = serializers.CharField(max_length=255)
    unit_price = serializers.DecimalField(max_digits=6,decimal_places=2)
```

> 使用序列化对象, 而非直接通过模型对象获取数据, 好处在于:
>
> * 简化序列化代码, 并使序列化方式统一, 可以在任意位置通过相同的方式一次性完成序列化, 而不是对模型对象进行零散的维护
> * 隐藏不需要显示的字段
{: .prompt-tip}

在完成了序列化对象的设置之后, 便可以修改视图函数的代码:

```python
@api_view()  # 该装饰器替换默认的request对象为djangorestframework的request对象
def product_list(request):

    queryset = Product.objects.all()
    serializer = ProductSerializer(queryset, many=True)

    # 使用djangorestframwork的Response返回数据
    return Response(serializer.data)


@api_view()
def product_detail(request, product_id):
    product = get_object_or_404(Product, pk=product_id)
    serializer = ProductSerializer(product)
    return Response(serializer.data)
```

此时通过URL访问视图便可以得到具体的数据了

![result](/assets/img/img_202311111749462365.png)

> django rest framework 会自动把Decimal类型的数据转换为字符串, 如果不希望转换, 可以在`project/settings.py`文件中加入一下设置:
>
> ```python
> REST_FRAMEWORK = {
>     'COERCE_DECIMAL_TO_STRING': False
> }
> ```
>
{: .prompt-tip}

## 自定义的序列化字段

序列化对象的目的在于确定接口的呈现形式, 以此来与后台的数据模型解耦.
从命名规则上来说, 数据库字段的名称与前端需要的字段名称可能不同, 通过序列化对象便可以轻松的解决这一问题.
同时, 后台的数据模型可能会做各种类型的修改, 但是这些修改只关乎后台, 而无需让用户知道.
通过自定义序列化字段便可以实现这一目的, 比如:

* 增加模型中不存在的字段
* 更改字段的名称
* etc

假设需要再接口中增加一个含税价格字段, 可以使用如下代码:

```python
from decimal import Decimal
from rest_framework import serializers
from store.models import Product


class ProductSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    title = serializers.CharField(max_length=255)
    price = serializers.DecimalField(max_digits=6,
                                     decimal_places=2,
                                     source="unit_price")

    price_with_tax = serializers.SerializerMethodField(
        method_name="calc_price_with_tax")

    def calc_price_with_tax(self, product: Product):
        return product.unit_price * Decimal(1.17)

```

## 关联查询的序列化

Product和Collection之间存在多对一的关系, 在Product中通过外键`collection`进行链接.
那么在序列化Product时便可以将collection的数据同时序列化.

### PrimaryKeyRelatedField

```python
collection = serializers.PrimaryKeyRelatedField(
        queryset=Collection.objects.all()
    )
```

> 根据官方文档说明
>
> ```text
> `queryset` - 
> The queryset used for model instance lookups when validating the field input. 
> Relationships must either set a queryset explicitly, or set read_only=True.
> ```
>
> 该参数是用来对于输入数据进行验证的, 应该是说反序列化时进行数据库查重的范围. 所以, 如果该字段不需要修改, 仅仅是读取内容, 也可以用参数`read_only=True`替代.
> [相关文档](https://www.django-rest-framework.org/api-guide/relations/#primarykeyrelatedfield)
>
{: .prompt-tip}

### StringRelatedField

如果希望关联查询出的不仅是id, 比如Collection的`title`, 可以写成

```python
collection_title = serializers.StringRelatedField(source='collection')
```

> 由于`collection`成员已经被定义为了`PrimaryKeyRelatedField`, 所以使用了`collection_title`作为成员名
> 但是`Collection`对象中并没有与该成员同名的字段, 所以必须显示的声明其`source`为`collection`.
>
{: .prompt-info}

但此时运行程序, 会出现一个严重问题, 为了获取每个Product对应的Collection标题, 额外执行了与Product数量相同的SQL查询
![result](/assets/img/img_202311112225443281.png)

> 原因在于Product在查询时并没有同步查询Collection的其他字段, 所以在序列化的时候才会根据每一个Product重新查询其对应的collection的`title`
{: .prompt-danger}

为了解决直接以问题, 需要在视图方法中加入对`Collection`的关联查询:

```python
@api_view()
def product_list(request):
    queryset = Product.objects.select_related('collection').all()
    serializer = ProductSerializer(queryset, many=True)
    return Response(serializer.data)
```

### Nested Serializer

Product有自己的序列化对象, Collection同样可以有自己的序列化对象.

```python
class CollectionSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    title = serializers.CharField(max_length=255)
```

添加一个简单的序列化对象, 同时将ProductSerializer的collection成员修改为:

```python
class ProductSerializer(serializers.Serializer):
    # ... code...
    collection = CollectionSerializer()
```

![result](/assets/img/img_202311112240579548.png)

> 需要注意的是, 此种方式同样需要对collection进行关联查询, 否则也会出现额外的SQL查询
{: .prompt-warning}

### HyperlinkedRelatedField

当不需要直接显示关联表的内容, 而是要给出查询关联表内容的链接地址时, 就需要用到链接序列化.

1. 修改collection成员的类型为:

    ```python
    collection = serializers.HyperlinkedRelatedField(
            read_only=True,
            view_name='collection-detail'
        )
    ```

    > 与`PrimaryKeyRelatedField`字段类似, 如果不需要对该字段进行修改, 可以直接使用`read_only=True`, 否则需要提供一个`queryset`.
    {: .prompt-info}

2. 在`store/views.py`中创建对应的视图方法:

    ```python
    @api_view()
    def collection_detail(request, pk):
        collection = get_object_or_404(Collection, pk=pk)
        serializer = CollectionSerializer(collection)
        return Response(serializer.data)
    ```

3. 在`store/urls.py`中配置对应的路由:

    ```python
    urlpatterns = [
        #...
        path('products/', views.product_list),
        path('collections/<int:pk>',
            views.collection_detail,
            name='collection-detail')
    ]
    ```

4. 设置`product_detail`方法的上下文参数:

    ```python
    @api_view()  # 该装饰器替换默认的request对象为djangorestframework的request对象
    def product_list(request):

        queryset = Product.objects.select_related('collection').all()
        serializer = ProductSerializer(
            queryset, many=True, context={"request": request})

        # 使用djangorestframwork的Response返回数据
        return Response(serializer.data)
    ```

> ![result](/assets/img/img_202311112310499438.png){: .right}
> **运行结果:**
>
> 可以看到, 在运行结果中列出的是一个完整的url, 而起对应的便是上面配置的视图方法
>
> 由于collection作为Product的外联字段, 所以即便没有进行`select_relation()`关联查询, 也不会产生额外的查询SQL.
>
{: .prompt-info}

> 需要注意的是, 在进行链接字段序列化时, 视图方法的形参名字只能是`pk`, 在其他代码不做修改, 仅修改形参名便会出现运行异常
>
> ```python
> # Could not resolve URL for hyperlinked relationship using view name "collection-detail". 
> # You may have failed to include the related model in your API, 
> # or incorrectly configured the `lookup_field` attribute on this field.
> ```
>
{: .prompt-warning}

> 超链接字段的使用前提是该字段本身必须是模型中的一个关系型字段, 如果只是单纯的序列化对象中定义的字段那么不会有任何结果!
{: .prompt-danger}
