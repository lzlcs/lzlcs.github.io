---
title: 6.824 分布式系统 Lab1 简记
date: 2023-10-4 12:15:00
mathjax: true
categories:
- Labs
tags: 
- Golang
- MapReduce
- Lab
- Distributed System
---

关于各种库函数调用的问题, 可以问 `gpt`

# `Master` 

`Master` 进程与 `Worker` 进程的通信
1. 接收 `Worker` 进程传来的任务完成的消息
2. 分配给 `Worker` 新的任务

实现两个队列 `toBeMap`, `toBeReduce` 用来记录两种待分配的任务(使用切片) 

实现两个映射 `beingMapped`, `beingReduced` 用来记录哪些任务正在进行, 并记录各个任务被分配的起始时间

任务的记录: 实现结构体 `Task` 记录任务编号, 任务名(文件名或 `Reduce` 编号) \

1. `CallExample` 函数进行通信
    - 此处因为要修改上述两个队列和两个映射, 所以加锁
    - 遍历两个映射, 检查是否超时, 如果超时就从映射中删除, 加入对应队列(注意不能在遍历的时候删)
    - 接收 `Worker` 传进来的 `ExampleArgs` 分析这个 `Worker` 之前做了什么任务
        - 如果之前做了任务, 就要把这个任务从对应映射中删除, 表示做完了
    - 开始分配任务, 具体分配方法是把任务存到 `ExampleReply` 中
        - 检查 `toBeMap` 队列, 如果不为空就分配, 任务类型为 `1` 
        - 检查 `beingMapped` 映射, 如果不为空就让 `Worker` 等待, 任务类型为 `0`
        - 检查 `toBeReduced` 队列, 如果不为空就分配, 任务类型为 `2`
        - 检查 `beingReduced` 映射, 如果不为空就让 `Worker` 等待, 任务类型为 `0`
        - 最后如果上述两个队列两个映射都空, 那么任务结束, 让 `Worker` 终止, 任务类型为 `-1`
2. `Done` 函数: 检查两个映射两个队列是否为空, 为空返回 `true` \
    此处有读操作, 加锁
3. `MakeCoordinator` 函数, 初始化两个队列两个映射, 把任务塞到  `toBeMap` 和 `toBeReduced` 两个队列中 \
此处有写操作, 加锁

# `RPC` 

`ExampleArgs` 和 `ReplyArgs` 都存一个任务, 再存一个任务类型

# `Worker` 

1. `CallExample` 函数
    - 首先初始化 `args` 上次执行的任务类型为 -1,  `reply` 初始化为空 
    - 写一个死循环, 循环开头调用 `call`, 获得新的 `reply`
    - 调用成功之后, 用 `reply` 更新 `args` 从而表明该 `Worker` 上次完成的任务
    - 接下来根据任务类型执行函数
        - 如果任务类型为 0, 更新 `args` 为初始状态, 任务类型 `-1`, 等待一秒
        - 如果任务类型为 1, 调用 `mapProcess` 函数
        - 如果任务类型为 2, 调用 `reduceProcess` 函数
        - 如果任务类型为 -1, `break` 出死循环
    - 最后记得把 `reply` 更新为初始状态, 准备下一次 `call`
2. `Worker` 函数: 调用 `CallExample` 即可
3. `mapProcess` 函数
    - 读取目标文件, 调用 `mapf` 把文件内容拆成键值对
    - 根据 `map` 的任务编号, 创建 `nReduce` 个临时文件, 名字为 `tmpfile-X-Y`
    - 遍历键值对, 解析为 `json`, 写入对应的 `ihash(key) % nReduce` 临时文件中
    - 最后改名 `mr-tmp-X-Y`, 关闭所有文件
4. `reduceProcess` 函数
    - 寻找 `map-tmp-X-Y` (`Y` 为 `reduce` 任务编号) 的所有文件
    - 打开所有文件, 把所有键值对从 `json` 格式解析出来
    - 抄 `mrsequencial.go` 文件的排序方式
    - 创建文件 `mr-out-Y`
    - 抄 `mrsequencial.go` 中的键值对处理方式
