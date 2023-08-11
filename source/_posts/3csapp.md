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

# Chapter 3: 程序的机器级表示

编译器把高级语言代码换算成汇编语言, 之后再经过汇编器和编译器生成机器代码

学习汇编语言的好处: 理解编译器的优化能力, 分析代码中隐含的低效率

## 3.1 历史观点

$Intel$ 处理器俗称 $x86$, 本节介绍芯片指令集发展的历史

## 3.2 程序编码

```bash
gcc -Og -o p p1.c p2.c
```
`gcc` 表示 `GCC C` 编译器 \
编译选项 `-Og` 告诉编译器使用符合原始代码结构的机器代码的优化等级 \
(使用较高级别的优化会使得代码严重变形, 不利于梳理汇编和源代码的关系, 不利于学习) \
`-o p` 表示生成可执行文件 `p` 

实际上 `gcc` 调用了一整套程序, 包括预处理器, 编译器, 汇编器, 链接器

## 3.2.1 机器级代码

对于机器级编程, 有两种抽象尤为重要
1. 由指令集架构来定义机器级程序的格式和行为
2. 使用虚拟地址, 也就是一个非常大的字节数组
操作系统负责把虚拟地址翻译成实际的物理地址

一些在 C 中对程序员隐藏的寄存器状态:
1. `%rip` 表示 PC, 给出将要执行的下一条指令在内存中的地址
2. 整数寄存器包含 16 个命名的位置, 存储 64 位的值
3. 条件码寄存器保存最近执行的算数或逻辑指令的状态信息, 来实现条件变化
4. 一组向量寄存器可以存放一个或多个整数或浮点值

## 3.2.2 代码示例

```bash
gcc -Og -S mstore.c
```
`-S` 参数可以生成一个 `.s` 文件, 包含汇编代码

```bash
gcc -Og -c mstore.c
```
`-c` 参数编译并汇编此代码, 生成 `.o` 文件

```bash
gdb mstore.o
(gdb) x/14xb multstore
```
展示程序的字节表示

```bash
objdump -d mstore.o
```
反汇编器, 根据机器码生成相应的汇编代码

关于机器代码和反汇编表示的特性:
1. `x86-64` 的指令长度 x 字节, `1 <= x <= 15`, 操作越常用, 所需字节数越少
2. 设计指令格式的方式是: 从某个给定位置开始, 可以将字节唯一解码成机器指令
3. 反汇编器不需要访问源代码
4. 反汇编器产生的汇编代码跟编译器产生的汇编代码有所区别

### 3.2.3 关于格式的注解

所有以 `.` 开头的行都是指导汇编器和链接器工作的伪指令, 通常可以忽略

`ATT` 汇编格式与 `Intel` 有所不同, 本书使用 `ATT` 格式

## 3.3 数据格式

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.7jkwgbr684w0.webp)

由于系统从 16 位发展而来, 所以 16 位被称为 字 \
32 位被称为双字, 64 位为四字

大多数 GCC 生成的汇编指令代码都有一个字符后缀, 表明操作数的大小

`movb` 传送字节, `movw` 传送字, `movl` 传送双字, `movq` 传送四字

## 3.4 访问信息

一组 `x86-64` 的 CPU 包含一组 16 个存储 64 位的 **通用目的寄存器**, 存储整数和指针

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5js7ywj3kcw0.webp)

指令可以对这些寄存器的低位字节进行访问

### 3.4.1 操作数指示符

大多数指令由一个或多个操作数, 操作数被分为三种类型
1. 立即数: 表示常数, `$` 后加 C 格式的常数
2. 寄存器: 表示某个寄存器的内容, 用 $r_a$ 表示任意寄存器, $R[r_a]$ 表示它的值
3. 内存引用: $M_b[Addr]$ 表示从 $Addr$ 开始的 $b$ 个字节的引用, 通常省略下标 $b$

多种不同的寻址模式
1. $\$x$ 即 $x$
2. $r_a$ 即 $R[r_a]$
3. $Imm(r_a, r_i, s)$ 即 $M[Imm + R[r_a] + R[r_i] * s)]$ 
    - $r_a$ 基址寄存器
    - $r_i$ 变址寄存器
    - $s$ 比例因子, 值只能为 1, 2, 4, 8

### 3.4.2 数据传送指令

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5h8n3yati90.png)
> 注意 `movl` 指令会把最高四个字节设置为 0, 这是惯例

> 常规的 `movq` 只能以表示为 32 位补码数字的立即数作为操作数, 并进行符号扩展得到 64 位值 
> `movabsq` 能以任意 64 位立即数作为操作数, 并且只能以寄存器作为目的

传送指令的两个操作数不能都是内存引用, 需要先将源内存引用传送到寄存器, 再传送到目标内存引用

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.69cyhxhnmc00.png)

数据传送改变寄存器的方式
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2z8sg6a88xk0.webp)

### 3.4.3 数据传送示例

代码和汇编的互相转化

### 3.4.4 压入和弹出栈数据

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.443ubut3zz40.webp)

根据惯例, 栈顶地址最小

## 3.5 算数和逻辑操作

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5nkw0oiuw380.png)

### 3.5.1 加载有效地址

`leaq` 的第一个操作数是内存引用, 第二个操作数是寄存器 \
效果就是把寄存器存储上第一个操作数

`leaq` 有时与地址无关, 用它来实现简单的乘法操作
```cpp
long scale(long x, long y, long z) {
    long t = x + 4 * y + 12 * z;
    return t;
}
```
```
scale:
    leaq (%rdi, %rsi, 4), %rax  // x + 4 * y
    leaq (%rdx, %rdx, 2), %rdx  // z + 2 * z = 3z
    leaq (%rax, %rdx, 4), %rax  // x + 4 * y + 4 * 3z = x + 4y + 12z
```

### 3.5.2 一元和二元操作

第二组的操作是一元操作 `INC (寄存器, 内存引用)` \
第三组是二元操作, `subq (立即数, 寄存器, 内存引用), (寄存器, 内存引用)`

### 3.5.3 移位操作

先给出移位量(必须使用八位寄存器, 通常是 `%cl`), 第二项给出要移位的数

### 3.5.4 讨论

函数会返回寄存器 `%rax` 的值

### 3.5.5 特殊的算数操作

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.7drv1ofepjo0.webp)

128 位的数称为 八字(oct word)

乘法指令:  \
`mul` 无符号乘法, `imul` 有符号乘法 \
这些是单操作数的指令, 跟之前的 `mul` 有所不同 \
`imulq` 和 `mulq` 都要求 一个参数必须在 `%rax` 中, 另一个参数是操作数 \
结果存储在 `%rdx` (高 64 位, `%rax` (低 64 位)

除法指令:
`div` 无符号除法, `idiv` 有符号除法 \
被除数存储在 `%rdx` (高 64 位, `%rax` (低 64 位) 中, 除数是操作数 \
商存储在 `%rax` 中, 余数存储在 `%rdx` 中

被除数如果是 64 位的值, 存储在 `%rax` 中, `%rdx` 应该设置成 全0 或者 `%rax` 的符号位 \
设置符号位可以由 `cqto` 指令来完成

## 3.6 控制

机器代码提供两种方式来实现有条件的行为:
1. 条件控制
2. 条件传送

### 3.6.1 条件码

CPU 不仅维护整数寄存器, 还维护一组单个位的 **条件码寄存器**

1. `CF`: 进位标志, 最近的操作是否产生进位
2. `ZF`: 零标志, 最近的操作得出的结果为 0
3. `SF`: 符号标志, 最近的操作得出的结果为负数
4. `OF`: 溢出标志, 最近的操作导致一个补码溢出

> `leaq` 指令不改变条件码, 图 `3-10` 中所有的操作都会改变条件码

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.28ycgvvihkp.webp)

### 3.6.2 访问条件码

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.36ttwwgcbb40.png)

`set` 命令把一个字节设置成 0 或者 1

### 3.6.3 跳转指令

跳转由两种方法
1. 直接跳转, `jmp .L1`
2. 间接跳转, `jmp *%rax`

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.55kb1waancc0.webp)

### 3.6.4 跳转指令的编码

跳转指令编码方式:
1. PC相对: 目标指令的地址与紧跟在跳转指令后面那条指令的地址之间的差作为编码
2. 绝对地址: 用四个字节直接指定目标

### 3.6.5 用条件控制来实现条件分支

```cpp
if (text_expr) 
    then-statement;
else
    else-statement;
```

```
t = test-expr
if (!t)
    goto false;
then-statement;
goto done;

false:
    else-statement;
done:
```

### 3.6.6 用条件传送来实现条件分支

处理器使用流水线来获得高性能, 通过重叠连续指令的步骤获得高性能 \
当机器遇到条件分支的时候, 会使用分支预测逻辑来猜测跳转指令是否会执行 \
错误预测一个跳转会让处理器丢掉跳转指令后所有的工作, 重新开始

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6v82v2jsq400.png)

条件传送指令可以根据操作数来推断类型, 所以不用限定类型

```cpp
v = test-expr ? then-expr : else-expr;
```

```cpp
    if (!test-expr)
    goto false;
    v = then-expr;
    goto done;
false:
    v = else-expr;
done:
```

```cpp
v = then-expr;
ve = else-expr;
t = test-expr;
if (!t) v = ve;
```

条件传送的缺点:
1. 两个表达式需要提前求值, 如果在条件不满足的情况下求值可能会出现错误
2. 表达式如果需要很长时间来计算, 反而得不偿失

### 3.6.7 循环

`do-while` 循环转换后
```cpp
loop:
    body_statement;
    t = test_expr;
    if (t) goto loop;
```

`while` 循环法一
```cpp
    goto test;
loop:
    body_statement;
test:
    t = test_expr;
    if (t) goto loop;
```

`while` 循环法二
```cpp
t = test_expr;
if (!t) goto done;

loop:
    body_statement;
    t = test_expr;
    if (t) goto loop;
done:
```

`for` 循环:
```cpp
for (init_expr; test_expr; update_expr)
    body_statement;
```
转换成
```cpp
init_expr;
while (test_expr) {
    body_statement;
    update_expr;
}
```

### 3.6.8 `switch` 语句

`switch` 语句的优点:
1. 提高代码可读性
2. 使用跳转表使得数据结构实现高效, 执行 `switch` 的语句和情况呢数量无关

新的运算符 `&&` 得到指向代码位置的指针

## 3.7 过程

假设过程 $P$ 调用过程 $Q$, 执行之后回到 $P$, 包括如下机制
1. 传递控制: 进入 $Q$ 时, PC 要设置成 $Q$ 的起始地址, 返回时要设置成调用 $Q$ 下面指令的地址
2. 传递数据: $P$ 向 $Q$ 传递若干参数, $Q$ 向 $P$ 返回一个值
3. 分配和释放内存: $Q$ 为变量分配空间, 返回前释放

### 3.7.1 运行时栈

栈指针 `%rsp` 指向栈顶元素, 用 `pushq` 和 `popq` 入栈出栈 \
减小栈指针可以为一组没有初始值的变量分配空间, 增大栈指针可以释放空间

过程所需空间超出寄存器能存放的大小时, 就会在栈上分配空间, 这个部分称为 **栈帧**

$P$ 调用 $Q$ 时, 会把返回地址压入栈中, $Q$ 返回时从这个地址开始继续执行

### 3.7.2 转移控制

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.78ygtm5gl3c0.webp)

### 3.7.3 数据传送

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.53r8048unb80.webp)

大于六个的参数会放在栈中, 第七个参数位置是 `(%rsp)`, 第八个参数位置是 `8(%rsp)` \
无论多出的参数是什么类型, 都会在栈中使用 8 位存储

### 3.7.4 栈上的局部存储

局部数据必须放在内存中的常见情况
1. 寄存器不够用
2. 使用取址运算符
3. 局部变量是数组或结构体

### 3.7.5 寄存器中的局部存储空间

`%rbx`, `%rbp`, `%r12 ~ %r15` 是 **被调用者保存寄存器**, 调用 $Q$ 时这些寄存器的值不变 \
`%rsp` 是栈指针 \
其他寄存器是 **调用者保存寄存器**

## 3.8 数组分配与访问

### 3.8.1 基本原则

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5tsehwm671s0.png)

### 3.8.2 指针运算

介绍 $C$ 中的指针运算

### 3.8.3 嵌套的数组

对于 $T D[R][C];$, 数据类型 $T$ 的大小为 $L$, $D[i][j]$ 的内存地址为 
$$
\&D[i][j] = x_D + L(C \times i + j)
$$

### 3.8.4 定长数组

程序使用常数作为数组的维度或者缓冲区的大小时, 最好设置一个常量 \
使得修改时不用处处修改反而只需修改一步

定长数组在编译器优化的时候可以使用相对地址而不是绝对地址, 这样效率更高

### 3.8.5 变长数组

以变量作为数组维度大小

这样的数组不能使用移位和加法来计算乘法, 只能直接使用乘法运算, 效率低

## 3.9 异质的数据结构

### 3.9.1 结构体

结构体中的变量在内存中紧密排列在一起

### 3.9.2 联合

联合的总大小等于它的最大字段的大小

1. 事先知道一个数据结构中两个不同字段的使用时互斥的 \
2. 访问不同数据类型的位模式

### 3.9.3 数据对齐

结构体的大小一定是对齐值的整数倍 \
对齐值是结构体里占用字节数最大的类型的字节数, 结构体则继续递归定义 \
结构体每个数据起始的字节位置一定是本数据的类型所占字节数的整数倍, 结构体除外








# 练习

## 3.1

根据规则算数

## 3.2 

根据操作数中字数最小的来确定

## 3.3

1. `()` 中只能是四个字的数据
2. `%rax` 是四个字, 不是两个字
3. 两个操作数不能都是内存引用
4. 没有寄存器叫 `%sl`
5. 立即数不能做 `mov` 的第二个操作数
6. 从 32 位复制到 64 位, 可能会出现数据截断的问题
7. `%si` 是一个字而不是字节

## 3.4

强制类型转换:
1. 先全读进来, 该扩展的扩展, 
2. 截断或者直接赋值


1. `char -> int` 一字节变双字
    ```
    movsbl (%rdi), %eax
    movl %eax, (%rsi)
    ```
2. `char -> unsigned`同上
    ```
    movsbl (%rdi), %eax
    movl %eax, (%rsi)
    ```
3. `unsigned char -> long` 
    ```
    movzbl (%rdi), %eax
    movl %eax, (%rsi)
    ```
4. `int -> char` 截断
    ```
    movl (%rdi), %eax
    movb %al, (%rsi)
    ```
5. `unsigned -> unsigned char`
    ```
    movl (%rdi), %eax
    movb %al, (%rsi)
    ```
6. `char -> short`
    ```
    movsbw (%rdi), %ax
    movw %ax, (%rsi)
    ```
## 3.5

```cpp
void decode1(long *xp, long *yp, long *zp) {
    int a = *xp, b = *yp, c = *zp;
    *yp = a, *zp = b, *xp = c;
}
```

## 3.6 

算数题

## 3.7

`5 * x + 2 * y + 8 * z`

## 3.8

算数题

## 3.9

```
movq %rdi, %rax
sal $4, $rax
movl %esi, %ecx
sar %cl, %rax
```

## 3.10

```cpp
long arith2(long x, long y, long z) {
    long t1 = x | y;
    long t2 = t1 >> 3;
    long t3 = ~t2;
    long t4 = z - t3;
    return t4;
}
```

## 3.11

1. 把 `%rdx` 设置为 0
2. `movq $0, %rdx`
3. `xorq %rdx, %rdx` 使用三个字节 \
    `movq $0, %rdx` 使用七个字节 (立即数占用四个)

## 3.12

```
uremdiv:
    movq %rdx, %r8
    movq %rdi, %rax
    movl $0, $edx
    divq %rsi
    movq %rax, (%r8)
    movq %rdx, (%rcx)
    ret
```

## 3.13

1. `int`, `<`
2. `short`, `>=`
3. `unsigned char`, `<=`
4. `long / unsigned long / 指针`, `!=`

## 3.14

1. `long`, `<`
2. `short / unsigned short`, `==`
3. `unsigned char`, `>`
4. `int`, `<`

## 3.15

1. 4003fe
2. 400425
3. 400543, 400545
4. 400560

## 3.16

1.  
```cpp
void cond(long a, long *p) {
    if (p == 0)
        goto L1;
    if (*p >= a)
        goto L1;

    *p = a;

L1:
    return;
}
```

2. 源代码中有 `&&`, 如果 第一个条件为假, 直接跳过

## 3.17

```cpp
long gotodiff_se_alt(long x, long y) {
    long result;
    if (x < y)
        goto L1;
    result = x - y;
    return result;
L1:
    result = y - x;
    return result;
}
```
如果没有 `else` 语句, 之前的那种会简洁一些

## 3.18

```cpp
long test(long x, long y, long z) {
    long val = x + y + z;
    if (x < -3) {
        if (y < z)
            val = x * y;
        else
            val = y * z;
    } else if (x > 2) 
        val = x * z;

    return val;
}
```

## 3.19

1. $2 \times (31 - 16) = 30$
2. $16 + 30 = 46$

## 3.20

1. `/` 
2.  
```cpp
long arith(long x) {
    int tmp = 7 + x;
    if (x >= 0) tmp = x;
    return tmp >> 3;
}
```

## 3.21

```cpp
long test(long x, long y) {
    long t1 = 8 * x;
    if (y > 0) { 
        if (x < y) t1 = y - x;
        else t1 = y - x;
    } else if (y <= -2) t1 = x + y;
    return t1;
}
```

## 3.22

1. 13
2. 20

## 3.23

1. `%rax`, `%rcx`, `%rdx`
2. $p$ 永远指向 $x$, 所以编译器优化 `x += 1 + y`, `leaq 1(%rcx, %rax), %rax`
3. 略去

## 3.24

```cpp
long loop_while(long a, long b) {
    long result = 1;
    while (a < b) {
        result *= a + b;
        a += 1;
    }
    return result;
}
```
## 3.25

```cpp
long loop_while2(long a, long b) {
    long result = b;
    while (b > 0) {
        result *= a;
        b -= a;
    }
    return result;
}
```

## 3.26

1. `while` 的第二种方法
2. 
```cpp

long fun_a(unsigned long x) {
    long val = 0;
    while (x != 0) {
        val ^= x;
        x >>= 1;
    }
    return val & 1;
}
```
3. 返回 $x$ 中 1 的个数的奇偶性

## 3.27

```cpp
long fact_for(long n) {
    long i = 2;;
    long result = 1;
    if (i > n) goto done;
loop:
    result *= i;
    i++;
    if (i <= n) goto loop;
done:
    return result;
}
```

## 3.28

1. 
```cpp
long fun_b(unsigned long x) {
    long val = 0;
    long i;
    
    for (i = 64; i != 0; i--) {
        val = (val + val) | (x & 1);
        x >>= 1;
    }
    return val;
}
```
2. 初始测试不可能失败
3. 反转 $x$ 中的位

## 3.29

1. `i++` 未执行
2.  
```cpp
long sum = 0;
long i = 0;
while (i < 10) {
    if (i & 1)
        goto update;
    sum += i;
update:
    i++;
}
```

## 3.30

1. -1, 0, 1, 2, 4, 5, 7
2. 0 和 7, 2 和 4

## 3.31

```cpp
void switcher(long a, long b, long c, long *dest) {
    long val;
    switch (a) {
            
        case 5:
            c = b ^ 15;
        case 0:
            val = 112 + c;
            break;
        case 2:
        case 7:
            val = (b + c) << 2;
            break;
        case 4:
            val = a;
            break;
        default:
            val = b;
    }
    *dest = val;
    return;    
}
```

## 3.32

模拟

## 3.33

1. 把 $a$ 强制转换成 `long`, 所以 $a$ 是 `int`, $u$ 是 `long *`
2. 把 $b$ 的低位字节 放到 $v$ 上, 所以 $v$ 是 `char *`
3. 返回值是 6, `sizeof(a)` 是 4, 所以 `sizeof(b)` 是 2, $b$ 是 `short`

```cpp
int procprob(int a, short b, long *u, char *v) {
    *u += a;
    *v += b;
    return sizeof(a) + sizeof(b);
}
```

## 3.34

1. `a0 ~ a5`
2. `a6`, `a7`
3. 被调用者保存寄存器不够用

## 3.35

1. `%rbx` 保存 $x$ 的值
2. 
```cpp
long rfun(unsigned long x) {
    if (x == 0)
        return x;
    unsigned long nx = x >> 2; 
    long rv = rfun(nx);
    return x + rv;
}
```

## 3.36

算数

## 3.37

1. `leaq 2(%rdx), %rax`
2. `movw 6(%rdx), %ax`
3. `leaq (%rdx, %rcx, 2), %rax`
4. `movw 2(%rdx, %rcx, 8), %ax`
5. `leaq -10(%rdx, %rcx, 2), %rax`

## 3.38

$M=5$, $N=7$

## 3.39
$M=4, N=16$ 
- $\&A[i][0] = x_a + 64i$
- $\&B[0][k] = x_b + 4k$
- $\&B[N][k] = x_b + 64N + 4k = x_b + 1024 + 4k$

## 3.40

```cpp
void fix_set_diag(fix_matrix A, int val) {
    int i = 0;
    int N = 16;
    int iend = N * (N + 1);

    do {
        *(A + i) = val;
        i += (N + 1);
    } while (i != iend);

}
```

1. `A[k][k] = *(A + k * N + k)`, `A[k + 1][k + 1] = *(A + (k + 1) * N + k + 1` \
   $i_{old} = k * N + k$, $i_{new} = (k + 1) * N + k + 1$, $delta = N + 1$
2. $i_{end} = N * (N - 1) + (N - 1) + delta = N^2 - 1 + delta = N * (N + 1)$

## 3.41

- 8, 12, 16
- 24
- 
```cpp
sp->s.x = sp->s.y;
sp->p = &(sp->s.x);
sp->next = sp;
```

## 3.42

```cpp
struct ELE {
    long v;
    struct ELE *p;
};
long fun(struct ELE *ptr) {
    long tmp = 0;
    while (ptr) {
        tmp += ptr->v;
        ptr = ptr->p;
    }
    return tmp;
}
```
计算单链表中所有元素的值

## 3.43

1. `short char* int* int char`
2.
```
movw 8(%rdi), %ax
movw %ax, (%rsi)

addq $10, %rdi
movq %rdi, (%rsi)

movq %rdi (%rsi)

movq (%rdi), %rax
movl (%rdi, %rax, 4), %eax
movl %eax, (%rsi)

movq 8(%rdi), %rax
movb (%rax), %al
movb %al, (%rsi)

```

## 3.44

算数

## 3.45

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.12ivr9wu089c.webp)
56 个字节大小 \
一个策略是降序排列
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5tahd0a6qho0.webp)
40 个字节大小
