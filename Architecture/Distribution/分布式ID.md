# 分布式ID

分布式ID的两大核心需求：

- **全局唯一**
- **趋势有序**
- **高性能**

| 实现             | 原理                                                         |
| ---------------- | ------------------------------------------------------------ |
| UUID/GUID        | 基于 `UUID` 实现全球唯一的ID。                               |
| 数据库自增ID     | 基于数据库的 `auto_increment` 自增ID完全可以充当 `分布式ID` 。 |
| 数据库多主模式   |                                                              |
| 号段模式         |                                                              |
| Redis            |                                                              |
| 雪花算法         |                                                              |
| 滴滴TinyId       |                                                              |
| 百度Uidgenerator |                                                              |
| 美团Leaf         |                                                              |

## UUID/GUID

UUID，可以根据标准方法生成，不依赖中央机构的注册和分配，UUID具有唯一性，这与其他方案有所不同。

用作订单号`UUID`这样的字符串没有丝毫的意义，看不出和订单相关的有用信息；而对于数据库来说用作业务`主键ID`，它不仅是太长还是字符串，存储性能差查询也很耗时，所以不推荐用作`分布式ID`。

UUID的标准包含32个16进位数字，以连字号分为五段，形式为8-4-4-4-12的32个字元，范例550e8400-e29b-41d4-a716-446655440000 

在其规范的文本表示中，UUID的16个8位字节表示为32个十六进制的数字，显示在由连字符分割的-的五个组中，8-4-4-4-12总共36个字符，例如

```
123e4567-e89b-12d3-a456-426655440000
xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx
```


四位数字 M表示 UUID 版本 版本为

"版本1" UUID 是根据时间和节点 ID（通常是MAC地址）生成;

"版本2" UUID是根据标识符（通常是组或用户ID）、时间和节点ID生成;

"版本3" 和 "版本5" 确定性UUID 通过散列 (hashing) 命名空间 (namespace) 标识符和名称生成;

"版本4" UUID 使用随机性或伪随机性生成。



**优点**

- 生成足够简单，本地生成无网络消耗，具有唯一性

**缺点**

- 无序的字符串，不具备趋势自增特性
- 没有具体的业务含义，看不出和订单相关的有用信息
- 长度过长16 字节128位，36位长度的字符串，存储以及查询对MySQL的性能消耗较大，MySQL官方明确建议主键要尽量越短越好，作为数据库主键 `UUID` 的无序性会导致数据位置频繁变动，严重影响性能

**适用场景**

- 可以用来生成如token令牌一类的场景，足够没辨识度，而且无序可读，长度足够
- 可以用于无纯数字要求、无序自增、无可读性要求的场景



## 数据库自增ID

基于数据库的 `auto_increment` 自增ID完全可以充当 `分布式ID` 。当我们需要一个ID的时候，向表中插入一条记录返回`主键ID`，但这种方式有一个比较致命的缺点，访问量激增时MySQL本身就是系统的瓶颈，用它来实现分布式服务风险比较大，不推荐。相关SQL如下：

```mysql
CREATE DATABASE `SEQ_ID`;
CREATE TABLE SEQID.SEQUENCE_ID (
    id bigint(20) unsigned NOT NULL auto_increment, 
    value char(10) NOT NULL default '',
    PRIMARY KEY (id),
) ENGINE=MyISAM;

insert into SEQUENCE_ID(value)  VALUES ('values');
```

**优点**

- 实现简单，ID单调自增，数值类型查询速度快

**缺点**

- DB单点存在宕机风险，无法扛住高并发场景

**适用场景**

- 小规模的，数据访问量小的业务场景
- 无高并发场景，插入记录可控的场景



## 数据库多主模式

单点数据库方式不可取，那对上述的方式做一些高可用优化，换成主从模式集群。一个主节点挂掉没法用，那就做双主模式集群，也就是两个Mysql实例都能单独的生产自增ID。

**问题**：如果两个MySQL实例的自增ID都从1开始，会生成重复的ID怎么办？

**解决方案**：设置`起始值`和`自增步长`

MySQL_1 配置：

```mysql
set @@auto_increment_offset = 1;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
-- 自增ID分别为：1、3、5、7、9 ...... 
```

MySQL_2 配置：

```mysql
set @@auto_increment_offset = 2;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
-- 自增ID分别为：2、4、6、8、10 ......
```

那如果集群后的性能还是扛不住高并发咋办？则进行MySQL扩容增加节点：

![MySQL数据库多主模式](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/MySQL数据库多主模式.jpg)

从上图可以看出，水平扩展的数据库集群，有利于解决数据库单点压力的问题，同时为了ID生成特性，将自增步长按照机器数量来设置。增加第三台`MySQL`实例需要人工修改一、二两台`MySQL实例`的起始值和步长，把`第三台机器的ID`起始生成位置设定在比现有`最大自增ID`的位置远一些，但必须在一、二两台`MySQL实例`ID还没有增长到`第三台MySQL实例`的`起始ID`值的时候，否则`自增ID`就要出现重复了，**必要时可能还需要停机修改**。

**优点**

- 解决DB单点问题

**缺点**

- 不利于后续扩容，而且实际上单个数据库自身压力还是大，依旧无法满足高并发场景

**适用场景**

- 数据量不大，数据库不需要扩容的场景

这种方案，除了难以适应大规模分布式和高并发的场景，普通的业务规模还是能够胜任的，所以这种方案还是值得积累。



## 数据库号段模式

号段模式是当下分布式ID生成器的主流实现方式之一，可以理解为从数据库批量的获取自增ID，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，生成1~1000的自增ID并加载到内存。表结构如下：

```mysql
CREATE TABLE id_generator (
  id int(10) NOT NULL,
  max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(20) NOT NULL COMMENT '号段的步长',
  biz_type	int(20) NOT NULL COMMENT '业务类型',
  version int(20) NOT NULL COMMENT '版本号',
  PRIMARY KEY (`id`)
) 
```

biz_type ：代表不同业务类型

max_id ：当前最大的可用id

step ：代表号段的长度

version ：是一个乐观锁，每次都更新version，保证并发时数据的正确性

| id   | biz_type | max_id | step | version |
| ---- | -------- | ------ | ---- | ------- |
| 1    | 101      | 1000   | 2000 | 0       |

等这批号段ID用完，再次向数据库申请新号段，对`max_id`字段做一次`update`操作，`update max_id= max_id + step`，update成功则说明新号段获取成功，新的号段范围是`(max_id ,max_id +step]`。

```mysql
update id_generator set max_id=max_id+${step}, version = version+1 where version=${version} and biz_type=${XXX}
```

由于多业务端可能同时操作，所以采用版本号`version`乐观锁方式更新，这种`分布式ID`生成方式不强依赖于数据库，不会频繁的访问数据库，对数据库的压力小很多。




## Redis模式

`Redis`也同样可以实现，原理就是利用`redis`的 `incr`命令实现ID的原子性自增。

```shell
 # 初始化自增ID为1
127.0.0.1:6379> set seq_id 1
OK

# 增加1，并返回递增后的数值
127.0.0.1:6379> incr seq_id
(integer) 2
```

用`redis`实现需要注意一点，要考虑到`redis`持久化的问题。`redis`有两种持久化方式`RDB`和`AOF`：

- `RDB`：会定时打一个快照进行持久化，假如连续自增但`redis`没及时持久化，而这会`redis`挂掉了，重启`redis`后会出现ID重复的情况
- `AOF`：会对每条写命令进行持久化，即使`Redis`挂掉了也不会出现ID重复的情况，但由于incr命令的特殊性，会导致`Redis`重启恢复的数据时间过长



**优点**

- 有序递增，可读性强
- 能够满足一定性能

**缺点**

- 强依赖于Redis，可能存在单点问题
- 占用宽带，而且需要考虑网络延时等问题带来地性能冲击

**适用场景**

- 对性能要求不是太高，而且规模较小业务较轻的场景，而且Redis的运行情况有一定要求，注意网络问题和单点压力问题，如果是分布式情况，那考虑的问题就更多了，所以一帮情况下这种方式用的比较少

Redis的方案其实可靠性有待考究，毕竟依赖于网络，延时故障或者宕机都可能导致服务不可用，这种风险是不得不考虑在系统设计内的。



## 雪花算法（Snowflake）

雪花算法（Snowflake）是Twitter公司内部分布式项目采用的ID生成算法，开源后广受国内大厂的好评，在该算法影响下各大公司相继开发出各具特色的分布式生成器。

![雪花算法（SnowFlake）](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/雪花算法（SnowFlake）.jpg)

`Snowflake`生成的是Long类型的ID，一个Long类型占8个字节，每个字节占8比特，也就是说一个Long类型占64个比特。Snowflake ID组成结构：`正数位`（占1比特）+ `时间戳`（占41比特）+ `机器ID`（占5比特）+ `数据中心`（占5比特）+ `自增值`（占12比特），总共64比特组成的一个Long类型。

- **第一个bit位（1bit）**：Java中long的最高位是符号位代表正负，正数是0，负数是1，一般生成ID都为正数，所以默认为0
- **时间戳部分（41bit）**：毫秒级的时间，不建议存当前时间戳，而是用（当前时间戳 - 固定开始时间戳）的差值，可以使产生的ID从更小的值开始；41位的时间戳可以使用69年，(1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69年
- **工作机器id（10bit）**：也被叫做`workId`，这个可以灵活配置，机房或者机器号组合都可以
- **序列号部分（12bit）**：自增值支持同一毫秒内同一个节点可以生成4096个ID



**优点**

- 每秒能够生成百万个不同的ID，性能佳
- 时间戳值在高位，中间是固定的机器码，自增的序列在地位，整个ID是趋势递增的
- 能够根据业务场景数据库节点布置灵活挑战bit位划分，灵活度高

**缺点**

- **强依赖于机器时钟**，如果时钟回拨，会导致重复的ID生成，所以一般基于此的算法发现时钟回拨，都会抛异常处理，阻止ID生成，这可能导致服务不可用

**适用场景**

- 雪花算法有很明显的缺点就是时钟依赖，如果确保机器不存在时钟回拨情况的话，那使用这种方式生成分布式ID是可行的，当然小规模系统完全是能够使用的



## 百度（Uid-Generator）

`uid-generator`是基于`Snowflake`算法实现的，与原始的`snowflake`算法不同在于，`uid-generator`支持自`定义时间戳`、`工作机器ID`和 `序列号` 等各部分的位数，而且`uid-generator`中采用用户自定义`workId`的生成策略。

`uid-generator`需要与数据库配合使用，需要新增一个`WORKER_NODE`表。当应用启动时会向数据库表中去插入一条数据，插入成功后返回的自增ID就是该机器的`workId`数据由host，port组成。



**对于`uid-generator` ID组成结构**：

`workId`占用了22个bit位，时间占用了28个bit位，序列化占用了13个bit位。这里的时间单位是秒，而不是毫秒，`workId`也不一样，而且同一应用每次重启就会消费一个`workId`。



## 美团（Leaf）

`Leaf`同时支持号段模式和`snowflake`算法模式，可以切换使用。

### Leaf-segment数据库方案

![Leaf-segment数据库方案](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Leaf-segment数据库方案.png)

在建一张表`leaf_alloc`：

```mysql
DROP TABLE IF EXISTS `leaf_alloc`;
CREATE TABLE `leaf_alloc` (
  `biz_tag` varchar(128)  NOT NULL DEFAULT '' COMMENT '业务key',
  `max_id` bigint(20) NOT NULL DEFAULT '1' COMMENT '当前已经分配了的最大id',
  `step` int(11) NOT NULL COMMENT '初始步长，也是动态调整的最小步长',
  `description` varchar(256)  DEFAULT NULL COMMENT '业务key的描述',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '数据库维护的更新时间',
  PRIMARY KEY (`biz_tag`)
) ENGINE=InnoDB;
```

test_tag在第一台Leaf机器上是1~1000的号段，当这个号段用完时，会去加载另一个长度为step=1000的号段，假设另外两台号段都没有更新，这个时候第一台机器新加载的号段就应该是3001~4000。同时数据库对应的biz_tag这条数据的max_id会从3000被更新成4000，更新号段的SQL语句如下：

```mysql
Begin
UPDATE table SET max_id=max_id+step WHERE biz_tag=xxx
SELECT tag, max_id, step FROM table WHERE biz_tag=xxx
Commit
```

**优点**

- Leaf服务可以很方便的线性扩展，性能完全能够支撑大多数业务场景
- ID号码是趋势递增的8byte的64位数字，满足上述数据库存储的主键要求
- 容灾性高：Leaf服务内部有号段缓存，即使DB宕机，短时间内Leaf仍能正常对外提供服务
- 可以自定义max_id的大小，非常方便业务从原有的ID方式上迁移过来

**缺点**

- ID号码不够随机，能够泄露发号数量的信息，不太安全
- TP999数据波动大，当号段使用完之后还是会hang在更新数据库的I/O上，tg999数据会出现偶尔的尖刺
- DB宕机会造成整个系统不可用



**双buffer优化**

针对第二个缺点是因为在号段用完后才会出现，因此可以在消耗完前提前获取下一个号段，从而解决问题：

![Leaf-segment双buffer优化](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Leaf-segment双buffer优化.png)

采用双buffer的方式，Leaf服务内部有两个号段缓存区segment。当前号段已下发10%时，如果下一个号段未更新，则另启一个更新线程去更新下一个号段。当前号段全部下发完后，如果下个号段准备好了则切换到下个号段为当前segment接着下发，循环往复：

- 每个biz-tag都有消费速度监控，通常推荐segment长度设置为服务高峰期发号QPS的600倍（10分钟），这样即使DB宕机，Leaf仍能持续发号10-20分钟不受影响
- 每次请求来临时都会判断下个号段的状态，从而更新此号段，所以偶尔的网络抖动不会影响下个号段的更新



**Leaf高可用容灾**

对于第三点“DB可用性”问题，采用一主两从的方式，同时分机房部署，Master和Slave之间采用**半同步方式**同步数据。

![Leaf-segment高可用容灾](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Leaf-segment高可用容灾.png)




### Leaf-snowflake方案

Leaf-segment方案可以生成趋势递增的ID，同时ID号是可计算的，不适用于订单ID生成场景，比如竞对在两天中午12点分别下单，通过订单id号相减就能大致计算出公司一天的订单量，这个是不能忍受的。面对这一问题，我们提供了 Leaf-snowflake方案。Leaf-snowflake方案完全沿用snowflake方案的bit位设计，即是“1+41+10+12”的方式组装ID号。对于workerID的分配，当服务集群数量较小的情况下，完全可以手动配置。Leaf服务规模较大，动手配置成本太高。所以使用Zookeeper持久顺序节点的特性自动对snowflake节点配置wokerID。Leaf-snowflake是按照下面几个步骤启动的：

- 启动Leaf-snowflake服务，连接Zookeeper，在leaf_forever父节点下检查自己是否已经注册过（是否有该顺序子节点）
- 如果有注册过直接取回自己的workerID（zk顺序节点生成的int类型ID号），启动服务
- 如果没有注册过，就在该父节点下面创建一个持久顺序节点，创建成功后取回顺序号当做自己的workerID号，启动服务

![Leaf-snowflake方案](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/Leaf-snowflake方案.png)



## 滴滴（TinyID）

`Tinyid`是基于号段模式原理实现的与`Leaf`如出一辙，每个服务获取一个号段（1000,2000]、（2000,3000]、（3000,4000]

![滴滴（TinyID）](C:/Users/DELL/Downloads/lemon-guide-main/images/Solution/滴滴（TinyID）.png)

`Tinyid`提供`http`和`tinyid-client`两种方式接入。



### Http方式接入

**第一步**：导入Tinyid源码

```shell
git clone https://github.com/didi/tinyid.git
```

**第二步**：创建数据表

```mysql
-- 建表SQL
CREATE TABLE `tiny_id_info` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `biz_type` varchar(63) NOT NULL DEFAULT '' COMMENT '业务类型，唯一',
  `begin_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '开始id，仅记录初始值，无其他含义。初始化时begin_id和max_id应相同',
  `max_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '当前最大id',
  `step` int(11) DEFAULT '0' COMMENT '步长',
  `delta` int(11) NOT NULL DEFAULT '1' COMMENT '每次id增量',
  `remainder` int(11) NOT NULL DEFAULT '0' COMMENT '余数',
  `create_time` timestamp NOT NULL DEFAULT '2010-01-01 00:00:00' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT '2010-01-01 00:00:00' COMMENT '更新时间',
  `version` bigint(20) NOT NULL DEFAULT '0' COMMENT '版本号',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_biz_type` (`biz_type`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT 'id信息表';

CREATE TABLE `tiny_id_token` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `token` varchar(255) NOT NULL DEFAULT '' COMMENT 'token',
  `biz_type` varchar(63) NOT NULL DEFAULT '' COMMENT '此token可访问的业务类型标识',
  `remark` varchar(255) NOT NULL DEFAULT '' COMMENT '备注',
  `create_time` timestamp NOT NULL DEFAULT '2010-01-01 00:00:00' COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT '2010-01-01 00:00:00' COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT 'token信息表';

-- 添加tiny_id_info
INSERT INTO `tiny_id_info` (`id`, `biz_type`, `begin_id`, `max_id`, `step`, `delta`, `remainder`, `create_time`, `update_time`, `version`) VALUES (1, 'test', 1, 1, 100000, 1, 0, '2018-07-21 23:52:58', '2018-07-22 23:19:27', 1);
INSERT INTO `tiny_id_info` (`id`, `biz_type`, `begin_id`, `max_id`, `step`, `delta`, `remainder`, `create_time`, `update_time`, `version`) VALUES(2, 'test_odd', 1, 1, 100000, 2, 1, '2018-07-21 23:52:58', '2018-07-23 00:39:24', 3);

-- 添加tiny_id_token
INSERT INTO `tiny_id_token` (`id`, `token`, `biz_type`, `remark`, `create_time`, `update_time`) VALUES(1, '0f673adf80504e2eaa552f5d791b644c', 'test', '1', '2017-12-14 16:36:46', '2017-12-14 16:36:48');
INSERT INTO `tiny_id_token` (`id`, `token`, `biz_type`, `remark`, `create_time`, `update_time`) VALUES(2, '0f673adf80504e2eaa552f5d791b644c', 'test_odd', '1', '2017-12-14 16:36:46', '2017-12-14 16:36:48');
```

**第三步**：配置数据库

```properties
datasource.tinyid.names=primary
datasource.tinyid.primary.driver-class-name=com.mysql.jdbc.Driver
datasource.tinyid.primary.url=jdbc:mysql://ip:port/databaseName?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8
datasource.tinyid.primary.username=root
datasource.tinyid.primary.password=123456
```

**第四步**：启动`tinyid-server`后测试

```properties
# 获取分布式自增ID
http://localhost:9999/tinyid/id/nextIdSimple?bizType=test&token=0f673adf80504e2eaa552f5d791b644c'
返回结果: 3

# 批量获取分布式自增ID
http://localhost:9999/tinyid/id/nextIdSimple?bizType=test&token=0f673adf80504e2eaa552f5d791b644c&batchSize=10'
返回结果:  4,5,6,7,8,9,10,11,12,13
```



### Java客户端方式接入

**第一步**：引入依赖

```xml
       <dependency>
            <groupId>com.xiaoju.uemc.tinyid</groupId>
            <artifactId>tinyid-client</artifactId>
            <version>${tinyid.version}</version>
        </dependency>
```

**第二步**：配置文件

```properties
tinyid.server =localhost:9999
tinyid.token =0f673adf80504e2eaa552f5d791b644c
```

**第三步**：`test` 、`tinyid.token`是在数据库表中预先插入数据，`test` 是具体业务类型，`tinyid.token`表示可访问的业务类型

```java
// 获取单个分布式自增ID
Long id =  TinyId . nextId( " test " );
// 按需批量分布式自增ID
List< Long > ids =  TinyId . nextId( " test " , 10 );
```

