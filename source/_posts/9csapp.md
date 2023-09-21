---
title: CSAPP 学习笔记 (第九章)
date: 2023-08-26 14:33:23
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# Chapter 9: 虚拟内存

虚拟内存是一种对主存的抽象概念
1. 将主存看作存储在磁盘上的地址空间的高速缓存
2. 为每个进程提供一致的地址空间, 简化内存管理
3. 保护每个进程的地址空间不被其他进程破坏

## 9.1 物理和虚拟寻址

物理寻址: 通过主存中字节的编号读取  \
虚拟寻址: CPU 生成虚拟地址, 通过内存管理单元翻译成 物理地址

## 9.2 地址空间

地址空间是一个非负整数地址的有序集合, 如果地址空间中的整数是连续的, 则称为线性地址空间

CPU 从一个有 $N = 2^n$ 个地址的地址空间中生成虚拟地址, 这个地址空间称为虚拟地址空间 \
物理地址空间对应系统中物理内存的 $M$ 个字节

## 9.3 虚拟内存作为缓存的工具

VM 把虚拟内存分割成大小一定的块, 称为虚拟页, 大小为 $P = 2^p$ 字节 \
物理内存也被分割成物理页(页帧), 大小也为 $P$

虚拟页面的集合有如下三种不相交的子集
1. 未分配的: 不占用磁盘空间
2. 缓存的, 分配到物理内存中的已分配页
3. 未缓存的: 未缓存到物理内存中的已分配页

### 9.3.1 DRAM 缓存的组织结构

SRAM 缓存 指的是 CPU 和主存之间 L1, L2, L3 高速缓存 \
DRAM 缓存 指的是 虚拟内存系统的缓存, 在主存中缓存虚拟页

DRAM 缓存的不命中处罚很大, 所以使用很高的相联度 \
减少缓存行之间的竞争, 所以 DRAM 缓存使用全相联

同时虚拟页往往很大, 因为较大的页可以覆盖更多的内存区域, 提高地址转换的命中率

### 9.3.2 页表

页表将虚拟地址映射到物理地址, 是一个页表条目 (Page Table Entry, PTE) 的数组

![](https://github.com/lzlcs/image-hosting/raw/master/image.434aowhbij00.webp)

如果有效位为 1, 地址字段就指向 DRAM 中相应的物理页的起始地址 \
如果有效位为 0, 地址字段就指向 该虚拟页在磁盘上的起始位置

### 9.3.3 页命中

现在要读取 VP2 中虚拟内存的一个字 
1. 地址翻译硬件将虚拟地址作为页表的索引来定位 PTE2
2. 有效位是 1, 证明该虚拟页已经被缓存在 DRAM 中
3. 在后面的地址字段获取 VP2 对应的 PP2 在 DRAM 的物理地址

### 9.3.4 缺页

DRAM 缓存不命中 被称为 缺页

CPU 引用了 VP3 中的一个字
1. 地址翻译硬件定位 PTE3
2. 有效位是 0, VP3 在磁盘中, 触发缺页异常
3. 调用内核中的缺页异常处理程序, 选择一个牺牲页, VP4
4. 如果 VP4 被修改了, 它就会被写回到磁盘中
5. PTE3 的地址字段 指向 DRAM 中 PP3 的起始地址
6. PTE4 的地址字段 指向 虚拟内存中 VP4 的起始地址

现代操作系统都使用的 按需页面调度, 即当有不命中发生时才换入页面

### 9.3.5 分配页面

现在分配 VP5, 在磁盘上创建一个页面, 更新 页表使得 PTE5 指向该页的起始地址

![](https://github.com/lzlcs/image-hosting/raw/master/image.6icphhrha5g0.png)

### 9.3.6 又是局部性救了我们

局部性会让程序趋向于在一个较小的活动页面集合上工作, 这个集合称为工作集

好的时间局部性会让虚拟内存系统工作得很好 \
但如果工作集的大小超过了物理内存的大小, 那么程序就会抖动, 页面不断地换进换出

## 9.4 虚拟内存作为内存管理的工具

实际上操作系统为每个进程都提供了一个独立的页表

![](https://github.com/lzlcs/image-hosting/raw/master/image.57mnaz61bq80.webp)

1. 简化链接
2. 简化加载
3. 简化共享
4. 简化内存分配

## 9.5 虚拟内存作为内存保护的工具

在每个 PTE 上增加一些许可位
1. `SUP` 表示程序是否必须在管理员模式下才能运行
2. `READ` 表示是否可读
3. `WRITE` 表示是否可写

如果一些指令违反了许可条件, CPU 触发一般保护保障 \
一般称为 段错误 `Segmentation Fault`

## 9.6 地址翻译

![](https://github.com/lzlcs/image-hosting/raw/master/image.6rc9dte9fqo0.png)

有一个页表基址寄存器, 指向页表第一项
![](https://github.com/lzlcs/image-hosting/raw/master/image.27msntc6vfwg.webp)

如果有效位为 1
1. 虚拟地址被分为两半, $n-1 到 p$ 位是虚拟页号, $0 到 p-1$ 位是虚拟页偏移量
2. MMU 根据虚拟页号找到对应的 PTE, 从高速缓存 / 主存中找到 PTE
3. 根据 PTE 的地址字段找到对应的物理页号, 后面跟上虚拟页号就是真正的物理地址

因为物理页和虚拟页是一样大的, 所以偏移量也相同
![](https://github.com/lzlcs/image-hosting/raw/master/image.3vae7c31oqw0.webp)
如果有效位为 0

1. 调用缺页异常处理程序
2. 确定牺牲页, 如果它被修改了就写回到磁盘
3. 更新相应的 PTE, 重新解析虚拟地址

### 9.6.1 结合高速缓存和虚拟内存

![](https://github.com/lzlcs/image-hosting/raw/master/image.6g7t2lbr8eo0.png)

### 9.6.2 使用 TLB 加速地址翻译

MMU 中包括一个小缓存, 称作翻译后备缓冲器 (Translation Lookaside Buffer, TLB)

![](https://github.com/lzlcs/image-hosting/raw/master/image.4iq8bgkq99y0.png)

### 9.6.3 多级页表

![](https://github.com/lzlcs/image-hosting/raw/master/image.2j4jiaujx3g0.png)

一级页表有 $1024$ 项, 总大小是 $4KB$ \
二级页表同理, 这跟一页的大小正好相等, 减少了内存需求

![](https://github.com/lzlcs/image-hosting/raw/master/image.5wdqvl8ylbg0.webp)

### 9.6.4 综合: 端到端的地址翻译

![](https://github.com/lzlcs/image-hosting/raw/master/image.2v2c540qo1e0.webp)

![](https://github.com/lzlcs/image-hosting/raw/master/image.3u6wrzimc9g0.webp)

## 9.7 案例研究: Intel Core i7 / Linux 内存系统

### 9.7.1 Core i7 地址翻译

![](https://github.com/lzlcs/image-hosting/raw/master/image.33lg2o7p3gu0.png)

### 9.7.2 Linux 虚拟内存系统

**虚拟内存区域**
![](https://github.com/lzlcs/image-hosting/raw/master/image.62fnb2wg1q40.png)
**Linux 虚拟内存处理**

![](https://github.com/lzlcs/image-hosting/raw/master/image.1tzyb7hm9y8w.png)

## 9.8 内存映射

Linux 把一个虚拟内存区域和一个磁盘上的 对象关联到一起, 来初始化虚拟内存区域的内容 \
对象有两种
1. Linux 文件系统中的普通文件: 文件以页大小切片, 等到用到的时候再交换到内存中
2. 匿名文件: 全是二进制 0

### 9.8.1 再看共享对象

**共享对象**

多个进程共享同一个对象, 每个进程对该对象的写操作其他进程也可见

**私有对象**

一个进程对该对象的写操作其他进程不可见 \
使用写时复制技术, 当该进程对这个对象进行写操作时复制一份副本从而不干扰其他进程 \
同时对复制的对象进行保护, 防止其他进程访问

### 9.8.2 再看 fork 函数

`fork` 函数创建了一个副本, 把当前进程的所有对象变成私有对象, 使用写时复制

### 9.8.3 再看 execve 函数

1. 删除已存在的用户区域
2. 为新程序创建新的区域, 都是私有的, 写时复制的
3. 映射共享区域
4. 设置 PC 指向新代码入口

### 9.8.4 使用 mmap 函数的用户级内存映射

用户使用 `mmap` 创建一个新的虚拟内存区域

## 9.9 动态内存分配

当需要额外虚拟内存时, 使用动态内存分配器更方便, 可移植性更高

动态内存分配器维护一个堆, 紧接着未初始化的数据后, 并向上生长 \
分配器将堆视为一组大小不同的块的集合, 每个块是连续的虚拟内存片 \
可能是已分配的, 也可能是空闲的

**显式分配器**

要求应用程序显式释放已分配的块 \
C 语言: `malloc`, `free`
Cpp: `new`, `delete`

**隐式分配器** (垃圾收集器)

分配器检测一个块已经不再被使用, 那么就释放这个块, 这个过程叫做垃圾收集

### 9.9.1 `free` 和 `malloc` 函数

![](https://github.com/lzlcs/image-hosting/raw/master/image.3bubcv07gde0.png)

### 9.9.2 为什么要使用动态内存分配

直接使用 `int arr[MAXN]` 这种硬编码不具有广泛的适用性

更好的方法是 使用动态内存分配 `arr = (int *)malloc(n * 4)`

### 9.9.3 分配器的要求和目标

要求
1. 处理任意请求序列: 满足释放请求之前有分配请求即可
2. 立即响应请求
3. 只使用堆
4. 地址要对齐
5. 不修改已分配的块

目标
1. 最大化吞吐率: 每个单位时间内完成的请求数
2. 最大化内存利用率: 尽量减少碎片

### 9.9.4 碎片

**内部碎片**: 分配了很多, 但是没全用上 \
**外部碎片**: 分配内存剩下的小块加起来足够满足需求, 但是他们不连续

### 9.9.5 实现问题

1. 如何记录空闲块
2. 如何选择空闲块来放置新分配的块
3. 如何处理放置新块之后剩下的空闲块
4. 如何处理刚刚被释放的块

### 9.9.6 隐式空闲链表

![](https://github.com/lzlcs/image-hosting/raw/master/image.7qkkgeijc58.webp)

强加一个双字的对齐条件, 就是块大小一定是 8 的倍数, 这样二进制表示的后三位全是 0

### 9.9.7 放置已分配的块

应用请求大小为 $k$ 的块时, 分配器搜索链表找到一个足够大的空闲块 \
搜索的方式是由 **放置策略** 决定的
1. 首次适配: 搜索到第一个足够大的空闲块就分配
2. 下一次适配: 顾名思义
3. 最佳适配: 搜索整个堆, 找到最合适的放置位置

### 9.9.8 分割空闲块

1. 直接使用整个空闲块: 内部碎片较多
2. 第一部分变成分配块, 第二部分还是空闲块

### 9.9.9 获取额外的堆内存

找不到合适的空闲块的时候
1. 合并在内存中物理相邻的空闲块
2. 调用 `sbrk` 函数申请更大的堆空间

### 9.9.10 合并空闲块

分配器释放空闲块的时候, 可能有其他空闲块与之相邻

**假碎片** 空闲块被切割成许多小的空闲块从而无法使用

合并策略
1. 立即合并: 每个块被释放时就合并相邻块
2. 推迟合并: 如在分配失败的时候扫描整个堆从而合并空闲块

### 9.9.11 带边界标记的合并

![](https://github.com/lzlcs/image-hosting/raw/master/image.4gfwy1lm55e0.png)

释放当前块之后, 检查本块头部之前的 上一块的尾部, 如果为 $f$ 则合并 \
检查本块尾部 之后的 下一块的头部, 如果为 $f$ 则合并

### 9.9.12 综合: 实现一个简单的分配器

![](https://github.com/lzlcs/image-hosting/raw/master/image.6qh599fznw00.webp)

设置序言块和结尾块作为哨兵, 他们只包含头脚
```cpp
#include <cstdlib>
#include <cstdio>
#include <cmath>
#include <iostream>
#include <cassert>

using namespace std;

static char *mem_heap;
static char *mem_brk;
static char *mem_max_addr;

const unsigned MAX_HEAP = (unsigned)-1;

void mem_init(void) {
    mem_heap = (char *)malloc(MAX_HEAP);
    mem_brk = (char *)mem_heap;
    mem_max_addr = (char *)(mem_heap + MAX_HEAP);
}

void *mem_sbrk(int incr) {
    char *old_brk = mem_brk;

    if ((incr < 0) || ((mem_brk + incr) > mem_max_addr)) {
        printf("mem_sbrk error");
        return (void *)(-1);
    }

    mem_brk += incr;
    return (void *) old_brk;
}

#define MAX(a, b) ((a) > (b) ? a : b)

#define WSIZE 4
#define DSIZE 8
#define CHUNKSIZE (1 << 12)

#define PACK(size, alloc) ((size) | (alloc))

#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp) - WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp) - DSIZE)))

static void *coalesce(void *bp) {
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));

    size_t size = GET_SIZE(HDRP(bp));

    if (prev_alloc && next_alloc) return bp;
    else if (prev_alloc && !next_alloc) {

        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

   } else if (!prev_alloc && next_alloc) {

        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);

    } else {

        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));

        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    return bp;
}

static char *heap_listp;

static void *extend_heap(size_t words) {
    char *bp;
    size_t size;


    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;
    
    bp = (char *)mem_sbrk(size);

    if ((long)bp == -1) return NULL;

    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1));

    return coalesce(bp);
}

int mm_init(void) {
    mem_init();
    if ((heap_listp = (char *)mem_sbrk(4 * WSIZE)) == (void *)-1) return -1;

    PUT(heap_listp, 0);
    PUT(heap_listp + (1 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (2 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (3 * WSIZE), PACK(0, 1));
    heap_listp += (2 * WSIZE);

    return 0;
}

void mm_free(void *bp) {
    size_t size = GET_SIZE(HDRP(bp));

    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));
    coalesce(bp);
}

static void *find_fit(size_t asize) {
    void *cur_listp = heap_listp;

    while (GET_SIZE(cur_listp) != 0) {
        if (GET_SIZE(cur_listp) >= asize &&
            !GET_ALLOC(cur_listp))
            return cur_listp;

        cur_listp = NEXT_BLKP(cur_listp);
    }
   
    return NULL;
}

static void place(void *bp, size_t asize) {

    size_t rest_size = GET_SIZE(HDRP(bp)) - asize;
    char *cur_bp = (char *)bp;

    if (rest_size >= 2 * DSIZE) {

        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));

        bp = NEXT_BLKP(bp);

        PUT(HDRP(bp), PACK(rest_size, 0));
        PUT(FTRP(bp), PACK(rest_size, 0));

    } else {
    
        asize = GET_SIZE(HDRP(bp));
        PUT(HDRP(cur_bp), PACK(asize, 1));
        PUT(FTRP(cur_bp), PACK(asize, 1));
    }
}

void *mm_malloc(size_t size) {
    size_t asize;
    size_t extendsize;
    char *bp;

    if (size == 0) return NULL;

    if (size <= DSIZE) asize = 2 * DSIZE;
    else asize = DSIZE * ((size + (DSIZE) + (DSIZE - 1)) / DSIZE);

    bp = (char *)find_fit(asize);

    if (bp != NULL) {
        place(bp, asize);
        return bp;
    }

    extendsize = MAX(asize, CHUNKSIZE);

    bp = (char *)extend_heap(extendsize / WSIZE);

    if (bp == NULL) return NULL;

    place(bp, asize);

    return bp;
}
```
```cpp
#include <cstdlib>
#include <cstdio>
#include <cmath>

static char *mem_heap;
static char *mem_brk;
static char *mem_max_addr;

const unsigned MAX_HEAP = (unsigned)-1;

void mem_init(void) {
    mem_heap = (char *)malloc(MAX_HEAP);
    mem_brk = (char *)mem_heap;
    mem_max_addr = (char *)(mem_heap + MAX_HEAP);
}

void *mem_sbrk(int incr) {
    char *old_brk = mem_brk;

    if ((incr < 0) || ((mem_brk + incr) > mem_max_addr)) {
        printf("mem_sbrk error");
        return (void *)(-1);
    }

    mem_brk += incr;
    return mem_brk;
}

#define MAX(a, b) ((a) > (b) ? a : b)

#define WSIZE 4
#define DSIZE 8
#define CHUNKSIZE (1 << 12)

#define PACK(size, alloc) ((size) | (alloc))

#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

#define HDRP(bp) ((char *)(bp) - WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE(((char *)(bp) - WSIZE)))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE(((char *)(bp) - DSIZE)))

static void *coalesce(void *bp) {
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));

    size_t size = GET_SIZE(HDRP(bp));

    if (prev_alloc && next_alloc) return bp;
    else if (prev_alloc && !next_alloc) {

        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp), PACK(size, 0));
        PUT(FTRP(bp), PACK(size, 0));

   } else if (!prev_alloc && next_alloc) {

        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);

    } else {

        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));

        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        bp = PREV_BLKP(bp);
    }
    return bp;
}

static char *heap_listp;

static void *extend_heap(size_t words) {
    char *bp;
    size_t size;

    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;

    if ((long)(bp = (char *)mem_sbrk(size)) == -1) return NULL;

    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1));

    return coalesce(bp);
}

int mm_init(void) {
    if ((heap_listp = (char *)mem_sbrk(4 * WSIZE)) == (void *)-1) return -1;

    PUT(heap_listp, 0);
    PUT(heap_listp + (1 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (2 * WSIZE), PACK(DSIZE, 1));
    PUT(heap_listp + (3 * WSIZE), PACK(0, 1));
    heap_listp += (2 * WSIZE);

    return 0;
}

void mm_free(void *bp) {
    size_t size = GET_SIZE(HDRP(bp));

    PUT(HDRP(bp), PACK(size, 0));
    PUT(HDRP(bp), PACK(size, 0));
    coalesce(bp);
}

static void *find_fit(size_t asize) {
    void *cur_listp = heap_listp;

    while (GET_SIZE(cur_listp) != 0) {
        if (GET_SIZE(cur_listp) >= asize &&
            !GET_ALLOC(cur_listp))
            return cur_listp;

        cur_listp = NEXT_BLKP(cur_listp);
    }
   
    return NULL;
}

static void place(void *bp, size_t asize) {

    size_t rest_size = GET_SIZE(HDRP(bp)) - asize;
    char *cur_bp = (char *)bp;

    if (rest_size >= 2 * DSIZE) {

        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));

        bp = NEXT_BLKP(bp);

        PUT(HDRP(bp), PACK(rest_size, 0));
        PUT(FTRP(bp), PACK(rest_size, 0));

    } else {
    
        asize = GET_SIZE(HDRP(bp));
        PUT(HDRP(cur_bp), PACK(asize, 1));
        PUT(FTRP(cur_bp), PACK(asize, 1));
    }
}

void *mm_malloc(size_t size) {
    size_t asize;
    size_t extendsize;
    char *bp;

    if (size == 0) return NULL;

    if (size <= DSIZE) asize = 2 * DSIZE;
    else asize = DSIZE * ((size + (DSIZE) + (DSIZE - 1)) / DSIZE);

    if ((bp = (char *)find_fit(asize)) != NULL) {
        place(bp, asize);
        return bp;
    }

    extendsize = MAX(asize, CHUNKSIZE);

    if ((bp = (char *)extend_heap(extendsize / WSIZE)) == NULL) return NULL;
    place(bp, asize);
    return bp;
}
```

## 9.9.13 显式空闲链表

![](https://github.com/lzlcs/image-hosting/raw/master/image.p0o0bqo03eo.png)

将空闲块组织成显式的双向链表
1. 分配时间从 所有块数量的线性时间 减少到了 空闲块数量的线性时间
2. 释放一个块的时间取决于所使用的排序策略
    * LIFO, 把每个新的空闲块放在第一位, 释放一个块可在常数时间内完成, 加上边界标记之后合并也是常数
    * 地址顺序: 释放一个块需要线性时间来定位前驱, 优点是内存利用率更高

### 9.9.14 分离的空闲链表

一般的思路是把块按大小分类, 然后类似邻接表 \
每一类作为相邻的链表头, 后面跟着一系列符合大小规格的空闲块

**简单分离存储** \
每个页被分成相同大小的若干块, 比如第一页每一个字节是一块, 第二页每两个字节是一块, 第三页每四个字节是一块...依此类推
1. 每次请求分配内存时, 就使用满足要求的最小块, 不分割直接分配
2. 剩余空间不足时, 请求一个新页, 分成若干块挂在相应大小的链表上, 继续分配
3. 释放块的时候, 直接挂在相应大小的链表上, 释放的时候不会合并


优点如下:
1. 每页中的块大小相同, 所以查询地址就可以知道该块大小
2. 没有合并, 所以就不需要 $a/f$ 标记, 头脚部可以省略
3. 分配和释放都在链表开始操作, 所以不需要双向链表

缺点: 会造成极多的内部和外部碎片

**分离适配**

划定某大小区间的块为一个类
1. 请求分配内存时, 寻找当前类中合适的块, 找不到就换块大小区间更大的类 \
   分割是可选的, 分割之后剩下的块放到适合的类中
3. 释放时, 可以合并, 合并之后的块也放到合适的类中

**伙伴系统**

与分离适配大致相同, 每个大小类都是 2 的幂

## 9.10 垃圾收集

垃圾收集器就是隐式分配器

### 9.10.1 垃圾收集器的基本知识

垃圾收集器将内存视为一张有向可达图, 分为一组根节点和一组堆节点

设一个节点为 $p$ 当从根节点有一条路径可以到达 $p$ 的时候, $p$ 就是可达的 \
垃圾收集器维护可达图, 释放不可达节点到空闲链表从而实现回收功能

![](https://github.com/lzlcs/image-hosting/raw/master/image.2osnohf0xi00.png)

`ML`, `Java` 这些语言的垃圾收集器能够维护可达图的精确表示, 能够回收所有垃圾

`C`, `Cpp` 这些语言的垃圾收集器不能维护可达图的精确表示, 所以被称为保守的垃圾收集器 \
可达的块一定被标记为可达, 不可达的块不一定被标记不可达

### 9.10.2 Mark & Sweep 垃圾收集器

`Mark` 阶段遍历可达图, 把可达块做好标记 \
`Sweep` 阶段遍历堆中的块, 把没标记过的块释放

### 9.10.3 C 程序的保守 Mark & Sweep

C 语言不会在内存中标记类型, 所以无法判断这个数据是不是指针

在 `Mark` 阶段, 无法判断指针是否有效, 所以就只能全都标记为有效块

## 9.11 C 程序中常见的与内存有关的错误

### 9.11.1 间接引用坏指针

访问无意义的地址空间, 引发段错误 \
第二句把 `val` 的值当作地址, 非常容易引发错误
```cpp
scanf("%d", &val);
scanf("%d", val);
```

### 9.11.2 读未初始化的内存

全局变量总是被设置为 0, 但是在堆中的变量不一定 \
需要显式设定变量, 或者使用 `calloc`

### 9.11.3 允许栈缓冲区溢出

写入超过字符数组大小的字符串, 造成缓冲区溢出

### 9.11.4 假设指针和他们指向的对象是相同大小的

指针的大小跟所指向的对象大小不一定相同

```cpp
int **A = (int **)malloc(n * sizeof(int));
int **A = (int **)malloc(n * sizeof(int *));
```
第一句话可能在释放这个块时产生错误, 并且很难找出来

### 9.11.5 造成错位错误

```cpp
int a[10]{};
int b[10]{};

a[12] = 2;

cout << b[0] << endl;
```

可能修改 `b` 数组

### 9.11.6 引用指针, 而不是它指向的对象

```cpp
int x = 0;
int *y = &x;

cout << x << ' ' << y << endl;

*y--;

cout << x << ' ' << y << endl;

y = &x;
(*y)--;

cout << x << ' ' << y << endl;
```

注意运算符优先级

### 9.11.7 误解指针运算

指针的算数操作是相对于它们所指向的对象的大小为单位来进行的

### 9.11.8 引用不存在的变量

```cpp
int *stackref() {
    int val;
    return &val;
}
```

当调用完这个函数的时候, 栈帧已经被修改, `&val` 指向的不再是 `val` 

当后面在对 这个地址上的值进行修改时, 可能会对另一个函数造成影响

### 9.11.9 引用空闲堆块中的数据

再次访问已经被释放了的内存


### 9.11.10 引起内存泄漏

分配了一些空间而没有正确释放, 导致系统可用内存逐渐减少, 最终导致系统性能下降或者崩溃

### 9.12 小结

虚拟内存是对主存的一个抽象, 处理器使用虚拟地址来引用主存 \
有硬件使用页表来把 虚拟地址翻译成 物理地址

程序使用动态内存分配器来操作内存, 有显式分配器和隐式分配器两种


# 练习

## 9.1 

略

## 9.2

$PTE 数量 = 2^n / P$

## 9.3 

虚拟页偏移量和物理页偏移量位数相同 \
根据页大小确定 偏移量的位数, 用总位数减去即可

## 9.4

![](https://github.com/lzlcs/image-hosting/raw/master/image.4lsph5f37do0.webp)

## 9.5

```cpp
#include <unistd.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>

void mmapcopy(int fd, int size) {

    char *bufp = (char *)mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0);
    write(STDOUT_FILENO, bufp, size);

    munmap(bufp, size);

    return;
}

int main(int argc, char **argv) {

    int fd = open(argv[1], O_RDONLY, 0);

    struct stat st;
    fstat(fd, &st);

    mmapcopy(fd, st.st_size);

    close(fd);

    return 0;
}
```

## 9.6

请求的字节再加上 4(头部), 向上取最接近 8 倍数的数, 即为块大小

## 9.7 

有头有脚 8 + 1, 有头无脚 4 + 1 \
单字舍入 4, 双字舍入 8

## 9.8 ~ 9.9

见正文

## 9.10 

每次请求和释放不同大小的块

## 9.11 ~ 9.13 

略

## 9.14


注意 `open` 和 `mmap` 函数的各种参数变化

```cpp
#include <unistd.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <cstdio>
#include <fcntl.h>

void change(int fd, int size) {

    char *bufp = (char *)mmap(NULL, size, PROT_WRITE, MAP_SHARED, fd, 0);

    *bufp = 'J';

    munmap(bufp, size);

    return;
}

int main(int argc, char **argv) {

    int fd = open(argv[1], O_RDWR, 0);

    struct stat st;
    fstat(fd, &st);

    change(fd, st.st_size);

    close(fd);

    return 0;
}
```
## 9.15

块大小是原 `size + 4` 向上 8 的倍数取整, `head` 是十进制大小转换为二进制再 或 `0x1`

## 9.16

空闲块头脚都有, 所以最小都是 16

## 9.17

```cpp
static void *find_fit(size_t asize) {
    void *save_ptr = last_find;

    while (GET_SIZE(last_find) != 0) {
        if (GET_SIZE(last_find) >= asize &&
            !GET_ALLOC(last_find))
                return last_find;
        last_find = NEXT_BLKP(last_find);
    }

    while (last_find < save_ptr) {
        if (GET_SIZE(last_find) >= asize &&
            !GET_ALLOC(last_find))
                return last_find;
        last_find = NEXT_BLKP(last_find);
    }
   
    return NULL;
}
```

## 9.18

跳过

## 9.19 ~ 9.20

略

