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

## 源码分析

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
`wait_list` 链表记录所有等待中的进程 \
在进程结构体中使用 `blocked_ticks` 变量记录剩余阻塞的 `ticks`

每次处理时钟中断的时候, 使用 `thread_foreach` 函数遍历 `wait_list` 并检测 `blocked_ticks` 值

1. `thread.h` 中
```c

 struct thread
  {
    // ...
    struct list_elem waitelem;
    uint32_t blocked_ticks;
    // ...
  };
```
2. `thread.c` 中, `blocked_ticks` 值初始化为 0, 定义 `wait_list` 并初始化
```c
struct list wait_list;
void thread_init(void)
{
  // ...
  list_init(&wait_list);
  // ...
}

static void init_thread(struct thread *t, const char *name, int priority)
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
  thread_enable_wait(t, ticks);

  intr_set_level(old_level);
}
// ...
static void
timer_interrupt(struct intr_frame *args UNUSED)
{
  // ...
  thread_disable_wait(); 
}
```
4. 在 `thread.h` 中声明,  `thread.c` 中实现 `thread_check_blocked` 
```c
void thread_enable_wait(struct thread* t, int ticks);
void thread_disable_wait();
```
```c
void thread_enable_wait(struct thread* t, int ticks) 
{
  list_push_back(&wait_list, &t->waitelem);
  t->blocked_ticks = ticks;
  thread_block();
}

void thread_disable_wait() 
{
  struct list_elem *e;

  for (e = list_begin(&wait_list); e != list_end(&wait_list); e = list_next(e))
  {
    struct thread *t = list_entry(e, struct thread, waitelem);

    t->blocked_ticks--;
    if (t->blocked_ticks == 0)
    {
      list_remove(e);        
      thread_unblock(t);
    }
  }
}
```

# Part II: Priority Scheduling

## 实现优先级队列

### 源码分析

**`Pintos` 中链表的实现**

```c
/* List element. */
struct list_elem 
{
    struct list_elem *prev;     /* Previous list element. */
    struct list_elem *next;     /* Next list element. */
};

/* List. */
struct list 
{
    struct list_elem head;      /* List head. */
    struct list_elem tail;      /* List tail. */
};

#define list_entry(LIST_ELEM, STRUCT, MEMBER)           \
        ((STRUCT *) ((uint8_t *) &(LIST_ELEM)->next     \
                     - offsetof (STRUCT, MEMBER.next)))
```

注意到并没有保存具体的数据, 因为 `C` 中没有模板 \
所以 `Pintos` 使用了一种很难懂的 `list_entry` 的宏定义

以 `C++` 的模板类重写:

```cpp
template <class T> 
T* list_entry(struct list_elem*, struct T, list_elem_name);
```

由于 `C` 中没有面向对象 \
所以对于每一个链表都需要结构体中有一个对应的 `list_elem`

在 `struct thread` 中:
1. `allelem` 对应链表 `all_list`
2. `elem` 对应链表 `ready_list`, 和 `sych.c` 中共用 `elem`

使用 `list_insert_ordered` 函数实现 新进程 根据优先级插入 `ready_list`

**`Pintos` 朴素的调度算法**

通过 `schedule` 函数可以看到, `Pintos` 把 CPU 交给 `ready_list` 中第一个进程


### 实现思路


**测试点 `alarm-priority`**

1. 在 `thread.h` 和 `thread.c` 中实现 `thread_cmp_priority` 函数从而实现比较
```c
bool thread_greater_priority (const struct list_elem *a,
                              const struct list_elem *b,
                              void *aux);
```
```c
bool thread_greater_priority (const struct list_elem *a,
                              const struct list_elem *b,
                              void *aux) 
{
  int x = list_entry(a, struct thread, elem)->priority;
  int y = list_entry(b, struct thread, elem)->priority;
  return x > y;
}
```
2. 在 `thread.c` 中 `next_thread_to_run` 函数中先排序再选择

```c
static struct thread *
next_thread_to_run (void) 
{
  if (list_empty (&ready_list))
    return idle_thread;
  else
  {
    struct list_elem *t = list_min(&ready_list, thread_greater_priority, NULL);
    list_remove(t);
    return list_entry (t, struct thread, elem);
    // list_sort(&ready_list, thread_greater_priority, NULL);
    // return list_entry (list_pop_front(&ready_list), struct thread, elem);
  }
}
```

> 方法1: 插入的时候选择合适位置插入 $O(n)$, 查询 $O(1)$ \
> 方法2: 查询的时候遍历 `ready_list`, 其他地方不做修改 插入 $O(1)$, 查询 $O(n)$ \
> 方法3: 每次查询的时候排序 `ready_list`, 其他地方不做修改, 插入 $O(1)$, 查询 $O(n^2)$ \
> 方法3 简便, 但是在之后的大数据测试点中会性能差一点点从而无法通过, 此处使用方法 2
> 在之后的 `sema`, `cond` 和 `lock` 都使用方法 3

## 优先级改变及抢占式调度


**测试点 `priority_change`, `priority_preempt`, `priority_fifo`**

在更改优先级的时候, 要重新选择当前运行的进程 \
有两个地方可以设置优先级
1. `thread_set_priority` 
2. `thread_create`

注意到 `thread_yield` 函数执行的就是重新选择应该运行的进程功能
```c
tid_t
thread_create (const char *name, int priority,
               thread_func *function, void *aux) 
{
  //...
  /* Add to run queue. */
  thread_unblock (t);
  thread_yield();

  return tid;
}
```
```c
void
thread_set_priority (int new_priority) 
{
  thread_current ()->priority = new_priority;
  thread_yield();
}
```

## 进程同步

### 源码分析

**信号量的实现**

同步问题的核心是信号量 `semaphore`

```c
struct semaphore 
  {
    unsigned value;             /**< Current value. */
    struct list waiters;        /**< List of waiting threads. */
  };

void sema_init (struct semaphore *, unsigned value);
void sema_down (struct semaphore *);
void sema_up (struct semaphore *);
```
1. `sema_init` 函数设置信号量值, 初始化等待进程的链表
2. `sema_down` 如果当前的信号量为0, 则原子性地把当前进程放到等待链表里
3. `sema_up` 使当前等待链表中的第一个进程变成就绪态, 由于之前的 `sema_down`函数中
   使用 `while` 来判断信号量是否为 0, 所以如果条件不满足, 进程仍然会被重新放到等待链表中

**监视器的实现**

```c
struct condition 
  {
    struct list waiters;        /**< List of waiting threads. */
  };

void cond_init (struct condition *);
void cond_wait (struct condition *, struct lock *);
void cond_signal (struct condition *, struct lock *);
void cond_broadcast (struct condition *, struct lock *);
```
1. `cond_init` 初始化等待链表(存放信号量结构体)
2. `cond_wait` 先初始化信号量, 放入等待链表, 释放锁, 等待 `cond_signal`
3. `cond_signal` 增加信号量, 唤醒等待中的 `cond_wait` 
4. `cond_broadcast` 增加等待队列中的全部信号量

### 实现思路

**测试点 `priority_sema`**

1. `sema_up` 调度的时候涉及优先级的问题, `thread_yield` 重新调度一下进程
```c
void
sema_up (struct semaphore *sema) 
{
  //...
  if (!list_empty (&sema->waiters)) 
  {
    list_sort (&sema->waiters, thread_greater_priority, NULL);
    thread_unblock (list_entry (list_pop_front(&sema->waiters),
                                struct thread, elem));
  }
  sema->value++;
  thread_yield();
  intr_set_level (old_level);
}
```

**测试点 `priority_condvar`**

注意这个测试点不能使用 `list_insert_ordered` 的方式插入 \
因为 `semaphore` 的链表中一开始没有进程, 优先级永远是 0 \
所以要在 `cond_signal` 的时候排序再获取最大值

1. 实现 `sema_less_priority` 函数, 方便取最大值 \
   注意这里可以直接取 `semaphore` 队列中的第一项作为优先级是因为之前 `list_insert_ordered` 保证了信号量等待链表的顺序
```c
bool sema_greater_priority(const struct list_elem* a, 
                           const struct list_elem *b, void *aux);
```
```c
bool sema_greater_priority(const struct list_elem* a, 
                           const struct list_elem *b, void *aux UNUSED)
{
  struct semaphore_elem* sx = list_entry(a, struct semaphore_elem, elem);
  struct semaphore_elem* sy = list_entry(b, struct semaphore_elem, elem);

  int px = list_entry(list_begin(&sx->semaphore.waiters), 
                      struct thread, elem)->priority;
  int py = list_entry(list_begin(&sy->semaphore.waiters), 
                      struct thread, elem)->priority;

  return px > py;
}
```
2. 修改 `cond_signal` 函数
```c
void
cond_signal (struct condition *cond, struct lock *lock UNUSED) 
{
  // ...

  if (!list_empty (&cond->waiters)) 
  {
    list_sort(&cond->waiters, sema_greater_priority, NULL);
    sema_up (&list_entry (list_pop_front(&cond->waiters),
                          struct semaphore_elem, elem)->semaphore);
  }
}
```

## 优先级捐赠

**分析测试用例**

注意在 任务三 中不会使用优先级捐赠, 所以要判断 `thread_mlfqs`

1. `priority_donate_one` 表明: 线程在获取锁的时候, 发现比自己优先更低的进程持有锁
   那么就把优先级捐赠给持有锁的线程, 释放锁的时候把持有锁进程改回原来的优先级
2. `priority_donate_multiple*` 表明: 在有多个进程向持有锁的进程捐赠优先级时
   持有锁进程的优先级应该设置为这些进程优先级的最大值
3. `priority_donate_nest` 表明: 在提升持有锁的线程优先级之后, 如果这个线程在等待其他锁, 
   那么就递归地把优先级捐赠给目标进程
4. `priority_donate_sema` 是信号量和锁的混合使用
5. `priority_donate_lower` 在被捐赠之后, 改变自身的优先级, 如果优先级变低, 不能影响捐赠
   但是在恢复优先级的时候应该恢复到新的优先级
6. `priority_donate_chain` 依旧是嵌套捐赠问题

**模型分析**

总体上是一个锁和进程的分层树形结构, 一层点表示进程, 一层点表示锁

1. `lock_acquire` 
    - 首先连上目标锁的节点, 然后一直向上捐赠优先级
    - 锁被释放之后, 把锁节点转换为自己的子节点
2. `lock_release`: 断开自己的目标锁的子节点

**更改数据结构**


1. 进程节点中存自己所有的子节点(锁), 存一个父节点(锁), 存一个本身优先级
3. 锁节点中存 自己的父节点(进程), 存一个 所有子节点(进程)中最大的优先级捐赠 \
   最大优先级在子节点(进程) 向自己连边的时候会更新, 所以不用存子节点(进程)

```c
struct thread
{
    // ...
    int base_priority;                  /**< 进程原本的优先级*/
    struct list locks;                  /**< 所有的 子节点(锁)*/ 
    struct lock *locks_waiting;         /**< 父节点(锁)*/
    // ...
}

static void init_thread (struct thread *t, const char *name, int priority)
{
  // ...
  t->base_priority = priority;
  list_init (&t->locks);
  t->locks_waiting = NULL;
  // ...
}
```
```c
struct lock 
{
    // ...
    struct list_elem elem;      /**< 配合进程中的链表 */
    int max_priority;           /**< 从子节点(进程)捐赠上来的最大优先级 */
};

void lock_init (struct lock *lock)
{
  // ...
  lock->max_priority = 0;
  // ...
}
```

**更改函数**

0. 函数声明
```c
// thread.h
void thread_update_priority (struct thread *t);
```
```c
// synch.h

bool lock_greater_priority (const struct list_elem *a, const struct list_elem *b, void *aux);

void set_lock_point(struct lock *lock);
void change_lock_point(struct lock* lock);
```

1. `thread_update` 函数, 在子节点(锁)中选一个最大的优先级更新
```c
void
thread_update_priority (struct thread *t)
{
  enum intr_level old_level = intr_disable ();
  int max_priority = t->base_priority;
  int lock_priority;

  if (!list_empty (&t->locks))
  {
    list_sort (&t->locks, lock_greater_priority, NULL);
    lock_priority = list_entry (list_front (&t->locks), struct lock, elem)->max_priority;
    if (lock_priority > max_priority)
      max_priority = lock_priority;
  }

  t->priority = max_priority;
  intr_set_level (old_level);
}
```
2. 实现  `lock_greater_priority` 函数实现锁节点之间的比较
```c
bool
lock_greater_priority (const struct list_elem *a, const struct list_elem *b, void *aux UNUSED)
{
  int x = list_entry (a, struct lock, elem)->max_priority;
  int y = list_entry (b, struct lock, elem)->max_priority; 
  return x > y;
}
```
3. `lock_acquire` 连边的过程
```c
void
lock_acquire (struct lock *lock)
{

  ASSERT (lock != NULL);
  ASSERT (!intr_context ());
  ASSERT (!lock_held_by_current_thread (lock));

  set_lock_point(lock);
  sema_down (&lock->semaphore);
  change_lock_point(lock);
}

void set_lock_point(struct lock *lock)
{
  if (lock->holder == NULL || thread_mlfqs) return;

  struct thread* cur = thread_current();
  cur->locks_waiting = lock;

  struct lock* tmp = lock;
  while (tmp && cur->priority > tmp->max_priority)
  {
    tmp->max_priority = cur->priority;
    thread_update_priority(tmp->holder);
    tmp = tmp->holder->locks_waiting;
  }
}

void change_lock_point(struct lock* lock)
{
  lock->holder = thread_current();

  if (thread_mlfqs) return;
  enum intr_level old_level = intr_disable ();

  struct thread* cur = thread_current ();

  cur->locks_waiting = NULL;
  lock->max_priority = 0;

  list_push_back(&cur->locks, &lock->elem);

  intr_set_level (old_level);
}
```
1. `lock_release` 函数, 断开子节点(锁)
```c
void
lock_release (struct lock *lock) 
{
  ASSERT (lock != NULL);
  ASSERT (lock_held_by_current_thread (lock));

  if (!thread_mlfqs)
  {
    list_remove (&lock->elem);
    // 此处 locks 链表可能为空, 所以 thread_update_priority 中要判断一下是否为空
    thread_update_priority (thread_current ());
  }

  lock->holder = NULL;
  sema_up (&lock->semaphore);
}
```
1. 最后的 `thread_set_priority` 函数, 如果子节点(锁)为空或者新优先级更大, 则更改当前优先级
```c
void
thread_set_priority (int new_priority)
{
  if (thread_mlfqs) return;

  enum intr_level old_level = intr_disable ();

  struct thread *cur = thread_current ();

  cur->base_priority = new_priority;

  if (list_empty (&cur->locks) || new_priority > cur->priority)
  {
    cur->priority = new_priority;
  }

  thread_yield ();
  intr_set_level (old_level);
}
```

# Part III: 多级反馈队列

**浮点数精度问题**

根据文档, 具体的实现方式是通过移位运算 \
选择一个目录, 然后实现 `fixed_point.h` (我选择放在 `lib` 目录下)
```c
#ifndef __THREAD_FIXED_POINT_H
#define __THREAD_FIXED_POINT_H

/* Basic definitions of fixed point. */
typedef int fixed_t;
/* 16 LSB used for fractional part. */
#define FP_SHIFT_AMOUNT 16
/* Convert a value to fixed-point value. */
#define FP_CONST(A) ((fixed_t)(A << FP_SHIFT_AMOUNT))
/* Add two fixed-point value. */
#define FP_ADD(A, B) (A + B)
/* Add a fixed-point value A and an int value B. */
#define FP_ADD_MIX(A, B) (A + (B << FP_SHIFT_AMOUNT))
/* Substract two fixed-point value. */
#define FP_SUB(A, B) (A - B)
/* Substract an int value B from a fixed-point value A */
#define FP_SUB_MIX(A, B) (A - (B << FP_SHIFT_AMOUNT))
/* Multiply a fixed-point value A by an int value B. */
#define FP_MULT_MIX(A, B) (A * B)
/* Divide a fixed-point value A by an int value B. */
#define FP_DIV_MIX(A, B) (A / B)
/* Multiply two fixed-point value. */
#define FP_MULT(A, B) ((fixed_t)(((int64_t)A) * B >> FP_SHIFT_AMOUNT))
/* Divide two fixed-point value. */
#define FP_DIV(A, B) ((fixed_t)((((int64_t)A) << FP_SHIFT_AMOUNT) / B))
/* Get integer part of a fixed-point value. */
#define FP_INT_PART(A) (A >> FP_SHIFT_AMOUNT)
/* Get rounded integer of a fixed-point value. */
#define FP_ROUND(A) (A >= 0 ? ((A + (1 << (FP_SHIFT_AMOUNT - 1))) >> FP_SHIFT_AMOUNT) \
                            : ((A - (1 << (FP_SHIFT_AMOUNT - 1))) >> FP_SHIFT_AMOUNT))

#endif /* thread/fixed_point.h */
```

在 `timer.c` 和 `thread.h` 中加入 `#include <fixed_point.h>`

**更改数据结构和设置全局变量**

根据文档, 每个进程需要存一个 `nice` 和 `recent_cpu` 值 \
还要在全局变量中存一个 `load_avg` 值 \
`recent_cpu` 和 `load_avg` 都是 `fixed_point` 类型

```c
// thread.h
struct thread
  {
    // ...
    int nice;
    fixed_t recent_cpu;
    // ...
  };
```
```c
// thread.c
fixed_t load_avg;

void
thread_init (void) 
{
  // ...
  load_avg = FP_CONST(0);
  // ...
}

static void
init_thread (struct thread *t, const char *name, int priority)
{
  // ...
  t->nice = 0;
  t->recent_cpu = FP_CONST(0);
  // ...
}
```

**`Niceness` 部分**

根据文档一步步实现

1. `thread_get_nice` 
2. `thread_set_nice` 设置 `nice` 值, 计算优先级, 再调度一下
```c
/** Sets the current thread's nice value to NICE. */
void
thread_set_nice (int nice UNUSED) 
{
  struct thread* cur = thread_current();
  cur->nice = nice;
  thread_mlfqs_update_priority(cur);
  thread_yield();
}

/** Returns the current thread's nice value. */
int
thread_get_nice (void) 
{
  return thread_current()->nice;
}
```

**`load_avg` `recent_cpu` 部分** 

根据文档给的公式实现
```c
/** Returns 100 times the system load average. */
int
thread_get_load_avg (void) 
{
  return FP_ROUND (FP_MULT_MIX (load_avg, 100));
}

/** Returns 100 times the current thread's recent_cpu value. */
int
thread_get_recent_cpu (void) 
{
  return FP_ROUND (FP_MULT_MIX (thread_current ()->recent_cpu, 100));
}
```

**`timer_interrupt` 部分**

按照文档, 更新 `priority`, `nice`, `load_avg`, `recent_cpu`

```c
static void
timer_interrupt(struct intr_frame *args UNUSED)
{
  ticks++;
  thread_tick();

  if (thread_mlfqs)
  {
    thread_mlfqs_increase_recent_cpu_by_one ();
    if (ticks % TIMER_FREQ == 0)
      thread_mlfqs_update_load_avg_and_recent_cpu ();
    else if (ticks % 4 == 0)
      thread_mlfqs_update_priority (thread_current ());
  }
  thread_disable_wait();
}
```

声明辅助函数
```c
// thread.h
void thread_mlfqs_update_priority(struct thread* t);
void thread_mlfqs_increase_recent_cpu_by_one ();
void thread_mlfqs_update_load_avg();
void thread_mlfqs_update_recent_cpu();
```
实现辅助函数
```c
void thread_mlfqs_update_priority(struct thread *t)
{
  if (t == idle_thread) return;

  t->priority = FP_INT_PART(FP_SUB_MIX(FP_SUB(FP_CONST(PRI_MAX),
                                              FP_DIV_MIX(t->recent_cpu, 4)),
                                       2 * t->nice));
  t->priority = t->priority < PRI_MIN ? PRI_MIN : t->priority;
  t->priority = t->priority > PRI_MAX ? PRI_MAX : t->priority;
}

void thread_mlfqs_increase_recent_cpu_by_one(void)
{
  struct thread *current_thread = thread_current();
  if (current_thread == idle_thread) return;
  current_thread->recent_cpu = FP_ADD_MIX(current_thread->recent_cpu, 1);
}

void thread_mlfqs_update_load_avg(void)
{
  size_t ready_threads = list_size(&ready_list);
  if (thread_current() != idle_thread) ready_threads++;

  load_avg = FP_ADD(FP_DIV_MIX(FP_MULT_MIX(load_avg, 59), 60), FP_DIV_MIX(FP_CONST(ready_threads), 60));
}

void thread_mlfqs_update_recent_cpu(void)
{
  struct list_elem *e;
  for (e = list_begin(&all_list); e != list_end(&all_list); e = list_next(e))
  {
    struct thread *t = list_entry(e, struct thread, allelem);
    if (t == idle_thread) continue;

    t->recent_cpu = FP_ADD_MIX(FP_MULT(FP_DIV(FP_MULT_MIX(load_avg, 2), FP_ADD_MIX(FP_MULT_MIX(load_avg, 2), 1)), t->recent_cpu), t->nice);
    thread_mlfqs_update_priority(t);
  }
}
```

`All 27 tests passed.`

