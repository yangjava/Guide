## Pulsar の 保证消息的顺序性、幂等性和可靠性

# 一、背景

前面两篇文章，已经介绍了[关于Pulsar消费者的详细使用](https://blog.csdn.net/Howinfun/article/details/119970626?spm=1001.2014.3001.5501)和[自研的Pulsar组件](https://blog.csdn.net/Howinfun/article/details/120010186?spm=1001.2014.3001.5501)。

接下来，将简单分析如何保证消息的顺序性、幂等性和可靠性；但并不会每个分析都会进行代码实战，进行代码实战的都是比较有意思的点，如消费消息如何保证顺序性和幂等性，而其他的其实都是比较简单的，就不做代码实战了。

# 二、特性分析

## 2.1、顺序性

保证消息是按顺序发送，按顺序消费，一个接着一个。

### 2.1.1、活动图

![在这里插入图片描述](https://img-blog.csdnimg.cn/db46dfcd1af24ac3af160216b3473716.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

### 2.1.2、分析

**producer：**

发送者保证消息的顺序性其实是比较简单的：

1. 利用单队列发送
   - 一个业务对应一个队列
   - 一个队列只能由一个消费者监听消费
2. 利用 Pulsar 的分区Topic
   - producer发送消息时需要指定key属性，Pulsar自动会根据Key值将消息分配到指定的分区中
   - 支持多个消费者消费，多个消费者可以监听同一个分区，但是相同的Key只会分配给同一个消费者

生产者这里就不做什么实战的，都是比较简单的点，没啥好说的。

**consumer：**

消费者保证消息的顺序性有下面两种方式：

1. 当前线程执行
   - 单线程执行保证了消费的顺序性
   - 消费效率低
2. 自定义线程池列表异步并发消费
   - 如果直接使用线程池，那么虽然能提高消费效率，但是并不能保证顺序性
   - 这里我们会自定义线程池列表，列表中的线程池的核心线程数和最大线程数都是1，保证顺序消费
   - Producer发送的消息体中，需指定key，我们会根据key#hashCode定位到对应的线程池，这里参考HashMap的做法。

### 2.1.3、代码实战

消费者保证消息顺序性的第二点的实现还是比较有意思的：如何自定义线程池列表、如何根据消息的key来定位线程池。

**代码如下：**

1. 发送消息：

```java
/**
 * 指定key发送消息
 * @author winfun
 **/
@Slf4j
public class ThirdProducerDemo {

    public static void main(String[] args) throws PulsarClientException {
        PulsarClient client = PulsarClient.builder()
                .serviceUrl("pulsar://127.0.0.1:6650")
                .build();

        ProducerBuilder<String> productBuilder = client.newProducer(Schema.STRING).topic("winfun/study/test-topic3")
                .blockIfQueueFull(Boolean.TRUE).batchingMaxMessages(100).enableBatching(Boolean.TRUE).sendTimeout(3, TimeUnit.SECONDS);

        Producer<String> producer = productBuilder.create();
        for (int i = 0; i < 100; i++) {
            MsgDTO msgDTO = new MsgDTO();
            String content = "hello"+i;
            String key;
            if (content.contains("1")){
                key = "k213e434y1df";
            }else if (content.contains("2")){
                key = "keasdgashgfy2";
            }else {
                key = "other";
            }
            msgDTO.setId(key);
            msgDTO.setContent(content);
            producer.send(JSONUtil.toJsonStr(msgDTO));
        }
        producer.close();
    }
}
```

1. 消费消息

```java
/**
 * 顺序性消费-消费者demo
 * @author: winfun
 **/
@Slf4j
@PulsarListener(topics = {"test-topic3"})
public class SuccessionConsumerListener extends BaseMessageListener {

    List<ExecutorService> executorServiceList = new ArrayList<>();

    /**
     * 初始化自定义线程池列表
     */
    @PostConstruct
    public void initCustomThreadPool(){
        for (int i = 0; i < 10; i++) {
            /**
             * 1、核心线程数和最大线程数都为1，避免多线程消费导致顺序被打乱
             * 2、使用有界队列，设定最大长度，避免无限任务数导致OOM
             * 3、使用CallerRunsPolicy拒绝策略，让当前线程执行，避免消息丢失，也可以直接让消费者执行当前任务，阻塞住其他任务，也能保证顺序性
             */
            ExecutorService threadPoolExecutor = new ThreadPoolExecutor(
                    1,
                    1,
                    60,
                    TimeUnit.MINUTES,
                    new LinkedBlockingDeque<>(100),
                    new ThreadFactoryBuilder().setNameFormat(String.format("custom-thread-pool-%d",i)).get(),
                    new ThreadPoolExecutor.CallerRunsPolicy()
            );
            this.executorServiceList.add(threadPoolExecutor);
        }
    }

    /**
     * 消费消息
     * 自定义监听器实现方法
     * 消息如何响应由开发者决定：
     * Consumer#acknowledge
     * Consumer#reconsumeLater
     * Consumer#negativeAcknowledge
     *
     * @param consumer 消费者
     * @param msg      消息
     */
    @Override
    protected void doReceived(Consumer<String> consumer, Message<String> msg) {
        String value = msg.getValue();
        MsgDTO msgDTO = JSONUtil.toBean(value, MsgDTO.class);
        // 匹配列表中对应的线程池
        int index = (this.executorServiceList.size()-1)&this.spreed(msgDTO.getId().hashCode());
        log.info("成功获取线程池列表索引，msgId is {}, index is {}",msgDTO.getId(),index);
        ExecutorService executorService = this.executorServiceList.get(index);
        executorService.execute(()->{
            log.info("成功消费消息，threadName is {},msg is {}",Thread.currentThread().getName(),msg);
            consumer.acknowledgeAsync(msg);
        });
    }

    /**
     * hashCode扩展，保证hashCode前后十六位都能完美进行位运算
     * @param hashCode
     * @return
     */
    private int spreed(int hashCode){
        return (hashCode ^ (hashCode >>> 16)) & hashCode;
    }
    
    /***
     * 是否开启异步消费，默认开启
     * @return {@link Boolean }
     **/
    @Override
    public Boolean enableAsync() {
        // 首先关闭线程池异步并发消费
        return Boolean.FALSE;
    }
}
```

## 2.2、幂等性

幂等性的话，我们主要是分析一下消费者的，如何保证消费者只正确消费一次消息还是非常重要的。

### 2.2.1、活动图

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-nuZCojk9-1630761143537)(https://note.youdao.com/yws/res/68018/2731974BABE646C2B85AB62ED274729A)]](https://img-blog.csdnimg.cn/90b18a8dff634bdda26bfc019e604996.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

### 2.2.2、分析

**producer：**

生产者如何保证幂等性，感觉这个话题没什么好讨论的，如果发生失败就重新发送，否则就正常发送就好了。

**consumer：**

消费者保证消息幂等性，最主要是利用中间表来保存消费记录：

1. 本地新增表来保存消息消费记录
2. 在消息消费前，先判断MessageId判断是否存在消费记录
3. 如果存在，直接响应
4. 如果不存在，则开启本地事务，接着进行消息消费
5. 当成功消费时提交事务，否则回滚

### 2.2.3、代码实战

如何利用消费记录表和本地事务来完成消息消费的幂等性，看下面代码：

1. 发送消息

```java
**
 *
 * @author winfun
 **/
@Slf4j
public class FourthProducerDemo {

    public static void main(String[] args) throws PulsarClientException {
        PulsarClient client = PulsarClient.builder()
                .serviceUrl("pulsar://127.0.0.1:6650")
                .build();

        ProducerBuilder<String> productBuilder = client.newProducer(Schema.STRING).topic("winfun/study/test-topic4")
                .blockIfQueueFull(Boolean.TRUE).batchingMaxMessages(100).enableBatching(Boolean.TRUE).sendTimeout(3, TimeUnit.SECONDS);

        Producer<String> producer = productBuilder.create();
        for (int i = 0; i < 20; i++) {
            MsgDTO msgDTO = new MsgDTO();
            String content = "hello"+i;
            String key;
            if (content.contains("1")){
                key = "k213e434y1df";
            }else if (content.contains("2")){
                key = "keasdgashgfy2";
            }else {
                key = "other";
            }
            msgDTO.setId(key);
            msgDTO.setContent(content);
            producer.send(JSONUtil.toJsonStr(msgDTO));
        }
        producer.close();
    }
}
```

1. 消费消息

```java
package com.github.howinfun.consumer.idempotent;

import cn.hutool.json.JSONUtil;
import com.github.howinfun.core.entity.MessageConsumeRecord;
import com.github.howinfun.core.service.MessageConsumeRecordService;
import com.github.howinfun.dto.MsgDTO;
import io.github.howinfun.listener.BaseMessageListener;
import io.github.howinfun.listener.PulsarListener;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Objects;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import javax.annotation.PostConstruct;
import jodd.util.concurrent.ThreadFactoryBuilder;
import lombok.extern.slf4j.Slf4j;
import org.apache.pulsar.client.api.Consumer;
import org.apache.pulsar.client.api.Message;
import org.apache.pulsar.client.api.PulsarClientException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;

/**
 * 幂等性消费-消费者demo
 * @author: winfun
 * @date: 2021/9/2 12:49 下午
 **/
@Slf4j
@PulsarListener(topics = {"test-topic4"})
public class IdempotentConsumerListener extends BaseMessageListener {

    List<ExecutorService> executorServiceList = new ArrayList<>();

    @Autowired
    private MessageConsumeRecordService service;

    /**
     * 初始化自定义线程池列表
     */
    @PostConstruct
    public void initCustomThreadPool(){
        for (int i = 0; i < 10; i++) {
            /**
             * 1、核心线程数和最大线程数都为1，避免多线程消费导致顺序被打乱
             * 2、使用有界队列，设定最大长度，避免无限任务数导致OOM
             * 3、使用CallerRunsPolicy拒绝策略，让当前线程执行，避免消息丢失，也可以直接让消费者执行当前任务，阻塞住其他任务，也能保证顺序性
             */
            ExecutorService threadPoolExecutor = new ThreadPoolExecutor(
                    1,
                    1,
                    60,
                    TimeUnit.MINUTES,
                    new LinkedBlockingDeque<>(100),
                    new ThreadFactoryBuilder().setNameFormat(String.format("custom-thread-pool-%d",i)).get(),
                    new ThreadPoolExecutor.CallerRunsPolicy()
            );
            this.executorServiceList.add(threadPoolExecutor);
        }
    }

    /**
     * 消费消息
     * 自定义监听器实现方法
     * 消息如何响应由开发者决定：
     * Consumer#acknowledge
     * Consumer#reconsumeLater
     * Consumer#negativeAcknowledge
     *
     * @param consumer 消费者
     * @param msg      消息
     */
    @Override
    protected void doReceived(Consumer<String> consumer, Message<String> msg) {
        boolean flag = preReceived(msg);
        if (Boolean.FALSE.equals(flag)){
            String value = msg.getValue();
            MsgDTO msgDTO = JSONUtil.toBean(value, MsgDTO.class);
            int index = (this.executorServiceList.size()-1)&this.spreed(msgDTO.getId().hashCode());
            log.info("成功获取线程池列表索引，msgId is {}, index is {}",msgDTO.getId(),index);
            ExecutorService executorService = this.executorServiceList.get(index);
            executorService.execute(()->{
                try {
                    this.doInnerReceived(consumer,msg);
                } catch (PulsarClientException e) {
                    log.error("消息消费失败",e);
                }
            });
        }else {
            log.info("此消息的消费记录已存在，直接响应,messageId is {}", msg.getMessageId().toString());
            try {
                consumer.acknowledge(msg);
            } catch (PulsarClientException e) {
                log.error("消息提交失败",e);
            }
        }
    }

    /**
     * 消费前判断，messageId是否存在对应的消费记录
     * @param msg 消息
     * @return 存在结果
     */
    private boolean preReceived(Message<String> msg){
        MessageConsumeRecord record = this.service.getByMessageId(msg.getMessageId().toString());
        if (Objects.isNull(record)){
            return false;
        }
        return true;
    }

    /**
     * 消息消费
     * @param consumer 消费者
     * @param msg 消息
     */
    @Transactional(rollbackFor = Exception.class)
    public void doInnerReceived(Consumer<String> consumer,Message<String> msg) throws PulsarClientException {
        String messageContent = msg.getValue();
        String messageId = msg.getMessageId().toString();
        log.info("成功消费消息，threadName is {},msg is {}",Thread.currentThread().getName(),messageContent);
        this.service.save(new MessageConsumeRecord()
                                  .setMessageId(messageId)
                                  .setMessageContent(messageContent)
                                  .setCreateTime(new Date()));
        // 模拟重复消费，如果消息内容包含8，则插入数据库，但是不响应
        if (messageContent.contains("8")){
            log.info("消息已被消费入库，但不响应，模拟重复消费，messageId is {},messageContent is {}",messageId,messageContent);
        }else {
            consumer.acknowledge(msg);
        }
    }

    /**
     * hashCode扩展，保证hashCode前后十六位都能完美计算
     * @param hashCode
     * @return
     */
    private int spreed(int hashCode){
        return (hashCode ^ (hashCode >>> 16)) & hashCode;
    }
    
    /***
     * 是否开启异步消费，默认开启
     * @return {@link Boolean }
     **/
    @Override
    public Boolean enableAsync() {
        // 首先关闭线程池异步并发消费
        return Boolean.FALSE;
    }
}
```

## 2.3、可靠性

### 2.3.1、活动图

**生产者：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/37e0c312951e461aa1e18001a3a0f5fd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

**消费者：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/82c9a8fbc7204f36b99f08783a9a7bf9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

关于保证消息的可靠性，我们只分析 Producer 和 Consuemr，Pulsar服务器就不分析了。

### 2.3.2、分析

**producer：**

生产者主要还是利用中间表来保证消息发送的可靠性：

1. 发送消息前，先插入一条发送记录表
2. 接着开启本地事务，开始发送消息
3. 发送完毕，接到broker返回的响应
4. 更新发送记录为已发送
5. 开启定时任务，定时扫描未发送的记录，重新进行发送

**consumer：**

消费者保证消息的可靠性，只需要利用Pulsar提供的重试策略即可：

1. 开启重试策略，指定重试次数、重试队列和死信队列
2. 捕获异常，调用reconsumeLater方法进行重新消费
3. 监控死信队列，即使进行消息消费异常人工处理