---
title: learnsicp
date: 2023-07-02 15:04:05
tags:
- Scheme
categories: Book
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


# 练习

## 1

### 1.1

```scheme
10 ;10
(+ 3 5 4) ;12
(- 9 1) ;8
(/ 6 2) ;3
(+ (* 2 4) (- 4 6)) ;-16
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
   (* 3 (- 6 2) (-2 7)))
```

### 1.3

```scheme
(define (process a b c)
    (cond ((and (< a b) (< a c)) (+ b c))
          ((and (< b a) (< b c)) (+ a c))
          ((and (< c a) (< c b)) (+ a b))))
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
