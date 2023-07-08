---
title: SICP 学习笔记 (第二章) 
date: 2023-07-02 15:04:05
mathjax: true
tags:
- Scheme
- Python
- SICP
categories: 
- Book
---

本书代码使用 `python` 重构

# Chapter 2: 构造数据抽象

第一章中关注计算过程, 以及函数, 高阶函数
本章讨论复合数据: 对象

1. 复合数据可以减少无用的细节纷扰
2. 复合数据可以进一步提高程序的模块性
3. 复合数据真正提高程序设计语言的表达能力

数据抽象能让我们在程序的不同部分建立起抽象屏障

关键思想: 
1. **闭包**: 用于组合数据对象的粘合剂不但能用于组合基本的数据对象, 也可以作用于复杂的数据对象
2. 复合数据对象能成为以 **混合和匹配** 的方式组合程序模块的 **方便界面** 

## 2.1 抽象数据索引

基本思想: 构造出一些使用复合数据对象的程序, 使他们就像是在"抽象数据上操作一样

### 2.1.1 实例: 有理数的算术运算

如下是构造有理数类的一个实现方法
```python
def GCD(a, b):
    return a if b == 0 else GCD(b, a % b)
class Number:
    numer = 0
    denom = 1
    def __init__(self, a = 0, b = 1):
        self.numer, self.denom = a, b

    def get_out(self, b):
        return self.numer, self.denom, b.numer, b.denom

    def easier(self, a, b):
        factor = GCD(a, b)
        return Number(a // factor, b // factor)

    def calculate(self, other, f, g):
        n1, d1, n2, d2 = self.get_out(other)
        new_numer = f(n1, d1, n2, d2)
        new_denom = g(n1, d1, n2, d2)
        return self.easier(new_numer, new_denom)

    def __add__(self, other):
        return self.calculate(other, lambda n1, d1, n2, d2: n1 * d2 + n2 * d1,
                                     lambda n1, d1, n2, d2: d1 * d2)

    def __sub__(self, other):
        return self.calculate(other, lambda n1, d1, n2, d2: n1 * d2 - n2 * d1,
                                     lambda n1, d1, n2, d2: d1 * d2)

    def __mul__(self, other):
        return self.calculate(other, lambda n1, d1, n2, d2: n1 * n2,
                                     lambda n1, d1, n2, d2: d1 * d2)

    def __truediv__(self, other):
        return self.calculate(other, lambda n1, d1, n2, d2: n1 * d2,
                                     lambda n1, d1, n2, d2: n2 * d1)

    def __str__(self):
        return f"{self.numer}/{self.denom}"
     

x = Number(6, 5)
y = Number(9, 10)
print(x + y)
print(x - y)
print(x * y)
print(x / y)
print(x == y)
print(x != y)
```

### 2.1.2 抽象屏障

`python` 类内的 `calculate` 等函数一般不被外接调用, 所以这类函数和外界由抽象屏障 \

**抽象屏障** 用于隔离不同层级之间的差异，通过屏障实现对上层程式与下层程式的沟通 \

这种方法可以将数据构建的依赖限制在小范围内\
有利于对代码进行维护和修改后，整个系统的功能保持一致性。 

    
> 例如之前的练习 2.1, 只需要改变 `__init__` 函数即可


### 2.1.3 数据意味着什么

数据是一组适当的选择函数和构造函数, 以及使这些函数成为一套合法表示, 它们必须满足的一组特定条件

对任何对象 $x$, $y$, 如果 $z$ 是 `Point(x, y)` \
那么 `x_point` 就是 $x$, `y_point` 就是 $y$, 可以仅使用函数来实现点对

下面是点对的一种函数式表示
```python
def point(x, y):
    def match(m):
        return x if m == 0 else y
    return match
def x_point(p):
    return p(0)
def y_point(p):
    return p(1)

z = point(2, 4)
print(x_point(z), y_point(z))
```

### 2.1.4 拓展练习: 区间算数

```python
class Interval:
    upper, lower = 0, 0
    def __init__(self, a, b):
        self.lower, self. upper = a, b

    def lbound(self):
        return self.lower
    def ubound(self):
        return self.upper
    def get(self, other):
        return self.lbound(), self.ubound(), other.lbound(), other.ubound()

    def __add__(self, other):
        a, b, c, d = self.get(other)
        return Interval(a + c, b + d)

    def __sub__(self, other):
        a, b, c, d = self.get(other)
        return Interval(a - d, b - c)

    def __mul__(self, other):
        a, b, c, d = self.get(other)
        lst = [a * c, a * d, b * c, b * d]
        return Interval(min(lst), max(lst))


    def __truediv__(self, other):
        _, _, c, d = self.get(other)
        if (c <= 0 and d >= 0):
            raise AssertionError("除以跨零区间")

        return self * Interval(1 / d, 1 / c)

    def width(self):
        return (self.ubound() - self.lbound()) / 2.0

    def __str__(self):
        return f"({self.lower}, {self.upper})"

class CenterWidthInterval(Interval):
    def __init__(self, c, w):
        super().__init__(c - w, c + w)

class PercentInterval(Interval):
    percent = 0
    def __init__(self, x, p):
        super().__init__(x * (1 - p), x * (1 + p))
        self.percent = p
    def get_percent(self):
        return self.percent

def part1(r1, r2):
    return r1 * r2 / (r1 + r2)

def part2(r1, r2):
    one = Interval(1, 1)
    return one / (one / r1 + one / r2)

a = PercentInterval(100, 0.5)
b = PercentInterval(100, 0.5)

c = PercentInterval(100, 0.0)
print(c / c)
print(part1(a, b))
print(part2(a, b))
```

## 2.2 层次性数据和闭包性质

在 **盒子和指针** 表示方式中, 每个对象表示为一个指向盒子的 **指针**, 盒子里包含着对象的表示 \

**闭包性质**: 通过它组合起数据对象的结果本身还可以通过同样的操作再进行组合

### 2.2.1 序列(链表)的表示

使用 `python` 中的 `list`
```python
def cons(val, lst):
    if (not isinstance(lst, List)):
        return List([val] + [lst])
    return List([val] + lst.val)

def create_list(*args):
    return List(*args)

class List:
    def __init__(self, *args):
        self.val = list(args)

    def car(self):
        return self.val[0]
    def cdr(self):
        return List(*self.val[1:])

    def list_ref(self, n):
        return self.val[n]

    def length(self):
        return len(self.val)

    def append(self, other):
        return List(*(self.val + other.val))

    def last_pair(self):
        return self.val[self.length() - 1]

    def reverse(self):
        return List(*reversed(self.val))

    def __str__(self):
        res = "("
        for x in self.val:
            res += " " + str(x)
        return res + ')'
```
使用链表
```python
class Pair:
    def __init__(self, a = None, b = None):
        self.x, self.y = a, b

    def cons(self, a, b):
        return Pair(a, b)
    def car(self):
        return self.x
    def cdr(self):
        return self.y
    def is_null(self):
        return self.y == self.x == None

    def creat_list(self, *args):
        res = None 
        for arg in reversed(args):
            res = self.cons(arg, res)
        return res if res != None else Pair()

    def len(self):
        count, tmp = 1, self.y
        while (isinstance(tmp, Pair)):
            tmp, count = tmp.cdr(), count + 1
        return 0 if self.x == None else count

    def list_ref(self, n):
        count, tmp = 1, self.y
        while (isinstance(tmp, Pair) and count < n):
            tmp, count = tmp.cdr(), count + 1
        if (not isinstance(tmp, Pair)):
            raise IndexError("n 过大")
        return tmp.car()

    def append(self, other):
        if (self.cdr() != None):
            return self.cons(self.car(), self.cdr().append(other)) # type: ignore
        else:
            return self.cons(self.car(), other)

    def reverse(self, prev = None):
        if (self.is_null()):
            return prev

        if (self.cdr() == None):
            return self.cons(self.car(), prev)
        
        rest = self.cdr()
        self.y = prev

        return rest.reverse(self) # type: ignore

    def last_pair(self):
        tmp = self
        while (isinstance(tmp.y, Pair) and (not tmp.y.is_null())):
            tmp = tmp.y
        return Pair(tmp.car())
        
    def to_string(self, cur):
        res = ""
        if isinstance(cur.x, Pair):
            res = '(' + self.to_string(cur.x) + ')'
        elif cur.x != None:
            res = str(cur.x)
        if isinstance(cur.y, Pair):
            res += ' ' + self.to_string(cur.y)
        elif cur.y != None:
            res += " . " + str(cur.y)

        return res


    def __str__(self):
        return '(' + self.to_string(self) + ')'

```

### 2.2.2 层次性结构

```python
class TreeNode:
    def __init__(self, value = None, children = None):
        self.value = value
        self.children = children or []

    def add_child(self, child_node):
        self.children.append(child_node)

    def __str__(self):
        return self._str_recursive(self)

    def is_leaf(self):
        return len(self.children) == 0

    def count_leaves(self):

        def iter(cur_node):
            if (cur_node.is_leaf()):
                return 1
            return sum([iter(x) for x in cur_node.children])

        return iter(self)


    def deep_reverse(self):
    
        def iter(cur_node):
            if (cur_node.is_leaf()):
                return TreeNode(cur_node.value)
            new_children = []
            for child in cur_node.children:
                new_children += [iter(child)]
            return TreeNode(cur_node.value, reversed(new_children))
            
        return iter(self)

    def fringe(self):

        def iter(cur_node):
            if (cur_node.is_leaf()):
                return [cur_node.value]
            res = []
            for child in cur_node.children:
                res += iter(child)
            return res

        return iter(self)

    def map(self, proc):

        def iter(cur_node):
            res = []
            for child in cur_node.children:
                res += [iter(child)]
            return TreeNode(proc(cur_node.value), res)

        return iter(self)

        
    def square(self):
        def square(x):
            return x * x
        return self.map(square)


    def _str_recursive(self, node, level = 0):
        indent = " " * level
        res = f"{indent}{node.value}\n"

        for child in node.children:
            res += self._str_recursive(child, level + 1)
        return res


root = TreeNode(1)
node_b = TreeNode(2)
node_c = TreeNode(3)
node_d = TreeNode(4)
node_e = TreeNode(5)
node_f = TreeNode(6)

root.add_child(node_b)
root.add_child(node_c)
node_b.add_child(node_d)
node_b.add_child(node_e)
node_c.add_child(node_f)

# 打印树
print(root)
print(root.count_leaves())
print(root.deep_reverse())
print(root.fringe())
print(root.map(lambda x: x * x))
print(root.square())

```

### 2.2.3 序列作为一种约定的界面

工作的过程可以抽象为一下几步: 枚举器, 过滤器, 转换器, 累计器 

```python
def sum_odd_squares(cur):
    if (cur.is_leaf()):
        if (cur.value % 2 == 1):
            return cur.value * cur.value
        return 0

    res = 0 
    for child in cur.children:
        res += sum_odd_squares(child)
    return res

def even_fibs(n):
    a, b = 0, 1
    res = []
    for _ in range(n):
        tmp = a + b
        a, b = b, tmp
        if (tmp % 2 == 0):
            res.append(tmp)
    return res
```

以上这两个函数都混杂了各个步骤, 不利于程序的抽象表达

```python
def add(a, b):
    return a + b

def car(lst):
    return lst[0]
def cdr(lst):
    return list(lst[1:])

def cons(a, b):
    return [a] + b

def filter(predicate, lst):
    return [x for x in lst if predicate(x)]

def accumulate(op, initial, lst):
    res = initial
    for x in reversed(lst):
        res = op(x, res)
    return res

def enumerate_interval(low, high):
    return [x for x in range(low, high + 1)]

def enumerate_tree(cur_node):
    if (cur_node.is_leaf()):
        return [cur_node.value]

    res = []
    for x in cur_node.children:
        res += enumerate_tree(x)
    return res


def sum_odd_squares_new(cur_node):
    lst = enumerate_tree(cur_node)
    lst = filter(lambda x: x % 2 == 1, lst)
    lst = map(lambda x: x * x, lst)
    return accumulate(add, 0, lst)

@cache
def fib(x):
    if (x <= 1):
        return 1
    return fib(x - 1) + fib(x - 2)

def even_fibs_new(n):
    lst = enumerate_interval(0, n)
    lst = map(fib, lst)
    lst = filter(lambda x: x % 2 == 0, lst)
    return accumulate(cons, None, lst)
```

抽象出 `enumerate`, `map`, `filter`, `accumulate` 进行模块化设计

范围广大的许多操作都可以表述为序列操作

**嵌套映射**

```python
def make_pairs(n):
    lst = accumulate(append, [], 
                     list(map(lambda i: list(map(lambda j: [i, j], 
                                                 enumerate_interval(1, i - 1))),
                              enumerate_interval(1, n))))
    lst = filter(lambda x: is_prime(sum(x)), lst)
    return list(map(lambda x: [*x, sum(x)], lst))
```

```python
def permutations(lst):
    if (lst == []):
        return [[]]
    
    tmp = permutations(lst[1:])

    res = []
    for x in tmp:
        for i in range(len(tmp[0]) + 1):
            res += [x[:i] + [lst[0]] + x[i:]]

    return res

def flatmap(proc, seq):
    return accumulate(append, [], list(map(proc, seq)))

def remove(item, sequence):
    return filter(lambda x: not (x == item), sequence)

def permutations_new(s):
    if (s == []):
        return [[]]
    return flatmap(lambda x: list(map(lambda p: [x] + p,
                                     permutations(remove(x, s)))), s)

print(permutations([1, 2, 3]))
print(permutations_new([1, 2, 3]))
```


# 练习

## 2.1

更改 `__init__` 函数
```python
    def __init__(self, a = 0, b = 1):
        tmpa, tmpb = abs(a), abs(b)
        if (a * b < 0):
            tmpa = -tmpa
        self.numer, self.denom = tmpa, tmpb
```

## 2.2

```python
class Point:
    x, y = 0, 0
    def __init__(self, a = 0, b = 0):
        self.x, self.y = a, b
    
    def x_point(self):
        return self.x

    def y_point(self):
        return self.y

    def __str__(self):
        return f"({self.x}, {self.y})"

class Segment:
    x, y = Point(), Point()
    def __init__(self, a = Point(), b = Point()):
        self.x, self.y = a, b

    def start_segment(self):
        return self.x
    
    def end_segment(self):
        return self.y
    
    def mid_segment(self):
        p1, p2 = self.start_segment(), self.end_segment()
        a, b, c, d = p1.x_point(), p1.y_point(), p2.x_point(), p2.y_point()
        return Point((a + b) / 2, (c + d) / 2)

x = Segment(Point(1, 1), Point(2, 3))
print(x.mid_segment())
```

## 2.3

一个是用起始结束点表示, 一个是用对角线表示
```python
class Rectangle:
    start, end = Point(), Point()
    def __init__(self, a, b):
        self.start, self.end = a, b

    def start_point(self):
        return self.start

    def end_point(self):
        return self.end


class Rectangle_2:
    diagonal = Segment()
    def __init__(self, seg = Segment()):
        self.diagonal = seg

    def start_point(self):
        return self.diagonal.start_segment()

    def end_point(self):
        return self.diagonal.end_segment()

def length(rec):
    p1, p2 = rec.start_point(), rec.end_point()
    return abs(p1.x_point() - p2.x_point())

def width(rec):
    p1, p2 = rec.start_point(), rec.end_point()
    return abs(p1.y_point() - p2.y_point())

def perimeter(rec):
    return 2 * (length(rec) + width(rec))

def area(rec):
    return length(rec) * width(rec)
        
a = Rectangle(Point(1, 1), Point(2, 3))
b = Rectangle_2(Segment(Point(1, 1), Point(2, 3)))

print(perimeter(a))
print(area(a))
print(perimeter(b))
print(area(b))
```

## 2.4

```python
def cons(x, y):
    return lambda m: m(x, y)

def car(z):
    return z(lambda p, q: p)

def cdr(z):
    return z(lambda p, q: q)

t = cons(1, 2)
print(car(t), cdr(t))
```

```
cdr(t)
cdr(lambda m: m(1, 2))
(lambda m: m(1, 2))(lambda p, q: q)
(lambda p, q: q)(1, 2)
2
```

## 2.5

```python
def cons(x, y):
    a, b = pow(2, x), pow(3, y)
    return lambda z: z(a, b)

def car(z):
    return z(lambda p, q: p)

def cdr(z):
    return z(lambda p, q: q)

t = cons(3, 2)
print(car(t), cdr(t))
```

## 2.6

丘奇计数:
```python
zero = lambda f: lambda x: x
one = lambda f: lambda x: f(x)
two = lambda f: lambda x: f(f(x))

def add1(n):
    return lambda f: lambda x: f(n(f)(x))

def add(a, b):
    return lambda f: lambda x: a(f)(b(f)(x))

def church_to_int(n):
    x, y = 0, lambda x: x + 1
    return n(y)(x)

def int_to_church(n):
    res = zero
    for _ in range(n):
        res = add1(res)
    return res

print(church_to_int(add(one, two)))
print(church_to_int(int_to_church(10)))
print(church_to_int(two))
```

## 2.7

见正文 2.1.4

## 2.8

$$
\begin{align}
& 定义两个区间, Interval(al, au), Interval(bl, bu), 其中 al < au, \; bl < bu \\\\
& p1, p2, p3, p4 = al - bl, al - bu, au - bl, au - bu \\\\
& p1 - p2 = bu - bl > 0, \; p1 > p2 \\\\
& p3 - p2 = au + bu - al - bl > 0, \; p3 > p2 \\\\
& p4 - p2 = au - al > 0, \; p4 > p2 \\\\
& p2 = min(p1, p2, p3, p4) \\\\
& p1 - p3 = al - au < 0, \; p1 < p3 \\\\
& p2 - p3 = -(p3 - p2) < 0, \; p2 < p3 \\\\
& p4 - p3 = bu - bl < 0, \; p4 < p3 \\\\
& p3 = max(p1, p2, p3, p4) \\\\
& 综上, 新的区间应是 Interval(p2, p3)
\end{align}
$$

## 2.9


$$
\begin{align}
& 定义两个区间, Interval(al, au), Interval(bl, bu), 其中 al < au, \; bl < bu \\\\
& 加法结果 Interval(al + bl, au + bu), 减法结果 Interval(al - bu, au - bl) \\\\
& 加法宽度: \frac{(au + bu - al + bl)} 2 = \frac{(au - al)}2 + \frac{(bu - bl)} 2 \\\\
& 减法宽度: \frac{(au - bl - al + bu)} 2 = \frac{(au - al)}2 + \frac{(bu - bl)} 2 \\\\

\end{align}
$$

```python
a = Interval(1, 2)
b = Interval(-4, -2)

print(a, b, (a * b), (a / b))

print(a.width(), b.width(), (a * b).width(), (a / b).width())
```

## 2.10

见 2.1.4 `__truediv__` 函数

## 2.11

```python
def mul(x, y):
    a, b, c, d = x.get(y)
    resa, resb = None, None
    if (a <= b <= 0 and c <= d <= 0):
        return Interval(b * d, a * c)
    elif (a <= b <= 0 and d >= c >= 0):
        return Interval(a * d, b * c)
    elif (b >= a >= 0 and d >= c >= 0):
        return Interval(a * c, b * d)
    elif (b >= a >= 0 and c <= d <= 0):
        return Interval(b * c, a * d)
    elif (b >= 0 >= a and d >= c >= 0):
        return Interval(a * d, b * d)
    elif (b >= 0 >= a and c <= d <= 0):
        return Interval(b * c, a * c)
    elif (b >= a >= 0 and c <= 0 <= d):
        return Interval(b * c, b * d)
    elif (a <= b <= 0 and c <= 0 <= d):
        return Interval(a * d, a * c)
    elif (a <= 0 <= b and c <= 0 <= d):
        return Interval(min(a * d, b * c), max(a * c, b * d))

# 正确性测试
def random_interval():
    a = random.randint(0, 10000) - 50
    b = random.randint(0, 10000) - 50
    if (a > b):
        a, b = b, a
    return Interval(a, b)

def equal(a, b):
    return a.lbound() == b.lbound() and a.ubound() == b.ubound()

for _ in range(100000):
    x, y = random_interval(), random_interval()

    if (not equal(mul(x, y), x * y)):
        print(mul(x, y), x * y)
        raise AssertionError
    else:
        print("OK ", _)

# 时间测试

import time

start_time = time.time()

for _ in range(1000000):
    x, y = random_interval(), random_interval()
    z = mul(x, y)

end_time = time.time()

print("time: ", end_time - start_time)

start_time = time.time()

for _ in range(1000000):
    x, y = random_interval(), random_interval()
    z = x * y

end_time = time.time()

print("time: ", end_time - start_time)

# time:  2.0358922481536865
# time:  2.1305015087127686
```

经过时间测试, 新的分类乘法函数比之前的函数快了一丁点


## 2.12

见 2.1.4 `PercentInterval` 类

## 2.13

令 $a, b = PercentInterval(x_1, p_1), PercentInterval(x_2, p_2)$ \
两者相乘, $c = Interval(x_1 x_2(1-p_1)(1-p_2), (x_1 x_2(1+p_1)(1+p_2))$ \
又因为 $p_1p_2$ 非常小, 所以 $c$ 可以近似为 $Interval(x_1x_2(1-p_1-p_2), x_1x_2(1+p_1+p_2))$ \
所以可转换为 $c = PercentInterval(x_1x_2, p_1+p_2)$

## 2.14

浮点数误差导致误差甚至完全错误的结果

## 2.15

```python
a = PercentInterval(100, 0.5)
b = PercentInterval(100, 0.5)

print(part1(a, b))
print(part2(a, b))

# (8.333333333333334, 225.0)
# (25.0, 75.0)
```
看似 `part2` 计算的结果更加准确

`Eva` 的说法正确

## 2.16

这是区间的运算, 两种表达式并不等价

设计: ...?


## 2.17 

见正文

## 2.18

见正文

## 2.19

```python
def car(lst):
    return lst[0]
def cdr(lst):
    return lst[1:]

def no_more(coin_values):
    return len(coin_values) == 0

def first_denomination(coin_values):
    return car(coin_values)

def except_first_denomination(coin_values):
    return cdr(coin_values)


def cc(amount, coin_values):
    if (amount == 0):
        return 1
    if (amount < 0 or no_more(coin_values)):
        return 0
    
    value = first_denomination(coin_values)
    a = cc(amount, except_first_denomination(coin_values))
    b = cc(amount - value, coin_values)
    return a + b

us_coins = [50, 25, 10, 5, 1]
uk_coins = [100, 50, 25, 10, 5, 2, 1, 0.5]

print(cc(100, uk_coins))
```

不会, 因为组合方式是一定的

## 2.20

```python
def same_parity(*args):
    tmp, res = args[0] % 2, []
    for arg in args:
        if (arg % 2 == tmp):
            res.append(arg)
    return res

print(same_parity(1, 2, 3, 4, 5, 6, 7))
print(same_parity(2, 3, 4, 5, 6, 7))
```

## 2.21

```python
def square_list_items_1(lst):
    res = []
    for x in lst:
        res.append(x * x)
    return res

def map(lst, proc):
    res = []
    for x in lst:
        res.append(proc(x))
    return res

def square_list_items_2(list):
    return map(list, lambda x: x * x)

lst = [1, 2, 3, 4]
print(square_list_items_1(lst))
print(square_list_items_2(lst))
```

## 2.22

第一个程序是每次把结果放在开头
第二个程序是把一个 `list` `cons` 在一个 `atom` 上

## 2.23

```python
def for_each(lst, proc):
    if (len(lst) == 0):
        return
    proc(lst[0])
    for_each(lst[1:], proc)
```

## 2.24

```scheme
(1 (2 (3 4)))

1 
  2
    3
    4
```

## 2.25

```python
x = [1, 3, [5, 7], 9]
y = [[7]]
z = [1, [2, [3, [4, [5, [6, 7]]]]]]

def car(lst):
    return lst[0]
def cdr(lst):
    return lst[1:]

print(car(cdr(car(cdr(cdr(x))))))
print(car(car(y)))
print(car(cdr(car(cdr(car(cdr(car(cdr((car(cdr((car(cdr(z)))))))))))))))
```

## 2.26

```Scheme
(1 2 3 4 5 6)
((1 2 3) 4 5 6)
((1 2 3) (4 5 6))
```
## 2.27

见正文

## 2.28

见正文

## 2.29

```python
class Mobile:
    def __init__(self, l = None, r = None):
        self.left, self.right = l, r

    def left_branch(self):
        return self.left

    def right_branch(self):
        return self.right

    def total_weight(self):

        def iter(cur_node):
            if isinstance(cur_node, Mobile):
                a = iter(cur_node.left_branch().branch_structure()) # type: ignore
                b = iter(cur_node.right_branch().branch_structure()) # type: ignore
                return a + b
            
            return cur_node

        return iter(self)

    def is_balance(self):

        def check(cur_node):
            if isinstance(cur_node, Mobile):
                lb = cur_node.left_branch()
                rb = cur_node.right_branch()

                assert(lb != None and rb != None)

                ls = lb.branch_structure()
                rs = rb.branch_structure()

                lw = ls if type(ls) == int else ls.total_weight()
                rw = rs if type(rs) == int else rs.total_weight()

                ll, rl = lb.branch_length(), rb.branch_length()

                if (lw * ll != rw * rl):
                    return False
    
                return check(ls) and check(rs)
                
            return True

        return check(self)

class Branch:
    def __init__(self, l, s):
        self.length, self.structure = l, s

    def branch_length(self):
        return self.length

    def branch_structure(self):
        return self.structure

lb = Branch(2, 3)
rb = Branch(3, 2)
m = Mobile(lb, rb)

lb2 = Branch(2, m)
rb2 = Branch(5, 2)
m2 = Mobile(lb2, rb2)

print(m2.total_weight())
print(m2.is_balance())
```

只需要对选择函数做修改即可

## 2.30

```python
def square(self):

    def square(x):
        return x * x
    def iter(cur_node):
        res = []
        for child in cur_node.children:
            res += [iter(child)]
        return TreeNode(square(cur_node.value), res)

    return iter(self)
```

```python
print(root.map(lambda x: x * x))
print(root.square())
```

## 2.31

```python
def square_tree(cur_node):
    return cur_node.map(lambda x: x * x)
```

## 2.32

```python
def subsets(s):
    if (s == []):
        return [[]]
    rest = subsets(s[1:])
    return rest + list(map(lambda x: [s[0]] + x, rest))

```

先拿出第一个元素, 找出后面所有元素形成的子集 \
然后把每个子集前面加上这第一个元素, 在算上原来的子集, 就是新子集

## 2.33

```python
def map_new(p, seq):
    return accumulate(lambda x, y: cons(p(x), y), [], seq)
    
def append(lst1, lst2):
    return accumulate(cons, lst2, lst1)

def length(seq):
    return accumulate(lambda x, y: y + 1, 0, seq)
```
def horner_eval(x, coefficient_sequence):
    return accumulate(lambda this_coeff, higher_terms: this_coeff + higher_terms * x, 
                      0, coefficient_sequence)

## 2.34

```python
def horner_eval(x, coefficient_sequence):
    return accumulate(lambda this_coeff, higher_terms: this_coeff + higher_terms * x, 
                      0, coefficient_sequence)
```

## 2.35

```python
def count_leaves_new(cur_node):
    return accumulate(lambda x, y: y + 1, 0, 
                      list(map(lambda x: x, enumerate_tree(cur_node))))

```

## 2.36

```python
def accumulate_n(op, init, seqs):

    def firsts(seqs):
        return [car(x) for x in seqs]

    def rests(seqs):
        return [cdr(x) for x in seqs]

    if (len(car(seqs)) == 0):
        return []
    return cons(accumulate(op, init, firsts(seqs)),
                accumulate_n(op, init, rests(seqs)))
```

## 2.37

```python
def mul(a, b):
    return a * b

def matrix_vector(m, v):
    return list(map(lambda mi: accumulate(add, 0, accumulate_n(mul, 1, [mi, v])), m))

def transpose(m):
    return accumulate_n(cons, [], m)

def matrix_matrix(m, n):
    cols = transpose(n)
    return list(map(lambda mi: matrix_vector(cols, mi), m))
```

## 2.38

```python
def ford_left(op, initial, lst):
    res = initial
    for x in lst:
        res = op(res, x)
    return res

def div(a, b):
    return a / b
print(accumulate(div, 1, [1, 2, 3]))
print(ford_left(div, 1, [1, 2, 3]))

def my_list(*args):
    return list(args)

print(accumulate(my_list, [], [1, 2, 3]))
print(ford_left(my_list, [], [1, 2, 3]))
```

## 2.39

```python
def reverse(seq):
    return accumulate(lambda x, y: y + [x], [], seq)
def reverse_new(seq):
    return ford_left(lambda x, y: [y] + x, [], seq)

print(reverse([1, 2, 3]))
print(reverse_new([1, 2, 3]))
```

## 2.40

```python
def unique_pairs(n):
    return flatmap(lambda x: list(map(lambda y: [x, y], 
                                      enumerate_interval(1, x - 1))),
                   enumerate_interval(1, n))

def prime_sum_pairs(n):
    tmp = unique_pairs(n)
    tmp = filter(lambda pair: is_prime(sum(pair)), tmp)
    return list(map(lambda pair: [*pair, sum(pair)], tmp))

print(prime_sum_pairs(6))
```

## 2.41

```python
def unique_triple(n):
    return flatmap(lambda x: list(map(lambda y: [x] + y,
                                     unique_pairs(x - 1))),
                   enumerate_interval(1, n))

def question_2_41(n, s):
    tmp = unique_triple(n)
    tmp = filter(lambda x: sum(x) == s, tmp)
    return sorted(tmp)
```

## 2.42

```python
def queens(board_size):

    empty_board = [[]]

    def is_safe(k, positions):
        cur = positions[k - 1]
        rest = positions[:k - 1]
        if (rest.count(cur) != 0):
            return False

        for i in range(0, k - 1):
            if (k - i - 1 == abs(positions[i] - cur)):
                return False
        
        return True 

    def adjoin_position(new_row, k, rest_of_queens):
        return rest_of_queens + [new_row]

    def queen_cols(k):
        if (k == 0):
            return empty_board
        return filter(lambda positions: is_safe(k, positions),
                      flatmap(lambda rest_of_queens: 
                                list(map(lambda new_row: 
                                           adjoin_position(new_row, k, rest_of_queens),
                                         enumerate_interval(1, board_size))),
                              queen_cols(k - 1)))

    return queen_cols(board_size)
```

不抽象版本
```python
def new_queen(n, cur, state):
    def is_safe(k, positions):
        cur = positions[k]
        rest = positions[:k]
        if (rest.count(cur) != 0):
            return False

        for i in range(0, k):
            if (k - i == abs(positions[i] - cur)):
                return False
        
        return True 

    if (n == cur):
        return [state]

    res = []
    for row in range(n):
        if (is_safe(cur, state + [row])):
            res += new_queen(n, cur + 1, state + [row])

    return res
```

## 2.43

原来是枚举 `queen_cols(k - 1)` 次 `enumerate_interval` \
现在是枚举 `enumerate_interval` 次 `queen_cols(k - 1)`

看似没有区别, 实际上原来往下递归之后结果就保存住了 \
改代码之后往下递归被重复执行了 `board_size` 次 \
所以大概时间是 $T * board\_size$ 次



