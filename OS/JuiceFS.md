 文件系统是计算机中一个非常重要的组件，为存储设备提供一致的访问和管理方式。在不同的操作系统中，文件系统会有一些差别，但也有一些共性几十年都没怎么变化：

1. 文件以树形目录进行组织；
2. 数据是以文件的形式存在，提供 Open、Read、Write、Seek、Close 等API 进行访问；
3. 提供原子的重命名（Rename）操作改变文件或者目录的位置。

文件系统提供的访问和管理方法支撑了绝大部分的计算机应用，Unix 的“万物皆文件”的理念更是凸显了它的重要地位。JuiceFS 是一款开源分布式文件系统，创新的将对象存储作为底层存储介质，实现了存储空间的无限扩展。任何存入 JuiceFS 的文件都会按照特定规则被拆分成固定大小的数据块保存在对象存储，数据块的元数据则保存在 Redis、MySQL 等数据库中。

OSCHINA 请来了 Juicedata 合伙人苏锐 [@苏锐](https://my.oschina.net/u/5537108) 和大家一起探讨关于分布式文件系统相关的问题。

苏锐，Juicedata 合伙人，作为1号成员参与创建分布式文件系统 JuiceFS，先通过全球公有云上的 SaaS 产品获得国内外几十家商业客户，之后于 2021 年 1 月 JuiceFS 开源。

------

**问：**JuiceFS 是依赖 redis、mysql 什么版本呢？如何考虑系统安全问题呢？

**答：Redis 可以参考**[**最佳实践**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Fredis_best_practices)**，各种引擎用流行的版本都能支持，JuiceFS 会尽量兼容更多的引擎和版本。**

**问：**JuiceFS 的对象存储相对传统的块存储分布式文件系统有啥优点？JuiceFS 的存储现在是针对什么业务场景使用的？

**答：JuiceFS 是一个 object-based 的分布式文件系统，相比 block-based 有更好的弹性。Object Storage 可以认为是完全弹性的，但是 Block Storage 大多做不到。目前 JuiceFS 主要用在 大数据、AI、DevOps 和各种需要 NAS 的应用场景，在 juicefs.com 上可以看到很多案例和客户实践。**

**问：**如果文件被拆分成固定大小的数据块，那这些数据块是怎么保证顺序的，以及数据库块的大小是固定的吗？会不会出现大量的内存碎片？读取的时候是不是要占用大量内存进行合并数据库 ？

**答：比如想读 1GB 的文件，在存储中是 sequential read。如果全部读出来返回给请求端，更占内存，调用端的等待时间更长。都是 4K/64K/256K/1M 等容量去顺序读，以一个 streaming 的方式持续返回给调用端。所以 JuiceFS 把文件分成 block 存储不会影响性能。相反，还能提升性能。因为更容易通过并发方式同时读很多的 block 返回给调用端，当然获得高吞吐的同时，需要用一点内存资源来换。文件被分成很多 data block 存在对象存储中，每个 block key 会存在 meta engine 里（直接存太占空间了，设计上 16 个 block 为一个 chunk，存储 chunkid + offset）。block size 默认设为 4MiB 是最大值，实际会有小于该值的 block。碎片合并有的。读取的时候不用担心，各种存储系统在读的时候也都是按某个大小的 page/block 去读。**

**问：**考虑到存取速度、修复难度以及迁移，对于底层文件系统的选择有什么推荐的吗？

**答：存取速度容易判断，结合你的业务场景来测试评估就行了。我想说的是在分布式存储中，只用一些 Benchmark Tool 来看跑分和实际业务场景中的表现是有不同的，一定用业务场景来测试。 迁移方面，通用流行的访问协议都不难，比如 JuiceFS 完全兼容 POSIX、HDFS、S3，还提供 juicesync 这个数据迁移工具，兼容几十种对象存储的 API。要考虑的是数据迁移中，业务的影响。**

**问：**JuiceFS 的跨节点、机架、机房、区域的副本放置策略，可用性程度，能不能简单讲讲：一致性协议的实现以及涉及到元数据管理，主丛切换的及时性和正确性，最后就是各种突发情况下主从同步的策略，全面高效的负载均衡(空间/吞吐/副本)。

**答：JuiceFS 社区版使用流行的数据库引擎，包括 Redis、MySQL、PostgreSQL、TiKV 这些，每一个都有很多运维的实践经验。部署和运维的方案都用这些数据库引擎社区推荐的，必要的地方 JuiceFS 会提供一些最佳实践，比如**[**使用 Redis 做元数据引擎**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Fredis_best_practices)**。**

**问：**JuiceFS 是否提供开放 Key-Value Storage 的 API 接口，以及有哪些支持的程序设计语言？另外，对于有大量小文件写入的效率如何？

**答：可以用 S3 API 读写 JuiceFS 数据，参考**[**文档**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Fs3_gateway)**，如果以 POSIX 方式访问，各种编程语言都原生支持了。编程语言的 SDK 目前有 Java SDK（是兼容 HDFS API 的）。**

**问：**对于大文件的拆分性能是如何？

**答：JuiceFS 的优势所在，参照**[**文档**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Fbenchmark%2F)**。**

**问：**1.JuiceFS和 NAS NFS 存储有啥区别？JuiceFS 优势在哪里？2.JuiceFS 使用了什么设计模式？

**答：1. 相比 NAS，JuiceFS 为云环境设计；弹性容量；更丰富的访问方式，包括 POSIX、HDFS、S3、K8s CSI；在 大数据、AI、DevOps 和众多需要 NAS 的场景上都适用，更高性价比 2. 参考**[**架构设计**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Farchitecture)**采用元数据与数据分离的设计方案。**

**问：**同 Ceph 是一类产品吗？如果是优势在哪里？

**答：和 CephFS 是一类产品，详情可以参考**[**文档**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Fcomparison%2Fjuicefs_vs_cephfs)**。**

**问：**看了一篇利用 JuiceFS 实现 mysql 数据备份性能提升 10 倍的文章 里面用到的 JuiceFS 自带的性能分析工具真的不错，希望老师能分享下它的设计和架构。

**答：可以参考**[**文档**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcommunity%2Fperformance_evaluation_guide)**，然后再去看对应的代码实现。**

**问：**JuiceFS 主要应用场景是什么呢

**答：JuiceFS 可以应用于大数据、AI、容器平台、DevOps 等场景，具体可以参考**[**文档**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fjuicefs.com%2Fdocs%2Fzh%2Fcloud%2Fhow_to)**，也可以关注公众号：Juicedata。**

**问：**在官方文档中看到了 JuiceFS 和 Ceph、Alluxio 等产品的对比，但是感觉实际中的竞争对手可能是 MinIO 或者 OZone，请问是否做过与这2个产品的对比呢？最近在做对象存储的技术调研，主要是存储照片、音视频和文本文件等，麻烦给一些意见，感谢！

**答：MinIO 和 OZone 都是对象存储。JuiceFS 是文件系统，可以使用他俩做后端的持久层服务。如果您的需求是存储和访问，没有计算、分析、训练等场景，应该直接用对象存储更适合。**

**问：**请问下 JuiceFS 对于海量的小文件存储适合吗？性能如何？客户有历史文件，包括上亿的文件夹和文件。

**答：使用 Redis 做元数据引擎推荐存储 2亿以内文件，使用 TiKV 适合更大规模。性能要看场景，对于训练等读为主的场景有 cache 机制加速，性能不错。**

**如果 1 亿文件 Redis，MySQL，TiKV 都没有问题。要看你对哪个系统更熟悉，更有维护经验。性能方面要看业务场景、访问方式来做判断。**

**问：**请问在底层实现上是如何保证分布式文件系统的文件完整性的？

**答：简单说几点：1. 先写数据再写元数据，保证元数据写成功则数据完整；2. 元数据这边靠引擎的事务性来保证完整；3. 写进去之后 数据和元数据本身的可靠性分别依赖对象存储和元数据引擎来保证；4. 会使用对象存储提供的 checksum 能力。**

**问：**这个是类似 Fastdfs 做文件存储的吗，有什么区别现在企业中哪个好用？

**答：JuiceFS 面向云环境设计，在云环境中可以很容易用起来，也天然的做到弹性伸缩。在访问时支持 POSIX、HDFS、S3，在上面做应用开发更方便，尤其是组织各种 Pipeline。**

**问：**如果数据库坏了，恢复了比如说一天前的备份，文件数据也能确保恢复到一天前的状态，并能持续工作下去吗？

**答：可以部分恢复，各种数据库也都有更细粒度的容灾方案，比如 Redis AOF、MySQL Binlog，可以最大限度减少数据丢失风险。另一方面，生产环境上也会用主备策略。数据删除后，无论是元数据还是对象存储中的数据都会被删除。如果元数据恢复到之前一天的状态，数据不能被恢复，也是读不出来的。**

**问：**有没有可能，在对象存储中也存一个类似 binlog 的概念的日志，当数据库数据丢失又未备份时，可以通过这个类似 binlog 的结构，还原整个数据库？

**答：新版上增加了自动备份元数据到对象存储的功能，算是多一个备份。**

问：我认为，分布式文件系统，依赖于关系型数据库，是设计思路错误。正确的应该是反过来，关系型数据库，依赖于分布式文件系统。这样，单个数据库中，对应的后台硬盘空间是无限的。无论应用系统有多少容量的数据，关系型数据库都可以在不分库、不分表的情况下，通过增加新磁盘、调整分布式文件系统配置参数，来解决问题。不知道苏老师怎么看？

**答：理解您的说法，我也部分认可。JuiceFS 的架构设计在 High level 上与 Google File System 的那篇论文一直，采用元数据与数据分离管理的方式，也有很多其他的分布式文件系统用了这个思路。不同的地方时，在 Low level design 中，目前看到的大部分 DFS 面向裸磁盘设计。**

**但是放在云时代，我们重新抽象的去看这个问题，再结合我们自己十余年使用 DFS 的业务经验。DFS 承载了非常丰富的 Workload，对于规模、性能、持久性、可用性、成本等不同维度的要求是不一样的。我们尝试回答一个问题：如何用一个产品更好、更灵活的满足更多、更丰富的业务场景？**

**然后，在 JuiceFS 的架构设计上，选择用对象存储做数据持久层（类似 chunk server，data node），因为对象存储提供了成熟的 scalebility、availibility、high throughput、low cost 三方面的优势，这正是数据持久层需要的核心能力。**

**元数据引擎做了插件式设计，Redis、MySQL、TiKV、FoundationDB 等有各自的优势，能满足不同的业务场景需求。选择这些成熟、流行的系统，另一个好处是在开发者中已经积累了大量的使用和运维经验，不用增加学习负担。这是 JuiceFS 设计社区产品的理念，友好的使用体验，低学习门槛，合适但无需极致（我们另有极致版的设计）。**