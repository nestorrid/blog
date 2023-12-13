---
title: 设计模式-迭代器模式
date: 2023-11-21 12:53:30 +0800
categories: [笔记, 设计模式]
tags: ['design pattern']
mermaid: true
---

## 应用场景

迭代器用于对数据的遍历. 最主要的作用便是对外界隐藏内部的数据结构和遍历方式, 只需要暴露一个统一的遍历方法, 从而解耦外部对于自身数据结构的依赖. 即便在内部修改数据结构或者实现方式也不会对外部代码产生影响.

假设一个需求, 存在一个数据集, 按照表的形式存储数据. 即包含多行数据, 每行包含数个数据列.

在外部需要对表格内的数据进行遍历. 每次仅获取一个单元格的数据, 从首行首列开始直至最后.

虽然没太大实际意义, 但可以用来掩饰迭代器模式.

## 简单的python实现

```python
import random


class DataSource:

    def __init__(self):
        self.__data = [[random.randint(0, 100)
                        for _ in range(2)] for _ in range(2)]
        self.__current_index = 0
        self.__element_count = self.__count_elements()
        self.__column_count = len(self.__data[0])

    def has_next(self):
        return self.__current_index < self.__element_count

    def next(self):
        x = self.__current_index // self.__column_count
        y = self.__current_index % self.__column_count
        self.__current_index += 1
        return self.__data[x][y]

    def __iter__(self):
        return self

    def __next__(self):
        if self.has_next():
            x = self.__current_index // self.__column_count
            y = self.__current_index % self.__column_count
            self.__current_index += 1
            return self.__data[x][y]

        self.__current_index = 0
        raise StopIteration

    def __count_elements(self):
        count = 0
        for lst in self.__data:
            count += len(lst)
        return count


if __name__ == "__main__":
    ds = DataSource()

    # 基于`__iter__` 和 `__next__` 两个内置魔术方法实现遍历
    for n in ds:
        print(n)

    print("---")

    # 基于两个自定义方法实现遍历
    while ds.has_next():
        print(ds.next())

```
