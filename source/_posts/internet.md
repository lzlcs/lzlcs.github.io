---
title: 计算机网络
date: 2023-09-16 16:14:00
mathjax: true
tags:
- Internet
categories: 
- Base
---

# Chapter 1

## 因特网概述

1. 若干节点和链路互联形成网络
2. 若干网络通过路由器互联形成互联网
3. 因特网是世界上最大的互联网


`Internet Service Provider` 因特网服务提供者, 如电信移动联通 \
用户先接入 `ISP` 之后再接入因特网

**因特网的组成**

1. 边缘部分: 用户直接使用的主机, 进行通信和资源共享
2. 核心部分: 大量网络和路由器组成, 提供连通性和交换

**三种交换方式**

1. 电路交换: 
    - 建立连接
    - 通话
    - 释放连接
1. 报文交换:
    - 把要传输的数据(报文)整个发送出去
2. 分组交换:
    - 报文分组之后加上首部
    - 分别通过路由器网络发送到目标位置
    - 组合各组成为完整报文

![](https://github.com/lzlcs/image-hosting/raw/master/image.7fkr0a82w8w0.png)

**性能指标**

![](https://github.com/lzlcs/image-hosting/raw/master/image.76c1gair1ps0.webp)

1. 速率: `bps`: 表示数据传送的速率 `min(主机接口速率, 线路带宽, 路由器接口速率)`
   ![](https://github.com/lzlcs/image-hosting/raw/master/image.622gbg6uucg0.webp)
2. 带宽: `bps`: 表示数据传送的能力
3. 吞吐量: 单位时间内通过某个网络或接口的实际数据量
4. 时延: 数据传送耗费的时间
   ![](https://github.com/lzlcs/image-hosting/raw/master/image.2pb9mo6nd9u0.webp)
5. 时延带宽积: $传播时延 \times 带宽$
   - 发送的第一个比特到达终点时, 发送端已经发送了 时延带宽积 个比特
6. 往返时间 `(RTT)`
7. 利用率: 利用率越高, 时延越高
    - 信道利用率: 某个信道有数据通过的时间占比
    - 网络利用率: 网络中所有信道的利用率加权平均
8. 丢包率: 一定时间内丢失的分组数量比全部分组数量
    - 传输过程中出现误码
    - 到达的分组交换机队列已满

## 计算机网络体系结构

**常见的结构**

`TCP/IP`: 网络接口层, 网际层, 运输层, 应用层 \
原理体系结构(用于教学): 物理层, 数据链路层, 网络层, 运输层, 应用层

![](https://github.com/lzlcs/image-hosting/raw/master/image.ap3xsleznq0.png)

**术语**

1. 实体: 任何可发送或可接收信息的硬件或软件进程 \
   对等实体: 收发双方相同层次的实体
2. 协议: 控制两个对等实体进行逻辑通信的规则的集合
    - 语法: 定义所交换信息的格式
    - 语义: 定义接收双方所要完成的操作
    - 同步: 定义收发双方的时序关系
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.5kwzh7x7wro0.png)
3. 服务: 两个对等实体之间的协议使得他们能向上一层提供服务

![](https://github.com/lzlcs/image-hosting/raw/master/image.3nfltb1mk460.webp)


# 物理层

**传输媒体**

1. 导引型传输媒体: 同轴电缆, 双绞线, 光纤, 电力线
2. 非导引型传输媒体: 无线电波, 微波, 红外线, 可见光

**传输方式**

串行, 并行 \
同步, 异步 \
单工, 半双工, 全双工

**编码和调制**

编码: 
![](https://github.com/lzlcs/image-hosting/raw/master/image.39viprf3a4u0.webp)
调制: 
![](https://github.com/lzlcs/image-hosting/raw/master/image.6ggh08kbc1k0.webp)

**信道的极限容量**

![](https://github.com/lzlcs/image-hosting/raw/master/image.31p7lezchzu0.webp)

**信道复用技术**

1. 频分复用: 子信道占据一定频带, 频带之间有隔离频带
2. 时分复用: 类似 CPU 并行
3. 波分复用: 光的频分复用
4. 码分复用: 谁发乘谁

# 链路层

数据链路层向上层提供的服务模型
1. 不可靠传输服务: 丢弃有误码的帧
2. 可靠传输服务: 实现准确的信息传送 (不局限于链路层)

**封装成帧**

![](https://github.com/lzlcs/image-hosting/raw/master/image.6t9vhndrirg0.webp)

**透明传输**

透明传输对上层传下来的数据没有任何要求

面向字节的物理链路: 字节填充
![](https://github.com/lzlcs/image-hosting/raw/master/image.1zzorbcw6a2o.webp)
面向比特的物理链路: 比特填充
![](https://github.com/lzlcs/image-hosting/raw/master/image.50y01r0rz2o0.webp)

最大传送单元 (MTU): 帧的数据部分的长度上限

**差错检验**

1. 奇偶校验: 添加一位奇偶校验码, 但是在偶数个位出错的时候会漏检
2. 循环冗余校验(CRC): [视频](https://www.bilibili.com/video/BV1NT411g7n6?p=20)

**可靠传输**

1. 停止等待协议
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.3bsjjwyu57c0.webp)
2. 回退 N 帧协议
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.5c5wfavh0rk0.webp)
3. 选择重传协议
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.76k0knf06ko0.png)

**点对点协议 PPP**

1. 帧格式 
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.38e5hht4mrg0.webp)
2. 透明传输
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.2jzk39pf2gy0.png)
    ![](https://github.com/lzlcs/image-hosting/raw/master/image.7c7fxkefow80.webp)

**网络适配器(网卡)**

![](https://github.com/lzlcs/image-hosting/raw/master/image.2zgynsooagw0.webp)

**`MAC`地址**

以太网卡用于接入有限局域网 \
WiFi 网卡用于接入无限局域网

MAC 地址是对网络上各接口的唯一标识

![image](https://github.com/lzlcs/image-hosting/raw/master/image.wbgtpas4lgg.webp)

网卡从网络中接收的每一个帧, 检查帧首部的 MAC 地址

1. 广播地址 (`FF-FF-FF-FF-FF-FF`), 接收帧
2. 网卡支持的多播地址, 接收帧
3. 网卡的全球单播 MAC 地址与之符合: 接收帧

混合模式: 不管 MAC 地址是什么, 都接收帧

**CSMA/CD 协议**

共享式以太网使用

载波监听多址接入/碰撞检测

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6o5964bv2j80.webp)

不能进行全双工通信, 只能进行半双工通信
![image](https://github.com/lzlcs/image-hosting/raw/master/image.613hy4rquq00.png)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.32xbmelxuii0.webp)

使用集线器连线可以形成星型网络

**网桥**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.2hhyerfyi5w0.webp)

自学习和转发流程
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4esn18mt7eg0.png)

生成树协议 STP

多个透明网桥来提供冗余链路

为防止回路, 网桥之间产生生成树, 当出现故障的时候就会重新生成生成树

**交换式以太网**

仅使用交换机的以太网就是交互式以太网

交换机的接口连接交换机或计算机的时候, 可以使用全双工工作方式

![image](https://github.com/lzlcs/image-hosting/raw/master/image.7240fgicbhs0.webp)

交换机的接口连接集线器的时候, 只能使用 CSMA/CD 协议工作在半双工模式

一般的交换机使用存储转发, 有一些交换机使用直接转发

直接转发:
1. 时延很小
2. 不检查纠错码将帧转发出去

**共享式以太网和交换式以太网区别**

交换机每个端口连接的网络 构成一个独立的冲突域 \
交换机互联的工作站构成一个整体的广播域

集线器互联的工作站构成整体的广播域和冲突域
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3h73vzop7gi0.webp)

**虚拟局域网 `VLAN`**

交换式以太网的巨大广播域会产生一些问题: 广播风暴


以太网交换机进行配置之后, 把一些接口分配 PVID 形成局域网 \
广播的时候交换机把传入的数据打上标签, 然后传出的时候再去标签
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5xzg2oy1cyw0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.7bz9pzovpm40.webp)

**无限局域网 `WLAN`**

1. 有固定基础设施的无线局域网
2. 无固定基础设施的无限局域网

固定基础设施是预先建立的多个固定的通讯基站

无线局域网的最小构件是 基本服务集 BSS, 每个 BSS 有一个接入点 AP \
每个 AP 在安装的时候会被分配一个 唯一的服务器标识符 SSID

每个 AP 可以连接到 分配系统 DS, 从而和其他的 BSS 连接从而构成扩展服务集 ESS

主机从一个服务集漫游到另一个服务集的时候, 仍然可以保持和其他主机的连接 \
区别是使用的 AP 不同

1. 关联服务 \
    移动站和接入点 AP 建立关联的办法有两种
    - 被动扫描: 被动等待 AP 传来的信标帧 
        ![image](https://github.com/lzlcs/image-hosting/raw/master/image.4jd2ouqauly0.webp)
    - 主动扫描: 向 AP 传探测请求帧, 收到探测相应帧
2. 重建关联服务和分离服务: 更换连接的 AP 和 终止连接 AP 的时候使用

无固定基础设施的无线局域网主要是依靠连接在网络里的节点转发 \
因此每个节点都要有路由功能

自组织网络内部有特定的路由选择协议, 整个内部网络通过网关(协议转换器)连接到因特网

无线局域网使用很多种物理协议 \
数据链路层使用 CSMA/CA 协议
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3ee6kep6zmy0.webp)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6118f48nv600.webp)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.3q84ygcfk1c0.webp)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.3wm8xi4s6te0.webp)

MAC 帧

![image](https://github.com/lzlcs/image-hosting/raw/master/image.5q9z3jo6p2c0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4vl0ffdd8x40.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1zvjvq2gkadc.webp)

# 网络层

将分组通过源主机经过多个网络和多端链路传输到目的主机 \
功能包括分组转发和路由选择

网络层面向上层提供的两种服务
1. 面向连接的虚电路服务
    ![image](https://github.com/lzlcs/image-hosting/raw/master/image.6oyzo56um540.png)
2. 无连接的数据报服务
    ![image](https://github.com/lzlcs/image-hosting/raw/master/image.456qhlftm7w0.webp)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.4s2okmqofso0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.76lzgu70s3s0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.ka84kd1c3k0.webp)

划分子网来缓解 IP 地址被大量浪费的问题

无分类方式易于实现路由聚合
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1q3n3wqf3dxc.png)

**IP 地址和 MAC 地址**

在数据包的传送过程中, IP 地址不改变, MAC 地址改变

如果只使用 MAC 地址通信:

![image](https://github.com/lzlcs/image-hosting/raw/master/image.1kbhlqsy2cow.webp)

使用 地址解析协议 ARP 来通过 IP 地址找到相应的 MAC 地址

![image](https://github.com/lzlcs/image-hosting/raw/master/image.7dvsu3kb4bo0.webp)

**静态路由配置**

1. 设置默认路由: `xxx.xxx.xxx.xxx/0`
2. 设置特定主机路由: `xxx.xxx.xxx.xxx/32`

由于最长前缀匹配的原因, 特定主机路由优先级最高, 默认路由优先级最低

![image](https://github.com/lzlcs/image-hosting/raw/master/image.3lasy0kjsvc0.webp)

**动态路由配置**

1. RIP

![image](https://github.com/lzlcs/image-hosting/raw/master/image.5mpksydlz680.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.21t9j568tmqo.webp)

2. OSPF

链路状态: 本路由器和哪些路由器相邻, 以及相应链路的代价
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3ki5fp1oi680.png)

防止邻居表中路由器太多, 选定指定路由器 DR 和备用指定路由器 BDR \
其他的路由器只与 DR 和 BDR 建立邻居关系, 并通过他们交换信息

3. BDP

每个 AS 都会有至少一个 BGP 发言人

不同 AS 之间通过 BGP 发言人通信, BGP 发言人决定网络内部的较优路线

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6ewn5qlyi5s0.png)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.5gwxkqsl8zs0.webp)

**路由器**

路由器是有多个输入输出端口, 进行转发分组的专用计算机

**ICMP**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.4t2th9hzqo00.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4annwamt7ty0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1sky5l8ui0m8.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.72ohq7d3pi40.png)

典型应用
1. ping
2. 跟踪路由
![image](https://github.com/lzlcs/image-hosting/raw/master/image.zbrzzl444wg.png)

**虚拟专用网**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6nzxr7swjh80.png)

**NAT**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6kuzzajnlrk0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4dlt369n8ow0.png)

**IP 多播**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.5yt6kpu6si80.png)

局域网多播: 有五位 无法映射到 MAC 地址, 所以需要软件过滤以下 \
因特网 IP 多播: 使用 `IGMP` 和 多播路由选择协议

**IGMP**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.2sm435r2y340.png)

**多播路由选择协议**

1. RPM 算法: 先用 RPB 建立广播树, 然后剔除非成员路由器
2. 组共享多播路由选择: 
![image](https://github.com/lzlcs/image-hosting/raw/master/image.79nkq1n84xo0.png)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6be47q3vqho0.png)

**IPV6**


![image](https://github.com/lzlcs/image-hosting/raw/master/image.5ubsszsajsw0.webp)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.t1b6hgebmr4.webp)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.4xcirgi77zw0.png)

![image](https://github.com/lzlcs/image-hosting/raw/master/image.13ng6dbmw0tc.webp)

从 IPV4 过渡到 IPV6
1. 双协议栈, 同时使用 IPV4 和 IPV6 协议栈
2. 隧道技术, 把 IPV6 和 IPV4 互相转换

ICMPv6: 合并原本的 ICMP, IGCM, ARP

![image](https://github.com/lzlcs/image-hosting/raw/master/image.57vsyo7mhbo0.webp)

**SDN**

把网络的数据和控制部分分离, 使用软件控制

**OpenFlow** 

SDN 体系结构中 控制层面和数据层面之间的通信接口

![image](https://github.com/lzlcs/image-hosting/raw/master/image.69sm3v5aimk0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.lpq098csjdc.png)

# 运输层

运输层为运行在不同主机上的应用进程提供直接的逻辑通信服务

![image](https://github.com/lzlcs/image-hosting/raw/master/image.565yw01gsv40.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.ah4tftqcig8.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.59qntf5h0hc0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2mw10shtuzy0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2g1h2zarm8ys.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.i6lsl3z9x1s.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.46k2dvpiaak0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1xgszvzh6068.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.545awauihww0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3o1ux0t46xa0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.7c1c5seudxk0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4ao22qdzxtk0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.32c1ysd3ram0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5io8tk0viso0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.42bqq7tj0t40.webp)

网际层拥塞控制: 主动队列管理 AQM \
随机早期丢弃: RED
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5irhsnxl9cc0.webp)

超时重传时间应略大于往返时间 RTT
![image](https://github.com/lzlcs/image-hosting/raw/master/image.77h01sa8w080.png)

# 应用层

网络应用程序在各种段系统上的组织方式和他们之间的关系
1. 客户/服务器方式( Client/Server, C/S 方式 )
2. 对等方式 (Peer to Peer, P2P)

**DHCP**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.45vnargyh9y0.png)

**DNS**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6zsply9142c0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.62whgb04j6c0.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1fn7pd9j2dz4.png)

**FTP**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6vyv0v1ezec0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.55xi1lj6cq00.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.6ntxj1pr1lg0.webp)

**电子邮件**
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1p3zxbrpq8cg.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1ikdg5j82hk0.png)

MIME:
![image](https://github.com/lzlcs/image-hosting/raw/master/image.39n5eyxw5jc0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.6pjrd6znv8w.png)

**HTTP**
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2xkfjoxvw200.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5gn33t6j58c0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5e2bzsx8bt40.png)

# 网络安全

![image](https://github.com/lzlcs/image-hosting/raw/master/image.47b56gaz7xk0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3f9lbsjtiws0.png)

1. 对称密钥密码体制: DES, 3DES, AES \
    通信双方使用相同的密钥加密和解密
2. 公钥密码体制: RSA \
    被通信的一方公布公钥, 自己留着私钥, 对它通信的乙方使用它的公钥加密之后, 它用私钥解密

报文鉴别: 对于不需要保密内容的情况, 只需要检测报文从而保证完整性 \
MD, MD5, SHA-1, SHA-2, SHA-3, HMAC 等方法加密报文

数字签名: 对报文摘要进行数字签名
![image](https://github.com/lzlcs/image-hosting/raw/master/image.ens7qv7mvw8.png)

实体鉴别: 
![image](https://github.com/lzlcs/image-hosting/raw/master/image.50808vxcj5k0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.hwk8fnfp3xk.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.7gfrl8fusac0.png)

密钥分发:
![image](https://github.com/lzlcs/image-hosting/raw/master/image.6atxjt6oyd40.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.36pskyu3k5w0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.32nvbwton1s0.png)

**访问控制**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.6wk883baunk0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3adu13sg2n00.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1r83zcs570tc.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4z7yvv4zt2w0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1bxawnjxbwcg.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.4wvhj1jwv8q0.png)

1. 物理层安全
![image](https://github.com/lzlcs/image-hosting/raw/master/image.49owm73n9020.png)
早期使用的简单的安全机制
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1w9a8k37ki68.webp)
2. 数据链路层安全
![image](https://github.com/lzlcs/image-hosting/raw/master/image.536cizfm7cc0.png)
3. 网络层安全
![image](https://github.com/lzlcs/image-hosting/raw/master/image.pdnwsbphtkg.webp)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2tmg30ffy400.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2ez9qjzbfqf4.webp)
4. 运输层安全
![image](https://github.com/lzlcs/image-hosting/raw/master/image.17wd3owrvm5c.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.3gw6fcupzeq0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.7avsan900500.png)
5. 应用层安全 \
PGP
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5i0wq54nfxw0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5py26tgqhqo0.png)

**防火墙**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.60qw850oz280.png)
1. 分组过滤路由器
![image](https://github.com/lzlcs/image-hosting/raw/master/image.5fa1ucsx5pw0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.uaenkhm750g.png)
2. 应用网关
![image](https://github.com/lzlcs/image-hosting/raw/master/image.28mu0v47vaf4.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.18zuj1ha6vs0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.2l9jgu8rsey0.png)
![image](https://github.com/lzlcs/image-hosting/raw/master/image.qix53r7ddhc.png)

**网络攻击和防范**
1. 网络扫描
![image](https://github.com/lzlcs/image-hosting/raw/master/image.70zcgstytmc0.png)
2. 网络监听
![image](https://github.com/lzlcs/image-hosting/raw/master/image.1iu4dhanlwo0.png)
3. 拒绝服务攻击
![image](https://github.com/lzlcs/image-hosting/raw/master/image.7ex7tnkwo7g0.png)

