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







