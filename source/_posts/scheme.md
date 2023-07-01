---
title: Scheme 学习笔记
date: 2023-07-01 16:52:51
tags: 
- Scheme
categories: Language
---

# 前期准备

1. 安装 `MIT scheme`: `sudo apt-get install mit-scheme`
2. 在命令行输入 `scheme` 进入 `scheme` 解释器

# 基本表达式

`scheme` 中一对括号代表一次计算的步骤, 括号中的计算式采用前缀表达式 \
形如这些由括号, 标记, 分隔符组成的式子, 被称为 `S-表达式`
```scheme
(+ 1 2) ;3
(+ 1 2 4) ;7
```
当然括号可以嵌套
```scheme
(* (+ 1 3) (- 3 1)) ;8
```
像 `+`, `1`, `2` 这些基本表达式被称为原子(`atom`)
每个表达式用空格, 制表符或换行符来分割

## 四则运算

1. `+`, `-`, `*`
1. `/` 注意`scheme`会处理分数, 比如
    ```scheme
    (/ 3 4) ;3/4
    (/ 29 3 7) ;29/21
    ```
    函数 `exact->inexact` 用于把分数转换为小数

## 其他运算

1. `quotient` 整除, `modulo` 取模, `sqrt` 求平方根
2. `sin`, `cos`, `tan`, `asin`, `acos`, `atan`
3. `(exp a b)` 求指数, `(log x)` 求以 `e` 为底的对数

# 生成表 (`list`)

## Cons 单元

表的元素称为 `Cons单元`,是一个存放了两个地址的内存空间
```scheme
(cons 1 2) ;(1 . 2)
(cons (cons 1 3) (cons 2 3)) ;((1 . 3) 2 . 3)
```
`cons` 分配了两个地址空间, 一个指向 1, 一个指向 2 , `cons` 也是 `construction` 的简称 \
指向 1 的部分被称作 `car` -> `Contents of the Address part of the Register` \
指向 2 的部分被称作 `cdr` -> `Contents of the Department part of the Register`

## 表

表是一个由 `Cons单元` 串起来的类似链表的结构, 末尾为空列表 `'()`
```scheme
(cons 2 (cons 3 (cons 4 '()))) ;(2 3 4)
(cosn x y) ;如果 y 是个表, 那么就把 x 插到表头
```

## 原子

不使用 `cons` 来构建的元素都为原子 (`atom`)
如 数字, 字符, 字符串, 空表

## 引用

`quote` 阻止表达式被求值, 可以简写成 `'`

```scheme
(quote (+ 3 5)) ;(+ 3 5)
'(+ 3 5) ;(+ 3 5)
```

## 特殊形式

除了 `quote`, `define`, `lambda`, `if`, `set!` 等都是特殊形式

## `car` 和 `cdr`

```scheme
;; 这里使用引用就是不让表达式 (2 3 4) 被求值, 也无法求值
(car '(2 3 4)) ;2
(cdr '(2 3 4)) ;(3 4)
```

## `list`

`list` 函数可以有多个参数, 构建任意长度的表
```scheme
(list 2 3 4 '(3 4) '((3 4 5) (4))) ;(2 3 4 (3 4) ((3 4 5) (4)))
```


# 定义函数

1. 定义变量(没有参数的函数)
    ```scheme
    (define x 5) 
    ```
2. 定义函数的两种方法
   ```scheme
   (define func 
     (lambda (arguments)
       (+ arguments 5)))
   ;; 以下是短形式
   (define (func arguments)
     (+ arguments 5))
   ```
使用 `(load [filename])` 载入文件从而测试函数

# 条件分支

1. `if` 表达式: `(if condition process1 process2)` \
    如果 `condition` 为真, 则执行 `process1`, 否则 `process2`, 这两个都是 `S-表达式`

2. `and` 和 `or` 接受多个参数, 有短路特性
3. `cond` 表达式: 字面意思, 很好懂
    ```scheme
    (cond 
      (condition1 process1)
      (condition2 process2)
      ....
      (else process_else))
    ```

其他做出判断的函数

1. 函数 `null?` 表示表是否为空
2. `eq?` 比较两个参数的地址, `eqv?` 比较两个参数的值(不能为表或字符串) \
   `equal?` 比较值(适用于字符串和表)
3. `pair?`, `list?` 顾名思义, 注意空表是 `list` 不是 `pair`
4. `=`, `>`, `<`, `<=`, `>=` 接受任意数量的数字参数
5. `char=?`, `char<?`, `char>?`, `char<=`, `char>=`
6. `string=?`, `string-ci=?`

# 局部变量

```scheme
(let ((name1 value1)
      (name2 value2)
      ....
      (namen valuen))
  (body))
```
`letrec` 替代 `let` 之后可以允许变量递归调用自己

注意这些局部变量只能在 `body` 中使用

## 递归和尾递归

普通的线性递归转换为尾递归的方法就是把结果传到参数里, 这样可以直接返回, 避免空间占用
```scheme
(define (fact n)
  (if (= n 1) 
    1
    (* n fact (- n 1))))
(define (fact n res)
  (if (= n 1) 
    res
    (fact (- n 1) (* n res))))
```

# 高阶函数

类似 `python`, 略


# 副作用

`begin` 表达式后可跟多个参数, 依次执行这些参数之后返回最后一个参数结果

# 宏

## Common Lisp 式

编写代码的代码, 在CS61A的 Project4 中实现的是这种形式

```scheme
`() 括号里的东西都被引用
,x 解除x的引用
,@x 把作为列表的x展开
```





