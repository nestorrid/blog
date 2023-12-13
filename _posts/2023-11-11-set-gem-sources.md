---
title: 修改gem国内源
date: 2023-11-11 13:56:48 +08000
categories: [速查, 配置]
tags: [gem, config]
---

默认gem源可能出现下载速度太慢的问题, 可以直接改为国内源:

```bash
gem sources --remove https://rubygems.org/
gem sources -a https://mirrors.ustc.edu.cn/rubygems/
gem sources -l
gem sources -u
```

> 如果安装了`bundle` 可以同时将bundle的源一同改为国内源
>
> ```bash
> bundle config mirror.https://rubygems.org https://mirrors.ustc.edu.cn/rubygems/
> ```

## 可选源

* 腾讯源: `https://mirrors.cloud.tencent.com/rubygems/`
* ruby-china: `https://gems.ruby-china.com/`
* 中科大: `https://mirrors.ustc.edu.cn/rubygems/`
