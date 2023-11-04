---
title: 6.824 分布式系统 FaRM
date: 2023-11-4 12:00:00
mathjax: true
categories:
- Papers
tags: 
- FaRM
- Distributed System
---

# 特性

1. 所有的 `replica` 在一个数据中心, 配置管理器决定每个数据分区中哪个服务器是 `Primary`, 哪个服务器是 `Backup`, 使用 `ZooKeeper` 来实现配置管理器
2. 使用主从复制的方式容错, 读写数据都在 `Primary` 上
3. 事务协调器 `CM`, 执行事务, 充当两阶段提交中事务协调器的角色

# 高性能

1. 数据分片, 并行处理
2. 数据存放 `RAM` 中而不是持久化到磁盘
3. 使用 `Remote Direct Memory Access, RDMA`, 直接对服务器的内存进行读写

**`RAM` 断电的问题**

尽量让其不断电, 所以为每个服务器配备一个备用的电池 <br>
至少可以撑到电池没电, 从而让管理员重新接上电源

服务器会在断电的时候向其他服务器发送信号, 会停止当前正在进行的任务并把 `RAM` 中的数据写入磁盘, 这样可以放置大范围的供电故障引起的数据丢失

这种方案只适用于断电的故障, 对于其他类型的让服务器关机的故障并不起作用

**`RDMA`**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6ax2zcwkdtc0.webp)

传统网络: 应用 -> `Socket`(用于缓存) -> `TCP` -> `NIC Driver` -> `NIC` -> 另一台机器的 `NIC` -> `NIC Driver` -> `buffer` -> 应用

`Kernal Bypass`: 应用 -> `NIC` -> 另一台机器的 `NIC` -> 应用, 但是需要应用来实现 `TCP` 相关的功能

`RDMA`: 用特殊的网卡(支持 `RDMA`), 不再发送 `RPC`, 而是直接对内存进行读写

# 乐观并发控制

乐观锁机制下, 可以在没有锁的情况下读取数据 <br>
之后将对数据的修改缓存在本地, 事务结束的时候尝试提交 <br>
由事务管理系统来验证你的提交是否合法

如果你读写操作的顺序与执行顺序一致, 那么成功提交, 否则终止并重新执行

如何验证是否合法?

# `API`

```go
txCreate()
o = txRead(OID)
o.f += 1
txWrite(OID, o)
ok = txCommit()
```

`OID` 包括 `Region` 和 `Access` 定位访问的位置

# 服务器内存布局

每个服务器分为多个 `Region`, 每个 `Region` 存储数据对象 <br>
还存储着 n 个日志队列和消息队列, n 是服务器数量, 其他服务器通过 `RDMA` 修改队列

数据对象的头部数据, 高位是锁标志位, 低位是对象的版本号 

# 

![image](https://github.com/lzlcs/image-hosting/raw/master/image.57dak5yskgw0.webp)

第一个阶段是 `Execute phase`, `C` 通过 `RDMA` 向 $P_{1-3}$ 读取数据

**`LOCK`**

客户端(扮演协调器的角色)把数据写入 `Primary`, `Primary` 尝试使用 `CAS` 来锁定版本, 如果失败就返回 NO, 成功就加锁返回 YES

**`VALIDATE`**

`RDMA` 读版本, 检查是否改变

**`COMMIT BACKUP`**

发送给 `backup`, 等待 `NIC` 确认

**`COMMIT PRIMARY`**

发送数据给 `primary` 更新数据并解锁, 等待 `NIC` 确认

**`TRUNCATE`**

通知各服务器删除对应日志



