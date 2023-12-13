---
title: 17. 自定义通用视图
date: 2023-11-13 01:22:11 +0800
categories: [笔记, Django]
tags: [python, django, backend, REST]
---

当REST framework所提供的通用视图不能很好的满足需求是, 也可以创建自定义的通用视图. 以`ProductDetail`为例, 原始代码如下:

```python
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

由于接口需要处理`GET`,`PUT`, `PATCH`和`DELETE`, 我们可以通过继承通用视图`RetrieveUpdateDestroyAPIView`

显然, `get`, `put`, `patch`三个方法都没有额外的逻辑, 只有在`delete`时需要进行额外处理. 修改后的代码如下:

```python
class ProductDetail(generics.RetrieveUpdateDestroyAPIView):

    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def delete(self, request, *args, **kwargs):
        product = get_object_or_404(Product, *args, **kwargs)
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

同样也可以通过继承`GenericView`和所需的`Mixin`来实现, 与rest现有的通用视图方式相同, 代码如下:

```python
class RetrieveDestroyAPIView(mixins.RetrieveModelMixin,
                             mixins.DestroyModelMixin,
                             GenericAPIView):
    """
    Concrete view for retrieving or deleting a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```
