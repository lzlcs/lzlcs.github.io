---
title: CSAPP 学习笔记 (第三章) 
date: 2023-08-04 14:33:23
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# Chapter 4: 处理器体系结构

一个处理器支持的指令和指令的字节级编码称为它的 **指令集体系结构(ISA)**

## 4.1 Y86-64 指令集体系结构

### 4.1.1 程序员可见的状态

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2jxl71rlhei0.png)
15 个寄存器 (省略 `%r15`) 以简化指令的编码 \

### 4.1.2 Y86-64 指令

Y86-64 只包含八字节的整数操作

1. `movq` 被分解成了四个指令
    - `irmovq` 立即数到寄存器
    - `rrmovq` 寄存器到寄存器
    - `mrmovq` 内存到寄存器
    - `rmmovq` 寄存器到内存
2. 四个整数操作指令: `addq`, `subq`, `andq`, `xorq` \
   这些指令设置三个条件码 `ZF`, `SF`, `OF`
3. 七个跳转指令: `jmp`, `je`, `jne`, `jg`, `jge`, `jl`, `jle`
1. 六个条件传送指令: `cmove`, `cmovne`, `cmovg`, `cmovge`, `cmovl`, `cmovle`
1. `call` 将返回地址入栈, 跳转到目的地址 \
   `ret` 把返回地址出栈, 跳转到返回地址
1. `pushq`, `popq` 入栈出栈
1. `halt` 指令停止指令的执行, 并将状态码设置成 `HLT`


### 4.1.3 指令编码

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2dv3hwhumk00.png)

每条指令的第一个字节表明指令的类型, 高四位是代码部分, 低四位是功能部分

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6965wwbntzs0.webp)

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.71p8q0sfgug0.webp)

`RISC` 和 `CISC` 的区别:
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.29lao3kl50is.png)

### 4.1.4 Y86-64 异常

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.pzpx8rvb528.webp)

正常的处理器遇到错误时会调用异常处理程序, 在这里只是简单地让程序终止

### 4.1.5 Y86-64 程序

Y86-64 将常数加载到寄存器, 因为它不能在算数指令中使用立即数

### 4.1.6 一些 Y86-64 指令的详情

`pushq %rsp` 压入原始值, 之后再把 `%rsp` 减去 8

## 4.2 逻辑控制语言和硬件控制语言 `HDL`

逻辑 1 使用 1.0V 左右的电压表示, 逻辑 0 使用 0.0V 左右的电压表示

实现一个数字系统需要三个主要的组成部分
1. 计算对位进行操作的函数的组合逻辑
2. 存储位的存储器单元
3. 控制存储器单元更新的时钟信号

`HCL` 硬件控制语言, 用于描述处理器的控制逻辑

### 4.2.1 逻辑门

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.hj9p4vcwrug.webp)

### 4.2.2 组合电路和 HCL 布尔表达式

很多的逻辑门组合成的网可以构建一个计算块, 称为组合电路, 构建有如下限制
1. 输入来源: 一个系统输入 / 某个存储器单元的输出 / 某个逻辑门的输出
2. 多个逻辑门的输出不能连接在一起, 否则信号可能冲突
3. 逻辑门组合的网不能有环, 否则会产生歧义

---
```verilog
bool out = (s && a) || (!s && b);
```
这是多路复用器 (`MUX`), `s = 0` 时输出 `b`, 否则输出 `a`

注意 `HCL` 和 `C` 的区别:
1. 电路的输入变化了, 经过延迟之后输出也会变化
2. 参数只能是 0 / 1
3. 没有短路效果

### 4.2.3 字级的组合电路和 HCL 整数表达式

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.38bhs0bw8bm0.png)

字级相等操作: `bool Eq = (A == B)`

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.fuwslqxadgw.png)

字级多路复用器: \
这种格式按顺序求 冒号之前的值, 到第一个结果为 1 的位置再执行冒号之后的代码 \
结尾的 1 表示 `else`
```verilog
word Out = [
    s: A;
    1: B;
];
```
```
word Out = [
    !s0 && !s1: A;
    !s0       : B;
    !s1       : C;
    1         : D;
];
```

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.12bozstdokds.webp)

### 4.2.4 集合关系

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1l9updfx4oow.webp)

拆解 `code`:
```
bool s1 = (code == 2) || (code == 3);
bool s0 = (code == 1) || (code == 3);
```
也可以
```
bool s1 = code in { 2, 3 };
bool s0 = code in { 1, 3 };
```

## 4.2.5 存储器和时钟

考虑两类存储设备:
1. 寄存器: 时钟信号控制寄存器加载输入值
2. 内存(随机访问存储器): 用地址来决定读写哪个字

硬件寄存器: 时钟是低电位则输出不变, 时钟变成高电位的时候, 输出变为输入

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.22uac83gf2ao.webp)





# 练习

## 4.1 ~ 4.2

体力活

## 4.3

```
sum:
    xorq %rax,%rax # sum = 0
    andq %rsi,%rsi # Set CC
    jmp test # Goto test
loop:
    mrmovq (%rdi),%r10 # Get *start
    addq %r10,%rax # Add to sum
    iaddq $8,%rdi # start++
    iaddq $-1,%rsi # count--. Set CC
test:
    jne loop # Stop when 0
    ret # Return
```

## 4.4

```
rsum:
    andq %rsi, %rsi
    jle .L3
    pushq %rbx
    mrmovq (%rdi), %rbx
    irmovq $1, %r10
    subq %r8, %rsi
    irmovq $8, %r10
    subq %r9, %rdi
    call rsum
    addq %rbx, %rax
    popq %rbx
    ret
.L3:
    irmovq $0, %rax
    ret
```

## 4.5

```
absSum:
    irmovq $8, %r8
    irmovq $1, %r9

    xorq %rax, %rax

    andq %rsi, %rsi

    jmp test

loop:
    mrmovq (%rdi), %r10
    xorq %r11, %r11
    subq %r10, %r11
     
    jle pos
    rrmovq %r11, %r10

pos:
    addq %r10, %rax
    addq %r8, %rdi
    subq %r9, %rsi

test:
    jne loop
    ret

```

## 4.6

```
absSum:
    irmovq $8, %r8
    irmovq $1, %r9

    xorq %rax, %rax

    andq %rsi, %rsi

    jmp test

loop:
    mrmovq (%rdi), %r10
    xorq %r11, %r11
    subq %r10, %r11

    cmovg %r11, %r10

    addq %r10, %rax
    addq %r8, %rdi
    subq %r9, %rsi

test:
    jne loop
    ret
```

## 4.7

压入原始值

## 4.8

把栈顶的值弹出给栈指针 \
Y86-64 中, `mrmovq (%rsp), %rsp`

## 4.9

```verilog
bool xor = (!a && b) || (!b && a);
```

## 4.10

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6xh7p6q5sqs0.webp)

## 4.11

```
word MIN3 = [
    A <= B && A <= C: A;
    B <= C          : B;
    1               : C;
];
```

## 4.12

```
word MID3 = [
    B <= A && A <= C: A;
    C <= A && A <= B: A;
    A <= B && B <= C: B;
    C <= B && B <= A: B;
    1               : C;
];
```
