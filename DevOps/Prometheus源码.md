Prometheus 是云原生监控领域的事实标准，越来越多的开源项目开始支持 Prometheus 监控数据格式。从本篇开始，我将和大家一起阅读分析 Prometheus 源码。学习 Prometheus 的设计理念，了解 Prometheus 的局限性与不足。本系列分八个板块逐一拆解 Prometheus 源码。本文基于 Prometheus v2.13.0。

- 工作原理与架构
- 时序数据库模块（TSDB）
- 配置文件加载模块（Configuration Reloader）
- 服务发现模块（Service Discovery Manager）
- 数据抓取模块（Scrape Manager）
- RES API 模块（Web Handler）
- 查询引擎（Query Engine & PromQL）
- 性能与优劣势总结

***1\***|***1\*****1. Prometheus 介绍**
Prometheus 是基于时序 [数据库](http://www.wityx.com/database/)（Time Series Database, TSDB）的监控告警系统。Prometheus 在 2016 年加入 CNCF 基金会，成为继 Kubernetes 后第二个毕业项目，其火热程度可见一斑。相比于传统的监控方案，Prometheus 有以下几个优势：

- 高效的存储引擎

Prometheus 将监控指标以时序数列的格式存储在时序数据库 TSDB 中。相比于传统的关系型数据库，时序数据库便于对已有数据进行聚合；在高并发的情况下，读写性能也远高于关系型数据库。Prometheus 2.0 版本重构了底层时序存储引擎。目前，单个 Prometheus 服务器可以做到每秒存储百万条指标数据，同时占用磁盘空间也很小

- 强大的查询能力：PromQL

Prometheus 有独立的 PromQL 查询语言，另外还提供了很多内置的基于时间的处理函数，降低数据聚合的难度

- 面向服务的架构

Prometheus 采用拉模型收集时序数据，数据拉取行为是由服务端来决定的。服务端可以通过某种服务发现机制来自动发现监控对象。而对于推模型的监控系统，客户端需要负责在服务端上进行注册及监控数据推送，这在微服务架构里实现起来比较难的。当大量客户端向服务的主动推送数据时，服务端的压力较大

- 与 Kubernetes 天然集成

Kubernetes 本身的指标也是以 Prometheus 格式暴露出来的

- 逐步完善的生态

OpenMetrics：Prometheus 的数据格式逐渐成为一种标准。OpenMetrics 正在从 Prometheus 的数据格式中分离出来，逐渐成为监控数据格式的国际标准

Thanos：支持数据存储的可伸缩，弥补 Prometheus 数据持久化方面的不足Prometheus

Prometheus Operator：简化 Prometheus 配置管理

***1\***|***2\*****2. 架构分析**

图的左边开始是监控数据源。任何应用服务想要接入 Prometheus，都需要提供 HTTP 接口（通常是 x.x.x.x/metrics 地址），并暴露 Prometheus 格式的监控数据。Prometheus Server 通过 HTTP 协议周期性抓取监控目标的监控数据、打时间戳、存储到本地。Prometheus 提供了 Client 库帮助开发人员在自己的应用中集成符合 Prometheus 格式标准的监控指标。

而对于不适合直接在代码中集成 Client 库的场景，比如应用来自第三方、不是由自己维护，应用不支持 HTTP 协议，那就需要为这些场景单独编写 Exporter 程序。Exporter 作为代理，把监控数据暴露出来。比如 Mysql Exporter，Node Exporter。

Prometheus 将采集到的数据存储在本地时序数据库中，但缺少数据副本。这也是 Prometheus 自身在数据持久化方面做的不足的地方。但这些存储问题都有其他的解决方案，Prometheus 支持 remote write 方式将数据存储到远端。

Prometheus 支持通过 Kubernetes、静态文本、Consul、DNS 等多种服务发现方式来获取抓取目标（targets）。最后，用户编写 PromQL 语句查询数据并进行可视化。

***1\***|***3\*****3. 核心组件**

Prometheus 的功能由多个互相协作的组件共同完成。这些组件也即本文开头所列出的模块，比如 Service Discovery Manager。我们后续会逐一介绍。Prometheus 源码入口 [main() 函数](https://github.com/prometheus/prometheus/blob/v2.13.0/cmd/prometheus/main.go#L89-L740) 完成参数初始化工作，并依次启动各依赖组件。

首先，main 函数解析命令行参数（详见附录 A），并读取配置文件信息（由 --config.file 参数提供）。Prometheus 特别区分了命令行参数配置（flag-based configuration）和文件配置（file-based configuration）。前者用于简单的设置，并且不支持热更新，修改需要启停 Prometheus Server 一次；后者支持热更新。

main 函数完成初始化、启动所有的[组件](https://github.com/prometheus/prometheus/blob/v2.13.0/cmd/prometheus/main.go#L490-L738)。这些组件包括：Termination Handler、Service Discovery Manager、Web Handler 等。各组件是独立的 Go Routine 在运行，之间又通过各种方式相互协调，包括使用 Channel、引用对象 Reference、传递 Context（Context 包的使用可以参考作者的 《Golang Context 包详解》一文）。

这些 Go Routine 的协作使用了 oklog/run 框架。oklog/run 是一套基于 Actor [设计模式](http://www.wityx.com/post/240_1_1.html)的 Go Routine 编排框架，实现了多个 Go Routine 作为统一整体运行并有序依次退出。这在很多开源项目中都有使用，进一步了解可参考作者的另一篇文章《Go routine 编排框架：oklog/run 包》 ***1\***|***4\*****4. 参考文档**

[「K8S 技术落地实践」Prometheus 在 K8S 上的监控实践](https://mp.weixin.qq.com/s/8wwZVYdxAZpkit4cYgiJVA) 来自本文作者在杭州容器 Meetup 的分享

[Prometheus Internal architecture](https://github.com/prometheus/prometheus/blob/master/documentation/internal_architecture.md)

***1\***|***5\*****附录 A：Prometheus 启动参数**

- web：Prometheus 服务器 HTTP 连接参数，REST API 启用相关参数，Prometheus Console 网页配置
- storage：TSDB 相关配置
- query：查询执行相关配置

| 参数名                                   | 解释                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| config.file                              | 配置文件的位置，包含抓取监控对象列表（用于服务发现）和 recording rules 文件位置 |
| web.listen-address                       | Prometheus server 端监听请求（API 请求、Dashboard 界面）的地址和端口，默认本地端口 9090 |
| web.read-timeout                         | 设置 Prometheus server 读取客户端请求的超时时间。以此来避免客户端因超慢的写操作，长时间占用链接资源。ReadTimeout 覆盖了从连接请求被接受到请求报文被完全读取的时间。默认 5m |
| web.max-connections                      | 最大连接数，默认 512                                         |
| web.external-url                         | 设置 Prometheus 对外的 URL。需要设置外部访问的 Prometheus 通常因为启用了反向代理。如果 external-url 包含路径，则 Prometheus 所有对外的接口，都会带上该路径作为前缀。eg. `http://<host>:<port>/<path>/api/v1/query?query=up` |
| web.route-prefix                         | 用于替换 external-url 的路径前缀，eg. `http://<host>:<port>/<route-prefix>/api/v1/query?query=up` |
| web.user-assets                          | 网页静态 asset 文件夹路径                                    |
| web.enable-lifecycle                     | 开启后，可以实现通过请求 /-/reload 热加载更新配置，默认 false |
| web.enable-admin-api                     | 开启后，允许使用一些管理员级别的 api，包括删除时间序列等，`/api/v1/admin/tsdb/delete_series`。默认 false |
| web.console.templates                    | Prometheus Console 网站模板文件夹路径                        |
| web.console.libraries                    | Prometheus Console 使用的库路径                              |
| web.page-title                           | Console 页面标题                                             |
| web.cors.origin                          | 设置 Prometheus 服务端允许的域，用正则表达式表示。eg. `https?://(domain*)\.com` |
| storage.tsdb.path                        | 监控数据存储路径，默认 data/                                 |
| storage.tsdb.min-block-duration          | 设置数据块最小时间跨度，默认 2h 的数据量。监控数据是按块（block）存储，每一个块中包含该时间窗口内的所有样本数据（data chunks) |
| storage.tsdb.max-block-duration          | 设置数据块最大时间跨度，默认为最大保留时间的 10%             |
| storage.tsdb.wal-segment-size            | 设置 WAL 分段存储每个分段的大小。默认 128MB                  |
| storage.tsdb.retention.time              | 监控数据最大保留时间，默认 15d                               |
| storage.tsdb.no-lockfile                 | 不在数据存储目录中创建文件锁                                 |
| storage.tsdb.wal-compression             | 开启后，会对 WAL 文件进行压缩（成本是带来 CPU 开销）。默认 false |
| storage.remote.flush-deadline            |                                                              |
| storage.remote.read-sample-limit         | 一次最多从远端存储中读取采样数据量，默认 5e7 条，0 表示无限制 |
| storage.remote.read-concurrent-limit     | 最大并发读，默认 10 个请求，0 表示无限制                     |
| rules.alert.for-outage-tolerance         |                                                              |
| rules.alert.for-grace-period             |                                                              |
| rules.alert.resend-delay                 |                                                              |
| alertmanager.notification-queue-capacity |                                                              |
| alertmanager.timeout                     |                                                              |
| query.lookback-delta                     | 在计算 PromQL 表达式结果时，最大回看时间                     |
| query.timeout                            | 查询超时时间                                                 |
| query.max-concurrency                    | 最大并发处理查询请求数，默认 20 个请求                       |
| query.max-samples                        | 能载入到内存中最大采样数据量，默认 5e7 条                    |

# Prometheus时序数据库-内存中的存储结构



## 前言

笔者最近担起了公司监控的重任，而当前监控最流行的数据库即是Prometheus。按照笔者打破砂锅问到底的精神，自然要把这个开源组件源码搞明白才行。在经过一系列源码/资料的阅读以及各种Debug之后，对其内部机制有了一定的认识。今天，笔者就来介绍下Prometheus的存储结构。
由于篇幅较长，所以笔者分为两篇，本篇主要是描述Prometheus监控数据在内存中的存储结构。下一篇，主要描述的是监控数据在磁盘中的存储结构。



## Gorilla

Prometheus的存储结构-TSDB是参考了Facebook的Gorilla之后，自行实现的。所以阅读 这篇文章《Gorilla: A Fast, Scalable, In-Memory Time Series Database》 ，可以对Prometheus为何采用这样的存储结构有着清晰的理解。



## 监控数据点

下面是一个非常典型的监控曲线。
![img](https://oscimg.oschina.net/oscnet/up-a345f9accbdaa28c5f971a0c2785cce68a1.png)
可以观察到，监控数据都是由一个一个数据点组成，所以可以用下面的结构来保存最基本的存储单元

```
type sample struct {
	t int64
	v float64
}
```

同时我们还需要注意到的信息是，我们需要知道这些点属于什么机器的哪种监控。这种信息在Promtheus中就用Label(标签来表示)。一个监控项一般会有多个Label(例如图中)，所以一般用labels []Label。
由于在我们的习惯中，并不关心单独的点，而是要关心这段时间内的曲线情况。所以自然而然的，我们存储结构肯定逻辑上是这个样子: ![img](https://oscimg.oschina.net/oscnet/up-57a83d238c0bd7b1540ded37b6802cd4b57.png)
这样，我们就可以很容易的通过一个Labels(标签们)找到对应的数据了。



## 监控数据在内存中的表示形式



### 最近的数据保存在内存中

Prometheus将最近的数据保存在内存中，这样查询最近的数据会变得非常快，然后通过一个compactor定时将数据打包到磁盘。数据在内存中最少保留2个小时(storage.tsdb.min-block-duration。至于为什么设置2小时这个值，应该是Gorilla那篇论文中观察得出的结论 ![img](https://oscimg.oschina.net/oscnet/up-c191096c12ab24d2e0e679446256fbef078.png)
即压缩率在2小时时候达到最高，如果保留的时间更短，就无法最大化的压缩。



### 内存序列(memSeries)

接下来，我们看下具体的数据结构

```
type memSeries stuct {
	......
	ref uint64 // 其id
	lst labels.Labels // 对应的标签集合
	chunks []*memChunk // 数据集合
	headChunk *memChunk // 正在被写入的chunk
	......
}
```

其中memChunk是真正保存数据的内存块，将在后面讲到。我们先来观察下memSeries在内存中的组织。 ![img](https://oscimg.oschina.net/oscnet/up-ff822df5479e5097e07aa91acce21a6d419.png)
由此我们可以看到，针对一个最终端的监控项(包含抓取的所有标签，以及新添加的标签,例如ip)，我们都在内存有一个memSeries结构。



### 寻址memSeries

如果通过一堆标签快速找到对应的memSeries。自然的,Prometheus就采用了hash。主要结构体为:

```
type stripeSeries struct {
	series [stripeSize]map[uint64]*memSeries // 记录refId到memSeries的映射
	hashes [stripeSize]seriesHashmap // 记录hash值到memSeries,hash冲突采用拉链法
	locks  [stripeSize]stripeLock // 分段锁
}
type seriesHashmap map[uint64][]*memSeries
```

由于在Prometheus中会频繁的对map[hash/refId]memSeries进行操作，例如检查这个labelSet对应的memSeries是否存在，不存在则创建等。由于golang的map非线程安全，所以其采用了分段锁去拆分锁。 ![img](https://oscimg.oschina.net/oscnet/up-5326b8be04e0d095fc337cc3863f1ef1c1c.png)
而hash值是依据labelSets的值而算出来。



### 数据点的存储

为了让Prometheus在内存和磁盘中保存更大的数据量，势必需要进行压缩。而memChunk在内存中保存的正是采用XOR算法压缩过的数据。在这里，笔者只给出Gorilla论文中的XOR描述 ![img](https://oscimg.oschina.net/oscnet/up-a2811e159a48dd360d7c9845f29a6525d99.png)
更具体的算法在论文中有详细描述。总之，使用了XOR算法后，平均每个数据点能从16bytes压缩到1.37bytes，也就是说所用空间直接降为原来的1/12!



### 内存中的倒排索引

上面讨论的是标签全部给出的查询情况。那么我们怎么快速找到某个或某几个标签(非全部标签)的数据呢。这就需要引入以Label为key的倒排索引。我们先给出一组标签集合

```
{__name__:http_requests}{group:canary}{instance:0}{job:api-server}   
{__name__:http_requests}{group:canary}{instance:1}{job:api-server}
{__name__:http_requests}{group:production}{instance:1}{job,api-server}
{__name__:http_requests}{group:production}{instance:0}{job,api-server}
```

可以看到，由于标签取值不同，我们会有四种不同的memSeries。如果一次性给定4个标签，应该是很容易从map中直接获取出对应的memSeries(尽管Prometheus并没有这么做)。但大部分我们的promql只是给定了部分标签，如何快速的查找符合标签的数据呢？
这就引入倒排索引。 先看一下，上面例子中的memSeries在内存中会有4种，同时内存中还夹杂着其它监控项的series
![img](https://oscimg.oschina.net/oscnet/up-0065f1f2c4e080313de3bee7e5e83ff17d1.png)
如果我们想知道job:api-server,group为production在一段时间内所有的http请求数量,那么必须获取标签携带 ({__name__:http_requests}{job:api-server}{group:production})的所有监控数据。
如果没有倒排索引，那么我们必须遍历内存中所有的memSeries(数万乃至数十万)，一一按照Labels去比对,这显然在性能上是不可接受的。而有了倒排索引，我们就可以通过求交集的手段迅速的获取需要哪些memSeries。 ![img](https://oscimg.oschina.net/oscnet/up-dd6641c51ca9ab93ecaa8f98efc0074bab6.png)
注意，这边倒排索引存储的refId必须是有序的。这样，我们就可以在O(n)复杂度下顺利的算出交集,另外，针对其它请求形式，还有并集/差集的操作,对应实现结构体为:

```
type intersectPostings struct {...}  // 交集
type mergedPostings struct {...} // 并集
type removedPostings struct {...} // 差集
```

倒排索引的插入组织即为Prometheus下面的代码

```
Add(labels,t,v) 
	|->getOrCreateWithID
		|->memPostings.Add
		
// Add a label set to the postings index.
func (p *MemPostings) Add(id uint64, lset labels.Labels) {
	p.mtx.Lock()
	// 将新创建的memSeries refId都加到对应的Label倒排里去
	for _, l := range lset {
		p.addFor(id, l)
	}
	p.addFor(id, allPostingsKey) // allPostingKey "","" every one都加进去

	p.mtx.Unlock()
}
```



### 正则支持

事实上，给定特定的Label:Value还是无法满足我们的需求。我们还需要对标签正则化，例如取出所有ip为1.1.*前缀的http_requests监控数据。为了这种正则，Prometheus还维护了一个标签所有可能的取值。对应代码为:

```
Add(labels,t,v) 
	|->getOrCreateWithID
		|->memPostings.Add
func (h *Head) getOrCreateWithID(id, hash uint64, lset labels.Labels){
	...
	for _, l := range lset {
		valset, ok := h.values[l.Name]
		if !ok {
			valset = stringset{}
			h.values[l.Name] = valset
		}
		// 将可能取值塞入stringset中
		valset.set(l.Value)
		// 符号表的维护
		h.symbols[l.Name] = struct{}{}
		h.symbols[l.Value] = struct{}{}
	}
	...
}
```

那么，在内存中，我们就有了如下的表 ![img](https://oscimg.oschina.net/oscnet/up-eca6d7331dca280548c106450fa56c09fe3.png)
图中所示，有了label和对应所有value集合的表，一个正则请求就可以很容的分解为若干个非正则请求并最后求交/并/查集，即可得到最终的结果。



## 总结

Prometheus作为当今最流行的时序数据库，其中有非常多的值得我们借鉴的设计和机制。这一篇笔者主要描述了监控数据在内存中的存储结构。下一篇，将会阐述监控数据在磁盘中的存储结构，敬请期待！

# Prometheus[时序数据库](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fso.csdn.net%2Fso%2Fsearch%3Fq%3D%E6%97%B6%E5%BA%8F%E6%95%B0%E6%8D%AE%E5%BA%93%26spm%3D1001.2101.3001.7020)-磁盘中的存储结构



## 前言

之前的文章里，笔者详细描述了监控数据在Prometheus内存中的结构。而其在磁盘中的存储结构，也是非常有意思的,关于这部分内容，将在本篇文章进行阐述。



## 磁盘目录结构

首先我们来看Prometheus运行后，所形成的文件目录结构
![img](https://img-blog.csdnimg.cn/img_convert/2f49f2b47a94427ce4aeda75c6b06623.png)
在笔者自己的机器上的具体结构如下:

```
 
```

1.  

   `prometheus-data`

2.  

   `|-01EY0EH5JA3ABCB0PXHAPP999D (block)`

3.  

   `|-01EY0EH5JA3QCQB0PXHAPP999D (block)`

4.  

   `|-chunks`

5.  

   `|-000001`

6.  

   `|-000002`

7.  

   `.....`

8.  

   `|-000021`

9.  

   `|-index`

10.  

    `|-meta.json`

11.  

    `|-tombstones`

12.  

    `|-wal`

13.  

    `|-chunks_head`



### Block

一个Block就是一个独立的小型数据库，其保存了一段时间内所有查询所用到的信息。包括标签/索引/符号表数据等等。Block的实质就是将一段时间里的内存数据组织成文件形式保存下来。
![img](https://img-blog.csdnimg.cn/img_convert/16cfd571730ded5921ee827a354d5837.png)
最近的Block一般是存储了2小时的数据，而较为久远的Block则会通过compactor进行合并，一个Block可能存储了若干小时的信息。值得注意的是，合并操作只是减少了索引的大小(尤其是符号表的合并)，而本身数据(chunks)的大小并没有任何改变。



#### meta.json

我们可以通过检查meta.json来得到当前Block的一些元信息。

```
 
```

1.  

   `{`

2.  

   `"ulid":"01EY0EH5JA3QCQB0PXHAPP999D"`

3.  

   `*// maxTime-minTime = 7200s => 2 h*`

4.  

   `"minTime": 1611664000000`

5.  

   `"maxTime": 1611671200000`

6.  

   `"stats": {`

7.  

   `"numSamples": 1505855631,`

8.  

   `"numSeries": 12063563,`

9.  

   `"numChunks": 12063563`

10.  

    `}`

11.  

    `"compaction":{`

12.  

    `"level" : 1`

13.  

    `"sources: [`

14.  

    `"01EY0EH5JA3QCQB0PXHAPP999D"`

15.  

    `]`

16.  

    `}`

17.  

    `"version":1`

18.  

    `}`

19.  

     

其中的元信息非常清楚明了。这个Block记录了2个小时的数据。
![img](https://img-blog.csdnimg.cn/img_convert/2e0d61829f9c16801ea65c7f0c8dfc78.png)
让我们再找一个比较陈旧的Block看下它的meta.json.

```
 
```

1.  

   `"ulid":"01EXTEH5JA3QCQB0PXHAPP999D",`

2.  

   `*// maxTime - maxTime =>162h*`

3.  

   `"minTime":1610964800000,`

4.  

   `"maxTime":1611548000000`

5.  

   `......`

6.  

   `"compaction":{`

7.  

   `"level": 5,`

8.  

   `"sources: [`

9.  

   `31个01EX......`

10.  

    `]`

11.  

    `},`

12.  

    `"parents: [`

13.  

    `{`

14.  

    `"ulid": 01EXTEH5JA3QCQB1PXHAPP999D`

15.  

    `...`

16.  

    `}`

17.  

    `{`

18.  

    `"ulid": 01EXTEH6JA3QCQB1PXHAPP999D`

19.  

    `...`

20.  

    `}`

21.  

    `{`

22.  

    `"ulid": 01EXTEH5JA31CQB1PXHAPP999D`

23.  

    `...`

24.  

    `}`

25.  

    `]`

从中我们可以看到，该Block是由31个原始Block经历5次压缩而来。最后一次压缩的三个Block ulid记录在parents中。如下图所示:
![img](https://img-blog.csdnimg.cn/img_convert/7bc79567d4543911ac7a7398ba92d262.png)



## Chunks结构



### CUT文件切分

所有的Chunk文件在磁盘上都不会大于512M,对应的源码为:

```
 
```

1.  

   `func (w *Writer) WriteChunks(chks ...Meta) error {`

2.  

   `......`

3.  

   `for i, chk := range chks {`

4.  

   `cutNewBatch := (i != 0) && (batchSize+SegmentHeaderSize > w.segmentSize)`

5.  

   `......`

6.  

   `if cutNewBatch {`

7.  

   `......`

8.  

   `}`

9.  

   `......`

10.  

    `}`

11.  

    `}`

当写入磁盘单个文件超过512M的时候，就会自动切分一个新的文件。

一个Chunks文件包含了非常多的内存Chunk结构,如下图所示:
![img](https://img-blog.csdnimg.cn/img_convert/c7cc46b6293654e04d7c685c41d7d49c.png)
图中也标出了，我们是怎么寻找对应Chunk的。通过将文件名(000001，前32位)以及(offset,后32位)编码到一个int类型的refId中，使得我们可以轻松的通过这个id获取到对应的chunk数据。



### chunks文件通过mmap去访问

由于chunks文件大小基本固定(最大512M),所以我们很容易的可以通过mmap去访问对应的数据。直接将对应文件的读操作交给操作系统，既省心又省力。对应代码为:

```
 
```

1.  

   `func NewDirReader(dir string, pool chunkenc.Pool) (*Reader, error) {`

2.  

   `......`

3.  

   `for _, fn := range files {`

4.  

   `f, err := fileutil.OpenMmapFile(fn)`

5.  

   `......`

6.  

   `}`

7.  

   `......`

8.  

   `bs = append(bs, realByteSlice(f.Bytes()))`

9.  

   `}`

10.  

    `通过sgmBytes := s.bs[offset]就直接能获取对应的数据`

![img](https://img-blog.csdnimg.cn/img_convert/ccb698af66aeabb9cc30af2b8b20fd44.png)



## index索引结构

前面介绍完chunk文件，我们就可以开始阐述最复杂的索引结构了。



### 寻址过程

索引就是为了让我们快速的找到想要的内容，为了便于理解。笔者就通过一次数据的寻址来探究Prometheus的磁盘索引结构。考虑查询一个

```
 
```

1.  

   `拥有系列三个标签`

2.  

   `({__name__:http_requests}{job:api-server}{instance:0})`

3.  

   `且时间为start/end的所有序列数据`

我们先从选择Block开始,遍历所有Block的meta.json，找到具体的Block
![img](https://img-blog.csdnimg.cn/img_convert/44b81bc99fbbda939fb17be503332fc5.png)
前文说了，通过Labels找数据是通过倒排索引。我们的倒排索引是保存在index文件里面的。那么怎么在这个单一文件里找到倒排索引的位置呢？这就引入了TOC(Table Of Content)



### TOC(Table Of Content)

![img](https://img-blog.csdnimg.cn/img_convert/f1ee1011bbd1956f988f96e1ed66322d.png)
由于index文件一旦形成之后就不再会改变，所以Prometheus也依旧使用mmap来进行操作。采用mmap读取TOC非常容易:

```
 
```

1.  

   `func NewTOCFromByteSlice(bs ByteSlice) (*TOC, error) {`

2.  

   `......`

3.  

   `*// indexTOCLen = 6\*8+4 = 52*`

4.  

   `b := bs.Range(bs.Len()-indexTOCLen, bs.Len())`

5.  

   `......`

6.  

   `return &TOC{`

7.  

   `Symbols: d.Be64(),`

8.  

   `Series: d.Be64(),`

9.  

   `LabelIndices: d.Be64(),`

10.  

    `LabelIndicesTable: d.Be64(),`

11.  

    `Postings: d.Be64(),`

12.  

    `PostingsTable: d.Be64(),`

13.  

    `}, nil`

14.  

    `}`



### Posting offset table 以及 Posting倒排索引

首先我们访问的是Posting offset table。由于倒排索引按照不同的LabelPair(key/value)会有非常多的条目。所以Posing offset table就是决定到底访问哪一条Posting索引。offset就是指的这一Posting条目在文件中的偏移。
![img](https://img-blog.csdnimg.cn/img_convert/35c4376de22dc5708d0e8e2f857184ef.png)



### Series

我们通过三条Postings倒排索引索引取交集得出

```
 
```

1.  

   `{series1,Series2,Series3,Series4}`

2.  

   `∩`

3.  

   `{series1,Series2,Series3}`

4.  

   `∩`

5.  

   `{Series2,Series3}`

6.  

   `=`

7.  

   `{Series2,Series3}`

也就是要读取Series2和Serie3中的数据，而Posting中的Ref(Series2)和Ref(Series3)即为这两Series在index文件中的偏移。
![img](https://img-blog.csdnimg.cn/img_convert/28edecaa911dfff49ced2809b15584e2.png)
Series以Delta的形式记录了chunkId以及该chunk包含的时间范围。这样就可以很容易过滤出我们需要的chunk,然后再按照chunk文件的访问，即可找到最终的原始数据。



### SymbolTable

值得注意的是，为了尽量减少我们文件的大小，对于Label的Name和Value这些有限的数据，我们会按照字母序存在符号表中。由于是有序的，所以我们可以直接将符号表认为是一个
[]string切片。然后通过切片的下标去获取对应的sting。考虑如下符号表:
![img](https://img-blog.csdnimg.cn/img_convert/e730dda36c6af34790a78dda8437a78e.png)
读取index文件时候，会将SymbolTable全部加载到内存中，并组织成symbols []string这样的切片形式，这样一个Series中的所有标签值即可通过切片下标访问得到。



### Label Index以及Label Table

事实上，前面的介绍已经将一个普通数据寻址的过程全部讲完了。但是index文件中还包含label索引以及label Table，这两个是用来记录一个Label下面所有可能的值而存在的。
这样，在正则的时候就可以非常容易的找到我们需要哪些LabelPair。详情可以见前篇。
![img](https://img-blog.csdnimg.cn/img_convert/bcf521d3a3a3ca6e9e29440481bc9dae.png)

事实上，真正的Label Index比图中要复杂一点。它设计成一条LabelIndex可以表示(多个标签组合)的所有数据。不过在Prometheus代码中只会采用存储一个标签对应所有值的形式。



## 完整的index文件结构

这里直接给出完整的index文件结构，摘自Prometheus中index.md文档。

```
 
```

1.  

   `┌────────────────────────────┬─────────────────────┐`

2.  

   `│ magic(0xBAAAD700) <4b> │ version(1) <1 byte> │`

3.  

   `├────────────────────────────┴─────────────────────┤`

4.  

   `│ ┌──────────────────────────────────────────────┐ │`

5.  

   `│ │ Symbol Table │ │`

6.  

   `│ ├──────────────────────────────────────────────┤ │`

7.  

   `│ │ Series │ │`

8.  

   `│ ├──────────────────────────────────────────────┤ │`

9.  

   `│ │ Label Index 1 │ │`

10.  

    `│ ├──────────────────────────────────────────────┤ │`

11.  

    `│ │ ... │ │`

12.  

    `│ ├──────────────────────────────────────────────┤ │`

13.  

    `│ │ Label Index N │ │`

14.  

    `│ ├──────────────────────────────────────────────┤ │`

15.  

    `│ │ Postings 1 │ │`

16.  

    `│ ├──────────────────────────────────────────────┤ │`

17.  

    `│ │ ... │ │`

18.  

    `│ ├──────────────────────────────────────────────┤ │`

19.  

    `│ │ Postings N │ │`

20.  

    `│ ├──────────────────────────────────────────────┤ │`

21.  

    `│ │ Label Index Table │ │`

22.  

    `│ ├──────────────────────────────────────────────┤ │`

23.  

    `│ │ Postings Table │ │`

24.  

    `│ ├──────────────────────────────────────────────┤ │`

25.  

    `│ │ TOC │ │`

26.  

    `│ └──────────────────────────────────────────────┘ │`

27.  

    `└──────────────────────────────────────────────────┘`



## tombstones

由于Prometheus Block的数据一般在写完后就不会变动。如果要删除部分数据，就只能记录一下删除数据的范围，由下一次compactor组成新block的时候删除。而记录这些信息的文件即是tomstones。



## Prometheus入门书籍推荐



## 总结

Prometheus作为时序数据库，设计了各种文件结构来保存海量的监控数据，同时还兼顾了性能。只有彻底了解其存储结构，才能更好的指导我们应用它！

# Prometheus时序数据库-数据的插入



## 前言

在之前的文章里，笔者详细的阐述了Prometheus时序数据库在内存和磁盘中的存储结构。有了前面的铺垫，笔者就可以在本篇文章阐述下数据的插入过程。



## 监控数据的插入

在这里，笔者并不会去讨论Promtheus向各个Endpoint抓取数据的过程。而是仅仅围绕着数据是如何插入Prometheus的过程做下阐述。对应方法:

```
func (a *headAppender) Add(lset labels.Labels, t int64, v float64) (uint64, error) {
	......
	// 如果lset对应的series没有，则建一个。同时把新建的series放入倒排Posting映射里面
	s, created := a.head.getOrCreate(lset.Hash(), lset) 
	if created { // 如果新创建了一个，则将新建的也放到a.series里面
		a.series = append(a.series, record.RefSeries{
			Ref:    s.ref,
			Labels: lset,
		})
	}
	return s.ref, a.AddFast(s.ref, t, v)
}
```

我们就以下面的add函数调用为例:

```
app.Add(labels.FromStrings("foo", "bar"), 0, 0)
```

首先是getOrCreate,顾名思义，不存在则创建一个。创建的过程包含了seriesHashMap/Postings(倒排索引)/LabelIndex的维护。如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-bb69c43d88d81adb175754cfe3cea43460f.png)
然后是AddFast方法

```
func (a *headAppender) AddFast(ref uint64, t int64, v float64) error{
		// 拿出对应的memSeries
		s := a.head.series.getByID(ref)
		......
		// 设置为等待提交状态
		s.pendingCommit=true
		......
		// 为了事务概念，放入temp存储，等待真正commit时候再写入memSeries
		a.samples = append(a.samples, record.RefSample{Ref: ref,T:   t,V:   v,})
		// 
}
```

Prometheus在add数据点的时候并没有直接add到memSeries(也就是query所用到的结构体里),而是加入到一个临时的samples切片里面。同时还将这个数据点对应的memSeries同步增加到另一个sampleSeries里面。 ![img](https://oscimg.oschina.net/oscnet/up-caf4f4f54755889dfaa960d7ac9214d39e7.png)



### 事务可见性

为什么要这么做呢？就是为了实现commit语义，只有commit过后数据才可见(能被查询到)。否则，无法见到这些数据。而commit的动作主要就是WAL(Write Ahead Log)以及将headerAppender.samples数据写到其对应的memSeries中。这样，查询就可见这些数据了，如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-ab6300a60c3907ed962089e8ec9cf48e923.png)



### WAL

由于Prometheus最近的数据是保存在内存里面的，未防止服务器宕机丢失数据。其在commit之前先写了日志WAL。等服务重启的时候，再从WAL日志里面获取信息并重放。 ![img](https://oscimg.oschina.net/oscnet/up-f0ed5e3bd362b5c56e4a9af31f746c3df30.png)
为了性能，Prometheus了另一个goroutine去做文件的sync操作，所以并不能保证WAL不丢。进而也不能保证监控数据完全不丢。这点也是监控业务的特性决定的。

写入代码为:

```
commit()
|=>
func (a *headAppender) log() error {
	......
	// 往WAL写入对应的series信息
	if len(a.series) > 0 {
		rec = enc.Series(a.series, buf)
		buf = rec[:0]

		if err := a.head.wal.Log(rec); err != nil {
			return errors.Wrap(err, "log series")
		}
	}
	......
	// 往WAL写入真正的samples
	if len(a.samples) > 0 {
		rec = enc.Samples(a.samples, buf)
		buf = rec[:0]

		if err := a.head.wal.Log(rec); err != nil {
			return errors.Wrap(err, "log samples")
		}
	}
}
```

对应的WAL日志格式为:



#### Series records

```
┌────────────────────────────────────────────┐
│ type = 1 <1b>                              │
├────────────────────────────────────────────┤
│ ┌─────────┬──────────────────────────────┐ │
│ │ id <8b> │ n = len(labels) <uvarint>    │ │
│ ├─────────┴────────────┬─────────────────┤ │
│ │ len(str_1) <uvarint> │ str_1 <bytes>   │ │
│ ├──────────────────────┴─────────────────┤ │
│ │  ...                                   │ │
│ ├───────────────────────┬────────────────┤ │
│ │ len(str_2n) <uvarint> │ str_2n <bytes> │ │
│ └───────────────────────┴────────────────┘ │
│                  . . .                     │
└────────────────────────────────────────────┘
```



#### Sample records

```
┌──────────────────────────────────────────────────────────────────┐
│ type = 2 <1b>                                                    │
├──────────────────────────────────────────────────────────────────┤
│ ┌────────────────────┬───────────────────────────┐               │
│ │ id <8b>            │ timestamp <8b>            │               │
│ └────────────────────┴───────────────────────────┘               │
│ ┌────────────────────┬───────────────────────────┬─────────────┐ │
│ │ id_delta <uvarint> │ timestamp_delta <uvarint> │ value <8b>  │ │
│ └────────────────────┴───────────────────────────┴─────────────┘ │
│                              . . .                               │
└──────────────────────────────────────────────────────────────────┘
```

见Prometheus WAL.md



## 落盘存储

之前描述的所有数据都是写到内存里面。最终落地是通过compator routine将每两个小时的数据打包到一个Blocks里面。 ![img](https://oscimg.oschina.net/oscnet/up-67948a457f3bc2946c6321862a35c00e152.png)
具体可见笔者之前的博客《Prometheus时序数据库-磁盘中的存储结构》



## 总结

在这篇文章里，笔者详细描述了Prometheus数据的插入过程。在下一篇文章里面，笔者会继续 阐述Prometheus数据的查询过程。

# Prometheus时序数据库-数据的查询



## 前言

在之前的博客里，笔者详细阐述了Prometheus数据的插入过程。但我们最常见的打交道的是数据的查询。Prometheus提供了强大的Promql来满足我们千变万化的查询需求。在这篇文章里面，笔者就以一个简单的Promql为例，讲述下Prometheus查询的过程。



## Promql

一个Promql表达式可以计算为下面四种类型:

```
瞬时向量(Instant Vector) - 一组同样时间戳的时间序列(取自不同的时间序列，例如不同机器同一时间的CPU idle)
区间向量(Range vector) - 一组在一段时间范围内的时间序列
标量(Scalar) - 一个浮点型的数据值
字符串(String) - 一个简单的字符串
```

我们还可以在Promql中使用svm/avg等集合表达式，不过只能用在瞬时向量(Instant Vector)上面。为了阐述Prometheus的聚合计算以及篇幅原因，笔者在本篇文章只详细分析瞬时向量(Instant Vector)的执行过程。



## 瞬时向量(Instant Vector)

前面说到，瞬时向量是一组拥有同样时间戳的时间序列。但是实际过程中，我们对不同Endpoint采样的时间是不可能精确一致的。所以，Prometheus采取了距离指定时间戳之前最近的数据(Sample)。如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-5467e767b9d532b67ec7bdc1d7813fb763c.png)
当然，如果是距离当前时间戳1个小时的数据直观看来肯定不能纳入到我们的返回结果里面。 所以Prometheus通过一个指定的时间窗口来过滤数据(通过启动参数--query.lookback-delta指定，默认5min)。



## 对一条简单的Promql进行分析

好了，解释完Instant Vector概念之后，我们可以着手进行分析了。直接上一条带有聚合函数的Promql把。

```
SUM BY (group) (http_requests{job="api-server",group="production"})
```

首先,对于这种有语法结构的语句肯定是将其Parse一把，构造成AST树了。调用

```
promql.ParseExpr
```

由于Promql较为简单，所以Prometheus直接采用了LL语法分析。在这里直接给出上述Promql的AST树结构。 ![img](https://oscimg.oschina.net/oscnet/up-4d586b4f4c2a76699138218fb1678cc6623.png)
Prometheus对于语法树的遍历过程都是通过vistor模式,具体到代码为:

```
ast.go vistor设计模式
func Walk(v Visitor, node Node, path []Node) error {
	var err error
	if v, err = v.Visit(node, path); v == nil || err != nil {
		return err
	}
	path = append(path, node)

	for _, e := range Children(node) {
		if err := Walk(v, e, path); err != nil {
			return err
		}
	}

	_, err = v.Visit(nil, nil)
	return err
}
func (f inspector) Visit(node Node, path []Node) (Visitor, error) {
	if err := f(node, path); err != nil {
		return nil, err
	}

	return f, nil
}
```

通过golang里非常方便的函数式功能，直接传递求值函数inspector进行不同情况下的求值。

```
type inspector func(Node, []Node) error
```



## 求值过程

具体的求值过程核心函数为:

```
func (ng *Engine) execEvalStmt(ctx context.Context, query *query, s *EvalStmt) (Value, storage.Warnings, error) {
	......
	querier, warnings, err := ng.populateSeries(ctxPrepare, query.queryable, s) 	// 这边拿到对应序列的数据
	......
	val, err := evaluator.Eval(s.Expr) // here 聚合计算
	......

}
```



### populateSeries

首先通过populateSeries的计算出VectorSelector Node所对应的series(时间序列)。这里直接给出求值函数

```
 func(node Node, path []Node) error {
 	......
 	querier, err := q.Querier(ctx, timestamp.FromTime(mint), timestamp.FromTime(s.End))
 	......
 	case *VectorSelector:
 		.......
 		set, wrn, err = querier.Select(params, n.LabelMatchers...)
 		......
 		n.unexpandedSeriesSet = set
 	......
 	case *MatrixSelector:
 		......
 }
 return nil
```

可以看到这个求值函数，只对VectorSelector/MatrixSelector进行操作，针对我们的Promql也就是只对叶子节点VectorSelector有效。 ![img](https://oscimg.oschina.net/oscnet/up-26277b2ae672bbc7adb99275c7e2c9cfe34.png)



#### select

获取对应数据的核心函数就在querier.Select。我们先来看下qurier是如何得到的.

```
querier, err := q.Querier(ctx, timestamp.FromTime(mint), timestamp.FromTime(s.End))
```

根据时间戳范围去生成querier,里面最重要的就是计算出哪些block在这个时间范围内，并将他们附着到querier里面。具体见函数

```
func (db *DB) Querier(mint, maxt int64) (Querier, error) {
	for _, b := range db.blocks {
		......
		// 遍历blocks挑选block
	}
	// 如果maxt>head.mint(即内存中的block),那么也加入到里面querier里面。
	if maxt >= db.head.MinTime() {
		blocks = append(blocks, &rangeHead{
			head: db.head,
			mint: mint,
			maxt: maxt,
		})
	}
	......
}
```

![img](https://oscimg.oschina.net/oscnet/up-530937ab4d6605719c009aec5cd93c1ce69.png)
知道数据在哪些block里面，我们就可以着手进行计算VectorSelector的数据了。

```
 // labelMatchers {job:api-server} {__name__:http_requests} {group:production}
 querier.Select(params, n.LabelMatchers...)
```

有了matchers我们很容易的就能够通过倒排索引取到对应的series。为了篇幅起见，我们假设数据都在headBlock(也就是内存里面)。那么我们对于倒排的计算就如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-4aceb095eddec46d92dacec57380e52d13e.png)
这样，我们的VectorSelector节点就已经有了最终的数据存储地址信息了，例如图中的memSeries refId=3和4。 ![img](https://oscimg.oschina.net/oscnet/up-a75ebed4c0e194acd32dbbb48e8cd8646bd.png)
如果想了解在磁盘中的数据寻址，可以详见笔者之前的博客

```
<<Prometheus时序数据库-磁盘中的存储结构>>
```



### evaluator.Eval

通过populateSeries找到对应的数据，那么我们就可以通过evaluator.Eval获取最终的结果了。计算采用后序遍历，等下层节点返回数据后才开始上层节点的计算。那么很自然的，我们先计算VectorSelector。

```
func (ev *evaluator) eval(expr Expr) Value {
	......
	case *VectorSelector:
	// 通过refId拿到对应的Series
	checkForSeriesSetExpansion(ev.ctx, e)
	// 遍历所有的series
	for i, s := range e.series {
		// 由于我们这边考虑的是instant query,所以只循环一次
		for ts := ev.startTimestamp; ts <= ev.endTimestamp; ts += ev.interval {
			// 获取距离ts最近且小于ts的最近的sample
			_, v, ok := ev.vectorSelectorSingle(it, e, ts)
			if ok {
					if ev.currentSamples < ev.maxSamples {
						// 注意，这边的v对应的原始t被替换成了ts,也就是instant query timeStamp
						ss.Points = append(ss.Points, Point{V: v, T: ts})
						ev.currentSamples++
					} else {
						ev.error(ErrTooManySamples(env))
					}
				}
			......
		}
	}
}
```

如代码注释中看到，当我们找到一个距离ts最近切小于ts的sample时候，只用这个sample的value,其时间戳则用ts(Instant Query指定的时间戳)代替。

其中vectorSelectorSingle值得我们观察一下:

```
func (ev *evaluator) vectorSelectorSingle(it *storage.BufferedSeriesIterator, node *VectorSelector, ts int64) (int64, float64, bool){
	......
	// 这一步是获取>=refTime的数据，也就是我们instant query传入的
	ok := it.Seek(refTime)
	......
		if !ok || t > refTime { 
		// 由于我们需要的是<=refTime的数据，所以这边回退一格，由于同一memSeries同一时间的数据只有一条，所以回退的数据肯定是<=refTime的
		t, v, ok = it.PeekBack(1)
		if !ok || t < refTime-durationMilliseconds(LookbackDelta) {
			return 0, 0, false
		}
	}
}
```

就这样，我们找到了series 3和4距离Instant Query时间最近且小于这个时间的两条记录，并保留了记录的标签。这样，我们就可以在上层进行聚合。 ![img](https://oscimg.oschina.net/oscnet/up-5873a7d0f658c20589cf5c1669c530be3a4.png)



## SUM by聚合

叶子节点VectorSelector得到了对应的数据后，我们就可以对上层节点AggregateExpr进行聚合计算了。代码栈为:

```
evaluator.rangeEval
	|->evaluate.eval.func2
		|->evelator.aggregation grouping key为group
```

具体的函数如下图所示:

```
func (ev *evaluator) aggregation(op ItemType, grouping []string, without bool, param interface{}, vec Vector, enh *EvalNodeHelper) Vector {
	......
	// 对所有的sample
	for _, s := range vec {
		metric := s.Metric
		......
		group, ok := result[groupingKey] 
		// 如果此group不存在，则新加一个group
		if !ok {
			......
			result[groupingKey] = &groupedAggregation{
				labels:     m, // 在这里我们的m=[group:production]
				value:      s.V,
				mean:       s.V,
				groupCount: 1,
			}
			......
		}
		switch op {
		// 这边就是对SUM的最终处理
		case SUM:
			group.value += s.V
		.....
		}
	}
	.....
	for _, aggr := range result {
		enh.out = append(enh.out, Sample{
		Metric: aggr.labels,
		Point:  Point{V: aggr.value},
		})
	}
	......
	return enh.out
}
```

好了，有了上面的处理，我们聚合的结果就变为: ![img](https://oscimg.oschina.net/oscnet/up-f46d0e91b714bd5547e581bb21272bc22fe.png)
这个和我们的预期结果一致,一次查询的过程就到此结束了。



## 总结

Promql是非常强大的，可以满足我们的各种需求。其运行原理自然也激起了笔者的好奇心，本篇文章虽然只分析了一条简单的Promql,但万变不离其宗,任何Promql都是类似的运行逻辑。希望本文对读者能有所帮助。

# Prometheus时序数据库-数据的抓取



## 前言

在之前的文章里，笔者详细的阐述了数据的存储/写入以及查询过程。那么Prometheus又是怎么去主动获得数据的呢？这个问题，笔者将在本篇文章详细阐述。



## Pull模式

Prometheus是通过pull从各个Target(目标)Exporter中去获取数据。 ![img](https://oscimg.oschina.net/oscnet/up-1ce226071079a48fe7ab7f4b90153503f64.png)
作为一个成熟的监控系统，自然可以对这些Target进行自动发现。下面，笔者就阐述下Prometheus自动发现的过程。



## 配置的加载

首先，我们需要告诉Prometheus要通过什么样的方式去自动获取Targets。这就是我们的scrape_configs。



### scrape_configs

先给出一个scrape_configs的例子:

```
prometheus.yml
global:
	scrape_interval: 60s
	scrape_timeout: 40s
	......
scrape_configs:
	- job_name: 'prometheus'
		static_configs:
		- targets: ['127.0.0.1:9090']
	- job_name: 'node'
		http_sd_configs:
			- url:"http://cmdbIp:8080/queryAll'
```

这个yml文件里面，定义了两个job,分别为抓取127.0.0.1:9100这个监控Prometheus自身的job。第二个为我们自定义的http_sd_confgis,向我们的cmdb获取数据。这样，Prometheus就可以和我们本身的运维基础设施联动了。 ![img](https://oscimg.oschina.net/oscnet/up-4c6a868c46581fba3f238dab6d9457c8698.png)
好了，为了能够完成这样的自发现功能，我们首先要解析出scrape_configs中的元数据。



### scrape_configs的元数据解析

这样的元配置数据信息肯定是在启动的时候就已经加载好了。所以我们去观察main.go。

```
main.go

func main() {
		// 对scrape_configs配置建立providers，providers会提供最终所有的Target
		func(cfg *config.Config) error {
			c := make(map[string]sd_config.ServiceDiscoveryConfig)
			for _, v := range cfg.ScrapeConfigs {
				c[v.JobName] = v.ServiceDiscoveryConfig
			}
			return discoveryManagerScrape.ApplyConfig(c)
		},
}
```

上面的匿名函数中，将读取到的scrape_configs和其job名建立映射，然后调用discoveryManagerScrape.ApplyConfig去建立providers(Targets的提供者)。我们观察下代码:

```
discoveryManagerScrape.ApplyConfig

func (m *Manager) ApplyConfig(cfg map[string]sd_config.ServiceDiscoveryConfig) error {
	......
	// 为每个scrape_config都建立一个provider并注册
	for name, scfg := range cfg {
		failedCount += m.registerProviders(scfg, name)
		discoveredTargets.WithLabelValues(m.name, name).Set(0)
	}	
	// 启动所有的providers，也就启动了不停的抓取过程
	for _, prov := range m.providers {
		m.startProvider(m.ctx, prov)
	}
}
```

其中注册过程如下，根据不同的scrapeConfigs类型生成不同的provider,然后放到一个scrapeManager的成员providers切片里面。
![img](https://oscimg.oschina.net/oscnet/up-dd46333194531b9c3daa57bf818573da35b.png)
在我们解析出所有的provider之后，就可以启动provider来进行不停抓取所有Target的过程了。注意这里只是监控目标的抓取，并非抓取真正的数据。

```
func (m *Manager) startProvider(ctx context.Context, p *provider) {
	......
	// 抓取最新Targets的goroutine
	go p.d.Run(ctx, updates)
	// 将Prometheus现有Targets集合替换为新Targets(刚抓取到的)的携程
	go m.updater(ctx, p, updates)
}
```

Prometheus对每个provider分了两个goroutine来进行更新Targets(监控对象)集合的操作,第一个是不同的获取最新Targets的goroutine。在抓取到之后，会由updater goroutine来热更新当前抓取目标。更新完目标targets后，再触发triggerSend chan。 ![img](https://oscimg.oschina.net/oscnet/up-76f3a42f5666b35b9b786e582167e6ab45b.png)
这样做，虽然将逻辑变复杂了，但很好的将scrape的过程与更新target的过程隔离了开来。



### 防止更新过于频繁

为了防止更新过于频繁，Prometheus通过一个ticker去限制更新频率。

```
manager.go

func (m *Manager) sender(){
	ticker := time.NewTicker(m.updatert)
	defer ticker.Stop()
	......
	for {
	.....
		select {
			......
			case <-ticker.C:
				......
				case <-m.triggerSend
				......
				case m.synch <- m.allGroups
				......
		}
	}
}
```

如代码中所见，只有在ticker每秒触发后才会去检测triggerSend信号，这样就不会频繁的触发重建scrape targets逻辑了。代码中，我们最终将allGroups,也就是m.targets(刚刚更新过的)的所有数据全部取出送入到m.synch
![img](https://oscimg.oschina.net/oscnet/up-c8ef1005449837700ddea2bcca8ab3c8402.png)
通过m.synch触发更新manger.targetSets,最后触发reloader信号,如下代码所示:

```
scrape/manager.go

func (m *Manager) Run(tsets <-chan map[string][]*targetgroup.Group) error {
	go m.reloader()
	for {
		select {
		case ts := <-tsets:
			m.updateTsets(ts)

			select {
			case m.triggerReload <- struct{}{}:
			default:
			}

		case <-m.graceShut:
			return nil
		}
	}
}
```



### reloader操作

好了，终于到我们的reloader操作了，也就是真正改变Prometheus抓取目标动作的地方。 首先，我们对每一个provider(及其底下的subs)建立一个scrapePool，并对每个target建立一个goroutine去exporter定时抓取数据。
![img](https://oscimg.oschina.net/oscnet/up-f462774df28d83cc659e9dbe3427a475bee.png)

```
func (sp *scrapePool) sync(targets []*Target) {
	
	// 这段hash代码会有问题，hash出来不是唯一的
	for _, t := range targets {
		......
		// 为每一个target建立一个goroutine
		go l.run(interval, timeout, nil)
		......
	}
	......
	// Stop and remove old targets and scraper loops.
	for hash := range sp.activeTargets {
		if _, ok := uniqueTargets[hash]; !ok {
			......
			l.stop()
			.....
		}
		......
	}
	......
}
```

由于我们这边是reloader逻辑，所以Prometheus还会去比对这次的target集合和之前的target集合的差异。如果有新的，则新建goroutine去scrap，如果老集合中有的元素已经过期了，则stop对应的goroutine。
值得注意的是，这边Prometheus直接用hash代替targets去判断是否存在，而hash并不能和target一一对应，所以这边是有些问题的！需要拉链法进一步去判断。



## 抓取单个Target的实际数据

我们通过scrapeLoop去定时从不同的Target获取数据并写入到TSDB(时序数据库里面)。

```
scrape.go

func (sl *scrapeLoop) run(interval, timeout time.Duration, errc chan<- error) {
	......
mainLoop:
	......
	// 发起http请求，并获取结果
	contentType, scrapeErr := sl.scraper.scrape(scrapeCtx, buf)
	......
	// 解析http Response,并写入OpenTSDB
	sl.append(b, contentType, start)
	......
}
```

![img](https://oscimg.oschina.net/oscnet/up-9bc6ac0c6d9106a5c3ab82568d7bdf954b6.png)
解析http Response并插入TSDB的函数为:

```
scrape.go

func (sl *scrapeLoop) append(b []byte, contentType string, ts time.Time) {
......
loop:
	for {
		var et textparse.Entry
		// 解析parse
		......
		// 解析后的数据写入TSDB
		app.AddFast(ce.lset, ce.ref, t, v);
	}
	......
	// 没有错误的话，再提交
	if err := app.Commit(); err != nil {
		return total, added, seriesAdded, err
	}
	......
}
```

通过TSDB事务的概念，Prometheus使得我们的数据写入是原子的，即不会出现只看到部分数据的现象。至于事务的实现，可以见笔者之前的博客《Prometheus时序数据库-数据的插入》。



## 总结

Prometehus提供了非常灵活的接口供其和我们自身的运维基础设施进行联动。在实际应用过程中，我们只需要实现对应的discovery逻辑即可。在本篇文章里面，笔者详细阐述了Promtheus的自动发现以及数据抓取过程,希望对读者有所帮助

# Prometheus时序数据库-报警的计算

在前面的文章中，笔者详细的阐述了Prometheus的数据插入存储查询等过程。但作为一个监控神器，报警计算功能是必不可少的。自然的Prometheus也提供了灵活强大的报警规则可以让我们自由去发挥。在本篇文章里，笔者就带读者去看下Prometheus内部是怎么处理报警规则的。



## 报警架构

Prometheus只负责进行报警计算，而具体的报警触发则由AlertManager完成。如果我们不想改动AlertManager以完成自定义的路由规则，还可以通过webhook外接到另一个系统(例如,一个转换到kafka的程序)。 ![img](https://oscimg.oschina.net/oscnet/up-66fea7ae49f9ad663511fdb0381b815650f.png)
在本篇文章里，笔者并不会去设计alertManager,而是专注于Prometheus本身报警规则的计算逻辑。



## 一个最简单的报警规则

```
rules:
	alert: HTTPRequestRateLow
	expr: http_requests < 100
	for: 60s
	labels:
		severity: warning
	annotations:
		description: "http request rate low"
	
```

这上面的规则即是http请求数量<100从持续1min,则我们开始报警，报警级别为warning



## 什么时候触发这个计算

在加载完规则之后，Prometheus按照evaluation_interval这个全局配置去不停的计算Rules。代码逻辑如下所示:

```
rules/manager.go

func (g *Group) run(ctx context.Context) {
	iter := func() {
		......
		g.Eval(ctx,evalTimestamp)
		......
	}
	// g.interval = evaluation_interval
	tick := time.NewTicker(g.interval)
	defer tick.Stop()
	......
	for {
		......
		case <-tick.C:
			......
			iter()
	}
}
```

而g.Eval的调用为:

```
func (g *Group) Eval(ctx context.Context, ts time.Time) {
	// 对所有的rule
	for i, rule := range g.rules {
		......
		// 先计算出是否有符合rule的数据
		vector, err := rule.Eval(ctx, ts, g.opts.QueryFunc, g.opts.ExternalURL)
		......
		// 然后发送
		ar.sendAlerts(ctx, ts, g.opts.ResendDelay, g.interval, g.opts.NotifyFunc)
	}
	......
}
```

整个过程如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-6e8f9b9a7caf0eddebfd3be0b7a4dd52e1f.png)



## 对单个rule的计算

我们可以看到，最重要的就是rule.Eval这个函数。代码如下所示:

```
func (r *AlertingRule) Eval(ctx context.Context, ts time.Time, query QueryFunc, externalURL *url.URL) (promql.Vector, error) {
	// 最终调用了NewInstantQuery
	res, err = query(ctx,r.vector.String(),ts)
	......
	// 报警组装逻辑
	......
	// active 报警状态变迁
}
```

这个Eval包含了报警的计算/组装/发送的所有逻辑。我们先聚焦于最重要的计算逻辑。也就是其中的query。其实，这个query是对NewInstantQuery的一个简单封装。

```
func EngineQueryFunc(engine *promql.Engine, q storage.Queryable) QueryFunc {
	return func(ctx context.Context, qs string, t time.Time) (promql.Vector, error) {
		q, err := engine.NewInstantQuery(q, qs, t)
		......
		res := q.Exec(ctx)
	}
}
```

也就是说它执行了一个瞬时向量的查询。而其查询的表达式按照我们之前给出的报警规则，即是

```
http_requests < 100 
```

既然要计算表达式，那么第一步，肯定是将其构造成一颗AST。其树形结构如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-6d2d0df21a05dda8316f31c400275d63b81.png)
解析出左节点是个VectorSelect而且知道了其lablelMatcher是

```
__name__:http_requests
```

那么我们就可以左节点VectorSelector进行求值。直接利用倒排索引在head中查询即可(因为instant query的是当前时间，所以肯定在内存中)。 ![img](https://oscimg.oschina.net/oscnet/up-b409c8950485e16ce78206848806397d852.png)
想知道具体的计算流程，可以见笔者之前的博客《Prometheus时序数据库-数据的查询》 计算出左节点的数据之后，我们就可以和右节点进行比较以计算出最终结果了。具体代码为:

```
func (ev *evaluator) eval(expr Expr) Value {
	......
	case *BinaryExpr:
	......
		case lt == ValueTypeVector && rt == ValueTypeScalar:
			return ev.rangeEval(func(v []Value, enh *EvalNodeHelper) Vector {
				return ev.VectorscalarBinop(e.Op, v[0].(Vector), Scalar{V: v[1].(Vector)[0].Point.V}, false, e.ReturnBool, enh)
			}, e.LHS, e.RHS)
	.......
}
```

最后调用的函数即为:

```
func (ev *evaluator) VectorBinop(op ItemType, lhs, rhs Vector, matching *VectorMatching, returnBool bool, enh *EvalNodeHelper) Vector {
	// 对左节点计算出来的所有的数据sample
	for _, lhsSample := range lhs {
		......
		// 由于左边lv = 75 < 右边rv = 100，且op为less
		/**
			vectorElemBinop(){
				case LESS
					return lhs, lhs < rhs
			}
		**/
		// 这边得到的结果value=75,keep = true
		value, keep := vectorElemBinop(op, lv, rv)
		......
		if keep {
			......
			// 这边就讲75放到了输出里面，也就是说我们最后的计算确实得到了数据。
			enh.out = append(enh.out.sample)
		}
	}
}
```

如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-05d4a71b5931f57ed08682b2681f4d8b5d5.png)
最后我们的expr输出即为

```
sample {
	Point {t:0,V:75}
	Metric {__name__:http_requests,instance:0,job:api-server}
		
}
```



## 报警状态变迁

计算过程讲完了，笔者还稍微讲一下报警的状态变迁，也就是最开始报警规则中的rule中的for,也即报警持续for(规则中为1min)，我们才真正报警。为了实现这种功能，这就需要一个状态机了。笔者这里只阐述下从Pending(报警出现)->firing(真正发送)的逻辑。

在之前的Eval方法里面，有下面这段

```
func (r *AlertingRule) Eval(ctx context.Context, ts time.Time, query QueryFunc, externalURL *url.URL) (promql.Vector, error) {
	for _, smpl := range res {
	......
			if alert, ok := r.active[h]; ok && alert.State != StateInactive {
			alert.Value = smpl.V
			alert.Annotations = annotations
			continue
		}
		// 如果这个告警不在active map里面，则将其放入
		// 注意，这里的hash依旧没有拉链法，有极小概率hash冲突
r.active[h] = &Alert{
			Labels:      lbs,
			Annotations: annotations,
			ActiveAt:    ts,
			State:       StatePending,
			Value:       smpl.V,
		}
	}
	......
	// 报警状态的变迁逻辑
	for fp, a := range r.active {
		// 如果当前r.active的告警已经不在刚刚计算的result里面了		if _, ok := resultFPs[fp]; !ok {
			// 如果状态是Pending待发送
			if a.State == StatePending || (!a.ResolvedAt.IsZero() && ts.Sub(a.ResolvedAt) > resolvedRetention) {
				delete(r.active, fp)
			}
			......
			continue
		}
		// 对于已有的Active报警，如果其Active的时间>r.holdDuration，也就是for指定的
		if a.State == StatePending && ts.Sub(a.ActiveAt) >= r.holdDuration {
			// 我们将报警置为需要发送
			a.State = StateFiring
			a.FiredAt = ts
		}
		......
	
	}
}
```

上面代码逻辑如下图所示: ![img](https://oscimg.oschina.net/oscnet/up-83a00965989fe9f3a5e6f5e93f49343fb5d.png)



## 总结

Prometheus作为一个监控神器，给我们提供了各种各样的遍历。其强大的报警计算功能就是其中之一。了解其中告警的计算原理，才能让我们更好的运用它。