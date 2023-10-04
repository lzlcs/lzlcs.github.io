---
title: Pintos Project 2
date: 2023-09-29 15:40:00
mathjax: true
tags:
- C
- OS
- Lab
categories: 
- Labs
---

Project 2 请在 `Linux` 环境下测试, 因为 `Windows` 下换行是 `\r\n`, `Linux` 环境下是 `\n`

# 预先准备

先定义好一些变量并做好初始化, 之后会讲这些变量的意思
```c
// thread.h
struct lock filesys_lock;
struct thread_shadow
{
    tid_t tid;
    struct thread* from;              // 记录自己本体
    int exit_code;                    // 本体结束的时候给出的退出码
    bool is_alive, is_being_waited;   // 本体是否活着, 本体是否正在被等待

    struct list_elem child_elem;      // 配合 child_list
};

struct file_shadow
{
   int fd;
   struct file* f;
   struct list_elem elem;
};

struct thread
  {
    // ...
    /* Owned by userprog/process.c. */
    uint32_t *pagedir;                  /**< Page directory. */
    struct thread* parent;   // 记录父进程
    struct list child_list;  // 记录子进程的影子

    struct thread_shadow *to;       // 记录自己的影子
    struct semaphore sema_create; // 用于创建的信号量
    struct semaphore sema_wait;   // 用于等待的信号量
    bool create_success;
    int exit_code;
    // ...
  };
```
```c
// thread.c
void
thread_init (void) 
{
  // ...
  lock_init(&filesys_lock);
  // ...
}

tid_t
thread_create (const char *name, int priority,
               thread_func *function, void *aux) 
{
  // ...
  /* Initialize thread. */
  init_thread (t, name, priority);
  tid = t->tid = allocate_tid ();
  t->parent = thread_current();
  
  t->to = malloc(sizeof(struct thread_shadow));
  t->to->from = t;
  t->to->exit_code = 0;
  t->to->tid = tid;
  t->to->is_alive = true;
  t->to->is_being_waited = false;
  list_push_back(&t->parent->child_list, &t->to->child_elem);

  // ...
}

static void
init_thread (struct thread *t, const char *name, int priority)
{

  // ... 
  t->exit_code = 0;
  list_init(&t->child_list);  
  sema_init(&t->sema_create, 0);
  sema_init(&t->sema_wait, 0);
  t->create_success = true;

  list_init(&t->file_list);
  t->next_fd = 2;

  // ...
}
```
# Part I: 参数分离

1. 阅读文档中 `Program Startup Details` 了解如何把命令行参数压栈
1. 阅读 `string.c`, `process.h` 了解函数功能
    - `strtok_r`: 第一次调用时, `s` 是要分离参数的字符串, 之后调用时 `s` 位置必须为空
      每次返回值是字符串的下一个参数指针, 如果没有下一个参数, 返回 `NULL`
2. 阅读 `process.c`, `process.h` 了解函数功能
    - `process_executeute` 创建进程, 创建进程中的唯一线程, 线程执行 `start_process` 函数
    - `start_process` 设置中断, 调用 `load` 函数创建用户地址空间
    - `load` 函数执行完成之后, 调用 `setup_stack` 创建用户栈

`start_process` 函数中 `load` 函数返回后, 说明地址空间和栈都已经创建好, 此时要把各种参数压栈

1. 修改 `process_executeute` 函数, 正确获取变量名

```c
tid_t
process_execute (const char *file_name) 
{
  char *fn_copy, *get_name;
  tid_t tid;

  // 把文件拷贝两次, 这样就不会访问到 原来 file_name 的内容从而越界
  fn_copy = palloc_get_page(0);
  get_name = palloc_get_page(0);
  if (fn_copy == NULL || get_name == NULL) return TID_ERROR;

  strlcpy (fn_copy, file_name, PGSIZE);
  strlcpy (get_name, file_name, PGSIZE);

  // 获取进程名字
  char *save_ptr;
  get_name = strtok_r(get_name, " ", &save_ptr);  

  /* Create a new thread to executeute FILE_NAME. */
  tid = thread_create (get_name, PRI_DEFAULT, start_process, fn_copy);
  // 释放没用的 get_name 内存
  palloc_free_page(get_name);


  if (tid == TID_ERROR)
    palloc_free_page (fn_copy); 

  return tid;
}
```

2. 修改 `start_process` 函数, 获取正确文件名, 分离参数并压栈

```c
static void
start_process (void *file_name_)
{
  char *file_name = file_name_;
  struct intr_frame if_;
  bool success;
  /* Initialize interrupt frame and load executeutable. */
  memset (&if_, 0, sizeof if_);
  if_.gs = if_.fs = if_.es = if_.ds = if_.ss = SEL_UDSEG;
  if_.cs = SEL_UCSEG;
  if_.eflags = FLAG_IF | FLAG_MBS;

  // new
  // 创建备份页面
  char *cmd = palloc_get_page(0);
  strlcpy(cmd, file_name_, PGSIZE);
  // 提取文件名
  char *save_ptr, *token;
  file_name = strtok_r(file_name, " ", &save_ptr);

  success = load (file_name, &if_.eip, &if_.esp);

  if (!success)
  {
    palloc_free_page(cmd);
    palloc_free_page(file_name);
    thread_exit ();
  }

  // 把参数信息放到栈中
  push_argument(&if_.esp, cmd);
  // hex_dump((uintptr_t)if_.esp, if_.esp, (PHYS_BASE) - if_.esp, true); 

  // 此时 cmd 已经解析完了, 之后不会再用到, 所以释放
  palloc_free_page(file_name);
  palloc_free_page(cmd);

  // end new

  asm volatile ("movl %0, %%esp; jmp intr_exit" : : "g" (&if_) : "memory");
  NOT_REACHED ();
}
```
3. 实现压栈的函数, 根据官方文档一步一步设置栈

![image](https://github.com/lzlcs/image-hosting/raw/master/image.1hxbzkrp6af4.png)
```c
void push_argument(void **esp, char *cmd)
{
  // (*esp) 等价于 if_.esp
  // (*(int *)(*esp)) 表示栈中存的真实值

  // 参数数量和参数列表地址
  int argc = 0, argv[64];
  char *token, *save_ptr; 

  (*esp) = PHYS_BASE;

  for (token = strtok_r(cmd, " ", &save_ptr); token != NULL;
       token = strtok_r(NULL, " ", &save_ptr))
  {
    size_t len = strlen(token);
    (*esp) -= (len + 1);
    memcpy((*esp), token, len + 1);
    argv[argc++] = (*esp);
  }

  (*esp) = (int)(*esp) & 0xfffffffc;  // word_align
  (*esp) -= 4, (*(int *)(*esp)) = 0;  // argv[argc]

  for (int i = argc - 1; i >= 0; i--) // argv[i];
    (*esp) -= 4, (*(int *)(*esp)) = argv[i];

  (*esp) -= 4, (*(int*)(*esp)) = (*esp) + 4; // argv
  (*esp) -= 4, (*(int*)(*esp)) = argc;       // argc
  (*esp) -= 4, (*(int*)(*esp)) = 0;          // return address
}
```

此时参数传递的部分完成, 但是因为还没有实现进程的父子结构以及写的系统调用 \
虽然此时 `make check` 一定会失败, 但是可以通过 `start_process` 设置栈之后调用 `hex_dump` 
```
cd ~/pintos/src/userprog/build
make && pintos --filesys-size=2 -p tests/userprog/args-multiple -a args-multiple -- -f extract run 'args-multiple some arguments for you!'
```
结果如下即为正确
```
bfffffb0              00 00 00 00-05 00 00 00 c0 ff ff bf |    ............|
bfffffc0  f2 ff ff bf ed ff ff bf-e3 ff ff bf df ff ff bf |................|
bfffffd0  da ff ff bf 00 00 00 00-00 00 79 6f 75 21 00 66 |..........you!.f|
bfffffe0  6f 72 00 61 72 67 75 6d-65 6e 74 73 00 73 6f 6d |or.arguments.som|
bffffff0  65 00 61 72 67 73 2d 6d-75 6c 74 69 70 6c 65 00 |e.args-multiple.|
```

# Part II: 系统调用

**准备工作**

首先通过阅读 `syscall.c` 上方的 `<syscall-nr.h>` 中得知 \
每个系统调用有自己的编号, 在 `Project2` 中只需要实现 `0 ~ 12` 这 13 个系统调用

通过阅读文档可知, 系统调用的编号在栈顶 `esp` 位置 \
通过 `pintos/src/lib/user/syscall.c` 的内容可知, 使用系统调用的方式是通过 `syscall` 的宏, 
四个宏实现不同参数数量的系统调用, 参数位置分别是 `esp + 4`, `esp + 8`, `esp + 12`

阅读 `syscall.c` 中定义的函数
1. `syscall_init` 把系统调用注册为中断
2. `syscall_handler` 进行系统调用的中断处理, 也就是选择对应的系统调用

**具体实现**

0. 在 `thread` 中定义错误码 `exit_code`
```c
struct thread
{
    int exit_code;
};
```
1. 完善 `syscall_handler`
```c
static void
syscall_handler (struct intr_frame *f UNUSED) 
{
  int number = *(int *)(f->esp);
  (func[number])(f);
}
```
2. 在 `syscall_init` 中进行系统调用的注册
```c
void syscall_halt(struct intr_frame *f)
{
}
void syscall_exit(struct intr_frame *f)
{
}
void syscall_exec(struct intr_frame *f)
{
}
void syscall_wait(struct intr_frame *f)
{
}
void syscall_create(struct intr_frame *f)
{
}
void syscall_remove(struct intr_frame *f)
{  
}
void syscall_open(struct intr_frame *f)
{
}
void syscall_filesize(struct intr_frame *f)
{
}
void syscall_read(struct intr_frame *f)
{
}
void syscall_write(struct intr_frame *f)
{
}
void syscall_seek(struct intr_frame * f)
{
}
void syscall_close(struct intr_frame *f)
{
}

int (*func[20])(struct intr_frame *);

void
syscall_init (void) 
{
  intr_register_int (0x30, 3, INTR_ON, syscall_handler, "syscall");

  func[SYS_HALT] = syscall_halt;
  func[SYS_EXIT] = syscall_exit;
  func[SYS_EXEC] = syscall_exec;
  func[SYS_WAIT] = syscall_wait;
  func[SYS_CREATE] = syscall_create;
  func[SYS_REMOVE] = syscall_remove;
  func[SYS_OPEN] = syscall_open;
  func[SYS_FILESIZE] = syscall_filesize;
  func[SYS_READ] = syscall_read;
  func[SYS_WRITE] = syscall_write;
  func[SYS_SEEK] = syscall_seek;
  func[SYS_TELL] = syscall_tell;
  func[SYS_CLOSE] = syscall_close;
}
```

接下来根据文档一个个实现函数
1. `syscall_halt`
```c
void syscall_halt(struct intr_frame *f)
{
  shutdown_power_off();
}
```
2. `syscall_exit` 错误码为第一个参数

```c
void syscall_exit(struct intr_frame *f)
{
  int exit_code = *(int *)(f->esp + 4);
  thread_current()->exit_code = exit_code;
  thread_exit();
}
```
3. `syscall_wait` 和 `syscall_execute`: 实现父子进程
   - 可以通过信号量实现等待机制
   - 考虑进程结束之后, 还需要留下信息给父进程访问, 新定义一个 `thread_shadow` \
     影子需要记录本体的 `tid`, 以及是否在运行, 是否已经被等待过一次了

```c
// thread.h
#include "threads/synch.h"
struct thread_shadow
{
    tid_t tid;
    struct thread* from;              // 记录自己本体
    int exit_code;                    // 本体结束的时候给出的退出码
    bool is_alive, is_being_waited;   // 本体是否活着, 本体是否正在被等待

    struct list_elem child_elem;      // 配合 child_list
};
struct thread
{
    struct thread* parent;   // 记录父进程
    struct list child_list;  // 记录子进程的影子

    struct thread_shadow *to;       // 记录自己的影子
    struct semaphore sema_wait;   // 用于等待的信号量
};

```
在操作系统其他代码中, 直接调用的都是 `process_wait`, `process_exit`, `process_execute` 函数
- `process_wait` 按照文档实现
```c
// process.c
void
process_exit (void)
{
  struct thread *cur = thread_current ();
  uint32_t *pd;

  // 更新子节点信息
  struct list* l = &cur->child_list;
  struct list_elem* e;
  for (e = list_begin(l); e != list_end(l); e = list_next(e))
  {
    struct thread_shadow *tmp = list_entry(e, struct thread_shadow, child_elem);
    // 加此判断, tmp->from 不为空
    if (tmp->is_alive)
      tmp->from->parent = NULL;
  }

  if (cur->parent == NULL) free(cur->to);
  else 
  {
    // 把信息下放到影子
    cur->to->exit_code = cur->exit_code;
    if (cur->to->is_being_waited)
      sema_up(&cur->sema_wait);

    cur->to->is_alive = false;
    cur->to->from = NULL;
  }

  pd = cur->pagedir;
  if (pd != NULL) 
    {
      cur->pagedir = NULL;
      pagedir_activate (NULL);
      pagedir_destroy (pd);
    }


  // 记得在尾部加上 退出信息
  printf("%s: exit(%d)\n", cur->name, cur->to->exit_code);
}
```
- `process_execute` 需要保证子进程创建完成之后才返回, 从而获得子进程是否创建成功的状态 \
    在 `thread` 中加入一个信号量, 在 `process_execute` 中 `P` 操作, 在 `start_process` 创建完成之后 `V` 操作, 
    还需要保存一个子进程是否创建成功的布尔变量 \
    注意 `sema_wait` 是等待进程结束, `sema_create` 是等待进程创建结束
```c
// thread.h
struct thread
{
    struct semaphore sema_create;
    bool create_success;
}
```
```c
// process.c
tid_t
process_execute (const char *file_name) 
{
  // ...
  struct thread* cur = thread_current();
  sema_down(&cur->sema_create);
  if (!cur->create_success) return -1;

  return tid;
}
static void
start_process (void *file_name_)
{
  // ...

  struct thread* cur = thread_current();
  if (!success)
  {
    palloc_free_page(cmd);

    cur->exit_code = -1;
    cur->parent->create_success = false;   
    sema_up(&cur->parent->sema_create);
    thread_exit ();
  }

  // 把参数信息放到栈中
  push_argument(&if_.esp, cmd);
  // hex_dump((uintptr_t)if_.esp, if_.esp, (PHYS_BASE) - if_.esp, true); 


  palloc_free_page(cmd);
  
  cur->parent->create_success = true;
  sema_up(&cur->parent->sema_create);

  asm volatile ("movl %0, %%esp; jmp intr_exit" : : "g" (&if_) : "memory");
  NOT_REACHED ();
}
```

```c
void syscall_exec(struct intr_frame *f)
{
  char *cmd = *(char **)(f->esp + 4);
  f->eax = process_execute(cmd);
}
void syscall_wait(struct intr_frame *f)
{
  int pid = *(int *)(f->esp + 4);
  f->eax = process_wait(pid);
}
```

没有实现 `syscall_write` 向 `stdout` 输出的功能, 此时 `make test` 依然会全部失败

# Part III: 用户访存检查

因为按理来说 `kernel` 不可能出现访存越界的情况\
所以关键点在于处理用户调用 `syscall` 时访问内存的错误

此处使用文档中的第一种做法

1. 为用户态内存出错设置错误码
```c
// exception.c
static void
kill (struct intr_frame *f) 
{
   thread_current()->exit_code = -1;
   // ...
}
```
2. 实现检查指针的函数 \
   注意字符串, 要先检查字符串指针的有效性, 再逐步检查每个字符的有效性
```c
// syscall.c
void error_exit()
{
  thread_current()->exit_code = -1;
  thread_exit();
}

void *check_ptr(void *ptr, int byte)
{
  // 如果不在用户内存中就报错
  if (!is_user_vaddr(ptr)) error_exit();

  // 依次检查每个字节是否在页表中
  int pd = thread_current()->pagedir;
  for (int i = 0; i < byte; i++)
    if (pagedir_get_page(pd, ptr + i) == NULL) error_exit();
  return ptr;
}
void check_str(char *ptr)
{
  // 不能直接使用 strlen
  char *tmp = ptr;
  while (true)
  {
    tmp = check_ptr(tmp, 1);
    if ((*tmp) == '\0') break;
    tmp++;
  }
}
```
3. 重写之前的系统调用
```c
void syscall_exit(struct intr_frame *f)
{
  int exit_code = *(int *)check_ptr(f->esp + 4, 4);
  thread_current()->exit_code = exit_code;
  thread_exit();
}
void syscall_exec(struct intr_frame *f)
{
  char *cmd = *(char **)check_ptr(f->esp + 4, 4);
  check_str(cmd);
  f->eax = process_execute(cmd);
}
void syscall_wait(struct intr_frame *f)
{
  int pid = *(int *)check_ptr(f->esp + 4, 4);
  f->eax = process_wait(pid);
}
```

# Part IV: 文件操作

阅读 `file.c` 和 `filesys.c` 

因为这些文件操作都只能操作进程打开的文件, 所以每个进程需要记录它打开的文件, 并给这些文件一个唯一的编号从而方便后续访问
编号从 2 开始, 因为 `stdin` 占有 0, `stdout` 占有 1

类似文件描述符, 再定义一个影子
```c
struct file_shadow
{
   int fd;
   struct file* f;
   struct list_elem elem;
};
```
```c
struct thread
{
    // ...
    struct file *exec_file;
    struct list file_list;
    int next_fd;      // 进程的文件编号分配, 注意初始化为 2
    // ...
}
```
注意文件锁的问题, 文件需要互斥访问
```c
// thread.h
struct lock filesys_lock
```
```c
// process.c
static void
start_process (void *file_name_)
{
  // ...
  lock_acquire(&filesys_lock);
  success = load (file_name, &if_.eip, &if_.esp);
  lock_release(&filesys_lock);
  // ...
}
```

实现辅助函数 `foreach_file` 用来找到目标文件
```c
// syscall.c
struct file_shadow *foreach_file(int fd)
{
  struct thread* cur = thread_current();
  struct list * l = &cur->file_list;
  struct list_elem *e;
  for (e = list_begin(l); e != list_end(l); e = list_next(e))
  {
    struct file_shadow* tmp = list_entry(e, struct file_shadow, elem);
    if (tmp->fd == fd) return tmp;
  }
  return NULL;
}
```
然后实现这么多的系统调用, `f->eax` 是返回值
```c
// syscall.c
void syscall_create(struct intr_frame *f)
{
  char *file_name = *(char **)check_ptr(f->esp + 4, 4);
  check_str(file_name);
  int file_size = *(int *)check_ptr(f->esp + 8, 4);

  lock_acquire(&filesys_lock);
  f->eax = filesys_create(file_name, file_size);
  lock_release(&filesys_lock);
}

void syscall_remove(struct intr_frame *f)
{  
  char *file_name = *(char **)check_ptr(f->esp + 4, 4);
  check_str(file_name);

  lock_acquire(&filesys_lock);
  f->eax = filesys_remove(file_name);
  lock_release(&filesys_lock);
}

void syscall_open(struct intr_frame *f)
{
  char *file_name = *(char **)check_ptr(f->esp + 4, 4);
  check_str(file_name);

  lock_acquire(&filesys_lock);
  struct file* open_file = filesys_open(file_name);
  lock_release(&filesys_lock);

  if (open_file == NULL) {
    f->eax = -1;
    return;
  }

  struct thread *cur = thread_current();
  struct file_shadow *tmp = malloc(sizeof(struct file_shadow)); 
  // 打开文件, 记录文件编号, 插入打开文件的队列
  tmp->f = open_file, tmp->fd = cur->next_fd++;
  list_push_back(&cur->file_list, &tmp->elem);
  f->eax = tmp->fd;
}

void syscall_filesize(struct intr_frame *f)
{
  int fd = *(int *)check_ptr(f->esp + 4, 4);

  struct file_shadow* tmp = foreach_file(fd);
  if (tmp->f == NULL) f->eax = -1;
  else 
  {
    lock_acquire(&filesys_lock);
    f->eax = file_length(tmp->f);
    lock_release(&filesys_lock);
  }
}

void syscall_read(struct intr_frame *f)
{
  int fd = *(int *)check_ptr(f->esp + 4, 4);
  char *buf = *(char **)check_ptr(f->esp + 8, 4);
  check_str(buf);

  int size = *(int *)check_ptr(f->esp + 12, 4);

  if (fd == 1) error_exit();

  // 文档中有写 input_getc 处理标准输入
  if (fd == 0) 
  {
    for (int i = 0; i < size; i++)
      *buf = input_getc(), buf++;
    f->eax = size;
  } 
  else 
  {
    struct file_shadow* tmp = foreach_file(fd);
    if (tmp == NULL) f->eax = -1;
    else
    {
      lock_acquire(&filesys_lock);
      f->eax = file_read(tmp->f, buf, size);
      lock_release(&filesys_lock);
    }
  }
}
void syscall_write(struct intr_frame *f)
{
  int fd = *(int *)check_ptr(f->esp + 4, 4);
  char *buf = *(char **)check_ptr(f->esp + 8, 4);
  check_str(buf);
  int size = *(int *)check_ptr(f->esp + 12, 4);

  if (fd == 0) error_exit();
  // 文档中写 putbuf 处理输出
  if (fd == 1)
  {
    putbuf(buf, size);
    f->eax = size;
  }
  else
  {
    struct file_shadow* tmp = foreach_file(fd);
    if (tmp == NULL) f->eax = -1;
    else 
    {
      lock_acquire(&filesys_lock);
      f->eax = file_write(tmp->f, buf, size);
      lock_release(&filesys_lock);
    }
  }
}

void syscall_seek(struct intr_frame * f)
{
  int fd = *(int *)check_ptr(f->esp + 4, 4);
  int pos = *(int *)check_ptr(f->esp + 8, 4);

  struct file_shadow* tmp = foreach_file(fd);
  if (tmp != NULL) 
  {
    lock_acquire(&filesys_lock);
    file_seek(tmp->f, pos);
    lock_release(&filesys_lock);
  }
}

void syscall_tell(struct intr_frame *f)
{
  int fd = *(int *)check_ptr(f->esp + 4, 4);

  struct file_shadow* tmp = foreach_file(fd);
  if (tmp != NULL) 
  {
    lock_acquire(&filesys_lock);
    f->eax = file_tell(tmp->f);
    lock_release(&filesys_lock);
  }
  else f->eax = -1;
}

void syscall_close(struct intr_frame *f)
{
  int fd = *(int *)check_ptr(f->esp + 4, 4);

  struct file_shadow* tmp = foreach_file(fd);
  if (tmp != NULL)
  {
    lock_acquire(&filesys_lock);
    file_close(tmp->f);
    // 从队列中移除, 释放 file_shadow
    list_remove(&tmp->elem);
    free(tmp);
    lock_release(&filesys_lock);
  }
}
```
最后当进程结束的时候, 要释放所有文件
```c
void
process_exit (void)
{
  // ...
  while (!list_empty(&cur->file_list))
  {
    e = list_pop_front(&cur->file_list);
    struct file_shadow* tmp = list_entry(e, struct file_shadow, elem);
    lock_acquire(&filesys_lock);
    file_close(tmp->f);
    lock_release(&filesys_lock);
    free(tmp);
  }

  printf("%s: exit(%d)\n", cur->name, cur->to->exit_code);
}
```

# Part V: 拒绝写入可执行文件

在程序运行的时候, 其他进程不能修改程序, 很合理
```c
struct thread
{
    struct file* exec_file;
}
```

```c
static void
start_process (void *file_name_)
{
  // ...
  // 把参数信息放到栈中
  push_argument(&if_.esp, cmd);
  // hex_dump((uintptr_t)if_.esp, if_.esp, (PHYS_BASE) - if_.esp, true); 

  lock_acquire(&filesys_lock);
  struct file *f = filesys_open(file_name);
  file_deny_write(f);
  lock_release(&filesys_lock);
  thread_current()->exec_file = f;
  // ...
}
void
process_exit (void)
{
  // ...
  lock_acquire(&filesys_lock);
  file_close(cur->exec_file);
  lock_release(&filesys_lock);

  printf("%s: exit(%d)\n", cur->name, cur->to->exit_code);
}
```

至此, 80 个测试点 `All Pass`


