---
title: 10. 使用Django开发RESTFUL API
date: 2023-11-11 16:08:45 +08000
categories: [笔记, Django]
tags: [python, django, backend, REST]
image: https://www.django-rest-framework.org/img/logo.png
---

REST(Representational State Transfer), 一套客户端与服务器进行交互的规则

## Resources

可以理解为能够通过URL访问的对象, 比如`Product`, `Collection`等.

比如:

```url
http://your.url/products
http://your.url/products/1
http://your.url/products/1/reviews
http://your.url/products/1/reviews/1
...
```

当客户端访问这些URL的时候, 后台会根据请求的URL以`HTML`, `XML`, `JSON`等呈现形式返回对应的数据.

## HTTP method

`GET`: 获取数据
`POST`: 创建数据
`PUT`: 更新数据
`PATCH`: 部分更新
`DELETE`: 删除数据

例如创建一个Product, 可以发送:

```text
POST /products

    {
        "title": "new product",
        "price": 0.99,
        ...
    }
```

更新某个Product的数据, 则可以发送:

```text
PATCH /products/1

    {
        "price": 1.99,
        ...
    }
```

## 安装 djangorestframework

```bash
pipenv install djangorestframework
```

配置app, 在`project/settings.py`文件中, 添加`INSTALLED_APP`配置:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
]
```

## 创建 API Views

以 `store` app为例, 在 `store/views.py` 文件中创建方法:

```python
from django.shortcuts import render
from django.http import HttpResponse


def product_list(request):
    return HttpResponse("ok")

```

同时修改 `store/urls.py` 文件:

```python
from django.urls import path, include
from . import views

urlpatterns = [
    path('', views.index),
    path('products/', views.product_list)
]
```

最后在 `project/urls.py` 中加入app的对应配置:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('store/', include('store.urls'))
]
```

此时, 通过浏览器访问 `http://127.0.0.1:8000/store/products/` 如果显示`ok`, 那么变代表创建成功了.

## 使用 djangorestframework

[官方文档](https://www.django-rest-framework.org)

Django的View通过`HttpRequest`和`HttpResponse`接收和返回请求, 但是这两个对象更多的是基于Django自身的`MTV`模式而设计的.

为了更好的使用REST API, 需要通过`djangorestframework`所封装的`Request`和`Response`对象进行替换.

修改`store/views.py`文件

```python
from django.shortcuts import render
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view()  # 该装饰器替换默认的request对象为djangorestframework的request对象
def product_list(request):
    # 使用djangorestframwork的Response返回数据
    return Response("ok")
```

此时, 如果通过浏览器调用URL, 便会显示API的运行结果页面, 如下图所示
![result](/assets/img/img_202311112248208661.png)
__运行结果__

但如果通过非浏览器调用, 就仅仅只会获得返回的json数据.

### 根据ID获取对应的对象

按照REST的规则, 获取全部对象的url是`GET products/`, 而获取指定id对象的url便应该是`GET products/id`.

在`store/views.py`新建视图方法:

```python
# store/views.py
@api_view()
def product_detail(request, product_id):
    return Response("ok")
```

同时更新配置文件:

```python
# store/urls.py
from django.urls import path, include
from . import views

urlpatterns = [
    path('', views.index),
    path('products/', views.product_list),
    path('products/<int:product_id>/', views.product_detail)
]
```

> 视图方法中的形参名称`product_id`与路由配置中的形参名称必须相同, `<int:product_id>`限定了参数形式必须为int类型, 如果传入其他类型的参数则会跳转至404页面.
{: .prompt-tip }
