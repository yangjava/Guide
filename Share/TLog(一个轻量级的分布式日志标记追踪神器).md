# TLog

一个轻量级的分布式日志标记追踪神器，10分钟即可接入，自动对日志打标签完成微服务的链路追踪

官网：https://gitee.com/dromara/TLog

## 概述

TLog是一个轻量级的分布式日志标记追踪神器，10分钟即可接入，自动对日志打标签完成微服务的链路追踪。支持log4j，log4j2，logback三大日志框架，支持dubbo，dubbox，springcloud三大RPC框架

## 轻量级，但是很强大

TLog通过对日志打标签完成企业级微服务的日志追踪。它不收集日志，使用简单， 产生全局唯一追踪码。除了追踪码以外，TLog还支持SpanId和上下游服务信息标签的追加。你还可以自定义方法级别的标签，让日志的定位轻而易举。

## 10分钟即可接入TLog

为用户使用方便而设计，提供完全零侵入式接入方式，自动探测项目中使用的RPC框架和日志框架， 进行字节码的注入完成系统级日志标签的追加。你的项目用不到10分钟即可接入TLog。

## 适配主流RPC和日志框架

TLog适配了市面上主流的RPC框架：dubbo，dubbox，spring cloud的open feign。 同时适配了三大主流Log框架：log4j,logback,log4j2。支持springboot(1.5.X~2.X)

## 稳定，快速

TLog提供Javaagent，字节码注入，日志框架适配三种接入模式，无论是哪一种，都保证了无性能损耗。支持在业务异步线程，线程池， 日志异步输出这几种场景下追踪不中断。

https://tlog.yomahub.com/

## 项目官网请点击：[项目官网](https://gitee.com/link?target=http%3A%2F%2Fyomahub.com%2Ftlog)

## 项目文档请点击：[项目文档](https://gitee.com/link?target=https%3A%2F%2Fyomahub.com%2Ftlog%2Fdocs)

## 示例工程请点击：[示例工程](https://gitee.com/bryan31/tlog-example)

## 特性

- 通过对日志打标签完成轻量级微服务日志追踪
- 提供三种接入方式：javaagent完全无侵入接入，字节码一行代码接入，基于配置文件的接入
- 对业务代码无侵入式设计，使用简单，10分钟即可接入
- 支持常见的log4j，log4j2，logback三大日志框架，并提供自动检测，完成适配
- 支持dubbo，dubbox，springcloud三大RPC框架
- 支持Spring Cloud Gateway和Soul网关
- 适配HttpClient和Okhttp的http调用标签传递
- 支持三种任务框架，JDK的TimerTask，Quartz，XXL-JOB
- 支持日志标签的自定义模板的配置，提供多个系统级埋点标签的选择
- 支持异步线程的追踪，包括线程池，多级异步线程等场景
- 几乎无性能损耗，快速稳定，经过压测，损耗在0.01%

# TLog能解决什么痛点

随着微服务盛行，很多公司都把系统按照业务边界拆成了很多微服务，在排错查日志的时候。因为业务链路贯穿着很多微服务节点，导致定位某个请求的日志以及上下游业务的日志会变得有些困难。

这时候很多童鞋会开始考虑上SkyWalking，Pinpoint等分布式追踪系统来解决，基于OpenTracing规范，而且通常都是无侵入性的，并且有相对友好的管理界面来进行链路Span的查询。

但是搭建分布式追踪系统，熟悉以及推广到全公司的系统需要一定的时间周期，而且当中涉及到链路span节点的存储成本问题，全量采集还是部分采集？如果全量采集，就以SkyWalking的存储来举例，ES集群搭建至少需要5个节点。这就需要增加服务器成本。况且如果微服务节点多的话，一天下来产生几十G上百G的数据其实非常正常。如果想保存时间长点的话，也需要增加服务器磁盘的成本。

当然分布式追踪系统是一个最终的解决方案，如果您的公司已经上了分布式追踪系统，那TLog并不适用。

##### INFO

TLog提供了一种最简单的方式来解决日志追踪问题，它不收集日志，也不需要另外的存储空间，它只是自动的对你的日志进行打标签，自动生成TraceId贯穿你微服务的一整条链路。并且提供上下游节点信息。适合中小型企业以及想快速解决日志追踪问题的公司项目使用。

为此TLog适配了三大日志框架，支持自动检测适配。支持dubbo，dubbox，spring cloud三大RPC框架，更重要的是，你的项目接入TLog，可能连十分钟就不需要 ：）

# 项目特性

目前TLog的支持的特性如下：

- 通过对日志打标签完成轻量级微服务日志追踪
- 提供三种接入方式：javaagent完全无侵入接入，字节码一行代码接入，基于配置文件的接入
- 对业务代码无侵入式设计，使用简单，10分钟即可接入
- 支持常见的log4j，log4j2，logback三大日志框架，并提供自动检测，完成适配
- 支持dubbo，dubbox，springcloud三大RPC框架
- 支持Spring Cloud Gateway和Soul网关
- 适配HttpClient和Okhttp的http调用标签传递
- 支持三种任务框架，JDK的TimerTask，Quartz，XXL-JOB
- 支持日志标签的自定义模板的配置，提供多个系统级埋点标签的选择
- 支持异步线程的追踪，包括线程池，多级异步线程等场景
- 几乎无性能损耗，快速稳定，经过压测，损耗在0.01%

# 如何选择你的接入方式

##### INFO

TLog提供三大接入方式，兼容各种各样的项目环境，请对照以下表，选择适合你项目的接入方式

|                            | springboot项目自启动 | 非springboot自启动 | springboot项目外置容器 | 非springboot项目外置容器 |
| -------------------------- | -------------------- | ------------------ | ---------------------- | ------------------------ |
| Javaagent方式              | **适合**             | 不适合             | 不适合                 | 不适合                   |
| 字节码注入方式             | **适合**             | **适合**           | 不适合                 | 不适合                   |
| 日志框架适配器方式(最稳定) | **适合**             | **适合**           | **适合**               | **适合**                 |

**自启动是指由main函数作为项目的启动入口(springboot项目的starter-web这种也属于自启动模式)**

**外置容器是指项目部署在类似tomcat容器中的，tomcat作为外容器，项目部署在webapp目录下的**

**javaagent本质上也是字节码注入方式，只不过是完全无侵入项目的方式。字节码在某些复杂的项目上由于类加载机制的不同，有可能会失效，所以你的项目结构如果很复杂，发现javaagent和字节码方式不起作用的话，那还是推荐日志框架适配器方式，这种相对最稳定**

##### INFO

TLog接入方式对于特性的支持如下表

|                            | 同步日志 | MDC      | 异步日志 |
| -------------------------- | -------- | -------- | -------- |
| Javaagent方式              | **支持** | 不支持   | 不支持   |
| 字节码注入方式             | **支持** | 不支持   | 不支持   |
| 日志框架适配器方式(最稳定) | **支持** | **支持** | **支持** |

# 全量依赖

TLog对springboot和spring native提供了2种不同的依赖，此种方式只需依赖一个包，必须的包会传递依赖进来

**springboot依赖**

```xml
<dependency>  <groupId>com.yomahub</groupId>  <artifactId>tlog-all-spring-boot-starter</artifactId>  <version>1.3.6</version></dependency>
```

Copy

**spring native依赖**

```xml
<dependency>  <groupId>com.yomahub</groupId>  <artifactId>tlog-all</artifactId>  <version>1.3.6</version></dependency>
```

# 按需依赖

如果你不想依赖不必要的包，TLog对springboot提供了按需依赖

模板形式为:

```xml
<dependency>  <groupId>com.yomahub</groupId>  <artifactId>tlog-XXX-spring-boot-starter</artifactId>  <version>1.3.6</version></dependency>
```

Copy

具体模块和描述如下表

| 模块名                               | 描述                                  |
| ------------------------------------ | ------------------------------------- |
| **tlog-dubbo-spring-boot-starter**   | 适用于apache dubbo的项目              |
| **tlog-dubbox-spring-boot-starter**  | 适用于当当的dubbox的项目              |
| **tlog-feign-spring-boot-starter**   | 适用于spring cloud中open feign的项目  |
| **tlog-gateway-spring-boot-starter** | 适用于spring cloud中的gateway网关服务 |
| **tlog-soul-spring-boot-starter**    | 适用于soul网关服务                    |
| **tlog-web-spring-boot-starter**     | 适用于有spring web的项目              |
| **tlog-xxljob-spring-boot-starter**  | 适用于xxl-job的项目                   |

##### INFO

所有模块均包含log4j,log4j2,logback三大日志框架的支持，这些模块也可以组合起来使用，例如spring cloud openfeign同时需要tlog-feign-spring-boot-starter和tlog-web-spring-boot-starter