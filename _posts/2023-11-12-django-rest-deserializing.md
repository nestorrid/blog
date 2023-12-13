---
title: 13. 模型反序列化
date: 2023-11-12 01:57:01 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---


在客户端发送`POST`请求至后台时, 需要将其发送的数据转化为python对象, 此时便需要进行反序列化

## 反序列化的简单实现

基于REST的规范, 获取资源和创建资源应该分别对应同一个URL的`GET`和`POST`方法.

所以我们可以直接修改`products/`路由对应的视图:

```python
@api_view(['GET', 'POST'])
def product_list(request):

    if request.method == 'GET':
        queryset = Product.objects.all()
        serializer = ProductSerializer(queryset, many=True,
                                       context={'request': request})
        return Response(serializer.data)

    if request.method == 'POST':
        serializer = ProductSerializer(data=request.data)
        return Response("ok")
```

当视图方法设置为`GET`, `POST` 时, Django REST framework会自动在接口页面创建一个表单, 以方便我们直接对接口进行测试.
![post form](/assets/img/img_202311120215571964.png)

## 数据校验

在视图方法中通过序列化对象进行数据校验, 只需要调用`is_valid()`方法即可. 数据会自动根据数据模型的字段和验证器定义进行校验.

```python
if request.method == 'POST':
        serializer = ProductSerializer(data=request.data)
        if serializer.is_valid():
            print(serializer.validated_data)
            return Response("ok")

        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

此时, 如果想接口发送一个空对象, 则会得到如下返回信息:

```python
# HTTP 400 Bad Request
# Allow: OPTIONS, POST, GET
# Content-Type: application/json
# Vary: Accept

{
    "title": [
        "This field is required."
    ],
    "price": [
        "This field is required."
    ],
    "collection": [
        "This field is required."
    ]
}
```

> REST framework提供了一个更为简单的实现方式
>
> 通过给`is_valid()`方法添加关键字参数`raise_exception=True`便可以得到完全相同的结果
>
> ```python
> if request.method == 'POST':
>   serializer = ProductSerializer(data=request.data)
>   serializer.is_valid(raise_exception=True)
>   print(serializer.validated_data)
>   return Response("ok")
> ```
>
{: .prompt-tip}

### 前端数据验证

对于`POST`数据不仅仅要进行数据类型, 非空, 查重等数据库验证, 有些时候也会需要进行一些数据有效性的验证.
比如创建用户操作时, 密码和确认密码是否相同一类.

此时可以直接在序列化对象中直接进行额外验证的编写, 在调用`is_valid()`方法时便会在一同进行自定义验证.

在`ProductSerializer`中添加如下代码:

```python
def validate(self, attrs):
    if attrs['unit_price'] > 100:
        raise serializers.ValidationError("太贵了没人买")

    return attrs
```

> 虽然在序列化对象时定义了字段命为`price`, 但这里的字段名称是`unit_price`.
>
> 因为验证是在数据反序列化之后才会进行的, 所以`attrs`对象中的键都是`Product`模型类所定义的字段名.
>
{: .prompt-warning}
