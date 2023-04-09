---
title: LearnGit
date: 2023-01-07 10:39:49
categories:
- Tools
tags: 
- Git
- Tools
---

A note about learning git.

<!--more-->

# INSTALL(Ubuntu)

```
sudo apt-get install git
```

# CONFIG

## 配置全局用户名和邮箱

```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

# BASIC OPERATORS

## 初始化

* 在当前需要使用git管理的文件夹中

```
git init
```

* 此时执行 `ll` 会发现当前目录多了一个 `.git` 文件夹
  * 如果你不知道你在做什么, 请不要随意修改该文件夹

## 添加文件

* 创建一个文本文件, 并输入一些内容
* 添加到暂存区

```
git add new.md
```

* `git add` 可以一次性提交多个文件

```
git add file1.md file2.md file3.md
```

## 提交文件

* 提交到仓库
* `-m` 表示本次的提交说明

```
git commit -m "creat a new file named new.md"
```

* 尝试修改文件

## 查看文件状态

```
git status
```

## 比较前后文件

```
git diff new.md
```

* 此时可以重新 `add commit`, 之后查看新状态

* 多试几次, 这样就会得到多个提交记录

## 提交记录

```
git log
```

* `--pretty=oneline` 可以简化输出

## 版本回退

* `HEAD` 表示当前版本

* `HEAD^` 表示上一个版本, `HEAD^^` 表示再上一个版本
* `HEAD~100` 表示上100个版本

```
git reset --hard HEAD^
```

此时已经是前一个版本了, 此时 `git log` 发现最新的版本完全消失了!

当然可以找上面的 一长串版本号来回退

```
git reset d799
```

此时 `git log` 发现又回来了

但如果找不到之前的版本号呢

```
git reflog
```

这条命令可以显示记录下的每一个命令

## 撤销修改

* 回退到上次commit

```
git checkout -- file
```

* 撤销暂存区文件

```
git reset HEAD <file>
```

## 删除文件

```
git rm file
```
