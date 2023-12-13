---
title: 常用Linux命令
date: 2023-11-16 18:43:52 +0800
categories: [笔记, Docker]
tags: [docker, linux, shell]
---

## 基础命令

* `cd`: 修改文件夹
* `ls`: 列出当前文件夹下的内容, 可以使用参数:
  * `-a`: 显示隐藏文件
  * `-l`: 显示详情
  * `-1`: 每行一个
* `pwd`: 显示当前文件夹完整路径
* `cp`: 复制文件, `cp src dist`
* `mv`: 移动或重命名文件
* `rm`: 删除文件或文件夹, 常用参数
  * `-r`或`-rf`: 递归, 在处理文件夹时使用, 表示自动遍历子文件夹

## 文件操作

* `cat`: 连接文件, 也可用于显示单个文件的内容
  
  ```bash
  cat text.txt
  cat text1.txt > text2.txt
  cat text1.txt text2.txt
  cat text1.txt text2.txt > new.txt
  ```

  > `>` 用于修改输出源, 默认为`stdout`, 即显示器.
  >
  > 通过`> text2.txt`即可以将文件内容输出到新的文件之中.
  >
  > 包含输出结果的命令都可以与 `>` 连用, 比如:
  >
  > `echo something > file.txt`
  >
  {: .prompt-tip}

  > `>` 会覆盖目标文件, 如果只是希望添加新内容, 则应该使用`>>`
  >
  > ```bash
  > echo "new line" >> file.txt
  > ```
  >
  {: .prompt-warning}

* `more`: 显示长文件内容, 通过空格翻页, 回车载入新行, 但只能下翻.
* `less`: 同样显示长文件内容, 可以上下翻页, 需要单独安装: ```apt install less```
* `head`: 显示前n行内容: ```head -n 10 filename```
* `tail`: 显示最后n行内容: ```tail -n 10 filename```

## 文件搜索

* `grep`: global reg expression, 正则搜索命令. 区分大小写, 可以与`-i`连用忽略大小写

  ```bash
  grep hello file.txt
  grep -i hello file.txt
  grep -i hello fi*
  grep -i -r hello .
  grep -ir helllo .
  ```

* `find`: 查找指定文件, 如果不指定任何参数, 则默认列出当前文件夹和子文件夹中的内容.
  * `-type d`: 仅查找文件夹
  * `-type f`: 仅查找文件

  ```bash
  find ~/code -type f -name "*.py"
  ```

## 命令连用

* `;`: 分割多条命令, 一次性执行.

  ```bash
  mkdir temp; cd temp; echo done
  ```

* `&&`: 当某一条指令执行失败时自动结束.

  ```bash
  mkdir temp && cd temp && echo done
  ```

* `||`: 当上一条执行失败时执行下一条指令.

  ```bash
  mkdir temp || echo "failed to create temporary directory"
  ```

* `|`: 将上一条指令的输出作为下一条指令的输入
  
  ```bash
  ls -l /etc | less
  ls -l /etc | head
  ls -l /etc | grep -i init
  ```

* `\`: 连接指令太长的时候用来换行.

## 进程管理

* `ps`: 显示当前正在运行的所有进程
* `&`: 在后台执行某项命令, 如`sleep 100 &`
* `kill [pid]`: 结束某一进程

## 环境变量

* `printenv`: 打印全部环境变量. 也可以指定要显示的环境变量, 如`printenv PATH`, 也可以使用`echo $PATH`来显示环境变量.

用户环境变量保存在`~/.bashrc`文件中, 可以向环境变量中添加一些自定义数据或者变量, 便可以在任意位置直接使用. 也可以对一些常用命令和参数的组合设置别名.

```bash
echo TEST_VAR=AAA >> .bashrc
echo $TEST_VAR
echo "alias rmd='rm -rf'" >> .bashrc 
```

> 添加的环境变量需要重启shell会话才会生效
> 也可以通过命令重新加载: `source ~/.bashrc`
>
{: .prompt-tip}
