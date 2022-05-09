# Hadoop

The Apache™ Hadoop® project develops open-source software for reliable, scalable, distributed computing.

The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

## 简介

Hadoop是一套开源的用于大规模数据集的分布式存储和处理的工具平台。它最早由Yahoo的技术团队根据Google所发布的公开论文思想用JAVA语言开发，现在则隶属于apache基金会。

Hadoop之父：Doug Cutting。

Hadoop以分布式文件系统HDFS（Hadoop distributed file system）和Map Reduce分布式计算框架为核心，为用户提供了底层细节透明的分布式基础设施。

HDFS的高容错性、高伸缩性等优点，允许用户将Hadoop部署在廉价的硬件上，构建分布式文件存储系统。

Map Reduce分布式计算框架则允许用户在不了解分布式系统底层细节的情况下开发并行、分布式的应用程序，充分利用大规模的计算资源，解决传统高性能单机无法解决的大数据处理问题。

总之，Hadoop是目前分析海量数据的首选工具，并已经被各行各业广泛应用于以下场景：

- 大数据量存储：分布式存储（各种云盘，百度，360~还有云平台均有Hadoop应用）
- 日志处理： Hadoop擅长这个
- 海量计算： 并行计算
- ETL：数据抽取到oracle、mysql、DB2、mongdb及主流数据库
- 使用HBase做数据分析：用扩展性应对大量读写操作—Facebook构建了基于HBase的实时数据分析系统
- 机器学习：比如Apache Mahout项目（Apache Mahout简介 常见领域：协作筛选、集群、归类）
- 搜索引擎：Hadoop + lucene实现
- 数据挖掘：目前比较流行的广告推荐
- 用户行为特征建模
- 个性化广告推荐

## **Hadoop技术生态**

自2008年成为Apache基金会的顶级项目后，经过长时间的发展，围绕着Hadoop又出现了大量的开源扩展技术框架，从而形成了一个庞大的Hadoop技术生态体系。

![Hadoop-Frame](F:\work\openGuide\BigData\Hadoop-Frame.png)

**Hadoop技术生态圈：**

我们通常说到的hadoop包括两部分，一是Hadoop核心技术（或者说狭义上的hadoop），对应为apache开源社区的一个项目，主要包括三部分内容：hdfs，mapreduce，yarn。其中hdfs用来存储海量数据，mapreduce用来对海量数据进行计算，yarn是一个通用的资源调度框架（是在hadoop2.0中产生的）。

另一部分指广义的，广义上指一个生态圈，泛指大数据技术相关的开源组件或产品，如hbase、hive、spark、pig、zookeeper、kafka、flume、phoenix、sqoop等。

生态圈中的这些组件或产品相互之间会有依赖，但又各自独立。比如habse和kafka会依赖zookeeper，hive会依赖mapreduce。

## **（一）Hdfs**

Hdfs是一种分布式文件系统，是Hadoop体系中数据存储管理的基础。它是一个高度容错的系统，能检测和应对硬件故障，用于在低成本的通用硬件上运行。Hdfs简化了文件的一致性模型，通过流式数据访问，提供高吞吐量应用程序数据访问功能，适合带有大型数据集的应用程序。

## **（二）Mapreduce**

MapReduce分为第一代（称为 MapReduce 1.0或者MRv1，对应hadoop第1代）和第二代（称为MapReduce 2.0或者MRv2，对应hadoop第2代）。第一代MapReduce计算框架，它由两部分组成：编程模型（programming model）和运行时环境（runtime environment）。它的基本编程模型是将问题抽象成Map和Reduce两个阶段，其中Map阶段将输入数据解析成key/value，迭代调用map()函数处理后，再以key/value的形式输出到本地目录，而Reduce阶段则将key相同的value进行规约处理，并将最终结果写到HDFS上。它的运行时环境由两类服务组成：JobTracker和TaskTracker，其中，JobTracker负责资源管理和所有作业的控制，而TaskTracker负责接收来自JobTracker的命令并执行它。

MapReduce 2.0或者MRv2具有与MRv1相同的编程模型，唯一不同的是运行时环境。MRv2是在MRv1基础上经加工之后，运行于资源管理框架YARN之上的MRv1，它不再由JobTracker和TaskTracker组成，而是变为一个作业控制进程ApplicationMaster，且ApplicationMaster仅负责一个作业的管理，至于资源的管理，则由YARN完成。

总结下，MRv1是一个独立的离线计算框架，而MRv2则是运行于YARN之上的MRv1。

## **（三）Hive**

Hive是一种基于Hadoop的数据仓库，由facebook开源，最初用于解决海量结构化的日志数据统计问题。Hive定义了一种类似SQL的查询语言(HQL),将SQL转化为MapReduce任务在Hadoop上执行。通常用于离线分析。

## **（四）Hbase**

HBase是Google Bigtable的克隆版。它是一个针对结构化数据的可伸缩、高可靠、高性能、分布式和面向列的动态模式数据库。和传统关系数据库不同，HBase采用了BigTable的数据模型：增强的稀疏排序映射表（Key/Value），其中，键由行关键字、列关键字和时间戳构成。HBase提供了对大规模数据的随机、实时读写访问，同时，HBase中保存的数据可以使用MapReduce来处理，它将数据存储和并行计算完美地结合在一起。

## **（五）Zookeeper**

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。ZooKeeper包含一个简单的原语集，提供Java和C的接口。

## **（六）Sqoop**

Sqoop是一款开源的工具，主要用于在Hadoop和传统的数据库(mysql、postgresql等)进行数据的传递，可以将一个关系型数据库（例如：MySQL、Oracle、Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

Sqoop分为一代（称为Sqoop1）和二代（称为Sqoop2），其中Sqoop1的架构，仅仅使用一个Sqoop客户端，Sqoop2的架构，引入了Sqoop server集中化管理connector，以及rest api，web，UI，并引入权限安全机制。

## **（七）Pig**

Apache Pig是MapReduce的一个抽象。它是一个工具/平台，用于分析较大的数据集，并将它们表示为数据流。Pig通常与 Hadoop 一起使用；我们可以使用Apache Pig在Hadoop中执行所有的数据处理操作。要编写数据分析程序，Pig提供了一种称为 Pig Latin 的高级语言。该语言提供了各种操作符，程序员可以利用它们开发自己的用于读取，写入和处理数据的功能。

要使用 Apache Pig 分析数据，程序员需要使用Pig Latin语言编写脚本。所有这些脚本都在内部转换为Map和Reduce任务。Apache Pig有一个名为 Pig Engine 的组件，它接受Pig Latin脚本作为输入，并将这些脚本转换为MapReduce作业。
所以使用PIG，可以让不太擅长编写Java程序的程序员来进行大数据分析处理。

## **（八）Mahout**

Mahout起源于2008年，最初是Apache Lucent的子项目，它在极短的时间内取得了长足的发展，现在是Apache的顶级项目。

Mahout的主要目标是创建一些可扩展的机器学习领域经典算法的实现，旨在帮助开发人员更加方便快捷地创建智能应用程序。Mahout现在已经包含了聚类、分类、推荐引擎（协同过滤）和频繁集挖掘等广泛使用的数据挖掘方法。除了算法，Mahout还包含数据的输入/输出工具、与其他存储系统（如数据库、MongoDB 或Cassandra）集成等数据挖掘支持架构。

## **（九）Flume**

Flume是Cloudera（一个知名的基于开源hadoop的大数据发行商）设计开发的一个开源的日志收集工具， 具有分布式、高可靠、高容错、易于定制和扩展的特点。它将数据从产生、传输、处理并最终写入目标的路径的过程抽象为数据流，在具体的数据流中，数据源支持在Flume中定制数据发送方，从而支持收集各种不同协议数据。同时，Flume数据流提供对日志数据进行简单处理的能力，如过滤、格式转换等。此外，Flume还具有能够将日志写往各种数据目标（可定制）的能力。总的来说，Flume是一个可扩展、适合复杂环境的海量日志收集系统。

## **（十）Spark**

Spark是一个通用计算引擎，能对大规模数据进行快速分析，可用它来完成各种各样的运算，包括 SQL 查询、文本处理、机器学习等，而在 Spark 出现之前，我们一般需要学习各种各样的引擎来分别处理这些需求。Spark不依赖于MapReduce，它使用了自己的数据处理框架。Spark使用内存进行计算，速度更快。Spark本身就是一个生态系统，除了核心API之外，Spark生态系统中还包括其他附加库，可以在大数据分析和机器学习领域提供更多的能力，如Spark SQL，Spark Streaming，Spark MLlib，Spark GraphX，BlinkDB，Tachyon等。

## **（十一）Storm**

Storm是Twitter开源的分布式实时大数据处理框架，最早开源于github，从0.9.1版本之后，归于Apache社区，被业界称为实时版Hadoop。它与Spark Streaming的最大区别在于它是逐个处理流式数据事件，而Spark Streaming是微批次处理，因此，它比Spark Streaming更实时。

## **（十二）Impala**

Impala是Cloudera公司主导开发的新型查询系统，它提供SQL语义，能查询存储在Hadoop的HDFS和HBase中的PB级大数据。已有的Hive系统虽然也提供了SQL语义，但由于Hive底层执行使用的是MapReduce引擎，仍然是一个批处理过程，难以满足查询的交互性。相比之下，Impala的最大特点也是最大卖点就是它的快速。

另外Impala可以Hive结合使用，它可以直接使用Hive的元数据库Metadata。

## **（十三）Kafka**

Kafka是一种分布式的，基于发布/订阅的消息系统,类似于消息对列的功能，可以接收生产者（如webservice、文件、hdfs、hbase等）的数据，本身可以缓存起来，然后可以发送给消费者（同上），起到缓冲和适配的作。

## **（十四）Yarn**

Yarn是一种新的 Hadoop 资源管理器，它是一个通用资源管理系统，可为上层应用提供统一的资源管理和调度。它将资源管理和处理组件分开，它的引入为集群在利用率、资源统一管理和数据共享等方面带来了巨大好处。可以把它理解为大数据集群的操作系统。可以在上面运行各种计算框架（包括MapReduce、Spark、Storm、MPI等）。

## **（十五）Hue**

Hue是一个开源的Apache Hadoop UI系统,通过使用Hue我们可以在浏览器端的Web控制台上与Hadoop集群进行交互来分析处理数据，例如操作HDFS上的数据，运行MapReduce Job等等。

## **（十六）Oozie**

在Hadoop中执行的任务有时候需要把多个Map/Reduce作业连接到一起，这样才能够达到目的。Oozie让我们可以把多个Map/Reduce作业组合到一个逻辑工作单元中，从而完成更大型的任务。wuOozie是一种Java Web应用程序，它运行在Java servlet容器中，并使用数据库来存储相关信息。

## **（十七）Ambari**

Ambari是一个开源的大数据集群管理系统，可以用来就是创建、管理、监视 Hadoop 的集群，并提供WEB可视化的界面来让用户进行管理。

## **分阶段学习**

**1、先从单个组件学习**

一般是先从hadoop核心组件HDFS,maperduce开始学习，然后再逐步学习其它组件。

**2、单个组件的基础学习**

先掌握单个组件（以及依赖组件）的安装和运行，开始可以先是单机安装，hadoop生态圈的各个组件基本都支持在一台机器上进行安装和运行，以便于简化开发阶段的环境准备。而在应用上线后，往往是集群方式安装和运行。

同时要理解组件的基本架构和原理，对组件有一个整体层面的了解。

另外站在使用者角度（如开发者角度）去学习组件的使用，比如对于hdfs，知道如何通过命令行方式使用hdfs提供的命令进行文件的操作，如何通过组件提供的api（如java api）来编写程序进行操作。

**3、对单个组件进行深入学习，包括但不限于如下方面：**

1）深入了解组件的原理和架构

2）了解组件分布式部署的配置和性能调优

3）阅读组件的源代码，理解其实现机制

4）发现组件源代码中的问题和不足，向开源社区提交issue，为开源社区做贡献

**4、针对业务场景给出大数据的解决方案**
比如利用哪些组件配合来解决对应的业务问题，集群如何配置等。

## Hadoop的整体框架

Hadoop由HDFS、MapReduce、HBase、Hive和ZooKeeper等成员组成，其中最基础最重要元素为底层用于存储集群中所有存储节点文件的文件系统HDFS（Hadoop Distributed File System）来执行MapReduce程序的MapReduce引擎。

##### Hadoop中有3个核心组件：

分布式文件系统：HDFS —— 实现将文件分布式存储在很多的服务器上

分布式运算编程框架：MAPREDUCE —— 实现在很多机器上分布式并行运算

分布式资源调度平台：YARN —— 帮用户调度大量的mapreduce程序，并合理分配运算资源

## Hadoop HDFS

Hadoop之（HDFS）是一种分布式文件系统，设计用于在商用硬件上运行。 它与现有的分布式文件系统有许多相似之处。 但是，与其他分布式文件系统的差异很大。
HDFS具有高度容错能力，旨在部署在低成本硬件上。
HDFS提供对应用程序数据的高吞吐量访问，适用于具有大型数据集的应用程序。
HDFS放宽了一些POSIX要求，以实现对文件系统数据的流式访问

HDFS优势：

```
1、可构建在廉价机器上，设备成本相对低
2、高容错性，HDFS将数据自动保存多个副本，副本丢失后，自动恢复，防止数据丢失或损坏
3、适合批处理，HDFS适合一次写入、多次查询（读取）的情况，适合在已有的数据进行多次分析，稳定性好
4、适合存储大文件，其中的大表示可以存储单个大文件，因为是分块存储，以及表示存储大量的数据
```

HDFS劣势：

```
1、由于提高吞吐量，降低实时性
2、由于每个文件都会在namenode中记录元数据，如果存储了大量的小文件，会对namenode造成很大的压力3、不合适小文件处理，在mapreduce的过程中小文件的数量会造成map数量的增大，导致资源被占用，而且速度慢。
4、不适合文件的修改，文件只能追加在文件的末尾，不支持任意位置修改，不支持多个写入者操作
```

## HDFS架构

HDFS架构图如下图所示：

![Hadoop HDFS](F:\work\openGuide\BigData\Hadoop HDFS.png)

HDFS具有主/从架构。HDFS集群由单个NameNode，和多个datanode构成。

**NameNode**：管理文件系统命名空间的主服务器和管理客户端对文件的访问组成，如打开，关闭和重命名文件和目录。负责管理文件目录、文件和block的对应关系以及block和datanode的对应关系，维护目录树，接管用户的请求。

1、将文件的元数据保存在一个文件目录树中
2、在磁盘上保存为：fsimage 和 edits
3、保存datanode的数据信息的文件，在系统启动的时候读入内存。

**DataNode**：（数据节点）管理连接到它们运行的节点的存储，负责处理来自文件系统客户端的读写请求。DataNodes还执行块创建，删除

**Client**：(客户端)代表用户通过与nameNode和datanode交互来访问整个文件系统，HDFS对外开放文件命名空间并允许用户数据以文件形式存储。用户通过客户端（Client）与HDFS进行通讯交互。

**块和复制：**
我们都知道linux操作系统中的磁盘的块的大小默认是512，而hadoop2.x版本中的块的大小默认为128M，那为什么hdfs中的存储块要设计这么大呢？
其目的是为了减小寻址的开销。只要块足够大，磁盘传输数据的时间必定会明显大于这个块的寻址时间。

那为什么要以块的形式存储文件，而不是整个文件呢？
1、因为一个文件可以特别大，可以大于有个磁盘的容量，所以以块的形式存储，可以用来存储无论大小怎样的文件。
2、简化存储系统的设计。因为块是固定的大小，计算磁盘的存储能力就容易多了
3、以块的形式存储不需要全部存在一个磁盘上，可以分布在各个文件系统的磁盘上，有利于复制和容错，数据本地化计算

既然namenode管理着文件系统的命名空间，维护着文件系统树以及整颗树内的所有文件和目录，这些信息以文件的形式永远的保存在本地磁盘上，分别问命名空间镜像文件fsimage和编辑日志文件Edits。datanode是文件的工作节点，根据需要存储和检索数据块，并且定期的向namenode发送它们所存储的块的列表。那么就知道namenode是多么的重要，一旦那么namenode挂了，那整个分布式文件系统就不可以使用了，所以对于namenode的容错就显得尤为重要了，hadoop为此提供了两种**容错机制**：

**容错机制一：**

​    就是通过对那些组成文件系统的元数据持久化，分别问命名空间镜像文件fsimage（文件系统的目录树）和编辑日志文件Edits（针对文件系统做的修改操作记录）。磁盘上的映像FsImage就是一个Checkpoint,一个里程碑式的基准点、同步点,有了一个Checkpoint之后,NameNode在相当长的时间内只是对内存中的目录映像操作,同时也对磁盘上的Edits操作,直到关机。下次开机的时候,NameNode要从磁盘上装载目录映像FSImage,那其实就是老的Checkpoint,也许就是上次开机后所保存的映像,而自从上次开机后直到关机为止对于文件系统的所有改变都记录在Edits文件中;将记录在Edits中的操作重演于上一次的映像,就得到这一次的新的映像,将其写回磁盘就是新的Checkpoint（也就是fsImage）。但是这样有很大一个缺点，如果Edits很大呢，开机后生成原始映像的过程也会很长，所以对其进行改进：每当 Edits长到一定程度,或者每隔一定的时间,就做一次Checkpoint，但是这样就会给namenode造成很大的负荷，会影响系统的性能。于是就有了SecondaryNameNode的需要,这相当于NameNode的助理,专替NameNode做Checkpoint。当然,SecondaryNameNode的负载相比之下是偏轻的。所以如果为NameNode配上了热备份,就可以让热备份兼职,而无须再有专职的SecondaryNameNode。

SecondaryNameNode主要负责下载NameNode中的fsImage文件和Edits文件，并合并生成新的fsImage文件，并推送给NameNode，工作原理如下：

1、secondarynamenode请求主namenode停止使用edits文件，暂时将新的写操作记录到一个新的文件中；
2、secondarynamenode从主namenode获取fsimage和edits文件（通过http get）
3、secondarynamenode将fsimage文件载入内存，逐一执行edits文件中的操作，创建新的fsimage文件。
4、secondarynamenode将新的fsimage文件发送回主namenode（使用http post）.
5、namenode用从secondarynamenode接收的fsimage文件替换旧的fsimage文件；用步骤1所产生的edits文件替换旧的edits文件。同时，还更新fstime文件来记录检查点执行时间。
6、最终，主namenode拥有最新的fsimage文件和一个更小的edits文件。当namenode处在安全模式时，管理员也可调用hadoop dfsadmin –saveNameSpace命令来创建检查点。

​    从上面的过程中我们清晰的看到secondarynamenode和主namenode拥有相近内存需求的原因（因为secondarynamenode也把fsimage文件载入内存）。因此，在大型集群中，secondarynamenode需要运行在一台专用机器上。

   创建检查点的触发条件受两个配置参数控制。通常情况下，secondarynamenode每隔一小时（有fs.checkpoint.period属性设置）创建检查点；此外，当编辑日志的大小达到64MB（有fs.checkpoint.size属性设置）时，也会创建检查点。系统每隔五分钟检查一次编辑日志的大小。

**容错机制二：**

高可用方案

## HDFS读数据流程

![Hadoop HDFS Read](F:\work\openGuide\BigData\Hadoop HDFS Read.png)

1、客户端通过FileSystem对象（DistributedFileSystem）的open()方法来打开希望读取的文件。

2、DistributedFileSystem通过远程调用（RPC）来调用namenode，获取到每个文件的起止位置。对于每一个块，namenode返回该块副本的datanode。这些datanode会根据它们与客户端的距离（集群的网络拓扑结构）排序，如果客户端本身就是其中的一个datanode，那么就会在该datanode上读取数据。DistributedFileSystem远程调用后返回一个FSDataInputStream（支持文件定位的输入流）对象给客户端以便于读取数据，然后FSDataInputStream封装一个DFSInputStream对象。该对象管理datanode和namenode的IO。

3、客户端对这个输入流调用read()方法，存储着文件起始几个块的datanode地址的DFSInputStream随即连接距离最近的文件中第一个块所在的datanode，通过数据流反复调用read()方法，可以将数据从datanode传送到客户端。当读完这个块时，DFSInputStream关闭与该datanode的连接，然后寻址下一个位置最佳的datanode。

   客户端从流中读取数据时，块是按照打开DFSInputStream与datanode新建连接的顺序读取的。它也需要询问namenode来检索下一批所需块的datanode的位置。一旦客户端完成读取，就对FSDataInputStream调用close()方法。

  **注意**：在读取数据的时候，如果DFSInputStream在与datanode通讯时遇到错误，它便会尝试从这个块的另外一个临近datanode读取数据。他也会记住那个故障datanode，以保证以后不会反复读取该节点上后续的块。DFSInputStream也会通过校验和确认从datanode发送来的数据是否完整。如果发现一个损坏的块， DFSInputStream就会在试图从其他datanode读取一个块的复本之前通知namenode。

  **总结**：在这个设计中，namenode会告知客户端每个块中最佳的datanode，并让客户端直接联系该datanode且检索数据。由于数据流分散在该集群中的所有datanode，所以这种设计会使HDFS可扩展到大量的并发客户端。同时，namenode仅需要响应位置的请求（这些信息存储在内存中，非常高效），而无需响应数据请求，否则随着客户端数量的增长，namenode很快会成为一个瓶颈。

## HDFS写数据流程

![Hadoop HDFS Write](F:\work\openGuide\BigData\Hadoop HDFS Write.png)

1、首先客户端通过DistributedFileSystem上的create()方法指明一个预创建的文件的文件名

2、DistributedFileSystem再通过RPC调用向NameNode申请创建一个新文件（这时该文件还没有分配相应的block）。namenode检查是否有同名文件存在以及用户是否有相应的创建权限，如果检查通过，namenode会为该文件创建一个新的记录，否则的话文件创建失败，客户端得到一个IOException异常。DistributedFileSystem返回一个FSDataOutputStream以供客户端写入数据，与FSDataInputStream类似，FSDataOutputStream封装了一个DFSOutputStream用于处理namenode与datanode之间的通信。

3、当客户端开始写数据时（，DFSOutputStream把写入的数据分成包（packet）, 放入一个中间队列——数据队列（data queue）中去。DataStreamer从数据队列中取数据，同时向namenode申请一个新的block来存放它已经取得的数据。namenode选择一系列合适的datanode（个数由文件的replica数决定）构成一个管道线（pipeline），这里我们假设replica为3，所以管道线中就有三个datanode。

4、DataSteamer把数据流式的写入到管道线中的第一个datanode中，第一个datanode再把接收到的数据转到第二个datanode中，以此类推。

5、DFSOutputStream同时也维护着另一个中间队列——确认队列（ack queue），确认队列中的包只有在得到管道线中所有的datanode的确认以后才会被移出确认队列

如果某个datanode在写数据的时候当掉了，下面这些对用户透明的步骤会被执行：

  管道线关闭，所有确认队列上的数据会被挪到数据队列的首部重新发送，这样可以确保管道线中当掉的datanode下流的datanode不会因为当掉的datanode而丢失数据包。

  在还在正常运行的datanode上的当前block上做一个标志，这样当当掉的datanode重新启动以后namenode就会知道该datanode上哪个block是刚才当机时残留下的局部损坏block，从而可以把它删掉。

  已经当掉的datanode从管道线中被移除，未写完的block的其他数据继续被写入到其他两个还在正常运行的datanode中去，namenode知道这个block还处在under-replicated状态（也即备份数不足的状态）下，然后他会安排一个新的replica从而达到要求的备份数，后续的block写入方法同前面正常时候一样。有可能管道线中的多个datanode当掉（虽然不太经常发生），但只要dfs.replication.min（默认为1）个replica被创建，我们就认为该创建成功了。剩余的replica会在以后异步创建以达到指定的replica数。

6、当客户端完成写数据后，它会调用close()方法。这个操作会冲洗（flush）所有剩下的package到pipeline中。

7、等待这些package确认成功，然后通知namenode写入文件成功。这时候namenode就知道该文件由哪些block组成（因为DataStreamer向namenode请求分配新block，namenode当然会知道它分配过哪些blcok给给定文件），它会等待最少的replica数被创建，然后成功返回。


**注意**：hdfs在写入的过程中，有一点与hdfs读取的时候非常相似，就是：DataStreamer在写入数据的时候，每写完一个datanode的数据块,都会重新向nameNode申请合适的datanode列表。这是为了保证系统中datanode数据存储的均衡性。

hdfs写入过程中，datanode管线的确认应答包并不是每写完一个datanode，就返回一个确认应答，而是一直写入，直到最后一个datanode写入完毕后，统一返回应答包。如果中间的一个datanode出现故障，那么返回的应答就是前面完好的datanode确认应答，和故障datanode的故障异常。这样我们也就可以理解，在写入数据的过程中，为什么数据包的校验是在最后一个datanode完成。











# 安装

> 开启Hadoop：start-all.sh
>
> 关闭Hadoop：stop-all.sh

## 单机模式

只能启动MapReduce，所以不使用单机模式

## 伪分布式

### 环境配置

> 能启动HDFS、MapReduce、Yran的大部分功能

1. 关闭防火墙

   ```sh
   service iptables stop
   chkconfig iptables off
   ```

2. 修改云主机的主机名，Hadoop集群中要求主机名中不能出现`-`或者`_`

   ```sh
   cd /etc/sysconfig
   vim network
   ```

   修改

   ```
   HOSTNAME=hadoop01
   ```

   保存退出，重新生效

   ```sh
   source network
   ```

3. 需要将IP和主机名进行映射

   ```sh
   cd /etc
   vim hosts
   ```

   添加host映射

   ```sh
   10.9.162.133 hadoop01
   ```

4. 重启

   ```sh
   reboot
   ```

5. 配置免密登录

   产生密钥：`ssh-keygen`

   拷贝密钥：`ssh-copy-id root@hadoop01`

   测试是否能够免密登录：`ssh hadoop01`

   如果成功，则退出：`logout`

6. 下载Hadoop的安装包

   ```sh
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/hadoop-alone.tar.gz
   ```

7. 解压安装包

   ```sh
   tar -xvf hadoop-alone.tar.gz
   ```

8. 配置环境变量

   ```sh
   vim /etc/profile
   ```

   文件末尾添加

   ```sh
   export HADOOP_HOME=/home/software/hadoop-2.7.1
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ```

   保存退出，重新生效

   ```sh
   source /etc/profile
   ```

9. 格式化`namenode`

   ```sh
   hadoop namenode -format
   ```

   如果出现

   `Storage directory /home/software/hadoop-2.7.1/tmp/dfs/name has been successfully formatted.`

   表示格式化成功

10. 启动hadoop

    ```sh
    start-all.sh
    ```

    通过jps查看，有6个进程

    ```
    Jps
    NameNode
    DataNode
    SecondaryNameNode
    ResourceManager
    NodeManager
    ```

11. 通过网页浏览

    可以通过浏览器访问HDFS的页面，访问地址为：IP地址:50070

    可以通过浏览器访问Yarn的页面，访问地址为：IP地址:8088



### 配置文件的修改

> 进入：`cd hadoop-2.7.1/etc/hadoop`

1. 配置`hadoop-env.sh`

   - 编辑hadoop-env.sh:`vim hadoop-env.sh`

   - 修改JAVA_HOME的路径：

     `export JAVA_HOME=/home/software/jdk1.8`

   - 修改HADOOP_CONF_DIR的路径

     `export HADOOP_CONF_DIR=/home/software/hadoop-2.7.1/etc/hadoop`

   - 保存退出

   - 重新生效：`source hadoop-env.sh`

2. 配置`core-site.xml`

   - 编辑core-site.xml：`vim core-site.xml`

   - 指定如下内容

     ```xml
     <property>
         <!-- 指定HDFS中的主节点 - namenode -->
         <name>fs.defaultFS</name>               
         <value>hdfs://hadoop01:9000</value>
     </property>
     <property>
         <!-- 执行Hadoop运行时的数据存放目录 -->
         <name>hadoop.tmp.dir</name>
         <value>/home/software/hadoop-2.7.1/tmp</value>
     </property>
     
     ```

   - 保存退出

3. 配置`hdfs-site.xml`

   - 编辑文件：`vim hdfs-site.xml`

   - 添加如下内容

     ```xml
     <property>
         <!-- 设置HDFS中的副本数量 -->
         <!-- 在伪分布式下，值设置为1 -->
         <name>dfs.replication</name>
         <value>1</value>
     </property>
     ```

   - 保存退出

4. 配置`mapred-site.xml`

   - 将`mapred-site.xml.template`复制为`mapred-site.xml`

     ```sh
     cp mapred-site.xml.template mapred-site.xml
     ```

   - 编辑新文件：`vim mapred-site.xml`

   - 添加如下配置

     ```xml
     <property>
         <!-- 指定将MapReduce在Yarn上运行  -->
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
     </property>
     ```

   - 保存退出

5. 配置`yarn-site.xml`

   - 编辑yarn-site.xml：`vim yarn-site.xml`

   - 添加如下内容：

     ```xml
     <!-- 指定Yarn的主节点 - resourcemanager -->
     <property>
         <name>yarn.resourcemanager.hostname</name>
         <value>hadoop01</value>
     </property>
     <!-- NodeManager的数据获取方式 -->
     <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
     </property>
     ```

   - 保存退出

6. 配置`slaves`

   - 编辑slaves：`vim slaves`
   - 添加从节点信息，例如：hadoop01
   - 保存退出



## 完全分布式

> 能启动Hadoop的所有功能



![](https://note.youdao.com/yws/api/personal/file/0603301A60424E53818AAA123932BFB5?method=download&shareKey=0a2dcfc06ac8fed63470e80cb6334a26)



> 如上图所示，完全分布式的Hadoop的结构，最少需要13个节点，才能保证高可用，也就是最少需要13台服务器

只有三台服务器，在三台服务器上搭建Hadoop完全分布式集群

![](https://note.youdao.com/yws/api/personal/file/39FB12A84E2B431D8F66D78355CD81C3?method=download&shareKey=ef1c63c763f83a1a08fe47848de1433d)



### 环境配置

> 因为第一台 服务器搭建过伪分布式，所以需要将伪分布式Hadoop文件重命名

```sh
cd /home/software/
mv hadoop-2.7.1/ hadoop-alone
```



1. 三台云主机关闭防火墙

   ```sh
   service iptables stop
   chkconfig iptables off
   ```

2. 修改三台云主机的主机名

   ```sh
   vim /etc/sysconfig/network
   ```

   修改HOSTNAME属性，依次改为hadoop01、hadoop02、hadoop03

   保存退出，并且重新生效

   ```sh
   source /etc/sysconfig/network
   ```

3. 三台云主机添加IP映射

   > 编辑hosts文件`vim /etc/hosts`，添加如下内容

   ```sh
   10.9.162.133 hadoop01
   10.9.152.65 hadoop02
   10.9.130.83 hadoop03
   ```

4. 三台云主机重启：`reboot`

5. 三台云主机开启Zookeeper

   ```sh
   cd /home/software/zookeeper-3.4.8/bin 
   sh zkServer.sh start	#开启Zookeeper
   sh zkServer.sh status	#检查Zookeeper的状态，应该为2个follower1个leader
   ```

6. 返回第一个节点software目录下：`cd /home/software`

7. 下载Hadoop完全分布式的安装包

   ```sh
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/hadoop-dist.tar.gz
   ```

8. 三台机器进行免密登录

   ```sh
   ssh-keygen
   ssh-copy-id	root@hadoop01
   ssh-copy-id	root@hadoop02
   ssh-copy-id	root@hadoop03
   ```

   三台集器测试：

   ```sh
   ssh hadoop01 #如果不需要密码logout
   ssh hadoop02 #如果不需要密码logout
   ssh hadoop03 #如果不需要密码logout
   ```

9. 解压安装包

   ```sh
   tar -xvf hadoop-dist.tar.gz
   ```

10. 将Hadoop的安装包拷贝到其他两个节点上

    ```sh
    scp -r hadoop-2.7.1/ root@hadoop02:/home/software/
    scp -r hadoop-2.7.1/ root@hadoop03:/home/software/
    ```

11. 三台节点配置环境变量

    > 修改文件：`vim /etc/profile`

    ```sh
    export HADOOP_HOME=/home/software/hadoop-2.7.1
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    ```

    保存退出，并且重新生效：`source /etc/profile`

12. 在任意一个节点格式化Zookeeper

    > 实际上就是在Zookeeper中注册Hadoop节点

    ```sh
    hdfs zkfc -formatZK
    ```

    如果成功，日志会显示：`Successfully created /hadoop-ha/ns in ZK`

    如果出现：`HA is not available`，说明系统兼容性不够，重装系统解决

13. 三台云主机启动`JournalNode`

    ```sh
    hadoop-daemon.sh start journalnode
    ```

14. 在第一个节点上格式化`NameNode`

    ```sh
    hadoop namenode -format
    ```

    如果格式化成功，则出现：`Storage directory /home/software/hadoop-2.7.1/tmp/hdfs/name has been successfully formatted. `

15. 在第一个节点上启动NameNode

    ```sh
    hadoop-daemon.sh start namenode
    ```

16. 在第二个节点上格式化NameNode

    ```sh
    hdfs namenode -bootstrapStandby
    ```

    如果格式化成功，则出现：`Storage directory /home/software/hadoop-2.7.1/tmp/hdfs/name has been successfully formatted.`

17. 在第二个节点启动NameNode

    ```sh
    hadoop-daemon.sh start namenode
    ```

18. 在==三个节点上都启动==DataNode

    ```sh
    hadoop-daemon.sh start datanode
    ```

19. 在==第三个节点==上启动YARN

    ```sh
    start-yarn.sh
    ```

20. 在第一个节点上启动ResourceManager

    ```sh
    yarn-daemon.sh start resourcemanager
    ```

21. 在第一个和第二个节点上启动zkfc

    ```sh
    hadoop-daemon.sh start zkfc
    ```



> 如果都启动成功，通过jps查看：
>
> - 第一个节点出现8个进程
>
>   ```
>   - Jps
>   - NameNode
>   - DataNode
>   - JournalNode
>   - ResourceManager
>   - NodeManager
>   - DFSZKFailoverController
>   - QuorumPeerMain
>   ```
>
> - 第二个节点出现7个进程
>
>   ```
>   - Jps
>   - NameNode
>   - DataNode
>   - JournalNode
>   - NodeManager
>   - DFSZKFailoverController
>   - QuorumPeerMain
>   ```
>
> - 第三个节点出现6个进程
>
>   ```
>   - Jps
>   - DataNode
>   - JournalNode
>   - ResourceManager
>   - NodeManager
>   - QuorumPeerMain
>   ```
>
> 如果少了QuorumPeerMain，表示Zookeeper启动出错
>
> 如果少了NameNode/DataNode/JournalNode/DFSZKFailoverController
>
> - 可以通过`hadoop-daemon.sh start namenode/datanode/journalnode/zkfc`
>
> 如果少了ResourceManager/NodeManager
>
> - 可以通过`yarn-daemon.sh start resourcemanager/nodemanager`
>
> ==Hadoop完全分布式开启，需要先启动ZooKeeper，再开Hadoop==



### 配置文件的修改

> 上述下载的Hadoop的jar包中已经配置好了

1. 编辑core-site.xml，添加如下内容：

   ```xml
   <!--指定hdfs的nameservice，为整个集群起一个别名-->
   <property>
       <name>fs.defaultFS</name>                
       <value>hdfs://ns</value>
   </property>
   <!--指定Hadoop数据临时存放目录-->
   <property>
       <name>hadoop.tmp.dir</name>
       <value>/home/software/hadoop-2.7.1/tmp</value>
   </property> 
   <!--指定zookeeper的存放地址-->
   <property>
       <name>ha.zookeeper.quorum</name>
       <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
   </property> 
   ```

2. 编辑hdfs-site.xml

   ```xml
   <!--执行hdfs的nameservice为ns，注意要和core-site.xml中的名称保持一致-->
   <property>
       <name>dfs.nameservices</name>
       <value>ns</value>
   </property>
   <!--ns集群下有两个namenode，分别为nn1, nn2-->
   <property>
       <name>dfs.ha.namenodes.ns</name>
       <value>nn1,nn2</value>
   </property>
   <!--nn1的RPC通信-->
   <property>
       <name>dfs.namenode.rpc-address.ns.nn1</name>
       <value>hadoop01:9000</value>
   </property>
   <!--nn1的http通信-->
   <property>
       <name>dfs.namenode.http-address.ns.nn1</name>
       <value>hadoop01:50070</value>
   </property>
   <!-- nn2的RPC通信地址 -->
   <property>
       <name>dfs.namenode.rpc-address.ns.nn2</name>
       <value>hadoop02:9000</value>
   </property>
   <!-- nn2的http通信地址 -->
   <property>
       <name>dfs.namenode.http-address.ns.nn2</name>
       <value>hadoop02:50070</value>
   </property>
   <!--指定namenode的元数据在JournalNode上存放的位置，这样，namenode2可以从journalnode集群里的指定位置上获取信息，达到热备效果-->
   <property>
       <name>dfs.namenode.shared.edits.dir</name>
       <value>qjournal://hadoop01:8485;hadoop02:8485;hadoop03:8485/ns</value>
   </property>
   <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
   <property>
       <name>dfs.journalnode.edits.dir</name>
       <value>/home/software/hadoop-2.7.1/tmp/journal</value>
   </property>
   <!-- 开启NameNode故障时自动切换 -->
   <property>
       <name>dfs.ha.automatic-failover.enabled</name>
       <value>true</value>
   </property>
   <!-- 配置失败自动切换实现方式 -->
   <property>
       <name>dfs.client.failover.proxy.provider.ns</name>
       <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
   </property>
   <!-- 配置隔离机制 -->
   <property>
       <name>dfs.ha.fencing.methods</name>
       <value>sshfence</value>
   </property>
   <!-- 使用隔离机制时需要ssh免登陆 -->
   <property>
       <name>dfs.ha.fencing.ssh.private-key-files</name>
       <value>/root/.ssh/id_rsa</value>
   </property>
   <!--配置namenode存放元数据的目录，可以不配置，如果不配置则默认放到hadoop.tmp.dir下-->
   <property>    
       <name>dfs.namenode.name.dir</name>    
       <value>file:///home/software/hadoop-2.7.1/tmp/hdfs/name</value>    
   </property>    
   <!--配置datanode存放元数据的目录，可以不配置，如果不配置则默认放到hadoop.tmp.dir下-->
   <property>    
       <name>dfs.datanode.data.dir</name>    
       <value>file:///home/software/hadoop-2.7.1/tmp/hdfs/data</value>    
   </property>
   <!--配置副本数量-->    
   <property>    
       <name>dfs.replication</name>    
       <value>3</value>    
   </property>   
   <!--设置用户的操作权限，false表示关闭权限验证，任何用户都可以操作-->                                                                    
   <property>    
       <name>dfs.permissions</name>    
       <value>false</value>    
   </property>  
   ```

3. 编辑mapred-site.xml

   ```xml
   <property>    
       <name>mapreduce.framework.name</name>    
       <value>yarn</value>    
   </property> 
   ```

4. 编辑yarn-site.xml

   ```xml
   <!--配置yarn的高可用-->
   <property>
       <name>yarn.resourcemanager.ha.enabled</name>
       <value>true</value>
   </property>
   <!--指定两个resourcemaneger的名称-->
   <property>
       <name>yarn.resourcemanager.ha.rm-ids</name>
       <value>rm1,rm2</value>
   </property>
   <!--配置rm1的主机-->
   <property>
       <name>yarn.resourcemanager.hostname.rm1</name>
       <value>hadoop01</value>
   </property>
   <!--配置rm2的主机-->
   <property>
       <name>yarn.resourcemanager.hostname.rm2</name>
       <value>hadoop03</value>
   </property>
   <!--开启yarn恢复机制-->
   <property>
       <name>yarn.resourcemanager.recovery.enabled</name>
       <value>true</value>
   </property>
   <!--执行rm恢复机制实现类-->
   <property>
       <name>yarn.resourcemanager.store.class</name>
       <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
   </property>
   <!--配置zookeeper的地址-->
   <property>
       <name>yarn.resourcemanager.zk-address</name>
       <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
   </property>
   <!--执行yarn集群的别名-->
   <property>
       <name>yarn.resourcemanager.cluster-id</name>
       <value>ns-yarn</value>
   </property>
   <!-- 指定nodemanager启动时加载server的方式为shuffle server -->
   <property>    
       <name>yarn.nodemanager.aux-services</name>    
       <value>mapreduce_shuffle</value>    
   </property>  
   <!-- 指定resourcemanager地址 -->
   <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>hadoop03</value>
   </property>
   ```

   

   # 常见参数


   | **配置所在文件** | **参数**                                                     | **参数默认值**                  | **作用**                                                     |
   | ---------------- | ------------------------------------------------------------ | ------------------------------- | ------------------------------------------------------------ |
   | hdfs-site.xml    | dfs.namenode.support.allow.format                            | true                            | 表示设置NameNode是否允许被格式化。  在生产系统，把它设置为false，阻止任何格式化操作在一个运行的DFS上。  建议初次格式化后，修改配置禁止，改成false |
   | hdfs-site.xml    | dfs.heartbeat.interval                                       | 3                               | DataNode的心跳间隔，默认单位为秒  在集群网络通信状态不好的时候，适当调大 |
   | hdfs-site.xml    | dfs.blocksize                                                | 134217728                       | 块大小，默认是128MB  必须得是1024(page size)的整数倍         |
   | hdfs-site.xml    | dfs.namenode.checkpoint.period  或者：  fs.checkpoint.period | 3600                            | edits和fsimage文件合并周期阈值，默认单位为s                  |
   | hdfs-site.xml    | dfs.stream-buffer-size                                       | 4096                            | 文件流缓存大小。需要是硬件page大小的整数倍。在读写操作时，数据缓存大小。  注意：是1024的整数倍  注意和core-default.xml中指定文件类型的缓存是不同的，这个是dfs共用的 |
   | mapred-site.xml  | mapreduce.task.io.sort.mb                                    | 100                             | 任务内部排序缓冲区大小，默认单位是MB  此参数调大，能够减少Spil溢写次数，减少磁盘I/O     建议：250MB~400MB |
   | mapred-site.xml  | mapreduce.map.sort.spill.percent                             | 0.8                             | Map阶段溢写文件的阈值。不建议修改此值                        |
   | mapred-site.xml  | mapreduce.reduce.shuffle.parallelcopies                      | 5                               | ReduceTask  启动的并发拷贝数据的线程数（fetch线程数）  建议，尽可能等于或接近于Map任务数量，达到并行抓取的效果 |
   | mapred-site.xml  | mapreduce.job.reduce.slowstart.completedmaps                 | 0.05                            | 当Map任务数量完成率在5%时，Reduce任务启动，这个参数建议不要轻易改动，如果Map任务总量非常大时，可以将此参数调低，让reduce更早开始工作。 |
   | mapred-site.xml  | io.sort.factor                                               | 10                              | 文件合并（Merge）因子，如果文件数量太多，可以适当调大，从而减少I/O次数 |
   | mapred-site.xml  | mapred.**compress**.map.output                               | false                           | 是否对Map的输出结果文件进行压缩，默认是不压缩。但是如果Map的结果文件很大，可以开启压缩，在Reduce的远程拷贝阶段可以节省网络带宽。 |
   | mapred-site.xml  | **mapred.map.tasks.speculative.execution**                   | true  <br />==一般设置为false== | 启动map任务的推测执行机制       推测执行是Hadoop对“拖后腿”的任务的一种优化机制，当一个作业的某些任务运行速度明显慢于同作业的其他任务时，Hadoop会在另一个节点上为“慢任务”启动一个备份任务，这样两个任务同时处理一份数据，而Hadoop最终会将优先完成的那个任务的结果作为最终结果，并将另一个任务杀掉。     启动推测执行机制的目的是更快的完成job，     但是在集群计算资源紧张时，比如同时在运行很多个job，启动推测机制可能会带来相反效果。如果是这样，就改成false。     对于这个参数的控制，轻易不要改动。 |
   | mapred-site.xml  | mapred.reduce.tasks.speculative.execution                    | true<br />==一般设置为false==   | 启动reduce任务的推测执行机制                                 |

   

   



# 概述

1. 全称为Hadoop     Distributed File System ，是Hadoop提供的一个==**分布式的，可扩展，可靠的**==文件系统
2. HDFS是根据谷歌的论文：《The Google File System》进行设计的
3. 本身HDFS中包含三个主要的进程：`NameNode`，`DataNode`，`SecondaryNameNode`。这三个进程一般是分布式不同的主机上，所以一般习惯上是用进程的名字称呼节点



# 特点

## 优点

1. ==支持超大文件==。超大文件在这里指的是几百M，几百GB，甚至几TB大小的文件。一般来说Hadoop的文件系统会存储TB级别或者PB级别的数据。所以在企业的应用中，数据节点有可能有上千个
2. ==检测和快速应对硬件故障==。在集群的环境中，硬件故障是常见的问题。因为有上千台服务器连接在一起，这样会导致高故障率。因此故障检测和自动恢复(心跳机制)是HDFS文件系统的一个设计目标
3. ==流式数据访问==。HDFS的数据处理规模比较大，应用一次需要访问大量的数据，同时这些应用一般都是批量处理，而不是用户交互式处理。应用程序能以流的形式访问数据集。主要的是数据的吞吐量，而不是访问速度
4. ==简化的一致性模型==。大部分hdfs操作文件时，需要一次写入，多次读取。在HDFS中，一个文件一旦经过创建、写入、关闭后，一般就不需要修改了。这样简单的一致性模型，有利于提高吞吐量（可以追加）
5. ==高容错性==。数据自动保存多个副本，副本丢失后自动恢复
6. ==可构建在廉价机器上==。构建在廉价机器上可以轻松的通过扩展机器数量来近乎线性的提高集群存储能力

## 缺点

1. ==不能低延迟数据访问==。如和用户进行交互的应用，需要数据在毫秒或秒的范围内得到响应。由于Hadoop针对海量数据的吞吐量做了优化，牺牲了获取数据的延迟，所以对于低延迟来说，不适合用hadoop来做
2. ==不适合存储大量的小文件==。HDFS支持超大的文件，是通过数据分布在数据节点，数据的元数据保存在名字节点上。名字节点的内存大小，决定了HDFS文件系统可保存的文件数量。虽然现在的系统内存都比较大，但大量的小文件还是会影响名字节点的性能
3. ==不支持多用户写入、修改文件==。HDFS的文件只能有一次写入，不支持修改和追加写入（2.0版本支持追加），也不支持修改。只有这样数据的吞吐量才能大
4. ==不支持超强的事务==。没有像关系型数据库那样，对事务有强有力的支持



# 基本结构

1. HDFS中基本架构是由一个NameNode+多个DataNode来组成的
2. 在HDFS中存储数据的时候，会对文件进行切块，切块之后会将不同的数据块（Block）放到不同的节点上
3. 在HDFS中，默认会对每一个Block进行备份，备份称之为副本（replicas）。在HDFS中，连同原来的Block构成3个副本
4. HDFS仿照Linux设计了一套文件存储系统，所以和Linux一样，是以`/`为根路径，每一个文件都存在权限问题



![](https://note.youdao.com/yws/api/personal/file/DAFE71577648405993EEDC166C2FD61F?method=download&shareKey=71c6bb094f8b6d89c0a622f025dfcdf6)



# Block

1. 数据块（Block）是HDFS存储数据的基本单位

2. 当在DHFS上存储超大文件时，HDFS会以一个标准将文件切分成几块，分别存储到不同的节点上，切出的数据称为Block

   > 每一个Block的大小默认不超过128MB
   >
   > 假设一个文件是1GB，那么这个文件会被切成8个Block
   >
   > - 这个Block的大小可以通过`dfs.blocksize`属性来设置
   >
   > - 单位是字节。
   >
   > - 配置文件的位置：`/home/ssoftware/hadoop-2.7.1/etc/hadoop/hdfs-site.xml`

3. 如果文件大小不足128MB，那么这个文件是多大，所对应的的Block就是多大。

   > 例如：一个文件是20MB，那么这个文件对应的Block大小就是20MB
   >
   > 如果文件是150MB，那么这个文件对应了2个Block：128MB+22MB

4. 每一个Block分配一个BlockID

**切块的意义**：

- 能够存储超大文件
- 能够快速备份



# NameNode

> NameNode负责管理==DataNode==和记录==元数据==

## 元数据

1. 元数据：描述数据的数据

   - 文件在HDFS的存储位置
   - 上传的用户及用户组
   - 文件的权限
   - 文件的大小
   - Block的大小
   - 文件和BlockID的映射关系
   - 副本数量
   - Block和DataNode的映射

2. NameNode将元数据维系在==内存==和==硬盘==中

   - 维系在内存中的目的是为了查询快
   - 维系在磁盘中的目的是为了崩溃恢复

3. 元数据在磁盘中的存储位置由：`hadoop.tmp.dir`来决定，如果不配置，那么默认放在`/tmp`下

4. 元数据的存储和edits以及fsimage文件相关

   - edits：记录==写操作==的文件
   - fsimage：记录==元数据==的文件，但是这个文件的元数据和内存中的元数据不是同步的。fsimage中的元数据往往是落后于内存中的元数据

5. 当NameNode收到请求之后，NameNode先将这个请求记录到`edits_inprogress`中，记录成功之后修改内存中的元数据，内存中的元数据修改成功之后会给客户端返回一个ACK，表示成功。这个过程不会涉及到fsimage文件的修改

   ![](https://note.youdao.com/yws/api/personal/file/8B7E424EC1A8401B99CA3BD21931E9A3?method=download&shareKey=ae43126b7f14b4a84d87eaf70bcbb985)

6. 当NameNode收到读请求后，直接查阅内存中的元数据，此时不涉及edits和fsimage

7. 存储元数据的目录：`dfs/name/current`

## 持久化文件：edits和fsimage

1. 在达到合适条件的时候，触发`edits`文件的滚动和`fsimage`文件的更新：

   - `edits_inprogress`会滚动生成`edits`文件
   - 同时这个过程中更新`fsimage`中的元数据
   - 产生一个新的`edits_inprogress`

   ![](https://note.youdao.com/yws/api/personal/file/C3ABC7FF875E42BE9F539304E16625F0?method=download&shareKey=2962db523d97426d11ac853d2cba0a28)

2. edits_inprogress文件的滚动条件

   - ==空间==：当`edits_inprogress`文件达到指定大小时，会产生滚动

     > 默认是64MB
     >
     > 这个大小可以通过`fs.checkpoint.size`来调节
     >
     > 单位是字节
     >
     > 这个属性是放在`core-site.xml`中的

   - ==时间==：当距离上一次更新达到指定时间间隔之后，会产生滚动

     > 默认是3600s（1天）
     >
     > 这个大小可以通过`fs.checkpoint.period`来调节
     >
     > 单位是s（秒）
     >
     > 这个属性是放在`core-site.xml`中的

   - 重启：当HDFS/NameNode重启的时候，触发`edits_inprogress`文件的滚动

   - 强制：`Hadoop dfsadmin -rollEdits`

## NameNode

1. NameNode通过心跳机制来管理DataNode

2. DataNode会定时给NameNode发送心跳信息

   > 默认是3s
   >
   > 这个值可以通过`dfs.heartbeat.interval`来修改
   >
   > 这个属性是配置在`hdfs-site.xml`中的

3. 心跳机制通过RPC来实现的

4. 当NameNode超过10min没有收到DataNode的心跳，NameNode就会认为这个DataNode已经lost（丢失），那么NameNode就会将这个DataNode上的数据备份到其他节点上从而保证整个集群中的副本数量

5. 心跳信息：

   - clusterid（集群编号）：

     > 当HDFS搭建好后，对NameNode进行格式化（Hadoop namenode -format）,格式化的时候自动计算产生一个clusterid。
     >
     > 每次格式化都会重新计算这个clusterid。
     >
     > 当HDFS集群启动之后，NameNode将这个clusterid分发给每一个DataNode，并且要求DataNode发送的每一条心跳中都携带这个clusterid
     >
     > NameNode在收到DataNode的信息之后也会先校验这个clusterid是否一致

   - 当前节点状态：预服役、服役、预退役

     ![](https://note.youdao.com/yws/api/personal/file/7D4943275CA94B98A576F4A2A89EDE7B?method=download&shareKey=d88c2f2785a4a9fc78f5609c9b9d1112)

   - 当前节点存储的Block的BlockID

6. 在Hadoop2.0的集群中:

   - 如果存在`SecondaryNameNode`，则更新过程需要`SecondaryNameNode`参与
   - 如果不存在`SecondaryNameNode`，则更新过程发生在`NameNode`上

7. 无论是Hadoop1.0还是2.0，当HDFS启动NameNode会做一次edits和fsimage合并的操作。这样做的目的是确保fsimage里的元数据更新

8. 当HDFS启动的时候，NameNode会将fsimage中的元数据信息加载到内存中，然后等待每个DataNode向NameNode汇报自身的存储的信息，比如存储了哪些文件块，块大小，块id等。NameNode收到这些信息之后，会做汇总和检测，检测数据是否完整，副本数量是否达到要求，如果检测出现问题，HDFS会进入安全模式，在安全模式做数据或副本的复制，直到修复完成后，安全模式自动退出

9. 如果HDFS处于安全模式，只能对外读服务，不能写服务

10. 在Hadoop1.0中，NameNode存在单点故障问题。在Hadoop2.0的伪分布式中，NameNode依然存在单点故障，但是在Hadoop2.0的全分布式中，可以通过舍弃SecondaryNameNode的方式来配置2个NameNode来保证NameNode的高可用





# SecondaryNameNode

1. SecondaryNameNode不是NameNode的备份，仅仅是辅助NameNode来进行edits文件的滚动和fsimage文件的更新的

2. edits_inprogress文件的滚动：

   - 如果存在SecondaryNameNode，那么edits_inprogress文件的滚动过程是发生在SecondaryNameNode中
   - 如果没有SecondaryNameNode，那么edits_inprogress文件的滚动就发生在NameNode中

3. 到目前为止，HDFS的完全分布式只支持：

   - NameNode+SecondaryNameNode+多个DataNode结构
   - 或者双NameNode+多个DataNode结构

   > 因为在HDFS集群中，NameNode是一个核心节点，如果NameNode只有一个，容易出现单点故障，所有NameNode 必须存在一个备份节点（NameNode做到高可用），就考虑使用：双NameNode+多个DataNode



# DataNode

1. DataNode的职责是用于存储Block
2. 存储Block的位置由`hadoop.tmp.dir`属性来决定
3. DataNode会定时向NameNode发送心跳信息
4. DataNode的节点状态
   - 预服役
   - 服役
   - 预退役
   - 退役





# 副本放置策略

1. 第一个副本：
   - 集群内部上传：谁上传，第一个副本就存在谁身上
   - 集群外部上传：NameNode选择相对比较空闲的节点存放数据
2. 第二个副本：
   - 在Hadoop 2.7之前：放在和第一个副本不同机架的节点上
   - 在Hadoop 2.7开始：放在和第一个副本相同机架的节点上
3. 第三个副本：
   - 在Hadoop 2.7之前：放在和第二个副本相同机架的节点上
   - 在Hadoop 2.7开始：放在和第二个副本不同机架的节点上
4. 更多副本：选择相对空闲的节点放入

# 机架感知策略

1. HDFS中所谓的机架指的并不是物理机架，而是逻辑机架，这个逻辑机架本质上就是一个映射
2. 在实际过程中，可以通过Shell/Python脚本来指定机架，实际上就是制定一个Map，在这个Map中
   - 键：节点主机名/ip地址
   - 值：机架编号
3. 习惯上，会将同一个物理机架或者是同一个用户组内的节点配置在同一个逻辑机架上



# HDFS基本命令

| 命令                                                      | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| hadoop fs -mkdir /park                                    | 在hdfs 的根目录下，创建 park目录                             |
| hadoop fs   -ls /                                         | 查看hdfs根目录下有哪些目录                                   |
| hadoop fs -put   /root/1.txt /park                        | 将linux操作系统root目录下的1.txt放在hdfs的park目录下         |
| hadoop fs -get   /park/jdk /home                          | 把hdfs文件系统下park目录的文件下载到linux的home目录下        |
| hadoop  fs -rm /park/文件名                               | 删除hdfs 的park目录的指定文件                                |
| hadoop fs  -rmdir /park                                   | 删除park目录，但是前提目录里没有文件                         |
| hadoop fs  -rmr /park                                     | 删除park目录，即使目录里有文件                               |
| hadoop fs  -cat /park/a.txt                               | 查看park目录下的a.txt文件                                    |
| hadoop fs  -tail /park/a.txt                              | 查看park目录下a.txt文件末尾的数据                            |
| haddop  jar xxx.jar                                       | 执行jar包                                                    |
| hadoop fs -cat  /park/result/part-r-00000                 | 查看 /park/result/part-r-00000文件的内容                     |
| hadoop   fs  –mv  /park02 /park01                         | 将HDFS上的park02目录重名为park01命令。                       |
| hadoop fs -mv /park02/1.txt /park01                       | 将park02目录下的1.txt移动到/park01目录下                     |
| hadoop fs  -touchz /park/2.txt                            | 创建一个空文件                                               |
| hadoop fs  -getmerge /park /root/tmp                      | 将park目录下的所有文件合并成一个文件，并下载到linux的root目录下的tmp目录 |
| hadoop dfsadmin  -safemode leave                          | 离开hadoop安全模式                                           |
| hadoop  dfsadmin -safemode enter                          | 进入安全模式                                                 |
| hadoop  dfsadmin -rollEdits                               | 手动执行fsimage文件和Edis文件合并元数据                      |
| hadoop  dfsadmin -report                                  | 查看存活的datanode节点信息                                   |
| hadoop  fsck /park                                        | 汇报/park目录 健康状况                                       |
| hadoop  fsck /park/1.txt -files -blocks -locations -racks | 查看1.txt 这个文件block信息以及机架信息        ![](https://note.youdao.com/yws/api/personal/file/64FAC148DCAE4B9C8590FCEE28DD08F9?method=download&shareKey=59ad7e24b57c59a52a965d4d71c2f2cc) |
| hadoop fs -expunge                                        | 手动清空hdfs回收站                                           |



# 其他命令

| 命令                                              | 说明                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| hadoop fs -cp   /park01/1.txt /park02             | 将HDFS上 /park01下的1.txt拷贝一份到 /park02目录下。     <br />目标路径可以有多个，用空格隔开，比如：hadoop  fs -cp /park01/1.txt /park02 /park03…… |
| hadoop fs  -du /park/1.txt                        | 查看HDFS上某个文件的大小。也可以查看指定目录，如果是目录的话，则列出目录下所有的文件及其大小，比如：hadoop fs -du /park |
| ~~hadoop fs  -copyFromLocal /home/1.txt /park01~~ | 将本地文件1.txt上传到/park01目录下                           |
| ~~hadoop fs  -copyToLocal /park01/1.txt /home~~   | 将HDFS上的1.txt 拷贝到本地文件系统                           |
| hadoop fs -lsr   /                                | 递归查看指定目录下的所有内容                                 |





# 一、概述

1. 在HDFS中，回收站机制默认是关闭的，即从HDFS上删除文件的时候是立即删除的
2. 可以通过配置来手动开启回收站，指定被删除文件的保留时间

 

# 二、配置

1. 进入Hadoop的安装目录下的子目录etc/hadoop：

   `cd hadoop-2.7.1/etc/hadoop`

2. 配置core-site.xml

   编辑core-site.xml：`vim core-site.xml`

   添加如下内容：

   ```xml
   <property>
       <name>fs.trash.interval</name>
       <value>1440</value>
   </property>
   ```

   保存退出



 

# 三、注意事项

1. 回收站的配置中，value的时间单位是分钟。如果配置程0，则表示不开启HDFS的回收站机制。

2. 配置为1440表示回收间隔为1天，即文件在回收站存在1天后，会被清除

3. 当启动回收站后，再删除文件时：

   ![](https://note.youdao.com/yws/api/personal/file/109A64776AA043D28E0A0A70B44B1790?method=download&shareKey=19ace0871c18629e27c8c89ec11c9570)

4. 可以通过递归查看指令，查看回收站中的文件：

   `hadoop fs -lsr /user/root/.Trash`

   ![](https://note.youdao.com/yws/api/personal/file/445CB040F9FE443A8073160C4785C225?method=download&shareKey=5a5a72499ede4f7932e219041656d3cc)

5. 如果想恢复被删除的文件，执行hdfs 的mv 指令即可

# 概述

1. dfs目录表示HDFS的存储目录

   - `dfs/name`表示NameNode的持久化目录
    - `dfs/data`表示DataNode的存储目录
    - `dfs/namesecondary`表示SecondaryNameNode的存储目录

2. 实际过程中，由于NameNode、DataNode以及SecondaryNameNode应该分布在不同的节点上，所以name、data、nameseconda三个目录也应该出现在不同的节点上

3. 当执行格式化指令时，会在指定的数据目录下，生成`dfs/name`目录

   ![](https://note.youdao.com/yws/api/personal/file/A5C555C79B3F4C68B5B72554E50A0118?method=download&shareKey=bcd0399b62b08223893d1c9e8f0cf64a)

4. 当格式化后，启动HDFS前，会生成一个最初的`fsimage_0000000000000000000`文件，该文件中存储的根节点的信息

5. `dfs/name/in_use.lock`文件的作用是防止在同一台服务器上启动多个NameNode，避免管理紊乱

   ![](https://note.youdao.com/yws/api/personal/file/585C69008B81432E86D2A8F514DF22E2?method=download&shareKey=f494d83721d416d42f990cb6074ec291)

6. 当启动HDFS时，会生成edits文件

7. HDFS中有事务id的概念，当HDFS每接收一个写操作（比如：mkdir put mv），都会分配全局递增的事务id(txid)，然后写到edits文件中

8. 每生成一个新的edits文件

   - edits文件中都会以`OP_START_LOG_SEGMENT`开头
   - 当一个edits文件写完后，会以`OP_END_LOG_SEGMENT`结尾。

   > 即在`OP_START_LOG_SEGMENT`——`OP_END_LOG_SEGMENT`存储的是这个edits文件所有的事务记录

9. `edits_inprogress`文件的作用是记录当前正在执行的事务文件。后面的编号是以上一次`Txid+1`来命名的

10. 初次使用HDFS时，有一个默认的`edits`和`fsimage`的合并周期（==1分钟==），以后在使用HDFS的过程中，达到条件`edits_inprogress会和fsimage`进行合并

    > 条件是自定义的，比如3600s合并一次

11. 上传文件底层会拆分成如下的事务过程：

    - `OP_ADD` 将文件加入到指定的HDFS目录下，并以._Copyging_结尾，表示此文件还未写完
    - `ALLOCATE_BLOCK_ID`：为文件分配块ID
    - `SET_GENSTAMP_V2`：为块生成时间戳版本号，是全局唯一的
    - `ADD_BLOCK`：写块数据
    - `OP_CLOSE`：表示块数据写完
    - `OP_RENAME_OLD`：将文件重命名，表示写完

12. 当停止HDFS后再次启动HDFS时，当执行一个事务之后，底层会触发一次合并，然后生成一个新的edits文件

13. `seen_txid`记录是的最新的`edits_inprogress`文件末尾的数字

14. `VERSION`文件中的重要内容：

    - clusterID：集群编号
    - storageType：节点类型
    - blockpollID：块池编号

15. `fsimage_N`文件存储的N号事务前的所有的元数据信息

16. `fsimage_N.md5`存储的是fsimage文件的md5校验码，可以通过MD5的校验和来校验文件的完整性：`md5sum fsimage_n`



# edits文件和fsimage文件

1. 查看edits文件：`hdfs oev -i edits文件 -o xxx.xml`。例如：
   - `hdfs oev -i edits_0000000000000000001-0000000000000000003 -o edits.xml`
2. 查看fsimage文件：`hdfs oiv -i fsimage文件 -o xxx.xml -p XML`。例如：
   - `hdfs oiv -i fsimage_0000000000000000012 -o fsimage.xml -p XML`
3. fsimage介绍：fsimage是一个二进制文件，当中记录了HDFS中所有文件和目录的元数据信息。



[TOC]



# 读取/下载流程

![](https://note.youdao.com/yws/api/personal/file/11C95D01C50F4917957A5BC4750CCDBA?method=download&shareKey=6e26da8057859783962e243b6094ebde)

1. 客户端发起RPC请求到NameNode

2. NameNode收到请求之后，查询元数据

   - 如果查询成功，给客户端返回信号，表示允许读

3. 客户端收到信号之后，向NameNode再次发送请求，请求获取文件第一个Block的存储位置

4. NameNode收到请求之后，会将这个Block的存储位置放到一个队列中返回给客户端

   > 默认情况下，副本数量为3
   >
   > 所以放到队列中的Block位置是3个

5. 客户端收到队列之后，会将这个Block的位置从队列中全部取出，从这些位置中选取一个比较近的DataNode来进行读取。读取`Block`以及对应的`.meta`

   > `.meta`对当前Block的描述，例如这个Block的产生时间、Block的大小等
   >
   > 距离：网络拓扑距离

6. 读取完成之后，会对Block的大小进行校验

   - 如果校验失败，那么客户端就会通知NameNode，同时客户端重新选取地址重新读取
   - 如果检验成功，那么客户端会继续向NameNode发送请求要下一个Block的地址，重复4、5、6步骤

7. 当客户端读取网所有的Block之后，客户端就会给NameNode发送信号要求关闭文件（关流）



# 写入/上传流程

![](https://note.youdao.com/yws/api/personal/file/8A365C4DF35748BFA92BD0AE20127C17?method=download&shareKey=a42a689b1886bf03aab017ad08f83c6d)

1. 客户端发起RPC请求到NameNode
2. NameNode收到请求之后，先进行校验：
   - 校验是否有权限操作要写入的路径
   - 校验这个路径下是否有同名文件
3. 校验通过，NameNode给客户端返回一个信号，表示允许写入
4. 客户端收到信号之后，就向NameNode发送请求，请求获取第一个Block的写入位置
5. NameNode收到请求之后，将位置放到一个队列中，返回给客户端
6. 客户端收到队列之后
   - 从队列中将全部的位置取出，从中选择一个较近的节点，写入这个Block的第一个副本。
   - 同时客户端告诉这个节点第二个和第三个副本的存储位置。当前节点通过pipeline（管道，本质上是一个Channel），将第二个副本写到对应节点上
   - 第二个副本所在的节点再通过管道将第三个副本数据写到第三个节点上。
     - 等第三个副本写完之后，会给第二个副本所在的节点返回一个ACK表示写入成功
     - 第二个副本所在节点收到ACK之后，会给第一个副本所在的节点返回一个ACK表示写入成功
     - 第一个副本所在节点收到ACK之后，给客户端返回一个ACK表示数据完全写入成功
7. 客户端收到ACK之后，就会向NameNode要下一个Block的位置，重复5、6、7三个步骤
8. 当客户端将所有的Block写完之后，通知NameNode关闭文件（关流）





# 删除流程

1. 客户端发起RPC请求到NameNode

2. NameNode收到请求之后，会进行校验：

   - 校验是否有这个文件
   - 校验是否有删除权限

3. 校验通过，NameNode就会将这个写操作记录到`edits_inprogress`中，同时更改内存中的元数据。元数据更改完成之后，NameNode就会给客户端返回一个ACK表示删除成功

   > 这个过程中，仅仅是元数据作了修改，这个文件对应的Block并没有被删除，也就意味着，客户端收到AKC不代表文件已经删除

4. 当NameNode收到DataNode的心跳的时候，会校验心跳信息

   - 如果发现心跳信息和元数据不一致，那么NameNode就会进行心跳响应，要求DataNode删除对应的Block

5. 当DataNode收到心跳响应之后，才会删除Block，此时这个文件才算真正从HDFS上移除

[TOC]

# 准备pom.xml

> 需要导入HDFS的依赖jar包

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.7.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.7.1</version>
    </dependency>
</dependencies>

```





# API操作

## 读取/下载文件

> 连接HDFS需要提供URI和Configuration
>
> - uri：创建一个URI对象指向连接地址
>   - 50070端口：HDFS提供的HTTP访问端口
>   - 9000端口：配置HDFS的RPC访问端口
> - configuration：配置对象

```java
// 下载文件
@Test
public void get() throws IOException {

    // 创建一个URI对象来指向连接地址
    // 50070 这个HDFS提供的HTTP访问端口
    // 9000 配置的HDFS的RPC访问端口
    URI uri = URI.create("hdfs://10.9.162.133:9000");
    Configuration conf = new Configuration();
    // 连接HDFS
    FileSystem fs = FileSystem.get(uri, conf);
    // 打开文件
    // 返回一个输入流
    InputStream in = fs.open(new Path("/a.txt"));
    // 创建一个输出流
    FileOutputStream out = new FileOutputStream("D:\\a.txt");
    // 利用输入流来读取数据，然后将读取的数据利用输出流写出
    IOUtils.copyBytes(in, out, conf);
    // 关流
    in.close();
    out.close();

}
```



## 写入/上传文件

```java
// 上传文件
@Test
public void put() throws IOException, InterruptedException {

    // 连接HDFS
    URI uri = URI.create("hdfs://10.9.162.133:9000");
    // 放到配置文件中的内容，都可以通过这个参数配置
    Configuration conf = new Configuration();
    conf.set("dfs.replication", "1");
    // conf.set("dfs.blocksize", "");
    // 权限验证：所知即所得
    FileSystem fs = FileSystem.get(uri, conf, "root");
    // 创建文件
    OutputStream out = fs.create(new Path("/log/b.log"));
    // 创建一个输入流指向要上传的文件
    FileInputStream in = new FileInputStream("D:\\a.txt");
    // 数据传输
    IOUtils.copyBytes(in, out, conf);
    // 关流
    in.close();
    out.close();

}
```



## 删除文件

```java
// 删除文件
@Test
public void delete() throws IOException, InterruptedException {

    // 连接HDFS
    URI uri = URI.create("hdfs://10.9.162.133:9000");
    Configuration conf = new Configuration();
    FileSystem fs = FileSystem.get(uri, conf, "root");
    // 删除文件
    fs.delete(new Path("/log"), true);
}
```



## 在HDFS上创建文件夹

## 查询HDFS指定目录下的文件

## 递归查看指定目录下的文件

## 重命名

## 获取文件的块信息

[TOC]

# 概述

1. MapReduce是Hadoop提供的一套用于==分布式计算==的框架

2. MapReduce是Doug根据Google的论文《The Google MapReduce》阐述的原来来实现的

3. MapReduce会将计算过程拆分成2个阶段：

   - Map（映射）阶段
   - Reduce（规约）阶段

   ![](https://note.youdao.com/yws/api/personal/file/19844CDA90CC450CBCE9C8D9AF8AA50A?method=download&shareKey=fea94506e953299da6dc7eec1770c128)

# 案例

> 字符统计
>
> 数据文件：characters.txt
>
> **统计文件中每个字符出现的次数**



## Mapper类

### 继承Mapper类

> CharCountMapper，需要继承hadoop.mapreduce.Mapper
>
> ==MapReduce中要求被传输的数据必须能够被序列化==，所以4个泛型必须是支持序列化的

 - KEYIN - 输入的键的类型。

   > 在默认情况下，表示的是读取的这一行的字节偏移量。 `LongWritable`是MapReduce针对`long`类型提供的序列化形式
   >
   > ![](https://note.youdao.com/yws/api/personal/file/B13E61AE0D9E4CDDB014C4D140B191C9?method=download&shareKey=06c31ee859d671d469baabd97bf21d5e)

 - VALUEIN - 输入的值的类型。

   > 在默认情况下，表示的就是读取的这一行数据。`Text`是MapReduce针对`String`类型提供的序列化形式

 - KEYOUT - 输出的键的类型。

   > 当前案例是计算每一个字符出现的次数，所以输出的键应该是字符

 - VALUEOUT - 输出的值的类型。

   > 当前案例是计算每一个字符出现的次数，所以输出的值应该是次数



### 重写`map()`方法

> MapTask的处理逻辑是放在map()方法中的。每读取一行，调用一次`map()`方法。`map()`方法参数：
>
> - `key`：字节偏移量
> - `value`：读取的这一行数据
> - `context`：环境参数，作用之一是将数据从Map写到Reduce



```java
//传入4个泛型
public class CharCountMapper extends Mapper<LongWritable, Text,Text,LongWritable> {

    //MapTask处理逻辑放在map()方法中的
    //每读取一行就会调用一次map()方法
    /*
        key     -   字节偏移量
        value   -   读取的这一行数据
        context -   环境参数，作用之一是将数据从Map写到Reduce
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //value表示的是一行数据
        //统计这一行中每一个字符出现的次数
        //现将这一行中的每一个字符拆分出来
        //hello -   {h,e,l,l,o}
        char[] cs = value.toString().toCharArray();
        for (char c : cs) {
            context.write(new Text(c+""),new LongWritable(1));
        }
    }
}
```



## Reducer类

### 继承Reducer

> 需要继承hadoop.mapreduce.Reducer

需要传入4个泛型

- `KEYIN`,`VALUEIN` - 输入的键值类型  -  Reduce的数据从Map传来的
- `KEYOUT`,`VALUEOUT` - 输出的键值类型 - 计算每一个字符出现的次数，所以输出的是字符:次数

### 重写`reduce()`方法

> 计算逻辑写在`reduce()`方法中，三个参数：
>
> - key：从Map传来的key值
> - values：这个键对应的是所有的值
> - context：环境参数，此处作用是将结果最终写会到HDFS上

```java
// KEYIN,VALUEIN - 输入的键值类型  - Reduce的数据从Map来的
// KEYOUT,VALUEOUT - 输出的键值类型 - 计算每一个字符出现的次数，所以输出的是字符:次数
public class CharCountReducer extends Reducer<Text, LongWritable, Text, LongWritable> {
    // key - 输入的键 - 当前案例中表示的是字符。每一个键调用一次reduce方法
    // values - 这个键所对应的所有的值
    // context - 环境参数。此处的作用是将结果最终写回到HDFS上
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        // 记录总次数
        long sum = 0;
        // key:h
        // values:1,1,1,1,1...
        // 迭代遍历这个values
        for (LongWritable val : values) {
            sum += val.get();
        }
        context.write(key, new LongWritable(sum));
    }
}
```



## Driver类

> 此类是入口类，main方法所在的类
>
> 1. 申请任务
> 2. 设置Mapper类
> 3. 设置Reducer类
> 4. 设置Mapper的输出类型（==如果输出类型和Reducer相同，只写Reducer的就行==）
> 5. 设置Reducer的输出类型
> 6. 设置输入路径
> 7. 设置输出路径
> 8. 提交任务

```java
public class CharCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        // 申请任务
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 设置入口类
        job.setJarByClass(CharCountDriver.class);
        // 设置Mapper类
        job.setMapperClass(CharCountMapper.class);
        // 设置Reducer类
        job.setReducerClass(CharCountReducer.class);

        // 设置Mapper的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        // 设置Reducer的输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        // 设置输入路径
        FileInputFormat.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/characters.txt"));
        // 设置输出路径
        // 要求输出路径在HDFS上不存在
        FileOutputFormat.setOutputPath(job, new Path("hdfs://hadoop01:9000/result/charcount"));

        // 提交任务
        job.waitForCompletion(true);
    }

}
```



## 结果

> 会在指定的路径下生成两个文件
>
> - `_SUCCESS`：此文件夹是空的
> - `part-r-00000`：数据统计的结果存在该文件中

```
 	1430
!	3
"	6
'	7
,	59
-	2
.	66
:	1
;	1
?	1
A	20
B	3
C	3
D	1
E	1
F	1
G	5
I	23
L	5
M	4
N	21
O	3
P	2
S	4
T	11
W	15
Y	2
a	480
b	99
c	164
d	226
e	760
f	180
g	137
h	348
i	476
j	19
k	46
l	282
m	148
n	388
o	520
p	77
q	6
r	343
s	370
t	588
u	163
v	71
w	132
x	5
y	109
z	6
```



# 练习

## 单词统计

> 源文件：words.txt

```
hello tom hello bob
hello joy
hello rose
hello joy
hello jerry
hello tom
hello rose
hello joy
```

### Mapper类

> `WordCountMapper`

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordCountMapper extends Mapper<LongWritable, Text,Text,LongWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        String[] s = value.toString().split(" ");
        for (String s1 : s) {
            context.write(new Text(s1),new LongWritable(1));
        }
    }
}
```



### Reducer类

> `WordCountReducer`

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordCountReducer extends Reducer<Text, LongWritable,Text,LongWritable> {

    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        long sum = 0;
        for (LongWritable value : values) {
            sum += value.get();
        }
        context.write(key,new LongWritable(sum));
    }
}
```



### Driver类

> `WordCountDriver`

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        //申请任务
        Job job = Job.getInstance(new Configuration());

        //设置启动类入口
        job.setJarByClass(WordCountDriver.class);
        //设置Mapper类
        job.setMapperClass(WordCountMapper.class);
        //设置Reducer类
        job.setReducerClass(WordCountReducer.class);

        //设置Mapper输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        //设置Reducer输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        //设置处理的文件
        FileInputFormat.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/words.txt"));
        //设置处理结果输出路径
        FileOutputFormat.setOutputPath(job,new Path("hdfs://hadoop01:9000/result/wordcount"));

        //提交任务
        job.waitForCompletion(true);
    }
}

```



### 结果

```
bob		1
hello	9
jerry	1
joy		3
rose	2
tom		2
```



## IP去重

> 源文件：`ip.txt`

```
10.9.80.16
10.9.132.111
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.65
10.9.162.13
10.9.162.133
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.133
10.9.129.86
10.9.132.111
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.66
10.9.80.16
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.65
10.9.162.13
10.9.132.111
10.9.130.83
10.9.21.119
10.9.132.111
10.9.130.8
10.9.80.16
10.9.152.65
10.9.21.119
10.9.132.111
10.9.130.83
10.9.80.16
10.9.152.66
10.9.80.16
10.9.152.65
10.9.21.119
10.9.80.16
10.9.152.65
10.9.162.13
10.9.162.133
10.9.21.119
10.9.132.1
10.9.162.133
10.9.21.119
10.9.132.111
10.9.130.8
10.9.80.16
10.9.152.33
10.9.152.66
10.9.80.16
10.9.152.65
10.9.21.119
10.9.80.16
10.9.152.65
10.9.162.13
10.9.162.133
10.9.152.65
10.9.162.13
10.9.162.133
10.9.21.119
10.9.132.1
10.9.162.133
10.9.21.119
10.9.132.111
10.9.130.8
10.9.132.111
10.9.130.8
10.9.80.16
10.9.152.33
10.9.152.66
10.9.80.16
10.9.152.65
```



### Mapper类

```java
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

// 统计每一个IP出现的次数 --- 去重，不需要次数
public class IPMapper extends Mapper<LongWritable, Text, Text, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // value - 表示IP
        context.write(value, NullWritable.get());
    }
}
```



### Reducer类

```java
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class IPReducer extends Reducer<Text, NullWritable, Text, NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        // key:IP
        // values:null,null,null...
        context.write(key, NullWritable.get());
    }
}
```



### Driver类

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class IPDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(IPDriver.class);
        job.setMapperClass(IPMapper.class);
        job.setReducerClass(IPReducer.class);

        // 如果Mapper和Reducer输出类型一致，那么可以只写一个
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/ip.txt"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/ip"));
        job.waitForCompletion(true);
    }
}
```



### 结果

```
10.9.129.86
10.9.130.8
10.9.130.83
10.9.132.1
10.9.132.111
10.9.152.33
10.9.152.65
10.9.152.66
10.9.162.13
10.9.162.133
10.9.21.119
10.9.80.133
10.9.80.16
```



## 找出每个人的最高分

> 源文件：`score2.txt`

```
Bob 684
Alex 265
Grace 543
Henry 341
Adair 345
Chad 664
Colin 464
Eden 154
Grover 630
Bob 340
Alex 367
Grace 567
Henry 367
Adair 664
Chad 543
Colin 574
Eden 663
Grover 614
Bob 312
Alex 513
Grace 641
Henry 467
Adair 613
Chad 697
Colin 271
Eden 463
Grover 452
Bob 548
Alex 285
Grace 554
Henry 596
Adair 681
Chad 584
Colin 699
Eden 708
Grover 345
```

### Mapper类

```java
public class MaxScoreMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // Adair 345
        String[] arr = value.toString().split(" ");
        context.write(new Text(arr[0]), new IntWritable(Integer.parseInt(arr[1])));
    }
}
```



### Reducer类

```java
public class MaxScoreReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        IntWritable max = new IntWritable(0);
        // 在MapReduce为了节省内存空间，采取了地址复用机制
        // 在遍历迭代器的时候，迭代对象只创建一次
        // key:Bob
        // values:312 684 340 548
        // IntWritable val = new IntWritable();
        // val.set(312);
        // val.get() > max.get() -> 312 > 0 -> true
        // max = val; - val和max都是对象，所以赋值给的是地址 -> max和val的地址就一样了
        // val.set(684); - val和max的地址一样，那就意味着max中的值也变成了684
        // val.get() > max.get() -> 684 > 684 -> false
        // val.set(340); - max的值也跟着变成了340
        // 这么遍历下去，max的最终结果应该遍历的最后一个值
        for (IntWritable val : values) {
            if (val.get() > max.get())
                // max = val;
                max.set(val.get());
        }
        context.write(key, max);
    }
}

```



### Driver类

```java
public class MaxScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(MaxScoreDriver.class);
        job.setMapperClass(MaxScoreMapper.class);
        job.setReducerClass(MaxScoreReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                        new Path("hdfs://hadoop01:9000/txt/score2.txt"));
        FileOutputFormat.setOutputPath(job,
                        new Path("hdfs://hadoop01:9000/result/maxscore2"));
        job.waitForCompletion(true);
    }
}
```



### 结果

```
Adair	681
Alex	513
Bob		684
Chad	697
Colin	699
Eden	708
Grace	641
Grover	630
Henry	596
```



## 计算每个人的总分

> 源文件：一个文件夹`score2`
>
> - `chinese.txt`：每个人的语文成绩
> - `english.txt`：每个人的英语成绩
> - `math.txt`：每个人的数学成绩



### Mapper类

```java
public class TotalScoreMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] arr = value.toString().split(" ");
        context.write(new Text(arr[0]), new IntWritable(Integer.parseInt(arr[1])));
    }
}
```



### Reducer类

```java
public class TotalScoreReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}
```



### Driver类

```java
public class TotalScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(TotalScoreDriver.class);
        job.setMapperClass(TotalScoreMapper.class);
        job.setReducerClass(TotalScoreReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/score2/"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/totalscore"));
        job.waitForCompletion(true);
    }
}
```



### 结果

```
Adair	171
Alex	211
Bob	200
Chad	247
Colin	182
Eden	185
Grace	139
Grover	159
Henry	194
```



# 出现错误的解决方案

## Windows配置hadoop环境

> Windows环境对Hadoop不兼容，在运行MapReduce代码时会报错，需要配置Windows的Hadoop环境

![](https://note.youdao.com/yws/api/personal/file/B09AA4EAF4864B618F89D77FD5433E41?method=download&shareKey=ea9f00b989b52aa42a72d3d813e80bd1)

1. 将`Hadoop-alone.tar.gz`解压到一个目录下

2. 将`hadoopbin_for_hadoop2.7.1.zip`解压到Windows的`Hadoop/bin`目录下

   ![](https://note.youdao.com/yws/api/personal/file/784362D867A74625B07BB63F4691F8B6?method=download&shareKey=c66b8b36b2fcb52aa3ac9da11a89e927)

3. 添加环境变量

   - 新建：`HADOOP_HOME	-> D:\software\hadoop-2.7.1\`
   - PATH添加：`%HADOOP_HOME%\bin`

4. 双击`/Hadoop/bin`目录下的`winutils.exe`。出现黑窗口一闪而过证明成功

5. 重启IDEA即可



## 双击`winutils.ext`出现确实dell错误

将`mscvr120.dll`复制到`C:\Windows\System`下



## 运行程序出现异常

> 出现`Could not locate executable null\bin\winutils.exe in the Hadoop binaries`

- 先检查环境变量是否配置正确 
- 如果环境变量正确，那么在Driver添加`System.setProperty("hadoop.home.dir", "Hadoop的安装路径"); `
- 可以将hadoop解压目录下的hadoop.dll复制到`C:\Windows\System32`，重启IDEA，重新运行 
- 如果上面三种方案，全部都没有解决，将`NativeIO.java`文件复制到IDEA中，根据要求建好包

> 如果出现：`NativeIO$Windows `

- 先检查环境变量是否配置正确 
- 如果环境变量配置正确，那么也是将`NativeIO.java`文件复制到IDEA中，根据要求建好包 

> 如果出现：`Permission denied`

检查环境变量：`HADOOP_USER_NAME`的值是不是`root`

> 如果出现：`WebException: Inner Exception: 由于目标计算机积极拒绝，无法连接。 127.0.0.1:50075/50070`

- 在Linux中通过`jps`查看Hadoop的进程是否齐全
- 如果不齐全，重启Hadoop
- 如果重启依然不齐
  - 删除tmp目录：`rm -rf /home/software/hadoop-2.7.1/tmp `
  - 格式化NameNode：`hadoop namenode -format `
  - 开启Hadoop：`start-all.sh `



## 连接工具

如果进程齐全，检查Linux的hosts文件

[TOC]

# Writable - 序列化

## 概述

1. 在Hadoop的集群工作过程中，一般是利用==RPC==来进行集群节点之间的通信和消息的传输，所以要求MapReduce处理的对象必须可以==进行序列化/反序列==操作
2. Hadoop并没有使用Java原生的序列化，而是利用的是==Avro==实现的序列化和反序列，并且在此基础上进行了更好的封装，提供了便捷的API
3. 在Hadoop中要求被序列化的对象对应的类必须实现`Writable`接口，重写其中的`write`方法以及`readFields`方法
4. 序列化过程中要求==属性值不能为null==



## 案例

> 统计每个人的总流量，源文件：`flow.txt`

```
13877779999 bj zs 2145
13766668888 sh ls 1028
13766668888 sh ls 9987
13877779999 bj zs 5678
13544445555 sz ww 10577
13877779999 sh zs 2145
13766668888 sh ls 9987
```



### Flow类

```java
// 实现Hadoop提供的序列化接口
public class Flow implements Writable {
    private String phone = "";
    private String address = "";
    private String name = "";
    private int flow;
 
    public String getPhone() {
        return phone;
    }
 
    public void setPhone(String phone) {
        this.phone = phone;
    }
 
    public String getAddress() {
        return address;
    }
 
    public void setAddress(String address) {
        this.address = address;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getFlow() {
        return flow;
    }
 
    public void setFlow(int flow) {
        this.flow = flow;
    }
 
    // 序列化
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(phone);
        out.writeUTF(address);
        out.writeUTF(name);
        out.writeInt(flow);
    }
 
    // 反序列化
    @Override
    public void readFields(DataInput in) throws IOException {
        phone = in.readUTF();
        address = in.readUTF();
        name = in.readUTF();
        flow = in.readInt();
    }
}
```



### Mapper类

```java
public class SerialFlowMapper extends Mapper<LongWritable, Text, Text, Flow> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 13877779999 bj zs 2145
        String[] arr = value.toString().split(" ");
        Flow f = new Flow();
        f.setPhone(arr[0]);
        f.setCity(arr[1]);
        f.setName(arr[2]);
        f.setFlow(Integer.parseInt(arr[3]));
        context.write(new Text(f.getName()), f);
    }
}
```



### Reducer类

```java
// 统计每一个人花费的总流量 - Text, IntWritable
// 统计每一个人出现的城市 - Text, Text
// 统计每一个人拥有的手机号 - Text, Text
public class SerialFlowReducer extends Reducer<Text, Flow, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<Flow> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (Flow val : values) {
            sum += val.getFlow();
        }
        context.write(key, new IntWritable(sum));
    }
}
```



### Driver类

```java
public class SerialFlowDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(SerialFlowDriver.class);
        job.setMapperClass(SerialFlowMapper.class);
        job.setReducerClass(SerialFlowReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Flow.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/flow.txt"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/serialflow"));
        job.waitForCompletion(true);

    }
}
```



### 结果

```
ls	21002
ww	10577
zs	9968
```



## 练习

> 统计每个人三次考试成绩的最高分、三次考试成绩的平均分（保留2位小数）
>
> 源文件：`score.txt`

```
Bob 90 64 92
Alex 64 63 68
Grace 57 86 24
Henry 39 79 78
Adair 88 82 64
Chad 66 74 37
Colin 64 86 74
Eden 71 85 43
Grover 99 86 43
```



### Student类

```java
public class Student implements Writable {
    private String name = "";
    private List<Integer> score = new ArrayList<>();

    public Student() {}

    public String getName() {return name;}

    public void setName(String name) {this.name = name;}

    public List<Integer> getScore() {return score;}

    public void setScore(List<Integer> score) {this.score = score;}

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(name);
        out.writeInt(score.size());
        for (Integer integer : score) {
            out.writeInt(integer);
        }
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.name=in.readUTF();
        int size = in.readInt();
        this.score = new ArrayList<>(size);
        for (int i = 0;i<size;i++) {
            this.score.add(in.readInt());
        }
    }
}
```



### Mapper类

```java
public class ScoreMapper extends Mapper<LongWritable, Text,Text,Student> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] arr = value.toString().split(" ");
        Student s = new Student();
        List<Integer> list = new ArrayList<>();
        
        s.setName(arr[0]);
        list.add(Integer.valueOf(arr[1]));
        list.add(Integer.valueOf(arr[2]));
        list.add(Integer.valueOf(arr[3]));
        s.setScore(list);
        context.write(new Text(s.getName()),s);
    }
}
```



### 最高分Reducer类

```java
public class MaxScoreReducer extends Reducer<Text,Student,Text, IntWritable> {

    @Override
    protected void reduce(Text key, Iterable<Student> values, Context context) throws IOException, InterruptedException {
        int score = 0;
        List<Integer> list = new ArrayList<>();
        for (Student value : values) {
            list = value.getScore();
        }
        for (Integer integer : list) {
            if (score<integer) {
                score = integer;
            }
        }
        context.write(key,new IntWritable(score));
    }
}
```



### 平均分Reducer类

```java
public class AvgScoreReducer extends Reducer<Text,Student,Text, Text> {

    @Override
    protected void reduce(Text key, Iterable<Student> values, Context context) throws IOException, InterruptedException {
        List<Integer> list = new ArrayList<>();
        for (Student value : values) {
            list = value.getScore();
        }
        int score = 0;
        for (Integer integer : list) {
            score += integer;
        }

        DecimalFormat df = new DecimalFormat("0.00");
        double avgScore = score / list.size();
        String format = df.format(avgScore);

        context.write(key,new Text(format));
    }
}
```



### 最高分Driver类

```java
public class Driver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Job job = Job.getInstance(new Configuration());

        job.setJarByClass(Driver.class);
        job.setMapperClass(ScoreMapper.class);
        job.setReducerClass(MaxScoreReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Student.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/score.txt"));
        FileOutputFormat.setOutputPath(job, new Path("hdfs://hadoop01:9000/result/score3"));

        job.waitForCompletion(true);
    }
}

```



### 平均分Driver类

```java
public class Driver2 {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Job job = Job.getInstance(new Configuration());

        job.setJarByClass(Driver2.class);
        job.setMapperClass(ScoreMapper.class);
        job.setReducerClass(AvgScoreReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Student.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        FileInputFormat.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/score.txt"));
        FileOutputFormat.setOutputPath(job, new Path("hdfs://hadoop01:9000/result/score4"));

        job.waitForCompletion(true);
    }
}
```



### 结果

> 最高分

```
Adair	88
Alex	68
Bob		92
Chad	74
Colin	86
Eden	85
Grace	86
Grover	99
Henry	79
```

> 平均分

```
Adair	78.00
Alex	65.00
Bob	82.00
Chad	59.00
Colin	74.67
Eden	66.33
Grace	55.67
Grover	76.00
Henry	65.33
```



# Partitioner - 分区

## 概述

1. 分区的作用是对数据进行分类
2. 在MapReduce中，分区默认是从0开始，依次递增
3. 在MapReduce中，每一个分区需要对应一个`ReduceTask`（Reduce产生的线程），每一个ReduceTask都会对应一个结果文件
4. 在MapReduce中，如果没有指定分区，那么默认使用的是`HashPartitioner`。如果不指定，默认ReduceTask的数量只有1个，所以只产生一个结果文件

## 案例

> 按地区，统计每一个人花费的总流量
>
> 源文件：flow.txt

```
13877779999 bj zs 2145
13766668888 sh ls 1028
13766668888 sh ls 9987
13877779999 bj zs 5678
13544445555 sz ww 10577
13877779999 sh zs 2145
13766668888 sh ls 9987
```

![](https://note.youdao.com/yws/api/personal/file/FE5A1445DC4B47CEA6C494C7036EF4B8?method=download&shareKey=58fd7b1f1501561e20f076a0ad37c9f4)

### Flow类

```java
public class Flow implements Writable {

    private String phone = "";
    private String city = "";
    private String name = "";
    private int flow;

    public String getPhone() {return phone;}

    public void setPhone(String phone) {this.phone = phone;}

    public String getCity() {return city;}

    public void setCity(String city) {this.city = city;}

    public String getName() {return name;}

    public void setName(String name) {this.name = name;}

    public int getFlow() {return flow;}

    public void setFlow(int flow) {this.flow = flow;}

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(phone);
        out.writeUTF(city);
        out.writeUTF(name);
        out.writeInt(flow);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.phone = in.readUTF();
        this.city = in.readUTF();
        this.name = in.readUTF();
        this.flow = in.readInt();
    }
}
```



### Mapper类

```java
public class PartFlowMapper extends Mapper<LongWritable, Text, Text, Flow> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] arr = value.toString().split(" ");
        Flow f = new Flow();
        f.setPhone(arr[0]);
        f.setCity(arr[1]);
        f.setName(arr[2]);
        f.setFlow(Integer.parseInt(arr[3]));
        context.write(new Text(f.getName()), f);
    }
}
```



### Partitioner类

```java
// 按照城市对数据进行分类 - 不同的城市发往不同的分区
public class CityPartitioner extends Partitioner<Text, Flow> {
    @Override
    public int getPartition(Text text, Flow flow, int numPartitions) {
        // 获取城市
        String city = flow.getCity();
        // 分区是有编号的，编号是从0开始‘
        // 所以就需要将城市和编号进行对应
        if (city.equals("bj"))
            return 0;
        else if (city.equals("sh"))
            return 1;
        else
            return 2;
    }
}
```



### Reducer类

```java
public class PartFlowReducer extends Reducer<Text, Flow, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<Flow> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (Flow val : values) {
            sum += val.getFlow();
        }
        context.write(key, new IntWritable(sum));
    }
}
```



### Driver类

```java
public class PartFlowDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(PartFlowDriver.class);
        job.setMapperClass(PartFlowMapper.class);
        job.setReducerClass(PartFlowReducer.class);

        // 指定分区类
        job.setPartitionerClass(CityPartitioner.class);
        // 每一个分区对应一个ReduceTask
        // 现在设置了3个分区，那么就需要设置3个ReduceTask
        job.setNumReduceTasks(3);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Flow.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/flow.txt"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/partflow"));

        job.waitForCompletion(true);
    }
}
```



### 结果

> 生成了3个结果文件，00000，00001，00002

![](https://note.youdao.com/yws/api/personal/file/A58AB5B2E6A74DC48D28EDD66BA6670B?method=download&shareKey=a4f1c90c4e48f657958c523e63b03267)

```sh
#00000
zs	7823
#00001
ls	21002
zs	2145
#00002
ww	10577
```





## 练习

> 按月份，统计每一个人的总成绩
>
> 源文件：目录score1

```sh
#chinese.txt
1 zhang 89
2 zhang 73
3 zhang 67
1 wang 49
2 wang 83
3 wang 27
1 li 77
2 li 66
3 li 89
#english.txt
1 zhang 55
2 zhang 69
3 zhang 75
1 wang 44
2 wang 64
3 wang 86
1 li 76
2 li 84
3 li 93
#math.txt
1 zhang 85
2 zhang 59
3 zhang 95
1 wang 74
2 wang 67
3 wang 96
1 li 45
2 li 76
3 li 67
```



### Score类

```java
public class Score implements Writable {

    private int month;
    private String name = "";
    private int score;

    public int getMonth() {return month;}

    public void setMonth(int month) {this.month = month;}

    public String getName() {return name;}

    public void setName(String name) {this.name = name;}

    public int getScore() {return score;}

    public void setScore(int score) {this.score = score;}

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(month);
        out.writeUTF(name);
        out.writeInt(score);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.month = in.readInt();
        this.name = in.readUTF();
        this.score = in.readInt();
    }
}
```



### Mapper类

```java
public class PartScoreMapper extends Mapper<LongWritable, Text, Text, Score> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 拆分字段
        String[] arr = value.toString().split(" ");
        // 封装对象
        Score s = new Score();
        s.setMonth(Integer.parseInt(arr[0]));
        s.setName(arr[1]);
        s.setScore(Integer.parseInt(arr[2]));
        context.write(new Text(s.getName()), s);
    }
}
```



### Partitioner类

```java
public class MonthPartitioner extends Partitioner<Text, Score> {
    @Override
    public int getPartition(Text text, Score score, int numPartitions) {
        int month = score.getMonth();
        // 1月 -> 0号线程
        // 2月 -> 1号线程
        // 3月 -> 2号线程
        return month - 1;
    }
}
```



### Reducer类

```java
public class PartScoreReducer extends Reducer<Text, Score, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<Score> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (Score val : values) {
            sum += val.getScore();
        }
        context.write(key, new IntWritable(sum));
    }
}
```



### Driver类

```java
public class PartScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(PartScoreDriver.class);
        job.setMapperClass(PartScoreMapper.class);
        job.setReducerClass(PartScoreReducer.class);

        job.setPartitionerClass(MonthPartitioner.class);
        job.setNumReduceTasks(3);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Score.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/score1/"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/partscore"));

        job.waitForCompletion(true);
    }
}

```



### 结果

```sh
#00000
li		198
wang	167
zhang	229
#00001
li		226
wang	214
zhang	201
#00002
li		249
wang	209
zhang	237
```





# Comparable - 排序

## 概述

1. 在MapReduce中，会默认对Mapper输出的==键==来进行==自然排序==，所以要求Mapper==输出的键==对应的类型必须实现Comparable接口。

   > 考虑到这个类型的对象还要被序列化，所以还要实现Writable接口，所以Hadoop提供了`WritableComparable`接口

2. 如果针对多字段进行排序，称之为二次排序

3. 在排序的时候，如果两个键`compareTo`的结果是0，那么在MapReduce中，会认为这两个键实际上是同一个键，然后在Reduce阶段将这两个键对应的值放到一组去

## 案例

> 对统计的总成绩进行降序排序
>
> 源文件：HDFS中 `/result/totalscore`

```
Adair	171
Alex	211
Bob		200
Chad	247
Colin	182
Eden	185
Grace	139
Grover	159
Henry	194
```



### Score类

```java
public class Score implements WritableComparable<Score> {

    private String name = "";
    private int score;

    public String getName() { return name;}

    public void setName(String name) {this.name = name;}

    public int getScore() {return score;}

    public void setScore(int score) {this.score = score;}

    // 按照分数进行降序排序
    @Override
    public int compareTo(Score o) {
        return o.score - this.score;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(name);
        out.writeInt(score);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.name = in.readUTF();
        this.score = in.readInt();
    }
}
```



### Mapper类

```java
public class SortScoreMapper extends Mapper<LongWritable, Text, Score, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] arr = value.toString().split("\t");
        Score s = new Score();
        s.setName(arr[0]);
        s.setScore(Integer.parseInt(arr[1]));
        context.write(s, NullWritable.get());
    }
}
```



### Reducer类

```java
public class SortScoreReducer extends Reducer<Score, NullWritable, Text, IntWritable> {
    @Override
    protected void reduce(Score key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        context.write(new Text(key.getName()), new IntWritable(key.getScore()));
    }
}
```



### Driver类

```java
public class SortScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(SortScoreDriver.class);
        job.setMapperClass(SortScoreMapper.class);
        job.setReducerClass(SortScoreReducer.class);

        job.setMapOutputKeyClass(Score.class);
        job.setMapOutputValueClass(NullWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 在MapReduce中，如果一个文件以_开头，MapReduce默认不处理这个文件
        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/result/totalscore"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/sortscore"));

        job.waitForCompletion(true);
    }
}
```



### 结果

> 将结果按照成绩降序排序

```
Chad	247
Alex	211
Bob		200
Henry	194
Eden	185
Colin	182
Adair	171
Grover	159
Grace	139
```



## 练习

> 先按照月份升序排序，如果是同一个月，按照利润降序
>
> 源文件：profit2.txt

```
2 tom 345
1 rose 235
1 tom 234
2 jim 572
3 rose 123
1 jim 321
2 tom 573
3 jim 876
3 tom 648
```



### Profit类

> 利润
>
> ==排序逻辑在compareTo方法中实现==

```java
public class Profit implements WritableComparable<Profit> {

    private int month;
    private String name;
    private int profit;

    public int getMonth() {return month;}

    public void setMonth(int month) {this.month = month;}

    public String getName() {return name;}

    public void setName(String name) {this.name = name;}

    public int getProfit() {return profit;}

    public void setProfit(int profit) {this.profit = profit;}

    @Override
    public String toString() {
        return "Profit{" +
                "month=" + month +
                ", name='" + name + '\'' +
                ", profit=" + profit +
                '}';
    }

    // 先按照月份排序，如果是同一个月，那么就按照例如降序
    @Override
    public int compareTo(Profit o) {
        int r1 = this.month - o.month;
        if (r1 == 0) {
            // 如果为0，说明是同一个月，同一个月要按照利润降序排序
            int r2 = o.profit - this.profit;
            return r2 == 0 ? 1 : r2;
        }
        return r1;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(month);
        out.writeUTF(name);
        out.writeInt(profit);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.month = in.readInt();
        this.name = in.readUTF();
        this.profit = in.readInt();
    }
}
```



### Mapper类

```java
public class SortProfitMapper extends Mapper<LongWritable, Text, Profit, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] arr = value.toString().split(" ");
        Profit p = new Profit();
        p.setMonth(Integer.parseInt(arr[0]));
        p.setName(arr[1]);
        p.setProfit(Integer.parseInt(arr[2]));
        context.write(p, NullWritable.get());
    }
}
```



### Reducer类

```java
public class SortProfitReducer extends Reducer<Profit, NullWritable, Profit, NullWritable> {
    @Override
    protected void reduce(Profit key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        context.write(key, NullWritable.get());
    }
}
```



### Driver类

```java
public class SortProfitDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(SortProfitDriver.class);
        job.setMapperClass(SortProfitMapper.class);
        job.setReducerClass(SortProfitReducer.class);

        job.setOutputKeyClass(Profit.class);
        job.setOutputValueClass(NullWritable.class);


        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/profit3.txt"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/sortprofit3"));

        job.waitForCompletion(true);
    }
}
```



### 结果

```
Profits{month=1, name='jim', profits=321}
Profits{month=1, name='rose', profits=235}
Profits{month=1, name='tom', profits=234}
Profits{month=2, name='tom', profits=573}
Profits{month=2, name='jim', profits=572}
Profits{month=2, name='tom', profits=345}
Profits{month=3, name='jim', profits=876}
Profits{month=3, name='tom', profits=648}
Profits{month=3, name='rose', profits=123}
```



# Combiner - 合并

1. 因为实际过程中MapTask的数量要远多于ReduceTask的数量，所以所有的计算压力最终都会落到ReduceTask上，所以就要考虑降低ReduceTask的计算压力

   > 考虑将ReduceTask一部分的计算前移，在MapTask这一端先进行一次汇总，最后ReduceTask再进行最后的汇总

2. Combiner的执行逻辑和Reducer是一样的，所以实际过程中将Reducer类也作为Combiner类来使用

3. Combiner的作用是为了减少数据总条数，不改变计算结果

4. 如果需要添加合并类，只需要在Driver中添加`job.setCombinerClass(xxxDriver.class)`即可

5. Combiner一定程度上确实可以提高效率，在实际开发中也建议尽量增加Combiner

   > 注意：并不是所有的场景都适合使用Combiner
   >
   > 例如：求和、去重、获取最值可以使用Combiner
   >
   > 求平均值等场景不适合用Combiner

![](https://note.youdao.com/yws/api/personal/file/42A84220CE54488ABC66CCC133C67D75?method=download&shareKey=f193aa36348d761c5d706e2700bc71aa)



```java
public class SortProfitDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(SortProfitDriver.class);
        job.setMapperClass(SortProfitMapper.class);
        job.setReducerClass(SortProfitReducer.class);

        job.setOutputKeyClass(Profit.class);
        job.setOutputValueClass(NullWritable.class);

        // 指定合并类
        // job.setCombinerClass(SortProfitReducer.class);


        FileInputFormat.addInputPath(job,
                new Path("hdfs://hadoop01:9000/txt/profit3.txt"));
        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/sortprofit3"));

        job.waitForCompletion(true);
    }
}
```

[TOC]

# 数据本地化策略

1. 当JobTracker访问资源的时候需要向NameNode请求数据
2. JobTracker获取到数据的描述信息，根据描述信息对数据进行了切片（InputSplit），然后将切片发给不同MapTask来执行
3. MapTask在TaskTracker上执行，在执行的时候需要获取实际的数据
4. TaskTracker需要去访问DataNode，为了节省带宽资源，所以往往将DataNode和TaskTracker放在同一个节点上 --- ==数据本地化策略==
5. 为了减少网络资源的消耗，往往还会将切片的大小和实际的Block的大小设置的相同

![](https://note.youdao.com/yws/api/personal/file/17173E318E2B4371B758877A18A06F97?method=download&shareKey=a1df6a0bf2101c93b9f656d9d1963190)

> **JobTracker**：对外接收任务，拆分任务；对内管理TaskTracker
>
> **TaskTracker**：执行任务
>
> **Split**：切片，对数据的逻辑切片，每一个线程要处理的数据量，每一个Split会分配一个MapTask
>
> **Block**：切块，数据真正的物理切片，数据的存储大小



## 切片过程

1. 如果文件为空（即文件大小是0B），将整个文件作为一个切片来处理

2. 在MapReduce中，文件分为==可切分==和==不可切分==。

   > 注意：这个切分和不可切分指的是逻辑切分。例如绝大部分的压缩文件都是不可切的

3. 如果文件不可切（不能逻辑切分），那么将整个文件作为一个切片来处理

4. 默认情况下，Split大小和Block大小是一致的

5. 如果需要调整Split大小

   - 调大SplitSize，需要调大minSize
   - 调小SplitSize，需要调小maxSize

   ==底层代码==

   ```java
   protected long computeSplitSize(long blockSize,long minSize,long maxSize){
       return Math.max(minSize,Math.max(maxSize,blockSize));
   }
   ```

   ==Driver类代码==

   ```java
   //设置minSize
   FileInputFormat.setMinInputSplitSize(xxx);
   //设置maxSize
   FileInputFormat.setMaxInputSplitSize(xxx);
   ```

6. 在计算分片的时候，需要注意这个切片的阈值。

   > 切片阈值默认1.1，即 `剩余字节个数/splitSize > 1.1`的时候才会计算分片
   >
   > 如果 `剩余字节个数/splitSize <= 1.1`，那么剩余的字节个数整体作为1个切片处理

   ![](https://note.youdao.com/yws/api/personal/file/C8C973BD7DE84C648F60363B2AFE96E2?method=download&shareKey=161a1da508270400db2e40c20437d9a8)



# MapReduce执行流程

## Map阶段流程

![](https://note.youdao.com/yws/api/personal/file/0B2A229F26694314B5E1590525C03905?method=download&shareKey=a524df4374707d9b02b8ef2117c55f67)



## Reduce阶段流程

![](https://note.youdao.com/yws/api/personal/file/9195DF42DE4B47AD916AD54E0459E7B2?method=download&shareKey=35916c07cad94bee8d1b87a7a722adb3)



## 整体流程

> 参考图片1：

![](https://note.youdao.com/yws/api/personal/file/038E1E905A524A5A872BE484AB88D8A8?method=download&shareKey=9e08c06c4351aa88ae8f7c6c0b9e6f8a)

 

> 参考图片2：

![](https://note.youdao.com/yws/api/personal/file/7AC16D7EB04A47F0B0BDF267CD3E4583?method=download&shareKey=1c9aa210785d24e6460657fad61750a7)



> 参考图片3：

![](https://note.youdao.com/yws/api/personal/file/D4117E0547BA4567AB9EBB657C3C35E5?method=download&shareKey=c7c8a3e6326a1a92f1075d5c03d8e978)



# Job执行流程

## 提交流程

1. 检查输入和输出路径
   - 检查输入路径是否存在，如果不存在抛出FileNotFountException
   - 检查输出路径是否存在
     - 如果存在，抛出FileAlreadyExitsException
     - 如果不存在，就会常见这个路径
2. 计算切片数量，产生切片
3. 如果有需要，可以为分布式缓存设置存根账户信息
4. 将这个任务的jar包和配置信息提交到HDFS上存储，完成后会删除
5. 将job提交给JobTracker，并且选择是否监控这个job的执行状态

## 执行流程

1. JobTracker在收到Job任务之后，会将这个Job任务拆分成MapTask和ReduceTask

   - MapTask的数量由切片数量来决定
   - ReduceTask的数量由分区数量来决定

   拆分完成后，JobTracker等待TaskTracker的心跳

2. JobTracker在收到TaskTracker的心跳之后，会分配任务。

   > 注意：再分配任务的时候MapTask要考虑数据本地化，ReduceTask则要分配到相对空闲的节点上。在这个过程中，每一个TaskTracker不一定只领取到一个任务，可能会领取多个任务

3. TaskTracker通过心跳领取到任务之后，连接对应的节点将jar包下载过来，解析jar包获取执行逻辑，这一步体现了：==逻辑移动，数据固定==的思想

4. TaskTracker解析完jar包之后，在本节点（服务器）上开启一个JVM子进程执行MapTask或者ReduceTask。

   > 默认情况下，每执行一个MapTask或者ReduceTask就会开启一次JVM子进程，执行完成之后就会关闭这个JVM子进程。也就意味着，如果一个TaskTracker领取到多个任务，那么就会开启和关闭多次JVM子进程

# Map端Shuffle

1. MapTask调用`map()`方法处理数据，默认情况下是读取一行处理一行。`map()`方法在处理完数据之后，会将这个数据写到MapTask自带的缓冲区中

   > 每一个MapTask都会自带一个缓冲区

2. 数据在缓冲区中进行分区、排序，如果指定了Combiner，那么还会进行combine合并

   > 数据在缓冲区中进行排序的时候，将完全杂乱的数据排成有序的数据，这个过程采用的是==快速排序==

3. 缓冲区本质上是一个环形的字节数组，默认大小是100MB，维系在内存中

4. 当缓冲区使用达到阈值的时候，会将缓冲区的数据进行溢写(spill)，将缓冲区中的数据溢写到磁盘上，产生一个溢写文件

   > 溢写阈值默认是0.8，即表示当缓冲区的容量使用达到80%的时候进行溢写

5. 当缓冲区溢写之后，MapTask产生的新数据依然是放到缓冲区中，达到条件再次溢写，每一次溢写都会产生一个新的溢写文件

6. 单个溢写文件中的数据应该是分好区且排好序的，多个溢写文件之间是整体无序的，但局部是有序的

7. 在MapTask处理完所有数据之后，会将所有的溢出文件进行==合并==(merge)，合并成一个文件==(final out)==；

   > - 如果MapTask处理完之后，一部分结果在溢写文件中，一部分结果在缓冲区中，那么就将溢写文件和缓冲区中的数据都merge到final out
   > - 如果没有产生溢写过程，那么MapTask在处理完成之后会将数据直接写到final out中。
   >
   > 也就意味着MapTask在处理完数据之后一定会产生一个final out文件

8. 在merge过程中，数据会再次进行分区排序，所以final out文件中数据都是分好区且排好序的。如果制定了Combiner，会在溢写文件>=3个时，在merge过程会进行combine操作

   > merge过程中的排序是将局部有序的数据整理为整体有序的数据，这个过程采用==归并排序==



**注意问题：**

> - 溢写过程不一定会产生
> - 原始数据的大小并不能决定是否溢写，得看MapTask处理之后产生的数据量，而不是处理之前的数据量
> - 溢写过程本质上是数据从内存写到磁盘的过程，这个过程中要考虑序列化因素，所以溢写文件的大小不一定等于【缓冲区大小*溢写阈值】
> - 将缓冲区设置为环形的目的是为了重复利用这个缓冲区，而且不用频繁的寻址
> - 设置溢写阈值的目的是为了尽量减少甚至避免MapTask在写出结果过程中产生阻塞



![](https://note.youdao.com/yws/api/personal/file/22B2627C0F294ED88765BFB488A5E50D?method=download&shareKey=68e37c3f76ee909b36f0f0181300355b)



> Map阶段的处理流程

![](https://note.youdao.com/yws/api/personal/file/0B2A229F26694314B5E1590525C03905?method=download&shareKey=a524df4374707d9b02b8ef2117c55f67)





# Reduce端Shuffle

1. ReduceTask启动阈值默认是`0.05`，即当有5%的MapTask执行结束之后，就会启动ReduceTask来抓取数据

2. ReduceTask启动fetch线程通过http请求去MapTask抓取数据，在抓取数据的时候只抓取当前ReduceTask处理的分区数据

   > 默认每一个ReduceTask可以启动5个fetch线程

3. ReduceTask通过fetch线程抓取到数据之后，每一段数据都会临时存储在本地的一个小文件中，在抓取完成之后，会对这个小文件进行==merge==(合并)，在merge过程中，数据会再次进行排序

   > 这一次排序依然是将数据从局部有序整理成 - 整体有序，采用==归并排序==
   >
   > 在Reduce的merge过程中，merge因子默认为10，即每10个小文件合并成一个文件

4. merge完成之后，会将相同的键对应的值放到一组中，这个过程称之为：==分组==(group)；分组完成之后，每一个键调用一次`Reduce()`方法计算结果

   > 分组之后，这一组值会产生一个迭代器，对应了reduce()方法中的values，这个迭代器实际上是一个伪迭代器
   >
   > 在分组的时候，实际上并不是真的产生了一个迭代器，而是去读取文件中数据，每次从这个文件中读取2行数据，比较2行数据的键是否一致
   >
   > - 如果一致，则将第一行读取到的数据放到reduce中处理，同时继续读取
   > - 如果不一致，就会标记第二行重新调用reduce()方法，同时标记上一行迭代结束
   >
   > 这里的迭代器本质上不是产生一个容器，而是去读取文件
   >
   > ![](https://note.youdao.com/yws/api/personal/file/BAF3D3F166854027B6F10415CF8AC972?method=download&shareKey=43c540d9d3eafd9c8cf6223f17eb850b)



> Reduce阶段的处理流程

![](https://note.youdao.com/yws/api/personal/file/9195DF42DE4B47AD916AD54E0459E7B2?method=download&shareKey=35916c07cad94bee8d1b87a7a722adb3)



# Shuffle整体流程

> 参考图片1：👇👇👇

![](https://note.youdao.com/yws/api/personal/file/8478B92FA75A4778AAE2EB72E3E04BEE?method=download&shareKey=767a2da352cc85147c8eb0a2cd4a6fa9)


> 参考图片2：👇👇👇

![](https://note.youdao.com/yws/api/personal/file/038E1E905A524A5A872BE484AB88D8A8?method=download&shareKey=9e08c06c4351aa88ae8f7c6c0b9e6f8a)



# Shuffle优化

1. ⭐**增大缓冲区**。在实际开发过程中，会将缓冲区大小设置为250MB~400MB
2. **增大溢写阈值**。例如：可以将阈值增大到0.9，但是这种方案可能会增加线程阻塞的风险
3. ⭐**尽量增加Combiner过程**
4. ⭐**如果网络资源比较紧张，可以适当对final out进行压缩**。但是同样ReduceTask通过fetch线程抓取到数据之后需要进行解压。所以这个方法是对网络资源和解压时间的取舍
5. ⭐**适当增加fetch的线程数量**。需要考虑服务器的线程承载量
6. **增大merge因子**。这种方案增加底层运算的复杂度，不建议使用
7. **适当的调小ReduceTask的启动阈值**。实际过程中不建议调节

[TOC]



# InputFormat - 输入格式

1. InputFormat是MapReduce中提供的一个输入格式的顶级父类，提供了2个方法：

   - `getSplits`：规定切片的过程
   - `createRecordReader`：针对指定的切片来提供输入流来读取数据的

2. InputFormat发生在Map之前。在MapTask之前，先对文件切片，然后提供输入流读取数据，将读取的数据发送给MapTask，也就意味着InputFormat读出来的数据是什么格式，Mapper接收到数据就是什么格式

3. 在默认情况下，输入格式类用的是：`TextInputFormat`，默认是按行读取，然后将读取的数据发送给Map。在TextInputFormat中，为了保证数据的完整性，每一个MapTask都是从当前切片的第二行开始，处理到下一个切片的第一行

   

   ![](https://note.youdao.com/yws/api/personal/file/16551EDE6A1A4F4AB213E058DFD49755?method=download&shareKey=2c8eb1068bb9715a2ee65d3a40d2268d)

   

## 自定义输入格式

> 写一个类继承InputFormat，考虑到切片过程相对比较繁琐，可以考虑继承`FileInputFormat`，这个类中实现了`getSplit()`方法，只需要覆盖`createRecordReader()`即可



### 源文件

> 统计该文件的每个人的总分。
>
> 因为不像正常的一人一行数据，这个文件是一个人，三行数据，需要自定义输入格式，每次读三行

```
tom
math 90
english 98
jary
math 78
english 87
rose
math 87
english 90
bob
math 67
english 87
alex
math 59
english 80
helen
math 79
english 60
```



### 自定义的InputFormat类

```java
// 泛型表示的输出类型 - 泛型定义的是什么类型，那么Mapper接收的就是类型
public class AuthInputFormat extends FileInputFormat<Text, Text> {
    @Override
    public RecordReader<Text, Text> createRecordReader(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        return new AuthReader();
    }
}

class AuthReader extends RecordReader<Text, Text> {

    private LineReader reader;
    private Text key;
    private Text value;
    private static final byte[] blank = new Text(" ").getBytes();

    // 初始化的方法，在这个方法中一般获取一个真正的用于读取数据的流
    @Override
    public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {

        // 获取地址
        FileSplit fsplit = (FileSplit) split;
        Path path = fsplit.getPath();
        URI uri = URI.create(path.toString());
        // 连接HDFS
        FileSystem fs = FileSystem.get(uri, context.getConfiguration());
        // 获取输入流用于读取数据
        InputStream in = fs.open(path);
        // 获取的这个输入流是一个字节流
        // 目的是为了读取3行，每3行组成1条数据 - 字节流在读取的时候还得自己判断什么时候才读完三行
        // 考虑将字节流包装成字符流，并且这个字符流最好能够按行读取
        reader = new LineReader(in);

    }

    // 判断是否有下一个键值对来提供给map方法处理，如果有返回true
    // 可以试图读取数据，如果读取到了数据，那么说明还有数据提供给map处理，返回true
    // 如果没有读取到数据，那么说明所有的数据已经读取完了，那么map方法也没有数据处理，返回false
    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {

        // 初始化变量
        key = new Text();
        value = new Text();
        Text tmp = new Text();
        // 试图读取
        // 读取第一行
        // 如果没有读取到数据，返回0
        if (reader.readLine(tmp) == 0) return false;
        key.set(tmp.toString());
        // 读取第二行
        if (reader.readLine(tmp) == 0) return false;
        value.set(tmp.toString());
        
        // 需要在值之间拼接空格
        value.append(blank, 0, blank.length);
        
        // 读取第三行
        if (reader.readLine(tmp) == 0) return false;
        byte[] data = tmp.getBytes();
        value.append(data, 0, data.length);
        // 读取完成之后的结构
        // key = tom
        // value = math 90 english 98

        return true;
    }

    // 获取键
    @Override
    public Text getCurrentKey() throws IOException, InterruptedException {
        return key;
    }

    // 获取值
    @Override
    public Text getCurrentValue() throws IOException, InterruptedException {
        return value;
    }

    // 获取当前MapTask的执行进度 - 实际上就是获取数据读取的进度
    // 是否覆盖这个方法对整个结果没有影响
    @Override
    public float getProgress() throws IOException, InterruptedException {
        return 0;
    }

    // 回收
    @Override
    public void close() throws IOException {
        if (reader != null)
            reader.close();
        key = null;
        value = null;
    }

}
```



### Score类

```java
public class Score implements Writable {

    private int math;
    private int english;

    public int getMath() {return math;}

    public void setMath(int math) {this.math = math;}

    public int getEnglish() {return english;}

    public void setEnglish(int english) {this.english = english;}

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(math);
        out.writeInt(english);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.math = in.readInt();
        this.english = in.readInt();
    }
}
```



### Mapper类

```java
public class AuthMapper extends Mapper<Text, Text, Text, Score> {
    @Override
    protected void map(Text key, Text value, Context context) throws IOException, InterruptedException {
        // key = tom
        // value = math 90 english 98
        String[] arr = value.toString().split(" ");
        Score s = new Score();
        s.setMath(Integer.parseInt(arr[1]));
        s.setEnglish(Integer.parseInt(arr[3]));
        context.write(key, s);
    }
}
```



### Reducer类

```java
public class AuthReducer extends Reducer<Text, Score, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<Score> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        Score s = values.iterator().next();
        sum += s.getMath() + s.getEnglish();
        context.write(key, new IntWritable(sum));
    }
}
```



### Driver类

```java
public class AuthDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(AuthDriver.class);
        job.setMapperClass(AuthMapper.class);
        job.setReducerClass(AuthReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Score.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 指定输入格式类
        job.setInputFormatClass(AuthInputFormat.class);

        FileInputFormat.addInputPath(job,
                         new Path("hdfs://hadoop01:9000/txt/score3.txt"));
        FileOutputFormat.setOutputPath(job,
                         new Path("hdfs://hadoop01:9000/result/authinput"));

        job.waitForCompletion(true);
    }
}
```



### 结果

```
tom		188
jary	165
rose 	177
bob		154
alex	139
helen	139
```



## 多源输入

> 一次性的指定多个输入路径，并且允许为不同的输入路径来指定不同的Input Format以及Mapper类
>
> `MultipleInputs.addInputPath(Job,Path,输入格式类,Mapper)`

### Mapper类1

```java
// 处理score.txt
public class ScoreMapper1 extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // Alex 64 63 68
        String[] arr = value.toString().split(" ");
        Text name = new Text(arr[0]);
        for (int i = 1; i < arr.length; i++) {
            context.write(name, new IntWritable(Integer.parseInt(arr[i])));
        }

    }
}
```



### Mapper类2

```java
// 处理score3.txt
public class ScoreMapper2 extends Mapper<Text, Text, Text, IntWritable> {
    @Override
    protected void map(Text key, Text value, Context context) throws IOException, InterruptedException {
        // key = tom
        // value = math 90 english 98
        String[] arr = value.toString().split(" ");
        context.write(key, new IntWritable(Integer.parseInt(arr[1])));
        context.write(key, new IntWritable(Integer.parseInt(arr[3])));
    }
}
```



### Reducer类

```java
public class ScoreReducer extends Reducer<Text, IntWritable, Text, Text> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        double sum = 0;
        int count = 0;
        for (IntWritable val : values) {
            sum += val.get();
            count++;
        }
        double avg = sum / count;
        DecimalFormat df = new DecimalFormat("0.00");
        String str = df.format(avg);
        context.write(key, new Text(str));
    }
}
```



### Driver类



```java
import cn.tedu.authinput.AuthInputFormat;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.MultipleInputs;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class ScoreDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(ScoreDriver.class);
        job.setReducerClass(ScoreReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        // 多源输入
        MultipleInputs.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/score.txt"),
                TextInputFormat.class, ScoreMapper1.class);
        MultipleInputs.addInputPath(job, new Path("hdfs://hadoop01:9000/txt/score3.txt"),
                AuthInputFormat.class, ScoreMapper2.class);

        FileOutputFormat.setOutputPath(job,
                new Path("hdfs://hadoop01:9000/result/multipleinputs"));
        job.waitForCompletion(true);
    }
}
```





# OutPutFormat - 输出格式

1. OutPutFormat是输出格式的顶级父类，如果需要自定义输出格式，需要继承这个类
2. ==实际开发中，很少自定义输出格式==，而是采用==MapReduce中默认的输出格式==
3. 在MapReduce中，如果没有指定输出格式，那么默认采用的输出格式类为：`TextOutputFormat`



## 多源输出

### Mapper类

```java
public class MoutMapper extends Mapper<LongWritable, Text, Text, Text> {

    public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        String[] arr = line.split(" ");
        context.write(new Text(arr[0]), new Text(arr[1]));
    }
}
```



### Reduce类

```java
public class MoutReducer extends Reducer<Text, Text, Text, Text> {

    private MultipleOutputs<Text, Text> mo;

    @Override
    protected void setup(Reducer<Text, Text, Text, Text>.Context context) throws IOException, InterruptedException {
        mo = new MultipleOutputs<>(context);
    }

    public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        String name = key.toString();
        Text value = values.iterator().next();
        if (name.charAt(0) <= 'I')
            mo.write("a2i", key, value);
        else
            mo.write("j2z", key, value);
    }
}
```



### Driver类

```java
public class MoutDriver {

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "JobName");
        job.setJarByClass(cn.tedu.multiout.MoutDriver.class);
        job.setMapperClass(MoutMapper.class);
        job.setReducerClass(MoutReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        FileInputFormat.setInputPaths(job, new Path("hdfs://192.168.32.147:9000/txt/score2.txt"));
        FileOutputFormat.setOutputPath(job, new Path("hdfs://192.168.32.147:9000/result4"));

        MultipleOutputs.addNamedOutput(job, "a2i", TextOutputFormat.class, Text.class, Text.class);
        MultipleOutputs.addNamedOutput(job, "j2z", TextOutputFormat.class, Text.class, Text.class);

        if (!job.waitForCompletion(true))
            return;
    }

}
```

# 数据倾斜

1. 数据本身具有倾斜的特性。数据本身就是不平均的，数据倾斜不能避免

2. Map端也会产生数据倾斜。倾斜条件：

   - 多源输入
   - 输入的文件不可切分且大小不均等

   Map端的数据倾斜无法避免且无法解决

3. 在实际生产场景中，绝大部分的数据倾斜都是发生在了Reduce端

   > Reduce端产生数据倾斜的本质原因是因为数据本身具有倾斜特性，但是Reduce端产生倾斜的表面原因是因为数据的分类（分区）。
   >
   > 针对Reduce端的数据倾斜，经常采用的方案：==二/两阶段聚合==

## 二/两阶段聚合

> 倾斜度越高，二阶段聚合的效率就越高

1. 先将数据打散，打散之后先分布聚合
2. 按照业务指定分类，对数据进行最后的汇总

![](https://note.youdao.com/yws/api/personal/file/C3D6A39FDB9A45488295E60001B1AD36?method=download&shareKey=7707b5e6af1354cc5feda562fb62ae52)



# 小文件

1. 小文件的危害

   - 存储：每一个小文件在HDFS上都会产生一条元数据。如果存储大量的小文件，那么就会产生大量的元数据。如果元数据过多，会大量占用NameNode的内存，并且还会导致元数据的查询效率变低
   - 计算：每一个小文件在MapReduce中对应一个切片。如果计算大量的小文件，那么就会产生大量的切片，每一个切片会对应一个MapTask(本质上是一个线程)，大量的切片就会产生大量的线程。如果线程数量过多，这个时候就会导致服务器的效率变低甚至会崩溃

2. 目前为止，市面上针对小文件的常见处理手段：==合并和打包==

3. Hadoop提供了一种原生的打包手段：`Hadoop Archive`，可以将一个或者多个小文件打成一个har包

   > 例如：`hadoop archive -archiveName txt.har -p /txt/`



# 推测执行机制

1. 推测执行机制实际上是Hadoop提供的一种针对慢任务的优化方法：当出现慢任务的时候，Hadoop会将这个慢任务复制一份放到其他节点上，两个节点同时执行相同的任务，谁先执行完，那么结果就作为最后的结果，另一个没有执行完的任务就会被kill掉 
2. 慢任务出现的场景：
   - 任务分配不均匀 
   - 机器性能不均等 
   - 数据倾斜
3. ==实际开发中==，数据倾斜导致的慢任务出现的概率更高，但是此时推测执行机制是无效的，反而还会加剧集群的资源消耗，也因此==一般会关闭推测执行机制==

# 概述

1. YARN(Yet  Another Resource Negotiator，迄今另一个资源协调者)是Hadoop2.0中提供一套用于进行资源管理和任务调度的框架 
2. YARN产生的原因：
   -	**内因**：在Hadoop1.0中，每一个MapReduce程序都会拆分出很多个MapTask和ReduceTask，JobTracker需要将这些MapTask和ReduceTask分布到不同的TaskTracker上执行，并且还需要监控这些任务的执行情况，这就使得JobTracker要完成的任务较多，这就使得JobTracker的扩展性变得非常差。在Hadoop1.0的官方数据中，给定JobTracker最多只能管理4000个TaskTracker 
    -	**外果**：随着大数据的发展，产生了越来越多的计算框架，例如MapReduce、Spark、Flink等，这些计算框架都是基于Hadoop集群来使用，那么这个时候不同框架之间就可能会产生资源的使用冲突，所以在Hadoop2.0中提供了YARN来做资源的统一管理 
3. YARN节点的组成
   - ResourceManager：资源管理（主节点）
   - NodeManager：执行任务（从节点）



# Job执行流程

1. 当`ResourceManager`中的`ApplicationsManager`收到Job任务之后，会先将这个Job任务临时存储，并且等待`NodeManager`的心跳

2. 当`ApplicationsManager`收到`NodeManager`的心跳的时候，会将Job任务交给这个NodeManager来处理

3. NodeManager收到Job任务之后，会在本节点内部开启一个`ApplicationMaster`进程，然后Job任务分给这个ApplicationMaster

4. ApplicationMaster收到Job任务之后，会将这个任务拆分，拆分成子任务（MapTask和ReduceTask），然后向ResourceManager中的ApplicationsManager发送请求，申请资源。在申请资源的时候，ApplicationMaster会多要资源

   > 例如一个Job任务拆分出4个MapTask和1个ReduceTask，在申请资源的时候会申请13（4*3+1）份资源（默认情况下，一份资源包含了1G内存+1个CPU核）

5. ResourceManager中的ApplicationsManager在收到请求之后：

   - 会将这个请求传递给ResourceManager中的ResourceScheduler

   - ResourceScheduler在收到请求之后，会将所需要的资源封装成Container对象（包含任务执行可以占用的内存、CPU核数等）传递给ApplicationsManager

   - 然后ApplicationsManager再返回给ApplicationMaster。

   - ApplicationMaster会多要资源，但是ResourceManager只会给恰好的资源数

     > 例如一个Job拆分出4个MapTask和1个ReduceTask，在申请资源的时候会要13份资源，ResourceManager只会返回5份资源

   ![](https://note.youdao.com/yws/api/personal/file/CA7CD7DAF5484E4AA8C62DE6D34291A6?method=download&shareKey=6cb760ac63f669f641b5b71503450738)

6. ApplicationMaster收到Container之后，会对资源进行二次分配，将资源分配给具体的子任务，资源分配完成之后，ApplicationMaster会将子任务分配到不同的NodeManager上执行，ApplicationMaster还会监控这些子任务的执行情况

   > 再分配任务的时候，MapTask要考虑数据本地化

7. YARN形成的结构是：

   - ResourceManager管理ApplicationMaster
   - ApplicationMaster管理具体的子任务

   在YARN中，每一个Job任务都对应一个ApplicationMaster

8. ResourceManager的组成

   - ApplicationsManager：对外接收任务，对内管理ApplicationMaster
   - ResourceScheduler：资源的分配

   ![](https://note.youdao.com/yws/api/personal/file/5611DA9FBBF9472AA2A77D8DCA21FE68?method=download&shareKey=79bda4fb2255cbab6b71fc59d33d070b)

9. ResourceScheduler在ResourceManager中被设计成了==可插拔==的组件

   > 资源调度策略：
   >
   > - **FCFS**（First Come First Service）：先来先服务。这种策略也是最简单的，YARN中如果不指定，默认也是采取这种策略
   > - **优先级策略**：在发起请求的时候，需要给请求设置优先级，优先级越高，优先给这个请求分配资源
   > - **短任务优先**：在一个时间段内，如果请求之间执行时间的差别比较大，那么可以考虑将资源优先分配给执行时间短的任务
   > - **混合策略**：在实际开发中，往往不是使用单一策略，而是多种策略组合，会先考虑短任务优先策略，但是在任务执行时间差别不大的情况下，会在考虑优先级策略，如果优先级一样，最后考虑FCFS
