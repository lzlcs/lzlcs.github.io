---
title: CSAPP 学习笔记 (第四章) (暂跳过流水线实现)
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

### 4.2.5 存储器和时钟

考虑两类存储设备:
1. 寄存器: 时钟信号控制寄存器加载输入值
2. 内存(随机访问存储器): 用地址来决定读写哪个字

硬件寄存器: 时钟是低电位则输出不变, 时钟变成高电位的时候, 输出变为输入

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.22uac83gf2ao.webp)

## 4.3 Y86-64 的顺序实现

### 4.3.1 将处理组织成阶段

各个阶段:
1. 取指: 从内存中取出 PC 所指的代码
2. 译码: 读入至多两个操作数
3. 执行: 执行代码
4. 访存: 将数据写入内存或从内存读出
5. 写回: 最多写两个结果到寄存器文件
6. 更新 PC: 下一条指令

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.4daz640bsx40.png)
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2t22bubmw7q0.png)
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1ki6n5ahao00.png)
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.65xvm99mvko0.webp)


### 4.3.2 SEQ 硬件结构

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6ylve3dk4t0.png)

### 4.3.3 SEQ 的时序

SEQ 的实现包括:
1. 组合逻辑
2. 两种存储设备
    - 时钟寄存器: PC, 条件码寄存器
    - 随机访问存储器: RAM, 寄存器

组合逻辑不需要时序控制, 只要输入变化就会结果变化


从不回读: 处理器不需要为了完成一条指令的执行而去读这个指令更新了的状态

当时钟开始下一个周期时, 所有状态才会更新

### 4.3.4 SEQ 阶段的实现

常数表:
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.252twoaililc.png)

`imem_error` 指令地址不合法 \
`dmem_error` 内存地址不合法 \
`instr_valid` 指令是否合法 \
`need_regids` 是否需要寄存器 \
`need_valC` 是否需要 `valC`

**取指阶段**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.73zd07zl3gw0.png)

```
bool need_regids = 
        icode in { IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, 
                   IIRMOVQ, IRMMOVQ, IMRMOVQ };
bool need_valC = 
        icode in { IRMMOVQ, IMRMOVQ, IIRMOVQ, IJXX, ICALL };
```

新的指令地址: `valP = PC + 1 + need_regids + 8 * need_valC`

**译码和写回阶段**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.35hadi9oyai0.webp)

```
word srcA = [
        icode in { IRRMOVQ, IRMMOVQ, IOPQ, IPUSHQ } : rA;
        icode in { IPOPQ, IRET } : RRSP;
        1 : RNONE;
]

word srcB = [
        icode in { IOPQ, IRMMOVQ, IMRMOVQ } : rB;
        icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
        1 : RNONE;
]

word dstE = [
        icode in { IRRMOVQ } : rB;
        icode in { IIRMOVQ, IOPQ } : rB;
        icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
        1 : RNONE;
]

word dstM = [
        icode in { IMRMOVQ, IPOPQ } : rA;
        1 : RNONE;
]

```

**执行阶段**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.f89igfzn714.webp)

```
word aluA = [
        icode in { IRRMOVQ, IOPQ } : valA;
        icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ } : valC;
        icode in { ICALL, IPUSHQ } : -8;
        icode in { IRET, IPOPQ } : 8;
]

word aluB = [
        icode in { IOPQ, IRMMOVQ, IMRMOVQ, IPUSHQ, 
                   IPOPQ, ICALL, IRET } : valB;
        icode in { IRRMOVQ, IIRMOVQ } : 0;
]

word alufun = [
        icode == IOPQ : ifun;
        1 : ALUADD;
]

bool setCC = icode in { IOPQ };
```

在这里省略 `cond` 模块的设计

**访存阶段**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.32vg9spt8js0.webp)

```
word mem_addr = [
        icode in { IRMMOVQ, IPUSHQ, ICALL, IMRMOVQ } : valE;
        icode in { IPOPQ, IRET } : valA;
]

word mem_data = [
        icode in { IRMMOVQ, IPUSHQ } : valA;
        icode in { ICALL } : valP;
]

bool mem_read = 
        icode in { IMRMOVQ, IPOPQ, IRET };

bool mem_write = 
        icode in { IRMMOVQ, IPUSHQ, ICALL };

word Stat = [
        imem_error || dmem_error : SADR;
        !instr_valid : SINS;
        icode == IHALT : SHLT;
        1 : SAOK;
]
```

**更新 PC 阶段**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2si6ar556eq0.webp)

```
word new_pc = [
        icode == ICALL : valC;
        icode == IJXX && Cnd : valC;
        icode == IRET : valM;
        1 : valP;
]
```

**SEQ小结**

顺序执行使得每个时钟周期都必须完成这一系列操作, 所以时钟频率会非常低

## 4.4 流水线的通用原理

### 4.4.1 计算流水线

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6yys0oce0hc0.png)

$$
吞吐量 = \frac { 1 条指令 }{ (20 + 300)ps } \times \frac {1000ps}{lns} = 3.12GPS
$$
$lns = 10^{-9}s$

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.418nh9ufbak0.png)

把任务分成一些阶段, 在中间放上流水线寄存器
1. 时钟周期为 $120ps$
2. 吞吐量大约是 $8.33GPS$
3. 延迟是 $360ps$

### 4.4.2 流水线操作的详细说明

在时钟上升前, 前一个时钟周期的操作已经到了流水线寄存器的输入端, 但寄存器的状态还未改变 \
在时钟上升时, 流水线寄存器更新 \
如此循环往复

### 4.4.3 流水线的局限性

**不一致的划分**

时钟周期是按照最慢的阶段来划分的 
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.yqlvy2soh28.png)

**流水线过深, 收益反而下降**

更多寄存器产生的延迟成为了时钟周期的限制因素

### 4.4.4 带反馈的流水线系统

对于执行程序的系统, 相邻指令可能是相关的

1. 数据相关
2. 控制相关

应当正确处理反馈的影响

## 4.5 Y86-64 的流水线实现

### 4.5.1 SEQ+: 重新安排计算阶段

创建寄存器来保存上一条指令的状态从而计算 PC
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5eqgg2739kg0.webp)

这种改进称作电路重定时, 可以用来平衡流水线中各个阶段的延迟

### 4.5.2 插入流水线寄存器

流水线寄存器按照如下标号
1. $F$, 保存 PC 的预测值
2. $D$, 取指和译码阶段间, 保存最新取出的指令信息
3. $E$, 译码和执行阶段间, 保存译码的指令和寄存器读出的值
3. $M$, 执行和访存阶段间, 保存最新执行的指令的结果, 保存分支条件和目标
1. $W$, 访存和反馈路径之间, 提供给寄存器文件写, 向 PC 选择逻辑提供返回地址

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.3knykm65sz60.webp)

### 4.5.3 对信号进行重新排列和标号

在信号前面加上大写的流水线寄存器名字 \
在信号名前面加上小写的阶段首字母

注意 `SelectA` 模块
1. `call` 在访存阶段需要 valP
2. `jxx` 指令在执行阶段可能需要 valP

但这些指令都不需要 valA, 所以这两者合并

## 4.5.4 预测下一个 PC

每个时钟周期都发射一条指令, 每个时钟周期都要有一条新指令 \
所以在取出当前指令时, 需要马上取出下一条指令 \
如果取出的时条件分支指令, 那么到执行阶段才会知道是否跳转, `ret` 同理

所以对于指令进行分支预测, 当错误的时候再有错误处理 

Y86-64 采用 valP 作为预测结果, 这是非常简单的处理方式

## 4.5.5 流水线冒险

1. 数据冒险
2. 控制冒险

这与之前的反馈处理相似

**暂停**

停止流水线中的指令, 使其处于译码状态, 直到指令后 产生源操作数的指令 到写回阶段

实际操作是动态地插入 `nop` 指令, 称作插入气泡

这样会严重降低处理器效率

**转发**

保存 `e_valE`, `m_valM`, `M_valE`, `W_valM`, `W_valE` 作为转发源 \
以及两个转发目的 `valA`, `valB`

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.70wvuj6utr80.png)

**加载/使用数据冒险**

暂停处理 加载/使用冒险 的方法称为 **加载互锁** \
加载互锁和转发结合起来可以处理所有可能的数据冒险

**避免控制冒险**

处理器无法根据取指阶段的指令来确定下一条指令的地址时会出现控制冒险

在 Y86-64 中, 只有 `ret` 和 `jxx` 指令(预测错误)会造成控制冒险

在分支逻辑到了执行阶段, 发现之前预测的指令不对时, 已经有两条错误的指令进入流水线 \
此时只需要在下一个周期往译码和执行之间插入气泡, 并同时取出跳转指令之后的指令, 这样就能取消两条错误指令

### 4.5.6 异常处理

1. 在流水线中, 可能引发多个异常, 一般返回流水线最深的指令引起的异常
2. 由于错误的分支预测, 将被取消的指令产生的异常
3. 一条指令导致了异常, 后面的指令在异常指令完成前改变了部分状态

当处于访存或写回阶段中的指令导致异常时, 流水线必须禁止更新条件码寄存器或者数据内存

### 4.5.7 PIPE 各阶段的实现

**PC 选择和取指阶段**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1t1hnxiuehfk.webp)



**译码和写回阶段**









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

## 4.13

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1ki6n5ahao00.png)

## 4.14

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.f82f8hmm6x4.webp)

## 4.15

压入栈原始值, 一致

## 4.16

最终结果是把 栈指针 所指的值 $\rightarrow$ 栈指针

## 4.17

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.397ht1qwqzs0.png)

## 4.18

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.62suaq2o9780.webp)

## 4.19

```
bool need_valC = 
        icode in { IRMMOVQ, IMRMOVQ, IIRMOVQ, IJXX, ICALL };
```

## 4.20

```
word srcB = [
        icode in { IOPQ, IRMMOVQ, IMRMOVQ } : rB;
        icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
        1 : RNONE;
]
```

## 4.21

```
word dstM = [
        icode in { IMRMOVQ, IPOPQ } : rA;
        1 : RNONE;
]
```
## 4.22

`dstM` 优先级更高

## 4.23

```
word aluB = [
        icode in { IOPQ, IRMMOVQ, IMRMOVQ, IPUSHQ, 
                   IPOPQ, ICALL, IRET } : valB;
        icode in { IRRMOVQ, IIRMOVQ } : 0;
]
```

## 4.24

```
word dstE = [
        icode in { IRRMOVQ } && Cnd : rB;
        icode in { IIRMOVQ, IOPQ } : rB;
        icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
        1 : RNONE;
]
```

## 4.25

```
word mem_data = [
        icode in { IRMMOVQ, IPUSHQ } : valA;
        icode in { ICALL } : valP;
]
```

## 4.26

```
bool mem_write = 
        icode in { IRMMOVQ, IPUSHQ, ICALL };
```

## 4.27

```
word Stat = [
        imem_error || dmem_error : SADR;
        !instr_valid : SINS;
        icode == IHALT : SHLT;
        1 : SAOK;
]
```

## 4.28

1. ABC, DEF
2. AB, CD, EF
3. A, BC, D, EF
4. 五个阶段, A, B, C, D, EF, 周期时长 $100ps$

## 4.29

1. 延迟 $(300+20k)ps$, 吞吐量 $\frac {1000}{\frac{300}{k}+20} = \frac{1000k}{300+20k}$
2. $k$ 趋近于无穷大, 吞吐量为 $50$, 延迟也为无穷大

## 4.30

```
word f_stat = [
        imem_error : SADR;
        !instr_valid : SINS;
        f_icode == IHALT : SHLT;
        1 : SAOK;
]
```

## 4.31

```
word d_valA = [
        D_icode in { ICALL, IJXX } : D_valP;
        d_srcA == e_dstE : e_valE;
        d_srcA == M_dstM : m_valM;
        d_srcA == M_dstE : M_valE;
        d_srcA == W_dstM : W_valM;
        d_srcA == W_dstE : W_valE;
        1 : d_rvalA;
]
```

## 4.32


