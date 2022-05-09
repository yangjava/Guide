## 什么是消息队列

消息队列（Message Queue）是一种进程间通信或同一进程的不同线程间的通信方式。

## 消息中间件应用场景

以下介绍消息队列在实际应用中常用的使用场景。异步处理，应用解耦，流量削锋和消息通讯等场景。

**什么时候需要用ActiveMQ**

- 异步处理 - 相比于传统的串行、并行方式，提高了系统吞吐量。
- 应用解耦 - 系统间通过消息通信，不用关心其他系统的处理。
- 流量削锋 - 可以通过消息队列长度控制请求量；可以缓解短时间内的高并发请求。
- 日志处理 - 解决大量日志传输。
- 消息通讯 - 消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。

### 异步处理

场景说明：汽车触发围栏报警后，需要发送报警邮件和报警短信。传统的做法有两种1.串行的方式；2.并行方式。

（1）串行方式：将报警信息写入数据库成功后，发送报警邮件，再发送报警短信。以上三个任务全部完成后，该报警信息加入统计列表。

（2）并行方式：报警信息写入数据库成功后，同时发送报警邮件和短信。

假设三个业务节点每个使用50毫秒钟，不考虑网络等其他开销，则串行方式的时间是150毫秒，并行的时间可能是100毫秒。

因为CPU在单位时间内处理的请求数是一定的，假设CPU1秒内吞吐量是100次。则串行方式1秒内CPU可处理的请求量是7次（1000/150）。并行方式处理的请求量是10次（1000/100）。

小结：如以上案例描述，传统的方式系统的性能（并发量，吞吐量，响应时间）会有瓶颈。如何解决这个问题呢？

引入消息队列，将不是必须的业务逻辑，异步处理。改造后的架构如下：

报警信息写入数据库成功后，直接响应，邮件和短信发送通过消息队列异步处理。

### 应用解耦

场景介绍，在spring cloud分布式微服务项目中，工单管理和设备管理分别是两个微服务，如果A工单被张三接单了，那么工单状态要设为已派单，检验员设为张三，设备状态要置为在检。

传统的做法是，先调用工单管理的工单更新接口，成功之后再调用设备管理的设备更新接口，成功之后再返回操作提示给用户。这样做的缺点是应用耦合，如果在派单操作的时候正好设备管理微服务挂了或者阻塞了，那么派单操作就会失败或者要等待很长时间无反馈。另外如果设备管理的接口有变动，那么工单管理里面的代码也要改动。

引入消息中间件，派单的时候，工单管理的工单更新接口处理好后把信息写入消息队列，然后直接返回操作反馈给用户。不管工单管理服务正不正常，正常就从消息队列里订阅消息处理，不正常就等待回复正常后再订阅消息处理。

### 流量削峰

场景介绍，一个典型应用场景：秒杀活动，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列。这样，用户的请求，服务器接收后，首先写入消息队列。。

这样做有如下好处：

- 可以控制活动的人数：假如消息队列长度超过最大数量，则直接抛弃用户请求或跳转到错误页面
- 可以缓解短时间内高流量压垮应用：秒杀业务根据消息队列中的请求信息，再做后续处理

### 消息通讯

消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。

点对点通讯：

客户端A和客户端B使用同一队列，进行消息通讯。

聊天室通讯：

客户端A，客户端B，客户端N订阅同一主题，进行消息发布和接收。实现类似聊天室效果。

以上实际是消息队列的两种消息模式，点对点或发布订阅模式。

## 什么时候需要消息队列

- **异步处理：例如短信通知、终端状态推送、App推送、用户注册等**
  有些业务不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。
- **数据同步：业务数据推送同步**
- **重试补偿：记账失败重试**
- **系统解耦：通讯上下行、终端异常监控、分布式事件中心**
  降低工程间的强依赖程度，针对异构系统进行适配。在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。通过消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口，当应用发生变化时，可以独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束

- **流量消峰：秒杀场景下的下单处理**
  在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量无法提取预知；如果以为了能处理这类瞬间峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
- **发布订阅：HSF的服务状态变化通知、分布式事件中心**
- **数据流处理：日志服务、监控上报**
  分布式系统产生的海量数据流，如：业务日志、监控数据、用户行为等，针对这些数据流进行实时或批量采集汇总，然后进行大数据分析是当前互联网的必备技术，通过消息队列完成此类数据收集是最好的选择
- **分布式事务**
- **RPC调用**

## 消息队列核心概念

**Broker（消息服务器）**
Broker的概念来自与Apache ActiveMQ，通俗的讲就是MQ的服务器。

**Producer（生产者）**
业务的发起方，负责生产消息传输给broker

**Consumer（消费者）**
业务的处理方，负责从broker获取消息并进行业务逻辑处理

**Topic（主题）**
发布订阅模式下的消息统一汇集地，不同生产者向topic发送消息，由MQ服务器分发到不同的订阅 者，实现消息的广播

**Queue（队列）**
PTP模式下，特定生产者向特定queue发送消息，消费者订阅特定的queue完成指定消息的接收。

- 本地队列
  本地队列按照功能可划分为初始化队列，传输队列，目标队列和死信队列。
  初始化队列：用作消息触发功能。
  传输队列：是暂存待传的消息，条件许可的情况下，通过管道将消息传送到其他的队列管理器。
  目标队列：是消息的目的地，可以长期存放消息。
  死信队列：如果消息不能送达目标队列，也不能再路由出去，则被自动放入死信队列保存。
- 别名队列&远程队列
  是一个队列定义，用来指定远端队列管理器的队列。使用了远程队列，程序就不需要知道目标队列的位置。
- 模型队列
  模型队列定义了一套本地队列的属性结合，一旦打开模型队列，队列管理器会按照这些属性动态地创建出一个本地队列。

**Message（消息体）**
根据不同通信协议定义的固定格式进行编码的数据包，来封装业务数据，实现消息的传输

## 消息模式

### PTP点对点

点对点模型用于消息生产者和消息消费者之间点到点的通信。

![img](https://www.zixi.org/static/uploads/2020/9/20200930104437.png)

点对点模式包含三个角色：

- 消息队列（Queue）
- 发送者(Sender)
- 接收者(Receiver)

每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，可以放在内存 中也可以持久化，直到他们被消费或超时。

特点：

- 每个消息只有一个消费者（Consumer）(即一旦被消费，消息就不再在消息队列中)
- 发送者和接收者之间在时间上没有依赖性
- 接收者在成功接收消息之后需向队列应答成功
- 利用FIFO先进先出的特性，可以保证消息的顺序性。

### Pub/Sub发布订阅

![img](https://www.zixi.org/static/uploads/2020/9/20200930104439.png)

发布订阅模型包含三个角色：

- 主题（Topic）
- 发布者（Publisher）
- 订阅者（Subscriber）

多个发布者将消息发送到Topic，系统将这些消息传递给多个订阅者。

特点：

- 每个消息可以有多个消费者：和点对点方式不同，发布消息可以被所有订阅者消费
- 发布者和订阅者之间有时间上的依赖性。
- 针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息。
- 为了消费消息，订阅者必须保持运行的状态。

## 常用协议

### AMQP

AMQP即Advanced Message Queuing Protocol，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP 的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

优点：可靠、通用

### MQTT

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是IBM开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和致动器（比如通过Twitter让房屋联网）的通信协议。

优点：格式简洁、占用带宽小、移动端通信、PUSH、嵌入式系统

### STOMP

STOMP（Streaming Text Orientated Message Protocol）是流文本定向消息协议，是一种为MOM(Message Oriented Middleware，面向消息的中间件)设计的简单文本协议。STOMP提供一个可互操作的连接格式，允许客户端与任意STOMP消息代理（Broker）进行交互。

优点：命令模式（非topic\queue模式）

### XMPP

XMPP（可扩展消息处理现场协议，Extensible Messaging and Presence Protocol）是基于可扩展标记语言（XML）的协议，多用于即时消息（IM）以及在线现场探测。适用于服务器之间的准即时操作。核心是基于XML流传输，这个协议可能最终允许因特网用户向因特网上的其他任何人发送即时消息，即使其操作系统和浏览器不同。

优点：通用公开、兼容性强、可扩展、安全性高，但XML编码格式占用带宽大

## 常用MQ产品对比（RabbitMq、Kafaka）

### 架构方面

- Kafaka是正常的mq架构，包括provider broker consumer。**Kafaka（默认）没有消息确认机制**。
- RabbitMq中的broker由exchange、binder queue三部分组成，其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费，**RabbitMq有消息确认机制**。

### 吞吐量方面

- Kafaka采用zero-copy方式，即数据存储和获取是本地磁盘顺序批量操作，具有O(1)复杂度，数据处理效率很高；
- RabbitMq在吞吐量方面不如Kafaka，**RabbitMq支持对消息可靠的传递，支持事务，不支持批量的操作**；

### 可用性方面

- Kafka的broker采用主备模式，所以可用性很高；
- RabbitMq支持miror queue，主queue失效，minor queue生效；

### 集群负载方面

- Kafaka使用zookeeper实现负载均衡，zookeeper管理集群中的broker sonsumer，通过zookeeper的协调机制，producer会记录topic对应的broker，对broker进行轮询或者随机访问broker，实现负载均衡；
- RabbitMq需要单独自定义负载均衡；

### 总结

| MQ       | 吞吐量                           | 应用场景                                                     | 特点                                                         |
| -------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| RabbitMq | 3500-4000msg/s                   | 非海量高可靠场景，大规模企业应用、ESB、复杂路由策略、易购系统整合 | 协议丰富、兼容性强、功能完善、消息格式比较大、速度较慢、消息持久化对性能影响较大 |
| ZeroMq   | >800000msg/s                     | 高并发连接场景，如：在线游戏。海量高实时性场景。如：股票行情 | 偏重于网络开发，开发成本高，高级功能需自行实现，不建议做传统MQ使用 |
| ActiveMq | ~3600msg/s                       | 非海量高可靠场景、企业级应用、分布式事务（XA）、异构系统整合 | 相对RabbitMq较轻量级、性能相近、完整JMS支持、配置较复杂      |
| Redis    | ~15000msg/s                      | 高吞吐低延时、大量小消息体（<10k）、顺序性或排序要求、异构系统整合 | 轻量级MQ的快速简单实现、容灾与负载等功能需自行实现           |
| Kafka    | IN ~70000msg/s，OUT >150000msg/s | 日志等海量数据流、DB数据同步、高堆积离线数据处理             | 非典型MQ更偏向于流式数据批处理                               |

## RabbitMQ

RabbitMQ 是实现 AMQP（高级消息队列协议）的消息中间件的一种，最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。 RabbitMQ 主要是为了实现系统之间的双向解耦而实现的。当生产者大量产生数据时，消费者无法快速消费，那么需要一个中间层。保存这个数据。

RabbitMQ 是一个开源的 AMQP 实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP 等，支持 AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

![img](https://www.zixi.org/static/uploads/2020/9/20200930104441.png)

### 关键字

**Channel（通道）**
道是两个管理器之间的一种单向点对点的的通信连接，如果需要双向交流，可以建立一对通道。

**Exchange（消息交换机）**
Exchange类似于数据通信网络中的交换机，提供消息路由策略。

RabbitMq中，producer不是通过信道直接将消息发送给queue，而是先发送给Exchange。一个Exchange可以和多个Queue进行绑定，producer在传递消息的时候，会传递一个ROUTING_KEY，Exchange会根据这个ROUTING_KEY按照特定的路由算法，将消息路由给指定的queue。和Queue一样，Exchange也可设置为持久化，临时或者自动删除。

Exchange有4种类型：direct(默认)，fanout， topic， 和headers。
不同类型的Exchange转发消息的策略有所区别：

- direct
  直接交换器，工作方式类似于单播，Exchange会将消息发送完全匹配ROUTING_KEY的Queue
- fanout
  广播是式交换器，不管消息的ROUTING_KEY设置为什么，Exchange都会将消息转发给所有绑定的Queue。
- topic
  主题交换器，工作方式类似于组播，Exchange会将消息转发和ROUTING_KEY匹配模式相同的所有队列，比如，ROUTING_KEY为user.stock的Message会转发给绑定匹配模式为 * .stock，user.stock， * . * 和#.user.stock.#的队列。（ * 表是匹配一个任意词组，#表示匹配0个或多个词组）
- headers
  消息体的header匹配（ignore）

**Binding（绑定）**
所谓绑定就是将一个特定的 Exchange 和一个特定的 Queue 绑定起来。Exchange 和Queue的绑定可以是多对多的关系。

**Routing Key（路由关键字）**
exchange根据这个关键字进行消息投递。

**vhost（虚拟主机）**
在RabbitMq server上可以创建多个虚拟的message broker，又叫做virtual hosts (vhosts)。每一个vhost本质上是一个mini-rabbitmq server，分别管理各自的exchange，和bindings。vhost相当于物理的server，可以为不同app提供边界隔离，使得应用安全的运行在不同的vhost实例上，相互之间不会干扰。producer和consumer连接rabbit server需要指定一个vhost。

### 消息发送及接受过程

假设P1和C1注册了相同的Broker，Exchange和Queue。P1发送的消息最终会被C1消费。
基本的通信流程大概如下所示：

- P1生产消息，发送给服务器端的Exchange
- Exchange收到消息，根据ROUTINKEY，将消息转发给匹配的Queue1
- Queue1收到消息，将消息发送给订阅者C1
- C1收到消息，发送ACK给队列确认收到消息
- Queue1收到ACK，删除队列中缓存的此条消息

Consumer收到消息时需要显式的向rabbit broker发送basic。ack消息或者consumer订阅消息时设置auto_ack参数为true。

在通信过程中，队列对ACK的处理有以下几种情况：

- 如果consumer接收了消息，发送ack，rabbitmq会删除队列中这个消息，发送另一条消息给consumer。
- 如果cosumer接受了消息， 但在发送ack之前断开连接，rabbitmq会认为这条消息没有被deliver，在consumer在次连接的时候，这条消息会被redeliver。
- 如果consumer接受了消息，但是程序中有bug，忘记了ack，rabbitmq不会重复发送消息。
- rabbitmq2。0。0和之后的版本支持consumer reject某条（类）消息，可以通过设置requeue参数中的reject为true达到目地，那么rabbitmq将会把消息发送给下一个注册的consumer。

### 消息的ACK机制

即消息的Ackownledge确认机制，为了保证消息不丢失，消息队列提供了消息Acknowledge机制，即ACK机制，当Consumer确认消息已经被消费处理，发送一个ACK给消息队列，此时消息队列便可以删除这个消息了。如果Consumer宕机/关闭，没有发送ACK，消息队列将认为这个消息没有被处理，会将这个消息重新发送给其他的Consumer重新消费处理。

### 消息的事务支持

消息的收发处理支持事务，例如：在任务中心场景中，一次处理可能涉及多个消息的接收、处理，这应该处于同一个事务范围内，如果一个消息处理失败，事务回滚，消息重新回到队列中。

### 消息的持久化

消息的持久化，对于一些关键的核心业务来说是非常重要的，启用消息持久化后，消息队列宕机重启后，消息可以从持久化存储恢复，消息不丢失，可以继续消费处理。

### 消息处理模式

**fanout 模式**
模式特点：

- 可以理解他是一个广播模式
- 不需要routing key它的消息发送时通过Exchange binding进行路由的~~在这个模式下routing key失去作用
- 这种模式需要提前将Exchange与Queue进行绑定，一个Exchange可以绑定多个Queue，一个Queue可以同多个Exchange进行绑定
- 如果接收到消息的Exchange没有与任何Queue绑定，则消息会被抛弃。

**direct 模式**
任何发送到Direct Exchange的消息都会被转发到routing_key中指定的Queue。

- 一般情况可以使用rabbitMQ自带的Exchange：”” (该Exchange的名字为空字符串)， 也可以自定义Exchange
- 这种模式下不需要将Exchange进行任何绑定(bind)操作。当然也可以进行绑定。可以将不同的routing_key与不同的queue进行绑定，不同的queue与不同exchange进行绑定
- 消息传递时需要一个“routing_key”
- 如果消息中不存在routing_key中绑定的队列名，则该消息会被抛弃。

如果一个exchange 声明为direct，并且bind中指定了routing_key，那么发送消息时需要同时指明该exchange和routing_key。

简而言之就是：生产者生成消息发送给Exchange， Exchange根据Exchange类型和basic_publish中的routing_key进行消息发送 消费者：订阅Exchange并根据Exchange类型和binding key(bindings 中的routing key) ，如果生产者和订阅者的routing_key相同，Exchange就会路由到那个队列。

**topic 模式**
前面讲到direct类型的Exchange路由规则是完全匹配binding key与routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。

topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中，但这里的匹配规则有些不同。
它约定：

- routing key为一个句点号“. ”分隔的字符串（我们将被句点号“。 ”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”
- binding key与routing key一样也是句点号“. ”分隔的字符串
- binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）

![img](https://www.zixi.org/static/uploads/2020/9/20200930104442.png)

以上图中的配置为例，routingKey=”quick.orange.rabbit”的消息会同时路由到Q1与Q2，routingKey=”lazy.orange.fox”的消息会路由到Q1，routingKey=”lazy.brown.fox”的消息会路由到Q2，routingKey=”lazy.pink.rabbit”的消息会路由到Q2（只会投递给Q2一次，虽然这个routingKey与Q2的两个bindingKey都匹配）；routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”的消息将会被丢弃，因为它们没有匹配任何bindingKey。

### 集群

RabbitMQ，部署分三种模式：单机模式，普通集群模式，镜像集群模式。

**普通集群模式**
多台机器部署，每个机器放一个rabbitmq实例，但是创建的queue只会放在一个rabbitmq实例上，每个实例同步queue的元数据。

![img](https://www.zixi.org/static/uploads/2020/9/20200930104444.png)

如果消费时连的是其他实例，那个实例会从queue所在实例拉取数据。这就会导致拉取数据的开销，如果那个放queue的实例宕机了，那么其他实例就无法从那个实例拉取，即便开启了消息持久化，让rabbitmq落地存储消息的话，消息不一定会丢，但得等这个实例恢复了，然后才可以继续从这个queue拉取数据，**这就没什么高可用可言，主要是提供吞吐量**，让集群中多个节点来服务某个queue的读写操作。

**镜像集群模式**

![img](https://www.zixi.org/static/uploads/2020/9/20200930104446.png)

queue的元数据和消息都会存放在多个实例，每次写消息就自动同步到多个queue实例里。这样任何一个机器宕机，其他机器都可以顶上，但是性能开销太大，消息同步导致网络带宽压力和消耗很重，另外，没有扩展性可言，如果queue负载很重，加机器，新增的机器也包含了这个queue的所有数据，并没有办法线性扩展你的queue。此时，需要开启镜像集群模式，在rabbitmq管理控制台新增一个策略，将数据同步到指定数量的节点，然后你再次创建queue的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。

## Kafka

### 介绍

Kafka 是 Apache 的子项目，是一个高性能跨语言的分布式发布/订阅消息队列系统（没有严格实现 JMS 规范的点对点模型，但可以实现其效果），在企业开发中有广泛的应用。高性能是其最大优势，劣势是消息的可靠性（丢失或重复），这个劣势是为了换取高性能，开发者可以以稍降低性能，来换取消息的可靠性。

![img](https://www.zixi.org/static/uploads/2020/9/20200930104447.png)

![img](https://www.zixi.org/static/uploads/2020/9/20200930104449.png)

### Topics/logs

一个Topic可以认为是一类消息，每个topic将被分成多个partition(区)，每个partition在存储层面是append log文件。任何发布到此partition的消息都会被直接追加到log文件的尾部，每条消息在文件中的位置称为offset（偏移量），offset为一个long型数字，它是唯一标记一条消息。它唯一的标记一条消息。kafka并没有提供其他额外的索引机制来存储offset，因为在kafka中几乎不允许对消息进行“随机读写”。

![img](https://www.zixi.org/static/uploads/2020/9/20200930104451.png)

Kafka和JMS（Java Message Service）实现(activeMQ)不同的是:即使消息被消费，消息仍然不会被立即删除。日志文件将会根据broker中的配置要求，保留一定的时间之后删除；比如log文件保留2天，那么两天后，文件会被清除，无论其中的消息是否被消费。kafka通过这种简单的手段，来释放磁盘空间，以及减少消息消费之后对文件内容改动的磁盘IO开支。

对于consumer而言，它需要保存消费消息的offset，对于offset的保存和使用，有consumer来控制；当consumer正常消费消息时，offset将会"线性"的向前驱动，即消息将依次顺序被消费。事实上consumer可以使用任意顺序消费消息，它只需要将offset重置为任意值。(offset将会保存在zookeeper中，参见下文)

kafka集群几乎不需要维护任何consumer和producer状态信息，这些信息有zookeeper保存；因此producer和consumer的客户端实现非常轻量级，它们可以随意离开，而不会对集群造成额外的影响。

partitions的设计目的有多个。最根本原因是kafka基于文件存储。通过分区，可以将日志内容分散到多个server上，来避免文件尺寸达到单机磁盘的上限，每个partiton都会被当前server(kafka实例)保存；可以将一个topic切分多任意多个partitions，来消息保存/消费的效率。此外越多的partitions意味着可以容纳更多的consumer，有效提升并发消费的能力。(具体原理参见下文)。

### Distribution

一个Topic的多个partitions，被分布在kafka集群中的多个server上；每个server(kafka实例)负责partitions中消息的读写操作；此外kafka还可以配置partitions需要备份的个数(replicas)，每个partition将会被备份到多台机器上，以提高可用性。

基于replicated方案，那么就意味着需要对多个备份进行调度；每个partition都有一个server为"leader"；leader负责所有的读写操作，如果leader失效，那么将会有其他follower来接管(成为新的leader)；follower只是单调的和leader跟进，同步消息即可。由此可见作为leader的server承载了全部的请求压力，因此从集群的整体考虑，有多少个partitions就意味着有多少个"leader"，kafka会将"leader"均衡的分散在每个实例上，来确保整体的性能稳定。

**Producers**
Producer将消息发布到指定的Topic中，同时Producer也能决定将此消息归属于哪个partition；比如基于"round-robin"方式或者通过其他的一些算法等。

**Consumers**
本质上kafka只支持Topic。每个consumer属于一个consumer group；反过来说，每个group中可以有多个consumer。发送到Topic的消息，只会被订阅此Topic的每个group中的一个consumer消费。

如果所有的consumer都具有相同的group，这种情况和queue模式很像；消息将会在consumers之间负载均衡。

如果所有的consumer都具有不同的group，那这就是"发布-订阅"；消息将会广播给所有的消费者。

在kafka中，一个partition中的消息只会被group中的一个consumer消费；每个group中consumer消息消费互相独立；我们可以认为一个group是一个"订阅"者，一个Topic中的每个partions，只会被一个"订阅者"中的一个consumer消费，不过一个consumer可以消费多个partitions中的消息。kafka只能保证一个partition中的消息被某个consumer消费时，消息是顺序的。事实上，从Topic角度来说，消息仍不是有序的。

Kafka的设计原理决定，对于一个topic，同一个group中不能有多于partitions个数的consumer同时消费，否则将意味着某些consumer将无法得到消息。

**Guarantees**

- 发送到partitions中的消息将会按照它接收的顺序追加到日志中
- 对于消费者而言，它们消费消息的顺序和日志中消息顺序一致。
- 如果Topic的"replicationfactor"为N，那么允许N-1个kafka实例失效。

### 使用场景

- Messaging
  对于一些常规的消息系统，kafka是个不错的选择；partitons/replication和容错，可以使kafka具有良好的扩展性和性能优势。不过到目前为止，我们应该很清楚认识到，**kafka并没有提供JMS中的“事务性”、“消息传输担保(消息确认机制)”、“消息分组”等企业级特性；kafka只能使用作为“常规”的消息系统，在一定程度上，尚未确保消息的发送与接收绝对可靠(比如：消息重发，消息发送丢失等)**；
- Websit activity tracking
  kafka可以作为“网站活性跟踪”的最佳工具；可以将网页/用户操作等信息发送到kafka中。并实时监控，或者离线统计分析等；
- Log Aggregation
  kafka的特性决定它非常适合作为“日志收集中心”；application可以将操作日志“批量”、“异步”的发送到kafka集群中，而不是保存在本地或者DB中；kafka可以批量提交消息/压缩消息等，这对producer端而言，几乎感觉不到性能的开支。此时consumer端可以使hadoop等其他系统化的存储和分析系统；

### 消息可靠性保障

Kafka就比较适合高吞吐量并且允许少量数据丢失的场景，如果非要保证“消息可靠传输”，可以使用JMS。

Kafka Producer 消息发送有两种方式(配置参数 producer.type)：

- producer.type=sync(默认值): 后台线程中消息发送是同步方式，对应的类为 kafka.producer.SyncProducer；
- producer.type=async: 后台线程中消息发送是异步方式，对应的类为 kafka.producer.AyncProducer；优点是可批量发送消息(消息个数达到 batch.num.messages=200 或时间达到时发送)、吞吐量佳，缺点是发送不及时可能导致丢失；

对于同步方式(producer.type=sync)？Kafka Producer 消息发送有三种确认方式(配置参数 acks)：

- acks=0: producer 不等待 Leader 确认，只管发出即可；最可能丢失消息，适用于高吞吐可丢失的业务；
- acks=1(默认值): producer 等待 Leader 写入本地日志后就确认；之后 Leader 向 Followers 同步时，如果 Leader 宕机会导致消息没同步而丢失，producer 却依旧认为成功；
- acks=all/-1: producer 等待 Leader 写入本地日志、而且 Leader 向 Followers 同步完成后才会确认，最可靠；

### 设计原理

kafka的设计初衷是希望作为一个统一的信息收集平台，能够实时的收集反馈信息，并需要能够支撑较大的数据量，且具备良好的容错能力。

**持久性**
kafka使用文件存储消息，这就直接决定kafka在性能上严重依赖文件系统的本身特性。且无论任何OS下，对文件系统本身的优化几乎没有可能。文件缓存/直接内存映射等是常用的手段。因为kafka是对日志文件进行append操作，因此磁盘检索的开支是较小的；同时为了减少磁盘写入的次数，broker会将消息暂时buffer起来，当消息的个数(或尺寸)达到一定阀值时，再flush到磁盘，这样减少了磁盘IO调用的次数。

**性能**
需要考虑的影响性能点很多，除磁盘IO之外，我们还需要考虑网络IO，这直接关系到kafka的吞吐量问题。kafka并没有提供太多高超的技巧；对于producer端，可以将消息buffer起来，当消息的条数达到一定阀值时，批量发送给broker；对于consumer端也是一样，批量fetch多条消息。不过消息量的大小可以通过配置文件来指定。对于kafka broker端，似乎有个sendfile系统调用可以潜在的提升网络IO的性能:将文件的数据映射到系统内存中，socket直接读取相应的内存区域即可，而无需进程再次copy和交换。 其实对于producer/consumer/broker三者而言，CPU的开支应该都不大，因此启用消息压缩机制是一个良好的策略；压缩需要消耗少量的CPU资源，不过对于kafka而言，网络IO更应该需要考虑。可以将任何在网络上传输的消息都经过压缩。kafka支持gzip/snappy等多种压缩方式。

**生产者**
负载均衡: producer将会和Topic下所有partition leader保持socket连接；消息由producer直接通过socket发送到broker，中间不会经过任何“路由层“。事实上，消息被路由到哪个partition上，有producer客户端决定。比如可以采用“random““key-hash““轮询“等，如果一个topic中有多个partitions，那么在producer端实现“消息均衡分发“是必要的。

其中partition leader的位置(host:port)注册在zookeeper中，producer作为zookeeper client，已经注册了watch用来监听partition leader的变更事件。
异步发送：将多条消息暂且在客户端buffer起来，并将他们批量的发送到broker，小数据IO太多，会拖慢整体的网络延迟，批量延迟发送事实上提升了网络效率。不过这也有一定的隐患，比如说当producer失效时，那些尚未发送的消息将会丢失。

**消费者**
consumer端向broker发送“fetch”请求，并告知其获取消息的offset；此后consumer将会获得一定条数的消息；consumer端也可以重置offset来重新消费消息。

在JMS实现中，Topic模型基于push方式，即broker将消息推送给consumer端。不过在kafka中，采用了pull方式，即consumer在和broker建立连接之后，主动去pull(或者说fetch)消息；这中模式有些优点，首先consumer端可以根据自己的消费能力适时的去fetch消息并处理，且可以控制消息消费的进度(offset)；此外，消费者可以良好的控制消息消费的数量，batch fetch。

其他JMS实现，消息消费的位置是有prodiver保留，以便避免重复发送消息或者将没有消费成功的消息重发等，同时还要控制消息的状态。这就要求JMS broker需要太多额外的工作。在kafka中，partition中的消息只有一个consumer在消费，且不存在消息状态的控制，也没有复杂的消息确认机制，可见kafka broker端是相当轻量级的。当消息被consumer接收之后，consumer可以在本地保存最后消息的offset，并间歇性的向zookeeper注册offset。由此可见，consumer客户端也很轻量级。

![img](https://www.zixi.org/static/uploads/2020/9/20200930104451.png)

### 消息传送机制

对于JMS实现，消息传输担保非常直接:有且只有一次(exactly once)。
在kafka中稍有不同:

![img](https://www.zixi.org/static/uploads/2020/9/20200930104454.png)

- at most once: 最多一次，这个和JMS中"非持久化"消息类似。发送一次，无论成败，将不会重发。
- at least once: 消息至少发送一次，如果消息未能接受成功，可能会重发，直到接收成功。
- exactly once: 消息只会发送一次。

at most once: 消费者fetch消息，然后保存offset，然后处理消息；当client保存offset之后，但是在消息处理过程中出现了异常，导致部分消息未能继续处理。那么此后"未处理"的消息将不能被fetch到，这就是"at most once"。

at least once: 消费者fetch消息，然后处理消息，然后保存offset。如果消息处理成功之后，但是在保存offset阶段zookeeper异常导致保存操作未能执行成功，这就导致接下来再次fetch时可能获得上次已经处理过的消息，这就是"at least once"，原因offset没有及时的提交给zookeeper，zookeeper恢复正常还是之前offset状态。

exactly once: kafka中并没有严格的去实现(基于2阶段提交，事务)，我们认为这种策略在kafka中是没有必要的。

通常情况下“at-least-once”是我们首选。(相比at most once而言，重复接收数据总比丢失数据要好)。

> 如何保证消息不被重复消费：使用幂等性消费者

### 集群

![img](https://www.zixi.org/static/uploads/2020/9/20200930104456.png)

kafka高可用由多个broker组成，每个broker是一个节点；

创建一个topic，这个topic会划分为多个partition，每个partition存在于不同的broker上，每个partition就放一部分数据。

kafka是一个分布式消息队列，就是说一个topic的数据，是分散放在不同的机器上，每个机器就放一部分数据。

在0.8版本以前，是没有HA机制的，就是任何一个broker宕机了，那个broker上的partition就废了，没法写也没法读，没有什么高可用性可言。

0.8版本以后，才提供了HA机制，也就是就是replica副本机制。每个partition的数据都会同步到其他的机器上，形成自己的多个replica副本。然后所有replica会选举一个leader出来，那么生产和消费都跟这个leader打交道，然后其他replica就是follower。

写的时候，leader会负责把数据同步到所有follower上去，读的时候就直接读leader上数据即可。

kafka会均匀的将一个partition的所有replica分布在不同的机器上，从而提高容错性。

如果某个broker宕机了也没事，它上面的partition在其他机器上都有副本的，如果这上面有某个partition的leader，那么此时会重新选举一个新的leader出来，大家继续读写那个新的leader即可。这就有所谓的高可用性了。

写数据的时候，生产者就写leader，然后leader将数据落地写本地磁盘，接着其他follower自己主动从leader来pull数据。一旦所有follower同步好数据了，就会发送ack给leader，leader收到所有follower的ack之后，就会返回写成功的消息给生产者。

### 复制备份

- kafka将每个partition数据复制到多个server上，任何一个partition有一个leader和多个follower(可以没有)；
- 备份的个数可以通过broker配置文件来设定。leader处理所有的read-write请求，follower需要和leader保持同步。Follower和consumer一样，消费消息并保存在本地日志中；
- leader负责跟踪所有的follower状态，如果follower"落后"太多或者失效，leader将会把它从replicas同步列表中删除；
- 当所有的follower都将一条消息保存成功，此消息才被认为是"committed"，那么此时consumer才能消费它；
- 即使只有一个replicas实例存活，仍然可以保证消息的正常发送和接收，只要zookeeper集群存活即可(不同于其他分布式存储，比如hbase需要"多数派"存活才行)；
- 当leader失效时，需在followers中选取出新的leader，可能此时follower落后于leader，因此需要选择一个"up-to-date"的follower；
- 选择follower时需要兼顾一个问题，就是新leaderserver上所已经承载的partition leader的个数，如果一个server上有过多的partition leader，意味着此server将承受着更多的IO压力。在选举新leader，需要考虑到"负载均衡"；

## 如何处理消息丢失

消息丢失会出现在三个环节，分别是生产者、mq中间件、消费者：

**RabbitMQ**

- 生产者端的控制
  使用事务，出错后重试。
  或者使用异步回调（confirm），由消费者回调生产者方法通知生产者消息是否正常处理。
- 中间件的控制
  持久化消息到磁盘（不能保证绝对不丢）；
- 消费者端控制
  关闭Auto ACK，手动ACK。

**Kafka**
大体和RabbitMQ相同。

- 设置acks=all，此时kafka会确保消息同步到所有follower才会ACK。
  当broker leader宕机了，并且消息没有同步到follower，此时选举出了新的leader并没有同步到刚才的消息，那么这条消息便丢了，解决的方法是，。

## 如何保证消息的顺序性

**Rabbitmq**
需要保证顺序的消息投递到同一个queue中，这个queue只能有一个consumer，如果需要提升性能，可以用内存队列做排队，然后分发给底层不同的worker来处理。

**Kafka**
写入一个partition中的数据一定是有序的。生产者在写的时候 ，可以指定一个key，比如指定订单id作为key，这个订单相关数据一定会被分发到一个partition中去。消费者从partition中取出数据的时候也一定是有序的，把每个数据放入对应的一个内存队列，一个partition中有几条相关数据就用几个内存队列，消费者开启多个线程，每个线程处理一个内存队列。



## 常用的几款消息队列的对比

### 前言

消息队列的作用：

1、应用耦合：多应用间通过消息队列对同一消息进行处理，避免调用接口失败导致整个过程失败；

2、异步处理：多应用对消息队列中同一消息进行处理，应用间并发处理消息，相比串行处理，减少处理时间；

3、限流削峰：广泛应用于秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况；

4、消息驱动的系统：系统分为消息队列、消息生产者、消息消费者，生产者负责产生消息，消费者(可能有多个)负责对消息进行处理；

首先选择消息队列要满足以下几个条件：

1、开源

2、流行

3、兼容性强

消息队列需要：

1、消息的可靠传递：确保不丢消息；

2、Cluster：支持集群，确保不会因为某个节点宕机导致服务不可用，当然也不能丢消息；

3、性能：具备足够好的性能，能满足绝大多数场景的性能要求。

### RabbitMQ

RabbitMQ 2007年发布，是一个在 AMQP (高级消息队列协议)基础上完成的，可复用的企业消息系统，是当前最主流的消息中间件之一。

#### 优点

1、RabbitMQ 的特点 Messaging that just works，“开箱即用的消息队列”。 RabbitMQ 是一个相对轻量的消息队列，非常容易部署和使用；

2、多种协议的支持：支持多种消息队列协议，算的上是最流行的消息队列之一；

3、灵活的路由配置，和其他消息队列不同的是，它在生产者 （Producer）和队列（Queue）之间增加了一个Exchange模块，你可以理解为交换机。这个Exchange模块的作用和交换机也非常相似，根据配置的路由规则将生产者发出的消息分发到不同的队 列中。路由的规则也非常灵活，甚至你可以自己来实现路由规则。

4、健壮、稳定、易用、跨平台、支持多种语言、文档齐全，RabbitMQ的客户端支持的编程语言大概是所有消息队列中最多的；

5、管理界面较丰富，在互联网公司也有较大规模的应用；

6、社区比较活跃。

#### 缺点

1、RabbitMQ 对消息堆积的处理不好，在它的设计理念里面，消息队列是一个管道，大量的消息积压是一种不正常的情况，应当尽量去避免。当大量消息积压的时候，会导致RabbitMQ的性能急剧下降；

2、性能上有瓶颈，它大概每秒钟可以处理几万到十几万条消息，这个对于大多数场景足够使用了，如果对需求对性能要求非常高，那么就不太合适了。

3、RabbitMQ 使用 Erlang。开发，Erlang 的学习成本还是很高的，如果后期进行二次开发，就不太容易了。

### RocketMQ

RocketMQ出自阿里公司的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进，消息可靠性上比 Kafka 更好。经历过多次双十一的考验，性能和稳定性还是值得信赖的，RocketMQ在阿里集团被广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog分发等场景。

#### 优点

1、单机吞吐量：十万级；

2、可用性：非常高，分布式架构；

3、消息可靠性：经过参数优化配置，消息可以做到0丢失，RocketMQ 的所有消息都是持久化的，先写入系统 PAGECACHE，然后刷盘，可以保证内存与磁盘都有一份数据；

4、功能支持：MQ功能较为完善，还是分布式的，扩展性好；

5、支持10亿级别的消息堆积，不会因为堆积导致性能下降；

6、源码是java，我们可以自己阅读源码，定制自己公司的MQ，可以掌控。

#### 缺点

1、支持的客户端语言不多，目前是 java 及 c++，其中 c++ 不成熟；

2、社区活跃度一般，作为国产的消息队列，相比国外的比较流行的同类产品，在国际上还没有那么流行，与周边生态系统的集成和兼容程度要略逊一筹；

3、没有在 mq 核心中去实现 JMS 等接口，有些系统要迁移需要修改大量代码。

### Kafka

Apache Kafka是一个分布式消息发布订阅系统。它最初由LinkedIn公司基于独特的设计实现为一个分布式的提交日志系统( a distributed commit log)，之后成为Apache项目的一部分。

这是一款为大数据而生的消息中间件，在数据采集、传输、存储的过程中发挥着举足轻重的作用。

#### 优点

1、性能卓越，单机写入TPS约在百万条/秒，最大的优点，就是吞吐量高；

2、性能卓越，单机写入TPS约在百万条/秒，消息大小10个字节；

3、可用性：非常高，kafka是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用；

4、消费者采用Pull方式获取消息, 消息有序, 通过控制能够保证所有消息被消费且仅被消费一次;

5、有优秀的第三方Kafka Web管理界面Kafka-Manager；

6、在日志领域比较成熟，被多家公司和多个开源项目使用；

7、功能支持：功能较为简单，主要支持简单的MQ功能，在大数据领域的实时计算以及日志采集被大规模使用

#### 缺点

由于“攒一波再处理”导致延迟比较高

### Pulsar

Pulsar 是一个用于服务器到服务器的消息系统，具有多租户、高性能等优势。 Pulsar 最初由 Yahoo 开发，目前由 Apache 软件基金会管理。

#### 优点

1、更多功能：Pulsar Function、多租户、Schema registry、n 层存储、多种消费模式和持久性模式等；

2、Pulsar 的单个实例原生支持多个集群，可跨机房在集群间无缝地完成消息复制；

3、极低的发布延迟和端到端延迟；

4、可无缝扩展到超过一百万个 topic；

5、简单的客户端 API，支持 Java、Go、Python 和 C++。

6、Pulsar 的单个实例原生支持多个集群，可跨机房在集群间无缝地完成消息复制。

#### 缺点

正处于成长期，流行度和成熟度相对没有那么高

### 如何选择合适的消息队列

如果对于消息队列的功能和性能要求不是很高，那么RabbitMQ就够了，开箱即用。

如果系统使用消息队列主要场景是处理在线业务，比如在交易系统中用消息队列传递订单，RocketMQ 的低延迟和金融级的稳定性就可以满足。

要处理海量的消息，像收集日志、监控信息或是前端的埋点这类数据，或是你的应用场景大量使用 了大数据、流计算相关的开源产品，那 Kafka 就是最合适的了。

如果数据量很大，同时不希望有 Kafka 的高延迟，刚好业务场景是金融场景。RocketMQ 对 Topic 运营不太友好，特别是不支持按 Topic 删除失效消息，以及不具备宕机 Failover 能力。那么 Pulsar 可能就是你的一个选择了。

## 常用消息队列（ActiveMQ、RabbitMQ、RocketMQ、Kafka）比较

| 特性MQ           | ActiveMQ   | RabbitMQ   | RocketMQ         | Kafka            |
| ---------------- | ---------- | ---------- | ---------------- | ---------------- |
| 生产者消费者模式 | 支持       | 支持       | 支持             | 支持             |
| 发布订阅模式     | 支持       | 支持       | 支持             | 支持             |
| 请求回应模式     | 支持       | 支持       | 不支持           | 不支持           |
| Api完备性        | 高         | 高         | 高               | 高               |
| 多语言支持       | 支持       | 支持       | java             | 支持             |
| 单机吞吐量       | 万级       | 万级       | 万级             | 十万级           |
| 消息延迟         | 无         | 微秒级     | 毫秒级           | 毫秒级           |
| 可用性           | 高（主从） | 高（主从） | 非常高（分布式） | 非常高（分布式） |
| 消息丢失         | 低         | 低         | 理论上不会丢失   | 理论上不会丢失   |
| 文档的完备性     | 高         | 高         | 高               | 高               |
| 提供快速入门     | 有         | 有         | 有               | 有               |
| 社区活跃度       | 高         | 高         | 有               | 高               |
| 商业支持         | 无         | 无         | 商业云           | 商业云           |