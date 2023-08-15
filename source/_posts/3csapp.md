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


## 3.10 在机器级程序中将控制和数据结合起来

### 3.10.1 理解指针

1. 每个指针都对应一种类型 (`void *` 代表通用指针)
2. 每个指针都有一个值 (`NULL(0)` 表示空指针)
3. 指针使用 `&` 运算符创建
4. `*` 操作符用于间接引用指针
5. 数组和指针紧密联系
6. 指针强制转换不改变它的值只改变它的类型
7. 指针可以指向函数

### 3.10.2 应用: 使用 GDB 调试器

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.fyuaz0a60o.webp)

### 3.10.3 内存越界引用和缓冲区溢出

C 对数组引用不进行任何边界检查, 局部变量和状态信息都存放在栈中 \
所以对越界的数组进行写入操作会破坏掉状态信息, 程序使用状态信息的时候就会出现严重错误

缓冲区溢出可以让程序执行它原本不愿意执行的函数 \
输入给程序一个字符串, 包含一些可执行代码的字节码 (攻击代码) \
还有一些字节会用一个指向攻击代码的指针覆盖返回地址, 执行 `ret` 之后直接跳转到 攻击代码

### 3.10.4 对抗缓冲区溢出攻击

**栈随机化**:  \
过去栈的位置十分容易预测, 许多系统容易受到同一种病毒的攻击, 这种现象称为安全单一化
栈随机化的方式是在程序开始的时候, 在栈上分配一些内存
```c
int main() {
    int local;
    printf("local at %p\n", &local);
    return 0;
}
```
这段内存不能太大也不能太小, 否则会造成空间浪费或者没有足够多的地址变化

栈随机化是 **地址空间布局随机化** 的一种, 每次运行程序的不同部分 \
如程序代码, 库代码, 栈, 全局变量和堆, 这些都会被加载到内存的不同区域

暴力破解栈随机化: 插入一段 `no op` 序列, 对 `PC` 加一, 最终到达攻击代码 \
这段 `no op` 序列如果有 256 个字节长, 那么程序返回到 \
`no op` 的地址时, 就会顺着 `no op` 序列继续操作, 最终到达攻击代码 \
32 位系统栈随机化范围一般是 $2^{23}$, 只需要枚举 $\frac{2^{23}}{256}$ 次即可 \
64 位系统栈随机化范围一般是 $2^{32}$, 只需要枚举 $\frac{2^{32}}{256}$ 次即可(很多)

**栈破坏检测**: \
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5wuhh5wgw740.webp)
通过设置金丝雀值(随即生成), 返回时检测金丝雀值是否被改变从而确认栈是否异常

**限制可执行代码区域**: \
限制哪部分内存可以存储可执行代码

### 3.10.5 支持变长栈帧

在函数中使用变长数组的时候, 外部函数无法确定应该分配多少栈空间 \
`%rbp` 为基指针, 在函数内部修改栈指针分配空间之后, 再把栈指针设置成基指针

## 3.11 浮点代码

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.3w4ivs5wv880.webp)

### 3.11.1 浮点传送和转换操作

保存在内存中的数据 $M_{32}$ 和 $M_{64}$, 保存在 $XMM$ 寄存器中的数据 $X$

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.592c9ve19r80.png)

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.459e70calhu0.png)

### 3.11.2 过程中的浮点代码

`%xmm0 ~ %xmm7` 存储 7 个参数, 多余的参数可以使用栈空间 \
`%xmm0` 为返回值 \
所有的 `xmm` 寄存器都是调用者保存的, 被调用者可以直接覆盖

当参数中整数和浮点数混合时, 分别使用两套寄存器保存 \
第一个整数参数, 第二个..... 保存在 `%rdi`, `%rsi`..... \
第一个浮点数参数, 第二个.... 保存在 `%xmm0`, `%xmm1` .....

### 3.11.3 浮点运算操作

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.3v0ud7q19qm0.png)

### 3.11.4 定义和使用浮点常数

浮点数不能以立即数形式出现, 需要保存在内存中, 代码再读入

### 3.11.5 在浮点代码中使用位级操作

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.61garb5k7ug0.webp)

### 3.11.6 浮点比较操作

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.72g4fdc2qa40.webp)

浮点比较指令有三个条件码
1. 零标志位 `ZF`
2. 进位标志位 `CF`
3. 奇偶标志位 `PF`, 两个操作数任意一个是 `nan` 该位为 1

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6x72lf3o49s0.webp)

### 3.11.7 对浮点代码的观察讨论

类似整数的寄存器和汇编代码风格

## 3.12 小结

1. 了解机器, 数据类型, 指令集
2. 程序如何将数据存储在不同的区域中
3. 机器级程序和汇编代码表示



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

## 3.46

1. ![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5v51kxh3oa80.webp)
2. ![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2gowpyq4xsw0.webp)
3. 试图返回到 `0x00 00 00 00 00 40 00 34`
4. `%rbx` 的值被破坏了
5. - `malloc(strlen(buf) + 1)`
   - 检测返回值是否为 `NULL`

## 3.47

1. $2^{13}$
2. $\frac{2^{13}}{128} = 64$

## 3.48

1. `buf -> %rsp`, `v -> 24(%rsp)`
2. `buf -> 16(%rsp)`, `v -> 8(%rsp)`, `金丝雀 -> 40(%rsp)`

在有保护的代码中, 数组放在上面可以更好防护

## 3.49

1. `leaq` 计算 $8n+22$, 与上 16 之后 \
   如果 $n$ 为奇数, 那么结果为 $8n+8$, $n$ 为偶数则为 $8n+16$
2. 整除 8 (向上取整)
3. ![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6ok0vhfnueg0.webp)
4. $s_2$ 是 $s_1 - e_1 - e_2 - 8n$, 是个相对于 $s_1$ 8 对齐的位置 \
   $p$ 是 $s_1 - e_1 - 8n$, 本身就是 8 对齐的位置

## 3.50

$d$, $i$, $l$, $f$

## 3.51

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6argxj9soeg0.png)

## 3.52

1. `%xmm0`, `%rdi`, `%xmm1`, `%esi`
2. `%edi`, `%rsi`, `%rdx`, `%rcx`
3. `%rdi`, `%xmm0`, `%esi`, `%xmm1`
4. `%xmm0`, `%rdi`, `%xmm1`, `%xmm2`

## 3.53

根据加法的交换性, 中间两项可以交换
```cpp
double funct1(int p, float q, long r, double s);
double funct1(int p, long q, float r, double s);
```

## 3.54

`x * y - w / z`

## 3.55

指数字段是 1028, 减去偏移量 1023, 指数为 5 \
$1.0 \times 2^5 = 32.0$

## 3.56

1. 常数 `LC1` 是除了符号位低位全为 1 的掩码, 与操作去掉符号位, 相当于 `fabs(x)`
2. `x = 0.0`
3. 符号位异或 1, 相当于 取反 `-x`

## 3.57

```cpp
double funct3(int *ap, double b, long c, float *dp) {
    int a = *ap;
    float d = *dp;
    if (a < b) return c * d;
    else return c + 2 * d;
}
```

## 3.58

```cpp
long decode(long x, long y, long z) {
    int res = y - z;
    return (res << 63 >> 63) ^ (x * res);
}
```

## 3.59

设 $x_{63}$, $y_{63}$ 为 $x$, $y$ 的符号位 \
$ux = x + x_{63}2^{64}$, 同理 $uy = y + y_{63}2^{64}$ \
$ux \times uy = xy + (x_{63}y + y_{63}x) 2^{64} + x_{63}y_{64}2^{128}$ 最后一项溢出舍去

$xy = ux\times uy - (x_{63}y + y_{63}x)2^{64}$ \
设 $s_x = -1$ 当 $x_{63}$ 为 1 时, $s_x = 0$ 当 $x_{63}$ 为 0 时, $s_y$ 同理 \
$xy = ux\times uy + (s_xy + s_yx)2^{64}$
```
# void store_prod(int128_t* dest, int64_t x, int64_t y)
# dest in %rdi, x in %rsi, y in %rdx
store_prod:
  # 这两行把 y 转换成 int_128
  movq %rdx, %rax     
  cqto # 此时 %rax 存着 y 的高位算数补全结果, 即 sy
  # 这两行先把 x 算数右移, 得到 sx
  movq %rsi, %rcx     
  sarq $63, %rcx

  # 这三行计算 (sx * y + sy * x)
  imulq %rax, %rcx    
  imulq %rsi, %rdx   
  addq %rdx, %rcx   
  # 这行计算 ux * uy, 低 64 位存储在 %rax, 高 64 位存储在 %rdx
  mulq %rsi        

  在高 64 位直接加上 (sx * y + sy * x)
  addq %rcx, %rdx

  # 存储
  movq %rax, (%rdi)   # set lower 64bits
  movq %rdx, 8(%rdi)  # set higher 64bits
  ret
```

## 3.60

```cpp
long loop(long x, int n) {
    long result = 0;
    long mask;
    for (mask = 1; mask != 0; mask <<= n) 
        result |= x & mask;
    return result;
}
```

## 3.61

```
# long cread(long *xp)
# xp in %rdi
cread:
  movq (%rdi), %rax
  testq %rdi, %rdi
  movl $0, %edx
  cmove %rdx, %rax
  ret
```
第一行直接取数据, 可能会出现空指针错误

```
cread_alt:
  movl $0, %eax
  testq %rdi, %rdi
  cmovne (%rdi), %rax
```
```cpp
long cread_alt(long *xp) {
    return (!xp ? 0 : *xp);
}
```
第一个分支可以合法计算, 第二个分支当指针不等于零的时候才会取数据 \
这样不符合之前标准的条件传送的标准写法, 是一种取巧的做法

```cpp
v = then-expr;
ve = else-expr;
t = test-expr;
if (!t) v = ve;
```

## 3.62

```cpp
typedef enum { MODE_A, MODE_B, MODE_C, MODE_D, MODE_E } mode_t;
long swith(long *p1, long *p2, mode_t action) {
    long res = 0;
    switch (action) {
    case MODE_A:
        res = *p2;
        *p2 = *p1;
        break;
    case MODE_B:
        res = *p1 + *p2;
        *p1 = res;
        break;
    case MODE_C:
        *p1 = 59;
        res = *p2;
        break;
    case MODE_D:
        *p1 = *p2;
        res = 27;
    case MODE_E:
        res = 27;
        break;
    default:
        res = 12;
        break;
    }
    return res;
}
```

## 3.63

```cpp
long switch_(long x, long n) {
    long result = x;
    switch(n) {
    case 60:
    case 62:
        result = 8 * x;
        break;
    case 63:
        result = x >> 3;
        break;
    case 64:
        x = x * 15;
    case 65:
        x = x * x;
        break;
    default:
        result = x + 73;
        break;
    }
    return result;
}
```

## 3.64

1. $D[i][j][k] = x_D + iST + jT + k$
2.
```
store_ele:
  # t1 = 13j
  leaq (%rsi, %rsi, 2), %rax
  leaq (%rsi, %rax, 4), %rax
  # t2 = 65i
  movq %rdi, %rsi           
  salq $6, %rsi           
  # t3 = 65i + 13j + k
  addq %rsi, %rdi        
  addq %rax, %rdi       
  addq %rdi, %rdx      
  # t1 = *(A + 8 * t3)
  movq A(,%rdx,8), %rax
  movq %rax, (%rcx)   
  movl $3640, %eax   
  ret
```
```
RST * 8 = 3640;
ST = 65;
T = 13;
```
得到结果
```
R = 7;
S = 5;
T = 13;
```

## 3.65

1. `%rdx`
2. `%rax`
3. `15`

## 3.66

1. `#define NR(n) (3 * n)`
1. `#define NC(n) (4 * n + 1)`

## 3.67

1. 
```
104  +------------------+
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
 64  +------------------+ <-- %rdi
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
 32  +------------------+
     |         z        |
 24  +------------------+
     |        &z        |
 16  +------------------+
     |         y        |
  8  +------------------+
     |         x        |
  0  +------------------+ <-- %rsp
```

2. 传递了 `%rdi` 存储着新结构体应该放的位置
3. 由于调用 `call`, 栈指针减去 8, 通过栈指针访问 `strA`, 通过 `%rdi` 访问 `strB`
4. 返回 `eval` 之后, 就直接使用 `%rsp + 偏移量` 的方式来访问 `r`
5. 
```
104  +------------------+
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
     |                  |
 88  +------------------+
     |        z         |
 80  +------------------+
     |        x         |
 72  +------------------+
     |        y         |
 64  +------------------+ <-- %rdi(eval pass in)
     |                  |  \
     |                  |   -- %rax(process pass out)
     |                  |
     |                  |
     |                  |
     |                  |
 32  +------------------+
     |         z        |
 24  +------------------+
     |        &z        |
 16  +------------------+
     |         y        |
  8  +------------------+
     |         x        |
  0  +------------------+ <-- %rsp in eval
     |                  |
 -8  +------------------+ <-- %rsp in process
```
6. 调用者查找空间, 给被调用者传递地址, 被调用者将数据存储在此位置并返回该地址

## 3.68

1. `p->y` 存储在 `184(%rdi)`, $176 < 4AB \le 184$ 故 $AB = 45 或 46$
2. `q->t` 存储在 `8(%rsi)`, $4 < B \le 8$
3. `q->u` 存储在 `32(%rsi)`, 数组 `s` 从 12 开始, 到 32 结束, $12 < 2A \le 20$

故 `AB` 有唯一解: `A = 9`, `B = 5`

## 3.69

`CNT = 7`
```cpp
typedef struct {
    long idx;
    long x[4];
} a_struct;
```

`first` 在 `(%rdi)`, `last` 在 `288(%rdi)` \
`bp->a[i]` 是 `(%rdi + 40i + 8)` 说明 `first` 占了八个字节, 每个 `a_struct` 大小为 40 \
`(288 - 8) / 40 = 7`, `CNT = 7`

此时 `%rdx` 存储着 `ap->idx`, `%rcx` 是 `8 + (8 + %rsi + 40i) + 8 * idx` \
第一个 8 只能是 `idx`, 第二个括号内是 `ap` 的地址, 第三项就是 寻找 `x`, 显然类型为 `long`

## 3.70

1. 0, 8, 0, 8
2. 16
3. 
```cpp
union ele {
  struct {
    long *p;
    long y;
  } e1;
  struct {
    long x;
    union ele *next;
  } e2;
};

void proc(union ele *up) {
  up->e2.x = *(up->e2.next->e1.p) - up->e2.next->.e1.y
}
```

## 3.71

```cpp
#include <stdio.h>
#include <assert.h>

void good_echo() {
    char buf[11];
    while (1) {
        char *p = fgets(buf, 11, stdin);
        if (!p) break;
        printf("%s", p);
    }
}
```

## 3.72

1. 当 `n` 为偶数, $s_2 = s_1 - (8n + 24)$, 当 `n` 为奇数, $s_2 = s_1 - (8n + 16)$
2. 大于等于 $s_2$ 的 16 的最小倍数
3. `n` 为偶数, $e_1 + e_2 = 24$, `n` 为奇数, $e_1 + e_2 = 16$ \
   如果 $e_1=0$, 那么 $s_2 = p, e_2 = 0$ 与第一条不符合 \
   $e_1$ 最小值为 1, 此时 $n$ 为奇数 \
   $e_1$ 最大值为 24, 此时 $n$ 为偶数

## 3.73

```
find_range:
    vxorps %xmm1, %xmm1, %xmm1
    vucomiss %xmm1, %xmm0
    jp .P
    ja .A
    jb .B
    je .E
.A:
    movl $2, %eax
    jmp .Done
.B:
    movl $0, %eax
    jmp .Done
.E:
    movl $1, %eax
    jmp .Done
.P:
    movl $3, %eax
.Done
```

## 3.74

```
vxorps %xmm1, %xmm1, %xmm1
movq $1, %rax
movq $2, %r8
movq $0, %r9
movq $3, %r10
vucomiss %xmm1, %xmm0
cmovaq %r8, %rax
cmovbq %r9, %rax
cmovpq %r10, %ra
```

## 3.75

1. 按顺序来, `%xmm0` 第一个复数的实部, `%xmm1` 第一个复数的虚部 \
   `%xmm2` 第二个复数的实部, `%xmm3` 第二个复数的虚部 
2. `%xmm0`, `%xmm1` 作为返回值的实部和虚部
