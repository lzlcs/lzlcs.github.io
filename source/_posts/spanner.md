---
title: 6.824 分布式系统 Spanner
date: 2023-11-2 17:00:00
mathjax: true
categories:
- Papers
tags: 
- Spanner
- Distributed System
---

# 实现

![image](https://github.com/lzlcs/image-hosting/raw/master/image.q77bmz33y4w.png)

`Spanner` 是 `Zone` 的集合, 一个 `Zone` 有
1. 一个 `zoneMaster`: 为 `spanserver` 分配数据
2. 几百到几千个 `spanserver`: 为客户端提供数据服务
3. `location proxy` 给客户端定位分配好的 `spanserver`

`universemaster` 是一个控制台, 显示所有 `zone` 的状态信息 <br>
`placement dirver` 分钟级处理 `zone` 间的自动化迁移, 定期与 `spanserver` 交互从而进行副本约束或负载均衡

![image](https://github.com/lzlcs/image-hosting/raw/master/image.46c39kldbhw0.webp)

每个 `spanserver` 负责 100~1000 个 `tablet` <br>
其状态被保存在一系列类 B 树的文件和一个预写日志中 (`WAH`) <br>
他们都在一个叫做 `Colossus` 的分布式文件系统中(`GFS` 的后继)

每个键值对都会按照 `key` 哈希从而被分区, 所有数据中心的对应分区组成一个 `Paxos`

其中一个 `Paxos` 组的 `Leader` 成为跨组事务的 `Leader` 协调者, 即 `coordinator leader`, 其他的是 `paticipant leader`

# `TrueTime`

为了实现一致性, 事务被分配了全局的唯一 `ID`, 使用全球时钟同步机制 (基于 `GPS` 和 原子钟)

`TrueTime API` 提供如下三个接口
1. `TT.now()`, 返回一个范围区间 `[earliest, latest]`
2. `TT.before(t)`, 如果确保真实事件不到 `t`, 返回 `true`
3. `TT.after(t)`, 如果确保真实事件超过 `t`, 返回 `true`

# 事务

支持如下几种事务
1. 读写事务
2. 只读事务
3. 快照读, 客户端提供时间戳
3. 快照读, 客户端提供时间范围

考虑原子钟数据传输导致的误差, `Spanner` 使用延时提交的方式 <br>
假设原子钟的误差是 `x`, 事务 B 要在 事务 A 提交之后再过 `x` 的时间提交, 所以写事务的时延至少是 `2x`

**单个 `Paxos` 组**

1. 读写: 获取当前时间戳, 执行读写操作并提交时间戳
2. 只读: 获取当前时间戳, 获取当前应该读取的版本
3. 快照读: 自带时间戳, 获取当前应该读取的版本

**多个 `Paxos` 组**

1. 读写需要两阶段提交
    * `Prepare`: 客户端将数据发送给多个 `Paxos` 组的 `Leader`, `Leader` 发起 `Prepare`, 其他 `Follower` 需要锁住对应的资源
    * `Commit`: 协调者发起 `commit`, 使用协调者本地的时间戳
2. 读: 同上单个 `Paxos` 组

![2035097-20200806224202777-988561656](https://github.com/lzlcs/image-hosting/raw/master/2035097-20200806224202777-988561656.2ppu8wqi02w0.webp)







