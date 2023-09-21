---
title: CSAPP 学习笔记 (第二章) 
date: 2023-08-01 12:50:00
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

![](https://github.com/lzlcs/image-hosting/raw/master/image.jcndpw581w8.webp)

![](https://github.com/lzlcs/image-hosting/raw/master/image.2rfa517i1ok0.webp)

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

![](https://github.com/lzlcs/image-hosting/raw/master/image.77vyucib1bc0.webp)

### 2.2.2 无符号和二进制补码编码

$B2U_w$ 表示无符号的二进制, $B2U_w = \Sigma_{i = 0}^{w - 1} x_i 2^i$ \
$B2T_w$ 表示二进制补码, $B2T_w = -x_{w-1}2^{w-1} +\Sigma_{i = 0}^{w - 2} x_i 2^i$ \
$B2O_w$ 表示二进制反码, $B2O_w = -(x_{w-1}2^{w-1}-1) +\Sigma_{i = 0}^{w - 2} x_i 2^i$ \
$B2S_w$ 表示符号数值, $B2S_w = (-1)^{x_{w-1}} \times \Sigma_{i = 0}^{w - 2} x_i 2^i$

二进制补码所能表示的值是不对称的 

![](https://github.com/lzlcs/image-hosting/raw/master/image.3sv21t52zx60.webp)

### 2.2.3 有符号和无符号数的转换

注意强制类型转换:
```cpp
int x = -1;
cout << (unsigned)x << endl;
// 4294967295 = 2 ^ 32 - 1
```
![](https://github.com/lzlcs/image-hosting/raw/master/image.3ig3ten820c0.webp)

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

1. 对于无符号截断 k 位: $x \enspace mod \enspace 2^k$
2. 对于有符号截断 k 位: 转成无符号之后同上操作, 之后把剩下的数字转成有符号

### 2.2.7 关于有符号和无符号数的建议

在非必要的情况下不要使用无符号数

## 2.3 整数运算

### 2.3.1 无符号加法

若当前整数由 w 位二进制位表示, 两个数加法就是 $(x+y)\enspace mod\enspace2^w$

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

$x \times y = x \times y \enspace mod \enspace 2^w$

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

## 2.4 浮点

### 2.4.1 二进制小数

$b_mb_{m-1}\cdots b_0.b_{-1}b_{-2} \cdots b_n$ 表示 $d = \Sigma_{i = n}^{m} 2^i d_i$

若将分数转换为小数, 则分母不为 2 的幂 的分数不能被准确表示

### 2.4.2 IEEE 浮点表示

使用 $V = (-1)^s \times M \times 2^E$ 的形式来表示一个数
1. 符号 `sign` 1 表示负数, 0 表示正数
2. 有效数 `significand` M 是一个二进制小数
3. 指数 `exponent` E, 对浮点数加权

包括 1 位 `s`, $k$ 位 E,  $n$ 位 M
1. `float`: $s = 1$, $k = 8$, $n = 23$ 共 32 位
1. `double`: $s = 1$, $k = 11$, $n = 52$ 共 64 位

三种模式:

设 $Bias = 2^{k-1}-1$
1. 规格化值: 指数区域不是全 0 也不是全 1
    - 设 $k$ 位指数区域表示的数为 $e$, $E = e - Bias$ \
      单精度 $e \in [-126, 127]$, 双精度 $e \in [-1022, 1023]$
    - 设 $f$ 为小数区域表示的数, $M = 1 + f$ 这样结合指数可以免费获得一个有效位
2. 非规格化值: 指数区域全零
    - $E = 1 - Bias$
    - $M = f$

    非规格化值由两个目的
    1. 保证能表示 0
    2. 用来表示非常接近 0 的数

3. 特殊数值: 指数区域全是 1
    - $f = 0$, 根据符号位表示正无穷或者负无穷
    - $f \not= 0$, 表示 `NaN` 不是一个数字

### 2.4.3 数值实例

非规格化值如此定义 $E$ 是为了和规格化值免费多出的一位相适配

![](https://github.com/lzlcs/image-hosting/raw/master/image.3x8snzo6zxq0.webp)
如此定义的非规格化值和规格化值是无缝衔接的

![](https://github.com/lzlcs/image-hosting/raw/master/image.3jvwbxeylxg0.webp)

### 2.4.4 舍入

浮点运算只能近似表示实数运算, \
所以对于 $x$ 应该有一种系统的方法使它能被转换成浮点数, 这就是舍入


一般有四种舍入方式: 
![](https://github.com/lzlcs/image-hosting/raw/master/image.44qais3bbvo0.webp)

相似的, 这些舍入方法可以使用在二进制数上

### 2.4.5 浮点运算

浮点加法, 乘法没有结合律, 因为舍入的存在
浮点乘法没有分配律
```cpp
3.14 + 1e10 - 1e10 = 0 // 3.14 被舍入
3.14 + (1e10 - 1e10) = 3.14
```

浮点运算有单调性 \
1. $a \geq b$, 则 $a + c \geq b + c$ \
2. $a \geq b$, 则 
    - $ac \geq bc \enspace (c > 0)$
    - $ac \leq bc \enspace (c < 0)$
3. $a^2 \geq 0 \enspace (a \not= NaN)$

之前的整数运算因为溢出的缘故没有单调性

### 2.4.6 C 中的浮点

类型转换:

`int -> float -> double` 安全
`double -> float -> int` 不安全

注意浮点数转换成整数的时候数值会向零截断

> 第三版中删除了 IA32 的内容

## 2.5 小结

大多数机器使用二进制补码作为整数的表示, 使用 IEEE 标准作为浮点数的表示方式 \
有限的编码长度可能导致溢出, 舍入和错误的发生



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

## 2.45 

简单算数

## 2.46

A. $0.000000000000000000000001100[1100]\cdots$ \
B. $2^{-10} \times \frac{1}{10}$ 约为 $9.54 \times 10_{-8} \
C, D 简单算数

## 2.47 ~ 2.48

简单算数

## 2.49

在小数部分的最高位放上 1, 后面 $n-1$ 个 0 最后再放个 1 这样就超出范围了 \
$2^{n+1}+1$, $n=23$ 时十进制表示为16777217

## 2.50 

简单舍入

## 2.51

A. 题干中 23 位末尾 0 变 1
B. 只有最后一位 1, $2^{-22} \times \frac{1}{10}$, 约为 $2.38 * 10^{-8}
C. D. 简单算数

## 2.52 

算数

## 2.53

```cpp
#define POS_INFINITY (1.9 * 1e308)
#define NEG_INFINITY (-POS_INFINITY)
#define NEG_ZERO (1.0 / NEG_INFINITY)
```

## 2.54

- A. YES \
- B. NO: `float` 中只有23位存储小数, 所以需要存储25位的数字时有时会出现问题
   ```cpp
   x = 0x01000001;
   cout << x << endl;
   cout << (int)(float)x << endl;
   x = 0x01FFFFFF;
   cout << x << endl;
   cout << (int)(float)x << endl;
   ```
- C. NO: `double` 范围更大
- D. YES
- E. YES
- F. YES
- G. YES
- H. NO: 
    ```cpp
    f = 1e20, d = 3.14;
    cout << d << endl << (f + d) - f << endl;
    ```

## 2.55 ~ 2.57

在正文中已经编译运行, 本机器使用小端法

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

void show_short(short x) {
    show_bytes((byte_pointer)(&x), sizeof(short));
}

void show_long(long x) {
    show_bytes((byte_pointer)(&x), sizeof(long));
}

void show_double(double x) {
    show_bytes((byte_pointer)(&x), sizeof(double));
}

void show_pointer(void *x) {
    show_bytes((byte_pointer)(&x), sizeof(void *));
}

void test_show_bytes(int val) {
    int ival = val;
    float fval = (float)val;
    int* pval = &val;
    short sval = val;
    long lval = val;
    double dval = (double)val;
    show_int(ival);
    show_float(fval);
    show_pointer(pval);
    show_short(sval);
    show_long(lval);
    show_double(dval);
}
```

## 2.58

```cpp
int is_little_endian() {
    unsigned int x = 1;
    byte_pointer y = (byte_pointer)&x;

    return (int)(y[0] == 0x01);
}
```

## 2.59

```cpp
int _2_59(int x, int y) {
    int mask = 0xFF;
    return (x & mask) | (y & (~mask));
}
```

## 2.60

```cpp
int replace_byte(unsigned x, int i, unsigned char b) {
    int mask = ~(0xFF << (i << 3));
    int replace_value = (int)b << (i << 3);

    return x & mask | replace_value;
}
```

## 2.61

```cpp
int A(int x) {
    return !(~x);
}

int B(int x) {
    return !x;
}

int C(int x) {
    int mask = 0xFF;
    return A(x | ~mask);
}

int D(int x) {

    int mask = 0xFF;

    int shift_val = (sizeof(int) - 1) << 3;
    int xright = x >> shift_val;

    return B(xright & mask);
}
```

## 2.62

```cpp
int int_shifts_are_arithmetic() {
    return !(~(-1 >> 1));
}
```

## 2.63

一个数不变: `x | 0`, `x & -1` \
-1 左移可以构造高位全为 1 的数字

```cpp
unsigned srl(unsigned x, int k) {
    unsigned xsra = (int)x >> k;

    int w = (sizeof(int) << 3);

    int mask = -1 << (w - k);
    return xsra & (~mask);
}

int sra(int x, int k) {
    unsigned xsrl = (unsigned)x >> k;

    int w = (sizeof(int) << 3);
    int mask = -1 << (w - k);

    int sign = (x & (1 << (w - 1)));

    mask &= (!sign) - 1;

    return xsrl | mask;
}
```

## 2.64

```cpp
int any_odd_one(unsigned x) {
    return !!(0xAAAAAAAA & x);
}
```

## 2.65

统计 1 个数的奇偶性, 根据异或不进位加法的特性 \
把前 16 位和后 16 位异或起来, 这样后 16 位 1 的个数的奇偶性与先前相同 \
依次到 8 位, 4 位, 2 位, 1 位, 这样只需要判断最后一位的奇偶性即可

```cpp
int odd_ones(unsigned x) {
    x ^= (x >> 16);
    x ^= (x >> 8);
    x ^= (x >> 4);
    x ^= (x >> 2);
    x ^= (x >> 1);
    return x & 0x1;
}
```

## 2.66

1. 首先生成一个数字, 使得最高有效的 1 到最低位全是 1 \
    `x |= x >> 1` 这样最高的两位都是 1 \
    `x |= x >> 2` 这样最高的四位都是 1 \
    依此类推 \
    `x |= x >> 16` 达成目标
2. 右移一次再加一 (如果 `x = 0` 则加 0) \
    不加一再右移是为了防止溢出

```cpp
int leftmost_one(unsigned x) {
    x |= x >> 1;
    x |= x >> 2;
    x |= x >> 4;
    x |= x >> 8;
    x |= x >> 16;
    // x = 0 时加的数为 0
    int add_number = (x && true);
    return (x >> 1) + add_number;
}
```

## 2.67

A. 左移 32 位的行为未定义
B, C 如下
```cpp
int int_size_is_32() {
    int set_msb = 1 << 31;
    int beyond_msb = set_msb << 1;
    return set_msb && !beyond_msb;
}

int int_size_is_32_for_16_bit() {
    int set_msb = 1 << 15 << 15 << 1;
    int beyond_msb = set_msb << 1;
    return set_msb && !beyond_msb;
}
```

## 2.68

```cpp
int lower_one_mask(int n) {
    int w = sizeof(int) << 3;
    return (unsigned)-1 >> (w - n);
}
```

## 2.69

```cpp
unsigned rotate_left(unsigned x, int n) {
    int w = sizeof(int) << 3;
    int offset = w - n;

    int lower = x >> offset;
    int upper = x << n;
    return lower | upper;
}
```
## 2.70

如果 x 为正, 那么第 n-1 位直到第 w-1 位都必须为 0 \
如果 x 为负, 那么第 n-1 位直到第 w-1 位都必须为 1

右移去掉 大于 n-1 位的数, 算数左移回来, 如果运算之后结果不变就是可以表示

```cpp
int fits_bits(int x, int n) {
    int w = sizeof(int) << 3;
    int offset = w - n;
    return (x << offset >> offset) == x;
}
```

## 2.71

原代码只提取了位, 并没有转换成 `signed` 形式

```cpp
int xbyte(packed_t word, int bytenum) {

    int w = sizeof(int) << 3;
    int bitnum = bytenum << 3;
    int delta = w - 8;

    return int((word >> bitnum) & 0xFF) << delta >> delta;
}
```

## 2.72 

A. `sizeof` 运算符返回 `unsigned` 值, `maxbytes - sizeof(val)` 结果为负数就会出错
B. `maxbytes >= (int)sizeof(val)` 即可

## 2.73

```cpp
int saturating_add(int x, int y) {
    int sum = x + y;
    int sig_mask = INT_MIN;

    int pos_over = !(x & sig_mask) && !(y & sig_mask) && (sum & sig_mask);
    int neg_over = (x & sig_mask) && (y & sig_mask) && !(sum & sig_mask);

    (pos_over && (sum = INT_MAX)) || (neg_over && (sum = INT_MIN));

    return sum;
}
```

## 2.74

```cpp
int tadd_ok(int x, int y) {
    int sum = x + y;
    int pos_over = (x > 0 && y > 0 && sum < 0);
    int neg_over = (x < 0 && y < 0 && sum > 0);
    return !(pos_over || neg_over);
    
}
int tsub_ok(int x, int y) {
    int res = 1;
    (y == INT_MIN && x > 0 && (res = 0));
    return res && tadd_ok(x, -y);
}
```

## 2.75

`signed`: 
$$
\begin{align*}
  &(x - 2^{32}s_x)(y - 2^{32}s_y) \enspace mod \enspace 2^{32} \\
= &(xy -2^{32}s_xy - 2^{32}s_yx - s_xs_y2^{64}) \enspace mod \enspace 2^{32} \\
= &(xy - s_xy - s_yx) \enspace mod \enspace 2^{32}

\end{align*}
$$

所以转换回去只需要 $+ s_xy + s_yx$

```cpp
int signed_high_prod(int x, int y) {
    long long tmp = 1ll * x * y;
    return (int)(tmp >> 32);
}

unsigned unsigned_high_prod(unsigned x, unsigned y) {
    int sig_x = x >> 31;
    int sig_y = y >> 31;
    return signed_high_prod(x, y) + sig_x * y + sig_y * x;
}
```

## 2.76

```cpp
void* another_calloc(size_t nmemb, size_t size) {
    if (nmemb == 0 || size == 0) return NULL;
    if (nmemb * size / size != nmemb) return NULL;
    
    void* begin = malloc(nmemb);
    memset(begin, 0, size * nmemb);
    return begin;
}
```

## 2.77

```cpp
int x = 1;
cout << (x << 4) + x << endl;
cout << x - (x << 3) << endl;
cout << (x << 6) - (x << 2) << endl;
cout << (x << 4) - (x << 7) << endl;
```

## 2.78

```cpp
int divide_power2(int x, int k) {
    int w = sizeof(int) << 3;
    int sign = x >> (w - 1) & 1;
    return (x + sign) >> k;
}
```

## 2.79

```cpp
int mul3div4(int x) {
    x = (x << 1) + x;
    return divide_power2(x, 2);
}
```

## 2.80

把最低两位分开计算
1. 此时 `upper` 部分可以被 4 整除, 之后再乘 3 不会有溢出风险
2. `lower <= 3` 可以直接乘 3 除以 4, 注意向 0 舍入的问题

```cpp
int _2_80(int x) {
    int upper = x & ~0x3;
    int first = upper >> 2;
    first += (first << 1);

    int lower = x & 0x3;
    int is_neg = x & INT_MIN;
    lower += lower << 1;
    is_neg && (lower += 3);
    lower >>= 2;

    return first + lower;
}
```

## 2.81

```cpp
int _2_81_A(int k) {
    return -1 << k;
}

int _2_81_B(int k, int j) {
    return ~(-1 << k) << j;
}
```

## 2.82

A. False, `x = -2147483648`
B. True
C. True
D. True
E. True

## 2.83
A.
$$
2^{len(y)}Y - y = Y \\
Y = \frac{y}{2^{len(y)}-1}
$$
B. $\frac57$, $\frac35$, $\frac{19}{63}$

## 2.84

```cpp

unsigned f2u(float x) {
    return *(unsigned*)&x;
}

int float_le(float x, float y) {
    unsigned ux = f2u(x);
    unsigned uy = f2u(y);

    unsigned sx = ux >> 31;
    unsigned sy = uy >> 31;

    // 防止 x = +0.0, y = -0.0 的情况
    return (ux << 1 == 0 && uy << 1 == 0) ||
           (sx && !sy) ||
           (!sx && !sy && ux <= uy) ||
           (sx && sy && ux >= uy);
}
```

## 2.85

1. A. `1 2+1023 11000000` -> `1 10..01 1100...`
2. B. `0 bias+n 11..11` n 为第三段的位数
3. C. 最小规格化数: $2^{1-bias}$, 倒数为 $2^{bias-1}$ \
    $E = bias-1$, $e = E + bias = 253$, \
    `0 11..1101 000000..`

## 2.86 ~ 2.88

计算题

## 2.89

- A. True
- B. False (`x - y` 可能溢出)
- C. True `double` 精度很高, 在 `int` 范围内可以有结合律
- D. False 乘法结果为 64 位, 有可能舍入, 没有结合律
- E. False 除零错误

## 2.90

```cpp
float u2f(unsigned x) {
    return *(float*) &x;
}

float fpwr2(int x) {
    unsigned exp, frac;
    unsigned u;

    int bias = (1 << (8 - 1)) - 1;

    if (x < -126 - 23) {
        exp = 0;
        frac = 0;
    } else if (x < -126) {
        exp = 0;
        frac = 1 << (x + 23 + 126);
    } else if (x < 128) {
        exp = x + bias;
        frac = 0;
    } else {
        exp = 0xFF;
        frac = 0;
    }
    u = exp << 23 | frac;
    return u2f(u);
}
```

## 2.91 

算数

## 2.92

```cpp
typedef unsigned float_bits;

float_bits float_negate(float_bits f) {
    unsigned sign = f >> 31;
    unsigned exp = f >> 23 & 0xFF;
    unsigned frac = f & 0x7FFFFF;

    if (exp == 0xFF && frac != 0) return f;
    return (~sign << 31) | (exp << 23) | frac;
}
```

## 2.93

```cpp
float_bits float_absval(float_bits f) {
    unsigned sign = f >> 31;
    unsigned exp = f >> 23 & 0xFF;
    unsigned frac = f & 0x7FFFFF;

    if (exp == 0xFF && frac != 0) return f;
    return (0 << 31) | (exp << 23) | frac;
}
```

## 2.94

```cpp
float_bits float_twice(float_bits f) {
    unsigned sign = f >> 31;
    unsigned exp = f >> 23 & 0xFF;
    unsigned frac = f & 0x7FFFFF;

    if (exp == 0) frac <<= 1;
    else if (exp == 0xFF - 1) exp = 0xff, frac = 0;
    else exp += 1;

    return (sign << 31) | (exp << 23) | frac;
}
```

## 2.95

```cpp
float_bits float_half(float_bits f) {
    unsigned sign = f >> 31;
    unsigned exp = f >> 23 & 0xFF;
    unsigned frac = f & 0x7FFFFF;


    int addition = ((frac & 0x3) == 0x3);

    if (exp == 0xFF) return f;
    else if (exp == 0) 
        frac = (frac >> 1) + addition;
    else if (exp == 1) {
        unsigned rest = f & 0x7FFFFFFF;
        rest = (rest >> 1) + addition;

        exp = rest >> 23 & 0xFF;
        frac = rest & 0x7FFFFF;
    } else exp -= 1;

    return (sign << 31) | (exp << 23) | frac;
}
```

## 2.96

```cpp
int float_f2i(float_bits f) {
    unsigned sign = f >> 31;
    unsigned exp = f >> 23 & 0xFF;
    unsigned frac = f & 0x7FFFFF;

    int bias = 127, num = 0;
    if (exp >= 0 && exp < bias) num = 0;
    else if (exp >= 31 + bias) num = 0x80000000;
    else {
        int E = exp - bias;
        int M = frac | 0x800000;
        if (E > 23) num = M << (E - 23);
        else num = M >> (23 - E);
    }
    
    return sign ? -num : num;
}
```

## 2.97

```cpp
float_bits float_i2f(int x) {
    int bias = 127;
    // 特判 0 和 INT_MIN
    if (x == 0) return 0;
    if (x == INT_MIN) 
        return (1 << 31) | ((bias + 31) << 23) | 0;
    
    // 判断符号位
    int sign = 0;
    if (x < 0) x = -x, sign = 1;

    // 计算有效位
    int tmp = x, bits = 0;
    while (tmp) tmp >>= 1, bits++;
    int fbits = bits - 1;

    int exp = bias + fbits, frac, exp_sig;

    // 去掉最高位 1 之后的位
    int rest = x & bit_mask(fbits);

    if (fbits <= 23) { // 可以被直接表示, 不需要舍入
        frac = rest << (23 - fbits);
        exp_sig = exp << 23 | frac;
    } else {

        int offset = fbits - 23; // 应该舍弃的位
        int round_mid = 1 << (offset - 1);

        int round_part = rest & bit_mask(offset);
        frac = rest >> offset;

        exp_sig = exp << 23 | frac;

        exp_sig += ((round_part > round_mid) || 
                    (round_part == round_mid && ((frac & 0x01) == 1)));
    }

    return (sign << 31) | exp_sig;
}
```
