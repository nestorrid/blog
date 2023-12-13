---
title: Jekyll创建静态博客
date: 2023-11-11 14:02:00 +08000
categories: [笔记, jekyll]
tags: [jekyll]
---

## 安装依赖包

1. [Ruby](https://www.ruby-lang.org/en/downloads/)
2. [RubyGems](https://rubygems.org/pages/download)
3. [GCC](https://gcc.gnu.org/install/)和[Make](https://www.gnu.org/software/make/)

具体安装引导参见[官方文档](https://jekyllrb.com/docs/installation/)

## 安装jekyll

```bash
   gem install jekyll bundler
```

> 在安装jekyll时遇到了ruby版本问题, 执行了一下重装命令, 通过brew升级即可
>
> ```bash
> brew install ruby
> ```

## 创建Jekyll网站

可以是一个已经存在的文件夹, 通过`git clone`创建的文件夹同样可以创建jekyll项目

```bash
    jekyll new my_site
```

实话实说, 这一步运行速度感人...

## 进入jekyll目录

```bash
    cd my_site
```

## 启动本地服务

```bash
    bundle exec jekyll serve
```
