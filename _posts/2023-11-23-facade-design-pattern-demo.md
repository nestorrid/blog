---
title: 设计模式-外观模式
date: 2023-11-23 02:44:11 +0800
categories: [笔记, 设计模式]
tags: ['design pattern']
mermaid: true
---

## 问题描述

假设要实现一个推送消息到用户的功能. 大体流程是:

* 首先需要跟服务器建立连接
* 检查收否有身份验证的Token
* 如果没有则要通过授权信息向服务器请求Token. 
* 然后将Token,消息和目标用户的id一同发送给服务器
* 等待服务器返回发送结果
* 关闭连接

如下图所示:
![uml](/assets/img/img_202311230930147088.png)

显然这是一个比较复杂的复杂流程, 没有人会在不同的地方去维护这一堆随时有可能发生变化的方法.

## 解决方案

外观模式便是为了把一些列复杂的操作或者子系统封装为一个统一的对外接口, 使得使用者可以通过简单的操作获得所要的结果. 同时将内部的子系统对其屏蔽, 在子系统进行就该的情况下依然不会影响外部使用.

基本上现在的REST API都属于外观模式的体现.

就像车险经纪人, 那么多条款保项的咱也不懂, 他就把你的信息拿去, 依照他们的保险政策算出一套你能接受的他最赚的保险方案, 然后让你交钱. 告诉你办妥了, 放心开吧!

你猜怎么着? 来年还找你要钱! 又亏了一年~~

## 简单的Python实现

```python
class Connection:

    def __init__(self, server):
        self.__server = server

    def send(self, token, message, *clients):
        print(f"TOKEN: {token}")
        print(f"CLIENTS: {clients}")
        print(f"send message: `{message}` to server...")
        func = getattr(self.__server, "_NotificationServer__recive")
        func(token, message, *clients)

    def disconnect(self):
        print("disconnect notification server.")


class NotificationServer:

    def connect(self) -> Connection:
        conn = Connection(self)
        return conn

    def authenticate(self, appid, apikey):
        if appid and apikey:
            print("Authenticated")
            return "SECRET-TOKEN"

    def __recive(self, token, message, *clients):
        if token:
            for client in clients:
                print(
                    f"Server is send message: `{message}` to client: `{client}`")


class NotificationService:

    def __init__(self):
        self.__server = NotificationServer()
        self.__conn = None
        self.__token = None

    def send(self, message, *clients):
        self.__setup()
        self.__conn.send(self.__token, message, *clients)
        self.__disconnect()

    def __setup(self):
        self.__create_conn()
        self.__get_autentication()

    def __disconnect(self):
        self.__conn.disconnect()
        self.__conn = None

    def __create_conn(self):
        if self.__conn is None:
            print(">> Connecting to server...")
            self.__conn = self.__server.connect()
            print('>> Connection created.')

    def __get_autentication(self):
        if self.__token is None:
            self.__token = self.__server.authenticate("appid", "apikey")
            print(f">> Get token: `{self.__token}` from server.")


if __name__ == "__main__":
    service = NotificationService()
    service.send("天气不错", 111, 222, 333)

```

> 为了模拟服务器交互所以在本地做了个Connection和NotificationServer之间的引用.
>
> 实际应用中Connection必然是直接与API接口进行交互.
>
{: .prompt-info}

最终输出结果:

```bash
>> Connecting to server...
>> Connection created.
Authenticated
>> Get token: `SECRET-TOKEN` from server.
TOKEN: SECRET-TOKEN
CLIENTS: (111, 222, 333)
send message: `天气不错` to server...
Server is send message: `天气不错` to client: `111`
Server is send message: `天气不错` to client: `222`
Server is send message: `天气不错` to client: `333`
disconnect notification server.
```