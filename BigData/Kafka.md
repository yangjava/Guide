

[TOC]

# 消息队列概述

## 分类

1. Pull to Pull - P2P ： 点对点模式
   - 市面上绝大部分的消息队列都是点对点模式或者支持点对点模式
   - 点对点模式下，数据往往只能被消费一次
2. Public/Subscribe - 发布订阅模式
   - 市面上会有较少的队列使用发布订阅模式
   - 发布订阅模式下，数据往往可以多次消费



## 消息系统的适用场景

1. 解耦：各个系统之间通过消息系统这个统一接口交换数据，无须了解彼此的存在
2. 冗余：部分消息系统拥有持久化的能力，可以规避消息处理前丢失的风险
3. 消峰限流：消息系统可顶住峰值流量，业务系统可根据处理能力从消息系统中获取数据
4. 异步通信：在不需要立即处理请求的场景下，可以讲请求放入消息系统中，在需要的时候再取出处理



## 消息队列对比

### RabbitMQ

1. RabbitMQ是使用Erlang编写的一个开源的消息队列，本身支持很多的协议：AMQP，XMPP, SMTP,  STOMP，也正因如此，它非常重量级，更适合于企业级的开发
2. 实现了Broker构架，这意味着消息在发送给客户端时先在中心队列排队。对路由，负载均衡或者数据持久化都有很好的支持

### Redis

1. Redis是一个基于Key-Value对的NoSQL数据库，开发维护很活跃
2. 虽然它是一个Key-Value数据库存储系统，但它本身支持MQ功能，所以完全可以当做一个轻量级的队列服务来使用

### ZeroMQ

1. ZeroMQ号称最快的消息队列系统，尤其针对大吞吐量的需求场景
2. ZeroMQ能够实现RabbitMQ不擅长的高级/复杂的队列，但是开发人员需要自己组合多种技术框架，技术上的复杂度是对这MQ能够应用成功的挑战
3. ZeroMQ仅提供非持久性的队列，也就是说如果宕机，数据将会丢失。其中，Twitter的Storm     0.9.0以前的版本中默认使用ZeroMQ作为数据流的传输（Storm从0.9版本开始同时支持ZeroMQ和Netty（NIO）作为传输模块）

### ActiveMQ

1. ActiveMQ是Apache下的一个子项目
2. 类似于ZeroMQ，它能够以代理人和点对点的技术实现队列
3. 类似于RabbitMQ，它少量代码就可以高效地实现高级应用场景。

# Kafka介绍

![](https://note.youdao.com/yws/api/personal/file/429E74DE8BFE492FB6890313BBA613A0?method=download&shareKey=584a7bd00b9ca963f493abd0ff560c23)

## 概述

1. Kafka是LinkedIn(领英)开发后来贡献给了Apache的一套纯粹的==发布订阅模式的、分布式的实时消息队列==

2. **Kafka是一个分布式的流处理平台，主要包含三个功能**：

   - 发布和订阅消息，类似于消息队列或者企业中的消息传递系统
   - 存储数据的时候有容错(分布式+副本机制)和持久化机制
   - 实时处理消息流（数据产生的时候处理记录【数据】）

3. **Kafka的应用场景**：

   - 能够在系统或者应用之间，构建可靠的、实时的用于获取数据流的通道
   - 能够构建一个转化或者处理数据流的应用

4. Kafka会把接收到的数据存储到本地磁盘上，而且单节点的Kafka的吞吐量是60MB/s~100MB/s

   > Kafka底层采用“零拷贝”机制

   ![](https://note.youdao.com/yws/api/personal/file/D356408191684E35AF860E733082C016?method=download&shareKey=6652da5c82f22828ba67b625b41a051b)

5. Kafka的开发语言是Scala，所以基于Scala的特性，Kafka本身就具有很好的安全性和扩展性

6. Kafka默认不会清理日志，如果需要自动清理，需要在`server.properties`中将`log.cleaner.enable`设置为true

   ```sh
   #  日志删除间隔时间
   log.retention.hours=168
   # 日志文件大小，达到这个大小会产生一个新的日志文件
   log.segment.bytes=1073741824
   # 设置是否启用日志清理
   log.cleaner.enable=true
   ```

   



## 使用场景

### Messaging

1. 对于一些常规的消息系统,kafka是个不错的选择，partitons/replication和容错,可以使kafka具有良好的扩展性和性能优势
2. kafka并没有提供JMS中的"事务性""消息传输担保(消息确认机制)""消息分组"等企业级特性；kafka只能使用作为"常规"的消息系统,在一定程度上,尚未确保消息的发送与接收绝对可靠(比如,消息重发,消息发送丢失等)

### Website activity tracking

kafka可以作为"网站活性跟踪"的最佳工具;可以将网页/用户操作等信息发送到kafka中.并实时监控,或者离线统计分析等

### Metric

Kafka通常被用于可操作的监控数据。这包括从分布式应用程序来的聚合统计用来生产集中的运营数据提要。

### Log Aggregation

==kafka的特性决定它非常适合作为"日志收集中心"==；application可以将操作日志"批量""异步"的发送到kafka集群中,而不是保存在本地或者DB中;kafka可以批量提交消息/压缩消息等，这对producer端而言,几乎感觉不到性能的开支。此时consumer端可以使hadoop等其他系统化的存储和分析系统





## 特点

1. 高吞吐率：在廉价的商用机器上单机可支持100W条/s消息
2. 消息持久化：所有的消息都会存储在磁盘上，不会产生消息的丢失
3. 支持完全分布式
4. 同时满足适应在线流处理和离线批处理





# 安装

1. 启动ZooKeeper

   ```sh
   cd /home/software/zookeeper-3.4.8/bin
   sh zkServer.sh start
   ```

2. 在第一个节点上进入software目录，下载KafKa安装包

   ```sh
   cd /home/software
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/kafka_2.11-1.0.0.tgz
   ```

3. 解压安装包

   ```sh
   tar -xvf kafka_2.11-1.0.0.tgz
   ```

4. 进入Kafka目录下的config目录

   ```sh
   cd kafka_2.11-1.0.0/config
   ```

5. 编辑`server.properties`：`vim server.properties`

   修改如下内容

   ```sh
   # 给每一个Kafka节点配置编号
   broker.id=0
   # 消息的存储陌路
   log.dirs=/home/software/kafka_2.11-1.0.0/kafka-logs
   # Zookeeper的连接地址
   zookeeper.connect=hadoop01:2181,hadoop02:2181,hadoop03:2181
   ```

6. 进入Kafka的bin目录启动Kafka

   ```sh
   cd ../bin
   sh kafka-server-start.sh ../config/server.properties
   ```

   



# 基本指令

| **命令**                                                     | **作用**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| sh kafka-topics.sh  --create --zookeeper hadoop01:2181 --replication-factor 1 --partitions 1  --topic news | 创建主题  <br />在创建的时候，副本数量要小于等于节点数量     |
| sh kafka-topics.sh --list --zookeeper hadoop01:2181          | 查看所有的topic                                              |
| sh kafka-console-producer.sh --broker-list hadoop01:9092 --topic news | 启动生产者                                                   |
| sh kafka-console-consumer.sh --zookeeper hadoop01:2181 --topic news | 启动消费者                                                   |
| sh kafka-topics.sh --delete --zookeeper hadoop01:2181  --topic news | 删除topic  <br />在删除的时候，默认情况下不是立即删除，而是会等待30~60s之后才会删除 |
| sh kafka-topics.sh --describe --zookeeper hadoop01:2181 --topic news | 描述topic的信息                                              |

[TOC]

# 基本概念

![](https://note.youdao.com/yws/api/personal/file/0B5CBC85D4BB4E6F8DE89F9657A28F4A?method=download&shareKey=e9df5ab69ec6a801ee0da1c608dc0f7c)



1. broker：经纪人

   - 在Kafka集群中，每一个Kafka节点就是一个Broker
   - 每一个Broker需要分配一个唯一的编号

2. topic：话题/主题

   - 作用：对数据进行分类
   - 在Kafka中，每一条数据必须指定一个topic，即每一条数据都是对应一个分类
   - 一个topic至少包含1个partition，可以包含多个partition

3. partition：分区

   - 每一个partition在磁盘上都会对应一个目录，一个topic有几个partition，在磁盘上就对应了几个目录
   - 如果存在多个分区，这些分区会自动平均分配到每一个Kafka节点上，这样做的目的是为了数据的负载均衡

4. replication：副本

   - 保证了数据的高可用性
   - 指定几个副本，就会将当前的topic备份几个
   - 副本数量不能超过节点的数量

5. producer：消息生产者

6. consumer：消费者

   - 从Kafka中获取数据
   - 在Kafka中数据可以被获取多次

7. leader和follower：

   - 副本之间会进行选举，自动选出一个leader副本和follower副本

   - leader和follower说的是副本关系，和当前Kafka节点没有关系

     > Kafka节点中即可能存在leader副本也可能存在follower副本

   - producer和consumer在Kafka进行交互的时候，只和leader副本进行交互

   - 当leader副本的数量发生变化的时候，同步给follower副本

8. controller：控控制器

   - 作用：在副本之中，选举出一个leader副本
   - 当Kafka集群启动的时候，ZooKeeper会在Kafka集群的某一个节点上来启动一个Controller进程
   - Controller会自动在ZooKeeper上注册一个临时节点，当Controller进程意外结束的时候，ZooKeeper上的这个节点也会随之消失，ZooKeeper就会在Kafka的其他存活节点上来重新启动一个Controller
   - ZooKeeper会自动给Controller分配一个递增的id

   ![](https://note.youdao.com/yws/api/personal/file/D84AB5154F764F159FEE0DE57DF1239A?method=download&shareKey=4ae7f9c74ad8b4c9fc37f0806e48c322)

9. Consumer group - 消费者组：

   - 可以将一个或者多个消费者放到一个消费者组中
   - 如果不指定，则默认一个消费者对应了一个消费者组
   - ==组间共享，组内竞争==：不同的消费者组可以共享相同的数据，但是数据只能被某个消费者组中的一个消费者处理



# 消息流处理

1. producer产生数据之后，将数据发送给leader中
2. 当leader收到数据之后，会将数据写道本地日志中，等待follower请求
3. follower定时给leader发送信号，询问是否有新的信息需要同步
4. 当leader收到信号之后，会将新的数据发送给follower进行同步
5. follower在收到信息之后，会将信息记录到本地日志中。如果记录成功，会给leader返回一个ACK
6. 当leader收到ACK之后，会记录这个副本所对应的brokerid(==ISR==)，同时给producer返回一个ACK
7. ISR：当leader所在节点宕机的时候，可以考虑从ISR队列中选择一个副本成为新的leader

[TOC]

# offset



# 索引