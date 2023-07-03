---
title: SICP 学习笔记 (第一章) 
date: 2023-07-02 15:04:05
tags:
- Scheme
- SICP
categories: 
- Book
---

# Chapter 1: 构造过程抽象

## 1.1 程序设计的基本元素

三种机制
1. 基本表达形式: 最简单的元素
2. 组合的方法: 构造复合元素
3. 抽象的方法: 命名并操作复合元素

### 1.1.1 表达式

一些十进制的数字是表达式, 可以用表示过程的基本元素(如 `+`, `*`) 来组合这些数字表达式 \
表示把有关过程应用到这些数字上面, 形成类似 `(+ 3 4)` 的组合式 \
`(运算符 运算对象1 运算对象2 ....)` 这种运算符在最前的表达方式是前缀表达式
1. 可以适用任意个运算对象而不出现歧义 
2. 允许运算对象本身也是个组合式

解释器通常按照 **读入-求值-打印** 循环求出表达式的值, 无论表达式多复杂

### 1.1.2 命名和环境

```scheme
(define x 2) ;x
```
在 `scheme` 中使用 `define` 来绑定名称和值, `define` 是最简单的抽象方法 \
忽略计算值的一系列过程而直接使用名称从而简化程序

要实现这种绑定的功能, 解释器必须具备存储的能力 \
这种存储称之为环境(全局环境)

### 1.1.3 组合式的求值

1. 对每个运算对象求值, 这一求值过程是递归的
2. 对每个运算对象求值的结果应用运算符

可以采用树的形式来表示组合式的求值过程, 组合式的值为父节点, 子节点为运算符和运算对象 \
自底向上计算, 在越来越高的层次中组合起来, 这种计算过程称为 **树形积累**

叶子节点可以分为以下几类:
1. 数字, 代表它本身
2. 运算符, 代表操作方式
3. 其他: 在环境中寻找绑定的值
4. 特殊形式, 到此为止唯一的特殊形式是 `define`, 它需要自身额外的求值规则

`Lisp` 对各种表达式求值就是在应用一般规则的同时注意为数不多的特殊规则

### 1.1.4 复合过程

**过程定义** 可以为复合操作提供名字, 从而在之后的运算中作为一个单元来使用

```scheme
(define (square x) (* x x))
(define (过程名称 形参列表) (过程主体))
```
过程名称和形参列表写在一起, 和调用这个过程的时候一样
```scheme
(square 4) ;16
```
此时实际参数 4 取代形式参数 `x` 进行运算, 结果为 16 \
`(square 4)` 也称为一个表达式, 可以被嵌入其他表达式中

```scheme
(define (sum_square x y)
  (+ (square x) (square y)))
(define (f x)
  (sum_square (+ x 1) (* x 2)))
```

### 1.1.5 过程应用的代换模型

**代换模型** 就是将过程体的形参用实参代替之后, 对过程体求值的计算过程
> 替换只是一个最简单的模型, 之后会讨论其他更精细的模型

* 正则序求值: 先把所有过程都展开, 最后统一求值
```scheme
; x = 5
(+ (* (+ 5 1) (+ 5 1)) (* (* 5 2) (* 5 2)))
```
* 应用序求值: 先求值所有参数, 之后再展开过程
```scheme
(sum_square 6 10)
(+ (* 6 6) (* 10 10))
(+ 36 100)
136
```
1. `Lisp`采用应用序求值, 这样可以避免一些重复计算
1. 在无法使用简单的替换来模拟的过程中, 正则序更加复杂
1. 但在某些方面, 正则序也可以成为重要工具, 如"无限"数据结构

### 1.1.6 条件表达式和谓词

`cond` 关键字针对这种 **分情况分析**
```scheme
(define (abs x)
  (cond ((> x 0) x)
        ((< x 0) (- x))
        ((= x 0) 0)))
(define (abs x)
  (cond ((< x 0) (- x))
        (else x)))
(define (abs x)
  (if (< x 0)
      (- x)
      x))
```
下为 `cond` 特殊形式的一般结构
```scheme
(cond ((p1) (e1))
      ((p2) (e2))
      (......)
      (else (en)))
```
这些 `p1`, `p2` 等被称作**谓词**, 是一个结果为 `#t`(真) 或 `#f`(假) 的表达式\
其中 `else` 的值永远为 `#t` \
流程就是找到第一个结果为真的谓词, 之后执行其后的表达式

如果在没有 `else` 的情况下, 所有的谓词都是 `#f`, 那么 `cond` 的值就未定义

----
下为 `if` 特殊形式的一般结构
```scheme
(if (predicate)
    (expression_true)
    (expression_false))
```
如果谓词为真则执行第一个表达式, 否则执行第二个表达式

基本谓词运算符: `>`, `<`, `=`, `<=`, `>=` 等 \
复合谓词运算符: `and`, `or`, `not`
```scheme
(and (e1) (e2) ... (en)) ; 对所有表达式做 and 操作
(or (e1) (e2) ... (en)) ; 对所有表达式做 or 操作
(not (expression)) ; 对这个表达式取反
```
`and` 和 `or` 都有短路特性, 具体表现为:
1. 当 `and` 后某个表达式为 `#f` 时, 后面的表达式不再求值
2. 当 `or` 后某个表达式为 `#t` 时, 后面的表达式不再求值

### 1.1.7 实例: 牛顿法求平方根

定义平方根函数 $\sqrt x = y, 使得y>=0且 y^2 =x$ \
1. 说明性描述: 注重是什么
2. 行动性描述: 注重怎么办

牛顿法求平方根过程:
对 $\sqrt x$ 的值有一个猜测 $y$, 求出 $y$ 和 $\frac{x}{y}$的平均数作为新的猜测

```scheme
(define (good-enough? guess x)
    (< (abs (- x (square guess))) 0.001))
(define (average x y) (/ (+ x y) 2))
(define (improve guess x)
    (average guess (/ x guess)))

(define (sqrt guess x)
    (cond ((good-enough? guess x) guess)
          (else (sqrt (improve guess x) x))))

(display (sqrt 1.0 2))
```
> `good-enough?` 后面的问号只是对返回谓词的过程的命名习惯 \
> `1.0` 在 `MIT Scheme` 解释器中是有分数表述的 \
> 所以为了让其以浮点形式计算, 初始化 `guess` 为 `1.0`

### 1.1.8 过程作为黑箱抽象

**过程抽象** 封装一个过程, 调用这个过程的时候只需要知道它能返回什么而不需要知道它的实现细节

以平方根的计算过程举例:
* 一个 `sqrt` 过程可以分解为 `good-enough?`, `improve`, `average`, `abs` 等过程
* 也就是把 `sqrt` 分解成了可以清楚标明的工作
* 在实现 `sqrt` 的过程中, 不需要考虑这些辅助过程的细节, 只需要相信它们

**局部名**: 过程中形参的名字, 是过程使用者不需要关注的内容
* **约束变量** 形参的名字所代表的变量
* **自由变量** 不是被约束的变量

在抽象过程时, 如果对约束变量有任何改变, 这样就会影响外部的过程\
因为这个变量可能在外部过程中还被用到, 这个内部的过程也就不是"黑箱"

**作用域**: 一个约束变量所能生效的表达式 \
比如过程主体就是形参的作用域

在 `good-enough?` 中, `guess` 和 `x` 是约束变量, `<`, `abs` 等则不是 \
所以约束变量的名字要避开自由变量的名词, 从而避免发生冲突

----
现在 `sqrt` 过程由一些相互分离的过程定义, 而用户只需要用到 `sqrt` \
在构建大型程序时, 过多的辅助过程往往会扰乱视线, 还可能出现重名的情况 \
所以可以把这些辅助过程局部化, 这就需要内部定义

```scheme
(define (mysqrt x)
  (define (good-enough? guess x)
    (< (abs (- (* guess guess) x)) 0.001)) 
  (define (improve guess x)
    (average guess (/ x guess)))
  (define (sqrt-iter guess x)
    (cond ((good-enough? guess x) guess)
          (else (sqrt-iter (improve guess x) x))))
  (sqrt-iter 1.0 x))
```
注意到所有的辅助过程都在 `sqrt` 的形参 `x` 的作用域内, 所以不用来回传 `x`的值


```scheme
(define (mysqrt x)
  (define (good-enough? guess)
    (< (abs (- (* guess guess) x)) 0.001)) 
  (define (improve guess)
    (average guess (/ x guess)))
  (define (sqrt-iter guess)
    (cond ((good-enough? guess) guess)
          (else (sqrt-iter (improve guess)))))
  (sqrt-iter 1.0))
```
在辅助过程中 `x` 是自由变量, 由外部传给 `x` 的实际参数决定 \
这种方法被称作**词法作用域**, 也叫**静态作用域**\
在定义本过程的环境中寻找 `x` 在哪里被定义


## 1.2 过程及其产生的计算

本节考察过程所产生计算的顺序和消耗时间空间资源的速率

### 1.2.1 线性的递归和迭代

1. **递归计算过程**: 
    ```scheme
    (define (fact n)
    (cond ((= n 1) 1)
            (else (* n (fact (- n 1))))))
    ```
    * 在展开阶段, 递归构造一个**推迟进行**的操作所构成的链条
    * 在收缩阶段, 实际执行这些运算

    所以解释器要维护那些之后要执行的收缩阶段的轨迹 \
    轨迹的长度与 $n$ 成正比, 所以说这是一个 **线性递归过程**
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
2. **迭代计算过程**
    ```scheme
    (define (fact n)
      (define (iter product counter)
        (cond ((> counter n) product)
              (else (iter (* product counter) (+ counter 1)))))
      (iter 1 1))
    ```
    在这个计算过程中没有任何的展开和收缩, 需要保存的只有 `product` 和 `counter` \
    这种可以使用固定状态描述计算过程的就是迭代计算过程 \
    同样, 这也是 **线性迭代过程**

> 注意递归过程和递归计算过程并不同, 递归过程是一个语法上的概念 \
> 上述的迭代计算过程的具体实现也是采用递归过程

在迭代计算过程中, 形参都提供了计算状态的完整描述, 如果因为某些不可抗力程序终止 \
那么需要重新唤醒这个程序, 只需要提供两个形参即可 \
而递归计算过程还有一些隐含的变量并没有显式地保存在程序变量中, 而是由解释器维护 \
所以递归计算过程的链条越长, 解释器需要保存的信息也越多

在其他大多数语言中, 即使是迭代计算过程, 只要使用递归过程实现, 那么消耗的空间就是线性的 \
所以需要借助一些语法糖比如 `for`, `while` 才能实现迭代计算过程

而`scheme`的解释器实现没有这个缺点, 只要是迭代计算过程, 消耗的空间就是常数级别
具有这一特性的实现是 **尾递归** 的

### 1.2.2 树形递归

考虑斐波那契数列的计算, $f(n) = f(n-1)+f(n-2)$ \
每次计算的时候都要调用两次自身, 画成结构图就是一棵树
```scheme
(define (fib n)
  (cond ((or (= n 1) (= n 0)) 1)
        (else (+ (fib (- n 1)) (fib (- n 2))))))

```
一般来说, 树形递归结构的深度代表空间占用, 节点数代表运算次数(时间占用)

这种树形递归求$f(n)$ 的节点数正好是$f(n+1)$, 而斐波那契数列的值是成指数增长 \
所以这种计算的方式效率极低, 可以采用线性迭代计算过程
```scheme
(define (fib n)
  (define (fib-iter a b counter)
    (cond ((= counter n) b)
          (else (fib-iter b (+ a b) (+ counter 1)))))
  (fib-iter 0 1 0))
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

**增长的阶** 粗略度量某个过程所需资源的情况 \
$R(n)$ 表示计算过程在处理 $n$ 规模的问题时所需要的资源量 \
称 $R(n)$ 具有 $\Theta(f(n))$的增长阶, 其中 $R(n) = k \times f(n)$ 其中 $k$ 为常数

在线性递归计算过程中 时间需求的增长为 $\Theta(n)$, 空间需求的增长为 $\Theta(n)$ \
在线性迭代计算过程中 时间需求的增长为 $\Theta(n)$, 空间需求的增长为 $\Theta(1)$ 即常数空间

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
1. 如果这个数 $b$ 是偶数, 那么计算它的 $(a^{{\frac b 2})^2}$
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

结果是死循环, `scheme` 以应用序求值, 对于 `new-if` 这个过程 \
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

第一个: 递归计算过程
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
第二个: 迭代计算过程
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
(define (fast-pow a b)
  (cond ((zero? b) 1)
        ((even? b) (square (fast-pow a (/ b 2))))
        (else (* a (fast-pow a (- b 1))))))

$$
\begin{align}
 & \gamma = \frac{1 - \sqrt 5}{2}, \phi =\frac{1 + \sqrt 5}{2} 为方程 x^2-x-1=0两根 \\
 & 所以 \phi^2 = \phi + 1, \gamma^2 = \gamma + 1 \\
 & \phi ^n = \phi^{n-2} \times\ \phi^2 =\phi^{n-2}(\phi+1)=\phi^{n-1}+\phi^{n-2}, \gamma 同理 \\
 & f(0) = \frac{\phi^0-\gamma^0}{\sqrt5}, f(1)=\frac{\phi^1-\gamma^1}{\sqrt5} \\
 & f(n)=f(n-1)+f(n-2) 和 \gamma, \phi 的递推公式相同, f(n) = \frac{\phi^n-\gamma^n}{\sqrt 5} \\
 & \because \lvert \frac{\gamma}{\sqrt5} \rvert < \frac 1 2 \; \therefore \lvert \frac{\gamma^n}{\sqrt5} \rvert <\frac 12 \\
 & f(n) = \frac{\phi^n}{\sqrt5} - \frac{\gamma^n}{\sqrt5},  得证f(n) 是距离\frac{\phi^n}{\sqrt5} 最近的整数
\end{align}
$$

### 1.14

![img](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/QQ截图20230703174405.47ncpxb1kgc0.webp)
空间 $\Theta(n)$ \
时间 [$\Theta(n^5$)](https://sicp-solutions.net/post/sicp-solution-exercise-1-14/)

### 1.15 

1. 5次, 因为 $0.1 \times 3^5$ 才能大于 $12.15$
2. 每次问题规模缩小三分之一, 而且为递归计算过程, 空间时间都是 $\Theta(logn)$


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
& a \rightarrow (bp + a(p + q)) \rightarrow b(2pq+p^2) + a(p^2+q^2+2pq+p^2) \\
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


