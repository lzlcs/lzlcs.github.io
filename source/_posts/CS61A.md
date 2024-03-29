---
title: 编程入门:CS61A (Fall2022)
date: 2023-05-08 21:21:36
tags: 
- Course
- Python
- Scheme
- SQL
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
seq4 = [x if (x % 2 == 0) else x + 1 for x in seq]
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

### 2.4 Mulable Data

对象把数据值和行为结合在一起 
1. 对象具有属性: `<表达式>.<属性名称>`
2. 对象具有方法: `<表达式>.<方法名称>`

可变对象: 列表, 字典, 集合
不可变对象: 整数, 浮点数, 字符串, 元组

列表:
```python
a = [1, 2, 3]
b = a
# 此时 a, b 指向同一个列表
b += [4] # a 也会被改变
print(a) 
```
可以使用列表复制来避免这个问题
```python
c = list(a)
c += [5]
print(a) # a 没有受到上一条语句的影响
```

`is` 可以检测两个对象所指的列表是否是同一个列表, 这比 `==` 限制条件更强
```python
c.pop()
c == a # True
c is a # False
```

列表的其他常用方法
* `append(el)`: el加到末尾。返回None。
* `extend(lst)`: 在后方连接lst。返回None。
* `insert(i, el)`: 在索引i处插入el。返回None。
* `remove(el)`: 移除第一个el。返回None。如果 el 不在列表中会报错
* `pop(i)`: 移除并返回索引i处的元素。

--------

元组 (tuple) 是一个不可变对象, 类似 list
```python
t = (1, 2, [2, 3])
t[2].pop() # 如果 tuple 中的元素是可变对象, 那就可以改变这个元素的值
# 但是需要使用 可变对象自身的方法
# t[2] += [4] Error
t[2].append(4) # 可行
# t[0] = 3 不可行, 整数类型不可变
print(t)
```

字典 (dictionary) 是一个可变对象, 可以以任何不可变对象为索引 \
当然, 使用元组作为索引的时候, 元组内不能包含可变对象

```python
t = { "A": 5 }
t.get("A", 0) # 第一个参数为简直, 第二个参数为默认值 (如果键值不存在才返回)
t.get("B", 0)
```
还有使用 for 来创建一个字典的方法
```
a = { x: x + 1 for x in range(3) }
```

`nonlocal` 关键字: 使得内嵌函数能使用上一级函数的变量 \
实际应用
```python
def my_save(money):
    def out(x):
        nonlocal money
        if (x > money):
            print("余额不足")
        else:
            money -= x
        return money
    return out

money = my_save(100)
print(money(10))
print(money(80))
print(money(20))
```
可变对象不需要 nonlocal 关键字, 因为他们指向的内存是同一个

### 2.5 Object-Orient Programming

对象是具有属性和方法的值
定义类, 类中每个方法的第一个参数都是 `self`, 但是在调用的时候不用传入这个参数
```python
class <name>:
    <suite>
```
初始化对象的方法有特定的名字 `__init__()`, 称作构造函数 \
通过赋值将对象绑定到新名称不会创建一个新对象

点表达式: `<expression>.<name>`, 通过点表达式访问类内部的属性和方法

```
getattr(a, "xxx") # 调用名称为 xxx 的属性
hasattr(a, "xxx") # 查询是否有名为 xxx 的属性
```
有两种方法调用类内部定义的函数
```
<类名称>.<函数名>(实例化名称, 1)
<实例化名称>.<函数名>(1)
```
命名约定:
1. 类名称使用驼峰命名法
2. 属性名称使用下划线分割由小写字母组成的单词
3. 以下划线开头的名称通常是由类内部的方法访问, 而不是由外部访问

对于每个实例化对象, 有些属性一直不变
```python
class Account:
    interest = 0.02  # 一个类属性
    def __init__(self, account_holder):
        self.balance = 0
        self.holder = account_holder
    # 这里可以定义其他方法

Account.interest = 0.04 # 这种方法会改变所有实例化对象的 interest 属性
# 单独改变某一实例化对象的 interest 属性不会影响其他实例化对象
```
-----
继承: 子类可以覆盖基类特定的属性和方法, 未指定的内容则视为和基类相同
```python
class Account:
    """一个具有非负余额的银行账户。"""
    interest = 0.02
    def __init__(self, account_holder):
        self.balance = 0
        self.holder = account_holder
    def deposit(self, amount):
        """将账户余额增加指定金额并返回新的余额。"""
        self.balance = self.balance + amount
        return self.balance
    def withdraw(self, amount):
        """将账户余额减少指定金额并返回新的余额。"""
        if amount > self.balance:
            return 'Insufficient funds'
        self.balance = self.balance - amount
        return self.balance

class CheckingAccount(Account):
    """一个收取取款费用的银行账户。"""
    withdraw_charge = 1
    interest = 0.01
    def withdraw(self, amount):
        return Account.withdraw(self, amount + self.withdraw_charge)
```
多重继承: 一个子类可以继承自多个基类
```python
class SavingsAccount(Account):
    deposit_charge = 2
    def deposit(self, amount):
        return Account.deposit(self, amount - self.deposit_charge)

class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
    def __init__(self, account_holder):
        self.holder = account_holder
        self.balance = 1  # 一美元免费！
```
如果多个基类都有同一个方法, 继承的顺序不在本节讨论范围内


### 2.9 Recusive Objects

对象可以把其他对象当作属性值, 这样的对象称为递归对象

链表类:
1. 空链表用空元组表示, 长度为零, 没有元素
2. 内置方法名字为 `__len__` 时, 当使用 python 内置函数 `len` 的时候就会调用这个方法

```python
def link_expression(s):
    """返回一个表示s的字符串。"""
    if s.rest is Link.empty:
        rest = ''
    else:
        rest = ', ' + link_expression(s.rest)
    return 'Link({0}{1})'.format(s.first, rest)

def join_link(s, separator):
    """ 链表输出更紧凑 """
    if s is Link.empty:
        return ""
    elif s.rest is Link.empty:
        return str(s.first)
    else:
        return str(s.first) + separator + join_link(s.rest, separator)

def map_link(f, s):
    if s is Link.empty:
        return s
    else:
        return Link(f(s.first), map_link(f, s.rest))

def filter_link(f, s):
    if s is Link.empty:
        return s
    else:
        filtered = filter_link(f, s.rest)
        if f(s.first):
            return Link(s.first, filtered)
        else:
            return filtered

class Link:
    empty = ()
    def __init__(self, first, rest=empty):
        assert rest is Link.empty or isinstance(rest, Link)
        # instance 是内置函数 
        # 如果 rest 是 Link 的实例或者是 Link的某个子类的实例, 返回 True
        self.first = first
        self.rest = rest
    def __getitem__(self, i):
        if i == 0:
            return self.first
        else:
            return self.rest[i-1]
    def __len__(self):
        return 1 + len(self.rest)
    __repr__ = link_expression

```

树的实现
```python
class Tree:
    def __init__(self, label, branches=()):
        self.label = label
        for branch in branches:
            assert isinstance(branch, Tree)
        self.branches = branches

    def __repr__(self):
        if self.branches:
            return 'Tree({0}, {1})'.format(self.label, repr(self.branches))
        else:
            return 'Tree({0})'.format(repr(self.label))

    def is_leaf(self):
        return not self.branches
```
集合的实现:
1. 通过链表
2. 通过有序序列
3. 通过二叉搜索树, 随机插入元素时复杂度较优
4. 通过平衡树, 任意元素插入时复杂度较优

python 内部集合的实现: 基于哈希的一种方法, 超出课程讨论范围


## Chapter 3

### 3.1 Introduction

本章重点介绍程序本身, 研究解释器的设计和执行程序时他们创建的计算过程

### 3.2 Functional Programming

本节介绍 `Scheme` 语言的一个子集, 只使用表达式, 专门用于符号计算, 没有可变值

`Scheme` 完全使用前缀表示法, 使用小括号来分割操作符和操作数
```Scheme
(+ 1 3)
(>= 2 1)
(if <condition> <consequent> <alternative>)
(and <e1> ... <en>)
(or <e1> ... <en>)
(not <e1>)
(define pi 3.14)
(define (square x) (* x x))
(lambda (x) (x * x)) 
; 匿名函数, 功能等同于上一句
(square 21)
(square (+ 5 2))
(define x (cons 1 2))
(car x) 
; 第一个
(crd x) 
; 第二个 

(list 1 2 3 4)
(car) 
; 1
(crd)
; 2 3 4
```

### 3.3 Exceptions

在设计程序时, 应该时刻注意程序中有可能出现的错误 \
设计持久服务的程序更应该对错误具有健壮性

```python
try:
    <try suite>
except <exception class> as <name>:
    <except suite>
...
```

异常对象的应用: 第一章中牛顿插值中出现 ValueError 时返回最后一次猜测
```python
class IterImproveError(Exception):
    def __init__(self, last_guess):
        self.last_guess = last_guess
def improve(update, done, guess=1, max_updates=1000):
    k = 0
    try:
        while not done(guess) and k < max_updates:
            guess = update(guess)
            k = k + 1
        return guess
    except ValueError:
        raise IterImproveError(guess)
def find_zero(f, guess=1):
    def done(x):
        return f(x) == 0
    try:
        return improve(newton_update(f), done, guess)
    except IterImproveError as e:
        return e.last_guess
```

### 3.4 Interpreters for Languages with Combination

元语言抽象: 一种以其他语言为基础建立语言的技术

现在来用 python 实现一个 scheme 格式的计算器\
它将以字符串为输入, 并将这些表达式求值后返回结果 \
如果输入的字符串不符合语法规则, 程序将引发适当的异常

实现一个 Pair 类来存储表达式树
```python
expr = Pair('+', Pair(Pair('*', Pair(3, Pair(4, nil))), Pair(5, nil)))
print(expr) # (+ (* 3 4) 5)
```

解释器分为词法分析器和语法分析器
1. 词法分析器
    ```python
    tokenize_line('(+ 1 (* 2.3 45))')
    ['(', '+', 1, '(', '*', 2.3, 45, ')', ')']
    ```
2. 语法分析器: 一个树递归的过程

解释器: 解析, 评估, 应用, 评估/应用 循环



### 3.5 Interpreters for Languages with Abstraction


## Chapter 4

### 4.1

### 4.2 Implict Sequences

序列可以不显性地创造, 可以在用到这个值的时候延迟计算
```python
r = range(100, 100000000)
print(r[11234214])
```

迭代器: python 中按顺序处理数据的方式
```python
p = [2, 3, 5]
it = iter(p)
print(next(it))
print(next(it))
print(next(it))
print(next(it)) # StopIteration
```
类似的, 对于迭代器, 多个名字指向的是同一个迭代器, 对迭代器使用 iter 函数也返回它自己而不是一个副本
```python
it2 = it1
next(it2) # StopIteration
it3 = iter(p)
next(it3)
it4 = iter(it3)
next(it4)
```
迭代器就可以像这样每次计算出下一个迭代器, 从而实现顺序访问

通常来说, `range()`, `list`, `dictionary`, `set`, `tuple` 是可以被迭代的对象 \
特殊的, 迭代器本身也是可迭代对象, 因为它可以作为 `iter()` 函数的参数

但是, `dictionary`, `set` 这些对象的值改变不影响迭代器, 键的增加和删除会让之前的迭代器全部失效

内置迭代器:
* `map(f, iterable)` - 为可迭代对象中的每个元素 x 创建一个 f(x) 的迭代器。在某些情况下，计算这个可迭代对象中的值列表将给我们与 [func(x) for x in iterable] 相同的结果。但是，请记住，迭代器可以具有无限的值，因为它们是惰性求值的，而列表不能具有无限的元素。
* `filter(f, iterable)` - 为可迭代对象中满足 f(x) 的每个元素 x 创建一个迭代器。
* `zip(iterables*)` - 创建一个迭代器，其中包含来自每个可迭代对象的对应元素的元组。
* `reversed(iterable)` - 以逆序创建一个包含输入可迭代对象中所有元素的迭代器。
* `list(iterable)` - 创建一个包含输入的可迭代对象中所有元素的列表。
* `tuple(iterable)` - 创建一个包含输入的可迭代对象中所有元素的元组。
* `sorted(iterable)` - 创建一个包含输入的可迭代对象中所有元素的排序列表。
* `reduce(f, iterable)` - 必须导入 functools。将两个参数函数 f 从左到右累积地应用于可迭代对象的项，以将序列减少为单个值。

for 语句就是使用可迭代对象的迭代器进行循环的, 可以使用 while 来模拟
```python
try:
    while (True):
        x = next(it)
        print(x)
except StopIteration:
    pass
```
-------

生成器函数是一类特殊的函数, 使用 `yield` 而不是 `return` 来返回一系列元素 \
每次调用生成器的 `next` 方法时, 生成器函数会运行到下一个 `yield` 语句为止



# SQL

`SQL` 是一种声明式语言, 程序是描述结果, 解释器指出如何生成结果
命令式语言: 程序是计算过程的描述, 解释器执行该过程

1. `select` 语句从零创建一个新表, 或者从之前的表中创建
    * `select` 语句总是包含以逗号分割的列描述来说明表
    * 一个列描述是一个表达式, 后面可以接 `as` 和表的名称
    * `select [expression] as [name], [expression] as [name]....;`
    * `select [columns] from [table] where [condition] order by [order]`
    * 所有的 `SQL` 语句都以分号结尾
2. `creat table` 为表创建一个全局名称
    * `creat table [name] as [select statement]`
3. `analyze`, `delete`, `explain`, `insert`, `replace`, `update`

安装 `sqlite` 来解释 `SQL` 代码


