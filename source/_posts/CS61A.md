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


# 完成 lab 的基础

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

# Textbook

## Chapter 1

### 1.1 Getting Started

* 安装 Python: `sudo apt-get install python3`
* 使用命令 `python` 进入 Python 解释器, `>>>` 是提示符
* `<C-p>`, `<C-n>` 获得历史记录中下一条 / 上一条命令
* `<C-d>` 退出 `python` 解释器

**永远不用担心错误, 直面它**

### 1.2 Elements of Programming

每一个强大的编程语言都有这样三种机制
* 基元表达式和语句
* 组合的方法
* 抽象的方法

* 所有表达式都可以使用函数调用表示法
* `Python` 使用表达式树来计算表达式
* `<C-l>` 清空屏幕

### 1.3 Defining New Funtions

* 定义函数
```
def <函数名>(<形式参数>):
    return <表达式>
```
* 引入函数
```
from <库名称> import <函数名称> 
```

`def` 语句和赋值语句就是把一个值和一个名称绑定, 且任何这个名字之前的绑定都会丢失
连续赋值 ` A, B, C = 1, 2, 3 ` 

程序用来维护这些内容的内存即环境, 程序用来维护这些内容的内存即环境
当调用一个函数的时候, 就进入了以这个函数的环境中
在这里寻找名称绑定的时候优先使用函数内部的变量名(包括形参列表)
如果没有找到才会使用函数外的变量名

* 纯函数: 函数执行功能并产生一个返回值, 所以更容易形成嵌套的表达式
* 非纯函数: 函数在执行过程中执行一些其他的动作比如打印

形参的名称并不影响函数功能

命名时的共同约定:
1. 函数名, 变量名使用小写字母, 单词之间以下划线分隔
2. 函数名, 变量名使用描述性的语言, 便于了解功能
3. 避免使用单个字符, 除了它的作用显而易见
4. 尽量不使用 l 或 o 等单个字符变量以免与数字混淆

函数作为一层抽象, 用户并不需要了解它的内部实现

### 1.4 Designing Funtions

好的函数应该具备以下几个特点:
1. 每个函数都应该只有一个具体的工作
2. 不要重复自己的代码, 如果有, 请抽象成一个函数
3. 对函数进行一般定义, 如 pow(x, 2) 可以取代 square(x), 所以只有 pow 在 Python 标准库中

----------------------

给函数写注释: 
```python
def square(a):
    """
    返回 a 的平方
    """
    return a * a
```
此时使用 `help(square)` 可以看到三个双引号包裹的内容, 按 q 退出

在写 Python 程序的时候, 除了最简单的函数其他都要写注释

```python
pi = 3.14 # 圆周率近似为 3.14
```
使用 # 号开头的作为单行注释, 解释器忽略其后的内容


----------------------

函数的默认参数
```python
def fun1(a, b = 10):
    print(a + b)
```
如果调用 `fun1(10)` 则会打印 20, 此时 b 的值默认为 10
如果用 `fun1(10, 20)` 则会打印 30, b 重新被定为 20

注意具有默认参数的形参只能统一放在函数列表后方

### 1.5 Control

注意缩进时每个套件内都要使用相同的缩进方式, 否则会报错

条件表达式
1. False, True 两个布尔值, 注意仅 0, None, False 的值为 False
2. \>, <, <=, >=, ==, != 顾名思义
3. or, and, not 与或非
   1. or 的短路特性: A or B 当 A 为 True 时, 返回 A, 不再计算 B, 否则返回 B
    2. and 的短路特性: A and B 当 A 为 False 时, 返回 A, 不再计算 B, 否则返回 B

条件语句
```python
if <表达式>:
    <套件>
elif <表达式>:
    <套件>
else:
    <套件>
```

环语句
```
while <表达式>:
    <循环主体>
```
注意使用 `<C-c>` 来结束无限循环

测试语句
1. assert
```
assert <表达式>
```
表达式为真时, assert 不会有任何效果, 当表达式为假时会报错
2. doctest
```python
def fun(a)
    """ print(a)

    >>> fun(1)
    1
    >>> fun(2)
    2
    >>> fun(3)
    4
    """
    return a
from doctest import testmod
testmod()
```
testmod 函数把定义过的函数并且有如此形式的函数全部做测试, 如果函数运行结果不符则报错
当然可以单独测试某个函数
```python
from doctest import run_docstring_examples
run_docstring_examples(fun, globals(), True)
```
第一个参数是函数名字, 第二个参数为 globals() 的返回值, 第三个 True 表示你想看到测试过程

### 1.6 Higher-Order Functions

操纵函数的函数称为高阶函数, 把函数作为参数传入可以实现更高度的抽象
在本地定义的函数称为闭包
把有两个参数 x, y 的函数 转化为 使用一个参数 x 的高阶函数返回一个参数为 y 的函数, 这种转换称为柯里化

lambda 表达式动态创建函数值, 返回值是一个未命名的函数
```python
def combine(f, g)
    return lambda x: f(g(x))
combine(lambda x: x * x, lambda x: x + 1)
s = lambda x: x * x * x
s(12)
```

在 Python 中, 函数是一等公民, 它可以被传递, 分配给其他变量, 并作为参数传递给其他函数
Python 装饰器本身是一个函数, 它可以修改被装饰函数的行为

```python
def decorator(func):
    def wrapper(x):
        # 在调用被装饰函数之前的额外逻辑
        print("在调用被装饰函数之前的额外逻辑")
        result = func(x)
        # 在调用被装饰函数之后的额外逻辑
        print("在调用被装饰函数之后的额外逻辑")
        return result
    return wrapper
```

### 1.7 Recusive Functions

一个在内部调用自身的函数称作递归函数

递归基本结构
1. 一个条件判断语句, 是判断当递归进入最简单的情况时的边界情况
2. 相信你下一层递归的结果, 然后借此来计算本层的递归结果
类似一种归纳证明的模式
递归函数可以避免一些本地名称分配的问题, 不需要很多变量即可实现所需功能

* 两个互相调用的函数称作互递归, 有些互递归的函数可以转换为普通递归
* 一个函数多次调用自己就是树递归

## Chapter 2

### 2.1 Introduction 

`type` 函数可以检测任何值的类型

* 数字类型: `int`, `float`, `complex`
    * 注意 float 类型是个近似表示, 有系统误差

### 2.2 Data Abstraction

将程序中处理数据表示的部分和处理数据操作的部分分离开来的设计方法称为数据抽象

列表: 通过方括号中的一系列用逗号隔开的表达式构建而成
```python
pair = [10, 20]
```
访问方式: 
1. 使用变量
    ```python
        x, y = pair
    ```
2. 使用下标访问 `pair[0]`, `pair[1]`
3. `getitem(pair, 0)`

使用数据抽象的时候, 最好让函数不依赖于特定表述, 从而方便维护

### 2.3 Sequences

`list` 是一个可以有任意长度的序列, 有很多的内置行为

```python
seq = [1, 3, 4, 5]
len(seq) # 输出 seq 长度
[1, 2] + seq # + 代表连接序列
seq * 2 # * 代表复制, 即 * 2 就是复制两次
```

当然, `list` 可以包含很多数据类型, 包括 `list` 自己, 这样就形成了二维 `list`

`for` 语句可以遍历序列的每一个元素

```python
for item in seq:
    print(item)
```
`for` 还可以用来拆包
```python
pairs = [[1, 2], [2, 2], [2, 3], [4, 4]]
for x, y in pairs:
    print(x, ' ', y)
```

`range(a, b)` 左开右闭, a 默认为 0
```python
for i in range(20, 40):
    print(i)
# 当 for 后面定义的名称在循环中用不到时, 应使用 _
# 这不是硬性要求, 但是这是一个习惯
for _ in range(40):
    print("Hello World!")
```

列表推导式, 当想对序列中每个元素做相同的操作时
```python
seq2 = [x + 1 for x in seq]
seq3 = [x for x in seq if (25 % x == 0)]
# 一般形式
[<映射表达式> for <名称> in <序列表达式> if <筛选表达式>]
```

聚合: 把序列中的值经过某种运算最后留下一个值, 如 `min`, `max`, `sum`等

列表成员
```python
seq = [1, 3, 4, 5]
3 in seq # 输出 True
6 in seq # 输出 False
6 not in seq # 输出 True
```

切片: `seq[a:b]` 表示下标为 a~b 的这些元素组成的序列, a 默认为 0, b 默认为 `list` 长度

空列表: `not list == True`

------------------
字符串: `string`, 由单引号或双引号包围的任意文本

字符串有长度, 支持用下标选择某个字符, 支持 `+`, `*`, 成员

三个引号可以表示多行字符串

字符串强制转换: 可以转换数字, 列表

-------------

用列表作为其他列表元素的能力被称为数据类型的闭包属性

组合的结果本身可以使用相同的方法进行组合, 则这种方法有闭包属性

可以使用列表来构造一棵树
```python
def tree(root_label, branches=[]):
    for branch in branches:
        assert is_tree(branch), 'branches must be trees'
    return [root_label] + list(branches)
def label(tree):
    return tree[0]
def branches(tree):
    return tree[1:]
def is_tree(tree):
    if type(tree) != list or len(tree) < 1:
        return False
    for branch in branches(tree):
        if not is_tree(branch):
            return False
    return True
def is_leaf(tree):
    return not branches(tree)
```

```python
# 构建斐波那契树
def fib_tree(n):
    if n == 0 or n == 1:
        return tree(n)
    else:
        left, right = fib_tree(n-2), fib_tree(n-1)
        fib_n = label(left) + label(right)
        return tree(fib_n, [left, right])
```

----------
可以使用列表构建链表
```python
empty = 'empty'
def is_link(s):
    """s is a linked list if it is empty or a (first, rest) pair."""
    return s == empty or (len(s) == 2 and is_link(s[1]))
def link(first, rest):
    """Construct a linked list from its first element and the rest."""
    assert is_link(rest), "rest must be a linked list."
    return [first, rest]
def first(s):
    """Return the first element of a linked list s."""
    assert is_link(s), "first only applies to linked lists."
    assert s != empty, "empty linked list has no first element."
    return s[0]
def rest(s):
    """Return the rest of the elements of a linked list s."""
    assert is_link(s), "rest only applies to linked lists."
    assert s != empty, "empty linked list has no rest."
    return s[1]
def len_link(s):
    """Return the length of linked list s."""
    length = 0
    while s != empty:
        s, length = rest(s), length + 1
    return length
def getitem_link(s, i):
    """Return the element at index i of linked list s."""
    while i > 0:
        s, i = rest(s), i - 1
    return first(s)
```
