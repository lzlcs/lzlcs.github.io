---
title: 6.824 分布式系统 Lab2 简记
date: 2023-10-27 16:00:00
mathjax: true
categories:
- Labs
tags: 
- Golang
- Raft
- Lab
- Distributed System
---

# 总体架构

建议先总体看完四个 `lab`, 确定好架构再来实际写代码, 不然就会陷入不停重构的泥潭

**注意 2D 之前可以把 log[0].Index 看为 0** <br>
**注意 2C 之前可以忽略代码中调用 rf.persist**

总体架构
```
encodeState 把需要存储的状态转换为字节数组
persist     持久化当前状态
readPersist 读取之前持久化的状态

Snapshot 交由上层调用, 截断日志并加载新的快照

AppendEntries   处理来自其他 Leader 的 AppendEntries RPC
InstallSnapshot 处理来自其他 Leader 的 InstallSnapshot RPC
RequestVotes    处理来自其他 Candidate 的 RequestVotes RPC

Start 新增一条日志

BroadCast      Leader 向其他节点广播, 实现附加日志/快照 和心跳
StartElection  Candidate 向其他节点广播, 试图获取投票

Make 初始化 Raft 结构体
	 -> ElectClock 实现选举时钟的协程
	 -> HeartClock 实现心跳时钟的协程
	 -> ApplyMsg   实现应用日志的协程

toState          进行状态转换
GetState         获取当前 Raft 的状态
setElectionTime  重置选举计时器
lastLog			 获取最后一条日志
Log				 根据下标获取真正的日志条目

```

# 初期准备工作

首先抄论文定义 `Raft` 结构体, `LogEntry` 结构体<br>
定义三种 RPC 的 `Args` 和 `Reply` <br>
定义状态常量
```go
const (
	LEADER    int = 0
	CANDIDATE int = 1
	FOLLOWER  int = 2
)
```
在 `Make` 中初始化一下


实现 `GetState` 函数, 注意加锁

```go
func (rf *Raft) GetState() (int, bool) {

	rf.mu.Lock()
	defer rf.mu.Unlock()

	return rf.currentTerm, rf.state == LEADER
}
```

实现一些辅助函数, 注意之前有哨兵, 所以此处不会越界
```go
func (rf *Raft) lastLog() LogEntry {
	return rf.log[len(rf.log) - 1]
}

func (rf *Raft) Log(trueIndex int) *LogEntry {
	return &rf.log[trueIndex-rf.log[0].Index]
}

func min(a, b int) int {
	if a > b {
		return b
	}
	return a
}

func max(a, b int) int {
	if a < b {
		return b
	}
	return a
}
```

# Lab 2A

## 实现心跳计时器

根据 `Students' Guide`, 最好不要使用 `timer` 或者 `ticker` 这样容易出错的计时器

我的实现是在 `Make` 里 `go` 一个协程, 每 100ms 检查一下是否应该发送心跳了

```go
func (rf *Raft) HeartClock() {
	for !rf.killed() {

		rf.mu.Lock()

		if rf.state == LEADER {

			rf.BroadCast()
		}
		rf.mu.Unlock()

		time.Sleep(100 * time.Millisecond)
	}
}
```

## 实现选举计时器

仍然是在 `Make` 里 `go` 一个协程, 每 20ms 检查一下是否超时 <br>
如果当前状态不是 `Leader` (不能造自己的反) <br>
且当前时间已经在计时器设置的时间之后了, 那么就开始一轮选举

根据 `Students' Guide` 里的选举计时器重置的位置 <br>
第二个就是这里, 我放在 `StartElection` 函数的开头重置选举时间
```go
func (rf *Raft) ElectClock() {

	for !rf.killed() {

		rf.mu.Lock()
		if rf.state != LEADER && time.Now().After(rf.timeout) {
			// new election
			rf.StartElection()
		}
		rf.mu.Unlock()

		time.Sleep(20 * time.Millisecond)
	}
}
```

## 实现选举

严格按照论文的 `Figure 2` 的候选人这节实现
1. 首先变成候选人
	1. 自增任期号
	3. 给自己投票
	1. 重置选举计时器
	2. 发送请求投票的 RPC 给其他所有服务器
1. 如果接收到大多数服务器的选票，那么就变成领导人
1. 如果接收到来自新的领导人的附加日志（AppendEntries）RPC，则转变成跟随者(这部分在之后实现)
1. 如果选举过程超时，则再次发起一轮选举(这部分在 `ElectClock` 实现)
```go
func (rf *Raft) StartElection() {
	total := len(rf.peers)

	rf.setElectionTime()

	rf.state = CANDIDATE
	rf.currentTerm += 1
	rf.votedFor = rf.me
	rf.persist()

	ticketCount := 1

	// 要在外面获取 args 参数, 因为如果在 go func 里面的话
	// 这些参数都有可能会变化
	args := RequestVoteArgs{rf.currentTerm, rf.me, rf.lastLog().Index, rf.lastLog().Term}

	for i := range rf.peers {

		if i == rf.me {
			continue
		}

		// go 一个协程从而(几乎)同时发送 RPC 给其他节点
		go func(server int) {

			reply := RequestVoteReply{}

			// 不能在持有锁的情况下发起 RPC
			if !rf.peers[server].Call("Raft.RequestVote", &args, &reply) {
				return
			}

			rf.mu.Lock()
			defer rf.mu.Unlock()

			// 每次重新获取锁之后, 都要检查状态是否符合预期
			if reply.Term != rf.currentTerm || rf.state != CANDIDATE {
				return
			}

			// Figure 2 中对于所有服务器的规则
			if reply.Term > rf.currentTerm {
				rf.currentTerm = reply.Term
				rf.toState(FOLLOWER)
				rf.votedFor = -1
				rf.persist()
				return
			}

			if !reply.VoteGranted {
				return
			}

			ticketCount += 1

			if ticketCount <= total/2 {
				return
			}

			// Figure 2 中领导人的规则: 变成 Leader 之后就马上广播
			rf.toState(LEADER)
			rf.BroadCast()

		}(i)
	}
}
```

```go
func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {
	// Your code here (2A, 2B).
	rf.mu.Lock()
	defer rf.mu.Unlock()
	defer rf.persist()

	// 接收者实现第一条
	if args.Term < rf.currentTerm {

		reply.VoteGranted = false
		reply.Term = rf.currentTerm
		return
	}

	// 所有服务器规则 第二条
	if args.Term > rf.currentTerm {
		reply.Term = args.Term
		rf.state = FOLLOWER
		rf.votedFor = -1
	}

	// assert args.Term >= rf.currentTerm
	// 接收者实现第二条
	if rf.votedFor != -1 && rf.votedFor != args.CandidateId {
		reply.VoteGranted = false
		return
	}

	term, index := rf.lastLog().Term, rf.lastLog().Index
	// 接收者实现第二条, 至少与自己一样新
	if args.LastLogTerm > term ||
		(args.LastLogTerm == term && args.LastLogIndex >= index) {

		// 重置选举时间的第三种情况
		// vote for args.CandidateId
		rf.setElectionTime()
		rf.toState(FOLLOWER)
		rf.votedFor = args.CandidateId
		reply.VoteGranted = true
	}

	rf.currentTerm = args.Term
}
```

## 实现部分 `BroadCast` 和 `AppendEntriesRPC`

```go
func (rf *Raft) BroadCast() {

	for i := range rf.peers {

		if i == rf.me {
			continue
		}
		go func(server int) {

			rf.mu.Lock()

			if rf.state != LEADER {
				rf.mu.Unlock()
				return
			}

			// 2D 快照部分
			if rf.nextIndex[server] <= rf.log[0].Index {
				// ...
				return
			}

			// 正常发送附加日志请求
			// 2A 可以忽略日志复制, args.Entries 直接置为空
			prevLogIndex := rf.nextIndex[server] - 1
			prevLogTerm := rf.Log(prevLogIndex).Term

			entries := []LogEntry{}
			for i := prevLogIndex + 1; i < len(rf.log)+rf.log[0].Index; i++ {
				entries = append(entries, *rf.Log(i))
			}

			args := AppendEntriesArgs{
				Term:         rf.currentTerm,
				LeaderId:     rf.me,
				PrevLogIndex: prevLogIndex,
				PrevLogTerm:  prevLogTerm,
				Entries:      entries, // []LogEntry{}
				LeaderCommit: rf.commitIndex,
			}

			reply := AppendEntriesReply{}

			rf.mu.Unlock()

			if !rf.peers[server].Call("Raft.AppendEntries", &args, &reply) {
				return
			}

			rf.mu.Lock()
			defer rf.mu.Unlock()

			// 重新获取锁, 检查一下状态
			if rf.state != LEADER || rf.currentTerm != args.Term {
				return
			}

			if reply.Term > rf.currentTerm {
				rf.currentTerm = reply.Term
				rf.toState(FOLLOWER)
				rf.votedFor = -1
				rf.persist()
				return
			}

			// 2B, 2C 部分, 2A 可忽略
			if reply.Success {

				rf.matchIndex[server] = max(rf.matchIndex[server], args.PrevLogIndex+len(args.Entries))

				rf.nextIndex[server] = rf.matchIndex[server] + 1

				n := len(rf.matchIndex)
				tmp := make([]int, n)
				copy(tmp, rf.matchIndex)

				tmp[rf.me] = rf.lastLog().Index

				sort.Ints(tmp)
				newCommitIndex := tmp[n/2]

				if newCommitIndex > rf.commitIndex && newCommitIndex <= rf.lastLog().Index &&
					rf.Log(newCommitIndex).Term == rf.currentTerm {

					rf.commitIndex = newCommitIndex
				}

				return
			}

			rf.nextIndex[server] = max(rf.matchIndex[server]+1, reply.XIndex)

			boundary := max(reply.XIndex, rf.log[0].Index)
			for boundary <= rf.lastLog().Index && rf.Log(boundary).Term == reply.XTerm {
				boundary += 1
				rf.nextIndex[server] = boundary
			}
			// rf.nextIndex[server] = max(rf.matchIndex[server]+1, rf.nextIndex[server]-1)

		}(i)
	}
}


func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {
	rf.mu.Lock()
	defer rf.mu.Unlock()
	defer rf.persist()

	reply.Success = false
	reply.Term = rf.currentTerm

	if args.Term < rf.currentTerm {
		return
	}

	if args.Term > rf.currentTerm {
		rf.currentTerm = args.Term
		reply.Term = args.Term
		rf.votedFor = -1
	}

	// assert args.Term >= rf.currentTerm
	rf.setElectionTime()
	rf.toState(FOLLOWER)

	// 2A 的话此时返回就行
	//...
}
```

最后实现一个 `GetState` 函数, `lab2A` 大部分就完成了

# Lab 2B

## `Start` 函数

代码里很清楚了

## 完成 `AppendEntries` 和 `BroadCast` 函数

**`XTerm`, `XIndex`优化**

```
------情况1-------
Index     12345678
------------------
FOLLOWER  4
LEADER    4666

此时下标超限, XTerm = -1, XIndex = 2
设置 nextIndex 为 XIndex

------情况2-------
Index     12345678
------------------
FOLLOWER  455
LEADER    4666

返回的 XTerm 是冲突日志的任期, 就是 5
返回的 XIndex 是冲突日志的第一条日志下标, 就是 2
如果 Leader 的 XIndex 位置 Term != XTerm, 那么 nextIndex = XIndex

------情况3-------
Index     12345678
------------------
FOLLOWER  4444
LEADER    44666

返回 XTerm 为 4
返回 XIndex 为 1
如果 Leader 的 XIndex 位置 Term == XTerm, 那么往后找到第一条不等于 XTerm 的日志下标 3
```


```go
// BroadCast 部分代码
prevLogIndex := rf.nextIndex[server] - 1
prevLogTerm := rf.Log(prevLogIndex).Term

entries := []LogEntry{}
for i := prevLogIndex + 1; i < len(rf.log)+rf.log[0].Index; i++ {
	entries = append(entries, *rf.Log(i))
}

args := AppendEntriesArgs{
	Term:         rf.currentTerm,
	LeaderId:     rf.me,
	PrevLogIndex: prevLogIndex,
	PrevLogTerm:  prevLogTerm,
	Entries:      entries,
	LeaderCommit: rf.commitIndex,
}

reply := AppendEntriesReply{}

rf.mu.Unlock()

if !rf.peers[server].Call("Raft.AppendEntries", &args, &reply) {
	return
}

rf.mu.Lock()
defer rf.mu.Unlock()

// 重新获取锁, 检查一下状态
if rf.state != LEADER || rf.currentTerm != args.Term {
	return
}

if reply.Term > rf.currentTerm {
	rf.currentTerm = reply.Term
	rf.toState(FOLLOWER)
	rf.votedFor = -1
	rf.persist()
	return
}

// 如果成功的话
if reply.Success {

	// 这里要用之前的 args.PrevLogIndex+len(args.Entries) 更新
	// 因为现在是重新获取锁, rf.lastLog().Index 可能有变化
	rf.matchIndex[server] = max(rf.matchIndex[server], args.PrevLogIndex+len(args.Entries))

	rf.nextIndex[server] = rf.matchIndex[server] + 1

	// 下面这段是排序取中位数, 更新 rf.commitIndex
	n := len(rf.matchIndex)
	tmp := make([]int, n)
	copy(tmp, rf.matchIndex)

	tmp[rf.me] = rf.lastLog().Index

	sort.Ints(tmp)
	newCommitIndex := tmp[n/2]

	if newCommitIndex > rf.commitIndex && newCommitIndex <= rf.lastLog().Index &&
		rf.Log(newCommitIndex).Term == rf.currentTerm {

		rf.commitIndex = newCommitIndex
	}

	return
}

// 这部分是加了优化的, 不加优化的朴素跳转是下面的注释段
rf.nextIndex[server] = max(rf.matchIndex[server]+1, reply.XIndex)

boundary := max(reply.XIndex, rf.log[0].Index)
for boundary <= rf.lastLog().Index && rf.Log(boundary).Term == reply.XTerm {
	rf.nextIndex[server] = boundary + 1
	boundary += 1
}
// rf.nextIndex[server] = max(rf.matchIndex[server]+1, rf.nextIndex[server]-1)
```

```go
// AppendEntries 部分代码

// 情况 1
if args.PrevLogIndex > rf.lastLog().Index {
	reply.XIndex = rf.lastLog().Index + 1
	reply.XTerm = -1
	return
}

// 情况 2, 3
if rf.Log(args.PrevLogIndex).Term != args.PrevLogTerm {

	// 冲突任期
	reply.XTerm = rf.Log(args.PrevLogIndex).Term

	prevIndex := args.PrevLogIndex - 1
	for prevIndex >= rf.log[0].Index && rf.Log(prevIndex).Term == reply.XTerm {
		prevIndex -= 1
	}

	reply.XIndex = prevIndex + 1

	return
}

for _, entry := range args.Entries {
	if entry.Index >= len(rf.log)+rf.log[0].Index {
		rf.log = append(rf.log, entry)

	// 这里一定要加判断, 不能直接截断
	// 如果是延迟的正确 RPC, 截断可能会丢失之后的日志
	} else if entry.Term != rf.Log(entry.Index).Term {
		rf.log = rf.log[:entry.Index-rf.log[0].Index]
		rf.log = append(rf.log, entry)
	}
}

rf.commitIndex = max(rf.commitIndex, min(args.LeaderCommit, rf.lastLog().Index))
reply.Success = true
```

## `ApplyMsg`

```go
func (rf *Raft) applyMsg() {

	for !rf.killed() {

		// 每 10ms 判断一下有没有日志需要发送
		time.Sleep(10 * time.Millisecond)

		rf.mu.Lock()

		// 发送快照的部分
		if rf.lastApplied < rf.log[0].Index {
			tmp := ApplyMsg{

				SnapshotValid: true,
				Snapshot:      rf.snapshot,
				SnapshotTerm:  rf.log[0].Term,
				SnapshotIndex: rf.log[0].Index,
			}
			rf.mu.Unlock()

			// 发送的时候要解锁, 优化性能
			rf.applyCh <- tmp

			rf.mu.Lock()
			// 要用之前的信息更新
			rf.lastApplied = tmp.SnapshotIndex
		}

		if rf.lastApplied >= rf.commitIndex {
			rf.mu.Unlock()
			continue
		}

		// 把需要发送的日志都复制下来
		commitIndex := rf.commitIndex
		applyEntries := make([]LogEntry, commitIndex-rf.lastApplied)

		firstIndex := rf.log[0].Index
		copy(applyEntries, rf.log[rf.lastApplied+1-firstIndex:commitIndex+1-firstIndex])

		rf.mu.Unlock()

		for _, entry := range applyEntries {
			rf.applyCh <- ApplyMsg{
				CommandValid: true,
				Command:      entry.Command,
				CommandIndex: entry.Index,
				CommandTerm:  entry.Term,
			}
		}

		rf.mu.Lock()
		// 同理之前的信息更新
		rf.lastApplied = max(rf.lastApplied, commitIndex)
		rf.mu.Unlock()
	}
}
```

# Lab 2C

## 三个状态持久化

这部分就是调库保存一下状态即可

```go

func (rf *Raft) encodeState() []byte {
	w := new(bytes.Buffer)
	e := labgob.NewEncoder(w)
	e.Encode(rf.log)
	e.Encode(rf.votedFor)
	e.Encode(rf.currentTerm)
	return w.Bytes()
}

func (rf *Raft) persist() {
	// 2C 第二个参数可写成 nil
	rf.persister.Save(rf.encodeState(), rf.persister.ReadSnapshot())
}

func (rf *Raft) readPersist(data []byte) {
	if data == nil || len(data) < 1 { // bootstrap without any state?
		return
	}

	r := bytes.NewBuffer(data)
	d := labgob.NewDecoder(r)
	var log []LogEntry
	var votedFor int
	var currentTerm int

	if d.Decode(&log) != nil ||
		d.Decode(&votedFor) != nil ||
		d.Decode(&currentTerm) != nil {
		return
	}

	rf.log = log
	rf.votedFor = votedFor
	rf.currentTerm = currentTerm
}
```

然后再 `log`, `voteFor`, `currentTerm` 改变的时候 `persist` 一下就好了

# Lab 2D

## `Snapshot`

```go
func (rf *Raft) Snapshot(index int, snapshot []byte) {
	rf.mu.Lock()
	defer rf.mu.Unlock()

	firstIndex := rf.log[0].Index

	// 边界检查
	if index <= firstIndex || index > rf.lastLog().Index {
		return
	}

	// 日志/快照复制
	var tmp []LogEntry
	rf.log = append(tmp, rf.log[index-firstIndex:]...)
	rf.snapshot = make([]byte, len(snapshot))
	copy(rf.snapshot, snapshot)
	rf.log[0].Command = nil

	// 持久化
	rf.persister.Save(rf.encodeState(), snapshot)
}
```

## `BroadCast`

```go
// 此时应该发送快照
if rf.nextIndex[server] <= rf.log[0].Index {
	args := &InstallSnapshotArgs{
		Term:              rf.currentTerm,
		LeaderId:          rf.me,
		LastIncludedIndex: rf.log[0].Index,
		LastIncludedTerm:  rf.log[0].Term,
		Snapshot:          rf.persister.ReadSnapshot(),
	}
	rf.mu.Unlock()

	reply := &InstallSnapshotReply{}

	if !rf.peers[server].Call("Raft.InstallSnapshot", args, reply) {
		return
	}

	rf.mu.Lock()
	defer rf.mu.Unlock()

	// 重新获取锁的状态检查
	if rf.state != LEADER || rf.currentTerm != args.Term {
		return
	}

	// 经典步骤
	if reply.Term > rf.currentTerm {
		rf.currentTerm = reply.Term
		rf.toState(FOLLOWER)
		rf.votedFor = -1
		rf.persist()
		return
	}

	// 更新一下两个数组
	rf.matchIndex[server] = max(rf.matchIndex[server], args.LastIncludedIndex)
	rf.nextIndex[server] = rf.matchIndex[server] + 1
	return
}
```

## `InstallSnapshot`

```go
func (rf *Raft) InstallSnapshot(args *InstallSnapshotArgs, reply *InstallSnapshotReply) {
	rf.mu.Lock()
	defer rf.mu.Unlock()

	// 前面的还是经典判断
	reply.Term = rf.currentTerm

	if args.Term < rf.currentTerm {
		return
	}

	if args.Term > rf.currentTerm {
		rf.currentTerm = args.Term
		reply.Term = args.Term
	}

	rf.setElectionTime()
	rf.toState(FOLLOWER)

	// 上面重新获取锁了, 所以看看是不是真的还需要发送快照
	if rf.commitIndex >= args.LastIncludedIndex {
		return
	}

	// 截断日志的部分
	if rf.lastLog().Index <= args.LastIncludedIndex {
		rf.log = make([]LogEntry, 1)
	} else {
		rf.log = append([]LogEntry{}, rf.log[args.LastIncludedIndex-rf.log[0].Index:]...)
	}

	// 更新哨兵
	rf.log[0] = LogEntry{
		Term:    args.LastIncludedTerm,
		Index:   args.LastIncludedIndex,
		Command: nil,
	}
	// 复制日志并持久化
	rf.snapshot = make([]byte, len(args.Snapshot))
	copy(rf.snapshot, args.Snapshot)
	rf.persister.Save(rf.encodeState(), args.Snapshot)

	// 更新 lastApplied 和 commitIndex 方便 ApplyMsg 提交
	rf.lastApplied, rf.commitIndex = 0, args.LastIncludedIndex
}
```

