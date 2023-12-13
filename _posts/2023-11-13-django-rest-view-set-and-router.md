---
title: 18. Django rest framework 视图集
date: 2023-11-13 02:52:50 +0800
categories: [笔记, Django]
tags: [python, django, backend, REST]
---

回顾之前的内容, 通过通用视图对接口进行了简化, 最终得到了代码:

```python
class ProductList(generics.ListCreateAPIView):

    serializer_class = ProductSerializer
    queryset = Product.objects.select_related('collection').all()


class ProductDetail(generics.RetrieveUpdateDestroyAPIView):

    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def delete(self, request, *args, **kwargs):
        product = self.get_object()
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

这两个类相较最初的视图方法和视图类已经进行了极大的简化, 但是任然存在重复的代码或者模式.

事实上, `select_related('collection')`也仅仅只是为了测试字符串关联查询而临时加上去的. 所以两个类真正不同的地方仅有`delete`函数而已.

REST Framework针对这种情况提供了另外一种解决方案, 视图集, 代码如下:

```python
from rest_framework.viewsets import ModelViewSet

class ProductViewSet(ModelViewSet):
    serializer_class = ProductSerializer
    queryset = Product.objects.all()

    def delete(self, request, *args, **kwargs):
        product = self.get_object()
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

仅需要更改一下父类, 并把代码进行简单和并即可, 而查看`ModelViewSet`的源码可以发现, 其本质也是一个包含多种`Mixin`的通用视图:

```python
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):
    """
    A viewset that provides default `create()`, `retrieve()`, `update()`,
    `partial_update()`, `destroy()` and `list()` actions.
    """
    pass
```

## 路由设置

当使用了视图集时, 对于接口路由的配置也需要进行对应的改动, 不再需要手动的设置接口对应的具体视图与名称, 而是交由`Router`进行自动设置. 代码如下:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter

from .views import generic_api


router = DefaultRouter()
router.register('products', generic_api.ProductViewSet)
router.register('collectons', generic_api.CollectionViewSet)

urlpatterns = [
    path('', include(router.urls)),
    # other path you need.
]
```

只需简单几个步骤:

1. 导入`DefaultRouter`, 并创建对象
2. 注册视图名与对应的视图集
3. 在urlpatterns中包含生成的路由配置

查看`router.urls`可以看到, 我们已经得到了一个完整的路由配置列表, 与之前的配置功能一样, 并自动为每一个路由生成了名称.

```bash
[
    <URLPattern '^products/$' [name='product-list']>,
    <URLPattern '^products/(?P<pk>[^/.]+)/$' [name='product-detail']>,
    <URLPattern '^collectons/$' [name='collection-list']>,
    <URLPattern '^collectons/(?P<pk>[^/.]+)/$' [name='collection-detail']>
]
```

除了自动完成了具体的路由配置之外, DefaultRouter还会配置一个根目录路由, 并在访问app是提供一个页面:

![preview](/assets/img/img_202311130250548202.png)
__`store/` API页面__

> 如果不想要这个页面, 也可以使用`SimpleRouter`.
>
{: .prompt-tip}

## 关于视图集DELETE方法的一个大坑

在尝试将通用视图改为视图集的实现方式是, 可以正确的执行查询和更新操作, 但是在删除时出现了问题:

![exception](/assets/img/img_202311130227323161.png)

即删除逻辑并没有执行, 而是直接调用了序列化对象的`delete()`方法, 导致直接抛出了依赖保护异常.

在将配置文件切换回通用视图后正常, 转到视图集后便出现问题. 查询源码发现了问题所在.

在通过`APIView`方式处理时, 代码如下:

```python
class ProductDetail(generics.RetrieveUpdateDestroyAPIView):
    # code

    def delete(self, request, *args, **kwargs):
        # logic
        return Response(status=status.HTTP_204_NO_CONTENT)
```

通过查看 `RetrieveUpdateDestroyAPIView` 类的源码:

```python

class RetrieveUpdateDestroyAPIView(mixins.RetrieveModelMixin,
                                   mixins.UpdateModelMixin,
                                   mixins.DestroyModelMixin,
                                   GenericAPIView):

    # other method

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)

```

可以看到, 父类的`delete`方法调用的是`self.destroy()`方法, 而这个`destroy`方法则是`destroyModelMixin`所定义的方法

```python
class DestroyModelMixin:
    """
    Destroy a model instance.
    """
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        self.perform_destroy(instance)
        return Response(status=status.HTTP_204_NO_CONTENT)

    def perform_destroy(self, instance):
        instance.delete()
```

再看`ModelViewSet`类的源码:

```python
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):
    """
    A viewset that provides default `create()`, `retrieve()`, `update()`,
    `partial_update()`, `destroy()` and `list()` actions.
    """
    pass

```

这或直接写了个pass, 也就是说自定义的`ViewSet`不会重写父类的`delete`方法, 因为父类就没有`delete`方法.

而`ViewSet`能够执行删除操作, 显然是交由路由配置之后直接调用了`destroy()`方法.

所以之前在`ViewSet`中定义的`delete`方法虽然有判断逻辑, 但是却被直接跳过了.

解决方式也很简单, 将`ViewSet`中的`delete`方法名改为`destroy`即可:

```python
class ProductViewSet(ModelViewSet):
    serializer_class = ProductSerializer
    queryset = Product.objects.all()

    def destroy(self, request, *args, **kwargs):
        product = self.get_object()
        print(product)
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
