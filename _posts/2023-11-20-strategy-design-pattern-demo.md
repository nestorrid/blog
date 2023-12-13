---
title: 设计模式-策略模式
date: 2023-11-20 04:08:39 +0800
categories: [笔记, 设计模式]
tags: ['design pattern']
mermaid: true
---

## 应用场景

策略模式与状态模式很像, 都属于行为模式. 从最终呈现的UML类图上来看可能两者没什么太大区别.

区别在于, 状态模式主要目的是让主体可以依据不同的状态确定行为. 比如不同的工具执行不同的操作.

策略模式则主要用来封装算法, 主体状态并没有变化, 但是可以自由组合要执行的行为方式.

举例来说, 编写一个简单的图片文件处理程序, 需要将给定的图片进行压缩, 然后为其添加滤镜.

```python
import abc


class ImageCompresser(abc.ABC):

    @abc.abstractmethod
    def compress(self, name):
        pass


class PngCompresser(ImageCompresser):

    def compress(self, name):
        print(f"compressing png file {name}")


class JpegCompresser(ImageCompresser):

    def compress(self, name):
        print(f"compressing jpeg file {name}")


class ImageFilter(abc.ABC):

    @abc.abstractmethod
    def apply(self, name):
        pass


class BlackAndWhiteFilter(ImageFilter):

    def apply(self, name):
        print(f"apply black and white filter for {name}")


class BlurFilter(ImageFilter):

    def apply(self, name):
        print(f"apply blur filter for {name}")


class ImageHandler:

    def handle_image(self, name,
                     *filters: ImageFilter,
                     compressor: ImageCompresser = None):
        if compressor:
            compressor.compress(name)

        for img_filter in filters:
            img_filter.apply(name)

        print(f"saving image:{name}")
        print("done.")


if __name__ == "__main__":

    handler = ImageHandler()
    handler.handle_image("some image",
                         BlackAndWhiteFilter(), BlurFilter(),
                         compressor=PngCompresser())

```
