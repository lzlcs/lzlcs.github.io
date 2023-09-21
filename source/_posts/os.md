---
title: 操作系统
date: 2023-09-10 10:33:23
mathjax: true
tags:
- C
- OS
categories: 
- Base
---

# 操作系统介绍

操作系统提供很多 `API` 给应用程序从而访问设备, 所以 OS 有时被称为标准库

**虚拟化**: 将物理资源转换为更通用的虚拟形式

1. 虚拟化 CPU: 将单个 CPU 转化成看似无限数量的 CPU
2. 虚拟化内存: 每个进程访问自己的虚拟地址空间

**并发**: 同时处理很多事情

在运行多线程程序时, 可能会出现并发问题导致运行结果呢出错

**持久性**: 硬件和软件持久地存储数据

文件系统: 操作系统中管理磁盘的系统

**设计目标**
1. 建立抽象, 使系统更方便被使用
2. 高性能
3. 保护程序和硬件

# 抽象: 进程

## 内容

进程是运行中的程序

共享资源使用的最基本技术
1. 时分共享: 允许资源使用物理实体一段时间, 再由另一个资源使用一小段时间
2. 空分共享: 资源在空间上被划分给使用它的人

存在一些策略调度这些进程

**抽象: 进程**

进程的机器状态组成: 进程可以访问的内存空间(地址空间), 寄存器状态

**进程 `API`**

操作系统必须提供的接口:
1. 创建新进程
2. 销毁进程
2. 等待进程运行
1. 其他控制
1. 获取进程状态

**进程创建: 更多细节**

1. 将所有代码和静态数据放到内存中
    - 早期操作系统是在运行程序之前把这些都放到内存中
    - 现代操作系统使用惰性加载: 用到了再放到内存中
2. 为运行时栈分配内存: 局部变量, 函数参数和返回地址
    - 操作系统可能使用参数初始化栈: `argc`, `argv`
1. 为运行时堆分配内存: 用于动态分配数据
1. 执行一些其他操作, 比如 IO 设备相关的操作
1. 启动程序: 跳转到 `main` 函数

**进程状态**

1. 运行: 正在执行指令
2. 就绪: 准备好运行, 但是操作系统选择先做别的事
1. 阻塞: 直到发生一些其他事情时才会开始运行

![](https://github.com/lzlcs/image-hosting/raw/master/image.1nblosu9jtxc.webp)

**数据结构**

1. 保存寄存器值的数据结构
2. 保存进程的数据结构

## 作业

1. CPU 利用率为 100%
2. 9
3. 在等待 IO 的过程中运行进程 2, 可以节省时间, 总时间为 5
4. 同 问题 2 的时间需求, 产生浪费
5. 同 问题 3 的时间需求, 得到很高利用率
6. 先进行进程 234 的操作, 而回到进程 1 的时候, CPU 资源被浪费
7. 优先 IO, CPU 资源利用率升高
8. 随机问题

# 插叙: 进程 `API`

## 内容

**`fork()`系统调用**

创建子进程, 有独立的地址空间 \
父进程的 `fork()` 返回值非零, 子进程 `fork()` 返回值为 0

子进程和父进程可能会并发运行, 从而无法得到确定的结果

**`wait()`系统调用**

在父进程部分的代码调用 `wait(NULL)` 即可等到子进程结束

这样就会得到确定的结果

**`exec()`系统调用**

允许子进程运行其他的程序

把子进程的地址空间里放进 目标程序的地址空间 \
堆, 栈, 代码静态数据都会重新初始化 \
这样子进程在运行 `exec` 函数之后的代码就永远不会被执行

**为什么这样设计`API`**

允许程序运行 `fork()` 和 `execve()` 之间的代码 \
可以进行一些设置环境之类的操作

`shell` 执行程序: 
1. 用户输入命令
1. `fork()` 创建子进程
2. `exec()` 或者某个变体运行目标程序
3. `wait()` 等待程序完成
4. 继续输出提示符等待下一步输入

`shell` 进行重定向: 在调用 `exec()` 之前关闭 `stdout` 打开文件即可

管道也是使用类似的方式实现, 不过使用的是 `pipe()` 系统调用

**其他`API`**

`kill()` 向进程发送信号, 要求进程睡眠终止之类的指令

## 作业

**5.1**
```c
#include <stdio.h>
#include <unistd.h>

int main() {

    int x = 100;

    int rc = fork();

    if (rc == 0) {
        printf("child: \n");
        x = 101;
        printf("after: %d\n", x);
        
    } else {

        printf("parent: \n");
        x = 102;
        printf("after: %d\n", x);
    }

    return 0;
}
```
父子进程之间有独立的地址空间, 更改变量不会对另一个进程有什么影响

**5.2**

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {

    int fd = open("example.txt", O_CREAT | O_WRONLY, 0644);

    int rc = fork();

    if (rc == 0) {
        char *s = "child write\n";
        int tmp = write(fd, s, strlen(s));
    } else {
        char *s = "parent write\n";
        int tmp = write(fd, s, strlen(s));
        wait(NULL);
        close(fd);
    }

    return 0;
}
```

地址空间独立, 互相写入的操作不会被干扰 \
但是并发写入文件可能会出现顺序问题

**5.3**

```c
#include <stdio.h>
#include <unistd.h>

int main() {

    int rc = fork();

    if (rc == 0) {
        printf("hello\n");
    } else {
        sleep(1);
        printf("goodbye\n");
    }

    return 0;
}
```

使用 `sleep()` 函数取巧

**5.4**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    int rc = fork();

    if (rc == 0) {
        execl("/bin/ls", "-l", NULL);
    } else {
        wait(NULL);
    }

    return 0;
}
```

变种多的原因是参数的种类数量不同

**5.5**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    int rc = fork();

    if (rc == 0) {
        int tmp = wait(NULL);
        printf("child wait output: %d, pid: %d\n", tmp, getpid());
        _exit(0);
    } else {
        int tmp = wait(NULL);
        printf("parent wait output: %d\n", tmp);
    }

    return 0;
}
```

父进程 `wait` 返回子进程的进程号 \
子进程返回 `-1`

**5.6**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    int rc = fork();

    if (rc == 0) {
        int tmp = waitpid(rc, NULL, 0);
        printf("child wait output: %d, pid: %d\n", tmp, getpid());
        _exit(0);
    } else {
        int tmp = waitpid(rc, NULL, 0);
        printf("parent wait output: %d\n", tmp);
    }

    return 0;
}
```

`waitpid` 提供更精细的控制

**5.7**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    int rc = fork();

    if (rc == 0) {
        close(STDOUT_FILENO);
        printf("child\n");
    } else {
        printf("parent\n");
    }

    return 0;
}
```

子进程不会输出到屏幕上了

**5.8**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {

    int pipe_fd[2];
    int rc1, rc2;

    pipe(pipe_fd);

    rc1 = fork();

    if (rc1 == 0) {
        close(pipe_fd[0]);
        dup2(pipe_fd[1], STDOUT_FILENO);

        execlp("echo", "echo", "Hello", NULL);
        _exit(1);

    } else {

        waitpid(rc1, NULL, 0);

        rc2 = fork();

        if (rc2 == 0) {
            close(pipe_fd[1]);
            dup2(pipe_fd[0], STDIN_FILENO);

            int x = execlp("cat", "cat", NULL);
            
            _exit(1);
        } else {

            close(pipe_fd[0]), close(pipe_fd[1]);

            waitpid(rc2, NULL, 0);
        }
    }

    return 0;
}
```

# 机制: 受限直接执行

## 内容

在保持控制的同时实现高性能

**基本技巧: 受限直接执行(LDE)**

![](https://github.com/lzlcs/image-hosting/raw/master/image.8tk9ru1ygps.png)

**问题 1: 受限制的操作**

操作系统使用 内核模式 \
应用程序使用 用户模式 

内核模式具有所有权限


![](https://github.com/lzlcs/image-hosting/raw/master/image.66adso3ctbc0.webp)

LDE 协议的两个阶段
1. 系统引导时初始化陷阱表
2. 运行进程时如图

**问题 2: 在进程之中切换**

协作方式: 等待系统调用

应用将 CPU 控制权转移给操作系统的方式
1. 通过系统调用, 友好地交还控制权
2. 执行非法操作
被动地等待应用交还会存在隐患

--------

非协作方式: 操作系统进行控制

使用时钟中断的方式: 每隔几毫秒产生中断, 进程将 CPU 控制权转移给操作系统

-------

保存和恢复上下文

调度进程来决定 操作系统重新获得控制权之后的 操作

上下文切换: 为当前执行的进程保存一些寄存器的值, 为即将执行的进程恢复一些寄存器的值

![](https://github.com/lzlcs/image-hosting/raw/master/image.4r6z6euckay0.webp)

此协议中有两种类型的寄存器 保存 / 恢复
1. 时钟中断时: 硬件隐式保存用户寄存器
2. 进程切换时: 内核寄存器被 OS 明确保存

**并发问题**

中断的时候不允许发生另外的中断, 还有一些锁机制来防止并发错误的产生


## 作业

**测量系统调用**

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/time.h>

int main() {

    int fd = open("q1.c", O_RDONLY);

    char buf[256];

    struct timeval start, end;

    gettimeofday(&start, NULL);

    for (int i = 0; i < 1e7; i++) {
        read(fd, buf, 0);
    }

    gettimeofday(&end, NULL);

    printf("%d\n", (int)(end.tv_usec - start.tv_usec));

    return 0;
}
```

一百万次系统调用大概 0.6s

**测量上下文切换**

暂时跳过

# 进程调度: 介绍

## 内容

**工作负载假设**

1. 假设每个工作运行相同时间
2. 所有的工作同时到达
3. 一旦开始, 每个工作保持运行直到完成
4. 所有的工作只使用 CPU
5. 每个工作的运行时间已知

**调度指标**

性能指标: 
$$
T_{周转时间} = T_{完成时间}-T_{到达时间}
$$
公平指标:
$$
T_{响应时间} = T_{首次运行}-T_{到达时间}
$$


**存在五条限制**

`FIFO`: 先进先出式运行进程

**取消假设 1**

`FIFO` 如果先运行一个时间很长的进程, 那么平均周转时间就会很高 \
`SJF` 最短任务优先: 可以证明在 2345 条限制之下这个调度方案最优

**取消假设 2**

`SJF` 当前执行一个很长的任务, 中途来了几个很短的任务, 此时平均周转时间很高 \
`STJF` 最短完成时间优先: 剩余的完成任务时间最短的最先运行, 这样可以大大提高周转时间 \
可以证明在 345 条限制下这个调度方案最优

**取消假设 3**

此时的调度程序变成抢占式的, 可以暂停一个进程开始另一个进程

`PP` 轮转: 在一个时间片(通常为中断周期的整数倍)内执行一个进程, 下一个时间片轮转地执行其他进程, 
这样的好处是可以减少响应时间, 也就是公平性, 坏处是增加周转时间, 也就是低性能

摊销: 上下文切换的时间开销占比过大时, 可以增大时间片从而摊销上下文时间切换

**取消假设 4**

在 IO 期间进程不会使用 CPU, 所以可以把 IO 时间用来执行其他程序 \
根据 IO 的分布, 把进程划分成一些子进程, 把他们都当作单独的进程来调度

**取消假设 5**

结合上面的所有想法, 平衡响应时间和周转时间

## 作业

123 略过

4. 按照运行时间从小到大的到达顺序的工作负载
5. 时间片的长度大于最长的进程
6. 逐渐增大
7. $$
    \frac{q\Sigma_{i = 0}^{n-1}i}{N} = \frac{q(N-1)}{2}
    $$

# 调度: 多级反馈队列

## 内容

`Multi-level Feedback Queue(MLFQ)` 多级反馈队列

**`MLFQ` 的基本规则**

`MLFQ` 中有许多队列, 每个队列有不同的优先级, 每个进程只属于一个队列
1. A 的优先级比 B 高, 运行 A
2. A 的优先级与 B 相同, 轮转运行 AB

**尝试 1: 如何改变优先级**

3. 工作进入系统时使用最高优先级
4. 工作用完整个时间片只有, 降低优先级 \
   工作在时间片中主动释放 CPU, 优先级不变

缺点:
1. 饥饿问题: 太多短工作使得长工作饿死
2. 愚弄调度程序: 时间片的 99% 访问无关文件, 从而一直保持在高优先级

**尝试 2: 提升优先级**

5. 经过一段时间, 就将所有工作加入最高优先级队列

**尝试 3: 更好的计时方式**

重写第四条:

4. 一旦工作用完了其在某一层的时间配额, 那就降低优先级

**`MLFQ` 调优**

比如不同优先级队列可变的时间片长度

## 作业(略)

# 调度: 比例分额

每隔一段时间抽奖决定下一个应该运行的进程 \
根据进程需要更改概率

**基本概念: 彩票数表示份额**

彩票数占总彩票数的百分比代表进程占用资源的份额

随机性的好处:
1. 几乎遇不到最差的情况
2. 几乎不需要记录除彩票数的其他状态
3. 迅速

**彩票机制**

1. 彩票货币: 分给子进程的货币按照比例兑换全局彩票
2. 彩票转让: 某个进程想让另一个进程更快, 可以进行彩票转让
3. 彩票通胀: 用于进程之间相互信任的环境, 临时提升自己的彩票数量

**实现**

把进程按照彩票数从多到少排序, 按顺序遍历从而找到下一个运行的进程

**分配彩票**

除了用户自己分配之外, 分配问题到现在还没有最佳答案

**步长调度: 一个确定性的公平分配算法**

1. 每个进程用一个大数除以彩票数, 得到步长, 总步长设为 0
2. 每次选择所有进程中总步数最小的开始运行一段时间
3. 运行完毕之后总步数加上步长, 随后回到第二步

**比例分额模式的缺点**

1. 无法确定各个进程应该有多少彩票数
2. 不能很好适配 IO 操作

## 作业

2. 彩票数量不平衡时, 彩票数量多的几乎是一直运行到结束 \
   出现饥饿现象
3. 不公平性不多
4. 不公平性增大

# 多处理器调度

暂时搁置, 学完并发再回来读




# 抽象: 地址空间

**早期系统**

![](https://github.com/lzlcs/image-hosting/raw/master/image.48xmqeb3hbc0.png)

**多道程序和时分共享**

一种粗糙的机器共享(慢): 一个程序使用全部内存, 上下文切换的时候把程序数据放到硬盘中 \
加载其他程序的信息再继续运行

**地址空间**

地址空间包括运行程序的所有内存状态: 包括代码段, 栈段, 堆段等等

**目标**

1. 透明: 让运行的程序不知道内存被虚拟化了
2. 效率: 追求更低的时间空间复杂度
3. 保护: 保护进程之间相互隔离, 不受其他进程影响

程序中打印的地址都是虚拟地址, 需要操作系统翻译成物理地址

# 插叙: 内存操作 `API`

**内存类型**

栈内存: 申请和释放操作由编译器隐式管理, 也被称为自动内存 \
堆内存: 申请和释放操作由程序员显式管理

**`malloc()`调用**

```c
#include <stdlib.h>
void *malloc(size_t size);
```
传入参数的时候使用 `sizeof()` 和 `strlen(s) + 1` \
`malloc` 返回 `void *`, 所以最好是显式写出强制类型转换

**`free()` 调用**

释放不使用的堆内存: `free(x)`


