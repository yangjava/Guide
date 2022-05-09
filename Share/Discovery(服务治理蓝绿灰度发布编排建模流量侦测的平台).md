# Discovery

**Nepxion Discovery is a solution for Spring Cloud with blue green, gray, weight, limitation, circuit breaker, degrade, isolation, tracing, dye, failover** 

官网介绍：蓝绿、灰度、权重、限流、熔断、降级、隔离、追踪、流量染色、故障转移

项目地址：https://gitee.com/nepxion/Discovery?_from=gitee_search

Github地址：https://github.com/Nepxion/DiscoveryGuide

## 简介

### 诞生故事

- 2017年12月开始筹划
- 2018年03月开始编码
- 2018年06月在GitHub开源
- 2018年06月发布v1.0.0，支持Camden版
- 2018年06月发布v2.0.0，支持Dalston版
- 2018年07月发布v3.0.0，支持Edgware版
- 2018年07月发布v4.0.0，支持Finchley版
- 2019年04月发布v5.0.0，支持Greenwich版
- 2020年04月发布v6.0.0，支持Hoxton版
- 2021年04月完成v7.0.0，支持2020版
- 2022年04月完成v8.0.0，支持2021版

### 功能概述

Discovery【探索】微服务框架，基于Spring Cloud & Spring Cloud Alibaba，Discovery服务注册发现、Ribbon & Spring Cloud LoadBalancer负载均衡、Feign & RestTemplate & WebClient调用、Spring Cloud Gateway & Zuul过滤等组件全方位增强的企业级微服务开源解决方案，更贴近企业级需求，更具有企业级的插件引入、开箱即用特征

① 微服务框架支持的基本功能，如下

- 支持阿里巴巴Spring Cloud Alibaba中间件生态圈
- 支持阿里巴巴Nacos、Eureka、Consul和Zookeeper四个服务注册发现中心
- 支持阿里巴巴Nacos、携程Apollo、Redis、Zookeeper、Consul和Etcd六个远程配置中心
- 支持阿里巴巴Sentinel、Hystrix和Resilience4J三个熔断限流降级权限中间件
- 支持OpenTracing和OpenTelemetry规范下的调用链中间件，Jaeger、SkyWalking和Zipkin等
- 支持Prometheus Micrometer和Spring Boot Admin两个指标中间件
- 支持Java Agent解决异步跨线程ThreadLocal上下文传递
- 支持Spring Spel解决蓝绿灰度参数的驱动逻辑
- 支持Spring Matcher解决元数据匹配的通配逻辑
- 支持Spring Cloud Gateway、Zuul网关和微服务三大模块的蓝绿灰度发布等一系列功能
- 支持和兼容Spring Cloud Edgware版、Finchley版、Greenwich版、Hoxton版和202x版以及更高的Spring Cloud版本
- 支持和兼容Java8～Java16以及更高的SDK版本

② 微服务框架支持的应用功能，如下

- 全链路蓝绿灰度发布
  - 全链路版本、区域、 IP地址和端口匹配蓝绿发布
  - 全链路版本、区域、 IP地址和端口权重灰度发布
  - 全链路蓝 | 绿 | 兜底、蓝 | 兜底的蓝绿路由类型
  - 全链路稳定、灰度的灰度路由类型
  - 全链路网关、服务端到端混合蓝绿灰度发布
  - 全链路域网关、非域网关部署
  - 全链路条件驱动、非条件驱动
  - 全链路前端触发后端蓝绿灰度发布
  - 全局订阅式蓝绿灰度发布
  - 全链路自定义网关、服务的过滤器、负载均衡策略类触发蓝绿灰度发布
  - 全链路动态变更元数据的蓝绿灰度发布
  - 全链路Header、Parameter、Cookie、域名、RPC Method等参数化规则策略驱动
  - 全链路本地和远程、局部和全局无参数化规则策略驱动
  - 全链路条件表达式、通配表达式支持
  - 全链路内置Header，支持定时Job的服务调用蓝绿灰度发布
- 全链路蓝绿灰度发布编排建模和流量侦测
  - 全链路蓝绿发布编排建模
  - 全链路灰度发布编排建模
  - 全链路蓝绿发布流量侦测
  - 全链路灰度发布流量侦测
  - 全链路蓝绿灰度发布混合流量侦测
- 全链路蓝绿灰度发布容灾
  - 发布失败下的版本故障转移
  - 并行发布下的版本偏好
- 服务无损下线，实时性的流量绝对无损
  - 全局唯一ID屏蔽
  - IP地址和端口屏蔽
- 异步场景下全链路蓝绿灰度发布
  - 异步跨线程Agent插件
  - Hystrix线程池隔离插件
- 全链路数据库和消息队列蓝绿发布
  - 基于多DataSource的数据库蓝绿发布
  - 基于多Queue的消息队列蓝绿发布
- 网关动态路由
  - 路由动态添加
  - 路由动态修改
  - 路由动态删除
  - 路由动态批量更新
  - 路由查询
  - 路由动态变更后的事件通知
- 统一配置订阅执行器
- 全链路规则策略推送
  - 基于远程配置中心的规则策略订阅推送
  - 基于Swagger和Rest的规则策略推送
  - 基于平台端和桌面端的规则策略推送
- 全链路环境隔离和路由
  - 全链路环境隔离
  - 全链路环境路由
- 全链路可用区亲和性隔离和路由
  - 全链路可用区亲和性隔离
  - 全链路可用区亲和性路由
- 全链路服务隔离和准入
  - 消费端服务隔离
  - 提供端服务隔离
  - 注册发现隔离和准入
- 全链路服务限流熔断降级权限
  - Sentinel基于服务名的防护
  - Sentinel基于组的防护
  - Sentinel基于版本的防护
  - Sentinel基于区域的防护
  - Sentinel基于环境的防护
  - Sentinel基于可用区的防护
  - Sentinel基于IP地址和端口的防护
  - Sentinel自定义Header、Parameter、Cookie的防护
  - Sentinel自定义业务参数的防护
  - Sentinel自定义组合式的防护
- 全链路监控
  - 蓝绿灰度埋点和熔断埋点的调用链监控
  - 蓝绿灰度埋点和熔断埋点的日志监控
  - 熔断埋点的指标监控
- 全链路服务侧注解
- 全链路服务侧API权限
- 元数据流量染色
  - Git插件自动化的元数据流量染色
  - 服务名前缀的元数据流量染色
  - 运维平台参数化的元数据流量染色
  - 注册中心动态化的元数据流量染色
  - 用户自定义的元数据流量染色
- 多活、多云、多机房流量切换
- Docker容器化和Kubernetes平台无缝支持部署
- 自动化测试、压力测试

③ 微服务框架易用性表现，如下

- 引入相关依赖到pom.xml
- 元数据Metadata流量染色。5大元数据根据不同的使用场景按需设置
  - 定义所属组名 - metadata.group，也可以通过服务名前缀来自动产生服务组名
  - 定义版本号 - metadata.version，也可以通过Git插件方式自动产生版本号
  - 定义所属区域名 - metadata.region
  - 定义所属环境 - metadata.env
  - 定义所属可用区 - metadata.zone
- 执行采用【约定大于配置】的准则，使用者根据不同的使用场景开启和关闭相关功能项或者属性值，达到最佳配置
- 规则策略文件设置和推送，或者通过业务Header、Parameter、Cookie触发，并通过Json格式的Header路由策略全链路传递

## 蓝绿灰度发布概念

### 蓝绿发布

蓝绿发布 Blue-Green Deployment

① 概念

不停机旧版本，部署新版本，通过用户标记将流量在新版本和老版本切换，属无损发布

② 优点

新版本升级和老版本回滚迅速。用户可以灵活控制流量走向

③ 缺点

成本较高，需要部署两套环境（蓝/绿）。新版本出现问题，切换不及时，会造成大面积故障

### 灰度发布

灰度发布 Gray Release（又名金丝雀发布 Canary Release）

① 概念

不停机旧版本，部署新版本，低比例流量（例如：5%）切换到新版本，高比例流量（例如：95%）走旧版本，通过监控观察无问题，逐步扩大范围，最终把所有流量都迁移到新版本上，下线旧版本。属无损发布

② 优点

灵活简单，不需要用户标记驱动。安全性高，新版本如果出现问题，只会发生在低比例的流量上

③ 缺点

流量配比递增的配置修改，带来额外的操作成本。用户覆盖狭窄，低比例流量未必能发现所有问题

### 滚动发布

滚动发布 Rolling Release

① 概念

每次只升级一个或多个服务，升级完成监控观察，不断执行这个过程，直到集群中的全部旧版本升级到新版本。停止旧版本的过程中，无法精确计算旧版本是否已经完成它正在执行的工作，需要靠业务自身去判断。属有损发布

② 优点

出现问题影响范围很小，只会发生在若干台正在滚动发布的服务上

③ 缺点

发布和回滚需要较长的时间周期。按批次停止旧版本，启动新版本，由于旧版本不保留，一旦全部升级完毕后才发现问题，则无法快速回滚，必须重新降级部署

## 全链路蓝绿灰度发布

### 全链路蓝绿发布

> 经典场景：当调用请求从网关或者服务发起的时候，通过Header | Parameter | Cookie一种或者几种参数进行驱动，在路由过滤中，根据这些参数，选择在配置中心配置的蓝路由 | 绿路由 | 兜底路由的规则策略（Json格式），并把命中的规则策略转化为策略路由Header（n-d-开头），实现全链路传递。每个端到端服务接收到策略路由Header后，执行负载均衡时，该Header跟注册中心的对应元数据进行相关比较，不符合条件的实例进行过滤，从而实现全链路蓝绿发布

> 实施概要：只涉及当前正在发布的服务，例如，对于 〔网关〕->〔A服务〕->〔B服务〕->〔C服务〕->〔D服务〕调用链来说，如果当前只是B服务和C服务正在实施发布，那么，只需要把B服务和C服务配置到规则策略中，其它则不需要配置。发布结束后，即B服务和C服务的所有实例都完全一致，例如，版本号都只有唯一一个，那么清除掉在配置中心配置的规则策略即可，从而进行下一轮全链路蓝绿发布

### 全链路灰度发布

> 经典场景：当调用请求从网关或者服务发起的时候，在路由过滤中，根据在配置中心配置的随机权重值，执行权重算法，选择灰度路由 | 稳定路由的规则策略（Json格式），并把命中的规则策略转化为策略路由Header（n-d-开头），实现全链路传递。每个端到端服务接收到策略路由Header后，执行负载均衡时，该Header跟注册中心的对应元数据进行相关比较，不符合条件的实例进行过滤，从而实现全链路灰度发布

> 实施概要：只涉及当前正在发布的服务，例如，对于 〔网关〕->〔A服务〕->〔B服务〕->〔C服务〕->〔D服务〕调用链来说，如果当前只是B服务和C服务正在实施发布，那么，只需要把B服务和C服务配置到规则策略中，其它则不需要配置。发布结束后，即B服务和C服务的所有实例都完全一致，例如，版本号都只有唯一一个，那么清除掉在配置中心配置的规则策略即可，从而进行下一轮全链路灰度发布

## 异步场景下全链路蓝绿灰度发布

Discovery框架存在着如下全链路传递上下文的场景，包括

- 策略路由Header全链路从网关传递到服务
- 调用链埋点全链路从网关传递到服务
- 业务自定义的上下文的传递

上述上下文会在如下异步场景中丢失，包括

- WebFlux Reactor响应式异步
- Spring异步，@Async注解异步
- Hystrix线程池隔离模式异步
- 线程，线程池异步
- SLF4J日志异步

通过DiscoveryAgent，解决上述痛点。Discovery框架利用DiscoveryAgent字节码增强技术，完美解决各种调用场景下的异步，包括

- Spring Cloud Gateway过滤器中的上下文传递
- Zuul过滤器中的上下文传递
- Feign拦截器中的上下文转发
- RestTemplate拦截器中的上下文转发
- WebClient拦截器中的上下文转发

DiscoveryAgent不仅适用于Discovery框架，也适用于一切具有类似使用场景的基础框架（例如：Dubbo）和业务系统

ThreadLocal的作用是提供线程内的局部变量，在多线程环境下访问时能保证各个线程内的ThreadLocal变量各自独立。在异步场景下，由于出现线程切换的问题，例如，主线程切换到子线程，会导致线程ThreadLocal上下文丢失。DiscoveryAgent通过Java Agent方式解决这些痛点

涵盖所有Java框架的异步场景，解决如下8个异步场景下丢失线程ThreadLocal上下文的问题

- WebFlux Reactor
- `@`Async
- Hystrix Thread Pool Isolation
- Runnable
- Callable
- Single Thread
- Thread Pool
- SLF4J MDC

## 全链路环境隔离和路由

基于服务实例的元数据Metadata的env参数和全链路传递的环境Header值进行对比实现隔离，当从网关传递来的环境Header（n-d-env）值和提供端实例的元数据Metadata环境配置值相等才能调用。环境隔离下，调用端实例找不到符合条件的提供端实例，把流量路由到一个通用或者备份环境

### 全链路环境隔离

在网关或者服务端，配置环境元数据，在同一套环境下，env值必须是一样的，这样才能达到在同一个注册中心下，环境隔离的目的

```
spring.cloud.nacos.discovery.metadata.env=env1
```

### 全链路环境路由

在环境隔离执行的时候，如果无法找到对应的环境，则会路由到一个通用或者备份环境，默认为env为common的环境，可以通过如下参数进行更改

```
# 流量路由到指定的环境下。不允许为保留值default，缺失则默认为common
spring.application.strategy.environment.route=common
```

需要注意

- 如果存在环境，优先寻址环境的服务实例
- 如果不存在环境，则寻址Common环境的服务实例（未设置元数据Metadata的env参数的服务实例也归为Common环境）
- 如果Common环境也不存在，则调用失败
- 如果没有传递环境Header（n-d-env）值，则执行Spring Cloud Ribbon轮询策略
- 环境隔离和路由适用于测试环境，性能压测等场景

##  全链路可用区亲和性隔离和路由

### 全链路可用区亲和性隔离

基于调用端实例和提供端实例的元数据Metadata的zone配置值进行对比实现隔离

```
spring.cloud.nacos.discovery.metadata.zone=zone
```

通过如下开关进行开启和关闭

```
# 启动和关闭可用区亲和性，即同一个可用区的服务才能调用，同一个可用区的条件是调用端实例和提供端实例的元数据Metadata的zone配置值必须相等。缺失则默认为false
spring.application.strategy.zone.affinity.enabled=false
```

### 全链路可用区亲和性路由

在可用区亲和性隔离执行的时候，调用端实例找不到同一可用区的提供端实例，把流量路由到其它可用区或者不归属任何可用区

通过如下开关进行开启和关闭

```
# 启动和关闭可用区亲和性失败后的路由，即调用端实例没有找到同一个可用区的提供端实例的时候，当开关打开，可路由到其它可用区或者不归属任何可用区，当开关关闭，则直接调用失败。缺失则默认为true
spring.application.strategy.zone.route.enabled=true
```

需要注意

- 不归属任何可用区，含义是服务实例未设置任何zone元数据值。可用区亲和性路由功能，是为了尽量保证流量不损失
- 本框架提供的可用区亲和性功能适用于一切注册中心
- 如果采用Eureka注册中心，Ribbon在Eureka Client上会自动开启可用区亲和性功能，跟本框架提供的功能相似。它不提供禁止“可用区亲和性失败后的路由”，如果使用者希望实现“找不到相同可用区，直接调用失败”的功能，可以结合本框架上述两个开关来实现

## 全链路服务隔离和准入

### 消费端服务隔离

#### 基于组负载均衡隔离

元数据中的Group在一定意义上代表着系统ID或者系统逻辑分组，基于Group策略意味着只有同一个系统中的服务才能调用

基于Group是否相同的策略，即消费端拿到的提供端列表，两者的Group必须相同。只需要在网关或者服务端，开启如下配置即可

```
# 启动和关闭消费端的服务隔离（基于Group是否相同的策略）。缺失则默认为false
spring.application.strategy.consumer.isolation.enabled=true
```

通过修改discovery-guide-service-b的Group名为其它名称，执行Postman调用，将发现从discovery-guide-service-a无法拿到discovery-guide-service-b的任何实例。意味着在discovery-guide-service-a消费端进行了隔离

### 提供端服务隔离

#### 基于组Header传值策略隔离

元数据中的Group在一定意义上代表着系统ID或者系统逻辑分组，基于Group策略意味着只有同一个系统中的服务才能调用

基于Group是否相同的策略，即服务端被消费端调用，两者的Group必须相同，否则拒绝调用，异构系统可以通过Header方式传递n-d-service-group值进行匹配。只需要在服务端（不适用网关），开启如下配置即可

```
# 启动和关闭提供端的服务隔离（基于Group是否相同的策略）。缺失则默认为false
spring.application.strategy.provider.isolation.enabled=true

# 路由策略的时候，需要指定对业务RestController类的扫描路径。此项配置作用于RPC方式的调用拦截和消费端的服务隔离两项工作
spring.application.strategy.scan.packages=com.nepxion.discovery.guide.service.feign
```

### 注册发现隔离和准入

#### 基于IP地址黑白名单注册准入

微服务启动的时候，禁止指定的IP地址注册到注册中心。支持黑/白名单，白名单表示只允许指定IP地址前缀注册，黑名单表示不允许指定IP地址前缀注册

- 全局过滤，指注册到服务注册发现中心的所有微服务，只有IP地址包含在全局过滤字段的前缀中，都允许注册（对于白名单而言），或者不允许注册（对于黑名单而言）
- 局部过滤，指专门针对某个微服务而言，那么真正的过滤条件是全局过滤 + 局部过滤结合在一起

#### 基于最大注册数限制注册准入

微服务启动的时候，一旦微服务集群下注册的实例数目已经达到上限（可配置），将禁止后续的微服务进行注册

- 全局配置值，只下面配置所有的微服务集群，最多能注册多少个
- 局部配置值，指专门针对某个微服务而言，如果该值如存在，全局配置值失效

#### 基于IP地址黑白名单发现准入

微服务启动的时候，禁止指定的IP地址被服务发现。它使用的方式和[基于IP地址黑白名单注册准入](https://gitee.com/nepxion/Discovery?_from=gitee_search#基于IP地址黑白名单注册准入)一致

#### 自定义注册发现准入

- 集成AbstractRegisterListener，实现自定义禁止注册
- 集成AbstractDiscoveryListener，实现自定义禁止被发现。需要注意，在Consul下，同时会触发service和management两个实例的事件，需要区别判断
- 集成AbstractLoadBalanceListener，实现自定义禁止被负载均衡

## 全链路服务限流熔断降级权限

集成Sentinel熔断隔离限流降级平台

通过集成Sentinel，在服务端实现该功能

Sentinel订阅配置中心的使用方式，如下

- Key为
  - Nacos、Redis、Zookeeper配置中心，Group为{group}，DataId为{serviceId}-{规则类型}
  - Apollo、Consul、Etcd配置中心，Key的格式为{group}-{serviceId}-{规则类型}
  - {group}为注册中心元数据group值
- Value为Json格式的规则

支持远程配置中心和本地规则文件的读取逻辑，即优先读取远程配置，如果不存在或者规则错误，则读取本地规则文件。动态实现远程配置中心对于规则的热刷新

支持如下开关开启该动能，默认是关闭的

```
# 启动和关闭Sentinel限流降级熔断权限等原生功能的数据来源扩展。缺失则默认为false
spring.application.strategy.sentinel.datasource.enabled=true
```

### 原生Sentinel注解

参照下面代码，为接口方法增加@SentinelResource注解，value为sentinel-resource，blockHandler和fallback是防护其作用后需要执行的方法

```
@RestController
@ConditionalOnProperty(name = DiscoveryConstant.SPRING_APPLICATION_NAME, havingValue = "discovery-guide-service-b")
public class BFeignImpl extends AbstractFeignImpl implements BFeign {
    private static final Logger LOG = LoggerFactory.getLogger(BFeignImpl.class);

    @Override
    @SentinelResource(value = "sentinel-resource", blockHandler = "handleBlock", fallback = "handleFallback")
    public String invoke(@PathVariable(value = "value") String value) {
        value = doInvoke(value);

        LOG.info("调用路径：{}", value);

        return value;
    }

    public String handleBlock(String value, BlockException e) {
        return value + "-> B server sentinel block, cause=" + e.getClass().getName() + ", rule=" + e.getRule() + ", limitApp=" + e.getRuleLimitApp();
    }

    public String handleFallback(String value) {
        return value + "-> B server sentinel fallback";
    }
}
```

### 原生Sentinel规则

原生Sentinel规则的用法，请参照Sentinel官方文档

#### 流控规则

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-flow，规则内容如下

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "refResource": null,
        "controlBehavior": 0,
        "warmUpPeriodSec": 10,
        "maxQueueingTimeMs": 500,
        "clusterMode": false,
        "clusterConfig": null
    }
]
```

#### 降级规则

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-degrade，规则内容如下

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "default",
        "count": 2,
        "timeWindow": 10,
        "grade": 0,
        "passCount": 0
    }
]
```

### 基于Sentinel-LimitApp扩展的防护

该功能对于上面5种规则都有效，这里以授权规则展开阐述

授权规则中，limitApp，如果有多个，可以通过“,”分隔。"strategy": 0 表示白名单，"strategy": 1 表示黑名单

支持如下开关开启该动能，默认是关闭的

```
# 启动和关闭Sentinel LimitApp限流等功能。缺失则默认为false
spring.application.strategy.sentinel.limit.app.enabled=true
```

#### 基于服务名的防护

修改配置项Sentinel Request Origin Key为服务名Header，修改授权规则中limitApp为对应的服务名，可实现基于服务名的防护

配置项，该配置项默认为n-d-service-id，可以不配置

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-id
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示所有discovery-guide-service-a服务允许访问discovery-guide-service-b服务

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "discovery-guide-service-a",
        "strategy": 0
    }
]
```

#### 基于组的防护

修改配置项Sentinel Request Origin Key为组Header，修改授权规则中limitApp为对应的组名，可实现基于组的防护

配置项

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-group
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示隶属my-group组的所有服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "my-group",
        "strategy": 0
    }
]
```

#### 基于版本的防护

修改配置项Sentinel Request Origin Key为版本Header，修改授权规则中limitApp为对应的版本，可实现基于版本的防护机制

配置项

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-version
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示版本为1.0的所有服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "1.0",
        "strategy": 0
    }
]
```

#### 基于区域的防护

修改配置项Sentinel Request Origin Key为区域Header，修改授权规则中limitApp为对应的区域，可实现基于区域的防护

配置项

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-region
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示区域为dev的所有服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "dev",
        "strategy": 0
    }
]
```

#### 基于环境的防护

修改配置项Sentinel Request Origin Key为环境Header，修改授权规则中limitApp为对应的环境，可实现基于环境的防护

配置项

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-env
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示环境为env1的所有服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "env1",
        "strategy": 0
    }
]
```

#### 基于可用区的防护

修改配置项Sentinel Request Origin Key为可用区Header，修改授权规则中limitApp为对应的可用区，可实现基于可用区的防护

配置项

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-zone
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示可用区为zone1的所有服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "zone1",
        "strategy": 0
    }
]
```

#### 基于IP地址和端口的防护

修改配置项Sentinel Request Origin Key为IP地址和端口Header，修改授权规则中limitApp为对应的区域值，可实现基于IP地址和端口的防护

配置项

```
spring.application.strategy.sentinel.request.origin.key=n-d-service-address
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示地址和端口为192.168.0.88:8081和192.168.0.88:8082的服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "192.168.0.88:8081,192.168.0.88:8082",
        "strategy": 0
    }
]
```

#### 自定义组合式的防护

通过适配类实现自定义组合式的防护，支持自定义Header、Parameter、Cookie参数的防护，自定义业务参数的防护，以及自定义前两者组合式的防护

```
// 自定义版本号+地域名，实现组合式熔断
public class MySentinelStrategyRequestOriginAdapter extends DefaultSentinelStrategyRequestOriginAdapter {
    @Override
    public String parseOrigin(HttpServletRequest request) {
        String version = request.getHeader(DiscoveryConstant.N_D_SERVICE_VERSION);
        String location = request.getHeader("location");

        return version + "&" + location;
    }
}
```

在配置类里@Bean方式进行适配类创建

```
@Bean
public SentinelStrategyRequestOriginAdapter sentinelStrategyRequestOriginAdapter() {
    return new MySentinelStrategyRequestOriginAdapter();
}
```

增加服务discovery-guide-service-b的规则，Group为discovery-guide-group，Data Id为discovery-guide-service-b-sentinel-authority，规则内容如下，表示版本为1.0且传入Header的location=shanghai，同时满足这两个条件下的所有服务都允许访问服务discovery-guide-service-b

```
[
    {
        "resource": "sentinel-resource",
        "limitApp": "1.0&shanghai",
        "strategy": 0
    }
]
```

## 全链路监控

### 全链路调用链监控

### 全链路日志监控

###  全链路指标监控

###  全链路告警监控

# DiscoveryPlatform

**Nepxion DiscoveryPlatform is a platform for Nepxion Discovery with service governance, blue green and gray release orchestration, modelling, flow inspection** 

服务治理、蓝绿灰度发布编排建模、流量侦测的平台

## 简介

### 功能概述

Nepxion Discovery Platform基于Nepxion Discovery 6.x.x版和Spring Cloud Hoxton版制作，也支持和兼容Spring Cloud Edgware版 ~ 202x版接入，支持如下功能

- 支持四个注册中心
- 支持六个配置中心
- 支持MySQL数据库和H2内存数据库，用户可以无缝扩展到其它数据库（例如，Oracle）
- 支持数据库方式登录和Ldap方式登录
- 支持Shiro和JWT的登录以及鉴权
- 支持管理员/角色/权限配置
- 支持页面配置，在线添加、删除、修改各类中间件主页或者业务系统主页的集成以及跳转
- 支持蓝绿灰度链路编排
  - 支持链路单写数据，采用类似Apollo版本控制模式，界面标识增/删/改标识，通过发布方式达到数据库和配置中心最终数据一致性
  - 支持版本和区域维度链路编排
- 支持蓝绿灰度混合发布
  - 支持蓝绿灰度策略双写数据库和配置中心，采用类似Apollo版本控制模式，界面标识增/删/改标识，通过发布方式达到数据库和配置中心最终数据一致性
  - 支持版本和区域维度蓝绿灰度
  - 支持蓝绿灰度策略启用/禁用模式
  - 支持蓝绿灰度策略多实例动态路由一致性检查
  - 支持网关、服务、组为入口
  - 支持全局兜底、蓝绿兜底、灰度兜底策略编排
  - 支持无限级蓝绿灰度策略编排
  - 支持自定义蓝绿条件策略
  - 支持蓝绿条件策略校验
  - 支持内置Header
- 支持双网关动态路由
  - 支持网关动态路由双写数据库和配置中心，采用类似Apollo版本控制模式，界面标识增/删/改标识，通过发布方式达到数据库和配置中心最终数据一致性
  - 支持网关动态路由启用/禁用模式
  - 支持网关动态路由多实例一致性检查
  - 支持Spring Cloud Gateway内置断言器（基于Path、Host、Header、Cookie、Query、Method、RemoteAddr、Weight等无代码方式）和过滤器（基于StripPrefix、PrefixPath、RewritePath、RequestRateLimiter、CircuitBreaker、AddRequestHeader、AddRequestParameter、AddResponseHeader、RedirectTo等无代码方式）
  - 支持用户自定义断言器和过滤器，可以实现类似Access Token、网页访问黑/白名单，自定义用户数据（List和Map结构）过滤等低代码方式
  - 支持Zuul网关内置动态路由
- 支持服务负载屏蔽的黑名单实例摘除
  - 支持黑名单双写数据库和配置中心，采用类似Apollo版本控制模式，界面标识增/删/改标识，通过发布方式达到数据库和配置中心最终数据一致性
  - 支持黑名单启用/禁用模式
  - 支持黑名单多实例一致性检查
  - 基于时间戳前缀的全局唯一ID黑名单
  - 基于IP地址和端口黑名单
- 支持界面显示所连的注册中心和配置中心

请访问https://github.com/Nepxion/DiscoveryPlatform获取源码和示例

### 链路编排

链路编排，即在全链路蓝绿发布或者灰度发布的过程中，把若干个服务的实例按照版本/区域维度实现编排成N个逻辑链路。举个例子，根据生产环境上服务的版本新旧，编排成新版本链路和旧版本链路，供蓝绿发布进行条件驱动，或者供灰度发布进行百分比驱动

链路编排功能引入，具有如下的意义

- 蓝绿发布和灰度发布混合实施的时候，编排的链路可以供两种发布规则策略共享
- 蓝绿发布和灰度发布结束后，使用者不需要删除发布的规则策略，而是禁用它，以便于下一轮发布，不再重复配置，只需要开启它，并发布即可
- 蓝绿发布和灰度发布下一轮开始时，条件策略等未改变，只是改变链路中的服务列表，使用者只需要直接编辑链路编排中的相关链路，并发布即可

# DiscoveryAgent

Nepxion Discovery Agent is a java agent to resolve loss of ThreadLocal in cross-thread scenario, such as Spring Async、Hystrix Thread、Runnable、Callable、Single Thread、Thread Pool、MDC 异步跨线程Agent