---
title: 设计模式-享元模式
date: 2023-11-23 19:24:38 +0800
categories: [笔记, 设计模式]
tags: ['design pattern']
mermaid: true
---

也叫轻量级模式, 轻量模式.

## 问题描述

属于概念比较好理解的模式. 核心思想就是把高内存占用的只读资源共享, 使其能够在多个位置进行重用.

在不重用的情况下, 每个元素独占一个图片资源, 当重复元素变多就会由于系统资源不足而崩溃.

最常见的属于各类App中的图片素材, 比如按钮底图, 占位符图片. 比较形象的就是扫雷游戏里那一屏幕密密麻麻的方块图片.

## 简单的python实现

```python
from pathlib import Path
from enum import Enum
import tracemalloc


class Point:

    def __init__(self, x, y, icon):
        self.x = x
        self.y = y
        self.icon = icon

    def draw(self):
        print(f">> draw point at: ({self.x},{self.y}) with image:{self.icon}")


class IconType(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3


class PointIcon:

    def __init__(self, icon_type, path):
        self.icon_type = icon_type
        if path:
            self.icon = Path(path).read_bytes()


class IconFactory:

    __singleton = None

    def __init__(self):
        self.__icons = {}
        IconFactory.__singleton = self

    @staticmethod
    def instance():
        if IconFactory.__singleton is None:
            IconFactory.__singleton = IconFactory()

        return IconFactory.__singleton

    def get_icon(self, icon_type, path):
        # key = f"icon-type-{icon_type.name}-{icon_type.__hash__()}"
        if icon_type in self.__icons:
            return self.__icons[icon_type]

        self.__icons[icon_type] = Path(path).read_bytes()


if __name__ == "__main__":

    def test_for_100_points():
        # pathlib.py:1051: size=8941 KiB, count=100, average=89.4 KiB
        # flyweight_pattern.py:44: size=24.8 KiB, count=401, average=63 B
        # pathlib.py:508: size=48 B, count=1, average=48 B
        tracemalloc.start()

        points = [Point(i, i, PointIcon(IconType.RED, "temp.jpg"))
                  for i in range(0, 100)]

        snapshot = tracemalloc.take_snapshot()
        stats = snapshot.statistics('lineno')

        for stat in stats:
            result = str(stat).rsplit('/', 1)[-1]
            print(result)

        tracemalloc.stop()

    def test_for_100_points_with_flyweight():
        tracemalloc.start()

        icon = IconFactory.instance().get_icon(IconType.RED, "temp.jpg")
        points = [Point(i, i, icon) for i in range(0, 100)]

        snapshot = tracemalloc.take_snapshot()
        stats = snapshot.statistics('lineno')

        for stat in stats:
            result = str(stat).rsplit('/', 1)[-1]
            print(result)

        tracemalloc.stop()

test_for_100_points()
print('---')
test_for_100_points_with_flyweight()
```

最终输出结果:

```bash
pathlib.py:1051: size=8941 KiB, count=100, average=89.4 KiB
flyweight_pattern.py:71: size=24.8 KiB, count=401, average=63 B
flyweight_pattern.py:37: size=48 B, count=1, average=48 B
pathlib.py:508: size=48 B, count=1, average=48 B
---
pathlib.py:1051: size=89.4 KiB, count=1, average=89.4 KiB
flyweight_pattern.py:87: size=10.2 KiB, count=201, average=52 B
flyweight_pattern.py:51: size=320 B, count=2, average=160 B
flyweight_pattern.py:60: size=160 B, count=1, average=160 B
```
