---
title: CS61A (Fall2022)
date: 2023-05-08 21:21:36
tags: 
- Course
- Python
- Scheme
categories: Course
---

[Lab 和 hw 的 Github 地址](https://github.com/lzlcs/Courses)

# Courses

## Lecture01

* `<C-p>`, `<C-n>` 获得历史记录中下一条 / 上一条命令
* `<C-d>` 退出 `python` 解释器

一个例子:
```
from urllib.request import urlopen
shakespeare = urlopen('http://composingprograms.com/shakespeare.txt')
words = set(shakespeare.read().decode().split())
{w for w in words if len(w) == 6 and w[::-1] in words}
```

**永远不用担心错误, 直面它**

## Lecture02

* 所有表达式都可以使用函数调用表示法
* `Python` 使用表达式树来计算表达式
* `<C-l>` 清空屏幕

* 定义函数
```
def <函数名>(<形式参数>):
    return <表达式>
```
* 引入函数
```
from <库名称> import <函数名称> 
```

赋值语句就是把一个值和一个名称绑定, 程序用来维护这些内容的内存即环境
连续赋值 ` A, B, C = 1, 2, 3 ` 
当调用一个函数的时候, 就进入了以这个函数的环境中
在这里寻找名称绑定的时候优先使用函数内部的变量名
如果没有找到才会使用函数外的变量名

* 纯函数: 函数执行功能并产生一个返回值, 所以更容易形成嵌套的表达式
* 非纯函数: 函数在执行过程中执行一些其他的动作比如打印

# Labs

## Lab00

### 安装

* 安装: WSL2 & WindowsTerminal
* 安装 `pyhton3`: `sudo apt install python3`
* 安装编辑器: `neovim`

### 使用终端

* `~` 表示 `home` 目录
* `echo $HOME$` 显示 `home` 目录的路径
* `pwd` 显示当前目录路径

> 注意分辨你在终端中还是 `python` 解释器中

### 组织文件

* `ls` 列出当前目录下所有文件
* `cd directoryname` 进入子目录
    * `cd ..` 进入父目录
* `mkdir directoryname` 创建一个文件夹
* `unzip lab00.zip` 解压文件
* `mv PATH1 PATH2` 移动文件

### `Python` 基础

* `python3` 进入 `python` 解释器
* 算数表达式 `+`, `-`, `*`, `%`.
  `**` (幂), `/`(浮点除法), `//`(整除).
* 一些被 `'` 或者  `"` 包含的字符序列
* 赋值语句: `=`.

### 完成作业

* `python3 ok -q python-basics -u --local` 在 `lab00` 目录下
    * `--local` 跳过邮箱验证
* 在 `lab00.py` 补全代码, `python3 ok --local`.

### 有用的 `python` 命令行中命令

* `-i` 逐行运行 `python`
