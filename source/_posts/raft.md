---
title: 6.824 分布式系统 Raft 共识算法
date: 2023-10-5 18:15:00
mathjax: true
categories:
- Papers
tags: 
- Raft
- Distribute System
---

# 复制状态机

复制状态机通常使用日志复制来解决分布式系统中容错问题

![image](https://github.com/lzlcs/image-hosting/raw/master/image.5hxgfzm5f0g.png)

共识算法的主要工作是保持日志复制的一致性, 每台服务器上的共识模块接收来自
客户端的命令, 并将其添加到日志中

共识算法的特征
1. 确保所有非拜占庭错误的安全性, 从不返回一个错误的结果 (即使是网络延时, 分区, 数据包丢失)
2. 只要过半服务器是可运行的, 并且可以和客户端通信, 那么共识算法就可用 <br>
   服务器崩溃之后, 他们可能会已经存储的状态来进行恢复, 重新加入集群
3. 它们在保证日志一致性上不依赖于时序
4. 过半的服务器响应了单轮 RPC 命令就可视为完成

> 拜占庭错误: 伪造信息恶意响应的错误
> 非拜占庭错误: `fail-stop` 不响应, 但不会伪造信息

# `Paxos` 存在的问题

1. 难以理解
2. 难以实现

> 几乎所有一开始想实现 `Paxos` 的系统在实现的过程中为了解决一些难题然后慢慢修改成跟 `Paxos` 完全不一样的架构

# 为可理解性设计

1. 问题分解: `leader` 选举, 日志复制, 安全性, 成员变更
2. 减少状态的数量来简化状态空间: 比如使用随机化算法

# `Raft` 共识算法

`Raft` 是用来管理 复制日志 的算法 

`Raft` 首先选举一个 `leader` , 由该 `leader` 全权负责复制日志的一致性 <br>
`leader` 从客户端接收日志条目, 然后复制给其他服务器 <br>
当 `leader` 失联的时候, `Raft` 就选择一个新的 `leader` 

## `Raft` 基础

集群中的每一个节点都只能是以下三种身份之一
1. `leader`: 处理来自客户端的请求(如果客户端跟 `follower` 通信, `follower` 会将请求重定向到 `leader` 上)
2. `follower`: 被动响应来自 `leader` 和 `candidate` 的请求
3. `candidate`: 选举时出现的临时状态

![image](https://github.com/lzlcs/image-hosting/raw/master/image.7cwjtz1wgww0.png)

1. `none -> follower`: 集群启动
2. `follower -> candidate`: 集群中没有 `leader` 或 `leader` 失联
3. `candidate -> follower`: `candidate` 选举失败
4. `candidate -> leader`: `candidate` 选举成功
5. `leader -> follower`: `leader` 失联之后重新连接 或者 发现其他节点有更新的数据

![image](https://github.com/lzlcs/image-hosting/raw/master/image.1nqynw289hsw.webp)

`Raft` 将时间分为任意长度的任期, 任期从选举开始 <br>
选举失败之后, `Raft` 会很快开始新一轮选举

任期还包含时间戳的作用, 可以让集群发现过期的信息:
1. 节点通信的时候会交换任期号, 如果比对方小就更新
2. `candidate` 或 `leader` 发现自己的任期号过期了, 那么就回到 `follower` 状态
3. 如果一个节点接收了带过期任期号的请求, 那么他会拒绝此次请求

`Raft` 算法中服务器节点使用 RPC 进行通信, 一般的共识算法只需要两种 RPC
1. `RequestVote RPCs`: 由 `candidate` 在选举过程中发出
2. `AppendEntries RPCs`: 由 `leader` 发出, 用来做日志复制和心跳机制

(`Raft` 的第三种 RPC: 在节点中传输快照)

## `leader` 选举

一个服务器节点只要从 `candidate` 或 `leader` 接收到有效的 RPC 就一直保持 `follower` 的状态
`leader` 会周期性地向所有的 `follower` 发起心跳来维持自己的 `leader` 地位
> 心跳是不包含日志条目的 `AppendEntries RPCs`

一段时间内 `follower` 没有接收到其他 `candidate` 或 `leader` 的信息 (选举超时) <br>
那么开始选举, 自增选举号并给自己投票, 同时发送 `RequestVote RPCs` 给其他节点 <br>
转换为 `candidate` 状态持续到如下事件发生
1. 它获得超过半数的选票成为 `leader`
> 当 `candidate` 获取集群中过半节点针对同一任期的投票, 他就赢得选举成为 `leader`
2. 其他节点赢得了选举
3. 一段时间内没有其他节点赢得选举


对于同一个任期, 每个节点只给来求票的第一个 `candidate` 投票 <br>
`candidate` 选举成功之后, 他会向其他的所有节点发送心跳信息停止其他选举行为

一个 `candidate` 可能在等待投票的过程中, 收到其他自称 `leader` 的心跳
1. 如果对方的任期号小, 那么忽略
2. 如果对方的任期号大于等于自己, 那么转换为 `follower` 状态

如果 `candidate` 既没有赢也没有输, 比如所有的 `follower` 同时变成 `candidate` 都把票投给自己 <br>
采用随机选举超时时间, 总会有一个 `candidate` 率先结束超时, 成为 `leader` 然后发送心跳给其他节点

`candidate` 在等待选票的时候, 也会设置一个随机的超时时间, 这样可以防止多次投票无果的情况

## 日志复制

每个客户端请求都包含一条将被复制状态机执行的命令

1. `leader` 以一个新条目的形式把该命令追加到自己的日志中
2. 以同步的形式向集群中的其他节点发送 `AppendEntries RPCs` <br>
3. 当条目被安全的复制之后, `leader` 将条目应用到自己的状态机
4. 状态机执行该指令, 然后将结果返回客户端

如果 `follower` 故障, `leader` 会不断进行 `AppendEntries RPCs` (即使已经对客户端有了响应)
直到所有 `follower` 都存储了所有的日志条目

![image](https://github.com/lzlcs/image-hosting/raw/master/image.2pqmlr5z2we0.webp)

日志由有序编号的日志条目组成, 每个日志条目存储它被创建时的任期号和日志内容, 如果一个日志被复制到
大多数服务器上, 就被认为可以提交了
1. 如果不同日志的的两个条目由相同的日志号和任期号, 则他们所存储的内容是相同的 
> `leader` 在一个 `term` 内在给定的一个日志编号最多创建一条日志条目, 该条目在日志中的位置也从来不会被改变
2. 如果不同日志的两个条目有着相同的日志号和任期号, 则他们之前的条目都是完全一样的
> 当发送一个 `AppendEntries RPC` 时, `leader` 会把紧接着之前的日志条目的 日志编号和内容 都传过来, 如果 `follower` 没有在它的日志中找到相同的 日志编号和内容 的日志, 就会拒绝新的日志条目

**不一致处理**

会不断向前找 `follower` 直到找到和自己日志条目相同的位置, 然后复制直到最新的日志条目

## 安全

完善算法, 使得算法在各种服务器故障中仍然可用

**`leader` 故障**

1. 被选出来的 `leader` 一定包含所有被提交的日志条目: <br>
   如果投票者自己的日志(任期号和日志号)比 `candidate` 的还要新, 那么它就会拒绝投票
2. 只有 `leader` 当前任期内的日志条目才通过计算副本数目的方式来提交

**`follower` 和 `candidate` 故障**

1. `leader` 无限重试发送给他们的 RPC 
2. 如果完成了 RPC 但是在回复之前宕机了, 恢复之后会收到同样的请求, 此时重新做一遍(RPC 是幂等的, 再做一遍不影响结果)

**安全与可用性限制**

$$
广播时间 << 选举超时时间 << 平均故障时间
$$

# 集群成员变更

`Raft` 实现了自动变更集群成员, 难点是处理脑裂问题

`Raft` 使用一种两阶段的方法: 集群先切换到一个过渡的配置: 称为联合一致
1. `leader` 发起 $C_{old, new}$, 使整个集群进入这个状态 <br>
   所有 RPC 都要在新旧两个配置都达到大多数才算成功
2. `leader` 发起 $C_{new}$, RPC 在新配置中达到大多数才算成功

这两个请求被包装成普通的 `AppendEntries RPC` <br>
只要服务器收到了这种配置日志条目, 它就会用新配置来做出决策, 无论其是否被提交

**故障处理**

1. `leader` 在 $C_{old,new}$ 未提交前故障
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2jnu9s3xyds0.png)
如果新 `leader` 没有 $C_{old,new}$, 那么变更失败 <br>
如果新 `leader` 有 $C_{old, new}$ 那么继续变更
2. `leader` 在 $C_{old,new}$ 已提交但 $C_{new}$ 未发起时故障 <br>
已提交之后, 根据安全性规则, 有 $C_{old,new}$ 的才有资格当选 `leader`

3. `leader` 在 $C_{new}$ 已发起时故障 <br>
此时 $C_{old,new}$ 已经复制到大多数节点, 不需要管老节点了 <br>
$C_{old,new}$ 按照新老规则选举, $C_{new}$ 按照新规则选举 <br>
如果 $C_{old,new}$ 当选, 那么它继续发出 $C_{new}$ <br>
如果 $C_{new}$ 当选, 那么它继续发出 $C_{new}$

补充规则
1. 新增节点时, 需要等新增的节点完成日志同步再开始集群成员变更
2. 缩减节点时, `leader` 可能本身就是要缩减的节点, 完成 $C_{new}$ 提交之后自动退位
3. 如果在选举超时时间内, `follower` 收到 `candidate`的投票请求, 那么忽略此请求: 防止下线的服务器选举影响 `raft` 的正常运行

# 日志压缩

1. 增量压缩如日志清洗 和 日志结构合并树 (LSM 树)
2. 快照技术: 某个时间点前整个系统的状态以快照的方式持久化

![image](https://github.com/lzlcs/image-hosting/raw/master/image.66zalv6td740.webp)

发送快照 `InstallSnap RPCs`

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6k0ko0xgtzk0.webp)

1. 直接替换
2. 替换一部分前缀

**性能问题**

1. 何时生成快照: 简单的策略是等到日志容量到达阈值的时候
2. 写时复制的功能: 操作系统天然支持这样的功能

# 客户端交互

1. 客户端随机挑选一个节点, 如果该节点不是 `leader`, 就给客户端返回最近的一次 `leader` 的信息, 如果 `leader` 崩溃了(超时), 客户端就重新再随机选个节点, 重复以上动作
2. 防止同一条指令被执行两次: 客户端对每一条指令都

**只读操作**

只读操作可能会访问到老 `leader` 从而返回错误的信息 <br>
每个 `leader` 上任的时候, 发送一个空日志 <br>
`leader` 处理只读请求的时候, 必须检查自己是否被替代


