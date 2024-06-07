## java实现Raft协议

###### 练习：myraft3

###### 模拟 选举过程，节点加入、退出

1. 分别运行**Server1.java**，**Server2.java**，**Server3.java**，**Server4.java**，**Server5.java**，相当于五个节点
2. 假设先运行的是 <u>Server1</u>，输出日志如下

```
start  //表示启动
followerToCandidate //当前没有leader，Server1转变为candidate
start election  //Server1发起一次选举
start election  //由于此时没有别的节点，因此它得不到选票
start election
超时未完成选举，从Candidate转变为Follower  //选举超时
candidateToFollower  //已经从Candidate转变为Follower
followerToCandidate  //还是没有leader，再次转变为candidate
//在没有别的节点加入的情况，会不断循环这个过程
```

3. 然后运行 <u>Server2</u>，<u>Server2</u>的输出日志如下

```
start  //启动
...GET MESSAGE: server election 4444 1  
get an election  //收到Server1发来的选举申请
...GET MESSAGE: server election 4444 1
get an election
refuse 1   // 拒绝Server1选举申请
...GET MESSAGE: server election 4444 1
get an election
refuse 1
followerToCandidate  //转变成candidate
start election  //发起选举
start election
...GET MESSAGE: server vote 1  
获得选票，当前选票来自 1  //收到来自Server1的选票
```

一张选票没有过半数，所以还是没有leader

而Server1的输出日志新增：

```
...GET MESSAGE: server refuse  //收到反对票
获得反对票，从Candidate转变为Follower
candidateToFollower  //转变回follower
...GET MESSAGE: server election 5555 2  
get an election  //收到Server2的选举申请
...GET MESSAGE: server election 5555 2
get an election
vote for 2  //给Server2投票
reset
wait
```

投票规则应该是按，日志至少比自己更新的情况，先到先得投。但是这里没模拟到日志的处理，所以简单的按端口号大小进行投票

4. 再开启Server3，Server4， Server5

   此时Server4的输出日志如下，Server4当选leader，不断发送心跳给follower

```
start
...GET MESSAGE: server election 6666 3
get an election
refuse 3
...GET MESSAGE: server election 4444 1
get an election
refuse 1
followerToCandidate
start election
...GET MESSAGE: server election 6666 3
refuse 3
start election
start election
...GET MESSAGE: server vote 2
获得选票，当前选票来自 1
...GET MESSAGE: server vote 3
获得选票，当前选票来自 2
...GET MESSAGE: server vote 1
获得选票，当前选票来自 3
election***************success!!!
CandidateToLeader
sent heartbeats
sent heartbeats
sent heartbeats
```

其它的Server输出日志：显示收到leader的心跳包

```
start
...GET MESSAGE: server heartBeat
reset
wait
...GET MESSAGE: server heartBeat
reset
```

5. 此时结束Server4，观察其他的输出日志：

   Server5：发现没有leader后，发起选举，然后成为了新的leader

```
followerToCandidate
start election
...GET MESSAGE: server vote 1
获得选票，当前选票来自 1
...GET MESSAGE: server vote 3
获得选票，当前选票来自 2
...GET MESSAGE: server vote 2
获得选票，当前选票来自 3
election***************success!!!
CandidateToLeader
sent heartbeats
sent heartbeats
sent heartbeats
```

此时再启动Server4：收到心跳包，leader还是Server5

```
start
...GET MESSAGE: server heartBeat
reset
wait
...GET MESSAGE: server heartBeat
reset
```

