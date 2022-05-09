写文章

# 阿里技术专家详解DDD系列 第五讲：聊聊如何避免写流水账代码

[![阿里巴巴淘系技术](https://pic2.zhimg.com/v2-227b5ef96dc4d1b3a45f7233dfb987e1_xs.jpg)](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)

[阿里巴巴淘系技术](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)[](https://www.zhihu.com/question/48510028)

已认证的官方帐号

170 人赞同了该文章

继续填坑啦！！！谢谢大家对DDD系列的期待，**持续更新，欢迎关注此账号查收后续内容。**

第一篇内容地址：

[阿里巴巴淘系技术：阿里技术专家详解 DDD 系列 第一讲- Domain Primitivezhuanlan.zhihu.com![图标](https://pic4.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/340911587)

第二篇内容地址：

[阿里巴巴淘系技术：阿里技术专家详解DDD系列 第二讲 - 应用架构zhuanlan.zhihu.com![图标](https://pic2.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/343388831)

第三篇内容地址：

[阿里巴巴淘系技术：阿里技术专家详解DDD系列 第三讲 - Repository模式zhuanlan.zhihu.com![图标](https://pic1.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/348706530)



第四篇内容地址：

[阿里巴巴淘系技术：阿里技术专家详解DDD系列 第四讲 - 领域层设计规范zhuanlan.zhihu.com![图标](https://zhstatic.zhihu.com/assets/zhihu/editor/zhihu-card-default.svg)](https://zhuanlan.zhihu.com/p/356518017)



向读者们道歉，由于工作太忙，又对文章质量有追求，所以这篇文章产出速度较慢，但可以向大家保证：文章中的内容都经过了反复实践和踩坑。

在过去一年里我们团队做了大量的老系统重构和迁移，其中有大量的代码属于流水账代码，通常能看到是开发在对外的API接口里直接写业务逻辑代码，或者在一个服务里大量的堆接口，导致业务逻辑实际无法收敛，接口复用性比较差。所以这讲主要想系统性的解释一下如何通过DDD的重构，将原有的流水账代码改造为逻辑清晰、职责分明的模块。

## 案例简介

这里举一个简单的常见案例：下单链路。假设我们在做一个checkout接口，需要做各种校验、查询商品信息、调用库存服务扣库存、然后生成订单：

![img](https://pic3.zhimg.com/80/v2-dc1178ffbf8bb38781b2ad306dae8452_1440w.jpg)

一个比较典型的代码如下：

```text
@RestController
@RequestMapping("/")
public class CheckoutController {

    @Resource
    private ItemService itemService;

    @Resource
    private InventoryService inventoryService;

    @Resource
    private OrderRepository orderRepository;

    @PostMapping("checkout")
    public Result<OrderDO> checkout(Long itemId, Integer quantity) {
        // 1) Session管理
        Long userId = SessionUtils.getLoggedInUserId();
        if (userId <= 0) {
            return Result.fail("Not Logged In");
        }
        
        // 2）参数校验
        if (itemId <= 0 || quantity <= 0 || quantity >= 1000) {
            return Result.fail("Invalid Args");
        }

        // 3）外部数据补全
        ItemDO item = itemService.getItem(itemId);
        if (item == null) {
            return Result.fail("Item Not Found");
        }

        // 4）调用外部服务
        boolean withholdSuccess = inventoryService.withhold(itemId, quantity);
        if (!withholdSuccess) {
            return Result.fail("Inventory not enough");
        }
      
        // 5）领域计算
        Long cost = item.getPriceInCents() * quantity;

        // 6）领域对象操作
        OrderDO order = new OrderDO();
        order.setItemId(itemId);
        order.setBuyerId(userId);
        order.setSellerId(item.getSellerId());
        order.setCount(quantity);
        order.setTotalCost(cost);

        // 7）数据持久化
        orderRepository.createOrder(order);

        // 8）返回
        return Result.success(order);
    }
}
```

为什么这种典型的流水账代码在实际应用中会有问题呢？其本质问题是违背了SRP（Single Responsbility Principle）单一职责原则。这段代码里混杂了业务计算、校验逻辑、基础设施、和通信协议等，在未来无论哪一部分的逻辑变更都会直接影响到这段代码，长期当后人不断的在上面叠加新的逻辑时，会造成代码复杂度增加、逻辑分支越来越多，最终造成bug或者没人敢重构的历史包袱。

所以我们才需要用DDD的分层思想去重构一下以上的代码，通过不同的代码分层和规范，拆分出逻辑清晰，职责明确的分层和模块，也便于一些通用能力的沉淀。

主要的几个步骤分为：

1. 分离出独立的Interface接口层，负责处理网络协议相关的逻辑
2. 从真实业务场景中，找出具体用例（Use Cases），然后将具体用例通过专用的Command指令、Query查询、和Event事件对象来承接
3. 分离出独立的Application应用层，负责业务流程的编排，响应Command、Query和Event。每个应用层的方法应该代表整个业务流程中的一个节点
4. 处理一些跨层的横切关注点，如鉴权、异常处理、校验、缓存、日志等



下面会针对每个点做详细的解释。



## Interface接口层

随着REST和MVC架构的普及，经常能看到开发同学直接在Controller中写业务逻辑，如上面的典型案例，但实际上MVC Controller不是唯一的重灾区。以下的几种常见的代码写法通常都可能包含了同样的问题：

- HTTP 框架：如Spring MVC框架，Spring Cloud等
- RPC 框架：如Dubbo、HSF、gRPC等
- 消息队列MQ的“消费者”：比如JMS的 onMessage，RocketMQ的MessageListener等
- Socket通信：Socket通信的receive、WebSocket的onMessage等
- 文件系统：WatcherService等
- 分布式任务调度：SchedulerX等

这些的方法都有一个共同的点就是都有自己的网络协议，而如果我们的业务代码和网络协议混杂在一起，则会直接导致代码跟网络协议绑定，无法被复用。

所以，在DDD的分层架构中，我们单独会抽取出来Interface接口层，作为所有对外的门户，将网络协议和业务逻辑解耦。



**▐ 接口层的组成**

接口层主要由以下几个功能组成：

1. 网络协议的转化：通常这个已经由各种框架给封装掉了，我们需要构建的类要么是被注解的bean，要么是继承了某个接口的bean。
2. 统一鉴权：比如在一些需要AppKey+Secret的场景，需要针对某个租户做鉴权的，包括一些加密串的校验
3. Session管理：一般在面向用户的接口或者有登陆态的，通过Session或者RPC上下文可以拿到当前调用的用户，以便传递给下游服务。
4. 限流配置：对接口做限流避免大流量打到下游服务
5. 前置缓存：针对变更不是很频繁的只读场景，可以前置结果缓存到接口层
6. 异常处理：通常在接口层要避免将异常直接暴露给调用端，所以需要在接口层做统一的异常捕获，转化为调用端可以理解的数据格式
7. 日志：在接口层打调用日志，用来做统计和debug等。一般微服务框架可能都直接包含了这些功能。

当然，如果有一个独立的网关设施/应用，则可以抽离出鉴权、Session、限流、日志等逻辑，但是目前来看API网关也只能解决一部分的功能，即使在有API网关的场景下，应用里独立的接口层还是有必要的。

在interface层，鉴权、Session、限流、缓存、日志等都比较直接，只有一个异常处理的点需要重点说下。

## **▐ 返回值和异常处理规范，Result vs Exception**

注：这部分主要还是面向REST和RPC接口，其他的协议需要根据协议的规范产生返回值。

在我见过的一些代码里，接口的返回值比较多样化，有些直接返回DTO甚至DO，另一些返回Result。

接口层的核心价值是对外，所以如果只是返回DTO或DO会不可避免的面临异常和错误栈泄漏到使用方的情况，包括错误栈被序列化反序列化的消耗。所以，这里提出一个规范：

> 规范：Interface层的HTTP和RPC接口，返回值为Result，捕捉所有异常规范：Application层的所有接口返回值为DTO，不负责处理异常

Application层的具体规范等下再讲，在这里先展示Interface层的逻辑。

举个例子：

```text
@PostMapping("checkout")
public Result<OrderDTO> checkout(Long itemId, Integer quantity) {
    try {
        CheckoutCommand cmd = new CheckoutCommand();
        OrderDTO orderDTO = checkoutService.checkout(cmd);    
        return Result.success(orderDTO);
    } catch (ConstraintViolationException cve) {
        // 捕捉一些特殊异常，比如Validation异常
        return Result.fail(cve.getMessage());
    } catch (Exception e) {
        // 兜底异常捕获
        return Result.fail(e.getMessage());
    }
}
```

当然，每个接口都要写异常处理逻辑会比较烦，所以可以用AOP做个注解

```text
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ResultHandler {

}

@Aspect
@Component
public class ResultAspect {
    @Around("@annotation(ResultHandler)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        Object proceed = null;
        try {
            proceed = joinPoint.proceed();
        } catch (ConstraintViolationException cve) {
            return Result.fail(cve.getMessage());
        } catch (Exception e) {
            return Result.fail(e.getMessage());
        }
        return proceed;
    }
}
```

然后最终代码则简化为：

```text
@PostMapping("checkout")
@ResultHandler
public Result<OrderDTO> checkout(Long itemId, Integer quantity) {
    CheckoutCommand cmd = new CheckoutCommand();
    OrderDTO orderDTO = checkoutService.checkout(cmd);
    return Result.success(orderDTO);
}
```

## **▐ 接口层的接口的数量和业务间的隔离**

在传统REST和RPC的接口规范中，通常一个领域的接口，无论是REST的Resource资源的GET/POST/DELETE，还是RPC的方法，是追求相对固定的，统一的，而且会追求统一个领域的方法放在一个领域的服务或Controller中。

但是我发现在实际做业务的过程中，特别是当支撑的上游业务比较多时，刻意去追求接口的统一通常会导致方法中的参数膨胀，或者导致方法的膨胀。举个例子：假设有一个宠物卡和一个亲子卡的业务公用一个开卡服务，但是宠物需要传入宠物类型，亲子的需要传入宝宝年龄。

```text
// 可以是RPC Provider 或者 Controller
public interface CardService {

    // 1）统一接口，参数膨胀
    Result openCard(int petType, int babyAge);

    // 2）统一泛化接口，参数语意丢失
    Result openCardV2(Map<String, Object> params);

    // 3）不泛化，同一个类里的接口膨胀
    Result openPetCard(int petType);
    Result openBabyCard(int babyAge);
}
```

可以看出来，无论是怎么操作，都有可能导致CardService这个服务未来越来越难以维护，方法越来越多，一个业务的变更有可能会导致整个服务/Controller的变更，最终变得无法维护。我曾经参与过的一个服务，提供了几十个方法，上万行代码，可想而知无论是使用方对接口的理解成本还是对代码的维护成本都是极高的。所以，这里提出另一个规范：

> 规范：一个Interface层的类应该是“小而美”的，应该是面向“一个单一的业务”或“一类同样需求的业务”，需要尽量避免用同一个类承接不同类型业务的需求。

基于上面的这个规范，可以发现宠物卡和亲子卡虽然看起来像是类似的需求，但并非是“同样需求”的，可以预见到在未来的某个时刻，这两个业务的需求和需要提供的接口会越走越远，所以需要将这两个接口类拆分开：

```text
public interface PetCardService {
    Result openPetCard(int petType);
}

public interface BabyCardService {
    Result openBabyCard(int babyAge);
}
```

这个的好处是符合了Single Responsibility Principle单一职责原则，也就是说一个接口类仅仅会因为一个（或一类）业务的变化而变化。一个建议是当一个现有的接口类过度膨胀时，可以考虑对接口类做拆分，拆分原则和SRP一致。

也许会有人问，如果按照这种做法，会不会产生大量的接口类，导致代码逻辑重复？答案是不会，因为在DDD分层架构里，接口类的核心作用仅仅是协议层，每类业务的协议可以是不同的，而真实的业务逻辑会沉淀到应用层。也就是说Interface和Application的关系是多对多的：

![img](https://pic2.zhimg.com/80/v2-2286732143495286016b21bf71d25415_1440w.jpg)

因为业务需求是快速变化的，所以接口层也要跟着快速变化，通过独立的接口层可以避免业务间相互影响，但我们希望相对稳定的是Application层的逻辑。所以我们接下来看一下Application层的一些规范。



## Application层

## **▐ Application层的组成部分**

Application层的几个核心类：

- ApplicationService应用服务：最核心的类，负责业务流程的编排，但本身不负责任何业务逻辑
- DTO Assembler：负责将内部领域模型转化为可对外的DTO
- Command、Query、Event对象：作为ApplicationService的入参
- 返回的DTO：作为ApplicationService的出参

Application层最核心的对象是ApplicationService，它的核心功能是承接“业务流程“。但是在讲ApplicationService的规范之前，必须要先重点的讲几个特殊类型的对象，即Command、Query和Event。

## **▐ Command、Query、Event对象** 

从本质上来看，这几种对象都是Value Object，但是从语义上来看有比较大的差异：

- Command指令：指调用方明确想让系统操作的指令，其预期是对一个系统有影响，也就是写操作。通常来讲指令需要有一个明确的返回值（如同步的操作结果，或异步的指令已经被接受）。
- Query查询：指调用方明确想查询的东西，包括查询参数、过滤、分页等条件，其预期是对一个系统的数据完全不影响的，也就是只读操作。
- Event事件：指一件已经发生过的既有事实，需要系统根据这个事实作出改变或者响应的，通常事件处理都会有一定的写操作。事件处理器不会有返回值。这里需要注意一下的是，Application层的Event概念和Domain层的DomainEvent是类似的概念，但不一定是同一回事，这里的Event更多是外部一种通知机制而已。



简单总结下：

![img](https://pic3.zhimg.com/80/v2-bae799c2336b2555f96193d6358c28ee_1440w.jpg)

- **为什么要用CQE对象？**

通常在很多代码里，能看到接口上有多个参数，比如上文中的案例：

```text
Result<OrderDO> checkout(Long itemId, Integer quantity);
```

如果需要在接口上增加参数，考虑到向前兼容，则需要增加一个方法：

```text
Result<OrderDO> checkout(Long itemId, Integer quantity);
Result<OrderDO> checkout(Long itemId, Integer quantity, Integer channel);
```

或者常见的查询方法，由于条件的不同导致多个方法：

```text
List<OrderDO> queryByItemId(Long itemId);
List<OrderDO> queryBySellerId(Long sellerId);
List<OrderDO> queryBySellerIdWithPage(Long sellerId, int currentPage, int pageSize);
```

可以看出来，传统的接口写法有几个问题：

1. 接口膨胀：一个查询条件一个方法
2. 难以扩展：每新增一个参数都有可能需要调用方升级
3. 难以测试：接口一多，职责随之变得繁杂，业务场景各异，测试用例难以维护



但是另外一个最重要的问题是：这种类型的参数罗列，本身没有任何业务上的”语意“，只是一堆参数而已，无法明确的表达出来意图。



- **CQE的规范**

所以在Application层的接口里，强力建议的一个规范是：

规范：ApplicationService的接口入参只能是一个Command、Query或Event对象，CQE对象需要能代表当前方法的语意。唯一可以的例外是根据单一ID查询的情况，可以省略掉一个Query对象的创建

按照上面的规范，实现案例是：

```text
public interface CheckoutService {
    OrderDTO checkout(@Valid CheckoutCommand cmd);
    List<OrderDTO> query(OrderQuery query);
    OrderDTO getOrder(Long orderId); // 注意单一ID查询可以不用Query
}

@Data
public class CheckoutCommand {
    private Long userId;
    private Long itemId;
    private Integer quantity;
}

@Data
public class OrderQuery {
    private Long sellerId;
    private Long itemId;
    private int currentPage;
    private int pageSize;
}
```



这个规范的好处是：提升了接口的稳定性、降低低级的重复，并且让接口入参更加语意化。



- **CQE vs DTO**

从上面的代码能看出来，ApplicationService的入参是CQE对象，但是出参却是一个DTO，从代码格式上来看都是简单的POJO对象，那么他们之间有什么区别呢？

- CQE：CQE对象是ApplicationService的输入，是有明确的”意图“的，所以这个对象必须保证其”正确性“。
- DTO：DTO对象只是数据容器，只是为了和外部交互，所以本身不包含任何逻辑，只是贫血对象。



但可能最重要的一点：因为CQE是”意图“，所以CQE对象在理论上可以有”无限“个，每个代表不同的意图；但是DTO作为模型数据容器，和模型一一对应，所以是有限的。



- **CQE的校验**

CQE作为ApplicationService的输入，必须保证其正确性，那么这个校验是放在哪里呢？

在最早的代码里，曾经有这样的校验逻辑，当时写在了服务里：

```text
if (itemId <= 0 || quantity <= 0 || quantity >= 1000) {
    return Result.fail("Invalid Args");
}
```

这种代码在日常非常常见，但其最大的问题就是大量的非业务代码混杂在业务代码中，很明显的违背了单一职责原则。但因为当时入参仅仅是简单的int，所以这个逻辑只能出现在服务里。现在当入参改为了CQE之后，我们可以利用java标准JSR303或JSR380的Bean Validation来前置这个校验逻辑。

> 规范：CQE对象的校验应该前置，避免在ApplicationService里做参数的校验。可以通过JSR303/380和Spring Validation来实现

前面的例子可以改造为：

```text
@Validated // Spring的注解
public class CheckoutServiceImpl implements CheckoutService {
    OrderDTO checkout(@Valid CheckoutCommand cmd) { // 这里@Valid是JSR-303/380的注解
        // 如果校验失败会抛异常，在interface层被捕捉
    }
}

@Data
public class CheckoutCommand {

    @NotNull(message = "用户未登陆")
    private Long userId;

    @NotNull
    @Positive(message = "需要是合法的itemId")
    private Long itemId;

    @NotNull
    @Min(value = 1, message = "最少1件")
    @Max(value = 1000, message = "最多不能超过1000件")
    private Integer quantity;
}
```

这种做法的好处是，让ApplicationService更加清爽，同时各种错误信息可以通过Bean Validation的API做各种个性化定制。



- **避免复用CQE**

因为CQE是有“意图”和“语意”的，我们需要尽量避免CQE对象的复用，哪怕所有的参数都一样，只要他们的语意不同，尽量还是要用不同的对象。

- **规范：针对于不同语意的指令，要避免CQE对象的复用**

❌ 反例：一个常见的场景是“Create创建”和“Update更新”，一般来说这两种类型的对象唯一的区别是一个ID，创建没有ID，而更新则有。所以经常能看见有的同学用同一个对象来作为两个方法的入参，唯一区别是ID是否赋值。这个是错误的用法，因为这两个操作的语意完全不一样，他们的校验条件可能也完全不一样，所以不应该复用同一个对象。正确的做法是产出两个对象：

```text
public interface CheckoutService {
    OrderDTO checkout(@Valid CheckoutCommand cmd);
    OrderDTO updateOrder(@Valid UpdateOrderCommand cmd);
}

@Data
public class UpdateOrderCommand {

    @NotNull(message = "用户未登陆")
    private Long userId;

    @NotNull(message = "必须要有OrderID")
    private Long orderId;

    @NotNull
    @Positive(message = "需要是合法的itemId")
    private Long itemId;

    @NotNull
    @Min(value = 1, message = "最少1件")
    @Max(value = 1000, message = "最多不能超过1000件")
    private Integer quantity;

}
```

## **▐** **ApplicationService**

ApplicationService负责了业务流程的编排，是将原有业务流水账代码剥离了校验逻辑、领域计算、持久化等逻辑之后剩余的流程，是“胶水层”代码。

参考一个简易的交易流程：

![img](https://pic1.zhimg.com/80/v2-a5d57772ff94690f48ab9a2dc27b8ed0_1440w.jpg)

在这个案例里可以看出来，交易这个领域一共有5个用例：下单、支付成功、支付失败关单、物流信息更新、关闭订单。这5个用例可以用5个Command/Event对象代替，也就是对应了5个方法。

我见过3种ApplicationService的组织形态：

## **1**

一个ApplicationService类是一个完整的业务流程，其中每个方法负责处理一个Use Case。这种的好处是可以完整的收敛整个业务逻辑，从接口类即可对业务逻辑有一定的掌握，适合相对简单的业务流程。坏处就是对于复杂的业务流程会导致一个类的方法过多，有可能代码量过大。这种类型的具体案例如：

```text
public interface CheckoutService {

    // 下单
    OrderDTO checkout(@Valid CheckoutCommand cmd);

    // 支付成功
    OrderDTO payReceived(@Valid PaymentReceivedEvent event);

    // 支付取消
    OrderDTO payCanceled(@Valid PaymentCanceledEvent event);

    // 发货
    OrderDTO packageSent(@Valid PackageSentEvent event);

    // 收货
    OrderDTO delivered(@Valid DeliveredEvent event);

    // 批量查询
    List<OrderDTO> query(OrderQuery query);

    // 单个查询
    OrderDTO getOrder(Long orderId);
}
```

2

针对于比较复杂的业务流程，可以通过增加独立的CommandHandler、EventHandler来降低一个类中的代码量：

```text
@Component
public class CheckoutCommandHandler implements CommandHandler<CheckoutCommand, OrderDTO> {
    @Override
    public OrderDTO handle(CheckoutCommand cmd) {
        //
    }
}

public class CheckoutServiceImpl implements CheckoutService {
    @Resource
    private CheckoutCommandHandler checkoutCommandHandler;
    
    @Override
    public OrderDTO checkout(@Valid CheckoutCommand cmd) {
        return checkoutCommandHandler.handle(cmd);
    }
}
```

3

比较激进一点，通过CommandBus、EventBus，直接将指令或事件抛给对应的Handler，EventBus比较常见。具体案例代码如下，通过消息队列收到MQ消息后，生成Event，然后由EventBus做路由到对应的Handler：

```text
// Application层
// 在这里框架通常可以根据接口识别到这个负责处理PaymentReceivedEvent
// 也可以通过增加注解识别
@Component
public class PaymentReceivedHandler implements EventHandler<PaymentReceivedEvent> {
    @Override
    public void process(PaymentReceivedEvent event) {
        //
    }
}

// Interface层，这个是RocketMQ的Listener
public class OrderMessageListener implements MessageListenerOrderly {

    @Resource
    private EventBus eventBus;
    
    @Override
    public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
        
        PaymentReceivedEvent event = new PaymentReceivedEvent();
        eventBus.dispatch(event); // 不需要指定消费者
        
        return ConsumeOrderlyStatus.SUCCESS;
    }
}
```



**不建议**

这种做法可以实现Interface层和某个具体的ApplicationService或Handler的完全静态解藕，在运行时动态dispatch，做的比较好的框架如AxonFramework。虽然看起来很便利，但是根据我们自己业务的实践和踩坑发现，当代码中的CQE对象越来越多，handler越来越复杂时，运行时的dispatch缺乏了静态代码间的关联关系，导致代码很难读懂，特别是当你需要trace一个复杂调用链路时，因为dispatch是运行时的，很难摸清楚具体调用到的对象。所以我们虽然曾经有过这种尝试，但现在已经不建议这么做了。



- **Application Service 是业务流程的封装，不处理业务逻辑**

虽然之前曾经无数次重复ApplicationService只负责业务流程串联，不负责业务逻辑，但如何判断一段代码到底是业务流程还是逻辑呢？举个之前的例子，最初的代码重构后：

```text
@Service
@Validated
public class CheckoutServiceImpl implements CheckoutService {

    private final OrderDtoAssembler orderDtoAssembler = OrderDtoAssembler.INSTANCE;
    @Resource
    private ItemService itemService;
    @Resource
    private InventoryService inventoryService;
    @Resource
    private OrderRepository orderRepository;

    @Override
    public OrderDTO checkout(@Valid CheckoutCommand cmd) {
        ItemDO item = itemService.getItem(cmd.getItemId());
        if (item == null) {
            throw new IllegalArgumentException("Item not found");
        }

        boolean withholdSuccess = inventoryService.withhold(cmd.getItemId(), cmd.getQuantity());
        if (!withholdSuccess) {
            throw new IllegalArgumentException("Inventory not enough");
        }

        Order order = new Order();
        order.setBuyerId(cmd.getUserId());
        order.setSellerId(item.getSellerId());
        order.setItemId(item.getItemId());
        order.setItemTitle(item.getTitle());
        order.setItemUnitPrice(item.getPriceInCents());
        order.setCount(cmd.getQuantity());

        Order savedOrder = orderRepository.save(order);

        return orderDtoAssembler.orderToDTO(savedOrder);
    }
}
```

- **判断是否业务流程的几个点：**

**1**

不要有if/else分支逻辑：也就是说代码的Cyclomatic Complexity（循环复杂度）应该尽量等于1

通常有分支逻辑的，都代表一些业务判断，应该将逻辑封装到DomainService或者Entity里。但这不代表完全不能有if逻辑，比如，在这段代码里：

```text
boolean withholdSuccess = inventoryService.withhold(cmd.getItemId(), cmd.getQuantity());
if (!withholdSuccess) {
    throw new IllegalArgumentException("Inventory not enough");
}
```

虽然CC > 1，但是仅仅代表了中断条件，具体的业务逻辑处理并没有受影响。可以把它看作为Precondition。



**2**

不要有任何计算：

在最早的代码里有这个计算：

```text
// 5）领域计算
Long cost = item.getPriceInCents() * quantity;
order.setTotalCost(cost);
```

通过将这个计算逻辑封装到实体里，避免在ApplicationService里做计算

```text
@Data
public class Order {

    private Long itemUnitPrice;
    private Integer count;

    // 把原来一个在ApplicationService的计算迁移到Entity里
    public Long getTotalCost() {
        return itemUnitPrice * count;
    }
}

order.setItemUnitPrice(item.getPriceInCents());
order.setCount(cmd.getQuantity());
```



**3**

一些数据的转化可以交给其他对象来做：

比如DTO Assembler，将对象间转化的逻辑沉淀在单独的类中，降低ApplicationService的复杂度

```text
OrderDTO dto = orderDtoAssembler.orderToDTO(savedOrder);
```

- **常用的ApplicationService“套路”**

我们可以看出来，ApplicationService的代码通常有类似的结构：AppService通常不做任何决策（Precondition除外），仅仅是把所有决策交给DomainService或Entity，把跟外部交互的交给Infrastructure接口，如Repository或防腐层。

一般的“套路”如下：

- 准备数据：包括从外部服务或持久化源取出相对应的Entity、VO以及外部服务返回的DTO。
- 执行操作：包括新对象的创建、赋值，以及调用领域对象的方法对其进行操作。需要注意的是这个时候通常都是纯内存操作，非持久化。
- 持久化：将操作结果持久化，或操作外部系统产生相应的影响，包括发消息等异步操作。



如果涉及到对多个外部系统（包括自身的DB）都有变更的情况，这个时候通常处在“分布式事务”的场景里，无论是用分布式TX、TCC、还是Saga模式，取决于具体场景的设计，在此处暂时略过。

## **▐** **DTO Assembler**

一个经常被忽视的问题是 ApplicationService应该返回 Entity 还是 DTO？这里提出一个规范，在DDD分层架构中：

> ApplicationService应该永远返回DTO而不是Entity



为什么呢？

1. 构建领域边界：ApplicationService的入参是CQE对象，出参是DTO，这些基本上都属于简单的POJO，来确保Application层的内外互相不影响。
2. 降低规则依赖：Entity里面通常会包含业务规则，如果ApplicationService返回Entity，则会导致调用方直接依赖业务规则。如果内部规则变更可能直接影响到外部。
3. 通过DTO组合降低成本：Entity是有限的，DTO可以是多个Entity、VO的自由组合，一次性封装成复杂DTO，或者有选择的抽取部分参数封装成DTO可以降低对外的成本。



因为我们操作的对象是Entity，但是输出的对象是DTO，这里就需要一个专属类型的对象叫DTO Assembler。DTO Assembler的唯一职责是将一个或多个Entity/VO，转化为DTO。注意：DTO Assembler通常不建议有反操作，也就是不会从DTO到Entity，因为通常一个DTO转化为Entity时是无法保证Entity的准确性的。

通常，Entity转DTO是有成本的，无论是代码量还是运行时的操作。手写转换代码容易出错，为了节省代码量用Reflection会造成极大的性能损耗。所以这里我还是不遗余力的推荐MapStruct这个库。MapStruct通过静态编译时代码生成，通过写接口和配置注解就可以生成对应的代码，且因为生成的代码是直接赋值，其性能损耗基本可以忽略不计。

通过MapStruct，代码即可简化为：

```text
import org.mapstruct.Mapper;
@Mapper
public interface OrderDtoAssembler {
    OrderDtoAssembler INSTANCE = Mappers.getMapper(OrderDtoAssembler.class);
    OrderDTO orderToDTO(Order order);
}

public class CheckoutServiceImpl implements CheckoutService {
    private final OrderDtoAssembler orderDtoAssembler = OrderDtoAssembler.INSTANCE;

    @Override
    public OrderDTO checkout(@Valid CheckoutCommand cmd) {
        // ...
        Order order = new Order();  
        // ...
        Order savedOrder = orderRepository.save(order);
        return orderDtoAssembler.orderToDTO(savedOrder);
    }
}
```

结合之前的Data Mapper，DTO、Entity和DataObject之间的关系如下图：

![img](https://pic2.zhimg.com/80/v2-9749db9c247d9bf5ce1afd0c1deec5d5_1440w.jpg)

## **▐** **Result vs Exception** 

最后，上文曾经提及在Interface层应该返回Result，在Application层应该返回DTO，在这里再次重复提出规范：

> Application层只返回DTO，可以直接抛异常，不用统一处理。所有调用到的服务也都可以直接抛异常，除非需要特殊处理，否则不需要刻意捕捉异常

异常的好处是能明确的知道错误的来源，堆栈等，在Interface层统一捕捉异常是为了避免异常堆栈信息泄漏到API之外，但是在Application层，异常机制仍然是信息量最大，代码结构最清晰的方法，避免了Result的一些常见且繁杂的Result.isSuccess判断。所以在Application层、Domain层，以及Infrastructure层，遇到错误直接抛异常是最合理的方法。



## **▐** **Anti-Corruption Layer防腐层** 

本文仅仅简单描述一下ACL的原理和作用，具体的实施规范可能要等到另外一篇文章。

在ApplicationService中，经常会依赖外部服务，从代码层面对外部系统产生了依赖。比如上文中的：

```text
ItemDO item = itemService.getItem(cmd.getItemId());
boolean withholdSuccess = inventoryService.withhold(cmd.getItemId(), cmd.getQuantity());
```

会发现我们的ApplicationService会强依赖ItemService、InventoryService以及ItemDO这个对象。如果任何一个服务的方法变更，或者ItemDO字段变更，都会有可能影响到ApplicationService的代码。也就是说，我们自己的代码会因为强依赖了外部系统的变化而变更，这个在复杂系统中应该是尽量避免的。那么如何做到对外部系统的隔离呢？需要加入ACL防腐层。

ACL防腐层的简单原理如下：

- 对于依赖的外部对象，我们抽取出所需要的字段，生成一个内部所需的VO或DTO类
- 构建一个新的Facade，在Facade中封装调用链路，将外部类转化为内部类
- 针对外部系统调用，同样的用Facade方法封装外部调用链路

无防腐层的情况：

![img](https://pic2.zhimg.com/80/v2-c8c1950272b46a499d196e02333b7339_1440w.jpg)

有防腐层的情况：

![img](https://pic3.zhimg.com/80/v2-251ec60197da62a65d1613a4d20767be_1440w.jpg)

具体简单实现，假设所有外部依赖都命名为ExternalXXXService：

```text
// 自定义的内部值类
@Data
public class ItemDTO {
    private Long itemId;
    private Long sellerId;
    private String title;
    private Long priceInCents;
}

// 商品Facade接口
public interface ItemFacade {
    ItemDTO getItem(Long itemId);
}
// 商品facade实现
@Service
public class ItemFacadeImpl implements ItemFacade {

    @Resource
    private ExternalItemService externalItemService;

    @Override
    public ItemDTO getItem(Long itemId) {
        ItemDO itemDO = externalItemService.getItem(itemId);
        if (itemDO != null) {
            ItemDTO dto = new ItemDTO();
            dto.setItemId(itemDO.getItemId());
            dto.setTitle(itemDO.getTitle());
            dto.setPriceInCents(itemDO.getPriceInCents());
            dto.setSellerId(itemDO.getSellerId());
            return dto;
        }
        return null;
    }
}

// 库存Facade
public interface InventoryFacade {
    boolean withhold(Long itemId, Integer quantity);
}
@Service
public class InventoryFacadeImpl implements InventoryFacade {

    @Resource
    private ExternalInventoryService externalInventoryService;

    @Override
    public boolean withhold(Long itemId, Integer quantity) {
        return externalInventoryService.withhold(itemId, quantity);
    }
}
```

通过ACL改造之后，我们ApplicationService的代码改为：

```text
@Service
public class CheckoutServiceImpl implements CheckoutService {

    @Resource
    private ItemFacade itemFacade;
    @Resource
    private InventoryFacade inventoryFacade;
    
    @Override
    public OrderDTO checkout(@Valid CheckoutCommand cmd) {
        ItemDTO item = itemFacade.getItem(cmd.getItemId());
        if (item == null) {
            throw new IllegalArgumentException("Item not found");
        }

        boolean withholdSuccess = inventoryFacade.withhold(cmd.getItemId(), cmd.getQuantity());
        if (!withholdSuccess) {
            throw new IllegalArgumentException("Inventory not enough");
        }

        // ...
    }
}
```

很显然，这么做的好处是ApplicationService的代码已经完全不再直接依赖外部的类和方法，而是依赖了我们自己内部定义的值类和接口。如果未来外部服务有任何的变更，需要修改的是Facade类和数据转化逻辑，而不需要修改ApplicationService的逻辑。

Repository可以认为是一种特殊的ACL，屏蔽了具体数据操作的细节，即使底层数据库结构变更，数据库类型变更，或者加入其他的持久化方式，Repository的接口保持稳定，ApplicationService就能保持不变。

在一些理论框架里ACL Facade也被叫做Gateway，含义是一样的。



## Orchestration vs Choreography

在本文最后想聊一下复杂业务流程的设计规范。在复杂的业务流程里，我们通常面临两种模式：Orchestration 和 Choreography。很无奈，这两个英文单词的百度翻译/谷歌翻译，都是“编排”，但实际上这两种模式是完全不一样的设计模式。

Orchestration的编排（比如SOA/微服务的服务编排Service Orchestration）是我们通常熟悉的用法，Choreography是最近出现了事件驱动架构EDA才慢慢流行起来。网上可能会有其他的翻译，比如编制、编舞、协作等，但感觉都没有真正的把英文单词的意思表达出来，所以为了避免误解，在下文我尽量还是用英文原词。如果谁有更好的翻译方法欢迎联系我。

## **▐** **模式简介**

Orchestration：通常出现在脑海里的是一个交响乐团（Orchestra，注意这两个词的相似性），如下图。交响乐团的核心是一个唯一的指挥家Conductor，在一个交响乐中，所有的音乐家必须听从Conductor的指挥做操作，不可以独自发挥。所以在Orchestration模式中，所有的流程都是由一个节点或服务触发的。我们常见的业务流程代码，包括调用外部服务，就是Orchestration，由我们的服务统一触发。

![img](https://pic2.zhimg.com/80/v2-79de25873eae7f90c0e2b8c1ff610ec1_1440w.jpg)

Choreography：通常会出现在脑海的场景是一个舞剧（来自于希腊文的舞蹈，Choros），如下图。其中每个不同的舞蹈家都在做自己的事，但是没有一个中心化的指挥。通过协作配合，每个人做好自己的事，整个舞蹈可以展现出一个完整的、和谐的画面。所以在Choreography模式中，每个服务都是独立的个体，可能会响应外部的一些事件，但整个系统是一个整体。

![img](https://pic4.zhimg.com/80/v2-3569e9f56a3baaf031f6ad582a72d79b_1440w.jpg)

## **▐** **案例** 

用一个常见的例子：下单后支付并发货如果这个案例是Orchestration，则业务逻辑为：下单时从一个预存的账户里扣取资金，并且生成物流单发货，从图上看是这样的：

![img](https://pic3.zhimg.com/80/v2-98204f5270b52eddfb1f77161205a612_1440w.jpg)

如果这个案例是Choreography，则业务逻辑为：下单，然后等支付成功事件，然后再发货，类似这样：

![img](https://pic3.zhimg.com/80/v2-802ef5d3893f21c1f572b5e7aee4b616_1440w.jpg)

## **模式的区别和选择** 

虽然看起来这两种模式都能达到一样的业务目的，但是在实际开发中他们有巨大的差异：



- **从代码依赖关系来看：**

Orchestration：涉及到一个服务调用到另外的服务，对于调用方来说，是强依赖的服务提供方。

Choreography：每一个服务只是做好自己的事，然后通过事件触发其他的服务，服务之间没有直接调用上的依赖。但要注意的是下游还是会依赖上游的代码（比如事件类），所以可以认为是下游对上游有依赖。



- **从代码灵活性来看：**

Orchestration：因为服务间的依赖关系是写死的，增加新的业务流程必然需要修改代码。

Choreography：因为服务间没有直接调用关系，可以增加或替换服务，而不需要改上游代码。



- **从调用链路来看：**

Orchestration：是从一个服务主动调用另一个服务，所以是Command-Driven指令驱动的。

Choreography：是每个服务被动的被外部事件触发，所以是Event-Driven事件驱动的。



- **从业务职责来看：**

Orchestration：有主动的调用方（比如：下单服务）。无论下游的依赖是谁，主动的调用方都需要为整个业务流程和结果负责。Choreography：没有主动调用方，每个服务只关心自己的触发条件和结果，没有任何一个服务会为整个业务链路负责



- **小结：**

![img](https://pic2.zhimg.com/80/v2-80cb9847e4617a386e4614c5d0799e95_1440w.jpg)

另外需要重点明确的：“指令驱动”和“事件驱动”的区别不是“同步”和“异步”。指令可以是同步调用，也可以是异步消息触发（但异步指令不是事件）；反过来事件可以是异步消息，但也完全可以是进程内的同步调用。所以指令驱动和事件驱动差异的本质不在于调用方式，而是一件事情是否“已经”发生。

- **所以在日常业务中当你碰到一个需求时，该如何选择是用Orchestration还是Choreography？**

这里给出两个判断方法：

1、明确依赖的方向：

![img](https://pic4.zhimg.com/80/v2-1711eb67ce5bab3c83f8f20738fc3d83_1440w.jpg)

在代码中的依赖是比较明确的：如果你是下游，上游对你无感知，则只能走事件驱动；如果上游必须要对你有感知，则可以走指令驱动。反过来，如果你是上游，需要对下游强依赖，则是指令驱动；如果下游是谁无所谓，则可以走事件驱动。

\2. 找出业务中的“负责人”：

![img](https://pic3.zhimg.com/80/v2-0e97571723f1ba687df03d4b61f63446_1440w.jpg)

第二种方法是根据业务场景找出其中的“负责人”。比如，如果业务需要通知卖家，下单系统的单一职责不应该为消息通知负责，但订单管理系统需要根据订单状态的推进主动触发消息，所以是这个功能的负责人。

在一个复杂业务流程里，通常两个模式都要有，但也很容易设计错误。如果出现依赖关系很奇怪，或者代码里调用链路/负责人梳理不清楚的情况，可以尝试转换一下模式，可能会好很多。



- **哪个模式更好？**

很显然，没有最好的模式，只有最合适自己业务场景的模式。

❌ 反例：最近几年比较流行的Event-Driven Architecture（EDA）事件驱动架构，以及Reactive-Programming响应式编程（比如RxJava），虽然有很多创新，但在一定程度上是“当你有把锤子，所有问题都是钉子”的典型案例。他们对一些基于事件的、流处理的问题有奇效，但如果拿这些框架硬套指令驱动的业务，就会感到代码极其“不协调”，认知成本提高。所以在日常选型中，还是要先根据业务场景梳理出来是哪些流程中的部分是Orchestration，哪些是Choreography，然后再选择相对应的框架。

## **▐** **跟DDD分层架构的关系** 

最后，讲了这么多O vs C，跟DDD有啥关系？很简单：

- O&C其实是Interface层的关注点，Orchestration = 对外的API，而Choreography = 消息或事件。当你决策了O还是C之后，需要在interface层承接这些“驱动力”。
- 无论O&C如何设计，Application层都“无感知”，因为ApplicationService天生就可以处理Command、Query和Event，至于这些对象怎么来，是Interface层的决策。

所以，虽然Orchestration 和 Choreography是两种完全不同的业务设计模式，但最终落到Application层的代码应该是一致的，这也是为什么Application层是“用例”而不是“接口”，是相对稳定的存在。



## 总结

只要是做业务的，一定会需要写业务流程和服务编排，但不代表这种代码一定质量差。通过DDD的分层架构里的Interface层和Application层的合理拆分，代码可以变得优雅、灵活，能更快的响应业务但同时又能更好的沉淀。本文主要介绍了一些代码的设计规范，帮助大家掌握一定的技巧。



**Interface层：**

- 职责：主要负责承接网络协议的转化、Session管理等
- 接口数量：避免所谓的统一API，不必人为限制接口类的数量，每个/每类业务对应一套接口即可，接口参数应该符合业务需求，避免大而全的入参
- 接口出参：统一返回Result
- 异常处理：应该捕捉所有异常，避免异常信息的泄漏。可以通过AOP统一处理，避免代码里有大量重复代码。



**Application层：**

- 入参：具像化Command、Query、Event对象作为ApplicationService的入参，唯一可以的例外是单ID查询的场景。
- CQE的语意化：CQE对象有语意，不同用例之间语意不同，即使参数一样也要避免复用。
- 入参校验：基础校验通过Bean Validation api解决。Spring Validation自带Validation的AOP，也可以自己写AOP。
- 出参：统一返回DTO，而不是Entity或DO。
- DTO转化：用DTO Assembler负责Entity/VO到DTO的转化。
- 异常处理：不统一捕捉异常，可以随意抛异常。



**部分Infra层：**

- 用ACL防腐层将外部依赖转化为内部代码，隔离外部的影响



**业务流程设计模式：**

- 没有最好的模式，取决于业务场景、依赖关系、以及是否有业务“负责人”。避免拿着锤子找钉子。



## **▐** **前瞻预告** 

- CQRS是Application层的一种设计模式，是基于Command和Query分离的一种设计理念，从最简单的对象分离，到目前最复杂的Event-Sourcing。这个topic有很多需要深入的点，也经常可以被用到，特别是结合复杂的Aggregate。后面单独会拉出来讲，标题暂定为《CQRS的7层境界》
- 在当今复杂的微服务开发环境下，依赖外部团队开发的服务是不可避免的，但强耦合带来的成本（无论是变更、代码依赖、甚至Maven Jar包间接依赖）是一个复杂系统长期不可忽视的点。ACL防腐层是一种隔离理念，将外部耦合去除，让内部代码更加纯粹。ACL防腐层可以有很多种，Repository是一种特殊的面相数据持久化的ACL，K8S-sidecar-istio 可以说是一种网络层的ACL，但在Java/Spring里可以有比Istio更高效、更通用的方法，待后文介绍。
- 当你开始用起来DDD时，会发现很多代码模式都非常类似，比如主子订单就是总分模式、类目体系的CPV模式也可以用到一些活动上，ECS模式可以在互动业务上发挥作用等等。后面会尝试总结出一些通用的领域设计模式，他们的设计思路、可以解决的问题类型、以及实践落地的方法。



（本篇内容作者：阿里巴巴淘系技术 殷浩）

————————————————————————————————————————

阿里巴巴集团淘系技术部官方账号。淘系技术部是阿里巴巴新零售技术的王牌军，支撑淘宝、天猫核心电商以及淘宝直播、闲鱼、躺平、阿里汽车、阿里房产等创新业务，服务9亿用户，赋能各行业1000万商家。我们打造了全球领先的线上新零售技术平台，并作为核心技术团队保障了11次双十一购物狂欢节的成功。详情可查看我们官网：[阿里巴巴淘系技术部官方网站](https://link.zhihu.com/?target=http%3A//tech.taobao.org/)

点击下方主页关注我们，你将收获更多来自阿里一线工程师的技术实战技巧&成长经历心得。另，不定期更新最新岗位招聘信息和简历内推通道，欢迎各位以最短路径加入我们。

[阿里巴巴淘系技术www.zhihu.com![图标](https://pic2.zhimg.com/v2-227b5ef96dc4d1b3a45f7233dfb987e1_ipico.jpg)](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)



发布于 04-20

[软件架构](https://www.zhihu.com/topic/19556273)

[系统架构](https://www.zhihu.com/topic/19578413)

[代码](https://www.zhihu.com/topic/19559575)

赞同 170

15 条评论

分享

喜欢收藏申请转载



赞同 170

分享

### 推荐阅读

- # 撇开代码不说，谈谈我对架构的6个冷思考

- 撇开代码不说，谈谈我对架构的6个冷思考 作者：王延炯 普通程序员如何定义架构？架构师又是如何定义架构？这位老司机一直用最简单的方式对架构进行定义：架构是一种用计算机解决问题的综合…

- Larry发表于码力全开工...

- # 4 次版本迭代，我们将项目性能提升了 360 倍！

- 一、项目描述二、第一版：面向过程——2个月三、第二版：面向对象——21天四、第三版：完全解耦（队列+多线程）——3天五、第四版：高度抽象（一键启动）——4小时六、关于继续优化的思考《…

- 芋道源码发表于芋道源码

- ![这里有一份关于系统架构知识的汇总 ，请查收...](https://pic2.zhimg.com/v2-da8bf1004bc83dcd1d2b04cf15921491_250x0.jpg?source=172ae18b)

- # 这里有一份关于系统架构知识的汇总 ，请查收...

- 嘉为科技

- ![基于BPMN2.0的工单系统架构设计（下）](https://pic3.zhimg.com/v2-ed33956154728de308a38e4265e548e1_250x0.jpg?source=172ae18b)

- # 基于BPMN2.0的工单系统架构设计（下）

- 文刀

## 15 条评论

切换为时间排序

写下你的评论...







发布

- [![cainStrayer](https://pic2.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/cainstrayer)[cainStrayer](https://www.zhihu.com/people/cainstrayer)04-27

  看是容易看懂.但是有时候设计就是这样的 你遵循了很多接口设计原则 封装处理了很多东西 其实到头来他也违背了你的初衷也就是让他不simple了。总得来说 也不能说大堆的if不行 大堆的业务和检验和数据库交互在一起很烂 毕竟java只是语言 设计也只是为了弥补不足

  1回复踩 举报

- [![Listen](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/listen-34-97)[Listen](https://www.zhihu.com/people/listen-34-97)04-23

  终于等到了，期待下篇

  1回复踩 举报

- [![raoxx](https://pic4.zhimg.com/v2-9cf66cb7d1fecaca6e0a1578815f36e1_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/raoxx-12)[raoxx](https://www.zhihu.com/people/raoxx-12)05-07

  个人比较喜欢这类的文章![[笑哭]](https://pic4.zhimg.com/v2-ca0015e8ed8462cfce839fba518df585.png)![[笑哭]](https://pic4.zhimg.com/v2-ca0015e8ed8462cfce839fba518df585.png)![[笑哭]](https://pic4.zhimg.com/v2-ca0015e8ed8462cfce839fba518df585.png)

  赞回复踩 举报

- [![账号已销](https://pic4.zhimg.com/v2-d2b4ec37e538fcfb8863d4c5f9ec28a9_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/zhou-yi-26-60)[账号已销](https://www.zhihu.com/people/zhou-yi-26-60)04-24

  Result和Exception感觉还是没有一个最优解，只能说是按照约定来设计……返回Result的方法在每次调用时都要封装拆解才能拿到数据对象，代码非常冗杂，确实只适合在接口层使用……但是在应用层里用Exception又会增大代码运行消耗，而且为了识别异常类型，需要建很多Exception类，并且一些系统间通信的问题比如Redis/DB访问超时之类的，又无法准确捕获，日志里就只有一个系统异常，难以溯源。

  赞回复踩 举报

- ![知乎用户](https://pic3.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户回复[账号已销](https://www.zhihu.com/people/zhou-yi-26-60)04-26

  遇到超时之类的异常，如果用Result返回，还是要手工try catch，但是用Exception，根据堆栈可以快速定位到代码

  赞回复踩 举报

- [![账号已销](https://pic4.zhimg.com/v2-d2b4ec37e538fcfb8863d4c5f9ec28a9_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/zhou-yi-26-60)[账号已销](https://www.zhihu.com/people/zhou-yi-26-60)回复知乎用户04-27

  主要还是老项目里AOP捕获异常不能printStackTrace()或者日志输出e，真的是奇葩规定……

  赞回复踩 举报

- [![诸葛小亮](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/zhu-ge-xiao-liang-42-75)[诸葛小亮](https://www.zhihu.com/people/zhu-ge-xiao-liang-42-75)04-22

  感谢分享收货很大，尤其是ApplicationService的入参和出参中，关于CQE和DTO的讲解，从net转到java，也在寻找DDD在java中落地实践。

  

  文章中提到了mapstruct，推荐一个spring增强库：[https://github.com/ZhaoRd/mapstruct-spring-plus](http://link.zhihu.com/?target=https%3A//github.com/ZhaoRd/mapstruct-spring-plus)

  

  可以减少编写mapper接口

  赞回复踩 举报

- [![绅士的白起](https://pic1.zhimg.com/v2-44167151d25bcab83d20535142e28285_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/yu-bang-qi)[绅士的白起](https://www.zhihu.com/people/yu-bang-qi)04-21

  有收获

  赞回复踩 举报

- [![LuckyLight](https://pic2.zhimg.com/680f7dcda1f216a400a863dab4d73343_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/remy-72)[LuckyLight](https://www.zhihu.com/people/remy-72)04-21

  可以的![[赞同]](https://pic2.zhimg.com/v2-419a1a3ed02b7cfadc20af558aabc897.png)

  赞回复踩 举报

- ![知乎用户](https://pic2.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户04-21

  看完还是很懵逼

  赞回复踩 举报

- [![易先锋](https://pic4.zhimg.com/v2-54b9ebc19aa367045635be2112d851e8_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/yi-xian-feng-19)[易先锋](https://www.zhihu.com/people/yi-xian-feng-19)04-21

  受益匪浅，感谢作者

  赞回复踩 举报

- [![我的系列](https://pic2.zhimg.com/v2-01803f97369cce3314070a748faec26c_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/wo-de-xi-lie)[我的系列](https://www.zhihu.com/people/wo-de-xi-lie)04-20

  受益匪浅，期待后续精彩文章

  赞回复踩 举报

- [![红领巾](https://pic2.zhimg.com/v2-4d91604fbea2e87ac4ea4063c5f9d927_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/okweiteng)[红领巾](https://www.zhihu.com/people/okweiteng)04-20

  解答了很多困惑

  赞回复踩 举报

- [![Allen](https://pic1.zhimg.com/v2-418f866fec56c0e8310ba7ef3055a197_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/liu-wen-bo-57)[Allen](https://www.zhihu.com/people/liu-wen-bo-57)04-20

  学习了

  赞回复踩 举报

- [![CDboy](https://pic2.zhimg.com/v2-5e8b72a162ff37c87503a7767a84785a_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/cdboy-25)[CDboy](https://www.zhihu.com/people/cdboy-25)04-20

  顶![[思考]](https://pic4.zhimg.com/v2-bffb2bf11422c5ef7d8949788114c2ab.png)

  赞回复踩 举报

写文章

# 阿里技术专家详解DDD系列 第二讲 - 应用架构

[![阿里巴巴淘系技术](https://pic1.zhimg.com/v2-227b5ef96dc4d1b3a45f7233dfb987e1_xs.jpg?source=172ae18b)](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)

[阿里巴巴淘系技术](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)[](https://www.zhihu.com/question/48510028)

已认证的官方帐号

401 人赞同了该文章

填坑。谢谢大家对这个系列的期待，持续更新，欢迎关注此账号。

第一篇内容附地址：

[阿里巴巴淘系技术：阿里技术专家详解 DDD 系列 第一讲- Domain Primitivezhuanlan.zhihu.com![图标](https://zhstatic.zhihu.com/assets/zhihu/editor/zhihu-card-default.svg)](https://zhuanlan.zhihu.com/p/340911587)



架构这个词源于英文里的“Architecture“，源头是土木工程里的“建筑”和“结构”，而架构里的”架“同时又包含了”架子“（scaffolding）的含义，意指能快速搭建起来的固定结构。而今天的应用架构，意指软件系统中固定不变的代码结构、设计模式、规范和组件间的通信方式。在应用开发中架构之所以是最重要的第一步，因为一个好的架构能让系统安全、稳定、快速迭代。在一个团队内通过规定一个固定的架构设计，可以让团队内能力参差不齐的同学们都能有一个统一的开发规范，降低沟通成本，提升效率和代码质量。

在做架构设计时，一个好的架构应该需要实现以下几个目标：

- **独立于框架：**架构不应该依赖某个外部的库或框架，不应该被框架的结构所束缚。
- **独立于UI：**前台展示的样式可能会随时发生变化（今天可能是网页、明天可能变成console、后天是独立app），但是底层架构不应该随之而变化。
- **独立于底层数据源：**无论今天你用MySQL、Oracle还是MongoDB、CouchDB，甚至使用文件系统，软件架构不应该因为不同的底层数据储存方式而产生巨大改变。
- **独立于外部依赖：**无论外部依赖如何变更、升级，业务的核心逻辑不应该随之而大幅变化。
- **可测试：**无论外部依赖了什么数据库、硬件、UI或者服务，业务的逻辑应该都能够快速被验证正确性。

这就好像是建筑中的楼宇，一个好的楼宇，无论内部承载了什么人、有什么样的活动、还是外部有什么风雨，一栋楼都应该屹立不倒，而且可以确保它不会倒。但是今天我们在做业务研发时，更多的会去关注一些宏观的架构，比如SOA架构、微服务架构，而忽略了应用内部的架构设计，很容易导致代码逻辑混乱，很难维护，容易产生bug而且很难发现。今天，我希望能够通过案例的分析和重构，来推演出一套高质量的DDD架构。



## 1.案例分析 

我们先看一个简单的案例需求如下：

用户可以通过银行网页转账给另一个账号，支持跨币种转账。

同时因为监管和对账需求，需要记录本次转账活动。

拿到这个需求之后，一个开发可能会经历一些技术选型，最终可能拆解需求如下：



**1、**从MySql数据库中找到转出和转入的账户，选择用 MyBatis 的 mapper 实现 DAO；**2、**从 Yahoo（或其他渠道）提供的汇率服务获取转账的汇率信息（底层是 http 开放接口）；

**3、**计算需要转出的金额，确保账户有足够余额，并且没超出每日转账上限；

**4、**实现转入和转出操作，扣除手续费，保存数据库；

**5、**发送 Kafka 审计消息，以便审计和对账用；



而一个简单的代码实现如下：

```text
public class TransferController {

    private TransferService transferService;

    public Result<Boolean> transfer(String targetAccountNumber, BigDecimal amount, HttpSession session) {
        Long userId = (Long) session.getAttribute("userId");
        return transferService.transfer(userId, targetAccountNumber, amount, "CNY");
    }
}

public class TransferServiceImpl implements TransferService {

    private static final String TOPIC_AUDIT_LOG = "TOPIC_AUDIT_LOG";
    private AccountMapper accountDAO;
    private KafkaTemplate<String, String> kafkaTemplate;
    private YahooForexService yahooForex;

    @Override
    public Result<Boolean> transfer(Long sourceUserId, String targetAccountNumber, BigDecimal targetAmount, String targetCurrency) {
        // 1. 从数据库读取数据，忽略所有校验逻辑如账号是否存在等
        AccountDO sourceAccountDO = accountDAO.selectByUserId(sourceUserId);
        AccountDO targetAccountDO = accountDAO.selectByAccountNumber(targetAccountNumber);

        // 2. 业务参数校验
        if (!targetAccountDO.getCurrency().equals(targetCurrency)) {
            throw new InvalidCurrencyException();
        }

        // 3. 获取外部数据，并且包含一定的业务逻辑
        // exchange rate = 1 source currency = X target currency
        BigDecimal exchangeRate = BigDecimal.ONE;
        if (sourceAccountDO.getCurrency().equals(targetCurrency)) {
            exchangeRate = yahooForex.getExchangeRate(sourceAccountDO.getCurrency(), targetCurrency);
        }
        BigDecimal sourceAmount = targetAmount.divide(exchangeRate, RoundingMode.DOWN);

        // 4. 业务参数校验
        if (sourceAccountDO.getAvailable().compareTo(sourceAmount) < 0) {
            throw new InsufficientFundsException();
        }

        if (sourceAccountDO.getDailyLimit().compareTo(sourceAmount) < 0) {
            throw new DailyLimitExceededException();
        }

        // 5. 计算新值，并且更新字段
        BigDecimal newSource = sourceAccountDO.getAvailable().subtract(sourceAmount);
        BigDecimal newTarget = targetAccountDO.getAvailable().add(targetAmount);
        sourceAccountDO.setAvailable(newSource);
        targetAccountDO.setAvailable(newTarget);

        // 6. 更新到数据库
        accountDAO.update(sourceAccountDO);
        accountDAO.update(targetAccountDO);

        // 7. 发送审计消息
        String message = sourceUserId + "," + targetAccountNumber + "," + targetAmount + "," + targetCurrency;
        kafkaTemplate.send(TOPIC_AUDIT_LOG, message);

        return Result.success(true);
    }

}
```

我们可以看到，一段业务代码里经常包含了参数校验、数据读取存储、业务计算、调用外部服务、发送消息等多种逻辑。在这个案例里虽然是写在了同一个方法里，在真实代码中经常会被拆分成多个子方法，但实际效果是一样的，而在我们日常的工作中，绝大部分代码都或多或少的接近于此类结构。在Martin Fowler的 P of EAA书中，这种很常见的代码样式被叫做Transaction Script（事务脚本）。虽然这种类似于脚本的写法在功能上没有什么问题，但是长久来看，他有以下几个很大的问题：可维护性差、可扩展性差、可测试性差。



## **问题1-可维护性能差**

一个应用最大的成本一般都不是来自于开发阶段，而是应用整个生命周期的总维护成本，所以代码的可维护性代表了最终成本。

可维护性 = 当依赖变化时，有多少代码需要随之改变

参考以上的案例代码，事务脚本类的代码很难维护因为以下几点：

- 数据结构的不稳定性：AccountDO类是一个纯数据结构，映射了数据库中的一个表。这里的问题是数据库的表结构和设计是应用的外部依赖，长远来看都有可能会改变，比如数据库要做Sharding，或者换一个表设计，或者改变字段名。
- 依赖库的升级：AccountMapper依赖MyBatis的实现，如果MyBatis未来升级版本，可能会造成用法的不同（可以参考iBatis升级到基于注解的MyBatis的迁移成本）。同样的，如果未来换一个ORM体系，迁移成本也是巨大的。
- 第三方服务依赖的不确定性：第三方服务，比如Yahoo的汇率服务未来很有可能会有变化：轻则API签名变化，重则服务不可用需要寻找其他可替代的服务。在这些情况下改造和迁移成本都是巨大的。同时，外部依赖的兜底、限流、熔断等方案都需要随之改变。
- 第三方服务API的接口变化：YahooForexService.getExchangeRate返回的结果是小数点还是百分比？入参是（source, target）还是（target, source）？谁能保证未来接口不会改变？如果改变了，核心的金额计算逻辑必须跟着改，否则会造成资损。
- 中间件更换：今天我们用Kafka发消息，明天如果要上阿里云用RocketMQ该怎么办？后天如果消息的序列化方式从String改为Binary该怎么办？如果需要消息分片该怎么改？

我们发现案例里的代码对于任何外部依赖的改变都会有比较大的影响。如果你的应用里有大量的此类代码，你每一天的时间基本上会被各种库升级、依赖服务升级、中间件升级、jar包冲突占满，最终这个应用变成了一个不敢升级、不敢部署、不敢写新功能、并且随时会爆发的炸弹，终有一天会给你带来惊喜。



## **问题2-可拓展性差** 

事务脚本式代码的第二大缺陷是：虽然写单个用例的代码非常高效简单，但是当用例多起来时，其扩展性会变得越来越差。

可扩展性 = 做新需求或改逻辑时，需要新增/修改多少代码

参考以上的代码，如果今天需要增加一个跨行转账的能力，你会发现基本上需要重新开发，基本上没有任何的可复用性：

- 数据来源被固定、数据格式不兼容：原有的AccountDO是从本地获取的，而跨行转账的数据可能需要从一个第三方服务获取，而服务之间数据格式不太可能是兼容的，导致从数据校验、数据读写、到异常处理、金额计算等逻辑都要重写。
- 业务逻辑无法复用：数据格式不兼容的问题会导致核心业务逻辑无法复用。每个用例都是特殊逻辑的后果是最终会造成大量的if-else语句，而这种分支多的逻辑会让分析代码非常困难，容易错过边界情况，造成bug。
- 逻辑和数据存储的相互依赖：当业务逻辑增加变得越来越复杂时，新加入的逻辑很有可能需要对数据库schema或消息格式做变更。而变更了数据格式后会导致原有的其他逻辑需要一起跟着动。在最极端的场景下，一个新功能的增加会导致所有原有功能的重构，成本巨大。

在事务脚本式的架构下，一般做第一个需求都非常的快，但是做第N个需求时需要的时间很有可能是呈指数级上升的，绝大部分时间花费在老功能的重构和兼容上，最终你的创新速度会跌为0，促使老应用被推翻重构。



## **问题3-可测试性能差** 

除了部分工具类、框架类和中间件类的代码有比较高的测试覆盖之外，我们在日常工作中很难看到业务代码有比较好的测试覆盖，而绝大部分的上线前的测试属于人肉的“集成测试”。低测试率导致我们对代码质量很难有把控，容易错过边界条件，异常case只有线上爆发了才被动发现。而低测试覆盖率的主要原因是业务代码的可测试性比较差。

可测试性 = 运行每个测试用例所花费的时间 * 每个需求所需要增加的测试用例数量

参考以上的一段代码，这种代码有极低的可测试性：

- 设施搭建困难：当代码中强依赖了数据库、第三方服务、中间件等外部依赖之后，想要完整跑通一个测试用例需要确保所有依赖都能跑起来，这个在项目早期是及其困难的。在项目后期也会由于各种系统的不稳定性而导致测试无法通过。
- 运行耗时长：大多数的外部依赖调用都是I/O密集型，如跨网络调用、磁盘调用等，而这种I/O调用在测试时需要耗时很久。另一个经常依赖的是笨重的框架如Spring，启动Spring容器通常需要很久。当一个测试用例需要花超过10秒钟才能跑通时，绝大部分开发都不会很频繁的测试。
- 耦合度高：假如一段脚本中有A、B、C三个子步骤，而每个步骤有N个可能的状态，当多个子步骤耦合度高时，为了完整覆盖所有用例，最多需要有N * N * N个测试用例。当耦合的子步骤越多时，需要的测试用例呈指数级增长。

在事务脚本模式下，当测试用例复杂度远大于真实代码复杂度，当运行测试用例的耗时超出人肉测试时，绝大部分人会选择不写完整的测试覆盖，而这种情况通常就是bug很难被早点发现的原因。



## **总结分析**



我们重新来分析一下为什么以上的问题会出现？因为以上的代码违背了至少以下几个软件设计的原则：

- 单一性原则（Single Responsibility Principle）：单一性原则要求一个对象/类应该只有一个变更的原因。但是在这个案例里，代码可能会因为任意一个外部依赖或计算逻辑的改变而改变。
- 依赖反转原则（Dependency Inversion Principle）：依赖反转原则要求在代码中依赖抽象，而不是具体的实现。在这个案例里外部依赖都是具体的实现，比如YahooForexService虽然是一个接口类，但是它对应的是依赖了Yahoo提供的具体服务，所以也算是依赖了实现。同样的KafkaTemplate、MyBatis的DAO实现都属于具体实现。
- 开放封闭原则（Open Closed Principle）：开放封闭原则指开放扩展，但是封闭修改。在这个案例里的金额计算属于可能会被修改的代码，这个时候该逻辑应该需要被包装成为不可修改的计算类，新功能通过计算类的拓展实现。



我们需要对代码重构才能解决这些问题。



## 2.重构方案

在重构之前，我们先画一张流程图，描述当前代码在做的每个步骤：

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='604'></svg>)



这是一个传统的三层分层结构：UI层、业务层、和基础设施层。上层对于下层有直接的依赖关系，导致耦合度过高。在业务层中对于下层的基础设施有强依赖，耦合度高。我们需要对这张图上的每个节点做抽象和整理，来降低对外部依赖的耦合度。



## **2.1 - 抽象数据存储层**



第一步常见的操作是将Data Access层做抽象，降低系统对数据库的直接依赖。具体的方法如下：

- 新建Account实体对象：一个实体（Entity）是拥有ID的域对象，除了拥有数据之外，同时拥有行为。Entity和数据库储存格式无关，在设计中要以该领域的通用严谨语言（Ubiquitous Language）为依据。
- 新建对象储存接口类AccountRepository：Repository只负责Entity对象的存储和读取，而Repository的实现类完成数据库存储的细节。通过加入Repository接口，底层的数据库连接可以通过不同的实现类而替换。



具体的简单代码实现如下：

Account实体类：

```text
@Data
public class Account {
    private AccountId id;
    private AccountNumber accountNumber;
    private UserId userId;
    private Money available;
    private Money dailyLimit;

    public void withdraw(Money money) {
        // 转出
    }

    public void deposit(Money money) {
        // 转入
    }
}
```

和AccountRepository及MyBatis实现类：

```text
public interface AccountRepository {
    Account find(AccountId id);
    Account find(AccountNumber accountNumber);
    Account find(UserId userId);
    Account save(Account account);
}

public class AccountRepositoryImpl implements AccountRepository {

    @Autowired
    private AccountMapper accountDAO;

    @Autowired
    private AccountBuilder accountBuilder;

    @Override
    public Account find(AccountId id) {
        AccountDO accountDO = accountDAO.selectById(id.getValue());
        return accountBuilder.toAccount(accountDO);
    }

    @Override
    public Account find(AccountNumber accountNumber) {
        AccountDO accountDO = accountDAO.selectByAccountNumber(accountNumber.getValue());
        return accountBuilder.toAccount(accountDO);
    }

    @Override
    public Account find(UserId userId) {
        AccountDO accountDO = accountDAO.selectByUserId(userId.getId());
        return accountBuilder.toAccount(accountDO);
    }

    @Override
    public Account save(Account account) {
        AccountDO accountDO = accountBuilder.fromAccount(account);
        if (accountDO.getId() == null) {
            accountDAO.insert(accountDO);
        } else {
            accountDAO.update(accountDO);
        }
        return accountBuilder.toAccount(accountDO);
    }

}
```

Account实体类和AccountDO数据类的对比如下：

- Data Object数据类：AccountDO是单纯的和数据库表的映射关系，每个字段对应数据库表的一个column，这种对象叫Data Object。DO只有数据，没有行为。AccountDO的作用是对数据库做快速映射，避免直接在代码里写SQL。无论你用的是MyBatis还是Hibernate这种ORM，从数据库来的都应该先直接映射到DO上，但是代码里应该完全避免直接操作 DO。
- Entity实体类：Account 是基于领域逻辑的实体类，它的字段和数据库储存不需要有必然的联系。Entity包含数据，同时也应该包含行为。在 Account 里，字段也不仅仅是String等基础类型，而应该尽可能用上一讲的 Domain Primitive 代替，可以避免大量的校验代码。



DAO 和 Repository 类的对比如下：

- DAO对应的是一个特定的数据库类型的操作，相当于SQL的封装。所有操作的对象都是DO类，所有接口都可以根据数据库实现的不同而改变。比如，insert 和 update 属于数据库专属的操作。
- Repository对应的是Entity对象读取储存的抽象，在接口层面做统一，不关注底层实现。比如，通过 save 保存一个Entity对象，但至于具体是 insert 还是 update 并不关心。Repository的具体实现类通过调用DAO来实现各种操作，通过Builder/Factory对象实现AccountDO 到 Account之间的转化



## 2.1.1 Repository和Entity

- 通过Account对象，避免了其他业务逻辑代码和数据库的直接耦合，避免了当数据库字段变化时，大量业务逻辑也跟着变的问题。
- 通过Repository，改变业务代码的思维方式，让业务逻辑不再面向数据库编程，而是面向领域模型编程。
- Account属于一个完整的内存中对象，可以比较容易的做完整的测试覆盖，包含其行为。
- Repository作为一个接口类，可以比较容易的实现Mock或Stub，可以很容易测试。
- AccountRepositoryImpl实现类，由于其职责被单一出来，只需要关注Account到AccountDO的映射关系和Repository方法到DAO方法之间的映射关系，相对于来说更容易测试。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='599'></svg>)



## **2.2 - 抽象第三方服务**



类似对于数据库的抽象，所有第三方服务也需要通过抽象解决第三方服务不可控，入参出参强耦合的问题。在这个例子里我们抽象出 ExchangeRateService 的服务，和一个ExchangeRate的Domain Primitive类：

```text
public interface ExchangeRateService {
    ExchangeRate getExchangeRate(Currency source, Currency target);
}

public class ExchangeRateServiceImpl implements ExchangeRateService {

    @Autowired
    private YahooForexService yahooForexService;

    @Override
    public ExchangeRate getExchangeRate(Currency source, Currency target) {
        if (source.equals(target)) {
            return new ExchangeRate(BigDecimal.ONE, source, target);
        }
        BigDecimal forex = yahooForexService.getExchangeRate(source.getValue(), target.getValue());
        return new ExchangeRate(forex, source, target);
    }
```

### **2.2.1 防腐层（ACL）**

这种常见的设计模式叫做Anti-Corruption Layer（防腐层或ACL）。很多时候我们的系统会去依赖其他的系统，而被依赖的系统可能包含不合理的数据结构、API、协议或技术实现，如果对外部系统强依赖，会导致我们的系统被”腐蚀“。这个时候，通过在系统间加入一个防腐层，能够有效的隔离外部依赖和内部逻辑，无论外部如何变更，内部代码可以尽可能的保持不变。



![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='521'></svg>)



ACL 不仅仅只是多了一层调用，在实际开发中ACL能够提供更多强大的功能：

- 适配器：很多时候外部依赖的数据、接口和协议并不符合内部规范，通过适配器模式，可以将数据转化逻辑封装到ACL内部，降低对业务代码的侵入。在这个案例里，我们通过封装了ExchangeRate和Currency对象，转化了对方的入参和出参，让入参出参更符合我们的标准。
- 缓存：对于频繁调用且数据变更不频繁的外部依赖，通过在ACL里嵌入缓存逻辑，能够有效的降低对于外部依赖的请求压力。同时，很多时候缓存逻辑是写在业务代码里的，通过将缓存逻辑嵌入ACL，能够降低业务代码的复杂度。
- 兜底：如果外部依赖的稳定性较差，一个能够有效提升我们系统稳定性的策略是通过ACL起到兜底的作用，比如当外部依赖出问题后，返回最近一次成功的缓存或业务兜底数据。这种兜底逻辑一般都比较复杂，如果散落在核心业务代码中会很难维护，通过集中在ACL中，更加容易被测试和修改。
- 易于测试：类似于之前的Repository，ACL的接口类能够很容易的实现Mock或Stub，以便于单元测试。
- 功能开关：有些时候我们希望能在某些场景下开放或关闭某个接口的功能，或者让某个接口返回一个特定的值，我们可以在ACL配置功能开关来实现，而不会对真实业务代码造成影响。同时，使用功能开关也能让我们容易的实现Monkey测试，而不需要真正物理性的关闭外部依赖。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='589'></svg>)

## **2.3 - 抽象中间件**

类似于2.2的第三方服务的抽象，对各种中间件的抽象的目的是让业务代码不再依赖中间件的实现逻辑。因为中间件通常需要有通用型，中间件的接口通常是String或Byte[] 类型的，导致序列化/反序列化逻辑通常和业务逻辑混杂在一起，造成胶水代码。通过中间件的ACL抽象，减少重复胶水代码。

在这个案例里，我们通过封装一个抽象的AuditMessageProducer和AuditMessage DP对象，实现对底层kafka实现的隔离：

```text
@Value
@AllArgsConstructor
public class AuditMessage {

    private UserId userId;
    private AccountNumber source;
    private AccountNumber target;
    private Money money;
    private Date date;

    public String serialize() {
        return userId + "," + source + "," + target + "," + money + "," + date;   
    }

    public static AuditMessage deserialize(String value) {
        // todo
        return null;
    }
}

public interface AuditMessageProducer {
    SendResult send(AuditMessage message);
}

public class AuditMessageProducerImpl implements AuditMessageProducer {

    private static final String TOPIC_AUDIT_LOG = "TOPIC_AUDIT_LOG";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Override
    public SendResult send(AuditMessage message) {
        String messageBody = message.serialize();
        kafkaTemplate.send(TOPIC_AUDIT_LOG, messageBody);
        return SendResult.success();
    }
}
```

具体的分析和2.2类似，在此略过。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='589'></svg>)

### **2.4 - 封装业务逻辑**

在这个案例里，有很多业务逻辑是跟外部依赖的代码混合的，包括金额计算、账户余额的校验、转账限制、金额增减等。这种逻辑混淆导致了核心计算逻辑无法被有效的测试和复用。在这里，我们的解法是通过Entity、Domain Primitive和Domain Service封装所有的业务逻辑：



**2.4.1 - 用Domain Primitive封装跟实体无关的无状态计算逻辑**

在这个案例里使用ExchangeRate来封装汇率计算逻辑：

```text
BigDecimal exchangeRate = BigDecimal.ONE;
if (sourceAccountDO.getCurrency().equals(targetCurrency)) {
    exchangeRate = yahooForex.getExchangeRate(sourceAccountDO.getCurrency(), targetCurrency);
}
BigDecimal sourceAmount = targetAmount.divide(exchangeRate, RoundingMode.DOWN);
```

变为：

```text
ExchangeRate exchangeRate = exchangeRateService.getExchangeRate(sourceAccount.getCurrency(), targetMoney.getCurrency());
Money sourceMoney = exchangeRate.exchangeTo(targetMoney);
```

### **2.4.2 - 用Entity封装单对象的有状态的行为，包括业务校验** 

用Account实体类封装所有Account的行为，包括业务校验如下：

```text
@Data
public class Account {

    private AccountId id;
    private AccountNumber accountNumber;
    private UserId userId;
    private Money available;
    private Money dailyLimit;

    public Currency getCurrency() {
        return this.available.getCurrency();
    }

    // 转入
    public void deposit(Money money) {
        if (!this.getCurrency().equals(money.getCurrency())) {
            throw new InvalidCurrencyException();
        }
        this.available = this.available.add(money);
    }

    // 转出
    public void withdraw(Money money) {
        if (this.available.compareTo(money) < 0) {
            throw new InsufficientFundsException();
        }
        if (this.dailyLimit.compareTo(money) < 0) {
            throw new DailyLimitExceededException();
        }
        this.available = this.available.subtract(money);
    }
}
```

原有的业务代码则可以简化为：

```text
sourceAccount.deposit(sourceMoney);
targetAccount.withdraw(targetMoney);
```

### **2.4.3 - 用Domain Service封装多对象逻辑**



在这个案例里，我们发现这两个账号的转出和转入实际上是一体的，也就是说这种行为应该被封装到一个对象中去。特别是考虑到未来这个逻辑可能会产生变化：比如增加一个扣手续费的逻辑。这个时候在原有的TransferService中做并不合适，在任何一个Entity或者Domain Primitive里也不合适，需要有一个新的类去包含跨域对象的行为。这种对象叫做Domain Service。

我们创建一个AccountTransferService的类：

```text
public interface AccountTransferService {
    void transfer(Account sourceAccount, Account targetAccount, Money targetMoney, ExchangeRate exchangeRate);
}

public class AccountTransferServiceImpl implements AccountTransferService {
    private ExchangeRateService exchangeRateService;

    @Override
    public void transfer(Account sourceAccount, Account targetAccount, Money targetMoney, ExchangeRate exchangeRate) {
        Money sourceMoney = exchangeRate.exchangeTo(targetMoney);
        sourceAccount.deposit(sourceMoney);
        targetAccount.withdraw(targetMoney);
    }
}
```

而原始代码则简化为一行：

```text
accountTransferService.transfer(sourceAccount, targetAccount, targetMoney, exchangeRate);
```

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='589'></svg>)



**2.5 - 重构后结果分析**

这个案例重构后的代码如下：

```text
public class TransferServiceImplNew implements TransferService {

    private AccountRepository accountRepository;
    private AuditMessageProducer auditMessageProducer;
    private ExchangeRateService exchangeRateService;
    private AccountTransferService accountTransferService;

    @Override
    public Result<Boolean> transfer(Long sourceUserId, String targetAccountNumber, BigDecimal targetAmount, String targetCurrency) {
        // 参数校验
        Money targetMoney = new Money(targetAmount, new Currency(targetCurrency));

        // 读数据
        Account sourceAccount = accountRepository.find(new UserId(sourceUserId));
        Account targetAccount = accountRepository.find(new AccountNumber(targetAccountNumber));
        ExchangeRate exchangeRate = exchangeRateService.getExchangeRate(sourceAccount.getCurrency(), targetMoney.getCurrency());

        // 业务逻辑
        accountTransferService.transfer(sourceAccount, targetAccount, targetMoney, exchangeRate);

        // 保存数据
        accountRepository.save(sourceAccount);
        accountRepository.save(targetAccount);

        // 发送审计消息
        AuditMessage message = new AuditMessage(sourceAccount, targetAccount, targetMoney);
        auditMessageProducer.send(message);

        return Result.success(true);
    }
}
```

可以看出来，经过重构后的代码有以下几个特征：

- 业务逻辑清晰，数据存储和业务逻辑完全分隔。
- Entity、Domain Primitive、Domain Service都是独立的对象，没有任何外部依赖，但是却包含了所有核心业务逻辑，可以单独完整测试。
- 原有的TransferService不再包括任何计算逻辑，仅仅作为组件编排，所有逻辑均delegate到其他组件。这种仅包含Orchestration（编排）的服务叫做Application Service（应用服务）。



我们可以根据新的结构重新画一张图：

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='589'></svg>)



然后通过重新编排后该图变为：

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='586'></svg>)



我们可以发现，通过对外部依赖的抽象和内部逻辑的封装重构，应用整体的依赖关系变了：

- 最底层不再是数据库，而是Entity、Domain Primitive和Domain Service。这些对象不依赖任何外部服务和框架，而是纯内存中的数据和操作。这些对象我们打包为Domain Layer（领域层）。领域层没有任何外部依赖关系。
- 再其次的是负责组件编排的Application Service，但是这些服务仅仅依赖了一些抽象出来的ACL类和Repository类，而其具体实现类是通过依赖注入注进来的。Application Service、Repository、ACL等我们统称为Application Layer（应用层）。应用层 依赖 领域层，但不依赖具体实现。
- 最后是ACL，Repository等的具体实现，这些实现通常依赖外部具体的技术实现和框架，所以统称为Infrastructure Layer（基础设施层）。Web框架里的对象如Controller之类的通常也属于基础设施层。

如果今天能够重新写这段代码，考虑到最终的依赖关系，我们可能先写Domain层的业务逻辑，然后再写Application层的组件编排，最后才写每个外部依赖的具体实现。这种架构思路和代码组织结构就叫做Domain-Driven Design（领域驱动设计，或DDD）。所以DDD不是一个特殊的架构设计，而是所有Transction Script代码经过合理重构后一定会抵达的终点。



## 3. DDD的六边形架构

在我们传统的代码里，我们一般都很注重每个外部依赖的实现细节和规范，但是今天我们需要敢于抛弃掉原有的理念，重新审视代码结构。在上面重构的代码里，如果抛弃掉所有Repository、ACL、Producer等的具体实现细节，我们会发现每一个对外部的抽象类其实就是输入或输出，类似于计算机系统中的I/O节点。这个观点在CQRS架构中也同样适用，将所有接口分为Command（输入）和Query（输出）两种。除了I/O之外其他的内部逻辑，就是应用业务的核心逻辑。基于这个基础，Alistair Cockburn在2005年提出了Hexagonal Architecture（六边形架构），又被称之为Ports and Adapters（端口和适配器架构）。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='448' height='279'></svg>)

在这张图中：

- I/O的具体实现在模型的最外层
- 每个I/O的适配器在灰色地带
- 每个Hex的边是一个端口
- Hex的中央是应用的核心领域模型

在Hex中，架构的组织关系第一次变成了一个二维的内外关系，而不是传统一维的上下关系。同时在Hex架构中我们第一次发现UI层、DB层、和各种中间件层实际上是没有本质上区别的，都只是数据的输入和输出，而不是在传统架构中的最上层和最下层。

除了2005年的Hex架构，2008年 Jeffery Palermo的Onion Architecture（洋葱架构）和2017年 Robert Martin的Clean Architecture（干净架构），都是极为类似的思想。除了命名不一样、切入点不一样之外，其他的整体架构都是基于一个二维的内外关系。这也说明了基于DDD的架构最终的形态都是类似的。Herberto Graca有一个很全面的图包含了绝大部分现实中的端口类，值得借鉴。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='960' height='540'></svg>)

**3.1 - 代码组织结构**

为了有效的组织代码结构，避免下层代码依赖到上层实现的情况，在Java中我们可以通过POM Module和POM依赖来处理相互的关系。通过Spring/SpringBoot的容器来解决运行时动态注入具体实现的依赖的问题。一个简单的依赖关系图如下：

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1080' height='599'></svg>)

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='492' height='520'></svg>)

**3.1.1 - Types 模块**

Types模块是保存可以对外暴露的Domain Primitives的地方。Domain Primitives因为是无状态的逻辑，可以对外暴露，所以经常被包含在对外的API接口中，需要单独成为模块。Types模块不依赖任何类库，纯 POJO 。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='764' height='604'></svg>)



**3.1.2 - Domain 模块**

Domain 模块是核心业务逻辑的集中地，包含有状态的Entity、领域服务Domain Service、以及各种外部依赖的接口类（如Repository、ACL、中间件等。Domain模块仅依赖Types模块，也是纯 POJO 。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='798' height='1042'></svg>)



**3.1.3 - Application模块**

Application模块主要包含Application Service和一些相关的类。Application模块依赖Domain模块。还是不依赖任何框架，纯POJO。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='698' height='434'></svg>)



**3.1.4 - Infrastructure模块**

Infrastructure模块包含了Persistence、Messaging、External等模块。比如：Persistence模块包含数据库DAO的实现，包含Data Object、ORM Mapper、Entity到DO的转化类等。Persistence模块要依赖具体的ORM类库，比如MyBatis。如果需要用Spring-Mybatis提供的注解方案，则需要依赖Spring。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='656' height='502'></svg>)



**3.1.5 - Web模块**

Web模块包含Controller等相关代码。如果用SpringMVC则需要依赖Spring。

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='540' height='314'></svg>)

**3.1.6 - Start模块**

Start模块是SpringBoot的启动类。



## 3.2 - 测试



- Types，Domain模块都属于无外部依赖的纯POJO，基本上都可以100%的被单元测试覆盖。
- Application模块的代码依赖外部抽象类，需要通过测试框架去Mock所有外部依赖，但仍然可以100%被单元测试。
- Infrastructure的每个模块的代码相对独立，接口数量比较少，相对比较容易写单测。但是由于依赖了外部I/O，速度上不可能很快，但好在模块的变动不会很频繁，属于一劳永逸。
- Web模块有两种测试方法：通过Spring的MockMVC测试，或者通过HttpClient调用接口测试。但是在测试时最好把Controller依赖的服务类都Mock掉。一般来说当你把Controller的逻辑都后置到Application Service中时，Controller的逻辑变得极为简单，很容易100%覆盖。
- Start模块：通常应用的集成测试写在start里。当其他模块的单元测试都能100%覆盖后，集成测试用来验证整体链路的真实性。



## 3.3 - 代码的演进/变化速度

在传统架构中，代码从上到下的变化速度基本上是一致的，改个需求需要从接口、到业务逻辑、到数据库全量变更，而第三方变更可能会导致整个代码的重写。但是在DDD中不同模块的代码的演进速度是不一样的：

- Domain层属于核心业务逻辑，属于经常被修改的地方。比如：原来不需要扣手续费，现在需要了之类的。通过Entity能够解决基于单个对象的逻辑变更，通过Domain Service解决多个对象间的业务逻辑变更。
- Application层属于Use Case（业务用例）。业务用例一般都是描述比较大方向的需求，接口相对稳定，特别是对外的接口一般不会频繁变更。添加业务用例可以通过新增Application Service或者新增接口实现功能的扩展。
- Infrastructure层属于最低频变更的。一般这个层的模块只有在外部依赖变更了之后才会跟着升级，而外部依赖的变更频率一般远低于业务逻辑的变更频率。

所以在DDD架构中，能明显看出越外层的代码越稳定，越内层的代码演进越快，真正体现了领域“驱动”的核心思想。



## 4 总结



DDD不是一个什么特殊的架构，而是任何传统代码经过合理的重构之后最终一定会抵达的终点。DDD的架构能够有效的解决传统架构中的问题：

- 高可维护性：当外部依赖变更时，内部代码只用变更跟外部对接的模块，其他业务逻辑不变。
- 高可扩展性：做新功能时，绝大部分的代码都能复用，仅需要增加核心业务逻辑即可。
- 高可测试性：每个拆分出来的模块都符合单一性原则，绝大部分不依赖框架，可以快速的单元测试，做到100%覆盖。
- 代码结构清晰：通过POM module可以解决模块间的依赖关系， 所有外接模块都可以单独独立成Jar包被复用。当团队形成规范后，可以快速的定位到相关代码。





本系列 持续更新，欢迎关注我们蹲守下一篇内容~



——————————————————————————————————————

阿里巴巴集团淘系技术部官方账号。淘系技术部是阿里巴巴新零售技术的王牌军，支撑淘宝、天猫核心电商以及淘宝直播、闲鱼、躺平、阿里汽车、阿里房产等创新业务，服务9亿用户，赋能各行业1000万商家。我们打造了全球领先的线上新零售技术平台，并作为核心技术团队保障了11次双十一购物狂欢节的成功。

点击下方主页关注我们，你将收获更多来自阿里一线工程师的技术实战技巧&成长经历心得。另，不定期更新最新岗位招聘信息和简历内推通道，欢迎各位以最短路径加入我们。

[阿里巴巴淘系技术www.zhihu.com![图标](https://pic2.zhimg.com/v2-227b5ef96dc4d1b3a45f7233dfb987e1_ipico.jpg)](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)



发布于 01-11

[软件架构](https://www.zhihu.com/topic/19556273)

[系统架构](https://www.zhihu.com/topic/19578413)

[领域驱动设计（DDD）](https://www.zhihu.com/topic/19826540)

赞同 401

57 条评论

分享

喜欢收藏申请转载



赞同 401

分享

### 推荐阅读

- # 阿里技术专家详解DDD系列 第三讲 - Repository模式

- 继续填坑。谢谢大家对这个系列的期待， 持续更新，欢迎关注此账号查收后续内容。第一篇内容地址： 阿里巴巴淘系技术：阿里技术专家详解 DDD 系列 第一讲- Domain Primitive第二篇内容地址：…

- 阿里巴巴淘系技术

- # 领域驱动设计（DDD）：领域接口化设计

- 领域接口化设计把服务对象（service）和资源库对象（repository）设计成接口是最常见的。但是这对接口化的认识还远远不够，我们需要更深入地去分析接口化设计和更全面地应用接口化编程。所…

- 不够具体发表于领域驱动设...

- ![领域驱动设计(DDD) 架构](https://pic3.zhimg.com/v2-365ac2b984f29342f40618e939c92cda_250x0.jpg?source=172ae18b)

- # 领域驱动设计(DDD) 架构

- YoungChen

- ![DDD 实践手册(2. 实现分层架构)](https://pic1.zhimg.com/v2-c5db8dc11f281f21bd20eafeb9fd2e0b_250x0.jpg?source=172ae18b)

- # DDD 实践手册(2. 实现分层架构)

- Joshu...发表于领域驱动设...

## 57 条评论

切换为时间排序

写下你的评论...







发布

- [![王岩令](https://pic2.zhimg.com/v2-ee41ddb2e0de957be4df80435a930000_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/fu-yan-39)[王岩令](https://www.zhihu.com/people/fu-yan-39)01-21

  我不懂技术，但是作为消费者，你们的APP没有一个好用的。

  ![img](https://pic4.zhimg.com/v2-c8b55e81c076a66520ad2e0af98f938f.webp)

  16回复踩 举报

- [![太空人](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/spanceman)[太空人](https://www.zhihu.com/people/spanceman)02-08

  DDD包含战略设计和战术设计，目前的讨论都还只是停留在战术设计部分，但DDD的战略设计是更重要的，IT同学在学习DDD时经常会出现重战术而轻战略的误区，需要注意。

  3回复踩 举报

- [![太空人](https://pic2.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/spanceman)[太空人](https://www.zhihu.com/people/spanceman)02-08

  推荐大家去读一下《中台架构与实现-基于DDD和微服务》，对于复杂业务系统的构建，很有帮助。

  3回复踩 举报

- [![不善言辞](https://pic1.zhimg.com/v2-c23d76fc0e5b40023dc4f36fffd31ae1_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/bu-shan-yan-ci-6-94)[不善言辞](https://www.zhihu.com/people/bu-shan-yan-ci-6-94)02-04

  受益匪浅，公司还是用的分层架构，可以推荐一下DDD的书籍吗

  3回复踩 举报

- [![小包](https://pic1.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/xiao-bao-60-14)[小包](https://www.zhihu.com/people/xiao-bao-60-14)01-12

  刚刚看完，理论结合实践把整个模型说的比较清晰。希望能把案例项目开源出来，期待

  3回复踩 举报

- [![磊磊](https://pic2.zhimg.com/v2-aaa790171f3eee42af08509b17e028eb_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/javmain)[磊磊](https://www.zhihu.com/people/javmain)02-01

  源码开源一下，谢谢啦

  2回复踩 举报

- [![离得](https://pic4.zhimg.com/v2-2db13807a82311b3eb78b31b665a8829_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/chi-de-36-97)[离得](https://www.zhihu.com/people/chi-de-36-97)01-22

  第三讲求更

  2回复踩 举报

- [![Yang Thomas](https://pic1.zhimg.com/v2-ab8e501d63f76c4866409ea1871751bd_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/yang-thomas-81)[Yang Thomas](https://www.zhihu.com/people/yang-thomas-81)01-19

  写的真是太好了

  2回复踩 举报

- ![知乎用户](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户01-12

  第三讲什么时候开始

  2回复踩 举报

- [![太空人](https://pic2.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/spanceman)[太空人](https://www.zhihu.com/people/spanceman)02-08

  挺好的！但是，其中把Repository归到应用层，不太合适。Repository只是领域业务的数据持久细节，应该被隐藏在领域服务里面，不应该被暴露出来。而应用服务的职责是组合、编排多个领域服务来完成复杂的业务处理，应用服务不应该关注领域服务的细节。

  1回复踩 举报

- [![阳光沙滩裤](https://pic4.zhimg.com/v2-73bc507c1946b6632b9e4f2b76d2944a_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/yang-guang-sha-tan-ku)[阳光沙滩裤](https://www.zhihu.com/people/yang-guang-sha-tan-ku)02-03

  我们部门的新的研发框架就是基于张建飞的COLA 简洁架构在做的，采用COLA的架构思想，使用gateway作为数据对象（Data Object）和领域对象（Entity）之间的转义网关，其中，gateway除了转义的作用，还起到了防腐解耦的作用，解除了业务代码对底层数据（DO、DTO等）的直接依赖，从而提升系统的可维护性

  1回复踩 举报

- [![刘欢](https://pic1.zhimg.com/044f01a5455a19179c3babe8e81bb3f3_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/liu-huan-47-10)[刘欢](https://www.zhihu.com/people/liu-huan-47-10)02-02

  if (sourceAccountDO.getCurrency().equals(targetCurrency)) {
  exchangeRate = yahooForex.getExchangeRate(sourceAccountDO.getCurrency(), targetCurrency);
  }

  这里是不是写错啦，看了半天没看懂

  1回复踩 举报

- [![比特曼](https://pic1.zhimg.com/f3a1f6c2d_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/iittttt)[比特曼](https://www.zhihu.com/people/iittttt)回复[刘欢](https://www.zhihu.com/people/liu-huan-47-10)02-03

  应该少了个感叹号

  赞回复踩 举报

- [![刘欢](https://pic4.zhimg.com/044f01a5455a19179c3babe8e81bb3f3_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/liu-huan-47-10)[刘欢](https://www.zhihu.com/people/liu-huan-47-10)回复[比特曼](https://www.zhihu.com/people/iittttt)02-03

  是的

  赞回复踩 举报

- ![知乎用户](https://pic2.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户04-26

  受益匪浅，有demo代码连接不？

  赞回复踩 举报

- [![Jeff Yu](https://pic1.zhimg.com/v2-24b5fb9e2892912186bcf8b6740378a3_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/jeff-yu-71)[Jeff Yu](https://www.zhihu.com/people/jeff-yu-71)03-24

  “再其次的是负责组件编排的Application Service，但是这些服务仅仅依赖了一些抽象出来的ACL类和Repository类，而其具体实现类是通过依赖注入注进来的。Application Service、Repository、ACL等我们统称为Application Layer（应用层）。应用层 依赖 领域层，但不依赖具体实现”

  这里有一个疑问，从代码的实现角度上来说，应用层会建一个application的module，那么Application Service、Repository、ACL这些接口是不是就定义到这个module下面了，但是这又和代码中的把repository的接口定义在domain中是不一致的。

  特别想知道是我理解那个地方是有偏差了？

  赞回复踩 举报

- [![hujianbest](https://pic2.zhimg.com/v2-4a19b0f743d47ebe1a8b3cf483380212_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/hujianbest)[hujianbest](https://www.zhihu.com/people/hujianbest)03-14

  有个疑问请教一下，外层的代码越稳定，越内层的代码演进越快；但是外层依赖内层，这样内层在变化的时候，外层不会受影响需要跟着变吗？记得有人说过要向稳定的方向依赖

  赞回复踩 举报

- [![流年忘返](https://pic1.zhimg.com/705541815360349e042fbb3dc5d534fe_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/fei-xiao-long)[流年忘返](https://www.zhihu.com/people/fei-xiao-long)03-13

  完全赞同

  ![img](https://pic3.zhimg.com/v2-cfc14c7293afd962ecbd1dc31dafd002.gif)

  赞回复踩 举报

- ![知乎用户](https://pic1.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户02-25

  讲的非常好, 有很多启发, 期待自己能在新项目里用起来, 阿里还是厉害!!

  赞回复踩 举报

- [![紫色loli](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/zi-se-loli)[紫色loli](https://www.zhihu.com/people/zi-se-loli)02-23

  一个参数使用一个DP，这是不是太多了点？

  赞回复踩 举报

123下一页


首发于[code](https://www.zhihu.com/column/c_1355278956606742528)

写文章

# 阿里技术专家详解 DDD 系列 第一讲- Domain Primitive

[![阿里巴巴淘系技术](https://pic4.zhimg.com/v2-227b5ef96dc4d1b3a45f7233dfb987e1_xs.jpg?source=172ae18b)](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)

[阿里巴巴淘系技术](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)[](https://www.zhihu.com/question/48510028)

已认证的官方帐号

723 人赞同了该文章

## 导读

对于一个架构师来说，在软件开发中如何降低系统复杂度是一个永恒的挑战，无论是 94 年 GoF 的 Design Patterns ， 99 年的 Martin Fowler 的 Refactoring ， 02 年的 P of EAA ，还是 03 年的 Enterprise Integration Patterns ，都是通过一系列的设计模式或范例来降低一些常见的复杂度。

但是问题在于，这些书的理念是通过技术手段解决技术问题，但并没有从根本上解决业务的问题。所以 03 年 Eric Evans 的 Domain Driven Design 一书，以及后续 Vaughn Vernon 的 Implementing DDD ， Uncle Bob 的 Clean Architecture 等书，真正的从业务的角度出发，为全世界绝大部分做纯业务的开发提供了一整套的架构思路。

**（本系列持续更新，感兴趣请关注我们知乎号，不错过下一波干货）**

## 前言

由于 DDD 不是一套框架，而是一种架构思想，所以在代码层面缺乏了足够的约束，导致 DDD 在实际应用中上手门槛很高，甚至可以说绝大部分人都对 DDD 的理解有所偏差。举个例子， Martin Fowler 在他个人博客里描述的一个 Anti-pattern，Anemic Domain Model ①（贫血域模型）在实际应用当中层出不穷，而一些仍然火热的 ORM 工具比如 Hibernate，Entity Framework 实际上助长了贫血模型的扩散。同样的，传统的基于数据库技术以及 MVC 的四层应用架构（UI、Business、Data Access、Database），在一定程度上和 DDD 的一些概念混淆，导致绝大部分人在实际应用当中仅仅用到了 DDD 的建模的思想，而其对于整个架构体系的思想无法落地。

我第一次接触 DDD 应该是 2012 年，当时除了大型互联网公司，基本上商业应用都还处于单机的时代，服务化的架构还局限于单机 +LB 用 MVC 提供 Rest 接口供外部调用，或者用 SOAP 或 WebServices 做 RPC 调用，但其实更多局限于对外部依赖的协议。让我关注到 DDD 思想的是一个叫 Anti-Corruption Layer（防腐层）的概念，特别是其在解决外部依赖频繁变更的情况下，如何将核心业务逻辑和外部依赖隔离的机制。到了 2014 年， SOA 开始大行其道，微服务的概念开始冒头，而如何将一个 Monolith 应用合理的拆分为多个微服务成为了各大论坛的热门话题，而 DDD 里面的 Bounded Context（限界上下文）的思想为微服务拆分提供了一套合理的框架。而在今天，在一个所有的东西都能被称之为“服务”的时代（XAAS）， DDD 的思想让我们能冷静下来，去思考到底哪些东西可以被服务化拆分，哪些逻辑需要聚合，才能带来最小的维护成本，而不是简单的去追求开发效率。

所以今天，我开始这个关于 DDD 的一系列文章，希望能继续在总结前人的基础上发扬光大 DDD 的思想，但是通过一套我认为合理的代码结构、框架和约束，来降低 DDD 的实践门槛，提升代码质量、可测试性、安全性、健壮性。

未来会覆盖的内容包括：

- 最佳架构实践：六边形应用架构 / Clean 架构的核心思想和落地方案
- 持续发现和交付：Event Storming > Context Map > Design Heuristics > Modelling
- 降低架构腐败速度：通过 Anti-Corruption Layer 集成第三方库的模块化方案
- 标准组件的规范和边界：Entity, Aggregate, Repository, Domain Service, Application Service, Event, DTO Assembler 等
- 基于 Use Case 重定义应用服务的边界
- 基于 DDD 的微服务化改造及颗粒度控制
- CQRS 架构的改造和挑战
- 基于事件驱动的架构的挑战
- 等等

今天先给大家带来一篇最基础，但极其有价值的Domain Primitive的概念。



## Domain Primitive

就好像在学任何语言时首先需要了解的是基础数据类型一样，在全面了解 DDD 之前，首先给大家介绍一个最基础的概念: Domain Primitive（DP）。

Primitive 的定义是：

> 不从任何其他事物发展而来
> 初级的形成或生长的早期阶段

就好像 Integer、String 是所有编程语言的Primitive一样，在 DDD 里， DP 可以说是一切模型、方法、架构的基础，而就像 Integer、String 一样， DP 又是无所不在的。所以，第一讲会对 DP 做一个全面的介绍和分析，但我们先不去讲概念，而是从案例入手，看看为什么 DP 是一个强大的概念。



## 案例分析

我们先看一个简单的例子，这个 case 的业务逻辑如下：

> 一个新应用在全国通过 地推业务员 做推广，需要做一个用户注册系统，同时希望在用户注册后能够通过用户电话（先假设仅限座机）的地域（区号）对业务员发奖金。

先不要去纠结这个根据用户电话去发奖金的业务逻辑是否合理，也先不要去管用户是否应该在注册时和业务员做绑定，这里我们看的主要还是如何更加合理的去实现这个逻辑。一个简单的用户和用户注册的代码实现如下：

```text
public class User {
    Long userId;
    String name;
    String phone;
    String address;
    Long repId;
}

public class RegistrationServiceImpl implements RegistrationService {

    private SalesRepRepository salesRepRepo;
    private UserRepository userRepo;

    public User register(String name, String phone, String address) 
      throws ValidationException {
        // 校验逻辑
        if (name == null || name.length() == 0) {
            throw new ValidationException("name");
        }
        if (phone == null || !isValidPhoneNumber(phone)) {
            throw new ValidationException("phone");
        }
        // 此处省略address的校验逻辑

        // 取电话号里的区号，然后通过区号找到区域内的SalesRep
        String areaCode = null;
        String[] areas = new String[]{"0571", "021", "010"};
        for (int i = 0; i < phone.length(); i++) {
            String prefix = phone.substring(0, i);
            if (Arrays.asList(areas).contains(prefix)) {
                areaCode = prefix;
                break;
            }
        }
        SalesRep rep = salesRepRepo.findRep(areaCode);

        // 最后创建用户，落盘，然后返回
        User user = new User();
        user.name = name;
        user.phone = phone;
        user.address = address;
        if (rep != null) {
            user.repId = rep.repId;
        }

        return userRepo.save(user);
    }

    private boolean isValidPhoneNumber(String phone) {
        String pattern = "^0[1-9]{2,3}-?\\d{8}$";
        return phone.matches(pattern);
    }
}
```

我们日常绝大部分代码和模型其实都跟这个是类似的，乍一看貌似没啥问题，但我们再深入一步，从以下四个维度去分析一下：接口的清晰度（可阅读性）、数据验证和错误处理、业务逻辑代码的清晰度、和可测试性。

**▍问题1 - 接口的清晰度**

在Java代码中，对于一个方法来说所有的参数名在编译时丢失，留下的仅仅是一个参数类型的列表，所以我们重新看一下以上的接口定义，其实在运行时仅仅是：

```text
User register(String, String, String);
```

所以以下的代码是一段编译器完全不会报错的，很难通过看代码就能发现的 bug ：

```text
service.register("殷浩", "浙江省杭州市余杭区文三西路969号", "0571-12345678");
```

当然，在真实代码中运行时会报错，但这种 bug 是在运行时被发现的，而不是在编译时。普通的 Code Review 也很难发现这种问题，很有可能是代码上线后才会被暴露出来。这里的思考是，有没有办法在编码时就避免这种可能会出现的问题？

另外一种常见的，特别是在查询服务中容易出现的例子如下：

```text
User findByName(String name);
User findByPhone(String phone);
User findByNameAndPhone(String name, String phone);
```

在这个场景下，由于入参都是 String 类型，不得不在方法名上面加上 `ByXXX `来区分，而 `findByNameAndPhone `同样也会陷入前面的入参顺序错误的问题，而且和前面的入参不同，这里参数顺序如果输错了，方法不会报错只会返回 `null`，而这种 bug 更加难被发现。这里的思考是，有没有办法让方法入参一目了然，避免入参错误导致的 bug ？

### ▍问题2 - 数据验证和错误处理

在前面这段数据校验代码：

```text
if (phone == null || !isValidPhoneNumber(phone)) {
    throw new ValidationException("phone");
}
```

在日常编码中经常会出现，一般来说这种代码需要出现在方法的最前端，确保能够 fail-fast 。但是假设你有多个类似的接口和类似的入参，在每个方法里这段逻辑会被重复。而更严重的是如果未来我们要拓展电话号去包含手机时，很可能需要加入以下代码：

```text
if (phone == null || !isValidPhoneNumber(phone) || !isValidCellNumber(phone)) {
    throw new ValidationException("phone");
}
```

如果你有很多个地方用到了 phone 这个入参，但是有个地方忘记修改了，会造成 bug 。这是一个 DRY 原则被违背时经常会发生的问题。

如果有个新的需求，需要把入参错误的原因返回，那么这段代码就变得更加复杂：

```text
if (phone == null) {
    throw new ValidationException("phone不能为空");
} else if (!isValidPhoneNumber(phone)) {
    throw new ValidationException("phone格式错误");
}
```

可以想像得到，代码里充斥着大量的类似代码块时，维护成本要有多高。

最后，在这个业务方法里，会（隐性或显性的）抛 `ValidationException`，所以需要外部调用方去try/catch，而业务逻辑异常和数据校验异常被混在了一起，是否是合理的？

在传统Java架构里有几个办法能够去解决一部分问题，常见的如BeanValidation注解或ValidationUtils类，比如：

```text
// Use Bean Validation
User registerWithBeanValidation(
  @NotNull @NotBlank String name,
  @NotNull @Pattern(regexp = "^0?[1-9]{2,3}-?\\d{8}$") String phone,
  @NotNull String address
);

// Use ValidationUtils:
public User registerWithUtils(String name, String phone, String address) {
    ValidationUtils.validateName(name); // throws ValidationException
    ValidationUtils.validatePhone(phone);
    ValidationUtils.validateAddress(address);
    ...
}
```

但这几个传统的方法同样有问题，

**BeanValidation：**

- 通常只能解决简单的校验逻辑，复杂的校验逻辑一样要写代码实现定制校验器
- 在添加了新校验逻辑时，同样会出现在某些地方忘记添加一个注解的情况，DRY原则还是会被违背



**ValidationUtils类：**

- 当大量的校验逻辑集中在一个类里之后，违背了Single Responsibility单一性原则，导致代码混乱和不可维护
- 业务异常和校验异常还是会混杂

所以，有没有一种方法，能够一劳永逸的解决所有校验的问题以及降低后续的维护成本和异常处理成本呢？

### ▍问题3 - 业务代码的清晰度

在这段代码里：

```text
String areaCode = null;
String[] areas = new String[]{"0571", "021", "010"};
for (int i = 0; i < phone.length(); i++) {
    String prefix = phone.substring(0, i);
    if (Arrays.asList(areas).contains(prefix)) {
        areaCode = prefix;
        break;
    }
}
SalesRep rep = salesRepRepo.findRep(areaCode);
```

实际上出现了另外一种常见的情况，那就是从一些入参里抽取一部分数据，然后调用一个外部依赖获取更多的数据，然后通常从新的数据中再抽取部分数据用作其他的作用。这种代码通常被称作“胶水代码”，其本质是由于外部依赖的服务的入参并不符合我们原始的入参导致的。比如，如果`SalesRepRepository`包含一个`findRepByPhone`的方法，则上面大部分的代码都不必要了。

所以，一个常见的办法是将这段代码抽离出来，变成独立的一个或多个方法：

```text
private static String findAreaCode(String phone) {
    for (int i = 0; i < phone.length(); i++) {
        String prefix = phone.substring(0, i);
        if (isAreaCode(prefix)) {
            return prefix;
        }
    }
    return null;
}

private static boolean isAreaCode(String prefix) {
    String[] areas = new String[]{"0571", "021"};
    return Arrays.asList(areas).contains(prefix);
}
```

然后原始代码变为：

```text
String areaCode = findAreaCode(phone);
SalesRep rep = salesRepRepo.findRep(areaCode);
```

而为了复用以上的方法，可能会抽离出一个静态工具类 `PhoneUtils `。但是这里要思考的是，静态工具类是否是最好的实现方式呢？当你的项目里充斥着大量的静态工具类，业务代码散在多个文件当中时，你是否还能找到核心的业务逻辑呢？

### ▍问题4 - 可测试性

为了保证代码质量，每个方法里的每个入参的每个可能出现的条件都要有 TC 覆盖（假设我们先不去测试内部业务逻辑），所以在我们这个方法里需要以下的 TC ：

![img](https://pic2.zhimg.com/80/v2-91a09472af63fccdcfd1231f2882fcfd_1440w.jpg)

假如一个方法有 N 个参数，每个参数有 M 个校验逻辑，至少要有 N * M 个 TC 。

如果这时候在该方法中加入一个新的入参字段 `fax` ，即使 `fax` 和 `phone` 的校验逻辑完全一致，为了保证 TC 覆盖率，也一样需要 M 个新的 TC 。

而假设有 P 个方法中都用到了 `phone `这个字段，这 P 个方法都需要对该字段进行测试，也就是说整体需要：

**P \* N \* M**

个测试用例才能完全覆盖所有数据验证的问题，在日常项目中，这个测试的成本非常之高，导致大量的代码没被覆盖到。而没被测试覆盖到的代码才是最有可能出现问题的地方。

在这个情况下，降低测试成本 == 提升代码质量，如何能够降低测试的成本呢？



## 解决方案

我们回头先重新看一下原始的 use case，并且标注其中可能重要的概念：

一个新应用在全国通过 地推业务员 做推广，需要做一个用户的注册系统，在用户注册后能够通过用户电话号的区号对业务员发奖金。

在分析了 use case 后，发现其中地推业务员、用户本身自带 ID 属性，属于 Entity（实体），而注册系统属于 Application Service（应用服务），这几个概念已经有存在。但是发现电话号这个概念却完全被隐藏到了代码之中。我们可以问一下自己，取电话号的区号的逻辑是否属于用户（用户的区号？）？是否属于注册服务（注册的区号？）？如果都不是很贴切，那就说明这个逻辑应该属于一个独立的概念。所以这里引入我们第一个原则：

**Make Implicit Concepts Explicit**

**将隐性的概念显性化**

在这里，我们可以看到，原来电话号仅仅是用户的一个参数，属于隐形概念，但实际上电话号的区号才是真正的业务逻辑，而我们需要将电话号的概念显性化，通过写一个Value Object：

```text
public class PhoneNumber {
  
    private final String number;
    public String getNumber() {
        return number;
    }

    public PhoneNumber(String number) {
        if (number == null) {
            throw new ValidationException("number不能为空");
        } else if (isValid(number)) {
            throw new ValidationException("number格式错误");
        }
        this.number = number;
    }

    public String getAreaCode() {
        for (int i = 0; i < number.length(); i++) {
            String prefix = number.substring(0, i);
            if (isAreaCode(prefix)) {
                return prefix;
            }
        }
        return null;
    }

    private static boolean isAreaCode(String prefix) {
        String[] areas = new String[]{"0571", "021", "010"};
        return Arrays.asList(areas).contains(prefix);
    }

    public static boolean isValid(String number) {
        String pattern = "^0?[1-9]{2,3}-?\\d{8}$";
        return number.matches(pattern);
    }

}
```

这里面有几个很重要的元素：

1. 通过 `private final String number` 确保 `PhoneNumber` 是一个（Immutable）Value Object。（一般来说 VO 都是 Immutable 的，这里只是重点强调一下）
2. 校验逻辑都放在了 constructor 里面，确保只要 `PhoneNumber `类被创建出来后，一定是校验通过的。
3. 之前的 `findAreaCode` 方法变成了 `PhoneNumber `类里的 `getAreaCode `，突出了 `areaCode` 是 `PhoneNumber` 的一个计算属性。

这样做完之后，我们发现把 `PhoneNumber` 显性化之后，其实是生成了一个 Type（数据类型）和一个 Class（类）：

- Type 指我们在今后的代码里可以通过 `PhoneNumber` 去显性的标识电话号这个概念
- Class 指我们可以把所有跟电话号相关的逻辑完整的收集到一个文件里

这两个概念加起来，构造成了本文标题的 Domain Primitive（DP）。

我们看一下全面使用了 DP 之后效果：

```text
public class User {
    UserId userId;
    Name name;
    PhoneNumber phone;
    Address address;
    RepId repId;
}

public User register(
  @NotNull Name name,
  @NotNull PhoneNumber phone,
  @NotNull Address address
) {
    // 找到区域内的SalesRep
    SalesRep rep = salesRepRepo.findRep(phone.getAreaCode());

    // 最后创建用户，落盘，然后返回，这部分代码实际上也能用Builder解决
    User user = new User();
    user.name = name;
    user.phone = phone;
    user.address = address;
    if (rep != null) {
        user.repId = rep.repId;
    }

    return userRepo.saveUser(user);
}
```

我们可以看到在使用了 DP 之后，所有的数据验证逻辑和非业务流程的逻辑都消失了，剩下都是核心业务逻辑，可以一目了然。我们重新用上面的四个维度评估一下：

### ▍评估1 - 接口的清晰度

重构后的方法签名变成了很清晰的：

```text
public User register(Name, PhoneNumber, Address)
```

而之前容易出现的bug，如果按照现在的写法

```text
service.register(new Name("殷浩"), new Address("浙江省杭州市余杭区文三西路969号"), new PhoneNumber("0571-12345678"));
```

让接口 API 变得很干净，易拓展。

### ▍评估2 - 数据验证和错误处理

```text
public User register(
  @NotNull Name name,
  @NotNull PhoneNumber phone,
  @NotNull Address address
) // no throws
```

如前文代码展示的，重构后的方法里，完全没有了任何数据验证的逻辑，也不会抛 `ValidationException` 。原因是因为 DP 的特性，只要是能够带到入参里的一定是正确的或 null（Bean Validation 或 lombok 的注解能解决 null 的问题）。所以我们把数据验证的工作量前置到了调用方，而调用方本来就是应该提供合法数据的，所以更加合适。

再展开来看，使用DP的另一个好处就是代码遵循了 DRY 原则和单一性原则，如果未来需要修改 `PhoneNumber `的校验逻辑，只需要在一个文件里修改即可，所有使用到了 `PhoneNumber` 的地方都会生效。

### ▍评估3 - 业务代码的清晰度

```text
SalesRep rep = salesRepRepo.findRep(phone.getAreaCode());
User user = xxx;
return userRepo.save(user);
```

除了在业务方法里不需要校验数据之外，原来的一段胶水代码 `findAreaCode `被改为了 `PhoneNumber `类的一个计算属性 `getAreaCode `，让代码清晰度大大提升。而且胶水代码通常都不可复用，但是使用了 DP 后，变成了可复用、可测试的代码。我们能看到，在刨除了数据验证代码、胶水代码之后，剩下的都是核心业务逻辑。（ Entity 相关的重构在后面文章会谈到，这次先忽略）

### ▍评估4 - 可测试性

![img](https://pic4.zhimg.com/80/v2-78c099245efa683fb35a7d7f0ab27833_1440w.jpg)

当我们将 `PhoneNumber `抽取出来之后，在来看测试的 TC ：

- 首先 `PhoneNumber` 本身还是需要 M 个测试用例，但是由于我们只需要测试单一对象，每个用例的代码量会大大降低，维护成本降低。
- 每个方法里的每个参数，现在只需要覆盖为 null 的情况就可以了，其他的 case 不可能发生（因为只要不是 null 就一定是合法的）

所以，单个方法的 TC 从原来的 N * M 变成了今天的 N + M 。同样的，多个方法的 TC 数量变成了

N + M + P

这个数量一般来说要远低于原来的数量 N* M * P ，让测试成本极大的降低。

### ▍评估总结

![img](https://pic4.zhimg.com/80/v2-014d1240f404c93353da3881bc78b8c3_1440w.jpg)

## 进阶使用

在上文我介绍了 DP 的第一个原则：将隐性的概念显性化。在这里我将介绍 DP 的另外两个原则，用一个新的案例。

### ▍案例1 - 转账

假设现在要实现一个功能，让A用户可以支付 x 元给用户 B ，可能的实现如下：

```text
public void pay(BigDecimal money, Long recipientId) {
    BankService.transfer(money, "CNY", recipientId);
}
```

如果这个是境内转账，并且境内的货币永远不变，该方法貌似没啥问题，但如果有一天货币变更了（比如欧元区曾经出现的问题），或者我们需要做跨境转账，该方法是明显的 bug ，因为 `money `对应的货币不一定是 CNY 。

在这个 case 里，当我们说“支付 x 元”时，除了 x 本身的数字之外，实际上是有一个隐含的概念那就是货币“元”。但是在原始的入参里，之所以只用了 `BigDecimal` 的原因是我们认为 CNY 货币是默认的，是一个隐含的条件，但是在我们写代码时，需要把所有隐性的条件显性化，而这些条件整体组成当前的上下文。所以 DP 的第二个原则是：

**Make Implicit Context Explicit**

**将 隐性的 上下文 显性化**

所以当我们做这个支付功能时，实际上需要的一个入参是支付金额 + 支付货币。我们可以把这两个概念组合成为一个独立的完整概念：`Money`。

```text
@Value
public class Money {
    private BigDecimal amount;
    private Currency currency;
    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

而原有的代码则变为：

```text
public void pay(Money money, Long recipientId) {
    BankService.transfer(money, recipientId);
}
```

通过将默认货币这个隐性的上下文概念显性化，并且和金额合并为 `Money` ，我们可以避免很多当前看不出来，但未来可能会暴雷的bug。

### ▍案例2 - 跨境转账

前面的案例升级一下，假设用户可能要做跨境转账从 CNY 到 USD ，并且货币汇率随时在波动：

```text
public void pay(Money money, Currency targetCurrency, Long recipientId) {
    if (money.getCurrency().equals(targetCurrency)) {
        BankService.transfer(money, recipientId);
    } else {
        BigDecimal rate = ExchangeService.getRate(money.getCurrency(), targetCurrency);
        BigDecimal targetAmount = money.getAmount().multiply(new BigDecimal(rate));
        Money targetMoney = new Money(targetAmount, targetCurrency);
        BankService.transfer(targetMoney, recipientId);
    }
}
```

在这个case里，由于 `targetCurrency `不一定和 `money` 的 `Curreny `一致，需要调用一个服务去取汇率，然后做计算。最后用计算后的结果做转账。

这个case最大的问题在于，金额的计算被包含在了支付的服务中，涉及到的对象也有2个 `Currency` ，2 个 `Money` ，1 个 `BigDecimal `，总共 5 个对象。这种涉及到多个对象的业务逻辑，需要用 DP 包装掉，所以这里引出 DP 的第三个原则：

**Encapsulate Multi-Object Behavior**

**封装 多对象 行为**

在这个 case 里，可以将转换汇率的功能，封装到一个叫做 `ExchangeRate `的 DP 里：

```text
@Value
public class ExchangeRate {
    private BigDecimal rate;
    private Currency from;
    private Currency to;

    public ExchangeRate(BigDecimal rate, Currency from, Currency to) {
        this.rate = rate;
        this.from = from;
        this.to = to;
    }

    public Money exchange(Money fromMoney) {
        notNull(fromMoney);
        isTrue(this.from.equals(fromMoney.getCurrency()));
        BigDecimal targetAmount = fromMoney.getAmount().multiply(rate);
        return new Money(targetAmount, to);
    }
}
```

`ExchangeRate`汇率对象，通过封装金额计算逻辑以及各种校验逻辑，让原始代码变得极其简单：

```text
public void pay(Money money, Currency targetCurrency, Long recipientId) {
    ExchangeRate rate = ExchangeService.getRate(money.getCurrency(), targetCurrency);
    Money targetMoney = rate.exchange(money);
    BankService.transfer(targetMoney, recipientId);
}
```



## 讨论和总结



**▍Domain Primitive 的定义**

让我们重新来定义一下 Domain Primitive ：Domain Primitive 是一个在特定领域里，拥有精准定义的、可自我验证的、拥有行为的 Value Object 。

- DP是一个传统意义上的Value Object，拥有Immutable的特性
- DP是一个完整的概念整体，拥有精准定义
- DP使用业务域中的原生语言
- DP可以是业务域的最小组成部分、也可以构建复杂组合

注：Domain Primitive的概念和命名来自于Dan Bergh Johnsson & Daniel Deogun的书 Secure by Design。

### ▍使用 Domain Primitive 的三原则

- 让隐性的概念显性化
- 让隐性的上下文显性化
- 封装多对象行为

### ▍Domain Primitive 和 DDD 里 Value Object 的区别

在 DDD 中， Value Object 这个概念其实已经存在：

- 在 Evans 的 DDD 蓝皮书中，Value Object 更多的是一个非 Entity 的值对象
- 在Vernon的IDDD红皮书中，作者更多的关注了Value Object的Immutability、Equals方法、Factory方法等

Domain Primitive 是 Value Object 的进阶版，在原始 VO 的基础上要求每个 DP 拥有概念的整体，而不仅仅是值对象。在 VO 的 Immutable 基础上增加了 Validity 和行为。当然同样的要求无副作用（side-effect free）。

### ▍Domain Primitive 和 Data Transfer Object (DTO) 的区别

在日常开发中经常会碰到的另一个数据结构是 DTO ，比如方法的入参和出参。DP 和 DTO 的区别如下：

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='731' height='273'></svg>)

### ▍什么情况下应该用 Domain Primitive

常见的 DP 的使用场景包括：

- 有格式限制的 `String`：比如`Name`，`PhoneNumber`，`OrderNumber`，`ZipCode`，`Address`等
- 有限制的`Integer`：比如`OrderId`（>0），`Percentage`（0-100%），`Quantity`（>=0）等
- 可枚举的 `int `：比如 `Status`（一般不用Enum因为反序列化问题）
- `Double` 或 `BigDecimal`：一般用到的 `Double` 或 `BigDecimal `都是有业务含义的，比如 `Temperature`、`Money`、`Amount`、`ExchangeRate`、`Rating` 等
- 复杂的数据结构：比如 `Map> `等，尽量能把 `Map` 的所有操作包装掉，仅暴露必要行为



## 实战 - 老应用重构的流程

在新应用中使用 DP 是比较简单的，但在老应用中使用 DP 是可以遵循以下流程按部就班的升级。在此用本文的第一个 case 为例。

### ▍第一步 - 创建 Domain Primitive，收集所有 DP 行为

在前文中，我们发现取电话号的区号这个是一个可以独立出来的、可以放入 PhoneNumber 这个 Class 的逻辑。类似的，在真实的项目中，以前散落在各个服务或工具类里面的代码，可以都抽出来放在 DP 里，成为 DP 自己的行为或属性。这里面的原则是：所有抽离出来的方法要做到无状态，比如原来是 static 的方法。如果原来的方法有状态变更，需要将改变状态的部分和不改状态的部分分离，然后将无状态的部分融入 DP 。因为 DP 本身不能带状态，所以一切需要改变状态的代码都不属于 DP 的范畴。

(代码参考 PhoneNumber 的代码，这里不再重复)

### ▍第二步 - 替换数据校验和无状态逻辑

为了保障现有方法的兼容性，在第二步不会去修改接口的签名，而是通过代码替换原有的校验逻辑和根 DP 相关的业务逻辑。比如：

```text
public User register(String name, String phone, String address)
        throws ValidationException {
    if (name == null || name.length() == 0) {
        throw new ValidationException("name");
    }
    if (phone == null || !isValidPhoneNumber(phone)) {
        throw new ValidationException("phone");
    }
    
    String areaCode = null;
    String[] areas = new String[]{"0571", "021", "010"};
    for (int i = 0; i < phone.length(); i++) {
        String prefix = phone.substring(0, i);
        if (Arrays.asList(areas).contains(prefix)) {
            areaCode = prefix;
            break;
        }
    }
    SalesRep rep = salesRepRepo.findRep(areaCode);
    // 其他代码...
}
```

通过 DP 替换代码后：

```text
public User register(String name, String phone, String address)
        throws ValidationException {
    
    Name _name = new Name(name);
    PhoneNumber _phone = new PhoneNumber(phone);
    Address _address = new Address(address);
    
    SalesRep rep = salesRepRepo.findRep(_phone.getAreaCode());
    // 其他代码...
}
```

通过 new PhoneNumber(phone) 这种代码，替代了原有的校验代码。

通过 _phone.getAreaCode() 替换了原有的无状态的业务逻辑。

### ▍第三步 - 创建新接口

创建新接口，将DP的代码提升到接口参数层：

```text
public User register(Name name, PhoneNumber phone, Address address) {
    SalesRep rep = salesRepRepo.findRep(phone.getAreaCode());
}
```

▍第四步 - 修改外部调用

外部调用方需要修改调用链路，比如：

```text
service.register("殷浩", "0571-12345678", "浙江省杭州市余杭区文三西路969号");
```

改为：

```text
service.register(new Name("殷浩"), new PhoneNumber("0571-12345678"), new Address("浙江省杭州市余杭区文三西路969号"));
```

通过以上 4 步，就能让你的代码变得更加简洁、优雅、健壮、安全。你还在等什么？今天就去尝试吧！

链接：

① [https://martinfowler.com/bliki/AnemicDomainModel.html](https://link.zhihu.com/?target=https%3A//martinfowler.com/bliki/AnemicDomainModel.html)



**关注我们，后续还有DDD系列分享。**



——————————————————————————————————————

阿里巴巴集团淘系技术部官方账号。淘系技术部是阿里巴巴新零售技术的王牌军，支撑淘宝、天猫核心电商以及淘宝直播、闲鱼、躺平、阿里汽车、阿里房产等创新业务，服务9亿用户，赋能各行业1000万商家。我们打造了全球领先的线上新零售技术平台，并作为核心技术团队保障了11次双十一购物狂欢节的成功。

点击下方主页关注我们，你将收获更多来自阿里一线工程师的技术实战技巧&成长经历心得。另，不定期更新最新岗位招聘信息和简历内推通道，欢迎各位以最短路径加入我们。

[阿里巴巴淘系技术www.zhihu.com![图标](https://pic2.zhimg.com/v2-227b5ef96dc4d1b3a45f7233dfb987e1_ipico.jpg)](https://www.zhihu.com/org/a-li-ba-ba-tao-xi-ji-zhu)





发布于 2020-12-31

[软件架构](https://www.zhihu.com/topic/19556273)

[系统架构](https://www.zhihu.com/topic/19578413)

[软件开发](https://www.zhihu.com/topic/19552332)

赞同 723

51 条评论

分享

喜欢收藏申请转载



赞同 723

分享

### 文章被以下专栏收录

- [![code](https://pic2.zhimg.com/4b70deef7_xs.jpg?source=172ae18b)](https://www.zhihu.com/column/c_1355278956606742528)

- ## [code](https://www.zhihu.com/column/c_1355278956606742528)

- 高质量文章

- [![老包子说架构](https://pic1.zhimg.com/4b70deef7_xs.jpg?source=172ae18b)](https://www.zhihu.com/column/c_1293977603725135872)

- ## [老包子说架构](https://www.zhihu.com/column/c_1293977603725135872)

- 聊聊码农的世界观，开源，技术，生活

### 推荐阅读

- # 阿里技术专家详解DDD系列 第三讲 - Repository模式

- 继续填坑。谢谢大家对这个系列的期待， 持续更新，欢迎关注此账号查收后续内容。第一篇内容地址： 阿里巴巴淘系技术：阿里技术专家详解 DDD 系列 第一讲- Domain Primitive第二篇内容地址：…

- 阿里巴巴淘系技术

- # 阿里技术专家详解DDD系列 第二讲 - 应用架构

- 填坑。谢谢大家对这个系列的期待，持续更新，欢迎关注此账号。 第一篇内容附地址： 阿里巴巴淘系技术：阿里技术专家详解 DDD 系列 第一讲- Domain Primitive 架构这个词源于英文里的“Archi…

- 阿里巴巴淘系技术

- # 阿里技术专家详解 DDD 系列- Domain Primitive

- 导读：对于一个架构师来说，在软件开发中如何降低系统复杂度是一个永恒的挑战，无论是 94 年 GoF 的 Design Patterns ， 99 年的 Martin Fowler 的 Refactoring ， 02 年的 P of EAA ，还是…

- 阿里云云栖...发表于我是程序员

- # 领域驱动设计（DDD）：领域接口化设计

- 领域接口化设计把服务对象（service）和资源库对象（repository）设计成接口是最常见的。但是这对接口化的认识还远远不够，我们需要更深入地去分析接口化设计和更全面地应用接口化编程。所…

- 不够具体发表于领域驱动设...

## 51 条评论

切换为时间排序

写下你的评论...







发布

- [![霜之哀伤](https://pic2.zhimg.com/v2-098e4d5ec0d38d248f30df4ab51098f7_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/shuang-zhi-ai-shang-57-66)[霜之哀伤](https://www.zhihu.com/people/shuang-zhi-ai-shang-57-66)01-08

  首先感谢分享！然后呢，我还是觉得落地ddd需要代价，ddd的好处不是免费的。像例子中这样的问题，项目中的确存在，不过也不严重。但是ddd带来的问题，还没讨论怎么解决。一个是开发效率，需要更多的思考，更多的类，更多的代码。一个是和数据库数据类型的映射。

  9回复踩 举报

- [![less2](https://pic1.zhimg.com/3de94eefe0564225983db7ebcb6afaf7_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/less2)[less2](https://www.zhihu.com/people/less2)01-02

  DDD in practice 。vo还是要充分考虑是否一定需要，不然滥用比不用更恶心

  9回复踩 举报

- [![Forward](https://pic1.zhimg.com/v2-4a10907d36fe17d7b628f61ec5bed531_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/liu-bai-forever)[Forward](https://www.zhihu.com/people/liu-bai-forever)回复[less2](https://www.zhihu.com/people/less2)01-04

  agree

  赞回复踩 举报

- [![xpwu](https://pic2.zhimg.com/v2-11d66a1c5eac3f0767bdb35b145307be_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/xpwu-32)[xpwu](https://www.zhihu.com/people/xpwu-32)02-01

  照搬不是一个好现象。既然是阿里的，完全可以对着你的工程实践来与其他人聊你使用的真实过程，以及你的理解。另外一句话，真实的工程一切都是一个度及其演化。

  4回复踩 举报

- ![知乎用户](https://pic2.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户04-06

  ddd也不是银弹，不管你用何种范式来开发应用，业务本身的复杂度是规避不了的，最终仍然会体现在代码上，大多数公司的业务其实没有那么复杂，在我看来其实采用贫血模式也无妨，反而更容易理解。对于文章所说的ef和hibernate助长了贫血模式，我表示完全反对，我反而觉得他们是适合充血模式的，倒是MyBatis这种框架会助长开发人员倾向于面向表结构编程

  3回复踩 举报

- [![薛赵明](https://pic1.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/inter12)[薛赵明](https://www.zhihu.com/people/inter12)03-09

  错误还是蛮多的

  1.对象的属性缺乏修饰符,没有private

  2.对象属性方式居然是直接 User.name,违反了面向对象的封装

  3.为了Make Implicit Concepts Explicit 把phoneNUmber强提为一个对象，完全错误的做法，领域抽象更应该是基于实际业务模型来定义，而不是为了原则去抽象，PhoneNUmber从业务建模建模角度就不是一个对象。

  4.可测试行基本是混淆概念，任何的测试纬度不会应该你的重构而减少，条件的是客观存在的，无非是形式的替换。

  

  这篇文章基本是误导人。

  2回复踩 举报

- [![果粒橙](https://pic1.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/vzzzzzzzz)[果粒橙](https://www.zhihu.com/people/vzzzzzzzz)回复[薛赵明](https://www.zhihu.com/people/inter12)03-15

  这。。 我看完 感觉 好矛盾 既觉得 好高级，一用起来 又觉得有点麻烦,db的时候 还得考虑数据类型。 都不知道 该不该用了。。

  赞回复踩 举报

- [![moonlight](https://pic2.zhimg.com/f08a592c181029383688d19bf9709d73_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/moonlight-92-17)[moonlight](https://www.zhihu.com/people/moonlight-92-17)01-04

  ef助长了贫血模型？这是怎么得来的结论？恰恰相反，ef是当前最适合充血模型的工具，它是在认认真真地解决阻抗失配的问题，毕竟其设计思想是基于马丁福勒的企业应用架构模式。当然，如果你只是将dbfirst就当做是ef当我没说。另外在Java这边我没发现什么好的对象跟踪实现和orm。
  对，mybatis只能算dmt而不是orm。

  2回复踩 举报

- [![大宽宽](https://pic1.zhimg.com/v2-0d277f6cffdf1141d9aaa7763b92b9d7_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/xing-jiankuan)[大宽宽](https://www.zhihu.com/people/xing-jiankuan)01-01

  请问DP在DAO层转换类型是否有比较成熟的落地方案？毕竟到了db还是得变成varchar，int……

  2回复踩 举报

- [![霜之哀伤](https://pic1.zhimg.com/v2-098e4d5ec0d38d248f30df4ab51098f7_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/shuang-zhi-ai-shang-57-66)[霜之哀伤](https://www.zhihu.com/people/shuang-zhi-ai-shang-57-66)回复[大宽宽](https://www.zhihu.com/people/xing-jiankuan)01-08

  这个好解决。主要是dp需要更多的思考，更多的类，更多的代码，导致开发效率降低

  赞回复踩 举报

- [![unforgiven](https://pic1.zhimg.com/v2-c2514b4c65735d5bdfea7da0f00bcf0a_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/xu-hui-65-4)[unforgiven](https://www.zhihu.com/people/xu-hui-65-4)回复[大宽宽](https://www.zhihu.com/people/xing-jiankuan)8 小时前

  Jpa embeded 注解

  赞回复踩 举报

- [![xiao liu](https://pic4.zhimg.com/v2-56ac7bfd9ba009f96277ca66e4c52962_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/xiao-liu-44-86)[xiao liu](https://www.zhihu.com/people/xiao-liu-44-86)01-22

  中国的技术大牛都是学习人家的东西 外国大牛都是搞出新的玩意。

  1回复踩 举报

- [![熊辉](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/xiong-hui-21-44)[熊辉](https://www.zhihu.com/people/xiong-hui-21-44)01-01

  ![[赞同]](https://pic2.zhimg.com/v2-419a1a3ed02b7cfadc20af558aabc897.png)

  1回复踩 举报

- [![布莱恩特](https://pic4.zhimg.com/v2-dd367acde8723b7e916ff3a482d639ec_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/tan-chen-56-80)[布莱恩特](https://www.zhihu.com/people/tan-chen-56-80)2020-12-31

  我司首席科学家Martin Fowler![[赞同]](https://pic2.zhimg.com/v2-419a1a3ed02b7cfadc20af558aabc897.png)

  1回复踩 举报

- [![一鲸](https://pic1.zhimg.com/v2-5bb58ff82ef2747759ad82e5d96da9e6_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/wei-wei-2-34-86)[一鲸](https://www.zhihu.com/people/wei-wei-2-34-86)回复[布莱恩特](https://www.zhihu.com/people/tan-chen-56-80)01-01

  原来是 thought works 的朋友![[爱]](https://pic1.zhimg.com/v2-0942128ebfe78f000e84339fbb745611.png)

  1回复踩 举报

- [![熊一木](https://pic1.zhimg.com/v2-4e800341edfebaa5f26a5a38941ed104_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/jeffbear)[熊一木](https://www.zhihu.com/people/jeffbear)04-03

  谢谢分享，很赞，普及了不少DDD的相关知识。不过有点几个小地方可能值得商榷。
  1）SOA，就我个人的经验，大概在2006-2007年就已经炒起来的。当时我在IBM中国的时候整天听到周围都是这些。我们现在讲的Microservice（微服务）算是从那时的SOA概念发展过来的；
  2）2012年的时候，在硅谷这边的但凡是互联网公司几乎没有公司是单机了；大概从10年开始AWS这种SaaS/IaaS就开始有很大的用户增长了；很多都在上面直接Host ALB + EC2 Cluster 等等。还有好多公司直接租data center什么的。

  赞回复踩 举报

- ![知乎用户](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户03-25

  DDD就是玩概念，没有足够的经验，啥东西都是空谈

  赞回复踩 举报

- [![Cliveyou](https://pic1.zhimg.com/v2-19f1577dd50e04c323c9524b6a5abf1a_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/taozhiwei-72)[Cliveyou](https://www.zhihu.com/people/taozhiwei-72)03-21

  想知道UserId这个类的具体用法![[思考]](https://pic4.zhimg.com/v2-bffb2bf11422c5ef7d8949788114c2ab.png)

  赞回复踩 举报

- [![cccccccccccccccc](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/sun-peng-8-95)[cccccccccccccccc](https://www.zhihu.com/people/sun-peng-8-95)03-19

  看一遍 会一点 看多很多遍 会了很多

  赞回复踩 举报

- [![盘翎](https://pic4.zhimg.com/v2-3de59791e9d9ec3aa48efb6921946c85_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/pan-ling-56-6)[盘翎](https://www.zhihu.com/people/pan-ling-56-6)03-16

  感谢讲解，有所收获。

  ![img](https://pic2.zhimg.com/v2-90359a720808ff45062287127cfa1039.gif)

  赞回复踩 举报

- [![流年忘返](https://pic1.zhimg.com/705541815360349e042fbb3dc5d534fe_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/fei-xiao-long)[流年忘返](https://www.zhihu.com/people/fei-xiao-long)03-13

  由数据domain自己处理，外部提供功能。
  这种思路没问题，在实例团队合作中，用写静态工具处理，在业务流程中显示调用成本要低一些。
  这种完全封装的方式适合业务稳定，需求变更不频繁的情况。

  赞回复踩 举报

- ![知乎用户](https://pic1.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户03-12

  关注了，希望不是你们的kpi指标，刮一阵风。

  赞回复踩 举报

- ![知乎用户](https://pic1.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)知乎用户03-05

  如果说校验需要联动其他字段校验或者查库校验，比如省份城市，要怎么解决？

  赞回复踩 举报

- [![cccccccccccccccc](https://pic4.zhimg.com/da8e974dc_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/sun-peng-8-95)[cccccccccccccccc](https://www.zhihu.com/people/sun-peng-8-95)02-25

  速度 我都等不及了。快点出第四篇

  赞回复踩 举报

- [![王乐乐](https://pic4.zhimg.com/v2-b67b6397aafe032a483c3c1f49cc20c6_s.jpg?source=06d4cd63)](https://www.zhihu.com/people/wang-le-le-13-72)[王乐乐](https://www.zhihu.com/people/wang-le-le-13-72)02-23

  这个就要考虑到项目初始规划的时候怎么设计了，对不同的业务DDD也不同，需多方面考量![[思考]](https://pic4.zhimg.com/v2-bffb2bf11422c5ef7d8949788114c2ab.png)

  赞回复踩 举报

123下一页

- [首页](https://www.zhihu.com/)
- [会员](https://www.zhihu.com/xen/vip-web)
- [发现](https://www.zhihu.com/explore)
- [等你来答](https://www.zhihu.com/question/waiting)



登录加入知乎

# 领域驱动设计(DDD)靠谱吗？

关注问题写回答

[IT 工程师](https://www.zhihu.com/topic/19550910)

[程序员](https://www.zhihu.com/topic/19552330)

[Java](https://www.zhihu.com/topic/19561132)

[IT 行业](https://www.zhihu.com/topic/19587634)

[领域驱动设计（DDD）](https://www.zhihu.com/topic/19826540)

# 领域驱动设计(DDD)靠谱吗？

怎么感觉在国内没什么人用啊显示全部 

关注者

**214**

被浏览

**91,542**

关注问题写回答

邀请回答

好问题 12

添加评论

分享



#### 29 个回答

默认排序

[![你微笑时很美](https://pic2.zhimg.com/v2-dc8cd91d459a8554c4b6930187f370b6_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/ni-wei-xiao-shi-hen-mei-7-7)

[你微笑时很美](https://www.zhihu.com/people/ni-wei-xiao-shi-hen-mei-7-7)

不只做一名编码者，更要做一名思考者

185 人赞同了该回答

由于公司领导的要求，所有的软件开发都要将DDD作为指导思想，并且要接受敏捷的思想；"迫不得已"下拜读了《实现领域驱动设计》这本书，将会在公司的内部系统上全面实践DDD。DDD把领域模型的重要性提高到了数据模型之上，在传统的MVC分层架构下。我们将项目结构分为Controller，Service，DAO 这三个主要的层，所有的业务逻辑都在Service中体现，而我们的实体类Entity却只是充当一个与数据库做ORM映射的数据容器而已，它并没有反映出模型的业务价值。所以又把这种模型称为“贫血模型”。“贫血模型”有什么坏处呢？在我们的代码 中将会到处看到各种的setter方法和各种各样的参数校验的代码，尤其是在Service层，但是这些代码它并没有反映出它的业务价值。这就是事务脚本的架构下，所呈现出来的弊端，这种模式下认为数据模型优先，所以会导致开发人员和产品经理在讨论问题的时候，完全是从两个角度在思考问题。开发人员听到需求后，脑袋里想的并不是如何反应出业务的价值，而是考虑的是数据库表怎么设计，字段该怎么加这些问题。所以DDD中提出了通用语言这么一个概念，并且基本将通用语言的概念贯穿于整个落地的过程。这样会大大的减少成员之间的沟通成本（前提是大家都从心里接受了DDD）。

为什么DDD难以落地呢？
第一，国内关于DDD的最佳实践还是太少了，除了知名的几个大厂以外很少看到有关于DDD的落地实践。这里附上美团的DDD实践，[美团领域驱动设计](https://link.zhihu.com/?target=https%3A//tech.meituan.com/2017/12/22/ddd-in-practice.html)；最佳实践太少意味着，我们可以参考的资料就少，承担的项目失败的风险就大。第二，DDD中出现了很多的概念和术语，比如 聚合根，值对象，六边形架构，CQRS(命令和查询职责分离)，事件驱动等等概念。很多人看到了这么多概念的时候，心中就开始打退堂鼓了。第三，DDD需要我们在领域建模花费很多的时间和精力，而且还可能导致付出和收益不成正比的情况。因为在界限上下文的划分上是非常考验架构师的业务水平。如果没有将业务模型很好的识别出来，那么可能很快模型就会在迭代的过程中腐败掉了。
上面就只是我自己对DDD的一些简单的理解，知识问题有不正确的地方，可以指正交流。（勿喷）

[编辑于 2020-05-29](https://www.zhihu.com/question/328870859/answer/1252486665)

赞同 18519 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![Johny Sinn](https://pic4.zhimg.com/v2-e1a027153bbad97530494eac3e21d0b4_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/linzihao-239)

[Johny Sinn](https://www.zhihu.com/people/linzihao-239)

一位见多识广、智慧超凡的IT人 & 超级猛男

47 人赞同了该回答

领域驱动设计DDD越来越受到重视，国内有很多团队在使用领域驱动设计DDD，但是每一个团队对DDD的理解可能不一样。如果领域的设计不能很好地指导开发工作，那么DDD的威力就发挥不出来了。我们接触到的“领域驱动”，很有可能是假的“领域驱动”，是别人理解消化后的领域驱动，即使是架构师或者项目经理，他们不一定真正理解领域驱动，**人们对领域驱动有很多误解，****比如“领域驱动只适合大型项目”，****“领域驱动会加大工作量，多加了一层Domain，太多转换”，****“领域驱动应该采用贫血模型”等等，****这些都是错误的认知。**Java项目的架构通常是采用MVC三层架构，MVC全名是Model View Controller，是模型Model－视图View－控制器Controller的缩写，但是MVC跟领域驱动的分层架构是不同。领域驱动的分层架构：**用户界面层/表示层Facade**用户界面层负责向用户显示信息和解释用户指令。这里指的用户可以是另一个计算机系统，不一定是使用用户界面的人。该层包含与其他应用系统（如Web服务、RMI接口、Web应用程序以及批处理前端）交互的接口与通信设施。它负责Request的解释、验证以及转换。另外，它也负责Response的序列化，如通过HTTP协议向web浏览器或web服务客户端传输HTML或XML，或远程Java客户端的DTO类和远程外观接口的序列化。该层的主要职责是与外部用户（包括Web服务、其他系统）交互，如接受用户的访问，展示必要的数据信息。用户界面层facade目录：（1）api存放Controller类，接受用户或者外部系统的访问，展示必要的数据信息。（2）handle存放GlobalExceptionHandler全局异常处理类或者全局拦截器。（3）model存放DTO（Request、Reponse）、Factory（Assembler）类，Factory负责数据传输对象DTO与领域对象Domain相互转换。应用层Application应用层定义了软件要完成的任务，并且指挥表达领域概念的对象来解决问题。该层所负责的工作对业务来说意义重大，也是与其他系统的应用层进行交互的必要通道。应用层要尽量简单。它不包含任务业务规则或知识，只是为了下一层的领域对象协助任务、分配工作。它没有反映业务情况的状态，但它可以具有反映用户或程序的某个任务的进展状态。应用层主要负责组织整个应用的流程，是面向用例设计的。该层非常适合处理事务，日志和安全等。相对于领域层，应用层应该是很薄的一层。它只是协调领域层对象执行实际的工作。应用层中主要组件是Service，因为主要职责是协调各组件工作，所以通常会与多个组件交互，如其他Service，Domain、Factory等等。应用层application目录：（1）service存放Service类，调用Domain执行命令操作，负责Domain的任务编排和分配工作。（2）external存放ExternalService类，负责与其他系统的应用层进行交互，通常是我们主动访问第三方服务。（3）model存放DTO（ExtRequest、ExtReponse）类。应用层application可以合并到领域层biz目录。领域层/模型层Biz领域层主要负责表达业务概念，业务状态信息和业务规则。Domain层是整个系统的核心层，几乎全部的业务逻辑会在该层实现。领域模型层主要包含以下的内容：实体(Entities):具有唯一标识的对象值对象(Value Objects): 无需唯一标识。领域服务(Domain): 与业务逻辑相关的，具有属性和行为的对象。聚合/聚合根(Aggregates & Aggregate Roots): 聚合是指一组具有内聚关系的相关对象的集合。工厂(Factories): 创建复杂对象，隐藏创建细节。仓储(Repository): 提供查找和持久化对象的方法。领域层biz目录：（1）domain存放Domain类，Domain负责业务逻辑，调用Repository对象来执行数据库操作。Domain没有直接访问数据库的代码，具体的数据库操作是通过调用Repository对象完成的。注意，除了CQRS模式外，Repository都应该是由Domain调用的，而不是由Service调用。（2）repository存放Repository类，调用Dao或者Mapper对象类执行数据库操作。（3）factory存放Factory类，负责Domain和实体Entity的转换。基础设施层Infrastructure基础设施层为上面各层提供通用的技术能力：为应用层传递消息，为领域层提供持久化机制，为用户界面层绘制屏幕组件。基础设施层以不同的方式支持所有三个层，促进层之间的通信。基础设施包括独立于我们的应用程序存在的一切：外部库，数据库引擎，应用程序服务器，消息后端等。基础设施层Infrastructure目录：（1）commons存放通用工具类Utils、常量类Constant、枚举类Enum、BizErrCode错误码类等等。（2）persistence存放Dao或者Mapper类，负责把持久化数据映射成实体Entity对象。注意，领域对象是具有属性和行为的对象，是有状态的，数据和行为都是可以重用的。在很多Java项目中，Service类是无状态的，而且一般是单例，业务逻辑直接写到Service类里面，这种方式本质上是面向过程的编程方式，丢失了面向对象的所有好处，重用性极差。应用“贫血模型”会把属于对象的数据和行为分离，领域对象不再是一个整体，破坏了领域模型。只有对领域驱动有足够的认知，工程师才会正确运用领域的理念去编程，告诉工程师业务逻辑是写在Domain不管用，他们可能还是会在Service里面写业务逻辑代码，创建很多private私有方法。**领域驱动开发的关注点在于领域模型，****所有的考虑都应该从领域的角度出发，重心放在业务。****领域模型必须能够精准地表达业务逻辑，****领域模型需要在开发过程中不断被完善，****并且能够指导工程师的开发工作。**[![img](https://pic1.zhimg.com/v2-60cff800a48c2e396ac1ff5c97c12e9e_hd.jpg?source=b555e01d)ZMI紫米PurPods 真无线降噪音乐蓝牙耳机 通话降噪超长京东¥ 199.00去购买](https://union-click.jd.com/jdc?e=jdext-1373816220936933377-0&p=JF8AARUDIgZlGFwSBxUPVxxdFzISBlQaWxQDEwBSG1slRk1fC0RrTEdXRhcQRQtaV1MJBABAHUBZCQVbFAMTB1QaWhIFEgdKQh5JXyJGAFoFYGYXcjVrC252b3wzHCJBYmZRWRdrFQsSBV0aWhUFETdVGloVBhcEVh1aJTISAmVNNRUDEwZUGlkWABI3VCtbEgETBVYZWRUCEA5UK1sdBiLR-4-Onb3Lt_DN8bvXn7eAkvDBvJQ3ZStYJVlHUxxeRxUAFAVcG1wWARMPVxxTFwAQAVMHWiUCEwZWH1kdBxQFOxhaFwobAzsZWhQDEAVQH10TMhI3VisFewNBB1cZUhxVfF0LTlxKRE5OOxhcFgQQAlwfaxcDEwVX)

[编辑于 05-07](https://www.zhihu.com/question/328870859/answer/1435876181)

赞同 479 条评论

分享

收藏喜欢收起

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![方丈的寺院](https://pic1.zhimg.com/v2-91a9607f8197a4413d31db77852d110d_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/fang-sheng-62-75)

[方丈的寺院](https://www.zhihu.com/people/fang-sheng-62-75)

程序员/马桶上思考人生

53 人赞同了该回答

DDD落地为什么这么难敏捷迭代，放弃建模现代互联网产研团队的构成一般是市场/运营、产品、UI交互、前端、后端、测试。这些角色的分工是将一个产品开发上线的各个过程拆分出来，然后每个过程专人负责，可以有效提高生产效率，这一套流程是标准的**流水线作业**。这样做的好处毋庸置疑，坏处也很明显，每个人只盯着自己那一块，而**忽略整体**了。再来看DDD,领域建模设计核心有两个统一语言(软件的开发人员/使用人员都使用同一套语言，即对某个概念，名词的认知是统一的）面向领域（以领域去思考问题，而不是模块）为了实现这两个核心，需要一个关键的角色，领域专家。他负责问题域，和问题解决域，他应该通晓研发的这个产品需要解决哪些问题，专业术语，关联关系。这个角色一般团队是没有配备的。最接近这个角色的就是产品了，但实际上产品并不是干这个活的。在我们团队落地过程中，有一段时间苦于没有领域专家，我想push产品成为领域专家，担当起这个角色。 最后不了了之，产品很配合，但是内驱力不强。为什么内驱力不强，因为给他带来的收益不够。前面已经提到敏捷迭代后，每个角色都是流水线上的螺丝钉，大家都只盯着自己这一块。对自己有利的去参与，和自己无关的不管。我们先看统一语言与面向领域的好处因为大家都使用统一的一套通用语言，所以沟通成本会大大减小，不会在讨论A的时候以为是B。对使用产品的用户有好处，他能在产品不断更新过程中，有一套统一流畅的体验。用户不用在每次软件更新时都要抱怨为什么之前的一个数据保存后没有用到了。面向领域去开发产品有助于我们深入分析产品的内在逻辑，专注于解决当前产品的核心问题，而不是冗余的做很多功能模块，或者几个用户/运营反馈的问题就去更改产品逻辑，完了上线后用户不用，你还在那边骂用户朝三暮四，乱提需求。这些好处粗看一下，其实对产品研发的各个角色都有意义。但细看一下呢，沟通成本大大减小，对于运营，产品，UI交互没啥问题。一个问题理解的不一致，组织个会议，大家好好聊聊就行了。用户体验一致对产研团队有啥好处呢，反正用户骂的不是我，是客户和运营。深入分析产品的内在逻辑有啥用呢？一款产品的成功有很多因素，主要靠上面，我只是一个小兵，我管不了那么多。有空我多研究研究我的专业领域，多去看几篇面试文章。产品黄了，我好跳槽。因为本人是后端研发，所以这里不对其他角色过多展开。只想对研发说，你跳槽换个公司就好了吗？还是crud boy。还是重复着写着很多冗余的功能，冗余的代码。需求方让你写什么，你就写什么，最后在一天天的加班中丧失了对代码的兴趣，没有了梦想。 我们都知道改变别人很难，所以先从改变自己开始，先让自己变优秀了，才能影响他人框架易学，思想难学如果抛开其他角色，单从研发角度考虑DDD呢。开发进行领域建模，然后遵从康威定律，将软件架构设计映射到业务模型中。（虽然这个领域，开发可能识别的不对，暂且忽略这个问题）康威（梅尔·康威）定律 任何组织在设计一套系统（广义概念上的系统）时，所交付的设计方案在结构上 都与该组织的沟通结构保持一致。
纯研发实施DDD,为什么也这么难呢？没有标准DDD是一套思想，一套领域建模设计，一套在特定上下文环境中使用的。所以在1千个团队中实行DDD,可能有1千套不同的方案。一个实行DDD多年的人，换了一个公司，换了一个团队，把他原有的那套带过去，推行下去，一般都不适用。所以DDD的学习和实践不像学习一个函数，API，框架那样有直接的反馈效果，他需要结合团队的实际情况去实行，才能达到效果。期待DDD解决所有的问题程序员都是很实际的，没有好处的东西是不会去做的。你必须能够有效的帮助他提升，他才会去接受。 比如当初有团队成员提出来，我们实行了这一套后，是不是不用加班了，或者加班时间可以减小。
有测试提出实行这一套后，bug率能降低多少。
研发需要一个可以量化的效果，抱歉DDD做不到。没有哪个团队实行了DDD后，解决了软件开发的所有问题。关于这一点，可以读一下[驱动方法不能改变任何事情](https://link.zhihu.com/?target=https%3A//www.infoq.cn/article/star-driven-approaches)

[发布于 2019-06-22](https://www.zhihu.com/question/328870859/answer/723586073)

赞同 536 条评论

分享

收藏喜欢收起

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![logo](https://pic3.zhimg.com/v2-b6979a32f5ef8d20aa071f980b81bf36_250x250.jpeg)PlayStation](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

广告



不感兴趣[知乎广告介绍](https://www.zhihu.com/promotion-intro)

[PS Store 假期限时优惠来袭！多款热门游戏2折起！PS Store假期限时优惠来了，包括 Little Nightmares II、Ghost of Tsushima、Persona 5 The Royal、Grand Theft Auto V 等多款热门游戏低至2折起！查看详情](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

[![曾著](https://pic1.zhimg.com/v2-37145f435b6b361cfe9df42bd15698d8_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/joeaniu)

[曾著](https://www.zhihu.com/people/joeaniu)

互联网创业者，关注高性能并发系统、移动互联网、创业、敏捷开发

20 人赞同了该回答

简单回答一下。 如果你说的DDD是《DDD》和《iDDD》书中写的模式和实践，例如CQRS，那的确用的人不多。 但如果你说的DDD是Bounded Context，Domain这些思路，那几乎所有软件都在用，可能只是没用DDD这套概念去阐述而已。

[发布于 2019-06-12](https://www.zhihu.com/question/328870859/answer/712818348)

赞同 20添加评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![悬壶醉世](https://pic1.zhimg.com/v2-f6170b6e6274207186539d4ec6104f5c_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/deng-hang-hui-92)

[悬壶醉世](https://www.zhihu.com/people/deng-hang-hui-92)

每次答题瞎逛的时候问问自己今天工作学习有没有尽力。

4 人赞同了该回答

靠谱。。。。。DDD是面向架构思想的书。适合对于项目有统筹能力的人。我认为书中有几个要点1，统一沟通语言。2，领域专家。3，一个符合业务现状的模型，持续管理模型。
实际上这些东西在小公司的实践上会有所难度。难度在于：1，我们给沟通模型，实际走到了现在的话我们前端，后台，产品甚至运维使用的模型都千差万别。文档也是如此。因此我们很难去统一沟通语言。能做到的仅仅只是，每个工种的模型统一。2，我们的领域专家是产品经理，实现的模型与开发所需不符。需求改来改去，模型也没维护……但是真的安排一个中间人负责沟通产品，维护模型，说实话有点难。虽然可以实现，但大概不会有人这么做。3，模型的管理会有很多问题。比如临时的变更，比如业务的扩大，个人能力维护这个模型也挺麻烦的。所以需要培养同事对模型进行共同维护。。。。分工，版本管理又是一些麻烦事。。所以具体实现上也需要各自想办法。。。
用是有人在用的。。。。。微服务本来也是DDD的一个实现。大型公司也有成熟案例。
可是实现上来说，很少有人敢说自己的实现很好。DDD作为一系列思想也没给出具体的指导，大多都实现都与业务相关，所以不好吹。。。。
但是实现的过程中，你会发现DDD虽然很麻烦但是真的解决了很多问题，即使无法量化。

[发布于 2020-07-05](https://www.zhihu.com/question/328870859/answer/1319414023)

赞同 4添加评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![Bird Frank](https://pic1.zhimg.com/16137c755_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/bird-frank)

[Bird Frank](https://www.zhihu.com/people/bird-frank)

SoftwareDeveloper

3 人赞同了该回答

DDD是靠谱的，但在国内用的人确实也不多。这一个可以看知乎上与DDD相关的问题和文章点赞数、回答量和评论数都不多，另一个可以看很多公司对架构师、项目经理等的招聘要求也还没有出现DDD的字样。DDD在国内使用不多的原因可能有这样几个，这几个原因都和DDD本身是否靠谱无关。无知者无谓。DDD是用于分析、解决复杂业务领域的方法。很多项目的开发者不能在初期对项目所涉业务领域和需求的复杂度做出正确的评估，并往往低估业务复杂性。这导致他们不会去寻求能帮助理解、分析复杂业务系统的方法。现在国内的项目也可能确实不是很复杂，但是表面看来只是一些表单加一些CRUD操作的应用却往往也会隐藏一两个比较复杂的业务逻辑，这些业务逻辑还往往与核心业务相关。等到开发团队跌进这个坑里的时候往往已经太晚了。项目开发中的短视行为。DDD的很多方法、原则的目的是为了提升软件系统的可维护性、可扩展性、应对业务需求变更的能力。在很多外包项目中，甲方的关注点在降低成本（特别是业务部门和采购部门决策权分开的情况），而乙方的关注点则是在满足的交付的前提下利润最大化。我更是见过要求软件系统的开发商和维护商不能是同一家企业的奇葩规定。这种情况下，开发者（一方）自然不会把很多精力去提升“可维护性、可扩展性、应变变更”这些长远见效的、比较“需”的事情上。错误的软件质量观。Uncle Bob 说过，应该用软件开发所耗费的精力来衡量软件质量，这个精力包括了软件全生命周期中所耗费的开发、维护等工作的精力。但是现在很多甲方和软件公司对软件质量的评估依然主要看缺陷率，他们也没有能力对可维护性、可扩展性这些指标进行评估，这也导致上面第2点说的“短视行为”。程序员的能力。DDD是默认了采用面向对象的分析方法与思维方式的。但是很多程序员，包括很多所谓架构师、Team Leader，对OOP都没有很好掌握和运用，更不用说面向对象的分析设计了。

[发布于 03-02](https://www.zhihu.com/question/328870859/answer/1757729846)

赞同 31 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![古明地觉](https://pic1.zhimg.com/v2-deebf49946a10c35dd08a7ff0855ed65_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/gu-ming-di-jue-89-16)

[古明地觉](https://www.zhihu.com/people/gu-ming-di-jue-89-16)

养猫，养鸟，喜欢在网上冲浪

8 人赞同了该回答

DDD就是玄学在没有领域专家的情况下，每个人设计的领域对象肯定大有不同，一个团队里十个开发，能设计出十种不通的领域划分，最后迭代几个版本，代码根本没法维护互联网开发新技术新理念层出不穷，每段时间都会出来一个奇葩东西，最后被淘汰现在DDD就是，随便在知乎上抓个人就能给你长篇大论一套DDD原理，让他写个Demo放Github上，都够呛能写出来我们团队在设计一个非常复杂系统的时候用过DDD，而且一直在用，坑很多，最后还是得看本质1.这个技术用了能提高员工多少效率2.这个技术带来了什么好处3.这个技术维护成本和设备成本是不是增加了用了DDD我花了多久时间构思领域模型，花了多久时间和小伙伴讨论领域模型，花了多少时间打代码，花了多少时间改bug，如果用了DDD，你办公效率反倒下降了，团队间讨论更频繁了，那我强烈建议放弃这个技术，没必要为了追赶潮流而学习技术，计算机技术永远是避繁就简，互联网永远是敏捷开发。DDD这个概念国内已经炒了几年了，好不好用试试就知道了。

[编辑于 03-16](https://www.zhihu.com/question/328870859/answer/1713027163)

赞同 86 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![只喝可乐的猫](https://pic4.zhimg.com/v2-b5690808bebbe1583ee0e6399d410816_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/zheng-zhe-ming-33)

[只喝可乐的猫](https://www.zhihu.com/people/zheng-zhe-ming-33)

Ones and Zeros

4 人赞同了该回答

哈哈，原因是 ddd vs transaction script 并没有说 ddd 一定好，而领域模型设计和具体业务关联紧密，而ddd在社区没有被广泛接受的最佳实践案例，真正能搞懂用明白ddd的人并不多，所以工程领域当然选择简单明了的更符合直觉的transaction script 编程模型啦，更不要说大家手头干的绝大多数活都是搬砖crud，哪有什么复杂逻辑。不过，ddd 中的一些设计模式已经被发扬光大广泛应用了，比如 SpringData 中的 Repository，用的人不要太多。

[编辑于 2019-06-29](https://www.zhihu.com/question/328870859/answer/713103450)

赞同 4添加评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![kyssky](https://pic1.zhimg.com/v2-62ac5b08d6ae4e3ec8b55145e5e25e65_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/kyssky_p)

[kyssky](https://www.zhihu.com/people/kyssky_p)

彡(￣_￣；)彡风中凌乱

28 人赞同了该回答

DDD这玩意就是旧酒装新壶 , 没有什么创新的东西要是真的牛逼就不会这么就都没有火了 , 微服务也救不了他只相信一句真理 , 软件工程本身的复杂度是没有办法解决的 , 我们只能尽可能的将代码的运行逻辑梳理清楚 , 尽可能的将关键逻辑分离 ,方便以后修改

[发布于 2020-07-09](https://www.zhihu.com/question/328870859/answer/1328408231)

赞同 282 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![萧萧](https://pic4.zhimg.com/v2-f0607f488dcb7cf3bac107335c5f7683_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/xiao-xiao-99-94)

[萧萧](https://www.zhihu.com/people/xiao-xiao-99-94)

10 人赞同了该回答

﻿# 参考资料 - Domain-Driven Design: Tackling Complexity in the Heart of Software前言领域驱动设计是一个传播非常广泛的名词, 笔者对这个概念听闻很早,但是一直也没有提起太大的兴趣去学习, 愿因很简单, 因为它属于一种工程设计思想, 解决的问题不够标准具象, 其内在价值存在争议.但无奈工作中恰恰就有一个公司内部自建的 "中台化解决方案" 的框架, 号称运用了 DDD 的思想, 虽然笔者不太认同, 但是也要先学习才能有发言权 。虽然市面上充斥着各种博文,教程都说 Eric Evans 的原书偏概念, 缺乏实现细节. 所谓”取法乎上，得其中“， ”取法乎中， 得其下“ , 为了真正弄清楚DDD 的核心价值到底在哪, 笔者选择了阅读 Eric Evans 的原书, 真正阅读完毕后发现: - **事实恰恰相反, DDD 原书非常接地气, 而那些所谓解读 DDD 的博客或者教程才是将 DDD 概念抽象化的罪魁祸首**![img](https://pic1.zhimg.com/80/v2-41eb39d9541ce79a7ffab157e1ca502f_1440w.jpg?source=1940ef5c)
所以本篇文章旨在将 DDD 原书中的一些核心思想简单地传达出来. 希望可以帮助一些感兴趣的初学者. 笔者会尽可能引用书中的原话来传达自己的观点, 涉及到概念定义的, 会同时引用英文. 避免一些 DDD 的信徒口诛笔伐, 攻击笔者没有真正理解 DDD .读者也推荐所有有意学习 DDD 的人, 一定要以 Eric Evans 的原书为参考资料进行学习, 避免二手教程的曲解与误导.常见的误解: 领域驱动设计需要应用充血模型什么是领域既然是 "领域驱动设计" , 这6个字里面显然核心是 "领域" , 那第一个要搞清楚的问题就是"领域" 的定义是什么. 既然 Eric Evans 提出的概念, 必然要以原书的内容为准.Every software program relates to some activity or interest of its user. That subject area to which the user applies the program **is the domain of the software**. Some domains involve the physical world: The domain of an airline-booking program involves real people getting on real aircraft. Some domains are intangible: The domain of an account- ing program is money and finance. Software domains usually have lit- tle to do with computers, though there are exceptions: The domain of a source-code control system is software development itself.
所有软件程序都是用于执行用户的某项活动，或是满足用户的某种需求。这部分**用户使用软件的问题区域就是软件的领域**。一些领域涉及物质世界，例如，机票预订程序的领域中包括飞机乘客在内。有些领域则是无形的，例如，会计程序的金融领域。软件领域一般与计算机关系不大，当然也有例外，例如，源代码控制系统的领域就是软件开发本身
上面这段定义很长, 有点抽象, 但是相信每个读者都能理解, 这里的领域其实说的就是软件应用的 "业务领域".有趣的是, 这个概念粗一看很高级, 但是你仔细想一下, "业务" 驱动设计, 这不是一个常识吗..恐怕很难找出一个不被业务驱动的设计吧. 所以这个定义并不是领域驱动设计的核心所在. 我们继续顺着原书挖掘一下为了创建真正能为用户活动所用的软件，开发团队必须运用一整套与这些活动有关的知识体系。所需知识的广度可能令人望而生畏，庞大而复杂的信息也可能超乎想象。模型正是解决此类 信息超载问题的工具。模型这种知识形式对知识进行了选择性的简化和有意的结构化。适当的模型可以使人理解信息的意义，并专注于问题。
上面这段话清晰地解释了, 由于"业务领域" 的知识太过庞大, 在指导软件设计时, 我们需要对业务知识进行提炼和精简, 建立一个所谓的模型来指导设计开发. 而 DDD 的真正价值就是在描述该如何建立这个能够指导开发的 **"领域模型"** , Domain-Driven Design 其实是一种简称, 其真正想表达的是 Domain Model Driven Design, 即领域模型驱动设计领域模型 -- 业务知识的压缩表达领域模型是书中第二个重要的概念定义, 可以看一下原话.A domain model is not a particular diagram; it is the idea that the diagram is intended to convey. It is not just the knowledge in a domain expert’s head; **it is a rigorously organized and selective abstraction of that knowledge.** A diagram can represent and communicate a model, as can carefully written code, as can an English sentence.
领域模型并不是某个模型图，而是这种图所要传达的思想。它绝不单单是领域专家头脑中的知识，而**是对这类知识严格的组织且有选择的抽象**。图可以表示和传达一种模型，同样，精心书写的代码或文字也能达到同样的目的。
上见面这段话其实非常重要, 它强调了领域模型并不拘泥于形式, 重要的是它能够传递经过提炼后的业务知识的抽象与精炼. 它可以是一副图,也可以是一段代码, 甚至可以是一句话. - 这是一个很重要的观点, 但很可能许多 DDD 学习者并不会留意,乃至误解为领域模型一定需要是一个标准的建模方式例如 UML 绘制出的复杂方案理清上面这一点以后, 相信细心读者都会产生疑问, 这不还是玄学吗? 凡事一旦上升到思想高度, 必然千人千面. 但实际不是.领域模型这个概念强调的是业务知识的提炼与抽象, 可能有一些读者不明白业务知识有什么好抽象的, 主要原因是**很多程序员所开发的软件业务领域还不够复杂, 领域知识不够密集**, 相信看如下原书中的这个举例, 很多人立刻就能明白, 到底为什么要构建一个 "领域模型" 了领域建模举例- PCB印制电路板设计软件相信第一眼看到 PCB 这个概念, 很多程序员是懵逼的, 并不知道硬件电路设计软件要怎么设计, 至少我是懵逼的, 大学的硬件电路知识都快交回给老师了, 唯一残存的一些概念可以支撑自己不对这个 PCB 望而生畏.这个例子非常精妙地体现了领域知识对于软件设计的作用, 如果你对一个领域的知识完全不了解, 你根本不知道从何入手去设计这个软件. 而我们日常所熟悉的电子商城,支付, 社交类软件, 其领域概念非常生活化, 以至于似乎都感知不到领域知识的存在,每个程序员一上手都能展开设计Eric Evans 也没有硬件知识储备, 所以他需要先和 PCB 专家沟通学习. 这个过程我放几张原书截图, 读者就能直观感受到建模过程, 理解为什么需要领域建模首先经过多次沟通, 发现有一个术语 net 经常用到, 在这个领域, net 是一种导线,可以连接 PCB 任意数量的原件![img](https://pic2.zhimg.com/80/v2-e2718e5247348f4ade40920da475fe10_1440w.jpg?source=1940ef5c)
然后尝试画一个符合软件概念的图例来尝试表达与专家沟通![img](https://pic1.zhimg.com/80/v2-ee083ec7365eee569ba961395bafa4c8_1440w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-767d5caf5e7432d04d6c31d887077960_1440w.jpg?source=1940ef5c)
这个例子非常棒的一点在于, 它还恰好反映了术语存在冲突和混淆的情况, 例如 Component 在软硬件中都有其各自的含义, 这导致了沟通障碍, 通过与领域专家持续地讨论与学习后, 程序员可以与领域专家统一术语, 形成 **Ubiquitous Language**, 进一步细化精确模型![img](https://pic4.zhimg.com/80/v2-6d4e4d9f073d1e678f52532ede2bef0a_1440w.jpg?source=1940ef5c)
进一步讨论, 得知 PCB 专家期望通过软件来计算"电路跳数"![img](https://pic3.zhimg.com/80/v2-935d9db078ab5587847b906075e206c1_1440w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-e5edec229b229b4958211206558255ca_1440w.jpg?source=1940ef5c)
继续讨论细化模型![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1170' height='482'></svg>)![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1196' height='432'></svg>)
领域建模小结从上面这个过程中我们可以看出, 领域建模的核心其实是对业务知识的消化与提炼, 将其转化为可以被软件表达或者模拟的形式. 上面部分截图中的讨论, 其实程序员都会有很切身的体验, 这在与业务人员或者产品经理的沟通中其实很常见. 双方需要对一个概念达成统一 , 用于沟通业务规则如何去实现.这进一步反映了, 领域驱动设计其实不是什么高深的概念, 绝大多数开发者即使没有专门学习过领域驱动设计, 其实也不可避免地在应用着 DDD. 当然本文至此只是简单介绍了 Eric Evans 所描述的 DDD 的核心思想:领域建模通用语言构建更多具体的关于如何进行领域建模的细节, 也是 DDD 这本书的核心内容所在. 感兴趣的同学可以自行阅读原书, 笔者也会另开文章整理总结. 但是相比这些术层面的内容, 更为重要地是理解 DDD 所要解决的问题.也就是 DDD 被提出来的目的.否则, 甚至会产生领域驱动设计就是运用"充血模型" 这种错误的理解, 原书中其实都单独强调了, 充血模型和贫血模型都可以用于领域驱动设计.Then there are those aspects of the domain that are more clearly expressed as actions or operations, rather than as objects. Although it is a slight departure from object-oriented modeling tradition, it is often best to express these as SERVICES, rather than forcing responsi- bility for an operation onto some ENTITY or VALUE OBJECT.
领域中还有一些方面适合用动作或操作来表示，这比用对象表示更加清楚。这些方面最好用 SERVICE来表示，而不应把操作的责任强加到ENTITY或VALUE OBJECT上，尽管这样做稍微违背了 面向对象的建模传统。SERVICE是应客户端请求来完成某事。在软件的技术层中有很多SERVICE。 在领域中也可以使用SERVICE，当对软件要做的某项无状态的活动进行建模时，就可以将该活动 作为一项SERVICE。

[编辑于 03-28](https://www.zhihu.com/question/328870859/answer/1756417134)

赞同 107 条评论

分享

收藏喜欢收起

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![logo](https://pic3.zhimg.com/v2-b6979a32f5ef8d20aa071f980b81bf36_250x250.jpeg)PlayStation](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

广告



不感兴趣[知乎广告介绍](https://www.zhihu.com/promotion-intro)

[PS Store 假期限时优惠来袭！多款热门游戏2折起！PS Store假期限时优惠来了，包括 Little Nightmares II、Ghost of Tsushima、Persona 5 The Royal、Grand Theft Auto V 等多款热门游戏低至2折起！查看详情](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

[![QR Code of Downloading Zhihu App](https://static.zhihu.com/heifetz/assets/sidebar-download-qrcode.7caef4dd.png)下载知乎客户端与世界分享知识、经验和见解](http://zhi.hu/BDXoI)

[广告![广告](https://pic1.zhimg.com/v2-cb8d75ddaf94586fc2d232ebc336e058_540x450.jpeg)](https://www.zhihu.com/xen/market/ecom-page/1369976837221777408)



相关问题

[什么是ddd领域驱动架构，尽量说人话，回答要在50个字以内?](https://www.zhihu.com/question/448211945) 5 个回答

[如何评价领域驱动设计（DDD）？](https://www.zhihu.com/question/43554974) 11 个回答

[DDD中领域事件的好处是什么？](https://www.zhihu.com/question/31926656) 8 个回答

[哪些互联网公司在使用领域驱动设计？](https://www.zhihu.com/question/31687912) 11 个回答

[Java的mvc项目是否一定要遵循所有标准？](https://www.zhihu.com/question/22028524) 6 个回答

相关推荐

[![live](https://pic1.zhimg.com/90/v2-95844cb3a7d334a3e1311eb449f96458_250x0.jpg?source=31184dd1)Java 架构师指南王波103 人读过阅读](https://www.zhihu.com/pub/book/119630835)[![live](https://pic4.zhimg.com/90/v2-d807977676c1c28639cac3497e5c9ee2_250x0.jpg?source=31184dd1)Java EE 架构设计与开发实践15 人读过阅读](https://www.zhihu.com/pub/book/119638215)[![live](https://pic1.zhimg.com/90/v2-9aea51a2bce31f7ee5a337c8df583943_250x0.jpg?source=31184dd1)Java Web 开发系统项目教程48 人读过阅读](https://www.zhihu.com/pub/book/119632434)

[广告![广告](https://pic3.zhimg.com/v2-85be51948f3fb31a1c2d29bdc2b1fbda_540x450.jpeg)](https://www.zhihu.com/roundtable/lixiafood)



[刘看山](https://liukanshan.zhihu.com/)[知乎指南](https://www.zhihu.com/question/19581624)[知乎协议](https://www.zhihu.com/term/zhihu-terms)[知乎隐私保护指引](https://www.zhihu.com/term/privacy)
[应用](https://www.zhihu.com/app)[工作](https://app.mokahr.com/apply/zhihu)申请开通知乎机构号
[侵权举报](https://zhuanlan.zhihu.com/p/28852607)[网上有害信息举报专区](http://www.12377.cn/)
[京 ICP 证 110745 号](https://tsm.miit.gov.cn/dxxzsp/)
[京 ICP 备 13052560 号 - 1](https://beian.miit.gov.cn/)
[![img](https://pic3.zhimg.com/80/v2-d0289dc0a46fc5b15b3363ffa78cf6c7.png)京公网安备 11010802020088 号](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010802020088)
互联网药品信息服务资格证书
（京）- 非经营性 - 2017 - 0067违法和不良信息举报：010-82716601
[儿童色情信息举报专区](https://www.zhihu.com/term/child-jubao)
[证照中心](https://www.zhihu.com/certificates)[Investor Relations](https://ir.zhihu.com/)
[联系我们](https://www.zhihu.com/contact) © 2021 知乎

- [首页](https://www.zhihu.com/)
- [会员](https://www.zhihu.com/xen/vip-web)
- [发现](https://www.zhihu.com/explore)
- [等你来答](https://www.zhihu.com/question/waiting)



登录加入知乎

# 领域驱动设计(DDD)靠谱吗？

关注问题写回答

[IT 工程师](https://www.zhihu.com/topic/19550910)

[程序员](https://www.zhihu.com/topic/19552330)

[Java](https://www.zhihu.com/topic/19561132)

[IT 行业](https://www.zhihu.com/topic/19587634)

[领域驱动设计（DDD）](https://www.zhihu.com/topic/19826540)

# 领域驱动设计(DDD)靠谱吗？

怎么感觉在国内没什么人用啊显示全部 

关注者

**214**

被浏览

**91,542**

关注问题写回答

邀请回答

好问题 12

添加评论

分享



#### 29 个回答

默认排序

[![你微笑时很美](https://pic2.zhimg.com/v2-dc8cd91d459a8554c4b6930187f370b6_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/ni-wei-xiao-shi-hen-mei-7-7)

[你微笑时很美](https://www.zhihu.com/people/ni-wei-xiao-shi-hen-mei-7-7)

不只做一名编码者，更要做一名思考者

185 人赞同了该回答

由于公司领导的要求，所有的软件开发都要将DDD作为指导思想，并且要接受敏捷的思想；"迫不得已"下拜读了《实现领域驱动设计》这本书，将会在公司的内部系统上全面实践DDD。DDD把领域模型的重要性提高到了数据模型之上，在传统的MVC分层架构下。我们将项目结构分为Controller，Service，DAO 这三个主要的层，所有的业务逻辑都在Service中体现，而我们的实体类Entity却只是充当一个与数据库做ORM映射的数据容器而已，它并没有反映出模型的业务价值。所以又把这种模型称为“贫血模型”。“贫血模型”有什么坏处呢？在我们的代码 中将会到处看到各种的setter方法和各种各样的参数校验的代码，尤其是在Service层，但是这些代码它并没有反映出它的业务价值。这就是事务脚本的架构下，所呈现出来的弊端，这种模式下认为数据模型优先，所以会导致开发人员和产品经理在讨论问题的时候，完全是从两个角度在思考问题。开发人员听到需求后，脑袋里想的并不是如何反应出业务的价值，而是考虑的是数据库表怎么设计，字段该怎么加这些问题。所以DDD中提出了通用语言这么一个概念，并且基本将通用语言的概念贯穿于整个落地的过程。这样会大大的减少成员之间的沟通成本（前提是大家都从心里接受了DDD）。

为什么DDD难以落地呢？
第一，国内关于DDD的最佳实践还是太少了，除了知名的几个大厂以外很少看到有关于DDD的落地实践。这里附上美团的DDD实践，[美团领域驱动设计](https://link.zhihu.com/?target=https%3A//tech.meituan.com/2017/12/22/ddd-in-practice.html)；最佳实践太少意味着，我们可以参考的资料就少，承担的项目失败的风险就大。第二，DDD中出现了很多的概念和术语，比如 聚合根，值对象，六边形架构，CQRS(命令和查询职责分离)，事件驱动等等概念。很多人看到了这么多概念的时候，心中就开始打退堂鼓了。第三，DDD需要我们在领域建模花费很多的时间和精力，而且还可能导致付出和收益不成正比的情况。因为在界限上下文的划分上是非常考验架构师的业务水平。如果没有将业务模型很好的识别出来，那么可能很快模型就会在迭代的过程中腐败掉了。
上面就只是我自己对DDD的一些简单的理解，知识问题有不正确的地方，可以指正交流。（勿喷）

[编辑于 2020-05-29](https://www.zhihu.com/question/328870859/answer/1252486665)

赞同 18519 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![Johny Sinn](https://pic4.zhimg.com/v2-e1a027153bbad97530494eac3e21d0b4_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/linzihao-239)

[Johny Sinn](https://www.zhihu.com/people/linzihao-239)

一位见多识广、智慧超凡的IT人 & 超级猛男

47 人赞同了该回答

领域驱动设计DDD越来越受到重视，国内有很多团队在使用领域驱动设计DDD，但是每一个团队对DDD的理解可能不一样。如果领域的设计不能很好地指导开发工作，那么DDD的威力就发挥不出来了。我们接触到的“领域驱动”，很有可能是假的“领域驱动”，是别人理解消化后的领域驱动，即使是架构师或者项目经理，他们不一定真正理解领域驱动，**人们对领域驱动有很多误解，****比如“领域驱动只适合大型项目”，****“领域驱动会加大工作量，多加了一层Domain，太多转换”，****“领域驱动应该采用贫血模型”等等，****这些都是错误的认知。**Java项目的架构通常是采用MVC三层架构，MVC全名是Model View Controller，是模型Model－视图View－控制器Controller的缩写，但是MVC跟领域驱动的分层架构是不同。领域驱动的分层架构：**用户界面层/表示层Facade**用户界面层负责向用户显示信息和解释用户指令。这里指的用户可以是另一个计算机系统，不一定是使用用户界面的人。该层包含与其他应用系统（如Web服务、RMI接口、Web应用程序以及批处理前端）交互的接口与通信设施。它负责Request的解释、验证以及转换。另外，它也负责Response的序列化，如通过HTTP协议向web浏览器或web服务客户端传输HTML或XML，或远程Java客户端的DTO类和远程外观接口的序列化。该层的主要职责是与外部用户（包括Web服务、其他系统）交互，如接受用户的访问，展示必要的数据信息。用户界面层facade目录：（1）api存放Controller类，接受用户或者外部系统的访问，展示必要的数据信息。（2）handle存放GlobalExceptionHandler全局异常处理类或者全局拦截器。（3）model存放DTO（Request、Reponse）、Factory（Assembler）类，Factory负责数据传输对象DTO与领域对象Domain相互转换。应用层Application应用层定义了软件要完成的任务，并且指挥表达领域概念的对象来解决问题。该层所负责的工作对业务来说意义重大，也是与其他系统的应用层进行交互的必要通道。应用层要尽量简单。它不包含任务业务规则或知识，只是为了下一层的领域对象协助任务、分配工作。它没有反映业务情况的状态，但它可以具有反映用户或程序的某个任务的进展状态。应用层主要负责组织整个应用的流程，是面向用例设计的。该层非常适合处理事务，日志和安全等。相对于领域层，应用层应该是很薄的一层。它只是协调领域层对象执行实际的工作。应用层中主要组件是Service，因为主要职责是协调各组件工作，所以通常会与多个组件交互，如其他Service，Domain、Factory等等。应用层application目录：（1）service存放Service类，调用Domain执行命令操作，负责Domain的任务编排和分配工作。（2）external存放ExternalService类，负责与其他系统的应用层进行交互，通常是我们主动访问第三方服务。（3）model存放DTO（ExtRequest、ExtReponse）类。应用层application可以合并到领域层biz目录。领域层/模型层Biz领域层主要负责表达业务概念，业务状态信息和业务规则。Domain层是整个系统的核心层，几乎全部的业务逻辑会在该层实现。领域模型层主要包含以下的内容：实体(Entities):具有唯一标识的对象值对象(Value Objects): 无需唯一标识。领域服务(Domain): 与业务逻辑相关的，具有属性和行为的对象。聚合/聚合根(Aggregates & Aggregate Roots): 聚合是指一组具有内聚关系的相关对象的集合。工厂(Factories): 创建复杂对象，隐藏创建细节。仓储(Repository): 提供查找和持久化对象的方法。领域层biz目录：（1）domain存放Domain类，Domain负责业务逻辑，调用Repository对象来执行数据库操作。Domain没有直接访问数据库的代码，具体的数据库操作是通过调用Repository对象完成的。注意，除了CQRS模式外，Repository都应该是由Domain调用的，而不是由Service调用。（2）repository存放Repository类，调用Dao或者Mapper对象类执行数据库操作。（3）factory存放Factory类，负责Domain和实体Entity的转换。基础设施层Infrastructure基础设施层为上面各层提供通用的技术能力：为应用层传递消息，为领域层提供持久化机制，为用户界面层绘制屏幕组件。基础设施层以不同的方式支持所有三个层，促进层之间的通信。基础设施包括独立于我们的应用程序存在的一切：外部库，数据库引擎，应用程序服务器，消息后端等。基础设施层Infrastructure目录：（1）commons存放通用工具类Utils、常量类Constant、枚举类Enum、BizErrCode错误码类等等。（2）persistence存放Dao或者Mapper类，负责把持久化数据映射成实体Entity对象。注意，领域对象是具有属性和行为的对象，是有状态的，数据和行为都是可以重用的。在很多Java项目中，Service类是无状态的，而且一般是单例，业务逻辑直接写到Service类里面，这种方式本质上是面向过程的编程方式，丢失了面向对象的所有好处，重用性极差。应用“贫血模型”会把属于对象的数据和行为分离，领域对象不再是一个整体，破坏了领域模型。只有对领域驱动有足够的认知，工程师才会正确运用领域的理念去编程，告诉工程师业务逻辑是写在Domain不管用，他们可能还是会在Service里面写业务逻辑代码，创建很多private私有方法。**领域驱动开发的关注点在于领域模型，****所有的考虑都应该从领域的角度出发，重心放在业务。****领域模型必须能够精准地表达业务逻辑，****领域模型需要在开发过程中不断被完善，****并且能够指导工程师的开发工作。**[![img](https://pic1.zhimg.com/v2-60cff800a48c2e396ac1ff5c97c12e9e_hd.jpg?source=b555e01d)ZMI紫米PurPods 真无线降噪音乐蓝牙耳机 通话降噪超长京东¥ 199.00去购买](https://union-click.jd.com/jdc?e=jdext-1373816220936933377-0&p=JF8AARUDIgZlGFwSBxUPVxxdFzISBlQaWxQDEwBSG1slRk1fC0RrTEdXRhcQRQtaV1MJBABAHUBZCQVbFAMTB1QaWhIFEgdKQh5JXyJGAFoFYGYXcjVrC252b3wzHCJBYmZRWRdrFQsSBV0aWhUFETdVGloVBhcEVh1aJTISAmVNNRUDEwZUGlkWABI3VCtbEgETBVYZWRUCEA5UK1sdBiLR-4-Onb3Lt_DN8bvXn7eAkvDBvJQ3ZStYJVlHUxxeRxUAFAVcG1wWARMPVxxTFwAQAVMHWiUCEwZWH1kdBxQFOxhaFwobAzsZWhQDEAVQH10TMhI3VisFewNBB1cZUhxVfF0LTlxKRE5OOxhcFgQQAlwfaxcDEwVX)

[编辑于 05-07](https://www.zhihu.com/question/328870859/answer/1435876181)

赞同 479 条评论

分享

收藏喜欢收起

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![方丈的寺院](https://pic1.zhimg.com/v2-91a9607f8197a4413d31db77852d110d_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/fang-sheng-62-75)

[方丈的寺院](https://www.zhihu.com/people/fang-sheng-62-75)

程序员/马桶上思考人生

53 人赞同了该回答

DDD落地为什么这么难敏捷迭代，放弃建模现代互联网产研团队的构成一般是市场/运营、产品、UI交互、前端、后端、测试。这些角色的分工是将一个产品开发上线的各个过程拆分出来，然后每个过程专人负责，可以有效提高生产效率，这一套流程是标准的**流水线作业**。这样做的好处毋庸置疑，坏处也很明显，每个人只盯着自己那一块，而**忽略整体**了。再来看DDD,领域建模设计核心有两个统一语言(软件的开发人员/使用人员都使用同一套语言，即对某个概念，名词的认知是统一的）面向领域（以领域去思考问题，而不是模块）为了实现这两个核心，需要一个关键的角色，领域专家。他负责问题域，和问题解决域，他应该通晓研发的这个产品需要解决哪些问题，专业术语，关联关系。这个角色一般团队是没有配备的。最接近这个角色的就是产品了，但实际上产品并不是干这个活的。在我们团队落地过程中，有一段时间苦于没有领域专家，我想push产品成为领域专家，担当起这个角色。 最后不了了之，产品很配合，但是内驱力不强。为什么内驱力不强，因为给他带来的收益不够。前面已经提到敏捷迭代后，每个角色都是流水线上的螺丝钉，大家都只盯着自己这一块。对自己有利的去参与，和自己无关的不管。我们先看统一语言与面向领域的好处因为大家都使用统一的一套通用语言，所以沟通成本会大大减小，不会在讨论A的时候以为是B。对使用产品的用户有好处，他能在产品不断更新过程中，有一套统一流畅的体验。用户不用在每次软件更新时都要抱怨为什么之前的一个数据保存后没有用到了。面向领域去开发产品有助于我们深入分析产品的内在逻辑，专注于解决当前产品的核心问题，而不是冗余的做很多功能模块，或者几个用户/运营反馈的问题就去更改产品逻辑，完了上线后用户不用，你还在那边骂用户朝三暮四，乱提需求。这些好处粗看一下，其实对产品研发的各个角色都有意义。但细看一下呢，沟通成本大大减小，对于运营，产品，UI交互没啥问题。一个问题理解的不一致，组织个会议，大家好好聊聊就行了。用户体验一致对产研团队有啥好处呢，反正用户骂的不是我，是客户和运营。深入分析产品的内在逻辑有啥用呢？一款产品的成功有很多因素，主要靠上面，我只是一个小兵，我管不了那么多。有空我多研究研究我的专业领域，多去看几篇面试文章。产品黄了，我好跳槽。因为本人是后端研发，所以这里不对其他角色过多展开。只想对研发说，你跳槽换个公司就好了吗？还是crud boy。还是重复着写着很多冗余的功能，冗余的代码。需求方让你写什么，你就写什么，最后在一天天的加班中丧失了对代码的兴趣，没有了梦想。 我们都知道改变别人很难，所以先从改变自己开始，先让自己变优秀了，才能影响他人框架易学，思想难学如果抛开其他角色，单从研发角度考虑DDD呢。开发进行领域建模，然后遵从康威定律，将软件架构设计映射到业务模型中。（虽然这个领域，开发可能识别的不对，暂且忽略这个问题）康威（梅尔·康威）定律 任何组织在设计一套系统（广义概念上的系统）时，所交付的设计方案在结构上 都与该组织的沟通结构保持一致。
纯研发实施DDD,为什么也这么难呢？没有标准DDD是一套思想，一套领域建模设计，一套在特定上下文环境中使用的。所以在1千个团队中实行DDD,可能有1千套不同的方案。一个实行DDD多年的人，换了一个公司，换了一个团队，把他原有的那套带过去，推行下去，一般都不适用。所以DDD的学习和实践不像学习一个函数，API，框架那样有直接的反馈效果，他需要结合团队的实际情况去实行，才能达到效果。期待DDD解决所有的问题程序员都是很实际的，没有好处的东西是不会去做的。你必须能够有效的帮助他提升，他才会去接受。 比如当初有团队成员提出来，我们实行了这一套后，是不是不用加班了，或者加班时间可以减小。
有测试提出实行这一套后，bug率能降低多少。
研发需要一个可以量化的效果，抱歉DDD做不到。没有哪个团队实行了DDD后，解决了软件开发的所有问题。关于这一点，可以读一下[驱动方法不能改变任何事情](https://link.zhihu.com/?target=https%3A//www.infoq.cn/article/star-driven-approaches)

[发布于 2019-06-22](https://www.zhihu.com/question/328870859/answer/723586073)

赞同 536 条评论

分享

收藏喜欢收起

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![logo](https://pic3.zhimg.com/v2-b6979a32f5ef8d20aa071f980b81bf36_250x250.jpeg)PlayStation](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

广告



不感兴趣[知乎广告介绍](https://www.zhihu.com/promotion-intro)

[PS Store 假期限时优惠来袭！多款热门游戏2折起！PS Store假期限时优惠来了，包括 Little Nightmares II、Ghost of Tsushima、Persona 5 The Royal、Grand Theft Auto V 等多款热门游戏低至2折起！查看详情](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

[![曾著](https://pic1.zhimg.com/v2-37145f435b6b361cfe9df42bd15698d8_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/joeaniu)

[曾著](https://www.zhihu.com/people/joeaniu)

互联网创业者，关注高性能并发系统、移动互联网、创业、敏捷开发

20 人赞同了该回答

简单回答一下。 如果你说的DDD是《DDD》和《iDDD》书中写的模式和实践，例如CQRS，那的确用的人不多。 但如果你说的DDD是Bounded Context，Domain这些思路，那几乎所有软件都在用，可能只是没用DDD这套概念去阐述而已。

[发布于 2019-06-12](https://www.zhihu.com/question/328870859/answer/712818348)

赞同 20添加评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![悬壶醉世](https://pic1.zhimg.com/v2-f6170b6e6274207186539d4ec6104f5c_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/deng-hang-hui-92)

[悬壶醉世](https://www.zhihu.com/people/deng-hang-hui-92)

每次答题瞎逛的时候问问自己今天工作学习有没有尽力。

4 人赞同了该回答

靠谱。。。。。DDD是面向架构思想的书。适合对于项目有统筹能力的人。我认为书中有几个要点1，统一沟通语言。2，领域专家。3，一个符合业务现状的模型，持续管理模型。
实际上这些东西在小公司的实践上会有所难度。难度在于：1，我们给沟通模型，实际走到了现在的话我们前端，后台，产品甚至运维使用的模型都千差万别。文档也是如此。因此我们很难去统一沟通语言。能做到的仅仅只是，每个工种的模型统一。2，我们的领域专家是产品经理，实现的模型与开发所需不符。需求改来改去，模型也没维护……但是真的安排一个中间人负责沟通产品，维护模型，说实话有点难。虽然可以实现，但大概不会有人这么做。3，模型的管理会有很多问题。比如临时的变更，比如业务的扩大，个人能力维护这个模型也挺麻烦的。所以需要培养同事对模型进行共同维护。。。。分工，版本管理又是一些麻烦事。。所以具体实现上也需要各自想办法。。。
用是有人在用的。。。。。微服务本来也是DDD的一个实现。大型公司也有成熟案例。
可是实现上来说，很少有人敢说自己的实现很好。DDD作为一系列思想也没给出具体的指导，大多都实现都与业务相关，所以不好吹。。。。
但是实现的过程中，你会发现DDD虽然很麻烦但是真的解决了很多问题，即使无法量化。

[发布于 2020-07-05](https://www.zhihu.com/question/328870859/answer/1319414023)

赞同 4添加评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![Bird Frank](https://pic1.zhimg.com/16137c755_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/bird-frank)

[Bird Frank](https://www.zhihu.com/people/bird-frank)

SoftwareDeveloper

3 人赞同了该回答

DDD是靠谱的，但在国内用的人确实也不多。这一个可以看知乎上与DDD相关的问题和文章点赞数、回答量和评论数都不多，另一个可以看很多公司对架构师、项目经理等的招聘要求也还没有出现DDD的字样。DDD在国内使用不多的原因可能有这样几个，这几个原因都和DDD本身是否靠谱无关。无知者无谓。DDD是用于分析、解决复杂业务领域的方法。很多项目的开发者不能在初期对项目所涉业务领域和需求的复杂度做出正确的评估，并往往低估业务复杂性。这导致他们不会去寻求能帮助理解、分析复杂业务系统的方法。现在国内的项目也可能确实不是很复杂，但是表面看来只是一些表单加一些CRUD操作的应用却往往也会隐藏一两个比较复杂的业务逻辑，这些业务逻辑还往往与核心业务相关。等到开发团队跌进这个坑里的时候往往已经太晚了。项目开发中的短视行为。DDD的很多方法、原则的目的是为了提升软件系统的可维护性、可扩展性、应对业务需求变更的能力。在很多外包项目中，甲方的关注点在降低成本（特别是业务部门和采购部门决策权分开的情况），而乙方的关注点则是在满足的交付的前提下利润最大化。我更是见过要求软件系统的开发商和维护商不能是同一家企业的奇葩规定。这种情况下，开发者（一方）自然不会把很多精力去提升“可维护性、可扩展性、应变变更”这些长远见效的、比较“需”的事情上。错误的软件质量观。Uncle Bob 说过，应该用软件开发所耗费的精力来衡量软件质量，这个精力包括了软件全生命周期中所耗费的开发、维护等工作的精力。但是现在很多甲方和软件公司对软件质量的评估依然主要看缺陷率，他们也没有能力对可维护性、可扩展性这些指标进行评估，这也导致上面第2点说的“短视行为”。程序员的能力。DDD是默认了采用面向对象的分析方法与思维方式的。但是很多程序员，包括很多所谓架构师、Team Leader，对OOP都没有很好掌握和运用，更不用说面向对象的分析设计了。

[发布于 03-02](https://www.zhihu.com/question/328870859/answer/1757729846)

赞同 31 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![古明地觉](https://pic1.zhimg.com/v2-deebf49946a10c35dd08a7ff0855ed65_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/gu-ming-di-jue-89-16)

[古明地觉](https://www.zhihu.com/people/gu-ming-di-jue-89-16)

养猫，养鸟，喜欢在网上冲浪

8 人赞同了该回答

DDD就是玄学在没有领域专家的情况下，每个人设计的领域对象肯定大有不同，一个团队里十个开发，能设计出十种不通的领域划分，最后迭代几个版本，代码根本没法维护互联网开发新技术新理念层出不穷，每段时间都会出来一个奇葩东西，最后被淘汰现在DDD就是，随便在知乎上抓个人就能给你长篇大论一套DDD原理，让他写个Demo放Github上，都够呛能写出来我们团队在设计一个非常复杂系统的时候用过DDD，而且一直在用，坑很多，最后还是得看本质1.这个技术用了能提高员工多少效率2.这个技术带来了什么好处3.这个技术维护成本和设备成本是不是增加了用了DDD我花了多久时间构思领域模型，花了多久时间和小伙伴讨论领域模型，花了多少时间打代码，花了多少时间改bug，如果用了DDD，你办公效率反倒下降了，团队间讨论更频繁了，那我强烈建议放弃这个技术，没必要为了追赶潮流而学习技术，计算机技术永远是避繁就简，互联网永远是敏捷开发。DDD这个概念国内已经炒了几年了，好不好用试试就知道了。

[编辑于 03-16](https://www.zhihu.com/question/328870859/answer/1713027163)

赞同 86 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![只喝可乐的猫](https://pic4.zhimg.com/v2-b5690808bebbe1583ee0e6399d410816_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/zheng-zhe-ming-33)

[只喝可乐的猫](https://www.zhihu.com/people/zheng-zhe-ming-33)

Ones and Zeros

4 人赞同了该回答

哈哈，原因是 ddd vs transaction script 并没有说 ddd 一定好，而领域模型设计和具体业务关联紧密，而ddd在社区没有被广泛接受的最佳实践案例，真正能搞懂用明白ddd的人并不多，所以工程领域当然选择简单明了的更符合直觉的transaction script 编程模型啦，更不要说大家手头干的绝大多数活都是搬砖crud，哪有什么复杂逻辑。不过，ddd 中的一些设计模式已经被发扬光大广泛应用了，比如 SpringData 中的 Repository，用的人不要太多。

[编辑于 2019-06-29](https://www.zhihu.com/question/328870859/answer/713103450)

赞同 4添加评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![kyssky](https://pic1.zhimg.com/v2-62ac5b08d6ae4e3ec8b55145e5e25e65_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/kyssky_p)

[kyssky](https://www.zhihu.com/people/kyssky_p)

彡(￣_￣；)彡风中凌乱

28 人赞同了该回答

DDD这玩意就是旧酒装新壶 , 没有什么创新的东西要是真的牛逼就不会这么就都没有火了 , 微服务也救不了他只相信一句真理 , 软件工程本身的复杂度是没有办法解决的 , 我们只能尽可能的将代码的运行逻辑梳理清楚 , 尽可能的将关键逻辑分离 ,方便以后修改

[发布于 2020-07-09](https://www.zhihu.com/question/328870859/answer/1328408231)

赞同 282 条评论

分享

收藏喜欢

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![萧萧](https://pic4.zhimg.com/v2-f0607f488dcb7cf3bac107335c5f7683_xs.jpg?source=1940ef5c)](https://www.zhihu.com/people/xiao-xiao-99-94)

[萧萧](https://www.zhihu.com/people/xiao-xiao-99-94)

10 人赞同了该回答

﻿# 参考资料 - Domain-Driven Design: Tackling Complexity in the Heart of Software前言领域驱动设计是一个传播非常广泛的名词, 笔者对这个概念听闻很早,但是一直也没有提起太大的兴趣去学习, 愿因很简单, 因为它属于一种工程设计思想, 解决的问题不够标准具象, 其内在价值存在争议.但无奈工作中恰恰就有一个公司内部自建的 "中台化解决方案" 的框架, 号称运用了 DDD 的思想, 虽然笔者不太认同, 但是也要先学习才能有发言权 。虽然市面上充斥着各种博文,教程都说 Eric Evans 的原书偏概念, 缺乏实现细节. 所谓”取法乎上，得其中“， ”取法乎中， 得其下“ , 为了真正弄清楚DDD 的核心价值到底在哪, 笔者选择了阅读 Eric Evans 的原书, 真正阅读完毕后发现: - **事实恰恰相反, DDD 原书非常接地气, 而那些所谓解读 DDD 的博客或者教程才是将 DDD 概念抽象化的罪魁祸首**![img](https://pic1.zhimg.com/80/v2-41eb39d9541ce79a7ffab157e1ca502f_1440w.jpg?source=1940ef5c)
所以本篇文章旨在将 DDD 原书中的一些核心思想简单地传达出来. 希望可以帮助一些感兴趣的初学者. 笔者会尽可能引用书中的原话来传达自己的观点, 涉及到概念定义的, 会同时引用英文. 避免一些 DDD 的信徒口诛笔伐, 攻击笔者没有真正理解 DDD .读者也推荐所有有意学习 DDD 的人, 一定要以 Eric Evans 的原书为参考资料进行学习, 避免二手教程的曲解与误导.常见的误解: 领域驱动设计需要应用充血模型什么是领域既然是 "领域驱动设计" , 这6个字里面显然核心是 "领域" , 那第一个要搞清楚的问题就是"领域" 的定义是什么. 既然 Eric Evans 提出的概念, 必然要以原书的内容为准.Every software program relates to some activity or interest of its user. That subject area to which the user applies the program **is the domain of the software**. Some domains involve the physical world: The domain of an airline-booking program involves real people getting on real aircraft. Some domains are intangible: The domain of an account- ing program is money and finance. Software domains usually have lit- tle to do with computers, though there are exceptions: The domain of a source-code control system is software development itself.
所有软件程序都是用于执行用户的某项活动，或是满足用户的某种需求。这部分**用户使用软件的问题区域就是软件的领域**。一些领域涉及物质世界，例如，机票预订程序的领域中包括飞机乘客在内。有些领域则是无形的，例如，会计程序的金融领域。软件领域一般与计算机关系不大，当然也有例外，例如，源代码控制系统的领域就是软件开发本身
上面这段定义很长, 有点抽象, 但是相信每个读者都能理解, 这里的领域其实说的就是软件应用的 "业务领域".有趣的是, 这个概念粗一看很高级, 但是你仔细想一下, "业务" 驱动设计, 这不是一个常识吗..恐怕很难找出一个不被业务驱动的设计吧. 所以这个定义并不是领域驱动设计的核心所在. 我们继续顺着原书挖掘一下为了创建真正能为用户活动所用的软件，开发团队必须运用一整套与这些活动有关的知识体系。所需知识的广度可能令人望而生畏，庞大而复杂的信息也可能超乎想象。模型正是解决此类 信息超载问题的工具。模型这种知识形式对知识进行了选择性的简化和有意的结构化。适当的模型可以使人理解信息的意义，并专注于问题。
上面这段话清晰地解释了, 由于"业务领域" 的知识太过庞大, 在指导软件设计时, 我们需要对业务知识进行提炼和精简, 建立一个所谓的模型来指导设计开发. 而 DDD 的真正价值就是在描述该如何建立这个能够指导开发的 **"领域模型"** , Domain-Driven Design 其实是一种简称, 其真正想表达的是 Domain Model Driven Design, 即领域模型驱动设计领域模型 -- 业务知识的压缩表达领域模型是书中第二个重要的概念定义, 可以看一下原话.A domain model is not a particular diagram; it is the idea that the diagram is intended to convey. It is not just the knowledge in a domain expert’s head; **it is a rigorously organized and selective abstraction of that knowledge.** A diagram can represent and communicate a model, as can carefully written code, as can an English sentence.
领域模型并不是某个模型图，而是这种图所要传达的思想。它绝不单单是领域专家头脑中的知识，而**是对这类知识严格的组织且有选择的抽象**。图可以表示和传达一种模型，同样，精心书写的代码或文字也能达到同样的目的。
上见面这段话其实非常重要, 它强调了领域模型并不拘泥于形式, 重要的是它能够传递经过提炼后的业务知识的抽象与精炼. 它可以是一副图,也可以是一段代码, 甚至可以是一句话. - 这是一个很重要的观点, 但很可能许多 DDD 学习者并不会留意,乃至误解为领域模型一定需要是一个标准的建模方式例如 UML 绘制出的复杂方案理清上面这一点以后, 相信细心读者都会产生疑问, 这不还是玄学吗? 凡事一旦上升到思想高度, 必然千人千面. 但实际不是.领域模型这个概念强调的是业务知识的提炼与抽象, 可能有一些读者不明白业务知识有什么好抽象的, 主要原因是**很多程序员所开发的软件业务领域还不够复杂, 领域知识不够密集**, 相信看如下原书中的这个举例, 很多人立刻就能明白, 到底为什么要构建一个 "领域模型" 了领域建模举例- PCB印制电路板设计软件相信第一眼看到 PCB 这个概念, 很多程序员是懵逼的, 并不知道硬件电路设计软件要怎么设计, 至少我是懵逼的, 大学的硬件电路知识都快交回给老师了, 唯一残存的一些概念可以支撑自己不对这个 PCB 望而生畏.这个例子非常精妙地体现了领域知识对于软件设计的作用, 如果你对一个领域的知识完全不了解, 你根本不知道从何入手去设计这个软件. 而我们日常所熟悉的电子商城,支付, 社交类软件, 其领域概念非常生活化, 以至于似乎都感知不到领域知识的存在,每个程序员一上手都能展开设计Eric Evans 也没有硬件知识储备, 所以他需要先和 PCB 专家沟通学习. 这个过程我放几张原书截图, 读者就能直观感受到建模过程, 理解为什么需要领域建模首先经过多次沟通, 发现有一个术语 net 经常用到, 在这个领域, net 是一种导线,可以连接 PCB 任意数量的原件![img](https://pic2.zhimg.com/80/v2-e2718e5247348f4ade40920da475fe10_1440w.jpg?source=1940ef5c)
然后尝试画一个符合软件概念的图例来尝试表达与专家沟通![img](https://pic1.zhimg.com/80/v2-ee083ec7365eee569ba961395bafa4c8_1440w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-767d5caf5e7432d04d6c31d887077960_1440w.jpg?source=1940ef5c)
这个例子非常棒的一点在于, 它还恰好反映了术语存在冲突和混淆的情况, 例如 Component 在软硬件中都有其各自的含义, 这导致了沟通障碍, 通过与领域专家持续地讨论与学习后, 程序员可以与领域专家统一术语, 形成 **Ubiquitous Language**, 进一步细化精确模型![img](https://pic4.zhimg.com/80/v2-6d4e4d9f073d1e678f52532ede2bef0a_1440w.jpg?source=1940ef5c)
进一步讨论, 得知 PCB 专家期望通过软件来计算"电路跳数"![img](https://pic3.zhimg.com/80/v2-935d9db078ab5587847b906075e206c1_1440w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-e5edec229b229b4958211206558255ca_1440w.jpg?source=1940ef5c)
继续讨论细化模型![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1170' height='482'></svg>)![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1196' height='432'></svg>)
领域建模小结从上面这个过程中我们可以看出, 领域建模的核心其实是对业务知识的消化与提炼, 将其转化为可以被软件表达或者模拟的形式. 上面部分截图中的讨论, 其实程序员都会有很切身的体验, 这在与业务人员或者产品经理的沟通中其实很常见. 双方需要对一个概念达成统一 , 用于沟通业务规则如何去实现.这进一步反映了, 领域驱动设计其实不是什么高深的概念, 绝大多数开发者即使没有专门学习过领域驱动设计, 其实也不可避免地在应用着 DDD. 当然本文至此只是简单介绍了 Eric Evans 所描述的 DDD 的核心思想:领域建模通用语言构建更多具体的关于如何进行领域建模的细节, 也是 DDD 这本书的核心内容所在. 感兴趣的同学可以自行阅读原书, 笔者也会另开文章整理总结. 但是相比这些术层面的内容, 更为重要地是理解 DDD 所要解决的问题.也就是 DDD 被提出来的目的.否则, 甚至会产生领域驱动设计就是运用"充血模型" 这种错误的理解, 原书中其实都单独强调了, 充血模型和贫血模型都可以用于领域驱动设计.Then there are those aspects of the domain that are more clearly expressed as actions or operations, rather than as objects. Although it is a slight departure from object-oriented modeling tradition, it is often best to express these as SERVICES, rather than forcing responsi- bility for an operation onto some ENTITY or VALUE OBJECT.
领域中还有一些方面适合用动作或操作来表示，这比用对象表示更加清楚。这些方面最好用 SERVICE来表示，而不应把操作的责任强加到ENTITY或VALUE OBJECT上，尽管这样做稍微违背了 面向对象的建模传统。SERVICE是应客户端请求来完成某事。在软件的技术层中有很多SERVICE。 在领域中也可以使用SERVICE，当对软件要做的某项无状态的活动进行建模时，就可以将该活动 作为一项SERVICE。

[编辑于 03-28](https://www.zhihu.com/question/328870859/answer/1756417134)

赞同 107 条评论

分享

收藏喜欢收起

继续浏览内容

![img](https://pic4.zhimg.com/80/v2-88158afcff1e7f4b8b00a1ba81171b61_720w.png)

知乎

发现更大的世界

打开

![img](https://picb.zhimg.com/80/v2-a448b133c0201b59631ccfa93cb650f3_1440w.png)

Chrome

继续

[![logo](https://pic3.zhimg.com/v2-b6979a32f5ef8d20aa071f980b81bf36_250x250.jpeg)PlayStation](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

广告



不感兴趣[知乎广告介绍](https://www.zhihu.com/promotion-intro)

[PS Store 假期限时优惠来袭！多款热门游戏2折起！PS Store假期限时优惠来了，包括 Little Nightmares II、Ghost of Tsushima、Persona 5 The Royal、Grand Theft Auto V 等多款热门游戏低至2折起！查看详情](https://e.cn.miaozhen.com/r/k=2235684&p=7sBdA&dx=__IPDX__&rt=2&pro=s&ns=183.129.243.163&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&mo=3&m0=__OPENUDID__&m0a=__DUID__&m1=__ANDROIDID1__&m1a=__ANDROIDID__&m2=__IMEI__&m4=__AAID__&m5=__IDFA__&m6=__MAC1__&m6a=__MAC__&m11=__OAID__&m14=__CAID__&m5a=__IDFV__&mn=__ANAME__&vv=1&vo=38096a7b9&vr=2&o=https%3A%2F%2Fstore.playstation.com%2Fzh-hans-hk%2Fcategory%2Fc7b6ba79-ea10-4a72-a8a1-1e74c76eb086%3Femcid%3Ddi-pl-406558)

[![QR Code of Downloading Zhihu App](https://static.zhihu.com/heifetz/assets/sidebar-download-qrcode.7caef4dd.png)下载知乎客户端与世界分享知识、经验和见解](http://zhi.hu/BDXoI)

[广告![广告](https://pic1.zhimg.com/v2-cb8d75ddaf94586fc2d232ebc336e058_540x450.jpeg)](https://www.zhihu.com/xen/market/ecom-page/1369976837221777408)



相关问题

[什么是ddd领域驱动架构，尽量说人话，回答要在50个字以内?](https://www.zhihu.com/question/448211945) 5 个回答

[如何评价领域驱动设计（DDD）？](https://www.zhihu.com/question/43554974) 11 个回答

[DDD中领域事件的好处是什么？](https://www.zhihu.com/question/31926656) 8 个回答

[哪些互联网公司在使用领域驱动设计？](https://www.zhihu.com/question/31687912) 11 个回答

[Java的mvc项目是否一定要遵循所有标准？](https://www.zhihu.com/question/22028524) 6 个回答

相关推荐

[![live](https://pic1.zhimg.com/90/v2-95844cb3a7d334a3e1311eb449f96458_250x0.jpg?source=31184dd1)Java 架构师指南王波103 人读过阅读](https://www.zhihu.com/pub/book/119630835)[![live](https://pic4.zhimg.com/90/v2-d807977676c1c28639cac3497e5c9ee2_250x0.jpg?source=31184dd1)Java EE 架构设计与开发实践15 人读过阅读](https://www.zhihu.com/pub/book/119638215)[![live](https://pic1.zhimg.com/90/v2-9aea51a2bce31f7ee5a337c8df583943_250x0.jpg?source=31184dd1)Java Web 开发系统项目教程48 人读过阅读](https://www.zhihu.com/pub/book/119632434)

[广告![广告](https://pic3.zhimg.com/v2-85be51948f3fb31a1c2d29bdc2b1fbda_540x450.jpeg)](https://www.zhihu.com/roundtable/lixiafood)



[刘看山](https://liukanshan.zhihu.com/)[知乎指南](https://www.zhihu.com/question/19581624)[知乎协议](https://www.zhihu.com/term/zhihu-terms)[知乎隐私保护指引](https://www.zhihu.com/term/privacy)
[应用](https://www.zhihu.com/app)[工作](https://app.mokahr.com/apply/zhihu)申请开通知乎机构号
[侵权举报](https://zhuanlan.zhihu.com/p/28852607)[网上有害信息举报专区](http://www.12377.cn/)
[京 ICP 证 110745 号](https://tsm.miit.gov.cn/dxxzsp/)
[京 ICP 备 13052560 号 - 1](https://beian.miit.gov.cn/)
[![img](https://pic3.zhimg.com/80/v2-d0289dc0a46fc5b15b3363ffa78cf6c7.png)京公网安备 11010802020088 号](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010802020088)
互联网药品信息服务资格证书
（京）- 非经营性 - 2017 - 0067违法和不良信息举报：010-82716601
[儿童色情信息举报专区](https://www.zhihu.com/term/child-jubao)
[证照中心](https://www.zhihu.com/certificates)[Investor Relations](https://ir.zhihu.com/)
[联系我们](https://www.zhihu.com/contact) © 2021 知乎

- [首页](https://www.oschina.net/)
- [资讯](https://www.oschina.net/news)
- [专区](https://www.oschina.net/groups)
- [问答](https://www.oschina.net/question)
- [活动](https://www.oschina.net/event)
- [软件库](https://www.oschina.net/project)
- [发现](https://www.oschina.net/explore)
- [博客](https://www.oschina.net/blog)
- [Gitee](https://gitee.com/?utm_source=oschina&utm_medium=link-index&utm_campaign=home)

[![OSCHINA](https://static.oschina.net/new-osc/img/logo_new.svg)](https://www.oschina.net/)

- [首页](https://www.oschina.net/)
- [资讯](https://www.oschina.net/news)
- [专区](https://www.oschina.net/groups)
- [问答](https://www.oschina.net/question)
- [活动](https://www.oschina.net/event)
- [软件库](https://www.oschina.net/project)
- [发现](https://www.oschina.net/explore)
- [博客](https://www.oschina.net/blog)
- [Gitee](https://gitee.com/?utm_source=oschina&utm_medium=link-index&utm_campaign=home)

- 

- 

- ![村长杨京京](https://oscimg.oschina.net/oscnet/up-cf904f0d61ea0fcf18b586fd616ec1a4.jpeg!/both/50x50?t=1459925359000)

- 

[开源博客](https://www.oschina.net/blog)

[写博客](https://www.oschina.net/home/go?page=blog%2Fwrite)







































[malaoko的个人空间](https://my.oschina.net/u/379999)*/*[工作日志](https://my.oschina.net/u/379999?tab=newest&catalogId=1287543)*/*

正文

# [领域驱动设计DDD实战进阶第一波(九)：订单上下文POCO模型](https://my.oschina.net/u/379999/blog/1923119)

原创

[malaoko](https://my.oschina.net/u/379999)

[工作日志](https://my.oschina.net/u/379999?tab=newest&catalogId=1287543)

2018/08/06 12:09

阅读数 3.9K

[![img](https://static.oschina.net/uploads/img/202008/31180726_2pwW.png)](https://www.oschina.net/group/backend)

本文被收录于专区

[服务端](https://www.oschina.net/group/backend)

[进入专区参与更多专题讨论 ](https://www.oschina.net/group/backend)



# [DDD实战进阶第一波(九)：开发一般业务的大健康行业直销系统（订单上下文POCO模型）](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.cnblogs.com%2Fmalaoko%2Fp%2F9092967.html)

在本系列前面的文章中，我们主要讨论了产品上下文与经销商上下文相关的实现，大家对DDD的方法与架构已经有了初步的了解。

但是在这两个界限上下文中，业务逻辑很简单，也没有用到更多的值对象的内容。从这篇文章开始，我们来讲讲订单界限上下文实现的内容，

里面的业务逻辑相对复杂一些，而且有大量值对象的引入来进行逻辑的处理。

订单上下文的需求主要是生成相应的订单项，每个订单项中有相关的订单产品和购买数量并生成订单项总额、订单项总PV，同时订单项总额

和订单项总PV会累加到订单总额和订单总PV中，同时会根据订单总额扣减当前经销商的电子币，也会根据购买产品的PV，累加当前经销商的PV值。

**1.订单界限上下文的领域模型：**

![img](https://images2018.cnblogs.com/blog/56104/201805/56104-20180526145029731-518421733.png)

从上图的领域模型中，大家可以看出订单是聚合根，订单明细是聚合的实体；订单聚合根总有总价、总PV、收获地址三个值对象，订单明细实体有

明细总价、明细总PV、产品信息三个值对象。

**2.明细总价值对象：**

```
    public partial class OrderItemTotalPrice
    {
        public decimal SubTotalPrice { get; set; }
    }
```

**3.明细总PV值对象：**

```
 public partial class OrderItemTotalPV
    {
        public decimal SubTotalPV { get; set; }
    }
```

**4.产品信息值对象：**

```
 public partial class ProductSKUs
    {
        public string ProductSPUName { get; set; }
        public decimal ProductPrice { get; set; }
        public decimal ProductPV { get; set; }
        public Guid ProductSKUId { get; set; }
    }
```

 

**5.订单明细实体：**

```
 public partial class OrderItem : IEntity
    {
        public string Code { get; set; }
        [Key]
        public Guid Id { get ; set ; }

        public OrderItemTotalPrice OrderItemTotalPrice { get; set; }
        public OrderItemTotalPV OrderItemTotalPV { get; set; }
        public ProductSKUs ProductSKUs { get; set; }

        public int Count { get; set; }

    }
```

 

订单明细实体引入了OrderItemTotalPrice、OrderItemTotalPV、ProductSKUs三个值对象，同时具有自己的Code与Count两个属性。

**6.订单总价对象：**

```
 public partial class OrderTotalPrice
    {
        public decimal TotalPrice { get; set; }
    }
```

**7.订单总PV值对象：**

```
 public partial class OrderTotalPV
    {
        public decimal TotalPV { get; set; }
    }
```

**8.订单收货地址值对象：**

```
 public partial class OrderStreet
    {
       //省
        public string Privince { get; set; }
       //市
        public string City { get; set; }
       //区(县)
        public string Zero { get; set; }
       //街道地址
        public string Street { get; set; }
    }
```

 

**9.订单聚合根：**

```
 public partial class Orders : IAggregationRoot
    {
        public string Code { get; set ; }
        [Key]
        public Guid Id { get ; set; }

        public OrderStreet OrderStreet { get; set; }
        public OrderTotalPV OrderTotalPV { get; set; }
        public OrderTotalPrice OrderTotalPrice { get; set; }
        public DateTime OrderDateTime { get; set; }
        public Guid OrderDealerId { get; set; }
        public List<OrderItem> OrderItems { get; set; }
        public string Telephone { get; set; }
    }
```

 

订单聚合根引入了OrderStreet、OrderTotalPV、OrderTotalPrice三个值对象；Code、DateTime（下单时间）、Telephone（联系电话）、OrderItems(订单项实体集)等几个属性。

**10.生成数据库表：**

根据前面文章的说明，我们可以依据上述POCO模型生成对应的数据库表，要注意的是，OrderItems可以自动识别为Orders的关联表，其他几个值对象我们要考虑是否是生成

单独的表还是作为相关实体或聚合根的表的列存在，一般情况下，我们是将这些值对象作为相关聚合根或实体表的列存在的。EF Core无法自动处理这些值对象如何存储到数据库中，

我们需要手工指定：

```
 public class OrderEFCoreContext:DbContext,IOrderContext
    {
        public DbSet<Orders> Order { get; set; }
        public DbSet<OrderItem> OrderItem { get; set; }
        protected override void OnConfiguring(DbContextOptionsBuilder optionBuilder)
        {           

            optionBuilder.UseSqlServer("数据库连接字符串");
           
        }
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Orders>().OwnsOne(p => p.OrderStreet);
            modelBuilder.Entity<Orders>().OwnsOne(p => p.OrderTotalPrice);
            modelBuilder.Entity<Orders>().OwnsOne(p => p.OrderTotalPV);

            modelBuilder.Entity<OrderItem>().OwnsOne(p => p.OrderItemTotalPrice);
            modelBuilder.Entity<OrderItem>().OwnsOne(p => p.OrderItemTotalPV);
            modelBuilder.Entity<OrderItem>().OwnsOne(p => p.ProductSKUs);
        }
```

 

从上面代码可以看出，在OnModelCreating时，可以指定6个值对象包含在对应的聚合根和实体相关的表中。

[Poco](https://www.oschina.net/p/poco)

© 著作权归作者所有

举报

打赏

0 赞

0 收藏

分享

### 作者的其它热门文章

[领域驱动设计DDD实战进阶第一波(十)：订单上下文领域逻辑、订单上下文应用服务用例与接口](https://my.oschina.net/u/379999/blog/1923120)

[领域驱动设计DDD实战进阶第一波(二)：开发一般业务的大健康行业直销系统（搭建DDD的轻量级框架）](https://my.oschina.net/u/379999/blog/1922141)

[领域驱动设计DDD实战进阶第一波(一)：开发一般业务的大健康行业直销系统（概述）](https://my.oschina.net/u/379999/blog/1922121)

[领域驱动设计DDD实战进阶第一波(四)：实现产品上下文仓储与应用服务层](https://my.oschina.net/u/379999/blog/1922176)

![村长杨京京](https://oscimg.oschina.net/oscnet/up-cf904f0d61ea0fcf18b586fd616ec1a4.jpeg!/both/50x50?t=1459925359000)

### 关于作者

[**M**](https://my.oschina.net/u/379999)



![Lv1](https://static.oschina.net/new-osc/img/level/lv1_small.png)

[malaoko](https://my.oschina.net/u/379999)

关注

[私信](https://my.oschina.net/u/379999)

[提问](https://www.oschina.net/question/ask?user=379999)

[文章15](https://my.oschina.net/u/379999)

经验值

0

[粉丝1](https://my.oschina.net/u/379999/followers)[关注0](https://my.oschina.net/u/379999/following)

#### 作者的专辑

[全部](https://my.oschina.net/u/379999)

[工作日志(15)](https://my.oschina.net/u/379999?tab=newest&catalogId=1287543)

[![img](https://static.oschina.net/uploads/cooperation/blog_detail_right_sidebar_2_lpZpe.jpg)](https://www.oschina.net/action/visit/ad?id=1357)

源创计划

[立即入驻](https://www.oschina.net/sharing-plan)

自媒体入驻开源社区，

获百万流量，打造个人技术品牌

### 推荐关注

换一批 

[![LieBrother](https://oscimg.oschina.net/oscnet/up-tlikoeet8kr4g9kp6m9o7ivsjtj09afo!/both/50x50?t=1544854311000)](https://my.oschina.net/liebrother)

[LieBrother](https://my.oschina.net/liebrother)

文章 55

访问 14.9W

关注

[![polly](https://static.oschina.net/uploads/user/2/4572_50.jpg?t=1370685237000)](https://my.oschina.net/polly)

[polly](https://my.oschina.net/polly)

文章 71

访问 37.4W

关注

[![sky-flutter](https://static.oschina.net/uploads/user/391/783094_50.jpg?t=1384409096000)](https://my.oschina.net/u/783094)

[sky-flutter](https://my.oschina.net/u/783094)

文章 6

访问 3.7K

关注

[![太猪-YJ](https://static.oschina.net/uploads/user/1578/3156785_50.jpeg?t=1483688711000)](https://my.oschina.net/xiaoyoung)

[太猪-YJ](https://my.oschina.net/xiaoyoung)

文章 124

访问 25.5W

关注

[![文振熙](https://static.oschina.net/uploads/user/1197/2394822_50.png?t=1439268264000)](https://my.oschina.net/wenzhenxi)

[文振熙](https://my.oschina.net/wenzhenxi)

文章 173

访问 71W

关注

#### OSCHINA 社区

[关于我们](https://www.oschina.net/home/aboutosc)[联系我们](https://www.oschina.net/home/aboutosc)[加入我们](https://www.oschina.net/news/131099/oschina-hiring)[合作伙伴](https://www.oschina.net/home/aboutosc#partners)[Open API](https://www.oschina.net/openapi)

#### 在线工具

[Gitee.com](https://gitee.com/?utm_source=oschina&utm_medium=link-bottom&utm_campaign=home)[企业研发管理](https://gitee.com/enterprises?utm_source=oschina&utm_medium=link-bottom&utm_campaign=enterprises)[CopyCat-代码克隆检测](https://copycat.gitee.com/?utm_source=oschina&utm_medium=link-bottom&utm_campaign=copycat)[实用在线工具](https://tool.oschina.net/)[国家反诈中心APP下载](https://oscimg.oschina.net/oscnet/up-82a1d21cdcbd86bf819ffd854dc0f1c6d72.png)

#### QQ交流群

[![QQ交流群](https://static.oschina.net/new-osc/img/qq_qrcode_2_new.png)](https://jq.qq.com/?_wv=1027&k=rfiPgVgE)

[530688128](https://jq.qq.com/?_wv=1027&k=rfiPgVgE)

#### 微信公众号

![微信公众号](https://static.oschina.net/new-osc/img/wechat_qrcode.jpg?t=1484694603000)

### OSCHINA APP

聚合全网技术文章，根据你的阅读喜好进行个性推荐

[下载 APP](https://www.oschina.net/app)

©OSCHINA(OSChina.NET)

 

工信部

 [开源软件推进联盟](http://www.copu.org.cn/) 

指定官方社区

深圳市奥思网络科技有限公司版权所有

 [粤ICP备12009483号](http://beian.miit.gov.cn/)

![img](https://oscimg.oschina.net/oscnet/up-02f2706a81344119fb5cdcdda304068f2e0.png)

![返回顶部](https://static.oschina.net/new-osc/img/icon/back-to-top.svg)

顶部

[程序员大本营技术文章内容聚合第一站](https://www.pianshen.com/)



[首页 /](https://www.pianshen.com/) [联系我们 /](mailto:pianshen@gmx.com) [版权申明 /](https://www.pianshen.com/copyright.html) [隐私条款](https://www.pianshen.com/privacy-policy.html)

 

<iframe id="pianshen.com_980x300_responsive_DFP" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" width="1" height="1" style="box-sizing: border-box; width: 970px; height: 250px;"></iframe>

## 使用DDD开发电商系统中的订单业务

技术标签： [领域驱动设计](https://www.pianshen.com/tag/领域驱动设计/) [java](https://www.pianshen.com/tag/java/) [后端](https://www.pianshen.com/tag/后端/)

 

### 使用DDD开发电商系统中的订单业务

电商系统中的订单模块属于老生常谈的业务了。基本上一谈到电商系统就会聊一聊订单相关的业务。
这篇文章将使用UML来分析订单业务，使用领域模型实现订单业务。

## 分析业务

订单流程可以分为：确认订单、提交订单、付款、配送、售后服务五个部分。

- 确认订单(`incomplete`)，此时主要是确认订单的数据比如：顾客信息、配送地址、订单商品。
- 提交订单(`pending`)此时会对订单数据进行校验以及更新商品库存，比如顾客是否有效，商家是否处于营业，购买的商品库存是否满足等等。
- 付款，包括等待付款(`awaiting payment`)和付款完成(`paid`)两步骤。
  在创建订单完成并开始付款时，会将订单标记为等待付款。当付款完成后，会将订单状态标记为等待打包。此时将进入配送阶段。
- 配送，整个配送阶段由等待打包(`awaiting fulfillment`)、等待揽收(`awaiting shipment`)、
  已发货(`shipped`) / 部分发货(`partially shipped`)、等待收货(`awaiting pickup`)、确认收货(`completed`)组成。
  其中前四个状态由商家或配送员完成，后两个状态由顾客或配送员完成。
- 售后服务，对于已收到的商品存在问题(`disputed`)时，可以申请售后服务。售后服务包括：换货、退货退款、退款。
  如果是换货，在换货完成后最终还是回到`completed`状态。
  对于退款来说，退款完成后将产生新的状态分为：部分退款(`partially refunded`)、退款(`refunded`)两个状态。

整个订单状态图如下：

![image](https://www.pianshen.com/images/100/7532e53b29b87440c1572df680e7be14.JPEG)

接下来我将分别对这五个部分展开讲解。

### 确认订单

此阶段主要是确认下单的信息，比如选择配送地址、支付方式、配送时间、打印发票、使用优惠券等。

### 提交订单

此阶段将对确认订单中的数据进行校验，如果校验不合法将返回给客户端重新修改不合法的内容，
并更新订单中商品库存。

大体流程如下图：

![image](https://www.pianshen.com/images/638/cf52f19e79837d5c33158c2afe9cfa16.JPEG)

我把校验信息的部分做了简化，这个阶段只是为了说明提交订单的主要流程流程。

### 付款

当订单创建完成响应给客户端以后，客户端将发起支付。
发起支付的瞬间将订单状态更新为待支付，并设置订单的支付超时时间等等。并且还可以与商家协商改价等。

这部分的内容将来交由`trade`模块详细讲解，此处将不过多讲解。

### 配送

付款成功后，将由商家来支持配送，商家会通知仓储打包、通知物流公司揽件、发货、收货等等一系列的流程。
这些流程都可以根据上面的有限状态图看到。通常情况下发货都是一次性发出，但有的时候可能出现部分发货的情况。
比如：下单的商品里可能是预订单，此时还没有货可发，所以会先发有货的商品。再比如商品可能需要从多个不同地点的仓库发出等等。

通过上面的分析，我们可以得出订单与快递运单是一个一对多的关系。
可是大家经常看到的订单都是一对一的关系，是的通常情况下会这样。
对于更好的描述大家会对这种一对多的关系根据仓库、商家等等进行自动拆单。但是对于一个模型的分析他最终还是一个一对多的关系。

具体的类图如下：

![image](https://www.pianshen.com/images/549/03d6c06a3786802e0091d5a7b4024145.png)

此类图模型并不完整，他只是描述了今天所用到的知识。

### 售后服务

对于售后服务中心的退款、退货退款，我将在分析交易模块时再和订单模块整合讲解。此处也不做深入讲解。

### 总结

整篇文章设计的领域模型有：实体、值对象、领域服务。
其中`CustomerValidator`与`customer`模块进行通信用于校验顾客信息是否正确。
`CheckoutCounter`与`inventory`模块进行通信用于校验和更新商品库存。
有的时候你会发现有些职责不是实体所拥有的但是还需要实体进行参与其中，
这个时候就是引入领域服务的关键。
就比如实体`order`身上的`customer id`，在创建订单时需要校验用户信息是否正确，
如果你直接调用`order.validateCustomer()`会感到怪怪的。
因为校验用户信息并不是订单对象的职责，所以此时是引入领域服务的关键。

有关此示例的源码你可以查看[示例代码](https://gitee.com/tgioer/learn-ddd-order-example/tree/master/src/main/java/com/mallfoundry/order)。

## 结束

上述文章仅代表我个人思考，禁止转载，如有误人子弟之处，请您指出。

如有问题请加QQ群讨论：

![image](https://www.pianshen.com/images/868/daaeb88942402d52465e3c9f685dd1cc.png)

版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](https://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。本文链接：https://blog.csdn.net/tgioer/article/details/104211256

[原作者删帖](https://www.pianshen.com/copyright.html#del)  [不实内容删帖](https://www.pianshen.com/copyright.html#others)  [广告或垃圾文章投诉](mailto:pianshen@gmx.com?subject=投诉本文含广告或垃圾信息（请附上违规链接地址）)

<iframe id="pianshen.com_728x90_responsive_DFP" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" width="728" height="90" style="box-sizing: border-box; width: 0px; height: 0px;"></iframe>

------

### 智能推荐

[![img](https://www.pianshen.com/thumbs/792/10d6babff51a6460d789b209e97aeb40.JPEG)](https://www.pianshen.com/article/6916456482/)

### [电商系统之订单系统](https://www.pianshen.com/article/6916456482/)

电商系统之订单系统 01 概述 订单系统作为电商系统的“纽带”贯穿了整个电商系统的关键流程。其他模块都是围绕订单系统进行构建的。订单系统的演变也是随着电商平台的业务变化而逐渐演变进化着，接下来就和大家一起来解析电商平台的“生命纽带”。 上帝视角订单系统 订单系统的作用是：管理订单类型、订单状态，收集关于商品、优惠、用户、收货信息、支付信息等一系列的订...

[![img](https://www.pianshen.com/thumbs/430/1be6cf3a7dc23e42e9865978e6691586.png)](https://www.pianshen.com/article/82661074967/)

### [电商项目基本业务概述||电商后台管理系统的功能|| 电商后台管理系统的开发模式（前后端分离）|| 电商后台管理系统的技术选型](https://www.pianshen.com/article/82661074967/)

电商项目基本业务概述 根据不同的应用场景，电商系统一般都提供了 PC 端、移动 APP、移动 Web、微信小程序等多种终端访问方式。 电商后台管理系统的功能 电商后台管理系统用于管理用户账号、商品分类、商品信息、订单、数据统计等业务功能。 电商后台管理系统的开发模式（前后端分离） 电商后台管理系统整体采用前后端分离的开发模式，其中前端项目是基于 Vue 技术栈的 SPA 项目。 电商后台管理系统的...

[![img](https://www.pianshen.com/thumbs/656/e71eff4f81bf01d222344d612e7dfd60.png)](https://www.pianshen.com/article/397733568/)

### [Docker第八篇-docker-compose教程（介绍，安装，入门示例）](https://www.pianshen.com/article/397733568/)

文章目录 docker-compose介绍 docker-compose安装 安装docker（已安装最新的请忽略此步骤） docker-compose安装与卸载 docker-compose简单示例 docker-compose介绍 Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。 Compose 定位是：定义和运行多个 Docker 容器的应...

[![img](https://www.pianshen.com/thumbs/945/11632650793269650a741a712925ec69.JPEG)](https://www.pianshen.com/article/5481706532/)

### [Step By Step 1595 选夫婿2](https://www.pianshen.com/article/5481706532/)

选夫婿2 Time Limit: 1000 ms Memory Limit: 32768 KiB Submit Statistic Discuss Problem Description     倾国倾城的大家闺秀潘小姐要选夫婿啦！武林中各门各派，武林外各大户人家，闻讯纷纷前来，强势围观。前来参与竞选的...

[![img](https://www.pianshen.com/thumbs/532/df28dff041c41b99357aae24024d7874.JPEG)](https://www.pianshen.com/article/63121894234/)

### [Windows环境下安装Git教程](https://www.pianshen.com/article/63121894234/)

msysGit是 Git 版本控制系统在 Windows 下的版本。下面为大家介绍如何在windows环境下安装msysgit： 1.下载msysGit 1.1、 访问 msysGit 的项目主页(https://git-for-windows.github.io)，下载 msysGit。注：在网速慢的环境下可以寻找国内资源。 1.2、点击“Downloa...

<iframe frameborder="0" width="728" height="90" id="sas_iframe_30012" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" allowtransparency="true" style="box-sizing: border-box;"></iframe>

### 猜你喜欢

[![img](https://www.pianshen.com/thumbs/806/5a3c82756674bd8df0ed1d7763d60936.JPEG)](https://www.pianshen.com/article/3907504241/)

### [2013年春节前订票经历及经验分享](https://www.pianshen.com/article/3907504241/)

2013年春节前订票经历及经验分享 订票经历 日期 订票方式 订票情况 备注 2013-2-17 9:00准时守候在电脑前，手工订票， 订票失败 出票失败 没有足够的票。请参考截图1 2013-2-18 采用外挂软件1订票 订票失败 此软件不断提交订票请求，被官网封掉了，无法继续订票。请参考截图2 2013-2-19 采用外挂软件2订票 订票成功 请参考截图3 截图1： 截图2： 截图3： 终于出...

[![img](https://www.pianshen.com/thumbs/957/95f1664f0c6a8057577409d320812a25.png)](https://www.pianshen.com/article/5612734343/)

### [神经网络的参数为什么不能初始化为全零](https://www.pianshen.com/article/5612734343/)

训练神经网络时，随机初始化权重非常重要。对于Logistic回归，可以将权重初始化为零，但如果将神经网络的各参数全部初始化为0，再使用梯度下降法，这样将会完全无效。 如图所示，这是一个简单的两层神经网络(输入层是第0层)，如果将所有的w矩阵和b向量都初始为全0 则矩阵 是 是   是 将偏置项b初始化为0实际上是可行的，但把W初始化成全零就成问题了，它的问题在于...

[![img](https://www.pianshen.com/thumbs/565/1891c6de067846c5b7419bc1d4077ba5.png)](https://www.pianshen.com/article/7297662388/)

### [启发式搜索(Informed Search)](https://www.pianshen.com/article/7297662388/)

写在前面     我们之前几篇博客都是在讨论无信息搜索，包括深度优先、广度优先、代价一致等。我们这个内容主要讨论什么是启发式搜索和启发式函数，介绍两个比较常见的贪婪算法和A*搜索算法。并且告诉大家怎么才可以选择一个更好的启发式函数。最后是图搜索中的应用部分。现阶段，启发式算法以仿自然体算法为主，主要有模拟退火、蚁群算法、神经网络等。 一、...

[![img](https://www.pianshen.com/thumbs/587/c5f0fee3b6e93e050a84ee9b47c0c953.png)](https://www.pianshen.com/article/33611181720/)

### [疫情模拟中的SIR模型与扩展的SIRD模型](https://www.pianshen.com/article/33611181720/)

一.SIR模型  SIR模型起源于流行病学的研究，是模拟传染病动力学的经典模型。至今仍在流行病学中占据中心位置，核心在于微分方程。 SIR模型描述了流行病下三大人群：易感者 susceptible、感染者 infectious、痊愈者 recovered之间的关系。 SIR模型表述了三大人群之间的相互转化关系，并用状态转化函数来表示。 将t时刻三大人群的人数分别用以时间为自变量的函数S...

[![img](https://www.pianshen.com/thumbs/760/0d69ca1988f7a6c3c106bff1f178d420.png)](https://www.pianshen.com/article/9908843124/)

### [微信小程序客服之如何接入多客服](https://www.pianshen.com/article/9908843124/)

微信公众号越来越难做（掉粉严重，打开率大幅降低），在公众号变现这块也受到了影响。小程序上线后，不少人拥抱小程序电商，和小程序公司合作，搜索引擎竞价等，把广告打出去了，流量进来了。 不管是在用户的体验上，还是销售转化上大大提高，与此同时，接待量变大，尽管客服每天不断的优化客服QA，复制粘贴回答，依旧忙不过来，偶尔还出错，效率提升上不去怎么办？ 因为小程序自带的客服功能，比较轻。假如管理员可以设置，让...

<iframe frameborder="0" width="728" height="90" id="sas_iframe_26322" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" allowtransparency="true" style="box-sizing: border-box;"></iframe>

### 赞助商广告

- [在百万程序员中推广你的产品](mailto:pianshen@gmx.com?subject=申请广告合作)

<iframe frameborder="0" width="300" height="250" id="sas_iframe_26300" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" allowtransparency="true" style="box-sizing: border-box;"></iframe>

### 相关文章

- [DDD实战进阶第一波(十二)：开发一般业务的大健康行业直销系统（订单上下文POCO模型）](https://www.pianshen.com/article/8886696110/)
- [开发电商系统要避免哪些误区？](https://www.pianshen.com/article/91921695664/)
- [DDD实战进阶第一波(八)：开发一般业务的大健康行业直销系统（实现经销商上下文领域层之POCO模型）](https://www.pianshen.com/article/1604696758/)
- [DDD实战进阶第一波(三)：开发一般业务的大健康行业直销系统（搭建支持DDD的轻量级框架二）](https://www.pianshen.com/article/7310866108/)
- [SAP中完整的采购订单业务信息凭证流](https://www.pianshen.com/article/63581872413/)
- [【全栈之路】如何基于SpringBoot、uniapp、elementui快速开发电商系统（准备工作）](https://www.pianshen.com/article/1370852777/)
- [2.1电商项目的订单系统](https://www.pianshen.com/article/53741528163/)
- [ssm开发电商项目详细代码分享及数据库表结构设计——重点用户下订单功能与用户支付功能（免费）](https://www.pianshen.com/article/26541435874/)
- [领域驱动设计（DDD）在美团点评业务系统的实践](https://www.pianshen.com/article/6637262162/)
- [高并发电商秒杀的演进](https://www.pianshen.com/article/7554730803/)

<iframe id="pianshen.com_300x600_responsive_DFP" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" width="300" height="600" style="box-sizing: border-box; width: 300px; height: 600px;"></iframe>

### 热门文章

- [Project configuration is not up-to-date with pom.xml](https://www.pianshen.com/article/89901336473/)
- [在 Ubuntu Edgy 6.10 中成功安装 JBuilder 2006 Enterprise Edition ！](https://www.pianshen.com/article/4004628725/)
- [C++11新特性（62）- 模板函数的默认模板参数](https://www.pianshen.com/article/1593569175/)
- [001-看到了就记录下来](https://www.pianshen.com/article/6513495941/)
- [AI一分钟 | 小米在香港提交招股书募资100亿美元；寒武纪发布首款云端AI芯片和第三代终端IP...](https://www.pianshen.com/article/1292870145/)
- [港科夜闻丨香港科大商学院副院长徐岩在《哈佛商业评论》中国年会发表讲话...](https://www.pianshen.com/article/66161702497/)
- [CTF,web题，faster](https://www.pianshen.com/article/2527907725/)
- [腾讯回应封杀质疑 腾讯公关总监说了这样一番话](https://www.pianshen.com/article/38451232825/)
- [Android Studio--Gradle Project工具栏如何显示](https://www.pianshen.com/article/22611272648/)
- [freemarker模板引擎](https://www.pianshen.com/article/69621389054/)

<iframe frameborder="0" width="300" height="250" id="sas_iframe_26711" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" allowtransparency="true" style="box-sizing: border-box;"></iframe>

### 推荐文章

- [数据库分片以及schema概念](https://www.pianshen.com/article/26251079531/)
- [千兆网络PHY芯片 RTL8211E的实践应用（原理图及PCB实现）](https://www.pianshen.com/article/94671623664/)
- [Python实现经典排序算法-堆排序](https://www.pianshen.com/article/5862405390/)
- [无人机小区上空盘一圈测体温，背后技术靠谱吗？](https://www.pianshen.com/article/64731146442/)
- [LBP变体—LTP纹理特征](https://www.pianshen.com/article/3481644149/)
- [在windows下安装scrapy](https://www.pianshen.com/article/77081343355/)
- [系统缓存全解析 [转\]](https://www.pianshen.com/article/209667519/)
- [Python 中实用却不常见的小技巧！](https://www.pianshen.com/article/2325346376/)
- [【CF 应用开发大赛】乐窝－分享幽默搞笑段子](https://www.pianshen.com/article/50181952682/)
- [大数据小视角4：小议Lambda 与 Kappa 架构，不可变数据的计算探索](https://www.pianshen.com/article/1021374062/)

<iframe frameborder="0" width="300" height="600" id="sas_iframe_26323" scrolling="no" marginheight="0" marginwidth="0" topmargin="0" leftmargin="0" allowtransparency="true" style="box-sizing: border-box;"></iframe>

### 相关标签

- [DDD](https://www.pianshen.com/tag/DDD/)
- [领域驱动设计](https://www.pianshen.com/tag/领域驱动设计/)
- [软件架构](https://www.pianshen.com/tag/软件架构/)
- [DDD实战](https://www.pianshen.com/tag/DDD实战/)
- [技术积累](https://www.pianshen.com/tag/技术积累/)
- [全栈之路](https://www.pianshen.com/tag/全栈之路/)
- [spring](https://www.pianshen.com/tag/spring/)
- [java](https://www.pianshen.com/tag/java/)
- [springboot](https://www.pianshen.com/tag/springboot/)
- [uniapp](https://www.pianshen.com/tag/uniapp/)

Copyright © 2018-2021 - All Rights Reserved - [www.pianshen.com](https://www.pianshen.com/) 网站内容人工审核和清理中！

<iframe src="about:blank" class="_ntnrjf7826-hj" style="box-sizing: border-box; color: rgb(51, 51, 51); font-family: &quot;open sans&quot;, sans-serif; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial; width: 0px !important; height: 0px !important; border: 0px !important; position: absolute !important; top: -10000px !important; left: -10000px !important;"></iframe>

<iframe src="about:blank" class="_ntnrjf7826-hj" style="box-sizing: border-box; color: rgb(51, 51, 51); font-family: &quot;open sans&quot;, sans-serif; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial; width: 0px !important; height: 0px !important; border: 0px !important; position: absolute !important; top: -10000px !important; left: -10000px !important;"></iframe>

<iframe src="about:blank" width="728" height="90" frameborder="0" allow="autoplay;fullscreen;" scrolling="no" marginheight="0" marginwidth="0" id="sas_26328_iframe" style="box-sizing: border-box;"></iframe>

![img](https://ced-ns.sascdn.com/diff/templates/images/close-retina.png)[![CSDN首页](https://img-home.csdnimg.cn/images/20201124032511.png)](https://www.csdn.net/)

- [首页](https://www.csdn.net/)
- [博客](https://blog.csdn.net/)
- [程序员学院](https://edu.csdn.net/)
- [下载](https://download.csdn.net/)
- [论坛](https://bbs.csdn.net/)
- [问答](https://ask.csdn.net/)
- [代码](https://codechina.csdn.net/?utm_source=csdn_toolbar)
- [直播](https://live.csdn.net/?utm_source=csdn_toolbar)
- [能力认证](https://ac.csdn.net/)
- [高校](https://studentclub.csdn.net/)

 

[![img](https://profile.csdnimg.cn/D/B/6/2_yangjava2014)](https://blog.csdn.net/yangjava2014)

[会员中心](https://mall.csdn.net/vip)

[收藏](https://i.csdn.net/#/user-center/collection-list?type=1)

[动态](https://blog.csdn.net/nav/watchers)

[消息](https://i.csdn.net/#/msg/index)

[创作中心](https://mp.csdn.net/)

# DDD 使用总结

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[饮水偲源](https://blog.csdn.net/hesiyuanlx) 2019-10-23 16:17:27 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 720 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏

分类专栏： [学习笔记；](https://blog.csdn.net/hesiyuanlx/category_6831524.html) 文章标签： [DDD](https://so.csdn.net/so/search/s.do?q=DDD&t=blog&o=vip&s=&l=&f=&viparticle=) [领域](https://so.csdn.net/so/search/s.do?q=领域&t=blog&o=vip&s=&l=&f=&viparticle=)

版权

###### 概念

领域：边界内要解决的业务问题域
子领域：在领域范围内对应一个更小的问题域或更小的业务范围

###### 拆分过程

确认研究对象，即研究领域。
将对象进行细分，拆分为子领域。
每个子领域再拆分，行成更小子领域。

###### 核心域

决定产品和公司核心竞争力的子域是核心域，它是业务成功的主要因素和公司的核心竞争力。
注册、登陆、充值、体现、下单、商品信息推送。

###### 通用域

同时被多个子域使用的通用功能子域是通用域。
授权、认证等

支撑域：不属于通用功能，但是不包含决定产品和公司核心竞争力的功能。
订单拆分。

###### 限界上下文

确定领域边界。

###### 通用语言

在事件风暴过程中，通过团队交流达成共识的，能够简单、清晰、准确描述业务涵义和规则的语言就是通用语言。
通用语言包含属于和用例场景，并且能够直接反应在代码中。通用语言中的名词可以给领域对象命名、如商品、订单等，对应实体对象；而动词则表示是一个动作或实际，如商品已下单、订单已付款等，对领域事件或或者命令。

###### 实体

1.标识符->历经状态变更后标识符不产生变化，拥有延续性和标识，甚至超出软件生命周期。
2.多个属性、操作或行为的载体。在事件风暴中，我们可以根据命令、操作或者事件，找出这些行为的业务实体对象，进而按照一定的业务规则将依存度高和业务关联紧密的多个实体对象和值对象进行聚类，行程聚合。
3.有ID标识，通过ID判断相等性，ID在聚合内唯一。状态可变，依附于聚合根，其生命周期由聚合根管理。实体一般会持久化，但是与数据库持久化对象不一定是一对一。

业务形态：实体和值对象是组成领域模型的基础单元。
代码形态：充血模型，跨多个实体的领域逻辑在领域服务中实现。
ps：充血模型：大多业务逻辑和持久化放在Domain Object里，Business Logic只是简单封装部分业余逻辑以及控制事务、权限等，层次Client ->(Business Facade)->BUsiness Logic -> Domain Object ->Data Access
优点面向对象，Business Logic符合单一职责，不像贫血模型包含所有的业务逻辑太过沉重。
缺点，如何划分业务逻辑，哪些逻辑在Domain Object，Domain Logic包含了持久化，对于开发者来说十分混乱，其次，Business Logic要控制事务，并且为上一层提供一个统一的服务调用入口点，必须把Domain Logic里实现的业务逻辑全部封装一遍。

贫血模型，只有get和set。所有业务逻辑都在Business Logic层。
优点，更层次单项依赖。
缺点，不够面向对象，领域对象知识作为保存状态和传递状态使用，只有数据没有行为。

##### 聚合

领域模型内的实体和值对象就好比个体，而能让实体和值对象协同工作的组织就是聚合，它用来确保这些领域对象在实现共同的业务逻辑时，能保证数据的一致性。
1.高内聚、低耦合，它是领域模型中最底层的边界，可以作为拆分微服务的最小单位，但不建议对微服务过渡拆分。对性能有机制要求的场景，聚合可以独立未一个微服务。

##### 聚合根

1.是一个实体。
2.聚合的管理者，负责协调实体和值对象按照固定的业务规则协同完成共通的业务逻辑。
3.具有全局唯一标识，有独立的生命周期。
4.一个聚合只有一个聚合根。

个人理解：
订单->聚合根
订单金额
订单商品列表
订单状态等实体聚合为一个聚合

##### 聚合的一些设计原则

1.在一致性边界内建模真正的不变。
聚合用来封装真正的不变性，而不是简单地将对象组合在一起。聚合内有一套不变的业务规则，各实体和值对象按照统一的业务规则运行，实现对象数据的一致性，边界之外的任何东西都与该聚合无关，这就是聚合能实现业务高内聚的原因。

2.聚合设计的足够小。
如果聚合设计得过大，聚合会因为包含过多的实体，导致实体之间的管理过于复杂，高频操作时会出现并发冲突或者数据库锁，最终导致系统可用性变差。而小聚合设计则可以降低由于业务过大导致聚合重构的可能性，让领域模型更能适应业务的变化。

3.通过唯一标识引用其他聚合。
聚合之间的通过关联外部聚合根ID的方式引用，而不是直接对象引用方式。外部聚合的对象放在聚合边界内管理，容易导致聚合的边界不清晰，也会增加结合直接的耦合度。

4.在边界之外使用最终一致性。
聚合内数据强一致性吗，而聚合之间的数据保障最终一致性，实现聚合之间的解耦。

5.通过应用层实现跨聚合的服务调用。
避免跨聚合的领域服务调用和跨聚合的数据库表关联（同时意味着跨聚合事务也是不推荐的？）



- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarThumbUp.png)点赞
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarComment.png)评论](https://blog.csdn.net/hesiyuanlx/article/details/102699833#commentBox)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarShare.png)分享](javascript:;)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)收藏](javascript:;)
- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarReport.png)举报
- [关注](javascript:;)
- 一键三连

[linux下*ddd*工具的*使用*](https://download.csdn.net/download/change66/6675111)

12-07

[介绍linux环境下的*ddd*工具的*使用*方法。](https://download.csdn.net/download/change66/6675111)

[*DDD*（领域驱动设计）*总结*](https://blog.csdn.net/qq773837256/article/details/52023905)

[绯雨倾城](https://blog.csdn.net/qq773837256)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[基本概念： 　　领域驱动设计(简称 *ddd*)概念来源于2004年著名建模专家eric evans发表的他最具影响力的书籍:《domain-driven design –tackling complexity in the heart of software》(中文译名：领域驱动设计—软件核心复杂性应对之道)一书。，书中提出了“领域驱动设计(简称 *ddd*)”的概念。      领](https://blog.csdn.net/qq773837256/article/details/52023905)





[![img](https://profile.csdnimg.cn/D/B/6/3_yangjava2014)](https://blog.csdn.net/yangjava2014)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/commentFlag@2x.png)

![表情包](https://csdnimg.cn/release/blogv2/dist/pc/img/emoticon.png)

相关推荐

[*DDD*(领域驱动设计)*总结*_深圳-吴迪的博客](https://blog.csdn.net/qq_32109957/article/details/114358667)

4-29

[*DDD*(领域驱动设计)*总结* 基本概念: 领域驱动设计(简称 *ddd*)概念来源于2004年著名建模专家eric evans发表的他最具影响力的书籍:《domain-driven design –tackling complexity in the heart of software》(中文译名:领域驱动设计—软件核心...](https://blog.csdn.net/qq_32109957/article/details/114358667)

[*DDD*领域*总结*_技术分子博客_*ddd*领域](https://blog.csdn.net/u014205434/article/details/108850322)

4-11

[*DDD*领域*总结* *DDD*是什么? "而*DDD*则是对传统的以数据为中心的建模方式的反思结果。" *DDD*战略: 领域 限界上下文(可以通俗理解为业务场景或语境) 上下文映射 架构等 上下文映射图: 上下文映射图帮助我们理解业务领域、模型间的边界,以及这些...](https://blog.csdn.net/u014205434/article/details/108850322)

[对 *DDD* 的*总结*与思考（一）](https://blog.csdn.net/weixin_34384681/article/details/91415285)

[weixin_34384681的博客](https://blog.csdn.net/weixin_34384681)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 85

[我的 boss 曾经问过我一个很基本的问题：你觉得如何划分一个项目的结构？ 我想现在的我依然很难回答这个问题。尽管目前对于纯技术的资料已经汗牛充栋，对于架构设计却依然是一门隐学——我们大都在意的是数据库如何拆分，服务如何设计之类。火热的微服务也只是一种实现上，软件层的拆分，而不是逻辑层的分离。*DDD* 其实是软件知识体系中的哲学，它是无关于任何技术学科，比如数据库、分布式甚至是软件工程的，它只是一个...](https://blog.csdn.net/weixin_34384681/article/details/91415285)

[基于*DDD*项目的设计*总结*](https://blog.csdn.net/iteye_19891/article/details/81690850)

[jia.hej](https://blog.csdn.net/iteye_19891)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 178

[一、为什么要*使用*领域模型 • 有助于团队创建一个业务部门与IT部门都能理解的通用模型，并用该模型来沟通业务需求、数据实体、过程模型。 • 模型是模块化、可扩展、易于维护的，同时设计还反映了业务模型。 • 提高了业务领域对象的可重用性和可测性。 二、领域的分层架构 在Eric Evans《领域驱动设计--软件核心复杂性应对之道》中对领域的分层架构如下： [img\]/upload/at...](https://blog.csdn.net/iteye_19891/article/details/81690850)

[领域驱动设计 (*DDD*) *总结*_琦小虾的代码世界](https://blog.csdn.net/ajianyingxiaoqinghan/article/details/107294929)

4-3

[*DDD* (Domain-Driven Design),即领域驱动设计是思考问题的方法论,用于对实际问题建模,它以一种领域专家、设计人员、开发人员都能理解的通用语言作为相互交流的工具,然后将这些概念设计成一个领域模型。由领域模型驱动软件设计,用代码来实现...](https://blog.csdn.net/ajianyingxiaoqinghan/article/details/107294929)

[商品领域*ddd*_干货*总结*:*使用* *DDD* 指导微服务拆分的逻辑...](https://blog.csdn.net/weixin_39602976/article/details/111644457)

4-7

[在大量*使用**DDD*指导微服务拆分的实践后,我们发现很多系统设计存在一些常见的误区,主要分为三类:未成功做出抽象、抽象程度过高、错误的抽象。 未成功做出抽象 在实际开发过程中,大家都有一个体会,设计阶段只考虑了一些常见的服务,但是发现项目...](https://blog.csdn.net/weixin_39602976/article/details/111644457)

[*DDD*-建模过程分析](https://blog.csdn.net/whos2016/article/details/103430565)

[最爱下雨天](https://blog.csdn.net/whos2016)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 970

[*DDD*-建模过程分析 Eric Evans的“Domain-Driven Design领域驱动设计”，简称*DDD*，Evans *DDD*是一套综合软件系统分析和设计的面向对象建模方法，其核心就是建立正确的足够精良且符合业务需求的领域模型。 目前广为流行的项目开发模式如瀑布模型、敏捷开发模型等，都将软件分析与设计作为两个独立的阶段，这样的割裂导致需求分析的结果往往无法直接进行设计编程，而实现的软件结果往...](https://blog.csdn.net/whos2016/article/details/103430565)

[深入认识*DDD*](https://blog.csdn.net/Hackerfang/article/details/79983336)

[Hackerfang的博客](https://blog.csdn.net/Hackerfang)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1122

[数字化转型中*DDD*的重要性愈加突出数字化转型是当前非常热门的一个话题。那到底什么是数字化转型呢？它是指通过各种数字化的手段对各行各业进行改造以重塑其运营模式、商业模式和客户交互等模式。我对其本质特征的理解是自动化一切，以达到极致效率，最典型的表现便是：1. 通过互联网技术使信息传递自动化，从而消除一切信息不对等。传统产业的很多运作模式其实是基于信息不对等的历史状况产生的。因为信息不对等，传统B2B...](https://blog.csdn.net/Hackerfang/article/details/79983336)

[最近很火的*DDD*(领域驱动设计)，在微服务中怎么*使用*？](https://jsldl.blog.csdn.net/article/details/100082427)

[技术领导力](https://blog.csdn.net/yellowzf3)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1012

[许多朋友给挨踢哥留言，想深入学习*DDD*，以及如何在微服务设计中应用。本文先来探讨*DDD*。什么是*DDD*？ 领域建模的基础是要先理解领域，让自己成为领域专家。如...](https://jsldl.blog.csdn.net/article/details/100082427)

[*DDD*第1篇 - 为什么*使用**DDD*？](https://blog.csdn.net/yasinshaw/article/details/103306562)

[yasinshaw的博客](https://blog.csdn.net/yasinshaw)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1099

[点击上方“xy的技术圈”，选择“设为星标”认真写文章，用心做分享。微信公众号：xy的技术圈个人网站：yasinshaw.com正文先说声抱歉，最近两三个星期都没有产出新的文章了。一方面是...](https://blog.csdn.net/yasinshaw/article/details/103306562)

[如何运用*DDD* - 实体](https://blog.csdn.net/weixin_45969084/article/details/103391249)

[weixin_45969084的博客](https://blog.csdn.net/weixin_45969084)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 194

[概述 本文将介绍领域驱动设计（*DDD*）战术模式中另一个常见且非常重要的概念 - 实体。相对战术模式中其他的一些概念（例如 值对象、领域服务等）来说，实体应该比较容易让人理解和运用。但是我们如何去发现所在领域中的实体呢？如何保证建立的实体是富含行为的？实体运用时又有那些注意的细节呢？本文将从不同的角度来带大家重新认识一下“实体”这个概念，并且给出相应的代码片段（本教程的代码片段都*使用*的是C#,后期的...](https://blog.csdn.net/weixin_45969084/article/details/103391249)

[嵌入式开发中*使用**DDD*进行调试](https://blog.csdn.net/junecauzhang/article/details/7671817)

[junecauzhang的专栏](https://blog.csdn.net/junecauzhang)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 557

[嵌入式开发中*使用**DDD*进行调试 (2010-12-30 16:12:15) 标签： 杂谈 分类： 编程开发     在嵌入式程序开发过程中，程序员要进行大量的调试，以此验证程序的正确性，修改潜在的错误。调试器对于程序员来说是不可或缺的必备工具。在Linux环境中，有很多调试工具和调试辅助工具，例如GDB、XXGDB、RHIDE](https://blog.csdn.net/junecauzhang/article/details/7671817)

[领域驱动架构学习*总结*](https://blog.csdn.net/weixin_30830327/article/details/99106464)

[weixin_30830327的博客](https://blog.csdn.net/weixin_30830327)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 249

[基本概念 　　领域驱动设计(简称 *DDD*)概念来源于2004年著名建模专家Eric Evans发表的他最具影响力的书籍:《Domain-Driven Design –Tackling Complexity in the Heart of Software》(中文译名：领域驱动设计—软件核心复杂性应对之道)一书。，书中提出了“领域驱动设计(简称 *DDD*)”的概念。 领...](https://blog.csdn.net/weixin_30830327/article/details/99106464)

[*DDD*（领域驱动设计）*总结*](https://blog.csdn.net/woshihyykk/article/details/108538608)

[SToNE的博客](https://blog.csdn.net/woshihyykk)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 505

[领域驱动设计*总结*基本概念：1．实体(entity)：2．值对象(value object)3．聚合及聚合根(aggregate、aggregate root)：4．工厂(factories)：5．仓储（repositories）：6．服务（services）：7．domain事件8．DTO设计领域模型的一般步骤一些思考1. 建立完整自封闭的领域模型。2. 领域服务建模3．领域对象、领域服务以及repository之间的互相依赖 基本概念： 领域驱动设计(简称 *ddd*)概念来源于2004年著名建模专家e](https://blog.csdn.net/woshihyykk/article/details/108538608)

[*DDD*学习笔记 - *总结*篇](https://blog.csdn.net/hz_940611/article/details/103329520)

[haozz的博客](https://blog.csdn.net/hz_940611)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 739

[19 | *总结*（一）：微服务设计和拆分要坚持哪些原则 课程链接：https://time.geekbang.org/column/article/171185 由于企业发展历程以及企业技术和文化的不同，*DDD* 和微服务的实施策略也会有差异。那么面对这种差异，应该如何落地 *DDD* 和微服务呢？ 微服务的演进策略 在从单体向微服务演进时，演进策略大体分为两种：绞杀者策略和修缮者策略。 ...](https://blog.csdn.net/hz_940611/article/details/103329520)

[学习*DDD**总结*](https://blog.csdn.net/weixin_33728708/article/details/91562202)

[weixin_33728708的博客](https://blog.csdn.net/weixin_33728708)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 46

[为什么80%的码农都做不了架构师？>>> ...](https://blog.csdn.net/weixin_33728708/article/details/91562202)

[前端*DDD**总结*与思考](https://blog.csdn.net/weixin_55003637/article/details/115767043)

[weixin_55003637的博客](https://blog.csdn.net/weixin_55003637)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 313

[软件开发架构演化与*DDD*起源 单体服务架构：大概10年前，我在武汉工作的时候，甲方客户购买我们的产品，一般都是连着设备一起购买，一套软件系统，一台惠普或者戴尔的企业级服务器，再加一个黑色的铁盒，销售部可以卖小几百万左右、外加每年维护费用，这就是一个非常典型的单体服务开发、运作的方式。 SOA服务化、微服务架构：前端开发同学普遍离业务域结构比较远，一般比较难真正意义上去理解服务化与微服务，我个人对微服务和SOA区别的理解，SOA服务化，往往侧重某一类具体业务场景下功能的实现，它必须有一个明确的业务场景来做支.](https://blog.csdn.net/weixin_55003637/article/details/115767043)

[Windows版YOLOv4目标检测实战：中国交通标志识别](https://edu.csdn.net/course/detail/29363)

06-03

<p> <span style="color:#0070C0;">课程演示环境：Windows10</span> </p> <p> <span style="color:#0070C0;"><br /> 需要学习Ubuntu系统YOLOv4的同学请前往《YOLOv4目标检测实战：中国交通标志识别》</span> </p> <p> <br /> </p> <p> 在自动驾驶中，交通标志识别是一项重要的任务。本项目以中国交通标志数据集TT100K为训练对象，采用YOLOv4目标检测方法实现实时交通标志识别。 </p> <p> <br /> </p> <p> 本课程的YOLOv4使用AlexyAB/darknet，在Windows系统上做项目演示。具体项目过程包括：安装YOLOv4、TT100K标注格式转换成PASCAL VOC格式、训练集和测试集自动划分、修改配置文件、训练网络模型、测试训练出的网络模型、性能统计、先验框聚类分析和NMS（非极大抑制）方法修改。  </p> <p> <br /> </p> <p> 本课程会讲述使用Python程序将TT100K数据集的格式转换成PASCAL VOC格式和YOLO格式的方法，并提供相应代码。 </p> <p> <img src="https://img-bss.csdnimg.cn/202006030216268035.jpg" alt="" /> </p>

[华为手机鸿蒙 OS 2.0 开机界面演示](https://blog.csdn.net/weixin_39787242/article/details/116453280)

[weixin_39787242的博客](https://blog.csdn.net/weixin_39787242)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[本文转载自IT之家 IT之家5 月 5 日消息上个月底，部分华为用户收到了鸿蒙 OS 2.0 开发者 Beta 公测版推送，IT之家也多次报道了与 EMUI 系统的对比。 今日，数码博主 @长安数码君 发布了华为手机鸿蒙 OS 2.0 开机界面与 EMUI 开机界面的对比，鸿蒙 OS 2.0 开机界面动画出现了不小的变动，并出现了 HarmonyOS 的标志。 最值得一提的是，鸿蒙 OS 2.0 开机界面终于去掉了原本底部的Powered by Android 字样。 IT之家了解到，...](https://blog.csdn.net/weixin_39787242/article/details/116453280)

[当年，兔子学姐靠这个面试小抄拿了个22k](https://fantianzuo.blog.csdn.net/article/details/116449776)

[hebtu666](https://blog.csdn.net/hebtu666)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[都是当时准备面试*总结*的，学生面试基本不会跑出这个范围，懂行的应该能看出来。](https://fantianzuo.blog.csdn.net/article/details/116449776)

[转载：微服务拆分](https://blog.csdn.net/weixin_30466953/article/details/101826824)

[weixin_30466953的博客](https://blog.csdn.net/weixin_30466953)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 234

[这篇文章讲的太好了，生怕以后被删掉。转到我博文列表里把-。- 原文地址 正文： 开发者在刚开始尝试实现自己的微服务架构时往往会产生一系列问题 ： 微服务到底应该怎么划分？ 一个典型的微服务到底应该有多微？ 如果做了微服务设计，最后真的会有好处吗？ 回答上面的问题需要首先了解微服务设计的逻辑，科学的架构设计应该通过一些输入并逐步推导出结果，架构师要避免凭空设计和...](https://blog.csdn.net/weixin_30466953/article/details/101826824)

©️2020 CSDN 皮肤主题: 编程工作室 设计师:CSDN官方博客 [返回首页](https://blog.csdn.net/)

- [关于我们](https://www.csdn.net/company/index.html#about)
- [招贤纳士](https://www.csdn.net/company/index.html#recruit)
- [广告服务](https://www.csdn.net/company/index.html#advertisement)
- [开发助手](https://plugin.csdn.net/)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/tel.png)400-660-0108
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/email.png)[kefu@csdn.net](mailto:webmaster@csdn.net)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/cs.png)[在线客服](https://csdn.s2.udesk.cn/im_client/?web_plugin_id=29181)
- 工作时间 8:30-22:00

- [公安备案号11010502030143](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010502030143)
- [京ICP备19004658号](http://beian.miit.gov.cn/publish/query/indexFirst.action)
- [京网文〔2020〕1039-165号](https://csdnimg.cn/release/live_fe/culture_license.png)
- [经营性网站备案信息](https://csdnimg.cn/cdn/content-toolbar/csdn-ICP.png)
- [北京互联网违法和不良信息举报中心](http://www.bjjubao.org/)
- [网络110报警服务](http://www.cyberpolice.cn/)
- [中国互联网举报中心](http://www.12377.cn/)
- [家长监护](https://download.csdn.net/index.php/tutelage/)
- [Chrome商店下载](https://chrome.google.com/webstore/detail/csdn开发者助手/kfkdboecolemdjodhmhmcibjocfopejo?hl=zh-CN)
- ©1999-2021北京创新乐知网络技术有限公司
- [版权与免责声明](https://www.csdn.net/company/index.html#statement)
- [版权申诉](https://blog.csdn.net/blogdevteam/article/details/90369522)
- [出版物许可证](https://img-home.csdnimg.cn/images/20210414021151.jpg)
- [营业执照](https://img-home.csdnimg.cn/images/20210414021142.jpg)

[![img](https://profile.csdnimg.cn/A/9/7/3_hesiyuanlx)](https://blog.csdn.net/hesiyuanlx)

[饮水偲源](https://blog.csdn.net/hesiyuanlx)

码龄9年[![img](https://csdnimg.cn/identity/nocErtification.png) 暂无认证](https://i.csdn.net/#/uc/profile?utm_source=14998968)




- 3万+

  访问

- [![img](https://csdnimg.cn/identity/blog4.png)](https://blog.csdn.net/blogdevteam/article/details/103478461)

  等级

- 835

  积分

- 6

  粉丝

- 3

  获赞

- 2

  评论

- 16

  收藏

![持之以恒](https://csdnimg.cn/medal/chizhiyiheng@240.png)

![勤写标兵Lv1](https://csdnimg.cn/medal/qixiebiaobing1@240.png)

[私信](https://im.csdn.net/chat/hesiyuanlx)

关注

![img](https://csdnimg.cn/cdn/content-toolbar/csdn-sou.png?v=1587021042)

### 热门文章

- [EDAS平台使用感受 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 8074](https://blog.csdn.net/hesiyuanlx/article/details/79207330)
- [[DUBBO\] Thread pool is EXHAUSTED! 关于duboo provider并发限流的错误及解决方案 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 7946](https://blog.csdn.net/hesiyuanlx/article/details/78748487)
- [docker镜像新增文件/修改文件方法 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 2324](https://blog.csdn.net/hesiyuanlx/article/details/107312088)
- [几乎原生Mysql配置 执行Update语句卡住一直执行很长时间才返回(问题排查) ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1913](https://blog.csdn.net/hesiyuanlx/article/details/106546933)
- [Dubbo 性能调优经历（一） ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1878](https://blog.csdn.net/hesiyuanlx/article/details/79207466)

### 分类专栏

- [![img](https://img-blog.csdnimg.cn/20201014180756926.png?x-oss-process=image/resize,m_fixed,h_64,w_64)mysq3篇](https://blog.csdn.net/hesiyuanlx/category_10044210.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756927.png?x-oss-process=image/resize,m_fixed,h_64,w_64)事故1篇](https://blog.csdn.net/hesiyuanlx/category_10044211.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756925.png?x-oss-process=image/resize,m_fixed,h_64,w_64)jdk8时间工具1篇](https://blog.csdn.net/hesiyuanlx/category_9900964.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756925.png?x-oss-process=image/resize,m_fixed,h_64,w_64)学习笔记；28篇](https://blog.csdn.net/hesiyuanlx/category_6831524.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756738.png?x-oss-process=image/resize,m_fixed,h_64,w_64)java14篇](https://blog.csdn.net/hesiyuanlx/category_7432298.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756724.png?x-oss-process=image/resize,m_fixed,h_64,w_64)阿里java开发规范3篇](https://blog.csdn.net/hesiyuanlx/category_7432299.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756918.png?x-oss-process=image/resize,m_fixed,h_64,w_64)dubbo学习笔记](https://blog.csdn.net/hesiyuanlx/category_7432646.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756919.png?x-oss-process=image/resize,m_fixed,h_64,w_64)duboo从入门到深渊2篇](https://blog.csdn.net/hesiyuanlx/category_7432972.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756922.png?x-oss-process=image/resize,m_fixed,h_64,w_64)dubbo调优2篇](https://blog.csdn.net/hesiyuanlx/category_7433520.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756922.png?x-oss-process=image/resize,m_fixed,h_64,w_64)linux2篇](https://blog.csdn.net/hesiyuanlx/category_8561040.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756916.png?x-oss-process=image/resize,m_fixed,h_64,w_64)IDEA1篇](https://blog.csdn.net/hesiyuanlx/category_8820833.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756925.png?x-oss-process=image/resize,m_fixed,h_64,w_64)redis2篇](https://blog.csdn.net/hesiyuanlx/category_8953945.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756918.png?x-oss-process=image/resize,m_fixed,h_64,w_64)缓存2篇](https://blog.csdn.net/hesiyuanlx/category_8953946.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756930.png?x-oss-process=image/resize,m_fixed,h_64,w_64)分布式缓存1篇](https://blog.csdn.net/hesiyuanlx/category_8953947.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756923.png?x-oss-process=image/resize,m_fixed,h_64,w_64)key与槽1篇](https://blog.csdn.net/hesiyuanlx/category_8953948.html)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/arrowDownWhite.png)

### 最新评论

- 参数校验反射工具包

  [weixin_40801109: ](https://blog.csdn.net/weixin_40801109)是不是少了个 ValidateFiled

- 阿里java开发规范学习 汇总

  [lily-0622: ](https://blog.csdn.net/yao940622)感谢您的分享

### 最新文章

- [HTTPS 学习](https://blog.csdn.net/hesiyuanlx/article/details/110525915)
- [腾讯云 CODING学习笔记](https://blog.csdn.net/hesiyuanlx/article/details/107688494)
- [腾讯云TdSQL 学习笔记](https://blog.csdn.net/hesiyuanlx/article/details/107655853)

[2020年11篇](https://blog.csdn.net/hesiyuanlx/article/month/2020/12)

[2019年12篇](https://blog.csdn.net/hesiyuanlx/article/month/2019/10)

[2018年13篇](https://blog.csdn.net/hesiyuanlx/article/month/2018/12)

[2017年11篇](https://blog.csdn.net/hesiyuanlx/article/month/2017/12)

![img]()



![img](https://g.csdnimg.cn/side-toolbar/3.0/images/guide.png)![img](https://g.csdnimg.cn/side-toolbar/3.0/images/kefu.png)举报

[![CSDN首页](https://img-home.csdnimg.cn/images/20201124032511.png)](https://www.csdn.net/)

- [首页](https://www.csdn.net/)
- [博客](https://blog.csdn.net/)
- [程序员学院](https://edu.csdn.net/)
- [下载](https://download.csdn.net/)
- [论坛](https://bbs.csdn.net/)
- [问答](https://ask.csdn.net/)
- [代码](https://codechina.csdn.net/?utm_source=csdn_toolbar)
- [直播](https://live.csdn.net/?utm_source=csdn_toolbar)
- [能力认证](https://ac.csdn.net/)
- [高校](https://studentclub.csdn.net/)

 

[![img](https://profile.csdnimg.cn/D/B/6/2_yangjava2014)](https://blog.csdn.net/yangjava2014)

[会员中心](https://mall.csdn.net/vip)

[收藏](https://i.csdn.net/#/user-center/collection-list?type=1)

[动态](https://blog.csdn.net/nav/watchers)

[消息](https://i.csdn.net/#/msg/index)

[创作中心](https://mp.csdn.net/)

# 领域驱动设计（DDD）在美团点评业务系统的实践

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/reprint.png)

[Winsdons](https://blog.csdn.net/qq_36510261) 2017-12-28 16:07:29 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 3329 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏 2

**前言**

至少30年以前，一些软件设计人员就已经意识到领域建模和设计的重要性，并形成一种思潮，Eric Evans将其定义为领域驱动设计（Domain-Driven Design，简称DDD）。在互联网开发“小步快跑，迭代试错”的大环境下，DDD似乎是一种比较“古老而缓慢”的思想。

然而，由于互联网公司也逐渐深入实体经济，业务日益复杂，我们在开发中也越来越多地遇到传统行业软件开发中所面临的问题。本文就先来讲一下这些问题，然后再尝试在实践中用DDD的思想来解决这些问题。

**问题**

### **过度耦合**

业务初期，我们的功能大都非常简单，普通的CRUD就能满足，此时系统是清晰的。随着迭代的不断演化，业务逻辑变得越来越复杂，我们的系统也越来越冗杂。模块彼此关联，谁都很难说清模块的具体功能意图是啥。修改一个功能时，往往光回溯该功能需要的修改点就需要很长时间，更别提修改带来的不可预知的影响面。

下图是一个常见的系统耦合病例。

![cff5b75c7635ceb69a489eb4c55823d16f6ec2f3](https://yqfile.alicdn.com/cff5b75c7635ceb69a489eb4c55823d16f6ec2f3.png)

订单服务接口中提供了查询、创建订单相关的接口，也提供了订单评价、支付、保险的接口。同时我们的表也是一个订单大表，包含了非常多字段。在我们维护代码时，牵一发而动全身，很可能只是想改下评价相关的功能，却影响到了创单核心路径。虽然我们可以通过测试保证功能完备性，但当我们在订单领域有大量需求同时并行开发时，改动重叠、恶性循环、疲于奔命修改各种问题。

上述问题，归根到底在于系统架构不清晰，划分出来的模块内聚度低、高耦合。

有一种解决方案，按照演进式设计的理论，让系统的设计随着系统实现的增长而增长。我们不需要作提前设计，就让系统伴随业务成长而演进。这当然是可行的，敏捷实践中的重构、测试驱动设计及持续集成可以对付各种混乱问题。重构——保持行为不变的代码改善清除了不协调的局部设计，测试驱动设计确保对系统的更改不会导致系统丢失或破坏现有功能，持续集成则为团队提供了同一代码库。

在这三种实践中，重构是克服演进式设计中大杂烩问题的主力，通过在单独的类及方法级别上做一系列小步重构来完成。我们可以很容易重构出一个独立的类来放某些通用的逻辑，但是你会发现你很难给它一个业务上的含义，只能给予一个技术维度描绘的含义。这会带来什么问题呢？新同学并不总是知道对通用逻辑的改动或获取来自该类。显然，制定项目规范并不是好的idea。我们又闻到了代码即将腐败的味道。

事实上，你可能意识到问题之所在。在解决现实问题时，我们会将问题映射到脑海中的概念模型，在模型中解决问题，再将解决方案转换为实际的代码。上述问题在于我们解决了设计到代码之间的重构，但提炼出来的设计模型，并不具有实际的业务含义，这就导致在开发新需求时，其他同学并不能很自然地将业务问题映射到该设计模型。设计似乎变成了重构者的自娱自乐，代码继续腐败，重新重构……无休止的循环。

用DDD则可以很好地解决领域模型到设计模型的同步、演化，最后再将反映了领域的设计模型转为实际的代码。

注：模型是我们解决实际问题所抽象出来的概念模型，领域模型则表达与业务相关的事实；设计模型则描述了所要构建的系统。

### **贫血症和失忆症**

贫血领域对象

贫血领域对象（Anemic Domain Object）是指仅用作数据载体，而没有行为和动作的领域对象。

在我们习惯了J2EE的开发模式后，Action/Service/DAO这种分层模式，会很自然地写出过程式代码，而学到的很多关于OO理论的也毫无用武之地。使用这种开发方式，对象只是数据的载体，没有行为。以数据为中心，以数据库ER设计作驱动。分层架构在这种开发模式下，可以理解为是对数据移动、处理和实现的过程。

以笔者最近开发的系统抽奖平台为例：





- 场景需求



奖池里配置了很多奖项，我们需要按运营预先配置的概率抽中一个奖项。
实现非常简单，生成一个随机数，匹配符合该随机数生成概率的奖项即可。





- 贫血模型实现方案



先设计奖池和奖项的库表配置。

![591175fe5c72108470f4d9bcbb21934b79b30e48](https://yqfile.alicdn.com/591175fe5c72108470f4d9bcbb21934b79b30e48.png)

- 设计AwardPool和Award两个对象，只有简单的get和set属性的方法



```none
class AwardPool { int awardPoolId; List<Award> awards; 



 



public List<Award> getAwards() { 



 



return awards; 



 



} 



 



public void setAwards(List<Award> awards) {



 



this.awards = awards;



 



} 



 



......}



 



class Award { int awardId; int probability;//概率 



 



......}
```



- Service代码实现

设计一个LotteryService，在其中的drawLottery()方法写服务逻辑



```none
AwardPool awardPool = awardPoolDao.getAwardPool(poolId);//sql查询，将数据映射到AwardPool对象



 



for (Award award : awardPool.getAwards()) { 



 



//寻找到符合award.getProbability()概率的award



 



}
```



- 按照我们通常思路实现，可以发现：在业务领域里非常重要的抽奖，我的业务逻辑都是写在Service中的，Award充其量只是个数据载体，没有任何行为。简单的业务系统采用这种贫血模型和过程化设计是没有问题的，但在业务逻辑复杂了，业务逻辑、状态会散落到在大量方法中，原本的代码意图会渐渐不明确，我们将这种情况称为由贫血症引起的失忆症。

更好的是采用领域模型的开发方式，将数据和行为封装在一起，并与现实世界中的业务对象相映射。各类具备明确的职责划分，将领域逻辑分散到领域对象中。继续举我们上述抽奖的例子，使用概率选择对应的奖品就应当放到AwardPool类中。

**为什么选择DDD**

### **软件系统复杂性应对**

解决复杂和大规模软件的武器可以被粗略地归为三类：抽象、分治和知识。

分治 把问题空间分割为规模更小且易于处理的若干子问题。分割后的问题需要足够小，以便一个人单枪匹马就能够解决他们；其次，必须考虑如何将分割后的各个部分装配为整体。分割得越合理越易于理解，在装配成整体时，所需跟踪的细节也就越少。即更容易设计各部分的协作方式。评判什么是分治得好，即高内聚低耦合。

抽象 使用抽象能够精简问题空间，而且问题越小越容易理解。举个例子，从北京到上海出差，可以先理解为使用交通工具前往，但不需要一开始就想清楚到底是高铁还是飞机，以及乘坐他们需要注意什么。

知识 顾名思义，DDD可以认为是知识的一种。

DDD提供了这样的知识手段，让我们知道如何抽象出限界上下文以及如何去分治。

### **与微服务架构相得益彰**

微服务架构众所周知，此处不做赘述。我们创建微服务时，需要创建一个高内聚、低耦合的微服务。而DDD中的限界上下文则完美匹配微服务要求，可以将该限界上下文理解为一个微服务进程。

上述是从更直观的角度来描述两者的相似处。

在系统复杂之后，我们都需要用分治来拆解问题。一般有两种方式，技术维度和业务维度。技术维度是类似MVC这样，业务维度则是指按业务领域来划分系统。

微服务架构更强调从业务维度去做分治来应对系统复杂度，而DDD也是同样的着重业务视角。

如果两者在追求的目标（业务维度）达到了上下文的统一，那么在具体做法上有什么联系和不同呢？

我们将架构设计活动精简为以下三个层面：

- 业务架构——根据业务需求设计业务模块及其关系
- 系统架构——设计系统和子系统的模块
- 技术架构——决定采用的技术及框架

以上三种活动在实际开发中是有先后顺序的，但不一定孰先孰后。在我们解决常规套路问题时，我们会很自然地往熟悉的分层架构套（先确定系统架构），或者用PHP开发很快（先确定技术架构），在业务不复杂时，这样是合理的。

跳过业务架构设计出来的架构关注点不在业务响应上，可能就是个大泥球，在面临需求迭代或响应市场变化时就很痛苦。

DDD的核心诉求就是将业务架构映射到系统架构上，在响应业务变化调整业务架构时，也随之变化系统架构。而微服务追求业务层面的复用，设计出来的系统架构和业务一致；在技术架构上则系统模块之间充分解耦，可以自由地选择合适的技术架构，去中心化地治理技术和数据。

可以参见下图来更好地理解双方之间的协作关系：

![ef7f2dcde8de45ef0732ef787f21a2e4d17fb7e5](https://yqfile.alicdn.com/ef7f2dcde8de45ef0732ef787f21a2e4d17fb7e5.png)

**如何实践DDD**

我们将通过上文提到的抽奖平台，来详细介绍我们如何通过DDD来解构一个中型的基于微服务架构的系统，从而做到系统的高内聚、低耦合。

首先看下抽奖系统的大致需求：

运营——可以配置一个抽奖活动，该活动面向一个特定的用户群体，并针对一个用户群体发放一批不同类型的奖品（优惠券，激活码，实物奖品等）。

用户-通过活动页面参与不同类型的抽奖活动。

设计领域模型的一般步骤如下：

1. 根据需求划分出初步的领域和限界上下文，以及上下文之间的关系；
2. 进一步分析每个上下文内部，识别出哪些是实体，哪些是值对象；
3. 对实体、值对象进行关联和聚合，划分出聚合的范畴和聚合根；
4. 为聚合根设计仓储，并思考实体或值对象的创建方式；
5. 在工程中实践领域模型，并在实践中检验模型的合理性，倒推模型中不足的地方并重构。

### **战略建模**

战略和战术设计是站在DDD的角度进行划分。战略设计侧重于高层次、宏观上去划分和集成限界上下文，而战术设计则关注更具体使用建模工具来细化上下文。

### **领域**

现实世界中，领域包含了问题域和解系统。一般认为软件是对现实世界的部分模拟。在DDD中，解系统可以映射为一个个限界上下文，限界上下文就是软件对于问题域的一个特定的、有限的解决方案。

### **限界上下文**



**限界上下文**



**一个由显示边界限定的特定职责。领域模型便存在于这个边界之内。在边界内，每一个模型概念，包括它的属性和操作，都具有特殊的含义。**



一个给定的业务领域会包含多个限界上下文，想与一个限界上下文沟通，则需要通过显示边界进行通信。系统通过确定的限界上下文来进行解耦，而每一个上下文内部紧密组织，职责明确，具有较高的内聚性。

一个很形象的隐喻：细胞质所以能够存在，是因为细胞膜限定了什么在细胞内，什么在细胞外，并且确定了什么物质可以通过细胞膜。

### **划分限界上下文**

划分限界上下文，不管是Eric Evans还是Vaughn Vernon，在他们的大作里都没有怎么提及。

显然我们不应该按技术架构或者开发任务来创建限界上下文，应该按照语义的边界来考虑。

我们的实践是，考虑产品所讲的通用语言，从中提取一些术语称之为概念对象，寻找对象之间的联系；或者从需求里提取一些动词，观察动词和对象之间的关系；我们将紧耦合的各自圈在一起，观察他们内在的联系，从而形成对应的界限上下文。形成之后，我们可以尝试用语言来描述下界限上下文的职责，看它是否清晰、准确、简洁和完整。简言之，限界上下文应该从需求出发，按领域划分。

前文提到，我们的用户划分为运营和用户。其中，运营对抽奖活动的配置十分复杂但相对低频。用户对这些抽奖活动配置的使用是高频次且无感知的。根据这样的业务特点，我们首先将抽奖平台划分为C端抽奖和M端抽奖管理平台两个子域，让两者完全解耦。

![20139dd6a9ad74cf0f361d0d3a5f4530ac5c6497](https://yqfile.alicdn.com/20139dd6a9ad74cf0f361d0d3a5f4530ac5c6497.png)

在确认了M端领域和C端的限界上下文后，我们再对各自上下文内部进行限界上下文的划分。下面我们用C端进行举例。

产品的需求概述如下：

```
1.



 抽奖活动有活动限制，例如用户的抽奖次数限制，抽奖的开始和结束的时间等；



2. 一个抽奖活动包含多个奖品，可以针对一个或多个用户群体；



3. 奖品有自身的奖品配置，例如库存量，被抽中的概率等，最多被一个用户抽中的次数等等；



4. 用户群体有多种区别方式，如按照用户所在城市区分，按照新老客区分等；



5. 活动具有风控配置，能够限制用户参与抽奖的频率。
```

根据产品的需求，我们提取了一些关键性的概念作为子域，形成我们的限界上下文。

![07834d47071b0e49c42fe1725e710fa230ea9def](https://yqfile.alicdn.com/07834d47071b0e49c42fe1725e710fa230ea9def.png)

首先，抽奖上下文作为整个领域的核心，承担着用户抽奖的核心业务，抽奖中包含了奖品和用户群体的概念。

- 在设计初期，我们曾经考虑划分出抽奖和发奖两个领域，前者负责选奖，后者负责将选中的奖品发放出去。但在实际开发过程中，我们发现这两部分的逻辑紧密连接，难以拆分。并且单纯的发奖逻辑足够简单，仅仅是调用第三方服务进行发奖，不足以独立出来成为一个领域。

对于活动的限制，我们定义了活动准入的通用语言，将活动开始/结束时间，活动可参与次数等限制条件都收拢到活动准入上下文中。

对于抽奖的奖品库存量，由于库存的行为与奖品本身相对解耦，库存关注点更多是库存内容的核销，且库存本身具备通用性，可以被奖品之外的内容使用，因此我们定义了独立的库存上下文。

由于C端存在一些刷单行为，我们根据产品需求定义了风控上下文，用于对活动进行风控。

最后，活动准入、风控、抽奖等领域都涉及到一些次数的限制，因此我们定义了计数上下文。

可以看到，通过DDD的限界上下文划分，我们界定出抽奖、活动准入、风控、计数、库存等五个上下文，每个上下文在系统中都高度内聚。

### **上下文映射图**

在进行上下文划分之后，我们还需要进一步梳理上下文之间的关系。

康威（梅尔·康威）定律

任何组织在设计一套系统（广义概念上的系统）时，所交付的设计方案在结构上都与该组织的沟通结构保持一致。

康威定律告诉我们，系统结构应尽量的与组织结构保持一致。这里，我们认为团队结构（无论是内部组织还是团队间组织）就是组织结构，限界上下文就是系统的业务结构。因此，团队结构应该和限界上下文保持一致。

梳理清楚上下文之间的关系，从团队内部的关系来看，有如下好处：

1. 任务更好拆分，一个开发人员可以全身心的投入到相关的一个单独的上下文中；
2. 沟通更加顺畅，一个上下文可以明确自己对其他上下文的依赖关系，从而使得团队内开发直接更好的对接。

从团队间的关系来看，明确的上下文关系能够带来如下帮助：

1. 每个团队在它的上下文中能够更加明确自己领域内的概念，因为上下文是领域的解系统；
2. 对于限界上下文之间发生交互，团队与上下文的一致性，能够保证我们明确对接的团队和依赖的上下游。

限界上下文之间的映射关系

- 合作关系（Partnership）：两个上下文紧密合作的关系，一荣俱荣，一损俱损。
- 共享内核（Shared Kernel）：两个上下文依赖部分共享的模型。
- 客户方-供应方开发（Customer-Supplier Development）：上下文之间有组织的上下游依赖。
- 遵奉者（Conformist）：下游上下文只能盲目依赖上游上下文。
- 防腐层（Anticorruption Layer）：一个上下文通过一些适配和转换与另一个上下文交互。
- 开放主机服务（Open Host Service）：定义一种协议来让其他上下文来对本上下文进行访问。
- 发布语言（Published Language）：通常与OHS一起使用，用于定义开放主机的协议。
- 大泥球（Big Ball of Mud）：混杂在一起的上下文关系，边界不清晰。
- 另谋他路（SeparateWay）：两个完全没有任何联系的上下文。

上文定义了上下文映射间的关系，经过我们的反复斟酌，抽奖平台上下文的映射关系图如下：

![932e1657eca020527878d72754a21a99be9bea81](https://yqfile.alicdn.com/932e1657eca020527878d72754a21a99be9bea81.png)

由于抽奖，风控，活动准入，库存，计数五个上下文都处在抽奖领域的内部，所以它们之间符合“一荣俱荣，一损俱损”的合作关系（PartnerShip，简称PS）。

同时，抽奖上下文在进行发券动作时，会依赖券码、平台券、外卖券三个上下文。抽奖上下文通过防腐层（Anticorruption Layer，ACL）对三个上下文进行了隔离，而三个券上下文通过开放主机服务（Open Host Service）作为发布语言（Published Language）对抽奖上下文提供访问机制。

通过上下文映射关系，我们明确的限制了限界上下文的耦合性，即在抽奖平台中，无论是上下文内部交互（合作关系）还是与外部上下文交互（防腐层），耦合度都限定在数据耦合（Data Coupling）的层级。

### **战术建模——细化上下文**

梳理清楚上下文之间的关系后，我们需要从战术层面上剖析上下文内部的组织关系。首先看下DDD中的一些定义。

实体

当一个对象由其标识（而不是属性）区分时，这种对象称为实体（Entity）。

例：最简单的，公安系统的身份信息录入，对于人的模拟，即认为是实体，因为每个人是独一无二的，且其具有唯一标识（如公安系统分发的身份证号码）。

在实践上建议将属性的验证放到实体中。

值对象

当一个对象用于对事务进行描述而没有唯一标识时，它被称作值对象（Value Object）。

例：比如颜色信息，我们只需要知道{"name":"黑色"，"css":"#000000"}这样的值信息就能够满足要求了，这避免了我们对标识追踪带来的系统复杂性。

值对象很重要，在习惯了使用数据库的数据建模后，很容易将所有对象看作实体。使用值对象，可以更好地做系统优化、精简设计。

它具有不变性、相等性和可替换性。

在实践中，需要保证值对象创建后就不能被修改，即不允许外部再修改其属性。在不同上下文集成时，会出现模型概念的公用，如商品模型会存在于电商的各个上下文中。在订单上下文中如果你只关注下单时商品信息快照，那么将商品对象视为值对象是很好的选择。

聚合根

Aggregate(聚合）是一组相关对象的集合，作为一个整体被外界访问，聚合根（Aggregate Root）是这个聚合的根节点。

聚合是一个非常重要的概念，核心领域往往都需要用聚合来表达。其次，聚合在技术上有非常高的价值，可以指导详细设计。

聚合由根实体，值对象和实体组成。

如何创建好的聚合？

- 边界内的内容具有一致性：在一个事务中只修改一个聚合实例。如果你发现边界内很难接受强一致，不管是出于性能或产品需求的考虑，应该考虑剥离出独立的聚合，采用最终一致的方式。
- 设计小聚合：大部分的聚合都可以只包含根实体，而无需包含其他实体。即使一定要包含，可以考虑将其创建为值对象。
- 通过唯一标识来引用其他聚合或实体：当存在对象之间的关联时，建议引用其唯一标识而非引用其整体对象。如果是外部上下文中的实体，引用其唯一标识或将需要的属性构造值对象。

如果聚合创建复杂，推荐使用工厂方法来屏蔽内部复杂的创建逻辑。

聚合内部多个组成对象的关系可以用来指导数据库创建，但不可避免存在一定的抗阻。如聚合中存在List<值对象>，那么在数据库中建立1:N的关联需要将值对象单独建表，此时是有ID的，建议不要将该ID暴露到资源库外部，对外隐蔽。

领域服务

一些重要的领域行为或操作，可以归类为领域服务。它既不是实体，也不是值对象的范畴。

当我们采用了微服务架构风格，一切领域逻辑的对外暴露均需要通过领域服务来进行。如原本由聚合根暴露的业务逻辑也需要依托于领域服务。

领域事件

领域事件是对领域内发生的活动进行的建模。

抽奖平台的核心上下文是抽奖上下文，接下来介绍下我们对抽奖上下文的建模。

![d89c5be90ff70f76619d1fd5bfffab4b62f5b6f8](https://yqfile.alicdn.com/d89c5be90ff70f76619d1fd5bfffab4b62f5b6f8.png)

在抽奖上下文中，我们通过抽奖(DrawLottery)这个聚合根来控制抽奖行为，可以看到，一个抽奖包括了抽奖ID（LotteryId）以及多个奖池（AwardPool），而一个奖池针对一个特定的用户群体（UserGroup）设置了多个奖品（Award）。

另外，在抽奖领域中，我们还会使用抽奖结果（SendResult）作为输出信息，使用用户领奖记录（UserLotteryLog）作为领奖凭据和存根。

谨慎使用值对象

在实践中，我们发现虽然一些领域对象符合值对象的概念，但是随着业务的变动，很多原有的定义会发生变更，值对象可能需要在业务意义具有唯一标识，而对这类值对象的重构往往需要较高成本。因此在特定的情况下，我们也要根据实际情况来权衡领域对象的选型。

### **DDD工程实现**

在对上下文进行细化后，我们开始在工程中真正落地DDD。

### **模块**

模块（Module）是DDD中明确提到的一种控制限界上下文的手段，在我们的工程中，一般尽量用一个模块来表示一个领域的限界上下文。

如代码中所示，一般的工程中包的组织方式为{com.公司名.组织架构.业务.上下文.*}，这样的组织结构能够明确的将一个上下文限定在包的内部。

![942c73282a50a0ec227408c90b6f021b89066b56](https://yqfile.alicdn.com/942c73282a50a0ec227408c90b6f021b89066b56.png)

代码演示1 模块的组织

对于模块内的组织结构，一般情况下我们是按照领域对象、领域服务、领域资源库、防腐层等组织方式定义的。

![942c73282a50a0ec227408c90b6f021b89066b56](https://yqfile.alicdn.com/942c73282a50a0ec227408c90b6f021b89066b56.png)

代码演示2 模块的组织

每个模块的具体实现，我们将在下文中展开。

### **领域对象**

前文提到，领域驱动要解决的一个重要的问题，就是解决对象的贫血问题。这里我们用之前定义的抽奖（DrawLottery）聚合根和奖池（AwardPool）值对象来具体说明。

抽奖聚合根持有了抽奖活动的id和该活动下的所有可用奖池列表，它的一个最主要的领域功能就是根据一个抽奖发生场景（DrawLotteryContext），选择出一个适配的奖池，即chooseAwardPool方法。

chooseAwardPool的逻辑是这样的：DrawLotteryContext会带有用户抽奖时的场景信息（抽奖得分或抽奖时所在的城市），DrawLottery会根据这个场景信息，匹配一个可以给用户发奖的AwardPool。

![681201fb8d8ea75196f9e637030058071dbda661](https://yqfile.alicdn.com/681201fb8d8ea75196f9e637030058071dbda661.png)

代码演示3 DrawLottery

在匹配到一个具体的奖池之后，需要确定最后给用户的奖品是什么。

这部分的领域功能在AwardPool内。

![a37ac9b9d104d4cdd56114c759fd45b6d2d219bb](https://yqfile.alicdn.com/a37ac9b9d104d4cdd56114c759fd45b6d2d219bb.png)

代码演示4 AwardPool

与以往的仅有getter、setter的业务对象不同，领域对象具有了行为，对象更加丰满。同时，比起将这些逻辑写在服务内（例如**Service），领域功能的内聚性更强，职责更加明确。

### **资源库**

领域对象需要资源存储，存储的手段可以是多样化的，常见的无非是数据库，分布式缓存，本地缓存等。资源库（Repository）的作用，就是对领域的存储和访问进行统一管理的对象。在抽奖平台中，我们是通过如下的方式组织资源库的。

![bf1007de90d301d57e451c8125fcd82bb26d2b99](https://yqfile.alicdn.com/bf1007de90d301d57e451c8125fcd82bb26d2b99.png)
代码演示5 Repository组织结构

资源库对外的整体访问由Repository提供，它聚合了各个资源库的数据信息，同时也承担了资源存储的逻辑（例如缓存更新机制等）。

在抽奖资源库中，我们屏蔽了对底层奖池和奖品的直接访问，而是仅对抽奖的聚合根进行资源管理。代码示例中展示了抽奖资源获取的方法（最常见的Cache Aside Pattern）。

比起以往将资源管理放在服务中的做法，由资源库对资源进行管理，职责更加明确，代码的可读性和可维护性也更强。

![7e46fa00df645305f5b3bc8a6bf35a625cdf93e5](https://yqfile.alicdn.com/7e46fa00df645305f5b3bc8a6bf35a625cdf93e5.png)

代码演示6 DrawLotteryRepository

### **防腐层**

亦称适配层。在一个上下文中，有时需要对外部上下文进行访问，通常会引入防腐层的概念来对外部上下文的访问进行一次转义。

有以下几种情况会考虑引入防腐层：

- 需要将外部上下文中的模型翻译成本上下文理解的模型。
- 不同上下文之间的团队协作关系，如果是供奉者关系，建议引入防腐层，避免外部上下文变化对本上下文的侵蚀。
- 该访问本上下文使用广泛，为了避免改动影响范围过大。

如果内部多个上下文对外部上下文需要访问，那么可以考虑将其放到通用上下文中。

在抽奖平台中，我们定义了用户城市信息防腐层

(UserCityInfoFacade)，用于外部的用户城市信息上下文（微服务架构下表现为用户城市信息服务）。

以用户信息防腐层举例，它以抽奖请求参数(LotteryContext)为入参，以城市信息(MtCityInfo)为输出。

![bf14b97b5e8712f0dc4fb7663bad2eb3fa31dc99](https://yqfile.alicdn.com/bf14b97b5e8712f0dc4fb7663bad2eb3fa31dc99.png)
代码演示7 UserCityInfoFacade

### **领域服务**

上文中，我们将领域行为封装到领域对象中，将资源管理行为封装到资源库中，将外部上下文的交互行为封装到防腐层中。此时，我们再回过头来看领域服务时，能够发现领域服务本身所承载的职责也就更加清晰了，即就是通过串联领域对象、资源库和防腐层等一系列领域内的对象的行为，对其他上下文提供交互的接口。

我们以抽奖服务为例（issueLottery），可以看到在省略了一些防御性逻辑（异常处理，空值判断等）后，领域服务的逻辑已经足够清晰明了。

![657507197d14f62e09dec86e28cc1346ebef408c](https://yqfile.alicdn.com/657507197d14f62e09dec86e28cc1346ebef408c.png)
代码演示8 LotteryService

### **数据流转**

![c53e996b88897f265f379f6bbfff2af4911553ec](https://yqfile.alicdn.com/c53e996b88897f265f379f6bbfff2af4911553ec.png)



在抽奖平台的实践中，我们的数据流转如上图所示。



首先领域的开放服务通过信息传输对象（DTO）来完成与外界的数据交互；在领域内部，我们通过领域对象（DO）作为领域内部的数据和行为载体；在资源库内部，我们沿袭了原有的数据库持久化对象（PO）进行数据库资源的交互。同时，DTO与DO的转换发生在领域服务内，DO与PO的转换发生在资源库内。

与以往的业务服务相比，当前的编码规范可能多造成了一次数据转换，但每种数据对象职责明确，数据流转更加清晰。

#### **上下文集成**

通常集成上下文的手段有多种，常见的手段包括开放领域服务接口、开放HTTP服务以及消息发布-订阅机制。

在抽奖系统中，我们使用的是开放服务接口进行交互的。最明显的体现是计数上下文，它作为一个通用上下文，对抽奖、风控、活动准入等上下文都提供了访问接口。

同时，如果在一个上下文对另一个上下文进行集成时，若需要一定的隔离和适配，可以引入防腐层的概念。这一部分的示例可以参考前文的防腐层代码示例。

#### **分离领域**

接下来讲解在实施领域模型的过程中，如何应用到系统架构中。

我们采用的微服务架构风格，与Vernon在《实现领域驱动设计》并不太一致，更具体差异可阅读他的书体会。

如果我们维护一个从前到后的应用系统：

下图中领域服务是使用微服务技术剥离开来，独立部署，对外暴露的只能是服务接口，领域对外暴露的业务逻辑只能依托于领域服务。而在Vernon著作中，并未假定微服务架构风格，因此领域层暴露的除了领域服务外，还有聚合、实体和值对象等。此时的应用服务层是比较简单的，获取来自接口层的请求参数，调度多个领域服务以实现界面层功能。

![31f7e7ef99bfbaa6a2150e399ef7e5eea3ba659d](https://yqfile.alicdn.com/31f7e7ef99bfbaa6a2150e399ef7e5eea3ba659d.png)

随着业务发展，业务系统快速膨胀，我们的系统属于核心时：

应用服务虽然没有领域逻辑，但涉及到了对多个领域服务的编排。当业务规模庞大到一定程度，编排本身就富含了业务逻辑（除此之外，应用服务在稳定性、性能上所做的措施也希望统一起来，而非散落各处），那么此时应用服务对于外部来说是一个领域服务，整体看起来则是一个独立的限界上下文。

此时应用服务对内还属于应用服务，对外已是领域服务的概念，需要将其暴露为微服务。

![54f3c73a66881eddca18d32b3da85c067eb9aeaf](https://yqfile.alicdn.com/54f3c73a66881eddca18d32b3da85c067eb9aeaf.png)

注：具体的架构实践可按照团队和业务的实际情况来，此处仅为作者自身的业务实践。除分层架构外，如CQRS架构也是不错的选择

以下是一个示例。我们定义了抽奖、活动准入、风险控制等多个领域服务。在本系统中，我们需要集成多个领域服务，为客户端提供一套功能完备的抽奖应用服务。这个应用服务的组织如下：

![92dd50dffebdf40d73507eebe9ae32ba7642395b](https://yqfile.alicdn.com/92dd50dffebdf40d73507eebe9ae32ba7642395b.png)

代码演示9 LotteryApplicationService

**结语**

在本文中，我们采用了分治的思想，从抽象到具体阐述了DDD在互联网真实业务系统中的实践。通过领域驱动设计这个强大的武器，我们将系统解构的更加合理。

但值得注意的是，如果你面临的系统很简单或者做一些SmartUI之类，那么你不一定需要DDD。尽管本文对贫血模型、演进式设计提出了些许看法，但它们在特定范围和具体场景下会更高效。读者需要针对自己的实际情况，做一定取舍，适合自己的才是最好的。

本篇通过DDD来讲述软件设计的术与器，本质是为了高内聚低耦合，紧靠本质，按自己的理解和团队情况来实践DDD即可。

另外，关于DDD在迭代过程中模型腐化的相关问题，本文中没有提及，将在后续的文章中论述，敬请期待。

鉴于作者经验有限，我们对领域驱动的理解难免会有不足之处，欢迎大家共同探讨，共同提高。



- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarThumbUp.png)点赞1
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarComment.png)评论](https://blog.csdn.net/qq_36510261/article/details/78923338#commentBox)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarShare.png)分享](javascript:;)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)收藏2](javascript:;)
- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarReport.png)举报
- [关注](javascript:;)
- 一键三连

[*领域**驱动设计*在*美团**点评**业务系统*的*实践*](https://download.csdn.net/download/weixin_38562085/14939812)

01-27

[至少30年以前，一些软件设计人员就已经意识到*领域*建模和设计的重要性，并形成一种思潮，Eric Evans将其定义为*领域**驱动设计*（Domain-DrivenDesign，简称*DDD*）。在互联网开发“小步快跑，迭代试错”的大环境下，*DDD*似乎是一种比较“古老而缓慢”的思想。然而，由于互联网公司也逐渐深入实体经济，业务日益复杂，我们在开发中也越来越多地遇到传统行业软件开发中所面临的问题。本文就先来讲一下这些问题，然后再尝试在*实践*中用*DDD*的思想来解决这些问题。 业务初期，我们的功能](https://download.csdn.net/download/weixin_38562085/14939812)

[实现*领域**驱动设计*（*DDD*之父作序力荐 让*DDD*思想真正落地的首创巨著）](http://download.csdn.net/download/liweifeng1988/9440554)

02-23

[*领域**驱动设计*（*DDD*）是教我们如何做好软件的，同时也是教我们如何更好地使用面向对象技术的。它为我们提供了设计软件的全新视角，同时也给开发者留下了一大难题：如何将*领域**驱动设计*付诸*实践*？Vaughn Vernon 的这本《实现*领域**驱动设计*》为我们给出了全面的解答。 　　《实现*领域**驱动设计*》分别从战略和战术层面详尽地讨论了如何实现*DDD*，其中包含了大量的最佳*实践*、设计准则和对一些问题的折中性讨论。《实现*领域**驱动设计*》共分为14 章，在*DDD* 战略部分，《实现*领域**驱动设计*》向我们讲解了*领域*、限界上下文、上下文映射图和架构等内容，战术部分包括实体、值对象、*领域*服务、*领域*事件、聚合和资源库等内容。一个虚构的案例研究贯穿全书，这对于实例讲解*DDD* 实现来说非常有用。 　　《实现*领域**驱动设计*》在*DDD* 的思想和实现之间建立起了一座桥梁，架构师和程序员均可阅读，同时也可以作为一本*DDD* 参考书。](http://download.csdn.net/download/liweifeng1988/9440554)





[![img](https://profile.csdnimg.cn/D/B/6/3_yangjava2014)](https://blog.csdn.net/yangjava2014)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/commentFlag@2x.png)

![表情包](https://csdnimg.cn/release/blogv2/dist/pc/img/emoticon.png)

相关推荐

[*领域**驱动设计*在*美团**点评**业务系统*的*实践*_王卫东 博客](https://blog.csdn.net/wwd0501/article/details/108903744)

4-23

[*领域**驱动设计*在*美团**点评**业务系统*的*实践* 至少30年以前,一些软件设计人员就已经意识到*领域*建模和设计的重要性,并形成一种思潮,Eric Evans将其定义为*领域**驱动设计*(Domain-Driven Design,简称*DDD*)。在互联网开发“小步快跑,迭代试错”的大环境...](https://blog.csdn.net/wwd0501/article/details/108903744)

[*DDD*(*领域**驱动设计*)_王卫东 博客](https://blog.csdn.net/wwd0501/article/details/95062535/)

5-4

[*领域**驱动设计*(简称 *ddd*)概念来源于2004年著名建模专家eric evans发表的他最具影响力的书籍:《domain-driven design –tackling complexity in the heart of software》(中文译名:*领域**驱动设计*—软件核心复杂性应对之道)一书。,书中提出了...](https://blog.csdn.net/wwd0501/article/details/95062535/)

[实现*领域**驱动设计*完整版](https://download.csdn.net/download/wupitt/10154633)

12-11

[《实现*领域**驱动设计*》分别从战略和战术层面详尽地讨论了如何实现*DDD*，其中包含了大量的最佳*实践*、设计准则](https://download.csdn.net/download/wupitt/10154633)

[浅谈*领域**驱动设计*（*DDD*:Domain-Driven Design）](https://blog.csdn.net/xuxuerlai/article/details/81318565)

[xuxuerlai的博客](https://blog.csdn.net/xuxuerlai)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[来源：《*领域**驱动设计*》是2010年04月人民邮电出版社出版的图书，作者是Eric Evans。本书介绍了面向对象开发人员、系统分析人员合理地组织工作，彼此协作，有条不紊地进行复杂系统的开发，帮助建立丰富而实用的*领域*模型。 博主也只是刚开始接触这个，理解的也不是很透彻，如有不到位的地方，还请各位海涵！！！ 在这里，我是把它当成一种设计思想，或者说是一种设计模式，比较适用于帮助开发人员拓展思维。 ...](https://blog.csdn.net/xuxuerlai/article/details/81318565)

[*领域**驱动设计*在互联网业务开发中的*实践*_*美团*技术团队](https://blog.csdn.net/meituantech/article/details/80062248)

4-12

[至少30年以前,一些软件设计人员就已经意识到*领域*建模和设计的重要性,并形成一种思潮,Eric Evans将其定义为*领域**驱动设计*(Domain-Driven Design,简称*DDD*)。在互联网开发“小步快跑,迭代试错”的大环境下,*DDD*似乎是一种比较“古老而缓慢”的...](https://blog.csdn.net/meituantech/article/details/80062248)

[【你问我答】*DDD*(*领域**驱动设计*)*实践*中遇到问题了?尽管...](https://blog.csdn.net/MeituanTech/article/details/98744756)

4-14

[这一次,我们有业务线技术团队来解你疑惑~ 参与方式 在“留言”区提问,也可以加入我们的微信群参与更多讨论。 加群方式:长按识别二维码,添加*美团**点评*小助手(MTDPtech),回复关键词【*DDD*】,加入*领域**驱动设计*爱好者微信群~ ...](https://blog.csdn.net/MeituanTech/article/details/98744756)

[*领域**驱动设计*之我见-*领域*建模](https://blog.csdn.net/tidu2chengfo/article/details/81106194)

[乔布斯基的博客](https://blog.csdn.net/tidu2chengfo)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 2153

[前面两节絮絮叨叨重点讲了一句话：*领域**驱动设计*的核心在*领域*模型，*领域*建模核心在精通*领域*业务。那么该如何做好*领域*建模呢？需要精通的能力都没有捷径可走，但是也不是没有方法可循，下文就*领域*业务和建模两方面做一下讲解。 做好*领域*建模，首先要做的工作是要精通*领域*业务知识，那么*领域*业务知识从哪里来呢？前面章节讲了从事软件研发的工程师并非全部来自于科班出身的学生，有些计算机相关学科天然带着*领域*业务知识，例如GI...](https://blog.csdn.net/tidu2chengfo/article/details/81106194)

[5 年 Java 经验，字节、*美团*、快手核心部门面试总结（真题解析）](https://joonwhee.blog.csdn.net/article/details/109588711)

[程序员囧辉](https://blog.csdn.net/v123411739)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 7272

[卷，继续卷](https://joonwhee.blog.csdn.net/article/details/109588711)

[*DDD*专家张逸:《解构*领域**驱动设计*》前言_中生代技术](https://blog.csdn.net/k6T9Q8XKs6iIkZPPIFq/article/details/110251519)

4-13

[*领域*驱动架构系列 Hacker News热文:请停止学习框架,学习*领域**驱动设计*(*DDD*)(获500个点赞) 京东平台研发朱志国:*领域**驱动设计*(*DDD*)理论启示 *DDD*专家张逸:构建*领域**驱动设计*知识体系 *领域**驱动设计*(*DDD*)在*美团**点评**业务系统*的*实践* ...](https://blog.csdn.net/k6T9Q8XKs6iIkZPPIFq/article/details/110251519)

[设计模式在*美团*外卖营销业务中的*实践*_*美团*技术团队](https://blog.csdn.net/MeituanTech/article/details/104991174)

5-5

[关于*DDD*的*实践*,大家可以参考此前*美团*技术团队推出的《*领域**驱动设计*在互联网业务开发中的*实践*》一文。 同时,我们也需要在代码工程中贯彻和实现*领域*模型。因为代码工程是*领域*模型在工程*实践*中的直观体现,也是*领域*模型在技术层面的直接表述。](https://blog.csdn.net/MeituanTech/article/details/104991174)

[*DDD*（*领域**驱动设计*）总结](https://blog.csdn.net/qq773837256/article/details/52023905)

[绯雨倾城](https://blog.csdn.net/qq773837256)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[基本概念： 　　*领域**驱动设计*(简称 *ddd*)概念来源于2004年著名建模专家eric evans发表的他最具影响力的书籍:《domain-driven design –tackling complexity in the heart of software》(中文译名：*领域**驱动设计*—软件核心复杂性应对之道)一书。，书中提出了“*领域**驱动设计*(简称 *ddd*)”的概念。      领](https://blog.csdn.net/qq773837256/article/details/52023905)

[*领域*模型*驱动设计*（Domain Driven Design）入门概述](https://blog.csdn.net/johnstrive/article/details/16805121)

[在路上 >> 关注我的公众号：【架构师也是人】，更多干货，和你分享](https://blog.csdn.net/johnstrive)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 5万+

[软件开发要干什么： 反映真实世界要自动化的业务流程解决现实问题 *领域*Domain Domain特指软件关注的*领域*在不能充分了解业务*领域*的情况下是不可能做出一个好的软件  *领域*建模 *领域*模型*驱动设计* } 分层架构 } 实体 } 值对象 } 服务 } 模块 } 聚合 } 工厂 } 资源库  分层架构：](https://blog.csdn.net/johnstrive/article/details/16805121)

[*美团**点评*技术专家：*DDD*，微服务架构，在收单供应链系统中的*实践*！](https://jsldl.blog.csdn.net/article/details/103082993)

[技术领导力](https://blog.csdn.net/yellowzf3)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 477

[点击“技术领导力”关注∆每天早上8:30推送作者：牟宗彦来源：GIAC大会小编按：本文取自互联网公开材料，感谢*美团**点评*技术专家牟宗彦在GIAC大会上的分享。*DDD*随着...](https://jsldl.blog.csdn.net/article/details/103082993)

[大家一直在谈的*领域**驱动设计*（*DDD*），我们在互联网*业务系统*是这么*实践*的](https://blog.csdn.net/MeituanTech/article/details/98745140)

[美团技术团队](https://blog.csdn.net/MeituanTech)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 3576

[点击上方蓝字订阅，不错过下一篇好文章前言至少30年以前，一些软件设计人员就已经意识到*领域*建模和设计的重要性，并形成一种思潮，Eric Evans将其定义为*领域**驱动设计*（D...](https://blog.csdn.net/MeituanTech/article/details/98745140)

[*领域**驱动设计*在互联网业务开发中的*实践*](https://blog.csdn.net/maoyeqiu/article/details/108804440)

[maoyeqiu的专栏](https://blog.csdn.net/maoyeqiu)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 125

[至少30年以前，一些软件设计人员就已经意识到*领域*建模和设计的重要性，并形成一种思潮，Eric Evans将其定义为*领域**驱动设计*（Domain-Driven Design，简称*DDD*）。在互联网开发“小步快跑，迭代试错”的大环境下，*DDD*似乎是一种比较“古老而缓慢”的思想。然而，由于互联网公司也逐渐深入实体经济，业务日益复杂，我们在开发中也越来越多地遇到传统行业软件开发中所面临的问题。本文就先来讲一下这些问题，然后再尝试在*实践*中用*DDD*的思想来解决这些问题。 过度耦合 业务初期，我们的功能大都非常简单，普](https://blog.csdn.net/maoyeqiu/article/details/108804440)

[*领域**驱动设计*之设计原则篇](https://blog.csdn.net/guzhangyu12345/article/details/96107597)

[简单，坚持](https://blog.csdn.net/guzhangyu12345)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 358

[语言从c的面向过程到java的面向对象，在程序设计、组织的角度来看是在抽象、直观化、便于模块整合上的一次进步。现在的许多通用框架，比如spring、mybatis为应用程序提供了对象的管理以及数据仓库的操作封装；对于一般规模的应用程序来说，业务模型与代码结构设计的重要性尚未凸显出来。但是如果业务复杂、涉及交互模块多，或者随着时间推进，需求、业务逻辑的变更、调整，将带来代码调整、辅助功能...](https://blog.csdn.net/guzhangyu12345/article/details/96107597)

[*DDD*（*领域**驱动设计*）总结](https://blog.csdn.net/woshihyykk/article/details/108538608)

[SToNE的博客](https://blog.csdn.net/woshihyykk)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 505

[*领域**驱动设计*总结基本概念：1．实体(entity)：2．值对象(value object)3．聚合及聚合根(aggregate、aggregate root)：4．工厂(factories)：5．仓储（repositories）：6．服务（services）：7．domain事件8．DTO设计*领域*模型的一般步骤一些思考1. 建立完整自封闭的*领域*模型。2. *领域*服务建模3．*领域*对象、*领域*服务以及repository之间的互相依赖 基本概念： *领域**驱动设计*(简称 *ddd*)概念来源于2004年著名建模专家e](https://blog.csdn.net/woshihyykk/article/details/108538608)

[*领域**驱动设计*在互联网业务如何*实践*？](https://blog.csdn.net/soledadzz/article/details/90623950)

[形势所逼，不进则退](https://blog.csdn.net/soledadzz)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 178

[这篇文章会在关于 *DDD* 的落地*实践*上，给你一些启发。在探讨*领域*驱动战术设计的一些问题时，总会有人纠结：这个*领域*对象应该定义成实体，还是值对象？*领域*服务和应用服务的区别是...](https://blog.csdn.net/soledadzz/article/details/90623950)

[*美团**点评*技术专家：*DDD*，微服务架构，在收单供应链系统中的*实践*！](https://blog.csdn.net/weixin_45727359/article/details/103556141)

[weixin_45727359的博客](https://blog.csdn.net/weixin_45727359)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 170

[小编按：本文取自互联网公开材料，感谢*美团**点评*技术专家牟宗彦在GIAC大会上的分享。*DDD*随着微服务的东西颇有些扶正的意思，但笔者以为*DDD*的精髓在于战略设计。战术设计的部分如果拘泥于方墙...](https://blog.csdn.net/weixin_45727359/article/details/103556141)

[*领域**驱动设计* *DDD* - 桌游](https://blog.csdn.net/weixin_33737774/article/details/91457276)

[weixin_33737774的博客](https://blog.csdn.net/weixin_33737774)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 28

[图片来自 fsharpforfunandprofit.com/*ddd*/ 转载于:https://juejin.im/post/5b7fd9596fb9a019ed7b8998](https://blog.csdn.net/weixin_33737774/article/details/91457276)

[*DDD*（*领域**驱动设计*）的这些问题，你都知道吗？](https://blog.csdn.net/MeituanTech/article/details/98745050)

[美团技术团队](https://blog.csdn.net/MeituanTech)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 2445

[点击上方蓝字可以订阅哦本文中的问题精选自上期【你问我答】——*DDD*（*领域**驱动设计*）专题中读者的提问。【你问我答】是由*美团**点评*技术团队推出的线上问答服务，你在工作学习中遇到...](https://blog.csdn.net/MeituanTech/article/details/98745050)

[离职*美团*，面试了阿里、百度多家互联网公司，熬夜为大家肝出这些](https://blog.csdn.net/Ppikaqiu/article/details/108664602)

[如果频繁就私信一下呢](https://blog.csdn.net/Ppikaqiu)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 143

[前言 由于个人发展的原因和工作上的变动，产生了想出来看看机会的想法。经过了一段时间的准备，3月下旬开始出来面试，面到了 5月下旬，如愿拿到了自己心仪公司的 offer。按照自己的习惯，将这次面试过程中的一些经验总结、心得体会记录下来，自己留个记录，也希望可以帮助到一些同学。 个人情况 坐标北京，15 年本科毕业于普通一本，毕业后就职于一家传统电信公司，17 年后就职于*美团**点评*。 4 年经验应该具备哪些技能 首先，简单的聊一下我认为的 4 年经验左右、优秀的 Java 程序员应该具备的技能有哪些，按](https://blog.csdn.net/Ppikaqiu/article/details/108664602)

[如何系统学习*领域**驱动设计*（*DDD*）？](https://blog.csdn.net/GitChat/article/details/81078607)

[技术杂谈](https://blog.csdn.net/GitChat)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 7143

[作者简介 张逸，曾先后就职于中兴通讯、惠普 GDCC、中软国际、ThoughtWorks 等大型中外企业，任职角色为高级软件工程师、架构师、技术总监、首席咨询师。 精通包括 Java、Scala、Python、C#、JavaScript、Ruby 等多种语言，熟练掌握面向对象思想、测试驱动开发与重构、*领域**驱动设计*、函数式编程、架构、大数据分析、敏捷与过程改进，并致力于大型软件企业...](https://blog.csdn.net/GitChat/article/details/81078607)

©️2020 CSDN 皮肤主题: 书香水墨 设计师:CSDN官方博客 [返回首页](https://blog.csdn.net/)

- [关于我们](https://www.csdn.net/company/index.html#about)
- [招贤纳士](https://www.csdn.net/company/index.html#recruit)
- [广告服务](https://www.csdn.net/company/index.html#advertisement)
- [开发助手](https://plugin.csdn.net/)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/tel.png)400-660-0108
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/email.png)[kefu@csdn.net](mailto:webmaster@csdn.net)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/cs.png)[在线客服](https://csdn.s2.udesk.cn/im_client/?web_plugin_id=29181)
- 工作时间 8:30-22:00

- [公安备案号11010502030143](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010502030143)
- [京ICP备19004658号](http://beian.miit.gov.cn/publish/query/indexFirst.action)
- [京网文〔2020〕1039-165号](https://csdnimg.cn/release/live_fe/culture_license.png)
- [经营性网站备案信息](https://csdnimg.cn/cdn/content-toolbar/csdn-ICP.png)
- [北京互联网违法和不良信息举报中心](http://www.bjjubao.org/)
- [网络110报警服务](http://www.cyberpolice.cn/)
- [中国互联网举报中心](http://www.12377.cn/)
- [家长监护](https://download.csdn.net/index.php/tutelage/)
- [Chrome商店下载](https://chrome.google.com/webstore/detail/csdn开发者助手/kfkdboecolemdjodhmhmcibjocfopejo?hl=zh-CN)
- ©1999-2021北京创新乐知网络技术有限公司
- [版权与免责声明](https://www.csdn.net/company/index.html#statement)
- [版权申诉](https://blog.csdn.net/blogdevteam/article/details/90369522)
- [出版物许可证](https://img-home.csdnimg.cn/images/20210414021151.jpg)
- [营业执照](https://img-home.csdnimg.cn/images/20210414021142.jpg)

[![img](https://profile.csdnimg.cn/A/4/B/3_qq_36510261)](https://blog.csdn.net/qq_36510261)

[Winsdons](https://blog.csdn.net/qq_36510261)

码龄5年[![img](https://csdnimg.cn/identity/nocErtification.png) 暂无认证](https://i.csdn.net/#/uc/profile?utm_source=14998968)







- 99万+

  访问

- [![img](https://csdnimg.cn/identity/blog7.png)](https://blog.csdn.net/blogdevteam/article/details/103478461)

  等级

- 1万+

  积分

- 676

  粉丝

- 102

  获赞

- 36

  评论

- 250

  收藏

![签到新秀](https://csdnimg.cn/medal/qiandao1@240.png)

![新秀勋章](https://csdnimg.cn/medal/xinxiu@240.png)

[私信](https://im.csdn.net/chat/qq_36510261)

关注

![img](https://csdnimg.cn/cdn/content-toolbar/csdn-sou.png?v=1587021042)

### 热门文章

- [Python薪资又涨了，这可咋办啊！ ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 15248](https://blog.csdn.net/qq_36510261/article/details/78676494)
- [2017云栖社区之星评选暨年度颁奖盛典_投票即可参与抽奖 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 14561](https://blog.csdn.net/qq_36510261/article/details/78560787)
- [初学者必读的Linux入门到精通 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 13121](https://blog.csdn.net/qq_36510261/article/details/77894659)
- [域名解析产品——HTTPDNS使用教程 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 11877](https://blog.csdn.net/qq_36510261/article/details/77945089)
- [快速选择合适的机器学习算法 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 11422](https://blog.csdn.net/qq_36510261/article/details/72640023)

### 最新评论

- 教你用Python爬虫股票评论，简单分析股民用户情绪

  [DDLW_PLEH: ](https://blog.csdn.net/life_is_too_long)你好，请问有完整代码嘛

- 从头了解Gradient Boosting算法

  [一舫: ](https://blog.csdn.net/weixin_41997327)博主，图挂了

- 分布式文件系统FastDFS动态扩容

  [weixin_44128224: ](https://blog.csdn.net/weixin_44128224)你抄别人的东西,自己能看懂吗

- 海云数据冯一村：AI能力服务平台如何在公安系统落地

  [qq_25746349: ](https://blog.csdn.net/qq_25746349)本人是这家公司员工，入职时间部门什么的就不说了。从19年11月到现在3月中旬，一分钱工资不发，年会上某领导给员工保证，1月20号肯定发一个月工资，大概率发两个月，并说可以录音。以上是某领导原话。到现在3月了，还是一分钱没看见，虽然是疫情期间，但是，公积金从19年11月之后公司就再也没交过，也就是说，所有员工的公积金都在断交状态，我也是前两天看支付宝查询公积金才发现，不清楚为什么断交这么久没有人发现或者没有人告他？！！！在不发工资这段期间，高层领导开会说的一直都是多少号多少号发工资，我已经记不清骗自己员工多少次了。4、5个月不发工资，我才工作没多久，能有多少积蓄可以扛住，众多员工的基本生活现在都成了问题？现在疫情严重，工作不好找，也不敢轻易离职，真的是哑巴吃黄莲的感觉。更别提那些有房贷车贷信用卡等等要养家户口的人了，好歹我现在单身一个人。但是这样不为员工的生活考虑，不把员工当人的公司有什么可以留恋的？天天吹牛*说上市，上你**！

- spring boot redis分布式锁

  [向佐撞: ](https://blog.csdn.net/sail331x)用Ctrl+A看完你的代码~

### 最新文章

- [新优选商城上线发布会，京庐空间执行总裁赵娅勤女士接受采访！](https://blog.csdn.net/qq_36510261/article/details/86301749)
- [【2018清博盛典暨新媒体大数据峰会】学界、产业、创业者跨界年度观察](https://blog.csdn.net/qq_36510261/article/details/79217738)
- [微信月活9亿的高效运维之路](https://blog.csdn.net/qq_36510261/article/details/79217709)

[2019年1篇](https://blog.csdn.net/qq_36510261/article/month/2019/01)

[2018年125篇](https://blog.csdn.net/qq_36510261/article/month/2018/01)

[2017年624篇](https://blog.csdn.net/qq_36510261/article/month/2017/12)

![img](https://kunyu.csdn.net/1.png?p=57&a=2488&c=0&k=&spm=1001.2101.3001.5001&d=1&t=3&u=e5f73e033c664146a27e70f7ad8a7d51)

### 目录

1. [过度耦合](https://blog.csdn.net/qq_36510261/article/details/78923338#t0)
2. [贫血症和失忆症](https://blog.csdn.net/qq_36510261/article/details/78923338#t1)
3. [软件系统复杂性应对](https://blog.csdn.net/qq_36510261/article/details/78923338#t2)
4. [与微服务架构相得益彰](https://blog.csdn.net/qq_36510261/article/details/78923338#t3)
5. [战略建模](https://blog.csdn.net/qq_36510261/article/details/78923338#t4)
6. [领域](https://blog.csdn.net/qq_36510261/article/details/78923338#t5)
7. [限界上下文](https://blog.csdn.net/qq_36510261/article/details/78923338#t6)
8. [划分限界上下文](https://blog.csdn.net/qq_36510261/article/details/78923338#t7)
9. [上下文映射图](https://blog.csdn.net/qq_36510261/article/details/78923338#t8)
10. [战术建模——细化上下文](https://blog.csdn.net/qq_36510261/article/details/78923338#t9)
11. [DDD工程实现](https://blog.csdn.net/qq_36510261/article/details/78923338#t10)
12. [模块](https://blog.csdn.net/qq_36510261/article/details/78923338#t11)
13. [领域对象](https://blog.csdn.net/qq_36510261/article/details/78923338#t12)
14. [资源库](https://blog.csdn.net/qq_36510261/article/details/78923338#t13)
15. [防腐层](https://blog.csdn.net/qq_36510261/article/details/78923338#t14)
16. [领域服务](https://blog.csdn.net/qq_36510261/article/details/78923338#t15)
17. [数据流转](https://blog.csdn.net/qq_36510261/article/details/78923338#t16)



![img](https://g.csdnimg.cn/side-toolbar/3.0/images/guide.png)![img](https://g.csdnimg.cn/side-toolbar/3.0/images/kefu.png)举报![img](https://g.csdnimg.cn/side-toolbar/3.0/images/fanhuidingbucopy.png)

[![CSDN首页](https://img-home.csdnimg.cn/images/20201124032511.png)](https://www.csdn.net/)

- [首页](https://www.csdn.net/)
- [博客](https://blog.csdn.net/)
- [程序员学院](https://edu.csdn.net/)
- [下载](https://download.csdn.net/)
- [论坛](https://bbs.csdn.net/)
- [问答](https://ask.csdn.net/)
- [代码](https://codechina.csdn.net/?utm_source=csdn_toolbar)
- [直播](https://live.csdn.net/?utm_source=csdn_toolbar)
- [能力认证](https://ac.csdn.net/)
- [高校](https://studentclub.csdn.net/)

 

[![img](https://profile.csdnimg.cn/D/B/6/2_yangjava2014)](https://blog.csdn.net/yangjava2014)

[会员中心](https://mall.csdn.net/vip)

[收藏](https://i.csdn.net/#/user-center/collection-list?type=1)

[动态](https://blog.csdn.net/nav/watchers)

[消息](https://i.csdn.net/#/msg/index)

[创作中心](https://mp.csdn.net/)

# 互联网账户系统如何设计（上篇）

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/reprint.png)

[菜鸟笔记](https://blog.csdn.net/yin767833376) 2019-01-04 11:10:15 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 969 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏 2

分类专栏： [Java开发](https://blog.csdn.net/yin767833376/category_6124656.html) 文章标签： [账户设计](https://so.csdn.net/so/search/s.do?q=账户设计&t=blog&o=vip&s=&l=&f=&viparticle=) [互联网账户系统](https://so.csdn.net/so/search/s.do?q=互联网账户系统&t=blog&o=vip&s=&l=&f=&viparticle=) [互联网账户设计](https://so.csdn.net/so/search/s.do?q=互联网账户设计&t=blog&o=vip&s=&l=&f=&viparticle=) [账户设计演变](https://so.csdn.net/so/search/s.do?q=账户设计演变&t=blog&o=vip&s=&l=&f=&viparticle=)

在很多互联网公司业务发展的早期，业务模式比较单一的情况下，涉及用户账户资金交易相关的逻辑也比较简单，但是随着公司业务模式的不断创新及类型的多元化发展，会渐渐发现现有系统账户逻辑越来越雍肿，不仅难以支持新业务的扩张，对现有业务的支持也适配困难，最终导致新业务系统不得不重新搭建自己的业务账户逻辑，造成重复建设不说，也往往给后续的财务资金核算造成混乱。

 

**以某互联网A租车公司的业务发展路径为例?**

 

**阶段A**

 

A公司在早期开展租车业务时根据用户使用场景规定用户必须在缴纳押金以后才可以租车，并且支持用户进行余额充值，余额可以支付租车费用以及购买相关优惠产品。所以账户资金流是这样子的：

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/l89kosVutonRLMtcWQmkTSZElqUe6CwWQ5DYhp3hbcY5DVaicFJPehUBzVAyTJwp6ibrMaEZO3ibcmOelYunf4VCQ/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

 

通过上述设计基本上就满足了简单的业务需求，用户缴纳押金单独存放在一个押金账户，用户的每次押金冲退都记录账户流水(缴押金记+，退押金记-)；用户余额充值单独存放在一个余额账户，用户的每次余额充值、消费、退款都记录账户流水（余额充值记+、余额消费记-、余额退款记-）。

 

事实上，以上账户逻辑的设计基本已经能满足业务早期的发展的需要了，并且如果账户流水记录完整（例如记录必要的业务类型，如余额购买了一张租车次卡）那么后续也能基本满足早期业务的财务核算需求。遗憾的是，很多公司在类似以上简单账户逻辑的设计上都比较混乱，如有的公司将账户直接绑定在用户信息表上、有些直接更新账户余额，没有完整记录账户流水或账户流水记录业务信息缺乏等，这种情况即使业务没有多元化发展，也很难满足后续业务逻辑的扩展，特别是会造成严重的财务数据核算困难，更不用谈业务多元化发展后如何能够快速支撑了，造成这种问题的原因是多方面的，这里也就不在赘述。

 

下面我们继续看A公司租车业务的发展演进情况！

 

**假设在A公司租车业务发展过程中为了鼓励用户进行余额充值，采用了充值+返现的形式进行活动，如：“充值100赠送20”，此时用户余额账户的总金额应该是120，那么账户逻辑如何支持呢？**

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/l89kosVutomxmRiafRjSMl568WRZMTR76ibNyuzxdeaMichVn3Tby20pvPj07gsYIKYnwicIfMRo5Rfm2xQvAG3QYQ/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

 

这里注意，之所以需要区分余额中真实充值和赠送的原因在于，如果产品逻辑允许余额退款，那么清晰地区分了真实充值和赠送所对应的余额的话，退款逻辑的实现将会比较简单，否则就会变得比较痛苦，并且不清晰的逻辑也会增加造成公司资金损失的风险；另外如果余额返现账户变动流水记录得比较明确的话，对财务后续的收入核算也会方便很多。

 

但是，是否这样就能满足业务需求了呢？

 

显然，这样还不能让逻辑完全运行起来，因为增加了账户相应地交易逻辑与资金逻辑都需要进行相应的改变才行，以上业务场景中原来余额充值只需要调用余额账户记账一次，现在需要根据充返逻辑再调用余额返现账户记账一次；而余额消费则需要根据业务规则进行余额消费记账，假设业务规则为“余额消费优先扣减余额返现账户，再次扣减余额账户”，那么系统交易及记账逻辑如下图所示：

 

![img](https://mmbiz.qpic.cn/mmbiz_png/l89kosVutomxmRiafRjSMl568WRZMTR76L7fbDu1G7rlAuMWxiaYcH5CvU1yOaNlRcl2BdLUxxUybcaBic0jfzV4w/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

 

从逻辑上看业务交易系统需要根据业务规则多次调用余额账户及余额返现账户进行记账，并且需要从流程上保证两个账户记账调用的事务一致性，例如一笔消费订单金额为20元，此时余额账户余额为10元，余额返现账户余额为5元，在优先消费返现账户金额扣款5元后无法再从余额账户消费15元时，交易失败后需要回滚余额返现账户消费逻辑,余额返现账户“交易冲正 +5”。

 

**阶段B**

 

在阶段A中，单个业务会根据不同的产品设计进行账户逻辑的迭代，增加余额充返账户后虽然从账户逻辑层面只是增加了新的资金账户，但是交易逻辑却是进行了较大的变更和调整；如果此时该业务场景又出现新的逻辑变化，例如某一天该租车业务针对某些信用良好的用户进行免押金用车活动，并且支持这类用户在退押金时可以选择将押金的全部或部分金额进行余额充值，那么在流程设计上还会存在账户转账的情况（押金账户->余额账户）。

 

每次产品业务的迭代涉及用户资金逻辑，都不免会影响交易逻辑及账户逻辑本身，但如果业务品类单一，这种迭代及扩展通过硬编码方式多少还能继续支持。但是，如果随着公司业务向多元化发展，问题往往就变得复杂了。

 

**假设A公司在主营租车业务发展态势进入到趋于平缓的阶段后，为了扩展业务范围需要尝试新的业务，例如，某团队提出不能仅仅只搞租车也得弄个网约车平台，那怎么弄？**

 

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/l89kosVutomxmRiafRjSMl568WRZMTR767aLLcQ3fIZ5QHBMM4Te9OTTDF5xxlcB2V0Njl3KIT4CwZMPnzXuPGw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

 

首先肯定是需要先集结队伍啦![img](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/smiley/smiley_?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)。集结完队伍就需要好好分析分析下业务场景了，我们先大概看看一个网约车平台的大概业务场景是什么样子的，其中涉及的资金交易的流程应该如何设计呢？

 

在A公司的租车业务中，从参与角色角度来看涉及的逻辑并不算太复杂，只有“用户、车、租车公司(内部可称租车业务线)”，而从交易场景上看，用户缴纳押金、预存余额及余额消费都只是**单向的C->B**交易模型，如果不考虑公司层在线交易账户体系（这里是指公司层面的交易账户）业务复杂度还不算十分复杂，并且在这种单向业务模式下，没有公司层面的在线交易账户本身并不影响业务流程，收入核算只需要线下计算即可，这也就是为什么前面会特意强调账户流水业务关键信息不能缺失的原因了。

 

而约车业务则是**多向的C->B-C-B**的交易模型，因为从参与角色上看除了普通打车用户外，还有司机、打车平台都会紧紧地参与到整个业务流程中；而从账户资金流上看用户支付的车费不会以C->C的模式直接支付给司机，而是会由打车平台代收，打车平台扣除订单抽成后剩下的车费才会打到司机在打车平台开的账户上，司机才能从个人账户提现。

 

![img](https://mmbiz.qpic.cn/mmbiz_png/l89kosVutomGx6tyMUeRGB2aaQTHaVSHavD6DHbht7QtyythBsnZvTo9riaP3RgfYCLByYiaT3awZTNDMukicMduA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

从上图的分析中我们可以看到，我们需要为普通用户、司机账户、打车平台分别设置必要的账户逻辑，普通用户账户逻辑与之前的租车业务比较类似，用户可以直接支付打车费用、也可以通过余额充值后使用余额支付打车费用；而平台则需要代收用户的打车费用并且需要按照服务规则从代收的打车费用中扣掉部分服务费，然后通过代收/付平台账户将车费实时结算给司机端收款账户，司机通过个人收款账户发起提现后经过结算账户完成提现。资金流如下：

 

![img](https://mmbiz.qpic.cn/mmbiz_png/l89kosVutomGx6tyMUeRGB2aaQTHaVSHS0Jn8DySNs3dI1UnwYhhvZku7wFicX1o4dlpqp1Kt0EUGYXyZXLhRkA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

 

此外，为了满足产品逻辑的扩张，例如要具备冻结司机账户的业务功能，则需要设置冻结账户；平台为了进行市场营销活动，如发放红包则需要设置平台市场营销账户等。

 

**阶段C**

 

业务发展到这个阶段，初步看好像并没有太多的逻辑变动，并且看着只需要扩展下之前租车业务的账户逻辑就好了。但真的是这样吗？

 

事实上从单纯的产品逻辑来看，同一套个人用户账户貌似是没有什么问题的，但是根据很多互联网公司业务发展的路径来看，不同的业务线根据发展情况不同往往会分别设置不同的法律主体，而且根据业务性质的不同监管层面的政策也是完全不一样的，例如在国内运营的租车业务司机车费代收代发这类业务是要求运营公司具备支付牌照的，没有则会牵扯到非法二清这样的法律风险，而一般来说在没有支付牌照又想继续运营的话，替代方案目前主要是采用银行资金托管的方式，即用户资金通过银行三类账户进行托管。

 

一旦涉及这类业务，整个系统的业务复杂度会成倍地增长，所以如果采用同一套个人用户账户的话，资金流将会变得难以拆分，对整个系统的升级也会造成比较大的困扰。

 

另外如果除了租车业务、打车业务外又尝试了别的新业务，如直播之类。业务类型完全不同，且财务本身就要求进行分类的话，只能重新设计新的账户逻辑了；但，从某种角度看这类账户逻辑实质上又是与之前业务存在共性的。如直播类平台也是普通用户充值，购买平台礼物后打赏给主播，此时平台会对用户赠送的礼物抽成后将其转换为可提现的余额结算给主播账户，这类账户逻辑与约车平台其实是很类似的。

 

**那么，是否存在既能统一财务数据又能良好地支持业务的横向扩展\**相对通用的账户系统模型\**呢？**

 

接下来继续和大家探讨一套可以持续扩展的业务账户系统该如何设计？

 

**业务模型**

 

首先我们分析下大多数在线资金业务涉及的逻辑实体，用户是第一存在，但是用户根据参与的性质不同又分为**普通消费端用户、普通服务端用户**，拿打车来说一个普通打车的人属于普通消费端用户，而司机个人则属于普通消费用户，并且司机身份也是可以转换为普通打车用户，另外就是**平台用户**，是指公司内部各个不同的业务主体，它们关联的法律主体可能是同一个也可能是不同的。

 

所以综上所说，我们需要的可能是一套“为不同公司主体下，不同业务的同一个／不同用户提供不同账户交易逻辑支撑，并且可以满足业务及用户平滑扩张的系统”。

 

事实上要达到这样的效果是很难的，需要在系统平台层面进行良好地系统及业务架构设计，并且确保在制定业务规则时遵循一定的制度及规范才行。

 

以下图示为目前业内相对比较完整的通用账户体系模型，俗称“三户模型”，最早是电信行业为适配复杂通信业务场景而设计的一套账户体系模型；而大部分互联网公司业务的复杂度是低于复杂的电信业务的，所以在这里我们对此模型进行部分改进，以期形成适应互联网公司业务发展的通用账户体系模型。

 

![img](https://mmbiz.qpic.cn/mmbiz_jpg/l89kosVutomGx6tyMUeRGB2aaQTHaVSHGPsiatQ6Lfw4uH5TcN51a72evGYib3fTObyJkCQNgCiamMdftptuibSWVg/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

 

在上述模型中信息被划分成了三个类型：客户、用户、账户。客户是用户身份信息的承载实体，例如张三这个人是一个客户；而客户也不仅仅只是针对个人，针对业务线所关联的法律主体也划分在客户的范畴，如某某打车公司。而有了基本的客户信息后，企业客户具体可以开展什么业务，普通个人用户是否使用了该企业客户提供的服务，在模型中是采用用户这个概念来承载的。企业客户开展该业务时根据业务的设计需要开通什么平台账户，普通个人用户使用该业务服务需要开通什么样的个人账户都可以根据业务的设计通过用户下设立相应地账户进行隔离区分。

 

假设此时A公司的租车业务与打车业务法律主体尚未进行拆分属于同一家公司，但是财务上要求隔离两类业务的资金流，那么可以在账户系统上为租车业务开立租车业务用户，如果张三使用了租车业务则为张三设置租车业务用户并在该用户下开立余额、余额返现、押金三类账户并配置业务及记账规则，此外若希望业务资金在平台上也有个完整体现也可以在租车业务用户下设立相应的平台账户，只是业务流程上可以采用缓冲或异步记账的方式。

 

而打车业务则需要根据普通客户信息为普通打车用户开立相应的个人账户，若是司机则需要开通司机相关的业务账户，而平台也需要开通相应的平台账户。在信息流上，客户信息为所有业务所共用，用户信息则是根据不同的业务设置，账户是根据业务挂在相应地业务用户下。如若各业务间没有横向地联系，各业务层账户体系根据自身业务分别运行、互不干扰，而如果存在联系，如租车余额可以进行打车，则可以设计为允许业务层间账户的互转，只是这种资金流会更复杂，需要配置的记账分录也会更加复杂。

 

以上述模型的定义而设计的账户系统会初步形成一个具备支持多类业务账户逻辑并行的通用中台账户系统雏形，从财务角度看账户层次会相对清晰。另外，图中还定义了机构的概念以支持总公司-分公司这样具备层级关系的财务核算需求，当然这种情况在互联网公司的扁平化模式下并不常见，所以可以暂时忽略。

 

本篇通过业务场景举例从业务模型上大概阐述了互联网账户系统如何设计得相对通用和清晰，事实上在系统设计上也需要进行更多的设计，并且需要根据公司实际业务情况进行一定的取舍。

 

 

[互联网账户系统如何设计（下篇）](https://blog.csdn.net/yin767833376/article/details/85763281)

 

来源：http://51jdk.com/7313500be4a84c05bc555c1bf4904b4e.html



- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarThumbUp.png)点赞
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarComment.png)评论](https://blog.csdn.net/yin767833376/article/details/85762913#commentBox)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarShare.png)分享](javascript:;)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)收藏2](javascript:;)
- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarReport.png)举报
- [关注](javascript:;)
- 一键三连

[*账户*体系数据库*设计*](http://download.csdn.net/download/keypanj2ee/10816973)

11-29

[（1）.*账户*：收入和支出的主要对象实体。 （2）.支出：该*账户*的支出金额。 （3）.收入：该*账户*的收入金额、 （4）.余额：该*账户*在进行收入以及支出事件之后当前金额数。 （5）.支出清单：*账户*每次详细的支出记录。 （6）.收入清单：*账户*每次详细的收入记录。](http://download.csdn.net/download/keypanj2ee/10816973)

[用户中心方案](http://download.csdn.net/download/lisenustc/10821893)

12-01

[*设计*文档，用户中心方案*设计*模板；*设计*文档，用户中心方案*设计*模板](http://download.csdn.net/download/lisenustc/10821893)





[![img](https://profile.csdnimg.cn/D/B/6/3_yangjava2014)](https://blog.csdn.net/yangjava2014)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/commentFlag@2x.png)

![表情包](https://csdnimg.cn/release/blogv2/dist/pc/img/emoticon.png)

相关推荐

[*互联网**账户**系统*如何*设计*(下篇)_菜鸟笔记](https://blog.csdn.net/yin767833376/article/details/85763281)

4-13

[在上一篇文章中(*互联网**账户**系统*如何*设计*(上篇)?)我们通过场景举例的方式,讨论了一套相对通用的*互联网*业务*账户**系统*,从业务模型上应该如何定义。那么除了从业务模型上进行定义外,在具体*系统*实现上又该如何*设计*?又有哪些需要注意的地方呢?在...](https://blog.csdn.net/yin767833376/article/details/85763281)

[*互联网**账户**系统*如何*设计*(转载) - fcvtb的博客](https://blog.csdn.net/fcvtb/article/details/88796474)

12-21

[*互联网**账户**系统*如何*设计*在很多*互联网*公司业务发展的早期,业务模式比较单一的情况下,涉及用户*账户*资金交易相关的逻辑也比较简单,但是随着公司业务模式的不断创新及类型的多元化发展,会渐渐发现现有*系统**账户*逻辑越来越雍肿,不仅难以支持新业务...](https://blog.csdn.net/fcvtb/article/details/88796474)

[多账号体系下用户*系统*的*设计*.txt](https://download.csdn.net/download/u014445777/9653172)

10-13

[*系统*解析多账号体系下的*系统**设计*](https://download.csdn.net/download/u014445777/9653172)

[SpringBoot实战教程：SpringBoot 博客项目开发及讲解](https://edu.csdn.net/course/detail/29029)

05-14

<p> <span style="color:#4d4d4d;">当前课程中博客项目的实战源码是我在 GitHub上开源项目 My-Blog，目前已有 2000 多个 star：</span> </p> <p> <span style="color:#4d4d4d;"><img src="https://img-bss.csdnimg.cn/202103310649344285.png" alt="" /><br /> </span> </p> <p> <span style="color:#4d4d4d;">本课程是一个 Spring Boot 技术栈的实战类课程，课程共分为 3 大部分，前面两个部分为基础环境准备和相关概念介绍，第三个部分是 Spring Boot 个人博客项目功能的讲解，<span style="color:#565656;">通过本课程的学习，不仅仅让你掌握基本的 Spring Boot 开发能力以及 Spring Boot 项目的大部分开发使用场景，同时帮你提前甄别和处理掉将要遇到的技术难点，认真学完这个课程后，你将会对 Spring Boot 有更加深入而全面的了解，同时你也会得到一个大家都在使用的博客系统源码，你可以根据自己的需求和想法进行改造，也可以直接使用它来作为自己的个人网站，这个课程一定会给你带来巨大的收获。</span></span> </p> <p> <span style="color:#4d4d4d;"><span style="color:#565656;"> </span></span> </p> <p> <span style="color:#e53333;"><span style="color:#e53333;"><strong>课程特色</strong></span></span> </p> <p> <span style="color:#e53333;"><span style="color:#e53333;"><strong> </strong></span></span> </p> <p> <span style="color:#4d4d4d;"><span style="color:#565656;"> </span></span> </p> <ol> <li> <span style="color:#565656;">课程内容紧贴 Spring Boot 技术栈，涵盖大部分 Spring Boot 使用场景。</span> </li> <li> <span style="color:#565656;">开发教程详细完整、文档资源齐全、实验过程循序渐进简单明了。</span> </li> <li> <span style="color:#565656;">实践项目页面美观且实用，交互效果完美。</span> </li> <li> <span style="color:#565656;">包含从零搭建项目、以及完整的后台管理系统和博客展示系统两个系统的功能开发流程。</span> </li> <li> <span style="color:#565656;">技术栈新颖且知识点丰富，学习后可以提升大家对于知识的理解和掌握，对于提升你的市场竞争力有一定的帮助。</span> </li> </ol> <p> <strong>实战项目预览</strong> </p> <p> <span style="color:#4d4d4d;"><span style="color:#565656;"><span style="color:#e53333;"><strong> </strong></span></span></span> </p> <p> <span style="color:#4d4d4d;"><img src="https://img-bss.csdn.net/202005150303066258.png" alt="" /><br /> </span> </p> <p>   </p> <p> <span style="color:#4d4d4d;"> </span> </p> <p> <span style="color:#4d4d4d;"><img src="https://img-bss.csdn.net/202005150305396930.png" alt="" /><br /> </span> </p> <p> <span style="color:#4d4d4d;"> </span> </p> <p> <span style="color:#4d4d4d;"><img src="https://img-bss.csdn.net/202005150305528842.png" alt="" /><br /> </span> </p> <p> <span style="color:#4d4d4d;"> </span> </p> <p> <span style="color:#4d4d4d;"><img src="https://img-bss.csdn.net/202005150306056323.png" alt="" /><br /> </span> </p>

[*互联网**账户**系统*如何*设计*_茅坤宝骏氹的博客](https://blog.csdn.net/moakun/article/details/82857738)

3-31

[转载自*互联网**账户**系统*如何*设计* 在很多*互联网*公司业务发展的早期,业务模式比较单一的情况下,涉及用户*账户*资金交易相关的逻辑也比较简单,但是随着公司业务模式的不断创新及类型的多元化发展,会渐渐发现现有*系统**账户*逻辑越来越雍肿,不仅难以支持...](https://blog.csdn.net/moakun/article/details/82857738)

[*互联网**账户**系统*的具体实现_ff00yo的博客](https://blog.csdn.net/ff00yo/article/details/88744217)

4-24

[在上一篇文章中我们通过场景举例的方式,讨论了一套相对通用的*互联网*业务*账户**系统*,从业务模型上应该如何定义。那么除了从业务模型上进行定义外,在具体*系统*实现上又该如何*设计*?又有哪些需要注意的地方呢?在本篇内容中小码农就和大家一起讨论...](https://blog.csdn.net/ff00yo/article/details/88744217)

[*互联网**账户**系统*如何*设计*（上篇）？](https://blog.csdn.net/b644ROfP20z37485O35M/article/details/82230160)

[JAVA葵花宝典](https://blog.csdn.net/b644ROfP20z37485O35M)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 8232

[在很多*互联网*公司业务发展的早期，业务模式比较单一的情况下，涉及用户*账户*资金交易相关的逻辑也比较简单，但是随着公司业务模式的不断创新及类型的多元化发展，会渐渐发现现有*系统*账...](https://blog.csdn.net/b644ROfP20z37485O35M/article/details/82230160)

[全新 PowerDesigner 16.6 数据库*设计*与建模（精讲版）](https://edu.csdn.net/course/detail/24751)

05-15

[  PowerDesigner数据库*设计*与建模，本课程讲述了如何使用PowerDesigner进行数据库分析与建模。包括企业架构及业务流程分析，实体关系模型*设计*，面向对象和数据库建模的集成等功能模块进行项目需求分析、结构规划、生成框架代码，以及如何从现有*系统*逆向转工程代码，生成所需*系统*模型的全过程。软件*设计*师专题课程的第一篇<<软件*设计*与建模>>请参看https://edu.csdn.net/course/detail/24752。本课程作者联络QQ:494657271  ](https://edu.csdn.net/course/detail/24751)

[2021软考网络工程师--基础知识视频教程](https://edu.csdn.net/course/detail/601)

04-10

<p> 基础知识视频教程内容包括：数据通信基础、局域网、城域网、广域通信网、网络互连与互联网等内容，为顺利通过软考和自身能力提高打下坚实基础，让小白成长为网络工程师。 </p> <p> <img src="https://img-bss.csdnimg.cn/202104020319097172.png" alt="" /> </p> <p> <img src="https://img-bss.csdnimg.cn/202104020320063996.png" alt="" /><img src="https://img-bss.csdnimg.cn/202104020320211423.jpg" alt="" /><img src="https://img-bss.csdnimg.cn/202104020320472438.png" alt="" /><img src="https://img-bss.csdnimg.cn/202104020321091097.jpg" alt="" /> </p>

[*互联网**账户**系统*如何*设计*（下篇）？](https://blog.csdn.net/b644ROfP20z37485O35M/article/details/82655452)

[JAVA葵花宝典](https://blog.csdn.net/b644ROfP20z37485O35M)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 2720

[导读在上一篇文章中（*互联网**账户**系统*如何*设计*（上篇）？）我们通过场景举例的方式，讨论了一套相对通用的*互联网*业务*账户**系统*，从业务模型上应该如何定义。那么除了从业务模型上进行定...](https://blog.csdn.net/b644ROfP20z37485O35M/article/details/82655452)

[U一点·料+-+阿里巴巴集团+1688用户体验*设计*部](https://download.csdn.net/download/mailroom/9925630)

08-09

[U一点·料+-+阿里巴巴集团+1688用户体验*设计*部](https://download.csdn.net/download/mailroom/9925630)

[*互联网*财富管理平台应该怎么做？（上篇）](https://blog.csdn.net/JDDTechTalk/article/details/109091305)

[JDDTechTalk的博客](https://blog.csdn.net/JDDTechTalk)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 5919

[在构造了很多*互联网*金融*系统*之后，我经常问自己一个问题。 对于一个*互联网*金融平台， 给用户提供什么样的理财产品是用户最喜欢的？ 之所以想到这个问题，是因为*互联网*公司都是将用户体验强调到极致的，只有极致的用户体验才能虏获用户的心，只有获得了用户才能获得市场，业务才能持续开展，平台才能够赚到钱。这个问题听起来简单，但是仔细一想又很复杂。 首先，用户都想要赚钱，所以利率最高的产品是最受欢迎的。 这个可以从这几年发展的风起云涌的P2P可以看出来，这些所谓的理财产品，利率低于年化10%的人们都嫌低，但是，后来市场又血淋](https://blog.csdn.net/JDDTechTalk/article/details/109091305)

[浅谈账号*系统**设计*](https://blog.csdn.net/dianfanglian2905/article/details/101501778)

[dianfanglian2905的博客](https://blog.csdn.net/dianfanglian2905)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 414

[现在几乎大部分的App都支持使用多个第三方账号进行登录，如：微信、QQ、微博等，我们把此称为多账号统一登陆。而这些账号的表*设计*，流程*设计*至关重要，不然后续扩展性贼差。本文不提供任何代码实操，但是梳理一下博主根据我司账号模块的*设计*，提供思路，仅供参考。 一、 自建的登陆体系 1.1 手机号登陆注册 该*设计*的思路是每个手机号对应一个用户，手机号为必填项。 流程： 首先输入手机号，然后发...](https://blog.csdn.net/dianfanglian2905/article/details/101501778)

[用户*系统**设计*](https://blog.csdn.net/gglinux/article/details/68948901)

[gglinux的专栏](https://blog.csdn.net/gglinux)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[用户*系统*，主要分为账号体系和用户信息两大类。账号体系包括，登陆验证、注册、第三方授权、以及权限管理。用户信息包括，用户地理位置、用户属性、用户设备信息、还有用户日志信息。本文会介绍用户*系统*的具体落地方案。登陆验证在一般项目账号体系中，一般会要求支持手机、邮箱、账号、QQ、微信、微博实现登陆。后面三种方式都是基于第三方授权后，完成的身份验证。手机、邮箱、账号则是相对传统的登录方式。用户身份与登录的授权](https://blog.csdn.net/gglinux/article/details/68948901)

[*账户*体系的*设计*（1）](https://blog.csdn.net/weixin_33910434/article/details/93632881)

[weixin_33910434的博客](https://blog.csdn.net/weixin_33910434)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 613

[回来看看自己的博客园，自己总是三天打鱼，2天晒网。从flask到爬虫，现在啥也没学会，残念。 也许要一以贯之吧。 我是想说说*账户*体系的*设计*的。 但我没有什么强大的*设计*能力，而且我也总觉得*账户**设计*这个事和会计没啥关系，不知道很多人为什么一定要扯上会计的事。 我们还是从第三方支付来只管感受一下，小白用户对*账户*的感受是咋样的？ 直接就以微信支付来说好了，先看微信支付是否有*账户*？ 我们觉得是有...](https://blog.csdn.net/weixin_33910434/article/details/93632881)

[*互联网**账户**系统*如何*设计*（下篇）？](https://blog.csdn.net/weixin_28926077/article/details/113488337)

[weixin_28926077的博客](https://blog.csdn.net/weixin_28926077)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 16

[转https://www.javazhiyin.com/18029.html](https://blog.csdn.net/weixin_28926077/article/details/113488337)

[银行*账户*(*设计*模式--状态模式)](https://blog.csdn.net/qq_41024297/article/details/103283019)

[岛岛咕](https://blog.csdn.net/qq_41024297)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 2059

[题目描述：某银行*系统*中定义了*账户*的三种状态： （1）如果*账户*（Account）余额不小于0，*账户*状态为绿色，即正常状态，既可以存，也可以取款； （2）如果余额小于0，并且大于等于-1000，*账户*状态为蓝色，即欠费状态，既可以存，也可以取款； （3）如果*账户*中余额小于-1000，*账户*状态为红色，即透支状态，只能存款，不能取款。 实现截图： ...](https://blog.csdn.net/qq_41024297/article/details/103283019)

[*互联网*产品用户*系统**设计*的几个细节](https://blog.csdn.net/weixin_34067049/article/details/92896372)

[weixin_34067049的博客](https://blog.csdn.net/weixin_34067049)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 83

[用户是我们做产品的重中之重，一个好的产品肯定有一定量的用户来支撑的。所以在我们的产品*设计*和开发中一定要做好用户*系统**设计*，来对用户进行跟踪和管理，通过对用户的分析，来寻找吸引更多用户的渠道，试验提高用户贡献值的方式。 用户的几个来源 1、主动使用 指的是用户主动在应用市场下载应用，这些用户是通过搜索或者浏览的方式了解了我们的应用并下载使用，这些用户相比其他用户有主动...](https://blog.csdn.net/weixin_34067049/article/details/92896372)

[*互联网*金融-资金*账户**系统**设计*](https://blog.csdn.net/qyvlik/article/details/100888297)

[qyvlik的专栏](https://blog.csdn.net/qyvlik)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1037

[*互联网*金融-资金*账户**系统**设计* 支付*系统**设计* *互联网**账户**系统*如何*设计*（上篇）？ *互联网**账户**系统*如何*设计*（下篇）？ 支付对账*系统*怎么*设计*？ 移动端支付*系统*如何*设计*有效地防重失效机制？ 如何做一个对账*系统* 聊聊对账*系统*的*设计*方案 ...](https://blog.csdn.net/qyvlik/article/details/100888297)

[支付*系统**设计*：支付*系统*的*账户*模型（一）](https://blog.csdn.net/kagurawill/article/details/82797053)

[kagurawill的博客](https://blog.csdn.net/kagurawill)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 5400

[*账户*体系是支付*系统*的基础，它的*设计*直接影响整个*系统*的特性。这里探讨如何针对电子商务*系统*的支付*账户*体系*设计*。我们从一些基本概念开始入手，了解怎么建模。 支付*账户*和登录账号 *账户*体系*设计*首先要区分两个概念，支付*账户*和登录账号。 这是两个不同业务领域的概念：支付*账户*指用户在支付*系统*中用于交易的资金所有者权益的凭证；登录账号指用户在*系统*中的登录的凭证和个人信息。 一个用户可以有多个登录...](https://blog.csdn.net/kagurawill/article/details/82797053)

[MySQL实战45讲【完结】.rar](http://download.csdn.net/download/qq_16790931/15077243)

02-05

[MySQL教程，PDF电子高清版加MP3音频讲解](http://download.csdn.net/download/qq_16790931/15077243)

[Java 基础高频面试题（2021年最新版）](https://joonwhee.blog.csdn.net/article/details/115364158)

[程序员囧辉](https://blog.csdn.net/v123411739)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 5万+

[最新 Java 基础高频面试题](https://joonwhee.blog.csdn.net/article/details/115364158)

©️2020 CSDN 皮肤主题: 大白 设计师:CSDN官方博客 [返回首页](https://blog.csdn.net/)

- [关于我们](https://www.csdn.net/company/index.html#about)
- [招贤纳士](https://www.csdn.net/company/index.html#recruit)
- [广告服务](https://www.csdn.net/company/index.html#advertisement)
- [开发助手](https://plugin.csdn.net/)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/tel.png)400-660-0108
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/email.png)[kefu@csdn.net](mailto:webmaster@csdn.net)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/cs.png)[在线客服](https://csdn.s2.udesk.cn/im_client/?web_plugin_id=29181)
- 工作时间 8:30-22:00

- [公安备案号11010502030143](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010502030143)
- [京ICP备19004658号](http://beian.miit.gov.cn/publish/query/indexFirst.action)
- [京网文〔2020〕1039-165号](https://csdnimg.cn/release/live_fe/culture_license.png)
- [经营性网站备案信息](https://csdnimg.cn/cdn/content-toolbar/csdn-ICP.png)
- [北京互联网违法和不良信息举报中心](http://www.bjjubao.org/)
- [网络110报警服务](http://www.cyberpolice.cn/)
- [中国互联网举报中心](http://www.12377.cn/)
- [家长监护](https://download.csdn.net/index.php/tutelage/)
- [Chrome商店下载](https://chrome.google.com/webstore/detail/csdn开发者助手/kfkdboecolemdjodhmhmcibjocfopejo?hl=zh-CN)
- ©1999-2021北京创新乐知网络技术有限公司
- [版权与免责声明](https://www.csdn.net/company/index.html#statement)
- [版权申诉](https://blog.csdn.net/blogdevteam/article/details/90369522)
- [出版物许可证](https://img-home.csdnimg.cn/images/20210414021151.jpg)
- [营业执照](https://img-home.csdnimg.cn/images/20210414021142.jpg)

[![img](https://profile.csdnimg.cn/1/5/8/3_yin767833376)](https://blog.csdn.net/yin767833376)

[菜鸟笔记](https://blog.csdn.net/yin767833376)

码龄7年[![img](https://csdnimg.cn/identity/nocErtification.png) 暂无认证](https://i.csdn.net/#/uc/profile?utm_source=14998968)




- 102万+

  访问

- [![img](https://csdnimg.cn/identity/blog7.png)](https://blog.csdn.net/blogdevteam/article/details/103478461)

  等级

- 9430

  积分

- 262

  粉丝

- 209

  获赞

- 195

  评论

- 454

  收藏

![领英](https://csdnimg.cn/medal/linkedin@240.png)

![GitHub](https://csdnimg.cn/medal/github@240.png)

![阅读者勋章Lv1](https://csdnimg.cn/medal/yuedu3@240.png)

![专栏达人](https://csdnimg.cn/medal/zhuanlandaren@240.png)

![勤写标兵Lv1](https://csdnimg.cn/medal/qixiebiaobing1@240.png)

[私信](https://im.csdn.net/chat/yin767833376)

关注

![img](https://csdnimg.cn/cdn/content-toolbar/csdn-sou.png?v=1587021042)

### 热门文章

- [二维码的两种生成方法（前端js生成，后端java生成） ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 42387](https://blog.csdn.net/yin767833376/article/details/78850390)
- [Mysql区分大小写（大小写敏感）的问题总结 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 35294](https://blog.csdn.net/yin767833376/article/details/52932545)
- [解决了winscp连接不上的问题 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 34812](https://blog.csdn.net/yin767833376/article/details/51720233)
- [设置Session永不过期，Session有效时间的问题 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 33720](https://blog.csdn.net/yin767833376/article/details/51735191)
- [Mysql里的JSON系列操作函数 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 29006](https://blog.csdn.net/yin767833376/article/details/52032927)

### 分类专栏

- [![img](https://img-blog.csdn.net/20170228123309392?imageView2/1/w/64/h/64/interlace/1)Java开发160篇](https://blog.csdn.net/yin767833376/category_9268142.html)
- [![img](https://img-blog.csdn.net/20170228124240563?imageView2/1/w/64/h/64/interlace/1)前端开发专栏30篇](https://blog.csdn.net/yin767833376/category_9268192.html)
- [![img](https://img-blog.csdn.net/20170228123653516?imageView2/1/w/64/h/64/interlace/1)bootstrap专栏16篇](https://blog.csdn.net/yin767833376/category_9268190.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756757.png?x-oss-process=image/resize,m_fixed,h_64,w_64)Java开发58篇](https://blog.csdn.net/yin767833376/category_6124656.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756754.png?x-oss-process=image/resize,m_fixed,h_64,w_64)数据库38篇](https://blog.csdn.net/yin767833376/category_6124657.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756919.png?x-oss-process=image/resize,m_fixed,h_64,w_64)spring25篇](https://blog.csdn.net/yin767833376/category_6226412.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756913.png?x-oss-process=image/resize,m_fixed,h_64,w_64)mybatis20篇](https://blog.csdn.net/yin767833376/category_6240114.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756780.png?x-oss-process=image/resize,m_fixed,h_64,w_64)freemark && bootstrap16篇](https://blog.csdn.net/yin767833376/category_6253315.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756918.png?x-oss-process=image/resize,m_fixed,h_64,w_64)Linux20篇](https://blog.csdn.net/yin767833376/category_6273006.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756919.png?x-oss-process=image/resize,m_fixed,h_64,w_64)redis11篇](https://blog.csdn.net/yin767833376/category_6280112.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756928.png?x-oss-process=image/resize,m_fixed,h_64,w_64)tomcat2篇](https://blog.csdn.net/yin767833376/category_6296841.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756918.png?x-oss-process=image/resize,m_fixed,h_64,w_64)设计模式5篇](https://blog.csdn.net/yin767833376/category_6677926.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756928.png?x-oss-process=image/resize,m_fixed,h_64,w_64)Dubbo4篇](https://blog.csdn.net/yin767833376/category_6707329.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756925.png?x-oss-process=image/resize,m_fixed,h_64,w_64)支付平台5篇](https://blog.csdn.net/yin767833376/category_7121804.html)
- [![img](https://img-blog.csdnimg.cn/20201014180756913.png?x-oss-process=image/resize,m_fixed,h_64,w_64)MQ1篇](https://blog.csdn.net/yin767833376/category_7839394.html)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/arrowDownWhite.png)

### 最新评论

- BTree和B+Tree详解

  [m0_48452161: ](https://blog.csdn.net/m0_48452161)有点东西啊

- bootstrap中table的行拖拽实现

  [哂笑年少°: ](https://blog.csdn.net/w15562397578)博主可否发一下免费的js包

- MySQL按中文排序

  [蔚尺丈八声: ](https://blog.csdn.net/qq_30282617)我写了一个按拼音和笔顺排序中文的cn_sort库，可解决多音字，楼主若感兴趣可以看下：https://github.com/bmxbmx3/cn_sort

- rocketmq初学者入门

  [taiguolaotu: ](https://blog.csdn.net/taiguolaotu)rocketmq 如何实现自动监听消息(自动监听topic和消息) 监听器怎么写的请问？

- 一探前端开发中的JS调试技巧

  [SpiderLiH: ](https://blog.csdn.net/weixin_38819889)js调试总结的不错，可惜的就是把文章粘贴复制过来的时候，格式 换行 没有调整下

### 最新文章

- [redis设计与实现：简单动态字符串SDS](https://blog.csdn.net/yin767833376/article/details/106210525)
- [机器学习：KNN用java代码实现](https://blog.csdn.net/yin767833376/article/details/106209896)
- [如何设计一个秒杀系统总结](https://blog.csdn.net/yin767833376/article/details/103028616)

[2020年2篇](https://blog.csdn.net/yin767833376/article/month/2020/05)

[2019年7篇](https://blog.csdn.net/yin767833376/article/month/2019/11)

[2018年16篇](https://blog.csdn.net/yin767833376/article/month/2018/12)

[2017年83篇](https://blog.csdn.net/yin767833376/article/month/2017/12)

[2016年192篇](https://blog.csdn.net/yin767833376/article/month/2016/12)

![img](https://kunyu.csdn.net/1.png?p=57&a=2488&c=0&k=&spm=1001.2101.3001.5001&d=1&t=3&u=1a9f406f467b40be98c29c723ea37b8d)



![img](https://g.csdnimg.cn/side-toolbar/3.0/images/guide.png)![img](https://g.csdnimg.cn/side-toolbar/3.0/images/kefu.png)举报![img](https://g.csdnimg.cn/side-toolbar/3.0/images/fanhuidingbucopy.png)

[![CSDN首页](https://img-home.csdnimg.cn/images/20201124032511.png)](https://www.csdn.net/)

- [首页](https://www.csdn.net/)
- [博客](https://blog.csdn.net/)
- [程序员学院](https://edu.csdn.net/)
- [下载](https://download.csdn.net/)
- [论坛](https://bbs.csdn.net/)
- [问答](https://ask.csdn.net/)
- [代码](https://codechina.csdn.net/?utm_source=csdn_toolbar)
- [直播](https://live.csdn.net/?utm_source=csdn_toolbar)
- [能力认证](https://ac.csdn.net/)
- [高校](https://studentclub.csdn.net/)

 

[![img](https://profile.csdnimg.cn/D/B/6/2_yangjava2014)](https://blog.csdn.net/yangjava2014)

[会员中心](https://mall.csdn.net/vip)

[收藏](https://i.csdn.net/#/user-center/collection-list?type=1)

[动态](https://blog.csdn.net/nav/watchers)

[消息](https://i.csdn.net/#/msg/index)

[创作中心](https://mp.csdn.net/)

# 商品领域ddd_DDD领域驱动设计实战详解（一 核心概念）

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[weixin_39600319](https://blog.csdn.net/weixin_39600319) 2020-11-29 12:05:45 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 86 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏

文章标签： [商品领域ddd](https://www.csdn.net/tags/MtjaIgysMTEwNDktYmxvZwO0O0OO0O0O.html)

版权

##### 工程师小C的小店[我也想开通小店](https://mp.csdn.net/console/MyShop)

[![img](https://csdn-test-oss.oss-cn-beijing.aliyuncs.com/images/20210402024431.jpeg)Python编程三剑客：Python编程从入门到实践第2版+快速上手第2版+极客编程（套装共3册）*作者：[美\] 埃里克·马瑟斯（Eric Matthes）**出版社：人民邮电出版社**好评：100.0%* *销售量：6**￥148.2*](javascript:;)[更多](javascript:;)



![02402343d1a719c8f2532066f0a1e72b.png](https://img-blog.csdnimg.cn/img_convert/02402343d1a719c8f2532066f0a1e72b.png)

> 设计不只是感观，设计就是产品的工作方式。——Steve Jobs
> 相信很多小伙伴，或多或少都会对DDD，有过一定的了解，但总是感觉像天书一样，不知道如何下手，或许也看过很多关于 DDD 的文章, 也买过一些书籍, 但是发现内容冗长, 同时讲解的也不够系统，导致我们的知识体系总是零散的，无法串联起来，在遇到实际项目时 不知道该如何入手，由于 DDD 不是一套框架，而是一种架构思想，所以在代码层面缺乏了足够的约束，导致 DDD 在实际应用中上手门槛很高，甚至可以说绝大部分人都对 DDD 的理解有所偏差，就算已经在项目中使用的人，每个人的理解也不一样。
> 后续我会整理一系列的文章，从不同的角度对DDD在实际项目中如何落地进行说明，希望可以帮助到大家，同时让DDD的架构思想能够得到推广

本篇博文作为第一篇，给大家从整体的概念上介绍下DDD，让大家先有个全局的认识，这里我借用了一些其他博客的内容，这里进行了一定的整理，希望能够形成一个系统的系列。

## DDD初识

DDD（Domain-Driven Design 领域驱动设计）是由Eric Evans最先提出，目的是对软件所涉及到的领域进行建模，以应对系统规模过大时引起的软件复杂性的问题。整个过程大概是这样的，开发团队和领域专家一起通过 通用语言(Ubiquitous Language)去理解和消化领域知识，从领域知识中提取和划分为一个一个的子领域（核心子域，通用子域，支撑子域），并在子领域上建立模型，再重复以上步骤，这样周而复始，构建出一套符合当前领域的模型。



![83e9294f41c4c93dac87f2d138c445e8.png](https://img-blog.csdnimg.cn/img_convert/83e9294f41c4c93dac87f2d138c445e8.png)

DDD 的相关术语与基本概念

讨论完宏观概念以后，让我们来认识一下 DDD 的一些名词的概念。

**统一语言：**定义上下文的含义。它的价值是可以解决交流障碍，不管你是 RD、PM、QA 等什么角色，让每个团队使用统一的语言（概念）来交流，甚至可读性更好的代码。 通用语言包含属于和用例场景，并且能直接反应在代码中。 可以在事件风暴（开会）中来统一语言，甚至是中英文的映射、业务与代码模型的映射等。可以使用一个表格来记录。

**限界上下文：**定义上下文的边界。领域模型存在边界之内。对于同一个概念，不同上下文会有不同的理解，比如商品，在销售阶段叫商品，在运输阶段就叫货品。 首先我们在描述领域时，必定会提到“限界上下文”,简单理解就是领域所处的环境以及邻域处理问题的边界。理论上，限界上下文的边界就是微服务的边界，因此，理解限界上下文在设计中非常重要。



![bfde01611523d666138b64ad32c2e4c6.png](https://img-blog.csdnimg.cn/img_convert/bfde01611523d666138b64ad32c2e4c6.png)

**领域：**领域就是范围。范围的重点是边界。领域的核心思想是将问题逐级细分来减低业务和系统的复杂度，这也是 DDD 讨论的核心。领域既可以表示整个业务系统，也可以表示某个核心子域或者支持子域。可以简单的理解为一个比较完整的含有自己属性和行为的大对象（虽然不恰当，但是辅助理解吧）。在微服务体系中可以理解为一个微服务（我们微服务拆分，通常也是以领域为概念来拆分的，他们两个可以相互理解）。

**子域：**领域可以进一步划分成子领域，即子域。这是处理高度复杂领域的设计思想，它试图分离技术实现的复杂性。这个拆分的里面在很多架构里都有。

**核心域：**在领域划分过程中，会不断划分子域，子域按重要程度会被划分成三类：核心域、通用域、支撑域。决定产品核心竞争力的子域就是核心域，是业务成功的主要促成因素，没有太多个性化诉求。*桃树的例子*，有根、茎、叶、花、果、种子等六个子域，不同人理解的核心域不同，比如在果园里，核心域就是*果是核心域*，在公园里，*核心域则是花*。有时为了核心域的营养供应，还会剪掉通用域和支撑域（茎、叶等）。

**通用域：**如果一个子域被应用于整个业务系统，被多个子域使用的通用功能就是通用域，没有太多企业特征，比如权限认证。

**支撑域：**对应着业务的某些重要方面，但不是核心，对于功能来讲是必须存在的，但它不对产品核心竞争力产生影响，也不包含通用功能，有企业特征，不具有通用性，比如数据代码类的数字字典系统。

**聚合：**聚合概念类似于你理解的包的概念，每个包里包含一类实体或者行为，它有助于分散系统复杂性，也是一种高层次的抽象，可以简化对领域模型的理解。
拆分的实体不能都放在一个服务里，这就涉及到了拆分，那么有拆分就有聚合。聚合是为了保证领域内对象之间的一致性问题。
在定义聚合的时候，应该遵守不变形约束法则：

1. 聚合边界内必须具有哪些信息，如果没有这些信息就不能称为一个有效的聚合；
2. 聚合内的某些对象的状态必须满足某个业务规则：

- 一个聚合只有一个聚合根，聚合根是可以独立存在的，聚合中其他实体或值对象依赖与聚合根。
- 只有聚合根才能被外部访问到，聚合根维护聚合的内部一致性。

**聚合根:**一个上下文内可能包含多个聚合，每个聚合都有一个根实体，叫做聚合根，一个聚合只有一个聚合根。

**实体** Domain 或 entity。《领域驱动设计模式、原理与实践》一书中讲到，实体是具有身份和连贯性的领域概念，可以看出，实体其实也是一种特殊的领域，这里我们需要注意两点：唯一标示（身份）、连续性。两者缺一不可。
你可以想象，文章可以是实体，作者也可以是，因为它们有 id 作为唯一标示。

**值对象:**为了更好地展示领域模型之间的关系，制定的一个对象，本质上也是一种实体，但相对实体而言，它没有状态和身份标识，它存在的目的就是为了表示一个值，通常使用值对象来传达数量的形式来表示。比如 money，让它具有 id 显然是不合理的，你也不可能通过 id 查询一个 money。定义值对象要依照具体场景的区分来看，你甚至可以把 Article 中的 Author 当成一个值对象，但一定要清楚，Author 独立存在的时候是实体，或者要拿 Author 做复杂的业务逻辑，那么 Author 也会升级为聚合根。

今天就先描述这么多，还有一些概念，我后续会整理出来并更新到这篇文章里例如:上下文映射图、CQRS、领域服务、领域事件、模块、工厂、资源库、集成限界上下文等。

后续会整理成代码，结合代码在做进一步的说明。

##### 工程师小C的小店[我也想开通小店](https://mp.csdn.net/console/MyShop)

[![img](https://img-bss.csdnimg.cn/2020852192818_25789.jpg?imageMogr2/auto-orient/thumbnail/360x360!/format/jpg%7Cwatermark/1/image/aHR0cHM6Ly9pbWctYnNzLmNzZG5pbWcuY24vYkphdmFfOS5wbmc=/dissolve/85/gravity/Center/dx/0/dy/0/)缓存中间件Redis技术入门与应用场景实战（SpringBoot2.x + 抢红包系统设计与实战）*讲师：修罗debug**好评：100.0%* *销售量：1**￥99*](javascript:;)[更多](javascript:;)



- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarThumbUp.png)点赞
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarComment.png)评论](https://blog.csdn.net/weixin_39600319/article/details/111644461#commentBox)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarShare.png)分享](javascript:;)
- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)收藏](javascript:;)
- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarReport.png)举报
- [关注](javascript:;)
- 一键三连

[*领域**驱动设计**DDD*和CQRS落地](https://blog.csdn.net/jiangbb8686/article/details/103780934)

[jiangbb8686的博客](https://blog.csdn.net/jiangbb8686)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 947

[*DDD*分层架构 Evans在它的《*领域**驱动设计*：软件*核心*复杂性应对之道》书中推荐采用分层架构去实现*领域**驱动设计*： image 其实这种分层架构我们早已驾轻就熟，MVC模式就是我们所熟知的一种分层架构，我们尽可能去设计每一层，使其保持高度内聚性，让它们只对下层进行依赖，体现了高内聚低耦合的思想。 分层架构的落地就简单明了了，用户界面层我们可以理解成web层的Controller，应用层和...](https://blog.csdn.net/jiangbb8686/article/details/103780934)

[基于*DDD*的微服务设计和开发*实战*](https://blog.csdn.net/iamlake/article/details/93327267)

[Liyiming](https://blog.csdn.net/iamlake)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 7589

[基于*DDD*的微服务设计和开发*实战* 作者：欧创新、邓頔、文艺 目录 基于*DDD*的微服务设计和开发*实战* 1 目标 2 适用范围 3 *DDD* 分层架构视图 展现层 应用层 *领域*层 基础层 4 服务视图 微服务内的服务视图 1、接口服务 2、应用服务 3、*领域*服务 4、基础服务 微服务外的服务视图 1. 前端应用与微...](https://blog.csdn.net/iamlake/article/details/93327267)





[![img](https://profile.csdnimg.cn/D/B/6/3_yangjava2014)](https://blog.csdn.net/yangjava2014)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/commentFlag@2x.png)

![表情包](https://csdnimg.cn/release/blogv2/dist/pc/img/emoticon.png)



- [![weixin_39600319](https://profile.csdnimg.cn/C/E/6/3_weixin_39600319)](https://blog.csdn.net/weixin_39600319)

  [weixin_39600319![img](https://csdnimg.cn/release/blogv2/dist/components/img/bloger@2x.png)](https://blog.csdn.net/weixin_39600319)**:**这篇文章对你有帮助吗？作为一名程序工程师，在评论区留下你的困惑或你的见解，大家一起来交流吧！

相关推荐

[*商品**领域**ddd*_*DDD*战略设计相关*核心**概念*的理解_weixin_39...](https://blog.csdn.net/weixin_39963080/article/details/111644453)

4-13

[*商品**领域**ddd*_*DDD*战略设计相关*核心**概念*的理解 前言 本文想再讨论一下关于*领域*、业务、业务模型、解决方案、BC、*领域*模型、微服务这些*概念*的含义和关系。初衷是我发现现在*DDD**领域*建模以及解决方案落地过程中,常常对这些*概念*理解不清楚或者有...](https://blog.csdn.net/weixin_39963080/article/details/111644453)

[*领域**驱动设计*(*DDD*)部分*核心**概念*的个人理解_weixin_3435...](https://blog.csdn.net/weixin_34351321/article/details/90629401)

5-5

[*领域**驱动设计*(*DDD*)是一种基于模型驱动的软件设计方式。它以*领域*为*核心*,分析*领域*中的问题,通过建立一个*领域*模型来有效的解决*领域*中的*核心*的复杂问题。Eric Ivans为*领域**驱动设计*提出了大量的最佳实践和经验技巧。只有对*领域*的不断深入认识,才...](https://blog.csdn.net/weixin_34351321/article/details/90629401)

[*领域**驱动设计*（*DDD*）相关架构介绍与演变过程分析（图文*详解*）](https://blog.csdn.net/qq_32828253/article/details/110673205)

[July的博客](https://blog.csdn.net/qq_32828253)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 3349

[我们生活中都听说了*DDD*，也了解了*DDD*，那么怎么将一个新项目从头开始按照*DDD*的过程进行划分与架构涉及呢？ 专业术语 各种服务 IAAS：基础设施服务，Infrastructure-as-a-service PAAS：平台服务，Platform-as-a-service SAAS：软件服务，Software-as-a-service 架构演变 从图中已经可以很容易看出架构的演进过程，通过对三个层的举例来进行说明： SAAS：比如我们最早的就是单体应用，多个业务之间可能都没有进行分层，之后我们业务多.](https://blog.csdn.net/qq_32828253/article/details/110673205)

[*商品**领域**ddd*_*DDD*之*领域**驱动设计*知多少](https://blog.csdn.net/weixin_39887961/article/details/111644455)

[weixin_39887961的博客](https://blog.csdn.net/weixin_39887961)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 30

[*DDD*之*领域**驱动设计*知多少|0x00 数据团队痛点作为数据*领域*的开发，面对海量的数据、快速迭代的框架，我并不害怕，因为这是我的主攻方向。但我却有一种困惑，与工程团队是相似的，即面对复杂的业务场景，总有一种剪不断理还乱的心情。大部分的需求，当我们吭哧吭哧搞完之后，往往会发现如下的问题：业务逻辑复杂，模块之间耦合度高，没有合适的建模方法；多团队协同，系统间的依赖关系复杂，没有统一建模语言；没...](https://blog.csdn.net/weixin_39887961/article/details/111644455)

[*商品**领域**ddd*_*DDD* *领域**驱动设计*-*商品*建模之路_小樱茉莉...](https://blog.csdn.net/weixin_36448199/article/details/113010783)

4-15

[关于改版的业务设计,还是想尝试 *DDD* *领域**驱动设计*,之前写的一些相关文章,都是直接进行战术设计,而非在战略设计基础上进行,所以最后可能会出现一些问题,所以这次的过程是:边了解业务、边了解 I*DDD* 书中关于战略设计的部分,然后尝试使用战略...](https://blog.csdn.net/weixin_36448199/article/details/113010783)

[*领域**驱动设计*(*DDD*)部分*核心**概念*的个人理解_weixin_3074...](https://blog.csdn.net/weixin_30748995/article/details/98679636)

4-15

[*领域**驱动设计*(*DDD*)是一种基于模型驱动的软件设计方式。它以*领域*为*核心*,分析*领域*中的问题,通过建立一个*领域*模型来有效的解决*领域*中的*核心*的复杂问题。Eric Ivans为*领域**驱动设计*提出了大量的最佳实践和经验技巧。只有对*领域*的不断深入认识,才...](https://blog.csdn.net/weixin_30748995/article/details/98679636)

[*DDD*战略设计相关*核心**概念*的理解](https://blog.csdn.net/qianshangding0708/article/details/111055579)

[qianshanding0708的博客](https://blog.csdn.net/qianshangding0708)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 116

[01前言本文想再讨论一下关于*领域*、业务、业务模型、解决方案、BC、*领域*模型、微服务这些*概念*的含义和关系。初衷是我发现现在*DDD**领域*建模以及解决方案落地过程中，常常对这些*概念*理解不清楚或者...](https://blog.csdn.net/qianshangding0708/article/details/111055579)

[架构设计-*商品*模块的*领域**驱动设计*思路及实现](https://blog.csdn.net/weixiaodedao/article/details/106710341)

[微笑的小小刀](https://blog.csdn.net/weixiaodedao)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 144

[开头先说两句小刀博客:https://www.lixiang.red欢迎留言一起讨论项目背景因脱敏关系,这里面三个角色就用A,B,C来代替,可以抽象理解为, A 是需求发起方,B是平台...](https://blog.csdn.net/weixiaodedao/article/details/106710341)

[*DDD* *领域**驱动设计*-*商品*建模之路](https://blog.csdn.net/weixin_34211761/article/details/90163999)

[weixin_34211761的博客](https://blog.csdn.net/weixin_34211761)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 230

[1. 业务流程 业务场景：发布*商品* 业务场景就上面四个字，看起来很简单，但其实具体分析起来，所包含的东西还是蛮多的，整个发布*商品*过程，就像一个*商品*诞生的生命周期一样，需要经历各个阶段和过程，直到*商品*正式发布出来，并且在这个过程中，有一系列的其他*概念*由*商品*衍生出来，比如库存、分类、品牌等等，还会有一些用户的行为参与，比如小二的后台审核等。 在业务分析过...](https://blog.csdn.net/weixin_34211761/article/details/90163999)

[*DDD**领域**驱动设计**实战*电商活动中心重构](https://javaedge.blog.csdn.net/article/details/113842131)

[JavaEdge全是干货的技术号](https://blog.csdn.net/qq_33589510)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 249

[现实的业务，非常复杂。即使同一事物，在多个业务下意义可能完全不同。比如【*商品*】： 在*商品*详情页语境指【*商品*基本信息】 在下单页语境指【购买项】 在物流页面语境又是【被运送的货物】 *DDD* *核心*思想就是让正确的*领域*模型发挥作用。*DDD* 指导开发将不同子业务单元划分为不同子*领域*，在各个子*领域*内部分别建模应对业务的复杂性。 1 背景 经典的MVC开发，因为初期业务也比较简单，为光速上线， 综合考虑成本和风险，经常创建一个大模型，各个模块都想着直接复用这同一模型。但随业务发展，各子*领域*的逻辑越来越复杂，对该大模](https://javaedge.blog.csdn.net/article/details/113842131)

[*商品**领域**ddd*_*DDD* *领域**驱动设计*-*商品*建模之路](https://blog.csdn.net/weixin_29003437/article/details/113010778)

[weixin_29003437的博客](https://blog.csdn.net/weixin_29003437)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 65

[1. 业务流程业务场景：发布*商品*业务场景就上面四个字，看起来很简单，但其实具体分析起来，所包含的东西还是蛮多的，整个发布*商品*过程，就像一个*商品*诞生的生命周期一样，需要经历各个阶段和过程，直到*商品*正式发布出来，并且在这个过程中，有一系列的其他*概念*由*商品*衍生出来，比如库存、分类、品牌等等，还会有一些用户的行为参与，比如小二的后台审核等。在业务分析过程中，我还是比较喜欢画一张简单的业务流程图，并不一定很...](https://blog.csdn.net/weixin_29003437/article/details/113010778)

[*商品**领域**ddd*_为 Gopher 打造 *DDD* 系列：*领域*模型开篇](https://blog.csdn.net/weixin_39788451/article/details/111644454)

[weixin_39788451的博客](https://blog.csdn.net/weixin_39788451)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 28

[前言： 八叉树是一位拥有数十年编程经验，醉心于代码艺术的工程师。freedom是他结合《实现*领域**驱动设计*》与《六边形架构》两文为一众Gopher打造出最符合*DDD*战术设计的轮子！*DDD*是什么？ *领域**驱动设计*(*DDD*) 做为一种软件工程的方法论，它可以帮助我们设计高质量的软件，或者说任何工程的设计都需要方法论，不论是城市设计、建筑设计、室内设计。比如没有方法论的情况下楼是可以盖起来的，或许...](https://blog.csdn.net/weixin_39788451/article/details/111644454)

[*DDD**领域**驱动设计**实战*(三)-深入理解实体](https://javaedge.blog.csdn.net/article/details/108878702)

[JavaEdge全是干货的技术号](https://blog.csdn.net/qq_33589510)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 603

[实体和值对象都是*领域*模型中的*领域*对象。 实体 拥有唯一标识符，且标识符在历经各种状态变更后仍能保持一致。对这些对象而言，重要的不是其属性，而是其延续性和标识，对象的延续性和标识会跨越甚至超出软件的生命周期。我们把这样的对象称为实体。 实体的业务形态 *DDD*不同设计过程中，实体的形态不同。 在战略设计时，实体是*领域*模型的一个重要对象。*领域*模型中的实体是多个属性、操作或行为的载体。 在事件风暴中，我们可以根据命令、操作或者事件，找出产生这些行为的业务实体对象，进而按照一定的业务规则将依存度高和业务关联紧密的](https://javaedge.blog.csdn.net/article/details/108878702)

[*DDD**领域**驱动设计*(Domain Driven Design)（转）](https://blog.csdn.net/a11101171/article/details/51971483)

[伟大的忠忠的专栏](https://blog.csdn.net/a11101171)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 3563

[摘要本文将介绍*领域**驱动设计*(Domain Driven Design)的官方参考架构，该架构分成了Interfaces、Applications和Domain三层以及包含各类基础设施的 Infrastructure。本文会对架构中一些重要组件和问题进行讨论，给出一些分析结论。目录 架构概述 架构*详解* 2.1. Interfaces-接口层 2.1.1.](https://blog.csdn.net/a11101171/article/details/51971483)

[电商系统中的*商品*模型的分析与设计—续](https://blog.csdn.net/hegaoye308444582/article/details/48448541)

[hegaoye308444582的博客](https://blog.csdn.net/hegaoye308444582)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 3428

[对电商系统*商品*模型有一个粗浅的描述，后来有博友对货品和*商品*的区别以及属性有一些疑问。我也对此做一些研究，再次简单的对*商品*模型做一个介绍。](https://blog.csdn.net/hegaoye308444582/article/details/48448541)

[*DDD**领域**驱动设计**实战*(一)-*领域*模型、子域、*核心*域、通用域和支撑域等基本*概念*](https://javaedge.blog.csdn.net/article/details/108820172)

[JavaEdge全是干货的技术号](https://blog.csdn.net/qq_33589510)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1044

[一些名词在你的微服务设计和开发过程中不一定都用得上，但它可以帮你理解*DDD*的*核心*设计思想和理念。而这些思想和理念，在IT战略设计、业务建模和微服务设计中都是可以借鉴的。 让我们来理清它们与微服务的关系，了解它们在微服务设计中的作用。 *领域*和子域 *领域*用于确定边界，这也是为何*DDD*在设计中不断强调边界。 *DDD*会按规则细分业务*领域*，细分到一定程度后，*DDD*会将问题范围限定在特定边界内，在该边界内建立*领域*模型，进而用代码实现该*领域*模型，解决相应业务问题。 *领域*就是该边界内要解决的业务问题域。*领域*也有大小之分，其](https://javaedge.blog.csdn.net/article/details/108820172)

[*领域**驱动设计*（*DDD*）部分*核心**概念*的个人理解](https://blog.csdn.net/tangxuehua/article/details/45628401)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 128

[*领域**驱动设计*（*DDD*）是一种软件设计的思考方式。它以*领域*为*核心*，分析*领域*中的问题，通过建立一个*领域*模型来有效的解决*领域*中的*核心*的复杂问题。Eric Ivans为*领域**驱动设计*提出了大量的最佳实践和经验技巧。只有对*领域*的不断深入认识，才能得到一个解决*领域**核心*问题的*领域*模型。如果一个应用的复杂性不是在技术方面的，而是在*领域*本身，即*领域*内的业务很复杂，那这种应用，使用*领域**驱动设计*的价值就越大。*领域*驱动开发](https://blog.csdn.net/tangxuehua/article/details/45628401)

[浅析*DDD*(*领域**驱动设计*)](https://blog.csdn.net/kkkkkxiaofei/article/details/62237121)

[追](https://blog.csdn.net/kkkkkxiaofei)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1万+

[最近在做一些微服务相关的设计，内容包括服务的划分，Restful API的设计等。其中比较棘手的就是Service的职责划分：如何抽象具有统一业务范畴的Model，使其模块化，又如何高度提炼并组合多模块，使得业务可独立服务化。为了找寻答案，看了不少书籍和博客，在*DDD*中找到了一些思路，个人觉得受益匪浅，或许也可以受用于大家，特分享于此。 什么是*DDD*软件开发不是一蹴而就的事情，我们不可能在不了解产品](https://blog.csdn.net/kkkkkxiaofei/article/details/62237121)

[可以落地的*DDD*到底长什么样？](https://smile.blog.csdn.net/article/details/81572072)

[纯洁的微笑](https://blog.csdn.net/ityouknow)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 3万+

[*领域**驱动设计*的*概念* 大家都知道软件开发不是一蹴而就的事情，我们不可能在不了解产品(或行业*领域*)的前提下进行软件开发，在开发前通常需要进行大量的业务知识梳理，然后才能到软件...](https://smile.blog.csdn.net/article/details/81572072)

[最近很火的*DDD*(*领域**驱动设计*)，在微服务中怎么使用？](https://jsldl.blog.csdn.net/article/details/100082427)

[技术领导力](https://blog.csdn.net/yellowzf3)

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 1012

[许多朋友给挨踢哥留言，想深入学习*DDD*，以及如何在微服务设计中应用。本文先来探讨*DDD*。什么是*DDD*？ *领域*建模的基础是要先理解*领域*，让自己成为*领域*专家。如...](https://jsldl.blog.csdn.net/article/details/100082427)

©️2020 CSDN 皮肤主题: 数字20 设计师:CSDN官方博客 [返回首页](https://blog.csdn.net/)

- [关于我们](https://www.csdn.net/company/index.html#about)
- [招贤纳士](https://www.csdn.net/company/index.html#recruit)
- [广告服务](https://www.csdn.net/company/index.html#advertisement)
- [开发助手](https://plugin.csdn.net/)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/tel.png)400-660-0108
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/email.png)[kefu@csdn.net](mailto:webmaster@csdn.net)
- ![img](https://g.csdnimg.cn/common/csdn-footer/images/cs.png)[在线客服](https://csdn.s2.udesk.cn/im_client/?web_plugin_id=29181)
- 工作时间 8:30-22:00

- [公安备案号11010502030143](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11010502030143)
- [京ICP备19004658号](http://beian.miit.gov.cn/publish/query/indexFirst.action)
- [京网文〔2020〕1039-165号](https://csdnimg.cn/release/live_fe/culture_license.png)
- [经营性网站备案信息](https://csdnimg.cn/cdn/content-toolbar/csdn-ICP.png)
- [北京互联网违法和不良信息举报中心](http://www.bjjubao.org/)
- [网络110报警服务](http://www.cyberpolice.cn/)
- [中国互联网举报中心](http://www.12377.cn/)
- [家长监护](https://download.csdn.net/index.php/tutelage/)
- [Chrome商店下载](https://chrome.google.com/webstore/detail/csdn开发者助手/kfkdboecolemdjodhmhmcibjocfopejo?hl=zh-CN)
- ©1999-2021北京创新乐知网络技术有限公司
- [版权与免责声明](https://www.csdn.net/company/index.html#statement)
- [版权申诉](https://blog.csdn.net/blogdevteam/article/details/90369522)
- [出版物许可证](https://img-home.csdnimg.cn/images/20210414021151.jpg)
- [营业执照](https://img-home.csdnimg.cn/images/20210414021142.jpg)

[![img](https://profile.csdnimg.cn/C/E/6/3_weixin_39600319)](https://blog.csdn.net/weixin_39600319)

[weixin_39600319](https://blog.csdn.net/weixin_39600319)

码龄4年[![img](https://csdnimg.cn/identity/nocErtification.png) 暂无认证](https://i.csdn.net/#/uc/profile?utm_source=14998968)







- 2万+

  访问

- [![img](https://csdnimg.cn/identity/blog1.png)](https://blog.csdn.net/blogdevteam/article/details/103478461)

  等级

- 29

  积分

- 10

  粉丝

- 7

  获赞

- 0

  评论

- 31

  收藏

![勤写标兵Lv2](https://csdnimg.cn/medal/qixiebiaobing2@240.png)

[私信](https://im.csdn.net/chat/weixin_39600319)

关注

![img](https://csdnimg.cn/cdn/content-toolbar/csdn-sou.png?v=1587021042)

### 热门文章

- [python将列表中的偶数变成平方、奇数不变_编写程序,将列表s=[9,7,8,3,2,1,5,6\]中的偶数变成它的平方,奇数保持不变,运行效果如书上图所示。_学小易找答案... ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 2204](https://blog.csdn.net/weixin_39600319/article/details/111715995)
- [er图转换为关系模型的方法_数据库建模方法论 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 943](https://blog.csdn.net/weixin_39600319/article/details/111218754)
- [python负数的表示方法_python判断正负数方式 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 891](https://blog.csdn.net/weixin_39600319/article/details/110735668)
- [判断该分解是否保持函数依赖性_经验模态分解及其模态混叠消除的研究进展 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 773](https://blog.csdn.net/weixin_39600319/article/details/110937622)
- [python同时输入多个字符串_python如何连接多个字符串？ ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/readCountWhite.png) 731](https://blog.csdn.net/weixin_39600319/article/details/110959797)

### 最新文章

- [如何安装suse linux操作系统,如何运行openSUSE？Win10中安装SUSE Linux子系统的详细图文教程...](https://blog.csdn.net/weixin_39600319/article/details/116579507)
- [linux下qq安装目录,ubuntu上安装QQ(包括多个软件安装方法)](https://blog.csdn.net/weixin_39600319/article/details/116577104)
- [php如何解析crontab命令,解析Ubuntu下crontab命令的用法](https://blog.csdn.net/weixin_39600319/article/details/116446257)

2021

[04月7篇](https://blog.csdn.net/weixin_39600319/article/month/2021/04)

[03月15篇](https://blog.csdn.net/weixin_39600319/article/month/2021/03)

[02月30篇](https://blog.csdn.net/weixin_39600319/article/month/2021/02)

[01月30篇](https://blog.csdn.net/weixin_39600319/article/month/2021/01)

[2020年176篇](https://blog.csdn.net/weixin_39600319/article/month/2020/12)

![img](https://kunyu.csdn.net/1.png?p=57&a=2488&c=0&k=&spm=1001.2101.3001.5001&d=1&t=3&u=da0a4b0bd60740f4acfcea2f48ff2441)



![img](https://g.csdnimg.cn/side-toolbar/3.0/images/guide.png)![img](https://g.csdnimg.cn/side-toolbar/3.0/images/kefu.png)举报![img](https://g.csdnimg.cn/side-toolbar/3.0/images/fanhuidingbucopy.png)