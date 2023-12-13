---
title: pylint提示django模型类没有成员
date: 2023-11-11 14:08:51 +08000
categories: [速查, 异常]
tags: [django, pylint, vscode]
---

在使用Django orm模型类时调用`objects` 成员的方法, 如:

```python
query = Character.objects.filter(pk__lt=10)
```

由于`Character`继承自django的Model类, 而本身没有objects成员变量, 便会导致pylint提示代码出错

**解决方案:**

安装`pylint-django`插件

```bash
pipenv install pylint-django
```

在`settings.json`配置文件中添加:

```json
"pylint.args": [
        "--load-plugins=pylint_django"
    ],
```
