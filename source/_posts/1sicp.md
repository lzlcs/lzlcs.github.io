---
title: SICP 学习笔记 (第一章) 
date: 2023-07-02 15:04:05
mathjax: true
tags:
- Scheme
- Python
- SICP
categories: 
- Book
---

习题和书中代码使用 `python` 重写

-----

# Chapter 1: 构造函数抽象

## 1.1 程序设计的基本元素

三种机制
1. 基本表达形式: 最简单的元素
2. 组合的方法: 构造复合元素
3. 抽象的方法: 命名并操作复合元素

### 1.1.1 表达式

在 `scheme` 中: 

一些十进制的数字是表达式, 可以用表示函数的基本元素(如 `+`, `*`) 来组合这些数字表达式 \
表示把有关函数应用到这些数字上面, 形成类似 `(+ 3 4)` 的组合式 \
`(运算符 运算对象1 运算对象2 ....)` 这种运算符在最前的表达方式是前缀表达式
1. 可以适用任意个运算对象而不出现歧义 
2. 允许运算对象本身也是个组合式

解释器通常按照 **读入-求值-打印** 循环求出表达式的值, 无论表达式多复杂

在 `python` 解释器中, 直接输入正常的中缀表达式就可以打印结果

### 1.1.2 命名和环境

```python
x = 2
```
这是把值绑定到变量名的方式, 是最简单的抽象方法 \
忽略计算值的一系列函数而直接使用名称从而简化程序

要实现这种绑定的功能, 解释器必须具备存储的能力 \
这种存储称之为环境(全局环境)

### 1.1.3 组合式的求值

1. 对每个运算对象求值, 这一求值函数是递归的
2. 对每个运算对象求值的结果应用运算符

可以采用树的形式来表示组合式的求值函数, 组合式的值为父节点, 子节点为运算符和运算对象 \
自底向上计算, 在越来越高的层次中组合起来, 这种计算函数称为 **树形积累**

叶子节点可以分为以下几类:
1. 数字, 代表它本身
2. 运算符, 代表操作方式
3. 其他: 在环境中寻找绑定的值
4. 特殊形式

### 1.1.4 复合函数

**函数定义** 可以为复合操作提供名字, 从而在之后的运算中作为一个单元来使用

```python
def 函数名称(形参列表):
    函数主体
def square(x):
    return x
```
函数名称和形参列表写在一起, 和调用这个函数的时候一样
```python
square(4) #16
```
此时实际参数 4 取代形式参数 `x` 进行运算, 结果为 16 \
`square(4)` 也称为一个表达式, 可以被嵌入其他表达式中

```python
def sum_square(x, y):
    return square(x) + square(y)
def f(x):
    return sum_square(x + 1, x * 2)
```

### 1.1.5 函数应用的代换模型

**代换模型** 就是将函数体的形参用实参代替之后, 对函数体求值的计算函数
> 替换只是一个最简单的模型, 之后会讨论其他更精细的模型

* 正则序求值: 先把所有函数都展开, 最后统一求值
```python
f(5)
sum_square(5 + 1, 5 * 2)
square(5 + 1) + square(5 * 2)
(5 + 1) * (5 + 1) + (5 * 2) * (5 * 2)
```
* 应用序求值: 先求值所有参数, 之后再展开函数
```python
sum_square(6, 10)
(6 * 6) + (10 * 10)
36 + 100
136
```
1. `Lisp`采用应用序求值, 这样可以避免一些重复计算
1. 在无法使用简单的替换来模拟的函数中, 正则序更加复杂
1. 但在某些方面, 正则序也可以成为重要工具, 如"无限"数据结构

### 1.1.6 条件表达式和谓词

`if` 关键字针对这种 **分情况分析**
```python
if (p1):
    e1
elif (p2):
    e2
elif (p3):
    e3
else:
    e4
```

这些 `p1`, `p2` 等被称作**谓词**, 是一个结果为 `True`(真) 或 `False`(假) 的表达式\
流程就是找到第一个结果为真的谓词, 之后执行其后的表达式 \
基本谓词: `>=`, `<=`, `>`, `<`, `==` \
复合谓词运算符: `and`, `or`, `not`
```python
(e1) and (e2) and ... and (en) # 对所有表达式做 and 操作
(e1) or (e2) or ... or (en) # 对所有表达式做 or 操作
not (expression) # 对这个表达式取反
```
`and` 和 `or` 都有短路特性, 具体表现为:
1. 当 `and` 后某个表达式为 `False` 时, 后面的表达式不再求值
2. 当 `or` 后某个表达式为 `True` 时, 后面的表达式不再求值

### 1.1.7 实例: 牛顿法求平方根

定义平方根函数 $\sqrt x = y, 使得y>=0且 y^2 =x$ \
1. 说明性描述: 注重是什么
2. 行动性描述: 注重怎么办

牛顿法求平方根函数:
对 $\sqrt x$ 的值有一个猜测 $y$, 求出 $y$ 和 $\frac{x}{y}$的平均数作为新的猜测

```python
def square(x):
    return x * x
def good_enough(guess, x):
    return abs(square(guess) - x) < 0.001
def average(a, b):
    return (a + b) / 2
def improve(guess, x):
    return average(guess, (x / guess))
def sqrt(guess, x):
    if (good_enough(guess, x)):
        return guess
    else:
        return sqrt(improve(guess, x), x)

print(sqrt(1.0, 2))
```

### 1.1.8 函数作为黑箱抽象

**函数抽象** 封装一个函数, 调用这个函数的时候只需要知道它能返回什么而不需要知道它的实现细节

以平方根的计算函数举例:
* 一个 `sqrt` 函数可以分解为 `good-enough`, `improve`, `average`, `abs` 等函数
* 也就是把 `sqrt` 分解成了可以清楚标明的工作
* 在实现 `sqrt` 的函数中, 不需要考虑这些辅助函数的细节, 只需要相信它们

**局部名**: 函数中形参的名字, 是函数使用者不需要关注的内容
* **约束变量** 形参的名字所代表的变量
* **自由变量** 不是被约束的变量

在抽象函数时, 如果对约束变量有任何改变, 这样就会影响外部的函数\
因为这个变量可能在外部函数中还被用到, 这个内部的函数也就不是"黑箱"

**作用域**: 一个约束变量所能生效的表达式 \
比如函数主体就是形参的作用域

在 `good-enough` 中, `guess` 和 `x` 是约束变量, `<`, `abs` 等则不是 \
所以约束变量的名字要避开自由变量的名词, 从而避免发生冲突

----
现在 `sqrt` 函数由一些相互分离的函数定义, 而用户只需要用到 `sqrt` \
在构建大型程序时, 过多的辅助函数往往会扰乱视线, 还可能出现重名的情况 \
所以可以把这些辅助函数局部化, 这就需要内部定义

注意到所有的辅助函数都在 `sqrt` 的形参 `x` 的作用域内, 所以不用来回传 `x`的值

```python
def sqrt(x):
    def square(x):
        return x * x
    def good_enough(guess):
        return abs(square(guess) - x) < 0.001
    def average(a, b):
        return (a + b) / 2
    def improve(guess):
        return average(guess, (x / guess))
    def iter(guess):
        if (good_enough(guess)):
            return guess
        else:
            return iter(improve(guess))
    return iter(x)

print(sqrt(1.0, 2))
```

在辅助函数中 `x` 是自由变量, 由外部传给 `x` 的实际参数决定 \
这种方法被称作**词法作用域**, 也叫**静态作用域**\
在定义本函数的环境中寻找 `x` 在哪里被定义


## 1.2 函数及其产生的计算

本节考察函数所产生计算的顺序和消耗时间空间资源的速率

### 1.2.1 线性的递归和迭代

1. **递归计算函数**: 
    ```python
    def factorial(n):
        if (n == 0):
            return 1
        else:
            return n * factorial(n - 1)
    ```
    * 在展开阶段, 递归构造一个**推迟进行**的操作所构成的链条
    * 在收缩阶段, 实际执行这些运算

    所以解释器要维护那些之后要执行的收缩阶段的轨迹 \
    轨迹的长度与 $n$ 成正比, 所以说这是一个 **线性递归函数**
    ```scheme
    (factorial 6)
    (* 6 (factorial 5))
    (* 6 (* 5 (factorial 4)))
    (* 6 (* 5 (* 4 (factorial 3))))
    (* 6 (* 5 (* 4 (* 3 (factorial 2)))))
    (* 6 (* 5 (* 4 (* 3 (* 2 (factorial 1))))))
    (* 6 (* 5 (* 4 (* 3 (* 2 1)))))
    (* 6 (* 5 (* 4 (* 3 2))))
    (* 6 (* 5 (* 4 6)))
    (* 6 (* 5 24))
    (* 6 120)
    720
    ```
2. **迭代计算函数**
    ```python
    def fact(n):
        def iter(count, res):
            if (count > n):
                return res
            else:
                return iter(count + 1, res * count)
        return iter(1, 1)
    ```
    (注意在 `python` 中不会由有这个迭代的优化, 据说是为了保存递归状态) \
    在这个计算函数中没有任何的展开和收缩, 需要保存的只有 `product` 和 `counter` \
    这种可以使用固定状态描述计算函数的就是迭代计算函数 \
    同样, 这也是 **线性迭代函数**

> 注意递归函数和递归计算函数并不同, 递归函数是一个语法上的概念 \
> 上述的迭代计算函数的具体实现也是采用递归函数

在迭代计算函数中, 形参都提供了计算状态的完整描述, 如果因为某些不可抗力程序终止 \
那么需要重新唤醒这个程序, 只需要提供两个形参即可 \
而递归计算函数还有一些隐含的变量并没有显式地保存在程序变量中, 而是由解释器维护 \
所以递归计算函数的链条越长, 解释器需要保存的信息也越多

在其他大多数语言中, 即使是迭代计算函数, 只要使用递归函数实现, 那么消耗的空间就是线性的 \
所以需要借助一些语法糖比如 `for`, `while` 才能实现迭代计算函数

而`scheme`的解释器实现没有这个缺点, 只要是迭代计算函数, 消耗的空间就是常数级别
具有这一特性的实现是 **尾递归** 的

### 1.2.2 树形递归

考虑斐波那契数列的计算, $f(n) = f(n-1)+f(n-2)$ \
每次计算的时候都要调用两次自身, 画成结构图就是一棵树
```python
def fib(n):
    if (n <= 1):
        return 1
    else:
        return fib(n - 1) + fib(n - 2)
```
一般来说, 树形递归结构的深度代表空间占用, 节点数代表运算次数(时间占用)

这种树形递归求$f(n)$ 的节点数正好是$f(n+1)$, 而斐波那契数列的值是成指数增长 \
所以这种计算的方式效率极低, 可以采用线性迭代计算函数
```python
def fib(n):
    def iter(count, a, b):
        if (count > n):
            return b
        else:
            return iter(count + 1, b, a + b)
    return iter(1, 0, 1)
```
总体来说, 树形递归在这个问题上效率较低, 但非常直观 \
然而在其他的一些问题上, 树形递归不仅直观而且效率很高, 比如前序遍历

---
实例: 换零钱
将总数为 $a$ 的现金换成硬币可以由以下两种情况
* 将总数为 $a$ 的现金换成第一种硬币之外的所有其他硬币的方式的数目
* 将总数为 $a-d$ 的现金换成所有种类的硬币的数目
两者相加即为答案

其中分类的标准是有没有使用第一种硬币, 第一组都没有使用, 第二组一定使用
```python
def change(a):
    def to(x):
        tmp = [0, 1, 5, 10, 25, 50]
        return tmp[x]
    def iter(cur, which):
        if (cur == 0):
            return 1
        if (cur < 0) or (which == 0):
            return 0
        return iter(cur, which - 1) + iter(cur - to(which), which)
    return iter(a, 5)
```
其中冗余计算和斐波那契数列中的一样, 这可以通过记忆化来实现优化

### 1.2.3 增长的阶

**增长的阶** 粗略度量某个函数所需资源的情况 \
$R(n)$ 表示计算函数在处理 $n$ 规模的问题时所需要的资源量 \
称 $R(n)$ 具有 $\Theta(f(n))$的增长阶, 其中 $R(n) = k \times f(n)$ 其中 $k$ 为常数

在线性递归计算函数中 时间需求的增长为 $\Theta(n)$, 空间需求的增长为 $\Theta(n)$ \
在线性迭代计算函数中 时间需求的增长为 $\Theta(n)$, 空间需求的增长为 $\Theta(1)$ 即常数空间

由于是粗略的描述, 所以类似 $n^2+3n+1$这样的所需步数仍然是 $\Theta(n^2)$

### 1.2.4 求幂

线性求幂 \
时间 $\Theta(n)$, 空间 $\Theta(n)$
```python
def my_pow(a, b):
    if (b == 0):
        return 1
    else:
        return a * my_pow(a, b - 1)
```
时间 $\Theta(n)$, 空间 $\Theta(1)$ (`scheme` 中空间复杂度)
```python
def my_pow(a, b):
    def iter(count, res):
        if (count == b):
            return res
        else:
            return iter(count + 1, res * a)
    return iter(0, 1)
```
快速幂: \
每个整数都可以进行二进制拆分, 基于这样的思想来进行快速幂, 假设底数为 $a$
1. 如果这个数 $b$ 是偶数, 那么计算它的 $a ^{\frac{b}{2}}$ 的平方
2. 如果这个数 $b$ 是奇数, 那么计算它的 $a \times a^{b - 1}$

总体来说问题规模每次缩减一半, 时间空间复杂度都是 $\Theta(logn)$ 的
```python
def square(x):
    return x * x
def fast_pow(a, b):
    if (b == 0):
        return 1
    elif (b % 2 == 0):
        return square(pow(a, b // 2))
    else:
        return pow(a, b - 1)
```

### 1.2.5 最大公约数

欧几里得算法 $GCD(a, b) = GCD(b, a \% b)$ \
当 $b$ 为 0的时候返回 $a$ 为最大公约数

```python
def GCD(a, b):
    if (b == 0):
        return a
    else:
        return GCD(b, a % b)
```

拉梅定理: $GCD$ 递归了 k 层, 这对数中较小的数一定大于第 k 个斐波那契数列 \
据此估计 $GCD$ 时间复杂度 $\Theta(logn)$ 空间复杂度 $\Theta(1)$ 

### 1.2.6 素数检测

**寻找因子**: 法一 $\Theta(\sqrt n)$, 法二 $\Theta(logn)$

1. 结论: 只需要寻找 $\sqrt n$ 个因子就可以找出全部因子 \
   证明: 如果 $d$ 是 $n$ 的因子, $n / d$ 也必然是 $n$ 的因子, 而两者不可能同时大于 $\sqrt n$
   ```python
    def square(x):
        return x * x
    def min_divisor(n):
        def divide(x):
            return n % x == 0
        def iter(cur):
            if (square(cur) > n):
                return n
            elif (divide(cur)):
                return cur
            else:
                return iter(cur + 1)
        return iter(2)

    def is_prime(x):
        return min_divisor(x) == x
   ```
2. 费马检查
    > 费马小定理: 如果 $n$ 是一个素数, $a$ 是小于 $n$ 的任意正整数, $a$ 的 $n$ 次方与 $a$ 模 $n$ 同余
   
    费马检查就是根据费马小定理而来的一个随机算法
    1. 对于给定的整数 $n$, 随机任取一个 $a<n$ 并计算出 $a^n$ 模 $n$ 的余数
    2. 如果得到的结果不等于 $a$, 那它就一定不是素数, 如果它是 $a$, 那么它由很大可能是素数
    3. 多取几个 $a$ 的值, 这样结果的可信度就越来越高

    > 然则这种检测方式不一定准确, 满足这个性质的非素数称作 $Carmicheal$ 数 \
    > 但在很大的素数面前, 碰到$Carmicheal$ 数的概率极低极低
    ```python
    def is_prime(p):
        def pow(a, b):
            if (b == 0):
                return 1 % p
            elif (b % 2 == 0):
                return square(pow(a, b // 2)) % p
            else:
                return a * pow(a, b - 1) % p

        def check(x):
            return pow(x, p) == x
        def calc(times):
            if (times == 0):
                return True
            elif (not check(random.randint(1, p - 1))):
                return False
            else:
                return calc(times - 1)
        return calc(5)
    ```
3. $Miller-Robin$ 素数检测
    * 根据费马小定理变形: $a^{n-1}\equiv 1(mod \; n)$
    * 根据二次探测定理: 对于素数 $p$, 如果 $x^2 \equiv 1 (mod \; p)$, 小于 $p$ 的解有两个, $1$ 和$p-1$ 
       $x^2 - 1 \equiv 0 (mod \; p)$ 即 $(x+1)(x-1)\equiv 0 (mod \; p)$ \
       因为 $p$ 为素数, 所以要么 $(x+1)(x-1) =0$ 要么 $(x+1)(x-1)$ 是 $p$ 的倍数 \
       对于第一种情况 $x=1$, 对于第二种情况 $x = p-1$ , 注意要求解小于 $p$

    具体检测步骤如下:
    1. 在计算快速幂的时候, 如果当前指数为偶数, 那么检测非平凡平方根
    3. 由二次探测定理, 设平方之前的数为 $x$, 平方之后的数为 $x^2$ 
    4. 如果 $x^2 \equiv 1 (mod \; p)$, 但是 $x \not \equiv (1或p-1) (mod \; p)$ 则 $p$ 一定不是素数
    5. 此时的 $x$ 就是书中所说的非平凡平方根

    ```python
    def miller_robin(p):
        def nontrivial_square_root(x):
            tmp = square(x) % p
            if ((x != 1) and (x != p - 1) and (tmp == 1)):
                return 0
            else:
                return tmp

        def pow(a, b):
            if (b == 0):
                return 1
            elif (b % 2 == 1):
                return a * pow(a, b - 1) % p
            else:
                return nontrivial_square_root(pow(a, b // 2))

        def test():
            return pow(random.randint(1, p - 1), p - 1) == 1

        def iter(times):
            if (times == 0):
                return True
            elif (test()):
                return iter(times - 1)
            else:
                return False

        return iter(7)
    ```

## 1.3 用高阶函数做抽象

函数的参数可以是另一个函数, 这类操作函数的函数称作高阶函数

### 1.3.1 函数作为参数

书中所说的范围内整数和, 范围内整数立方和, 计算 $\pi$ 序列三个函数都有相同的模式
```python
def sum(a, b):
    if (a > b):
        return 0
    else:
        return (<term>(a) + sum(<next>(a), b))
```
与数学中的 $\Sigma$ 类似, 这个高阶函数表达的是求和的概念
```python
def sum(term, a, to, b):
    if (a > b):
        return 0
    else:
        return (term(a) + sum(term, to(a), to, b))
```
这才是完整的高阶函数, 使用实例:
```python
import sys
sys.setrecursionlimit(10000000)

def cube(x):
    return x * x * x
def identity(x):
    return x
def inc(x):
    return x + 1
def pi_sum(a, b):
    def pi_term(x):
        return 1 / (x * (x + 2))
    def pi_to(x):
        return x + 4
    return 8 * sum(pi_term, a, pi_to, b)

print(sum(cube, 1, inc, 10))
print(sum(identity, 1, inc, 10)) 
print(pi_sum(1, 10000))
```

### 1.3.2 用 $Lambda$ 构造过程

$lambda$ 就是所谓匿名函数, 基本语法如下

```python
lambda <形参列表>: <函数体>
```

$lambda$ 的优势在于, 把函数当作参数传入的时候, 不需要给函数命名, 从而省去麻烦 \
$lambda$ 也可以当作运算符使用

```python
print((lambda x: x + 1)(5))
```

### 1.3.3 过程作为一般性的方法
1. 区间这般寻找方程的根: 主要是根据零点存在定理的二分法
    ```python
    from math import sin

    def search(func, a, b):
        mid = (a + b) / 2
        tmp = func(mid)
        if (abs(a - b) < 0.001):
            return mid
        elif (tmp < 0):
            return search(func, mid, b)
        else:
            return search(func, a, mid)

    def get_point(func, a, b):
        x, y = func(a), func(b)
        if (x < 0 and y > 0):
            return search(func, a, b)
        elif (x > 0 and y < 0):
            return search(func, b, a)
        else:
            return "Error"

    print(get_point(sin, 2.0, 4.0))
    print(get_point(lambda x: x * x * x - 2 * x - 3, 1.0, 2.0))
    ```
2. 寻找函数的不动点 \
    如果 $x$ 满足 $f(x) = x$ 则 $x$ 为函数 $f$ 的一个不动点 \
    可以对某一个随机猜测不断应用 $f$, 等到两次结果差距不大时, 就可以找到近似的不动点
    ```python
    from math import cos, sin

    tolerance = 1e-5
    def fixed_point(func, first_guess):

        def close_enough(a, b):
            return (abs(a - b) < tolerance)

        def calc(val):
            next_val = func(val)
            if (close_enough(val, next_val)):
                return next_val
            else:
                return calc(next_val)
            
        return calc(first_guess)

    print(fixed_point(cos, 1.0))
    print(fixed_point(lambda y: sin(y) + cos(y), 1.0))
    ```
    类似找平方根的过程, 令 $f(y) = \frac x y$, $f(y)=y$ 即 $x = y^2$ \
    但是直接使用上述程序会陷入死循环, 因为
    $y_2 = \frac{x}{y_1}$, $y_3 = \frac{x}{\frac{x}{y_1}} = y_1$
    解决上述问题的一个方法叫做平均阻尼, 不让有关的猜测变化得过于剧烈 \
    因为实际答案总是在 $y$ 和 $\frac x y$ 之间, 所以取平均值, 求 $f(y) = \frac{1}{2}(y + \frac x y)$ 的不动点
    ```python
    def my_sqrt(x):
        return fixed_point(lambda y: (y + x / y) / 2, 1.0)

    print(my_sqrt(11))
    ```

### 1.3.4 函数作为返回值

可以创建返回值为函数的函数

平均阻尼函数以及重做平方根过程:
```python
def average_damp(func):
    return (lambda x: (x + func(x)) / 2)
def mysqrt(x):
    return fixed_point(average_damp(lambda y: x / y), 1.0)
def cube_root(x):
    return fixed_point(average_damp(lambda y: x / (y * y)), 1.0)
```

**牛顿法**: \
如果 $g(x)$ 是一个可微分函数, 那么 $g(x)=0$ 的一个解就是 $f(x)$ 的不动点 \
其中 $f(x) = x - \frac{f(x)}{g'(x)}$,  $g'(x) = \frac{g(x+dx)-g(x)}{dx}$

```python
dx = 0.00001
def deriv(g):
    return lambda x: (g(x + dx) - g(x)) / dx
def newton_transform(g):
    return lambda x: x - g(x) / deriv(g)(x)
def newton_method(g, guess):
    return fixed_point(newton_transform(g), guess)
def sqrt(x):
    return newton_method(lambda y: y * y - x, 1.0)
```

**抽象和第一级过程**

抽象出平均阻尼和牛顿法的一般过程并重新定义 `sqrt` 函数
```python
def fixed_point_of_transform(g, transform, guess):
    return fixed_point(transform(g), guess)

def sqrt1(x):
    return fixed_point_of_transform(lambda y: x / y, average_damp, 1.0)
def sqrt2(x):
    return fixed_point_of_transform(lambda y: y * y - x,
                                    newton_transform, 1.0)
```

在程序设计语言中带有最少限制的元素被称为具有第一级的状态, 特权包括
1. 可以绑定到变量上
2. 可以提供给过程作为参数
3. 可以由过程作为结果返回
4. 可以包含在数据结构中

# 练习

## 1.1

```python
10 #10
3 + 5 + 4 # 12
9 - 1 #8
6 / 2 #3
(2 * 4) + (4 - 6) #6
a = 3
b = a + 1
a + b + (a * b) # 19
a == b # False
if ((b > a) and (b < (a * b))):
    return b
else:
    return a # 4

if (a == 4):
    return 6
elif (b == 4):
    return 6 + 7 + a
else:
    return 25 #16

2 + (b if (b > a) else a) #6

x = 0
if (a > b):
    x = a
elif (a < b):
    x = b
else: 
    x = -1
x * (a + 1) #16
```

## 1.2

```python
(5 + 4 + (2 - (3 - (6 + 4 / 5)))) / (3 * (6 - 2) * (2 - 7))
```

## 1.3

```python
def process(a, b, c):
    if (a <= b) and (a <= c):
        return b + c
    elif (b <= a) and (b <= c):
        return a + c
    elif (c <= a) and (c <= b):
        return a + b
```

## 1.4

函数行为: `a` + (`b` 的绝对值) \
如果 `b > 0` 操作符为 `+` \
如果 `b < 0` 操作符为 `-`

## 1.5

1. 正则序:
```python
# 在 (p) 这里无限展开, 发生错误
```
2. 应用序
```python
0 # 不计算 (p) 的结果, 因为谓词为真
```

## 1.6

结果是死循环, `scheme` 以应用序求值, 对于 `new-if` 这个函数 \
需要求出实参的每一个值, 所以就会陷入无限递归中, 失去了 `if` 的短路特性


## 1.7

```python
def new_good_enough(last, guess, x):
    return abs(last / guess - 1) < 0.0001
def sqrt(last, guess, x):
    if (new_good_enough(last, guess, x)):
        return guess
    else:
        return sqrt(guess, improve(guess, x), x)
```

## 1.8

```python
def new_improve(guess, x):
    return (x / square(guess) + 2 * guess) / 3.0

def calc(last, guess, x):
    if (new_good_enough(last, guess, x)):
        return guess
    else:
        return calc(guess, new_improve(guess, x), x)
```

## 1.9

第一个: 递归计算函数
```scheme
(+ 4 5)
(inc (+ 3 5))
(inc (inc (+ 2 5)))
(inc (inc (inc (+ 1 5))))
(inc (inc (inc (inc (+ 0 5)))))
(inc (inc (inc (inc 5))))
(inc (inc (inc 6)))
(inc (inc 7))
(inc 8)
9
```
第二个: 迭代计算函数
```scheme
(+ 4 5)
(+ 3 6)
(+ 2 7)
(+ 1 8)
(+ 0 9)
9
```

## 1.10

```python
A(1, 10) # 1024
A(2, 4) # 65536
A(3, 3) # 65536
```

$f(x)=2x$ \
$g(x)=2^n$ \
$h(x)=2\uparrow\uparrow n$

## 1.11

递归
```python
def calc(a, b, c)
    return 3 * a + 2 * b + c
def f(n):
    if (n < 3):
        return n
    else:
        return calc(f(n - 3), f(n - 2), f(n - 1))
```
迭代
```python
def f(n):
    def iter(a, b, c, count):
        if (count == n):
            return c
        return iter(b, c, calc(a, b, c), count + 1)
    return n if (n < 3) else iter(0, 1, 2, 2)
```

## 1.12

```python
def pascal(row, col):
    if ((row == col) or (col == 0)):
        return 1
    return pascal(row - 1, col) + pascal(row - 1, col - 1)
```

## 1.13

$$
\begin{align}
 & \gamma = \frac{1 - \sqrt 5}{2}, \phi =\frac{1 + \sqrt 5}{2} 为方程 x^2-x-1=0两根 \\\\
 & 所以 \phi^2 = \phi + 1, \gamma^2 = \gamma + 1 \\\\
 & \phi ^n = \phi^{n-2} \times\ \phi^2 =\phi^{n-2}(\phi+1)=\phi^{n-1}+\phi^{n-2}, \gamma 同理 \\\\
 & f(0) = \frac{\phi^0-\gamma^0}{\sqrt5}, f(1)=\frac{\phi^1-\gamma^1}{\sqrt5} \\\\
 & f(n)=f(n-1)+f(n-2) 和 \gamma, \phi 的递推公式相同, f(n) = \frac{\phi^n-\gamma^n}{\sqrt 5} \\\\
 & \because \lvert \frac{\gamma}{\sqrt5} \rvert < \frac 1 2 \; \therefore \lvert \frac{\gamma^n}{\sqrt5} \rvert <\frac 12 \\\\
 & f(n) = \frac{\phi^n}{\sqrt5} - \frac{\gamma^n}{\sqrt5},  得证f(n) 是距离\frac{\phi^n}{\sqrt5} 最近的整数
\end{align}
$$

## 1.14

![img](https://github.com/lzlcs/image-hosting/raw/master/QQ截图20230703174405.47ncpxb1kgc0.webp)
空间 $\Theta(n)$ \
时间 [$\Theta(n^5$)](https://sicp-solutions.net/post/sicp-solution-exercise-1-14/)

## 1.15 

1. 5次, 因为 $0.1 \times 3^5$ 才能大于 $12.15$
2. 每次问题规模缩小三分之一, 而且为递归计算函数, 空间时间都是 $\Theta(logn)$


## 1.16

二进制分割, $2^11 = 2^1 \times 2^2 \times 2^8$  \
`cur` 保存 2 的几次幂, 如果 `count` 是奇数就乘到答案里

```python
def fast_pow(a, b):
    def iter(res, cur, count):
        if (count == 0):
            return res
        elif (count % 2 == 0):
            return iter(res, cur * cur, count / 2)
        else:
            return iter(res * cur, cur, count - 1)
    return iter(1, a, b)
```

## 1.17

```python
def double(x):
    return x + x
def halve(x):
    return x / 2
def mul(a, b):
    if (b == 0):
        return 0
    elif (b % 2 == 0):
        return mul(double(a), halve(b))
    else:
        return a + mul(a, b - 1)
```

## 1.18

```python
def mul(a, b):
    def iter(res, cur, count):
        if (count == 0):
            return res
        elif (count % 2 == 0):
            return iter(res, double(cur), halve(count))
        else:
            return iter(res + cur, cur, count - 1)
    return iter(0, a, b)
```

## 1.19

基于矩阵乘法的矩阵快速幂求斐波那契数列

$$
\begin{align}
& a \rightarrow (bp + a(p + q)) \rightarrow b(2pq+p^2) + a(p^2+q^2+2pq+p^2) \\\\
& b \rightarrow bp + aq \rightarrow b(p^2+q^2) + a(2pq+q^2)
\end{align}
$$
所以 $p' = p^2+q^2$, $q'= 2pq+q^2$
```python
def fib(n):
    def iter(a, b, p, q, count):
        if (count == 0):
            return b
        elif (count % 2 == 0):
            return iter(a, b, (p * p + q * q), (2 * p * q + q * q), count / 2)
        else:
            return iter(b * q + a * q + a * p, 
                        b * p + a * q, p, q, count - 1)
    return iter(1, 0, 0, 1, n)
```

## 1.20 

正则序展开过长, 省略: 共18次 `remainder` \
应用序展开: 共4次 `remainder`
```scheme
(GCD 206 40)
(GCD 40 6)
(GDD 6 4)
(GCD 4 2)
(GCD 2 0)
```

## 1.21

```scheme
199
1999
7
```

## 1.22

```python
import time

x = 13 
l, r = fast_pow(10, x), fast_pow(10, x + 1)

starttime = time.time() 

for i in range(l, r):
    if (is_prime_simple(i)):
        print(i)
        break

endtime = time.time()
print("time: ", (endtime - starttime))
```
在 $10^{11} 到 10^{12}$ 寻找
```python
100000000003
time:  0.1265561580657959
```
在 $10^{12} 到 10^{13}$ 寻找
```python
1000000000039
time:  0.41858363151550293
```
在 $10^{13} 到 10^{14}$ 寻找
```python
10000000000037
time:  1.2202045917510986
```

总体上符合 $\sqrt 10$的规律

## 1.23

改 `next` 函数之后:
```python
def next(n):
    if (n == 2):
        return 3
    return n + 2
```
```python
100000000003
time:  0.07922077178955078
1000000000039
time:  0.26198792457580566
10000000000037
time:  0.6970314979553223
```

## 1.24

使用新的素数检测之后, 检测 100 个素数的平均时间:

$10^{100}$
time:  0.00022767770169961331 \
$10^{200}$
time:  0.0010467009110884233 \
$10^{300}$
time:  0.002386471237799134 \
$10^{400}$
time:  0.005082491672400272 \
$10^{500}$
time:  0.00900224724201241 \
$10^{600}$
time:  0.014897625855725221 \
$10^{700}$
time:  0.023792396892200817 

不符合预期的结果可能因为系统内部自身调度的原因

## 1.25

不能, 在最后一起取模的话, 函数内部计算数值过大, 时间复杂度过高

## 1.26 

他使用这种是树形递归, 每层节点数是上一层的二倍

一共有 $logn$ 层, 节点数 1 2 4 ..., 根据等比数列求和公式可得复杂度 $\Theta(n)$

## 1.27

```python
for x in range(2, int(1e6)):
    if (is_prime_simple(x) != is_prime(x)):
        print(x)
```

10万之内的 $Carmicheal$ 数
```
561
1105
1729
2465
2821
6601
8911
10585
15841
29341
41041
46657
52633
62745
63973
75361
```

## 1.28

$Miller-Robin$ 放在正文了

## 1.29


$Simpson$
```python
def simpson(f, a, b, n):
    h = (b - a) / n
    def k(x):
        if (x == 0) or (x == n):
            return 1
        elif (x % 2 == 0):
            return 2
        else:
            return 4

    def y(x):
        return f(a + x * h)
    def term(x):
        return k(x) * y(x)
    def inc(x):
        return x + 1

    return h * sum(term, 0.0, inc, n) / 3.0

print(simpson(cube, 0.0, 1.0, 100))
print(simpson(cube, 0.0, 1.0, 1000))
```
```
0.24999999999999992
0.2500000000000003
```
普通积分
```
0.249999875000001
0.24999999874993412
```

## 1.30

```python
def sum(term, a, to, b):
    def iter(count, res):
        if (count > b):
            return res
        else:
            return iter(to(count), res + term(count))
    return iter(a, 0)
```

## 1.31
```python
def product(f, a, to, b):
    if (a > b):
        return 1
    else:
        return f(a) * product(f, to(a), to, b)

def product_new(f, a, to, b):
    def iter(count, res):
        if (count > b):
            return res
        else:
            return iter(to(count), res * f(count))
    return iter(a, 1)

def identity(x):
    return x
def inc(x):
    return x + 1
def factorial(n):
    return product(identity, 1, inc, n)

def get_pi(n):
    def pi_term(f, g, x):
        if (x % 2 == f):
            return x + 2
        if (x % 2 == g):
            return x + 1
    def upper_term(x):
        return pi_term(0, 1, x)
    def lower_term(x):
        return pi_term(1, 0, x)

    return 4 * product_new(upper_term, 1, inc, n) / product_new(lower_term, 1, inc, n)

print(factorial(6))
print(get_pi(10000))
```

## 1.32

```python
def add(a, b):
    return a + b
def mul(a, b):
    return a * b

def accumulate(opt, init, term, a, to, b):
    if (a > b):
        return init
    else:
        return opt(term(a), accumulate(opt, init, term, to(a), to, b))

def accumulate_new(opt, init, term, a, to, b):
    def iter(count, res):
        if (count > b):
            return res
        else:
            return iter(to(count), opt(res, term(count)))
    return iter(init)

print(accumulate(add, 0, cube, 1, inc, 10))
print(accumulate(mul, 1, identity, 1, inc, 5))
```

## 1.33

```python
def filter_accumulate(opt, init, term, filt, a, to, b):
    if (a > b):
        return init
    tmp = filter_accumulate(opt, init, term, filt, to(a), to, b)
    if (filt(a)):
        return opt(a, tmp)
    return tmp

def q_a(a, b):
    return filter_accumulate(add, 0, identity, miller_robin, a, inc, b)

def GCD(a, b):
    if (b == 0):
        return a
    else:
        return GCD(b, a % b)
def q_b(n):
    def filt(x):
        return GCD(x, n) == 1
    return filter_accumulate(mul, 1, identity, filt, 1, inc, n - 1)
```

## 1.34

发生错误 \
展开之后是 `(2 2)` 

## 1.35

$\phi$ 是方程 $x^2 - x - 1 =0$ 的根, 变形一下可得 $f(x) = 1 + \frac 1 x$ 的不动点是 $\phi$
```python
print(fixed_point(lambda x: 1 + 1 / x, 1.0))
```

## 1.36

```python
def damp():
    print(fixed_point(lambda x: (log(1000) / log(x) + x) / 2, 2.0))

print(fixed_point(lambda x: log(1000) / log(x), 2.0)) ;34 步
print(damp())                                         ;9 步
```

## 1.37

至少要 11 次才能到 $0.618$ 的精度 \
递归
```python
def cont_frac(n, d, k):
    def calc(count):
        if (count == k):
            return n(k) / d(k)
        else:
            return n(count) / (d(count) + calc(count + 1))
    return calc(1)

print(cont_frac(lambda x: 1.0, lambda x: 1.0, 11))
```
迭代
```
def cont_frac(n, d, k):
    def calc(count, res):
        if (count == 0):
            return res
        else:
            return calc(count - 1, n(count) / (d(count) + res))
    return calc(k, 0)
```

## 1.38

```python
print(2 + cont_frac(lambda x: 1.0, 
                    lambda x: 2 * (x + 1) / 3 if (x % 3 == 2) else 1.0, 100))
```

## 1.39
```python
def tan_cf (x, k):
    return cont_frac(lambda b: x if (b == 1) else -x * x,
                     lambda b: 2 * b - 1, k)
```

## 1.40

```python
def cubic(a, b, c):
    return lambda x: x * x * x + a * x * x + b * x + c

print(newton_method(cubic(2, 5, 5), 1.0))
```

## 1.41

```python
def double(func):
    return lambda x: func(func(x))
def inc(x):
    return x + 1

print(double(double(double(inc)))(5)) ;13
```

## 1.42

```python
def inc(x):
    return x + 1
def square(x):
    return x * x
def compose(f, g):
    return lambda x: f(g(x))

print(compose(square, inc)(6))
```

## 1.43

```python
def square(x):
    return x * x

def compose(f, g):
    return lambda x: f(g(x))

def repeat(f, n):
    if (n == 1):
        return f
    else:
        return compose(f, repeat(f, n - 1))

print(repeat(square, 2)(5))
```

## 1.44

```python
def smooth(f):
    return lambda x: (f(x) + f(x - dx) + f(x + dx)) / 3

def smooth_n(f, n):
    return repeat(smooth, n)(f)

print(smooth(square)(5))
print(smooth_n(square, 10)(5))
```

## 1.45

```python
import math
def n_root(x, n):
    return fixed_point_of_transform(lambda y: x / pow(y, n - 1),
                                    repeat(average_damp, int(math.log2(n))), 1.0)
```
由实验可知, 平均的阻尼次数是 $\lfloor log_2^n \rfloor$

## 1.46

```python
def interative_improve(good_enough, improve):
    def calc(guess):
        new_guess = improve(guess)
        if (good_enough(guess, new_guess)):
            return guess
        else:
            return calc(new_guess)
    return calc

def sqrt(x):
    return interative_improve(lambda a, b: abs(a - b) < tolerance,
                              average_damp(lambda y: x / y))(1.0)

def fixed_point(func, first_guess):
    return interative_improve(lambda a, b: abs(a - b) < tolerance,
                              func)(first_guess)

print(sqrt(120))
print(fixed_point(cos, 1.0))
```
