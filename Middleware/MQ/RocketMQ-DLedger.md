DLedger——基于 Raft 的 Commitlog 存储 Library
https://github.com/openmessaging/openmessaging-storage-dledger

 故事的起源
自分布式系统诞生以来，容灾和一致性，一直是经常被讨论的话题。
Master-Slave 架构是最容易被想到的设计，简单而易于实现，被早期大部分分布式系统采用，包括 RocketMQ 早期的高可用架构，其略显粗陋的一致性保证、缺少自动 Failover 等，并不能满足需求。
后来，Hadoop 迅猛发展改变了这一面貌。Hadoop 生态里面的 Zookeeper 组件，可以作为一个高可用的锁而存在，由此引发了大量系统通过 Zookeeper 选主，然后主备复制日志，来达到高可用和一致性的目的。Hadoop 自身  NameNode 组件 的高可用机制便是这一典型实现。

基于 ZooKeeper 的设计，通过一些复杂的写入 fence，基本可以满足需求。但 Zookeeper 自身的复杂性，加重了整个设计，在具体实施和运维时，不仅增加资源成本，还累积了系统风险，让维护人员叫苦不堪。

再后来，Raft 论文的出现，再一次改变了局面。其简洁易懂的设计，没有任何外部依赖，就可以轻松搞定一个高可靠、高可用、强一致的数据复制系统，让广大分布式系统研发人员如获至宝。

本文的主角 DLedger 就是这样的一个实践者。

 DLedger 的定位
DLedger 定位是一个工业级的 Java Library，可以友好地嵌入各类 Java 系统中，满足其高可用、高可靠、强一致的需求。
和这一定位比较接近的是  Ratis。
Ratis 是一个典型的"日志 + 状态机"的实现，虽然其状态机可以自定义，却仍然不满足消息领域的需求。 在消息领域，如果根据日志再去构建“消息状态机”，就会产生 Double IO 的问题，造成极大的资源浪费，因此，在消息领域，是不需要状态机的，日志和消息应该是合二为一。
相比于 Ratis，DLedger 只提供日志的实现，只拥有日志写入和读出的接口，且对顺序读出和随机读出做了优化，充分适应消息系统消峰填谷的需求。
DLedger 的纯粹日志写入和读出，使其精简而健壮，总代码不超过4000行，测试覆盖率高达70%。而且这种原子化的设计，使其不仅可以充分适应消息系统，也可以基于这些日志去构建自己的状态机，从而适应更广泛的场景。
综上所述，DLedger 是一个基于 Raft 实现的、高可靠、高可用、强一致的 Commitlog 存储 Library。

 DLedger 的实现
DLedger 的实现大体可以分为以下两个部分： 1.选举 Leader 2.复制日志 其整体架构如下图 

本文不展开讨论实现的细节，详情可以参考论文[1]。有兴趣的也可以直接参看源码，项目总体不超过4000行代码，简洁易读。

 DLedger 的应用案例
在 Apache RocketMQ 中，DLedger 不仅被直接用来当做消息存储，也被用来实现一个嵌入式的 KV 系统，以存储元数据信息。

更多的接入案例，敬请期待。

 案例1 DLedger 作为 RocketMQ 的消息存储
架构如下图所示： 其中：

DLedgerCommitlog 用来代替现有的 Commitlog 存储实际消息内容，它通过包装一个 DLedgerServer 来实现复制；

依靠 DLedger 的直接存取日志的特点，消费消息时，直接从 DLedger 读取日志内容作为消息返回给客户端；

依靠 DLedger 的 Raft 选举功能，通过 RoleChangeHandler 把角色变更透传给 RocketMQ 的Broker，从而达到主备自动切换的目标

 案例2 利用 DLedger 实现一个高可用的嵌入式 KV 存储
架构图如下所示： 

其中：

DLedger 用来存储 KV 的增删改日志；

通过将日志一条条 Apply 到本地 Map，比如 HashMap 或者 第三方 的 RocksDB等

整个系统的高可用、高可靠、强一致通过 DLedger 来实现。

 社区发展计划
目前 DLedger 已经成为  OpenMessaging 中存储标准的默认实现。DLedger 会维持自身定位不变，作为一个精简的 Commitlog 存储 Library，后续主要是做性能优化和一些必要的特性补充，比如支持手工配置 Leader 节点，支持蜕化成 Master-Slave 架构等。 基于 DLedger 的开发，也可以作为独立项目进行孵化，比如  OpenMessaging KV。
欢迎社区的朋友们一起来共建。

优先 Leader
 目标
优先选举某个节点为 Leader，以控制负载。 假设某个组中包含 A B C 三个节点，指定 A 为优先节点，则期望以下自动行为：

如果 A 节点在线，且与其它节点的日志高度差别不大时，优先选举 A 为 Leader

如果 A 节点掉线后又上线，且日志高度追到与当前 Leader 较为接近时，当前 Leader 节点要尝试让出 Leader 角色给 A 节点

 方案
优先Leader的整体方案依赖抢主（Take Leadership）和主转让(Leadership Transfer)两个子操作。

 抢主 Take Leadership
Take Leadership 是指当前节点在想要主动成为Leader角色时，执行的抢主操作。

如果当前节点是Follower, 将term+1，转为Candidate，进入抢主状态。如果已经是Candidate，也将term+1，进入抢主状态，具备如下特性：

在日志高度一样时，不会为同一个term的其他Candidate投票

将会以更小的重试间隔（voteInterval）进行选主。

状态只维持一个term的选举。

其他操作同普通的选主过程

抢主操作并不保证改节点一定能成为主。

 主转让 Leadership Transfer
指定节点转让 Leader 角色是优先 Leader 的一个依赖功能，同时也可以作为独立的功能存在，实现手动的负载均衡。下面是描述其实现方案：

首先 Leader 节点（假设为A节点）提供接口，接收将当前的Leadership 转让给 B 节点。

A 节点收到转让命令之后，为了避免过长时间的不可用，先检查B节点的落后的数据量 是否小于 配置的阈值（maxLeadershipTransferWaitIndex)，超了则返回转换失败，结束任务。

如果B节点进度落后不多，则标记 A 节点状态为 "切换"，拒绝接收新的数据写入。

A 节点检查B节点的日志数据是否已经是最新，如果还缺数据，则 A 节点继续往 B 节点推送数据。

当 B 节点的数据同步完成之后，A 节点往 B 节点发送命令，让其进行 Take Leadership 操作，争抢主节点。

 Preferred Leader
在 Preferred 节点上，选举的过程类似“抢主”的逻辑，但是不提升Term，避免打断集群。只有收到明确的“抢主”命令时才提升Term。 在 Leader 上，轮询检查，优先节点是否为主，如果不是主并且该节点健康，就发起 Leadership Transfer 操作, 尝试将主节点转让给该优先节点。

快速开始
 条件
64位JDK 1.8+;

Maven 3.2.x.

 建立
mvn clean install -DskipTests
1.
 运行命令行
## Get Command Usage
java -jar target/DLedger.jar

## Start DLedger Server
nohup java -jar target/DLedger.jar server &

## Append Data to DLedger
java -jar target/DLedger.jar append -d "Hello World"

## Get Data from DLedger
java -jar target/DLedger.jar get -i 0
1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.

-----------------------------------
DLedger——基于 Raft 的 Commitlog 存储 Library
https://blog.51cto.com/u_14299346/2382896