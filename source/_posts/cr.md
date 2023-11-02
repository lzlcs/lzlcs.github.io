---
title: 6.824 分布式系统 Chain Replication && CRAQ
date: 2023-10-30 18:00:00
mathjax: true
categories:
- Papers
tags: 
- Chain Replication
- Distributed System
---

# 朴素 `Chain Replication`

## 背景

在诸如 `Primary/Backup`, `Raft`, `Paxos` 这类的备份模型中, `Leader` 这类的节点需要向所有节点进行广播从而实现同步, 但是在服务器不断增加的情况下主节点承受的压力越来越大, 所以就出现了性能瓶颈

链式复制 (`Chain Replication`) 被提出 <br>
主要思路是每个节点只向一个节点发送同步请求

## 架构

![image](https://github.com/lzlcs/image-hosting/raw/master/image.k8ui2y7x9ww.png)

**以 `Fail-Stop` 为基础**

1. 服务器失效了就停止服务
2. 环境有能力知道哪个服务器故障了

**永不失败的 `Master`**

假设有一个永不失败的 `Master` (可以理解为 `Paxos/Raft/Zab`集群)

1. 实现心跳
2. 通知链上服务器它的前后节点
3. 通知客户端链的头尾节点

## 实现

每个节点: <br>
维护 `Pending` 表示未处理完成的请求 <br>
维护 `Hist` 存储 `Tail` 已经 `update` 完成的请求

写请求从 `Master` 到 `Head`, 之后依次传递到每个节点

读请求访问 `Tail` 节点, 返回 `Tail` 节点本地的数据

## 容错

**头节点故障**

头节点存储的 `Pending` 请求全部丢失

**尾节点故障**

倒数第二个节点的 `Pending` 全变成 `Hist`, 它成为新的尾节点

**中间节点故障**

每个服务器存储一个 `send`, 表示往后续节点发送了请求, 但是尾节点没有处理过的请求

尾节点处理之后, 向前置节点发送 `ACK` 让其删除 `send`

`Master` 通知它的前置后继节点让其更新

# `CRAQ`

## 主要思想

主要思想是均摊 `Tail` 节点处理读请求的压力

1. 每个节点存储多个版本
2. 当节点接收到新版本时附加到版本库中
3. 如果该节点不是 `Tail` 节点, 那么设置为 `dirty` 状态
3. 如果该节点是 `Tail`, 那么标记为 `clean`, 向回传 `ACK`
1. 当节点收到后继节点传来的 `ACK` 消息时, 删除版本库中所有的内容, 然后标记为 `clean`

**处理读请求**

* 如果 `object` 的最新版本状态为 `clean`, 那么直接返回
* 如果 `object` 的最新版本状态为 `dirty`, 向 `Tail` 请求最新版本号, 并返回该版本的值

**隐式 `clean/dirty`**

如果只有一个版本, 那么就是 `clean`, 否则就是 `dirty`

## 一致性模型

`CRAQ` 提供三种强一致性模型

1. 强一致性模型: 从任意节点读取到最新版本的数据
2. 最终一致性模型: 返回节点已知的最新版本数据(`clean`)
3. 具有最大不一致边界的最终一致性: 返回最新版本数据 (`dirty`)

