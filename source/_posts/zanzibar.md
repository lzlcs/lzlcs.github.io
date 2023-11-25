---
title: 6.824 分布式系统 SUNDR
date: 2023-11-23 14:00:00
mathjax: true
categories:
- Papers
tags: 
- SUNDR
- Distributed System
---

# 基本架构

包括 client 和 server

![image](https://github.com/lzlcs/image-hosting/raw/master/image.4g2ti3ldq900.webp)

client 可以远程挂载 SUNDR 文件系统, 通过 SUNDR 获取文件和修改文件

具体的方式是 把 client 对于文件系统的系统调用转换为到 server的 RPCs, 同时给 SUNDR 一个独立的磁盘分区

每个文件系统有唯一的超管可以写入

用户有一对密钥, 通过私钥操作文件系统, 通过交换公钥来验证身份

# SUNDR 协议

获取-修改一致性: 获取操作准确反映之前所有的修改操作 <br>
分叉一致性: 类似字典树的 log 组织形式, 从而保持一致性


# 设计思想

1. client 校验所有 log 的签名
2. 检查最后一个日志条目, 防止被恶意回滚
3. 在 client 构建文件系统树
4. 新增 log 并签名, 上传

# 快照和版本向量

server 维护 log, client 维护 snapshot

i-handle 唯一标识 snapshot <br>
每个 i-handle 对应一个哈希表, 记录所有 inode 的哈希值 <br>
文件被修改时, inode 的哈希值被更新, i-handle 也被更新

版本向量: 每个 client 对 i-handle 的修改次数组成向量














