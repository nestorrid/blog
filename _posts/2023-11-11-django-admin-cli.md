---
title: Django 常用命令
date: 2023-11-11 14:07:02 +08000
categories: [速查, 工具]
tags: [django]
---

## 创建django项目

```bash
django-admin startproject name
```

## 启动服务

```bash
django-admin runserver
python manage.py runserver [port]
```

## 创建应用

```bash
python manage.py startapp appname
```

## 创建数据迁移文件

```bash
python manage.py makemigrations
```

## 执行数据库迁移

```bash
python manage.py migrate
```
