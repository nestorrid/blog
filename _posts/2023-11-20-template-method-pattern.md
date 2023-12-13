---
title: 设计模式-模板方法模式
date: 2023-11-20 04:52:12 +0800
categories: [笔记, 设计模式]
tags: ['design pattern']
mermaid: true
---

## 应用场景

在处理一些大致内容相同, 但是有些细微差距的问题时可以使用模板方法模式. 基于同一套模板下, 不同的具体实现只针对细节发生变化, 而其他的内容则是依照父类模板执行.

举例来说, 一个代驾司机的标准工作流程大致可分为:

1. 接单
2. 到达指定车主位置
3. 代驾到指定目的地
4. 收钱
5. 要好评,再见~

那么区别在哪? 在于面对不同级别的车子时的心理状态~

顶级豪车得毕恭毕敬, 百万豪车是尊崇有加, 高端家用得谦卑有礼, 低档代步...

所以模板方法模式...精髓在于见人下菜碟, 典型的势利眼模式...

简单的python实现:

```python
import abc


class SubstituteDriver(abc.ABC):

    def take_order(self):
        print("接到客户订单")
        print("赶往指定地点")
        self._inner_thought()
        print("代驾到指定目的地")
        print("收钱")
        print("要好评, 再见~")

    @abc.abstractmethod
    def _inner_thought(self):
        pass


class SubstituteDriverForAudi(SubstituteDriver):

    def _inner_thought(self):
        print("得多要钱")


class SubstituteDriverForAlto(SubstituteDriver):

    def _inner_thought(self):
        print("一脸嫌弃")


if __name__ == "__main__":
    audi = SubstituteDriverForAudi()
    audi.take_order()

    alto = SubstituteDriverForAlto()
    alto.take_order()
```

本质上来说, 模板方法模式不一定要基于抽象类, 而是将流程进行明确的划分, 让每一步流程都有自己独立的处理方法. 子类要做的则是继承模板类后, 根据实际情况重写父类的方法以满足自身需求即可.
