[TOC]

# 消息队列历史

> 早期消息队列是解决通信解耦问题的

A、B两个系统，进行数据通信，A要将数据传递给B实现计算，B要将计算结果返回给A使用

![](https://note.youdao.com/yws/api/personal/file/97753B5BC9514F10B789258DD501DD66?method=download&shareKey=8756de7c20a88be00e23a0e1f68580e3)

无论A是传递数据，还是B返回计算结果，有任何一步失败，都会导致整个通信流程重新来。引入消息队列后将有效改善这个问题

![](https://note.youdao.com/yws/api/personal/file/357FAE59404D40748E68008F41597BD6?method=download&shareKey=966ca153aa9e3638a6ea4560a79f5b54)



# RabbitMQ

rabbitmq是企业级的消息队列。它实现了SUN公司提出的AMQP（advanced message queue protocol）协议，是一个高级的消息队列

> 消息队列产品：
>
> - rabbitmq
> - rocketmq
> - kafka
> - JMS：性能比较低



# rabbitmq的结构



## 客户端角色

rabbitmq支持很多语言，可以连接rabbitmq成为客户端，角色不同取决于客户端使用rabbitmq实现的功能

1. ==生产端==：发送消息的客户端
2. ==消费端==：连接消息队列，接收消费者发送的消息的客户端



## 核心组件

> 连接组件：长连接，短连接

![](https://note.youdao.com/yws/api/personal/file/A7671D5D82CE438AB7CCB636BF2AD800?method=download&shareKey=1668abd40b50783598207f62f0644ca3)

### 长连接

长连接基于TCP/IP协议，创建长连接比较消耗资源，所以不能频繁创建、销毁。

==一般一个进程都会绑定一个长连接去使用==



### 短连接

短连接基于长连接，消耗资源非常有限。客户端使用完后可以关闭、销毁



### 交换机：`exchange`

rabbitmq要在内存中保存大量的队列，交换机完成对发送过来的消息的高并发处理。

并发稳定性取决于客户端代码

exchange组件是使用erlang语言实现的

![](https://note.youdao.com/yws/api/personal/file/F4B32D7440F14B22AFC0BDD0755331D4?method=download&shareKey=955620469b9d82a740c219763d0f1d63)



### 队列：`queue`

是rabbitmq存储消息的组件，可以在内存中存储，也可以实现持久化。

队列从生产端发送到的交换机中获取消息

消费端绑定监听队列，实现消费



# 安装和启动

## 安装

> rabbitmq的exchange组件是erlang语言开发的，安装rabbitmq之前，要安装对应高版本的erlang环境

![](https://note.youdao.com/yws/api/personal/file/5D1A302CFC29438890C4F71C22F3CF5C?method=download&shareKey=2daeb5f5d40dedae21746353d7760e02)

```sh
[root@10-9-104-184 resources]# rpm -ivh rabbitmq-server-3.7.7-1.el6.noarch.rpm 
```

> rpm安装后，安装文件在`/usr/lib/rabbitmq/`中



## 启动

> rabbitmq默认用户：guest；密码：guest
>
> 默认只允许本地登录使用这个用户



### 调整环境配置

1. 开启web控制台插件

   > rabbitmq提供了一个web控制台，需要在启动rabbitmq之前开启控制台插件

   ```sh
   [root@10-9-104-184 bin]# rabbitmq-plugins enable rabbitmq_management
   ```

   ![](https://note.youdao.com/yws/api/personal/file/F9839F4647B84CA190E371D930ADE8C3?method=download&shareKey=3e97812447c40b4fa5686447745e5bc3)

   

2. 将用户权限打开

   > rabbitmq有一个模板配置文件
   >
   > `/usr/share/doc/rabbitmq-server-3.7.7/rabbitmq.config.example`
   >
   > 将这个文件拷贝到`/etc/rabbitmq`，并重命名`rabbitmq.config`

   ```sh
   [root@10-9-104-184 rabbitmq-server-3.7.7]# cp 
   	/usr/share/doc/rabbitmq-server-3.7.7/rabbitmq.config.example 
   	/etc/rabbitmq/rabbitmq.config
   ```

   > 修改`/etc/rabbitm1/rabbitmq.config`配置文件的61行，将user限制打开，参数为空，表示没有限制

   ![](https://note.youdao.com/yws/api/personal/file/35082F031BC042EBB6758615EA850115?method=download&shareKey=d7393965f48eb9a768bd5855c743c567)



### 启动

> 方式一：通过服务的方式启动
>
> - 打开：start
> - 关闭：stop
> - 重启：restart

```sh
[root@10-9-104-184 resources]# service rabbitmq-server start 
```



> 方式二：通过命令脚本启动
>
> - 启动命令后加`&`号，后台运行，通过ps查看PID，使用kill关闭
> - 直接启动，CTRL+C关闭

先进入rabbitmq根目录的bin文件夹

`/usr/lib/rabbitmq/bin`

```sh
[root@10-9-104-184 bin]# ./rabbitmq-server start
[root@10-9-104-184 bin]# ./rabbitmq-server -detached
```

![](https://note.youdao.com/yws/api/personal/file/232E2516AB684ADEBD877278DD2DD98A?method=download&shareKey=aa17939c7161c70bf1957a1e09dc96b9)



### 登录web控制台页面

> rabbitmq服务成功启动后，通过浏览器：`服务器ip:15672`访问rabbitmq的控制台页面

![](https://note.youdao.com/yws/api/personal/file/74D05FD2C1FF4904B9C8AC6B64760FA5?method=download&shareKey=6d67403006ccbeda7675284dcdb51884)

[TOC]

# 五种工作模式

> 消息队列使用过程中，可以根据不同场景选择不同的消息队列模式
>
> - 简单模式
> - 争抢模式
> - 发布订阅模式
> - 路由模式
> - 主题模式



## 简单模式

> 强调的不是生产端发送消息的逻辑，强调的是消费端监听的逻辑

![](https://note.youdao.com/yws/api/personal/file/AF271C0605014B98911D74588EFD27EA?method=download&shareKey=93615b92204b999ac1697d2c558362c5)

- 生产端：封装整理消息，连接rabbitmq，将消息发送给交换机
- 交换机：接收所有当前连接的生产端的消息，根据消息的参数，决定后端连接哪个队列发送该消息
- 队列：生成时就会绑定交换机，负责存储消息
- ==消费端==：简单模式中，一个队列只被一个消费端监听，当队列有消息时，消费者就会拿走消息进行处理



> 应用场景：邮件、短信、聊天



## 争抢模式（也叫工作模式）

> 强调的是多个消费者同时监听一个队列，形成消息的争抢

![](https://note.youdao.com/yws/api/personal/file/1AA77EEEBB0D492B92B975AA16FBD432?method=download&shareKey=63ab18bd17e95bef8e6f205a71da3889)

- 生成端：发送消息到交换机
- 交换机：转发消息到指定队列
- 队列：存储消息
- 消费端：多个消费端同时连接同一个队列实现消息监听，形成争抢的结构（哪个消费端空闲率高，线程资源多，就更有机会抢到消息）



> 应用场景：抢红包，资源分配系统

![](https://note.youdao.com/yws/api/personal/file/5602E08F13FF4FA18B8FB640A05B2D41?method=download&shareKey=bfeeaec19790057e4c8aadee47c4228d)



## 发布订阅模式

> 发送一条消息，同步到多个队列中
>
> 强调生产端、交换机发送消息的逻辑

![](https://note.youdao.com/yws/api/personal/file/34D6EF568E32431F96B98C32D222E813?method=download&shareKey=664c289adca6cf554a9472e48f3f2bcc)

- 生产端：将消息发送给交换机
- 交换机：将消息同步复制发送到所有与该交换机绑定的队列，交换机类型：`fanout`
- 队列：存储消息
- 消费端：可能是简单模式，也可能是争抢模式



> 应用场景：邮件群发、广告的发送





## 路由模式

> 路由模式是rabbitmq==消息队列中最常见的一种方式==，可以根据生产端的消息参数决定当前消息该发给哪个队列，不发给哪些队列
>
> 路由：从起点到目的地的路线，就是路由规则的定义

![](https://note.youdao.com/yws/api/personal/file/423E13518D9E442D95AF770787C38A2B?method=download&shareKey=a4855c2141f8f350f7adce6447f49bdb)

- 生产端：发送消息，消息中携带具体的路由key：`routingKey`
- 交换机：接收到消息后
  - 会在与该交换机绑定的队列中匹配路由key，匹配到就将消息发送给这个队列
  - 匹配不到，消息就在交换机中直接删除
- 队列：存储消息
- 消费端：简单模式或者争抢模式



![](https://note.youdao.com/yws/api/personal/file/FF736409AE4B4D718B63D4E390E3CDDF?method=download&shareKey=1649f0f913c36a54819fd73bce3ca225)

> 👆👆👆👆👆如上图所示，当生产者发送消息的路由键为error时，两个队列均可以收到消息；当生产者发送消息的路由键为info或warning时，只有第二个队列可收到消息。



> 应用场景：指定邮箱发送邮件、系统error消息路由





## 主题模式

> 将路由key进行多级划分，实现多级传递的一种使用方式
>
> 交换机类型：`topic`

![](https://note.youdao.com/yws/api/personal/file/DF9C0E6148EA48C58F35D382953ACB2A?method=download&shareKey=8beb004351be1e4b152e93978015f178)



和路由模式使用逻辑类似，都是对消息中的路由key与绑定的队列key进行匹配，区别在于主题模式的队列使用的路由key可以不是具体的内容，可以是一个范围的匹配

| 符号 | 解释           |
| ---- | -------------- |
| *    | 匹配一个字符串 |
| #    | 匹配多级字符串 |



> 例子：消息的目的地：中国.北京.朝阳

| 队列                | 能否接收消息                                |
| ------------------- | ------------------------------------------- |
| queue1(`中国.#`)    | true                                        |
| queue2(`*.*.朝阳`） | true                                        |
| queue3(`中国.*`)    | false<br />中国.上海，中国.新疆    可以接收 |



![](https://note.youdao.com/yws/api/personal/file/EA56B5E62C1648408639EC5F3B3AB99F?method=download&shareKey=f1d123a776ea1a23825eaa34139b2cfd)

> 以上图为例
>
> Q1队列的路由键为`*.orange.*`,Q2队列的路由键为`*.*.rabbit`和`lazy.#`
>
> - 当消息的路由键为`quick.orange.rabbit`时，两个队列均能收到
> - 当消息的路由键为`lazy.orange.male.rabbit`时，只有Q2队列能收到。



> 应用场景：多级消息传递的同类匹配

![](https://note.youdao.com/yws/api/personal/file/94185E3BE14046F0BFD67DC6E5E3685F?method=download&shareKey=9a3d630a7da56554e11483b020709660)







# Java代码实现五种模式

> 使用springboot整合依赖：`spring-boot-starter-amqp`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



## 初始化连接对象

```java
//实现生产端或者消费端之前，创建连接对象
private Channel channel;
//给channel赋值
@Before
public void initChannel() throws IOException, TimeoutException {
    //获取连接工厂对象，赋值连接信息 ip port user pw vh
    ConnectionFactory factory=new ConnectionFactory();
    factory.setHost("10.9.104.184");
    factory.setPort(5672);
    //factory.setUsername("guest");//默认用户名就是guest
    //factory.setPassword("guest");//默认密码就是guest
    //factory.setVirtualHost("/");//默认值就是：/

    Connection connection = factory.newConnection();
    channel=connection.createChannel();
}
```



## 简单模式

> 可以使用默认的交换机（AMQP default）：路由模式
>
> 代码中，默认交换机的名称为：`""`
>
> 这个交换机的存在，是为了防止任何一个队列在生成时，没有绑定其他交换机，会默认绑定到这个AMPQ default交换机上，默认的路由key就是队列的名称
>
> ![](https://note.youdao.com/yws/api/personal/file/9D524573E1B5447CA397E2B73D000908?method=download&shareKey=89faf6e244dc45d9a2dece97b0f4668d)



### 生产者

> 将一个消息文本msg发送到默认的AMQP default交换机上，并携带一个路由key（队列的名称）

```java
//生产端代码：将消息发送到默认的交换机：AMQP default，携带一个路由key（存在的队列名称）
@Test
public void send() throws IOException {
    String msg = "多日不见，甚是想念，今夜三更，后山见面"
    //声明一个队列,队列不存在时创建
    channel.queueDeclare("小红",false,false,false,null);
    //发送
    channel.basicPublish("", "小红", null, msg.getBytes());
}
```

> 声明一个队列：

 `channel.queueDeclare("小红",false,false,false,null);`

| 参数位置                                 | 接收类型 | 作用                                                         |
| ---------------------------------------- | -------- | ------------------------------------------------------------ |
| 第一个参数queue                          | String   | ==队列名称==                                                 |
| 第二个参数durable(持久的)[ˈdjʊərəbl]     | Boolean  | ==队列是否支持持久化==（web控制台中显示的`D`表示队列持久化）<br />不持久化，在rabbitmq重启后，队列就会消失 |
| 第三个参数exclusive(专属的)[ɪkˈskluːsɪv] | Boolean  | ==队列是否专属==（专属于当前Connection）<br />true：操作连接这个队列，只有当前这个Connection才能进 |
| 第四个参数autoDelete                     | Boolean  | ==是否自动删除==<br />true：当所有订阅该队列的消费者都解除了订阅，该队列会被删除 |
| 第五个参数args                           | Map      | ==创建队列的各种属性==，例如消息存活的最长时间、队列保存的最多消息个数 |

![](https://note.youdao.com/yws/api/personal/file/39B882109D1A4BF59AD5E8D46CA711CF?method=download&shareKey=ecfbc5d05c57291182952a7fd0558cc8)



> 发送消息：

`channel.basicPublish("", "小红", null, msg.getBytes());`

| 参数位置             | 接收类型        | 作用                                         |
| -------------------- | --------------- | -------------------------------------------- |
| 第一个参数exchange   | String          | ==交换机名称==                               |
| 第二个参数routingKey | String          | ==路由Key==，默认交换机中路由Key就是队列名称 |
| 第三个参数props      | BasicProperties | ==消息的属性==                               |
| 第四个参数body       | Byte[]          | ==消息体的二进制数据==                       |



### 消费者

> Consumer：消费者 [kənˈsjuːmə(r)] 

```java
//消费端代码：
@Test
public void consumer() throws IOException, InterruptedException {
    //创建消费对象
    //consumer绑定channel后，就具备了绑定queue的能力
    QueueingConsumer consumer = new QueueingConsumer(channel);

    //绑定消费对象和队列：小红，实现一对一监听
    channel.basicConsume("小红", true, consumer);

    //监听获取消息信息
    QueueingConsumer.Delivery delivery = consumer.nextDelivery();//从监听的队列中获取一个消息

    //delivery接收的消息体body以外，还有header、property
    System.out.println(new String(delivery.getBody()));

}
```

> 绑定消费对象

`channel.basicConsume("小红", true, consumer);`

| 参数位置           | 接收类型 | 作用                                                         |
| ------------------ | -------- | ------------------------------------------------------------ |
| 第一个参数queue    | String   | ==队列名称==                                                 |
| 第二个参数autoAck  | Boolean  | ==是否自动回执==<br />回执：告诉rabbitmq是否消费成功<br />成功返回true：rabbitmq会将该消息删除<br />失败返回false：该消失继续保存在队列中 |
| 第三个参数callback | Consumer | ==绑定的消费者==                                             |



## 争抢模式

> 多个消费者监听同一个队列，当消费者点击某个按钮时，在队列中获取一个数据

### 生产者

```java
@Test
public void send() throws IOException {
    String msg="小红情书：我就喜欢你每次给我送的大白薯";
    channel.queueDeclare("小红",false,false,false,null);
    for(int i=0;i<100;i++) {
        channel.basicPublish("", "小红", null, msg.getBytes());
    }
}
```



### 消费者

```java
//消费端逻辑
@Test
public void consumer01() throws Exception {

    //创建出来消费端对象
    QueueingConsumer consumer=new QueueingConsumer(channel);
    channel.basicConsume("小红",true,consumer);

    while(true){
        QueueingConsumer.Delivery delivery = consumer.nextDelivery();//从监听的队列中获取一个

        //存在的消息
        //delivery接收的除了消息体body以外还有header properties
        System.out.println("阿强拿到消息："+new String(delivery.getBody()));//body是二进制数组
    }
}

//消费端逻辑

@Test
public void consumer02() throws Exception {
    //创建出来消费端对象
    QueueingConsumer consumer=new QueueingConsumer(channel);
    channel.basicConsume("小红",true,consumer);
    
    while(true){
        QueueingConsumer.Delivery delivery = consumer.nextDelivery();//从监听的队列中获取一个

        //存在的消息
        //delivery接收的除了消息体body以外还有header properties
        System.out.println("阿明拿到消息："+new String(delivery.getBody()));//body是二进制数组
    }
}
```





## 发布订阅模式

> 同步发送到多个队列执行群发，使用已有的交换机`amp.fanout`也可以实现发布订阅
>
> 也可以自定义交换机，自定义绑定关系，实现消息的发送

### 生产者

```java
//交换机类型
private static final String type = "fanout";
//交换机名称
private static final String exName = type + "Ex";
//队列名称
private static final String q1 = "queue01" + type;
private static final String q2 = "queue02" + type;

@Test
public void send() throws IOException {
    //准消息
    String msg = "hello " + type;

    //声明交换机 exName type
    channel.exchangeDeclare(exName, type);

    //声明队列，多个队列，同时绑定一个fanout交换机
    channel.queueDeclare(q1,false,false,false,null);
    channel.queueDeclare(q2,false,false,false,null);

    //绑定
    channel.queueBind(q1,exName,"keke");
    channel.queueBind(q2,exName,"keke");

    //发送消息
    channel.basicPublish(exName,"keke",null,msg.getBytes());
}
```



### 消费者

> 可以是简单模式，也可以是争抢模式



## 路由模式

> 队列是详细的路由绑定，消息携带路由key，交换机匹配队列的路由进行消息的发送

### 生产者

```java
//交换机类型
private static final String type = "direct";
//交换机名称
private static final String exName = type + "Ex";
//队列名称
private static final String q1 = "queue01" + type;
private static final String q2 = "queue02" + type;

@Test
public void send() throws IOException {

    //准消息
    String msg = "hello " + type;

    //声明交换机 exName type
    channel.exchangeDeclare(exName, type);

    //声明队列，多个队列，同时绑定一个fanout交换机
    channel.queueDeclare(q1,false,false,false,null);
    channel.queueDeclare(q2,false,false,false,null);

    //绑定
    channel.queueBind(q1,exName,"北京");
    channel.queueBind(q1,exName,"北戴河");
    channel.queueBind(q1,exName,"哈尔滨");
    channel.queueBind(q2,exName,"上海");

    //发送消息
    channel.basicPublish(exName,"哈尔滨",null,msg.getBytes());
}
```



## 主题模式

> Topic主题模式和routing路由模式类似，只不过这里的交换机使用的是topic类型，topic类型的交换机和direct的不同就在于topic可以匹配通配符。
>
> - `*`代表匹配一个元素
> - `#`代表匹配一个或多个元素

### 生产者

```java
//准备几个静态常量
private static final String type="topic";//交换机类型
private static final String exName=type+"Ex";//交换机名称
private static final String q1="queue01"+type;
private static final String q2="queue02"+type;

@Test
public void send() throws IOException {

    //准消息
    String msg = "hello " + type;

    //声明交换机 exName type
    channel.exchangeDeclare(exName, type);

    //声明队列，多个队列，同时绑定一个fanout交换机
    channel.queueDeclare(q1,false,false,false,null);
    channel.queueDeclare(q2,false,false,false,null);

    //绑定
    channel.queueBind(q1,exName,"中国.#");
    channel.queueBind(q2,exName,"中国.上海.*");

    //发送消息
    channel.basicPublish(exName,"中国.北京.朝阳",null,msg.getBytes());
}
```

[TOC]

# 秒杀系统

## 整体逻辑

![](https://note.youdao.com/yws/api/personal/file/9301541EA0B548C9998D48DDEB158A3C?method=download&shareKey=02052ae98c4a3b7248b23414c153dd71)

> - 接收前端请求的系统，尽量提升单位时间并发，减少线程处理任务的时间。作为生产端发送消息。
>
> - 消息携带信息精简准确，谁秒杀了什么商品。userId/seckillId
>
> - 消费端可以启动在多个进程中，并发争抢处理消息，实现数据层的判断减库存入库。



## 秒杀减库存权限问题

为了避免抢购的商品没有库存，后面的用户（上百万）抢购操作依然连库判断，使数据库压力增大，可以在Redis缓存中生成一个同商品数量的list【seckill_oppo:1,1,1,......】

每一个有权限入库的用户，都是在Redis中获取到了数据的。没有获取数据的就直接return

![](https://note.youdao.com/yws/api/personal/file/256D319EDC2544EF9C331455D1103580?method=download&shareKey=e0889f43b8c4adc232b5702ed353887a)





# 代码编写

## 系统框架的搭建

1. springboot整合rabbitmq

   - 方式一：使用springboot自动配置

     > RabbitmqAutoConfiguration类
     >
     > 自动读取属性，创建连接，使用时，通过注入RabbitTemplate来实现消费者功能

   - 方式二：自定义配置

     > 通过注解标识类
     >
     > @Configuration
     >
     > @ConfigurationProperties("xx")
     >
     > @Bean创建对象 ConnectionFactory
     >
     > 生产端，消费端注入实现发送，监听消费
     >
     > 消费端异步监听

2. `application.properties`

   ```properties
   #rabbitmq
   spring.rabbitmq.host=10.9.104.184
   spring.rabbitmq.virtual-host=/
   spring.rabbitmq.username=guest
   spring.rabbitmq.password=guest
   ```

3. 生产端

   > 注入RabbitTemplate使用

   - `convertAndSend`：关注的是消息body作为源数据发送，发送到哪

     - exchange：已经存在的交换机名称
     - routingKey：消息路由key
     - msg：消息体，可以是对象，可以是String

     ```java
     @RestController
     public class SenderController {
         @Autowired
         private RabbitTemplate rabbitTemplate;
         //接收请求，发送消息
         @RequestMapping("/send")
         public String sendMsg(String msg) {
             //直接将消息作为消息提内容，让template发送
             rabbitTemplate.convertAndSend("directEx", "北京", msg);
         }
     }
     ```

     

   - `send`：关注的是消息本身携带的各种属性

     - exchange：交换机名称
     - routingKey：路由key
     - Message：封装了消息体和消息的属性
       - byte[]：消息体的字节数组
       - MessageProperties：消息的属性

     ```java
     @RestController
     public class SenderController {
     
         @Autowired
         private RabbitTemplate rabbitTemplate;
     
         //接收请求，发送消息
         @RequestMapping("/send")
         public String sendMsg(String msg) {
             //发送方法二：消息需要携带一些属性
             MessageProperties properties = new MessageProperties();
             properties.setPriority(100);
             properties.setUserId("110");
             Message message = new Message(msg.getBytes(), properties);
             rabbitTemplate.send("directEx", "北京",message);
         }
     }
     ```

     

## 声明新的交换机和队列

> 使用RabbitTemplate，只能使用已经存在的交换机和队列

要在系统启动后，生成需要的交换机和队列

- 通过配置类，加载到容器内存对象：Queue表示队列
- 通过配置类，加载到容器内存对象：Exchange表示交换机
- 通过@Bean交给容器管理

```java
/*
实现一个配置类编写
通过声明大量的对象，使得连接rabbitmq的底层
conncetion可以创建很多组件内容
和调用底层代码queueDeclear exchangeDeclear效果一样
 */

@Configuration
public class QueueCompConfig {

    //声明队列对象
    @Bean
    public Queue queue01(){
        //springboot在底层通过连接
        //调用queueDeclear name false false false null
        return new Queue("seckill01",false,false,false,null);
    }

    @Bean
    public Queue queue02(){
        //springboot在底层通过连接
        //调用queueDeclear name false false false null
        return new Queue("seckill02",false,false,false,null);
    }

    //配置声明交换机对象
    @Bean
    public DirectExchange exD1(){
        return new DirectExchange("seckillD01");//默认不自动删除，默认持久化
    }

    //配置声明交换机对象
    @Bean
    public DirectExchange exD2(){
        return new DirectExchange("seckillD02");//默认不自动删除，默认持久化
    }

    //绑定关系，通过返回代码对象实现 Binding
    @Bean
    public Binding bind01(){
        return BindingBuilder.bind(queue01()).to(exD1()).with("seckill");
        //seckill01使用seckill的路由绑定到了seckillD01
    }

    @Bean
    public Binding bind02(){
        return BindingBuilder.bind(queue02()).to(exD2()).with("haha");
        //seckill02使用haha的路由绑定到了seckillD02
    }
}
```



## 消费的监听逻辑

SpringBoot整合后，可以使用==注解的形式==，在任意一个`@Component`标识的类中，实现消费的逻辑。可以结合其他对象的注入将消费逻辑实现的更丰富

```java
@Component
public class SeckillConsumer2 {
    //任意编辑一个方法，实现消费逻辑
    //方法的参数就是发送到rabbitmq中的对象
    //可以String 接收body 也可以是Message接收
    
    //包含消息属性
    @RabbitListener(queues = "seckill01")//监听的队列
    public void consume(String msg){
        /*try{}catch(){}*/
        //执行消费逻辑
        System.out.println("消费者2接收到消息："+msg);
    }
}
```





## 生产者

```java
@RestController
public class SenderController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    //接收请求，发送消息
    @RequestMapping("/send")
    public String sendMsg(String msg) {
        rabbitTemplate.convertAndSend("seckillD1", "seckill", msg);
        return "success";
    }
```



## 消费者

### 接口文件

| 后台接收 | /seckill/manage/{seckillId}                                  |
| -------- | ------------------------------------------------------------ |
| 请求方式 | Get                                                          |
| 请求参数 | Long seckillId 路径传参                                      |
| 返回数据 | SysResult的返回对象 Integer status 200表示秒杀成功 String msg:ok表示成功 Object data:其他数据 |



### 代码

```java
@Component
public class SeckillConsumer {

    @Autowired
    private SecMapper secMapper;
    //任意编辑一个方法，实现消费逻辑
    //方法的参数就是发送到rabbitmq中的对象
    //可以String 接收body 也可以是Message接收

    //包含消息属性
    @RabbitListener(queues = "seckill01")
    public void consume(String msg){
        //接收到消息msg="1330119123/1"包含了user信息 商品id
        //解析出来
        Long userPhone=Long.parseLong(msg.split("/")[0]);
        Long seckillId=Long.parseLong(msg.split("/")[1]);

        //先连接redis 从redis中获取一个 seckill_1 rpop元素，成功了
        //说明元素没有被rpop完，具备秒杀减库存的权利
        //手动创建这个list 在redis集群，没创建会报错
        String listKey="seckill_"+seckillId;
        String rpop = jedisCluster.rpop(listKey);
        if(rpop==null){
            //如果rpop结果是null说明元素已经被拿完了，后续减库存
            //都不做了
            System.out.println("用户"+userPhone+"秒杀"+seckillId+"由于redis库存见底，秒杀失败");
            //解决了大量请求到达数据判断number>0时出现
            //线程安全问题导致的超卖
            return;
        }


        //执行减库存逻辑 seckill表格 number=number-1
        /*
        UPDATE seckill SET number=number-1 WHERE seckill_id=2
        AND number>0 AND NOW()>start_time AND NOW()<end_time
         */

        int result=secMapper.decrSeckillNum(seckillId);
        //判断减库存是否成功，result=1成功 result=0失败
        if(result==1){
            //成功 当前用户具备购买商品的资格
            //将成功的信息封装数据入库记录success表格
            Success suc=new Success();
            suc.setCreateTime(new Date());
            suc.setSeckillId(seckillId);
            suc.setUserPhone(userPhone);
            suc.setState(1);
            secMapper.insertSuccess(suc);
        }else{
            //失败，result==0 减库存失败，是因为卖完了
            System.out.println("用户"+userPhone+"秒杀"+seckillId+"由于库存见底，秒杀失败");
        }
    }
}
```



## 展示秒杀成功用户

### 接口文件

| 后台接收 | /seckill/manage/{seckillId}/userPhone      |
| -------- | ------------------------------------------ |
| 请求方式 | Get                                        |
| 请求参数 | Long seckillId 路径传参                    |
| 返回数据 | List<Success>对象,将成功者信息封装返回页面 |



### 代码

```java
@RequestMapping("{seckillId}/userPhone")
public List<Success> querySuccess(@PathVariable Long seckillId){
    return seckillMapper.selectSuccess(seckillId);
}
```





## `seckillMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.seckill.mapper.SeckillMapper">
    <!--查询所有秒杀商品-->
    <select id="selectAll" resultType="Seckill">
        select *
        from seckill;
    </select>

    <!--查询单个秒杀商品-->
    <select id="selectOne" resultType="Seckill">
        select *
        from seckill
        where seckill_id = #{seckillId};
    </select>

    <!--秒杀成功，减库存-->
    <update id="decrSeckillNum">
        update seckill
        set number = number - 1
        where seckill_id = #{seckillId}
        and number &gt; 0
        and now() &gt; start_time
        and now() &lt; end_time;
    </update>

    <!--秒杀成功用户信息-->
    <insert id="insertSuccess">
        insert into success (seckill_id, user_phone, state, create_time)
        values (#{seckillId}, #{userPhone}, #{state}, #{createTime});
    </insert>

    <!--所有秒杀成功的用户-->
    <select id="selectSuccess" resultType="Success">
        select *
        from success
        where seckill_id = #{seckillId};
    </select>

</mapper>
```

# RabbitMQ实操

  * [1\. 常用命令](#1-%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4)
    * [1\.1 vhost操作](#11-vhost%E6%93%8D%E4%BD%9C)
    * [1\.2 用户操作](#12-%E7%94%A8%E6%88%B7%E6%93%8D%E4%BD%9C)
    * [1\.3 给用户赋权限](#13-%E7%BB%99%E7%94%A8%E6%88%B7%E8%B5%8B%E6%9D%83%E9%99%90)
  * [2\. docker下命令](#2-docker%E4%B8%8B%E5%91%BD%E4%BB%A4)
    * [2\.1 docker下vhost操作](#21-docker%E4%B8%8Bvhost%E6%93%8D%E4%BD%9C)
    * [2\.2 docker下用户操作](#22-docker%E4%B8%8B%E7%94%A8%E6%88%B7%E6%93%8D%E4%BD%9C)
    * [2\.3 docker下给用户赋权限](#23-docker%E4%B8%8B%E7%BB%99%E7%94%A8%E6%88%B7%E8%B5%8B%E6%9D%83%E9%99%90)

## 1. 常用命令

### 1.1 vhost操作

````shell
# 添加vhost
rabbitmqctl add_vhost /testhost

# 列出vhost
rabbitmqctl list_vhosts

# 删除vhost
rabbitmqctl delete_vhost /testhost
````

### 1.2 用户操作

````shell
# 添加用户  rabbitmqctl add_user {username} {password}
rabbitmqctl add_user admin admin

# 修改用户密码 rabbitmqctl change_password {username} {newpassword}
rabbitmqctl change_password admin 666

# 验证用户密码
rabbitmqctl authenticate_user admin 666

# 删除用户
rabbitmqctl delete_user admin

# 列出用户
rabbitmqctl list_users

# 给用户设置标签 none management monitoring administrator 多个用,分隔
# rabbitmqctl set_user_tags {username} {tag ...}
rabbitmqctl set_user_tags admin administrator
````

### 1.3 给用户赋权限

````shell
# rabbitmqctl set_permissions [-p host] {user} {conf} {write} {read}
# vhost 授予用户访问权限的vhost名称 默认 /
# user 可以访问指定vhost的用户名
# conf 一个用于匹配用户在那些资源上拥有可配置的正则表达式
# write 一个用于匹配用户在那些资源上拥有可写的正则表达式
# read 一个用于匹配用户在那些资源上拥有可读的正则表达式

# 授予admin用户可访问虚拟主机testhost，并在所有的资源上具备可配置、可写及可读的权限
rabbitmqctl set_permissions -p /testhost admin ".*" ".*" ".*"

# 授予admin用户可访问虚拟主机testhost2，在以queue开头的资源上具备可配置权限、并在所有的资源上可写及可读的权限
rabbitmqctl set_permissions -p /testhost2 admin "^queue.*" ".*" ".*"

# 清除权限
rabbitmqctl clear_permissions -p /testhost admin

# 虚拟主机的权限
rabbitmqctl list_permissions -p /testhost

# 用户权限
rabbitmqctl list_user_permissions admin
````

## 2. docker下命令

### 2.1 docker下vhost操作

```shell
# 添加vhost
docker exec -i rabbitmq sh -c 'rabbitmqctl add_vhost /testhost'

# 列出vhost
docker exec -i rabbitmq sh -c 'rabbitmqctl list_vhosts'

# 删除vhost
docker exec -i rabbitmq sh -c 'rabbitmqctl delete_vhost /testhost'
```

### 2.2 docker下用户操作

```shell
# 添加用户  rabbitmqctl add_user {username} {password}
docker exec -i rabbitmq sh -c 'rabbitmqctl add_user admin admin'

# 修改用户密码 rabbitmqctl change_password {username} {newpassword}
docker exec -i rabbitmq sh -c 'rabbitmqctl change_password admin 666'

# 验证用户密码
docker exec -i rabbitmq sh -c 'rabbitmqctl authenticate_user admin 666'

# 删除用户
docker exec -i rabbitmq sh -c 'rabbitmqctl delete_user admin'

# 列出用户
docker exec -i rabbitmq sh -c 'rabbitmqctl list_users'

# 给用户设置标签 none management monitoring administrator 多个用,分隔
# rabbitmqctl set_user_tags {username} {tag ...}
docker exec -i rabbitmq sh -c 'rabbitmqctl set_user_tags admin administrator'
```

### 2.3 docker下给用户赋权限

```shell
# rabbitmqctl set_permissions [-p host] {user} {conf} {write} {read}
# vhost 授予用户访问权限的vhost名称 默认 /
# user 可以访问指定vhost的用户名
# conf 一个用于匹配用户在那些资源上拥有可配置的正则表达式
# write 一个用于匹配用户在那些资源上拥有可写的正则表达式
# read 一个用于匹配用户在那些资源上拥有可读的正则表达式

# 授予admin用户可访问虚拟主机testhost，并在所有的资源上具备可配置、可写及可读的权限
docker exec -i rabbitmq sh -c 'rabbitmqctl set_permissions -p /testhost admin ".*" ".*" ".*"'

# 授予admin用户可访问虚拟主机testhost2，在以queue开头的资源上具备可配置权限、并在所有的资源上可写及可读的权限
docker exec -i rabbitmq sh -c 'rabbitmqctl set_permissions -p /testhost2 admin "^queue.*" ".*" ".*"'

# 清除权限
docker exec -i rabbitmq sh -c 'rabbitmqctl clear_permissions -p /testhost admin'

# 虚拟主机的权限
docker exec -i rabbitmq sh -c 'rabbitmqctl list_permissions -p /testhost'

# 用户权限
docker exec -i rabbitmq sh -c 'rabbitmqctl list_user_permissions admin'
```

# RabbitMQ学习笔记

  * [1 RabbitMQ 简介](#1-rabbitmq-%E7%AE%80%E4%BB%8B)
  * [2 消息通信](#2-%E6%B6%88%E6%81%AF%E9%80%9A%E4%BF%A1)
    * [2\.1 核心概念](#21-%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5)
    * [2\.2 交换机](#22-%E4%BA%A4%E6%8D%A2%E6%9C%BA)
    * [2\.3 消息确认机制](#23-%E6%B6%88%E6%81%AF%E7%A1%AE%E8%AE%A4%E6%9C%BA%E5%88%B6)
  * [3 运行和管理](#3-%E8%BF%90%E8%A1%8C%E5%92%8C%E7%AE%A1%E7%90%86)

## 1 RabbitMQ 简介

## 2 消息通信

### 2.1 核心概念

- 消费者（Producer）
- 生产者（Consumer）
- 消息代理（Message Broker）
- 队列（Queue）
- 交换机（Exchange）：用来发送消息的 AMQP 实体。
- 绑定（Binding）
- 路由键（Routing Key）
- 虚拟主机（vhost）

### 2.2 交换机

**1. 分类**

- Direct exchange（直连交换机）
- Fanout exchange（扇型交换机）
- Topic exchange（主题交换机）
- Headers exchange（头交换机）

**2. 额外属性**

除交换机类型外，在声明交换机时还可以附带许多其他的属性，其中最重要的几个分别是：

- Name
- Durability （消息代理重启后，交换机是否还存在）
- Auto-delete （当所有与之绑定的消息队列都完成了对此交换机的使用后，删掉它）
- Arguments（依赖代理本身）

交换机可以有两个状态：持久（durable）、暂存（transient）。

> 持久化的交换机会在消息代理（broker）重启后依旧存在，而暂存的交换机则不会（它们需要在代理再次上线后重新被声明）。

### 2.3 消息确认机制

## 3 运行和管理