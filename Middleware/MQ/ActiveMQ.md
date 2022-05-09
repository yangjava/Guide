# ActiveMQ

## 概述

ActiveMQ是一种开源的，实现了JMS1.1规范的，面向消息(MOM)的中间件，为应用程序提供高效的、可扩展的、稳定的和安全的企业级消息通信。ActiveMQ使用Apache提供的授权，任何人都可以对其实现代码进行修改。

ActiveMQ的设计目标是提供标准的，面向消息的，能够跨越多语言和多系统的应用集成消息通信中间件。

**官网**： http://activemq.apache.org/

## 特性(features)

ActiveMQ实现了JMS标准并提供了很多附加的特性。这些附加的特性包括:

**官网**：https://activemq.apache.org/features

1.JMX管理（java Management Extensions，即java管理扩展）

2.主从管理（master/salve，这是集群模式的一种，主要体现在可靠性方面，当主中介（代理）出现故障，那么从代理会替代主代理的位置，不至于使消息系统瘫痪）

3.消息组通信（同一组的消息，仅会提交给一个客户进行处理）

4.有序消息管理（确保消息能够按照发送的次序被接受者接收）

5.消息优先级（优先级高的消息先被投递和处理）

6.订阅消息的延迟接收（订阅消息在发布时，如果订阅者没有开启连接，那么当订阅者开启连接时，消息中介将会向其提交之前的，其未处理的消息）

7.接收者处理过慢（可以使用动态负载平衡，将多数消息提交到处理快的接收者，这主要是对PTP消息所说）、虚拟接收者（降低与中介的连接数目）

8.成熟的消息持久化技术（部分消息需要持久化到数据库或文件系统中，当中介崩溃时，信息不会丢失）

9.支持游标操作（可以处理大消息）

10.支持消息的转换、通过使用Apache的Camel可以支持EIP、使用镜像队列的形式轻松的对消息队列进行监控等。

支持JMS规范：ActiveMQ完全实现了JMS1.1规范。  

JMS规范提供了同步消息和异步消息投递方式、有且仅有一次投递语义（指消息的接收者对一条消息必须接收到一次，并且仅有一次）、订阅消息持久接收等。如果仅使用JMS规范，表明无论您使用的是哪家厂商的消息代理，都不会影响到您的程序。  

连接方式的多样化：ActiveMQ提供了广泛的连接模式，包括HTTP/S、JGroups、JXTA、muticast、SSL、TCP、UDP、XMPP等。提供了如此多的连接模式表明了ActiveMQ具有较高的灵活性。  

可插入式的持久化和安全：ActiveMQ提供了多种持久化方案，您可以根据实际需要进行选择。同时，也提供了完整的客户授权模式。  

使用Java创建消息应用程序：最常见的使用ActiveMQ的方式就是使用Java程序来发送和接收消息。  

与其他的Java容器紧密集成：ActiveMQ提供了和其它流行的Java容器的结合，包括Apache Geronimo、Apache Tomcat、JBoss、Jetty等。  

客户端API：ActiveMQ提供了多种客户端可访问的API，包括Java、C/C++，.NET，Perl、PHP、Python、Ruby等。当然，ActiveMQ中介必须运行在Java虚拟机中，但是使用它的客户端可以使用其他的语言来实现。  

中介集群：多个ActiveMQ中介可以一起协同工作，来完成某项复杂的工作，这被称为网络型中介（network of brokers），这种类型的中介将会支持多种拓扑类型。

## ActiveMQ概念

### Queue

队列存储，常用与点对点消息模型

默认只能由唯一的一个消费者处理。一旦处理消息删除。

### Topic

主题存储，用于订阅/发布消息模型

主题中的消息，会发送给所有的消费者同时处理。只有在消息可以重复处 理的业务场景中可使用。

Queue/Topic都是 Destination 的子接口

### ConnectionFactory

连接工厂，jms中用它创建连接

连接工厂是客户用来创建连接的对象，例如ActiveMQ提供的ActiveMQConnectionFactory。

### Connection

JMS Connection封装了客户与JMS提供者之间的一个虚拟的连接。

### Destination

消息的目的地

目的地是客户用来指定它生产的消息的目标和它消费的消息的来源的对象。JMS1.0.2规范中定义了两种消息传递域：点对点（PTP）消息传递域和发布/订阅消息传递域。

### Session

JMS Session是生产和消费消息的一个单线程上下文。会话用于创建消息生产者（producer）、消息消费者（consumer）和消息（message）等。会话提供了一个事务性的上下文，在这个上下文中，一组发送和接收被组合到了一个原子操作中。



点对点消息传递域的特点如下：

- 每个消息只能有一个消费者。
- 消息的生产者和消费者之间没有时间上的相关性。无论消费者在生产者发送消息的时候是否处于运行状态，它都可以提取消息。



发布/订阅消息传递域的特点如下：

- 每个消息可以有多个消费者。
- 生产者和消费者之间有时间上的相关性。
- 订阅一个主题的消费者只能消费自它订阅之后发布的消息。JMS规范允许客户创建持久订阅，这在一定程度上放松了时间上的相关性要求 。持久订阅允许消费者消费它在未处于激活状态时发送的消息。
  在点对点消息传递域中，目的地被成为队列（queue）；在发布/订阅消息传递域中，目的地被成为主题（topic）

### 拉模式与推模式

a.点对点消息，如果没有消费者在监听队列，消息将保留在队列中，直至消息消费者连接到队列为止。这种消息传递模型是

传统意义上的懒模型或轮询模型。在此模型中，消息不是自动推动给消息消费者的，而是要由消息消费者从队列中请求获得(拉模式)。

b.pub/sub消息传递模型基本上是一个推模型。在该模型中，消息会自动广播，消息消费者无须通过主动请求或轮询主题的方法来获得新的消息。

## **ActiveMQ角色介绍**

![ActiveMQ角色](png\ActiveMQ角色.png)

- **Destination 目的地**：消息发送者需要指定Destination才能发送消息，接收者需要指定Destination才能接收消息。
- **Producer消息生产者**：负责发送Message到目的地
- **Consumer/Receiver消息消费者**：负责从目的地中消费（处理/监听/订阅）Message
- **Message消息**：消息封装一次通信的内容

## ActiveMQ两种消息模式

### 点对点( Point-to-Point)

**基于点对点的消息模型**

点对点的模式主要建立在一个队列上面，当连接一个列队的时候，发送端不需要知道接收端是否正在接收，可以直接向ActiveMQ发送消息，发送的消息，将会先进入队列中，如果有接收端在监听，则会发向接收端，如果没有接收端接收，则会保存在activemq服务器，直到接收端接收消息，点对点的消息模式可以有多个发送端，多个接收端，但是一条消息，只会被一个接收端给接收到，哪个接收端先连上ActiveMQ，则会先接收到，而后来的接收端则接收不到那条消息

消息生产者生产消息发送到 queue 中，然后消息消费者从 queue 中取出并且消费消息。

消息被消费以后，queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息。

Queue 支持存在多个消费者，但是对一个消息而言，消息只能被一个消费者消费,其它 的则不能消费此消息了。 当消费者不存在时，消息会一直保存，直到有消费消费

### 发布/订阅(Publish/Subscribe)

**基于订阅/发布的消息模型**

订阅/发布模式，同样可以有着多个发送端与多个接收端，但是接收端与发送端存在时间上的依赖，就是如果发送端发送消息的时候，接收端并没有监听消息，那么ActiveMQ将不会保存消息，将会认为消息已经发送，换一种说法，就是发送端发送消息的时候，接收端不在线，是接收不到消息的，哪怕以后监听消息，同样也是接收不到的。这个模式还有一个特点，那就是，发送端发送的消息，将会被所有的接收端给接收到，不类似点对点，一条消息只会被一个接收端给接收到

消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消 息。 发布到 topic 的消息会被所有订阅者消费。 当生产者发布消息，不管是否有消费者。都不会保存消息 。一定要先有消息的消费者，后有消息的生产者。

## **ActiveMQ的使用**

### **ActiveMQ环境搭建**

- 具体安装步骤参考官网http://activemq.apache.org
- ActiveMQ运行需要Java的支持，首先需要配置Java环境变量
- 切换到解压后的activemq的bin目录下去启动
- 在bin目录下执行 **./activemq start**命令，即可启动MQ服务，如果启动服务需要指定配置文件，命令为 **./activemq start xbean:file:../conf/myConfig.xml**，不指定默认为conf目录下的activemq.xml。停止MQ服务的命令为 **./activemq stop**。
- 在conf目录下找到activemq.xml配置文件打开，里面包含如下内容

       ```
       <transportConnectors>
           <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
           <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
           <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
           <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
           <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
           <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
       </transportConnectors>
       ```

这里配置的是MQ服务的各种传输协议连接和默认端口。再往下会发现这行内容`<import resource="jetty.xml"/>`，activemq.xml文件中导入了一个名为`jetty.xml`的配置文件，在conf目录下找到jetty.xml文件打开，里面配置了访问MQ服务web控制台的一些信息，

```
<bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
    <!-- the default port number for the web console -->
    <property name="host" value="0.0.0.0"/>
    <property name="port" value="8161"/>
</bean>
```

- 启动后有两个端口号，一个是web控制台:8161，一个是消息服务broker连接端口：61616
- web管理控制台admin URL地址：`http://localhost:8161` 默认登录账号 admin 密码 admin，注意：Linux防火前要关闭 ;通过这个地址可以即时访问交互信息.如下图

![ActiveMQ-console](png\ActiveMQ-console.png)

消息服务broker URL地址   `tcp://localhost:61616`

### docker安装ActiveMQ

拉取镜像：`docker pull webcenter/activemq`

查看镜像：`docker images`

创建[挂载](https://so.csdn.net/so/search?q=挂载&spm=1001.2101.3001.7020)目录：

```bash
mkdir /usr/soft/activemq

mkdir /usr/soft/activemq/log
```

运行activeMQ镜像：

```bash
	docker run --name='activemq' \
      -itd \
	  -p 8161:8161 \
	  -p 61616:61616 \
	  -e ACTIVEMQ_ADMIN_LOGIN=admin \
	  -e ACTIVEMQ_ADMIN_PASSWORD=123456 \
	  --restart=always \
	  -v /usr/soft/activemq:/data/activemq \
	  -v /usr/soft/activemq/log:/var/log/activemq \
	  webcenter/activemq:latest
```

61616是 activemq 的容器使用端口
8161是 web 页面管理端口
/usr/soft/activemq 是将activeMQ运行文件挂载到该目录
/usr/soft/activemq/log是将activeMQ运行日志挂载到该目录
-e ACTIVEMQ_ADMIN_LOGIN=admin 指定登录名
-e ACTIVEMQ_ADMIN_PASSWORD=123456 登录密码

浏览器访问IP:8161，即可看到欢迎页，点击登录，输入账号密码，可进入activeMQ后台。

### 控制台使用

队列表头说明：

表头名称                            描述
Name                              队列名称。
Number Of Pending Messages          等待消费的消息，这个是未出队列的数量（类似未读消息数量），公式：未出队列的数量=总入队数-总出队数。
Number Of Consumers                消费者数量，消费者端的消费者数量。
Messages Enqueued                  进队消息数，进队列的总消息量，包括出队列的。这个数只增不减。
Messages Dequeued                  出队消息数，可以理解为是消费者消费掉的数量。

## 消息可靠性机制

### 持久性

JMS 支持以下两种消息提交模式：

- PERSISTENT。指示JMS Provider持久保存消息，以保证消息不会因为JMS Provider的失败而丢失。
- NON_PERSISTENT。不要求JMS Provider持久保存消息。

### 优先级

可以使用消息优先级来指示JMS Provider首先提交紧急的消息。优先级分10个级别，从0（最低）到9（最高）。如果不指定优先级，默认级别是4。需要注意的是，JMS Provider并不一定保证按照优先级的顺序提交消息。

### 消息过期

可以设置消息在一定时间后过期，默认是永不过期。

### 临时目的地

可以通过会话上的createTemporaryQueue方法和createTemporaryTopic方法来创建临时目的地。它们的存在时间只限于创建它们的连接所保持的时间。只有创建该临时目的地的连接上的消息消费者才能够从临时目的地中提取消息。

### 持久订阅

首先消息生产者必须使用PERSISTENT提交消息。客户可以通过会话上的createDurableSubscriber方法来创建一个持久订阅，该方法的第一个参数必须是一个topic，第二个参数是订阅的名称。 JMS Provider会存储发布到持久订阅对应的topic上的消息。如果最初创建持久订阅的客户或者任何其它客户使用相同的连接工厂和连接的客户ID、相同的主题和相同的订阅名再次调用会话上的createDurableSubscriber方法，那么该持久订阅就会被激活。JMS Provider会象客户发送客户处于非激活状态时所发布的消息。 持久订阅在某个时刻只能有一个激活的订阅者。持久订阅在创建之后会一直保留，直到应用程序调用会话上的unsubscribe方法。

### 本地事务

在一个JMS客户端，可以使用本地事务来组合消息的发送和接收。JMS Session接口提供了commit和rollback方法。事务提交意味着生产的所有消息被发送，消费的所有消息被确认；事务回滚意味着生产的所有消息被销毁，消费的所有消息被恢复并重新提交，除非它们已经过期。 事务性的会话总是牵涉到事务处理中，commit或rollback方法一旦被调用，一个事务就结束了，而另一个事务被开始。关闭事务性会话将回滚其中的事务。 需要注意的是，如果使用请求/回复机制，即发送一个消息，同时希望在同一个事务中等待接收该消息的回复，那么程序将被挂起，因为知道事务提交，发送操作才会真正执行。 需要注意的还有一个，消息的生产和消费不能包含在同一个事务中。

**ActiveMQ事务消息和非事务消息**

消息分为事务消息和非事务消息

事务消息：创建会话Session使用transacted=true

```
connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
```

非事务消息：创建会话Session使用transacted=false

```
connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
```

事务消息必须在发送和接收完消息后显式地调用session.commit();

事务性消息，不管设置何种消息确认模式，都会自动被确认；与设置的确认机制无关,但官方推荐事务性消息使用事务确认机制.

**ActiveMQ消息确认机制**

```
消息只有在被确认之后，才认为已经被成功消费，然后消息才会从队列或主题中删除。消息的成功消费通常包含三个阶段：
```

(1)、客户接收消息；(2)、客户处理消息;(3)、消息被确认；

确认机制前面提过一下,共有四种:

(1)、Session.AUTO_ACKNOWLEDGE；客户（消费者）成功从receive方法返回时，或者从MessageListener.onMessage方法成功返回时，会话自动确认消息,然后自动删除消息.

(2)、Session.CLIENT_ACKNOWLEDGE；客户通过显式调用消息的acknowledge方法确认消息,。 即在接收端调用message.acknowledge();方法,否则,消息是不会被删除的.

(3)、Session. DUPS_OK_ACKNOWLEDGE ；不是必须确认，是一种“懒散的”消息确认，消息可能会重复发送，在第二次重新传送消息时，消息头的JMSRedelivered会被置为true标识当前消息已经传送过一次，客户端需要进行消息的重复处理控制。

(4)、 Session.SESSION_TRANSACTED；事务提交并确认。

## ActiveMQ消息类型

1、TextMessage 文本消息：携带一个java.lang.String作为有效数据(负载)的消息，可用于字符串类型的信息交换；

2、ObjectMessage 对象消息：携带一个可以序列化的Java对象作为有效负载的消息，可用于Java对象类型的信息交换；

3、MapMessage 映射消息：携带一组键值对的数据作为有效负载的消息，有效数据值必须是Java原始数据类型（或者它们的包装类）及String。即：byte , short , int , long , float , double , char , boolean , String

4、BytesMessage 字节消息 ：携带一组原始数据类型的字节流作为有效负载的消息；

5、StreamMessage 流消息：携带一个原始数据类型流作为有效负载的消息，它保持了写入流时的数据类型，写入什么类型，

则读取也需要是相同的类型；

需要注意的是:如果使用对象消息做Demo时,如果使用之前的连接创建方式可能会无法接收到消息,因为安全问题.所以,如果想要使用Demo测试对象消息,创建连接时建议改为这样(官网推荐的连接方式):

```
//创建对象消息连接工厂
ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(BROKER_URL);
List<String > list = new ArrayList<String>();
list.add("com.kinglong.activemq.receiver");
list.add("com.kinglong.activemq.model");
activeMQConnectionFactory.setTrustedPackages(list);
```



**ActiveMQ持久化消息与非持久化消息**

```
messageProducer.setDeliveryMode(DeliveryMode. NON_PERSISTENT);//不持久化
messageProducer.setDeliveryMode(DeliveryMode.

PERSISTENT);//持久化的，当然activemq发送消息默认都是持久化的
```

说明:

设置完后,如果为持久化,那么消息在没有被消费前都会被写入本地磁盘kahadb文件中保存起来,即使服务器宕机,也不会影响

消息.如果是非持久化的,那么,服务一旦宕机之类的情况发生,消息即会被删除.

ActiveMQ默认是持久化的.

### ActiveMQ消息过滤

ActiveMQ提供了一种机制，可根据消息选择器中的标准来执行消息过滤，只接收符合过滤标准的消息；

生产者可在消息中放入特有的标志，而消费者使用基于这些特定的标志来接收消息；

1、发送消息放入特殊标志：message . setString Property ( name , value ) ;

2、接收消息使用基于特殊标志的消息选择器:

```
MessageConsumer createConsumer(Destination destination, String messageSelector);
```

注：消息选择器是一个字符串，语法与数据库的SQL相似，相当于SQL语句where条件后面的内容；

**ActiveMQ消息接收方式**

同步接收:receive()方法接收消息叫同步接收,就是之前的Demo代码使用的接收方式.在不使用循环方法时接收端代码执行

一次即结束.

异步接收:使用监听器接收消息，这种接收方式叫异步接收,接收端会一直处于监听状态,只要有消息产生,即会接收消息.





### ActiveMQ的安装与启动

ActiveMQ的代码测试

构建maven项目，引入依赖

```
<dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-all</artifactId>
            <version>5.9.0</version>
        </dependency>
```

生产者类

```
public class MyProducer {

    private static final String ACTIVEMQ_URL = "tcp://192.168.168.242:61616";

    public static void main(String[] args) throws JMSException {
        // 创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        // 打开连接
        connection.start();
        // 创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 创建队列目标,并标识队列名称，消费者根据队列名称接收数据
        Destination destination = session.createQueue("myQueue");
        // 创建一个生产者
        MessageProducer producer = session.createProducer(destination);
        // 向队列推送10个文本消息数据
        for (int i = 1 ; i <= 10 ; i++){
            // 创建文本消息
            TextMessage message = session.createTextMessage("第" + i + "个文本消息");
            //发送消息
            producer.send(message);
            //在本地打印消息
            System.out.println("已发送的消息：" + message.getText());
        }
        //关闭连接
        connection.close();
    }

}
```

消费者类

```

public class MyConsumer {

    private static final String ACTIVEMQ_URL = "tcp://192.168.168.242:61616";

    public static void main(String[] args) throws JMSException {
        // 创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        // 打开连接
        connection.start();
        // 创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 创建队列目标,并标识队列名称，消费者根据队列名称接收数据
        Destination destination = session.createQueue("myQueue");
        // 创建消费者
        MessageConsumer consumer = session.createConsumer(destination);
        // 创建消费的监听
        consumer.setMessageListener(new MessageListener() {
            public void onMessage(Message message) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    System.out.println("消费的消息：" + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

当我们运行两个消费者类，消息又是怎么被消费的呢？是两个消费者都能收到生产者生产的message，还是只有其中一个消费者能消费呢？

```
```

即队列中的数据会平均的分给每一个消费者消费，且每一条数据只能被消费一次

以上是基于队列点对点的传输类型，以下是基于发布/订阅模式传输的类型测试

生产者

```
public class MyProducerForTopic {

    private static final String ACTIVEMQ_URL = "tcp://192.168.168.242:61616";

    public static void main(String[] args) throws JMSException {
        // 创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        // 打开连接
        connection.start();
        // 创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 创建队列目标,并标识队列名称，消费者根据队列名称接收数据
        Destination destination = session.createTopic("topicTest");
        // 创建一个生产者
        MessageProducer producer = session.createProducer(destination);
        // 向队列推送10个文本消息数据
        for (int i = 1 ; i <= 10 ; i++){
            // 创建文本消息
            TextMessage message = session.createTextMessage("第" + i + "个文本消息");
            //发送消息
            producer.send(message);
            //在本地打印消息
            System.out.println("已发送的消息：" + message.getText());
        }
        //关闭连接
        connection.close();
    }

}
```

消费者

```
public class MyConsumerForTopic {

    private static final String ACTIVEMQ_URL = "tcp://192.168.168.242:61616";

    public static void main(String[] args) throws JMSException {
        // 创建连接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        // 创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        // 打开连接
        connection.start();
        // 创建会话
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        // 创建队列目标,并标识队列名称，消费者根据队列名称接收数据
        Destination destination = session.createTopic("topicTest");
        // 创建消费者
        MessageConsumer consumer = session.createConsumer(destination);
        // 创建消费的监听
        consumer.setMessageListener(new MessageListener() {
            public void onMessage(Message message) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    System.out.println("消费的消息：" + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

现在如果我们先启动生产者，再启动消费者，会发现消费者是无法接收到之前生产者之前所生产的数据，只有消费者先启动，再让生产者消费才可以正常接收数据，这也是发布/订阅的主题模式与点对点的队列模式的一个明显区别。

而如果启动两个消费者，那么每一个消费者都能完整的接收到生产者生产的数据，即每一条数据都被消费了两次，这是发布/订阅的主题模式与点对点的队列模式的另一个明显区别。

**ActiveMQ集群**

## ActiveMQ结合spring开发

Spring提供了对JMS的支持，需要添加Spring 支持JMS的包

#### POM.xml 

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.0</version>
</dependency>
<!--结合spring-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.4.2</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>3.0.1.RELEASE-A</version>
</dependency>
```

#### 生产者配置

**service-jms-provider.xml**

```xml
<bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
    <property name="connectionFactory">
        <bean class="org.apache.activemq.ActiveMQConnectionFactory">
            <property name="brokerURL">
                <value>tcp://192.168.98.165:61616</value>
            </property>
        </bean>
    </property>
    <property name="maxConnections" value="50"/>
</bean>
<bean id="destination" class="org.apache.activemq.command.ActiveMQQueue">
    <constructor-arg index="0" value="spring-queue"/>
</bean>

<!--<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
    <constructor-arg index="0" value="spring-topic"/>
</bean>-->
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="defaultDestination" ref="destination"/>
    <property name="messageConverter">
        <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
    </property>
</bean>
```

#### 编写发送端代码

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext context=
            new ClassPathXmlApplicationContext(
                    "classpath:META-INF/spring/service-jms-provider.xml");

    JmsTemplate jmsTemplate=(JmsTemplate) context.getBean("jmsTemplate");

    jmsTemplate.send(new MessageCreator() {
        public Message createMessage(Session session) throws JMSException {
            TextMessage message=session.createTextMessage();
            message.setText("Hello,charjay");
            return message;
        }
    });
}
```



#### 消费者配置

**service-jms-consumer.xml**

```xml
   <bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
       <property name="connectionFactory">
           <bean class="org.apache.activemq.ActiveMQConnectionFactory">
               <property name="brokerURL">
                   <value>tcp://192.168.98.165:61616</value>
               </property>
           </bean>
       </property>
       <property name="maxConnections" value="50"/>
   </bean>
   <bean id="destination" class="org.apache.activemq.command.ActiveMQQueue">
       <constructor-arg index="0" value="spring-queue"/>
   </bean>

<!--   <bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
       <constructor-arg index="0" value="spring-topic"/>
   </bean>-->

   <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
       <property name="connectionFactory" ref="connectionFactory"/>
       <property name="defaultDestination" ref="destination"/>
       <property name="messageConverter">
           <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
       </property>
   </bean>
```

#### 编写接收端代码

```java
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context=
                new ClassPathXmlApplicationContext(
                        "classpath:META-INF/spring/service-jms-consumer.xml");

        JmsTemplate jmsTemplate=(JmsTemplate) context.getBean("jmsTemplate");
        String msg=(String)jmsTemplate.receiveAndConvert();
        System.out.println(msg);
    }
```

#### spring的发布订阅模式配置

```xml
<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
     <constructor-arg index="0" value="spring-topic"/>
 </bean>

 <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
     <property name="connectionFactory" ref="connectionFactory"/>
     <property name="defaultDestination" ref="destinationTopic"/>
     <property name="messageConverter">
         <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
     </property>
 </bean>
```

#### 以事件通知方式来配置消费者

#### 更改消费端的配置

```xml
<!--监听-->
  <bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
      <property name="connectionFactory" ref="connectionFactory"/>
      <property name="destination" ref="destination"/>
      <property name="messageListener" ref="messageListener"/>
  </bean>
  <bean id="messageListener" class="com.charjay.spring.SpringJmsListener"/>
```

#### 增加FirstMessageListener监听类

```java
public class SpringJmsListener implements MessageListener{
    @Override
    public void onMessage(Message message) {
        try {
            System.out.println(((TextMessage)message).getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

#### 启动spring容器

```java
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context=
                new ClassPathXmlApplicationContext(
                        "classpath:META-INF/spring/service-jms-consumer.xml");

        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

#### 容错连接

```java
failover:(tcp://192.168.98.165:61616,tcp://192.168.98.166:61616)
```

# ActiveMQ支持的传输协议

1）client端和broker端的通讯协议：TCP、UDP 、NIO、SSL、Http（s）、vm

2）定义一个nio协议

vim conf/activemq.xml

```xml
<transportConnectors>
    <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="nio" uri="nio://0.0.0.0:61618?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>    
</transportConnectors>
```



重启后使用

```java
public final static String ACTIVE_MQ_URL="nio://192.168.98.165:61616";
```

# ActiveMQ持久化存储

## kahaDB  默认的存储方式

```xml
<persistenceAdapter>
    <kahaDB directory="${activemq.data}/kahadb"/>
</persistenceAdapter>
```

## AMQ 基于文件的存储方式

- 写入速度很快，容易恢复。

- 文件默认大小是32M

## JDBC 基于数据库的存储

ACTIVEMQ_ACKS ： 存储持久订阅的信息

ACTIVEMQ_LOCK ： 锁表（用来做集群的时候，实现master选举的表）

ACTIVEMQ_MSGS ： 消息表

### 丢失的消息

一些consumer连接到broker1、消费broker2上的消息。消息先被broker1从broker2消费掉，然后转发给这些consumers。假设，转发消息的时候broker1重启了，这些consumers发现brokers1连接失败，通过failover连接到broker2.但是因为有一部分没有消费的消息被broker2已经分发到broker1上去了，这些消息就好像消失了。除非有消费者重新连接到broker1上来消费



从5.6版本开始，在destinationPolicy上新增了一个选项replayWhenNoConsumers属性，这个属性可以用来解决当broker1上有需要转发的消息但是没有消费者时，把消息回流到它原始的broker。同时把enableAudit设置为false，为了防止消息回流后被当作重复消息而不被分发

通过如下配置，在activeMQ.xml中。 分别在两台服务器都配置。即可完成消息回流处理

## ActiveMQ之虚拟主题

ActiveMQ支持的虚拟Destinations分为有两种，分别是
1.虚拟主题（Virtual Topics）
2.组合 Destinations（CompositeDestinations）

这两种虚拟Destinations可以看做对简单的topic和queue用法的补充，基于它们可以实现一些简单有用的EIP功能，虚拟主题类似于1对多的分支功能+消费端的cluster+failover，组合Destinations类似于简单的destinations直接的路由功能。

虚拟主题（Virtual Topics）
ActiveMQ中，topic只有在持久订阅（durablesubscription）下是持久化的。存在持久订阅时，每个持久订阅者，都相当于一个持久化的queue的客户端，它会收取所有消息。这种情况下存在两个问题：
1.同一应用内consumer端负载均衡的问题：同一个应用上的一个持久订阅不能使用多个consumer来共同承担消息处理功能。因为每个都会获取所有消息。queue模式可以解决这个问题，broker端又不能将消息发送到多个应用端。所以，既要发布订阅，又要让消费者分组，这个功能jms规范本身是没有的。
2.同一应用内consumer端failover的问题：由于只能使用单个的持久订阅者，如果这个订阅者出错，则应用就无法处理消息了，系统的健壮性不高。
为了解决这两个问题，ActiveMQ中实现了虚拟Topic的功能。使用起来非常简单。
对于消息发布者来说，就是一个正常的Topic，名称以VirtualTopic.开头。例如VirtualTopic.TEST。
对于消息接收端来说，是个队列，不同应用里使用不同的前缀作为队列的名称，即可表明自己的身份即可实现消费端应用分组。例如Consumer.A.VirtualTopic.TEST，说明它是名称为A的消费端，同理Consumer.B.VirtualTopic.TEST说明是一个名称为B的客户端。可以在同一个应用里使用多个consumer消费此queue，则可以实现上面两个功能。又因为不同应用使用的queue名称不同（前缀不同），所以不同的应用中都可以接收到全部的消息。每个客户端相当于一个持久订阅者，而且这个客户端可以使用多个消费者共同来承担消费任务。

优点：1. 虚拟主题定义方式灵活，可以使用VirtualTopic前缀（缺点是发布者/订阅者都要修改，优点是不需要修改mq配置）；也可以在修改mq配置（优点是发布者和部分订阅者不需要修改，缺点是修改mq配置）。2. 不关心或者不想用队列的订阅者不用做任何改动。3. 可以通过通配符来修改某些符合命名规则的主题为虚拟主题。

缺点：1. 需要修改mq配置。但是通过配置文件不需要发布者做任何改动。（发布者改动可能影响所有订阅者，因此只修改订阅者成本小。）代码示例：

```
Destination dest = session.createQueue("Consumer.A.VirtualTopic.Orders");
```

代码如下：

```
package test.mq.visualDestinations;

 

 
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.DeliveryMode;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.MapMessage;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;

public class Sender {
       public static void main(String[] args) throws JMSException {
           
           ConnectionFactory   ConnectionFactory=new ActiveMQConnectionFactory(
                "tcp://localhost:61616"
                );
        
        Connection connection=ConnectionFactory.createConnection();
        connection.start();

        Session session=connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
        
        Destination destination=session.createTopic("VirtualTopic.mytopic1");
         
        MessageProducer messageProducer=session.createProducer(destination);
     
        for(int i=1;i<=5;i++){
             TextMessage textMessage=session.createTextMessage();
             textMessage.setText("我是TOM ID为"+i);
             messageProducer.send(textMessage);
             System.out.println("生产者："+textMessage.getText());
            
        }
         session.commit();
         session.close();
         connection.close();    
    }
}
```

```
package test.mq.visualDestinations;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;

public class Receiver1 {
    public static void main(String[] args) throws Exception {
        ConnectionFactory cf = new ActiveMQConnectionFactory("tcp://localhost:61616");
        Connection connection =  cf.createConnection();
        connection.start();
        
    Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
    Destination destination = session.createQueue("Consumer.A.VirtualTopic.mytopic1");
        MessageConsumer consumer = session.createConsumer(destination);
        int i = 0;
        while(i < 5){
            Thread.sleep(1000);
            i++;
            TextMessage message = (TextMessage)consumer.receive();
            session.commit();
            System.out.println("1接收到的消息是:"+message.getText());
        }
        session.close();
        connection.close();
    }
}
```

```
package test.mq.visualDestinations;

import java.util.Enumeration;
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.MessageListener;
import javax.jms.Session;
import javax.jms.TextMessage;
import org.apache.activemq.ActiveMQConnectionFactory;
public class Receiver2{
    
    public static void main(String[] args) throws JMSException {
        ConnectionFactory   connectionFactory=new ActiveMQConnectionFactory(
                "tcp://localhost:61616"
                );
        for(int i=0;i<5;i++){
            Thread    t=new MyThread2(connectionFactory);
            t.start();
            try {
                Thread.sleep(1000l);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    
    }
}
class MyThread2 extends Thread{
         private ConnectionFactory   connectionFactory=null;
         public  MyThread2(ConnectionFactory   connectionFactory){
         this.connectionFactory = connectionFactory;
         }
       public void run(){
            try {
                final Connection  connection = connectionFactory.createConnection();
                connection.start();
                Enumeration names=connection.getMetaData().getJMSXPropertyNames();
                 
                final Session session=connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE); 
                Destination destination=session.createQueue("Consumer.B.VirtualTopic.mytopic1");
                MessageConsumer Consumer=session.createConsumer(destination);
                Consumer.setMessageListener(new MessageListener() {
                    @Override
                    public void onMessage(Message msg) {
                    TextMessage     txtmsg=(TextMessage) msg; 
                    try {
                        System.out.println("接收信息2--->"+txtmsg.getText());
                    } catch (JMSException e1) {
                        e1.printStackTrace();
                    }
                    try {
                        session.commit();
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                    try {
                        session.close();
                    } catch (JMSException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    try {
                        connection.close();
                    } catch (JMSException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    }
                });
                 
                
                
                
            } catch (JMSException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            
       }    
    }
```

其实把消费者队列化了。

修改虚拟主题的前缀：

默认前缀是VirtualTopic.>

自定义消费虚拟地址默认格式：Consumer.*.VirtualTopic.>

修改配置：

```
<broker xmlns="http://activemq.apache.org/schema/core">
       <destinationInterceptors>
              <virtualDestinationInterceptor>
                     <virtualDestinations>
                            <virtualTopic name=">" prefix="VirtualTopicConsumers.*." selectorAware="false" />
                     </virtualDestinations>
              </virtualDestinationInterceptor>
       </destinationInterceptors>
</broker>
```

消息分组特性

        上面的例子就用到了queue模式的分组消息特性，如果只有一个应用多节点部署的情况下，可以采用queue的选择器selector来做分组。
    
        这种模式下每个节点的消费之可以消费固定的生产者生产的消息，但是也有个缺点，如果某个节点挂了，那么其对应的生产者消息也将无法消费。
    
         Message Group是针对queue，对topic无感！如果在queue模式下，一个生产者对应多个消费者，每生产一条消息，会被消费随即 抢到，如果我们不希望这样，只希望固定的消息被固定的消费者消费，那么就采用group对消息进行一个类似标记的作用。分组要依赖消息选择器，selector。
虚拟主题模式（VirtualTopic）

消息模型

虚拟主题模式是topic和queue模式下的结合可以修改ActiveMq本身的配置文件实现。也可以通过发送和接收主题的名称配置来实现。第一种对消息中间件的侵入性太强，升级需要重启MQ会导致消息丢失的问题。推荐使用第二种方式。在这种模式下发送消息使用的是topic的方式发送。接收消息可以使用queue模式下的注解方式接收消息。发送的目标主题名称必须加前缀“VirtualTopic.”例如我们有一个目标主题：restuarant_vrtopic，发送目标则为：“VirtualTopic.restuarant_vrtopic”。接收使用队列方式，接收目标统一加前缀“Consumer.*.”。其中*代表消费者的唯一标识。下面将详细说明：

## ActiveMQ之VirtualTopic是什么？

https://www.likecs.com/show-203438836.html

一句话总结： VirtualTopic是为了解决持久化模式下多消费端同时接收同一条消息的问题。

 

想象这样一个场景：

 

生产端产生了一笔订单，作为消息MessageOrder发了出去。

这笔订单既要入订单系统归档，又要入结算系统收款，那怎么办呢？

 

现在分析该消息的需求：

 

持久化：订单很重要，丢了可不行

同时接收：既要归档，又要结算

生产端只需向一个Destination发送：一把钥匙开一把锁，保持发送的一致性，否则容易乱套

 

方案A: 使用Topic订阅模式，虽然满足1对多同时接收，然而持久化模式下只能有一个持有clientID的消费者连接，不满足持久化需求

方案B: 使用单队列，队列是1对1模式，消息只能给一个消费者，不满足同时接收的需求

方案C: 使用多队列，显然生产者不太愿意一条消息发送很多次，分别发送给不同的队列，万一队列A发送成功，队列B发送失败怎么办？一致性无法保证，容易乱套

 

所以，JMS现有规范无法解决这个问题，于是，ActiveMQ使用VirtualTopic作为JMS规范的补充登场。

 

那VirtualTopic如何同时满足上述需求呢？

 

简单说来，就是将Topic和Queue相结合，各取所长。

 

在方案C中，我们发现使用多队列可以满足持久化和同时接收两个需求，但意味着生产者要发送消息给多个队列，一致性不好，那既然生产者不想分发，那么由Broker来分发可好？

 

VirtualTopic就是这样一种存在，对生产者而言它是Topic，对消费者而言它是Queue，内部的处理机制就是由Broker将接收到的消息二次分发给每一个Queue，然后由不同的Queue对应不同的应用实现持久化，不同的消费端只关心并连接到自己的Queue接收消息即可。

 

现在来复盘开始提出的场景：

![img](https://images2018.cnblogs.com/blog/1192583/201807/1192583-20180730112207060-1297917580.png)

显然，三个需求都得到了解决。

 

**总结一下：**

**1. 虚拟Topic是一种特殊命名的Topic，系统根据命名规则将该Topic内的消息分发给当前存在的名称对应的Queue，分发是非持久化的，新加入的Queue是接收不到过去的消息的。**

**2. 虚拟Topic还是Topic，不是什么新的存在，具有普通Topic的所有功能，只是名字特殊而已。**

**3. 虚拟Topic的功能完全是中间件本身额外附加的机制，对于生产者和消费者都是无感知的。**

**4. 对于运维人员来说，还是正常监控队列即可，虚拟Topic是非持久化的，不存在积压。**

![img](https://images2018.cnblogs.com/blog/1192583/201807/1192583-20180730111949606-710910232.png)

 

## 存在的问题

怎么解决系统之间关心同样的消息，同时系统内部又想达到负载和故障转移的目的？



# ActiveMQ源码

## 消息队列MQ概述

消息队列（Message Queue，简称MQ），指保存消息的一个容器，本质是个队列。

消息（Message）是指在应用之间传送的数据，消息可以非常简单，比如只包含文本字符串，也可以更复杂，可能包含嵌入对象。

消息队列的基本模型，向消息队列中存放数据的叫做生产者，从消息队列中获取数据的叫做消费者。

## 整体架构

![ActiveMQ](png\ActiveMQ.png)

上图为整体架构，会涉及三类角色：

1）**Producer** 消息生产者：负责产生和发送消息到 Broker

2）**Broker** 消息处理中心：负责消息存储、确认、重试等，一般其中会包含多个 queue

3）**Consumer** 消息消费者：负责从 Broker 中获取消息，并进行相应处理

#### 详细的流程

producer发送给broker,broker发送给consumer,consumer回复消费确认，broker删除/备份消息等。

1）RPC 通信

Producer生产消息向Broker发送会涉及到通信的问题，同样Consumer 消费消息也会涉及到通信的问题。

上图中的Producer,Broker,Consumer最后就通过RPC将数据流串起来了，所以需要解决通信的问题。

你可以基于Netty 来做底层通信，用 Zookeeper、Euraka 等来做注册中心，然后自定义一套新的通信协议。

也可以直接利用成熟的 RPC 框架 Dubbo 或者 Thrift 实现即可，这样不需要考虑服务注册与发现、负载均衡、通信协议、序列化方式等一系列问题了。

2）Broker存储

消息到达服务端后需要存储到Broker。

大家关注的流量削峰、最终一致性等需求都是需要Broker先存储下来，然后选择时机投递，这才达到流量削峰、泄洪的目的，所以Broker一个非常重要的功能就是存储。

存储可以做成很多方式，比如存储在内存里，存储在分布式KV里，存储在磁盘里，存储在数据库里等等，存储的选型需要综合考虑性能/高可用和开发维护成本等诸多因素。

目前主流的方案：追加写日志文件（数据部分） + 索引文件的方式，索引设计上可以考虑稠密索引或者稀疏索引，查找消息可以利用跳转表、二份查找等，还可以通过操作系统的页缓存、零拷贝等技术来提升磁盘文件的读写性能。

3）消费模型

消息到达Broker后，最终还是需要Consumer去消费消息，这里就会涉及到到消费模型。

目前主要就两种：单播和广播。所谓单播，就是点到点；而广播，是一点对多点。

4）高级特性

如果Consumer端把消息消费了，除了需要消息确认，还会涉及到比如：重复消息、顺序消息、消息延迟、事务消息等需要考虑的高级特性。

## 消息队列MQ模型 

消息队列MQ主要包含两种模型：点对点与发布订阅两种模型。

1.点对点模型


点对点模用于 消息生产者 和 消息消费者 之间 点到点 的通信，包含三个角色：

消息队列（Queue）

发送者(Sender)

接收者(Receiver)

每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，可以放在 内存 中也可以 持久化，直到他们被消费或超时。

特点

每个消息只有一个消费者（Consumer）(即一旦被消费，消息就不再在消息队列中)

发送者和接收者之间在时间上没有依赖性

接收者在成功接收消息之后需向队列应答成功

2.发布订阅消息模型Topic


发布订阅模型包含三个角色：

主题（Topic）

发布者（Publisher）

订阅者（Subscriber）

多个发布者将消息发送到Topic,系统将这些消息传递给多个订阅者。

特点

每个消息可以有多个消费者：和点对点方式不同，发布消息可以被所有订阅者消费

发布者和订阅者之间有时间上的依赖性。

针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息。

为了消费消息，订阅者必须保持运行的状态。

## 消息队列选型

广泛来说，电商、金融等对事务性要求很高的，可以考虑RocketMQ，技术挑战不是特别高，用 RabbitMQ 是不错的选择，如果是大数据领域的实时计算、日志采集等场景可以考虑 Kafka。

## ActiveMQ的POM文件

```
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-broker</artifactId>
            <version>5.14.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-pool</artifactId>
            <version>5.14.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-spring</artifactId>
            <version>5.14.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>artemis-jms-client</artifactId>
            <version>2.3.0</version>
        </dependency>
```

## 源码分析-Broker源码

```
public class BrokerServiceTest {

    public static void main(String[] args) throws Exception {
        // 配置管理broker、connector
        BrokerService brokerService = new BrokerService();
        // 配置----------------------------------------------------------
        // 设定broker绑定的address地址
        String bindAddress = "tcp://0.0.0.0:61616";
        brokerService.setBrokerName("MyBroker");
        // 添加连接器
        // 1、将根据bindAddress的schema构建connector连接器，构建TransportServer（如果是Http，那么使用Jetty的connector实现）
        brokerService.addConnector(bindAddress);
        // broker是否持久化消息


        // 1、true表示开启持久化
        brokerService.setPersistent(true);

//        brokerService.setDeleteAllMessagesOnStartup(true);
//        brokerService.getSystemUsage().getMemoryUsage().setLimit(1024L * 1024 * 64);
//        KahaDBPersistenceAdapter persistenceAdapter = new KahaDBPersistenceAdapter();
//        persistenceAdapter.setDirectory(new File("kahadb"));
//        brokerService.setPersistenceAdapter(persistenceAdapter);

        // 添加插件机制
        BrokerPlugin messageLogPlugin=new MessageLogPlugin();
        brokerService.setPlugins(new BrokerPlugin[]{messageLogPlugin});

        // 是否使用jmx
        brokerService.setUseJmx(false);
        // 允许警报降级
        brokerService.setAdvisorySupport(false);
        // 启动-----------------------------------------------------------
        // 启动brokerService
        // 创建了Broker的实现类RegionBroker，其中包含了QueueRegion和TopicRegion等region实现。
        //service方法处理command，比如发送Message将调用broker的send，
        // broker将找到对应的region -> destination -> message store -> addMessage
        // 这样一个链路去处理
        // dispatchSync分发消息
        brokerService.start();
        // 等待结束-------------------------------------------------------
        // 如何从brokerService获取监控信息
//        while (true){
//            ActiveMQDestination[] destinations = brokerService.getDestinations();
//            brokerService.getBrokerName();
//            brokerService.getAdminView();
//        }
        // 主线程await，直到brokerService状态是stop
        brokerService.waitUntilStopped();
    }
}
```

自上而下有三部分组成：

1、connector：连接器

2、region：主要分为topic和queue，分别处理相关内容

3、message store：存储

## 源码分析-插件机制

MessageLogPlugin

```
import org.apache.activemq.broker.Broker;
import org.apache.activemq.broker.BrokerPlugin;

public class MessageLogPlugin implements BrokerPlugin {
    @Override
    public Broker installPlugin(Broker broker) throws Exception {
        return new MessageLogFilter(broker);
    }
}
```

MessageLogFilter

```
import org.apache.activemq.broker.Broker;
import org.apache.activemq.broker.BrokerFilter;
import org.apache.activemq.broker.ProducerBrokerExchange;
import org.apache.activemq.command.Message;

public class MessageLogFilter extends BrokerFilter {

    public MessageLogFilter(Broker next) {
        super(next);
    }

    @Override
    public void send(ProducerBrokerExchange producerExchange, Message message) throws Exception {
        System.out.println("message======:{}"+message.getMessageId());
        super.send(producerExchange,message);
    }
}
```

源码分析-存储

KahaDBTest

```
import org.apache.activemq.store.kahadb.disk.journal.Journal;
import org.apache.activemq.store.kahadb.disk.journal.Location;
import org.apache.activemq.util.ByteSequence;

import java.io.File;
import java.io.IOException;

/**
 * hakadb是activemq的持久化数据库，作为消息队列的存储，
 * 每个消息有一个消息ID，提供了对消息的快速的查找，更新，以
 * 及消息的事物支持，以及意外磬机之后的恢复。
 * 丰富的功能决定了他在存储结构上与redis中简单的双端队列不同。
 */

/**
 * 主要由索引(B-Tree Index)，数据(PageFile)，日志(Journal)组成。
 * 为了实现消息的快速定位更新，通过b-tree来索引数据，主要对应目录中的；
 * 为了数据的写入效率，数据都是顺序存储在文件中，
 * 这部分工作被抽象到了PageFile类；
 * Journal负责了操作日志，用于写入和恢复最终都是决定与Journal
 */
public class KahaDBTest {

    public static void main(String[] args) throws IOException {
        File dir = new File("target/tests/Test");
        dir.mkdirs();
        Journal dataManager = new Journal();
        dataManager.setDirectory(dir);
        dataManager.start();
        //写入日志
        ByteSequence write = new ByteSequence("Hello world".getBytes());
        Location location = dataManager.write(write, true);
        //读取日志
        ByteSequence read = dataManager.read(location);
        System.out.println(new String(read.data));
    }
}

/**
 * 核心类
 *
 * Journal 读写日志的facade
 * write方法，向日志系统写入日志。返回Location。
 * read方法，根据Location从之日系统中读取日志
 *
 * Location 日志系统中定位日志Record
 * 由日志的DataFileId,offset,size,type组成，用来定位日志
 *
 * DataFile 日志文件的存储文件
 * 日志都存储在db-{1,2,3,...n}.log文件中，每个这个文件都由DataFile类来封装，其中1,2,3..就是DataFileId，还有一些长度和文件指针的信息
 *
 *
 * DataFileAppender 多线程下优化了批量写入
 * 两个线程，一个队列，把并发写变成批量单线程写入，具体实现略
 *
 * DataFileAccessorPool和DataFileAccessor
 * 封装了对日志的读操作，暂时略，对理解整体结构没有大的影响。
 */

/**
 * 存储结构
 * 存储的时候包括{dbname}.data和{dbname}.redo {dbname}.free文件。
 * data文件是保存真正消息的文件。
 * 当data文件META信息中的cleanShutdown为false的时候会通过redo文件来做recovery。
 *
 */
```



# ActiveMQ broker解析

在 ActiveMQ 中，broker 集中管理持久的、临时的 queue 和 topic。

```
public class BrokerFilter implements Broker {
    // 省略其他代码
    protected final Broker next;
}
```

最终的 Broker 链是这样的：

StatisticsBroker -> TransactionBroker -> CompositeDestinationBroker -> AdvisoryBroker -> ManagedRegionBroker

创建Broker 链：

```
// org.apache.activemq.broker.BrokerService
protected Broker createBroker() throws Exception {
    // 创建 RegionBroker
    regionBroker = createRegionBroker();
    // 添加拦截器
    Broker broker = addInterceptors(regionBroker);
    // Add a filter that will stop access to the broker once stopped
    broker = new MutableBrokerFilter(broker) {
        Broker old;

        @Override
        public void stop() throws Exception {
            old = this.next.getAndSet(new ErrorBroker("Broker has been stopped: " + this) {
                // Just ignore additional stop actions.
                @Override
                public void stop() throws Exception {
                }
            });
            old.stop();
        }

        @Override
        public void start() throws Exception {
            if (forceStart && old != null) {
                this.next.set(old);
            }
            getNext().start();
        }
    };
    return broker;
}
```

创建 RegionBroker：

```
protected Broker createRegionBroker(DestinationInterceptor destinationInterceptor) throws IOException {
    RegionBroker regionBroker;
    if (isUseJmx()) {
        try {
            regionBroker = new ManagedRegionBroker(this, getManagementContext(), getBrokerObjectName(),
                getTaskRunnerFactory(), getConsumerSystemUsage(), destinationFactory, destinationInterceptor,getScheduler(),getExecutor());
        } catch(MalformedObjectNameException me){
            LOG.warn("Cannot create ManagedRegionBroker due " + me.getMessage(), me);
            throw new IOException(me);
        }
    } else {
        regionBroker = new RegionBroker(this, getTaskRunnerFactory(), getConsumerSystemUsage(), destinationFactory,
                destinationInterceptor,getScheduler(),getExecutor());
    }
    destinationFactory.setRegionBroker(regionBroker);
    regionBroker.setKeepDurableSubsActive(keepDurableSubsActive);
    regionBroker.setBrokerName(getBrokerName());
    regionBroker.getDestinationStatistics().setEnabled(enableStatistics);
    regionBroker.setAllowTempAutoCreationOnSend(isAllowTempAutoCreationOnSend());
    if (brokerId != null) {
        regionBroker.setBrokerId(brokerId);
    }
    return regionBroker;
}
```

为RegionBroker添加BrokerFilter：

```
protected Broker addInterceptors(Broker broker) throws Exception {
    if (isSchedulerSupport()) {
        SchedulerBroker sb = new SchedulerBroker(this, broker, getJobSchedulerStore());
        if (isUseJmx()) {
            JobSchedulerViewMBean view = new JobSchedulerView(sb.getJobScheduler());
            try {
                ObjectName objectName = BrokerMBeanSupport.createJobSchedulerServiceName(getBrokerObjectName());
                AnnotatedMBean.registerMBean(getManagementContext(), view, objectName);
                this.adminView.setJMSJobScheduler(objectName);
            } catch (Throwable e) {
                throw IOExceptionSupport.create("JobScheduler could not be registered in JMX: "
                        + e.getMessage(), e);
            }
        }
        broker = sb;
    }
    if (isUseJmx()) {
        HealthViewMBean statusView = new HealthView((ManagedRegionBroker)getRegionBroker());
        try {
            ObjectName objectName = BrokerMBeanSupport.createHealthServiceName(getBrokerObjectName());
            AnnotatedMBean.registerMBean(getManagementContext(), statusView, objectName);
        } catch (Throwable e) {
            throw IOExceptionSupport.create("Status MBean could not be registered in JMX: "
                    + e.getMessage(), e);
        }
    }
    if (isAdvisorySupport()) {
        broker = new AdvisoryBroker(broker);
    }
    broker = new CompositeDestinationBroker(broker);
    broker = new TransactionBroker(broker, getPersistenceAdapter().createTransactionStore());
    if (isPopulateJMSXUserID()) {
        UserIDBroker userIDBroker = new UserIDBroker(broker);
        userIDBroker.setUseAuthenticatePrincipal(isUseAuthenticatedPrincipalForJMSXUserID());
        broker = userIDBroker;
    }
    if (isMonitorConnectionSplits()) {
        broker = new ConnectionSplitBroker(broker);
    }
    if (plugins != null) {
        for (int i = 0; i < plugins.length; i++) {
            BrokerPlugin plugin = plugins[i];
            broker = plugin.installPlugin(broker);
        }
    }
    return broker;
}
```

 我们创建的topic和queue都记录在ManagedRegionBroker这个类中：

```
public class ManagedRegionBroker extends RegionBroker {
    private static final Logger LOG = LoggerFactory.getLogger(ManagedRegionBroker.class);
    private final ManagementContext managementContext;
    private final ObjectName brokerObjectName;
    private final Map<ObjectName, DestinationView> topics 
        = new ConcurrentHashMap<ObjectName, DestinationView>();
    private final Map<ObjectName, DestinationView> queues 
        = new ConcurrentHashMap<ObjectName, DestinationView>();
    private final Map<ObjectName, DestinationView> temporaryQueues 
        = new ConcurrentHashMap<ObjectName, DestinationView>();
    private final Map<ObjectName, DestinationView> temporaryTopics 
        = new ConcurrentHashMap<ObjectName, DestinationView>();
    private final Map<ObjectName, SubscriptionView> queueSubscribers 
        = new ConcurrentHashMap<ObjectName, SubscriptionView>();
    private final Map<ObjectName, SubscriptionView> topicSubscribers 
        = new ConcurrentHashMap<ObjectName, SubscriptionView>();
    private final Map<ObjectName, SubscriptionView> durableTopicSubscribers 
        = new ConcurrentHashMap<ObjectName, SubscriptionView>();
    private final Map<ObjectName, SubscriptionView> inactiveDurableTopicSubscribers 
        = new ConcurrentHashMap<ObjectName, SubscriptionView>();
    private final Map<ObjectName, SubscriptionView> temporaryQueueSubscribers 
        = new ConcurrentHashMap<ObjectName, SubscriptionView>();
    private final Map<ObjectName, SubscriptionView> temporaryTopicSubscribers 
        = new ConcurrentHashMap<ObjectName, SubscriptionView>();
    private final Map<ObjectName, ProducerView> queueProducers 
        = new ConcurrentHashMap<ObjectName, ProducerView>();
    private final Map<ObjectName, ProducerView> topicProducers 
        = new ConcurrentHashMap<ObjectName, ProducerView>();
    private final Map<ObjectName, ProducerView> temporaryQueueProducers 
        = new ConcurrentHashMap<ObjectName, ProducerView>();
    private final Map<ObjectName, ProducerView> temporaryTopicProducers 
        = new ConcurrentHashMap<ObjectName, ProducerView>();
    private final Map<ObjectName, ProducerView> dynamicDestinationProducers 
        = new ConcurrentHashMap<ObjectName, ProducerView>();
    private final Map<SubscriptionKey, ObjectName> subscriptionKeys 
        = new ConcurrentHashMap<SubscriptionKey, ObjectName>();
    private final Map<Subscription, ObjectName> subscriptionMap 
        = new ConcurrentHashMap<Subscription, ObjectName>();
    private final Set<ObjectName> registeredMBeans 
        = new CopyOnWriteArraySet<ObjectName>();
    /* This is the first broker in the broker interceptor chain. */
    private Broker contextBroker;
}
```

# ActiveMQ Advisory Message

http://activemq.apache.org/advisory-message.html

ActiveMQ broker 内部维持了一些 topic，保存了一些系统信息，客户端可以订阅这些 topic 来获取信息，即 advisory message。

 

列举3个 topic 的例子：

1. topic名：ActiveMQ.Advisory.Connection

消息类型：客户端连接建立和断开的消息

获取方式：

```
AdvisorySupport.getConnectionAdvisoryTopic()
```

2. topic名：ActiveMQ.Advisory.Producer.Queue.XXX

消息类型：XXX队列当前有几个producer，以及producer删除的消息

获取方式：

```
AdvisorySupport.getProducerAdvisoryTopic(Destination destination)
```

3. topic名：ActiveMQ.Advisory.Queue，ActiveMQ.Advisory.Topic， ActiveMQ.Advisory.TempQueue， ActiveMQ.Advisory.TempTopic

消息类型：destination创建和销毁的消息

获取方式：

```
AdvisorySupport.getDestinationAdvisoryTopic(Destination destination)
```

示例代码：

```
public static void main(String[] args) throws JMSException {
    ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
    // Create a Connection
    Connection connection = connectionFactory.createConnection();
    connection.start();

    ActiveMQSession session = (ActiveMQSession) connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    // Create the destination (Topic or Queue)
    Queue queue = session.createQueue("TEST.FOO");
    // topic://ActiveMQ.Advisory.Producer.Queue.TEST.FOO
    ActiveMQTopic topic = AdvisorySupport.getProducerAdvisoryTopic(queue);
    
    MessageConsumer consumer = session.createConsumer(topic);
    consumer.setMessageListener(new MessageListener() {
        public void onMessage(Message msg) {
            if(msg instanceof ActiveMQMessage) {
                System.out.println(msg);
            }
        }
    });
             
}
```

# 图解ActiveMQ virtual topic

普通的 topic 是发布/订阅模式：消息会被广播发送给所有的订阅者，订阅者拿到的是全部消息，如下图：

![img](https://images2018.cnblogs.com/blog/461546/201803/461546-20180329154252178-888277050.png)

而 virtual topic，在消息的传递过程中，多加了一个队列节点，如下图：

![img](https://images2018.cnblogs.com/blog/461546/201803/461546-20180329154410669-1970734820.png)

全量的消息先发送到队列，然后再分发给消费者。这么做有什么好处呢？

假定consumer1和consumer2分别是2个进程，2个进程共同处理消息，这算不算负载均衡呢？

其次，如果consumer1挂掉了，队列的消息还能发送给consumer2，这是不是failover呢？

示例代码：

producer

```
public static void main(String[] args) throws JMSException {
    ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");  
    Connection connection = factory.createConnection();  
    connection.start();  
    Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);  
    
    // 创建virtual topic，前缀必须是"VirtualTopic."，当然这是可配置的
    Topic topic = session.createTopic("VirtualTopic.bank"); 
    MessageProducer producer = session.createProducer(topic);  
    producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
    
    for (int i = 0; i < 1; i++) {
        TextMessage message = session.createTextMessage();  
        message.setText("hello zhang");  
        // 发布主题消息  
        producer.send(message);  
        System.out.println("Sent message: " + message.getText());  
    }
    
    session.close();  
    connection.close();  
}
```

consumer

```
public static void main(String[] args) throws JMSException {
    ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");  
    ActiveMQConnection connection = (ActiveMQConnection) factory.createConnection();  
    connection.start(); 
    
    Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);  
    // 根据 virtual topic 创建队列。格式为 "Consumer.*.VirtualTopic.>"
    Queue queueA = session.createQueue("Consumer.A.VirtualTopic.bank");  
    Queue queueB = session.createQueue("Consumer.B.VirtualTopic.bank");  
    
    // 队列A创建订阅  
    MessageConsumer consumerA1 = session.createConsumer(queueA); 
    consumerA1.setMessageListener(new MessageListener() {  
        public void onMessage(Message message) {  
            TextMessage tm = (TextMessage) message;  
            System.out.println("A1: " + tm); 
        }  
    });     
    MessageConsumer consumerA2 = session.createConsumer(queueA);  
    consumerA2.setMessageListener(new MessageListener() {  
        public void onMessage(Message message) {  
            TextMessage tm = (TextMessage) message;  
            System.out.println("A2: " + tm); 
        }  
    });  
      
    // 队列B创建订阅
    MessageConsumer consumerB1 = session.createConsumer(queueB);  
    consumerB1.setMessageListener(new MessageListener() {  
        public void onMessage(Message message) {  
            TextMessage tm = (TextMessage) message;  
            System.out.println("B1: " + tm);
        }  
    });  
    MessageConsumer consumerB2 = session.createConsumer(queueB);  
    consumerB2.setMessageListener(new MessageListener() {  
        public void onMessage(Message message) {  
            TextMessage tm = (TextMessage) message;  
            System.out.println("B2: " + tm); 
        }  
    });
    
    // session.close(); 
    // connection.close();  
}
```

上面只是demo，正常情况下，consumer应该在单独的进程中。

# ActiveMQ queue 分页

分页：即获取部分数据，queue按页从message cursor读取消息，然后分发给consumer。

页大小：

```
public abstract class BaseDestination implements Destination {
    /**
     * The maximum number of messages to page in to the destination from
     * persistent storage
     */
    public static final int MAX_PAGE_SIZE = 200;
}
```

存放分页消息的数据结构：

```
public class Queue extends BaseDestination implements Task, UsageListener {
    // message cursor，可视为消息的数据源
    protected PendingMessageCursor messages;
    // 所有的分页消息
    private final PendingList pagedInMessages = new OrderedPendingList();
    // 剩余的没有dispatch的分页消息
    protected PendingList pagedInPendingDispatch = new OrderedPendingList();
}
```

把消息添加到分页中：

```
protected void pageInMessages(boolean force) throws Exception {
    doDispatch(doPageInForDispatch(force, true));
}
```



```
  1 private PendingList doPageInForDispatch(boolean force, boolean processExpired) throws Exception {
  2     List<QueueMessageReference> result = null;
  3     PendingList resultList = null;
  4 
  5     // 根据maxPageSize和message cursor中的大小，决定需要读取的消息数量
  6     int toPageIn = Math.min(getMaxPageSize(), messages.size());
  7     int pagedInPendingSize = 0;
  8     pagedInPendingDispatchLock.readLock().lock();
  9     try {
 10         pagedInPendingSize = pagedInPendingDispatch.size();
 11     } finally {
 12         pagedInPendingDispatchLock.readLock().unlock();
 13     }
 14 
 15     LOG.debug("{} toPageIn: {}, Inflight: {}, pagedInMessages.size {}, pagedInPendingDispatch.size {}, enqueueCount: {}, dequeueCount: {}, memUsage:{}",
 16             new Object[]{
 17                     destination.getPhysicalName(),
 18                     toPageIn,
 19                     destinationStatistics.getInflight().getCount(),
 20                     pagedInMessages.size(),
 21                     pagedInPendingSize,
 22                     destinationStatistics.getEnqueues().getCount(),
 23                     destinationStatistics.getDequeues().getCount(),
 24                     getMemoryUsage().getUsage()
 25             });
 26     if (isLazyDispatch() && !force) {
 27         // Only page in the minimum number of messages which can be
 28         // dispatched immediately.
 29         toPageIn = Math.min(getConsumerMessageCountBeforeFull(), toPageIn);
 30     }
 31     if (toPageIn > 0 && (force || (!consumers.isEmpty() && pagedInPendingSize < getMaxPageSize()))) {
 32         int count = 0;
 33         result = new ArrayList<QueueMessageReference>(toPageIn);
 34         messagesLock.writeLock().lock();
 35         try {
 36             try {
 37                 messages.setMaxBatchSize(toPageIn);
 38                 messages.reset();
 39                 while (messages.hasNext() && count < toPageIn) {
 40                     MessageReference node = messages.next();
 41                     messages.remove();
 42 
 43                     QueueMessageReference ref = createMessageReference(node.getMessage());
 44                     if (processExpired && ref.isExpired()) {
 45                         if (broker.isExpired(ref)) {
 46                             messageExpired(createConnectionContext(), ref);
 47                         } else {
 48                             ref.decrementReferenceCount();
 49                         }
 50                     } else {
 51                         // 添加QueueMessageReference到result中
 52                         result.add(ref);
 53                         count++;
 54                     }
 55                 }
 56             } finally {
 57                 messages.release();
 58             }
 59         } finally {
 60             messagesLock.writeLock().unlock();
 61         }
 62         // Only add new messages, not already pagedIn to avoid multiple
 63         // dispatch attempts
 64         pagedInMessagesLock.writeLock().lock();
 65         try {
 66             if(isPrioritizedMessages()) {
 67                 resultList = new PrioritizedPendingList();
 68             } else {
 69                 resultList = new OrderedPendingList();
 70             }
 71             for (QueueMessageReference ref : result) {
 72                 if (!pagedInMessages.contains(ref)) {
 73                     //分别添加QueueMessageReference到 pagedInMessages 和 resultList
 74                     //resultList作为返回值，直接传递给doDispatch(PendingList list)，
 75                     //在doDispatch中，分发给消费者后，就会从 resultList 中删除，
 76                     pagedInMessages.addMessageLast(ref);
 77                     resultList.addMessageLast(ref);
 78                 } else {
 79                     ref.decrementReferenceCount();
 80                     // store should have trapped duplicate in it's index, also cursor audit
 81                     // we need to remove the duplicate from the store in the knowledge that the original message may be inflight
 82                     // note: jdbc store will not trap unacked messages as a duplicate b/c it gives each message a unique sequence id
 83                     LOG.warn("{}, duplicate message {} paged in, is cursor audit disabled? Removing from store and redirecting to dlq", this, ref.getMessage());
 84                     if (store != null) {
 85                         ConnectionContext connectionContext = createConnectionContext();
 86                         store.removeMessage(connectionContext, new MessageAck(ref.getMessage(), MessageAck.POSION_ACK_TYPE, 1));
 87                         broker.getRoot().sendToDeadLetterQueue(connectionContext, ref.getMessage(), null, new Throwable("duplicate paged in from store for " + destination));
 88                     }
 89                 }
 90             }
 91         } finally {
 92             pagedInMessagesLock.writeLock().unlock();
 93         }
 94     } else {
 95         // Avoid return null list, if condition is not validated
 96         resultList = new OrderedPendingList();
 97     }
 98 
 99     return resultList;
100 }
101 
102 //分发消息
103 private void doDispatch(PendingList list) throws Exception {
104     boolean doWakeUp = false;
105 
106     pagedInPendingDispatchLock.writeLock().lock();
107     try {
108         //存在需要重新发送的消息
109         if (!redeliveredWaitingDispatch.isEmpty()) {
110             // Try first to dispatch redelivered messages to keep an
111             // proper order
112             redeliveredWaitingDispatch = doActualDispatch(redeliveredWaitingDispatch);
113         }
114         //存在没有分发的分页消息
115         if (!pagedInPendingDispatch.isEmpty()) {
116             // Next dispatch anything that had not been
117             // dispatched before.
118             pagedInPendingDispatch = doActualDispatch(pagedInPendingDispatch);
119         }
120         // and now see if we can dispatch the new stuff.. and append to
121         // the pending
122         // list anything that does not actually get dispatched.
123         if (list != null && !list.isEmpty()) {
124             if (pagedInPendingDispatch.isEmpty()) {
125                 //doActualDispatch进行实际的分发消息：
126                 //分发给消费者的消息，会从list中删除，list中保存剩下的消息，doActualDispatch返回list
127                 pagedInPendingDispatch.addAll(doActualDispatch(list));
128             } else {
129                 for (MessageReference qmr : list) {
130                     if (!pagedInPendingDispatch.contains(qmr)) {
131                         pagedInPendingDispatch.addMessageLast(qmr);
132                     }
133                 }
134                 doWakeUp = true;
135             }
136         }
137     } finally {
138         pagedInPendingDispatchLock.writeLock().unlock();
139     }
140 
141     if (doWakeUp) {
142         // avoid lock order contention
143         asyncWakeup();
144     }
145 }
146 
147 // 实际分发消息
148 private PendingList doActualDispatch(PendingList list) throws Exception {
149     List<Subscription> consumers;
150     consumersLock.writeLock().lock();
151 
152     try {
153         if (this.consumers.isEmpty()) {
154             // slave dispatch happens in processDispatchNotification
155             return list;
156         }
157         consumers = new ArrayList<Subscription>(this.consumers);
158     } finally {
159         consumersLock.writeLock().unlock();
160     }
161 
162     Set<Subscription> fullConsumers = new HashSet<Subscription>(this.consumers.size());
163 
164     for (Iterator<MessageReference> iterator = list.iterator(); iterator.hasNext();) {
165 
166         MessageReference node = iterator.next();
167         Subscription target = null;
168         for (Subscription s : consumers) {
169             if (s instanceof QueueBrowserSubscription) {
170                 continue;
171             }
172             if (!fullConsumers.contains(s)) {
173                 if (!s.isFull()) {
174                     if (dispatchSelector.canSelect(s, node) && assignMessageGroup(s, (QueueMessageReference)node) && !((QueueMessageReference) node).isAcked() ) {
175                         // Dispatch it.
176                         s.add(node);
177                         LOG.trace("assigned {} to consumer {}", node.getMessageId(), s.getConsumerInfo().getConsumerId());
178                         //从list中删除
179                         iterator.remove();
180                         target = s;
181                         break;
182                     }
183                 } else {
184                     // no further dispatch of list to a full consumer to
185                     // avoid out of order message receipt
186                     fullConsumers.add(s);
187                     LOG.trace("Subscription full {}", s);
188                 }
189             }
190         }
191 
192         if (target == null && node.isDropped()) {
193             iterator.remove();
194         }
195 
196         // return if there are no consumers or all consumers are full
197         if (target == null && consumers.size() == fullConsumers.size()) {
198             return list;
199         }
200 
201         // If it got dispatched, rotate the consumer list to get round robin
202         // distribution.
203         if (target != null && !strictOrderDispatch && consumers.size() > 1
204                 && !dispatchSelector.isExclusiveConsumer(target)) {
205             consumersLock.writeLock().lock();
206             try {
207                 if (removeFromConsumerList(target)) {
208                     addToConsumerList(target);
209                     consumers = new ArrayList<Subscription>(this.consumers);
210                 }
211             } finally {
212                 consumersLock.writeLock().unlock();
213             }
214         }
215     }
216 
217     return list;
218 }
```

# ActiveMQ Message Groups

http://activemq.apache.org/message-groups.html

与Exclusive Consumer相比，Message Groups的对消息分组的粒度更细。具有相同groupId的消息会被投送到同一个消费者，除非这个消费者挂了。

代码示例：

```
Mesasge message = session.createTextMessage("<foo>hey</foo>");
// 设置groupId
message.setStringProperty("JMSXGroupID", "IBM_NASDAQ_20/4/05");
// 设置sequence
message.setIntProperty("JMSXGroupSeq", -1);

producer.send(message);
```

对应的代码在 org.apache.activemq.broker.region.Queue 中：

```
// 判断消息能否分发给消费者，返回true表示可以
// Subscription 表示消费者，QueueMessageReference 表示消息
protected boolean assignMessageGroup(Subscription subscription, QueueMessageReference node) 
            throws Exception {
    // 默认为true
    boolean result = true;
    // Keep message groups together.
    // 获取消息的"JMSXGroupID"属性
    String groupId = node.getGroupID();
    // 获取消息的"JMSXGroupSeq"属性
    int sequence = node.getGroupSequence();
    if (groupId != null) {
        // MessageGroupMap是一个Map，键是groupId，值是消费者
        MessageGroupMap messageGroupOwners = getMessageGroupOwners();
        // If we can own the first, then no-one else should own the
        // rest.
        if (sequence == 1) {
            assignGroup(subscription, messageGroupOwners, node, groupId);
        } else {

            // Make sure that the previous owner is still valid, we may
            // need to become the new owner.
            ConsumerId groupOwner;
            // 根据groupId取出消费者
            groupOwner = messageGroupOwners.get(groupId);
            if (groupOwner == null) {
                assignGroup(subscription, messageGroupOwners, node, groupId);
            } else {
                if (groupOwner.equals(subscription.getConsumerInfo().getConsumerId())) {
                    // A group sequence < 1 is an end of group signal.
                    if (sequence < 0) {
                        messageGroupOwners.removeGroup(groupId);
                        subscription.getConsumerInfo().
                        setLastDeliveredSequenceId(subscription.getConsumerInfo().getLastDeliveredSequenceId() - 1);
                    }
                } else {
                    result = false;
                }
            }
        }
    }

    return result;
}

// 往MessageGroupMap中插入键值对
protected void assignGroup(Subscription subs, MessageGroupMap messageGroupOwners, 
                        MessageReference n, String groupId) throws IOException {
    messageGroupOwners.put(groupId, subs.getConsumerInfo().getConsumerId());
    Message message = n.getMessage();
    message.setJMSXGroupFirstForConsumer(true);
    subs.getConsumerInfo().
        setLastDeliveredSequenceId(subs.getConsumerInfo().getLastDeliveredSequenceId() + 1);
}
```

# ActiveMQ 消息的重新投递

正常情况下：consumer 消费完消息后，会发送"标准确认"给 broker，这个确认对象以 MessageAck 类表征：

```
// 省略其他代码。类中定义了各种确认的类型
public class MessageAck extends BaseCommand {
    public static final byte DATA_STRUCTURE_TYPE = CommandTypes.MESSAGE_ACK;
    public static final byte DELIVERED_ACK_TYPE = 0;
    public static final byte STANDARD_ACK_TYPE = 2;
    public static final byte POSION_ACK_TYPE = 1;
    public static final byte REDELIVERED_ACK_TYPE = 3;
    public static final byte INDIVIDUAL_ACK_TYPE = 4;
    public static final byte UNMATCHED_ACK_TYPE = 5;
    public static final byte EXPIRED_ACK_TYPE = 6;

    protected byte ackType;
    
    // messageCount 表示确认的消息的数量，即 consumer 可以对消息进行批量确认
    public MessageAck(Message message, byte ackType, int messageCount) {
        this.ackType = ackType;
        this.destination = message.getDestination();
        this.lastMessageId = message.getMessageId();
        this.messageCount = messageCount;
    }
}
```

但是当 consumer 处理消息失败时，会怎样呢？例如：发生了除数为 0，抛出异常

```
consumer.setMessageListener(new MessageListener() {
    public void onMessage(Message message) {
        logger.info("Received: " + message);
        float f = 1 / 0;
    }
});
```

consumer 会进行重新投递，重新把消息给 listener 处理。具体流程是：consumer 消费消息失败，抛出异常，回滚，然后重新投递。

```
// void org.apache.activemq.ActiveMQMessageConsumer.rollback() throws JMSException
if (messageListener.get() != null) {
    session.redispatch(this, unconsumedMessages);
}
```

 

下面的代码设置 RedeliveryPolicy：

```
RedeliveryPolicy queuePolicy = new RedeliveryPolicy();
queuePolicy.setInitialRedeliveryDelay(0);
queuePolicy.setRedeliveryDelay(1000);
queuePolicy.setUseExponentialBackOff(false);
queuePolicy.setMaximumRedeliveries(1);
RedeliveryPolicyMap map = connection.getRedeliveryPolicyMap();
map.put(new ActiveMQQueue(">"), queuePolicy);
```

这个重新投递策略是 consumer 端的（consumer 重新投递给自己的消息 listener），而不是 broker 重新投递给 consumer 的，理解这一点特别重要。超过了最大投递次数后，consumer 会发送给 broker 一个 POSION_ACK_TYPE 类型的 MessageAck 响应，正常情况是 STANDARD_ACK_TYPE 类型的。

 

consumer发送正常消息确认的调用栈：

![img](https://images2018.cnblogs.com/blog/461546/201803/461546-20180330140658780-1265422128.png)

主要逻辑在：

```
// void org.apache.activemq.ActiveMQMessageConsumer
private void afterMessageIsConsumed(MessageDispatch md, boolean messageExpired) throws JMSException {
    if (unconsumedMessages.isClosed()) {
        return;
    }
    if (messageExpired) {
        // 过期消息
        acknowledge(md, MessageAck.EXPIRED_ACK_TYPE);
        stats.getExpiredMessageCount().increment();
    } else {
        stats.onMessage();
        if (session.getTransacted()) {
            // Do nothing.
        } else if (isAutoAcknowledgeEach()) {
            // 设置为 Session.AUTO_ACKNOWLEDGE
            if (deliveryingAcknowledgements.compareAndSet(false, true)) {
                synchronized (deliveredMessages) {
                    if (!deliveredMessages.isEmpty()) {
                        if (optimizeAcknowledge) {
                            ackCounter++;

                            // AMQ-3956 evaluate both expired and normal msgs as
                            // otherwise consumer may get stalled
                            if (ackCounter + deliveredCounter >= (info.getPrefetchSize() * .65) || (optimizeAcknowledgeTimeOut > 0 && System.currentTimeMillis() >= (optimizeAckTimestamp + optimizeAcknowledgeTimeOut))) {
                                MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);
                                if (ack != null) {
                                    deliveredMessages.clear();
                                    ackCounter = 0;
                                    session.sendAck(ack);
                                    optimizeAckTimestamp = System.currentTimeMillis();
                                }
                                // AMQ-3956 - as further optimization send
                                // ack for expired msgs when there are any.
                                // This resets the deliveredCounter to 0 so that
                                // we won't sent standard acks with every msg just
                                // because the deliveredCounter just below
                                // 0.5 * prefetch as used in ackLater()
                                if (pendingAck != null && deliveredCounter > 0) {
                                    session.sendAck(pendingAck);
                                    pendingAck = null;
                                    deliveredCounter = 0;
                                }
                            }
                        } else {
                            MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);
                            if (ack!=null) {
                                deliveredMessages.clear();
                                session.sendAck(ack);
                            }
                        }
                    }
                }
                deliveryingAcknowledgements.set(false);
            }
        } else if (isAutoAcknowledgeBatch()) {
            ackLater(md, MessageAck.STANDARD_ACK_TYPE);
        } else if (session.isClientAcknowledge()||session.isIndividualAcknowledge()) {
            boolean messageUnackedByConsumer = false;
            synchronized (deliveredMessages) {
                messageUnackedByConsumer = deliveredMessages.contains(md);
            }
            if (messageUnackedByConsumer) {
                ackLater(md, MessageAck.DELIVERED_ACK_TYPE);
            }
        }
        else {
            throw new IllegalStateException("Invalid session state.");
        }
    }
}
```

consumer 发送有毒消息确认的调用栈：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180402093855816-622593213.png)

 

broker 接收消息确认的调用栈：

![img](https://images2018.cnblogs.com/blog/461546/201803/461546-20180330141038881-1602681897.png)

 

那么，在什么情况下，broker 会重新发送消息给 cosumer 呢？答案是：

broker 把一个消息推送给 consumer 后，但是还没收到任何确认，如果这时消费者断开连接，broker 会把这个消息加入到重新投递队列中，推送给其他的消费者。

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180402094952361-820760933.png)

#  ActiveMQ producer 流量控制

http://activemq.apache.org/producer-flow-control.html

翻译：

流量控制是指：如果broker检测到destination的内存限制、temp文件限制、file store限制被超过了，就会减慢消息的流动。producer会被阻塞直到有可用资源，或者收到一个JMSException：这些行为是可以配置的。

值得一提的是：默认的<systemUsage>设置会导致producer阻塞，当达到memoryLimit或<systemUsage>的限制值时，这种阻塞行为有时被误认为是一个"挂起的producer"，而事实上producer只是在等待可用空间。

同步发送的消息会自动使用按producer的流量控制；通常会应用到同步发送持久化消息，除非你开启useAsyncSend标志。

使用异步发送的producer（通常讲，发送非持久化消息的producer）不需要等待来自broker的确认；所以，当达到memory limit时，你不会被通知到。如果你想知道broker的限制值达到了，你需要配置 ProducerWindowSize 连接参数，然后异步的消息也能按producer控制流量了。

```
ActiveMQConnectionFactory connctionFactory = ... 
connctionFactory.setProducerWindowSize(1024000);
```

 

 ProducerWindowSize 是producer发送的最大字节数，在等待来自broker的消息确认前。 如果你在发送非持久化消息（默认异步发送），并且希望知道queue或topic的memory limit是否达到了。那么你需要设置connection factory 为 'alwaysSyncSend'。但是，这会降低速度，它能保证你的producer马上知道内存问题。 如果你喜欢，你可以关掉指定jms queue和topic的流量控制，例如：

```
<destinationPolicy>  
    <policyMap>    
        <policyEntries>      
            <policyEntry topic="FOO.>" producerFlowControl="false"/> 
            </policyEntries>  
    </policyMap>
</destinationPolicy>
```

注意，在ActiveMQ 5.x中引入了新的file cursor，非持久化消息会被刷到临时文件存储中来减少内存使用量。所以，你会发现queue的memoryLimit永远达不到，因为file cursor花不了多少内存，如果你真的要把所有非持久化消息保存在内存中，并且当memoryLimit达到时停止producer，你应该配置<vmQueueCursor>。

```
<policyEntry queue=">" producerFlowControl="true" memoryLimit="1mb">      
    <pendingQueuePolicy>    
        <vmQueueCursor/>  
    </pendingQueuePolicy>
</policyEntry>
```

上面的片段能保证，所有的消息保存在内存中，并且每一个队列只有1Mb的限制。

How Producer Flow Control works

如果你在发送持久化消息，broker会发送一个ProducerAck消息给producer，它告知producer前一个发送窗口已经被处理了，所以producer现在可以发送下一个窗口。这和consumer的消息确认很像。当没有空间可用时，调用send()操作会无限阻塞，另一种方法是在客户端抛出异常。通过设置sendFailIfNoSpace为true，broker会导致send()抛出javax.jms.ResourceAllocationException，异常会传播到客户端。下面是配置示例：

```
<systemUsage> 
    <systemUsage sendFailIfNoSpace="true">   
        <memoryUsage>     
            <memoryUsage limit="20 mb"/>   
        </memoryUsage> 
    </systemUsage>
</systemUsage>
```



 这种做法的好处是，客户端会捕获一个javax.jms.ResourceAllocationException异常，等一会然后重试send()，而不再是无限地等待。

 

从5.3.1开始，加入了sendFailIfNoSpaceAfterTimeout属性，如果broker在配置的时间内仍然没有空余空间，此时send()才会失败，并且把异常传递到客户端，下面是配置：

```
<systemUsage> 
    <systemUsage sendFailIfNoSpaceAfterTimeout="3000">   
        <memoryUsage>     
            <memoryUsage limit="20 mb"/>   
        </memoryUsage> 
    </systemUsage>
</systemUsage>
```



 

关闭流量控制

通常的需求是关闭流量控制，这样消息分发可以一直进行直到磁盘空间被 pending messages 耗尽。

通过配置<systemUsage>元素的某些属性，你可以降低producer的速率。

```
<systemUsage>  
    <systemUsage>    
        <memoryUsage>      
            <memoryUsage limit="64 mb" />    
        </memoryUsage>    
        <storeUsage>      
            <storeUsage limit="100 gb" />    
        </storeUsage>    
        <tempUsage>      
            <tempUsage limit="10 gb" />    
        </tempUsage>  
    </systemUsage>
</systemUsage>
```

<memoryUsage>对应NON_PERSISTENT消息的内存容量，<storeUsage> 对应PERSITENT消息的磁盘容量，<tempUsage>对应临时文件的磁盘容量。

 

结合代码分析：

client-side： org.apache.activemq.ActiveMQMessageProducer.send

broker-side：从 org.apache.activemq.broker.region.Queue.send 开始

```
//org.apache.activemq.broker.region.Queue
public void send(final ProducerBrokerExchange producerExchange, final Message message) throws Exception {
    final ConnectionContext context = producerExchange.getConnectionContext();
    // There is delay between the client sending it and it arriving at the
    // destination.. it may have expired.
    message.setRegionDestination(this);
    ProducerState state = producerExchange.getProducerState();
    if (state == null) {
        LOG.warn("Send failed for: {}, missing producer state for: {}", message, producerExchange);
        throw new JMSException("Cannot send message to " + getActiveMQDestination() + " with invalid (null) producer state");
    }
    final ProducerInfo producerInfo = producerExchange.getProducerState().getInfo();
    //是否发送ProducerAck
    final boolean sendProducerAck = !message.isResponseRequired() && producerInfo.getWindowSize() > 0
            && !context.isInRecoveryMode();
    if (message.isExpired()) {
        // message not stored - or added to stats yet - so check here
        broker.getRoot().messageExpired(context, message, null);
        if (sendProducerAck) {
            ProducerAck ack = new ProducerAck(producerInfo.getProducerId(), message.getSize());
            context.getConnection().dispatchAsync(ack);
        }
        return;
    }
    if (memoryUsage.isFull()) { //如果内存耗尽
        // 尽管这里有大段代码，但是调试没进这儿
    }
    doMessageSend(producerExchange, message);
    if (sendProducerAck) {
        ProducerAck ack = new ProducerAck(producerInfo.getProducerId(), message.getSize());
        context.getConnection().dispatchAsync(ack);
    }
}
```

把消息放进 broker 的 pendingList 之前，会检查可用空间：

```
//org.apache.activemq.broker.region.Queue
void doMessageSend(final ProducerBrokerExchange producerExchange, final Message message) 
    throws IOException,    Exception {
    final ConnectionContext context = producerExchange.getConnectionContext();
    ListenableFuture<Object> result = null;
    boolean needsOrderingWithTransactions = context.isInTransaction();

    producerExchange.incrementSend();
    //检查使用空间
    checkUsage(context, producerExchange, message);
    sendLock.lockInterruptibly();
    try {
        if (store != null && message.isPersistent()) {
            try {
                message.getMessageId().setBrokerSequenceId(getDestinationSequenceId());
                if (messages.isCacheEnabled()) {
                    result = store.asyncAddQueueMessage(context, message, isOptimizeStorage());
                    result.addListener(new PendingMarshalUsageTracker(message));
                } else {
                    store.addMessage(context, message);
                }
                if (isReduceMemoryFootprint()) {
                    message.clearMarshalledState();
                }
            } catch (Exception e) {
                // we may have a store in inconsistent state, so reset the cursor
                // before restarting normal broker operations
                resetNeeded = true;
                throw e;
            }
        }
        // did a transaction commit beat us to the index?
        synchronized (orderIndexUpdates) {
            needsOrderingWithTransactions |= !orderIndexUpdates.isEmpty();
        }
        if (needsOrderingWithTransactions ) {
            // If this is a transacted message.. increase the usage now so that
            // a big TX does not blow up
            // our memory. This increment is decremented once the tx finishes..
            message.incrementReferenceCount();

            registerSendSync(message, context);
        } else {
            // Add to the pending list, this takes care of incrementing the
            // usage manager.
            sendMessage(message);
        }
    } finally {
        sendLock.unlock();
    }
    if (!needsOrderingWithTransactions) {
        messageSent(context, message);
    }
    if (result != null && message.isResponseRequired() && !result.isCancelled()) {
        try {
            result.get();
        } catch (CancellationException e) {
            // ignore - the task has been cancelled if the message
            // has already been deleted
        }
    }
}
```

对持久化消息和非持久化消息分类检查：

```
// org.apache.activemq.broker.region.Queue
private void checkUsage(ConnectionContext context,ProducerBrokerExchange producerBrokerExchange, Message message) 
    throws ResourceAllocationException, IOException, InterruptedException {
    if (message.isPersistent()) { // 持久化消息
        if (store != null && systemUsage.getStoreUsage().isFull(getStoreUsageHighWaterMark())) {
            final String logMessage = "Persistent store is Full, " + getStoreUsageHighWaterMark() + "% of "
                + systemUsage.getStoreUsage().getLimit() + ". Stopping producer ("
                + message.getProducerId() + ") to prevent flooding "
                + getActiveMQDestination().getQualifiedName() + "."
                + " See http://activemq.apache.org/producer-flow-control.html for more info";

            waitForSpace(context, producerBrokerExchange, systemUsage.getStoreUsage(), getStoreUsageHighWaterMark(), logMessage);
        }
    } else if (messages.getSystemUsage() != null && systemUsage.getTempUsage().isFull()) {
        // 非持久化消息
        final String logMessage = "Temp Store is Full ("
                + systemUsage.getTempUsage().getPercentUsage() + "% of " + systemUsage.getTempUsage().getLimit()
                +"). Stopping producer (" + message.getProducerId()
            + ") to prevent flooding " + getActiveMQDestination().getQualifiedName() + "."
            + " See http://activemq.apache.org/producer-flow-control.html for more info";

        waitForSpace(context, producerBrokerExchange, messages.getSystemUsage().getTempUsage(), logMessage);
    }
}
```

最后进入具体的处理逻辑：

```
// org.apache.activemq.broker.region.BaseDestination
protected final void waitForSpace(ConnectionContext context, ProducerBrokerExchange producerBrokerExchange, 
        Usage<?> usage, int highWaterMark, String warning) 
        throws IOException, InterruptedException, ResourceAllocationException {
    // 如果配置了sendFailIfNoSpace="true"
    if (!context.isNetworkConnection() && systemUsage.isSendFailIfNoSpace()) {
        getLog().debug("sendFailIfNoSpace, forcing exception on send, usage: {}: {}", usage, warning);
        throw new ResourceAllocationException(warning);
    }
    if (!context.isNetworkConnection() && systemUsage.getSendFailIfNoSpaceAfterTimeout() != 0) {
        if (!usage.waitForSpace(systemUsage.getSendFailIfNoSpaceAfterTimeout(), highWaterMark)) {
            getLog().debug("sendFailIfNoSpaceAfterTimeout expired, forcing exception on send, usage: {}: {}", usage, warning);
            throw new ResourceAllocationException(warning);
        }
    } else {
        long start = System.currentTimeMillis();
        long nextWarn = start;
        producerBrokerExchange.blockingOnFlowControl(true);
        destinationStatistics.getBlockedSends().increment();
        while (!usage.waitForSpace(1000, highWaterMark)) {
            if (context.getStopping().get()) {
                throw new IOException("Connection closed, send aborted.");
            }

            long now = System.currentTimeMillis();
            if (now >= nextWarn) {
                getLog().info("{}: {} (blocking for: {}s)", new Object[]{ usage, warning, new Long(((now - start) / 1000))});
                nextWarn = now + blockedProducerWarningInterval;
            }
        }
        long finish = System.currentTimeMillis();
        long totalTimeBlocked = finish - start;
        destinationStatistics.getBlockedTime().addTime(totalTimeBlocked);
        producerBrokerExchange.incrementTimeBlocked(this,totalTimeBlocked);
        producerBrokerExchange.blockingOnFlowControl(false);
    }
}
```



 

如果配置了sendFailIfNoSpace="true"，并且抛出异常了，处理异常的调用栈如下：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180404145245595-1373888149.png)

```
//org.apache.activemq.broker.TransportConnection
public void serviceException(Throwable e) {
    if (...) {
        ...
    }
    else if (!stopping.get() && !inServiceException) {
        inServiceException = true;
        try {
            SERVICELOG.warn("Async error occurred: ", e);
            ConnectionError ce = new ConnectionError();
            ce.setException(e);
            if (pendingStop) {
                dispatchSync(ce);
            } else {
                dispatchAsync(ce);
            }
        } finally {
            inServiceException = false;
        }
    }
}
```



 

那么 producer 是如何处理 ConnectionError 消息呢？

在org.apache.activemq.ActiveMQConnection.onCommand(Object o) 方法中：



```
public Response processConnectionError(final ConnectionError error) throws Exception {
    executor.execute(new Runnable() {
        @Override
        public void run() {
            onAsyncException(error.getException());
        }
    });
    return null;
}
```

# [ActiveMQ broker和客户端之间的确认](https://www.cnblogs.com/allenwas3/p/8715963.html)

生产者发送消息：producer ---------> broker

broker返回确认：broker ---------> producer

生产者发送同步消息，broker会返回Response；发送异步消息，broker不会返回确认；满足一定条件时，broker会返回ProducerAck:

```
final boolean sendProducerAck = !message.isResponseRequired() && producerInfo.getWindowSize() > 0
                && !context.isInRecoveryMode();
```

 

broker 分发消息：broker ---------> consumer

消费者返回确认： consumer ---------> broker

如果消息被正常处理掉，consumer返回 STANDARD_ACK_TYPE 的 MessageAck，如果消息没有被正常处理，且超过了客户端重新投递次数，consumer则返回 POSION_ACK_TYPE 的 MessageAck。在收到 MessageAck 后，broker 才会删除消息。

通常我们使用 ActiveMQ，会这样创建 Session，设置为自动确认：

```
ActiveMQSession session = (ActiveMQSession) connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```

假定存在队列 TEST.FOO，它有1个消费者consumer1，1 个生产者producer1，当producer1向队列发送1条消息，broker 把这条消息分发给consumer1，如果配置自动确认，consumer进程会自动发送确认，broker收到确认后会删除消息。

反之如果配置为CLIENT_ACKNOWLEDGE，则需要手动确认，即显式调用代码：

```
consumer.acknowledge();
```

如果consumer1收到消息后，并不调用acknowledge()，即不发送消息确认，broker 也一直会保存消息。

 

client 和 broker 之间所有消息都继承自 BaseCommand：例如 ActiveMQTextMessage，ConnectionInfo，KeepAliveInfo，BrokerInfo 等。

# [ActiveMQ 的线程池](https://www.cnblogs.com/allenwas3/p/8760442.html)

ActiveMQ 的线程池实质上也是 ThreadPoolExecutor，不过它的任务模型有自己的特点，我们先看一个例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) throws InterruptedException {
    // TaskRunnerFactory 的作用是创建线程池
    TaskRunnerFactory factory = new TaskRunnerFactory();
    factory.init();
    // 创建 PooledTaskRunner
    TaskRunner taskRunner = factory.createTaskRunner(new Task() {
        // iterate 的返回值很重要，true表示继续，false表示停止
        public boolean iterate() {
            System.out.println("hello zhang");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return true;
        }
    }, "zhang");
    
    // 调用线程池的 execute(runnable)
    taskRunner.wakeup();
    
    LockSupport.park();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Task 接口真正处理业务逻辑。factory.createTaskRunner 的作用只是创建一个命名的 PooledTaskRunner。

PooledTaskRunner 封装了线程池 executor 和任务 runnable，只有在调用 PooledTaskRunner.wakeup() 时，才会调用 executor.execute(runnable)，即真正执行任务。

 

以 Queue 类为例，它继承了 Task 接口，并且有自己的 taskRunner：

```
// org.apache.activemq.broker.region.Queue
public void initialize() throws Exception {
    // 省略其他代码
    // 创建queue的taskRunner
    this.taskRunner = taskFactory.createTaskRunner(this, "Queue:" + destination.getPhysicalName());
}
```

 queue 的 iterate：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//org.apache.activemq.broker.region.Queue
@Override
public boolean iterate() {
    MDC.put("activemq.destination", getName());
    boolean pageInMoreMessages = false;
    synchronized (iteratingMutex) {

        // If optimize dispatch is on or this is a slave this method could be called recursively
        // we set this state value to short-circuit wakeup in those cases to avoid that as it
        // could lead to errors.
        iterationRunning = true;

        // do early to allow dispatch of these waiting messages
        synchronized (messagesWaitingForSpace) {
            Iterator<Runnable> it = messagesWaitingForSpace.values().iterator();
            while (it.hasNext()) {
                if (!memoryUsage.isFull()) {
                    Runnable op = it.next();
                    it.remove();
                    op.run();
                } else {
                    registerCallbackForNotFullNotification();
                    break;
                }
            }
        }

        if (firstConsumer) {
            firstConsumer = false;
            try {
                if (consumersBeforeDispatchStarts > 0) {
                    int timeout = 1000; // wait one second by default if
                                        // consumer count isn't reached
                    if (timeBeforeDispatchStarts > 0) {
                        timeout = timeBeforeDispatchStarts;
                    }
                    if (consumersBeforeStartsLatch.await(timeout, TimeUnit.MILLISECONDS)) {
                        LOG.debug("{} consumers subscribed. Starting dispatch.", consumers.size());
                    } else {
                        LOG.debug("{} ms elapsed and {} consumers subscribed. Starting dispatch.", timeout, consumers.size());
                    }
                }
                if (timeBeforeDispatchStarts > 0 && consumersBeforeDispatchStarts <= 0) {
                    iteratingMutex.wait(timeBeforeDispatchStarts);
                    LOG.debug("{} ms elapsed. Starting dispatch.", timeBeforeDispatchStarts);
                }
            } catch (Exception e) {
                LOG.error(e.toString());
            }
        }

        messagesLock.readLock().lock();
        try{
            pageInMoreMessages |= !messages.isEmpty();
        } finally {
            messagesLock.readLock().unlock();
        }

        pagedInPendingDispatchLock.readLock().lock();
        try {
            pageInMoreMessages |= !pagedInPendingDispatch.isEmpty();
        } finally {
            pagedInPendingDispatchLock.readLock().unlock();
        }

        // Perhaps we should page always into the pagedInPendingDispatch
        // list if
        // !messages.isEmpty(), and then if
        // !pagedInPendingDispatch.isEmpty()
        // then we do a dispatch.
        boolean hasBrowsers = browserDispatches.size() > 0;

        if (pageInMoreMessages || hasBrowsers || !redeliveredWaitingDispatch.isEmpty()) {
            try {
　　　　　　　　　 //分发消息                pageInMessages(hasBrowsers);
            } catch (Throwable e) {
                LOG.error("Failed to page in more queue messages ", e);
            }
        }

        if (hasBrowsers) {
            ArrayList<MessageReference> alreadyDispatchedMessages = null;
            pagedInMessagesLock.readLock().lock();
            try{
                alreadyDispatchedMessages = new ArrayList<MessageReference>(pagedInMessages.values());
            }finally {
                pagedInMessagesLock.readLock().unlock();
            }

            Iterator<BrowserDispatch> browsers = browserDispatches.iterator();
            while (browsers.hasNext()) {
                BrowserDispatch browserDispatch = browsers.next();
                try {
                    MessageEvaluationContext msgContext = new NonCachedMessageEvaluationContext();
                    msgContext.setDestination(destination);

                    QueueBrowserSubscription browser = browserDispatch.getBrowser();

                    LOG.debug("dispatch to browser: {}, already dispatched/paged count: {}", browser, alreadyDispatchedMessages.size());
                    boolean added = false;
                    for (MessageReference node : alreadyDispatchedMessages) {
                        if (!((QueueMessageReference)node).isAcked() && !browser.isDuplicate(node.getMessageId()) && !browser.atMax()) {
                            msgContext.setMessageReference(node);
                            if (browser.matches(node, msgContext)) {
                                browser.add(node);
                                added = true;
                            }
                        }
                    }
                    // are we done browsing? no new messages paged
                    if (!added || browser.atMax()) {
                        browser.decrementQueueRef();
                        browserDispatches.remove(browserDispatch);
                    }
                } catch (Exception e) {
                    LOG.warn("exception on dispatch to browser: {}", browserDispatch.getBrowser(), e);
                }
            }
        }

        if (pendingWakeups.get() > 0) {
            pendingWakeups.decrementAndGet();
        }
        MDC.remove("activemq.destination");
        iterationRunning = false;

        return pendingWakeups.get() > 0;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 队列分发消息：

```
protected void pageInMessages(boolean force) throws Exception {
    doDispatch(doPageInForDispatch(force, true));
}
```

# [ActiveMQ queue和topic，持久订阅和非持久订阅](https://www.cnblogs.com/allenwas3/p/8794737.html)

消息的 destination 分为 queue 和 topic，而消费者称为 subscriber（订阅者）。queue 中的消息只会发送给一个订阅者，而 topic 的消息，会发送给每一个订阅者。在 broker 中，处理 queue 消息和 topic 消息的逻辑是不同的。queue 先存储消息，然后把消息分发给消费者，topic 收到消息的同时，就会分发。

Queue 中有 doMessageSend 和 iterate 方法，doMessageSend 负责接收生产者的消息，iterate 负责分发消息给消费者。Topic 中也有 doMessageSend 和 iterate 方法，doMessageSend 负责接收生产者的消息，并且分发给消费者。

queue 有持久和临时2种类型（topic相同）：
队列默认为持久队列，一旦创建，一直存在于broker中。而临时队列被创建后，在connection关闭后，broker就会删除它。

topic 订阅有持久和非持久2种类型：
broker 会把消息全部推送给持久订阅，即便该订阅者中途offline了，如果是非持久订阅，一旦它下线，broker 不会为它保留消息，直到它上线后，开始继续发送消息。

需要注意：假定有一个 topic，生产者向该 topic 发送一条消息，但此时该 topic 没有任何订阅者，则该消息不会保存，它会被删除。

（创建持久订阅）代码示例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    //该连接上会创建durable subscriber，需要指定唯一clientID
    ActiveMQConnectionFactory connectionFactory =
            new ActiveMQConnectionFactory("tcp://localhost:61616?jms.clientID=10086");
            
    ActiveMQConnection connection = (ActiveMQConnection)connectionFactory.createConnection();
    connection.start();
    ActiveMQSession session = (ActiveMQSession) connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    //创建topic
    ActiveMQTopic destination = (ActiveMQTopic) session.createTopic("topic_zhang");
    //创建持久订阅者
    TopicSubscriber consumer = session.createDurableSubscriber(destination, "subscriber_zhang");
    //普通消费者，即非持久订阅者
    ActiveMQMessageConsumer consumer2 = (ActiveMQMessageConsumer) session.createConsumer(destination);
    
}
```

# [ActiveMQ topic 普通订阅和持久订阅](https://www.cnblogs.com/allenwas3/p/8807641.html)

直观的结果：当生产者向 topic 发送消息，

\1. 若不存在持久订阅者和在线的普通订阅者，这个消息不会保存，当普通订阅者上线后，它是收不到消息的。

\2. 若存在离线的持久订阅者，broker 会为该持久订阅者保存消息，当该持久订阅者上线后，会收到消息。

本质：producer 发送消息给 topic，broker 接收消息并且分发给 consumers。consumers 包括持久订阅者和在线的普通订阅者，对于持久订阅者，broker 把消息添加到它的 message cursor 中；对于普通订阅者，broker 直接分发消息。

如果，1个 topic 有2个持久订阅者，并且这2个持久订阅者都不在线，这时 producer 向 topic 发送1条消息，则 broker 会保存2条消息。因此，如果1个 topic 有很多不在线的持久订阅者，会导致 broker 消耗过多存储。

对于持久化的消息，这很好验证：为方便查看消息，将 broker 持久化方式配置为jdbc，则可以在 ACTIVEMQ_MSGS 表中看到持久化消息。

持久订阅者1：

```
new ActiveMQConnectionFactory("tcp://localhost:61616?jms.clientID=10086");
...
TopicSubscriber consumer = session.createDurableSubscriber(destination, "subscriber_zhang");
```

持久订阅者2：

```
new ActiveMQConnectionFactory("tcp://localhost:61616?jms.clientID=10087");
...
TopicSubscriber consumer = session.createDurableSubscriber(destination, "subscriber_zhang");
```

对于持久消息，验证 broker 为每个持久订阅者保存1条消息：
\1. 启动 broker；
\2. 分别启动2个持久订阅者，然后关闭它们，这样 broker 有了2个离线的持久订阅者；
\3. 启动 producer 向 topic 发送1条消息；
\4. 查看 ACTIVEMQ_MSGS 表

对于非持久消息，直接跟代码了，这里不说明。

下图是 Topic 接收消息并分发的调用栈：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180412140157376-104941960.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// org.apache.activemq.broker.region.policy.SimpleDispatchPolicy
// 分发策略很简单，就是遍历consumers
public boolean dispatch(MessageReference node, MessageEvaluationContext msgContext, List<Subscription> consumers)
        throws Exception {
    int count = 0;
    for (Subscription sub : consumers) {
        // Don't deliver to browsers
        if (sub.getConsumerInfo().isBrowser()) {
            continue;
        }
        // Only dispatch to interested subscriptions
        if (!sub.matches(node, msgContext)) {
            sub.unmatched(node);
            continue;
        }

        //持久化订阅是 DurableTopicSubscription
        //普通订阅是 TopicSubscription
        sub.add(node);
        count++;
    }

    return count > 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

持久化订阅：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// org.apache.activemq.broker.region.DurableTopicSubscription
public void add(MessageReference node) throws Exception {
    if (!active.get() && !keepDurableSubsActive) {
        return;
    }
    // 调用 PrefetchSubscription.add
    super.add(node);
}

//  org.apache.activemq.broker.region.PrefetchSubscription
public void add(MessageReference node) throws Exception {
    synchronized (pendingLock) {
        // The destination may have just been removed...
        if( !destinations.contains(node.getRegionDestination()) && node!=QueueMessageReference.NULL_MESSAGE) {
            // perhaps we should inform the caller that we are no longer valid to dispatch to?
            return;
        }

        // Don't increment for the pullTimeout control message.
        if (!node.equals(QueueMessageReference.NULL_MESSAGE)) {
            enqueueCounter++;
        }
        //首先加入到 message cursor 中，pending 类型为 StoreDurableSubscriberCursor
        pending.addMessageLast(node);
    }
    dispatchPending();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

持久化订阅者上线后，也会触发消息分发动作即 dispatchPending，调用栈如下图：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180412145330299-1793647858.png)

持久化订阅者使用的 message cursor 是 StoreDurableSubscriberCursor。

 

普通订阅：

org.apache.activemq.broker.region.TopicSubscription.add(MessageReference node)

# [ActiveMQ 中 consumer 的优先级，message 的优先级](https://www.cnblogs.com/allenwas3/p/8854319.html)

[http://activemq.apache.org/consumer-priority.html ](http://activemq.apache.org/consumer-priority.html )consumer 优先级

http://activemq.apache.org/activemq-message-properties.html 消息优先级

 

1、设置 consumer 的优先级：

```
queue = new ActiveMQQueue("TEST.QUEUE?consumer.priority=10");
consumer = session.createConsumer(queue);
```

priority 的取值从0到127。broker 按照 consumer 的优先级给 queue 的 consumers 排序，首先把消息分发给优先级最高的 consumer。一旦该 consumer 的 prefetch buffer 满了，broker 就把消息分发给优先级次高的，prefetch buffer 不满的 consumer。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// org.apache.activemq.broker.region.Queue
// consumer priority 的比较器
private final Comparator<Subscription> orderedCompare = new Comparator<Subscription>() {

    @Override
    public int compare(Subscription s1, Subscription s2) {
        // We want the list sorted in descending order
        // 倒序，即数值大的优先级高
        int val = s2.getConsumerInfo().getPriority() - s1.getConsumerInfo().getPriority();
        if (val == 0 && messageGroupOwners != null) {
            // then ascending order of assigned message groups to favour less loaded consumers
            // Long.compare in jdk7
            long x = s1.getConsumerInfo().getLastDeliveredSequenceId();
            long y = s2.getConsumerInfo().getLastDeliveredSequenceId();
            val = (x < y) ? -1 : ((x == y) ? 0 : 1);
        }
        return val;
    }
};

// 添加 consumer 的时候，会触发排序
// 在 consumers 列表中，靠前的 consumer，先分发消息
private void addToConsumerList(Subscription sub) {
    if (useConsumerPriority) {
        consumers.add(sub);
        Collections.sort(consumers, orderedCompare);
    } else {
        consumers.add(sub);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

2、设置 message 的优先级需要在 broker 端和 producer 端配置：

2.1 在 broker 端设置 TEST.BAT 队列为 prioritizedMessages = "true"

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<policyEntry queue="TEST.BAT" prioritizedMessages="true" producerFlowControl="true" memoryLimit="1mb">
    <deadLetterStrategy>
        <individualDeadLetterStrategy queuePrefix="TEST"/>
    </deadLetterStrategy>
    <pendingQueuePolicy>
        <storeCursor/>
    </pendingQueuePolicy>
</policyEntry>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2.2 producer 发送消息时，设置 message 的优先级

```
TextMessage message = session.createTextMessage(text);
producer.send(destination, message, DeliveryMode.NON_PERSISTENT, 1, 0);
```

设置 message 的优先级，需要调用：

```
void javax.jms.MessageProducer.send(Destination destination, Message message, int deliveryMode, int priority, long timeToLive)
throws JMSException
```

而不能这么写：

```
TextMessage message = session.createTextMessage(text);
message.setJMSPriority(0);
```

初步看是 ActiveMQ 的 bug。消息的 priority 值，从0到9。消息配置了优先级之后，消息存放在 PrioritizedPendingList 中。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 省略部分代码
private class PrioritizedPendingListIterator implements Iterator<MessageReference> {
    private int index = 0;
    private int currentIndex = 0;
    List<PendingNode> list = new ArrayList<PendingNode>(size());

    PrioritizedPendingListIterator() {
        for (int i = MAX_PRIORITY - 1; i >= 0; i--) {
            // priority 值大的优先级高
            OrderedPendingList orderedPendingList = lists[i];
            if (!orderedPendingList.isEmpty()) {
                list.addAll(orderedPendingList.getAsList());
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

# [ActiveMQ broker 集群, 静态发现和动态发现](https://www.cnblogs.com/allenwas3/p/8872822.html)

下载 activemq 压缩包解压后，conf 目录下有各种示例配置文件，红线标出的是静态发现和动态发现的配置。

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180418104310815-502816355.png)

 

\1. 静态配置

启动3个 broker，端口分别为61616，61618，61620，配置如下：

```
<networkConnectors></networkConnectors>
<transportConnectors>
    <transportConnector name="openwire" uri="tcp://localhost:61616"/>
</transportConnectors>
<networkConnectors>
    <networkConnector uri="static:(tcp://localhost:61616)" duplex="true"/>
</networkConnectors>
<transportConnectors>
    <transportConnector name="openwire" uri="tcp://localhost:61618"/>
</transportConnectors>
<networkConnectors>
    <networkConnector uri="static:(tcp://localhost:61616,tcp://localhost:61618)" duplex="true"/>
</networkConnectors>
<transportConnectors>
    <transportConnector name="openwire" uri="tcp://localhost:61620"/>
</transportConnectors>
```

3个 broker 组成了一张网，当 producer 发送消息给 broker：61616 后，broker：61618 的消费者可以收到该消息。消息从 broker：61616 流动到 broker：61618，底层原理是 broker：61618 是 broker：61616 的一个消费者。

\2. 动态配置

同样地， 启动3个 broker，端口分别为61616，61618，61620，配置如下：

```
<networkConnectors>
    <networkConnector uri="multicast://default"/>
</networkConnectors>
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61616" discoveryUri="multicast://default" />
</transportConnectors>
<networkConnectors>
    <networkConnector uri="multicast://default"/>
</networkConnectors>
<transportConnectors>
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61618" discoveryUri="multicast://default" />
</transportConnectors>
<networkConnectors>
    <networkConnector uri="multicast://default"/>
</networkConnectors>
<transportConnectors>
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61620" discoveryUri="multicast://default" />
</transportConnectors>
```

使用多播协议，把 3 个 broker 动态地组成了一张网。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 省略其他代码
public class MulticastDiscoveryAgent implements DiscoveryAgent, Runnable {

    public static final String DEFAULT_DISCOVERY_URI_STRING = "multicast://239.255.2.3:6155";
    public static final String DEFAULT_HOST_STR = "default"; 
    public static final String DEFAULT_HOST_IP  = System.getProperty("activemq.partition.discovery", "239.255.2.3"); 
    public static final int    DEFAULT_PORT  = 6155; 
        
    private static final Logger LOG = LoggerFactory.getLogger(MulticastDiscoveryAgent.class);
    private static final String TYPE_SUFFIX = "ActiveMQ-4.";
    private static final String ALIVE = "alive.";
    private static final String DEAD = "dead.";
    private static final String DELIMITER = "%";
    private static final int BUFF_SIZE = 8192;
    private static final int DEFAULT_IDLE_TIME = 500;
    private static final int HEARTBEAT_MISS_BEFORE_DEATH = 10;
    
    public void run() {
        byte[] buf = new byte[BUFF_SIZE];
        DatagramPacket packet = new DatagramPacket(buf, 0, buf.length);
        while (started.get()) {
            // 发送多播数据
            doTimeKeepingServices();
            try {
                // 接收多播数据
                mcast.receive(packet);
                if (packet.getLength() > 0) {
                    String str = new String(packet.getData(), packet.getOffset(), packet.getLength());
                    processData(str);
                }
            } catch (SocketTimeoutException se) {
                // ignore
            } catch (IOException e) {
                if (started.get()) {
                    LOG.error("failed to process packet: " + e);
                }
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

\3. broker 集群的原理

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180423231250735-1361686485.png)

如上图，61616和61618组成集群，当61618加入一个consumer时，61618向61616发送一条ConsumerInfo消息，这样61618就成为了61616的consumer。

ConsumerInfo 示例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ConsumerInfo {
    commandId = 4, 
    responseRequired = false, 
    consumerId = dynamic-broker1->dynamic-broker2-1872-1524494145961-2:1:1:1, 
    destination = queue://TEST.BAT, 
    prefetchSize = 1, 
    maximumPendingMessageLimit = 0, 
    browser = false, 
    dispatchAsync = true, 
    selector = null, 
    clientId = ID:USER-20140617MT-1882-1524494166015-0:1, 
    subscriptionName = null, 
    noLocal = false, 
    exclusive = true, 
    retroactive = false, 
    priority = -5, 
    brokerPath = [ID:USER-20140617MT-1877-1524494148767-0:1], 
    optimizedAcknowledge = false, 
    noRangeAcks = false, 
    additionalPredicate = org.apache.activemq.command.NetworkBridgeFilter@413249b
}
```

# [ActiveMQ 处理不同类型的消息](https://www.cnblogs.com/allenwas3/p/8910576.html)

ActiveMQ 中的消息都继承自 org.apache.activemq.command.BaseCommand 类。

broker 处理消息的调用栈如下：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180422225410818-2089988050.png)

TransportConnection 类实现了 CommandVisitor 接口，描述了处理各种消息的逻辑。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class TransportConnection implements Connection, Task, CommandVisitor {
    @Override
    public Response service(Command command) {
        ...
        // command 即消息。以 ProducerInfo 为例
        response = command.visit(this);
        ...
    }

    @Override
    public Response processAddProducer(ProducerInfo info) throws Exception {
        SessionId sessionId = info.getProducerId().getParentId();
        ConnectionId connectionId = sessionId.getParentId();
        TransportConnectionState cs = lookupConnectionState(connectionId);
        if (cs == null) {
            throw new IllegalStateException("Cannot add a producer to a connection that had not been registered: "
                    + connectionId);
        }
        SessionState ss = cs.getSessionState(sessionId);
        if (ss == null) {
            throw new IllegalStateException("Cannot add a producer to a session that had not been registered: "
                    + sessionId);
        }
        // Avoid replaying dup commands
        if (!ss.getProducerIds().contains(info.getProducerId())) {
            ActiveMQDestination destination = info.getDestination();
            if (destination != null && !AdvisorySupport.isAdvisoryTopic(destination)) {
                if (getProducerCount(connectionId) >= connector.getMaximumProducersAllowedPerConnection()){
                    throw new IllegalStateException("Can't add producer on connection " + connectionId + ": at maximum limit: " + connector.getMaximumProducersAllowedPerConnection());
                }
            }
            broker.addProducer(cs.getContext(), info);
            try {
                ss.addProducer(info);
            } catch (IllegalStateException e) {
                broker.removeProducer(cs.getContext(), info);
            }

        }
        return null;
    }

}

// org.apache.activemq.command.ProducerInfo
public Response visit(CommandVisitor visitor) throws Exception {
    return visitor.processAddProducer(this);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

# [ActiveMQ 的连接和会话](https://www.cnblogs.com/allenwas3/p/8934336.html)

要了解 connection 和 session 的概念，可以先从 ConnectionState 和 SessionState 入手：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 省略部分代码
public class ConnectionState {
    ConnectionInfo info;
    private final ConcurrentHashMap<TransactionId, TransactionState> transactions = new ConcurrentHashMap<TransactionId, TransactionState>();
    private final ConcurrentHashMap<SessionId, SessionState> sessions = new ConcurrentHashMap<SessionId, SessionState>();
    private final List<DestinationInfo> tempDestinations = Collections.synchronizedList(new ArrayList<DestinationInfo>());
    private final AtomicBoolean shutdown = new AtomicBoolean(false);
    private boolean connectionInterruptProcessingComplete = true;
    private HashMap<ConsumerId, ConsumerInfo> recoveringPullConsumers;

    public ConnectionState(ConnectionInfo info) {
        this.info = info;
        // Add the default session id.
        addSession(new SessionInfo(info, -1));
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从代码可以看出，连接里有事务集合、会话集合、临时队列集合等，这说明：
\1. 事务属于一个连接； 2. 会话属于一个连接； 3. 临时队列的生存期是连接的有效期

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 省略部分代码
public class SessionState {
    final SessionInfo info;

    private final Map<ProducerId, ProducerState> producers = new ConcurrentHashMap<ProducerId, ProducerState>();
    private final Map<ConsumerId, ConsumerState> consumers = new ConcurrentHashMap<ConsumerId, ConsumerState>();
    private final AtomicBoolean shutdown = new AtomicBoolean(false);

    public SessionState(SessionInfo info) {
        this.info = info;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从上面能看出，producer 和 consumer 是属于某个会话的，producer 和 consumer 都有唯一的 ID 。

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 省略部分代码
public class ProducerState {
    final ProducerInfo info;
    private TransactionState transactionState;
}

public class ConsumerState {        
    final ConsumerInfo info;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ProducerState 和 ConsumerState 只是做了简单的封装。

其中 ConnectionInfo, SessionInfo, ProducerInfo, ConsumerInfo 都是消息类型，均继承自 BaseCommand 接口。

# [ActiveMQ 配置jdbc主从](https://www.cnblogs.com/allenwas3/p/8955458.html)

使用 jdbc 方式配置主从模式，持久化消息存放在数据库中。

在同一时刻，只有一个 master broker，master 接受客户端的连接，slave 不接受连接。
当 master 因为关机而下线后，其中一个 slave 会提升为 master，然后接受客户端连接。但原来 master 的非持久消息丢失了，而持久消息保存在数据库中。

broker xml 配置：使用 MySQL 数据源

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!--
    Licensed to the Apache Software Foundation (ASF) under one or more
    contributor license agreements.  See the NOTICE file distributed with
    this work for additional information regarding copyright ownership.
    The ASF licenses this file to You under the Apache License, Version 2.0
    (the "License"); you may not use this file except in compliance with
    the License.  You may obtain a copy of the License at
   
    http://www.apache.org/licenses/LICENSE-2.0
   
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
-->
<!--  
    Use JDBC for message persistence
    For more information, see:
    
    http://activemq.apache.org/persistence.html
    
    You need to add Derby database to your classpath in order to make this example work.
    Download it from http://db.apache.org/derby/ and put it in the ${ACTIVEMQ_HOME}/lib/optional/ folder
    Optionally you can configure any other RDBM as shown below
    
    To run ActiveMQ with this configuration add xbean:conf/activemq-jdbc.xml to your command
    
    e.g. $ bin/activemq xbean:conf/activemq-jdbc.xml
 -->
<beans
  xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
  http://activemq.apache.org/schema/core http://activemq.apache.org/schema/core/activemq-core.xsd">
  
  <broker useJmx="false" brokerName="jdbcBroker" xmlns="http://activemq.apache.org/schema/core">
    <persistenceAdapter>
       <jdbcPersistenceAdapter dataDirectory="${activemq.base}/data" dataSource="#mysql-ds"/>
    </persistenceAdapter>

    <transportConnectors>
       <transportConnector name="default" uri="tcp://0.0.0.0:61616"/>
    </transportConnectors>
  </broker>
 
  <!-- Embedded Derby DataSource Sample Setup -->
 <!--  <bean id="derby-ds" class="org.apache.derby.jdbc.EmbeddedDataSource">
    <property name="databaseName" value="derbydb"/>
    <property name="createDatabase" value="create"/>
  </bean> -->
 
  <!-- Postgres DataSource Sample Setup -->
  <!--
  <bean id="postgres-ds" class="org.postgresql.ds.PGPoolingDataSource">
    <property name="serverName" value="localhost"/>
    <property name="databaseName" value="activemq"/>
    <property name="portNumber" value="0"/>
    <property name="user" value="activemq"/>
    <property name="password" value="activemq"/>
    <property name="dataSourceName" value="postgres"/>
    <property name="initialConnections" value="1"/>
    <property name="maxConnections" value="10"/>
  </bean>
  -->

  <!-- MySql DataSource Sample Setup -->

  <bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://192.168.40.8:3306/db_zhang?relaxAutoCommit=true"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
    <property name="maxActive" value="200"/>
    <property name="poolPreparedStatements" value="true"/>
  </bean>

  <!-- Oracle DataSource Sample Setup -->
  <!--
  <bean id="oracle-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
    <property name="url" value="jdbc:oracle:thin:@localhost:1521:AMQDB"/>
    <property name="username" value="scott"/>
    <property name="password" value="tiger"/>
    <property name="maxActive" value="200"/>
    <property name="poolPreparedStatements" value="true"/>
  </bean>
  -->

</beans>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

客户端配置，producer 和 consumer 是一样的：

```
new ActiveMQConnectionFactory("failover:(tcp://localhost:61616,tcp://localhost:61618)");
```

 

以 MySQL 数据库为例，启动 broker 后，MySQL 中会创建 3 张表：
ACTIVEMQ_MSGS 存放持久消息
ACTIVEMQ_LOCK 表中只有一条记录，broker 执行 SELECT * FROM ACTIVEMQ_LOCK FOR UPDATE 获取锁，获得锁的 broker 是 master
ACTIVEMQ_ACKS

broker 获取锁的调用栈如下：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180426223943214-116275680.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// org.apache.activemq.store.jdbc.DefaultDatabaseLocker
public void doStart() throws Exception {

    LOG.info("Attempting to acquire the exclusive lock to become the Master broker");
    // SELECT * FROM ACTIVEMQ_LOCK FOR UPDATE
    String sql = getStatements().getLockCreateStatement();
    LOG.debug("Locking Query is "+sql);
    
    while (true) {
        try {
            connection = dataSource.getConnection();
            connection.setAutoCommit(false);
            lockCreateStatement = connection.prepareStatement(sql);
            // 执行 SELECT * FROM ACTIVEMQ_LOCK FOR UPDATE
            // 如果成功，则跳出循环。如果超时，则抛异常
            lockCreateStatement.execute();
            break;
        } catch (Exception e) {
            try {
                if (isStopping()) {
                    throw new Exception(
                            "Cannot start broker as being asked to shut down. " 
                                    + "Interrupted attempt to acquire lock: "
                                    + e, e);
                }
                if (exceptionHandler != null) {
                    try {
                        exceptionHandler.handle(e);
                    } catch (Throwable handlerException) {
                        LOG.error( "The exception handler "
                                + exceptionHandler.getClass().getCanonicalName()
                                + " threw this exception: "
                                + handlerException
                                + " while trying to handle this exception: "
                                + e, handlerException);
                    }

                } else {
                    LOG.debug("Lock failure: "+ e, e);
                }
            } finally {
                // Let's make sure the database connection is properly
                // closed when an error occurs so that we're not leaking
                // connections 
                if (null != connection) {
                    try {
                        connection.rollback();
                    } catch (SQLException e1) {
                        LOG.debug("Caught exception during rollback on connection: " + e1, e1);
                    }
                    try {
                        connection.close();
                    } catch (SQLException e1) {
                        LOG.debug("Caught exception while closing connection: " + e1, e1);
                    }
                    
                    connection = null;
                }
            }
        } finally {
            if (null != lockCreateStatement) {
                try {
                    lockCreateStatement.close();
                } catch (SQLException e1) {
                    LOG.debug("Caught while closing statement: " + e1, e1);
                }
                lockCreateStatement = null;
            }
        }

        LOG.info("Failed to acquire lock.  Sleeping for " + lockAcquireSleepInterval + " milli(s) before trying again...");
        try {
            Thread.sleep(lockAcquireSleepInterval);
        } catch (InterruptedException ie) {
            LOG.warn("Master lock retry sleep interrupted", ie);
        }
    }

    LOG.info("Becoming the master on dataSource: " + dataSource);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

broker 执行 SELECT * FROM ACTIVEMQ_LOCK FOR UPDATE 获取锁，获取成功，则成为 master，如果失败，则睡眠一段时间后，继续获取锁。

超时而抛出的异常：

![img](https://images2018.cnblogs.com/blog/461546/201804/461546-20180426224227665-1400332589.png)

 [ActiveMQ 事务和XA](https://www.cnblogs.com/allenwas3/p/9003418.html)

\1. 客户端怎样显式地使用事务？

producer 开启事务（代码片段）：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ActiveMQSession session = (ActiveMQSession)connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Destination destination = session.createQueue("TEST.FOO");
MessageProducer producer = session.createProducer(destination);
producer.setDeliveryMode(DeliveryMode.PERSISTENT);

// 开启事务 
// 发送 TransactionInfo 消息 BEGIN
session.getTransactionContext().begin();

for (int i = 0; i < 2; i++) {
    // Create a message
    String text = "zhang";
    TextMessage message = session.createTextMessage(text);
    producer.send(message);
}
// session.getTransactionContext().rollback();
//提交事务
// 发送 TransactionInfo 消息 COMMIT_ONE_PHASE
session.getTransactionContext().commit();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

\2. broker 处理事务的入口：

```
TransportConnection.processBeginTransaction
TransportConnection.processCommitTransactionOnePhase
TransportConnection.processCommitTransactionTwoPhase
```

broker 处理事务的逻辑在 TransactionBroker 类中。

那么，具体在 Queue 中是怎样体现事务的呢？

ActiveMQ 客户端默认不会开启事务，而如果客户端显式地开启了事务，则 Queue 中可能会存在多个事务，一个事务中必然会有一个消息列表，当客户端提交事务时，Queue 接收事务对应的消息列表，而如果客户端回滚事务，则 Queue 会删除这些消息。

Queue 中的事务变量：

```
// 键是Transaction，值是对应的消息列表
final ConcurrentHashMap<Transaction, SendSync> sendSyncs = new ConcurrentHashMap<Transaction, SendSync>();
private final LinkedList<Transaction> orderIndexUpdates = new LinkedList<Transaction>();
```

Queue 内部类 SendSync 封装了消息和同步操作：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class SendSync extends Synchronization {

    class MessageContext {
        public Message message;
        public ConnectionContext context;

        public MessageContext(ConnectionContext context, Message message) {
            this.context = context;
            this.message = message;
        }
    }

    final Transaction transaction;
    // 这就是我要找的消息列表
    List<MessageContext> additions = new ArrayList<MessageContext>();

    public SendSync(Transaction transaction) {
        this.transaction = transaction;
    }

    public void add(ConnectionContext context, Message message) {
        additions.add(new MessageContext(context, message));
    }

    @Override
    public void beforeCommit() throws Exception {
        synchronized (orderIndexUpdates) {
            orderIndexUpdates.addLast(transaction);
        }
    }

    @Override
    public void afterCommit() throws Exception {
        ArrayList<SendSync> syncs = new ArrayList<SendSync>(200);
        sendLock.lockInterruptibly();
        try {
            synchronized (orderIndexUpdates) {
                Transaction next = orderIndexUpdates.peek();
                while( next!=null && next.isCommitted() ) {
                    syncs.add(sendSyncs.remove(orderIndexUpdates.removeFirst()));
                    next = orderIndexUpdates.peek();
                }
            }
            for (SendSync sync : syncs) {
                sync.processSend();
            }
        } finally {
            sendLock.unlock();
        }
        for (SendSync sync : syncs) {
            sync.processSent();
        }
    }

    // called with sendLock
    private void processSend() throws Exception {

        for (Iterator<MessageContext> iterator = additions.iterator(); iterator.hasNext(); ) {
            MessageContext messageContext = iterator.next();
            // It could take while before we receive the commit
            // op, by that time the message could have expired..
            if (broker.isExpired(messageContext.message)) {
                broker.messageExpired(messageContext.context, messageContext.message, null);
                destinationStatistics.getExpired().increment();
                iterator.remove();
                continue;
            }
            sendMessage(messageContext.message);
            messageContext.message.decrementReferenceCount();
        }
    }

    private void processSent() throws Exception {
        for (MessageContext messageContext : additions) {
            messageSent(messageContext.context, messageContext.message);
        }
    }

    @Override
    public void afterRollback() throws Exception {
        try {
            for (MessageContext messageContext : additions) {
                messageContext.message.decrementReferenceCount();
            }
        } finally {
            sendSyncs.remove(transaction);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

\3. 那么 XA 事务又是什么呢？ActiveMQ 实现了分布式事务，当系统中存在多数据源的情况下，也许会需要使用 XA ，为了方便，只提供一个单数据源的例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Xid xid = new MyXid(1, new byte[]{0x01}, new byte[]{0x02});
session.getTransactionContext().start(xid, XAResource.TMSUCCESS);
// 操作mq
session.getTransactionContext().end(xid, XAResource.TMSUCCESS);
int prepare = session.getTransactionContext().prepare(xid);
System.out.println("prepare:" + prepare);
// 根据prepare结果决定是否提交
session.getTransactionContext().commit(xid, false);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这些操作步骤，和 MySQL的 XA 是一样的，也是 start，end，prepare，commit，实现的都是javax transaction 那一套接口。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyXid implements Xid {
    private int formatId;
    private byte[] globalTid;
    private byte[] branchQ;
    
    public MyXid(int formatId, byte[] globalTid, byte[] branchQ) {
        this.formatId = formatId;
        this.globalTid = globalTid;
        this.branchQ = branchQ;
    }
    
    public byte[] getBranchQualifier() {
        return this.branchQ;
    }

    public int getFormatId() {
        return formatId;
    }

    public byte[] getGlobalTransactionId() {
        return this.globalTid;
    }
}
```

# [ActiveMQ producer 提交事务时突然宕机，会发生什么](https://www.cnblogs.com/allenwas3/p/9133133.html)

producer 在提交事务时，发生宕机，commit 的命令没有发送到 broker，这时会发生什么？

ActiveMQ 开启事务发送消息的步骤：

```
session.getTransactionContext().begin();

producer.send(message);

session.getTransactionContext().commit();
```

在第三步加断点，然后关闭 producer 进程，模仿宕机。

broker 感知到 producer 的连接关闭后，会触发删除连接操作，回滚该连接下没有提交的事务。

![img](https://images2018.cnblogs.com/blog/461546/201806/461546-20180604141755519-2105821492.png)

# Activemq的消息事务 

消息事务
消息事务，是保证消息传递原子性的一个重要特征，和JDBC的事务特征类似。
一个事务性发送，其中一组消息要么能够全部保证到达服务器，要么都不到达服务器。
生产者、消费者与消息服务器直接都支持事务性；
ActionMQ的事务主要偏向在生产者的应用。

ActionMQ 消息事务流程图：

[![img](https://typora-oss.oss-cn-beijing.aliyuncs.com/image-20201016100112087.png)](https://typora-oss.oss-cn-beijing.aliyuncs.com/image-20201016100112087.png)

一、生产者事务：

没有加入事务的时候，会有部分信息过去，结果如图：

[![img](https://typora-oss.oss-cn-beijing.aliyuncs.com/image-20201016101148802.png)](https://typora-oss.oss-cn-beijing.aliyuncs.com/image-20201016101148802.png)

方式一：



```java
 /**
     * 事务性发送--方案一
     */
    @Test
    public void sendMessageTx(){
        //获取连接工厂
        ConnectionFactory connectionFactory = jmsMessagingTemplate.getConnectionFactory();

        Session session = null;
        try {
            //创建连接
            Connection connection = connectionFactory.createConnection();

            /**
             * 参数一：是否开启消息事务
             */
            session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);

            //创建生产者
            MessageProducer producer = session.createProducer(session.createQueue(name));

            for(int i=1;i<=10;i++){
                //模拟异常
                if(i==4){
                    int a = 10/0;
                }

                TextMessage textMessage = session.createTextMessage("消息--" + i);
                producer.send(textMessage);
            }

            //注意：一旦开启事务发送，那么就必须使用commit方法进行事务提交，否则消息无法到达MQ服务器
            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
            //消息事务回滚
            try {
                session.rollback();
            } catch (JMSException e1) {
                e1.printStackTrace();
            }
        }


    }
```

结果，没有发送出去

方式二：



```java
/**
 * ActiveMQ配置类
 */
@Configuration
public class ActiveMQConfig {

    /**
     * 添加Jms事务管理器
     */
    @Bean
    public PlatformTransactionManager createTransactionManager(ConnectionFactory connectionFactory){
        return new JmsTransactionManager(connectionFactory);
    }

}

/**
 * 消息发送的业务类
 */
@Service
public class MessageService {

    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Value("${activemq.name}")
    private String name;

    @Transactional // 对消息发送加入事务管理（同时也对JDBC数据库的事务生效）
    public void sendMessage(){
        for(int i=1;i<=10;i++) {
            //模拟异常
            if(i==4){
                int a = 10/0;
            }

            jmsMessagingTemplate.convertAndSend(name, "消息---"+i);
        }
    }

}
```

二、消费者事务



```java
/**
 * 用于监听消息类（既可以用于队列的监听，也可以用于主题监听）
 */
@Component // 放入IOC容器
public class MsgListener {

    /**
     * 接收TextMessage的方法
     */
    @JmsListener(destination = "${activemq.name}")
    public void receiveMessage(Message message,Session session){
        if(message instanceof TextMessage){
            TextMessage textMessage = (TextMessage)message;

            try {
                System.out.println("接收消息："+textMessage.getText());


                int i=10/0;

                //提交事务
                session.commit();
            } catch (JMSException e) {
                e.printStackTrace();
                //回滚事务
                try {
                    session.rollback();//一旦事务回滚，MQ会重发消息，一共重发6次
                } catch (JMSException e1) {
                    e1.printStackTrace();
                }
            }

        }
    }

}
```

注意如果在消费者异常了，会收到消息，然后重发6次，要是期间还是异常，就会到私信队列中

[![img](https://typora-oss.oss-cn-beijing.aliyuncs.com/image-20201016101936691.png)](https://typora-oss.oss-cn-beijing.aliyuncs.com/image-20201016101936691.png)

消息中间件大多支持事务消息，activemq也不例外。

关于事务的定义及ACID特性这里不赘述。

------

对比Mysql数据库来说，

Mysql有事务的概念，

Activemq也有事务的概念

这里说的都是本地事务，rocketMq还支持分布式事务

------

java制定了jdbc来规范对数据库的访问

同样

java也有jms（java message services）来规范对于消息中间件的访问，activemq是完全支持jms1.1规范的

------

那java通过mysql来实现事务，

无外乎大概这样：



```
 public static void main(String[] args) {
        Connection conn =getConnection();
        try {
            conn.setAutoCommit(false);
            insertUser(conn);
            insertAddress(conn);
            conn.commit();
        } catch (SQLException e) {
            System.out.println("************事务处理出现异常***********");
            e.printStackTrace();
            try {
                conn.rollback();
                System.out.println("*********事务回滚成功***********");
            } catch (Exception e2) {
                // TODO: handle exception
                e2.printStackTrace();
            }finally {
                try {
                    conn.close();
                } catch (SQLException e1) {
                    // TODO Auto-generated catch block
                    e1.printStackTrace();
                }
            }
        }
    }
```

那我们接上文：[Spring整合Activemq](https://www.cnblogs.com/heliusKing/p/12243548.html)

来看看activemq如何实现**事务消息**

------



```
    @Test
    public void p2pSender() {
        //获取连接工厂
        ConnectionFactory connectionFactory = jmsQueueTemplate.getConnectionFactory();
        Session session = null;
        try {
            Connection connection = connectionFactory.createConnection();
            // 参数一：是否开启消息事务
            session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);

            MessageProducer producer = session.createProducer(session.createQueue("test.trsaction"));
            //发送10条消息，开启事务后，要么一起成功，要么一起失败
            for (int i = 0; i < 10; i++) {
                if (i == 4) {
                    //出现异常
//                    throw new RuntimeException("i cannot equals 4");
                }
                TextMessage textMessage = session.createTextMessage("消息-----" + i);
                producer.send(textMessage);
            }
            //开启事务后，需要手动提交
            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
            //消息事务回滚
            try {
                session.rollback();
            } catch (JMSException ex) {
                ex.printStackTrace();
            }
        }
    }
```

上图我们手动制造了一个异常。

没有异常时：可以成功发送10条消息

[![image-20200130211242542](https://img2018.cnblogs.com/blog/1005988/202001/1005988-20200130213402346-1598333650.png)](https://img2018.cnblogs.com/blog/1005988/202001/1005988-20200130213402346-1598333650.png)

我们把异常那行代码打开

，再次执行：

从控制台可以发现，信息并没有发出去。

[![image-20200130211348932](https://img2018.cnblogs.com/blog/1005988/202001/1005988-20200130213401959-1903847987.png)](https://img2018.cnblogs.com/blog/1005988/202001/1005988-20200130213401959-1903847987.png)

即现在有事务效果。

------

另外需要一提的是:

整合入spring后，应该委托spring进行事务管理。

对于Mysql，我们配置了`DatasourceTransactionManager`，

使用@transactional注解即可。

对于activemq也是一样，不过使用的 就是JmsTransactionManager了。

------

还需要注意的是，分布式事务这块，mysql和activemq同为资源管理器（RM）,都实现了XA协议,activemq这个用的不多， 支持的也不完全，不赘述。

# ActiveMQ顺序消费消息+消息分组

简介
Queue中的消息是按照顺序发送给Consumers的。然而，当你有多个Consumer同时从相同的Queue提取消息时，顺序将不能得到保证。因为这些消息时被多个线程并发的处理。但是，有时候保证消息的顺序是很重要的。例如，你可能不希望插入订单操作结束之前执行更新订单的操作。那么我们可以通过Exclusive Consumer和Message Groups来实现这一目的。

独有消费者
从ActiveMQ4.X版本开始支持ExclusiveConsumer（或者说是Exclusive Queues）。Broker会从多个Consumer中挑选一个Consumer来处理所有的消息，从而保证消息的有序处理。如果这个Consumer失效，那么Broker会自动切换到其他的Consumer。

可以通过Destination的Option来创建一个Exclusive Consumer，如下：

queue = new ActiveMQQueue("Test.Queue?consumer.exclusive=true");
consumer = session.createConsumer(queue);
1
2
另外，还可以给Consumer设置优先级，以便针对网络情况进行优化。如下：

queue = new ActiveMQQueue("Test.Queue?consumer.exclusive=true&consumer.priority=10");
1
消息分组
从Apache官方文档的话说，是Exclusive Consumer功能的增强。逻辑上，可以看成是一种并发的Exclusive Consumer。JMS消息属性JMXGroupID被用来区分Message Group。Message Groups特性保证所有具有相同JMSGroupID的消息会被分发到相同的Consumer（只要这个Consumer保持Active）。另一方面，Message Groups也是一种负载均衡的机制。

在一个消息被分发到Consumer前，Broker会检查消息的JMSGroupID属性。如果存在，那么broker会检查是否有某个Consumer拥有这个Message Group。如果没有，那么broker会选择一个Consumer，并将它关联到这个Message Group。此后，这个Consumer会接收这个Message Group的所有消息。直到：

Consumer被关闭。
Message Group被关闭。通过发送一个消息，并设置这个消息的JMSXGroupSeq为-1.
从4.1版本开始，ActiveMQ支持一个布尔字段JMSXGroupFirstForConsumer 。当某个message group的第一个消息被发送到consumer的时候，这个字段被设置。如果客户使用failover transport连接到broker。在由于网络问题等造成客户重新连接到broker的时候，相同message group的消息可能会被分发到不同与之前的consumer，因此JMSXGroupFirstForConsumer字段也会被重新设置。

创建一个Message Groups
创建一个Message Groups，只需要在Message对象上设置属性即可。如下：

Message message = session.createTextMessage("hello,world");
message.setStringProperty("JMSXGroupID","GroupA");
...
producer.send(message);
1
2
3
4
关闭一个Message Groups
关闭一个Message Group，也只需要在Message对象上设置相应的属性即可。如下：

message.setStringProperty("JMSXGroupID","GroupA");
message.setIntProperty("JMSXGroupSeq", -1);

## ActiveMQ Consumer 使用 push 还是 pull 获取消息

ActiveMQ是一个消息中间件，对于消费者而言有两种方式从消息中间件获取消息：

①Push方式：由消息中间件主动地将消息推送给消费者；②Pull方式：由消费者主动向消息中间件拉取消息。看一段官网对Push方式的解释：

```
To be able to achieve high performance it is important to stream messages to consumers as fast as possible 
so that the consumer always has a buffer of messages, in RAM, ready to process 
- rather than have them explicitly pull messages from the server which adds significant latency per message.
```

采用Push方式，可以尽可能快地将消息发送给消费者(stream messages to consumers as fast as possible)

而采用Pull方式，会增加消息的延迟，即消息到达消费者的时间有点长(adds significant latency per message)。

 

但是，Push方式会有一个坏处：如果消费者的处理消息的能力很弱(一条消息需要很长的时间处理)，而消息中间件不断地向消费者Push消息，消费者的缓冲区可能会溢出。

那ActiveMQ是怎么解决这个问题的呢？那就是 [ **prefetch limit**](http://activemq.apache.org/what-is-the-prefetch-limit-for.html)

**prefetch limit 规定了一次可以向消费者Push(推送)多少条消息。**

```
 Once the prefetch limit is reached, no more messages are dispatched to the consumer 
until the consumer starts sending back acknowledgements of messages (to indicate that the message has been processed)
```

当推送消息的数量到达了perfetch limit规定的数值时，消费者还没有向消息中间件返回ACK，消息中间件将不再继续向消费者推送消息。

 

**那prefetch limit的值设置为多少合适？视具体的应用场景而定。**

```
 If you have very few messages and each message takes a very long time to process 
you might want to set the prefetch value to 1 so that a consumer is given one message at a time. 
```

如果消息的数量很少(生产者生产消息的速率不快)，但是每条消息 消费者需要很长的时间处理，那么prefetch limit设置为1比较合适。这样，消费者每次只会收到一条消息，当它处理完这条消息之后，向消息中间件发送ACK，此时消息中间件再向消费者推送下一条消息。

**prefetch limit 设置成0意味着什么？**

```
Specifying a prefetch limit of zero means the consumer will poll for more messages, one at a time, 
instead of the message being pushed to the consumer.
```

意味着此时，消费者去轮询消息中间件获取消息。不再是Push方式了，而是Pull方式了。即消费者主动去消息中间件拉取消息。

 

perfetch limit是“消息预取”的值，这是针对消息中间件如何向消费者发消息 而设置的。与之相关的还有针对 消费者以何种方式向消息中间件返回确认ACK(响应)：比如消费者是每次消费一条消息之后就向消息中间件确认呢？还是采用“延迟确认”---即采用批量确认的方式(消费了若干条消息之后，统一再发ACK)。这就是 [Optimized Acknowledge](http://activemq.apache.org/performance-tuning.html)

```
ActiveMQ can acknowledge receipt of messages back to the broker in batches (to improve performance). 
```

 

[引用 一段话](http://shift-alt-ctrl.iteye.com/blog/2020182)：“如果prefetchACK为true，那么prefetch必须大于0；当prefetchACK为false时，你可以指定prefetch为0以及任意大小的正数。
不过，当prefetch=0是，表示consumer将使用PULL(拉取)的方式从broker端获取消息，broker端将不会主动push消息给client端，直到client端发送PullCommand时；
当prefetch>0时，就开启了broker push模式，此后只要当client端消费且ACK了一定的消息之后，会立即push给client端多条消息。”

 

**那么，在程序中如何采用Push方式或者Pull方式呢？**

从是否阻塞来看，消费者有两种方式获取消息。同步方式和异步方式。

同步方式使用的是ActiveMQMessageConsumer的receive()方法。而异步方式则是采用消费者实现MessageListener接口，监听消息。

 

使用同步方式receive()方法获取消息时，prefetch limit即可以设置为0，也可以设置为大于0

prefetch limit为零 意味着：“receive()方法将会首先发送一个PULL指令并阻塞，直到broker端返回消息为止，这也意味着消息只能逐个获取(类似于Request<->Response)”

prefetch limit 大于零 意味着：“broker端将会批量push给client 一定数量的消息(<= prefetch)，client端会把这些消息(unconsumedMessage)放入到本地的队列中，**只要此队列有消息，那么receive方法将会立即返回**，当一定量的消息ACK之后，broker端会继续批量push消息给client端。”

 

当使用MessageListener异步获取消息时，prefetch limit必须大于零了。因为，prefetch limit 等于零 意味着消息中间件不会主动给消费者Push消息，而此时消费者又用MessageListener被动获取消息(不会主动去轮询消息)。这二者是矛盾的。

 

**此外，还有一个要注意的地方，即消费者采用同步获取消息(receive方法) 与 异步获取消息的方法(MessageListener) ，对消息的确认时机是不同的。**

# ActiveMQ消息的消费原理

### 消费端消费消息：

　　在 [初识ActiveMQ](https://www.cnblogs.com/wuzhenzhao/p/10084045.html) 中我提到过，两种方法可以接收消息，一种是使用同步阻塞的ActiveMQMessageConsumer#receive方法。另一种是使用消息监听器MessageListener。这里需要注意的是，在同一个session下，这两者不能同时工作，也就是说不能针对不同消息采用不同的接收方式。否则会抛出异常。至于为什么这么做，最大的原因还是在事务性会话中，两种消费模式的事务不好管控。

　　先通过ActiveMQMessageConsumer#receive 方法来对消息的接受一探究竟：

```
public` `Message receive() ``throws` `JMSException {``    ``checkClosed();``    ``//检查receive和MessageListener是否同时配置在当前的会话中，有则抛出异常``    ``checkMessageListener();``    ``//如果PrefetchSizeSize为0并且unconsumerMessage为空，则发起pull命令``    ``sendPullCommand(``0``);``    ``MessageDispatch md = dequeue(-``1``);``//出列，获取消息``    ``if` `(md == ``null``) {``      ``return` `null``;``    ``}``    ``beforeMessageIsConsumed(md);``    ``//发送ack给到broker``    ``afterMessageIsConsumed(md, ``false``);``    ``//获取消息并返回``    ``return` `createActiveMQMessage(md);``  ``}
```

　　下面简单的说一下以上几个核心方法中做了什么不为人知的事：

　　sendPullCommand(0) ：发送pull命令从broker上获取消息，前提是prefetchSize=0并且unconsumedMessages为空。unconsumedMessage表示未消费的消息，这里面预读取的消息大小为prefetchSize的值

```
protected` `void` `sendPullCommand(``long` `timeout) ``throws` `JMSException {``    ``clearDeliveredList();``    ``if` `(info.getCurrentPrefetchSize() == ``0` `&& unconsumedMessages.isEmpty()) {``      ``MessagePull messagePull = ``new` `MessagePull();``      ``messagePull.configure(info);``      ``messagePull.setTimeout(timeout);``      ``//向服务端异步发送messagePull指令``      ``session.asyncSendPacket(messagePull);``    ``}``  ``}
```

　　这里发送异步消息跟消息生产的原理是一样的。通过包装链去调用 Sokect 发送请求。

　　clearDeliveredList()：

　　在上面的sendPullCommand方法中，会先调用clearDeliveredList方法，主要用来清理已经分发的消息链表deliveredMessages，存储分发给消费者但还为应答的消息链表

　　　　Ø 如果session是事务的，则会遍历deliveredMessage中的消息放入到previouslyDeliveredMessage中来做重发
　　　　Ø 如果session是非事务的，根据ACK的模式来选择不同的应答操作

　　这是个同步的过程：

```
private` `void` `clearDeliveredList() {``  ``if` `(clearDeliveredList) {``//判断是否清楚``    ``synchronized` `(deliveredMessages) {``//采用双重检查锁``      ``if` `(clearDeliveredList) {``        ``if` `(!deliveredMessages.isEmpty()) {``          ``if` `(session.isTransacted()) {``//是事务消息``            ``if` `(previouslyDeliveredMessages == ``null``) {``              ``previouslyDeliveredMessages = ``new` `PreviouslyDeliveredMap<MessageId, Boolean>(session.getTransactionContext().getTransactionId());``            ``}``            ``for` `(MessageDispatch delivered : deliveredMessages) {``              ``previouslyDeliveredMessages.put(delivered.getMessage().getMessageId(), ``false``);``            ``}``            ``LOG.debug(``"{} tracking existing transacted {} delivered list ({}) on transport interrupt"``,``                 ``getConsumerId(), previouslyDeliveredMessages.transactionId, deliveredMessages.size());``          ``} ``else` `{``            ``if` `(session.isClientAcknowledge()) {``              ``LOG.debug(``"{} rolling back delivered list ({}) on transport interrupt"``, getConsumerId(), deliveredMessages.size());``              ``// allow redelivery``              ``if` `(!``this``.info.isBrowser()) {``                ``for` `(MessageDispatch md: deliveredMessages) {``                  ``this``.session.connection.rollbackDuplicate(``this``, md.getMessage());``                ``}``              ``}``            ``}``            ``LOG.debug(``"{} clearing delivered list ({}) on transport interrupt"``, getConsumerId(), deliveredMessages.size());``            ``deliveredMessages.clear();``            ``pendingAck = ``null``;``          ``}``        ``}``        ``clearDeliveredList = ``false``;``      ``}``    ``}``  ``}``}
```

　　dequeue(-1) ：从unconsumedMessage中取出一个消息，在创建一个消费者时，就会为这个消费者创建一个未消费的消息通道，这个通道分为两种，一种是简单优先级队列分发通道SimplePriorityMessageDispatchChannel ；另一种是先进先出的分发通道FifoMessageDispatchChannel.至于为什么要存在这样一个消息分发通道，大家可以想象一下，如果消费者每次去消费完一个消息以后再去broker拿一个消息，效率是比较低的。所以通过这样的设计可以允许session能够一次性将多条消息分发给一个消费者。默认情况下对于queue来说，prefetchSize的值是1000

```
private` `MessageDispatch dequeue(``long` `timeout) ``throws` `JMSException {``    ``try` `{``      ``long` `deadline = ``0``;``      ``if` `(timeout > ``0``) {``        ``deadline = System.currentTimeMillis() + timeout;``      ``}``      ``while` `(``true``) {``//protected final MessageDispatchChannel unconsumedMessages;``        ``MessageDispatch md = unconsumedMessages.dequeue(timeout);` `      ``...........``  ``}
```

　　beforeMessageIsConsumed(md)：这里面主要是做消息消费之前的一些准备工作，如果ACK类型不是DUPS_OK_ACKNOWLEDGE或者队列模式（简单来说就是除了Topic和DupAck这两种情况），所有的消息先放到deliveredMessages链表的开头。并且如果当前是事务类型的会话，则判断transactedIndividualAck，如果为true，表示单条消息直接返回ack。

　　否则，调用ackLater，批量应答, client端在消费消息后暂且不发送ACK，而是把它缓存下来(pendingACK)，等到这些消息的条数达到一定阀值时，只需要通过一个ACK指令把它们全部确认；这比对每条消息都逐个确认，在性能上要提高很多。

```
private` `void` `beforeMessageIsConsumed(MessageDispatch md) ``throws` `JMSException {``    ``md.setDeliverySequenceId(session.getNextDeliveryId());``    ``lastDeliveredSequenceId = md.getMessage().getMessageId().getBrokerSequenceId();``    ``if` `(!isAutoAcknowledgeBatch()) {``      ``synchronized``(deliveredMessages) {``        ``deliveredMessages.addFirst(md);``      ``}``      ``if` `(session.getTransacted()) {``        ``if` `(transactedIndividualAck) {``          ``immediateIndividualTransactedAck(md);``        ``} ``else` `{``          ``ackLater(md, MessageAck.DELIVERED_ACK_TYPE);``        ``}``      ``}``    ``}``  ``}　
```

　　afterMessageIsConsumed：这个方法的主要作用是执行应答操作，这里面做以下几个操作
　　　　Ø 如果消息过期，则返回消息过期的ack
　　　　Ø 如果是事务类型的会话，则不做任何处理
　　　　Ø 如果是AUTOACK或者（DUPS_OK_ACK且是队列），并且是优化ack操作，则走批量确认ack
　　　　Ø 如果是DUPS_OK_ACK，则走ackLater逻辑
　　　　Ø 如果是CLIENT_ACK，则执行ackLater

```
private` `void` `afterMessageIsConsumed(MessageDispatch md, ``boolean` `messageExpired) ``throws` `JMSException {``    ``if` `(unconsumedMessages.isClosed()) {``      ``return``;``    ``}``    ``if` `(messageExpired) {``      ``acknowledge(md, MessageAck.EXPIRED_ACK_TYPE);``      ``stats.getExpiredMessageCount().increment();``    ``} ``else` `{``      ``stats.onMessage();``      ``if` `(session.getTransacted()) {``        ``// Do nothing.``      ``} ``else` `if` `(isAutoAcknowledgeEach()) {``        ``if` `(deliveryingAcknowledgements.compareAndSet(``false``, ``true``)) {``          ``synchronized` `(deliveredMessages) {``            ``if` `(!deliveredMessages.isEmpty()) {``              ``if` `(optimizeAcknowledge) {``                ``ackCounter++;` `                ``// AMQ-3956 evaluate both expired and normal msgs as``                ``// otherwise consumer may get stalled``                ``if` `(ackCounter + deliveredCounter >= (info.getPrefetchSize() * .``65``) || (optimizeAcknowledgeTimeOut > ``0` `&& System.currentTimeMillis() >= (optimizeAckTimestamp + optimizeAcknowledgeTimeOut))) {``                  ``MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);``                  ``if` `(ack != ``null``) {``                    ``deliveredMessages.clear();``                    ``ackCounter = ``0``;``                    ``session.sendAck(ack);``                    ``optimizeAckTimestamp = System.currentTimeMillis();``                  ``}``                  ``// AMQ-3956 - as further optimization send``                  ``// ack for expired msgs when there are any.``                  ``// This resets the deliveredCounter to 0 so that``                  ``// we won't sent standard acks with every msg just``                  ``// because the deliveredCounter just below``                  ``// 0.5 * prefetch as used in ackLater()``                  ``if` `(pendingAck != ``null` `&& deliveredCounter > ``0``) {``                    ``session.sendAck(pendingAck);``                    ``pendingAck = ``null``;``                    ``deliveredCounter = ``0``;``                  ``}``                ``}``              ``} ``else` `{``                ``MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);``                ``if` `(ack!=``null``) {``                  ``deliveredMessages.clear();``                  ``session.sendAck(ack);``                ``}``              ``}``            ``}``          ``}``          ``deliveryingAcknowledgements.set(``false``);``        ``}``      ``} ``else` `if` `(isAutoAcknowledgeBatch()) {``        ``ackLater(md, MessageAck.STANDARD_ACK_TYPE);``      ``} ``else` `if` `(session.isClientAcknowledge()||session.isIndividualAcknowledge()) {``        ``boolean` `messageUnackedByConsumer = ``false``;``        ``synchronized` `(deliveredMessages) {``          ``messageUnackedByConsumer = deliveredMessages.contains(md);``        ``}``        ``if` `(messageUnackedByConsumer) {``          ``ackLater(md, MessageAck.DELIVERED_ACK_TYPE);``        ``}``      ``}``      ``else` `{``        ``throw` `new` `IllegalStateException(``"Invalid session state."``);``      ``}``    ``}``  ``}
```

　　其实在以上消息的接收过程中，我们仅仅能看到这个消息从一个本地变量中出队，并没有对远程消息中心发送通讯获取，那么这个消息时什么时候过来的呢？也就是消息出队中  unconsumedMessages 这个东东时什么时候初始化的呢 ？那么接下去我们应该去通过创建连接的时候去看看了，具体连接的时候都做了什么呢：connectionFactory.createConnection()

```
protected` `ActiveMQConnection createActiveMQConnection(String userName, String password) ``throws` `JMSException {``    ``if` `(brokerURL == ``null``) {``      ``throw` `new` `ConfigurationException(``"brokerURL not set."``);``    ``}``    ``ActiveMQConnection connection = ``null``;``    ``try` `{``// 果然发现了这个东东的初始化``      ``Transport transport = createTransport();``      ``// 创建连接``      ``connection = createActiveMQConnection(transport, factoryStats);``      ``// 设置用户密码``      ``connection.setUserName(userName);``      ``connection.setPassword(password);``      ``// 对连接做包装``      ``configureConnection(connection);``      ``// 启动一个后台传输线程``      ``transport.start();``      ``// 设置客户端消费的id``      ``if` `(clientID != ``null``) {``        ``connection.setDefaultClientID(clientID);``      ``}`` ` `      ``return` `connection;``    ``} ......``  ``}
```

　　创建连接的过程就是创建除了一个带有链路包装的TcpTransport 并且创建连接，最后启动一个传输线程，而这里的 transport.start() 调用的应该是TcpTransport 里面的方法，然而这个类中并没有 start，而是在父类
ServiceSupport.start()中：

```
public` `void` `start() ``throws` `Exception {``    ``if` `(started.compareAndSet(``false``, ``true``)) {``      ``boolean` `success = ``false``;``      ``stopped.set(``false``);``      ``try` `{``        ``preStart();``//一些初始化``        ``doStart();``        ``success = ``true``;``      ``} ``finally` `{``        ``started.set(success);``      ``}``      ``for``(ServiceListener l:``this``.serviceListeners) {``        ``l.started(``this``);``      ``}``    ``}``  ``}
```

　　doStart 方法前做了一系列的初始化，然后调用 TcpTransport的doStart() 方法：

```
protected` `void` `doStart() ``throws` `Exception {``    ``connect();``    ``stoppedLatch.set(``new` `CountDownLatch(``1``));``    ``super``.doStart();``  ``}
```

　　继而构建一个连接 设置一个 CountDownLatch 门闩 ，调用父类 TransportThreadSupport 的方法，新建了一个精灵线程并且启动：

```
protected` `void` `doStart() ``throws` `Exception {``    ``runner = ``new` `Thread(``null``, ``this``, ``"ActiveMQ Transport: "` `+ toString(), stackSize);``    ``runner.setDaemon(daemon);``    ``runner.start();``  ``}
```

　　调用TransportThreadSupport.doStart(). 创建了一个线程，传入的是 this，调用子类的 run 方法，也就是 TcpTransport.run().

```
public` `void` `run() {``    ``LOG.trace(``"TCP consumer thread for "` `+ ``this` `+ ``" starting"``);``    ``this``.runnerThread=Thread.currentThread();``    ``try` `{``      ``while` `(!isStopped()) {``        ``doRun();``      ``}``    ``} ``catch` `(IOException e) {``      ``stoppedLatch.get().countDown();``      ``onException(e);``    ``} ``catch` `(Throwable e){``      ``stoppedLatch.get().countDown();``      ``IOException ioe=``new` `IOException(``"Unexpected error occurred: "` `+ e);``      ``ioe.initCause(e);``      ``onException(ioe);``    ``}``finally` `{``      ``stoppedLatch.get().countDown();``    ``}``  ``}
```

　　run 方法主要是从 socket 中读取数据包，只要 TcpTransport 没有停止，它就会不断去调用 doRun：这里面，通过 wireFormat 对数据进行格式化，可以认为这是一个反序列化过程。wireFormat 默认实现是 OpenWireFormat，activeMQ 自定义的跨语言的wire 协议

```
protected` `void` `doRun() ``throws` `IOException {``    ``try` `{``//通过 readCommand 去读取数据``      ``Object command = readCommand();``      ``//消费消息``      ``doConsume(command);``    ``} ``catch` `(SocketTimeoutException e) {``    ``} ``catch` `(InterruptedIOException e) {``    ``}``  ``}``protected` `Object readCommand() ``throws` `IOException {``    ``return` `wireFormat.unmarshal(dataIn);``}
```

　　这里的读取流的部分就是从Socket里面读取，而这个连接的 输入/输出流的初始化在 TcpTransport

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void initializeStreams() throws Exception {
        TcpBufferedInputStream buffIn = new TcpBufferedInputStream(socket.getInputStream(), ioBufferSize) {
            @Override
            public int read() throws IOException {
                receiveCounter++;
                return super.read();
            }
            @Override
            public int read(byte[] b, int off, int len) throws IOException {
                receiveCounter++;
                return super.read(b, off, len);
            }
            @Override
            public long skip(long n) throws IOException {
                receiveCounter++;
                return super.skip(n);
            }
            @Override
            protected void fill() throws IOException {
                receiveCounter++;
                super.fill();
            }
        };
        //Unread the initBuffer that was used for protocol detection if it exists
        //so the stream can start over
        if (initBuffer != null) {
            buffIn.unread(initBuffer.buffer.array());
        }
        this.dataIn = new DataInputStream(buffIn);
        TcpBufferedOutputStream outputStream = new TcpBufferedOutputStream(socket.getOutputStream(), ioBufferSize);
        this.dataOut = new DataOutputStream(outputStream);
        this.buffOut = outputStream;

    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　doConsume：流程走到了消费消息： 

```
public` `void` `doConsume(Object command) {``    ``if` `(command != ``null``) {``//表示已经拿到了消息``      ``if` `(transportListener != ``null``) {``        ``transportListener.onCommand(command);``      ``} ``else` `{``        ``LOG.error(``"No transportListener available to process inbound command: "` `+ command);``      ``}``    ``}``  ``}
```

　　TransportSupport 类中唯一的成员变量是 TransportListener transportListener;，这也意味着一个 Transport 支持类绑定一个传送监听器类，传送监听器接口 TransportListener 最重要的方法就是 void onCommand(Object command);，它用来处理命令。那么这个 transportListener 是在那里初始化的呢？可以思考一下 既然是TransportSupport 唯一的成员变量，而我们锁创建的TcpTransport 是他的子类，那么是不是在创建该transport的时候亦或是在对他进行包装处理的时候做了初始化呢？ 我们会在流程中看到在新建 ActiveMQConnectionFactory 的时候有一行关键的代码：

```
connection = createActiveMQConnection(transport, factoryStats);
```

　　在这个方法里面追溯下去：会进入 ActiveMQConnection 的构造方法

```
protected` `ActiveMQConnection(``final` `Transport transport, IdGenerator clientIdGenerator, IdGenerator connectionIdGenerator, JMSStatsImpl factoryStats) ``throws` `Exception {` `    ``this``.transport = transport;``    ``this``.clientIdGenerator = clientIdGenerator;``    ``this``.factoryStats = factoryStats;` `    ``// Configure a single threaded executor who's core thread can timeout if``    ``// idle``    ``executor = ``new` `ThreadPoolExecutor(``1``, ``1``, ``5``, TimeUnit.SECONDS, ``new` `LinkedBlockingQueue<Runnable>(), ``new` `ThreadFactory() {``      ``@Override``      ``public` `Thread newThread(Runnable r) {``        ``Thread thread = ``new` `Thread(r, ``"ActiveMQ Connection Executor: "` `+ transport);``        ``//Don't make these daemon threads - see https://issues.apache.org/jira/browse/AMQ-796``        ``//thread.setDaemon(true);``        ``return` `thread;``      ``}``    ``});``    ``// asyncConnectionThread.allowCoreThreadTimeOut(true);``    ``String uniqueId = connectionIdGenerator.generateId();``    ``this``.info = ``new` `ConnectionInfo(``new` `ConnectionId(uniqueId));``    ``this``.info.setManageable(``true``);``    ``this``.info.setFaultTolerant(transport.isFaultTolerant());``    ``this``.connectionSessionId = ``new` `SessionId(info.getConnectionId(), -``1``);` `    ``this``.transport.setTransportListener(``this``);` `    ``this``.stats = ``new` `JMSConnectionStatsImpl(sessions, ``this` `instanceof` `XAConnection);``    ``this``.factoryStats.addConnection(``this``);``    ``this``.timeCreated = System.currentTimeMillis();``    ``this``.connectionAudit.setCheckForDuplicates(transport.isFaultTolerant());``  ``}
```

　　从以上代码我们发现 this.transport.setTransportListener(this); 那么这个this是什么呢 ？ 正是ActiveMQConnection ，看了一眼该类，发现这个类实现了 TransportListener ，本身就是一个TransportListener。所以上面 transportListener.onCommand(command); 就是 ActiveMQConnection.onCommand(command)。除了和 Transport相互绑定，还对线程池执行器 executor 进行了初始化。这哥执行器是后来要进行消息处理的。

　　这里面会针对不同的消息做分发，在ActiveMQMessageConsumer#receive方法中锁dequeue所返回的对象是MessageDispatch 。假设这里传入的 command 是MessageDispatch，那么这个 command 的 visit 方法就会调用processMessageDispatch 方法。剪切出其中的代码片段：

```
public` `Response processMessageDispatch(MessageDispatch md) ``throws` `Exception {``    ``// 等待 Transport 中断处理完成``    ``waitForTransportInterruptionProcessingToComplete();``    ``// 这里通过消费者 ID 来获取消费者对象``//（ActiveMQMessageConsumer 实现了 ActiveMQDispatcher 接口），所以``//MessageDispatch 包含了消息应该被分配到那个消费者的映射信息``    ``ActiveMQDispatcher dispatcher = dispatchers.get(md.getConsumerId());``    ``if` `(dispatcher != ``null``) {``    ``// Copy in case a embedded broker is dispatching via``    ``// vm://``    ``// md.getMessage() == null to signal end of queue``    ``// browse.``    ``Message msg = md.getMessage();``    ``if` `(msg != ``null``) {``    ``msg = msg.copy();``    ``msg.setReadOnlyBody(``true``);``    ``msg.setReadOnlyProperties(``true``);``    ``msg.setRedeliveryCounter(md.getRedeliveryCounter());``    ``msg.setConnection(ActiveMQConnection.``this``);``    ``msg.setMemoryUsage(``null``);``    ``md.setMessage(msg);``    ``}``    ``// 调用会话ActiveMQSession 自己的 dispatch 方法来处理这条消息``    ``dispatcher.dispatch(md);``    ``} ``else` `{``      ``LOG.debug(``"{} no dispatcher for {} in {}"``, ``this``, md, dispatchers);``    ``}``    ``return` `null``;``}
```

　　其中 ActiveMQDispatcher dispatcher = dispatchers.get(md.getConsumerId());这行代码的 dispatchers 是在 通过session.createConsumer(destination); 的时候通过 ActiveMQMessageConsumer 的构造方法中有一行代码 ：this.session.addConsumer(this); 将 this传入，即 ActiveMQMessageConsumer 对象。而这个 addConsumer 方法：

```
protected` `void` `addConsumer(ActiveMQMessageConsumer consumer) ``throws` `JMSException {``    ``this``.consumers.add(consumer);``    ``if` `(consumer.isDurableSubscriber()) {``      ``stats.onCreateDurableSubscriber();``    ``}``    ``this``.connection.addDispatcher(consumer.getConsumerId(), ``this``);``  ``}
```

　　可以发现这里的初始化了：this.connection.addDispatcher(consumer.getConsumerId(), this); 这里的this 即 ActiveMQSession。所以回到 ActiveMQConnection#onCommand方法内 processMessageDispatch 这个方法最后调用了 dispatcher.dispatch(md); 这个方法的核心功能就是处理消息的分发。：

```
public` `void` `dispatch(MessageDispatch messageDispatch) {``    ``try` `{``      ``executor.execute(messageDispatch);``    ``} ``catch` `(InterruptedException e) {``      ``Thread.currentThread().interrupt();``      ``connection.onClientInternalException(e);``    ``}``  ``}
```

　　这里离我们真正要找的进行消息入队的结果很近了，进入executor.execute(messageDispatch);这个方法：

```
void` `execute(MessageDispatch message) ``throws` `InterruptedException {` `    ``...........``//如果会话不是异步分发并且没有使用 sessionpool 分发，则调用 dispatch 发送消息``    ``if` `(!session.isSessionAsyncDispatch() && !dispatchedBySessionPool) {``      ``dispatch(message);``    ``} ``else` `{``//将消息直接放到队列里``      ``messageQueue.enqueue(message);``      ``wakeup();``    ``}``  ``}
```

　　这里最后终于发现了入队，判断是否异步分发，不是的话走dispatch(message) 否则进入异步分发。默认是采用异步消息分发。所以，直接调用 messageQueue.enqueue，把消息放到队列中，并且调用 wakeup 方法:

```
public` `void` `wakeup() {``    ``if` `(!dispatchedBySessionPool) {``//进一步验证``      ``// //判断 session 是否为异步分发``      ``if` `(session.isSessionAsyncDispatch()) {``        ``try` `{``          ``TaskRunner taskRunner = ``this``.taskRunner;``          ``if` `(taskRunner == ``null``) {``            ``synchronized` `(``this``) {``              ``if` `(``this``.taskRunner == ``null``) {``                ``if` `(!isRunning()) {``                  ``// stop has been called``                  ``return``;``                ``}``//通过 TaskRunnerFactory 创建了一个任务运行类 taskRunner，这里把自己作为一个 task 传入到 createTaskRunner 中，``//说明当前的类一定是实现了 Task 接口的. 简单来说，就是通过线程池去执行一个任务，完成异步调度``//这里由于executor != null 所以这个task的类型是PooledTaskRunner``                ``this``.taskRunner = session.connection.getSessionTaskRunner().createTaskRunner(``this``,``                    ``"ActiveMQ Session: "` `+ session.getSessionId());``              ``}``              ``taskRunner = ``this``.taskRunner;``            ``}``          ``}``          ``taskRunner.wakeup();``        ``} ``catch` `(InterruptedException e) {``          ``Thread.currentThread().interrupt();``        ``}``      ``} ``else` `{``// 异步分发``        ``while` `(iterate()) {``        ``}``      ``}``    ``}``  ``}
```

　　所以，对于异步分发的方式，会调用 ActiveMQSessionExecutor 中的 iterate方法，我们来看看这个方法的代码 iterate （）：这个方法里面做两个事

　　Ø 把消费者监听的所有消息转存到待消费队列中
　　Ø 如果 messageQueue 还存在遗留消息，同样把消息分发(调度)出去

```
public` `boolean` `iterate() {``    ``// Deliver any messages queued on the consumer to their listeners.    // 将消费者上排队的任何消息传递给它们的侦听器。``    ``for` `(ActiveMQMessageConsumer consumer : ``this``.session.consumers) {``      ``if` `(consumer.iterate()) {``        ``return` `true``;``      ``}``    ``}``    ``// No messages left queued on the listeners.. so now dispatch messages``    ``// queued on the session    // 侦听器上没有留下排队等待的消息。现在分派消息``    ``MessageDispatch message = messageQueue.dequeueNoWait();``    ``if` `(message == ``null``) {``      ``return` `false``;``    ``} ``else` `{``// 分发(调度)消息``      ``dispatch(message);``      ``return` `!messageQueue.isEmpty();``    ``}``  ``}
```

　　dispatch(message);消息确认分发。通过ActiveMQSessionExecutor的dispatch 方法，转到了 ActiveMQMessageConsumer 消费者类的  dispatch 方法：

```
public` `void` `dispatch(MessageDispatch md) {``    ``MessageListener listener = ``this``.messageListener.get();``    ``try` `{``      ``clearMessagesInProgress();``      ``clearDeliveredList();``      ``synchronized` `(unconsumedMessages.getMutex()) {``        ``if` `(!unconsumedMessages.isClosed()) {``// 判断消息是否为重发消息``          ``if` `(``this``.info.isBrowser() || !session.connection.isDuplicate(``this``, md.getMessage())) {``            ``if` `(listener != ``null` `&& unconsumedMessages.isRunning()) {``             ``//我这边通过consumer.receive()处理消息，所以这里listener为空，走下面``            ``} ``else` `{``              ``if` `(!unconsumedMessages.isRunning()) {``                ``// delayed redelivery, ensure it can be re delivered``                ``session.connection.rollbackDuplicate(``this``, md.getMessage());``              ``}` `              ``if` `(md.getMessage() == ``null``) {``                ``// End of browse or pull request timeout.``                ``unconsumedMessages.enqueue(md);``              ``} ``else` `{``                ``if` `(!consumeExpiredMessage(md)) {``                  ``unconsumedMessages.enqueue(md);``                  ``if` `(availableListener != ``null``) {``                    ``availableListener.onMessageAvailable(``this``);``                  ``}``　　　　　　.........``}
```

　　最终会走入 unconsumedMessages.enqueue(md);添加消息。这里需要注意的是enqueue 方法：由于消费者可能处于阻塞状态，这里做了入队后回释放锁，也就是接触阻塞。

```
public` `void` `enqueue(MessageDispatch message) {``    ``synchronized` `(mutex) {``      ``list.addLast(message);``      ``mutex.notify();``    ``}``  ``}
```

　　到这里为止，消息如何接受以及他的处理方式的流程，我们已经搞清楚了。其实在这个消息消费的流程中，已经在建立连接，创建消费者的时候就已经初始化好了消息队列了。结合上面的过程来看看整个消费流程的流程图

![img](https://img2018.cnblogs.com/blog/1383365/201812/1383365-20181213153131957-765939975.png)

###  消费端的 PrefetchSize：

　　在消息发布的时候我们曾经研究过 producerWindowSize 。主要用来约束在异步发送时producer端允许积压的(尚未ACK)的消息的大小，且只对异步发送有意义。对于客户端，也是类似存在这么一个属性来约束客户端的消息处理。activemq 的 consumer 端也有窗口机制，通过 prefetchSize 就可以设置窗口大小。不同的类型的队列，prefetchSize 的默认值也是不一样的.

　　Ø 持久化队列和非持久化队列的默认值为 1000

　　Ø 持久化 topic 默认值为 100

　　Ø 非持久化队列的默认值为 Short.MAX_VALUE-1

　测试方法是在MQ上生产1000条消息，先后启动comsumer1，comsumer2 两个消费者并且循环调用1000次消费，我们会发现 comsumer2 拿不到消息，这个时候我们可以通过debug进入comsumer1 的ActiveMQConnect会发现里面有个属性的size=1000.其实就是这个prefetchSize，翻译过来是预取大小，消费端会根据prefetchSize 的大小批量获取数据。意思是在创建连接的时候会取获取1000条消息预加载到缓存中等待处理，这样子导致comsumer2去获取消息的时候 broker上已经空了。

#### prefetchSize 的设置方法：

　　在 createQueue 中添加 consumer.prefetchSize，就可以看到效果

```
Destination destination=session.createQueue(``"myQueue?consumer.prefetchSize=10"``);
```

　　既然有批量加载，那么一定有批量确认，这样才算是彻底的优化，这就涉及到 optimizeAcknowledge

　　ActiveMQ 提供了 optimizeAcknowledge 来优化确认，它表示是否开启“优化ACK”，只有在为 true 的情况下，prefetchSize 以及optimizeAcknowledgeTimeout 参数才会有意义优化确认一方面可以减轻 client 负担（不需要频繁的确认消息）、减少通信开销，另一方面由于延迟了确认（默认 ack 了 0.65*prefetchSize 个消息才确认），这个在源码中有体现。在ActiveMQMessageConsumer#receive方法内的处理消息后的 afterMessageIsConsumed 方法内有一个判断：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
if (ackCounter + deliveredCounter >= (info.getPrefetchSize() * .65) ||
     (optimizeAcknowledgeTimeOut > 0 && 
         System.currentTimeMillis() >= (optimizeAckTimestamp + optimizeAcknowledgeTimeOut))) {
     MessageAck ack = makeAckForAllDeliveredMessages(MessageAck.STANDARD_ACK_TYPE);
       if (ack != null) {
             deliveredMessages.clear();
             ackCounter = 0;
             session.sendAck(ack);//满足条件则发送批量应答ACK
             optimizeAckTimestamp = System.currentTimeMillis();
       }
       // AMQ-3956 - as further optimization send
       // ack for expired msgs when there are any.
       // This resets the deliveredCounter to 0 so that
       // we won't sent standard acks with every msg just
       // because the deliveredCounter just below
       // 0.5 * prefetch as used in ackLater()
       if (pendingAck != null && deliveredCounter > 0) {
            session.sendAck(pendingAck);
            pendingAck = null;
            deliveredCounter = 0;
       }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　broker 再次发送消息时又可以批量发送如果只是开启了 prefetchSize，每条消息都去确认的话，broker 在收到确认后也只是发送一条消息，并不是批量发布，当然也可以通过设置 DUPS_OK_ACK来手动延迟确认， 我们需要在 brokerUrl 指定 optimizeACK 选项

```
ConnectionFactory connectionFactory= new ActiveMQConnectionFactory("tcp://192.168.11.153:61616?jms.optimizeAcknowledge=true&jms.optimizeAcknowledgeTimeOut=10000");
```

　　Ø 注意，如果 optimizeAcknowledge 为 true，那么 prefetchSize 必须大于 0. 当 prefetchSize=0 的时候，表示 consumer 通过 PULL 方式从 broker 获取消息.

　　optimizeAcknowledge 和 prefetchSize 的作用，两者协同工作，通过批量获取消息、并延迟批量确认，来达到一个高效的消息消费模型。它比仅减少了客户端在获取消息时的阻塞次数，还能减少每次获取消息时的网络通信开销

　　Ø 需要注意的是，如果消费端的消费速度比较高，通过这两者组合是能大大提升 consumer 的性能。如果 consumer 的消费性能本身就比较慢，设置比较大的 prefetchSize 反而不能有效的达到提升消费性能的目的。因为过大的prefetchSize 不利于 consumer 端消息的负载均衡。因为通常情况下，我们都会部署多个 consumer 节点来提升消费端的消费性能。这个优化方案还会存在另外一个潜在风险，当消息被消费之后还没有来得及确认时，client 端发生故障，那么这些消息就有可能会被重新发送给其他consumer，那么这种风险就需要 client 端能够容忍“重复”消息。

###  消息的确认过程：

　　消息确认有四种 ACK_MODE，分别是：

　　　　1. AUTO_ACKNOWLEDGE = 1 自动确认

　　　　2.CLIENT_ACKNOWLEDGE = 2 客户端手动确认

　　　　3.DUPS_OK_ACKNOWLEDGE = 3 自动批量确认

　　　　4.SESSION_TRANSACTED = 0 事务提交并确认

 　ACK_MODE 的选择影响着消息消费流程的走向。虽然 Client 端指定了 ACK 模式,但是在 Client 与 broker 在交换 ACK 指令的时候,还需要告知 ACK_TYPE,ACK_TYPE 表示此确认指令的类型，不同的ACK_TYPE 将传递着消息的状态，broker 可以根据不同的 ACK_TYPE 对消息进行不同的操作。

#### ACK_TYPE应答类型：

　　DELIVERED_ACK_TYPE = 0 消息"已接收"，但尚未处理结束

　　STANDARD_ACK_TYPE = 2 "标准"类型,通常表示为消息"处理成功"，broker 端可以删除消息了

　　POSION_ACK_TYPE = 1 消息"错误",通常表示"抛弃"此消息，比如消息重发多次后，都无法正确处理时，消息将会被删除或者 DLQ(死信队列)，在消息处理的时候，dispatch方法内会判断该消息是否为重发消息

```
if` `(``this``.info.isBrowser() || !session.connection.isDuplicate(``this``, md.getMessage())) {``            ``if` `(listener != ``null` `&& unconsumedMessages.isRunning()) {``            ``// 这段为非重发消息，走else``          ``} ``else` `{``            ``// deal with duplicate delivery``            ``ConsumerId consumerWithPendingTransaction;``            ``if` `(redeliveryExpectedInCurrentTransaction(md, ``true``)) {``              ``LOG.debug(``"{} tracking transacted redelivery {}"``, getConsumerId(), md.getMessage());``              ``if` `(transactedIndividualAck) {``                ``immediateIndividualTransactedAck(md);``              ``} ``else` `{``                ``session.sendAck(``new` `MessageAck(md, MessageAck.DELIVERED_ACK_TYPE, ``1``));``              ``}``            ``} ``else` `if` `((consumerWithPendingTransaction = redeliveryPendingInCompetingTransaction(md)) != ``null``) {``              ``LOG.warn(``"{} delivering duplicate {}, pending transaction completion on {} will rollback"``, getConsumerId(), md.getMessage(), consumerWithPendingTransaction);``              ``session.getConnection().rollbackDuplicate(``this``, md.getMessage());``              ``dispatch(md);``            ``} ``else` `{``// 走POSION_ACK_TYPE 添加Active_DLQ 死信队列``              ``LOG.warn(``"{} suppressing duplicate delivery on connection, poison acking: {}"``, getConsumerId(), md);``              ``posionAck(md, ``"Suppressing duplicate delivery on connection, consumer "` `+ getConsumerId());``            ``}``          ``}
```

　　REDELIVERED_ACK_TYPE = 3 消息需"重发"，比如 consumer 处理消息时抛出了异常，broker 稍后会重新发送此消息

　　INDIVIDUAL_ACK_TYPE = 4 表示只确认"单条消息",无论在任何 ACK_MODE 下

　　UNMATCHED_ACK_TYPE = 5 在 Topic 中，如果一条消息在转发给“订阅者”时，发现此消息不符合 Selector 过滤条件，那么此消息将 不会转发给订阅者，消息将会被存储引擎删除(相当于在 Broker 上确　　　　     认了消息)。

　　Client 端在不同的 ACK 模式时,将意味着在不同的时机发送 ACK 指令,每个 ACK Command 中会包含 ACK_TYPE,那么 broker 端就可以根据 ACK_TYPE 来决定此消息的后续操作。在 afterMessageIsConsumed 消息接收处理后会根据条件来设置 ACK_TYPE.

### 消息的重发机制原理:

　　在正常情况下，有几中情况会导致消息重新发送

　　Ø 在事务性会话中，没有调用 session.commit 确认消息宕机或者调用session.rollback 方法回滚消息

　　Ø 在非事务性会话中，ACK 模式为 CLIENT_ACKNOWLEDGE (客户端手动应答)的情况下，没有调用 session.commit或者调用了 recover 方法；

　　一个消息被 redelivedred 超过默认的最大重发次数（默认 6 次）时，消费端会给 broker 发送一个”poison ack”表示这个消息有毒，告诉 broker 不要再发了。这个时候 broker 会把这个消息放到 DLQ（死信队列）。设置方法如下：

```
ActiveMQConnectionFactory connectionFactory1 = (ActiveMQConnectionFactory) connectionFactory;
RedeliveryPolicy redeliveryPolicy = new RedeliveryPolicy();
redeliveryPolicy.setMaximumRedeliveries(2);
connectionFactory1.setRedeliveryPolicy(redeliveryPolicy);
```

**死信队列：**

　　ActiveMQ 中默认的死信队列是 ActiveMQ.DLQ，如果没有特别的配置，有毒的消息都会被发送到这个队列。默认情况下，如果持久消息过期以后，也会被送到 DLQ 中。

 　只要在处理消息的时候抛出一个异常就可以演示，会看到控制台对于失败消息会重发6次，登陆ActiveMQ控制台会看到一个 ActiveMQ.DLQ。在创建队列的时候可以直接指定从ActiveMQ.DLQ去消费消息。

　　死信队列配置策略：

　　缺省所有队列的死信消息都被发送到同一个缺省死信队列，不便于管理，可以通过 individualDeadLetterStrategy 或 sharedDeadLetterStrategy 策略来进行修改。在activemq.xml上

```
<destinationPolicy>``   ``<policyMap>``     ``<policyEntries>``      ``<policyEntry topic=``">"` `>``          ``<!-- The constantPendingMessageLimitStrategy is used to prevent``             ``slow topic consumers to block producers and affect other consumers``             ``by limiting the number of messages that are retained``             ``For more information, see:` `             ``http:``//activemq.apache.org/slow-consumer-handling.html``          ``-->``        ``<pendingMessageLimitStrategy>``         ``<constantPendingMessageLimitStrategy limit=``"1000"``/>``        ``</pendingMessageLimitStrategy>``      ``</policyEntry>` `　　　　　　``// “>”表示对所有队列生效，如果需要设置指定队列，则直接写队列名称``     ``<policyEntry queue=``">"``>``       ``<deadLetterStrategy>``   ``　　　　 ``//queuePrefix:设置死信队列前缀``   ``　　　　　``//useQueueForQueueMessage 设置队列保存到死信。``       ``<individualDeadLetterStrategy queuePrefix=``"DLQ."``useQueueForQueueMessages=``"true"``/>``       ``</deadLetterStrategy>``     ``</policyEntry>``     ``</policyEntries>``    ``</policyMap>``</destinationPolicy>
```

　　自动丢弃过期消息

```
<deadLetterStrategy>``  ``<sharedDeadLetterStrategy processExpired=``"false"` `/>``</deadLetterStrategy>
```

### ActiveMQ VirtualTopic：

　　ActiveMQ支持的虚拟Destinations分为有两种，分别是

- **虚拟主题（Virtual Topics）**
- **组合 Destinations（CompositeDestinations）**

　　这两种虚拟Destinations可以看做对简单的topic和queue用法的补充，基于它们可以实现一些简单有用的EIP功能，虚拟主题类似于1对多的分支功能+消费端的cluster+failover，组合Destinations类似于简单的destinations直接的路由功能。

**组合队列（Composite Destinations）：**

　　当你想把同一个消息一次发送到多个消息队列，那么可以在客户端使用组合队列。

```
// send to 3 queues as one logical operation
Queue queue = new ActiveMQQueue("FOO.A,FOO.B,FOO.C");
producer.send(queue, someMessage);
```

　　当然，也可以混合使用队列和主题，只需要使用前缀：queue:// 或 topic://

```
// send to queues and topic one logical operation
Queue queue = new ActiveMQQueue("FOO.A,topic://NOTIFY.FOO.A");
producer.send(queue, someMessage);
```

### **虚拟主题（Virtual Topics）：**

　　ActiveMQ中，topic只有在持久订阅（durablesubscription）下是持久化的。存在持久订阅时，每个持久订阅者，都相当于一个持久化的queue的客户端，它会收取所有消息。这种情况下存在两个问题：

1. 同一应用内consumer端负载均衡的问题：同一个应用上的一个持久订阅不能使用多个consumer来共同承担消息处理功能。因为每个都会获取所有消息。queue模式可以解决这个问题，broker端又不能将消息发送到多个应用端。所以，既要发布订阅，又要让消费者分组，这个功能jms规范本身是没有的。
2. 同一应用内consumer端failover的问题：由于只能使用单个的持久订阅者，如果这个订阅者出错，则应用就无法处理消息了，系统的健壮性不高。

　　为了解决这两个问题，ActiveMQ中实现了虚拟Topic的功能。使用起来非常简单。对于消息发布者来说，就是一个正常的Topic，名称以VirtualTopic.开头。例如VirtualTopic.TEST。对于消息接收端来说，是个队列，不同应用里使用不同的前缀作为队列的名称，即可表明自己的身份即可实现消费端应用分组。例如Consumer.A.VirtualTopic.TEST，说明它是名称为A的消费端，同理Consumer.B.VirtualTopic.TEST说明是一个名称为B的客户端。可以在同一个应用里使用多个consumer消费此queue，则可以实现上面两个功能。又因为不同应用使用的queue名称不同（前缀不同），所以不同的应用中都可以接收到全部的消息。每个客户端相当于一个持久订阅者，而且这个客户端可以使用多个消费者共同来承担消费任务。

　　默认虚拟主题的前缀是 ：VirtualTopic.* 。自定义消费虚拟地址默认格式：Consumer.*.VirtualTopic.> 。自定义消费虚拟地址可以改，比如下面的配置就把它修改了。xml配置示例如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<broker xmlns="http://activemq.apache.org/schema/core">
    <destinationInterceptors>
        <virtualDestinationInterceptor>
            <virtualDestinations>
                <virtualTopic name=">" prefix="VirtualTopicConsumers.*." selectorAware="false"/><!-- 修改的Consumer的开头格式-->
            </virtualDestinations>
        </virtualDestinationInterceptor>
    </destinationInterceptors>
</broker>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　那么生产者发送的时候的代码如下：

```
Destination destination = session.createTopic("VirtualTopic.helloTopic");
```

　　生产者是这样的：

```
Destination destination = session.createQueue("VirtualTopicConsumers.A.VirtualTopic.helloTopic");
Destination destination = session.createQueue("VirtualTopicConsumers.B.VirtualTopic.helloTopic");
```

### ActiveMQ 静态网络配置：broker网络连接(broker的高性能方案)：

　　修改 activeMQ 服务器的 activeMQ.xml, 增加如下配置，这个配置只能实现单向连接，实现双向连接需要各个节点都配置如下配置。

```
<networkConnectors>``  ``<networkConnector uri=``"static://(tcp://192.168.254.135:61616,tcp://192.168.254.136:61616)"``/>``</networkConnectors>
```

　　两个 Brokers 通过一个 static 的协议来进行网络连接。一个 Consumer 连接到BrokerB 的一个地址上，当 Producer 在 BrokerA 上以相同的地址发送消息，此时消息会被转移到 BrokerB 上，也就是说 BrokerA 会转发消息到BrokerB 上。

 　在activeMQ中，进行了静态网络桥接的两台节点而言，当 Producer 在 BrokerA 上以相同的地址发送10条消息。一个 Consumer 连接到BrokerB去消费消息，当消费了一半的时候出现异常了，那么剩下来未处理的消息会被存放到 BrokerB 的待处理消息队列中，此时要通过BrokerA再去消费是消费不到的，万一此刻BrokerB 挂了，那么哪些没有消费的消息将会丢失。mq给我们提供了一个有效的消息回流机制。

```
<policyEntry queue=``">"` `enableAudit=``"false"``>``  ``<networkBridgeFilterFactory>``     ``<conditionalNetworkBridgeFilterFactory replayWhenNoConsumers=``"true"``/>``  ``</networkBridgeFilterFactory>``</policyEntry>
```

### ActiveMQ 的优缺点：

　　ActiveMQ 采用消息推送方式，所以最适合的场景是默认消息都可在短时间内被消费。数据量越大，查找和消费消息就越慢，消息积压程度与消息速度成反比。

　　**缺点：**

- 吞吐量低。由于 ActiveMQ 需要建立索引，导致吞吐量下降。这是无法克服的缺点，只要使用完全符合 JMS 规范的消息中间件，就要接受这个级别的TPS。
- 无分片功能。这是一个功能缺失，JMS 并没有规定消息中间件的集群、分片机制。而由于 ActiveMQ 是伟企业级开发设计的消息中间件，初衷并不是为了处理海量消息和高并发请求。如果一台服务器不能承受更多消息，则需要横向拆分。ActiveMQ 官方不提供分片机制，需要自己实现。

　　**适用场景：**

　　对 TPS 要求比较低的系统，可以使用 ActiveMQ 来实现，一方面比较简单，能够快速上手开发，另一方面可控性也比较好，还有比较好的监控机制和界面

　　**不适用的场景:**

　　消息量巨大的场景。ActiveMQ 不支持消息自动分片机制，如果消息量巨大，导致一台服务器不能处理全部消息，就需要自己开发消息分片功能。

# [ActiveMQ消息的持久化策略](https://www.cnblogs.com/wuzhenzhao/p/10084066.html)

### 持久化消息和非持久化消息的存储原理：

　　正常情况下，非持久化消息是存储在内存中的，持久化消息是存储在文件中的。能够存储的最大消息数据在${ActiveMQ_HOME}/conf/activemq.xml文件中的systemUsage节点SystemUsage配置设置了一些系统内存和硬盘容量。

```
<systemUsage>``  ``<systemUsage>``   ``<memoryUsage>``//该子标记设置整个ActiveMQ节点的“可用内存限制”。这个值不能超过ActiveMQ本身设置的最大内存大小。其中的``//percentOfJvmHeap属性表示百分比。占用70%的堆内存``     ``<memoryUsage percentOfJvmHeap=``"70"` `/>``       ``</memoryUsage>``        ``<storeUsage>``//该标记设置整个ActiveMQ节点，用于存储“持久化消息”的“可用磁盘空间”。该子标记的limit属性必须要进行设置``          ``<storeUsage limit=``"100 gb"``/>``        ``</storeUsage>``       ``<tempUsage>``//一旦ActiveMQ服务节点存储的消息达到了memoryUsage的限制，非持久化消息就会被转储到 temp store区域，虽然``//我们说过非持久化消息不进行持久化存储，但是ActiveMQ为了防止“数据洪峰”出现时非持久化消息大量堆积致使内存耗``//尽的情况出现，还是会将非持久化消息写入到磁盘的临时区域——temp store。这个子标记就是为了设置这个temp``//store区域的“可用磁盘空间限制``     ``<tempUsage limit=``"50 gb"``/>``   ``</tempUsage>`` ``</systemUsage>``</systemUsage>
```

　　Ø 从上面的配置我们需要得到一个结论，当非持久化消息堆积到一定程度的时候，也就是内存超过指定的设置阀值时，ActiveMQ会将内存中的非持久化消息写入到临时文件，以便腾出内存。但是它和持久化消息的区别是，重启之后，持久化消息会从文件中恢复，非持久化的临时文件会直接删除。

### 消息的持久化策略：

 　消息持久性对于可靠消息传递来说是一种比较好的方法，即时发送者和接受者不是同时在线或者消息中心在发送者发送消息后宕机了，在消息中心重启后仍然可以将消息发送出去。消息持久性的原理很简单，就是在发送消息出去后，消息中心首先将消息存储在本地文件、内存或者远程数据库，然后把消息发送给接受者，发送成功后再把消息从存储中删除，失败则继续尝试。接下来我们来了解一下消息在broker上的持久化存储实现方式。

### 持久化存储支持类型：

　　ActiveMQ支持多种不同的持久化方式，主要有以下几种，不过，无论使用哪种持久化方式，消息的存储逻辑都是一致的。

　　　　Ø KahaDB存储（默认存储方式）。

　　　　Ø JDBC存储。

　　　　Ø Memory存储。

　　　　Ø LevelDB存储。

　　　　Ø JDBC With ActiveMQ Journal。

#### KahaDB存储：

　　KahaDB是目前默认的存储方式,可用于任何场景,提高了性能和恢复能力。消息存储使用一个事务日志和仅仅用一个索引文件来存储它所有的地址。KahaDB是一个专门针对消息持久化的解决方案,它对典型的消息使用模式进行了优化。在Kaha中,数据被追加到data logs中。当不再需要log文件中的数据的时候,log文件会被丢弃。

 　配置方式：在${ActiveMQ_HOME}/conf/activemq.xml文件中：

```
<persistenceAdapter>``   ``<kahaDB directory=``"${activemq.data}/kahadb"``/>``</persistenceAdapter>
```

　　KahaDB的存储原理：

在data/kahadb这个目录下，会生成四个文件

Ø db.data 它是消息的索引文件，本质上是B-Tree（B树），使用B-Tree作为索引指向db-*.log里面存储的消息。
Ø db.redo 用来进行消息恢复。
Ø db-*.log 存储消息内容。新的数据以APPEND的方式追加到日志文件末尾。属于顺序写入，因此消息存储是比较快的。默认是32M，达到阀值会自动递增。
Ø lock文件 锁，表示当前获得kahadb读写权限的broker。

###  JDBC存储：

　　使用JDBC持久化方式，数据库会创建3个表：activemq_msgs，activemq_acks和activemq_lock。
　　ACTIVEMQ_MSGS ：消息表，queue和topic都存在这个表中
　　ACTIVEMQ_ACKS ：存储持久订阅的信息和最后一个持久订阅接收的消息ID
　　ACTIVEMQ_LOCKS ：锁表，用来确保某一时刻，只能有一个ActiveMQ broker实例来访问数据库

　　JDBC存储实践:

```
<persistenceAdapter>``  ``<jdbcPersistenceAdapter dataSource=``"# MySQL-DS "` `createTablesOnStartup=``"true"` `/>``</persistenceAdapter>
```

　　dataSource指定持久化数据库的bean，createTablesOnStartup是否在启动的时候创建数据表，默认值是true，这样每次启动都会去创建数据表了，一般是第一次启动的时候设置为true，之后改成false

　　Mysql持久化Bean配置:

```
<bean id=``"Mysql-DS"` `class``=``"org.apache.commons.dbcp.BasicDataSource"` `destroy-method=``"close"``>``  ``<property name=``"driverClassName"` `value=``"com.mysql.jdbc.Driver"``/>``  ``<property name=``"url"` `value=``"jdbc:mysql://192.168.11.156:3306/activemq?relaxAutoCommit=true"``/>``  ``<property name=``"username"` `value=``"root"``/>``  ``<property name=``"password"` `value=``"root"``/>``</bean>
```

　　配置完以后需要往 ${ActiveMQ_HOME}/lib 文件夹中添加相应 jar 包：然后重启就OK了。

![img](https://img2018.cnblogs.com/blog/1383365/201812/1383365-20181212110627266-797371899.png)

### LevelDB存储：

　　LevelDB持久化性能高于KahaDB，虽然目前默认的持久化方式仍然是KahaDB。并且，在ActiveMQ 5.9版本提供了基于LevelDB和Zookeeper的数据复制方式，用于Master-slave方式的首选数据复制方案。不过，据ActiveMQ官网对LevelDB的表述：LevelDB官方建议使用以及不再支持，推荐使用的是KahaDB。

```
<persistenceAdapter>``  ``<levelDBdirectory=``"activemq-data"``/>``</persistenceAdapter>
```

### Memory 消息存储：

　　基于内存的消息存储，内存消息存储主要是存储所有的持久化的消息在内存中。persistent=”false”,表示不设置持久化存储，直接存储到内存中。

```
<beans>``<broker brokerName=``"test-broker"` `persistent=``"false"``xmlns=``"http://activemq.apache.org/schema/core"``>``<transportConnectors>``<transportConnector uri=``"tcp://localhost:61635"``/>``</transportConnectors> </broker>``</beans>
```

### JDBC Message store with ActiveMQ Journal：

　　这种方式克服了JDBC Store的不足，JDBC每次消息过来，都需要去写库和读库。ActiveMQ Journal，使用高速缓存写入技术，大大提高了性能。当消费者的消费速度能够及时跟上生产者消息的生产速度时，journal文件能够大大减少需要写入到DB中的消息。举个例子，生产者生产了1000条消息，这1000条消息会保存到journal文件，如果消费者的消费速度很快的情况下，在journal文件还没有同步到DB之前，消费者已经消费了90%的以上的消息，那么这个时候只需要同步剩余的10%的消息到DB。如果消费者的消费速度很慢，这个时候journal文件可以使消息以批量方式写到DB。

Ø 将原来的 persistenceAdapter 标签注释掉
Ø 添加如下标签

```
<persistenceFactory>``  ``<journalPersistenceAdapterFactory dataSource=``"#Mysql-DS"` `dataDirectory=``"activemqdata"``/>``</persistenceFactory>
```

### 持久化消息和非持久化消息的发送策略：消息同步发送和异步发送

　　ActiveMQ支持同步、异步两种发送模式将消息发送到broker上。同步发送过程中，发送者发送一条消息会阻塞直到broker反馈一个确认消息，表示消息已经被broker处理。这个机制提供了消息的安全性保障，但是由于是阻塞的操作，会影响到客户端消息发送的性能。异步发送的过程中，发送者不需要等待broker提供反馈，所以性能相对较高。但是可能会出现消息丢失的情况。所以使用异步发送的前提是在某些情况下允许出现数据丢失的情况。

　　默认情况下，非持久化消息是异步发送的，持久化消息并且是在非事务模式下是同步发送的。但是在开启事务的情况下，消息都是异步发送。由于异步发送的效率会比同步发送性能更高。所以在发送持久化消息的时候，尽量去开启事务会话。除了持久化消息和非持久化消息的同步和异步特性以外，我们还可以通过以下几种方式来设置异步发送：

```
1``.ConnectionFactory connectionFactory=``new` `ActiveMQConnectionFactory(``"tcp://192.168.11.153:61616?jms.useAsyncSend=true"``);``2``.((ActiveMQConnectionFactory) connectionFactory).setUseAsyncSend(``true``);``3``.((ActiveMQConnection)connection).setUseAsyncSend(``true``);
```

### 消息发送的源码：

　　以producer.send为入口 进入的是ActiveMQSession 实现：

```
public` `void` `send(Destination destination, Message message, ``int` `deliveryMode, ``int` `priority, ``long` `timeToLive, AsyncCallback onComplete) ``throws` `JMSException {``    ``checkClosed(); ``//检查session的状态，如果session关闭则抛异常``    ``if` `(destination == ``null``) {``      ``if` `(info.getDestination() == ``null``) {``        ``throw` `new` `UnsupportedOperationException(``"A destination must be specified."``);``      ``}``      ``throw` `new` `InvalidDestinationException(``"Don't understand null destinations"``);``    ``}``    ``//检查destination的类型，如果符合要求，就转变为ActiveMQDestination``    ``ActiveMQDestination dest;``    ``if` `(destination.equals(info.getDestination())) {``      ``dest = (ActiveMQDestination)destination;``    ``} ``else` `if` `(info.getDestination() == ``null``) {``      ``dest = ActiveMQDestination.transform(destination);``    ``} ``else` `{``      ``throw` `new` `UnsupportedOperationException(``"This producer can only send messages to: "` `+ ``this``.info.getDestination().getPhysicalName());``    ``}``    ``if` `(dest == ``null``) {``      ``throw` `new` `JMSException(``"No destination specified"``);``    ``}` `    ``if` `(transformer != ``null``) {``      ``Message transformedMessage = transformer.producerTransform(session, ``this``, message);``      ``if` `(transformedMessage != ``null``) {``        ``message = transformedMessage;``      ``}``    ``}``    ``//如果发送窗口大小不为空，则判断发送窗口的大小决定是否阻塞``    ``if` `(producerWindow != ``null``) {``      ``try` `{``        ``producerWindow.waitForSpace();``      ``} ``catch` `(InterruptedException e) {``        ``throw` `new` `JMSException(``"Send aborted due to thread interrupt."``);``      ``}``    ``}``    ``//发送消息到broker的topic``    ``this``.session.send(``this``, dest, message, deliveryMode, priority, timeToLive, producerWindow, sendTimeout, onComplete);` `    ``stats.onMessage();``  ``}
```

　　ActiveMQSession的send方法，this.session.send(this, dest, message, deliveryMode, priority, timeToLive, producerWindow, sendTimeout, onComplete)：

```
protected` `void` `send(ActiveMQMessageProducer producer, ActiveMQDestination destination, Message message, ``int` `deliveryMode, ``int` `priority, ``long` `timeToLive,``            ``MemoryUsage producerWindow, ``int` `sendTimeout, AsyncCallback onComplete) ``throws` `JMSException {` `    ``checkClosed();``    ``if` `(destination.isTemporary() && connection.isDeleted(destination)) {``      ``throw` `new` `InvalidDestinationException(``"Cannot publish to a deleted Destination: "` `+ destination);``    ``}``    ``//互斥锁，如果一个session的多个producer发送消息到这里，会保证消息发送的有序性``    ``synchronized` `(sendMutex) {``      ``// tell the Broker we are about to start a new transaction``      ``doStartTransaction();``//告诉broker开始一个新事务，只有事务型会话中才会开启``      ``TransactionId txid = transactionContext.getTransactionId();``//从事务上下文中获取事务id``      ``long` `sequenceNumber = producer.getMessageSequence();` `      ``//Set the "JMS" header fields on the original message, see 1.1 spec section 3.4.11``      ``message.setJMSDeliveryMode(deliveryMode); ``//在JMS协议头中设置是否持久化标识``      ``long` `expiration = 0L;``//计算消息过期时间``      ``if` `(!producer.getDisableMessageTimestamp()) {``        ``long` `timeStamp = System.currentTimeMillis();``        ``message.setJMSTimestamp(timeStamp);``        ``if` `(timeToLive > ``0``) {``          ``expiration = timeToLive + timeStamp;``        ``}``      ``}``      ``message.setJMSExpiration(expiration);``//设置消息过期时间``      ``message.setJMSPriority(priority);``//设置消息的优先级``      ``message.setJMSRedelivered(``false``);;``//设置消息为非重发` `      ``// transform to our own message format here``      ``ActiveMQMessage msg = ActiveMQMessageTransformation.transformMessage(message, connection);``      ``msg.setDestination(destination);``      ``msg.setMessageId(``new` `MessageId(producer.getProducerInfo().getProducerId(), sequenceNumber));` `      ``// Set the message id.``      ``if` `(msg != message) {``//如果消息是经过转化的，则更新原来的消息id和目的地``        ``message.setJMSMessageID(msg.getMessageId().toString());``        ``// Make sure the JMS destination is set on the foreign messages too.``        ``message.setJMSDestination(destination);``      ``}``      ``//clear the brokerPath in case we are re-sending this message``      ``msg.setBrokerPath(``null``);` `      ``msg.setTransactionId(txid);``      ``if` `(connection.isCopyMessageOnSend()) {``        ``msg = (ActiveMQMessage)msg.copy();``      ``}``      ``msg.setConnection(connection);``      ``msg.onSend();``//把消息属性和消息体都设置为只读，防止被修改``      ``msg.setProducerId(msg.getMessageId().getProducerId());``      ``if` `(LOG.isTraceEnabled()) {``        ``LOG.trace(getSessionId() + ``" sending message: "` `+ msg);``      ``}``      ``//如果onComplete没有设置(这里传进来就是null)，且发送超时时间小于0，且消息不需要反馈，且连接器不是同步发送模式，且消息非持久化或者连接器是异步发送模式``      ``//或者存在事务id的情况下，走异步发送，否则走同步发送``      ``if` `(onComplete==``null` `&& sendTimeout <= ``0` `&& !msg.isResponseRequired() && !connection.isAlwaysSyncSend() && (!msg.isPersistent() || connection.isUseAsyncSend() || txid != ``null``)) {``        ``this``.connection.asyncSendPacket(msg);``        ``if` `(producerWindow != ``null``) {``          ``int` `size = msg.getSize();``//异步发送的情况下，需要设置producerWindow的大小``          ``producerWindow.increaseUsage(size);``        ``}``      ``} ``else` `{``        ``if` `(sendTimeout > ``0` `&& onComplete==``null``) {``          ``this``.connection.syncSendPacket(msg,sendTimeout);``//带超时时间的同步发送``        ``}``else` `{``          ``this``.connection.syncSendPacket(msg, onComplete);``//带回调的同步发送``        ``}``      ``}` `    ``}``  ``}
```

　　我们从上面的代码可以看到，在执行发送操作之前需要把消息做一个转化，并且将我们设置的一些属性注入导指定的属性中，我们先来看看异步发送，会发现异步发送的时候涉及到producerWindowSize的大小：

ProducerWindowSize的含义

　　producer每发送一个消息，统计一下发送的字节数，当字节数达到ProducerWindowSize值时，需要等待broker的确认，才能继续发送。

　　主要用来约束在异步发送时producer端允许积压的(尚未ACK)的消息的大小，且只对异步发送有意义。每次发送消息之后，都将会导致memoryUsage大小增加(+message.size)，当broker返回producerAck时，memoryUsage尺寸减少(producerAck.size，此size表示先前发送消息的大小)。

可以通过如下2种方式设置:
Ø 在brokerUrl中设置: "tcp://localhost:61616?jms.producerWindowSize=1048576",这种设置将会对所有的producer生效。
Ø 在destinationUri中设置: "myQueue?producer.windowSize=1048576",此参数只会对使用此Destination实例的producer生效，将会覆盖brokerUrl中的producerWindowSize值。
注意：此值越大，意味着消耗Client端的内存就越大。

　　接下去我们进入异步发送流程，看看消息是怎么异步发送的this.connection.asyncSendPacket(msg)：

```
private` `void` `doAsyncSendPacket(Command command) ``throws` `JMSException {``    ``try` `{``      ``this``.transport.oneway(command);``    ``} ``catch` `(IOException e) {``      ``throw` `JMSExceptionSupport.create(e);``    ``}``  ``}
```

　　这里的 Command 其实就是之前一步所转化的message ，并且经过一系列的属性注入。因为ActiveMQMessage 继承了 baseCommand ，该类实现了 Command 。所以可以转化，然后我们发现 oneway 方法又很多的实现，都是基于 transport ，那么我们就需要来看看这个 transport 是什么。这里我们把代码往前翻并没有发现他的初始化，按照我们以往的思路，这里就会在初始化连接的时候进行初始化该对象：

```
ConnectionFactory connectionFactory = ``new` `ActiveMQConnectionFactory(``"tcp://192.168.254.135:61616"``);``Connection connection= connectionFactory.createConnection();
```

　　这里进入 ActiveMQConnectionFactory 的 createConnection方法会来到：

```
protected` `ActiveMQConnection createActiveMQConnection(String userName, String password) ``throws` `JMSException {``    ``if` `(brokerURL == ``null``) {``      ``throw` `new` `ConfigurationException(``"brokerURL not set."``);``    ``}``    ``ActiveMQConnection connection = ``null``;``    ``try` `{``// 果然发现了这个东东的初始化``      ``Transport transport = createTransport();``      ``// 创建连接``      ``connection = createActiveMQConnection(transport, factoryStats);``      ``// 设置用户密码``      ``connection.setUserName(userName);``      ``connection.setPassword(password);``      ``// 对连接做包装``      ``configureConnection(connection);``      ``// 启动一个后台传输线程``      ``transport.start();``      ``// 设置客户端消费的id``      ``if` `(clientID != ``null``) {``        ``connection.setDefaultClientID(clientID);``      ``}` `      ``return` `connection;``    ``} ......``  ``}
```

　　这里我们发现了 Transport transport = createTransport(); 这就是他的初始化：我们可以发现

```
protected` `Transport createTransport() ``throws` `JMSException {``    ``try` `{``      ``URI connectBrokerUL = brokerURL;``      ``String scheme = brokerURL.getScheme();``      ``if` `(scheme == ``null``) {``        ``throw` `new` `IOException(``"Transport not scheme specified: ["` `+ brokerURL + ``"]"``);``      ``}``      ``if` `(scheme.equals(``"auto"``)) {``        ``connectBrokerUL = ``new` `URI(brokerURL.toString().replace(``"auto"``, ``"tcp"``));``      ``} ``else` `if` `(scheme.equals(``"auto+ssl"``)) {``        ``connectBrokerUL = ``new` `URI(brokerURL.toString().replace(``"auto+ssl"``, ``"ssl"``));``      ``} ``else` `if` `(scheme.equals(``"auto+nio"``)) {``        ``connectBrokerUL = ``new` `URI(brokerURL.toString().replace(``"auto+nio"``, ``"nio"``));``      ``} ``else` `if` `(scheme.equals(``"auto+nio+ssl"``)) {``        ``connectBrokerUL = ``new` `URI(brokerURL.toString().replace(``"auto+nio+ssl"``, ``"nio+ssl"``));``      ``}` `      ``return` `TransportFactory.connect(connectBrokerUL);``    ``} ``catch` `(Exception e) {``      ``throw` `JMSExceptionSupport.create(``"Could not create Transport. Reason: "` `+ e, e);``    ``}``  ``}
```

　　　这里有点类似于基于URL驱动的意思，这里进来先是构建一个 URI ，根据URL去创建一个连接TransportFactory.connect，会发现默认使用的是tcp的协议。这里由于我们在创建连接的时候就已经指定了tcp所以这里的判断都没用，直接进入创建连接TransportFactory.connect(connectBrokerUL)：

```
public` `static` `Transport connect(URI location) ``throws` `Exception {``    ``TransportFactory tf = findTransportFactory(location);``    ``return` `tf.doConnect(location);``  ``}
```

　　这里做连接需要创建一个 tf 对象。这就要看看findTransportFactory(location) ：

```
public` `static` `TransportFactory findTransportFactory(URI location) ``throws` `IOException {``    ``String scheme = location.getScheme();``    ``if` `(scheme == ``null``) {``      ``throw` `new` `IOException(``"Transport not scheme specified: ["` `+ location + ``"]"``);``    ``}``    ``TransportFactory tf = TRANSPORT_FACTORYS.get(scheme);``    ``if` `(tf == ``null``) {``      ``// Try to load if from a META-INF property.``      ``try` `{``        ``tf = (TransportFactory)TRANSPORT_FACTORY_FINDER.newInstance(scheme);``        ``TRANSPORT_FACTORYS.put(scheme, tf);``      ``} ``catch` `(Throwable e) {``        ``throw` `IOExceptionSupport.create(``"Transport scheme NOT recognized: ["` `+ scheme + ``"]"``, e);``      ``}``    ``}``    ``return` `tf;``  ``}
```

　　不难理解以上的 代码是根据 scheme通过TRANSPORT_FACTORYS 这个map 来创建的 TransportFactory ，如果获取不到，就会通过TRANSPORT_FACTORY_FINDER 去获取一个实例。TRANSPORT_FACTORY_FINDER 这个FINDER是什么东西呢？ 我们看看他的初始化：

```
private` `static` `final` `FactoryFinder TRANSPORT_FACTORY_FINDER = ``new` `FactoryFinder(``"META-INF/services/org/apache/activemq/transport/"``);
```

　　我们通过源码中指定路径以下的东西：

![img](https://img2018.cnblogs.com/blog/1383365/201812/1383365-20181211165227328-896449961.png)

 

 　这有点类似于 java 中SPI规范的意思。我们可以看看 tcp 其中的内容：

```
class``=org.apache.activemq.transport.tcp.TcpTransportFactory
```

　　这里是键值对的方式，上述获取实例的代码中其实就是获取一个 TcpTransportFactory 实例，那么我们就知道tf.doConnect(location) 是哪个实现类做的，就是TcpTransportFactory，但是我们点开一看并未发现 TcpTransportFactory实现，这就说明该类使用的是父类里面的方法，这里就是TransportFactory 类：

```
public` `Transport doConnect(URI location) ``throws` `Exception {``    ``try` `{``      ``Map<String, String> options = ``new` `HashMap<String, String>(URISupport.parseParameters(location));``      ``if``( !options.containsKey(``"wireFormat.host"``) ) {``        ``options.put(``"wireFormat.host"``, location.getHost());``      ``}``      ``WireFormat wf = createWireFormat(options);``      ``//创建一个Transport 这里才是我们要找的真相``      ``Transport transport = createTransport(location, wf);``      ``//配置configure，这个里面是对Transport做链路包装，思想类似于dubbo的cluster``      ``Transport rc = configure(transport, wf, options);``      ``//remove auto``      ``IntrospectionSupport.extractProperties(options, ``"auto."``);` `      ``if` `(!options.isEmpty()) {``        ``throw` `new` `IllegalArgumentException(``"Invalid connect parameters: "` `+ options);``      ``}``      ``return` `rc;``    ``} ``catch` `(URISyntaxException e) {``      ``throw` `IOExceptionSupport.create(e);``    ``}``  ``}
```

　　我们进入 createTransport(location, wf) 方法，这里是使用Tcp子类的实现。会发现里面创建了一个 Sokect 连接 ，这就是准备后来进行发送的Sokect。然后这里返回的 Transport 就是 TcpTransport .接下去就是对这个 transport 进行包装 configure(transport, wf, options)：

```
public` `Transport configure(Transport transport, WireFormat wf, Map options) ``throws` `Exception {``//组装一个复合的transport，这里会包装两层，一个是IactivityMonitor.另一个是WireFormatNegotiator``    ``transport = compositeConfigure(transport, wf, options);``    ``//再做一层包装,MutexTransport``    ``transport = ``new` `MutexTransport(transport);``    ``//包装ResponseCorrelator``    ``transport = ``new` `ResponseCorrelator(transport);``    ``return` `transport;``  ``}
```

　　到目前为止，这个transport实际上就是一个调用链了，他的链结构为ResponseCorrelator(MutexTransport(WireFormatNegotiator(IactivityMonitor(TcpTransport()))每一层包装表示什么意思呢？

　　ResponseCorrelator 用于实现异步请求。
　　MutexTransport 实现写锁，表示同一时间只允许发送一个请求
　　WireFormatNegotiator 实现了客户端连接broker的时候先发送数据解析相关的协议信息，比如解析版本号，是否使用缓存等
　　InactivityMonitor 用于实现连接成功成功后的心跳检查机制，客户端每10s发送一次心跳信息。服务端每30s读取一次心跳信息。

　　通过这层层的分析，我们回到 ActiveMQConnection 发送消息的doAsyncSendPacket 方法：

```
private` `void` `doAsyncSendPacket(Command command) ``throws` `JMSException {``    ``try` `{``      ``this``.transport.oneway(command);``    ``} ``catch` `(IOException e) {``      ``throw` `JMSExceptionSupport.create(e);``    ``}``  ``}
```

　　这里的 oneway(command)方法会先后经历上述调用链的处理最后调用到 TcpTransport 的oneway(command) ，我们一步一步来看看都做了些什么：

　　ResponseCorrelator.oneway(command):里面就设置了两个属性

```
public` `void` `oneway(Object o) ``throws` `IOException {``    ``Command command = (Command)o; ``//对前面的对象做一个强转，组装一些信息``    ``command.setCommandId(sequenceGenerator.getNextSequenceId());``    ``command.setResponseRequired(``false``);``    ``next.oneway(command);``  ``}
```

　　MutexTransport.oneway(command):

```
public` `void` `oneway(Object command) ``throws` `IOException {``    ``writeLock.lock();``// 通过 ReentrantLock做加锁``    ``try` `{``      ``next.oneway(command);``    ``} ``finally` `{``      ``writeLock.unlock();``    ``}``  ``}
```

　　WireFormatNegotiator.oneway(command):这个里面调用了父类的 oneway ，父类是 TransportFilter 类

```
public` `void` `oneway(Object command) ``throws` `IOException {``    ``boolean` `wasInterrupted = Thread.interrupted();``    ``try` `{``      ``if` `(readyCountDownLatch.getCount() > ``0` `&& !readyCountDownLatch.await(negotiateTimeout, TimeUnit.MILLISECONDS)) {``        ``throw` `new` `IOException(``"Wire format negotiation timeout: peer did not send his wire format."``);``      ``}``    ``} ``catch` `(InterruptedException e) {``      ``InterruptedIOException interruptedIOException = ``new` `InterruptedIOException(``"Interrupted waiting for wire format negotiation"``);``      ``interruptedIOException.initCause(e);``      ``try` `{``        ``onException(interruptedIOException);``      ``} ``finally` `{``        ``Thread.currentThread().interrupt();``        ``wasInterrupted = ``false``;``      ``}``      ``throw` `interruptedIOException;``    ``} ``finally` `{``      ``if` `(wasInterrupted) {``        ``Thread.currentThread().interrupt();``      ``}``    ``}``    ``super``.oneway(command); ``//里面没做什么事情进入下一个调用链``  ``}
```

　　从WireFormatNegotiator的父类TransportFilter进入下一个调用链应该调用的是InactivityMonitor.oneway(command)，可是并未发现又该类实现，所以这里进入InactivityMonitor 的父类AbstractInactivityMonitor：

```
public` `void` `oneway(Object o) ``throws` `IOException {``    ``// To prevent the inactivity monitor from sending a message while we``    ``// are performing a send we take a read lock. The inactivity monitor``    ``// sends its Heart-beat commands under a write lock. This means that``    ``// the MutexTransport is still responsible for synchronizing sends``    ``sendLock.readLock().lock();``//获取发送读锁 锁定``    ``inSend.set(``true``);``//设置属性``    ``try` `{``      ``doOnewaySend(o);``//通过这个逻辑进入下一个调用链``    ``} ``finally` `{``      ``commandSent.set(``true``);``      ``inSend.set(``false``);``      ``sendLock.readLock().unlock();``    ``}``  ``}
```

　　在doOnewaySend 里面的next.oneway(command) 方法最终调用 TcpTransport 的实现：

```
public` `void` `oneway(Object command) ``throws` `IOException {``    ``checkStarted();``    ``//进行格式化内容 通过Sokct 发送``    ``wireFormat.marshal(command, dataOut);``    ``// 流的刷新``    ``dataOut.flush();``}
```

　　最后通过Sokect进行数据的传输。这样子异步发送的流程就结束了。下面来走一下同步的流程：通过this.connection.syncSendPacket() 进入同步发送流程。

```
public` `Response syncSendPacket(Command command, ``int` `timeout) ``throws` `JMSException {``    ``if` `(isClosed()) {``      ``throw` `new` `ConnectionClosedException();``    ``} ``else` `{` `      ``try` `{``// 进行发送，阻塞获取结果``        ``Response response = (Response)(timeout > ``0``            ``? ``this``.transport.request(command, timeout)``            ``: ``this``.transport.request(command));``        ``if` `(response.isException()) {``          ``ExceptionResponse er = (ExceptionResponse)response;``          ``if` `(er.getException() ``instanceof` `JMSException) {``            ``throw` `(JMSException)er.getException();``          ``}``         ``。。。。。。。。。``        ``return` `response;``      ``} ``catch` `(IOException e) {``        ``throw` `JMSExceptionSupport.create(e);``      ``}``    ``}``  ``}
```

　　这里的 transport 跟异步发送过程中的transport时一样的，即 ResponseCorrelator(MutexTransport(WireFormatNegotiator(IactivityMonitor(TcpTransport())) 一个调用链，进入ResponseCorrelator 的实现：

```
public` `Object request(Object command, ``int` `timeout) ``throws` `IOException {``    ``FutureResponse response = asyncRequest(command, ``null``);``    ``return` `response.getResult(timeout);``  ``}
```

　　从这个方法我们可以得到的信息时，在发送的时候采用的是 asyncRequest 方法，意思是异步请求，但是在下行采用  response.getResult(timeout) 去同步阻塞的方式去获取结果：

```
public` `Response getResult(``int` `timeout) ``throws` `IOException {``    ``final` `boolean` `wasInterrupted = Thread.interrupted();``    ``try` `{``      ``Response result = responseSlot.poll(timeout, TimeUnit.MILLISECONDS);``       ``.........      ``  ``}
```

　　这里会从 ArrayBlockingQueue 去 阻塞的处理请求。其实这里的同步发送实质上采用的不阻塞发送，阻塞的去等待broker 的反馈结果。

　　最后整理一下这个发送流程图

![img](https://img2018.cnblogs.com/blog/1383365/201812/1383365-20181213165753505-2032843567.png)

### 延迟和定时消息投递（Delay and Schedule Message Delivery）：

　　有时候我们不希望消息马上被broker投递出去，而是想要消息60秒以后发给消费者，或者我们想让消息没隔一定时间投递一次，一共投递指定的次数。。。类似这种需求，ActiveMQ提供了一种broker端消息定时调度机制。我们只需要把几个描述消息定时调度方式的参数作为属性添加到消息，broker端的调度器就会按照我们想要的行为去处理消息。当然需要在xml中配置schedulerSupport属性为true（broker的属性）即：<broker schedulerSupport="true">

使用延迟消息必须遵守如下配置属性：属性名称 类型 描述

1. AMQ_SCHEDULED_DELAY （long） 消息延迟时间单位：毫秒
2. AMQ_SCHEDULED_PERIOD（ long） 消息发送周期单位时间：毫秒。如 5秒一次 配置 AMQ_SCHEDULED_PERIOD = 5*1000
3. AMQ_SCHEDULED_REPEAT （int） 消息重复发送次数
4. AMQ_SCHEDULED_CRON （string） 使用Cron 表达式 设置定时发送

　　延迟60秒发送消息

```
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
long time = 60 * 1000;
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, time);
producer.send(message);
```

　　开始延迟30秒发送，重复发送10次，每次之间间隔10秒

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
long delay = 30 * 1000;
long period = 10 * 1000;
int repeat = 9;
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, repeat);
producer.send(message);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　使用Cron 表示式定时发送消息

```
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
message.setStringProperty(ScheduledMessage.AMQ_SCHEDULED_CRON, "0 * * * *");
producer.send(message);
```

　　Cron 的优先级大于消息延迟，只要设置了Cron 表达式会优先执行Cron规则，如下：消息定时发送10次，每个小时执行，延迟1秒之后发送。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
message.setStringProperty(ScheduledMessage.AMQ_SCHEDULED_CRON, "0 * * * *");
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, 1000);
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, 1000);
message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, 9);
producer.send(message);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

# [初识ActiveMQ及整合springboot](https://www.cnblogs.com/wuzhenzhao/p/10084045.html)

### 消息中间件的初步认识

什么是消息中间件？

　　消息中间件是利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。通过提供消息传递和消息排队模型，可以在分布式架构下扩展进程之间的通信。

消息中间件能做什么？

　　消息中间件主要解决的就是分布式系统之间消息传递的问题，它能够屏蔽各种平台以及协议之间的特性，实现应用程序之间的协同。举个非常简单的例子，就拿一个电商平台的注册功能来简单分析下，用户注册这一个服务，不单单只是 insert 一条数据到数据库里面就完事了，还需要发送激活邮件、发送新人红包或者积分、发送营销短信等一系列操作。假如说这里面的每一个操作，都需要消耗 1s，那么整个注册过程就需要耗时 4s 才能响应给用户。

#### ActiveMQ 简介

ActiveMQ 是完全基于 JMS 规范实现的一个消息中间件产品。是 Apache 开源基金会研发的消息中间件。ActiveMQ主要应用在分布式系统架构中，帮助构建高可用、高性能、可伸缩的企业级面向消息服务的系统ActiveMQ 特性

\1. 多语言和协议编写客户端

　　语言：java/C/C++/C#/Ruby/Perl/Python/PHP

　　应用协议 :

　　openwire/stomp/REST/ws/notification/XMPP/AMQP

\2. 完全支持 jms1.1 和 J2ee1.4 规范

\3. 对 spring 的支持，ActiveMQ 可以很容易内嵌到 spring模块中

#### ActiveMQ 安装

\1. 登录到 http://activemq.apache.org/components/classic/download/，找到 ActiveMQ 的下载地址 

　我这里用的是apache-activemq-5.15.10-bin.tar.gz ，jdk是1.8.0_161

\2. 直 接 copy 到 服 务 器 上 通 过 tar -zxvf apache-activeMQ.tar.gz
\3. 启动运行
　　a) 普通启动：到 bin 目录下， sh activemq start
　　b) 启 动 并 指 定 日 志 文 件 sh activemq start > /tmp/activemqlog
\4. 检查是否已启动
　　ActiveMQ默认采用 61616 端口提供 JMS服务，使用 8161端口提供管理控制台服务，执行以下命令可以检查是否成功启动 ActiveMQ 服务
　　netstat -an|grep 61616

　　可以通过./activemq console来查看日志。
\5. 通过 http://192.168.11.156:8161 访问 activeMQ 管理页面 ，默认帐号密码 admin/admin
\6. 关闭 ActiveMQ; sh activemq stop

#### 下面来看一下ActiveMQ的简单应用：

　　消息的发布：

```
public` `static` `void` `main(String[] args) {``    ` `    ``ConnectionFactory connectionFactory = ``new` `ActiveMQConnectionFactory(``"tcp://192.168.254.135:61616"``);``    ``Connection connection = ``null``;``    ``try` `{` `      ``connection = connectionFactory.createConnection();``      ``connection.start();``      ``// 延迟确认``      ``Session session = connection.createSession(Boolean.TRUE, Session.DUPS_OK_ACKNOWLEDGE);``      ``// 创建目的地``      ``Destination destination = session.createQueue(``"myQueue"``);``      ``// 创建消费者``      ``MessageProducer producer = session.createProducer(destination);``      ``TextMessage message = session.createTextMessage(``"Hello World"``);``      ``producer.send(message);``      ``// 表示消息被自动确认``      ``session.commit();``      ``session.close();``    ``} ``catch` `(JMSException e) {``      ``e.printStackTrace();``    ``} ``finally` `{``      ``if` `(connection != ``null``) {``        ``try` `{``          ``connection.close();``        ``} ``catch` `(JMSException e) {``          ``e.printStackTrace();``        ``}``      ``}``    ``}``  ``}
```

　　对应的客户端消费：

```
public` `static` `void` `main(String[] args) {``    ``ConnectionFactory connectionFactory = ``new` `ActiveMQConnectionFactory(``"tcp://192.168.254.135:61616"``);``    ``Connection connection = ``null``;``    ``try` `{` `      ``connection = connectionFactory.createConnection();``      ``connection.start();``      ``// 延迟确认``      ``Session session = connection.createSession(Boolean.TRUE, Session.DUPS_OK_ACKNOWLEDGE);``      ``// 创建目的地``      ``Destination destination = session.createQueue(``"myQueue"``);``      ``// 创建消费者``      ``MessageConsumer consumer = session.createConsumer(destination);``      ``TextMessage textMessage = (TextMessage) consumer.receive();``      ``System.out.println(textMessage.getText());``      ``// 表示消息被自动确认``      ``session.commit();``      ``session.close();``    ``} ``catch` `(JMSException e) {``      ``e.printStackTrace();``    ``} ``finally` `{``      ``if` `(connection != ``null``) {``        ``try` `{``          ``connection.close();``        ``} ``catch` `(JMSException e) {``          ``e.printStackTrace();``        ``}``      ``}``    ``}``  ``}
```

　　如果需要做到消息监听的话：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
        ConnectionFactory connectionFactory=
                new ActiveMQConnectionFactory
                        ("tcp://192.168.1.101:61616");
        Connection connection=null;
        try {

            connection=connectionFactory.createConnection();
            connection.start();

            Session session=connection.createSession
                    (Boolean.TRUE,Session.AUTO_ACKNOWLEDGE);
            //创建目的地
            Destination destination=session.createQueue("myQueue");
            //创建发送者
            MessageConsumer consumer=session.createConsumer(destination);


            MessageListener messageListener=new MessageListener() {
                @Override
                public void onMessage(Message message) {
                    try {
                        System.out.println(((TextMessage)message).getText());
                        session.commit();
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            };

            consumer.setMessageListener(messageListener);
            System.in.read();
        } catch (JMSException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(connection!=null){
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　基于订阅发布的消息发送：

```
public` `static` `void` `main(String[] args) {``    ``ConnectionFactory connectionFactory=``        ``new` `ActiveMQConnectionFactory``            ``(``"tcp://192.168.254.135:61616"``);``    ``Connection connection=``null``;``    ``try` `{``      ``connection=connectionFactory.createConnection();``      ``connection.start();``      ``Session session=connection.createSession(Boolean.TRUE,Session.AUTO_ACKNOWLEDGE);``      ``//创建目的地``      ``Destination destination=session.createTopic(``"myTopic"``);``      ``//创建发送者``      ``MessageProducer producer=session.createProducer(destination);``      ``producer.setDeliveryMode(DeliveryMode.PERSISTENT);``      ``//创建需要发送的消息``      ``TextMessage message=session.createTextMessage(``"topic -message"``);``      ``//Text  Map Bytes Stream Object``      ``producer.send(message);``      ``session.commit();``//      session.rollback();``      ``session.close();``    ``} ``catch` `(JMSException e) {``      ``e.printStackTrace();``    ``}``finally` `{``      ``if``(connection!=``null``){``        ``try` `{``          ``connection.close();``        ``} ``catch` `(JMSException e) {``          ``e.printStackTrace();``        ``}``      ``}``    ``}``  ``}
```

　　基于订阅发布的消息消费：这里需要先启动消费者

```
public` `static` `void` `main(String[] args) {``    ``ConnectionFactory connectionFactory=``        ``new` `ActiveMQConnectionFactory``            ``(``"tcp://192.168.254.135:61616"``);``    ``Connection connection=``null``;``    ``try` `{``      ``connection=connectionFactory.createConnection();``      ``connection.setClientID(``"wuzz"``);``      ``connection.start();``      ``Session session=connection.createSession``          ``(Boolean.TRUE,Session.AUTO_ACKNOWLEDGE);``      ``//创建目的地``      ``Topic destination=session.createTopic(``"myTopic"``);``      ``//创建发送者``      ``MessageConsumer consumer=session.createDurableSubscriber(destination,``"wuzz"``);``      ``TextMessage textMessage=(TextMessage) consumer.receive();``      ``System.out.println(textMessage.getText());``      ``session.commit(); ``//消息被确认``      ``session.close();``    ``} ``catch` `(JMSException e) {``      ``e.printStackTrace();``    ``}``finally` `{``      ``if``(connection!=``null``){``        ``try` `{``          ``connection.close();``        ``} ``catch` `(JMSException e) {``          ``e.printStackTrace();``        ``}``      ``}``    ``}``  ``}
```

　　明白了ActiveMQ的基本使用，下面从源码的层面去学习一下ActIiveMQ的原理

### springboot整合ActiveMQ:

1.pom.xml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions><!-- 去掉springboot默认配置 -->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency> <!-- 引入log4j2依赖 -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-all</artifactId>
            <version>5.15.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.0</version>
        </dependency>
    </dependencies>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2.application.yml:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
server:
  port: 8881

spring:
  activemq:
    broker-url: tcp://192.168.1.101:61616
    user: admin
    password: admin
    pool:
      enabled: true
    packages:
      trust-all: true   # 如果使用ObjectMessage传输对象，必须要加上这个信任包，否则会报ClassNotFound异常
  jms:
    pub-sub-domain: true  # 启动主题消息
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

3.ActiveMqConfig 配置类：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Configuration
public class ActiveMqConfig {

    @Value("${spring.activemq.broker-url}")
    private String brokerUrl;

    @Value("${spring.activemq.user}")
    private String username;

    @Value("${spring.activemq.password}")
    private String password;


    @Bean
    public ConnectionFactory connectionFactory() {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(username, password, brokerUrl);
//        RedeliveryPolicy policy=new RedeliveryPolicy();
//        policy.setUseExponentialBackOff(Boolean.TRUE);
//        policy.setMaximumRedeliveries(2);
//        policy.setInitialRedeliveryDelay(1000L);
//        activeMQConnectionFactory.setRedeliveryPolicy(policy);
        return connectionFactory;


    }

    // queue模式的ListenerContainer
    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerQueue(ConnectionFactory connectionFactory) {
        DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
        //使用异步发送
//        ((ActiveMQConnectionFactory) activeMQConnectionFactory).setUseAsyncSend(true);
        //使用异步发送
//        ((ActiveMQConnection)activeMQConnectionFactory).setUseAsyncSend(true);
        //手动应答
        bean.setSessionAcknowledgeMode(4);
        bean.setConnectionFactory(connectionFactory);
        return bean;
    }
    
    // topic模式的ListenerContainer
    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerTopic(ConnectionFactory connectionFactory) {
        DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
        bean.setPubSubDomain(true);
        bean.setConnectionFactory(connectionFactory);
        return bean;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

\4. MqProducer 生产者：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Service
public class MqProducer {


    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    /**
     * 发送字符串消息队列
     *
     * @param queueName 队列名称
     * @param message   字符串
     */
    public void sendStringQueue(String queueName, String message) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQQueue(queueName), message);
    }

    /**
     * 发送字符串集合消息队列
     *
     * @param queueName 队列名称
     * @param list      字符串集合
     */
    public void sendStringListQueue(String queueName, List<String> list) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQQueue(queueName), list);
    }

    /**
     * 发送对象消息队列
     *
     * @param queueName 队列名称
     * @param obj       对象
     */
    public void sendObjQueue(String queueName, Serializable obj) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQQueue(queueName), obj);
    }

    /**
     * 发送对象集合消息队列
     *
     * @param queueName 队列名称
     * @param objList   对象集合
     */
    public void sendObjListQueue(String queueName, List<Serializable> objList) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQQueue(queueName), objList);
    }

    /**
     * 发送字符串消息主题
     *
     * @param topicName 主题名称
     * @param message   字符串
     */
    public void sendStringTopic(String topicName, String message) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQTopic(topicName), message);
    }

    /**
     * 发送字符串集合消息主题
     *
     * @param topicName 主题名称
     * @param list      字符串集合
     */
    public void sendStringListTopic(String topicName, List<String> list) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQTopic(topicName), list);
    }

    /**
     * 发送对象消息主题
     *
     * @param topicName 主题名称
     * @param obj       对象
     */
    public void sendObjTopic(String topicName, Serializable obj) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQTopic(topicName), obj);
    }

    /**
     * 发送对象集合消息主题
     *
     * @param topicName 主题名称
     * @param objList   对象集合
     */
    public void sendObjListTopic(String topicName, List<Serializable> objList) {
        this.jmsMessagingTemplate.convertAndSend(new ActiveMQTopic(topicName), objList);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

4.队列消费者 QueueConsumer：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Component
public class QueueConsumer {

    @JmsListener(destination = "stringQueue", containerFactory = "jmsListenerContainerQueue")
    public void consumer(ActiveMQMessage message, Session session) throws Exception {
        System.out.println(session.getAcknowledgeMode());
        System.out.println(session.getTransacted());
        System.out.println("接受队列消息，内容为：{}" + message);
        message.acknowledge();
    }

//    @JmsListener(destination = "stringQueue", containerFactory = "jmsListenerContainerQueue")
//    public void receiveStringQueue(String msg) {
//        System.out.println("接收到消息...." + msg);
//    }


//    @JmsListener(destination = "stringListQueue", containerFactory = "jmsListenerContainerQueue")
//    public void receiveStringListQueue(List<String> list) {
//        System.out.println("接收到集合队列消息...." + list);
//    }
//
//
//    @JmsListener(destination = "objQueue", containerFactory = "jmsListenerContainerQueue")
//    public void receiveObjQueue(ObjectMessage objectMessage) throws Exception {
//        System.out.println("接收到对象队列消息...." + objectMessage.getObject());
//    }
//
//
//    @JmsListener(destination = "objListQueue", containerFactory = "jmsListenerContainerQueue")
//    public void receiveObjListQueue(ObjectMessage objectMessage) throws Exception {
//        System.out.println("接收到的对象队列消息..." + objectMessage.getObject());
//    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

5.主题消费者A ，这里为了测试topic消息，我们使用两个消费者去订阅。ATopicConsumer：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Component
public class ATopicConsumer {

    @JmsListener(destination = "stringTopic", containerFactory = "jmsListenerContainerTopic")
    public void receiveStringTopic(String msg) {
        System.out.println("ATopicConsumer接收到消息...." + msg);
    }


//    @JmsListener(destination = "stringListTopic", containerFactory = "jmsListenerContainerTopic")
//    public void receiveStringListTopic(List<String> list) {
//        System.out.println("ATopicConsumer接收到集合主题消息...." + list);
//    }
//
//
//    @JmsListener(destination = "objTopic", containerFactory = "jmsListenerContainerTopic")
//    public void receiveObjTopic(ObjectMessage objectMessage) throws Exception {
//        System.out.println("ATopicConsumer接收到对象主题消息...." + objectMessage.getObject());
//    }
//
//
//    @JmsListener(destination = "objListTopic", containerFactory = "jmsListenerContainerTopic")
//    public void receiveObjListTopic(ObjectMessage objectMessage) throws Exception {
//        System.out.println("ATopicConsumer接收到的对象集合主题消息..." + objectMessage.getObject());
//    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　BTopicConsumer：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Component
public class BTopicConsumer {

    @JmsListener(destination = "stringTopic", containerFactory = "jmsListenerContainerTopic")
    public void receiveStringTopic(String msg) {
        System.out.println("BTopicConsumer接收到消息...." + msg);
    }

//    @JmsListener(destination = "stringListTopic", containerFactory = "jmsListenerContainerTopic")
//    public void receiveStringListTopic(List<String> list) {
//        System.out.println("BTopicConsumer接收到集合主题消息...." + list);
//    }
//
//
//    @JmsListener(destination = "objTopic", containerFactory = "jmsListenerContainerTopic")
//    public void receiveObjTopic(ObjectMessage objectMessage) throws Exception {
//        System.out.println("BTopicConsumer接收到对象主题消息...." + objectMessage.getObject());
//    }
//
//
//    @JmsListener(destination = "objListTopic", containerFactory = "jmsListenerContainerTopic")
//    public void receiveObjListTopic(ObjectMessage objectMessage) throws Exception {
//        System.out.println("BTopicConsumer接收到的对象集合主题消息..." + objectMessage.getObject());
//    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

6.实体类 User:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class User implements Serializable {

    private String id;
    private String name;
    private Integer age;

    public User() {
    }

    public User(String id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
　　//省略get set 跟 toString
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

7.测试类：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
public class TestController {

    @Autowired
    private MqProducer mqProducer;

    @RequestMapping(value = "/testStringQueue.json", method = {RequestMethod.GET})
    public void testStringQueue() {

        for (int i = 1; i <= 100; i++) {
            System.out.println("第" + i + "次发送字符串队列消息");
            mqProducer.sendStringQueue("stringQueue", "消息：" + i);
        }
    }


//    @RequestMapping(value = "/testStringListQueue.json", method = {RequestMethod.GET})
//    public void testStringListQueue() {
//
//        List<String> idList = new ArrayList<>();
//        idList.add("id1");
//        idList.add("id2");
//        idList.add("id3");
//
//        System.out.println("正在发送集合队列消息ing......");
//        mqProducer.sendStringListQueue("stringListQueue", idList);
//    }
//
//
//    @RequestMapping(value = "/testObjQueue.json", method = {RequestMethod.GET})
//    public void testObjQueue() {
//
//        System.out.println("正在发送对象队列消息......");
//        mqProducer.sendObjQueue("objQueue", new User("1", "小明", 20));
//    }
//
//
//    @RequestMapping(value = "/testObjListQueue.json", method = {RequestMethod.GET})
//    public void testObjListQueue() {
//
//        System.out.println("正在发送对象集合队列消息......");
//
//        List<Serializable> userList = new ArrayList<>();
//        userList.add(new User("1", "小明", 21));
//        userList.add(new User("2", "小雪", 22));
//        userList.add(new User("3", "小花", 23));
//
//        mqProducer.sendObjListQueue("objListQueue", userList);
//    }

    @RequestMapping(value = "/testStringTopic.json", method = {RequestMethod.GET})
    public void testStringTopic() {

        for (int i = 1; i <= 100; i++) {
            System.out.println("第" + i + "次发送字符串主题消息");
            mqProducer.sendStringTopic("stringTopic", "消息：" + i);
        }
    }

//    @RequestMapping(value = "/testStringListTopic.json", method = {RequestMethod.GET})
//    public void testStringListTopic() {
//
//        List<String> idList = new ArrayList<>();
//        idList.add("id1");
//        idList.add("id2");
//        idList.add("id3");
//
//        System.out.println("正在发送集合主题消息ing......");
//        mqProducer.sendStringListTopic("stringListTopic", idList);
//    }
//
//
//    @RequestMapping(value = "/testObjTopic.json", method = {RequestMethod.GET})
//    public void testObjTopic() {
//
//        System.out.println("正在发送对象主题消息......");
//        mqProducer.sendObjTopic("objTopic", new User("1", "小明", 20));
//    }
//
//
//    @RequestMapping(value = "/testObjListTopic.json", method = {RequestMethod.GET})
//    public void testObjListTopic() {
//
//        System.out.println("正在发送对象集合主题消息......");
//
//        List<Serializable> userList = new ArrayList<>();
//        userList.add(new User("1", "小明", 21));
//        userList.add(new User("2", "小雪", 22));
//        userList.add(new User("3", "小花", 23));
//
//        mqProducer.sendObjListTopic("objListTopic", userList);
//    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　启动后访问对应接口就可以。

# [ActiveMq性能优化](https://www.cnblogs.com/zhishan/archive/2012/11/09/2762672.html)

ActiveMq运行是比较稳定的，数据的吞吐速度也很高，如果出现入队列或者出队列慢的问题，先检查一下自己的代码，是不是本身取到数据后处理过慢。

本文的关于性能优化，其实是列举出一些需要注意的点，请确保你的项目没有一下问题：

\1. 使用spring的JmsTemplate

 JmsTemplate的send和convertAndSend会使用持久化mode，即使你设置了NON_PERSISTENT。这会导致入队列速度变得非常慢。

 解决办法，使用下面的MyJmsTemplate代替JmsTemplate。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyJmsTemplate extends JmsTemplate {
    private Session session;

    public MyJmsTemplate() {
        super();
    }

    public MyJmsTemplate(ConnectionFactory connectionFactory) {
        super(connectionFactory);
    }

    public void doSend(MessageProducer producer, Message message) throws JMSException {
        if (isExplicitQosEnabled()) {
            producer.send(message, getDeliveryMode(), getPriority(), getTimeToLive());
        } else {
            producer.send(message);
        }
    }

    public Session getSession() {
        return session;
    }

    public void setSession(Session session) {
        this.session = session;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

\2. DeliveryMode的选择，如果你入队列的数据，不考虑MQ挂掉的情况（这概率很小），使用NON_PERSISTENT会显著提高数据写入速度。

\3. 生产者使用事物会提高入队列性能，但是消费者如果启动了事物则会显著影响数据的消费速度。相关代码如下：

```
    Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```

代码中的false代表不启动事物。

\4. 消费者的消息处理即onMessage方法优化，举例如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class SmsMoPool implements MessageListener {
    private final static Logger logger = LoggerFactory.getLogger(SmsMoPool.class);
    private DefaultEventPubliser moEventPublisher;
    private final EventFactory eventFactory = new DefaultEventFactory();
    private DefaultDataGather dataGather;
    private ExecutorService pool = Executors.newFixedThreadPool(5);

    @Override
    public void onMessage(final Message message) {
        pool.execute(new Runnable() {
            @Override
            public void run() {
                final ObjectMessage msg = (ObjectMessage) message;
                Serializable obj = null;
                try {
                    obj = msg.getObject();
                } catch (JMSException e) {
                    logger.error("从消息队列获得上行信息异常{}", e);
                }
                if (obj != null) {
                    dataGather.incrementDateCount(MasEntityConstants.TRAFFIC_SMS_MO_IN);
                    AgentToServerReq req = (AgentToServerReq) obj;
                    if (logger.isInfoEnabled()) {
                        logger.info("驱动-->调度：{}", req.toXmlStr());
                    }
                    Event event = eventFactory.createMoEvent(req);
                    moEventPublisher.publishEvent(event);
                }
            }
        });
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这段代码使用了线程池，另一点要注意的是msg.getObject();这个方法是一个比较耗时的方法，你的代码中不应该出现多次getObject()。

\5. 消费者使用预取，如何使用预取，下面以spring版本为例

```
  <bean class="org.apache.activemq.command.ActiveMQQueue">
                <constructor-arg value="data.mo?consumer.prefetchSize=100"/>
            </bean>
```

预取数量根据具体入队列数据而定，以上设置100，是针对2000/sec入队列速度设定的。
另外如果是慢消费者，这里可设置为1。

\6. 检查你的MQ数据吞吐速度，保持生产和消费的平衡，不会出现大量积压。

\7. ActiveMQ使用TCP协议时 tcpNoDelay=默认是false ，设置为true可以提高性能。

还是spring版本的：

```
 <bean id="mqPoolConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <bean id="mqConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory" p:useAsyncSend="true"
                  p:brokerURL="failover://(tcp://127.0.0.1:61616?tcpNoDelay=true)"/>
        </property>
    </bean>
```

## 前言

MQ是现在大型系统架构中必不可少的一个重要中间件，之前有偏文章[《MQ(消息队列)常见的应用场景解析》](http://www.17ij.com/mquse/)介绍过MQ的应用场景，现在流行的几个MQ是rabbitmq,rocketma,kafka,这几个MQ比较最容易找到相关的文章，而也有些系统使用的是activemq，因activemq是相对比较传统的MQ，在使用过程中还是会遇到很多坑，这里简单列举几个大家可能会遇到的问题，把自己使用acitvemq的经验和大家分享一下。

## Mysql 持久化

现在大家使用MQ，基本都是会把数据进行持久化，MQ默认存储持久化数据使用kahaDB，但是鉴于大家对mysql比较熟悉，很多人会选择mysql进行数据的持久化，因为mysql查看数据还是比较方便的。如果需要把持久化方式改为mysql，则需要修改如下配置：

```xml
 <persistenceAdapter>
            <jdbcPersistenceAdapter dataDirectory="${activemq.data}" dataSource="#mysql-ds" createTablesOnStartup="false" useDatabaseLock="false"/>
            <!-- 下面是默认的kahaDB方式，注释掉 -->
	    <!-- <kahaDB directory="${activemq.data}/kahadb"/> -->
        </persistenceAdapter>
```

这里的配置有几个地方大家需要关注下：

| 配置            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| dataDirectory   | 需要配置和broker 的dataDirectory 一致                        |
| dataSource      | 数据源的选择，关联数据库的具体配置，下文会具体说明           |
| useDatabaseLock | 是否使用数据库锁，主要是在程序启动的时候会同步查询数据，导致数据库锁 |

还需要配置数据库的连接、账号、密码等：

```xml
 <!-- MySql DataSource  Setup -->
    <bean id="mysql-ds" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    	<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    	<property name="url" value="jdbc:mysql://0.0.0.0:3306/activemq?relaxAutoCommit=true"/>
    	<property name="username" value="root"/>
    	<property name="password" value="******"/>
    	<property name="poolPreparedStatements" value="true"/>
    </bean>
```

其中，id 名和上文提到的datasource应该是一样的。否则，不知道连接哪个实例。

### 数据库连接池问题

启动activemq如果提示数据库的连接池有问题，这可能是少了lib，增加

- mysql-connector-java-5.1.30.jar
- commons-dbcp2-2.1.1.jar
- commons-pool2-2.4.2.jar

三个包，放到lib目录即可

### 管理界面无法打开

如果正常启动了，但是管理界面无法启动，那么需要修改下管理界面的数据库连接。

> 使用MQ主要原因之一是MQ性能比传统关系数据库性能要好，但是把MQ数据存储的mysql其实不是一个很好的选择，反其道而行之，虽然这样用的团队不少，但是强烈推荐不要这么做。还是用默认的存储方式，确保性能为主。

## activeMQ过期配置

前文说过，activemq性能本来就不是最优的，特别是使用了mysql作为数据库存储工具后，性能更加不靠谱，所以性能优化，是个重要的工作，定期清理MQ的过期信息，就显的非常重要了。

### 定期清理无效的队列

配置如下：

```xml
<destinationPolicy>
            <policyMap>
              <policyEntries>


                <policyEntry queue=">" gcInactiveDestinations="true" inactiveTimoutBeforeGC="10000">
                <deadLetterStrategy>
                    <sharedDeadLetterStrategy processExpired="true" expiration="30000"/>
                </deadLetterStrategy>
                 </policyEntry>



                <policyEntry topic=">" gcInactiveDestinations="true" inactiveTimoutBeforeGC="10000" >
                   
                  <pendingMessageLimitStrategy>
                    <constantPendingMessageLimitStrategy limit="1000"/>
                  </pendingMessageLimitStrategy>
                </policyEntry>

              </policyEntries>
              
            </policyMap>
        </destinationPolicy>
```

定期自动清理无效的Topic和Queue,这个配置，只会清除设置的时间内，没有被订阅，同时队列没有遗留数据的队列。
同时，对于boker节点，需要设置schedulePeriodForDestinationPurge 参数，表示多长之间执行一次检测。

```xml
 <broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost01" dataDirectory="${activemq.data}" 
    useJmx="true" schedulePeriodForDestinationPurge="5000">
```

### 设置消息的全局过期时间

开发的时候，大家应该都知道可以设置消息的过期时间，是否有统一设置消息的过期时间呢？
在broker节点下增加如下的配置：

```xml
  <plugins>
          <!-- 86400000 为一天，设置为10天过期 -->
            <timeStampingBrokerPlugin ttlCeiling="10000"
            zeroExpirationOverride="10000" />
        </plugins>
```

为了便于测试，我设置的是10s,当然，生产环境根据自己的是实际设置的会比较长。过期的时间会进入死信，死信也会沿用此时间，到期后，系统就会自动删除信息了。
经过我个人的实践经验，MQ积累的数据达千万级别后，性能下降的比较厉害，定期清理MQ的消息，是优化性能非常重要的一个操作。

## 总结

现如今，MQ的选择很多，建议还是优先选择rabbitmq、rocketmq或者是kafka，如果已经选择activemq，需要持续关注MQ的消费情况，最好能设置过期时间，定期清理消息队列的数据，避免数据的积累，造成性能的下降。







参考资料

https://www.cnblogs.com/55zjc/p/15975284.html
