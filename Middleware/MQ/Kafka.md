# Kafka

# 一、简介

 Apache Kafka 是一个分布式的流处理平台（分布式的基于发布/订阅模式的消息队列【Message Queue】）。

流处理平台有以下3个特性：

> - 可以让你发布和订阅流式的记录。这一方面与消息队列或者企业消息系统类似。
> - 可以储存流式的记录，并且有较好的容错性。
> - 可以在流式记录产生时就进行处理。



## 1.1 消息队列的两种模式



### 1.1.1 点对点模式

生产者将消息发送到queue中，然后消费者从queue中取出并且消费消息。消息被消费以后，queue中不再存储，所以消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只能被一个消费者消费。

![img](https://static001.geekbang.org/infoq/d7/d76d96bb57db4e594a7e35c54a542aa4.png)

 



### 1.1.2 发布/订阅模式

生产者将消息发布到topic中，同时可以有多个消费者订阅该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

![img](https://static001.geekbang.org/infoq/75/75234eb455a6305c485e803336cd1436.webp)

 



## 1.2 Kafka 适合什么样的场景

它可以用于两大类别的应用：

- 构造实时流数据管道，它可以在系统或应用之间可靠地获取数据。(相当于message queue)。
- 构建实时流式应用程序，对这些流数据进行转换或者影响。(就是流处理，通过kafka stream topic和topic之间内部进行变化)。

为了理解Kafka是如何做到以上所说的功能，从下面开始，我们将深入探索Kafka的特性。

首先是一些概念：

- Kafka作为一个集群，运行在一台或者多台服务器上。
- Kafka 通过 topic 对存储的流数据进行分类。
- 每条记录中包含一个key，一个value和一个timestamp（时间戳）。



## 1.3 主题和分区

Kafka的消息通过主题（Topic）进行分类，就好比是数据库的表，或者是文件系统里的文件夹。主题可以被分为若干个分区（Partition），一个分区就是一个提交日志。消息以追加的方式写入分区，然后以先进先出的顺序读取。**注意，由于一个主题一般包含几个分区，因此无法在整个主题范围内保证消息的顺序，但可以保证消息在单个分区内的顺序。**主题是逻辑上的概念，在物理上，一个主题是横跨多个服务器的。

![img](https://static001.geekbang.org/infoq/e5/e5390d6d8ec1362888de487948dc3600.png)

 

**Kafka 集群保留所有发布的记录（无论他们是否已被消费），并通过一个可配置的参数——保留期限来控制（可以同时配置时间和消息大小，以较小的那个为准）。**举个例子， 如果保留策略设置为2天，一条记录发布后两天内，可以随时被消费，两天过后这条记录会被抛弃并释放磁盘空间。

有时候我们需要增加分区的数量，比如为了扩展主题的容量、降低单个分区的吞吐量或者要在单个消费者组内运行更多的消费者（因为一个分区只能由消费者组里的一个消费者读取）。从消费者的角度来看，基于键的主题添加分区是很困难的，因为分区数量改变，键到分区的映射也会变化，所以对于基于键的主题来说，建议在一开始就设置好分区，避免以后对其进行调整。

**（注意：不能减少分区的数量，因为如果删除了分区，分区里面的数据也一并删除了，导致数据不一致。如果一定要减少分区的数量，只能删除topic重建）**



## 1.4 生产者和消费者

**生产者（发布者）**创建消息，一般情况下，一个消息会被发布到一个特定的主题上。生产者在默认情况下把消息均衡的分布到主题的所有分区上，而并不关心特定消息会被写入哪个分区。不过，生产者也可以把消息直接写到指定的分区。这通常通过消息键和分区器来实现，分区器为键生成一个散列值，并将其映射到指定的分区上。生产者也可以自定义分区器，根据不同的业务规则将消息映射到分区。

**消费者（订阅者）**读取消息，消费者可以订阅一个或者多个主题，并按照消息生成的顺序读取它们。消费者通过检查消息的偏移量来区分已经读取过的消息。偏移量是一种元数据，它是一个不断递增的整数值，在创建消息时，kafka会把它添加到消息里。在给定的分区里，每个消息的偏移量都是唯一的。消费者把每个分区最后读取的消息偏移量保存在zookeeper或者kafka上，如果消费者关闭或者重启，它的读取状态不会丢失。

消费者是消费者组的一部分，也就是说，会有一个或者多个消费共同读取一个主题。**消费者组保证每个分区只能被同一个组内的一个消费者使用。如果一个消费者失效，群组里的其他消费者可以接管失效消费者的工作。**

![img](https://static001.geekbang.org/infoq/b9/b9b7b658f87409795cbb44a927aa33c9.png)



## 1.5 broker和集群

**broker：**一个独立的kafka服务器被称为broker。broker接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker为消费者提供服务，对读取分区的请求作出相应，返回已经提交到磁盘上的消息。

**集群：**交给同一个zookeeper集群来管理的broker节点就组成了kafka的集群。

broker是集群的组成部分，每个集群都有一个broker同时充当集群控制器的角色。控制器负责管理工作，包括将分区分配给broker和监控broker。在broker中，一个分区从属于一个broker，该broker被称为分区的首领。一个分区可以分配给多个broker（Topic设置了多个副本的时候），这时会发生分区复制。如下图：

![img](https://static001.geekbang.org/infoq/0c/0cd6427635ad6169cfe591b57149ee0b.png)

 

**broker如何处理请求：**broker会在它所监听的每个端口上运行一个Acceptor线程，这个线程会创建一个连接并把它交给Processor线程去处理。Processor线程（也叫网络线程）的数量是可配的，Processor线程负责从客户端获取请求信息，把它们放进请求队列，然后从响应队列获取响应信息，并发送给客户端。如下图所示：

 

![img](https://static001.geekbang.org/infoq/33/33d93646bdda77afd2c6ff010cb4ec73.png)

**生产请求和获取请求都必须发送给分区的首领副本（分区Leader）。**如果broker收到一个针对特定分区的请求，而该分区的首领在另外一个broker上，那么发送请求的客户端会收到一个“非分区首领”的错误响应。Kafka客户端要自己负责把生产请求和获取请求发送到正确的broker上。

客户端如何知道该往哪里发送请求呢？客户端使用了另外一种请求类型——元数据请求。这种请求包含了客户端感兴趣的主题列表，服务器的响应消息里指明了这些主题所包含的分区、每个分区都有哪些副本，以及哪个副本是首领。元数据请求可以发给任意一个broker，因为所有的broker都缓存了这些信息。客户端缓存这些元数据，并且会定时从broker请求刷新这些信息。此外如果客户端收到“非首领”错误，它会在尝试重新发送请求之前，先刷新元数据。

![img](https://static001.geekbang.org/infoq/23/23c1bf348230b60425bfce6be3d2c2cd.png)



## 1.6 Kafka 基础架构

![img](https://static001.geekbang.org/infoq/64/64e93ec83bae01eb5aaf72d3741bd7e6.png)

 



#  二、Kafka架构深入



## 2.1 Kafka工作流程及文件存储机制



### 2.1.1 工作流程

![img](https://static001.geekbang.org/infoq/f3/f316cbb244112572b1825c2850bf90f0.png)

 

Kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。

Topic是逻辑上的概念，而partition（分区）是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到哪个offset，以便出错恢复时，从上次的位置继续消费。



### 2.1.2 文件存储机制

 

![img](https://static001.geekbang.org/infoq/0a/0aecbcc2d8a765d8a6ae44084c69723b.png)

 

由于生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引的机制，将每个partition分为多个segment。（由log.segment.bytes决定，控制每个segment的大小，也可通过[log.segment.ms](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flog.segment.ms%2F)控制，指定多长时间后日志片段会被关闭）每个segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号。例如：bing这个topic有3个分区，则其对应的文件夹为：bing-0、bing-1和bing-2。

 索引文件和日志文件命名规则：每个 LogSegment 都有一个基准偏移量，用来表示当前 LogSegment 中第一条消息的 offset。偏移量是一个 64位的长整形数，固定是20位数字，长度未达到，用 0 进行填补。如下图所示：

![img](https://static001.geekbang.org/infoq/a6/a6495cf02988820a69a8478116cf8d2e.png)

 

index和log文件以当前segment的第一条消息的offset命名。index文件记录的是数据文件的offset和对应的物理位置，正是有了这个index文件，才能对任一数据写入和查看拥有O(1)的复杂度，index文件的粒度可以通过参数log.index.interval.bytes来控制，默认是是每过4096字节记录一条index。下图为index文件和log文件的结构示意图：

![img](https://static001.geekbang.org/infoq/c7/c71dabbb7663728a5f4d9ff8ef896d81.png)

 

查找message的流程（比如要查找offset为170417的message）：

> 1. 首先用二分查找确定它是在哪个Segment文件中，其中0000000000000000000.index为最开始的文件，第二个文件为0000000000000170410.index（起始偏移为170410+1 = 170411），而第三个文件为0000000000000239430.index（起始偏移为239430+1 = 239431）。所以这个offset = 170417就落在第二个文件中。其他后续文件可以依此类推，以起始偏移量命名并排列这些文件，然后根据二分查找法就可以快速定位到具体文件位置。
> 2. 用该offset减去索引文件的编号，即170417 - 170410 = 7，也用二分查找法找到索引文件中等于或者小于7的最大的那个编号。可以看出我们能够找到[4，476]这组数据，476即offset=170410 + 4 = 170414的消息在log文件中的偏移量。
> 3. 打开数据文件（0000000000000170410.log），从位置为476的那个地方开始顺序扫描直到找到offset为170417的那条Message。



### 2.1.3 数据过期机制

当日志片段大小达到log.segment.bytes指定的上限（默认是1GB）或者日志片段打开时长达到l[og.segment.ms](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flog.segment.ms%2F)时，当前日志片段就会被关闭，一个新的日志片段被打开。如果一个日志片段被关闭，就开始等待过期。**当前正在写入的片段叫做活跃片段，活跃片段永远不会被删除，**所以如果你要保留数据1天，但是片段包含5天的数据，那么这些数据就会被保留5天，因为片段被关闭之前，这些数据无法被删除。



## 2.2 Kafka生产者



### 2.2.1 分区策略

> 1. 多Partition分布式存储，利于集群数据的均衡。
> 2. 并发读写，加快读写速度。
> 3. 加快数据恢复的速率：当某台机器挂了，每个Topic仅需恢复一部分的数据，多机器并发。

**分区的原则**

> 1. 指明partition的情况下，使用指定的partition；
> 2. 没有指明partition，但是有key的情况下，将key的hash值与topic的partition数进行取余得到partition值；
> 3. 既没有指定partition，也没有key的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与topic可用的partition数取余得到partition值，也就是常说的round-robin算法。

```java
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
    int numPartitions = partitions.size();
    if (keyBytes == null) {
        //key为空时，获取一个自增的计数，然后对分区做取模得到分区编号
        int nextValue = nextValue(topic);
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        if (availablePartitions.size() > 0) {
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return availablePartitions.get(part).partition();
        } else {
            // no partitions are available, give a non-available partition
            return Utils.toPositive(nextValue) % numPartitions;
        }
    } else {
        // hash the keyBytes to choose a partition
        // key不为空时，通过key的hash对分区取模（疑问：为什么这里不像上面那样，使用availablePartitions呢？）
        // 根据《Kafka权威指南》Page45理解：为了保证相同的键，总是能路由到固定的分区，如果使用可用分区，那么因为分区数变化，会导致相同的key，路由到不同分区
        // 所以如果要使用key来映射分区，最好在创建主题的时候就把分区规划好
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
}
 
private int nextValue(String topic) {
    //为每个topic维护了一个AtomicInteger对象，每次获取时+1
    AtomicInteger counter = topicCounterMap.get(topic);
    if (null == counter) {
        counter = new AtomicInteger(ThreadLocalRandom.current().nextInt());
        AtomicInteger currentCounter = topicCounterMap.putIfAbsent(topic, counter);
        if (currentCounter != null) {
            counter = currentCounter;
        }
    }
    return counter.getAndIncrement();
}
```

 



### 2.2.2 数据可靠性保证

 

**kafka提供了哪些方面的保证**

- kafka可以保证分区消息的顺序。如果使用同一个生产者往同一个分区写入消息，而且消息B在消息A之后写入，那么kafka可以保证消息B的偏移量比消息A的偏移量大，而且消费者会先读取到消息A再读取消息B。
- 只有当消息被写入分区的所有副本时，它才被认为是“已提交”的。生产者可以选择接收不同类型的确认，比如在消息被完全提交时的确认、在消息被写入分区首领时的确认，或者在消息被发送到网络时的确认。
- 只要还有一个副本是活跃的，那么已经提交的信息就不会丢失。
- 消费者只能读取到已经提交的消息。

**复制**

Kafka的复制机制和分区的多副本架构是kafka可靠性保证的核心。把消息写入多个副本可以使kafka在发生奔溃时仍能保证消息的持久性。

kafka的topic被分成多个分区，分区是基本的数据块。每个分区可以有多个副本，其中一个是首领。所有事件都是发给首领副本，或者直接从首领副本读取事件。其他副本只需要与首领副本保持同步，并及时复制最新的事件。

Leader维护了一个动态的in-sync replica set（ISR），意为和leader保持同步的follower集合。当ISR中的follower完成数据同步后，leader就会发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由[replica.lag.time.max.ms](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Freplica.lag.time.max.ms%2F)参数设定。Leader不可用时，将会从ISR中选举新的leader。满足以下条件才能被认为是同步的：

- 与zookeeper之间有一个活跃的会话，也就是说，它在过去的6s（可配置）内向zookeeper发送过心跳。
- 在过去的10s（可配置）内从首领那里获取过最新的数据。

**影响Kafka消息存储可靠性的配置**

![img](https://static001.geekbang.org/infoq/d5/d5f8b70b898645c9aae7b6438d3ba590.webp)

**ack应答机制**

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没有必要等ISR中的follower全部接收成功。所以Kafka提供了三种可靠性级别，用户可以根据对可靠性和延迟的要求进行权衡。acks：

> -  **0：** producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接收到还没写入磁盘就已经返回，当broker故障时可能丢失数据；
> -  **1：** producer等待leader的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么将会丢失数据；
> -  **-1（all）：**producer等待broker的ack，partition的leader和ISR里的follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成重复数据。（极端情况下也有可能丢数据：ISR中只有一个Leader时，相当于1的情况）。

**消费一致性保证**

![img](https://static001.geekbang.org/infoq/83/839ec275ddac91ee821c39bb4c97ff67.png)

 

**（1）follower故障**

 follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。

等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。

**（2）leader故障**

 leader发生故障后，会从ISR中选出一个新的leader，之后为了保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。

 注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。



### 2.2.3 消息发送流程

**Kafka 的producer 发送消息采用的是异步发送的方式。**在消息发送过程中，涉及到了两个线程——main线程和sender线程，以及一个线程共享变量——RecordAccumulator。main线程将消息发送给RecordAccumulator，sender线程不断从RecordAccumulator中拉取消息发送到Kafka broker。

![img](https://static001.geekbang.org/infoq/04/04b988b7c4f2f4638244c2f63f4dc06b.png)

 

为了提高效率，消息被分批次写入kafka。批次就是一组消息，这些消息属于同一个主题和分区。（如果每一个消息都单独穿行于网络，会导致大量的网络开销，把消息分成批次传输可以减少网络开销。不过要在时间延迟和吞吐量之间做出权衡：批次越大，单位时间内处理的消息就越多，单个消息的传输时间就越长）。批次数据会被压缩，这样可以提升数据的传输和存储能力，但要做更多的计算处理。

**相关参数：**

- **batch.size：**只有数据积累到batch.size后，sender才会发送数据。（单位：字节，注意：不是消息个数）。
- [linger.ms](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flinger.ms%2F)**：**如果数据迟迟未达到batch.size，sender等待 [linger.ms](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flinger.ms%2F)之后也会发送数据。（单位：毫秒）。
- [client.id](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fclient.id%2F)**：**该参数可以是任意字符串，服务器会用它来识别消息的来源，还可用用在日志和配额指标里。
- [max.in](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fmax.in%2F)**.flight.requests.per.connection：**该参数指定了生产者在收到服务器响应之前可以发送多少个消息。它的值越高，就会占用越多的内存，不过也会提升吞吐量。**把它设置为1可以保证消息时按发送的顺序写入服务器的，即使发生了重试。**



## 2.3 Kafka消费者



### 2.3.1 消费方式

 consumer采用pull（拉）的模式从broker中读取数据。

 push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是尽可能以最快的速度传递消息，但是这样容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式可以根据consumer的消费能力以适当的速率消费消息。

 pull模式的不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一点，kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可消费，consumer会等待一段时间后再返回。



### 2.3.2 分区分配策略

 一个consumer group中有多个consumer，一个topic有多个partition，所以必然会涉及到partition的分配问题，即确定哪个partition由哪个consumer来消费。Kafka提供了3种消费者分区分配策略：RangeAssigor、RoundRobinAssignor、StickyAssignor。

 PartitionAssignor接口用于用户定义实现分区分配算法，以实现Consumer之间的分区分配。消费组的成员订阅它们感兴趣的Topic并将这种订阅关系传递给作为订阅组协调者的Broker。协调者选择其中的一个消费者来执行这个消费组的分区分配并将分配结果转发给消费组内所有的消费者。Kafka默认采用RangeAssignor的分配算法。



#### 2.3.2.1 RangeAssignor

 RangeAssignor对每个Topic进行独立的分区分配。对于每一个Topic，首先对分区按照分区ID进行排序，然后订阅这个Topic的消费组的消费者再进行排序，之后尽量均衡的将分区分配给消费者。这里只能是尽量均衡，因为分区数可能无法被消费者数量整除，那么有一些消费者就会多分配到一些分区。分配示意图如下：

 

![img](https://static001.geekbang.org/infoq/11/11fb2f572900bb2560ddb61eb414f408.webp)

 

分区分配的算法如下：

```java
@Override
public Map<String, List<TopicPartition>> assign(Map<String, Integer> partitionsPerTopic,
                                                Map<String, Subscription> subscriptions) {
    Map<String, List<String>> consumersPerTopic = consumersPerTopic(subscriptions);
    Map<String, List<TopicPartition>> assignment = new HashMap<>();
    for (String memberId : subscriptions.keySet())
        assignment.put(memberId, new ArrayList<TopicPartition>());
    //for循环对订阅的多个topic分别进行处理
    for (Map.Entry<String, List<String>> topicEntry : consumersPerTopic.entrySet()) {
        String topic = topicEntry.getKey();
        List<String> consumersForTopic = topicEntry.getValue();
 
        Integer numPartitionsForTopic = partitionsPerTopic.get(topic);
        if (numPartitionsForTopic == null)
            continue;
        //对消费者进行排序
        Collections.sort(consumersForTopic);
        //计算平均每个消费者分配的分区数
        int numPartitionsPerConsumer = numPartitionsForTopic / consumersForTopic.size();
        //计算平均分配后多出的分区数
        int consumersWithExtraPartition = numPartitionsForTopic % consumersForTopic.size();
 
        List<TopicPartition> partitions = AbstractPartitionAssignor.partitions(topic, numPartitionsForTopic);
        for (int i = 0, n = consumersForTopic.size(); i < n; i++) {
            //计算第i个消费者，分配分区的起始位置
            int start = numPartitionsPerConsumer * i + Math.min(i, consumersWithExtraPartition);
            //计算第i个消费者，分配到的分区数量
            int length = numPartitionsPerConsumer + (i + 1 > consumersWithExtraPartition ? 0 : 1);
            assignment.get(consumersForTopic.get(i)).addAll(partitions.subList(start, start + length));
        }
    }
    return assignment;
}
```

这种分配方式明显的一个问题是随着消费者订阅的Topic的数量的增加，不均衡的问题会越来越严重，比如上图中4个分区3个消费者的场景，C0会多分配一个分区。如果此时再订阅一个分区数为4的Topic，那么C0又会比C1、C2多分配一个分区，这样C0总共就比C1、C2多分配两个分区了，而且随着Topic的增加，这个情况会越来越严重。分配结果：

![img](https://static001.geekbang.org/infoq/b7/b756c3df735085b5154684ed3549212a.webp)

 

订阅2个Topic，每个Topic4个分区，共3个Consumer

- **C0：**[T0P0，T0P1，T1P0，T1P1]
- **C1：**[T0P2，T1P2]
- **C2：**[T0P3，T1P3]



#### 2.3.2.2 RoundRobinAssignor

RoundRobinAssignor的分配策略是将消费组内订阅的所有Topic的分区及所有消费者进行排序后尽量均衡的分配（RangeAssignor是针对单个Topic的分区进行排序分配的）。如果消费组内，消费者订阅的Topic列表是相同的（每个消费者都订阅了相同的Topic），那么分配结果是尽量均衡的（消费者之间分配到的分区数的差值不会超过1）。如果订阅的Topic列表是不同的，那么分配结果是不保证“尽量均衡”的，因为某些消费者不参与一些Topic的分配。

![img](https://static001.geekbang.org/infoq/52/52ac1ace14ea67e0488a0ad63890174d.webp)

 

以上两个topic的情况，相比于之前RangeAssignor的分配策略，可以使分区分配的更均衡。不过考虑这种情况，假设有三个消费者分别为C0、C1、C2，有3个Topic T0、T1、T2，分别拥有1、2、3个分区，并且C0订阅T0，C1订阅T0和T1，C2订阅T0、T1、T2，那么RoundRobinAssignor的分配结果如下：

 

![img](https://static001.geekbang.org/infoq/23/23e43e7dd78f1bacc43347192e2a33b5.webp)

 

看上去分配已经尽量的保证均衡了，不过可以发现C2承担了4个分区的消费而C1订阅了T1，是不是把T1P1交给C1消费能更加的均衡呢？



#### 2.3.2.3 StickyAssignor

StickyAssignor分区分配算法，目的是在执行一次新的分配时，能在上一次分配的结果的基础上，尽量少的调整分区分配的变动，节省因分区分配变化带来的开销。Sticky是“粘性的”，可以理解为分配结果是带“粘性的”——每一次分配变更相对上一次分配做最少的变动。其目标有两点：

- 分区的分配尽量的均衡。
- 每一次重分配的结果尽量与上一次分配结果保持一致。

当这两个目标发生冲突时，优先保证第一个目标。第一个目标是每个分配算法都尽量尝试去完成的，而第二个目标才真正体现出StickyAssignor特性的。

StickyAssignor算法比较复杂，下面举例来说明分配的效果（对比RoundRobinAssignor），前提条件：

- 有4个Topic：T0、T1、T2、T3，每个Topic有2个分区。
- 有3个Consumer：C0、C1、C2，所有Consumer都订阅了这4个分区。

 

![img](https://static001.geekbang.org/infoq/b0/b0c06dc1c92973ed6a1b28ccfef093e3.webp)

 

上面红色的箭头代表的是有变动的分区分配，可以看出，StickyAssignor的分配策略，变动较小。



### 2.3.3 offset的维护

由于Consumer在消费过程中可能会出现断电宕机等故障，Consumer恢复后，需要从故障前的位置继续消费，所以Consumer需要实时记录自己消费到哪个位置，以便故障恢复后继续消费。Kafka0.9版本之前，Consumer默认将offset保存在zookeeper中，从0.9版本开始，Consumer默认将offset保存在Kafka一个内置的名字叫_consumeroffsets的topic中。默认是无法读取的，可以通过设置consumer.properties中的exclude.internal.topics=false来读取。



#### 2.3.4 kafka高效读写数据（了解）

**顺序写磁盘**

Kafka 的 producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。数据表明，同样的磁盘，顺序写能到600M/s，而随机写只有100K/s。这与磁盘的机械结构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

**零拷贝技术**

零拷贝主要的任务就是避免CPU将数据从一块存储拷贝到另外一块存储，主要就是利用各种零拷贝技术，避免让CPU做大量的数据拷贝任务，减少不必要的拷贝，或者让别的组件来做这一类简单的数据传输任务，让CPU解脱出来专注于别的任务。这样就可以让系统资源的利用更加有效。



# 参考文献

1. [Kafka中文文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fkafka.apachecn.org%2F)
2. [[Kafka系列\]之指定了一个offset,怎么查找到对应的消息？](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb32ac197aacb)
3. [尚硅谷 Kafka 教程( Kafka 框架快速入门)](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1a4411B7V9)
4. [Kafka分区分配策略分析——重点：StickyAssignor](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.cnblogs.com%2Fhzmark%2Fp%2Fsticky_assignor.html)
5. [Kafka 日志存储](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F65415304)
6. [浅析Linux中的零拷贝技术](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.jianshu.com%2Fp%2Ffad3339e3448)
7. 《Kafka权威指南》