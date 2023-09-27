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

在 `thread.c` 中搜索 `&ready_list`, 发现 `thread_unlock` 和 `thread_yield` 函数中有 `push_back`, 
使用 `list_insert_ordered` 函数替换即可

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
2. 在 `thread.c` 中 两个函数 函数中作如下修改

```c
// thread_unblock 中 list_push_back 替换为
list_insert_ordered(&ready_list, &t->elem, thread_greater_priority, NULL);

// thread_yield 中 list_push_back 替换为
list_insert_ordered(&ready_list, &cur->elem, thread_greater_priority, NULL);
```

> 此方法 插入 $O(n)$, 查询 $O(1)$ \
> 方法2: 查询的时候遍历 `ready_list`, 其他地方不做修改 插入 $O(1)$, 查询 $O(n)$ \
> 方法3: 每次查询的时候排序 `ready_list`, 其他地方不做修改, 插入 $O(1)$, 查询 $O(n^2)$ \
> 此处为了和之后的 信号量部分保持风格一致, 选择第一种方法

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

1. 在 `sema_down` 中更改插入链表的方式
```c
// sema_down 中 list_push_back 替换为
list_insert_ordered(&sema->waiters, &thread_current ()->elem, 
                    thread_greater_priority, NULL);
```
2. `sema_up` 调度的时候涉及优先级的问题, `thread_yield` 重新调度一下进程
```c
void
sema_up (struct semaphore *sema) 
{
  //...
  sema->value++;
  thread_yield();  // new
  intr_set_level (old_level);
}
```

**测试点 `priority_condvar`**

注意这个测试点不能使用上述 `list_insert_ordered` 的方式插入 \
因为 `semaphore` 的链表中一开始没有进程, 优先级永远是 0 \
所以要在 `cond_signal` 的时候排序再获取最大值

1. 实现 `sema_less_priority` 函数, 方便取最大值 \
   注意这里可以直接取 `semaphore` 队列中的第一项作为优先级是因为之前 `list_insert_ordered` 保证了信号量等待链表的顺序
```c
bool sema_less_priority(const struct list_elem* a, 
                        const struct list_elem *b, void *aux UNUSED);
```
```c
bool sema_less_priority(const struct list_elem* a, 
                        const struct list_elem *b, void *aux UNUSED)
{
  struct semaphore_elem* sx = list_entry(a, struct semaphore_elem, elem);
  struct semaphore_elem* sy = list_entry(b, struct semaphore_elem, elem);

  int px = list_entry(list_begin(&sx->semaphore.waiters), 
                      struct thread, elem)->priority;
  int py = list_entry(list_begin(&sy->semaphore.waiters), 
                      struct thread, elem)->priority;

  return px < py;
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
    struct list_elem* t = list_max(&cond->waiters, sema_less_priority, NULL);
    list_remove(t);
    sema_up (&list_entry (t, struct semaphore_elem, elem)->semaphore);
  }
}
```

**优先级捐赠**

分析测试用例
1. `priority_donate_one` 表明: 线程在获取锁的时候, 发现比自己优先更低的进程持有锁
   那么就把优先级捐赠给持有锁的线程, 释放锁的时候把持有锁进程改回原来的优先级
2. `priority_donate_multiple*` 表明: 在有多个进程向持有锁的进程捐赠优先级时
   持有锁进程的优先级应该设置为这些进程优先级的最大值
