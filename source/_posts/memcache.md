---
title: 6.824 分布式系统 Scaling Memcache at Facebook
date: 2023-11-21 15:00:00
mathjax: true
categories:
- Papers
tags: 
- Memcache
- Distributed System
---

# 背景

社交网络的基本要求
1. 允许近距离即时交流
2. 从不同源即时聚合内容
3. 可以访问和更新热点的共享内容
4. 扩展到处理每秒数百万个用户需求

# 概览

`memcached` -> 一个开源的存放在内存的哈希表实现 <br>
提供 `set`, `get`, `delete` 操作

![image](https://github.com/lzlcs/image-hosting/raw/master/image.3vdqq5c7thw0.png)

1. 用户操作的特点是浏览的信息量远大于他们创造的信息量 <br>
因此获取数据是主要负载, 缓存可能获得很大的收益
2. 读取操作可能发生在不同的源, 比如 MySQL 或者 HDFS

**查询操作描述**
1. 读操作: 先看 memcache 中缓存是否命中, 否则就去数据库查询
2. 写操作: 先在数据库里更新, 然后再把对应的 kv 从 memcache 中删除
> 为什么要删除而不是更新: 因为删除操作是幂等的, 并且 memcache 不是数据的权威来源, 在分布式系统中可能出现错误

> 为什么选择 memcache 来解决读取数据库的热点问题: 隔离缓存层和持久化层, 这样在工作负载变更的的时候可以分别进行适配

![image](https://github.com/lzlcs/image-hosting/raw/master/image.7caaqb4yyjk0.webp)

# 集群中的延迟和加载

1. 减少获取缓存数据的延迟 
2. 减少缓存未命中后从数据库加载的延迟

## 减少延迟

一个集群中有数以百计的 memcache 服务器, kv 键值对通过一致性哈希从而均匀地分布在这些服务器中 <br>
因此每个 web server 都会访问很多 memcache server, 这就形成了一种多对多的结构
1. 可能造成数据拥塞
2. 单个服务器的性能可能会成为瓶颈

每个 web server 上会有 memcache client, 包含一系列功能 <br>
每个 memcache client 维护当前可用的 memcache server 集合 (配置通过辅助配置系统更新(如 zookeeper))

**并行请求和批处理**

写 web app 的时候, 试图最小化网络来回的次数 <br>
通过建造一个数据依赖的有向无环图, 可以并行获取深度相同的节点

**c-s 交互**

客户端有一个库, 这个库可以被嵌入到应用程序或者作为一个名为 mcrouter 的独立代理 <br>
这个代理提供对 server 的接口并负责将 请求/回复 从其他 server 路由

对于 `get` 操作
1. 客户端使用 UDP 连接, 跳过 mcrouter 直接与 server 连接来提高性能
2. 客户端将丢包等错误视作缓存为命中, 不过在查询数据库之后不会将其插入 memcache, 从而防止网络过载

对于 `delete` 和 `set` 操作
1. 客户端使用 TCP 连接, 通过本机的 mcrouter 执行这两种操作
2. mcouter 把多个 TCP 连接合并成一个 TCP 连接从而减少内存消耗

**拥塞**

client 使用在请求队列上使用滑动窗口机制 <br>
请求被回复时, 适当增加窗口大小, 否则适当减少窗口大小

## 减少加载

### 租赁

1. stale sets: memcache 中的数据被更新, 变成陈旧数据
2. thundering herds: 特定键发生大量读写活动, 反复写入让 cache 无效, 许多读取操作都转到数据库那边

**租约机制**

1. 触发租约: 缓存未命中, 向 server 获取租约
2. 分配租约: 租约与 key 唯一绑定
3. 写入缓存: 写入缓存时需要验证令牌
4. 失效处理: 实现方式很多, 此处略过

对于第一种: 租约协调并发的写入可以解决 <br>
对于第二种: 限制写入速率(10秒发放一个租约), 通知客户端延迟写入

**旧数据**

应用可以自主决定等待新数据还是继续使用旧数据

**Memcache 池**

1. 默认池: 大多数数据
2. small pool: 存储访问频率高, 未命中成本不高的数据
3. large pool: 存储访问频率低, 未命中成本高的数据

## 故障处理

**小规模失效**

一小组机器成为 glutter, 用于故障处理 <br>
client 未收到 server 的回复时, 把 glutter 当作 server

**大规模失效**

将 client 的请求重定向到其他集群

# 在一个区域复制

需求增加的时候, 简单地增加服务器数量可能会造成拥塞

# 分区

server 被分为多个前端集群和一个包含数据库的存储集群 <br>
所有的前端集群都使用相同的 memcache server, 形成区域性池

1. 数据复制: 存储集群存储准确的数据, 可能会把数据复制到前端集群
2. 失效处理: 存储集群修改数据的时候会向前端集群发送失效通知(批处理)
















