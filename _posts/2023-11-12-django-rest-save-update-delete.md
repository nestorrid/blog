---
title: 14. Django REST framework 数据修改
date: 2023-11-12 16:18:57 +0800
categories: [笔记, Django]
tags: [python, django, backend, REST]
---

## 保存数据

对于简单数据的表单, rest framework 提供了很简单的数据保存方式.

比如, 按照之前的内容, 新增一个`collection_list`视图方法, 用于对`Collection`进行获取和新增

```python
@api_view(['GET', 'POST'])
def collection_list(request):
    if request.method == 'GET':
        queryset = Collection.objects.all()
        serializer = CollectionSerializer(queryset, many=True)
        return Response(serializer.data)

    if request.method == 'POST':
        serializer = CollectionSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```

通过URL:`/store/collections/`发送POST请求进行接口测试:

```python
# HTTP 201 Created
# Allow: POST, OPTIONS, GET
# Content-Type: application/json
# Vary: Accept

{
    "id": 11,
    "title": "new collection"
}
```

通过模型类定义的数据库验证, 以及序列化对象定义的前段数据验证, `is_valid()`方法便可以保证数据的有效性, 并通过`sava()`方法直接进行保存.

但有时仍然需要进行一些其他的操作, 比如处理前段不可见的隐藏字段, 生成日志文件等等.

`ModelSeriallizer`提供了两个方法, 用于处理数据的新增和更新, 分别是`create`和`update`.

仅需要重写这两个方法, RESTFramework会根据请求类型自动进行调用. 代码如下:

```python
class CollectionSerializer(serializers.ModelSerializer):

    class Meta:
        model = Collection
        fields = ['id', 'title']

    def create(self, validated_data):
        collection = Collection(**validated_data)
        print(collection.title)
        collection.save()
        return collection

    def update(self, instance, validated_data):
        print("OLD TITLE:", instance.title, "-",
              "NEW TITLE:", validated_data.get('title'))
        instance.title = validated_data.get('title')
        instance.save()
        return instance
```

## 更新数据

数据的更新通过`PUT`和`PATCH`两种请求方式完成

* `PUT`: PUT请求在传输时,要求一次性将请求所用的数据传输完成,即必须将所有的请求数据都完整地发送到服务器端.
* `PATCH`: PATCH请求可以对资源进行部分修改,即客户端只需将要修改的部分发送给服务器,而无需将整个数据实体发送给服务器.因此,PATCH方法可以在节省带宽的同时提高效率,并可以支持增量更新.

两种方式的区别在于:

1. 数据传输方式不同: PUT请求要求一次性将请求所需的所有数据传输完毕,而PATCH请求只传输要修改的部分数据,节约了带宽和传输时间.
2. 适用场景不同: PUT请求通常用于更新和替换整个资源,而PATCH请求则适用于对资源进行部分修改,可以在不修改整个资源的情况下实现增量更新.
3. 安全性不同: 由于PUT请求要求一次性发送所有数据,因此可能会存在重复更新或错误更新等问题.而PATCH请求只更新要修改的部分数据,因此更加安全,不易出错.

修改`product_detail`视图方法:

```python
@api_view(['GET', 'PUT', 'PATCH'])
def product_detail(request, product_id):
    product = get_object_or_404(Product, pk=product_id)

    if request.method == 'GET':
        serializer = ProductSerializer(product)
        return Response(serializer.data)

    if request.method == 'PUT':
        serializer = ProductSerializer(product, data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    if request.method == 'PATCH':
        serializer = ProductSerializer(
            product, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)
```

`GET`方式的实现代码与之前没有太大区别, `PUT`与`PATCH`的实现代码也没有太大区别.

但是`PATCH`方法在进行序列化对象初始化的时候有一个额外关键字参数: `partial=True`

> 这一参数表示允许进行部分字段的更新! 如果没有声明这一参数, 那么`PUT`和`PATCH`的处理方式将完全相同.
{: .prompt-info}

通过`PUT`和`PATCH`两种请求向`store/product/1001`发送测试数据:

```json
{ "price": 100}
```

`PUT`请求结果:

```python
# HTTP 400 Bad Request
# Allow: PATCH, GET, PUT, OPTIONS
# Content-Type: application/json
# Vary: Accept

{
    "title": [
        "该字段是必填项。"
    ],
    "inventory": [
        "该字段是必填项。"
    ],
    "collection": [
        "该字段是必填项。"
    ]
}
```

`PATCH`请求结果:

```python
# HTTP 200 OK
# Allow: PATCH, GET, PUT, OPTIONS
# Content-Type: application/json
# Vary: Accept

{
    "id": 1001,
    "title": "Dried Peach",
    "inventory": 39,
    "price": 100.0,
    "price_with_tax": 117.0,
    "collection": 4
}
```

假设一个用户更新昵称的需求, 显然其他的用户资料并没有变动, 仅仅只是修改了昵称

虽然可以通过默认加载原始数据到表单中, 在发送更新数据时同时发送未修改的原始数据.

但这无疑是对带宽和用户流量的浪费, 也造成了不必要的数据库读写.

在`PUT`和`PATCH`的代码逻辑几乎一样的情况下, 可以把上面的代码进行一下优化:

```python
@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def product_detail(request, product_id):
    product = get_object_or_404(Product, pk=product_id)
    
    # other code...

    if request.method in ['PUT', 'PATCH']:
        serializer = ProductSerializer(product,
                                       data=request.data,
                                       partial=request.method == 'PATCH')
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    # other code ...
```

## 删除数据

删除数据作为一项危险操作, 应当进行充分的操作确认, 既要对用户进行删除操作确认, 对于重要内容的删除也可以进行密码确认.
而在服务端进行删除之前同样要进行各种操作, 如是否可以删除的判断, 用户操作权限的查询等.

但是排除所有的确认, 删除操作本身非常简单.

```python
@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def product_detail(request, product_id):
    product = get_object_or_404(Product, pk=product_id)

    # other code ...

    if request.method == 'DELETE':
        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

向`store/product/1`发送`DELETE`请求, 得到如下提示:

```text
ProtectedError at /store/products/1/
("Cannot delete some instances of model 'Product' 
because they are referenced through protected foreign keys: 
    'OrderItem.product'.", {<OrderItem: OrderItem object (40)>, <OrderItem: OrderItem object (765)>})
```

其原因在于`OrderItem`类中定义了`Product`外键, 并组织了其删除操作:

```python
class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.PROTECT)
    # here
    product = models.ForeignKey(Product, on_delete=models.PROTECT)
    quantity = models.PositiveSmallIntegerField()
    unit_price = models.DecimalField(max_digits=6, decimal_places=2)
```

如果不进行任何处理便直接进行删除操作, 便有可能使得程序出现异常:

![exception](/assets/img/img_202311121735237199.png)

可以在删除前进行判断, 代码如下:

```python
@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def product_detail(request, product_id):
    product = get_object_or_404(Product, pk=product_id)

    # other code ...

    if request.method == 'DELETE':
        if product.orderitem_set.count() > 0:
            return Response({'error': 'Product has been ordered.'},
                            status=status.HTTP_405_METHOD_NOT_ALLOWED)

        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

再次发送删除请求, 获得回复:

```python
# HTTP 405 Method Not Allowed
# Allow: GET, DELETE, OPTIONS, PUT, PATCH
# Content-Type: application/json
# Vary: Accept

{
    "error": "Product has been ordered."
}
```
