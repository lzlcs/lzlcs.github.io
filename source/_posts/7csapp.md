---
title: CSAPP 学习笔记 (第七章)
date: 2023-08-29 17:23:00
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# Chapter 7: 链接

链接是将各种代码和数据片段收集并组合成一个单一文件的过程 \
链接可以在 编译时, 加载时, 运行时进行, 通常是由链接器来执行

## 7.1 编译器驱动程序

```
gcc -Og -o prog main.c sum.c
```

详细步骤

1. `cpp` 预处理器, 将 `.c` 翻译成 `ASCII` 码的中间文件 `.i`
2. `ccl` C 编译器, 将 `.i` 翻译成 汇编语言文件 `.s`
3. `as` 汇编器, 将 `.s` 翻译成可重定位目标文件 `.o`
4. `ld` 链接器, 将 若干 `.o` 文件和必要的系统文件组合在一起, 产生可执行目标文件
5. `shell` 调用操作系统中的 加载器 `loader`, 把可执行文件的代码和数据复制到内存, 开始执行

```
cpp main.c ./tmp/main.i
ccl ./tmp/main.i -Og -o ./tmp/main.s
as -o ./tmp/main.o ./tmp/main.s
ld -o prog [system files] ./tmp/main.o ./tmp/sum.o
./prog
```

## 7.2 静态链接

可重定位目标文件由各种不同的代码和数据节组成 \
指令在一节中, 初始化了的全局变量在一节中, 未初始化的变量又在另外一节中

`ld` 这样的静态链接器
1. 输入: 一组可重定位目标文件
2. 输出: 完全链接的可执行目标文件

链接器的任务
1. 符号解析: 源文件中用到的函数, 全局变量, `static` 静态变量都是符号 \
   链接器找到这些符号的定义, 并将他们关联起来
2. 重定位: 编译器和链接器生成的都是 地址从零开始 的代码和数据 \
   链接器把符号定义和内存位置关联起来, 从而重定位这些节 \
   之后 修改所有对这些符号的引用到新的位置

## 7.3 目标文件

1. 可重定位目标文件: 包含二进制代码和数据, 可被链接
2. 可执行目标文件: 包含二进制代码和数据, 可被运行
3. 共享目标文件: 特殊的可重定位目标文件, 可在加载或者执行的时候被动态链接

每个系统都有不同的目标文件格式, 但概念大致相同

现在 `x86 Linux` 采用 `ELF (Executable and Linkable Format)` 格式, 讨论集中在 `ELF` 进行

## 7.4 可重定位目标文件

![](https://github.com/lzlcs/image-hosting/raw/master/image.z5yv0jb4ygg.webp)

ELF头 和 节头部表中间的都是节
1. ELF 头: 系统字的大小, 字节顺序, 头大小, 目标文件类型, 机器类型, 节头部表位置大小数量等
2. `.text` 程序机器代码
3. `.rodata` 只读数据
1. `.data` 已初始化的全局和静态变量
1. `.bss` 未初始化的静态变量, 初始化为 0 的全局/静态变量
1. `.symlab` 符号表, 存放函数, 全局/静态变量的信息
1. `.rel.text` 存储代码段的重定位信息
1. `.rel.data` 存储 函数, 全局/静态变量的 重定位信息
1. `.debug` 局部变量信息, 原始 C 代码, 编译器加 `-g` 参数时才会出现
1. `.line` 行号和 `.text` 节中二进制代码的映射
1. `.strlab`, 字符串表, 包括所有符号表, 节名字

## 7.5 符号和符号表

设一个可重定位目标文件是 $m$ 

有三种不同的符号
1. $m$ 定义, 其他模块可使用 的全局符号
2. 其他模块定义, $m$ 可使用 的全局符号
3. $m$ 定义, $m$ 使用的局部符号, `static` 的函数和变量

```cpp
typedef struct {
    int name;   // 符号表中的字节偏移, 指出符号名字的位置
    char type:4, // 指明类型是函数还是数据
         binding:4; // 表明符号是本地的还是全局的
    char reserved; // 未使用
    short section; // 到节头部表的索引
    long value; // 在 .o 文件中是符号对应数据的相对位置
                // 在可执行目标文件中是符号对应数据的绝对位置
    long size; // 目标大小
} Elf64_Symbol;
```

可重定位目标文件中有三个伪节:
1. `ABS` 不该被重定位的符号
2. `UNDEF` 未定义的符号, 本模块引用, 其他模块定义
3. `COMMON` 未初始化的全局变量, 注意和 `.bss` 的区别

## 7.6 符号解析

1. 局部符号, 本模块定义本模块引用
2. 全局符号, 引用时发现不是在本模块定义就会假设在其他模块中定义 \
   生成一个符号表条目, 交给链接器处理

### 7.6.1 链接器如何解析多重定义的全局符号

编译器向汇编器输出每个全局符号: \
强(函数, 已初始化的全局变量) \
弱(未初始化的全局变量)

1. 不允许有多个同名的强符号
2. 如果有一个强符号和弱符号同名, 选择强符号
3. 多个弱符号同名, 任选一个

`-fno-common` 参数 让链接器在遇到多重定义的全局符号时触发错误 \
`-Werror` 警告当成错误

### 7.6.2 与静态库链接

静态库以一种 `archive` 的形式存放在磁盘中, 文件后缀名为 `.a` \
这种文件是一组连接起来的可重定位目标文件的集合, 有一个头部描述每个成员目标文件的信息

链接的时候只需要复制 程序引用的符号 所在的可重定位目标文件即可

```cpp
// addvec.c
int addcnt = 0;

void addvec(int *a, int *b, int *c, int n) {
    
    addcnt++;
    for (int i = 0; i < n; i++)
        c[i] = a[i] + b[i];

}
```

```cpp
// mulvec.c

int mulcnt = 0;

void mulvec(int *a, int *b, int *c, int n) {

    mulcnt++;
    for (int i = 0; i < n; i++)
        c[i] = a[i] * b[i];

}
```

```cpp
// vector.h

void addvec(int *a, int *b, int *c, int n);
void mulvec(int *a, int *b, int *c, int n);
```

构建静态库
```
gcc -c addvec.c mulvec.c
ar rcs libvector.a addvec.o mulvec.o
```

```cpp
// main.c
#include <stdio.h>
#include "vector.h"

int a[2] = { 1, 2 };
int b[2] = { 3, 4 };
int c[2];

int main() {
    addvec(a, b, c, 2);

    printf("c = [%d, %d]", c[0], c[1]);

    return 0;
}
```

使用静态库编译 \
`-static` 表示使用静态库
`-L. -lvector` 或者 `./libvector.a` 确认静态库位置
```
gcc -c main.c
gcc -static -o prog main.o ./libvector.a
./prog
```

## 7.6.3 链接器如何使用静态库来解析引用

在符号解析阶段, 链接器从左到右扫描可重定位目标文件和存档文件

链接器维护如下内容
1. `E` 表示应该合并到可执行文件的可重定位目标集合
2. `U` 表示引用了但是还未定义的符号
3. `D` 表示已经定义了的符号

链接器解析流程
1. 如果输入的是目标文件, 那么加入 `E`, 并修改 `U`, `D`
1. 如果输入的是存档文件, 那么扫描存档的成员目标文件 $m$ \
   如果 $m$ 定义的一个符号在 U 中, 那么把 $m$ 加入到 `E`, 修改 `U`, `D`
1. 扫描完所有输入之后, 如果 `U` 是空的, 那么就构架可执行目标文件

如果定义一个符号的库在引用这个符号的文件之前, 那么就会报错

建议把 库文件放到最后, 并且按照拓扑序组织库文件 \
$x$ 调用 $y$, 先放 $x$ 再放 $y$

## 7.7 重定位

1. 重定位节和符号定义:
   把所有可重定位目标文件的所有相同类型的节合并 \
   将运行时的内存地址赋给每个节每个符号, 这样指令和全局变量就有正确的地址了
2. 重定位节中的符号引用:
   把所有引用指向的地址修改正确

### 7.7.1 重定位条目

汇编器遇到最终位置未知的引用, 就会生成重定位条目 \
代码的重定位条目放在 `.ret.text` 中, 已初始化数据的重定位条目放在 `.ret.data` 中

```c
typedef struct {
    long offset;  // 偏移量
    long type:32, // 重定位类型
         symbol:32; // 符号表下标
    long addend;  // 某些类型的
} Elf64_Rela;
```

列出两种最基本的类型
1. `R_X86_64_PC32` 重定位相对引用
2. `R_X86_64_32` 重定位绝对引用

## 7.7.2 重定位符号引用

![](https://github.com/lzlcs/image-hosting/raw/master/image.5kopgorsjq0.png)

**重定位 PC 相对引用**

找到应该修改的字段位置, 包括 **`0xf`** 所在的 4 个字节, 所以
```
offset = 0xf;
symbol = sum;
type = R_X86_64_PC32
addend = -4;
```
假设 `ADDR(s) = ADDR(.text) = 0x4004d0`, `ADDR(symbol) = ADDR(sum) = 0x4004d0`

应该修改字段的地址 `refaddr = ADDR(s) + offset = 0x4004df` \

CPU 在执行完 `call` 指令之后, PC 变成 `0x4004de`, 此时 +5 才能得到 `sum` 的地址 \
所以这个字段改成 `05 00 00 00`

`*refptr = ADDR(sum) - refaddr + addend = 0x5`, 所以得到 `addend` 值为 -4

**重定位 PC 绝对引用**

```
offset = 0xa;
symbol = array;
type = R_X86_64_32;
addend = 0
```

假设 `ADDR(array) = 0x601018`

所以值应该修改成 `18 10 60 00`, 所以 `addend = 0`

## 7.8 可执行目标文件

![](https://github.com/lzlcs/image-hosting/raw/master/image.1lgcnlqk4mu8.png)

## 7.9 加载可执行目标文件

加载: 将程序复制到内存并运行

![](https://github.com/lzlcs/image-hosting/raw/master/image.6ne6dj1d8q80.webp)

## 7.10 动态链接共享库

静态库的缺点
1. 需要定期维护和更新
2. 基本每个程序都用的函数, 会在内存中被复制 $n$ 遍

动态链接: 在程序运行或加载的时候, 共享库加载到任意内存地址, 并和一个内存中的程序链接

动态链接器: 执行动态链接

做到共享的方式:
1. 在存储空间中只有一份 `.so` 文件
2. 在内存中可以有 `.text` 的共享对象

```
gcc -shared -fpic -o libvector.so addvec.c mulvec.c
gcc -o prog main.c ./libvector.so
```

`-fpic` 指示编译器生成与位置无关的代码, `-shared` 表示共享

`prog` 有一个 `.interp` 节, 包含动态链接器的路径名

加载器加载并运行动态链接器, 动态链接器开始工作
1. 重定位 `libc.so` 的代码和数据到内存
2. 重定位 `libvector.so` 的代码和数据到内存
3. 重定位 `main.c` 中在如上两个共享库中的符号引用

## 7.11 从应用程序中加载和链接共享库

动态链接的作用:
1. 分发软件: 用户下载新的共享库文件, 程序运行时直接链接新的共享库文件
2. 构建高性能 Web 服务器: 生成动态内容

通过操作系统和语言提供的接口来显式地调用共享库

## 7.12 位置无关代码

可以加载而无需重定位的代码称为位置无关代码 (`Position-Independent Code, PIC`)

共享库的编译必须使用 `-fpic` 参数

## 7.13 库打桩机制

库打桩允许你截获对共享库的调用, 取而代之执行自己的代码

### 7.13.1 编译时打桩

有源码时适用

写个头文件, 头文件里有个宏, 宏把调用的库函数改成自己的函数 \
修改源码, 包括上这个头文件

```
gcc -c mymalloc.c
gcc -I. -o intc int.c mymalloc.o
```

### 7.13.2 链接时打桩

有可重定位目标文件时适用

`__wrap_name` 是你的函数, `__real_name` 是原始函数

```
gcc -c mymalloc.c int.c
gcc -Wl, --wrap, malloc -Wl, --wrap, free -o intl int.o mymalloc.o
```

`-Wl, --wrap, malloc` 表示让链接器重定位的时候
1. 把 `int.c` 中调用的函数定位到 `__wrap_malloc` 中
2. 把调用的 `__real_malloc` 当作原始函数

### 7.13.3 运行时打桩

有可执行文件时适用

写一个文件, 不要包含 `malloc.h`, 有一个叫 `malloc` 的函数 \
函数中使用 运行时链接 来调用 `malloc` \
然后把动态链接器中的 `LD_PRELOAD` 环境变量改一下, 使得动态连接的时候先使用自己定义的库

这样动态链接的时候就会优先使用自己定义的库

## 7.14 处理目标文件的工具

![](https://github.com/lzlcs/image-hosting/raw/master/image.4et8avjt5lk0.webp)

## 7.15 小结

三种目标文件
1. 可重定位目标文件 `.o`
2. 可执行目标文件
3. 共享目标文件 `.so`

链接器通过符号解析和重定位来连接

# 练习



## 7.1 

![](https://github.com/lzlcs/image-hosting/raw/master/image.6ajht71dsdw0.webp)

## 7.2

1. `main.1`, `main.1`
2. 错误
3. `x.2`, `x.2`

## 7.3

```
gcc -static -o prog p.o libx.a
gcc -static -o prog p.o libx.a liby.a
gcc -static -o prog p.o libx.a liby.a libx.a
```

## 7.4

`0x4004df`, `0x4004e8`

## 7.5

`swap` 地址为 `0x4004e8` \
`call` 下一条指令是 `0x4004de`

相减得 `0xa`

## 7.6

![](https://github.com/lzlcs/image-hosting/raw/master/image.6ruvbzkj36k0.png)

## 7.7

`bar5.c` 中 `static double x;`

## 7.8

`static` 既不是强符号也不是弱符号, 所以不管他

1. 都是 `main.1`
2. 未知
3. 错误

## 7.9

`main` 函数第一条指令是 `push`, 号码为 `0x55`, 所以输出 `0x55`

## 7.10

```
gcc -static -o prog p.o libx.a p.o
gcc -static -o prog p.o libx.a liby.a libx.a
gcc -static -o prog p.o libx.a liby.a libx.a libz.a
```

## 7.11 

`z[2]` 在 `.bss` 里, 一开始不给他们分配内存

运行的时候给他们分配 8 字节, `0x228 -> 0x230`

## 7.12

`0xa`, `0x22`

## 7.13

A
```
ar -t /usr/lib32/libc.a | wc -l
ar -t /usr/lib32/libm.a | wc -l
```

B
```
readelf -S prog1
readelf -S prog2
```
发现节中多了 `.debug` 的部分 \

```
objdump -d prog1
objdump -d prog2
```
但是可执行代码的部分相同

C
```
lozical@LAPTOP-RPT2HO0D:/usr/bin$ cd /usr/bin && ldd gcc
        linux-vdso.so.1 (0x00007fffed336000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9614183000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f96143b7000)
```



