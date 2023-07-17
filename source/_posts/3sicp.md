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


# Chapter 3: 模块化, 对象和状态

前两章介绍了组成程序的基本元素, 把函数和数据组合在一起构造出复合的实体 \
认识到抽象有着至关重要的作用

但我们还需要一些能够帮助我们构造起模块化的大型系统的策略 \
使这些系统能够自然地划分为一些具有内聚力的部分, 从而使得这些部分能够分别开发和维护

有一种强有力的策略是基于被模拟系统的结构去设计程序的结构 
1. 对于有关的物理系统里的每个对象, 我们构造起一个与之对应的计算对象
2. 对于该系统中的每个活动, 在自己的计算系统中定义一种符号操作

如果我们在系统的组织上非常成功, 那么在需要添加新特征或者排除旧东西错误的时候, 就只需要在局部工作


本章研究两种特点鲜明的组织策略
1. 将注意力集中在对象上, 将一个大型系统看成一大批对象
2. 将注意力集中在流过系统的信息流上

对于对象途径而言, 我们把必须关注计算对象可以怎么样变化又同时保证其标识 \
这将迫使我们抛弃老的计算的代换模型, 转向环境模型

流方式能够用于松解对时间的模拟和各种事件发生的顺序, 可以通过延时求值来做到这一点

## 3.1 赋值和局部状态

我们可以使用几个变量来刻画一个对象的状态

```python
balance = 100

def withdraw(amount):
    global balance
    if (balance >= amount):
        balance = balance - amount
        return balance
    return "Insufficient fundds"
```

之前的所有函数都可以看作数学函数, 因为他们对某些确定的参数只会返回确定的结果

但这样的实现有些问题, 因为 `balance` 被定义在全局, 可以被其他函数检查或者修改

```python
def new_withdraw():
    balance = 100
    def helper(amount):
        nonlocal balance
        if (balance >= amount):
            balance = balance - amount
            return balance
        return "Insufficient funds"
    return helper

x = new_withdraw()

print(x(25))
```

```python
class Account:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        if (self.balance >= amount):
            self.balance = self.balance - amount
            return self.balance
        return "Insufficient funds"

    def deposit(self, amount):
        self.balance = self.balance
        self.balance += amount
        return self.balance

acc = Account(100)
print(acc.withdraw(50))
print(acc.withdraw(60))
print(acc.deposit(40))
print(acc.withdraw(60))
```

这类似于之前的消息传递过程

### 3.1.2 引进赋值所带来的利益

现在考虑如何设计出一个生成随机出的函数

```python
x2 = rand_update(x1)
x2 = rand_update(x2)
```

蒙特卡洛模拟: 从大集合里面随机选择实验样本, 并对这些实验结果的统计估计做出推断

随机选取的两个整数之间最大公约数是 1 的概率为 $\frac{6}{\pi ^2}$

```python
import math
import random

def estimate_pi(trials):
    return math.sqrt(6 / monte_carlo(trials, cesaro_test))

def cesaro_test():
    return math.gcd(random.randint(1, 100000), random.randint(1, 100000)) == 1

def monte_carlo(trials, experiment):
    passed_trials = 0
    for _ in range(trials):
        if (experiment()):
            passed_trials += 1
    return passed_trials / trials

print(estimate_pi(1000000))
```


```python
def rand_update(_):
    return random.randint(1, 1000000)

def random_gcd_test(trials, initial_x):
    x = initial_x
    passed_trials = 0
    for _ in range(trials):
        x1 = rand_update(x)
        x2 = rand_update(x1)
        if (math.gcd(x1, x2) == 1):
            passed_trials += 1
        x = x2
    return passed_trials / trials

print(math.sqrt(6 / random_gcd_test(100000, 1)))
```

第一段不需要显式地操作 $x_1$, $x_2$, 而且生成随机数的操作被隔离, 可以更好地进行模块化

与所有状态都必须显式地操作和传递额外参数的方法相比 \
通过引进赋值和将状态隐藏在局部变量的技术, 可以让我们以一种更模块化的方式来构造系统

### 3.1.3 引进赋值的代价

```python
def make_simplified_withdraw(balance):
    def helper(amount):
        nonlocal balance
        balance -= amount
        return balance
    return helper

def make_decrementer(balance):
    return lambda amount: balance - amount
```

只要我们不适用赋值, 同样参数对于同一函数的两次求值结果必然相同, 相当于计算数学函数 \

代换模型的基础是: 变量只不过是值的名字

但是引入赋值之后, 一个变量就索引着一个保存值的位置

**同一和变化**:

如果一个语言支持在表达式里"同一的东西可以相互替换", \
这种替换不会改变表达式的值, 这个语言就具有**引用透明性**

包含赋值操作也就打破了引用透明性


不用任何赋值的程序设计称为函数式编程 \
大量使用赋值的程序设计称为命令式编程

命令式编程的一个问题式需要考虑赋值的先后顺序
```python
def factorial(n):
    product, counter = 1, 1
    for _ in range(n):
        product *= counter
        counter += 1

    return product

print(factorial(5))
```


更换 `product` 和 `counter` 的求值顺序, 结果就会变得不同 \
所以带有赋值的程序强迫人们去考虑赋值的相对顺序 \
而在函数式编程中, 这样的问题根本不会存在 \
在考虑并发执行的进程中, 命令式编程的复杂度还会增加


## 3.2 求值的环境模型

一个环境就是框架的一个序列, 每个框架是包含着约束的表格 \
这些约束将一些变量名字关联于对应的值, 并且在同一个框架里, 一个变量至多只能有一个约束

每个框架还包含着一个指针, 指向这一框架的外围环境 (全局的框架没有这个指针)

### 3.2.1 求值规则

在求值的环境模型里 \
将一个函数应用于一些实参, 将构造出一个新框架, 将函数中的形参约束到调用时的实参 \
之后在构造的新环境的上下文中求值函数主体, 新框架的外围环境就是调用函数时的环境

### 3.2.2 简单过程的应用

本节分析一个实例

### 3.2.3 将框架看作局部状态的展台

本节分析另一个实例

### 3.2.4 内部定义

内部定义的函数被约束到当前环境中

## 3.3 用变动数据做模拟

在第二章中, 各种数据抽象包括选择函数和构造函数 \
现在我们要加入改变函数

### 3.3.1 变动的表结构

第一部分讲了 `set-car`, `set-cdr`

第二部分: 共享和相等

```python
a = [1, 2]
b = [a] + [a]
print(b)
b[0][0] = 0
print(b)

b = [[1, 2], [1, 2]]
print(b)

b[0][0] = 0
print(b)
```

两次输出不同

通过 `is` 检查两者地址是否相同

```python
a = [1, 2]
b = [a] + [a]
print(b)
b[0][0] = 0
print(b)

print(b[0] is b[1])

b = [[1, 2], [1, 2]]
print(b)

b[0][0] = 0
print(b)

print(b[0] is b[1])
```




















# 练习

## 3.1

```python
def make_accumulater(sum):
    def helper(x):
        nonlocal sum
        sum += x
        return sum
    return helper

A = make_accumulater(5)
print(A(10))
print(A(10))
```

## 3.2

```python
import math
def make_monitored(func):
    times = 0
    def dispatch(x):
        nonlocal times
        if (x == "how_many_calls?"):
            return times
        elif (x == "reset_count"):
            times = 0
        else:
            times += 1
            return func(x)
    return dispatch


s = make_monitored(math.sqrt)

print(s(100))
print(s("how_many_calls?"))
s("reset_count")
print(s(144))
print(s("how_many_calls?"))
```

## 3.3

```python
class Account:
    def __init__(self, balance, password):
        self.balance = balance
        self.password = password

    def withdraw(self, password, amount):
        if (password != self.password):
            return "Incorrect password"

        if (self.balance >= amount):
            self.balance = self.balance - amount
            return self.balance
        return "Insufficient funds"

    def deposit(self, amount):
        self.balance = self.balance
        self.balance += amount
        return self.balance

s = Account(100, "3")

print(s.withdraw("3", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
```

## 3.4

```python
call_the_cops = lambda: "cops are coming"
class Account:
    def __init__(self, balance, password):
        self.balance = balance
        self.password = password
        self.times = 0

    def withdraw(self, password, amount):
        if (password != self.password):
            self.times += 1
            if (self.times == 7):
                return call_the_cops()
            return "Incorrect password"

        if (self.balance >= amount):
            self.balance = self.balance - amount
            return self.balance
        return "Insufficient funds"

    def deposit(self, amount):
        self.balance = self.balance
        self.balance += amount
        return self.balance

s = Account(100, "3")

print(s.withdraw("3", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
print(s.withdraw("4", 50))
```


## 3.5

```python

x0, y0, r = 100, 100, 50 
def estimate_integral(p, x1, x2, y1, y2, trails):
    experiment = lambda: p(x1, x2, y1, y2)
    s = abs(x1 - x2) * abs(y1 - y2)
    return s * monte_carlo(trails, experiment) / r / r

def p(x1, x2, y1, y2):
    x = random.randint(x1, x2) - x0
    y = random.randint(y1, y2) - y0
    return x * x + y * y <= r * r

print(estimate_integral(p, 0, 200, 0, 200, 100000))
```

## 3.6

```python
class Rand:
    def __init__(self, init):
        self.cur = init

    def calculate(self):
        def update_func(x):
            return 0

        self.cur = update_func(self.cur)
        return self.cur

    def generate(self):
        return self.calculate()

    def reset(self, new_value):
        self.cur = new_value
```

## 3.7

```python
call_the_cops = lambda: "cops are coming"

class Account:
    def __init__(self, balance, password):
        self.balance = balance
        self.password = password
        self.times = 0

    def wrong_password(self):
        self.times += 1
        if self.times == 7:
            return call_the_cops()
        return "Incorrect password"

    def withdraw(self, password, amount):
        if password != self.password:
            return self.wrong_password()

        if self.balance >= amount:
            self.balance -= amount
            return self.balance
        return "Insufficient funds"

    def deposit(self, amount):
        self.balance += amount
        return self.balance

    def make_joint(self, old_pass, new_pass):
        if old_pass != self.password:
            return self.wrong_password()

        class JointAccount(Account):
            def __init__(self, original_account, new_pass):
                self.ori = original_account
                self.password = new_pass
                self.times = 0

            def withdraw(self, password, amount):
                if (password == self.password):
                    return self.ori.withdraw(self.ori.password, amount)

            def deposit(self, amount):
                return self.ori.deposit(amount)

        return JointAccount(self, new_pass)

peter = Account(100, "111")
paul = peter.make_joint("111", "222")
assert(isinstance(paul, Account)) #type: ignore
amy = paul.make_joint("222", "333")
assert(isinstance(amy, Account)) #type: ignore

print(peter.withdraw("111", 10))  # 输出: 90
print(peter.deposit(30))
print(paul.withdraw("222", 10))  # 输出: 80
print(paul.deposit(30))
print(amy.withdraw("333", 10))  # 输出: 70
print(amy.deposit(30))
```

还有另外一种取巧的写法

```python
call_the_cops = lambda: "cops are coming"

class Account:
    def __init__(self, balance, password):
        self.balance = balance
        self.password = [password]
        self.times = 0

    def wrong_password(self):
        self.times += 1
        if self.times == 7:
            return call_the_cops()
        return "Incorrect password"

    def withdraw(self, password, amount):
        if not password in self.password:
            return self.wrong_password()

        if self.balance >= amount:
            self.balance -= amount
            return self.balance
        return "Insufficient funds"

    def deposit(self, amount):
        self.balance += amount
        return self.balance

    def make_joint(self, old_pass, new_pass):
        if not old_pass in self.password:
            return self.wrong_password()
        self.password += [new_pass]
        return self

peter = Account(100, "111")
paul = peter.make_joint("111", "222")
assert(isinstance(paul, Account)) #type: ignore
amy = paul.make_joint("222", "333")
assert(isinstance(amy, Account)) #type: ignore

print(peter.withdraw("111", 10))  # 输出: 90
print(peter.deposit(30))
print(paul.withdraw("222", 10))  # 输出: 80
print(paul.deposit(30))
print(amy.withdraw("333", 10))  # 输出: 70
print(amy.deposit(30))
```

## 3.8

```python
x = 1 

def f(y):
    global x
    x *= y
    return x

a = f(0)
b = f(1)

print(a + b)

x = 1

a = f(1)
b = f(0)

print(a + b)
```

## 3.9

```python
                +------------------------------------------------------------------------------------------+
global env -->  |                                                                                          |
                |                                                                                          |
                +------------------------------------------------------------------------------------------+
                   ^               ^              ^                ^               ^               ^
            (f 6)  |        (f 5)  |       (f 4)  |         (f 3)  |        (f 2)  |        (f 1)  |
                   |               |              |                |               |               |
                +------+        +------+        +------+         +------+        +------+        +------+
                |      |        |      |        |      |         |      |        |      |        |      |
          E1 -> | n: 6 |  E2->  | n: 5 |  E3 -> | n: 4 |  E4 ->  | n: 3 |  E5 -> | n: 2 |  E6 -> | n: 1 |
                |      |        |      |        |      |         |      |        |      |        |      |
                +------+        +------+        +------+         +------+        +------+        +------+

             (* 6 (f 5))      (* 5 (f 4))     (* 4 (f 3))      (* 3 (f 2))     (* 2 (f 1))        1
```

```python

         +-----------------------------------------------------------------------------------------------------------------------------+
global   |                                                                                                                             |
env -->  |                                                                                                                             |
         |                                                                                                                             |
         +-----------------------------------------------------------------------------------------------------------------------------+
            ^               ^                 ^                ^               ^                 ^                  ^               ^
      (f 6) |     (i 1 1 6) |       (i 1 2 6) |      (i 2 3 6) |     (i 6 4 6) |      (i 24 5 6) |      (i 120 6 6) |   (i 720 7 6) |
            |               |                 |                |               |                 |                  |               |
        +-------+        +-------+        +-------+        +-------+        +-------+        +-------+        +-------+        +-------+
        |       |        | p: 1  |        | p: 1  |        | p: 2  |        | p: 6  |        | p: 24 |        | p:120 |        | p:720 |
  E1 -> | n: 6  |  E2 -> | c: 1  |  E3 -> | c: 2  |  E4 -> | c: 3  |  E5 -> | c: 4  |  E6 -> | c: 5  |  E7 -> | c: 6  |  E8 -> | c: 7  |
        |       |        | m: 6  |        | m: 6  |        | m: 6  |        | m: 6  |        | m: 6  |        | m: 6  |        | m: 6  |
        +-------+        +-------+        +-------+        +-------+        +-------+        +-------+        +-------+        +-------+
        (i 1 1 6)        (i 1 2 6)        (i 2 3 6)        (i 6 4 6)       (i 24 5 6)       (i 120 6 6)      (i 720 7 6)       720
```

## 3.10

```
              +-----------------------------------------------------------------------------------------+
global env -> |                                                                                         |
              |   w1                                        w2                                          |
              +---|-----------------------------------------|-------------------------------------------+
                  |                               ^         |                               ^
                  |          (make-withdraw 100)  |         |                               |
                  |                               |         |                               |
                  |                    +--------------+     |                      +--------------+
                  |                    |              |     |                      |              |
                  |             E1 ->  | initial: 100 |     |               E1 ->  | initial: 100 |
                  |                    |              |     |                      |              |
                  |                    +--------------+     |                      +--------------+
                  |                               ^         |                               ^
                  | ((lambda (balance) ...) 100)  |         | ((lambda (balance) ...) 100)  |
                  |                               |         |                               |
                  |                    +--------------+     |                      +--------------+
                  |                    |              |     |                      |              |
                  |             E2 ->  | balance: 50  |     |               E2 ->  | balance: 100 |
                  |                    |              |     |                      |              |
                  |                    +--------------+     |                      +--------------+
                  |                      |        ^         |                          |       ^
                  |                      |        |         |                          |       |
                  |                      v        |         |                          v       |
                  +------------------> [*][*]-----+         +----------------------->[*][*]----+
                                        |                                             |
                                        |                                             |
                                        v                                             v
                         parameters: amount                             parameters: amount
                         body: (if (>= balance amount)                  body: (if (>= balance amount)
                                   (begin (set! balance                           (begin (set! balance
                                                (- balance amount))                      (- balance amount))
                                          balance)                                balance)
                                   "Insufficient funds")                          "Insufficient funds")
```

## 3.11

```
          +---------------------------------------------------------------+
global -> |                                                               |
env       | make-account        acc2    acc                               |
          +----|-----------------|--------|-------------------------------+
               |       ^         |        |             ^
               |       |         |        |             |
               v       |         |        |   E1 -> +------------------+
             [*][*]----+         |        |         | balance: 30      |<----------+
              |                  |        |         |                  |           |
              |                  |        |         | withdraw --------------->[*][*]----> parameters: amount
              v                  |        |         |                  |                   body: ...
  parameters: balance            |        |         |                  |<----------+
  body: (define withdraw ...)    |        |         |                  |           |
        (define deposit ...)     |        |         | deposit ---------------->[*][*]----> parameters: amount
        (define dispatch ...)    |        |         |                  |                   body: ...
        (lambda (m) ...)         |        |         |                  |<----------+
                                 |        |         |                  |           |
                                 |        +---------->dispatch --------------->[*][*]----> parameters: m
                                 |                  |                  |                   body: ...
                                 |                  +------------------+
                                 |
                                 |
                                 |
                                 |  E6 -> +------------------+
                                 |        | balance: 100     |<----------+
                                 |        |                  |           |
                                 |        | withdraw --------------->[*][*]----> parameters: amount
                                 |        |                  |                   body: ...
                                 |        |                  |<----------+
                                 |        |                  |           |
                                 |        | deposit ---------------->[*][*]----> parameters: amount
                                 |        |                  |                   body: ...
                                 |        |                  |<----------+
                                 |        |                  |           |
                                 +--------->dispatch --------------->[*][*]----> parameters: m
                                          |                  |                   body: ...
                                          +------------------+
```

`acc` 的局部状态保存在环境 `E1` 中 \
因为环境不同, 所以局部状态不同 \
两者并不共享任何部分

## 3.12

```
x --> [*]----> [*]----> '()
       |        |
       v        v
       'a       'b

y --> [*]----> [*]----> '()
       |        |
       v        v
       'c       'd

z --> [*]---->[*]---->[*]---->[*]----> '()
       |       |       |       |
       v       v       v       v
      'a      'b      'c      'd
```

```
w------+
       |
       |
       v
x --> [*]----> [*]----+
       |        |     |
       v        v     |
       'a       'b    |
                      |
       +--------------+
       |
       v
y --> [*]----> [*]----> '()
       |        |
       v        v
       'c       'd

z --> [*]---->[*]---->[*]---->[*]----> '()
       |       |       |       |
       v       v       v       v
      'a      'b      'c      'd
```

两个 `<response>`:
```scheme
(b)
(b c d)
```

## 3.13

```
         +-----------------------+
         |                       |
         v                       |
z ----> [*]----> [*]----> [*]----+
         |        |        |
         v        v        v
        'a       'b       'c
```
无限循环

## 3.14

数组反转

```scheme
(a b c d)
(d c b a)
```

## 3.15

改变的是 `x`

`z1` 变两个
`z2` 只变一个

## 3.16

```python
car = lambda x: x[0]
cdr = lambda x: x[1:] if len(x) > 1 else []

def count_pairs(x):
    if (not type(x) == list or x == []):
        return 0
    return count_pairs(car(x)) + count_pairs(cdr(x)) + 1

def c(x):
    def inner(x, memo_list):
        if (type(x) == list and x != []
            and (not x in memo_list)):
            return inner(car(x), inner(cdr(x), [x] + memo_list))
        return memo_list
    return len(inner(x, []))

x = [[1], 2]
y = [[[1]], 1]
z = [[[1], 1], [1], 1]

print(count_pairs(x), c(x))
print(count_pairs(y), c(y))
print(count_pairs(z), c(z))
```

## 3.17

见 3.16

## 3.18


用 `python` 中的 `list` 实在是不好模拟, 所以用链表
直接用的快慢指针法

```python
class LinkNode:
    def __init__(self, val, next):
        self.val, self.next = val, next

x = LinkNode(3, LinkNode(4, LinkNode(5, None)))

x.next = x

def check(head):
    if (head == None or head.next == None):
        return False
    slow, fast = head, head.next
    while (fast and fast.next):
        if (slow == fast):
            return True
        slow, fast = slow.next, fast.next.next
    return False

print(check(x))
```
## 3.19

见 3.18

```
          +-------------------------------------------------------+
global -> |                                                       |
env       |  z                           x                        |
          +--|---------------------------|------------------------+
             |           ^               |          ^
             |           |               |          |
             |           |               |      +----------+
             |           |               |      | x: 17    |
             |           |               |      | y: 2     |
             |           |               |      |          |
             |           |               |      | set-x! -----> ...
             |           |               |      | set-y! -----> ...
             |           |               +------->dispatch ---> parameters: m
             |           |                      |  ^ ^     |    body: ...
             |           |                      +--|-|-----+
             |        +----------+                 | |
             |  E2 -> | x: ------------------------+ |
             |        | y: --------------------------+
             |        |          |
             |        | set-x! -----> ...
             |        | set-y! -----> ...
             +--------->dispatch ---> parameters: m
                      |          |    body: (cond ((eq? m 'car) 'car)
                      +----------+                ((eq? m 'cdr) 'cdr)
                                                  ((eq? m 'set-car!) 'set-car!)
                                                  ((eq? m 'set-cdr!) 'set-cdr!)
                                                  (else
                                                    (error "..." m)))
```



