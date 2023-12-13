---
title: 使用jekyll框架基于chirpy主题与gitee建站
date: 2023-11-11 14:04:24 +08000
categories: [笔记, jekyll]
tags: [jekyll, gitee]
---

## 选择模板

jekyll框架存有很多优质模板, 可以在以下链接查阅:
[http://jekyllthemes.org/](http://jekyllthemes.org/)

本次建站是用[jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy.git)主题.

由于主题本身基于`Node.js`制作, 需要提前安装[Node.js](https://nodejs.org/en)

在`Gitee`中创建仓库,并从Github中导入`jekyll-theme-chirpy`, 导入时可以直接改名为自己希望的名字, 随后克隆到本地

```bash
git clone https://gitee.com/nestalk/noteblog.git
cd noteblog
```

运行主题初始化脚本, 自动移除不相关的示例文件以及Git配置, 创建项目所需的js文件

```bash
bash tools/init
```

运行bundle安装依赖:

```bash
bundle 
```

启动本地服务:

```bash
bundle exec jekyll serve
```

## 配置主题

`_config.yml`:

```yml
lang: zh-CN
title: Nestor's noteblog # the main title
tagline: Design can be art.<br/> Design can be simple.<br/> That's why it's so complicated. # it will display as the sub-title
description: >- # used by seo meta and the atom feed
  技术学习笔记, 如: Python, Django等等. 以及个人博客文章.
social:
  # Change to your full name.
  # It will be displayed as the default author of the posts and the copyright owner in the Footer
  name: Nestor
  email: admin@nestor.me # change to your email address
  links:
    # The first element serves as the copyright owner's link
    - https://gitee.com/nestalk # change to your github homepage
    - https://weibo.com/u/1239998601 # change to your twitter homepage  
theme_mode: dark # [light|dark]    
avatar: /assets/img/avatar.png
```

关于分享的一些设置还没研究透, 留下几个问题:

* git链接 -> gitee链接
* Twitter -> weibo

## 添加文章

添加文章始终应该在`_post`目录下, 直接创建文件即可, 不需要额外的文件夹.
但是文件的命名规则是固定的:

`YYYY-MM-DD-TITLE.md`

### 添加头文件

```yaml
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```

* `categories` 最多包含两项, 但是`tag`几个都行
* `math`: 数学公式功能, 出于性能因素默认关闭, 但是可以在需要的页面上手动打开 `math: true`, 基于`LaTex_math`实现, 相关[拓展资料](https://www.jianshu.com/p/0ea47ae02262)
* `toc`: 是否显示目录, 默认为`true`, 可以手动设置为`false`, 一般没必要关闭.
* `comments`: 是否开启评论功能, 如果开启需要一个支持评论的后台系统, 需要的时候在研究. 对于已有评论系统的可以手动设置`false`关闭
* `mermaid`: 流程图插件, 需要使用时标记为`true`即可, [mermaid文档](http://mermaid.js.org/intro/)
* `img_path`: 当前md文件中的所有图片都基于该路径进行查找
* `images:`: 封面图, 可以为当前文章添加一个封面图片, 要求尺寸为`1200 * 630`, 可以仅提供一个路径, 或者进行完整设置:

```yaml
---
image:
    path: /path/to/image
    alt: image alternative text
---
```

* `pin`: 是否置顶文章

为了简化操作, 在vscode中制作了一个专门的code snippet来解决这一问题:

``` json
    "front matter for markdown" : {
        "scope": "markdown, md",
        "prefix": "matter",
        "body": [
            "---",
            "title: '${1: title}'",
            "date: '${2: YYYY-MM-DD HH:MM:SS}' +08000",
            "categories: ['${3: first category, second category}']",
            "tags: ['${4: tags}']",
            "---"
        ],
        "description": "Generate Front Matter data for jekyll markdown post files."
    }
```

### 调整图片

设置图片标题:

```markdown
![img-description](/path/to/image)
_Image Caption_
```

设置图片大小:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: width="700" height="400" }
或者
![Desktop View](/assets/img/sample/mockup.png){: w="700" h="400" }
```

调整图片位置:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .normal }
![Desktop View](/assets/img/sample/mockup.png){: .left }
![Desktop View](/assets/img/sample/mockup.png){: .right }
```

明暗适配:

```markdown
![Light mode only](/path/to/light-mode.png){: .light }
![Dark mode only](/path/to/dark-mode.png){: .dark }
```

添加阴影:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```

### 提示信息

Chirpy集成了Prompts插件, 可以为段落引用添加样式, 如:

```markdown
> Example line for prompt.
{: .prompt-info }
```

可选的提示样式包括`tip`, `info`, `warning`, `danger`

### 修改Favicon

[Favicon在线制作工具](https://realfavicongenerator.net)

上传一个正方形图片后会生成一个favicons文件夹, 下载后删除解压包中的文件:

* browserconfig.xml
* site.webmanifest

将剩余的图片复制到`/assets/img/favicons`文件夹下进行覆盖即可.
