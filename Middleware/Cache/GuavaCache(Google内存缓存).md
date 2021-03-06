# Guava cache

缓存分为本地缓存和远端缓存。常见的远端缓存有Redis，MongoDB；本地缓存一般使用map的方式保存在本地内存中。一般我们在业务中操作缓存，都会操作缓存和数据源两部分。如：put数据时，先插入DB，再删除原来的缓存；ge数据时，先查缓存，命中则返回，没有命中时，需要查询DB，再把查询结果放入缓存中 。如果访问量大，我们还得兼顾本地缓存的线程安全问题。必要的时候也要考虑缓存的回收策略。

今天说的 Guava Cache 是google guava中的一个内存缓存模块，用于将数据缓存到JVM内存中。他很好的解决了上面提到的几个问题：

- 很好的封装了get、put操作，能够集成数据源 ；
- 线程安全的缓存，与ConcurrentMap相似，但前者增加了更多的元素失效策略，后者只能显示的移除元素；
- Guava Cache提供了三种基本的缓存回收方式：基于容量回收、定时回收和基于引用回收。定时回收有两种：按照写入时间，最早写入的最先回收；按照访问时间，最早访问的最早回收；
- 监控缓存加载/命中情况

Guava Cache的架构设计灵感ConcurrentHashMap，在简单场景中可以通过HashMap实现简单数据缓存，但如果要实现缓存随时间改变、存储的数据空间可控则缓存工具还是很有必要的。Cache存储的是键值对的集合，不同时是还需要处理缓存过期、动态加载等算法逻辑，需要额外信息实现这些操作，对此根据面向对象的思想，还需要做方法与数据的关联性封装，主要实现的缓存功能有：自动将节点加载至缓存结构中，当缓存的数据超过最大值时，使用LRU算法替换；它具备根据节点上一次被访问或写入时间计算缓存过期机制，缓存的key被封装在WeakReference引用中，缓存的value被封装在WeakReference或SoftReference引用中；还可以统计缓存使用过程中的命中率、异常率和命中率等统计数据。

#### 构建缓存对象[#](https://www.cnblogs.com/rickiyang/p/11074159.html#2694015556)

我们先看一个示例，再来讲解使用方式：

```java
package com.rickiyang.learn.cache;

import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Random;
import java.util.concurrent.TimeUnit;

/**
 * @author: rickiyang
 * @date: 2019/6/12
 * @description:
 */
public class GuavaCacheService {

    public void setCache() {
        LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
                //设置并发级别为8，并发级别是指可以同时写缓存的线程数
                .concurrencyLevel(8)
                //设置缓存容器的初始容量为10
                .initialCapacity(10)
                //设置缓存最大容量为100，超过100之后就会按照LRU最近虽少使用算法来移除缓存项
                .maximumSize(100)
                //是否需要统计缓存情况,该操作消耗一定的性能,生产环境应该去除
                .recordStats()
                //设置写缓存后n秒钟过期
                .expireAfterWrite(60, TimeUnit.SECONDS)
                //设置读写缓存后n秒钟过期,实际很少用到,类似于expireAfterWrite
                //.expireAfterAccess(17, TimeUnit.SECONDS)
                //只阻塞当前数据加载线程，其他线程返回旧值
                //.refreshAfterWrite(13, TimeUnit.SECONDS)
                //设置缓存的移除通知
                .removalListener(notification -> {
                    System.out.println(notification.getKey() + " " + notification.getValue() + " 被移除,原因:" + notification.getCause());
                })
                //build方法中可以指定CacheLoader，在缓存不存在时通过CacheLoader的实现自动加载缓存
                .build(new DemoCacheLoader());

        //模拟线程并发
        new Thread(() -> {
            //非线程安全的时间格式化工具
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("HH:mm:ss");
            try {
                for (int i = 0; i < 10; i++) {
                    String value = cache.get(1);
                    System.out.println(Thread.currentThread().getName() + " " + simpleDateFormat.format(new Date()) + " " + value);
                    TimeUnit.SECONDS.sleep(3);
                }
            } catch (Exception ignored) {
            }
        }).start();

        new Thread(() -> {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("HH:mm:ss");
            try {
                for (int i = 0; i < 10; i++) {
                    String value = cache.get(1);
                    System.out.println(Thread.currentThread().getName() + " " + simpleDateFormat.format(new Date()) + " " + value);
                    TimeUnit.SECONDS.sleep(5);
                }
            } catch (Exception ignored) {
            }
        }).start();
        //缓存状态查看
        System.out.println(cache.stats().toString());

    }

    /**
     * 随机缓存加载,实际使用时应实现业务的缓存加载逻辑,例如从数据库获取数据
     */
    public static class DemoCacheLoader extends CacheLoader<Integer, String> {
        @Override
        public String load(Integer key) throws Exception {
            System.out.println(Thread.currentThread().getName() + " 加载数据开始");
            TimeUnit.SECONDS.sleep(8);
            Random random = new Random();
            System.out.println(Thread.currentThread().getName() + " 加载数据结束");
            return "value:" + random.nextInt(10000);
        }
    }
}
```

上面一段代码展示了如何使用Cache创建一个缓存对象并使用它。

LoadingCache是Cache的子接口，相比较于Cache，当从LoadingCache中读取一个指定key的记录时，如果该记录不存在，则LoadingCache可以自动执行加载数据到缓存的操作。

在调用CacheBuilder的build方法时，必须传递一个CacheLoader类型的参数，CacheLoader的load方法需要我们提供实现。当调用LoadingCache的get方法时，如果缓存不存在对应key的记录，则CacheLoader中的load方法会被自动调用从外存加载数据，load方法的返回值会作为key对应的value存储到LoadingCache中，并从get方法返回。

当然如果你不想指定重建策略，那么你可以使用无参的build()方法，它将返回Cache类型的构建对象。

CacheBuilder 是Guava 提供的一个快速构建缓存对象的工具类。CacheBuilder类采用builder设计模式，它的每个方法都返回CacheBuilder本身，直到build方法被调用。 该类中提供了很多的参数设置选项，你可以设置cache的默认大小，并发数，存活时间，过期策略等等。

#### 可选配置分析[#](https://www.cnblogs.com/rickiyang/p/11074159.html#451324657)

##### 缓存的并发级别

Guava提供了设置并发级别的api，使得缓存支持并发的写入和读取。同 ConcurrentHashMap 类似Guava cache的并发也是通过分离锁实现。在一般情况下，将并发级别设置为服务器cpu核心数是一个比较不错的选择。

```java
CacheBuilder.newBuilder()
		// 设置并发级别为cpu核心数
		.concurrencyLevel(Runtime.getRuntime().availableProcessors()) 
		.build();
```

##### 缓存的初始容量设置

我们在构建缓存时可以为缓存设置一个合理大小初始容量，由于Guava的缓存使用了分离锁的机制，扩容的代价非常昂贵。所以合理的初始容量能够减少缓存容器的扩容次数。

```java
CacheBuilder.newBuilder()
		// 设置初始容量为100
		.initialCapacity(100)
		.build();
```

##### 设置最大存储

Guava Cache可以在构建缓存对象时指定缓存所能够存储的最大记录数量。当Cache中的记录数量达到最大值后再调用put方法向其中添加对象，Guava会先从当前缓存的对象记录中选择一条删除掉，腾出空间后再将新的对象存储到Cache中。

1. **基于容量的清除(size-based eviction):** 通过CacheBuilder.maximumSize(long)方法可以设置Cache的最大容量数，当缓存数量达到或接近该最大值时，Cache将清除掉那些最近最少使用的缓存;
2. **基于权重的清除: ** 使用CacheBuilder.weigher(Weigher)指定一个权重函数，并且用CacheBuilder.maximumWeight(long)指定最大总重。比如每一项缓存所占据的内存空间大小都不一样，可以看作它们有不同的“权重”（weights）。

##### 缓存清除策略

###### 1. 基于存活时间的清除

- expireAfterWrite 写缓存后多久过期
- expireAfterAccess 读写缓存后多久过期
- refreshAfterWrite 写入数据后多久过期,只阻塞当前数据加载线程,其他线程返回旧值

这几个策略时间可以单独设置,也可以组合配置。

###### 2. 上面提到的基于容量的清除

###### 3. 显式清除

任何时候，你都可以显式地清除缓存项，而不是等到它被回收，Cache接口提供了如下API：

1. 个别清除：Cache.invalidate(key)
2. 批量清除：Cache.invalidateAll(keys)
3. 清除所有缓存项：Cache.invalidateAll()

###### 4. 基于引用的清除（Reference-based Eviction）

在构建Cache实例过程中，通过设置使用弱引用的键、或弱引用的值、或软引用的值，从而使JVM在GC时顺带实现缓存的清除，不过一般不轻易使用这个特性。

- CacheBuilder.weakKeys()：使用弱引用存储键。当键没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式，使用弱引用键的缓存用而不是equals比较键。
- CacheBuilder.weakValues()：使用弱引用存储值。当值没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式，使用弱引用值的缓存用而不是equals比较值。
- CacheBuilder.softValues()：使用软引用存储值。软引用只有在响应内存需要时，才按照全局最近最少使用的顺序回收。考虑到使用软引用的性能影响，我们通常建议使用更有性能预测性的缓存大小限定（见上文，基于容量回收）。使用软引用值的缓存同样用==而不是equals比较值。

##### 清理什么时候发生

也许这个问题有点奇怪，如果设置的存活时间为一分钟，难道不是一分钟后这个key就会立即清除掉吗？我们来分析一下如果要实现这个功能，那Cache中就必须存在线程来进行周期性地检查、清除等工作，很多cache如redis、ehcache都是这样实现的。

使用CacheBuilder构建的缓存不会”自动”执行清理和回收工作，也不会在某个缓存项过期后马上清理，也没有诸如此类的清理机制。相反，它会在写操作时顺带做少量的维护工作，或者偶尔在读操作时做——如果写操作实在太少的话。

这样做的原因在于：如果要自动地持续清理缓存，就必须有一个线程，这个线程会和用户操作竞争共享锁。此外，某些环境下线程创建可能受限制，这样CacheBuilder就不可用了。参考如下示例：

```java
package com.rickiyang.learn.cache;

import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.TimeUnit;

/**
 * @author: rickiyang
 * @date: 2019/6/12
 * @description:
 */
public class GuavaCacheService {


    static Cache<Integer, String> cache = CacheBuilder.newBuilder()
            .expireAfterWrite(5, TimeUnit.SECONDS)
            .build();

    public static void main(String[] args) throws Exception {
        new Thread(() -> {
            while (true) {
                SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
                System.out.println(sdf.format(new Date()) + " size: " + cache.size());
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {

                }
            }
        }).start();
        SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
        cache.put(1, "a");
        System.out.println("写入 key:1 ,value:" + cache.getIfPresent(1));
        Thread.sleep(10000);
        cache.put(2, "b");
        System.out.println("写入 key:2 ,value:" + cache.getIfPresent(2));
        Thread.sleep(10000);
        System.out.println(sdf.format(new Date())
                + " sleep 10s , key:1 ,value:" + cache.getIfPresent(1));
        System.out.println(sdf.format(new Date())
                + " sleep 10s, key:2 ,value:" + cache.getIfPresent(2));
    }
}

部分输出结果：
23:57:36 size: 0
写入 key:1 ,value:a
23:57:38 size: 1
23:57:40 size: 1
23:57:42 size: 1
23:57:44 size: 1
23:57:46 size: 1
写入 key:2 ,value:b
23:57:48 size: 1
23:57:50 size: 1
23:57:52 size: 1
23:57:54 size: 1
23:57:56 size: 1
23:57:56 sleep 10s , key:1 ,value:null
23:57:56 sleep 10s, key:2 ,value:null
23:57:58 size: 0
23:58:00 size: 0
23:58:02 size: 0
    ...
    ...
```

上面程序设置了缓存过期时间为5S，每打印一次当前的size需要2S，打印了5次size之后写入key 2，此时的size为1，说明在这个时候才把第一次应该过期的key 1给删除。

**给移除操作添加一个监听器：**

可以为Cache对象添加一个移除监听器，这样当有记录被删除时可以感知到这个事件。

```java
RemovalListener<String, String> listener = notification -> System.out.println("[" + notification.getKey() + ":" + notification.getValue() + "] is removed!");
        Cache<String,String> cache = CacheBuilder.newBuilder()
                .maximumSize(5)
                .removalListener(listener)
                .build();
```

**但是要注意的是：**

**默认情况下，监听器方法是在移除缓存时同步调用的。因为缓存的维护和请求响应通常是同时进行的，代价高昂的监听器方法在同步模式下会拖慢正常的缓存请求。在这种情况下，你可以使用`RemovalListeners.asynchronous(RemovalListener, Executor)`把监听器装饰为异步操作。**

##### 自动加载

上面我们说过使用get方法的时候如果key不存在你可以使用指定方法去加载这个key。在Cache构建的时候通过指定CacheLoder的方式。如果你没有指定，你也可以在get的时候显式的调用call方法来设置key不存在的补救策略。

Cache的get方法有两个参数，第一个参数是要从Cache中获取记录的key，第二个记录是一个Callable对象。

当缓存中已经存在key对应的记录时，get方法直接返回key对应的记录。如果缓存中不包含key对应的记录，Guava会启动一个线程执行Callable对象中的call方法，call方法的返回值会作为key对应的值被存储到缓存中，并且被get方法返回。

```java
package com.rickiyang.learn.cache;

import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;

/**
 * @author: rickiyang
 * @date: 2019/6/12
 * @description:
 */
public class GuavaCacheService {


    private static Cache<String, String> cache = CacheBuilder.newBuilder()
            .maximumSize(3)
            .build();

    public static void main(String[] args) {

        new Thread(() -> {
            System.out.println("thread1");
            try {
                String value = cache.get("key", new Callable<String>() {
                    public String call() throws Exception {
                        System.out.println("thread1"); //加载数据线程执行标志
                        Thread.sleep(1000); //模拟加载时间
                        return "thread1";
                    }
                });
                System.out.println("thread1 " + value);
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }).start();
        new Thread(() -> {
            System.out.println("thread2");
            try {
                String value = cache.get("key", new Callable<String>() {
                    public String call() throws Exception {
                        System.out.println("thread2"); //加载数据线程执行标志
                        Thread.sleep(1000); //模拟加载时间
                        return "thread2";
                    }
                });
                System.out.println("thread2 " + value);
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }).start();
    }

}

输出结果为：
thread1
thread2
thread2
thread1 thread2
thread2 thread2
```

可以看到输出结果：两个线程都启动，输出thread1，thread2，接着又输出了thread2，说明进入了thread2的call方法了，此时thread1正在阻塞，等待key被设置。然后thread1 得到了value是thread2，thread2的结果自然也是thread2。

这段代码中有两个线程共享同一个Cache对象，两个线程同时调用get方法获取同一个key对应的记录。由于key对应的记录不存在，所以两个线程都在get方法处阻塞。此处在call方法中调用Thread.sleep(1000)模拟程序从外存加载数据的时间消耗。

从结果中可以看出，虽然是两个线程同时调用get方法，但只有一个get方法中的Callable会被执行(没有打印出load2)。Guava可以保证当有多个线程同时访问Cache中的一个key时，如果key对应的记录不存在，Guava只会启动一个线程执行get方法中Callable参数对应的任务加载数据存到缓存。当加载完数据后，任何线程中的get方法都会获取到key对应的值。

##### 统计信息

可以对Cache的命中率、加载数据时间等信息进行统计。在构建Cache对象时，可以通过CacheBuilder的recordStats方法开启统计信息的开关。开关开启后Cache会自动对缓存的各种操作进行统计，调用Cache的stats方法可以查看统计后的信息。

```java
package com.rickiyang.learn.cache;

import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

/**
 * @author: rickiyang
 * @date: 2019/6/12
 * @description:
 */
public class GuavaCacheService {


    public static void main(String[] args) {
        Cache<String, String> cache = CacheBuilder.newBuilder()
                .maximumSize(3)
                .recordStats() //开启统计信息开关
                .build();
        cache.put("1", "v1");
        cache.put("2", "v2");
        cache.put("3", "v3");
        cache.put("4", "v4");

        cache.getIfPresent("1");
        cache.getIfPresent("2");
        cache.getIfPresent("3");
        cache.getIfPresent("4");
        cache.getIfPresent("5");
        cache.getIfPresent("6");

        System.out.println(cache.stats()); //获取统计信息
    }

}

输出：
CacheStats{hitCount=3, missCount=3, loadSuccessCount=0, loadExceptionCount=0, totalLoadTime=0, evictionCount=1}
```

## Guava Cache源码浅析

# 1. 简介

Guava Cache是指在JVM的内存中缓存数据，相比较于传统的数据库或redis存储，访问内存中的数据会更加高效，无网络开销。

根据Guava官网介绍，下面的这几种情况可以考虑使用Guava Cache：

\1. 愿意消耗一些内存空间来提升速度。

\2. 预料到某些键会被多次查询。

\3. 缓存中存放的数据总量不会超出内存容量。

因此，Guava Cache特别适合存储那些**访问量大、不经常变化、数据量不是很大**的数据，以改善程序性能。

# 2. 类图

![img](https://img2020.cnblogs.com/blog/319908/202111/319908-20211122220855791-1659559415.png)

 

Guava Cache的类图中，主要涉及了5个类：CacheBuilder、LocalCache、Segment、EntryFactory和ReferenceEntry，大部分业务逻辑都在前面三个类，依次介绍如下：

## 2.1 CacheBuilder

CacheBuilder是一个用于构建Cache的类，是建造者模式的一个例子，主要的方法有：

- maximumSize(long maximumSize): 设置缓存存储的所有元素的最大个数。
- maximumWeight(long maximumWeight): 设置缓存存储的所有元素的最大权重。
- expireAfterAccess(long duration, TimeUnit unit): 设置元素在最后一次访问多久后过期。
- expireAfterWrite(long duration, TimeUnit unit): 设置元素在写入缓存后多久过期。
- concurrencyLevel(int concurrencyLevel): 设置并发水平，即允许多少线程无冲突的访问Cache，默认值是4，该值越大，LocalCache中的segment数组也会越大，访问效率越高，当然空间占用也大一些。
- removalListener(RemovalListener<? super K1, ? super V1> listener): 设置元素删除通知器，在任意元素无论何种原因被删除时会调用该通知器。
- setKeyStrength(Strength strength): 设置元素的key是强引用，还是弱引用，默认强引用，并且该属性也指定了EntryFactory使用是强引用还是弱引用。
- setValueStrength(Strength strength) : 设置元素的value是强引用，还是弱引用，默认强引用。

## 2.2 LocalCache

 LocalCache是一个支持并发访问的Hash Map，它实现了ConcurrentMap，其**内部会持有一个segment数组**，元素的增删改查都是通过调用segment的对应方法来实现的，

其主要的方法有:

- get(Object key): 查询一个key，内部实现是调用了Segment的get方法。
- public V put(K key, V value): 添加一个对象到cache中，内部实现是调用了Segment的put方法。
- remove(Object key) : 删除一个key，内部实现是调用了Segment的remove方法。
- replace(K key, V value)：更新一个key，内部实现是调用了Segment的update方法。

## 2.3 Segment

 segment是实际元素的持有者，它**内部持有一个table数组**，数组的每个元素又对应一个链表，链表上则保存了实际的元素，它的主要方法对应LocalCache提供的增删改查的接口，这里就不再啰嗦了。

## 2.4 EntryFactory

 EntryFactory是entry的创建工厂，可支持创建强引用、弱引用、强读引用、强写引用、强读写引用、弱读引用、弱写引用、弱读写引用等类型的元素。

强引用和弱引用就是java四种引用类型里面的强弱引用，默认是强引用，而读引用是指创建的元素会记录最后一次的访问时间，如果用户在CahceBuilder中调用了expireAfterAccess或者maximumWeight则会使用读引用类型的工厂，写引用类型也是同样的逻辑。

## 2.5 ReferenceEntry

 ReferenceEntry是元素的接口定义，它的实现类就是EntryFactory中创建的元素，包含了8种类型的元素，元素中至少包含了key、value和hash三个字段，其中hash是当前元素的hash值，如果是读引用则会多一个accessTime字段，以强引用的构造方法为例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static class StrongEntry<K, V> extends AbstractReferenceEntry<K, V> {
    final K key;

    StrongEntry(K key, int hash, @Nullable ReferenceEntry<K, V> next) {
      this.key = key;
      this.hash = hash;
      this.next = next;
    }

    @Override
    public K getKey() {
      return this.key;
    }

    // The code below is exactly the same for each entry type.

    final int hash;
    final @Nullable ReferenceEntry<K, V> next;
    volatile ValueReference<K, V> valueReference = unset();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

强读引用的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 StrongAccessEntry(K key, int hash, @Nullable ReferenceEntry<K, V> next) {
      super(key, hash, next); // 继承了StrongEntry，并多了accessTime
    }

    // The code below is exactly the same for each access entry type.

    volatile long accessTime = Long.MAX_VALUE;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 2.6 LocalCache示例

上面对LoacheCache所涉及的主要的类都做了介绍，下面画一张示例图给个直观感受，该例子中的Cache中包含的segment数组大小为4(默认值是4)，第二个segment的table数组大小为4，其中第二个table中的链表中有3个元素（简便起见，其他segment和table中的元素就不画了），

![img](https://img2020.cnblogs.com/blog/319908/202111/319908-20211122220912982-1233207682.png)

 

 

# 3. 主要方法

上面介绍了几个主要的类，下面从使用者的角度来把这几个类串联起来，主要包含了：创建Cache、添加对象、访问对象和删除对象。

## 3.1 创建Cache

 创建一个Cache的实现代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
      .maximumSize(10000) // 最大元素个数
      .expireAfterWrite(Duration.ofMinutes(10)) // 元素写入10分钟后期
      .removalListener(MY_LISTENER)  // 自定义的一个监听器
      .build(
          new CacheLoader<Key, Graph>() {  // 元素加载器，当查询元素不存在时，会自动调用该方法进行加载，然后再返回元素
            public Graph load(Key key) throws AnyException {
              return createExpensiveGraph(key);
            }
         });
 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

## 3.2 添加元素

添加元素访问的是LocalCache的put方法(**注意这个方法是没有锁的**)，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
  public V put(K key, V value) {
    checkNotNull(key);
    checkNotNull(value);
    int hash = hash(key);   // 首先计算key的hash值，并根据hash选定segment，再调用segment的put方法
    return segmentFor(hash).put(key, hash, value, false);
  }

/**
   * Returns the segment that should be used for a key with the given hash.
   *
   * @param hash the hash code for the key
   * @return the segment
   */
  Segment<K, V> segmentFor(int hash) {
    // 
    return segments[(hash >>> segmentShift) & segmentMask];
  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

再看下segment中的put方法(**注意这个方法是有锁的**)：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Nullable
    V put(K key, int hash, V value, boolean onlyIfAbsent) {
      lock();  // 一开始先加锁
      try {
        long now = map.ticker.read(); // 当前时间，单位纳秒
        preWriteCleanup(now); // 删除过期元素

        int newCount = this.count + 1;
        if (newCount > this.threshold) { // 必要时先扩容
          expand();
          newCount = this.count + 1;
        }

        //  根据hash再定位在table中的位置
        AtomicReferenceArray<ReferenceEntry<K, V>> table = this.table;
        int index = hash & (table.length() - 1);
        // 取得table中对应位置的链表的首个元素
        ReferenceEntry<K, V> first = table.get(index);

        // 遍历该链表，如果已在链表中则更新值.
        for (ReferenceEntry<K, V> e = first; e != null; e = e.getNext()) {
          K entryKey = e.getKey();
          if (e.getHash() == hash
              && entryKey != null
              && map.keyEquivalence.equivalent(key, entryKey)) {
            // We found an existing entry.

            ValueReference<K, V> valueReference = e.getValueReference();
            V entryValue = valueReference.get();

            if (entryValue == null) {
              ++modCount;
              if (valueReference.isActive()) {
                enqueueNotification(
                    key, hash, entryValue, valueReference.getWeight(), RemovalCause.COLLECTED);
                setValue(e, key, value, now);
                newCount = this.count; // count remains unchanged
              } else {
                setValue(e, key, value, now);
                newCount = this.count + 1;
              }
              this.count = newCount; // write-volatile
              evictEntries(e);
              return null;
            } else if (onlyIfAbsent) {
              // Mimic
              // "if (!map.containsKey(key)) ...
              // else return map.get(key);
              recordLockedRead(e, now);
              return entryValue;
            } else {
              // clobber existing entry, count remains unchanged
              ++modCount;
              enqueueNotification(
                  key, hash, entryValue, valueReference.getWeight(), RemovalCause.REPLACED);
              setValue(e, key, value, now);
              evictEntries(e);
              return entryValue;
            }
          }
        }

        // 在链表中未找到，则创建一个新的元素，并添加在链表的头部，即2.6章节示例中的table[1]和entry1之间.
        ++modCount;// 链表更新操作次数加1
        ReferenceEntry<K, V> newEntry = newEntry(key, hash, first);
        setValue(newEntry, key, value, now);
        table.set(index, newEntry);// 添加到链表头部
        newCount = this.count + 1;
        this.count = newCount; // segment内的元素个数加1
        evictEntries(newEntry);
        return null;
      } finally {
        unlock();
        postWriteCleanup(); // 前面删除元素时，会把删除通知加入到队列中，在这里遍历删除通知队列并发出通知
      }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

添加方法的代码如上所示，重点有两个地方：

\1. LocalCache的put方法中是不加锁的，而Segment中的put方法是加锁的，因此在访问量很大的时候，可以通过提高concurrencyLevel的值来提高segment数组大小，减少锁冲突。

\2. 在执行put方法时，会**“顺便”**执行清理操作，删除过期的元素，因为Guava Cache没有后台线程，因此删除操作是在每次的put操作和一定次数的read操作时执行的，且清理的是当前segment的过期元素，这也告诉我们过期的元素并不是立即被删除的，即内存不是立即释放的，会随着我们的读写操作来释放的，当然**如果Guava Cache本身访问量不大，导致累积了大量过期元素后，再来访问可能会有较大的访问耗时**。

## 3.3 访问元素

 访问元素访问的是LocalCache的get方法(**注意这个方法是没有锁的**)，代码如下:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public @Nullable V getIfPresent(Object key) {    // 和put一样，先对key做hash，再定位segment，然后调用get访问
    int hash = hash(checkNotNull(key));
    V value = segmentFor(hash).get(key, hash);
    if (value == null) {
      globalStatsCounter.recordMisses(1);
    } else {
      globalStatsCounter.recordHits(1);
    }
    return value;
  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

继续看segment的get方法(**注意这个方法是没有锁的**)：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Nullable
    V get(Object key, int hash) {
      try {
        if (count != 0) { // read-volatile
          long now = map.ticker.read();
         // 查询存活的元素
          ReferenceEntry<K, V> e = getLiveEntry(key, hash, now);
          if (e == null) {
            return null;
          }

          V value = e.getValueReference().get();
          if (value != null) {
            recordRead(e, now);
            // 检查是否需要刷新元素
            return scheduleRefresh(e, e.getKey(), hash, value, now, map.defaultLoader);
          }
         // 删除非强引用的队列，包含key队列和value队列
          tryDrainReferenceQueues();
        }
        return null;
      } finally {
        postReadCleanup();// 检查是否有过期元素待删除
      }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面再看下getLiveEntry和postReadCleanup方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Nullable
    ReferenceEntry<K, V> getLiveEntry(Object key, int hash, long now) {
      ReferenceEntry<K, V> e = getEntry(key, hash);
      if (e == null) {
        return null;
      } else if (map.isExpired(e, now)) { // 检查元素是否过期
        tryExpireEntries(now);
        return null;
      }
      return e;
    }
@Nullable
    ReferenceEntry<K, V> getEntry(Object key, int hash) {
      // 根据hash定位table中位置的链表，并进行遍历，检查hash是否相等
      for (ReferenceEntry<K, V> e = getFirst(hash); e != null; e = e.getNext()) {
        if (e.getHash() != hash) {
          continue;
        }

        K entryKey = e.getKey();
        if (entryKey == null) { // 被垃圾回收期回收，清理引用队列
          tryDrainReferenceQueues();
          continue;
        }

        if (map.keyEquivalence.equivalent(key, entryKey)) {
          return e;
        }
      }

      return null;
    }
```

　　　　/** Returns first entry of bin for given hash. */
　　　　ReferenceEntry<K, V> getFirst(int hash) {
　　　　　　// read this volatile field only once 
　　　　　　AtomicReferenceArray<ReferenceEntry<K, V>> table = this.table;
　　　　　　return table.get(hash & (table.length() - 1));
　　　　}

```
 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
void postReadCleanup() {
     // DRAIN_THRESHOLD=63，即每读64次会执行一次清理操作
      if ((readCount.incrementAndGet() & DRAIN_THRESHOLD) == 0) {
        cleanUp();
      }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

读方法相对要简单一些，重点有两个地方：

\1. 查找到元素后检查是否过期，过期则删除，否则返回。

\2. put方法每次调用都执行清理方法，get方法每调用64次get方法，才会执行一次清理。

注意，前面示例中的CacheBuilder创建LocalCache时，添加了元素加载器，当get方法中发现元素不存在时

 

## 3.4 删除元素

 删掉元素是invalidate()接口，该接口最终调用了segment的remove方法实现，如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
V remove(Object key, int hash) {
      lock();  // 和put有些类似，先加锁，再搜索，然后从链表删除
      try {
        long now = map.ticker.read();
        preWriteCleanup(now);

        int newCount = this.count - 1;
        AtomicReferenceArray<ReferenceEntry<K, V>> table = this.table;
        int index = hash & (table.length() - 1);
        ReferenceEntry<K, V> first = table.get(index);

        for (ReferenceEntry<K, V> e = first; e != null; e = e.getNext()) {
          K entryKey = e.getKey();
          if (e.getHash() == hash
              && entryKey != null
              && map.keyEquivalence.equivalent(key, entryKey)) {
            ValueReference<K, V> valueReference = e.getValueReference();
            V entryValue = valueReference.get();

            RemovalCause cause;
            if (entryValue != null) {
              cause = RemovalCause.EXPLICIT;
            } else if (valueReference.isActive()) {
              cause = RemovalCause.COLLECTED;
            } else {
              // currently loading
              return null;
            }

            ++modCount;
            // 删除方法有些特别，看下面分析
            ReferenceEntry<K, V> newFirst =
                removeValueFromChain(first, e, entryKey, hash, entryValue, valueReference, cause);
            newCount = this.count - 1;
            table.set(index, newFirst);
            this.count = newCount; // write-volatile
            return entryValue;
          }
        }

        return null;
      } finally {
        unlock();
        postWriteCleanup();
      }
    }

// removeValueFromChain调用了removeEntryFromChain
@GuardedBy("this")
    @Nullable
    ReferenceEntry<K, V> removeEntryFromChain(
        ReferenceEntry<K, V> first, ReferenceEntry<K, V> entry) {
      int newCount = count;
      ReferenceEntry<K, V> newFirst = entry.getNext();
     // 删除元素时，没有直接从链表上面摘除，而是遍历first和entry之间的元素，并拷贝新建新的元素构建链表
      for (ReferenceEntry<K, V> e = first; e != entry; e = e.getNext()) {
        ReferenceEntry<K, V> next = copyEntry(e, newFirst);
        if (next != null) {
          newFirst = next;
        } else {
          removeCollectedEntry(e);
          newCount--;
        }
      }
      this.count = newCount;
      return newFirst;
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注意删除的时候，并没有直接从链表摘除，而是做了一次遍历新建了一个链表，举个例子：

![img](https://img2020.cnblogs.com/blog/319908/202111/319908-20211127221303610-1725644794.png)

 为什么要做一次遍历呢？先看一下StrongEntry的定义：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static class StrongEntry<K, V> extends AbstractReferenceEntry<K, V> {
    final K key;
    final int hash;
    final @Nullable ReferenceEntry<K, V> next;
    volatile ValueReference<K, V> valueReference = unset();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

key，hash和next都是final的，通过这种新建链表的方式，可以保证当前的并发读线程是能读到数据的**（读方法无锁）**，即使是过期的，这其实就是CopyOnWrite的思想。

# 4. 小结

从上面分析可以看出，guava cache是一款非常优秀的本地缓存组件，为了得到更好的效率，减少写操作锁冲突（读操作无锁），**可以将concurrencyLevel设置为当前CPU核数的2两倍**。

初始化代码如下：

```
Cache<String, Integer> lcache = CacheBuilder.newBuilder()
                  .maximumSize(100)
                  .concurrencyLevel(Runtime.getRuntime().availableProcessors()*2) // 当前CPU核数*2
.expireAfterWrite(30, TimeUnit.SECONDS) .build();
```

之后就可以通过put和getIfPresent来进行元素访问了，例如：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 赋值for(int i=0; i<10000; i++) {
    lcache.put(String.valueOf(i), i);
}
// 查询    
Integer value = lcache.getIfPresent("10");
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

https://github.com/tomliugen