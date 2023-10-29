---
title: 6.824 分布式系统 Lab2 简记
date: 2023-10-8 12:15:00
mathjax: true
categories:
- Labs
tags: 
- Golang
- Raft
- Lab
- Distributed System
---

# Lab 2A

## 初期准备工作

首先抄论文定义 `Raft` 结构体, `LogEntry` 结构体, 定义状态常量 <br>
这里需要记录状态和两个计时器
```go
const (
	LEADER    int = 0
	CANDIDATE int = 1
	FOLLOWER  int = 2
)

type LogEntry struct {
	Command   interface{}
	Term      int
	Index	  int
}

type Raft struct {
	mu        sync.Mutex          // Lock to protect shared access to this peer's state
	peers     []*labrpc.ClientEnd // RPC end points of all peers
	persister *Persister          // Object to hold this peer's persisted state
	me        int                 // this peer's index into peers[]
	dead      int32               // set by Kill()

	applyCh chan ApplyMsg
	state   int			    // 
	timeout time.Time		// 选举超时时间

	currentTerm int
	votedFor    int
	log         []LogEntry

	commitIndex int
	lastApplied int

	nextIndex  []int
	matchIndex []int

	snapshotData []byte
}
```
在 `Make` 中初始化一下

```go
func Make(peers []*labrpc.ClientEnd, me int,
	persister *Persister, applyCh chan ApplyMsg) *Raft {

	// Your initialization code here (2A, 2B, 2C).
	rf := &Raft {

		peers:		  peers,
		persister:	  persister,
		me:			  me,
		dead:		  0,
	
		state:		  FOLLOWER,
		applyCh: 	  applyCh,
	
		currentTerm:  0,
		voteFor: 	  -1,
		// 注意论文中日志下标以一开头, 这里初始化一个哨兵
		log:          make([]LogEntry, 1),
	
		commitIndex:  0,
		lastApplied:  0,
	
		nextIndex:    []int{},
		matchIndex:   []int{},
	
		heartTimer:   time.NewTimer(100 * time.Millisecond),
		electTimer:	  time.NewTimer(0),
	}

	rf.readPersist(persister.ReadRaftState())

	go rf.ticker()

	return rf
}
```

先照着论文把两种 `RPC` 的结构体都定义一下 <br>
`AppendEntries` 抄一下 `RequestVote`, 定义两个函数 `AppendEntries` 和 `sendAppendEntries`

```go
type RequestVoteArgs struct {
	Term		 int
	CandidateId  int
	LastLogIndex int
	LastLogTerm  int
}

type RequestVoteReply struct {
	Term        int
	VoteGranted bool
}

func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {
	
}

func (rf *Raft) sendRequestVote(server int, args *RequestVoteArgs, reply *RequestVoteReply) bool {
	ok := rf.peers[server].Call("Raft.RequestVote", args, reply)
	return ok
}

type AppendEntriesArgs struct {
	Term		 int
	LeaderId	 int
	PrevLogIndex int
	PrevLogTerm  int
	Entries		 []LogEntry
	LeaderCommit int
}

type AppendEntriesReply struct {
	Term     int
	Success  bool
}

func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {

}

func (rf *Raft) sendAppendEntries(server int, args *AppendEntriesArgs, reply *AppendEntriesReply) bool {
	ok := rf.peers[server].Call("Raft.AppendEntries", args, reply)
	return ok
} 
```

实现 `GetState` 函数, 注意加锁

```go
func (rf *Raft) GetState() (int, bool) {

	rf.mu.Lock()
	defer rf.mu.Unlock()

	return rf.currentTerm, rf.state == LEADER
}
```

实现两个辅助函数, 注意之前有哨兵, 所以此处不会越界
```go
func (rf *Raft) lastLog() LogEntry {
	return rf.log[len(rf.log) - 1]
}

func (rf *Raft) firstLog() LogEntry {
	return rf.log[0]
}
```

## 实现心跳

