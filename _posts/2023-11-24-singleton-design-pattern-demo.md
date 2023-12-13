---
title: 设计模式-单例模式
date: 2023-11-24 10:20:49 +0800
categories: [笔记, 设计模式]
tags: ['design pattern', ⭐️]
mermaid: true
---

## 应用场景

单例模式属于比较常见的基础设计模式. 即一个类在全局中仅存在一个实例化的对象.

比较常见的地方比如项目的配置管理对象. 因为保存了各个模块之间的统一配置信息以保证其在相同的环境下运行. 单例模式就是很好的选择.

## 简单的单例实现

简单的java单例

```java
public class ConfigManager {

    private static ConfigManager instance = new ConfigManager();
    private Map<String, Object> config = new HashMap<String, Object>();

    private ConfigManager() {
    }

    public static ConfigManager getInstance() {
        return instance;
    }

    public Object getConfig(String key) {
        return config.get(key);
    }

    public void setConfig(String key, Object value) {
        config.put(key, value);
    }
}
```

## python中的单例实现

作为比较常用的设计模式, python对于单例的实现相较java来说就差了点意思.

### 通过模块实现

在python中每一个`.py`文件都被看做一个模块, 而模块本身就是单例的.

因此最简单的单例实现方式就是直接在模块中定义类, 创建单例实例供外部引用

```python
# singleton.py
class Singleton:

    desc = "this is a Singleton Class"

    def action(self):
        print(self.desc)


instance = Singleton()

# app.py
from singleton import instance

instance.action()

```

### 通过装饰器实现单例

```python
import functools


def singleton(cls):
    __instance = {}

    @functools.wraps(cls)
    def instance(*args, **kwargs):
        if cls not in __instance:
            __instance[cls] = cls(*args, **kwargs)
        return __instance[cls]
    return instance


@singleton
class Singleton:

    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return "<Singleton name='%s' id=0x'%x'>" % (self.name, id(self))


if __name__ == "__main__":
    s1 = Singleton("a")
    s2 = Singleton("b")

    print(repr(s1))
    print(repr(s2))
```

输出结果:

```bash
<Singleton name='a' id='0x100c49ed0'>
<Singleton name='a' id='0x100c49ed0'>
```

虽然构造`s2`时使用了新的名称, 但是在最终输出的时候获得却是同一个对象, 名称和地址都没有改变.

### 通过类实现

```python
class Singleton:

    def __init__(self, name):
        self.name = name

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, '_instance'):
            Singleton._instance = Singleton(*args, **kwargs)

        return Singleton._instance

    def __str__(self):
        return f"{self.name}@0x{hex(id(self))}"


if __name__ == "__main__":
    s1 = Singleton.instance("s1")
    s2 = Singleton.instance("s2")
    s3 = Singleton("s3")

    print(s1, s2, s3)

```

输出结果:

```bash
s1@0x100f86090 s1@0x100f86090 s3@0x100f86210
```

### 单例模式的线程安全问题

从结果上来看, 在调用instance()方法获取实例的时候是能够实现单例的, 但是存在两个问题:

1. python的类不是线程安全的, 如果获取实例的间隔小于初始化的时间, 则可能存在多个实例.
2. 没有私有化的构造函数, 无法避免因为直接创建对象实例的误操作而产生的影响.

如果延长类的构建时间, 并通过线程来调用单例类:

```python
 def task(name):
        s = Singleton.instance(name)
        print(s)

    for i in range(0, 3):
        threading.Thread(target=task, args=[f"name{i}",]).start()
```

得到的输出结果就是:

```bash
name2@0x102b3d110
name0@0x102b3ca90
name1@0x102b3cdd0
```

显然, 每一个线程都创建了一个对象的实例.

为了解决线程安全的问题, 需要对类的定义做一下处理:

```python
@classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, '_instance'):
            with Singleton.__instance_lock:
                if not hasattr(Singleton, '_instance'):
                    Singleton._instance = Singleton(*args, **kwargs)

        return Singleton._instance
```

> 进行两次`if not hasattr(Singleton, '_instance'):`判断的作用:
>
> 1. 判断当前有没有实例, 如果没有, 看看有没有锁, 如果有锁说明其他对象正在创建实例, 等待解锁;
> 2. 在别的线程解锁自己锁定后, 判断是否存在实例, 如果没有则创建实例
>
> 外层判断最大的作用是避免在已经有实例的情况下开启线程锁, 增加资源消耗.
>
> 内层判断的作用是避免其他线程创建成功后再次进行重复创建.
>
> 总的来说, 外层判断可以省略, 但是会略问增加资源消耗. 但内层判断必须保留, 否则总是会创建新的实例.
>
{: .prompt-tip}

再次调用之前的多线程函数获得结果:

```bash
name0@0x1050e0b90
name0@0x1050e0b90
name0@0x1050e0b90
```

而同样的线程安全问题, 在通过装饰器实现的单例中同样存在. 因此对于装饰器方法也应进行修改:

```python
def singleton(cls):
    __instance = {}
    __instance_lock = threading.Lock()

    @functools.wraps(cls)
    def instance(*args, **kwargs):
        if cls not in __instance:
            with __instance_lock:
                if cls not in __instance:
                    __instance[cls] = cls(*args, **kwargs)

        return __instance[cls]
    return instance
```

### 基于__new__方法实现单例

通过上面的两种几种形式都可以实现单例, 但依旧有一个问题没有解决, 就是没有避免直接创建实例的误操作.

在python中对象的实例化首先是通过`object`对象的`__new__`方法创建, 然后再调用`__init__`方法进行初始化的.

所以重写`__new__`方法也可以实现单例.

```python
import threading


class Singleton:

    __instance_lock = threading.Lock()

    def __init__(self):
        if not hasattr(self, 'count'):
            self.count = 0
        self.count += 1

    @staticmethod
    def instance():
        return Singleton()

    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            with cls.__instance_lock:
                if not hasattr(cls, '_instance'):
                    cls._instance = object.__new__(cls)

        return cls._instance

    def __str__(self):
        return f"<Singleton@{hex(id(self))} called:{self.count}>"


if __name__ == "__main__":

    def single_thread():
        s1 = Singleton()
        s2 = Singleton()
        s3 = Singleton.instance()

        print(s1, s2, s3)

    def task1():
        s = Singleton()
        print(s)

    def task2():
        s = Singleton.instance()
        print(s)

    def multiple_thread():
        for _ in range(3):
            threading.Thread(target=task1).start()
            threading.Thread(target=task2).start()

    multiple_thread()
```

输出结果:

```bash
<Singleton@0x1047dcd50 called:1>
<Singleton@0x1047dcd50 called:2>
<Singleton@0x1047dcd50 called:3>
<Singleton@0x1047dcd50 called:4>
<Singleton@0x1047dcd50 called:5>
<Singleton@0x1047dcd50 called:6>
```

在instance()静态方法中直接返回了一个实例化对象, 而通过重写`__new__`方法则修改了对象的创建方式.

如此一来, 无论是通过instance方法还是直接创建, 都可以获得一个线程安全的单例对象.

> 需要注意的是, 每次获取单例对象的时候本`__init__`方法都会被执行, 对象的生命周期并没有发生改变, 只是把创建对象的内存分配方式进行了修改.
>
> 也就是说, 如果向初始化对象传递参数, 那么单例对象会被重复初始化, 并且状态停留在最后一次初始化的状态下. 而之前的初始化信息则会丢失.
>
{: .prompt-warning}

### 通过metaclass来实现单例

基于metaclass来实现单例, 同样是通过修改初始化方式. 在python中对象实例的创建流程大致如下图所示:

![img](/assets/img/img_202311241307079570.png)

本质上来说, 代码`Foo()`实际上是调用了metaclass也就是type的`__call__`方法.

而type的__call__方法则是调用Foo继承自object或者重写的`__new__`方法获得一个新的空对象.

随后再的调用Foo的`__init__`方法, 对空对象进行初始化, 最后将初始化完成的对象返回给调用者.

所以, 实际返回实例化对象的方法是type.__call__, 而不是类的__init__.

通过自定一个type类的子类, 重写其`__call__`方法, 便可以实现单例模式.

修改后的流程大致如下图所示:

![img](/assets/img/img_202311241434531606.png)

python实现代码:

```python
import threading


class SingletonMeta(type):

    __instance_lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            with SingletonMeta.__instance_lock:
                if not hasattr(cls, '_instance'):
                    obj = object.__new__(cls)
                    cls.__init__(obj, *args, **kwargs)
                    cls._instance = obj
        return cls._instance


class Singleton(metaclass=SingletonMeta):

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f"<{self.name}@{hex(id(self))}>"


if __name__ == "__main__":

    def task(n):
        s = Singleton(f"s{n}")
        print(s)

    for i in range(3):
        threading.Thread(target=task, args=(i,)).start()
```

输出结果:

```bash
<s0@0x100db4a90>
<s0@0x100db4a90>
<s0@0x100db4a90>
```

## 总结

虽然python实现单例的方式有很多, 但是总的来说还是觉得没有java简单便利.

从几种放发的对比上来说, 最简单的莫过于通过模块实现.

从重用的角度来说, 装饰器和元类模式是最好的.

通过类方法实现在安全性上略差, 无法避免使用过程中误操作的实例化的潜在问题.

通过__new__方法则是在重用性上有比较大的短板.

综合来说, 直接通过模块实现单例, 或者通过元类来实现单例效果最佳.
