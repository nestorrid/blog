---
title: Docker笔记
date: 2023-11-15 22:26:58 +0800
categories: [笔记, Docker]
tags: [docker, linux, shell]
image: /assets/img/img_202311152245318150.png
---

## 安装docker

[Docker文档](https://docs.docker.com/get-docker/)

文档链接进入下载页面, 选择对应的系统下载安装即可.

比较难受的是docker注册网站需要科学上网才能访问.

安装并运行后, 终端执行以下命令查看是否成功:

```bash
docker version
```

## 理解Docker的IMAGE和CONTAINER

* `Image`: 包含了一个简化的操作系统, 比如`ubuntu`, 第三方库, 项目文件 环境变量等内容;
* `Container`: 一个独立封装的虚拟环境.

## Docker 基础命令

* `docker ps`: 显示所有正在运行的容器
  * `docker ps -a`: 显示所有容器
* `docker start -i <id | name>`: 运行指定的容器, 可以是容器id或者容器别名. 如果使用id, 输入没有重名的前几位就可以, 别名则需要全名.
* `docker run -it <image>`: 基于`image`在交互模式下创建新的`container`
* `docker run -d <image>`: 在后台运行image
* `docker run -d -p <host_port>:<container_port> <--name container_name> <image>`:在后台运行image, 创建指定名称的container, 并创建主机到容器的端口映射.
* `docker exec <container_name> <linux_cmd><`: 在正在运行的容器中执行指令.
* `docker image ls`: 列出所有本地镜像
* `docker image rm <id | name>`: 删除指定的镜像
* `docker container prune`: 删除所有停止的容器
* `docker image prune`: 删除所有未使用的镜像
* `docker build -t <image_name:tag> <path_to_Dockerfile>`: 创建本地镜像
* `docker cp <source> <dist>`: 在容器与主机之间进行文件复制, 容器路径需要使用`container:path`主机路径则直接指定路径即可.

## Dockerfile

### 选择合适的基础镜像

Docker的配置文件, 没有扩展名的文本文件. 直接在项目根目录下创建并编辑即可

需要为项目确定一个基础镜像, 详情可以查看[官方文档](https://docs.docker.com/samples/)

在`Dockerfile`中指定镜像:

```text
FROM node:14.16.0-alpine3.13
```

### 复制文件到镜像文件系统

通过`COPY`或者`ADD`命令可以将特定的文件内容放入docker镜像

```text
COPY . /app
```

或者

```text
WORKDIR /app
COPY . .
```

> 与shell的`cp`命令不同.
>
> 在`Dockerfile`中, 使用COPY命令的第一个`.`代表的是配置文件所在的目录.
>
> 而`/app`代表的是镜像文件的目录系统, 源文件路径和目标文件路径分属两个不同的文件系统.
>
> 通过`WORKDIR`指定是镜像中的工作目录
>
> 所以`COPY . .`的人话就是`将Dockerfile所在目录的全部分件复制到镜像中的工作目录下`
>
{: .prompt-tip}

### 忽略特定内容

配置docker忽略的文件`.dockerignore`, 与`.gitignore`文件类似

```text
node_modules/
```

### 运行命令

在部署docker镜像之后要执行的命令, 通常用来安装依赖等.

```text
RUN npm install
RUN apt install python
...
```

> 配合运行命令, 可以在打包镜像时忽略依赖库文件, 而在运行镜像时进行安装, 如此便可以极大的减小镜像文件所占用的空间.
>
{: .prompt-tip}

### 设置环境变量

直接在镜像环境中创建环境变量, 便可以直接在`shell`中进行使用.

```text
ENV API_URL=https://some.url
```

### 开放端口

指定容器的监听端口, 似的外部请求可以通过该端口与程序建立连接.

```text
EXPOSE 3000
```

> 该端口只是容器监听的端口, 而不是主机监听的端口. 在主机访问该端口对应的程序时需要设置端口映射.
>
> ```bash
> docker run -d -p 8080:3000 my_image
> ```
>
> 此时访问`127.0.0.1:8080`时就会映射到容器中监听3000端口的程序.
>
{: .prompt-info}

### 修改用户

默认情况下, docker在运行容器时使用的是`root`用户, 也就包含了全部权限, 出于安全考虑, 可以为镜像指定特定的用户和组以做权限管理.

```text
RUN addgroup app && adduser -S -G app app
```

> 在基于该指令创建镜像时出错, 可能是基础镜像的问题, 修改命令后正常:
>
> ```text
> RUN addgroup app
> RUN adduser --system app
> RUN adduser app app
> ```
>
{: .prompt-tip}

### 启动指定项目

通过`CMD`或`ENTRYPOINT`来指定镜像启动时的执行命令.

```text
CMD npm start
CMD ["npm","start"]
ENTRYPOINT npm start
ENTRYPOINT ["npm", "start"]
```

> 1. 使用["npm", "start"]可以避免额外启动一个shell进程
> 2. CMD和ENTRYPOINT的区别在于通过docker启动时直接填写命令可以替换CMD指令
> 3. RUN和CMD的区别在于RUN是在构建镜像时执行, CMD是在启动镜像时执行.
>
{: .prompt-tip}

### 优化docker镜像构建流程

docker在构建镜像时会为`Dockerfile`中的每一行配置创建一个单独的`layer`, 在重新构建时, 如果上一层`layer`没有任何变化则会直接使用之前的内容.

在之前的配置文件中, 通过`COPY . .`将文件复制到了镜像中, 然后`RUN npm install`.

```text
COPY . .
RUN npm install
```

如果这样配置, 在进行任何文件内容修改, 如修改了一行代码, 也会导致文件内容变动, docker就只能重建镜像的文件系统, 也就是`COPY . .`这一层, 而在安装依赖之前的层发生了变动, 依赖就会被重新安装.

因为依赖文件通常单独储存, 以node项目为例, 项目依赖都储存在`package*.json`中, 所以可以进行优化:

```text
COPY package*.json .
RUN npm install
COPY . .
```

此时构建镜像之后, 随便修改一个文件, 然后重新构建, 可以看到构建记录显示:

![result](/assets/img/img_202311162146456205.png)

由于`package*.json`文件没有变动, 直接通过缓存完成了复制依赖配置与安装依赖的步骤. 而构建速度也得到了极大的提升.

因此, 在进行`Dockerfile`配置时, 不经常发生变动的内容应当优先配置, 而变动发生较多的内容则最后配置.

## 设置镜像标签

可以直接在构建镜像时直接设置标签:

```bash
docker build -t image_name:tag_name .
```

为已有的镜像添加tag:

```bash
docker image tag my_image:1 my_image:2
```

删除指定的标签:

```bash
docker image remove my_image:2
```

> 如果镜像只有一个标签, 那么会直接删除镜像
>
{: .prompt-tip}

## 打包和安装镜像

除了通过Docker hub上传和加载镜像之外, 也可以直接将镜像进行打包, 并在其他的docker环境中载入.

* 将指定的镜像文件打包:

  ```bash
  docker image save -o package.tar test_node:latest
  ```

* 载入指定的镜像文件:

  ```bash
  docker image load -i package.tar
  ```

## 创建主机到容器的文件映射

在运行容器时可以将项目文件夹映射到容器之中, 这样在修改项目源码时容器中的内容也会一同更新, 从而省去了重新构建镜像的操作

```bash
docker run -d -p 8080:3000 -v $(pwd):/app my_image
```
