---
title: 1. 第一个Django项目
date: 2023-11-5 01:23:36 +08000
categories: [笔记, Django]
tags: [python, django, backend]
image: /assets/img/48223734b31de65181a4a38de6d8ac54.png
---

## 前期准备

[Django在线文档](https://docs.djangoproject.com/en/4.2/)

下载pipenv

```pip install pipenv```

下载: [vscode](https://code.visualstudio.com)

VS Code必备插件:

* python

* pylint

  > 在vscode中会出现某些生成代码存在格式问题, 如import未在文件头等, 可以通过注释解决, 如:
  >
  > ```pylint: disable=C0415```

* autopep8

## 创建项目

```dash
mkdir project
cd project 

pipenv install django
pipenv shell

django-admin startproject projectname .
```

## 创建 Django app

```bash
python manage.py startapp app
python manage.py runserver
```

在`settings.py`文件中添加创建的app

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    # 'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'playground'
]
```

## 编写一个View

在创建的app中找到`views.py`文件

```python
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def say_hello(request):
    return HttpResponse('hello world')
```

在app目录下创建`urls.py`文件

```python
# 这是app内部的urls配置
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.say_hello)
]
```

在项目的`urls.py`文件中配置子程序的url

```python
from django.contrib import admin
# 需要包含其他的配置文件,添加 `include` 引用
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    # 配置子app的路由
    path('playground/', include('playground.urls'))
]

```

## 使用模板

在app目录中创建一个模板目录 `templates`

创建一个html文件, 比如`hello.html`, 编辑代码

```html
<h1>hello {{ name }}</h1>
```

修改`app/views.py`的代码

```python
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def say_hello(request):
    return render(request, 'hello.html' , {'name': 'Nestor'})

```

## 使用Django debug toolbar进行调试

文档:[https://django-debug-toolbar.readthedocs.io/en/latest/](https://django-debug-toolbar.readthedocs.io/en/latest/)

```zsh
pipenv install django-debug-toolbar
```

1. 在`settings.py`文件中添加调试工具相关配置

   ```python
   INSTALLED_APPS = [
       # ...
       "debug_toolbar",
       # ...
   ]
   
   MIDDLEWARE = [
       # ...
       "debug_toolbar.middleware.DebugToolbarMiddleware",
       # ...
   ]
   
   # 直接添加在`settings.py`文件中 
   INTERNAL_IPS = [
       # ...
       "127.0.0.1",
       # ...
   ]
   ```

2. 在`urls.py`配置文件中添加debug路由

   ```python
   from django.urls import include, path
   
   urlpatterns = [
       # ...
       path("__debug__/", include("debug_toolbar.urls")),
   ]
   ```
