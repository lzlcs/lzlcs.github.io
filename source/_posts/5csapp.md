---
title: CSAPP 学习笔记 (第五章)
date: 2023-08-29 12:00:23
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# Chapter 5: 优化程序性能

## 5.1 优化编译器的能力和局限性

现代编译器向用户提供一些对它们所使用的优化的控制, 最简单的是指定优化级别 \
`-Og`, `-O1`, `-O2`, `-O3` 从低到高是不同的优化等级

编译器对程序使用安全的优化 \
一个函数以两个指针为参数, 编译器就要考虑两个指针相同的情况 \
多次调用函数的时候, 就要考虑函数的副作用---对全局变量产生的影响

## 5.2 表示程序性能

`CPE(Cycles Per Elements)` 表示每个元素所需的周期数, 在度量执行重复计算的程序时很适当

时钟运行频率: 每秒处理器执行多少个周期

## 5.3 程序示例

使用如下程序来测试优化等级
```cpp
#include <iostream>
#include <cstdlib>

using namespace std;

typedef int data_t;

typedef struct {
    long len;
    data_t *data;
} vec_rec, *vec_ptr;

vec_ptr new_vec(long len) {

    vec_ptr res = (vec_ptr)malloc(sizeof(vec_rec));
    data_t *data = NULL;

    if (!res) return NULL;

    res->len = len;

    if (len > 0) {

        data = (data_t *)calloc(len, sizeof(data_t));

        if (!data) {
            free((void *)res);
            return NULL;
        }
    }

    res->data = data;
    return res;
}

int get_vec_element(vec_ptr v, long index, data_t *dest) {
    if (index < 0 || index >= v->len) return 0;

    *dest = v->data[index];
    return 1;
}

long vec_length(vec_ptr v) {
    return v->len;
}

#define IDENT 0
#define OP +

void combine1(vec_ptr v, data_t *dest) {

    *dest = IDENT;

    for (long i = 0; i < vec_length(v); i++) {
        data_t val;
        get_vec_element(v, i, &val);
        *dest = *dest OP val;
    }

}

int main() {

    vec_ptr v = new_vec(10);

    const int times = 1e7;

    for (int i = 0; i < times; i++) {

        for (int j = 0; j < vec_length(v); j++)
            v->data[j] = rand();
        
        data_t sum;
        combine1(v, &sum);
    }

    return 0;
}
```

```
g++ -o tmp tmp.cpp && time ./tmp
g++ -Og -o tmp tmp.cpp && time ./tmp
g++ -O1 -o tmp tmp.cpp && time ./tmp
g++ -O2 -o tmp tmp.cpp && time ./tmp
g++ -O3 -o tmp tmp.cpp && time ./tmp
```

可以轻松观察优化等级带来的影响

## 5.4 消除循环的低效率

把 `combine1` 中循环的结束条件中 `vec_length` 函数放到函数之外
```cpp
void combine2(vec_ptr v, data_t *dest) {

    *dest = IDENT;

    long length = vec_length(v);

    for (long i = 0; i < length; i++) {
        data_t val;
        get_vec_element(v, i, &val);
        *dest = *dest OP val;
    }

}
```
可以发现在低等级的优化级别中, 运行速度明显加快

但是如果 `vec_length` 是一个线性复杂度的函数, 整段函数的复杂度变为 $O(n^2)$ \
这时使用代码移动可以让整段函数重新变成 $O(n)$

**代码移动** 多次执行相同运算结果相同, 可以把这段代码放到外部

## 5.5 减少过程调用

可以直接操作数组来获取元素而不是通过调用函数

```cpp
data_t *get_vec_start(vec_ptr v) {
    return v->data;
}

void combine3(vec_ptr v, data_t *dest) {

    *dest = IDENT;

    long length = vec_length(v);
    data_t *data = get_vec_start(v);

    for (long i = 0; i < length; i++) {
        *dest = *dest OP data[i];
    }

}
```

这种变换损害了程序的模块性, 但是提高了运行速度

## 5.6 消除不必要的内存引用

`combine3` 循环中每次都要调用 `dest` 指针所指向的内存 \
可以通过引入中间变量而消除这种不必要的内存引用

```cpp
void combine4(vec_ptr v, data_t *dest) {

    long length = vec_length(v);
    data_t *data = get_vec_start(v);

    data_t res = IDENT;

    for (long i = 0; i < length; i++) {
        res = res OP data[i];
    }

    *dest = res;
}
```

## 5.7 理解现代处理器

现在的处理器都是超标量乱序发射的, 可以达到更好的指令级并行度 \
包括指令控制单元 `Instruction Control Unit (ICU)` 和 执行单元 `Execution Unit(EU)`

找到程序的关键路径, 最小化 CPE

## 5.8 循环展开

下面代码被称作 `2 * 1` 的循环展开
```cpp
void combine5(vec_ptr v, data_t *dest) {

    long length = vec_length(v);
    data_t *data = get_vec_start(v);

    data_t res = IDENT;

    long i;
    for (i = 0; i < length; i += 2) {
        res = (res OP data[i]) OP data[i + 1];
    }

    for (; i < length; i++) res = (res OP data[i]);

    *dest = res;
}
```

## 5.9 提高并行性

### 5.9.1 多个累计变量

使用 2 个临时变量来存储, 从而提高并行性, 这被称作 `2 * 2` 的循环展开

```cpp
void combine6(vec_ptr v, data_t *dest) {

    long length = vec_length(v);
    data_t *data = get_vec_start(v);

    data_t acc0 = IDENT;
    data_t acc1 = IDENT;


    long i;
    for (i = 0; i < length; i += 2) {
        acc0 = acc0 OP data[i];
        acc1 = acc1 OP data[i + 1];
    }

    for (; i < length; i++) acc0 = (acc0 OP data[i]);

    *dest = acc0 OP acc1;
}
```
此时要注意 OP 运算的可交换和可结合性

## 5.9.2 重新结合变换

根据结合律, 尽量减少关键路径的长度, 这称之为 `2 * 1a` 变换

```cpp
void combine7(vec_ptr v, data_t *dest) {

    long length = vec_length(v);
    data_t *data = get_vec_start(v);

    data_t res = IDENT;

    long i;
    for (i = 0; i < length; i += 2) {
        res = res OP (data[i] OP data[i + 1]);
    }

    for (; i < length; i++) res = (res OP data[i]);

    *dest = res;
}
```

## 5.10 优化合并代码的结果小结

多项优化技术确实减少了 没有优化 的 C 程序运行时间

## 5.11 一些限制因素

### 5.11.1 寄存器溢出

并行度 $p$ 超过寄存器数量, 这个时候就要用内存, 反而不如之前

### 5.11.2 分支预测和预测错误处罚

尽量使用条件传送风格的代码

## 5.12 理解内存性能

读写操作也会影响程序性能

## 5.13 应用: 性能提高技术

1. 高级设计: 选用适当的算法和数据结构
2. 基本编码原则:
    1.  消除连续的函数调用
    2. 消除不必要的内存引用
    3. 低级优化:
        1. `k * k` 的循环展开
        2. 使用条件传送风格写代码   

## 5.14 确认和消除性能瓶颈

在大型程序中, 知道哪里需要优化也是一个难题

本节使用 代码剖析程序, 收集程序性能数据并分析

## 5.14.1 程序剖析

```
gcc -Og -pg -o prog prog.c
./prog file.txt
gprof prog
```

## 5.14.2 使用剖析程序来指导优化

观察程序运行的瓶颈在哪个位置, 从而优化那个函数

## 5.15 小结

如何优化代码 \
如何找到应该优化的代码


# 练习

## 5.1 

`*xp = 0`

## 5.2 

数学题, 略

## 5.3

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5olf3rvjv8w0.webp)

## 5.4

1. 没什么不同, 只是省略了倒腾
2. 实现了, 内存别名都一样
3. 少了几步无意义的 `mov` 指令


## 5.5

```
这两条语句无关, 可以并行执行, CPE 为 5
并且与 result 无关, 可以不管他们直接执行
int tmp1 = a[i] * xpwr;
int tmp2 = x * xpwr;

上一次迭代的 tmp1, tmp2 计算到 result 中
result += tmp1;
xpwr = tmp2;
```

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.2xb6ce0y7m40.webp)

1. 2n 次乘法, n 次加法
2. 如图



## 5.6

```
这三条语句必须顺序执行, CPE 为 8
int tmp1 = x * result;

int tmp2 = a[i] + tmp1;

result = tmp2;
```

1. n 次乘法, n 次加法
2. 如上

## 5.7

```cpp
void combine5_5(vec_ptr v, data_t *dest) {

    long length = vec_length(v);
    data_t *data = get_vec_start(v);

    data_t res = IDENT;

    long i;
    for (i = 0; i < length; i += 5) {
        res = res OP data[i];
        res = res OP data[i + 1];
        res = res OP data[i + 2];
        res = res OP data[i + 3];
        res = res OP data[i + 4];
    }

    for (; i < length; i++) res = (res OP data[i]);

    *dest = res;
}
```

## 5.8

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.223emvu88q1s.png)

x, y, z 跟 r 无关, 所以可以并行计算

## 5.9

```cpp
while (i1 < n && i2 < n) {
    long v1 = src1[i1], v2 = src2[v2];
    long take1 = v1 < v2;
    dest[id++] = take1 ? v1 : v2;
    i1 += take1, i2 += (1 - take1);
}
```

## 5.10

1. $0 \le i \le 998$ `a[i] = i + 1`
2. $1 \le i \le 999$ `a[i] = 0`
3. `A` 每次读取的目标在这个函数中还没有被写, 可以并行执行
3. 类似 `A`, 理由同上

## 5.11

加法操作要依赖读操作, 写操作要依赖加法操作, 关键路径就是这个三个操作之和, 9

## 5.12

```cpp
void psum1(float a[], float p[], long n) {
    long i;
    p[0] = a[0];

    int sum = a[0];

    for (i = 1; i < n; i++) {
        sum += a[i];
        p[i] = sum;
    }
}
```

## 5.13 ~ 5.19

暂时跳过
