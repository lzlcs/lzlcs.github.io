---
title: Pintos Project 1
date: 2023-09-25 15:28:00
mathjax: true
tags:
- C
- OS
- Lab
categories: 
- Labs
---

# Part I: Alarm Clock

**开关中断来保证操作的原子性**

以 `timer.c` 中 `timer_ticks()` 函数举例

```c
int64_t timer_ticks (void) 
{
    enum intr_level old_level = intr_disable ();
    int64_t t = ticks;
    intr_set_level (old_level);
    return t;
}
```

调用链中一些关键函数
1. `intr_disable()` 
    - 保存之前的中断状态
    - 使用汇编关中断
2. `intr_enable()`
    - 保存之前的中断状态
    - 使用汇编开中断
3. `intr_get_level()`
    - 把表示是否中断的寄存器 `flag` 压栈, 再弹出到 `flags` 变量中
    - 根据 `flag` 值获得 当前中断的状态: 是否开启
4. `intr_set_level()`
    - 调用 `intr_disable()` 和 `intr_enable()`
5. `intr_context()`: 返回是否是外中断(I/O 等)

`timer_ticks` 中先关闭中断, 获取 `ticks` 值, 最后把中断设置成初始状态

**进程状态的转换**

1. 运行态 至 阻塞态: `thread_block()`
2. 阻塞态 至 就绪态: `thread_unblock()`
3. 运行态 至 就绪态: `thread_yield()`
4. 就绪态 至 运行态: 需要 `schedule()` 函数调度

**时钟中断的处理**

在 `timer_interrupt` 函数中
- `ticks++` 记录系统时间片
- 调用 `thread_tick` 记录进程的时间片 \
    如果当前进程使用的时间片超出限制 \
    那么当中断处理结束的时候把该进程放入就绪队列

**最初 `timer_sleep` 中的 忙等 实现**

在间隔时间未到 形参 `ticks` 的情况下不断执行 `thread_yield()` 函数

实际的运行结果就是 在时间到达 `ticks` 的情况下, 该进程处在就绪状态即可

## 实现思路

在执行 `timer_sleep` 的时候, 把当前进程阻塞 \
在进程结构体中使用 `blocked_ticks` 变量记录剩余阻塞的 `ticks`

每次处理时钟中断的时候, 使用 `thread_foreach` 函数遍历每个进程并检测 `blocked_ticks` 值

1. `thread.h` 中
```c
 struct thread
  {
    // ...
    uint32_t blocked_ticks;
    /* Owned by thread.c. */
    unsigned magic;
  };
```
2. `thread.c` 中, `blocked_ticks` 值初始化为 0
```c
static void
init_thread(struct thread *t, const char *name, int priority)
{
  // ...
  t->blocked_ticks = 0;
  //...
}
```
3. `timer.c` 中
```c
void timer_sleep(int64_t ticks)
{  
  // 测试点 alarm-zero, alarm-negative
  // 注意对数据进行边界检查
  if (ticks <= 0) return;

  int64_t start = timer_ticks();

  ASSERT(intr_get_level() == INTR_ON);

  // 保证操作的原子性
  enum intr_level old_level = intr_disable();

  struct thread* t = thread_current();
  t->blocked_ticks = ticks;
  thread_block();

  intr_set_level(old_level);
}
// ...
static void
timer_interrupt(struct intr_frame *args UNUSED)
{
  ticks++;
  thread_tick();
  thread_foreach(thread_check_blocked, NULL);
}
```
4. 在 `thread.h` 中声明,  `thread.c` 中实现 `thread_check_blocked` 
```c
void thread_check_blocked(struct thread* t, void *aux);
```
```c
void thread_check_blocked(struct thread*t, void *aux)
{
  if (t->status == THREAD_BLOCKED && t->blocked_ticks > 0) 
  {
    t->blocked_ticks--;
    if (t->blocked_ticks == 0) 
      thread_unblock(t);
  }
}
```

