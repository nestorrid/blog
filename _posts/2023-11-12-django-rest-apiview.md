---
title: 15. Django 视图类
date: 2023-11-12 19:48:01 +0800
categories: [笔记, Django]
tags: [python, django, backend]
---

在之前的代码中, 所有的请求都是通过python方法来进行处理的, 这会使得项目代码可维护性很差.

在实际应用中, 更多的则是使用视图类来对接口进行封装.

1. `store`文件夹下创建一个新的 `views` 包.
2. 创建两个文件`products_api.py`和`collections_api.py`

并对之前的视图方法进行重构, 结果如下:

```python

# collections_api.py
from django.shortcuts import render, get_object_or_404
from django.db.models import Count

from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework import status

from ..models import Collection
from ..serializers import CollectionSerializer


class CollectionList(APIView):

    def get(self, request: Request):
        queryset = Collection.objects.all().annotate(products_count=Count('products'))
        serializer = CollectionSerializer(queryset, many=True)
        return Response(serializer.data)

    def post(self, request: Request):
        serializer = CollectionSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)


class CollectionDetail(APIView):

    def get_collection(self, pk) -> Collection:
        return get_object_or_404(
            Collection.objects.annotate(products_count=Count('products')), pk=pk)

    def get(self, request, pk):
        collection: Collection = self.get_collection(pk)
        serializer = CollectionSerializer(collection)
        return Response(serializer.data)

    def put(self, request, pk):
        collection = self.get_collection(pk)
        serializer = CollectionSerializer(collection,
                                          data=request.data,
                                          partial=request.method == 'PATCH')
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    def patch(self, request, pk):
        return self.put(request, pk)

    def delete(self, request, pk):
        collection = self.get_collection(pk)
        if collection.products.count() > 0:
            return Response({"error": "Collection have more than one product."},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)
        collection.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

```python
from django.shortcuts import render, get_object_or_404

from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

from ..serializers import ProductSerializer
from ..models import Product


class ProductList(APIView):

    def get(self, request):
        queryset = Product.objects.all()
        serializer = ProductSerializer(queryset, many=True,
                                       context={'request': request})
        return Response(serializer.data)

    def post(self, request):
        if request.method == 'POST':
            serializer = ProductSerializer(data=request.data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            print(serializer.validated_data)
            return Response(serializer.data, status=status.HTTP_201_CREATED)


class ProdcutDetail(APIView):

    def get(self, request, product_id):
        product = get_object_or_404(Product, pk=product_id)
        serializer = ProductSerializer(product)
        return Response(serializer.data)

    def put(self, request, product_id):
        product = get_object_or_404(Product, pk=product_id)
        serializer = ProductSerializer(product,
                                       data=request.data,
                                       partial=request.method == 'PATCH')
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    def patch(self, request, product_id):
        return self.put(request, product_id)

    def delete(self, request, product_id):
        product = get_object_or_404(Product, pk=product_id)
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

完成后的结构为`store.model_api.APIView`, 如此一来, 便可以将相关的视图方法封装到同一个python模块中统一维护

同时一个雷对应了一个路由, 通过路由视图类的方法实现来确定可以接受的请求类型, 代码整体就变得清晰明了.

同时, 还需要编辑`store/urls.py`配置文件, 使路由能够找到对应的视图类:

```python
from django.urls import path, include
from .views import product_api, collection_api

urlpatterns = [
    # path('', views.index),
    path('products/', product_api.ProductList.as_view()),
    path('products/<int:product_id>/', product_api.ProdcutDetail.as_view()),
    path('collections/', collection_api.CollectionList.as_view()),
    path('collections/<int:pk>',
         collection_api.CollectionDetail.as_view(),
         name='collection-detail')
]

```
