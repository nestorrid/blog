---
title: 16. REST Framework GenericView 通用视图
date: 2023-11-12 20:26:04 +0800
categories: [笔记, Django]
tags: [python, django, backend]
---

在之前的代码中, 基本实现了对于`Prodcut`和`Collection`的增删改查接口.

但是在API的实现代码中存在着大量几乎相同的代码, 如:

```python
# ProductList
def get(self, request):
        queryset = Product.objects.all()
        serializer = ProductSerializer(queryset, many=True,
                                       context={'request': request})
        return Response(serializer.data)

# CollectionList
def get(self, request: Request):
        queryset = Collection.objects.all().annotate(products_count=Count('products'))
        serializer = CollectionSerializer(queryset, many=True)
        return Response(serializer.data)
```

他们本质上是相同的功能, 只是处理的目标对象不同, 而编码模式也是几乎一样.

为了减少这种类似的重复编码, 就可以使用`rest Framework`所提供的`generic view`来实现

## 通用视图

rest framework提供了多种通用视图, 统一封装在 `rest_framework.generic` 包下.

[详细官方文档](https://www.django-rest-framework.org/api-guide/generic-views/#generic-views)

以`ProductList`视图为例, 由于接口能够处理`GET`和`POST`请求, 同时返回的是一个结果集, 就可以直接使用`ListCreateAPIView`来快速实现:

```python
class ProductList(generics.ListCreateAPIView):

    serializer_class = ProductSerializer
    queryset = Product.objects.select_related('collection').all()
```

> 注意: `serializer_class`的值是类而不是对象, 不能加`()`
{: .prompt-warning}

由于`products/`路由在查询时需要查询关联的`collection`, 所以需要重写`get_queryset`方法来完善代码逻辑.

如果在查询时存在其他的业务需求, 比如根据用户类型和权限进行不同的查询等等, 则需要重写`get_queryset`方法:

```python
def get_queryset(self):
        # logic code here
        queryset = Model.objects.all()
        return queryset
```

`serializer_class`同理, 可以通过重写`get_serializer_class`方法添加额外业务处理.

`get_serializer_cotext`方法指定了序列化的上下文, 如在进行超链接关系字段的序列化时就需要用到上下文中的`request`, 但是`GenericView`已经对其进行了更为完善的封装, 在仅需要设定`request`上下文的情况下无需进行更改. 查看父类的`get_serializer_context`可知:

```python

# GenericAPIView

def get_serializer_context(self):
        """
        Extra context provided to the serializer class.
        """
        return {
                'request': self.request,
                'format': self.format_kwarg,
                'view': self
}
```

通过GenericView, 之前的视图方法被压缩到仅仅两行, 除此之外, GenericView还会为`POST`请求自动生成一个表单, 如图所示:
![form](/assets/img/img_202311122320571347.png)

对于这类接口, 甚至仅需要做一下配置便可以快速完成, 不需要额外代码:

```python
# `store/urls.py`
urlpatterns = [
    path('products/', generics.ListCreateAPIView.as_view(
        queryset=Product.objects.select_related('collection').all(),
        serializer_class=ProductSerializer
    )),
    # other routes...
]
```

> 虽然可以通过路由配置文件直接实现通用视图接口, 但不建议这么使用.
>
> 1. 后续可能会增加业务逻辑, 一旦增加那么还是要提取到单独的视图类中
> 2. 配置文件中会引入逻辑代码, 以及额外的导入似的后期不便于维护
>
> 仅当临时测试新建接口时用一用就可以了.
>
{: .prompt-tip}
