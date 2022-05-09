# Flume

**Flume is a distributed, reliable, and available service for efficiently collecting（收集）, aggregating聚合）, and moving（移动） large amounts of log data大数据）. It has a simple and flexible architecture based on streaming data flows. It is robust and fault tolerant with tunable reliability mechanisms and many failover and recovery mechanisms. It uses a simple extensible data model that allows for online analytic application.**

官网简介：Flume是一种分布式，可靠且可用的服务，用于有效地收集，聚合和移动大量日志数据。它具有基于流数据流的简单灵活的体系结构。它具有可调整的可靠性机制以及许多故障转移和恢复机制，具有强大的功能和容错能力。它使用一个简单的可扩展数据模型，允许在线分析应用程序。

### 概述

Flume是一个分布式、可靠、和高可用的海量日志聚合的系统，支持在系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。

flume 作为 cloudera 开发的实时日志收集系统，受到了业界的认可与广泛应用。Flume 初始的发行版本目前被统称为 Flume OG（original generation），属于 cloudera。

但随着 FLume 功能的扩展，Flume OG 代码工程臃肿、核心组件设计不合理、核心配置不标准等缺点暴露出来，尤其是在 Flume OG 的最后一个发行版本 0.9.4. 中，日志传输不稳定的现象尤为严重，为了解决这些问题，2011 年 10 月 22 号，cloudera 完成了 Flume-728，对 Flume 进行了里程碑式的改动：重构核心组件、核心配置以及代码架构，重构后的版本统称为 Flume NG（next generation）；改动的另一原因是将 Flume 纳入 apache 旗下，cloudera Flume 改名为 Apache Flume。

Flume的版本：

- Flume-og：Flume0.X版本。在分布式和线程并发上做的并不好 
- Flume-ng：Flume1.X版本，能够较好的支持线程的并发

## 参考资料

官方网站： http://flume.apache.org/
用户文档： http://flume.apache.org/FlumeUserGuide.html
开发文档： http://flume.apache.org/FlumeDeveloperGuide.html

## 设计目标

**(1) 可靠性**

当节点出现故障时，日志能够被传送到其他节点上而不会丢失。Flume提供了三种级别的可靠性保障，从强到弱依次分别为：end-to-end（收到数据agent首先将event写到磁盘上，当数据传送成功后，再删除；如果数据发送失败，可以重新发送。），Store on failure（这也是scribe采用的策略，当数据接收方crash时，将数据写到本地，待恢复后，继续发送），Best effort（数据发送到接收方后，不会进行确认）。

**(2) 可扩展性**

Flume采用了三层架构，分别为agent，collector和storage，每一层均可以水平扩展。其中，所有agent和collector由master统一管理，这使得系统容易监控和维护，且master允许有多个（使用ZooKeeper进行管理和负载均衡），这就避免了单点故障问题。

**(3) 可管理性**

所有agent和colletor由master统一管理，这使得系统便于维护。多master情况，Flume利用ZooKeeper和gossip，保证动态配置数据的一致性。用户可以在master上查看各个数据源或者数据流执行情况，且可以对各个数据源配置和动态加载。Flume提供了web 和shell script command两种形式对数据流进行管理。

**(4) 功能可扩展性**

用户可以根据需要添加自己的agent，collector或者storage。此外，Flume自带了很多组件，包括各种agent（file， syslog等），collector和storage（file，HDFS等）。

## 核心概念

#### Client

Client生产数据，运行在一个独立的线程。

#### Event

 一个数据单元，消息头和消息体组成。（Events可以是日志记录、 avro 对象等。）

#### Flow

 Event从源点到达目的点的迁移的抽象。

#### Agent

 一个独立的Flume进程，包含组件Source、 Channel、 Sink。（Agent使用JVM 运行Flume。每台机器运行一个agent，但是可以在一个agent中包含多个sources和sinks。）

#### Source

 数据收集组件。（source从Client收集数据，传递给Channel）

#### Channel

 中转Event的一个临时存储，保存由Source组件传递过来的Event。（Channel连接 sources 和 sinks ，这个有点像一个队列。）

#### Sink

从Channel中读取并移除Event， 将Event传递到FlowPipeline中的下一个Agent（如果有的话）（Sink从Channel收集数据，运行在一个独立线程。）

## Flume架构

![flume](png\flume\flume.png)

Flume采用了**分层架构：分别为agent，collector和storage**。其中，agent和collector均由两部分组成：**source和sink，source是数据来源，sink是数据去向**。

### Agent

Flume 运行的核心是 Agent。Flume以agent为最小的独立运行单位。一个agent就是一个JVM。它是一个完整的数据收集工具，含有三个核心组件，分别是source、 channel、 sink。通过这些组件， Event 可以从一个地方流向另一个地方。

### Source

Source是数据的收集端，负责将数据捕获后进行特殊的格式化，将数据封装到事件（event） 里，然后将事件推入Channel中。 Flume提供了很多内置的Source， 支持 Avro， log4j， syslog 和 http post(body为json格式)。可以让应用程序同已有的Source直接打交道，如AvroSource，SyslogTcpSource。 如果内置的Source无法满足需要， Flume还支持自定义Source。

### Channel

Channel是连接Source和Sink的组件，大家可以将它看做一个数据的缓冲区（数据队列），它可以将事件暂存到内存中也可以持久化到本地磁盘上， 直到Sink处理完该事件。介绍两个较为常用的Channel， MemoryChannel和FileChannel。

### Sink

Sink从Channel中取出事件，然后将数据发到别处，可以向文件系统、数据库、 hadoop存数据， 也可以是其他agent的Source。在日志数据较少时，可以将数据存储在文件系统中，并且设定一定的时间间隔保存数据。

## Flume拦截器、数据流以及可靠性

### Flume拦截器

　　当我们需要对数据进行过滤时，除了我们在Source、 Channel和Sink进行代码修改之外， Flume为我们提供了拦截器，拦截器也是chain形式的。

　　拦截器的位置在Source和Channel之间，当我们为Source指定拦截器后，我们在拦截器中会得到event，根据需求我们可以对event进行保留还是抛弃，抛弃的数据不会进入Channel中。

### Flume数据流

Flume 的核心是把数据从数据源收集过来，再送到目的地。为了保证输送一定成功，在送到目的地之前，会先缓存数据，待数据真正到达目的地后，删除自己缓存的数据。
Flume 传输的数据的基本单位是 Event，如果是文本文件，通常是一行记录，这也是事务的基本单位。 Event 从 Source，流向 Channel，再到 Sink，本身为一个 byte 数组，并可携带 headers 信息。 Event 代表着一个数据流的最小完整单元，从外部数据源来，向外部的目的地去。

值得注意的是，Flume提供了大量内置的Source、Channel和Sink类型。不同类型的Source,Channel和Sink可以自由组合。组合方式基于用户设置的配置文件，非常灵活。

　　比如：Channel可以把事件暂存在内存里，也可以持久化到本地硬盘上。Sink可以把日志写入HDFS, HBase，甚至是另外一个Source等等。Flume支持用户建立多级流，也就是说，多个agent可以协同工作，并且支持Fan-in、Fan-out、Contextual Routing、Backup Routes，这也正是Flume强大之处。

### Flume可靠性

Flume 使用事务性的方式保证传送Event整个过程的可靠性。 Sink 必须在Event 被存入 Channel 后，或者，已经被传达到下一站agent里，又或者，已经被存入外部数据目的地之后，才能把 Event 从 Channel 中 remove 掉。这样数据流里的 event 无论是在一个 agent 里还是多个 agent 之间流转，都能保证可靠，因为以上的事务保证了 event 会被成功存储起来。比如 Flume支持在本地保存一份文件 channel 作为备份，而memory channel 将event存在内存 queue 里，速度快，但丢失的话无法恢复。

# Flume核心组件

Flume主要由3个重要的组件构成：
Source： 完成对日志数据的收集，分成transtion 和 event 打入到channel之中 Flume提供了各种source的实现，包括Avro Source、 Exce Source、 Spooling Directory Source、 NetCat Source、 Syslog Source、 Syslog TCP Source、Syslog UDP Source、 HTTP Source、 HDFS Source， etc。
Channel： Flume Channel主要提供一个队列的功能，对source提供中的数据进行简单的缓存。
　　　　 Flume对于Channel， 则提供了Memory Channel、 JDBC Chanel、 File Channel，etc

Sink： Flume Sink取出Channel中的数据，进行相应的存储文件系统，数据库，或者提交到远程服务器。
　　　　包括HDFS sink、 Logger sink、 Avro sink、 File Roll sink、 Null sink、 HBasesink， etc。

## Source

　　Spool Source 如何使用？
　　在实际使用的过程中，可以结合log4j使用，使用log4j的时候，将log4j的文件分割机制设为1分钟一次，将文件拷贝到spool的监控目录。

　　 log4j有一个TimeRolling的插件，可以把log4j分割的文件到spool目录。基本实现了实时的监控。 Flume在传完文件之后，将会修 改文

　　件的后缀，变为.COMPLETED（后缀也可以在配置文件中灵活指定）

　　Exec Source 和Spool Source 比较
　　1） ExecSource可以实现对日志的实时收集，但是存在Flume不运行或者指令执行出错时，将无法收集到日志数据，无法何证日志数据

　　　　的完整性。
　　2） SpoolSource虽然无法实现实时的收集数据，但是可以使用以分钟的方式分割文件，趋近于实时。
　　3）总结：如果应用无法实现以分钟切割日志文件的话，可以两种 收集方式结合使用。

## Channel

　　1）MemoryChannel可以实现高速的吞吐， 但是无法保证数据完整性
　　2）MemoryRecoverChannel在官方文档的建议上已经建义使用FileChannel来替换。
　　　　FileChannel保证数据的完整性与一致性。在具体配置不现的FileChannel时，建议FileChannel设置的目录和程序日志文件保存的目录

　　　　设成不同的磁盘，以便提高效率。 

## Sink

　　Flume Sink在设置存储数据时，可以向文件系统中，数据库中， hadoop中储数据，在日志数据较少时，可以将数据存储在文件系中，并

　　且设定一定的时间间隔保存数据。在日志数据较多时，可以将相应的日志数据存储到Hadoop中，便于日后进行相应的数据分析。 





# 基本概念

## Event

1. Flume将收集到的每一条日志封装成了一个Event对象，所以一个Event对象就是一条日志

   > 可以说：Flume中流动的是Event

2. Event本质上就是一个json串，即Flume收集到日志之后，会将日志封装成一个json串。一个Event固定的包含2个部分

   - headers
   - body

## Agent

1. Agent是Flume构成的基本结构，固定的包含3个组件

   - Source：从数据源采集数据
   - Channel：临时存储数据
   - Sink：将数据写到目的地

   可以选择是否添加Sinkgroup




# 流动模型

## 单级流动

![](https://note.youdao.com/yws/api/personal/file/BA513F3E75BB491B852A42FD5B66B47E?method=download&shareKey=0b4bfb663d26a883c353de47427af05c)

## 多级流动

![](https://note.youdao.com/yws/api/personal/file/75298BF9F4B4428EB530A05443FD0D06?method=download&shareKey=92d12fb794e4c05e974f8f7e3c9fca36)

## 扇入流动

![](https://note.youdao.com/yws/api/personal/file/B2E52E0F98E846A5901647999DCBC42D?method=download&shareKey=5e6f6cc10eedecf7bdf24116575d9d48)

## 扇出流动

![](https://note.youdao.com/yws/api/personal/file/4C006111DFAA447F81C779ECAF9B8488?method=download&shareKey=1a0ef6860c26b5d8b3715e3e0db92928)

## 复杂流动

![](https://note.youdao.com/yws/api/personal/file/2DD5A10976FC408A8FBDC04C5DEEAB9C?method=download&shareKey=599cdb40913e2af71c4e6fb2a5bd6a18)





# Flume安装

1. 安装JDK 1.6以上，建议JDK 1.7 或者 JDK 1.8

2. 下载/上传Flume的安装包

   > 在`/home/software/`目录下

   ```sh
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/apache-flume-1.7.0-bin.tar.gz
   ```

3. 解压安装

   ```sh
   tar -xvf apache-flume-1.7.0-bin.tar.gz
   ```

4. 在conf目录下，创建一个配置文件，比如：`template.conf`(名字可以不固定，后缀也可以不固定)，添加如下配置

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1 
   a1.sinks=s1  
   
   #描述/配置a1的r1
   a1.sources.r1.type=netcat  
   a1.sources.r1.bind=0.0.0.0  
   a1.sources.r1.port=8090
   
   #描述a1的s1
   a1.sinks.s1.type=logger   
   
   #描述a1的c1
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1  
   a1.sinks.s1.channel=c1 
   ```

5. 在当前目录执行以下命令

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f template.conf  -Dflume.root.logger=INFO,console
   ```

6. 复制一个窗口，通过nc来访问

   ```sh
   nc hadoop01 8090
   hello flume
   ```

   或者通过外部http请求访问对应端口，比如：`http://10.42.0.33:8090/hello`

7. 查看是否有日志信息打印



# AVRO Source

## 概述

1. 监听Avro 端口来接收外部avro客户端的事件流
2. avro-source接收到的是经过avro序列化后的数据，然后反序列化数据继续传输。
3. 源数据必须是经过avro序列化后的数据
4. 利用Avro source可以实现**多级流动、扇出流、扇入流**等效果
5. 可以接收通过flume提供的avro客户端发送的日志信息

## 可配置选项说明

> 高亮属性为必选项

| **配置项**     | **说明**             |
| -------------- | -------------------- |
| ==channels==   | 绑定通道             |
| ==type==       | avro                 |
| ==bind==       | 需要监听的主机名或IP |
| ==port==       | 要监听的端口         |
| threads        | 工作线程最大线程数   |
| selector.*     | 选择器配置           |
| interceptors.* | 拦截器配置           |



## 示例

1. 在Flume目录下的data目录下创建`avrosource.conf`文件

2. 根据指定的配置文件启动Flume，配置文件内容如下：

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的source1
   a1.sources.r1.type=avro  
   a1.sources.r1.bind=0.0.0.0
   a1.sources.r1.port=8090
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sinka
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

3. 执行启动命令

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f avrosource.conf  -Dflume.root.logger=INFO,console
   ```

   如果出现`Avro source source1 started`，说明启动成功

4. 新窗口进入Flume的bin目录，创建文件`a.txt`

   ```sh
   cd /home/software/apache-flume-1.7.0-bin/bin
   vim a.txt
   	#文件内容：hello,i'm flume
   ```

5. 将文件序列化之后发送给Flume

   ```sh
   sh flume-ng avro-client -H hadoop01 -p 8090 -c ../conf -F a.txt
   ```

6. Flume日志就会打印该文件的内容日志



# Exec Source

## 概述

可以将命令产生的输出作为源来进行传递

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明           |
| -------------- | -------------- |
| ==channels==   | 绑定的通道     |
| ==type==       | exec           |
| ==command==    | 要执行的命令   |
| selector.*     | 选择器配置     |
| interceptors.* | 拦截器列表配置 |

## 示例

1. 在flume的data目录下创建文件：`execsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的source1
   a1.sources.r1.type=exec  
   a1.sources.r1.command=ls /home
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动Flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f execsource.conf  -Dflume.root.logger=INFO,console
   ```

3. flume日志信息

   ![](https://note.youdao.com/yws/api/personal/file/11701248C6B743F2905B3CD12A3E81B1?method=download&shareKey=65789f23a8e33558dfa8ad52f38070a2)



# Spooling Directory Source

## 概述

1. flume会持续监听指定的目录，把放入这个目录中的文件当做Source来处理
2. 注意：一旦文件被放到“自动收集”目录中后，便必能修改，如果修改，flume会报错
3. 此外，也不能有重名的文件，如果有，flume也会报错

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明                         |
| -------------- | ---------------------------- |
| ==channels==   | 绑定通道                     |
| ==type==       | spooldir                     |
| ==spoolDir==   | 读取文件的路径，即"搜集目录" |
| selector.*     | 选择器配置                   |
| interceptors.* | 拦截器配置                   |

## 示例

1. 在flume的data目录中创建文件：`spooldirsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的source1
   a1.sources.r1.type=spooldir
   a1.sources.r1.spoolDir=/home/data
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 创建搜集目录

   ```sh
   cd /home/
   mkdir data
   ```

3. 启动flum

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f spooldirsource.conf  -Dflume.root.logger=INFO,console
   ```

4. 向搜集目录添加文件

   ```sh
   [root@hadoop01 home]# cp a.txt data 
   ```

5. 查看flume控制台日志打印信息



# NetCat Source

## 概述

1. 一个NetCat Source用来监听一个指定端口，并接收监听到的数据
2. 接收的数据是字符串形式

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明                 |
| -------------- | -------------------- |
| ==channels==   | 绑定通道             |
| ==type==       | netcat               |
| ==port==       | 指定要绑定到的端口号 |
| selector.*     | 选择器配置           |
| interceptors.* | 拦截器配置           |

## 示例

1. 在flume的data目录下创建文件：`ncsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的r1
   a1.sources.r1.type=netcat
   a1.sources.r1.bind=0.0.0.0
   a1.sources.r1.port=8090
   
   #描述a1的s1
   a1.sinks.s1.type=logger
   
   #描述a1的c1
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f ncsource.conf -Dflume.root.logger=INFO,console
   ```

3. 新窗口通过nc发送消息

   ```sh
   nc hadoop01 8090
   hello
   ```

4. 查看flume控制台打印的日志信息



# Sequence Generator Source

## 概述

1. 一个简单的序列发生器，不断的产生时间，值从0开始，每次递增1
2. ==主要用来测试==

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明               |
| -------------- | ------------------ |
| ==channels==   | 绑定的通道         |
| ==type==       | seq                |
| selector.*     | 选择器配置         |
| interceptors.* | 拦截器配置         |
| batchSize      | 递增步长， 默认是1 |

## 示例

1. 在flume的data目录下创建文件：`seqsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.sinks=s1
   a1.channels=c1
   
   #描述/配置a1的source1
   a1.sources.r1.type=seq
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动flume

   ```sh
   ../bing/flume-ng agent -n a1 -c ../conf -f seqsource.conf -Dflume.root.logger=INFO,console
   ```



# HTTP Source

## 概述

1. 此Source接收HTTP的GET和POST请求作为Flume的事件
2. GET方式只用于试验，所以实际使用过程中以POST请求居多
3. 如果想让flume正确解析HTTP协议信息，比如解析出请求头、请求体等信息，需要提供一个可插拔的“处理器”来将请求转换为事件对象，这个处理器必须实现HTTPSourceHandler接口。
4. 这个处理器接受一个HttpServletRequest对象，并返回一个Flume Event对象集合

> 常用Handler

1. JSONHandler

   - 可以处理JSON格式的数据，并支持UTF-8     UTF-16 UTF-32字符集

   - 该handler接受Event数组，并根据请求头中指定的编码将其转换为Flume     Event

   - 如果没有指定编码，默认编码为UTF-8

   - 格式：

     ```json
     [
         {
             "headers" : {
                 "timestamp" : "434324343",
                 "host" : "random_host.example.com"
             }
             "body" : "random_body"
         },
         {
             "headers" : {
                 "namenode" : "namenode.example.com",
                 "datanode" : "random_datanode.example.com"
             },
             "body" : "really_random_body"
         }
     ]
     ```

     

2. BlobHandler

   - BlobHandler是一种将请求中上传文件信息转化为event的处理器
   - BlobHandler适合大文件的传输

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明       |
| -------------- | ---------- |
| ==channels==   | 绑定的通道 |
| ==type==       | http       |
| selector.*     | 选择器配置 |
| interceptors.* | 拦截器配置 |
| ==port==       | 端口       |

## 示例

1. 在flume的data目录下创建文件：`httpsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.sinks=s1
   a1.channels=c1
   
   #描述/配置a1的source1
   a1.sources.r1.type=http
   a1.sources.r1.port=8090
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f httpsource.conf -Dflume.root.logger=INFO,console
   ```

3. 新窗口执行curl命令,模拟HTTP的post请求

   ```sh
   curl -X POST -d '[{"headers":{"a":"a1","b":"b1"},"body":"hello http-flume"}]'  http://10.42.0.33:8090
   ```

4. 查看flume控制台日志打印



# 自定义Source

1. Flume中的Source的顶级接口：Source
2. 如果需要自定义Source，那么需要写一个类实现EventDrivenSource或者PollableSource
   - EventDrivenSource：被动型Source。需要自定义线程来获取数据
   - PollableSource：主动型Source。主动提供线程来获取数据
3. 实际场景中，需要根据不同的场景来提供不同的Source
4. 自定义的Source需要打成jar包放到flume安装目录的lib目录下，然后再格式文件中需要给定类的全路径名



## Source类

```java
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.EventDrivenSource;
import org.apache.flume.channel.ChannelProcessor;
import org.apache.flume.conf.Configurable;
import org.apache.flume.event.EventBuilder;
import org.apache.flume.source.AbstractSource;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class AuthSource extends AbstractSource implements EventDrivenSource, Configurable {
    private int step = 1;
    private ExecutorService es;

    /*
        获取格式文件中指定属性的值
        a1.sources.s1.step = xxx
     */
    @Override
    public void configure(Context context) {
        //用户指定的自增的步长
        String step = context.getString("step");
        //  如果用户指定了步长，就要按照用户指定的数值自增。如果用户没有指定步长，默认每次自增1
        if (step != null) {
            this.step = Integer.parseInt(step);
        }
    }

    /*
        启动source
     */
    @Override
    public synchronized void start() {
        System.out.println("Source开始执行");
        es = Executors.newFixedThreadPool(5);
        //获取Channel处理器
        ChannelProcessor cp = super.getChannelProcessor();

        //提交任务，开始执行任务
        es.submit(new Add(step,cp));
    }

    @Override
    public synchronized void stop() {
        es.shutdownNow();
        System.out.println("source执行结束");
    }


}

class Add implements Runnable {

    private int step;
    private ChannelProcessor cp;

    public Add(int step,ChannelProcessor cp) {
        this.cp = cp;
        this.step = step;
    }

    @Override
    public void run() {
        int i = 0;
        while (true) {
            //flume会将收集的数据封装成event
            //构建一个headers
            Map<String, String> headers = new ConcurrentHashMap<>();
            headers.put("tmie",System.currentTimeMillis()+"");
            // 第一个参数：body
            // 第二个参数：headers
            byte[] body = (i + "").getBytes();
            Event e = EventBuilder.withBody(body, headers);

            //将event传递给Channel
            cp.processEvent(e);
            i += step;
        }
    }
}

```



## 打成jar包

![](https://note.youdao.com/yws/api/personal/file/278A4A2791924BCA8D193F3FA74C30C2?method=download&shareKey=157792d198a02ecc3a0bdd729e94aefc)

> Build - Build Artifacts，生成jar包

![](https://note.youdao.com/yws/api/personal/file/707E498512494AA396B9CCDE9D9D91BA?method=download&shareKey=da288ac1cfc549f6c647d20159db8ae9)

> 生成的jar包存在`out -> artifacts -> G_Flume_jar`下

![](https://note.youdao.com/yws/api/personal/file/360D20E5A8B04E97A794A6FC1510303F?method=download&shareKey=a3c4599dbe08faa373547d23d66e5098)

## Flume配置文件

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

# 自定义source的类型要写类的全路径名
a1.sources.s1.type = cn.tedu.source.AuthSource

a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

a1.sinks.k1.type = logger

a1.sources.s1.channels = c1
a1.sinks.k1.channel = c1
```

[TOC]

# Memory Channel

1. 内存通道，将接收到的数据临时存储在内存中
2. 读写速度比较快，但是不可靠



## 可配置选项说明

> 高亮属性为必选项

| 配置项                  | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| ==type==                | memory                                                       |
| ==capacity==            | 默认：100    事件存储在信道中的最大数量  <br />建议实际工作调节：10万     <br />首先估算出每个event的大小，然后再服务的内存来调节 |
| ==transactionCapacity== | 默认：100    每个事务中的最大事件数  <br />建议实际工作调节：1000~3000 |

> capacity：10000-30000	对应transactionCapacity为：1000-3000（1%）

## 示例

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

# 配置Source
a1.sources.s1.type = netcat
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 8090

# 配置Channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

# 配置Sink
a1.sinks.k1.type = logger

# 将Source和Channel绑定
a1.sources.s1.channels = c1

# 将Sink和Channel绑定
a1.sinks.k1.channel = c1
```



# File Channel

1. 文件通道，将接收到的数据临时存储在磁盘上
2. 读写速度比较慢，但是可靠



## 可配置选项说明

> 高亮属性为必选项

| 配置项       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| ==type==     | file                                                         |
| ==dataDirs== | 指定存放的目录，逗号分隔的目录列表，用以存放日志文件。使用单独的磁盘上的多个目录可以提高文件通道效率。 |



## 示例

1. 创建数据存目录

   ```sh
   cd /home
   mkdir flumedata	
   ```

   

2. 在flume的data目录下创建：`filechannel.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   
   # 配置FileChannel
   a1.channels.c1.type = file
   # 指定临时存储的目录
   a1.channels.c1.dataDirs = /home/flumedata/
   
   # 配置Sink
   a1.sinks.k1.type = logger
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   ```

3. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f filechannel.conf -Dflume.root.logger=INFO,console
   ```

4. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

5. 此时flume控制台会打印信息

   ```sh
   2020-03-18 12:29:31,359 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 65 6C 6C 6F 2C 69 27 6D 20 66 6C 75 6D 65    hello,i'm flume }
   2020-03-18 12:29:40,706 (Log-BackgroundWorker-c1) [INFO - org.apache.flume.channel.file.EventQueueBackingStoreFile.beginCheckpoint(EventQueueBackingStoreFile.java:227)] Start checkpoint for /root/.flume/file-channel/checkpoint/checkpoint, elements to sync = 1
   2020-03-18 12:29:40,727 (Log-BackgroundWorker-c1) [INFO - org.apache.flume.channel.file.EventQueueBackingStoreFile.checkpoint(EventQueueBackingStoreFile.java:252)] Updating checkpoint metadata: logWriteOrderID: 1584505750768, queueSize: 0, queueHead: 0
   2020-03-18 12:29:40,745 (Log-BackgroundWorker-c1) [INFO - org.apache.flume.channel.file.Log.writeCheckpoint(Log.java:1052)] Updated checkpoint for file: /home/flumedata/log-1 position: 172 logWriteOrderID: 1584505750768
   ```

   

6. 查看`/home/flumedata`目录，会生成一个`log-x.meta`，该文件记录的是传输的信息







# 其他Channel

## JDBC Channel

1. 事件会被持久化（存储）到可靠的数据库里
2. 目前只支持嵌入式Derby数据库。但是Derby数据库不太好用，所以JDBC Channel目前仅用于测试，不能用于生产环境。



## 内存溢出通道

1. 优先把Event存到内存中，如果存不下，在溢出到文件中
2. 目前处于测试阶段，还未能用于生产环境

[TOC]

# Logger Sink

1. 将数据以日志形式打印
2. 在打印的时候为了防止屏幕显示的内容过多，每次打印body内容不超过16个字节
3. 打印的时候需要指定级别，例如：INFO、DEBUG、WANGN、ERROR等



## 可配置项说明

> 高亮属性为必选项

| 配置项      | 说明     |
| ----------- | -------- |
| ==channel== | 绑定通道 |
| ==type==    | logger   |



## 示例

1. 在flume的data目录下创建：`loggersink.conf`

   ```sh
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的r1
   a1.sources.r1.type=netcat
   a1.sources.r1.bind=0.0.0.0
   a1.sources.r1.port=8090
   
   #描述a1的s1
   a1.sinks.s1.type=logger
   #描述a1的c1
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   ```

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f loggersink.conf -Dflume.root.logger=INFO,console
   ```

3. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

   



# File_roll Sink

1. 将收集的数据写到指定的目录下
2. 需要指定文件滚动事件的时间，不指定会默认30s生成一个小文件



## 可配置项说明

> 高亮属性为必选项

| 配置项             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| ==channel==        | 绑定通道                                                     |
| ==type==           | file_roll                                                    |
| ==sink.directory== | 文件被存储的目录                                             |
| sink.rollInterval  | 30 记录日志到文件里，每隔30秒生成一个新日志文件。如果设置为0，则禁止滚动，从而导致所有数据被写入到一个文件中。 |



## 示例

1. 创建数据收集目录

   ```sh
   cd /home
   mkdir flumedata
   ```

   

2. 在flume的data目录下创建：`filerollsink.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   # 配置Sink
   a1.sinks.k1.type = file_roll
   a1.sinks.k1.sink.directory = /home/flumedata
   # 如果不指定，默认每隔30s生成一个文件
   a1.sinks.k1.sink.rollInterval = 3600
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   
   ```

3. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f filerollsink.conf -Dflume.root.logger=INFO,console
   ```

4. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

5. 查看数据数收集目录：`/home/flumedata`，会生成文件，内容为传输的数据

   ```sh
   cat 1584506172023-1 
   hello
   ```

   




# HDFS Sink

1. 将数据收集来之后写到HDFS中
2. 数据往HDFS中写的时候只支持三种格式：
   - 序列：`SequenceFile`
   - 文本：`DataStream`
   - 压缩：`CompressedStream`
3. ==使用的时候最好指定滚动时间==，不指定默认30s生成一个小文件，导致HDFS上产生大量的小文件



## 可配置选项说明

> 高亮属性为必选项

| 配置项                | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| ==channel==           | 绑定的通道                                                   |
| ==type==              | hdfs                                                         |
| ==hdfs.path==         | HDFS 目录路径  （hdfs://namenode/flume/webdata/)             |
| hdfs.inUseSuffix      | .tmp    Flume正在处理的文件所加的后缀                        |
| ==hdfs.rollInterval== | 文件生成的间隔事件，默认是30，单位是秒                       |
| hdfs.rollSize         | 生成的文件大小，默认是1024个字节 ，0表示不开启此项           |
| hdfs.rollCount        | 每写几条数据就生成一个新文件，默认数量为10  每写几条数据就生成一个新文件， |
| ==hdfs.fileType==     | SequenceFile/DataStream/CompressedStream                     |
| hdfs.retryInterval    | 80    Time  in seconds between consecutive attempts to close a file. Each close call  costs multiple RPC round-trips to the Namenode, so setting this too low can  cause a lot of load on the name node. If set to 0 or less, the sink will not  attempt to close the file if the first attempt fails, and may leave the file  open or with a ”.tmp” extension. |



## 示例

1. 启动Hadoop伪分布式

   ```sh
   start-all.sh
   ```

2. 在flume的data目录下创建：`hdfssink.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   # 配置Sink
   a1.sinks.k1.type = hdfs
   a1.sinks.k1.hdfs.path = hdfs://hadoop01:9000/flume
   a1.sinks.k1.hdfs.fileType = DataStream
   a1.sinks.k1.hdfs.rollInterval = 3600
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   ```

3. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f hdfssink.conf -Dflume.root.logger=INFO,console
   ```

4. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   lunch
   OK
   breakfast
   OK
   ```

5. 查看HDFS文件，在`/flume`目录下会生成一个数据文件，内容为输入的数据

   ```
   lunch
   breakfast
   ```

   



# Avro Sink

1. 将源数据利用AVRO序列化后写到指定的节点上
2. 将数据序列化之后写出，结合AVRO Source来实现**多级流动**、**扇入流动**、**扇出流动**的效果

## 可配置选项说明

> 高亮属性为必选项

| 配置项       | 说明           |
| ------------ | -------------- |
| ==channel==  | 绑定的通道     |
| ==type==     | avro           |
| ==hostname== | 要发送的主机   |
| ==port==     | 要发往的端口号 |

## 示例

### 多级流动

![](https://note.youdao.com/yws/api/personal/file/22B6EC52909945D293C7CAF7B249F5DB?method=download&shareKey=db3cb0f371cc39bdaf7b8f98cba8fa6c)

> 多级流动案例。hadoop01   ->  hadoop02  ->  hadoop03 --日志打印输出

1. 在flume的data目录下创建`duoji.conf`配置文件

   - hadoop01的`duoji.conf`

     ```sh
     a1.sources = s1
     a1.channels = c1
     a1.sinks = k1
     
     # 配置Source
     a1.sources.s1.type = netcat
     a1.sources.s1.bind = 0.0.0.0
     a1.sources.s1.port = 8090
     
     # 配置Channel
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 10000
     a1.channels.c1.transactionCapacity = 100
     
     # 配置Sink
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop02
     a1.sinks.k1.port = 8090
     
     # 将Source和Channel绑定
     a1.sources.s1.channels = c1
     
     # 将Sink和Channel绑定
     a1.sinks.k1.channel = c1
     ```

     

   - hadoop02的`duoji.conf`

     ```sh
     a1.sources = s1
     a1.channels = c1
     a1.sinks = k1
     
     # 配置Source
     a1.sources.s1.type = avro
     a1.sources.s1.bind = 0.0.0.0
     a1.sources.s1.port = 8090
     
     # 配置Channel
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 10000
     a1.channels.c1.transactionCapacity = 100
     
     # 配置Sink
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop03
     a1.sinks.k1.port = 8090
     
     # 将Source和Channel绑定
     a1.sources.s1.channels = c1
     
     # 将Sink和Channel绑定
     a1.sinks.k1.channel = c1
     ```

     

   - hadoop03的`duoji.conf`

     ```sh
     a1.sources = s1
     a1.channels = c1
     a1.sinks = k1
     
     # 配置Source
     a1.sources.s1.type = avro
     a1.sources.s1.bind = 0.0.0.0
     a1.sources.s1.port = 8090
     
     # 配置Channel
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 10000
     a1.channels.c1.transactionCapacity = 100
     
     # 配置Sink
     a1.sinks.k1.type = logger
     
     # 将Source和Channel绑定
     a1.sources.s1.channels = c1
     
     # 将Sink和Channel绑定
     a1.sinks.k1.channel = c1
     ```

2. 倒序启动flume

   ```sh
   #hadoop03
   ../bin/flume-ng agent -n a1 -c ../conf -f duoji.conf -Dflume.root.logger=INFO,console
   #hadoop02
   ../bin/flume-ng agent -n a1 -c ../conf -f duoji.conf -Dflume.root.logger=INFO,console
   #hadoop01
   ../bin/flume-ng agent -n a1 -c ../conf -f duoji.conf -Dflume.root.logger=INFO,console
   ```

3. 在hadoop01节点通过nc向hadoop01发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

4. 会在hadoop03节点收到传输的avro信息

   ```sh
   2020-03-18 12:01:40,835 (SinkRunner-PollingRunner-DefaultSinkProcessor) 
   	[INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] 
   	Event: { headers:{} body: 68 65 6C 6C 6F                                  hello }
   ```

   

### 扇入流动

![](https://note.youdao.com/yws/api/personal/file/64039AC203C843609816525E6B0662B1?method=download&shareKey=f1031d526037a58c6daaf243a7e0d422)

### 扇出流动

![](https://note.youdao.com/yws/api/personal/file/EAFB58C25ADB4432AC65A8A39C438817?method=download&shareKey=801e001bb844708a9deb1d8f9d6c36a0)



# 自定义Sink

需要去写一个类实现Sink、Configurable，在Sink完成过程中需要注意事务的问题。在这个过程中需要覆盖configure、start、process、stop方法



## Sink

```java
import org.apache.flume.*;
import org.apache.flume.conf.Configurable;
import org.apache.flume.sink.AbstractSink;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.PrintStream;
import java.util.Map;

// 模拟File_roll Sink
public class AuthSink extends AbstractSink implements Sink, Configurable {

    private File file;
    private PrintStream ps;

    @Override
    public void configure(Context context) {
        // 获取属性值
        String dir = context.getString("dir");
        // 判断是否指定了输出的目录
        if (dir == null) {
            throw new NullPointerException("存储的目录路径不能为空！！！");
        }
        file = new File(dir);
    }

    @Override
    public synchronized void start() {
        System.out.println("Sink已经启动~~~");
        // 判断路径是否存在
        if (!file.exists())
            // 如果路径不存在，试着创建这个路径
            file.mkdirs();
        // 初始化流对象
        try {
            ps = new PrintStream(file.getPath() + "/" + System.currentTimeMillis());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        super.start();
    }

    @Override
    public Status process() {

        // 获取Channel
        Channel c = super.getChannel();
        // 获取Transaction
        Transaction t = c.getTransaction();
        // 开启事务
        t.begin();
        // 创建Event
        Event e;
        try {
            while ((e = c.take()) != null) {
                Map<String, String> headers = e.getHeaders();
                byte[] body = e.getBody();
                ps.println("headers:");
                for (Map.Entry<String, String> entry : headers.entrySet()) {
                    ps.println("\t" + entry.getKey() + ":" + entry.getValue());
                }
                ps.println("body:");
                ps.println("\t" + new String(body));
            }
            t.commit();
        } catch (Exception ex) {
            t.rollback();
            ex.printStackTrace();
            return Status.BACKOFF;
        } finally {
            t.close();
        }
        return Status.READY;
    }

    @Override
    public synchronized void stop() {
        ps.close();
        System.out.println("Sink已经执行完了");
        super.stop();
    }
}
```



## Flume配置文件

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

a1.sources.s1.type = netcat
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 8090

a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

a1.sinks.k1.type = cn.tedu.sink.AuthSink
a1.sinks.k1.sink.dir = /home/flumedata
a1.sinks.k1.sink.rollInterval = 3600

a1.sources.s1.channels = c1
a1.sinks.k1.channel = c1
```





# 事务

![](https://note.youdao.com/yws/api/personal/file/4EF741C480EF43CAB0B2AE9669120C0A?method=download&shareKey=3eac90fc9df9241ebcb87b1db2f5c186)

# Selector

1. Selector本身是Source的子组件，需要配置在Source上

2. 模式：

   - replication：复制模式

     > 当第一个节点收到数据的时候，会将这个数据复制然后发送给每一个节点，所以每一个节点的数据是相同的

   - multiplexing：路由模式/多路复用模式

     > 当第一个节点收到数据的时候，会根据这个数据中的指定字段来进行判断，然后转发给不同的节点。所以在这种模式下，每一个节点收到的数据是不同的

3. 如果不指定，默认使用的是复制模式

4. 在实际生产中，如果对数据进行备份，那么使用复制模式；如果对数据分类，那么使用路由模式





## 复制模式

> Selector默认是复制模式（replicating），即把Source复制，然后分发给多个sink

### 可配置选项

| 配置项            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| selector.type     | replicating   表示复制模式，source的selector如果不配置，默认就是这种模式  在复制模式下，当source接收到数据后，会复制多分，分发给每一个avro sink |
| selector.optional | 标志通道为可选                                               |

### 示例

1. hadoop01配置

   ```sh
   a1.sources = s1
   a1.channels = c1 c2
   a1.sinks = k1 k2
   
   # 配置Source
   a1.sources.s1.type = http
   a1.sources.s1.port = 8090
   a1.sources.s1.selector.type = replicating
   a1.sources.s1.selector.optional = c2
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 10000
   a1.channels.c2.transactionCapacity = 100
   
   
   # 配置Sink
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop03
   a1.sinks.k1.port = 8090
   
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop02
   a1.sinks.k2.port = 8090
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1 c2
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   ```

2. hadoop02和hadoop03配置

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.sinks=s1
   a1.channels=c1
   
   #描述/配置a1的source1
   a1.sources.r1.type=avro
   a1.sources.r1.bind=0.0.0.0
   a1.sources.r1.port=8090
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   ```

   



## 多路复用模式

> 在这种模式下，用户可以指定转发的规则。selector根据规则进行数据的分发



### 可配置选项

| 配置项             | 说明                                     |
| ------------------ | ---------------------------------------- |
| selector.type      | `multiplexing` 表示路由模式              |
| selector.header    | 指定要监测的头的名称                     |
| selector.mapping.* | 匹配规则                                 |
| selector.default   | 如果未满足匹配规则，则默认发往指定的通道 |



### 示例

> 01机利用HTTP Source接收数据，根据路由规则，发往02，03机。02，03通过avro source接收数据，通过logger sink打印数据

1. hadoop01机配置

   ```sh
   a1.sources = s1
   a1.channels = c1 c2
   a1.sinks = k1 k2
   
   # 配置Source
   a1.sources.s1.type = http
   a1.sources.s1.port = 8090
   a1.sources.s1.selector.type = multiplexing
   a1.sources.s1.selector.header = class
   a1.sources.s1.selector.mapping.big1910 = c1
   a1.sources.s1.selector.mapping.big1911 = c2
   a1.sources.s1.selector.default = c2
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 10000
   a1.channels.c2.transactionCapacity = 100
   
   
   # 配置Sink
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop03
   a1.sinks.k1.port = 8090
   
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop02
   a1.sinks.k2.port = 8090
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1 c2
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   
   ```

2. hadoop02和hadoop03机配置

   ```sh
   #配置Agent a1 的组件
   a1.sources = r1
   a1.sinks = s1
   a1.channels = c1
   
   #描述/配置a1的source1
   a1.sources.r1.type = avro
   a1.sources.r1.bind = 0.0.0.0
   a1.sources.r1.port = 8090
   
   #描述sink
   a1.sinks.s1.type = logger
   
   #描述内存channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   ```

   

   [TOC]

   

   # Interceptor

   1. 本身是Source的子组件，需要配置在Source上
   2. 可以配置多个，按照配置顺序形成拦截链

   

   ## Timestamp Interceptor

   1. 这个拦截器在事件头中插入以毫秒为单位的当前处理时间
   2. 头的名字为`timestamp`，值为当前处理的时间戳
   3. 如果在之前已经有这个时间戳，则保留原有的时间戳

   > 结合HDFS Sink使用，可以实现按天收集的效果

   ### 可配置项

   | 配置项           | 说明                                |
   | ---------------- | ----------------------------------- |
   | ==type==         | timestamp                           |
   | preserveExisting | false    如果时间戳已经存在是否保留 |

   ### 示例

   1. 配置文件：`timestampin.conf`

      ```sh
      a1.sources = s1
      a1.channels = c1
      a1.sinks = k1
      
      # 配置Source
      a1.sources.s1.type = netcat
      a1.sources.s1.bind = 0.0.0.0
      a1.sources.s1.port = 8090
      a1.sources.s1.interceptors = i1
      a1.sources.s1.interceptors.i1.type = timestamp
      
      # 配置Channel
      a1.channels.c1.type = memory
      a1.channels.c1.capacity = 10000
      a1.channels.c1.transactionCapacity = 100
      
      # 配置Sink
      a1.sinks.k1.type = logger
      
      # 将Source和Channel绑定
      a1.sources.s1.channels = c1
      
      # 将Sink和Channel绑定
      a1.sinks.k1.channel = c1
      
      ```

   2. 启动flume

      ```sh
      ../bin/flume-ng agent -n a1 -c ../conf -f timestampin.conf -Dflume.root.logger=INFO,console
      ```

   3. 通过nc发送数据

      ```sh
      nc hadoop01 8090
      hello
      OK
      ```

   4. flume控制台日志打印

      ```sh
      Event: { headers:{timestamp=1584548779108} body: 68 65 6C 6C 6F                                  hello }
      ```

   

   > 结合HDFS Sink使用，实现按天收集的效果
   >
   > 当天收集的数据会放到一个文件夹内，00:00过后会放入新的文件夹

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i1
   a1.sources.s1.interceptors.i1.type = timestamp
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   # 配置Sink
   a1.sinks.k1.type = hdfs
   a1.sinks.k1.hdfs.path = hdfs://hadoop01:9000/log/reportTime=%Y-%m-%d
   a1.sinks.k1.hdfs.fileType = DataStream
   a1.sinks.k1.hdfs.rollInterval = 3600
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   
   ```

   

   ## Host Interceptor

   > 在日志信息的headers中添加一个host字段

   1. 这个拦截器插入当前Agent的主机名或者IP
   2. 头的名字为host或者配置的名称
   3. 值是主机名或者ip地址，基于配置

   

   ### 可配置项

   | 配置参数         | 说明                                                |
   | ---------------- | --------------------------------------------------- |
   | ==type==         | host                                                |
   | preserveExisting | false    如果主机名已经存在是否保留                 |
   | useIP            | true    如果配置为true则用IP，配置为false则用主机名 |
   | hostHeader       | host    加入头时使用的名称                          |

   

   ### 示例

   1. 配置文件：`hostin.conf`

      ```sh
      a1.sources = s1
      a1.channels = c1
      a1.sinks = k1
      
      # 配置Source
      a1.sources.s1.type = netcat
      a1.sources.s1.bind = 0.0.0.0
      a1.sources.s1.port = 8090
      a1.sources.s1.interceptors = i2
      a1.sources.s1.interceptors.i2.type = host
      
      # 配置Channel
      a1.channels.c1.type = memory
      a1.channels.c1.capacity = 10000
      a1.channels.c1.transactionCapacity = 100
      
      # 配置Sink
      a1.sinks.k1.type = logger
      
      # 将Source和Channel绑定
      a1.sources.s1.channels = c1
      
      # 将Sink和Channel绑定
      a1.sinks.k1.channel = c1
      ```

   2. 启动flume

      ```sh
      ../bin/flume-ng agent -n a1 -c ../conf -f hostin.conf -Dflume.root.logger=INFO,console
      ```

   3. 通过nc发送数据

      ```sh
      nc hadoop01 8090
      hello
      OK
      ```

   4. flume控制台打印的信息

      ```sh
      Event: { headers:{host=10.42.0.33} body: 68 65 6C 6C 6F      hello }
      ```

      

   

   ## Static Interceptor

   > 在日志信息的headers中添加一个指定的字段

   1. 此拦截器允许用户增加静态头信息，使用静态的值到所有事件
   2. 目前的实现中不允许一次指定多个头
   3. 如果需要增加多个静态头，可以指定多个Static Interceptors

   

   ### 可配置项

   | 配置项           | 说明                  |
   | ---------------- | --------------------- |
   | ==type==         | static                |
   | preserveExisting | true                  |
   | ==key==          | key    要增加的头名   |
   | ==value==        | value    要增加的头值 |

   

   ### 示例

   1. flume配置文件：`staticin.conf`

      ```sh
      a1.sources = s1
      a1.channels = c1
      a1.sinks = k1
      
      # 配置Source
      a1.sources.s1.type = netcat
      a1.sources.s1.bind = 0.0.0.0
      a1.sources.s1.port = 8090
      a1.sources.s1.interceptors = i3
      a1.sources.s1.interceptors.i3.type = static
      a1.sources.s1.interceptors.i3.key = subject
      a1.sources.s1.interceptors.i3.value = bigdata
      
      # 配置Channel
      a1.channels.c1.type = memory
      a1.channels.c1.capacity = 10000
      a1.channels.c1.transactionCapacity = 100
      
      # 配置Sink
      a1.sinks.k1.type = logger
      
      # 将Source和Channel绑定
      a1.sources.s1.channels = c1
      
      # 将Sink和Channel绑定
      a1.sinks.k1.channel = c1
      ```

   2. 启动flume

      ```sh
      ../bin/flume-ng agent -n a1 -c ../conf -f staticin.conf -Dflume.root.logger=INFO,console
      ```

   3. 通过nc发送数据

      ```sh
      nc hadoop1 8090
      hi
      OK
      ```

   4. flume控制台打印信息

      ```sh
      Event: { headers:{subject=bigdata} body: 68 69        hi }
      ```

      

   

   

   ## UUID Interceptor

   > 在日志信息的headers中添加一个id，作用往往是标记数据的唯一性，保证数据不重复

   1. 这个拦截器在所有事件头中增加一个全局一致性的标志，就是UUID

   

   ### 可配置项

   | 配置项           | 说明                                                         |
   | ---------------- | ------------------------------------------------------------ |
   | ==type==         | `org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder  ` |
   | headerName       | `id`：头名称                                                 |
   | preserveExisting | `true`：如果头已经存在，是否保留                             |
   | prefix           | `""`：在UUID前拼接的字符串前缀                               |

   

   ### 示例

   1. flume配置文件：`uuidin.conf`

      ```sh
      a1.sources = s1
      a1.channels = c1
      a1.sinks = k1
      
      # 配置Source
      a1.sources.s1.type = netcat
      a1.sources.s1.bind = 0.0.0.0
      a1.sources.s1.port = 8090
      a1.sources.s1.interceptors = i4
      a1.sources.s1.interceptors.i4.type = org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder
      
      # 配置Channel
      a1.channels.c1.type = memory
      a1.channels.c1.capacity = 10000
      a1.channels.c1.transactionCapacity = 100
      
      # 配置Sink
      a1.sinks.k1.type = logger
      
      # 将Source和Channel绑定
      a1.sources.s1.channels = c1
      
      # 将Sink和Channel绑定
      a1.sinks.k1.channel = c1
      
      ```

   2. 启动flume

      ```sh
      ../bin/flume-ng agent -n a1 -c ../conf -f uuidin.conf -Dflume.root.logger=INFO,console
      ```

   3. 通过nc发送数据

      ```sh
      nc hadoop01 8090
      nihao
      OK
      ```

   4. flume控制台打印信息

      ```sh
      Event: { headers:{id=e230e125-56de-4e15-a8ee-a02b408d698a} body: 6E 69 68 61 6F       nihao }
      ```

      

   

   ## Search And Replace Interceptor

   > 会根据指定的正则表达式去搜索Body中的数据，将符合正则表达式的数据进行替换

   1. 这个拦截器提供了简单的基于字符串的正则搜索和替换功能

   

   ### 可配置项

   | 配置项            | 说明                           |
   | ----------------- | ------------------------------ |
   | ==type==          | search_replace                 |
   | ==searchPattern== | 要搜索和替换的正则表达式       |
   | ==replaceString== | 要替换为的字符串               |
   | charset           | UTF-8    字符集编码，默认utf-8 |

   

   ### 示例

   1. flume的配置文件：`searchreplace.conf`

      ```sh
      a1.sources = s1
      a1.channels = c1
      a1.sinks = k1
      
      a1.sources.s1.type = http
      a1.sources.s1.port = 8090
      a1.sources.s1.interceptors = i1
      a1.sources.s1.interceptors.i1.type = search_replace
      a1.sources.s1.interceptors.i1.searchPattern = [0-9]
      a1.sources.s1.interceptors.i1.replaceString = *
      
      a1.channels.c1.type = memory
      a1.channels.c1.capacity = 10000
      a1.channels.c1.transactionCapacity = 100
      
      a1.sinks.k1.type = logger
      
      a1.sources.s1.channels = c1
      a1.sinks.k1.channel = c1
      ```

   2. 启动flume

      ```sh
      ../bin/flume-ng agent -n a1 -c ../conf -f searchreplace.conf -Dflume.root.logger=INFO,console
      ```

   3. 通过curl发送http请求

      ```sh
      curl -X POST -d '[{"headers":{"class":"bigdata"},"body":"data123456"}]' http://hadoop01:8090
      ```

   4. flume控制台打印信息

      ```sh
      Event: { headers:{class=big1910} body: 62 69 67 2A 2A 2A 2A    data****** }
      ```

      

   

   

   ## Regex Filtering Interceptor

   > 需要制定一个正则表达式，只要符合正则表达式的数据就会被拦截掉

   1. 此拦截器通过解析事件体去匹配给定正则表达式来筛选事件
   2. 所提供的正则表达式即可以用来包含或剔除事件

   

   ### 可配置项

   | 配置项        | 说明                                                         |
   | ------------- | ------------------------------------------------------------ |
   | ==type==      | `regex_filter`                                               |
   | ==regex==     | `".*"`： 所要匹配的正则表达式                                |
   | excludeEvents | ` false`：如果是true则刨除匹配的事件，false则包含匹配的事件。 |

   

   ### 示例

   1. flume配置文件：`regexfilter.conf`

      ```sh
      a1.sources = s1
      a1.channels = c1
      a1.sinks = k1
      
      # 配置Source
      a1.sources.s1.type = netcat
      a1.sources.s1.bind = 0.0.0.0
      a1.sources.s1.port = 8090
      a1.sources.s1.interceptors = i1
      a1.sources.s1.interceptors.i1.type = regex_filter
      a1.sources.s1.interceptors.i1.regex = .*[0-9].*
      a1.sources.s1.interceptors.i1.excludeEvents = true
      
      # 配置Channel
      a1.channels.c1.type = memory
      a1.channels.c1.capacity = 10000
      a1.channels.c1.transactionCapacity = 100
      
      # 配置Sink
      a1.sinks.k1.type = logger
      
      # 将Source和Channel绑定
      a1.sources.s1.channels = c1
      
      # 将Sink和Channel绑定
      a1.sinks.k1.channel = c1
      ```

   2. 启动flume

      ```sh
      ../bin/flume-ng agent -n a1 -c ../conf -f regexfilter.conf -Dflume.root.logger=INFO,console
      ```

   3. 通过curl发送HTTP请求

      ```sh
      curl -X POST -d '[{"headers":{"class":"big1910"},"body":"big1910abc"}]' http://hadoop01:8090
      ```

   4. body含有数字的都被过滤掉了，所以没有日志打印

      ```sh
      2020-03-19 01:33:07,894 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 41 63 63 65 70 74 3A 20 2A 2F 2A 0D             Accept: */*. }
      2020-03-19 01:33:07,894 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 43 6F 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 Content-Type: ap }
      2020-03-19 01:33:07,894 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 0D                                              . }
      ```

      

   [TOC]

   

   > Processor：控制器

   # 概述

   1. 在Flume中，如果不指定，那么默认一个Sink就对应一个Sinkgroup，允许将一个或者多个Sink绑定在一个Sinkgroup
   2. Flume Sink Processor 可以通过切换组内Sink用来实现负载均衡的效果，或在一个Sink故障时切换到另一个Sink

   ![](https://note.youdao.com/yws/api/personal/file/35730BF7E3D548E8A2673E31F6BF0C70?method=download&shareKey=1309d6d45fd2ffad3ae42846f73be07e)

   # Default Sink Processor

   1. 只接受一个Sink
   2. 这是默认的策略，即如果==不配置Processor用的是这个策略==

   

   ## 可配置选项

   | 配置项             | 说明                 |
   | ------------------ | -------------------- |
   | ==sinks==          | 用空格分隔的Sink集合 |
   | ==processor.type== | default              |

   

   

   # Failover Sink Processor

   > 使用的时候需要指定优先级，只要优先级高的Sink存活，那么优先级低的Sink就收不到数据

   1. 维护一个sink们的优先表。确保只要一个Sink是可用的就能正常处理

   2. 失败处理原理：为失效的Sink指定一个冷却时间，在冷却时间到达后再重新使用

   3. Sink们可以被配置一个优先级，==数字越大优先级越高==

   4. 如果Sink发送事件失败，则下一个最高优先级的Sink将会尝试接着发送该事件

   5. 如果没有指定优先级，则优先级顺序取决于Sink们的配置顺序，先配置的默认优先级高

   6. 在配置的过程中，设置一个`group processor`，并未每一个Sink都指定一个优先级

      > 优先级必须是唯一的

   7. 另外可以设置`maxpenalty`属性指定限定失败时间

   

   ## 可配置项

   | 配置项               | 说明                                                         |
   | -------------------- | ------------------------------------------------------------ |
   | ==sinks==            | 绑定的sink                                                   |
   | ==processor.type==   | failover                                                     |
   | processor.priority   | 设置优先级，注意，每个sink的优先级必须是唯一的               |
   | processor.maxpenalty | 30000    The  maximum backoff period for the failed Sink (in millis) |

   

   ## 示例

   ```sh
   a1.sources = s1
   a1.channels = c1 c2
   a1.sinks = k1 k2
   
   a1.sinkgroups = g1
   a1.sinkgroups.g1.sinks = k1 k2
   a1.sinkgroups.g1.processor.type = failover
   a1.sinkgroups.g1.processor.priority.k1 = 7
   a1.sinkgroups.g1.processor.priority.k2 = 3
   a1.sinkgroups.g1.processor.maxpenalty = 100000
   
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 10000
   a1.channels.c2.transactionCapacity = 100
   
   a1.sources.s1.channels = c1 c2
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   ```

   

   

   

   

   

   