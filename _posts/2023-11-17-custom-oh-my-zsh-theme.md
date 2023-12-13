---
title: 自定义终端主题
date: 2023-11-17 19:17:12 +0800
categories: [笔记, 杂项]
tags: [zsh, theme, ⭐️]
image: /assets/img/img_202311180006318654.png
---

## 安装Oh-my-zsh

1. 使用`zsh`作为终端shell程序, 可以通过命令设置:
  ```bash
  chsh -s /bin/zsh
  ```

2. 安装Oh-my-zsh:
  ```bash
  sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
  ```

3. 安装`Powerline`, 一款状态栏工具, 可以梅花终端和vim界面.
  ```bash
  pip install powerline-status
  ```

4. 安装Powerline的字体库, 依次运行命令:
  ```bash
  # clone
  git clone https://github.com/powerline/fonts.git --depth=1
  # or use gitee mirror when git is unable to access
  git clone https://gitee.com/nestalk/fonts.git --depth=1
  # install
  cd fonts
  ./install.sh
  # clean-up a bit
  cd ..
  rm -rf fonts
  ```

5. 更换字体, 在终端, vscode中修改终端的字体为`Meslo LG`
   * 终端: 设置 -> 描述文件 -> 文本 -> 字体
   * VSCODE: cmd+shift+p -> settings.json:
     ```json
     {
      "editor.fontFamily": "Menlo, Monaco, 'Courier New', monospace, 'Meslo LG'"
     }
     ```

     > 事实上vscode不修改字体也没太大问题, 改了有时候反倒不舒服. 如果存在乱码可以通过自定义主题替换一下乱码字符.
     >
     {: .prompt-tip}

## 设置ohmyzsh主题

oh-my-zsh内置了很多主题, 保存在目录`~/.oh-my-zsh/themes`下.

也可以访问[github 主题页](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)获取更多主题.

额外的[社区主题仓库](https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes)

通过`vim ~/.zshrc`命令修改配置文件, 可以指定主题等.

## 设置语法高亮

安装`zsh-syntax-highlighting`插件.

```bash
brew install zsh-syntax-highlighting
```

在Mac上配置语法高亮插件:

```bash
echo "source $(brew --prefix)/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```

> 不同系统配置有所区别, 具体可参照[插件文档](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)
>
{: .prompt-tip}

## 添加代码补全插件

提供模糊代码补全的插件`zsh-completions`.

```bash
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
```

在`~/.zshrc`文件中添加插件

```text
plugins=(   
    # other plugins...
    zsh-completions
)
```

## 添加历史命令补全插件

通过历史命令自动补全的插件`zsh-autosuggestions`.

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

同样在`~/.zshrc`文件中添加插件:

```text
plugins=( 
    # other plugins...
    zsh-autosuggestions
)
```

## 自定义主题

折腾了不断实践, 最终结果还算满意, 基于内置的`agnoster`重新做了一下自定义.

在Mac终端的最终显示效果如下:

![preview](/assets/img/img_202311180103427767.png)

* `⌘`: 表示默认用户在本机登录, 如果通过如果
* `❖`: 表示文件路径, 就是个装饰
* `◌`: git默认前缀, 会根据仓库状态有所改变
* 红色部分代表处于虚拟环境下.

为了使主题的一些配置生效, 需要在`./zshrc`文件中配置两个变量

```python
DEFAULT_USER="your name"
VIRTUAL_ENV_DISABLE_PROMPT=false
```

之后复制一份内置主题的文件, 并在其基础上自由发挥了, 至于主题的改法, 基本都是搜索加试错扣出来的.

反正做好一次以后也就不用再折腾了, 时间也算花的值得.

完成之后把主题的配置文件备份在[gitee仓库](https://gitee.com/nestalk/ohmyzsh_custom_theme), 以后也就走不丢了.
