---
title: 6.824 分布式系统 BitCoin
date: 2023-11-25 18:00:00
mathjax: true
categories:
- Papers
tags: 
- BitCoin
- Distributed System
---


# 概述

1. 用密码学代替信用
2. 去金融中介化
3. 杜绝可逆支付

# 交易

![image](https://github.com/lzlcs/image-hosting/raw/master/image.2v8fwzpbkem0.png)

一笔交易的组成
1. 哈希值: 把上次交易的数据和收款人的公钥组合并哈希
2. 付款人对哈希值进行数字签名

验证交易: 
1. 使用付款人的公钥来验证签名是否正确
2. 使用比特币的分布式记账系统来验证是否有双花的现象

# 时间戳服务器

区块: 大约每 10min, 产生一个区块, 包含这 10min 内所有的交易信息

![image](https://github.com/lzlcs/image-hosting/raw/master/image.5ch0z8hz1580.webp)

1. 前一个区块的哈希值 + 10min内交易的数据
2. 打上时间戳
3. 把上面两个信息合并然后哈希, 形成当前区块的哈希值

# 工作量证明

引入工作量证明机制, 从而解决分布式记账系统中, 谁有记账权的问题

![image](https://github.com/lzlcs/image-hosting/raw/master/image.8hg9wp85ems.webp)

让所有节点去计算一个尽可能小的哈希数值 <br>
$$
Hash(nonce || prev\_hash || tx1 || tx2 || tx3 || ...) < target
$$
计算的难度随着 target 的减小而线性增大 <br>
合理的计算难度应该保持在 10min 算出来一次的水平

为了保证每次都是重新计算, 所以要在区块头部加一个随机数 nonce

**计算的过程(挖矿的过程)**

尝试不同的 nonce 值从而寻找一个解满足上式

**多个决定如何投票**

只要诚实节点的算力总和比攻击者的算力高, 正确的区块链就会最长, 选择最长的区块链作为正确的链即可

因为修改一个区块, 就要修改之后的所有区块 <br>
攻击者提供的算力较少, 所以追不上正确链的增长速度

这样就唯一确定了正确的区块链

**确定计算难度**

区块生成的速度过快: 减少 target 值 <br>
区块生成的速度过慢: 增加 target 值 <br>
从而让每个区块生成的时间维持在 10min 左右即可

# 网络

1. 每一笔新的交易都要向其他节点广播
2. 每个节点把收到的交易信息存到区块中
3. 每个节点为自己的区块寻找工作量证明
4. 找到工作量证明之后, 向其他节点广播这个区块
5. 其他节点验证区块中所有交易都是有效的且没有双花现象时接收区块
6. 记录收到区块的哈希值作为新区快的一部分, 表示接收区块

**接收不同区块的处理**

1. 保存所有区块作为分支, 使用最先过来的区块的分支
2. 下一次工作量证明的时候, 其中一个分支变得更长, 使用最长的分支

**容错**

如果一个节点收到一个区块时发现自己缺了一个区块, 就要去请求这个区块

# 激励

1. 每个新区块的第一笔交易是特殊交易, 是奖励给区块创建者的新货币(货币总数到达一定量的时候, 不产生新货币)
2. 交易手续费: 输出值小于输入值的差价

挖矿奖励大约每四年减半一次

**激励可以减少攻击**

如果攻击者有比诚实节点更多的算力, 为了完成一次双花交易而使用如此庞大的算力不如去做矿工

# 回收磁盘空间

![image](https://github.com/lzlcs/image-hosting/raw/master/image.1hm7fs7g7qxs.webp)

使用默克尔树来压缩数据

如果一些交易已经达成共识(最长的链), 那么可以删去他们, 只保留最新的数据

# 简化的支付验证

用户只需要保存最长链的区块头 (向其他节点查询来保证最长链)

将交易链接到对应的默克尔树分支, 一步一步往上算, 如果结果与区块头的默克尔树相同, 那么就验证成功

# 杂项

**合并和分割交易额**

可以多对多进行交易 (找零过程就是自己输入自己输出)

**隐私问题**

公钥和个人信息分离








