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

### 2.1.10 C 中的移位运算

1. `x << k` 丢弃 k 个最高位, 在最低位添加 k 个 0
2. `x >> k` 丢弃 k 个最低位
    1. 逻辑右移: 在最高位添加 k 个 0
    2. 算数右移: 符号位不变, 其后添加 k - 1 个零

几乎所有的编译器都采用算数右移

## 2.2 整数表示

### 2.2.1 整数数据类型

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.77vyucib1bc0.webp)

### 2.2.2 无符号和二进制补码编码

$B2U_w$ 表示无符号的二进制, $B2U_w = \Sigma_{i = 0}^{w - 1} x_i 2^i$ \
$B2T_w$ 表示二进制补码, $B2T_w = -x_{w-1}2^{w-1} +\Sigma_{i = 0}^{w - 2} x_i 2^i$ \
$B2O_w$ 表示二进制反码, $B2O_w = -(x_{w-1}2^{w-1}-1) +\Sigma_{i = 0}^{w - 2} x_i 2^i$ \
$B2S_w$ 表示符号数值, $B2S_w = (-1)^{x_{w-1}} \times \Sigma_{i = 0}^{w - 2} x_i 2^i$

二进制补码所能表示的值是不对称的 

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.3sv21t52zx60.webp)

### 2.2.3 有符号和无符号数的转换

注意强制类型转换:
```cpp
int x = -1;
cout << (unsigned)x << endl;
// 4294967295 = 2 ^ 32 - 1
```
![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.3ig3ten820c0.webp)

### 2.2.4 C 中的有符号和无符号数

类型转换发生:
1. 显式类型转换 `int(12345u)`
2. 赋值时, 等号右边会被强制转换为等号左边
3. `printf` 时, `%d`, `%u`, `%x`, 为有符号十进制, 无符号十进制, 十六进制
4. 运算时若出现无符号数据, 则把运算中的所有数据转换为无符号再计算

### 2.2.5 扩展一个数字的位表示

字长小的整数转换到字长大的整数:
1. 零扩展: 在最高位补 0
2. 符号扩展(补码使用): 在最高位补最高有效位

### 2.2.6 截断数字

1. 对于无符号截断 k 位: $x \; mod \; 2^k$
2. 对于有符号截断 k 位: 转成无符号之后同上操作, 之后把剩下的数字转成有符号

### 2.2.7 关于有符号和无符号数的建议

在非必要的情况下不要使用无符号数

## 2.3 整数运算

### 2.3.1 无符号加法

若当前整数由 w 位二进制位表示, 两个数加法就是 $(x+y)\; mod\;2^w$

模数加法形成一种数学结构, 称作阿贝尔群, 可交换可结合, 每个元素有一个加法逆元 \
对于元素 $x$, 它的加法逆元就是 $2^w-x$ 

### 2.3.2 二进制补码加法

设 $x + y = z$
1. 正溢出: $z>=2^{w-1}$, $sum = x + y - 2^w$
1. 正常: $2^{w-1}>z>=-2^{w-1}$, $sum = x + y$
1. 负溢出: $z<-2^{w-1}$, $sum = x + y + 2^w$

### 2.3.3 二进制补码的非

1. $x != -2^{w-1}$, $x$ 的加法逆元是 $-x$
2. 否则 $x$ 的加法逆元是它自己

一种执行取非的技术时 `~x + 1`

### 2.3.4 无符号乘法

$x \times y = x \times y \; mod \; 2^w$

### 2.3.5 二进制补码乘法

x, y 转成无符号进行乘法, 取后 w 位转成有符号整数

### 2.3.6 乘以常数

$2^w = 1u << w$ (无符号整数) \
$2^w = 1 << w$ (二进制补码) \
在不溢出的情况下成立

可以把乘的这个常数进行二进制拆分\
对于拆分出的某段 111.111, 设最低位为 m, 最高位为 n
1. FormA: `(x << m) + (x << m + 1) + ... + (x << n)`
1. FormB: `(x << n + 1) - (x << m)`


### 2.3.7 除以 2 的幂

整数除法向 0 舍, 右移运算永远向下舍 `-5 / 2 = -2`, `-5 >> 1 = -3`

以下代码等价于 $x / 2^k$
```cpp
(x < 0 ? (x + (1 << k) - 1) : x) >> k
```

### 2.3.8 关于整数运算的最后思考

补码是一种很聪明的表示方法, 绝大多数运算都和无符号整数的位级操作类似 \
数据溢出可能是造成难以调试的 bug 的一个原因


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

## 2.16

注意算术右移和逻辑右移的区别

## 2.17 ~ 2.22

简单算数

## 2.23

A. 模拟
B. `fun1` 提取最后八位(无符号), `fun2` 提取最后八位(有符号)

## 2.24

算数

## 2.25

`length - 1` 是 `11....11111` 运算两边出现无符号数时, 都转换为无符号数 \
所以 `for (i = 0; i < UMAX; i++)`, `i` 会超出数组范围

修改: `i < length`

## 2.26

A. 两者的减法可能为负数, 此时出现错误
B. 无符号中, 负数会被认为是很大的数, 大于零而产生错误
C. `strlen(s) > strlen(t)`

## 2.27

```cpp
int uadd_ok(unsigned x, unsigned y) {
    return x <= (x + y);
}
```

## 2.28 ~ 2.29

简单算数

## 2.30

```cpp
int tadd_ok(int x, int y) {
    return !((x >= 0 && y >= 0 && x + y < 0) ||
             (x < 0 && y < 0 && x + y > 0));

}
```

## 2.31

哈哈哈哈哈哈
就算溢出了该等式仍旧成立

## 2.32

$TMIN$ 的加法逆元是他自己, 要特判

```cpp
int tsub_ok(int x, int y) {
    const int TMIN = -2147483648;
    if (y == TMIN) return x < 0;
    return tadd_ok(x, -y);
}
```

## 2.33 ~ 2.34

简单计算

## 2.35

1. 当 $t=0$ 时, $p < 2^w$, $x \times y = p$ 不会溢出
2. $p$ 除以 $x$ 等于 $q$, 余 $r$
3. $x \times y = x \times q + r + t 2^w$
    - $q=y$, $0 = r + t2^w$, 又 $\lvert r \rvert < 2^w$, 所以 $r = t = 0$
    - $r = t = 0$, $x \times y = x \times q$, 显然 $q=y$

## 2.36

```cpp
int tmult_ok(int x, int y) {
    int64_t tmp = (int64_t)x * y;
    return tmp == (int)tmp;
}
```

## 2.37

A. 一点没用 \
B. 不能改 `malloc` 的代码, 所以只能在溢出的时候返回空
```cpp
uint64_t required_size = ele_cnt * (uint64_t) ele_size;
size_t request_size = (size_t) required_size;
if (required_size != request_size)
    return NULL;
```

## 2.38

对于任何值都可以

## 2.39

$n$ 是最高位时, `1 << n + 1` 为 $0$, FormB 可以写成 `-x << m`

## 2.40 

简单算数

## 2.41

当 $n = m$ 或者 $n = m + 1$ 时, 使用 FormA, 此时只需要做一次或者两次移位运算 \
其他情况下, FormB 需要做两次移位运算, FormA 需要更多次移位运算

## 2.42

```cpp
int div16(int x) {
    // 加的是 0 还是 15
    int bias = (x >> 31) & 0xF;
    return (x + bias) >> 4;
}
```

## 2.43

$M = 31, N = 8$

## 2.44

A. False, $x = TMIN_{32}$, `-2147483648 > 0 || 2147483647 < 0` 为 `false` \
B. True, 如果 `(x & 7) == 7`, 那么最低三位一定为 `111`, 右移 29 位之后成为符号位, 一定为负 \
C. False, $x = 46341$ \
D. True \
E: False, $x = TMIN_{32}$ \
F: True, 都会被转换成无符号再计算 \
G: True, $x \times (-y - 1) + uy \times ux$, 也就是 $-x$ 
