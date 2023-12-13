---
title: 12. REST 模型序列化
date: 2023-11-12 00:10:52 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

之前的序列化对象继承自`serializers.Serializer`, 功能很强大, 但问题在于重复代码太多.

在Product模型中定义了`title = models.CharField(max_length=255)`

而在ProductSerializer中则要定义`title = serializers.CharField(max_length=255)`

几乎一样的代码, 只是类型发生变化, 但却实实在在的需要写两遍, 而且一旦修改模型, 序列化对象也要一同修改.

显然这并不是一个好的解决方案. 事实上, `serializers.Serializer`本身便是为了序列化普通的Python对象而存在的.

而对于Django模型类, Django REST Framework提供了另外一种方式: `ModelSerializer`.

[官方文档](https://www.django-rest-framework.org/api-guide/serializers/#modelserializer)

---

## 创建模型序列化对象

```python
class CollectionSerializer(serializers.ModelSerializer):

    class Meta:
        model = Collection
        fields = ['id', 'title']
```

只是单纯的对`id`和`title`进行了序列化, 与之前定义的`CollectionSerializer`类的功能完全一样.

仅需要简单的声明`class Meta`, 并列出需要的字段即可.

与之相比, ProductSerializer则会相对复杂一些, 重写后的代码如下:

```python
class ProductSerializer(serializers.ModelSerializer):

    class Meta:
        model = Product
        fields = ['id', 'title', 'price', 'price_with_tax', 'collection']

    # 对`unit_price` 字段进行了重名民
    price = serializers.DecimalField(max_digits=6,
                                     decimal_places=2,
                                     source="unit_price")

    # 添加了模型类中不存在的字段, 并定义了其数据生成方法
    price_with_tax = serializers.SerializerMethodField(
        method_name="calc_price_with_tax")

    # 修改了字段的数据类型
    collection = serializers.HyperlinkedRelatedField(
        read_only=True,
        view_name='collection-detail'
    )

    def calc_price_with_tax(self, product: Product):
        return product.unit_price * Decimal(1.17)

```

可以看出, 模型序列化可以简化大量的编码.

* 对于同名, 同类型的字段, 仅需要在字段列表中列出即可.
* 不同名但是同值的字段, 在列表中定义字段名, 编写成员变量, 并为其指定模型字段
* 模型中不存在的字段, 在列表中声明, 然后定义同名的成员变量
* 模型和字段列表中都有, 但是声明了同名成员变量的字段, 以成员变量的序列化声明为准.

> 对于`fields`成员, 可以使用`__all__`来声明包含全部的模型字段.
> 但这是一种非常不好的习惯, 看似简单方便之下隐藏着重大隐患.
>
> * 模型的全部字段将暴露在接口之中, 可能泄露隐藏信息
> * 模型类修改后, 接口也会发生变动, 破坏了接口不变原则
>
> > *Only lazy programmers will do that. You don't want to be one of them.
>
{: .prompt-warning}

> 但是对于字段较多的模型, 同样有简化的方法:
>
> 通过`exclude=[fields]`来排除特定字段,同时包含其余字段即可.
>
{: .prompt-tip}
