# 缓存
<!-- GFM-TOC -->
* [缓存](#缓存)
    * [一、缓存特征](#一缓存特征)
    * [二、缓存位置](#二缓存位置)
    * [三、CDN](#三cdn)
    * [四、缓存问题](#四缓存问题)
    * [五、数据分布](#五数据分布)
    * [六、一致性哈希](#六一致性哈希)
    * [七、LRU](#七lru)
    * [参考资料](#参考资料)
    <!-- GFM-TOC -->


## 一、缓存特征

### 命中率

当某个请求能够通过访问缓存而得到响应时，称为缓存命中。

缓存命中率越高，缓存的利用率也就越高。

### 最大空间

缓存通常位于内存中，内存的空间通常比磁盘空间小的多，因此缓存的最大空间不可能非常大。

当缓存存放的数据量超过最大空间时，就需要淘汰部分数据来存放新到达的数据。

### 淘汰策略

- FIFO（First In First Out）：先进先出策略，在实时性的场景下，需要经常访问最新的数据，那么就可以使用 FIFO，使得最先进入的数据（最晚的数据）被淘汰。

- LRU（Least Recently Used）：最近最久未使用策略，优先淘汰最久未使用的数据，也就是上次被访问时间距离现在最久的数据。该策略可以保证内存中的数据都是热点数据，也就是经常被访问的数据，从而保证缓存命中率。

- LFU（Least Frequently Used）：最不经常使用策略，优先淘汰一段时间内使用次数最少的数据。

## 二、缓存位置

### 浏览器

当 HTTP 响应允许进行缓存时，浏览器会将 HTML、CSS、JavaScript、图片等静态资源进行缓存。

### ISP

网络服务提供商（ISP）是网络访问的第一跳，通过将数据缓存在 ISP 中能够大大提高用户的访问速度。

### 反向代理

反向代理位于服务器之前，请求与响应都需要经过反向代理。通过将数据缓存在反向代理，在用户请求反向代理时就可以直接使用缓存进行响应。

### 本地缓存

使用 Guava Cache 将数据缓存在服务器本地内存中，服务器代码可以直接读取本地内存中的缓存，速度非常快。

### 分布式缓存

使用 Redis、Memcache 等分布式缓存将数据缓存在分布式缓存系统中。

相对于本地缓存来说，分布式缓存单独部署，可以根据需求分配硬件资源。不仅如此，服务器集群都可以访问分布式缓存，而本地缓存需要在服务器集群之间进行同步，实现难度和性能开销上都非常大。

### 数据库缓存

MySQL 等数据库管理系统具有自己的查询缓存机制来提高查询效率。

### Java 内部的缓存

Java 为了优化空间，提高字符串、基本数据类型包装类的创建效率，设计了字符串常量池及 Byte、Short、Character、Integer、Long、Boolean 这六种包装类缓冲池。

### CPU 多级缓存

CPU 为了解决运算速度与主存 IO 速度不匹配的问题，引入了多级缓存结构，同时使用 MESI 等缓存一致性协议来解决多核 CPU 缓存数据一致性的问题。

## 三、CDN

内容分发网络（Content distribution network，CDN）是一种互连的网络系统，它利用更靠近用户的服务器从而更快更可靠地将 HTML、CSS、JavaScript、音乐、图片、视频等静态资源分发给用户。

CDN 主要有以下优点：

- 更快地将数据分发给用户；
- 通过部署多台服务器，从而提高系统整体的带宽性能；
- 多台服务器可以看成是一种冗余机制，从而具有高可用性。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/15313ed8-a520-4799-a300-2b6b36be314f.jpg"/> </div><br>

## 四、缓存问题

### 缓存穿透

指的是对某个一定不存在的数据进行请求，该请求将会穿透缓存到达数据库。

解决方案：

- 对这些不存在的数据缓存一个空数据；
- 对这类请求进行过滤。

### 缓存雪崩

指的是由于数据没有被加载到缓存中，或者缓存数据在同一时间大面积失效（过期），又或者缓存服务器宕机，导致大量的请求都到达数据库。

在有缓存的系统中，系统非常依赖于缓存，缓存分担了很大一部分的数据请求。当发生缓存雪崩时，数据库无法处理这么大的请求，导致数据库崩溃。

解决方案：

- 为了防止缓存在同一时间大面积过期导致的缓存雪崩，可以通过观察用户行为，合理设置缓存过期时间来实现；
- 为了防止缓存服务器宕机出现的缓存雪崩，可以使用分布式缓存，分布式缓存中每一个节点只缓存部分的数据，当某个节点宕机时可以保证其它节点的缓存仍然可用。
- 也可以进行缓存预热，避免在系统刚启动不久由于还未将大量数据进行缓存而导致缓存雪崩。


### 缓存一致性

缓存一致性要求数据更新的同时缓存数据也能够实时更新。

解决方案：

- 在数据更新的同时立即去更新缓存；
- 在读缓存之前先判断缓存是否是最新的，如果不是最新的先进行更新。

要保证缓存一致性需要付出很大的代价，缓存数据最好是那些对一致性要求不高的数据，允许缓存数据存在一些脏数据。

### 缓存 “无底洞” 现象

指的是为了满足业务要求添加了大量缓存节点，但是性能不但没有好转反而下降了的现象。

产生原因：缓存系统通常采用 hash 函数将 key 映射到对应的缓存节点，随着缓存节点数目的增加，键值分布到更多的节点上，导致客户端一次批量操作会涉及多次网络操作，这意味着批量操作的耗时会随着节点数目的增加而不断增大。此外，网络连接数变多，对节点的性能也有一定影响。

解决方案：

- 优化批量数据操作命令；
- 减少网络通信次数；
- 降低接入成本，使用长连接 / 连接池，NIO 等。

## 五、数据分布

### 哈希分布

哈希分布就是将数据计算哈希值之后，按照哈希值分配到不同的节点上。例如有 N 个节点，数据的主键为 key，则将该数据分配的节点序号为：hash(key)%N。

传统的哈希分布算法存在一个问题：当节点数量变化时，也就是 N 值变化，那么几乎所有的数据都需要重新分布，将导致大量的数据迁移。

### 顺序分布

将数据划分为多个连续的部分，按数据的 ID 或者时间分布到不同节点上。例如 User 表的 ID 范围为 1 \~ 7000，使用顺序分布可以将其划分成多个子表，对应的主键范围为 1 \~ 1000，1001 \~ 2000，...，6001 \~ 7000。

顺序分布相比于哈希分布的主要优点如下：

- 能保持数据原有的顺序；
- 并且能够准确控制每台服务器存储的数据量，从而使得存储空间的利用率最大。

## 六、一致性哈希

Distributed Hash Table（DHT） 是一种哈希分布方式，其目的是为了克服传统哈希分布在服务器节点数量变化时大量数据迁移的问题。

### 基本原理

将哈希空间 [0, 2<sup>n</sup>-1] 看成一个哈希环，每个服务器节点都配置到哈希环上。每个数据对象通过哈希取模得到哈希值之后，存放到哈希环中顺时针方向第一个大于等于该哈希值的节点上。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/68b110b9-76c6-4ee2-b541-4145e65adb3e.jpg"/> </div><br>

一致性哈希在增加或者删除节点时只会影响到哈希环中相邻的节点，例如下图中新增节点 X，只需要将它前一个节点 C 上的数据重新进行分布即可，对于节点 A、B、D 都没有影响。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/66402828-fb2b-418f-83f6-82153491bcfe.jpg"/> </div><br>

### 虚拟节点

上面描述的一致性哈希存在数据分布不均匀的问题，节点存储的数据量有可能会存在很大的不同。

数据不均匀主要是因为节点在哈希环上分布的不均匀，这种情况在节点数量很少的情况下尤其明显。

解决方式是通过增加虚拟节点，然后将虚拟节点映射到真实节点上。虚拟节点的数量比真实节点来得多，那么虚拟节点在哈希环上分布的均匀性就会比原来的真实节点好，从而使得数据分布也更加均匀。

## 七、LRU

以下是基于 双向链表 + HashMap 的 LRU 算法实现，对算法的解释如下：

- 访问某个节点时，将其从原来的位置删除，并重新插入到链表头部。这样就能保证链表尾部存储的就是最近最久未使用的节点，当节点数量大于缓存最大空间时就淘汰链表尾部的节点。
- 为了使删除操作时间复杂度为 O(1)，就不能采用遍历的方式找到某个节点。HashMap 存储着 Key 到节点的映射，通过 Key 就能以 O(1) 的时间得到节点，然后再以 O(1) 的时间将其从双向队列中删除。

```java
public class LRU<K, V> implements Iterable<K> {

    private Node head;
    private Node tail;
    private HashMap<K, Node> map;
    private int maxSize;

    private class Node {

        Node pre;
        Node next;
        K k;
        V v;

        public Node(K k, V v) {
            this.k = k;
            this.v = v;
        }
    }


    public LRU(int maxSize) {

        this.maxSize = maxSize;
        this.map = new HashMap<>(maxSize * 4 / 3);

        head = new Node(null, null);
        tail = new Node(null, null);

        head.next = tail;
        tail.pre = head;
    }


    public V get(K key) {

        if (!map.containsKey(key)) {
            return null;
        }

        Node node = map.get(key);
        unlink(node);
        appendHead(node);

        return node.v;
    }


    public void put(K key, V value) {

        if (map.containsKey(key)) {
            Node node = map.get(key);
            unlink(node);
        }

        Node node = new Node(key, value);
        map.put(key, node);
        appendHead(node);

        if (map.size() > maxSize) {
            Node toRemove = removeTail();
            map.remove(toRemove.k);
        }
    }


    private void unlink(Node node) {

        Node pre = node.pre;
        Node next = node.next;

        pre.next = next;
        next.pre = pre;

        node.pre = null;
        node.next = null;
    }


    private void appendHead(Node node) {
        Node next = head.next;
        node.next = next;
        next.pre = node;
        node.pre = head;
        head.next = node;
    }


    private Node removeTail() {

        Node node = tail.pre;

        Node pre = node.pre;
        tail.pre = pre;
        pre.next = tail;

        node.pre = null;
        node.next = null;

        return node;
    }


    @Override
    public Iterator<K> iterator() {

        return new Iterator<K>() {
            private Node cur = head.next;

            @Override
            public boolean hasNext() {
                return cur != tail;
            }

            @Override
            public K next() {
                Node node = cur;
                cur = cur.next;
                return node.k;
            }
        };
    }
}
```

## 参考资料

- 大规模分布式存储系统
- [缓存那些事](https://tech.meituan.com/cache_about.html)
- [一致性哈希算法](https://my.oschina.net/jayhu/blog/732849)
- [内容分发网络](https://zh.wikipedia.org/wiki/%E5%85%A7%E5%AE%B9%E5%82%B3%E9%81%9E%E7%B6%B2%E8%B7%AF)
- [How Aspiration CDN helps to improve your website loading speed?](https://www.aspirationhosting.com/aspiration-cdn/)

# 分布式缓存

## 淘汰算法

### 最不经常使用算法(LFU)

这个缓存算法使用一个计数器来记录条目被访问的频率。通过使用LFU缓存算法，最低访问数的条目首先被移除。这个方法并不经常使用，因为它无法对一个拥有最初高访问率之后长时间没有被访问的条目缓存负责。

![最不经常使用算法（LFU）](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/最不经常使用算法（LFU）.png)



### 最近最少使用算法(LRU)

这个缓存算法将最近使用的条目存放到靠近缓存顶部的位置。当一个新条目被访问时，LRU将它放置到缓存的顶部。当缓存达到极限时，较早之前访问的条目将从缓存底部开始被移除。这里会使用到昂贵的算法，而且它需要记录“年龄位”来精确显示条目是何时被访问的。此外，当一个LRU缓存算法删除某个条目后，“年龄位”将随其他条目发生改变。

![最近最少使用算法（LRU）](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/最近最少使用算法（LRU）.png)

一个缓存被访问后，近期再被访问的可能性很大。可以记录每个缓存记录的最近访问时间，最近未被访问时间最长的数据会被首先淘汰。

优点：实现简单，能适应访问热点
缺点：对偶发的访问敏感，影响命中率
簿记开销：时间 `O(1)`，空间 `O(N)`



### 自适应缓存替换算法(ARC)

在IBM Almaden研究中心开发，这个缓存算法同时跟踪记录LFU和LRU，以及驱逐缓存条目，来获得可用缓存的最佳使用。



### 先进先出算法(FIFO)

FIFO是英文First In First Out 的缩写，是一种先进先出的数据缓存器，他与普通存储器的区别是没有外部读写地址线，这样使用起来非常简单，但缺点就是只能顺序写入数据，顺序的读出数据，其数据地址由内部读写指针自动加1完成，不能像普通存储器那样可以由地址线决定读取或写入某个指定的地址。

![先进先出算法（FIFO）](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/先进先出算法（FIFO）.png)

越早进入缓存的数据，其不再被访问的可能性越大。因此在淘汰缓存时，应选择在内存中停留时间最长的缓存记录。使用队列即可实现该策略：

![缓存淘汰策略-FIFO](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存淘汰策略-FIFO.png)

优点：实现简单，适合线性访问的场景
缺点：无法适应特定的访问热点，缓存的命中率差
簿记开销：时间 `O(1)`，空间 `O(N)`



### 最近最常使用算法(MRU)

这个缓存算法最先移除最近最常使用的条目。一个MRU算法擅长处理一个条目越久，越容易被访问的情况。



## 更新策略

缓存更新的策略主要分为三种：

- **Cache Aside Pattern（旁路缓存）**
- **Read/Write Through Pattern（读写穿透）**
- **Write Behind Caching Pattern（异步写入）**



**缓存使用场景**

**分布式系统中要么通过2PC、3PC或Paxos协议保证强一致性，要么就是拼命的降低并发时脏数据的概率**。缓存系统适用的场景就是非强一致性的场景，所以它属于CAP中的AP，只能做到BASE理论中说的**最终一致性**。异构数据库本来就没办法强一致，我们只是**尽可能减少不一致的时间窗口，达到最终一致性**。同时结合设置过期时间的兜底方案。



**缓存场景分析**

- 对于读多写少的数据，请使用缓存
- 为了保持数据库和缓存的一致性，会导致系统吞吐量的下降
- 为了保持数据库和缓存的一致性，会导致业务代码逻辑复杂
- 缓存做不到绝对一致性，但可以做到最终一致性
- 对于需要保证缓存数据库数据一致的情况，请尽量考虑对一致性到底有多高要求，选定合适的方案，避免过度设计



### Cache Aside(旁路缓存)

`Cache Aside（旁路缓存）` 是最广泛使用的缓存模式之一，如果能正确使用 `Cache Aside` 的话，能极大的提升应用性能，`Cache Aside`可用来读或写操作。`Cache Aside`的提出是为了尽可能地解决缓存与数据库的数据不一致问题。

#### Read Cache Aside

`Cache Aside` 的读请求流程如下：

![Cache-Aside读请求](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Cache-Aside读请求.jpg)

- **读的时候，先读缓存，缓存命中的话，直接返回数据**
- **缓存没有命中的话，就去读数据库，从数据库取出数据，放入缓存后，同时返回响应**



#### Write Cache Aside

`Cache Aside` 的写请求流程如下：

![Cache-Aside写请求](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Cache-Aside写请求.jpg)

- **更新的时候，先更新数据库，然后再删除缓存**



### Read/Write Through(读写穿透)

`Read/Write Through（读写穿透）` 模式中，服务端把缓存作为主要数据存储。应用程序跟数据库缓存交互，都是通过**抽象缓存层**完成的。

#### Read Through

`Read Through` 和 `Cache Aside` 很相似，不同点在于程序不需要再去管理从哪去读数据（缓存还是数据库）。相反它会直接从缓存中读数据，该场景下是缓存去决定从哪查询数据。当我们比较两者的时候这是一个优势因为它会让程序代码变得更简洁。`Read Through`的简要流程如下

![Read-Through简要流程](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Read-Through简要流程.png)

- **从缓存读取数据，读到直接返回**
- **如果读取不到的话，从数据库加载，写入缓存后，再返回响应**



该模式只在 `Cache Aside` 之上进行了一层封装，它会让程序代码变得更简洁，同时也减少数据源上的负载。流程如下：

![Read-Through流程](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Read-Through流程.png)



#### Write Through

`Write Through` 模式下的所有写操作都经过缓存，每次向缓存中写数据时，缓存会把数据持久化到对应的数据库中去，且这两个操作都在一个事务中完成。因此，只有两次都写成功才是最终写成功。用写延迟保证了数据一致性。当发生写请求时，也是由**缓存抽象层**完成数据源和缓存数据的更新，流程如下：

![Write-Through](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Write-Through.png)

- **向缓存中写数据，并向数据库写数据**
- **使用事务保证一致性，两者都写成功才成功**



当使用 `Write Through `的时候一般都配合使用 `Read Through`。`Write Through `适用情况有：

- **需要频繁读取相同数据**
- **不能忍受数据丢失（相对 `Write Behind` 而言）和数据不一致**

**`Write Through` 的潜在使用例子是银行系统。**



### Write Behind(异步写入)

`Write Behind（异步写入，又叫Write Back）` 和 `Read/Write Through` 相似，都是由 `Cache Provider` 来负责缓存和数据库的读写。它们又有个很大的不同：`Read/Write Through` 是同步更新缓存和数据的，`Write Behind` 则是只更新缓存，不直接更新数据库，通过**批量异步**的方式来更新数据库。

![WriteBehind流程](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/WriteBehind流程.png)

这种方式下，缓存和数据库的一致性不强，**对一致性要求高的系统要谨慎使用**。但是它适合频繁写的场景，MySQL的**InnoDB Buffer Pool 机制**就使用到这种模式。如上图，应用程序更新两个数据，Cache Provider 会立即写入缓存中，但是隔一段时间才会批量写入数据库中。优缺点如下：

- **优点**：是数据写入速度非常快，适用于频繁写的场景
- **缺点**：是缓存和数据库不是强一致性，对一致性要求高的系统慎用



## 数据一致性

![缓存双写一致性](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存双写一致性.png)

一致性就是数据保持一致，在分布式系统中，可以理解为多个节点中数据的值是一致的。

- **强一致性**：这种一致性级别是最符合用户直觉的，它要求系统写入什么，读出来的也会是什么，用户体验好，但实现起来往往对系统的性能影响大
- **弱一致性**：这种一致性级别约束了系统在写入成功后，不承诺立即可以读到写入的值，也不承诺多久之后数据能够达到一致，但会尽可能地保证到某个时间级别（比如秒级别）后，数据能够达到一致状态
- **最终一致性**：最终一致性是弱一致性的一个特例，系统会保证在一定时间内，能够达到一个数据一致的状态。这里之所以将最终一致性单独提出来，是因为它是弱一致性中非常推崇的一种一致性模型，也是业界在大型分布式系统的数据一致性上比较推崇的模型

### 业务延时双删

先删除缓存，再更新数据库中如何避免脏数据？采用延时双删策略。

![延时双删流程](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/延时双删流程.png)

- **先删除缓存**
- **再写数据库**
- **休眠1秒后再次删除缓存**（这1秒=业务可能最大耗时，主要是等待正在加载脏数据的请求完成）



**① 读写分离架构**

读写架构中，先删除缓存，再更新数据库中如何避免脏数据？采用延时双删策略。

- **先淘汰缓存**
- **再写数据库**
- **休眠1秒后再次淘汰缓存**（这1秒=主从同步可能最大耗时+业务可能最大耗时，主要是等待正在加载脏数据的请求完成）



**② 延时双删导致吞吐量降低**

延时双删的方式同步淘汰策略导致了吞吐量降低如何解决？

- **将第二次删除作为异步**



### MQ重试机制

不管是**延时双删**还是**Cache-Aside的先操作数据库再删除缓存**，都可能会存在第二步的删除缓存失败，导致的数据不一致问题，可以引入**删除缓存重试机制**来解决。

![删缓存失败-解决方案一](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/删缓存失败-解决方案一.png)

![缓存一致性-基于MQ的解决方案](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存一致性-基于MQ的解决方案.jpg)

流程如下：

- **更新数据库数据**
- **删除缓存中的数据，可此时缓存服务出现不可用情况，造成无法删除缓存数据**
- **当删除缓存数据失败时，将需要删除缓存的 Key 发送到消息队列 (MQ) 中**
- **应用自己消费需要删除缓存 Key 的消息**
- **应用接收到消息后，删除缓存，如果删除缓存确认 MQ 消息被消费，如果删除缓存失败，则让消息重新入队列，进行多次尝试删除缓存操作**



### biglog异步删除

重试删除缓存机制会造成好多**业务代码入侵**。其实还可以这样优化：通过数据库的**binlog来异步淘汰key**。

![删缓存失败-解决方案二](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/删缓存失败-解决方案二.png)

![缓存一致性-基于Canal的解决方案](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存一致性-基于Canal的解决方案.jpg)

流程如下：

- **更新数据库数据**
- **MySQL将数据更新日志写入binlog中**
- **Canal订阅&消费MySQL binlog，并提取出被更新数据的表名及ID**
- **调用应用删除缓存接口**
- **删除缓存数据**
- **Redis 不可用时，将更新数据的表名及 ID 发送到 MQ 中**
- **应用接收到消息后，删除缓存，如果删除缓存确认 MQ 消息被消费，如果删除缓存失败，则让消息重新入队列，进行多次尝试删除缓存操作，直到缓存删除成功为止**



## 策略选择

### 删除or更新

操作缓存的时候，到底是删除缓存呢，还是更新缓存？日常开发中，我们一般使用的就是`Cache-Aside`模式。有些小伙伴可能会问， `Cache-Aside`在写入请求的时候，为什么是**删除缓存而不是更新缓存**呢？

![Cache-Aside写入流程](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Cache-Aside写入流程.png)

我们在操作缓存的时候，到底应该删除缓存还是更新缓存呢？我们先来看个例子：

![Cache-Aside写入流程案例](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Cache-Aside写入流程案例.png)

1. 线程A先发起一个写操作，第一步先更新数据库
2. 线程B再发起一个写操作，第二步更新了数据库
3. 由于网络等原因，线程B先更新了缓存
4. 线程A更新缓存

这时候，缓存保存的是A的数据（老数据），数据库保存的是B的数据（新数据），数据**不一致**了，脏数据出现啦。如果是**删除缓存取代更新缓存**则不会出现这个脏数据问题。**更新缓存相对于删除缓存**，还有两点劣势：

- 如果你写入的缓存值，是经过复杂计算才得到的话。更新缓存频率高的话，就浪费性能啦
- 在写数据库场景多，读数据场景少的情况下，数据很多时候还没被读取到，又被更新了，这也浪费了性能呢(实际上，写多的场景，用缓存也不是很划算的)



### 双写顺序

双写的情况下，先操作数据库还是先操作缓存？`Cache-Aside` 缓存模式中，有些小伙伴还是会有疑问，在写请求过来的时候，为什么是**先操作数据库呢**？为什么**不先操作缓存**呢？比如一条数据同时存在数据库、缓存，现在你要更新此数据，不管先更新数据库，还是先更新缓存，这两种方式都有问题。

**方案一：先更新数据库，后更新缓存**

![双写顺序-先DB后缓存](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/双写顺序-先DB后缓存.jpg)

如下案例会造成数据不一致。

A先把数据库更新为 123，由于网络问题，更新缓存的动作慢了。这时，B 去更新数据库了，改为了 456，紧接着把缓存也更新为456。现在A更新缓存的请求到了，把缓存更新为了 123。那么这时数据就不一致了，数据库里是最新的 456，而缓存是 123，是旧数据。因为数据库更新、缓存更新这2个动作不是原子的，在高并发操作时，这2个动作直接会插入其他动作。

![双写顺序-先DB后缓存-案例](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/双写顺序-先DB后缓存-案例.jpg)



**方案二：先更新缓存，再更新数据库**

![双写顺序-先缓存后DB](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/双写顺序-先缓存后DB.jpg)

如下案例也同样可能数据不一致。

缓存更新成功，数据为最新的，但数据库更新失败，回滚了，还是旧数据。还是非原子操作的原因。

![双写顺序-先缓存后DB-案例](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/双写顺序-先缓存后DB-案例.jpg)

**注意**：先操作数据库再操作缓存，不一样也会导致数据不一致嘛？它俩又不是原子性操作的。这个是**会的**，但是这种方式，一般因为删除缓存失败等原因，才会导致脏数据，这个概率就很低。接下来我们再来分析这种**删除缓存失败**的情况，**如何保证一致性**。



### 缓存更新

除了缓存服务器自带的缓存失效策略之外（Redis默认的有6中策略可供选择），还可以根据具体的业务需求进行自定义的缓存淘汰。常见的更新策略如下：

- **LRU/LFU/FIFO**：都是属于当**缓存不够用**时采用的更新算法

  适合内存空间有限，数据长期不变动，基本不存在数据一不致性业务。比如一些一经确定就不允许变更的信息。

- **超时剔除**：给缓存数据设置一个过期时间

  适合于能够容忍一定时间内数据不一致性的业务，比如促销活动的描述文案。

- **主动更新**：如果数据源的数据有更新，则主动更新缓存

  对于数据的一致性要求很高，比如交易系统，优惠劵的总张数。



常见数据更新方式有两大类，其余基本都是这两类的变种：

**方式一：先删缓存，再更新数据库**

![缓存更新-先删缓存再更新数据库](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存更新-先删缓存再更新数据库.png)

这种做法是遇到数据更新，我们先去删除缓存，然后再去更新DB，如左图。让我们来看一下整个操作的流程：

- A请求需要更新数据，先删除对应的缓存，还未更新DB
- B请求来读取数据
- B请求看到缓存里没有，就去读取DB并将旧数据写入缓存（脏数据）
- A请求更新DB

可以看到B请求将脏数据写入了缓存，如果这是一个读多写少的数据，可能脏数据会存在比较长的时间（要么有后续更新，要么等待缓存过期），这是业务上不能接受的。



**方式二：先更新数据库，再删除缓存**

![缓存更新-先更新数据库再删除缓存](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存更新-先更新数据库再删除缓存.png)

上图的右侧部分可以看到在A更新DB和删除缓存之间B请求会读取到老数据，因为此时A操作还没有完成，并且这种读到老数据的时间是非常短的，可以满足数据最终一致性要求。



**删除缓存而非更新缓存原因**

上图可以看到我们用的是删除缓存，而不是更新缓存，原因如下图：

![缓存更新-删除缓存原因](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存更新-删除缓存原因.png)

上图我用操作代替了删除或更新，当我们做删除操作时，A先删还是B先删没有关系，因为后续读取请求都会从DB加载出最新数据；但是当我们对缓存做的是更新操作时，就会对A先更新缓存还是B先更新缓存敏感了，如果A后更新，那么缓存里就又存在脏数据了，所以 go-zero 只使用删除缓存的方式。



**缓存更新请求处理流程**

我们来一起看看完整的请求处理流程：

![缓存更新-请求处理流程](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存更新-请求处理流程.png)

**注意**：不同颜色代表不同请求。

- 请求1更新DB
- 请求2查询同一个数据，返回了老的数据，这个短时间内返回旧数据是可以接受的，满足最终一致性
- 请求1删除缓存
- 请求3再来请求时缓存里没有，就会查询数据库，并回写缓存再返回结果
- 后续的请求就会直接读取缓存了



## 缓存问题

### 缓存雪崩

当**大量缓存数据在同一时间过期（失效）或者 Redis 故障宕机**时，如果此时有大量的用户请求，都无法在 Redis 中处理，于是全部请求都直接访问数据库，从而导致数据库的压力骤增，严重的会造成数据库宕机，从而形成一系列连锁反应，造成整个系统崩溃，这就是**缓存雪崩**的问题。

![缓存雪崩](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存雪崩.png)

发生缓存雪崩的原因及解决方案：

**① 大量数据同时过期**

- **均匀设置过期时间**：给缓存数据的过期时间加上一个随机数
- **互斥锁**：加个互斥锁，保证同一时间内只有一个请求来构建缓存
- **二级缓存**：每一级缓存的失效时间都不同
- **队列控制**：使用MQ控制读取数据库的请求数据。即发N个消息，单线程消费
- **热点数据缓存永远不过期**
  - **物理不过期**：针对热点key不设置过期时间
  - **逻辑过期**：把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建

**② Redis故障宕机**

- **服务熔断或请求限流机制**：启动服务熔断机制，暂停业务对缓存服务访问，直接返回错误；或启用限流机制
- **构建Redis缓存高可靠集群**：通过主从节点的方式构建Redis缓存高可靠集群
- **开启Redis持久化机制**：一旦重启，能直接从磁盘上自动加载数据恢复内存中的数据



### 缓存击穿

如果缓存中的**某个热点数据过期**了，此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库很容易就被高并发的请求冲垮，这就是**缓存击穿**的问题，如秒杀活动。

![缓存击穿](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存击穿.png)

缓存击穿跟缓存雪崩很相似，可以认为缓存击穿是缓存雪崩的一个子集。应对缓存击穿可以采取以下两种方案：

- **互斥锁方案**：保证同一时间只有一个业务线程更新缓存，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值

- **热点数据缓存永远不过期**

  - **物理不过期**：针对热点key不设置过期时间
  - **逻辑过期**：把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建

  

### 缓存穿透

当用户访问的数据，**既不在缓存中，也不在数据库中**，导致请求在访问缓存时，发现缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据，没办法构建缓存数据来服务后续的请求。那么当有大量这样的请求到来时，数据库的压力骤增，这就是**缓存穿透**的问题。

![缓存穿透](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存穿透.png)

缓存穿透的发生一般有这两种情况：

- **业务误操作**：缓存中的数据和数据库中的数据都被误删除了，所以导致缓存和数据库中都没有数据
- **黑客恶意攻击**：故意大量访问某些读取不存在数据的业务

应对缓存穿透的方案，常见的方案有三种：

- **拦截非法请求**

- **布隆过滤器**：构造一个`BloomFilter`过滤器，初始化全量数据，当接到请求时，在`BloomFilter`中判断这个key是否存在，如果不存在，直接返回即可，无需再查询`缓存和DB`

- **缓存空对象**：查存DB 时，如果数据不存在，预热一个`特殊空值`到缓存中。这样，后续查询都会命中缓存，但是要对特殊值，解析处理

  ![缓存空对象](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/缓存空对象.png)



**布隆过滤器工作原理**

布隆过滤器由「**初始值都为 0 的位图数组**」和「 **N 个哈希函数**」两部分组成。当我们在写入数据库数据时，在布隆过滤器里做个标记，这样下次查询数据是否在数据库时，只需要查询布隆过滤器，如果查询到数据没有被标记，说明不在数据库中。布隆过滤器会通过 3 个操作完成标记：

- 第一步：使用 N 个哈希函数分别对数据做哈希计算，得到 N 个哈希值
- 第二步：将第一步得到的 N 个哈希值对位图数组的长度取模，得到每个哈希值在位图数组的对应位置
- 第三步：将每个哈希值在位图数组的对应位置的值设置为 1

举个例子，假设有一个位图数组长度为 8，哈希函数 3 个的布隆过滤器：

![布隆过滤器](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/布隆过滤器.jpg)

![Guava布隆过滤器](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Guava布隆过滤器.jpg)



## Hot Key

### 产生原因

- **用户消费的数据远大于生产的数据**（热卖商品、热点新闻、热点评论、明星直播）

  在日常工作生活中一些突发的的事件，例如：双十一期间某些热门商品的降价促销，当这其中的某一件商品被数万次点击浏览或者购买时，会形成一个较大的需求量，这种情况下就会造成热点问题。同理，被大量刊发、浏览的热点新闻、热点评论、明星直播等，这些典型的读多写少的场景也会产生热点问题。

- **请求分片集中，超过单 Server 的性能极限**

  在服务端读数据进行访问时，往往会对数据进行分片切分，此过程中会在某一主机 Server 上对相应的 Key 进行访问，当访问超过 Server 极限时，就会导致热点 Key 问题的产生。



### 问题危害

当某一热点 Key 的请求在某一主机上超过该主机网卡上限时，由于流量的过度集中，会导致服务器中其它服务无法进行。如果热点过于集中，热点 Key 的缓存过多，超过目前的缓存容量时，就会导致缓存分片服务被打垮现象的产生。当缓存服务崩溃后，此时再有请求产生，会缓存到后台 DB 上，由于DB 本身性能较弱，在面临大请求时很容易发生请求穿透现象，会进一步导致雪崩现象，严重影响设备的性能。

- **流量集中，达到物理网卡上限**
- **请求过多，缓存分片服务被打垮**
- **DB 击穿，引起业务雪崩**



### 发现热key

- 预估热key，如秒杀商品，火爆新闻
- 在客户端进行统计
- 可用Proxy，如Codis可以在Proxy端收集
- 利用redis自带命令，monitor，hotkeys。**执行缓慢，不推荐使用**
- 利用流式计算引擎统计访问次数，如Storm、Spark Streaming、Flink



### 解决方案

通常的解决方案主要集中在对客户端和 Server 端进行相应的改造。

#### 读写分离

![读写分离方案解决热读](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/读写分离方案解决热读.jpg)

架构中各节点的作用如下：

- **SLB 层做负载均衡**
- **Proxy 层做读写分离自动路由**
- **Master 负责写请求**
- **ReadOnly 节点负责读请求**
- **Slave 节点和 Master 节点做高可用**

实际过程中 Client 将请求传到 SLB，SLB 又将其分发至多个 Proxy 内，通过 Proxy 对请求的识别，将其进行分类发送。例如，将同为 Write 的请求发送到 Master 模块内，而将 Read 的请求发送至 ReadOnly 模块。而模块中的只读节点可以进一步扩充，从而有效解决热点读的问题。读写分离同时具有可以灵活扩容读热点能力、可以存储大量热点Key、对客户端友好等优点。



#### 热点数据

![热点数据解决方案](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/热点数据解决方案.jpg)

该方案通过主动发现热点并对其进行存储来解决热点 Key 的问题。首先 Client 也会访问 SLB，并且通过 SLB 将各种请求分发至 Proxy 中，Proxy 会按照基于路由的方式将请求转发至后端的 Redis 中。在热点 key 的解决上是采用在服务端增加缓存的方式进行。具体来说就是在 Proxy 上增加本地缓存，本地缓存采用 LRU 算法来缓存热点数据，后端 DB 节点增加热点数据计算模块来返回热点数据。Proxy 架构的主要有以下优点：

- **Proxy 本地缓存热点，读能力可水平扩展**
- **DB 节点定时计算热点数据集合**
- **DB 反馈 Proxy 热点数据**
- **对客户端完全透明，不需做任何兼容**



### 热点key处理

####  热点数据的读取

![热点数据的读取](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/热点数据的读取.jpg)

在热点 Key 的处理上主要分为写入跟读取两种形式，在数据写入过程当 SLB 收到数据 K1 并将其通过某一个 Proxy 写入一个 Redis，完成数据的写入。假若经过后端热点模块计算发现 K1 成为热点 key 后， Proxy 会将该热点进行缓存，当下次客户端再进行访问K1时，可以不经Redis。最后由于Proxy是可以水平扩充的，因此可以任意增强热点数据的访问能力。



####  热点数据的发现

![热点数据的发现](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/热点数据的发现.jpg)

对于 db 上热点数据的发现，首先会在一个周期内对 Key 进行请求统计，在达到请求量级后会对热点 Key 进行热点定位，并将所有的热点 Key 放入一个小的 LRU 链表内，在通过 Proxy 请求进行访问时，若 Redis 发现待访点是一个热点，就会进入一个反馈阶段，同时对该数据进行标记。DB 计算热点时，主要运用的方法和优势有：

- **基于统计阀值的热点统计**
- **基于统计周期的热点统计**
- **基于版本号实现的无需重置初值统计方法**
- **DB 计算同时具有对性能影响极其微小、内存占用极其微小等优点**



## Big Key

Big Key指数据量大的key，由于其数据大小远大于其它key，导致经过分片之后，某个具体存储这个Big Key的实例内存使用量远大于其他实例，造成内存不足，拖累整个集群的使用。

### 常见场景

- 热门话题下的讨论
- 大V的粉丝列表
- 序列化后的图片
- 没有及时处理的垃圾数据



### 大Key影响

- 大key会大量占用内存，在Redis集群中无法均衡
- Reids性能下降，影响主从复制
- 在主动删除或过期删除时操作时间过长而引起服务阻塞



### 如何发现大Key

- redis-cli --bigkeys命令。可以找到某个实例5种数据类型(String、hash、list、set、zset)的最大key。但如果Redis的key比较多，执行该命令会比较慢
- 获取生产Redis的rdb文件，通过rdbtools分析rdb生成csv文件，再导入MySQL或其他数据库中进行分析统计，根据size_in_bytes统计bigkey



### 解决方案

优化big key的原则就是string减少字符串长度，而list、hash、set、zset等则减少成员数：

- string类型的big key，尽量不要存入Redis中，可以使用文档型数据库MongoDB或缓存到CDN上。如果必须用Redis存储，最好单独存储，不要和其他的key一起存储

- 设置一个阈值，当value的长度超过阈值时，对内容启动压缩，降低kv的大小
- 评估`大key`所占的比例，由于很多框架采用`池化技术`，如：Memcache，可以预先分配大对象空间。真正业务请求时，直接拿来即用
- 颗粒划分，将大key拆分为多个小key，独立维护，成本会降低不少
- 大key要设置合理的过期时间，尽量不淘汰那些大key



## 其它问题

### 缓存预热

缓存预热是指系统上线后，提前将相关的缓存数据加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题，用户直接查询事先被预热的缓存数据。如果不进行预热，那么Redis初始状态数据为空，系统上线初期，对于高并发的流量，都会访问到数据库中， 对数据库造成流量的压力。

**缓存预热思路**

- **数据量不大的时候**：工程启动的时候进行加载缓存动作
- **数据量大的时候**：设置一个定时任务脚本，进行缓存的刷新
- **数据量太大的时候**：优先保证热点数据进行提前加载到缓存



**预热解决方案**

- 直接写个缓存刷新页面，上线时手工操作下
- 数据量不大，可以在项目启动的时候自动进行加载
- 定时刷新缓存



**缓存加载策略**

- **使用时加载缓存**。当需要使用缓存数据时，就从数据库中查出，第一次查出后，接下来的请求都能从缓存中查询到数据
- **预加载缓存**。在项目启动的时候，预加载类似“国家信息、货币信息、用户信息，新闻信息”等不是经常变更的数据



### 缓存降级

缓存降级是指当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

**降级的最终目的是保证核心服务可用，即使是有损的**。而且有些服务是无法降级的（如加入购物车、结算）。



**分级降级预案：**

- **一般**：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级
- **警告**：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警
- **错误**：比如可用率低于90%，或数据库连接池被打爆，或访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级
- **严重错误**：比如因为特殊原因数据错误了，此时需要紧急人工降级



### 热点数据

数据库里有2000W数据，Redis中只存20W的数据，如何保证 Redis 中的数据都是热点数据？

当 Redis 中的数据集上升到一定大小的时候，就需要实施数据淘汰策略，以保证 Redis 的内存不会被撑爆；那么如何保证 Redis 中的数据都是热点数据，需要先看看 Redis 有哪些数据淘汰策略。

**Redis 数据淘汰策略**

- volatile-lru：从已设置过期时间的数据集中，挑选最近最少使用的数据淘汰
- volatile-ttl：从已设置过期时间的数据集中，挑选将要过期的数据淘汰
- volatile-random：从已设置过期时间的数据集中，任意选择数据淘汰
- allkeys-lru：当内存不足以容纳新写入数据时，移除最近最少使用的key
- allkeys-random：从数据集中任意选择数据淘汰
- no-eviction：禁止淘汰数据，也就是说当内存不足时，新写入操作会报错

**到了4.0版本后，又增加以下两种淘汰策略：**

- volatile-lfu：从已设置过期时间的数据集中，挑选最不经常使用的数据淘汰（注意lfu和lru的区别）
- allkeys-lfu：当内存不足以容纳新写入数据时，移除最不经常使用的key

**如何选择策略规则**

针对题目中的问题，还需要考虑数据的分布情况：

- 如果数据呈现幂律分布，一部分数据访问频率高，一部分数据访问频率低，则可以使用allkeys-lru或allkeys-lfu
- 如果数据呈现平等分布，所有的数据访问频率都相同，则使用allkeys-random





# Caffeine简介

在本文中，我们来看看 [Caffeine](https://github.com/ben-manes/caffeine) — 一个高性能的 Java 缓存库。

Caffeine的底层数据存储采用ConcurrentHashMap。因为Caffeine面向JDK8，在jdk8中ConcurrentHashMap增加了红黑树，在hash冲突严重时也能有良好的读性能。

> Caffeine VS guava

Caffeine是Spring 5默认支持的Cache，可见Spring对它的看重，Spring抛弃Guava转向了Caffeine。

> 缓存和 Map 之间的一个根本区别在于缓存可以回收存储的 item。

回收策略为在指定时间删除哪些对象。此策略直接影响缓存的命中率 — 缓存库的一个重要特征。

Caffeine 因使用 Window TinyLfu 回收策略，提供了一个近乎最佳的命中率。

## Caffeine是Spring 5默认支持的Cache

Caffeine是Spring 5默认支持的Cache，可见Spring对它的看重，那么Spring为什么喜新厌旧的抛弃Guava而追求Caffeine呢？

> 它能提供高命中率和出色的并发能力。
> 缓存的淘汰策略是为了预测哪些数据在短期内最可能被再次用到，从而提升缓存的命中率。LRU由于实现简单、高效的运行时表现以及在常规的使用场景下有不错的命中率，或许是目前最佳的实现途径。但 LRU 通过历史数据来预测未来是局限的，它会认为最后到来的数据是最可能被再次访问的，从而给与它最高的优先级。这样就意味着淘汰真正热点数据，为了解决这个问题业界运用一些数据结构上的改进巧妙的解决这个问题。

### 举个例子

1. Mysql的缓存池，内部实现是一个LRU，但是其内部有个**中间点,**指向倒数3/8，一半是old区，另一半是young区，新数据插入是直接插入young区，这样就保护了真正的老数据不会被冲刷掉。
2. 多级队列的形式

LFU结合频率这一属性给予更好的预测缓存数据是否在未来被使用。

但是传统LFU有其局限性：

- LFU实现需要维护大而复杂的元数据（频次统计数据等）
- 大多数实际工作负载中，访问频率随着时间的推移而发生根本变化，而传统LFU无法周期衰减频率

传统LFU的实现通过外接一个HashMap统计频率，但是HashMap存在Hash冲突，这会导致频率统计的不准确。

> 为了解决这些问题，Caffeine提出一种新的算法W-TinyLFU，它可以解决频率统计不准确以及访问频率衰减问题。这个方法让我们从空间、效率、以及适配矩阵的长宽引起的哈希碰撞的错误率上做权衡。

传统Hash存在Hash冲突的问题，使用LFU算法时候记录频率的话一旦发生hash冲突可能造成频率的统计错误。

> W-TinyLFU算法使用一种Count-Min Sketch解决维护空间大的问题，类似布隆过滤器，降低冲突可能性，原理是多次hash分散开来，取最小值作为频率，一次Hash冲突的几率是1%的话，4次Hash的几率就是1%的4次方，大大降低的冲突可能性。

# 常见的缓存数据淘汰算法

一个缓存组件是否好用，其中一个重要的指标就是他的缓存命中率，而命中率又和缓存组件本身的缓存数据淘汰算法息息相关，本文意在讲解一些业界内常见的页面置换算法，以及介绍下Caffeine Cache的W-TinyLFU算法，以便于更好的理解缓存组件。

## 1 FIFO

FIFO（First in First out）先进先出。可以理解为是一种类似队列的算法实现

- 算法：最先进来的数据，被认为在未来被访问的概率也是最低的，因此，当规定空间用尽且需要放入新数据的时候，会优先淘汰最早进来的数据
- 优点：最简单、最公平的一种数据淘汰算法，逻辑简单清晰，易于实现
- 缺点：这种算法逻辑设计所实现的缓存的命中率是比较低的，因为没有任何额外逻辑能够尽可能的保证常用数据不被淘汰掉

下面简单演示了FIFO的工作过程，假设存放元素尺寸是3，且队列已满，放置元素顺序如下图所示，当来了一个新的数据“ldy”后，因为元素数量到达了阈值，则首先要进行太淘汰置换操作，然后加入新元素，操作如图展示：
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914175109745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200914175109745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)

## 2 LRU

LRU（The Least Recently Used）最近最久未使用算法。相比于FIFO算法智能些

- 算法：如果一个数据最近很少被访问到，那么被认为在未来被访问的概率也是最低的，当规定空间用尽且需要放入新数据的时候，会优先淘汰最久未被访问的数据
- 优点：LRU可以有效的对访问比较频繁的数据进行保护，也就是针对热点数据的命中率提高有明显的效果。
- 缺点：对于周期性、偶发性的访问数据，有大概率可能造成缓存污染，也就是置换出去了热点数据，把这些偶发性数据留下了，从而导致LRU的数据命中率急剧下降。
  下图展示了LRU简单的工作过程，访问时对数据的提前操作，以及数据满且添加新数据的时候淘汰的过程的展示如下：
  [![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914183905435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200914183905435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)
  此处介绍的LRU是有明显的缺点，如上所述，对于偶发性、周期性的数据没有良好的抵抗力，很容易就造成缓存的污染，影响命中率，因此衍生出了很多的LRU算法的变种，用以处理这种偶发冷数据突增的场景，比如：LRU-K、Two Queues等，目的就是当判别数据为偶发或周期的冷数据时，不会存入空间内，从而降低热数据的淘汰率。

下图展示了LRU-K的简单工作过程，简单理解，LRU中的K是指数据被访问K次，传统LRU与此对比则可以认为传统LRU是LRU-1。可以看到LRU-K有两个队列，新来的元素先进入到历史访问队列中，该队列用于记录元素的访问次数，采用的淘汰策略是LRU或者FIFO，当历史队列中的元素访问次数达到K的时候，才会进入缓存队列。
[![LRU-K](https://img-blog.csdnimg.cn/20200914201833261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200914201833261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)
下图展示了Two Queues的工作过程，与LRU-K相比，他也同样是两个队列，不同之处在于，他的队列一个是缓存队列，一个是FIFO队列，当新元素进来的时候，首先进入FIFO队列，当该队列中的元素被访问的时候，会进入LRU队列，过程如下：
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916093048357.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200916093048357.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)

## 3 LFU

LFU（The Least Frequently Used）最近很少使用算法，与LRU的区别在于LRU是以时间衡量，LFU是以时间段内的次数

- 算法：如果一个数据在一定时间内被访问的次数很低，那么被认为在未来被访问的概率也是最低的，当规定空间用尽且需要放入新数据的时候，会优先淘汰时间段内访问次数最低的数据
- 优点：LFU也可以有效的保护缓存，相对场景来讲，比LRU有更好的缓存命中率。因为是以次数为基准，所以更加准确，自然能有效的保证和提高命中率
- 缺点：因为LFU需要记录数据的访问频率，因此需要额外的空间；当访问模式改变的时候，算法命中率会急剧下降，这也是他最大弊端。

下面描述了LFU的简单工作过程，首先是访问元素增加元素的访问次数，从而提高元素在队列中的位置，降低淘汰优先级，后面是插入新元素的时候，因为队列已经满了，所以优先淘汰在一定时间间隔内访问频率最低的元素
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916100317414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200916100317414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)

## 4 W-TinyLFU

W-TinyLFU（Window Tiny Least Frequently Used）是对LFU的的优化和加强。

- 算法：当一个数据进来的时候，会进行筛选比较，进入W-LRU窗口队列，以此应对流量突增，经过淘汰后进入过滤器，通过访问访问频率判决是否进入缓存。如果一个数据最近被访问的次数很低，那么被认为在未来被访问的概率也是最低的，当规定空间用尽的时候，会优先淘汰最近访问次数很低的数据；
- 优点：使用Count-Min Sketch算法存储访问频率，极大的节省空间；定期衰减操作，应对访问模式变化；并且使用window-lru机制能够尽可能避免缓存污染的发生，在过滤器内部会进行筛选处理，避免低频数据置换高频数据。
- 缺点：是由谷歌工程师发明的一种算法，目前已知应用于Caffeine Cache组件里，应用不是很多。

关于Count-Min Sketch算法，可以看作是布隆过滤器的同源的算法，假如我们用一个hashmap来存储每个元素的访问次数，那这个量级是比较大的，并且hash冲突的时候需要做一定处理，否则数据会产生很大的误差，Count-Min Sketch算法将一个hash操作，扩增为多个hash，这样原来hash冲突的概率就降低了几个等级，且当多个hash取得数据的时候，取最低值，也就是Count Min的含义所在。

下图展示了Count-Min Sketch算法简单的工作原理：

1. 假设有四个hash函数，每当元素被访问时，将进行次数加1；
2. 此时会按照约定好的四个hash函数进行hash计算找到对应的位置，相应的位置进行+1操作；
3. 当获取元素的频率时，同样根据hash计算找到4个索引位置；
4. 取得四个位置的频率信息，然后根据Count Min取得最低值作为本次元素的频率值返回，即Min(Count);
   [![在这里插入图片描述](https://img-blog.csdnimg.cn/20200919112318270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200919112318270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xfZG9uZ3lhbmc=,size_16,color_FFFFFF,t_70#pic_center)

# 入门级使用
