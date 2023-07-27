---
title: CSAPP 学习笔记 (第二章) 
date: 2023-07-26 12:50:00
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# Chapter 2: 信息的表示和处理

二进制信号可以被更好地表示, 基于二值信号的电路简单又可靠

单独的位没有用处, 但是使用一串01序列以及某种解释, 就能表示任意有限集合的元素

这里考虑三种数字编码: 无符号整数, 二进制补码, 浮点数编码

## 2.1 信息存储

大多数计算机使用 8 个比特为 1 字节来作为可寻址的存储最小单位 \
程序把存储器视为一个非常大的字节数组, 也就是虚拟存储器

存储器的每一个字节都由一个唯一的数字表示, 这称为地址

所有可能地址的集合称为虚拟地址空间

### 2.1.1 十六进制表示法

二进制四合一

进制转换程序: 
```cpp
char number_to_char(int x) {
    if (x < 10) return char('0' + x);
    return 'A' + (x - 10);
}

int char_to_number(char x) {
    if ('0' <= x && x <= '9') return int(x - '0');
    return int(x - 'A' + 10);
}

string base_transform(int bef, int aft, string str) {
    string res = "";
    int tmp = 0;
    for (auto c: str)
        if (c == ' ') continue;
        else tmp *= bef, tmp += char_to_number(c);

    while (tmp)
        res.insert(res.begin(), number_to_char(tmp % aft)), 
        tmp /= aft;
    
    return res;
}
```

### 2.1.2 字 

(本节于第三版中与 2.1.3 合并)

对于一个字长为 n 位的机器, 虚拟地址的范围为 $2^n - 1$ \
32 位机器限制内存为位 4GB, 但如今基本都是 64 位的机器

### 2.1.3 数据大小

C语言中给每个数据类型的字节数随着机器和编译器的不同而不同

### 2.1.4 寻址和字节顺序

多字节对象被存储为连续的字节序列 \
如一个 `int` 型变量 `x` 地址为 `0x100`, 那么 `int` 的四个字节就是 `0x100`, `0x101`, `0x102`, `0x103`

由两种存储的方式
1. 大端法: 最高有效字节在前
2. 小端法: 最低有效字节在前

带来的问题:
1. 不同存储方式的机器的信息交流
2. 阅读表示整数的字节序列
3. 编写规避正常的类型系统的程序时

字节表示程序:
```cpp
typedef unsigned char* byte_pointer;

void show_bytes(byte_pointer start, size_t len) {
    for (int i = 0; i < len; i++)
        printf("%.2x ", start[i]);
    printf("\n");
}

void show_int(int x) {
    show_bytes((byte_pointer)(&x), sizeof(int));
}

void show_float(float x) {
    show_bytes((byte_pointer)(&x), sizeof(float));
}

void show_pointer(void *x) {
    show_bytes((byte_pointer)(&x), sizeof(void *));
}

void test_show_bytes(int val) {
    int ival = val;
    float fval = (float)val;
    int* pval = &val;
    show_int(ival);
    show_float(fval);
    show_pointer(pval);
}
```

### 2.1.5 表示字符串

C 中的字符串是以 00 为结尾的字符数组, 每个字符都由某个标准编码表示 (比如 `ASCII`)

### 2.1.6 表示代码

不同的机器使用不同且不兼容的指令和编码格式, 二进制代码很少可以做移植

### 2.1.7 布尔代数和环

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.jcndpw581w8.webp)

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2rfa517i1ok0.webp)

布尔 `and` 和 `xor` 分别表示模 2 的乘法和加法, 每个元素都是它自己的加法逆元 `a ^ a = 0`

将这四种运算 (`&`, `|`, `~`, `^`) 扩展到位向量上, 对于每一位都进行这些运算

位向量的一个有用的应用就是表示有限集合 `&` 就是交集, `|` 就是并集, `~` 相当于补集

### 2.1.8 C 中的位级运算

位运算的一个常见用法是实现掩码运算, 把掩码的特定位置设置成 1, 与 x 进行按位与运算
这样就会得到 x 在这些特定位的值, 其他位为 0
> `~0` 是全为 1 的掩码, 这样的可移植性强于 `0xFFFFFFFF`

### 2.1.9 C 中的逻辑运算

`||`, `&&`, `!` 这些运算认为非零的参数都为 `true`, 0 为 `False` \
当且仅当运算两边只有 ${0, 1}$ 时, 逻辑运算和位级运算等价 \

其次逻辑运算具有短路效应:
1. `||` 前若为 `true`, 直接返回 `true`,  `||` 后的表达式不计算
1. `&&` 前若为 `false`, 直接返回 `false`,  `&&` 后的表达式不计算




# 练习选做

## 2.1 ~ 2.6 

简单的进制转换和运算

## 2.7

打印字符串中编码程序:
```cpp
const char *s = "mnopqr";
show_bytes((byte_pointer)s, strlen(s));
```

## 2.8 ~ 2.9

简单的代数运算

## 2.10

```
a                b
a                a ^ b
a ^ a ^ b = b    a ^ b
b                b ^ a ^ b = a
b                a
```

## 2.11

A. $k$ (下标从 0 开始) \
B. 根据上述代码, 如果 `*x` 和 `*y` 相同, 异或的结果只能为 0 \
C. `<=` 改成 `<`

## 2.12

```cpp
int x = 0x87654321;

// My solution
// int y = (1 << 8) - 1;

int y = 0xFF;

printf("0x%.8x\n", (y & x));

int z = ~y;

printf("0x%.8x\n", (z ^ x));

printf("0x%.8x\n", (y | x));
```

## 2.13

```cpp
int bis(int x, int m) {
    int result = x | m;
    return result;
}

int bic(int x, int m) {
    int result = x & ~m;
    return result;
}

int bool_or(int x, int y) {
    int result = bis(x, y);
    return result;
}

int bool_xor(int x, int y) {
    // x ^ y = (x & ~y) | (y & ~x)
    int result = bis(bic(x, y), bic(y, x));
    return result;
}
```

## 2.14

注意分辨逻辑运算和位级运算

## 2.15 

```cpp
!(x ^ y)
// 此处不能写 ~
```
