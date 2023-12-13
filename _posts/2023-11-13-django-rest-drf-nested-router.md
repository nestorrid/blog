---
title: 19. 使用drf-nested-routers实现路由嵌套
date: 2023-11-13 16:30:20 +0800
categories: [笔记, Django]
tags: [python, django, backend, REST]
---

在django中, 基础的路由配置格式为:

```python
urlpatterns = [
    path('', views.index),
    path('products/', generic_api.ProductList.as_view()),
    path('products/<int:pk>/', generic_api.ProductDetail.as_view())
]
```

但这需要为每一个公开的接口配置具体的路由, 且接口变动需要手动进行维护.

通过rest的Router可以简化路由的配置.

```python
router = DefaultRouter()
router.register('products', viewset_api.ProductViewSet)
router.register('collectons', viewset_api.CollectionViewSet)

urlpatterns = [
    path('', include(router.urls)),
    # other path you need.
]
```

这样便可以动态的生成路由信息, 仅需要关注编码即可. 但缺乏多级路由的支持. 比如

* `products/1/reviews`: id为1的产品的评论列表
* `products/1/reviews/2` id为1的产品下id为2的评论详情

根据REST的规则, 很容易便可以明白这些路由的含义, 但是在路由配置时, 通过REST Framework提供的路由便比较难以处理.

为了更为方便的进行多级路由的配置, 可以使用`drf-nested-routers`

## 安装 drf-nested-routers

[Gitee镜像主页](https://gitee.com/mirrors_alanjds/drf-nested-routers)
[Github主页](https://github.com/alanjds/drf-nested-routers)

通过pipenv或者pip进行安装即可:

```bash
pipenv install drf-nested-routers
```

## 配置嵌套路由

在`store/urls.py`中对`Product`和`Review`配置路由:

```python
from django.urls import path, include
from rest_framework_nested import routers

from .views import viewset_api

router = routers.DefaultRouter()
router.register('products', viewset_api.ProductViewSet, basename='products')
router.register('collectons', viewset_api.CollectionViewSet)

# 参数
# router - 所属的父路由对象
# 'products' - 对应父路由对象中的哪一个路由, 便是之前通过 router.register()注册的内容
# lookup - 查询前缀, 会自动生成一个`product-pk`键
products_router = routers.NestedDefaultRouter(router, 'products', lookup='product')
products_router.register(
    'reviews', viewset_api.ReviewViewSet, basename='product-reviews')

urlpatterns = [
    path(r'', include(router.urls)),
    path(r'', include(products_router.urls))
]
```

> drf的路由嵌套并没有层级限制, 可以在次技术上进行更深层的嵌套.
>
> 比如为`review`再添加一个`reply`等等.
>
{: .prompt-tip}

## 创建 Review 模型

一个用于记录`Product`评论的简单表, 代码如下:

```python
class Review(models.Model):
    product = models.ForeignKey(
        Product, on_delete=models.CASCADE, related_name="reviews")
    name = models.CharField(max_length=255)
    description = models.TextField()
    date = models.DateField(auto_now=True)
```

为了能够配置ViewSet, 还需要为`Review`创建一个序列化对象:

```python
class ReviewSerializer(serializers.ModelSerializer):
    class Meta:
        model = Review
        fields = ['id', 'name', 'date', 'description']

    def create(self, validated_data):
        product_id = self.context['product_id']
        return Review.objects.create(product_id=product_id, **validated_data)
```

> 关于`create`方法.
>
> 在创建`Review`的具体数据时, 我们需要知道该数据时对应的哪一个product.
>
> 虽然可以通过手动输入`product_id`, 但实际上url之中便已经包含了具体的该值, 只需要从其中获取即可.
> `self.context['product_id']`的作用便是从请求的上下文中获取具体的值.
>
{: .prompt-info}

## 编写对应的Viewset

```python
class ReviewViewSet(ModelViewSet):

    serializer_class = ReviewSerializer

    def get_queryset(self):
        return Review.objects.filter(
            product_id=self.kwargs['product_pk'])

    def get_serializer_context(self):
        context = super().get_serializer_context()
        context['product_id'] = self.kwargs['product_pk']
        return context
```

> 直接获取所有的Review操作没有实际意义, 必须知道产品的id才能获取其相应的review, 也就不能简单的使用`objects.all()`
>
> 由于增加了额外的逻辑, 便需要重写`get_queryset`方法. 添加对`product_id`的过滤器
> 而`get_serializer_context`方法则是将URL中的id储存在上下文中供序列化对象使用.
>
{: .prompt-info}

> 几个ID的对应关系:
>
> * `product_pk`, 由路由配置自动生成, `routers.NestedDefaultRouter(router, 'products', lookup='product')`
> 作用是让`ViewSet`能够获取URL的参数.
> * `product_id`, 序列化对象和模型类中外键字段名.
>
{: .prompt-tip}
