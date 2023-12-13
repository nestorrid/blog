---
title: 设计模式-责任链模式
date: 2023-11-21 15:55:54 +0800
categories: [笔记, 设计模式]
tags: ['design pattern']
mermaid: true
---

## 应用场景

相当于为一项任务创建一条流水线, 类似于django的中间件等功能都是采用责任链模式来完成的.

即接到一个任务时, 不同的对象处理完自己的部分任务, 然后将其交给下一个对象一次处理.

类似于HttpRequest请求, 可能包含:

* 登录验证
* 权限验证
* IP验证
* 一系列验证
* 处理请求
* 返回结果

责任链模式的好处在于每一个处理任务之间彼此独立, 相互没有依赖, 可以在不修改硬编码的情况下根据需要自行配置处理流程.

假设用责任链模式完成一个小功能, 计算

```text
y = (|x|+10)^3/2-1
```

## 简单的python实现

```python
import sys
from importlib import import_module


def get_abs(num):
    return abs(num)


def plus_ten(num):
    return num + 10


def power_by_three(num):
    return num ** 3


def divide_by_two(num):
    return num / 2


def minus_one(num):
    return num - 1


def operation_name(name):
    module_name = __file__.rsplit('.', 1)[0].rsplit('/', 1)[-1]+"."
    return module_name + name


OPERATIONS = [
    operation_name("get_abs"),
    operation_name("plus_ten"),
    operation_name("power_by_three"),
    operation_name("divide_by_two"),
    operation_name('minus_one')
]


def cached_import(module_path, class_name):
    # Check whether module is loaded and fully initialized.
    if not (
        (module := sys.modules.get(module_path))
        and (spec := getattr(module, "__spec__", None))
        and getattr(spec, "_initializing", False) is False
    ):
        module = import_module(module_path)
    return getattr(module, class_name)


def import_string(full_path):
    """
    Import a dotted module path and return the attribute/class designated by the
    last name in the path. Raise ImportError if the import failed.
    """
    try:
        module_path, class_name = full_path.rsplit(".", 1)
    except ValueError as err:
        raise ImportError("%s doesn't look like a module path" %
                          full_path) from err

    try:
        return cached_import(module_path, class_name)
    except AttributeError as err:
        raise ImportError(
            'Module "%s" does not define a "%s" attribute/class'
            % (module_path, class_name)
        ) from err


if __name__ == "__main__":

    x = -3

    for opreation in OPERATIONS:
        f = import_string(opreation)
        x = f(x)

    print(x)
```

代码本身...啥也不是...

通过几个相互之间没啥关系, 且功能单一的函数构成了一个责任链, 通过`OPERATIONS`数组保存.

按照数组的顺序遍历, 获取每一步操作, 跟着步骤走.

最终得到正确的计算结果. 责任链模式就算是跑通了.

正常来说, 这个`OPERATIONS`对象肯定是放在类似于`settings.py`一类的配置文件中的.

也就可以方便的通过配置文件来修改程序的流程而不需要重新编码了.
