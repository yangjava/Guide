# Caffeine

英['kæfiːn]美['kæfiːn] n.咖啡因

**A high performance caching library for Java**

官网介绍：Java的高性能缓存库

## 概述

Caffeine是一种高性能的缓存库，是基于Java 8的最佳（最优）缓存框架。

Cache（缓存），基于Google Guava，Caffeine提供一个内存缓存，大大改善了设计Guava's cache 和 ConcurrentLinkedHashMap 的体验。

```
 LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
     .maximumSize(10_000)
     .expireAfterWrite(5, TimeUnit.MINUTES)
     .refreshAfterWrite(1, TimeUnit.MINUTES)
     .build(key -> createExpensiveGraph(key));
```

缓存类似于ConcurrentMap，但二者并不完全相同。最基本的区别是，ConcurrentMap保存添加到其中的所有元素，直到显式地删除它们。另一方面，缓存通常配置为自动删除条目，以限制其内存占用。在某些情况下，LoadingCache或AsyncLoadingCache可能很有用，因为它是自动缓存加载的。

### 特性

Caffeine提供了灵活的结构来创建缓存，并且有以下特性：

- 自动加载条目到缓存中，可选异步方式
- 可以基于大小剔除
- 可以设置过期时间，时间可以从上次访问或上次写入开始计算
- 异步刷新
- keys自动包装在弱引用中
- values自动包装在弱引用或软引用中
- 条目剔除通知
- 缓存访问统计

## 实战

Caffeine Cache 的github地址：https://github.com/ben-manes/caffeine

### Maven依赖

```
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.6.2</version>
</dependency>
```

创建一个 Caffeine 缓存（类似一个map）：

```
Cache<String, Object> manualCache = Caffeine.newBuilder()
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .maximumSize(10_000)
        .build();
```

常见用法：

```
public static void main(String... args) throws Exception {
Cache<String, String> cache = Caffeine.newBuilder()
        //5秒没有读写自动删除
        .expireAfterAccess(5, TimeUnit.SECONDS)
        //最大容量1024个，超过会自动清理空间
        .maximumSize(1024)
        .removalListener(((key, value, cause) -> {
            //清理通知 key,value ==> 键值对   cause ==> 清理原因
        }))
        .build();

//添加值
cache.put("张三", "浙江");
//获取值
cache.getIfPresent("张三");
//remove
cache.invalidate("张三");
}
```

### 缓存加载/填充策略(Population)

填充策略是指如何在key不存在的情况下，如何创建一个对象进行返回，主要分为下面四种

Caffeine Cache提供了三种缓存填充策略：手动加载、同步加载和异步加载。

#### Manual(手动加载)

在每次get key的时候指定一个同步的函数，如果key不存在就调用这个函数生成一个值。

```
  Cache<Key, Graph> cache = Caffeine.newBuilder()
      .expireAfterWrite(10, TimeUnit.MINUTES)
      .maximumSize(10_000)
      .build();
  
  //判断是否存在如果不存返回null
  // Lookup an entry, or null if not found
  Graph graph = cache.getIfPresent(key);
  
  //如果一个key不存在，那么会进入指定的函数生成value
  // Lookup and compute an entry if absent, or null if not computable
  graph = cache.get(key, k -> createExpensiveGraph(key));
  
 // Insert or update an entry
 cache.put(key, graph);
 
 //移除一个key
 // Remove an entry
 cache.invalidate(key);
```

Cache接口可以显式地控制检索、更新和删除条目。 

我们可以通过cache.getIfPresent(key) 方法来获取一个key的值，通过cache.put(key, value)方法显示的将数控放入缓存，但是这样子会覆盖缓原来key的数据。更加建议使用cache.get(key，k - > value) 的方式，get 方法将一个参数为 key 的 Function (createExpensiveGraph) 作为参数传入。如果缓存中不存在该键，则调用这个 Function 函数，并将返回值作为该缓存的值插入缓存中。get 方法是以阻塞方式执行调用，即使多个线程同时请求该值也只会调用一次Function方法。这样可以避免与其他线程的写入竞争，这也是为什么使用 get 优于 getIfPresent 的原因。

**注意**：如果调用该方法返回NULL（如上面的 createExpensiveGraph 方法），则cache.get返回null，如果调用该方法抛出异常，则get方法也会抛出异常。

可以使用Cache.asMap() 方法获取ConcurrentMap进而对缓存进行一些更改。

实战代码

```
public static void main(String... args) throws Exception {
        Cache<String, Integer> cache = Caffeine.newBuilder().build();

        Integer age1 = cache.getIfPresent("张三");
        System.out.println(age1);

        //当key不存在时，会立即创建出对象来返回，age2不会为空
        Integer age2 = cache.get("张三", k -> {
            System.out.println("k:" + k);
            return 18;
        });
        System.out.println(age2);
}
// 返回结果
null
k:张三
18
```

#### Loading (自动加载)

构造Cache时候，build方法传入一个CacheLoader实现类。实现load方法，通过key加载value。

```
 LoadingCache<Key, Graph> cache = Caffeine.newBuilder()
     .maximumSize(10_000)
     .expireAfterWrite(10, TimeUnit.MINUTES)
     .build(key -> createExpensiveGraph(key));
 
 // Lookup and compute an entry if absent, or null if not computable
 Graph graph = cache.get(key);
 
 // Lookup and compute entries that are absent
 Map<Key, Graph> graphs = cache.getAll(keys);
```

LoadingCache通过关联一个CacheLoader来构建Cache

通过LoadingCache的getAll方法，可以批量查询 

实战代码

```
public static void main(String... args) throws Exception {

    //此时的类型是 LoadingCache 不是 Cache
    LoadingCache<String, Integer> cache = Caffeine.newBuilder().build(key -> {
        System.out.println("自动填充:" + key);
        return 18;
    });

    Integer age1 = cache.getIfPresent("张三");
    System.out.println(age1);

    // key 不存在时 会根据给定的CacheLoader自动装载进去
    Integer age2 = cache.get("张三");
    System.out.println(age2);
}
// 返回结果
null
自动填充:张三
18
```

#### Asynchronous Manual (异步手动)

AsyncLoadingCache是继承自LoadingCache类的，异步加载使用Executor去调用方法并返回一个CompletableFuture。异步加载缓存使用了响应式编程模型。

如果要以同步方式调用时，应提供CacheLoader。要以异步表示时，应该提供一个AsyncCacheLoader，并返回一个CompletableFuture。

```
 AsyncCache<Key, Graph> cache = Caffeine.newBuilder()
     .expireAfterWrite(10, TimeUnit.MINUTES)
     .maximumSize(10_000)
     .buildAsync();
 
 
 // Lookup and asynchronously compute an entry if absent
 CompletableFuture<Graph> graph = cache.get(key, k -> createExpensiveGraph(key));
```

AsyncCache是另一种Cache，它基于Executor计算条目，并返回一个CompletableFuture。 

实战代码

```
public static void main(String... args) throws Exception {
    AsyncCache<String, Integer> cache = Caffeine.newBuilder().buildAsync();

    //会返回一个 future对象， 调用future对象的get方法会一直卡住直到得到返回，和多线程的submit一样
    CompletableFuture<Integer> ageFuture = cache.get("张三", name -> {
        System.out.println("name:" + name);
        return 18;
    });

    Integer age = ageFuture.get();
    System.out.println("age:" + age);
}
// 返回结果
name:张三
age:18
```

#### Asynchronously Loading (异步加载)

```
  AsyncLoadingCache<Key, Graph> cache = Caffeine.newBuilder()
      .maximumSize(10_000)
      .expireAfterWrite(10, TimeUnit.MINUTES)
      
      // Either: Build with a synchronous computation that is wrapped as asynchronous 
      .buildAsync(key -> createExpensiveGraph(key));
      
      // Or: Build with a asynchronous computation that returns a future
      .buildAsync((key, executor) -> createExpensiveGraphAsync(key, executor));
  
  // Lookup and asynchronously compute an entry if absent
  CompletableFuture<Graph> graph = cache.get(key);
  
 // Lookup and asynchronously compute entries that are absent
 CompletableFuture<Map<Key, Graph>> graphs = cache.getAll(keys);
```

AsyncLoadingCache 是关联了 AsyncCacheLoader 的 AsyncCache

实战代码

```
public static void main(String... args) throws Exception {
   
    AsyncLoadingCache<String, Integer> cache = Caffeine.newBuilder().buildAsync(name -> {
        System.out.println("name:" + name);
        return 18;
    });
    CompletableFuture<Integer> ageFuture = cache.get("张三");

    Integer age = ageFuture.get();
    System.out.println("age:" + age);
}
```

###  回收策略(eviction)

Caffeine提供了三种回收策略：基于大小回收(size-based)，基于时间回收(time-based)，基于引用回收(reference-based)。

#### Size-based(基于大小)

基于大小的回收策略有两种方式：一种是基于缓存大小，一种是基于权重。

```
  // 根据缓存的计数进行驱逐
  // Evict based on the number of entries in the cache
  LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
      .maximumSize(10_000)
      .build(key -> createExpensiveGraph(key));
  
  / 根据缓存的权重来进行驱逐（权重只是用于确定缓存大小，不会用于决定该缓存是否被驱逐）
  // Evict based on the number of vertices in the cache
  LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
      .maximumWeight(10_000)
      .weigher((Key key, Graph graph) -> graph.vertices().size())
     .build(key -> createExpensiveGraph(key));
```

如果缓存的条目数量不应该超过某个值，那么可以使用Caffeine.maximumSize(long)。如果超过这个值，则会剔除很久没有被访问过或者不经常使用的那个条目。

如果，不同的条目有不同的权重值的话，那么你可以用Caffeine.weigher(Weigher)来指定一个权重函数，并且使用Caffeine.maximumWeight(long)来设定最大的权重值。

简单的来说，要么限制缓存条目的数量，要么限制缓存条目的权重值，二者取其一。限制数量很好理解，限制权重的话首先你得提供一个函数来设定每个条目的权重值是多少，然后才能显示最大的权重是多少。

**注意：maximumWeight与maximumSize不可以同时使用。**

#### Time-based(基于时间)

```
  // 基于固定的到期策略进行退出
  // Evict based on a fixed expiration policy
 LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
      .expireAfterAccess(5, TimeUnit.MINUTES)
      .build(key -> createExpensiveGraph(key));
  LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
      .expireAfterWrite(10, TimeUnit.MINUTES)
      .build(key -> createExpensiveGraph(key));
  
  
  // 基于不同的到期策略进行退出
  // Evict based on a varying expiration policy
 LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
     .expireAfter(new Expiry<Key, Graph>() {
       public long expireAfterCreate(Key key, Graph graph, long currentTime) {
         // Use wall clock time, rather than nanotime, if from an external resource
         long seconds = graph.creationDate().plusHours(5)
             .minus(System.currentTimeMillis(), MILLIS)
             .toEpochSecond();
         return TimeUnit.SECONDS.toNanos(seconds);
       }
       public long expireAfterUpdate(Key key, Graph graph, 
           long currentTime, long currentDuration) {
         return currentDuration;
       }
       public long expireAfterRead(Key key, Graph graph,
           long currentTime, long currentDuration) {
         return currentDuration;
       }
     })
     .build(key -> createExpensiveGraph(key));
```

Caffeine提供了三种定时驱逐策略：

- expireAfterAccess(long, TimeUnit): 在最后一次访问或者写入后开始计时，在指定的时间后过期。假如一直有请求访问该key，那么这个缓存将一直不会过期。
- expireAfterWrite(long, TimeUnit): 在最后一次写入缓存后开始计时，在指定的时间后过期。
- expireAfter(Expiry): 自定义策略，过期时间由Expiry实现独自计算。

缓存的删除策略使用的是惰性删除和定时删除。这两个删除策略的时间复杂度都是O(1)。

建议，主动维护缓存中条目，而不是等到访问的时候发现缓存条目已经失效了才去重新加载。意思就是，提前加载，定期维护。

可以在构建的时候Caffeine.scheduler(Scheduler)来指定调度线程

#### Reference-based (基于引用)

Java中四种引用类型

| 引用类型                 | 被垃圾回收时间 | 用途                                                         | 生存时间          |
| ------------------------ | -------------- | ------------------------------------------------------------ | ----------------- |
| 强引用 Strong Reference  | 从来不会       | 对象的一般状态                                               | JVM停止运行时终止 |
| 软引用 Soft Reference    | 在内存不足时   | 对象缓存                                                     | 内存不足时终止    |
| 弱引用 Weak Reference    | 在垃圾回收时   | 对象缓存                                                     | gc运行后终止      |
| 虚引用 Phantom Reference | 从来不会       | 可以用虚引用来跟踪对象被垃圾回收器回收的活动，当一个虚引用关联的对象被垃圾收集器回收之前会收到一条系统通知 | JVM停止运行时终止 |

```
  // 当key和value都没有引用时驱逐缓存
  // Evict when neither the key nor value are strongly reachable
  LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
      .weakKeys()
      .weakValues()
      .build(key -> createExpensiveGraph(key));
  
  // 当垃圾收集器需要释放内存时驱逐
  // Evict when the garbage collector needs to free memory
  LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
      .softValues()
     .build(key -> createExpensiveGraph(key));
```

**注意：AsyncLoadingCache不支持弱引用和软引用。**

Caffeine.weakKeys()： 使用弱引用存储key。如果没有其他地方对该key有强引用，那么该缓存就会被垃圾回收器回收。由于垃圾回收器只依赖于身份(identity)相等，因此这会导致整个缓存使用身份 (==) 相等来比较 key，而不是使用 equals()。

Caffeine.weakValues() ：使用弱引用存储value。如果没有其他地方对该value有强引用，那么该缓存就会被垃圾回收器回收。由于垃圾回收器只依赖于身份(identity)相等，因此这会导致整个缓存使用身份 (==) 相等来比较 key，而不是使用 equals()。

Caffeine.softValues() ：使用软引用存储value。当内存满了过后，软引用的对象以将使用最近最少使用(least-recently-used ) 的方式进行垃圾回收。由于使用软引用是需要等到内存满了才进行回收，所以我们通常建议给缓存配置一个使用内存的最大值。 softValues() 将使用身份相等(identity) (==) 而不是equals() 来比较值。

**注意：Caffeine.weakValues()和Caffeine.softValues()不可以一起使用。**

### 移除 (Removal)

概念：

- 驱逐（eviction）：由于满足了某种驱逐策略，后台自动进行的删除操作
- 无效（invalidation）：表示由调用方手动删除缓存
- 移除（removal）：监听驱逐或无效操作的监听器

#### invalidate(手动删除缓存)

```
 // individual key
 cache.invalidate(key)
 // bulk keys
 cache.invalidateAll(keys)
 // all keys
 cache.invalidateAll()
```

#### removalListener(移除事件监听)

```
 Cache<Key, Graph> graphs = Caffeine.newBuilder()
     .removalListener((Key key, Graph graph, RemovalCause cause) ->
         System.out.printf("Key %s was removed (%s)%n", key, cause))
     .build();
```

您可以通过Caffeine.removalListener(RemovalListener) 为缓存指定一个删除侦听器，以便在删除数据时执行某些操作。 RemovalListener可以获取到key、value和RemovalCause（删除的原因）。

删除侦听器的里面的操作是使用Executor来异步执行的。默认执行程序是ForkJoinPool.commonPool()，可以通过Caffeine.executor(Executor)覆盖。当操作必须与删除同步执行时，请改为使用CacheWrite，CacheWrite将在下面说明。

**注意**：由RemovalListener抛出的任何异常都会被记录（使用Logger）并不会抛出。

###  刷新(Refresh)

```
 LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
     .maximumSize(10_000)
     .refreshAfterWrite(1, TimeUnit.MINUTES)
     .build(key -> createExpensiveGraph(key)); 
```

通过LoadingCache.refresh(K)进行异步刷新，通过覆盖CacheLoader.reload(K, V)可以自定义刷新逻辑

刷新和驱逐是不一样的。刷新的是通过LoadingCache.refresh(key)方法来指定，并通过调用CacheLoader.reload方法来执行，刷新key会异步地为这个key加载新的value，并返回旧的值（如果有的话）。驱逐会阻塞查询操作直到驱逐作完成才会进行其他操作。

与expireAfterWrite不同的是，refreshAfterWrite将在查询数据的时候判断该数据是不是符合查询条件，如果符合条件该缓存就会去执行刷新操作。例如，您可以在同一个缓存中同时指定refreshAfterWrite和expireAfterWrite，只有当数据具备刷新条件的时候才会去刷新数据，不会盲目去执行刷新操作。如果数据在刷新后就一直没有被再次查询，那么该数据也会过期。

刷新操作是使用Executor异步执行的。默认执行程序是ForkJoinPool.commonPool()，可以通过Caffeine.executor(Executor)覆盖。

如果刷新时引发异常，则使用log记录日志，并不会抛出。

###  写入外部存储(Writer)

CacheWriter 方法可以将缓存中所有的数据写入到第三方。

```java
LoadingCache<String, Object> cache2 = Caffeine.newBuilder()
    .writer(new CacheWriter<String, Object>() {
        @Override public void write(String key, Object value) {
            // 写入到外部存储
        }
        @Override public void delete(String key, Object value, RemovalCause cause) {
            // 删除外部存储
        }
    })
    .build(key -> function(key));
```

如果你有多级缓存的情况下，这个方法还是很实用。

CacheWriter允许缓存充当一个底层资源的代理，当与CacheLoader结合使用时，所有对缓存的读写操作都可以通过Writer进行传递。Writer可以把操作缓存和操作外部资源扩展成一个同步的原子性操作。并且在缓存写入完成之前，它将会阻塞后续的更新缓存操作，但是读取（get）将直接返回原有的值。如果写入程序失败，那么原有的key和value的映射将保持不变，如果出现异常将直接抛给调用者。

CacheWriter可以同步的监听到缓存的创建、变更和删除操作。加载（例如，LoadingCache.get）、重新加载（例如，LoadingCache.refresh）和计算（例如Map.computeIfPresent）的操作不被CacheWriter监听到。

**注意：CacheWriter不能与弱键或AsyncLoadingCache一起使用。**

## 可能的用例（Possible Use-Cases）

CacheWriter是复杂工作流的扩展点，需要外部资源来观察给定Key的变化顺序。这些用法Caffeine是支持的，但不是本地内置。

#### 写模式（Write Modes）

CacheWriter可以用来实现一个直接写（write-through ）或回写（write-back ）缓存的操作。

write-through式缓存中，写操作是一个同步的过程，只有写成功了才会去更新缓存。这避免了同时去更新资源和缓存的条件竞争。

write-back式缓存中，对外部资源的操作是在缓存更新后异步执行的。这样可以提高写入的吞吐量，避免数据不一致的风险，比如如果写入失败，则在缓存中保留无效的状态。这种方法可能有助于延迟写操作，直到指定的时间，限制写速率或批写操作。

通过对write-back进行扩展，我们可以实现以下特性：

- 批处理和合并操作
- 延迟操作并到一个特定的时间执行
- 如果超过阈值大小，则在定期刷新之前执行批处理
- 如果操作尚未刷新，则从写入后缓冲器（write-behind）加载
- 根据外部资源的特点，处理重审，速率限制和并发

可以参考一个简单的[例子](https://github.com/ben-manes/caffeine/tree/master/examples/write-behind-rxjava)，使用RxJava实现。

#### 分层（Layering）

CacheWriter可能用来集成多个缓存进而实现多级缓存。

多级缓存的加载和写入可以使用系统外部高速缓存。这允许缓存使用一个小并且快速的缓存去调用一个大的并且速度相对慢一点的缓存。典型的off-heap、file-based和remote 缓存。

受害者缓存（Victim Cache）是一个多级缓存的变体，其中被删除的数据被写入二级缓存。这个delete(K, V, RemovalCause) 方法允许检查为什么该数据被删除，并作出相应的操作。

#### 同步监听器（Synchronous Listeners）

同步监听器会接收一个key在缓存中的进行了那些操作的通知。监听器可以阻止缓存操作，也可以将事件排队以异步的方式执行。这种类型的监听器最常用于复制或构建分布式缓存。

### 统计(Statistics)

```
 Cache<Key, Graph> graphs = Caffeine.newBuilder()
     .maximumSize(10_000)
     .recordStats()
     .build(); 
```

使用Caffeine.recordStats()，你可以打开统计功能。Cache.stats()方法会返回一个CacheStats对象，该对象提供以下统计信息：

- hitRate(): 命中率
- hitCount(): 返回命中缓存的总数
- evictionCount(): 缓存回收数量
- averageLoadPenalty(): 加载新值所花费的平均时间

### Cleanup

缓存的删除策略使用的是惰性删除和定时删除，但是我也可以自己调用cache.cleanUp()方法手动触发一次回收操作。cache.cleanUp()是一个同步方法。

### 策略(Policy)

在创建缓存的时候，缓存的策略就指定好了。但是我们可以在运行时可以获得和修改该策略。这些策略可以通过一些选项来获得，以此来确定缓存是否支持该功能。

#### Size-based

```
cache.policy().eviction().ifPresent(eviction -> {
  eviction.setMaximum(2 * eviction.getMaximum());
});
```

如果缓存配置的时基于权重来驱逐，那么我们可以使用weightedSize() 来获取当前权重。这与获取缓存中的记录数的Cache.estimatedSize() 方法有所不同。

缓存的最大值(maximum)或最大权重(weight)可以通过getMaximum()方法来读取，并使用setMaximum(long)进行调整。当缓存量达到新的阀值的时候缓存才会去驱逐缓存。

如果有需用我们可以通过hottest(int) 和 coldest(int)方法来获取最有可能命中的数据和最有可能驱逐的数据快照。

#### Time-based

```
cache.policy().expireAfterAccess().ifPresent(expiration -> ...);
cache.policy().expireAfterWrite().ifPresent(expiration -> ...);
cache.policy().expireVariably().ifPresent(expiration -> ...);
cache.policy().refreshAfterWrite().ifPresent(expiration -> ...);
```

ageOf(key，TimeUnit) 提供了从expireAfterAccess，expireAfterWrite或refreshAfterWrite策略的角度来看条目已经空闲的时间。最大持续时间可以从getExpiresAfter(TimeUnit)读取，并使用setExpiresAfter(long，TimeUnit)进行调整。

如果有需用我们可以通过hottest(int) 和 coldest(int)方法来获取最有可能命中的数据和最有可能驱逐的数据快照。

#### 测试（Testing）

```
FakeTicker ticker = new FakeTicker(); // Guava's testlib
Cache<Key, Graph> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .executor(Runnable::run)
    .ticker(ticker::read)
    .maximumSize(10)
    .build();

cache.put(key, graph);
ticker.advance(30, TimeUnit.MINUTES)
assertThat(cache.getIfPresent(key), is(nullValue());
```

测试的时候我们可以使用Caffeine..ticker(ticker)来指定一个时间源，并不需要等到key过期。

FakeTicker这个是guawa test包里面的Ticker，主要用于测试。依赖：

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava-testlib</artifactId>
    <version>23.5-jre</version>
</dependency>
```

## 实战

### SpringBoot 中默认Cache-Caffine Cache

SpringBoot 1.x版本中的默认本地cache是Guava Cache。在2.x（**Spring Boot 2.0(spring 5)** ）版本中已经用Caffine Cache取代了Guava Cache。毕竟有了更优的缓存淘汰策略。

下面我们来说在SpringBoot2.x版本中如何使用cache。

#### Maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.6.2</version>
</dependency>
```

#### 添加注解开启缓存支持

添加@EnableCaching注解：

```java
@SpringBootApplication
@EnableCaching
public class SingleDatabaseApplication {

    public static void main(String[] args) {
        SpringApplication.run(SingleDatabaseApplication.class, args);
    }
}
```

#### 配置文件的方式注入相关参数

properties文件

```ini
spring.cache.cache-names=cache1
spring.cache.caffeine.spec=initialCapacity=50,maximumSize=500,expireAfterWrite=10s
```

或Yaml文件

```yaml
spring:
  cache:
    type: caffeine
    cache-names:
    - userCache
    caffeine:
      spec: maximumSize=1024,refreshAfterWrite=60s
```

如果使用refreshAfterWrite配置,必须指定一个CacheLoader.不用该配置则无需这个bean,如上所述,该CacheLoader将关联被该缓存管理器管理的所有缓存，所以必须定义为CacheLoader<Object, Object>，自动配置将忽略所有泛型类型。

```java
import com.github.benmanes.caffeine.cache.CacheLoader;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration
public class CacheConfig {

    /**
     * 相当于在构建LoadingCache对象的时候 build()方法中指定过期之后的加载策略方法
     * 必须要指定这个Bean，refreshAfterWrite=60s属性才生效
     * @return
     */
    @Bean
    public CacheLoader<String, Object> cacheLoader() {
        CacheLoader<String, Object> cacheLoader = new CacheLoader<String, Object>() {
            @Override
            public Object load(String key) throws Exception {
                return null;
            }
            // 重写这个方法将oldValue值返回回去，进而刷新缓存
            @Override
            public Object reload(String key, Object oldValue) throws Exception {
                return oldValue;
            }
        };
        return cacheLoader;
    }
}
```

**Caffeine常用配置说明：**

```makefile
initialCapacity=[integer]: 初始的缓存空间大小

maximumSize=[long]: 缓存的最大条数

maximumWeight=[long]: 缓存的最大权重

expireAfterAccess=[duration]: 最后一次写入或访问后经过固定时间过期

expireAfterWrite=[duration]: 最后一次写入后经过固定时间过期

refreshAfterWrite=[duration]: 创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存

weakKeys: 打开key的弱引用

weakValues：打开value的弱引用

softValues：打开value的软引用

recordStats：开发统计功能

注意：

expireAfterWrite和expireAfterAccess同时存在时，以expireAfterWrite为准。

maximumSize和maximumWeight不可以同时使用

weakValues和softValues不可以同时使用
```

需要说明的是，使用配置文件的方式来进行缓存项配置，一般情况能满足使用需求，但是灵活性不是很高，如果我们有很多缓存项的情况下写起来会导致配置文件很长。所以一般情况下你也可以选择使用bean的方式来初始化Cache实例。

下面的演示使用bean的方式来注入：

```java
package com.demo.learn.cache;

import com.github.benmanes.caffeine.cache.CacheLoader;
import com.github.benmanes.caffeine.cache.Caffeine;
import org.apache.commons.compress.utils.Lists;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * @description:
 */
@Configuration
public class CacheConfig {


    /**
     * 创建基于Caffeine的Cache Manager
     * 初始化一些key存入
     * @return
     */
    @Bean
    @Primary
    public CacheManager caffeineCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        ArrayList<CaffeineCache> caches = Lists.newArrayList();
        List<CacheBean> list = setCacheBean();
        for(CacheBean cacheBean : list){
            caches.add(new CaffeineCache(cacheBean.getKey(),
                    Caffeine.newBuilder().recordStats()
                            .expireAfterWrite(cacheBean.getTtl(), TimeUnit.SECONDS)
                            .maximumSize(cacheBean.getMaximumSize())
                            .build()));
        }
        cacheManager.setCaches(caches);
        return cacheManager;
    }


    /**
     * 初始化一些缓存的 key
     * @return
     */
    private List<CacheBean> setCacheBean(){
        List<CacheBean> list = Lists.newArrayList();
        CacheBean userCache = new CacheBean();
        userCache.setKey("userCache");
        userCache.setTtl(60);
        userCache.setMaximumSize(10000);

        CacheBean deptCache = new CacheBean();
        deptCache.setKey("userCache");
        deptCache.setTtl(60);
        deptCache.setMaximumSize(10000);

        list.add(userCache);
        list.add(deptCache);

        return list;
    }

    class CacheBean {
        private String key;
        private long ttl;
        private long maximumSize;

        public String getKey() {
            return key;
        }

        public void setKey(String key) {
            this.key = key;
        }

        public long getTtl() {
            return ttl;
        }

        public void setTtl(long ttl) {
            this.ttl = ttl;
        }

        public long getMaximumSize() {
            return maximumSize;
        }

        public void setMaximumSize(long maximumSize) {
            this.maximumSize = maximumSize;
        }
    }

}
```

创建了一个`SimpleCacheManager`作为Cache的管理对象，然后初始化了两个Cache对象，分别存储user，dept类型的缓存。当然构建Cache的参数设置我写的比较简单，你在使用的时候酌情根据需要配置参数。

#### 使用注解来对 cache 增删改查

我们可以使用spring提供的 `@Cacheable`、`@CachePut`、`@CacheEvict`等注解来方便的使用caffeine缓存。

如果使用了多个cahce，比如redis、caffeine等，必须指定某一个CacheManage为@primary，在@Cacheable注解中没指定 cacheManager 则使用标记为primary的那个。

cache方面的注解主要有以下5个：

- @Cacheable 触发缓存入口（这里一般放在创建和获取的方法上，`@Cacheable`注解会先查询是否已经有缓存，有会使用缓存，没有则会执行方法并缓存）
- @CacheEvict 触发缓存的eviction（用于删除的方法上）
- @CachePut 更新缓存且不影响方法执行（用于修改的方法上，该注解下的方法始终会被执行）
- @Caching 将多个缓存组合在一个方法上（该注解可以允许一个方法同时设置多个注解）
- @CacheConfig 在类级别设置一些缓存相关的共同配置（与其它缓存配合使用）

说一下`@Cacheable` 和 `@CachePut`的区别：

@Cacheable：它的注解的方法是否被执行取决于Cacheable中的条件，方法很多时候都可能不被执行。

@CachePut：这个注解不会影响方法的执行，也就是说无论它配置的条件是什么，方法都会被执行，更多的时候是被用到修改上。

简要说一下Cacheable类中各个方法的使用：

```java
public @interface Cacheable {

    /**
     * 要使用的cache的名字
     */
    @AliasFor("cacheNames")
    String[] value() default {};

    /**
     * 同value()，决定要使用那个/些缓存
     */
    @AliasFor("value")
    String[] cacheNames() default {};

    /**
     * 使用SpEL表达式来设定缓存的key，如果不设置默认方法上所有参数都会作为key的一部分
     */
    String key() default "";

    /**
     * 用来生成key，与key()不可以共用
     */
    String keyGenerator() default "";

    /**
     * 设定要使用的cacheManager，必须先设置好cacheManager的bean，这是使用该bean的名字
     */
    String cacheManager() default "";

    /**
     * 使用cacheResolver来设定使用的缓存，用法同cacheManager，但是与cacheManager不可以同时使用
     */
    String cacheResolver() default "";

    /**
     * 使用SpEL表达式设定出发缓存的条件，在方法执行前生效
     */
    String condition() default "";

    /**
     * 使用SpEL设置出发缓存的条件，这里是方法执行完生效，所以条件中可以有方法执行后的value
     */
    String unless() default "";

    /**
     * 用于同步的，在缓存失效（过期不存在等各种原因）的时候，如果多个线程同时访问被标注的方法
     * 则只允许一个线程通过去执行方法
     */
    boolean sync() default false;

}
```

基于注解的使用方法：

```java
package com.demo.learn.cache;

import com.demo.learn.entity.User;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

/**
 * @description: 本地cache
 */
@Service
public class UserCacheService {


    /**
     * 查找
     * 先查缓存，如果查不到，会查数据库并存入缓存
     * @param id
     */
    @Cacheable(value = "userCache", key = "#id", sync = true)
    public void getUser(long id){
        //查找数据库
    }

    /**
     * 更新/保存
     * @param user
     */
    @CachePut(value = "userCache", key = "#user.id")
    public void saveUser(User user){
        //todo 保存数据库
    }


    /**
     * 删除
     * @param user
     */
    @CacheEvict(value = "userCache",key = "#user.id")
    public void delUser(User user){
        //todo 保存数据库
    }
}
```

如果你不想使用注解的方式去操作缓存，也可以直接使用SimpleCacheManager获取缓存的key进而进行操作。

注意到上面的key使用了spEL 表达式。Spring Cache提供了一些供我们使用的SpEL上下文数据，下表直接摘自Spring官方文档：

| 名称          | 位置       | 描述                                                         | 示例                   |
| ------------- | ---------- | ------------------------------------------------------------ | ---------------------- |
| methodName    | root对象   | 当前被调用的方法名                                           | `#root.methodname`     |
| method        | root对象   | 当前被调用的方法                                             | `#root.method.name`    |
| target        | root对象   | 当前被调用的目标对象实例                                     | `#root.target`         |
| targetClass   | root对象   | 当前被调用的目标对象的类                                     | `#root.targetClass`    |
| args          | root对象   | 当前被调用的方法的参数列表                                   | `#root.args[0]`        |
| caches        | root对象   | 当前方法调用使用的缓存列表                                   | `#root.caches[0].name` |
| Argument Name | 执行上下文 | 当前被调用的方法的参数，如findArtisan(Artisan artisan),可以通过#artsian.id获得参数 | `#artsian.id`          |
| result        | 执行上下文 | 方法执行后的返回值（仅当方法执行后的判断有效，如 unless cacheEvict的beforeInvocation=false） | `#result`              |

**注意：**

1.当我们要使用root对象的属性作为key时我们也可以将“#root”省略，因为Spring默认使用的就是root对象的属性。 如

```java
@Cacheable(key = "targetClass + methodName +#p0")
```

2.使用方法参数时我们可以直接使用“#参数名”或者“#p参数index”。 如：

```java
@Cacheable(value="userCache", key="#id")
@Cacheable(value="userCache", key="#p0")
```

**SpEL提供了多种运算符**

| **类型**   | **运算符**                                     |
| ---------- | ---------------------------------------------- |
| 关系       | <，>，<=，>=，==，!=，lt，gt，le，ge，eq，ne   |
| 算术       | +，- ，* ，/，%，^                             |
| 逻辑       | &&，\|\|，!，and，or，not，between，instanceof |
| 条件       | ?: (ternary)，?: (elvis)                       |
| 正则表达式 | matches                                        |
| 其他类型   | ?.，?[…]，![…]，^[…]，$[…]                     |

# Caffeine源码

### 算法优化：W-TinyLFU

说到优化，Caffine Cache到底优化了什么呢？我们刚提到过LRU，常见的缓存淘汰算法还有FIFO，LFU：

1. FIFO：先进先出，在这种淘汰算法中，先进入缓存的会先被淘汰，会导致命中率很低。
2. LRU：最近最少使用算法，每次访问数据都会将其放在我们的队尾，如果需要淘汰数据，就只需要淘汰队首即可。仍然有个问题，如果有个数据在 1 分钟访问了 1000次，再后 1 分钟没有访问这个数据，但是有其他的数据访问，就导致了我们这个热点数据被淘汰。
3. LFU：最近最少频率使用，利用额外的空间记录每个数据的使用频率，然后选出频率最低进行淘汰。这样就避免了 LRU 不能处理时间段的问题。

上面三种策略各有利弊，实现的成本也是一个比一个高，同时命中率也是一个比一个好。Guava Cache虽然有这么多的功能，但是本质上还是对LRU的封装，如果有更优良的算法，并且也能提供这么多功能，相比之下就相形见绌了。

**LFU的局限性**：在 LFU 中只要数据访问模式的概率分布随时间保持不变时，其命中率就能变得非常高。比如有部新剧出来了，我们使用 LFU 给他缓存下来，这部新剧在这几天大概访问了几亿次，这个访问频率也在我们的 LFU 中记录了几亿次。但是新剧总会过气的，比如一个月之后这个新剧的前几集其实已经过气了，但是他的访问量的确是太高了，其他的电视剧根本无法淘汰这个新剧，所以在这种模式下是有局限性。

**LRU的优点和局限性**：LRU可以很好的应对突发流量的情况，因为他不需要累计数据频率。但LRU通过历史数据来预测未来是局限的，它会认为最后到来的数据是最可能被再次访问的，从而给与它最高的优先级。

在现有算法的局限性下，会导致缓存数据的命中率或多或少的受损，而命中略又是缓存的重要指标。HighScalability网站刊登了一篇文章，由前Google工程师发明的W-TinyLFU——一种现代的缓存 。Caffine Cache就是基于此算法而研发。Caffeine 因使用 **Window TinyLfu** 回收策略，提供了一个**近乎最佳的命中率**。

> 当数据的访问模式不随时间变化的时候，LFU的策略能够带来最佳的缓存命中率。然而LFU有两个缺点：
>
> 首先，它需要给每个记录项维护频率信息，每次访问都需要更新，这是个巨大的开销；
>
> 其次，如果数据访问模式随时间有变，LFU的频率信息无法随之变化，因此早先频繁访问的记录可能会占据缓存，而后期访问较多的记录则无法被命中。
>
> 因此，大多数的缓存设计都是基于LRU或者其变种来进行的。相比之下，LRU并不需要维护昂贵的缓存记录元信息，同时也能够反应随时间变化的数据访问模式。然而，在许多负载之下，LRU依然需要更多的空间才能做到跟LFU一致的缓存命中率。因此，一个“现代”的缓存，应当能够综合两者的长处。

TinyLFU维护了近期访问记录的频率信息，作为一个过滤器，当新记录来时，只有满足TinyLFU要求的记录才可以被插入缓存。如前所述，作为现代的缓存，它需要解决两个挑战：

一个是如何避免维护频率信息的高开销；

另一个是如何反应随时间变化的访问模式。

首先来看前者，TinyLFU借助了数据流Sketching技术，Count-Min Sketch显然是解决这个问题的有效手段，它可以用小得多的空间存放频率信息，而保证很低的False Positive Rate。但考虑到第二个问题，就要复杂许多了，因为我们知道，任何Sketching数据结构如果要反应时间变化都是一件困难的事情，在Bloom Filter方面，我们可以有Timing Bloom Filter，但对于CMSketch来说，如何做到Timing CMSketch就不那么容易了。TinyLFU采用了一种基于滑动窗口的时间衰减设计机制，借助于一种简易的reset操作：每次添加一条记录到Sketch的时候，都会给一个计数器上加1，当计数器达到一个尺寸W的时候，把所有记录的Sketch数值都除以2，该reset操作可以起到衰减的作用 。

W-TinyLFU主要用来解决一些稀疏的突发访问元素。在一些数目很少但突发访问量很大的场景下，TinyLFU将无法保存这类元素，因为它们无法在给定时间内积累到足够高的频率。因此W-TinyLFU就是结合LFU和LRU，前者用来应对大多数场景，而LRU用来处理突发流量。

在处理频率记录的方案中，你可能会想到用hashMap去存储，每一个key对应一个频率值。那如果数据量特别大的时候，是不是这个hashMap也会特别大呢。由此可以联想到 Bloom Filter，对于每个key，用n个byte每个存储一个标志用来判断key是否在集合中。原理就是使用k个hash函数来将key散列成一个整数。

在W-TinyLFU中使用Count-Min Sketch记录我们的访问频率，而这个也是布隆过滤器的一种变种。如下图所示:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190615192738125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E5NTM3MTM0Mjg=,size_16,color_FFFFFF,t_70)

如果需要记录一个值，那我们需要通过多种Hash算法对其进行处理hash，然后在对应的hash算法的记录中+1，为什么需要多种hash算法呢？由于这是一个压缩算法必定会出现冲突，比如我们建立一个byte的数组，通过计算出每个数据的hash的位置。比如张三和李四，他们两有可能hash值都是相同，比如都是1那byte[1]这个位置就会增加相应的频率，张三访问1万次，李四访问1次那byte[1]这个位置就是1万零1，如果取李四的访问评率的时候就会取出是1万零1，但是李四命名只访问了1次啊，为了解决这个问题，所以用了多个hash算法可以理解为long[][]二维数组的一个概念，比如在第一个算法张三和李四冲突了，但是在第二个，第三个中很大的概率不冲突，比如一个算法大概有1%的概率冲突，那四个算法一起冲突的概率是1%的四次方。通过这个模式我们取李四的访问率的时候取所有算法中，李四访问最低频率的次数。所以他的名字叫Count-Min Sketch。

# 问题

## 固定数据（Pinning Entries）

固定数据是不能通过驱逐策略去将数据删除的。当数据是一个有状态的资源时（如锁），那么这条数据是非常有用的，你有在客端使用完这个条数据的时候才能删除该数据。在这种情况下如果驱逐策略将这个条数据删掉的话，将导致资源泄露。

通过使用权重将该数据的权重设置成0，并且这个条数据不计入maximum size里面。 当缓存达到maximum size 了以后，驱逐策略也会跳过该条数据，不会进行删除操作。我们还必须自定义一个标准来判断这个数据是否属于固定数据。

通过使用Long.MAX_VALUE（大约300年）的值作为key的有效时间，这样可以将一条数据从过期中排除。自定义到期必须定义，这可以评估条目是否固定。

将数据写入缓存时我们要指定该数据的权重和到期时间。这可以通过使用cache.asMap()获取缓存列表后，再来实现引脚和解除绑定。

## 递归调用（Recursive Computations）

在原子操作内执行的加载，计算或回调可能不会写入缓存。 ConcurrentHashMap不允许这些递归写操作，因为这将有可能导致活锁（Java 8）或IllegalStateException（Java 9）。

解决方法是异步执行这些操作，例如使用AsyncLoadingCache。在异步这种情况下映射已经建立，value是一个CompletableFuture，并且这些操作是在缓存的原子范围之外执行的。但是，如果发生无序的依赖链，这也有可能导致死锁。

## Caffeine Cache 配置套路

使用 Caffeine Cache，除了 Spring 中常见的 @EnableCache、@Cacheable 等注解外，直接使用 Caffeine.newBuilder().build() 方法创建 LoadingCache 也是推荐服务常用的方式。

我们先来看看 Caffeine#builder 都有哪些配置套路：

![img](https://static001.geekbang.org/infoq/19/19f70629e7cbea116336cf37a2a91997.png)

## expireAfterWrite、expireAfterAccess 都配置？

虽然 expireAfterWrite 和 expireAfterAccess 同时配置不报错，但 access 包含了 write，所以选一个就好了亲。

## reference-based 驱逐有啥特点？

只要配置上都会使用 == 来比较对象相等，而不是 equals；还有一个非常重要的配置，也是决定缓存如丝般顺滑的秘诀：刷新策略 refreshAfterWrite。该配置使得 Caffeine 可以在数据加载后超过给定时间时刷新数据。下文详解。

## Builder 上也能踩坑

和 lombok 的 builder 不同，Caffeine#builder 的策略调用两次将会导致运行时异常！这是因为 Caffeine 构建时每个策略都保存了已设置的标记位，所以重复设置并不是覆盖而是直接抛异常：

```java
public Caffeine<K, V> maximumWeight(@NonNegative long maximumWeight) {
  requireState(this.maximumWeight == UNSET_INT,
      "maximum weight was already set to %s", this.maximumWeight);
  requireState(this.maximumSize == UNSET_INT,
      "maximum size was already set to %s", this.maximumSize);
  this.maximumWeight = maximumWeight;
  requireArgument(maximumWeight >= 0, "maximum weight must not be negative");
  return this;
}
```

比如上述代码，maximumWeight() 调用两次的话就会抛出异常并提示 maximum weight was already set to xxx。

## Caffeine Cache 精华

### get 方法都做了什么？

首先在实现类 LocalLoadingCache<K, V> 中可以看到；

```java
default @Nullable V get(K key) {
    return cache().computeIfAbsent(key, mappingFunction());
}
```

但突然发现这个 get 方法没有实现类！Why？我们跟踪 cache() 方法就可以发现端倪：

```java
public BoundedLocalCache<K, V> cache() {
    return cache;
}
public UnboundedLocalCache<K, V> cache() {
    return cache;
}
```

根据调用 Caffeine.newBuilder().build() 的过程，决定了具体生成的是 BoundedLocalCache 还是 UnboundedLocalCache；

判定 BoundedLocalCache 的条件如下：

```java
public <K1 extends K, V1 extends V> LoadingCache<K1, V1> build(
    @NonNull CacheLoader<? super K1, V1> loader) {
  requireWeightWithWeigher();
 
  @SuppressWarnings("unchecked")
  Caffeine<K1, V1> self = (Caffeine<K1, V1>) this;
  return isBounded() || refreshes()
      ? new BoundedLocalCache.BoundedLocalLoadingCache<>(self, loader)
      : new UnboundedLocalCache.UnboundedLocalLoadingCache<>(self, loader);
}
```

其中的 isBounded()、refreshes() 方法分别如下：

```java
boolean isBounded() {
  return (maximumSize != UNSET_INT)
      || (maximumWeight != UNSET_INT)
      || (expireAfterAccessNanos != UNSET_INT)
      || (expireAfterWriteNanos != UNSET_INT)
      || (expiry != null)
      || (keyStrength != null)
      || (valueStrength != null);
}
boolean refreshes() {
  // 调用了 refreshAfter 就会返回 false
  return refreshNanos != UNSET_INT;
}
```

可以看到一般情况下常规的配置都是 BoundedLocalCache。所以我们以它为例继续看 BoundedLocalCache#computeIfAbsent 方法吧：

```java
  public @Nullable V computeIfAbsent(K key,
    Function<? super K, ? extends V> mappingFunction,
    boolean recordStats, boolean recordLoad) {
  // 常用的 LoadingCache#get 方法 recordStats、recordLoad 都为 true
  // mappingFunction 即 builder 中传入的 CacheLoader 实例包装
 
  requireNonNull(key);
  requireNonNull(mappingFunction);
  // 默认的 ticker read 返回的是 System.nanoTime();
  // 关于其他的 ticker 见文末参考文献，可以让使用者自定义超时的计时方式
  long now = expirationTicker().read();
 
  // data 是 ConcurrentHashMap<Object, Node<K, V>>
  // key 根据代码目前都是 LookupKeyReference 对象
  // 可以发现 LookupKeyReference 保存的是 System.identityHashCode(key) 结果
  // 关于 identityHashCode 和 hashCode 的区别可阅读文末参考资料
  Node<K, V> node = data.get(nodeFactory.newLookupKey(key));
  if (node != null) {
    V value = node.getValue();
    if ((value != null) && !hasExpired(node, now)) {
      // isComputingAsync 中将会判断当前是否为异步类的缓存实例
      // 是的话再判断 node.getValue 是否完成。BoundedLocaCache 总是返回 false
      if (!isComputingAsync(node)) {
        // 此处在 BoundedLocaCache 中也是直接 return 不会执行
        tryExpireAfterRead(node, key, value, expiry(), now);
        setAccessTime(node, now);
      }
 
      // 异步驱逐任务提交、异步刷新操作
      // CacheLoader#asyncReload 就在其中的 refreshIfNeeded 方法被调用
      afterRead(node, now, recordStats);
      return value;
    }
  }
  if (recordStats) {
    // 记录缓存的加载成功、失败等统计信息
    mappingFunction = statsAware(mappingFunction, recordLoad);
  }
 
  // 这里2.8.0版本不同实现类生成的都是 WeakKeyReference
  Object keyRef = nodeFactory.newReferenceKey(key, keyReferenceQueue());
 
  // 本地缓存没有，使用加载函数读取到缓存
  return doComputeIfAbsent(key, keyRef, mappingFunction,
    new long[] { now }, recordStats);
}
```

上文中 hasExpired 判断数据是否过期，看代码就很明白了：是通过 builder 的配置 + 时间计算来判断的。

```java
boolean hasExpired(Node<K, V> node, long now) {
  return
    (expiresAfterAccess() &&
      (now - node.getAccessTime() >= expiresAfterAccessNanos()))
  | (expiresAfterWrite() &&
      (now - node.getWriteTime() >= expiresAfterWriteNanos()))
  | (expiresVariable() &&
      (now - node.getVariableTime() >= 0));
}
```

继续看代码，doComputeIfAbsent 方法主要内容如下：

```java
  @Nullable V doComputeIfAbsent(K key, Object keyRef,
    Function<? super K, ? extends V> mappingFunction,
    long[] now, boolean recordStats) {
  @SuppressWarnings("unchecked")
  V[] oldValue = (V[]) new Object[1];
  @SuppressWarnings("unchecked")
  V[] newValue = (V[]) new Object[1];
  @SuppressWarnings("unchecked")
  K[] nodeKey = (K[]) new Object[1];
  @SuppressWarnings({"unchecked", "rawtypes"})
  Node<K, V>[] removed = new Node[1];
 
  int[] weight = new int[2]; // old, new
  RemovalCause[] cause = new RemovalCause[1];
 
  // 对 data 这个 ConcurrentHashMap 调用 compute 方法，计算 key 对应的值
  // compute 方法的执行是原子的，并且会对 key 加锁
  // JDK 注释说明 compute 应该短而快并且不要在其中更新其他的 key-value
  Node<K, V> node = data.compute(keyRef, (k, n) -> {
    if (n == null) {
      // 没有值的时候调用 builder 传入的 CacheLoader#load 方法
      // mappingFunction 是在 LocalLoadingCache#newMappingFunction 中创建的
      newValue[0] = mappingFunction.apply(key);
      if (newValue[0] == null) {
        return null;
      }
      now[0] = expirationTicker().read();
 
      // builder 没有指定 weigher 时，这里默认为 SingletonWeigher，总是返回 1
      weight[1] = weigher.weigh(key, newValue[0]);
      n = nodeFactory.newNode(key, keyReferenceQueue(),
          newValue[0], valueReferenceQueue(), weight[1], now[0]);
      setVariableTime(n, expireAfterCreate(key, newValue[0], expiry(), now[0]));
      return n;
    }
 
    // 有值的时候对 node 实例加同步块
    synchronized (n) {
      nodeKey[0] = n.getKey();
      weight[0] = n.getWeight();
      oldValue[0] = n.getValue();
 
      // 设置驱逐原因，如果数据有效直接返回
      if ((nodeKey[0] == null) || (oldValue[0] == null)) {
        cause[0] = RemovalCause.COLLECTED;
      } else if (hasExpired(n, now[0])) {
        cause[0] = RemovalCause.EXPIRED;
      } else {
        return n;
      }
 
      // 默认的配置 writer 是 CacheWriter.disabledWriter()，无操作；
      // 自己定义的 CacheWriter 一般用于驱逐数据时得到回调进行外部数据源操作
      // 详情可以参考文末的资料
      writer.delete(nodeKey[0], oldValue[0], cause[0]);
      newValue[0] = mappingFunction.apply(key);
      if (newValue[0] == null) {
        removed[0] = n;
        n.retire();
        return null;
      }
      weight[1] = weigher.weigh(key, newValue[0]);
      n.setValue(newValue[0], valueReferenceQueue());
      n.setWeight(weight[1]);
 
      now[0] = expirationTicker().read();
      setVariableTime(n, expireAfterCreate(key, newValue[0], expiry(), now[0]));
      setAccessTime(n, now[0]);
      setWriteTime(n, now[0]);
      return n;
    }
  });
 
  // 剩下的代码主要是调用 afterWrite、notifyRemoval 等方法
  // 进行后置操作，后置操作中将会再次尝试缓存驱逐
  // ...
  return newValue[0];
}
```

看完上面的代码，遇到这些问题也就心里有数了。

### 缓存的数据什么时候淘汰？

显式调用 invalid 方法时；弱引用、软引用可回收时；get 方法老值存在且已完成异步加载后调用 afterRead。

get 方法老值不存在，调用 doComputeIfAbsent 加载完数据后调用 afterWrite。

### CacheLoader#load和 CacheLoader#asyncReload 有什么区别？

首先 CacheLoader#load 方法是必须提供的，缓存调用时将是同步操作（回顾上文 data.compute 方法），会阻塞当前线程。

而 CacheLoader#asyncReload 需要配合builder#refreshAfterWrite 使用这样将在computeIfAbsent->afterRead->refreshIfNeeded 中调用，并异步更新到 data 对象上；并且，load 方法没有传入oldValue，而 asyncReload 方法提供了oldValue，这意味着如果触发 load 操作时，缓存是不能保证 oldValue 是否存在的（可能是首次，也可能是已失效）。

### 加载数据耗时较长，对性能的影响是什么？

CacheLoader#load 耗时长，将会导致缓存运行过程中查询数据时阻塞等待加载，当多个线程同时查询同一个 key 时，业务请求可能阻塞，甚至超时失败；

CacheLoader#asyncReload 耗时长，在时间周期满足的情况下，即使耗时长，对业务的影响也较小

### 说好的如丝般顺滑呢？

首要前提是外部数据查询能保证单次查询的性能（一次查询天长地久那加本地缓存也于事无补）；然后，我们在构建 LoadingCache 时，配置 refreshAfterWrite 并在 CacheLoader 实例上定义 asyncReload 方法；

**灵魂追问：只有以上两步就够了吗？**

机智的我突然觉得事情并不简单。还有一个时间设置的问题，我们来看看：

> 如果 expireAfterWrite 周期 < refreshAfterWrite 周期会如何？此时查询失效数据时总是会调用 load 方法，refreshAfterWrite 根本没用！
>
> 如果 CacheLoader#asyncReload 有额外操作，导致它自身实际执行查询耗时超过 expireAfterWrite 又会如何？还是 CacheLoader#load 生效，refreshAfterWrite 还是没用！

所以丝滑的正确打开方式，是 refreshAfterWrite 周期明显小于 expireAfterWrite 周期，并且 CacheLoader#asyncReload 本身也有较好的性能，才能如丝般顺滑地加载数据。此时就会发现业务不断进行 get 操作，根本感知不到数据加载时的卡顿！

### 用本地缓存会不会出现缓存穿透？怎么防止？

computeIfAbsent 和 doComputeIfAbsent 方法可以看出如果加载结果是 null，那么每次从缓存查询，都会触发 mappingFunction.apply，进一步调用 CacheLoader#load。从而流量会直接打到后端数据库，造成缓存穿透。

防止的方法也比较简单，在业务可接受的情况下，如果未能查询到结果，则返回一个非 null 的“假对象”到本地缓存中。

**灵魂追问：如果查不到，new 一个对象返回行不行？**

key 范围不大时可以，builder 设置了 size-based 驱逐策略时可以，但都存在消耗较多内存的风险，可以定义一个默认的 PLACE_HOLDER 静态对象作为引用。

**灵魂追问：都用同一个假对象引用真的大丈夫（没问题）？**

这么大的坑本菜鸟怎么能错过！缓存中存的是对象引用，如果业务 get 后修改了对象的内容，那么其他线程再次获取到这个对象时，将会得到修改后的值！鬼知道那个深夜定位出这个问题的我有多兴奋（苍蝇搓手）。

当时缓存中保存的是 List<Item>，而不同线程中对这些 item 的 score 进行了不同的 set 操作，导致同一个 item 排序后的分数和顺序变幻莫测。本菜鸟一度以为是推荐之神降临，冥冥中加持 CTR 所以把 score 变来变去。

**灵魂追问：那怎么解决缓存被意外修改的问题呢？怎么 copy 一个对象呢？**

So easy，就在 get 的时候 copy 一下对象就好了。

灵魂追问4：怎么 copy 一个对象？……停！咱们以后有机会再来说说这个浅拷贝和深拷贝，以及常见的拷贝工具吧，聚焦聚焦……

### 某次加载数据失败怎么办，还能用之前的缓存值吗？

根据 CacheLoader#load和 CacheLoader#asyncReload 的参数区别，我们可以发现：

应该在 asyncReload 中来处理，如果查询数据库异常，则可以返回 oldValue 来继续使用之前的缓存；否则只能通过 load 方法中返回预留空对象来解决。使用哪一种方法需要根据具体的业务场景来决定。

【踩坑】返回 null 将导致 Caffeine 认为该值不需要缓存，下次查询还会继续调用 load 方法，缓存并没生效。

### 多个线程同时 get 一个本地缓存不存在的值，会如何？

根据代码可以知道，已经进入 doComputeIfAbsent 的线程将阻塞在 data.compute 方法上；

比如短时间内有 N 个线程同时 get 相同的 key 并且 key 不存在，则这 N 个线程最终都会反复执行 compute 方法。但只要 data 中该 key 的值更新成功，其他进入 computeIfAbsent 的线程都可直接获得结果返回，不会出现阻塞等待加载；

所以，如果一开始就有大量请求进入 doComputeIfAbsent 阻塞等待数据，就会造成短时间请求挂起、超时的问题。由此在大流量场景下升级服务时，需要考虑在接入流量前对缓存进行预热（我查我自己，嗯），防止瞬时请求太多导致大量请求挂起或超时。

灵魂追问：如果一次 load 耗时 100ms，一开始有 10 个线程冷启动，最终等待时间会是 1s 左右吗？

其实……要看情况，回顾一下 data.compute 里面的代码：

```java
if (n == null) {
    // 这部分代码其他后续线程进入后已经有值，不再执行
}
synchronized (n) {
  // ...
 
  if ((nodeKey[0] == null) || (oldValue[0] == null)) {
    cause[0] = RemovalCause.COLLECTED;
  } else if (hasExpired(n, now[0])) {
    cause[0] = RemovalCause.EXPIRED;
  } else {
    // 未失效时在这里返回，不会触发 load 函数
    return n;
  }
 
  // ...
}
```

所以，如果 load 结果不是 null，那么只第一个线程花了 100ms，后续线程会尽快返回，最终时长应该只比 100ms 多一点。但如果 load 结果返回 null（缓存穿透），相当于没有查到数据，于是后续线程还会再次执行 load，最终时间就是 1s 左右。

## 示例代码：

```
package com.demo.controller;

import com.alibaba.fastjson.JSON;
import com.github.benmanes.caffeine.cache.*;
import com.google.common.testing.FakeTicker;
import com.xiaolyuh.entity.Person;
import com.xiaolyuh.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.OptionalLong;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.Executor;
import java.util.concurrent.TimeUnit;

@RestController
public class CaffeineCacheController {

    @Autowired
    PersonService personService;

    Cache<String, Object> manualCache = Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(10_000)
            .build();

    LoadingCache<String, Object> loadingCache = Caffeine.newBuilder()
            .maximumSize(10_000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build(key -> createExpensiveGraph(key));

    AsyncLoadingCache<String, Object> asyncLoadingCache = Caffeine.newBuilder()
            .maximumSize(10_000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            // Either: Build with a synchronous computation that is wrapped as asynchronous
            .buildAsync(key -> createExpensiveGraph(key));
    // Or: Build with a asynchronous computation that returns a future
    // .buildAsync((key, executor) -> createExpensiveGraphAsync(key, executor));

    private CompletableFuture<Object> createExpensiveGraphAsync(String key, Executor executor) {
        CompletableFuture<Object> objectCompletableFuture = new CompletableFuture<>();
        return objectCompletableFuture;
    }

    private Object createExpensiveGraph(String key) {
        System.out.println("缓存不存在或过期，调用了createExpensiveGraph方法获取缓存key的值");
        if (key.equals("name")) {
            throw new RuntimeException("调用了该方法获取缓存key的值的时候出现异常");
        }
        return personService.findOne1();
    }

    @RequestMapping("/testManual")
    public Object testManual(Person person) {
        String key = "name1";
        Object graph = null;

        // 根据key查询一个缓存，如果没有返回NULL
        graph = manualCache.getIfPresent(key);
        // 根据Key查询一个缓存，如果没有调用createExpensiveGraph方法，并将返回值保存到缓存。
        // 如果该方法返回Null则manualCache.get返回null，如果该方法抛出异常则manualCache.get抛出异常
        graph = manualCache.get(key, k -> createExpensiveGraph(k));
        // 将一个值放入缓存，如果以前有值就覆盖以前的值
        manualCache.put(key, graph);
        // 删除一个缓存
        manualCache.invalidate(key);

        ConcurrentMap<String, Object> map = manualCache.asMap();
        System.out.println(map.toString());
        return graph;
    }

    @RequestMapping("/testLoading")
    public Object testLoading(Person person) {
        String key = "name1";

        // 采用同步方式去获取一个缓存和上面的手动方式是一个原理。在build Cache的时候会提供一个createExpensiveGraph函数。
        // 查询并在缺失的情况下使用同步的方式来构建一个缓存
        Object graph = loadingCache.get(key);

        // 获取组key的值返回一个Map
        List<String> keys = new ArrayList<>();
        keys.add(key);
        Map<String, Object> graphs = loadingCache.getAll(keys);
        return graph;
    }

    @RequestMapping("/testAsyncLoading")
    public Object testAsyncLoading(Person person) {
        String key = "name1";

        // 查询并在缺失的情况下使用异步的方式来构建缓存
        CompletableFuture<Object> graph = asyncLoadingCache.get(key);
        // 查询一组缓存并在缺失的情况下使用异步的方式来构建缓存
        List<String> keys = new ArrayList<>();
        keys.add(key);
        CompletableFuture<Map<String, Object>> graphs = asyncLoadingCache.getAll(keys);

        // 异步转同步
        loadingCache = asyncLoadingCache.synchronous();
        return graph;
    }

    @RequestMapping("/testSizeBased")
    public Object testSizeBased(Person person) {
        LoadingCache<String, Object> cache = Caffeine.newBuilder()
                .maximumSize(1)
                .build(k -> createExpensiveGraph(k));

        cache.get("A");
        System.out.println(cache.estimatedSize());
        cache.get("B");
        // 因为执行回收的方法是异步的，所以需要调用该方法，手动触发一次回收操作。
        cache.cleanUp();
        System.out.println(cache.estimatedSize());

        return "";
    }

    @RequestMapping("/testTimeBased")
    public Object testTimeBased(Person person) {
        String key = "name1";
        // 用户测试，一个时间源，返回一个时间值，表示从某个固定但任意时间点开始经过的纳秒数。
        FakeTicker ticker = new FakeTicker();

        // 基于固定的到期策略进行退出
        // expireAfterAccess
        LoadingCache<String, Object> cache1 = Caffeine.newBuilder()
                .ticker(ticker::read)
                .expireAfterAccess(5, TimeUnit.SECONDS)
                .build(k -> createExpensiveGraph(k));

        System.out.println("expireAfterAccess：第一次获取缓存");
        cache1.get(key);

        System.out.println("expireAfterAccess：等待4.9S后，第二次次获取缓存");
        // 直接指定时钟
        ticker.advance(4900, TimeUnit.MILLISECONDS);
        cache1.get(key);

        System.out.println("expireAfterAccess：等待0.101S后，第三次次获取缓存");
        ticker.advance(101, TimeUnit.MILLISECONDS);
        cache1.get(key);

        // expireAfterWrite
        LoadingCache<String, Object> cache2 = Caffeine.newBuilder()
                .ticker(ticker::read)
                .expireAfterWrite(5, TimeUnit.SECONDS)
                .build(k -> createExpensiveGraph(k));

        System.out.println("expireAfterWrite：第一次获取缓存");
        cache2.get(key);

        System.out.println("expireAfterWrite：等待4.9S后，第二次次获取缓存");
        ticker.advance(4900, TimeUnit.MILLISECONDS);
        cache2.get(key);

        System.out.println("expireAfterWrite：等待0.101S后，第三次次获取缓存");
        ticker.advance(101, TimeUnit.MILLISECONDS);
        cache2.get(key);

        // Evict based on a varying expiration policy
        // 基于不同的到期策略进行退出
        LoadingCache<String, Object> cache3 = Caffeine.newBuilder()
                .ticker(ticker::read)
                .expireAfter(new Expiry<String, Object>() {

                    @Override
                    public long expireAfterCreate(String key, Object value, long currentTime) {
                        // Use wall clock time, rather than nanotime, if from an external resource
                        return TimeUnit.SECONDS.toNanos(5);
                    }

                    @Override
                    public long expireAfterUpdate(String key, Object graph,
                                                  long currentTime, long currentDuration) {

                        System.out.println("调用了 expireAfterUpdate：" + TimeUnit.NANOSECONDS.toMillis(currentDuration));
                        return currentDuration;
                    }

                    @Override
                    public long expireAfterRead(String key, Object graph,
                                                long currentTime, long currentDuration) {

                        System.out.println("调用了 expireAfterRead：" + TimeUnit.NANOSECONDS.toMillis(currentDuration));
                        return currentDuration;
                    }
                })
                .build(k -> createExpensiveGraph(k));

        System.out.println("expireAfter：第一次获取缓存");
        cache3.get(key);

        System.out.println("expireAfter：等待4.9S后，第二次次获取缓存");
        ticker.advance(4900, TimeUnit.MILLISECONDS);
        cache3.get(key);

        System.out.println("expireAfter：等待0.101S后，第三次次获取缓存");
        ticker.advance(101, TimeUnit.MILLISECONDS);
        Object object = cache3.get(key);

        return object;
    }

    @RequestMapping("/testRemoval")
    public Object testRemoval(Person person) {
        String key = "name1";
        // 用户测试，一个时间源，返回一个时间值，表示从某个固定但任意时间点开始经过的纳秒数。
        FakeTicker ticker = new FakeTicker();

        // 基于固定的到期策略进行退出
        // expireAfterAccess
        LoadingCache<String, Object> cache = Caffeine.newBuilder()
                .removalListener((String k, Object graph, RemovalCause cause) ->
                        System.out.printf("Key %s was removed (%s)%n", k, cause))
                .ticker(ticker::read)
                .expireAfterAccess(5, TimeUnit.SECONDS)
                .build(k -> createExpensiveGraph(k));

        System.out.println("第一次获取缓存");
        Object object = cache.get(key);

        System.out.println("等待6S后，第二次次获取缓存");
        // 直接指定时钟
        ticker.advance(6000, TimeUnit.MILLISECONDS);
        cache.get(key);

        System.out.println("手动删除缓存");
        cache.invalidate(key);

        return object;
    }

    @RequestMapping("/testRefresh")
    public Object testRefresh(Person person) {
        String key = "name1";
        // 用户测试，一个时间源，返回一个时间值，表示从某个固定但任意时间点开始经过的纳秒数。
        FakeTicker ticker = new FakeTicker();

        // 基于固定的到期策略进行退出
        // expireAfterAccess
        LoadingCache<String, Object> cache = Caffeine.newBuilder()
                .removalListener((String k, Object graph, RemovalCause cause) ->
                        System.out.printf("执行移除监听器- Key %s was removed (%s)%n", k, cause))
                .ticker(ticker::read)
                .expireAfterWrite(5, TimeUnit.SECONDS)
                // 指定在创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存
                .refreshAfterWrite(4, TimeUnit.SECONDS)
                .build(k -> createExpensiveGraph(k));

        System.out.println("第一次获取缓存");
        Object object = cache.get(key);

        System.out.println("等待4.1S后，第二次次获取缓存");
        // 直接指定时钟
        ticker.advance(4100, TimeUnit.MILLISECONDS);
        cache.get(key);

        System.out.println("等待5.1S后，第三次次获取缓存");
        // 直接指定时钟
        ticker.advance(5100, TimeUnit.MILLISECONDS);
        cache.get(key);

        return object;
    }

    @RequestMapping("/testWriter")
    public Object testWriter(Person person) {
        String key = "name1";
        // 用户测试，一个时间源，返回一个时间值，表示从某个固定但任意时间点开始经过的纳秒数。
        FakeTicker ticker = new FakeTicker();

        // 基于固定的到期策略进行退出
        // expireAfterAccess
        LoadingCache<String, Object> cache = Caffeine.newBuilder()
                .removalListener((String k, Object graph, RemovalCause cause) ->
                        System.out.printf("执行移除监听器- Key %s was removed (%s)%n", k, cause))
                .ticker(ticker::read)
                .expireAfterWrite(5, TimeUnit.SECONDS)
                .writer(new CacheWriter<String, Object>() {
                    @Override
                    public void write(String key, Object graph) {
                        // write to storage or secondary cache
                        // 写入存储或者二级缓存
                        System.out.printf("testWriter:write - Key %s was write (%s)%n", key, graph);
                        createExpensiveGraph(key);
                    }

                    @Override
                    public void delete(String key, Object graph, RemovalCause cause) {
                        // delete from storage or secondary cache
                        // 删除存储或者二级缓存
                        System.out.printf("testWriter:delete - Key %s was delete (%s)%n", key, graph);
                    }
                })
                // 指定在创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存
                .refreshAfterWrite(4, TimeUnit.SECONDS)
                .build(k -> createExpensiveGraph(k));

        cache.put(key, personService.findOne1());
        cache.invalidate(key);

        System.out.println("第一次获取缓存");
        Object object = cache.get(key);

        System.out.println("等待4.1S后，第二次次获取缓存");
        // 直接指定时钟
        ticker.advance(4100, TimeUnit.MILLISECONDS);
        cache.get(key);

        System.out.println("等待5.1S后，第三次次获取缓存");
        // 直接指定时钟
        ticker.advance(5100, TimeUnit.MILLISECONDS);
        cache.get(key);

        return object;
    }

    @RequestMapping("/testStatistics")
    public Object testStatistics(Person person) {
        String key = "name1";
        // 用户测试，一个时间源，返回一个时间值，表示从某个固定但任意时间点开始经过的纳秒数。
        FakeTicker ticker = new FakeTicker();

        // 基于固定的到期策略进行退出
        // expireAfterAccess
        LoadingCache<String, Object> cache = Caffeine.newBuilder()
                .removalListener((String k, Object graph, RemovalCause cause) ->
                        System.out.printf("执行移除监听器- Key %s was removed (%s)%n", k, cause))
                .ticker(ticker::read)
                .expireAfterWrite(5, TimeUnit.SECONDS)
                // 开启统计
                .recordStats()
                // 指定在创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存
                .refreshAfterWrite(4, TimeUnit.SECONDS)
                .build(k -> createExpensiveGraph(k));

        for (int i = 0; i < 10; i++) {
            cache.get(key);
            cache.get(key + i);
        }
        // 驱逐是异步操作，所以这里要手动触发一次回收操作
        ticker.advance(5100, TimeUnit.MILLISECONDS);
        // 手动触发一次回收操作
        cache.cleanUp();

        System.out.println("缓存命数量：" + cache.stats().hitCount());
        System.out.println("缓存命中率：" + cache.stats().hitRate());
        System.out.println("缓存逐出的数量：" + cache.stats().evictionCount());
        System.out.println("加载新值所花费的平均时间：" + cache.stats().averageLoadPenalty());

        return cache.get(key);
    }

    @RequestMapping("/testPolicy")
    public Object testPolicy(Person person) {
        FakeTicker ticker = new FakeTicker();

        LoadingCache<String, Object> cache = Caffeine.newBuilder()
                .ticker(ticker::read)
                .expireAfterAccess(5, TimeUnit.SECONDS)
                .maximumSize(1)
                .build(k -> createExpensiveGraph(k));

        // 在代码里面动态的指定最大Size
        cache.policy().eviction().ifPresent(eviction -> {
            eviction.setMaximum(4 * eviction.getMaximum());
        });

        cache.get("E");
        cache.get("B");
        cache.get("C");
        cache.cleanUp();
        System.out.println(cache.estimatedSize() + ":" + JSON.toJSON(cache.asMap()).toString());

        cache.get("A");
        ticker.advance(100, TimeUnit.MILLISECONDS);
        cache.get("D");
        ticker.advance(100, TimeUnit.MILLISECONDS);
        cache.get("A");
        ticker.advance(100, TimeUnit.MILLISECONDS);
        cache.get("B");
        ticker.advance(100, TimeUnit.MILLISECONDS);
        cache.policy().eviction().ifPresent(eviction -> {
            // 获取热点数据Map
            Map<String, Object> hottestMap = eviction.hottest(10);
            // 获取冷数据Map
            Map<String, Object> coldestMap = eviction.coldest(10);

            System.out.println("热点数据:" + JSON.toJSON(hottestMap).toString());
            System.out.println("冷数据:" + JSON.toJSON(coldestMap).toString());
        });

        ticker.advance(3000, TimeUnit.MILLISECONDS);
        // ageOf通过这个方法来查看key的空闲时间
        cache.policy().expireAfterAccess().ifPresent(expiration -> {

            System.out.println(JSON.toJSON(expiration.ageOf("A", TimeUnit.MILLISECONDS)));
        });
        return cache.get("name1");
    }
}
```

#  参考资料

caffeineWiki：https://github.com/ben-manes/caffeine/wiki 

caffeine：https://github.com/ben-manes/caffeine

caffeine：https://github.com/ben-manes/caffeine/blob/master/README.md



### 



