# Sentinel

## 简介

A powerful flow control component enabling reliability, resilience and monitoring for microservices.

 (面向云原生微服务的高可用流控防护组件)

Sentinel是阿里开源的项目，提供了流量控制、熔断降级、系统负载保护等多个维度来保障服务之间的稳定性。
官网：https://github.com/alibaba/Sentinel/wiki

2012年，Sentinel诞生于阿里巴巴，其主要目标是流量控制。2013-2017年，Sentinel迅速发展，并成为阿里巴巴所有微服务的基本组成部分。 它已在6000多个应用程序中使用，涵盖了几乎所有核心电子商务场景。2018年，Sentinel演变为一个开源项目。2020年，Sentinel Golang发布。

## **特征**

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Apache Dubbo、gRPC、Quarkus 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。同时 Sentinel 提供 Java/Go/C++ 等多语言的原生实现。
- **完善的 SPI 扩展机制**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

## 主要特性

![sentinel](png\sentinel.png)

## Sentinel 的使用

Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

### Maven依赖

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
    <version>1.8.3</version>
</dependency>
```

#### 定义资源

接下来，我们把需要控制流量的代码用 Sentinel API `SphU.entry("HelloWorld")` 和 `entry.exit()` 包围起来即可。在下面的例子中，我们将 `System.out.println("hello world");` 这端代码作为资源，用 API 包围起来（埋点）。参考代码如下:

```
public static void main(String[] args) {
    initFlowRules();
    while (true) {
        Entry entry = null;
        try {
	    entry = SphU.entry("HelloWorld");
            /*您的业务逻辑 - 开始*/
            System.out.println("hello world");
            /*您的业务逻辑 - 结束*/
	} catch (BlockException e1) {
            /*流控逻辑处理 - 开始*/
	    System.out.println("block!");
            /*流控逻辑处理 - 结束*/
	} finally {
	   if (entry != null) {
	       entry.exit();
	   }
	}
    }
}
```

#### 定义规则

接下来，通过规则来指定允许该资源通过的请求次数，例如下面的代码定义了资源 `HelloWorld` 每秒最多只能通过 20 个请求。

```
private static void initFlowRules(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

#### 检查效果

Demo 运行之后，我们可以在日志 `~/{userhome}/logs/csp/${appName}-metrics.log.xxx` 

里看到下面的输出:

```
|--timestamp-|------date time----|-resource-|p |block|s |e|rt
1529998904000|2018-06-26 15:41:44|HelloWorld|20|0    |20|0|0
1529998905000|2018-06-26 15:41:45|HelloWorld|20|5579 |20|0|728
1529998906000|2018-06-26 15:41:46|HelloWorld|20|15698|20|0|0
1529998907000|2018-06-26 15:41:47|HelloWorld|20|19262|20|0|0
1529998908000|2018-06-26 15:41:48|HelloWorld|20|19502|20|0|0
1529998909000|2018-06-26 15:41:49|HelloWorld|20|18386|20|0|0
```

其中 `p` 代表通过的请求, `block` 代表被阻止的请求, `s` 代表成功执行完成的请求个数, `e` 代表用户自定义的异常, `rt` 代表平均响应时长。

可以看到，这个程序每秒稳定输出 "hello world" 20 次，和规则中预先设定的阈值是一样的。

#### 启动 Sentinel 控制台

sentinel_dashboard的引入

https://github.com/alibaba/Sentinel/releases，下载sentinel-dashboard-1.8.3.jar

由于dashboard是springboot的项目，在CMD模式下使用命令

```
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.8.3.jar
```

进行控制看板服务的启动。

-Dserver.port=8080 代表看板项目的端口号，

-Dcsp.sentinel.dashboard.server=localhost:8080代表本看板服务将会注册到自己的看板上，

-Dproject.name=sentinel-dashboard代表本看板服务的项目名称

访问localhost:8080；输入用户名，密码，均是sentinel,如果要自定义用户名和密码，在启动命令加上-Dsentinel.dashboard.auth.username=sentinel，
-Dsentinel.dashboard.auth.password=123456即可。

通过观察控制台，我们发现我们的项目名为app1的项目已经注册到了控制台



## 如何使用

### 简介

Sentinel 可以简单的分为 Sentinel 核心库和 Dashboard。核心库不依赖 Dashboard，但是结合 Dashboard 可以取得最好的效果。

这篇文章主要介绍 Sentinel 核心库的使用。如果希望有一个最快最直接的了解，可以参考 [新手指南](https://github.com/alibaba/Sentinel/wiki/新手指南#公网-demo) 来获取一个最直观的感受。

我们说的资源，可以是任何东西，服务，服务里的方法，甚至是一段代码。使用 Sentinel 来进行资源保护，主要分为几个步骤:

1. 定义资源
2. 定义规则
3. 检验规则是否生效

先把可能需要保护的资源定义好（埋点），之后再配置规则。也可以理解为，只要有了资源，我们就可以在任何时候灵活地定义各种流量控制规则。在编码的时候，只需要考虑这个代码是否需要保护，如果需要保护，就将之定义为一个资源。

对于主流的框架，我们提供适配，只需要按照适配中的说明配置，Sentinel 就会默认定义提供的服务，方法等为资源。

### 定义资源

#### 方式一：主流框架的默认适配

为了减少开发的复杂程度，我们对大部分的主流框架，例如 Web Servlet、Dubbo、Spring Cloud、gRPC、Spring WebFlux、Reactor 等都做了适配。您只需要引入对应的依赖即可方便地整合 Sentinel。

#### 方式二：抛出异常的方式定义资源

`SphU` 包含了 try-catch 风格的 API。用这种方式，当资源发生了限流之后会抛出 `BlockException`。这个时候可以捕捉异常，进行限流之后的逻辑处理。示例代码如下:

```
// 1.5.0 版本开始可以利用 try-with-resources 特性（使用有限制）
// 资源名可使用任意有业务语义的字符串，比如方法名、接口名或其它可唯一标识的字符串。
try (Entry entry = SphU.entry("resourceName")) {
  // 被保护的业务逻辑
  // do something here...
} catch (BlockException ex) {
  // 资源访问阻止，被限流或被降级
  // 在此处进行相应的处理操作
}
```

**特别地**，若 entry 的时候传入了热点参数，那么 exit 的时候也一定要带上对应的参数（`exit(count, args)`），否则可能会有统计错误。这个时候不能使用 try-with-resources 的方式。另外通过 `Tracer.trace(ex)` 来统计异常信息时，由于 try-with-resources 语法中 catch 调用顺序的问题，会导致无法正确统计异常数，因此统计异常信息时也不能在 try-with-resources 的 catch 块中调用 `Tracer.trace(ex)`。

手动 exit 示例：

```
Entry entry = null;
// 务必保证 finally 会被执行
try {
  // 资源名可使用任意有业务语义的字符串，注意数目不能太多（超过 1K），超出几千请作为参数传入而不要直接作为资源名
  // EntryType 代表流量类型（inbound/outbound），其中系统规则只对 IN 类型的埋点生效
  entry = SphU.entry("自定义资源名");
  // 被保护的业务逻辑
  // do something...
} catch (BlockException ex) {
  // 资源访问阻止，被限流或被降级
  // 进行相应的处理操作
} catch (Exception ex) {
  // 若需要配置降级规则，需要通过这种方式记录业务异常
  Tracer.traceEntry(ex, entry);
} finally {
  // 务必保证 exit，务必保证每个 entry 与 exit 配对
  if (entry != null) {
    entry.exit();
  }
}
```

热点参数埋点示例：

```
Entry entry = null;
try {
    // 若需要配置例外项，则传入的参数只支持基本类型。
    // EntryType 代表流量类型，其中系统规则只对 IN 类型的埋点生效
    // count 大多数情况都填 1，代表统计为一次调用。
    entry = SphU.entry(resourceName, EntryType.IN, 1, paramA, paramB);
    // Your logic here.
} catch (BlockException ex) {
    // Handle request rejection.
} finally {
    // 注意：exit 的时候也一定要带上对应的参数，否则可能会有统计错误。
    if (entry != null) {
        entry.exit(1, paramA, paramB);
    }
}
```

`SphU.entry()` 的参数描述：

| 参数名    | 类型        | 解释                                                         | 默认值          |
| --------- | ----------- | ------------------------------------------------------------ | --------------- |
| entryType | `EntryType` | 资源调用的流量类型，是入口流量（`EntryType.IN`）还是出口流量（`EntryType.OUT`），注意系统规则只对 IN 生效 | `EntryType.OUT` |
| count     | `int`       | 本次资源调用请求的 token 数目                                | 1               |
| args      | `Object[]`  | 传入的参数，用于热点参数限流                                 | 无              |

**注意**：`SphU.entry(xxx)` 需要与 `entry.exit()` 方法成对出现，匹配调用，否则会导致调用链记录异常，抛出 `ErrorEntryFreeException` 异常。常见的错误：

- 自定义埋点只调用 `SphU.entry()`，没有调用 `entry.exit()`
- 顺序错误，比如：`entry1 -> entry2 -> exit1 -> exit2`，应该为 `entry1 -> entry2 -> exit2 -> exit1`

#### 方式三：返回布尔值方式定义资源

`SphO` 提供 if-else 风格的 API。用这种方式，当资源发生了限流之后会返回 `false`，这个时候可以根据返回值，进行限流之后的逻辑处理。示例代码如下:

```
  // 资源名可使用任意有业务语义的字符串
  if (SphO.entry("自定义资源名")) {
    // 务必保证finally会被执行
    try {
      /**
      * 被保护的业务逻辑
      */
    } finally {
      SphO.exit();
    }
  } else {
    // 资源访问阻止，被限流或被降级
    // 进行相应的处理操作
  }
```

**注意**：`SphO.entry(xxx)` 需要与 SphO.exit()`方法成对出现，匹配调用，位置正确，否则会导致调用链记录异常，抛出`ErrorEntryFreeException` 异常。

#### 方式四：注解方式定义资源

Sentinel 支持通过 `@SentinelResource` 注解定义资源并配置 `blockHandler` 和 `fallback` 函数来进行限流之后的处理。示例：

```
// 原本的业务方法.
@SentinelResource(blockHandler = "blockHandlerForGetUser")
public User getUserById(String id) {
    throw new RuntimeException("getUserById command failed");
}

// blockHandler 函数，原方法调用被限流/降级/系统保护的时候调用
public User blockHandlerForGetUser(String id, BlockException ex) {
    return new User("admin");
}
```

注意 `blockHandler` 函数会在原方法被限流/降级/系统保护的时候调用，而 `fallback` 函数会针对所有类型的异常。请注意 `blockHandler` 和 `fallback` 函数的形式要求，更多指引可以参见 [Sentinel 注解支持文档](https://github.com/alibaba/Sentinel/wiki/注解支持)。

注：1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。

若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。

#### 方式五：异步调用支持

Sentinel 支持异步调用链路的统计。在异步调用中，需要通过 `SphU.asyncEntry(xxx)` 方法定义资源，并通常需要在异步的回调函数中调用 `exit` 方法。以下是一个简单的示例：

```
try {
    AsyncEntry entry = SphU.asyncEntry(resourceName);

    // 异步调用.
    doAsync(userId, result -> {
        try {
            // 在此处处理异步调用的结果.
        } finally {
            // 在回调结束后 exit.
            entry.exit();
        }
    });
} catch (BlockException ex) {
    // Request blocked.
    // Handle the exception (e.g. retry or fallback).
}
```

`SphU.asyncEntry(xxx)` 不会影响当前（调用线程）的 Context，因此以下两个 entry 在调用链上是平级关系（处于同一层），而不是嵌套关系：

```
// 调用链类似于：
// -parent
// ---asyncResource
// ---syncResource
asyncEntry = SphU.asyncEntry(asyncResource);
entry = SphU.entry(normalResource);
```

若在异步回调中需要嵌套其它的资源调用（无论是 `entry` 还是 `asyncEntry`），只需要借助 Sentinel 提供的上下文切换功能，在对应的地方通过 `ContextUtil.runOnContext(context, f)` 进行 Context 变换，将对应资源调用处的 Context 切换为生成的异步 Context，即可维持正确的调用链路关系。示例如下：

```
public void handleResult(String result) {
    Entry entry = null;
    try {
        entry = SphU.entry("handleResultForAsync");
        // Handle your result here.
    } catch (BlockException ex) {
        // Blocked for the result handler.
    } finally {
        if (entry != null) {
            entry.exit();
        }
    }
}

public void someAsync() {
    try {
        AsyncEntry entry = SphU.asyncEntry(resourceName);

        // Asynchronous invocation.
        doAsync(userId, result -> {
            // 在异步回调中进行上下文变换，通过 AsyncEntry 的 getAsyncContext 方法获取异步 Context
            ContextUtil.runOnContext(entry.getAsyncContext(), () -> {
                try {
                    // 此处嵌套正常的资源调用.
                    handleResult(result);
                } finally {
                    entry.exit();
                }
            });
        });
    } catch (BlockException ex) {
        // Request blocked.
        // Handle the exception (e.g. retry or fallback).
    }
}
```

此时的调用链就类似于：

```
-parent
---asyncInvocation
-----handleResultForAsync
```

更详细的示例可以参考 Demo 中的 [AsyncEntryDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/AsyncEntryDemo.java)，里面包含了普通资源与异步资源之间的各种嵌套示例。

### 规则的种类

Sentinel 的所有规则都可以在内存态中动态地查询及修改，修改之后立即生效。同时 Sentinel 也提供相关 API，供您来定制自己的规则策略。

Sentinel 支持以下几种规则：**流量控制规则**、**熔断降级规则**、**系统保护规则**、**来源访问控制规则** 和 **热点参数规则**。

#### 流量控制规则 (FlowRule)

##### 流量规则的定义

重要属性：

| Field           | 说明                                                         | 默认值                        |
| --------------- | ------------------------------------------------------------ | ----------------------------- |
| resource        | 资源名，资源名是限流规则的作用对象                           |                               |
| count           | 限流阈值                                                     |                               |
| grade           | 限流阈值类型，QPS 模式（1）或并发线程数模式（0）             | QPS 模式                      |
| limitApp        | 流控针对的调用来源                                           | `default`，代表不区分调用来源 |
| strategy        | 调用关系限流策略：直接、链路、关联                           | 根据资源本身（直接）          |
| controlBehavior | 流控效果（直接拒绝/WarmUp/匀速+排队等待），不支持按调用关系限流 | 直接拒绝                      |
| clusterMode     | 是否集群限流                                                 | 否                            |

同一个资源可以同时有多个限流规则，检查规则时会依次检查。

##### 通过代码定义流量控制规则

理解上面规则的定义之后，我们可以通过调用 `FlowRuleManager.loadRules()` 方法来用硬编码的方式定义流量控制规则，比如：

```
private void initFlowQpsRule() {
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule(resourceName);
    // set limit qps to 20
    rule.setCount(20);
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    rule.setLimitApp("default");
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

#### 熔断降级规则 (DegradeRule)

熔断降级规则包含下面几个重要的属性：

| Field              | 说明                                                         | 默认值     |
| ------------------ | ------------------------------------------------------------ | ---------- |
| resource           | 资源名，即规则的作用对象                                     |            |
| grade              | 熔断策略，支持慢调用比例/异常比例/异常数策略                 | 慢调用比例 |
| count              | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值 |            |
| timeWindow         | 熔断时长，单位为 s                                           |            |
| minRequestAmount   | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5          |
| statIntervalMs     | 统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入）   | 1000 ms    |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）           |            |

同一个资源可以同时有多个降级规则。

理解上面规则的定义之后，我们可以通过调用 `DegradeRuleManager.loadRules()` 方法来用硬编码的方式定义流量控制规则。

```
private void initDegradeRule() {
    List<DegradeRule> rules = new ArrayList<>();
    DegradeRule rule = new DegradeRule();
    rule.setResource(KEY);
    // set threshold RT, 10 ms
    rule.setCount(10);
    rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
    rule.setTimeWindow(10);
    rules.add(rule);
    DegradeRuleManager.loadRules(rules);
}
```

#### 系统保护规则 (SystemRule)

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统规则包含下面几个重要的属性：

| Field             | 说明                                   | 默认值      |
| ----------------- | -------------------------------------- | ----------- |
| highestSystemLoad | `load1` 触发值，用于触发自适应控制阶段 | -1 (不生效) |
| avgRt             | 所有入口流量的平均响应时间             | -1 (不生效) |
| maxThread         | 入口流量的最大并发数                   | -1 (不生效) |
| qps               | 所有入口资源的 QPS                     | -1 (不生效) |
| highestCpuUsage   | 当前系统的 CPU 使用率（0.0-1.0）       | -1 (不生效) |

理解上面规则的定义之后，我们可以通过调用 `SystemRuleManager.loadRules()` 方法来用硬编码的方式定义流量控制规则。

```
private void initSystemRule() {
    List<SystemRule> rules = new ArrayList<>();
    SystemRule rule = new SystemRule();
    rule.setHighestSystemLoad(10);
    rules.add(rule);
    SystemRuleManager.loadRules(rules);
}
```

注意系统规则只针对入口资源（EntryType=IN）生效。

#### 访问控制规则 (AuthorityRule)

很多时候，我们需要根据调用方来限制资源是否通过，这时候可以使用 Sentinel 的访问控制（黑白名单）的功能。黑白名单根据资源的请求来源（`origin`）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过。

授权规则，即黑白名单规则（`AuthorityRule`）非常简单，主要有以下配置项：

- `resource`：资源名，即规则的作用对象
- `limitApp`：对应的黑名单/白名单，不同 origin 用 `,` 分隔，如 `appA,appB`
- `strategy`：限制模式，`AUTHORITY_WHITE` 为白名单模式，`AUTHORITY_BLACK` 为黑名单模式，默认为白名单模式

#### 热点规则 (ParamFlowRule)



### 查询更改规则

引入了 transport 模块后，可以通过以下的 HTTP API 来获取所有已加载的规则：

```
http://localhost:8719/getRules?type=<XXXX>
```

其中，`type=flow` 以 JSON 格式返回现有的限流规则，degrade 返回现有生效的降级规则列表，system 则返回系统保护规则。

获取所有热点规则：

```
http://localhost:8719/getParamRules
```

### 定制自己的持久化规则

上面的规则配置，都是存在内存中的。即如果应用重启，这个规则就会失效。因此我们提供了开放的接口，您可以通过实现 [`DataSource`](https://github.com/alibaba/Sentinel/blob/master/sentinel-extension/sentinel-datasource-extension/src/main/java/com/alibaba/csp/sentinel/datasource/AbstractDataSource.java) 接口的方式，来自定义规则的存储数据源。通常我们的建议有：

- 整合动态配置系统，如 ZooKeeper、[Nacos](https://github.com/alibaba/Nacos)、Apollo 等，动态地实时刷新配置规则
- 结合 RDBMS、NoSQL、VCS 等来实现该规则
- 配合 Sentinel Dashboard 使用

更多详情请参考 [动态规则配置](https://github.com/alibaba/Sentinel/wiki/动态规则扩展)。

### 规则生效的效果

#### 判断限流降级异常

在 Sentinel 中所有流控降级相关的异常都是异常类 `BlockException` 的子类：

- 流控异常：`FlowException`
- 熔断降级异常：`DegradeException`
- 系统保护异常：`SystemBlockException`
- 热点参数限流异常：`ParamFlowException`

我们可以通过以下函数判断是否为 Sentinel 的流控降级异常：

```
BlockException.isBlockException(Throwable t);
```

除了在业务代码逻辑上看到规则生效，我们也可以通过下面简单的方法，来校验规则生效的效果：

- **暴露的 HTTP 接口**：通过运行下面命令 `curl http://localhost:8719/cnode?id=<资源名称>`，观察返回的数据。如果规则生效，在返回的数据栏中的 `block` 以及 `block(m)` 中会有显示
- **日志**：Sentinel 提供秒级的资源运行日志以及限流日志，详情可以参考: [日志](https://github.com/alibaba/Sentinel/wiki/日志)

### block 事件

Sentinel 提供以下扩展接口，可以通过 `StatisticSlotCallbackRegistry` 向 `StatisticSlot` 注册回调函数：

- `ProcessorSlotEntryCallback`: callback when resource entry passed (`onPass`) or blocked (`onBlocked`)
- `ProcessorSlotExitCallback`: callback when resource entry successfully completed (`onExit`)

可以利用这些回调接口来实现报警等功能，实时的监控信息可以从 `ClusterNode` 中实时获取。

### 其它 API

#### 业务异常统计 Tracer

业务异常记录类 `Tracer` 用于记录业务异常。相关方法：

- `traceEntry(Throwable, Entry)`：向传入 entry 对应的资源记录业务异常（非 `BlockException` 异常），异常数目为传入的 `count`。

如果用户通过 `SphU` 或 `SphO` 手动定义资源，则 Sentinel 不能感知上层业务的异常，需要手动调用 `Tracer.trace(ex)` 来记录业务异常，否则对应的异常不会统计到 Sentinel 异常计数中。注意不要在 try-with-resources 形式的 `SphU.entry(xxx)` 中使用，否则会统计不上。

从 1.3.1 版本开始，注解方式定义资源支持自动统计业务异常，无需手动调用 `Tracer.trace(ex)` 来记录业务异常。Sentinel 1.3.1 以前的版本需要手动记录。

#### 上下文工具类 ContextUtil

相关静态方法：

**标识进入调用链入口（上下文）**：

以下静态方法用于标识调用链路入口，用于区分不同的调用链路：

- `public static Context enter(String contextName)`
- `public static Context enter(String contextName, String origin)`

其中 `contextName` 代表调用链路入口名称（上下文名称），`origin` 代表调用来源名称。默认调用来源为空。返回值类型为 `Context`，即生成的调用链路上下文对象。

流控规则中若选择“流控方式”为“链路”方式，则入口资源名即为上面的 `contextName`。

**注意**：

- `ContextUtil.enter(xxx)` 方法仅在调用链路入口处生效，即仅在当前线程的初次调用生效，后面再调用不会覆盖当前线程的调用链路，直到 exit。`Context` 存于 ThreadLocal 中，因此切换线程时可能会丢掉，如果需要跨线程使用可以结合 `runOnContext` 方法使用。
- origin 数量不要太多，否则内存占用会比较大。

**退出调用链（清空上下文）**：

- `public static void exit()`：该方法用于退出调用链，清理当前线程的上下文。

**获取当前线程的调用链上下文**：

- `public static Context getContext()`：获取当前线程的调用链路上下文对象。

**在某个调用链上下文中执行代码**：

- `public static void runOnContext(Context context, Runnable f)`：常用于异步调用链路中 context 的变换。

## 流量控制

**流量控制**（flow control），其原理是监控应用流量的 QPS 或并发线程数等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

通过流控规则来指定允许该资源通过的请求次数，例如下面的代码定义了资源 HelloWorld 每秒最多只能通过 20 个请求。 参考的规则定义如下：

```
private static void initFlowRules(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：

- `resource`：资源名，即限流规则的作用对象
- `count`: 限流阈值
- `grade`: 限流阈值类型（QPS 或并发线程数）
- `limitApp`: 流控针对的调用来源，若为 `default` 则不区分调用来源
- `strategy`: 调用关系限流策略
- `controlBehavior`: 流量控制效果（直接拒绝、Warm Up、匀速排队）

限流参数

| Field           | 说明                                                         | 默认值                        |
| :-------------- | :----------------------------------------------------------- | :---------------------------- |
| resource        | 资源名，资源名是限流规则的作用对象                           |                               |
| count           | 限流阈值                                                     |                               |
| grade           | 限流阈值类型，QPS 或线程数模式                               | QPS 模式                      |
| limitApp        | 流控针对的调用来源                                           | `default`，代表不区分调用来源 |
| strategy        | 判断的根据是资源自身，还是根据其它关联资源 (`refResource`)，还是根据链路入口 | 根据资源本身                  |
| controlBehavior | 流控效果（直接拒绝 / 排队等待 / 慢启动模式）                 | 直接拒绝                      |

### 基于QPS/并发数的流量控制

流量控制主要有两种统计类型，一种是统计并发线程数，另外一种则是统计 QPS。类型由 `FlowRule` 的 `grade` 字段来定义。其中，0 代表根据并发数量来限流，1 代表根据 QPS 来进行流量控制。其中线程数、QPS 值，都是由 `StatisticSlot` 实时统计获取的。

```
public static final int FLOW_GRADE_THREAD = 0;
public static final int FLOW_GRADE_QPS = 1;
```

#### 并发线程数控制

并发数控制用于保护业务线程池不被慢调用耗尽。Sentinel 并发控制不负责创建和管理线程池，而是简单统计当前请求上下文的线程数目（正在执行的调用数目），如果超出阈值，新的请求会被立即拒绝，效果类似于信号量隔离。**并发数控制通常在调用端进行配置。**

#### QPS流量控制

当 QPS 超过某个阈值的时候，则采取措施进行流量控制。流量控制的效果包括以下几种：**直接拒绝**、**Warm Up**、**匀速排队**。对应 `FlowRule` 中的 `controlBehavior` 字段。

### 流量控制strategy

```
    public static final int STRATEGY_DIRECT = 0;
    public static final int STRATEGY_RELATE = 1;
    public static final int STRATEGY_CHAIN = 2;
```

#### 直接拒绝

**直接拒绝**（`RuleConstant.CONTROL_BEHAVIOR_DEFAULT`）方式是默认的流量控制方式，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出`FlowException`。这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位时。

#### Warm Up

Warm Up（`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。

#### 匀速排队

匀速排队（`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。

### 基于调用关系的流量控制

调用关系包括调用方、被调用方；一个方法又可能会调用其它方法，形成一个调用链路的层次关系。Sentinel 通过 `NodeSelectorSlot` 建立不同资源间的调用的关系，并且通过 `ClusterBuilderSlot` 记录每个资源的实时统计信息。

#### 根据调用方限流

`ContextUtil.enter(resourceName, origin)` 方法中的 `origin` 参数标明了调用方身份。这些信息会在 `ClusterBuilderSlot` 中被统计。

#### 根据调用链路入口限流：链路限流

`odeSelectorSlot` 中记录了资源之间的调用链路，这些资源通过调用关系，相互之间构成一棵调用树。这棵树的根节点是一个名字为 `machine-root` 的虚拟节点，调用链的入口都是这个虚节点的子节点。

### 具有关系的资源流量控制：关联流量控制

当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联。比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。如果放任读写操作争抢资源，则争抢本身带来的开销会降低整体的吞吐量。可使用关联限流来避免具有关联关系的资源之间过度的争抢，举例来说，`read_db` 和 `write_db` 这两个资源分别代表数据库读写，我们可以给 `read_db` 设置限流规则来达到写优先的目的：设置 `strategy` 为 `RuleConstant.STRATEGY_RELATE` 同时设置 `refResource` 为 `write_db`。这样当写库操作过于频繁时，读数据的请求会被限流。

## 集群流控

为什么要使用集群流控呢？假设我们希望给某个用户限制调用某个 API 的总 QPS 为 50，但机器数可能很多（比如有 100 台）。这时候我们很自然地就想到，找一个 server 来专门来统计总的调用量，其它的实例都与这台 server 通信来判断是否可以调用。这就是最基础的集群流控的方式。

另外集群流控还可以解决流量不均匀导致总体限流效果不佳的问题。假设集群中有 10 台机器，我们给每台机器设置单机限流阈值为 10 QPS，理想情况下整个集群的限流阈值就为 100 QPS。不过实际情况下流量到每台机器可能会不均匀，会导致总量没有到的情况下某些机器就开始限流。因此仅靠单机维度去限制的话会无法精确地限制总体流量。而集群流控可以精确地控制整个集群的调用总量，结合单机限流兜底，可以更好地发挥流量控制的效果。

集群流控中共有两种身份：

- Token Client：集群流控客户端，用于向所属 Token Server 通信请求 token。集群限流服务端会返回给客户端结果，决定是否限流。
- Token Server：即集群流控服务端，处理来自 Token Client 的请求，根据配置的集群规则判断是否应该发放 token（是否允许通过）。

### 模块结构

Sentinel 1.4.0 开始引入了集群流控模块，主要包含以下几部分：

- `sentinel-cluster-common-default`: 公共模块，包含公共接口和实体
- `sentinel-cluster-client-default`: 默认集群流控 client 模块，使用 Netty 进行通信，提供接口方便序列化协议扩展
- `sentinel-cluster-server-default`: 默认集群流控 server 模块，使用 Netty 进行通信，提供接口方便序列化协议扩展；同时提供扩展接口对接规则判断的具体实现（`TokenService`），默认实现是复用 `sentinel-core` 的相关逻辑

### 集群流控规则

#### 规则

`FlowRule` 添加了两个字段用于集群限流相关配置：

```
private boolean clusterMode; // 标识是否为集群限流配置
private ClusterFlowConfig clusterConfig; // 集群限流相关配置项
```

其中 用一个专门的 `ClusterFlowConfig` 代表集群限流相关配置项，以与现有规则配置项分开：

```
// （必需）全局唯一的规则 ID，由集群限流管控端分配.
private Long flowId;

// 阈值模式，默认（0）为单机均摊，1 为全局阈值.
private int thresholdType = ClusterRuleConstant.FLOW_THRESHOLD_AVG_LOCAL;

private int strategy = ClusterRuleConstant.FLOW_CLUSTER_STRATEGY_NORMAL;

// 在 client 连接失败或通信失败时，是否退化到本地的限流模式
private boolean fallbackToLocalWhenFail = true;
```

- `flowId` 代表全局唯一的规则 ID，Sentinel 集群限流服务端通过此 ID 来区分各个规则，因此**务必保持全局唯一**。一般 flowId 由统一的管控端进行分配，或写入至 DB 时生成。
- `thresholdType` 代表集群限流阈值模式。其中**单机均摊模式**下配置的阈值等同于单机能够承受的限额，token server 会根据客户端对应的 namespace（默认为 `project.name` 定义的应用名）下的连接数来计算总的阈值（比如独立模式下有 3 个 client 连接到了 token server，然后配的单机均摊阈值为 10，则计算出的集群总量就为 30）；而全局模式下配置的阈值等同于**整个集群的总阈值**。

`ParamFlowRule` 热点参数限流相关的集群配置与 `FlowRule` 相似。

#### 集群规则配置方式

在集群流控的场景下，我们推荐使用动态规则源来动态地管理规则。

对于客户端，我们可以按照[原有的方式](https://github.com/alibaba/Sentinel/wiki/动态规则扩展)来向 `FlowRuleManager` 和 `ParamFlowRuleManager` 注册动态规则源，例如：

```
ReadableDataSource<String, List<FlowRule>> flowRuleDataSource = new NacosDataSource<>(remoteAddress, groupId, dataId, parser);
FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
```

对于集群流控 token server，由于集群限流服务端有作用域（namespace）的概念，因此我们需要注册一个自动根据 namespace 生成动态规则源的 PropertySupplier:

```
// Supplier 类型：接受 namespace，返回生成的动态规则源，类型为 SentinelProperty<List<FlowRule>>
// ClusterFlowRuleManager 针对集群限流规则，ClusterParamFlowRuleManager 针对集群热点规则，配置方式类似
ClusterFlowRuleManager.setPropertySupplier(namespace -> {
    return new SomeDataSource(namespace).getProperty();
});
```

然后每当集群限流服务端 namespace set 产生变更时，Sentinel 会自动针对新加入的 namespace 生成动态规则源并进行自动监听，并删除旧的不需要的规则源。

## 网关限流

Sentinel 支持对 Spring Cloud Gateway、Zuul 等主流的 API Gateway 进行限流。

![sentinel网关限流](png\sentinel网关限流.png)



Sentinel 1.6.0 引入了 Sentinel API Gateway Adapter Common 模块，此模块中包含网关限流的规则和自定义 API 的实体和管理逻辑：

- `GatewayFlowRule`：网关限流规则，针对 API Gateway 的场景定制的限流规则，可以针对不同 route 或自定义的 API 分组进行限流，支持针对请求中的参数、Header、来源 IP 等进行定制化的限流。
- `ApiDefinition`：用户自定义的 API 定义分组，可以看做是一些 URL 匹配的组合。比如我们可以定义一个 API 叫 `my_api`，请求 path 模式为 `/foo/**` 和 `/baz/**` 的都归到 `my_api` 这个 API 分组下面。限流的时候可以针对这个自定义的 API 分组维度进行限流。

其中网关限流规则 `GatewayFlowRule` 的字段解释如下：

- `resource`：资源名称，可以是网关中的 route 名称或者用户自定义的 API 分组名称。

- `resourceMode`：规则是针对 API Gateway 的 route（`RESOURCE_MODE_ROUTE_ID`）还是用户在 Sentinel 中定义的 API 分组（`RESOURCE_MODE_CUSTOM_API_NAME`），默认是 route。

- `grade`：限流指标维度，同限流规则的 `grade` 字段。

- `count`：限流阈值

- `intervalSec`：统计时间窗口，单位是秒，默认是 1 秒。

- `controlBehavior`：流量整形的控制效果，同限流规则的 `controlBehavior` 字段，目前支持快速失败和匀速排队两种模式，默认是快速失败。

- `burst`：应对突发请求时额外允许的请求数目。

- `maxQueueingTimeoutMs`：匀速排队模式下的最长排队时间，单位是毫秒，仅在匀速排队模式下生效。

- ```
  paramItem
  ```

  ：参数限流配置。若不提供，则代表不针对参数进行限流，该网关规则将会被转换成普通流控规则；否则会转换成热点规则。其中的字段：

  - `parseStrategy`：从请求中提取参数的策略，目前支持提取来源 IP（`PARAM_PARSE_STRATEGY_CLIENT_IP`）、Host（`PARAM_PARSE_STRATEGY_HOST`）、任意 Header（`PARAM_PARSE_STRATEGY_HEADER`）和任意 URL 参数（`PARAM_PARSE_STRATEGY_URL_PARAM`）四种模式。
  - `fieldName`：若提取策略选择 Header 模式或 URL 参数模式，则需要指定对应的 header 名称或 URL 参数名称。
  - `pattern`：参数值的匹配模式，只有匹配该模式的请求属性值会纳入统计和流控；若为空则统计该请求属性的所有值。（1.6.2 版本开始支持）
  - `matchStrategy`：参数值的匹配策略，目前支持精确匹配（`PARAM_MATCH_STRATEGY_EXACT`）、子串匹配（`PARAM_MATCH_STRATEGY_CONTAINS`）和正则匹配（`PARAM_MATCH_STRATEGY_REGEX`）。（1.6.2 版本开始支持）

用户可以通过 `GatewayRuleManager.loadRules(rules)` 手动加载网关规则，或通过 `GatewayRuleManager.register2Property(property)` 注册动态规则源动态推送（推荐方式）。

### Spring Cloud Gateway

从 1.6.0 版本开始，Sentinel 提供了 Spring Cloud Gateway 的适配模块，可以提供两种资源维度的限流：

- route 维度：即在 Spring 配置文件中配置的路由条目，资源名为对应的 routeId
- 自定义 API 维度：用户可以利用 Sentinel 提供的 API 来自定义一些 API 分组

使用时需引入以下模块（以 Maven 为例）：

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
    <version>x.y.z</version>
</dependency>
```

使用时只需注入对应的 `SentinelGatewayFilter` 实例以及 `SentinelGatewayBlockExceptionHandler` 实例即可（若使用了 Spring Cloud Alibaba Sentinel，则只需按照[文档](https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel#spring-cloud-gateway-支持)进行配置即可，无需自己加 Configuration）。比如：

```
@Configuration
public class GatewayConfiguration {

    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;

    public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        // Register the block exception handler for Spring Cloud Gateway.
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
}
```

Demo 示例：[sentinel-demo-spring-cloud-gateway](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-spring-cloud-gateway)

比如我们在 Spring Cloud Gateway 中配置了以下路由：

```
server:
  port: 8090
spring:
  application:
    name: spring-cloud-gateway
  cloud:
    gateway:
      enabled: true
      discovery:
        locator:
          lower-case-service-id: true
      routes:
        # Add your routes here.
        - id: product_route
          uri: lb://product
          predicates:
            - Path=/product/**
        - id: httpbin_route
          uri: https://httpbin.org
          predicates:
            - Path=/httpbin/**
          filters:
            - RewritePath=/httpbin/(?<segment>.*), /$\{segment}
```

同时自定义了一些 API 分组：

```
private void initCustomizedApis() {
    Set<ApiDefinition> definitions = new HashSet<>();
    ApiDefinition api1 = new ApiDefinition("some_customized_api")
        .setPredicateItems(new HashSet<ApiPredicateItem>() {{
            add(new ApiPathPredicateItem().setPattern("/product/baz"));
            add(new ApiPathPredicateItem().setPattern("/product/foo/**")
                .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
        }});
    ApiDefinition api2 = new ApiDefinition("another_customized_api")
        .setPredicateItems(new HashSet<ApiPredicateItem>() {{
            add(new ApiPathPredicateItem().setPattern("/ahas"));
        }});
    definitions.add(api1);
    definitions.add(api2);
    GatewayApiDefinitionManager.loadApiDefinitions(definitions);
}
```

那么这里面的 route ID（如 `product_route`）和 API name（如 `some_customized_api`）都会被标识为 Sentinel 的资源。比如访问网关的 URL 为 `http://localhost:8090/product/foo/22` 的时候，对应的统计会加到 `product_route` 和 `some_customized_api` 这两个资源上面，而 `http://localhost:8090/httpbin/json` 只会对应到 `httpbin_route` 资源上面。

> **注意**：有的时候 Spring Cloud Gateway 会自己在 route 名称前面拼一个前缀，如 `ReactiveCompositeDiscoveryClient_xxx` 这种。请观察簇点链路页面实际的资源名。

您可以在 `GatewayCallbackManager` 注册回调进行定制：

- `setBlockHandler`：注册函数用于实现自定义的逻辑处理被限流的请求，对应接口为 `BlockRequestHandler`。默认实现为 `DefaultBlockRequestHandler`，当被限流时会返回类似于下面的错误信息：`Blocked by Sentinel: FlowException`。

**注意**：

- Sentinel 网关流控默认的粒度是 route 维度以及自定义 API 分组维度，默认**不支持 URL 粒度**。若通过 Spring Cloud Alibaba 接入，请将 `spring.cloud.sentinel.filter.enabled` 配置项置为 false（若在网关流控控制台上看到了 URL 资源，就是此配置项没有置为 false）。
- 若使用 Spring Cloud Alibaba Sentinel 数据源模块，需要注意网关流控规则数据源类型是 `gw-flow`，若将网关流控规则数据源指定为 flow 则不生效。

### 网关流控实现原理

当通过 `GatewayRuleManager` 加载网关流控规则（`GatewayFlowRule`）时，无论是否针对请求属性进行限流，Sentinel 底层都会将网关流控规则转化为热点参数规则（`ParamFlowRule`），存储在 `GatewayRuleManager` 中，与正常的热点参数规则相隔离。转换时 Sentinel 会根据请求属性配置，为网关流控规则设置参数索引（`idx`），并同步到生成的热点参数规则中。

外部请求进入 API Gateway 时会经过 Sentinel 实现的 filter，其中会依次进行 **路由/API 分组匹配**、**请求属性解析**和**参数组装**。Sentinel 会根据配置的网关流控规则来解析请求属性，并依照参数索引顺序组装参数数组，最终传入 `SphU.entry(res, args)` 中。Sentinel API Gateway Adapter Common 模块向 Slot Chain 中添加了一个 `GatewayFlowSlot`，专门用来做网关规则的检查。`GatewayFlowSlot` 会从 `GatewayRuleManager` 中提取生成的热点参数规则，根据传入的参数依次进行规则检查。若某条规则不针对请求属性，则会在参数最后一个位置置入预设的常量，达到普通流控的效果。

## 热点参数限流

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。

### 基本使用

要使用热点参数限流功能，需要引入以下依赖：

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-parameter-flow-control</artifactId>
    <version>x.y.z</version>
</dependency>
```

然后为对应的资源配置热点参数限流规则，并在 `entry` 的时候传入相应的参数，即可使热点参数限流生效。

> 注：若自行扩展并注册了自己实现的 `SlotChainBuilder`，并希望使用热点参数限流功能，则可以在 chain 里面合适的地方插入 `ParamFlowSlot`。

那么如何传入对应的参数以便 Sentinel 统计呢？我们可以通过 `SphU` 类里面几个 `entry` 重载方法来传入：

```
public static Entry entry(String name, EntryType type, int count, Object... args) throws BlockException

public static Entry entry(Method method, EntryType type, int count, Object... args) throws BlockException
```

其中最后的一串 `args` 就是要传入的参数，有多个就按照次序依次传入。比如要传入两个参数 `paramA` 和 `paramB`，则可以：

```
// paramA in index 0, paramB in index 1.
// 若需要配置例外项或者使用集群维度流控，则传入的参数只支持基本类型。
SphU.entry(resourceName, EntryType.IN, 1, paramA, paramB);
```

**注意**：若 entry 的时候传入了热点参数，那么 exit 的时候也一定要带上对应的参数（`exit(count, args)`），否则可能会有统计错误。正确的示例：

```
Entry entry = null;
try {
    entry = SphU.entry(resourceName, EntryType.IN, 1, paramA, paramB);
    // Your logic here.
} catch (BlockException ex) {
    // Handle request rejection.
} finally {
    if (entry != null) {
        entry.exit(1, paramA, paramB);
    }
}
```

对于 `@SentinelResource` 注解方式定义的资源，若注解作用的方法上有参数，Sentinel 会将它们作为参数传入 `SphU.entry(res, args)`。比如以下的方法里面 `uid` 和 `type` 会分别作为第一个和第二个参数传入 Sentinel API，从而可以用于热点规则判断：

```
@SentinelResource("myMethod")
public Result doSomething(String uid, int type) {
  // some logic here...
}
```

> **注意**：目前 Sentinel 自带的 adapter 仅 Dubbo 方法埋点带了热点参数，其它适配模块（如 Web）默认不支持热点规则，可通过自定义埋点方式指定新的资源名并传入希望的参数。注意自定义埋点的资源名不要和适配模块生成的资源名重复，否则会导致重复统计。

### 热点参数规则

热点参数规则（`ParamFlowRule`）类似于流量控制规则（`FlowRule`）：

| 属性              | 说明                                                         | 默认值   |
| ----------------- | ------------------------------------------------------------ | -------- |
| resource          | 资源名，必填                                                 |          |
| count             | 限流阈值，必填                                               |          |
| grade             | 限流模式                                                     | QPS 模式 |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持             | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持   | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持 | 0ms      |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | `false`  |
| clusterConfig     | 集群流控相关配置                                             |          |

我们可以通过 `ParamFlowRuleManager` 的 `loadRules` 方法更新热点参数规则，下面是一个示例：

```
ParamFlowRule rule = new ParamFlowRule(resourceName)
    .setParamIdx(0)
    .setCount(5);
// 针对 int 类型的参数 PARAM_B，单独设置限流 QPS 阈值为 10，而不是全局的阈值 5.
ParamFlowItem item = new ParamFlowItem().setObject(String.valueOf(PARAM_B))
    .setClassType(int.class.getName())
    .setCount(10);
rule.setParamFlowItemList(Collections.singletonList(item));

ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
```

## 黑白名单控制

很多时候，我们需要根据调用来源来判断该次请求是否允许放行，这时候可以使用 Sentinel 的来源访问控制（黑白名单控制）的功能。来源访问控制根据资源的请求来源（`origin`）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过。

> 调用方信息通过 `ContextUtil.enter(resourceName, origin)` 方法中的 `origin` 参数传入。

### 规则配置

来源访问控制规则（`AuthorityRule`）非常简单，主要有以下配置项：

- `resource`：资源名，即限流规则的作用对象。
- `limitApp`：对应的黑名单/白名单，不同 origin 用 `,` 分隔，如 `appA,appB`。
- `strategy`：限制模式，`AUTHORITY_WHITE` 为白名单模式，`AUTHORITY_BLACK` 为黑名单模式，默认为白名单模式。

比如我们希望控制对资源 `test` 的访问设置白名单，只有来源为 `appA` 和 `appB` 的请求才可通过，则可以配置如下白名单规则：

```
AuthorityRule rule = new AuthorityRule();
rule.setResource("test");
rule.setStrategy(RuleConstant.AUTHORITY_WHITE);
rule.setLimitApp("appA,appB");
AuthorityRuleManager.loadRules(Collections.singletonList(rule));
```

## 使用注解限流

Sentinel可以手动的通过Sentinel提供的SphU类来保护资源。这种做法不好的地方在于每个需要限制的地方都得写代码，从 0.1.1 版本开始，Sentinel 提供了 @SentinelResource 注解的方式，非常方便。

要使用注解来保护资源需要引入下面的Maven依赖：

```xml
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-annotation-aspectj</artifactId>
	<version>1.4.1</version>
</dependency>
```

引入之后我们需要配置SentinelResourceAspect切面让其生效，因为是通过SentinelResourceAspect切面来实现的，我这边以Spring Boot中使用进行配置示列：

```kotlin
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect;

@Configuration
public class AopConfiguration {

    @Bean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }
    
}
```

然后在需要限制的方法上加SentinelResource注解即可：

```typescript
@SentinelResource(value = "get", blockHandler = "exceptionHandler")
@Override
public String get(String id) {
   return "http://baidu.com";
}

public String exceptionHandler(String id, BlockException e) {
   e.printStackTrace();
   return "错误发生在" + id;
}
```

### @SentinelResource

> 注意：注解方式埋点不支持 private 方法。

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

- value：资源名称，必需项（不能为空）
- entryType：entry 类型，可选项（默认为 EntryType.OUT）
- blockHandler / blockHandlerClass:

blockHandler 对应处理 BlockException的函数名称，可选项。blockHandler 函数访问范围需要是 public，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 blockHandlerClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

- fallback /fallbackClass

  ：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。

- defaultFallback

  （since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。

#### fallback 函数签名和位置要求：

- 返回值类型必须与原函数返回值类型一致；
- 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
- fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。

#### defaultFallback 函数签名要求：

- 返回值类型必须与原函数返回值类型一致；
- 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
- defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。
- exceptionsToIgnore（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

业务方法：

```typescript
@SentinelResource(value = "get2", blockHandler = "handleException", blockHandlerClass = { ExceptionUtil.class })
@Override
public String get2() {
	return "http://cxytiandi.com";
}
```

异常处理类：

```typescript
import com.alibaba.csp.sentinel.slots.block.BlockException;

public final class ExceptionUtil {

    public static String handleException(BlockException ex) {
        System.err.println("错误发生: " + ex.getClass().getCanonicalName());
        return "error";
    }
    
}
```

### 如何测试？

我们可以在Spring Boot的启动类中定义规则，然后快速访问接口，就可以看出效果啦，或者用压力测试工具ab等。

```csharp
@SpringBootApplication
public class App {
	public static void main(String[] args) {
		initFlowRules();
		SpringApplication.run(App.class, args);
	}
	
	private static void initFlowRules() {
		List<FlowRule> rules = new ArrayList<>();
		
		FlowRule rule = new FlowRule();
		rule.setResource("get");
		rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
		rule.setCount(1);
		rules.add(rule);
		
		rule = new FlowRule();
		rule.setResource("get2");
		rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
		rule.setCount(1);
		rules.add(rule);
		
		FlowRuleManager.loadRules(rules);
	}
}
```

## Sentinel 限流降级实战

**Sentinel 限流降级配置**

**pom.xml引入依赖**

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

 **application.properties配置文件增加sentinel配置：**

```
spring.cloud.sentinel.transport.port=8719
spring.cloud.sentinel.transport.dashboard=localhost:8070
spring.cloud.sentinel.log.dir=C:/logs/sentinel-web
```

**Sentinel简单限流（url匹配限流）**

```


@RestController
@RequestMapping("/")
public class SentinelController {

    @Value("${server.port}")
    private String serverPort;
    
    @RequestMapping(value="/sentinel", produces = MediaType.APPLICATION_JSON_VALUE)
    public Result sentinel() {
        return Result.ok(serverPort);
    }
}
```

配置URL作为资源

**@SentinelResource注解使用**

```
@RestController
@RequestMapping("/")
public class SentinelController {

    @Value("${server.port}")
    private String serverPort;
    
    //blockHandler 函数会在原方法被限流/降级/系统保护的时候调用，而 fallback 函数会针对所有类型的异常。
    @RequestMapping(value="/limit", produces = MediaType.APPLICATION_JSON_VALUE)
    @SentinelResource(value="limit", blockHandler = "limitBlockHandler")
    public Result limit() {
        return Result.ok(serverPort);
    }
    
    public Result limitBlockHandler(BlockException be) {
        return Result.fail(serverPort);
    }
}
```

@SentinelResource中，**value**配置的是资源名，不带有/，**blockHandler**配置的是限流回调的方法

**@SentinelResource不生效：**
出现一个@SentinelResource不生效的情况，
原因是：在Sentinel dashboard同时配置了/limit和limit两个限流规划，导致limit不生效，blockHandler不被回调。

blockHandler 函数访问范围需要是 **public**，**返回类型**需要与原方法相匹配，**参数类型**需要和原方法相匹配并且最后加一个**额外的参数**，类型为 **BlockException**。
blockHandler 函数默认需要和原方法在**同一个类**中。若希望使用其他类的函数，则可以指定 **blockHandlerClass** 为对应的类的 Class 对象，注意对应的函数必需为 **static** 函数，否则无法解析。

 **Sentinel fallback异常处理**

```
@RestController
@RequestMapping("/")
public class SentinelController {

    //blockHandler 函数会在原方法被限流/降级/系统保护的时候调用，而 fallback 函数会针对所有类型的异常。
    @RequestMapping(value="/fallback", produces = MediaType.APPLICATION_JSON_VALUE)
    @SentinelResource(value="fallback", blockHandler = "limitBlockHandler", fallback = "failFallback")
    public Result fail(String a) {
        if(StringUtils.isBlank(a)) {
            throw new RuntimeException("参数错误");
        }
        return Result.ok(null, a);
    }
    
    
    //参数要一致，不然fallback匹配不到，后面可以多一个Throwable参数
    public Result failFallback(String a, Throwable t) {
        return Result.failMsg("出现异常：" + serverPort);
    }

}
```

fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。**fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。**

**fallback 函数签名和位置要求：**
**返回值类型**必须与原函数返回值类型一致；
**方法参数列表**需要和原函数一致，或者可以**额外多一个 Throwable 类型**的参数用于接收对应的异常。
fallback 函数**默认需要和原方法在同一个类**中。若希望使用其他类的函数，则可以指定 **fallbackClass** 为对应的类的 Class 对象，注意对应的函数必需为 **static 函数**，否则无法解析。

 

注：1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。

若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。

**同时配置blockHandler和fallback**

若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。
当配置了blockHandler，但blockHandler没有匹配上的时候（如参数不一致），会使用fallback异常处理返回

```

@RestController
@RequestMapping("/")
public class SentinelController {
    
    //blockHandler 函数会在原方法被限流/降级/系统保护的时候调用，而 fallback 函数会针对所有类型的异常。
    @RequestMapping(value="/fallback", produces = MediaType.APPLICATION_JSON_VALUE)
    @SentinelResource(value="fallback", blockHandler = "failBlockHandler", fallback = "failFallback")
    public Result fail(String a) {
        if(StringUtils.isBlank(a)) {
            throw new RuntimeException("参数错误");
        }
        return Result.ok(null, a);
    }
    
    
    public Result failBlockHandler(String a, BlockException be) {
        return Result.fail("failBlockHandler：" + serverPort);
    }
    
    
    //参数要一致，不然fallback匹配不到，后面可以多一个Throwable参数
    //当配置了blockHandler，但blockHandler没有匹配上的时候，会使用fallback异常处理返回
    public Result failFallback(String a, Throwable t) {
        return Result.failMsg("出现异常：" + serverPort);
    }

}
```

Sentinel自定义blockHandlerClass和自定义fallbackClass

```
/**
 * 自定义BlockHandler
 * 方法必需为 static 方法，否则无法解析
 *
 */
public class MyBlockHandler {

    /**
     * 参数一定要和方法对应，否则无法匹配到
     * @param a 
     * @param be BlockException
     * @return
     */
    public static Result blockHandler(String a, BlockException be) {
        return Result.fail("访问频繁，请稍候再试");
    }
    
    
    /**
     * 参数一定要和方法对应，否则无法匹配到
     * @param a 
     * @param t Throwable
     * @return
     */
    public static Result exceptionHandler(String a, Throwable t) {
        return Result.fail("服务暂时不可用，请稍候再试");
    }
}
```

自定义blockHandlerClass和fallbackClass

```


@RestController
@RequestMapping("/")
public class SentinelController {

    @Value("${server.port}")
    private String serverPort;

    //自定义blockHandlerClass和fallbackClass
    //注意对应的函数必需为 static 函数，否则无法解析。
    @RequestMapping(value="/myBlockHandler", produces = MediaType.APPLICATION_JSON_VALUE)
    @SentinelResource(value="myBlockHandler", blockHandlerClass = MyBlockHandler.class, blockHandler = "blockHandler", 
        fallbackClass = MyBlockHandler.class, fallback = "exceptionHandler")
    public Result myBlockHandler(String a) {
        if(StringUtils.isBlank(a)) {
            throw new RuntimeException("参数错误");
        }
        return Result.ok(null, a);
    } 
}
```

**Sentinel exceptionsToIgnore：**

（since 1.6.0），用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

**Sentinel全局限流降级结果返回**

实现**BlockExceptionHandler**接口，实现**handle**方法。
**旧版本**的sentinel是实现**UrlBlockHandler**接口，新版本（1.8版本）是没有这个接口的。
默认的结果处理类是：

```
com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.DefaultBlockExceptionHandler
```

处理

```
@Component
public class SentinelExceptionHandler implements BlockExceptionHandler {

        @Override
        public void handle(HttpServletRequest request, HttpServletResponse response, BlockException ex) throws Exception {
        
        String msg = null;
        
            if (ex instanceof FlowException) {
                msg = "访问频繁，请稍候再试";
                
            } else if (ex instanceof DegradeException) {
                msg = "系统降级";
                
            } else if (ex instanceof ParamFlowException) {
                msg = "热点参数限流";
                
            } else if (ex instanceof SystemBlockException) {
                msg = "系统规则限流或降级";
                
            } else if (ex instanceof AuthorityException) {
                msg = "授权规则不通过";
                
            } else {
                msg = "未知限流降级";
            }
            // http状态码
            response.setStatus(500);
            response.setCharacterEncoding("utf-8");
            response.setHeader("Content-Type", "application/json;charset=utf-8");
            response.setContentType("application/json;charset=utf-8");
            
            response.getWriter().write(JSONUtil.toJsonStr(Result.failMsg(msg)));
        }

}
```

**Sentinel持久化配置**

**pom.xml引入依赖**

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

**sentinel结合Nacos持久化配置**
sentinel持久化支持的类型有，详情见：

```
com.alibaba.cloud.sentinel.datasource.config.DataSourcePropertiesConfiguration
```

```
private FileDataSourceProperties **file**;
private NacosDataSourceProperties **nacos**;
private ZookeeperDataSourceProperties **zk**;
private ApolloDataSourceProperties **apollo**;
private RedisDataSourceProperties **redis**;
private ConsulDataSourceProperties **consul**;
```

**sentinel结合Nacos持久化配置：**

```
#sentinel结合Nacos持久化配置
#配置的属性见：com.alibaba.cloud.sentinel.datasource.config.NacosDataSourceProperties
spring.cloud.sentinel.datasource.ds1.nacos.server-addr=http://127.0.0.1:8848
spring.cloud.sentinel.datasource.ds1.nacos.data-id=sentinel-config
#//group-id默认是：DEFAULT_GROUP
spring.cloud.sentinel.datasource.ds1.nacos.group-id=SENTINEL_GROUP
#RuleType类型见：com.alibaba.cloud.sentinel.datasource.RuleType
#data-type这个属性没有提示，和Nacos配置的类型保持一致，配置在：com.alibaba.cloud.sentinel.datasource.config.AbstractDataSourceProperties
spring.cloud.sentinel.datasource.ds1.nacos.data-type=json
#authority（授权规则）、degrade（降级规则）、flow（流控规则）、param（热点规则）、system（系统规则）五种规则持久化到Nacos中。 另外还增加网关的两个（api分组，限流）
#rule-type这个属性没有提示，为空时，会报空指针错误
spring.cloud.sentinel.datasource.ds1.nacos.rule-type=flow
```

每种数据源都有两个共同的配置项： data-type、 converter-class 以及 rule-type，配置在：

```
com.alibaba.cloud.sentinel.datasource.config.AbstractDataSourceProperties
```

data-type 配置项表示 Converter 类型，Spring Cloud Alibaba Sentinel 默认提供两种内置的值，分别是 **json 和 xml** (不填默认是json)。

如果不想使用内置的 json 或 xml 这两种 Converter，可以填写 custom 表示自定义 Converter，然后再配置 converter-class 配置项，该配置项需要写类的全路径名(比如 spring.cloud.sentinel.datasource.ds1.file.converter-class=com.alibaba.cloud.examples.JsonFlowRuleListConverter)。
rule-type 配置表示该数据源中的规则属于哪种类型的规则(flow，degrade，authority，system, param-flow, gw-flow, gw-api-group)。

注意：
当某个数据源规则信息加载失败的情况下，不会影响应用的启动，会在日志中打印出错误信息。
默认情况下，xml 格式是不支持的。需要添加 jackson-dataformat-xml 依赖后才会自动生效。
如果规则加载没有生效，有可能是 jdk 版本导致的，请关注 759 issue 的处理。

sentinel缺少**spring.cloud.sentinel.datasource.ds1.nacos.rule-type**报错：DataSource ds1 build error: null

```
2021-03-31 11:20:59.718 ERROR 12168 --- [ restartedMain] c.a.c.s.c.SentinelDataSourceHandler : [Sentinel Starter] DataSource ds1 build error: null
java.lang.NullPointerException: null
at com.alibaba.cloud.sentinel.custom.SentinelDataSourceHandler.lambda$registerBean$3(SentinelDataSourceHandler.java:185) ~[spring-cloud-starter-alibaba-sentinel-2.2.5.RELEASE.jar:2.2.5.RELEASE]
at java.util.HashMap.forEach(HashMap.java:1289) ~[na:1.8.0_241]
at com.alibaba.cloud.sentinel.custom.SentinelDataSourceHandler.registerBean(SentinelDataSourceHandler.java:128) ~[spring-cloud-starter-alibaba-sentinel-2.2.5.RELEASE.jar:2.2.5.RELEASE]
at com.alibaba.cloud.sentinel.custom.SentinelDataSourceHandler.lambda$afterSingletonsInstantiated$0(SentinelDataSourceHandler.java:93) ~[spring-cloud-starter-alibaba-sentinel-2.2.5.RELEASE.jar:2.2.5.RELEASE]
at java.util.TreeMap.forEach(TreeMap.java:1005) ~[na:1.8.0_241]
at com.alibaba.cloud.sentinel.custom.SentinelDataSourceHandler.afterSingletonsInstantiated(SentinelDataSourceHandler.java:80) ~[spring-cloud-starter-alibaba-sentinel-2.2.5.RELEASE.jar:2.2.5.RELEASE]
```

原因就是com.alibaba.cloud.sentinel.custom.SentinelDataSourceHandler.registerBean(AbstractDataSourceProperties, String)中的dataSourceProperties.getRuleType()为空，**所以配置文件中的spring.cloud.sentinel.datasource.ds1.nacos.rule-type不能为空**。

```
// converter type now support xml or json.
// The bean name of these converters wrapped by
// 'sentinel-{converterType}-{ruleType}-converter'
builder.addPropertyReference("converter", "sentinel-" + propertyValue.toString() + "-" + dataSourceProperties.getRuleType().getName() + "-converter");
```

 

提示是没有rule-type属性的，可能是因为rule-type是继承过来的或者是开发工具的问题。

rule-type在Sts提示没有该属性，但启动是没问题的。

**四、其它限流规则**

**1、Sentinel熔断降级规则说明**
熔断降级规则（DegradeRule）包含下面几个重要的属性：



```
Field            说明                                                    　　　　　　　　　　　　　　　　默认值
resource        资源名，即规则的作用对象    
grade            熔断策略，支持慢调用比例/异常比例/异常数策略                            　　　　　　　　　　慢调用比例
count            慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值    
timeWindow        熔断时长，单位为 s    
minRequestAmount    熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入）        　　5
statIntervalMs        统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入）                        　　1000 ms
slowRatioThreshold    慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）    
```



 

**2、Sentinel【热点规则】热点参数规则**
热点参数规则（ParamFlowRule）类似于流量控制规则（FlowRule）：



```
属性            说明                                                            　　　　　　　　　　　　　　　　　　默认值
resource        资源名，必填    
count            限流阈值，必填    
grade            限流模式                                                        　　　　　　　　　　　　　　　　　　QPS 模式
durationInSec        统计窗口时间长度（单位为秒），1.6.0 版本开始支持                                    　　　　　　　　1s
controlBehavior    流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持                                　　　　　　快速失败
maxQueueingTimeMs    最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持                            　　　　　　　　0ms
paramIdx        热点参数的索引，必填，对应 SphU.entry(xxx, args) 中的参数索引位置    
paramFlowItemList    参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 count 阈值的限制。仅支持基本类型和字符串类型    
clusterMode        是否是集群参数流控规则                                                　　　　　　　　　　　　　　　　false
clusterConfig        集群流控相关配置
```



 

**3、Sentinel系统规则**
系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。
系统保护规则是应用整体维度的，而不是资源维度的，并且仅对入口流量生效。入口流量指的是进入应用的流量（EntryType.IN），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：
Load 自适应（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 maxQps * minRt 估算得出。设定参考值一般是 CPU cores * 2.5。
CPU usage（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
平均 RT：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
并发线程数：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
入口 QPS：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。


系统规则json文件配置（最简单配置），配置QPS为1：

```
[
    {
        "qps":1.0
    }
]
```

 

系统规则json文件配置，配置QPS为1（这段json配置是从新增那里获取的，能正常持久化）：

```
[
    {
        "id":12,
        "app":"SPRING-CLOUD-SENTINEL",
        "ip":"192.168.170.1",
        "port":8720,
        "highestSystemLoad":-1.0,
        "avgRt":-1,
        "maxThread":-1,
        "qps":1.0,
        "highestCpuUsage":-1.0,
        "gmtCreate":null,
        "gmtModified":null
    }
]
```

 

highestSystemLoad：对应页面的Load
avgRt：对应页面的RT
maxThread：对应页面的线程数
qps：对应入口QPS
highestCpuUsage：对应CUP使用率

 

**五、、Feign 支持**
Sentinel 适配了 Feign 组件。如果想使用，除了引入 spring-cloud-starter-alibaba-sentinel 的依赖外还需要 2 个步骤：

配置文件打开 Sentinel 对 Feign 的支持：feign.sentinel.enabled=true

加入 spring-cloud-starter-openfeign 依赖使 Sentinel starter 中的自动化配置类生效：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

 

 

**六、RestTemplate 支持**
Spring Cloud Alibaba Sentinel 支持对 RestTemplate 的服务调用使用 Sentinel 进行保护，在构造 RestTemplate bean的时候需要加上 **@SentinelRestTemplate** 注解。

```
@Bean
@SentinelRestTemplate(blockHandler = "handleException", blockHandlerClass = ExceptionUtil.class)
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

@SentinelRestTemplate 注解的属性支持限流(blockHandler, blockHandlerClass)和降级(fallback, fallbackClass)的处理。
其中 blockHandler 或 fallback 属性对应的方法必须是对应 blockHandlerClass 或 fallbackClass 属性中的静态方法。

注意：应用启动的时候会检查 @SentinelRestTemplate 注解对应的限流或降级方法是否存在，如不存在会抛出异常

@SentinelRestTemplate 注解的限流(blockHandler, blockHandlerClass)和降级(fallback, fallbackClass)属性不强制填写。
当使用 RestTemplate 调用被 Sentinel 熔断后，会返回 RestTemplate request block by sentinel 信息，或者也可以编写对应的方法自行处理返回信息。这里提供了 SentinelClientHttpResponse 用于构造返回信息。
Sentinel RestTemplate 限流的资源规则提供两种粒度：
httpmethod:schema://host:port/path：协议、主机、端口和路径
httpmethod:schema://host:port：协议、主机和端口

 

 

**七、动态数据源支持**
SentinelProperties 内部提供了 TreeMap 类型的 datasource 属性用于配置数据源信息。

比如配置 4 个数据源：

```
spring.cloud.sentinel.datasource.ds1.file.file=classpath: degraderule.json
spring.cloud.sentinel.datasource.ds1.file.rule-type=flow

#spring.cloud.sentinel.datasource.ds1.file.file=classpath: flowrule.json
#spring.cloud.sentinel.datasource.ds1.file.data-type=custom
#spring.cloud.sentinel.datasource.ds1.file.converter-class=com.alibaba.cloud.examples.JsonFlowRuleListConverter
#spring.cloud.sentinel.datasource.ds1.file.rule-type=flow

spring.cloud.sentinel.datasource.ds2.nacos.server-addr=localhost:8848
spring.cloud.sentinel.datasource.ds2.nacos.data-id=sentinel
spring.cloud.sentinel.datasource.ds2.nacos.group-id=DEFAULT_GROUP
spring.cloud.sentinel.datasource.ds2.nacos.data-type=json
spring.cloud.sentinel.datasource.ds2.nacos.rule-type=degrade

spring.cloud.sentinel.datasource.ds3.zk.path = /Sentinel-Demo/SYSTEM-CODE-DEMO-FLOW
spring.cloud.sentinel.datasource.ds3.zk.server-addr = localhost:2181
spring.cloud.sentinel.datasource.ds3.zk.rule-type=authority

spring.cloud.sentinel.datasource.ds4.apollo.namespace-name = application
spring.cloud.sentinel.datasource.ds4.apollo.flow-rules-key = sentinel
spring.cloud.sentinel.datasource.ds4.apollo.default-flow-rule-value = test
spring.cloud.sentinel.datasource.ds4.apollo.rule-type=param-flow
```

d1, ds2, ds3, ds4 是 ReadableDataSource 的名字，可随意编写。后面的 file ，zk ，nacos , apollo 就是对应具体的数据源，它们后面的配置就是这些具体数据源各自的配置。注意数据源的依赖要单独引入（比如 sentinel-datasource-nacos)。

每种数据源都有两个共同的配置项： data-type、 converter-class 以及 rule-type。
data-type 配置项表示 Converter 类型，Spring Cloud Alibaba Sentinel 默认提供两种内置的值，分别是 json 和 xml (不填默认是json)。
如果不想使用内置的 json 或 xml 这两种 Converter，可以填写 custom 表示自定义 Converter，然后再配置 converter-class 配置项，该配置项需要写类的全路径名(比如 spring.cloud.sentinel.datasource.ds1.file.converter-class=com.alibaba.cloud.examples.JsonFlowRuleListConverter)。
rule-type 配置表示该数据源中的规则属于哪种类型的规则(flow，degrade，authority，system, param-flow, gw-flow, gw-api-group)。

 

 

**八、Spring Cloud Gateway 支持**

若想跟 Sentinel Starter 配合使用，需要加上 spring-cloud-alibaba-sentinel-gateway 依赖，同时需要添加 spring-cloud-starter-gateway 依赖来让 spring-cloud-alibaba-sentinel-gateway 模块里的 Spring Cloud Gateway 自动化配置类生效：

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

同时请将 spring.cloud.sentinel.filter.enabled 配置项置为 false（若在网关流控控制台上看到了 URL 资源，就是此配置项没有置为 false）。

 

 

**九、Endpoint 支持**
在使用 Endpoint 特性之前需要在 Maven 中添加 spring-boot-starter-actuator 依赖，并在配置中允许 Endpoints 的访问。


Spring Boot 1.x 中添加配置 management.security.enabled=false。暴露的 endpoint 路径为 /sentinel
Spring Boot 2.x 中添加配置 management.endpoints.web.exposure.include=*。暴露的 endpoint 路径为 /actuator/sentinel


Sentinel Endpoint 里暴露的信息非常有用。包括当前应用的所有规则信息、日志目录、当前实例的 IP，Sentinel Dashboard 地址，Block Page，应用与 Sentinel Dashboard 的心跳频率等等信息。







# Sentinel源码

# 总体流程

先来了解一下总体流程：

![setinel整体架构](png\setinel整体架构.png)



上面的图是官网的图，从设计模式上来看，典型的的责任链模式。外部请求进来后，要经过责任链上各个节点的处理，而Sentinel的限流、熔断就是通过责任链上的这些节点实现的。

从限流算法来看，Sentinel使用滑动窗口算法来进行限流。要想深入了解原理，还是得从源码上入手，下面，直接进入Sentinel的源码阅读。

## 核心组件

### Resource

resource是sentinel中最重要的一个概念，sentinel通过资源来保护具体的业务代码或其他后方服务。sentinel把复杂的逻辑给屏蔽掉了，用户只需要为受保护的代码或服务定义一个资源，然后定义规则就可以了，剩下的通通交给sentinel来处理了。并且资源和规则是解耦的，规则甚至可以在运行时动态修改。定义完资源后，就可以通过在程序中埋点来保护你自己的服务了，埋点的方式有两种：

- try-catch 方式（`通过 SphU.entry(...)`），当 catch 到BlockException时执行异常处理(或fallback)
- if-else 方式（`通过 SphO.entry(...)`），当返回 false 时执行异常处理(或fallback)

以上这两种方式都是通过硬编码的形式定义资源然后进行资源埋点的，对业务代码的侵入太大，从0.1.1版本开始，sentinel加入了注解的支持，可以通过注解来定义资源，具体的注解为：SentinelResource 。通过注解除了可以定义资源外，还可以指定 blockHandler 和 fallback 方法。

在sentinel中具体表示资源的类是：ResourceWrapper ，他是一个抽象的包装类，包装了资源的 Name 和EntryType。他有两个实现类，分别是：StringResourceWrapper 和 MethodResourceWrapper。顾名思义，StringResourceWrapper 是通过对一串字符串进行包装，是一个通用的资源包装类，MethodResourceWrapper 是对方法调用的包装。

### Context

Context是对资源操作时的上下文环境，每个资源操作(`针对Resource进行的entry/exit`)必须属于一个Context，如果程序中未指定Context，会创建name为"sentinel_default_context"的默认Context。一个Context生命周期内可能有多个资源操作，Context生命周期内的最后一个资源exit时会清理该Context，这也预示这整个Context生命周期的结束。Context主要属性如下：

```
public class Context {
   // context名字，默认名字 "sentinel_default_context"
   private final String name;
   // context入口节点，每个context必须有一个entranceNode
   private DefaultNode entranceNode;
   // context当前entry，Context生命周期中可能有多个Entry，所有curEntry会有变化
   private Entry curEntry;
   // The origin of this context (usually indicate different invokers, e.g. service consumer name or origin IP).
   private String origin = "";
   private final boolean async;
}
```

注意：一个Context生命期内Context只能初始化一次，因为是存到ThreadLocal中，并且只有在非null时才会进行初始化。

如果想在调用 SphU.entry() 或 SphO.entry() 前，自定义一个context，则通过ContextUtil.enter()方法来创建。context是保存在ThreadLocal中的，每次执行的时候会优先到ThreadLocal中获取，为null时会调用 `MyContextUtil.myEnter(Constants.CONTEXT_DEFAULT_NAME, "", resourceWrapper.getType())`创建一个context。当Entry执行exit方法时，如果entry的parent节点为null，表示是当前Context中最外层的Entry了，此时将ThreadLocal中的context清空。

### Entry

刚才在Context身影中也看到了Entry的出现，现在就谈谈Entry。每次执行 SphU.entry() 或 SphO.entry() 都会返回一个Entry，Entry表示一次资源操作，内部会保存当前invocation信息。在一个Context生命周期中多次资源操作，也就是对应多个Entry，这些Entry形成parent/child结构保存在Entry实例中，entry类CtEntry结构如下：

```
class CtEntry extends Entry {
   protected Entry parent = null;
   protected Entry child = null;

   protected ProcessorSlot<Object> chain;
   protected Context context;
}
public abstract class Entry implements AutoCloseable {
   private long createTime;
   private Node curNode;
   /**
    * {@link Node} of the specific origin, Usually the origin is the Service Consumer.
    */
   private Node originNode;
   private Throwable error; // 是否出现异常
   protected ResourceWrapper resourceWrapper; // 资源信息
}
```

### DefaultNode

Node（*关于StatisticNode的讨论放到下一小节*）默认实现类DefaultNode，该类还有一个子类EntranceNode；context有一个entranceNode属性，Entry中有一个curNode属性。

- **EntranceNode**：该类的创建是在初始化Context时完成的（ContextUtil.trueEnter方法），注意该类是针对Context维度的，也就是一个context有且仅有一个EntranceNode。
- **DefaultNode**：该类的创建是在NodeSelectorSlot.entry完成的，当不存在context.name对应的DefaultNode时会新建（new DefaultNode(resourceWrapper, null)，对应resouce）并保存到本地缓存（NodeSelectorSlot中private volatile Map<String, DefaultNode> map）；获取到context.name对应的DefaultNode后会将该DefaultNode设置到当前context的curEntry.curNode属性，也就是说，在NodeSelectorSlot中是一个context有且仅有一个DefaultNode。

看到这里，你是不是有疑问？为什么一个context有且仅有一个DefaultNode，我们的resouece跑哪去了呢，其实，这里的一个context有且仅有一个DefaultNode是在NodeSelectorSlot范围内，NodeSelectorSlot是ProcessorSlotChain中的一环，获取ProcessorSlotChain是根据Resource维度来的。总结为一句话就是：**针对同一个Resource，多个context对应多个DefaultNode；针对不同Resource，(不管是否是同一个context)对应多个不同DefaultNode**。

# 源码阅读

## **源码阅读入口及总体流程**

读源码先得找到源码入口。我们经常使用@ SentinelResource来标记一个方法，可以将这个被@ SentinelResource标记的方法看成是一个Sentinel资源。因此，我们以@ SentinelResource为入口，找到其切面，看看切面拦截后所做的工作，就可以明确Sentinel的工作原理了。直接看注解@SentinelResource的切面代码（SentinelResourceAspect）。

```
@Around("sentinelResourceAnnotationPointcut()")
public Object invokeResourceWithSentinel(ProceedingJoinPoint pjp) throws Throwable {
    //标有注解(@SentinelResource)的目标原始方法
    Method originMethod = resolveMethod(pjp);
    //获取注解对象(@SentinelResource)
    SentinelResource annotation = originMethod.getAnnotation(SentinelResource.class);
    if (annotation == null) {
     	 throw new IllegalStateException("Wrong state for SentinelResource annotation");
    }
    //资源名称
    String resourceName = getResourceName(annotation.value(), originMethod);

    EntryType entryType = annotation.entryType();
    Entry entry = null;
    try {
        //sentinel 逻辑代码
        entry = SphU.entry(resourceName, entryType, 1, pjp.getArgs());
        //执行目标方法
        Object result = pjp.proceed();
        return result;
    } catch (BlockException ex) {
        // 处理BlockException
        return handleBlockException(pjp, annotation, ex);
    } catch (Throwable ex) {
        // 处理非BlockException
        // 获取忽略处理的异常
        Class<? extends Throwable>[] exceptionsToIgnore = annotation.exceptionsToIgnore();
        //判断当前异常是否在 忽略异常列表中，如果存在，则直接抛出
        if (exceptionsToIgnore.length > 0 && exceptionBelongsTo(ex, exceptionsToIgnore)) {
         	 throw ex;
        }
        //如果当前异常在exceptionsToTrace属性中定义了，就进行处理
        if (exceptionBelongsTo(ex, annotation.exceptionsToTrace())) {
            traceException(ex);
            return handleFallback(pjp, annotation, ex);
        }
        //前面的条件都不符合，则直接抛出异常
        throw ex;
    } finally {
        if (entry != null) {
         	 entry.exit(1, pjp.getArgs());
        }
    }
}
```

可以清晰的看到Sentinel的行为方式。进入SentinelResource切面后，会执行SphU.entry方法，在这个方法中会对被拦截方法做限流和熔断的逻辑处理。

如果触发熔断和限流，会抛出BlockException，我们可以指定blockHandler方法来处理BlockException。而对于业务上的异常，我们也可以配置fallback方法来处理被拦截方法调用产生的异常。

所以，Sentinel熔断限流的处理主要是在SphU.entry方法中，其主要处理逻辑见下图源码。

```
    private Entry entryWithPriority(ResourceWrapper resourceWrapper, int count, boolean prioritized, Object... args) throws BlockException {
        Context context = ContextUtil.getContext();
        if (context instanceof NullContext) {
            return new CtEntry(resourceWrapper, (ProcessorSlot)null, context);
        } else {
            if (context == null) {
                context = CtSph.InternalContextUtil.internalEnter("sentinel_default_context");
            }

            if (!Constants.ON) {
                return new CtEntry(resourceWrapper, (ProcessorSlot)null, context);
            } else {
                ProcessorSlot<Object> chain = this.lookProcessChain(resourceWrapper);
                if (chain == null) {
                    return new CtEntry(resourceWrapper, (ProcessorSlot)null, context);
                } else {
                    CtEntry e = new CtEntry(resourceWrapper, chain, context);

                    try {
                        chain.entry(context, resourceWrapper, (Object)null, count, prioritized, args);
                    } catch (BlockException var9) {
                        e.exit(count, args);
                        throw var9;
                    } catch (Throwable var10) {
                        RecordLog.info("Sentinel unexpected exception", var10);
                    }

                    return e;
                }
            }
        }
    }

```

可见，在SphU.entry方法中，Sentinel实现限流、熔断等功能的流程可以总结如下：

- 获取Sentinel上下文（Context）；
- 获取资源对应的责任链；
- 生成资源调用凭证（Entry）；
- 执行责任链中各个节点。

接下来，围绕这几个方面，对Sentinel的服务机制做一个系统的阐述。

## **获取Sentinel上下文（Context）**

Context，顾名思义，就是Sentinel熔断限流执行的上下文，包含资源调用的节点和Entry信息。

来看看Context的特征：

- Context是线程持有的，利用ThreadLocal与当前线程绑定。

```
private static ThreadLocal<Context> contextHolder = new ThreadLocal();
```

- Context包含的内容

```
public class Context {
    // 名称
    private final String name;
    // 入口
    private DefaultNode entranceNode;
    // 当前Entry
    private Entry curEntry;
    private String origin;
    private final boolean async;
    ...
    
    }
```

这里就引出了Sentinel的三个比较重要的概念：**Conetxt，Node，Entry**。

这三个类是Sentinel的核心类，提供了资源调用路径、资源调用统计等信息。

**Context**

> Context是当前线程所持有的Sentinel上下文。
>
> 进入Sentinel的逻辑时，会首先获取当前线程的Context，如果没有则新建。当任务执行完毕后，会清除当前线程的context。Context 代表调用链路上下文，贯穿一次调用链路中的所有 Entry。
>
> Context 维持着入口节点（entranceNode）、本次调用链路的 当前节点（curNode）、调用来源（origin）等信息。Context 名称即为调用链路入口名称。

**Node**

> Node是对一个@SentinelResource标记的资源的统计包装。
>
> Context中记录本当前线程资源调用的入口节点。
>
> 我们可以通过入口节点的childList，可以追溯资源的调用情况。而每个节点都对应一个@SentinelResource标记的资源及其统计数据，例如：passQps，blockQps，rt等数据。

**Entry**

> Entry是Sentinel中用来表示是否通过限流的一个凭证，如果能正常返回，则说明你可以访问被Sentinel保护的后方服务，否则Sentinel会抛出一个BlockException。
>
> 另外，它保存了本次执行entry()方法的一些基本信息，包括资源的Context、Node、对应的责任链等信息，后续完成资源调用后，还需要更具获得的这个Entry去执行一些善后操作，包括退出Entry对应的责任链，完成节点的一些统计信息更新，清除当前线程的Context信息等。

## Context的创建与销毁

首先我们要清楚的一点就是，每次执行entry()方法，试图冲破一个资源时，都会生成一个上下文。这个上下文中会保存着调用链的根节点和当前的入口。

Context是通过ContextUtil创建的，具体的方法是trueEntry，代码如下：

```
protected static Context trueEnter(String name, String origin) {
    // 先从ThreadLocal中获取
    Context context = contextHolder.get();
    if (context == null) {
        // 如果ThreadLocal中获取不到Context
        // 则根据name从map中获取根节点，只要是相同的资源名，就能直接从map中获取到node
        Map<String, DefaultNode> localCacheNameMap = contextNameNodeMap;
        DefaultNode node = localCacheNameMap.get(name);
        if (node == null) {
            // 省略部分代码
            try {
                LOCK.lock();
                node = contextNameNodeMap.get(name);
                if (node == null) {
                    // 省略部分代码
                    // 创建一个新的入口节点
                    node = new EntranceNode(new StringResourceWrapper(name, EntryType.IN), null);
                    Constants.ROOT.addChild(node);
                    // 省略部分代码
                }
            } finally {
                LOCK.unlock();
            }
        }
        // 创建一个新的Context，并设置Context的根节点，即设置EntranceNode
        context = new Context(node, name);
        context.setOrigin(origin);
        // 将该Context保存到ThreadLocal中去
        contextHolder.set(context);
    }
    return context;
}
```

上面的代码中我省略了部分代码，只保留了核心的部分。从源码中还是可以比较清晰的看出生成Context的过程：

- 1.先从ThreadLocal中获取，如果能获取到直接返回，如果获取不到则继续第2步
- 2.从一个static的map中根据上下文的名称获取，如果能获取到则直接返回，否则继续第3步
- 3.加锁后进行一次double check，如果还是没能从map中获取到，则创建一个EntranceNode，并把该EntranceNode添加到一个全局的ROOT节点中去，然后将该节点添加到map中去(这部分代码在上述代码中省略了)
- 4.根据EntranceNode创建一个上下文，并将该上下文保存到ThreadLocal中去，下一个请求可以直接获取

那保存在ThreadLocal中的上下文什么时候会清除呢？从代码中可以看到具体的清除工作在ContextUtil的exit方法中，当执行该方法时，会将保存在ThreadLocal中的context对象清除，具体的代码非常简单，这里就不贴代码了。

那ContextUtil.exit方法什么时候会被调用呢？有两种情况：一是主动调用ContextUtil.exit的时候，二是当一个入口Entry要退出，执行该Entry的trueExit方法的时候，此时会触发ContextUtil.exit的方法。但是有一个前提，就是当前Entry的父Entry为null时，此时说明该Entry已经是最顶层的根节点了，可以清除context。

## **获取@SentinelResource标记资源对应的责任链**

资源对应的责任链是限流逻辑具体执行的地方，采用的是典型的责任链模式。

先来看看默认的的责任链的组成：

```
public class DefaultSlotChainBuilder implements SlotChainBuilder {
    public DefaultSlotChainBuilder() {
    }

    public ProcessorSlotChain build() {
        ProcessorSlotChain chain = new DefaultProcessorSlotChain();
        List<ProcessorSlot> sortedSlotList = SpiLoader.of(ProcessorSlot.class).loadInstanceListSorted();
        Iterator var3 = sortedSlotList.iterator();

        while(var3.hasNext()) {
            ProcessorSlot slot = (ProcessorSlot)var3.next();
            if (!(slot instanceof AbstractLinkedProcessorSlot)) {
                RecordLog.warn("The ProcessorSlot(" + slot.getClass().getCanonicalName() + ") is not an instance of AbstractLinkedProcessorSlot, can't be added into ProcessorSlotChain", new Object[0]);
            } else {
                chain.addLast((AbstractLinkedProcessorSlot)slot);
            }
        }

        return chain;
    }
}
```

默认的处理节点

```
# Sentinel default ProcessorSlots
com.alibaba.csp.sentinel.slots.nodeselector.NodeSelectorSlot
com.alibaba.csp.sentinel.slots.clusterbuilder.ClusterBuilderSlot
com.alibaba.csp.sentinel.slots.logger.LogSlot
com.alibaba.csp.sentinel.slots.statistic.StatisticSlot
com.alibaba.csp.sentinel.slots.block.authority.AuthoritySlot
com.alibaba.csp.sentinel.slots.system.SystemSlot
com.alibaba.csp.sentinel.slots.block.flow.FlowSlot
com.alibaba.csp.sentinel.slots.block.degrade.DegradeSlot
```

默认的责任链中的处理节点包括NodeSelectorSlot、ClusterBuilderSlot、StatisticSlot、FlowSlot、DegradeSlot等。调用链（ProcessorSlotChain）和其中包含的所有Slot都实现了ProcessorSlot接口，采用责任链的模式执行各个节点的处理逻辑，并调用下一个节点。

每个节点都有自己的作用，后面将会看到这些节点具体是干什么的。

此外，相同资源（@SentinelResource标记的方法）对应的责任链是一致的。也就是说，每个资源对应一条单独的责任链，可以看下源码中资源责任链的获取逻辑：先从缓存获取，没有则新建。

```
    ProcessorSlot<Object> lookProcessChain(ResourceWrapper resourceWrapper) {
        ProcessorSlotChain chain = (ProcessorSlotChain)chainMap.get(resourceWrapper);
        if (chain == null) {
            synchronized(LOCK) {
                chain = (ProcessorSlotChain)chainMap.get(resourceWrapper);
                if (chain == null) {
                    if (chainMap.size() >= 6000) {
                        return null;
                    }

                    chain = SlotChainProvider.newSlotChain();
                    Map<ResourceWrapper, ProcessorSlotChain> newMap = new HashMap(chainMap.size() + 1);
                    newMap.putAll(chainMap);
                    newMap.put(resourceWrapper, chain);
                    chainMap = newMap;
                }
            }
        }

        return chain;
    }
```

## **生成调用凭证Entry**

生成的Entry是CtEntry。其构造参数包括资源包装（ResourceWrapper）、资源对应的责任链以及当前线程的Context。

```
class CtEntry extends Entry {
    protected Entry parent = null;
    protected Entry child = null;
    protected ProcessorSlot<Object> chain;
    protected Context context;
    protected LinkedList<BiConsumer<Context, Entry>> exitHandlers;

    CtEntry(ResourceWrapper resourceWrapper, ProcessorSlot<Object> chain, Context context) {
        super(resourceWrapper);
        this.chain = chain;
        this.context = context;
        this.setUpEntryFor(context);
    }

    private void setUpEntryFor(Context context) {
        if (!(context instanceof NullContext)) {
            this.parent = context.getCurEntry();
            if (this.parent != null) {
                ((CtEntry)this.parent).child = this;
            }

            context.setCurEntry(this);
        }
    }
    ...
    
    }
```

可以看到，新建CtEntry记录了当前资源的责任链和Context，同时更新Context，将Context的当前Entry设置为自己。可以看到，CtEntry是一个双向链表，构建了Sentinel资源的调用链路。

## **责任链的执行**

接下来就进入了责任链的执行。责任链和其中的Slot都实现了ProcessorSlot，责任链的entry方法会依次执行责任链各个slot，所以下面就进入了责任链中的各个Slot。为了突出重点，这次本文只研究与限流功能有关的Slot。

### **NodeSelectorSlot -- 获取当前资源对应Node，构建节点调用树**

此节点负责获取或者构建当前资源对应的Node，这个Node被用于后续资源调用的统计及限流和熔断条件的判断。同时，NodeSelectorSlot还会完成调用链路构建。来看源码：

```
public class NodeSelectorSlot extends AbstractLinkedProcessorSlot<Object> {
    private volatile Map<String, DefaultNode> map = new HashMap(10);

    public NodeSelectorSlot() {
    }

    public void entry(Context context, ResourceWrapper resourceWrapper, Object obj, int count, boolean prioritized, Object... args) throws Throwable {
        DefaultNode node = (DefaultNode)this.map.get(context.getName());
        if (node == null) {
            synchronized(this) {
                node = (DefaultNode)this.map.get(context.getName());
                if (node == null) {
                    node = new DefaultNode(resourceWrapper, (ClusterNode)null);
                    HashMap<String, DefaultNode> cacheMap = new HashMap(this.map.size());
                    cacheMap.putAll(this.map);
                    cacheMap.put(context.getName(), node);
                    this.map = cacheMap;
                    ((DefaultNode)context.getLastNode()).addChild(node);
                }
            }
        }

        context.setCurNode(node);
        this.fireEntry(context, resourceWrapper, node, count, prioritized, args);
    }

    public void exit(Context context, ResourceWrapper resourceWrapper, int count, Object... args) {
        this.fireExit(context, resourceWrapper, count, args);
    }
}
```

熟悉的代码风格。我们知道一个资源对应一个责任链。每个调用链中都有NodeSelectorSlot。NodeSelectSlot中的node缓存map是非静态变量，所以map只对当前这个资源共用，不同的资源对应的NodeSelectSlot及Node的缓存都是不一样的。

所以NodeSelectorSlot的的作用是：

- 在资源对应的调用链执行时，获取当前context对应的Node，这个Node代表着这个资源的调用情况。
- 将获取到的node设为当前node，添加到之前的node后面，形成树状的调用路径。（通过Context中的当前Entry进行）
- 触发下一个Slot的执行。

这里有个很有趣的问题，就是我们在责任链的NodeSelectorSlot中获取资源对应的Node时，为什么用的是Context的name，而不是SentinelResource的name呢？

首先，我们知道一个资源对应一条责任链。但是进入一个资源调用的Context却可能是不同的。如果使用资源名来作为key，获取对应的Node，那么通过不同context进来的调用方法获取到的Node就都是同一个了。所以通过这种方式，可以将相同resource对应的node按Context区分开。

**举个例子，S**entinel功能的实现不仅仅可以通过@SentinelResource注解方法来实现，也可以通过引入相关依赖（sentinel-dubbo-adapter），利用Dubbo的Filter机制直接对DUBBO接口进行保护。我们来比较@SentinelResource和Dubbo方式生成Context的区别：

**@SentinelResource**

> 生成的context的name是：sentinel_default_context。所有资源对应的Context都是这个值。

**Dubbo Filter方式**

> 生成的context的name是Dubbo的接口限定名或者方法限定名。

如果出现嵌套在Dubbo Filter方式下面的其他SentinelResource的资源调用，那么这些资源调用的就会就会出现不同的Context。

所以有这样一种情况，不同的dubbo接口进来，这些dubbo接口都调用了同一个@SentinelResource标记的方法，那么这个方法对应的SentinelReource的在执行时对应的Context就是不同的。

另一个问题是，既然资源按Context分出了不同的node，那我们想看资源总数统计是怎么办呢？这就涉及到ClusterNode了。详细可见ClusterBuilderSlot。

### **ClusterBuilderSlot -- 聚合相同资源不同Context的Node**

此节点负责聚合相同资源不同Context对应的Node，以供后续限流判断使用。

```
public class ClusterBuilderSlot extends AbstractLinkedProcessorSlot<DefaultNode> {
    private static volatile Map<ResourceWrapper, ClusterNode> clusterNodeMap = new HashMap();
    private static final Object lock = new Object();
    private volatile ClusterNode clusterNode = null;

    public ClusterBuilderSlot() {
    }

    public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count, boolean prioritized, Object... args) throws Throwable {
        if (this.clusterNode == null) {
            synchronized(lock) {
                if (this.clusterNode == null) {
                    this.clusterNode = new ClusterNode(resourceWrapper.getName(), resourceWrapper.getResourceType());
                    HashMap<ResourceWrapper, ClusterNode> newMap = new HashMap(Math.max(clusterNodeMap.size(), 16));
                    newMap.putAll(clusterNodeMap);
                    newMap.put(node.getId(), this.clusterNode);
                    clusterNodeMap = newMap;
                }
            }
        }

        node.setClusterNode(this.clusterNode);
        if (!"".equals(context.getOrigin())) {
            Node originNode = node.getClusterNode().getOrCreateOriginNode(context.getOrigin());
            context.getCurEntry().setOriginNode(originNode);
        }

        this.fireEntry(context, resourceWrapper, node, count, prioritized, args);
    }
```

可以看到，ClusterNode的获取是以资源名为key。ClusterNode将会成为当前node的一个属性，主要目的是为了聚合同一个资源不同Context情况下的多个node。默认的限流条件判断就是依据ClusterNode中的统计信息来进行的。

### **StatisticSlot -- 资源调用统计**

此节点主要负责资源调用的统计信息的计算和更新。与前面以及后面的slot不同，StatisticSlot的执行时先触发下一个slot的执行，等下面的slot执行完才会执行自己的逻辑。

```
public class StatisticSlot extends AbstractLinkedProcessorSlot<DefaultNode> {
    public StatisticSlot() {
    }

    public void entry(Context context, ResourceWrapper resourceWrapper, DefaultNode node, int count, boolean prioritized, Object... args) throws Throwable {
        Iterator var8;
        ProcessorSlotEntryCallback handler;
        try {
            this.fireEntry(context, resourceWrapper, node, count, prioritized, args);
            node.increaseThreadNum();
            node.addPassRequest(count);
            if (context.getCurEntry().getOriginNode() != null) {
                context.getCurEntry().getOriginNode().increaseThreadNum();
                context.getCurEntry().getOriginNode().addPassRequest(count);
            }

            if (resourceWrapper.getEntryType() == EntryType.IN) {
                Constants.ENTRY_NODE.increaseThreadNum();
                Constants.ENTRY_NODE.addPassRequest(count);
            }

            Iterator var13 = StatisticSlotCallbackRegistry.getEntryCallbacks().iterator();

            while(var13.hasNext()) {
                ProcessorSlotEntryCallback<DefaultNode> handler = (ProcessorSlotEntryCallback)var13.next();
                handler.onPass(context, resourceWrapper, node, count, args);
            }
        } catch (PriorityWaitException var10) {
            node.increaseThreadNum();
            if (context.getCurEntry().getOriginNode() != null) {
                context.getCurEntry().getOriginNode().increaseThreadNum();
            }

            if (resourceWrapper.getEntryType() == EntryType.IN) {
                Constants.ENTRY_NODE.increaseThreadNum();
            }

            var8 = StatisticSlotCallbackRegistry.getEntryCallbacks().iterator();

            while(var8.hasNext()) {
                handler = (ProcessorSlotEntryCallback)var8.next();
                handler.onPass(context, resourceWrapper, node, count, args);
            }
        } catch (BlockException var11) {
            BlockException e = var11;
            context.getCurEntry().setBlockError(var11);
            node.increaseBlockQps(count);
            if (context.getCurEntry().getOriginNode() != null) {
                context.getCurEntry().getOriginNode().increaseBlockQps(count);
            }

            if (resourceWrapper.getEntryType() == EntryType.IN) {
                Constants.ENTRY_NODE.increaseBlockQps(count);
            }

            var8 = StatisticSlotCallbackRegistry.getEntryCallbacks().iterator();

            while(var8.hasNext()) {
                handler = (ProcessorSlotEntryCallback)var8.next();
                handler.onBlocked(e, context, resourceWrapper, node, count, args);
            }

            throw e;
        } catch (Throwable var12) {
            context.getCurEntry().setError(var12);
            throw var12;
        }

    }
```

这也很好理解，作为统计组件，总要等熔断或者限流处理完之后才能做统计吧。下面看一下具体的统计过程。

![Sentinel中的统计](png\Sentinel中的统计.png)

上面这张图已经很清晰的描述了StatisticSlot的数据统计的过程。可以注意一下无异常和阻塞异常的情况，主要是更新线程数、通过请求数量和阻塞请求数量。不管是DefaultNode，还是ClusterNode，都继承自StatisticNode。所以Node的数据更新要来到StatisticNode。

参考Sentinel数据统计框图，描述了Node统计数据更新的大体流程如下：

![Sentinel统计流程](png\Sentinel统计流程.png)

我们从StatisticNode.addPassRequest()方法入手，以passQps为例，探究StatisticNode是如何更新通过请求的QPS计数的。

```
    public void addPassRequest(int count) {
        super.addPassRequest(count);
        this.clusterNode.addPassRequest(count);
    }
```

```
public class StatisticNode implements Node {
    private transient volatile Metric rollingCounterInSecond;
    private transient Metric rollingCounterInMinute;
    private LongAdder curThreadNum;
    private long lastFetchTime;

    public StatisticNode() {
        this.rollingCounterInSecond = new ArrayMetric(SampleCountProperty.SAMPLE_COUNT, IntervalProperty.INTERVAL);
        this.rollingCounterInMinute = new ArrayMetric(60, 60000, false);
        this.curThreadNum = new LongAdder();
        this.lastFetchTime = -1L;
    }

    public Map<Long, MetricNode> metrics() {
        long currentTime = TimeUtil.currentTimeMillis();
        currentTime -= currentTime % 1000L;
        Map<Long, MetricNode> metrics = new ConcurrentHashMap();
        List<MetricNode> nodesOfEverySecond = this.rollingCounterInMinute.details();
        long newLastFetchTime = this.lastFetchTime;
        Iterator var7 = nodesOfEverySecond.iterator();

        while(var7.hasNext()) {
            MetricNode node = (MetricNode)var7.next();
            if (this.isNodeInTime(node, currentTime) && this.isValidMetricNode(node)) {
                metrics.put(node.getTimestamp(), node);
                newLastFetchTime = Math.max(newLastFetchTime, node.getTimestamp());
            }
        }

        this.lastFetchTime = newLastFetchTime;
        return metrics;
    }
```

从源码可见，计数变量rollingCounterInSecond和rollingCounterInMinute都是Metric，两个变量的时间维度分别是秒和分钟。rollingCounterInSecond和rollingCounterInMinute用的是Metric的实现类ArrayMetric。

从ArrayMetric追溯下去：

```
public class ArrayMetric implements Metric {
    private final LeapArray<MetricBucket> data;

    public ArrayMetric(int sampleCount, int intervalInMs) {
        this.data = new OccupiableBucketLeapArray(sampleCount, intervalInMs);
    }

```

```
    public void addPass(int count) {
        WindowWrap<MetricBucket> wrap = this.data.currentWindow();
        ((MetricBucket)wrap.value()).addPass(count);
    }
```

统计信息都是保存到ArrayMetric的data，也就是LeapArray<MertricBucket>中的。

```
public abstract class LeapArray<T> {
    protected int windowLengthInMs;
    protected int sampleCount;
    protected int intervalInMs;
    private double intervalInSecond;
    protected final AtomicReferenceArray<WindowWrap<T>> array;
    private final ReentrantLock updateLock = new ReentrantLock();

    public LeapArray(int sampleCount, int intervalInMs) {
        AssertUtil.isTrue(sampleCount > 0, "bucket count is invalid: " + sampleCount);
        AssertUtil.isTrue(intervalInMs > 0, "total time interval of the sliding window should be positive");
        AssertUtil.isTrue(intervalInMs % sampleCount == 0, "time span needs to be evenly divided");
        this.windowLengthInMs = intervalInMs / sampleCount;
        this.intervalInMs = intervalInMs;
        this.intervalInSecond = (double)intervalInMs / 1000.0D;
        this.sampleCount = sampleCount;
        this.array = new AtomicReferenceArray(sampleCount);
    }
```

LeapArray是时间窗口数组。基本信息包括：时间窗口长度（ms，windowLengthInMs），取样数（也就是时间窗口的数量，sampleCount），时间间隔（ms，intervalInMs），以及时间窗口数组（array）。时间窗口长度、取样数及时间间隔有下面的关系：

windowLengthInMs = intervalInMs / sampleCount

代码中rollingCounterInSecond使用的intervalInMs 是1000（ms），也就是1s，sampleCount=2。所以，窗口时长就是windowLengthInMs = 500ms。rollingCounterInMinute使用的intervalInMs 是60 * 1000（ms），也就是60s。sampleCount=60，所以，windowLengthInMs = 1000ms，也就是1s。

时间窗口数组（array）是类型是AtomicReferenceArray，可见这是一个原子操作的的数组引用。数组元素类型是WindowWrap<MetricBucket>。windowWrap是对时间窗口的一个包装，包括窗口的开始时间（windowStart）及窗口的长度（windowLengthInMs），以及本窗口的计数器（value，类型为MetricBucket）。窗口实际的计数是由MetricBucket进行的，计数信息是保存在MetricBucket里计数器counters（类型为（LongAdder））。

回到StatisticNode.addPassRequest方法，以rollingCounterInSecond.addPass(count)为例，探究Sentinel如何进行滑动窗口计数的。

### **滑动窗口机制**

时间窗口是用WindowWrap对象表示的，其属性如下：

```
private final long windowLengthInMs;  // 时间窗口的长度
private long windowStart; // 时间窗口开始时间
private T value; // MetricBucket对象，保存各个指标数据
```

sentinel时间基准由tick线程来做，每1ms更新一次时间基准，逻辑如下：

```
currentTimeMillis = System.currentTimeMillis();
Thread daemon = new Thread(new Runnable() {
    @Override
    public void run() {
        while (true) {
            currentTimeMillis = System.currentTimeMillis();
            try {
                TimeUnit.MILLISECONDS.sleep(1);
            } catch (Throwable e) {
            }
        }
    }
});
daemon.setDaemon(true);
daemon.setName("sentinel-time-tick-thread");
daemon.start();
```

sentinel默认有每秒和每分钟的滑动窗口，对应的LeapArray类型，它们的初始化逻辑是：

```
protected int windowLengthInMs; // 单个滑动窗口时间值
protected int sampleCount; // 滑动窗口个数
protected int intervalInMs; // 周期值（相当于所有滑动窗口时间值之和）

public LeapArray(int sampleCount, int intervalInMs) {
    this.windowLengthInMs = intervalInMs / sampleCount;
    this.intervalInMs = intervalInMs;
    this.sampleCount = sampleCount;

    this.array = new AtomicReferenceArray<WindowWrap<T>>(sampleCount);
}
```

针对每秒滑动窗口，`windowLengthInMs=500，sampleCount=2，intervalInMs=1000`，针对每分钟滑动窗口，`windowLengthInMs=1000，sampleCount=60，intervalInMs=60000`，对应代码：

```
private transient volatile Metric rollingCounterInSecond = new ArrayMetric(2, 1000);
private transient Metric rollingCounterInMinute = new ArrayMetric(60, 60 * 1000);
```

currentTimeMillis时间基准（tick线程）每1ms更新一次，通过currentWindow(timeMillis)方法获取当前时间点对应的WindowWrap对象，然后更新对应的各种指标，用于做限流、降级时使用。注意，当前时间基准对应的事件窗口初始化时lazy模式，并且会复用的。

介绍了滑动时间窗口的创建过程，如下：

- 1、根据当前时间，算出该时间的timeId，timeId就是在整个时间轴的位置
- 2、据timeId算出当前时间窗口在采样窗口区间中的索引idx
- 3、根据当前时间算出当前窗口应该对应的窗口开始时间time，以毫秒为单位
- 4、循环判断直到获取到一个当前时间窗口
- 5、根据索引idx，在采样窗口数组中取得一个时间窗口old

### **获取当前时间窗口**

（1）取当前时间戳对应的数组下标

long timeId = time / windowLength

int idx = (int)(timeId % array.length());

time为当前时间，windowLength为时间窗口长度，rollingCounterInSecond的时间窗口长度是500ms。array 是单位时间内时间窗口的数量，rollingCounterInSecond的单位时间（1s）时间窗口数是2。timeId是当前时间对时间窗口的整除。time每增加一个windowLength的长度，timeId就会增加1，时间窗口就会往前滑动一个。

（2）计算窗口开始时间

窗口开始时间 = 当前时间（ms）-当前时间（ms）%时间窗口长度（ms）

获取的窗口开始时间均为时间窗口的整数倍。

（3）获取时间窗口

首先，根据数组下标从LeapArray的数组中获取时间窗口。

- 如果获取到的时间窗口自为空，则新建时间窗口（CAS）。
- 如果获取到的时间窗口非空，且时间窗口的开始时间等于我们计算的开始时间，说明当前时间正好在这个时间窗口里，直接返回该时间窗口。
-  如果获取到的时间窗口非空，且时间窗口的开始时间小于我们计算的开始时间，说明时间窗口已经过期（距离上次获取时间窗口已经过去比较久的场景），需要更新时间窗口（加锁操作），将时间窗口的开始时间设为计算出来的开始时间，将时间窗口里的计数器重置为0。
-  如果获取到的时间窗口非空，且时间窗口的开始时间大于我们计算的开始时间，创建新的时间窗口。这个一般不会走进这个分支，因为说明当前时间已经落后于时间窗口了，获取到的时间窗口是将来的时间，那就没有意义了。



### **对时间窗口的计数器进行累加**

时间窗口计数器是一个LongAdder数组，这个数组用于存放通过请求数、异常请求数、阻塞请求数等数据。如下图：

```
public enum MetricEvent {
    PASS,    // 通过计数
    BLOCK,   // 阻塞计数
    EXCEPTION, // 异常计数
    SUCCESS,   //成功计数
    RT,        //响应时间
    OCCUPIED_PASS;

    private MetricEvent() {
    }
}
```

其中，通过计数、阻塞计数、异常计数为执行StatisticSlot的entry方法时更新。成功计数及响应时间是执行StatisticSlot的exit方法时更新。其实就是分别在被拦截方法执行前和执行后进行相应计数的更新。当然，addPass就是在计数数组的第一个元素上进行累加。

计数数组元素类型是LongAdder。LongAdder是JDK8添加到JUC中的。它是一个线程安全的、比Atomic*系工具性能更好的"计数器"。



### **FlowSlot -- 限流判断**

FlowSlot是进行限流条件判断的节点。之前在StatisticSlot对相关资源调用做的统计，在FlowSlot限流判断时将会得到使用。

直接来到限流操作的核心逻辑–限流规则检查器（FlowRuleChecker）：

```
public class FlowRuleChecker {
    public FlowRuleChecker() {
    }

    public void checkFlow(Function<String, Collection<FlowRule>> ruleProvider, ResourceWrapper resource, Context context, DefaultNode node, int count, boolean prioritized) throws BlockException {
        if (ruleProvider != null && resource != null) {
            Collection<FlowRule> rules = (Collection)ruleProvider.apply(resource.getName());
            if (rules != null) {
                Iterator var8 = rules.iterator();

                while(var8.hasNext()) {
                    FlowRule rule = (FlowRule)var8.next();
                    if (!this.canPassCheck(rule, context, node, count, prioritized)) {
                        throw new FlowException(rule.getLimitApp(), rule);
                    }
                }
            }

        }
    }
```

主要的流程包括：

- 获取资源对应的限流规则
- 根据限流规则检查是否被限流

如果被限流，则抛出限流异常FlowException。FlowException继承自BlockException。

那么FlowSlot检查是否限流的过程是怎么样的？

默认情况下，限流使用的节点是当前节点的cluster node。主要分析的限流方式是QPS限流。来看一下限流的关键代码（DefaultController）：

```
public class DefaultController implements TrafficShapingController {
    private static final int DEFAULT_AVG_USED_TOKENS = 0;
    private double count;
    private int grade;

    public DefaultController(double count, int grade) {
        this.count = count;
        this.grade = grade;
    }

    public boolean canPass(Node node, int acquireCount) {
        return this.canPass(node, acquireCount, false);
    }

    public boolean canPass(Node node, int acquireCount, boolean prioritized) {
        int curCount = this.avgUsedTokens(node);
        if ((double)(curCount + acquireCount) > this.count) {
            if (prioritized && this.grade == 1) {
                long currentTime = TimeUtil.currentTimeMillis();
                long waitInMs = node.tryOccupyNext(currentTime, acquireCount, this.count);
                if (waitInMs < (long)OccupyTimeoutProperty.getOccupyTimeout()) {
                    node.addWaitingRequest(currentTime + waitInMs, acquireCount);
                    node.addOccupiedPass(acquireCount);
                    this.sleep(waitInMs);
                    throw new PriorityWaitException(waitInMs);
                }
            }

            return false;
        } else {
            return true;
        }
    }
```

- 获取节点的当前qps计数；
- 判断获取新的计数后是否超过阈值
- 超过阈值单返回false，表示被限流，后面会抛出FlowException。否则返回true，不被限流。

可以看到限流判断非常简单，只需要对qps计数进行检查就可以了。这归功于StatisticSlot做的数据统计。



### **责任链小结**

通过上面的讲解，再来看下面这张图，是不是很清晰了？

![setinel整体架构](png\setinel整体架构.png)

( 引用于Sentinel官网)

NodeSelectorSlot用于获取资源对应的Node，并构建Node调用树，将SentinelSource的调用链路以Node Tree的形式组起来。ClusterBuilderSlot为当前Node创建对应的ClusterNode，聚合相同资源对应的不同Context的Node，后续的限流依据就是这个ClusterNode。

ClusterNode继承自StatisticNode，记录着相应资源处理的一些统计数据。StatisticSlot用于更新资源调用的相关计数，用于后续的限流判断使用。FlowSlot根据资源对应Node的调用计数，判断是否进行限流。至此，Sentinel的责任链执行逻辑就完整了。



## **Sentienl 的收尾工作**

无论执行成功还是失败，或者是阻塞，都会执行Entry.exit()方法，来看一下这个方法。

```

    protected void exitForContext(Context context, int count, Object... args) throws ErrorEntryFreeException {
        if (context != null) {
            if (context instanceof NullContext) {
                return;
            }

            if (context.getCurEntry() != this) {
                String curEntryNameInContext = context.getCurEntry() == null ? null : context.getCurEntry().getResourceWrapper().getName();

                for(CtEntry e = (CtEntry)context.getCurEntry(); e != null; e = (CtEntry)e.parent) {
                    e.exit(count, args);
                }

                String errorMessage = String.format("The order of entry exit can't be paired with the order of entry, current entry in context: <%s>, but expected: <%s>", curEntryNameInContext, this.resourceWrapper.getName());
                throw new ErrorEntryFreeException(errorMessage);
            }

            if (this.chain != null) {
                this.chain.exit(context, this.resourceWrapper, count, args);
            }

            this.callExitHandlersAndCleanUp(context);
            context.setCurEntry(this.parent);
            if (this.parent != null) {
                ((CtEntry)this.parent).child = null;
            }

            if (this.parent == null && ContextUtil.isDefaultContext(context)) {
                ContextUtil.exit();
            }

            this.clearEntryContext();
        }

    }
```

- 判断要退出的entry是否是当前context的当前entry；
- 如果要退出的entry不是当前context的当前entry，则不退出此entry，而是退出context的的当前entry及其所有父entry，并抛出异常；
- 如果要退出的entry是当前context的当前entry（这种是正常情况），先退出当前entry对应的责任链的所有slot。在这一步，StatisticSlot会更新node的success计数和RT计数；
- 将context的当前entry置为被退出的entry的父entry；
- 如果被退出entry的父entry为空，且context为默认context，自动退出默认context（清除ThreadLocal）。
- 清除被退出entry的context引用

## **总结**

通过阅读Sentinel的源码，可以很清晰的理解Sentinel的限流过程了，而对上面的源码阅读，总结如下：

- 三大组件Context、Entry、Node，是Sentinel的核心组件，各类信息及资源调用情况都由这三大类持有；
- 采用责任链模式完成Sentinel的信息统计、熔断、限流等操作；
- 责任链中NodeSelectSlot负责选择当前资源对应的Node，同时构建node调用树；
- 责任链中ClusterBuilderSlot负责构建当前Node对应的ClusterNode，用于聚合同一资源对应不同Context的Node；
- 责任链中的StatisticSlot用于统计当前资源的调用情况，更新Node与其对用的ClusterNode的各种统计数据；
- 责任链中的FlowSlot根据当前Node对应的ClusterNode（默认）的统计信息进行限流；
- 资源调用统计数据（例如PassQps）使用滑动时间窗口进行统计；
- 所有工作执行完毕后，执行退出流程，补充一些统计数据，清理Context。

# 参考文献

[https://github.com/alibaba/Sentinel/wiki](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Falibaba%2FSentinel%2Fwiki)