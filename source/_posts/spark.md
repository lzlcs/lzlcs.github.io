---
title: 6.824 分布式系统 Resilient Distributed Datasets
date: 2023-11-17 19:00:00
mathjax: true
categories:
- Papers
tags: 
- Spark
- Distributed System
---


# RDD 和编程模型

**目标**

基于工作集的应用 (多个并行操作重用中间结果)

同时还要保证 自动容错, 可伸缩性, 位置感知性调度 这些优势

实现容错性使用 记录更新日志的方式 (而不是使用操作成本高的数据检查点) <br>
并且使用粗粒度转换, 即在大量记录上执行的单个操作

**抽象**

RDD 是只读的, 分区记录的集合

> 转换: 对 RDD 的操作
> 动作: 向应用程序返回值或向存储系统导出数据的操作

`Spark` 使用延迟计算, 当动作发生的时候, 才会计算新的 RDD

缓存: 用户可以请求将 RDD 缓存 <br>
分区: 用户可以请求将 RDDs 按照某种规则分区

```scala
lines = spark.textFile("...")
errors = lines.filter(_.startsWith("ERROR"))

errors.cache()
errors.count() 
```

直到最后一行才开始缓存 RDD

**对比**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.3fs7a0velx60.png)


**编程接口**

开发者写一个 `driver` 程序连接到集群来运行 `worker`

# 接口

![image](https://github.com/lzlcs/image-hosting/raw/master/image.7hvk3tcsybc0.webp)


