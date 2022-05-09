# 监控系统

## 什么是APM系统？

APM系统可以帮助理解系统行为、用于分析性能问题的工具，以便发生故障的时候，能够快速定位和解决问题，这就是APM系统，全称是(**Application Performance Monitor**)。

谷歌公开的论文提到的 Google Dapper可以说是最早的APM系统了，给google的开发者和运维团队帮了大忙，所以谷歌公开论文分享了Dapper。

而后，很多的技术公司基于这篇论文的原理，设计开发了很多出色的APM框架，例如Pinpoint、SkyWalking等。

## APM核心思想是什么？

在服务各节点彼此调用的时候，记录并传递一个应用级别的标记，这个标记可以用来关联各个服务节点之间的关系。比如两个节点之间使用 HTTP 作为请求协议的话，那么这些标记就会被加入到 HTTP 头中。因此如何传递这些标记是与节点之间使用的通讯协议有关的，有些协议就很容易加入这样的内容，但有些协议就相对困难甚至不可能，因此这一点就直接决定了实现分布式追踪系统的难度。

## APM的发展历程

目前APM的发展主要经历了前面的三个阶段：

第一阶段：以网络监控基础设施为主，主要监控主机 的CPU 使用率、I/O、内存资源、网速等，主要以各类网络管理系统（NMS）和各种系统监控工具为代表。

第二阶段：以监控各种基础组件为主，随着互联网的快速发展，为了降低应用开发难度，各种基础组件（如数据库、中间件等）开始大量涌现，所以这个时期应用性能管理主要是监控和管理各种基础组件的性能。

第三阶段：以监控应用本身的性能为主, IT 运维管理的复杂度开始出现爆炸性的增长，应用性能管理的重点也开始聚焦于应用本身的性能与管理上。

第四节阶段属于正在发展的阶段：

云计算方兴未艾，而DevOps以及微服务的兴起对传统APM产生了很大的冲击，传统厂商也在做一些革新，也做一些微服务方面的尝试和云计算方面的尝试。

随着Machine Learning、AI的技术的兴起，对定位故障、定位问题，也会起到一些帮助，基于大数据的分析的手段也会有一些帮助，目前市场上正在初步尝试阶段。

2016年Gartner对APM的定义分为三个维度

1. DEM-Digital experience monitoring：数字体验监控，浏览器及移动设备用户体验监控及利用主动拨测的实现的业务可用性及性能监控。

1. ADTD-Application discovery, tracing and diagnostics：应用自动发现、追踪和故障诊断，自动发现应用之间的逻辑关系，自动建模、应用组件的深入监控及性能关联分析。

1. AA-Application analytics：应用分析，通过机器学习，进行针对JAVA及.NET等应用的根源分析。

## 什么是监控系统？

在微服务架构中，监控系统按照原理和作用可以分为三大类（并非严格分类，仅从日常使用角度来看）：

- 日志类（Logging）
- 调用链类（Tracing）
- 度量类（Metrics）

![APM](png\APM.png)

**Logging**

就是记录系统行为的离散事件，例如，服务在处理某个请求时打印的错误日志，我们可以将这些日志信息记录到 ElasticSearch 或是其他存储中，然后通过 Kibana 或是其他工具来分析这些日志了解服务的行为和状态。大多数情况下，日志记录的数据很分散，并且相互独立，比如错误日志、请求处理过程中关键步骤的日志等等。

**Tracing**

即我们常说的分布式链路追踪，记录单个请求的处理流程，其中包括服务调用和处理时长等信息。在微服务架构系统中一个请求会经过很多服务处理，调用链路会非常长，要确定中间哪个服务出现异常是非常麻烦的一件事。通过分布式链路追踪，运维人员就可以构建一个请求的视图，这个视图上展示了一个请求从进入系统开始到返回响应的整个流程。这样，就可以从中了解到所有服务的异常情况、网络调用，以及系统的性能瓶颈等。

**Metrics**

是系统在一段时间内某一方面的某个度量，可聚合的数据，且通常是固定类型的时序数据，例如，电商系统在一分钟内的请求次数。我们常见的监控系统中记录的数据都属于这个范畴，例如Promethus、Open-Falcon 等，这些监控系统最终给运维人员展示的是一张张二维的折线图。Metrics 是可以聚合的，例如，为电商系统中

## 监控系统关注的对象和指标都是什么？

一般我们做「监控系统」都是需要做分层式监控的，也就是说将我们要监控的对象进行分层，一般主要分为：

1. 系统层：系统层主要是指CPU、磁盘、内存、网络等服务器层面的监控，这些一般也是运维同学比较关注的对象。
2. 应用层：应用层指的是服务角度的监控，比如接口、框架、某个服务的健康状态等，一般是服务开发或框架开发人员关注的对象。
3. 用户层：这一层主要是与用户、与业务相关的一些监控，属于功能层面的，大多数是项目经理或产品经理会比较关注的对象。



知道了监控的分层后，我们再来看一下监控的指标一般有哪些：

1. 延迟时间：主要是响应一个请求所消耗的延迟，比如某接口的HTTP请求平均响应时间为100ms。
2. 请求量：是指系统的容量吞吐能力，例如每秒处理多少次请求（QPS）作为指标。
3. 错误率：主要是用来监控错误发生的比例，比如将某接口一段时间内调用时失败的比例作为指标。

## 好的APM应满足的条件

总的来说，一个优秀的APM系统应该满足以下五个条件

1. 低消耗，高效率：被跟踪的系统为跟踪所付出的系统资源代价要尽量小，现在主流的APM对于系统资源的消耗在2.5%-5%左右，但是这个数值应该越小越好，因为在大规模的分布式系统下，一个单节点的资源是无法把控的，可能是超强配置，也可能是老爷机，只跑几个小服务，但是本身性能已经十分吃紧了，如果这时候跟踪应用再一跑，很可能这个节点就挂掉了，得不偿失。

1. 低侵入性，足够透明：作为跟踪系统，侵入性是不可能不存在的，关键这种侵入性要在哪个层面，如何在越底层的层面上侵入，对于开发者的感知和需要配合跟踪系统的工作就越少，如果在代码层面就需要进行侵入，那对于本身业务就比较复杂的应用来说，代码就更加冗余复杂了，也不利于开发者快节奏的开发。

1. 灵活的延展性：不能随着微服务和集群规模的扩大而使分布式跟踪系统瘫痪，要能够充分考虑到未来分布式服务的规模，跟踪系统至少要在未来几年内完全吃得消。

1. 跟踪数据可视化和迅速反馈：要有可视化的监控界面，从跟踪数据收集、处理到结果的展现尽量做到快速，就可以对系统的异常状况作出快速的反应

1. 持续的监控： 要求分布式跟踪系统必须是7x24小时工作的，否则将难以定位到系统偶尔抖动的行为

## 什么是链路追踪

谷歌在 2010 年 4 月发表了一篇论文《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》介绍了分布式追踪的概念，之后很多互联网公司都开始根据这篇论文打造自己的分布式链路追踪系统。APM 系统的核心技术就是分布式链路追踪。

- 论文在线地址：https://storage.googleapis.com/pub-tools-public-publication-data/pdf/36356.pdf，
- 国内的翻译版：https://bigbully.github.io/Dapper-translation/

在此文中阐述了Google在生产环境下对于分布式链路追踪系统Drapper的设计思路与使用经验。随后各大厂商基于这篇论文都开始自研自家的分布式链路追踪产品。如阿里的Eagle eye(鹰眼)、zipkin，京东的“Hydra”、大众点评的“CAT”、新浪的“Watchman”、唯品会的“Microscope”、窝窝网的“Tracing”都是基于这片文章的设计思路而实现的。所以要学习分布式链路追踪，对于Dapper论文的理解至关重要。但是也带来了新的问题：各家的分布式追踪方案是互不兼容的，这才诞生了 OpenTracing。

OpenTracing 是一个 Library，定义了一套通用的数据上报接口，要求各个分布式追踪系统都来实现这套接口。这样一来，应用程序只需要对接OpenTracing，而无需关心后端采用的到底什么分布式追踪系统，因此开发者可以无缝切换分布式追踪系统，也使得在通用代码库增加对分布式追踪的支持成为可能。

目前，主流的分布式追踪实现基本都已经支持 OpenTracing。

## 分布式追踪系统原理

分布式追踪系统大体分为三个部分，**数据采集**、**数据持久化**、**数据展示**。

数据采集是指在代码中埋点，设置请求中要上报的阶段，以及设置当前记录的阶段隶属于哪个上级阶段。数据持久化则是指将上报的数据落盘存储，数据展示则是前端查询与之关联的请求阶段，并在界面上呈现。

## OpenTracing数据模型

**Trace**

一个 Trace 代表一个事务、请求或是流程在分布式系统中的执行过程。OpenTracing 中的一条 Trace调用链，由多个 Span 组成，一个 Span 代表系统中具有开始时间和执行时长的逻辑单元，Span 一般会有一个名称，一条 Trace中 Span 是首尾连接的。

**Span**

Span 代表系统中具有开始时间和执行时长的逻辑单元，Span 之间通过嵌套或者顺序排列建立逻辑因果关系。

如果按时间关系呈现的话如下所示：

```plain
––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time
[Span A···················································]
[Span B··············································]
[Span D··········································]
[Span C········································]
[Span E·······] [Span F··] [Span G··] [SpanH··]
```

每个 Span 中可以包含以下的信息：

- **操作名称**：例如访问的具体 RPC 服务，访问的 URL 地址等；
- **起始时间**：2021-1-25 22:00:00
- **结束时间**：2021-1-30 22:00:00
- **Span Tag**：一组键值对（k-v）构成的Span标签集合，其中键必须为字符串类型，值可以是字符串、bool 值或者数字；
- **Span Log**：一组 Span 的日志集合；
- **SpanContext**：Trace 的全局上下文信息；
- **References**：Span 之间的引用关系，下面详细说明 Span 之间的引用关系；

在一个 Trace 中，一个 Span 可以和一个或者多个 Span 间存在因果关系。

目前，OpenTracing 定义了 ChildOf 和 FollowsFrom 两种 Span 之间的引用关系。这两种引用类型代表了子节点和父节点间的直接因果关系。

- **ChildOf 关系**：一个 Span 可能是一个父级 Span 的孩子，即为 ChildOf 关系。

下面这些情况会构成 ChildOf 关系：一个 HTTP 请求之中，被调用的服务端产生的 Span，与发起调用的客户端产生的 Span，就构成了 ChildOf 关系；一个 SQL Insert 操作的 Span，和 ORM 的 save 方法的 Span构成 ChildOf 关系。

很明显，上述 ChildOf 关系中的父级 Span 都要等待子 Span 的返回，子 Span 的执行时间影响了其所在父级 Span 的执行时间，父级 Span 依赖子 Span 的执行结果。除了串行的任务之外，我们的逻辑中还有很多并行的任务， 它们对应的 Span 也是并行的，这种情况下一个父级 Span 可以合并所有子 Span 的执行结果并等待所有并行子 Span 结束。 

- **FollowsFrom 关系**：表示跟随关系，意为在某个阶段之后发生了另一个阶段，用来描述顺序执行关系
- **Logs** 每个 Span 可以进行多次 Logs 操作，每一次 Logs 操作，都需要带一个时间戳，以及一个可选的附加信息。
- **Tags** 每个 Span 可以有多个键值对形式的 Tags，Tags 是没有时间戳的，只是为Span 添加一些简单解释和补充信息。
- **SpanContext 和 Baggage** SpanContext 表示进程边界，在跨进调用时需要将一些全局信息，例如：TraceId、当前 SpanId 等信息封装到 Baggage 中传递到另一个进程（下游系统）中。

## OpenTracing核心接口语义

**Span 接口**

Span接口必须实现以下的功能：

- 获取关联的 SpanContext：通过 Span 获取关联的SpanContext 对象。
- 关闭（Finish）Span：完成已经开始的 Span。
- 添加 Span Tag：为 Span 添加 Tag 键值对。
- 添加 Log：为 Span 增加一个 Log 事件。
- 添加 Baggage Item：向 Baggage 中添加一组键值对。
- 获取 Baggage Item：根据 Key 获取 Baggage 中的元素。

**SpanContext 接口**

SpanContext 接口必须实现以下功能：

- 用户可以通过 Span 实例或者 Tracer 的 Extract 能力获取 SpanContext 接口实例。

**Tracer 接口**

Tracer 接口必须实现以下功能：

- 创建 Span：创建新的 Span。
- 注入 SpanContext：主要是将跨进程调用携带的 Baggage 数据记录到当前 SpanContext 中。提取 SpanContext ，主要是将当前 SpanContext 中的全局信息提取出来，封装成 Baggage 用于后续的跨进程调用。

## 常见的APM系统

我们前面提到了APM系统，APM 系统（Application PerformanceManagement，即应用性能管理）是对企业的应用系统进行实时监控，实现对应用性能管理和故障定位的系统化解决方案，在运维中常用。

**CAT**（开源）： 

由国内美团点评开源的，基于 Java 语言开发，目前提供 Java、C/C++、Node.js、Python、Go 等语言的客户端，监控数据会全量统计。

CAT是由美团点评开源的项目，基于Java开发的实时应用监控平台，包括实时应用监控，业务监控，可以提供十几张报表展示。Cat的定位是实时监控平台，但与其说是监控平台，更像是个数据仓库，在数据仓库的基础上提供丰富的报表分析功能。不过CAT实现跟踪的手段，是要在代码里硬编码写一些“埋点”，也就是侵入式的。这样做有利有弊，好处是可以在自己需要的地方加埋点，比较有针对性；坏处是必须改动现有系统，很多开发团队不愿意。

github地址：https://github.com/dianping/cat

国内很多公司在用，例如美团点评、携程、拼多多等。CAT 需要开发人员手动在应用程序中埋点，对代码侵入性比较强。

**Zipkin**（开源）：

由 Twitter 公司开发并开源，Java 语言实现。侵入性相对于 CAT 要低一点，需要对web.xml 等相关配置文件进行修改，但依然对系统有一定的侵入性。

这个是twitter开源出来的，也是参考Dapper的体系来做的。Zipkin的Java应用端是通过一个叫Brave的组件来实现对应用内部的性能分析数据采集。Brave（https://github.com/openzipkin/brave）这个组件通过实现一系列的Java拦截器，来做到对http/servlet请求、数据库访问的调用过程跟踪。然后通过在spring之类的配置文件里加入这些拦截器，完成对Java应用的性能数据采集。

github地址：https://github.com/openzipkin/zipkin

Zipkin 可以轻松与 Spring Cloud 进行集成，也是 Spring Cloud 推荐的 APM 系统。

**Pinpoint**（开源）：

韩国团队开源的 APM 产品，运用了字节码增强技术，只需要在启动时添加启动参数即可实现 APM 功能，对代码无侵入。Pinpoint是一个APM（应用程序性能管理）工具，适用于用Java / PHP编写的大型分布式系统。

受Dapper的启发，Pinpoint提供了一种解决方案，通过跟踪分布式应用程序之间的事务，帮助分析系统的整体结构以及它们中的组件如何相互连接。

对Java领域的性能分析有兴趣的都应该看看这个开源项目，这个是一个韩国团队实现并开源出来的，通过JavaAgent的机制来做字节码代码植入(另外还有ASM字节码技术)，实现加入traceid和抓取性能数据的目的。

github地址：https://github.com/naver/pinpoint

目前支持 Java 和 PHP 语言，底层采用 HBase 来存储数据，探针收集的数据粒度非常细，但性能损耗较大，因其出现的时间较长，完成度也很高，文档也较为丰富，应用的公司较多。

**SkyWalking**（开源）：

国人开源的产品，2019 年 4 月 17 日 SkyWalking 从 Apache 基金会的孵化器毕业成为顶级项目。

Skywalking是由国内一位叫吴晟的工程师开源，已加入Apache孵化器，是一个APM系统，为微服务架构和云原生架构系统设计。它通过探针自动收集所需的指标，并进行分布式追踪。通过这些调用链路以及指标，Skywalking APM会感知应用间关系和服务间关系，并进行相应的指标统计。Skywalking支持链路追踪和监控应用组件基本涵盖主流框架和容器，如国产PRC Dubbo和motan等，国际化的spring boot，spring cloud。

github地址：https://github.com/apache/incubator-skywalking

目前SkyWalking 支持 Java、.Net、Node.js 等探针，数据存储支持MySQL、ElasticSearch等。还有很多不开源的 APM 系统，例如，淘宝鹰眼、Google Dapper 等等。

## **Zipkin、Pinpoint、SkyWalking、CAT、Jaeger对比**

[Zipkin](https://github.com/openzipkin/zipkin)是Twitter开源的调用链分析工具，目前基于springcloud sleuth得到了广泛的使用，特点是轻量，使用部署简单。

[Pinpoint](https://github.com/naver/pinpoint)是韩国人开源的基于字节码注入的调用链分析，以及应用监控分析工具。特点是支持多种插件，UI功能强大，接入端无代码侵入。

[SkyWalking](https://github.com/apache/skywalking)是本土开源的基于字节码注入的调用链分析，以及应用监控分析工具。特点是支持多种插件，UI功能较强，接入端无代码侵入。目前已成为Apache顶级项目。

[CAT](https://github.com/dianping/cat)是大众点评开源的基于编码和配置的调用链分析，应用监控分析，日志采集，监控报警等一系列的监控平台工具。

[Jaeger](https://github.com/jaegertracing/jaeger)Uber开源，在界面上较为完善（对比zipkin），但是也无告警功能，代码入侵度也比zipkin小。go语言开发，完全按照CNCF的标准。

## 对比



| 方案       | 依赖                                                      | 实现方式                                   | 存储                                            | JVM监控 | trace查询    | 侵入         | 部署成本 |
| ---------- | --------------------------------------------------------- | ------------------------------------------ | ----------------------------------------------- | ------- | ------------ | ------------ | -------- |
| Pinpoint   | Java 6，7，8 maven3+ Hbase0.94+                           | java探针，字节码增强                       | HBase                                           | 支持    | 需要二次开发 | 最低         | 较高     |
| SkyWalking | Java 6，7，8 maven3.0+ nodejs zookeeper elasticsearch     | java探针，字节码增强                       | elasticsearch , H2 ，mysql,TIDN,Sharding Sphere | 支持    | 支持         | 低           | 低       |
| Zipkin     | Java 6，7，8 Maven3.2+ rabbitMQ                           | 拦截请求，发送（HTTP，mq）数据至zipkin服务 | 内存 ， mysql ， Cassandra ， Elasticsearch     | 不支持  | 支持         | 高，需要开发 | 中       |
| CAT        | Java 6 7 8、Maven 3+ MySQL 5.6 5.7、Linux 2.6+ hadoop可选 | 代码埋点（拦截器，注解，过滤器等）         | mysql , hdfs                                    | 不支持  | 支持         | 高，需要埋点 | 中       |

基于对程序源代码和配置文件的低侵入考虑，推荐的选型顺序依次是 Pinpoint > SkyWalking > Zipkin > CAT

Pinpoint：基本不用修改源码和配置文件，只要在启动命令里指定javaagent参数即可，对于运维人员来讲最为方便；

SkyWalking：不用修改源码，需要修改配置文件；

Zipkin：需要对Spring、web.xml之类的配置文件做修改，相对麻烦一些；

CAT：因为需要修改源码设置埋点，因此基本不太可能由运维人员单独完成，而必须由开发人员的深度参与了；

相对于传统的监控软件（Zabbix之流）的区别，APM跟关注在对于系统内部执行、系统间调用的性能瓶颈分析，这样更有利于定位到问题的具体原因，而不仅仅像传统监控软件一样只提供一些零散的监控点和指标，就算告警了也不知道问题是出在哪里。

## 基本原理

| 类别     | Zipkin                                     | Pinpoint             | SkyWalking           | CAT                                | Jaeger   |
| -------- | ------------------------------------------ | -------------------- | -------------------- | ---------------------------------- | -------- |
| 实现方式 | 拦截请求，发送（HTTP，mq）数据至zipkin服务 | java探针，字节码增强 | java探针，字节码增强 | 代码埋点（拦截器，注解，过滤器等） | 拦截请求 |

## 接入

| 类别                   | Zipkin                                  | Pinpoint        | SkyWalking      | CAT      | Jaeger          |
| ---------------------- | --------------------------------------- | --------------- | --------------- | -------- | --------------- |
| 接入方式               | 基于linkerd或者sleuth方式，引入配置即可 | javaagent字节码 | javaagent字节码 | 代码侵入 | 引入配置        |
| agent到collector的协议 | http,MQ                                 | thrift          | gRPC            | http/tcp | http/udp/thrift |
| OpenTracing            | √                                       | ×               | √               | ×        | √               |

## 分析

| 类别         | Zipkin | Pinpoint | SkyWalking | CAT    | Jaeger |
| ------------ | ------ | -------- | ---------- | ------ | ------ |
| 颗粒度       | 接口级 | 方法级   | 方法级     | 代码级 | 接口级 |
| 全局调用统计 | ×      | √        | √          | √      | ×      |
| traceid查询  | √      | ×        | √          | ×      | √      |
| 报警         | ×      | √        | √          | √      | ×      |
| JVM监控      | ×      | ×        | √          | √      | ×      |

## 页面UI展示

| 类别   | Zipkin | Pinpoint | SkyWalking | CAT   | Jaeger |
| ------ | ------ | -------- | ---------- | ----- | ------ |
| 健壮度 | **     | *****    | ****       | ***** | ***    |

## 数据存储

| 类别     | Zipkin                     | Pinpoint | SkyWalking                                               | CAT         | Jaeger                     |
| -------- | -------------------------- | -------- | -------------------------------------------------------- | ----------- | -------------------------- |
| 数据存储 | ES、MySQL、Cassandra、内存 | Hbase    | ES、H2、MySQL、TiDB、Sharding-Sphere、InfluxDB（研发中） | MySQL、HDFS | ES、MySQL、Cassandra、内存 |

## PinPoint和skyWalking支持的插件对比

| 类别       | Pinpoint                                           | SkyWalking                             |
| ---------- | -------------------------------------------------- | -------------------------------------- |
| web容器    | Tomcat6/7/8,Resin,Jetty,JBoss,Websphere            | Tomcat7/8/9,Resin,Jetty                |
| JDBC       | Oracle,mysql                                       | Oracle,mysql,Sharding-JDBC             |
| 消息中间件 | ActiveMQ, RabbitMQ                                 | RocketMQ 4.x,Kafka                     |
| 日志       | log4j, Logback                                     | log4j,log4j2, Logback                  |
| HTTP库     | Apache HTTP Client, GoogleHttpClient, OkHttpClient | Apache HTTP Client, OkHttpClient,Feign |
| Spring体系 | spring,springboot                                  | spring,springboot,eureka,hystrix       |
| RPC框架    | Dubbo,Thrift                                       | Dubbo,Motan,gRPC,ServiceComb           |
| NOSQL      | Memcached, Redis, CASSANDRA                        | Memcached, Redis                       |

## 社区活跃度（github Star）

| 类别       | [Zipkin](https://starchart.cc/openzipkin/zipkin) | [Pinpoint](https://starchart.cc/naver/pinpoint) | [SkyWalking](https://starchart.cc/apache/skywalking) | [CAT](https://starchart.cc/dianping/cat) | [Jaeger](https://starchart.cc/jaegertracing/jaeger) |
| ---------- | ------------------------------------------------ | ----------------------------------------------- | ---------------------------------------------------- | ---------------------------------------- | --------------------------------------------------- |
| 2019-09-29 | 11.7k                                            | 9.4k                                            | 10.5k                                                | 11.5k                                    | 8.5k                                                |
| 2020-02-23 | 12.5k                                            | 9.9k                                            | 12.3k                                                | 12.7k                                    | 10.3k                                               |

## PinPoint和skyWalking支持的插件对比

| 类别       | Pinpoint                                           | SkyWalking                             |
| ---------- | -------------------------------------------------- | -------------------------------------- |
| web容器    | Tomcat6/7/8,Resin,Jetty,JBoss,Websphere            | Tomcat7/8/9,Resin,Jetty                |
| JDBC       | Oracle,mysql                                       | Oracle,mysql,Sharding-JDBC             |
| 消息中间件 | ActiveMQ, RabbitMQ                                 | RocketMQ 4.x,Kafka                     |
| 日志       | log4j, Logback                                     | log4j,log4j2, Logback                  |
| HTTP库     | Apache HTTP Client, GoogleHttpClient, OkHttpClient | Apache HTTP Client, OkHttpClient,Feign |
| Spring体系 | spring,springboot                                  | spring,springboot,eureka,hystrix       |
| RPC框架    | Dubbo,Thrift                                       | Dubbo,Motan,gRPC,ServiceComb           |
| NOSQL      | Memcached, Redis, CASSANDRA                        | Memcached, Redis                       |

## 性能分析

摘自：[https://juejin.im/post/5a7a9e0af265da4e914b46f1](https://link.jianshu.com/?t=https%3A%2F%2Fjuejin.im%2Fpost%2F5a7a9e0af265da4e914b46f1)

模拟了三种并发用户：500，750，1000。使用jmeter测试，每个线程发送30个请求，设置思考时间为10ms。使用的采样率为1，即100%，这边与生产可能有差别。pinpoint默认的采样率为20，即50%，通过设置agent的配置文件改为100%。zipkin默认也是1。组合起来，一共有12种。

在三种链路监控组件中，skywalking的探针对吞吐量的影响最小，zipkin的吞吐量居中。pinpoint的探针对吞吐量的影响较为明显，在500并发用户时，测试服务的吞吐量从1385降低到774，影响很大。然后再看下CPU和memory的影响，在内部服务器进行的压测，对CPU和memory的影响都差不多在10%之内。

#### skywalking官方性能测试：https://github.com/SkyAPMTest/Agent-Benchmarks

## 总结

主流APM工具为了更好地进行推广，主要采用了侵入程度低的方式完成对应用代码的改造。并且为了应对云计算、微服务、容器化的迅速发展与应用带来的APM监控的数据的海量增长的趋势，数据落地方式也主要以海量存储数据库为主。

未来在数据分析和性能分析方面，大数据和机器学习将在APM领域发挥重要的作用，APM的功能也将从单一的资源监控和应用监控，向异常检测、性能诊断、未来预测等自动化、智能化等方向发展。