---
title: Pipenv 安装 mysqlclient出错
date: 2023-11-11 14:11:02 +08000
categories: [速查, 异常]
tags: [django, pip, pipenv, mysql, exception]
---

在使用`pipenv install mysqlclient`时报错:

```bash
RuntimeError: Failed to lock Pipfile.lock!
```

本来以为是mysqlclient或者pipenv的问题, 反复尝试无果, 又随便找了几个其他的包尝试安装, 包括`requests`,`pydoc`等

发现依然会出现同样的问题,但是像`django`等一些包就安装很顺利

最后找到一些解决方案, 需要安装一个名为`pkg-connfig`的包,

```zsh
sudo apt-get install pkg-config
```

但是Mac 并不支持`apt-get`

于是只能换一种思路, 使用`brew`来安装`pkg-config`

经尝试

```zsh
brew install pkg-config
```

运行完成之后`mysqlclient`便可以通过pipenv正常安装了

> PS: brew安装:
>
> ```bash
> /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
> ```
