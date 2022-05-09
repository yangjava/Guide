# Skywalking

## 简介

```
SkyWalking: an APM(application performance monitor) system, especially designed for microservices, cloud native and container-based architectures.
```

**官网介绍**：分布式系统的应用程序性能监控工具，专为微服务、云原生和基于容器的 (Kubernetes) 架构而设计。

核心功能如下：

- 服务、服务实例、端点指标分析
- 根本原因分析。在运行时分析代码
- 服务拓扑图分析
- 服务、服务实例和端点依赖分析
- 缓慢的服务和端点检测
- 性能优化
- 分布式跟踪和上下文传播
- 数据库访问指标。检测慢速数据库访问语句（包括SQL语句）
- 消息队列性能和消耗延迟监控
- 警报
- 浏览器性能监控
- 基础设施（VM、网络、磁盘等）监控
- 跨指标、跟踪和日志的协作

![SkyWalking_Architecture](png\skywalking\SkyWalking简介.png)

SkyWalking 支持从多个来源和多种格式收集遥测（指标、跟踪和日志）数据，包括

1. Java、.NET Core、NodeJS、PHP 和 Python 自动仪器代理。
2. Go、C++ 和 Rust SDK。
3. LUA 代理，特别适用于 Nginx、OpenResty 和 Apache APISIX。
4. 浏览器代理。
5. 服务网格可观察性。控制平面和数据平面。
6. 度量系统，包括 Prometheus、OpenTelemetry、Spring Sleuth(Micrometer)、Zabbix。
7. 日志。
8. Zipkin v1/v2 跟踪。（无分析）

SkyWalking OAP 使用 STAM（流拓扑分析方法）来分析基于跟踪的代理场景中的拓扑，以获得更好的性能。阅读[STAM 的论文以](https://wu-sheng.github.io/STAM/)获取更多详细信息。

Skywalking是由国内开源爱好者吴晟（原OneAPM工程师，目前在华为）开源并提交到Apache孵化器的产品，它同时吸收了Zipkin/Pinpoint/CAT的设计思路，支持非侵入式埋点。是一款基于分布式跟踪的应用程序性能监控系统。另外社区还发展出了一个叫OpenTracing的组织，旨在推进调用链监控的一些规范和标准工作。

官方文档：https://skywalking.apache.org/docs/#SkyWalking

## 概念与架构

SkyWalking 是一个开源可观测平台，用于收集、分析、聚合和可视化来自服务和云原生基础设施的数据。SkyWalking 提供了一种简单的方法来保持分布式系统的清晰视图，甚至跨云。它是一种现代 APM，专为云原生、基于容器的分布式系统而设计。

### 为什么要使用 SkyWalking？

SkyWalking 为在许多不同场景中观察和监控分布式系统提供解决方案。首先，与传统方法一样，SkyWalking 为 Java、C#、Node.js、Go、PHP 和 Nginx LUA 等服务提供自动仪器代理。（呼吁 Python 和 C++ SDK 贡献）。在多语言、持续部署的环境中，云原生基础架构变得更加强大，但也更加复杂。SkyWalking 的服务网格接收器允许 SkyWalking 接收来自 Istio/Envoy 和 Linkerd 等服务网格框架的遥测数据，让用户了解整个分布式系统。

SkyWalking 为**服务**（Service）、**服务实例**（Service Instance）、**端点**（Endpoint ）提供可观察性能力。如今，Service、Instance 和 Endpoint 等术语随处可见，因此值得在 SkyWalking 的上下文中定义它们的具体含义：

- **服务**。表示为传入请求提供相同行为的一组/一组工作负载。您可以在使用仪器代理或 SDK 时定义服务名称。SkyWalking 还可以使用您在 Istio 等平台中定义的名称。
- **服务实例**。服务组中的每个单独的工作负载都称为一个实例。就像`pods`在 Kubernetes 中一样，它不需要是单个操作系统进程，但是，如果您使用仪器代理，则实例实际上是一个真正的操作系统进程。
- **端点**。用于传入请求的服务中的路径，例如 HTTP URI 路径或 gRPC 服务类 + 方法签名。

SkyWalking 允许用户了解Services 和Endpoints 的拓扑关系，查看每个Service/Service Instance/Endpoint 的指标，并设置报警规则。

此外，您还可以集成

1. 其他使用带有 Zipkin、Jaeger 和 OpenCensus 的 SkyWalking 本地代理和 SDK 的分布式跟踪。
2. 其他度量系统，如 Prometheus、Sleuth(Micrometer)、OpenTelemetry。

### 架构

SkyWalking 在逻辑上分为四个部分：Probes（探针）、Platform backend（平台后端）、Storage（存储） 和 UI。

![SkyWalking_Architecture](png\skywalking\SkyWalking架构图.png)

- **探测器（Probes）**收集数据并根据 SkyWalking 要求重新格式化（不同的探测器支持不同的来源）。
- **平台后端（Platform backend）**支持数据聚合、分析和流式处理，包括跟踪、度量和日志。
- **存储（Storage）**通过开放/可插入接口存储 SkyWalking 数据。您可以选择现有的实现，例如 ElasticSearch、H2、MySQL、TiDB、InfluxDB，也可以自己实现。欢迎为新的存储实现者打补丁！
- **UI**是一个高度可定制的基于 Web 的界面，允许 SkyWalking 最终用户可视化和管理 SkyWalking 数据。

### 设计目标

本文档概述了 SkyWalking 项目的核心设计目标。

- **保持可观察性**。无论目标系统采用何种部署方式，SkyWalking 都会为其提供集成解决方案以保持可观察性。基于此，SkyWalking 提供了多种运行时表单和探针。
- **拓扑、度量和跟踪一起**。了解分布式系统的第一步是拓扑图。它以易于阅读的布局可视化整个复杂系统。在拓扑下，OSS人员对服务、实例、端点、调用等指标有更高的要求。跟踪采用详细日志的形式，以了解这些指标。例如，当端点延迟变长时，您希望查看最慢的跟踪以找出原因。所以你可以看到，从大局到细节，都是需要的。SkyWalking 集成并提供了许多功能，使这成为可能且易于理解。
- **重量轻**。需要两个重量轻的部分。(1)在probe中，我们只依赖网络通信框架，更喜欢gRPC。这样，探针应该尽可能小，以避免库冲突和VM的有效负载，例如JVM中的permsize要求。(2) 作为一个可观察性平台，它是您项目环境中的二级和三级系统。所以我们正在使用我们自己的轻量级框架来构建后端核心。然后您无需部署和维护大数据技术平台。SkyWalking 在技术栈中应该很简单。
- **可插拔**。SkyWalking 核心团队提供了很多默认的实现，但肯定是不够的，也不适合所有场景。因此，我们提供了许多可插拔的功能。
- **便携性**。SkyWalking 可以在多种环境中运行，包括： (1) 使用传统的注册中心，如 eureka。(2) 使用包含服务发现的RPC框架，如Spring Cloud、Apache Dubbo。(3) 在现代基础设施中使用 Service Mesh。(4) 使用云服务。(5) 跨云部署。SkyWalking 在所有这些情况下都应该运行良好。
- **互操作性**。可观测性领域如此广阔，以至于 SkyWalking 几乎不可能支持所有系统，即使在其社区的支持下也是如此。目前，它支持与其他 OSS 系统的互操作性，尤其是探针，如 Zipkin、Jaeger、OpenTracing 和 OpenCensus。SkyWalking 能够接受和读取这些数据格式对最终用户来说非常重要，因为用户不需要切换他们的库。

## Probe(探针)

### 简介

在 SkyWalking 中，探针是指集成到目标系统中的代理或 SDK 库，负责收集遥测数据，包括跟踪和指标。根据目标系统技术堆栈，探针执行此类任务的方式有很大不同。但最终，他们都朝着同一个目标努力——收集和重新格式化数据，然后将它们发送到后端。

在高层次上，所有 SkyWalking 探测器都有三个典型类别。

- **基于语言的本地代理（Language based native agent）。**这些代理运行在目标服务用户空间中，例如用户代码的一部分。例如，SkyWalking Java 代理使用`-javaagent`命令行参数在运行时操作代码，这`manipulate`意味着更改和注入用户的代码。另一种代理使用目标库提供的某种钩子或拦截机制。如您所见，这些代理基于语言和库。
- **服务网格探针（Service Mesh probes）。**Service Mesh 探针从 Sidecar、Service Mesh 中的控制面板或代理收集数据。过去，proxy 只是作为整个集群的入口，但是有了 Service Mesh 和 sidecar，我们现在可以执行可观察性功能。
- **3rd 方仪器库（3rd-party instrument library）。**SkyWalking 接受许多广泛使用的仪器库数据格式。它分析数据，将其传输到 SkyWalking 的跟踪、度量或两者格式。此功能从接受 Zipkin 跨度数据开始。

您不需要同时使用**基于语言的本机代理**和**服务网格探测**，因为它们都用于收集指标数据。否则，您的系统将遭受两倍的有效负载，并且分析数将翻倍。

关于如何使用这些探针，有几种推荐的方法：

1. 仅使用**基于语言的本地代理**。
2. 仅使用**3rd 方仪器库**，例如 Zipkin 仪器生态系统。
3. 仅使用**服务网格探测**。
4. 在跟踪状态下使用带有**基于语言的本机代理**或**3rd 方仪器库的****Service Mesh 探针。**（高级用法）

**跟踪状态**中的含义是什么？

默认情况下，**基于语言的本机代理**和**3rd 方仪器库**都将分布式跟踪发送到后端，在后端对这些跟踪执行分析和聚合。**跟踪状态**意味着后端将这些跟踪视为日志。换句话说，后端保存它们，并在跟踪和指标之间建立链接，例如`which endpoint and service does the trace belong?`.

### Service Auto Instrument Agent

The service auto instrument agent是基于语言的本地代理的子集。这种代理基于一些特定于语言的特性，尤其是基于 VM 的语言的特性。

#### Auto Instrument 

许多用户在第一次听说“无需更改任何一行代码”时就了解了这些代理。SkyWalking 也曾在其自述页面中提到这一点。然而，这并不能反映全貌。对于最终用户来说，确实在大多数情况下他们不再需要修改他们的代码。但重要的是要了解代码实际上仍由代理修改，这通常称为“运行时代码操作”。其底层逻辑是自动仪表代理使用VM接口进行代码修改，动态添加仪表代码，比如通过 `javaagent premain`.

事实上，尽管 SkyWalking 团队已经提到大多数自动仪器代理都是基于 VM 的，但您可以在编译时而不是运行时构建此类工具。

#### 有什么限制？

自动检测非常有用，因为您可以在编译期间执行自动检测，而无需依赖 VM 功能。但它也有一些限制：

- **在许多情况下，进程内传播的可能性更高（Higher possibility of in-process propagation in many cases）**。许多高级语言，如 Java 和 .NET，用于构建业务系统。大多数业务逻辑代码在每个请求的同一线程中运行，这导致传播基于线程 ID，以便堆栈模块确保上下文是安全的。
- **仅适用于某些框架或库（Only works in certain frameworks or libraries）**。由于代理负责在运行时修改代码，因此代理插件开发人员已经知道这些代码。通常有一个此类探针支持的框架或库的列表。
- **并非总是支持跨线程操作(Cross-thread operations are not always supported)**。就像上面提到的进程内传播一样，大多数代码（尤其是业务代码）在每个请求的单个线程中运行。但在其他一些情况下，它们会跨不同的线程进行操作，例如将任务分配给其他线程、任务池或批处理。一些语言甚至可能提供协程或类似的组件，例如`Goroutine`，允许开发人员以低负载运行异步进程。在这种情况下，auto instrument将面临问题。

所以，auto instrument没有什么神秘的。简而言之，代理开发人员编写激活脚本以使instrument代码为您工作。就是这样！

### 服务网格探针(Service Mesh Probe)

Service Mesh 探针使用 Service Mesh 实现者（如 Istio）中提供的可扩展机制。

#### 什么是服务网格？

以下解释来自 Istio 的文档。

> 术语“服务网格”通常用于描述构成此类应用程序的微服务网络以及它们之间的交互。随着服务网格的规模和复杂性的增长，它可能变得更难理解和管理。它的要求可能包括发现、负载平衡、故障恢复、度量和监控，以及通常更复杂的操作要求，例如 A/B 测试、金丝雀发布、速率限制、访问控制和端到端身份验证。

#### 探针从哪里收集数据？

Istio 是典型的 Service Mesh 设计和实现者。它定义了广泛使用的**控制面板**和**数据面板。**这是 Istio 架构：

![Skywalking-ServiceMesh](png\skywalking\Skywalking-ServiceMesh.svg)

Service Mesh 探针可以选择从**数据面板**收集数据。在 Istio 中，这意味着从 Envoy sidecar（数据面板）收集遥测数据。探针从每个请求的客户端和服务器端收集两个遥测实体。

#### Service Mesh 如何让后端工作？

在这种探针中，您可以看到没有与它们相关的痕迹。那么 SkyWalking 平台是如何运作的呢？

Service Mesh 探针从每个请求中收集遥测数据，因此他们了解源、目标、端点、延迟和状态等信息。从这些信息中，后端可以通过将这些调用组合成行来告诉整个拓扑图，以及每个节点通过其传入请求的度量。后端通过解析跟踪数据请求相同的指标数据。简而言之： **Service Mesh 指标的工作方式与跟踪解析器生成的指标完全相同。**

## Backend(后端)

### 可观测性分析平台（Observability Analysis Platform）

SkyWalking 是一个可观察性分析平台，可为在棕色和绿色区域中运行的服务以及使用混合模型的服务提供完全可观察性。

#### 能力

SkyWalking 涵盖了可观察性的所有 3 个领域，包括**Tracing**、**Metrics**和**Logging**。

- **追踪（Tracing）**。SkyWalking 原生数据格式，包括 Zipkin v1 和 v2，以及 Jaeger。
- **指标（Metrics）**。SkyWalking 与 Istio、Envoy 和 Linkerd 等 Service Mesh 平台集成，将可观察性构建到数据面板或控制面板中。此外，SkyWalking 原生代理可以在指标模式下运行，大大提高了性能。
- **日志（Logging）**。包括从磁盘或通过网络收集的日志。原生代理可以自动将跟踪上下文与日志绑定，或者使用 SkyWalking 通过文本内容绑定跟踪和日志。

有 3 个强大的本地语言引擎旨在分析来自上述领域的可观察性数据。

1. 可观察性分析语言处理原生跟踪和服务网格数据。
2. Meter Analysis Language负责对原生meter数据进行metrics计算，采用Prometheus、OpenTelemetry等稳定且应用广泛的metrics系统。
3. 日志分析语言专注于日志内容，与仪表分析语言协同工作。

### 可观察性分析语言（Observability Analysis Language）

OAL（可观察性分析语言）用于以流模式分析传入数据。

OAL 专注于服务、服务实例和端点中的指标。因此，该语言易于学习和使用。

`oal-rt`从 6.3 开始，OAL 引擎作为(OAL Runtime)嵌入在 OAP 服务器运行时中。现在可以在`/config`文件夹中找到 OAL 脚本，用户只需更改并重新启动服务器即可运行它们。但是，OAL 脚本是一种编译语言，OAL Runtime 会动态生成 java 代码。

您可以在系统环境中打开 set`SW_OAL_ENGINE_DEBUG=Y`以查看生成了哪些类。

#### 语法

脚本应该命名`*.oal`

```
// Declare the metrics.
METRICS_NAME = from(SCOPE.(* | [FIELD][,FIELD ...]))
[.filter(FIELD OP [INT | STRING])]
.FUNCTION([PARAM][, PARAM ...])

// Disable hard code 
disable(METRICS_NAME);
```

#### 范围

主要的**SCOPE**是`All`, `Service`, `ServiceInstance`, `Endpoint`, `ServiceRelation`, `ServiceInstanceRelation`, 和`EndpointRelation`。还有一些次要作用域属于主要作用域。

请参阅[范围定义](https://skywalking.apache.org/docs/main/v8.6.0/en/concepts-and-designs/scope-definitions)，您可以在其中找到所有现有的范围和字段。

##### 筛选

使用过滤器通过使用字段名称和表达式来构建字段值的条件。

表达式支持通过`and`,`or`和进行链接`(...)`。OP 支持`==`, `!=`, `>`, `<`, `>=`, `<=`, `in [...]`, `like %...`, `like ...%`, `like %...%`, `contain`and `not contain`, 以及基于字段类型的类型检测。在不兼容的情况下，可能会触发编译或代码生成错误。

##### 聚合函数

默认功能由 SkyWalking OAP 核心提供，可以实现附加功能。

提供的功能

- `longAvg`. 每个范围实体的所有输入的平均值。输入字段必须很长。

> instance_jvm_memory_max = from(ServiceInstanceJVMMemory.max).longAvg();

在这种情况下，输入代表每个 ServiceInstanceJVMMemory 范围的请求，而 avg 是基于 field 的`max`。

- `doubleAvg`. 每个范围实体的所有输入的平均值。输入字段必须是双精度。

> instance_jvm_cpu = from(ServiceInstanceJVMCPU.usePercent).doubleAvg();

在这种情况下，输入表示每个 ServiceInstanceJVMCPU 范围的请求，而 avg 是基于 field 的`usePercent`。

- `percent`. 数字或比率表示为 100 的分数，其中输入与条件匹配。

> endpoint_percent = from(Endpoint.*).percent(status == true);

在这种情况下，所有输入代表每个端点的请求，条件是`endpoint.status == true`。

- `rate`. 比率表示为 100 的分数，其中输入与条件匹配。

> browser_app_error_rate = from(BrowserAppTraffic.*).rate(trafficCategory == BrowserAppTrafficCategory.FIRST_ERROR, trafficCategory == BrowserAppTrafficCategory.NORMAL);

在这种情况下，所有输入代表每个浏览器应用程序流量的请求，`numerator`条件是`trafficCategory == BrowserAppTrafficCategory.FIRST_ERROR`，`denominator`条件是`trafficCategory == BrowserAppTrafficCategory.NORMAL`。参数（1）是`numerator`条件。参数（2）是`denominator`条件。

- `count`. 每个范围实体的调用总和。

> service_calls_sum = from(Service.*).count();

在这种情况下，每个服务的调用次数。

- `histogram`. 请参阅[WIKI](https://en.wikipedia.org/wiki/Heat_map)中的热图。

> all_heatmap = from(All.latency).histogram(100, 20);

在这种情况下，所有传入请求的热力学热图。参数（1）是延迟计算的精度，比如上面的例子，113ms和193ms被认为在101-200ms组中是一样的。参数（2）是组数量。在上述情况下，21（参数值 + 1）组为 0-100ms、101-200ms、... 1901-2000ms、2000+ms

- `apdex`. 请参阅[WIKI 中的 Apdex](https://en.wikipedia.org/wiki/Apdex)。

> service_apdex = from(Service.latency).apdex(name, status);

在这种情况下，每个服务的 apdex 分数。参数（1）为服务名称，反映从config文件夹中的service-apdex-threshold.yml加载的Apdex阈值。参数（2）是这个请求的状态。状态（成功/失败）反映 Apdex 计算。

- `p99`, `p95`, `p90`, `p75`, `p50`. 请参阅[WIKI 中的百分位数](https://en.wikipedia.org/wiki/Percentile)。

> all_percentile = from(All.latency).percentile(10);

**percentile**是自 7.0.0 以来引入的第一个多值指标。作为具有多个值的指标，可以通过`getMultipleLinearIntValues`GraphQL 查询进行查询。在这种情况下，请参阅所有传入请求的`p99`、`p95`、`p90`、`p75`和`p50`。该参数精确到 p99 的延迟，例如在上述情况下，120ms 和 124ms 被认为产生相同的响应时间。在 7.0.0 之前，`p99`, `p95`, `p90`, `p75`, `p50`func(s) 用于分别计算指标。它们在 7.x 中仍受支持，但不再推荐使用，也不包含在当前的官方 OAL 脚本中。

> all_p99 = from(All.latency).p99(10);

在这种情况下，所有传入请求的 p99 值。该参数精确到 p99 的延迟，例如在上述情况下，120ms 和 124ms 被认为产生相同的响应时间。

##### 指标名称

存储实施者、警报和查询模块的指标名称。类型推断由核心支持。

##### GROUP

所有指标数据将按 Scope.ID 和最小级别 TimeBucket 分组。

- 在`Endpoint`范围内，Scope.ID 与Endpoint ID 相同（即基于服务及其端点的唯一ID）。

##### 禁用

`Disable`是 OAL 中的高级语句，仅在某些情况下使用。一些聚合和指标是通过核心硬代码定义的。示例包括`segment`和`top_n_database_statement`。该`disable`语句旨在使它们处于非活动状态。默认情况下，它们都没有被禁用。

**注意**，所有禁用语句都应该在`oal/disable.oal`脚本文件中。

#### 例子

```
// Calculate p99 of both Endpoint1 and Endpoint2
endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1", "Endpoint2")).summary(0.99)

// Calculate p99 of Endpoint name started with `serv`
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like "serv%").summary(0.99)

// Calculate the avg response time of each Endpoint
endpoint_avg = from(Endpoint.latency).avg()

// Calculate the p50, p75, p90, p95 and p99 of each Endpoint by 50 ms steps.
endpoint_percentile = from(Endpoint.latency).percentile(10)

// Calculate the percent of response status is true, for each service.
endpoint_success = from(Endpoint.*).filter(status == true).percent()

// Calculate the sum of response code in [404, 500, 503], for each service.
endpoint_abnormal = from(Endpoint.*).filter(responseCode in [404, 500, 503]).count()

// Calculate the sum of request type in [RequestType.RPC, RequestType.gRPC], for each service.
endpoint_rpc_calls_sum = from(Endpoint.*).filter(type in [RequestType.RPC, RequestType.gRPC]).count()

// Calculate the sum of endpoint name in ["/v1", "/v2"], for each service.
endpoint_url_sum = from(Endpoint.*).filter(name in ["/v1", "/v2"]).count()

// Calculate the sum of calls for each service.
endpoint_calls = from(Endpoint.*).count()

// Calculate the CPM with the GET method for each service.The value is made up with `tagKey:tagValue`.
service_cpm_http_get = from(Service.*).filter(tags contain "http.method:GET").cpm()

// Calculate the CPM with the HTTP method except for the GET method for each service.The value is made up with `tagKey:tagValue`.
service_cpm_http_other = from(Service.*).filter(tags not contain "http.method:GET").cpm()

disable(segment);
disable(endpoint_relation_server_side);
disable(top_n_database_statement);
```

### 指标分析语言（Meter Analysis Language）

指标系统提供一种称为 MAL（仪表分析语言）的功能分析语言，允许用户在 OAP 流系统中分析和汇总仪表数据。表达式的结果可以由代理分析器或 OC/Prometheus 分析器获取。

### 语言数据类型

在 MAL 中，表达式或子表达式可以计算为以下两种类型之一：

- **样本族**：一组样本（指标），包含一系列名称相同的指标。
- **标量**：一个简单的数值，支持整数/长整数和浮点/双精度。

## Sample family（样本族）

一组样本，作为 MAL 中的基本单元。例如：

```
instance_trace_count
```

上面的示例系列可能包含以下由外部模块提供的示例，例如代理分析器：

```
instance_trace_count{region="us-west",az="az-1"} 100
instance_trace_count{region="us-east",az="az-3"} 20
instance_trace_count{region="asia-north",az="az-1"} 33
```

### Tag filter（标签过滤）

MAL 支持四种类型的操作来过滤样本族中的样本：

- tagEqual：过滤标签与提供的字符串完全相同。
- tagNotEqual：过滤标签不等于提供的字符串。
- tagMatch：过滤与提供的字符串进行正则表达式匹配的标签。
- tagNotMatch：过滤与提供的字符串不匹配的标签。

例如，这会过滤 us-west 和 asia-north 区域以及 az-1 az 的所有 instance_trace_count 样本：

```
instance_trace_count.tagMatch("region", "us-west|asia-north").tagEqual("az", "az-1")
```

### Value filter（值过滤）

MAL 支持六种类型的操作来按值过滤样本族中的样本：

- valueEqual：过滤值与提供的值完全相等。
- valueNotEqual：过滤值等于提供的值。
- valueGreater：过滤大于提供值的值。
- valueGreaterEqual：过滤大于或等于提供值的值。
- valueLess：过滤小于提供值的值。
- valueLessEqual：过滤小于或等于提供值的值。

例如，这会过滤所有 instance_trace_count 样本的值 >= 33：

```
instance_trace_count.valueGreaterEqual(33)
```

### Tag manipulator（标签操纵器）

MAL 允许标签操作者更改（即添加/删除/更新）标签及其值。

### K8s

MAL 支持使用 K8s 的元数据来操作标签及其值。此功能需要授权 OAP Server 访问 K8s 的`API Server`.

##### retagByK8sMeta

`retagByK8sMeta(newLabelName, K8sRetagType, existingLabelName, namespaceLabelName)`. 根据现有标签的值向示例系列添加新标签。提供多种内部转换类型，包括

- K8sRetagType.Pod2Service

为样本添加一个标签，使用`service`作为键，`$serviceName.$namespace`作为值，并根据标签键的给定值，表示一个pod的名称。

例如：

```
container_cpu_usage_seconds_total{namespace=default, container=my-nginx, cpu=total, pod=my-nginx-5dc4865748-mbczh} 2
```

表达：

```
container_cpu_usage_seconds_total.retagByK8sMeta('service' , K8sRetagType.Pod2Service , 'pod' , 'namespace')
```

输出：

```
container_cpu_usage_seconds_total{namespace=default, container=my-nginx, cpu=total, pod=my-nginx-5dc4865748-mbczh, service='nginx-service.default'} 2
```

##### 二元运算符

MAL 中提供以下二元算术运算符：

- +（加法）
- \- （减法）
- *（乘法）
- / （分配）

二元运算符在标量/标量、sampleFamily/scalar 和 sampleFamily/sampleFamily 值对之间定义。

在两个标量之间：它们计算为另一个标量，该标量是运算符应用于两个标量操作数的结果：

```
1 + 2
```

在样本族和标量之间，运算符应用于样本族中每个样本的值。例如：

```
instance_trace_count + 2
```

或者

```
2 + instance_trace_count
```

结果是

```
instance_trace_count{region="us-west",az="az-1"} 102 // 100 + 2
instance_trace_count{region="us-east",az="az-3"} 22 // 20 + 2
instance_trace_count{region="asia-north",az="az-1"} 35 // 33 + 2
```

在两个样本族之间，对左侧样本族中的每个样本及其右侧样本族中的匹配样本应用二元运算符。将生成一个名称为空的新样本族。只有匹配的标签将被保留。右侧样本族中没有匹配样本的样本将不会出现在结果中。

另一个样本家庭`instance_trace_analysis_error_count`是

```
instance_trace_analysis_error_count{region="us-west",az="az-1"} 20
instance_trace_analysis_error_count{region="asia-north",az="az-1"} 11 
```

示例表达式：

```
instance_trace_analysis_error_count / instance_trace_count
```

这将返回一个包含跟踪分析错误率的结果样本系列。区域为 us-west 和 az az-3 的样本不匹配，也不会出现在结果中：

```
{region="us-west",az="az-1"} 0.8  // 20 / 100
{region="asia-north",az="az-1"} 0.3333  // 11 / 33
```

##### 聚合操作

样本族支持以下聚合操作，可用于聚合单个样本族的样本，从而生成具有较少样本（有时只有一个样本）且具有聚合值的新样本族：

- sum（计算维度的总和）
- min（选择最小尺寸）
- 最大（选择最大尺寸）
- avg（计算维度的平均值）

这些操作可用于聚合整体标签尺寸或通过输入`by`参数保留不同的尺寸。

```
<aggr-op>(by: <tag1, tag2, ...>)
```

示例表达式：

```
instance_trace_count.sum(by: ['az'])
```

将输出以下结果：

```
instance_trace_count{az="az-1"} 133 // 100 + 33
instance_trace_count{az="az-3"} 20
```

### 功能

`Duraton`是时间范围的文本表示。接受的格式基于 ISO-8601 持续时间格式 {@code PnDTnHnMn.nS}，其中一天被视为 24 小时。

例子：

- “PT20.345S”——解析为“20.345 秒”
- “PT15M” – 解析为“15 分钟”（其中一分钟为 60 秒）
- “PT10H”——解析为“10 小时”（其中 1 小时为 3600 秒）
- “P2D”——解析为“2 天”（一天是 24 小时或 86400 秒）
- “P2DT3H4M”——解析为“2天3小时4分钟”
- “P-6H3M” – 解析为“-6 小时 +3 分钟”
- “-P6H3M” – 解析为“-6 小时 -3 分钟”
- “-P-6H+3M” – 解析为“+6 小时 -3 分钟”

#### increase（增加）

`increase(Duration)`：计算时间范围内的增量。

#### rate（平均速率）

`rate(Duration)`：计算时间范围内每秒的平均增长率。

#### irate（瞬时速率）

`irate()`：计算时间范围内每秒的瞬时增长率。

#### tag（标签）

`tag({allTags -> })`：更新样本的标签。用户可以添加、删除、重命名和更新标签。

#### histogram（直方图）

`histogram(le: '<the tag name of le>')`：将基于较少的直方图桶转换为计量系统直方图桶。 `le`参数表示存储桶的标签名称。

#### histogram_percentile

`histogram_percentile([<p scalar>])`. 表示计量系统从桶中计算 p 百分位数 (0 ≤ p ≤ 100)。

#### time（时间）

`time()`：返回自 1970 年 1 月 1 日 UTC 以来的秒数。

#### 下采样操作

MAL 应该指导计量系统如何对指标进行下采样。它不仅指聚合原始样本到 `minute`级别，还表示来自`minute`更高级别的数据，例如`hour`和`day`。

在 MAL中调用下采样函数`downsampling`，它接受以下类型：

- AVG  
- SUM 
- LATEST
- MIN (TODO)
- MAX (TODO)
- MEAN (TODO)
- COUNT (TODO)

默认类型是`AVG`.

如果用户想从以下位置获取最新时间`last_server_state_sync_time_in_seconds`：

```
last_server_state_sync_time_in_seconds.tagEqual('production', 'catalog').downsampling(LATEST)
```

#### 度量级函数

Metric 分为三个级别：服务、实例和端点。他们从度量标签中提取级别相关标签，然后通知仪表系统该度量所属的级别。

- `servcie([svc_label1, svc_label2...])`从数组参数中提取服务级别标签。
- `instance([svc_label1, svc_label2...], [ins_label1, ins_label2...])`从第一个数组参数中提取服务级别标签，从第二个数组参数中提取实例级别标签。
- `endpoint([svc_label1, svc_label2...], [ep_label1, ep_label2...])`从第一个数组参数中提取服务级别标签，从第二个数组参数中提取端点级别标签。







### 基础概念

span 、trace segment 、contextCarrier 、contextSnapshot

#### span 

Span 是分布式追踪系统中一个非常重要的概念，可以理解为一次方法调用、一个程序块的调用、一次 RPC 调用或者数据库访问。

依据是跨线程还是跨进程的链路，将 Span粗略分为两类：LocalSpan 和RemoteSpan。LocalSpan 代表一次普通的 Java 方法调用，与跨进程无关，多用于当前进程中关键逻辑代码块的记录，或在跨线程后记录异步线程执行的链路信息。RemoteSpan 可细分为 EntrySpan 和 ExitSpan：

-  EntrySpan 代表一个应用服务的提供端或服务端的人口端点，如 Web 容器的服务端的入口、RPC 服务器的消费者、消息队列的消费者;

- ExitSpan （SkyWalking 的早期版本中称其为 LeafSpan)，代表一个应用服务的客户端或消息队列的生产者，如 Redis 客户端的一次 Redis 调用、MySQL 客户端的一次数据库查询、RPC 组件的一次请求、消息队列生产者的生产消息。

#### Trace Segment

Trace Segment 是 SkyWalking 中特有的概念，通常指在支持多线程的语言中，一个线程中归属于同一个操作的所有 Span 的聚合。这些 Span 具有相同的唯一标识SegmentID。Trace Segment对应的实体类位于org.apache.skywalking.apm.agent,core.context.trace.TraceSegment，其中重要的属性如下。

- TraceSegmentld：此 Trace Segment 操作的唯一标识。使用雪花算法生成，保证全局唯一。
- Refs：此 Trace Segment 的上游引用。对于大多数上游是 RPC 调用的情况，Refs只有一个元素，但如果是消息队列或批处理框架，上游可能会是多个应用服务，所以就会存在多个元素。
- Spans：用于存储，从属于此 Trace Segment 的 Span 的集合。
- RelatedGlobalTraces ：此 Trace Segment 的 Trace Id。大多数时候它只包含一个元素，但如果是消息队列或批处理框架，上游是多个应用服务，会存在多个元素。
- lgnore ：是否忽略。如果Ignore 为true，则此Trace Segment 不会上传到SkyWalking 后端。
- IsSizeLimited：从属于此 Trace Segment 的 Span 数量限制，初始化大小可以通过config.agent.span_limit_per_segment 参数来配置，默认长度为 300。若超过配置值，在创建新的 Span 的时候，会变成 NoopSpan。NoopSpan 表示没有任何实际操作的 Span 实现，用于保持内存和 GC 成本尽可能低。
- CreateTime：此 Trace Segment 的创建时间。

#### ContextCarrier

分布式追踪要解决的一个重要问题是跨进程调用链的连接，ContextCarrier 的就是为了解决这个问题。如客户端 A、服务端 B 两个应用服务，当发生一次 A 调用 B 的时候,

跨进程传播的步骤如下。

- 1）客户端 A 创建空的 ContextCarrier。
- 2）通过 ContextManager#createExitSpan 方法创建一个ExitSpan，或者使用ContextManager#inject 在过程中传入并初始化 ContextCarrier。
- 3）使用 ContextCarrier.items()将 ContextCarrier 所有元素放到调用过程中的请求信息中，如 HTTP HEAD、Dubbo RPC 框架的 attachments、消息队列 Kafka 消息的 header 中。
- 4）ContextCarrier 随请求传输到服务端。
- 5）服务端 B 接收具有 ContextCarrier 的请求，并提取 ContextCarrier 相关的所有信息。
- 6）通过 ContextManager#createEntrySpan 方法创建 EntrySpan，或者使用ContextManager#extract 建立分布式调用关联，即绑定服务端 B 和客户端A。

#### ContextSnapshot

除了跨进程，跨线程也是需要支持的，例如异步线程（内存中的消息队列）在Java中很常见。跨线程和跨进程十分相似，都需要传播上下文，唯一的区别是，跨线程不需要序列化。以下是跨线程传播的步骤。

- 1）使用 ContextManager#capture 方法获取 ContextSuapshot 对象。
- 2）让子线程以任何方式，通过方法参数或由现有参数携带来访问 ContextSnapshot。
- 3）在子线程中使用 ContextManager#continued。

## UI

仪表盘：查看被监控服务的运行状态

拓扑图：以拓扑图的方式展现服务直接的关系，并以此为入口查看相关信息

追踪：以接口列表的方式展现，追踪接口内部调用过程

性能剖析：单独端点进行采样分析，并可查看堆栈信息

告警：触发告警的告警列表，包括实例，请求超时等。

自动刷新：刷新当前数据内容（我这好像没有自动刷新）

Global、Server、Instance、Endpoint不同展示面板，可以调整内部内容

- Services load：服务每分钟请求数
- Slow Services：慢响应服务，单位ms
- Un-Health services(Apdex):Apdex性能指标，1为满分。
- Global Response Latency：百分比响应延时，不同百分比的延时时间，单位ms
- Global Heatmap：服务响应时间热力分布图，根据时间段内不同响应时间的数量显示颜色深度

#### Service服务维度

Service Apdex（数字）:当前服务的评分 

Service Apdex（折线图）：不同时间的Apdex评分

Successful Rate（数字）：请求成功率

Successful Rate（折线图）：不同时间的请求成功率

Servce Load（数字）：每分钟请求数

Servce Load（折线图）：不同时间的每分钟请求数

Service Avg Response Times：平均响应延时，单位ms

Global Response Time Percentile：百分比响应延时

#### Instance实例维度

Servce Instances Load：每个服务实例的每分钟请求数

Show Service Instance：每个服务实例的最大延时

Service Instance Successful Rate：每个服务实例的请求成功率

Service Instance Load：当前实例的每分钟请求数

Service Instance Successful Rate：当前实例的请求成功率

Service Instance Latency：当前实例的响应延时

JVM CPU:jvm占用CPU的百分比

JVM Memory：JVM内存占用大小，单位m

JVM GC Time：JVM垃圾回收时间，包含YGC和OGC

JVM GC Count：JVM垃圾回收次数，包含YGC和OGC

CLR XX：类似JVM虚拟机，这里用不上就不做解释了

#### Endpoint端点（API）维度

 Endpoint Load in Current Service：每个端点的每分钟请求数

Slow Endpoints in Current Service：每个端点的最慢请求时间，单位ms

Successful Rate in Current Service：每个端点的请求成功率

Endpoint Load：当前端点每个时间段的请求数据

Endpoint Avg Response Time：当前端点每个时间段的请求行响应时间

Endpoint Response Time Percentile：当前端点每个时间段的响应时间占比

Endpoint Successful Rate：当前端点每个时间段的请求成功率

### DataSource展示栏

当前数据库：选择查看数据库指标

Database Avg Response Time：当前数据库事件平均响应时间，单位ms

Database Access Successful Rate：当前数据库访问成功率

Database Traffic：CPM，当前数据库每分钟请求数

Database Access Latency Percentile：数据库不同比例的响应时间，单位ms

Slow Statements：前N个慢查询，单位ms

All Database Loads：所有数据库中CPM排名

Un-Health Databases：所有数据库健康排名，请求成功率排名

## 拓扑图

1：选择不同的服务关联拓扑

2：查看单个服务相关内容

3：服务间连接情况

4：分组展示服务拓扑

：服务告警信息

：服务端点追踪信息

：服务实例性能信息

：api信息面板

## 追踪

-  左侧：api接口列表，红色-异常请求，蓝色-正常请求
-  右侧：api追踪列表，api请求连接各端点的先后顺序和时间

## 性能剖析

新建任务：新建需要分析的端点

左侧列表：任务及对应的采样请求

 右侧：端点链路及每个端点的堆栈信息

新建任务

服务：需要分析的服务

端点：链路监控中端点的名称，可以再链路追踪中查看端点名称

监控时间：采集数据的开始时间

监控持续时间：监控采集多长时间

起始监控时间：多少秒后进行采集

监控间隔：多少秒采集一次

最大采集数：最大采集多少样本



## 下载与安装

SkyWalking有两种版本，ES版本和非ES版。如果我们决定采用ElasticSearch作为存储，那么就下载es版本

agent目录将来要拷贝到各服务所在机器上用作探针

bin目录是服务启动脚本

config目录是配置文件

oap-libs目录是oap服务运行所需的jar包

webapp目录是web服务运行所需的jar包

接下来，要选择存储了，支持的存储有：

- H2
- ElasticSearch 6, 7
- MySQL
- TiDB
- **InfluxDB**

作为监控系统，首先排除H2和MySQL，这里推荐InfluxDB，它本身就是时序数据库，非常适合这种场景

## SpringBoot 实例部署

原来的启动方式为：

```bash
java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar
```

那么使用 skywalking agent，启动命令为：

```bash
java -javaagent:/opt/apache-skywalking-apm-bin/agent/skywalking-agent.jar -Dskywalking.agent.service_name=xxxtest -Dskywalking.collector.backend_service=127.0.0.1:11800 -jar /opt/spring-boot-demo-0.0.1-SNAPSHOT.jar
```

说明：

- `-javaagent` 指定agent包位置。这里将apache-skywalking-apm-6.6.0.tar.gz解压到/opt目录了，因此路径为：/opt/apache-skywalking-apm-bin/agent/skywalking-agent.jar
- `-Dskywalking.agent.service_name` 指定服务名
- `-Dskywalking.collector.backend_service` 指定skywalking oap地址，由于在本机，地址为：127.0.0.1:11800
- `-jar` 指定jar包的路径，这里直接放到/opt/目录了。

访问ui

http://x.x.x.x:8088/

# 告警

Apache SkyWalking告警是由一组规则驱动，这些规则定义在config/alarm-settings.yml文件中。

告警规则的定义分为三部分。

1. **告警规则**：定义了触发告警所考虑的条件。
2. **webhook**：当告警触发时，被调用的服务端点列表。
3. **gRPCHook**：当告警触发时，被调用的远程gRPC方法的主机和端口。
4. **Slack Chat Hook**：当告警触发时，被调用的Slack Chat接口。
5. **微信 Hook**：当告警触发时，被调用的微信接口。
6. **钉钉 Hook**：当告警触发时，被调用的钉钉接口。

#### 告警规则

告警规则有两种类型，单独规则（Individual Rules）和复合规则（Composite Rules），复合规则是单独规则的组合。

##### 单独规则（Individual Rules）

单独规则主要有以下几点：

- **规则名称**：在告警信息中显示的唯一名称，必须以_rule结尾。
- **metrics-name**：度量名称，也是OAL脚本中的度量名。默认配置中可以用于告警的度量有：**服务**，**实例**，**端点**，**服务关系**，**实例关系**，**端点关系**。它只支持long,double和int类型。
- **include-names**：包含在此规则之内的实体名称列表。
- **exclude-names**：排除在此规则以外的实体名称列表。
- **include-names-regex**：提供一个正则表达式来包含实体名称。如果同时设置包含名称列表和包含名称的正则表达式，则两个规则都将生效。
- **exclude-names-regex**：提供一个正则表达式来排除实体名称。如果同时设置排除名称列表和排除名称的正则表达式，则两个规则都将生效。
- **include-labels**：包含在此规则之内的标签。
- **exclude-labels**：排除在此规则以外的标签。
- **include-labels-regex**：提供一个正则表达式来包含标签。如果同时设置包含标签列表和包含标签的正则表达式，则两个规则都将生效。
- **exclude-labels-regex**：提供一个正则表达式来排除标签。如果同时设置排除标签列表和排除标签的正则表达式，则两个规则都将生效。

标签的设置必须把数据存储在meter-system中，例如：Prometheus, Micrometer。以上四个标签设置必须实现LabeledValueHolder接口。

- **threshold**：阈值。

对于多个值指标，例如**percentile**，阈值是一个数组。像value1 value2 value3 value4 value5这样描述。
每个值可以作为度量中每个值的阈值。如果不想通过此值或某些值触发警报，则将值设置为 -。
例如在**percentile**中，value1是P50的阈值，value2是P75的阈值，那么-，-，value3, value4, value5的意思是，没有阈值的P50和P75的**percentile**告警规则。

- **op**：操作符，支持>, >=, <, <=, =。
- **period**：多久告警规则需要被检查一下。这是一个时间窗口，与后端部署环境时间相匹配。
- **count**：在一个周期窗口中，如果按**op**计算超过阈值的次数达到**count**，则发送告警。
- **only-as-condition**：true或者false，指定规则是否可以发送告警，或者仅作为复合规则的条件。
- **silence-period**：在时间N中触发报警后，在**N -> N + silence-period**这段时间内不告警。 默认情况下，它和**period**一样，这意味着相同的告警（同一个度量名称拥有相同的Id）在同一个周期内只会触发一次。
- **message**：该规则触发时，发送的通知消息。

举个例子：

```plain
rules:
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 10
    message: 服务【{name}】的平均响应时间在最近10分钟内有2分钟超过1秒
  service_instance_resp_time_rule:
    metrics-name: service_instance_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 10
    message: 实例【{name}】的平均响应时间在最近10分钟内有2分钟超过1秒
  endpoint_resp_time_rule:
    metrics-name: endpoint_avg
    threshold: 1000
    op: ">"
    period: 10
    count: 2
    message: 端点【{name}】的平均响应时间在最近10分钟内有2分钟超过1秒
```

##### 复合规则（Composite Rules）

复合规则仅适用于针对相同实体级别的告警规则，例如都是服务级别的告警规则：service_percent_rule && service_resp_time_percentile_rule。
**不可以**编写不同实体级别的告警规则，例如服务级别的一个告警规则和端点级别的一个规则：service_percent_rule && endpoint_percent_rule。

复合规则主要有以下几点：

- **规则名称**：在告警信息中显示的唯一名称，必须以_rule结尾。
- **expression**：指定如何组成规则，支持&&, ||, ()操作符。
- **message**：该规则触发时，发送的通知消息。

举个例子：

```plain
rules:
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 10
    message: 服务【{name}】的平均响应时间在最近10分钟内有2分钟超过1秒
  service_sla_rule:
    metrics-name: service_sla
    op: "<"
    threshold: 8000
    period: 10
    count: 2
    silence-period: 10
    message: 服务【{name}】的成功率在最近10分钟内有2分钟低于80％
composite-rules:
  comp_rule:
    expression: service_resp_time_rule && service_sla_rule
    message: 服务【{name}】在最近10分钟内有2分钟超过1秒平均响应时间超过1秒并且成功率低于80％
```

#### Webhook

Webhook 要求一个点对点的 Web 容器。告警的消息会通过 HTTP 请求进行发送，请求方法为 POST，Content-Type 为 application/json，JSON 格式包含以下信息：

- **scopeId**：目标 Scope 的 ID。
- **name**：目标 Scope 的实体名称。
- **id0**：Scope 实体的 ID。
- **id1**：未使用。
- **ruleName**：您在 alarm-settings.yml 中配置的规则名。
- **alarmMessage**. 告警消息内容。
- **startTime**. 告警时间戳，当前时间与 UTC 1970/1/1 相差的毫秒数。

举个例子：

```plain
[{
	"scopeId": 1, 
	"scope": "SERVICE",
	"name": "one-more-service", 
	"id0": "b3JkZXItY2VudGVyLXNlYXJjaC1hcGk=.1",  
	"id1": "",  
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "服务【one-more-service】的平均响应时间在最近10分钟内有2分钟超过1秒",
	"startTime": 1617670815000
}, {
	"scopeId": 2,
	"scope": "SERVICE_INSTANCE",
	"name": "e4b31262acaa47ef92a22b6a2b8a7cb1@192.168.30.11 of one-more-service",
	"id0": "dWF0LWxib2Mtc2VydmljZQ==.1_ZTRiMzEyNjJhY2FhNDdlZjkyYTIyYjZhMmI4YTdjYjFAMTcyLjI0LjMwLjEzOA==",
	"id1": "",
    "ruleName": "instance_jvm_young_gc_count_rule",
	"alarmMessage": "实例【e4b31262acaa47ef92a22b6a2b8a7cb1@192.168.30.11 of one-more-service】的YoungGC次数在最近10分钟内有2分钟超过10次",
	"startTime": 1617670815000
}, {
	"scopeId": 3,
	"scope": "ENDPOINT",
	"name": "/one/more/endpoint in one-more-service",
	"id0": "b25lcGllY2UtYXBp.1_L3RlYWNoZXIvc3R1ZGVudC92aXBsZXNzb25z",
	"id1": "",
    "ruleName": "endpoint_resp_time_rule",
	"alarmMessage": "端点【/one/more/endpoint in one-more-service】的平均响应时间在最近10分钟内有2分钟超过1秒",
	"startTime": 1617670815000
}]
```

#### gRPCHook

告警消息将使用 Protobuf 类型通过gRPC远程方法发送。消息格式的关键信息定义如下：

```plain
syntax = "proto3";

option java_multiple_files = true;
option java_package = "org.apache.skywalking.oap.server.core.alarm.grpc";

service AlarmService {
    rpc doAlarm (stream AlarmMessage) returns (Response) {
    }
}

message AlarmMessage {
    int64 scopeId = 1;
    string scope = 2;
    string name = 3;
    string id0 = 4;
    string id1 = 5;
    string ruleName = 6;
    string alarmMessage = 7;
    int64 startTime = 8;
}

message Response {
}
```

#### Slack Chat Hook

您需要遵循[传入Webhooks入门指南](https://api.slack.com/messaging/webhooks)并创建新的Webhooks。

如果您按以下方式配置了Slack Incoming Webhooks，则告警消息将按 Content-Type 为 application/json 通过HTTP的 POST 方式发送。

举个例子：

```plain
slackHooks:
  textTemplate: |-
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": ":alarm_clock: *Apache Skywalking Alarm* \n **%s**."
      }
    }
  webhooks:
    - https://hooks.slack.com/services/x/y/z
```

#### 微信Hook

只有微信的企业版才支持 Webhooks ，如何使用微信的 Webhooks 可参见[如何配置群机器人](https://work.weixin.qq.com/help?doc_id=13376)。

如果您按以下方式配置了微信的 Webhooks ，则告警消息将按 Content-Type 为 application/json 通过HTTP的 POST 方式发送。

举个例子：

```plain
wechatHooks:
  textTemplate: |-
    {
      "msgtype": "text",
      "text": {
        "content": "Apache SkyWalking 告警: \n %s."
      }
    }
  webhooks:
    - https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=dummy_key
```

#### 钉钉 Hook

您需要遵循[自定义机器人开放](https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq/uKPlK)并创建新的Webhooks。为了安全起见，您可以为Webhook网址配置可选的密钥。

如果您按以下方式配置了钉钉的 Webhooks ，则告警消息将按 Content-Type 为 application/json 通过HTTP的 POST 方式发送。

举个例子：

```plain
dingtalkHooks:
  textTemplate: |-
    {
      "msgtype": "text",
      "text": {
        "content": "Apache SkyWalking 告警: \n %s."
      }
    }
  webhooks:
    - url: https://oapi.dingtalk.com/robot/send?access_token=dummy_token
      secret: dummysecret
```

## 配置覆盖

### 概述

- 我们每次部署应用都需要到agent中修改服务名，如果部署多个应用，那么只能复制agent以便产生隔离，但是这样非常麻烦。我们可以用Skywalking提供的配置覆盖功能通过启动命令动态的指定服务名，这样agent就只需要部署一份即可。

- Skywalking支持以下几种配置：

- - 系统配置。

- - 探针配置。

- - 系统环境变量。

- - 配置文件中的值（上面的案例中我们就是这样使用)。



### 系统配置（System Properties）

- 使用`skywalking.`+配置文件中的配置名作为系统配置项来进行覆盖。

- 例如：通过命令行启动的时候加上如下的参数可以进行`agent.service_name`的覆盖



```shell
-Dskywalking.agent.service_name=skywalking_mysql
```



###  探针配置（Agent Options）

- 可以指定探针的时候加上参数，如果配置中包含分隔符（,或=），就必须使用引号（‘’）包裹起来。

```shell
-javaagent:/usr/local/skywalking/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar=[option]=[value],[option]=[value]
```

- 例如：

```shell
-javaagent:/usr/local/skywalking/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar=agent.service_name=skywalking_mysql
```

###  系统环境变量

- 案例：

- 由于agent.service_name配置项如下所示

```properties
agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}
```

- 可以在环境变量中设置SW_AGENT_NAME的值来指定服务名。

###  覆盖的优先级

- 探针配置>系统配置>系统环境变量>配置文件中的值。

- 简而言之，推荐使用`探针方式`或`系统配置`的方式。

## 获取追踪的ID

### 概述

- Skywalking提供了Trace工具包，用于在追踪链路的时候进行信息的打印或者获取对应的追踪ID。

### 应用示例

- 在pom.xml中增加Trace工具包的坐标：

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>8.3.0</version>
</dependency>
```

- 修改Controller.java

```java
package com.example.springboot2.web;

import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;
import org.apache.skywalking.apm.toolkit.trace.TraceContext;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class HelloController {

    @RequestMapping(value = "/hello")
    public String hello() {
        return "你好啊";
    }

    /**
     * TraceContext.traceId() 可以打印出当前追踪的ID，方便在Rocketbot中搜索
     * ActiveSpan提供了三个方法进行信息打印：
     *   error：会将本次调用转为失败状态，同时可以打印对应的堆栈信息和错误提示
     *   info：打印info级别额信息
     *   debug：打印debug级别的信息
     *
     * @return
     */
    @RequestMapping(value = "/exception")
    public String exception() {
        ActiveSpan.info("打印info信息");
        ActiveSpan.debug("打印debug信息");
        //使得当前的链路报错，并且提示报错信息
        try {
            int i = 10 / 0;
        } catch (Exception e) {
            ActiveSpan.error(new RuntimeException("报错了"));
            //返回trace id
            return TraceContext.traceId();
        }

        return "你怎么可以这个样子";
    }
}
```

- 将项目打包，部署，并使用如下的命令启动：

```shell
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dserver.port=8082   -Dskywalking.agent.service_name=skywalking_mysql -jar springboot2-1.0.jar &
```

- 访问接口，获取追踪的ID：

- 可以搜索到对应的追踪记录，但是显示调用是失败的，这是因为使用了ActiveSpan.error方法，点开追踪的详细信息。

## 过滤指定的端点

- 在开发的过程中，有一些端点（接口）并不需要去进行监控，比如Swagger相关的端点。这个时候我们使用Skywalking提供的过滤插件来进行过滤。

### 应用示例

- 在上面的SpringBoot项目中增加如下的控制器：

```java
package com.example.springboot2.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class FilterController {

    /**
     * 此接口可以被追踪
     *
     * @return
     */
    @GetMapping(value = "/include")
    public String include() {
        return "include";
    }

    /**
     * 此接口不可以被追踪
     *
     * @return
     */
    @GetMapping(value = "/exclude")
    public String exclude() {
        return "exclude";
    }
}
```

- 将项目打包并上传到/usr/local/skywalking中。

- 将agent中的agent/optional-plugins/apm-trace-ignore-plugin-8.3.0.jar插件复制到plugins目录中。

```shell
cd /usr/local/skywalking/apache-skywalking-apm-bin-es7/agent
```

```shell
cp optional-plugins/apm-trace-ignore-plugin-8.3.0.jar plugins/
```

- 启动SpringBoot应用，并添加过滤参数。

```shell
java -javaagent:/usr/local/skywalking/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dserver.port=8082 -Dskywalking.agent.service_name=skywalking_mysql -Dskywalking.trace.ignore_path=/exclude -jar springboot2-1.0.jar &
```

这里添加`-Dskywalking.trace.ignore_path`参数来标识需要过滤那些请求，支持`Ant Path`表达式：

- `/path/*`，`/path/**`，`/path/?`

- `?` 匹配任何单字符

- `*`匹配0或者任意数量的字符

- `**`匹配0或者更多的目录

- 调用接口，接口的地址为：

- - http://192.168.159.103:8082/include。

- - http://192.168.159.103:8082/exclude。

- 会发现exclude接口已经被过滤了，只有include接口能被看到。

## 告警功能

## 源码

SkyWalking 源码的整体结构如下图所示：

1、apm-application-toolkit 模块：SkyWalking 提供给用户调用的工具箱。

该模块提供了对 log4j、log4j2、logback 等常见日志框架的接入接口，提供了 @Trace 注解等。
apm-application-toolkit 模块类似于暴露 API 定义，对应的处理逻辑在 apm-sniffer/apm-toolkit-activation 模块中实现。

2、apm-commons 模块：SkyWalking 的公共组件和工具类。

其中包含两个子模块，apm-datacarrier 模块提供了一个生产者-消费者模式的缓存组件（DataCarrier），无论是在 Agent 端还是 OAP 端都依赖该组件。
apm-util 模块则提供了一些常用的工具类，例如，字符串处理工具类（StringUtil）、占位符处理的工具类（PropertyPlaceholderHelper、PlaceholderConfigurerSupport）等等。


3、apm-protocol 模块：该模块中只有一个 apm-network 模块，我们需要关注的是其中定义的 .proto 文件，定义 Agent 与后端 OAP 使用 gRPC 交互时的协议。

4、apm-sniffer 模块：apm-sniffer 模块中有 4 个子模块：

apm-agent 模块：其中包含了刚才使用的 SkyWalkingAgent 这个类，是整个 Agent 的入口。

apm-agent-core 模块：SkyWalking Agent 的核心实现都在该模块中。

apm-sdk-plugin 模块：SkyWalking Agent 使用了微内核+插件的架构，该模块下包含了 SkyWalking Agent 的全部插件，

apm-toolkit-activation 模块：apm-application-toolkit 模块的具体实现。

5、apm-webapp 模块：SkyWalking Rocketbot 对应的后端。

6、oap-server 模块：SkyWalking OAP 的全部实现都在 oap-server 模块，其中包含了多个子模块，
exporter 模块：负责导出数据。

server-alarm-plugin 模块：负责实现 SkyWalking 的告警功能。

server-cluster-pulgin 模块：负责 OAP 的集群信息管理，其中提供了接入多种第三方组件的相关插件，

server-configuration 模块：负责管理 OAP 的配置信息，也提供了接入多种配置管理组件的相关插件，

server-core模块：SkyWalking OAP 的核心实现都在该模块中。

server-library 模块：OAP 以及 OAP 各个插件依赖的公共模块，其中提供了双队列 Buffer、请求远端的 Client 等工具类，这些模块都是对立于 SkyWalking OAP 体系之外的类库，我们可以直接拿走使用。

server-query-plugin 模块：SkyWalking Rocketbot 发送的请求首先由该模块接收处理，目前该模块只支持 GraphQL 查询。

server-receiver-plugin 模块：SkyWalking Agent 发送来的 Metrics、Trace 以及 Register 等写入请求都是首先由该模块接收处理的，不仅如此，该模块还提供了多种接收其他格式写入请求的插件，

server-starter 模块：OAP 服务启动的入口。

server-storage-plugin 模块：OAP 服务底层可以使用多种存储来保存 Metrics 数据以及Trace 数据，该模块中包含了接入相关存储的插件，

7、skywalking-ui 目录：SkyWalking Rocketbot 的前端。

# 流程

Skywalking主要由Agent、OAP、Storage、UI四大模块组成（如下图）：Agent和业务程序运行在一起，采集链路及其它数据，通过gRPC发送给OAP（部分Agent采用http+json的方式）；OAP还原链路（图中的Tracing），并分析产生一些指标（图中的Metric），最终存储到Storage中。

## Java Agent的原理

### 概述

- 我们知道，要使用Skywalking去监控服务，需要在其VM参数中添加“`-javaagent:/usr/local/skywalking/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar`”，这里就使用到了java agent技术。

### java agent是什么？

- java agent是java命令的一个参数，参数`javaagent`可以用于指定一个jar包。

- - 这个jar包的MANIFEST.MF文件必须指定Premain-Class项。

- - Premain-Class指定的那个类必须实现premain()方法。

- 当Java虚拟机启动时，在执行main函数之前，JVM会运行`-javaagent`所指定的jar包内Premain-Class这个类的premain方法。

### 如何使用java agent？

- 定义一个MANIFEST.MF文件，必须包含Premain-Class选项，通常也会加入Can-Redefine-Classes和Can-Retransform-Classes选项。

- 创建一个Premain-Class指定的类，类中包含premain方法，方法逻辑由用户自己确定。

- 将premain类和MANIFEST.MF文件打成jar包。

- 使用参数`-javaagent:jar包路径`启动要代理的方法。

### 搭建java agent工程

- 使用Maven搭建java_agent总工程，其中java_agent_demo子工程是真正的逻辑实现，而java_agent_user子工程是测试。

- 在java_agent_demo的工程中引入maven-assembly-plugin插件，用于将java_agent_demo工程打成符合java agent标准的jar包：

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <appendAssemblyId>false</appendAssemblyId>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <archive>
                    <!--自动添加META-INF/MANIFEST.MF -->
                    <manifest>
                        <addClasspath>true</addClasspath>
                    </manifest>
                    <manifestEntries>
                        <Premain-Class>PreMainAgent</Premain-Class>
                        <Agent-Class>PreMainAgent</Agent-Class>
                        <Can-Redefine-Classes>true</Can-Redefine-Classes>
                        <Can-Retransform-Classes>true</Can-Retransform-Classes>
                    </manifestEntries>
                </archive>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

- 在java_agent_demo工程创建PreMainAgent类。

```java
import java.lang.instrument.Instrumentation;

public class PreMainAgent {

    /**
     * 在这个 premain 函数中，开发者可以进行对类的各种操作。
     * 1、agentArgs 是 premain 函数得到的程序参数，随同 “– javaagent”一起传入。与 main 函数不同的是，
     * 这个参数是一个字符串而不是一个字符串数组，如果程序参数有多个，程序将自行解析这个字符串。
     * 2、Inst 是一个 java.lang.instrument.Instrumentation 的实例，由 JVM 自动传入。*
     * java.lang.instrument.Instrumentation 是 instrument 包中定义的一个接口，也是这个包的核心部分，
     * 集中了其中几乎所有的功能方法，例如类定义的转换和操作等等。
     *
     * @param agentArgs
     * @param inst
     */
    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("=========premain方法执行1========");
        System.out.println(agentArgs);
    }

    /**
     * 如果不存在 premain(String agentArgs, Instrumentation inst)
     * 则会执行 premain(String agentArgs)
     *
     * @param agentArgs
     */
    public static void premain(String agentArgs) {
        System.out.println("=========premain方法执行2========");
        System.out.println(agentArgs);
    }

}
```

- 通过IDEA，进行打包。

- java_agent_user工程新建一个测试类。

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("你好 世界");
    }
}
```



- 先运行一次，然后点击编辑MAIN启动类：

- 在VM Options中添加代码：

```shell
-javaagent:D:/project/java_agent/java_agent_demo/target/java_agent_demo-1.0.jar
```

- 启动的时候加载javaagent，指向java_agent_demo工程编译出来的javaagent的jar包地址。

### 统计方法的调用时间

- Skywalking中对每个调用的时长都进行了统计，我们要使用ByteBuddy和java agent技术来统计方法的调用时长。

- Byte Buddy是开源的、基于Apache2.0许可证的库，它致力于解决字节码操作和instrumentation API的复杂性。Byte Buddy所声称的目标是将显示的字节码操作隐藏在一个类型安全的领域特定语言背后，通过Byte Buddy，任何熟悉Java编程语言的人都有望非常容易的进行字节码的操作。Byte Buddy提供了额外的依赖API来生成java agent，可以轻松的增加我们已有的代码。

- 需要在java_agent_demo工程中添加如下的依赖：



```xml
<dependency>
    <groupId>net.bytebuddy</groupId>
    <artifactId>byte-buddy</artifactId>
    <version>1.9.2</version>
</dependency>
<dependency>
    <groupId>net.bytebuddy</groupId>
    <artifactId>byte-buddy-agent</artifactId>
    <version>1.9.2</version>
</dependency>
```

- 新建一个MyInterceptor的类，用来统计调用的时长。

```java
import net.bytebuddy.implementation.bind.annotation.Origin;
import net.bytebuddy.implementation.bind.annotation.RuntimeType;
import net.bytebuddy.implementation.bind.annotation.SuperCall;

import java.lang.reflect.Method;
import java.util.concurrent.Callable;

public class MyInterceptor {
    @RuntimeType
    public static Object intercept(@Origin Method method,
                                   @SuperCall Callable<?> callable)
            throws Exception {
        long start = System.currentTimeMillis();
        try {
            //执行原方法
            return callable.call();
        } finally {
            //打印调用时长
            System.out.println(method.getName() + ":" + (System.currentTimeMillis() - start)  + "ms");
        }
    }
}
```

- 修改PreMainAgent的代码：

```java
import net.bytebuddy.agent.builder.AgentBuilder;
import net.bytebuddy.description.method.MethodDescription;
import net.bytebuddy.description.type.TypeDescription;
import net.bytebuddy.dynamic.DynamicType;
import net.bytebuddy.implementation.MethodDelegation;
import net.bytebuddy.matcher.ElementMatchers;
import net.bytebuddy.utility.JavaModule;

import java.lang.instrument.Instrumentation;

public class PreMainAgent {

    public static void premain(String agentArgs, Instrumentation inst) {
        //创建一个转换器，转换器可以修改类的实现
        //ByteBuddy对java agent提供了转换器的实现，直接使用即可
        AgentBuilder.Transformer transformer = new AgentBuilder.Transformer() {
            public DynamicType.Builder<?> transform(DynamicType.Builder<?> builder, TypeDescription typeDescription, ClassLoader classLoader, JavaModule javaModule) {
                return builder
                        // 拦截任意方法
                        .method(ElementMatchers.<MethodDescription>any())
                        // 拦截到的方法委托给TimeInterceptor
                        .intercept(MethodDelegation.to(MyInterceptor.class));
            }
        };
        new AgentBuilder // Byte Buddy专门有个AgentBuilder来处理Java Agent的场景
                .Default()
                // 根据包名前缀拦截类
                .type(ElementMatchers.nameStartsWith("com.agent"))
                // 拦截到的类由transformer处理
                .transform(transformer)
                .installOn(inst);
    }
}
```

- 对java_agent_demo工程重新打包。

- 将java_agent_user工程中的Main类方法com.agent包下，修改代码的内容为：

```java
package com.agent;

/**
 * @author 许大仙
 * @version 1.0
 * @since 2021-01-21 08:46
 */
public class Main {
    public static void main(String[] args) {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("你好 世界");
    }
}
```

- 执行Main方法之后的显示结果为

- 我们在没有修改代码的情况下，利用了java agent和Byte Buddy统计出了方法的时长，Skywalking的agent也是基于这些技术来实现统计时长的调用的。

## Open Tracing介绍

### 概述

- Open Tracing通过提供平台无关、厂商无关的API，使得开发人员能够方便的添加（或更换）追踪系统的实现。Open Tracing最核心的概念就是Trace。

### Trace

- 在广义上，一个trace代表了一个事务或者流程（在分布式）系统中的执行过程。在Open Tracing标准中，trace是多个span的一个有向无环图（DAG），每一个span代表trace中被命名并计时的连续性的执行片段。

- 例如客户端发起一次请求，就可以认为是一次Trace。将上面的图通过Open Tracing的语义修改完之后做可视化，得到下面的图：

- 图中的每一个色块其实就是一个span。

### Span的概念

- 一个Span代表系统中具有开始时间和执行时长的逻辑运行单元。span之间通过嵌套或者顺序排列建立逻辑因果关系。

- Span里面的信息包含：操作的名字，开始时间和结束时间，可以携带多个`key:value`构成的Tags（key必须是String，value可以是String、Boolean或者数字等），还可以携带Logs信息（不一定所有的实现都支持），也必须是`key:value`形式。

- 下面的例子是一个Trace，里面有8个Span：

```latex
       [Span A]  ←←←(the root span)        
           |
    +------+------+   
    |             |
[Span B]      [Span C] ←←←(Span C 是    Span A 的孩子节点, ChildOf)   
    |            |
[Span D]     +---+-------+
             |           |
          [Span E]    [Span F] >>> [Span G] >>> [Span H]                    
                                        ↑
                                        ↑
                                        ↑
                        (Span G 在 Span F 后被调用, FollowsFrom)
```

- 一个Span可以和一个或者多个Span间存在因果关系。Open Tracing定义了两种关系：ChildOf和FollowsFrom。这两种引用类型代表了子节点和父节点间的直接因果关系。未来，OpenTracing将支持非因果关系的span引用关系。（例如：多个span被批量处理，span在同一个队列中，等等）

- ChildOf 很好理解，就是父亲 Span 依赖另一个孩子 Span。比如函数调用，被调者是调用者的孩子，比如说 RPC 调用，服务端那边的Span，就是 ChildOf 客户端的。很多并发的调用，然后将结果聚合起来的操作，就构成了 ChildOf 关系。

- 如果父亲 Span 并不依赖于孩子 Span 的返回结果，这时可以说它他构成 FollowsFrom 关系。

### Log的概念

- 每个Span可以进行多次Logs操作，每一个Logs操作，都需要一个带时间戳的时间名称、以及可选的任意大小的存储结果。

###  Tags的概念

- 每个Span可以有多个键值对（`key:value`）形式的Tags，Tags是没有时间戳的，支持简单的对Span进行注解和补充。



# 启动OAP 


启动OAP非常简单，OAP的代码是源码根目录下的oap-server，入口函数是 org.apache.skywalking.oap.server.starter 包下面的OAPServerStartUp类。直接启动即可。

Agent和OAP之间是通过gRPC来发送链路信息的。Agent端维护了一个队列（默认5个channel，每个channel大小为300）和一个线程池（默认1个线程，后面称为发送线程），链路数据采集后主线程（即业务线程）会写入这个队列，如果队列满了，主线程会直接把把数据丢掉（丢的时候会以debug级别打印日志）。发送线程会从队列取数据通过gRPC发送给后端OAP，OAP经过处理后写入存储。为了看得清楚，我把涉及的框架类画到了下面的图里面（格式是： {类名}#{方法名}({方法中调用的重要函数} )：

org.apache.skywalking.oap.server.receiver.trace.provider.handler.v8.grpc.TraceSegmentReportServiceHandler#collect

其内部功能很多，从是Segment这个数据流程来说就是：构建多个监听器，以监听器的模式来通过解析segmentObject各个属性，通过构建SourceBuilder对象来承载上下游的链路相关信息，并添加到entrySourceBuilders中；在build环节，进一步构建成各维度的souce数据，包括Trace(链路),Metrics(调用统计如调用次数，pxx，响应时长等) 信息都在这个环节创建。先大致看下其代码主体流程，接下来会分析内部更多的细节逻辑：

- SegmentAnalysisListener#parseSegment构建Segment（Source）,部分属性赋值
  1.1 赋值 起止时间
  1.2 赋值 是否error
  1.3 赋值 是否采样，这里是重点
- SegmentAnalysisListener#notifyFirstListener 更多的属性赋值
- 多个EntryAnalysisListener监听器处理Entry类型的span
  3.1 SegmentAnalysisListener#parseEntry赋值service和endpoint的Name和id
- 3.2 NetworkAddressAliasMappingListener#parseEntry 构造NetworkAddressAliasSetup完善ip_port地址与别名之间的映射关心，交给NetworkAddressAliasSetupDispatcher处理
- 3.3 MultiScopesAnalysisListener#parseEntry 遍历span列表
- 3.3.1 将每个span构建成SourceBuilder，设置上下游的游的Server、Instance、endpoint的name信息，这里mq和网关特殊处理，其上游保持ip端口，因为mq、网关通常没有搭载agent，没有相关的name信息。
- 3.3.2 setPublicAttrs：SourceBuilder中添加 tag信息，重点是时间bucket,setResponseCode，Status，type(http,rpc,db)
- 3.3.3 SourceBuilder添加到entrySourceBuilders,
- 3.3.4 parseLogicEndpoints//处理span的tag是LOGIC_ENDPOINT = "x-le"类型的，添加到 logicEndpointBuilders中（用途待梳理)
- MultiScopesAnalysisListener#parseExit监听器处理Exit类型的span
  4.1 将span构建成SourceBuilder，设置上下游的游的Server、Instance、Endpoint的name信息，尝试把下游的ip_port信息修改成别名。
  4.2 setPublicAttrs：SourceBuilder中添加 tag信息，重点是时间bucket,setResponseCode，Status，type(http,rpc,db)
  4.3 SourceBuilder添加到exitSourceBuilders，
  4.4 如果是db类型，构造slowStatementBuilder，判断时长设置慢查询标识，存入dbSlowStatementBuilders中。这里是全局的阈值 是个改造点。
- MultiScopesAnalysisListener#parseLocal监听器处理Local类型的span,通过parseLogicEndpoints方法处理span的tag是LOGIC_ENDPOINT = "x-le"类型的，添加到 logicEndpointBuilders中（用途待梳理)
- 6.1 SegmentAnalysisListener#build,设置endpoint的 id 和name，然后将Segment交给SourceReceiver#receive处理，而SourceReceiver#receive就是调用dispatcherManager#forward,最终交给SegmentDispatcher#dispatch处理了，
  6.2 MultiScopesAnalysisListener#build中根据以上流程中创建的数据，会再构造出多种Metric 类型的Source数据交给SourceReceiver处理；这些逻辑在这篇笔记中不展开，本篇已Segment流程为主

# SkyWalking源码

## 设计

如何要实现一个完整的分布式追踪系统。有以下几个问题需要我们仔细思考一下

- 怎么自动采集 span 数据：自动采集，对业务代码无侵入
- 如何跨进程传递 context
- traceId 如何保证全局唯一
- 请求量这么多采集会不会影响性能

接下我来看看 SkyWalking 是如何解决以上四个问题的

##### **怎么自动采集 span 数据**

SkyWalking 采用了**插件化 + javaagent** 的形式来实现了 span 数据的自动采集，这样可以做到对代码的无侵入性，插件化意味着可插拔，扩展性好(后文会介绍如何定义自己的插件)

##### 如何跨进程传递 context

我们知道数据一般分为 header 和 body, 就像 http 有 header 和 body, RocketMQ 也有 MessageHeader，Message Body, body 一般放着业务数据，所以不宜在 body 中传递 context，应该在 header 中传递 context

##### traceId 如何保证全局唯一

要保证全局唯一 ，我们可以采用分布式或者本地生成的 ID，使用分布式话需要有一个发号器，每次请求都要先请求一下发号器，会有一次网络调用的开销，所以 SkyWalking 最终采用了本地生成 ID 的方式，它采用了大名鼎鼎的 snowflow 算法，性能很高。

##### **请求量这么多，全部采集会不会影响性能?**

如果对每个请求调用都采集，那毫无疑问数据量会非常大，但反过来想一下，是否真的有必要对每个请求都采集呢，其实没有必要，我们可以设置采样频率，只采样部分数据，SkyWalking 默认设置了 3 秒采样 3 次，其余请求不采样

## 架构

SkyWalking 的基础如下架构，可以说几乎所有的的分布式调用都是由以下几个组件组成的

- **Agent** ：负责从应用中，收集链路信息，发送给 SkyWalking OAP 服务器。目前支持 SkyWalking、Zikpin、Jaeger 等提供的 Tracing 数据信息。而我们目前采用的是，SkyWalking Agent 收集 SkyWalking Tracing 数据，传递给服务器。
- **SkyWalking OAP** ：负责接收 Agent 发送的 Tracing 数据信息，然后进行分析(Analysis Core) ，存储到外部存储器( Storage )，最终提供查询( Query )功能。
- **Storage** ：Tracing 数据存储。目前支持 ES、MySQL、Sharding Sphere、TiDB、H2 多种存储器。
- **SkyWalking UI** ：负责提供控台，查看链路等等。

## 官方文档

在 https://github.com/apache/skywalking/tree/master/docs 地址下，提供了 SkyWalking 的**英文**文档。

推荐先阅读 https://github.com/SkyAPM/document-cn-translation-of-skywalking 地址，提供了 SkyWalking 的**中文**文档。

使用 SkyWalking 的目的，是实现**分布式链路追踪**的功能，所以最好去了解下相关的知识。这里推荐阅读两篇文章：

- [《OpenTracing 官方标准 —— 中文版》](https://github.com/opentracing-contrib/opentracing-specification-zh)
- Google 论文 [《Dapper，大规模分布式系统的跟踪系统》](http://www.iocoder.cn/Fight/Dapper-translation/?self)

## 调试环境搭建

##### 源码拉取

从官方仓库 https://github.com/OpenSkywalking/skywalking `Fork` 出属于自己的仓库。为什么要 `Fork` ？既然开始阅读、调试源码，我们可能会写一些注释，有了自己的仓库，可以进行自由的提交。

使用 `IntelliJ IDEA` 从 `Fork` 出来的仓库拉取代码。拉取完成后，`Maven` 会下载依赖包，可能会花费一些时间，耐心等待下。

本文基于 `master` 分支。

##### 启动 SkyWalking Collector

1. 在 IntelliJ IDEA Terminal 中，执行 `mvn compile -Dmaven.test.skip=true` 进行编译。
2. 设置 gRPC 的**自动生成**的代码目录，为**源码**目录 ：
   - /apm-network/target/generated-sources/protobuf/ 下的 `grpc-java` 和 `java` 目录
   - /apm-collector-remote/collector-remote-grpc-provider/target/generated-sources/protobuf/ 下的 `grpc-java` 和 `java` 目录
3. 运行 `org.skywalking.apm.collector.bootCollector.BootStartUp` 的 `#main(args)` 方法，启动 Collector 。
4. 访问 `http://127.0.0.1:10800/agent/jetty` 地址，返回 `["localhost:12800/"]` ，说明启动**成功**。

##### 启动 SkyWalking Agent

1. 在 IntelliJ IDEA Terminal 中，执行 `mvn compile -Dmaven.test.skip=true` 进行编译。在 /packages/skywalking-agent 目录下，我们可以看到编译出来的 Agent ：

2. 使用 Spring Boot 创建一个简单的 Web 项目。

   > 友情提示 ：**这里一定要注意下**。创建的 Web 项目，使用 IntelliJ IDEA 的**菜单** File / New / Module 或 File / New / Module from Existing Sources ，**保证 Web 项目和 skywalking 项目平级**。这样，才可以使用 IntelliJ IDEA 调试 Agent 。

3. 在 `org.skywalking.apm.agent.SkyWalkingAgent` 的 `#premain(...)` 方法，打上调试断点。

4. 运行 Web 项目的 Application 的 `#main(args)` 方法，并增加 JVM 启动参数，`-javaagent:/path/to/skywalking-agent/skywalking-agent.jar`。`/path/to` **参数值**为上面我们编译出来的 /packages/skywalking-agent 目录的绝对路径。

5. 如果在【**第三步**】的调试断点停住，说明 Agent 启动**成功**。

------

考虑到可能我们会在 Agent 上增加代码注释，这样每次不得不重新编译 Agent 。可以配置如下图，自动编译 Agent ：

- `-T 1C clean package -Dmaven.test.skip=true -Dmaven.compile.fork=true` 。

------

另外，使用 IntelliJ IDEA Remote 远程调试，也是可以的。

##### 启动 SkyWalking Web UI

考虑到调试过程中，我们要看下是否收集到追踪日志，可以安装 SkyWalking Web UI 进行查看。

## 模块

SkyWalking 源码的整体结构如下图所示：

1、apm-application-toolkit 模块：SkyWalking 提供给用户调用的工具箱。

该模块提供了对 log4j、log4j2、logback 等常见日志框架的接入接口，提供了 @Trace 注解等。
apm-application-toolkit 模块类似于暴露 API 定义，对应的处理逻辑在 apm-sniffer/apm-toolkit-activation 模块中实现。

2、apm-commons 模块：SkyWalking 的公共组件和工具类。

其中包含两个子模块，apm-datacarrier 模块提供了一个生产者-消费者模式的缓存组件（DataCarrier），无论是在 Agent 端还是 OAP 端都依赖该组件。
apm-util 模块则提供了一些常用的工具类，例如，字符串处理工具类（StringUtil）、占位符处理的工具类（PropertyPlaceholderHelper、PlaceholderConfigurerSupport）等等。
3、apm-protocol 模块：该模块中只有一个 apm-network 模块，我们需要关注的是其中定义的 .proto 文件，定义 Agent 与后端 OAP 使用 gRPC 交互时的协议。

4、apm-sniffer 模块：apm-sniffer 模块中有 4 个子模块：

apm-agent 模块：其中包含了刚才使用的 SkyWalkingAgent 这个类，是整个 Agent 的入口。

apm-agent-core 模块：SkyWalking Agent 的核心实现都在该模块中。

apm-sdk-plugin 模块：SkyWalking Agent 使用了微内核+插件的架构，该模块下包含了 SkyWalking Agent 的全部插件，

apm-toolkit-activation 模块：apm-application-toolkit 模块的具体实现。

5、apm-webapp 模块：SkyWalking Rocketbot 对应的后端。

6、oap-server 模块：SkyWalking OAP 的全部实现都在 oap-server 模块，其中包含了多个子模块，
exporter 模块：负责导出数据。

server-alarm-plugin 模块：负责实现 SkyWalking 的告警功能。

server-cluster-pulgin 模块：负责 OAP 的集群信息管理，其中提供了接入多种第三方组件的相关插件，

server-configuration 模块：负责管理 OAP 的配置信息，也提供了接入多种配置管理组件的相关插件，

server-core模块：SkyWalking OAP 的核心实现都在该模块中。

server-library 模块：OAP 以及 OAP 各个插件依赖的公共模块，其中提供了双队列 Buffer、请求远端的 Client 等工具类，这些模块都是对立于 SkyWalking OAP 体系之外的类库，我们可以直接拿走使用。

server-query-plugin 模块：SkyWalking Rocketbot 发送的请求首先由该模块接收处理，目前该模块只支持 GraphQL 查询。

server-receiver-plugin 模块：SkyWalking Agent 发送来的 Metrics、Trace 以及 Register 等写入请求都是首先由该模块接收处理的，不仅如此，该模块还提供了多种接收其他格式写入请求的插件，

server-starter 模块：OAP 服务启动的入口。

server-storage-plugin 模块：OAP 服务底层可以使用多种存储来保存 Metrics 数据以及Trace 数据，该模块中包含了接入相关存储的插件，

7、skywalking-ui 目录：SkyWalking Rocketbot 的前端。

## 源码解读

#### 源码分析--Agent 初始化

**SkyWalking Agent 启动初始化的过程**。

SkyWalking Agent 基于 **JavaAgent** 机制，实现应用**透明**接入 SkyWalking 。关于 JavaAgent 机制，笔者推荐如下两篇文章 ：

- [《Instrumentation 新功能》](https://www.ibm.com/developerworks/cn/java/j-lo-jse61/index.html)
- [《JVM源码分析之javaagent原理完全解读》](http://www.infoq.com/cn/articles/javaagent-illustrated)

> 友情提示 ：建议自己手撸一个简单的 JavaAgent ，更容易理解 SkyWalking Agent 。

`org.apache.skywalking.apm.agent.SkyWalkingAgent` ，在 `apm-sniffer/apm-agent` Maven 模块项目里，SkyWalking Agent **启动入口**。为什么说它是启动入口呢？在 `apm-sniffer/apm-agent` 的 [`pom.xml`](https://github.com/OpenSkywalking/skywalking/blob/23133f7d97d17b471f69e7214a01885ebcd2e882/apm-sniffer/apm-agent/pom.xml#L53) 文件的【我们可以看到 SkyWalkingAgent 被配置成 JavaAgent 的 **PremainClass** 。

```
 <premain.class>org.apache.skywalking.apm.agent.SkyWalkingAgent</premain.class>
```

- 调用 `SnifferConfigInitializer#initialize()` 方法，初始化 Agent 配置。
- 调用 `PluginBootstrap#loadPlugins()` 方法，加载 Agent 插件们。而后，创建 PluginFinder 。
- 调用 `ServiceManager#boot()` 方法，初始化 Agent 服务管理。在这过程中，Agent 服务们会被初始化。
- 基于 [byte-buddy](https://github.com/raphw/byte-buddy) ，初始化 Instrumentation 的 [`java.lang.instrument.ClassFileTransformer`](https://docs.oracle.com/javase/7/docs/api/java/lang/instrument/ClassFileTransformer.html) 。

#### **源码解读-初始化 Agent 配置**

SkyWalkingAgent#premain方法：

SnifferConfigInitializer.initializeCoreConfig(agentArgs);来进行配置的初始化

```
	//主要入口。使用byte-buddy 转换来增强所有在插件中定义的类。
    public static void premain(String agentArgs, Instrumentation instrumentation) throws PluginException {
        final PluginFinder pluginFinder;
        try {
            //初始化配置信息：javaagent:/……/agent.jar=k1=v1,k2=v2,……
            SnifferConfigInitializer.initializeCoreConfig(agentArgs);
        } catch (Exception e) {
            // try to resolve a new logger, and use the new logger to write the error log here
            LogManager.getLogger(SkyWalkingAgent.class)
                    .error(e, "SkyWalking agent initialized failure. Shutting down.");
            return;
        } finally {
            // refresh logger again after initialization finishes
            LOGGER = LogManager.getLogger(SkyWalkingAgent.class);
        }

        ……
    }
```

initializeCoreConfig方法

```
/**
     * 如果设置了指定的代理配置路径，则代理将尝试查找指定的代理配置。如果未设置指定的代理配置路径，则代理将尝试找到“ agent.config”，该目录应位于代理软件包的/ config目录中。
     * <p>
     * 另外，尝试通过system.properties覆盖配置。该位置的所有键都应以skywalking开头。例如在`skywalking.agent.service_name = yourAppName`中覆盖配置文件中的`agent.service_name`。
     就是javaagent:/……/agent.jar=k1=v1,k2=v2,……通过这种方式来覆盖配置的信息
     * <p>
     * 最后，“ agent.service_name”和“ collector.servers”不能为空。
     */ 

public static void initializeCoreConfig(String agentOptions) {
        AGENT_SETTINGS = new Properties();
     	//loadConfig来进行加载配置项
        try (final InputStreamReader configFileStream = loadConfig()) {
            AGENT_SETTINGS.load(configFileStream);
            for (String key : AGENT_SETTINGS.stringPropertyNames()) {
                String value = (String) AGENT_SETTINGS.get(key);
                //这里来处理${SW_AGENT_NAME:boot_demo} =》boot——demo
                AGENT_SETTINGS.put(key, PropertyPlaceholderHelper.INSTANCE.replacePlaceholders(value, AGENT_SETTINGS));
            }

        } catch (Exception e) {
            LOGGER.error(e, "Failed to read the config file, skywalking is going to run in default config.");
        }

       ……
    }
```

在initializeCoreConfig方法中，调用了*loadConfig*()来进行初始化配置内容

```
 private static ILog LOGGER = LogManager.getLogger(SnifferConfigInitializer.class);
    private static final String SPECIFIED_CONFIG_PATH = "skywalking_config";
    private static final String DEFAULT_CONFIG_FILE_NAME = "/config/agent.config";
    private static final String ENV_KEY_PREFIX = "skywalking.";
    private static Properties AGENT_SETTINGS;
    private static boolean IS_INIT_COMPLETED = false;	

	/**
     * 加载指定的配置文件或默认配置文件
     *
     * @return the config file {@link InputStream}, or null if not needEnhance.
     */
    private static InputStreamReader loadConfig() throws AgentPackageNotFoundException, ConfigNotFoundException {
        //获取skywalking_config配置的环境变量：
        String specifiedConfigPath = System.getProperty(SPECIFIED_CONFIG_PATH);
        //如果环境变量没有配置，就读取默认的配置信息：文件目录+/config/agent.config
        File configFile = StringUtil.isEmpty(specifiedConfigPath) ? new File(
            AgentPackagePath.getPath(), DEFAULT_CONFIG_FILE_NAME) : new File(specifiedConfigPath);

        if (configFile.exists() && configFile.isFile()) {
            try {
                //log日志中会显示这段信息
                LOGGER.info("Config file found in {}.", configFile);

                return new InputStreamReader(new FileInputStream(configFile), StandardCharsets.UTF_8);
            } catch (FileNotFoundException e) {
                throw new ConfigNotFoundException("Failed to load agent.config", e);
            }
        }
        throw new ConfigNotFoundException("Failed to load agent.config.");
    }
```

*最后使用initializeConfig*(Config.class);方法将配置文件放入到Config类中。

最终调用：ConfigInitializer.*initialize*(*AGENT_SETTINGS*, configClass);对Config的属性进行复制。

#### **源码解读-插件加载**

SkyWalkingAgent#premain方法：

new PluginFinder(new PluginBootstrap().loadPlugins());来加载所有插件

```java
	//主要入口。使用byte-buddy 转换来增强所有在插件中定义的类。
    public static void premain(String agentArgs, Instrumentation instrumentation) throws PluginException {
        final PluginFinder pluginFinder;
       	//配置项加载完成后
        …………
         //加载插件  
		 try {
             //PluginBootstrap().loadPlugins()详见 1. loadPlugins方法解析
             //pluginFinder详见 2.pluginFinder解析
            pluginFinder = new PluginFinder(new PluginBootstrap().loadPlugins());
        } catch (AgentPackageNotFoundException ape) {
            LOGGER.error(ape, "Locate agent.jar failure. Shutting down.");
            return;
        } catch (Exception e) {
            LOGGER.error(e, "SkyWalking agent initialized failure. Shutting down.");
            return;
        }
        …………
    }
```

loadPlugins方法解析

```java
 /**
     * 加载所有插件
     *
     * @return plugin definition list.
     */
	//	AbstractClassEnhancePluginDefine 是所有插件的父类。提供了增强目标类的概述
    public List<AbstractClassEnhancePluginDefine> loadPlugins() throws AgentPackageNotFoundException {
        //初始化一个classLoder 详见1.1~1.4
        //作用：隔离资源，不同的classLoader具有不同的classpath，避免乱加载
        AgentClassLoader.initDefaultLoader();

        PluginResourcesResolver resolver = new PluginResourcesResolver();】
            
        //获取各插件包下的skywalking-plugin.def 配置文件 详见1.5~
        List<URL> resources = resolver.getResources();

        if (resources == null || resources.size() == 0) {
            LOGGER.info("no plugin files (skywalking-plugin.def) found, continue to start application.");
            return new ArrayList<AbstractClassEnhancePluginDefine>();
        }

        for (URL pluginUrl : resources) {
            try {
                //将skywalking-plugin.def配置文件读成K-V的PluginDefine类，然后放到 pluginClassList 中，缓存在内存中。                      
                PluginCfg.INSTANCE.load(pluginUrl.openStream());
            } catch (Throwable t) {
                LOGGER.error(t, "plugin file [{}] init failure.", pluginUrl);
            }
        }
		//PluginCfg提供了getPluginClassList方法 获取所有的pluginClassList    
        List<PluginDefine> pluginClassList = PluginCfg.INSTANCE.getPluginClassList();

        //创建插件定义集合
        List<AbstractClassEnhancePluginDefine> plugins = new ArrayList<AbstractClassEnhancePluginDefine>();
        //迭代获取插件定义
        for (PluginDefine pluginDefine : pluginClassList) {
            try {
                LOGGER.debug("loading plugin class {}.", pluginDefine.getDefineClass());
                //获取插件定义
                AbstractClassEnhancePluginDefine plugin = (AbstractClassEnhancePluginDefine) Class.forName(pluginDefine.getDefineClass(), true, AgentClassLoader
                    .getDefault()).newInstance();
                //放到插件集合中
                plugins.add(plugin);
            } catch (Throwable t) {
                LOGGER.error(t, "load plugin [{}] failure.", pluginDefine.getDefineClass());
            }
        }

        plugins.addAll(DynamicPluginLoader.INSTANCE.load(AgentClassLoader.getDefault()));

        return plugins;

    }
```

AgentClassLoader.initDefaultLoader();

初始化AgentClassLoader

```java
 /**
     * 初始化默认classLoader
     *
     * @throws AgentPackageNotFoundException if agent package is not found.
     */
    public static void initDefaultLoader() throws AgentPackageNotFoundException {
        if (DEFAULT_LOADER == null) {
            synchronized (AgentClassLoader.class) {
                if (DEFAULT_LOADER == null) {
                    //父类是PluginBootstrap的ClassLoader()
                    DEFAULT_LOADER = new AgentClassLoader(PluginBootstrap.class.getClassLoader());
                }
            }
        }
    }

    public AgentClassLoader(ClassLoader parent) throws AgentPackageNotFoundException {
        super(parent);
        //获取AgentPackagePath的路径（在配置项加载中已经说过）
        File agentDictionary = AgentPackagePath.getPath();
        //classpath 插件路径
        classpath = new LinkedList<>();
        //MOUNT对应的文件夹是"plugins"和"activations" 见 1.2
        Config.Plugin.MOUNT.forEach(mountFolder -> classpath.add(new File(agentDictionary, mountFolder)));
    }
```

在Config类中，对MOUNT的默认赋值为：

```java
public static List<String> MOUNT = Arrays.asList("plugins", "activations");
```

也就是说，加载/plugins和/activations文件夹下的所有插件。

**plugins**：是对各种框架进行增强的插件，比如springMVC,Dubbo,RocketMq，Mysql等……

**activations**：是对一些支持框架，比如日志、openTracing等工具。

AgentClassLoader中最开始有一个*registerAsParallelCapable 的*static方法。

用来尝试解决classloader死锁

 https://github.com/apache/skywalking/pull/2016

```java
 static {
        registerAsParallelCapable();
    }
/**
* 将给定的类加载器类型注册为并行的classLoader。
*/
static boolean register(Class<? extends ClassLoader> c) {
    synchronized (loaderTypes) {
        if (loaderTypes.contains(c.getSuperclass())) {
            //如果且仅当其所有父类都具并行功能有时，才将类加载器注册为并行加载。 
            //注意：给定当前的类加载顺序，如果父类具有并行功能，所有更高级别的父类也必须具有并行能力。
            loaderTypes.add(c);
            return true;
        } else {
            return false;
        }
    }
}
```

AgentClassLoader.findClass 

将所有的jar包加载到内存中，做一个缓存。然后根据不同的class文件名，从缓存中提取文件。

```java
 @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        //找到所有的jar包
        List<Jar> allJars = getAllJars();
        
        String path = name.replace('.', '/').concat(".class");
        for (Jar jar : allJars) {
            JarEntry entry = jar.jarFile.getJarEntry(path);
            if (entry == null) {
                continue;
            }
            try {
                URL classFileUrl = new URL("jar:file:" + jar.sourceFile.getAbsolutePath() + "!/" + path);
                byte[] data;
                try (final BufferedInputStream is = new BufferedInputStream(
                    classFileUrl.openStream()); final ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
                    int ch;
                    while ((ch = is.read()) != -1) {
                        baos.write(ch);
                    }
                    data = baos.toByteArray();
                }
                return processLoadedClass(defineClass(name, data, 0, data.length));
            } catch (IOException e) {
                LOGGER.error(e, "find class fail.");
            }
        }
        throw new ClassNotFoundException("Can't find " + name);
    }
private List<Jar> allJars;
private ReentrantLock jarScanLock = new ReentrantLock();

private List<Jar> getAllJars() {
    //	这里其实相当于一个缓存
    if (allJars == null) {
        //加锁 锁类型为可重入锁：ReentrantLock
        jarScanLock.lock();
        try {
            if (allJars == null) {
                allJars = doGetJars();
            }
        } finally {
            jarScanLock.unlock();
        }
    }

    return allJars;
}
```

List<URL> resources = resolver.getResources();

加载skywalking-plugin.def配置文件

每个插件jar包中在resource文件下都有一个skywalking-plugin.def文件

**skywalking-plugin.def文件定义了插件的切入点。**



PluginFinder类解析

AbstractClassEnhancePluginDefine 分了3类：

nameMatchDefine：通过类名 **精确匹配**

signatureMatchDefine：通过签名/注解 或者其他条件 **间接匹配**

bootstrapClassMatchDefine：新增的bootstrap用的

```java
public class PluginFinder {
    // map的原因是skywalking-plugin.def配置中 key可能相同，但是实现有多中。
    private final Map<String, LinkedList<AbstractClassEnhancePluginDefine>> nameMatchDefine = new HashMap<String, LinkedList<AbstractClassEnhancePluginDefine>>();
    //
    private final List<AbstractClassEnhancePluginDefine> signatureMatchDefine = new ArrayList<AbstractClassEnhancePluginDefine>();
    //
    private final List<AbstractClassEnhancePluginDefine> bootstrapClassMatchDefine = new ArrayList<AbstractClassEnhancePluginDefine>();

    public PluginFinder(List<AbstractClassEnhancePluginDefine> plugins) {
        for (AbstractClassEnhancePluginDefine plugin : plugins) {
            //详见2.1
            ClassMatch match = plugin.enhanceClass();

            if (match == null) {
                continue;
            }
			//用一个明确的类名匹配该类。见2.1
            if (match instanceof NameMatch) {
                NameMatch nameMatch = (NameMatch) match;
                LinkedList<AbstractClassEnhancePluginDefine> pluginDefines = nameMatchDefine.get(nameMatch.getClassName());
                if (pluginDefines == null) {
                    pluginDefines = new LinkedList<AbstractClassEnhancePluginDefine>();
                    nameMatchDefine.put(nameMatch.getClassName(), pluginDefines);
                }
                pluginDefines.add(plugin);
            } else {
                signatureMatchDefine.add(plugin);
            }
			//bootstrap 插件中使用
            if (plugin.isBootstrapInstrumentation()) {
                bootstrapClassMatchDefine.add(plugin);
            }
        }
    }
	// TypeDescription 是对一个类型完整描述，包含了类全类名
    // 找到给定的一个类所有可以使用的全部插件
    // 分别从类名和辅助匹配条件两类插件中查找
    
    public List<AbstractClassEnhancePluginDefine> find(TypeDescription typeDescription) {
        List<AbstractClassEnhancePluginDefine> matchedPlugins = new LinkedList<AbstractClassEnhancePluginDefine>();
        
        //typeName就是全类名
        String typeName = typeDescription.getTypeName();
        if (nameMatchDefine.containsKey(typeName)) {
            matchedPlugins.addAll(nameMatchDefine.get(typeName));
        }
		//从间接匹配的插件中找
        for (AbstractClassEnhancePluginDefine pluginDefine : signatureMatchDefine) {
            IndirectMatch match = (IndirectMatch) pluginDefine.enhanceClass();
            if (match.isMatch(typeDescription)) {
                matchedPlugins.add(pluginDefine);
            }
        }

        return matchedPlugins;
    }
	//见下一章
    public ElementMatcher<? super TypeDescription> buildMatch() {
        ElementMatcher.Junction judge = new AbstractJunction<NamedElement>() {
            @Override
            public boolean matches(NamedElement target) {
                return nameMatchDefine.containsKey(target.getActualName());
            }
        };
        judge = judge.and(not(isInterface()));
        for (AbstractClassEnhancePluginDefine define : signatureMatchDefine) {
            ClassMatch match = define.enhanceClass();
            if (match instanceof IndirectMatch) {
                judge = judge.or(((IndirectMatch) match).buildJunction());
            }
        }
        return new ProtectiveShieldMatcher(judge);
    }

    public List<AbstractClassEnhancePluginDefine> getBootstrapClassMatchDefine() {
        return bootstrapClassMatchDefine;
    }
}
```

ClassMatch match = plugin.enhanceClass() 解析

enhanceClass实际是抽象接口，每个插件都有一个方法实现了这个接口。

以dubbo 2.7.X插件举例：

根据skywalking-plugin.def解析到，配置的切入点是DubboInstrumentation类，然后打开

发现是NameMatch.*byName，*即通过类名来进行匹配需要增强的类。

#### **源码解读-服务启动**

SkyWalkingAgent#premain方法：

```java
	//主要入口。使用byte-buddy 转换来增强所有在插件中定义的类。
    public static void premain(String agentArgs, Instrumentation instrumentation) throws PluginException {
       
       	//（1）配置项加载
        …………
        //（2）加载插件，并进行分类  
        …………
        //（3）设置 ByteBuddy
        // skyWalking 使用了ByteBuddy技术来进行字节码增强
        // IS_OPEN_DEBUGGING_CLASS 是否开启debug模式。 当为true时，会把增强过得字节码文件放到/debugging文件夹下，方便debug。
        final ByteBuddy byteBuddy = new ByteBuddy().with(TypeValidation.of(Config.Agent.IS_OPEN_DEBUGGING_CLASS));
		//AgentBuilder提供了便捷的API，用于定义Java agent.
        AgentBuilder agentBuilder = new AgentBuilder.Default(byteBuddy)
                .ignore( //指定以这些类名为开头的 不属于要增强的范围
                nameStartsWith("net.bytebuddy.")
                        .or(nameStartsWith("org.slf4j."))
                        .or(nameStartsWith("org.groovy."))
                        .or(nameContains("javassist"))
                        .or(nameContains(".asm."))
                        .or(nameContains(".reflectasm."))
                        .or(nameStartsWith("sun.reflect"))
                        .or(allSkyWalkingAgentExcludeToolkit())
                        .or(ElementMatchers.isSynthetic()));

      …………

        agentBuilder.type(pluginFinder.buildMatch())// 我们要通过插件增强的类，buildMatch解释见1.1
                    .transform(new Transformer(pluginFinder)) //Transformer 实际增强的方法，下期讲解
                    .with(AgentBuilder.RedefinitionStrategy.RETRANSFORMATION)
                    .with(new Listener())//监听器，打一些日志
                    .installOn(instrumentation);

        //（4）加载服务
        try {
            // 插件架构
            // agent-core 看做是内核
            // 所谓的服务就是各种插件，由core内核进行管理
            // 比如maven 就是插件架构
            // 插件架构的规范就是BootService，详见1.2
            ServiceManager.INSTANCE.boot();
        } catch (Exception e) {
            LOGGER.error(e, "Skywalking agent boot failure.");
        }
         //（5）注册关闭钩子
		// 注册一个JVM 的关闭钩子，当服务关闭时，调用shutdown方法释放资源。
        Runtime.getRuntime()
                .addShutdownHook(new Thread(ServiceManager.INSTANCE::shutdown, "skywalking service shutdown thread"));
    }
```

pluginFinder.buildMatch()详解

```java
//把所有需要增强的类构建成ElementMatcher
public ElementMatcher<? super TypeDescription> buildMatch() {
    ElementMatcher.Junction judge = new AbstractJunction<NamedElement>() {
        @Override
        public boolean matches(NamedElement target) {
            return nameMatchDefine.containsKey(target.getActualName());
        }
    };
    judge = judge.and(not(isInterface()));
    for (AbstractClassEnhancePluginDefine define : signatureMatchDefine) {
        ClassMatch match = define.enhanceClass();
        if (match instanceof IndirectMatch) {
            judge = judge.or(((IndirectMatch) match).buildJunction());
        }
    }
    // 注意 最后返回的是 ProtectiveShieldMatcher。ProtectiveShieldMatcher继承了ElementMatcher
    // 使用ProtectiveShieldMatcher的原因：
    /**
    * 在某些情况下，某些框架和库也使用binary技术。
    * 从社区反馈中，有些与byte-buddy有兼容的问题，会触发了“Can't resolve type description”异常。 
    * 因此，嵌套了一层。当原始matcher无法解析类型时，SkyWalking代理会忽略此类型。 
    * 注意：此忽略机制可能会遗漏某些检测手段，但在大多数情况下是没问题的。如果丢失，请注意警告日志。
    */
    return new ProtectiveShieldMatcher(judge);
}
```

ServiceManager.*INSTANCE*.boot();讲解

点进boot方法之后可以看到ServiceManager的代码如下：

```java
/**
 * agent 服务插件定义体系
 * 1. 所有的服务必须直接或者间接实现 BootService
 * 2. 使用@DefaultImplementor 表示一个服务的默认实现，使用@OverrideImplementor 表示一个服务的覆盖实现
 * 3. @OverrideImplementor 的 value 属性用于指定要覆盖服务的默认实现
 * 4. 覆盖实现必须明确指定一个默认实现，且只能覆盖默认实现，也就是说，一个覆盖实现不能去覆盖另一个覆盖实现。
 */
public enum ServiceManager {
    INSTANCE;

    private static final ILog LOGGER = LogManager.getLogger(ServiceManager.class);
    //BootService 见1.2.1
    private Map<Class, BootService> bootedServices = Collections.emptyMap();

    public void boot() {
        //加载所有的bootedServices 
        bootedServices = loadAllServices();
		//执行各生命周期方法
        prepare();
        startup();
        onComplete();
    }

    public void shutdown() {
        for (BootService service : bootedServices.values()) {
            try {
                service.shutdown();
            } catch (Throwable e) {
                LOGGER.error(e, "ServiceManager try to shutdown [{}] fail.", service.getClass().getName());
            }
        }
    }

    /**
     * 
     * @return
     */
    private Map<Class, BootService> loadAllServices() {
        Map<Class, BootService> bootedServices = new LinkedHashMap<>();
        List<BootService> allServices = new LinkedList<>();
        //加载所有服务
        load(allServices);
        for (final BootService bootService : allServices) {
            Class<? extends BootService> bootServiceClass = bootService.getClass();
            boolean isDefaultImplementor = bootServiceClass.isAnnotationPresent(DefaultImplementor.class);
            if (isDefaultImplementor) { // 有@DefaultImplementor注解
                if (!bootedServices.containsKey(bootServiceClass)) {
                    bootedServices.put(bootServiceClass, bootService);
                } else {
                    //ignore the default service
                }
            } else { // 没有@DefaultImplementor注解
                OverrideImplementor overrideImplementor = bootServiceClass.getAnnotation(OverrideImplementor.class);
                if (overrideImplementor == null) { //既没有@DefaultImplementor注解，又没有@DefaultImplementor注解
                    if (!bootedServices.containsKey(bootServiceClass)) {
                        bootedServices.put(bootServiceClass, bootService);
                    } else {
                        throw new ServiceConflictException("Duplicate service define for :" + bootServiceClass);
                    }
                } else { //有@DefaultImplementor注解
                    Class<? extends BootService> targetService = overrideImplementor.value();
                    if (bootedServices.containsKey(targetService)) {
                        boolean presentDefault = bootedServices.get(targetService)
                                                               .getClass()
                                                               .isAnnotationPresent(DefaultImplementor.class);
                        if (presentDefault) {
                            bootedServices.put(targetService, bootService);
                        } else {
                            throw new ServiceConflictException(
                                "Service " + bootServiceClass + " overrides conflict, " + "exist more than one service want to override :" + targetService);
                        }
                    } else {
                        bootedServices.put(targetService, bootService);
                    }
                }
            }

        }
        return bootedServices;
    }

    private void prepare() {
        for (BootService service : bootedServices.values()) {
            try {
                service.prepare();
            } catch (Throwable e) {
                LOGGER.error(e, "ServiceManager try to pre-start [{}] fail.", service.getClass().getName());
            }
        }
    }

    private void startup() {
        for (BootService service : bootedServices.values()) {
            try {
                service.boot();
            } catch (Throwable e) {
                LOGGER.error(e, "ServiceManager try to start [{}] fail.", service.getClass().getName());
            }
        }
    }

    private void onComplete() {
        for (BootService service : bootedServices.values()) {
            try {
                service.onComplete();
            } catch (Throwable e) {
                LOGGER.error(e, "Service [{}] AfterBoot process fails.", service.getClass().getName());
            }
        }
    }
```

BootService 代码

```java
/**
 * 是所有远程接口，当插件机制开始起作用时，需要启动该接口。 
 * 
 */
public interface BootService {
    void prepare() throws Throwable;

   	//boot方法将在 BootService 启动时被调用。
    void boot() throws Throwable;

    void onComplete() throws Throwable;

    void shutdown() throws Throwable;
}
```

@DefaultImplementor注解和@OverrideImplementor注解

@DefaultImplementor注解表示这是一个默认服务实现

@OverrideImplementor注解 表示这是一个服务的覆盖实现，覆盖的服务由value指定。

#### **源码解读-Agent 插件体系**

SkyWalkingAgent.Transformer#transform方法

```java
	 private static class Transformer implements AgentBuilder.Transformer {
        private PluginFinder pluginFinder;

        Transformer(PluginFinder pluginFinder) {
            this.pluginFinder = pluginFinder;
        }

        @Override
        public DynamicType.Builder<?> transform(final DynamicType.Builder<?> builder,
                                                final TypeDescription typeDescription,
                                                final ClassLoader classLoader,
                                                final JavaModule module) {
            List<AbstractClassEnhancePluginDefine> pluginDefines = pluginFinder.find(typeDescription);
            if (pluginDefines.size() > 0) {
                DynamicType.Builder<?> newBuilder = builder;
                EnhanceContext context = new EnhanceContext();
                for (AbstractClassEnhancePluginDefine define : pluginDefines) {
                    DynamicType.Builder<?> possibleNewBuilder = define.define(
                            typeDescription, newBuilder, classLoader, context);
                    if (possibleNewBuilder != null) {
                        newBuilder = possibleNewBuilder;
                    }
                }
                if (context.isEnhanced()) {
                    LOGGER.debug("Finish the prepare stage for {}.", typeDescription.getName());
                }

                return newBuilder;
            }

            LOGGER.debug("Matched class {}, but ignore by finding mechanism.", typeDescription.getTypeName());
            return builder;
        }
    }
```

插件结构（以dubbo-2.7.x-plugin为例）

DubboInstrumentation 代码解析

```java
/**
 * 任何XxxInstrumentation 用于定义插件拦截点的
 * 拦截点：指定类的指定方法（实例方法、构造方法、静态方法）
 * 一个Instrumentation 只能拦截一个类，即只能有一个enhanceClass，但是可以拦截这个类里面的多个方法，即getInstanceMethodsInterceptPoints()中可以new 多个InstanceMethodsInterceptPoint()
 */
public class DubboInstrumentation extends ClassInstanceMethodsEnhancePluginDefine {

    private static final String ENHANCE_CLASS = "org.apache.dubbo.monitor.support.MonitorFilter";

    private static final String INTERCEPT_CLASS = "org.apache.skywalking.apm.plugin.asf.dubbo.DubboInterceptor";

    /**
     * 指定插件要拦截的类
     */
    @Override
    protected ClassMatch enhanceClass() {
        return NameMatch.byName(ENHANCE_CLASS);
    }

    /**
     * 指定插件要拦截的方法，不拦截返回null
     */
    @Override
    public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        return null;
    }

    /**
     * 指定插件要拦截的实例方法（同样会有静态方法、构造方法，最后会总结）
     */
    @Override
    public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints() {
        return new InstanceMethodsInterceptPoint[]{
                new InstanceMethodsInterceptPoint() {
                    /**
                     * 返回要拦截的方法名
                     */
                    @Override
                    public ElementMatcher<MethodDescription> getMethodsMatcher() {
                        return named("invoke");
                    }

                    /**
                     * 指定拦截器全类名，用于拦截到指定方法之后做具体操作的
                     */
                    @Override
                    public String getMethodsInterceptor() {
                        //详见1.2DubboInterceptor
                        return INTERCEPT_CLASS;
                    }

                    /**
                     * 指定是否需要在拦截的时候对原方法参数进行修改
                     */
                    @Override
                    public boolean isOverrideArgs() {
                        return false;
                    }
                }
        };
    }
}
```

DubboInterceptor

```java
// spring AOP 切面时，@Around注解，方法前后
public class DubboInterceptor implements InstanceMethodsAroundInterceptor {

    public static final String ARGUMENTS = "arguments";

    /**
     * <h2>Consumer:</h2> The serialized trace context data will
     * inject to the {@link RpcContext#attachments} for transport to provider side.
     * <p>
     * <h2>Provider:</h2> The serialized trace context data will extract from
     * {@link RpcContext#attachments}. current trace segment will ref if the serialize context data is not null.
     */
    //原方法调用前
    @Override
    public void beforeMethod(EnhancedInstance objInst, Method method, Object[] allArguments, Class<?>[] argumentsTypes,
                             MethodInterceptResult result) throws Throwable {
        Invoker invoker = (Invoker) allArguments[0];
        Invocation invocation = (Invocation) allArguments[1];
        RpcContext rpcContext = RpcContext.getContext();
        boolean isConsumer = rpcContext.isConsumerSide();
        URL requestURL = invoker.getUrl();

        AbstractSpan span;

        final String host = requestURL.getHost();
        final int port = requestURL.getPort();

        boolean needCollectArguments;
        int argumentsLengthThreshold;
        if (isConsumer) {
            final ContextCarrier contextCarrier = new ContextCarrier();
            span = ContextManager.createExitSpan(generateOperationName(requestURL, invocation), contextCarrier, host + ":" + port);
            //invocation.getAttachments().put("contextData", contextDataStr);
            //@see https://github.com/alibaba/dubbo/blob/dubbo-2.5.3/dubbo-rpc/dubbo-rpc-api/src/main/java/com/alibaba/dubbo/rpc/RpcInvocation.java#L154-L161
            CarrierItem next = contextCarrier.items();
            while (next.hasNext()) {
                next = next.next();
                rpcContext.getAttachments().put(next.getHeadKey(), next.getHeadValue());
                if (invocation.getAttachments().containsKey(next.getHeadKey())) {
                    invocation.getAttachments().remove(next.getHeadKey());
                }
            }
            needCollectArguments = DubboPluginConfig.Plugin.Dubbo.COLLECT_CONSUMER_ARGUMENTS;
            argumentsLengthThreshold = DubboPluginConfig.Plugin.Dubbo.CONSUMER_ARGUMENTS_LENGTH_THRESHOLD;
        } else {
            ContextCarrier contextCarrier = new ContextCarrier();
            CarrierItem next = contextCarrier.items();
            while (next.hasNext()) {
                next = next.next();
                next.setHeadValue(rpcContext.getAttachment(next.getHeadKey()));
            }

            span = ContextManager.createEntrySpan(generateOperationName(requestURL, invocation), contextCarrier);
            span.setPeer(rpcContext.getRemoteAddressString());
            needCollectArguments = DubboPluginConfig.Plugin.Dubbo.COLLECT_PROVIDER_ARGUMENTS;
            argumentsLengthThreshold = DubboPluginConfig.Plugin.Dubbo.PROVIDER_ARGUMENTS_LENGTH_THRESHOLD;
        }

        Tags.URL.set(span, generateRequestURL(requestURL, invocation));
        collectArguments(needCollectArguments, argumentsLengthThreshold, span, invocation);
        span.setComponent(ComponentsDefine.DUBBO);
        SpanLayer.asRPCFramework(span);
    }
	//调用后
    @Override
    public Object afterMethod(EnhancedInstance objInst, Method method, Object[] allArguments, Class<?>[] argumentsTypes,
                              Object ret) throws Throwable {
        Result result = (Result) ret;
        if (result != null && result.getException() != null) {
            dealException(result.getException());
        }

        ContextManager.stopSpan();
        return ret;
    }
	//调用异常时
    @Override
    public void handleMethodException(EnhancedInstance objInst, Method method, Object[] allArguments,
                                      Class<?>[] argumentsTypes, Throwable t) {
        dealException(t);
    }

    /**
     * Log the throwable, which occurs in Dubbo RPC service.
     */
    private void dealException(Throwable throwable) {
        AbstractSpan span = ContextManager.activeSpan();
        span.log(throwable);
    }

    /**
     * Format operation name. e.g. org.apache.skywalking.apm.plugin.test.Test.test(String)
     *
     * @return operation name.
     */
    private String generateOperationName(URL requestURL, Invocation invocation) {
        StringBuilder operationName = new StringBuilder();
        operationName.append(requestURL.getPath());
        operationName.append("." + invocation.getMethodName() + "(");
        for (Class<?> classes : invocation.getParameterTypes()) {
            operationName.append(classes.getSimpleName() + ",");
        }

        if (invocation.getParameterTypes().length > 0) {
            operationName.delete(operationName.length() - 1, operationName.length());
        }

        operationName.append(")");

        return operationName.toString();
    }

    /**
     * Format request url. e.g. dubbo://127.0.0.1:20880/org.apache.skywalking.apm.plugin.test.Test.test(String).
     *
     * @return request url.
     */
    private String generateRequestURL(URL url, Invocation invocation) {
        StringBuilder requestURL = new StringBuilder();
        requestURL.append(url.getProtocol() + "://");
        requestURL.append(url.getHost());
        requestURL.append(":" + url.getPort() + "/");
        requestURL.append(generateOperationName(url, invocation));
        return requestURL.toString();
    }

    private void collectArguments(boolean needCollectArguments, int argumentsLengthThreshold, AbstractSpan span, Invocation invocation) {
        if (needCollectArguments && argumentsLengthThreshold > 0) {
            Object[] parameters = invocation.getArguments();
            if (parameters != null && parameters.length > 0) {
                StringBuilder stringBuilder = new StringBuilder();
                boolean first = true;
                for (Object parameter : parameters) {
                    if (!first) {
                        stringBuilder.append(",");
                    }
                    stringBuilder.append(parameter);
                    if (stringBuilder.length() > argumentsLengthThreshold) {
                        stringBuilder.append("...");
                        break;
                    }
                    first = false;
                }
                span.tag(ARGUMENTS, stringBuilder.toString());
            }
        }
    }
}
```

##### 总结一下

##### 拦截实例方法

XxxImplInstrumentation 继承 ClassInstanceMethodsEnhancePluginDefine

重写 getInstanceMethodsInterceptPoints（）来定义实例方法拦截点

XxxInterceptor implements InstanceMethodsAroundInterceptor

重写 beforeMethod afterMethod handleMethodException 方法实现拦截增强

##### 拦截构造方法

XxxImplInstrumentation 继承 ClassInstanceMethodsEnhancePluginDefine

重写 getConstructorsInterceptPoints（）来定义构造方法拦截点

XxxInterceptor implements InstanceConstructorInterceptor

重写 onConstruct 方法实现拦截增强

##### 拦截静态方法

XxxImplInstrumentation 继承 ClassStaticMethodsEnhancePluginDefine

重写 getStaticMethodsInterceptPoints（）来定义静态方法拦截点

XxxInterceptor implements StaticMethodsAroundInterceptor

重写 beforeMethod afterMethod handleMethodException 方法实现拦截增强

##### 对于实例方法和静态方法拦截点里的三个接口

getMethodsMatcher() 定义拦截哪个方法

getMethodsInterceptor() 定义拦截到方法之后要使用那个拦截器

isOverrideArgs() 指定在拦截器工作的时候是否可以对原方法入参进行修改

#### **源码解读-WitnessClass 机制**

AbstractClassEnhancePluginDefine#witnessClasses（）

AbstractClassEnhancePluginDefine是所有插件的父类

```java
/*
* Witness classname。为什么需要Witness classname？
* 比如：一个库存在两个发行的版本（例如1.0、2.0），
* 它们包含相同的目标类，但是由于版本迭代器的缘故，
* 它们可能具有相同的名称，但方法不同，或者方法参数列表不同。
* 因此，如果我要针对特定的版本（例如1.0），显然版本号没法判断；
* 这时需要“WitnessClasses”。您可以仅在此特定发行版本中添加任何类
* （类似于 com.company.1.x.A，仅在1.0中），您可以实现目标。
*/
protected String[] witnessClasses() {
    //数组的判断逻辑是 && ，有多个类时，必须全部满足，才可以使用。
    return new String[] {};
}
```

**示例**

mysql5.x

```java
public class Constants {
    public static final String WITNESS_MYSQL_5X_CLASS = "com.mysql.jdbc.ConnectionImpl";
}
```

mysql6.x

```java
public class Constants {
    public static final String WITNESS_MYSQL_6X_CLASS = "com.mysql.cj.api.MysqlConnection";
}
```

mysql8.x

```java
public class Constants {
    public static final String WITNESS_MYSQL_8X_CLASS = "com.mysql.cj.interceptors.QueryInterceptor";
}
```

解决了什么问题

解决了 我们拿不到业务系统所有用的组件版本号时，可以准确匹配所使用的agent增强插件。

### 源码分析-Trace收集

从这里开始，我们将开始介绍 Trace 格式、收集方式以及上报方式。

#### **Trace基本概念**

在开始介绍发送 Trace 的具体实现之前，先简单说一下 Trace 相关的基本概念，要是客官们已经了解了这些概念，可以跳过这部分。读者也可以参考一下《OpenTracing语义标准》(https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md) 。

OpenTracing中的一条Trace(调用链)可以被认为是一个由多个Span组成的有向无环图(DAG图)，Span 与 Span 的关系被命名为 References。Span 可以被理解为一次方法调用、一次 RPC 或是一次 DB 访问。

每个 Span 包含以下的状态:

An operation name，操作名称

A start timestamp，起始时间

A finish timestamp，结束时间

Span Tag，一组键值对构成的Span标签集合。键值对中，键必须为string，值可以是字符串，布尔，或者数字类型。

Span Log，一组span的日志集合。每次log操作包含一个键值对，以及一个时间戳。键值对中，键必须为string，值可以是任意类型。但是需要注意，不是所有的支持OpenTracing的Tracer,都需要支持所有的值类型。

SpanContext，Span上下文对象 (下面会详细说明)

References(Span间关系)，相关的零个或者多个Span（Span间通过SpanContext建立这种关系）

每一个 SpanContext 包含以下状态：

任何一个 OpenTracing 的实现，都需要将当前调用链的状态（例如：trace id 和 span id），依赖一个独特的 Span 去跨进程边界传输

Baggage Items，Trace 的随行数据，是一个键值对集合，它存在于 Trace 中，也需要跨进程边界传输

Span间关系

一个 Span 可以与一个或者多个**SpanContexts**存在因果关系。OpenTracing 目前定义了两种关系：（父子） 和（跟随）。这两种关系明确的给出了两个父子关系的Span的因果模型。

**ChildOf 引用：**一个 Span 可能是一个父级 Span 的孩子，即"ChildOf"关系。在 ChildOf 引用关系下，父级 Span 某种程度上取决于子 Span。下面这些情况会构成 ChildOf 关系：

一个 RPC 调用的服务端的 Span，和RPC服务客户端的 Span 构成 ChildOf 关系。

一个 sql insert 操作的 Span，和 ORM 的 save 方法的 Span 构成 ChildOf 关系。

很多 Span 可以并行工作（或者分布式工作）都可能是一个父级的 Span 的子项，它会合并所有子 Span 的执行结果，并在指定期限内返回。

**FollowFrom 引用：**个人觉得不是很常见，不说了。

回到 Skywalking ，在 Skywalking 的设计中，在 Trace级别 和 Span 级别之间加了一个 Segment 的概念，用于表示一个 OS 里面的 Span 集合。

Skywalking 中的Span分为三类：

1、EntrySpan：表示服务端的入口，包括但不限于Http服务、RPC服务、MQ-Consumer等

2、LocalSpan：表示本地的方法调用

3、ExitSpan：表示 Client 或是MQ-producer

Skywalking 中具体的 Span 实现在后面分析收集 Trace 的小节中再详细分析，这里先不深入分析了。

**TraceSegment**

TraceSegment 是一条 Trace 的一段，TraceSegment 用于记录当前线程的 Trace 信息。分布式系统基本会涉及跨线程的操作，例如， RPC、MQ等，怎么也要有 DB 访问吧，╮(╯_╰)╭，所以一条 Trace 基本都是由多个 TraceSegment 构成的。

我们常见的 RPC 调用啊、Http 调用啊之类的，每个 TraceSegment 只有一个 parent。但是当一个 Consumer 批量处理 MQ 消息的时候，其中的每条消息都来自不同的 Producer ，就会有多个 parent 了，同时这个 TraceSegment 也就属于多个 Trace 了。

traceSegmentId字段的类型是ID，它由三个long类型的字段(part1、part2、part3)构成，分别记录了 service_instance_id、线程Id、Context生成序列。

Context生成序列的格式是：

TraceSegment Id 的最终格式是：

再来是GlobalIdGenerator，它是用来生成 ID 对象的，直接看它的 generate() 方法：

**DistributedTraceId**

DistributedTraceId 用于生成全局的 TraceId，其中封装了一个 ID 类型的字段。DistributedTraceId 是个抽象类，它有两个实现类，如下图所示：

其中 NewDistirbutedTraceId 负责为新 Trace 生成编号，在请求进入我们的系统、第一次创建 TraceSegment 对象的时候，会创建NewDistirbutedTraceId对象，在其构造方法内部会调用 GlobalIdGenerator.generate() 方法生成创建 ID 对象。

PropagatedTraceId 负责处理 Trace 传播过程中的 TraceId，PropagatedTraceId的构造方法接收一个 String 类型的 TraceId ，解析之后得到 ID 对象。

TraceSegment中的 relatedGlobalTraces字段是DistributedTraceIds 类型，它的底层封装了一个 LinkedList 集合，用于记录当前 TraceSegment 关联的 TraceId。为什么是一个集合呢？与上面提到的，一个 TraceSegment 可能有多个 parent 的情况一样。

**Span**

TraceSegment 是由多个 Span 构成的，下图是 Span 的继承关系：

```
|-AsyncSpan
	|-AbstractSpan
		|-AbstractTracingSpan
			|-StackBasedTracingSpan
					|-ExitSpan
					|-EntrySpan
			|-LocalSpan
		|-NoopSpan
```

从顶层开始看呗，AsyncSpan 接口定义了一个异步 Span 的基本行为：

prepareForAsync()方法：当前 Span 在当前线程结束了，但是当前 Span 未被彻底关闭，依然是存活的。

asyncFinish()方法：当前Span 真正关闭。这与 prepareForAsync() 方法成对出现。

这两个方法在异步 RPC 中会见到。

AbstractSpan 也是一个接口，其中定义了 Span 的基本行为：

getSpanId() 方法:获得当前 Span 的编号，Span 编号是一个整数，在 TraceSegment 内唯一，从 0 开始自增，在创建 Span 对象时生成。

setOperationName()/setOperationId() 方法：设置操作名/操作编号。这两个方法是互斥的，在 AbstractTracingSpan 这个实现中，有 operationId 和 operationName 两个字段，只能有一个字段有值。

setComponent() 方法：设置组件。它有两个重载，在 AbstractTracingSpan 这个实现中，有 componentId 和 componentName 两个字段，两个重载分别用于设置这两个字段。在 ComponentsDefine 中可以找到 Skywalking目前支持的组件。

setLayer() 方法：设置 SpanLayer，也就是当前 Span 所处的层次。SpanLayer 是个枚举，可选项有 DB、RPC_FRAMEWORK、HTTP、MQ、CACHE。

tag(AbstractTag, String) 方法：为当前 Span 添加键值对的标签。一个 Span 可以投多个标签，AbstractTag 中就封装了 String 类型的 Key ，没啥可说的。

log() 方法：记录当前 Span 中发生的关键日志，一个 Span 可以包含多条日志。

start() 方法：开始 Span 。其实就是设置当前 Span 的开始时间以及调用层级等信息。

isEntry() 方法：当前是否是入口 Span。

isExit() 方法：当前是否是出口 Span。

ref() 方法：设置关联的 TraceSegment 。

AbstractTracingSpan 实现了 AbstractSpan 接口，其中定义了一些 Span 的基础字段，其中很多字段一眼看过去就知道是啥意思了，就不一一展开介绍了，其中有一个字段：

AbstractTracingSpan 中的方法也比较简单，基本都是 getter/setter 方法，其中有一个 finish() 方法，会更新 endTime 字段并将当前 Span 记录到给定的 TraceSegment 中。

StackBasedTracingSpan 这种 Span 可以多次调用 start() 方法和 end() 方法，就类似一个栈。其中多了两个字段：

StackBasedTracingSpan.finish() 方法会在栈彻底退出的时候，才会将当前 Span 添加到 TraceSegment 中：

EntrySpan 表示的是一个服务的入口 Span，主要用在服务提供方的入口，例如，Dubbo Provider、Tomcat、Spring MVC 等等。EntrySpan 是 TraceSegment 的第一个 Span ，这也是为什么称之为"入口" Span 的原因。那么为什么 EntrySpan 继承 StackBasedTracingSpan？从前面对 Skywalking Agent 的分析来看，Agent 只会根据插件在相应的位置

对方法进行增强，具体的增强逻辑就包含创建 EntrySpan 对象(后面在分析具体插件实现的时候，会看到具体的实现代码)，例如，Tomcat插件 和 Spring MVC 插件。很多 Web 项目会同时使用这两个插件，难道一个 TraceSegment 要有两个 EntrySpan 吗？

显然不合适，所以 EntrySpan 继承了 StackBasedTracingSpan，当请求经过 Tomcat 插件的时候，会创建 EntrySpan，当请求经过 Spring MVC 插件的时候，不会再创建新的 EntrySpan 了，只是 currentMaxDepth 字段加 1。

currentMaxDepth 字段是 EntrySpan 中用于记录当前 EntrySpan 的深度的，前面介绍StackBasedTracingSpan.finish() 方法代码时看到，只有 stackDepth 为 1 的时候，才能结束当前 Span。

EntrySpan 要关注的是其 start() 方法：

虽然 EntrySpan 是在第一个增强逻辑中创建的，但是后续每次 start()方法都会清空所有字段，所以 EntrySpan 除了 startTime 和 endTime 以外的字段都是以最后一次调用 start() 方法写入的为准。在 EntrySpan 中的 set* 方法会检测currentMaxDepth是否为最底层，如果不是，设置相关字段没有什么意义，例如 tag()方法：

ExitSpan 表示的是出口 Span，主要用于服务的消费者，例如，Dubbo Consumer、HttPClient等等。如果在一个调用栈里面出现多个插件创建 ExitSpan，则只会在第一个插件中创建 ExitSpan，后续调用的 ExitSpan.start() 方法并不会更新 startTime，其他的 set*() 方法也会做判断，只有 stackDepth 为1的时候，才会写入相应字段，也就是说，ExitSpan 中记录的信息是创建 ExitSpan 时填入的，与 EntrySpan 正好相反。

举个栗子，假如有一次通过 Http 方式进行的 Dubbo 调用，Dubbo A --> HttpClient --> Dubbo B，此时在 Dubbo A 的出口处，Dubbo 的插件会创建 ExitSpan 并调用 start() 方法，在 HttpClient 的出口处则只是再次调用了 start() 方法，该 ExitSpan 中记录的信息都是Dubbo A 出口处记录的。

一个 TraceSegment 可以有多个 ExitSpan，例如，Dubbo A 服务在处理一个请求时，会调用 Dubbo B 服务得到相应之后，紧接着调用了 Dubbo C 服务，这样，该 TraceSegment 就有了两个完全独立的 ExitSpan。

LocalSpan 表示的是一个本地方法调用，继承了 AbstractTracingSpan，没啥可说的，也不能递归，╮(╯_╰)╭。

行吧，Span 核心的内容就说到这里，继续往下看。

**TraceSegmentRef**

TraceSegment 通过 refs 集合记录父 TraceSegment 中的一个 Span，TraceSegmentRef 中的核心字段如下：

其中最重要的还是traceSegmentId 字段和spanId字段。

**总结**

好了，Trace 的基本概念以及 Skywalking 中的基础组建类，都大概介绍完了，主要就是：TraceSegment、ID、DistributedTraceId、Span、TraceSegmentRef。这些组建中的字段也并不复杂，都是为了确定从属关系( Span 属于 TraceSegment )以及父子关系( Span的父子关系、TraceSegment 的父子关系)。这些组建中的方法也都比较简单，基本都是getter/setter方法，没有超过10行的哈，easy，easy。

#### Trace Context 

从这一小节开始，我们将开始介绍 Trace Context 相关的组件。

**Context**

AbstractTracerContext 是 Skywalking 抽象链路上下文的顶层抽象类，在 一个线程中 Context 与 TraceSegment 一一对应，其中定义了链路上下文的的基本行为：

TracingContext 是 AbstractTracerContext 实现类，其核心方法如下：

下面开始分析 TracingContext 中的核心方法，首先是 createEntrySpan() 方法，它负责创建 EntrySpan，如果已经存在父 Span，当然没法再创建 EntrySpan咯，重新调用一下 start()方法咯，╮(╯_╰)╭，大致实现如下（其中省略了 DictionaryManager 的相关代码，后面再详细介绍这货）：

接下来看 createLocalSpan()方法，就是创建个 LocalSpan对象，然后加到 activeSpanStack 集合中，大致实现如下：

再往下自然就到了 createExitSpan() 方法，创建或是重用 ExitSpan，实现如下：

最后来看 stopSpan() 方法，它负责关闭指定的 Span对象：

在 TracingContext.finish() 方法中会关闭关联的 TraceSegment 对象，完成采样操作，还会通知相关的 Listener：

在开始介绍 inject() 方法和 extract() 方法之前，需要先介绍一下他们的参数——ContextCarrier，它记录了当前 TracingContext的一些基本信息，并实现了Serializable 接口哦，ContextCarrier 中的字段不再详细介绍，望文生义即可，嘎嘎嘎。

TracingContext.inject() 方法的功能就是将 TracingContext 对象填充到 ContextCarrier 中，看着很长，其实就是一些简单的字段，实现如下：

在 TracingContext 的 extract() 方法就是将ContextCarrier中各个字段，解出来放到 TraceSegmentRef 的相应字段中：

下面看一下 capture() 方法和 continued() 方法的参数—— ContextSnapshot，它与前面的 ContextCarrier 类似。因为只是跨线程传递，所以像 service_intance_id 这种字段就没必要传递了。capture() 方法和 continued() 方法的行为与 inject() 方法和 extract() 方法类似，这里不再展开分析了。

**ContextManager**

在前面介绍 ServiceManager SPI 加载 BootService 接口实现类的时候，我们看到有一个叫 ContextManager 的实现类，就是它来控制 Context 的。ContextManager 中的 prepare()、boot()、onComplete() 方法都是空实现。

ContextManager 的核心字段如下：

在 ContextManager 提供的 createEntrySpan()、createExitSpan() 以及 createLocalSpan()等方法中，都会调用其 getOrCreate() 这个 static 方法，在该方法中负责创建 TracingContext，如下所示：

之后会再调用TracingContext的相应方法，创建指定类型的 Span。

ContextManager 中的其他方法都是先调用 get() 这个静态方法拿到当前线程的 TracingContext，然后调用 TracingContext 的相应方法实现的，不展开了╮(╯_╰)╭。

**SamplingService**

前面的分析中多次看到 SamplingService，它负责进行采样，其核心字段如下：

在 SamplingService.boot()方法中会根据配置初始化上述字段：

在 SamplingService.trySampling()方法中会尝试增加 samplingFactorHolder 的值，当其值超过配置指定的值时，会返回false，表示该 Trace 未被采样到，具体实现如下：

**总结**

Span的Context记录分两种：

- ContextCarrier -- 用于跨进程传递上下文数据。
- ContextSnapshot -- 用于跨线程传递上下文数据。

好了，Context 相关的组件基本就介绍完了，其中包括 AbstractTracerContext 及其实现类、ContextManager，主要提供了如下方法：

创建 EntrySpan、LocalSpan、ExitSpan 三类Span 的方法。

关闭 Span 的 stopSpan() 方法。

用于跨进程传播的 inject()、extract() 方法，以及涉及到的 ContextCarrier 组件

用于跨线程传播的 capture()、continue()方法，以及涉及到的 ContextSnapshot组件。

#### DictionaryManager

上一节介绍了 Context 相关的组件，Skywalking Agent 会将 Context 中的数据发到 Skywalking 的服务端，其中有 operationName、peerHost、tag KV 等等一堆字符串，每次都传递一堆字符串会很浪费带宽的。常见的解决方案就是将字符串映射成数字，然后传输一组数字即可，Skywalking 也是这么搞得，涉及到的组件是 DictionaryManager，也会本节介绍的重点。

Skywalking 中有两个 DictionaryManager，一个是EndpointNameDictionary ，另一个NetworkAddressDictionary，注意，这俩货都通过枚举的方式实现了单例哈。

**EndpointNameDictionary**

本小节先来看EndpointNameDictionary，其中封装了两个集合：

OperationNameKey中包含四个字段，其 equals()方法会将这四个字段都考虑进去，如下所示：

EndpointNameDictionary中的核心方法是 find0() 方法，它负责查找指定的 OperationNameKey，查找成功就返回 Found，查找失败就记录到unRegisterEndpoints集合等待同步：

syncRemoteDictionary() 方法会定期将 unRegisterEndpoints集合同步到服务端，服务端会返回分配的映射id，并记录到endpointDictionary 集合：

这个 gRPC 服务定义在哪里呢？回到前面 skywalking-agent 的注册过程——Register.proto，之前只介绍了doServiceRegister和doServiceInstanceRegister，现在看其中的doEndpointRegister：

其中 Endpoints 参数的定义如下，与 OperationNameKey 中的字段同款：

返回值 EndpointMapping 的定义如下，其中的 endpointId 就是服务端为该 OperationName 分配的 id：

**NetworkAddressDictionary**

NetworkAddressDictionary 与 EndpointNameDictionary的功能类似，实现了 networkAddress 与数字之间的映射，其核心字段如下：

NetworkAddressDictionary 与服务端定期同步的方法是syncRemoteDictionary()方法，具体实现如下：

在前面介绍 skywalking-agent 初始化的时候，Agent 会定期向服务端发送心跳，在心跳发送完之后，就会调用 EndpointNameDictionary、NetworkAddressDictionary 的syncRemoteDictionary() 方法进行同步，看看代码ServiceAndEndpointRegisterClient 的153行和154行就知道了，嘎嘎嘎。

**PossibleFound**

最后，在EndpointNameDictionary、NetworkAddressDictionary 中提供的 find*()方法的返回值都是 PossibleFound 类型，它是一个抽象类，表示是否找到了对应的id：

PossibleFound 提供了 两个重载的 doInCondition()方法，根据查找结果执行不同的行为，直接上代码吧：

Found 接口以及 FoundAndObtain 接口在前面介绍 TracingContext 以及 StackBasedTracingSpan 中都有实现（只不过我给省略了，嘎嘎嘎），你可以自己翻一下，这几个接口的定义如下：

这里以 StackBasedTracingSpan.setPeer()方法的实现为例，简单看一下 Found 和 NotFound 接口的使用：

### 源码分析-Tomcat 插件

本节通过分析几个插件的实现，深入了解一下 ContextManager、TracingContext、TraceSegment 这些组建是怎么玩的，了解一下 skywalking-agent 中的数据流向是什么啥样的。首先是 Tomcat 插件，Skywalking 提供的 Tomcat 插件本身比较简单，但是要看懂这个插件的原理，需要对 Tomcat 本身的结构有一些了解。

**Tomcat 架构简析**

先来简单看一下的 Tomcat 的架构，如下图所示

Connector 组件是 Tomcat 中两个核心组件之一，它的主要任务是负责接收客户端发起的 TCP 连接请求（其实就是创建相应的 Request 和 Response 对象），而请求的处理则是由 Container 来负责的。

Container 是容器的父接口，所有子容器都必须实现这个接口，Container 容器的设计用的是典型的责任链模式，它有四个子容器组件构成，分别是：Engine、Host、Context、Wrapper，这四个组件不是平行的，而是父子关系，Engine 包含 Host，Host 包含 Context，Context 包含 Wrapper。通常一个 Servlet class 对应一个 Wrapper，如果有多个 Servlet 就可以定义多个 Wrapper，如果有多个 Wrapper 就要定义一个更高的 Container 了，如 Context。Context 还可以定义在父容器 Host 中，Host 不是必须的，但是要运行 war 程序，就必须要 Host，因为 war 中必有 web.xml 文件，这个文件的解析就需要 Host 了。如果要有多个 Host 就要定义一个顶级容器 Engine 了。而 Engine 没有父容器了，一个 Engine 代表一个完整的 Servlet 引擎。这些组件在 Tomcat 的 server.xml 文件中都能找到相应的配置，用过 Tomcat 的童鞋都知道，不解释了。

下面这张图大致展示了从 Connector 开始接收请求，然后请求一步步经过 Engine、Host、Context、Wrapper，最终 Servlet 的流程

其实容器的本质是一个 Pipeline，我们可以在这个Pipeline 上增加任意的 Valve，处理请求 Tomcat 线程会挨个执行这些 Valve 最终完成请求的处理，而且四个组件都会有自己的一套 Valve 集合，例如上图中的 StandEngineValve、StandHostValve、StandContextValve、StandWrapperValve，我们也可以在 Tomcat 的 server.xml 文件中自定义Valve（实际工作中只会撸业务代码，很少有人这么玩）。这些标准的 Valve 都是当前 Container 中最后一个 Valve，它们会负责将请求传给它们的子容器，以保证处理逻辑能继续向下执行。看下面这张图就比较明确了哈

接着来看四个级别的容器分别是干啥的。

**Engine**作为顶层容器，接口比较简单，它只定义了一些基本的关联关系，它可以添加 Host 类型的子容器，没啥可说的。

一个**Host**在 Engine 中代表一个虚拟主机，这个虚拟主机的作用就是运行多个应用，它负责安装和展开这些应用，并且标识这个应用以便能够区分它们。它的子容器通常是 Context，它除了关联子容器外，还有就是保存一个主机应该有的信息。

**Context**代表 Servlet 的 Context，它具备了 Servlet 运行的基本环境，理论上只要有 Context 就能运行 Servlet 了，也就是说 Tomcat 可以没有 Engine 和 Host。Context 最重要的功能就是管理它里面的 Servlet 实例，并和 Request 一起正确地找到处理请求的 Servlet。Servlet 实例在 Context 中是以 Wrapper 出现的。

**Wrapper**代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。Wrapper 是最底层的容器，它没有子容器了。

**Tomcat 插件分析**

来看tomcat-7.x-8.x-plugin 这个插件，这是 Skywalking 提供给 Tomcat 7 和 Tomcat 8的插件。在其 skywalking-plugin.def 文件中定义了TomcatInstrumentation 和 ApplicationDispatcherInstrumentation 插件类。

先看 TomcatInstrumentation.enhanceClass()方法，确定它拦截的是 Tomcat 中的哪个类呢？

StandardHostValve 是 Host容器中最后一个Valve，核心方法是 invoke() 方法，该方法中会通过 request 找到匹配的 Context 对象，并调用其 Pipeline 中的第一个 Valve处理请求，大致实现如下所示：

接着看 TomcatInstrumentation.getInstanceMethodsInterceptPoints()方法，它返回了两个 InstanceMethodsInterceptPoint 对象，一个拦截 invoke()方法，一个拦截 throwable()方法。

先来看拦截 invoke()方法的 TomcatInvokeInterceptor，它的 beforeMethod() 方法核心就是创建 EntrySpan，大致上线如下：

CarrierItem 有三个核心字段：

这样，CarrierItem既可以存储键值对，也可以串成一个链表，好吧，CarrierItem 还真实现了 Iterator 接口。

在 ContextCarrier.items() 方法中，会根据当前 Skywalking Agent 的配置创建一个CarrierItem链表：

从TomcatInvokeInterceptor.beforeMethod()方法的逻辑中可以看到，之后从 Http 请求的 Header 中获取对应的 value 值记录到对应 CarrierItem 中。setHeadValue() 方法会调用 ContextCarrier.deserialize() 方法解析该 value 值并初始化 ContextCarrier 中的各个字段，这里以 SW6CarrierItem 为例进行介绍（这里省略一些try/catch代码块和边界检查）：

这就填满了 ContextCarrier 的 8 个字段咯，╮(╯_╰)╭ 。

接下来看，ContextManager.createEntry() 方法的实现，前面说过其核心是调用 getOrCreate() 方法获取/创建当前 TracingContext 对象，然后调用 TracingContext.createEntry() 方法创建（或是重新 start ）当前 EntrySpan 对象，这里更详细的说一下一些实现细节吧。

ContextManager.createEntry() 方法首先会检测当前 ContextCarrier 是否合法，其实就是检查ContextCarrier的8个核心字段是否填充好了，如果合法，就证明是上游有 Trace 信息传递下来了：

行吧，TomcatInvokeInterceptor 的 beforeMethod()方法大概就是这样，它的 afterMethod()方法就简单很多了：

TracingContext.stopSpan() 方法的具体在前面已经详细分析过了，其中会调用StackBasedTracingSpan.finish() 方法尝试关闭当前 Span，这里会检测该 Span 的 operationId 字段，如果为空，则尝试再次通过 DictionaryManager组件用 operationName 换取 operationId，具体代码就不贴了。

接下来看tomcat-7.x-8.x-plugin 中的另一个插件类——ApplicationDispatcherInstrumentation，它拦截的是 Tomcat 的 ApplicationDispatcher.forward()方法以及ApplicationDispatcher 的全部构造方法。forward()方法主要处理 forward 跳转，写过 JSP 和 Servlet 程序的童鞋应该都知道 forward 和 redirect 的知识点，不展开说了。

ForwardInterceptor的实现比较简单，其onConstruct()方法如下：

beforeMethod()方法和 afterMethod()方法的实现大致如下：

到这里，tomcat-7.x-8.x-plugin 插件的具体实现就分析完了。

### 源码分析-**Dubbo 插件分析**

要搞清楚 Skywalking 提供的 Dubbo 插件的工作原理，需要先了解一下 Dubbo 中的 Filter 机制。Filter 在很多框架中都有使用过这个概念，基本上的作用都是类似的，在请求处理前后做一些通用的逻辑，而且Filter可以有多个，支持层层嵌套。

Dubbo 的Filter 概念基本上符合我们正常的预期理解，而且 Dubbo 官方针对 Filter 做了很多的原生支持，包括我们熟知的 RpcContext、accesslog、monitor 功能都是通过 Filter 来实现的。Filter 也是 Dubbo 用来实现功能扩展的重要机制，我们可以通过自定义 Filter 实现、启停指定 Filter 来改变 Dubbo 的行为来实现需求。

行吧，简单看一下Dubbo Filter 相关的知识点。首选是构造DubboFilter Chain的入口是在 ProtocolFilterWrapper.buildInvokerChain() 方法处，它将加载到的 Dubbo Filter 实例串成一个 Chain（这里）：

这里的核心有两步，上面明显能看出来的是将 Filter 实例串成 Chain，另一个核心步骤就是通过ExtensionLoader 加载 Filter 对象，原理是SPI，但是 Dubbo 的 SPI 实现有点优化，但是原理和思想基本一样，看一下实现吧：

介绍完 Dubbo Filter 的原理之后，我们来看 MonitorFilter 实现，它用于记录一些监控信息，来看看它的 invoke() 方法的实现：

有个地方记录开始时间、增加并发量，必然有个地方计算请求耗时、减掉并发量，在哪里呢？在 MonitorListener 里面，它是 MonitorFilter 配套的 Listener，其 invoke() 方法实现如下：

在 collect()方法里面，会将监控信息整理成 URL 并缓存起来：

DubboMonitor 实现了 Monitor 接口，其中有个 Map用于缓存 URL，然后在其构造方法中会启动一个定时任务，定时发送 URL：

在 DubboMonitor.collect()方法中会将相同的 URL 中的监控值累加，然后形成一个新的 URL 填充回statisticsMap 集合，这里就不展开介绍了。

行了，Dubbo MonitorFilter 的基础知识介绍完了，开始 Skywalking Dubbo 插件的分析吧（终于开始正题了）。apm-dubbo-2.7.x-plugin 插件的 skywalking-plugin.def 中定义的插件是 DubboInstrumentation，它拦截的是 MonitorFilter.invoke()方法，具体的 Interceptor 实现是 DubboInterceptor，在其 beforeMethod() 方法中会根据当前 MonitorFilter 所在的服务角色（Consumer/Provider）创建对应的 Span（ExitSpan/EntrySpan），具体实现如下：

在 ContextManager.createExitSpan()方法中除了创建 ExitSpan 之外，还会调用 inject() 方法将 Trace 信息记录到CarrierContext 中，这样后面通过CarrierItem 持久化的时候才有值。

DubboInterceptor.afterMethod() 方法实现比较简单，有异常就是通过 log 方式记录到当前 Span，最后尝试关闭当前 Span。

### 源码分析-DataCarrier 异步处理库

DataCarrier 是一个轻量级的生产者-消费者模式的实现库， Skywalking Agent 在收集到 Trace 数据之后，会先将 Trace 数据写入到 DataCarrier 中缓存，然后由后台线程定时发送到 Skywalking 服务端。呦吼，和前面介绍的 Dubbo MonitorFilter 的实现原理类似啊，emmm，其实多数涉及到大量网络传输的场景都会这么设计：先本地缓存聚合，然后定时发送。

DataCarrier 实现在 Skywalking 中是一个单独的模块

首先来看 Buffer 这个类，它是一个环形缓冲区，是整个 DataCarrier 最底层的一个类，其核心字段如下：

这里的 AtomicRangeInteger 是环形指针，我们可以指定其中的 value字段（AtomicInteger类型）从 start值（int类型）开始递增，当 value 递增到 end值（int类型）时，value字段会被重置为 start值，从而实现环形指正的效果。

Buffer.save() 方法负责向当前 Buffer 缓冲区中填充数据，实现如下：

**个人感觉：BLOCKING策略下，Buffer 在写满之后，index 发生循环，****可能会出现两个线程同时等待写入一个位置的情况，有并发问题啊，会丢数据~~~看官大佬们可以自己思考一下~~~**

生产者用的是 save() 方法，消费者用的是 Buffer.obtain() 方法：

Channels 是另一个比较基础的类，它底层封装了多个 Buffer 实例以及一个分区选择器，字段如下所示：

IDataPartitioner 提供了类似于 Kafka 的分区功能，当数据并行写入的时候，由分区选择器定将数据写入哪个分区，这样就可以有效降低并发导致的锁(或CAS)竞争，降低写入压力，可提高效率。IDataPartitioner 接口有两个实现，如下下图所示：

ProducerThreadPartitioner会根据写入线程的ID进行选择，这样可以保证相同的线程号写入的数据都在一个分区中。

SimpleRollingPartitioner简单循环自增选择器，使用无锁整型（volatile修饰）的自增，顺序选择线程号，在高负载时，会产生批量连续写入一个分区的效果，在中等负载情况下，提供较好性能。

下面来看 IConsumer 接口， DataCarrier 的消费者逻辑需要封装在其中。IConsumer 接口定义了消费者的基本行为：

后面即将介绍的TraceSegmentServiceClient 类就实现了 IConsumer 接口，后面展开说。

ConsumerThread 继承了Thread，表示的是消费者线程，其核心字段如下：

ConsumerThread.run() 方法会循环 consume() 方法：

ConsumerThread.consume() 方法是消费的核心：

另一个消费者线程是 MultipleChannelsConsumer，与 ConsumerThread 的区别在于:

ConsumerThread 处理的是一个确定的 IConsumer 消费一个 Channel 中的多个 Buffer。

MultipleChannelsConsumer 可以处理多组 Group，每个 Group 都是一个IConsumer+一个 Channels。

先来看 MultipleChannelsConsumer 的核心字段：

MultipleChannelsConsumer.consume()方法消费的依然是单个IConsumer，这里不再展开分析。在MultipleChannelsConsumer.run()方法中会循环每个 Group：

在MultipleChannelsConsumer.addNewTarget() 方法中会添加新的 Group。这里用了 Copy-on-Write，因为在添加的过程中，MultipleChannelsConsumer线程可能正在循环处理 consumeTargets 集合（这也是为什么consumeTargets用 volatile 修饰的原因）：

IDriver 负责将 ConsumerThread 和 IConsumer 集成到一起，其实现类如下：

先来看 ConsumeDriver，核心字段如下：

在 ConsumeDriver 的构造方法中，会初始化上述两个集合：

完成初始化之后，要调用 ConsumeDriver.begin() 方法，其中会根据 Buffer 的数量以及ConsumerThread 线程数进行分配：

如果 Buffer 个数较多，则一个 ConsumerThread 线程需要处理多个 Buffer。

如果 ConsumerThread 线程数较多，则一个 Buffer 被多个 ConsumerThread 线程处理（每个 ConsumerThread 线程负责消费这个 Buffer 的一段）。

如果两者数量正好相同，则是一对一关系。

逻辑虽然很简单，但是具体的分配代码比较长，这里就不粘贴了。

BulkConsumePool 是 IDriver 接口的另一个实现，其中的 allConsumers 字段（List类型）记录了当前启动的MultipleChannelsConsumer线程，在BulkConsumePool 的构造方法中会根据配置初始化该集合：

BulkConsumePool.add() 方法提供了添加新 Group的功能：

最后来看 DataCarrier ，它是整个 DataCarrier 库的入口和门面，其核心字段如下：

在 DataCarrier 的构造方法中会初始化 Channels 对象，默认使用 SimpleRollingPartitioner 以及 BLOCKING 策略，太简单，不展开了。

DataCarrier.produce() 方法实际上是调用 Channels.save()方法，实现数据写入，太简单，不展开了。

在 DataCarrier.consume()方法中，会初始化 ConsumeDriver 对象并调用 begin() 方法，启动 ConsumeThread 线程：

### 源码分析-发送Trace

考虑到减少**外部组件**的依赖，Agent 收集到 Trace 数据后，不是写入外部消息队列( 例如，Kafka )或者日志文件，而是 Agent 写入**内存消息队列**，**后台线程**【**异步**】发送给 Collector 。

我们先来看看 TraceSegmentServiceClient 的**属性**：

- `TIMEOUT` **静态**属性，发送等待超时时长，单位：毫秒。
- `lastLogTime` 属性，最后打印日志时间。该属性主要用于开发调试。
- `segmentUplinkedCounter` 属性，TraceSegment 发送数量。
- `segmentAbandonedCounter` 属性，TraceSegment 被丢弃数量。在 Agent 未连接上 Collector 时，产生的 TraceSegment 将被丢弃。
- `carrier` 属性，内存队列。
- `serviceStub` 属性，**非阻塞** Stub 。
- `status` 属性，连接状态。

**TracingContextListener**

这里先帮助看官老爷回顾一个点，TracingContext.finish() 方法在关闭 TraceSegment 的时候，会调用下面这行代码：

在 ListenerManager 中记录了所有 TracingContextListener 监听器：

在ListenerManager.notify() 方法中会遍历所有TracingContextListener监听器，通知他们该 TraceSegment 将会被关闭了：

**TraceSegmentServiceClient**

TraceSegmentServiceClient 实现了下面四个接口：

BootService

IConsumer

TracingContextListener

GRPCChannelListener

先来看看它的核心字段：

再来看TraceSegmentReportService 服务的 proto定义：

prepare() 方法会将当前TraceSegmentServiceClient对象作为 Listener 注册到 GRPCChannelManager 上，监听链接状态。当链接状态发生变化时，会通过 statusChanged() 方法（这是对GRPCChannelListener接口的实现）修改 status 字段值。

boot() 方法中会初始化上述核心字段，其中会创建 DataCarrier 对象，并调用其 consume() 方法启动 ConsumerThread 线程：

onComplete() 方法会将当前TraceSegmentServiceClient对象作为 Listener 注册到TracingContext.LISTENERS 集合中，easy，不展开说了。

再来看TraceSegmentServiceClient对 TracingContextListener 接口的实现，其 afterFinished() 方法会调用 DataCarrier.produce() 方法，将 TraceSegment 写入 DataCarrier 缓冲区暂存。

再来看 TraceSegmentServiceClient 对 IConsumer 接口的实现，在其 consume() 方法中会将 TraceSegment（从DataCarrier中读取到的）通过 gRPC 的方式发送到服务端，具体实现如下：

简单看一下GRPCStreamServiceStatus 的实现，其中封装了一个 volatile boolean 字段，来表示发送状态：

还提供了 wait4Finish() 方法用来等待整个发送过程结束：

最后，GRPCStreamServiceStatus.finish()方法会修改 status 的值为true，这样， wait4Finish() 方法也可以退出。

###### 发送Trace

XXXSegmentServiceClient有一个消费线程，不停的拉取缓冲队列中的数据，

- 如果是TraceSegmentServiceClient，则通过gRPC发送给后端OAP
- 如果是KafkaTraceSegmentServiceClient，则丢到Kafka中。

## Skywalking OAP源码

#### OAP流程

启动了OAP。Agent和OAP之间是通过gRPC来发送链路信息的。Agent端维护了一个队列（默认5个channel，每个channel大小为300）和一个线程池（默认1个线程，后面称为发送线程），链路数据采集后主线程（即业务线程）会写入这个队列，如果队列满了，主线程会直接把把数据丢掉（丢的时候会以debug级别打印日志）。发送线程会从队列取数据通过gRPC发送给后端OAP，OAP经过处理后写入存储。

![Skywalking-OAP](F:/work/openGuide/Middleware/Skywalking-OAP.png)



#### OAPServer启动

由server-starter和server-starter-es7调用server-bootstrap
server-starter和server-starter-es7的区别在于maven中引入的存储模块Module不同

| 启动器             | 插件                          |
| ------------------ | ----------------------------- |
| server-starter     | storage-elasticsearch-plugin  |
| server-starter-es7 | storage-elasticsearch7-plugin |

ModuleDefine与maven模块关系

- ModuleDefine一般对应一个模块
- ModuleProvider一般对应一个模块的子模块
- 比如配置模块,server-configuration下多个模块对应configuration模块的不同provider实现

| ModuleDefine名称    | application.yml 名称 | maven模块         |
| ------------------- | -------------------- | ----------------- |
| ConfigurationModule | configuration: none: | configuration-api |

类名设计

| 名称                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| ApplicationConfigLoader  | 负责加载application.yml                                      |
| ApplicationConfiguration | 将application.yml转化为ApplicationConfiguration              |
| ModuleManager            | 管理所有的Module                                             |
| ModuleDefine             | 一个ModuleDefine代表一个模块,比如存储模块,UI查询query模块,JVM模块等等;不同的模块相互依赖或者不依赖构建整个oapServer功能 |
| ModuleProvider           | ModuleDefine的具体实现,比如StorageModule包含es存储实现,mysql实现等等 |
| Service                  | 多个service构成一个完整的ModuleProvider,也就是将module的具体实现拆分成多个serviceImpl |

启动架构图
两个starter模块依赖es7或者低版本es,启动时根据es版本决定启动who
调用server-bootstrap启动
解析application.yml生成ApplicationConfiguration
ModuleManager根据ApplicationConfiguration加载所有的ModuleDefine以及对应的loadedModuleProvider
执行prepare,start,notifyAfterCompleted完成所有模块的启动

OAPServerBootstrap

```
public class OAPServerBootstrap {
    public static void start() {
    // 初始化mode为init或者no-init,表示是否初始化[example:底层存储组件等]
        String mode = System.getProperty("mode");
        RunningMode.setMode(mode);
      // 初始化ApplicationConfigurationd的加载器和Module的加载管理器
        ApplicationConfigLoader configLoader = new ApplicationConfigLoader();
        ModuleManager manager = new ModuleManager();
        try {
        // 加载yml生成ApplicationConfiguration配置
        
            ApplicationConfiguration applicationConfiguration = configLoader.load();
            
            // 初始化模块 通过spi获取所有Module实现,基于yml配置加载spi中存在的相关实现
            manager.init(applicationConfiguration);

            manager.find(TelemetryModule.NAME)
                   .provider()
                   .getService(MetricsCreator.class)
                   .createGauge("uptime", "oap server start up time", MetricsTag.EMPTY_KEY, MetricsTag.EMPTY_VALUE)
                   // Set uptime to second
                   .setValue(System.currentTimeMillis() / 1000d);

            if (RunningMode.isInitMode()) {
                log.info("OAP starts up in init mode successfully, exit now...");
                System.exit(0);
            }
        } catch (Throwable t) {
            log.error(t.getMessage(), t);
            System.exit(1);
        }
    }
}
```

#### 源码分析-OAP数据接收

启动时CoreModuleProvider核心provider会初始化GRPCServer, 只留下GRPCServer相关的核心代码

```
# CoreModuleProvider
public void prepare() {
    if (moduleConfig.isGRPCSslEnabled()) {
        grpcServer = new GRPCServer(moduleConfig.getGRPCHost(), moduleConfig.getGRPCPort(),
                                    moduleConfig.getGRPCSslCertChainPath(),
                                    moduleConfig.getGRPCSslKeyPath()
        );
    } else {
        grpcServer = new GRPCServer(moduleConfig.getGRPCHost(), moduleConfig.getGRPCPort());
    }
    if (moduleConfig.getMaxConcurrentCallsPerConnection() > 0) {
        grpcServer.setMaxConcurrentCallsPerConnection(moduleConfig.getMaxConcurrentCallsPerConnection());
    }
    if (moduleConfig.getMaxMessageSize() > 0) {
        grpcServer.setMaxMessageSize(moduleConfig.getMaxMessageSize());
    }
    if (moduleConfig.getGRPCThreadPoolQueueSize() > 0) {
        grpcServer.setThreadPoolQueueSize(moduleConfig.getGRPCThreadPoolQueueSize());
    }
    if (moduleConfig.getGRPCThreadPoolSize() > 0) {
        grpcServer.setThreadPoolSize(moduleConfig.getGRPCThreadPoolSize());
    }
    grpcServer.initialize();

}
```

GRPCServer 用于接收 SkyWalking Agent 发送的 gRPC 请求，SkyWalking Agent 会切分OAP服务列表配置项，得到 OAP 服务列表，然后从其中随机选择一个 OAP 服务创建长连接，实现后续的数据上报

GRPCServer 处理 gRPC 请求的逻辑，封装在了 ServerHandler 实现之中的。我们可以通过两者的 addHandler() 方法，为指定请求添加相应的 ServerHandler 实现, 比如常见的TraceSegmentReportServiceHandler上报trace数据，就是利用对应的handler处理的

##### 默认GRPC接收

TraceSegmentReportServiceHandler接收的agent的方法，只留核心的代码

**TraceSegmentReportServiceHandler**

```
    @Override
    public StreamObserver<SegmentObject> collect(StreamObserver<Commands> responseObserver) {
        return new StreamObserver<SegmentObject>() {
            @Override
            public void onNext(SegmentObject segment) {
                if (log.isDebugEnabled()) {
                    log.debug("received segment in streaming");
                }

                HistogramMetrics.Timer timer = histogram.createTimer();
                try {
                    segmentParserService.send(segment);
                } catch (Exception e) {
                    errorCounter.inc();
                    log.error(e.getMessage(), e);
                } finally {
                    timer.finish();
                }
            }

            @Override
            public void onError(Throwable throwable) {
                log.error(throwable.getMessage(), throwable);
                responseObserver.onCompleted();
            }

            @Override
            public void onCompleted() {
                responseObserver.onNext(Commands.newBuilder().build());
                responseObserver.onCompleted();
            }
        };
    }
```

通过GRPC接受发送过来的数据，调用 `ISegmentParserService#send#send(UpstreamSegment)` 方法，处理**一条** TraceSegment 。

**SegmentParserServiceImpl**`

```
@RequiredArgsConstructor
public class SegmentParserServiceImpl implements ISegmentParserService {
    private final ModuleManager moduleManager;
    private final AnalyzerModuleConfig config;
    @Setter
    private SegmentParserListenerManager listenerManager;

    @Override
    public void send(SegmentObject segment) {
        final TraceAnalyzer traceAnalyzer = new TraceAnalyzer(moduleManager, listenerManager, config);
        traceAnalyzer.doAnalysis(segment);
    }
}
```

TraceAnalyzer中doAnalysis方法: 各个listen是构建存储model，notifyListenerToBuild才触发存储，比如es

```
public void doAnalysis(SegmentObject segmentObject) {
    if (segmentObject.getSpansList().size() == 0) {
        return;
    }
    // 创建span的listener
    createSpanListeners();
    // 通知segment监听，构建存储model
    notifySegmentListener(segmentObject);
    // 通知 first exit local的各个listen，构建存储model
    segmentObject.getSpansList().forEach(spanObject -> {
        if (spanObject.getSpanId() == 0) {
            notifyFirstListener(spanObject, segmentObject);
        }

        if (SpanType.Exit.equals(spanObject.getSpanType())) {
            notifyExitListener(spanObject, segmentObject);
        } else if (SpanType.Entry.equals(spanObject.getSpanType())) {
            notifyEntryListener(spanObject, segmentObject);
        } else if (SpanType.Local.equals(spanObject.getSpanType())) {
            notifyLocalListener(spanObject, segmentObject);
        } else {
            log.error("span type value was unexpected, span type name: {}", spanObject.getSpanType()
                                                                                      .name());
        }
    });
    // 触发存储动作
    notifyListenerToBuild();
}

private void notifyListenerToBuild() {
    analysisListeners.forEach(AnalysisListener::build);
}
```

SegmentAnalysisListener.build方法存储

```
// 去掉非核心代码
public void build() {
    segment.setEndpointId(endpointId);
    segment.setEndpointName(endpointName);
    // 存储核心
    sourceReceiver.receive(segment);
}
```

SourceReceiverImpl.receive方法

```
public void receive(ISource source) {
     dispatcherManager.forward(source);
}
```

DispatcherManager.forward方法，只留下核心代码

```
public void forward(ISource source) {
    source.prepare();
    for (SourceDispatcher dispatcher : dispatchers) {
        dispatcher.dispatch(source);
    }   
}
```

SegmentDispatcher.dispatch方法，存储数据，比如es

```
public void dispatch(Segment source) {
    SegmentRecord segment = new SegmentRecord();
    segment.setSegmentId(source.getSegmentId());
    segment.setTraceId(source.getTraceId());
    segment.setServiceId(source.getServiceId());
    segment.setServiceInstanceId(source.getServiceInstanceId());
    segment.setEndpointName(source.getEndpointName());
    segment.setEndpointId(source.getEndpointId());
    segment.setStartTime(source.getStartTime());
    segment.setEndTime(source.getEndTime());
    segment.setLatency(source.getLatency());
    segment.setIsError(source.getIsError());
    segment.setDataBinary(source.getDataBinary());
    segment.setTimeBucket(source.getTimeBucket());
    segment.setVersion(source.getVersion());
    segment.setTagsRawData(source.getTags());
    segment.setTags(Tag.Util.toStringList(source.getTags()));

    RecordStreamProcessor.getInstance().in(segment);
}
```

最终会调RecordPersistentWorker.in存储

```
public void in(Record record) {
    InsertRequest insertRequest = recordDAO.prepareBatchInsert(model, record);
    batchDAO.asynchronous(insertRequest);    
}
```

最后到es存储BatchProcessEsDAO, 调es的bulk

```
public void asynchronous(InsertRequest insertRequest) {
    if (bulkProcessor == null) {
        this.bulkProcessor = getClient().createBulkProcessor(bulkActions, flushInterval, concurrentRequests);
    }

    this.bulkProcessor.add((IndexRequest) insertRequest);
}
```



#### TraceSegmentServiceHandler

调用 `ITraceSegmentService#send(UpstreamSegment)` 方法，处理**一条** TraceSegment 。

## TraceSegmentService

`org.skywalking.apm.collector.agent.stream.service.trace.ITraceSegmentService` ，继承 [Service](https://github.com/YunaiV/skywalking/blob/40823179d7228207b06b603b9a1c09dfc4f78593/apm-collector/apm-collector-core/src/main/java/org/skywalking/apm/collector/core/module/Service.java) 接口，TraceSegment 服务接口。

- 定义了 [`#send(UpstreamSegment)`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-define/src/main/java/org/skywalking/apm/collector/agent/stream/service/trace/ITraceSegmentService.java#L31) **接口**方法，处理**一条** TraceSegment 。

`org.skywalking.apm.collector.agent.stream.worker.trace.ApplicationIDService` ，实现 IApplicationIDService 接口，TraceSegment 服务实现类。

- 实现了 [`#send(UpstreamSegment)`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/TraceSegmentService.java#L39) 方法，代码如下：
  - 第 40 至 41 行：创建 SegmentParse 对象，后调用 `SegmentParse#parse(UpstreamSegment, Source)` 方法，解析并处理 TraceSegment 。

## SegmentParse

[`org.skywalking.apm.collector.agent.stream.parser.SegmentParse`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java) ，Segment 解析器。属性如下：

- `spanListeners` 属性，Span 监听器集合。**通过不同的监听器，对 TraceSegment 进行构建，生成不同的数据**。在 [`#SegmentParse(ModuleManager)` 构造方法](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L72) ，会看到它的初始化。
- `segmentId` 属性，TraceSegment 编号，即 [`TraceSegment.traceSegmentId`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-sniffer/apm-agent-core/src/main/java/org/skywalking/apm/agent/core/context/trace/TraceSegment.java#L47) 。
- `timeBucket` 属性，第一个 Span 的开始时间。

[`#parse(UpstreamSegment, Source)`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L86) 方法，解析并处理 TraceSegment 。在该方法里，我们会看到，本文开头提到的【**构造**】。整个构造的过程，实际分成**两步**：1）预构建；2）执行构建。代码如下：

- 第 88 至 89 行：从

   

  ```
  segment
  ```

   

  参数中，解析出 ：

  - `traceIds` ，关联的链路追踪**全局编号**。
  - `segmentObject` ，TraceSegmentObject 对象。

- 第 91 行：创建 SegmentDecorator 对象。该对象的用途，在 [「2.3 Standardization 标准化」](https://www.iocoder.cn/SkyWalking/collector-receive-trace/#) 统一解析。

- ——– 构建失败 ——–

- 第 94 行：调用 `#preBuild(List<UniqueId>, SegmentDecorator)` 方法，**预构建**。

- 第 97 至 99 行：调用

   

  ```
  #writeToBufferFile()
  ```

   

  方法，将 TraceSegment 写入 Buffer 文件

  暂存

  。为什么会判断

   

  ```
  source == Source.Agent
  ```

   

  呢？

  ```
  #parse(UpstreamSegment, Source)
  ```

   

  方法的调用，共有

  两个

   

  Source

   

  ：

  - 目前我们看到 TraceSegmentService 的调用使用的是 `Source.Agent` 。
  - 而后台线程，定时调用该方法重新构建使用的是 `Source.Buffer` ，如果不加盖判断，会预构建失败**重复**写入。

- 第 100 行：返回 `false` ，表示构建失败。

- ——– 构建成功 ——–

- 第 106 行：调用 [`#notifyListenerToBuild()`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L199) 方法，通知 Span 监听器们，**执行构建**各自的数据。在 [《SkyWalking 源码解析 —— Collector 存储 Trace 数据》](http://www.iocoder.cn/SkyWalking/collector-store-trace/?self) 详细解析。

- 第 109 行：调用 [`buildSegment(id, dataBinary)`](https://github.com/YunaiV/skywalking/blob/c15cf5e1356c7b44a23f2146b8209ab78c2009ac/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L177) 方法，**执行构建** TraceSegment 。

- 第 110 行：返回 `true` ，表示构建成功。

- 第 112 至 115 行：发生 InvalidProtocolBufferException 异常，返回 `false` ，表示构建失败。

# [Collector 存储 Trace 数据](https://www.iocoder.cn/SkyWalking/collector-store-trace/)

**本文主要基于 SkyWalking 3.2.6 正式版**

- [1. 概述](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [2. SpanListener](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [3. GlobalTrace](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [4. InstPerformance](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [5. SegmentCost](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [6. NodeComponent](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [7. NodeMapping](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [8. NodeReference](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [9. ServiceEntry](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [10. ServiceReference](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [11. Segment](http://www.iocoder.cn/SkyWalking/collector-store-trace/)
- [666. 彩蛋](http://www.iocoder.cn/SkyWalking/collector-store-trace/)

------

------

# 1. 概述

分布式链路追踪系统，链路的追踪大体流程如下：

1. Agent 收集 Trace 数据。
2. Agent 发送 Trace 数据给 Collector 。
3. Collector 接收 Trace 数据。
4. **Collector 存储 Trace 数据到存储器，例如，数据库**。

本文主要分享【第四部分】 **SkyWalking Collector 存储 Trace 数据**。

> 友情提示：Collector 接收到 TraceSegment 的数据，对应的类是 Protobuf 生成的。考虑到更加易读易懂，本文使用 TraceSegment 相关的**原始类**。

Collector 在接收到 Trace 数据后，经过**流式处理**，最终**存储**到存储器。如下图，**红圈部分**，为本文分享的内容：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/01.jpeg)

# 2. SpanListener

在 [《SkyWalking 源码分析 —— Collector 接收 Trace 数据》](http://www.iocoder.cn/SkyWalking/collector-receive-trace/) 一文中，我们看到 [`SegmentParse#parse(UpstreamSegment, Source)`](https://github.com/YunaiV/skywalking/blob/428190e783d887c8240546f321e76e0a6b5f5d18/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L86) 方法中：

- 在 [`#preBuild(List, SegmentDecorator)`](https://github.com/YunaiV/skywalking/blob/428190e783d887c8240546f321e76e0a6b5f5d18/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L118) 方法中，预构建的过程中，使用 Span 监听器们，从 TraceSegment 解析出不同的数据。
- 在**预构建**成功后，通知 Span 监听器们，去构建各自的数据，经过**流式处理**，最终**存储**到存储器。

[`org.skywalking.apm.collector.agent.stream.parser.SpanListener`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SpanListener.java) ，Span 监听器**接口**。

- 定义了 [`#build()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SpanListener.java#L28) 方法，构建数据，执行流式处理，最终存储到存储器。

SpanListener 的子类如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/02.png)

- 第一层，通用接口层，定义了从 TraceSegment 解析数据的方法。
  - ① [GlobalTraceSpanListener](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/GlobalTraceIdsListener.java) ：解析链路追踪全局编号数组( `TraceSegment.relatedGlobalTraces` )。
  - ② [RefsListener](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/RefsListener.java) ：解析父 Segment 指向数组( `TraceSegment.refs` )。
  - ③ [FirstSpanListener](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/FirstSpanListener.java) ：解析**第一个** Span (`TraceSegment.spans[0]`) 。
  - ③ [EntrySpanListener](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/EntrySpanListener.java) ：解析 EntrySpan (`TraceSegment.spans`)。
  - ③ [LocalSpanListener](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/LocalSpanListener.java) ：解析 LocalSpan (`TraceSegment.spans`)。
  - ③ [ExitSpanListener](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/ExitSpanListener.java) ：解析 ExitSpan (`TraceSegment.spans`)。
- 第二层，业务实现层，每个实现类对应一个数据实体类，一个 Graph 对象。如下图所示：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/03.png)

下面，我们以每个数据实体类为中心，逐个分享。

# 3. GlobalTrace

[`org.skywalking.apm.collector.storage.table.global.GlobalTrace`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/global/GlobalTrace.java) ，全局链路追踪，记录一次分布式链路追踪，包括的 TraceSegment 编号。

- GlobalTrace : TraceSegment = N : M ，一个 GlobalTrace 可以有多个 TraceSegment ，一个 TraceSegment 可以关联多个 GlobalTrace 。参见 [《SkyWalking 源码分析 —— Agent 收集 Trace 数据》「2. Trace」](http://www.iocoder.cn/SkyWalking/agent-collect-trace/?self) 。

- `org.skywalking.apm.collector.storage.table.global.GlobalTraceTable`

   

  ， GlobalTrace 表(

   

  ```
  global_trace
  ```

   

  )。字段如下：

  - `global_trace_id` ：全局链路追踪编号。
  - `segment_id` ：TraceSegment 链路编号。
  - `time_bucket` ：时间。

- [`org.skywalking.apm.collector.storage.es.dao.GlobalTraceEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/GlobalTraceEsPersistenceDAO.java) ，GlobalTrace 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/04.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.global.GlobalTraceSpanListener`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTraceSpanListener.java) ，**GlobalTrace 的 SpanListener** ，实现了 FirstSpanListener 、GlobalTraceIdsListener 接口，代码如下：

- `globalTraceIds` 属性，全局链路追踪编号**数组**。

- `segmentId` 属性，TraceSegment 链路编号。

- `timeBucket` 属性，时间。

- [`#parseFirst(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTraceSpanListener.java#L60) 方法，从 Span 中解析到 `segmentId` ，`timeBucket` 。

- [`#parseGlobalTraceId(UniqueId)`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTraceSpanListener.java#L66) 方法，解析全局链路追踪编号，添加到 `globalTraceIds` 数组。

- `#build()`

   

  方法，构建，代码如下：

  - 第 84 行：获取 GlobalTrace 对应的 `Graph<GlobalTrace>` 对象。
  - 第 86 至 92 行：循环 `globalTraceIds` 数组，创建 GlobalTrace 对象，逐个调用 `Graph#start(application)` 方法，进行流式处理。在这过程中，会保存 GlobalTrace 到存储器。

------

在 [`TraceStreamGraph#createGlobalTraceGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L93) 方法中，我们可以看到 GlobalTrace 对应的 `Graph<GlobalTrace>` 对象的创建。

- [`org.skywalking.apm.collector.agent.stream.worker.trace.global.GlobalTracePersistenceWorker`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTracePersistenceWorker.java) ，继承 PersistenceWorker 抽象类，GlobalTrace **批量**保存 Worker 。
  - [Factory](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTracePersistenceWorker.java#L51) 内部类，实现 AbstractLocalAsyncWorkerProvider 抽象类，在 [《SkyWalking 源码分析 —— Collector Streaming Computing 流式处理（一）》「3.2.2 AbstractLocalAsyncWorker」](http://www.iocoder.cn/SkyWalking/collector-streaming-first/?self) 有详细解析。
  - **PersistenceWorker** ，在 [《SkyWalking 源码分析 —— Collector Streaming Computing 流式处理（二）》「4. PersistenceWorker」](http://www.iocoder.cn/SkyWalking/collector-streaming-second/?self) 有详细解析。
  - [`#id()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTracePersistenceWorker.java#L39) 实现方法，返回 120 。
  - [`#needMergeDBData()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/global/GlobalTracePersistenceWorker.java#L43) 实现方法，返回 `false` ，存储时，不需要合并数据。GlobalTrace 只有新增操作，没有更新操作，因此无需合并数据。

# 4. InstPerformance

> 旁白君：InstPerformance 和 GlobalTrace 整体比较相似，分享的会比较简洁一些。

[`org.skywalking.apm.collector.storage.table.instance.InstPerformance`](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/instance/InstPerformance.java) ，应用实例性能，记录应用实例**每秒**的请求总次数，请求总时长。

- `org.skywalking.apm.collector.storage.table.instance.InstPerformanceTable`

   

  ， GlobalTrace 表(

   

  ```
  global_trace
  ```

   

  )。字段如下：

  - `application_id` ：应用编号。
  - `instance_id` ：应用实例编号。
  - `calls` ：调用总次数。
  - `cost_total` ：消耗总时长。
  - `time_bucket` ：时间。

- [`org.skywalking.apm.collector.storage.es.dao.InstPerformanceEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/InstPerformanceEsPersistenceDAO.java) ，InstPerformance 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/05.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.instance.InstPerformanceSpanListener`](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/instance/InstPerformanceSpanListener.java) ，**InstPerformance 的 SpanListener** ，实现了 FirstSpanListener 、EntrySpanListener 接口。

------

在 [`TraceStreamGraph#createInstPerformanceGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L101) 方法中，我们可以看到 InstPerformance 对应的 `Graph<InstPerformance>` 对象的创建。

- [`org.skywalking.apm.collector.agent.stream.worker.trace.instance.InstPerformancePersistenceWorker`](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/instance/InstPerformancePersistenceWorker.java) ，继承 PersistenceWorker 抽象类，InstPerformance **批量**保存 Worker 。
  - 类似 GlobalTracePersistenceWorker ，… 省略其它类和方法。
  - [`#needMergeDBData()`](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/instance/InstPerformancePersistenceWorker.java#L46) 实现方法，返回 `true` ，存储时，需要合并数据。`calls` 、`cost_total` 需要累加合并。

# 5. SegmentCost

> 旁白君：SegmentCost 和 GlobalTrace 整体比较相似，分享的会比较简洁一些。

[`org.skywalking.apm.collector.storage.table.segment.SegmentCost`](https://github.com/YunaiV/skywalking/blob/b8be916ac2bf7817544d6ff77a95a624bcc3efe6/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/segment/SegmentCost.java) ，TraceSegment 消耗时长，记录 TraceSegment 开始时间，结束时间，花费时长等等。

- SegmentCost : TraceSegment = 1 : 1 。

- `org.skywalking.apm.collector.storage.table.instance.SegmentCostTable`

   

  ， SegmentCostTable 表(

   

  ```
  segment_cost
  ```

   

  )。字段如下：

  - `segment_id` ：TraceSegment 编号。
  - `application_id` ：应用编号。
  - `start_time` ：开始时间。
  - `end_time` ：结束时间。
  - `service_name` ：操作名。
  - `cost` ：消耗时长。
  - `time_bucket` ：时间( `yyyyMMddHHmm` )。

- [`org.skywalking.apm.collector.storage.es.dao.SegmentCostEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/SegmentCostEsPersistenceDAO.java) ，SegmentCost 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/06.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.segment.SegmentCostSpanListener`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/segment/SegmentCostSpanListener.java) ，**SegmentCost 的 SpanListener** ，实现了 FirstSpanListener 、EntrySpanListener 、ExitSpanListener 、LocalSpanListener 接口。

------

在 [`TraceStreamGraph#createSegmentCostGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L172) 方法中，我们可以看到 SegmentCost 对应的 `Graph<SegmentCost>` 对象的创建。

- [`org.skywalking.apm.collector.agent.stream.worker.trace.segment.SegmentCostPersistenceWorker`](https://github.com/YunaiV/skywalking/blob/b8be916ac2bf7817544d6ff77a95a624bcc3efe6/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/segment/SegmentCostPersistenceWorker.java) ，继承 PersistenceWorker 抽象类，InstPerformance **批量**保存 Worker 。
  - 类似 GlobalTracePersistenceWorker ，… 省略其它类和方法。

# 6. NodeComponent

[`org.skywalking.apm.collector.storage.table.node.NodeComponent`](https://github.com/YunaiV/skywalking/blob/297c693e9e91200860a147ca41473f68d48d5955/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/node/NodeComponent.java) ，节点组件。

- `org.skywalking.apm.collector.storage.table.node.NodeComponentTable`

   

  ， NodeComponentTable 表(

   

  ```
  node_component
  ```

   

  )。字段如下：

  - `component_id` ：组件编号，参见 [ComponentsDefine](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-network/src/main/java/org/skywalking/apm/network/trace/component/ComponentsDefine.java) 的枚举。
  - `peer_id` ：对等编号。每个组件，或是服务提供者，有服务地址；又或是服务消费者，有调用服务地址。这两者都脱离不开**服务地址**。SkyWalking 将**服务地址**作为 `applicationCode` ，注册到 Application 。因此，此处的 `peer_id` 实际上是，**服务地址**对应的应用编号。
  - `time_bucket` ：时间( `yyyyMMddHHmm` )。

- [`org.skywalking.apm.collector.storage.es.dao.NodeComponentEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/NodeComponentEsPersistenceDAO.java) ，NodeComponent 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/07.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.node.NodeComponentSpanListener`](https://github.com/YunaiV/skywalking/blob/297c693e9e91200860a147ca41473f68d48d5955/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeComponentSpanListener.java) ，**NodeComponent 的 SpanListener** ，实现了 FirstSpanListener 、EntrySpanListener 、ExitSpanListener 接口，代码如下：

- `nodeComponents` 属性，节点组件**数组**，一次 TraceSegment 可以经过个节点组件，例如 SpringMVC => MongoDB 。

- `segmentId` 属性，TraceSegment 链路编号。

- `timeBucket` 属性，时间( `yyyyMMddHHmm` )。

- [`#parseEntry(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/297c693e9e91200860a147ca41473f68d48d5955/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeComponentSpanListener.java#L65) 方法，从 EntrySpan 中解析到 `segmentId` ，`applicationId` ，创建 NodeComponent 对象，添加到 `nodeComponents` 。**注意**，EntrySpan 使用 `applicationId` 作为 `peerId` 。

- [`#parseExit(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/297c693e9e91200860a147ca41473f68d48d5955/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeComponentSpanListener.java#L53) 方法，从 ExitSpan 中解析到 `segmentId` ，`peerId` ，创建 NodeComponent 对象，添加到 `nodeComponents` 。**注意**，ExitSpan 使用 `peerId` 作为 `peerId` 。

- [`#parseFirst(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/297c693e9e91200860a147ca41473f68d48d5955/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeComponentSpanListener.java#L78) 方法，从**首个** Span 中解析到 `timeBucket` 。

- `#build()`

   

  方法，构建，代码如下：

  - 第 84 行：获取 NodeComponent 对应的 `Graph<NodeComponent>` 对象。
  - 第 86 至 92 行：循环 `nodeComponents` 数组，逐个调用 `Graph#start(nodeComponent)` 方法，进行流式处理。在这过程中，会保存 NodeComponent 到存储器。

------

在 [`TraceStreamGraph#createNodeComponentGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L109) 方法中，我们可以看到 NodeComponent 对应的 `Graph<NodeComponent>` 对象的创建。

- `org.skywalking.apm.collector.agent.stream.worker.trace.node.NodeComponentAggregationWorker`

   

  ，继承 AggregationWorker 抽象类，NodeComponent 聚合 Worker 。

  - NodeComponent 的编号生成规则为 `${timeBucket}_${componentId}_${peerId}` ，并且 `timeBucket` 是**分钟级** ，可以使用 AggregationWorker 进行聚合，合并相同操作，减小 Collector 和 ES 的压力。
  - [Factory](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryAggregationWorker.java#L40) 内部类，实现 AbstractLocalAsyncWorkerProvider 抽象类，在 [《SkyWalking 源码分析 —— Collector Streaming Computing 流式处理（一）》「3.2.2 AbstractLocalAsyncWorker」](http://www.iocoder.cn/SkyWalking/collector-streaming-first/?self) 有详细解析。
  - **AggregationWorker** ，在 [《SkyWalking 源码分析 —— Collector Streaming Computing 流式处理（二）》「3. AggregationWorker」](http://www.iocoder.cn/SkyWalking/collector-streaming-second/?self) 有详细解析。
  - [`#id()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryAggregationWorker.java#L36) 实现方法，返回 106 。

- `org.skywalking.apm.collector.agent.stream.worker.trace.service.ServiceEntryRemoteWorker`

   

  ，继承 AbstractRemoteWorker 抽象类，应用注册远程 Worker 。

  - [Factory](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryRemoteWorker.java#L50) 内部类，实现 AbstractRemoteWorkerProvider 抽象类，在 [《SkyWalking 源码分析 —— Collector Streaming Computing 流式处理（一）》「3.2.3 AbstractRemoteWorker」](http://www.iocoder.cn/SkyWalking/collector-streaming-first/?self) 有详细解析。
  - **AbstractRemoteWorker** ，在 [《SkyWalking 源码分析 —— Collector Streaming Computing 流式处理（一）》「3.2.3 AbstractRemoteWorker」](http://www.iocoder.cn/SkyWalking/collector-streaming-first/?self) 有详细解析。
  - [`#id()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryRemoteWorker.java#L38) 实现方法，返回 10002 。
  - [`#selector`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryRemoteWorker.java#L46) 实现方法，返回 `Selector.HashCode` 。将**相同编号**的 NodeComponent 发给同一个 Collector 节点，统一处理。在 [《SkyWalking 源码分析 —— Collector Remote 远程通信服务》](http://www.iocoder.cn/SkyWalking/collector-remote-module/?self) 有详细解析。

- [`org.skywalking.apm.collector.agent.stream.worker.trace.service.ServiceEntryPersistenceWorker`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryPersistenceWorker.java) ，继承 PersistenceWorker 抽象类，NodeComponent **批量**保存 Worker 。

  - 类似 GlobalTracePersistenceWorker ，… 省略其它类和方法。
  - [`#needMergeDBData()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryPersistenceWorker.java#L43) 实现方法，返回 `true` ，存储时，需要合并数据。

# 7. NodeMapping

[`org.skywalking.apm.collector.storage.table.node.NodeComponent`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/node/NodeComponent.java) ，节点匹配，用于匹配服务消费者与提供者。

- `org.skywalking.apm.collector.storage.table.node.NodeMappingTable`

   

  ， NodeMappingTable 表(

   

  ```
  node_mapping
  ```

   

  )。字段如下：

  - `application_id` ：服务消费者应用编号。
  - `address_id` ：服务提供者应用编号。
  - `time_bucket` ：时间( `yyyyMMddHHmm` )。

- [`org.skywalking.apm.collector.storage.es.dao.NodeMappingEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/NodeMappingEsPersistenceDAO.java) ，NodeMapping 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/08.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.node.NodeMappingSpanListener`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeMappingSpanListener.java) ，**NodeMapping 的 SpanListener** ，实现了 FirstSpanListener 、RefsListener 接口，代码如下：

- `nodeMappings` 属性，节点匹配**数组**，一次 TraceSegment 可以经过个节点组件，例如调用多次远程服务，或者数据库。

- `timeBucket` 属性，时间( `yyyyMMddHHmm` )。

- [`#parseRef(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeMappingSpanListener.java#L52) 方法，从 TraceSegmentRef 中解析到 `applicationId` ，`peerId` ，创建 NodeMapping 对象，添加到 `nodeMappings` 。

- [`#parseFirst(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeMappingSpanListener.java#L66) 方法，从**首个** Span 中解析到`timeBucket` 。

- `#build()`

   

  方法，构建，代码如下：

  - 第 84 行：获取 NodeMapping 对应的 `Graph<NodeMapping>` 对象。
  - 第 86 至 92 行：循环 `nodeMappings` 数组，逐个调用 `Graph#start(nodeMapping)` 方法，进行流式处理。在这过程中，会保存 NodeMapping 到存储器。

------

在 [`TraceStreamGraph#createNodeMappingGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L120) 方法中，我们可以看到 NodeMapping 对应的 `Graph<NodeMapping>` 对象的创建。

- 和 NodeComponent 的 `Graph<NodeComponent>` 基本一致，胖友自己看下源码。
- [`org.skywalking.apm.collector.agent.stream.worker.trace.node.NodeMappingAggregationWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeMappingAggregationWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.node.NodeMappingRemoteWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeMappingRemoteWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.node.NodeMappingPersistenceWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/node/NodeMappingPersistenceWorker.java)

# 8. NodeReference

[`org.skywalking.apm.collector.storage.table.noderef.NodeReference`](https://github.com/YunaiV/skywalking/blob/112bcba1a7543e3c86fcbfb49718f7e4f3f4638f/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/noderef/NodeReference.java) ，节点调用统计，用于记录服务消费者对服务提供者的调用，基于**应用**级别的，以**分钟**为时间最小粒度的聚合统计。

- `org.skywalking.apm.collector.storage.table.noderef.NodeReference`

   

  ， NodeReference 表(

   

  ```
  node_reference
  ```

   

  )。字段如下：

  - `front_application_id` ：服务消费者应用编号。
  - `behind_application_id` ：服务提供者应用编号。
  - `s1_lte` ：( 0, 1000 ms ] 的调用次数。
  - `s3_lte` ：( 1000, 3000 ms ] 的调用次数。
  - `s5_lte` ：( 3000, 5000ms ] 的调用次数
  - `s5_gt` ：( 5000, +∞ ] 的调用次数。
  - `error` ：发生异常的调用次数。
  - `summary` ：总共的调用次数。
  - `time_bucket` ：时间( `yyyyMMddHHmm` )。

- [`org.skywalking.apm.collector.storage.es.dao.NodeReferenceEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/NodeReferenceEsPersistenceDAO.java) ，NodeReference 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/09.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.NodeReferenceSpanListener`](https://github.com/YunaiV/skywalking/blob/112bcba1a7543e3c86fcbfb49718f7e4f3f4638f/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferenceSpanListener.java) ，**NodeReference 的 SpanListener** ，实现了 EntrySpanListener 、ExitSpanListener 、RefsListener 接口，代码如下：

- `references` 属性，**父 TraceSegment** 调用产生的 NodeReference 数组。

- `nodeReferences` 属性，NodeReference 数组，最终会包含 `references` 数组。

- `timeBucket` 属性，时间( `yyyyMMddHHmm` )。

- `#parseRef(SpanDecorator, applicationId, instanceId, segmentId)`

   

  方法，代码如下：

  - 第 106 至 109 行：使用父 TraceSegment 的应用编号作为服务**消费者**编号，自己的应用编号作为服务**提供者**应用编号，创建 NodeReference 对象。
  - 第 111 行：将 NodeReference 对象，添加到 `references` 。**注意**，是 `references` ，而不是 `nodeReference` 。

- `#parseEntry(SpanDecorator, applicationId, instanceId, segmentId)`

   

  方法，代码如下：

  - 作为服务提供者，**接受**调用。
  - ——- **父 TraceSegment 存在** ——–
  - 第 79 至 85 行：`references` 非空，说明被父 TraceSegment 调用。因此，循环 `references` 数组，设置 `id` ，`timeBucket` 属性( 因为 `timeBucket` 需要从 EntrySpan 中获取，所以 `#parseRef(...)` 的目的，就是临时存储父 TraceSegment 的应用编号到 `references` 中 )。
  - 第 87 行：调用 [`#buildserviceSum(...)`](https://github.com/YunaiV/skywalking/blob/112bcba1a7543e3c86fcbfb49718f7e4f3f4638f/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferenceSpanListener.java#L122) 方法，设置调用次数，然后添加到 `nodeReferences` 中。
  - ——- **父 TraceSegment 不存在** ——–
  - 第 91 至 97 行：使用 `USER_ID` 的应用编号( 特殊，代表 “**用户**“ )作为服务**消费者**编号，自己的应用编号作为服务**提供者**应用编号，创建 NodeReference 对象。
  - 第 99 行：调用 [`#buildserviceSum(...)`](https://github.com/YunaiV/skywalking/blob/112bcba1a7543e3c86fcbfb49718f7e4f3f4638f/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferenceSpanListener.java#L122) 方法，设置调用次数，然后添加到 `nodeReferences` 中。

- `#parseExit(SpanDecorator, applicationId, instanceId, segmentId)`

   

  方法，代码如下：

  - 作为服务消费者，**发起**调用。
  - 第 64 至 71 行：使用自己的应用编号作为服务**消费者**编号，`peerId` 作为服务**提供者**应用编号，创建 NodeReference 对象。
  - 第 73 行：调用 [`#buildserviceSum(...)`](https://github.com/YunaiV/skywalking/blob/112bcba1a7543e3c86fcbfb49718f7e4f3f4638f/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferenceSpanListener.java#L122) 方法，设置调用次数，然后添加到 `nodeReferences` 中。

- `#build()`

   

  方法，构建，代码如下：

  - 第 84 行：获取 NodeReference 对应的 `Graph<NodeReference>` 对象。
  - 第 86 至 92 行：循环 `nodeReferences` 数组，逐个调用 `Graph#start(nodeReference)` 方法，进行流式处理。在这过程中，会保存 NodeReference 到存储器。

------

在 [`TraceStreamGraph#createNodeReferenceGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L131) 方法中，我们可以看到 NodeReference 对应的 `Graph<NodeReference>` 对象的创建。

- 和 NodeComponent 的 `Graph<NodeComponent>` 基本一致，胖友自己看下源码。
- [`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.NodeReferenceAggregationWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferenceAggregationWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.NodeReferenceRemoteWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferenceRemoteWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.NodeReferencePersistenceWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/noderef/NodeReferencePersistenceWorker.java)

# 9. ServiceEntry

[`org.skywalking.apm.collector.storage.table.service.ServiceEntry`](https://github.com/YunaiV/skywalking/blob/d8e7d053381e7317d413188c4248be1dacf4e85a/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/service/ServiceEntry.java) ，入口操作。

- ServiceEntry

   

  只保存分布式链路的入口操作

  ，不同于 ServiceName

   

  保存所有操作

  ，即 ServiceEntry 是 ServiceName 的

  子集

  。

  - **注意，子 TraceSegment 的入口操作也不记录**。

- `org.skywalking.apm.collector.storage.table.service.ServiceEntryTable`

   

  ， ServiceEntry 表(

   

  ```
  service_entry
  ```

   

  )。字段如下：

  - `application_id` ：应用编号。
  - `entry_service_id` ：入口操作编号。
  - `entry_service_name` ：入口操作名。
  - `register_time` ：注册时间。
  - `newest_time` ：最后调用时间。

- [`org.skywalking.apm.collector.storage.es.dao.ServiceEntryEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/ServiceEntryEsPersistenceDAO.java) ，ServiceEntry 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/10.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.service.ServiceEntrySpanListener`](https://github.com/YunaiV/skywalking/blob/112bcba1a7543e3c86fcbfb49718f7e4f3f4638f/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntrySpanListener.java) ，**ServiceEntry 的 SpanListener** ，实现了 EntrySpanListener 、FirstSpanListener 、RefsListener 接口，代码如下：

- `hasReference` 属性， 是否有 TraceSegmentRef 。

- `applicationId` 属性，应用编号。

- `entryServiceId` 属性，入口操作编号。

- `entryServiceName` 属性，入口操作名。

- `hasEntry` 属性，是否有 EntrySpan 。

- `timeBucket` 属性，时间( `yyyyMMddHHmm` )。

- [`#parseRef(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/d8e7d053381e7317d413188c4248be1dacf4e85a/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntrySpanListener.java#L75) 方法，是否有 TraceSegmentRef 。

- [`#parseFirst(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/d8e7d053381e7317d413188c4248be1dacf4e85a/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntrySpanListener.java#L89) 方法，从**首个** Span 中解析到 `timeBucket` 。

- [`#parseEntry(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/d8e7d053381e7317d413188c4248be1dacf4e85a/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntrySpanListener.java#L75) 方法，从 EntrySpan 中解析到 `applicationId` 、`entryServiceId` 、`entryServiceName` 、`hasEntry` 。

- `#build()`

   

  方法，构建，代码如下：

  - 第 96 行：只保存分布式链路的入口操作。
  - 第 98 至 103 行：创建 ServiceEntry 对象。
  - 第 107 行：获取 ServiceEntry 对应的 `Graph<ServiceEntry>` 对象。
  - 第 108 行：调用 `Graph#start(serviceEntry)` 方法，进行流式处理。在这过程中，会保存 ServiceEntry 到存储器。

------

在 [`TraceStreamGraph#createServiceEntryGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L142) 方法中，我们可以看到 ServiceEntry 对应的 `Graph<ServiceEntry>` 对象的创建。

- 和 NodeComponent 的 `Graph<NodeComponent>` 基本一致，胖友自己看下源码。
- [`org.skywalking.apm.collector.agent.stream.worker.trace.service.ServiceEntryAggregationWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryAggregationWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.service.ServiceEntryRemoteWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryRemoteWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.service.ServiceEntryPersistenceWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/service/ServiceEntryPersistenceWorker.java)

# 10. ServiceReference

[`org.skywalking.apm.collector.storage.table.serviceref.ServiceReference`](https://github.com/YunaiV/skywalking/blob/288d70975ed1c5f1ecfb7d51e2233ec75ad8d12a/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/serviceref/ServiceReference.java) ，入口操作调用统计，用于记录入口操作的调用，基于**入口操作**级别的，以**分钟**为时间最小粒度的聚合统计。

- 和 NodeReference 类似。

- **注意**，此处的 “**入口操作**“ 不同于 ServiceEntry ，**包含**每一条 TraceSegment 的入口操作。

- `org.skywalking.apm.collector.storage.table.serviceref.ServiceReferenceTable`

   

  ， ServiceReference 表(

   

  ```
  service_reference
  ```

   

  )。字段如下：

  - `entry_service_id` ：入口操作编号。
  - `front_service_id` ：服务消费者操作编号。
  - `behind_service_id` ：服务提供者操作编号。
  - `s1_lte` ：( 0, 1000 ms ] 的调用次数。
  - `s3_lte` ：( 1000, 3000 ms ] 的调用次数。
  - `s5_lte` ：( 3000, 5000ms ] 的调用次数
  - `s5_gt` ：( 5000, +∞ ] 的调用次数。
  - `error` ：发生异常的调用次数。
  - `summary` ：总共的调用次数。
  - `cost_summary` ：总共的花费时间。
  - `time_bucket` ：时间( `yyyyMMddHHmm` )。

- [`org.skywalking.apm.collector.storage.es.dao.ServiceReference`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/ServiceReference.java) ，ServiceReference 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/11.png)

------

[`org.skywalking.apm.collector.agent.stream.worker.trace.segment.ServiceReferenceSpanListener`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java) ，**ServiceReference 的 SpanListener** ，实现了 EntrySpanListener 、FirstSpanListener 、RefsListener 接口，代码如下：

- `referenceServices` 属性，ReferenceDecorator 数组，记录 TraceSegmentRef 数组。

- `serviceId` 属性，入口操作编号。

- `startTime` 属性，开始时间。

- `endTime` 属性，结束时间。

- `isError` 属性，是否有错误。

- `hasEntry` 属性，是否有 SpanEntry 。

- `timeBucket` 属性，时间( `yyyyMMddHHmm` )。

- [`#parseRef(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L79) 方法，将 TraceSegmentRef 添加到 `referenceServices` 。

- [`#parseFirst(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L74) 方法，从**首个** Span 中解析到 `timeBucket` 。

- [`#parseEntry(SpanDecorator, applicationId, instanceId, segmentId)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L85) 方法，从 EntrySpan 中解析 `serviceId` 、`startTime` 、`endTime` 、`isError` 、`hasEntry` 。

- `#build()`

   

  方法，构建，代码如下：

  - 第 114 行：判断 `hasEntry = true` ，存在 EntrySpan 。
  - ——— **有 TraceSegmentRef** ———
  - 第 117 至 120 行：创建 ServiceReference 对象，其中：
    - `entryServiceId` ：TraceSegmentRef 的入口编号。
    - `frontServiceId` ：TraceSegmentRef 的操作编号。
    - `behindServiceId` ： 自己 EntrySpan 的操作编号。
  - 第 121 行：调用 [`#calculateCost(...)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L94) 方法，设置调用次数。
  - 第 126 行：调用 [`#sendToAggregationWorker(...)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L141) 方法，发送 ServiceReference 给 AggregationWorker ，执行流式处理。
  - ——— **无 TraceSegmentRef** ———
  - 第 117 至 120 行：创建 ServiceReference 对象，其中：
    - `entryServiceId` ：自己 EntrySpan 的操作编号。
    - `frontServiceId` ：`Const.NONE_SERVICE_ID` 对应的操作编号( 系统内置，代表【空】 )。
    - `behindServiceId` ： 自己 EntrySpan 的操作编号。
  - 第 121 行：调用 [`#calculateCost(...)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L94) 方法，设置调用次数。
  - 第 126 行：调用 [`#sendToAggregationWorker(...)`](https://github.com/YunaiV/skywalking/blob/33a9634fff31a299b0baab5ffba603a58d6ff371/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceSpanListener.java#L141) 方法，发送 ServiceReference 给 AggregationWorker ，执行流式处理。

------

在 [`TraceStreamGraph#createServiceReferenceGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L153) 方法中，我们可以看到 ServiceReference 对应的 `Graph<ServiceReference>` 对象的创建。

- 和 NodeComponent 的 `Graph<NodeComponent>` 基本一致，胖友自己看下源码。
- [`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.ServiceEntryAggregationWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceAggregationWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.ServiceEntryRemoteWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferenceRemoteWorker.java)
- [`org.skywalking.apm.collector.agent.stream.worker.trace.noderef.ServiceEntryPersistenceWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/serviceref/ServiceReferencePersistenceWorker.java)

# 11. Segment

不同于上述所有数据实体，Segment 无需**解析**，直接使用 TraceSegment 构建，参见如下方法：

- [`SegmentParse#parse(UpstreamSegment, Source)`](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L109)
- [`SegmentParse#buildSegment(id, dataBinary)`](https://github.com/YunaiV/skywalking/blob/0f8c38a8a35cbda777fa9a9bc5f51c8651ae4051/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/parser/SegmentParse.java#L177)

[`org.skywalking.apm.collector.storage.table.segment.Segment`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-define/src/main/java/org/skywalking/apm/collector/storage/table/global/GlobalTrace.java) ，全局链路追踪，记录一次分布式链路追踪，包括的 TraceSegment 编号。

- `org.skywalking.apm.collector.storage.table.global.GlobalTraceTable`

   

  ， Segment 表(

   

  ```
  segment
  ```

   

  )。字段如下：

  - `_id` ：TraceSegment 编号。
  - `data_binary` ：TraceSegment 链路编号。
  - `time_bucket` ：时间( `yyyyMMddHHmm` )。

- [`org.skywalking.apm.collector.storage.es.dao.SegmentEsPersistenceDAO`](https://github.com/YunaiV/skywalking/blob/3d1d1f5219205d38f58f1b59f0e81d81c038d2f1/apm-collector/apm-collector-storage/collector-storage-es-provider/src/main/java/org/skywalking/apm/collector/storage/es/dao/SegmentEsPersistenceDAO.java) ，GlobalTrace 的 EsDAO 。

- 在 ES 存储例子如下图：![img](https://static.iocoder.cn/images/SkyWalking/2020_10_15/12.png)

------

在 [`TraceStreamGraph#createSegmentGraph()`](https://github.com/YunaiV/skywalking/blob/52e41bc200857d2eb1b285d046cb9d2dd646fb7b/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/graph/TraceStreamGraph.java#L164) 方法中，我们可以看到 Segment 对应的 `Graph<Segment>` 对象的创建。

- [`org.skywalking.apm.collector.agent.stream.worker.trace.segment.SegmentPersistenceWorker`](https://github.com/YunaiV/skywalking/blob/ee9400fc51d6ae21b1053bb0d8cca7ad4d51efe5/apm-collector/apm-collector-agent-stream/collector-agent-stream-provider/src/main/java/org/skywalking/apm/collector/agent/stream/worker/trace/segment/SegmentPersistenceWorker.java)

# [traceId 集成到日志组件](https://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/)

**本文主要基于 SkyWalking 3.2.6 正式版**

- [1. 概述](http://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/)
- [2. 使用例子](http://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/)
- \3. 实现代码
  - [3.1 TraceIdPatternLogbackLayout](http://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/)
  - [3.2 LogbackPatternConverterActivation](http://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/)
- [666. 彩蛋](http://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/)

------

------

# 1. 概述

本文主要分享 **traceId 集成到日志组件**，例如 log4j 、log4j2 、logback 等等。

我们首先看看**集成**的使用例子，再看看**集成**的实现代码。涉及代码如下：

- ![img](https://static.iocoder.cn/images/SkyWalking/2020_11_15/01.png)
- ![img](https://static.iocoder.cn/images/SkyWalking/2020_11_15/02.png)

本文以 **logback 1.x** 为例子。

# 2. 使用例子

1、**无需**引入相应的工具包，只需启动参数带上 `-javaagent:/Users/yunai/Java/skywalking/packages/skywalking-agent/skywalking-agent.jar` 。

2、在 `logback.xml` 配置 `%tid` 占位符：

![img](https://static.iocoder.cn/images/SkyWalking/2020_11_15/03.png)

3、使用 `logger.info(...)` ，会打印日志如下：

![img](https://static.iocoder.cn/images/SkyWalking/2020_11_15/04.png)

**注意**，traceId 打印到每条日志里，最终需要经过例如 ELK ，收集到日志中心。

# 3. 实现代码

## 3.1 TraceIdPatternLogbackLayout

[`org.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout`](https://www.iocoder.cn/SkyWalking/trace-id-integrate-into-logs/) ，实现 `ch.qos.logback.classic.PatternLayout` 类，实现支持 `%tid` 的占位符。代码如下：

- 第 33 行：添加 `tid` 的转换器为 [`org.skywalking.apm.toolkit.log.logback.v1.x.LogbackPatternConverter`](https://github.com/YunaiV/skywalking/blob/5106601937af942dabcad917b90d8c92886a2e4d/apm-application-toolkit/apm-toolkit-logback-1.x/src/main/java/org/skywalking/apm/toolkit/log/logback/v1/x/LogbackPatternConverter.java) 类。

## 3.2 LogbackPatternConverterActivation

[`org.skywalking.apm.toolkit.activation.log.logback.v1.x.LogbackPatternConverterActivation`](https://github.com/YunaiV/skywalking/blob/5106601937af942dabcad917b90d8c92886a2e4d/apm-sniffer/apm-toolkit-activation/apm-toolkit-logback-1.x-activation/src/main/java/org/skywalking/apm/toolkit/activation/log/logback/v1/x/LogbackPatternConverterActivation.java) ，实现 ClassInstanceMethodsEnhancePluginDefine 抽象类，定义了方法切面，代码如下：

![img](https://static.iocoder.cn/images/SkyWalking/2020_11_15/05.png)

------

[`org.skywalking.apm.toolkit.activation.log.logback.v1.x.PrintTraceIdInterceptor`](https://github.com/YunaiV/skywalking/blob/5106601937af942dabcad917b90d8c92886a2e4d/apm-sniffer/apm-toolkit-activation/apm-toolkit-logback-1.x-activation/src/main/java/org/skywalking/apm/toolkit/activation/log/logback/v1/x/PrintTraceIdInterceptor.java) ，实现 InstanceMethodsAroundInterceptor 接口，LogbackPatternConverterActivation 的拦截器。代码如下：

- [`#afterMethod(...)`](https://github.com/YunaiV/skywalking/blob/5106601937af942dabcad917b90d8c92886a2e4d/apm-sniffer/apm-toolkit-activation/apm-toolkit-logback-1.x-activation/src/main/java/org/skywalking/apm/toolkit/activation/log/logback/v1/x/PrintTraceIdInterceptor.java#L37) 方法，调用 `ContextManager#getGlobalTraceId()` 方法，使用全局链路追踪编号，而不是原有结果。



# Java Agent源码结构

- 链路数据模型抽象与封装

作用是封装链路数据协议约定的数据抽象，同时总体兼容了Opentracing规范。重点类有如下几个

- `org.apache.skywalking.apm.agent.core.context.trace.TraceSegment`
- `org.apache.skywalking.apm.agent.core.context.trace.TraceSegmentRef`
- `org.apache.skywalking.apm.agent.core.context.trace.EntrySpan`
- `org.apache.skywalking.apm.agent.core.context.trace.ExitSpan`
- `org.apache.skywalking.apm.agent.core.context.ids.DistributedTraceIds`

- 跨进程链路数据抽象与封装

重点类是`org.apache.skywalking.apm.agent.core.context.ContextCarrier`。即是对本文所讲解的协议规范的封装。

- 链路数据采集模型抽象与封装

重点类是`org.apache.skywalking.apm.agent.core.context.TracingContext`。TracingContext保存在ThreadLocal中，包含了链路中的所有数据，以及对数据的管控方法，主要是对Span的管控。同时提供对ContextCarrier数据的处理，包括：

- 将TraceSegment转换为ContextCarrier，即org.apache.skywalking.apm.agent.core.context.TracingContext#inject
- 从ContextCarrier数据中抽取TraceSegment数据，即org.apache.skywalking.apm.agent.core.context.TracingContext#extract

- 链路数据采集模块

重点是`org.apache.skywalking.apm.agent.core.context.ContextManager`类。ContextManager类是各种skywalking agent插件的枢纽。不同组件的skywalking插件，如mq，dubbo，tomcat，spring等skywalking agent插件均是通过调用ContextManager创建和管控TracingContext，TraceSegment，EntrySpan，ExitSpan，ContextCarrier等数据。可以说ContextManager管控着agent内的链路数据生命周期。

- 链路数据上传模块

重点是`org.apache.skywalking.apm.agent.core.remote`包中的`TraceSegmentServiceClient`类。
ContextManager采集节点内的链路数据片段（TraceSegment）后，通知TraceSegmentServiceClient将数据上报到Collector Server。其中涉及到
`TracingContextListener`与skywalking封装的`内存MQ`组件，后面会详细分析。

## Global(Distributed) Trace Id与Trace Segment Id

### 区别

skywalking中分别有两类全局唯一的id，即Global(Distributed) Trace Id与Trace Segment Id。Global(Distributed) Trace Id是指分布系统下的某条链路的唯一ID。Trace Segment Id是指链路所经过的分布系统服务节点事生成的链路片段ID。两者是一对多的关系。

### 生成流程

上述序列图表明，在实例化TracingContext时，会实例化TraceSegment，而在实例化TraceSegment时，同时生成了Global(Distributed) Trace Id与Trace Segment Id，源码如下：

```haxe
    public TraceSegment() {
        this.traceSegmentId = GlobalIdGenerator.generate();
        this.spans = new LinkedList<AbstractTracingSpan>();
        this.relatedGlobalTraces = new DistributedTraceIds();
        this.relatedGlobalTraces.append(new NewDistributedTraceId());
    }
```

- 生成trace segment id

其中，第1行代码`this.traceSegmentId = GlobalIdGenerator.generate();`，即是生成Trace Segment Id，调用了一次GlobalIdGenerator.generate()方法。
在第4行代码的`new NewDistributedTraceId()`创建了Global(Distributed) Trace Id。源码如下，可见同样是调用`GlobalIdGenerator.generate()`方法。

```scala
public class NewDistributedTraceId extends DistributedTraceId {
    public NewDistributedTraceId() {
        super(GlobalIdGenerator.generate());
    }
}
```

- 生成Global(Distributed) Trace Id

全局链路id的的逻辑较复杂，java agent中为这个id封装了`DistributedTraceIds`类，通过该类的append方法添加`NewDistributedTraceId `对象，表示一个Global(Distributed) Trace Id数据。NewDistributedTraceId是DistributedTraceId的子类，而DistributedTraceId类就是对ID类的包装，提供了一些工具方法。
`DistributedTraceIds`代表了相关链路id的集合，大多数情况下仅包含一条链路id。

### 关于GlobalIdGenerator

两个id均是调用同样的方法生成，即`org.apache.skywalking.apm.agent.core.context.ids.GlobalIdGenerator#generate`。
GlobalIdGenerator是生产全局ID字符串的逻辑封装类。下面看看`GlobalIdGenerator#generate`的源码。

```verilog
    public static ID generate() {
        if (RemoteDownstreamConfig.Agent.APPLICATION_INSTANCE_ID == DictionaryUtil.nullValue()) {
            throw new IllegalStateException();
        }
        IDContext context = THREAD_ID_SEQUENCE.get();

        return new ID(
            RemoteDownstreamConfig.Agent.APPLICATION_INSTANCE_ID,
            Thread.currentThread().getId(),
            context.nextSeq()
        );
    }
```

上述源码最终返回的即是根据协议规范封装的ID数据结构，第一个参数为当前应用的实例ID，第二个参数为当前线程号。第三个参数是通过IDContext.nextSeq()方法获取。而IDContext是从ThreadContext中获取的，所以IDContext是线程独有的。IDContext的源码比较好理解，IDContext.nextSeq()返回当前时间戳与线程内的自增序号组合的字符串。

## 总结

综上，在新链路开始，生成TraceSegment实例时，第一次调用`GlobalIdGenerator.generate()`作为`Trace Segment Id`，第二次调用`GlobalIdGenerator.generate()`作为`Global(Distributed) Trace Id`。



## 手把手教你编写Skywalking插件

## 前置知识

在正式进入编写环节之前，建议先花一点时间了解下javaagent（这是JDK 5引入的一个玩意儿，最好了解下其工作原理）；另外，Skywalking用到了byte-buddy（一个动态操作二进制码的库），所以最好也熟悉下。

当然不了解关系也不大，一般不影响你玩转Skywalking。

- [javaagent](https://link.segmentfault.com/?enc=%2B%2FD%2FVP6HHVJ4el9DaVjZNw%3D%3D.lx4ydyf0tEq5oKtQPtYgZOWikMOWE4mhV7NWdRzbGHdnU3NdKIDw2AS%2FQVp5%2Fn1GjzZ1YRJI40L9qZ5fn%2F%2BY%2FA%3D%3D)
- [byte-buddy 1.9.6 简述及原理1](https://link.segmentfault.com/?enc=c5izOPDlNaCoUUGV%2F89E4A%3D%3D.UWXceWWKJBI45oP1W%2BYy%2Befi3cr3cxSxclLSjr5UdQRFscYeyY44QVIpp0VjIAaDo4XYgN0sAU4OmKv1iYYzaw%3D%3D)

## 术语

Span:可理解为一次方法调用，一个程序块的调用，或一次RPC/数据库访问。只要是一个具有完整时间周期的程序访问，都可以被认为是一个span。SkyWalking `Span` 对象中的重要属性

| 属性          | 名称     | 备注                                                       |
| ------------- | -------- | ---------------------------------------------------------- |
| component     | 组件     | 插件的组件名称，如：Lettuce，详见:ComponentsDefine.Class。 |
| tag           | 标签     | k-v结构，关键标签，key详见：Tags.Class。                   |
| peer          | 对端资源 | 用于拓扑图，若DB组件，需记录集群信息。                     |
| operationName | 操作名称 | 若span=0，operationName将会搜索的下拉列表。                |
| layer         | 显示     | 在链路页显示，详见SpanLayer.Class。                        |

Trace:调用链，通过归属于其的Span来隐性的定义。一条Trace可被认为是一个由多个Span组成的有向无环图（DAG图），在SkyWalking链路模块你可以看到，Trace又由多个归属于其的trace segment组成。

Trace segment:Segment是SkyWalking中的一个概念，它应该包括单个OS进程中每个请求的所有范围，通常是基于语言的单线程。由多个归属于本线程操作的Span组成。

> **TIPS**
> Skywalking的这几个术语和Spring Cloud Sleuth类似，借鉴自谷歌的Dapper。我在 [《Spring Cloud Alibaba微服务从入门到进阶》](https://link.segmentfault.com/?enc=7TNjSh16NKm4l8SB0OWmwg%3D%3D.I3ThFmv4hAmBWJ2GEBboW4%2BBJAbcNE0DxxW8eyZxHA8mU3xYb6AZ0Y2JQSVyNg7D) 课程中有详细剖析调用链的实现原理，并且用数据库做了通俗的类比，本文不再赘述。

## 核心API

详见 [http://www.itmuch.com/books/skywalking/guides/Java-Plugin-Development-Guide.html](https://link.segmentfault.com/?enc=nfEe3JoFhIpqOU%2BWTsEEnA%3D%3D.9N0X5D1vPkXRLQQw2rIM8lXU12eaWmKtzKB%2BN5PG2sU5U%2B6OlrhGGASmmQYIaMz3yZms86YhVTuUDGfSxNjEk43hFe3Tc8fhEHEwuQlET01Dw5BzW3RC2NPwWCAQcqy1) 文章有非常详细的描述。

## 实战

本文以监控 `org.apache.commons.lang3.StringUtils.replace` 为例，手把手教你编写Skywalking插件。

### 依赖

首先，创建一个Maven项目，Pom.xml如下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itmuch.skywalking</groupId>
    <artifactId>apm-string-replace-plugin</artifactId>
    <packaging>jar</packaging>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <skywalking.version>6.6.0</skywalking.version>
        <shade.package>org.apache.skywalking.apm.dependencies</shade.package>
        <shade.net.bytebuddy.source>net.bytebuddy</shade.net.bytebuddy.source>
        <shade.net.bytebuddy.target>${shade.package}.${shade.net.bytebuddy.source}</shade.net.bytebuddy.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.skywalking</groupId>
            <artifactId>apm-agent-core</artifactId>
            <version>${skywalking.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.skywalking</groupId>
            <artifactId>apm-util</artifactId>
            <version>${skywalking.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-shade-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <shadedArtifactAttached>false</shadedArtifactAttached>
                            <createDependencyReducedPom>true</createDependencyReducedPom>
                            <createSourcesJar>true</createSourcesJar>
                            <shadeSourcesContent>true</shadeSourcesContent>
                            <relocations>
                                <relocation>
                                    <pattern>${shade.net.bytebuddy.source}</pattern>
                                    <shadedPattern>${shade.net.bytebuddy.target}</shadedPattern>
                                </relocation>
                            </relocations>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>6</source>
                    <target>6</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

这个Pom.xml中，除如下依赖以外，其他都得照抄，不管你开发什么框架的Skywalking插件，**否则无法正常构建插件**！！

```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.4</version>
  <scope>provided</scope>
</dependency>
```

### 编写Instrumentation类

```java
public class StringReplaceInstrumentation extends ClassInstanceMethodsEnhancePluginDefine {
    @Override
    protected ClassMatch enhanceClass() {
        // 指定想要监控的类
        return NameMatch.byName("org.apache.commons.lang3.StringUtils");
    }

    @Override
    public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        return new ConstructorInterceptPoint[0];
    }

    @Override
    public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints() {
        // 指定想要监控的实例方法，每个实例方法对应一个InstanceMethodsInterceptPoint
        return new InstanceMethodsInterceptPoint[0];
    }

    @Override
    public StaticMethodsInterceptPoint[] getStaticMethodsInterceptPoints() {
        // 指定想要监控的静态方法，每一个方法对应一个StaticMethodsInterceptPoint
        return new StaticMethodsInterceptPoint[]{
            new StaticMethodsInterceptPoint() {
                @Override
                public ElementMatcher<MethodDescription> getMethodsMatcher() {
                    // 静态方法名称
                    return ElementMatchers.named("replace");
                }

                @Override
                public String getMethodsInterceptor() {
                    // 该静态方法的监控拦截器类名全路径
                    return "com.itmuch.skywalking.plugin.stringreplace.StringReplaceInterceptor";
                }

                @Override
                public boolean isOverrideArgs() {
                    return false;
                }
            }
        };
    }
}
```

### 编写拦截器

```java
public class StringReplaceInterceptor implements StaticMethodsAroundInterceptor {
    @Override
    public void beforeMethod(Class aClass, Method method, Object[] argumentsTypes, Class<?>[] classes, MethodInterceptResult methodInterceptResult) {
        // 创建span(监控的开始)，本质上是往ThreadLocal对象里面设值
        AbstractSpan span = ContextManager.createLocalSpan("replace");

        /*
         * 可用ComponentsDefine工具类指定Skywalking官方支持的组件
         * 也可自己new OfficialComponent或者Component
         * 不过在Skywalking的控制台上不会被识别，只会显示N/A
         */
        span.setComponent(ComponentsDefine.TOMCAT);

        span.tag(new StringTag(1000, "params"), argumentsTypes[0].toString());
        // 指定该调用的layer，layer是个枚举
        span.setLayer(SpanLayer.CACHE);
    }

    @Override
    public Object afterMethod(Class aClass, Method method, Object[] objects, Class<?>[] classes, Object o) {
        String retString = (String) o;
        // 激活span，本质上是读取ThreadLocal对象
        AbstractSpan span = ContextManager.activeSpan();
        // 状态码，任意写，Tags也是个Skywalking的工具类，用来比较方便地操作tag
        Tags.STATUS_CODE.set(span, "20000");

        // 停止span(监控的结束)，本质上是清理ThreadLocal对象
        ContextManager.stopSpan();
        return retString;
    }

    @Override
    public void handleMethodException(Class aClass, Method method, Object[] objects, Class<?>[] classes, Throwable throwable) {
        AbstractSpan activeSpan = ContextManager.activeSpan();
        
        // 记录日志
        activeSpan.log(throwable);
        activeSpan.errorOccurred();
    }
}
```

### 编写配置文件

创建`resources/skywalking-plugin.def` ，内容如下：

```applescript
# Key=value的形式
# key随便写；value是Instrumentation类的包名类名全路径
my-string-replace-
plugin=org.apache.skywalking.apm.plugin.stringreplace.define.StringReplaceInstrumentation
```

### 构建

```armasm
mvn clean install
```

构建完成后，到target目录中，找到JAR包(非 `origin-xxx.jar` 、非`xxx-source.jar` )，扔到 `agent/plugins` 目录里面去，即可启动。

### 插件调试

插件的编写可能不是一步到位的，有时候可能会报点错什么的。如果想要Debug自己的插件，那么需要将你的插件代码和接入Java Agent的项目（也就是你配置了-javaagent启动的项目）扔到同一个工作空间内，可以这么玩：

- 使用IDEA，打开接入Java Agent的项目
- 找到File->New->Module from Exisiting Sources…，引入你的插件源码即可。

### 测试与监控效果

想办法让你接入Java Agent的项目，调用到如下代码

```java
String replace = StringUtils.replace("oldString", "old","replaced");
System.out.println(replace);
```

之后，就可以看到类似如下的图啦：

## 写在最后

本文只是弄了一个简单的例子，讲解插件编写的套路。总的来说，插件编写还是非常顺利的，单纯代码的层面，很少会遇到坑；但搜遍各种搜索引擎，竟然没有一篇手把手的文章…而且也没有文章讲解依赖该如何引入、Maven插件如何引入。于是只好参考Skywalking官方插件的写法，引入依赖和Maven插件了，这块反倒是费了点时间。

此外，如果你想真正掌握乃至精通skywalking插件编写，最好的办法，还是阅读官方的插件代码，详见：`https://github.com/apache/skywalking/tree/master/apm-sniffer/apm-sdk-plugin` ，随便挑两款看一下就知道怎么玩了。我在学习过程中参考的插件代码有：

- apm-feign-default-http-9.x-plugin
- apm-jdbc-commons

### Skywalking自定义Tag

#### Skywalking内置Tags

org.apache.skywalking.apm.agent.core.context.tag.Tags

| 取值 | 对应Tag      |
| ---- | ------------ |
| 1    | url          |
| 2    | status_code  |
| 3    | db.type      |
| 4    | db.instance  |
| 5    | db.statement |
| 6    | db.bind_vars |
| 7    | mq.queue     |
| 8    | mq.broker    |
| 9    | mq.topic     |
| 10   | http.method  |
| 11   | http.params  |
| 12   | x-le         |
| 13   | http.body    |
| 14   | http.headers |

我们提到了“skywalking收集链路时，使用的URL都是通配符，在链路中，无法针对某个pageId，或者其他通配符的具体的值进行查找。或许skywalking出于性能考虑，但是对于这种不定的通用大接口，的确无法用于针对性的性能分析了。”

 那么在skywalking 8.2版本中引入的对tag搜索的支持，就能够解决这个问题，我们可以根据tag对链路进行一次过滤，得到一类的链路。让我们来看看如何配置的。
我们根据一个叫`biz.id`的标签，查找过滤其值为"bbbc"的链路。那么这种自定义标签`biz.id`，如何配置呢？（skywalking 默认支持`http.method`等标签的搜索）

## 配置oap端application.yml

skywalking分析端是一个java进程，使用application.yml作为配置文件，其位置位于“apache-skywalking-apm-bin/config”文件夹中。

core.default.searchableTracesTags 配置项为可搜索标签的配置项，其默认为：“${SW_SEARCHABLE_TAG_KEYS:http.method,status_code,db.type,db.instance,mq.queue,mq.topic,mq.broker}”。如果没有配置环境变量SW_SEARCHABLE_TAG_KEYS，那么其默认就支持这几个在skywalking 中有使用到的几个tag。 那么我在里面修改了配置，加上了我们用到的“biz.id”、“biz.type”。
修改配置后，重启skywalking oap端即可支持。

## 代码进行打标签

racer#activeSpan()方法将会将自身作为构造去生成Span，最终仍是同一个Span。

使用H2数据库的时候，通过tag进行查询就会失效，会查不出链路，通过debug是可以看到对应的sql并无问题，拼出了biz.id的查询条件，具体原因还未查找，通过切换存储为es6解决了问题。(猜测普通的关系型数据库不支持，需要列式存储的数据库才可以)

源码分析

SegmentAnalysisListener

```
   private void appendSearchableTags(SpanObject span) {
        HashSet<Tag> segmentTags = new HashSet<>();
        span.getTagsList().forEach(tag -> {
            if (searchableTagKeys.contains(tag.getKey())) {
                final Tag spanTag = new Tag(tag.getKey(), tag.getValue());
                if (!segmentTags.contains(spanTag)) {
                    segmentTags.add(spanTag);
                }

            }
        });
        segment.getTags().addAll(segmentTags);
    }
```

# Skywalking的使用-异步链路追踪

### 1、异步链路追踪的概述

通过对 `Callable`,`Runnable`,`Supplier` 这3种接口的实现者进行增强拦截，将trace的上下文信息传递到子线程中，实现了异步链路追踪。

有非常多的方式来实现`Callable`,`Runnable`,`Supplier` 这3种接口，那么增强就面临以下问题：

1. 增强所有的实现类显然不可能，必须基于有限的约定
2. 不能让使用者大量修改代码，尽可能的基于现有的实现

可能基于以上问题的考虑，skywalking提供了一种即通用又快捷的方式来规范这一现象：

1. 只拦截增强带有`@TraceCrossThread` 注解的类：
2. 通过装饰的方式包装任务，避免大刀阔斧的修改

| 原始类      | 提供的包装类       | 拦截方法 | 使用技巧                        |
| ----------- | ------------------ | -------- | ------------------------------- |
| Callable<V> | CallableWrapper<V> | call     | CallableWrapper.of(xxxCallable) |
| Runnable    | RunnableWrapper    | run      | RunnableWrapper.of(xxxRunable)  |
| Supplier<V> | SupplierWrapper<V> | get      | SupplierWrapper.of(xxxSupplier) |

包装类 都有注解 `@TraceCrossThread` ，skywalking内部的拦截匹配逻辑是，标注了`@TraceCrossThread`的类，拦截 其名称为`call` 或`run`或 `get`  ，且没有入参的方法；对使用者来说大致分为2种方式：

1. 自定义类，实现接口 `Callable`、`Runnable`、`Supplier`，加`@TraceCrossThread`注解。当需要有更多的自定义属性时，考虑这种方式；参考 `CallableWrapper`、`RunnableWrapper`、`SupplierWrapper` 的实现方式。
2. 通过xxxWrapper.of 装饰的方式，即`CallableWrapper.of(xxxCallable)`、`RunnableWrapper.of(xxxRunable)`、`SupplierWrapper.of(xxxSupplier)`。大多情况下，通过这种包装模式即可。

### 2、异步链路追踪的使用

需引入如下依赖（版本限参考）：



```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>8.5.0</version>
</dependency>
```

#### 2.1.  CallableWrapper

Skywalking 通过`CallableWrapper`包装`Callable`

![img](https:////upload-images.jianshu.io/upload_images/4642883-6dfd54c30fd15cb6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

###### 2.1.1 thread+callable



```dart
private String async_thread_callable(String way,long time11,long time22 ) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<String>(CallableWrapper.of(()->{
            ActiveSpan.debug("async_Thread_Callable");
            String str1 = service.sendMessage(way, time11, time22);
            return str1;
        }));
        new Thread(futureTask).start();
        return futureTask.get();
    }
```

###### 2.1.2 threadPool+callable



```dart
    private String async_executorService_callable(String way,long time11,long time22 ) throws ExecutionException, InterruptedException {
        Future<String> callableResult = executorService.submit(CallableWrapper.of(() -> {
            String str1 = service.sendMessage(way, time11, time22);
            return str1;
        }));
        return (String) callableResult.get();
    }
```

#### 2.2.  RunnableWrapper

Skywalking 通过`RunnableWrapper`包装`Runnable`

![img](https:////upload-images.jianshu.io/upload_images/4642883-b78fbb8216c1f347.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png



###### 2.2.1 thread+runnable



```csharp
    private String async_thread_runnable(String way,long time11,long time22 ) throws ExecutionException, InterruptedException {
        //忽略返回值
        FutureTask futureTask = new FutureTask(RunnableWrapper.of(() -> {
            String str1 = service.sendMessage(way, time11, time22);
        }), "mockRunnableResult");
        new Thread(futureTask).start();
        return (String) futureTask.get();
    }
```

###### 2.2.2 threadPool+runnable



```dart
        private String async_executorService_runnable(String way,long time11,long time22 ) throws ExecutionException, InterruptedException {
        //忽略真实返回值，mock固定返回值
        Future<String> mockRunnableResult = executorService.submit(RunnableWrapper.of(() -> {
            String str1 = service.sendMessage(way, time11, time22);
        }), "mockRunnableResult");
        return (String) mockRunnableResult.get();
    }
```

###### 2.2.3 completableFuture + runAsync

通过RunnableWrapper.of(xxx)包装rannable即可。

#### 2.3.  SupplierWrapper

Skywalking 通过`SupplierWrapper<V>`包装`Supplier<V>`

![img](https:////upload-images.jianshu.io/upload_images/4642883-a229457310526e40.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png



###### 2.3.1 completableFuture + supplyAsync



```csharp
    private String async_completableFuture_supplyAsync(String way,long time11,long time22 ) throws ExecutionException, InterruptedException {
        CompletableFuture<String> stringCompletableFuture = CompletableFuture.supplyAsync(SupplierWrapper.of(() -> {
            String str1 = service.sendMessage(way, time11, time22);
            return str1;
        }));
        return stringCompletableFuture.get();
    }
```

### 3、异步链路追踪的内部原理

需要将trace信息，在线程之间传递，比如 线程A -调用-> 线程B 的场景：

- 线程A
  1. 调用`ContextManager.capture()`,将trace的上下文信息保存到一个`ContextSnapshot`的实例，并返回，此处命名为：contextSnapshot。
  2. 通过某种方式将contextSnapshot传递给线程B。
- 线程B
  1. 在任务执行前，线程中B获取到contextSnapshot对象，并将其作为入参调用`ContextManager.continued(contextSnapshot)`。
  2. 此方法中解析出trace的信息后，存储到线程B的线程上下文中。



我们都知道 ThreadLocal 作为一种多线程处理手段，将数据限制在当前线程中，避免多线程情况下出现错误。

一般的使用场景大多会是服务上下文、分布式日志跟踪。

但是在业务代码中，为了提高响应速度，将多个复杂、长时间的计算或调用过程异步进行，让主线程可以先进行其他操作。像我们项目中最常用的就是 CompletableFuture 了，默认会使用预设的 ForkJoin ThreadPool 执行。

这也就引入了一个问题，如果保证 ThreadLocal 的信息能够传递异步线程？通过 ThreadLocal？通过线程池？通过 Runnable 或者 Callable？

有些场景丢了就丢了，比如目前我们的服务上下文传递，一般都没有很严谨的处理 ......

但是，如果是分布式追踪的场景，丢了就要累惨了。

注：以下代码仅保留关键代码，其余无关紧要则忽略

# InheritableThreadLocal

InheritableThreadLocal 是 JDK 本身自带的一种线程传递解决方案。顾名思义，由当前线程创建的线程，将会继承当前线程里 ThreadLocal 保存的值。

其本质上是 ThreadLocal 的一个子类，通过覆写父类中创建初始化的相关方法来实现的。我们知道，ThreadLocal 实际上是 Thread 中保存的一个 ThreadLocalMap 类型的属性搭配使用才能让广大 Javaer 直呼真香的，所以 InheritableThreadLocal 也是如此。



```java
public class Thread implements Runnable {
    // 如果单纯使用 ThreadLocal，则 Thread 使用该属性值保存 ThreadLocalMap
    ThreadLocal.ThreadLocalMap threadLocals = null;
        // 否则使用该属性值
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
  
    private void init(ThreadGroup g, Runnable target, String name, long stackSize, AccessControlContext acc) {
          Thread parent = currentThread();

          if (parent.inheritableThreadLocals != null)
              this.inheritableThreadLocals =
                  ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
      }
}
```

init 方法作为 Thread 初始化的核心方法，相关 ThreadLocal 代码已经全部摘出。如我们所见，仅仅就只是这一点改动。在创建线程时，如果当前线程的 inheritableThreadLocals 不为空，则根据它创建出新的 InheritableThreadLocals 保存到新线程中。

Ps : ThreadLocal 作为老牌选手，默认都是使用时，直接初始化 Thread 的 threadLocals 属性。

只有像是 InheritableThreadLocal 这样的后辈，需要特殊处理一下。



```cpp
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    
    protected T childValue(T parentValue) {
        return parentValue;
    }
  
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

   // Thread 中 ThreadLocalMap 不存在时的初始化动作，需要改为初始化 inheritableThreadLocals
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

因此，原先 ThreadLocal 会从 Thread 的 threadLocals 获取 Map，那么 InheritableThreadLocal 就要从 inheritableThreadLocals 拿了。 childValue 方法用作从父线程中获取值，可以看到，这边是直接返回的，如果是复杂对象，就直接传引用了。当然，继承覆写该方法，可以实现浅拷贝、深拷贝等等方式。

# 缺点

这样的方式解决了创建线程时的 ThreadLocal 传值的问题，但不可能一直创建新的线程，那实在耗费资源。因此通用做法是线程复用，比如线程池呗。但是，递交异步任务是相应的 ThreadLocal 的值就无法传递过去了。

我们希望的是，异步线程执行任务的所使用的 ThreadLocal 值，是将任务提交给线程时主线程持有的。即从任务创建时传递到任务执行时。

想想，如果我们在创建异步任务时，在任务代码外获取当前线程的值临时保存，再传递给执行线程，在真正的任务执行前保存到当前线程即可。对，确实可以，但是麻烦不？每个创建异步任务的地方都要写。

那就把它封装到递交任务的方法中。

# RunnableWrapper & CallableWrapper

假设按照服务上下文的场景举例，目前项目中的执行异步操作的方案是定义一个 AsyncExecutor ，并声明执行 Supplier 返回 CompletableFuture 的方法。

既然这样就可以对方法做一些改造，保证上下文的传递。



```dart
private static ThreadLocal<String> contextHolder = new ThreadLocal<>();

public static <T> CompletableFuture<T> invokeToCompletableFuture(Supplier<T> supplier, String errorMessage) {
    // 第一步
    String context = contextHolder.get();
    Supplier<T> newSupplier = () -> {
         // 第二步
        String origin = contextHolder.get();
        try {
            contextHolder.set(context);
            // 第三步
            return supplier.get();
        } finally {
            // 第四步
            contextHolder.set(origin);
            log.info(origin);
        }
    };
    return CompletableFuture.supplyAsync(newSupplier).exceptionally(e -> {
        throw new ServerErrorException(errorMessage, e);
    });
}
// test code
public static void main(String[] args) throws ExecutionException, InterruptedException {
    contextHolder.set("main");
    log.info(contextHolder.get());
    CompletableFuture<String> context = invokeToCompletableFuture(() -> test.contextHolder.get(), "error");
    log.info(context.get());
}
```

总得来说，就是在将异步任务派发给线程池时，对其做一下上下文传递的处理。

第一步：主线程获取上下文，传递给任务暂存。

1 之后的操作都将是异步执行线程操作的。

第二步：异步执行线程将原有上下文取出，暂时保存。并将主线程传递过来的上下文设置。

第三步：执行异步任务

第四步：将原有上下文设置回去。

可以看到一般并不会在异步线程执行完任务之后直接进行 remove 。而是一开始取出原上下文（可能为 NULL，也可能是线程创建时 InheritableThreadLocal 继承过来的值。当然后续也会被清除的），并在任务执行结束重新放回。这样的方式可以说是异步 ThreadLocal 传递的标准范式（大佬说的）。

这样子既起到了显式清除主线程带来的上下文，也避免了如果线程池的拒绝策略为 CallerRunsPolicy ，后续处理时上下文丢失的问题。

Supplier 不算是典型例子，更为典型的应该是 Runnable 和 Callable。不过举一推三，都是修饰一下，再丢给线程池。



```csharp
public final class DelegatingContextRunnable implements Runnable {

    private final Runnable delegate;

    private final Optional<String> delegateContext;

    public DelegatingContextRunnable(Runnable delegate,
                                       Optional<String> context) {
        assert delegate != null;
        assert context != null;

        this.delegate = delegate;
        this.delegateContext = context;
    }

    public DelegatingContextRunnable(Runnable delegate) {
        // 修饰原有的任务，并保存当前线程的值
        this(delegate, ContextHolder.get());
    }

    public void run() {
        Optional<String> originalContext = ContextHolder.get();

        try {
            ContextHolder.set(delegateContext);
            delegate.run();
        } finally {
            ContextHolder.set(originalContext);
        }
    }
}

public final void execute(Runnable task) {
  // 递交给真正的执行线程池前，对任务进行修饰
  executor.execute(wrap(task));
}

protected final Runnable wrap(Runnable task) {
  return new DelegatingContextRunnable(task);
}
```

后续，使用线程池执行异步任务的时候，事先对任务进行封装代理即可。

不过，还是比较麻烦。自定义的线程池，需要显式处理任务。而且更严谨的做法，不同业务场景之间的线程池应该是隔离的，以免受到影响，就比如 Hystrix 的线程池。

每一个线程池都要处理就麻烦了。所以换个思路，代理线程池。

# DelegaingExecutor

这个就不多说了，实际很简单，就照搬我们上下文相关类库。



```java
public class DelegatingContextExecutor implements Executor  {

    private final Executor delegate;


    public DelegatingContextExecutor(Executor delegateExecutor) {
        this.delegate = delegateExecutor;
    }

    public final void execute(Runnable task) {
        delegate.execute(wrap(task));
    }

    protected final Runnable wrap(Runnable task) {
        return new DelegatingContextRunnable(task);
    }

    protected final Executor getDelegateExecutor() {
        return delegate;
    }
}
// 自定义的线程池，用于执行项目中的异步任务
public Executor queryExecutor() {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor();
    // 封装服务上下文的线程池修饰
    return new DelegatingContextExecutorService(threadPoolExecutor);
}
```

问题似乎都解决了，那还有什么？

对，适用场景不够通用。上面的做法只针对于指定的 ThreadLocal，其他场景例如链路追踪、应用容器或上层框架跨应用代码给下层 SDK 传递信息（像是契约包 Feign 的执行线程）。

那么 TransmittableThreadLocal 就是为了解决通用化场景而设计的。

# TransmittableThreadLocal

作为一个核心代码不超过一千行的工具框架，实际使用和架构设计都十分简单。

其使用方法本质上与上述提到的 CallableWrapper 和 DelegatingExecutor 是一样的，并且为了方便使用，对外提供了静态工厂方法或工具类。



```csharp
public final void execute(Runnable task) {
  executor.execute(TtlCallable.get(task));
}
// 或者
public Executor queryExecutor() {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor();
    // 封装服务上下文的线程池修饰
    return TtlExecutors.getTtlExecutorService(threadPoolExecutor);
}
```

当然，前提是 ThreadLocal 必须使用 TransmittableThreadLocal。至于为什么，我们源码分析时再细细说来。

先看看核心实现类的结构，以 Callable 和 ExecutorService 为例。

![img](https:////upload-images.jianshu.io/upload_images/22943445-5e9526ea84028f20?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

全链路追踪必备组件之 TransmittableThreadLocal 详解

整体主要是三个部分：任务（ TtlCallable ）、线程池（ ExecutorServiceTtlWrapper ）、ThreadLocal（ TransmittableThreadLocal ）。其实对应上述讲到的 CallableWrapper、DelegatingExecutor、InheritableThreadLocal。

但是无论是任务和线程池，本身还是依赖于 TransmittableThreadLocal 对于存储值的管理。

用官方的时序图直观展示一下，框架是如何起作用的：

![img](https:////upload-images.jianshu.io/upload_images/22943445-48b08f9ef0f4e387?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

全链路追踪必备组件之 TransmittableThreadLocal 详解

可以看到，从第三步创建完任务，第四步修饰完任务，后续大部分过程都依赖于 TransmittableThreadLocal 或 TransmittableThreadLocal 中声明的静态工具类 Transmitter 。Transmitter 主要负责 ThreadLocal 的管理和值的传递。

首先看看 TtlCallable。

# TtlCallable

该类实际上是 JDK Callable 的一个修饰。类比于，上文讲到的 RunnableWrapper，只是为了临时保存父线程 ThreadLocal 的值，以便在执行任务之前，赋值到子线程中。

因此，TtlCallable 和 TtlExecutorService 都实现了 TtlWrapper 接口。也许你以为，该接口是实现修饰的语义，但是它只提供了一个方法，表达了拆修饰的语义：



```java
public interface TtlWrapper<T> extends TtlEnhanced {
    @NonNull
    T unwrap();
}
```

毕竟核心是修饰，所以该类主要为了提供修饰的核心抽象，便于框架对其进行判断和管理。

该方法语义要求，必须返回修饰的源对象或下层对象（毕竟可能修饰了很多层），因此也是空值安全的。null 进来，null 出去。



```java
public final class TtlCallable<V> implements Callable<V>, TtlWrapper<Callable<V>>, TtlEnhanced, TtlAttachments {
    // 保存父线程的 ThreadLocal 快照
    private final AtomicReference<Object> capturedRef;
    // 实际执行任务
    private final Callable<V> callable;
    // 判断是否执行完，清除任务所保存的 ThreadLocal 快照
    private final boolean releaseTtlValueReferenceAfterCall;

    private TtlCallable(@NonNull Callable<V> callable, boolean releaseTtlValueReferenceAfterCall) {
        // 1.创建时， 从 Transmitter 抓取快照
        this.capturedRef = new AtomicReference<Object>(capture());
        this.callable = callable;
        this.releaseTtlValueReferenceAfterCall = releaseTtlValueReferenceAfterCall;
    }

    @Override
    public V call() throws Exception {
        Object captured = capturedRef.get();
        // 如果 releaseTtlValueReferenceAfterCall 为 true，则在执行线程取出快照后清除。
        if (captured == null || releaseTtlValueReferenceAfterCall && !capturedRef.compareAndSet(captured, null)) {
            throw new IllegalStateException("TTL value reference is released after call!");
        }
                // 2.使用 Transmitter 将快照重做到当前执行线程，并将原来的值取出
        Object backup = replay(captured);
        try {
            // 3.执行任务
            return callable.call();
        } finally {
            // 4.Transmitter 重新将原值放回执行线程
            restore(backup);
        }
    }
}
```

可以看到，从实例化到任务执行的顺序，和上文讲到的 CallableWrapper 是完全一致的。但是在其之上，提供了更为完整的特性和线程安全性。

- releaseTtlValueReferenceAfterCall 的可控，保证了任务执行完，依然被业务代码持有的场景下，避免 ThreadLocal 快照继续持有而造成的内存泄漏。毕竟，对于业务方来说，这个东西是我不关心的，无需跟随任务本身的生命周期。
- 快照使用 AtomicReference 保存，保证任务误重用下，清除快照动作的多线程安全性。

上面两者的合用，相当于期望一个任务只能被执行一次，尽量避免任务重用和继续持有。

任务重用的间隔之间，可能出现 ThreadLocal 值被修改的情况，那么后一次任务执行时，快照实际是不准确的。业务场景应该尽量避免这种情况出现才对。

该类提供了静态工厂方法，方便业务方创建。



```tsx
public static <T> TtlCallable<T> get(@Nullable Callable<T> callable) {
    return get(callable, false);
}

@Nullable
public static <T> TtlCallable<T> get(@Nullable Callable<T> callable, boolean releaseTtlValueReferenceAfterCall) {
    return get(callable, releaseTtlValueReferenceAfterCall, false);
}

@Nullable
public static <T> TtlCallable<T> get(@Nullable Callable<T> callable, boolean releaseTtlValueReferenceAfterCall, boolean idempotent) {
    if (null == callable) return null;

    if (callable instanceof TtlEnhanced) {
        // avoid redundant decoration, and ensure idempotency
        if (idempotent) return (TtlCallable<T>) callable;
        else throw new IllegalStateException("Already TtlCallable!");
    }
    return new TtlCallable<T>(callable, releaseTtlValueReferenceAfterCall);
}
```

可以看到，默认工厂方法的
 releaseTtlValueReferenceAfterCall 是 false。如果想要使用执行完清除，就要注意方法的使用。

其次，这里还有一个幂等的参数控制： idempotent 。如果传入的 Callable 已经是修饰过的，那么根据 idempotent 的值，要么返回原 Callable，要么报错。

我觉得这里有个两难的点。

我们调用静态工厂方法期望得到的是调用该方法时 ThreadLocal 的快照。所以理论上，应该无论传入什么 Callable，都应该返回一个保存当前本地线程值快照的 TtlCallable。

但是，如果这样的逻辑下，传入的是已修饰的类，那么最后结果就是在任务执行时，会造成外层修饰的快照被内层修饰的覆盖。实际使用的是之前保存的快照了。

因此默认情况就只能 FastFail 。

官方并不建议设置 idempotent 为 true，因为直接返回原修饰类，本身也就违反静态工厂方法的语义。所以官方建议： <b>DO NOT</b> set, only when you know why.

# ExecutorServiceTtlWrapper

该类并不需要多讲，本身与上文的 DelegatingExecutor 一样。



```java
class ExecutorServiceTtlWrapper extends ExecutorTtlWrapper implements ExecutorService, TtlEnhanced {
    private final ExecutorService executorService;

    ExecutorServiceTtlWrapper(@NonNull ExecutorService executorService) {
        super(executorService);
        this.executorService = executorService;
    }

    @NonNull
    @Override
    public <T> Future<T> submit(@NonNull Callable<T> task) {
        return executorService.submit(TtlCallable.get(task));
    }
}
```

其余方法都是一样的做法。

从上文看到，实际 ThreadLocal 的线程传递的核心在于 TransmittableThreadLocal 和 Transmitter。

# TransmittableThreadLocal

TransmittableThreadLocal 只继承了 InheritableThreadLocal 和实现了该框架提供的函数接口 TtlCopier。

因此 TransmittableThreadLocal 自身是一个 InheritableThreadLocal，同样具备了线程创建时传递的特性。

其次，从类体系上看，TransmittableThreadLocal 自身是比较简单的，本质上只是为了让框架能够进行线程传递，做了一些小动作而已。

![img](https:////upload-images.jianshu.io/upload_images/22943445-f201b5562f57a635?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

全链路追踪必备组件之 TransmittableThreadLocal 详解

可以看到提供的方法是十分少的，源码行数总共也才不超过200行。

首先说一下构造函数。



```java
private final boolean disableIgnoreNullValueSemantics;

public TransmittableThreadLocal() {
    this(false);
}

public TransmittableThreadLocal(boolean disableIgnoreNullValueSemantics) {
    this.disableIgnoreNullValueSemantics = disableIgnoreNullValueSemantics;
}
```

一共两个构造函数，有参构造函数允许设置 “是否禁用忽略空值语义”。默认是开启的，表现行为是如果是 null 值，那么 TransmittableThreadLocal 是不会传递这个值，并且如果 set null，同时执行 remove 操作。表达的意思就是，“我不要 null，不归我管。你敢给我，我就再也不管你了“。

这样设计可能是因为一开始设计服务于业务，是希望业务不要通过 NULL 来表达任何含义，同时避免 NPE 和优化 GC。但是后来官方考虑到作为一个基础服务框架，应该尽量保证完整的语义。毕竟这样的特性是 JDK 的 ThreadLocal 不兼容的。因此后来，官方为了保证兼容性，加了控制参数，允许禁用该特性。

# TtlCopier

TransmittableThreadLocal 实现了一个类，TtlCopier。顾名思义，该类定义了线程传递时，值复制的抽象语义。



```java
public interface TtlCopier<T> {
    T copy(T parentValue);
}
```

而 TransmittableThreadLocal 的默认实现是与 InheritableThreadLocal 相同的，返回值的引用。



```cpp
public T copy(T parentValue) {
    return parentValue;
}
```

同时，该接口也为业务方留下了扩展点。开发者可以重写该方法，来定义线程传递时，如何进行值的复制。

TransmittableThreadLocal 内部维护了一个非常关键的属性，用来注册项目中维护的 TransmittableThreadLocal，从而保证 Transmitter 去正确传递 ThreadLocal 的值。



```tsx
private static InheritableThreadLocal<WeakHashMap<TransmittableThreadLocal<Object>, ?>> holder =
        new InheritableThreadLocal<WeakHashMap<TransmittableThreadLocal<Object>, ?>>() {
            @Override
            protected WeakHashMap<TransmittableThreadLocal<Object>, ?> initialValue() {
                return new WeakHashMap<TransmittableThreadLocal<Object>, Object>();
            }

            @Override
            protected WeakHashMap<TransmittableThreadLocal<Object>, ?> childValue(WeakHashMap<TransmittableThreadLocal<Object>, ?> parentValue) {
                return new WeakHashMap<TransmittableThreadLocal<Object>, Object>(parentValue);
            }
        };
```

holder 是一个 InheritableThreadLocal，用来保存所有注册的 TransmittableThreadLocal。父子线程传递时，可以直接将父线程的注册表传递过来。使用 InheritableThreadLocal，主要保证了嵌套线程场景下，注册表的正确传递。官方有个 issue 以及为其 fix 的 release 版本，从 ThreadLocal 改成了 InheritableThreadLocal。嵌入Thread调用的bug

其次，存储的是 WeakHashMap ，value 都是无意义的 null，并且永远不会被使用。这样一来，保证项目使用 TransmittableThreadLocal 的话，不会引入新的内存泄漏问题。其内存泄漏的可能风险，就只完全来自于 InheritableThreadLocal 本身。



```csharp
@Override
public final T get() {
    T value = super.get();
    if (disableIgnoreNullValueSemantics || null != value) addThisToHolder();
    return value;
}

@Override
public final void set(T value) {
    if (!disableIgnoreNullValueSemantics && null == value) {
        // may set null to remove value
        remove();
    } else {
        super.set(value);
        addThisToHolder();
    }
}

@Override
public final void remove() {
    removeThisFromHolder();
    super.remove();
}

@SuppressWarnings("unchecked")
private void addThisToHolder() {
    if (!holder.get().containsKey(this)) {
        holder.get().put((TransmittableThreadLocal<Object>) this, null); // WeakHashMap supports null value.
    }
}

private void removeThisFromHolder() {
    holder.get().remove(this);
}
```

get & set 会将当前的 TransmittableThreadLocal 注册到 holder 中， remove 时，会删除对应注册。

可以看到，前文说到的
 disableIgnoreNullValueSemantics 的值在 get 和 set 时使用到。默认为 false 时，ThreadLocal 不会保存 null，holder 不会注册对应的 TransmittableThreadLocal。

TransmittableThreadLocal 就这样没了，可以看到就很简单。但是，线程传递的内容呢，为什么没有？

这是因为，TransmittableThreadLocal 将线程传递的所有工作全部委托给了其静态内部类 Transmitter。

# Transmitter

我们讲到 TransmittableThreadLocal 会将有值的对象，注册到 holder 中，以便 Transmitter 去知道传递哪一些实例的值。但是如果这样，那不是都要修改代码，将项目中的 ThreadLocal 都改掉吗？

这当然不可能，因此 Transmitter 承担了这个任务，允许业务代码将原有的 ThreadLocal 注册进来，以方便 Transmitter 来识别和传递。



```tsx
// 注册 ThreadLocal 的 threadLocalHolder 依然是 WeakHashMap
private static volatile WeakHashMap<ThreadLocal<Object>, TtlCopier<Object>> threadLocalHolder = new WeakHashMap<ThreadLocal<Object>, TtlCopier<Object>>();
// ThreadLocal 手动注册时用的锁
private static final Object threadLocalHolderUpdateLock = new Object();
// 标记 ThreadLocal 的值已清除，类似于设置一个 null
private static final Object threadLocalClearMark = new Object();
// 传递 TtlCopier，来确定 threadLocal 传递值的方式。默认是 引用传递，与 TransmittableThreadLocal 的 copy 一致。
public static <T> boolean registerThreadLocal(@NonNull ThreadLocal<T> threadLocal, @NonNull TtlCopier<T> copier) {
    return registerThreadLocal(threadLocal, copier, false);
}

@SuppressWarnings("unchecked")
public static <T> boolean registerThreadLocalWithShadowCopier(@NonNull ThreadLocal<T> threadLocal) {
    // 默认是内部定义个 shadowCopier
    return registerThreadLocal(threadLocal, (TtlCopier<T>) shadowCopier, false);
}

public static <T> boolean registerThreadLocal(@NonNull ThreadLocal<T> threadLocal, @NonNull TtlCopier<T> copier, boolean force) {
    // 如果是 TransmittableThreadLocal，则没有必要再维护了。默认就实现了其的传递。
    if (threadLocal instanceof TransmittableThreadLocal) {
        logger.warning("register a TransmittableThreadLocal instance, this is unnecessary!");
        return true;
    }
        
    synchronized (threadLocalHolderUpdateLock) {
        // force 为 false，则不会更新对应的 copier
        if (!force && threadLocalHolder.containsKey(threadLocal)) return false;
                // copy on write
        WeakHashMap<ThreadLocal<Object>, TtlCopier<Object>> newHolder = new WeakHashMap<ThreadLocal<Object>, TtlCopier<Object>>(threadLocalHolder);
        newHolder.put((ThreadLocal<Object>) threadLocal, (TtlCopier<Object>) copier);
        threadLocalHolder = newHolder;
        return true;
    }
}

public static <T> boolean registerThreadLocalWithShadowCopier(@NonNull ThreadLocal<T> threadLocal, boolean force) {
    return registerThreadLocal(threadLocal, (TtlCopier<T>) shadowCopier, force);
}
// 清除 ThreadLocal 的注册
public static <T> boolean unregisterThreadLocal(@NonNull ThreadLocal<T> threadLocal) {
    if (threadLocal instanceof TransmittableThreadLocal) {
        logger.warning("unregister a TransmittableThreadLocal instance, this is unnecessary!");
        return true;
    }

    synchronized (threadLocalHolderUpdateLock) {
        if (!threadLocalHolder.containsKey(threadLocal)) return false;

        WeakHashMap<ThreadLocal<Object>, TtlCopier<Object>> newHolder = new WeakHashMap<ThreadLocal<Object>, TtlCopier<Object>>(threadLocalHolder);
        newHolder.remove(threadLocal);
        threadLocalHolder = newHolder;
        return true;
    }
}
// 默认实现的 TtlCopier，直接引用传递
private static final TtlCopier<Object> shadowCopier = new TtlCopier<Object>() {
    @Override
    public Object copy(Object parentValue) {
        return parentValue;
    }
};
```

其实我自己有个想不明白的，既然已经用了
 threadLocalHolderUpdateLock 做锁，为什么还要用 copy on write？GC 友好？mark 一下。

剩下的部分，就是 Transmitter 怎么传递 ThreadLocal 的值了。

实际就是三个步骤，capture -> reply -> restore，crr。

# 1.抓取当前线程的值快照



```dart
// 快照类，用来保存当前线程的 TtlThreadLocal 和 ThreadLocal 的快照
private static class Snapshot {
    final WeakHashMap<TransmittableThreadLocal<Object>, Object> ttl2Value;
    final WeakHashMap<ThreadLocal<Object>, Object> threadLocal2Value;

    private Snapshot(WeakHashMap<TransmittableThreadLocal<Object>, Object> ttl2Value, WeakHashMap<ThreadLocal<Object>, Object> threadLocal2Value) {
        this.ttl2Value = ttl2Value;
        this.threadLocal2Value = threadLocal2Value;
    }
}

public static Object capture() {
    // 抓取快照
    return new Snapshot(captureTtlValues(), captureThreadLocalValues());
}
// 抓取 TransmittableThreadLocal 的快照
private static WeakHashMap<TransmittableThreadLocal<Object>, Object> captureTtlValues() {
    WeakHashMap<TransmittableThreadLocal<Object>, Object> ttl2Value = new WeakHashMap<TransmittableThreadLocal<Object>, Object>();
    // 从 TransmittableThreadLocal 的 holder 中，遍历所有有值的 TransmittableThreadLocal，将 TransmittableThreadLocal 取出和值复制到 Map 中。
    for (TransmittableThreadLocal<Object> threadLocal : holder.get().keySet()) {
        ttl2Value.put(threadLocal, threadLocal.copyValue());
    }
    return ttl2Value;
}

//  抓取注册的 ThreadLocal。
private static WeakHashMap<ThreadLocal<Object>, Object> captureThreadLocalValues() {
    final WeakHashMap<ThreadLocal<Object>, Object> threadLocal2Value = new WeakHashMap<ThreadLocal<Object>, Object>();
    // 从 threadLocalHolder 中，遍历注册的 ThreadLocal，将 ThreadLocal 和 TtlCopier 取出，将值复制到 Map 中。
    for (Map.Entry<ThreadLocal<Object>, TtlCopier<Object>> entry : threadLocalHolder.entrySet()) {
        final ThreadLocal<Object> threadLocal = entry.getKey();
        final TtlCopier<Object> copier = entry.getValue();

        threadLocal2Value.put(threadLocal, copier.copy(threadLocal.get()));
    }
    return threadLocal2Value;
}
```

# 2.将快照重做到执行线程



```dart
@NonNull
public static Object replay(@NonNull Object captured) {
    final Snapshot capturedSnapshot = (Snapshot) captured;
    return new Snapshot(replayTtlValues(capturedSnapshot.ttl2Value), replayThreadLocalValues(capturedSnapshot.threadLocal2Value));
}

// 重播 TransmittableThreadLocal，并保存执行线程的原值
@NonNull
private static WeakHashMap<TransmittableThreadLocal<Object>, Object> replayTtlValues(@NonNull WeakHashMap<TransmittableThreadLocal<Object>, Object> captured) {
    WeakHashMap<TransmittableThreadLocal<Object>, Object> backup = new WeakHashMap<TransmittableThreadLocal<Object>, Object>();
  
    for (final Iterator<TransmittableThreadLocal<Object>> iterator = holder.get().keySet().iterator(); iterator.hasNext(); ) {
        TransmittableThreadLocal<Object> threadLocal = iterator.next();

        // 遍历 holder，从 父线程继承过来的,或者之前注册进来的
        backup.put(threadLocal, threadLocal.get());

        // clear the TTL values that is not in captured
        // avoid the extra TTL values after replay when run task
        // 清除本次没有传递过来的 ThreadLocal，和对应值。毕竟一是可能会有因为 InheritableThreadLocal 而传递并保留的值。二来保证主线程 set 过的 ThreadLocal，不应该被传递过来。明确，其传递是由业务代码控制的，就是明确 set 过值的。
        if (!captured.containsKey(threadLocal)) {
            iterator.remove();
            threadLocal.superRemove();
        }
    }

    // 将 map 中的值，设置到 ThreadLocal 中。
    setTtlValuesTo(captured);

    // TransmittableThreadLocal 的回调方法，在任务执行前执行。
    doExecuteCallback(true);

    return backup;
}

private static void setTtlValuesTo(@NonNull WeakHashMap<TransmittableThreadLocal<Object>, Object> ttlValues) {
    for (Map.Entry<TransmittableThreadLocal<Object>, Object> entry : ttlValues.entrySet()) {
        TransmittableThreadLocal<Object> threadLocal = entry.getKey();
        // set 的同时，也就将 TransmittableThreadLocal 注册到当前线程的注册表了。
        threadLocal.set(entry.getValue());
    }
}

private static WeakHashMap<ThreadLocal<Object>, Object> replayThreadLocalValues(@NonNull WeakHashMap<ThreadLocal<Object>, Object> captured) {
    final WeakHashMap<ThreadLocal<Object>, Object> backup = new WeakHashMap<ThreadLocal<Object>, Object>();

    for (Map.Entry<ThreadLocal<Object>, Object> entry : captured.entrySet()) {
        final ThreadLocal<Object> threadLocal = entry.getKey();
        backup.put(threadLocal, threadLocal.get());

        final Object value = entry.getValue();
        // 如果值是标记已删除，则清除
        if (value == threadLocalClearMark) threadLocal.remove();
        else threadLocal.set(value);
    }

    return backup;
}
```

doExecuteCallback 是 TransmittableThreadLocal 定义的回调方法，保证任务执行前和执行后的回调动作。

isBefore 控制是执行前还是执行后。

内部调用了 beforeExecute 和 afterExecute 方法。默认是不做任何动作。



```cpp
private static void doExecuteCallback(boolean isBefore) {
    for (TransmittableThreadLocal<Object> threadLocal : holder.get().keySet()) {
        try {
            if (isBefore) threadLocal.beforeExecute();
            else threadLocal.afterExecute();
        } catch (Throwable t) {
            // 忽略所有异常，保证任务的执行
            if (logger.isLoggable(Level.WARNING)) {
                logger.log(Level.WARNING, "TTL exception when " + (isBefore ? "beforeExecute" : "afterExecute") + ", cause: " + t.toString(), t);
            }
        }
    }
}
protected void beforeExecute() {
}

protected void afterExecute() {
}
```

# 3.恢复备份的原快照



```dart
public static void restore(@NonNull Object backup) {
    final Snapshot backupSnapshot = (Snapshot) backup;
    restoreTtlValues(backupSnapshot.ttl2Value);
    restoreThreadLocalValues(backupSnapshot.threadLocal2Value);
}

private static void restoreTtlValues(@NonNull WeakHashMap<TransmittableThreadLocal<Object>, Object> backup) {
    // call afterExecute callback 任务执行完回调
    doExecuteCallback(false);

    for (final Iterator<TransmittableThreadLocal<Object>> iterator = holder.get().keySet().iterator(); iterator.hasNext(); ) {
        TransmittableThreadLocal<Object> threadLocal = iterator.next();

        // clear the TTL values that is not in backup
        // avoid the extra TTL values after restore
        // 恢复快照时，清除本次传递注册进来，但是原先不存在的 TransmittableThreadLocal
        if (!backup.containsKey(threadLocal)) {
            iterator.remove();
            threadLocal.superRemove();
        }
    }

    // restore TTL values
    // 恢复快照中的 value 到 TransmittableThreadLocal 中
    setTtlValuesTo(backup);
}

private static void setTtlValuesTo(@NonNull WeakHashMap<TransmittableThreadLocal<Object>, Object> ttlValues) {
    for (Map.Entry<TransmittableThreadLocal<Object>, Object> entry : ttlValues.entrySet()) {
        TransmittableThreadLocal<Object> threadLocal = entry.getKey();
        threadLocal.set(entry.getValue());
    }
}

private static void restoreThreadLocalValues(@NonNull WeakHashMap<ThreadLocal<Object>, Object> backup) {
    for (Map.Entry<ThreadLocal<Object>, Object> entry : backup.entrySet()) {
        final ThreadLocal<Object> threadLocal = entry.getKey();
        threadLocal.set(entry.getValue());
    }
}
```

# 对特殊场景以及 Lambda 的支持

Transmitter 定义了几个特殊场景下以及 Java 8 lambda 表达式的使用。

特殊场景就是指，执行前，清除当前执行线程 ThreadLocal 的值，包括 TtlThreadLocal 和注册 ThreadLocal 。

像一开始讲到的业务代码喜欢使用 Supplier，所以也对其做了支持。本质是为了简化工作。

不过，注意的是，快照的捕获则需要业务代码自己完成并传递。



```tsx
public static <R> R runSupplierWithCaptured(@NonNull Object captured, @NonNull Supplier<R> bizLogic) {
    Object backup = replay(captured);
    try {
        return bizLogic.get();
    } finally {
        restore(backup);
    }
}

public static <R> R runSupplierWithClear(@NonNull Supplier<R> bizLogic) {
    Object backup = clear();
    try {
        return bizLogic.get();
    } finally {
        restore(backup);
    }
}

public static <R> R runCallableWithCaptured(@NonNull Object captured, @NonNull Callable<R> bizLogic) throws Exception {
    Object backup = replay(captured);
    try {
        return bizLogic.call();
    } finally {
        restore(backup);
    }
}

public static <R> R runCallableWithClear(@NonNull Callable<R> bizLogic) throws Exception {
    Object backup = clear();
    try {
        return bizLogic.call();
    } finally {
        restore(backup);
    }
}
```

简化方法，使用起来也就是：



```dart
// 线程A
Object captured = Transmitter.capture();

// 线程B
@Async
String result = runSupplierWithCaptured(captured, () -> {
  
     System.out.println("Hello");
     ...
     return "World";
});
```

否则只能按照全套流程了：



```kotlin
// 线程A
Object captured = Transmitter.capture();

// 线程B
@Async
String result = runSupplierWithCaptured(captured, () -> {
  
     System.out.println("Hello");
     ...
     return "World";
});
Object backup = Transmitter.replay(captured); // (2)
try {
    System.out.println("Hello");
    // ...
    return "World";
} finally {
    // restore the TransmittableThreadLocal of thread B when replay
    Transmitter.restore(backup); (3)
```

# Clear

上面可以看到，一些方法是做了 clear 操作。

就是不依赖快照的捕获，将空值的快照信息，传递给重做方法执行，就能清除当前执行线程的值，并得到返回原值备份。



```dart
public static Object clear() {
    final WeakHashMap<TransmittableThreadLocal<Object>, Object> ttl2Value = new WeakHashMap<TransmittableThreadLocal<Object>, Object>();

    final WeakHashMap<ThreadLocal<Object>, Object> threadLocal2Value = new WeakHashMap<ThreadLocal<Object>, Object>();
    for (Map.Entry<ThreadLocal<Object>, TtlCopier<Object>> entry : threadLocalHolder.entrySet()) {
        final ThreadLocal<Object> threadLocal = entry.getKey();
        // threadLocalClearMark 标记为未被传递和注册，更为合适，从而避免和 null 混淆。否则无法区分原有就是 null，还是未被注册
        threadLocal2Value.put(threadLocal, threadLocalClearMark);
    }

    return replay(new Snapshot(ttl2Value, threadLocal2Value));
}
```

# 注意

如果注意到 TransmittableThreadLocal 是继承 InheritableThreadLocal，就应该知道，子线程创建时，值还是会被传递过去。这也就可能带来内存泄漏问题。

所以，同时提供
 DisableInheritableThreadFactoryWrapper，以方便业务代码自定义线程池，禁止值的继承传递。



```java
class DisableInheritableThreadFactoryWrapper implements DisableInheritableThreadFactory {
    private final ThreadFactory threadFactory;

    DisableInheritableThreadFactoryWrapper(@NonNull ThreadFactory threadFactory) {
        this.threadFactory = threadFactory;
    }

    @Override
    public Thread newThread(@NonNull Runnable r) {
        // 调用了 Transmitter 的 clear 方法，在创建子线程前，清除当前线程的值，并保存下来
        final Object backup = clear();
        try {
            return threadFactory.newThread(r);
        } finally {
            // 创建完，再重新恢复。以此，避免了值的继承传递。
            restore(backup);
        }
    }

    @NonNull
    @Override
    public ThreadFactory unwrap() {
        return threadFactory;
    }
}
```

对于 1.8 特性，还提供了
 ForkJoinWorkerThreadFactory 和 TtlForkJoinPoolHelper 等类的支持。

# Java Agent 支持

避免代码改动的话，可以使用 Java Agent，来隐式替换 JDK 的相应类。对于 1.8 的 CompletableFuture 和 Stream，在底层通过对 ForkJoinPool 的支持，也做了透明支持。

# 总结

到此，TransmittableThreadLocal 的源码解析就结束了。核心源码是不是很简单？但是某些思想和考量还是很值得学习的。

ThreadLocal 的使用，本身类似于全局变量，而且是可修改的。一旦中间过程被修改，就无法保证整体流程的前后一致性。它将是一个隐藏的强依赖，一个可能被忽略、意想不到的坑。（我不承认，我在还原大佬的话。）

应该尽量避免在业务代码中使用的。 **DO NOT** use, only when you know why .

嗯，还有加上一句，让其他人也明白，文档务必齐全。（说实话，我挺想用英文的，想想算了）。

# skywalking线程池插件,解决lambda使用问题

1. 简介
   分布式链路追踪其实原理都很简单，每一个请求过来的时候，把一个context存到ThreadLocal里面，然后随着线程一路传递下去就行，一个请求大多数时候都是一个线程去执行下去所以并没有什么问题，但是如果在请求中使用了ThreadPool那么ThreadLocal中的context就会失联(如果使用的是Thread，ThreadLocal是可以继续传递下去的，具体原因可以去看ThreadLocal和ThreadPool的实现这里就不过多解释了)，所以需要插件的支持去把这个失联的context给找回来

2. 官方插件
   skywalking 的跨线程池 其实是已经有官方插件的支持了。实现的原理也是非常简单，他会去织入业务方自己实现的Runnable类，在执行run()方法之前去传递context。

使用方法：
修改 skywalking-agent.jar所在目录的 config/agent.config文件，在里面修改/添加配置
#他将会去把指定包下所有`Runnable`的实现类给织入，可以前缀匹配
jdkthreading.threading_class_prefixes=com.chy
1
2
然后在plugins文件夹里加入插件apm-jdk-threading-plugin.jar就能够使用了(如果是下载官方编译好的插件，apm-jdk-threading-plugin.jar被放在了bootstrap-plugins文件夹中，移动到plugins即可)

缺点：
虽然简单但是却有一个缺点，无法使用lambda表达式，比如下面的代码，就无法正常的把链路给连接起来

@GetMapping("/ajax/{setId}/async")
    public List<DoctorListVO> getDoctorListasync(@PathVariable Integer setId) throws ExecutionException, InterruptedException {
        

        //线程池访问
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<List<DoctorListVO>> submit = executor.submit(() -> {
            //远程调用了别的服务
            return doctorSetAdminService.queryDoctorList(setId);
        });
        Future<List<DoctorListVO>> submit2 = executor.submit(() -> {
            return doctorSetAdminService.queryDoctorList(setId);
        });
    
        //和上面 queryDoctorList 一样只是queryDoctorListAsy() 方法都是打了 `@Async` 注解
        doctorSetAdminService.queryDoctorListAsy(setId);
        doctorSetAdminService.queryDoctorListAsy(setId);


        List<DoctorListVO> doctorListVOS = submit.get();
        List<DoctorListVO> doctorListVOS2 = submit2.get();
        return doctorListVOS;
    }

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
简单说明一下原因，上面提到过这个插件的原理是去修改所有Runnable的实现类，但是如果使用lambda去实现的Runnable类，虽然会生成一个匿名内部类，但是这个匿名内部类的和skywalking-agent使用的不同的类加载器，导致skywalking-agent无法去修改lambda表达式生成的Runnable的实现类.

3. 非官方插件
   为了解决lambda的问题，博主自己写了一个插件去解决这个问题，思路也很简单，既然我不能去修改lambda生成的类，那么我就换个地方去修改他，比如在调用的时候再去修改。
   所以我把切入点放在了ThreadPoolExecutor的execute(Runnable r)方法上面，也就是每当执行execute(Runnable r)方法的时候，我都会拦截下这个方法，然后把他的入参的Runnable给换成我的代理对象，然后继续去执行execute(Runnable r)方法

话不多说，直接上插件的地址，使用方法在readme里有写

github
gitee
这里需要注意的是，除了去把插件放入plugins文件夹外，还需要替换 skywalking-agent.jar 文件,原因在于博主的切入点是ThreadPoolExecutor 需要在字节码框架上做一些额外的配置。

还是执行这个代码

@GetMapping("/ajax/{setId}/async")
    public List<DoctorListVO> getDoctorListasync(@PathVariable Integer setId) throws ExecutionException, InterruptedException {
        

        //线程池访问
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<List<DoctorListVO>> submit = executor.submit(() -> {
            //远程调用了别的服务
            return doctorSetAdminService.queryDoctorList(setId);
        });
        Future<List<DoctorListVO>> submit2 = executor.submit(() -> {
            return doctorSetAdminService.queryDoctorList(setId);
        });
    
        //和上面 queryDoctorList 一样只是queryDoctorListAsy() 方法都是打了 `@Async` 注解
        doctorSetAdminService.queryDoctorListAsy(setId);
        doctorSetAdminService.queryDoctorListAsy(setId);


        List<DoctorListVO> doctorListVOS = submit.get();
        List<DoctorListVO> doctorListVOS2 = submit2.get();
        return doctorListVOS;
    }

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
结果如下



# 基于Dubbo的灰度发布

**前言**

保证系统的高可用和稳定性是互联网应用的基本要求。需求变化、版本迭代势必会影响系统的稳定性和可用性，如何兼顾需求变化和系统稳定呢？这个影响它的因素很多，发布是其中一个。我们要尝试尽可能让发布平滑、让新功能曝光、影响人群由少到多和由内部到外部、一旦有问题马上回滚等。

**灰度发布**

什么是灰度发布？看看百度百科的解释：灰度发布（又名金丝雀发布）是指在黑与白之间，能够平滑过渡的一种发布方式。在其上可以进行A/B testing，即让一部分用户继续用产品特性A，一部分用户开始用产品特性B，如果用户对B没有什么反对意见，那么逐步扩大范围，把所有用户都迁移到B上面来。灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。灰度发布开始到结束期间的这一段时间，称为灰度期。

这个解释很到位。

**Dubbo对灰度发布的支持**

实现灰度发布，就要在请求到达服务提供者前，能将请求按预设的规则去访问目标服务提供者。要做到这一点，就要在消费端或者在服务集群之前的类似网关这样的中间层做好路由。我们知道Dubbo调用是端到端的，不存在一个中间层，所以，在Dubbo的消费端就要做好请求路由。我们先看一下Dubbo的调用链路，如下图

![img](https://pic4.zhimg.com/80/v2-a7dceacc436f69355070f69e1c5af9d3_1440w.jpg)Dubbo调用链路——来自Dubbo官网

如上图所示，在Dubbo消费端，已经做了负载均衡了，但负载均衡不能满足我们对请求的路由需求。其实在负载均衡之前，invocation会先经过一个路由器链并返回一组合适的目标服务器地址列表，然后在这些备用的服务器地址中做负载均衡。源码如下：

![img](https://pic1.zhimg.com/80/v2-be17ba1ce29e18975b282a4f56f24230_1440w.jpg)AbstractClusterInvoker.invoke

所以只需要增加一个路由器，在路由器定义灰度的规则，就可以实现灰度发布了。

Dubbo原生已经按这种思路做了路由的支持了，它现在支持两种路由方式：条件路由、标签路由。

条件路由主要支持以服务或Consumer应用为粒度配置路由规则，源码对应的路由器是ConditionRouter.java。

标签路由主要是以Provider应用为粒度配置路由规则，通过将某一个或多个服务的提供者划分到同一个分组，约束流量只在指定分组中流转，从而实现流量隔离的目的，对应源码路由器是TagRouter.java。

具体的规则可以查阅文档：[http://dubbo.apache.org/zh-cn/docs/2.7/user/demos/routing-rule/](https://link.zhihu.com/?target=http%3A//dubbo.apache.org/zh-cn/docs/2.7/user/demos/routing-rule/)

Dubbo原生路由器支持的场景其实已经很丰富的了，比如条件路由器，可以设置排除预发布机器、指定Consumer应用访问指定的Provider应用，这其实就是蓝绿发布，还能设置黑白名单、按网段访问等等。但是这有一些问题，谁来管理这些规则，什么时候来管理。标签路由问题就更大一些，在配置了标签之后，还要侵入代码写入标签，至少也要在启动参数加上这个标签。使用是比较麻烦的。**现在很多公司已经在落地Devops，或者使用商业开发平台如EDAS，有的也会搭建自己的运维平台，发布流程都在追求自动化、半自动化，所以易用性很重要。**实现一个流畅的灰度发布流程，我们希望它能有一个统一的运维平台，能支持条件路由和权重路由，可以随时监控、回滚、或者继续发布。这样，我们有必要自定义一个路由器，并将其融入发布流程中。

**自定义路由器**

上篇文章我们介绍了Dubbo SPI的实现，这让Dubbo的扩展成本非常低，我们只需定义自己的路由器并把整合到应用就可以了，路由扩展文档请查阅文档：[http://dubbo.apache.org/zh-cn/docs/2.7/dev/impls/router/](https://link.zhihu.com/?target=http%3A//dubbo.apache.org/zh-cn/docs/2.7/dev/impls/router/)

自定义路由器就叫GrayRouter吧，规划一下它的功能，如下图

![img](https://pic3.zhimg.com/80/v2-baea1e01a0fa1c8a43b498ded1a6c0d2_1440w.jpg)GrayRouter功能

灰度路由器是以provider应用为粒度配置路由规则的，包含两个过滤器，条件过滤器和权重过滤器（不是Dubbo的过滤器），它们的主要任务根据配置过滤出备用provoider。配置信息放在分布式配置上，首次加载放入应用缓存，一旦有变更将自动更新。这样在发布时可以动态调整灰度参数，达到逐步扩大流量和影响人群的目的。下面是路由器的主要实现代码

![img](https://pic1.zhimg.com/80/v2-d6b318a59026823213cf671495e22dec_1440w.jpg)

![img](https://pic3.zhimg.com/80/v2-62ef19b25c5fd10db7d134f2d1eaa4b6_1440w.jpg)

![img](https://pic1.zhimg.com/80/v2-1dfdb59c59346d67b49ed6db9ea11448_1440w.jpg)

![img](https://pic2.zhimg.com/80/v2-9e279354e1ad55fa2fb5f10603a6ded9_1440w.jpg)

**发布流程可视化**

![img](https://pic4.zhimg.com/80/v2-5a000f6ee4b0e3c404ae0ca511e5736f_1440w.jpg)

![img](https://pic2.zhimg.com/80/v2-c2d21651ae49f0838e9233002ffb9a01_1440w.jpg)

![img](https://pic4.zhimg.com/80/v2-97ac73b13d3b3d2ef5f5e78502deb17b_1440w.jpg)

**监控**

我们使用了Skywalking做应用性能监控，下图是一个有两个节点的应用，灰度发布配置了权重发布，权重值10（全部是100），灰度流量分布效果如下图

![img](https://pic3.zhimg.com/80/v2-f3c67d8f30a3b6e5835733be47fecdd6_1440w.jpg)

**结束**

灰度发布在版本迭代中给系统稳定提供了有力的保证，应该将其纳入到发布流程中。

# [Springboot2.3+Dubbo2.7.3实现灰度跳转](https://www.cnblogs.com/penghq/p/13093812.html)

**1、jar包依赖**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- Aapche Dubbo相关 start  -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.11</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- Aapche Dubbo相关 end -->
</dependencies>  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**2、自定义LoadBalance**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.pacmp.config.balance;



import lombok.extern.slf4j.Slf4j;
import org.apache.dubbo.common.URL;
import org.apache.dubbo.rpc.Invocation;
import org.apache.dubbo.rpc.Invoker;
import org.apache.dubbo.rpc.cluster.loadbalance.AbstractLoadBalance;
import org.springframework.stereotype.Component;

import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

@Slf4j
@Component
public class GrayLoadBalance extends AbstractLoadBalance {

    public static final String NAME = "gray";

    public GrayLoadBalance() {
        log.info("初始化GrayLoadBalance成功！");
    }

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        List<Invoker<T>> list = new ArrayList<>();
        for (Invoker invoker : invokers) {
            list.add(invoker);
        }
        Map<String, String> map = invocation.getAttachments();
        String ifGary = map.get("ifGary");
        String userId = map.get("userId");
        log.info("userId:"+userId+"=====ifGary:"+ifGary);
        Iterator<Invoker<T>> iterator = list.iterator();
        while (iterator.hasNext()) {
            Invoker<T> invoker = iterator.next();
            String providerStatus = invoker.getUrl().getParameter("status", "prod");
            if (Objects.equals(providerStatus, NAME)) {
                if ("1".equals(ifGary)) {
                    log.info("userId:"+userId+"=====ifGary:"+ifGary+"=====去灰度服务");
                    return invoker;
                } else {
                    log.info("userId:"+userId+"=====ifGary:"+ifGary+"=====去正常服务");
                    iterator.remove();
                }
            }
        }
        return this.randomSelect(list, url, invocation);
    }


    /**
     * 重写了一遍随机负载策略
     *
     * @param invokers
     * @param url
     * @param invocation
     * @param <T>
     * @return
     */
    private <T> Invoker<T> randomSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        boolean sameWeight = true;
        int[] weights = new int[length];
        int firstWeight = this.getWeight((Invoker) invokers.get(0), invocation);
        weights[0] = firstWeight;
        int totalWeight = firstWeight;

        int offset;
        int i;
        for (offset = 1; offset < length; ++offset) {
            i = this.getWeight((Invoker) invokers.get(offset), invocation);
            weights[offset] = i;
            totalWeight += i;
            if (sameWeight && i != firstWeight) {
                sameWeight = false;
            }
        }

        if (totalWeight > 0 && !sameWeight) {
            offset = ThreadLocalRandom.current().nextInt(totalWeight);

            for (i = 0; i < length; ++i) {
                offset -= weights[i];
                if (offset < 0) {
                    return (Invoker) invokers.get(i);
                }
            }
        }
        return (Invoker) invokers.get(ThreadLocalRandom.current().nextInt(length));
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**3、在resources加配置文件，路径如下图（路径必须一致）**

**文件名：org.apache.dubbo.rpc.cluster.LoadBalance**

 

 

 

**![img](https://img2020.cnblogs.com/blog/1179838/202006/1179838-20200611154418753-1649818507.png)**

 

```
添加GrayLoadBalance类的路径
```

![img](https://img2020.cnblogs.com/blog/1179838/202006/1179838-20200611154625135-738714939.png)

**4、application.yml配置**

**消费者配置：**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# dubbo
dubbo:
  application:
    name: dubbo_consumer
  registry:
    address: zookeeper://127.0.0.1:2181
  scan:
    base-packages: com.pacmp.controller
  consumer:
    version: 2.0.0
  provider:
    loadbalance: gray
  protocol:
    port: 10000
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**生产者配置：**

```
loadbalance: gray   表示加载自定义loadbalance：com.pacmp.config.balance.GrayLoadBalance。
parameters:
      status: gray  表示这个生产者（服务），是否为灰度。如果不是灰度可以不用配置。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
## Dubbo配置
dubbo:
  application:
    name: dubbo_provider
  registry:
    address: zookeeper://127.0.0.1:2181
  protocol:
    name: dubbo
    port: -1
  scan:
    base-packages: com.pacmp
  provider:
    loadbalance: gray
    version: 2.0.0
    parameters:
      status: gray
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**5、调用示例**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.pacmp.controller;


import com.pacmp.service.DemoService;
import lombok.extern.slf4j.Slf4j;


import org.apache.dubbo.config.annotation.Reference;
import org.apache.dubbo.rpc.RpcContext;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.*;

/**
 * @Author xxx
 * @Date 2020/05/26 9:52
 * @Version 1.0
 * @Description 接入层
 */
@Slf4j
@RestController
@RequestMapping("/api")
public class ApiController {

    @Reference(check = false)
    private DemoService demoService;

    @GetMapping("/testUser")
    public String testUser(int userId, String version) {
        //ifGary=1代表灰度用户，0代表普通用户
        int ifGary = 0;
        if(userId<10){
            ifGary = 1;
        }
        RpcContext.getContext().setAttachment("ifGary", String.valueOf(ifGary));
        RpcContext.getContext().setAttachment("userId", String.valueOf(userId));
        return demoService.testUser(userId, version);
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**6、测试**

启2个生产者服务，1个消费者服务。可根据userId的不同来调用生产服务/灰度服务。

## Skywalking字节码插桩

10、静态方法插桩
Transform的transform()方法中调用每个插件的define()方法去做字节码增强，AbstractClassEnhancePluginDefine的define()方法中再调用自己的enhance()方法做字节码增强，enhance()方法源码如下：

public abstract class AbstractClassEnhancePluginDefine {

    /**
     * Begin to define how to enhance class. After invoke this method, only means definition is finished.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected DynamicType.Builder<?> enhance(TypeDescription typeDescription, DynamicType.Builder<?> newClassBuilder,
                                             ClassLoader classLoader, EnhanceContext context) throws PluginException {
        // 静态方法插桩
        newClassBuilder = this.enhanceClass(typeDescription, newClassBuilder, classLoader);
    
        // 构造器和实例方法插桩
        newClassBuilder = this.enhanceInstance(typeDescription, newClassBuilder, classLoader, context);
    
        return newClassBuilder;
    }
      
    /**
     * Enhance a class to intercept class static methods.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected abstract DynamicType.Builder<?> enhanceClass(TypeDescription typeDescription, DynamicType.Builder<?> newClassBuilder,
                                                  ClassLoader classLoader) throws PluginException;
      
    /**
     * Enhance a class to intercept constructors and class instance methods.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected abstract DynamicType.Builder<?> enhanceInstance(TypeDescription typeDescription,
                                                     DynamicType.Builder<?> newClassBuilder, ClassLoader classLoader,
                                                     EnhanceContext context) throws PluginException;  
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
enhance()方法中先调用enhanceClass()方法做静态方法插桩，再调用enhanceInstance()方法做构造器和实例方法插桩，本节先来看下静态方法插桩

ClassEnhancePluginDefine中实现了AbstractClassEnhancePluginDefine的抽象方法enhanceClass()：

public abstract class ClassEnhancePluginDefine extends AbstractClassEnhancePluginDefine {

    /**
     * Enhance a class to intercept class static methods.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected DynamicType.Builder<?> enhanceClass(TypeDescription typeDescription, DynamicType.Builder<?> newClassBuilder,
        ClassLoader classLoader) throws PluginException {
        // 获取静态方法拦截点
        StaticMethodsInterceptPoint[] staticMethodsInterceptPoints = getStaticMethodsInterceptPoints();
        String enhanceOriginClassName = typeDescription.getTypeName();
        if (staticMethodsInterceptPoints == null || staticMethodsInterceptPoints.length == 0) {
            return newClassBuilder;
        }
    
        for (StaticMethodsInterceptPoint staticMethodsInterceptPoint : staticMethodsInterceptPoints) {
            String interceptor = staticMethodsInterceptPoint.getMethodsInterceptor();
            if (StringUtil.isEmpty(interceptor)) {
                throw new EnhanceException("no StaticMethodsAroundInterceptor define to enhance class " + enhanceOriginClassName);
            }
    
            // 是否要修改原方法入参
            if (staticMethodsInterceptPoint.isOverrideArgs()) {
                // 是否为JDK类库的类 被Bootstrap ClassLoader加载
                if (isBootstrapInstrumentation()) {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                } else {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                .to(new StaticMethodsInterWithOverrideArgs(interceptor)));
                }
            } else {
                if (isBootstrapInstrumentation()) {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                } else {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .to(new StaticMethodsInter(interceptor)));
                }
            }
    
        }
    
        return newClassBuilder;
    }  
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
enhanceClass()方法处理逻辑如下：

获取静态方法拦截点
根据是否要修改原方法入参和是否为JDK类库的类走不通的分支处理逻辑
获取静态方法拦截点调用的是getStaticMethodsInterceptPoints()方法

public abstract class AbstractClassEnhancePluginDefine {

    /**
     * Static methods intercept point. See {@link StaticMethodsInterceptPoint}
     *
     * @return collections of {@link StaticMethodsInterceptPoint}
     */
    public abstract StaticMethodsInterceptPoint[] getStaticMethodsInterceptPoints();  
1
2
3
4
5
6
7
8
1）、不修改原方法入参
以mysql-8.x-plugin为例：

public class ConnectionImplCreateInstrumentation extends AbstractMysqlInstrumentation {

    private static final String JDBC_ENHANCE_CLASS = "com.mysql.cj.jdbc.ConnectionImpl";
    
    private static final String CONNECT_METHOD = "getInstance";
    
    @Override
    public StaticMethodsInterceptPoint[] getStaticMethodsInterceptPoints() {
        return new StaticMethodsInterceptPoint[] {
            new StaticMethodsInterceptPoint() {
                @Override
                public ElementMatcher<MethodDescription> getMethodsMatcher() {
                    return named(CONNECT_METHOD);
                }
    
                @Override
                public String getMethodsInterceptor() {
                    return "org.apache.skywalking.apm.plugin.jdbc.mysql.v8.ConnectionCreateInterceptor";
                }
    
                @Override
                public boolean isOverrideArgs() {
                    return false;
                }
            }
        };
    }
    
    @Override
    protected ClassMatch enhanceClass() {
        return byName(JDBC_ENHANCE_CLASS);
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
该插件拦截的是ConnectionImpl类中的静态方法getInstance()，不需要修改原方法入参，交给拦截器ConnectionCreateInterceptor来处理

public abstract class ClassEnhancePluginDefine extends AbstractClassEnhancePluginDefine {

    /**
     * Enhance a class to intercept class static methods.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected DynamicType.Builder<?> enhanceClass(TypeDescription typeDescription, DynamicType.Builder<?> newClassBuilder,
        ClassLoader classLoader) throws PluginException {
        // 获取静态方法拦截点
        StaticMethodsInterceptPoint[] staticMethodsInterceptPoints = getStaticMethodsInterceptPoints();
        String enhanceOriginClassName = typeDescription.getTypeName();
        if (staticMethodsInterceptPoints == null || staticMethodsInterceptPoints.length == 0) {
            return newClassBuilder;
        }
    
        for (StaticMethodsInterceptPoint staticMethodsInterceptPoint : staticMethodsInterceptPoints) {
            String interceptor = staticMethodsInterceptPoint.getMethodsInterceptor();
            if (StringUtil.isEmpty(interceptor)) {
                throw new EnhanceException("no StaticMethodsAroundInterceptor define to enhance class " + enhanceOriginClassName);
            }
    
            // 是否要修改原方法入参
            if (staticMethodsInterceptPoint.isOverrideArgs()) {
                // 是否为JDK类库的类 被Bootstrap ClassLoader加载
                if (isBootstrapInstrumentation()) {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                } else {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                .to(new StaticMethodsInterWithOverrideArgs(interceptor)));
                }
            } else {
                if (isBootstrapInstrumentation()) {
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                } else {
                  	// 1)
                    newClassBuilder = newClassBuilder.method(isStatic().and(staticMethodsInterceptPoint.getMethodsMatcher()))
                                                     .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                .to(new StaticMethodsInter(interceptor)));
                }
            }
    
        }
    
        return newClassBuilder;
    }  
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
mysql-8.x-plugin不需要修改原方法入参，并且拦截的类不是JDK类库的类，所以走的是代码1)处的分支处理逻辑

调用bytebuddy API，指定该方法为静态方法（isStatic()），指定方法名（staticMethodsInterceptPoint.getMethodsMatcher()），传入interceptor实例交给StaticMethodsInter去处理，StaticMethodsInter去做真正的字节码增强

StaticMethodsInter源码如下：

public class StaticMethodsInter {
    private static final ILog LOGGER = LogManager.getLogger(StaticMethodsInter.class);

    /**
     * A class full name, and instanceof {@link StaticMethodsAroundInterceptor} This name should only stay in {@link
     * String}, the real {@link Class} type will trigger classloader failure. If you want to know more, please check on
     * books about Classloader or Classloader appointment mechanism.
     */
    private String staticMethodsAroundInterceptorClassName;
    
    /**
     * Set the name of {@link StaticMethodsInter#staticMethodsAroundInterceptorClassName}
     *
     * @param staticMethodsAroundInterceptorClassName class full name.
     */
    public StaticMethodsInter(String staticMethodsAroundInterceptorClassName) {
        this.staticMethodsAroundInterceptorClassName = staticMethodsAroundInterceptorClassName;
    }
    
    /**
     * Intercept the target static method.
     *
     * @param clazz        target class 要修改字节码的目标类
     * @param allArguments all method arguments 原方法所有的入参
     * @param method       method description. 原方法
     * @param zuper        the origin call ref. 原方法的调用 zuper.call()代表调用原方法
     * @return the return value of target static method.
     * @throws Exception only throw exception because of zuper.call() or unexpected exception in sky-walking ( This is a
     *                   bug, if anything triggers this condition ).
     */
    @RuntimeType
    public Object intercept(@Origin Class<?> clazz, @AllArguments Object[] allArguments, @Origin Method method,
        @SuperCall Callable<?> zuper) throws Throwable {
        // 实例化自定义的拦截器
        StaticMethodsAroundInterceptor interceptor = InterceptorInstanceLoader.load(staticMethodsAroundInterceptorClassName, clazz
            .getClassLoader());
    
        MethodInterceptResult result = new MethodInterceptResult();
        try {
            interceptor.beforeMethod(clazz, method, allArguments, method.getParameterTypes(), result);
        } catch (Throwable t) {
            LOGGER.error(t, "class[{}] before static method[{}] intercept failure", clazz, method.getName());
        }
    
        Object ret = null;
        try {
            // 是否执行原方法
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                // 原方法的调用
                ret = zuper.call();
            }
        } catch (Throwable t) {
            try {
                interceptor.handleMethodException(clazz, method, allArguments, method.getParameterTypes(), t);
            } catch (Throwable t2) {
                LOGGER.error(t2, "class[{}] handle static method[{}] exception failure", clazz, method.getName(), t2.getMessage());
            }
            throw t;
        } finally {
            try {
                ret = interceptor.afterMethod(clazz, method, allArguments, method.getParameterTypes(), ret);
            } catch (Throwable t) {
                LOGGER.error(t, "class[{}] after static method[{}] intercept failure:{}", clazz, method.getName(), t.getMessage());
            }
        }
        return ret;
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
intercept()方法处理逻辑如下：

实例化自定义的拦截器
执行beforeMethod()方法
如果需要执行原方法，执行原方法调用，否则调用_ret()方法
如果方法执行抛出异常，调用handleMethodException()方法
最终调用finally中afterMethod()方法
public class MethodInterceptResult {
    private boolean isContinue = true;

    private Object ret = null;
    
    /**
     * define the new return value.
     *
     * @param ret new return value.
     */
    public void defineReturnValue(Object ret) {
        this.isContinue = false;
        this.ret = ret;
    }
    
    /**
     * @return true, will trigger method interceptor({@link InstMethodsInter} and {@link StaticMethodsInter}) to invoke
     * the origin method. Otherwise, not.
     */
    public boolean isContinue() {
        return isContinue;
    }
    
    /**
     * @return the new return value.
     */
    public Object _ret() {
        return ret;
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
这里是否执行原方法默认为true，如果插件的beforeMethod()方法实现中调用了defineReturnValue()传入了返回值，则不会再调用原方法，直接返回传入的返回值

2）、修改原方法入参
允许修改原方法入参会交给StaticMethodsInterWithOverrideArgs去处理

public class StaticMethodsInterWithOverrideArgs {
    private static final ILog LOGGER = LogManager.getLogger(StaticMethodsInterWithOverrideArgs.class);

    /**
     * A class full name, and instanceof {@link StaticMethodsAroundInterceptor} This name should only stay in {@link
     * String}, the real {@link Class} type will trigger classloader failure. If you want to know more, please check on
     * books about Classloader or Classloader appointment mechanism.
     */
    private String staticMethodsAroundInterceptorClassName;
    
    /**
     * Set the name of {@link StaticMethodsInterWithOverrideArgs#staticMethodsAroundInterceptorClassName}
     *
     * @param staticMethodsAroundInterceptorClassName class full name.
     */
    public StaticMethodsInterWithOverrideArgs(String staticMethodsAroundInterceptorClassName) {
        this.staticMethodsAroundInterceptorClassName = staticMethodsAroundInterceptorClassName;
    }
    
    /**
     * Intercept the target static method.
     *
     * @param clazz        target class
     * @param allArguments all method arguments
     * @param method       method description.
     * @param zuper        the origin call ref.
     * @return the return value of target static method.
     * @throws Exception only throw exception because of zuper.call() or unexpected exception in sky-walking ( This is a
     *                   bug, if anything triggers this condition ).
     */
    @RuntimeType
    public Object intercept(@Origin Class<?> clazz, @AllArguments Object[] allArguments, @Origin Method method,
        @Morph OverrideCallable zuper) throws Throwable {
        StaticMethodsAroundInterceptor interceptor = InterceptorInstanceLoader.load(staticMethodsAroundInterceptorClassName, clazz
            .getClassLoader());
    
        MethodInterceptResult result = new MethodInterceptResult();
        try {
            // beforeMethod可以修改原方法入参
            interceptor.beforeMethod(clazz, method, allArguments, method.getParameterTypes(), result);
        } catch (Throwable t) {
            LOGGER.error(t, "class[{}] before static method[{}] intercept failure", clazz, method.getName());
        }
    
        Object ret = null;
        try {
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                // 原方法的调用时传入修改后的原方法入参
                ret = zuper.call(allArguments);
            }
        } catch (Throwable t) {
            try {
                interceptor.handleMethodException(clazz, method, allArguments, method.getParameterTypes(), t);
            } catch (Throwable t2) {
                LOGGER.error(t2, "class[{}] handle static method[{}] exception failure", clazz, method.getName(), t2.getMessage());
            }
            throw t;
        } finally {
            try {
                ret = interceptor.afterMethod(clazz, method, allArguments, method.getParameterTypes(), ret);
            } catch (Throwable t) {
                LOGGER.error(t, "class[{}] after static method[{}] intercept failure:{}", clazz, method.getName(), t.getMessage());
            }
        }
        return ret;
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
StaticMethodsInterWithOverrideArgs和StaticMethodsInter的区别在于最后一个入参类型为OverrideCallable

public interface OverrideCallable {
    Object call(Object[] args);
}
1
2
3
插件的beforeMethod()方法实现中会修改原方法入参，然后在原方法的调用时传入修改后的原方法入参

小结：



11、构造器和实例方法插桩
ClassEnhancePluginDefine中实现了AbstractClassEnhancePluginDefine的抽象方法enhanceInstance()：

public abstract class ClassEnhancePluginDefine extends AbstractClassEnhancePluginDefine {

    /**
     * Enhance a class to intercept constructors and class instance methods.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected DynamicType.Builder<?> enhanceInstance(TypeDescription typeDescription,
        DynamicType.Builder<?> newClassBuilder, ClassLoader classLoader,
        EnhanceContext context) throws PluginException {
        // 构造器拦截点
        ConstructorInterceptPoint[] constructorInterceptPoints = getConstructorsInterceptPoints();
        // 实例方法拦截点
        InstanceMethodsInterceptPoint[] instanceMethodsInterceptPoints = getInstanceMethodsInterceptPoints();
        String enhanceOriginClassName = typeDescription.getTypeName();
        boolean existedConstructorInterceptPoint = false;
        if (constructorInterceptPoints != null && constructorInterceptPoints.length > 0) {
            existedConstructorInterceptPoint = true;
        }
        boolean existedMethodsInterceptPoints = false;
        if (instanceMethodsInterceptPoints != null && instanceMethodsInterceptPoints.length > 0) {
            existedMethodsInterceptPoints = true;
        }
    
        /**
         * nothing need to be enhanced in class instance, maybe need enhance static methods.
         */
        if (!existedConstructorInterceptPoint && !existedMethodsInterceptPoints) {
            return newClassBuilder;
        }
    
        /**
         * Manipulate class source code.<br/>
         *
         * new class need:<br/>
         * 1.Add field, name {@link #CONTEXT_ATTR_NAME}.
         * 2.Add a field accessor for this field.
         *
         * And make sure the source codes manipulation only occurs once.
         * 这个操作只会发生一次
         */
        // 如果当前拦截的类没有实现EnhancedInstance接口
        if (!typeDescription.isAssignableTo(EnhancedInstance.class)) {
            // 没有新增新的字段或者实现新的接口
            if (!context.isObjectExtended()) {
                // 新增一个private volatile的Object类型字段 _$EnhancedClassField_ws
                // 实现EnhancedInstance接口的get/set作为新增字段的get/set方法
                newClassBuilder = newClassBuilder.defineField(
                    CONTEXT_ATTR_NAME, Object.class, ACC_PRIVATE | ACC_VOLATILE)
                                                 .implement(EnhancedInstance.class)
                                                 .intercept(FieldAccessor.ofField(CONTEXT_ATTR_NAME));
                // 将记录状态的上下文EnhanceContext设置为已新增新的字段或者实现新的接口
                context.extendObjectCompleted();
            }
        }
    
        /**
         * 2. enhance constructors
         * 增强构造器
         */
        if (existedConstructorInterceptPoint) {
            for (ConstructorInterceptPoint constructorInterceptPoint : constructorInterceptPoints) {
                if (isBootstrapInstrumentation()) {
                    newClassBuilder = newClassBuilder.constructor(constructorInterceptPoint.getConstructorMatcher())
                                                     .intercept(SuperMethodCall.INSTANCE.andThen(MethodDelegation.withDefaultConfiguration()
                                                                                                                 .to(BootstrapInstrumentBoost
                                                                                                                     .forInternalDelegateClass(constructorInterceptPoint
                                                                                                                         .getConstructorInterceptor()))));
                } else {
                    newClassBuilder = newClassBuilder.constructor(constructorInterceptPoint.getConstructorMatcher())
                                                     .intercept(SuperMethodCall.INSTANCE.andThen(MethodDelegation.withDefaultConfiguration()
                                                                                                                 .to(new ConstructorInter(constructorInterceptPoint
                                                                                                                     .getConstructorInterceptor(), classLoader))));
                }
            }
        }
    
        /**
         * 3. enhance instance methods
         * 增强实例方法
         */
        if (existedMethodsInterceptPoints) {
            for (InstanceMethodsInterceptPoint instanceMethodsInterceptPoint : instanceMethodsInterceptPoints) {
                String interceptor = instanceMethodsInterceptPoint.getMethodsInterceptor();
                if (StringUtil.isEmpty(interceptor)) {
                    throw new EnhanceException("no InstanceMethodsAroundInterceptor define to enhance class " + enhanceOriginClassName);
                }
                ElementMatcher.Junction<MethodDescription> junction = not(isStatic()).and(instanceMethodsInterceptPoint.getMethodsMatcher());
              	// 如果拦截点为DeclaredInstanceMethodsInterceptPoint
                if (instanceMethodsInterceptPoint instanceof DeclaredInstanceMethodsInterceptPoint) {
                    // 拿到的方法必须是当前类上的 通过注解匹配可能匹配到很多方法不是当前类上的
                    junction = junction.and(ElementMatchers.<MethodDescription>isDeclaredBy(typeDescription));
                }
                if (instanceMethodsInterceptPoint.isOverrideArgs()) {
                    if (isBootstrapInstrumentation()) {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                    .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                    } else {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                    .to(new InstMethodsInterWithOverrideArgs(interceptor, classLoader)));
                    }
                } else {
                    if (isBootstrapInstrumentation()) {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                    } else {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .to(new InstMethodsInter(interceptor, classLoader)));
                    }
                }
            }
        }
    
        return newClassBuilder;
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
enhanceInstance()方法处理逻辑如下：

如果当前拦截的类没有实现EnhancedInstance接口且没有新增新的字段或者实现新的接口，则会新增一个private volatile的Object类型字段_$EnhancedClassField_ws，并实现EnhancedInstance接口的get/set作为新增字段的get/set方法，最后设置标记位，保证该操作只会发生一次
增强构造器
增强实例方法
1）、构造器插桩
构造器插桩会交给ConstructorInter去处理

public class ConstructorInter {
    private static final ILog LOGGER = LogManager.getLogger(ConstructorInter.class);

    /**
     * An {@link InstanceConstructorInterceptor} This name should only stay in {@link String}, the real {@link Class}
     * type will trigger classloader failure. If you want to know more, please check on books about Classloader or
     * Classloader appointment mechanism.
     */
    private InstanceConstructorInterceptor interceptor;
    
    /**
     * @param constructorInterceptorClassName class full name.
     */
    public ConstructorInter(String constructorInterceptorClassName, ClassLoader classLoader) throws PluginException {
        try {
            // 实例化自定义的拦截器
            interceptor = InterceptorInstanceLoader.load(constructorInterceptorClassName, classLoader);
        } catch (Throwable t) {
            throw new PluginException("Can't create InstanceConstructorInterceptorV2.", t);
        }
    }
    
    /**
     * Intercept the target constructor.
     *
     * @param obj          target class instance. 目标类的实例
     * @param allArguments all constructor arguments
     */
    @RuntimeType
    public void intercept(@This Object obj, @AllArguments Object[] allArguments) {
        try {
            EnhancedInstance targetObject = (EnhancedInstance) obj;
            // 在原生构造器执行之后再执行后调用onConstruct()方法
            // 只能访问到EnhancedInstance类型的字段 _$EnhancedClassField_ws
            // 拦截器的onConstruct把某些数据存储到_$EnhancedClassField_ws字段中
            interceptor.onConstruct(targetObject, allArguments);
        } catch (Throwable t) {
            LOGGER.error("ConstructorInter failure.", t);
        }
    
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
ConstructorInter处理逻辑如下：

构造函数中实例化自定义的拦截器
intercept()方法中调用拦截器的onConstruct()方法（在原生构造器执行之后再执行后）
public interface InstanceConstructorInterceptor {
    /**
     * Called after the origin constructor invocation.
     * 在原生构造器执行之后再执行
     */
    void onConstruct(EnhancedInstance objInst, Object[] allArguments) throws Throwable;
}
1
2
3
4
5
6
7
案例：

以activemq-5.x-plugin为例：

public class ActiveMQProducerInstrumentation extends ClassInstanceMethodsEnhancePluginDefine {
    public static final String INTERCEPTOR_CLASS = "org.apache.skywalking.apm.plugin.activemq.ActiveMQProducerInterceptor";
    public static final String ENHANCE_CLASS_PRODUCER = "org.apache.activemq.ActiveMQMessageProducer";
    public static final String CONSTRUCTOR_INTERCEPTOR_CLASS = "org.apache.skywalking.apm.plugin.activemq.ActiveMQProducerConstructorInterceptor";
    public static final String ENHANCE_METHOD = "send";
    public static final String CONSTRUCTOR_INTERCEPT_TYPE = "org.apache.activemq.ActiveMQSession";

    @Override
    public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        return new ConstructorInterceptPoint[] {
            new ConstructorInterceptPoint() {
                @Override
                public ElementMatcher<MethodDescription> getConstructorMatcher() {
                    return takesArgumentWithType(0, CONSTRUCTOR_INTERCEPT_TYPE);
                }
    
                @Override
                public String getConstructorInterceptor() {
                    return CONSTRUCTOR_INTERCEPTOR_CLASS;
                }
            }
        };
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
该插件拦截的是ActiveMQMessageProducer类中的第一个参数类型为ActiveMQSession的构造器，交给拦截器ActiveMQProducerInterceptor来处理

public class ActiveMQProducerConstructorInterceptor implements InstanceConstructorInterceptor {
    @Override
    public void onConstruct(EnhancedInstance objInst, Object[] allArguments) {
        ActiveMQSession session = (ActiveMQSession) allArguments[0];
        // 将broker地址保存到_$EnhancedClassField_ws字段中
        objInst.setSkyWalkingDynamicField(session.getConnection().getTransport().getRemoteAddress().split("//")[1]);
    }
}
1
2
3
4
5
6
7
8
所以插件中的构造器插桩是为了在_$EnhancedClassField_ws字段中保存一些数据方便后续使用，这也是ClassEnhancePluginDefine的enhanceInstance()方法中为什么让当前拦截的类新增_$EnhancedClassField_ws字段并实现EnhancedInstance接口的get/set作为新增字段的get/set方法

2）、实例方法插桩
public abstract class ClassEnhancePluginDefine extends AbstractClassEnhancePluginDefine {

    /**
     * Enhance a class to intercept constructors and class instance methods.
     *
     * @param typeDescription target class description
     * @param newClassBuilder byte-buddy's builder to manipulate class bytecode.
     * @return new byte-buddy's builder for further manipulation.
     */
    protected DynamicType.Builder<?> enhanceInstance(TypeDescription typeDescription,
        DynamicType.Builder<?> newClassBuilder, ClassLoader classLoader,
        EnhanceContext context) throws PluginException {
        // 构造器拦截点
        ConstructorInterceptPoint[] constructorInterceptPoints = getConstructorsInterceptPoints();
        // 实例方法拦截点
        InstanceMethodsInterceptPoint[] instanceMethodsInterceptPoints = getInstanceMethodsInterceptPoints();
        String enhanceOriginClassName = typeDescription.getTypeName();
        boolean existedConstructorInterceptPoint = false;
        if (constructorInterceptPoints != null && constructorInterceptPoints.length > 0) {
            existedConstructorInterceptPoint = true;
        }
        boolean existedMethodsInterceptPoints = false;
        if (instanceMethodsInterceptPoints != null && instanceMethodsInterceptPoints.length > 0) {
            existedMethodsInterceptPoints = true;
        }
    
        /**
         * nothing need to be enhanced in class instance, maybe need enhance static methods.
         */
        if (!existedConstructorInterceptPoint && !existedMethodsInterceptPoints) {
            return newClassBuilder;
        }
    
        /**
         * Manipulate class source code.<br/>
         *
         * new class need:<br/>
         * 1.Add field, name {@link #CONTEXT_ATTR_NAME}.
         * 2.Add a field accessor for this field.
         *
         * And make sure the source codes manipulation only occurs once.
         * 这个操作只会发生一次
         */
        // 如果当前拦截的类没有实现EnhancedInstance接口
        if (!typeDescription.isAssignableTo(EnhancedInstance.class)) {
            // 没有新增新的字段或者实现新的接口
            if (!context.isObjectExtended()) {
                // 新增一个private volatile的Object类型字段 _$EnhancedClassField_ws
                // 实现EnhancedInstance接口的get/set作为新增字段的get/set方法
                newClassBuilder = newClassBuilder.defineField(
                    CONTEXT_ATTR_NAME, Object.class, ACC_PRIVATE | ACC_VOLATILE)
                                                 .implement(EnhancedInstance.class)
                                                 .intercept(FieldAccessor.ofField(CONTEXT_ATTR_NAME));
                // 将记录状态的上下文EnhanceContext设置为已新增新的字段或者实现新的接口
                context.extendObjectCompleted();
            }
        }
    
        /**
         * 2. enhance constructors
         * 增强构造器
         */
        if (existedConstructorInterceptPoint) {
            for (ConstructorInterceptPoint constructorInterceptPoint : constructorInterceptPoints) {
                if (isBootstrapInstrumentation()) {
                    newClassBuilder = newClassBuilder.constructor(constructorInterceptPoint.getConstructorMatcher())
                                                     .intercept(SuperMethodCall.INSTANCE.andThen(MethodDelegation.withDefaultConfiguration()
                                                                                                                 .to(BootstrapInstrumentBoost
                                                                                                                     .forInternalDelegateClass(constructorInterceptPoint
                                                                                                                         .getConstructorInterceptor()))));
                } else {
                    newClassBuilder = newClassBuilder.constructor(constructorInterceptPoint.getConstructorMatcher())
                                                     .intercept(SuperMethodCall.INSTANCE.andThen(MethodDelegation.withDefaultConfiguration()
                                                                                                                 .to(new ConstructorInter(constructorInterceptPoint
                                                                                                                     .getConstructorInterceptor(), classLoader))));
                }
            }
        }
    
        /**
         * 3. enhance instance methods
         * 增强实例方法
         */
        if (existedMethodsInterceptPoints) {
            for (InstanceMethodsInterceptPoint instanceMethodsInterceptPoint : instanceMethodsInterceptPoints) {
                String interceptor = instanceMethodsInterceptPoint.getMethodsInterceptor();
                if (StringUtil.isEmpty(interceptor)) {
                    throw new EnhanceException("no InstanceMethodsAroundInterceptor define to enhance class " + enhanceOriginClassName);
                }
                ElementMatcher.Junction<MethodDescription> junction = not(isStatic()).and(instanceMethodsInterceptPoint.getMethodsMatcher());
              	// 1)如果拦截点为DeclaredInstanceMethodsInterceptPoint
                if (instanceMethodsInterceptPoint instanceof DeclaredInstanceMethodsInterceptPoint) {
                    // 拿到的方法必须是当前类上的 通过注解匹配可能匹配到很多方法不是当前类上的
                    junction = junction.and(ElementMatchers.<MethodDescription>isDeclaredBy(typeDescription));
                }
                if (instanceMethodsInterceptPoint.isOverrideArgs()) {
                    if (isBootstrapInstrumentation()) {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                    .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                    } else {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                    .to(new InstMethodsInterWithOverrideArgs(interceptor, classLoader)));
                    }
                } else {
                    if (isBootstrapInstrumentation()) {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                    } else {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .to(new InstMethodsInter(interceptor, classLoader)));
                    }
                }
            }
        }
    
        return newClassBuilder;
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
代码1)处判断如果拦截点为DeclaredInstanceMethodsInterceptPoint，会新增匹配条件：拿到的方法必须是当前类上的

/**
 * this interface for those who only want to enhance declared method in case of some unexpected issue, such as spring
 * controller
 *
 * Person
 *  sayHello();
 *
 * UserController
 *  addUser/saveUser
 *  removeUser
 *
 * //@GetMapping @PostMapping
 */
    public interface DeclaredInstanceMethodsInterceptPoint extends InstanceMethodsInterceptPoint {
    }
    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    如果要增强Person的sayHello()方法，那么可以直接通过类名方法名指定，但是如果需要增强所有Controller的方法，需要通过注解指定

通过注解匹配可能匹配到很多方法不是当前类上的，所以判断如果拦截点为DeclaredInstanceMethodsInterceptPoint会新增匹配条件：拿到的方法必须是当前类上的

1）实例方法插桩不修改原方法入参会交给InstMethodsInter去处理

public class InstMethodsInter {
    private static final ILog LOGGER = LogManager.getLogger(InstMethodsInter.class);

    /**
     * An {@link InstanceMethodsAroundInterceptor} This name should only stay in {@link String}, the real {@link Class}
     * type will trigger classloader failure. If you want to know more, please check on books about Classloader or
     * Classloader appointment mechanism.
     */
    private InstanceMethodsAroundInterceptor interceptor;
    
    /**
     * @param instanceMethodsAroundInterceptorClassName class full name.
     */
    public InstMethodsInter(String instanceMethodsAroundInterceptorClassName, ClassLoader classLoader) {
        try {
            // 对于同一份字节码,如果由不同的类加载器进行加载,则加载出来的两个实例不相同
            interceptor = InterceptorInstanceLoader.load(instanceMethodsAroundInterceptorClassName, classLoader);
        } catch (Throwable t) {
            throw new PluginException("Can't create InstanceMethodsAroundInterceptor.", t);
        }
    }
    
    /**
     * Intercept the target instance method.
     *
     * @param obj          target class instance.
     * @param allArguments all method arguments
     * @param method       method description.
     * @param zuper        the origin call ref.
     * @return the return value of target instance method.
     * @throws Exception only throw exception because of zuper.call() or unexpected exception in sky-walking ( This is a
     *                   bug, if anything triggers this condition ).
     */
    @RuntimeType
    public Object intercept(@This Object obj, @AllArguments Object[] allArguments, @SuperCall Callable<?> zuper,
        @Origin Method method) throws Throwable {
        EnhancedInstance targetObject = (EnhancedInstance) obj;
    
        MethodInterceptResult result = new MethodInterceptResult();
        try {
            interceptor.beforeMethod(targetObject, method, allArguments, method.getParameterTypes(), result);
        } catch (Throwable t) {
            LOGGER.error(t, "class[{}] before method[{}] intercept failure", obj.getClass(), method.getName());
        }
    
        Object ret = null;
        try {
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                ret = zuper.call();
            }
        } catch (Throwable t) {
            try {
                interceptor.handleMethodException(targetObject, method, allArguments, method.getParameterTypes(), t);
            } catch (Throwable t2) {
                LOGGER.error(t2, "class[{}] handle method[{}] exception failure", obj.getClass(), method.getName());
            }
            throw t;
        } finally {
            try {
                ret = interceptor.afterMethod(targetObject, method, allArguments, method.getParameterTypes(), ret);
            } catch (Throwable t) {
                LOGGER.error(t, "class[{}] after method[{}] intercept failure", obj.getClass(), method.getName());
            }
        }
        return ret;
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
对于同一份字节码，如果由不同的类加载器进行加载，则加载出来的两个实例不相同，所以在加载拦截器的时候传入了classLoader

2）实例方法插桩修改原方法入参会交给InstMethodsInterWithOverrideArgs去处理

public class InstMethodsInterWithOverrideArgs {
    private static final ILog LOGGER = LogManager.getLogger(InstMethodsInterWithOverrideArgs.class);

    /**
     * An {@link InstanceMethodsAroundInterceptor} This name should only stay in {@link String}, the real {@link Class}
     * type will trigger classloader failure. If you want to know more, please check on books about Classloader or
     * Classloader appointment mechanism.
     */
    private InstanceMethodsAroundInterceptor interceptor;
    
    /**
     * @param instanceMethodsAroundInterceptorClassName class full name.
     */
    public InstMethodsInterWithOverrideArgs(String instanceMethodsAroundInterceptorClassName, ClassLoader classLoader) {
        try {
            interceptor = InterceptorInstanceLoader.load(instanceMethodsAroundInterceptorClassName, classLoader);
        } catch (Throwable t) {
            throw new PluginException("Can't create InstanceMethodsAroundInterceptor.", t);
        }
    }
    
    /**
     * Intercept the target instance method.
     *
     * @param obj          target class instance.
     * @param allArguments all method arguments
     * @param method       method description.
     * @param zuper        the origin call ref.
     * @return the return value of target instance method.
     * @throws Exception only throw exception because of zuper.call() or unexpected exception in sky-walking ( This is a
     *                   bug, if anything triggers this condition ).
     */
    @RuntimeType
    public Object intercept(@This Object obj, @AllArguments Object[] allArguments, @Origin Method method,
        @Morph OverrideCallable zuper) throws Throwable {
        EnhancedInstance targetObject = (EnhancedInstance) obj;
    
        MethodInterceptResult result = new MethodInterceptResult();
        try {
            interceptor.beforeMethod(targetObject, method, allArguments, method.getParameterTypes(), result);
        } catch (Throwable t) {
            LOGGER.error(t, "class[{}] before method[{}] intercept failure", obj.getClass(), method.getName());
        }
    
        Object ret = null;
        try {
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                ret = zuper.call(allArguments);
            }
        } catch (Throwable t) {
            try {
                interceptor.handleMethodException(targetObject, method, allArguments, method.getParameterTypes(), t);
            } catch (Throwable t2) {
                LOGGER.error(t2, "class[{}] handle method[{}] exception failure", obj.getClass(), method.getName());
            }
            throw t;
        } finally {
            try {
                ret = interceptor.afterMethod(targetObject, method, allArguments, method.getParameterTypes(), ret);
            } catch (Throwable t) {
                LOGGER.error(t, "class[{}] after method[{}] intercept failure", obj.getClass(), method.getName());
            }
        }
        return ret;
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
不修改原方法入参和修改原方法入参的处理逻辑和静态方法插桩的处理逻辑相同

问题1：为什么StaticMethodsInter可以直接通过clazz.getClassLoader()获取类加载器，而InstMethodsInter需要从上层传递ClassLoader

StaticMethodsInter可以直接通过clazz.getClassLoader()获取类加载器是因为静态方法直接绑定了类

InstMethodsInter需要从上层传递ClassLoader有两个原因：第一个是一份字节码可能被多个ClassLoader加载，这样加载出来的每个实例都不相等，所以必须要绑定好ClassLoader。第二个原因是InstMethodsInter里对拦截器的加载前置到了构造方法，这是因为可能出现无法加载拦截器成功的情况，如果放到intercept()方法里去延后加载拦截器，那么拦截器加载失败产生的异常将和字节码修改导致的异常、业务异常出现混乱，这里是为了让异常边界清晰而做的处理

问题2：intercept()方法中通过zuper.call()执行原方法的调用，这里为什么不能替换成method.invoke(clazz.newInstance())

public class StaticMethodsInter {

    @RuntimeType
    public Object intercept(@Origin Class<?> clazz, @AllArguments Object[] allArguments, @Origin Method method,
        @SuperCall Callable<?> zuper) throws Throwable {
        // 实例化自定义的拦截器
        StaticMethodsAroundInterceptor interceptor = InterceptorInstanceLoader.load(staticMethodsAroundInterceptorClassName, clazz
            .getClassLoader());
    
        MethodInterceptResult result = new MethodInterceptResult();
        try {
            interceptor.beforeMethod(clazz, method, allArguments, method.getParameterTypes(), result);
        } catch (Throwable t) {
            LOGGER.error(t, "class[{}] before static method[{}] intercept failure", clazz, method.getName());
        }
    
        Object ret = null;
        try {
            // 是否执行原方法
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                // 原方法的调用
                ret = zuper.call();
//                ret = method.invoke(clazz.newInstance());
            }
        } catch (Throwable t) {
            try {
                interceptor.handleMethodException(clazz, method, allArguments, method.getParameterTypes(), t);
            } catch (Throwable t2) {
                LOGGER.error(t2, "class[{}] handle static method[{}] exception failure", clazz, method.getName(), t2.getMessage());
            }
            throw t;
        } finally {
            try {
                ret = interceptor.afterMethod(clazz, method, allArguments, method.getParameterTypes(), ret);
            } catch (Throwable t) {
                LOGGER.error(t, "class[{}] after static method[{}] intercept failure:{}", clazz, method.getName(), t.getMessage());
            }
        }
        return ret;
    }  
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
解答这个问题需要借助一个Java实时反编译工具friday，查看挂载SkyWalking Agent之后反编译的内容

@RestController
@RequestMapping("/api/hello")
public class UserController {

    @GetMapping
    public String sayHello() {
        return "hello";
    }

}
1
2
3
4
5
6
7
8
9
10
UserController反编译的内容：

@RestController
@RequestMapping(value={"/api/hello"})
public class UserController
implements EnhancedInstance {
    private volatile Object _$EnhancedClassField_ws;
    public static volatile /* synthetic */ InstMethodsInter delegate$mvblfc0;
    public static volatile /* synthetic */ InstMethodsInter delegate$hfbkh30;
    public static volatile /* synthetic */ ConstructorInter delegate$gr07501;
    private static final /* synthetic */ Method cachedValue$kkbY4FHP$ldstch2;
    public static volatile /* synthetic */ InstMethodsInter delegate$lvp69q1;
    public static volatile /* synthetic */ InstMethodsInter delegate$mpv7fs0;
    public static volatile /* synthetic */ ConstructorInter delegate$v0q1e31;
    private static final /* synthetic */ Method cachedValue$Hx3zGNqH$ldstch2;

    public UserController() {
        this(null);
        delegate$v0q1e31.intercept((Object)this, new Object[0]);
    }
    
    private /* synthetic */ UserController(auxiliary.YsFzTfDy ysFzTfDy) {
    }
    
    @GetMapping
    public String sayHello() {
        return (String)delegate$lvp69q1.intercept((Object)this, new Object[0], (Callable)new auxiliary.pEJy33Ip(this), cachedValue$Hx3zGNqH$ldstch2);
    }
    
    private /* synthetic */ String sayHello$original$70VVkKcL() {
        return "hello";
    }
    
    static {
        ClassLoader.getSystemClassLoader().loadClass("org.apache.skywalking.apm.dependencies.net.bytebuddy.dynamic.Nexus").getMethod("initialize", Class.class, Integer.TYPE).invoke(null, UserController.class, 544534948);
        cachedValue$Hx3zGNqH$ldstch2 = UserController.class.getMethod("sayHello", new Class[0]);
    }
    
    final /* synthetic */ String sayHello$original$70VVkKcL$accessor$Hx3zGNqH() {
        return this.sayHello$original$70VVkKcL();
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
SkyWalkingAgent的premain()方法中在构建agentBuilder时使用的策略是RETRANSFORMATION

public class SkyWalkingAgent {

    public static void premain(String agentArgs, Instrumentation instrumentation) throws PluginException {
      	// ...
        agentBuilder.type(pluginFinder.buildMatch()) // 指定ByteBuddy要拦截的类
                    .transform(new Transformer(pluginFinder))
                    .with(AgentBuilder.RedefinitionStrategy.RETRANSFORMATION) // redefine和retransform的区别在于是否保留修改前的内容
                    .with(new RedefinitionListener())
                    .with(new Listener())
                    .installOn(instrumentation);
1
2
3
4
5
6
7
8
9
10
redefine和retransform的区别在于是否保留修改前的内容，从反编译的内容可以看到sayHello()方法的内容已经被修改掉了，原方法被重命名为sayHello$original$+随机字符串

新的sayHello()方法中调用了InstMethodsInter的intercept()方法，最后传入的intercept()的method参数是被修改后的sayHello()方法，不再指向原生的sayHello()方法，如果在InstMethodsInter使用method.invoke(clazz.newInstance())就相当于自己调自己，就会死递归下去

小结：



12、插件拦截器加载流程
无论是静态方法插桩还是构造器和实例方法插桩都会调用InterceptorInstanceLoader的load()方法来实例化插件拦截器

public class InterceptorInstanceLoader {

    private static ConcurrentHashMap<String, Object> INSTANCE_CACHE = new ConcurrentHashMap<String, Object>();
    private static ReentrantLock INSTANCE_LOAD_LOCK = new ReentrantLock();
    /**
     * key:加载当前插件要拦截的那个类的类加载器
     * value:既能加载拦截器,又能加载要拦截的那个类的类加载器
     */
    private static Map<ClassLoader, ClassLoader> EXTEND_PLUGIN_CLASSLOADERS = new HashMap<ClassLoader, ClassLoader>();
    
    /**
     * Load an instance of interceptor, and keep it singleton. Create {@link AgentClassLoader} for each
     * targetClassLoader, as an extend classloader. It can load interceptor classes from plugins, activations folders.
     *
     * @param className         the interceptor class, which is expected to be found 插件拦截器全类名
     * @param targetClassLoader the class loader for current application context 当前应用上下文的类加载器
     * @param <T>               expected type
     * @return the type reference.
     */
    public static <T> T load(String className,
        ClassLoader targetClassLoader) throws IllegalAccessException, InstantiationException, ClassNotFoundException, AgentPackageNotFoundException {
        if (targetClassLoader == null) {
            targetClassLoader = InterceptorInstanceLoader.class.getClassLoader();
        }
        // org.example.Hello_OF_org.example.classloader.MyClassLoader@xxxxx
        String instanceKey = className + "_OF_" + targetClassLoader.getClass()
                                                                   .getName() + "@" + Integer.toHexString(targetClassLoader
            .hashCode());
        // className所代表的拦截器的实例 对于同一个classloader而言相同的类只加载一次
        Object inst = INSTANCE_CACHE.get(instanceKey);
        if (inst == null) {
            INSTANCE_LOAD_LOCK.lock();
            ClassLoader pluginLoader;
            try {
                pluginLoader = EXTEND_PLUGIN_CLASSLOADERS.get(targetClassLoader);
                if (pluginLoader == null) {
                    // targetClassLoader作为AgentClassLoader的父类加载器
                    pluginLoader = new AgentClassLoader(targetClassLoader);
                    EXTEND_PLUGIN_CLASSLOADERS.put(targetClassLoader, pluginLoader);
                }
            } finally {
                INSTANCE_LOAD_LOCK.unlock();
            }
            // 通过pluginLoader来实例化拦截器对象
            inst = Class.forName(className, true, pluginLoader).newInstance();
            if (inst != null) {
                INSTANCE_CACHE.put(instanceKey, inst);
            }
        }
    
        return (T) inst;
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
为什么针对不同的targetClassLoader，要初始化多个AgentClassLoader实例，并用targetClassLoader作为AgentClassLoader的parent

如果只实例化一个AgentClassLoader实例，由于应用系统中的类不存在于AgentClassLoader的classpath下，那此时AgentClassLoader加载不到应用系统中的类

针对每个targetClassLoader都初始化一个AgentClassLoader实例，并用targetClassLoader作为AgentClassLoader的父类加载器，通过双亲委派模型模型，targetClassLoader可以加载应用系统中的类



以dubbo插件为例，假设应用系统中Dubbo的类是由AppClassLoader加载的

DubboInterceptor要修改MonitorFilter的字节码，两个类需要能交互，前提就是DubboInterceptor能通过某种方式访问到MonitorFilter



让AgentClassLoader的父类加载器指向加载Dubbo的AppClassLoader，当DubboInterceptor去操作MonitorFilter的时候，通过双亲委派模型模型，AgentClassLoader的父类加载器AppClassLoader能加载到MonitorFilter

13、JDK类库插件工作原理
1）、Agent启动流程：将必要的类注入到Bootstrap ClassLoader
public class SkyWalkingAgent {

    public static void premain(String agentArgs, Instrumentation instrumentation) throws PluginException {
        final PluginFinder pluginFinder;
        try {
            // 初始化配置
            SnifferConfigInitializer.initializeCoreConfig(agentArgs);
        } catch (Exception e) {
            // try to resolve a new logger, and use the new logger to write the error log here
            LogManager.getLogger(SkyWalkingAgent.class)
                    .error(e, "SkyWalking agent initialized failure. Shutting down.");
            return;
        } finally {
            // refresh logger again after initialization finishes
            LOGGER = LogManager.getLogger(SkyWalkingAgent.class);
        }
    
        try {
            // 加载插件
            pluginFinder = new PluginFinder(new PluginBootstrap().loadPlugins());
        } catch (AgentPackageNotFoundException ape) {
            LOGGER.error(ape, "Locate agent.jar failure. Shutting down.");
            return;
        } catch (Exception e) {
            LOGGER.error(e, "SkyWalking agent initialized failure. Shutting down.");
            return;
        }
    
        // 定制化Agent行为
        // 创建ByteBuddy实例
        final ByteBuddy byteBuddy = new ByteBuddy().with(TypeValidation.of(Config.Agent.IS_OPEN_DEBUGGING_CLASS));
    
        // 指定ByteBuddy要忽略的类
        AgentBuilder agentBuilder = new AgentBuilder.Default(byteBuddy).ignore(
                nameStartsWith("net.bytebuddy.")
                        .or(nameStartsWith("org.slf4j."))
                        .or(nameStartsWith("org.groovy."))
                        .or(nameContains("javassist"))
                        .or(nameContains(".asm."))
                        .or(nameContains(".reflectasm."))
                        .or(nameStartsWith("sun.reflect"))
                        .or(allSkyWalkingAgentExcludeToolkit())
                        .or(ElementMatchers.isSynthetic()));
    
        // 将必要的类注入到Bootstrap ClassLoader
        JDK9ModuleExporter.EdgeClasses edgeClasses = new JDK9ModuleExporter.EdgeClasses();
        try {
            agentBuilder = BootstrapInstrumentBoost.inject(pluginFinder, instrumentation, agentBuilder, edgeClasses);
        } catch (Exception e) {
            LOGGER.error(e, "SkyWalking agent inject bootstrap instrumentation failure. Shutting down.");
            return;
        }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
SkyWalkingAgent的premain()方法中会调用BootstrapInstrumentBoost的inject()方法将必要的类注入到Bootstrap ClassLoader，inject()方法源码如下：

public class BootstrapInstrumentBoost {

    public static AgentBuilder inject(PluginFinder pluginFinder, Instrumentation instrumentation,
        AgentBuilder agentBuilder, JDK9ModuleExporter.EdgeClasses edgeClasses) throws PluginException {
        // 所有要注入到Bootstrap ClassLoader里的类
        Map<String, byte[]> classesTypeMap = new HashMap<>();
    
        /**
         * 针对于目标类是JDK核心类库的插件,根据插件的拦截点的不同(实例方法、静态方法、构造方法)
         * 使用不同的模板(xxxTemplate)来定义新的拦截器的核心处理逻辑,并且将插件本身定义的拦截器的全类名
         * 赋值给模板的TARGET_INTERCEPTOR字段
         * 最终,这些新的拦截器的核心处理逻辑都会被放入到BootstrapClassLoader中
         */
      	// 1)
        if (!prepareJREInstrumentation(pluginFinder, classesTypeMap)) {
            return agentBuilder;
        }
    
        if (!prepareJREInstrumentationV2(pluginFinder, classesTypeMap)) {
            return agentBuilder;
        }
    
        for (String highPriorityClass : HIGH_PRIORITY_CLASSES) {
            loadHighPriorityClass(classesTypeMap, highPriorityClass);
        }
        for (String highPriorityClass : ByteBuddyCoreClasses.CLASSES) {
            loadHighPriorityClass(classesTypeMap, highPriorityClass);
        }
    
        /**
         * Prepare to open edge of necessary classes.
         */
        for (String generatedClass : classesTypeMap.keySet()) {
            edgeClasses.add(generatedClass);
        }
    
        /**
         * 将这些类注入到Bootstrap ClassLoader
         * Inject the classes into bootstrap class loader by using Unsafe Strategy.
         * ByteBuddy adapts the sun.misc.Unsafe and jdk.internal.misc.Unsafe automatically.
         */
        ClassInjector.UsingUnsafe.Factory factory = ClassInjector.UsingUnsafe.Factory.resolve(instrumentation);
        factory.make(null, null).injectRaw(classesTypeMap);
        agentBuilder = agentBuilder.with(new AgentBuilder.InjectionStrategy.UsingUnsafe.OfFactory(factory));
    
        return agentBuilder;
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
代码1)处先调用prepareJREInstrumentation()，源码如下：

public class BootstrapInstrumentBoost {

    private static boolean prepareJREInstrumentation(PluginFinder pluginFinder,
        Map<String, byte[]> classesTypeMap) throws PluginException {
        TypePool typePool = TypePool.Default.of(BootstrapInstrumentBoost.class.getClassLoader());
        // 1)所有要对JDK核心类库生效的插件
        List<AbstractClassEnhancePluginDefine> bootstrapClassMatchDefines = pluginFinder.getBootstrapClassMatchDefine();
        for (AbstractClassEnhancePluginDefine define : bootstrapClassMatchDefines) {
            // 是否定义实例方法拦截点
            if (Objects.nonNull(define.getInstanceMethodsInterceptPoints())) {
                for (InstanceMethodsInterceptPoint point : define.getInstanceMethodsInterceptPoints()) {
                  	// 2)
                    if (point.isOverrideArgs()) {
                        generateDelegator(
                            classesTypeMap, typePool, INSTANCE_METHOD_WITH_OVERRIDE_ARGS_DELEGATE_TEMPLATE, point
                                .getMethodsInterceptor());
                    } else {
                      	// 3)
                        generateDelegator(
                            classesTypeMap, typePool, INSTANCE_METHOD_DELEGATE_TEMPLATE, point.getMethodsInterceptor());
                    }
                }
            }
    
            // 是否定义构造器拦截点
            if (Objects.nonNull(define.getConstructorsInterceptPoints())) {
                for (ConstructorInterceptPoint point : define.getConstructorsInterceptPoints()) {
                    generateDelegator(
                        classesTypeMap, typePool, CONSTRUCTOR_DELEGATE_TEMPLATE, point.getConstructorInterceptor());
                }
            }
    
            // 是否定义静态方法拦截点
            if (Objects.nonNull(define.getStaticMethodsInterceptPoints())) {
                for (StaticMethodsInterceptPoint point : define.getStaticMethodsInterceptPoints()) {
                    if (point.isOverrideArgs()) {
                        generateDelegator(
                            classesTypeMap, typePool, STATIC_METHOD_WITH_OVERRIDE_ARGS_DELEGATE_TEMPLATE, point
                                .getMethodsInterceptor());
                    } else {
                        generateDelegator(
                            classesTypeMap, typePool, STATIC_METHOD_DELEGATE_TEMPLATE, point.getMethodsInterceptor());
                    }
                }
            }
        }
        return bootstrapClassMatchDefines.size() > 0;
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
代码1)处调用pluginFinder的getBootstrapClassMatchDefine()方法，在初始化PluginFinder对象的时候，会对加载的所有插件做分类，要对JDK核心类库生效的插件都放入到一个List（bootstrapClassMatchDefine）中，这里调用getBootstrapClassMatchDefine()就是拿到所有要对JDK核心类库生效的插件

遍历所有要对JDK核心类库生效的插件，分别判断是否定义实例方法拦截点、是否定义构造器拦截点、是否定义静态方法拦截点，以实例方法拦截点为例，代码2)处再根据是否修改原方法入参走不同分支处理逻辑

以不重写原方法入参为例，代码3)处调用generateDelegator()方法生成一个代理器，这里会传入一个模板类名，对于实例方法拦截点且不修改原方法入参，模板类名为org.apache.skywalking.apm.agent.core.plugin.bootstrap.template.InstanceMethodInterTemplate

/**
 * --------CLASS TEMPLATE---------
 * <p>Author, Wu Sheng </p>
 * <p>Comment, don't change this unless you are 100% sure the agent core mechanism for bootstrap class
 * instrumentation.</p>
 * <p>Date, 24th July 2019</p>
 * -------------------------------
 * <p>
 * This class wouldn't be loaded in real env. This is a class template for dynamic class generation.
 * 在真实环境下这个类是不会被加载的,这是一个类模板用于动态类生成
 */
   public class InstanceMethodInterTemplate {
    /**
     * This field is never set in the template, but has value in the runtime.
       */
         private static String TARGET_INTERCEPTOR;

    private static InstanceMethodsAroundInterceptor INTERCEPTOR;
    private static IBootstrapLog LOGGER;

    /**
     * Intercept the target instance method.
       *
     * @param obj          target class instance.
     * @param allArguments all method arguments
     * @param method       method description.
     * @param zuper        the origin call ref.
     * @return the return value of target instance method.
     * @throws Exception only throw exception because of zuper.call() or unexpected exception in sky-walking ( This is a
     *                   bug, if anything triggers this condition ).
       */
         @RuntimeType
         public static Object intercept(@This Object obj, @AllArguments Object[] allArguments, @SuperCall Callable<?> zuper,
        @Origin Method method) throws Throwable {
        EnhancedInstance targetObject = (EnhancedInstance) obj;

        prepare();

        MethodInterceptResult result = new MethodInterceptResult();
        try {
            if (INTERCEPTOR != null) {
                INTERCEPTOR.beforeMethod(targetObject, method, allArguments, method.getParameterTypes(), result);
            }
        } catch (Throwable t) {
            if (LOGGER != null) {
                LOGGER.error(t, "class[{}] before method[{}] intercept failure", obj.getClass(), method.getName());
            }
        }

        Object ret = null;
        try {
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                ret = zuper.call();
            }
        } catch (Throwable t) {
            try {
                if (INTERCEPTOR != null) {
                    INTERCEPTOR.handleMethodException(targetObject, method, allArguments, method.getParameterTypes(), t);
                }
            } catch (Throwable t2) {
                if (LOGGER != null) {
                    LOGGER.error(t2, "class[{}] handle method[{}] exception failure", obj.getClass(), method.getName());
                }
            }
            throw t;
        } finally {
            try {
                if (INTERCEPTOR != null) {
                    ret = INTERCEPTOR.afterMethod(targetObject, method, allArguments, method.getParameterTypes(), ret);
                }
            } catch (Throwable t) {
                if (LOGGER != null) {
                    LOGGER.error(t, "class[{}] after method[{}] intercept failure", obj.getClass(), method.getName());
                }
            }
        }

        return ret;
         }

    /**
     * Prepare the context. Link to the agent core in AppClassLoader.
       *
     * 1.打通BootstrapClassLoader和AgentClassLoader
     * 拿到ILog生成日志对象
     * 拿到插件自定义的拦截器实例
     * 2.代替非JDK核心类库插件运行逻辑里的
     * InterceptorInstanceLoader.load(instanceMethodsAroundInterceptorClassName, classLoader)
       */
       private static void prepare() {
        if (INTERCEPTOR == null) {
            ClassLoader loader = BootstrapInterRuntimeAssist.getAgentClassLoader();

            if (loader != null) {
                IBootstrapLog logger = BootstrapInterRuntimeAssist.getLogger(loader, TARGET_INTERCEPTOR);
                if (logger != null) {
                    LOGGER = logger;
            
                    INTERCEPTOR = BootstrapInterRuntimeAssist.createInterceptor(loader, TARGET_INTERCEPTOR, LOGGER);
                }
            } else {
                LOGGER.error("Runtime ClassLoader not found when create {}." + TARGET_INTERCEPTOR);
            }
        }
       }
         }
         1
         2
         3
         4
         5
         6
         7
         8
         9
         10
         11
         12
         13
         14
         15
         16
         17
         18
         19
         20
         21
         22
         23
         24
         25
         26
         27
         28
         29
         30
         31
         32
         33
         34
         35
         36
         37
         38
         39
         40
         41
         42
         43
         44
         45
         46
         47
         48
         49
         50
         51
         52
         53
         54
         55
         56
         57
         58
         59
         60
         61
         62
         63
         64
         65
         66
         67
         68
         69
         70
         71
         72
         73
         74
         75
         76
         77
         78
         79
         80
         81
         82
         83
         84
         85
         86
         87
         88
         89
         90
         91
         92
         93
         94
         95
         96
         97
         98
         99
         100
         101
         102
         103
         104
         105
         106
         107
         108
         模板类的intercept()方法和实例方法插桩的InstMethodsInter的intercept()方法逻辑基本相同

generateDelegator()方法源码如下：

public class BootstrapInstrumentBoost {

    /**
     * Generate the delegator class based on given template class. This is preparation stage level code generation.
     * 根据给定的模板类生成代理器类,这是准备阶段级别的代码生成
     * <p>
     * One key step to avoid class confliction between AppClassLoader and BootstrapClassLoader
     * 避免AppClassLoader和BootstrapClassLoader之间的类冲突的一个关键步骤
     *
     * @param classesTypeMap    hosts injected binary of generated class 所有要注入到BootStrapClassLoader中的类,key:全类名 value:字节码
     * @param typePool          to generate new class 加载BootstrapInstrumentBoost的ClassLoader的类型池
     * @param templateClassName represents the class as template in this generation process. The templates are
     *                          pre-defined in SkyWalking agent core. 模板类名
     * @param methodsInterceptor 插件拦截器全类名
     */
    private static void generateDelegator(Map<String, byte[]> classesTypeMap, TypePool typePool,
        String templateClassName, String methodsInterceptor) {
        // methodsInterceptor + "_internal"
        String internalInterceptorName = internalDelegate(methodsInterceptor);
        try {
            // ClassLoaderA 已经加载了100个类,但是在这个ClassLoader的classpath下有200个类,那么这里
            // typePool.describe可以拿到当前ClassLoader的classpath下还没有加载的类的定义(描述)
            TypeDescription templateTypeDescription = typePool.describe(templateClassName).resolve();
    
            DynamicType.Unloaded interceptorType = new ByteBuddy().redefine(templateTypeDescription, ClassFileLocator.ForClassLoader
                    .of(BootstrapInstrumentBoost.class.getClassLoader()))
                                                                  // 改名为methodsInterceptor + "_internal"
                                                                  .name(internalInterceptorName)
                                                                  // TARGET_INTERCEPTOR赋值为插件拦截器全类名
                                                                  .field(named("TARGET_INTERCEPTOR"))
                                                                  .value(methodsInterceptor)
                                                                  // 组装好字节码还未加载
                                                                  .make();
    
            classesTypeMap.put(internalInterceptorName, interceptorType.getBytes());
    
            InstrumentDebuggingClass.INSTANCE.log(interceptorType);
        } catch (Exception e) {
            throw new PluginException("Generate Dynamic plugin failure", e);
        }
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
generateDelegator()方法就是将模板类交给ByteBuddy去编译成字节码，改了新的类名，并将TARGET_INTERCEPTOR属性赋值为插件拦截器全类名，然后就放入到classesTypeMap中（所有要注入到BootStrapClassLoader中的类）

再回到BootstrapInstrumentBoost的inject()方法：

public class BootstrapInstrumentBoost {

    public static AgentBuilder inject(PluginFinder pluginFinder, Instrumentation instrumentation,
        AgentBuilder agentBuilder, JDK9ModuleExporter.EdgeClasses edgeClasses) throws PluginException {
        // 所有要注入到Bootstrap ClassLoader里的类
        Map<String, byte[]> classesTypeMap = new HashMap<>();
    
        /**
         * 针对于目标类是JDK核心类库的插件,根据插件的拦截点的不同(实例方法、静态方法、构造方法)
         * 使用不同的模板(xxxTemplate)来定义新的拦截器的核心处理逻辑,并且将插件本身定义的拦截器的全类名
         * 赋值给模板的TARGET_INTERCEPTOR字段
         * 最终,这些新的拦截器的核心处理逻辑都会被放入到BootstrapClassLoader中
         */
      	// 1)
        if (!prepareJREInstrumentation(pluginFinder, classesTypeMap)) {
            return agentBuilder;
        }
    
        if (!prepareJREInstrumentationV2(pluginFinder, classesTypeMap)) {
            return agentBuilder;
        }
    
        for (String highPriorityClass : HIGH_PRIORITY_CLASSES) {
            loadHighPriorityClass(classesTypeMap, highPriorityClass);
        }
        for (String highPriorityClass : ByteBuddyCoreClasses.CLASSES) {
            loadHighPriorityClass(classesTypeMap, highPriorityClass);
        }
    
        /**
         * Prepare to open edge of necessary classes.
         */
        for (String generatedClass : classesTypeMap.keySet()) {
            edgeClasses.add(generatedClass);
        }
    
        /**
         * 将生成的类注入到Bootstrap ClassLoader
         * Inject the classes into bootstrap class loader by using Unsafe Strategy.
         * ByteBuddy adapts the sun.misc.Unsafe and jdk.internal.misc.Unsafe automatically.
         */
      	// 2)
        ClassInjector.UsingUnsafe.Factory factory = ClassInjector.UsingUnsafe.Factory.resolve(instrumentation);
        factory.make(null, null).injectRaw(classesTypeMap);
        agentBuilder = agentBuilder.with(new AgentBuilder.InjectionStrategy.UsingUnsafe.OfFactory(factory));
    
        return agentBuilder;
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
代码1)处调用prepareJREInstrumentation()方法，核心逻辑：针对于目标类是JDK核心类库的插件，根据插件的拦截点的不同（实例方法、静态方法、构造方法），使用不同的模板（xxxTemplate）来定义新的拦截器的核心处理逻辑，并且将插件本身定义的拦截器的全类名赋值给模板的TARGET_INTERCEPTOR字段

代码2)处将生成的类注入到Bootstrap ClassLoader

2）、JDK类库方法插桩
无论是静态方法插桩还是构造器和实例方法插桩都会判断是否是JDK类库中的类，如果是会调用BootstrapInstrumentBoost的forInternalDelegateClass()方法，以实例方法插桩为例：

public abstract class ClassEnhancePluginDefine extends AbstractClassEnhancePluginDefine {
    
    protected DynamicType.Builder<?> enhanceInstance(TypeDescription typeDescription,
        // ...
        /**
         * 3. enhance instance methods
         * 增强实例方法
         */
        if (existedMethodsInterceptPoints) {
            for (InstanceMethodsInterceptPoint instanceMethodsInterceptPoint : instanceMethodsInterceptPoints) {
                String interceptor = instanceMethodsInterceptPoint.getMethodsInterceptor();
                if (StringUtil.isEmpty(interceptor)) {
                    throw new EnhanceException("no InstanceMethodsAroundInterceptor define to enhance class " + enhanceOriginClassName);
                }
                ElementMatcher.Junction<MethodDescription> junction = not(isStatic()).and(instanceMethodsInterceptPoint.getMethodsMatcher());
                // 如果拦截点为DeclaredInstanceMethodsInterceptPoint
                if (instanceMethodsInterceptPoint instanceof DeclaredInstanceMethodsInterceptPoint) {
                    // 拿到的方法必须是当前类上的 通过注解匹配可能匹配到很多方法不是当前类上的
                    junction = junction.and(ElementMatchers.<MethodDescription>isDeclaredBy(typeDescription));
                }
                if (instanceMethodsInterceptPoint.isOverrideArgs()) {
                    if (isBootstrapInstrumentation()) {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                    .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                    } else {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .withBinders(Morph.Binder.install(OverrideCallable.class))
                                                                                    .to(new InstMethodsInterWithOverrideArgs(interceptor, classLoader)));
                    }
                } else {
                    if (isBootstrapInstrumentation()) {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .to(BootstrapInstrumentBoost.forInternalDelegateClass(interceptor)));
                    } else {
                        newClassBuilder = newClassBuilder.method(junction)
                                                         .intercept(MethodDelegation.withDefaultConfiguration()
                                                                                    .to(new InstMethodsInter(interceptor, classLoader)));
                    }
                }
            }
        }
    
        return newClassBuilder;
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
BootstrapInstrumentBoost的forInternalDelegateClass()方法源码如下：

public class BootstrapInstrumentBoost {

    public static Class forInternalDelegateClass(String methodsInterceptor) {
        try {
            // methodsInterceptor + "_internal"
            return Class.forName(internalDelegate(methodsInterceptor));
        } catch (ClassNotFoundException e) {
            throw new PluginException(e.getMessage(), e);
        }
    }
1
2
3
4
5
6
7
8
9
10
通过Class.forName()加载插件拦截器全类名+_internal的类，这个类在Agent启动流程根据模板类生成并注入到Bootstrap ClassLoader中，所以这里是能加载到

/**
 * --------CLASS TEMPLATE---------
 * <p>Author, Wu Sheng </p>
 * <p>Comment, don't change this unless you are 100% sure the agent core mechanism for bootstrap class
 * instrumentation.</p>
 * <p>Date, 24th July 2019</p>
 * -------------------------------
 * <p>
 * This class wouldn't be loaded in real env. This is a class template for dynamic class generation.
 * 在真实环境下这个类是不会被加载的,这是一个类模板用于动态类生成
 */
   public class InstanceMethodInterTemplate {
    /**
     * This field is never set in the template, but has value in the runtime.
       */
         private static String TARGET_INTERCEPTOR;

    private static InstanceMethodsAroundInterceptor INTERCEPTOR;
    private static IBootstrapLog LOGGER;

    /**
     * Intercept the target instance method.
       *
     * @param obj          target class instance.
     * @param allArguments all method arguments
     * @param method       method description.
     * @param zuper        the origin call ref.
     * @return the return value of target instance method.
     * @throws Exception only throw exception because of zuper.call() or unexpected exception in sky-walking ( This is a
     *                   bug, if anything triggers this condition ).
       */
         @RuntimeType
         public static Object intercept(@This Object obj, @AllArguments Object[] allArguments, @SuperCall Callable<?> zuper,
        @Origin Method method) throws Throwable {
        EnhancedInstance targetObject = (EnhancedInstance) obj;

        prepare();

        MethodInterceptResult result = new MethodInterceptResult();
        try {
            if (INTERCEPTOR != null) {
                INTERCEPTOR.beforeMethod(targetObject, method, allArguments, method.getParameterTypes(), result);
            }
        } catch (Throwable t) {
            if (LOGGER != null) {
                LOGGER.error(t, "class[{}] before method[{}] intercept failure", obj.getClass(), method.getName());
            }
        }

        Object ret = null;
        try {
            if (!result.isContinue()) {
                ret = result._ret();
            } else {
                ret = zuper.call();
            }
        } catch (Throwable t) {
            try {
                if (INTERCEPTOR != null) {
                    INTERCEPTOR.handleMethodException(targetObject, method, allArguments, method.getParameterTypes(), t);
                }
            } catch (Throwable t2) {
                if (LOGGER != null) {
                    LOGGER.error(t2, "class[{}] handle method[{}] exception failure", obj.getClass(), method.getName());
                }
            }
            throw t;
        } finally {
            try {
                if (INTERCEPTOR != null) {
                    ret = INTERCEPTOR.afterMethod(targetObject, method, allArguments, method.getParameterTypes(), ret);
                }
            } catch (Throwable t) {
                if (LOGGER != null) {
                    LOGGER.error(t, "class[{}] after method[{}] intercept failure", obj.getClass(), method.getName());
                }
            }
        }

        return ret;
         }

    /**
     * Prepare the context. Link to the agent core in AppClassLoader.
       *
     * 1.打通BootstrapClassLoader和AgentClassLoader
     * 拿到ILog生成日志对象
     * 拿到插件自定义的拦截器实例
     * 2.代替非JDK核心类库插件运行逻辑里的
     * InterceptorInstanceLoader.load(instanceMethodsAroundInterceptorClassName, classLoader)
       */
       private static void prepare() {
        if (INTERCEPTOR == null) {
            ClassLoader loader = BootstrapInterRuntimeAssist.getAgentClassLoader();

            if (loader != null) {
                IBootstrapLog logger = BootstrapInterRuntimeAssist.getLogger(loader, TARGET_INTERCEPTOR);
                if (logger != null) {
                    LOGGER = logger;
            
                    INTERCEPTOR = BootstrapInterRuntimeAssist.createInterceptor(loader, TARGET_INTERCEPTOR, LOGGER);
                }
            } else {
                LOGGER.error("Runtime ClassLoader not found when create {}." + TARGET_INTERCEPTOR);
            }
        }
       }
         }
         1
         2
         3
         4
         5
         6
         7
         8
         9
         10
         11
         12
         13
         14
         15
         16
         17
         18
         19
         20
         21
         22
         23
         24
         25
         26
         27
         28
         29
         30
         31
         32
         33
         34
         35
         36
         37
         38
         39
         40
         41
         42
         43
         44
         45
         46
         47
         48
         49
         50
         51
         52
         53
         54
         55
         56
         57
         58
         59
         60
         61
         62
         63
         64
         65
         66
         67
         68
         69
         70
         71
         72
         73
         74
         75
         76
         77
         78
         79
         80
         81
         82
         83
         84
         85
         86
         87
         88
         89
         90
         91
         92
         93
         94
         95
         96
         97
         98
         99
         100
         101
         102
         103
         104
         105
         106
         107
         108
         JDK类库中的类的实例方法插桩且不修改原方法入参会交给通过InstanceMethodInterTemplate生成的类去处理，实际也就是模板类InstanceMethodInterTemplate的TARGET_INTERCEPTOR赋值为插件拦截器全类名，和实例方法插桩的InstMethodsInter的intercept()方法相比这里多调用了一个prepare()方法

prepare()方法处理逻辑如下：

拿到AgentClassLoader
通过AgentClassLoader加载，拿到ILog生成日志对象
通过AgentClassLoader加载，拿到插件自定义的拦截器实例
InstanceMethodInterTemplate生成的类是由BootstrapClassLoader去加载的，而日志对象和插件自定义的拦截器都是通过AgentClassLoader去加载的，prepare()方法本质就是为了打通BootstrapClassLoader和AgentClassLoader


假设BootstrapClassLoader加载的由InstanceMethodInterTemplate生成的类是org.apache.skywalking.xxx.DubboInterceptor_internal，AgentClassLoader加载了日志用到的ILog和插件拦截器DubboInterceptor，AgentClassLoader的顶层父类加载器为BootstrapClassLoader，根据双亲委派模型，从下往上加载是可以拿到的，但是从上往下加载是拿不到的（BootstrapClassLoader中不能到ILog和DubboInterceptor），所以需要通过prepare()方法打通BootstrapClassLoader和AgentClassLoader

小结：



参考：

SkyWalking8.7.0源码分析（如果你对SkyWalking Agent源码感兴趣的话，强烈建议看下该教程）
————————————————
版权声明：本文为CSDN博主「邋遢的流浪剑客」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_40378034/article/details/122278500



https://blog.csdn.net/qq_40378034/article/details/122278500

