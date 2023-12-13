---
title: Django 常用基础配置
date: 2023-11-11 13:59:45 +08000
categories: [速查, 配置]
tags: [django, config]
---

## django-debug-toolbar基础配置

```python
# settings.py
INSTALLED_APPS = [
    # ...
    "debug_toolbar",
    # ...
]

MIDDLEWARE = [
    "debug_toolbar.middleware.DebugToolbarMiddleware",
    # ...
]

# 直接添加在`settings.py`文件中 
INTERNAL_IPS = [
    "127.0.0.1",
]

# urls.py
from django.urls import include, path

urlpatterns = [
    # ...
    path("__debug__/", include("debug_toolbar.urls")),
]

```

## MySQL基础配置

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'storefront',
        'HOST': 'localhost',
        "USER": 'root',
        "PASSWORD": 'pass'
    }
}
```
