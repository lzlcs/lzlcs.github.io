---
title: 6.824 分布式系统 Lab3 简记
date: 2023-11-16 19:00:00
mathjax: true
categories:
- Labs
tags: 
- Golang
- Raft
- Lab
- Distributed System
---

[`Github`](https://github.com/lzlcs/Courses)
调库可比造轮子简单多了

# 结构体设计

## RPC 设计

首先考虑四种 `RPC` 结构体的设计

因为要容错, 所以根据建议, 为每个客户端分配一个 64 位的大整数

每个客户端还要配备一个 `SeqId` 从而给命令编号

1. `PutAppendArgs`
    * `Op` 表示当前操作是 `Put` 还是 `Append`
    * `Value`, `Key` 
    * `ClientId` 和 `SeqId` 表示客户端 Id 和命令 Id
2. `PutAppendReply`: `Err`
3. `GetArgs`: `Key`, `ClientId`, `SeqId` 同上
4. `GetReply`: 
    * `Err`
    * `Value` 表示查询结果

## `Server` 设计

1. `chans` 记录每个日志下标 对应的 `channel`
3. `client2seq` 记录每个客户端对应的, 已经应用的 `SeqId`
3. `keyValue` 存储键值对
1. `lastApplied`: 记录上次快照的日志下标

## `Client` 设计

加入 `ClientId` 和 `SeqId` 即可

# 核心代码

## 客户端发送 RPC

```go
n := len(ck.servers)
for {
    for si := 0; si < n; si++ {
        srv := ck.servers[si]
        var reply GetReply
        ok := srv.Call("KVServer.Get", &args, &reply)
        if ok && (reply.Err == OK || reply.Err == ErrNoKey) {
            // ErrNoKey 返回空字符串
            return reply.Value
        }
    }
    time.Sleep(50 * time.Millisecond)
}
```

## 客户端处理 RPC

```go
func (kv *KVServer) Get(args *GetArgs, reply *GetReply) {

	oldOp := Op{"GET", args.ClientId, args.SeqId, args.Key, ""}
    // 尝试加入日志
	index, _, isLeader := kv.rf.Start(oldOp)

	if !isLeader || kv.killed() {
		return
	}

	// fmt.Println("SERVER GET: ", args.Key)
    // 得到对应 channel
	ch := kv.getCh(index)

    // 等待被处理
	select {
	case newOp := <-ch:
		if newOp.ClientId != oldOp.ClientId || newOp.SeqId != oldOp.SeqId {
			reply.Err = ErrWrongLeader
		} else {
			reply.Err = OK
			kv.mu.Lock()
			reply.Value = kv.keyvalue[args.Key]
			kv.mu.Unlock()
		}
	case <-time.After(100 * time.Millisecond):
        // 超时重传
		reply.Err = ErrWrongLeader
	}

    // 删除 channel
	kv.mu.Lock()
	delete(kv.chans, index)
	kv.mu.Unlock()
}
```

同时在 `StartKVServer` 中 `go` 一个协程

至于快照就只需要保存 `client2seq` 和 `keyValue` 就可以

```go
func (kv *KVServer) getLog() {
	for !kv.killed() {
		entry := <-kv.applyCh
		// fmt.Println("APPLYCH: ", entry.CommandIndex)

		if entry.CommandValid {

			op := entry.Command.(Op)
            // 如果 SeqId 合法
			if kv.exist(op.ClientId, op.SeqId) {
				kv.mu.Lock()
				if op.Name == "Put" {
					kv.keyvalue[op.Key] = op.Value
				} else if op.Name == "Append" {
					kv.keyvalue[op.Key] += op.Value
				}
                // 操作并记录 SeqId
				kv.client2seq[op.ClientId] = op.SeqId
				kv.mu.Unlock()
			}

            // 判断是否需要生成快照
			if kv.maxraftstate != -1 && kv.rf.Persister.RaftStateSize() > kv.maxraftstate {
				kv.rf.Snapshot(entry.CommandIndex, kv.makeSnapshot())
			}
            // 处理成功, 加入对应 channel
			kv.getCh(entry.CommandIndex) <- op
		} else if entry.SnapshotValid {
			kv.mu.Lock()
            // 如果快照比较新, 就更新
			if entry.SnapshotIndex > kv.lastApplied {
				kv.decodeSnapshot(entry.Snapshot)
				kv.lastApplied = entry.SnapshotIndex
			}
			kv.mu.Unlock()
		}
	}
}
```


