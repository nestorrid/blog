---
title: 常用开源库整理(持续更新...)
date: 2023-11-14 18:37:23 +0800
categories: [速查, 工具]
tags: [python, django, opensource]
pin: true
image: /assets/img/img_202311141802036688.png
---

## Python开源框架

### [django](https://www.djangoproject.com)

![django logo](/assets/img/48223734b31de65181a4a38de6d8ac54.png)

功能强大的主流python服务端开发框架, 即可以基于MTV(Model-Templates-View)模式进行完整站点开发, 也可以用于开发RESTful API.
[官方文档](https://docs.djangoproject.com/en/4.2/)

![sep](/assets/img/separator_1.png)

#### [mysqlclient](https://github.com/PyMySQL/mysqlclient)

python访问mysql数据库的支持库, 基于mysql的django项目需要安装

![sep](/assets/img/separator_1.png)

#### [django-rest-framework](https://github.com/encode/django-rest-framework/tree/master)

![dsf-logo](https://www.django-rest-framework.org/img/logo.png)

基于django进行RESTful API开发的必备框架, 可以极大的提升开发效率

[官方文档](https://www.django-rest-framework.org)

![sep](/assets/img/separator_1.png)

#### [django filter](https://github.com/carltongibson/django-filter)

可以动态的读取URL参数, 并修改Django的Queryset来实现数据筛选

[官方文档](https://django-filter.readthedocs.io/en/main/)

![sep](/assets/img/separator_1.png)

#### [drf nested routers](https://github.com/alanjds/drf-nested-routers)

依赖于Django和Django REST Framework, 快速创建嵌套格式的RESTAPI, 如

```text
GET store/products: 产品清单
GET store/products/1: 产品1的详情
GET store/products/2/reviews: 产品2的评论清单
GET store/products/2/reviews/3: 产品2的第3条评论
```

没有独立的文档页面, Github抽风时可以访问[Gitee镜像](https://gitee.com/mirrors_alanjds/drf-nested-routers)

![sep](/assets/img/separator_3.png)

## 测试与调试

### [django-debug-toolbar](https://github.com/jazzband/django-debug-toolbar)

一款可重用app, 通过简单配置实现对接口的调试, 可以方便的查看变量, SQL语句, 静态资源, 模板等一系列相关资源.

![preview](/assets/img/img_202311141520211655.png)
__django-debug-toolbar边栏预览__

[官方文档](https://django-debug-toolbar.readthedocs.io/en/latest/)

由[Jazzband](https://github.com/jazzband) python社区提供, 除了ddt之外, 该社区还提供了大量其他高价值的开源python框架, 如

* [django-redis](https://github.com/jazzband/django-redis): redis缓存管理后台
* [django-oauth-toolkit](https://github.com/jazzband/django-oauth-toolkit): 用于Django项目OAuth2认证的工具
* [djangorestframework-simplejwt](https://github.com/jazzband/djangorestframework-simplejwt): Drango REST Framework的JWT(Json Web Token)插件

![sep](/assets/img/separator_1.png)

### [pytest](https://github.com/pytest-dev/pytest)

强大的Python单元测试框架, 比python自带的unittest更为简单易用, 同时拥有大量的测试插件以满足不同场景的测试需求, 如:

* [pytest-django](https://github.com/pytest-dev/pytest-django): 用于测试Django项目
* [pytest-html](https://github.com/pytest-dev/pytest-html): 用于生成HTML测试报告
* [pytest-flask](https://github.com/pytest-dev/pytest-flask): 用于测试flask项目
* [pytest-selenium](https://github.com/pytest-dev/pytest-selenium): 用于结合selenium自动化脚本运行pytest

![sep](/assets/img/separator_1.png)

### [pytest-cov](https://github.com/pytest-dev/pytest-cov)

用于生成代码测试覆盖率报告.

[官方文档](https://pytest-cov.readthedocs.io/en/latest/)

![sep](/assets/img/separator_1.png)

### [pytest-django](https://github.com/pytest-dev/pytest-django)

基于pytest的django测试框架, 可以方便的编写自动化接口测试.

[官方文档](https://pytest-django.readthedocs.io/en/latest/)

![sep](/assets/img/separator_1.png)

### [tox](https://github.com/tox-dev/tox)

提供自动化的虚拟环境测试, 依赖测试, 单元测试, 持续集成测试等.

[官方文档](https://tox.wiki/en/latest/index.html)

![sep](/assets/img/separator_1.png)

### [locust](https://github.com/locustio/locust)

性能测试, 压力测试的开源框架, 通过python代码定义用户行为模拟数百万用户操作, 并给出对应的分析报告.
![screenshot](https://www.locust.io/static/img/screenshot_2.13.1.png)

[官方文档](https://docs.locust.io/en/stable/)

![sep](/assets/img/separator_1.png)

### [django-silk](https://github.com/jazzband/django-silk)

一个Django数据分析工具, 可以记录接口的调用, 返回时间, 产生的数据查询数量等用于分析.

![preview](https://raw.githubusercontent.com/jazzband/django-silk/master/screenshots/1.png)

由于没有独立的文档页, 如果Github抽风可以访问[Gitee镜像](https://gitee.com/mirrors_jazzband/django-silk)

![sep](/assets/img/separator_3.png)

## 在线工具

### [https://mockaroo.com](https://mockaroo.com)

一款免费的在线数据生成工具, 可以根据表结构随机生成数据, 提供CSV, SQL等多种格式的下载. 注册账户后还可以使用创建项目, 保存表结构等功能, 方便后续使用.

![sep](/assets/img/separator_1.png)

### [https://realfavicongenerator.net](https://realfavicongenerator.net)

站点的favicon生成器, 通过一张图片生成一套跨平台的通用图片集

![sep](/assets/img/separator_1.png)

### [draw.io](http://draw.io)

免费的在线绘图软件, 可以绘制各类常用图, 包括流程图, 线框图, ER图, 结构图等等.

也可以[获取桌面版](http://get.draw.io)

![preview](/assets/img/img_202311151738162426.png)

![sep](/assets/img/separator_1.png)

### [https://www.lucidchart.com](https://www.lucidchart.com/)

同样是一款在线绘图工具, 目前没有桌面版app, 相较draw.io来说有更好的UI, 在分享与协作方面功能更强, 也可以基于ER图导出DDL等.

但高级功能需要付费使用, 价格大概是Figma的一半, 如果仅有绘图需求ddraw.io基本足够使用

### [pixso](https://pixso.cn)

基本上复制了Figma模式和功能的国产设计软件, 包含了Figma和FigJam的核心功能, 目前还是免费试用.

在桌面应用上的流畅性相较Figma而言有差距, 社区资源也相对较少, 但足够满足使用需求.

![sep](/assets/img/separator_1.png)

### [cssgradient.io](https://cssgradient.io)

在线生成渐变效果的css工具.

同时还有专门的插画, 阴影编辑工具.

![sep](/assets/img/separator_1.png)

### [www.fontsquirrel.com](https://www.fontsquirrel.com)

字体站, 可以找到大量的免费开源字体, 同时可以将字体压缩为web字体.

![sep](/assets/img/separator_1.png)

### [typescale.com](https://typescale.com)

响应式站点的相对字体大小单位计算站点. 快速得到准确的rem数值.

![sep](/assets/img/separator_1.png)

### [bennettfeely.com](http://bennettfeely.com)

在线切图工具, 可以把图片切成所需要的形状.

![sep](/assets/img/separator_1.png)

### [cssspritestool.com](http://cssspritestool.com)

css图片合集在线制作工具

![sep](/assets/img/separator_1.png)

### [responsivebreakpoints.com](http://responsivebreakpoints.com)

响应式图片集在线生成工具

![sep](/assets/img/separator_1.png)

### [cloudconvert.com](https://cloudconvert.com)

文件格式在线转换工具

![sep](/assets/img/separator_1.png)

### [svgbackgrounds.com](http://svgbackgrounds.com)

svg背景图生成网站

![sep](/assets/img/separator_1.png)

### [cubic-bezier.com/](https://cubic-bezier.com/)

css动画函数曲线生成器.

![sep](/assets/img/separator_1.png)

### [animate.style](http://animate.style)

css动画样式库

![sep](/assets/img/separator_1.png)

### [yesno.wtf](https://yesno.wtf)

一个简单的api测试站点, 调用`/api`得到一个json对象的数据, 包含一个随机的yes,no,maybe结果以及一个gif动图地址...从动图的尿性来说...wtf域名选的很贴切...

![sep](/assets/img/separator_1.png)

### [brandcolors.net](https://brandcolors.net/)

品牌色板, 各类品牌的配色方案集合.

![sep](/assets/img/separator_1.png)

### [colors.muz.li](https://colors.muz.li/)

根据上传的图片自动分析出对应的配色方案与色板文件, 相当nice!!!

![sep](/assets/img/separator_1.png)

### [color.hailpixel.com](https://color.hailpixel.com/)

快速创建色板的在线工具, 简单上手到无以复加...

![sep](/assets/img/separator_1.png)

### [zhongguose.com](https://zhongguose.com/)

中国风配色查询网站, 访问速度快, 颜色也很舒服.
