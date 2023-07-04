---
title: SICP 学习笔记 (第一章) 
date: 2023-07-02 15:04:05
mathjax: true
tags:
- Scheme
- SICP
categories: 
- Book
---

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
```scheme
(define (change a)
  (define (to x)
    (cond ((= x 1) 1)
          ((= x 2) 5)
          ((= x 3) 10)
          ((= x 4) 25)
          ((= x 5) 50)))

  (define (change-iter cur type)
    (cond ((= cur 0) 1)
          ((< cur 0) 0)
          ((= type 0) 0)
          (else (+ (change-iter cur (- type 1))
                   (change-iter (- cur (to type)) type)))))

  (change-iter a 5))
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
```scheme
(define (pow a b)
  (cond ((= b 0) 1)
        (else (* a (pow a (- b 1))))))
```
时间 $\Theta(n)$, 空间 $\Theta(1)$
```scheme
(define (pow a b)
  (define (pow-iter cur count)
    (cond ((= count b) cur)
          (else (pow-iter (* cur a) (+ count 1)))))
  (pow-iter 1 0))
```
快速幂: \
每个整数都可以进行二进制拆分, 基于这样的思想来进行快速幂, 假设底数为 $a$
1. 如果这个数 $b$ 是偶数, 那么计算它的 $a ^{\frac{b}{2}}$ 的平方
2. 如果这个数 $b$ 是奇数, 那么计算它的 $a \times a^{b - 1}$

总体来说问题规模每次缩减一半, 时间空间复杂度都是 $\Theta(logn)$ 的
```scheme
(define (fast-pow a b)
  (cond ((zero? b) 1)
        ((even? b) (square (fast-pow a (/ b 2))))
        (else (* a (fast-pow a (- b 1))))))
```

### 1.2.5 最大公约数

欧几里得算法 $GCD(a, b) = GCD(b, a \% b)$ \
当 $b$ 为 0的时候返回 $a$ 为最大公约数

```scheme
(define (GCD a b)
  (cond ((zero? b) a)
        (else (GCD b (remainder a b)))))
```

拉梅定理: $GCD$ 递归了 k 层, 这对数中较小的数一定大于第 k 个斐波那契数列 \
据此估计 $GCD$ 时间复杂度 $\Theta(logn)$ 空间复杂度 $\Theta(1)$ 

### 1.2.6 素数检测

**寻找因子**: 法一 $\Theta(\sqrt n)$, 法二 $\Theta(logn)$

1. 结论: 只需要寻找 $\sqrt n$ 个因子就可以找出全部因子 \
   证明: 如果 $d$ 是 $n$ 的因子, $n / d$ 也必然是 $n$ 的因子, 而两者不可能同时大于 $\sqrt n$
   ```scheme
   (define (min-divisor n)
     (define (divides? x) (= (remainder n x) 0))
     (define (iter cur)
         (cond ((> (square cur) n) n)
               ((divides? cur) cur)
               (else (iter (+ cur 1)))))
     (iter 2))

   (define (prime? x)
     (= (min-divisor x) x))
   ```
2. 费马检查
    > 费马小定理: 如果 $n$ 是一个素数, $a$ 是小于 $n$ 的任意正整数, $a$ 的 $n$ 次方与 $a$ 模 $n$ 同余
   
    费马检查就是根据费马小定理而来的一个随机算法
    1. 对于给定的整数 $n$, 随机任取一个 $a<n$ 并计算出 $a^n$ 模 $n$ 的余数
    2. 如果得到的结果不等于 $a$, 那它就一定不是素数, 如果它是 $a$, 那么它由很大可能是素数
    3. 多取几个 $a$ 的值, 这样结果的可信度就越来越高

    > 然则这种检测方式不一定准确, 满足这个性质的非素数称作 $Carmicheal$ 数 \
    > 但在很大的素数面前, 碰到$Carmicheal$ 数的概率极低极低
    ```scheme
    (define (prime? n)
      (define (pow a b m)
        (cond ((= b 0) 1)
              ((odd? b) (remainder (* a (pow a (- b 1) m)) m))
              (else (remainder (square (pow a (/ b 2) m)) m))))
      (define (calc times)
        (define (test a)
          (= (pow a n n) a))
        (cond ((= times 0) #t)
              ((not (test (+ 1 (random (- n 1))))) #f)
              (else (calc (- times 1)))))
      (calc 5))
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

    ```scheme
    (define (prime? p)
      (define (nontrivial-square-root x)
        (if (and (not (= x 1))
                 (not (= x (- p 1)))
                 (= 1 (remainder (square x) p)))
            0
            (remainder (square x) p)))
    
      (define (pow a b)
        (cond ((zero? b) 1)
              ((even? b) (nontrivial-square-root (pow a (/ b 2))))
              (else (remainder (* a (pow a (- b 1))) p))))
    
      (define (test?)
        (= 1 (pow (+ 1 (random (- p 1))) 
                  (- p 1))))
    
      (define (iter times)
        (cond ((= times 0) #t)
              ((test?) (iter (- times 1)))
              (else #f)))
      (iter 7))
    ```

## 1.3 用高阶函数做抽象

函数的参数可以是另一个函数, 这类操作函数的函数称作高阶函数

### 1.3.1 函数作为参数

书中所说的范围内整数和, 范围内整数立方和, 计算 $\pi$ 序列三个函数都有相同的模式
```scheme
(define (sum a b)
  (if (> a b)
      0
      (+ (term a)
         (sum (next a) b))))
```
与数学中的 $\Sigma$ 类似, 这个高阶函数表达的是求和的概念
```scheme
(define (sum term next a b)
  (if (> a b)
      0
      (+ (term a)
         (sum term next (next a) b))))
```
这才是完整的高阶函数, 使用实例:
```scheme
(define (cube x) (* x x x))
(define (identity x) x)
(define (inc x) (+ x 1))
(define (pi-sum a b)
  (define (pi-term x) (/ 1.0 (* x (+ x 2))))
  (define (pi-next x) (+ x 4))
  (* 8 (sum pi-term pi-next a b)))

(display (sum cube inc 1 10))
(newline)
(display (sum identity inc 1 10))
(newline)
(display (pi-sum 1 1000))
```

### 1.3.2 用 $Lambda$ 构造过程

$lambda$ 就是所谓匿名函数, 基本语法如下

```scheme
(lambda (<formal-parameters>) (<body>))
```
其实
```scheme
(define (inc x) (+ x 1))
(define inc (lambda (x) (+ 1 x)))
```
两者作用相同, 第一行只是第二行的简化版本 \
$lambda$ 的优势在于, 把函数当作参数传入的时候, 不需要给函数命名, 从而省去麻烦
$lambda$ 也可以当作运算符使用
```scheme
((lambda (x) (* 2 x)) 5)
```

**$let$**可以创建局部变量, 一般形式如下:
```scheme
(let ((<var1> <exp1>)
      (<var1> <exp1>)
      (<var1> <exp1>)
      ...)
  <body>)
```
和以下形式等价
```scheme
((lambda (var1 var2 var3)
    (body))
  (exp1 exp2 exp3))
```

* $let$ 中的变量只能在后面的 $body$ 中使用, 相当于建立局部变量约束
* $let$ 中的变量是由外部的变量计算而来, 当形参重名的时候要注意这点
* 当然可以用内部 $define$ 来创建变量, 但约定俗成的是 $define$ 定义过程, $let$ 定义变量


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

## 1

### 1.1

```scheme
10 ;10
(+ 3 5 4) ;12
(- 9 1) ;8
(/ 6 2) ;3
(+ (* 2 4) (- 4 6)) ;6
(define a 3) ;a
(define b (+ a 1)) ;b
(+ a b (* a b)) ;19
(= a b) ;#f
(if (and (> b a) (< b (* a b)))
    b
    a) ;4
(cond ((= a 4) 6)
      ((= b 4) (+ 6 7 a))
      (else 25) ;16
(+ 2 (if (> b a) b a)) ;6
(* (cond ((> a b) a)
         ((< a b) b)
         (else -1))
   (+ a 1)) ;16
```

### 1.2

```scheme
(/ (+ 5 4 (- 2 (- 3 (+ 6 (/ 4 5)))))
   (* 3 (- 6 2) (- 2 7)))
```

### 1.3

```scheme
(define (process a b c)
    (cond ((and (<= a b) (<= a c)) (+ b c))
          ((and (<= b a) (<= b c)) (+ a c))
          ((and (<= c a) (<= c b)) (+ a b))))
```

### 1.4

函数行为: `a` + (`b` 的绝对值) \
如果 `b > 0` 操作符为 `+` \
如果 `b < 0` 操作符为 `-`

### 1.5

1. 正则序:
```scheme
(test 0 (p))
(if (= x 0) 0 (p)) ; 在 (p) 这里无限展开, 发生错误
```
2. 应用序
```scheme
0 ; 不计算 (p) 的结果, 因为谓词为真
```

### 1.6

结果是死循环, `scheme` 以应用序求值, 对于 `new-if` 这个函数 \
需要求出实参的每一个值, 所以就会陷入无限递归中, 失去了 `if` 的短路特性


### 1.7

```scheme
(define (new-good-enough? last guess x)
  (< (abs (- (/ last guess) 1)) 0.001)) 

(define (sqrt last guess x)
    (cond ((new-good-enough? last guess x) guess)
          (else (sqrt guess (improve guess x) x))))
```

### 1.8

```scheme
(define (new-improve guess x)
  (/ (+ (/ x (* guess guess)) (* 2 guess)) 3.0))

(define (calc last guess x)
    (cond ((new-good-enough? last guess x) guess)
          (else (calc guess (new-improve guess x) x))))
```

### 1.9

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

### 1.10

```scheme
(A 1 10) ;1024
(A 2 4)  ;65536
(A 3 3)  ;65536
```

$f(x)=2x$ \
$g(x)=2^n$ \
$h(x)=2\uparrow\uparrow n$

### 1.11

递归
```scheme
(define (calc a b c)
  (+ (* 3 a) (* 2 b) c))

(define (f n)
  (cond ((< n 3) n)
        (else (calc (f (- n 3)) (f (- n 2)) (f (- n 1))))))
```
迭代
```scheme
(define (f n)
  (define (calc a b c)
    (+ (* 3 a) (* 2 b) c))
  (define (f-iter a b c counter)
    (cond ((= counter n) c)
          (else (f-iter b c (calc a b c) (+ counter 1)))))
  (if (< n 3) n (f-iter 0 1 2 2)))
```

### 1.12

```scheme
(define (pascal row col)
  (cond ((or (= row col) (= col 0)) 1)
        (else (+ (pascal (- row 1) col)
                 (pascal (- row 1) (- col 1))))))
```

### 1.13

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

### 1.14

![img](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/QQ截图20230703174405.47ncpxb1kgc0.webp)
空间 $\Theta(n)$ \
时间 [$\Theta(n^5$)](https://sicp-solutions.net/post/sicp-solution-exercise-1-14/)

### 1.15 

1. 5次, 因为 $0.1 \times 3^5$ 才能大于 $12.15$
2. 每次问题规模缩小三分之一, 而且为递归计算函数, 空间时间都是 $\Theta(logn)$


### 1.16

二进制分割, $2^11 = 2^1 \times 2^2 \times 2^8$  \
`cur` 保存 2 的几次幂, 如果 `count` 是奇数就乘到答案里

```scheme
(define (fast-pow a b)
  (define (pow-iter res cur count)
    (cond ((zero? count) res)
          ((even? count) (pow-iter res (square cur) (/ count 2)))
          (else (pow-iter (* res cur) cur (- count 1)))))
  (pow-iter 1 a b))
```

### 1.17

```scheme
(define (double x) (+ x x))
(define (halve x) (/ x 2))
(define (mul a b)
  (cond ((zero? b) 0)
        ((even? b) (mul (double a) (halve b)))
        (else (+ a (mul a (- b 1))))))
```

### 1.18

```scheme
(define (mul a b)
  (define (iter res cur count)
    (cond ((zero? count) res)
          ((even? count) (iter res (double cur) (halve count)))
          (else (iter (+ res cur) cur (- count 1)))))
  (iter 0 a b))
```

### 1.19

基于矩阵乘法的矩阵快速幂求斐波那契数列

$$
\begin{align}
& a \rightarrow (bp + a(p + q)) \rightarrow b(2pq+p^2) + a(p^2+q^2+2pq+p^2) \\\\
& b \rightarrow bp + aq \rightarrow b(p^2+q^2) + a(2pq+q^2)
\end{align}
$$
所以 $p' = p^2+q^2$, $q'= 2pq+q^2$
```scheme
(define (fib n)
  (define (fib-iter a b p q counter)
    (cond ((= counter 0) b)
          ((even? counter)
            (fib-iter a 
                      b
                      (+ (square p) (square q))
                      (+ (* 2 p q) (square q))
                      (/ counter 2)))
          (else
            (fib-iter (+ (* b q) (* a q) (* a p))
                      (+ (* b p) (* a q))
                      p
                      q
                      (- counter 1)))))
  (fib-iter 1 0 0 1 n))
```

### 1.20 

正则序展开过长, 省略: 共18次 `remainder` \
应用序展开: 共四次 `remainder`
```scheme
(GCD 206 40)
(GCD 40 6)
(GDD 6 4)
(GCD 4 2)
(GCD 2 0)
```

### 1.21

```scheme
199
1999
7
```

### 1.22

这里使用 `scheme` 记录时间
```scheme
(define (timed-prime-test n)
  (newline)
  (display n)
  (start-prime-test n (runtime)))

(define (start-prime-test n start-time)
  (if (prime? n)
      (report-prime (- (runtime) start-time))))

(define (report-prime elapsed-time)
  (display " *** ")
  (display elapsed-time))

(define (search-for-primes l r)
  (cond ((= r l) 'exit)
        ((prime? l) (timed-prime-test l))
        (else (search-for-primes (+ l 1) r))))
```
在 $10^{13} 到 10^{14}$ 寻找
```scheme
10000000000037
 *** 1.2599999999999998
```
在 $10^{14} 到 10^{15}$ 寻找
```scheme
100000000000031
 *** 4.06
```
在 $10^{15} 到 10^{16}$ 寻找
```scheme
1000000000000037
 *** 12.66
```

总体上符合 $\sqrt 10$的规律

### 1.23

改 `next` 函数之后:
```scheme
(define (next x)
  (cond ((= x 2) 3)
```
在 $10^{13} 到 10^{14}$ 寻找
```scheme
10000000000037
 *** .8400000000000001
```
在 $10^{14} 到 10^{15}$ 寻找
```scheme
100000000000031
  *** 2.6
```
在 $10^{15} 到 10^{16}$ 寻找
```scheme
1000000000000037
 *** 8.220000000000002
```

近似于 1.5 的倍率, 原因可能和系统调用, 资源占用等方面有关

### 1.24

使用新的素数检测之后, 检测 \
$10^{400}$ 的素数是 0.03 秒左右
$10^{500}$ 的素数是 0.04 秒左右
$10^{1000}4 的素数是 0.4 秒左右

不符合预期的结果可能因为系统内部自身调度的原因

### 1.25

不能, 在最后一起取模的话, 函数内部计算数值过大, 时间复杂度过高

### 1.26 

他使用这种是树形递归, 每层节点数是上一层的二倍

一共有 $logn$ 层, 节点数 1 2 4 ..., 根据等比数列求和公式可得复杂度 $\Theta(n)$

### 1.27

```scheme
(define (check n)
  (define (pow a b m)
    (cond ((= b 0) 1)
          ((odd? b) (remainder (* a (pow a (- b 1) m)) m))
          (else (remainder (square (pow a (/ b 2) m)) m))))
  (define (calc times)
    (define (test a)
      (= (pow a n n) a))
    (cond ((= times 2) #t)
          ((not (test (- times 1))) #f)
          (else (calc (- times 1)))))
  (calc n))

(define (find-carmicheal cur n)
  (cond ((= n cur) 'exit)
        ((and (not (prime? cur)) (check cur)) 
         (begin (newline)
                (display cur)
                (find-carmicheal (+ cur 1) n)))
        (else (find-carmicheal (+ cur 1) n))))

(find-carmicheal 2 1000000)
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

### 1.28

$Miller-Robin$ 放在正文了

### 1.29

```scheme
(define (sum term next a b)
  (if (> a b)
      0
      (+ (term a)
         (sum term next (next a) b))))

(define (simpson f a b n)
  (define h (/ (- b a) n))

  (define (k x)
    (cond ((or (= 0 x) (= n x)) 1)
          ((even? x) 2)
          ((odd? x) 4)))

  (define (y x) (f (+ a (* x h))))
  (define (term x) (* (k x) (y x)))
  (define (inc x) (+ x 1))

  (/ (* h (sum term inc 0.0 n)) 3.0))

(define (cube x) (* x x x))

(newline)
(display (simpson cube 0.0 1.0 100))
(newline)
(display (simpson cube 0.0 1.0 1000))
(newline)
```

### 1.30

```scheme
(define (sum term a next b)
  (define (iter a result)
    (if (> a b)
        result
        (iter (next a) (+ result (term a)))))
  (iter a 0))
```

### 1.31
```scheme
(define (product f next a b)
  (cond ((> a b) 1)
        (else (* (f a) (product f next (next a) b)))))

(define (product-new f next a b)
  (define (iter count res)
    (cond ((> count b) res)
          (else (iter (next count) (* res (f count))))))
  (iter a 1))

(define (identify x) x)
(define (inc x) (+ x 1))

(define (factorial n)
  (product identify inc 1 n))

(define (get-pi n)
  (define (pi-term f? g? x) 
    (cond ((f? x) (+ x 2))
          ((g? x) (+ x 1))))
  (define (upper-term x) (pi-term even? odd? x))
  (define (lower-term x) (pi-term odd? even? x))

  (* 4.0 (/ (product-new upper-term inc 1 n)
            (product-new lower-term inc 1 n))))

(newline)
(display (factorial 6)) ;
(newline)
(display (get-pi 10000))
```

### 1.32

递归
```scheme
(define (accumulate opt init term next a b)
  (cond ((> a b) init)
        (else (opt (term a) 
                   (accumulate opt init term next (next a) b)))))
```
迭代
```scheme
(define (accumulate opt init term next a b)
  (define (iter count res)
    (cond ((> count b) res)
          (else (iter (next count) (opt res
                                        (term count))))))
  (iter a init))

```
### 1.33

```scheme
(define (filter-accumulate opt init term next filter a b)
  (cond ((> a b) init)
        ((filter a) (opt (term a)
                         (filter-accumulate opt init term next filter 
                                            (next a) b)))
        (else (filter-accumulate opt init term next filter (next a) b))))

(define (identify x) x)
(define (inc x) (+ x 1))

(define (question_a a b)
  (filter-accumulate + 0 identify inc prime? a b))

(define (question_b n)
  (define (filter x) (= 1 (GCD x n)))
  (filter-accumulate * 1 identify inc filter 1 (- n 1)))
```

### 1.34

发生错误 \
展开之后是 `(2 2)` 

### 1.35

$\phi$ 是方程 $x^2 - x - 1 =0$ 的根, 变形一下可得 $f(x) = 1 + \frac 1 x$ 的不动点是 $\phi$
```python
print(fixed_point(lambda x: 1 + 1 / x, 1.0))
```

### 1.36

```python
def damp():
    print(fixed_point(lambda x: (log(1000) / log(x) + x) / 2, 2.0))

print(fixed_point(lambda x: log(1000) / log(x), 2.0)) ;34 步
print(damp())                                         ;9 步
```

### 1.37

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

### 1.38

```python
print(2 + cont_frac(lambda x: 1.0, 
                    lambda x: 2 * (x + 1) / 3 if (x % 3 == 2) else 1.0, 100))
```

### 1.39
```python
def tan_cf (x, k):
    return cont_frac(lambda b: x if (b == 1) else -x * x,
                     lambda b: 2 * b - 1, k)
```

### 1.40

```python
def cubic(a, b, c):
    return lambda x: x * x * x + a * x * x + b * x + c

print(newton_method(cubic(2, 5, 5), 1.0))
```

### 1.41

```python
def double(func):
    return lambda x: func(func(x))
def inc(x):
    return x + 1

print(double(double(double(inc)))(5)) ;13
```

### 1.42

```python
def inc(x):
    return x + 1
def square(x):
    return x * x
def compose(f, g):
    return lambda x: f(g(x))

print(compose(square, inc)(6))
```

### 1.43

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

### 1.44

```python
def smooth(f):
    return lambda x: (f(x) + f(x - dx) + f(x + dx)) / 3

def smooth_n(f, n):
    return repeat(smooth, n)(f)

print(smooth(square)(5))
print(smooth_n(square, 10)(5))
```

### 1.45

```python
import math
def n_root(x, n):
    return fixed_point_of_transform(lambda y: x / pow(y, n - 1),
                                    repeat(average_damp, int(math.log2(n))), 1.0)
```
由实验可知, 平均的阻尼次数是 $\lfloor log_2^n \rfloor$

### 1.46

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
