---
title: CSAPP 学习笔记 (第八章)
date: 2023-09-01 17:23:00
mathjax: true
tags:
- C
- CSAPP
categories: 
- Book
---

# Chapter 8: 异常控制流

异常控制流 `Exceptional Control Flow, ECF`
1. 理解 ECF 帮助你理解重要的系统概念
2. 理解 ECF 帮助你理解应用程序是如何与操作系统交互的
3. 理解 ECF 帮助你编写有趣的新应用程序
4. 理解 ECF 帮助你理解并发
5. 理解 ECF 帮助你理解软件异常如何工作

## 8.1 异常

**异常** 控制流的突变, 用来响应处理器状态中某些变化

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.7cjcua34j1o0.png)

**事件** 状态变化

每当有异常发生时, 处理器通过异常表来找到对应的异常处理程序
1. 处理程序返回到 $I_{curr}$
2. 处理程序返回到 $I_{next}$
3. 处理程序中断原本程序

### 8.1.1 异常处理

每种类型的异常都有一个异常号, 有些是处理器设计者分配的, 有些是操作系统内核分配的

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.4nyapbmz38u0.png)

异常表的起始地址放在一个 **异常表基址寄存器** 的特殊 CPU 寄存器中

异常类似于过程调用, 但有些不同
1. 异常的返回地址是当前指令或者下一套指令, 过程调用中处理器把返回地址压入栈中
2. 处理器把一些额外的状态压入到栈中
3. 控制从用户程序转移到内核, 所有的项目被压到内核栈中, 而非用户栈
4. 异常处理程序运行在内核模式下, 可以访问任何系统资源

### 8.1.2 异常的类别

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1bx3tew0w7k0.png)

**中断**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1u9elplf1nb4.png)

**陷阱和系统调用**

系统调用: 在用户程序和内核之间提供一个像过程的接口

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6x8g7yx7dow0.png)

普通的函数运行在用户模式下, 只能访问与调用函数相同的栈 \
系统调用运行在内核模式下, 可以访问任意系统资源

**故障**

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.4slop7tpd8u0.png)

典型的故障是缺页异常

**终止**

致命错误, 终止程序

### 8.1.3 Linux/x86-64 系统中的异常

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.4htjh5tsey20.webp)

汇编中使用 `syscall` 进行系统调用, 编号存储在 `%rax` 中

## 8.2 进程

进程: 一个执行中的程序的实例

### 8.2.1 逻辑控制流

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5rmf3bsnpoo0.png)

进程是轮流使用处理器的, 每个进程的每段指令之间看起来有一点时间间隔 \
实际上是 CPU 将当前程序挂起, 转而执行其他进程, 执行完毕之后再切换回来

### 8.2.2 并发流

并发流: 一个流的执行在时间上去另一个流重叠, 两个流并发运行 \
并发: 多个流并发执行的一般现象 \
多任务: 一个进程与其他进程轮流进行的概念 \
时间片: 一个进程执行它的控制流的一部分的每一时间段

并行流: 两个流并发地运行在不同的处理器核或者计算机上

### 8.2.3 私有地址空间

进程为每个程序提供一种假象, 每个程序独占地使用系统内存空间 \
进程为每个程序提供私有地址空间, 基本格式相同

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5y3ky9m9ivk0.png)

### 8.2.4 用户模式与内核模式

使用某个寄存器中的模式位来提供这种功能, 模式位设置为 1 时启用内核模式

运行应用程序最开始都是在用户模式, 进入内核模式的唯一方法是异常

`Linux` 通过 `/proc` 文件系统允许用户模式访问内核数据结构 \
具体是把内核数据结构的输出放到这个目录下

### 8.2.5 上下文切换

使用上下文切换这样的异常控制流实现多任务

内核为每个进程维护一个上下文, 内核重新启动一个被暂挂的进程所需的状态

调度: 内核决定抢占进程并重新开始一个之前被抢占的进程的决策 \
通过上下文切换的机制来将控制转移到新的进程

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5shxebi3p600.webp)

## 8.3 系统调用错误处理

Unix 系统级函数遇到错误时, 通常返回 -1, 设置全局整数变量 `errno`

使用错误处理来包装函数, 这样可以简化代码

## 8.4 进程控制

### 8.4.1 获取进程 ID

每个进程都有唯一的无符号进程 (PID) \
`getpid()` 返回当前进程的 PID, `getppid()` 返回父进程的 PID

```
#include <sys/types.h>
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);
```

### 8.4.2 创建和终止进程

进程处在三种状态之一
1. 运行: 正在被执行或者等待被执行
2. 停止: 运行的进程被挂起
3. 终止: 进程永远停止

使用 `fork()` 函数创建进程
1. 调用一次返回两次: `fork()` 函数有两个返回值, 对于父进程返回子进程 ID, 对于子进程返回 0
2. 并发执行: 不能假定父进程和子进程执行的顺序
3. 相同但是独立的地址空间: 子进程复制一遍父进程的地址空间, 随后的修改都时互不干扰的
4. 共享文件: 子进程继承了父进程所有的打开文件

### 8.4.3 回收子进程

进程终止时, 内存并没有马上消除它, 内存把它冻起来, 等父进程来回收

终止了还未回收的进程称为僵死进程

父进程终止了, 孤儿进程由 `init` 进程接管

`init` 进程 PID 是 1, 系统启动时内核创建的, 不会终止, 是所有进程的祖先

父进程如果没有回收僵死子进程就终止了, 那么就让 `init` 回收它

`man waitpid`

### 8.4.4 让进程休眠

`man pause`, `man sleep`

只要休眠进程收到一个未被忽略的信号, `sleep` 函数就会提前返回

### 8.4.5 加载并运行程序

`execve` 函数在当前进程的上下文中加载并运行一个新程序

`int main(int argc, char **argv, char **envp);`

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6n50pv0ef980.png)

`getenv` 和 `unsetenv` 函数分别是 寻找环境变量和删除环境数组变量的函数

### 8.4.6 利用 fork 和 execve 运行程序

给出简单的有缺陷的 `shell` 实现, 缺陷是不回收子进程

使用信号修复缺陷

## 8.5 信号

信号提供一种机制, 通知用户进程 发生了这些异常

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.4dh6hijkln20.webp)

### 8.5.1 信号术语

1. 发送信号: 内核更新进程上下文的某个状态, 发送信号给目标进程的过程 \
   发送信号有如下两种原因: 系统检测到一个事件 / 一个进程调用了 `kill` 函数
2. 接收信号: 目的进程被内核强迫 对信号的发送做出反应 \
   进程可以执行一个信号处理程序的用户层函数捕获这个信号 \
   ![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.7fwkde0xfvg0.webp)

**待处理信号**: 发出而没有被接收的信号

一个进程中一个类型至多有一个待处理信号, 更多的待处理信号被丢弃

### 8.5.2 发送信号

**进程组** 每个进程属于一个进程组, 默认子进程和父进程一个进程组

`getpgrp()` 返回当前进程的进程组 ID \
`setpgid(pid, pgid)` 改变进程组

**用 `/bin/kill` 程序发送信号**

`/bin/kill -9 15213` 把信号 9 发送到 进程 15213 中 \
`/bin/kill -9 -15213` 把信号 9 发送到 进程组 15213 中所有的进程

**从键盘发送信号**

作业 (job) 表示为一条命令行求值而创建的进程

键入 `Ctrl-C` 终止前台作业, `Ctrl-Z` 停止(挂起)前台作业

**函数 `kill` 发送信号**

**函数 `alarm` 发送信号**

### 8.5.3 接收信号

当内核把进程 $p$ 从内核模式切换到用户模式的时候, 它会检查未被阻塞的待处理信号的集合

1. 集合为空: $I_{next}$
2. 集合不空: 选定集合中的最小的信号 $k$ 强制 $p$ 接收信号 $k$

进程可以使用 `signal` 函数来修改和信号相关联的默认行为 \
`SIGSTOP` 和 `SIGKILL` 的默认行为不能修改

`man signal`

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.7fsto8guq1g0.webp)

### 8.5.4 阻塞和解除阻塞信号

1. 隐式阻塞机制: 内核默认阻塞当前信号处理程序所处理的信号类型 的待处理信号
2. 显式阻塞机制: 使用一些辅助函数明确说明阻塞信号和解除阻塞信号

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5rr1oyus6m00.webp)

### 8.5.5 编写信号处理程序

**安全的信号处理**

事件处理程序和主程序并发运行, 如果两个程序并发地访问全局数据结构, 那么结果可能无法预测
1. 处理程序要尽可能简单: 例如处理程序设置个全局标志, 主程序周期性检查并处理
2. 处理程序中只调用异步信号安全的函数 \ 
   异步信号安全函数: 可重入 / 不能被信号处理程序中断
   ![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6ssqpltx58w0.webp)
2. 保存和回复 `errno`: 防止处理程序中有函数改变 `errno`
1. 访问全局数据结构时阻塞所有的信号
1. 用 `volatile` 声明全局变量
1. 用 `sig_atomic_t` 声明标志

**正确的信号处理**

未处理的信号是不排队的, 编写程序时要考虑不排队的问题

**可移植的信号处理**

不同的系统可能有不同的信号处理语义

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.5r15fpjhp8w0.webp)

使用此函数指定信号处理语义

### 8.5.6 同步流以避免讨厌的并发错误

处理相同位置数据的 并发程序 一向是个难题

**竞争** 是一种同步错误, 两个有操作顺序要求的函数, 因为并行而乱了顺序

父进程使用阻塞 `SIGCHLD`, 子进程再取消阻塞

### 8.5.7 显式地等待信号

使用 `sigsuspend` 函数来挂起进程, 并在收到信号的时候恢复进程执行

## 8.6 非本地跳转

非本地跳转 将控制直接从一个函数转移到另一个函数

`man setjmp`, `man longjmp`

## 8.7 操作进程的工具

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.1nhlwypex6yo.png)

## 8.8 小结

异常控制流

1. 硬件层: 处理器中的事件引发, 控制流转移到异常处理程序
2. 操作系统层: 内核用 ECF 提供进程的基本概念
3. 应用程序, 子进程, 运行程序, 捕获信号
4. 非本地跳转



# 练习

## 8.1

对错对

## 8.2

```
// 子进程
p1: x=1
p2: x=2
// 父进程
p2: x=0
```

## 8.3

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.4xlz1xpgbu40.webp)

`acbc`, `abcc`, `bacc` 都有拓扑序

## 8.4

![](https://cdn.staticaly.com/gh/lzlcs/image-hosting@master/image.6xop3ignvfc0.webp)

6 行 \
任何符合拓扑序的输出都可以

## 8.5

```c
unsigned int snooze(unsigned int secs) {
    int rear_secs = sleep(secs);
    printf("Slept for %d of %d secs.\n", secs - rear_secs, secs);
    return rear_secs;
}
```

## 8.6

```c
#include <stdio.h>

int main (int argc, char *argv[], char *envp[]) {

    printf("Command-line arguments:\n");

    for (int i = 0; i < argc; i++)
        printf("    argv[%2d]: %s\n", i, argv[i]);

    puts("");
    printf("Environment variables:\n");

    for (int i = 0; envp[i]; i++)
        printf("    envp[%2d]: %s\n", i, envp[i]);

    
    return 0;
}
```

## 8.7

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <stdlib.h>

void sigint_handler(int sig) {
    return;
}

unsigned int snooze(unsigned int secs) {
    int rear_secs = sleep(secs);

    printf("Slept for %d of %d secs.\n", secs - rear_secs, secs);

    return rear_secs;
}

int main (int argc, char *argv[]) {
    
    if (signal(SIGINT, sigint_handler) == SIG_ERR) {
        printf("signal error");
        _exit(0);
    }

    snooze(atoi(argv[1]));

    return 0;
}
```

## 8.8

`213`

## 8.9

只有第一行不是并发的

## 8.10

1. `fork`
2. `longjmp`, `execve`
3. `setjmp`

## 8.11

4

## 8.12

8

## 8.13

父进程 43, 子进程 2

每次输出的 $x$ 分别为
```
432
423
243
```

## 8.14

3

## 8.15

5

## 8.16

```
counter = 2
```

## 8.17

略

## 8.18

ACE

## 8.19

$2^n$

## 8.20

```c
#include <unistd.h>

int main (int argc, char *argv[], char *envp[]) {
    
    execve("/bin/ls", argv, envp);

    return 0;
}
```

## 8.21

`abc`, `bac`

## 8.22

```c
int mysystem(char *command) {
    pid_t pid;
    int status;

    if ((pid = fork()) == 0) {
        char *argv[4] = { "", "-c", command, NULL };
        execve("/bin/sh", argv, environ);
    }

    printf("child pid: %d\n", pid);

    if (waitpid(pid, &status, 0)) {
        if (WIFEXITED(status))
            return WEXITSTATUS(status);

        if (WIFSIGNALED(status))
            return WTERMSIG(status);
    }

    return -1;
}
```

## 8.23

信号没有排队机制

第一个信号处理时, 第二个信号是未处理状态, 第三四五信号被忽略

最终只处理两个信号

## 8.24


```c
while ((pid = waitpid(-1, &status, 0) > 0)) {
    if (WIFEXITED(status))
        printf("child %d terminated normally with exit status=%d\n", pid, WEXITSTATUS(status));
    else if (WIFSIGNALED(status)) {
        char* sigErr = strsignal(WTERMSIG(status))
        psignal(WTERMSIG(status), sigErr);
    }
}
```

## 8.25

```c
void sigalrm_handler(int sig) {
    if (alarm(0) == 0) _exit(0);
}

char* tfgets(char *str, int n, FILE* stream) {
    signal(SIGALRM, sigalrm_handler);
    alarm(5);
    fgets(str, n, stream);
    return str;
}
```

## 8.26

`shell lab`
