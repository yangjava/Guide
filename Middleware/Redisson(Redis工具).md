# 原文：[Redisson分布式锁学习总结：可重入锁 RedissonLock#lock 获取锁源码分析](https://blog.csdn.net/Howinfun/article/details/121322125?spm=1001.2014.3001.5501)

# 一、RedissonLock#lock 源码分析

## 1、根据锁key计算出 slot，一个slot对应的是redis集群的一个节点

redisson 支持分布式锁的功能，基本都是基于 lua 脚本来完成的，因为分布式锁肯定是具有比较复杂的判断逻辑，而lua脚本可以保证复杂判断和复杂操作的原子性。

redisson 的 RedissonLock 执行lua脚本，需要先找到当前锁key需要存放到哪个slot，即在集群中哪个节点进行操作，后续不同客户端或不同线程再使用这个锁key进行上锁，也需要到对应的节点的slot中进行加锁操作。

执行lua脚本的源码：

```java
org.redisson.command.CommandAsyncService#evalWriteAsync(java.lang.String, org.redisson.client.codec.Codec, org.redisson.client.protocol.RedisCommand<T>, java.lang.String, java.util.List<java.lang.Object>, java.lang.Object...)


@Override
public <T, R> RFuture<R> evalWriteAsync(String key, Codec codec, RedisCommand<T> evalCommandType, String script, List<Object> keys, Object... params) {
    // 根据锁key找到对应的redis节点
    NodeSource source = getNodeSource(key);
    return evalAsync(source, false, codec, evalCommandType, script, keys, params);
}

private NodeSource getNodeSource(String key) {
    // 计算锁key对应的slot
    int slot = connectionManager.calcSlot(key);
    return new NodeSource(slot);
}
```

计算 slot 分主从模式和集群模式，我们一般生产环境都是使用集群模式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b449886d7b54b0683cebfb50f741115.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

```java
public static final int MAX_SLOT = 16384;

@Override
public int calcSlot(String key) {
    if (key == null) {
        return 0;
    }

    int start = key.indexOf('{');
    if (start != -1) {
        int end = key.indexOf('}');
        key = key.substring(start+1, end);
    }
    // 使用 CRC16 算法来计算 slot，其中 MAX_SLOT 就是 16384，redis集群规定最多有 16384 个slot。
    int result = CRC16.crc16(key.getBytes()) % MAX_SLOT;
    log.debug("slot {} for {}", result, key);
    return result;
}
```

## 2、RedissonLock 之 lua 脚本加锁

```java
RedissonLock#tryLockInnerAsync

<T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);

    return evalWriteAsync(getName(), LongCodec.INSTANCE, command,
            "if (redis.call('exists', KEYS[1]) == 0) then " +
                    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return nil; " +
                    "end; " +
                    "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return nil; " +
                    "end; " +
                    "return redis.call('pttl', KEYS[1]);",
            Collections.singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}
```

### 2.1、KEYS

Collections.singletonList(getName())

KEYS：["myLock"]

### 2.2、ARGVS

internalLockLeaseTime，getLockName(threadId)

internalLockLeaseTime：其实就是 watchdog 的超时时间，默认是30000毫秒 Config#lockWatchdogTimeout。

```java
private long lockWatchdogTimeout = 30 * 1000;
```

getLockName(threadId)：客户端ID(UUID):线程ID(threadId)

```java
protected String getLockName(long threadId) {
    return id + ":" + threadId;
}
```

ARGVS:[30000,"UUID:threadId"]

### 2.3、lua 脚本分析

#### 1、分支一：不存在加锁记录，获取锁成功

**lua脚本：**

```java
"if (redis.call('exists', KEYS[1]) == 0) then " +
    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
    "return nil; " +
"end; " +
```

**分析：**

1. 利用 exists 命令判断 myLock 这个 key 是否存在

   ```sh
   exists myLock
   ```

2. 如果不存在，则执行下面两个操作

   1. 执行一个map的操作，给指定key的值增加1

      ```sh
      hincrby myLock UUID:threadId
      ```

      执行后多了一个map数据结构：

      ```makefile
      myLock:{
          "UUID:threadId":1
      }
      ```

   2. 给 myLock 设置过期时间为30000毫秒

      ```sh
      expire myLock 30000
      ```

3. 最后返回nil，即null

#### 2、分支二：锁记录已存在，重复加锁

**lua脚本：**

```java
"if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
    "return nil; " +
"end; " +
```

**分析：**

1. 判断之前加锁的是否为当前客户端当前线程

   ```sh
   hexists myLock UUID:threadId
   ```

2. 如果存在，则将加锁次数增加1

   ```sh
   hincrby myLock UUID:threadId 1
   ```

   增加1后，map集合内容为：

   ```makefile
   myLock:{
       "UUID:threadId":2
   }
   ```

   > 利用map这个数据结构，存放加锁的客户端线程信息，从而支持可重入锁。

3. 重新刷新 myLock 的过期时间为30000毫秒

   ```sh
   expire myLock 30000
   ```

#### 3、分支三：获取锁失败，直接返回锁剩余过期时间

**lua脚本：**

```java
"return redis.call('pttl', KEYS[1]);"
```

**分析：**

1. 利用 pttl 命令获取锁剩余毫秒数

   ```sh
   pttl myLock
   ```

2. 返回步骤1获取的毫秒数

## 3、watchdog 不断为锁续命

因为我们是利用 lock() 方法获取锁的，没有指定多久后释放，但是 redisson 不可能真的不设置锁key的过期时间。

因为要考虑到一个场景：一个客户端成功获取锁，但是没有设置多久释放，如果redisson 在redis实例中设置锁的时候也没有设置过期时间，如果这个时候客户端所在的服务器挂掉了，那么他就不会执行到unlock() 方法去释放锁了，那么这个时候就会导致死锁，其他任何的客户端都获取不到锁。

所以 redisson 会有一个 watchdog 的角色，每隔10_000毫秒就会为锁续命，详细可看看下面截图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/96752a73dd8447a191b5d4fd8f16f22e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

再看看定时任务详细的设计：

```java
private void scheduleExpirationRenewal(long threadId) {
    ExpirationEntry entry = new ExpirationEntry();
    ExpirationEntry oldEntry = EXPIRATION_RENEWAL_MAP.putIfAbsent(getEntryName(), entry);
    if (oldEntry != null) {
        oldEntry.addThreadId(threadId);
    } else {
        // 一开始就是null，直接放入 EXPIRATION_RENEWAL_MAP 中
        entry.addThreadId(threadId);
        // 调用定时任务
        renewExpiration();
    }
}

private void renewExpiration() {
    // 上面已经传入，不为空
    ExpirationEntry ee = EXPIRATION_RENEWAL_MAP.get(getEntryName());
    if (ee == null) {
        return;
    }
    
    // 开启定时任务，时间是 internalLockLeaseTime / 3 毫秒后执行
    Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            // 判断是否存在 ExpirationEntry，只要加锁了，肯定存在
            ExpirationEntry ent = EXPIRATION_RENEWAL_MAP.get(getEntryName());
            if (ent == null) {
                return;
            }
            Long threadId = ent.getFirstThreadId();
            if (threadId == null) {
                return;
            }
            
            RFuture<Boolean> future = renewExpirationAsync(threadId);
            future.onComplete((res, e) -> {
                if (e != null) {
                    log.error("Can't update lock " + getName() + " expiration", e);
                    return;
                }
                
                if (res) {
                    // reschedule itself
                    // 循环调用
                    renewExpiration();
                }
            });
        }
    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
    
    ee.setTimeout(task);
}

protected RFuture<Boolean> renewExpirationAsync(long threadId) {
    return evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            // 判断 myLock map 中是否存在当前客户端当前线程
            myLock:{
                "UUID:threadId":1
            }
            "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                    // 存在，刷新过期时间，30_000毫秒
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return 1; " +
                    "end; " +
                    "return 0;",
            Collections.singletonList(getName()),
            internalLockLeaseTime, getLockName(threadId));
}
```

## 4、死循环获取锁

关于死循环获取锁，这里是抓大放小，没有深入研究里面比较细的点，只有自己大概的猜测。
代码看下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/73364d93f0f44bafbb5ac5984db766cf.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

如果获取锁失败，在进入死循环前，会订阅指定渠道：`redisson_lock__channel:{myLock}`，然后进入死循环。

在死循环里面，首先会先尝试再获取一遍锁，因为可能之前获取锁的客户端刚好释放锁了。如果获取失败，那么就进入等待状态，等待时间是获取锁失败时返回的锁key的ttl。

> 订阅指定channel猜测：因为在客户端释放锁的时候，会往这个channel发送消息；因此可以利用此消息来提前让等待的线程被唤醒去尝试获取锁，因为此时锁已经被释放了。

## 5、其他的加锁方式

如果我们需要指定获取锁成功后持有锁的时长，可以执行下面方法，指定 leaseTime

```java
lock.lock(10, TimeUnit.SECONDS);
```

> 如果指定了 leaseTime，watchdog就不会再启用了。

如果不但需要指定持有锁的时长，还想避免锁获取失败时的死循环，可以同时指定 leaseTime 和 waitTime

```java
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
```

> 如果指定了 waitTime，只会在 waitTime 时间内循环尝试获取锁，超过 waitTime 如果还是获取失败，直接返回false。

# 原文链接：[Redisson分布式锁学习总结：可重入锁 RedissonLock#unlock 释放锁源码分析](https://blog.csdn.net/Howinfun/article/details/121322497?spm=1001.2014.3001.5501)

# 一、RedissonLock#lock 源码分析

## 1、根据锁key计算出 slot，一个slot对应的是redis集群的一个节点

redisson 支持分布式锁的功能，基本都是基于 lua 脚本来完成的，因为分布式锁肯定是具有比较复杂的判断逻辑，而lua脚本可以保证复杂判断和复杂操作的原子性。

redisson 的 RedissonLock 执行lua脚本，需要先找到当前锁key需要存放到哪个slot，即在集群中哪个节点进行操作，后续不同客户端或不同线程再使用这个锁key进行上锁，也需要到对应的节点的slot中进行加锁操作。

执行lua脚本的源码：

```java
org.redisson.command.CommandAsyncService#evalWriteAsync(java.lang.String, org.redisson.client.codec.Codec, org.redisson.client.protocol.RedisCommand<T>, java.lang.String, java.util.List<java.lang.Object>, java.lang.Object...)


@Override
public <T, R> RFuture<R> evalWriteAsync(String key, Codec codec, RedisCommand<T> evalCommandType, String script, List<Object> keys, Object... params) {
    // 根据锁key找到对应的redis节点
    NodeSource source = getNodeSource(key);
    return evalAsync(source, false, codec, evalCommandType, script, keys, params);
}

private NodeSource getNodeSource(String key) {
    // 计算锁key对应的slot
    int slot = connectionManager.calcSlot(key);
    return new NodeSource(slot);
}
```

计算 slot 分主从模式和集群模式，我们一般生产环境都是使用集群模式。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b449886d7b54b0683cebfb50f741115.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

```java
public static final int MAX_SLOT = 16384;

@Override
public int calcSlot(String key) {
    if (key == null) {
        return 0;
    }

    int start = key.indexOf('{');
    if (start != -1) {
        int end = key.indexOf('}');
        key = key.substring(start+1, end);
    }
    // 使用 CRC16 算法来计算 slot，其中 MAX_SLOT 就是 16384，redis集群规定最多有 16384 个slot。
    int result = CRC16.crc16(key.getBytes()) % MAX_SLOT;
    log.debug("slot {} for {}", result, key);
    return result;
}
```

## 2、RedissonLock 之 lua 脚本加锁

```java
RedissonLock#tryLockInnerAsync

<T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);

    return evalWriteAsync(getName(), LongCodec.INSTANCE, command,
            "if (redis.call('exists', KEYS[1]) == 0) then " +
                    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return nil; " +
                    "end; " +
                    "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return nil; " +
                    "end; " +
                    "return redis.call('pttl', KEYS[1]);",
            Collections.singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}
```

### 2.1、KEYS

Collections.singletonList(getName())

KEYS：["myLock"]

### 2.2、ARGVS

internalLockLeaseTime，getLockName(threadId)

internalLockLeaseTime：其实就是 watchdog 的超时时间，默认是30000毫秒 Config#lockWatchdogTimeout。

```java
private long lockWatchdogTimeout = 30 * 1000;
```

getLockName(threadId)：客户端ID(UUID):线程ID(threadId)

```java
protected String getLockName(long threadId) {
    return id + ":" + threadId;
}
```

ARGVS:[30000,"UUID:threadId"]

### 2.3、lua 脚本分析

#### 1、分支一：不存在加锁记录，获取锁成功

**lua脚本：**

```java
"if (redis.call('exists', KEYS[1]) == 0) then " +
    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
    "return nil; " +
"end; " +
```

**分析：**

1. 利用 exists 命令判断 myLock 这个 key 是否存在

   ```sh
   exists myLock
   ```

2. 如果不存在，则执行下面两个操作

   1. 执行一个map的操作，给指定key的值增加1

      ```sh
      hincrby myLock UUID:threadId
      ```

      执行后多了一个map数据结构：

      ```makefile
      myLock:{
          "UUID:threadId":1
      }
      ```

   2. 给 myLock 设置过期时间为30000毫秒

      ```sh
      expire myLock 30000
      ```

3. 最后返回nil，即null

#### 2、分支二：锁记录已存在，重复加锁

**lua脚本：**

```java
"if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
    "return nil; " +
"end; " +
```

**分析：**

1. 判断之前加锁的是否为当前客户端当前线程

   ```sh
   hexists myLock UUID:threadId
   ```

2. 如果存在，则将加锁次数增加1

   ```sh
   hincrby myLock UUID:threadId 1
   ```

   增加1后，map集合内容为：

   ```makefile
   myLock:{
       "UUID:threadId":2
   }
   ```

   > 利用map这个数据结构，存放加锁的客户端线程信息，从而支持可重入锁。

3. 重新刷新 myLock 的过期时间为30000毫秒

   ```sh
   expire myLock 30000
   ```

#### 3、分支三：获取锁失败，直接返回锁剩余过期时间

**lua脚本：**

```java
"return redis.call('pttl', KEYS[1]);"
```

**分析：**

1. 利用 pttl 命令获取锁剩余毫秒数

   ```sh
   pttl myLock
   ```

2. 返回步骤1获取的毫秒数

## 3、watchdog 不断为锁续命

因为我们是利用 lock() 方法获取锁的，没有指定多久后释放，但是 redisson 不可能真的不设置锁key的过期时间。

因为要考虑到一个场景：一个客户端成功获取锁，但是没有设置多久释放，如果redisson 在redis实例中设置锁的时候也没有设置过期时间，如果这个时候客户端所在的服务器挂掉了，那么他就不会执行到unlock() 方法去释放锁了，那么这个时候就会导致死锁，其他任何的客户端都获取不到锁。

所以 redisson 会有一个 watchdog 的角色，每隔10_000毫秒就会为锁续命，详细可看看下面截图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/96752a73dd8447a191b5d4fd8f16f22e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

再看看定时任务详细的设计：

```java
private void scheduleExpirationRenewal(long threadId) {
    ExpirationEntry entry = new ExpirationEntry();
    ExpirationEntry oldEntry = EXPIRATION_RENEWAL_MAP.putIfAbsent(getEntryName(), entry);
    if (oldEntry != null) {
        oldEntry.addThreadId(threadId);
    } else {
        // 一开始就是null，直接放入 EXPIRATION_RENEWAL_MAP 中
        entry.addThreadId(threadId);
        // 调用定时任务
        renewExpiration();
    }
}

private void renewExpiration() {
    // 上面已经传入，不为空
    ExpirationEntry ee = EXPIRATION_RENEWAL_MAP.get(getEntryName());
    if (ee == null) {
        return;
    }
    
    // 开启定时任务，时间是 internalLockLeaseTime / 3 毫秒后执行
    Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            // 判断是否存在 ExpirationEntry，只要加锁了，肯定存在
            ExpirationEntry ent = EXPIRATION_RENEWAL_MAP.get(getEntryName());
            if (ent == null) {
                return;
            }
            Long threadId = ent.getFirstThreadId();
            if (threadId == null) {
                return;
            }
            
            RFuture<Boolean> future = renewExpirationAsync(threadId);
            future.onComplete((res, e) -> {
                if (e != null) {
                    log.error("Can't update lock " + getName() + " expiration", e);
                    return;
                }
                
                if (res) {
                    // reschedule itself
                    // 循环调用
                    renewExpiration();
                }
            });
        }
    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);
    
    ee.setTimeout(task);
}

protected RFuture<Boolean> renewExpirationAsync(long threadId) {
    return evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            // 判断 myLock map 中是否存在当前客户端当前线程
            myLock:{
                "UUID:threadId":1
            }
            "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                    // 存在，刷新过期时间，30_000毫秒
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                    "return 1; " +
                    "end; " +
                    "return 0;",
            Collections.singletonList(getName()),
            internalLockLeaseTime, getLockName(threadId));
}
```

## 4、死循环获取锁

关于死循环获取锁，这里是抓大放小，没有深入研究里面比较细的点，只有自己大概的猜测。
代码看下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/73364d93f0f44bafbb5ac5984db766cf.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6YCB6Iqx55qE56iL5bqP54y_,size_20,color_FFFFFF,t_70,g_se,x_16)

如果获取锁失败，在进入死循环前，会订阅指定渠道：`redisson_lock__channel:{myLock}`，然后进入死循环。

在死循环里面，首先会先尝试再获取一遍锁，因为可能之前获取锁的客户端刚好释放锁了。如果获取失败，那么就进入等待状态，等待时间是获取锁失败时返回的锁key的ttl。

> 订阅指定channel猜测：因为在客户端释放锁的时候，会往这个channel发送消息；因此可以利用此消息来提前让等待的线程被唤醒去尝试获取锁，因为此时锁已经被释放了。

## 5、其他的加锁方式

如果我们需要指定获取锁成功后持有锁的时长，可以执行下面方法，指定 leaseTime

```java
lock.lock(10, TimeUnit.SECONDS);
```

> 如果指定了 leaseTime，watchdog就不会再启用了。

如果不但需要指定持有锁的时长，还想避免锁获取失败时的死循环，可以同时指定 leaseTime 和 waitTime

```java
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
```

> 如果指定了 waitTime，只会在 waitTime 时间内循环尝试获取锁，超过 waitTime 如果还是获取失败，直接返回false。

# 原文链接：[Redisson分布式锁学习总结：公平锁 RedissonFairLock#lock 获取锁源码分析](https://blog.csdn.net/Howinfun/article/details/121366238?spm=1001.2014.3001.5501)

# 一、RedissonFairLock#lock 源码分析

```java
public class RedissonFairLockDemo {

    public static void main(String[] args) {
        RedissonClient client = RedissonClientUtil.getClient("");
        RLock fairLock = client.getFairLock("myLock");
        // 最常见的使用方法
        try {
            fairLock.lock();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            fairLock.unlock();
        }
    }
}
```

## 1、根据锁key计算出 slot，一个slot对应的是redis集群的一个节点

RedissonFairLock 其实是 RedissonLock 的子类，它主要是基于 RedissonLock 做的扩展，主要扩展在于加锁和释放锁的地方，其他的逻辑都直接复用 RedissonLock：例如加锁前计算slot、watchdog机制等等。

## 2、RedissonFairLock 之 lua 脚本加锁

RedissonFairLock#tryLockInnerAsync：里面有两段 lua 脚本，我们现在只需要关注第二段即可。

```java
if (command == RedisCommands.EVAL_LONG) {
    return evalWriteAsync(getName(), LongCodec.INSTANCE, command,
            // remove stale threads
            "while true do " +
                "local firstThreadId2 = redis.call('lindex', KEYS[2], 0);" +
                "if firstThreadId2 == false then " +
                    "break;" +
                "end;" +

                "local timeout = tonumber(redis.call('zscore', KEYS[3], firstThreadId2));" +
                "if timeout <= tonumber(ARGV[4]) then " +
                    // remove the item from the queue and timeout set
                    // NOTE we do not alter any other timeout
                    "redis.call('zrem', KEYS[3], firstThreadId2);" +
                    "redis.call('lpop', KEYS[2]);" +
                "else " +
                    "break;" +
                "end;" +
            "end;" +

            // check if the lock can be acquired now
            "if (redis.call('exists', KEYS[1]) == 0) " +
                "and ((redis.call('exists', KEYS[2]) == 0) " +
                    "or (redis.call('lindex', KEYS[2], 0) == ARGV[2])) then " +

                // remove this thread from the queue and timeout set
                "redis.call('lpop', KEYS[2]);" +
                "redis.call('zrem', KEYS[3], ARGV[2]);" +

                // decrease timeouts for all waiting in the queue
                "local keys = redis.call('zrange', KEYS[3], 0, -1);" +
                "for i = 1, #keys, 1 do " +
                    "redis.call('zincrby', KEYS[3], -tonumber(ARGV[3]), keys[i]);" +
                "end;" +

                // acquire the lock and set the TTL for the lease
                "redis.call('hset', KEYS[1], ARGV[2], 1);" +
                "redis.call('pexpire', KEYS[1], ARGV[1]);" +
                "return nil;" +
            "end;" +

            // check if the lock is already held, and this is a re-entry
            "if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then " +
                "redis.call('hincrby', KEYS[1], ARGV[2],1);" +
                "redis.call('pexpire', KEYS[1], ARGV[1]);" +
                "return nil;" +
            "end;" +

            // the lock cannot be acquired
            // check if the thread is already in the queue
            "local timeout = redis.call('zscore', KEYS[3], ARGV[2]);" +
            "if timeout ~= false then " +
                // the real timeout is the timeout of the prior thread
                // in the queue, but this is approximately correct, and
                // avoids having to traverse the queue
                "return timeout - tonumber(ARGV[3]) - tonumber(ARGV[4]);" +
            "end;" +

            // add the thread to the queue at the end, and set its timeout in the timeout set to the timeout of
            // the prior thread in the queue (or the timeout of the lock if the queue is empty) plus the
            // threadWaitTime
            "local lastThreadId = redis.call('lindex', KEYS[2], -1);" +
            "local ttl;" +
            "if lastThreadId ~= false and lastThreadId ~= ARGV[2] then " +
                "ttl = tonumber(redis.call('zscore', KEYS[3], lastThreadId)) - tonumber(ARGV[4]);" +
            "else " +
                "ttl = redis.call('pttl', KEYS[1]);" +
            "end;" +
            "local timeout = ttl + tonumber(ARGV[3]) + tonumber(ARGV[4]);" +
            "if redis.call('zadd', KEYS[3], timeout, ARGV[2]) == 1 then " +
                "redis.call('rpush', KEYS[2], ARGV[2]);" +
            "end;" +
            "return ttl;",
            Arrays.asList(getName(), threadsQueueName, timeoutSetName),
            internalLockLeaseTime, getLockName(threadId), wait, currentTime);
}
```

lua 脚本虽然很长，但其实作者给的注释也是非常的清晰，让我们知道lua脚本每一步的含义，所以下面我将讲解每一个分支究竟利用redis命令做了什么。

### 2.1、KEYS

Arrays.asList(getName(), threadsQueueName, timeoutSetName)：

- getName(): 锁key
- threadsQueueName：prefixName("redisson_lock_queue", name)，用于锁排队
- timeoutSetName：prefixName("redisson_lock_timeout", name)，用于队列中每个客户端的等待超时时间

**KEYS：**["myLock","redisson_lock_queue:{myLock}","redisson_lock_timeout:{myLock}"]

### 2.2、ARGVS

internalLockLeaseTime, getLockName(threadId), wait, currentTime：

- internalLockLeaseTime：其实就是 watchdog 的超时时间，默认是30000毫秒，可看 Config#lockWatchdogTimeout。

  ```java
  private long lockWatchdogTimeout = 30 * 1000;
  ```

- getLockName(threadId)：return id + ":" + threadId，客户端ID(UUID):线程ID(threadId)

- wait：就是 threadWaitTime，默认30_0000毫秒

  ```java
  public RedissonFairLock(CommandAsyncExecutor commandExecutor, String name) {
      this(commandExecutor, name, 60000*5);
  }
  
  public RedissonFairLock(CommandAsyncExecutor commandExecutor, String name, long threadWaitTime) {
      super(commandExecutor, name);
      this.commandExecutor = commandExecutor;
      this.threadWaitTime = threadWaitTime;
      threadsQueueName = prefixName("redisson_lock_queue", name);
      timeoutSetName = prefixName("redisson_lock_timeout", name);
  }
  ```

- currentTime：当前时间时间戳

**ARGVS：**[30_000毫秒,"UUID:threadId",30_0000毫秒,当前时间戳]

### 2.3、lua 脚本分析

#### 1、分支一：清理过期的等待线程

**场景：**

这个死循环的作用主要用于清理过期的等待线程，主要避免下面场景，避免无效客户端占用等待队列资源

- 获取锁失败，然后进入等待队列，但是网络出现问题，那么后续很有可能就不能继续正常获取锁了。
- 获取锁失败，然后进入等待队列，但是之后客户端所在服务器宕机了。

```java
"while true do " +
    "local firstThreadId2 = redis.call('lindex', KEYS[2], 0);" +
    "if firstThreadId2 == false then " +
        "break;" +
    "end;" +

    "local timeout = tonumber(redis.call('zscore', KEYS[3], firstThreadId2));" +
    "if timeout <= tonumber(ARGV[4]) then " +
        // remove the item from the queue and timeout set
        // NOTE we do not alter any other timeout
        "redis.call('zrem', KEYS[3], firstThreadId2);" +
        "redis.call('lpop', KEYS[2]);" +
    "else " +
        "break;" +
    "end;" +
"end;" +
```

1. 开启死循环

2. 利用 lindex 命令判断等待队列中第一个元素是否存在，如果存在，直接跳出循环

   ```sh
   lidex redisson_lock_queue:{myLock} 0
   ```

3. 如果等待队列中第一个元素不为空（例如返回了LockName，即客户端UUID拼接线程ID），利用 zscore 在 超时记录集合(sorted set) 中获取对应的超时时间

   ```sh
   zscore redisson_lock_timeout:{myLock} UUID:threadId
   ```

4. 如果超时时间已经小于当前时间，那么首先从超时集合中移除该节点，接着也在等待队列中弹出第一个节点

   ```sh
   zrem redisson_lock_timeout:{myLock} UUID:threadId
   lpop redisson_lock_queue:{myLock}
   ```

5. 如果等待队列中的第一个元素还未超时，直接退出死循环

#### 2、分支二：检查是否可成功获取锁

**场景：**

- 其他客户端刚释放锁，并且等待队列为空
- 其他客户端刚释放锁，并且等待队列中的第一个元素就是当前客户端当前线程

```java
// check if the lock can be acquired now
"if (redis.call('exists', KEYS[1]) == 0) " +
    "and ((redis.call('exists', KEYS[2]) == 0) " +
        "or (redis.call('lindex', KEYS[2], 0) == ARGV[2])) then " +

    // remove this thread from the queue and timeout set
    "redis.call('lpop', KEYS[2]);" +
    "redis.call('zrem', KEYS[3], ARGV[2]);" +

    // decrease timeouts for all waiting in the queue
    "local keys = redis.call('zrange', KEYS[3], 0, -1);" +
    "for i = 1, #keys, 1 do " +
        "redis.call('zincrby', KEYS[3], -tonumber(ARGV[3]), keys[i]);" +
    "end;" +

    // acquire the lock and set the TTL for the lease
    "redis.call('hset', KEYS[1], ARGV[2], 1);" +
    "redis.call('pexpire', KEYS[1], ARGV[1]);" +
    "return nil;" +
"end;" +
```

1. 当前锁还未被获取 and（等待队列不存在 or 等待队列的第一个元素是当前客户端当前线程）

   ```sh
   exists myLock：判断锁是否存在
   
   exists redisson_lock_queue:{myLock}：判断等待队列是否为空
   
   lindex redisson_lock_timeout:{myLock} 0：获取等待队列中的第一个元素，用于判断是否等于当前客户端当前线程
   ```

2. 如果步骤1满足，从等待队列和超时集合中移除当前线程

   ```sh
   lpop redisson_lock_queue:{myLock}：弹出等待队列中的第一个元素，即当前线程
   
   zrem redisson_lock_timeout:{myLock} UUID:threadId：从超时集合中移除当前客户端当前线程
   ```

3. 刷新超时集合中，其他元素的超时时间，即更新他们得分数

   ```sh
   zrange redisson_lock_timeout:{myLock} 0 -1：从超时集合中获取所有的元素
   ```

   遍历，然后执行下面命令更新分数，即超时时间：

   ```sh
   zincrby redisson_lock_timeout:{myLock} -30w毫秒 keys[i]
   ```

   > 因为这里的客户端都是调用 lock()方法，就是等待直到最后获取到锁；所以某个客户端可以成功获取锁的时候，要帮其他等待的客户端刷新一下等待时间，不然在分支一的死循环中就被干掉了。

4. 最后，往加锁集合(map) myLock 中加入当前客户端当前线程，加锁次数为1，然后刷新 myLock 的过期时间，返回nil

   ```sh
   hset myLock UUID:threadId 1：将当前线程加入加锁记录中。
   espire myLock 3w毫秒：重置锁的过期时间。
   ```

   加入此节点后，map集合如下：

   ```makefile
   myLock:{
       "UUID:threadId":1
   }
   ```

   > 使用这个map记录加锁次数，主要用于支持可重入加锁。

#### 3、分支三：当前线程曾经获取锁，重复获取锁。

场景：

- 当前线程已经成功获取过锁，现在重新再次获取锁。
- 即：Redisson 的公平锁是支持可重入的。

```java
"if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then " +
    "redis.call('hincrby', KEYS[1], ARGV[2],1);" +
    "redis.call('pexpire', KEYS[1], ARGV[1]);" +
    "return nil;" +
"end;" +
```

1. 利用 hexists 命令判断加锁记录集合中，是否存在当前客户端当前线程

   ```sh
   hexists myLock UUID:threadId
   ```

2. 如果存在，那么增加加锁次数，并且刷新锁的过期时间

   ```vbnet
   hincrby myLock UUID:threadId 1：增加加锁次数
   
   pexpire myLock 30000毫秒：刷新锁key的过期时间
   ```

#### 4、分支四：当前线程本就在等待队列中，返回等待时间

```java
"local timeout = redis.call('zscore', KEYS[3], ARGV[2]);" +
"if timeout ~= false then " +
    // the real timeout is the timeout of the prior thread
    // in the queue, but this is approximately correct, and
    // avoids having to traverse the queue
    "return timeout - tonumber(ARGV[3]) - tonumber(ARGV[4]);" +
"end;" +
```

1. 利用 zscore 获取当前线程在超时集合中的超时时间

   ```sh
   zscore redisson_lock_timeout:{myLock} UUID:threadId
   ```

2. 返回实际的等待时间为：超时集合里的时间戳-30w毫秒-当前时间戳

#### 5、分支五：当前线程首次尝试获取锁，将当前线程加入到超时集合中，同时放入等待队列中

```java
"local lastThreadId = redis.call('lindex', KEYS[2], -1);" +
"local ttl;" +
"if lastThreadId ~= false and lastThreadId ~= ARGV[2] then " +
    "ttl = tonumber(redis.call('zscore', KEYS[3], lastThreadId)) - tonumber(ARGV[4]);" +
"else " +
    "ttl = redis.call('pttl', KEYS[1]);" +
"end;" +
"local timeout = ttl + tonumber(ARGV[3]) + tonumber(ARGV[4]);" +
"if redis.call('zadd', KEYS[3], timeout, ARGV[2]) == 1 then " +
    "redis.call('rpush', KEYS[2], ARGV[2]);" +
"end;" +
"return ttl;",
```

1. 利用 lindex 命令获取等待队列中排在最后的线程

   ```sh
   lindex redisson_lock_queue:{myLock} -1
   ```

2. 计算 ttl

   - 如果等待队列中最后的线程不为空且不是当前线程，根据此线程计算出ttl

   ```sh
   zscore redisson_lock_timeout:{myLock} lastThreadId：获取等待队列中最后的线程得过期时间
   
   ttl = timeout - 当前时间戳
   ```

   - 如果等待队列中不存在其他的等待线程，直接返回锁key的过期时间

   ```sh
   ttl = pttl myLock
   ```

3. 计算timeout，并将当前线程放入超时集合和等待队列中

   ```sh
   timeout = ttl + 30w毫秒 + 当前时间戳
   
   zadd redisson_lock_timeout:{myLock} timeout UUID:threadId：放入超时集合
   
   rpush redisson_lock_queue:{myLock} UUID:threadId：如果成功放入超市集合，同时放入等待队列
   ```

4. 最后返回ttl

## 3、watchdog 不断为锁续命

因为 RedissonFairLock 是基于 RedissonLock 做的，所以 watchdog 还是 RedissonLock 那一套。

## 4、死循环获取锁

因为 RedissonFairLock 是基于 RedissonLock 做的，所以死循环获取锁也还是 RedissonLock 那一套。

## 5、其他的加锁方式

如果我们需要指定获取锁成功后持有锁的时长，可以执行下面方法，指定 leaseTime

```java
lock.lock(10, TimeUnit.SECONDS);
```

> 如果指定了 leaseTime，watchdog就不会再启用了。

如果不但需要指定持有锁的时长，还想避免锁获取失败时的死循环，可以同时指定 leaseTime 和 waitTime

```java
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
```

> 如果指定了 waitTime，只会在 waitTime 时间内循环尝试获取锁，超过 waitTime 如果还是获取失败，直接返回false。