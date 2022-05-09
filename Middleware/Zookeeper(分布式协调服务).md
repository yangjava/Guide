# Zookeeper

[TOC]

## 概述

### 简介

**Apache ZooKeeper is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination.**

官网介绍：**Apache ZooKeeper 致力于开发和维护一个开源服务器，以高可靠的分布式协调服务。**

ApacheZooKeeper是 Apache 软件基金会的一个软件项目，它为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。ZooKeeper 曾经是 Hadoop 的一个子项目，但现在是一个独立的顶级项目。

Zookeeper是根据谷歌的论文《The Chubby Lock Service for Loosely Couple Distribute System 》所做的开源实现

ZooKeeper 是 Apache 的顶级项目。ZooKeeper 为分布式应用提供了高效且可靠的分布式协调服务，提供了诸如统一命名服务、配置管理和分布式锁等分布式的基础服务。在解决分布式数据一致性方面，ZooKeeper 并没有直接采用 Paxos 算法，而是采用了名为 ZAB 的一致性协议。

ZooKeeper 主要用来解决分布式集群中应用系统的一致性问题，它能提供基于类似于文件系统的目录节点树方式的数据存储。但是 ZooKeeper 并不是用来专门存储数据的，它的作用主要是用来维护和监控存储数据的状态变化。通过监控这些数据状态的变化，从而可以达到基于数据的集群管理。

很多大名鼎鼎的框架都基于 ZooKeeper 来实现分布式高可用，如：Dubbo、Kafka 等。

#### 参考资料

Apache ZooKeeper 官网：
  https://zookeeper.apache.org/

Apache ZooKeeper r3.7.0原版文档：
  https://zookeeper.apache.org/doc/r3.7.0/index.html

Apache ZooKeeper r3.5.6中文文档：
  https://www.docs4dev.com/docs/zh/zookeeper/r3.5.6/reference/

### 由来

Zookeeper最早起源于雅虎研究院的一个研究小组。在当时，研究人员发现，在雅虎内部很多大型系统基本都需要依赖一个类似的系统来进行分布式协调，但是这些系统往往都存在分布式单点问题。所以，雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架，以便让开发人员将精力集中在处理业务逻辑上。

关于“ZooKeeper”这个项目的名字，其实也有一段趣闻。在立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的Pig项目),雅虎的工程师希望给这个项目也取一个动物的名字。时任研究院的首席科学家RaghuRamakrishnan开玩笑地说：“在这样下去，我们这儿就变成动物园了！”此话一出，大家纷纷表示就叫动物园管理员吧一一一因为各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了，而Zookeeper正好要用来进行分布式环境的协调一一于是，Zookeeper的名字也就由此诞生了。

顾名思义 zookeeper 就是动物园管理员，他是用来管 hadoop（大象）、Hive(蜜蜂)、pig(小猪)的管理员， Apache Hbase 和 Apache Solr 的分布式集群都用到了 zookeeper；Zookeeper:是一个分布式的、开源的程序协调服务，是 hadoop 项目下的一个子项目。他提供的主要功能包括：配置管理、名字服务、分布式锁、集群管理。

### 作用

#### 配置管理

在我们的应用中除了代码外，还有一些就是各种配置。比如数据库连接等。一般我们都是使用配置文件的方式，在代码中引入这些配置文件。当我们只有一种配置，只有一台服务器，并且不经常修改的时候，使用配置文件是一个很好的做法，但是如果我们配置非常多，有很多服务器都需要这个配置，这时使用配置文件就不是个好主意了。这个时候往往需要寻找一种集中管理配置的方法，我们在这个集中的地方修改了配置，所有对这个配置感兴趣的
都可以获得变更。Zookeeper 就是这种服务，它使用 Zab 这种一致性协议来提供一致性。现在有很多开源项目使用 Zookeeper 来维护配置，比如在 HBase 中，客户端就是连接一个Zookeeper，获得必要的 HBase 集群的配置信息，然后才可以进一步操作。还有在开源的消息队列 Kafka 中，也使用 Zookeeper 来维护 broker 的信息。在 Alibaba 开源的 SOA 框架 Dubbo中也广泛的使用 Zookeeper 管理一些配置来实现服务治理。

#### 名字服务

名字服务这个就很好理解了。比如为了通过网络访问一个系统，我们得知道对方的 IP地址，但是 IP 地址对人非常不友好，这个时候我们就需要使用域名来访问。但是计算机是不能是域名的。怎么办呢？如果我们每台机器里都备有一份域名到 IP 地址的映射，这个倒是能解决一部分问题，但是如果域名对应的 IP 发生变化了又该怎么办呢？于是我们有了DNS 这个东西。我们只需要访问一个大家熟知的(known)的点，它就会告诉你这个域名对应的 IP 是什么。在我们的应用中也会存在很多这类问题，特别是在我们的服务特别多的时候，如果我们在本地保存服务的地址的时候将非常不方便，但是如果我们只需要访问一个大家都熟知的访问点，这里提供统一的入口，那么维护起来将方便得多了。

#### 分布式锁

Zookeeper 是一个分布式协调服务。这样我们就可以利用 Zookeeper 来协调多个分布式进程之间的活动。比如在一个分布式环境中，为了提高可靠性，我们的集群的每台服务器上都部署着同样的服务。但是，一件事情如果集群中的每个服务器都进行的话，那相互之间就要协调，编程起来将非常复杂。而如果我们只让一个服务进行操作，那又存在单点。通常还有一种做法就是使用分布式锁，在某个时刻只让一个服务去干活，当这台服务出问题的时候锁释放，立即 fail over 到另外的服务。这在很多分布式系统中都是这么做，这种设计有一个更好听的名字叫 Leader Election(leader 选举)。比如 HBase的 Master 就是采用这种机制。但要注意的是分布式锁跟同一个进程的锁还是有区别的，所以使用的时候要比同一个进程里的锁更谨慎的使用。

#### 集群管理

在分布式的集群中，经常会由于各种原因，比如硬件故障，软件故障，网络问题，有些节点会进进出出。有新的节点加入进来，也有老的节点退出集群。这个时候，集群中其他机器需要感知到这种变化，然后根据这种变化做出对应的决策。比如我们是一个分布式存储系统，有一个中央控制节点负责存储的分配，当有新的存储进来的时候我们要根据现在集群目前的状态来分配存储节点。这个时候我们就需要动态感知到集群目前的状态。还有，比如一
个分布式的 SOA 架构中，服务是一个集群提供的，当消费者访问某个服务时，就需要采用某种机制发现现在有哪些节点可以提供该服务(这也称之为服务发现，比如 Alibaba 开源的SOA 框架 Dubbo 就采用了 Zookeeper 作为服务发现的底层机制)。还有开源的 Kafka 队列就采用了 Zookeeper 作为 Cosnumer 的上下线管理。

## 特性

ZooKeeper 具有以下特性：

- **顺序一致性：**所有客户端看到的服务端数据模型都是一致的；从一个客户端发起的事务请求，最终都会严格按照其发起顺序被应用到 ZooKeeper 中。具体的实现可见下文：原子广播。
- **原子性：**所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，即整个集群要么都成功应用了某个事务，要么都没有应用。实现方式可见下文：事务。
- **单一视图：**无论客户端连接的是哪个 Zookeeper 服务器，其看到的服务端数据模型都是一致的。
- **高性能：**ZooKeeper 将数据全量存储在内存中，所以其性能很高。需要注意的是：由于 ZooKeeper 的所有更新和删除都是基于事务的，因此 ZooKeeper 在读多写少的应用场景中有性能表现较好，如果写操作频繁，性能会大大下滑。
- **高可用：**ZooKeeper 的高可用是基于副本机制实现的，此外 ZooKeeper 支持故障恢复，可见下文：选举 Leader。

## 核心概念

### 数据模型

**ZooKeeper 的数据模型是一个树形结构的文件系统。**

树中的节点被称为 znode，其中根节点为 /，每个节点上都会保存自己的数据和节点信息。znode 可以用于存储数据，并且有一个与之相关联的 ACL（详情可见 ACL）。ZooKeeper 的设计目标是实现协调服务，而不是真的作为一个文件存储，因此 znode 存储数据的**大小被限制在 1MB 以内。**

**ZooKeeper 的数据访问具有原子性。**其读写操作都是要么全部成功，要么全部失败。

znode 通过路径被引用。**znode 节点路径必须是绝对路径。**

znode 有两种类型：

- **临时的（ EPHEMERAL ）：**客户端会话结束时，ZooKeeper 就会删除临时的 znode。
- **持久的（PERSISTENT ）：**除非客户端主动执行删除操作，否则 ZooKeeper 不会删除持久的 znode。

在Zookeeper中存储的创建的结点和存储的数据包含结点的创建时间、修改时间、结点id、结点中存储数据的版本、权限版本、孩子结点的个数、数据的长度等信息。

`/zookeeper`是一个保留词，不能用作一个路径组件。`Zookeeper`使用`/zookeeper`来保存管理信息

### Znode

Znode是Zookeeper中数据的最小单元，每个Znode上都可以保存数据，同时还可以挂载子节点，znode之间的层级关系就像文件系统的目录结构一样，zookeeper将全部的数据存储在内存中以此来提高服务器吞吐量、减少延迟的目的。

Znode有三种类型，Znode的类型在创建时确定并且不能修改。

#### **持久节点(PERSISTENT):**

- 这种节点也是在 ZooKeeper 最为常用的，几乎所有业务场景中都会包含持久节点的创建。
- 之所以叫作持久节点是因为一旦将节点创建为持久节点，该数据节点会一直存储在 ZooKeeper 服务器上，即使创建该节点的客户端与服务端的会话关闭了，该节点依然不会被删除。
- 如果想删除持久节点，就要显式调用 delete 函数进行删除操作。
- Zookeeper规定所有非叶子节点必须是持久化节点

#### **临时节点(EPHEMERAL):**

- 从名称上可以看出该节点的一个最重要的特性就是临时性。
- 所谓临时性是指，如果将节点创建为临时节点，那么该节点数据不会一直存储在 ZooKeeper 服务器上。
- 当创建该临时节点的客户端会话因超时或发生异常而关闭时，该节点也相应在 ZooKeeper 服务器上被删除。
- 同样，也可以像删除持久节点一样主动删除临时节点。
- 在创建临时Znode的客户端会话结束时，服务器会将临时节点删除。临时节点不能有子节点(即使是临时子节点)。虽然每个临时Znode都会绑定一个特定的客户端会话，但是它们对所有客户端都是可见的

#### **有序节点(SEQUENTIAL):**

- 其实有序节点并不算是一种单独种类的节点，而是在之前提到的持久节点和临时节点特性的基础上，增加了一个节点有序的性质。
- 所谓节点有序是说在创建有序节点的时候，ZooKeeper 服务器会自动使用一个单调递增的数字作为后缀，追加到创建节点的后边。
- 例如一个客户端创建了一个路径为 works/task- 的有序节点，那么 ZooKeeper 将会生成一个序号并追加到该节点的路径后，最后该节点的路径为 works/task-1。
- 通过这种方式可以直观的查看到节点的创建顺序。

每个数据节点除了存储数据内容之外，还存储了数据节点本身的一些状态信息。Znode的状态(Stat)信息

```
 [zk: 127.0.0.1:2181(CONNECTED) 1] get /jannal-create/user/password
    123456
    cZxid = 0x6000000e4
    ctime = Wed Oct 03 22:46:07 CST 2018
    mZxid = 0x6000000e4
    mtime = Wed Oct 03 22:46:07 CST 2018
    pZxid = 0x6000000e4
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 6
    numChildren = 0
```

每一个节点都有一个自己的状态属性，记录了节点本身的一些信息，这些属性包括的内容如下表所示：

| 状态属性       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| cZxid          | Created ZXID表示该数据节点被创建时的事务ID                   |
| ctime          | Created Time表示节点被创建的时间                             |
| mZxid          | Modified ZXID 表示该节点最后一次被更新时的事务ID             |
| mtime          | Modified Time表示节点最后一次被更新的时间                    |
| pZxid          | 表示该节点的子节点列表最后一次被修改时的事务ID。只有子节点列表变更了才会变更pZxid,子节点内容变更不会影响pZxid |
| cversion       | 子节点的版本号这表示对此znode的子节点进行的更改次数          |
| dataVersion    | 数据节点版本号表示对该znode的数据所做的更改次数              |
| aclVersion     | 节点的ACL版本号表示对此znode的ACL进行更改的次数              |
| ephemeralOwner | 创建该临时节点的会话的SessionID。如果节点是持久节点，这个属性为0 |
| dataLength     | 数据内容的长度                                               |
| numChildren    | 当前节点的子节点个数                                         |

Zookeeper中版本(version)表示的是对数据节点的数据内容、子节点列表或是节点ACL信息的修改次数，初始为0表示没有被修改，每当对数据内容进行更新，version就会加1。如果前后两次修改并没有使得数据内容发生变化，version的值依然会变化。修改时如果设置version=-1,表示客户端并不要求使用乐观锁，可以忽略版本对比。

```
    //第一次dataVersion是0
    [zk: 127.0.0.1:2181(CONNECTED) 1] get /jannal-create/user/password
    123456
    cZxid = 0x6000000e4
    ctime = Wed Oct 03 22:46:07 CST 2018
    mZxid = 0x6000000e4
    mtime = Wed Oct 03 22:46:07 CST 2018
    pZxid = 0x6000000e4
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 6
    numChildren = 0
    
    //将值修改为1234567后dataVersion变为1
    [zk: 127.0.0.1:2181(CONNECTED) 3] set /jannal-create/user/password 1234567
    cZxid = 0x6000000e4
    ctime = Wed Oct 03 22:46:07 CST 2018
    mZxid = 0x6000000e6
    mtime = Wed Oct 03 23:02:52 CST 2018
    pZxid = 0x6000000e4
    cversion = 0
    dataVersion = 1
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 7
    numChildren = 0
    
    //将值修改为原来的123456后dataVersion变为2
    [zk: 127.0.0.1:2181(CONNECTED) 4] set /jannal-create/user/password 123456
    cZxid = 0x6000000e4
    ctime = Wed Oct 03 22:46:07 CST 2018
    mZxid = 0x6000000e7
    mtime = Wed Oct 03 23:02:57 CST 2018
    pZxid = 0x6000000e4
    cversion = 0
    dataVersion = 2
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 6
    numChildren = 0

```

### ACL

ZooKeeper 采用 ACL（Access Control Lists）策略来进行权限控制。

每个 znode 创建时都会带有一个 ACL 列表，用于决定谁可以对它执行何种操作。

Zookeeper的ACL由<schema>:<id>:<acl>三段组成。

- schema：可以取下列值：world, auth, digest, host/ip
- id： 标识身份，值依赖于schema做解析。
- acl：就是权限：cdwra分别表示create, delete,write,read, admin

#### **scheme**:

scheme对应于采用哪种方案来进行权限管理，zookeeper实现了一个pluggable的ACL方案，可以通过扩展scheme，来扩展ACL的机制。zookeeper-3.4.4缺省支持下面几种scheme:

- - **world**: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的
  - **auth**: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)
  - **digest**: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication
  - **ip**: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段
  - **super**: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)

另外，zookeeper-3.4.4的代码中还提供了对sasl的支持，不过缺省是没有开启的，需要配置才能启用，具体怎么配置在下文中介绍。

- **sasl**: sasl的对应的id，是一个通过sasl authentication用户的id，zookeeper-3.4.4中的sasl authentication是通过kerberos来实现的，也就是说用户只有通过了kerberos认证，才能访问它有权限的node.

#### **id**: 

id与scheme是紧密相关的，具体的情况在上面介绍scheme的过程都已介绍，这里不再赘述。

#### **permission**:

 zookeeper目前支持下面一些权限：

- CREATE(c): 创建权限，可以在在当前node下创建child node
- DELETE(d): 删除权限，可以删除当前的node
- READ(r): 读权限，可以获取当前node的数据，可以list当前node所有的child nodes
- WRITE(w): 写权限，可以向当前node写数据
- ADMIN(a): 管理权限，可以设置当前node的permission

注意：zookeeper对权限的控制是znode级别的，不具有继承性，即子节点不继承父节点的权限。这种设计在使用上还是有缺陷的，因为很多场景下，我们还是会把相关资源组织一下，放在同一个路径下面，这样就会有对一个路径统一授权的需求。

### Session

Session 指的是 ZooKeeper 服务器与客户端会话。在 ZooKeeper 中，一个客户端连接是指客户端和服务器之间的一个 TCP 长连接。客户端启动的时候，首先会与服务器建立一个 TCP 连接，从第一次连接建立开始，客户端会话的生命周期也开始了。通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，也能够向Zookeeper服务器发送请求并接受响应，同时还能够通过该连接接收来自服务器的Watch事件通知。 Session的sessionTimeout值用来设置一个客户端会话的超时时间。当由于服务器压力太大、网络故障或是客户端主动断开连接等各种原因导致客户端连接断开时，只要在sessionTimeout规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话仍然有效。

在为客户端创建会话之前，服务端首先会为每个客户端都分配一个sessionID。由于 sessionID 是 Zookeeper 会话的一个重要标识，许多与会话相关的运行机制都是基于这个 sessionID 的，因此，无论是哪台服务器为客户端分配的 sessionID，都务必保证全局唯一。

### Watcher

#### Watcher 监听机制：

 Watcher,就是事件监听器，Zookeeper 中非常重要的特性，Zookeeper允许用户注册一些watcher,并且在一些事件特定触发时，ZooKeeper会将通知发给客户端。我们基于 zookeeper 上创建的节点，可以对这些节点绑定监听事件，比如可以监听节点数据变更、节点删除、子节点状态变更等事件，通过这个事件机制，可以基于 zookeeper实现分布式锁、集群管理等功能。

#### watcher 特性：

当数据发生变化的时候， zookeeper 会产生一个 watcher 事件，并且会发送到客户端。但是客户端只会收到一次通知。如果后续这个节点再次发生变化，那么之前设置 watcher 的客户端不会再次收到消息。（watcher 是一次性的操作）。 可以通过循环监听去达到永久监听效果。

#### 如何注册事件机制：

ZooKeeper 的 Watcher 机制，总的来说可以分为三个过程：**客户端注册 Watcher、服务器处理 Watcher 和客户端回调 Watcher客户端。**注册 watcher 有 3 种方式，getData、exists、getChildren

## 集群角色

Zookeeper 集群是一个基于主从复制的高可用集群，每个服务器承担如下三种角色中的一种。

- **Leader：**它负责 发起并维护与各 Follwer 及 Observer 间的心跳。所有的写操作必须要通过 Leader 完成再由 Leader 将写操作广播给其它服务器。一个 Zookeeper 集群同一时间只会有一个实际工作的 Leader。
- **Follower：**它会响应 Leader 的心跳。Follower 可直接处理并返回客户端的读请求，同时会将写请求转发给 Leader 处理，并且负责在 Leader 处理写请求时对请求进行投票。一个 Zookeeper 集群可能同时存在多个 Follower。
- **Observer：**角色与 Follower 类似，但是无投票权。

当Leader服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB协议就会进入恢复模式并选举产生新的Leader。
这个过程大致是这样的：
**Leader election(选举阶段)**：节点在一开始都处于选举阶段，只要有一个节点得到超过半数节点的票数，它就可以当选准 Leader。
**Discovery(发现阶段)：**在这个阶段，Followers跟准Leader进行通信，同步Followers最近接收的事务提议。
**Synchronization(同步阶段)：**同步阶段主要是利用Leader前一阶段获得的最新提议历史，同步集群中所有的副本。同步完成之后准Leader才会成为真正的Leader。
**Broadcast(广播阶段)：** 到了这个阶段，Zookeeper集群才能正式对外提供事务服务，并且Leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。zookeeper有三种角色：老大Leader(领导者）   2、老二Follower （跟随者） 3、老三Observer（观察者）。其中，Follower和Observer归类为Learner（学习者）



按重要性排序是Leader > Follower > Observer

### Leader

Leader在集群中只有一个节点，可以说是老大No.1，是zookeeper集群的中心，负责协调集群中的其他节点。从性能的角度考虑，leader可以选择不接受客户端的连接。

主要作用有：

1、发起与提交写请求。

所有的跟随者Follower与观察者Observer节点的写请求都会转交给领导者Leader执行。Leader接受到一个写请求后，首先会发送给所有的Follower，统计Follower写入成功的数量。当有超过半数的Follower写入成功后，Leader就会认为这个写请求提交成功，通知所有的Follower commit这个写操作，保证事后哪怕是集群崩溃恢复或者重启，这个写操作也不会丢失。

2、与learner保持心跳

3、崩溃恢复时负责恢复数据以及同步数据到Learner

### Follower

Follow在集群中有多个，主要的作用有：

1、与老大Leader保持心跳连接

2、当Leader挂了的时候，经过投票后成为新的leader。leader的重新选举是由老二Follower们内部投票决定的。

3、向leader发送消息与请求

4、处理leader发来的消息与请求

###  Observer

可以说Observer是zookeeper集群中最边缘的存在。Observer的主要作用是提高zookeeper集群的读性能。通过leader的介绍我们知道zookeeper的一个写操作是要经过半数以上的Follower确认才能够写成功的。那么当zookeeper集群中的节点越多时，zookeeper的写性能就 越差。为了在提高zookeeper读性能（也就是支持更多的客户端连接）的同时又不影响zookeeper的写性能，zookeeper集群多了一个儿子Observer，只负责：

1、与leader同步数据

2、不参与leader选举，没有投票权。也不参与写操作的提议过程。

3、数据没有事务化到硬盘。即Observer只会把数据加载到内存。



## ZooKeeper 工作原理

### 读操作

**Leader/Follower/Observer 都可直接处理读请求，从本地内存中读取数据并返回给客户端即可。**

由于处理读请求不需要服务器之间的交互，**Follower/Observer 越多，整体系统的读请求吞吐量越大**，也即读性能越好。

![Zookeeper读操作](png\zookeeper\Zookeeper读操作.png)

 ![img](https://oscimg.oschina.net/oscnet/up-c61929f0f90b482800ca0959e55b3da3f85.png)

### 写操作

所有的写请求实际上都要交给 Leader 处理。Leader 将写请求以事务形式发给所有 Follower 并等待 ACK，一旦收到半数以上 Follower 的 ACK，即认为写操作成功。

#### **写 Leader**

![Zookeeper写Lead](png\zookeeper\Zookeeper写Lead.png) ![img](https://oscimg.oschina.net/oscnet/up-166f5dc0cb39337f7e7dabf9de020af0f43.png)

由上图可见，通过 Leader 进行写操作，主要分为五步：

1. 客户端向 Leader 发起写请求。
2. Leader 将写请求以事务 Proposal 的形式发给所有 Follower 并等待 ACK。
3. Follower 收到 Leader 的事务 Proposal 后返回 ACK。
4. Leader 得到过半数的 ACK（Leader 对自己默认有一个 ACK）后向所有的 Follower 和 Observer 发送 Commmit。
5. Leader 将处理结果返回给客户端。

**注意**

- Leader 不需要得到 Observer 的 ACK，即 Observer 无投票权。

- Leader 不需要得到所有 Follower 的 ACK，只要收到过半的 ACK 即可，同时 Leader 本身对自己有一个 ACK。上图中有 4 个 Follower，只需其中两个返回 ACK 即可，因为(2+1)/(4+1)>1/2(2+1)/(4+1)>1/2。

- Observer 虽然无投票权，但仍须同步 Leader 的数据从而在处理读请求时可以返回尽可能新的数据。

#### 写 Follower/Observer

![Zookeeper写Follower](png\zookeeper\Zookeeper写Follower.png)

Follower/Observer 均可接受写请求，但不能直接处理，而需要将写请求转发给 Leader 处理。

除了多了一步请求转发，其它流程与直接写 Leader 无任何区别。

## 事务

对于来自客户端的每个更新请求，ZooKeeper 具备严格的顺序访问控制能力。

**为了保证事务的顺序一致性，ZooKeeper 采用了递增的事务 id 号（zxid）来标识事务。**

**Leader 服务会为每一个 Follower 服务器分配一个单独的队列，然后将事务 Proposal 依次放入队列中，并根据 FIFO(先进先出) 的策略进行消息发送。**Follower 服务在接收到 Proposal 后，会将其以事务日志的形式写入本地磁盘中，并在写入成功后反馈给 Leader 一个 Ack 响应。**当 Leader 接收到超过半数 Follower 的 Ack 响应后，就会广播一个 Commit 消息给所有的 Follower 以通知其进行事务提交，**之后 Leader 自身也会完成对事务的提交。而每一个 Follower 则在接收到 Commit 消息后，完成事务的提交。

所有的提议（proposal）都在被提出的时候加上了 zxid。zxid 是一个 64 位的数字，它的高 32 位是 epoch 用来标识 Leader 关系是否改变，每次一个 Leader 被选出来，它都会有一个新的 epoch，标识当前属于那个 leader 的统治时期。低 32 位用于递增计数。

详细过程如下：

- Leader 等待 Server 连接；
- Follower 连接 Leader，将最大的 zxid 发送给 Leader；
- Leader 根据 Follower 的 zxid 确定同步点；
- 完成同步后通知 follower 已经成为 uptodate 状态；
- Follower 收到 uptodate 消息后，又可以重新接受 client 的请求进行服务了。

## 观察

**客户端注册监听它关心的 znode，当 znode 状态发生变化（数据变化、子节点增减变化）时，ZooKeeper 服务会通知客户端。**

客户端和服务端保持连接一般有两种形式：

- 客户端向服务端不断轮询
- 服务端向客户端推送状态

Zookeeper 的选择是服务端主动推送状态，也就是观察机制（ Watch ）。

ZooKeeper 的观察机制允许用户在指定节点上针对感兴趣的事件注册监听，当事件发生时，监听器会被触发，并将事件信息推送到客户端。

客户端使用 getData 等接口获取 znode 状态时传入了一个用于处理节点变更的回调，那么服务端就会主动向客户端推送节点的变更：

从这个方法中传入的 Watcher 对象实现了相应的 process 方法，每次对应节点出现了状态的改变，WatchManager 都会通过以下的方式调用传入 Watcher 的方法：

```
Set<Watcher> triggerWatch(String path, EventType type, Set<Watcher> supress) {
    WatchedEvent e = new WatchedEvent(type, KeeperState.SyncConnected, path);
    Set<Watcher> watchers;
    synchronized (this) {
        watchers = watchTable.remove(path);
    }
    for (Watcher w : watchers) {
        w.process(e);
    }
    return
```

Zookeeper 中的所有数据其实都是由一个名为 DataTree 的数据结构管理的，所有的读写数据的请求最终都会改变这颗树的内容，在发出读请求时可能会传入 Watcher 注册一个回调函数，而写请求就可能会触发相应的回调，由 WatchManager 通知客户端数据的变化。

通知机制的实现其实还是比较简单的，通过读请求设置 Watcher 监听事件，写请求在触发事件时就能将通知发送给指定的客户端。

## 会话

**ZooKeeper 客户端通过 TCP 长连接连接到 ZooKeeper 服务集群。会话 (Session) 从第一次连接开始就已经建立，之后通过心跳检测机制来保持有效的会话状态**。通过这个连接，客户端可以发送请求并接收响应，同时也可以接收到 Watch 事件的通知。

每个 ZooKeeper 客户端配置中都配置了 ZooKeeper 服务器集群列表。启动时，客户端会遍历列表去尝试建立连接。如果失败，它会尝试连接下一个服务器，依次类推。

一旦一台客户端与一台服务器建立连接，这台服务器会为这个客户端创建一个新的会话。**每个会话都会有一个超时时间，若服务器在超时时间内没有收到任何请求，则相应会话被视为过期。**一旦会话过期，就无法再重新打开，且任何与该会话相关的临时 znode 都会被删除。

通常来说，会话应该长期存在，而这需要由客户端来保证。客户端可以通过心跳方式（ping）来保持会话不过期。

![Zookeeper会话](png\zookeeper\Zookeeper会话.png)

![img](https://oscimg.oschina.net/oscnet/up-15bf9f99ba1b79dd16f89ccb76627f67f2f.png)

ZooKeeper 的会话具有四个属性：

- **sessionID：**会话 ID，唯一标识一个会话，每次客户端创建新的会话时，Zookeeper 都会为其分配一个全局唯一的 sessionID。
- **TimeOut：**会话超时时间，客户端在构造 Zookeeper 实例时，会配置 sessionTimeout 参数用于指定会话的超时时间，Zookeeper 客户端向服务端发送这个超时时间后，服务端会根据自己的超时时间限制最终确定会话的超时时间。
- **TickTime：**下次会话超时时间点，为了便于 Zookeeper 对会话实行”分桶策略”管理，同时为了高效低耗地实现会话的超时检查与清理，Zookeeper 会为每个会话标记一个下次会话超时时间点，其值大致等于当前时间加上 TimeOut。
- **isClosing：**标记一个会话是否已经被关闭，当服务端检测到会话已经超时失效时，会将该会话的 isClosing 标记为”已关闭”，这样就能确保不再处理来自该会话的新请求了。

Zookeeper 的会话管理主要是通过 SessionTracker 来负责，其采用了**分桶策略**（将类似的会话放在同一区块中进行管理）进行管理，以便 Zookeeper 对会话进行不同区块的隔离处理以及同一区块的统一处理。

## ZAB 协议

ZooKeeper 并没有直接采用 Paxos 算法，而是采用了名为 ZAB 的一致性协议。ZAB 协议不是 Paxos 算法，只是比较类似，二者在操作上并不相同。

Zab协议 的全称是 **Zookeeper Atomic Broadcast** （[Zookeeper](https://so.csdn.net/so/search?q=Zookeeper&spm=1001.2101.3001.7020)原子广播）。
**Zookeeper 是通过 Zab 协议来保证分布式事务的最终一致性**。

ZAB 协议是 Zookeeper 专门设计的一种支持崩溃恢复的原子广播协议。

ZAB 协议是 ZooKeeper 的数据一致性和高可用解决方案。

ZAB 协议定义了两个可以**无限循环**的流程：

- **选举 Leader：**用于故障恢复，从而保证高可用。
- **原子广播：**用于主从同步，从而保证数据一致性。

Zookeeper 客户端会随机的链接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据；如果是写请求，那么节点就会向 Leader 提交事务，Leader 接收到事务提交，会广播该事务，只要超过半数节点写入成功，该事务就会被提交。

**Zab 协议的特性**：
1）Zab 协议需要确保那些**已经在 Leader 服务器上提交（Commit）的事务最终被所有的服务器提交**。
2）Zab 协议需要确保**丢弃那些只在 Leader 上被提出而没有被提交的事务**。

### Zab 协议实现的作用

1）**使用一个单一的主进程（Leader）来接收并处理客户端的事务请求**（也就是写请求），并采用了Zab的原子广播协议，将服务器数据的状态变更以 **事务proposal** （事务提议）的形式广播到所有的副本（Follower）进程上去。

2）**保证一个全局的变更序列被顺序引用**。
Zookeeper是一个树形结构，很多操作都要先检查才能确定是否可以执行，比如P1的事务t1可能是创建节点"/a"，t2可能是创建节点"/a/bb"，只有先创建了父节点"/a"，才能创建子节点"/a/b"。

为了保证这一点，Zab要保证同一个Leader发起的事务要按顺序被apply，同时还要保证只有先前Leader的事务被apply之后，新选举出来的Leader才能再次发起事务。

3）**当主进程出现异常的时候，整个zk集群依旧能正常工作**。

### Zab协议原理

Zab协议要求每个 Leader 都要经历三个阶段：**发现，同步，广播**。

- **发现**：要求zookeeper集群必须选举出一个 Leader 进程，同时 Leader 会维护一个 Follower 可用客户端列表。将来客户端可以和这些 Follower节点进行通信。
- **同步**：Leader 要负责将本身的数据与 Follower 完成同步，做到多副本存储。这样也是提现了CAP中的高可用和分区容错。Follower将队列中未处理完的请求消费完成后，写入本地事务日志中。
- **广播**：Leader 可以接受客户端新的事务Proposal请求，将新的Proposal请求广播给所有的 Follower。

### Zab协议核心

Zab协议的核心：**定义了事务请求的处理方式**

1）所有的事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被叫做 **Leader服务器**。其他剩余的服务器则是 **Follower服务器**。

2）Leader服务器 负责将一个客户端事务请求，转换成一个 **事务Proposal**，并将该 Proposal 分发给集群中所有的 Follower 服务器，也就是向所有 Follower 节点发送数据广播请求（或数据复制）

3）分发之后Leader服务器需要等待所有Follower服务器的反馈（Ack请求），**在Zab协议中，只要超过半数的Follower服务器进行了正确的反馈**后（也就是收到半数以上的Follower的Ack请求），那么 Leader 就会再次向所有的 Follower服务器发送 Commit 消息，要求其将上一个 事务proposal 进行提交。

### Zab协议内容

Zab 协议包括两种基本的模式：**崩溃恢复** 和 **消息广播**

协议过程

当整个集群启动过程中，或者当 Leader 服务器出现网络中弄断、崩溃退出或重启等异常时，Zab协议就会 **进入崩溃恢复模式**，选举产生新的Leader。

当选举产生了新的 Leader，同时集群中有过半的机器与该 Leader 服务器完成了状态同步（即数据同步）之后，Zab协议就会退出崩溃恢复模式，**进入消息广播模式**。

这时，如果有一台遵守Zab协议的服务器加入集群，因为此时集群中已经存在一个Leader服务器在广播消息，那么该新加入的服务器自动进入恢复模式：找到Leader服务器，并且完成数据同步。同步完成后，作为新的Follower一起参与到消息广播流程中。

协议状态切换

当Leader出现崩溃退出或者机器重启，亦或是集群中不存在超过半数的服务器与Leader保存正常通信，Zab就会再一次进入崩溃恢复，发起新一轮Leader选举并实现数据同步。同步完成后又会进入消息广播模式，接收事务请求。

保证消息有序

在整个消息广播中，Leader会将每一个事务请求转换成对应的 proposal 来进行广播，并且在广播 事务Proposal 之前，Leader服务器会首先为这个事务Proposal分配一个全局单递增的唯一ID，称之为事务ID（即zxid），由于Zab协议需要保证每一个消息的严格的顺序关系，因此必须将每一个proposal按照其zxid的先后顺序进行排序和处理。

![ZookeeperZAB协议](png\zookeeper\ZookeeperZAB协议.png)

#### 消息广播

1）在zookeeper集群中，数据副本的传递策略就是采用消息广播模式。zookeeper中农数据副本的同步方式与二段提交相似，但是却又不同。二段提交要求协调者必须等到所有的参与者全部反馈ACK确认消息后，再发送commit消息。要求所有的参与者要么全部成功，要么全部失败。二段提交会产生严重的阻塞问题。

2）Zab协议中 Leader 等待 Follower 的ACK反馈消息是指“只要半数以上的Follower成功反馈即可，不需要收到全部Follower反馈”

消息广播具体步骤

1）客户端发起一个写操作请求。针对客户端的事务请求，leader服务器会先将该事物写到本地的log文件中

2）Leader 服务器将客户端的请求转化为事务 Proposal 提案，同时为每个 Proposal 分配一个全局的ID，即zxid。

3）Leader 服务器为每个 Follower 服务器分配一个单独的队列，然后将需要广播的 Proposal 依次放到队列中取，并且根据 FIFO 策略进行消息发送。

4）Follower 接收到 Proposal 后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向 Leader 反馈一个 Ack 响应消息。

- 如果写成功了，则给leader返回一个ACK消息
- 如果写失败了，follower认为该事务不能执行，就会返回no

5）Leader 接收到超过半数以上 Follower 的 Ack 响应消息后，即认为消息发送成功，可以发送 commit 消息。

6）Leader 向所有 Follower 广播 commit 消息，同时自身也会完成事务提交。Follower 接收到 commit 消息后，会将上一条事务提交。

**zookeeper 采用 Zab 协议的核心，就是只要有一台服务器提交了 Proposal，就要确保所有的服务器最终都能正确提交 Proposal。这也是 CAP/BASE 实现最终一致性的一个体现。**

**Leader 服务器与每一个 Follower 服务器之间都维护了一个单独的 FIFO 消息队列进行收发消息，使用队列消息可以做到异步解耦。 Leader 和 Follower 之间只需要往队列中发消息即可。如果使用同步的方式会引起阻塞，性能要下降很多。**

> 如果follower记录失败，但是leader去要求执行这个请求，follower会向leader发送一个请求，请求重新获取刚才的信息重新记录重新执行

为什么leader会收到follower返回的no？（为什么follower记录失败？）

- 网络问题。例如产生数据丢失导致follower没有请求
- follower在记录日志的时候发现日志被占用
- 磁盘问题。例如磁盘已满、磁盘损坏

#### 崩溃恢复

**一旦 Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与过半 Follower 的联系，那么就会进入崩溃恢复模式。**

1. 当leader服务器出现崩溃、重启等场景，或者因为网络问题导致过半的follower不能与leader服务器保持正常通信的时候，Zookeeper集群就会进入崩溃恢复模式
2. 进入崩溃恢复模式后，只要集群中存在过半的服务器能够彼此正常通信，那么就可以选举产生一个新的leader
3. 每次新选举的leader会自动分配一个全局递增的编号，即epochid
4. 当选举产生了新的leader服务器，同时集群中已经有过半的机器与该leader服务器完成了状态同步之后，ZAB协议就会退出恢复模式。其中，所谓的状态同步是指数据同步，用来保证集群中存在过半的机器能够和leader服务器的数据状态保持一致
5. 当集群中已经有过半的follower服务器完成了和leader服务器的状态同步，那么整个服务框架就可以进入消息广播模式了
6. 当一台同样遵守ZAB协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个Leader服务器在负责进行消息广播，那么新加入的服务器就会自觉地进入数据恢复模式：
   - 找到leader所在的服务器，并与其进行数据同步
   - 然后一起参与到消息广播流程中

> **作用**：避免单点故障
>
> 事务id由64位二进制数字（16位十六进制）组成 ，其中高32位对应了epochid，低32位对应了实际的事务id

在 Zab 协议中，为了保证程序的正确运行，整个恢复过程结束后需要选举出一个新的 Leader 服务器。因此 Zab 协议需要一个高效且可靠的 Leader 选举算法，从而确保能够快速选举出新的 Leader 。

Leader 选举算法不仅仅需要让 Leader 自己知道自己已经被选举为 Leader ，同时还需要让集群中的所有其他机器也能够快速感知到选举产生的新 Leader 服务器。

崩溃恢复主要包括两部分：**Leader选举** 和 **数据恢复**

Zab 协议如何保证数据一致性

假设两种异常情况：
1、一个事务在 Leader 上提交了，并且过半的 Folower 都响应 Ack 了，但是 Leader 在 Commit 消息发出之前挂了。
2、假设一个事务在 Leader 提出之后，Leader 挂了。

要确保如果发生上述两种情况，数据还能保持一致性，那么 Zab 协议选举算法必须满足以下要求：

**Zab 协议崩溃恢复要求满足以下两个要求**：
1）**确保已经被 Leader 提交的 Proposal 必须最终被所有的 Follower 服务器提交**。
2）**确保丢弃已经被 Leader 提出的但是没有被提交的 Proposal**。

根据上述要求
Zab协议需要保证选举出来的Leader需要满足以下条件：
1）**新选举出来的 Leader 不能包含未提交的 Proposal** 。
即新选举的 Leader 必须都是已经提交了 Proposal 的 Follower 服务器节点。
2）**新选举的 Leader 节点中含有最大的 zxid** 。
这样做的好处是可以避免 Leader 服务器检查 Proposal 的提交和丢弃工作。


Zab 如何数据同步

1）完成 Leader 选举后（新的 Leader 具有最高的zxid），在正式开始工作之前（接收事务请求，然后提出新的 Proposal），Leader 服务器会首先确认事务日志中的所有的 Proposal 是否已经被集群中过半的服务器 Commit。

2）Leader 服务器需要确保所有的 Follower 服务器能够接收到每一条事务的 Proposal ，并且能将所有已经提交的事务 Proposal 应用到内存数据中。等到 Follower 将所有尚未同步的事务 Proposal 都从 Leader 服务器上同步过啦并且应用到内存数据中以后，Leader 才会把该 Follower 加入到真正可用的 Follower 列表中。


Zab 数据同步过程中，如何处理需要丢弃的 Proposal

在 Zab 的事务编号 zxid 设计中，zxid是一个64位的数字。

其中低32位可以看成一个简单的单增计数器，针对客户端每一个事务请求，Leader 在产生新的 Proposal 事务时，都会对该计数器加1。而高32位则代表了 Leader 周期的 epoch 编号。

> epoch 编号可以理解为当前集群所处的年代，或者周期。每次Leader变更之后都会在 epoch 的基础上加1，这样旧的 Leader 崩溃恢复之后，其他Follower 也不会听它的了，因为 Follower 只服从epoch最高的 Leader 命令。

每当选举产生一个新的 Leader ，就会从这个 Leader 服务器上取出本地事务日志充最大编号 Proposal 的 zxid，并从 zxid 中解析得到对应的 epoch 编号，然后再对其加1，之后该编号就作为新的 epoch 值，并将低32位数字归零，由0开始重新生成zxid。

**Zab 协议通过 epoch 编号来区分 Leader 变化周期**，能够有效避免不同的 Leader 错误的使用了相同的 zxid 编号提出了不一样的 Proposal 的异常情况。

基于以上策略
**当一个包含了上一个 Leader 周期中尚未提交过的事务 Proposal 的服务器启动时，当这台机器加入集群中，以 Follower 角色连上 Leader 服务器后，Leader 服务器会根据自己服务器上最后提交的 Proposal 来和 Follower 服务器的 Proposal 进行比对，比对的结果肯定是 Leader 要求 Follower 进行一个回退操作，回退到一个确实已经被集群中过半机器 Commit 的最新 Proposal**。

#### 什么情况下zab协议会进入崩溃恢复模式？

- 1、当服务器启动时
- 2、当leader 服务器出现网络中断，崩溃或者重启的情况
- 3、当集群中已经不存在过半的服务器与Leader服务器保持正常通信。

#### zab协议进入崩溃恢复模式会做什么？

1、当leader出现问题，zab协议进入崩溃恢复模式，并且选举出新的leader。当新的leader选举出来以后，如果集群中已经有过半机器完成了leader服务器的状态同（数据同步），退出崩溃恢复，进入消息广播模式。

2、当新的机器加入到集群中的时候，如果已经存在leader服务器，那么新加入的服务器就会自觉进入崩溃恢复模式，找到leader进行数据同步。

#### 特殊情况下需要解决的两个问题：

##### 问题一：已经被处理的事务请求（proposal）不能丢（commit的）

> 当 leader 收到合法数量 follower 的 ACKs 后，就向各个 follower 广播 COMMIT 命令，同时也会在本地执行 COMMIT 并向连接的客户端返回「成功」。但是如果在各个 follower 在收到 COMMIT 命令前 leader 就挂了，导致剩下的服务器并没有执行都这条消息。

- 如何解决 *已经被处理的事务请求（proposal）不能丢（commit的）* 呢？

1、选举拥有 proposal 最大值（即 zxid 最大） 的节点作为新的 leader。

> 由于所有提案被 COMMIT 之前必须有合法数量的 follower ACK，即必须有合法数量的服务器的事务日志上有该提案的 proposal，因此，zxid最大也就是数据最新的节点保存了所有被 COMMIT 消息的 proposal 状态。

2、新的 leader 将自己事务日志中 proposal 但未 COMMIT 的消息处理。

3、新的 leader 与 follower 建立先进先出的队列， 先将自身有而 follower 没有的 proposal 发送给 follower，再将这些 proposal 的 COMMIT 命令发送给 follower，以保证所有的 follower 都保存了所有的 proposal、所有的 follower 都处理了所有的消息。通过以上策略，能保证已经被处理的消息不会丢。

##### 问题二：没被处理的事务请求（proposal）不能再次出现什么时候会出现事务请求被丢失呢？

> 当 leader 接收到消息请求生成 proposal 后就挂了，其他 follower 并没有收到此 proposal，因此经过恢复模式重新选了 leader 后，这条消息是被跳过的。 此时，之前挂了的 leader 重新启动并注册成了 follower，他保留了被跳过消息的 proposal 状态，与整个系统的状态是不一致的，需要将其删除。

- 如果解决呢？

Zab 通过巧妙的设计 zxid 来实现这一目的。

一个 zxid 是64位，高 32 是纪元（epoch）编号，每经过一次 leader 选举产生一个新的 leader，新 leader 会将 epoch 号 +1。低 32 位是消息计数器，每接收到一条消息这个值 +1，新 leader 选举后这个值重置为 0。

这样设计的好处是旧的 leader 挂了后重启，它不会被选举为 leader，因为此时它的 zxid 肯定小于当前的新 leader。当旧的 leader 作为 follower 接入新的 leader 后，新的 leader 会让它将所有的拥有旧的 epoch 号的未被 COMMIT 的 proposal 清除。

## 选举 Leader

**ZooKeeper 的故障恢复**

ZooKeeper 集群采用一主（称为 Leader）多从（称为 Follower）模式，主从节点通过副本机制保证数据一致。

- 如果 Follower 节点挂了 - ZooKeeper 集群中的每个节点都会单独在内存中维护自身的状态，并且各节点之间都保持着通讯，只要集群中有半数机器能够正常工作，那么整个集群就可以正常提供服务。
- 如果 Leader 节点挂了 - 如果 Leader 节点挂了，系统就不能正常工作了。此时，需要通过 ZAB 协议的选举 Leader 机制来进行故障恢复。

ZAB 协议的选举 Leader 机制简单来说，就是：基于过半选举机制产生新的 Leader，之后其他机器将从新的 Leader 上同步状态，当有过半机器完成状态同步后，就退出选举 Leader 模式，进入原子广播模式。

### 术语

**myid：**每个 Zookeeper 服务器，都需要在数据文件夹下创建一个名为 myid 的文件，该文件包含整个 Zookeeper 集群唯一的 ID（整数）。

**zxid：**类似于 RDBMS 中的事务 ID，用于标识一次更新操作的 Proposal ID。为了保证顺序性，该 zkid 必须单调递增。因此 Zookeeper 使用一个 64 位的数来表示，高 32 位是 Leader 的 epoch，从 1 开始，每次选出新的 Leader，epoch 加一。低 32 位为该 epoch 内的序号，每次 epoch 变化，都将低 32 位的序号重置。这样保证了 zkid 的全局递增性。

### 服务器状态

- **LOOKING：**不确定 Leader 状态。该状态下的服务器认为当前集群中没有 Leader，会发起 Leader 选举。
- **FOLLOWING：**跟随者状态。表明当前服务器角色是 Follower，并且它知道 Leader 是谁。
- **LEADING：**领导者状态。表明当前服务器角色是 Leader，它会维护与 Follower 间的心跳。
- **OBSERVING：**观察者状态。表明当前服务器角色是 Observer，与 Folower 唯一的不同在于不参与选举，也不参与集群写操作时的投票。

### 选票数据结构

每个服务器在进行领导选举时，会发送如下关键信息：

- **logicClock：**每个服务器会维护一个自增的整数，名为 logicClock，它表示这是该服务器发起的第多少轮投票。
- **state：**当前服务器的状态。
- **self_id：**当前服务器的 myid。
- **self_zxid：**当前服务器上所保存的数据的最大 zxid。
- **vote_id：**被推举的服务器的 myid。
- **vote_zxid：**被推举的服务器上所保存的数据的最大 zxid。

### 投票流程

![Zookeeper选举过程](png\zookeeper\Zookeeper选举过程.png)

**（1）自增选举轮次**

Zookeeper 规定所有有效的投票都必须在同一轮次中。每个服务器在开始新一轮投票时，会先对自己维护的 logicClock 进行自增操作。

**（2）初始化选票**

每个服务器在广播自己的选票前，会将自己的投票箱清空。该投票箱记录了所收到的选票。例：服务器 2 投票给服务器 3，服务器 3 投票给服务器 1，则服务器 1 的投票箱为(2, 3), (3, 1), (1, 1)。票箱中只会记录每一投票者的最后一票，如投票者更新自己的选票，则其它服务器收到该新选票后会在自己票箱中更新该服务器的选票。

**（3）发送初始化选票**

每个服务器最开始都是通过广播把票投给自己。

**（4）接收外部投票**

服务器会尝试从其它服务器获取投票，并记入自己的投票箱内。如果无法获取任何外部投票，则会确认自己是否与集群中其它服务器保持着有效连接。如果是，则再次发送自己的投票；如果否，则马上与之建立连接。

**（5）判断选举轮次**

收到外部投票后，首先会根据投票信息中所包含的 logicClock 来进行不同处理：

- 外部投票的 logicClock **大于**自己的 logicClock。说明该服务器的选举轮次落后于其它服务器的选举轮次，立即清空自己的投票箱并将自己的 logicClock 更新为收到的 logicClock，然后再对比自己之前的投票与收到的投票以确定是否需要变更自己的投票，最终再次将自己的投票广播出去。
- 外部投票的 logicClock **小于**自己的 logicClock。当前服务器直接忽略该投票，继续处理下一个投票。
- 外部投票的 logickClock 与自己的**相等**。当时进行选票 PK。

**（6）选票 PK**

选票 PK 是基于(self_id, self_zxid)与(vote_id, vote_zxid)的对比：

- 外部投票的 logicClock **大于**自己的 logicClock，则将自己的 logicClock 及自己的选票的 logicClock 变更为收到的 logicClock。
- 若 **logicClock** **一致**，则对比二者的 vote_zxid，若外部投票的 vote_zxid 比较大，则将自己的票中的 vote_zxid 与 vote_myid 更新为收到的票中的 vote_zxid 与 vote_myid 并广播出去，另外将收到的票及自己更新后的票放入自己的票箱。如果票箱内已存在(self_myid, self_zxid)相同的选票，则直接覆盖。
- 若二者 **vote_zxid** 一致，则比较二者的 vote_myid，若外部投票的 vote_myid 比较大，则将自己的票中的 vote_myid 更新为收到的票中的 vote_myid 并广播出去，另外将收到的票及自己更新后的票放入自己的票箱。

**（7）统计选票**

如果已经确定有过半服务器认可了自己的投票（可能是更新后的投票），则终止投票。否则继续接收其它服务器的投票。

**（8）更新服务器状态**

投票终止后，服务器开始更新自身状态。若过半的票投给了自己，则将自己的服务器状态更新为 LEADING，否则将自己的状态更新为 FOLLOWING。

通过以上流程分析，我们不难看出：要使 Leader 获得多数 Server 的支持，则 **ZooKeeper 集群节点数必须是奇数。且存活的节点数目不得少于 N + 1** 。

每个 Server 启动后都会重复以上流程。在恢复模式下，如果是刚从崩溃状态恢复的或者刚启动的 server 还会从磁盘快照中恢复数据和会话信息，zk 会记录事务日志并定期进行快照，方便在恢复时进行状态恢复。

## 原子广播（Atomic Broadcast）

**ZooKeeper 通过副本机制来实现高可用。**

那么，ZooKeeper 是如何实现副本机制的呢？答案是：ZAB 协议的原子广播。 ![img](https://oscimg.oschina.net/oscnet/up-5675f778bfc2caf41d78c998e665fe8a453.png)

ZAB 协议的原子广播要求：

**所有的写请求都会被转发给 Leader，Leader 会以原子广播的方式通知 Follow。当半数以上的 Follow 已经更新状态持久化后，Leader 才会提交这个更新，然后客户端才会收到一个更新成功的响应。**这有些类似数据库中的两阶段提交协议。

在整个消息的广播过程中，Leader 服务器会每个事物请求生成对应的 Proposal，并为其分配一个全局唯一的递增的事务 ID(ZXID)，之后再对其进行广播。

## 选举过程中的问题

### 脑裂问题

**脑裂问题**出现在集群中leader死掉，follower选出了新leader而原leader又复活了的情况下，因为ZK的过半机制是允许损失一定数量的机器而扔能正常提供给服务，当leader死亡判断不一致时就会出现多个leader。

方案：

ZK的过半机制一定程度上也减少了脑裂情况的出现，起码不会出现三个leader同时。ZK中的Epoch机制（时钟）每次选举都是递增+1，当通信时需要判断epoch是否一致，小于自己的则抛弃，大于自己则重置自己，等于则选举

## Zookeeper实战

### 单机版安装

1.解压zookeeper安装包（本人重命名为zookeeper，并移动到/usr/local路径下），此处只有解压命令

```
　tar -zxvf zookeeper-3.4.5.tar.gz
```

2.进入到zookeeper文件夹下，并创建data和logs文件夹（一般解压后都有data文件夹）

```
　　[root@localhost zookeeper]# cd /usr/local/zookeeper/

　　[root@localhost zookeeper]# mkdir logs
```

3.在conf目录下修改zoo.cfg文件（如果没有此文件，则自己新建该文件），修改为如下内容:

```
tickTime=2000
dataDir=/usr/myapp/zookeeper-3.4.5/data
dataLogDir=/usr/myapp/zookeeper-3.4.5/logs
clientPort=2181
```

最低配置

| 参数名                                | 默认    | 描述                                                         |
| ------------------------------------- | ------- | ------------------------------------------------------------ |
| clientPort                            |         | 服务的监听端口                                               |
| dataDir                               |         | 用于存放内存数据快照的文件夹，同时用于集群的myid文件也存在这个文件夹里 |
| tickTime                              | 2000    | Zookeeper的时间单元。Zookeeper中所有时间都是以这个时间单元的整数倍去配置的。例如，session的最小超时时间是2*tickTime。（单位：毫秒） |
| dataLogDir                            |         | 事务日志写入该配置指定的目录，而不是“ dataDir ”所指定的目录。这将允许使用一个专用的日志设备并且帮助我们避免日志和快照之间的竞争 |
| globalOutstandingLimit                | 1,000   | 最大请求堆积数。默认是1000。Zookeeper运行过程中，尽管Server没有空闲来处理更多的客户端请求了，但是还是允许客户端将请求提交到服务器上来，以提高吞吐性能。当然，为了防止Server内存溢出，这个请求堆积数还是需要限制下的。 |
| preAllocSize                          | 64M     | 预先开辟磁盘空间，用于后续写入事务日志。默认是64M，每个事务日志大小就是64M。如果ZK的快照频率较大的话，建议适当减小这个参数。 |
| snapCount                             | 100,000 | 每进行snapCount次事务日志输出后，触发一次快照， 此时，Zookeeper会生成一个snapshot.*文件，同时创建一个新的事务日志文件log.*。默认是100,000. |
| traceFile                             |         | 用于记录所有请求的log，一般调试过程中可以使用，但是生产环境不建议使用，会严重影响性能。 |
| maxClientCnxns                        |         | 最大并发客户端数，用于防止DOS的，默认值是10，设置为0是不加限制 |
| clientPortAddress / maxSessionTimeout |         | 对于多网卡的机器，可以为每个IP指定不同的监听端口。默认情况是所有IP都监听 clientPort 指定的端口 |
| minSessionTimeout                     |         | Session超时时间限制，如果客户端设置的超时时间不在这个范围，那么会被强制设置为最大或最小时间。默认的Session超时时间是在2 * tickTime ~ 20 * tickTime 这个范围 |
| fsync.warningthresholdms              | 1000    | 事务日志输出时，如果调用fsync方法超过指定的超时时间，那么会在日志中输出警告信息。默认是1000ms。 |
| autopurge.snapRetainCount             |         | 参数指定了需要保留的事务日志和快照文件的数目。默认是保留3个。和autopurge.purgeInterval搭配使用 |
| autopurge.purgeInterval               |         | 在3.4.0及之后版本，Zookeeper提供了自动清理事务日志和快照文件的功能，这个参数指定了清理频率，单位是小时，需要配置一个1或更大的整数，默认是0，表示不开启自动清理功能 |
| syncEnabled                           |         | Observer写入日志和生成快照，这样可以减少Observer的恢复时间。默认为true。 |

4.进入bin目录，启动、停止、重启分和查看当前节点状态

```
　　[root@localhost bin]# ./zkServer.sh start
　　[root@localhost bin]# ./zkServer.sh stop
　　[root@localhost bin]# ./zkServer.sh restart
　　[root@localhost bin]# ./zkServer.sh status
```

### 集群安装

zookeeper集群简介

Zookeeper集群中只要有过半的节点是正常的情况下,那么整个集群对外就是可用的。正是基于这个特性,要将 ZK 集群的节点数量要为奇数（2n+1），如 3、5、7 个节点)较为合适。

1.解压zookeeper安装包（本人重命名为zookeeper，并移动到/usr/local路径下），此处只有解压命令

```
$ tar -zxvf zookeeper-3.4.6.tar.gz
```

2.在各个zookeeper节点目录创建data、logs目录

```
　　[root@localhost zookeeper]# cd /usr/local/zookeeper/

　　[root@localhost zookeeper]# mkdir logs
```

3.修改zoo.cfg配置文件

```
tickTime=2000
initLimit=10 
syncLimit=5 
dataDir=/usr/myapp/zookeeper-3.4.5/data
dataLogDir=/usr/myapp/zookeeper-3.4.5/logs
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2888:3888
server.3=127.0.0.1:2888:3888
```

​		①、tickTime：基本事件单元，这个时间是作为Zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，每隔tickTime时间就会发送一个心跳；最小 的session过期时间为2倍tickTime

　　②、dataDir：存储内存中数据库快照的位置，除非另有说明，否则指向数据库更新的事务日志。注意：应该谨慎的选择日志存放的位置，使用专用的日志存储设备能够大大提高系统的性能，如果将日志存储在比较繁忙的存储设备上，那么将会很大程度上影像系统性能。

　　③、client：监听客户端连接的端口。

　　④、initLimit：允许follower连接并同步到Leader的初始化连接时间，以tickTime为单位。当初始化连接时间超过该值，则表示连接失败。

　　⑤、syncLimit：表示Leader与Follower之间发送消息时，请求和应答时间长度。如果follower在设置时间内不能与leader通信，那么此follower将会被丢弃。

　　⑥、server.A=B:C:D

　　　　A：其中 A 是一个数字，表示这个是服务器的编号；

　　　　B：是这个服务器的 ip 地址；

　　　　C：Zookeeper服务器之间的通信端口；

　　　　D：Leader选举的端口。

　　我们需要修改的第一个是 dataDir ,在指定的位置处创建好目录。

　　第二个需要新增的是 server.A=B:C:D 配置，其中 A 对应下面我们即将介绍的myid 文件。B是集群的各个IP地址，C:D 是端口配置。

*集群选项*

| 参数名                            | 默认 | 描述                                                         |
| --------------------------------- | ---- | ------------------------------------------------------------ |
| electionAlg                       |      | 之前的版本中， 这个参数配置是允许我们选择leader选举算法，但是由于在以后的版本中，只有“FastLeaderElection ”算法可用，所以这个参数目前看来没有用了。 |
| initLimit                         | 10   | Observer和Follower启动时，从Leader同步最新数据时，Leader允许initLimit * tickTime的时间内完成。如果同步的数据量很大，可以相应的把这个值设置的大一些。 |
| leaderServes                      | yes  | 默 认情况下，Leader是会接受客户端连接，并提供正常的读写服务。但是，如果你想让Leader专注于集群中机器的协调，那么可以将这个参数设置为 no，这样一来，会大大提高写操作的性能。一般机器数比较多的情况下可以设置为no，让Leader不接受客户端的连接。默认为yes |
| server.x=[hostname]:nnnnn[:nnnnn] |      | “x”是一个数字，与每个服务器的myid文件中的id是一样的。hostname是服务器的hostname，右边配置两个端口，第一个端口用于Follower和Leader之间的数据同步和其它通信，第二个端口用于Leader选举过程中投票通信。 |
| syncLimit                         |      | 表示Follower和Observer与Leader交互时的最大等待时间，只不过是在与leader同步完毕之后，进入正常请求转发或ping等消息交互时的超时时间。 |
| group.x=nnnnn[:nnnnn]             |      | “x”是一个数字，与每个服务器的myid文件中的id是一样的。对机器分组，后面的参数是myid文件中的ID |
| weight.x=nnnnn                    |      | “x”是一个数字，与每个服务器的myid文件中的id是一样的。机器的权重设置，后面的参数是权重值 |
| cnxTimeout                        | 5s   | 选举过程中打开一次连接的超时时间，默认是5s                   |
| standaloneEnabled                 |      | 当设置为false时，服务器在复制模式下启动                      |

 注意：
 zookeeper的启动日志在/bin目录下的zookeeper.out文件
 在启动第一个节点后，查看日志信息会看到如下异常：

```
java.net.ConnectException: Connection refused at java.net.PlainSocketImpl.socketConnect(Native Method) at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:339) at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:200) at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:182) at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392) at java.net.Socket.connect(Socket.java:579) at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:368) at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectAll(QuorumCnxManager.java:402) at org.apache.zookeeper.server.quorum.FastLeaderElection.lookForLeader(FastLeaderElection.java:840) at org.apache.zookeeper.server.quorum.QuorumPeer.run(QuorumPeer.java:762) 2016-07-30 17:13:16,032 [myid:1] - INFO [QuorumPeer[myid=1]/0:0:0:0:0:0:0:0:2181:FastLeaderElection@849] - Notification time out: 51200`
```

这是正常的，因为配置文件中配置了此节点是属于集群中的一个节点，zookeeper集群只有在过半的节点是正常的情况下，此节点才会正常，它是一直在检测集群其他两个节点的启动的情况。
 那在我们启动第二个节点之后，我们会看到原先启动的第一个节点不会在报错，因为这时候已经有过半的节点是正常的了。

### 开启observer

> 在ZooKeeper中，observer默认是不开启的，需要手动开启

在zoo.cfg中添加如下属性：

```
peerType=observer
```

在要配置为观察者的主机后添加观察者标记。例如：

```
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2888:3888
server.3=127.0.0.1:2888:3888:observer #表示将该节点设置为观察者
```

> observer不投票不选举，所以observer的存活与否不影响这个集群的服务。
>
> 例如：一共25个节点，其中1个leader和6个follower，剩余18个都是observer。那么即使所有的observer全部宕机，ZooKeeper依然对外提供服务，但是如果4个follower宕机，即使所有的observer都存活，这个ZooKeeper也不对外提供服务
>
> ==在ZooKeeper中过半性考虑的是leader和follower，而不包括observer==

## 常见指令

### 服务器端指令

| 指令                     | 说明               |
| ------------------------ | ------------------ |
| `sh zkServer.sh start`   | 启动服务器端       |
| `sh zkServer.sh stop`    | 停止服务器端       |
| `sh zkServer.sh restart` | 重启服务器端       |
| `sh zkServer.sh status`  | 查看服务器端的状态 |
| `sh zkCli.sh`            | 启动客户端         |

### 客户端指令

| 命令                               | 解释                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| help                               | 帮助命令                                                     |
| `quit`                             | 退出客户端                                                   |
| `ls /`                             | 查看根路径下的节点                                           |
| `create /log 'manage log servers'` | 在根节点下创建一个子节点log                                  |
| `creat -e /node2 ''`               | 在根节点下创建临时节点node2，客户端关闭后会删除              |
| `create -s /video ''`              | 在根节点下创建一个顺序节点`/video000000X`                    |
| `creare -e -s /node4 ''`           | 创建一个临时顺序节点`/node4000000X`,客户端关闭后删除         |
| `get /video`                       | 查看video节点的数据以及节点信息                              |
| `delete /log`                      | 删除根节点下的子节点log<br />==要求被删除的节点下没有子节点== |
| `rmr /video`                       | 递归删除                                                     |
| `set /video 'videos'`              | 修改节点数据                                                 |

### 常用四字命令

-  可以通过命令：echo stat|nc 127.0.0.1 2181 来查看哪个节点被选择作为follower或者leader
-  使用echo ruok|nc 127.0.0.1 2181 测试是否启动了该Server，若回复imok表示已经启动。
- echo dump| nc 127.0.0.1 2181 ,列出未经处理的会话和临时节点。
-  echo kill | nc 127.0.0.1 2181 ,关掉server
- echo conf | nc 127.0.0.1 2181 ,输出相关服务配置的详细信息。
-  echo cons | nc 127.0.0.1 2181 ,列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。
- echo envi |nc 127.0.0.1 2181 ,输出关于服务环境的详细信息（区别于 conf 命令）。
- echo reqs | nc 127.0.0.1 2181 ,列出未经处理的请求。
- echo wchs | nc 127.0.0.1 2181 ,列出服务器 watch 的详细信息。
- echo wchc | nc 127.0.0.1 2181 ,通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。
- echo wchp | nc 127.0.0.1 2181 ,通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。

## Zookeeper操作

### Maven依赖

```
<dependencies>
       <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
        </dependency>
</dependencies>
```

### 单机启动Zookeeper服务

```
package com.zookeeper;

import org.apache.zookeeper.server.ZooKeeperServerMain;

import java.util.Arrays;

public class Main {
    /**
     * 单机版ZooKeeper,启动类是ZooKeeperServerMain。
     * 最终调用ZooKeeperServer的startup()方法来处理request。
     * 可以通过配置选择使用JAVA自带NIO或者netty作为异步连接服务端。
     */
    public static void main(String[] args) {
        SingServer();
    }
    /**
     * 启动流程
     * ZooKeeper启动参数有两种配置方式：
     * 方式1：
     * main方法中传入四个参数，其中前两参数必填，后两个参数可选。
     * 分别为：对客户端暴露的端口clientPortAddress,
     * 存放事务记录、内存树快照记录的dataDir,
     * 用于指定seesion检查时间间隔的tickTime,
     * 控制最大客户端连接数的maxClientCnxns。
     *方式2：
     * 给出启动参数配置文件路径，
     * 当args启动参数只有一个时，ZooKeeperServerMain中main方法，
     * 会认为传入了配置文件路径，
     * 默认情况下该参数是传conf目录中的zoo.cfg。
     */
    public static void SingServer() {
        String clientPortAddress="2181";
        String dataDir="F:/data";
        String[] args= Arrays.asList(clientPortAddress,dataDir).toArray(new String[]{});
        ZooKeeperServerMain zooKeeperServerMain=new ZooKeeperServerMain();
        zooKeeperServerMain.main(args);
    }
}

```

### zookeeper节点操作

```
package com.zookeeper;

import lombok.extern.slf4j.Slf4j;
import org.apache.zookeeper.*;
import org.apache.zookeeper.ZooDefs.Ids;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;
import java.util.ArrayList;
@Slf4j
public class ZookeeperDemo {
    private static String connectString = "127.0.0.1:2181";
    private static int sessionTimeout = 50 * 1000;

    public static void main(String[] args) throws Exception{
        ZookeeperDemo zookeeperDemo=new ZookeeperDemo();
        ZooKeeper zooKeeper = zookeeperDemo.connectionZooKeeper();
        String result = zookeeperDemo.createZnode(zooKeeper, "/user", "zhangsan");
        log.info("创建zookeeper znode:{}",result);
        String znodeData = zookeeperDemo.getZnodeData(zooKeeper, "/user");
        log.info("获取zookeeper znode:{}",znodeData);
    }

    public ZooKeeper connectionZooKeeper() throws IOException {
        log.info("client连接Zookeeper");
        ZooKeeper zooKeeper = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            public void process(WatchedEvent event) {
                //可做其他操作（设置监听或观察者）
                log.info("监听机制");
            }
        });
        return zooKeeper;
    }


    /**
     * 创建节点
     * 1. CreateMode.PERSISTENT ：持久节点，一旦创建就保存到硬盘上面
     　　　  2.  CreateMode.SEQUENTIAL ： 顺序持久节点
     　　　  3.  CreateMode.EPHEMERAL ：临时节点，创建以后如果断开连接则该节点自动删除
     　　　  4.  CreateMode.EPHEMERAL_SEQUENTIAL ：顺序临时节点
     * @param zooKeeper Zookeeper已经建立连接的对象
     * @param path 要创建节点的路径
     * @param data 该节点上的数据
     * @return 返回创建的节点的路径
     * @throws KeeperException
     * @throws InterruptedException
     */
    public String createZnode(ZooKeeper zooKeeper, String path, String data) throws KeeperException, InterruptedException {
        byte[] bytesData = data.getBytes();
        //访问控制列表
        ArrayList<ACL> openAclUnsafe = Ids.OPEN_ACL_UNSAFE;
        //创建模式
        CreateMode mode = CreateMode.PERSISTENT;
        String result = zooKeeper.create(path, bytesData, openAclUnsafe, mode);
        return result;
    }

    /**
     * 获取节点上的数据
     * @param zooKeeper Zookeeper已经建立连接的对象
     * @param path 节点路径
     * @return 返回节点上的数据
     * @throws KeeperException
     * @throws InterruptedException
     */
    public String getZnodeData(ZooKeeper zooKeeper, String path) throws KeeperException, InterruptedException {
        byte[] data = zooKeeper.getData(path, false, new Stat());
        return new String(data);
    }

    /**
     * 设置节点上的数据
     * @param zooKeeper Zookeeper已经建立连接的对象
     * @param path 节点路径
     * @param data
     * @return
     * @throws KeeperException
     * @throws InterruptedException
     */
    public Stat setZnodeData(ZooKeeper zooKeeper, String path, String data) throws KeeperException, InterruptedException {
        return zooKeeper.setData(path, data.getBytes(), -1);
    }

    /**
     * 判断节点是否存在
     * @param zooKeeper
     * @param path 节点路径
     * @return
     * @throws KeeperException
     * @throws InterruptedException
     */
    public Stat isExitZKPath(ZooKeeper zooKeeper, String path) throws KeeperException, InterruptedException {
        Stat stat = zooKeeper.exists(path, false);
        return stat;
    }


}

```

## 开源客户端 ZkClient

### 简介

ZkClient是Github上一个开源的zookeeper客户端，在Zookeeper原生API接口之上进行了包装，是一个更易用的Zookeeper客户端，同时，zkClient在内部还实现了诸如Session超时重连、Watcher反复注册等功能

### Maven依赖

```
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.2</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

### zclient使用案例

```
package com.zookeeper;

import lombok.extern.slf4j.Slf4j;
import org.I0Itec.zkclient.ZkClient;
import org.I0Itec.zkclient.ZkConnection;

/**
 * ZkClient是在Zookeeper原声API接口之上进行了包装，
 * 是一个更易用的Zookeeper客户端，其
 * 内部还实现了诸如Session超时重连、
 * Watcher反复注册等功能。
 */
@Slf4j
public class ZkClientDemo {

    private static String connectString = "127.0.0.1:2181";
    private static int sessionTimeout = 50 * 1000;

    public static void main(String[] args) {
        ZkClientDemo zkClientDemo = new ZkClientDemo();
        ZkClient zkClient = zkClientDemo.connectionZooKeeper();
//        zclietDemo.createZnode(zkClient,"/user1","lisi");
        String znodeData = zkClientDemo.getZnodeData(zkClient, "/user1");
        log.info("znodeData:{}", znodeData);
    }

    /**
     * 使用ZkClient可以轻松的创建会话，连接到服务端。
     *
     * @param zkServers
     * @param connectionTimeout
     * @return
     */
    public ZkClient getZkClient(String zkServers, int connectionTimeout) {
        return new ZkClient(zkServers, connectionTimeout);
    }

    /**
     * ZkClient提供了递归创建节点的接口，
     * 即其帮助开发者完成父节点的创建，再创建子节点。
     * 结果表明已经成功创建了节点，
     * <p>
     * 值得注意的是，在原生态接口中是无法创建成功的（父节点不存在），
     * 但是通过ZkClient可以递归的先创建父节点，再创建子节点。
     *
     * @param zkClient
     * @param path
     */
    public void createPersistent(ZkClient zkClient, String path) {
        zkClient.createPersistent(path);
    }

    /**
     * ZkClient提供了递归删除节点的接口，
     * 即其帮助开发者先删除所有子节点（存在），再删除父节点。
     *
     * @param zkClient
     * @param path
     */
    public void deleteRecursive(ZkClient zkClient, String path) {
        zkClient.deleteRecursive(path);
    }


    public ZkClient connectionZooKeeper() {
        ZkClient zkClient = new ZkClient(new ZkConnection(connectString), sessionTimeout);
//        //对父节点添加监听子节点变化。
//        zkClient.subscribeChildChanges("", new IZkChildListener() {
//            @Override
//            public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
//               // 操作
//            }
//        });
        return zkClient;
    }

    public void createZnode(ZkClient zkClient, String path, Object data) {
        zkClient.createPersistent(path, data);
    }


    public String getZnodeData(ZkClient zkClient, String path) {
        return zkClient.readData(path);
    }


    public boolean isExitZKPath(ZkClient zkClient, String path) {
        return zkClient.exists(path);
    }
}
```

## Apache Curator（ZooKeeper客户端框架）

Curator是Netflix公司开源的一套zookeeper客户端框架，解决了很多Zookeeper客户端非常底层的细节开发工作，包括连接重连、反复注册Watcher和NodeExistsException异常等等。

Apache Curator是一个比较完善的ZooKeeper客户端框架，通过封装的一套高级API 简化了ZooKeeper的操作。通过查看官方文档，可以发现Curator主要解决了三类问题：

- 封装ZooKeeper client与ZooKeeper server之间的连接处理
- 提供了一套Fluent风格的操作API
- 提供ZooKeeper各种应用场景(recipe， 比如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等)的抽象封装

**Curator主要从以下几个方面降低了zk使用的复杂性：**

- 重试机制:提供可插拔的重试机制, 它将给捕获所有可恢复的异常配置一个重试策略，并且内部也提供了几种标准的重试策略(比如指数补偿)
- 连接状态监控: Curator初始化之后会一直对zk连接进行监听，一旦发现连接状态发生变化将会作出相应的处理
- zk客户端实例管理:Curator会对zk客户端到server集群的连接进行管理，并在需要的时候重建zk实例，保证与zk集群连接的可靠性
- 各种使用场景支持:Curator实现了zk支持的大部分使用场景（甚至包括zk自身不支持的场景），这些实现都遵循了zk的最佳实践，并考虑了各种极端情况

### Maven依赖

```
       <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.4.2</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

### Curator基本用法

```
package com.zookeeper;

import lombok.extern.slf4j.Slf4j;
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.transaction.CuratorTransactionBridge;
import org.apache.curator.framework.api.transaction.CuratorTransactionResult;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;

import java.util.List;

@Slf4j
public class CuratorDemo {

    //会话超时时间
    private static final int SESSION_TIMEOUT = 50 * 1000;

    //连接超时时间
    private static final int CONNECTION_TIMEOUT = 3 * 1000;

    //ZooKeeper服务地址
    private static final String CONNECT_ADDR = "127.0.0.1:2181";

    /**
     * Curator除了使用一般方法创建会话外，还可以使用fluent风格进行创建。
     * 值得注意的是session2会话含有隔离命名空间，
     * 即客户端对Zookeeper上数据节点的任何操作都是相对/base目录进行的，
     * 这有利于实现不同的Zookeeper的业务之间的隔离。
     */
    public static void main(String[] args) throws Exception {
        //1 重试策略：初试时间为1s 重试10次
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 10);
        //2 通过工厂创建连接
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(CONNECT_ADDR).connectionTimeoutMs(CONNECTION_TIMEOUT)
                .sessionTimeoutMs(SESSION_TIMEOUT)
                .retryPolicy(retryPolicy)
//命名空间           .namespace("super")
                .build();
        //3 开启连接
        client.start();

        System.out.println(ZooKeeper.States.CONNECTED);
        System.out.println(client.getState());

        //创建永久节点
        client.create().forPath("/curator","/curator data".getBytes());

        //创建永久有序节点
        client.create().withMode(CreateMode.PERSISTENT_SEQUENTIAL).forPath("/curator_sequential","/curator_sequential data".getBytes());

        //创建临时节点
        client.create().withMode(CreateMode.EPHEMERAL)
                .forPath("/curator/ephemeral","/curator/ephemeral data".getBytes());

        //创建临时有序节点
        client.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL) .forPath("/curator/ephemeral_path1","/curator/ephemeral_path1 data".getBytes());

        client.create().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath("/curator/ephemeral_path2","/curator/ephemeral_path2 data".getBytes());

        //测试检查某个节点是否存在
        Stat stat1 = client.checkExists().forPath("/curator");
        Stat stat2 = client.checkExists().forPath("/curator2");

        System.out.println("'/curator'是否存在： " + (stat1 != null ? true : false));
        System.out.println("'/curator2'是否存在： " + (stat2 != null ? true : false));

        //获取某个节点的所有子节点
        System.out.println(client.getChildren().forPath("/"));

        //获取某个节点数据
        System.out.println(new String(client.getData().forPath("/curator")));

        //设置某个节点数据
        client.setData().forPath("/curator","/curator modified data".getBytes());

        //创建测试节点
        client.create().creatingParentsIfNeeded()
                .forPath("/curator/del_key1","/curator/del_key1 data".getBytes());

        client.create().creatingParentsIfNeeded()
                .forPath("/curator/del_key2","/curator/del_key2 data".getBytes());

        client.create().forPath("/curator/del_key2/test_key","test_key data".getBytes());

        //删除该节点
        client.delete().forPath("/curator/del_key1");

        //级联删除子节点
        client.delete().guaranteed().deletingChildrenIfNeeded().forPath("/curator/del_key2");
    }
    
}


```

- orSetData()方法：如果节点存在则Curator将会使用给出的数据设置这个节点的值，相当于 setData() 方法
- creatingParentContainersIfNeeded()方法：如果指定节点的父节点不存在，则Curator将会自动级联创建父节点
- guaranteed()方法：如果服务端可能删除成功，但是client没有接收到删除成功的提示，Curator将会在后台持续尝试删除该节点
- deletingChildrenIfNeeded()方法：如果待删除节点存在子节点，则Curator将会级联删除该节点的子节点

### 事务管理

```
    /* 事务管理：碰到异常，事务会回滚
     * @throws Exception
     */
    @Test
    public void testTransaction() throws Exception{
    
    client.inTransaction().check().forPath("/nodeA")
                .and()
                .create().withMode(CreateMode.EPHEMERAL).forPath("/nodeB", "init".getBytes())
                .and()
                .create().withMode(CreateMode.EPHEMERAL).forPath("/nodeC", "init".getBytes())
                .and()
                .commit();
                
        //定义几个基本操作
        CuratorOp createOp = client.transactionOp().create()
                .forPath("/curator/one_path","some data".getBytes());
        
        CuratorOp setDataOp = client.transactionOp().setData()
                .forPath("/curator","other data".getBytes());
        
        CuratorOp deleteOp = client.transactionOp().delete()
                .forPath("/curator");
        
        //事务执行结果
        List<CuratorTransactionResult> results = client.transaction()
                .forOperations(createOp,setDataOp,deleteOp);
        
        //遍历输出结果
        for(CuratorTransactionResult result : results){
            System.out.println("执行结果是： " + result.getForPath() + "--" + result.getType());
        }
    }
//因为节点“/curator”存在子节点，所以在删除的时候将会报错，事务回滚
```

### 监听器

Curator提供了三种Watcher(Cache)来监听结点的变化：

- **Path Cache**：监视一个路径下1）孩子结点的创建、2）删除，3）以及结点数据的更新。产生的事件会传递给注册的PathChildrenCacheListener。
- **Node Cache**：监视一个结点的创建、更新、删除，并将结点的数据缓存在本地。
- **Tree Cache**：Path Cache和Node Cache的“合体”，监视路径下的创建、更新、删除事件，并缓存路径下所有孩子结点的数据。

```
/**
         * 在注册监听器的时候，如果传入此参数，当事件触发时，逻辑由线程池处理
         */
        ExecutorService pool = Executors.newFixedThreadPool(2);
        
        /**
         * 监听数据节点的变化情况
         */
        final NodeCache nodeCache = new NodeCache(client, "/zk-huey/cnode", false);
        nodeCache.start(true);
        nodeCache.getListenable().addListener(
            new NodeCacheListener() {
                @Override
                public void nodeChanged() throws Exception {
                    System.out.println("Node data is changed, new data: " + 
                        new String(nodeCache.getCurrentData().getData()));
                }
            }, 
            pool
        );
        
        /**
         * 监听子节点的变化情况
         */
        final PathChildrenCache childrenCache = new PathChildrenCache(client, "/zk-huey", true);
        childrenCache.start(StartMode.POST_INITIALIZED_EVENT);
        childrenCache.getListenable().addListener(
            new PathChildrenCacheListener() {
                @Override
                public void childEvent(CuratorFramework client, PathChildrenCacheEvent event)
                        throws Exception {
                        switch (event.getType()) {
                        case CHILD_ADDED:
                            System.out.println("CHILD_ADDED: " + event.getData().getPath());
                            break;
                        case CHILD_REMOVED:
                            System.out.println("CHILD_REMOVED: " + event.getData().getPath());
                            break;
                        case CHILD_UPDATED:
                            System.out.println("CHILD_UPDATED: " + event.getData().getPath());
                            break;
                        default:
                            break;
                    }
                }
            },
            pool
        );
        
        client.setData().forPath("/zk-huey/cnode", "world".getBytes());
        
        Thread.sleep(10 * 1000);
        pool.shutdown();
        client.close();
```

### 分布式锁

分布式编程时，比如最容易碰到的情况就是应用程序在线上多机部署，于是当多个应用同时访问某一资源时，就需要某种机制去协调它们。例如，现在一台应用正在rebuild缓存内容，要临时锁住某个区域暂时不让访问；又比如调度程序每次只想一个任务被一台应用执行等等。

下面的程序会启动两个线程t1和t2去争夺锁，拿到锁的线程会占用5秒。运行多次可以观察到，有时是t1先拿到锁而t2等待，有时又会反过来。Curator会用我们提供的lock路径的结点作为全局锁，这个结点的数据类似这种格式：[_c_64e0811f-9475-44ca-aa36-c1db65ae5350-lock-0000000005]，每次获得锁时会生成这种串，释放锁时清空数据。

```
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.RetryNTimes;

import java.util.concurrent.TimeUnit;

/**
 * Curator framework's distributed lock test.
 */
public class CuratorDistrLockTest {

    /** Zookeeper info */
    private static final String ZK_ADDRESS = "192.168.1.100:2181";
    private static final String ZK_LOCK_PATH = "/zktest";

    public static void main(String[] args) throws InterruptedException {
        // 1.Connect to zk
        CuratorFramework client = CuratorFrameworkFactory.newClient(
                ZK_ADDRESS,
                new RetryNTimes(10, 5000)
        );
        client.start();
        System.out.println("zk client start successfully!");

        Thread t1 = new Thread(() -> {
            doWithLock(client);
        }, "t1");
        Thread t2 = new Thread(() -> {
            doWithLock(client);
        }, "t2");

        t1.start();
        t2.start();
    }

    private static void doWithLock(CuratorFramework client) {
        InterProcessMutex lock = new InterProcessMutex(client, ZK_LOCK_PATH);
        try {
            if (lock.acquire(10 * 1000, TimeUnit.SECONDS)) {
                System.out.println(Thread.currentThread().getName() + " hold lock");
                Thread.sleep(5000L);
                System.out.println(Thread.currentThread().getName() + " release lock");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                lock.release();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Leader选举

当集群里的某个服务down机时，我们可能要从slave结点里选出一个作为新的master，这时就需要一套能在分布式环境中自动协调的Leader选举方法。Curator提供了LeaderSelector监听器实现Leader选举功能。同一时刻，只有一个Listener会进入takeLeadership()方法，说明它是当前的Leader。注意：**当Listener从takeLeadership()退出时就说明它放弃了“Leader身份”**，这时Curator会利用Zookeeper再从剩余的Listener中选出一个新的Leader。autoRequeue()方法使放弃Leadership的Listener有机会重新获得Leadership，如果不设置的话放弃了的Listener是不会再变成Leader的。

```
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.leader.LeaderSelector;
import org.apache.curator.framework.recipes.leader.LeaderSelectorListener;
import org.apache.curator.framework.state.ConnectionState;
import org.apache.curator.retry.RetryNTimes;
import org.apache.curator.utils.EnsurePath;

/**
 * Curator framework's leader election test.
 * Output:
 *  LeaderSelector-2 take leadership!
 *  LeaderSelector-2 relinquish leadership!
 *  LeaderSelector-1 take leadership!
 *  LeaderSelector-1 relinquish leadership!
 *  LeaderSelector-0 take leadership!
 *  LeaderSelector-0 relinquish leadership! 
 *      ...
 */
public class CuratorLeaderTest {

    /** Zookeeper info */
    private static final String ZK_ADDRESS = "192.168.1.100:2181";
    private static final String ZK_PATH = "/zktest";

    public static void main(String[] args) throws InterruptedException {
        LeaderSelectorListener listener = new LeaderSelectorListener() {
            @Override
            public void takeLeadership(CuratorFramework client) throws Exception {
                System.out.println(Thread.currentThread().getName() + " take leadership!");

                // takeLeadership() method should only return when leadership is being relinquished.
                Thread.sleep(5000L);

                System.out.println(Thread.currentThread().getName() + " relinquish leadership!");
            }

            @Override
            public void stateChanged(CuratorFramework client, ConnectionState state) {
            }
        };

        new Thread(() -> {
            registerListener(listener);
        }).start();

        new Thread(() -> {
            registerListener(listener);
        }).start();

        new Thread(() -> {
            registerListener(listener);
        }).start();

        Thread.sleep(Integer.MAX_VALUE);
    }

    private static void registerListener(LeaderSelectorListener listener) {
        // 1.Connect to zk
        CuratorFramework client = CuratorFrameworkFactory.newClient(
                ZK_ADDRESS,
                new RetryNTimes(10, 5000)
        );
        client.start();

        // 2.Ensure path
        try {
            new EnsurePath(ZK_PATH).ensure(client.getZookeeperClient());
        } catch (Exception e) {
            e.printStackTrace();
        }

        // 3.Register listener
        LeaderSelector selector = new LeaderSelector(client, ZK_PATH, listener);
        selector.autoRequeue();
        selector.start();
    }
}
```

# Zookeeper实战

## 数据发布/订阅

### 基本概念

（1）**数据发布/订阅系统即所谓的配置中心，也就是发布者将数据发布到ZooKeeper的一个节点或者一系列节点上，提供订阅者进行数据订阅，从而实现动态更新数据的目的，实现配置信息的集中式管理和数据的动态更新。**ZooKeeper采用的是推拉相结合的方式：客户端向服务器注册自己需要关注的节点，一旦该节点的数据发生改变，那么服务端就会向相应的客户端发送Wacher事件通知，客户端接收到消息通知后，需要主动到服务端获取最新的数据。

（2）实际系统开发过程中：**我们可以将初始化配置信息放到节点上集中管理，应用在启动时都会主动到ZooKeeper服务端进行一次配置读取，同时在指定节点注册Watcher监听，主要配置信息一旦变更，订阅者就可以获取读取最新的配置信息。**通常系统中需要使用一些通用的配置信息，比如机器列表信息、运行时的开关配置、数据库配置信息等全局配置信息，这些都会有以下3点特性：

　　1） 数据量通常比较小（通常是一些配置文件）

　　2） 数据内容在运行时会经常发生动态变化（比如数据库的临时切换等）

　　3） 集群中各机器共享，配置一致（比如数据库配置共享）。

（3）利用的ZooKeeper特性是：**ZooKeeper对任何节点（包括子节点）的变更，只要注册Wacther事件（使用Curator等客户端工具已经被封装好）都可以被其它客户端监听**

### 代码示例

```

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;

import java.util.concurrent.CountDownLatch;

public class ZooKeeper_Subsciption {
    private static final String ADDRESS = "xxx.xxx.xxx.xxx:2181";
    private static final int SESSION_TIMEOUT = 5000;
    private static final String PATH = "/configs";
    private static RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
    private static String config = "jdbc_configuration";
    private static CountDownLatch countDownLatch = new CountDownLatch(4);

    public static void main(String[] args) throws Exception {
        // 订阅该配置信息的集群节点（客户端）:sub1-sub3
        for (int i = 0; i < 3; i++) {
            CuratorFramework consumerClient = getClient();
            subscribe(consumerClient, "sub" + String.valueOf(i));
        }
        // 更改配置信息的集群节点（客户端）:pub
        CuratorFramework publisherClient = getClient();
        publish(publisherClient, "pub");

    }
    private static void init() throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(SESSION_TIMEOUT)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
        // 检查节点是否存在，不存在则初始化创建
        if (client.checkExists().forPath(PATH) == null) {
            client.create()
                    .creatingParentsIfNeeded()
                    .withMode(CreateMode.EPHEMERAL)
                    .forPath(PATH, config.getBytes());
        }
    }


    /**
     * 创建客户端并且初始化建立一个存储配置数据的节点
     *
     * @return
     * @throws Exception
     */
    private static CuratorFramework getClient() throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(SESSION_TIMEOUT)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
        if (client.checkExists().forPath(PATH) == null) {
            client.create()
                    .creatingParentsIfNeeded()
                    .withMode(CreateMode.EPHEMERAL)
                    .forPath(PATH, config.getBytes());
        }
        return client;
    }

    /**
     * 集群中的某个节点机器更改了配置信息：即发布了更新了数据
     *
     * @param client
     * @throws Exception
     */
    private static void publish(CuratorFramework client, String znode) throws Exception {

        System.out.println("节点[" + znode + "]更改了配置数据...");
        client.setData().forPath(PATH, "configuration".getBytes());
        countDownLatch.await();
    }

    /**
     * 集群中订阅的节点客户端（机器）获得最新的配置数据
     *
     * @param client
     * @param znode
     * @throws Exception
     */
    private static void subscribe(CuratorFramework client, String znode) throws Exception {
        // NodeCache监听ZooKeeper数据节点本身的变化
        final NodeCache cache = new NodeCache(client, PATH);
        // 设置为true：NodeCache在第一次启动的时候就立刻从ZooKeeper上读取节点数据并保存到Cache中
        cache.start(true);
        System.out.println("节点["+ znode +"]已订阅当前配置数据：" + new String(cache.getCurrentData().getData()));
        // 节点监听
        countDownLatch.countDown();
        cache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() {
                System.out.println("配置数据已发生改变, 节点[" + znode + "]读取当前新配置数据: " + new String(cache.getCurrentData().getData()));
            }
        });
    }
}
```

## Master选举

### 基本概念

　　（1）在一些读写分离的应用场景中，客户端写请求往往是由Master处理的，而另一些场景中，Master则常常负责处理一些复杂的逻辑，并将处理结果同步给集群中其它系统单元。比如一个广告投放系统后台与ZooKeeper交互，广告ID通常都是经过一系列海量数据处理中计算得到（非常消耗I/O和CPU资源的过程），那就可以只让集群中一台机器处理数据得到计算结果，之后就可以共享给整个集群中的其它所有客户端机器。

　　（2）利用ZooKeeper的特性：**利用ZooKeeper的强一致性，即能够很好地保证分布式高并发情况下节点的创建一定能够保证全局唯一性，ZooKeeper将会保证客户端无法重复创建一个已经存在的数据节点，也就是说如果多个客户端请求创建同一个节点，那么最终一定只有一个客户端请求能够创建成功，这个客户端就是Master，而其它客户端注在该节点上注册子节点Wacther，用于监控当前Master是否存活，如果当前Master挂了，那么其余客户端立马重新进行Master选举。**

　　（3）竞争成为Master角色之后，创建的子节点都是临时顺序节点，比如：_c_862cf0ce-6712-4aef-a91d-fc4c1044d104-lock-0000000001，并且序号是递增的。**需要注意的是这里有"lock"单词，这说明ZooKeeper这一特性，也可以运用于分布式锁。**

### 代码示例

 ```


import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.leader.LeaderSelector;
import org.apache.curator.framework.recipes.leader.LeaderSelectorListenerAdapter;
import org.apache.curator.retry.ExponentialBackoffRetry;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class ZooKeeper_Master {

    private static final String ADDRESS="xxx.xxx.xxx.xxx:2181";
    private static final int SESSION_TIMEOUT=5000;
    private static final String MASTER_PATH = "/master_path";
    private static final int CLIENT_COUNT = 5;

    private static RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);


    public static void main(String[] args) throws InterruptedException {

        ExecutorService service = Executors.newFixedThreadPool(CLIENT_COUNT);
        for (int i = 0; i < CLIENT_COUNT; i++) {
            final String index = String.valueOf(i);
            service.submit(() -> {
                masterSelect(index);
            });
        }
    }

    private static void  masterSelect(final String znode){
        // client成为master的次数统计
        AtomicInteger leaderCount = new AtomicInteger(1);
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(SESSION_TIMEOUT)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
        // 一旦执行完takeLeadership，就会重新进行选举
        LeaderSelector selector = new LeaderSelector(client, MASTER_PATH, new LeaderSelectorListenerAdapter() {
            @Override
            public void takeLeadership(CuratorFramework curatorFramework) throws Exception {
                System.out.println("节点["+ znode +"]成为master");
                System.out.println("节点["+ znode +"]已经成为master次数："+ leaderCount.getAndIncrement());
                // 睡眠5s模拟成为master后完成任务
                Thread.sleep(5000);
                System.out.println("节点["+ znode +"]释放master");
            }
        });
        // autoRequeue自动重新排队：使得上一次选举为master的节点还有可能再次成为master
        selector.autoRequeue();
        selector.start();
    }
}
 ```

## 分布式锁

### 基本概念

　　（1）对于排他锁：**ZooKeeper通过数据节点表示一个锁，例如/exclusive_lock/lock节点就可以定义一个锁，所有客户端都会调用create()接口，试图在/exclusive_lock下创建lock子节点，但是ZooKeeper的强一致性会保证所有客户端最终只有一个客户创建成功。也就可以认为获得了锁，其它线程Watcher监听子节点变化（等待释放锁，竞争获取资源）。**

​	 （2）对于共享锁：ZooKeeper同样可以通过数据节点表示一个锁，类似于/shared_lock/[Hostname]-请求类型（读/写）-序号的临时节点，比如/shared_lock/192.168.0.1-R-0000000000

### 排他锁（X）

这里主要讲讲分布式锁中的排他锁。排他锁（Exclusive Locks，简称X锁），又称为写锁或独占锁，是一种基本的锁类型。如果事务T1对数据对象O1加上了排他锁，那么在整个加锁期间，只允许T1对O1进行数据的读取和更新操作，其它任何事务都不能对O1进行任何类型的操作，直道T1释放了排他锁。

#### 定义锁

在ZooKeeper中，可以通过在ZooKeeper中创建一个数据节点来表示一个锁。比如，/exclusive_lock/lock节点（znode）就可以表示为一个锁。

#### 获取锁

在需要获取排他锁时，所有的客户端都会试图通过create()接口，在/exclusive_lock节点下创建临时的子节点/exclusive_lock/lock，但ZooKeeper的强一致性最终只会保证仅有一个客户端能创建成功，那么就认为该客户端获取了锁。同时，所有没有获取锁的客户端事务只能处于等待状态，这些处于等待状态的客户端事先可以在/exclusive_lock节点上注册一个子节点变更的Watcher监听，以便实时监听到子节点的变更情况。

#### 释放锁

 在“定义锁”部分，我们已经提到/exclusive_lock/lock是一个临时节点，因此在以下两种情况下可能释放锁。

- 当前获取锁的客户端发生宕机，那么ZooKeeper服务器上保存的临时性节点就会被删除；
- 正常执行完业务逻辑后，由客户端主动来将自己创建的临时节点删除。

无论什么情况下，临时节点/exclusive_lock/lock被移除，ZooKeeper都会通知在/exclusive_lock注册了子节点变更Watcher监听的客户端。这些客户端在接收到通知以后就会再次发起获取锁的操作，即重复“获取锁”过程。

主要流程图如下：

- 查看目标Node是否已经创建，已经创建，那么等待锁。
- 如果未创建，创建一个瞬时Node，表示已经占有锁。
- 如果创建失败，那么证明锁已经被其他线程占有了，那么同样等待锁。
- 当释放锁，或者当前Session超时的时候，节点被删除，唤醒之前等待锁的线程去争抢锁。

其实上面的实现有优点也有缺点：
 优点：
 实现比较简单，有通知机制，能提供较快的响应，有点类似reentrantlock的思想，对于节点删除失败的场景由Session超时保证节点能够删除掉。
 缺点：
 重量级，同时在大量锁的情况下会有“惊群”的问题。

“惊群”就是在一个节点删除的时候，大量对这个节点的删除动作有订阅Watcher的线程会进行回调，这对Zk集群是十分不利的。所以需要避免这种现象的发生。

解决“惊群”：

为了解决“惊群“问题，我们需要放弃订阅一个节点的策略，那么怎么做呢？

1. 我们将锁抽象成目录，多个线程在此目录下创建瞬时的顺序节点，因为Zk会为我们保证节点的顺序性，所以可以利用节点的顺序进行锁的判断。
2. 首先创建顺序节点，然后获取当前目录下最小的节点，判断最小节点是不是当前节点，如果是那么获取锁成功，如果不是那么获取锁失败。
3. 获取锁失败的节点获取当前节点上一个顺序节点，对此节点注册监听，当节点删除的时候通知当前节点。
4. 当unlock的时候删除节点之后会通知下一个节点。

### 代码示例

Curator提供的有四种锁，分别如下：

　　（1）InterProcessMutex：分布式可重入排它锁

　　（2）InterProcessSemaphoreMutex：分布式排它锁

　　（3）InterProcessReadWriteLock：分布式读写锁

　　（4）InterProcessMultiLock：将多个锁作为单个实体管理的容器

​                  InterProcessSemaphoreV2 信号量

主要是以InterProcessMutex为例，编写示例：

```


import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ZooKeeper_Lock {
    private static final String ADDRESS = "xxx.xxx.xxx.xxx:2181";
    private static final int SESSION_TIMEOUT = 5000;
    private static final String LOCK_PATH = "/lock_path";
    private static final int CLIENT_COUNT = 10;

    private static RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
    private static int resource = 0;

    public static void main(String[] args){
        ExecutorService service = Executors.newFixedThreadPool(CLIENT_COUNT);
        for (int i = 0; i < CLIENT_COUNT; i++) {
            final String index = String.valueOf(i);
            service.submit(() -> {
                distributedLock(index);
            });
        }
    }

    private static void distributedLock(final String znode) {
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(SESSION_TIMEOUT)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
        final InterProcessMutex lock = new InterProcessMutex(client, LOCK_PATH);
        try {
//            lock.acquire();
            System.out.println("客户端节点[" + znode + "]获取lock");
            System.out.println("客户端节点[" + znode + "]读取的资源为：" + String.valueOf(resource));
            resource ++;
//            lock.release();
            System.out.println("客户端节点[" + znode + "]释放lock");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 分布式可重入排它锁(InterProcessMutex)

此锁可以重入，但是重入几次需要释放几次。

```
@Test
    public void sharedReentrantLock() throws Exception {
        // 创建共享锁
        final InterProcessLock lock = new InterProcessMutex(client, lockPath);
        // lock2 用于模拟其他客户端
        final InterProcessLock lock2 = new InterProcessMutex(client2, lockPath);

        final CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock.acquire();
                    System.out.println("1获取锁===============");
                    // 测试锁重入
                    lock.acquire();
                    System.out.println("1再次获取锁===============");
                    Thread.sleep(5 * 1000);
                    lock.release();
                    System.out.println("1释放锁===============");
                    lock.release();
                    System.out.println("1再次释放锁===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock2.acquire();
                    System.out.println("2获取锁===============");
                    // 测试锁重入
                    lock2.acquire();
                    System.out.println("2再次获取锁===============");
                    Thread.sleep(5 * 1000);
                    lock2.release();
                    System.out.println("2释放锁===============");
                    lock2.release();
                    System.out.println("2再次释放锁===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        countDownLatch.await();
    }
```

**原理:**

　　InterProcessMutex通过在zookeeper的某路径节点下创建临时序列节点来实现分布式锁，即每个线程（跨进程的线程）获取同一把锁前，都需要在同样的路径下创建一个节点，节点名字由uuid + 递增序列组成。而通过对比自身的序列数是否在所有子节点的第一位，来判断是否成功获取到了锁。当获取锁失败时，它会添加watcher来监听前一个节点的变动情况，然后进行等待状态。直到watcher的事件生效将自己唤醒，或者超时时间异常返回。

#### 分布式排它锁(InterProcessSemaphoreMutex)

InterProcessSemaphoreMutex是一种不可重入的互斥锁，也就意味着即使是同一个线程也无法在持有锁的情况下再次获得锁，所以需要注意，不可重入的锁很容易在一些情况导致死锁。

```
@Test
    public void sharedLock() throws Exception {
        // 创建共享锁
        final InterProcessLock lock = new InterProcessSemaphoreMutex(client, lockPath);
        // lock2 用于模拟其他客户端
        final InterProcessLock lock2 = new InterProcessSemaphoreMutex(client2, lockPath);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock.acquire();
                    System.out.println("1获取锁===============");
                    // 测试锁重入
                    Thread.sleep(5 * 1000);
                    lock.release();
                    System.out.println("1释放锁===============");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock2.acquire();
                    System.out.println("2获取锁===============");
                    Thread.sleep(5 * 1000);
                    lock2.release();
                    System.out.println("2释放锁===============");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        Thread.sleep(20 * 1000);
    }
```

#### 分布式读写锁(InterProcessReadWriteLock)

读锁和读锁不互斥，只要有写锁就互斥。

```
@Test
    public void sharedReentrantReadWriteLock() throws Exception {
        // 创建共享可重入读写锁
        final InterProcessReadWriteLock locl1 = new InterProcessReadWriteLock(client, lockPath);
        // lock2 用于模拟其他客户端
        final InterProcessReadWriteLock lock2 = new InterProcessReadWriteLock(client2, lockPath);

        // 获取读写锁(使用 InterProcessMutex 实现, 所以是可以重入的)
        final InterProcessLock readLock = locl1.readLock();
        final InterProcessLock readLockw = lock2.readLock();

        final CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    readLock.acquire();
                    System.out.println("1获取读锁===============");
                    // 测试锁重入
                    readLock.acquire();
                    System.out.println("1再次获取读锁===============");
                    Thread.sleep(5 * 1000);
                    readLock.release();
                    System.out.println("1释放读锁===============");
                    readLock.release();
                    System.out.println("1再次释放读锁===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    Thread.sleep(500);
                    readLockw.acquire();
                    System.out.println("2获取读锁===============");
                    // 测试锁重入
                    readLockw.acquire();
                    System.out.println("2再次获取读锁==============");
                    Thread.sleep(5 * 1000);
                    readLockw.release();
                    System.out.println("2释放读锁===============");
                    readLockw.release();
                    System.out.println("2再次释放读锁===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        countDownLatch.await();
    }
```

#### 共享信号量（InterProcessSemaphoreV2）

```
@Test
    public void semaphore() throws Exception {
        // 创建一个信号量, Curator 以公平锁的方式进行实现
        final InterProcessSemaphoreV2 semaphore = new InterProcessSemaphoreV2(client, lockPath, 1);

        final CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    // 获取一个许可
                    Lease lease = semaphore.acquire();
                    logger.info("1获取读信号量===============");
                    Thread.sleep(5 * 1000);
                    semaphore.returnLease(lease);
                    logger.info("1释放读信号量===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    // 获取一个许可
                    Lease lease = semaphore.acquire();
                    logger.info("2获取读信号量===============");
                    Thread.sleep(5 * 1000);
                    semaphore.returnLease(lease);
                    logger.info("2释放读信号量===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        countDownLatch.await();
    }
```

**当然可以一次获取多个信号量:**

```
@Test
    public void semaphore() throws Exception {
        // 创建一个信号量, Curator 以公平锁的方式进行实现
        final InterProcessSemaphoreV2 semaphore = new InterProcessSemaphoreV2(client, lockPath, 3);

        final CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    // 获取2个许可
                    Collection<Lease> acquire = semaphore.acquire(2);
                    logger.info("1获取读信号量===============");
                    Thread.sleep(5 * 1000);
                    semaphore.returnAll(acquire);
                    logger.info("1释放读信号量===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    // 获取1个许可
                    Collection<Lease> acquire = semaphore.acquire(1);
                    logger.info("2获取读信号量===============");
                    Thread.sleep(5 * 1000);
                    semaphore.returnAll(acquire);
                    logger.info("2释放读信号量===============");

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        countDownLatch.await();
    }
```

#### 多重共享锁（InterProcessMultiLock）

```
@Test
    public void multiLock() throws Exception {
        // 可重入锁
        final InterProcessLock interProcessLock1 = new InterProcessMutex(client, lockPath);
        // 不可重入锁
        final InterProcessLock interProcessLock2 = new InterProcessSemaphoreMutex(client2, lockPath);
        // 创建多重锁对象
        final InterProcessLock lock = new InterProcessMultiLock(Arrays.asList(interProcessLock1, interProcessLock2));

        final CountDownLatch countDownLatch = new CountDownLatch(1);

        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    // 获取参数集合中的所有锁
                    lock.acquire();
                    // 因为存在一个不可重入锁, 所以整个 InterProcessMultiLock 不可重入
                    System.out.println(lock.acquire(2, TimeUnit.SECONDS));
                    // interProcessLock1 是可重入锁, 所以可以继续获取锁
                    System.out.println(interProcessLock1.acquire(2, TimeUnit.SECONDS));
                    // interProcessLock2 是不可重入锁, 所以获取锁失败
                    System.out.println(interProcessLock2.acquire(2, TimeUnit.SECONDS));

                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        countDownLatch.await();
    }
```

## 命名服务

在分布式系统中，通常需要一个全局唯一的名字，如生成全局唯一的订单号等，ZooKeeper 可以通过顺序节点的特性来生成全局唯一 ID，从而可以对分布式系统提供命名服务。

```
   private String createSeqNode(String pathPefix) {
      try {
            // 创建一个 ZNode 顺序节点
            String destPath = client.create()
                    .creatingParentsIfNeeded()
                    .withMode(CreateMode.*EPHEMERAL_SEQUENTIAL*)
//避免zookeeper的顺序节点暴增，可以删除创建的顺序节点
                    .forPath(pathPefix);
            return destPath;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```

节点创建完成后，会返回节点的完整的层次路径，生成的序号，放置在路径的末尾。一般为10位数字字符。

通过截取路径末尾的数字，作为新生成的ID。截取数字的代码如下：

```
public String makeId(String nodeName) {
    String str = createSeqNode(nodeName);
    if (null == str) {
        return null;
    }
    int index = str.lastIndexOf(nodeName);
    if (index >= 0) {
        index += nodeName.length();
        return index <= str.length() ? str.substring(index) : "";
    }
    return str;
}
```

## 集群管理

ZooKeeper 还能解决大多数分布式系统中的问题：

- 如可以通过创建临时节点来建立心跳检测机制。如果分布式系统的某个服务节点宕机了，则其持有的会话会超时，此时该临时节点会被删除，相应的监听事件就会被触发。
- 分布式系统的每个服务节点还可以将自己的节点状态写入临时节点，从而完成状态报告或节点工作进度汇报。
- 通过数据的订阅和发布功能，ZooKeeper 还能对分布式系统进行模块的解耦和任务的调度。
- 通过监听机制，还能对分布式系统的服务节点进行动态上下线，从而实现服务的动态扩容。

## 队列管理

ZooKeeper 可以处理两种类型的队列：

- 当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列。，在约定目录下创建临时目录节点，监听节点数目是否是我们要求的数目。创建一个父目录 /synchronizing，每个成员都监控标志（Set Watch）位目录 /synchronizing/start 是否存在，然后每个成员都加入这个队列，加入队列的方式就是创建 /synchronizing/member_i 的临时目录节点，然后每个成员获取 / synchronizing 目录的所有目录节点，也就是 member_i。判断 i 的值是否已经是成员的个数，如果小于成员个数等待 /synchronizing/start 的出现，如果已经相等就创建 /synchronizing/start。
- 队列按照 FIFO 方式进行入队和出队操作，例如实现生产者和消费者模型。分布式锁服务中的控制时序场景基本原理一致，入列有编号，出列按编号。在特定的目录下创建PERSISTENT_SEQUENTIAL节点，创建成功时Watcher通知等待的队列，队列删除序列号最小的节点用以消费。此场景下Zookeeper的znode用于消息存储，znode存储的数据就是消息队列中的消息内容，SEQUENTIAL序列号就是消息的编号，按序取出即可。由于创建的节点是持久化的，所以不必担心队列消息的丢失问题。

Curator也提供ZK Recipe的分布式队列实现。利用ZK的 PERSISTENTSEQUENTIAL节点，可以保证放入到队列中的项目是按照顺序排队的。如果单一的消费者从队列中取数据，那么它是先入先出的，这也是队列的特点。如果你严格要求顺序，你就得使用单一的消费者，可以使用leader选举只让leader作为唯一的消费者。但是，根据Netflix的Curator作者所说，ZooKeeper真心不适合做Queue，或者说ZK没有实现一个好的Queue，详细内容可以看 Tech Note 4，原因有五：

- ZK有1MB 的传输限制。实践中ZNode必须相对较小，而队列包含成千上万的消息，非常的大
- 如果有很多节点，ZK启动时相当的慢。而使用queue会导致好多ZNode。你需要显著增大 initLimit 和 syncLimit
- ZNode很大的时候很难清理。Netflix不得不创建了一个专门的程序做这事
- 当很大量的包含成千上万的子节点的ZNode时，ZK的性能变得不好
- ZK的数据库完全放在内存中。大量的Queue意味着会占用很多的内存空间

尽管如此，Curator还是创建了各种Queue的实现。如果Queue的数据量不太多，数据量不太大的情况下，酌情考虑，还是可以使用的。

### DistributedQueue

**DistributedQueue介绍**

DistributedQueue是最普通的一种队列。 它设计以下四个类：

- QueueBuilder - 创建队列使用QueueBuilder,它也是其它队列的创建类
- QueueConsumer - 队列中的消息消费者接口
- QueueSerializer - 队列消息序列化和反序列化接口，提供了对队列中的对象的序列化和反序列化
- DistributedQueue - 队列实现类

  QueueConsumer是消费者，它可以接收队列的数据。处理队列中的数据的代码逻辑可以放在QueueConsumer.consumeMessage()中。

  正常情况下先将消息从队列中移除，再交给消费者消费。但这是两个步骤，不是原子的。可以调用Builder的lockPath()消费者加锁，当消费者消费数据时持有锁，这样其它消费者不能消费此消息。如果消费失败或者进程死掉，消息可以交给其它进程。这会带来一点性能的损失。最好还是单消费者模式使用队列。

**示例程序**

```typescript
public class DistributedQueueExample
{
    private static final String PATH = "/example/queue";
    public static void main(String[] args) throws Exception
    {
        CuratorFramework clientA = CuratorFrameworkFactory.newClient("127.0.0.1:2181", new ExponentialBackoffRetry(1000, 3));
        clientA.start();
        CuratorFramework clientB = CuratorFrameworkFactory.newClient("127.0.0.1:2181", new ExponentialBackoffRetry(1000, 3));
        clientB.start();

        DistributedQueue<String> queueA = null;
        QueueBuilder<String> builderA = QueueBuilder.builder(clientA, createQueueConsumer("A"), createQueueSerializer(), PATH);
        queueA = builderA.buildQueue();
        queueA.start();
        
        DistributedQueue<String> queueB = null;
        QueueBuilder<String> builderB = QueueBuilder.builder(clientB, createQueueConsumer("B"), createQueueSerializer(), PATH);
        queueB = builderB.buildQueue();
        queueB.start();

        for (int i = 0; i < 100; i++)
        {
            queueA.put(" test-A-" + i);
            Thread.sleep(10);
            queueB.put(" test-B-" + i);
        }

        Thread.sleep(1000 * 10);// 等待消息消费完成
        queueB.close();
        queueA.close();
        clientB.close();
        clientA.close();
        System.out.println("OK!");
    }

    /** 队列消息序列化实现类 */
    private static QueueSerializer<String> createQueueSerializer()
    {
        return new QueueSerializer<String>()
        {
            @Override
            public byte[] serialize(String item)
            {
                return item.getBytes();
            }
            @Override
            public String deserialize(byte[] bytes)
            {
                return new String(bytes);
            }
        };
    }

    /** 定义队列消费者 */
    private static QueueConsumer<String> createQueueConsumer(final String name)
    {
        return new QueueConsumer<String>()
        {
            @Override
            public void stateChanged(CuratorFramework client, ConnectionState newState)
            {
                System.out.println("连接状态改变: " + newState.name());
            }
            @Override
            public void consumeMessage(String message) throws Exception
            {
                System.out.println("消费消息(" + name + "): " + message);
            }
        };
    }
}
```

以上程序创建两个client(A和B)，它们在同一路径上创建队列(A和B)，同时发消息、同时消费消息。

### DistributedIdQueue

  DistributedIdQueue和上面的队列类似，但是可以为队列中的每一个元素设置一个ID。可以通过ID把队列中任意的元素移除。

**DistributedIdQueue结束**

DistributedIdQueue的使用与上面队列的区别是：

- 通过下面方法创建：builder.buildIdQueue()
- 放入元素时：queue.put(aMessage, messageId);
- 移除元素时：int numberRemoved = queue.remove(messageId);

**示例程序**

在这个例子中，有些元素还没有被消费者消费时就移除了，这样消费者不会收到删除的消息。(此示例是根据上面例子修改而来)

```cpp
public class DistributedIdQueueExample
{
    private static final String PATH = "/example/queue";

    public static void main(String[] args) throws Exception
    {
        CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2181", new ExponentialBackoffRetry(1000, 3));
        client.start();
        
        DistributedIdQueue<String> queue = null;
        QueueConsumer<String> consumer = createQueueConsumer("A");
        QueueBuilder<String> builder = QueueBuilder.builder(client, consumer, createQueueSerializer(), PATH);
        queue = builder.buildIdQueue();
        queue.start();

        for (int i = 0; i < 10; i++)
        {
            queue.put(" test-" + i, "Id" + i);
            Thread.sleep((long) (50 * Math.random()));
            queue.remove("Id" + i);
        }

        Thread.sleep(1000 * 3);
        queue.close();
        client.close();
        System.out.println("OK!");
    }
......
}
```

### DistributedPriorityQueue

  优先级队列对队列中的元素按照优先级进行排序。Priority越小，元素月靠前，越先被消费掉。

**DistributedPriorityQueue介绍**

  通过builder.buildPriorityQueue(minItemsBeforeRefresh)方法创建。

  当优先级队列得到元素增删消息时，它会暂停处理当前的元素队列，然后刷新队列。minItemsBeforeRefresh指定刷新前当前活动的队列的最小数量。主要设置你的程序可以容忍的不排序的最小值。

  放入队列时需要指定优先级：queue.put(aMessage, priority);

**示例程序**

```cpp
public class DistributedPriorityQueueExample
{
    private static final String PATH = "/example/queue";
    public static void main(String[] args) throws Exception
    {
        CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2181", new ExponentialBackoffRetry(1000, 3));
        client.start();
        DistributedPriorityQueue<String> queue = null;
        QueueConsumer<String> consumer = createQueueConsumer("A");
        QueueBuilder<String> builder = QueueBuilder.builder(client, consumer, createQueueSerializer(), PATH);
        queue = builder.buildPriorityQueue(0);
        queue.start();
        for (int i = 0; i < 5; i++)
        {
            int priority = (int) (Math.random() * 100);
            System.out.println("test-" + i + " 优先级:" + priority);
            queue.put("test-" + i, priority);
            Thread.sleep((long) (50 * Math.random()));
        }
        Thread.sleep(1000 * 2);
        queue.close();
        client.close();
    }
......
}
```

### DistributedDelayQueue

  JDK中也有DelayQueue，不知道你是否熟悉。DistributedDelayQueue也提供了类似的功能，元素有个delay值，消费者隔一段时间才能收到元素。

**DistributedDelayQueue介绍**

放入元素时可以指定delayUntilEpoch：queue.put(aMessage, delayUntilEpoch);

**注意：**delayUntilEpoch不是离现在的一个时间间隔，比如20毫秒，而是未来的一个时间戳，如 System.currentTimeMillis() + 10秒。如果delayUntilEpoch的时间已经过去，消息会立刻被消费者接收。

**示例程序**

```cpp
public class DistributedDelayQueueExample
{
    private static final String PATH = "/example/queue";
    public static void main(String[] args) throws Exception
    {
        CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2181", new ExponentialBackoffRetry(1000, 3));
        client.start();
        DistributedDelayQueue<String> queue = null;
        QueueConsumer<String> consumer = createQueueConsumer("A");
        QueueBuilder<String> builder = QueueBuilder.builder(client, consumer, createQueueSerializer(), PATH);
        queue = builder.buildDelayQueue();
        queue.start();
        for (int i = 0; i < 10; i++)
        {
            queue.put("test-" + i, System.currentTimeMillis() + 3000);
        }
        System.out.println("put 完成！");
        Thread.sleep(1000 * 5);
        queue.close();
        client.close();
        System.out.println("OK!");
    }
......
}
```

### SimpleDistributedQueue

  前面虽然实现了各种队列，但是你注意到没有，这些队列并没有实现类似JDK一样的接口。SimpleDistributedQueue提供了和JDK一致性的接口(但是没有实现Queue接口)。

**SimpleDistributedQueue介绍**

SimpleDistributedQueue常用方法：

```java
// 创建
public SimpleDistributedQueue(CuratorFramework client, String path)

// 增加元素
public boolean offer(byte[] data) throws Exception

// 删除元素
public byte[] take() throws Exception

// 另外还提供了其它方法
public byte[] peek() throws Exception
public byte[] poll(long timeout, TimeUnit unit) throws Exception
public byte[] poll() throws Exception
public byte[] remove() throws Exception
public byte[] element() throws Exception
```

没有add方法，多了take方法。take方法在成功返回之前会被阻塞。而poll在队列为空时直接返回null。

# 参考资料

**官方**

- [ZooKeeper 官网](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fzookeeper.apache.org%2F)
- [ZooKeeper 官方文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fcwiki.apache.org%2Fconfluence%2Fdisplay%2FZOOKEEPER)
- [ZooKeeper Github](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fapache%2Fzookeeper)

**书籍**

- [《Hadoop 权威指南（第四版）》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fitem.jd.com%2F12109713.html)
- [《从 Paxos 到 Zookeeper 分布式一致性原理与实践》](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fitem.jd.com%2F11622772.html)

**文章**

- [分布式服务框架 ZooKeeper -- 管理分布式环境中的数据](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fopensource%2Fos-cn-zookeeper%2Findex.html)
- [ZooKeeper 的功能以及工作原理](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.cnblogs.com%2Ffelixzh%2Fp%2F5869212.html)
- [ZooKeeper 简介及核心概念](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fheibaiying%2FBigData-Notes%2Fblob%2Fmaster%2Fnotes%2FZooKeeper%E7%AE%80%E4%BB%8B%E5%8F%8A%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.md)
- [详解分布式协调服务 ZooKeeper](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdraveness.me%2Fzookeeper-chubby)
- [深入浅出 Zookeeper（一） Zookeeper 架构及 FastLeaderElection 机制](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.jasongj.com%2Fzookeeper%2Ffastleaderelection%2F)

# Zookeeper源码解析

　　关于Zookeeper，目前普遍的应用场景基本作为服务注册中心，用于服务发现。但这只是Zookeeper的一个的功能，根据Apache的官方概述：“The Apache ZooKeeper system for distributed coordination is a high-performance service for building distributed applications.” Zookeeper是一个用于构建分布式应用的coordination, 并且为高性能的。Zookeeper借助于它内部的节点结构和监听机制，能用于很大部分的分布式协调场景。配置管理、命名服务、分布式锁、服务发现和发布订阅等等，这些场景在Zookeeper中基本使用其节点的“变更+通知”来实现。因为分布式的重点在于通信，通信的作用也就是协调。

　　Zookeeper由Java语言编写（也有C语言的Api实现）,对于其原理，算是Paxos算法的实现，包含了Leader、Follower、Proposal等角色和选举之类的一些概念，但于Paxos还有一些不同（ZAB协议）。

## 源码获取

Zookeeper源码可以从Github（https://github.com/apache/zookeeper）上clone下来；

也可从Zookeeper官网（Apache）https://zookeeper.apache.org/releases.html上获取。

Zookeeper在3.5.5之前使用的是Ant构建，在3.5.5开始使用的是Maven构建。

## 概要框架设计

![Zookeeper架构图](png\zookeeper\Zookeeper架构图.png)

Zookeeper整体架构主要分为数据的存储，消息，leader选举和数据同步这几个模块。

leader选举主要是在集群处于混沌的状态下，从集群peer的提议中选择集群的leader，其他为follower或observer，维护集群peer的统一视图，保证整个集群的数据一致性，如果在leader选举成功后，存在follower日志落后的情况，则将事务日志同步给follower。

针对消息模块，peer之间的通信包需要序列化和反序列才能发送和处理，具体的消息处理由集群相应角色的消息处理器链来处理。

针对客户单的节点的创建，数据修改等操作，将会先写到内存数据库，如果有提交请求，则将数据写到事务日志，同时Zookeeper会定时将内存数据库写到快照日志，以防止没有提交的日志，在宕机的情况下丢失。

数据同步模块将leader的事务日志同步给Follower，保证整个集群数据的一致性。

## 工程结构

其中src中包含了C和Java源码，本次忽略C的Api。conf下为配置文件，也就是Zookeeper启动的配置文件。bin为Zookeeper启动脚本（server/client）。

　　org.apache.jute为Zookeeper的通信协议和序列化相关的组件，其通信协议基于TCP协议，它提供了Record接口用于序列化和反序列化，OutputArchive/InputArchive接口.

　　org.apache.zookeeper下为Zookeeper核心代码。包含了核心的业务实现。

## Zookeeper启动

### main方法入口

zookeeper一般使用命令工具启动，启动主要就是初始化所有组件，让server可以接收并处理来自client的请求。

我们一般使用命令行工具来部署zk server，zkServer.sh，这个脚本用来启动停止server，通过不同的参数和选项来达到不同的功能。该脚本最后会通过Java执行下面的main方法

```java
org.apache.zookeeper.server.quorum.QuorumPeerMain#main
```

不管单机还是集群都是使用`zkServer.sh`这个脚本来启动，只是参数不同，所以main方法入口也是一样的。所以这个入口方法主要是根据不同的入参判断是集群启动还是单机启动。

```
ZOOMAIN="org.apache.zookeeper.server.quorum.QuorumPeerMain"
#.......
nohup "$JAVA" $ZOO_DATADIR_AUTOCREATE "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" \
    "-Dzookeeper.log.file=${ZOO_LOG_FILE}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
    -XX:+HeapDumpOnOutOfMemoryError -XX:OnOutOfMemoryError='kill -9 %p' \
    -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
    if [ $? -eq 0 ]  #.......
```

该main方法主要做了以下几件事

1. 解析配置，如果传入的是配置文件(参数只有一个)，解析配置文件并初始化QuorumPeerConfig
2. 启动清理文件的线程
3. 判断是单机还是集群
   1. 集群：只有一个参数，并且配置了多个server
   2. 单机：上面的条件不满足，一般在启动的使用了以下两种配置的一种
      1. 使用的是文件配置，但是没有配置多台server
      2. 命令行配置多个（2-4）参数：port dataDir [tickTime] [maxClientCnxns]

### Zookeeper启动流程

![Zookeeper启动流程](png\zookeeper\Zookeeper启动流程.png)

Zookeeper启动时，首先解析配置文件，根据配置文件选择启动单例还是集群模式。集群模式启动，首先从磁盘中加载数据到内存数据树DataTree， 并添加committed交易日志到DataTree中。然后启动ServerCnxnFactory,监听客户端的请求。实际上是启动了一个基于Netty服务，客户端发送的数据，交由NettyServerCnxn处理，NettyServerCnxn数据包的处理，实际委托给ZooKeeperServer。

### 配置解析

配置解析主要有两种情况

1. 使用配置文件
2. 使用命令行参数

#### 使用配置文件

使用配置文件的时候是使用`QuorumPeerConfig`来解析配置的

1. 先校验文件的合法性
2. 配置文件是使用Java的properties形式写的，所以可以通过Properties.load来解析
3. 将解析出来的key、value赋值给对应的配置

#### 使用命令行参数

直接在命令指定对应的配置，这种情况只有在单机的时候才会使用，包含以下几个参数

- port，必填，sever监听的端口
- dataDir，必填，数据所在的目录
- tickTime，选填
- maxClientCnxns，选填，最多可处理的客户端连接数

```
public static void main(String[] args) {
        QuorumPeerMain main = new QuorumPeerMain();
        try {
            main.initializeAndRun(args);
        } catch (IllegalArgumentException e) {
            LOG.error("Invalid arguments, exiting abnormally", e);
            LOG.info(USAGE);
            System.err.println(USAGE);
            System.exit(2);
        } catch (ConfigException e) {
            LOG.error("Invalid config, exiting abnormally", e);
            System.err.println("Invalid config, exiting abnormally");
            System.exit(2);
        } catch (DatadirException e) {
            LOG.error("Unable to access datadir, exiting abnormally", e);
            System.err.println("Unable to access datadir, exiting abnormally");
            System.exit(3);
        } catch (AdminServerException e) {
            LOG.error("Unable to start AdminServer, exiting abnormally", e);
            System.err.println("Unable to start AdminServer, exiting abnormally");
            System.exit(4);
        } catch (Exception e) {
            LOG.error("Unexpected exception, exiting abnormally", e);
            System.exit(1);
        }
        LOG.info("Exiting normally");
        System.exit(0);
    }
```

QuorumPeerMain.main()接受至少一个参数，一般就一个参数，参数为zoo.cfg文件路径。main方法中没有很多的业务代码，实例化了一个QuorumPeerMain 对象，然后main.initializeAndRun(args)进行了实例化

```
protected void initializeAndRun(String[] args)
        throws ConfigException, IOException, AdminServerException
    {
        QuorumPeerConfig config = new QuorumPeerConfig();
        if (args.length == 1) {
            config.parse(args[0]);
        }

        // Start and schedule the the purge task
        DatadirCleanupManager purgeMgr = new DatadirCleanupManager(config
                .getDataDir(), config.getDataLogDir(), config
                .getSnapRetainCount(), config.getPurgeInterval());
        purgeMgr.start();

        // 当配置了多节点信息，return quorumVerifier!=null && (!standaloneEnabled || quorumVerifier.getVotingMembers().size() > 1);
        if (args.length == 1 && config.isDistributed()) {
            // 集群模式
            runFromConfig(config);
        } else {
            LOG.warn("Either no config or no quorum defined in config, running "
                    + " in standalone mode");
            // there is only server in the quorum -- run as standalone
            // 单机模式
            ZooKeeperServerMain.main(args);
        }
    }
```

initializeAndRun方法则通过实例化QuorumPeerConfig对象，通过parseProperties()来解析zoo.cfg文件中的配置，QuorumPeerConfig包含了Zookeeper整个应用的配置属性。接着开启一个DatadirCleanupManager对象来开启一个Timer用于清除并创建管理新的DataDir相关的数据。

　　最后进行程序的启动，因为Zookeeper分为单机和集群模式，所以分为两种不同的启动方式，当zoo.cfg文件中配置了standaloneEnabled=true为单机模式，如果配置server.0,server.1......集群节点，则为集群模式.

### 单机模式启动

当配置了standaloneEnabled=true 或者没有配置集群节点（sever.*）时，Zookeeper使用单机环境启动。单机环境启动入口为ZooKeeperServerMain类，ZooKeeperServerMain类中持有ServerCnxnFactory、ContainerManager和AdminServer对象;

```
public class ZooKeeperServerMain {
    /*.............*/
    // ZooKeeper server supports two kinds of connection: unencrypted and encrypted.
    private ServerCnxnFactory cnxnFactory;
    private ServerCnxnFactory secureCnxnFactory;
    private ContainerManager containerManager;

    private AdminServer adminServer;
    /*.............*/
}
```

ServerCnxnFactory为Zookeeper中的核心组件，用于网络通信IO的实现和管理客户端连接，Zookeeper内部提供了两种实现，一种是基于JDK的NIO实现，一种是基于netty的实现。

​     **ContainerManager**类，用于管理维护Zookeeper中节点Znode的信息，管理zkDatabase；

 　**AdminServer**是一个Jetty服务，默认开启8080端口，用于提供Zookeeper的信息的查询接口。该功能从3.5的版本开始。

 　**ZooKeeperServerMain**的main方法中同QuorumPeerMain中一致，先实例化本身的对象，再进行init，加载配置文件，然后启动。

#### 组件启动

zookeeper包含的主要组件有

- FileTxnSnapLog：管理FileTxLog和FileSnap
- ZooKeeperServer：维护一个处理器链表processor chain
- NIOServerCnxnFactory：管理来自客户端的连接
- Jetty，用来通过http管理zk

zookeeper维护了自己的数据结构和物理文件，而且要接收并处理client发送来的网络请求，所以在zookeeper启动的时候，要做好下面的准备工作

1. 初始化FileTxnSnapLog，创建了FileTxnLog实例和FIleSnap实例，并保存刚启动时候DataTree的snapshot
2. 初始化ZooKeeperServer 对象；
3. 实例化CountDownLatch线程计数器对象，在程序启动后，执行shutdownLatch.await();用于挂起主程序，并监听Zookeeper运行状态。
4. 创建adminServer（Jetty）服务并开启。
5. 创建ServerCnxnFactory对象，cnxnFactory = ServerCnxnFactory.createFactory(); Zookeeper默认使用NIOServerCnxnFactory来实现网络通信IO。
   1. 从解析出的配置中配置NIOServerCnxnFactory
   2. 初始化网络连接管理类:NIOServerCnxnFactory
      1. 初始化：WorkerService：用来业务处理的线程池
      2. 线程启动：
         SelectorThread（有多个）：处理网络请求，write和read
         AcceptThread：用来接收连接请求，建立连接，zk也支持使用reactor多线程，accept线程用来建立连接，selector线程用来处理read、write
         ConnectionExpirerThread：关闭超时的连接，所有的session都放在`org.apache.zookeeper.server.ExpiryQueue#expiryMap`里面维护，这个线程不断从里面拿出超时的连接关闭
   3. 启动ZookeeperServer，主要是用来创建SessionTrackerImpl，这个类是用来管理session的

#### 	启动单机模式

```
// 解析单机模式的配置对象，并启动单机模式
    protected void initializeAndRun(String[] args)
        throws ConfigException, IOException, AdminServerException
    {
        try {

            //注册jmx
           // JMX的全称为Java Management Extensions.是管理Java的一种扩展。
           // 这种机制可以方便的管理、监控正在运行中的Java程序。常用于管理线程，内存，日志Level，服务重启，系统环境等
            ManagedUtil.registerLog4jMBeans();
        } catch (JMException e) {
            LOG.warn("Unable to register log4j JMX control", e);
        }

        // 创建服务配置对象
        ServerConfig config = new ServerConfig();

        //如果入参只有一个，则认为是配置文件的路径
        if (args.length == 1) {
            // 解析配置文件
            config.parse(args[0]);
        } else {
            // 参数有多个，解析参数
            config.parse(args);
        }

        // 根据配置运行服务
        runFromConfig(config);
    }
```

服务启动

```
public void runFromConfig(ServerConfig config)
            throws IOException, AdminServerException {
        LOG.info("Starting server");
        FileTxnSnapLog txnLog = null;
        try {
            // Note that this thread isn't going to be doing anything else,
            // so rather than spawning another thread, we will just call
            // run() in this thread.
            // create a file logger url from the command line args
            //初始化日志文件
            txnLog = new FileTxnSnapLog(config.dataLogDir, config.dataDir);

           // 初始化zkServer对象
            final ZooKeeperServer zkServer = new ZooKeeperServer(txnLog,
                    config.tickTime, config.minSessionTimeout, config.maxSessionTimeout, null);

            // 服务结束钩子，用于知道服务器错误或关闭状态更改。
            final CountDownLatch shutdownLatch = new CountDownLatch(1);
            zkServer.registerServerShutdownHandler(
                    new ZooKeeperServerShutdownHandler(shutdownLatch));


            // Start Admin server
            // 创建admin服务，用于接收请求(创建jetty服务)
            adminServer = AdminServerFactory.createAdminServer();
            // 设置zookeeper服务
            adminServer.setZooKeeperServer(zkServer);
            // AdminServer是3.5.0之后支持的特性,启动了一个jettyserver,默认端口是8080,访问此端口可以获取Zookeeper运行时的相关信息
            adminServer.start();

            boolean needStartZKServer = true;


            //---启动ZooKeeperServer
            //判断配置文件中 clientportAddress是否为null
            if (config.getClientPortAddress() != null) {
                //ServerCnxnFactory是Zookeeper中的重要组件,负责处理客户端与服务器的连接
                //初始化server端IO对象，默认是NIOServerCnxnFactory:Java原生NIO处理网络IO事件
                cnxnFactory = ServerCnxnFactory.createFactory();

                //初始化配置信息
                cnxnFactory.configure(config.getClientPortAddress(), config.getMaxClientCnxns(), false);

                //启动服务:此方法除了启动ServerCnxnFactory,还会启动ZooKeeper
                cnxnFactory.startup(zkServer);
                // zkServer has been started. So we don't need to start it again in secureCnxnFactory.
                needStartZKServer = false;
            }
            if (config.getSecureClientPortAddress() != null) {
                secureCnxnFactory = ServerCnxnFactory.createFactory();
                secureCnxnFactory.configure(config.getSecureClientPortAddress(), config.getMaxClientCnxns(), true);
                secureCnxnFactory.startup(zkServer, needStartZKServer);
            }

            // 定时清除容器节点
            //container ZNodes是3.6版本之后新增的节点类型，Container类型的节点会在它没有子节点时
            // 被删除（新创建的Container节点除外），该类就是用来周期性的进行检查清理工作
            containerManager = new ContainerManager(zkServer.getZKDatabase(), zkServer.firstProcessor,
                    Integer.getInteger("znode.container.checkIntervalMs", (int) TimeUnit.MINUTES.toMillis(1)),
                    Integer.getInteger("znode.container.maxPerMinute", 10000)
            );
            containerManager.start();

            // Watch status of ZooKeeper server. It will do a graceful shutdown
            // if the server is not running or hits an internal error.

            // ZooKeeperServerShutdownHandler处理逻辑，只有在服务运行不正常的情况下，才会往下执行
            shutdownLatch.await();

            // 关闭服务
            shutdown();

            if (cnxnFactory != null) {
                cnxnFactory.join();
            }
            if (secureCnxnFactory != null) {
                secureCnxnFactory.join();
            }
            if (zkServer.canShutdown()) {
                zkServer.shutdown(true);
            }
        } catch (InterruptedException e) {
            // warn, but generally this is ok
            LOG.warn("Server interrupted", e);
        } finally {
            if (txnLog != null) {
                txnLog.close();
            }
        }
    }
```

Zookeeper中 ServerCnxnFactory默认采用了NIOServerCnxnFactory来实现，也可以通过配置系统属性zookeeper.serverCnxnFactory 来设置使用Netty实现；

```
static public ServerCnxnFactory createFactory() throws IOException {
        String serverCnxnFactoryName =
            System.getProperty(ZOOKEEPER_SERVER_CNXN_FACTORY);
        if (serverCnxnFactoryName == null) {
            //如果未指定实现类，默认使用NIOServerCnxnFactory
            serverCnxnFactoryName = NIOServerCnxnFactory.class.getName();
        }
        try {
            ServerCnxnFactory serverCnxnFactory = (ServerCnxnFactory) Class.forName(serverCnxnFactoryName)
                    .getDeclaredConstructor().newInstance();
            LOG.info("Using {} as server connection factory", serverCnxnFactoryName);
            return serverCnxnFactory;
        } catch (Exception e) {
            IOException ioe = new IOException("Couldn't instantiate "
                    + serverCnxnFactoryName);
            ioe.initCause(e);
            throw ioe;
        }
    }
```

cnxnFactory.startup(zkServer);方法启动了ServerCnxnFactory ，同时启动ZooKeeper服务

```
public void startup(ZooKeeperServer zks, boolean startServer)
            throws IOException, InterruptedException {
        // 启动相关线程
        //开启NIOWorker线程池，
        //启动NIO Selector线程
        //启动客户端连接处理acceptThread线程
        start();
        setZooKeeperServer(zks);

        //启动服务
        if (startServer) {
            // 加载数据到zkDataBase
            zks.startdata();
            // 启动定时清除session的管理器,注册jmx,添加请求处理器
            zks.startup();
        }
    }
```

zks.startdata();

```
public void startdata() throws IOException, InterruptedException {
        //初始化ZKDatabase，该数据结构用来保存ZK上面存储的所有数据
        //check to see if zkDb is not null
        if (zkDb == null) {
            //初始化数据数据，这里会加入一些原始节点，例如/zookeeper
            zkDb = new ZKDatabase(this.txnLogFactory);
        }
        //加载磁盘上已经存储的数据，如果有的话
        if (!zkDb.isInitialized()) {
            loadData();
        }
    }
```

zks.startup();

```
public synchronized void startup() {
        //初始化session追踪器
        if (sessionTracker == null) {
            createSessionTracker();
        }
        //启动session追踪器
        startSessionTracker();

        //建立请求处理链路
        setupRequestProcessors();

        //注册jmx
        registerJMX();

        setState(State.RUNNING);
        notifyAll();
    }
```

最终Zookeeper应用服务启动，并处于监听状态。

### 集群模式启动

集群模式下启动和单机启动有相似的地方，但是也有各自的特点。集群模式的配置方式和单机模式也是不一样的

#### 概念介绍：角色，服务器状态

集群模式会有多台server，每台server根据不同的角色会有不同的状态，server状态的定义如下

```java
public enum ServerState {
    LOOKING, FOLLOWING, LEADING, OBSERVING;
}
```

LOOKING：表示服务器处于选举状态，说明集群正在进行投票选举，选出leader

FOLLOWING：表示服务器处于following状态，表示当前server的角色是follower

LEADING：表示服务器处于leading状态，当前server角色是leader

OBSERVING：表示服务器处于OBSERVING状态，当前server角色是OBSERVER

对应server的角色有:

##### leader

投票选出的leader，可以处理读写请求。处理写请求的时候收集各个参与投票者的选票，来决出投票结果

##### follower

作用：

1. 参与leader选举，可能被选为leader
2. 接收处理读请求
3. 接收写请求，转发给leader，并参与投票决定写操作是否提交

##### observer

为了支持zk集群可扩展性，如果直接增加follower的数量，会导致投票的性能下降。也就是防止参与投票的server太多，导致leader选举收敛速度较慢，选举所需时间过长。

observer和follower类似，但是不参与选举和投票，

1. 接收处理读请求
2. 接收写请求，转发给leader，但是不参与投票，接收leader的投票结果，同步数据

这样在支持集群可扩展性的同时又不会影响投票的性能

#### 组件启动

集群模式下服务器启动的组件一部分和单机模式下类似，只是启动的流程和时机有所差别

- FileTxnSnapLog
- NIOServerCnxnFactory
- Jetty

也是会启动上面三个组件，但是因为集群模式还有其他组件需要启动，所以具体启动的逻辑不太一样。

除了上面这些组件外，集群模式下还有一些用来支撑集群模式的组件

- QuorumPeer：用来启动各个组件，是选举过程的mainloop，在loop中判断当前server状态来决定做不同的处理
- FastLeaderElection：默认选举算法
- QuorumCnxManager：选举过程中的网络通信组件

#### QuorumPeer

解除出来的QuorumPeerConfig配置都设置到QuorumPeer对应的属性中，主线程启动完QuorumPeer后，调用该线程的join方法等待该线程退出。

#### QuorumCnxManager

负责各个server之间的通信，维护了和各个server之间的连接，下面的线程负责与其他server建立连接

```
org.apache.zookeeper.server.quorum.QuorumCnxManager.Listener
```

还维护了与其他每个server连接对应的发送队列，SendWorker线程负责发送packet给其他server

```
final ConcurrentHashMap<Long, ArrayBlockingQueue<ByteBuffer>> queueSendMap;
```

这个map的key是建立网络连接的server的myid，value是对应的发送队列。

还有接收队列，RecvWorker是用来接收其他server发来的Message的线程，将收到的Message放入队列中

```
org.apache.zookeeper.server.quorum.QuorumCnxManager#recvQueue
```

#### 集群模式启动

Zookeeper主程序QuorumPeerMain加载配置文件后，配置容器对象QuorumPeerConfig中持有一个QuorumVerifier对象，该对象会存储其他Zookeeper server节点信息，如果zoo.cfg中配置了server.*节点信息，会实例化一个QuorumVeriferi对象。其中AllMembers = VotingMembers + ObservingMembers

```
public interface QuorumVerifier {
    long getWeight(long id);
    boolean containsQuorum(Set<Long> set);
    long getVersion();
    void setVersion(long ver);
    Map<Long, QuorumServer> getAllMembers();
    Map<Long, QuorumServer> getVotingMembers();
    Map<Long, QuorumServer> getObservingMembers();
    boolean equals(Object o);
    String toString();
}
```

如果quorumVerifier.getVotingMembers().size() > 1 则使用集群模式启动。调用runFromConfig(QuorumPeerConfig config)，同时会实例化ServerCnxnFactory 对象，初始化一个QuorumPeer对象。

　　QuorumPeer为一个Zookeeper节点， QuorumPeer 为一个线程类，代表一个Zookeeper服务线程，最终会启动该线程。

　　runFromConfig方法中设置了一些列属性。包括选举类型、server Id、节点数据库等信息。最后通过quorumPeer.start();启动Zookeeper节点。

public void runFromConfig(QuorumPeerConfig config)
            throws IOException, AdminServerException
    {
      try {
          // 注册jmx
          ManagedUtil.registerLog4jMBeans();
      } catch (JMException e) {
          LOG.warn("Unable to register log4j JMX control", e);
      }

      LOG.info("Starting quorum peer");
      try {
          ServerCnxnFactory cnxnFactory = null;
          ServerCnxnFactory secureCnxnFactory = null;
    
          if (config.getClientPortAddress() != null) {
              cnxnFactory = ServerCnxnFactory.createFactory();
              // 配置客户端连接端口
              cnxnFactory.configure(config.getClientPortAddress(),
                      config.getMaxClientCnxns(),
                      false);
          }
    
          if (config.getSecureClientPortAddress() != null) {
              secureCnxnFactory = ServerCnxnFactory.createFactory();
              // 配置安全连接端口
              secureCnxnFactory.configure(config.getSecureClientPortAddress(),
                      config.getMaxClientCnxns(),
                      true);
          }
    
          // ------------初始化当前zk服务节点的配置----------------
          // 设置数据和快照操作
          quorumPeer = getQuorumPeer();
          quorumPeer.setTxnFactory(new FileTxnSnapLog(
                      config.getDataLogDir(),
                      config.getDataDir()));
          quorumPeer.enableLocalSessions(config.areLocalSessionsEnabled());
          quorumPeer.enableLocalSessionsUpgrading(
              config.isLocalSessionsUpgradingEnabled());
          //quorumPeer.setQuorumPeers(config.getAllMembers());
          // 选举类型
          quorumPeer.setElectionType(config.getElectionAlg());
          // server Id
          quorumPeer.setMyid(config.getServerId());
          quorumPeer.setTickTime(config.getTickTime());
          quorumPeer.setMinSessionTimeout(config.getMinSessionTimeout());
          quorumPeer.setMaxSessionTimeout(config.getMaxSessionTimeout());
          quorumPeer.setInitLimit(config.getInitLimit());
          quorumPeer.setSyncLimit(config.getSyncLimit());
          quorumPeer.setConfigFileName(config.getConfigFilename());
    
          // 设置zk的节点数据库
          quorumPeer.setZKDatabase(new ZKDatabase(quorumPeer.getTxnFactory()));
          quorumPeer.setQuorumVerifier(config.getQuorumVerifier(), false);
          if (config.getLastSeenQuorumVerifier()!=null) {
              quorumPeer.setLastSeenQuorumVerifier(config.getLastSeenQuorumVerifier(), false);
          }
    
          // 初始化zk数据库
          quorumPeer.initConfigInZKDatabase();
          quorumPeer.setCnxnFactory(cnxnFactory);
          quorumPeer.setSecureCnxnFactory(secureCnxnFactory);
          quorumPeer.setLearnerType(config.getPeerType());
          quorumPeer.setSyncEnabled(config.getSyncEnabled());
          quorumPeer.setQuorumListenOnAllIPs(config.getQuorumListenOnAllIPs());
    
          // sets quorum sasl authentication configurations
          quorumPeer.setQuorumSaslEnabled(config.quorumEnableSasl);
          if(quorumPeer.isQuorumSaslAuthEnabled()){
              quorumPeer.setQuorumServerSaslRequired(config.quorumServerRequireSasl);
              quorumPeer.setQuorumLearnerSaslRequired(config.quorumLearnerRequireSasl);
              quorumPeer.setQuorumServicePrincipal(config.quorumServicePrincipal);
              quorumPeer.setQuorumServerLoginContext(config.quorumServerLoginContext);
              quorumPeer.setQuorumLearnerLoginContext(config.quorumLearnerLoginContext);
          }
          quorumPeer.setQuorumCnxnThreadsSize(config.quorumCnxnThreadsSize);
    
          // -------------初始化当前zk服务节点的配置---------------
          quorumPeer.initialize();
    
          //启动
          quorumPeer.start();
          quorumPeer.join();
      } catch (InterruptedException e) {
          // warn, but generally this is ok
          LOG.warn("Quorum Peer interrupted", e);
      }
    }

quorumPeer.start(); Zookeeper会首先加载本地磁盘数据，如果之前存在一些Zookeeper信息，则会加载到Zookeeper内存数据库中。通过FileTxnSnapLog中的loadDatabse();

```
public synchronized void start() {

        // 校验serverid如果不在peer列表中，抛异常
        if (!getView().containsKey(myid)) {
            throw new RuntimeException("My id " + myid + " not in the peer list");
         }

        // 加载zk数据库:载入之前持久化的一些信息
        loadDataBase();

        // 启动连接服务端
        startServerCnxnFactory();
        try {
            adminServer.start();
        } catch (AdminServerException e) {
            LOG.warn("Problem starting AdminServer", e);
            System.out.println(e);
        }
        // 启动之后马上进行选举，主要是创建选举必须的环境，比如：启动相关线程
        startLeaderElection();

        // 执行选举逻辑
        super.start();
    }
```

加载数据完之后同单机模式启动一样，会调用ServerCnxnFactory.start(),启动NIOServerCnxnFactory服务和Zookeeper服务，最后启动AdminServer服务。

　　与单机模式启动不同的是，集群会在启动之后马上进行选举操作，会在配置的所有Zookeeper server节点中选举出一个leader角色。startLeaderElection(); 

## 选举

Zookeeper中分为Leader、Follower和Observer三个角色，各个角色扮演不同的业务功能。在Leader故障之后，Follower也会选举一个新的Leader。

　　Leader为集群中的主节点，一个集群只有一个Leader，Leader负责处理Zookeeper的事物操作，也就是更改Zookeeper数据和状态的操作。

　　Follower负责处理客户端的读请求和参与选举。同时负责处理Leader发出的事物提交请求，也就是提议（proposal）。

　　Observer用于提高Zookeeper集群的读取的吞吐量，响应读请求，和Follower不同的是，Observser不参与Leader的选举，也不响应Leader发出的proposal。

　　有角色就有选举。有选举就有策略，Zookeeper中的选举策略有三种实现：包括了LeaderElection、AuthFastLeaderElection和FastLeaderElection，目前Zookeeper默认采用FastLeaderElection，前两个选举算法已经设置为@Deprecated；

### **Zookeeper节点信息**

　　serverId：服务节点Id，也就是Zookeeper dataDir中配置的myid ，server.*上指定的id。0,1,2,3,4..... ，该Id启动后不变

　　zxid：数据状态Id，zookeeper每次更新状态之后增加，可理解为全局有序id ，zxid越大，表示数据越新。Zxid是一个64位的数字，高32位为epoch，低32位为递增计数。

　　epoch：选举时钟,也可以理解为选举轮次，没进行一次选举，该值会+1；

　　ServerState：服务状态，Zookeeper节点角色状态，分为LOOKING、FOLLOWING、LEADING和OBSERVING，分别对应于不同的角色，当处于选举时，节点处于Looking状态。

　　每次投票，一个Vote会包含Zookeeper节点信息。

　　Zookeeper在启动之后会马上进行选举操作，不断的向其他Follower节点发送选票信息，同时也接收别的Follower发送过来的选票信息。最终每个Follower都持有共同的一个选票池，通过同样的算法选出Leader，如果当前节点选为Leader，则向其他每个Follower发送信息，如果没有则向Leader发送信息。

Zookeeper定义了Election接口；其中lookForLeader()就是选举操作。

启动peer选举策略实际启动的为fast leader 选举策略，如果peer状态为LOOKING， 创建投票（最后提交的日志id，时间戳，peerId）。

fast leader 选举策略启动时实际上启动了一个消息处理器Messenger。 消息处理器内部有一个发送消息工作线程WorkerSender，出列一个需要发送的消息，并把它放入管理器QuorumCnxManager的队列； 一个消息接收线程WorkerReceiver处理从QuorumCnxManager接收的消息。

发送消息工作线程WorkerSender，从FastLeaderElection的发送队列poll消息，并把它放入管理器QuorumCnxManager的队列，如果需要则建立消息关联的peer，并发送协议版本，服务id及选举地址, 如果连接peer的id大于 当前peer的id，则关闭连接，否则启动发送工作线程SendWorker和接收线程RecvWorker。 同时QuorumCnxManager在启动时，启动监听，监听peer的连接。发送消息线程SendWorker，从消息队列拉取消息，并通过Socket的DataOutputStream，发送给peer。

消息接收线程WorkerReceiver从QuorumCnxManager的接收队列中拉取消息，并解析出peer的状态（LOOKING, 观察，Follower，或者leader）， 事务id，leaderId，leader选举时间戳，peer的时间戳等信息；如果peer不在当前投票的视图范围之内，同步当前peer的状态（构建通知消息（服务id，事务id，peer状态，时间戳等），并放到发送队列）， 然后更新通知(事务id，leaderId，leader选举时间戳,peer时间戳)，如果当前peer的状态为LOOKING，则添加通知消息到peer的消息接收队列，如果peer状态为LOOKING，则同步当前节点的投票信息给peer， 若果当前节点为非looker，而peer为looker，则发送当前peer相信的leader信息。

接收工作线程RecvWorker，主要是从Socket的Data输入流中读取数据，并组装成消息，放到QuorumCnxManager的消息接收队列，待消息接收线程WorkerReceiver处理。

peer状态有四种LOOKING， OBSERVING，FOLLOWING和LEADING几种状态；LOOKING为初态，Leader还没有选举成功，其他为终态。

当前QuorumPeer处于LOOKING提议投票阶段，启动一个ReadOnlyZooKeeperServer服务，并设置当前peer投票。 ReadOnlyZooKeeperServer内部的处理器链为ReadOnlyRequestProcessor->PrepRequestProcessor->FinalRequestProcessor。 ，只读处理器ReadOnlyRequestProcessor，对CRUD相关的操作，进行忽略，只处理check请求，并通过NettyServerCnxn发送ReplyHeader，头部主要的信息为内存数据库的最大事务id。

创建投票，首先更新当前的投票信息，如果peer为参与者，首先投自己一票（当前peer的serverId，最大事务id，以及时间戳）,并发送通知到所有投票peer; 如果peer状态为LOOKING，且选举没有结束，则从接收消息队列拉取通知, 如果通知为空，则发送投票提议通知到所有投票peer, 否则判断下一轮投票视图是否包括当前通知的server和提议leader, 则判断peer的状态(LOOKING,OBSERVING,FOLLOWING,LEADING)。当前peer状态为LOOKING时,，如果通知的时间点，大于当前server时间点，则更新投票提议，并发送通知消息到所有投票peer。如果当前节点的Quorum Peer都进行投票回复，然后从接收队列中拉取通知投票消息，如果为空，则投票结束，更新当前投票状态为LEADING。当peer为OBSERVING,FOLLOWING状态，什么都不做;当peer状态为leading，则如果投票的时间戳和当前节点的投票时间戳一致，并且所有peer都回复，则结束投票。

### leader选举

```
public interface Election {
     public Vote lookForLeader() throws InterruptedException;
     public void shutdown();
}
```

说明：

　　选举的父接口为Election，其定义了lookForLeader和shutdown两个方法，lookForLeader表示寻找Leader，shutdown则表示关闭，如关闭服务端之间的连接。

　　AuthFastLeaderElection，同FastLeaderElection算法基本一致，只是在消息中加入了认证信息，其在3.4.0之后的版本中已经不建议使用。

　　FastLeaderElection，其是标准的fast paxos算法的实现，基于TCP协议进行选举。

　　LeaderElection，也表示一种选举算法，其在3.4.0之后的版本中已经不建议使用。

选举入口在下面的方法中

```
org.apache.zookeeper.server.quorum.FastLeaderElection#lookForLeader
```

说明：FastLeaderElection实现了Election接口，其需要实现接口中定义的lookForLeader方法和shutdown方法，其是标准的Fast Paxos算法的实现，各服务器之间基于TCP协议进行选举。

在上面的集群模式启动流程中，最后会调用startLeaderElection()来下进行选举操作。startLeaderElection()中指定了选举算法。同时定义了为自己投一票（坚持你自己，年轻人！），一个Vote包含了投票节点、当前节点的zxid和当前的epoch。Zookeeper默认采取了FastLeaderElection选举算法。最后启动QuorumPeer线程，开始投票。

```
synchronized public void startLeaderElection() {
       try {

           // 所有节点启动的初始状态都是LOOKING，因此这里都会是创建一张投自己为Leader的票
           if (getPeerState() == ServerState.LOOKING) {
               currentVote = new Vote(myid, getLastLoggedZxid(), getCurrentEpoch());
           }
       } catch(IOException e) {
           RuntimeException re = new RuntimeException(e.getMessage());
           re.setStackTrace(e.getStackTrace());
           throw re;
       }

       // if (!getView().containsKey(myid)) {
      //      throw new RuntimeException("My id " + myid + " not in the peer list");
        //}
        if (electionType == 0) {
            try {
                udpSocket = new DatagramSocket(myQuorumAddr.getPort());
                responder = new ResponderThread();
                responder.start();
            } catch (SocketException e) {
                throw new RuntimeException(e);
            }
        }
        //初始化选举算法，electionType默认为3
        this.electionAlg = createElectionAlgorithm(electionType);
    }
```

FastLeaderElection类中定义三个内部类Notification、 ToSend 和 Messenger ，Messenger 中又定义了WorkerReceiver 和 WorkerSender ![Zookeeper选举类图](png\zookeeper\Zookeeper选举类图.png)

​		Notification类表示收到的选举投票信息（其他服务器发来的选举投票信息），其包含了被选举者的id、zxid、选举周期等信息。

　　ToSend类表示发送给其他服务器的选举投票信息，也包含了被选举者的id、zxid、选举周期等信息。

　　Message类为消息处理的类，用于发送和接收投票信息，包含了WorkerReceiver和WorkerSender两个线程类。

 WorkerReceiver

　　说明：WorkerReceiver实现了Runnable接口，是选票接收器。其会不断地从QuorumCnxManager中获取其他服务器发来的选举消息，并将其转换成一个选票，然后保存到recvqueue中，在选票接收过程中，如果发现该外部选票的选举轮次小于当前服务器的，那么忽略该外部投票，同时立即发送自己的内部投票。其是将QuorumCnxManager的Message转化为FastLeaderElection的Notification。

　　其中，WorkerReceiver的主要逻辑在run方法中，其首先会从QuorumCnxManager中的recvQueue队列中取出其他服务器发来的选举消息，消息封装在Message数据结构中。然后判断消息中的服务器id是否包含在可以投票的服务器集合中，若不是，则会将本服务器的内部投票发送给该服务器

```
                        if(!self.getVotingView().containsKey(response.sid)){ // 当前的投票者集合不包含服务器
                            // 获取自己的投票
                            Vote current = self.getCurrentVote();
                            // 构造ToSend消息
                            ToSend notmsg = new ToSend(ToSend.mType.notification,
                                    current.getId(),
                                    current.getZxid(),
                                    logicalclock,
                                    self.getPeerState(),
                                    response.sid,
                                    current.getPeerEpoch());
                            // 放入sendqueue队列，等待发送
                            sendqueue.offer(notmsg);
                        }
```

若包含该服务器，则根据消息（Message）解析出投票服务器的投票信息并将其封装为Notification，然后判断当前服务器是否为LOOKING，若为LOOKING，则直接将Notification放入FastLeaderElection的recvqueue（区别于recvQueue）中。然后判断投票服务器是否为LOOKING状态，并且其选举周期小于当前服务器的逻辑时钟，则将本（当前）服务器的内部投票发送给该服务器，否则，直接忽略掉该投票。

WorkerSender

　　说明：WorkerSender也实现了Runnable接口，为选票发送器，其会不断地从sendqueue中获取待发送的选票，并将其传递到底层QuorumCnxManager中，其过程是将FastLeaderElection的ToSend转化为QuorumCnxManager的Message。

```
    protected class Messenger {
        // 选票发送器
        WorkerSender ws;
        // 选票接收器
        WorkerReceiver wr;
    }
```

说明：Messenger中维护了一个WorkerSender和WorkerReceiver，分别表示选票发送器和选票接收器。

```
       Messenger(QuorumCnxManager manager) {
            // 创建WorkerSender
            this.ws = new WorkerSender(manager);
            // 新创建线程
            Thread t = new Thread(this.ws,
                    "WorkerSender[myid=" + self.getId() + "]");
            // 设置为守护线程
            t.setDaemon(true);
            // 启动
            t.start();

            // 创建WorkerReceiver
            this.wr = new WorkerReceiver(manager);
            // 创建线程
            t = new Thread(this.wr,
                    "WorkerReceiver[myid=" + self.getId() + "]");
            // 设置为守护线程
            t.setDaemon(true);
            // 启动
            t.start();
        }
```

说明：会启动WorkerSender和WorkerReceiver，并设置为守护线程。

### 选举流程

#### 判断投票结果的策略

上面这个是其中的一种选举算法，选举过程中，各个server收到投票后需要进行投票结果抉择，判断投票结果的策略有两种

```java
// 按照分组权重
org.apache.zookeeper.server.quorum.flexible.QuorumHierarchical

// 简单按照是否是大多数，超过参与投票数的一半
org.apache.zookeeper.server.quorum.flexible.QuorumMaj
```

#### 选票的网络传输

zookeeper中选举使用的端口和正常处理client请求的端口是不一样的，而且由于投票的数据和处理请求的数据不一样，数据传输的方法也不一样。选举使用的网络传输相关的类和数据结构如下

![Zookeeper选举选票](png\zookeeper\Zookeeper选举选票.png)



### 选举过程

- 各自初始化选票
- - proposedLeader：一开始都是选举自己，myid
    - proposedZxid：最后一次处理成功的事务的zxid
    - proposedEpoch：上一次选举成功的leader的epoch，从currentEpoch 文件中读取
- 发送自己的选票给其他参选者
- 接收其他参选者的选票
- - 收到其他参选者的选票后会放入recvqueue，这个是阻塞队列，从里面超时获取
    - 如果超时没有获取到选票vote则采用退避算法，下次使用更长的超时时间
    - 校验选票的有效性，并且当前机器处于looking状态，开始判断是否接受
    - - 如果收到的选票的electionEpoch大于当前机器选票的logicalclock
        - - 进行选票pk，收到的选票和本机初始选票pk，如果收到的选票胜出则更新本地的选票为收到的选票
            - - pk的算法
                - - org.apache.zookeeper.server.quorum.FastLeaderElection#totalOrderPredicate
                    - 选取epoch较大的
                    - 如果epoch相等则取zxid较大的
                    - 如果zxid相等则取myid较大的
            - 如果本机初始选票胜出则更新为当前机器的选票
            - 更新完选票之后重新发出自己的选票
    - - 如果n.electionEpoch < logicalclock.get()则丢弃选票，继续准备接收其他选票
    - - 如果n.electionEpoch == logicalclock.get()并且收到的选票pk（pk算法totalOrderPredicate）之后胜出
        - - 更新本机选票并且，发送新的选票给其他参选者
        - 执行到这里，说明收到的这个选票有效，将选票记录下来，recvset
        - 统计选票
        - - org.apache.zookeeper.server.quorum.FastLeaderElection#getVoteTracker
            - 看看已经收到的投票中，和当前机器选票一致的票数
        - 判断投票结果
        - - org.apache.zookeeper.server.quorum.SyncedLearnerTracker#hasAllQuorums
            - 根据具体的策略判断
            - - QuorumHierarchical
                - QuorumMaj，默认是这个
                - - 判断投该票的主机数目是否占参与投票主机数的大部分，也就是大于1/2
            - 如果本轮选举成功
            - - 如果等finalizeWait时间后还没有其他选票的时候，就认为当前选举结束
                - 设置当前主机状态
                - 退出本轮选举

选举的整个流程为![Zookeeper选举详情](png\zookeeper\Zookeeper选举详情.png)

整个选举过程可大致理解不断的接收选票，比对选票，直到选出leader，每个zookeeper节点都持有自己的选票池，按照统一的比对算法，正常情况下最终选出来的leader是一致的。

### 源码详解

FastLeaderElection类：

说明：其维护了服务器之间的连接（用于发送消息）、发送消息队列、接收消息队列、推选者的一些信息（zxid、id）、是否停止选举流程标识等。

```
public class FastLeaderElection implements Election {
    //..........
    /**
     * Connection manager. Fast leader election uses TCP for
     * communication between peers, and QuorumCnxManager manages
     * such connections.
     */

    QuorumCnxManager manager;
    /*
        Notification表示收到的选举投票信息（其他服务器发来的选举投票信息），
        其包含了被选举者的id、zxid、选举周期等信息，
        其buildMsg方法将选举信息封装至ByteBuffer中再进行发送
     */
    static public class Notification {
       //..........
    }
    /**
     * Messages that a peer wants to send to other peers.
     * These messages can be both Notifications and Acks
     * of reception of notification.
     */
    /*
     ToSend表示发送给其他服务器的选举投票信息，也包含了被选举者的id、zxid、选举周期等信息
     */
    static public class ToSend {
        //..........
    }
    LinkedBlockingQueue<ToSend> sendqueue;
    LinkedBlockingQueue<Notification> recvqueue;

    /**
     * Multi-threaded implementation of message handler. Messenger
     * implements two sub-classes: WorkReceiver and  WorkSender. The
     * functionality of each is obvious from the name. Each of these
     * spawns a new thread.
     */
    protected class Messenger {
        /**
         * Receives messages from instance of QuorumCnxManager on
         * method run(), and processes such messages.
         */

        class WorkerReceiver extends ZooKeeperThread  {
             //..........
        }
        /**
         * This worker simply dequeues a message to send and
         * and queues it on the manager's queue.
         */

        class WorkerSender extends ZooKeeperThread {
            //..........
        }

        WorkerSender ws;
        WorkerReceiver wr;
        Thread wsThread = null;
        Thread wrThread = null;


    }
    //..........
    QuorumPeer self;
    Messenger messenger;
    AtomicLong logicalclock = new AtomicLong(); /* Election instance */
    long proposedLeader;
    long proposedZxid;
    long proposedEpoch;
    //..........
}
```

说明：构造函数中初始化了stop字段和manager字段，并且调用了starter函数

```
    public FastLeaderElection(QuorumPeer self, QuorumCnxManager manager){
        // 字段赋值
        this.stop = false;
        this.manager = manager;
        // 初始化其他信息
        starter(self, manager);
    }
```

调用

```
    private void starter(QuorumPeer self, QuorumCnxManager manager) {
        // 赋值，对Leader和投票者的ID进行初始化操作
        this.self = self;
        proposedLeader = -1;
        proposedZxid = -1;
        
        // 初始化发送队列
        sendqueue = new LinkedBlockingQueue<ToSend>();
        // 初始化接收队列
        recvqueue = new LinkedBlockingQueue<Notification>();
        // 创建Messenger，会启动接收器和发送器线程
        this.messenger = new Messenger(manager);
    }
```

说明：其完成在构造函数中未完成的部分，如会初始化FastLeaderElection的sendqueue和recvqueue，并且启动接收器和发送器线程。

核心函数分析

sendNotifications函数　　

```
    private void sendNotifications() {
        for (QuorumServer server : self.getVotingView().values()) { // 遍历投票参与者集合
            long sid = server.id;
            
            // 构造发送消息
            ToSend notmsg = new ToSend(ToSend.mType.notification,
                    proposedLeader,
                    proposedZxid,
                    logicalclock,
                    QuorumPeer.ServerState.LOOKING,
                    sid,
                    proposedEpoch);
            if(LOG.isDebugEnabled()){
                LOG.debug("Sending Notification: " + proposedLeader + " (n.leader), 0x"  +
                      Long.toHexString(proposedZxid) + " (n.zxid), 0x" + Long.toHexString(logicalclock)  +
                      " (n.round), " + sid + " (recipient), " + self.getId() +
                      " (myid), 0x" + Long.toHexString(proposedEpoch) + " (n.peerEpoch)");
            }
            // 将发送消息放置于队列
            sendqueue.offer(notmsg);
        }
    }
```

说明：其会遍历所有的参与者投票集合，然后将自己的选票信息发送至上述所有的投票者集合，其并非同步发送，而是将ToSend消息放置于sendqueue中，之后由WorkerSender进行发送。

QuorumPeer线程启动后会开启对ServerState的监听，如果当前服务节点属于Looking状态，则会执行选举操作。Zookeeper服务器启动后是Looking状态，所以服务启动后会马上进行选举操作。通过调用makeLEStrategy().lookForLeader()进行投票操作，也就是FastLeaderElection.lookForLeader();

QuorumPeer.run()：

```
public void run() {
        updateThreadName();

        //..........

        try {
            /*
             * Main loop
             */
            while (running) {
                switch (getPeerState()) {
                case LOOKING:
                    LOG.info("LOOKING");

                    if (Boolean.getBoolean("readonlymode.enabled")) {
                        final ReadOnlyZooKeeperServer roZk =
                            new ReadOnlyZooKeeperServer(logFactory, this, this.zkDb);
                        Thread roZkMgr = new Thread() {
                            public void run() {
                                try {
                                    // lower-bound grace period to 2 secs
                                    sleep(Math.max(2000, tickTime));
                                    if (ServerState.LOOKING.equals(getPeerState())) {
                                        roZk.startup();
                                    }
                                } catch (InterruptedException e) {
                                    LOG.info("Interrupted while attempting to start ReadOnlyZooKeeperServer, not started");
                                } catch (Exception e) {
                                    LOG.error("FAILED to start ReadOnlyZooKeeperServer", e);
                                }
                            }
                        };
                        try {
                            roZkMgr.start();
                            reconfigFlagClear();
                            if (shuttingDownLE) {
                                shuttingDownLE = false;
                                startLeaderElection();
                            }
                            setCurrentVote(makeLEStrategy().lookForLeader());
                        } catch (Exception e) {
                            LOG.warn("Unexpected exception", e);
                            setPeerState(ServerState.LOOKING);
                        } finally {
                            roZkMgr.interrupt();
                            roZk.shutdown();
                        }
                    } else {
                        try {
                           reconfigFlagClear();
                            if (shuttingDownLE) {
                               shuttingDownLE = false;
                               startLeaderElection();
                               }
                            setCurrentVote(makeLEStrategy().lookForLeader());
                        } catch (Exception e) {
                            LOG.warn("Unexpected exception", e);
                            setPeerState(ServerState.LOOKING);
                        }
                    }
                    break;
                case OBSERVING:
                    try {
                        LOG.info("OBSERVING");
                        setObserver(makeObserver(logFactory));
                        observer.observeLeader();
                    } catch (Exception e) {
                        LOG.warn("Unexpected exception",e );
                    } finally {
                        observer.shutdown();
                        setObserver(null);
                       updateServerState();
                    }
                    break;
                case FOLLOWING:
                    try {
                       LOG.info("FOLLOWING");
                        setFollower(makeFollower(logFactory));
                        follower.followLeader();
                    } catch (Exception e) {
                       LOG.warn("Unexpected exception",e);
                    } finally {
                       follower.shutdown();
                       setFollower(null);
                       updateServerState();
                    }
                    break;
                case LEADING:
                    LOG.info("LEADING");
                    try {
                        setLeader(makeLeader(logFactory));
                        leader.lead();
                        setLeader(null);
                    } catch (Exception e) {
                        LOG.warn("Unexpected exception",e);
                    } finally {
                        if (leader != null) {
                            leader.shutdown("Forcing shutdown");
                            setLeader(null);
                        }
                        updateServerState();
                    }
                    break;
                }
                start_fle = Time.currentElapsedTime();
            }
        } finally {
            LOG.warn("QuorumPeer main thread exited");
            MBeanRegistry instance = MBeanRegistry.getInstance();
            instance.unregister(jmxQuorumBean);
            instance.unregister(jmxLocalPeerBean);

            for (RemotePeerBean remotePeerBean : jmxRemotePeerBean.values()) {
                instance.unregister(remotePeerBean);
            }

            jmxQuorumBean = null;
            jmxLocalPeerBean = null;
            jmxRemotePeerBean = null;
        }
    }
```

FastLeaderElection.lookForLeader()：

```
public Vote lookForLeader() throws InterruptedException {
        try {
            self.jmxLeaderElectionBean = new LeaderElectionBean();
            MBeanRegistry.getInstance().register(
                    self.jmxLeaderElectionBean, self.jmxLocalPeerBean);
        } catch (Exception e) {
            LOG.warn("Failed to register with JMX", e);
            self.jmxLeaderElectionBean = null;
        }
        if (self.start_fle == 0) {
           self.start_fle = Time.currentElapsedTime();
        }
        try {
            HashMap<Long, Vote> recvset = new HashMap<Long, Vote>();

            HashMap<Long, Vote> outofelection = new HashMap<Long, Vote>();
            //等待200毫秒
            int notTimeout = finalizeWait;

            synchronized(this){
                //逻辑时钟自增+1
                logicalclock.incrementAndGet();
                updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
            }

            LOG.info("New election. My id =  " + self.getId() +
                    ", proposed zxid=0x" + Long.toHexString(proposedZxid));
            //发送投票信息
            sendNotifications();

            /*
             * Loop in which we exchange notifications until we find a leader
             */
            //判断是否为Looking状态
            while ((self.getPeerState() == ServerState.LOOKING) &&
                    (!stop)){
                /*
                 * Remove next notification from queue, times out after 2 times
                 * the termination time
                 */
                //获取接收其他Follow发送的投票信息
                Notification n = recvqueue.poll(notTimeout,
                        TimeUnit.MILLISECONDS);

                /*
                 * Sends more notifications if haven't received enough.
                 * Otherwise processes new notification.
                 */
                //未收到投票信息
                if(n == null){
                    //判断是否和集群离线了
                    if(manager.haveDelivered()){
                        //未断开，发送投票
                        sendNotifications();
                    } else {
                        //断开，重连
                        manager.connectAll();
                    }
                    /*
                     * Exponential backoff
                     */
                    int tmpTimeOut = notTimeout*2;
                    notTimeout = (tmpTimeOut < maxNotificationInterval?
                            tmpTimeOut : maxNotificationInterval);
                    LOG.info("Notification time out: " + notTimeout);
                } //接收到了投票，则处理收到的投票信息
                else if (validVoter(n.sid) && validVoter(n.leader)) {
                    /*
                     * Only proceed if the vote comes from a replica in the current or next
                     * voting view for a replica in the current or next voting view.
                     */
                    //其他节点的Server.state
                    switch (n.state) {
                    case LOOKING:
                        //如果其他节点也为Looking状态，说明当前正处于选举阶段，则处理投票信息。

                        // If notification > current, replace and send messages out
                        //如果当前的epoch（投票轮次）小于其他的投票信息，则说明自己的投票轮次已经过时，则更新自己的投票轮次
                        if (n.electionEpoch > logicalclock.get()) {
                            //更新投票轮次
                            logicalclock.set(n.electionEpoch);
                            //清除收到的投票
                            recvset.clear();
                            //比对投票信息
                            //如果本身的投票信息 低于 收到的的投票信息，则使用收到的投票信息，否则再次使用自身的投票信息进行发送投票。
                            if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                    getInitId(), getInitLastLoggedZxid(), getPeerEpoch())) {
                                //使用收到的投票信息
                                updateProposal(n.leader, n.zxid, n.peerEpoch);
                            } else {
                                //使用自己的投票信息
                                updateProposal(getInitId(),
                                        getInitLastLoggedZxid(),
                                        getPeerEpoch());
                            }
                            //发送投票信息
                            sendNotifications();
                        } else if (n.electionEpoch < logicalclock.get()) {
                            //如果其他节点的epoch小于当前的epoch则丢弃
                            if(LOG.isDebugEnabled()){
                                LOG.debug("Notification election epoch is smaller than logicalclock. n.electionEpoch = 0x"
                                        + Long.toHexString(n.electionEpoch)
                                        + ", logicalclock=0x" + Long.toHexString(logicalclock.get()));
                            }
                            break;
                        } else if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                proposedLeader, proposedZxid, proposedEpoch)) {
                            //同样的epoch,正常情况，所有节点基本处于同一轮次
                            //如果自身投票信息 低于 收到的投票信息，则更新投票信息。并发送
                            updateProposal(n.leader, n.zxid, n.peerEpoch);
                            sendNotifications();
                        }

                        if(LOG.isDebugEnabled()){
                            LOG.debug("Adding vote: from=" + n.sid +
                                    ", proposed leader=" + n.leader +
                                    ", proposed zxid=0x" + Long.toHexString(n.zxid) +
                                    ", proposed election epoch=0x" + Long.toHexString(n.electionEpoch));
                        }
                        //投票信息Vote归档，收到的有效选票 票池
                        recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));

                        //统计投票结果 ，判断是否能结束选举
                        if (termPredicate(recvset,
                                new Vote(proposedLeader, proposedZxid,
                                        logicalclock.get(), proposedEpoch))) {
                            //如果已经选出leader

                            // Verify if there is any change in the proposed leader
                            while((n = recvqueue.poll(finalizeWait,
                                    TimeUnit.MILLISECONDS)) != null){
                                if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                        proposedLeader, proposedZxid, proposedEpoch)){
                                    recvqueue.put(n);
                                    break;
                                }
                            }

                            /*
                             * This predicate is true once we don't read any new
                             * relevant message from the reception queue
                             */
                            //如果选票结果为当前节点，则更新ServerState，否则设置为Follwer
                            if (n == null) {
                                self.setPeerState((proposedLeader == self.getId()) ?
                                        ServerState.LEADING: learningState());

                                Vote endVote = new Vote(proposedLeader,
                                        proposedZxid, proposedEpoch);
                                leaveInstance(endVote);
                                return endVote;
                            }
                        }
                        break;
                    case OBSERVING:
                        LOG.debug("Notification from observer: " + n.sid);
                        break;
                    case FOLLOWING:
                    case LEADING:
                        /*
                         * Consider all notifications from the same epoch
                         * together.
                         */
                        //如果其他节点已经确定为Leader
                        //如果同一个的投票轮次，则加入选票池
                        //判断是否能过半选举出leader ，如果是，则checkLeader
                        /*checkLeader：
                         * 【是否能选举出leader】and
                         * 【(如果投票leader为自身，且轮次一致） or
                         * （如果所选leader不是自身信息在outofelection不为空，且leader的ServerState状态已经为leader）】
                         *
                         */
                        if(n.electionEpoch == logicalclock.get()){
                            recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));
                            if(termPredicate(recvset, new Vote(n.leader,
                                            n.zxid, n.electionEpoch, n.peerEpoch, n.state))
                                            && checkLeader(outofelection, n.leader, n.electionEpoch)) {
                                self.setPeerState((n.leader == self.getId()) ?
                                        ServerState.LEADING: learningState());

                                Vote endVote = new Vote(n.leader, n.zxid, n.peerEpoch);
                                leaveInstance(endVote);
                                return endVote;
                            }
                        }

                        /*
                         * Before joining an established ensemble, verify that
                         * a majority are following the same leader.
                         * Only peer epoch is used to check that the votes come
                         * from the same ensemble. This is because there is at
                         * least one corner case in which the ensemble can be
                         * created with inconsistent zxid and election epoch
                         * info. However, given that only one ensemble can be
                         * running at a single point in time and that each
                         * epoch is used only once, using only the epoch to
                         * compare the votes is sufficient.
                         *
                         * @see https://issues.apache.org/jira/browse/ZOOKEEPER-1732
                         */
                        outofelection.put(n.sid, new Vote(n.leader,
                                IGNOREVALUE, IGNOREVALUE, n.peerEpoch, n.state));
                        //说明此时 集群中存在别的轮次选举已经有了选举结果
                        //比对outofelection选票池，是否能结束选举，同时检查leader信息
                        //如果能结束选举 接收到的选票产生的leader通过checkLeader为true，则更新当前节点信息
                        if (termPredicate(outofelection, new Vote(n.leader,
                                IGNOREVALUE, IGNOREVALUE, n.peerEpoch, n.state))
                                && checkLeader(outofelection, n.leader, IGNOREVALUE)) {
                            synchronized(this){
                                logicalclock.set(n.electionEpoch);
                                self.setPeerState((n.leader == self.getId()) ?
                                        ServerState.LEADING: learningState());
                            }
                            Vote endVote = new Vote(n.leader, n.zxid, n.peerEpoch);
                            leaveInstance(endVote);
                            return endVote;
                        }
                        break;
                    default:
                        LOG.warn("Notification state unrecoginized: " + n.state
                              + " (n.state), " + n.sid + " (n.sid)");
                        break;
                    }
                } else {
                    if (!validVoter(n.leader)) {
                        LOG.warn("Ignoring notification for non-cluster member sid {} from sid {}", n.leader, n.sid);
                    }
                    if (!validVoter(n.sid)) {
                        LOG.warn("Ignoring notification for sid {} from non-quorum member sid {}", n.leader, n.sid);
                    }
                }
            }
            return null;
        } finally {
            try {
                if(self.jmxLeaderElectionBean != null){
                    MBeanRegistry.getInstance().unregister(
                            self.jmxLeaderElectionBean);
                }
            } catch (Exception e) {
                LOG.warn("Failed to unregister with JMX", e);
            }
            self.jmxLeaderElectionBean = null;
            LOG.debug("Number of connection processing threads: {}",
                    manager.getConnectionThreadCount());
        }
    }
```

说明：该函数用于开始新一轮的Leader选举，其首先会将逻辑时钟自增，然后更新本服务器的选票信息（初始化选票），之后将选票信息放入sendqueue等待发送给其他服务器

之后每台服务器会不断地从recvqueue队列中获取外部选票。如果服务器发现无法获取到任何外部投票，就立即确认自己是否和集群中其他服务器保持着有效的连接，如果没有连接，则马上建立连接，如果已经建立了连接，则再次发送自己当前的内部投票

在发送完初始化选票之后，接着开始处理外部投票。在处理外部投票时，会根据选举轮次来进行不同的处理。　　

　　　　**· 外部投票的选举轮次大于内部投票**。若服务器自身的选举轮次落后于该外部投票对应服务器的选举轮次，那么就会立即更新自己的选举轮次(logicalclock)，并且清空所有已经收到的投票，然后使用初始化的投票来进行PK以确定是否变更内部投票。最终再将内部投票发送出去。

　　　　**· 外部投票的选举轮次小于内部投****票**。若服务器接收的外选票的选举轮次落后于自身的选举轮次，那么Zookeeper就会直接忽略该外部投票，不做任何处理。

　　　　**· 外部投票的选举轮次等于内部投票**。此时可以开始进行选票PK，如果消息中的选票更优，则需要更新本服务器内部选票，再发送给其他服务器。

　　之后再对选票进行归档操作，无论是否变更了投票，都会将刚刚收到的那份外部投票放入选票集合recvset中进行归档，其中recvset用于记录当前服务器在本轮次的Leader选举中收到的所有外部投票，然后开始统计投票，统计投票是为了统计集群中是否已经有过半的服务器认可了当前的内部投票，如果确定已经有过半服务器认可了该投票，然后再进行最后一次确认，判断是否又有更优的选票产生，若无，则终止投票，然后最终的选票

若选票中的服务器状态为FOLLOWING或者LEADING时，其大致步骤会判断选举周期是否等于逻辑时钟，归档选票，是否已经完成了Leader选举，设置服务器状态，修改逻辑时钟等于选举周期，返回最终选票



lookForLeader方法中把当前选票和收到的选举进行不断的比对和更新，最终选出leader，其中比对选票的方法为totalOrderPredicate(): 其中的比对投票信息方式为：

　　1.　　首先判断epoch(选举轮次)，也就是选择epoch值更大的节点；如果收到的epoch更大，则当前阶段落后，更新自己的epoch，否则丢弃。

　　2.　　如果同于轮次中，则选择zxid更大的节点，因为zxid越大说明数据越新。

　　3.　　如果同一轮次，且zxid一样，则选择serverId最大的节点。

　　综上3点可理解为越大越棒！

totalOrderPredicate():

```
protected boolean totalOrderPredicate(long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {
        LOG.debug("id: " + newId + ", proposed id: " + curId + ", zxid: 0x" +
                Long.toHexString(newZxid) + ", proposed zxid: 0x" + Long.toHexString(curZxid));
        if(self.getQuorumVerifier().getWeight(newId) == 0){
            return false;
        }

        /*
         * We return true if one of the following three cases hold:
         * 1- New epoch is higher
         * 2- New epoch is the same as current epoch, but new zxid is higher
         * 3- New epoch is the same as current epoch, new zxid is the same
         *  as current zxid, but server id is higher.
         */

        return ((newEpoch > curEpoch) ||
                ((newEpoch == curEpoch) &&
                ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId)))));
    }
```

说明：该函数将接收的投票与自身投票进行PK，查看是否消息中包含的服务器id是否更优，其按照epoch、zxid、id的优先级进行PK。

```
   protected boolean termPredicate(
            HashMap<Long, Vote> votes,
            Vote vote) {

        HashSet<Long> set = new HashSet<Long>();

        /*
         * First make the views consistent. Sometimes peers will have
         * different zxids for a server depending on timing.
         */
        for (Map.Entry<Long,Vote> entry : votes.entrySet()) { // 遍历已经接收的投票集合
            if (vote.equals(entry.getValue())){ // 将等于当前投票的项放入set
                set.add(entry.getKey());
            }
        }

        //统计set，查看投某个id的票数是否超过一半
        return self.getQuorumVerifier().containsQuorum(set);
    }
```

说明：该函数用于判断Leader选举是否结束，即是否有一半以上的服务器选出了相同的Leader，其过程是将收到的选票与当前选票进行对比，选票相同的放入同一个集合，之后判断选票相同的集合是否超过了半数。

```
    protected boolean checkLeader(
            HashMap<Long, Vote> votes,
            long leader,
            long electionEpoch){
        
        boolean predicate = true;

        /*
         * If everyone else thinks I'm the leader, I must be the leader.
         * The other two checks are just for the case in which I'm not the
         * leader. If I'm not the leader and I haven't received a message
         * from leader stating that it is leading, then predicate is false.
         */

        if(leader != self.getId()){ // 自己不为leader
            if(votes.get(leader) == null) predicate = false; // 还未选出leader
            else if(votes.get(leader).getState() != ServerState.LEADING) predicate = false; // 选出的leader还未给出ack信号，其他服务器还不知道leader
        } else if(logicalclock != electionEpoch) { // 逻辑时钟不等于选举周期
            predicate = false;
        } 

        return predicate;
    }
```

说明：该函数检查是否已经完成了Leader的选举，此时Leader的状态应该是LEADING状态。

## leader和follower同步

zookeeper集群启动的时候，首先读取配置，接着开始选举，选举完成以后，每个server根据选举的结果设置自己的角色，角色设置完成后leader需要和所有的follower同步。上面一篇介绍了leader选举过程，这篇接着介绍启动过程中的leader和follower同步过程。

server刚启动的时候都处于LOOKING状态，选举完成后根据选举结果和对应配置进入对应的状态，设置状态的方法是：

```java
private void setPeerState(long proposedLeader, SyncedLearnerTracker voteSet) {
    ServerState ss = (proposedLeader == self.getId()) ?
            ServerState.LEADING: learningState();
    self.setPeerState(ss);
    if (ss == ServerState.LEADING) {
        leadingVoteSet = voteSet;
    }
}
```

1. 如果当前server.myId等于选举出的leader的myId——也就是proposedLeader，则当前server就是leader，设置peerState为ServerState.LEADING
2. 否则判断当前server的具体角色，因为follower和observer都是learner，需要根据各自的配置来决定该server的状态(配置文件里面的key是peerType，可选的值是participant、observer，如果不配置learnerType默认是LearnerType.PARTICIPANT)
   1. 如果配置的learnerType是LearnerType.PARTICIPANT，则状态为ServerState.FOLLOWING
   2. 否则，状态为ServerState.OBSERVING

### 准备同步

leader开始工作的入口就是leader.lead方法，这里的leader是Leader的实例

准备的过程是：

- 创建leader的实例，Leader，构造方法中传入LeaderZooKeeperServer的实例
- 调用leader.lead
  - 加载ZKDatabase
  - 监听指定的端口（配置的用来监听learner连接请求的端口，配置的第一个冒号后的端口），接收来自follower的请求
  - while循环，检查当前选举的状态是否发生变化需要重新进行选举

同时，follower设置完自己的状态后，也开始进行类似leader的工作

- 创建follower，也就是Follower的实例，同时创建FollowerZooKeeperServer
- 建立和leader的连接

### 进行同步

同步的总体过程如下：![Zookeeper同步过程](png\zookeeper\Zookeeper同步过程.png)

在准备阶段完成follower连接到leader，具备通信状态

1. leader阻塞等待follower发来的第一个packet
   1. 校验packet类型是否是Leader.FOLLOWERINFO或者Leader.OBSERVERINFO
   2. 读取learner信息
      1. sid
      2. protocolVersion
      3. 校验follower的version不能比leader的version还要新
2. leader发送packet(Leader.LEADERINFO)给follower
3. follower收到Leader.LEADERINFO后给leader回复Leader.ACKEPOCH
4. leader根据follower ack的packet内容来决定同步的策略
   1. lastProcessedZxid == peerLastZxid，leader的zxid和follower的相同
   2. peerLastZxid > maxCommittedLog && !isPeerNewEpochZxid follower超前，删除follower多出的txlog部分
   3. (maxCommittedLog >= peerLastZxid) && (minCommittedLog <= peerLastZxid) follower落后于leader，处于leader的中间 同步(peerLaxtZxid, maxZxid]之间的commitlog给follower
   4. peerLastZxid < minCommittedLog && txnLogSyncEnabled follower落后于leader，使用txlog和commitlog同步给follower
   5. 接下来leader会不断的发送packet给follower，follower处理leader发来的每个packet
5. 同步完成后follower回复ack给leader
6. leader、follower进入正式处理客户端请求的while循环

zookeeper为了保证启动后leader和follower的数据一致，在启动的时候就进行数据同步，leader与follower数据传输的端口和leader选举的端口不一样。数据同步完成后就可以接受client的请求进行处理了。

## Session建立

### 主要流程

- 服务端启动，客户端启动
- 客户端发起socket连接
- 服务端accept socket连接，socket连接建立
- 客户端发送ConnectRequest给server
- server收到后初始化ServerCnxn，代表一个和客户端的连接，即session，server发送ConnectResponse给client
- client处理ConnectResponse，session建立完成

### 客户发起连接

#### 和server建立socket连接

客户端要发起连接要先启动，不论是使用curator client还是zkClient，都是初始化`org.apache.zookeeper.ZooKeeper#ZooKeeper`。

Zookeeper初始化的主要工作是初始化自己的一些关键组件

- Watcher，外部构造好传入
- 初始化StaticHostProvider，决定客户端选择连接哪一个server
- ClientCnxn，客户端网络通信的组件，主要启动逻辑就是启动这个类

ClientCnxn包含两个线程

- SendThread，负责client端消息的发送和接收
- EventThread，负责处理event

ClientCnxn初始化的过程就是初始化启动这两个线程，客户端发起连接的主要逻辑在SendThread线程中

```java
// org.apache.zookeeper.ClientCnxn.SendThread#run
@Override
public void run() {
    clientCnxnSocket.introduce(this,sessionId);
    clientCnxnSocket.updateNow();
    clientCnxnSocket.updateLastSendAndHeard();
    int to;
    long lastPingRwServer = System.currentTimeMillis();
    final int MAX_SEND_PING_INTERVAL = 10000; //10 seconds
    while (state.isAlive()) {
        try {
            // client是否连接到server，如果没有连接到则连接server
            if (!clientCnxnSocket.isConnected()) {
                if(!isFirstConnect){
                    try {
                        Thread.sleep(r.nextInt(1000));
                    } catch (InterruptedException e) {
                        LOG.warn("Unexpected exception", e);
                    }
                }
                // don't re-establish connection if we are closing
                if (closing || !state.isAlive()) {
                    break;
                }
                // 这个里面去连接server
                startConnect();
                clientCnxnSocket.updateLastSendAndHeard();
            }

            // 省略中间代码...
            clientCnxnSocket.doTransport(to, pendingQueue, outgoingQueue, ClientCnxn.this);
            // 省略中间代码...
}
```

SendThread#run是一个while循环，只要client没有被关闭会一直循环，每次循环判断当前client是否连接到server，如果没有则发起连接，发起连接调用了startConnect

```java
private void startConnect() throws IOException {
    state = States.CONNECTING;

    InetSocketAddress addr;
    if (rwServerAddress != null) {
        addr = rwServerAddress;
        rwServerAddress = null;
    } else {
        // 通过hostProvider来获取一个server地址
        addr = hostProvider.next(1000);
    }
	// 省略中间代码...

    // 建立client与server的连接
    clientCnxnSocket.connect(addr);
}
```

到这里client发起了socket连接，server监听的端口收到client的连接请求后和client建立连接。

#### 通过一个request来建立session连接

socket连接建立后，client会向server发送一个ConnectRequest来建立session连接。两种情况会发送ConnectRequest

- 在上面的connect方法中会判断是否socket已经建立成功，如果建立成功就会发送ConnectRequest
- 如果socket没有立即建立成功（socket连接建立是异步的），则发送这个packet要延后到doTransport中

发送ConnectRequest是在下面的方法中

```java
// org.apache.zookeeper.ClientCnxn.SendThread#primeConnection
void primeConnection() throws IOException {
    LOG.info("Socket connection established to "
             + clientCnxnSocket.getRemoteSocketAddress()
             + ", initiating session");
    isFirstConnect = false;
    long sessId = (seenRwServerBefore) ? sessionId : 0;
    ConnectRequest conReq = new ConnectRequest(0, lastZxid,
                                               sessionTimeout, sessId, sessionPasswd);
    	// 省略中间代码...
    	// 将conReq封装为packet放入outgoingQueue等待发送
        outgoingQueue.addFirst(new Packet(null, null, conReq,
                                          null, null, readOnly));
    }
    clientCnxnSocket.enableReadWriteOnly();
    if (LOG.isDebugEnabled()) {
        LOG.debug("Session establishment request sent on "
                  + clientCnxnSocket.getRemoteSocketAddress());
    }
}
```

请求中带的参数

- lastZxid：上一个事务的id
- sessionTimeout：client端配置的sessionTimeout
- sessId：sessionId，如果之前建立过连接取的是上一次连接的sessionId
- sessionPasswd：session的密码

### 服务端创建session

#### 和client建立socket连接

在server启动的过程中除了会启动用于选举的网络组件还会启动用于处理client请求的网络组件

```java
org.apache.zookeeper.server.NIOServerCnxnFactory
```

主要启动了三个线程：

- AcceptThread：用于接收client的连接请求，建立连接后交给SelectorThread线程处理
- SelectorThread：用于处理读写请求
- ConnectionExpirerThread：检查session连接是否过期

client发起socket连接的时候，server监听了该端口，接收到client的连接请求，然后把建立练级的SocketChannel放入队列里面，交给SelectorThread处理

```java
// org.apache.zookeeper.server.NIOServerCnxnFactory.SelectorThread#addAcceptedConnection
public boolean addAcceptedConnection(SocketChannel accepted) {
    if (stopped || !acceptedQueue.offer(accepted)) {
        return false;
    }
    wakeupSelector();
    return true;
}
```

#### 建立session连接

SelectorThread是一个不断循环的线程，每次循环都会处理刚刚建立的socket连接

```java
// org.apache.zookeeper.server.NIOServerCnxnFactory.SelectorThread#run
while (!stopped) {
    try {
        select();
        //  处理对立中的socket
        processAcceptedConnections();
        processInterestOpsUpdateRequests();
    } catch (RuntimeException e) {
        LOG.warn("Ignoring unexpected runtime exception", e);
    } catch (Exception e) {
        LOG.warn("Ignoring unexpected exception", e);
    }
}

// org.apache.zookeeper.server.NIOServerCnxnFactory.SelectorThread#processAcceptedConnections
private void processAcceptedConnections() {
    SocketChannel accepted;
    while (!stopped && (accepted = acceptedQueue.poll()) != null) {
        SelectionKey key = null;
        try {
            // 向该socket注册读事件
            key = accepted.register(selector, SelectionKey.OP_READ);
            // 创建一个NIOServerCnxn维护session
            NIOServerCnxn cnxn = createConnection(accepted, key, this);
            key.attach(cnxn);
            addCnxn(cnxn);
        // 省略中间代码...
    }
}
```

说了这么久，我们说的session究竟是什么还没有解释，session中文翻译是会话，在这里就是zk的server和client维护的一个具有一些特别属性的网络连接，网络连接这里就是socket连接，一些特别的属性包括

- sessionId：唯一标示一个会话
- sessionTimeout：这个连接的超时时间，超过这个时间server就会把连接断开

所以session建立的两步就是

- 建立socket连接
- client发起建立session请求，server建立一个实例来维护这个连接

server收到ConnectRequest之后，按照正常处理io的方式处理这个request，server端的主要操作是

- 反序列化为ConnectRequest
- 根据request中的sessionId来判断是新的session连接还是session重连
  - 如果是新连接
    - 生成sessionId
    - 创建新的SessionImpl并放入org.apache.zookeeper.server.SessionTrackerImpl#sessionExpiryQueue
    - 封装该请求为新的request在processorChain中传递，最后交给FinalRequestProcessor处理
  - 如果是重连
    - 关闭sessionId对应的原来的session
      - 关闭原来的socket连接
      - sessionImp会在sessionExpiryQueue中由于过期被清理
    - 重新打开一个session
      - 将原来的sessionId设置到当前的NIOServerCnxn实例中，作为新的连接的sessionId
      - 校验密码是否正确密码错误的时候直接返回给客户端，不可用的session
      - 密码正确的话，新建SessionImpl
      - 返回给客户端sessionId

其中有一个session生成算法我们来看下

```java
public static long initializeNextSession(long id) {
    // sessionId是long类型，共8个字节，64位
    long nextSid;
    // 取时间戳的的低40位作为初始化sessionId的第16-55位，这里使用的是无符号右移，不会出现负数
    nextSid = (Time.currentElapsedTime() << 24) >>> 8;
    // 使用serverId（配置文件中指定的myid）作为高8位
    nextSid =  nextSid | (id <<56);
    // nextSid为long的最小值，这中情况不可能出现，这里只是作为一个case列在这里
    if (nextSid == EphemeralType.CONTAINER_EPHEMERAL_OWNER) {
        ++nextSid;  // this is an unlikely edge case, but check it just in case
    }
    return nextSid;
}
```

初始化sessionId的组成

myid(1字节)+截取的时间戳低40位(5个字节)+2个字节(初始化都是0)

每个server再基于这个id不断自增，这样的算法就保证了每个server的sessionId是全局唯一的。

## 处理写请求

### Leader消息处理

观察者、跟随者、和领导者启动server，分别为，LeaderZooKeeperServer,FollowerZooKeeperServer,ObserverZooKeeperServer. Leader的消息处处理器链为LeaderRequestProcessor->PrepRequestProcessor->ProposalRequestProcessor->CommitProcessor->ToBeAppliedRequestProcessor->FinalRequestProcessor; Follower的消息处理器链为SendAckRequestProcessor->SyncRequestProcessor->FollowerRequestProcessor->CommitProcessor->FinalRequestProcessor. Observer的消息处处理器链为SyncRequestProcessor->ObserverRequestProcessor->CommitProcessor->FinalRequestProcessor;

LeaderRequestProcessor处理器主要做的本地会话检查，并更新会话保活信息。 PrepRequestProcessor处理消息，首先添加到内部的提交请求队列，然后启动线程预处理请求。 预处理消息，主要是针对事务性的CUD， 则构建响应的请求，比如CreateRequest，SetDataRequest等。针对非事务性R，则检查会话的有效性。 事务性预处理请求，主要是将请求包装成事件变更记录ZooKeeperServer，并保存到Zookeeper的请求变更记录集outstandingChanges中。 ProposalRequestProcessor处理器，主要处理同步请求消息，针对同步请求，则发送消息到响应的server。 CommitProcessor，主要是过滤出需要提交的请求，比如CRUD等，并交由下一个处理器处理。 ToBeAppliedRequestProcessor处理器，主要是保证提议为最新。 FinalRequestProcessor首先由ZooKeeperServer处理CUD相关的请求操作，针对R类的相关操作，直接查询ZooKeeperServer的内存数据库。 ZooKeeperServer处理CUD操作，委托表给ZKDatabase，ZKDatabase委托给DataTree， DataTree根据CUD相关请求操作，CUD相应路径的 DataNode。针对R类操作，获取dataTree的DataNode的相关信息。

### Follower消息处理

### Observer消息处理

针对跟随者，SendAckRequestProcessor处理器，针对非同步操作，回复ACK。 SyncRequestProcessor处理器从请求队列拉取请求,针对刷新队列不为空的情况，如果请求队列为空，则提交请求日志，并刷新到磁盘，否则根据日志计数器和快照计数器计算是否需要拍摄快照。 FollowerRequestProcessor处理器，从请求队列拉取请求，如果请求为同步请求，则添加请求到同步队列, 并转发请求给leader，如果为CRUD相关的操作，直接转发请求给leader。

针对观察者，观察者请求处理器，从请求队列拉取请求，如果请求为同步请求，则添加请求到同步队列, 并转发请求给leader，如果为CRUD相关的操作，直接转发请求给leader。

### 处理写请求过程

zk为了保证分布式数据一致性，使用ZAB协议，在客户端发起一次写请求的时候时候，假设该请求请求到的是follower，follower不会直接处理这个请求，而是转发给leader，由leader发起投票决定该请求最终能否执行成功，所以整个过程client、被请求的follower、leader、其他follower都参与其中。以创建一个节点为例，总体流程如下

![Zookeeper处理写请求](png\zookeeper\Zookeeper处理写请求.png)

从图中可以看出，创建流程为

1. follower接受请求，解析请求
2. follower通过FollowerRequestProcessor将请求转发给leader
3. leader接收请求
4. leader发送proposal给follower
5. follower收到请求记录txlog、snapshot
6. follower发送ack给leader
7. leader收到ack后进行commit，并且通知所有的learner，发送commit packet给所有的learner

### Zookeeper的服务器

这里说的follower、leader都是server，在zk里面server总共有这么几种

![Zookeeper中的Server](png\zookeeper\Zookeeper中的Server.png)

由于server角色不同，对于请求所做的处理不同，每种server包含的processor也不同，下面细说下具体有哪些processor。

　ZooKeeperServer，为所有服务器的父类，其请求处理链为PrepRequestProcessor -> SyncRequestProcessor -> FinalRequestProcessor。

　　QuorumZooKeeperServer，其是所有参与选举的服务器的父类，是抽象类，其继承了ZooKeeperServer类。

　　LeaderZooKeeperServer，Leader服务器，继承了QuorumZooKeeperServer类，其请求处理链为PrepRequestProcessor -> ProposalRequestProcessor -> CommitProcessor -> Leader.ToBeAppliedRequestProcessor -> FinalRequestProcessor。

　　LearnerZooKeeper，其是Learner服务器的父类，为抽象类，也继承了QuorumZooKeeperServer类。

　　FollowerZooKeeperServer，Follower服务器，继承了LearnerZooKeeper，其请求处理链为FollowerRequestProcessor -> CommitProcessor -> FinalRequestProcessor。

　　ObserverZooKeeperServer，Observer服务器，继承了LearnerZooKeeper。

　　ReadOnlyZooKeeperServer，只读服务器，不提供写服务，继承QuorumZooKeeperServer，其处理链的第一个处理器为ReadOnlyRequestProcessor。

#### ZooKeeperServer源码分析

```
public class ZooKeeperServer implements SessionExpirer, ServerStats.Provider {}
```

说明：ZooKeeperServer是ZooKeeper中所有服务器的父类，其实现了Session.Expirer和ServerStats.Provider接口，SessionExpirer中定义了expire方法（表示会话过期）和getServerId方法（表示获取服务器ID），而Provider则主要定义了获取服务器某些数据的方法。

内部类

类的内部类

  　　1. DataTreeBuilder类

```
    public interface DataTreeBuilder {
        // 构建DataTree
        public DataTree build();
    }
```

　　说明：其定义了构建树DataTree的接口。

  　　2. BasicDataTreeBuilder类　　

```
    static public class BasicDataTreeBuilder implements DataTreeBuilder {
        public DataTree build() {
            return new DataTree();
        }
    }
```

　　说明：实现DataTreeBuilder接口，返回新创建的树DataTree。

  　　3. MissingSessionException类　　

```
    public static class MissingSessionException extends IOException {
        private static final long serialVersionUID = 7467414635467261007L;

        public MissingSessionException(String msg) {
            super(msg);
        }
    }
```

　　说明：表示会话缺失异常。

  　　4. ChangeRecord类　

```
    static class ChangeRecord {
        ChangeRecord(long zxid, String path, StatPersisted stat, int childCount,
                List<ACL> acl) {
            // 属性赋值
            this.zxid = zxid;
            this.path = path;
            this.stat = stat;
            this.childCount = childCount;
            this.acl = acl;
        }
        
        // zxid
        long zxid;

        // 路径
        String path;

        // 统计数据
        StatPersisted stat; /* Make sure to create a new object when changing */

        // 子节点个数
        int childCount;

        // ACL列表
        List<ACL> acl; /* Make sure to create a new object when changing */

        @SuppressWarnings("unchecked")
        // 拷贝
        ChangeRecord duplicate(long zxid) {
            StatPersisted stat = new StatPersisted();
            if (this.stat != null) {
                DataTree.copyStatPersisted(this.stat, stat);
            }
            return new ChangeRecord(zxid, path, stat, childCount,
                    acl == null ? new ArrayList<ACL>() : new ArrayList(acl));
        }
    }
```

　　说明：ChangeRecord数据结构是用于方便PrepRequestProcessor和FinalRequestProcessor之间进行信息共享，其包含了一个拷贝方法duplicate，用于返回属性相同的ChangeRecord实例。

类的属性　　

　　说明：类中包含了心跳频率，会话跟踪器（处理会话）、事务日志快照、内存数据库、请求处理器、未处理的ChangeRecord、服务器统计信息等。

　　类的构造函数

  　　1. ZooKeeperServer()型构造函数　　

```
    public ZooKeeperServer() {
        serverStats = new ServerStats(this);
    }
```

　　说明：其只初始化了服务器的统计信息。

  　　2. ZooKeeperServer(FileTxnSnapLog, int, int, int, DataTreeBuilder, ZKDatabase)型构造函数　　

```
    public ZooKeeperServer(FileTxnSnapLog txnLogFactory, int tickTime,
            int minSessionTimeout, int maxSessionTimeout,
            DataTreeBuilder treeBuilder, ZKDatabase zkDb) {
        // 给属性赋值
        serverStats = new ServerStats(this);
        this.txnLogFactory = txnLogFactory;
        this.zkDb = zkDb;
        this.tickTime = tickTime;
        this.minSessionTimeout = minSessionTimeout;
        this.maxSessionTimeout = maxSessionTimeout;
        
        LOG.info("Created server with tickTime " + tickTime
                + " minSessionTimeout " + getMinSessionTimeout()
                + " maxSessionTimeout " + getMaxSessionTimeout()
                + " datadir " + txnLogFactory.getDataDir()
                + " snapdir " + txnLogFactory.getSnapDir());
    }
```

　　说明：该构造函数会初始化服务器统计数据、事务日志工厂、心跳时间、会话时间（最短超时时间和最长超时时间）。

  　　3. ZooKeeperServer(FileTxnSnapLog, int, DataTreeBuilder)型构造函数　　

```
    public ZooKeeperServer(FileTxnSnapLog txnLogFactory, int tickTime,
            DataTreeBuilder treeBuilder) throws IOException {
        this(txnLogFactory, tickTime, -1, -1, treeBuilder,
                new ZKDatabase(txnLogFactory));
    }
```

　　说明：其首先会生成ZooKeeper内存数据库后，然后调用第二个构造函数进行初始化操作。

  　　4. ZooKeeperServer(File, File, int)型构造函数　

```
    public ZooKeeperServer(File snapDir, File logDir, int tickTime)
            throws IOException {
        this( new FileTxnSnapLog(snapDir, logDir),
                tickTime, new BasicDataTreeBuilder());
    }
```

　　说明：其会调用同名构造函数进行初始化操作。

  　　5. ZooKeeperServer(FileTxnSnapLog, DataTreeBuilder)型构造函数　　

```
    public ZooKeeperServer(FileTxnSnapLog txnLogFactory,
            DataTreeBuilder treeBuilder)
        throws IOException
    {
        this(txnLogFactory, DEFAULT_TICK_TIME, -1, -1, treeBuilder,
                new ZKDatabase(txnLogFactory));
    }
```

　　说明：其生成内存数据库之后再调用同名构造函数进行初始化操作。

　核心函数分析

loadData函数　

```
    public void loadData() throws IOException, InterruptedException {
        /*
         * When a new leader starts executing Leader#lead, it 
         * invokes this method. The database, however, has been
         * initialized before running leader election so that
         * the server could pick its zxid for its initial vote.
         * It does it by invoking QuorumPeer#getLastLoggedZxid.
         * Consequently, we don't need to initialize it once more
         * and avoid the penalty of loading it a second time. Not 
         * reloading it is particularly important for applications
         * that host a large database.
         * 
         * The following if block checks whether the database has
         * been initialized or not. Note that this method is
         * invoked by at least one other method: 
         * ZooKeeperServer#startdata.
         *  
         * See ZOOKEEPER-1642 for more detail.
         */
        if(zkDb.isInitialized()){ // 内存数据库已被初始化
            // 设置为最后处理的Zxid
            setZxid(zkDb.getDataTreeLastProcessedZxid());
        }
        else { // 未被初始化，则加载数据库
            setZxid(zkDb.loadDataBase());
        }
        
        // Clean up dead sessions
        LinkedList<Long> deadSessions = new LinkedList<Long>();
        for (Long session : zkDb.getSessions()) { // 遍历所有的会话
            if (zkDb.getSessionWithTimeOuts().get(session) == null) { // 删除过期的会话
                deadSessions.add(session);
            }
        }
        // 完成DataTree的初始化
        zkDb.setDataTreeInit(true);
        for (long session : deadSessions) { // 遍历过期会话
            // XXX: Is lastProcessedZxid really the best thing to use?
            // 删除会话
            killSession(session, zkDb.getDataTreeLastProcessedZxid());
        }
    }
```

　　说明：该函数用于加载数据，其首先会判断内存库是否已经加载设置zxid，之后会调用killSession函数删除过期的会话，killSession会从sessionTracker中删除session，并且killSession最后会调用DataTree的killSession函数，其源码如下　

```
    void killSession(long session, long zxid) {
        // the list is already removed from the ephemerals
        // so we do not have to worry about synchronizing on
        // the list. This is only called from FinalRequestProcessor
        // so there is no need for synchronization. The list is not
        // changed here. Only create and delete change the list which
        // are again called from FinalRequestProcessor in sequence.
        // 移除session，并获取该session对应的所有临时节点
        HashSet<String> list = ephemerals.remove(session);
        if (list != null) {
            for (String path : list) { // 遍历所有临时节点
                try {
                    // 删除路径对应的节点
                    deleteNode(path, zxid);
                    if (LOG.isDebugEnabled()) {
                        LOG
                                .debug("Deleting ephemeral node " + path
                                        + " for session 0x"
                                        + Long.toHexString(session));
                    }
                } catch (NoNodeException e) {
                    LOG.warn("Ignoring NoNodeException for path " + path
                            + " while removing ephemeral for dead session 0x"
                            + Long.toHexString(session));
                }
            }
        }
    }
```

　　说明：DataTree的killSession函数的逻辑首先移除session，然后取得该session下的所有临时节点，然后逐一删除临时节点。

submit函数　

```
    public void submitRequest(Request si) {
        if (firstProcessor == null) { // 第一个处理器为空
            synchronized (this) {
                try {
                    while (!running) { // 直到running为true，否则继续等待
                        wait(1000);
                    }
                } catch (InterruptedException e) {
                    LOG.warn("Unexpected interruption", e);
                }
                if (firstProcessor == null) {
                    throw new RuntimeException("Not started");
                }
            }
        }
        try {
            touch(si.cnxn);
            // 是否为合法的packet
            boolean validpacket = Request.isValid(si.type);
            if (validpacket) { 
                // 处理请求
                firstProcessor.processRequest(si);
                if (si.cnxn != null) {
                    incInProcess();
                }
            } else {
                LOG.warn("Received packet at server of unknown type " + si.type);
                new UnimplementedRequestProcessor().processRequest(si);
            }
        } catch (MissingSessionException e) {
            if (LOG.isDebugEnabled()) {
                LOG.debug("Dropping request: " + e.getMessage());
            }
        } catch (RequestProcessorException e) {
            LOG.error("Unable to process request:" + e.getMessage(), e);
        }
    }
```

　　说明：当firstProcessor为空时，并且running标志为false时，其会一直等待，直到running标志为true，之后调用touch函数判断session是否存在或者已经超时，之后判断请求的类型是否合法，合法则使用请求处理器进行处理。

processConnectRequest函数　　

　　说明：其首先将传递的ByteBuffer进行反序列化，转化为相应的ConnectRequest，之后进行一系列判断（可能抛出异常），然后获取并判断该ConnectRequest中会话id是否为0，若为0，则表示可以创建会话，否则，重新打开会话。

processPacket函数　

　　说明：该函数首先将传递的ByteBuffer进行反序列，转化为相应的RequestHeader，然后根据该RequestHeader判断是否需要认证，若认证失败，则构造认证失败的响应并发送给客户端，然后关闭连接，并且再补接收任何packet。若认证成功，则构造认证成功的响应并发送给客户端。若不需要认证，则再判断其是否为SASL类型，若是，则进行处理，然后构造响应并发送给客户端，否则，构造请求并且提交请求。

#### LeaderZooKeeperServer源码分析

　　 类的继承关系　

```
public class LeaderZooKeeperServer extends QuorumZooKeeperServer {}
```

　　说明：LeaderZooKeeperServer继承QuorumZooKeeperServer抽象类，其会继承ZooKeeperServer中的很多方法。

　　类的属性　　

```
public class LeaderZooKeeperServer extends QuorumZooKeeperServer {
    // 提交请求处理器
    CommitProcessor commitProcessor;
}
```

　　说明：其只有一个CommitProcessor类，表示提交请求处理器，其在处理链中的位置位于ProposalRequestProcessor之后，ToBeAppliedRequestProcessor之前。

　类的构造函数

```
    LeaderZooKeeperServer(FileTxnSnapLog logFactory, QuorumPeer self,
            DataTreeBuilder treeBuilder, ZKDatabase zkDb) throws IOException {
        super(logFactory, self.tickTime, self.minSessionTimeout,
                self.maxSessionTimeout, treeBuilder, zkDb, self);
    }
```

　　说明：其直接调用父类QuorumZooKeeperServer的构造函数，然后再调用ZooKeeperServer的构造函数，逐级构造。

　核心函数分析

  　　1. setupRequestProcessors函数　　

```
    protected void setupRequestProcessors() {
        // 创建FinalRequestProcessor
        RequestProcessor finalProcessor = new FinalRequestProcessor(this);
        // 创建ToBeAppliedRequestProcessor
        RequestProcessor toBeAppliedProcessor = new Leader.ToBeAppliedRequestProcessor(
                finalProcessor, getLeader().toBeApplied);
        // 创建CommitProcessor
        commitProcessor = new CommitProcessor(toBeAppliedProcessor,
                Long.toString(getServerId()), false);
        // 启动CommitProcessor
        commitProcessor.start();
        // 创建ProposalRequestProcessor
        ProposalRequestProcessor proposalProcessor = new ProposalRequestProcessor(this,
                commitProcessor);
        // 初始化ProposalProcessor
        proposalProcessor.initialize();
        // firstProcessor为PrepRequestProcessor
        firstProcessor = new PrepRequestProcessor(this, proposalProcessor);
        // 启动PrepRequestProcessor
        ((PrepRequestProcessor)firstProcessor).start();
    }
```

　　说明：该函数表示创建处理链，可以看到其处理链的顺序为PrepRequestProcessor -> ProposalRequestProcessor -> CommitProcessor -> Leader.ToBeAppliedRequestProcessor -> FinalRequestProcessor。

  　　2. registerJMX函数　

```
    protected void registerJMX() {
        // register with JMX
        try {
            // 创建DataTreeBean
            jmxDataTreeBean = new DataTreeBean(getZKDatabase().getDataTree());
            // 进行注册
            MBeanRegistry.getInstance().register(jmxDataTreeBean, jmxServerBean);
        } catch (Exception e) {
            LOG.warn("Failed to register with JMX", e);
            jmxDataTreeBean = null;
        }
    }
```

　　说明：该函数用于注册JMX服务，首先使用DataTree初始化DataTreeBean，然后使用DataTreeBean和ServerBean调用register函数进行注册，其源码如下　

```
    public void register(ZKMBeanInfo bean, ZKMBeanInfo parent)
        throws JMException
    {
        // 确保bean不为空
        assert bean != null;
        String path = null;
        if (parent != null) { // parent(ServerBean)不为空
            // 通过parent从bean2Path中获取path
            path = mapBean2Path.get(parent);
            // 确保path不为空
            assert path != null;
        }
        // 补充为完整的路径
        path = makeFullPath(path, parent);
        if(bean.isHidden())
            return;
        // 使用路径来创建名字
        ObjectName oname = makeObjectName(path, bean);
        try {
            // 注册Server
            mBeanServer.registerMBean(bean, oname);
            // 将bean和对应path放入mapBean2Path
            mapBean2Path.put(bean, path);
            // 将name和bean放入mapName2Bean
            mapName2Bean.put(bean.getName(), bean);
        } catch (JMException e) {
            LOG.warn("Failed to register MBean " + bean.getName());
            throw e;
      
```

　　说明：可以看到会通过parent来获取路径，然后创建名字，然后注册bean，之后将相应字段放入mBeanServer和mapBean2Path中，即完成注册过程。

  　　3. unregisterJMX函数　　

```
    protected void unregisterJMX() {
        // unregister from JMX
        try {
            if (jmxDataTreeBean != null) {
                // 取消注册
                MBeanRegistry.getInstance().unregister(jmxDataTreeBean);
            }
        } catch (Exception e) {
            LOG.warn("Failed to unregister with JMX", e);
        }
        jmxDataTreeBean = null;
    }
```

　　说明：该函数用于取消注册JMX服务，其会调用unregister函数，其源码如下　

```
    public void unregister(ZKMBeanInfo bean) {
        if(bean==null)
            return;
        // 获取对应路径
        String path=mapBean2Path.get(bean);
        try {
            // 取消注册
            unregister(path,bean);
        } catch (JMException e) {
            LOG.warn("Error during unregister", e);
        }
        // 从mapBean2Path和mapName2Bean中移除bean
        mapBean2Path.remove(bean);
        mapName2Bean.remove(bean.getName());
    }
```

　　说明：unregister与register的过程恰好相反，是移除bean的过程。

#### **FollowerZooKeeperServer源码分析**

　　类的继承关系　　

```
public class FollowerZooKeeperServer extends LearnerZooKeeperServer {}
```

　　说明：其继承LearnerZooKeeperServer抽象类，角色为Follower。其请求处理链为FollowerRequestProcessor -> CommitProcessor -> FinalRequestProcessor。

　类的属性　　

```
public class FollowerZooKeeperServer extends LearnerZooKeeperServer {
    private static final Logger LOG =
        LoggerFactory.getLogger(FollowerZooKeeperServer.class);
    // 提交请求处理器
    CommitProcessor commitProcessor;
    
    // 同步请求处理器
    SyncRequestProcessor syncProcessor;

    /*
     * Pending sync requests
     */
    // 待同步请求
    ConcurrentLinkedQueue<Request> pendingSyncs;
    
    // 待处理的事务请求
    LinkedBlockingQueue<Request> pendingTxns = new LinkedBlockingQueue<Request>();
}
```

　　说明：FollowerZooKeeperServer中维护着提交请求处理器和同步请求处理器，并且维护了所有待同步请求队列和待处理的事务请求队列。

　　2.3 类的构造函数　　

```
    FollowerZooKeeperServer(FileTxnSnapLog logFactory,QuorumPeer self,
            DataTreeBuilder treeBuilder, ZKDatabase zkDb) throws IOException {
        super(logFactory, self.tickTime, self.minSessionTimeout,
                self.maxSessionTimeout, treeBuilder, zkDb, self);
        // 初始化pendingSyncs
        this.pendingSyncs = new ConcurrentLinkedQueue<Request>();
    }
```

　　说明：其首先调用父类的构造函数，然后初始化pendingSyncs为空队列。

　核心函数分析

  　　1. logRequest函数　　

```
    public void logRequest(TxnHeader hdr, Record txn) {
        // 创建请求
        Request request = new Request(null, hdr.getClientId(), hdr.getCxid(),
                hdr.getType(), null, null);
        // 赋值请求头、事务体、zxid
        request.hdr = hdr;
        request.txn = txn;
        request.zxid = hdr.getZxid();
        if ((request.zxid & 0xffffffffL) != 0) { // zxid不为0，表示本服务器已经处理过请求
            // 则需要将该请求放入pendingTxns中
            pendingTxns.add(request);
        }
        // 使用SyncRequestProcessor处理请求(其会将请求放在队列中，异步进行处理)
        syncProcessor.processRequest(request);
    }
```

　　说明：该函数将请求进行记录（放入到对应的队列中），等待处理。

  　　2. commit函数　

```
    public void commit(long zxid) {
        if (pendingTxns.size() == 0) { // 没有还在等待处理的事务
            LOG.warn("Committing " + Long.toHexString(zxid)
                    + " without seeing txn");
            return;
        }
        // 队首元素的zxid
        long firstElementZxid = pendingTxns.element().zxid;
        if (firstElementZxid != zxid) { // 如果队首元素的zxid不等于需要提交的zxid，则退出程序
            LOG.error("Committing zxid 0x" + Long.toHexString(zxid)
                    + " but next pending txn 0x"
                    + Long.toHexString(firstElementZxid));
            System.exit(12);
        }
        // 从待处理事务请求队列中移除队首请求
        Request request = pendingTxns.remove();
        // 提交该请求
        commitProcessor.commit(request);
    }
```

　　说明：该函数会提交zxid对应的请求（pendingTxns的队首元素），其首先会判断队首请求对应的zxid是否为传入的zxid，然后再进行移除和提交（放在committedRequests队列中）。

  　　3. sync函数　　

```
    synchronized public void sync(){
        if(pendingSyncs.size() ==0){ // 没有需要同步的请求
            LOG.warn("Not expecting a sync.");
            return;
        }
        // 从待同步队列中移除队首请求
        Request r = pendingSyncs.remove();
        // 提交该请求
        commitProcessor.commit(r);
    }
```

　　说明：该函数会将待同步请求队列中的元素进行提交，也是将该请求放入committedRequests队列中。

### follower的processor链

这里的follower是FollowerZooKeeperServer，通过setupRequestProcessors来设置自己的processor链

```java
FollowerRequestProcessor -> CommitProcessor ->FinalRequestProcessor
```

每个processor对应的功能为:

#### FollowerRequestProcessor：

作用：将请求先转发给下一个processor，然后根据不同的OpCode做不同的操作

如果是：sync，先加入org.apache.zookeeper.server.quorum.FollowerZooKeeperServer#pendingSyncs，然后发送给leader

如果是：create等，直接转发

如果是：createSession或者closeSession，如果不是localsession则转发给leader

#### CommitProcessor ：

有一个WorkerService，将请求封装为CommitWorkRequest执行

作用：

转发请求，读请求直接转发给下一个processor

写请求先放在pendingRequests对应的sessionId下的list中，收到leader返回的commitRequest再处理

1. 处理读请求（不会改变服务器状态的请求）
2. 处理committed的写请求（经过leader 处理完成的请求）

维护一个线程池服务WorkerService，每个请求都在单独的线程中处理

1. 每个session的请求必须按顺序执行
2. 写请求必须按照zxid顺序执行
3. 确认一个session中写请求之间没有竞争

#### FinalRequestProcessor：

总是processor chain上最后一个processor

作用：

1. 实际处理事务请求的processor
2. 处理query请求
3. 返回response给客户端

#### SyncRequestProcessor：

作用：

1. 接收leader的proposal进行处理
2. 从org.apache.zookeeper.server.SyncRequestProcessor#queuedRequests中取出请求记录txlog和snapshot
3. 然后加入toFlush，从toFlush中取出请求交给org.apache.zookeeper.server.quorum.SendAckRequestProcessor#flush处理

### leader的processor链

这里的leader就是LeaderZooKeeperServer

通过setupRequestProcessors来设置自己的processor链

```java
PrepRequestProcessor -> ProposalRequestProcessor ->CommitProcessor -> Leader.ToBeAppliedRequestProcessor ->FinalRequestProcessor
```

具体情况可以参看代码：

```
@Override
    protected void setupRequestProcessors() {
        RequestProcessor finalProcessor = new FinalRequestProcessor(this);
        RequestProcessor toBeAppliedProcessor = new Leader.ToBeAppliedRequestProcessor(finalProcessor, getLeader());
        commitProcessor = new CommitProcessor(toBeAppliedProcessor,
                Long.toString(getServerId()), false,
                getZooKeeperServerListener());
        commitProcessor.start();
        ProposalRequestProcessor proposalProcessor = new ProposalRequestProcessor(this,
                commitProcessor);
        proposalProcessor.initialize();
        prepRequestProcessor = new PrepRequestProcessor(this, proposalProcessor);
        prepRequestProcessor.start();
        firstProcessor = new LeaderRequestProcessor(this, prepRequestProcessor);

        setupContainerManager();
    }
```



#### 客户端发起写请求

在客户端启动的时候会创建Zookeeper实例，client会连接到server，后面client在创建节点的时候就可以直接和server通信，client发起创建创建节点请求的过程是：

```java
org.apache.zookeeper.ZooKeeper#create(java.lang.String, byte[], java.util.List<org.apache.zookeeper.data.ACL>, org.apache.zookeeper.CreateMode)
org.apache.zookeeper.ClientCnxn#submitRequest
org.apache.zookeeper.ClientCnxn#queuePacket
```

1. 在`ZooKeeper#create`方法中构造请求的request

2. 在

   ```
   ClientCnxn#queuePacket
   ```

   方法中将request封装到packet中，将packet放入发送队列outgoingQueue中等待发送

   1. SendThread不断从发送队列outgoingQueue中取出packet发送

3. 通过 packet.wait等待server返回

#### follower和leader交互过程

client发出请求后，follower会接收并处理该请求。选举结束后follower确定了自己的角色为follower，一个端口和client通信，一个端口和leader通信。监听到来自client的连接口建立新的session，监听对应的socket上的读写事件，如果client有请求发到follower，follower会用下面的方法处理

```java
org.apache.zookeeper.server.NIOServerCnxn#readPayload
org.apache.zookeeper.server.NIOServerCnxn#readRequest
```

readPayload这个方法里面会判断是连接请求还是非连接请求，连接请求在之前session建立的文章中介绍过，这里从处理非连接请求开始。

#### follower接收client请求

follower收到请求之后，先请求请求的opCode类型（这里是create）构造对应的request，然后交给第一个processor执行，follower的第一个processor是FollowerRequestProcessor.

#### follower转发请求给leader

由于在zk中follower是不能处理写请求的，需要转交给leader处理，在FollowerRequestProcessor中将请求转发给leader，转发请求的调用堆栈是

```java
serialize(OutputArchive, String):82, QuorumPacket (org.apache.zookeeper.server.quorum), QuorumPacket.java
 writeRecord(Record, String):123, BinaryOutputArchive (org.apache.jute), BinaryOutputArchive.java
 writePacket(QuorumPacket, boolean):139, Learner (org.apache.zookeeper.server.quorum), Learner.java
 request(Request):191, Learner (org.apache.zookeeper.server.quorum), Learner.java
 run():96, FollowerRequestProcessor (org.apache.zookeeper.server.quorum), FollowerRequestProcessor.java
```

FollowerRequestProcessor是一个线程在zk启动的时候就开始运行，主要逻辑在run方法里面，run方法的主要逻辑是

先把请求提交给CommitProcessor（后面leader发送给follower的commit请求对应到这里），然后将请求转发给leader，转发给leader的过程就是构造一个QuorumPacket，通过之前选举通信的端口发送给leader。

### leader接收follower请求

leader获取leader地位以后，启动learnhandler，然后一直在LearnerHandler#run循环，接收来自learner的packet，处理流程是：

```java
processRequest(Request):1003, org.apache.zookeeper.server.PrepRequestProcessor.java
submitLearnerRequest(Request):150, org.apache.zookeeper.server.quorum.LeaderZooKeeperServer.java
run():625, org.apache.zookeeper.server.quorum.LearnerHandler.java
```

handler判断是REQUEST请求的话交给leader的processor链处理，将请求放入org.apache.zookeeper.server.PrepRequestProcessor#submittedRequests，即leader的第一个processor。这个processor也是一个线程，从submittedRequests中不断拿出请求处理

processor主要做了：

1. 交给CommitProcessor等待提交
2. 交给leader的下一个processor处理：ProposalRequestProcessor

#### leader 发送proposal给follower

ProposalRequestProcessor主要作用就是讲请求交给下一个processor并且发起投票，将proposal发送给所有的follower。

```
public void processRequest(Request request) throws RequestProcessorException {
        // LOG.warn("Ack>>> cxid = " + request.cxid + " type = " +
        // request.type + " id = " + request.sessionId);
        // request.addRQRec(">prop");


        /* In the following IF-THEN-ELSE block, we process syncs on the leader.
         * If the sync is coming from a follower, then the follower
         * handler adds it to syncHandler. Otherwise, if it is a client of
         * the leader that issued the sync command, then syncHandler won't
         * contain the handler. In this case, we add it to syncHandler, and
         * call processRequest on the next processor.
         */

        if (request instanceof LearnerSyncRequest){
            zks.getLeader().processSync((LearnerSyncRequest)request);
        } else {
            nextProcessor.processRequest(request);
            if (request.getHdr() != null) {
                // We need to sync and get consensus on any transactions
                try {
                    zks.getLeader().propose(request);
                } catch (XidRolloverException e) {
                    throw new RequestProcessorException(e.getMessage(), e);
                }
                syncProcessor.processRequest(request);
            }
        }
    }
    
```



```java
// org.apache.zookeeper.server.quorum.Leader#sendPacket
void sendPacket(QuorumPacket qp) {
    synchronized (forwardingFollowers) {
        // 所有的follower，observer没有投票权
        for (LearnerHandler f : forwardingFollowers) {
            f.queuePacket(qp);
        }
    }
}
```

#### follower 收到proposal

follower处理proposal请求的调用堆栈

```java
processRequest(Request):214, org.apache.zookeeper.server.SyncRequestProcessor.java
 logRequest(TxnHeader, Record):89, org.apache.zookeeper.server.quorumFollowerZooKeeperServer.java
 processPacket(QuorumPacket):147, org.apache.zookeeper.server.quorum.Follower.java
 followLeader():102, org.apache.zookeeper.server.quorum.Follower.java
 run():1199, org.apache.zookeeper.server.quorum.QuorumPeer.java
```

将请求放入org.apache.zookeeper.server.SyncRequestProcessor#queuedRequests

#### follower 发送ack

线程SyncRequestProcessor#run从org.apache.zookeeper.server.SyncRequestProcessor#toFlush中取出请求flush，处理过程

1. follower开始commit，记录txLog和snapShot
2. 发送commit成功请求给leader，也就是follower给leader的ACK

#### leader收到ack

leader 收到ack后判断是否收到大多数的follower的ack，如果是说明可以commit，commit后同步给follower

#### follower收到commit

还是Follower#followLeader里面的while循环收到leader的commit请求后，调用下面的方法处理

org.apache.zookeeper.server.quorum.FollowerZooKeeperServer#commit

最终加入CommitProcessor.committedRequests队列，CommitProcessor主线程发现队列不空表明需要把这个request转发到下一个processor

#### follower发送请求给客户端

follower的最后一个processor是FinalRequestProcessor，最后会创建对应的节点并且构造response返回给client



### leader处理请求源码

如下所示：

```
 /**
     * Keep a count of acks that are received by the leader for a particular
     * proposal
     *
     * @param zxid, the zxid of the proposal sent out
     * @param sid, the id of the server that sent the ack
     * @param followerAddr
     */
    synchronized public void processAck(long sid, long zxid, SocketAddress followerAddr) {        
        if (!allowedToCommit) return; // last op committed was a leader change - from now on 
                                     // the new leader should commit        
        if (LOG.isTraceEnabled()) {
            LOG.trace("Ack zxid: 0x{}", Long.toHexString(zxid));
            for (Proposal p : outstandingProposals.values()) {
                long packetZxid = p.packet.getZxid();
                LOG.trace("outstanding proposal: 0x{}",
                        Long.toHexString(packetZxid));
            }
            LOG.trace("outstanding proposals all");
        }
        
        if ((zxid & 0xffffffffL) == 0) {
            /*
             * We no longer process NEWLEADER ack with this method. However,
             * the learner sends an ack back to the leader after it gets
             * UPTODATE, so we just ignore the message.
             */
            return;
        }
            
            
        if (outstandingProposals.size() == 0) {
            if (LOG.isDebugEnabled()) {
                LOG.debug("outstanding is 0");
            }
            return;
        }
        if (lastCommitted >= zxid) {
            if (LOG.isDebugEnabled()) {
                LOG.debug("proposal has already been committed, pzxid: 0x{} zxid: 0x{}",
                        Long.toHexString(lastCommitted), Long.toHexString(zxid));
            }
            // The proposal has already been committed
            return;
        }
        Proposal p = outstandingProposals.get(zxid);
        if (p == null) {
            LOG.warn("Trying to commit future proposal: zxid 0x{} from {}",
                    Long.toHexString(zxid), followerAddr);
            return;
        }
        
        p.addAck(sid);        
        /*if (LOG.isDebugEnabled()) {
            LOG.debug("Count for zxid: 0x{} is {}",
                    Long.toHexString(zxid), p.ackSet.size());
        }*/
        
        boolean hasCommitted = tryToCommit(p, zxid, followerAddr);

        // If p is a reconfiguration, multiple other operations may be ready to be committed,
        // since operations wait for different sets of acks.
       // Currently we only permit one outstanding reconfiguration at a time
       // such that the reconfiguration and subsequent outstanding ops proposed while the reconfig is
       // pending all wait for a quorum of old and new config, so its not possible to get enough acks
       // for an operation without getting enough acks for preceding ops. But in the future if multiple
       // concurrent reconfigs are allowed, this can happen and then we need to check whether some pending
        // ops may already have enough acks and can be committed, which is what this code does.

        if (hasCommitted && p.request!=null && p.request.getHdr().getType() == OpCode.reconfig){
               long curZxid = zxid;
           while (allowedToCommit && hasCommitted && p!=null){
               curZxid++;
               p = outstandingProposals.get(curZxid);
               if (p !=null) hasCommitted = tryToCommit(p, curZxid, null);             
           }
        }
    }
    
```

调用实现，最终由CommitProcessor 接着处理请求：

```
 /**
     * @return True if committed, otherwise false.
     * @param a proposal p
     **/
    synchronized public boolean tryToCommit(Proposal p, long zxid, SocketAddress followerAddr) {       
       // make sure that ops are committed in order. With reconfigurations it is now possible
       // that different operations wait for different sets of acks, and we still want to enforce
       // that they are committed in order. Currently we only permit one outstanding reconfiguration
       // such that the reconfiguration and subsequent outstanding ops proposed while the reconfig is
       // pending all wait for a quorum of old and new config, so its not possible to get enough acks
       // for an operation without getting enough acks for preceding ops. But in the future if multiple
       // concurrent reconfigs are allowed, this can happen.
       if (outstandingProposals.containsKey(zxid - 1)) return false;
       
       // getting a quorum from all necessary configurations
        if (!p.hasAllQuorums()) {
           return false;                 
        }
        
        // commit proposals in order
        if (zxid != lastCommitted+1) {    
           LOG.warn("Commiting zxid 0x" + Long.toHexString(zxid)
                    + " from " + followerAddr + " not first!");
            LOG.warn("First is "
                    + (lastCommitted+1));
        }     
        
        // in order to be committed, a proposal must be accepted by a quorum              
        
        outstandingProposals.remove(zxid);
        
        if (p.request != null) {
             toBeApplied.add(p);
        }

        if (p.request == null) {
            LOG.warn("Going to commmit null: " + p);
        } else if (p.request.getHdr().getType() == OpCode.reconfig) {                                   
            LOG.debug("Committing a reconfiguration! " + outstandingProposals.size()); 
                 
            //if this server is voter in new config with the same quorum address, 
            //then it will remain the leader
            //otherwise an up-to-date follower will be designated as leader. This saves
            //leader election time, unless the designated leader fails                             
            Long designatedLeader = getDesignatedLeader(p, zxid);
            //LOG.warn("designated leader is: " + designatedLeader);

            QuorumVerifier newQV = p.qvAcksetPairs.get(p.qvAcksetPairs.size()-1).getQuorumVerifier();
       
            self.processReconfig(newQV, designatedLeader, zk.getZxid(), true);
       
            if (designatedLeader != self.getId()) {
                allowedToCommit = false;
            }
                   
            // we're sending the designated leader, and if the leader is changing the followers are 
            // responsible for closing the connection - this way we are sure that at least a majority of them 
            // receive the commit message.
            commitAndActivate(zxid, designatedLeader);
            informAndActivate(p, designatedLeader);
            //turnOffFollowers();
        } else {
            commit(zxid);
            inform(p);
        }
        zk.commitProcessor.commit(p.request);
        if(pendingSyncs.containsKey(zxid)){
            for(LearnerSyncRequest r: pendingSyncs.remove(zxid)) {
                sendSync(r);
            }               
        } 
        
        return  true;   
    }
```

该程序第一步是发送一个请求到Quorum的所有成员

```
    /**
     * Create a commit packet and send it to all the members of the quorum
     *
     * @param zxid
     */
    public void commit(long zxid) {
        synchronized(this){
            lastCommitted = zxid;
        }
        QuorumPacket qp = new QuorumPacket(Leader.COMMIT, zxid, null, null);
        sendPacket(qp);
    }
```

发送报文如下：

```
    /**
     * send a packet to all the followers ready to follow
     *
     * @param qp
     *                the packet to be sent
     */
    void sendPacket(QuorumPacket qp) {
        synchronized (forwardingFollowers) {
            for (LearnerHandler f : forwardingFollowers) {
                f.queuePacket(qp);
            }
        }
    }
```

第二步是通知Observer

```
    /**
     * Create an inform packet and send it to all observers.
     * @param zxid
     * @param proposal
     */
    public void inform(Proposal proposal) {
        QuorumPacket qp = new QuorumPacket(Leader.INFORM, proposal.request.zxid,
                                            proposal.packet.getData(), null);
        sendObserverPacket(qp);
    }
```

发送observer程序如下：

```
    /**
     * send a packet to all observers
     */
    void sendObserverPacket(QuorumPacket qp) {
        for (LearnerHandler f : getObservingLearners()) {
            f.queuePacket(qp);
        }
    }
```

第三步到

```
 zk.commitProcessor.commit(p.request);
```

2. CommitProcessor

CommitProcessor是多线程的，线程间通信通过queue，atomic和wait/notifyAll同步。CommitProcessor扮演一个网关角色，允许请求到剩下的处理管道。在同一瞬间，它支持多个读请求而仅支持一个写请求，这是为了保证写请求在事务中的顺序。

  1个commit处理主线程，它监控请求队列，并将请求分发到工作线程，分发过程基于sessionId，这样特定session的读写请求通常分发到同一个线程，因而可以保证运行的顺序。

　　0~N个工作进程，他们在请求上运行剩下的请求处理管道。如果配置为0个工作线程，主commit线程将会直接运行管道。

　　经典(默认)线程数是：在32核的机器上，一个commit处理线程和32个工作线程。

多线程的限制：

　　每个session的请求处理必须是顺序的。

　　写请求处理必须按照zxid顺序。

　　必须保证一个session内不会出现写条件竞争，条件竞争可能导致另外一个session的读请求触发监控。

当前实现解决第三个限制，仅仅通过不允许在写请求时允许读进程的处理。

```
 @Override
    public void run() {
        Request request;
        try {
            while (!stopped) {
                synchronized(this) {
                    while (
                        !stopped &&
                        ((queuedRequests.isEmpty() || isWaitingForCommit() || isProcessingCommit()) &&
                         (committedRequests.isEmpty() || isProcessingRequest()))) {
                        wait();
                    }
                }

                /*
                 * Processing queuedRequests: Process the next requests until we
                 * find one for which we need to wait for a commit. We cannot
                 * process a read request while we are processing write request.
                 */
                while (!stopped && !isWaitingForCommit() &&
                       !isProcessingCommit() &&
                       (request = queuedRequests.poll()) != null) {
                    if (needCommit(request)) {
                        nextPending.set(request);
                    } else {
                        sendToNextProcessor(request);
                    }
                }

                /*
                 * Processing committedRequests: check and see if the commit
                 * came in for the pending request. We can only commit a
                 * request when there is no other request being processed.
                 */
                processCommitted();
            }
        } catch (Throwable e) {
            handleException(this.getName(), e);
        }
        LOG.info("CommitProcessor exited loop!");
    }
```

主逻辑程序如下：

```
 /*
     * Separated this method from the main run loop
     * for test purposes (ZOOKEEPER-1863)
     */
    protected void processCommitted() {
        Request request;

        if (!stopped && !isProcessingRequest() &&
                (committedRequests.peek() != null)) {

            /*
             * ZOOKEEPER-1863: continue only if there is no new request
             * waiting in queuedRequests or it is waiting for a
             * commit. 
             */
            if ( !isWaitingForCommit() && !queuedRequests.isEmpty()) {
                return;
            }
            request = committedRequests.poll();

            /*
             * We match with nextPending so that we can move to the
             * next request when it is committed. We also want to
             * use nextPending because it has the cnxn member set
             * properly.
             */
            Request pending = nextPending.get();
            if (pending != null &&
                pending.sessionId == request.sessionId &&
                pending.cxid == request.cxid) {
                // we want to send our version of the request.
                // the pointer to the connection in the request
                pending.setHdr(request.getHdr());
                pending.setTxn(request.getTxn());
                pending.zxid = request.zxid;
                // Set currentlyCommitting so we will block until this
                // completes. Cleared by CommitWorkRequest after
                // nextProcessor returns.
                currentlyCommitting.set(pending);
                nextPending.set(null);
                sendToNextProcessor(pending);
            } else {
                // this request came from someone else so just
                // send the commit packet
                currentlyCommitting.set(request);
                sendToNextProcessor(request);
            }
        }      
    }
```

启动多线程处理程序

```
    /**
     * Schedule final request processing; if a worker thread pool is not being
     * used, processing is done directly by this thread.
     */
    private void sendToNextProcessor(Request request) {
        numRequestsProcessing.incrementAndGet();
        workerPool.schedule(new CommitWorkRequest(request), request.sessionId);
    }
```

真实逻辑是

```
 /**
     * Schedule work to be done by the thread assigned to this id. Thread
     * assignment is a single mod operation on the number of threads.  If a
     * worker thread pool is not being used, work is done directly by
     * this thread.
     */
    public void schedule(WorkRequest workRequest, long id) {
        if (stopped) {
            workRequest.cleanup();
            return;
        }

        ScheduledWorkRequest scheduledWorkRequest =
            new ScheduledWorkRequest(workRequest);

        // If we have a worker thread pool, use that; otherwise, do the work
        // directly.
        int size = workers.size();
        if (size > 0) {
            try {
                // make sure to map negative ids as well to [0, size-1]
                int workerNum = ((int) (id % size) + size) % size;
                ExecutorService worker = workers.get(workerNum);
                worker.execute(scheduledWorkRequest);
            } catch (RejectedExecutionException e) {
                LOG.warn("ExecutorService rejected execution", e);
                workRequest.cleanup();
            }
        } else {
            // When there is no worker thread pool, do the work directly
            // and wait for its completion
            scheduledWorkRequest.start();
            try {
                scheduledWorkRequest.join();
            } catch (InterruptedException e) {
                LOG.warn("Unexpected exception", e);
                Thread.currentThread().interrupt();
            }
        }
    }
```

请求处理线程run方法：

```
       @Override
        public void run() {
            try {
                // Check if stopped while request was on queue
                if (stopped) {
                    workRequest.cleanup();
                    return;
                }
                workRequest.doWork();
            } catch (Exception e) {
                LOG.warn("Unexpected exception", e);
                workRequest.cleanup();
            }
        }
```

调用commitProcessor的doWork方法

```
        public void doWork() throws RequestProcessorException {
            try {
                nextProcessor.processRequest(request);
            } finally {
                // If this request is the commit request that was blocking
                // the processor, clear.
                currentlyCommitting.compareAndSet(request, null);

                /*
                 * Decrement outstanding request count. The processor may be
                 * blocked at the moment because it is waiting for the pipeline
                 * to drain. In that case, wake it up if there are pending
                 * requests.
                 */
                if (numRequestsProcessing.decrementAndGet() == 0) {
                    if (!queuedRequests.isEmpty() ||
                        !committedRequests.isEmpty()) {
                        wakeup();
                    }
                }
            }
        }
```

将请求传递给下一个RP：Leader.ToBeAppliedRequestProcessor

3.Leader.ToBeAppliedRequestProcessor

Leader.ToBeAppliedRequestProcessor仅仅维护一个toBeApplied列表。

```
 /**
         * This request processor simply maintains the toBeApplied list. For
         * this to work next must be a FinalRequestProcessor and
         * FinalRequestProcessor.processRequest MUST process the request
         * synchronously!
         *
         * @param next
         *                a reference to the FinalRequestProcessor
         */
        ToBeAppliedRequestProcessor(RequestProcessor next, Leader leader) {
            if (!(next instanceof FinalRequestProcessor)) {
                throw new RuntimeException(ToBeAppliedRequestProcessor.class
                        .getName()
                        + " must be connected to "
                        + FinalRequestProcessor.class.getName()
                        + " not "
                        + next.getClass().getName());
            }
            this.leader = leader;
            this.next = next;
        }

        /*
         * (non-Javadoc)
         *
         * @see org.apache.zookeeper.server.RequestProcessor#processRequest(org.apache.zookeeper.server.Request)
         */
        public void processRequest(Request request) throws RequestProcessorException {
            next.processRequest(request);

            // The only requests that should be on toBeApplied are write
            // requests, for which we will have a hdr. We can't simply use
            // request.zxid here because that is set on read requests to equal
            // the zxid of the last write op.
            if (request.getHdr() != null) {
                long zxid = request.getHdr().getZxid();
                Iterator<Proposal> iter = leader.toBeApplied.iterator();
                if (iter.hasNext()) {
                    Proposal p = iter.next();
                    if (p.request != null && p.request.zxid == zxid) {
                        iter.remove();
                        return;
                    }
                }
                LOG.error("Committed request not found on toBeApplied: "
                          + request);
            }
        }
```

4. *FinalRequestProcessor前文已经说明，本文不在赘述。*

小结：从上面的分析可以知道，leader处理请求的顺序分别是：PrepRequestProcessor -> ProposalRequestProcessor ->CommitProcessor -> Leader.ToBeAppliedRequestProcessor ->**FinalRequestProcessor。**

请求先通过PrepRequestProcessor接收请求，并进行包装，然后请求类型的不同，设置同享数据；主要负责通知所有follower和observer；CommitProcessor 启动多线程处理请求；Leader.ToBeAppliedRequestProcessor仅仅维护一个toBeApplied列表；

FinalRequestProcessor来作为消息处理器的终结者，发送响应消息，并触发watcher的处理程序。

## 序列化

　　序列化主要在zookeeper.jute包中，其中涉及的主要接口如下

　　　　**· InputArchive**

　　　　**· OutputArchive**

　　　　**· Index**

　　　　**· Record**

#### InputArchive

　　其是所有反序列化器都需要实现的接口，其方法如下　

```
public interface InputArchive {
    // 读取byte类型
    public byte readByte(String tag) throws IOException;
    // 读取boolean类型
    public boolean readBool(String tag) throws IOException;
    // 读取int类型
    public int readInt(String tag) throws IOException;
    // 读取long类型
    public long readLong(String tag) throws IOException;
    // 读取float类型
    public float readFloat(String tag) throws IOException;
    // 读取double类型
    public double readDouble(String tag) throws IOException;
    // 读取String类型
    public String readString(String tag) throws IOException;
    // 通过缓冲方式读取
    public byte[] readBuffer(String tag) throws IOException;
    // 开始读取记录
    public void readRecord(Record r, String tag) throws IOException;
    // 开始读取记录
    public void startRecord(String tag) throws IOException;
    // 结束读取记录
    public void endRecord(String tag) throws IOException;
    // 开始读取向量
    public Index startVector(String tag) throws IOException;
    // 结束读取向量
    public void endVector(String tag) throws IOException;
    // 开始读取Map
    public Index startMap(String tag) throws IOException;
    // 结束读取Map
    public void endMap(String tag) throws IOException;
}
```

InputArchive实现类如下

BinaryInputArchive　　

CsvInputArchive　

XmlInputArchive　

#### OutputArchive

其是所有序列化器都需要实现此接口，其方法如下。　　

```
public interface OutputArchive {
    // 写Byte类型
    public void writeByte(byte b, String tag) throws IOException;
    // 写boolean类型
    public void writeBool(boolean b, String tag) throws IOException;
    // 写int类型
    public void writeInt(int i, String tag) throws IOException;
    // 写long类型
    public void writeLong(long l, String tag) throws IOException;
    // 写float类型
    public void writeFloat(float f, String tag) throws IOException;
    // 写double类型
    public void writeDouble(double d, String tag) throws IOException;
    // 写String类型
    public void writeString(String s, String tag) throws IOException;
    // 写Buffer类型
    public void writeBuffer(byte buf[], String tag)
        throws IOException;
    // 写Record类型
    public void writeRecord(Record r, String tag) throws IOException;
    // 开始写Record
    public void startRecord(Record r, String tag) throws IOException;
    // 结束写Record
    public void endRecord(Record r, String tag) throws IOException;
    // 开始写Vector
    public void startVector(List v, String tag) throws IOException;
    // 结束写Vector
    public void endVector(List v, String tag) throws IOException;
    // 开始写Map
    public void startMap(TreeMap v, String tag) throws IOException;
    // 结束写Map
    public void endMap(TreeMap v, String tag) throws IOException;

}
```

　　OutputArchive的类结构如下

BinaryOutputArchive　

CsvOutputArchive　

XmlOutputArchive

#### Index

　　其用于迭代反序列化器的迭代器。　　

```
public interface Index {
    // 是否已经完成
    public boolean done();
    // 下一项
    public void incr();
}
```

Index的类结构如下

BinaryIndex　

```
    static private class BinaryIndex implements Index {
        // 元素个数
        private int nelems;
        // 构造函数
        BinaryIndex(int nelems) {
            this.nelems = nelems;
        }
        // 是否已经完成
        public boolean done() {
            return (nelems <= 0);
        }
        // 移动一项
        public void incr() {
            nelems--;
        }
    }
```

CsxIndex　

```
    private class CsvIndex implements Index {
        // 是否已经完成
        public boolean done() {
            char c = '\0';
            try {
                // 读取字符
                c = (char) stream.read();
                // 推回缓冲区 
                stream.unread(c);
            } catch (IOException ex) {
            }
            return (c == '}') ? true : false;
        }
        // 什么都不做
        public void incr() {}
    }
```

XmlIndex　

```
    private class XmlIndex implements Index {
        // 是否已经完成
        public boolean done() {
            // 根据索引获取值
            Value v = valList.get(vIdx);
            if ("/array".equals(v.getType())) { // 判断是否值的类型是否为/array
                // 设置索引的值
                valList.set(vIdx, null);
                // 索引加1
                vIdx++;
                return true;
            } else {
                return false;
            }
        }
        // 什么都不做
        public void incr() {}
    }
```

Record

所有用于网络传输或者本地存储的类型都实现该接口，其方法如下　　

```
public interface Record {
    // 序列化
    public void serialize(OutputArchive archive, String tag)
        throws IOException;
    // 反序列化
    public void deserialize(InputArchive archive, String tag)
        throws IOException;
}
```

　　所有的实现类都需要实现seriallize和deserialize方法。 

#### **示例**

　　下面通过一个示例来理解OutputArchive和InputArchive的搭配使用。　

```


import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Set;
import java.util.TreeMap;

import org.apache.jute.BinaryInputArchive;
import org.apache.jute.BinaryOutputArchive;
import org.apache.jute.Index;
import org.apache.jute.InputArchive;
import org.apache.jute.OutputArchive;
import org.apache.jute.Record;

public class ArchiveTest {
    public static void main( String[] args ) throws IOException {
        String path = "F:\\test.txt";
        // write operation
        OutputStream outputStream = new FileOutputStream(new File(path));
        BinaryOutputArchive binaryOutputArchive = BinaryOutputArchive.getArchive(outputStream);
        
        binaryOutputArchive.writeBool(true, "boolean");
        byte[] bytes = "leesf".getBytes();
        binaryOutputArchive.writeBuffer(bytes, "buffer");
        binaryOutputArchive.writeDouble(13.14, "double");
        binaryOutputArchive.writeFloat(5.20f, "float");
        binaryOutputArchive.writeInt(520, "int");
        Person person = new Person(25, "leesf");
        binaryOutputArchive.writeRecord(person, "leesf");
        TreeMap<String, Integer> map = new TreeMap<String, Integer>();
        map.put("leesf", 25);
        map.put("dyd", 25);
        Set<String> keys = map.keySet();
        binaryOutputArchive.startMap(map, "map");
        int i = 0;
        for (String key: keys) {
            String tag = i + "";
            binaryOutputArchive.writeString(key, tag);
            binaryOutputArchive.writeInt(map.get(key), tag);
            i++;
        }
        
        binaryOutputArchive.endMap(map, "map");
        
        
        // read operation
        InputStream inputStream = new FileInputStream(new File(path));
        BinaryInputArchive binaryInputArchive = BinaryInputArchive.getArchive(inputStream);
        
        System.out.println(binaryInputArchive.readBool("boolean"));
        System.out.println(new String(binaryInputArchive.readBuffer("buffer")));
        System.out.println(binaryInputArchive.readDouble("double"));
        System.out.println(binaryInputArchive.readFloat("float"));
        System.out.println(binaryInputArchive.readInt("int"));
        Person person2 = new Person();
        binaryInputArchive.readRecord(person2, "leesf");
        System.out.println(person2);       
        
        Index index = binaryInputArchive.startMap("map");
        int j = 0;
        while (!index.done()) {
            String tag = j + "";
            System.out.println("key = " + binaryInputArchive.readString(tag) 
                + ", value = " + binaryInputArchive.readInt(tag));
            index.incr();
            j++;
        }
    }
    
    static class Person implements Record {
        private int age;
        private String name;
        
        public Person() {
            
        }
        
        public Person(int age, String name) {
            this.age = age;
            this.name = name;
        }

        public void serialize(OutputArchive archive, String tag) throws IOException {
            archive.startRecord(this, tag);
            archive.writeInt(age, "age");
            archive.writeString(name, "name");
            archive.endRecord(this, tag);
        }

        public void deserialize(InputArchive archive, String tag) throws IOException {
            archive.startRecord(tag);
            age = archive.readInt("age");
            name = archive.readString("name");
            archive.endRecord(tag);            
        }    
        
        public String toString() {
            return "age = " + age + ", name = " + name;
        }
    }
}
```

## 持久化

### 持久化总体框架

持久化的类主要在包org.apache.zookeeper.server.persistence下，此次也主要是对其下的类进行分析，其包下总体的类结构如下。

　    · TxnLog，接口类型，读取事务性日志的接口。

　　· FileTxnLog，实现TxnLog接口，添加了访问该事务性日志的API。

　　· Snapshot，接口类型，持久层快照接口。

　　· FileSnap，实现Snapshot接口，负责存储、序列化、反序列化、访问快照。

　　· FileTxnSnapLog，封装了TxnLog和SnapShot。

　　· Util，工具类，提供持久化所需的API。

### TxnLog源码分析

TxnLog是接口，规定了对日志的响应操作。

```
public interface TxnLog {
    
    /**
     * roll the current
     * log being appended to
     * @throws IOException 
     */
    // 回滚日志
    void rollLog() throws IOException;
    /**
     * Append a request to the transaction log
     * @param hdr the transaction header
     * @param r the transaction itself
     * returns true iff something appended, otw false 
     * @throws IOException
     */
    // 添加一个请求至事务性日志
    boolean append(TxnHeader hdr, Record r) throws IOException;

    /**
     * Start reading the transaction logs
     * from a given zxid
     * @param zxid
     * @return returns an iterator to read the 
     * next transaction in the logs.
     * @throws IOException
     */
    // 读取事务性日志
    TxnIterator read(long zxid) throws IOException;
    
    /**
     * the last zxid of the logged transactions.
     * @return the last zxid of the logged transactions.
     * @throws IOException
     */
    // 事务性操作的最新zxid
    long getLastLoggedZxid() throws IOException;
    
    /**
     * truncate the log to get in sync with the 
     * leader.
     * @param zxid the zxid to truncate at.
     * @throws IOException 
     */
    // 清空日志，与Leader保持同步
    boolean truncate(long zxid) throws IOException;
    
    /**
     * the dbid for this transaction log. 
     * @return the dbid for this transaction log.
     * @throws IOException
     */
    // 获取数据库的id
    long getDbId() throws IOException;
    
    /**
     * commmit the trasaction and make sure
     * they are persisted
     * @throws IOException
     */
    // 提交事务并进行确认
    void commit() throws IOException;
   
    /** 
     * close the transactions logs
     */
    // 关闭事务性日志
    void close() throws IOException;
    /**
     * an iterating interface for reading 
     * transaction logs. 
     */
    // 读取事务日志的迭代器接口
    public interface TxnIterator {
        /**
         * return the transaction header.
         * @return return the transaction header.
         */
        // 获取事务头部
        TxnHeader getHeader();
        
        /**
         * return the transaction record.
         * @return return the transaction record.
         */
        // 获取事务
        Record getTxn();
     
        /**
         * go to the next transaction record.
         * @throws IOException
         */
        // 下个事务
        boolean next() throws IOException;
        
        /**
         * close files and release the 
         * resources
         * @throws IOException
         */
        // 关闭文件释放资源
        void close() throws IOException;
    }
}
```

其中，TxnLog除了提供读写事务日志的API外，还提供了一个用于读取日志的迭代器接口TxnIterator。

对于LogFile而言，其格式可分为如下三部分

　　**LogFile:**

　　　　FileHeader TxnList ZeroPad

```
public class FileTxnLog implements TxnLog {
    private static final Logger LOG;
    
    // 预分配大小 64M
    static long preAllocSize =  65536 * 1024;
    
    // 魔术数字，默认为1514884167
    public final static int TXNLOG_MAGIC =
        ByteBuffer.wrap("ZKLG".getBytes()).getInt();

    // 版本号
    public final static int VERSION = 2;

    /** Maximum time we allow for elapsed fsync before WARNing */
    // 进行同步时，发出warn之前所能等待的最长时间
    private final static long fsyncWarningThresholdMS;

    // 静态属性，确定Logger、预分配空间大小和最长时间
    static {
        LOG = LoggerFactory.getLogger(FileTxnLog.class);

        String size = System.getProperty("zookeeper.preAllocSize");
        if (size != null) {
            try {
                preAllocSize = Long.parseLong(size) * 1024;
            } catch (NumberFormatException e) {
                LOG.warn(size + " is not a valid value for preAllocSize");
            }
        }
        fsyncWarningThresholdMS = Long.getLong("fsync.warningthresholdms", 1000);
    }
    
    // 最大(新)的zxid
    long lastZxidSeen;
    // 存储数据相关的流
    volatile BufferedOutputStream logStream = null;
    volatile OutputArchive oa;
    volatile FileOutputStream fos = null;

    // log目录文件
    File logDir;
    
    // 是否强制同步
    private final boolean forceSync = !System.getProperty("zookeeper.forceSync", "yes").equals("no");;
    
    // 数据库id
    long dbId;
    
    // 流列表
    private LinkedList<FileOutputStream> streamsToFlush =
        new LinkedList<FileOutputStream>();
    
    // 当前大小
    long currentSize;
    // 写日志文件
    File logFileWrite = null;
}
```

### SnapShot源码分析

　SnapShot是FileTxnLog的父类，接口类型，其方法如下　　

```
public interface SnapShot {
    
    /**
     * deserialize a data tree from the last valid snapshot and 
     * return the last zxid that was deserialized
     * @param dt the datatree to be deserialized into
     * @param sessions the sessions to be deserialized into
     * @return the last zxid that was deserialized from the snapshot
     * @throws IOException
     */
    // 反序列化
    long deserialize(DataTree dt, Map<Long, Integer> sessions) 
        throws IOException;
    
    /**
     * persist the datatree and the sessions into a persistence storage
     * @param dt the datatree to be serialized
     * @param sessions 
     * @throws IOException
     */
    // 序列化
    void serialize(DataTree dt, Map<Long, Integer> sessions, 
            File name) 
        throws IOException;
    
    /**
     * find the most recent snapshot file
     * @return the most recent snapshot file
     * @throws IOException
     */
    // 查找最新的snapshot文件
    File findMostRecentSnapshot() throws IOException;
    
    /**
     * free resources from this snapshot immediately
     * @throws IOException
     */
    // 释放资源
    void close() throws IOException;
} 
```

说明：可以看到SnapShot只定义了四个方法，反序列化、序列化、查找最新的snapshot文件、释放资源。

FileSnap实现了SnapShot接口，主要用作存储、序列化、反序列化、访问相应snapshot文件。

```
public class FileSnap implements SnapShot {
    // snapshot目录文件
    File snapDir;
    // 是否已经关闭标识
    private volatile boolean close = false;
    // 版本号
    private static final int VERSION=2;
    // database id
    private static final long dbId=-1;
    // Logger
    private static final Logger LOG = LoggerFactory.getLogger(FileSnap.class);
    // snapshot文件的魔数(类似class文件的魔数)
    public final static int SNAP_MAGIC
        = ByteBuffer.wrap("ZKSN".getBytes()).getInt();
}
```

### FileTxnSnapLog源码分析

```
public class FileTxnSnapLog {
    //the direcotry containing the 
    //the transaction logs
    // 日志文件目录
    private final File dataDir;
    //the directory containing the
    //the snapshot directory
    // 快照文件目录
    private final File snapDir;
    // 事务日志
    private TxnLog txnLog;
    // 快照
    private SnapShot snapLog;
    // 版本号
    public final static int VERSION = 2;
    // 版本
    public final static String version = "version-";

    // Logger
    private static final Logger LOG = LoggerFactory.getLogger(FileTxnSnapLog.class);
}
```

说明：类的属性中包含了TxnLog和SnapShot接口，即对FileTxnSnapLog的很多操作都会转发给TxnLog和SnapLog进行操作，这是一种典型的组合方法。

FileTxnSnapLog包含了PlayBackListener内部类，用来接收事务应用过程中的回调，在Zookeeper数据恢复后期，会有事务修正过程，此过程会回调PlayBackListener来进行对应的数据修正。

## Watcher机制

对于Watcher机制而言，主要涉及的类主要如下。

​		Watcher，接口类型，其定义了process方法，需子类实现。

　　Event，接口类型，Watcher的内部类，无任何方法。

　　KeeperState，枚举类型，Event的内部类，表示Zookeeper所处的状态。

　　EventType，枚举类型，Event的内部类，表示Zookeeper中发生的事件类型。

　　WatchedEvent，表示对ZooKeeper上发生变化后的反馈，包含了KeeperState和EventType。

　　ClientWatchManager，接口类型，表示客户端的Watcher管理者，其定义了materialized方法，需子类实现。

　　ZKWatchManager，Zookeeper的内部类，继承ClientWatchManager。

　　MyWatcher，ZooKeeperMain的内部类，继承Watcher。

　　ServerCnxn，接口类型，继承Watcher，表示客户端与服务端的一个连接。

　　WatchManager，管理Watcher。

### Watcher源码分析

## 客户端源码

Zookeeper 客户端主要有以下几个重要的组件。客户端会话创建可以分为三个阶段：一是初始化阶段、二是会话创建阶段、三是响应处理阶段。

| 类                    | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| Zookeeper             | Zookeeper 客户端入口                                         |
| ClientWatchManager    | 客户端 Watcher 管理器                                        |
| ClientCnxn            | 客户端核心线程，其内部又包含两个线程，即 SendThread 和 EventThread。前者是一个 IO 线程，主要负责 ZooKeeper 客户端和服务端之间的网络通信；后者是一个事件线程，主要负责对服务端事件进行处理。 |
| HostProvider          | 客户端地址列表管理器                                         |
| ClientCnxnSocketNetty | 最底层的通信 netty                                           |

客户端整体结构如下图：![Zookeeper客户端类图](png\zookeeper\Zookeeper客户端类图.png)

客户端在构造阶段创建 ClientCnxn 与服务端连接，后续命令都是通过 ClientCnxn 发送给服务端。ClientCnxn 是客户端与服务端通信的底层接口，它和 ClientCnxnSocketNetty 一起工作提供网络通信服务。服务端是 ZookeeperServer 类，收到 ClientCnxn 的请求处理后再通过 ClientCnxn 返回到客户端。

ClientCnxn 连接时可以同时指定多台服务器地址，根据指定的算法连接一台服务器，当某个服务器发生故障无法连接时，会自动连接到其他的服务器。实现这一机制的是 StaticHostProvider 类。

#### 创建阶段

**(1) ZooKeeper 创建** 【ZooKeeper】

```java
public ZooKeeper(String connectString, int sessionTimeout, Watcher watcher,
        long sessionId, byte[] sessionPasswd, boolean canBeReadOnly,
        HostProvider aHostProvider) throws IOException {
    
    // 1. watcher 保存在 ZKWatchManager 的 defaultWatcher 中，作为整个会话的默认 watcher
    watchManager = defaultWatchManager();
    watchManager.defaultWatcher = watcher;
   
    // 2. 解析 server 获取 IP 以及 PORT
    ConnectStringParser connectStringParser = new ConnectStringParser(
            connectString);
    hostProvider = aHostProvider;

    // 3. 创建 ClientCnxn 实例
    cnxn = new ClientCnxn(connectStringParser.getChrootPath(),
            hostProvider, sessionTimeout, this, watchManager,
            getClientCnxnSocket(), sessionId, sessionPasswd, canBeReadOnly);
    cnxn.seenRwServerBefore = true; // since user has provided sessionId
    // 4. 启动 SendThread 和 EventThread 线程，这两个线程均为守护线程
    cnxn.start();
}
```

创建底层通信 ClientCnxnSocketNIO 或 ClientCnxnSocketNetty

```java
public static final String ZOOKEEPER_CLIENT_CNXN_SOCKET = "zookeeper.clientCnxnSocket";
private static ClientCnxnSocket getClientCnxnSocket() throws IOException {
    String clientCnxnSocketName = System
            .getProperty(ZOOKEEPER_CLIENT_CNXN_SOCKET);
    if (clientCnxnSocketName == null) {
        clientCnxnSocketName = ClientCnxnSocketNIO.class.getName();
    }
    try {
        return (ClientCnxnSocket) Class.forName(clientCnxnSocketName)
                .newInstance();
    } catch (Exception e) {
        IOException ioe = new IOException("Couldn't instantiate "
                + clientCnxnSocketName);
        ioe.initCause(e);
        throw ioe;
    }
}
```

**(2) ClientCnxn 创建** 【ClientCnxn】

| 属性          | 说明                        |
| ------------- | --------------------------- |
| Packet        | 所有的请求都会封装成 packet |
| SendThread    | 请求处理                    |
| EventThread   | 事件处理                    |
| outgoingQueue | 即将发送的请求 packets      |
| pendingQueue  | 已经发送等待响应的 packets  |

ClientCnxn 创建时创建了两个线程 SendThread 和 EventThread，这两个线程都是守护线程，主线程结束时即关闭线程。

```java
// 已经发送等待响应的 packets
private final LinkedList<Packet> pendingQueue = new LinkedList<Packet>();
// 即将发送的请求 packets
private final LinkedBlockingDeque<Packet> outgoingQueue = new LinkedBlockingDeque<Packet>();

public ClientCnxn(String chrootPath, HostProvider hostProvider, int sessionTimeout, ZooKeeper zooKeeper,
        ClientWatchManager watcher, ClientCnxnSocket clientCnxnSocket,
        long sessionId, byte[] sessionPasswd, boolean canBeReadOnly) {
    this.zooKeeper = zooKeeper;
    this.watcher = watcher;
    this.sessionId = sessionId;
    this.sessionPasswd = sessionPasswd;
    this.sessionTimeout = sessionTimeout;
    this.hostProvider = hostProvider;
    this.chrootPath = chrootPath;

    connectTimeout = sessionTimeout / hostProvider.size();
    readTimeout = sessionTimeout * 2 / 3;
    readOnly = canBeReadOnly;

    sendThread = new SendThread(clientCnxnSocket);
    eventThread = new EventThread();
}

SendThread(ClientCnxnSocket clientCnxnSocket) {
    super(makeThreadName("-SendThread()"));
    state = States.CONNECTING;
    this.clientCnxnSocket = clientCnxnSocket;
    setDaemon(true);
}

EventThread() {
    super(makeThreadName("-EventThread"));
    setDaemon(true);
}
```

#### 会话创建

**(3) 启动两个守护线程** 【ClientCnxn】

```java
public void start() {
    sendThread.start();
    eventThread.start();
}
```

**(4) 开始创建连接** 【SendThread】

SendThread 的 run 方法启动时会调用 startConnect() 先创建连接。

```java
if (!clientCnxnSocket.isConnected()) {
    // don't re-establish connection if we are closing
    if (closing) {
        break;
    }
    startConnect();
    clientCnxnSocket.updateLastSendAndHeard();
}
```

startConnect() 会通过 hostProvider 获取 zookeeper 服务端地址，然后调用底层的 clientCnxnSocket 来创建一个连接。

```java
private void startConnect() throws IOException {
    if(!isFirstConnect){
        try {
            Thread.sleep(r.nextInt(1000));
        } catch (InterruptedException e) {
            LOG.warn("Unexpected exception", e);
        }
    }
    state = States.CONNECTING;

    InetSocketAddress addr;
    if (rwServerAddress != null) {
        addr = rwServerAddress;
        rwServerAddress = null;
    } else {
        addr = hostProvider.next(1000);
    }

    setName(getName().replaceAll("\\(.*\\)",
            "(" + addr.getHostString() + ":" + addr.getPort() + ")"));
    // ... 省略

    logStartConnect(addr);
    clientCnxnSocket.connect(addr);
}
```

**(5) 建立 TCP 连接** 【ClientCnxnSocketNetty】

ClientCnxnSocket 有两个实现类 ClientCnxnSocketNetty 和 ClientCnxnSocketNIO，负责底层的通信，以 ClientCnxnSocketNetty 为例通过 connect 方法创建了一个 TCP 连接。

```java
@Override
void connect(InetSocketAddress addr) throws IOException {
    firstConnect = new CountDownLatch(1);

    ClientBootstrap bootstrap = new ClientBootstrap(channelFactory);

    bootstrap.setPipelineFactory(new ZKClientPipelineFactory());
    bootstrap.setOption("soLinger", -1);
    bootstrap.setOption("tcpNoDelay", true);

    connectFuture = bootstrap.connect(addr);
    connectFuture.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture channelFuture) throws Exception {
            // this lock guarantees that channel won't be assgined after cleanup().
            connectLock.lock();
            try {
                if (!channelFuture.isSuccess() || connectFuture == null) {
                    LOG.info("future isn't success, cause: {}", channelFuture.getCause());
                    return;
                }
                // 1. 建立 tcp 连接
                channel = channelFuture.getChannel();

                // 2. 初始化参数
                disconnected.set(false);
                initialized = false;
                lenBuffer.clear();
                incomingBuffer = lenBuffer;

                // 3. 发送 ConnectRequest 请求，会话创建完成
                sendThread.primeConnection();
                updateNow();
                updateLastSendAndHeard();

                if (sendThread.tunnelAuthInProgress()) {
                    waitSasl.drainPermits();
                    needSasl.set(true);
                    sendPrimePacket();
                } else {
                    needSasl.set(false);
                }

                // we need to wake up on first connect to avoid timeout.
                wakeupCnxn();
                firstConnect.countDown();
                LOG.info("channel is connected: {}", channelFuture.getChannel());
            } finally {
                connectLock.unlock();
            }
        }
    });
}
```

**(6) 发送 ConnectRequest 请求创建会话** 【SendThread】

```java
void primeConnection() throws IOException {
    LOG.info("Socket connection established, initiating session, client: {}, server: {}",
            clientCnxnSocket.getLocalSocketAddress(),
            clientCnxnSocket.getRemoteSocketAddress());
    isFirstConnect = false;
    long sessId = (seenRwServerBefore) ? sessionId : 0;
    ConnectRequest conReq = new ConnectRequest(0, lastZxid,
            sessionTimeout, sessId, sessionPasswd);
    
    // 省略... 主要是对事件的处理

    // 1. 发送连接请求
    outgoingQueue.addFirst(new Packet(null, null, conReq,
            null, null, readOnly));
    clientCnxnSocket.connectionPrimed();
    if (LOG.isDebugEnabled()) {
        LOG.debug("Session establishment request sent on "
                + clientCnxnSocket.getRemoteSocketAddress());
    }
}
```

至此，一个完整的会话就创建完成。但还有个疑问，ConnectRequest 请求放到了 outgoingQueue 中到底是如何发送的，客户端又是如何处理请求的呢？

#### 发送请求

**(7) 发送请求** 【SendThread】

```java
@Override
public void run() {
    // 1. 初始化设置，如：心跳时间、上次发送请求时间、当前时间
    clientCnxnSocket.introduce(this, sessionId, outgoingQueue);
    clientCnxnSocket.updateNow();
    clientCnxnSocket.updateLastSendAndHeard();
    int to;
    long lastPingRwServer = Time.currentElapsedTime();
    final int MAX_SEND_PING_INTERVAL = 10000; //10 seconds
    while (state.isAlive()) {
        try {
            // 2. 建立会话连接
            if (!clientCnxnSocket.isConnected()) {
                // don't re-establish connection if we are closing
                if (closing) {
                    break;
                }
                startConnect();
                clientCnxnSocket.updateLastSendAndHeard();
            }

            // 3. 设置超时时间
            if (state.isConnected()) {
                // 省略...
                to = readTimeout - clientCnxnSocket.getIdleRecv();
            } else {
                to = connectTimeout - clientCnxnSocket.getIdleRecv();
            }
            
            if (to <= 0) {
                throw new SessionTimeoutException(
                        "Client session timed out, have not heard from server in "
                                + clientCnxnSocket.getIdleRecv() + "ms"
                                + " for sessionid 0x"
                                + Long.toHexString(sessionId));
            }
            if (state.isConnected()) {
                //1000(1 second) is to prevent race condition missing to send the second ping
                //also make sure not to send too many pings when readTimeout is small 
                int timeToNextPing = readTimeout / 2 - clientCnxnSocket.getIdleSend() - 
                        ((clientCnxnSocket.getIdleSend() > 1000) ? 1000 : 0);
                //send a ping request either time is due or no packet sent out within MAX_SEND_PING_INTERVAL
                if (timeToNextPing <= 0 || clientCnxnSocket.getIdleSend() > MAX_SEND_PING_INTERVAL) {
                    sendPing();
                    clientCnxnSocket.updateLastSend();
                } else {
                    if (timeToNextPing < to) {
                        to = timeToNextPing;
                    }
                }
            }

            // 省略...
            // 5. 处理请求，由底层的 ClientCnxnSocket 完成
            clientCnxnSocket.doTransport(to, pendingQueue, ClientCnxn.this);
        } catch (Throwable e) {
            // 6. closing=true 直接会话，跳出 while 循环
            if (closing) {
                break;
            } else {
                // 7. 清空 outgoingQueue 和 pendingQueue，等待一次 while 重新建立连接
                cleanup();
                if (state.isAlive()) {
                    eventThread.queueEvent(new WatchedEvent(
                            Event.EventType.None,
                            Event.KeeperState.Disconnected,
                            null));
                }
                clientCnxnSocket.updateNow();
                clientCnxnSocket.updateLastSendAndHeard();
            }
        }
    }

    // 8. 在第 6 步跳出 while 后会关闭连接，清空 outgoingQueue 和 pendingQueue
    synchronized (state) {
        cleanup();
    }
    clientCnxnSocket.close();
    if (state.isAlive()) {
        eventThread.queueEvent(new WatchedEvent(Event.EventType.None,
                Event.KeeperState.Disconnected, null));
    }
}
```

我们可以看到 while 循环中最终调用 clientCnxnSocket.doTransport(to, pendingQueue, ClientCnxn.this) 处理请求，以 ClientCnxnSocketNetty 为例。

**(8) 处理请求** 【ClientCnxnSocketNetty】

```java
@Override
void doTransport(int waitTimeOut,
                 List<Packet> pendingQueue,
                 ClientCnxn cnxn)
        throws IOException, InterruptedException {
    try {
        if (!firstConnect.await(waitTimeOut, TimeUnit.MILLISECONDS)) {
            return;
        }
        Packet head = null;
        if (needSasl.get()) {
            if (!waitSasl.tryAcquire(waitTimeOut, TimeUnit.MILLISECONDS)) {
                return;
            }
        } else {
            if ((head = outgoingQueue.poll(waitTimeOut, TimeUnit.MILLISECONDS)) == null) {
                return;
            }
        }
        // check if being waken up on closing.
        if (!sendThread.getZkState().isAlive()) {
            // adding back the patck to notify of failure in conLossPacket().
            addBack(head);
            return;
        }
        // channel disconnection happened
        if (disconnected.get()) {
            addBack(head);
            throw new EndOfStreamException("channel for sessionid 0x"
                    + Long.toHexString(sessionId)
                    + " is lost");
        }
        if (head != null) {
            doWrite(pendingQueue, head, cnxn);
        }
    } finally {
        updateNow();
    }
}
```

doTransport 中将处理请求的委托给了 doWrite 完成，在 doWrite 中会遍历所有的 outgoingQueue 发送请求。

```java
private void doWrite(List<Packet> pendingQueue, Packet p, ClientCnxn cnxn) {
    updateNow();
    while (true) {
        if (p != WakeupPacket.getInstance()) {
            if ((p.requestHeader != null) &&
                    (p.requestHeader.getType() != ZooDefs.OpCode.ping) &&
                    (p.requestHeader.getType() != ZooDefs.OpCode.auth)) {
                p.requestHeader.setXid(cnxn.getXid());
                synchronized (pendingQueue) {
                    pendingQueue.add(p);
                }
            }
            sendPkt(p);
        }
        if (outgoingQueue.isEmpty()) {
            break;
        }
        p = outgoingQueue.remove();
    }
}

private void sendPkt(Packet p) {
    // Assuming the packet will be sent out successfully. Because if it fails,
    // the channel will close and clean up queues.
    p.createBB();
    updateLastSend();
    sentCount++;
    channel.write(ChannelBuffers.wrappedBuffer(p.bb));
}
```

#### 响应阶段

**(9) 接收响应数据，处理业务** 【ZKClientHandler】

```java
@Override
public void messageReceived(ChannelHandlerContext ctx, MessageEvent e) throws Exception {
    updateNow();
    ChannelBuffer buf = (ChannelBuffer) e.getMessage();
    while (buf.readable()) {
        // incomingBuffer 的长度大小 buf 时会抛出异常
        if (incomingBuffer.remaining() > buf.readableBytes()) {
            int newLimit = incomingBuffer.position()
                    + buf.readableBytes();
            incomingBuffer.limit(newLimit);
        }
        buf.readBytes(incomingBuffer);
        incomingBuffer.limit(incomingBuffer.capacity());

        if (!incomingBuffer.hasRemaining()) {
            incomingBuffer.flip();
            if (incomingBuffer == lenBuffer) {
                // 1. 读取长度
                recvCount++;
                readLength();
            } else if (!initialized) {
                // 2. 连接信息
                readConnectResult();
                lenBuffer.clear();
                incomingBuffer = lenBuffer;
                initialized = true;
                updateLastHeard();
            } else {
                // 3. 请求信息
                sendThread.readResponse(incomingBuffer);
                lenBuffer.clear();
                incomingBuffer = lenBuffer;
                updateLastHeard();
            }
        }
    }
    wakeupCnxn();
}
```

重点关注一下 sendThread.readResponse(incomingBuffer) 是如何处理响应的。

**(10) 数据处理** 【SendThread】

```java
void readResponse(ByteBuffer incomingBuffer) throws IOException {
    ByteBufferInputStream bbis = new ByteBufferInputStream(
            incomingBuffer);
    BinaryInputArchive bbia = BinaryInputArchive.getArchive(bbis);
    ReplyHeader replyHdr = new ReplyHeader();

    replyHdr.deserialize(bbia, "header");
    if (replyHdr.getXid() == -2) {
        // 1. -2 pings
        return;
    }
    if (replyHdr.getXid() == -4) {
        // 2. -4 认证             
        if(replyHdr.getErr() == KeeperException.Code.AUTHFAILED.intValue()) {
            state = States.AUTH_FAILED;                    
            eventThread.queueEvent( new WatchedEvent(Watcher.Event.EventType.None, 
                    Watcher.Event.KeeperState.AuthFailed, null) );                                      
        }
        return;
    }
    if (replyHdr.getXid() == -1) {
        // 3. -1 事件
        WatcherEvent event = new WatcherEvent();
        event.deserialize(bbia, "response");

        // convert from a server path to a client path
        if (chrootPath != null) {
            String serverPath = event.getPath();
            if(serverPath.compareTo(chrootPath)==0)
                event.setPath("/");
            else if (serverPath.length() > chrootPath.length())
                event.setPath(serverPath.substring(chrootPath.length()));
            else {
                LOG.warn("Got server path " + event.getPath()
                        + " which is too short for chroot path "
                        + chrootPath);
            }
        }

        WatchedEvent we = new WatchedEvent(event);
        if (LOG.isDebugEnabled()) {
            LOG.debug("Got " + we + " for sessionid 0x"
                    + Long.toHexString(sessionId));
        }

        eventThread.queueEvent( we );
        return;
    }

    // 省略...
    // 4. 处理请求响应
    Packet packet;
    synchronized (pendingQueue) {
        if (pendingQueue.size() == 0) {
            throw new IOException("Nothing in the queue, but got "
                    + replyHdr.getXid());
        }
        packet = pendingQueue.remove();
    }
    
    try {
        // 4.1 服务端一定是顺序处理请求的，但最好验证一下请求和响应的 xid 是否一致
        if (packet.requestHeader.getXid() != replyHdr.getXid()) {
            packet.replyHeader.setErr(
                    KeeperException.Code.CONNECTIONLOSS.intValue());
            throw new IOException("Xid out of order. Got Xid "
                    + replyHdr.getXid() + " with err " +
                    + replyHdr.getErr() +
                    " expected Xid "
                    + packet.requestHeader.getXid()
                    + " for a packet with details: "
                    + packet );
        }

        packet.replyHeader.setXid(replyHdr.getXid());
        packet.replyHeader.setErr(replyHdr.getErr());
        packet.replyHeader.setZxid(replyHdr.getZxid());
        if (replyHdr.getZxid() > 0) {
            lastZxid = replyHdr.getZxid();
        }
        if (packet.response != null && replyHdr.getErr() == 0) {
            packet.response.deserialize(bbia, "response");
        }
    } finally {
        // 4.2 此至 packet 封装了请求和响应的消息，还需要根据业务进行具体的处理。如：同步 
        finishPacket(packet);
    }
}
```

#### 请求处理的全过程

**(1) create** 【Zookeeper】

```java
public String create(final String path, byte data[], List<ACL> acl,CreateMode createMode)
    throws KeeperException, InterruptedException {
    final String clientPath = path;
    PathUtils.validatePath(clientPath, createMode.isSequential());

    final String serverPath = prependChroot(clientPath);

    RequestHeader h = new RequestHeader();
    h.setType(createMode.isContainer() ? ZooDefs.OpCode.createContainer : ZooDefs.OpCode.create);
    CreateRequest request = new CreateRequest();
    CreateResponse response = new CreateResponse();
    request.setData(data);
    request.setFlags(createMode.toFlag());
    request.setPath(serverPath);
    if (acl != null && acl.size() == 0) {
        throw new KeeperException.InvalidACLException();
    }
    request.setAcl(acl);
    // 请求委托 cnxn 完成
    ReplyHeader r = cnxn.submitRequest(h, request, response, null);
    if (r.getErr() != 0) {
        throw KeeperException.create(KeeperException.Code.get(r.getErr()),
                clientPath);
    }
    if (cnxn.chrootPath == null) {
        return response.getPath();
    } else {
        return response.getPath().substring(cnxn.chrootPath.length());
    }
}
```

**(2) submitRequest** 【ClientCnxn】

```java
public ReplyHeader submitRequest(RequestHeader h, Record request,
        Record response, WatchRegistration watchRegistration)
        throws InterruptedException {
    return submitRequest(h, request, response, watchRegistration, null);
}

// 同步等待请求处理完成
public ReplyHeader submitRequest(RequestHeader h, Record request,
        Record response, WatchRegistration watchRegistration,
        WatchDeregistration watchDeregistration)
        throws InterruptedException {
    ReplyHeader r = new ReplyHeader();
    Packet packet = queuePacket(h, r, request, response, null, null, null,
            null, watchRegistration, watchDeregistration);
    synchronized (packet) {
        while (!packet.finished) {
            packet.wait();
        }
    }
    return r;
}
```

**(3) queuePacket** 【ClientCnxn】

```java
Packet queuePacket(RequestHeader h, ReplyHeader r, Record request,
        Record response, AsyncCallback cb, String clientPath,
        String serverPath, Object ctx, WatchRegistration watchRegistration,
        WatchDeregistration watchDeregistration) {
    Packet packet = null;

    // 封装成 Packet
    packet = new Packet(h, r, request, response, watchRegistration);
    packet.cb = cb;
    packet.ctx = ctx;
    packet.clientPath = clientPath;
    packet.serverPath = serverPath;
    packet.watchDeregistration = watchDeregistration;
    // The synchronized block here is for two purpose:
    // 1. synchronize with the final cleanup() in SendThread.run() to avoid race
    // 2. synchronized against each packet. So if a closeSession packet is added,
    // later packet will be notified.
    synchronized (state) {
        if (!state.isAlive() || closing) {
            conLossPacket(packet);
        } else {
            if (h.getType() == OpCode.closeSession) {
                closing = true;
            }
            // 加入到 outgoingQueue 等待发送请求
            outgoingQueue.add(packet);
        }
    }
    sendThread.getClientCnxnSocket().packetAdded();
    return packet;
}
```

**(4) finishPacket** 【ClientCnxn】

```java
private void finishPacket(Packet p) {
    int err = p.replyHeader.getErr();
    if (p.watchRegistration != null) {
        p.watchRegistration.register(err);
    }
    // 处理事件
    if (p.watchDeregistration != null) {
        Map<EventType, Set<Watcher>> materializedWatchers = null;
        try {
            materializedWatchers = p.watchDeregistration.unregister(err);
            for (Entry<EventType, Set<Watcher>> entry : materializedWatchers
                    .entrySet()) {
                Set<Watcher> watchers = entry.getValue();
                if (watchers.size() > 0) {
                    queueEvent(p.watchDeregistration.getClientPath(), err,
                            watchers, entry.getKey());
                    p.replyHeader.setErr(Code.OK.intValue());
                }
            }
        } catch (KeeperException.NoWatcherException nwe) {
            LOG.error("Failed to find watcher!", nwe);
            p.replyHeader.setErr(nwe.code().intValue());
        } catch (KeeperException ke) {
            LOG.error("Exception when removing watcher", ke);
            p.replyHeader.setErr(ke.code().intValue());
        }
    }
    
    // 响应请求内容 notifyAll
    if (p.cb == null) {
        synchronized (p) {
            p.finished = true;
            p.notifyAll();
        }
    } else {
        p.finished = true;
        eventThread.queuePacket(p);
    }
}
```

**(5) cleanup** 【ClientCnxn】

注意到另一种情况，当会话断开时有可能出现死锁的情况，如何解决这个问题呢？

```java
// 会话失效时清空所有的 pendingQueue 和 outgoingQueue
private void cleanup() {
    clientCnxnSocket.cleanup();
    synchronized (pendingQueue) {
        for (Packet p : pendingQueue) {
            conLossPacket(p);
        }
        pendingQueue.clear();
    }
    // We can't call outgoingQueue.clear() here because
    // between iterating and clear up there might be new
    // packets added in queuePacket().
    Iterator<Packet> iter = outgoingQueue.iterator();
    while (iter.hasNext()) {
        Packet p = iter.next();
        conLossPacket(p);
        iter.remove();
    }
}

// 手动结束请求
private void conLossPacket(Packet p) {
    if (p.replyHeader == null) {
        return;
    }
    switch (state) {
    case AUTH_FAILED:
        p.replyHeader.setErr(KeeperException.Code.AUTHFAILED.intValue());
        break;
    case CLOSED:
        p.replyHeader.setErr(KeeperException.Code.SESSIONEXPIRED.intValue());
        break;
    default:
        p.replyHeader.setErr(KeeperException.Code.CONNECTIONLOSS.intValue());
    }
    finishPacket(p);
}
```

## Curator框架源码

Curator是Netflix公司开源的一套Zookeeper客户端框架。目前已经作为Apache的顶级项目出现，是最流行的Zookeeper客户端之一。

接着看下quick start中关于分布式锁相关的内容
地址为：http://curator.apache.org/getting-started.html

```
InterProcessMutex lock = new InterProcessMutex(client, lockPath);
if ( lock.acquire(maxWait, waitUnit) ) 
{
    try 
    {
        // do some work inside of the critical section here
    }
    finally
    {
        lock.release();
    }
}
```

使用很简单，使用`InterProcessMutex`类，使用其中的`acquire()`方法，就可以获取一个分布式锁了。

### 使用示例

启动两个线程t1和t2去争夺锁，拿到锁的线程会占用5秒。运行多次可以观察到，有时是t1先拿到锁而t2等待，有时又会反过来。Curator会用我们提供的lock路径的结点作为全局锁，这个结点的数据类似这种格式：**[_c_64e0811f-9475-44ca-aa36-c1db65ae5350-lock-00000000001]**，每次获得锁时会生成这种串，释放锁时清空数据。

接下来看看加锁的示例：

```
public class Application {
    private static final String ZK_ADDRESS = "192.20.38.58:2181";
    private static final String ZK_LOCK_PATH = "/locks/lock_01";

    public static void main(String[] args) throws InterruptedException {
        CuratorFramework client = CuratorFrameworkFactory.newClient(
                ZK_ADDRESS,
                new RetryNTimes(10, 5000)
        );
        client.start();
        System.out.println("zk client start successfully!");

        Thread t1 = new Thread(() -> {
            doWithLock(client);
        }, "t1");
        Thread t2 = new Thread(() -> {
            doWithLock(client);
        }, "t2");

        t1.start();
        t2.start();
    }

    private static void doWithLock(CuratorFramework client) {
        InterProcessMutex lock = new InterProcessMutex(client, ZK_LOCK_PATH);
        try {
            if (lock.acquire(10 * 1000, TimeUnit.SECONDS)) {
                System.out.println(Thread.currentThread().getName() + " hold lock");
                Thread.sleep(5000L);
                System.out.println(Thread.currentThread().getName() + " release lock");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                lock.release();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Curator 加锁实现原理

直接看Curator加锁的代码：

```
public class InterProcessMutex implements InterProcessLock, Revocable<InterProcessMutex> {

    private final ConcurrentMap<Thread, LockData>   threadData = Maps.newConcurrentMap();

     private static class LockData
    {
        final Thread        owningThread;
        final String        lockPath;
        final AtomicInteger lockCount = new AtomicInteger(1);

        private LockData(Thread owningThread, String lockPath)
        {
            this.owningThread = owningThread;
            this.lockPath = lockPath;
        }
    }

    @Override
    public boolean acquire(long time, TimeUnit unit) throws Exception
    {
        return internalLock(time, unit);
    }


     private boolean internalLock(long time, TimeUnit unit) throws Exception
    {
        /*
           Note on concurrency: a given lockData instance
           can be only acted on by a single thread so locking isn't necessary
        */

        Thread          currentThread = Thread.currentThread();

        LockData        lockData = threadData.get(currentThread);
        if ( lockData != null )
        {
            // re-entering
            lockData.lockCount.incrementAndGet();
            return true;
        }

        String lockPath = internals.attemptLock(time, unit, getLockNodeBytes());
        if ( lockPath != null )
        {
            LockData        newLockData = new LockData(currentThread, lockPath);
            threadData.put(currentThread, newLockData);
            return true;
        }

        return false;
    }   
}
```

直接看`internalLock()`方法，首先是获取当前线程，然后查看当前线程是否在一个concurrentHashMap中，这里是`重入锁`的实现，如果当前已经已经获取了锁，那么这个线程获取锁的次数再+1

如果没有获取锁，那么就是用`attemptLock()`方法去尝试获取锁，如果`lockPath`不为空，说明获取锁成功，并将当前线程放入到map中。

接下来看看核心的加锁逻辑`attemptLock()`方法：

入参：
`time` : 获取锁等待的时间
`unit`：时间单位
`lockNodeBytes`：默认为null

```
public class LockInternals {    
    String attemptLock(long time, TimeUnit unit, byte[] lockNodeBytes) throws Exception
    {
        final long      startMillis = System.currentTimeMillis();
        final Long      millisToWait = (unit != null) ? unit.toMillis(time) : null;
        final byte[]    localLockNodeBytes = (revocable.get() != null) ? new byte[0] : lockNodeBytes;
        int             retryCount = 0;

        String          ourPath = null;
        boolean         hasTheLock = false;
        boolean         isDone = false;
        while ( !isDone )
        {
            isDone = true;

            try
            {
                if ( localLockNodeBytes != null )
                {
                    ourPath = client.create().creatingParentsIfNeeded().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path, localLockNodeBytes);
                }
                else
                {
                    ourPath = client.create().creatingParentsIfNeeded().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path);
                }
                hasTheLock = internalLockLoop(startMillis, millisToWait, ourPath);
            }
            catch ( KeeperException.NoNodeException e )
            {
                // gets thrown by StandardLockInternalsDriver when it can't find the lock node
                // this can happen when the session expires, etc. So, if the retry allows, just try it all again
                if ( client.getZookeeperClient().getRetryPolicy().allowRetry(retryCount++, System.currentTimeMillis() - startMillis, RetryLoop.getDefaultRetrySleeper()) )
                {
                    isDone = false;
                }
                else
                {
                    throw e;
                }
            }
        }

        if ( hasTheLock )
        {
            return ourPath;
        }

        return null;
    }
}
```



```
ourPath = client.create().creatingParentsIfNeeded().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path);
```

使用的临时顺序节点，首先他是临时节点，如果当前这台机器如果自己宕机的话，他创建的这个临时节点就会自动消失，如果有获取锁的客户端宕机了，zk可以保证锁会自动释放的

