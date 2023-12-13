---
title: 在vscode中设置pylint
date: 2023-11-19 22:09:10 +0800
categories: [速查, 配置]
tags: [pylint, python]
---

## 设置全局配置

在vscode中打开`settings.json`, 加入全局配置:

```python
{
    "pylint.args": [
        # 通过`--disable`参数标记在全局禁用的规则
        "--disable=C0116", # missing-function-docstring / C0116
        "--disable=C0114", # missing-module-docstring / C0114
        "--disable=C0115", # missing-class-docstring / C0115
        "--disable=C0415", # import-outside-toplevel / C0415
        "--disable=R1710", # inconsistent-return-statements / R1710
        "--disable=R0901", # too-many-ancestors / R0901
        # `--rcfile`参数用来指定项目独立的配置文件
        # 此处代表工作区根目录下的`.pylintrc`文件
        "--rcfile=${workspaceFolder}/.pylintrc"
    ]
}
```

这里加入了几个比较烦人的验证, docstring是应该编写的东西, 但有些简单功能甚至单行代码的函数也让写docstring就有些烦人了. 另外如`R1710`验证, 可以看一下下面的代码:

```python
def func(num):
    if num > 0:
        return True
```

如果没有禁用R1710,那么就必须在if外面再写一个`return False`. 代码规范是好事, 但有些脱了裤子放屁的强制性规范不要也罢.

## 生成配置文件

在项目根目录下运行命令来生成默认的配置文件:

```bash
pylint --generate-rcfile > .pylintrc
```

> 建议把`.pylintrc`加入到`.gitignore`文件中
>
{: .prompt-tip}

在该文件仲可以加入允许和禁用的检查策略, 或者配置插件

```text
disable=raw-checker-failed,
        bad-inline-option,
        ...

load-plugins=pylint_django        
```

[pylint官方配置文档](https://pylint.pycqa.org/en/latest/user_guide/configuration/all-options.html#)
