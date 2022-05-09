# Redis客户端

# 一、开篇

Redis作为目前通用的缓存选型，因其高性能而倍受欢迎。Redis的2.x版本仅支持单机模式，从3.0版本开始引入集群模式。

Redis的Java生态的客户端当中包含Jedis、Redisson、Lettuce，不同的客户端具备不同的能力是使用方式，本文主要分析Jedis客户端。

Jedis客户端同时支持单机模式、分片模式、集群模式的访问模式，通过构建Jedis类对象实现单机模式下的数据访问，通过构建ShardedJedis类对象实现分片模式的数据访问，通过构建JedisCluster类对象实现集群模式下的数据访问。

Jedis客户端支持单命令和Pipeline方式访问Redis集群，通过Pipeline的方式能够提高集群访问的效率。

本文的整体分析基于Jedis的3.5.0版本进行分析，相关源码均参考此版本。

# 二、Jedis访问模式对比

Jedis客户端操作Redis主要分为三种模式，分表是单机模式、分片模式、集群模式。

- 单机模式主要是创建Jedis对象来操作单节点的Redis，只适用于访问单个Redis节点。
- 分片模式（ShardedJedis）主要是通过创建ShardedJedisPool对象来访问分片模式的多个Redis节点，是Redis没有集群功能之前客户端实现的一个数据分布式方案，本质上是客户端通过一致性哈希来实现数据分布式存储。
- 集群模式（JedisCluster）主要是通过创建JedisCluster对象来访问集群模式下的多个Redis节点，是Redis3.0引入集群模式后客户端实现的集群访问访问，本质上是通过引入槽（slot）概念以及通过CRC16哈希槽算法来实现数据分布式存储。

单机模式不涉及任何分片的思想，所以我们着重分析分片模式和集群模式的理念。

## 2.1 分片模式

- 分片模式本质属于基于客户端的分片，在客户端实现如何根据一个key找到Redis集群中对应的节点的方案。
- Jedis的客户端分片模式采用一致性Hash来实现，一致性Hash算法的好处是当Redis节点进行增减时只会影响新增或删除节点前后的小部分数据，相对于取模等算法来说对数据的影响范围较小。
- Redis在大部分场景下作为缓存进行使用，所以不用考虑数据丢失致使缓存穿透造成的影响，在Redis节点增减时可以不用考虑部分数据无法命中的问题。

分片模式的整体应用如下图所示，核心在于客户端的一致性Hash策略。

![img](https://static001.geekbang.org/infoq/5d/5dcf4c77ac29860e017be8cb03147463.png)

（引用自：www.cnblogs.com）

## 2.2 集群模式

集群模式本质属于服务器分片技术，由Redis集群本身提供分片功能，从Redis 3.0版本开始正式提供。

集群的原理是：一个 Redis 集群包含16384 个哈希槽（Hash slot）， Redis保存的每个键都属于这16384个哈希槽的其中一个， 集群使用公式CRC16(key)%16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键key的CRC16校验和 。

集群中的每个节点负责处理一部分哈希槽。举个例子， 一个集群可以有三个哈希槽， 其中：

- 节点 A 负责处理 0 号至 5500 号哈希槽。
- 节点 B 负责处理 5501 号至 11000 号哈希槽。
- 节点 C 负责处理 11001 号至 16383 号哈希槽。

Redis在集群模式下对于key的读写过程首先将对应的key值进行CRC16计算得到对应的哈希值，将哈希值对槽位总数取模映射到对应的槽位，最终映射到对应的节点进行读写。以命令set("key", "value")为例子，它会使用CRC16算法对key进行计算得到哈希值28989，然后对16384进行取模得到12605，最后找到12605对应的Redis节点，最终跳转到该节点执行set命令。

集群模式的整体应用如下图所示，核心在于集群哈希槽的设计以及重定向命令。

![img](https://static001.geekbang.org/infoq/43/434f5f5dcab5cb818005c160ea41f67f.png)

（引用自：www.jianshu.com）

# 三、Jedis的基础用法

```java
// Jedis单机模式的访问
public void main(String[] args) {
    // 创建Jedis对象
    jedis = new Jedis("localhost", 6379);
    // 执行hmget操作
    jedis.hmget("foobar", "foo");
    // 关闭Jedis对象
    jedis.close();
}
 
// Jedis分片模式的访问
public void main(String[] args) {
    HostAndPort redis1 = HostAndPortUtil.getRedisServers().get(0);
    HostAndPort redis2 = HostAndPortUtil.getRedisServers().get(1);
    List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>(2);
    JedisShardInfo shard1 = new JedisShardInfo(redis1);
    JedisShardInfo shard2 = new JedisShardInfo(redis2);
    // 创建ShardedJedis对象
    ShardedJedis shardedJedis = new ShardedJedis(shards);
    // 通过ShardedJedis对象执行set操作
    shardedJedis.set("a", "bar");
}
 
// Jedis集群模式的访问
public void main(String[] args) {
    // 构建redis的集群池
    Set<HostAndPort> nodes = new HashSet<>();
    nodes.add(new HostAndPort("127.0.0.1", 7001));
    nodes.add(new HostAndPort("127.0.0.1", 7002));
    nodes.add(new HostAndPort("127.0.0.1", 7003));
 
    // 创建JedisCluster
    JedisCluster cluster = new JedisCluster(nodes);
 
    // 执行JedisCluster对象中的方法
    cluster.set("cluster-test", "my jedis cluster test");
    String result = cluster.get("cluster-test");
}
```

Jedis通过创建Jedis的类对象来实现单机模式下的数据访问，通过构建JedisCluster类对象来实现集群模式下的数据访问。

要理解Jedis的访问Redis的整个过程，可以通过先理解单机模式下的访问流程，在这个基础上再分析集群模式的访问流程会比较合适。

# 四、Jedis单机模式的访问

Jedis访问单机模式Redis的整体流程图如下所示，从图中可以看出核心的流程包含Jedis对象的创建以及通过Jedis对象实现Redis的访问。

熟悉Jedis访问单机Redis的过程，本身就是需要了解Jedis的创建过程以及执行Redis命令的过程。

- Jedis的创建过程核心在于创建Jedis对象以及Jedis内部变量Client对象。
- Jedis访问Redis的过程在于通过Jedis内部的Client对象访问Redis。

![img](https://static001.geekbang.org/infoq/b6/b6e483a2e5a65c19b6d690fa687f2a4f.png)

## 4.1 创建过程

Jedis本身的类关系图如下图所示，从图中我们能够看到Jedis继承自BinaryJedis类。

在BinaryJedis类中存在和Redis对接的Client类对象，Jedis通过父类的BinaryJedis的Client对象实现Redis的读写。

![img](https://static001.geekbang.org/infoq/76/76df582f46185a45ddc7328720cc9ce0.png)

Jedis类在创建过程中通过父类BinaryJedis创建了Client对象，而了解Client对象是进一步理解访问过程的关键。

```java
public class Jedis extends BinaryJedis implements JedisCommands, MultiKeyCommands,
    AdvancedJedisCommands, ScriptingCommands, BasicCommands, ClusterCommands, SentinelCommands, ModuleCommands {
 
  protected JedisPoolAbstract dataSource = null;
 
  public Jedis(final String host, final int port) {
    // 创建父类BinaryJedis对象
    super(host, port);
  }
}
 
public class BinaryJedis implements BasicCommands, BinaryJedisCommands, MultiKeyBinaryCommands,
    AdvancedBinaryJedisCommands, BinaryScriptingCommands, Closeable {
 
  // 访问redis的Client对象
  protected Client client = null;
 
  public BinaryJedis(final String host, final int port) {
    // 创建Client对象访问redis
    client = new Client(host, port);
  }
}
```

Client类的类关系图如下图所示，Client对象继承自BinaryClient和Connection类。在BinaryClient类中存在Redis访问密码等相关参数，在Connection类在存在访问Redis的socket对象以及对应的输入输出流。本质上Connection是和Redis进行通信的核心类。

![img](https://static001.geekbang.org/infoq/2d/2dcc1483ec904094b6705a2ae2e1c055.png)

Client类在创建过程中初始化核心父类Connection对象，而Connection是负责和Redis直接进行通信。

```java
public class Client extends BinaryClient implements Commands {
  public Client(final String host, final int port) {
    super(host, port);
  }
}
 
public class BinaryClient extends Connection {
  // 存储和Redis连接的相关信息
  private boolean isInMulti;
  private String user;
  private String password;
  private int db;
  private boolean isInWatch;
 
  public BinaryClient(final String host, final int port) {
    super(host, port);
  }
}
 
public class Connection implements Closeable {
  // 管理和Redis连接的socket信息及对应的输入输出流
  private JedisSocketFactory jedisSocketFactory;
  private Socket socket;
  private RedisOutputStream outputStream;
  private RedisInputStream inputStream;
  private int infiniteSoTimeout = 0;
  private boolean broken = false;
 
  public Connection(final String host, final int port, final boolean ssl,
      SSLSocketFactory sslSocketFactory, SSLParameters sslParameters,
      HostnameVerifier hostnameVerifier) {
    // 构建DefaultJedisSocketFactory来创建和Redis连接的Socket对象
    this(new DefaultJedisSocketFactory(host, port, Protocol.DEFAULT_TIMEOUT,
        Protocol.DEFAULT_TIMEOUT, ssl, sslSocketFactory, sslParameters, hostnameVerifier));
  }
}
```

## 4.2 访问过程

以Jedis执行set命令为例，整个过程如下：

- Jedis的set操作是通过Client的set操作来实现的。
- Client的set操作是通过父类Connection的sendCommand来实现。

```java
public class Jedis extends BinaryJedis implements JedisCommands, MultiKeyCommands,
    AdvancedJedisCommands, ScriptingCommands, BasicCommands, ClusterCommands, SentinelCommands, ModuleCommands {
  @Override
  public String set(final String key, final String value) {
    checkIsInMultiOrPipeline();
    // client执行set操作
    client.set(key, value);
    return client.getStatusCodeReply();
  }
}
 
public class Client extends BinaryClient implements Commands {
  @Override
  public void set(final String key, final String value) {
    // 执行set命令
    set(SafeEncoder.encode(key), SafeEncoder.encode(value));
  }
}
 
public class BinaryClient extends Connection {
  public void set(final byte[] key, final byte[] value) {
    // 发送set指令
    sendCommand(SET, key, value);
  }
}
 
public class Connection implements Closeable {
  public void sendCommand(final ProtocolCommand cmd, final byte[]... args) {
    try {
      // socket连接redis
      connect();
      // 按照redis的协议发送命令
      Protocol.sendCommand(outputStream, cmd, args);
    } catch (JedisConnectionException ex) {
    }
  }
}
```

# 五、Jedis分片模式的访问

基于前面已经介绍的Redis分片模式的一致性Hash的原理来理解Jedis的分片模式的访问。

关于Redis分片模式的概念：Redis在3.0版本之前没有集群模式的概念，这导致单节点能够存储的数据有限，通过Redis的客户端如Jedis在客户端通过一致性Hash算法来实现数据的分片存储。

本质上Redis的分片模式跟Redis本身没有任何关系，只是通过客户端来解决单节点数据有限存储的问题。

ShardedJedis访问Redis的核心在于构建对象的时候初始化一致性Hash对象，构建一致性Hash经典的Hash值和node的映射关系。构建完映射关系后执行set等操作就是Hash值到node的寻址过程，寻址完成后直接进行单节点的操作。

![img](https://static001.geekbang.org/infoq/ee/ee035e7078f1af9d5a914bad746f9dff.png)

## 5.1 创建过程

ShardedJedis的创建过程在于父类的Sharded中关于一致性Hash相关的初始化过程，核心在于构建一致性的虚拟节点以及虚拟节点和Redis节点的映射关系。

源码中最核心的部分代码在于根据根据权重映射成未160个虚拟节点，通过虚拟节点来定位到具体的Redis节点。

```java
public class Sharded<R, S extends ShardInfo<R>> {
 
  public static final int DEFAULT_WEIGHT = 1;
  // 保存虚拟节点和redis的node节点的映射关系
  private TreeMap<Long, S> nodes;
  // hash算法
  private final Hashing algo;
  // 保存redis节点和访问该节点的Jedis的连接信息
  private final Map<ShardInfo<R>, R> resources = new LinkedHashMap<>();
 
  public Sharded(List<S> shards, Hashing algo) {
    this.algo = algo;
    initialize(shards);
  }
 
  private void initialize(List<S> shards) {
    nodes = new TreeMap<>();
    // 遍历每个redis的节点并设置hash值到节点的映射关系
    for (int i = 0; i != shards.size(); ++i) {
      final S shardInfo = shards.get(i);
      // 根据权重映射成未160个虚拟节点
      int N =  160 * shardInfo.getWeight();
      if (shardInfo.getName() == null) for (int n = 0; n < N; n++) {
        // 构建hash值和节点映射关系
        nodes.put(this.algo.hash("SHARD-" + i + "-NODE-" + n), shardInfo);
      }
      else for (int n = 0; n < N; n++) {
        nodes.put(this.algo.hash(shardInfo.getName() + "*" + n), shardInfo);
      }
      // 保存每个节点的访问对象
      resources.put(shardInfo, shardInfo.createResource());
    }
  }
}
```

## 5.2 访问过程

ShardedJedis的访问过程就是一致性Hash的计算过程，核心的逻辑就是：通过Hash算法对访问的key进行Hash计算生成Hash值，根据Hash值获取对应Redis节点，根据对应的Redis节点获取对应的访问对象Jedis。

获取访问对象Jedis之后就可以直接进行命令操作。

```java
public class Sharded<R, S extends ShardInfo<R>> {
 
  public static final int DEFAULT_WEIGHT = 1;
  private TreeMap<Long, S> nodes;
  private final Hashing algo;
  // 保存redis节点和访问该节点的Jedis的连接信息
  private final Map<ShardInfo<R>, R> resources = new LinkedHashMap<>();
 
  public R getShard(String key) {
    // 根据redis节点找到对应的访问对象Jedis
    return resources.get(getShardInfo(key));
  }
 
  public S getShardInfo(String key) {
    return getShardInfo(SafeEncoder.encode(getKeyTag(key)));
  }
 
  public S getShardInfo(byte[] key) {
    // 针对访问的key生成对应的hash值
    // 根据hash值找到对应的redis节点
    SortedMap<Long, S> tail = nodes.tailMap(algo.hash(key));
    if (tail.isEmpty()) {
      return nodes.get(nodes.firstKey());
    }
    return tail.get(tail.firstKey());
  }
}
```

# 六、Jedis集群模式的访问

基于前面介绍的Redis的集群原理来理解Jedis的集群模式的访问。

Jedis能够实现key和哈希槽的定位的核心机制在于哈希槽和Redis节点的映射，而这个发现过程基于Redis的cluster slot命令。

关于Redis集群操作的命令：Redis通过cluster slots会返回Redis集群的整体状况。返回每一个Redis节点的信息包含：

- 哈希槽起始编号
- 哈希槽结束编号
- 哈希槽对应master节点，节点使用IP/Port表示
- master节点的第一个副本
- master节点的第二个副本

```java
127.0.0.1:30001> cluster slots
1) 1) (integer) 0 // 开始槽位
   2) (integer) 5460 // 结束槽位
   3) 1) "127.0.0.1" // master节点的host
      2) (integer) 30001 // master节点的port
      3) "09dbe9720cda62f7865eabc5fd8857c5d2678366" // 节点的编码
   4) 1) "127.0.0.1" // slave节点的host
      2) (integer) 30004 // slave节点的port
      3) "821d8ca00d7ccf931ed3ffc7e3db0599d2271abf" // 节点的编码
2) 1) (integer) 5461
   2) (integer) 10922
   3) 1) "127.0.0.1"
      2) (integer) 30002
      3) "c9d93d9f2c0c524ff34cc11838c2003d8c29e013"
   4) 1) "127.0.0.1"
      2) (integer) 30005
      3) "faadb3eb99009de4ab72ad6b6ed87634c7ee410f"
3) 1) (integer) 10923
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 30003
      3) "044ec91f325b7595e76dbcb18cc688b6a5b434a1"
   4) 1) "127.0.0.1"
      2) (integer) 30006
      3) "58e6e48d41228013e5d9c1c37c5060693925e97e"
```

Jedis访问集群模式Redis的整体流程图如下所示，从图中可以看出核心的流程包含JedisCluster对象的创建以及通过JedisCluster对象实现Redis的访问。

JedisCluster对象的创建核心在于创建JedisClusterInfoCache对象并通过集群发现来建立slot和集群节点的映射关系。

JedisCluster对Redis集群的访问在于获取key所在的Redis节点并通过Jedis对象进行访问。

![img](https://static001.geekbang.org/infoq/7f/7f1a84c2a2b6f13b741b4181cc166dfa.png)

## 6.1 创建过程

JedisCluster的类关系如下图所示，在图中可以看到核心变量JedisSlotBasedConnectionHandler对象。

![img](https://static001.geekbang.org/infoq/36/3615c1ba79b935d7a1bd2b7b15ef79d0.png)

JedisCluster的父类BinaryJedisCluster创建了JedisSlotBasedConnectionHandler对象，该对象负责和Redis的集群进行通信。

```java
public class JedisCluster extends BinaryJedisCluster implements JedisClusterCommands,
    MultiKeyJedisClusterCommands, JedisClusterScriptingCommands {
  public JedisCluster(Set<HostAndPort> jedisClusterNode, int connectionTimeout, int soTimeout,
      int maxAttempts, String password, String clientName, final GenericObjectPoolConfig poolConfig,
      boolean ssl, SSLSocketFactory sslSocketFactory, SSLParameters sslParameters,
      HostnameVerifier hostnameVerifier, JedisClusterHostAndPortMap hostAndPortMap) {
 
    // 访问父类BinaryJedisCluster
    super(jedisClusterNode, connectionTimeout, soTimeout, maxAttempts, password, clientName, poolConfig,
        ssl, sslSocketFactory, sslParameters, hostnameVerifier, hostAndPortMap);
  }
}
 
public class BinaryJedisCluster implements BinaryJedisClusterCommands,
    MultiKeyBinaryJedisClusterCommands, JedisClusterBinaryScriptingCommands, Closeable {
  public BinaryJedisCluster(Set<HostAndPort> jedisClusterNode, int connectionTimeout, int soTimeout,
      int maxAttempts, String user, String password, String clientName, GenericObjectPoolConfig poolConfig,
      boolean ssl, SSLSocketFactory sslSocketFactory, SSLParameters sslParameters,
      HostnameVerifier hostnameVerifier, JedisClusterHostAndPortMap hostAndPortMap) {
 
    // 创建JedisSlotBasedConnectionHandler对象
    this.connectionHandler = new JedisSlotBasedConnectionHandler(jedisClusterNode, poolConfig,
        connectionTimeout, soTimeout, user, password, clientName, ssl, sslSocketFactory, sslParameters, hostnameVerifier, hostAndPortMap);
 
    this.maxAttempts = maxAttempts;
  }
}
```

JedisSlotBasedConnectionHandler的核心在于创建并初始化JedisClusterInfoCache对象，该对象缓存了Redis集群的信息。

JedisClusterInfoCache对象的初始化过程通过initializeSlotsCache来完成，主要目的用于实现集群节点和槽位发现。

```java
public class JedisSlotBasedConnectionHandler extends JedisClusterConnectionHandler {
  public JedisSlotBasedConnectionHandler(Set<HostAndPort> nodes, GenericObjectPoolConfig poolConfig,
      int connectionTimeout, int soTimeout, String user, String password, String clientName,
      boolean ssl, SSLSocketFactory sslSocketFactory, SSLParameters sslParameters,
      HostnameVerifier hostnameVerifier, JedisClusterHostAndPortMap portMap) {
 
    super(nodes, poolConfig, connectionTimeout, soTimeout, user, password, clientName,
        ssl, sslSocketFactory, sslParameters, hostnameVerifier, portMap);
  }
}
 
public abstract class JedisClusterConnectionHandler implements Closeable {
  public JedisClusterConnectionHandler(Set<HostAndPort> nodes, final GenericObjectPoolConfig poolConfig,
      int connectionTimeout, int soTimeout, int infiniteSoTimeout, String user, String password, String clientName,
      boolean ssl, SSLSocketFactory sslSocketFactory, SSLParameters sslParameters,
      HostnameVerifier hostnameVerifier, JedisClusterHostAndPortMap portMap) {
 
    // 创建JedisClusterInfoCache对象
    this.cache = new JedisClusterInfoCache(poolConfig, connectionTimeout, soTimeout, infiniteSoTimeout,
        user, password, clientName, ssl, sslSocketFactory, sslParameters, hostnameVerifier, portMap);
 
    // 初始化jedis的Slot信息
    initializeSlotsCache(nodes, connectionTimeout, soTimeout, infiniteSoTimeout,
        user, password, clientName, ssl, sslSocketFactory, sslParameters, hostnameVerifier);
  }
 
 
  private void initializeSlotsCache(Set<HostAndPort> startNodes,
      int connectionTimeout, int soTimeout, int infiniteSoTimeout, String user, String password, String clientName,
      boolean ssl, SSLSocketFactory sslSocketFactory, SSLParameters sslParameters, HostnameVerifier hostnameVerifier) {
    for (HostAndPort hostAndPort : startNodes) {
 
      try (Jedis jedis = new Jedis(hostAndPort.getHost(), hostAndPort.getPort(), connectionTimeout,
          soTimeout, infiniteSoTimeout, ssl, sslSocketFactory, sslParameters, hostnameVerifier)) {
 
        // 通过discoverClusterNodesAndSlots进行集群发现
        cache.discoverClusterNodesAndSlots(jedis);
        return;
      } catch (JedisConnectionException e) {
      }
    }
  }
}
```

JedisClusterInfoCache的nodes用来保存Redis集群的节点信息，slots用来保存槽位和集群节点的信息。

nodes和slots维持的对象都是JedisPool对象，该对象维持了和Redis的连接信息。集群的发现过程由discoverClusterNodesAndSlots来实现，本质是执行Redis的集群发现命令cluster slots实现的。

```java
public class JedisClusterInfoCache {
  // 负责保存redis集群的节点信息
  private final Map<String, JedisPool> nodes = new HashMap<>();
  // 负责保存redis的槽位和redis节点的映射关系
  private final Map<Integer, JedisPool> slots = new HashMap<>();
 
  // 负责集群的发现逻辑
  public void discoverClusterNodesAndSlots(Jedis jedis) {
    w.lock();
 
    try {
      reset();
      List<Object> slots = jedis.clusterSlots();
 
      for (Object slotInfoObj : slots) {
        List<Object> slotInfo = (List<Object>) slotInfoObj;
 
        if (slotInfo.size() <= MASTER_NODE_INDEX) {
          continue;
        }
        // 获取redis节点对应的槽位信息
        List<Integer> slotNums = getAssignedSlotArray(slotInfo);
 
        // hostInfos
        int size = slotInfo.size();
        for (int i = MASTER_NODE_INDEX; i < size; i++) {
          List<Object> hostInfos = (List<Object>) slotInfo.get(i);
          if (hostInfos.isEmpty()) {
            continue;
          }
 
          HostAndPort targetNode = generateHostAndPort(hostInfos);
          // 负责保存redis节点信息
          setupNodeIfNotExist(targetNode);
          if (i == MASTER_NODE_INDEX) {
            // 负责保存槽位和redis节点的映射关系
            assignSlotsToNode(slotNums, targetNode);
          }
        }
      }
    } finally {
      w.unlock();
    }
  }
 
  public void assignSlotsToNode(List<Integer> targetSlots, HostAndPort targetNode) {
    w.lock();
    try {
      JedisPool targetPool = setupNodeIfNotExist(targetNode);
      // 保存槽位和对应的JedisPool对象
      for (Integer slot : targetSlots) {
        slots.put(slot, targetPool);
      }
    } finally {
      w.unlock();
    }
  }
 
  public JedisPool setupNodeIfNotExist(HostAndPort node) {
    w.lock();
    try {
      // 生产redis节点对应的nodeKey
      String nodeKey = getNodeKey(node);
      JedisPool existingPool = nodes.get(nodeKey);
      if (existingPool != null) return existingPool;
      // 生产redis节点对应的JedisPool
      JedisPool nodePool = new JedisPool(poolConfig, node.getHost(), node.getPort(),
          connectionTimeout, soTimeout, infiniteSoTimeout, user, password, 0, clientName,
          ssl, sslSocketFactory, sslParameters, hostnameVerifier);
      // 保存redis节点的key和对应的JedisPool对象
      nodes.put(nodeKey, nodePool);
      return nodePool;
    } finally {
      w.unlock();
    }
  }
}
```

JedisPool的类关系如下图所示，其中内部internalPool是通过apache common pool来实现的池化。

![img](https://static001.geekbang.org/infoq/86/869069efd6df314dc2aab0b6a6b461bf.png)

JedisPool内部的internalPool通过JedisFactory的makeObject来创建Jedis对象。

每个Redis节点都会对应一个JedisPool对象，通过JedisPool来管理Jedis的申请释放复用等。

```java
public class JedisPool extends JedisPoolAbstract {
 
  public JedisPool() {
    this(Protocol.DEFAULT_HOST, Protocol.DEFAULT_PORT);
  }
}
 
public class JedisPoolAbstract extends Pool<Jedis> {
 
  public JedisPoolAbstract() {
    super();
  }
}
 
public abstract class Pool<T> implements Closeable {
  protected GenericObjectPool<T> internalPool;
 
  public void initPool(final GenericObjectPoolConfig poolConfig, PooledObjectFactory<T> factory) {
    if (this.internalPool != null) {
      try {
        closeInternalPool();
      } catch (Exception e) {
      }
    }
    this.internalPool = new GenericObjectPool<>(factory, poolConfig);
  }
}
 
class JedisFactory implements PooledObjectFactory<Jedis> {
   
  @Override
  public PooledObject<Jedis> makeObject() throws Exception {
    // 创建Jedis对象
    final HostAndPort hp = this.hostAndPort.get();
    final Jedis jedis = new Jedis(hp.getHost(), hp.getPort(), connectionTimeout, soTimeout,
        infiniteSoTimeout, ssl, sslSocketFactory, sslParameters, hostnameVerifier);
 
    try {
      // Jedis对象连接
      jedis.connect();
      if (user != null) {
        jedis.auth(user, password);
      } else if (password != null) {
        jedis.auth(password);
      }
      if (database != 0) {
        jedis.select(database);
      }
      if (clientName != null) {
        jedis.clientSetname(clientName);
      }
    } catch (JedisException je) {
      jedis.close();
      throw je;
    }
    // 将Jedis对象包装成DefaultPooledObject进行返回
    return new DefaultPooledObject<>(jedis);
  }
}
```

## 6.2 访问过程

JedisCluster访问Redis的过程通过JedisClusterCommand来实现重试机制，最终通过Jedis对象来实现访问。从实现的角度来说JedisCluster是在Jedis之上封装了一层，进行集群节点定位以及重试机制等。

以set命令为例，整个访问通过JedisClusterCommand实现如下：

- 计算key所在的Redis节点。
- 获取Redis节点对应的Jedis对象。
- 通过Jedis对象进行set操作。

```java
public class JedisCluster extends BinaryJedisCluster implements JedisClusterCommands,
    MultiKeyJedisClusterCommands, JedisClusterScriptingCommands {
 
  @Override
  public String set(final String key, final String value, final SetParams params) {
    return new JedisClusterCommand<String>(connectionHandler, maxAttempts) {
      @Override
      public String execute(Jedis connection) {
        return connection.set(key, value, params);
      }
    }.run(key);
  }
}
```

JedisClusterCommand的run方法核心主要定位Redis的key所在的Redis节点，然后获取与该节点对应的Jedis对象进行访问。

在Jedis对象访问异常后，JedisClusterCommand会进行重试操作并按照一定策略执行renewSlotCache方法进行重集群节点重发现动作。

```java
public abstract class JedisClusterCommand<T> {
  public T run(String key) {
    // 针对key进行槽位的计算
    return runWithRetries(JedisClusterCRC16.getSlot(key), this.maxAttempts, false, null);
  }
   
  private T runWithRetries(final int slot, int attempts, boolean tryRandomNode, JedisRedirectionException redirect) {
 
    Jedis connection = null;
    try {
 
      if (redirect != null) {
        connection = this.connectionHandler.getConnectionFromNode(redirect.getTargetNode());
        if (redirect instanceof JedisAskDataException) {
          connection.asking();
        }
      } else {
        if (tryRandomNode) {
          connection = connectionHandler.getConnection();
        } else {
          // 根据slot去获取Jedis对象
          connection = connectionHandler.getConnectionFromSlot(slot);
        }
      }
      // 执行真正的Redis的命令
      return execute(connection);
    } catch (JedisNoReachableClusterNodeException jnrcne) {
      throw jnrcne;
    } catch (JedisConnectionException jce) {
 
      releaseConnection(connection);
      connection = null;
 
      if (attempts <= 1) {
        // 保证最后两次机会去重新刷新槽位和节点的对应的信息
        this.connectionHandler.renewSlotCache();
      }
      // 按照重试次数进行重试操作
      return runWithRetries(slot, attempts - 1, tryRandomNode, redirect);
    } catch (JedisRedirectionException jre) {
      // 针对返回Move命令立即触发重新刷新槽位和节点的对应信息
      if (jre instanceof JedisMovedDataException) {
        // it rebuilds cluster's slot cache recommended by Redis cluster specification
        this.connectionHandler.renewSlotCache(connection);
      }
 
      releaseConnection(connection);
      connection = null;
 
      return runWithRetries(slot, attempts - 1, false, jre);
    } finally {
      releaseConnection(connection);
    }
  }
}
```

JedisSlotBasedConnectionHandler的cache对象维持了slot和node的映射关系，通过getConnectionFromSlot方法来获取该slot对应的Jedis对象。

```java
public class JedisSlotBasedConnectionHandler extends JedisClusterConnectionHandler {
 
  protected final JedisClusterInfoCache cache;
 
  @Override
  public Jedis getConnectionFromSlot(int slot) {
    // 获取槽位对应的JedisPool对象
    JedisPool connectionPool = cache.getSlotPool(slot);
    if (connectionPool != null) {
      // 从JedisPool对象中获取Jedis对象
      return connectionPool.getResource();
    } else {
      // 获取失败就重新刷新槽位信息
      renewSlotCache();
      connectionPool = cache.getSlotPool(slot);
      if (connectionPool != null) {
        return connectionPool.getResource();
      } else {
        //no choice, fallback to new connection to random node
        return getConnection();
      }
    }
  }
}
```

# 七、Jedis的Pipeline实现

Pipeline的技术核心思想是将多个命令发送到服务器而不用等待回复，最后在一个步骤中读取该答复。这种模式的好处在于节省了请求响应这种模式的网络开销。

Redis的普通命令如set和Pipeline批量操作的核心的差别在于set命令的操作会直接发送请求到Redis并同步等待结果返回，而Pipeline的操作会发送请求但不立即同步等待结果返回，具体的实现可以从Jedis的源码一探究竟。

原生的Pipeline在集群模式下相关的key必须Hash到同一个节点才能生效，原因在于Pipeline下的Client对象只能其中的一个节点建立了连接。

在集群模式下归属于不同节点的key能够使用Pipeline就需要针对每个key保存对应的节点的client对象，在最后执行获取数据的时候一并获取。本质上可以认为在单节点的Pipeline的基础上封装成一个集群式的Pipeline。

## 7.1 Pipeline用法分析

Pipeline访问单节点的Redis的时候，通过Jedis对象的Pipeline方法返回Pipeline对象，其他的命令操作通过该Pipeline对象进行访问。

Pipeline从使用角度来分析，会批量发送多个命令并最后统一使用syncAndReturnAll来一次性返回结果。

```java
public void pipeline() {
    jedis = new Jedis(hnp.getHost(), hnp.getPort(), 500);
    Pipeline p = jedis.pipelined();
    // 批量发送命令到redis
    p.set("foo", "bar");
    p.get("foo");
    // 同步等待响应结果
    List<Object> results = p.syncAndReturnAll();
 
    assertEquals(2, results.size());
    assertEquals("OK", results.get(0));
    assertEquals("bar", results.get(1));
 }
 
 
public abstract class PipelineBase extends Queable implements BinaryRedisPipeline, RedisPipeline {
 
  @Override
  public Response<String> set(final String key, final String value) {
    // 发送命令
    getClient(key).set(key, value);
    // pipeline的getResponse只是把待响应的请求聚合到pipelinedResponses对象当中
    return getResponse(BuilderFactory.STRING);
  }
}
 
 
public class Queable {
 
  private Queue<Response<?>> pipelinedResponses = new LinkedList<>();
  protected <T> Response<T> getResponse(Builder<T> builder) {
    Response<T> lr = new Response<>(builder);
    // 统一保存到响应队列当中
    pipelinedResponses.add(lr);
    return lr;
  }
}
 
 
public class Pipeline extends MultiKeyPipelineBase implements Closeable {
 
  public List<Object> syncAndReturnAll() {
    if (getPipelinedResponseLength() > 0) {
      // 根据批量发送命令的个数即需要批量返回命令的个数，通过client对象进行批量读取
      List<Object> unformatted = client.getMany(getPipelinedResponseLength());
      List<Object> formatted = new ArrayList<>();
      for (Object o : unformatted) {
        try {
          // 格式化每个返回的结果并最终保存在列表中进行返回
          formatted.add(generateResponse(o).get());
        } catch (JedisDataException e) {
          formatted.add(e);
        }
      }
      return formatted;
    } else {
      return java.util.Collections.<Object> emptyList();
    }
  }
}
```

普通set命令发送请求给Redis后立即通过getStatusCodeReply来获取响应结果，所以这是一种请求响应的模式。

getStatusCodeReply在获取响应结果的时候会通过flush()命令强制发送报文到Redis服务端然后通过读取响应结果。

```java
public class BinaryJedis implements BasicCommands, BinaryJedisCommands, MultiKeyBinaryCommands,
    AdvancedBinaryJedisCommands, BinaryScriptingCommands, Closeable {
 
  @Override
  public String set(final byte[] key, final byte[] value) {
    checkIsInMultiOrPipeline();
    // 发送命令
    client.set(key, value);
    // 等待请求响应
    return client.getStatusCodeReply();
  }
}
 
 
public class Connection implements Closeable {
  public String getStatusCodeReply() {
    // 通过flush立即发送请求
    flush();
    // 处理响应请求
    final byte[] resp = (byte[]) readProtocolWithCheckingBroken();
    if (null == resp) {
      return null;
    } else {
      return SafeEncoder.encode(resp);
    }
  }
}
 
 
public class Connection implements Closeable {
  protected void flush() {
    try {
      // 针对输出流进行flush操作保证报文的发出
      outputStream.flush();
    } catch (IOException ex) {
      broken = true;
      throw new JedisConnectionException(ex);
    }
  }
}
```

# 八、结束语

Jedis作为Redis官方首选的Java客户端开发包，支持绝大部分的Redis的命令，也是日常中使用较多的Redis客户端。

了解了Jedis的实现原理，除了能够支持Redis的日常操作外，还能更好的应对Redis的额外操作诸如扩容时的技术选型。

通过介绍Jedis针对单机模式、分配模式、集群模式三种场景访问方式，让大家有个从宏观到微观的理解过程，掌握Jedis的核心思想并更好的应用到实践当中。

# 一、Lettuce 是啥？

一次技术讨论会上，大家说起 Redis 的 Java 客户端哪家强，我第一时间毫不犹豫地喊出 "Jedis, YES！"

“Jedis 可是官方客户端，用起来直接省事，公司中间件都用它。除了 Jedis 外难道还有第二个能打的？”我直接扔出王炸。

刚学 Spring 的小张听了不服：“SpringDataRedis 都用 RedisTemplate！Jedis？不存在的。”

“坐下吧秀儿，SpringDataRedis 就是基于 Jedis 封装的。”旁边李哥呷了一口刚开的快乐水，嘴角微微上扬，露出一丝不屑。

“现在很多都是用 Lettuce 了，你们不会不知道吧？”老王推了推眼镜淡淡地说道，随即缓缓打开镜片后那双心灵的窗户，用关怀的眼神俯视着我们几只菜鸡。

Lettuce？生菜？满头雾水的我赶紧打开了 Redis 官网的客户端列表。发现 Java 语言有三个官方推荐的实现：**Jedis**、**Lettuce**和 **Redission**。

![img](https://static001.geekbang.org/infoq/60/605f53d8435c87a18bb06abb958513ea.jpeg)

(截图来源：[https://redis.io/clients#java](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Fclients%23java%EF%BC%89))

Lettuce 是什么客户端？没听过。但发现它的官方介绍最长：

> Advanced Redis client for thread-safe sync, async, and reactive usage. Supports Cluster, Sentinel, Pipelining, and codecs.

赶紧查着字典翻译了下：

- 高级客户端
- 线程安全
- 支持同步、异步和反应式 API
- 支持集群、哨兵、管道和编解码

老王摆摆手示意我收好字典，不紧不慢介绍起来。



## 1.1 高级客户端

“师爷，你给翻译翻译，什么（哔——）叫做（哔——）高级客户端？”

“高级客户端嘛，高级嘛，就是 Advanced 啊！new 一下就能用，什么实现细节都不用管，拿起业务逻辑直接突突。”



## 1.2 线程安全

这是和 Jedis 主要不同之一。

Jedis 的连接实例是线程不安全的，于是需要维护一个连接池，每个线程需要时从连接池取出连接实例，完成操作后或者遇到异常归还实例。当连接数随着业务不断上升时，对物理连接的消耗也会成为性能和稳定性的潜在风险点。

Lettuce 使用 Netty 作为通信层组件，其连接实例是线程安全的，并且在条件具备时可访问操作系统原生调用 epoll, kqueue 等获得性能提升。

我们知道 Redis 服务端实例虽然可以同时连接多个客户端收发命令，但每个实例执行命令时都是单线程的。

这意味着如果应用可以通过多线程+单连接方式操作 Redis，将能够精简 Redis 服务端的总连接数，而多应用共享同一个 Redis 服务端时也能够获得更好的稳定性和性能。对于应用来说也减少了维护多个连接实例的资源消耗。



## 1.3 支持同步、异步和反应式 API

Lettuce 从一开始就按照非阻塞式 IO 进行设计，是一个纯异步客户端，对异步和反应式 API 的支持都很全面。

即使是同步命令，底层的通信过程仍然是异步模型，只是通过阻塞调用线程来模拟出同步效果而已。



## 1.4 支持集群、哨兵、管道和编解码

“这些特性都是标配，Lettuce 可是高级客户端！高级，懂吗？”老王说到这里兴奋地用手指点着桌面，但似乎不想多做介绍，我默默地记下打算好好学习一番。

（在项目使用过程中，pipeling 机制用起来和 Jedis 相比稍微抽象已点，下文会给出在使用过程中遇到的小坑和解决办法。）



## 1.5 在 Spring 中的使用情况

除了 Redis 官方介绍，我们也可以发现 Spring Data Redis 在升级到 2.0 时，将 Lettuce 升级到了 5.0。其实 Lettuce 早就在 SpringDataRedis 1.6 时就被官方集成了；而 SpringSessionDataRedis 则直接将 Lettuce 作为默认 Redis 客户端，足见其成熟和稳定。

Jedis 广为人知甚至是事实上的标准 Java 客户端（de-facto standard driver），和它推出时间早（1.0.0 版本 2010 年 9 月，Lettuce 1.0.0 是 2011 年 3 月）、API 直接易用、对 Redis 新特性支持最快等特点都密不可分。

但 Lettuce 作为后进，其优势和易用性也获得了 Spring 等社区的青睐。下面会分享我们在项目中集成 Lettuce 时的经验总结，供大家参考。



# 二、Jedis 和 Lettuce 有啥主要区别？

说了这么多，Lettuce 和老牌客户端 Jedis 主要都有哪些区别呢？我们可以看下 Spring Data Redis 帮助文档给出的对比表格：

![img](https://static001.geekbang.org/infoq/64/649d4223259e3f48e86d42c742ac65dc.jpeg)

（截图来源：[https://docs.spring.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.spring.io%2Fspring-data%2Fredis%2Fdocs%2Fcurrent%2Freference%2Fhtml%2F%23redis%3Aconnectors%3Aconnection)）

**注：其中 X 标记的是支持.**

经过比较我们可以发现：

- Jedis 支持的 Lettuce 都支持；
- Jedis 不支持的 Lettuce 也支持！

这么看来 Spring 中越来越多地使用 Lettuce 也就不奇怪了。



# 三、Lettuce 初体验

光说不练假把式，给大家分享我们尝试 Lettuce 时的收获，尤其是批量命令部分花了比较多的时间踩坑，下文详解。



## 3.1 快速开始

如果最简单的例子都令人费解，那这个库肯定流行不起来。Lettuce 的快速开始真的够快：

a. 引入 maven 依赖（其他依赖类似，具体可见文末参考资料）

```java
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>5.3.6.RELEASE</version>
</dependency>
```

b. 填上 Redis 地址，连接、执行、关闭。Perfect！

```java
import io.lettuce.core.*;
 
// Syntax: redis://[password@]host[:port][/databaseNumber]
// Syntax: redis://[username:password@]host[:port][/databaseNumber]
RedisClient redisClient = RedisClient.create("redis://password@localhost:6379/0");
StatefulRedisConnection<String, String> connection = redisClient.connect();
RedisCommands<String, String> syncCommands = connection.sync();
 
syncCommands.set("key", "Hello, Redis!");
 
connection.close();
redisClient.shutdown();
```



## 3.2 支持集群模式吗？支持！

Redis Cluster 是官方提供的 Redis Sharding 方案，大家应该非常熟悉不再多介绍，官方文档可参考 [Redis Cluster 101](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Ftopics%2Fcluster-tutorial)。

Lettuce 连接 Redis 集群对上述客户端代码一行换一下即可：

```java
// Syntax: redis://[password@]host[:port]
// Syntax: redis://[username:password@]host[:port]
RedisClusterClient redisClient = RedisClusterClient.create("redis://password@localhost:7379");
```



## 3.3 支持高可靠吗？支持！

Redis Sentinel 是官方提供的高可靠方案，通过 Sentinel 可以在实例故障时自动切换到从节点继续提供服务，官方文档可参考 [Redis Sentinel Documentation](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Ftopics%2Fsentinel)。

仍然是替换客户端的创建方式就可以了：

```java
// Syntax: redis-sentinel://[password@]host[:port][,host2[:port2]][/databaseNumber]#sentinelMasterId
RedisClient redisClient = RedisClient.create("redis-sentinel://localhost:26379,localhost:26380/0#mymaster");
```



## 3.4 支持集群下的 pipeline 吗？支持！

Jedis 虽然有 pipeline 命令，但不能支持 Redis Cluster。一般都需要自行归并各个 key 所在的 slot 和实例后再批量执行 pipeline。

官网对集群下的 pipeline 支持 PR 截至本文写作时（2021年2月）四年过去了仍然未合入，可见[ Cluster pipelining](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fredis%2Fjedis%2Fpull%2F1455)。

Lettuce 虽然号称支持 pipeling，但并没有直接看到 pipeline 这种 API，这是怎么回事？



### 3.4.1 实现 pipeline

使用 AsyncCommands 和 flushCommands 实现 pipeline，经过阅读官方文档可以知道，Lettuce 的同步、异步命令其实都共享同一个连接实例，底层使用 pipeline 的形式在发送/接收命令。

区别在于：

- connection.sync() 方法获取的同步命令对象，每一个操作都会立刻将命令通过 TCP 连接发送出去；
- connection.async() 获取的异步命令对象，执行操作后得到的是 RedisFuture<?>，在满足一定条件的情况下才批量发送。

由此我们可以通过异步命令+手动批量推送的方式来实现 pipeline，来看[官方示例](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2Fcore%2Frelease%2Freference%2Findex.html%23_pipelining_and_command_flushing)：

```java
StatefulRedisConnection<String, String> connection = client.connect();
RedisAsyncCommands<String, String> commands = connection.async();
 
// disable auto-flushing
commands.setAutoFlushCommands(false);
 
// perform a series of independent calls
List<RedisFuture<?>> futures = Lists.newArrayList();
for (int i = 0; i < iterations; i++) {
futures.add(commands.set("key-" + i, "value-" + i));
futures.add(commands.expire("key-" + i, 3600));
}
 
// write all commands to the transport layer
commands.flushCommands();
 
// synchronization example: Wait until all futures complete
boolean result = LettuceFutures.awaitAll(5, TimeUnit.SECONDS,
futures.toArray(new RedisFuture[futures.size()]));
 
// later
connection.close();
```



## 3.4.2 这么做有没有问题？

乍一看很完美，但其实有暗坑：setAutoFlushCommands(false) **设置后，会发现 sync() 方法调用的同步命令都不返回了！**这是为什么呢？我们再看看官方文档：

> Lettuce is a non-blocking and asynchronous client. It provides a synchronous API to achieve a blocking behavior on a per-Thread basis to create await (synchronize) a command response..... As soon as the first request returns, the first Thread’s program flow continues, while the second request is processed by Redis and comes back at a certain point in time

sync 和 async 在底层实现上都是一样的，只是 sync 通过阻塞调用线程的方式模拟了同步操作。并且 setAutoFlushCommands 通过源码可以发现就是作用在 connection 对象上，于是该操作对 sync 和 async 命令对象都生效。

所以，只要某个线程中设置了 auto flush commands 为 false，就会影响到所有使用该连接实例的其他线程。

```java
/**
* An asynchronous and thread-safe API for a Redis connection.
*
* @param <K> Key type.
* @param <V> Value type.
* @author Will Glozer
* @author Mark Paluch
*/
public abstract class AbstractRedisAsyncCommands<K, V> implements RedisHashAsyncCommands<K, V>, RedisKeyAsyncCommands<K, V>,
RedisStringAsyncCommands<K, V>, RedisListAsyncCommands<K, V>, RedisSetAsyncCommands<K, V>,
RedisSortedSetAsyncCommands<K, V>, RedisScriptingAsyncCommands<K, V>, RedisServerAsyncCommands<K, V>,
RedisHLLAsyncCommands<K, V>, BaseRedisAsyncCommands<K, V>, RedisTransactionalAsyncCommands<K, V>,
RedisGeoAsyncCommands<K, V>, RedisClusterAsyncCommands<K, V> {
    @Override
    public void setAutoFlushCommands(boolean autoFlush) {
        connection.setAutoFlushCommands(autoFlush);
    }
}
```

对应的，如果多个线程调用 async() 获取异步命令集，并在自身业务逻辑完成后调用 flushCommands()，那将会强行 flush 其他线程还在追加的异步命令，原本逻辑上属于整批的命令将被打散成多份发送。

虽然对于结果的正确性不影响，但如果因为线程相互影响打散彼此的命令进行发送，则对性能的提升就会很不稳定。

自然我们会想到：每个批命令创建一个 connection，然后……这不和 Jedis 一样也是靠连接池么？

回想起老王镜片后那穿透灵魂的目光，我打算硬着头皮再挖掘一下。果然，再次认真阅读文档后我发现了另外一个好东西：**Batch Execution**。



### 3.4.3 Batch Execution

既然 flushCommands 会对 connection 产生全局影响，那把 flush 限制在线程级别不就行了？我从文档中找到了示例官方示例。

回想起前文 Lettuce 是高级客户端，看了文档后发现确实高级，只需要定义接口就行了（让人想起 MyBatis 的 Mapper 接口），下面是项目中使用的例子：

```java
/
/**
 * 定义会用到的批量命令
 */
@BatchSize(100)
public interface RedisBatchQuery extends Commands, BatchExecutor {
    RedisFuture<byte[]> get(byte[] key);
    RedisFuture<Set<byte[]>> smembers(byte[] key);
    RedisFuture<List<byte[]>> lrange(byte[] key, long start, long end);
    RedisFuture<Map<byte[], byte[]>> hgetall(byte[] key);
}
```

调用时这样操作：

```java
// 创建客户端
RedisClusterClient client = RedisClusterClient.create(DefaultClientResources.create(), "redis://" + address);
 
// service 中持有 factory 实例，只创建一次。第二个参数表示 key 和 value 使用 byte[] 编解码
RedisCommandFactory factory = new RedisCommandFactory(connect, Arrays.asList(ByteArrayCodec.INSTANCE, ByteArrayCodec.INSTANCE));
 
// 使用的地方，创建一个查询实例代理类调用命令，最后刷入命令
List<RedisFuture<?>> futures = new ArrayList<>();
RedisBatchQuery batchQuery = factory.getCommands(RedisBatchQuery.class);
for (RedisMetaGroup redisMetaGroup : redisMetaGroups) {
    // 业务逻辑，循环调用多个 key 并将结果保存到 futures 结果中
    appendCommand(redisMetaGroup, futures, batchQuery);
}
 
// 异步命令调用完成后执行 flush 批量执行，此时命令才会发送给 Redis 服务端
batchQuery.flush();
```

就是这么简单。

此时批量的控制将在线程粒度上进行，并在调用 flush 或达到 @BatchSize 配置的缓存命令数量时执行批量操作。而对于 connection 实例，不用再设置 auto flush commands，保持默认的 true 即可，对其他线程不造成影响。

> ps：优秀、严谨的你肯定会想到：如果单命令执行耗时长或者谁放了个诸如 BLPOP 的命令的话，肯定会造成影响的，这个话题官方文档也有涉及，可以考虑使用连接池来处理。



## 3.5 还能再给力一点吗？

Lettuce 支持的当然不仅仅是上面所说的简单功能，还有这些也值得一试：



### 3.5.1 读写分离

我们知道 Redis 实例是支持主从部署的，从实例异步地从主实例同步数据，并借助 Redis Sentinel 在主实例故障时进行主从切换。

当应用对数据一致性不敏感、又需要较大吞吐量时，可以考虑主从读写分离方式。Lettuce 可以设置 StatefulRedisClusterConnection 的 readFrom 配置来进行调整：

![img](https://static001.geekbang.org/infoq/e2/e2343b1d805ce6b541972bb1a1342f9f.png)



### 3.5.2 配置自动更新集群拓扑

**当使用 Redis Cluster 时，服务端发生了扩容怎么办？**

Lettuce 早就考虑好了——通过 RedisClusterClient#setOptions 方法传入 ClusterClientOptions 对象即可配置相关参数（全部配置见文末参考链接）。

ClusterClientOptions 中的 topologyRefreshOptions 常见配置如下：

![img](https://static001.geekbang.org/infoq/c0/c078f6ad404735ab3bee59e6ebfd668a.jpeg)



### 3.5.3 连接池

虽然 Lettuce 基于线程安全的单连接实例已经具有非常好的性能，但也不排除有些大型业务需要通过线程池来提升吞吐量。另外对于事务性操作是有必要独占连接的。

Lettuce 基于 Apache Common-pool2 组件提供了连接池的能力（以下是官方提供的 RedisCluster 对应的客户端线程池使用示例）：

```java
RedisClusterClient clusterClient = RedisClusterClient.create(RedisURI.create(host, port));
 
GenericObjectPool<StatefulRedisClusterConnection<String, String>> pool = ConnectionPoolSupport
               .createGenericObjectPool(() -> clusterClient.connect(), new GenericObjectPoolConfig());
 
// execute work
try (StatefulRedisClusterConnection<String, String> connection = pool.borrowObject()) {
    connection.sync().set("key", "value");
    connection.sync().blpop(10, "list");
}
 
// terminating
pool.close();
clusterClient.shutdown();
```

这里需要说明的是：createGenericObjectPool 创建连接池默认设置 wrapConnections 参数为 true。**此时借出的对象 close 方法将通过动态代理的方式重载为归还连接；若设置为 false 则 close 方法会关闭连接。**

Lettuce 也支持异步的连接池（从连接池获取连接为异步操作），详情可参考文末链接。还有很多特性不能一一列举，都可以在官方文档上找到说明和示例，十分值得一读。



# 四、使用总结

Lettuce 相较于Jedis，使用上更加方便快捷，抽象度高。并且通过线程安全的连接降低了系统中的连接数量，提升了系统的稳定性。

对于高级玩家，Lettuce 也提供了很多配置、接口，方便对性能进行优化和实现深度业务定制的场景。

另外不得不说的一点，Lettuce 的官方文档写的非常全面细致，十分难得。社区比较活跃，Commiter 会积极回答各类 issue，这使得很多疑问都可以自助解决。

相比之下，Jedis 的文档、维护更新速度就比较慢了。JedisCluster pipeline 的 [PR](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fredis%2Fjedis%2Fpull%2F1455) 至今（2021年2月）四年过去还未合入。



# 参考资料

**其中两个 GitHub 的 issue 含金量很高，强烈推荐一读！**

1.Lettuce 快速开始：[https://lettuce.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2Fcore%2Frelease%2Freference%2Findex.html%23getting-started)

2.[Redis Java Clients](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fredis.io%2Fclients%23java)

3.Lettuce 官网：[https://lettuce.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2F)

4.SpringDataRedis [参考文档](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.spring.io%2Fspring-data%2Fredis%2Fdocs%2Fcurrent%2Freference%2Fhtml)

5.[Question about pipelining](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Flettuce-io%2Flettuce-core%2Fissues%2F624)

[6.Why is Lettuce the default Redis client used in Spring Session Redis](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fspring-projects%2Fspring-session%2Fissues%2F789)

7.Cluster-specific options：[https://lettuce.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2Fcore%2Frelease%2Freference%2Findex.html%23clientoptions.cluster-specific-options)

8.[Lettuce 连接池](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2Fcore%2Frelease%2Freference%2Findex.html%23_connection_pooling)

9.客户端配置：[https://lettuce.io/core/release](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2Fcore%2Frelease%2Freference%2Findex.html%23clientresources.configuration-settings)

10.SSL配置：[https://lettuce.io](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Flettuce.io%2Fcore%2Frelease%2Freference%2Findex.html%23ssl)