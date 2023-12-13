---
title: 20. 为REST API 添加过滤器, 搜索, 排序, 分页功能
date: 2023-11-13 19:53:59 +0800
categories: [笔记, Django]
tags: [python, django, backend]
---

通过`ProdcutViewSet`已经实现了对产品模型的增删改查操作, 但是在查询时的结果集则是数据库中的全部数据.

现在的`ProdcutViewSet`代码如下:

```python
class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def destroy(self, request, *args, **kwargs):
        product = self.get_object()
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

在此基础上要为其添加基于collection的过滤器, 则需要为`queryset`添加额外的逻辑, 这就需要重写`get_queryset`方法.

```python
def get_queryset(self):
    queryset = Product.objects.all()

    collection_id = self.request.query_params.get('collection_id')
    if collection_id:
        queryset = queryset.filter(collection_id=collection_id)

    return queryset
```

> 在重写了`get_queryset`方法后, 如果删除了`queryset`成员则必须修改`urls.py`中的配置
>
> `router.register('products', viewset_api.ProductViewSet, basename='products')`
>
> 增加`basename='products`参数, 作用是为路由指定前缀.
>
> 由于路由对象在生成接口时是一句`queryset`成员指定的模型名称来创建的, 即`Prodcut.object.all()`会自动生成`products`前缀.
>
> 在没有`queryset`成员的情况下就必须收订指定`basename`参数.
>
{: .prompt-info}

此时访问API: `store/producsts/?collection_id=3` 便可得到结果

```python
# HTTP 200 OK
# Allow: GET, POST, HEAD, OPTIONS
# Content-Type: application/json
# Vary: Accept

[
    {
        "id": 2,
        "title": "Island Oasis - Raspberry",
        "inventory": 40,
        "price": 84.64,
        "price_with_tax": 99.02879999999999,
        "collection_id": 3,
        "collection": "http://127.0.0.1:8000/store/collectons/3/"
    },
    # ...
]
```

## 通用过滤器

现在完成了针对`collection_id`的过滤, 但如果还需要其他的过滤字段或者其他过滤方式, 代码逻辑就会变得极其复杂.

可以通过使用`django-filter`来更好的处理接口过滤.

[Github](https://github.com/carltongibson/django-filter)
[Gitee镜像](https://gitee.com/mirrors_alex/django-filter)
[django-filter文档](https://django-filter.readthedocs.io/en/stable/)

## 安装 django-filter

```bash
pipenv install django-filter
```

在`INSTALLEDAPP`中加载过滤器

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # here
    'django_filters',
    # ...
]
```

## 配置过滤器

如果仅需要定值筛选, 那么通过简单的配置便可以直接完成

```python
from django_filters.rest_framework import DjangoFilterBackend

class ProductViewSet(ModelViewSet):

    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['collection_id']
```

再次访问接口: `store/products/?collection_id=3` 便可以顺利获取数据.

但是在对其他字段进行筛选时, 比如价格, 显然不能用确定价格额进行筛选, 而是要设定价格区间.

## 自定义过滤器对象

在`store`目录下创建一个`filters.py`文件, 代码如下:

```python
from django_filters.rest_framework import FilterSet

from .models import Product

class ProductFilter(FilterSet):

    class Meta:
        model = Product
        fields = {
            'collection_id': ['exact', 'in'],
            'unit_price': ['gt', 'lt', 'gte', 'lte'],
            'inventory': ['exact', 'gt', 'lt'],
            'title': ['icontains', 'istartswith', 'iendswith']
        }
```

通过定义过滤器类可以快速设定不同字段的过滤方式, 本质上就是对Django自身的lookup进行的引用.

详见[Django lookup 文档](https://docs.djangoproject.com/en/4.2/ref/models/querysets/#field-lookups)

在编写完过滤器对象后, 需要在ViewSet中进行对应的配置

```python
class ProductViewSet(ModelViewSet):

    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    filter_backends = [DjangoFilterBackend]
    filterset_class = ProductFilter
```

![preview](/assets/img/img_202311131814429529.png){: .right}

此时访问接口 `/store/products/`, 会自动提供一个`过滤器`按钮.

可以根据我们定义的过滤器类, 自动的创建一个过滤器表单, 在里面便可以直接根据可用的方式设置过滤数据, 如下图所示:

![filter](/assets/img/img_202311131817332685.png)
__过滤器表单__

提交表单, 我们得到的请求地址则变成了:

```text
GET /store/products/?collection_id=2&collection_id__in=&unit_price__gt=30&unit_price__lt=50&unit_price__gte=&unit_price__lte=&inventory=&inventory__gt=0&inventory__lt=&title__icontains=&title__istartswith=&title__iendswith=
```

## 添加搜索过滤器

搜索和过滤作用相似, REST Framework本身也提供了基础过滤器, 只是没有`Django-filter`强大, 但是相对而言搜索功能已经够用了.

在`ProductViewSet`中加入以下代码

```python
from rest_framework.filters import SearchFilter

class ProductViewSet(ModelViewSet):

    # code ...

    filter_backends = [DjangoFilterBackend, SearchFilter]
    search_fields = ['title', 'description']

    # code ...
```

重新访问接口, 便可以在过滤器按钮中找到搜索功能. 提交搜索关键字`bread mus`, 可以看到请求信息:

```text
/store/products/?search=bread+mus
```

## 数据排序

一个奇怪的命名和分组, 但是REST Framework确实把排序和搜索以及过滤器放倒了一起...

实现方式与搜索功能类似, 代码如下:

```python
from rest_framework.filters import SearchFilter, OrderingFilter

class ProductViewSet(ModelViewSet):

    # code ...

    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    search_fields = ['title', 'description']
    ordering_fields = ['unit_price', 'last_update']

    # code ...
```

通过过滤器表单请求按照更新时间倒序排列的数据, 可以看到请求信息为:

```text
GET /store/products/?ordering=-last_update
```

## 处理数据分页

同样利用REST Framework提供的分页类即可快速实现数据分页的功能.

```python
from rest_framework.pagination import PageNumberPagination
class ProductViewSet(ModelViewSet):

    # other code ...
    pagination_class = PageNumberPagination
```

为了确定每页数据的数量, 还需要在项目的`settings.py`文件中进行一个参数配置

```python
REST_FRAMEWORK = {
    'COERCE_DECIMAL_TO_STRING': False, # Decimal数字自动转化为字符串
    'PAGE_SIZE': 10 # 分页大小
}
```

重新访问接口 `/store/products/`, 可以看到结果集发生了变化:

![result](/assets/img/img_202311131941497659.png)

系统自动添加了`count`,`next`,`previous`数据, 同时原有的结果集被放倒了`results`之中.

> 在仅设置`PAGE_SIZE`的情况系下会在后台引发一个警告:
>
> > ```bash
> > ?: (rest_framework.W001) You have specified a default PAGE_SIZE pagination rest_framework setting, 
> > without specifying also a DEFAULT_PAGINATION_CLASS.
> > HINT: The default for DEFAULT_PAGINATION_CLASS is None. In previous versions this was PageNumberPagination. 
> > If you wish to define PAGE_SIZE globally whilst defining pagination_class on a per-view basis you may silence this check.
> > ```
>
> 原因在于定义了全局的`PAGE_SIZE`但是没有定义全局的分页类.
> 由于并不是所有数据都需要分页, 且不同app的分页规则同样不同, 可以通过自定义分页类, 并定义其成员变量的方式避免全局配置:
>
> ```python
> from rest_framework.pagination import PageNumberPagination
> 
> class DefaultPagination(PageNumberPagination):
>   page_size = 10
> 
> ```
>
> 当进行了分页类的自定义后, 即可以添加其他的逻辑, 也可以将分页大小等数据放到配置文件里. 最后用该类替换之前的分页类即可.
>
{: .prompt-tip}
