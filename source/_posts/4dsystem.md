---
title: 6.824 分布式系统 Lab4 简记
date: 2023-11-31 14:00:00
mathjax: true
categories:
- Labs
tags: 
- Golang
- Raft
- Lab
- Distributed System
---

[Github](https://github.com/lzlcs/Courses)
# Lab 4A
## RPCs

首先是 RPCs 和容错处理, 这部分和 lab3 一样, 就省略

## New Config

在 `sc.getLog` 中对于 `Raft` 提交的每一个 `Op` 都判断类型, 除了 `Query` 操作, 都需要再增加一个配置

**`JoinConfig`**

```go
func (sc *ShardCtrler) JoinConfig(servers map[int][]string) {
    // copyConfig 就是深拷贝 sc.configs[len(sc.configs)-1]
	newConfig := copyConfig(sc.configs[len(sc.configs)-1])

	// 添加组
	for gid, server := range servers {
		newConfig.Groups[gid] = server
	}

    // 附加到 config list 中
	sc.configs = append(sc.configs, newConfig)
    // 负载均衡之后再讲
	sc.rearrange()
}
```

**`LeaveConfig`**

```go
func (sc *ShardCtrler) LeaveConfig(gids []int) {
	newConfig := copyConfig(sc.configs[len(sc.configs)-1])

	leaveGid := make(map[int]bool)
	// 删除对应的组并记录哪个组被删除
	for _, gid := range gids {
		delete(newConfig.Groups, gid)
		leaveGid[gid] = true
	}

	// 如果该组被删除了, 那就 Shards[i] 置零
	for i := 0; i < len(newConfig.Shards); i++ {
		gid := newConfig.Shards[i]
		if leaveGid[gid] {
			newConfig.Shards[i] = 0
		}
	}

	sc.configs = append(sc.configs, newConfig)
	sc.rearrange()
}
```

**`MoveConfig`**

```go
func (sc *ShardCtrler) MoveConfig(shard, gid int) {
	newConfig := copyConfig(sc.configs[len(sc.configs)-1])
	newConfig.Shards[shard] = gid
	sc.configs = append(sc.configs, newConfig)
}
```

## 负载均衡

目标是尽可能的均匀分配并且移动的 shard 最少

具体算法如下
1. 首先算出每个组平均需要分配多少个 shard, 再算出有几个组需要比其他组多负担一个 shard
    > 比如 10 个 shard, 4 个 组, avg = 2, bonus = 2 
2. 维护一个 `waitToRearrange` 切片, 表示当前未被分配的 shards
1. 首先根据每个组的 shard 数量排序, 可证明如果前 bonus 个组负载更多的时候, 需要移动的 shard 数量最少 <br>
`goal = avg + (i < bonus)`
1. 如果当前组维护的 shard 数量小于 goal, 那么从等待队列中取出, 加满
2. 如果当前组维护的 shard 数量大于 goal, 那么把多余的加到等待队列中
```go
// 用于排序的结构体
type Temp struct {
	gid    int
	shards []int
}

func (sc *ShardCtrler) rearrange() {

	// 取出待负载均衡的配置
	newConfig := &sc.configs[len(sc.configs)-1]
	// 如果没有组, 那么直接返回
	if len(newConfig.Groups) == 0 {
		for i := 0; i < 10; i++ {
			newConfig.Shards[i] = 0
		}
		return
	}

	gid2Shards := make(map[int][]int)
	for shard, gid := range newConfig.Shards {
		gid2Shards[gid] = append(gid2Shards[gid], shard)
	}

	// shard[i] == 0 表明正在等待被分配
	waitToRearrange := gid2Shards[0]
	waitIndex := 0
	groupCount := len(newConfig.Groups)

	// 排序部分
	origin := make([]Temp, 0)
	for gid := range newConfig.Groups {
		if gid != 0 {
			x := Temp{gid, gid2Shards[gid]}
			origin = append(origin, x)
		}
	}

	sort.Slice(origin, func(i, j int) bool {
		// 为了稳定排序, 如果 a == b 就根据 gid 排序
		// 否则在多线程环境中会出错
		a, b := len(origin[i].shards), len(origin[j].shards)
		if a == b {
			return origin[i].gid > origin[j].gid
		}
		return a > b
	})

	// 计算 avg 和 bonus
	avg := 10 / groupCount
	bonus := 10 % groupCount

	for i := 0; i < len(origin); i++ {
		// goal 表示当前组需要几个 shard
		goal := avg
		if i < bonus {
			goal++
		}

		// 这个循环把缺少的都加进来
		for j := len(origin[i].shards); j < goal; j++ {
			origin[i].shards = append(origin[i].shards, waitToRearrange[waitIndex])
			waitIndex++
		}
		// 这个循环把多余的都加入等待队列
		for j := goal; j < len(origin[i].shards); j++ {
			waitToRearrange = append(waitToRearrange, origin[i].shards[j])
		}
	}

	// 重新设置 Shards
	for _, v := range origin {
		for _, s := range v.shards {
			// 这句话总共执行 10 次
			newConfig.Shards[s] = v.gid
		}
	}
}
```
# Lab 4B

摆了 [文章](https://github.com/OneSizeFitsQuorum/MIT6.824-2021/blob/master/docs/lab4.md)

目前遇到的便是文章中提到的空日志问题 <br>
在 Unreliable 的测试中大约百分之一的错误率 <br>
但是目前对空日志检测还没什么头绪, 故搁置

(目前的 github 上只有抄来的半成品)