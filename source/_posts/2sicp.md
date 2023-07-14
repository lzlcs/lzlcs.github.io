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

### 2.2.4 实例: 一个图形语言

语言中只有一种元素, 称为画家

勉强写了个分形的程序, 不能完成全部的 `painter` 功能 \
无法确认代码的正确性, 故跳过

```python
identity = lambda x: x

class Picture:

    def __init__(self, length = 0, width = 0, atom = True):
        self.l, self.w = length, width
        self.strs = [" " * length for _ in range(width)]
        if (atom):
            self.strs[0] = "*" * length
            self.strs[width - 1] = "*" * length

            for i in range(1, length - 1):
                pre = self.strs[i][:i]
                aft = self.strs[i][i + 1:]
                self.strs[i] = pre + "*" + aft

    def beside(self, other):
        res = Picture(self.l + other.l, self.w, False)
        for i in range(self.w):
            res.strs[i] = self.strs[i] + other.strs[i]
        return res

    def below(self, other):
        res = Picture(self.l, self.w + other.w, False)
        res.strs = other.strs + self.strs
        return res

    def flip_vert(self):
        res = Picture(self.l, self.w, False)
        res.strs = list(reversed(self.strs.copy()))
        return res

    def flip_horiz(self):
        res = Picture(self.l, self.w, False)
        for i in range(self.w):
            res.strs[i] = self.strs[i][::-1]
        return res

    def flip_pairs(self):
        tmp = self.beside(self.flip_vert())
        return tmp.below(tmp)

    def right_split(self, n):

        def helper(painter, count):
            if (count == 0):
                return painter
            smaller = helper(Picture(painter.l // 2, painter.w // 2), count - 1)
            return painter.beside(smaller.below(smaller))

        return helper(self, n)

    def up_split(self, n):
        def helper(painter, count):
            if (count == 0):
                return painter
            smaller = helper(Picture(painter.l // 2, painter.w // 2), count - 1)
            return painter.below(smaller.beside(smaller))

        return helper(self, n)

    def corner_split(self, n):
        
        def helper(painter, count):
            if (count == 0):
                return painter

            tmp = Picture(painter.l // 2, painter.w // 2)
            up = tmp.up_split(count - 1)
            right = tmp.right_split(count - 1)

            top_left = up.beside(up)
            bottom_right = right.below(right)

            corner = helper(tmp, count - 1)

            return painter.below(top_left).beside(bottom_right.below(corner))

        return helper(self, n)

    def square_limit(self, n):
        quarter = self.corner_split(n)
        half = quarter.flip_horiz().beside(quarter)
        return half.flip_vert().below(half)


    def show(self):
        for s in self.strs:
            print(s)

class Picture_new(Picture):

    def square_of_four(self, tl, tr, bl, br):
        def helper(painter):
            top = tl(painter).beside(tr(painter))
            bottom = bl(painter).beside(br(painter))
            return bottom.below(top)
        return helper

    def flip_pairs(self):
        combine4 = self.square_of_four(identity, Picture_new.flip_vert,
                                       identity, Picture_new.flip_vert)
        return combine4(self)

    def square_limit(self, n):
        def rotate180(painter):
            return painter.flip_vert().flip_horiz()
        combine4 = self.square_of_four(Picture_new.flip_horiz, identity, 
                                       rotate180,              Picture_new.flip_vert)
        return combine4(self.corner_split(n))

    def split(self, proc1, proc2):

        def helper(painter, count):
            if (count == 0):
                return painter
            smaller = helper(Picture(painter.l // 2, painter.w // 2), count - 1)
            return proc1(painter, proc2(smaller, smaller))

        return lambda n: helper(self, n)

    def right_split(self, n):
        return self.split(Picture.beside, Picture.below)(n)
    def up_split(self, n):
        return self.split(Picture.below, Picture.beside)(n)
```

## 2.3 符号数据

### 2.3.1 引号

`scheme` 语法, 略去

### 2.3.2 实例: 符号求导

简单的实现
```python
def is_variable(e):
    return type(e) != int and type(e) != list

def is_same_variable(v1, v2):
    return is_variable(v1) and is_variable(v2) and (v1 == v2)

def is_sum(e):
    return type(e) == list and e[0] == '+'

def addend(e):
    return e[1]

def augend(e):
    return e[2]

def make_sum(a1, a2):
    if (a1 == 0 or a2 == 0):
        return a1 or a2
    if (is_number(a1) and is_number(a2)):
        return a1 + a2
    return ['+', a1, a2]

def is_product(e):
    return type(e) == list and e[0] == '*'

def multiplier(e):
    return e[1]

def multiplicand(e):
    return e[2]

def make_product(m1, m2):
    if (m1 == 0 or m2 == 0):
        return 0
    if (m1 == 1):
        return m2
    if (m2 == 1):
        return m1
    if (is_number(m1) and is_number(m2)):
        return m1 * m2
    return ['*', m1, m2]

def is_number(x):
    return type(x) == int

def deriv(exp, var):
    if (is_number(exp)):
        return 0
    elif (is_variable(exp)):
        return int(is_same_variable(exp, var))
    elif (is_sum(exp)):
        return make_sum(deriv(addend(exp), var),
                        deriv(augend(exp), var))
    elif (is_product(exp)):
        return make_sum(make_product(multiplier(exp),
                                     deriv(multiplicand(exp), var)),
                        make_product(deriv(multiplier(exp), var),
                                     multiplicand(exp)))

    raise AssertionError("表达式不合法")
```

功能更全面的实现:
```python
class Expression:
    def __init__(self, val = None):
        self.val = val

    def is_variable(self):
        return type(self.val) != int and type(self.val) != list

    def is_same_variable(self, other):
        return self.is_variable() and other.is_variable() and self.val == other.val

    def is_number(self):
        return type(self.val) == int

    def type_exp(self, which):
        return type(self.val) == list and self.val[0] == which

    def is_sum(self):
        return self.type_exp('+')

    def is_product(self):
        return self.type_exp('*')

    def is_exponentiation(self):
        return self.type_exp("**") and self.second().is_number()

    def __add__(self, other):
        a1, a2 = self.val, other.val
        if (a1 == 0):
            return Expression(a2)
        if (a2 == 0):
            return Expression(a1)
        if (is_number(a1) and is_number(a2)):
            return Expression(a1 + a2)
        return Expression(['+', a1, a2])

    def __mul__(self, other):
        m1, m2 = self.val, other.val
        if (m1 == 0 or m2 == 0):
            return Expression(0)
        if (m1 == 1):
            return Expression(m2)
        if (m2 == 1):
            return Expression(m1)
        if (is_number(m1) and is_number(m2)):
            return Expression(m1 * m2)
        return Expression(['*', m1, m2])

    def __pow__(self, other):
        p1, p2 = self.val, other.val
        if (p2 == 1):
            return Expression(p1)
        if (p2 == 0):
            return Expression(1)

        return Expression(["**", p1, p2])

    def first(self):
        if (type(self.val) == list):
            return Expression(self.val[1])
        raise AssertionError()

    def second(self):
        if (type(self.val) == list):
            if (len(self.val) > 3):
                return Expression([self.val[0], *self.val[2:]])
            return Expression(self.val[2])
        raise AssertionError()

    def deriv(self, var):

        def helper(exp, var):
            if (exp.is_number()):
                return Expression(0)
            elif (exp.is_variable()):
                return Expression(int(exp.is_same_variable(var)))
            elif (exp.is_sum()):
                return helper(exp.first(), var) + helper(exp.second(), var)
            elif (exp.is_product()):
                return exp.first() * helper(exp.second(), var) + \
                       helper(exp.first(), var) * exp.second()

            elif (exp.is_exponentiation()):
                a, b = exp.first(), exp.second()
                return b * (a ** Expression(b.val - 1)) * helper(a, var)

            raise AssertionError("表达式不合法")

        return helper(self, Expression(var))

    def __str__(self):
        return str(self.val)

```

## 2.3.3 集合的表示

朴素不可重复集合: 
1. `union_set`, $\Theta(n^2)$
2. `intersection_set`, $\Theta(n^2)$
3. `is_element_of_set`, $\Theta(n)$ 
4. `adjoin_set`, $\Theta(n)$

```python
class Set:
    def __init__(self):
        self.val = []

    def union_set(self, other): 
        res = Set()
        for x in other.val:
            if (self.is_element_of_set(x)):
                res.val.append(x)
        return res

    def intersection_set(self, other):
        res = Set()
        for x in other.val:
            res.adjoin_set(x)
        for x in self.val:
            res.adjoin_set(x)
        return res

    def is_element_of_set(self, val):
        for x in self.val:
            if val == x:
                return True
        return False

    def adjoin_set(self, val):
        if (not self.is_element_of_set(val)):
            self.val.append(val)

    def __str__(self):
        return str(self.val)

```

二叉搜索树:

```python
class Node:
    def __init__(self, val = None, left = None, right = None):
        self.val, self.left, self.right = val, left, right

    def __str__(self):
        return str(self.val)

class BST:
    def __init__(self, lst = None):
        lst = [] if lst == None else lst

        def list_to_tree(lst):
            if (len(lst) == 0):
                return None
            l_size = len(lst) // 2
            return Node(lst[l_size], list_to_tree(lst[:l_size]), 
                                     list_to_tree(lst[l_size + 1:]))

        self.root = list_to_tree([-1e9] + lst + [1e9])

    def is_element_of_set(self, val):
        
        def helper(cur_node):
            if (cur_node == None):
                return False
            if (cur_node.val == val):
                return True
            if (cur_node.val < val):
                return helper(cur_node.right)
            if (cur_node.val > val):
                return helper(cur_node.left)

        return helper(self.root)

    def adjoin_set(self, val):

        def helper(cur_node):
            if (cur_node == None):
                return Node(val)
            if (cur_node.val < val):
                cur_node.right = helper(cur_node.right)
            if (cur_node.val > val):
                cur_node.left = helper(cur_node.left)
            return cur_node

        self.root = helper(self.root)

        
    def tree_to_list(self):
        
        def helper(cur_node):
            if (cur_node == None):
                return []
            tmp = cur_node.val
            res = helper(cur_node.left)
            if (tmp != 1e9 and tmp != -1e9):
                res.append(cur_node.val)
            res += helper(cur_node.right)
            return res

        return helper(self.root)

    def union_set(self, other):
        a, b = self.tree_to_list(), other.tree_to_list()

        i, j, n, m = 0, 0, len(a), len(b)
        res = []
        while (i < n and j < m):
            if (a[i] < b[j]):
                res.append(a[i])
                i = i + 1
            if (a[i] > b[j]):
                res.append(b[j])
                j = j + 1
            if (a[i] == b[j]):
                res.append(a[i])
                i, j = i + 1, j + 1
        while (i < n):
            res.append(a[i])
            i = i + 1
        while (j < m):
            res.append(b[j])
            j = j + 1

        return BST(res)

    def intersection_set(self, other):
        a, b = self.tree_to_list(), other.tree_to_list()
        i, j, n, m, res = 0, 0, len(a), len(b), []
        while (i < n and j < m):
            if (a[i] < b[j]):
                i = i + 1
            if (a[i] > b[j]):
                j = j + 1
            if (a[i] == b[j]):
                res.append(a[i])
                i, j = i + 1, j + 1
        return BST(res)

    def __str__(self):

        def helper(cur_node, depth):
            if (cur_node == None):
                return ""
            res = ""
            if (cur_node.val != 1e9 and cur_node.val != -1e9):
                res = " " * depth * 5 + str(cur_node.val) + "\n"
            res += helper(cur_node.left, depth + 1) + \
                   helper(cur_node.right, depth + 1)
            return res

        return helper(self.root, 0)
```

### 2.3.4 实例: 哈夫曼编码树

编码方式: 
1. 定长编码: 用相同位数的二进制码表示字符, 占用空间大
2. 变长编码: 频繁出现的字符指定较短的二进制码\
    如何解决何时到达字符结尾:
    1. 类似摩斯电码, 加间隔符
    2. 使用前缀码, 每个字符的完整编码都不是其他字符的前缀, 如 `Haffman` 编码


`Haffman` 树: 树叶代表被编码的符号, 其余节点表示一个集合, 包含其下树叶的所有符号 \
符号的权重只在构建哈夫曼树的时候才能用到, 在编码和解码的时候不需要使用

1. 编码: 在哈夫曼树上, 向左走表示增加 0, 向右走表示增加 1
2. 解码: 对于字符串, 在哈夫曼树上走, 0 表示左走, 1 表示右走, 走到叶子节点就是这段编码完成\
         回到根节点继续解码
3. 生成哈夫曼树: 每次取权重最小的两个点合并

```python
class Leaf:
    def __init__(self, symbol, weight):
        self.symbol, self.weight = symbol, weight
    
class CodeTree:
    def __init__(self, left, right):
        self.left, self.right = left, right
        self.symbol = left.symbol + right.symbol
        self.weight = left.weight + right.weight

    def choose_branch(self, x):
        if (x == 0):
            return self.left
        if (x == 1):
            return self.right
        raise AssertionError('表达式不合法')

    def decode(self, bits):

        def decode_1(bits, cur_branch):
            if (len(bits) == 0):
                return []
            next_branch = cur_branch.choose_branch(bits[0])
            if (isinstance(next_branch, Leaf)):
                return [next_branch.symbol] + decode_1(bits[1:], self)
            return decode_1(bits[1:], next_branch)

        return decode_1(bits, self)

    def encode(self, message):
        def in_tree(char, cur_branch):
            return char in cur_branch.symbol

        def encode_symbol(char, cur_branch):
            if (isinstance(cur_branch, Leaf)):
                return []
            if (in_tree(char, cur_branch.left)):
                return [0] + encode_symbol(char, cur_branch.left)
            if (in_tree(char, cur_branch.right)):
                return [1] + encode_symbol(char, cur_branch.right)
            raise AssertionError(f"不存在字母{char}")
        
        if (len(message) == 0):
            return []
        return encode_symbol(message[0], self) + \
               self.encode(message[1:])

def adjoin_set(x, lst):
    if (len(lst) == 0):
        return [x]
    if (x.weight < lst[0].weight):
        return [x] + lst
    return [lst[0]] + adjoin_set(x, lst[1:])

def make_leaf_set(pairs):
    if (len(pairs) == 0):
        return []
    pair = pairs[0]
    return adjoin_set(Leaf(pair[0], pair[1]),
                      make_leaf_set(pairs[1:]))

def successive_merge(lst):

    if (len(lst) == 0):
        return None
    if (len(lst) == 1):
        return lst[0]
    
    a, b = lst[0], lst[1]
    new_tree = CodeTree(a, b)
    return successive_merge(adjoin_set(new_tree, lst[2:]))


def generate_huffman_tree(pairs):
    return successive_merge(make_leaf_set(pairs))


sample_tree = CodeTree(Leaf('A', 4),
                       CodeTree(Leaf('B', 2),
                                CodeTree(Leaf('D', 1),
                                         Leaf('C', 1))))

sample_tree = generate_huffman_tree([['A', 4], ['B', 2], ['C', 1], ['D', 1]])

assert(isinstance(sample_tree, CodeTree))

print(sample_tree.decode([0, 1, 1, 0, 0, 1, 0, 1, 0, 1, 1, 1, 0]))
print(sample_tree.encode(['A', 'D', 'A', 'B', 'B', 'C', 'A']))
```


## 2.4 抽象数据的多重表示

数据抽象是把使用该数据的程序设计和实现有理数的程序相分离, 构筑一道抽象屏障

但是同一种数据对象也可能有多种表示方法, 比如复数的直角坐标形式和极坐标形式 \
这就需要一个抽象屏障去隔离互不相同的设计选择

本章学习构造通用过程, 可以在不止一种数据表示上操作的过程 \
讨论数据导向的程序设计

### 2.4.1 复数的表示

```python
import math

def add(x, y):
    return CartesianComplex(x.real_part() + y.real_part(),
                            x.imag_part() + y.imag_part())

def sub(x, y):
    return CartesianComplex(x.real_part() - y.real_part(),
                            x.imag_part() - y.imag_part())

def mul(x, y):
    return PolarComplex(x.magn_part() * y.magn_part(),
                        x.angl_part() + y.angl_part())

def div(x, y):
    return PolarComplex(x.magn_part() / y.magn_part(),
                        x.angl_part() - y.angl_part())

class CartesianComplex:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def real_part(self):
        return self.x

    def imag_part(self):
        return self.y

    def magn_part(self):
        a = self.real_part() * self.real_part()
        b = self.imag_part() * self.imag_part()
        return math.sqrt(a + b)

    def angl_part(self):
        return math.atan(self.imag_part() / self.real_part())

    __add__ = add
    __sub__ = sub
    __mul__ = mul
    __truediv__ = div

class PolarComplex:

    def __init__(self, r, a):
        self.r, self.a = r, a

    def real_part(self):
        return self.r * math.cos(self.a)

    def imag_part(self):
        return self.r * math.sin(self.a)

    def magn_part(self):
        return self.r

    def angl_part(self):
        return self.a

    __add__ = add
    __sub__ = sub
    __mul__ = mul
    __truediv__ = div

```

### 2.4.2 带标志数据

当两种表示方式放在一起的时候, 可以使用一个变量来表示当前变量的类型 \
这是一种剥去和加上标志的规范方式, 是重要的组织策略

```python
import math


class Complex:
    def __init__(self, a, b, is_rectangle = True):
        self.a, self.b, self.p = a, b, is_rectangle

    def real_part(self):
        if (self.p):
            return self.a
        return self.a * math.cos(self.b)

    def imag_part(self):
        if (self.p):
            return self.b
        return self.a * math.sin(self.b)

    def magn_part(self):
        if (self.p):
            a = self.real_part() * self.real_part()
            b = self.imag_part() * self.imag_part()
            return math.sqrt(a + b)
        return self.a

    def angl_part(self):
        if (self.p):
            return math.atan(self.imag_part() / self.real_part())
        return self.b

    def __add__(self, y):
        return Complex(self.real_part() + y.real_part(),
                       self.imag_part() + y.imag_part(), True)

    def __sub__(self, y):
        return Complex(self.real_part() - y.real_part(),
                       self.imag_part() - y.imag_part(), True)

    def __mul__(self, y):
        return Complex(self.magn_part() * y.magn_part(),
                       self.angl_part() + y.angl_part(), False)

    def __truediv__(self, y):
        return Complex(self.magn_part() / y.magn_part(),
                       self.angl_part() - y.angl_part(), False)
```

## 2.4.3 数据导向的程序设计和可加性

2.4.2 节实现的复数类有两个弱点:
1. 如果想增加一种表示方式, 那就需要更改一系列选择函数
2. 保证在整个系统里不出现名字相同的过程

综上, 这种形式不具有可加性

一种称为 **数据导向** 的程序设计的编程技术提供了这种能力

在处理一系列针对不同类型的通用操作的时候, 就像是在处理二维表格 \
第一维 就是所有的可能类型, 第二维就是所有的可能操作

通过这种表示: 只需要向表格里新加项目即可, 而不需要改变通用操作的代码

```python
import math

dir = dict()

def put(op, t, item):
    if (dir.get(op) == None):
        dir[op] = dict()
    dir[op][t] = item

def get(op, t):
    return dir[op][t]


def install_rectangular_package():
    def real_part(x):
        return x[0]
    def imag_part(x):
        return x[1]
    def make_from_real_imag(x, y):
        return [x, y]
    def magnitude(z):
        a = real_part(z) * real_part(z)
        b = imag_part(z) * imag_part(z)
        return math.sqrt(a + b)
    def angle(z):
        return math.atan(imag_part(z) / real_part(z))
    def make_from_mag_ang(r, a):
        return [r * math.cos(a), r * math.sin(a)]
    def tag(x):
        return ["rectangular"] + x

    put("real_part", "rectangular", real_part)
    put("imag_part", "rectangular", imag_part)
    put("magnitude", "rectangular", magnitude)
    put("angle", "rectangular", angle)
    put("make_from_real_imag", "rectangular",
        lambda x, y: tag(make_from_real_imag(x, y)))
    put("make_from_mag_ang", "rectangular",
        lambda x, y: tag(make_from_mag_ang(x, y)))

def install_polar_package():
    def magnitude(z):
        return z[0]
    def angle(z):
        return z[1]
    def make_from_mag_ang(r, a):
        return [r, a]
    def real_part(x):
        return magnitude(x) * math.cos(angle(x))
    def imag_part(x):
        return magnitude(x) * math.sin(angle(x))
    def make_from_real_imag(x, y):
        return [math.sqrt(x * x + y * y), math.atan(y / x)]
    
    def tag(x):
        return ["polar"] + x

    put("real_part", "polar", real_part)
    put("imag_part", "polar", imag_part)
    put("magnitude", "polar", magnitude)
    put("angle", "polar", angle)
    put("make_from_real_imag", "polar",
        lambda x, y: tag(make_from_real_imag(x, y)))
    put("make_from_mag_ang", "polar",
        lambda x, y: tag(make_from_mag_ang(x, y)))
       
install_rectangular_package()
install_polar_package()

def apply_generic(op, arg):
    type_tags = arg[0]
    proc = get(op, type_tags)
    return proc(arg[1:])

def real_part(z):
    return apply_generic("real_part", z)

def imag_part(z):
    return apply_generic("imag_part", z)

def magnitude(z):
    return apply_generic("magnitude", z)

def angle(z):
    return apply_generic("angle", z)

def make_from_real_imag(x, y):
    return get("make_from_real_imag", "rectangular")(x, y)

def make_from_mag_ang(x, y):
    return get("make_from_mag_ang", "polar")(x, y)

```

在数据导向的程序设计里, 最关键的想法就是 \
显式处理 **操作-类型表格** 从而管理程序的通用性操作

在 2.4.2 里是一种基于类型进行分配的组织方式, 每个操作管理自己的类型 \
相当于把表格按照行分割

另一种实现策略是把表格按照列分割, 类似之前只使用函数来构建一个抽象数据的过程 \
先传入数据, 返回一个函数, 在传入操作类型执行操作

```python
def make_from_real_imag(x, y):
    def dispatch(op):
        if (op == "real_part"):
            return x
        if (op == "imag_part"):
            return y
        if (op == "magnitude"):
            return math.sqrt(x * x + y * y)
        if (op == "angle"):
            return math.atan(y / x)
    return dispatch
```

## 2.5 带有通用型操作的系统

使用数据导向技术构造一个算数包, 包含有理数算数和复数算数以及常规算数

### 2.5.1 通用型算数运算

```python
import math

dir = dict()
def put(a, b, c):
    if (dir.get(a) == None):
        dir[a] = dict()
    dir[a][b] = c
get = lambda a, b: dir[a][b]


add_tag = lambda tag, x: [tag] + x

def install_number_package():
    put("add", ("number", "number"), 
        lambda x, y: x + y)
    put("sub", ("number", "number"), 
        lambda x, y: x - y)
    put("mul", ("number", "number"), 
        lambda x, y: x * y)
    put("div", ("number", "number"), 
        lambda x, y: x / y)

    put("make", "number", lambda x: x)

    put("is_zero", ("number",), 
        lambda x: x == 0)

    put("raise", ("number",),
        lambda x: make_rational(x, 1))

def install_rational_package():
    tag = lambda x: add_tag("rational", x)
    numer = lambda x: x[0]
    denom = lambda x: x[1]
    def make_rat(n, d):
        g = math.gcd(n, d)
        return [n // g, d // g]
    def add_rat(x, y):
        return make_rat(numer(x) * denom(y) + denom(x) * numer(y),
                        denom(x) * denom(y))
    def sub_rat(x, y):
        return make_rat(numer(x) * denom(y) - denom(x) * numer(y),
                        denom(x) * denom(y))
    def mul_rat(x, y):
        return make_rat(numer(x) * numer(y),
                        denom(x) * denom(y))
    def div_rat(x, y):
        return make_rat(numer(x) * denom(y),
                        denom(x) * numer(y))

    put("add", ("rational", "rational"),
        lambda x, y: tag(add_rat(x, y)))
    put("sub", ("rational", "rational"),
        lambda x, y: tag(sub_rat(x, y)))
    put("mul", ("rational", "rational"),
        lambda x, y: tag(mul_rat(x, y)))
    put("div", ("rational", "rational"),
        lambda x, y: tag(div_rat(x, y)))
    put("make", "rational", 
        lambda n, d: tag(make_rat(n, d)))

    put("numer", ("rational", ), numer)
    put("denom", ("rational", ), denom)

    put("is_zero", ("rational",),
        lambda x: denom(x) == 0)

    put("raise", ("rational",),
        lambda x: make_real(numer(x) / denom(x)))

def install_real_package():
    tag = lambda x: add_tag("real", x)

    put("add", ("real", "real"), 
        lambda x, y: tag([x[0] + y[0]]))
    put("sub", ("real", "real"), 
        lambda x, y: tag([x[0] - y[0]]))
    put("mul", ("real", "real"), 
        lambda x, y: tag([x[0] * y[0]]))
    put("div", ("real", "real"), 
        lambda x, y: tag([x[0] / y[0]]))
    put("make", "real", lambda x: tag([x]))

    put("is_zero", ("real",), 
        lambda x: x == 0)

    put("raise", ("real",),
        lambda x: make_complex_from_real_imag(x[0], 0))

def install_complex_package():
    def install_rectangular_package():
        def real_part(x):
            return x[0]
        def imag_part(x):
            return x[1]
        def make_from_real_imag(x, y):
            return [x, y]
        def magnitude(z):
            a = real_part(z) * real_part(z)
            b = imag_part(z) * imag_part(z)
            return math.sqrt(a + b)
        def angle(z):
            return math.atan(imag_part(z) / real_part(z))
        def make_from_mag_ang(r, a):
            return [r * math.cos(a), r * math.sin(a)]

        tag = lambda x: add_tag("rectangular", x)

        put("real_part", ("rectangular",), real_part)
        put("imag_part", ("rectangular",), imag_part)
        put("magnitude", ("rectangular",), magnitude)
        put("angle", ("rectangular",), angle)
        put("make_from_real_imag", "rectangular",
            lambda x, y: tag(make_from_real_imag(x, y)))
        put("make_from_mag_ang", "rectangular",
            lambda x, y: tag(make_from_mag_ang(x, y)))

        put("is_zero", ("rectangular",),
            lambda x: real_part(x) == 0 and imag_part(x) == 0)

    def install_polar_package():
        def magnitude(z):
            return z[0]
        def angle(z):
            return z[1]
        def make_from_mag_ang(r, a):
            return [r, a]
        def real_part(x):
            return magnitude(x) * math.cos(angle(x))
        def imag_part(x):
            return magnitude(x) * math.sin(angle(x))
        def make_from_real_imag(x, y):
            return [math.sqrt(x * x + y * y), math.atan(y / x)]
        
        tag = lambda x: add_tag("polar", x)

        put("real_part", ("polar",), real_part)
        put("imag_part", ("polar",), imag_part)
        put("magnitude", ("polar",), magnitude)
        put("angle", ("polar",), angle)
        put("make_from_real_imag", "polar",
            lambda x, y: tag(make_from_real_imag(x, y)))
        put("make_from_mag_ang", "polar",
            lambda x, y: tag(make_from_mag_ang(x, y)))

        put("is_zero", ("polar",),
            lambda x: magnitude(x) == 0)

    install_rectangular_package()
    install_polar_package()

    make_from_real_imag = lambda x, y: get("make_from_real_imag", "rectangular")(x, y)
    make_from_mag_ang = lambda r, a: get("make_from_mag_ang", "polar")(r, a)

    def add_complex(x, y):
        return make_from_real_imag(real_part(x) + real_part(y),
                                   imag_part(x) + imag_part(y))

    def sub_complex(x, y):
        return make_from_real_imag(real_part(x) - real_part(y),
                                   imag_part(x) - imag_part(y))

    def mul_complex(x, y):
        return make_from_mag_ang(magnitude(x) * magnitude(y),
                                 angle(x) + angle(y))

    def div_complex(x, y):
        return make_from_mag_ang(magnitude(x) / magnitude(y),
                                 angle(x) - angle(y))

    tag = lambda x: add_tag("complex", x)

    put("add", ("complex", "complex"),
        lambda x, y: tag(add_complex(x, y)))
    put("sub", ("complex", "complex"),
        lambda x, y: tag(sub_complex(x, y)))
    put("mul", ("complex", "complex"),
        lambda x, y: tag(mul_complex(x, y)))
    put("div", ("complex", "complex"),
        lambda x, y: tag(div_complex(x, y)))

    put("real_part", ("complex",), real_part)
    put("imag_part", ("complex",), imag_part)
    put("magnitude", ("complex",), magnitude)
    put("angle", ("complex",), angle)
        
    put("make_from_real_imag", "complex", 
        lambda x, y: tag(make_from_real_imag(x, y)))
    put("make_from_mag_ang", "complex", 
        lambda x, y: tag(make_from_mag_ang(x, y)))

    put("is_zero", ("complex",),
        lambda x: is_zero(x))

dir_depth = dict()
def install_depth_package():
    lst = ["number", "rational", "real", ["complex", "rectangular", "polar"]]
    for i, c in enumerate(lst):
        if (type(c) == list):
            for x in c:
                dir_depth[x] = i
        else:
            dir_depth[c] = i

is_number = lambda x: type(x) == int

def apply_generic(op, *args):
    args = list(args)

    get_type = lambda x: "number" if is_number(x) else x[0]
    get_content = lambda x: x if is_number(x) else x[1:]

    cur_type = tuple(map(get_type, args))
    max_type = max([dir_depth[x] for x in cur_type])

    for i in range(len(args)):
        while (dir_depth[get_type(args[i])] != max_type):
            args[i] = my_raise(args[i])

    aft_type = tuple(map(get_type, args))
    proc = get(op, aft_type)

    return proc(*list(map(get_content, args)))

add = lambda x, y: apply_generic("add", x, y)
sub = lambda x, y: apply_generic("sub", x, y)
mul = lambda x, y: apply_generic("mul", x, y)
div = lambda x, y: apply_generic("div", x, y)

make_number = lambda n: get("make", "number")(n)

numer = lambda x: apply_generic("numer", x)
denom = lambda x: apply_generic("denom", x)

make_rational = lambda n, d: get("make", "rational")(n, d)

make_real = lambda x: get("make", "real")(x)

real_part = lambda x: apply_generic("real_part", x)
imag_part = lambda x: apply_generic("imag_part", x)
magnitude = lambda x: apply_generic("magnitude", x)
angle = lambda x: apply_generic("angle", x)

make_complex_from_real_imag = lambda x, y: get("make_from_real_imag", "complex")(x, y)
make_complex_from_mag_ang = lambda x, y: get("make_from_mag_ang", "complex")(x, y)

equal = lambda x, y: x == y
is_zero = lambda x: apply_generic("is_zero", x)
my_raise = lambda x: apply_generic("raise", x)

install_number_package()
install_rational_package()
install_real_package()
install_complex_package()
install_depth_package()
```

### 2.5.2 不同类型数据的组合

目前定义的所有运算, 都把不同数据类型看成完全分离的东西 \
因此没有办法完成诸如 `number` 加上 `complex` 的操作

当然, 可以再定义复数加数字, 数字加复数两个函数, 从而达成目的 \
但是对于这样的系统, 引进一个新类型的代价就是不仅仅构造出针对这个类型的所有函数 \
还需要构造并安装好所有实现跨类型工作的函数, 这就是个非常复杂的任务

一般来说, 需要操作的两个类型都存在一种方式, 使得可以把一种类型看作另一种类型

比如数字加复数就是一个虚部为 0 的复数加复数

```python
def new_apply_generic(op, *args):
    get_type = lambda x: "number" if is_number(x) else x[0]
    get_content = lambda x: x if is_number(x) else x[1:]

    type_tags = tuple(map(get_type, args))
    if (dir[op].get(type_tags) != None):
        return get(op, type_tags)(*list(map(get_content, args)))

    if (len(args) == 2):
        type1, type2 = type_tags[0], type_tags[1]
        a1, a2 = args[0], args[1]
        x, y = get_coercion(type1, type2), get_coercion(type2, type1)

        if (x):
            return apply_generic(op, x(a1), a2)
        if (y):
            return apply_generic(op, a1, y(a2))

dir_coercion = dict()
def put_coercion(a, b, c):
    if (dir_coercion.get(a) == None):
        dir_coercion[a] = dict()
    dir_coercion[a][b] = c

def get_coercion(a, b):
    if (dir_coercion.get(a) != None and 
        dir_coercion[a].get(b) != None):
        return dir_coercion[a][b]
    return False

def number_to_complex(n):
    return make_complex_from_real_imag(n, 0)

put_coercion("number", "complex", number_to_complex)

add = lambda x, y: new_apply_generic("add", x, y)

x = make_number(3)
y = make_complex_from_real_imag(4, 2)
print(add(x, y))
```

对于 $n$ 个类型的系统, 转换函数可能需要写 $n^2$ 个

当然可以这两种类型转化成第三种类型来计算

类型的层次结构: 整数 -> 有理数 -> 实数 -> 复数 \
整数是有理数的子类型, 有理数是整数的超类型

类型塔: 一个类型只有至多一个超类型和至多一个子类型

可以采用逐步提高层级或者逐步下放层级的方式来实现类型统一

类型塔的另外一个优点, 每个类型能够 **继承** 其超类型中的所有操作 

**层次结构的不足**:
一个超类型可能有多种子类型, 一个子类型可能有多个超类型, \
所以并不存在唯一方式在层次结构中去提高一个层级或者下放一个层级

设计大型系统时, 处理一大批相互有关的类型而同时保持模块性是一个非常困难的问题, 目前仍在深入研究





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


## 2.44 ~ 2.52

目前以作者能力无法实现 `painter`, 故无法运行代码 \
难以确认作业代码的正确性, 故跳过

## 2.53

```scheme
(a b c)
((george))
(y1 y2)
#f
#f
#t
```

## 2.54

```python
def equal(lst1, lst2):
    if (len(lst1) != len(lst2)):
        return False
    for i in range(len(lst1)):
        if (lst1[i] != lst2[i]):
            return False
    return True

```

## 2.55

```scheme
(car ''abracadabra)
(car '(quote abracadabra))
(car (quote abracadabra))
quote
```


## 2.56 

见正文 "功能更全面的实现" 的 `is_exponentiation` 和 `__pow__` 函数

## 2.57

在正文 "功能更全面的实现" 中 `second` 函数多加两行代码即可

## 2.58

更改 
```python
class MidExpression(Expression):
    def first(self):
        if (type(self.val) == list):
            return MidExpression(self.val[0])
        raise AssertionError()

    def type_exp(self, which):
        return type(self.val) == list and self.val[1] == which

    def __pow__(self, other):
        p1, p2 = self.val, other.val
        if (p2 == 1):
            return MidExpression(p1)
        if (p2 == 0):
            return MidExpression(1)

        return MidExpression([p1, "**", p2])

    def __add__(self, other):
        a1, a2 = self.val, other.val
        if (a1 == 0):
            return MidExpression(a2)
        if (a2 == 0):
            return MidExpression(a1)
        if (is_number(a1) and is_number(a2)):
            return MidExpression(a1 + a2)
        return MidExpression([a1, "+", a2])

    def __mul__(self, other):
        m1, m2 = self.val, other.val
        if (m1 == 0 or m2 == 0):
            return MidExpression(0)
        if (m1 == 1):
            return MidExpression(m2)
        if (m2 == 1):
            return MidExpression(m1)
        if (is_number(m1) and is_number(m2)):
            return MidExpression(m1 * m2)
        return MidExpression([m1, "*", m2])


```

至于不带括号的表达式, 需要构建表达式树 \
无法通过修改谓词和选择函数来解决

## 2.59

见正文 `union`

## 2.60

1. `adjoin_set`, $\Theta(1)$
2. `union_set`, $\Theta(n)$
其他复杂度相同

但是这样做总体复杂度的系数比较大 \
在插入操作较多时建议使用这种, 否则使用上一种

```python
class MultiSet:
    def __init__(self, lst = None):
        self.val = lst if lst != None else []

    def is_element_of_set(self, val):
        for x in self.val:
            if (x == val):
                return True
        return False

    def adjoin_set(self, val):
        self.val.append(val)

    def union_set(self, other):
        return MultiSet(self.val + other.val)

    def intersection_set(self, other):
        res = MultiSet()
        for x in other.val:
            if (self.is_element_of_set(x)):
                res.val.append(x)
        return res

    def __str__(self):
        return str(self.val)
```

## 2.61 ~ 2.62

```python
class SortedSet:
    def __init__(self):
        self.val = []
    def copy(self):
        res = SortedSet()
        for x in self.val:
            res.val.append(x)
        return res

    def is_element_of_set(self, val):
        for x in self.val:
            if (x == val):
                return True
            if (x > val):
                return False

        return False

    def adjoin_set(self, val):
        pos, n = 0, len(self.val)
        while (pos < n and val > self.val[pos]):
            pos = pos + 1
        if (pos == n or (pos < n and val != self.val[pos])):
            self.val.insert(pos, val)


    def union_set(self, other):
        i, j, n, m = 0, 0, len(self.val), len(other.val)
        res = SortedSet()
        while (i < n and j < m):
            if (self.val[i] < other.val[j]):
                res.val.append(self.val[i])
                i = i + 1
            if (self.val[i] > other.val[j]):
                res.val.append(other.val[j])
                j = j + 1
            if (self.val[i] == other.val[j]):
                res.val.append(self.val[i])
                i, j = i + 1, j + 1
        while (i < n):
            res.val.append(self.val[i])
            i = i + 1
        while (j < m):
            res.val.append(other.val[j])
            j = j + 1
        return res


    
    def __str__(self):
        return str(self.val)
```

## 2.63

产生同样结果 \
第一种方法使用 `append` 复杂度为 $\Theta(n)$, 总复杂度为 $\Theta(n^2)$ \
第二种方法使用 `cons` 复杂度为 $\Theta(1)$, 总复杂度为 $\Theta(n)$

第二种方法更优

## 2.64

往下递归, 左半部分是左子树, 右半部分是右子树

```python
7
     3
          1
          5
     11
          9
```

## 2.65

见正文

## 2.66

残缺版, 没有 `key` 函数

```python
def lookup(given_key, dir):
    if (dir == None):
        return False
    entry_key = key(dir.val)
    if (entry_key == given_key):
        return dir.val
    if (entry_key > given_key):
        return lookup(given_key, dir.right)
    if (entry_key < given_key):
        return lookup(given_key, dir.left)
```


## 2.67

```python
ADABBCA
```

## 2.68

见正文

得到结果相同

## 2.69

见正文

## 2.70

```python
tree = generate_huffman_tree([["a", 2], ["na", 16], ["boom", 1], ["sha", 3],
                              ["get", 2], ["yip", 9], ["job", 2], ["wah", 1]])

assert(isinstance(tree, CodeTree))

print(tree.encode(["get", "a", "job"]))
print(tree.encode(["sha", "na", "na", "na", "na", "na", "na", "na", "na"]))
print(tree.encode(["get", "a", "job"]))
print(tree.encode(["sha", "na", "na", "na", "na", "na", "na", "na", "na"]))
print(tree.encode(["wah", "yip", "yip", "yip", "yip", "yip", "yip", "yip", "yip"]))
print(tree.encode(["sha", "boom"]))
```

```python
[1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0]
[1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0]
[1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0]
```

总共的二进制位数量是 84 \
使用定长编码数量是 36 * 3 = 108

## 2.71

```
        *
       /\
      *  16
     /\
    *  8
   / \
  *   4
 /\
1  2

                  *
                 /\
                *  512
               /\
              *  256
             /\
            * 128
           /\
          *  64
         /\
        *  32
       /\
      *  16
     /\
    *  8
   / \
  *   4
 /\
1  2
```
最频繁 1 个, 最不频繁 $n - 1$ 个

## 2.72

对于特殊情况

编码一个最频繁: $\Theta(1)$, 一个最不频繁: $\Theta(n)$

## 2.73

```python

dir = dict()

def is_number(x):
    return type(x) == int
def is_variable(exp):
    return type(exp) != int and type(exp) != list
def is_same_variable(exp, var):
    return is_variable(exp) and is_variable(var) and exp == var

def operator(exp):
    return exp[0]
def operands(exp):
    if (len(exp) > 3):
        return exp[1], [operator(exp), *exp[2:]]
    return exp[1], exp[2]

def get(a, b):
    return dir[a][b]
def put(a, b, func):
    if (dir.get(a) == None):
        dir[a] = dict()
    dir[a][b] = func

def deriv(exp, var):
    if (is_number(exp)):
        return 0
    if (is_variable(exp)):
        return int(is_same_variable(exp, var))

    return get("deriv", operator(exp))(*operands(exp), var)

def install_deriv_package():

    def add_helper(a, b):
        if (is_number(a) and is_number(b)):
            return a + b
        elif (a == 0):
            return b
        elif (b == 0):
            return a
        return ["+", a, b]

    def mul_helper(a, b):
        if (a == 1):
            return b
        if (b == 1):
            return a
        if (a == 0 or b == 0):
            return 0
        return ["*", a, b]

    def pow_helper(a, b):
        if (b == 0):
            return 1
        if (b == 1):
            return a
        return ["**", a, b]

    def add(a, b, var):
        return add_helper(deriv(a, var), deriv(b, var))
    def mul(a, b, var):
        return add_helper(mul_helper(a, deriv(b, var)),
                          mul_helper(deriv(a, var), b))
    def pow(a, b, var):
        return mul_helper(b - 1, 
                          mul_helper(pow_helper(a, b - 1),
                                     deriv(a, var)))

    put("deriv", "*", mul)
    put("deriv", "+", add)
    put("deriv", "**", pow)

install_deriv_package()

exp = ["*", 3, "x", "y", "x", ["+", "x", "1"]]
exp = ["**", "x", 3]

print(deriv(exp, "x"))
        
```

1. 把之前的 `is_sum`, `is_product` 等函数用 `get` 的第二个参数代替, 从而实现选择的功能 \
   因为 `number?` 和 `is_same_variable` 后面只有一个参数
2. 见代码
3. 加入乘幂
4. 除了题中的改动, 只需要改动 `install_deriv_package` 下面的 `put` 语句

## 2.74

题目不明确, 跳过


## 2.75

```python
def make_from_mag_ang(r, a):
    def dispatch(op):
        if (op == "real_part"):
            return r * math.cos(a)
        if (op == "imag_part"):
            return r * math.sin(a)
        if (op == "magnitude"):
            return r
        if (op == "angle"):
            return a
    return dispatch

```


## 2.76

1. 显式分派:
    1. 增加新操作需要使用者避免命名冲突
    2. 增加新类型需要改动通用操作
    3. 结论: 输
2. 数据导向:
    1. 很方便地通过包机制增加新类型和新的通用操作
    2. 结论: 赢
3. 消息传递:
    1. 将数据对象和操作整合在一起, 可以很方便地增加新类型
    2. 增加新操作时所有对象都要全部重新实例化
    4. 结论: 寄

## 2.77

代码见正文

因为不加那几行代码的话, 直接调用的是 `complex` 包里面的 `magnitude` 操作 \
此时 `complex` 包里面并没有这个操作, 所以加上就好了

`magnitude` 有三个, 第一个是最外层的, 第二个是 `complex` 包里面的, 第三个是 `rectangle` 里面的\

`apply_generic` 函数在前两次 `magnitude` 中被调用了两次\
第一次剥去 `complex`, 第二次剥去 `rectangle`

## 2.78

见正文

## 2.79

见正文

## 2.80

见正文

## 2.81

1. 没有相应的类型转换操作
2. 如果没有相应的操作, 那么就进行自己到自己的类型转换 \
   从而陷入死循环, 寄
2. 加一个 `if` 即可, 不写了

## 2.82

举例: 只有 `complex` 才实现了这个功能, 而传入的参数最高才是 `rational`  \
这样永远都找不到合适的过程

## 2.83

见正文中各个包的 `raise` 部分以及通用操作的 `raise`

## 2.84

见正文 `install_depth_package`  使用了一种比较简单的形式

## 2.85


