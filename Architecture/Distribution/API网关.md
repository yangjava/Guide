# API网关

## **API网关是什么？**

API网关是随着微服务（Microservice）概念兴起的一种架构模式。原本一个庞大的单体应用（All in one）业务系统被拆分成许多微服务（Microservice）系统进行独立的维护和部署，服务拆分带来的变化是API的规模成倍增长，API的管理难度也在日益增加，使用API网关发布和管理API逐渐成为一种趋势。一般来说，API网关是运行于外部请求与内部服务之间的一个流量入口，实现对外部请求的协议转换、鉴权、流控、参数校验、监控等通用功能。

- 限流：通过令牌桶等算法，把一些额外的流量挡在系统外面，不让其访问。
   
-  降级：由于系统可能已经过载了，此时，我们就放弃处理一些服务和页面的请求或者仅简单处理，比如直接返回一个报错。
   
-  熔断：有些时候，系统过载过度或者上线出了 bug，降级都解决不了问题。比如，缓存失效了，导致大量请求频繁访问了数据库，而这种频繁访问数据库可能造成了大量的 IO 操作，结果又去影响了数据库所在的操作系统，同时，这个操作系统上又有着别的重要服务，直接也被影响了。对于这种连锁反应，我们称之为雪崩。而为了防止雪崩，我们就会坚决把缓存失效导致数据库被频繁访问的服务给停掉，这就是熔断。
   

可以看到，像限流、降级、熔断这些系统保障策略，最合适的地方应该是有一个集中的请求入口点，就像古时候，老百姓进城需要过城门那样。

## **为什么要做API网关？**

在没有API网关之前，业务研发人员如果要将内部服务输出为对外的HTTP API接口。通常要搭建一个Web应用，用于完成基础的鉴权、限流、监控日志、参数校验、协议转换等工作，同时需要维护代码逻辑、基础组件的升级，研发效率相对比较低。此外，每个Web应用都需要维护机器、配置、数据库等，资源利用率也非常差。

更好的方式是采用API网关，实现一个API网关接管所有的入口流量，类似Nginx的作用，将所有用户的请求转发给后端的服务器，但网关做的不仅仅只是简单的转发，也会针对流量做一些扩展，比如鉴权、限流、权限、熔断、协议转换、错误码统一、缓存、日志、监控、告警等，这样将通用的逻辑抽出来，由网关统一去做，业务方也能够更专注于业务逻辑，提升迭代的效率。
通过引入API网关，客户端只需要与API网关交互，而不用与各个业务方的接口分别通讯，但多引入一个组件就多引入了一个潜在的故障点，因此要实现一个高性能、稳定的网关，也会涉及到很多点。

但对于服务数量众多、复杂度比较高、规模比较大的业务来说，引入 API 网关也有一系列的好处：

- 聚合接口使得服务对调用者透明，客户端与后端的耦合度降低
- 聚合后台服务，节省流量，提高性能，提升用户体验
- 提供安全、流控、过滤、缓存、计费、监控等 API 管理功能

客户往往需要通过统一的 API 网关进行服务能力的共享，提供发布、管理、保护和监控 API的能力，实现跨系统、跨协议的服务能力互通。

一是客户需要通过 API 网关的熔断/限流/降级服务治理等能力和服务治理框架层相结合，来保证物流系统更好地支撑峰值流量的冲击；

二是有些特殊场景的接口，比如支付接口，需要设置调用权限，API 网关配合中间框架更好实现黑白名单和权限的控制；

三是网关的流量镜像能力，可以转发到压测环境，客户能够更好地估计系统能够承载的最大量；

此外，API 网关通过流量的控制，还可以让客户更快地做好灰度发布、A/B 测试。

## 功能设计

一个API网关的基本功能包含了统一接入、协议适配、流量管理与容错、以及安全防护，这四大基本功能构成了网关的核心功能。网关首要的功能是负责统一接入，然后将请求的协议转换成内部的接口协议，在调用的过程中还要有限流、降级、熔断等容错的方式来保护网关的整体稳定，同时网关还要做到基本的安全防护（防刷控制），以及黑白名单（比如IP白名单）等基本安全措施

API 网关中的服务路由和服务治理：

**服务路由**

- 静态路由策略配置
- 后端服务的软负载均衡
- 后端服务的心跳检查
- 参数分流
- 流量的镜像复制
- 动态发现：加入后端微服务中心，实现动态发现后端服务实例

**服务治理**

- 后端服务的故障隔离
- 网关、服务、API 级别的限流和熔断
- 固定时段和周期时段的 API 维护开关

服务治理框架层的服务治理和流量管理：

**服务治理**

- 服务限流，支持 QPS、Thread 等多种限流方式
- 降级与熔断，支持基于RT、错误率的熔断策略以及手动降级策略
- 服务容错，支持 failover、failfast、failback等多种容错机制

**流量管理**

- 路由管理，支持基于黑白名单的路由规则
- 负载均衡，支持多种负载均衡规则，兼容 Spring Cloud Ribbon
- 参数分流，支持参数取模、名单分流、权重分流等
- 反向代理：类似于Nginx效果，实现外部HTTP请求反向代理转为内部RPC请求进行转发

## 架构示例

除了基本的四大功能，网关运行良好的环境还包括注册中心（比如：ZK读取已发布的API接口的动态配置）。为了实现高性能，将数据全部异构到缓存（如：Redis）中，同时还可以配合本地缓存来进一步提高网关系统的性能。为了提高网关的吞吐率，可以使用NIO+Servlet 3 异步的方式，还可以利用Servlet 3 的异步特性将请求线程与业务线程分开，为后续的线程池隔离做好基本的支撑。访问日志的存储我们可以放到Hbase中，如果要作为开放网关使用，那么需要一个支持OAuth2.0的授权中心。还可以引入Nginx + lua的方式将一些基本的校验判断放到应用系统之上，这样可以更轻量化的处理接入的问题，整体的网关架构示例如下所示：

![API网关](API网关.png)

### API注册

业务方如何接入网关?一般来说有几种方式。

- 第一种采用插件扫描业务方的API，比如Spring MVC的注解，并结合Swagger的注解，从而实现参数校验、文档&&SDK生成等功能，扫描完成之后，需要上报到网关的存储服务。
- 手动录入。比如接口的路径、请求参数、响应参数、调用方式等信息，但这种方式相对来说会麻烦一些，如果参数过多的话，前期录入会很费时费力。

- 配置文件导入。比如通过Swagger\OpenAPI等，比如阿里云的网关:

业务研发人员从创建API开始，完成参数录入、DSL脚本生成；接着可以通过文档和MOCK功能进行API测试；API测试完成后，为了保证上线稳定性，管理平台提供了发布审批、灰度上线、版本回滚等一系列安全保证措施；API运行期间会监控API的调用失败情况、记录请求日志，一旦发现异常及时发出告警；最后，对于不再使用的API进行下线操作后，会回收API所占用的各类资源并等待重新启用。整个生命周期，全部通过配置化、流程化的方式，由业务研发人员全自助管理，上手时间基本在10分钟以内，极大地提升了研发效率。

### 配置中心

API网关的配置中心存放API的相关配置信息——使用自定义的DSL（Domain-Specific Language，领域专用语言）来描述，用于向API网关的数据面下发API的路由、规则、组件等配置变更。配置中心的设计上使用统一配置管理服务和本地缓存结合的方式，实现动态配置，不停机发布。

API配置的详细说明

- **Name、Group**：名字、所属分组
- **Request**：请求的域名、路径、参数等信息
- **Response**：响应的结果组装、异常处理、Header、Cookies信息
- **Filters、FilterConfigs**：API使用到的功能组件和配置信息
- **Invokers**：后端服务(RPC/HTTP/Function)的请求规则和编排信息

### API路由

API网关的数据面在感知到API配置后，会在内存中建立请求路径与API配置的路由信息。通常HTTP请求路径上，会包含一些路径变量，考虑到性能问题，没有采用正则匹配的方式，而是设计了两种数据结构来存储。

一种是不包含路径变量的直接映射的MAP结构。其中，Key就是完整的域名和路径信息，Value是具体的API配置。

另外一种是包含路径变量的前缀树数据结构。通过前缀匹配的方式，先进行叶子节点精确查找，并将查找节点入栈处理，如果匹配不上，则将栈顶节点出栈，再将同级的变量节点入栈，如果仍然找不到，则继续回溯，直到找到（或没找到）路径节点并退出。

### 协议转换&服务调用

API调用的最后一步，就是协议转换以及服务调用了。网关需要完成的工作包括：获取HTTP请求参数、Context本地参数，拼装后端服务参数，完成HTTP协议到后端服务的协议转换，调用后端服务获取响应结果并转换为HTTP响应结果。

内部的API可能是由很多种不同的协议实现的，比如HTTP、Dubbo、GRPC等，但对于用户来说其中很多都不是很友好，或者根本没法对外暴露，比如Dubbo服务，因此需要在网关层做一次协议转换，将用户的HTTP协议请求，在网关层转换成底层对应的协议，比如`HTTP -> Dubbo`, 但这里需要注意很多问题，比如参数类型，如果类型搞错了，导致转换出问题，而日志又不够详细的话，问题会很难定位。

以调用后端RPC服务为例，通过JsonPath表达式获取HTTP请求不同部位的参数值，替换RPC请求参数相应部位的Value，生成服务参数DSL，最后借助RPC泛化调用完成本次服务调用。

### 服务发现

网关作为流量的入口，负责请求的转发，但首先需要知道转发给谁，如何寻址，这里有几种方式:

- 写死在代码/配置文件里，这种方式虽然比较挫，但也能使用，比如线上仍然使用的是物理机，IP变动不会很频繁，但扩缩容、包括应用上下线都会很麻烦，网关自身甚至需要实现一套健康监测机制。
- 域名。采用域名也是一种不错的方案，对于所有的语言都适用，但对于内部的服务，走域名会很低效，另外环境隔离也不太友好，比如预发、线上通常是同一个数据库，因此网关读取到的可能是同一个域名，这时候预发的网关调用的就是线上的服务。
- 注册中心。采用注册中心就不会有上述的这些问题，即使是在容器环境下，节点的IP变更比较频繁，但节点列表的实时维护会由注册中心搞定，对网关是透明的，另外应用的正常上下线、包括异常宕机等情况，也会由注册中心的健康检查机制检测到，并实时反馈给网关。并且采用注册中心性能也没有额外的性能损耗，采用域名的方式，额外需要走一次DNS解析、Nginx转发等，中间多了很多跳，性能会有很大的下降，但采用注册中心，网关是和业务方直接点对点的通讯，不会有额外的损耗。

### 服务调用

网关由于对接很多种不同的协议，因此可能需要实现很多种调用方式，比如HTTP、Dubbo等，基于性能原因，最好都采用异步的方式，而Http、Dubbo都是支持异步的，比如apache就提供了基于NIO实现的异步HTTP客户端。
因为网关会涉及到很多异步调用，比如拦截器、HTTP客户端、dubbo、redis等，因此需要考虑下异步调用的方式，如果基于回调或者future的话，代码嵌套会很深，可读性很差，可以参考zuul和spring cloud gateway的方案，基于响应式进行改造。

### 优雅下线

优雅下线也是网关需要关注的一个问题，网关底层会涉及到很多种协议，比如HTTP、Dubbo，而HTTP又可以继续细分，比如域名、注册中心等，有些自身就支持优雅下线，比如Nginx自身是支持健康监测机制的，如果检测到某一个节点已经挂掉了，就会把这个节点摘掉，对于应用正常下线，需要结合发布系统，首先进行逻辑下线，然后对后续Nginx的健康监测请求直接返回失败(比如直接返回500),然后等待一段时间(根据Nginx配置决定)，然后再将应用实际下线掉。另外对于注册中心的其实也类似，一般注册中心是只支持手动下线的，可以在逻辑下线阶段调用注册中心的接口将节点下线掉，而有些不支持主动下线的，需要结合缓存的配置，让应用延迟下线。另外对于其他比如Dubbo等原理也是类似。

### 性能

网关作为所有流量的入口，性能是重中之重，早期大部分网关都是基于同步阻塞模型构建的，比如Zuul 1.x。但这种同步的模型我们都知道，每个请求/连接都会占用一个线程，而线程在JVM中是一个很重的资源，比如Tomcat默认就是200个线程，如果网关隔离没有做好的话，当发生网络延迟、FullGC、第三方服务慢等情况造成上游服务延迟时，线程池很容易会被打满，造成新的请求被拒绝，但这个时候其实线程都阻塞在IO上，系统的资源被没有得到充分的利用。另外一点，容易受网络、磁盘IO等延迟影响。需要谨慎设置超时时间，如果设置不当，且服务隔离做的不是很完善的话，网关很容易被一个慢接口拖垮。

而异步化的方式则完全不同，通常情况下一个CPU核启动一个线程即可处理所有的请求、响应。一个请求的生命周期不再固定于一个线程，而是会分成不同的阶段交由不同的线程池处理，系统的资源能够得到更充分的利用。而且因为线程不再被某一个连接独占，一个连接所占用的系统资源也会低得多，只是一个文件描述符加上几个监听器等，而在阻塞模型中，每条连接都会独占一个线程，而线程是一个非常重的资源。对于上游服务的延迟情况，也能够得到很大的缓解，因为在阻塞模型中，慢请求会独占一个线程资源，而异步化之后，因为单条连接所占用的资源变的非常低，系统可以同时处理大量的请求。
如果是JVM平台，Zuul 2、Spring Cloud gateway等都是不错的异步网关选型，另外也可以基于Netty、Spring Boot2.x的webflux、vert.x或者servlet3.1的异步支持进行自研。

### 缓存

对于一些幂等的get请求，可以在网关层面根据业务方指定的缓存头做一层缓存，存储到Redis等二级缓存中，这样一些重复的请求，可以在网关层直接处理，而不用打到业务线，降低业务方的压力，另外如果业务方节点挂掉，网关也能够返回自身的缓存。

### 稳定性

稳定性是网关非常重要的一环，监控、告警需要做的很完善才可以，比如接口调用量、响应时间、异常、错误码、成功率等相关的监控告警，还有线程池相关的一些，比如活跃线程数、队列积压等，还有些系统层面的，比如CPU、内存、FullGC这些基本的。
网关是所有服务的入口，对于网关的稳定性的要求相对于其他服务会更高，最好能够一直稳定的运行，尽量少重启，但当新增功能、或者加日志排查问题时，不可避免的需要重新发布，因此可以参考zuul的方式，将所有的核心功能都基于不同的拦截器实现，拦截器的代码采用Groovy编写，存储到数据库中，支持动态加载、编译、运行，这样在出了问题的时候能够第一时间定位并解决，并且如果网关需要开发新功能，只需要增加新的拦截器，并动态添加到网关即可，不需要重新发布。

### 限流

限流对于每个业务组件来说，可以说都是一个必须的组件，如果限流做不好的话，当请求量突增时，很容易导致业务方的服务挂掉，比如双11、双12等大促时，接口的请求量是平时的数倍，如果没有评估好容量，又没有做限流的话，很容易服务整个不可用，因此需要根据业务方接口的处理能力，做好限流策略，相信大家都见过淘宝、百度抢红包时的降级页面。
因此一定要在接入层做好限流策略，对于非核心接口可以直接将降级掉，保障核心服务的可用性，对于核心接口，需要根据压测时得到的接口容量，制定对应的限流策略。限流又分为几种:

- 单机。单机性能比较高，不涉及远程调用，只是本地计数，对接口RT影响最小。但需要考虑下限流数的设置，比如是针对单台网关、还是整个网关集群，如果是整个集群的话，需要考虑到网关缩容、扩容时修改对应的限流数。
- 分布式。分布式的就需要一个存储节点维护当前接口的调用数，比如redis、sentinel等，这种方式由于涉及到远程调用，会有些性能损耗，另外也需要考虑到存储挂掉的问题，比如redis如果挂掉，网关需要考虑降级方案，是降级到本地限流，还是直接将限流功能本身降级掉。
  另外还有不同的策略:简单计数、令牌桶等，大部分场景下其实简单计数已经够用了，但如果需要支持突发流量等场景时，可以采用令牌桶等方案。还需要考虑根据什么限流，比如是IP、接口、用户维度、还是请求参数中的某些值，这里可以采用表达式，相对比较灵活。

### 熔断降级

熔断机制也是非常重要的一项。若某一个服务挂掉、接口响应严重超时等发生，则可能整个网关都被一个接口拖垮，因此需要增加熔断降级，当发生特定异常的时候，对接口降级由网关直接返回，可以基于Hystrix或者Resilience4j实现。

### 功能组件

当请求流量命中API请求路径进入服务端，具体处理逻辑由DSL中配置的一系列功能组件完成。网关提供了丰富的功能组件集成，包括链路追踪、实时监控、访问日志、参数校验、鉴权、限流、熔断降级、灰度分流等，如下图所示：

![API网关-功能组件](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-功能组件.png)







### 高性能设计

![API网关-高性能设计](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-高性能设计.png)



### 稳定性保障

提供了一些常规的稳定性保障手段，来保证自身和后端服务的可用性。如下图所示：

![API网关-稳定性保障](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-稳定性保障.png)

- **流量管控**：从用户自定义UUID限流、App限流、IP限流、集群限流等多个维度提供流量保护
- **请求缓存**：对于一些幂等的、查询频繁的、数据及时性不敏感的请求，业务研发人员可开启请求缓存功能
- **超时管理**：每个API都设置了处理超时时间，对于超时的请求，进行快速失败的处理，避免资源占用
- **熔断降级**：支持熔断降级功能，实时监控请求的统计信息，达到配置的失败阈值后，自动熔断，返回默认值



### 故障自愈

API网关服务端接入了弹性伸缩模块，可根据CPU等指标进行快速扩容、缩容。除此之外，还支持快速摘除问题节点，以及更细粒度的问题组件摘除。

![API网关-故障自愈](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-故障自愈.png)



### 可迁移

对于一些已经在对外提供API的Web服务，业务研发人员为了减少运维成本和后续的研发提效，考虑将其迁移到API网关。对于一些非核心API，可以考虑使用灰度发布功能直接迁移。但是对于一些核心API，上面的灰度发布功能是机器级别的，粒度较大，不够灵活，不能很好的支持灰度验证过程。



**解决方案**

API网关为业务研发人员提供了一个灰度SDK，接入SDK的Web服务，可在识别灰度流量后转发到API网关进行验证。灰度哪些API、灰度百分比可以在API网关管理端动态调节，实时生效，业务研发人员还可以通过SPI的方式自定义灰度策略。灰度验证通过后，再把API迁移到API网关，保障迁移过程的稳定性。



**灰度过程**

**灰度前**：在API网关管理平台创建API分组，域名配置为目前使用的域名。在Oceanus上，原域名规则不变。

![API网关-灰度前](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-灰度前.png)

**灰度中**：在API网关管理平台开启灰度功能，灰度SDK将灰度流量转发到网关服务，进行验证。

![API网关-灰度中](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-灰度中.png)

**灰度后**：通过灰度流量验证API网关上的API配置符合预期后再迁移。

![API网关-灰度后](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-灰度后.png)



### 自动生成DSL

业务研发人员在实际使用网关管理平台时，我们尽量通过图形化的页面配置来减轻DSL的编写负担。但服务参数转换的DSL配置，仍然需要业务研发人员手工编写。一般来说，生成服务参数DSL的流程是：

- 引入服务的接口包依赖
- 拿到服务参数类定义
- 编写Testcase生成JSON模板
- 填写参数映射规则
- 最后手工录入管理平台，发布API

整个过程非常繁琐，且容易出错。如果需要录入的API多达几十上百个，全部由业务研发人员手工录入的效率是非常低下的。



**解决方案**

那么能不能将服务参数DSL的生成过程给自动化呢？ 答案是可以的，业务RD只需在网关录入API文档信息，然后录入服务的Appkey、服务名、方法名信息，API网关管理端会从最新发布的服务框架控制台获取到服务参数的JSON Schema信息，JSON Schema定义了服务参数的类型和结构信息，管理端可根据这些信息，自动生成服务参数的JSON Mock数据。结合API文档的信息，自动替换参数名相同的Value值。 这套DSL自动生成方案，使用过程中对业务透明、标准化，业务方只需升级最新版本服务框架即可使用，极大提升研发效率，目前受到业务研发人员的广泛好评。

![API网关-自动生成DSL](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-自动生成DSL.png)



### API操作提效

**快速创建API**

API网关的核心能力是建立在API配置的基础上的，但提供强大功能的同时带来了较高的复杂性，不少业务研发人员吐槽API配置太繁琐，学习成本高。快速创建API的功能应运而生，业务研发人员只需要提供少量的信息就可以创建API。快速创建API的功能当前分为4种类型（后端RPC服务API、后端HTTP服务API、SSO CallBack API、Nest API），未来会根据业务应用场景的不同，提供更多的快速创建API类型。



**批量操作**

业务研发人员在API网关上，需要管理非常多的业务分组，每个业务分组，最多可以有200个API配置，多个API可能有很多相同的配置，如组件配置，错误码配置和跨域配置的。每个API对于相同的配置都要配置一遍，操作重复度很高。因此API网关支持批量操作多个API：勾选多个API后，通过【批量操作】功能可一次性完成多个API配置更新，降低业务重复配置的操作成本。



**API导入导出**

API网关提供在不同研发环境相互导入导出API的能力，业务研发人员在线下测试完成后，只需要使用API导入导出功能，即可将配置导出到线上生产环境，避免重复配置。



### 自定义组件

API网关提供了丰富的系统组件完成鉴权、限流、监控能力，能够满足大部分的业务需求。但仍有一些特殊的业务需求，如自定义验签、自定义结果处理等。API网关通过提供加载自定义组件能力，支持业务完成一些自定义逻辑的扩展。下图是自定义组件实现的一个实例。getName中填写自定义组件申请时的名称，invoke方法中实现自定义组件的业务逻辑，如继续执行、进行页面跳转、直接返回结果、抛出异常等。

![API网关-自定义组件](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-自定义组件.png)



### 服务编排

一般情况下，网关上配置的一个API对应后端一个RPC或者HTTP服务。如果调用端有聚合和编排后端服务的需求，那么有多少后端服务，就必须发起多少次HTTP的请求调用。由此就会带来一些问题，调用端的HTTP请求次数过多，效率低，在调用端聚合服务的逻辑过重。

服务编排的需求应运而生，服务编排是对既有服务进行编排调用，同时对获取的数据进行处理。主要应用在数据聚合场景：一次HTTP请求返回的数据需要调用多个或多次服务（RPC或HTTP）才能获取到完整的结果。

通过独立部署的方式提供服务编排能力，API网关与服务编排服务之间通过RPC进行调用。这样可以解耦API网关与服务编排服务，避免因服务编排能力影响集群上的其他服务，同时多一次RPC调用并不会有明显耗时增加。使用上对业务研发人员也是透明的，非常方便，业务研发人员在管理端配置好服务编排的API，通过配置中心同时下发到API网关服务端和服务编排服务上，即可开始使用服务编排能力。整体的交互架构图如下：

![API网关-服务编排](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-服务编排.png)



## 流量治理

### API鉴权

请求安全是API网关非常重要的能力，集成了丰富的安全相关的系统组件，包括有基础的请求签名、SSO单点登录、基于SSO鉴权的UAC/UPM访问控制、用户鉴权Passport、商家鉴权EPassport、商家权益鉴权、反爬等等。业务研发人员只需要简单配置，即可使用。



### 黑白名单

### 流量控制

### 熔断器

### 服务降级

### 流量调度

### 流量Copy

### 流量预热

作者：程序员良许
链接：https://www.zhihu.com/question/309582197/answer/2077037990
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



### 日志

由于所有的请求都是由网关处理的，因此日志也需要相对比较完善，比如接口的耗时、请求方式、请求IP、请求参数、响应参数(注意脱敏)等，另外由于可能涉及到很多微服务，因此需要提供一个统一的traceId方便关联所有的日志，可以将这个traceId置于响应头中，方便排查问题。

### 隔离

比如线程池、http连接池、redis等应用层面的隔离，另外也可以根据业务场景，将核心业务部署带单独的网关集群，与其他非核心业务隔离开。

### 网关管控平台

这块也是非常重要的一环，需要考虑好整个流程的用户体验，比如接入到网关的这个流程，能不能尽量简化、智能，比如如果是dubbo接口，我们可以通过到git仓库中获取源码、解析对应的类、方法，从而实现自动填充，尽量帮用户减少操作；另外接口一般是从测试->预发->线上，如果每次都要填写一遍表单会非常麻烦，我们能不能自动把这个事情做掉，另外如果网关部署到了多个可用区、甚至不同的国家，那这个时候，我们还需要接口数据同步功能，不然用户需要到每个后台都操作一遍，非常麻烦。
这块个人的建议是直接参考阿里云、aws等提供的网关服务即可，功能非常全面。

### 其他

其他还有些需要考虑到的点，比如接口mock，文档生成、sdk代码生成、错误码统一、服务治理相关的等，这里就不累述了。

## 总结

目前的网关还是中心化的架构，所有的请求都需要走一次网关，因此当大促或者流量突增时，网关可能会成为性能的瓶颈，而且当网关接入的大量接口的时候，做好流量评估也不是一项容易的工作，每次大促前都需要跟业务方一起针对接口做压测，评估出大致的容量，并对网关进行扩容，而且网关是所有流量的入口，所有的请求都是由网关处理，要想准确的评估出容量很复杂。可以参考目前比较流行的ServiceMesh，采用去中心化的方案，将网关的逻辑下沉到sidecar中，
sidecar和应用部署到同一个节点，并接管应用流入、流出的流量，这样大促时，只需要对相关的业务压测，并针对性扩容即可，另外升级也会更平滑，中心化的网关，即使灰度发布，但是理论上所有业务方的流量都会流入到新版本的网关，如果出了问题，会影响到所有的业务，但这种去中心化的方式，可以先针对非核心业务升级，观察一段时间没问题后，再全量推上线。另外ServiceMesh的方案，对于多语言支持也更友好。

### 集群隔离

API网关按业务线维度进行集群隔离，也支持重要业务独立部署。如下图所示：

![API网关-集群隔离](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-集群隔离.png)



### 请求隔离

服务节点维度，API网关支持请求的快慢线程池隔离。快慢线程池隔离主要用于一些使用了同步阻塞组件的API，例如SSO鉴权、自定义鉴权等，可能导致长时间阻塞共享业务线程池。快慢隔离的原理是统计API请求的处理时间，将请求处理耗时较长，超过容忍阈值的API请求隔离到慢线程池，避免影响其他正常API的调用。除此之外，也支持业务研发人员配置自定义线程池进行隔离。具体的线程隔离模型如下图所示：

![API网关-请求隔离](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-请求隔离.png)



### 灰度发布

API网关作为请求入口，往往肩负着请求流量灰度验证的重任。

**灰度场景**

在灰度能力上，支持灰度API自身逻辑，也支持灰度下游服务，也可以同时灰度API自身逻辑和下游服务。如下图所示：

![API网关-灰度场景](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-灰度场景.png)

灰度API自身逻辑时，通过将流量分流到不同的API版本实现灰度能力；灰度下游服务时，通过给流量打标，分流到指定的下游灰度单元中。



**灰度策略**

支持丰富的灰度策略，可以按照比例数灰度，也可以按照特定条件灰度。



## 监控告警

### 立体化监控

API网关提供360度的立体化监控，从业务指标、机器指标、JVM指标提供7x24小时的专业守护，如下表：

|      | 监控模块        | 主要功能                                                     |
| :--- | :-------------- | :----------------------------------------------------------- |
| 1    | 统一监控Raptor  | 实时上报请求调用信息、系统指标，负责应用层（JVM）监控、系统层（CPU、IO、网络）监控 |
| 2    | 链路追踪Mtrace  | 负责全链路参数透传、全链路追踪监控                           |
| 3    | 日志监控Logscan | 监控本地日志异常关键字：如5xx状态码、空指针异常等            |
| 4    | 远程日志中心    | API请求日志、Debug日志、组件日志等可上报远程日志中心         |
| 5    | 健康检查Scanner | 对网关节点进行心跳检测和API状态检测，及时发现异常节点和异常API |



### 多维度告警

有了全面的监控体系，自然少不了配套的告警机制，主要的告警能力包括：

|      | 告警类型         | 触发时机                                            |
| :--- | :--------------- | :-------------------------------------------------- |
| 1    | 限流告警         | API请求达到限流规则阈值触发限流告警                 |
| 2    | 请求失败告警     | 鉴权失败、请求超时、后端服务异常等触发请求失败告警  |
| 3    | 组件异常告警     | 自定义组件处理耗时长、失败率高告警                  |
| 4    | API异常告警      | API发布失败、API检查异常时触发API异常告警           |
| 5    | 健康检查失败告警 | API心跳检查失败、网关节点不通时触发健康检查失败告警 |



## 关键设计

### 异步外调

![API网关-异步外调](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-异步外调.png)

基于Netty实现异步外调主要有两种方式可以实现：

- **方式一：建立全局Map，上线文传递（不参与远程传输）requestId，响应时使用requestId进行映射上游信息**
- **方式二：直接将上游信息包装成Context进行上线文传递（不参与远程传输）**

方式一需要独立维护一个全局映射表，同时需要考虑请求超时和丢失的情况，否则会出现内存不断增长问题。



### 外调链接池化

![API网关-外调链接池化](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-外调链接池化.png)

使用Netty实现API网关外调微服务时，因建立连接需要极度消耗资源，所以需要考虑将外调的链接进行池化管理，设计时需要注意以下几点：

- **初始化适当连接（过多过少都不适合）**
- **考虑连接能随流量增减而进行自动扩缩容**
- **取出的连接需要检查是否可用**
- **连接需要考虑双向心跳探测**



### 释放连接

http的链接是独占的，所以在释放的时候要特别小心，一定要等服务端响应完了才能释放，还有就是链接关闭的处理也要小心，总结如下几点：

- Connection:close
- 空闲超时，关闭链接
- 读超时关闭链接
- 写超时，关闭链接
- Fin，Reset



- **写超时**：writeAndFlush包含Netty的encode时间和从队列里把请求发出去即flush的时间。因此后端超时开始需要在真正flush成功后开始计时，这样才最接近服务端超时时间（还有网络往返时间和内核协议栈处理时间）



### 对象池化设计

针对高并发系统，频繁创建对象不仅有分配内存开销，还对gc会造成压力。因此在实现时，会对频繁使用的对象（如线程池的任务task，StringBuffer等）进行重写，减少频繁的申请内存的开销。



### 上下文切换

![API网关-上下文切换](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/API网关-上下文切换.png)

整个网关没有涉及到IO操作，但在IO编解码和业务逻辑都用了异步，是有两个原因

- 防止开发写的代码有阻塞
- 业务逻辑打日志可能会比较多

在突发的情况下，但是我们在push线程时，支持用Netty的IO线程替代，这里做的工作比较少，这里由异步修改为同步后（通过修改配置调整），CPU的上下文切换减少20%，进而提高了整体的吞吐量，就是不能为了异步而异步，Zuul2的设计类似。



### 监控告警

**协议层**

- **攻击性请求**。只发头，不发/发部分body，采样落盘，还原现场，并报警
- **Line or Head or Body过大的请求**。采样落盘，还原现场，并报警



**应用层**

- **耗时监控**。有慢请求，超时请求，以及tp99，tp999等
- **QPS监控和报警**
- **带宽监控和报警**。支持对请求和响应的行、头、body单独监控
- **响应码监控**。特别是400和404
- **链接监控**。对接入端的链接，以及和后端服务的链接，后端服务链接上待发送字节大小也都做了监控
- **失败请求监控**
- **流量抖动报警**。流量抖动要么是出了问题，要么就是出问题的前兆



## 解决方案

![APIGateway产品](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/APIGateway产品.png)

### Shepherd API网关

![ShepherdAPI网关](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/ShepherdAPI网关.png)



### Mashape Kong

**访问地址**：https://github.com/Kong/kong

![Kong](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/kong.png)



### Soul

**访问地址**：https://github.com/Dromara/soul

![Soul](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Soul.png)



### Apiman

**访问地址**：https://apiman.gitbooks.io/apiman-user-guide/user-guide/gateway/policies.html

![apiman](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/apiman.png)



### Gravitee

**访问地址**：https://docs.gravitee.io/apim_policies_latency.html

![Gravitee](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Gravitee.png)



### Tyk

**访问地址**：https://tyk.io/docs



### Traefik

**访问地址**：https://traefik.cn

Træfɪk 是一个为了让部署微服务更加便捷而诞生的现代HTTP反向代理、负载均衡工具。 它支持多种后台 (Docker, Swarm, Kubernetes, Marathon, Mesos, Consul, Etcd, Zookeeper, BoltDB, Rest API, file…) 来自动化、动态的应用它的配置文件设置。

![traefik](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/traefik.png)

**功能特性**

- 它非常快
- 无需安装其他依赖，通过Go语言编写的单一可执行文件
- 支持 Rest API
- 多种后台支持：Docker, Swarm, Kubernetes, Marathon, Mesos, Consul, Etcd, 并且还会更多
- 后台监控, 可以监听后台变化进而自动化应用新的配置文件设置
- 配置文件热更新。无需重启进程
- 正常结束http连接
- 后端断路器
- 轮询，rebalancer 负载均衡
- Rest Metrics
- 支持最小化官方docker 镜像
- 后台支持SSL
- 前台支持SSL（包括SNI）
- 清爽的AngularJS前端页面
- 支持Websocket
- 支持HTTP/2
- 网络错误重试
- 支持Let’s Encrypt (自动更新HTTPS证书)
- 高可用集群模式



### 小豹API网关

**访问地址**：http://www.xbgateway.com

小豹API网关（企业级API网关），统一解决：认证、鉴权、安全、流量管控、缓存、服务路由，协议转换、服务编排、熔断、灰度发布、监控报警等。

![小豹API网关架构](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/小豹API网关架构.png)



### Others

- Orange：http://orange.sumory.com
- gateway：https://github.com/fagongzi/gateway

## API网关解决方案

私有云开源解决方案如下：

- Kong kong是基于Nginx+Lua进行二次开发的方案， [https://konghq.com/](https://link.zhihu.com/?target=https%3A//konghq.com/)
- Netflix Zuul，zuul是spring cloud的一个推荐组件，[https://github.com/Netflix/zuul](https://link.zhihu.com/?target=https%3A//github.com/Netflix/zuul)
- orange,这个开源程序是国人开发的， [http://orange.sumory.com/](https://link.zhihu.com/?target=http%3A//orange.sumory.com/)
- [小豹API网关](https://link.zhihu.com/?target=http%3A//www.xbgateway.com/)，这个是作者参与研发的API网关，国内少有的基于java的api网关，[小豹API网关](https://link.zhihu.com/?target=http%3A//www.xbgateway.com)

公有云解决方案：

- Amazon API Gateway，[https://aws.amazon.com/cn/api-gateway/](https://link.zhihu.com/?target=https%3A//aws.amazon.com/cn/api-gateway/)
- 阿里云API网关，[https://www.aliyun.com/product/apigateway/](https://link.zhihu.com/?target=https%3A//www.aliyun.com/product/apigateway/)
- 腾讯云API网关， [https://cloud.tencent.com/product/apigateway](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/product/apigateway)

自开发解决方案：

- 基于Nginx+Lua+ OpenResty的方案，可以看到Kong,orange都是基于这个方案
- 基于Netty、非阻塞IO模型。通过网上搜索可以看到国内的宜人贷等一些公司是基于这种方案，是一种成熟的方案。
- 基于Node.js的方案。这种方案是应用了Node.js天生的非阻塞的特性。
- 基于java Servlet的方案。zuul基于的就是这种方案，这种方案的效率不高，这也是zuul总是被诟病的原因。

微服务架构需要应用网关，虽然的Zuul和spring-cloud-gateway都是可以做到的很多，但是都要进行编码。如果是一个团队协作或者对聚合的效率有要求的话，需要采用异步的编程方式的提升功能。你还得考虑得熔断，fallback等等，以及其他接口的转发请求，这些加起来编码也不少。可以考虑一下fizz-gateway。其能够处理上述这个聚合要求，也提供后端界面的编辑，甚至如果你需要一些定制开发都可以使用javascript或者groove进行开发。

可以关注 Apache APISIX 这个 API 网关项目，我们在持续积极地邀请贡献者参与其中

我上班的公司选的是RestCloud的API网关，我听开发的人员说还不错

Nginx

Nginx 是异步框架的网页服务器，也可以用作反向代理、负载平衡器和 HTTP 缓存。

该软件由伊戈尔·赛索耶夫创建并于 2004 年首次公开发布。2011 年成立同名公司以提供支持。2019 年 3 月 11 日，Nginx 公司被 F5 Networks 以 6.7 亿美元收购。

Nginx 有以下的特点：

由 C 编写，占用的资源和内存低，性能高。单进程多线程，当启动 Nginx 服务器，会生成一个 master 进程，master 进程会 fork 出多个 worker 进程，由 worker 线程处理客户端的请求。支持反向代理，支持 7 层负载均衡（拓展负载均衡的好处）。高并发，Nginx 是异步非阻塞型处理请求，采用的 epollandqueue 模式。处理静态文件速度快。高度模块化，配置简单。社区活跃，各种高性能模块出品迅速。

![img](https://pics5.baidu.com/feed/9358d109b3de9c8225e19f8beafd5a0218d843b7.jpeg?token=89faafc02ae4fcec6613865efe4a6fc7)

如上图所示，Nginx 主要由 Master，Worker 和 Proxy Cache 三个部分组成。

**Master 主控：**NGINX 遵循主从架构。它将根据客户的要求为 Worker 分配工作。

将工作分配给 Worker 后，Master 将寻找客户的下一个请求，因为它不会等待 Worker 的响应。一旦响应来自 Worker，Master 就会将响应发送给客户端。

**Worker 工作单元：**Worker 是 NGINX 架构中的 Slave。每个工作单元可以单线程方式一次处理 1000 个以上的请求。

一旦处理完成，响应将被发送到主服务器。单线程将通过在相同的内存空间而不是不同的内存空间上工作来节省 RAM 和 ROM 的大小。多线程将在不同的内存空间上工作。

**Cache 缓存：**Nginx 缓存用于通过从缓存而不是从服务器获取来非常快速地呈现页面。在第一个页面请求时，页面将被存储在高速缓存中。

为了实现 API 的路由转发，需要只需要对 Nginx 作出如下的配置：

server {listen80 default_server; location /goapi {rewrite ^/goapi(.*)$1break;proxy_pass http://goapi:8080; }location /nodeapi {rewrite ^/nodeapi(.*)$1break;proxy_pass http://nodeapi:8080; }location /flaskapi {rewrite ^/flaskapi(.*)$1break;proxy_pass http://flaskapi:8080; }location /springapi {rewrite ^/springapi(.*)$1break;proxy_pass http://springapi:8080; }}

我们基于不同的服务 goapi，nodeapi，flaskapi 和 springapi，分别配置一条路由，在转发之前，需要利用 rewrite 来去掉服务名，并发送给对应的服务。

使用容器把 Nginx 和后端的四个服务部署在同一个网络下，通过网关连接路由转发的。

Nginx 的部署如下：

version: "3.7"services: web: container_name: nginx image: nginx volumes: - ./templates:/etc/nginx/templates - ./conf/default.conf:/etc/nginx/conf.d/default.conf ports: - "8080:80" environment: - NGINX_HOST=localhost - NGINX_PORT=80 deploy: resources: limits: cpus: '1' memory: 256M reservations: memory: 256M

K6 通过 Nginx 网关的测试结果如下：

![img](https://pics6.baidu.com/feed/b8389b504fc2d5629fdfe2586c6d4ae777c66c30.jpeg?token=775da4b20d91d8cc412881d7b81f371b)

每秒处理的请求数量是 1093，和不通过网关转发相比非常接近。

从功能上看，Nginx 可以满足用户对于 API 网关的大部分需求，可以通过配置和插件的方式来支持不同的功能，性能非常优秀。

缺点是没有管理的 UI 和管理 API，大部分的工作都需要手工配置 config 文件的方式来进行。商业版本的功能会更加完善。

Kong

Kong 是基于 NGINX 和 OpenResty 的开源 API 网关。

Kong 的总体基础结构由三个主要部分组成：NGINX 提供协议实现和工作进程管理，OpenResty 提供 Lua 集成并挂钩到 NGINX 的请求处理阶段。

而 Kong 本身利用这些挂钩来路由和转换请求。数据库支持 Cassandra 或 Postgres 存储所有配置。

![img](https://pics7.baidu.com/feed/0b55b319ebc4b7451db2bca74880c41f8b8215e5.jpeg?token=cbf536c1bbe38a034c15773fc2104019)

Kong 附带各种插件，提供访问控制，安全性，缓存和文档等功能。它还允许使用 Lua 语言编写和使用自定义插件。

Kong 也可以部署为 Kubernetes Ingress 并支持 GRPC 和 WebSockets 代理。

NGINX 提供了强大的 HTTP 服务器基础结构。它处理 HTTP 请求处理，TLS 加密，请求日志记录和操作系统资源分配（例如，侦听和管理客户端连接以及产生新进程）。

NGINX 具有一个声明性配置文件，该文件位于其主机操作系统的文件系统中。

虽然仅通过 NGINX 配置就可以实现某些 Kong 功能（例如，基于请求的 URL 确定上游请求路由），但修改该配置需要一定级别的操作系统访问权限，以编辑配置文件并要求 NGINX 重新加载它们。

而 Kong 允许用户执行以下操作：通过 RESTful HTTP API 更新配置。Kong 的 NGINX 配置是相当基本的：除了配置标准标头，侦听端口和日志路径外，大多数配置都委托给 OpenResty。

在某些情况下，在 Kong 的旁边添加自己的 NGINX 配置非常有用，例如在 API 网关旁边提供静态网站。在这种情况下，您可以修改 Kong 使用的配置模板。

NGINX 处理的请求经过一系列阶段。NGINX 的许多功能（例如，使用 C 语言编写的模块）都提供了进入这些阶段的功能（例如，使用 gzip 压缩的功能）。

虽然可以编写自己的模块，但是每次添加或更新模块时都必须重新编译 NGINX。为了简化添加新功能的过程，Kong 使用了 OpenResty。

OpenResty 是一个软件套件，捆绑了 NGINX，一组模块，LuaJIT 和一组 Lua 库。

其中最主要的是 ngx_http_lua_module一个NGINX 模块，该模块嵌入 Lua 并为大多数 NGINX 请求阶段提供 Lua 等效项。

这有效地允许在 Lua 中开发 NGINX 模块，同时保持高性能（LuaJIT 相当快），并且 Kong 用它来提供其核心配置管理和插件管理基础结构。

Kong 通过其插件体系结构提供了一个框架，可以挂接到上述请求阶段。从上面的示例开始，Key Auth 和 ACL 插件都控制客户端（也称为使用者）是否应该能够发出请求。

每个插件都在其处理程序中定义了自己的访问函数，并且该函数针对通过给定路由或服务启用的每个插件执行 kong.access()。

执行顺序由优先级值决定，如果 Key Auth 的优先级为 1003，ACL 的优先级为 950，则 Kong 将首先执行 Key Auth 的访问功能，如果它不放弃请求，则将执行 ACL，然后再通过将该 ACL 传递给上游 proxy_pass。

由于 Kong 的请求路由和处理配置是通过其 admin API 控制的，因此可以在不编辑底层 NGINX 配置的情况下即时添加和删除插件配置。

因为 Kong 本质上提供了一种在 API 中注入位置块（通过 API 定义）和配置的方法。它们通过将插件，证书等分配给这些 API。

我们使用以下的配置部署 Kong 到容器中（省略四个微服务的部署）：

version: '3.7'volumes: kong_data: {}networks: kong-net: external: falseservices: kong: image: "${KONG_DOCKER_TAG:-kong:latest}" user: "${KONG_USER:-kong}" depends_on: - db environment: KONG_ADMIN_ACCESS_LOG: /dev/stdout KONG_ADMIN_ERROR_LOG: /dev/stderr KONG_ADMIN_LISTEN: '0.0.0.0:8001' KONG_CASSANDRA_CONTACT_POINTS: db KONG_DATABASE: postgres KONG_PG_DATABASE: ${KONG_PG_DATABASE:-kong} KONG_PG_HOST: db KONG_PG_USER: ${KONG_PG_USER:-kong} KONG_PROXY_ACCESS_LOG: /dev/stdout KONG_PROXY_ERROR_LOG: /dev/stderr KONG_PG_PASSWORD_FILE: /run/secrets/kong_postgres_password secrets: - kong_postgres_password networks: - kong-net ports: - "8080:8000/tcp" - "127.0.0.1:8001:8001/tcp" - "8443:8443/tcp" - "127.0.0.1:8444:8444/tcp" healthcheck:test: ["CMD", "kong", "health"] interval: 10s timeout: 10s retries: 10 restart: on-failure deploy: restart_policy: condition: on-failure db: image: postgres:9.5 environment: POSTGRES_DB: ${KONG_PG_DATABASE:-kong} POSTGRES_USER: ${KONG_PG_USER:-kong} POSTGRES_PASSWORD_FILE: /run/secrets/kong_postgres_password secrets: - kong_postgres_password healthcheck:test: ["CMD", "pg_isready", "-U", "${KONG_PG_USER:-kong}"] interval: 30s timeout: 30s retries: 3 restart: on-failure deploy: restart_policy: condition: on-failure stdin_open: true tty: true networks: - kong-net volumes: - kong_data:/var/lib/postgresql/datasecrets: kong_postgres_password: file: ./POSTGRES_PASSWORD

数据库选择了 PostgreSQL，开源版本没有 Dashboard，我们使用 RestAPI 创建所有的网关路由：

curl -i -X POST http://localhost:8001/services \ --data name=goapi \ --data url='http://goapi:8080' curl -i -X POST http://localhost:8001/services/goapi/routes \ --data'paths[]=/goapi' \ --data name=goapi

需要先创建一个 service，然后在该 service 下创建一条路由。

使用 K6 压力测试的结果如下：

![img](https://pics5.baidu.com/feed/8326cffc1e178a82700e9371707fa985a877e8b3.jpeg?token=b27bc400c972a22ad541274128f42db9)

每秒请求数 705 要明显弱于 Nginx，所以所有的功能都是有成本的。

APISIX

Apache APISIX 是一个动态、实时、高性能的 API 网关， 提供负载均衡、动态上游、灰度发布、服务熔断、身份认证、可观测性等丰富的流量管理功能。

APISIX 于 2019 年 4 月由中国的支流科技创建，于 6 月开源，并于同年 10 月进入 Apache 孵化器。

支流科技对应的商业化产品的名字叫 API7 。APISIX 旨在处理大量请求，并具有较低的二次开发门槛。

APISIX 的主要功能和特点有：

云原生设计，轻巧且易于容器化。集成了统计和监视组件，例如 Prometheus，Apache Skywalking 和 Zipkin。支持 gRPC，Dubbo，WebSocket，MQTT 等代理协议，以及从 HTTP 到 gRPC 的协议转码，以适应各种情况。担当 OpenID 依赖方的角色，与 Auth0，Okta 和其他身份验证提供程序的服务连接。通过在运行时动态执行用户功能来支持无服务器，从而使网关的边缘节点更加灵活。支持插件热加载。不锁定用户，支持混合云部署架构。网关节点无状态，可以灵活扩展。

从这个角度来看，API 网关可以替代 Nginx 来处理南北流量，也可以扮演 Istio 控制平面和 Envoy 数据平面的角色来处理东西向流量。

APISIX 的架构如下图所示：

![img](https://pics2.baidu.com/feed/a8ec8a13632762d054eab9502790d2f2533dc68a.jpeg?token=830d081ac0b71a40a885db05260ffdc4)

APISIX 包含一个数据平面，用于动态控制请求流量；一个用于存储和同步网关数据配置的控制平面，一个用于协调插件的 AI 平面，以及对请求流量的实时分析和处理。

它构建在 Nginx 反向代理服务器和键值存储 etcd 的之上，以提供轻量级的网关。

它主要用 Lua 编写，Lua 是类似于 Python 的编程语言。它使用 Radix 树进行路由，并使用前缀树进行 IP 匹配。

使用 etcd 而不是关系数据库来存储配置可以使它更接近云原生，但是即使在任何服务器宕机的情况下，也可以确保整个网关系统的可用性。

所有组件都是作为插件编写的，因此其模块化设计意味着功能开发人员只需要关心自己的项目即可。

内置的插件包括流控和速度限制，身份认证，请求重写，URI 重定向，开放式跟踪和无服务器。

APISIX 支持 OpenResty 和 Tengine 运行环境，并且可以在 Kubernetes 的裸机上运行。它同时支持 X86 和 ARM64。

我们同样使用 Docker Compose 来部署 APISIX：

version: "3.7"services: apisix-dashboard: image: apache/apisix-dashboard:2.4 restart: always volumes: - ./dashboard_conf/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml ports: - "9000:9000" networks: apisix: ipv4_address: 172.18.5.18 apisix: image: apache/apisix:2.3-alpine restart: always volumes: - ./apisix_log:/usr/local/apisix/logs - ./apisix_conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro depends_on: - etcd ##network_mode: host ports: - "8080:9080/tcp" - "9443:9443/tcp" networks: apisix: ipv4_address: 172.18.5.11 deploy: resources: limits: cpus: '1' memory: 256M reservations: memory: 256M etcd: image: bitnami/etcd:3.4.9 user: root restart: always volumes: - ./etcd_data:/etcd_data environment: ETCD_DATA_DIR: /etcd_data ETCD_ENABLE_V2: "true" ALLOW_NONE_AUTHENTICATION: "yes" ETCD_ADVERTISE_CLIENT_URLS: "http://0.0.0.0:2379" ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379" ports: - "2379:2379/tcp" networks: apisix: ipv4_address: 172.18.5.10networks: apisix: driver: bridge ipam:config: - subnet: 172.18.0.0/16

开源的 APISIX 支持 Dashboard 的方式来管理路由，而不是像 KONG 把仪表盘功能限制在商业版本中。

但是 APISIX 的仪表盘不支持对路由 URI 进行改写，所以我们只好使用 RestAPI 来创建路由。

创建一个服务的路由的命令如下：

curl --location --request PUT 'http://127.0.0.1:8080/apisix/admin/routes/1' \--header 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \--header 'Content-Type: text/plain' \--data-raw '{"uri": "/goapi/*","plugins": {"proxy-rewrite": {"regex_uri": ["^/goapi(.*)$","$1"] } },"upstream": {"type": "roundrobin","nodes": {"goapi:8080": 1 } }}'

使用 K6 压力测试的结果如下：

![img](https://pics0.baidu.com/feed/0823dd54564e925829f3ff5e1afe0b50cebf4ec4.jpeg?token=c6d5e59506a94dd0bd29519612d4cf5b)

APISix 取得了 1155 的好成绩，表现出接近不经过网关的性能，可能缓存起到了很好的效果。

Tyk

Tyk 是一款基于 Golang 和 Redis 构建的开源 API 网关。它于 2014 年创建，比 AWS 的 API 网关即服务功能早。Tyk 用 Golang 编写，并使用 Golang 自己的 HTTP 服务器。

Tyk 支持不同的运行方式：云，混合（在自己的基础架构中为 GW）和本地。

![img](https://pics0.baidu.com/feed/8ad4b31c8701a18b3fc3dc411953dd002838fe1e.jpeg?token=0e14fe4335e7bb88ecdf4102d1fdee85)

Tyk 由 3 个组件组成：

**网关：**处理所有应用流量的代理。**仪表板：**可以从中管理 Tyk，显示指标和组织 API 的界面。**Pump：**负责持久保存指标数据，并将其导出到 MongoDB（内置），ElasticSearch 或 InfluxDB 等。

我们同样使用 Docker Compose 来创建 Tyk 网关来进行功能验证。

version: '3.7'services: tyk-gateway: image: tykio/tyk-gateway:v3.1.1 ports: - 8080:8080 volumes: - ./tyk.standalone.conf:/opt/tyk-gateway/tyk.conf - ./apps:/opt/tyk-gateway/apps - ./middleware:/opt/tyk-gateway/middleware - ./certs:/opt/tyk-gateway/certs environment: - TYK_GW_SECRET=foo depends_on: - tyk-redis tyk-redis: image: redis:5.0-alpine ports: - 6379:6379

Tyk 的 Dashboard 也是属于商业版本的范畴，所我们又一次需要借助 API 来创建路由，Tyk 是通过 app 的概念来创建和管理路由的，你也可以直接写 app 文件。

curl --location --request POST 'http://localhost:8080/tyk/apis/' \--header 'x-tyk-authorization: foo' \--header 'Content-Type: application/json' \--data-raw '{"name": "GO API","slug": "go-api","api_id": "goapi","org_id": "goapi","use_keyless": true,"auth": {"auth_header_name": "Authorization" },"definition": {"location": "header","key": "x-api-version" },"version_data": {"not_versioned": true,"versions": {"Default": {"name": "Default","use_extended_paths": true } } },"proxy": {"listen_path": "/goapi/","target_url": "http://host.docker.internal:18000/","strip_listen_path": true },"active": true}'

使用 K6 压力测试的结果如下：

![img](https://pics0.baidu.com/feed/e824b899a9014c08ec422e358507d8007af4f4ae.jpeg?token=14083856b22cbefefc1d4f2d29e5997b)

Tyk 的结果在 400-600 左右，性能上和 KONG 接近。

Zuul

Zuul 是 Netflix 开源的基于 Java 的 API 网关组件。

![img](https://pics1.baidu.com/feed/a8014c086e061d9589b93e01fc88d0d963d9ca95.jpeg?token=17ec0d4de3c3f1975150227088b596c8)

Zuul 包含多个组件：

**zuul-core：**该库包含编译和执行过滤器的核心功能。**zuul-simple-webapp：**该 Webapp 展示了一个简单的示例，说明如何使用 zuul-core 构建应用程序。**zuul-netflix：**将其他 NetflixOSS 组件添加到 Zuul 的库，例如，使用 Ribbon 路由请求。**zuul-netflix-webapp：**将 zuul-core 和 zuul-netflix 打包到一个易于使用的程序包中的 webapp。

Zuul 提供了灵活性和弹性，部分是通过利用其他 Netflix OSS 组件进行的：

Hystrix 用于流控。包装对始发地的呼叫，这使我们可以在发生问题时丢弃流量并确定流量的优先级。Ribbon 是来自 Zuul 的所有出站请求的客户，它提供有关网络性能和错误的详细信息，并处理软件负载平衡以实现均匀的负载分配。Turbine 实时汇总细粒度的指标，以便我们可以快速观察问题并做出反应。Archaius 处理配置并提供动态更改属性的能力。

Zuul 的核心是一系列过滤器，它们能够在路由 HTTP 请求和响应期间执行一系列操作。

以下是 Zuul 过滤器的主要特征：

**类型：**通常定义路由流程中应用过滤器的阶段。（尽管它可以是任何自定义字符串）**执行顺序：**在类型中应用，定义跨多个过滤器的执行顺序。**准则：**执行过滤器所需的条件。**动作：**如果符合条件，则要执行的动作。classDeviceDelayFilterextendsZuulFilter{ def static Random rand = new Random()@OverrideString filterType(){return'pre' }@OverrideintfilterOrder(){return5 }@OverridebooleanshouldFilter(){return RequestContext.getRequest(). getParameter("deviceType")?equals("BrokenDevice"):false }@OverrideObject run(){ sleep(rand.nextInt(20000)) // Sleep for a random number of// seconds between [0-20] }}

Zuul 提供了一个框架来动态读取，编译和运行这些过滤器。过滤器不直接相互通信。

而是通过每个请求唯一的 RequestContext 共享状态。过滤器使用 Groovy 编写。

![img](https://pics6.baidu.com/feed/b2de9c82d158ccbfd52f0e0191a46636b03541fb.jpeg?token=e567f28c7d6b254793b24fb7e4063217)

有几种与请求的典型生命周期相对应的标准过滤器类型：

Pre 过滤器在路由到原点之前执行。示例包括请求身份验证，选择原始服务器以及记录调试信息。Route 路由过滤器处理将请求路由到源。这是使用 Apache HttpClient 或 Netflix Ribbon 构建和发送原始 HTTP 请求的地方。在将请求路由到源之后，将执行 Post 过滤器。示例包括将标准 HTTP 标头添加到响应，收集统计信息和指标以及将响应从源流传输到客户端。在其他阶段之一发生错误时，将执行 Error 过滤器。

Spring Cloud 创建了一个嵌入式 Zuul 代理，以简化一个非常常见的用例的开发，在该用例中，UI 应用程序希望代理对一个或多个后端服务的调用。

此功能对于用户界面代理所需的后端服务很有用，从而避免了为所有后端独立管理 CORS 和身份验证问题的需求 。

要启用它，请使用 @EnableZuulProxy 注解一个 Spring Boot 主类，这会将本地调用转发到适当的服务。

Zuul 是 Java 的一个库，他并不是一款开箱即用的 API 网关，所以需要用 Zuul 开发一个应用来对其功能进行测试。

对应的 Java 的 POM 如下：

<projectxmlns="http://maven.apache.org/POM/4.0.0"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"><modelVersion>4.0.0</modelVersion><groupId>naughtytao.apigateway</groupId><artifactId>demo</artifactId><version>0.0.1-SNAPSHOT</version><parent><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-parent</artifactId><version>1.4.7.RELEASE</version><relativePath /><!-- lookup parent from repository --></parent><properties><project.build.sourceEncoding>UTF-8</project.build.sourceEncoding><project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding><java.version>1.8</java.version><!-- Dependencies --><spring-cloud.version>Camden.SR7</spring-cloud.version></properties><dependencyManagement><dependencies><dependency><groupId>org.springframework.cloud</groupId><artifactId>spring-cloud-dependencies</artifactId><version>${spring-cloud.version}</version><type>pom</type><scope>import</scope></dependency></dependencies></dependencyManagement><dependencies><dependency><groupId>org.springframework.cloud</groupId><artifactId>spring-cloud-starter-zuul</artifactId></dependency><dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-actuator</artifactId><exclusions><exclusion><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-logging</artifactId></exclusion></exclusions></dependency><dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-log4j2</artifactId></dependency><!-- enable authentication if security is included --><!-- <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-security</artifactId> </dependency> --><dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId></dependency><!-- API, java.xml.bind module --><dependency><groupId>jakarta.xml.bind</groupId><artifactId>jakarta.xml.bind-api</artifactId><version>2.3.2</version></dependency><!-- Runtime, com.sun.xml.bind module --><dependency><groupId>org.glassfish.jaxb</groupId><artifactId>jaxb-runtime</artifactId><version>2.3.2</version></dependency><dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-test</artifactId><scope>test</scope></dependency><dependency><groupId>org.junit.jupiter</groupId><artifactId>junit-jupiter-api</artifactId><version>5.0.0-M5</version><scope>test</scope></dependency></dependencies><build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin><plugin><groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId><version>3.3</version><configuration><source>1.8</source><target>1.8</target></configuration></plugin></plugins></build></project>

主要应用代码如下：

package naughtytao.apigateway.demo;import org.springframework.boot.SpringApplication;import org.springframework.boot.autoconfigure.EnableAutoConfiguration;import org.springframework.boot.autoconfigure.SpringBootApplication;import org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration;import org.springframework.cloud.netflix.zuul.EnableZuulProxy;import org.springframework.context.annotation.ComponentScan;import org.springframework.context.annotation.Bean;import naughtytao.apigateway.demo.filters.ErrorFilter;import naughtytao.apigateway.demo.filters.PostFilter;import naughtytao.apigateway.demo.filters.PreFilter;import naughtytao.apigateway.demo.filters.RouteFilter;@SpringBootApplication@EnableAutoConfiguration(exclude = { RabbitAutoConfiguration.class })@EnableZuulProxy@ComponentScan("naughtytao.apigateway.demo")publicclassDemoApplication{public static void main(String[] args) { SpringApplication.run(DemoApplication.class, args); }}

Docker 构建文件如下：

FROM maven:3.6.3-openjdk-11WORKDIR /usr/src/appCOPY src ./srcCOPY pom.xml ./RUN mvn -f ./pom.xml clean package -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=trueEXPOSE 8080ENTRYPOINT ["java","-jar","/usr/src/app/target/demo-0.0.1-SNAPSHOT.jar"]

路由的配置写在 application.properties 中：

\#Zuul routes.zuul.routes.goapi.url=http://goapi:8080zuul.routes.nodeapi.url=http://nodeapi:8080zuul.routes.flaskapi.url=http://flaskapi:8080zuul.routes.springapi.url=http://springapi:8080ribbon.eureka.enabled=falseserver.port=8080

我们同样使用 Docker Compose 运行 Zuul 的网关来进行验证：

version: '3.7'services: gateway: image: naughtytao/zuulgateway:0.1 ports: - 8080:8080 volumes: - ./config/application.properties:/usr/src/app/config/application.properties deploy: resources: limits: cpus: '1' memory: 256M reservations: memory: 256M

使用 K6 压力测试的结果如下：

![img](https://pics4.baidu.com/feed/d0c8a786c9177f3ec94d67bdf6b3e1cf9e3d568e.jpeg?token=9adf4fed6c3c18e3487b7c2daf350774)

在相同的配置条件下（单核，256M），Zuul 的压测结果要明显差于其它几个，只有 200 左右。

![img](https://pics6.baidu.com/feed/b17eca8065380cd7d40355da2938773c59828169.jpeg?token=adbcf6341b8b93be82a278597b8637a8)

在分配更多资源的情况下，4 核 2G，Zuul 的性能提升到 600-800，所以 Zuul 对于资源的需求还是比较明显的。

另外需要提及的是，我们使用的是 Zuul1，Netflix 已经推出了 Zuul2。Zuul2 对架构做出了较大的改进。

Zuul1 本质上就是一个同步 Servlet，采用多线程阻塞模型。Zuul2 的基于 Netty 实现了异步非阻塞编程模型。

同步的方式，比较容易调试，但是多线程本身需要消耗 CPU 和内存资源，所以它的性能要差一些。

而采用非阻塞模式的 Zuul，因为线程开销小，所支持的链接数量要更多，也更节省资源。

Gravitee

Gravitee 是 Gravitee.io 开源的，基于 Java 的，简单易用，性能高，且具成本效益的开源 API 平台，可帮助组织保护，发布和分析您的 API。

![img](https://pics3.baidu.com/feed/3ac79f3df8dcd1007f8e501ff5f79d18b9122f07.jpeg?token=0635562d2943febc1d32a8945933f231)

Gravitee 可以通过设计工作室和路径的两种方式来创建和管理 API：

![img](https://pics5.baidu.com/feed/a6efce1b9d16fdfae0c52ce433f3565c95ee7b90.jpeg?token=4a0c37e023486a5a450242b6b8254a6f)

Gravity 提供网关，API 门户和 API 管理，其中网关和管理 API 部分是开源的，门户需要注册许可证来使用。

![img](https://pics5.baidu.com/feed/b03533fa828ba61ed50931bbcf484d02324e59c0.jpeg?token=6ffbeb5a9df4cf5eb325db0f176d77e4)

![img](https://pics4.baidu.com/feed/1f178a82b9014a90f3ffe63d390be31ab11beef6.jpeg?token=4b9a928c7dc8010b466c9c81a6a7c348)

后台使用 MongoDB 作为存储，支持 ES 接入。

我们同样使用 Docker Compose 来部署整个 Gravitee 的栈：

\## Copyright (C) 2015 The Gravitee team (http://gravitee.io)## Licensed under the Apache License, Version 2.0 (the "License");# you may not use this file except in compliance with the License.# You may obtain a copy of the License at## http://www.apache.org/licenses/LICENSE-2.0## Unless required by applicable law or agreed to in writing, software# distributed under the License is distributed on an "AS IS" BASIS,# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.# See the License for the specific language governing permissions and# limitations under the License.#version: '3.7'networks: frontend: name: frontend storage: name: storagevolumes: data-elasticsearch: data-mongo:services: mongodb: image: mongo:${MONGODB_VERSION:-3.6} container_name: gio_apim_mongodb restart: always volumes: - data-mongo:/data/db - ./logs/apim-mongodb:/var/log/mongodb networks: - storage elasticsearch: image: docker.elastic.co/elasticsearch/elasticsearch:${ELASTIC_VERSION:-7.7.0} container_name: gio_apim_elasticsearch restart: always volumes: - data-elasticsearch:/usr/share/elasticsearch/data environment: - http.host=0.0.0.0 - transport.host=0.0.0.0 - xpack.security.enabled=false - xpack.monitoring.enabled=false - cluster.name=elasticsearch - bootstrap.memory_lock=true - discovery.type=single-node - "ES_JAVA_OPTS=-Xms512m -Xmx512m" ulimits: memlock: soft: -1 hard: -1 nofile: 65536 networks: - storage gateway: image: graviteeio/apim-gateway:${APIM_VERSION:-3} container_name: gio_apim_gateway restart: always ports: - "8082:8082" depends_on: - mongodb - elasticsearch volumes: - ./logs/apim-gateway:/opt/graviteeio-gateway/logs environment: - gravitee_management_mongodb_uri=mongodb://mongodb:27017/gravitee?serverSelectionTimeoutMS=5000&connectTimeoutMS=5000&socketTimeoutMS=5000 - gravitee_ratelimit_mongodb_uri=mongodb://mongodb:27017/gravitee?serverSelectionTimeoutMS=5000&connectTimeoutMS=5000&socketTimeoutMS=5000 - gravitee_reporters_elasticsearch_endpoints_0=http://elasticsearch:9200 networks: - storage - frontend deploy: resources: limits: cpus: '1' memory: 256M reservations: memory: 256M management_api: image: graviteeio/apim-management-api:${APIM_VERSION:-3} container_name: gio_apim_management_api restart: always ports: - "8083:8083" links: - mongodb - elasticsearch depends_on: - mongodb - elasticsearch volumes: - ./logs/apim-management-api:/opt/graviteeio-management-api/logs environment: - gravitee_management_mongodb_uri=mongodb://mongodb:27017/gravitee?serverSelectionTimeoutMS=5000&connectTimeoutMS=5000&socketTimeoutMS=5000 - gravitee_analytics_elasticsearch_endpoints_0=http://elasticsearch:9200 networks: - storage - frontend management_ui: image: graviteeio/apim-management-ui:${APIM_VERSION:-3} container_name: gio_apim_management_ui restart: always ports: - "8084:8080" depends_on: - management_api environment: - MGMT_API_URL=http://localhost:8083/management/organizations/DEFAULT/environments/DEFAULT/ volumes: - ./logs/apim-management-ui:/var/log/nginx networks: - frontend portal_ui: image: graviteeio/apim-portal-ui:${APIM_VERSION:-3} container_name: gio_apim_portal_ui restart: always ports: - "8085:8080" depends_on: - management_api environment: - PORTAL_API_URL=http://localhost:8083/portal/environments/DEFAULT volumes: - ./logs/apim-portal-ui:/var/log/nginx networks: - frontend

我们使用管理 UI 来创建四个对应的 API 来进行网关的路由，也可以用 API 的方式，Gravitee 是这个开源网关中，唯一管理 UI 也开源的产品。

![img](https://pics6.baidu.com/feed/d1160924ab18972b8bed23e177b1a1819f510a4e.jpeg?token=662f2b52de254a1ad73a5bbf2855344e)

使用 K6 压力测试的结果如下：

![img](https://pics5.baidu.com/feed/9213b07eca8065384d1c275b00a17b4cac34820d.jpeg?token=ac24f8c2e69b6030d8e4e1b4983c59a5)

和同样采用 Java 的 Zuul 类似，Gravitee 的响应只能达到 200 左右，而且还出现了一些错误。我们只好再一次提高网关的资源分配到 4 核 2G。

![img](https://pics5.baidu.com/feed/9e3df8dcd100baa14fd0a8d1c96c631ac9fc2e65.jpeg?token=3d008fb2b38662a74de258a460042d02)

提高资源分配后的性能来到了 500-700，稍微好于 Zuul。

总结

本文分析了几种开源 API 网关的架构和基本功能，为大家在架构选型的时候提供一些基本的参考信息，本文做作的测试数据比较简单，场景也比较单一，不能作为实际选型的依据。

**Nginx：**基于 C 开发的高性能 API 网关，拥有众多的插件，如果你的 API 管理的需求比较简单，接受手工配置路由，Nginx 是个不错的选择。

**Kong：**是基于 Nginx 的 API 网关，使用 OpenResty 和 Lua 扩展，后台使用 PostgreSQL，功能众多，社区的热度很高，但是性能上看比起 Nginx 有相当的损失。如果你对功能和扩展性有要求，可以考虑 Kong。

**APISIX：**和 Kong 的架构类似，但是采用了云原生的设计，使用 ETCD 作为后台，性能上比起 Kong 有相当的优势，适合对性能要求高的云原生部署的场景。特别提一下，APISIX 支持 MQTT 协议，对于构建 IOT 应用非常友好。

**Tyk：**使用 Golang 开发，后台使用 Redis，性能不错，如果你喜欢 Golang，可以考虑一下。

要注意的是 Tyk 的开源协议是 MPL，是属于修改代码后不能闭源，对于商业化应用不是很友好。

**Zuul：**是 Netflix 开源的基于 Java 的 API 网关组件，他并不是一款开箱即用的 API 网关，需要和你的 Java 应用一起构建，所有的功能都是通过集成其他组件的方式来使用。

适合对于 Java 比较熟悉，用 Java 构建的应用的场景，缺点是性能其他的开源产品要差一些，同样的性能条件下，对于资源的要求会更多。

**Gravitee：**是 Gravitee.io 开源的基于 Java 的 API 管理平台，它能对 API 的生命周期进行管理，即使是开源版本，也有很好的 UI 支持。

但是因为采用了 Java 构建，性能同样是短板，适合对于 API 管理有强烈需求的场景。
