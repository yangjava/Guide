# 如何设计一个安全的对外接口

## 前言

最近有个项目需要对外提供一个接口，提供公网域名进行访问，而且接口和交易订单有关，所以安全性很重要；这里整理了一下常用的一些安全措施以及具体如何去实现。



## 安全措施

个人觉得安全措施大体来看主要在两个方面，一方面就是如何保证数据在传输过程中的安全性，另一个方面是数据已经到达服务器端，服务器端如何识别数据，如何不被攻击；下面具体看看都有哪些安全措施。



### 1.数据加密

我们知道数据在传输过程中是很容易被抓包的，如果直接传输比如通过http协议，那么用户传输的数据可以被任何人获取；所以必须对数据加密，常见的做法对关键字段加密比如用户密码直接通过md5加密；现在主流的做法是使用https协议，在http和tcp之间添加一层加密层(SSL层)，这一层负责数据的加密和解密；



### 2.数据加签

数据加签就是由发送者产生一段无法伪造的一段数字串，来保证数据在传输过程中不被篡改；你可能会问数据如果已经通过https加密了，还有必要进行加签吗？数据在传输过程中经过加密，理论上就算被抓包，也无法对数据进行篡改；但是我们要知道加密的部分其实只是在外网，现在很多服务在内网中都需要经过很多服务跳转，所以这里的加签可以防止内网中数据被篡改；



### 3.时间戳机制

数据是很容易被抓包的，但是经过如上的加密，加签处理，就算拿到数据也不能看到真实的数据；但是有不法者不关心真实的数据，而是直接拿到抓取的数据包进行恶意请求；这时候可以使用时间戳机制，在每次请求中加入当前的时间，服务器端会拿到当前时间和消息中的时间相减，看看是否在一个固定的时间范围内比如5分钟内；这样恶意请求的数据包是无法更改里面时间的，所以5分钟后就视为非法请求了；



### 4.AppId机制

大部分网站基本都需要用户名和密码才能登录，并不是谁来能使用我的网站，这其实也是一种安全机制；对应的对外提供的接口其实也需要这么一种机制，并不是谁都可以调用，需要使用接口的用户需要在后台开通appid，提供给用户相关的密钥；在调用的接口中需要提供appid+密钥，服务器端会进行相关的验证；



### 5.限流机制

本来就是真实的用户，并且开通了appid，但是出现频繁调用接口的情况；这种情况需要给相关appid限流处理，常用的限流算法有令牌桶和漏桶算法；



### 6.黑名单机制

如果此appid进行过很多非法操作，或者说专门有一个中黑系统，经过分析之后直接将此appid列入黑名单，所有请求直接返回错误码；



### 7.数据合法性校验

这个可以说是每个系统都会有的处理机制，只有在数据是合法的情况下才会进行数据处理；每个系统都有自己的验证规则，当然也可能有一些常规性的规则，比如身份证长度和组成，电话号码长度和组成等等；



## 如何实现

以上大体介绍了一下常用的一些接口安全措施，当然可能还有其他我不知道的方式，希望大家补充，下面看看以上这些方法措施，具体如何实现；



### 1.数据加密

现在主流的加密方式有对称加密和非对称加密；
**对称加密**：对称密钥在加密和解密的过程中使用的密钥是相同的，常见的对称加密算法有DES，AES；优点是计算速度快，缺点是在数据传送前，发送方和接收方必须商定好秘钥，然后使双方都能保存好秘钥，如果一方的秘钥被泄露，那么加密信息也就不安全了；
**非对称加密**：服务端会生成一对密钥，私钥存放在服务器端，公钥可以发布给任何人使用；优点就是比起对称加密更加安全，但是加解密的速度比对称加密慢太多了；广泛使用的是RSA算法；

两种方式各有优缺点，而https的实现方式正好是结合了两种加密方式，整合了双方的优点，在安全和性能方面都比较好；

对称加密和非对称加密代码实现，jdk提供了相关的工具类可以直接使用，此处不过多介绍；
关于https如何配置使用相对来说复杂一些，可以参考本人的之前的文章[HTTPS分析与实战](https://my.oschina.net/OutOfMemory/blog/1620342)



### 2.数据加签

数据签名使用比较多的是md5算法，将需要提交的数据通过某种方式组合和一个字符串，然后通过md5生成一段加密字符串，这段加密字符串就是数据包的签名，可以看一个简单的例子：

```
str：参数1={参数1}&参数2={参数2}&……&参数n={参数n}$key={用户密钥};
MD5.encrypt(str);
```

注意最后的用户密钥，客户端和服务端都有一份，这样会更加安全；



### 3.时间戳机制

解密后的数据，经过签名认证后，我们拿到数据包中的客户端时间戳字段，然后用服务器当前时间去减客户端时间，看结果是否在一个区间内，伪代码如下：

```
long interval=5*60*1000；//超时时间
long clientTime=request.getparameter("clientTime");
long serverTime=System.currentTimeMillis();
if(serverTime-clientTime>interval){
    return new Response("超过处理时长")
}
```



### 4.AppId机制

生成一个唯一的AppId即可，密钥使用字母、数字等特殊字符随机生成即可；生成唯一AppId根据实际情况看是否需要全局唯一；但是不管是否全局唯一最好让生成的Id有如下属性：
**趋势递增**：这样在保存数据库的时候，使用索引性能更好；
**信息安全**：尽量不要连续的，容易发现规律；
关于全局唯一Id生成的方式常见的有类snowflake方式等；



### 5.限流机制

常用的限流算法包括：令牌桶限流，漏桶限流，计数器限流；
**1.令牌桶限流**
令牌桶算法的原理是系统以一定速率向桶中放入令牌，填满了就丢弃令牌；请求来时会先从桶中取出令牌，如果能取到令牌，则可以继续完成请求，否则等待或者拒绝服务；令牌桶允许一定程度突发流量，只要有令牌就可以处理，支持一次拿多个令牌；
**2.漏桶限流**
漏桶算法的原理是按照固定常量速率流出请求，流入请求速率任意，当请求数超过桶的容量时，新的请求等待或者拒绝服务；可以看出漏桶算法可以强制限制数据的传输速度；
**3.计数器限流**
计数器是一种比较简单粗暴的算法，主要用来限制总并发数，比如数据库连接池、线程池、秒杀的并发数；计数器限流只要一定时间内的总请求数超过设定的阀值则进行限流；

具体基于以上算法如何实现，Guava提供了RateLimiter工具类基于基于令牌桶算法：

```
RateLimiter rateLimiter = RateLimiter.create(5);
```

以上代码表示一秒钟只允许处理五个并发请求，以上方式只能用在单应用的请求限流，不能进行全局限流；这个时候就需要分布式限流，可以基于redis+lua来实现；



### 6.黑名单机制

如何为什么中黑我们这边不讨论，我们可以给每个用户设置一个状态比如包括：初始化状态，正常状态，中黑状态，关闭状态等等；或者我们直接通过分布式配置中心，直接保存黑名单列表，每次检查是否在列表中即可；



### 7.数据合法性校验

合法性校验包括：常规性校验以及业务校验；
常规性校验：包括签名校验，必填校验，长度校验，类型校验，格式校验等；
业务校验：根据实际业务而定，比如订单金额不能小于0等；



## 总结

本文大致列举了几种常见的安全措施机制包括：数据加密、数据加签、时间戳机制、AppId机制、限流机制、黑名单机制以及数据合法性校验；当然肯定有其他方式，欢迎补充。

# 一、RESTful风格API的好处

API（Application Programming Interface），顾名思义：是一组编程接口规范，客户端与服务端通过请求响应进行数据通信。REST（Representational State Transfer）决定了接口的形式与规则。**RESTful是基于http方法的API设计风格，而不是一种新的技术.**

1. 看Url就知道要什么资源
2. 看http method就知道针对资源干什么
3. 看http status code就知道结果如何

对接口开发提供了一种可以广泛适用的规范，为前端后端交互减少了接口交流的口舌成本，是**约定大于配置**的体现。通过下面的设计，大家来理解一下这三句话。

>当然也不是所有的接口，都能用REST的形式来表述。在实际工作中，灵活运用，我们用RESTful风格的目的是为大家提供统一标准，避免不必要的沟通成本的浪费，形成一种通用的风格。就好比大家都知道：伸出大拇指表示“你很棒“的意思，绝大部分人都明白，因为你了解了这种风格习惯。但是不排除有些地区伸出大拇指表示其他意思，就不适合使用！

# 二、RESTful API的设计风格

## 2.1、REST 是面向资源的（名词）

REST 通过 URI 暴露资源时，会强调不要在 URI 中出现动词。比如：

| 不符合REST的接口URI      | 符合REST接口URI       | 功能           |
| :----------------------- | :-------------------- | :------------- |
| GET /api/getDogs         | GET /api/dogs/{id}    | 获取一个小狗狗 |
| GET /api/getDogs         | GET /api/dogs         | 获取所有小狗狗 |
| GET /api/addDogs         | POST /api/dogs        | 添加一个小狗狗 |
| GET /api/editDogs/{id}   | PUT /api/dogs/{id}    | 修改一个小狗狗 |
| GET /api/deleteDogs/{id} | DELETE /api/dogs/{id} | 删除一个小狗狗 |

## 2.2、用HTTP方法体现对资源的操作（动词）

- GET ： 获取、读取资源
- POST ： 添加资源
- PUT ： 修改资源
- DELETE ： 删除资源

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416130005.png)

实际上，这四个动词实际上就对应着增删改查四个操作，这就利用了HTTP动词来表示对资源的操作。

## 2.3. HTTP状态码

通过HTTP状态码体现动作的结果,不要自定义

```
200 OK 
400 Bad Request 
500 Internal Server Error
```

在 APP 与 API 的交互当中，其结果逃不出这三种状态：

- 所有事情都按预期正确执行完毕 - 成功
- APP 发生了一些错误 – 客户端错误（如：校验用户输入身份证，结果输入的是军官证，就是客户端输入错误）
- API 发生了一些错误 – 服务器端错误（各种编码bug或服务内部自己导致的异常）

这三种状态与上面的状态码是一一对应的。如果你觉得这三种状态，分类处理结果太宽泛，http-status code还有很多。建议还是要遵循KISS(Keep It Stupid and Simple)原则，上面的三种状态码完全可以覆盖99%以上的场景。这三个状态码大家都记得住，而且非常常用，多了就不一定了。

## 2.4. Get方法和查询参数不应该改变数据

改变数据的事交给POST、PUT、DELETE

## 2.5. 使用复数名词

/dogs 而不是 /dog

## 2.6. 复杂资源关系的表达

GET /cars/711/drivers/ 返回 使用过编号711汽车的所有司机
GET /cars/711/drivers/4 返回 使用过编号711汽车的4号司机

## 2.7. 高级用法:HATEOAS

**HATEOAS**:Hypermedia as the Engine of Application State 超媒体作为应用状态的引擎。
RESTful API最好做到HATEOAS，**即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么**。比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。

```json
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
```

上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API或者可以调用什么API了。

## 2.8. 资源**过滤、排序、选择和分页**的表述

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416131439.png)

## 2.9. 版本化你的API

强制性增加API版本声明，不要发布无版本的API。如：/api/v1/blog

**面向扩展开放，面向修改关闭**：也就是说一个版本的接口开发完成测试上线之后，我们一般不会对接口进行修改，如果有新的需求就开发新的接口进行功能扩展。这样做的目的是：当你的新接口上线后，不会影响使用老接口的用户。如果新接口目的是替换老接口，也不要在v1版本原接口上修改，而是开发v2版本接口，并声明v1接口废弃！