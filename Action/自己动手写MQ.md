https://zhuanlan.zhihu.com/p/63632647

消息队列设计精要
大致整理如下：最简单的消息队列可以做成一个消息转发器，把一次RPC做成两次RPC。
构建一个整体的数据流，
例如producer发送给broker，broker发送给consumer，consumer回复消费确认，broker删除/备份消息等。利用RPC将数据流串起来（可以使用开源RPC框架，比如Dubbo等）负载均衡、服务发现、通信协议：Thrift，Dubbo序列化协议存储子系统：通过存储承载消息堆积，然后在合适的时机投递消息持久化、非持久化从速度来看，文件系统 &gt; 分布式KV（持久化）&gt; 分布式文件系统 &gt; 数据库，而可靠性却截然相反。需要根据具体业务选择消费关系的保存：单播：点对点多播：一点对多点通用设计：支持组间广播，不同的组注册不同的订阅。组内的不同机器，如果注册一个相同的ID，则单播；如果注册不同的ID（如IP地址+端口），则广播。广播关系的维护，一般由于消息队列本身都是集群，所以都维护在公共存储上，如config server、zookeeper等。消费确认（任务执行完整性）：把消息的送达和消息的处理分开，这样才真正的实现了消息队列的本质-解耦。对于没有特殊逻辑的消息，默认Auto Ack也是可以的但一定要允许消费方主动进行消费确认ack，并与broker约定下次投递时间高级特性：可靠投递（最终一致性）：高可用：超时重发与消息重复、消息队列幂等设计（broker多机器共享一个DB或者一个分布式文件/kv系统）、定时任务补偿不是所有的系统都要求最终一致性或者可靠投递。任何基础组件要服务于业务场景。消息可能会重复，并且在异常情况下，要接受消息的延迟。每当要发生不可靠的事情（RPC等）之前，先将消息落地，然后发送。当消息发送失败或者不知道成功失败（比如超时）时，消息状态是待发送。对于各种不确定（超时、down机、消息没有送达、送达后数据没落地、数据落地了回复没收到），其实对于发送方来说，都是一件事情，就是消息没有送达。定时任务不停轮询所有待发送消息，最终一定可以送达。重复消息：如何鉴别消息重复，并幂等的处理重复消息：版本号状态机一个消息队列server如何尽量减少重复消息的投递：鉴别消息重复：broker记录MessageId，直到投递成功后清除，重复的ID到来不做处理。减少重复投递：对于server投递到consumer的消息，由于不确定对端是在处理过程中还是消息发送丢失的情况下，有必要记录下投递的IP地址。决定重发之前询问这个IP，消息处理成功了吗？如果询问无果，再重发。顺序消息：顺序消息要求：允许消息丢失；从发送方到服务方到接受者都是单点单线程。pull模式实现比较容易（见下）一个主流消息队列的设计范式里，应该是不丢消息的前提下，尽量减少重复消息，不保证消息的投递顺序。事务特性：在与本地业务的同一个事务中，本地消息落地（落库、需要业务方提供数据库）消息只要投递到服务端确认后本地才做删除定时任务扫描本地消息库表进行补偿发送 性能优化异步：对于客户端来说，同步与异步主要是拿到一个Result，还是Future(Listenable)的区别。实现方式可以是线程池，NIO或者其他事件机制。服务端异步需要RPC协议支持。参考servlet 3.0规范，服务端可以吐一个future给客户端，并且在future done的时候通知客户端。实践：客户端必须等待服务端消息成功落地，才算是消息发送成功。>我们不希望消息的发送阻塞客户端的主流程，所以可以先使用线程池提交一个发送请求，主流程继续往下走。服务端是纯异步。客户端的线程池wait在服务端吐回的future上，直到服务端处理完毕，才解除阻塞继续进行。批量：消费者合适消费消息：通过网络请求小包合并成大包提高性能消息消费是push还是pull：push：push模型最大的致命伤是慢消费。如果消费者的速度比发送者的速度慢很多，势必造成消息在broker的堆积。最致命的是broker给consumer推送一堆consumer无法处理的消息，consumer不是reject就是error，然后来回踢皮球所以对于建立索引等慢消费，消息量有限且到来的速度不均匀的情况，pull模式比较合适。pull：pull模式存在消息延迟与忙等问题pull模式如果想做到全局顺序消息，就相对容易很多：producer对应partition，并且单线程。consumer对应partition，消费确认（或批量确认），继续消费即可。所以对于日志push送这种最好全局有序，但允许出现小误差的场景，pull模式非常合适。如果你不想看到通篇乱套的日志~~Anyway，需要顺序消息的场景还是比较有限的而且成本太高，请慎重考虑。



使用 JAVA 语言自己动手来写一个MQ (类似ActiveMQ,RabbitMQ)

主要角色

首先我们必须需要搞明白 MQ (消息队列) 中的三个基本角色 Producer

Broker

Consumer

整体架构如下所示

自定义协议

首先从上一篇中介绍了协议的相关信息,具体厂商的 MQ(消息队列) 需要遵循某种协议或者自定义协议 , 消息的 生产者和消费者需要遵循其协议(约定)才能后成功地生产消息和生产消息 ,所以在这里我们自定义一个协议如下．

消息处理中心 : 如果接收到的信息包含"SEND"字符串,即视为生产者发送的消息,消息处理中心需要将此信息存储等待消费者消费

消息处理中心 : 如果接受到的信息为CONSUME，既视为消费者发送消费请求，需要将存储的消息队列头部的信息转发给消费者，然后将此消息从队列中移除

消息处理中心 : 如果消息处理中心存储的消息满3条仍然没有消费者进行消费,则不再接受生产者的生产请求

消息生产者：需要遵循协议将生产的消息头部增加＂SEND:＂ 表示生产消息

消息消费者：需要遵循协议向消息处理中心发送＂CONSUME＂字符串表示消费消息

流程顺序

项目构建流程

下面将整个ＭＱ的构建流程过一遍 新建一个 Broker 类，内部维护一个 ArrayBlockingQueue 队列，提供生产消息和消费消息的方法， 仅仅具备存储服务功能

新建一个 BrokerServer 类,将 Broker 发布为服务到本地9999端口，监听本地9999端口的 Socket 链接，在接受的信息中进行我们的协议校验, 这里 仅仅具备接受消息,校验协议,转发消息功能;

新建一个 MqClient 类,此类提供与本地端口9999的Socket链接 , 仅仅具备生产消息和消费消息的方法

测试：新建两个 MyClient 类对象，分别执行其生产方法和消费方法

具体使用流程 生产消息：客户端执行生产消息方法，传入需要生产的信息，该信息需要遵循我们自定义的协议，消息处理中心服务在接受到消息会根据自定义的协议校验该消息是否合法，如果合法如果合法就会将该消息存储到Broker内部维护的 ArrayBlockingQueue 队列中．如果 ArrayBlockingQueue 队列没有达到我们协议中的最大长度将将消息添加到队列中，否则输出生产消息失败．

消息消息：客户端执行消费消息方法， Broker服务 会校验请求的信息的信息是否等于 CONSUME ，如果验证成功则从Broker内部维护的 ArrayBlockingQueue 队列的 Poll 出一个消息返回给客户端

代码演示

消息处理中心 Broker /**

* 消息处理中心

*/

public class Broker {undefined

// 队列存储消息的最大数量

private final static int MAX_SIZE = 3;

// 保存消息数据的容器

private static ArrayBlockingQueue messageQueue = new ArrayBlockingQueue(MAX_SIZE);

// 生产消息

public static void produce(String msg) {undefined

if (messageQueue.offer(msg)) {undefined

System.out.println("成功向消息处理中心投递消息：" + msg + "，当前暂存的消息数量是：" + messageQueue.size());

} else {undefined

System.out.println("消息处理中心内暂存的消息达到最大负荷，不能继续放入消息！");

}

System.out.println("=======================");

}

// 消费消息

public static String consume() {undefined

String msg = messageQueue.poll();

if (msg != null) {undefined

// 消费条件满足情况，从消息容器中取出一条消息

System.out.println("已经消费消息：" + msg + "，当前暂存的消息数量是：" + messageQueue.size());

} else {undefined

System.out.println("消息处理中心内没有消息可供消费！");

}

System.out.println("=======================");

return msg;

}

}

消息处理中心服务 BrokerServer /**

* 用于启动消息处理中心

*/

public class BrokerServer implements Runnable {undefined

public static int SERVICE_PORT = 9999;

private final Socket socket;

public BrokerServer(Socket socket) {undefined

this.socket = socket;

}

@Override

public void run() {undefined

try (

BufferedReader in = new BufferedReader(new InputStreamReader(

socket.getInputStream()));

PrintWriter out = new PrintWriter(socket.getOutputStream())

)

{undefined

while (true) {undefined

String str = in.readLine();

if (str == null) {undefined

continue;

}

System.out.println("接收到原始数据：" + str);

if (str.equals("CONSUME")) { //CONSUME 表示要消费一条消息

//从消息队列中消费一条消息

String message = Broker.consume();

out.println(message);

out.flush();

} else if (str.contains("SEND:")){undefined

//接受到的请求包含SEND:字符串 表示生产消息放到消息队列中

Broker.produce(str);

}else {undefined

System.out.println("原始数据:"+str+"没有遵循协议,不提供相关服务");

}

}

} catch (Exception e) {undefined

e.printStackTrace();

}

}

public static void main(String[] args) throws Exception {undefined

ServerSocket server = new ServerSocket(SERVICE_PORT);

while (true) {undefined

BrokerServer brokerServer = new BrokerServer(server.accept());

new Thread(brokerServer).start();

}

}

}

客户端 MqClient /**

* 访问消息队列的客户端

*/

public class MqClient {undefined

//生产消息

public static void produce(String message) throws Exception {undefined

//本地的的BrokerServer.SERVICE_PORT 创建SOCKET

Socket socket = new Socket(InetAddress.getLocalHost(), BrokerServer.SERVICE_PORT);

try (

PrintWriter out = new PrintWriter(socket.getOutputStream())

) {undefined

out.println(message);

out.flush();

}

}

//消费消息

public static String consume() throws Exception {undefined

Socket socket = new Socket(InetAddress.getLocalHost(), BrokerServer.SERVICE_PORT);

try (

BufferedReader in = new BufferedReader(new InputStreamReader(

socket.getInputStream()));

PrintWriter out = new PrintWriter(socket.getOutputStream())

) {undefined

//先向消息队列发送命令

out.println("CONSUME");

out.flush();

//再从消息队列获取一条消息

String message = in.readLine();

return message;

}

}

}

测试MQ public class ProduceClient {undefined

public static void main(String[] args) throws Exception {undefined

MqClient client = new MqClient();

client.produce("SEND:Hello World");

}

}

public class ConsumeClient {undefined

public static void main(String[] args) throws Exception {undefined

MqClient client = new MqClient();

String message = client.consume();

System.out.println("获取的消息为：" + message);

}

}

我们多执行几次客户端的生产方法和消费方法就可以看到一个完整的MQ的通讯过程,下面是我执行了几次的一些日志 接收到原始数据：SEND:Hello World

成功向消息处理中心投递消息：SEND:Hello World，当前暂存的消息数量是：1

=======================

接收到原始数据：SEND:Hello World

成功向消息处理中心投递消息：SEND:Hello World，当前暂存的消息数量是：2

=======================

接收到原始数据：SEND:Hello World

成功向消息处理中心投递消息：SEND:Hello World，当前暂存的消息数量是：3

=======================

接收到原始数据：SEND:Hello World

消息处理中心内暂存的消息达到最大负荷，不能继续放入消息！

=======================

接收到原始数据：Hello World

原始数据:Hello World没有遵循协议,不提供相关服务

接收到原始数据：CONSUME

已经消费消息：SEND:Hello World，当前暂存的消息数量是：2

=======================

接收到原始数据：CONSUME

已经消费消息：SEND:Hello World，当前暂存的消息数量是：1

=======================

接收到原始数据：CONSUME

已经消费消息：SEND:Hello World，当前暂存的消息数量是：0

=======================

接收到原始数据：CONSUME

消息处理中心内没有消息可供消费！

=======================

小结

本章示例代码主要源自分布式消息中间件实践一书 , 这里我们自己使用Java语言写了一个MQ消息队列 , 通过这个消息队列我们对MQ中的几个角色 "生产者,消费者,消费处理中心,协议" 有了更深的理解 ; 那么下一章节我们就来一块学习具体厂商的MQ RabbitMQ
————————————————
版权声明：本文为CSDN博主「kyle shi」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_30151653/article/details/114502484

# 仿照Kafka，从零开始自实现 MQ

仿照Kafka，从零开始自实现 MQ，实现了 Kafka 中 80% 的基础功能。学习 Kafka 的话如果只是看文章和源码，可能不久就会忘了，还是自己实现一个「精简版」的 Kafka 吧，

## 实现功能概览

1、基于内存Queue实现生产和消费API

- [X] 1） 创建内存Queue， 作为底层消息存储
- [X] 2） 定义Topic， 支持多个Topic
- [X] 3） 定义Producer， 支持Send消息
- [X] 4） 定义Consumer， 支持Poll消息

2、设计自定义Queue，实现消息确认和消费offset

- [X] 1） 自定义内存Message数组模拟Queue。
- [X] 2） 使用指针记录当前消息写入位置。
- [X] 3） 对于每个命名消费者， 用指针记录消费位置

3、拆分broker和client(包括producer和consumer)

- [X] 1） 将Queue保存到web server端
- [X] 2） 设计消息读写API接口， 确认接口， 提交offset接口
- [X] 3） producer和consumer通过httpclient访问Queue
- [X] 4） 实现消息确认， offset提交
- [X] 5） 实现consumer从offset增量拉取

## 项目目录

[bitkylin-mq](https://link.zhihu.com/?target=https%3A//github.com/bitkylin/featureLab/tree/master/bitkylin-mq)

## 项目设计及项目能力

### Server

### 一、Topic

1. 维护ArrayList用于模拟持久化消息「原因：消息需要随机访问」
2. 设定消息队列容量，达到容量时无法再生产消息
3. 当前消息的最大索引

### 二、ConsumerGroup

1. 消费者组由消费者组名和topic名共同决定，即不同topic的消费者组相互独立，不会相互影响
2. 需根据topic创建消费者组，即消费者组必须关联topic
3. 消费者组创建后，默认从头完整消费关联topic的所有消息
4. 同一个消费者组内，各个消费者总共消费一次「最少消费一次」所关联topic的所有消息

### 三、broker

1. 一个broker关联一个ConsumerGroup列表和一个Topic列表
2. 通过broker暴露的接口，可以展示关联ConsumerGroup列表和Topic列表的概览信息
3. 通过broker暴露的接口，可以向一个topic中生产消息
4. 通过broker暴露的接口，可以根据消费者组名和topic名消费消息

注：本次仅实现单个broker，broker后实现了topic和consumerGroup「消费者组」，细节结构图如下：



![img](https://pic2.zhimg.com/80/v2-70e4754717f2e8e32ba37d74043d92dd_1440w.jpg)

### client

1. 客户端通过topic名生产消息
2. 客户端根据消费者组名和topic名消费消息
3. 客户端消费消息时，可以同时获得消费者组的offset「偏移量」
4. 客户端消费消息成功后，需手动更新消费者组的offset。若不更新，客户端默认无法消费后面的消息。
5. 客户端消费消息失败时，不应更新消费者组的offset。此时客户端可以重复消费当条消息。
6. 多个客户端可以使用同一个消费者组消费同一个topic；可以使用不同的消费者组消费同一个topic；可以使用不同的消费者组消费不同的topic

客户端工作示意图如下：

![img](https://pic3.zhimg.com/80/v2-db94ce3a5152aff08aa9923ad926d14a_1440w.jpg)

## 项目结构

本项目共提供四个module：

```text
bitkylin-mq-server
bitkylin-mq-api
bitkylin-mq-client-producer
bitkylin-mq-client-consumer
```

各module的介绍如下：

### 1. bitkylin-mq-server

提供MQ服务端，提供broker以及其关联的ConsumerGroup和Topic等，主要实现如下功能：

- 展示MQ概览信息，包括topic和ConsumerGroup的详细信息
- 创建消费者组，创建消费者组后，即可使用该消费者组消费消息
- 生产消息，将消息发送至指定topic
- 基于指定消费者组消费消息，消费消息但不更新关联消费者组的offset
- 基于指定消费者组消费消息，消费消息且自动更新关联消费者组的offset
- 手动更新指定消费者组的偏移量

### 2. bitkylin-mq-api

提供供客户端使用的api，通过feignClient形式提供，客户端可直接使用，执行RPC，当前实现如下功能：

- 发送消息至指定topic
- 订阅指定topic的消息。自动创建消费者组，使用观察者模式轮询消息并消费。

### 3. bitkylin-mq-client-producer

消息生产客户端，通过feign-api生产消息，当前实现如下演示功能：
随机向topic名为「topic-1」和「topic-2」的topic中发送消息，每隔3秒发送一次消息。

### 4. bitkylin-mq-client-consumer

消息消费客户端，通过feign-api消费消息，当前实现如下演示功能：

- 创建消费者组「spring-group-1」订阅「topic-1」，并打印订阅的消息。
- 创建消费者组「spring-group-2」订阅「topic-2」，并打印订阅的消息。

## 代码演示

1. 运行module「bitkylin-mq-server」，启动MQ的broker，启动消息服务。
2. 运行module「bitkylin-mq-client-consumer」和「bitkylin-mq-client-producer」，开启消息订阅演示任务和消息发送演示任务。
3. 此时可通过「bitkylin-mq-client-consumer」的控制台，看到消息不断被消费。

```text
2021-01-24 01:55:58.008  INFO 2516 --- [pool-1-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-1: topic-1-msg:1
2021-01-24 01:56:00.996  INFO 2516 --- [pool-1-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-1: topic-1-msg:2
2021-01-24 01:56:04.000  INFO 2516 --- [pool-1-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-1: topic-1-msg:3
2021-01-24 01:56:07.004  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:4
2021-01-24 01:56:10.015  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:5
2021-01-24 01:56:13.011  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:6
2021-01-24 01:56:16.011  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:7
2021-01-24 01:56:19.006  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:8
2021-01-24 01:56:21.997  INFO 2516 --- [pool-1-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-1: topic-1-msg:9
2021-01-24 01:56:24.994  INFO 2516 --- [pool-1-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-1: topic-1-msg:10
2021-01-24 01:56:28.002  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:11
2021-01-24 01:56:30.991  INFO 2516 --- [pool-1-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-1: topic-1-msg:12
2021-01-24 01:56:34.014  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:13
2021-01-24 01:56:37.010  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:14
2021-01-24 01:56:40.004  INFO 2516 --- [pool-2-thread-1] .c.c.BitkylinMqClientConsumerApplication : 收到消息：spring-group-2: topic-2-msg:15
```

1. 打开postman，发送如下请求创建专用于postman的消费者组：

```text
POST http://localhost:8080/mq/broker/consumer-group/create

{
    "groupName": "postman-group-1",
    "topicName": "topic-1"
}
```

1. 发送如下请求即可消费消息，且自动确认「无需手动更新消费者组的offset」

```text
POST http://localhost:8080/mq/broker/message/simple-pull

{
    "groupName": "postman-group-1",
    "topicName": "topic-1"
}
```

可以发现，postman可以独立消费指定topic的消息，不受Spring程序消费的影响。当然，postman可以直接使用Spring程序一致的消费者组，以共同消费消息。

此时查询MQ的概览信息：

```text
GET http://localhost:8080/mq/broker/overview
```

响应：

```json
{
  "groupList": [
    {
      "groupName": "spring-group-1",
      "topic": {
        "name": "topic-1",
        "capacity": 1000,
        "maxIndex": 14
      },
      "offset": 15
    },
    {
      "groupName": "postman-group-1",
      "topic": {
        "name": "topic-1",
        "capacity": 1000,
        "maxIndex": 14
      },
      "offset": 5
    },
    {
      "groupName": "spring-group-2",
      "topic": {
        "name": "topic-2",
        "capacity": 1000,
        "maxIndex": 17
      },
      "offset": 18
    }
  ]
}
```

## 局限性

1. 每个topic的队列容量是固定的，队列满后拒绝生产消息，暂不支持清理历史消息。
2. 消息消费未加锁，如果一个消费者组的多个消费者高并发消费消息，可能导致同一条消息被消费多次。