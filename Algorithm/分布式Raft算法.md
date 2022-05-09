# [看动画轻松学会 Raft 算法](https://www.cnblogs.com/Finley/p/14467602.html)

由于 Paxos 算法过于晦涩难懂且难以实现，Diego Ongaro 提出了一种更易于理解和实现并能等价于 Paxos 算法的共识算法 - Raft 算法。

因为 Raft 算法清晰易懂越来越多的开源项目尝试引入 Raft 算法来解决分布式一致性问题。在分布式存储领域基于 Raft 算法构建的项目百花齐放，欣欣向荣。

介绍 Raft 算法的文章早已是汗牛充栋，本文先介绍两个非常优秀的网站:

[The Secret Lives of Data-CN](https://acehi.github.io/thesecretlivesofdata-cn/raft/) 以图文方式介绍 Raft 算法，是非常好的入门材料。将其阅读完后您大概率已经了解了 Raft 算法，如果您仍有疑问可以回来继续阅读本文。

既然您已经回来继续阅读，相信您已经了解 Raft 算法中的Leader 选举、日志复制等基本概念, 但仍有部分疑惑。没关系, 接下来我们会解决这些问题。

[Raft Scope](https://raft.github.io/raftscope/index.html) 是 Raft 官方提供的互动式演示程序，它展示了 Raft 集群的工作状态。您可以用它模拟节点宕机、心跳超时等各种情况。有了 Raft Scope 我们可以亲自“动手” 观察 Raft 集群是如何工作、如何处理各种故障的。

遗憾的是这个程序几乎没有任何说明非常难以上手。本文接下来将先介绍如何使用 Raft Scope 然后用它模拟几种 Raft 集群工作中会遭遇的典型状况。

- [Raft Scope 说明](https://www.cnblogs.com/Finley/p/14467602.html#raft-scope-说明)
- Leader 选举
  - [Follower 超时](https://www.cnblogs.com/Finley/p/14467602.html#follower-超时)
- [日志复制](https://www.cnblogs.com/Finley/p/14467602.html#日志复制)
- [脑裂问题](https://www.cnblogs.com/Finley/p/14467602.html#脑裂问题)
- [网络分区问题](https://www.cnblogs.com/Finley/p/14467602.html#网络分区问题)
- [总结](https://www.cnblogs.com/Finley/p/14467602.html#总结)

# Raft Scope 说明[#](https://www.cnblogs.com/Finley/p/14467602.html#raft-scope-说明)

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227174803261-1454825080.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227174803261-1454825080.png)

可以看到 Raft Scope 界面由三部分组成。

最下方有两个滑块：上面的是进度条您可以拖动它回看刚刚发生过事件，下面的是变速器滑块越靠左系统运行越慢。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227175206816-2128319808.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227175206816-2128319808.png)

左上角部分是一个由 5 个节点组成的 Raft 集群，每个圆圈代表集群中的一个节点。点击节点可以看到它的状态。对话框的右下角有一些按钮，我们可以点击按钮模拟各种状况。我们直接右键点击节点也可以看到这些按钮

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227175741914-1364045635.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227175741914-1364045635.png)[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227175753729-739220448.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227175753729-739220448.png)

这些按钮的功能是:

- stop: 节点停机
- resume: 启动停机的节点
- restart: 将节点立即重启
- time out: 模拟心跳超时，点击按钮后相应节点会认为 Leader 发生了心跳超时。
- request: 向集群提交新的数据

节点中间的数字是节点当前的任期号(Term), 节点的颜色似乎同样是用来表示任期的。
节点可能处于 Follower、Candidate 或者 Leader 状态。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227181605520-1749635100.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227181605520-1749635100.png)

S2 处于 Candidate 状态，实心原点表示它现在收到的投票。图中的两个原点表示收到了 S2 和 S4 的投票，这 5 个小圆点和集群中节点的位置是对应的，左下角的小圆点表示 S4, 最上面的小圆点表示 S1。在集群选举过程中节点外的动态边框表示 Election Timeout。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227181821834-585663929.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227181821834-585663929.png)

黑色实心边框表示 S5 是 Leader。Follower 外面的边框表示 HeartBeat 超时倒计时。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227182128673-1079742991.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227182128673-1079742991.png)

右上角的表格表示各节点的日志，每行表示一个节点。

表格最上面的数字是日志的序号(Log Index)。Log Index 是一个自增且连续的 ID，它可以作为一条日志唯一标识。节点中最大的 Log Index 也反映了这个节点的状态机是否与集群一致。

表格里的单元格表示日志项(Entry)，其中的数字表示提交日志的任期(Term)。虚线框表示日志尚未提交，实线框表示日志已经提交。

我们可以点击 leader 节点的 request 按钮来查看向 Raft 集群提交数据的过程。

# Leader 选举[#](https://www.cnblogs.com/Finley/p/14467602.html#leader-选举)

Raft Scope 启动后会立即进行第一次 Leader 选举，在集群运行过程任何一个 Follower 出现心跳超时都会引发新一轮选举。

我们可以点击任意一个 Follower 的 time out 按钮模拟心跳超时，随后此 Follower 会发起新一轮选举。

或者我们可以点击 Leader 的 stop、restart 来模拟 Leader 宕机或者重启，并观察随后的集群选举过程。

比较奇怪的是， Raft Scope 中的 Leader 节点也可以通过点击 time out 来模拟心跳超时，在实际的 Raft 集群中 Leader 节点通常不会对自己进行心跳检测。

Leader 选举的更多介绍可以查看：[Leader选举](https://acehi.github.io/thesecretlivesofdata-cn/raft/#election)。不过 The Secret Lives of Data 有两处说的可能不太清楚：

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227191855263-2133121887.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227191855263-2133121887.png)

这里的选举超时是指新一轮选举开始时，每个节点随机思考要不要竞选 Leader 的时间，这个时间一般100-到200ms，非常短。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227192255200-1432986206.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227192255200-1432986206.png)

Candidate 发起选举时会将自身任期(Term)+1并向其它所有节点发出 RequestVote 消息，这条消息中包含新任期和 Candidate 节点的最新 Log Index

收到 RequestVote 的节点会进行判断:

```python
def onRequestVote(self, request_vote)
    if request_vote.term <= self.term:
        # 若 RequestVote 中的任期小于或等于(<=)当前任期
        # 则继续 Follow 当前 Leader 并拒绝给 RequestVote投票
        return False
    if request_vote.log_index < self.log_index:
        # 若 request_vote 发送者的 log_index 不如自己新，节点也会拒绝给发送者投票
        # 这种机制确保了已经提交到集群中的日志不会丢失，即保证 Raft 算法的安全性
        return False
    if self.voted_for is None:
        # 若在本 term 中当前节点还未投票，则给 request_vote 的发送者投票
        self.voted_for = request_vote.sender
        return True
    else:
        return False
```

## Follower 超时[#](https://www.cnblogs.com/Finley/p/14467602.html#follower-超时)

现在我们研究一下 The Secret Lives of Data 没有详细说明的 Follower 超时处理过程。

我们可以点击任意一个 Follower 的 time out 按钮模拟心跳超时，随后此 Follower 会发起新一轮选举。

根据上文中的 onRequestVote 逻辑，超时的 Follower 的 Log Index 是否与集群中的大多数节点相同决定了这次选举的不同结果。

首先来看超时 Follower 的 Log Index 与集群中大多数相同的情况：

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227201047099-1062179668.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227201047099-1062179668.png)

现在我们点击 S5 的 time out 按钮，随后我们看到 S5 发起了一轮投票。因为 5 个节点的 Log Index 是一致的, 所以包含原 Leader 在内的大多数节点都投票给了 S5。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227201234556-1686794096.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227201234556-1686794096.png)

现在 S5 成为了新一任 Leader.

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227201411610-1880652878.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227201411610-1880652878.png)

接下来我们看另外一种情况。S5 由于网络问题没有收到带有 Log Entry 1 的心跳包并导致心跳超时，S5 随后会发起一次投票:

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228080935138-449288164.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228080935138-449288164.png)

由于 S5 的 Log Index 比较小其它节点拒绝投票给他，集群 Leader 和任期不变：

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228081359916-2051580778.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228081359916-2051580778.png)

# 日志复制[#](https://www.cnblogs.com/Finley/p/14467602.html#日志复制)

日志复制的介绍您可以查看：[日志复制](https://acehi.github.io/thesecretlivesofdata-cn/raft/#replication)

现在我们进一步探究日志复制的过程:

1. 客户端将更改提交给 Leader, Leader 会在自己的日志中写入一条未提交的记录(Entry)
2. 在下一次心跳时 Leader 会将更改发送给所有 Follower
3. 一旦收到过半节点的确认 Leader 就会提交自己日志中的记录4
4. 并向客户端返回写入成功
5. Leader 会在下一次心跳时通知所有节点提交日志

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228163354456-964717881.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228163354456-964717881.png)

这里比较复杂的情况是在第 4 步完成之后 Leader 崩溃。由于此时客户端已经收到了写入成功的回复，所以在选出新的 Leader 之后要继续完成提交。

在 Leader 提交了自己的日志后我们立即关掉 Leader:

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227232134293-1305699823.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227232134293-1305699823.png)

随后集群发起了一次选举，S3 成为新任 Leader:

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228000212889-451431779.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228000212889-451431779.png)

可能是因为 Raft Scope 存在 Bug, S3 本应该当选后立即完成提交工作。但是实际上需要我们再一次 Request 之后，日志1 和日志 2 才会被一起提交。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228000435789-1163143741.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210228000435789-1163143741.png)

# 脑裂问题[#](https://www.cnblogs.com/Finley/p/14467602.html#脑裂问题)

在 Leader 崩溃时可能会有多个节点近乎同时发现心跳超时并转变为 Candidate 开始选举：

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227202151747-1746630746.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227202151747-1746630746.png)

其它节点投票情况多种多样，但只要保证获**只有得到过半投票的候选人才能成为 Leader**。那么选举结果只有两种可能：

- 有且只有一个候选人获得过半投票成为 Leader 并开始新的任期
- 没有一个候选人获得过半投票，没有选出 Leader 进入下一轮投票

绝对不会选出多个 Leader

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227202528938-191507917.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227202528938-191507917.png)

# 网络分区问题[#](https://www.cnblogs.com/Finley/p/14467602.html#网络分区问题)

Raft 甚至可以在网络分区的情况下正常工作：

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227224809399-974288877.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227224809399-974288877.png)

在发生网络分区后可能存在 3 种情况:

1. 任意分区中的节点数都不超过一半：这种情况只有集群被分成 3 个或更多分区时才会出现，十分罕见。因为 Leader 选举和 Commit Log 都需要超过一半节点确认才可以进行，在这种情况下 Raft 集群不能正常工作。
2. leader 所在的分区有超过一半的节点：这种情况视作其它分区中的 Follower 宕机，系统仍然可以继续工作。在分区修复后，Follower 节点会重新与 Leader 同步。
3. leader 所在分区中节点数不超过一半，但存在节点数超过一半的分区。这种情况最为复杂:

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227224851661-503591283.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227224851661-503591283.png)

C、D、E 所在的分区节点数超过一半且与原来的 Leader 无法通信，随后 C、D、E 在心跳超时后会发起新一轮投票选出新的 Leader 并恢复工作。

原领导者 Node B 仍然会认为自己是集群的 Leader，但是由于只能与两个节点通信(包括自己)无法得到过半节点同意，所以无法完成日志提交。

在分区修复后 Node B 会收到 Node C 的心跳并发现对方的任期（Term）比自己高，Node B 会放弃 Leader 身份转为 Node C 的 Follower 与它保持同步。

[![img](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227230444393-2078111839.png)](https://img2020.cnblogs.com/blog/793413/202102/793413-20210227230444393-2078111839.png)

# 总结[#](https://www.cnblogs.com/Finley/p/14467602.html#总结)

经过本文探讨我们可以总结一下 Raft 的一些特性:

- 只要集群中有超过一半的节点可以正常工作，集群就可以工作
- 只要写入成功的数据就不会再丢失
- 任意节点上保存的状态可能会落后于集群共识但是永远不会出现错误的提交。只要系统仍然在正常工作，节点上的状态一定会在某个时间后与系统共识达成同步，即保证最终一致性
- 只要在某个节点上读到了某个变更, 在此之后这个节点上永远可以读到该变更，即保证单调一致性

推荐阅读:

- [The Secret Lives of Data-CN](https://acehi.github.io/thesecretlivesofdata-cn/raft/)
- [Raft Scope](https://raft.github.io/raftscope/index.html)
- [Raft算法详解](https://zhuanlan.zhihu.com/p/32052223)

raft算法之所以容易理解，其一是他将一致性问题划分成几个子问题，这几个子问题都是独立、可理解和解释的。从传统的思维来讲，对于一个复杂的系统或者工程，都是大化小，分解实现，然后去尝试融合解决整体逻辑。

# 1、Raft 详解

`Raft 算法`是分布式系统开发首选的`共识算法`。比如现在流行 Etcd、Consul、Nacos。

如果`掌握`了这个算法，就可以较容易地处理绝大部分场景的`容错`和`一致性`需求。比如分布式配置系统、分布式 NoSQL 存储等等，轻松突破系统的单机限制。

**Raft 算法是通过一切以领导者为准的方式，实现一系列值的共识和各节点日志的一致。**

## 1.1 Raft 角色

**跟随者（Follower）**：`普通群众`，默默接收和来自领导者的消息，当领导者心跳信息超时的时候，就主动站出来，推荐自己当候选人。

**候选人（Candidate）**：`候选人`将向其他节点请求投票 RPC 消息，通知其他节点来投票，如果赢得了大多数投票选票，就晋升当领导者。

**领导者（Leader）**：`霸道总裁`，一切以我为准。处理写请求、管理日志复制和不断地发送心跳信息，通知其他节点“我是领导者，我还活着，你们不要”发起新的选举，不用找新领导来替代我。

如下图所示，分别用三种图代表跟随者、候选人和领导者。

[![角色](https://oscimg.oschina.net/oscnet/up-974cdd0b755d2e888ce668b7ff7ef1ae.png)](https://oscimg.oschina.net/oscnet/up-974cdd0b755d2e888ce668b7ff7ef1ae.png)

1.1 数据库服务器

现在我们想象一下，有一个单节点系统，这个节点作为数据库服务器，且存储了一个值为 X。

[![数据库服务器](https://oscimg.oschina.net/oscnet/up-38aad9c62187d012699191b6c4bf27c8.png)](https://oscimg.oschina.net/oscnet/up-38aad9c62187d012699191b6c4bf27c8.png)

左边绿色的实心圈就是客户端，右边的蓝色实心圈就是节点 a（Node a）。Term 代表任期，后面会讲到。

[![客户端](https://oscimg.oschina.net/oscnet/up-a4240c944e2de4b105e5d05a80625379.png)](https://oscimg.oschina.net/oscnet/up-a4240c944e2de4b105e5d05a80625379.png)

客户端向单节点服务器发送了一条更新操作，设置数据库中存的值为 8。单机环境下（单个服务器节点），客户端从服务器拿到的值也是 8。一致性非常容易保证。

[![客户端向服务器发送数据](https://oscimg.oschina.net/oscnet/up-8973ab2e54257fe8168265ce85467f3f.gif)](https://oscimg.oschina.net/oscnet/up-8973ab2e54257fe8168265ce85467f3f.gif)

但如果有多个服务器节点，怎么保证一致性呢？比如有三个节点：a，b，c。如下图所示。这三个节点组成一个数据库集群。客户端对这三个节点进行更新操作，如何保证三个节点中存的值一致？这个就是分布式一致性问题。Raft 算法就是来解决这个问题的。当然还有其他协议也可以保证，本篇只针对 Raft 算法。

在多节点集群中，在节点故障、分区错误等异常情况下，Raft 算法如何保证在同一个时间，集群中只有一个领导者呢？下面就开始讲解 Raft 算法选举领导者的过程。

## 1.2 初始状态

初始状态下，集群中所有节点都是跟随者的状态。

如下图所示，有三个节点(Node) a、b、c，任期（Term）都为 0。

[![mark](https://oscimg.oschina.net/oscnet/up-b9350e7f167179a24fa9c6914c0fad6f.png)](https://oscimg.oschina.net/oscnet/up-b9350e7f167179a24fa9c6914c0fad6f.png)

Raft 算法实现了随机超时时间的特性，每个节点等待领导者节点心跳信息的超时时间间隔是随机的。比如 A 节点等待超时的时间间隔 150 ms，B 节点 200 ms，C 节点 300 ms。那么 a 先超时，最先因为没有等到领导者的心跳信息，发生超时。如下图所示，三个节点的超时计时器开始运行。

[![超时时间](https://oscimg.oschina.net/oscnet/up-501031a7051bf405999493e07f1b284f.gif)](https://oscimg.oschina.net/oscnet/up-501031a7051bf405999493e07f1b284f.gif)

## 1.3 发起投票

当 A 节点的超时时间到了后，A 节点成为候选者，并增加自己的任期编号，Term 值从 0 更新为 1，并给自己投了一票。

- Node A：Term = 1, Vote Count = 1。
- Node B：Term = 0。
- Node C：Term = 0。

[![成为候选者](https://oscimg.oschina.net/oscnet/up-c13af5ea7014c4d3a55f926c8b3ebef0.gif)](https://oscimg.oschina.net/oscnet/up-c13af5ea7014c4d3a55f926c8b3ebef0.gif)

## 1.4 成为领导者的简化过程

我们来看下候选者如何成为领导者的。

[![Leader 选举](https://oscimg.oschina.net/oscnet/up-582304792322d1ae79efa9782ca28f5b.gif)](https://oscimg.oschina.net/oscnet/up-582304792322d1ae79efa9782ca28f5b.gif)

- **第一步**：节点 A 成为候选者后，向其他节点发送请求投票 RPC 信息，请它们选举自己为领导者。
- **第二步**：节点 B 和 节点 C 接收到节点 A 发送的请求投票信息后，在编号为 1 的这届任期内，还没有进行过投票，就把选票投给节点 A，并增加自己的任期编号。
- **第三步**：节点 A 收到 3 次投票，得到了大多数节点的投票，从候选者成为本届任期内的新的领导者。
- **第四步**：节点 A 作为领导者，固定的时间间隔给 节点 B 和节点 C 发送心跳信息，告诉节点 B 和 C，我是领导者，组织其他跟随者发起新的选举。
- **第五步**：节点 B 和节点 C 发送响应信息给节点 A，告诉节点 A 我是正常的。

## 1.5 领导者的任期

英文单词是 term，领导者是有任期的。

- **自动增加**：跟随者在等待领导者心跳信息超时后，推荐自己为候选人，会增加自己的任期号，如上图所示，节点 A 任期为 0，推举自己为候选人时，任期编号增加为 1。
- **更新为较大值**：当节点发现自己的任期编号比其他节点小时，会更新到较大的编号值。比如节点 A 的任期为 1，请求投票，投票消息中包含了节点 A 的任期编号，且编号为 1，节点 B 收到消息后，会将自己的任期编号更新为 1。
- **恢复为跟随者**：如果一个候选人或者领导者，发现自己的任期编号比其他节点小，那么它会立即恢复成跟随者状态。这种场景出现在分区错误恢复后，任期为 3 的领导者受到任期编号为 4 的心跳消息，那么前者将立即恢复成跟随者状态。
- **拒绝消息**：如果一个节点接收到较小的任期编号值的请求，那么它会直接拒绝这个请求，比如任期编号为 6 的节点 A，收到任期编号为 5 的节点 B 的请求投票 RPC 消息，那么节点 A 会拒绝这个消息。
- 一个任期内，领导者一直都会领导者，直到自身出现问题（如宕机），或者网络问题（延迟），其他节点发起一轮新的选举。
- 在一次选举中，每一个服务器节点最多会对一个任期编号投出一张选票，投完了就没了。

假设一个集群由 N 个节点组成，那么大多数就是至少 N/2+1。例如： 3 个节点的集群，大多数就是 2。

## 1.6 防止多个节点同时发起投票

为了防止多个节点同时发起投票，会给每个节点分配一个随机的选举超时时间。这个时间内，节点不能成为候选者，只能等到超时。比如上述例子，节点 A 先超时，先成为了候选者。这种巧妙的设计，在大多数情况下只有一个服务器节点先发起选举，而不是同时发起选举，减少了因选票瓜分导致选举失败的情况。

[![成为候选者](https://oscimg.oschina.net/oscnet/up-f74d19cd8abf89e246e901938ffccaa7.gif)](https://oscimg.oschina.net/oscnet/up-f74d19cd8abf89e246e901938ffccaa7.gif)

## 1.7 触发新的一轮选举

如果领导者节点出现故障，则会触发新的一轮选举。如下图所示，领导者节点 B 发生故障，节点 A 和 节点 B 就会重新选举 Leader。

[![领导者故障](https://oscimg.oschina.net/oscnet/up-8abd1b53818ea03d5afdf6dc362548e1.gif)](https://oscimg.oschina.net/oscnet/up-8abd1b53818ea03d5afdf6dc362548e1.gif)

- 第一步 ：节点 A 发生故障，节点 B 和节点 C 没有收到领导者节点 A 的心跳信息，等待超时。
- 第二步：节点 C 先发生超时，节点 C 成为候选人。
- 第三步：节点 C 向节点 A 和 节点 B 发起请求投票信息。
- 第四步：节点 C 响应投票，将票投给了 C，而节点 A 因为发生故障了，无法响应 C 的投票请求。
- 第五步：节点 C 收到两票（大多数票数），成为领导者。
- 第六步：节点 C 向节点 A 和 B 发送心跳信息，节点 B 响应心跳信息，节点 A 不响应心跳信息。

## 1.8 Raft 算法的几个关键机制

Raft 算法通过以下几个关键机制，保证了一个任期只有一位领导，极大减少了选举失败的情况。

- 任期
- 领导者心跳信息
- 随机选举超时时间
- 先来先服务的投票原则
- 大多数选票原则

# 2. Raft算法的典型应用

Raft算法的典型应用包括：

- 领导选举
- 日志复制

对于一个集群只有一个leader（领导），那么我们就很容易理解。只要领导操作同步到对应的followers（跟随者），数据必然一致。当leader宕机，需要进行领导选举。

> 日志复制其实就是同步操作数据的过程。leader将操作日志同步到其他节点。

安全性：如何安全的同步，在不同的情况，我们都能保证一致性，这也就是安全性需要考虑的问题。

> 其实就是如此，raft首先假设了领导选举。然后实现了日志复制，最后在安全问题上解决上面的漏洞问题。

### 1.领导选举

目的：当集群初始化或者领导gg的时候选出一个新的领导。毕竟一个集群不能没有领导，如果没有，那么这个集群就不可用了。

触发机制：通过心跳。
[![1.png](https://segmentfault.com/img/bVbFEoJ)](https://segmentfault.com/img/bVbFEoJ)
这幅图展现了跟随者、候选者和领导者之间的状态转换关系，下面主要介绍他们的转换流程。

**跟随者**

如果他能持续从领导者或者候选者接收到有效的RPCs，那么他的状态就不会变。一直保持跟随者身份。但如果没有持续接受，也就是在一段时间没收到有效的RPCs，那就**选举超时**，他会变成候选者。

**候选者**

要变为候选者，也就意味着他要开启一轮新的选举。那么跟随者会增加自己的任期号，转为候选者。

成为候选者后，自己会并行发送一个为自己投票的RPCs请求 给其他服务器。

成为候选者的整个过程也会保持当前状态，知道满足下面三个条件

- 他赢得选举，转变为领导制
- 其他节点赢了，他转为跟随者
- 一段时间没有任何人获胜。说明选票被瓜分，重复执行。

**这里需要注意的点：**

1.对于同一任期号，每个节点一会投一张票。比如服务器A作为候选者生成了编号为5的任期号，那么如果接收到其他节点的编号为5的任期号的投票请求，他会忽略。这个过程遵循的是先来先投的原则。
2.候选者接收到领导者的声明。会判声明中RPC的任期号，如果比自己的还小，那么他还会保持候选者身份，否则转为跟随者。
3.对于上面第三点，重复执行，Raft采用随机超时选举时间进行了优化。降低选票瓜分的可能性。

### 2.Raft日志复制

直接去理解日志复制，是很容易的，客户端的一条指令到达，领导者会为这条指令创建一条日志条目，并且并行发送到其他跟随者。当日志被安全复制（所谓安全复制后面会有），领导会将日志应用到状态机（比如如果是mysql的insert，那么就是执行insert操作），然后响应客户端。

[![2.png](https://segmentfault.com/img/bVbFEo7)](https://segmentfault.com/img/bVbFEo7)

如上图，每条日志都会有对应的任期号，和指令。
每个日志都会有对应的索引。

**raft日志匹配特性**
1.如果在不同的日志中的两个条目拥有相同的索引和任期号，那么他们存储了相同的指令。
2.如果在不同的日志中的两个条目拥有相同的索引和任期号，那么他们之前的所有日志条目也全部相同。

第一点：一个任期只有一个领导人，并且领导人在一个任期中对于同一索引日志，只会创建一条日志，是不会改变的，是确定的。这就保证第一点成立。
第二点：要想全部相同，就要保证跟随者得到的日志是领导者发送的顺序附加上去的。领导者在发送新的日志时，会附加这条日志之前日志的索引和任期号。如果跟随者发现数据匹配，才会附加上去，否则拒绝。就是一个个状态保证了日志的匹配特性。

> 对于日志不一致的现象，raft是通过跟随者强制复制领导者的日志来保证的。

[![3.png](https://segmentfault.com/img/bVbFEpD)](https://segmentfault.com/img/bVbFEpD)

如上图，对于a-f，最终都会和leader同步，也就是说，d会丢弃日志。f的对应日志也会被丢弃和覆盖。

> 其实就是通过日志覆盖解决。但是对于日志覆盖，我们就会想到一个问题，会不会覆盖已经提交的日志（日志对应指令已经返回给客户端）。那当然不会，如果真有这样，就会有不一致，或者指令丢失现象。

**那么如何去做覆盖跟随者日志**

其实就是跟随者在append日志的时候，会进行错误校验。

在候选者成为领导者的时候，会为每个跟随者初始化一个nextIndex数组，数组的值初始化为自己最后条日志+1，其实就是理想化状态，默认认为日志都已经同步成功。但是理想总会因为各种原因导致不正确，就用上面那张图。leader会初始化所有nextIndex为11，但是在同步日志的过程中。f节点会出现校验错误的响应。因为f节点10索引对应的日志和leader10索引对应的日志不相同（主要是根据任期号判断）

这里再强调一下为什么根据任期号就可以判断日志是否一致，就是上面所说的日**志匹配原则。**

# 3.Raft算法的安全性

我们明白了如何选举和日志复制，但是没有考虑安全性问题。其实我上慢提到，比如一个宕机很久的跟随着会被选为领导者，进行日志覆盖操作会有丢失问题。

> 其实解决这个办法很简单，就是在领导选举的时候，只能让安全的节点当leader，所谓安全，就是对应节点拥有当前领导者已经提交的所有日志。Raft就是这么做的。

Raft中节点在投票的时候，会判断被投票的候选者对应的日志是否至少和自己一样新。如果不是，则不会给该候选者投票。

日志比较的方法：
1.最后一条日志的任期号。如果大说明新。如果小，说明不新。如果相等。跳到2
2.判断索引长度。大的更新。

**还有一个问题，就是领导人不能保证一个已经在大多数节点存在的日志是否已经提交。**

[![4.png](https://segmentfault.com/img/bVbFEpG)](https://segmentfault.com/img/bVbFEpG)

a、b、c、d、e代表不同的任期阶段

（a）S1是leader。同步任期2的数据给S2

（b）S1宕机，S5当选（S3、S4、S5投票），产生任期3的日志

（c）S5宕机，S1恢复当选（同步任期2的数据给S3）。

（d）S1宕机，S5当选（因为他的任期日志比其他的都新），复制了任期3的所有数据。

假如说在c阶段，S1提交了任期2的数据，那么如果出现d，则会导致任期2数据被覆盖，丢失。也就是说，S1在任期4时候，不能保证已经在大多数节点存在的日志（任期2的日志）是否提交。

> 所以raft永远不会通过计算副本数目的方式（大多数存在）去提交一个之前任期内（任期2）的日志条目。只有领导人当前任期里的日志条目通过计算副本数目可以被提交（e阶段）。这样之前任期的数据也会被提交。

**那这里我理解一点就是，加入S1当选为leader，如图c状态，那么，如果不再有新的日志出现，任期2对应的日志就不会提交。那么会导致客户端对应的任期2请求失败。**

### 跟随者和候选人宕机

这个就比较容易理解了，宕机的话RPCs就会失败，Raft通过无限重试却解决这个问题。
所以对于每个RPCs，做到幂等和无限重试，在节点恢复后，就还是会保证一致性状态。

### Raft集群成员变化

对于集群成员配置变化，如果直接更新每台机器配置，那么就会有安全性问题。以为对于同一时刻，不同节点使用的不同的配置去执行算法逻辑，这就是不安全的。

[![5.png](https://segmentfault.com/img/bVbFEpK)](https://segmentfault.com/img/bVbFEpK)

> 如图，蓝色代表新的配置。绿色代表老的配置。Old状态有三台机器Server1、2、3。 New加入两台server4、5。

那么随着配置时间应用的不同，可能会导致选举出两个leader。
比如server1、2使用老配置，那么1和2都有可能被当选为leader。3、4、5使用心得配置，他们之中的一个也会被当选为leader。

其实这个问题原因就是节点使用了不同配置执行算法逻辑。为了解决这个问题，raft采用两阶段方法（其实只需要保证不会让新或者旧配置单独作出决定就行）

[![6.png](https://segmentfault.com/img/bVbFEpO)](https://segmentfault.com/img/bVbFEpO)

**raft吧配置当作普通日志形式去提交。**

为了实现两阶段，引入了C(old、new)配置。
还有一点就是，一点一个新的配置日志增加到对应的节点日志中，那么该节点就会立刻使用这条新的日志配置。

> 对于C（old、new）配置，其实就是只有同时满足old和new配置的时候才会生效。

这样理想状态下，如果拥有C（old、new）配置的节点当选为leader。并且提交了该配置，那么说明C（old、new）配置已经在大多数节点应用。下次选举的产生的leader日志中必然会有该配置。这个时候在创建一条新的C（new）配置提交，即可。

# ZAB和Raft算法

ZAB中，leader将需要执行的命令发送到各个follower（步骤1），follower收到后给leader一个ACK消息（步骤2）。leader收到超过半数的ACK消息后，leader执行命令（步骤3），leader并向follower发送commit操作（步骤4）。
Raft协议是，leader向follower发送执行log（步骤1），超过半数follower收到log并保存后返回ACK后（步骤2），leade执行完日志（步骤3）后即可向客户端响应。
对比来看，Raft协议不需要等待follower执行完log，只保证半数的follower接收日志。所以在Raft协议里，所有请求都全部由leader处理。对于执行log可以类比Mysql的redo log。

> ### Raft共识算法

### 一.背景

拜占庭将军问题是分布式领域**最复杂、最严格的容错模型**。但在日常工作中使用的分布式系统面对的问题不会那么复杂，更多的是计算机故障挂掉了，或者网络通信问题而没法传递信息，这种情况**不考虑计算机之间互相发送恶意信息**，极大简化了系统对容错的要求，最主要的是达到一致性。

所以将拜占庭将军问题根据常见的工作上的问题进行简化：**假设将军中没有叛军，信使的信息可靠但有可能被暗杀的情况下，将军们如何达成一致性决定？**

对于这个简化后的问题，有许多解决方案，第一个被证明的共识算法是 Paxos，由拜占庭将军问题的作者 Leslie Lamport 在1990年提出，但因为 Paxos 难懂，难实现，所以斯坦福大学的教授Diego Ongaro 和 John Ousterhout在2014年发表了论文**《In Search of an Understandable Consensus Algorithm》**，其中提到了新的分布式协议 `Raft`。与 Paxos 相比，Raft 有着基本相同运行效率，但是更容易理解，也更容易被用在系统开发上。

------

### 二.概述

Raft实现一致性的机制是这样的：首先选择一个leader全权负责管理日志复制，leader从客户端接收log entries（日志条目），将它们复制给集群中的其它机器，然后负责告诉其它机器什么时候将日志应用于它们的状态机。举个例子，leader可以在无需询问其它server的情况下决定把新entries放在哪个位置，数据永远是从leader流向其它机器（leader的强一致性）。一个leader可以fail或者与其他机器失去连接，这种情形下会有新的leader被选举出来。

在任何时刻，每个server节点有三种状态：`leader`，`candidate`，`follower`。

- leader：作为客户端的接收者，接收客户端发送的日志复制请求，并将日志信息复制到 follower 节点中，维持网络各个节点的账本状态。
- candidate：在leader 选举阶段存在的状态，通过任期号term和票数进行领导人身份竞争，获胜者将成为下一任期的领导人。
- follower：作为leader 节点发送日志复制请求的接收者，与leader节点通信，接收账本信息，并确认账本信息的有效性，完成日志信息的提交和存储。

正常运行时，只有一个leader，其余全是follower。follower是被动的：它们不主动提出请求，只是响应leader和candidate的请求。leader负责处理所有客户端请求（如果客户端先连接某个follower，该follower要负责把它重定向到leader）。candidate状态用于选举领导节点。下图展示了这些状态以及它们之间的转化:

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314220524044-1841690773.png)

Raft将时间分解成任意长度的`terms`，如下图所示：

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314220535693-1767695662.png)

terms有连续单调递增的编号，每个term开始于选举，这一阶段每个candidate都试图成为leader。如果一个candidate选举成功，它就在该term剩余周期内履行leader职责。在某种情形下，可能出现选票分散，没有选出leader的情况，这时新的term立即开始。**Raft确保在任何term都只可能存在一个leader**。term在Raft用作逻辑时钟，servers可以利用term判断一些过时的信息：比如过时的leader。每台server都存储当前term号，它随时间单调递增。term号可以在任何server通信时改变：如果某个server节点的当前term号小于其它servers，那么这台server必须更新它的term号，保持一致；如果一个candidate或者leader发现自己的term过期，则降级成follower；如果某个server节点收到一个过时的请求（拥有过时的term号），它会拒绝该请求。

Raft servers使用RPC交互，基本的一致性算法只需要两种RPC。`RequestVote RPCs`由candidate在选举阶段发起。`AppendEntries RPCs`在leader复制数据时发起，leader在和follower做心跳时也用该RPC。servers发起一个RPC，如果没得到响应，则需要不断重试。另外，发起RPC是并行的。

------

### 三.具体共识流程

raft算法大致可以划分为两个阶段，即`Leader Selection`和`Log Relocation`，同时使用强一致性来减少需要考虑的状态。

#### 3.1 Leader Selection

Raft使用`heartbeat`（心跳机制）来触发选举。当server节点启动时，初始状态都是follower。每一个server都有一个定时器，超时时间为`election timeout`（**时间长度一般为150ms~300ms**），如果某server没有超时的情况下收到来自leader或者candidate的任何RPC，则定时器**重启**，如果超时，它就开始一次选举。leader给followers发RPC要么复制日志，要么就是用来告诉followers自己是leader，不用选举的心跳（告诉followers对状态机应用日志的消息夹杂在心跳中）。如果某个candidate获得了**超过半数**节点的选票（自己投了自己），它就赢得了选举成为新leader。

上述的具体过程如下：

➢ 初始状态下集群中的所有节点都处于 follower 状态。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314220617194-1813732136.png)

➢ 某一时刻，其中的一个 follower 由于没有收到 leader 的 heartbeat 率先发生 election timeout 进而发起选举。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221034628-908216094.png)

➢ 只要集群中超过半数的节点接受投票，candidate 节点将成为即切换 leader 状态。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221133536-155588085.png)

➢ 成为 leader 节点之后，leader 将定时向 follower 节点同步日志并发送 heartbeat。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221226332-2116574232.png)

**如果leader节点出现了故障，那怎么办？**

下面将说明当集群中的 leader 节点不可用时，raft 集群是如何应对的。

➢ 一般情况下，leader 节点定时发送 heartbeat 到 follower 节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221306319-1624406643.png)

➢ 由于某些异常导致 leader 不再发送 heartbeat ，或 follower 无法收到 heartbeat 。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221422239-219883405.png)

➢ 当某一 follower 发生election timeout 时，其状态变更为 candidate，并向其他 follower 发起投票。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221442435-917603935.png)

➢ 当超过半数的 follower 接受投票后，这一节点将成为新的 leader，leader 的任期号term加1并开始向 follower 同步日志。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221515727-1608774094.png)

➢ 当一段时间之后，如果之前的 leader 再次加入集群，则两个 leader 比较彼此的任期号，任期号低的leader将切换自己的状态为follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221529947-1104329632.png)

➢ 较早前 leader 中不一致的日志将被清除，并与现有 leader 中的日志保持一致。
![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221625602-1322176874.png)

还有第三种可能性就是candidate既没选举成功也没选举失败：如果多个follower同时成为candidate去拉选票，导致选票分散，任何candidate都没拿到大多数选票，这种情况下Raft使用超时机制`election timeout`来解决。所以同时出现多个candidate的可能性不大，即使机缘巧合同时出现了多个candidate导致选票分散，那么它们就等待自己的election timeout超时，重新开始一次新选举，实验也证明这个机制在选举过程中收敛速度很快。

#### 3.2 Log Relocation

在 raft 集群中，所有日志都必须首先提交至 leader 节点。leader 在每个 heartbeat 向 follower 发送AppendEntries RPC同步日志，follower如果发现没问题，复制成功后会给leader一个表示成功的ACK，leader收到超过半数的ACK后应用该日志，返回客户端执行结果。若 follower 节点宕机、运行缓慢或者丢包，则 leader 节点会不断重试AppendEntries RPC，直到所有 follower 节点最终都复制所有日志条目。

上述的具体过程如下：

➢ 首先有一条 uncommitted 的日志条目提交至 leader 节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091224975-1159667758.png)

➢ 在下一个 heartbeat，leader 将此条目复制给所有的 follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091241400-1910380809.png)

➢ 当大多数节点记录此条目之后，leader 节点认定此条目有效，将此条目设定为已提交并存储于本地磁盘。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091259300-469736576.png)

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091305693-1149042857.png)

➢ 在下一个 heartbeat，leader 通知所有 follower 提交这一日志条目并存储于各自的磁盘内。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091337576-1642812043.png)

**Network Partition 情况下进行复制日志:**

由于网络的隔断，造成集群中多数的节点在一段时间内无法访问到 leader 节点。按照 raft 共识算法，没有 leader 的那一组集群将会通过选举投票出新的 leader，甚至会在两个集群内产生不一致的日志条目。在集群重新完整连通之后，原来的 leader 仍会按照 raft 共识算法从步进数更高的 leader 同步日志并将自己切换为 follower。

➢ 集群的理想状态。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091525429-149847836.png)

➢ 网络间隔造成大多数的节点无法访问 leader 节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091610822-1817228267.png)

➢ 新的日志条目添加到 leader 中。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091636052-1248453194.png)

➢ leader 节点将此条日志同步至能够访问到 leader 的节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091700355-638880697.png)

➢ follower 确认日志被记录，但是确认记录日志的 follower 数量没有超过集群节点的半数，leader 节点并不将此条日志存档。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091721168-1485930780.png)

➢ 在被隔断的这部分节点，在 election timeout 之后，followers 中产生 candidate 并发起选举。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091731090-569718079.png)

➢ 多数节点接受投票之后，candidate 成为 leader。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092036233-1288529334.png)

➢ 一个日志条目被添加到新的 leader并复制给新 leader 的 follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092133129-1417768300.png)

➢ 多数节点确认之后，leader 将日志条目提交并存储。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092256566-1625387423.png)

➢ 在下一个 heartbeat，leader 通知 follower 各自提交并保存在本地磁盘。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092304566-153284042.png)

➢ 经过一段时间之后，集群重新连通到一起，集群中出现两个 leader 并且存在不一致的日志条目。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092348915-2020717248.png)

➢ 新的 leader 在下一次 heartbeat timeout 时向所有的节点发送一次 heartbeat。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092400133-1355454482.png)

➢ #1 leader 在收到任期号term更高的 #2 leader heartbeat 时放弃 leader 地位并切换到 follower 状态。
![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092431973-1040440738.png)

➢ 此时leader同步未被复制的日志条目给所有的 follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092530511-1595066180.png)

通过这种方式，只要集群中有效连接的节点超过总数的一半，集群将一直以这种规则运行下去并始终确保各个节点中的数据始终一致。

------

### 参考

[1] https://www.jianshu.com/p/8e4bbe7e276c

[2] Ongaro D, Ousterhout J. In search of an understandable consensus algorithm［C］// USENIX Annual Technical Conference. [s.l.]: USENIX. 2014: 305-319．

[3] https://www.cnblogs.com/aibabel/p/10973585.html

[4] Raft原理动画：http://thesecretlivesofdata.com/raft/

整理制作了一份golang的面试题系列，已上传GitHub，方便大家查看，欢迎star~。地址：https://github.com/zmk-c/GolangGuide

分类: [Hyperledger Fabric](https://www.cnblogs.com/zmk-c/category/1942949.html)

# [Raft](https://www.cnblogs.com/linguoguo/p/14387967.html)

# Raft是consoul和etcd的核心算法

 

### 1.1.1. Raft介绍

- Raft提供了一种在计算系统集群中分布状态机的通用方法，确保集群中的每个节点都同意一系列相同的状态转换
- 它有许多开源参考实现，具有Go，C ++，Java和Scala中的完整规范实现
- 一个Raft集群包含若干个服务器节点，通常是5个，这允许整个系统容忍2个节点的失效，每个节点处于以下三种状态之一
  - follower（跟随者） ：所有节点都以 follower 的状态开始。如果没收到 leader消息则会变成 candidate状态
  - candidate（候选人）：会向其他节点“拉选票”，如果得到大部分的票则成为leader，这个过程就叫做Leader选举(Leader Election)
  - leader（领导者）：所有对系统的修改都会先经过leader

### 1.1.2. Raft一致性算法

- Raft通过选出一个leader来简化日志副本的管理，例如，日志项(log entry)只允许从leader流向follower
- 基于leader的方法，Raft算法可以分解成三个子问题
  - Leader election (领导选举)：原来的leader挂掉后，必须选出一个新的leader
  - Log replication (日志复制)：leader从客户端接收日志，并复制到整个集群中
  - Safety (安全性)：如果有任意的server将日志项回放到状态机中了，那么其他的server只会回放相同的日志项

### 1.1.3. Raft动画演示

- 地址：http://thesecretlivesofdata.com/raft/
- 动画主要包含三部分：
  - 第一部分介绍简单版的领导者选举和日志复制的过程
  - 第二部分介绍详细版的领导者选举和日志复制的过程
  - 第三部分介绍如果遇到网络分区（脑裂），raft算法是如何恢复网络一致的

### 1.1.4. Leader election (领导选举)

- Raft 使用一种心跳机制来触发领导人选举
- 当服务器程序启动时，节点都是 follower(跟随者) 身份
- 如果一个跟随者在一段时间里没有接收到任何消息，也就是选举超时，然后他就会认为系统中没有可用的领导者然后开始进行选举以选出新的领导者
- 要开始一次选举过程，follower 会给当前term加1并且转换成candidate状态，然后它会并行的向集群中的其他服务器节点发送请求投票的 RPCs 来给自己投票。
- 候选人的状态维持直到发生以下任何一个条件发生的时候
  - 他自己赢得了这次的选举
  - 其他的服务器成为领导者
  - 一段时间之后没有任何一个获胜的人

### 1.1.5. Log replication (日志复制)

- 当选出 leader 后，它会开始接收客户端请求，每个请求会带有一个指令，可以被回放到状态机中
- leader 把指令追加成一个log entry，然后通过AppendEntries RPC并行地发送给其他的server，当该entry被多数server复制后，leader 会把该entry回放到状态机中，然后把结果返回给客户端
- 当 follower 宕机或者运行较慢时，leader 会无限地重发AppendEntries给这些follower，直到所有的follower都复制了该log entry
- raft的log replication要保证如果两个log entry有相同的index和term，那么它们存储相同的指令
- leader在一个特定的term和index下，只会创建一个log entry