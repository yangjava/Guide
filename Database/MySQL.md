# MySQL

## MySQL 整体架构一览 

MySQL 在整体架构上分为 Server 层和存储引擎层。其中 Server 层，包括连接器、查询缓存、分析器、优化器、执行器等，存储过程、触发器、视图和内置函数都在这层实现。数据引擎层负责数据的存储和提取，如 InnoDB、MyISAM、Memory 等引擎。在客户端连接到 Server 层后，Server 会调用数据引擎提供的接口，进行数据的变更。

![MySQL整体架构](png\MySQL\MySQL整体架构.png)

### 连接器

负责和客户端建立连接，获取用户权限以及维持和管理连接。

通过 `show processlist;` 来查询连接的状态。在用户建立连接后，即使管理员改变连接用户的权限，也不会影响到已连接的用户。默认连接时长为 8 小时，超过时间后将会被断开。

简单说下长连接：

优势：在连接时间内，客户端一直使用同一连接，避免多次连接的资源消耗。

劣势：在 MySQL 执行时，使用的内存被连接对象管理，由于长时间没有被释放，会导致系统内存溢出，被系统kill. 所以需要定期断开长连接，或执行大查询后，断开连接。MySQL 5.7 后，可以通过 `mysql_rest_connection` 初始化连接资源，不需要重连或者做权限验证。

### 查询缓存

当接受到查询请求时，会现在查询缓存中查询（key/value保存），是否执行过。没有的话，再走正常的执行流程。

但在实际情况下，查询缓存一般没有必要设置。因为在查询涉及到的表被更新时，缓存就会被清空。所以适用于静态表。在 MySQL8.0 后，查询缓存被废除。

### 分析器

词法分析：

如识别 select，表名，列名，判断其是否存在等。

语法分析：

判断语句是否符合 MySQL 语法。

### 优化器

确定索引的使用，join 表的连接顺序等，选择最优化的方案。

### 执行器

在具体执行语句前，会先进行权限的检查，通过后使用数据引擎提供的接口，进行查询。如果设置了慢查询，会在对应日志中看到 `rows_examined` 来表示扫描的行数。在一些场景下（索引），执行器调用一次，但在数据引擎中扫描了多行，所以**引擎扫描的行数和 rows_examined 并不完全相同。**

> 不预先检查权限的原因：如像触发器等情况，需要在执行器阶段才能确定权限，在优化器阶段无法验证。

### 使用 profiling 查看 SQL 执行过程

打开 profiling 分析语句执行过程：

```sql
mysql> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set, 1 warning (0.00 sec)
mysql> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

执行查询语句：

```smalltalk
mysql> SELECT * FROM s limit 10;
+------+--------+-----+-----+
| s_id | s_name | age | sex |
+------+--------+-----+-----+
|    1 | z      | 12  |   1 |
|    2 | s      | 14  |   0 |
|    3 | c      | 14  |   1 |
+------+--------+-----+-----+
3 rows in set (0.00 sec)
```

获取 profiles;

```smalltalk
mysql> show profiles;
+----------+------------+--------------------------+
| Query_ID | Duration   | Query                    |
+----------+------------+--------------------------+
|        1 | 0.00046600 | SELECT * FROM s limit 10 |
+----------+------------+--------------------------+

mysql> show profile;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000069 |
| checking permissions | 0.000008 |  权限检查
| Opening tables       | 0.000018 |  打开表
| init                 | 0.000019 |  初始化
| System lock          | 0.000010 |  锁系统
| optimizing           | 0.000004 |  优化查询
| statistics           | 0.000013 |
| preparing            | 0.000094 |  准备
| executing            | 0.000016 |  执行
| Sending data         | 0.000120 |
| end                  | 0.000010 |
| query end            | 0.000015 |
| closing tables       | 0.000014 |
| freeing items        | 0.000032 |
| cleaning up          | 0.000026 |
+----------------------+----------+
15 rows in set, 1 warning (0.00 sec)
```

查询具体的语句：

```smalltalk
mysql> show profile for query 1;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000069 |
| checking permissions | 0.000008 |
| Opening tables       | 0.000018 |
| init                 | 0.000019 |
| System lock          | 0.000010 |
| optimizing           | 0.000004 |
| statistics           | 0.000013 |
| preparing            | 0.000094 |
| executing            | 0.000016 |
| Sending data         | 0.000120 |
| end                  | 0.000010 |
| query end            | 0.000015 |
| closing tables       | 0.000014 |
| freeing items        | 0.000032 |
| cleaning up          | 0.000026 |
+----------------------+----------+
15 rows in set, 1 warning (0.00 sec)
```

## MySQL 日志模块

如前面所说，MySQL 整体分为 Server 层和数据引擎层，而每层也对应了自己的日志文件。如果选用的是 InnoDB 引擎，对应的是 redo log 文件。Server 层则对应了 binlog 文件。至于为什么存在了两种日志系统，咱们往下看。

### redo log

redo log 是 InnoDB 特有日志，为什么要引入 redo log 呢，想象这样一个场景，MySQL 为了保证持久性是需要把数据写入磁盘文件的。我们知道，在写入磁盘时，会进行文件的 IO，查找操作，如果每次更新操作都这样的话，整体的效率就会特别低，根本没法使用。

既然直接写入磁盘不行，解决方法就是先写进内存，在系统空闲时再更新到磁盘就可以了。但光更新内存不行，假如系统出现异常宕机和重启，内存中没有被写入磁盘的数据就会被丢掉，数据的一致性就出现问题了。这时 redo log 就发挥了作用，在更新操作发生时，InnoDb 会先写入 redo log 日志（记录了数据发生了怎么样的改变），同时更新内存，最后在适当的时间再写入磁盘，一般是找系统空闲的时间做。**先写日志，在写磁盘的操作，就是常说到的 WAL （Write-Ahead- Logging）技术。**

**redo log 的出现，除了在效率上有了很大的改善，还保证了 MySQL 具有了 crash-safe 的能力，在发生异常情况下，不会丢失数据。**

在具体实现上 redo log 的大小是固定的，可配置一组为 4 个文件，每个文件 1GB，更新时对四个文件进行循环写入。

![MySQL-RedoLog](png\MySQL\MySQL-RedoLog.png)

write pos 记录当前写入的位置，写完就后移，当第写入第 4 个文件的末尾时，从第 0 号位置重新写入。

check point 表示当前可以擦除的位置，当数据更新到磁盘时，check point 就向后移动。

write pos 和 check point 之间的位置，就是可以记录更新操作的空间。当 write pos 追上 check point ，不在能执行新的操作，先让 check point 去写入一些数据。

可以将 `innodb_flush_log_at_trx_commit` 设置成 1，开启 redo log 持久化的能力。

### binlog

binlog 则是 Server 层的日志，主要用于归档，在备份，主备同步，恢复数据时发挥作用，常见的日志格式有 `row`, `mixed`, `statement` 三种。具体的使用方法可以参见 [Binlog 恢复日志这篇。](https://www.cnblogs.com/michael9/p/11923483.html)

可以通过 `sync_binlog`=1 开启 binlog 写入磁盘。

这里对 binlog 和 redo 进行下区分：

1. 所有者不同，binlog 是 Server 层，所有引擎都可使用。redo log 是 InnoDB 特有的。
2. 类型不同，binlog 是逻辑日志，记录的是语句的原始逻辑（比 statement）。redo log 是物理日志，记录某个数据页被做了怎样的修改。
3. 数据写入的方式不同，binog 日志会一直追加，而 redo log 是循环写入。
4. 功能不同，binlog 用于归档，而 redo log 用于保证 crash-safe.

### 两阶段提交

下面执行器和 InnoDB 执行 Update 时内部流程：

以更新 `update T set c=c+1 where ID=2;` 语句为例：

1. 执行器通过 InooDB 引擎去 ID 所在行，ID 为主键。引擎通过树搜索找到该行，如果该行所在数据页在内存中，返回给执行器。否则先从磁盘读入内存，然后再返回。
2. 执行器拿到引擎给的数据，将 C 值加 1，等到新的一行，然后通过引擎接口重新写入新数据。
3. 引擎将该行更新到内存中，同时将该更新操作记录到 redo log 中，并更改 redo log 的状态为 prepare 状态。然后告知执行器，在合适的时间提交事务。
4. 执行器生成这个操作的 binlog，并将 binlog 写入磁盘。
5. 执行器调用引擎到的提交事务接口，将刚刚写入的 redo log 改成 commit 状态，更新完成。

[![img](https://img2020.cnblogs.com/blog/1861307/202009/1861307-20200922152056410-1828606460.png)](https://img2020.cnblogs.com/blog/1861307/202009/1861307-20200922152056410-1828606460.png)

浅色为执行器执行，深色为引擎执行。

在更新内存后，将写入 redo log 拆分了成两个步骤：prepare 和 commit，就是常说的两阶段提交。用于保证当有意外情况发生时，数据的一致性。

这里假设下，如果不采用两阶段提交会发生什么？

1. 先写 redo log 后写 binlog. 假设在写入 redo log 后，MySQL 发生异常重启，此时 binlog 没有写入。在重启后，由于 redolog 已经写入，此时数据库的内容是没有问题的。但此时，如果想要拿 binlog 进行备份或恢复，发现会少了最后一条的更新逻辑，导致数据不一致。
2. 先写 binlog 后写 redo log. binlog 写入后，MySQL 异常重启，redo log 没有写入。此时重启后，发现 redo log 没有成功写入，认为这个事务无效，而此时 binlog 却多了一条更新语句，拿去恢复后自然数据也是不一致的。

再分析下两阶段提交的过程：

1. 在写 redo log prepare 阶段奔溃，时刻 A 的位置。重启后，发现 redo log 没写入，回滚此次事务。
2. 如果在写 binlog 时奔溃，重启后，发现 binlog 未被写入，回滚操作。
3. binlog 写完，但在提交 redo log 的 commit 状态时发生 crash.
   1. 如果 redo log 中事务完整，有了 commit 标识，直接提交。
   2. 如果 redo log 中只有完整的 prepare, 判断对应 binlog 是否完整。
      1. 完整，提交事务
      2. 不完整，回滚事务。

如何判断 binlog 是否完整？

- statement 格式 binlog，会有 COMMIT; 标识
- row 格式的 binlog，会有 XID event. 标识
- 在 5.6 后，还有 binlog-checksum 参数，验证 binlog 正确性。

如何将 redo log 和 binlog 关联表示同一个操作？
结构中有一个共同的数据字段，XID. 在崩溃恢复时，会按顺序扫描 redo log:

- 如果有 prepare，又有 commit 的 redo log，直接提交。
- 如果只有 prepare，没有 commit 的 redo log, 拿 XID 去 binlog 找对应的事务做判断。

数据写入后，最终落盘和 redo log 有无关系？

1. 对于正常运行的 instance 来说，内存中页被修改后，和磁盘的数据页不一致，称为脏页。而落盘的过程，是把内存中的数据页写入磁盘。
2. 对于 crash 场景，InnoDB 判断一个数据页是否丢失了更新，会将其读到内存，然后让 redo log 更新内存内容。更新完成后，内存页就变成脏页，然后回到第一种情况的状态。

redo log buffer 和 redo log 的关系？

在一个事务的更新过程中，存在多个 SQL 语句，所以是要写多次日志的。
但在写的过程中，生产的日志要先保存起来，但在 commit 前，不能直接写到 redo log 中。
所以通过内存中 redo log buffer 先存 redo log 的日志。在 commit 时，将 buffer 中的内容写入 redo log.

## 总结

在文章开始部分，说明了 MySQL 的整体架构分为 Server 层和引擎层，并简要说明了一条语句的执行过程。接着 MySQL 在 5.5 后选用 InnoDB 作为默认的引擎，就是因为比原生的 MyISAM 多了事务以及 crash-safe 的能力。

而 crash-safe 就是由 redo log 实现的。与 redo log 类似的日志文件还有 binlog，是 Server 引擎的日志，用于归档和备份数据。

最后提到了，为了保证数据的一致性，将 redo log 和 binlog 放入相同的事务中，也就是常提到的两阶段提交操作。

# [MYSQL 整体架构浅析](https://www.cnblogs.com/rickiyang/p/13473854.html)

对于一个服务端开发来说 MYSQL 可能是他使用最熟悉的数据库工具，然而熟练掌握 MYSQL 语句的拼写和卓越的多条件查询不代表出现性能问题的时候你知道该怎么解决。致力于不当 SQL boby，我们从头开始入门 MYSQL，讲一些你可能不知道的 MYSQL。

#### 1. 一条 SQL 之旅[#](https://www.cnblogs.com/rickiyang/p/13473854.html#230785397)

现在有一条查询用户信息表的 SQL ：

```sql
Copyselect * from user where uid = 100001;
```

这条 SQL 是如何从你的应用程序到达 MYSQL 服务器并执行，然后查到结果再带给你的呢？要回答这个问题我们的看一下 MYSQL 的整体架构：

![img](https://img2020.cnblogs.com/blog/1607781/202008/1607781-20200811095743571-951229811.png)

1. 建立连接

   首先客户端与 MYSQL 服务器建立连接这个就不用说，客户端发起请求经过三次握手之后与服务器建立 TCP 连接；服务器收到请求之后对输入的用户名密码做权限校验， 校验通过之后后续的通信就基于这个长连接进行传输。

   一般来说一个 MYSQL 服务端是可以对应多个客户端的，所以在服务端可能会有很多个客户端连接同时保持，可以通过 `show processlist;` 命令来查看当前服务端有多少个连接：

   ```sql
   Copymysql> show processlist;
   +---------+------+---------------------+-------------------+---------+-------+-------+------------------+
   | Id      | User | Host                | db                | Command | Time  | State | Info             |
   +---------+------+---------------------+-------------------+---------+-------+-------+------------------+
   | 1753984 | test | 10.31.0.64:65264    | xxx    | Sleep   |    92 |       | NULL             |
   | 6348496 | test | 10.26.134.61:43080  | xxx        | Sleep   |     2 |       | NULL             |
   | 7201973 | test | 10.26.8.104:61642   | xxx   | Sleep   |   927 |       | NULL             |
   | 7201976 | test | 10.26.8.104:61650   | xxx   | Sleep   |   927 |       | NULL             |
   | 7414866 | test | 10.26.134.238:52010 | xxx | Sleep   |    32 |       | NULL             |
   ......
   ......
   ......
   +---------+------+---------------------+-------------------+---------+-------+-------+------------------+
   42 rows in set (0.00 sec)
   ```

   上面有个 `Time` 参数：表示该连接已经多久没有发生过数据传输。默认如果超过 8 小时没有发生过数据传输服务端就会自动关闭该连接。可以通过 `wait_timeout`参数来设置超时时间。

2. 查询缓存

   MySQL 查询缓存是 MySQL 中比较独特的一个缓存区域，用来缓存特定 Query 的整个结果集信息，且共享给所有客户端。为了提高完全相同的 Query 语句的响应速度，MySQL Server 会对查询语句进行 Hash 计算后，把得到的 hash 值与 Query 查询的结果集对应存放在Query Cache 中。当 MySQL Server 打开 Query Cache 之后，MySQL Server 会对接收到的每一个 SELECT 语句通过特定的 Hash 算法计算该 Query 的 Hash 值，然后通过该 hash 值到 Query Cache 中去匹配。

   查询缓存相关的配置参数有如下：

   ```sql
   Copymysql> show variables like '%query_cache%';
   +------------------------------+---------+
   | Variable_name                | Value   |
   +------------------------------+---------+
   | have_query_cache             | YES     |      --查询缓存是否可用
   | query_cache_limit            | 1048576 |      --可缓存具体查询结果的最大值
   | query_cache_min_res_unit     | 4096    |      --查询缓存分配的最小块的大小(字节)
   | query_cache_size             | 599040  |      --查询缓存的大小
   | query_cache_type             | ON      |      --是否支持查询缓存
   | query_cache_wlock_invalidate | OFF     |      --控制当有写锁加在表上的时候，是否先让该表相关的 Query Cache失效
   +------------------------------+---------+
   6 rows in set (0.02 sec)
   ```

   开启缓存

   ```sql
   Copymysql> set global query_cache_size = 600000; --设置缓存内存大小
   mysql> set global query_cache_type = ON;     --开启查询缓存
   ```

   关闭缓存

   ```sql
   Copymysql> set global query_cache_size = 0; --设置缓存内存大小为0， 即初始化是不分配缓存内存
   mysql> set global query_cache_type = OFF;     --关闭查询缓存
   ```

3. 分析器

   如果查询缓存未开启或者为命中的情况则会走正常的查询流程，第一步就是 sql 解析器，作用是将整个查询语句变为 MYSQL 服务器能理解的语言。

   1. 词法分析：将整个查询分解为多个元素；
   2. 语法分析：寻找 sql 语法规则产生一个序列并执行这些代码；
   3. 已经前两步之后会产生一个解析树，提供给优化器使用。

4. 查询优化器

   优化器工作主要包括两个部分：

   1. 逻辑优化；
   2. 物理优化。

   **逻辑优化阶段**：大牛们为了我们这些渣渣程序员操碎了心，怕你写的 sql 不好查询慢然后上网发帖 “MYSQL 垃圾”，“再也不用 MYSQL，我准备自己写个数据库”，于是默默在在后台给你整了个 sql 语句优化。优化内容主要有：

   一定能带来优化效果的：

   - 连接的消除（外连接，嵌套连接）
   - 语义优化
   - 冗余操作剪枝

   可能会带来性能的提升但是需要根据代价进行选择

   - 借用索引来优化分组、排序等操作
   - 合并分组
   - 连接查询条件下推
   - 公式条件的提取
   - 谓词上推

   **物理优化阶段：** 逻辑优化主要是针对语法规则和语义的规整、合并、裁剪，那么到了物理优化阶段就需要拿着 MYSQL 认为是比较完美的 sql 去查询底层存储单元。查询物理存储需要解决的问题包括：

   - 单表扫描中什么样的方式扫描效率最优
   - 两个表连接的时候如何 join 才能最快的获取数据
   - 多表连接的时候如何进行排序，是否要对每种组合都进行探索

   早期物理优化阶段使用基于关系代数规则和启发式规则对查询进行优化后就认为生辰的执行计划是最优的。后面引入了最小代价查询方式对每一个可能的可执行方式进行评估找到代价最小的作为最优执行计划，目前数据库的查询通常是将这两种方式融合到一起。

   关于优化器部分是可以讲上几天都讲不完的，在此我们只是简单介绍。

5. 执行器

   经过上面阶段已经将如何查询得到数据的最优解拿到，执行器需要做的是根据指令去获取数据。

#### 2. MYSQL 数据存储[#](https://www.cnblogs.com/rickiyang/p/13473854.html#2658051725)

MYSQL 作为一个数据库管理工具最底层肯定是将数据存入磁盘的，那么数据是如何进入磁盘的，进入磁盘之后的格式是什么，用什么方式来管理这些数据，让我们带着种种疑团走进 “今日说法”，一起来揭开这个秘密。

##### 2.1 存储引擎

根据对数据存储方式、使用方式的要求， MYSQL 提供了各种私人订制方案来让客户爸爸满意，这些技术方案通过使用不同的存储机制、索引方式、锁技巧最终得到不同的组合效果同而适配不同的应用场景，我们将这种组合得到的产物定义为：存储引擎。

存储引擎主要做了哪些事情呢，包括不限于：

- 用合适的格式存储数据
- 提供数据查询，更新的接口
- 各种条件下数据一致性的支持
- 索引机制的建立
- 提供数据备份、故障恢复、故障转移的能力

对于上面这些基本要求的实现，MYSQL 提供了哪些方案呢？

###### MyISAM

**ISAM** ：Indexed Sequential Access Method（有索引的顺序访问方法）。MyISAM 底层基于这个引擎做了一些改良，这是 MYSQL 5.5 版本之前默认数据库引擎，在当时那个年代提供了一些 **当时没有但是很必要** 的特性：

- 索引管理
- 字段管理
- 表锁

当时这些功能可是没有的，早期的程序员确实很辛苦啊，人家造轮子是真轮子用来跑的，现在造轮子也是轮子不过是站在巨人的肩膀看得更远了吧。

MyISAM 引擎对每张表的存贮会归结为三个文件：

- .frm：以表的名字开始，存储表定义；
- .MYD：存储表数据；
- .MYI：存储表索引。

因为 MyISAM 诞生的年代比较早，那时候也没有现在互联网这么庞大的需求，所以放在现在来看它其实是有很多的缺点，比如：

- 锁粒度太大，MyISAM 表锁有两种模式：表共享读锁，表独占写锁。读的时候他不会阻塞其他用户对同一表读的需求，但是在读期间不能写；在写的时候，会同时阻塞其他用户对表的读写操作。这个放在现在高并发的场景下肯定是个灾难。
- MyISAM 不支持事务。这种锁机制也注定了它不能支持事务，它生来就是为了 select 操作更快而优化的并且也是满足当时时代的场景。
- 在崩溃恢复方面，因为各种备份机制没有那么多，带来的直接后果就是它不支持崩溃后的安全恢复。

###### MEMORY

内存表，很显然这种表是为了读取速度而生。它既不支持事务也不支持外键等等，内存表的优势是读取快，那缺点就很多了：

- 对于 Varchar 类型使用固定大小的长度存储，浪费空间；
- 占用内存资源，可能会造成内存崩溃；
- 服务器重启，数据丢失。

###### InnoDB

从 MYSQL 5.5 开始，InnoDB成为表的默认引擎。它具有有史以来堪称完美的特点：行锁设计、支持多版本隔离控制、支持外键、支持一致性非锁定读、同时对内存和 CPU 的使用也随着新硬件的发展做了一定的优化。

InnoDB 引擎本身是基于磁盘存储数据的，但是因为 CPU速度和磁盘速度之间巨大的差距，所以在 InnoDB 引擎中大量使用缓冲池技术来提高读写速度。整体可以分为两个部分：

1. 缓冲区

   缓冲区的类型也是各种各样，主要包括：

   1. Buffer Pool：缓冲池，在主内存中开辟的一个区域，在 InnoDB 读数据的时候会先访问这里，以减少磁盘访问频次；
   2. Change Buffer：写缓冲区，避免每次增删改都进行IO操作；
   3. Adaptive Hash Index：自适应哈希索引，使用索引关键字的前缀构建哈希索引，提升查询速度；
   4. Log Buffer：日志缓冲区，保存要写入磁盘上的日志文件的数据，缓冲区的内容定期刷新到磁盘。

2. 磁盘数据

   磁盘中的数据结构可以分为两大类：表空间和重做日志。

   表空间又可分为：

   1. The System Tablespace（系统表空间）：存储更改缓冲区；
   2. File-Per-Table Tablespaces（独立表空间）：存储单个Innodb表的数据和索引；
   3. General Tablespaces（通用表空间）：使用`CREATE TABLESPACE`创建的共享表；
   4. Undo Tablespaces（undo表空间）：存储`undo`日志；
   5. Temporary Tablespaces（临时表空间）：临时表包括会话临时表和全局临时表。

   重做日志，redo log 保存的就是 buffer pool 刷到磁盘的数据。

InnoDB 在磁盘中对应的文件结构比较多，除去 redo log 和 bin log之外的主要文件有：

- .opt：数据库配置文件，包含数据库字符集属性；
- .frm：数据表元数据文件，不管是独立表空间还是系统表空间，每个表都对应一个；
- .ibd：数据库独立表空间文件，如果是独立表空间则对应一个.ibd文件，否则保存在系统表空间。

后面我们专门找一节来讲 InnoDB 引擎，这里简单概述。

#### 3. 常用命令[#](https://www.cnblogs.com/rickiyang/p/13473854.html#286564553)

那么对于我们使用的数据库如何查看当前使用的引擎呢：

查看当前 MYSQL 版本所支持的引擎：

```sql
Copymysql> show engines; 
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

查看 MYSQL 默认的存储引擎：

```sql
Copymysql> show variables like '%storage_engine%';
+----------------------------+--------+
| Variable_name              | Value  |
+----------------------------+--------+
| default_storage_engine     | InnoDB |
| default_tmp_storage_engine | InnoDB |
| storage_engine             | InnoDB |
+----------------------------+--------+
3 rows in set (0.01 sec)
```

查看某个表用了什么引擎(在显示结果里参数engine后面的就表示该表当前用的存储引擎):

```sql
Copymysql> show create table 表名;
```

修改表的存储引擎：

```sql
CopyALTER TABLE 表名 ENGINE = INNODB；
```

修改默认存储引擎

如果修改本次会话的默认存储引擎(重启后失效)，只对本会话有效，其他会话无效：

```sql
Copymysql> set default_storage_engine=innodb;
Query OK, 0 rows affected (0.00 sec)
```

修改全局会话默认存储引擎(重启后失效)，对所有会话有效：

```sql
Copymysql> set global default_storage_engine=innodb;
Query OK, 0 rows affected (0.00 sec)
```

希望重启后也有效，即编辑 `/etc/my.cnf`，[mysqld] 下面任意位置添加配置(所有对配置文件的修改，重启后生效)

```sql
Copydefault-storage-engine = InnoDB
```

关于 MYSQL 的结构部分限于篇幅就简单介绍，后面我们一一来看细节。

# [InnoDB引擎面面观](https://www.cnblogs.com/rickiyang/p/13520878.html)

自 2010 年 MYSQL 5.5.5 发布以来，InnoDB 已经取代 MyISAM 作为 MYSQL 的默认表类型，这得益于他在新时代前瞻性的功能开发：

☑ 遵循 ACID 模型开发的 DML 操作

☑ 对事务的支持

☑ 支持行级锁

☑ 支持外键

这些特性让 MYSQL 在新时代仍能够站在时代之巅，时至今日仍旧不为落后。

### InnoDB 架构[#](https://www.cnblogs.com/rickiyang/p/13520878.html#1688487024)

下图显示了构成`InnoDB`存储引擎体系结构的内存中和磁盘上的结构：

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghu7wvkdxyj30jg0eyq6j.jpg)

 图片来自于 MYSQL 官网

如上图所示，InnoDB 架构分为两大部分：内存存储 和 磁盘存储，每一部分分别有自己的组成部分：

内存存储部分包括：

- Buffer Pool
- Change Buffer
- Adaptive Hash Index
- Log Buffer

磁盘存储包括：

- Tables
- Indexes
- Tablespaces
- Doublewrite buffer
- Redo Log
- Undo Logs

就上面的各个部分我们单独拿出来一一介绍。

### 内存架构[#](https://www.cnblogs.com/rickiyang/p/13520878.html#2846353712)

#### Buffer Pool[#](https://www.cnblogs.com/rickiyang/p/13520878.html#2346328549)

Buffer Pool 最主要的功能就是加速读和加速写。

当读取数据的时候如果该数据所在的数据页正好在 Buffer Pool 中，那么就直接从 Buffer Pool 中读取数据；

当写入数据的时候，先将该数据放入 Buffer Pool 中，并记录 redo log 日志，对于写入操作到这里就算完成了。至于这个页什么时候会被刷入到磁盘，这就是刷脏的逻辑。

在 InnoDB 中数据是按照 页/ 块(默认为 16K )的方式存储到磁盘，并以同样的方式读取文件到 Buffer Pool 中，然后用同样大小空间做内存映射。既然是预读数据那么肯定存在冷热问题，常见的缓存淘汰算法 LRU 在这种场景必然逃不掉，我们先从 Buffer Pool 的结构开始着手。

Buffer Pool 由两个部分组成：控制块 和 缓存数据页：

![4](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghu7wuvoxnj30fs08j3zc.jpg)

二者一一对应，控制块中的内容主要是缓存数据页的页号、表空间号、数据页在 Buffer Pool 中的地址等一系列信息。

单个控制块的大小约为缓存数据页的 5% 左右，并且控制块的大小不计入 Buffer Pool 的分配空间内。如果你分配了 10G 的 Buffer Pool，那么实际占用的内存大小可能大于 10G，因为 MySQL 实例启动的时候，需要记入控制块的大小。

在 Buffer Pool 中保存的次级模块 叫做 Buffer chunks。一个 Buffer Pool 中有一个或多个 chunk，每个 chunk 大小默认为 128M，最小为 1M，每个 chunk 中包含一个 `buf_block_t` 的 blocks 数组，保存的内容即上面我们说的数据页。

Buffer Pool 肯定不能随便将数据页载入内存，所以在 chunk 中包含当面数据页数组和对应的控制体信息，在代码中 Buffer Pool 用 `buf_pool_t` 对象来表述，它包含四个部分：

- free 链表：存储当前 Buffer Pool 实例中所有的空闲页面
- flush_list 链表：存储所有被修改过且需要刷到文件中区的页面
- mutex：保护实例，同一时刻只能被一个线程访问的对象
- LRU list：chunks，加载到内存中的数据页块链表。

**free list**

初始化的时候会申请一定数量的 page，当然这些 page 都是 free page，所以在 free list 中保存的都是 未被使用的页。

在执行 sql 的过程中，每次拉取数据页到内存中都会判断 free list 的页面是否够用，如果不够就 flush LRU 链表和 flush_list 链表来释放空页；如果够用的情况就从 free list 中删除对应页面并将改页添加入 LRU list，申请的总页数保持不变。

**LRU list**

所有新读取进来的数据页都在这里。与传统的 LRU 算法不同的是将链表分为冷热两个部分，主要是为了防止预读的数据页和全表扫描对缓冲区污染。

LRU list 整体空间做了如下划分：

![2](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghu7wwzltzj30go0h4gn3.jpg)

 图片来自于 MYSQL 官网

LRU list 有两个区域，一个是链表头部的 new Sublist 区域(热数据)，另一个是 old Sublist 区域(冷数据)。

默认情况下 LRU List 的 3/8 用于 old sublist，new Sublist 与 old Sublist 的分界线是链表的中点字段：midpoint。

如果一个页之前没有在 Buffer Pool 中，首次被查询到会添加到 new Sublist 中，如果一个没有 where 条件的查询进行全表扫描那么会将大部分无效数据代入 new Sublist，造成 真正的热点数据进入 old Sublist 而被回收。

预读机制

针对预读造成的数据污染，InnoDB 也做出了相应的措施。最近访问的页默认会插入到 LRU list 的 5/8 之后的位置，即 Old Sublist 的头部位置。

如果有一个新的预读任务做了全表扫描，读取出来的页信息会首先放到 Old Sublist 的头部，这些页中可能有某些页才是本次查询真正的数据所在页，那么这些页在读取的时候会被加入 new Sublist 的头部。

老生代停留时窗口

尽管有这样的预读机制，对于不会命中索引的 ”like“ 查询仍然会造成大量的缓冲池污染。 MYSQL 又提供了一个 ”老生代停留时间窗口“ 的机制，配置参数 `innodb_old_blocks_time` 指定了一个在第一次访问到实际移动这个数据页到 new Sublist 头部的时间窗口，单位为：毫秒。默认 值为 1000 ，即为 1000 毫秒。

假设预读一批数据插入到 Old Sublist 的头部，此时设置的 window timewait = 2000，如果在 2000ms 内该数据被访问那么也不会被移动到 New Sublist 中，只有满足 被访问 且 在 Old Sublist 中停留的时间 > window timewait 才会被放入 New Sublist。

**flush list**

这里用于缓存那些当前发生过更改的数据。需要注意的是在 flush list 中保存的并不是数据页，而是数据页对应的控制块信息。

当某个数据页在 Buffer Pool 中第一次被更改过，会将它加入到 flush list 链表中，链表采用头插法，同时记录该数据页的两个属性：

oldest_modification：oldest 是指修改该页面的 mtr 第一次开始时候的 lsn 号

newest_modification：newest 是指最后一次修改该页面的 mtr 结束时候的 lsn 号

如果该页再次有改动的时候不会执行插入操作而是更改 newest_modification。

**mutex**

InnoDB 定义了很多的 Mutex，场景包括数据缓冲区，字典表系统锁表等等，用来保护有并发竞争的对象。BUffer Pool 中也是一样，比如对同一个数据页的更改操作必须要加锁，否则导致覆盖。

我们来看一些 Buffer Pool 相关的配置参数， `show variables like "Innodb_buffer_pool%";` 可以查看 buffer_pool 相关的配置参数：

```sql
Copymysql> show variables like "Innodb_buffer_pool%";
+-------------------------------------+----------------+
| Variable_name                       | Value          |
+-------------------------------------+----------------+
| innodb_buffer_pool_chunk_size       | 134217728      |
| innodb_buffer_pool_dump_at_shutdown | ON             |
| innodb_buffer_pool_dump_now         | OFF            |
| innodb_buffer_pool_dump_pct         | 25             |
| innodb_buffer_pool_filename         | ib_buffer_pool |
| innodb_buffer_pool_instances        | 1              |
| innodb_buffer_pool_load_abort       | OFF            |
| innodb_buffer_pool_load_at_startup  | ON             |
| innodb_buffer_pool_load_now         | OFF            |
| innodb_buffer_pool_size             | 134217728      |
+-------------------------------------+----------------+
```

InnoDB_buffer_pool_size

用于设置 InnoDB 缓存池（InnoDB_buffer_pool) 的大小，默认值是 47MB。InnoDB 缓存池的大小对 InnoDB 整体性能影响较大，如果当前的 MySQL 服务器专门用于提供 MySQL 服务，应尽量增加 `InnoDB_buffer_pool_size`的大小，把频繁访问的数据都放到内存中来，尽可能减少 InnoDB 对硬盘的访问，争取将 InnoDB 最大化成为一个内存存储引擎。

InnoDB_buffer_pool_instances

默认值是 1，表示 InnoDB 缓存池被划分到一个区域。适当地增加该参数（例如将该参数值设置为 2，此时 InnoDB 被划分成为两个区域），可以提升 InnoDB 的并发性能。如果 InnoDB 缓存池被划分成多个区域，建议每个区域不小于 1GB 的空间。

innodb_buffer_pool_dump_pct

可以设置一次读取的数据页填充 Buffer Pool 的占比，默认是 25%。如果经常做全表扫描那么缓冲区很可以总是被无效数据填充，所以这时候可以将 `innodb_buffer_pool_dump_pct`设置的小一些，对于这种大批量扫描就不会对缓冲区做侵入性覆盖。

查看 Buffer Pool 的运行状态: `show status like "Innodb_buffer_pool%";` :

```sql
Copymysql> show status like "Innodb_buffer_pool%";
+---------------------------------------+--------------------------------------------------+
| Variable_name                         | Value                                            |
+---------------------------------------+--------------------------------------------------+
| Innodb_buffer_pool_dump_status        | Dumping of buffer pool not started               |
| Innodb_buffer_pool_load_status        | Buffer pool(s) load completed at 170819  9:57:57 |
| Innodb_buffer_pool_resize_status      |                                                  |
| Innodb_buffer_pool_pages_data         | 324                                              |
| Innodb_buffer_pool_bytes_data         | 5308416                                          |
| Innodb_buffer_pool_pages_dirty        | 0                                                |
| Innodb_buffer_pool_bytes_dirty        | 0                                                |
| Innodb_buffer_pool_pages_flushed      | 39                                               |
| Innodb_buffer_pool_pages_free         | 7868                                             |
| Innodb_buffer_pool_pages_misc         | 0                                                |
| Innodb_buffer_pool_pages_total        | 8192                                             |
| Innodb_buffer_pool_read_ahead_rnd     | 0                                                |
| Innodb_buffer_pool_read_ahead         | 0                                                |
| Innodb_buffer_pool_read_ahead_evicted | 0                                                |
| Innodb_buffer_pool_read_requests      | 1620                                             |
| Innodb_buffer_pool_reads              | 290                                              |
| Innodb_buffer_pool_wait_free          | 0                                                |
| Innodb_buffer_pool_write_requests     | 515                                              |
+---------------------------------------+--------------------------------------------------+
```

可以看到一共有多少页（Innodb_buffer_pool_pages_total），空闲页数（Innodb_buffer_pool_pages_free），脏页数（Innodb_buffer_pool_pages_dirty）等等。通过这些状态可以调整配置来让缓存尽可能多地命中。

#### Change Buffer[#](https://www.cnblogs.com/rickiyang/p/13520878.html#3070399189)

Change Buffer 的主要目的是将对二级索引的数据操作缓存下来，以此减少二级索引的随机 IO，并达到操作合并的效果。

在 MySQL 5.5 之前的版本中，由于只支持缓存 insert 操作，所以最初叫做 insert buffer，只是后来的版本中支持了更多的操作类型缓存，才改叫 Change Buffer。

Change Buffer 物理上是一颗 BTree，一条 Change Buffer log 记录大概包含如下列：

![5](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghu7ww24xoj31by0p6n2w.jpg)

在 Change Buffer tree 上 唯一定为一条记录是通过三列 (space id, page no , counter) 作为主键的，其中 counter 是一个递增值，目的是为了维持不同操作的有序性，例如可以通过 counter 来保证在 merge 时执行如下序列时的循序和用户操作顺序是一致的：INSERT x, DELETE-MARK x, INSERT x。

#### Adaptive Hash Index[#](https://www.cnblogs.com/rickiyang/p/13520878.html#4146870477)

Adaptive Hash index（自适应哈希索引）的特性使得 InnoDB 在不牺牲事务特性或可靠性的前提下，为缓冲池提供适当的工作负载和足够的内存的时候，能够表现的更像 in-memory（内存）数据库。

该特性是通过变量 `innodb_adaptive_hash_index` 来使用的，可以说 Adaptive Hash index 不 是传统意义的索引，可以理解为在 Btree 上的 "索引"。

当对某个页面访问次数满足一定条件会将页面地址存于 Hash 表，下次查询可以非常快速的找到页面不需要 Btree 去查。

#### Log Buffer[#](https://www.cnblogs.com/rickiyang/p/13520878.html#2222683358)

Log Buffer (日志缓冲区)是一块内存区域用来保存要写入磁盘上的日子文件的数据。 Log Buffer 的大小由`innodb_log_buffer_size` 变量定义。默认大小为 16MB。Log Buffer 的内容会定期刷到磁盘上。

设置较大的 Log Buffer 让较大事务能够运行，而无需在事务提交之前将 redo log 中的数据写入磁盘。如果你的系统中有较多大事务类型的操作，增加日志缓冲区的大小可以节省磁盘 IO。

Log Buffer 通过 `innodb_flush_log_at_trx_commit` 参数来控制日志刷入磁盘的频率：

- 0：每秒写入日志并将其刷新到磁盘一次，未刷新日志的事务可能会在崩溃中丢失。
- 1：是默认值，每次事务提交时写入日志并刷新到磁盘，确保数据不会丢失，这种方式是最安全的，但同时也是最慢的。
- 2：在每次事务提交后写入日志，并每秒刷新一次磁盘。未刷新日志的事务可能会在崩溃中丢失。每次事务提交时 MySQL 都会把 log buffer 的数据写入 log file，但是 flush(刷到磁盘)操作并不会同时进行。该模式下，MySQL 会每秒执行一次 flush(刷到磁盘)操作。）也就是每次事务提交，该事务都会在 log file 里面，但刷入磁盘的操作是每秒一次的，不是写入 log file 时一起（同步）的，所以只有操作系统崩溃或断电的情况下才会丢失上一秒的事务。

上面 InnoDB 整体架构图中可以看到 Redo log buffer 是 InnoDB 内存区域的一部分。

#### Redo Log[#](https://www.cnblogs.com/rickiyang/p/13520878.html#2647828428)

InnoDB 使用 Redo Log 来保证数据的一致性和可持久性，它采用 WAL 机制，即先写日志再写数据。具体来说，InnoDB 进行写操作时，先将数据操作记录在 log buffer 中，然后将 log buffer 中的数据刷到磁盘 log file 中，后续数据再落到数据 ibd 文件这一步骤由 checkpoint 来保证。

redo log 包括两部分：一是内存中的日志缓冲(redo log buffer)，该部分日志是易失性的；二是磁盘上的重做日志文件(redo log file)，该部分日志是持久的。将 redo log buffer 写入 redo log file 遵循 `innodb_flush_log_at_trx_commit` 参数的设定。

redo log 相关的参数设定中有一个参数：`innodb_log_files_in_group`，表明一个日志组中有多少个日志文件，虽然 MySQL 5.6 开始已经放弃了日志组的概念，但参数名依旧保留了下来以兼容以前的配置。该参数的含义为有多少个 log 文件(最少为 2 个)。所以 redo log 总日志文件个数是有限的，redo log 采用顺序写的方式，在全部文件写满之后则会回到第一个文件的起始位置重新写入。

redo log 以块为单位进行存储的，每个块占 512 字节，称为 redo log block。所以不管是 log buffer 中还是 os buffer 中以及 redo log file on disk 中，都是这样以 512 字节的块存储的。

每个 redo log block 由 3 部分组成：**日志块头、日志块尾和日志主体**。其中日志块头占用 12 字节，日志块尾占用 8 字节，所以每个 redo log block 的日志主体部分只有 512-12-8=492 字节。

redo log block 数据格式

log block 中 492 字节的部分是 log body，它由 4 个部分构成：

- redo_log_type：占用 1 个字节，表示 redo log 的日志类型
- space：表示表空间的 ID，采用压缩的方式后，占用的空间可能小于 4 字节
- page_no：表示页的偏移量，同样是压缩过的
- redo_log_body 表示每个重做日志的数据部分，恢复时会调用相应的函数进行解析。例如 insert 语句和 delete 语句写入 redo log 的内容是不一样的。

checkpoint 机制

在 InnoDB 中 checkpoint 分为两种情况：

- sharp checkpoint：在重用 redo log 文件(例如切换日志文件)的时候，将所有已记录到 redo log 中对应的脏数据刷到磁盘。
- fuzzy checkpoint：一次只刷一小部分的日志到磁盘，而非将所有脏日志刷盘。有以下几种情况会触发该检查点：
  - master thread checkpoint：由 master 线程控制，**每秒或每 10 秒** 刷入一定比例的脏页到磁盘。
  - flush_lru_list checkpoint：从 MySQL5.6 开始可通过 `innodb_page_cleaners` 变量指定专门负责脏页刷盘的 page cleaner 线程的个数，该线程的目的是为了保证 LRU 列表有可用的空闲页。
  - async/sync flush checkpoint：同步刷盘还是异步刷盘。例如还有非常多的脏页没刷到磁盘(非常多是多少，有比例控制)，这时候会选择同步刷到磁盘，但这很少出现；如果脏页不是很多，可以选择异步刷到磁盘，如果脏页很少，可以暂时不刷脏页到磁盘。
  - dirty page too much checkpoint：脏页太多时强制触发检查点，目的是为了保证缓存有足够的空闲空间。too much 的比例由变量 `innodb_max_dirty_pages_pct` 控制，MySQL 5.6 默认的值为 75，即当脏页占缓冲池的 75% 后，就强制刷一部分脏页到磁盘。

由于刷脏页需要一定的时间来完成，所以记录 checkpoint 的位置是在每次刷盘结束之后才在 redo log 中标记的。

redo log 总的写入量叫 LSN（Log Secquence Numer）日志序列号，这个 redo log 变更实际写入到实际数据文件中的数量叫 checkpoint LSN，表示的是有多少变更已经实际写入到了相应的数据文件中。 一旦数据库崩溃 InnoDB 开始恢复数据的时候，先读取 checkpoint，然后从 checkpoint 所指示的 LSN 读取其之后的 Redo log 进行数据恢复，从而减少 Crash Recovery 的时间。

LSN

根据 LSN，可以获取到几个有用的信息：

1. 数据页的版本信息
2. 写入的日志总量，通过 LSN 开始号码和结束号码可以计算出写入的日志量
3. 可知道检查点的位置

LSN 不仅存在于 redo log 中，还存在于数据页。在每个数据页的头部，有一个 *fil_page_lsn* 记录了当前页最终的 LSN 值是多少。通过数据页中的 LSN 值和 redo log 中的 LSN 值比较，如果页中的 LSN 值小于 redo log 中 LSN 值，则表示数据丢失了一部分。这时候可以通过 redo log 的记录来恢复到 redo log 中记录的 LSN 值时的状态。

查看 redo log 中的 LSN 值：

```sql
Copy
mysql> show engine innodb status;

 =====================================
 2020-08-16 15:33:25 0x30f4 INNODB MONITOR OUTPUT
 =====================================
 Per second averages calculated from the last 20 seconds
 -----------------
 BACKGROUND THREAD
 -----------------
 srv_master_thread loops: 2 srv_active, 0 srv_shutdown, 229135 srv_idle
 srv_master_thread log flush and writes: 229137
 ----------
 SEMAPHORES
 ----------
 OS WAIT ARRAY INFO: reservation count 4
 OS WAIT ARRAY INFO: signal count 4
 RW-shared spins 0, rounds 4, OS waits 2
 RW-excl spins 0, rounds 0, OS waits 0
 RW-sx spins 0, rounds 0, OS waits 0
 Spin rounds per wait: 4.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
 ------------
 TRANSACTIONS
 ------------
 Trx id counter 25350
 Purge done for trx's n:o < 0 undo n:o < 0 state: running but idle
 ......
 ......
 ......
 ---
 LOG
 ---
 Log sequence number 2648197
 Log flushed up to   2648197
 Pages flushed up to 2648197
 Last checkpoint at  2648188
 0 pending log flushes, 0 pending chkp writes
 10 log i/o's done, 0.00 log i/o's/second
 ----------------------
 BUFFER POOL AND MEMORY
 ----------------------
 Total large memory allocated 8585216
 Dictionary memory allocated 124624
 Buffer pool size   512
 Free buffers       256
 Database pages     256
 Old database pages 0
 Modified db pages  0
 Pending reads      0
 Pending writes: LRU 0, flush list 0, single page 0
 Pages made young 0, not young 0
 0.00 youngs/s, 0.00 non-youngs/s
 Pages read 209, created 49, written 53
 0.00 reads/s, 0.00 creates/s, 0.00 writes/s
 No buffer pool page gets since the last printout
 Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
 LRU len: 256, unzip_LRU len: 0
 I/O sum[17]:cur[0], unzip sum[0]:cur[0]
 --------------
 ROW OPERATIONS
 --------------
 0 queries inside InnoDB, 0 queries in queue
 0 read views open inside InnoDB
 Process ID=3952, Main thread ID=5596, state: sleeping
 Number of rows inserted 65, updated 0, deleted 0, read 73
 0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
 ----------------------------
 END OF INNODB MONITOR OUTPUT
 ============================
```

这里面有几个字段：

`log sequence number`：就是当前的 redo log(in buffer) 中的 LSN

`log flushed up to`：是刷到 redo log file on disk 中的 LSN

`pages flushed up to` ：是已经刷到磁盘数据页上的 LSN

`last checkpoint at` ：是上一次检查点所在位置的 LSN

可以看到上面示例中因为是本地测试数据库，所以落盘的 LSN 和 当前检查的 LSN 已经齐平。

### 磁盘架构[#](https://www.cnblogs.com/rickiyang/p/13520878.html#2483520585)

磁盘空间从大类上划分比较简单：表空间 和 日志空间。

- 表空间：分为系统表空间(MySQL 目录的 ibdata1 文件)、临时表空间、常规表空间、Undo 表空间以及 独立表空间(file-per-table 表空间，MySQL5.7 默认打开 `file_per_table` 配置）。

  系统表空间又包括 InnoDB 数据字典、双写缓冲区(Doublewrite Buffer)、修改缓存(Change Buffer）、Undo 日志等。

- Redo 日志：存储的就是 Log Buffer 刷到磁盘的数据。

#### 表空间[#](https://www.cnblogs.com/rickiyang/p/13520878.html#21382251)

*表空间涉及的文件*

相关文件默认在磁盘中的`innodb_data_home_dir`目录下：

```cpp
Copy|- ibdata1  // 系统表空间文件
|- ibtmp1  // 默认临时表空间文件，可通过innodb_temp_data_file_path属性指定文件位置
|- test/  // 数据库文件夹
    |- db.opt  // test数据库配置文件，包含数据库字符集属性
    |- t.frm  // 数据表元数据文件，不管是使用独立表空间还是系统表空间，每个表都对应有一个
    |- t.ibd  // 数据库表独立表空间文件，如果使用的是独立表空间，则一个表对应一个ibd文件，否则保存在系统表空间文件中
```

**frm 文件**

创建一个 InnoDB 表时，MySQL 在数据库目录中创建一个 .frm 文件，frm 文件包含 MySQL 表的元数据(如表定义)。每个 InnoDB 表都有一个 .frm 文件。

与其他 MySQL 存储引擎不同， InnoDB 它还在 系统表空间 内的自身内部数据字典中编码有关表的信息。MySQL 删除表或数据库时，将删除一个或多个`.frm`文件以及 InnoDB 数据字典中的相应条目。

**ibd 文件**

对于在独立表空间创建的表，还会在数据库目录中生成一个 .ibd 表空间文件。

在通用表空间中创建的表在现有的常规表空间 .ibd 文件中创建。常规表空间文件可以在**MySQL 数据目录内部或外部创建**。

**ibdata 文件**

系统表空间文件，在 InnoDB 系统表空间中创建的表在 ibdata 中创建。

*表空间对应的存储结构*

在 InnoDB 存储引擎中，每张表都有一个主键（Primary Key）。从 InnoDB 存储引擎的**逻辑存储结构**看，所有数据都根据主键顺序被逻辑地存放在一个空间中，称之为表空间（Tablespace）。从外部来看，一张表是由连续的固定大小的 Page 构成，其实表空间文件内部被组织为更复杂的逻辑结构，自顶向下可分为段（Segment）、区（Extent）、页（Page）、Row（行），InnoDB 存储引擎的文件逻辑存储结构大致如下图所示：

![6](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghu7wynomqj31120u079w.jpg)

Segment 与数据库中的索引相映射。InnoDB 引擎内，数据段即为 B+ Tree 的叶子节点，索引段即为 B+ Tree 的非叶子节点，创建索引中很关键的步骤便是分配 Segment。

Segment 的下一级是 Extent，Extent 代表一组连续的 Page，默认大小均为 1MB。Extent 的作用是提高 Page 分配效率，在数据连续性方面也更佳，Segment 扩容时也是以 Extent 为单位分配。

Page 则是表空间数据存储的基本单位，InnoDB 将表文件按 Page 切分，依类型不同，Page 内容也有所区别，最为常见的是存储行记录的数据页。

Row 行 对应着表里的一条数据记录。

在默认情况下，InnoDB 存储引擎 Page 的大小为 16KB，即一个 Extent 中一共有 64 个连续的 Page。在创建 MySQL 实例时，可以通过指定`innodb_page_size`选项对 Page 的大小进行更改，需要注意的是 Page 的大小可能会影响 Extent 的大小：

| page size | page nums | extent size |
| :-------: | :-------: | :---------: |
|    4KB    |    256    |     1MB     |
|    8KB    |    128    |     1MB     |
|   16KB    |    64     |     1MB     |
|   32KB    |    64     |     2MB     |
|   64KB    |    64     |     4MB     |

从上表可以看出，一个 Extent 最小也有 1 MB，且最少拥有 64 个页。

*行记录格式*

InnoDB 存储引擎和大多数数据库一样，记录是以行的形式存储的，每个 16KB 大小的页中可以存放 2~200 条行记录。InnoDB 早期的文件格式为`Antelope`，可以定义两种行记录格式，分别是`Compact`和`Redundant`，InnoDB 1.0.x 版本开始引入了新的文件格式`Barracuda`。`Barracuda`文件格式下拥有两种新的行记录格式：`Compressed`和`Dynamic`。

`Compact`行记录格式是在 MySQL 5.0 中引入的，其首部是一个非 NULL 变长列长度列表，并且是逆序放置的，其长度为：

- 若列的长度小于等于 255 字节，用 1 个字节表示；
- 若列的长度大于 255 字节，用 2 个字节表示。

变长字段的长度最大不可以超过 2 字节，这是因为 MySQL 数据库中 VARCHAR 类型的最大长度限制为 65535。变长字段之后的第二个部分是 NULL 标志位，该标志位指示了该行数据中某列是否为 NULL 值，有则用 1 表示，NULL 标志位也是不定长的。接下来是记录头部信息，固定占用 5 字节。

`Redundant`是 MySQL 5.0 版本之前 InnoDB 的行记录格式，`Redundant`行记录格式的首部是每一列长度偏移列表，同样是逆序存放的。从整体上看，`Compact`格式的存储空间减少了约 20%，但代价是某些操作会增加 CPU 的使用。

**行溢出数据**

数据页默认的大小是 16KB（6384 字节），而定义的 VARCHAR 行长度大小 65535 字节，这里会存在一个也放不下的情况，于是数据会被放到大对象页中（Uncompressed BLOB Page），原数据中保留 768 字节 + 偏移量；

> VARCHAR 最大长度问题： 定义 **VARCHAR 所在行**可以存放 6384 字节，然而实际有行头数据开销，**最大值为 65532 字节**。 需要注意的是这里不是单个列的长度。
>
> 数据是否溢出使用大对象页存储： 由于数据存储使用的是 B+Tree 的结构，一个页中至少要有两个节点，并且页大小为 16KB。 所以，这个阈值是 8098 字节，小于此值当行数据会存储在本身页中，大于这个值则会使用 BLOB 页进行存储，（这时原行数据只存储前 768 字节数据 + BLOB 页的偏移量）

**关于 CHAR 的行存储结构**

在变长字符集的（例如：UTF8）的情况下，InnoDB 对 CHAR 的存储也是看做变长字符类型的，与 VARCHAR 没有区别。

关于 Redo Log 日志准备再开一篇说明，本文先到这里。

# [MySQL 索引结构](https://www.cnblogs.com/rickiyang/p/13559507.html)

谈到 MYSQL 索引服务端的同学应该是熟悉的不能再熟悉，新建表的时候怎么着都知道先来个主键索引，对于经常查询的列也会加个索引加快查询速度。那么 MYSQL 索引都有哪些类型呢？索引结构是什么样的呢？有了索引是如何检索数据的呢？我们围绕这些问题来探讨一下。

#### 你认为应该如何查询数据[#](https://www.cnblogs.com/rickiyang/p/13559507.html#267994065)

上一节谈到 InnoDB 引擎的时候聊过在 InnoDB 引擎是面向行存储的，数据都是存储在磁盘的数据页中，数据页里面按照固定的行格式存储着每一行数据。

InnoDB存储引擎是 B+ 树索引组织的，所以数据即索引，索引即数据。B+ 树的叶子节点存储的都是数据段的数据。InnoDB 引擎对数据的存储必须依赖于主键，主键对应的索引叫做聚集索引。如果不幸的是你建表没有建主键，InnoDB 会从表字段中寻找第一个非空的唯一索引作为聚集索引，如果还是不幸找不到，InnoDB 会生成一个不可见的名为 ROW_ID 的列，该列是一个 6 字节的自增数字，用来创建聚集索引。

小Tips:

**对于 ROW_ID 列的自增实现其实是来自于一个全局自增序列，这意味着所有使用到 ROW_ID 作为聚集索引的表都共享该序列，如果在高并发的情况就有保证不了唯一性的可能。**

大家都知道 MYSQL 中索引是使用 B+ 树的数据结构，在此也就不故弄玄虚。但是大家有没有想过除了 B+ 树还有什么数据结构也可以用于索引检索呢？我们不妨来看看。

##### 二叉树

对于二叉树而言，每个节点只能有两个子节点，如果是一颗单边二叉树，查询某个节点的次数与节点所处的高度相同，时间复杂度为O(n)；如果是一颗平衡二叉树，查找效率高出一半，时间复杂度为O(Log2n)。

并且二叉树还有另一个坏处，二叉树上的每一个节点都是数据节点，那么对于一个比较高的数如果要获取最下面的数据遍历的节点数将会很消耗性能。

##### Hash 表

散列表的好处是散列查询单条数据比较快，但是坏处也比较多，比如 Hash 碰撞的解决，范围查找等等。

##### B 树

B树是二叉树的升级版，又叫平衡多路查找树。它和平衡二叉树的区别在于：

1. 平衡二叉树最多两个子树，而 B 树每个节点都可以有多个子树，M 阶 B 树表示每个节点最多有M个子树。
2. 平衡二叉树每个节点只有一个数据和两个指向孩子的指针，而 B 树每个**中间节点**有 k-1 个关键字（可以理解为数据）和 k 个子树（ k 介于阶数 M 和 M/2 之间，M/2 向上取整）。
3. 所有叶子节点均在同一层、叶子节点除了包含关键字和关键字记录的指针外也有指向其子节点的指针，只不过其指针地址都为 null 。

另外，它们相同的点是节点数据也是按照左小右大的顺序排列。我们用一张图来对比它们的区别：

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1gi2zcqwmogj30lf0fetb7.jpg)

##### B+ 树

说到 B 树就连着B+树一起说了。B+ 树是应文件系统所需而产生的一种 B 树的变形树（文件的目录一级一级索引，只有最底层的叶子节点（文件）保存数据）非叶子节点只保存索引，不保存实际的数据）。

![4](https://tva1.sinaimg.cn/large/007S8ZIlgy1gi2zcswldej30h407gwfc.jpg)

这里有一个很重要的一点就是 B+ 树的非叶子节点不保存数据，只有索引。一棵 m 阶的 B+ 树和 m 阶的 B 树的异同点在于：

1. 节点的子树数和关键字数相同（B 树是关键字数比子树数少一）；
2. 所有的叶子结点中包含了全部关键字的信息，及指向含有这些关键字记录的指针（节点的关键字表示的是子树中的最大数，在子树中同样含有这个数据），且叶子结点本身依关键字的大小自小而大的顺序链接。 (而 B 树的叶子节点并没有包括全部需要查找的信息)；
3. **所有的非终端结点可以看成是索引部分**，结点中仅含有其子树根结点中最大（或最小）关键字。 (而 B 树的非终节点也包含需要查找的有效信息)。
4. 叶子节点之间通过指针连接。

对于 B 树和 B+ 树来说，两种数据结构都是为了减少磁盘 I/O 读写过于频繁而生，本身节点的个数是有限的，采用多叉结构就是为了让每一层放尽可能多的节点以此来降低整棵树的高度。但是为什么 InnoDB 索引结构最终选择了 B+ 树而不是B 树呢？

**B+ 树的磁盘读写代价更低**

B+ 树内部非叶子节点本身并不存储数据，所以非叶子节点的存储代价相比 B 树就小的多。存储容量减少同时也缩小了占用盘块的数量，那么数据的聚集程度直接也影响了查询磁盘的次数。

**B+ 树查询效率更加稳定**

树高确定的前提下所有的数据都在叶子节点，那么无论怎么查询所有关键字查询的路径长度是固定的。

**B+ 树对范围查询的支持更好**

B+ 树所有数据都在叶子节点，非叶子节点都是索引，那么做范围查询的时候只需要扫描一遍叶子节点即可；而 B 树因为非叶子节点也保存数据，范围查询的时候要找到具体数据还需要进行一次中序遍历。

#### MyISAM 和 InnoDB 索引组织的区别[#](https://www.cnblogs.com/rickiyang/p/13559507.html#2483461758)

在 MYSQL 中索引属于存储引级别的概念，存储引擎不同，索引的实现方式也不一样。我们分别看看看 MyISAM 和 InnoDB 中都是如何实现索引功能。

##### MyISAM 实现

MyISAM 也是使用 B+ 树作为索引存储结构，他的叶子节点 data 域存放的是数据的物理地址，即索引结构和真正的数据结构其实是分开存储的。

![2](https://tva1.sinaimg.cn/large/007S8ZIlgy1gi2zctubpej31740u00zg.jpg)

##### InnoDB 索引实现

MyISAM 索引和数据是分离的，但是在 InnoDB 中却大不相同，InnoDB 中采用主键索引的方式，所有的数据都保存在主键索索引中。

![3](https://tva1.sinaimg.cn/large/007S8ZIlgy1gi2zcrxqr2j315g0p279s.jpg)

所以这也是为什么 InnoDB 要求每个表都必须要有主键的原因。本身就是基于主键来组织的数据存储。

#### 索引类型[#](https://www.cnblogs.com/rickiyang/p/13559507.html#2796074083)

以下所有索引类型都是基于 InnoDB 引擎。

##### 主键索引

主键索引也就是我们说的聚集索引。上面说过主键索引是基于主键来创建的 B+ 树索引结构，如果没有指定主键，也找不到任何一列不重复的列可以作为主键的情况下，InnoDB 会新增一个隐藏列 RowId 作为主键继而创建聚集索引。

##### 二级索引(非主键索引)

二级索引就是指除了主键索引外的索引。主键索引和所有的二级索引都是各自维护各自的 B+ 树结构，但是有个不同的地方在于，二级索引的叶子节点存储的不是数据，而是主键索引对应的主键值。

即二级索引不再保存一份 data 数据，而是去主键索引中查数据。那么对于二级索引查找一条数据索要做的操作就是：

1. 首先在二级索引中找到叶子节点对应的数据主键值；
2. 根据这个主键值去聚集索引中找到真正对应的数据行。

所以这里需要两次 B+ Tree 查找。

##### 覆盖索引

覆盖索引简单来说就是只查询索引就能获取到数据不必再回表查询，换句话说要查询的列已经被索引列覆盖。

使用覆盖索引有如下优点：

1. 索引项通常比记录要小，所以 MySQL 访问更少的数据；
2. 索引都按值的大小顺序存储，相对于随机访问记录，需要更少的 I/O；
3. 大多数据引擎能更好的缓存索引。比如 MyISAM 只缓存索引;
4. 覆盖索引对于 InnoDB 表尤其有用，因为 InnoDB 使用聚集索引组织数据，如果二级索引中包含查询所需的数据，就不再需要在聚集索引中查找了。
5. 覆盖索引不能是任何索引，只有 B Tree 索引存储相应的值。而且不同的存储引擎实现覆盖索引的方式都不同，并不是所有存储引擎都支持覆盖索引( Memory 和 Falcon 就不支持)。

##### 联合索引

有的时候我们会对多个列建立一个索引，这种索引被称为联合索引。而关于联合索引的建立和使用，从工作开始你的各位 “师长” 都在教导你要遵循 “左前匹配原则”，那到底是为什么呢？什么是左前匹配原则呢？

比如我们有这样一张表：

```sql
CopyCREATE TABLE `test_tb` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `a` varchar(10) NOT NULL,
  `b` varchar(10) NOT NULL,
  `c` varchar(10) NOT NULL,
  `d` int(10) NOT NULL,
  `e` int(10) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_a_b_c` (`a`,`b`,`c`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

上表建立了一个联合索引：idx_a_b_c。下面给出一个 SQL， 大家看它会不会走索引查询：

```sql
Copyselect  *  from test_tb where b = '10';
```

很显然根据 “左前匹配原则” 肯定不会走索引查询，最终还是全表扫描。

原因就在于联合索引的结构上。上面对 a,b,c 三个字段建立索引，那么对应的 B+ Tree 索引结构每个节点其实是按照三个字段的前后顺序排列的，即 a 字段检索在最前面，然后是b，然后是c。如果你的查询不是按照这个顺序来检索，是不会被这个索引识别的。

##### 左前匹配原则

上面说到联合索引会遵循左前匹配原则，那么什么是左前匹配呢？

其实就是字面意义上的从建立索引的第一个字段开始先匹配查询条件，如果当前查询条件不是第一个字段那么就不会走该索引。

另外对于联合索引的使用也有一些限制，比如说：

***遇到范围查询 ( > ,<, between, like) 就会停止匹配***

比如哦我们看这个 SQL：

```sql
Copyselect * from test_tb where a = '1' and b = '3' and d < 20 and c = '5';
```

大家觉得这个 SQL 会如何使用索引呢？

其实这 SQL 在前面a，b的查询中是会走联合索引的，但是在经历了d的查询之后，到了c就不会使用索引了，因为d的查询已经将索引的顺序打乱了，从 d 条件过后就没有办法直接使用联合索引。

***在索引列上做操作（函数，自定义计算）***

同样对索引列做计算也是无法直接应用索引，不言自明，索引是对已有的数据进行归纳排序，你计算之后的数据是新的内容，索引并没有包含这些数据，无从查起。

***查询条件包含 or，可能导致索引失效***

比如有一张 user 表：

```sql
CopyCREATE TABLE `user` (  
  `id` int(11) NOT NULL AUTO_INCREMENT,  
  `userId` int(11) NOT NULL,  
  `age` int(11) NOT NULL,  
  `name` varchar(255) NOT NULL,  
  PRIMARY KEY (`id`),  
  KEY `idx_userId` (`userId`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 
```

如下查询语句：

```sql
Copyselect * from user where userId=3 or age > 20;
```

大家觉得这句会走索引吗？

我们可以分析一下，如果 userId 查询走了索引这没问题，但是遇到 age > 20 这肯定没法走索引，那么前面 userId 走索引做一次 索引扫描就没有意义，所以优化器认为这种情况下不如一开始就走全表扫描更省事。

但是如果 or 前后的两个字段都加了索引那么可能会走，这种要依情况而定。

***字符串类型的索引字段作为查询条件时一定要用引号括起来，否则索引不生效***

上面我们没有说的一点是，B+Tree 索引结构的 key 都是用数字表示的，因为数字比较省空间，就算是字符串格式的字段，最终也会被转为二进制表示。但是对于不加引号的字符串，MSYQL 会自动做一次隐式转换将字符串转为浮点类型，这就导致不匹配。

***like 通配符可能导致索引失效***

使用 like 模糊查询并不是所有的都会失效，只有以 “%” 开头的 like 查询才会失效。

***左连接查询或者右连接查询查询关联的字段编码格式不一样，可能导致索引失效。***

这个比较不常见，一般来说同一个库中的表使用的编码格式应该是一样的，但是不排除老项目新老表有区别。

##### 索引的缺点

上面一直在谈论索引的优点，凡事有利就有弊，它也不是没有缺点的：

- 磁盘空间占用。这个对于当前磁盘比买菜还便宜的硬件大通货时代其实算不上问题，但是要注意的是如果当前 MySQL 服务所在的机器有很多的大表，并且还创建了每一种可能的组合的索引，那么索引文件提及的增长可能超乎你的想象。
- 维护索引对更新类操作所带来的耗时。当对索引涉及到的列做更新或者新增操作时都会去维护相关的索引，这里也是一个耗时的点，所以索引不在多，而在精。

##### 检查一条 SQL 是否是 bad SQL - 执行计划

在 MySQL 中如何知道一条 sql 到底有没有用到索引呢？MySQL 提供了 *explain* 关键字来查询一条 sql 的执行效率。

比如我们有一张 user 表：

```sql
CopyCREATE TABLE `user` (  
  `id` int(11) NOT NULL AUTO_INCREMENT,  
  `userId` int(11) NOT NULL,  
  `age` int(11) NOT NULL,  
  `name` varchar(255) NOT NULL,  
  PRIMARY KEY (`id`),  
  KEY `idx_userId` (`userId`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 
```

查询下面 sql 的查询效率：

```sql
Copymysql> explain select * from user where id = 3;
+----+-------------+-------+------+---------------+------+---------+------+--------+-------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows   | Extra |
+----+-------------+-------+------+---------------+------+---------+------+--------+-------+
|  1 | SIMPLE      | user  | const  | PRIMARY    | PRIMARY | 4    | const |     1 | NULL  |
+----+-------------+-------+------+---------------+------+---------+------+--------+-------+
1 row in set (0.04 sec)
```

执行计划各个字段的含义如下：

| 列名          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 执行序号，MSYQL 会按照从大到小的顺序执行                     |
| select_type   | 查询类型：SIMPLE: 简单查询 PRIMARY: 外层查询 SUBQUERY: 子查询 DERIVED: 派生查询（FROM 中包含的子查询） UNION: UNION 中第二个或后面的那个查询 UNION RESULT: UNION 的结果 |
| table         | 引用的表                                                     |
| partitions    | 所属分区                                                     |
| type          | 访问类型[官方文档](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-join-types)，常见访问类型： system : 只有一条记录的表（=系统表） const : 通过索引一次就查询到 eq_ref : 唯一索引等值扫描 ref : 非唯一索引等值扫描 range : 范围索引扫描 index : 索引扫描 all : 全表扫描 |
| possible_keys | 可能使用的索引（优化前）                                     |
| key           | 实际使用的索引（优化后）                                     |
| key_len       | 使用索引的长度                                               |
| ref           | 上述表的连接匹配条件（哪些列或常量被用于查找索引列上的值）   |
| rows          | 必须扫描的行数                                               |
| Extra         | 附加信息[官方文档](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-extra-information)，常见附加信息： Using filesort : mysql 无法利用索引完成排序操作 Using temporary : 使用了临时表保存中间结果 Using index : select 操作使用了覆盖索引 Using where : 使用 where 过滤 using join buffer : 使用了连接缓存 impossible where : where 子句的值总是 false，不能用来获取任何记录 distinct : 优化 distinct，在找到第一个匹配的记录后停止扫描同样值的动作 |

这么多字段我们挑几个重点来解释一下：

**id**

执行序号，id 列的编号是 select 的序列号，有几个 select 就有几个 id，并且 id 的顺序是按select 出现的顺序增长的。id 越大执行优先级越高，id 相同则从上往下执行，id 为 NULL 最后执行。

**key_len**

key_len 长度表示在索引里使用的字节数，通过这个值可以估算出具体使用了索引中的哪些列。

ken_len 计算规则如下：

- **字符串** ：char(n)：n 字节长度； varchar(n)：n 字节存储字符串长度，如果是 utf-8， 则长度是 3n+2，这里的长度与字符集有直接关系；
- **数值类型**：tinyint：1 字节；smallint：2 字节 ；int：4 字节； bigint：8字节；
- **时间类型** ：date：3字节；timestamp：4字节；datetime：8字节。

如果字段允许为 NULL，需要 1 字节记录是否为 NULL； 索引最大长度是 768 字节，当字符串过长时，MySQL 会做一个类似做前缀索引的处理，将前半部分的字符串提取出来做索引。

**type**

type 显示的是访问类型，结果值从好到坏依次是：

```txt
Copysystem > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL 
```

一般来说，得保证查询至少达到 range 级别，最好能达到 ref。

下面对这几个类型简要说明：

- *system*
  该表只有一行（如：系统表）。这是 const 连接类型的特例。

- *const*
  该表最多只有一个匹配行，在整个查询过程中这个表最多只会有一条匹配的行，用到了 primary key 或者unique 索引。

  比如主键查询肯定只有一条记录被匹配到。

- *eq_ref*
  对于前面表格中的每个行组合，从该表中读取一行。除了 system 和 const 类型之外，这是最好的连接类型。当连接使用索引的所有部分且索引是 索引 `PRIMARY KEY` 或 `UNIQUE NOT NULL` 索引时使用它。

  ```sql
  CopySELECT * FROM ref_table,other_table WHERE ref_table.key_column=other_table.column;
  ```

- *ref*
  表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值。

- *fulltext*
  使用 FULLTEXT 索引执行连接。

- *ref_or_null*

  该连接类型如同 ref，但是添加了 MySQL 可以专门搜索包含 NULL 值的行。

  ```sql
  CopySELECT * FROM ref_table WHERE key_column IS NULL;
  ```

- *index_merge*
  该连接类型表示使用了索引合并优化方法。

  ```sql
  CopySELECT * FROM tbl_name WHERE key1 = 10 OR key2 = 20;
  
  SELECT * FROM tbl_name
  WHERE (key1 = 10 OR key2 = 20) AND non_key = 30;
  ```

- *unique_subquery*
  此类型替换 以下形式的 eq_ref 某些 IN子查询：

  ```sql
  Copyvalue IN (SELECT primary_key FROM single_table WHERE some_expr)
  ```

- *index_subquery*
  此连接类型类似于 unique_subquery。它替换 IN 子查询，但它适用于以下形式的子查询中的非唯一索引：

  ```sql
  Copyvalue IN (SELECT key_column FROM single_table WHERE some_expr)
  ```

- *range*
  给定范围内的检索，使用一个索引来检查行。通常发生在在索引列上使用范围查询，如 >，<，in 等时，非索引列是 ALL。

- *index*
  按索引次序扫描，先读索引，再读实际的行，结果也是全表扫描，主要优点是避免了排序。（索引是排好序的，并且 all 是从硬盘中读的，index 可能不在硬盘上。s

- *ALL*
  对前面表格中的每个行组合进行全表扫描。如果表是第一个未标记的表 const，通常不好，并且在所有其他情况下通常 非常糟糕。通常，您可以ALL通过添加基于常量值或早期表中的列值从表中启用行检索的索引来避免

**row**

这一列是 MYSQL 估计要读取并检测的行数，注意这个不是结果集的行数。

**Extra**

Extra 是 EXPLAIN 输出中另外一个很重要的列，该列显示 MySQL 在查询过程中的一些详细信息。

- *Distinct*：MySQL 发现第 1 个匹配行后，停止为当前的行组合搜索更多的行。
- *Not exists*：MySQL 能够对查询进行 LEFT JOIN 优化，发现1个匹配 LEFT JOIN 标准的行后，不再为前面的的行组合在该表内检查更多的行。
- *range checked for each record*：MySQL 没有发现好的可以使用的索引，但发现如果来自前面的表的列值已知。可能部分索引可以使用。
- *Using filesort*：**看到这个的时候，查询就需要优化了。MySQL 需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行。**
- *Using index*：从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。
- *Using temporary*：**为了解决查询，MySQL 需要创建一个临时表来容纳结果。看到这个就需要进行优化了，这通常发生在对不同的列集进行 order by 上，而不是 group by 上。**
- *Using where*：WHERE 子句用于限制哪一个行匹配下一个表或发送到客户。
- *Using sort_union| Using union|Using intersect*：这些函数说明如何为 index_merge 联接类型合并索引扫描。
- *Using index for group-by*：类似于访问表的 Using index 方式，Using index for group-by 表示 MySQL 发现了一个索引，可以用来查询 GROUP BY 或 DISTINCT 查询的所有列，而不要额外搜索硬盘访问实际的表。

了解了执行计划，当你不确定一条 sql 查询效率的时候 就可以使用 Explain 来查看。

分类: [MYSQL 系列](https://www.cnblogs.com/rickiyang/category/1825438.html)

4

0

[« ](https://www.cnblogs.com/rickiyang/p/13520878.html)上一篇： [InnoDB引擎面面观](https://www.cnblogs.com/rickiyang/p/13520878.html)
[» ](https://www.cnblogs.com/rickiyang/p/13652664.html)下一篇： [一文说清 InnoDB 的事务机制](https://www.cnblogs.com/rickiyang/p/13652664.html)

posted @ 2020-08-25 14:26 [rickiyang](https://www.cnblogs.com/rickiyang/) 阅读(6186) 评论(0) [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=13559507) [收藏](javascript:void(0)) [举报](javascript:void(0))



# [一文说清 InnoDB 的事务机制](https://www.cnblogs.com/rickiyang/p/13652664.html)

我们从一个转账的故事开始。

隔壁小王从美团上找到了一家水饺店，准备中午吃水饺。下单成功，支付20元。

商家这里响了一下：叮叮，您有美团外卖新订单啦，请及时处理。水饺一份，好嘞，下锅。

很快小王吃到外卖了，吃完美美地躺下开始睡觉。

突然手机一顿猛响。一个陌生的号码打过来的，又是卖房的吧。小王想想没理他，继续睡。

可是这哥么锲而不舍，一会又打过来了。小王忍无可忍准备接过电话骂他一顿。刚接电话听到对面一阵急促的声音传来：你好你中午是不是点了一份我们店的水饺？

小王这才意识到感情是水饺店的。赶忙回复到是的啊，咋了。

老板说：你中午下单付款了吗？

小王：我肯定付款了啊，不然怎么下单。

老板说：我没收到钱啊。你把付款的截图发给我。

小王说：我吃饭还能不付钱吗，你等着。

于是小王给老板截图了，老板拿着截图去找了美团技术，美团技术一查，转账失败。跟老板说不好意思，今天这代码是实习生写的，我们马上开除他，稍后转给你。这时候老板一颗悬着的心才放下，可不能一天就卖一份水饺还没收到钱，这不亏大了呢！

以上纯属虚构，没有诋毁美团实习生的意思。

从上面的问题看，付款成功了，转账失败了，这时候用户吃到了饭，但是老板没收到钱。放在正常的堂食，你不先付款，估计人儿就的赶你出去，一手交钱一手交货买卖不变的道理。

我们引申出一个概念：**最小操作单元**。即我们人为定义了一个业务场景，这个场景中的操作要么全部成功，要么全部失败。

英语原文中把这种最小操作单元定义为：transaction ，在英语中的解释是：

> an occasion when someone buys or sells something, or when money is exchanged or the activity of buying or selling something:
>
> - a business transaction
> - Each transaction at the foreign exchange counter seems to take forever
> - We need to monitor the transaction of smaller deals.

通俗的说就是我们做某事所发生的这个时机或这个场景，代指这整个的发生过程。在 MySQL 中我们把 transaction 翻译为 **事务**，个人感觉中文意思总和英文有点不搭。

上面这个例子中我们可以了解到 transaction 存在的主要意图：

1. 在最小操作单元中保持稳定的操作，即使在故障时也能恢复到操作之前的状态保持数据一致性。
2. 保持各个最小操作单元之前互相隔离，以防止互相交互产生的覆盖性错误。

一般需要事务来控制的场景发生在：

**更新--插入--选择--插入--**

即一个最小操作单元中保持两个及以上的非查询操作。

事务结束的两种可能方式：

- commit：提交最小操作单元中的所有操作。
- terminate：操作终止，最小操作单元中所有修改无效。

数据库操作的环境：

- 共享-多用户并发访问
- 不稳定-潜在的硬件/软件故障

事务所需环境：

- 不共享 - 一个事务内的操作不受其他事务影响
- 稳定 - 即使面对系统故障，当前事务的操作也能保留现场

一个事务一旦开始，则必须确保：

- 所有操作必须可回溯
- 所有操作对后续操作的影响必须是可见的

一个事务开始的过程中必须确保：

在该事务结束之前其他事务看不到它的结果。

如果事务中止：

必须确保当前事务所有可能影响数据一致性的操作都会被清理。

如果系统出现故障：

必须确保重新启动时所有未提交的事务都会被清理。

针对以上事务操作过程中可能会出现的问题，抽象出事务如果满足以下条件，则可以保证数据完整性：

- Automicity（原子性）

  要么事务中的所有任务都必须发生，要么都不发生。

- Consistency（一致性）

  每个事务都必须保留数据库的完整性约束（已声明的一致性规则）。它不能使数据处于矛盾状态。在执行期间，一系列数据库操作不会违反任何完整性约束。

- Isolation（隔离性）

  两个同时进行的事务不能互相干扰。交易中的中间结果必须对其他交易不可见。其他一系列数据库操作无法看到一系列数据库操作的中间状态。

- Durability（持久性）

  已完成的事务以后不能中止或放弃其结果。它们必须在崩溃后通过（例如）重新启动DBMS持续存在。保证已提交的一系列数据库操作将永久保留。

特意查证了一下，关于事务四大特性的提出最早是在 1983 年由 Andreas Reuter 和 Theo Haerder 两位关系型数据库研发的鼻祖在论文：`Principles of transaction-oriented database recovery` 中提出。[论文链接](https://dl.acm.org/doi/10.1145/289.291)，感兴趣的可以下载来看看。

事务的 ACID 特性概念简单，但不是很好理解，主要是因为这几个特性不是一种平级关系：

- 只有满足一致性，事务的执行结果才是正确的。
- 在无并发的情况下，事务串行执行，隔离性一定能够满足。此时只要能满足原子性，就一定能满足一致性。 在并发的情况下多个事务并行执行，事务不仅要满足原子性，还需要满足隔离性，才能满足一致性。
- 事务满足持久化是为了能应对数据库崩溃的情况。

#### InnoDB 如何实现事务[#](https://www.cnblogs.com/rickiyang/p/13652664.html#1891252834)

鉴于 MyISAM 引擎不支持事务，支持事务的引擎只有 InnoDB，所以下面关于事务的讲解都是基于 InnoDB引擎。

在 InnoDB引擎中实现事务最重要的东西就是日志文件，保证事务的四大特性主要依靠这两大日志：

- redo log ：保证事务持久性
- undo log：回滚日志，保证事务原子性

两大日志系统分别保证了持久性和原子性，另外还有两大特性是通过什么来保证的呢？

一致性 和 隔离性 是通过 MVCC 机制 和 锁机制来一起控制。先提前介绍，后面我们详解讨论。

典型的事务操作会遵循如下流程：

```sql
Copystart transaction;
...... # do your business
commit;
```

`start transaction` 标识事务的开始，直到遇到 `commit` 才会提交事务。在该事务过程中如果出现问题，会自动调用 rollback 逻辑回滚该事物已完成的 sql。

**非显式开启事务**

MySQL 中默认采用的是自动提交的模式：

```sql
Copymysql > show variables like 'autocommit';
+------------------+-------+
|   Variable_name  | Value |
+------------------+-------+
|   autocomment    | ON    |
+------------------+-------+
```

自动模式下，你无需显式的输入 `start transaction` 作为开头和使用 `commit` 作为结尾来标识一个事务。每个sql 语句都会被作为一个事务提交。

当然你也可以关闭自动提交事务机制：

```sql
Copymysql > set autocommit = 0;
```

需要注意的是：`autocommit` 参数的修改指**只针对当前连接**，在一个连接中修改该属性并不会影响别的连接。

**不被 autocommit 影响的操作**

MySQL 中提供了一些不会被 autocommit 属性值所影响的特殊指令，这些指定即使在事务中执行，他们也会立刻执行而不是等到 commit 语句之后再提交，这些特殊指令包括：`DDL(create table / drop table / alter table)`、`lock tables`等等。

**我们探讨事务到底在探讨什么？**

事务的定义我们已经了解，无非就是把几个有上下文关联的 sql 放在一起操作要么全部成功，要么全部失败。道理很简单，那我们分析这么多到底在分析什么呢？貌似难的点不在于打包执行，在于如果让这些打包命中不互相影响，事务执行过程中失败如何回滚操作且不污染现有数据。这些才是我们讨论事务应该关注的地方。

这些问题的根本其实又回到了事务的四大特性，不得不说 Theo Haerder 在 1983 年就能抽象出来如此高度凝练的总结实在是让当下汗颜。

下面我就从 InnoDB 如何保证四大特性入手，逐一分析事务机制的实现。

#### 保证原子性的关键技术 - undo log[#](https://www.cnblogs.com/rickiyang/p/13652664.html#2553145002)

对于事务的原子性来说，该事务内所有操作要么全部成功要么全部失败就是事务的原子性。

全部成功这个毋庸置疑，如果中间突然失败，原子性该如何保证呢？是否该回滚当前已经执行成功的操作。

InnoDB 提供了一种日志：undo log，它有两个作用：提供 回滚 和 多个行版本控制(MVCC)。

比如一条 delete 操作在 undo log 中会对应一条 insert 记录，反之亦然。当 update 操作时，它会记录一条相反的 update 记录。

当执行 rollback 时，就可以从 undo log 中的逻辑记录读取到相应的内容并进行回滚。

有时候应用到行版本控制的时候，也是通过 undo log 来实现的：当读取的某一行被其他事务锁定时，它可以从 undo log 中分析出该行记录以前的数据是什么，从而提供该行版本信息，让用户实现非锁定一致性读取。

**undo log 的存储方式**

InnoDB 存储引擎对 undo log 的管理采用段的方式。rollback segment 称为回滚段，每个回滚段中有 1024 个 undo log slot 。

在以前老版本，只支持 1 个 rollback segment，这样就只能记录 1024 个 undo log slot。后来 MySQL5.5 可以支持 128 个 rollback slot，即支持 128 * 1024 个 undo log 操作。

MySQL5.6 之前，undo log 表空间位于共享表空间的回滚段中，共享表空间的默认的名称是 ibdata，位于数据文件目录中。
MySQL5.6 之后，undo log 表空间可以配置成独立的文件，但是提前需要在配置文件中配置，完成数据库初始化后生效且不可改变 undo log 文件的个数。如果初始化数据库之前没有进行相关配置，那么就无法配置成独立的表空间了。
MySQL5.7 之后的独立 undo log 表空间配置参数如下：

```text
Copyinnodb_undo_directory = /data/undospace/ #undo独立表空间的存放目录
innodb_undo_logs = 128 #回滚段为128KB
innodb_undo_tablespaces = 4 #指定有4个undo log文件
```

**undo log 的删除时机**

undo log 文件的个数是有限制的，所以不用无限堆积日志文件。undo log 记录的是当前事务操作的反向记录，理论上当前事务结束，undo log 日志就可以废弃。上面也提到过的多版本并发控制机制在隔离级别为 `repeatable read` 的时候事务读取的数据都是该事务最新提交的版本，那么只要该事务不结束，行版本记录就不能删除。

另外不同的 sql 语句对应的 undo log 类型也不一样，比如：

- insert 语句：因为 insert 操作本身只对该事务可见，事务提交之前别的连接是看不到的，所以 insert 操作产生的 undo log 日志在事务提交之后会马上直接删除，后续不会再被别的功能使用。
- update / delete 语句：delete 操作在事务中并不会真的先删除数据，而是将该条数据打上 “delete_bit” 标识，后续的删除操作是在事务提交后的 purge 线程独立操作。这两种操作产生的 undo log 日志都可以用反向的 update 来代替，这种操作上面说过 MVCC 机制可能会用上，所以就不能在事务结束之后直接删除。

在事务提交之后，也不是马上就删除该事务对应的 undo log 日志，而是将该事务对应的文件块放入到删除列表中，未来通过 purge 来删除。并且提交事务时，还会判断 undo log 分配的页是否可以重用，如果可以重用，则会分配给后面来的事务，避免为每个独立的事务分配独立的 undo log 页而浪费存储空间和性能。

#### 持久性 - redo log[#](https://www.cnblogs.com/rickiyang/p/13652664.html#2219740795)

redo log 即重做日志，重做日志记录每次操作的物理修改。

说 redo log 之前其实是要先说一下 binlog，不然就不知道为什么要引入 redo log。

bin log = binary log，二进制日志，它记录了除了 select 之外所有的 DDL 和 DML 语句。以事件形式记录，还包含语句所执行的消耗的时间，MySQL 的二进制日志是事务安全型的。

binlog日志有两个最重要的使用场景：

1. mysql 主从复制： mysql replication 在 master 端开启 binlog，master 把它的二进制日志传递给 slaves 来达到 master-slave 数据一致的目的。
2. 数据恢复： 通过 mysqlbinlog 工具来恢复数据。

binlog 日志包括两类文件：

1. 二进制日志索引文件（文件名后缀为 .index）用于记录所有的二进制文件。
2. 二进制日志文件（文件名后缀为 .00000*）记录数据库所有的 DDL 和 DML 语句事件。

binlog 文件是通过追加的方式写入的，可通过配置参数`max_binlog_size`设置每个 binlog 文件的大小，当文件大小大于给定值后，日志会发生滚动，之后的日志记录到新的文件上。
binlog 有两种记录模式，statement 格式的话是记 sql 语句，row 格式会记录行的内容。

持久性问题一般在发生故障的情况才会重视。在启动 MySQL 之后无论上次是否正常关闭都会进行恢复操作，我们假设现在没有 redo log 只有 binlog，那么数据文件的更新和写入 binlog 只有两种情况：

- 先更新数据文件，再写入 binlog；
- 先写入 binlog，再更新数据文件。

如果先更新数据文件，接着服务器宕机，则导致 binlog 中缺少最后的更新信息；如果先写 binlog 再更新数据则可能导致数据文件未被更新。

所以在只有 binlog 的环境中 MySQL 是不具备 crash-safe 的能力。另外一开始的 MySQL 使用 MyISAM 引擎，它只有 binlog，所以自然不支持事务。后面引入了 InnoDB 之后才开始使用另外一套日志系统- redo log 来实现 crash-safe 功能。

redo log 和 binlog 的区别：

- redo log 是 InnoDB 引擎特有的，binlog 是MySQL server 层实现的功能，与引擎无关。
- redo log 是物理日志，记录 “在某个数据页做了什么修改”；binlog 是逻辑日志，记录 sql 语句的原始逻辑，比如 “给 ID = 1 这一行的 name value set ‘xiaoming’ ”。
- redo log 空间是固定的，用完之后会覆盖之前的数据；binlog 是追加写，当前文件写完之后会开启一个新文件继续写。

redo log 由两部分组成：

- 内存中的重做日志缓冲(redo log buffer)
- 重做日志文件(redo log file)

**一个更新事务的整体流程**

![2](https://tva1.sinaimg.cn/large/007S8ZIlgy1gimsswons7j31gk0pgwjh.jpg)

从一个事务的更新过程出发看看一个事务更新过程中 redo log 处于什么地位。

1. 首先检查 Buffer cache 中是否存在这条数据，如果存在直接返回，如果不存在则去索引树中读取这条数据并加载到 Buffer Cache。
2. 执行器拿到这条行数据之后对它执行相应的更新操作。
3. 将这条待更新的行数据调用执行引擎更新到 Buffer Cache 中，同时将这个记录更新到 redo log 里面，redo log 包含两个部分的更新，更新完毕，此时 redo log 处于 prepare 的状态，然后告诉执行器，你可以提交事务。
4. 执行器生成这个操作的 binlog 日志，并把 binlog 写入磁盘。
5. 执行器调用引擎的提交事务接口，引擎把刚写入的 redo log 改为 commit 状态，整个事务提交完成。

这里我们注意到在 redo log 的提交过程中引入了**两阶段提交**。

**两阶段提交**

为什么必须有 “两阶段提交” 呢？这是为了让两份日志之间的逻辑一致。

前面我们说过了，binlog 会记录所有的逻辑操作，并且是采用 “追加写” 的形式。如果你的 DBA 承诺说半个月内可以恢复，那么备份系统中一定会保存最近半个月的所有binlog，同时系统会定期做整库备份。

由于 redo log 和 binlog 是两个独立的逻辑，如果不用两阶段提交，要么就是先写完 redo log 再写 binlog，或者采用反过来的顺序，我们看看这两种方式会有什么问题，用上面的 update 示例做假设：

1. **先写 redo log 后写 binlog**。假设在 redo log 写完，binlog 还没有写完的时候，MySQL 进程异常重启。因为 redo log 已经写完，系统即使崩溃仍然能够把数据恢复回来。但是 binlog 里面就没有记录这个语句，因此备份日志的时候 binlog 里面就没有这条语句。

   但是如果需要用这个 binlog 来恢复临时库的话，由于这个语句的 binlog 丢失，恢复出来的值就与原库值不同。

2. **先写 binlog 后写 redo log**。如果在 binlog 写完之后宕机，由于 redo log 还没写，崩溃恢复以后这个事务无效，所以这一行的值还是未更新以前的值。但是 binlog 里面已经记录了崩溃前的更新记录， binlog 来恢复的时候就多了一个事务出来与原库的值不同。

可以看到，两阶段提交就是为了防止 binlog 和 redo log 不一致发生。同时我们也注意到为了这个崩溃恢复的一致性问题引入了很多新的东西，也让系统复杂了很多，所以有得有失。

InnoDB通过 `Force Log at Commit` 机制保证持久性：当事务提交(COMMIT)时，必须先将该事务的所有日志缓冲写入到重做日志文件进行持久化，才能 COMMIT 成功。

为了确保每次日志都写入 redo log 文件，在每次将 redo log buffer cache 写入重做日志文件后，InnoDB 引擎都需要调用一次 fsync 操作。因此磁盘的性能决定了事务提交的性能，也就是数据库的性能。

`innodb_flush_log_at_trx_commit` 参数控制重做日志刷新到磁盘的策略：

- 0：事务提交时不进行写入重做日志操作，仅在 master thread 每秒进行一次。
- 1：事务提交时必须调用一次`fsync`操作。
- 2：仅写入文件系统缓存，不进行`fsync`操作。

log buffer 根据如下规则写入到磁盘重做日志文件中：

- 事务提交时。
- 当 log buffer 中有一半的内存空间已经被使用。
- log checkpoint 时，**checkpoint在一定程度上代表了刷到磁盘时日志所处的LSN位置。**

#### 一致性 和 隔离性实现 - 锁机制 和 MVCC[#](https://www.cnblogs.com/rickiyang/p/13652664.html#980734516)

实现一致性和隔离性是保证数据准确性的关键一环，前面两个特性保证数据恢复不出问题，这两个特性要保证数据插入和读取不出问题。实现一致性和隔离性主要使用了两个机制：

- 锁机制
- 多版本并发控制

下面我们就事务会产生哪些问题，MySQL 提出什么方式来解决问题，这些方式的实现方案又是什么来讲解。

##### 并发下事务会产生哪些问题

事务 A 和 事务 B 同时操作一个资源，根据不同的情况可能会出现不同问题，总结下来有以下几种：

- 脏读

  事务 A 读到了事务 B 还未提交的数据。

- 幻读

  在当前事务中发现了不属于当前事务操作的数据。幻读是针对数据 insert 操作来说的。假设事务A对某些行的内容作了更改，但是还未提交，此时事务 B 插入了与事务 A 更改前的记录相同的记录行，并且在事务 A 提交之前先提交了，而这时，在事务A中查询，会发现好像刚刚的更改对于某些数据未起作用，但其实是事务 B 刚插入进来的，让用户感觉出现了幻觉，这就叫幻读。

- 可重复读

  可重复读指的是在一个事务内，最开始读到的数据和事务结束前的任意时刻读到的同一批数据都是一致的。通常针对数据 update 操作。

- 不可重复读

  在同一个事务中两次读取一个数据的结果不一样。对比可重复读，不可重复读指的是在同一事务内，不同的时刻读到的同一批数据可能是不一样的，可能会受到其他事务的影响，比如其他事务改了这批数据并提交了。

##### 为什么会提出隔离级别的概念

为了解决事务并发过程中可能会产生的这些问题，SQL 标准定义的四种隔离级别被 ANSI（美国国家标准学会）和 ISO/IEC（国际标准）采用，每种级别对事务的处理能力会有不同程度的影响。

SQL 标准定义了四种隔离级别，MySQL 全都支持。这四种隔离级别分别是：

1. 读未提交（READ UNCOMMITTED）
2. 读提交 （READ COMMITTED）
3. 可重复读 （REPEATABLE READ）
4. 串行化 （SERIALIZABLE）

从上往下，隔离强度逐渐增强，性能逐渐变差。采用哪种隔离级别要根据系统需求权衡决定，其中，**可重复读**是 MySQL 的默认级别。

事务隔离其实就是为了解决上面提到的脏读、不可重复读、幻读这几个问题，下面展示了 4 种隔离级别对这三个问题的解决程度。

| 隔离级别 | 脏读     | 不可重复读 | 幻读     |
| -------- | -------- | ---------- | -------- |
| 读未提交 | 会发生   | 会发生     | 会发生   |
| 读提交   | 不会发生 | 会发生     | 会发生   |
| 可重复读 | 不会发生 | 不会发生   | 会发生   |
| 串行化   | 不会发生 | 不会发生   | 不会发生 |

只有串行化的隔离级别解决了全部这 3 个问题，其他的 3 个隔离级别都有缺陷。

##### 如何设置事务隔离级别

我们可以通过以下语句查看当前数据库的隔离级别，通过下面语句可以看出我使用的 MySQL 的隔离级别是 `REPEATABLE-READ`，也就是可重复读，这也是 MySQL 的默认级别。

```sql
Copymysql> show variables like 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.02 sec)

或者：

mysql> SELECT @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
1 row in set (0.00 sec)
```

当然我们也能手动修改事务的隔离级别：

```sql
Copyset [作用域] transaction isolation level [事务隔离级别];
作用域包含：
SESSION：SESSION 只针对当前回话窗口
GLOBAL：全局生效

隔离级别包含：
READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE
```

我们来测试一下各个隔离级别对事务的影响。

新建表：

```sql
CopyCREATE TABLE `test_db` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL DEFAULT '' COMMENT 'name',
  PRIMARY KEY (`id`),
  KEY `name_idx` (`name`(191))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=COMPACT COMMENT='测试表';
```

插入一些测试数据。

##### 读未提交（READ UNCOMMITTED）

首先设置事务隔离级别：

```sql
Copyset global transaction isolation level READ UNCOMMITTED;
```

注意：**设置完全局隔离级别只对新打开的 session 有效，历史打开的是不会受到影响的。**

首先关闭事务自动提交：

```sql
Copyset autocommit = 0;
```

开启事务 A：

```sql
CopyQuery OK, 0 rows affected (0.00 sec)

mysql>
mysql> insert test_db (name) values ('xiaocee');
Query OK, 1 row affected (0.01 sec)
```

在事务A 中插入一条数据，并未提交事务。

接着开启事务B：

```sql
Copymysql> select * from test_db;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaocee   |
+----+-----------+
9 rows in set (0.00 sec)
```

事务 B 中能够查到这条数据。**即不同的事务能读到对方未提交的数据**。连脏读都无法解决，可重复读和幻读更没法解决。

##### 读已提交

读已提交的数据肯定能解决脏读问题，但是对于幻读和不可重复读无法将解决。

首先设置事务隔离级别：

```sql
Copyset global transaction isolation level READ COMMITTED;
```

现在数据库数据如下：

```sql
Copymysql> select * from test_db;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming2 |
|  2 | xiaohong  |
|  3 | xiaowei   |
|  4 | xiaowei1  |
|  5 | xiaoli    |
|  6 | xiaoche   |
|  8 | xiaoche   |
| 10 | xiaoche   |
| 12 | xiaocee   |
+----+-----------+
9 rows in set (0.00 sec)
```

开启事务 A 将 `id=1` 的数据改为 “xiaoming3”：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_db set name = 'xiaoming3' where id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

这里事务 A 未提交，接着开启事务B 做第一次查询：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test_db where id = 1;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming2 |
+----+-----------+
9 rows in set (0.00 sec)
```

事务B查询还是原始值。

下面提交事务 A：

```sql
Copymysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

接着在事务 B 中再查询一次：

```sql
Copymysql> select * from test_db where id = 1;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming3 |
+----+-----------+
1 row in set (0.00 sec)
```

当然这次查到的肯定是人家已提交的数据。这里发生的问题就是不可重复读：即同一个事务内每次读取同一条数据的结果不一样。

##### 可重复读

可重复读隔离级别的含义就是重读每次都一样不会有问题。这就意味着一个事务不会读取到别的事务未提交的修改。但是这里就会有另一个问题：在别的事务提交之前它读到的数据不会发生变化，那么另一个事务如果将结果 a 改为 b，接着又改为了 a，对于当前事务来说直到另一个事务提交之后它再读才会获取到最新结果，但是它并不知道这期间别的事务对数据做了更新，**这就是幻读的问题**。

首先设置事务隔离级别：

```sql
Copyset global transaction isolation level REPEATABLE READ;
```

现在数据库数据如下：

现在数据库数据如下：

```sql
Copymysql> select * from test_db;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming3 |
|  2 | xiaohong  |
|  3 | xiaowei   |
|  4 | xiaowei1  |
|  5 | xiaoli    |
|  6 | xiaoche   |
|  8 | xiaoche   |
| 10 | xiaoche   |
| 12 | xiaocee   |
+----+-----------+
9 rows in set (0.00 sec)
```

开启事务 A 将 `id=1` 的数据改为 “xiaoming4”：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_db set name = 'xiaoming4' where id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

这里事务 A 未提交，接着开启事务B 做第一次查询：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test_db where id = 1;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming3 |
+----+-----------+
9 rows in set (0.00 sec)
```

事务B查询还是原始值。

下面提交事务 A：

```sql
Copymysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

接着在事务 B 中再查询一次：

```sql
Copymysql> select * from test_db where id = 1;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming3 |
+----+-----------+
1 row in set (0.00 sec)
```

查询到还是一样的结果，下面提交事务B ，然后再查询：

```sql
Copymysql> commit;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test_db where id = 1;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming4 |
+----+-----------+
1 row in set (0.00 sec)
```

提交完之后再查就是 “xiaoming4”。

这也意味着在事务B未提交期间，事务A做任何操作对B来说都是盲视的。

##### 串行化读

串行化读意味着将所有事务变为顺序执行，所以就不存在上述的四种问题，当然这也意味着效率是最低的。

有了隔离级别的概念，那隔离级别又是怎么实现的呢？我们接下来要讲的锁机制就是实现隔离级别的重要手段。

##### 锁的类型

从锁定资源的角度看， MySQL 中的锁分类：

- 表级锁
- 行级锁
- 页面锁

**表级锁** 的特点是每次都整张表加锁，加锁速度快，但是锁的粒度太大，并发性就低，发生锁冲突的概率大。

表锁的种类主要包含两种：

- 读锁 （共享锁）：同一份数据多个读操作同时进行不会互相影响，但是读操作会阻塞写操作。
- 写锁（排他锁）：当前写操作没有完成之前会阻塞其他读和写操作。

**行级锁** 的特点是对一行数据加锁，加锁的开销会大但是锁粒度小发生锁冲突的概率就低并发度提高了。

行锁的种类包含：

- 读锁（S 共享锁）：允许一个事务读取某一行，其他事务在读取期间无法修改该行数据但可以读。
- 写锁（X 排他锁）：允许当前获得排它锁的事务操作数据，其他事务在操作期间无法更改或者读取。
- 意向排它锁（IX）：一个事务给该数据行加排它锁之前，必须先获得 IX 锁。
- 意向共享锁（IS）：一个事务给该数据行加共享锁之前必须先获得 IS 锁。

**页面锁** 因为MySQL 数据文件存储是按照页去划分的，所以这个锁是 MySQL 特有的。开销和加锁时间界于表锁和行锁之间，锁定粒度界于表锁和行锁之间，并发度一般。

在 InnoDB 引擎中默认使用行级锁，我们重点就行级锁的加锁、解锁来做一些说明。

行级锁上锁分为 隐式上锁 和 显式上锁。

隐式上锁是默认的上锁方式，`select`不会自动上锁，`insert`、`update`、`delete` 都会自动加排它锁。在语句执行完毕会释放。

显式上锁即通过手动的方式给 sql 语句加锁，比如：

共享锁：

```sql
Copyselect * from tableName lock in share mode;
```

排他锁：

```sql
Copyselect * from tableName for update;
```

##### 行级锁的实现方式

在 InnoDB 中行级锁的具体实现分为三种类型：

- 锁定单个行记录：Record Lock。
- 锁定一个范围，不包含记录本身：Gap Lock。
- 同时锁住一行数据 + 该数据上下浮动的间隙 ：next-Key Lock。

接下来我们通过一个示例来测试 InnoDB 中这三种锁的实现。

先创建一个测试表：

```sql
CopyCREATE TABLE `test_db` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL DEFAULT '' COMMENT 'name',
  PRIMARY KEY (`id`),
  KEY `name_idx` (`name`(191))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=COMPACT COMMENT='测试表';
```

插入两条数据：

![3](https://tva1.sinaimg.cn/large/007S8ZIlgy1gimssvpvj7j30h8072jri.jpg)

还记得我们上面说过 MySQL 是自动提交事务，为了测试锁我们需要关闭自动提交：

```sql
Copyset autocommit = 0;
```

这个设置只在当前连接中生效，记得每开一个连接都要设置一下。

**Record Lock 测试**

开启一个事务：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> update test_db set name = 'xiaoming1' where id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

查看事务状态：

```txt
Copymysql> show engine innodb status;
------------
TRANSACTIONS
------------
Trx id counter 25355
Purge done for trx's n:o < 0 undo n:o < 0 state: running but idle
History list length 0
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 283540073944880, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 25354, ACTIVE 40 sec
2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 5, OS thread handle 12524, query id 757 localhost ::1 root starting
show engine innodb status
--------
```

事务状态显示有一行在被锁定。

下面我们在当前连接中查询一下现在的数据库：

```sql
Copymysql> select * from test_db;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming1 |
|  2 | xiaohong  |
+----+-----------+
2 rows in set (0.00 sec)
```

发现当前数据库已经被修改了，是事务并没有提交。别急我们继续看看。

下面在一个新的连接开启第二个事务：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> mysql>  update test_db set name = 'xiaoming2' where id = 1;
```

这时候发现这一条语句卡住了无法执行。

查看事务状态：

```txt
Copymysql> show engine innodb status;
......

------- TRX HAS BEEN WAITING 6 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2 page no 4 n bits 72 index PRIMARY of table `test_db`.`test_db` trx id 2072 lock_mode X locks rec but not gap waiting
Record lock, heap no 4 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 000000000817; asc       ;;
 2: len 7; hex 02000001080422; asc       ";;
 3: len 9; hex 7869616f6d696e6732; asc xiaoming2;;

------------------
---TRANSACTION 2071, ACTIVE 50318 sec
2 lock struct(s), heap size 1136, 1 row lock(s), undo log entries 2
MySQL thread id 10, OS thread handle 123145423929344, query id 96 localhost 127.0.0.1 root starting
show engine innodb status
Trx read view will not see trx with id >= 2073, sees < 2072
```

从事务状态上可以看到对 id = 1 的这一行加了 record lock。

再看这一句：

> trx id 2072 lock_mode X locks rec but not gap waiting

X 锁就是我们上面说的排它锁，只对当前记录加锁，并不对间隙加锁。

**Gap Lock 测试**

测试 Gap Lock 我发现如果 where 条件是主键的时候，只会有 record lock 不会有gap lock。

所以 gap lock 的条件是 where 条件必须是非唯一键。

首先查询一下当前的数据:

```sql
Copymysql> set autocommit = 0;
Query OK, 0 rows affected (0.00 sec)

mysql>
mysql> select * from test_db;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | xiaoming4 |
|  2 | xiaohong  |
|  3 | xiaowei   |
|  4 | xiaowei1  |
|  5 | xiaoli    |
|  6 | xiaoche   |
| 10 | xiaohai   |
| 12 | xiaocee   |
+----+-----------+
8 rows in set (0.00 sec)
```

开启事务A：

```sql
Copymysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test_db where name ='xiaohai' for update;
+----+---------+
| id | name    |
+----+---------+
| 10 | xiaohai |
+----+---------+
1 row in set (0.00 sec)
```

这里我们做的事情是对 name 列做查询条件，它是非唯一索引可以被间隙锁命中。现在的 `id=10` 列 `name=xiaohai`，如果被间隙锁命中的话，`xiaoc*` -- `xiaoh*`中间的字符应该都是不能插入的。所以我们就用这种方式来试试。

开启事务B：

```sql
Copymysql> set autocommit = 0;
Query OK, 0 rows affected (0.00 sec)

mysql> insert test_db (id, name) values (8, 'xiaodai');
```

插入“xiaodai”，可以发现“卡住了”，查询一下事务状态：

```sql
Copymysql> show engine innodb status;
------------
TRANSACTIONS
------------
......
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s), undo log entries 1
MySQL thread id 32, OS thread handle 123145425444864, query id 385 localhost 127.0.0.1 root update
insert test_db (id, name) values (8, 'xiaodai')
------- TRX HAS BEEN WAITING 24 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2 page no 5 n bits 80 index name_idx of table `test_db`.`test_db` trx id 2133 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 7; hex 7869616f686169; asc xiaohai;;
 1: len 4; hex 8000000a; asc     ;;
......
------------------
```

这里的事务日志说了在插入之前这个索引已经被gap lock 锁住了，所以我们的测试是有效的。

那么 gap lock 的边界是多少呢？这里我实测是当前记录往前找到一个边界和往后找到一个边界，对于上面的测试数据来说就是：往前到 "xiaoche" ，往后到 “xiaohong”， 且你再插入一个等于当前锁定记录 “xiaohai” 的值也是可以的，这个就留给大家动手试试。

Gap Lock 解决了什么问题呢？上面我们说到 读已提交级别有不可重复读的问题。Gap Lock 就是为了防止在本事务还未提交之前，别的事务在当前事务周边插入或修改了数据造成读不一致。

**Next-key Lock 测试**

Next-key Lock 实际上是 Record Lock 和 gap Lock 的组合。

Next-key Lock 是在下一个索引记录本身和索引之前的 gap Lock 加上 S 锁或是 X 锁 ( 如果是读就加上 S 锁，如果是写就加 X 锁)。

**默认情况下，InnoDB 的事务隔离级别为 RR，系统参数 `innodb_locks_unsafe_for_binlog=false`。InnoDB 使用 next-key Lock 对索引进行扫描和搜索，这样就读取不到幻象行，避免了幻读的发生。**

这就相当于对当前数据和当前数据周围的数据都做了保护，当前数据不会发生幻读，当前数据周围的数据不会出现修改或新增从而导致读不一致。

但是需要注意的是，上面测试 Gap Lock 也说过，Gap Lock 只对非唯一索引列生效，同样 Next-key Lock如果也是作用于非唯一索引那么会自动降级为 Record Lock。

##### MVCC机制

什么是 MVCC?

MVCC，`Multi-Version Concurrency Control`，多版本并发控制。同一份数据临时保留多版本的一种方式，进而实现并发控制，简称一致性非锁定读。

上面我们讨论过在多个事务的场景下，通过锁机制可以保证当前事务读不到未提交的事务。但是加锁也会带来坏处，那就是阻塞，只有读读之间可以并发，读写，写读，写写都不能并发操作。引入多版本机制就是为了解决这个问题，减少阻塞时间，通过这个机制，只有写写是会阻塞，其余情况都不会阻塞操作。

比如我们还用 RR 隔离级别下的例子来说，事务A写了一个数据未提交，事务B读取数据，这时候是读不到A事务未提交的记录。B事务只能读到A事务未提交之前的版本。这里就使用了版本管理机制，每个连接在某个瞬间看到的是是数据库在当前的一个快照，每个事务在提交之前对其他的读者来说是不可见的。

一般来说 MVCC 只在 Read Committed 和 Repeatable Read 两个隔离级别下工作。Read Uncommitted 总是能读取到未提交的记录，不需要版本控制；Serializable 对所有的读取都对加锁，单独靠 MVCC 无法完成。

MVCC 的实现，是通过保存数据在某一个时间点的快照来实现的。因此每一个事务无论执行多长时间看到的数据，都是一样的。所以 MVCC 实现可重复读。

**MVCC 的实现**

**隐藏字段**

为了实现多版本控制，InnoDB 引擎在每一行数据中都添加了几个隐藏字段：

- `DB_TRX_ID`：记录最近一次对本记录做（insert/upadte）的事务 ID，大小为 6 字节；
- `DB_ROLL_PTR`：回滚指针，指向回滚段的 undo log，大小为 7 字节；
- `DB_ROW_ID`：单调递增的行 ID，大小为 6 字节，当表没有主键索引或者非空唯一索引的时候 InnoDB 就用这个字段创聚簇索引，这个字段跟MVCC的实现没有关系。

MVCC 在 InnoDB 的实现依赖 undo log 和 read view。undo log 中记录的是数据表记录行的多个版本，也就是事务执行过程中的回滚段,其实就是MVCC 中的一行原始数据的多个版本镜像数据。read view 主要用来判断当前版本数据的可见性。

**undo log**

undo log 上面讲解的时候说go会用于 MVCC 机制。因为 undo log 中存储的是老版本的数据，如果一个事务读取当前行，但是当前行记录不可见，那么可以顺着 undo log 链表找到满足其可见性的版本。

版本链

每条 undo log 也都有一个 old_trx_id 属性和一个 old_roll_pointer 属性（INSERT 操作对应的 undo log 没有这些属性，因为该记录没有更早的版本）用于记录上一个 undo log。最终这些 undo log 就连接起来形成了一个链表，这个链表称之为版本链，版本链的头节点就是当前记录的最新值。

**Read View（读视图）**

如果一个事务修改了记录但尚未提交，其他事务是不能读取记录的最新版本的。此时就需要判断版本链中的哪个版本是可以被当前事务访问的，为此 InnoDB 提出了 ReadView 的概念。 Read View 里面保存了“**对本事务不可见的其他活跃事务**”，主要是用来做可见性判断。

Read View 底层定义了一些关键字段：

| ReadView 字段  | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| trx_ids        | 在生成 ReadView 时当前系统中活跃的读写事务，即Read View初始化时当前未提交的事务列表。所以当进行RR读的时候，trx_ids中的事务对于本事务是不可见的（除了自身事务，自身事务对于表的修改对于自己当然是可见的）。理解起来就是创建RV时，将当前活跃事务ID记录下来，后续即使他们提交对于本事务也是不可见的。 |
| low_limit_id   | 在生成 ReadView 时当前系统中活跃的读写事务中最小的事务 ID，事务号 >= low_limit_id 的记录，对于当前 Read View 都是不可见的 |
| up_limit_id    | 系统应该给下一个事务分配的 ID 值，事务号 < up_limit_id ，对于当前Read View都是可见的 |
| creator_trx_id | 生成该 ReadView 的事务 ID                                    |

**一旦一个 Read View 被创建，这三个参数将不再发生变化，**理解这点很重要，其中 low_limit_id 和 up_limit_id 分别是 trx_Ids 数组的上下界。

**记录行修改的具体流程**

1. 首先当前事务对记录行加排他锁；
2. 然后把该行数据拷贝到 undo log 中，作为旧版本；
3. 拷贝完毕后，修改该行的数据，并且修改记录行最新的修改事务 id ，也就是 DB_TRX_ID 为当前事务 id；
4. 事务提交，提交前用 CAS 机制判断记录行当前最新修改的事务 id 是否发生了变化，如果没变，则提交成功；如果变了，说明存在其他事务修改了这个记录行，那么就应该回滚这个事务。也就是当前事务没有生效。

**记录行查询时的可见性判断算法**

在 InnoDB 中创建一个新事务后，执行第一个 select 语句的时候，InnoDB 会创建一个快照(ReadView)，快照中会保存系统当前不应该被本事务看到的其他活跃事务 id 列表（即trx_ids）。当用户在这个事务中要读取某个记录行的时候，InnoDB 会将该记录行的 DB_TRX_ID 与该 ReadView 中的一些变量进行比较，判断是否满足可见性条件。

具体的比较算法如下:

1. 如果 trx_id < up_limit_id, 那么表明 “最新修改该行的事务” 在 “当前事务” 创建快照之前就提交了，所以该记录行的值对当前事务是可见的。直接标识为可见，返回 true；
2. 如果 trx_id >= low_limit_id, 那么表明 “最新修改该行的事务” 在 “当前事务” 创建快照之后才被创建且修改该行的，所以该记录行的值对当前事务不可见。应该通过回滚指针找到上个记录行版本，判断是否可见。循环往复，直到可见；
3. 如果 up_limit_id <= trx_id < low_limit_id, 那就得通过二分查找判断 trx_id 是否在 trx_ids 列表出现过。
   1. 如果出现过，说明是当前 read view 中某个活跃的事务提交了，那当然是不可见的，应该通过回滚指针找到上个记录行版本，判断是否可见，循环往复，直到可见；
   2. 如果没有出现过，说明这个事务是已经提交了的，表示为可见，返回 true。

**需要注意的是，新建事务(当前事务)与正在内存中 commit 的事务不在活跃事务链表中。**

**不同隔离级别下 read view 生成原则**：

RC 级别

每个快照读操作都会生成最新的 read view，所以在 RC 级别中能看到别的事务提交的记录。

RR 级别

同一个事务中的第一个快照读才会创建 Read View, 之后的快照读获取的都是同一个Read View。

**关于MVCC 的总结**

上面介绍了 MVCC 在 innoDB 中的实现，我们回顾一下理想中的 MVCC 应该是什么样的：

- 每行数据都有一个版本，每次更新都更新该版本
- 每个事务只在当前版本上更新，各个事务无干扰
- 多个事务提交时比较版本号，如果成功则覆盖原纪录，否则放弃

MVCC 的理论听起来和 乐观锁一致。但是反观 InnoDB 中的实现，事务修改数据首先借助排它锁，事务失败还借助到 undo log 来实现回滚。理论上如果一个完整的 MVCC 实现应该借助版本号就可以，如果使用上了 X 锁那何必还浪费时间再使用 乐观锁呢？

事实上理想的 MVCC 可能会引发bug，单纯依靠版本控制无法完成一致性非锁定读。任何一个复杂的系统在掺杂各种变量的情况总会引发一些旁支问题。

比如，在理想的MVCC 模式下，TX1执行修改 Row1成功，修改Row2失败，此时需要回滚Row1；

但因为Row1没有被锁定，其数据可能又被 TX2 修改，如果此时回滚 Row1的内容，则会破坏 TX2 的修改结果。

MVCC 机制提供了读的非阻塞能力，对于写来说如果不用锁，肯定会出错。但是对于数据库系统来说，读才是大头，这已经解决了生产力的要求。

#### 总结[#](https://www.cnblogs.com/rickiyang/p/13652664.html#3168558274)

以上从数据库多事务并发可能会产生什么问题分析，数据库奠基者总结出事务的四大特性，为了性能和数据准确性的协调总结出不同的隔离级别，为了实现不同的隔离级别分别使用了什么技术。这些问题环环相扣，从问题出发寻找解决思路，锁机制，MVCC机制，都是为了解决数据一致性问题同时兼顾读效率而存在。为了持久性提出了两阶段提交弄出了 redo log，为了实现 原子性 和 MVCC 又多出了 undo log。所有的实现都是基于特定的场景和需求，站在需求场景下去理解这些概念就会更容易感受到设计者的初衷。

# [趁热打铁-再谈分布式事务](https://www.cnblogs.com/rickiyang/p/13704868.html)

继上一篇讲 MySQL InnoDB 下的事务之后我们趁热打铁，继续跟进分布式事务。

分布式事务主要解决分布式一致性的问题。说到底就是数据的分布式操作导致仅依靠本地事务无法保证原子性。与单机版的事务不同的是，单机是把多个命令打包成一个统一处理，分布式事务是将多个机器上执行的命令打包成一个命令统一处理。

上一篇我们讲 InnoDB 下的事务，MySQL 提供了redo log，undo log， Read View，两阶段提交，MVCC 机制等等来保障事务的安全。分布式事务是不是更难呢？拭目以待。

#### 常见的分布式事务场景[#](https://www.cnblogs.com/rickiyang/p/13704868.html#1372709517)

分布式事务其实就在我们身边，你一直在用，但是你却一直不注意它。

##### 转账

扣你账户的余额，增加别人账户余额，如果只扣了你的，别人没增加这是失败；如果没扣你的钱别人也增加了那银行的赔钱。

##### 下订单/扣库存

电商系统中这是很常见的一个场景，用户下单成功了，店家没收到单，不发货；用户取消了订单，但是店家却看到了订单，发了货。

##### 分库分表场景

当我们的数据量大了之后，我们可能会部署很多独立的数据库，但是你的一个逻辑可能会同时操作很多个数据库的表，这时候该如何保证所有的操作要么成功，要么失败。

##### 分布式系统调用问题

微服务的拆分让各个系统各司其职，但是带来的也有很多痛苦，一个操作可能会伴随很多的外部请求，如果某一个外部系统挂掉了，之前操作已完成的这些是否需要回滚。

针对上面这些问题，我们前面学过的数据库4大特性：ACID 似乎在这里想要达到就变得很困难，单机情况下你还可以通过锁和日志机制来控制数据，在分布式场景又该如何实现呢？在不同的分布式应用架构下，实现一个分布式事务要考虑的问题并不完全一样，比如对多资源的协调、事务的跨服务传播等，实现机制也是复杂多变。尽管有这么多工程细节需要考虑，但分布式事务最核心的还是其 ACID 特性，只是这种 ACID 变换了场景。

#### 分布式理论[#](https://www.cnblogs.com/rickiyang/p/13704868.html#2423163225)

##### CAP 定理

传统的 ACID 模型肯定无法解决分布式环境下的挑战，基于此加州大学伯克利分校 Eric Brewer教授提出 CAP 定理，他指出 现代 WEB 服务无法同时满足以下 3 个属性：

- 一致性(Consistency) ： 所有的客户端都能返回最新的操作。
- 可用性(Availability) ： 非故障的节点在合理的时间内返回合理的响应(不是错误和超时的响应)。
- 分区容错性(Partition tolerance) ： 即使出现单个组件无法可用，操作依然可以完成。

关于一致性的理解后面分出来三个方向：

- 强一致：任何一次读都能读到某个数据的最近一次写的数据。系统中的所有进程，看到的操作顺序，都和全局时钟下的顺序一致。简言之，在任意时刻，所有节点中的数据是一样的。
- 弱一致：数据更新后，如果能容忍后续的访问只能访问到部分或者全部访问不到，则是弱一致性。
- 最终一致：不保证在任意时刻任意节点上的同一份数据都是相同的，但是随着时间的迁移，不同节点上的同一份数据总是在向趋同的方向变化。简单说，就是在一段时间后，节点间的数据会最终达到一致状态。

关于一致性的理解不同，后面对于 CAP 理论的实现就有所不同。

另外 Eric Brewer教授指出现代 WEB 服务无法同时满足这 3 个属性，说的是无法同时满足，那这是为什么呢？

如果在某个分布式系统中无副本，那么必然满足强一致性，同时也满足可用性，但是如果这个数据宕机了，那么可用性就得不到保证。

如果一个系统满足 AP，那么一致性又得不到保证。所以 CAP 原则的精髓就是要么 AP，要么 CP，要么 AC，但是不存在 CAP。

##### BASE 定理

基于前面提到的 CAP，反正我们都无法同时满足CAP，设计系统的最高目的就是让他跑下去不出错，那么是不是可以不要求强一致性，最终让他一致即可。所以后面又提出来了 BASE 定理：

- Basically Available（基本可用）
- Soft state（软状态）
- Eventually consistent（最终一致性）

基于现在大型分布式系统的复杂性，我们无法保证服务永远达到999，那么是否可以取舍，核心服务达到999，非核心服务允许挂为了保全核心服务。另外在非核心服务 down 机过程中允许系统暂时出现不一致但是这个不一致并不影响系统的核心功能使用。

最终系统恢复，所有服务一起修复数据，最终达到一致的状态。

业内通常把严格遵循 ACID 的事务称为**刚性事务**，而基于 BASE 思想实现的事务称为**柔性事务**。柔性事务并不是完全放弃了 ACID，仅仅是放宽了一致性要求：事务完成后的一致性严格遵循，事务中的一致性可适当放宽。

#### 常见的分布式事务实现方案[#](https://www.cnblogs.com/rickiyang/p/13704868.html#3808151109)

分布式事务实现方案从类型上去分刚性事务、柔型事务。刚性事务：通常无业务改造，强一致性，原生支持回滚/隔离性，低并发，适合短事务。柔性事务：有业务改造，最终一致性，实现补偿接口，实现资源锁定接口，高并发，适合长事务。

- 刚性事务：XA 协议（2PC、JTA、JTS）、3PC
- 柔型事务：TCC/FMT、Saga（状态机模式、Aop模式）、本地事务消息、消息事务（半消息）、最多努力通知型事务

##### 两阶段提交(XA)

与本地事务一样，分布式事务场景下也可以采用两阶段提交的方案来实现。XA 的全称是 eXtended Architecture，它是一个分布式事务协议，通过二阶段提交协议保证强一致性。

XA 规范中定义了分布式事务处理模型，这个模型中包含四个核心角色：

- RM (Resource Managers)：资源管理器，提供数据资源的操作、管理接口，保证数据的一致性和完整性。最有代表性的就是数据库管理系统，当然有的文件系统、MQ 系统也可以看作 RM。
- TM (Transaction Managers)：事务管理器，是一个协调者的角色，协调跨库事务关联的所有 RM 的行为。
- AP (Application Program)：应用程序，按照业务规则调用 RM 接口来完成对业务模型数据的变更，当数据的变更涉及多个 RM 且要保证事务时，AP 就会通过 TM 来定义事务的边界，TM 负责协调参与事务的各个 RM 一同完成一个全局事务。
- CRMs (Communication Resource Managers)：主要用来进行跨服务的事务的传播。

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cwjfpyj31400tw439.jpg)

XA 协议大概的两个流程为：

1. 第一阶段（prepare）：事务管理器向所有本地资源管理器发起请求，询问是否是 ready 状态，所有参与者都将本事务能否成功的信息反馈发给协调者；
2. 第二阶段 (commit/rollback)：事务管理器根据所有本地资源管理器的反馈，通知所有本地资源管理器，步调一致地在所有分支上提交或者回滚。

XA 协议是如何满足 ACID 的呢？

原子性和持久性我们就不用说，我们看看隔离性和一致性。

**隔离性**

XA 协议中没有描述如何实现分布式事务的隔离性，但是 XA 协议要求每个资源管理器都要实现本地事务，也就是说基于 XA 协议实现的分布式事务的隔离性是由每个资源管理器本地事务的隔离性来保证的，当一个分布式事务的所有子事务都是隔离的，那么这个分布式事务天然的就实现了隔离性。

**一致性**

在单机环境下的一致性就是保证当前服务器数据一致即可。事务执行完毕数据最终一致，不同的隔离级别下事务执行过程的中间状态不能被别的事务观察到。

事务执行完毕最终一致这个好保证，但是在RR 隔离级别下不可见一个未提交事务的中间态在分布式情况该如何做到呢？单机上 MySQL 提供了MVCC机制，采用多版本控制来处理，那分布式事务场景也是否也可以提供这样的机制呢？XA 协议并没有定义怎么实现全局的快照，一个基本思路是用一个集中式或者逻辑上单调递增的东西来控制生成全局 Snapshot，每个事务或者每条 SQL 执行时都去获取一次，从而实现不同隔离级别下的一致性。当然开发的难度还是挺大。

存在的问题：

- 同步阻塞：当参与事务者存在占用公共资源的情况，其中一个占用了资源，其他事务参与者就只能阻塞等待资源释放，处于阻塞状态。
- 单点故障：一旦事务管理器出现故障，整个系统不可用。
- 数据不一致：在阶段二，如果事务管理器只发送了部分 commit 消息，此时网络发生异常，那么只有部分参与者接收到 commit 消息，也就是说只有部分参与者提交了事务，使得系统数据不一致。
- 不确定性：当事务管理器发送 commit 之后，并且此时只有一个参与者收到了 commit，那么当该参与者与事务管理器同时宕机之后，重新选举的事务管理器无法确定该条消息是否提交成功。

总体来说 XA 方案实现简单，但是带来的问题如果放在数据一致性要求严格的场景是无法保证数据正确性的。另外事务管理器单点会带来隐患，同步阻塞模型也致使并发能力弱。

##### TCC

关于 TCC（Try-Confirm-Cancel）的概念，最早是由 Pat Helland 于 2007 年发表的一篇名为《Life beyond Distributed Transactions:an Apostate’s Opinion》的论文提出。 TCC 事务机制相比于上面介绍的 XA，解决了其几个缺点：

1. 解决了协调者单点，由主业务方发起并完成这个业务活动。业务活动管理器也变成多点，引入集群。
2. 同步阻塞：引入超时，超时后进行补偿，并且不会锁定整个资源，将资源转换为业务逻辑形式，粒度变小。
3. 数据一致性，有了补偿机制之后，由业务活动管理器控制一致性。

TCC 其实就是采用的补偿机制，其核心思想是：针对每个操作，都要注册一个与其对应的确认和补偿（撤销）操作。TCC 模型完全交由业务实现，每个子业务都需要实现 Try-Confirm-Cancel 三个接口，对业务侵入大，资源锁定交由业务方。

- Try 阶段：尝试执行，完成所有业务检查（一致性）, 预留必须业务资源（准隔离性）。
- Confirm 阶段：确认执行真正执行业务，不作任何业务检查，只使用 Try 阶段预留的业务资源，Confirm 操作满足幂等性。要求具备幂等设计，Confirm 失败后需要进行重试。
- Cancel 阶段：取消执行，释放 Try 阶段预留的业务资源 Cancel 操作满足幂等性 Cancel 阶段的异常和 Confirm 阶段异常处理方案基本上一致。

![5](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cx10mhj30z60qq791.jpg)

一个完整的业务活动由一个主业务服务与若干子业务服务组成：

1. 主业务服务负责发起并完成整个业务活动；
2. 业务服务提供 TCC 型业务操作；
3. 业务活动管理器控制业务活动的一致性，它登记业务活动中的操作，并在业务活动提交时确认所有的TCC 型操作的 Confirm 操作，在业务活动取消时调用所有 TCC 型操作的 Cancel 操作。

比如一个转账操作：

1. 首先在 Try 阶段先把转账者的钱包冻结起来。
2. 在 Confirm 阶段，调用转账接口操作转账，转账成功后解冻。
3. 如果 Confirm 阶段成功那么就转账成功，否则执行转账失败确认逻辑。

基于 TCC 实现分布式事务，会将原来只需要一个接口就可以实现的逻辑拆分为 Try、Confirm、Cancel 三个接口，所以代码实现复杂度相对较高，需要在业务中写很多的补偿机制代码。

TCC将事务提交划分成两个阶段，Try即为一阶段，Confirm 和 Cancel 是二阶段并行的两个分支，二选一。从阶段划分上非常像2PC，我们是否可以说TCC是一种2PC或者2PC变种呢？

对比一下 XA 事务模型，TCC 的两阶段提交与 XA 还是有一些区别：

1. 2PC 的操作对象在于资源层，对于开发人员无感知；而 TCC 的操作在于业务层，具有较高开发成本。
2. 2PC 是一个整体的长事务，也是刚性事务；而 TCC 是一组的本地短事务，是柔性事务。
3. 2PC 的 Prepare (表决阶段)进行了操作表决；而 TCC 的 Try 并没有表决准备，直接兼备资源操作与准备能力。
4. 2PC 是全局锁定资源，所有参与者阻塞 交互等待 TM 通知；而 TCC 的资源锁定在于 Try 操作，业务方可以灵活选择业务资源的锁定粒度。

#### 本地消息表[#](https://www.cnblogs.com/rickiyang/p/13704868.html#1452065802)

本地消息表最初是由eBay架构师Dan Pritchett在一篇解释 BASE 原理的论文《Base：An Acid Alternative》（https://queue.acm.org/detail.cfm?id=1394128）中提及的，业界目前使用这种方案是比较多的，其核心思想是将分布式事务拆分成本地事务进行处理。

方案通过在事务主动发起方额外新建事务消息表，事务发起方处理业务和记录事务消息在本地事务中完成，轮询事务消息表的数据发送事务消息，事务被动方基于消息中间件消费事务消息表中的事务。

基于本地消息表的方案，每个事务发起方都需要额外新建事务消息记录表，用于记录分布式事务的消息的发生、处理状态。

![3](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cxytg1j31n80p6wk0.jpg)

事务发起方在处理完业务逻辑之后需要将当前事务保存在消息表中，之后将消息发送到消息中间件中，并将消息的状态设置为 “发送中”。

如果消息在投递过程中丢失怎么办呢？事务发起方可以设置一个定时任务主动扫描状态为 “发送中” 的消息重新投送。只有消息被业务方消费完毕返回消费成功的结果才确认成功并将消息状态改为“已发送”。

这里因为网络异常或者发送异常导致一个消息可能会被重复发送，所以要求接收方要做到幂等性，允许重复消费。

这种方案的好处就是方案简单，成本较低，实现也不复杂。

但是坏处也有很多，比如通过消息的方式延迟不好控制；本地消息表与业务耦合在一起没有做到通用性；本地消息表基于数据库来实现，有一定的瓶颈。

#### 事务消息[#](https://www.cnblogs.com/rickiyang/p/13704868.html#2220917445)

上面说的本地消息表的模式无法支持本地事务执行和消息发送一致性的问题，如果能在本地事务执行和发消息这两个操作上加上事务，那岂不是完美。

基于这个思路， 在 MQ 中存储消息的状态才是真理，消息生产者先把消息发送给MQ，此时消息状态为“待确认”，接着生产者去执行本地事务，如果执行成功就给MQ发送消息让他更改消息状态为 “待发送”并发送消息，如果执行失败则删除消息。

这样就保证了本地事务和消息发送一致性问题。

![4](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3csvpllj31di0lcjvz.jpg)

1. 首先事务发起方先往 MQ 发送一条预读消息，这条消息与普通消息的区别在于他只对 MQ 可见不会向下传播。
2. MQ接受到消息后，先进行持久化，则存储中会新增一条状态为`待发送`的消息，接着给事务发起方返回处理完成的 ACK；事务发起方收到处理完成的 ACK 之后开始执行本地事务。
3. 发起方会根据本地事务的执行状态来决定这个预读消息是应该继续往前还是回滚。另外 MQ 也应该支持自己反查来解决异常情况，如果发起方本地事务已经执行完毕发送消息到MQ,但是消息因为网络原因丢失，那么怎么解决。所以这个反查机制很重要。
4. 本地事务执行成功以后，MQ 也接收到成功通知，接着将消息状态更新为可发送，然后将消息推送给下游的消费者，这个时候消费者就可以去处理自己的本地事务 。

**注意点**：由于MQ通常都会保证消息能够投递成功，因此，如果业务没有及时返回ACK结果，那么就有可能造成MQ的重复消息投递问题。**因此，对于消息最终一致性的方案，消息的消费者必须要对消息的消费支持幂等，不能造成同一条消息的重复消费的情况。**

#### SAGA 事务模型[#](https://www.cnblogs.com/rickiyang/p/13704868.html#4273837675)

Saga是什么？Saga的定义是 “长时间活动的事务 ”(Long Lived Transaction，后文简称为LLT)。他是普林斯顿大学 HECTOR GARCIA-MOLINA 教授在1987年的一篇关于分布式数据库的论文中提出来的概念。

Long Lived 从字面意义上不清晰，Long 到底意味着多长？事务持续时间是一个小时、一天甚至一周吗？其实都不是，时间跨度并不重要。重要的是什么？关键的是跨系统的多次“事务”，Saga 往往由多个外部子事务构成，需要通过多次外部系统的消息交互，才能将整体事务从开始迁移到结束状态，这和我们原来常见的在一个数据库的短事务不一样。比如一个旅行的订单，是由机票、旅馆、租车三个子订单构成，都需要外部的确认，缺任何一个步骤，不能成行，这就是一个典型的 LLT。

看起来 Sage 的定义与别的分布式事务没有什么不同。分布式事务不就是多个不同的子事务构成一个整体吗？再来看看 补偿机制：

每个本地事务有相应的执行模块和补偿模块，当 Sage 事务中的任意一个本地事务出错， 可以通过调用相关事务对应的补偿方法恢复，达到事务的最终一致性。

Saga 模型是把一个分布式事务拆分为多个本地事务，每个本地事务都有相应的执行模块和补偿模块（对应TCC 中的 Confirm 和 Cancel），当 Saga 事务中任意一个本地事务出错时，可以通过调用相关的补偿方法恢复之前的事务，达到事务最终一致性。

由于 Saga 模型中没有 Prepare 阶段，因此事务间不能保证隔离性，当多个 Saga 事务操作同一资源时，就会产生更新丢失、脏数据读取等问题，这时需要在业务层控制并发，例如：

- 在应用层面加锁；
- 应用层面预先冻结资源。

Saga 恢复方式

Saga 支持向前和向后恢复：

- 向后恢复：补偿所有已完成的事务，如果任一子事务失败；
- 向前恢复：重试失败的事务，假设每个子事务最终都会成功。

虽然 Saga 和 TCC 都是补偿事务，但是由于提交阶段不同，所以两者也是有不同的：

- Saga 没有 Try 行为直接 Commit，所以会留下原始事务操作的痕迹，Cancel 属于不完美补偿，需要考虑对业务上的影响。TCC Cancel 是完美补偿的 Rollback，补偿操作会彻底清理之前的原始事务操作，用户是感知不到事务取消之前的状态信息的。
- Saga 的补偿操作通常可以异步执行，TCC 的 Cancel 和 Confirm 可以跟进需要是否异步化。
- Saga 对业务侵入较小，只需要提供一个逆向操作的 Cancel 即可；而 TCC 需要对业务进行全局性的流程改造。
- TCC 最少通信次数为 2n，而 Saga 为 n（n=子事务的数量）。

因为也是采用补偿机制，那么必然要求服务保持幂等性，如果服务调用超时需要通过幂等性来避免多次请求带来的问题。

事务特性的满足：

原子性：Saga 协调器保证整体事务要么全部执行成功，要么全部回滚。

一致性：Sage 保证最终一致性。

持久性：Saga 将整体事务拆分成独立的本地事务，所以持久性在本地事务中很好实现。

但是隔离性 Saga 无法实现，因为大事务被拆分为多个小事务，每个事务提交的时机不同很难保证已提交的小事务不被别人可见。

目前业界提供两类 Saga 的实现方式：

- 一种是集中式协调的实现方式。

  集中式协调方式就是通过一个 Saga 对象来追踪所有的 Saga 子任务的调用，由它来管理，追踪整个事务是否应该提交或补偿。

  这种方式带来的缺点就是这种协调方式必然要与第一个Saga 事务耦合，即与业务耦合在一起。

- 一种是分布式实现方式。

  分布式协调方式肯定就能避免耦合的问题。分布式实现的方案也很多，比如通过事件机制来实现，一条 Saga 事务链上的所有事务都订阅同一个事件，如果失败则通过失败对应的事件消息来回滚即可。

  这种方式带来的好处肯定是显而易见的，但是也会有另一个问题，多个事件带来的肯定是高并发的处理，那么会不会因为多个事件处理相关的问题带来一些循环依赖的问题。

#### 开源分布式事务框架简介[#](https://www.cnblogs.com/rickiyang/p/13704868.html#3979394233)

##### Seata

Seata（Simple Extensible Autonomous Transaction Architecture，简单可扩展自治事务框架）是 2019 年 1 月份蚂蚁金服和阿里巴巴共同开源的分布式事务解决方案。

Seata 会有 4 种分布式事务解决方案，分别是 AT 模式、TCC 模式、Saga 模式和 XA 模式。

![6](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3ctu9a9j31a80e2dgy.jpg)

**XA 模式**

XA 模式是 Seata 将会开源的另一种无侵入的分布式事务解决方案，任何实现了 XA 协议的数据库都可以作为资源参与到分布式事务中，目前主流数据库，例如 MySql、Oracle、DB2、Oceanbase 等均支持 XA 协议。

XA 协议有一系列的指令，分别对应一阶段和二阶段操作。“xa start” 和 “xa end” 用于开启和结束XA 事务；“xa prepare” 用于预提交 XA 事务，对应一阶段准备；“xa commit”和“xa rollback”用于提交、回滚 XA 事务，对应二阶段提交和回滚。

在 XA 模式下，每一个 XA 事务都是一个事务参与者。分布式事务开启之后，首先在一阶段执行“xa start”、“业务 SQL”、“xa end”和 “xa prepare” 完成 XA 事务的执行和预提交；二阶段如果提交的话就执行 “xa commit”，如果是回滚则执行“xa rollback”。这样便能保证所有 XA 事务都提交或者都回滚。

![10](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cu5wksj314i0u042c.jpg)

XA 模式下，用户只需关注自己的“业务 SQL”，Seata 框架会自动生成一阶段、二阶段操作；XA 模式的实现如下：

![11](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cxfe7kj316c0u0774.jpg)

- 一阶段：

在 XA 模式的一阶段，Seata 会拦截“业务 SQL”，在“业务 SQL”之前开启 XA 事务（“xa start”），然后执行“业务 SQL”，结束 XA 事务“xa end”，最后预提交 XA 事务（“xa prepare”），这样便完成 “业务 SQL”的准备操作。

- 二阶段提交：

执行“xa commit”指令，提交 XA 事务，此时“业务 SQL”才算真正的提交至数据库。

- 二阶段回滚：

执行“xa rollback”指令，回滚 XA 事务，完成“业务 SQL”回滚，释放数据库锁资源。

XA 模式下，用户只需关注“业务 SQL”，Seata 会自动生成一阶段、二阶段提交和二阶段回滚操作。XA 模式和 AT 模式一样是一种对业务无侵入性的解决方案；但与 AT 模式不同的是，XA 模式将快照数据和行锁等通过 XA 指令委托给了数据库来完成，这样 XA 模式实现更加轻量化。

**AT 模式**

AT 模式是一种无侵入的分布式事务解决方案。在 AT 模式下，用户只需关注自己的“业务 SQL”，用户的 “业务 SQL” 作为一阶段，Seata 框架会自动生成事务的二阶段提交和回滚操作。

![7](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cvu2tgj31a60iyacc.jpg)

AT 模式的一阶段、二阶段提交和回滚

均由 Seata 框架自动生成，用户只需编写“业务 SQL”，便能轻松接入分布式事务，AT 模式是一种对业务无任何侵入的分布式事务解决方案。

**TCC 模式**

2019 年 3 月份，Seata 开源了 TCC 模式，该模式由蚂蚁金服贡献。TCC 模式需要用户根据自己的业务场景实现 Try、Confirm 和 Cancel 三个操作；事务发起方在一阶段 执行 Try 方式，在二阶段提交执行 Confirm 方法，二阶段回滚执行 Cancel 方法。

![8](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cupwnnj30z20u0gpo.jpg)

TCC 三个方法描述：

- Try：资源的检测和预留；
- Confirm：执行的业务操作提交；要求 Try 成功 Confirm 一定要能成功；
- Cancel：预留资源释放。

用户接入 TCC 模式，最重要的事情就是考虑如何将业务模型拆成 2 阶段，实现成 TCC 的 3 个方法，并且保证 Try 成功 Confirm 一定能成功。相对于 AT 模式，TCC 模式对业务代码有一定的侵入性，但是 TCC 模式无 AT 模式的全局行锁，TCC 性能会比 AT 模式高很多。

**Saga 模式**

Saga 模式是 Seata 即将开源的长事务解决方案，将由蚂蚁金服主要贡献。在 Saga 模式下，分布式事务内有多个参与者，每一个参与者都是一个冲正补偿服务，需要用户根据业务场景实现其正向操作和逆向回滚操作。

分布式事务执行过程中，依次执行各参与者的正向操作，如果所有正向操作均执行成功，那么分布式事务提交。如果任何一个正向操作执行失败，那么分布式事务会去退回去执行前面各参与者的逆向回滚操作，回滚已提交的参与者，使分布式事务回到初始状态。

![9](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cyh4mdj30pu0qmjtf.jpg)

Saga 模式下分布式事务通常是由事件驱动的，各个参与者之间是异步执行的，Saga 模式是一种长事务解决方案。

##### ServiceComb

ServiceComb 是华为开源的微服务框架，目前已升级为 Apache 顶级项目。 准确来说它并不是一个纯粹的分布式事务框架而是微服务框架，最开始的版本是 Go 语言,后面支持了 Java。

ServiceComb 由 3 个子项目组成：

- java-chassis：服务治理
- service-center：服务注册
- saga：分布式事务解决

从名字上看很显然是基于 Saga 模式开发的柔性事务方案。Saga系统分为两部分：Alpha 和 Omega。Alpha 是独立的服务，扮演事务协调器的作用。Omega 作为开发组件，和业务进程运行在一起。

![12](https://tva1.sinaimg.cn/large/007S8ZIlgy1giy3cv5finj30n00c4q3g.jpg)

Omega 会以切面编程的方式向应用程序注入相关的处理模块。这里有拦截请求的模块， 用来帮助我们构建分布式事务调用的上下文。 同时在事务处理初始阶段处理事务的相关准备的操作，例如创建 Saga 起始事件，以及相关的子起始事件， 根据事务的执行的成功或者失败生产相关的事务终止或者失败事件。

Omega 会与 Alpha 进行链接会把这些事件通知给 Alpha。 Alpha 可以在后台进行分析，根据 Saga 事务执行的情况给 Omega 下达相关的指令进行相关的回滚恢复。

这样设计的好处是 Saga 实现代码与用户的代码分离， 用户只需要添加几个 annotation，Saga 实现就能 Saga 事件的执行情况并进行相关的处理。

但是不能忘记的是，软件开发者做的确实有些不地道，”借鉴“了国外友人的源码，却没有添加许可声明。

https://github.com/go-chassis/go-chassis/issues/28

经过网友的笔诛讨伐，好在及时浪子回头，从此在互联网的书页上留下了“淡淡的忧伤”。

# [带你了解 MySQL Binlog 不为人知的秘密](https://www.cnblogs.com/rickiyang/p/13841811.html)

MySQL 的 Binlog 日志是一种二进制格式的日志，Binlog 记录所有的 DDL 和 DML 语句(除了数据查询语句SELECT、SHOW等)，以 Event 的形式记录，同时记录语句执行时间。

Binlog 的主要作用有两个：

1. 数据恢复

   因为 Binlog 详细记录了所有修改数据的 SQL，当某一时刻的数据误操作而导致出问题，或者数据库宕机数据丢失，那么可以根据 Binlog 来回放历史数据。

2. 主从复制

   想要做多机备份的业务，可以去监听当前写库的 Binlog 日志，同步写库的所有更改。

Binlog 包括两类文件：

- 二进制日志索引文件(.index)：记录所有的二进制文件。
- 二进制日志文件(.00000*)：记录所有 DDL 和 DML 语句事件。

Binlog 日志功能默认是开启的，线上情况下 Binlog 日志的增长速度是很快的，在 MySQL 的配置文件 `my.cnf` 中提供一些参数来对 Binlog 进行设置。

```sql
Copy设置此参数表示启用binlog功能，并制定二进制日志的存储目录
log-bin=/home/mysql/binlog/

#mysql-bin.*日志文件最大字节（单位：字节）
#设置最大100MB
max_binlog_size=104857600

#设置了只保留7天BINLOG（单位：天）
expire_logs_days = 7

#binlog日志只记录指定库的更新
#binlog-do-db=db_name

#binlog日志不记录指定库的更新
#binlog-ignore-db=db_name

#写缓冲多少次，刷一次磁盘，默认0
sync_binlog=0
```

需要注意的是：

**max_binlog_size** ：Binlog 最大和默认值是 1G，该设置并不能严格控制 Binlog 的大小，尤其是 Binlog 比较靠近最大值而又遇到一个比较大事务时，为了保证事务的完整性不可能做切换日志的动作，只能将该事务的所有 SQL 都记录进当前日志直到事务结束。所以真实文件有时候会大于 max_binlog_size 设定值。
**expire_logs_days** ：Binlog 过期删除不是服务定时执行，是需要借助事件触发才执行，事件包括：

- 服务器重启
- 服务器被更新
- 日志达到了最大日志长度 `max_binlog_size`
- 日志被刷新

二进制日志由配置文件的 `log-bin` 选项负责启用，MySQL 服务器将在数据根目录创建两个新文件`mysql-bin.000001` 和 `mysql-bin.index`，若配置选项没有给出文件名，MySQL 将使用主机名称命名这两个文件，其中 `.index` 文件包含一份全体日志文件的清单。

**sync_binlog**：这个参数决定了 Binlog 日志的更新频率。默认 0 ，表示该操作由操作系统根据自身负载自行决定多久写一次磁盘。

sync_binlog = 1 表示每一条事务提交都会立刻写盘。sync_binlog=n 表示 n 个事务提交才会写盘。

根据 MySQL 文档，写 Binlog 的时机是：SQL transaction 执行完，但任何相关的 Locks 还未释放或事务还未最终 commit 前。这样保证了 Binlog 记录的操作时序与数据库实际的数据变更顺序一致。

检查 Binlog 文件是否已开启：

```sql
Copymysql> show variables like '%log_bin%';
+---------------------------------+------------------------------------+
| Variable_name                   | Value                              |
+---------------------------------+------------------------------------+
| log_bin                         | ON                                 |
| log_bin_basename                | /usr/local/mysql/data/binlog       |
| log_bin_index                   | /usr/local/mysql/data/binlog.index |
| log_bin_trust_function_creators | OFF                                |
| log_bin_use_v1_row_events       | OFF                                |
| sql_log_bin                     | ON                                 |
+---------------------------------+------------------------------------+
6 rows in set (0.00 sec)
```

MySQL 会把用户对所有数据库的内容和结构的修改情况记入 `mysql-bin.n` 文件，而不会记录 SELECT 和没有实际更新的 UPDATE 语句。

如果你不知道现在有哪些 Binlog 文件，可以使用如下命令：

```sql
Copyshow binary logs; #查看binlog列表
show master status; #查看最新的binlog

mysql> show binary logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       179 | No        |
| mysql-bin.000002 |       156 | No        |
+------------------+-----------+-----------+
2 rows in set (0.00 sec)
```

Binlog 文件是二进制文件，强行打开看到的必然是乱码，MySQL 提供了命令行的方式来展示 Binlog 日志：

```shell
Copymysqlbinlog mysql-bin.000002 | more
```

`mysqlbinlog` 命令即可查看。

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjusawxe99j31ne0u07kb.jpg)

看起来凌乱其实也有迹可循。Binlog 通过事件的方式来管理日志信息，可以通过 `show binlog events in` 的语法来查看当前 Binlog 文件对应的详细事件信息。

```sql
Copymysql> show binlog events in 'mysql-bin.000001';
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                              |
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
| mysql-bin.000001 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.21, Binlog ver: 4 |
| mysql-bin.000001 | 125 | Previous_gtids |         1 |         156 |                                   |
| mysql-bin.000001 | 156 | Stop           |         1 |         179 |                                   |
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
3 rows in set (0.01 sec)
```

这是一份没有任何写入数据的 Binlog 日志文件。

Binlog 的版本是V4，可以看到日志的结束时间为 Stop。出现 Stop event 有两种情况：

1. 是 master shut down 的时候会在 Binlog 文件结尾出现
2. 是备机在关闭的时候会写入 relay log 结尾，或者执行 RESET SLAVE 命令执行

本文出现的原因是我有手动停止过 MySQL 服务。

一般来说一份正常的 Binlog 日志文件会以 **Rotate event** 结束。当 Binlog 文件超过指定大小，Rotate event 会写在文件最后，指向下一个 Binlog 文件。

我们来看看有过数据操作的 Binlog 日志文件是什么样子的。

```sql
Copymysql> show binlog events in 'mysql-bin.000002';
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                              |
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
| mysql-bin.000002 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.21, Binlog ver: 4 |
| mysql-bin.000002 | 125 | Previous_gtids |         1 |         156 |                                   |
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
2 rows in set (0.00 sec)
```

上面是没有任何数据操作且没有被截断的 Binlog。接下来我们插入一条数据，再看看 Binlog 事件。

```sql
Copymysql> show binlog events in 'mysql-bin.000002';
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                                                    |
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
| mysql-bin.000002 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.21, Binlog ver: 4                                       |
| mysql-bin.000002 | 125 | Previous_gtids |         1 |         156 |                                                                         |
| mysql-bin.000002 | 156 | Anonymous_Gtid |         1 |         235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                    |
| mysql-bin.000002 | 235 | Query          |         1 |         323 | BEGIN                                                                   |
| mysql-bin.000002 | 323 | Intvar         |         1 |         355 | INSERT_ID=13                                                            |
| mysql-bin.000002 | 355 | Query          |         1 |         494 | use `test_db`; INSERT INTO `test_db`.`test_db`(`name`) VALUES ('xdfdf') |
| mysql-bin.000002 | 494 | Xid            |         1 |         525 | COMMIT /* xid=192 */                                                    |
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
7 rows in set (0.00 sec)
```

这是加入一条数据之后的 Binlog 事件。

我们对 event 查询的数据行关键字段来解释一下：

- Pos：当前事件的开始位置，每个事件都占用固定的字节大小，结束位置(End_log_position)减去Pos，就是这个事件占用的字节数。

  上面的日志中我们能看到，第一个事件位置并不是从 0 开始，而是从 4。MySQL 通过文件中的前 4 个字节，来判断这是不是一个 Binlog 文件。这种方式很常见，很多格式的文件，如 pdf、doc、jpg等，都会通常前几个特定字符判断是否是合法文件。

- Event_type：表示事件的类型

- Server_id：表示产生这个事件的 MySQL server_id，通过设置 my.cnf 中的 **server-id** 选项进行配置

- End_log_position：下一个事件的开始位置

- Info：包含事件的具体信息

#### Binlog 日志格式[#](https://www.cnblogs.com/rickiyang/p/13841811.html#2992485385)

针对不同的使用场景，Binlog 也提供了可定制化的服务，提供了三种模式来提供不同详细程度的日志内容。

- Statement 模式：基于 SQL 语句的复制(statement-based replication-SBR)
- Row 模式：基于行的复制(row-based replication-RBR)
- Mixed 模式：混合模式复制(mixed-based replication-MBR)

###### Statement 模式

保存每一条修改数据的SQL。

该模式只保存一条普通的SQL语句，不涉及到执行的上下文信息。

因为每台 MySQL 数据库的本地环境可能不一样，那么对于依赖到本地环境的函数或者上下文处理的逻辑 SQL 去处理的时候可能同样的语句在不同的机器上执行出来的效果不一致。

比如像 `sleep()`函数，`last_insert_id()`函数，等等，这些都跟特定时间的本地环境有关。

##### Row 模式

MySQL V5.1.5 版本开始支持Row模式的 Binlog，它与 Statement 模式的区别在于它不保存具体的 SQL 语句，而是记录具体被修改的信息。

比如一条 update 语句更新10条数据，如果是 Statement 模式那就保存一条 SQL 就够，但是 Row 模式会保存每一行分别更新了什么，有10条数据。

Row 模式的优缺点就很明显了。保存每一个更改的详细信息必然会带来存储空间的快速膨胀，换来的是事件操作的详细记录。所以要求越高代价越高。

##### Mixed 模式

Mixed 模式即以上两种模式的综合体。既然上面两种模式分别走了极简和一丝不苟的极端，那是否可以区分使用场景的情况下将这两种模式综合起来呢？

在 Mixed 模式中，一般的更新语句使用 Statement 模式来保存 Binlog，但是遇到一些函数操作，可能会影响数据准确性的操作则使用 Row 模式来保存。这种方式需要根据每一条具体的 SQL 语句来区分选择哪种模式。

MySQL 从 V5.1.8 开始提供 Mixed 模式，V5.7.7 之前的版本默认是Statement 模式，之后默认使用Row模式， 但是在 8.0 以上版本已经默认使用 Mixed 模式了。

查询当前 Binlog 日志使用格式：

```sql
Copymysql> show global variables like '%binlog_format%';
+---------------------------------+---------+
| Variable_name                   | Value   |
+---------------------------------+---------+
| binlog_format                   | MIXED   |
| default_week_format             | 0       |
| information_schema_stats_expiry | 86400   |
| innodb_default_row_format       | dynamic |
| require_row_format              | OFF     |
+---------------------------------+---------+
5 rows in set (0.01 sec)
```

#### 如何通过 `mysqlbinlog` 命令手动恢复数据[#](https://www.cnblogs.com/rickiyang/p/13841811.html#839816596)

上面说过每一条 event 都有位点信息，如果我们当前的 MySQL 库被无操作或者误删除了，那么该如何通过 Binlog 来恢复到删除之前的数据状态呢？

首先发现误操作之后，先停止 MySQL 服务，防止继续更新。

接着通过 `mysqlbinlog`命令对二进制文件进行分析，查看误操作之前的位点信息在哪里。

接下来肯定就是恢复数据，当前数据库的数据已经是错的，那么就从开始位置到误操作之前位点的数据肯定的都是正确的；如果误操作之后也有正常的数据进来，这一段时间的位点数据也要备份。

比如说：

误操作的位点开始值为 501，误操作结束的位置为705，之后到800的位点都是正确数据。

那么从 0 - 500 ，706 - 800 都是有效数据，接着我们就可以进行数据恢复了。

先将数据库备份并清空。

接着使用 `mysqlbinlog` 来恢复数据：

0 - 500 的数据：

```sql
Copymysqlbinlog --start-position=0  --stop-position=500  bin-log.000003 > /root/back.sql;
```

上面命令的作用就是将 0 -500 位点的数据恢复到自定义的 SQL 文件中。同理 706 - 800 的数据也是一样操作。之后我们执行这两个 SQL 文件就行了。

#### Binlog 事件类型[#](https://www.cnblogs.com/rickiyang/p/13841811.html#1756571331)

上面我们说到了 Binlog 日志中的事件，不同的操作会对应着不同的事件类型，且不同的 Binlog 日志模式同一个操作的事件类型也不同，下面我们一起看看常见的事件类型。

首先我们看看源码中的事件类型定义：

源码位置：/libbinlogevents/include/binlog_event.h

```c++
Copyenum Log_event_type
{
  /**
    Every time you update this enum (when you add a type), you have to
    fix Format_description_event::Format_description_event().
  */
  UNKNOWN_EVENT= 0,
  START_EVENT_V3= 1,
  QUERY_EVENT= 2,
  STOP_EVENT= 3,
  ROTATE_EVENT= 4,
  INTVAR_EVENT= 5,
  LOAD_EVENT= 6,
  SLAVE_EVENT= 7,
  CREATE_FILE_EVENT= 8,
  APPEND_BLOCK_EVENT= 9,
  EXEC_LOAD_EVENT= 10,
  DELETE_FILE_EVENT= 11,
  /**
    NEW_LOAD_EVENT is like LOAD_EVENT except that it has a longer
    sql_ex, allowing multibyte TERMINATED BY etc; both types share the
    same class (Load_event)
  */
  NEW_LOAD_EVENT= 12,
  RAND_EVENT= 13,
  USER_VAR_EVENT= 14,
  FORMAT_DESCRIPTION_EVENT= 15,
  XID_EVENT= 16,
  BEGIN_LOAD_QUERY_EVENT= 17,
  EXECUTE_LOAD_QUERY_EVENT= 18,

  TABLE_MAP_EVENT = 19,

  /**
    The PRE_GA event numbers were used for 5.1.0 to 5.1.15 and are
    therefore obsolete.
   */
  PRE_GA_WRITE_ROWS_EVENT = 20,
  PRE_GA_UPDATE_ROWS_EVENT = 21,
  PRE_GA_DELETE_ROWS_EVENT = 22,

  /**
    The V1 event numbers are used from 5.1.16 until mysql-trunk-xx
  */
  WRITE_ROWS_EVENT_V1 = 23,
  UPDATE_ROWS_EVENT_V1 = 24,
  DELETE_ROWS_EVENT_V1 = 25,

  /**
    Something out of the ordinary happened on the master
   */
  INCIDENT_EVENT= 26,

  /**
    Heartbeat event to be send by master at its idle time
    to ensure master's online status to slave
  */
  HEARTBEAT_LOG_EVENT= 27,

  /**
    In some situations, it is necessary to send over ignorable
    data to the slave: data that a slave can handle in case there
    is code for handling it, but which can be ignored if it is not
    recognized.
  */
  IGNORABLE_LOG_EVENT= 28,
  ROWS_QUERY_LOG_EVENT= 29,

  /** Version 2 of the Row events */
  WRITE_ROWS_EVENT = 30,
  UPDATE_ROWS_EVENT = 31,
  DELETE_ROWS_EVENT = 32,

  GTID_LOG_EVENT= 33,
  ANONYMOUS_GTID_LOG_EVENT= 34,

  PREVIOUS_GTIDS_LOG_EVENT= 35,

  TRANSACTION_CONTEXT_EVENT= 36,

  VIEW_CHANGE_EVENT= 37,

  /* Prepared XA transaction terminal event similar to Xid */
  XA_PREPARE_LOG_EVENT= 38,
  /**
    Add new events here - right above this comment!
    Existing events (except ENUM_END_EVENT) should never change their numbers
  */
  ENUM_END_EVENT /* end marker */
};
```

这么多的事件类型我们就不一一介绍，挑出来一些常用的来看看。

**FORMAT_DESCRIPTION_EVENT**

FORMAT_DESCRIPTION_EVENT 是 Binlog V4 中为了取代之前版本中的 START_EVENT_V3 事件而引入的。它是 Binlog 文件中的第一个事件，而且，该事件只会在 Binlog 中出现一次。MySQL 根据 FORMAT_DESCRIPTION_EVENT 的定义来解析其它事件。

它通常指定了 MySQL 的版本，Binlog 的版本，该 Binlog 文件的创建时间。

**QUERY_EVENT**

QUERY_EVENT 类型的事件通常在以下几种情况下使用：

- 事务开始时，执行的 BEGIN 操作
- STATEMENT 格式中的 DML 操作
- ROW 格式中的 DDL 操作

比如上文我们插入一条数据之后的 Binlog 日志：

```sql
Copymysql> show binlog events in 'mysql-bin.000002';
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                                                    |
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
| mysql-bin.000002 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.21, Binlog ver: 4                                       |
| mysql-bin.000002 | 125 | Previous_gtids |         1 |         156 |                                                                         |
| mysql-bin.000002 | 156 | Anonymous_Gtid |         1 |         235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                    |
| mysql-bin.000002 | 235 | Query          |         1 |         323 | BEGIN                                                                   |
| mysql-bin.000002 | 323 | Intvar         |         1 |         355 | INSERT_ID=13                                                            |
| mysql-bin.000002 | 355 | Query          |         1 |         494 | use `test_db`; INSERT INTO `test_db`.`test_db`(`name`) VALUES ('xdfdf') |
| mysql-bin.000002 | 494 | Xid            |         1 |         525 | COMMIT /* xid=192 */                                                    |
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
7 rows in set (0.00 sec)
```

**XID_EVENT**

在事务提交时，不管是 STATEMENT 还 是ROW 格式的 Binlog，都会在末尾添加一个 XID_EVENT 事件代表事务的结束。该事件记录了该事务的 ID，在 MySQL 进行崩溃恢复时，根据事务在 Binlog 中的提交情况来决定是否提交存储引擎中状态为 prepared 的事务。

**ROWS_EVENT**

对于 ROW 格式的 Binlog，所有的 DML 语句都是记录在 ROWS_EVENT 中。

ROWS_EVENT分为三种：

- WRITE_ROWS_EVENT
- UPDATE_ROWS_EVENT
- DELETE_ROWS_EVENT

分别对应 insert，update 和 delete 操作。

对于 insert 操作，WRITE_ROWS_EVENT 包含了要插入的数据。

对于 update 操作，UPDATE_ROWS_EVENT 不仅包含了修改后的数据，还包含了修改前的值。

对于 delete 操作，仅仅需要指定删除的主键（在没有主键的情况下，会给定所有列）。

**对比 QUERY_EVENT 事件，是以文本形式记录 DML 操作的。而对于 ROWS_EVENT 事件，并不是文本形式，所以在通过 mysqlbinlog 查看基于 ROW 格式的 Binlog 时，需要指定 `-vv --base64-output=decode-rows`。**

我们来测试一下，首先将日志格式改为 Rows：

```sql
Copymysql> set binlog_format=row;
Query OK, 0 rows affected (0.00 sec)

mysql>
mysql> flush logs;
Query OK, 0 rows affected (0.01 sec)
```

然后刷新一下日志文件，重新开始一个 Binlog 日志。我们插入一条数据之后看一下日志：

```sql
Copymysql> show binlog events in 'binlog.000008';
+---------------+-----+----------------+-----------+-------------+--------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                 |
+---------------+-----+----------------+-----------+-------------+--------------------------------------+
| binlog.000008 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.21, Binlog ver: 4    |
| binlog.000008 | 125 | Previous_gtids |         1 |         156 |                                      |
| binlog.000008 | 156 | Anonymous_Gtid |         1 |         235 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS' |
| binlog.000008 | 235 | Query          |         1 |         313 | BEGIN                                |
| binlog.000008 | 313 | Table_map      |         1 |         377 | table_id: 85 (test_db.test_db)       |
| binlog.000008 | 377 | Write_rows     |         1 |         423 | table_id: 85 flags: STMT_END_F       |
| binlog.000008 | 423 | Xid            |         1 |         454 | COMMIT /* xid=44 */                  |
+---------------+-----+----------------+-----------+-------------+--------------------------------------+
7 rows in set (0.01 sec)
```

#### 总结[#](https://www.cnblogs.com/rickiyang/p/13841811.html#2663513052)

这一篇我们详解了解 Binlog 日志是什么，里面都有什么内容，Binlog 事件，如何通过 Binlog 来恢复数据。Binlog 目前最重要的应用就是用于主从同步，那么下一篇我们讲来讲讲如何通过 Binlog 实现主从同步。

# [MySQL 主从复制原理不再难](https://www.cnblogs.com/rickiyang/p/13856388.html)

上篇我们分析过 Binlog 日志的作用以及存储原理，感兴趣的可以翻阅：

[一文带你了解 Binlog 日志](https://www.cnblogs.com/rickiyang/p/13841811.html)

Binlog 日志主要作用是数据恢复和主从复制。本身就是二进制格式的日志文件，网络传输无需进行协议转换。MySQL 集群的高可用，负载均衡，读写分离等功能都是基于Binlog 来实现的。

#### MySQL 主从复制主流架构模型[#](https://www.cnblogs.com/rickiyang/p/13856388.html#3507919906)

我们基于 Binlog 可以复制出一台 MySQL 服务器，也可以复制出多台，取决于我们想实现什么功能。主流的系统架构有如下几种方式：

##### 1. 一主一从 / 一主多从

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1mx9n5wj30lu0mwgne.jpg)

一主一从和一主多从是最常见的主从架构方式，一般实现主从配置或者读写分离都可以采用这种架构。

如果是一主多从的模式，当 Slave 增加到一定数量时，Slave 对 Master 的负载以及网络带宽都会成为一个严重的问题。

##### 2. 多主一从

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1mt1km7j30m80l4407.jpg)

MySQL 5.7 开始支持多主一从的模式，将多个库的数据备份到一个库中存储。

##### 3. 双主复制

理论上跟主从一样，但是两个MySQL服务器互做对方的从，任何一方有变更，都会复制对方的数据到自己的数据库。双主适用于写压力比较大的业务场景，或者 DBA 做维护需要主从切换的场景，通过双主架构避免了重复搭建从库的麻烦。（主从相互授权连接，读取对方binlog日志并更新到本地数据库的过程；只要对方数据改变，自己就跟着改变）

##### 4. 级联复制

![1](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1mr6mxzj30z60hg40u.jpg)

级联模式下因为涉及到的 slave 节点很多，所以如果都连在 master 上对主服务器的压力肯定是不小的。所以部分 slave 节点连接到它上一级的从节点上。这样就缓解了主服务器的压力。

级联复制解决了一主多从场景下多个从库复制对主库的压力，带来的弊端就是数据同步延迟比较大。

#### MySQL 主从复制原理[#](https://www.cnblogs.com/rickiyang/p/13856388.html#1845631569)

MySQL 主从复制涉及到三个线程：

一个在主节点的线程：`log dump thread`

从库会生成两个线程：一个 I/O 线程，一个 SQL 线程

如下图所示:

![4](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1ms5ezhj312s0imtb5.jpg)

主库会生成一个 log dump 线程,用来给从库 I/O 线程传 Binlog 数据。

从库的 I/O 线程会去请求主库的 Binlog，并将得到的 Binlog 写到本地的 relay log (中继日志)文件中。

SQL 线程,会读取 relay log 文件中的日志，并解析成 SQL 语句逐一执行。

##### 主节点 log dump 线程

当从节点连接主节点时，主节点会为其创建一个 log dump 线程，用于发送和读取 Binlog 的内容。在读取 Binlog 中的操作时，log dump 线程会对主节点上的 Binlog 加锁；当读取完成发送给从节点之前，锁会被释放。**主节点会为自己的每一个从节点创建一个 log dump 线程**。

##### 从节点I/O线程

当从节点上执行`start slave`命令之后，从节点会创建一个 I/O 线程用来连接主节点，请求主库中更新的Binlog。I/O 线程接收到主节点的 log dump 进程发来的更新之后，保存在本地 relay-log（中继日志）中。

##### relay log

这里又引申出一个新的日志概念。MySQL 进行主主复制或主从复制的时候会在要复制的服务器下面产生相应的 relay log。

relay log 是怎么产生的呢？

从服务器 I/O 线程将主服务器的 Binlog 日志读取过来，解析到各类 Events 之后记录到从服务器本地文件，这个文件就被称为 relay log。然后 SQL 线程会读取 relay log 日志的内容并应用到从服务器，从而使从服务器和主服务器的数据保持一致。中继日志充当缓冲区，这样 master 就不必等待 slave 执行完成才发送下一个事件。

relay log 相关参数查询：

```sql
Copymysql>  show variables like '%relay%';
+---------------------------+------------------------------------------------------------+
| Variable_name             | Value                                                      |
+---------------------------+------------------------------------------------------------+
| max_relay_log_size        | 0                                                          |
| relay_log                 | yangyuedeMacBook-Pro-relay-bin                             |
| relay_log_basename        | /usr/local/mysql/data/yangyuedeMacBook-Pro-relay-bin       |
| relay_log_index           | /usr/local/mysql/data/yangyuedeMacBook-Pro-relay-bin.index |
| relay_log_info_file       | relay-log.info                                             |
| relay_log_info_repository | TABLE                                                      |
| relay_log_purge           | ON                                                         |
| relay_log_recovery        | OFF                                                        |
| relay_log_space_limit     | 0                                                          |
| sync_relay_log            | 10000                                                      |
| sync_relay_log_info       | 10000                                                      |
+---------------------------+------------------------------------------------------------+
11 rows in set (0.03 sec)
```

**max_relay_log_size**

标记 relay log 允许的最大值，如果该值为 0，则默认值为 max_binlog_size(1G)；如果不为 0，则max_relay_log_size 则为最大的 relay_log 文件大小。

**relay_log_purge**

是否自动清空不再需要中继日志时。默认值为1(启用)。

**relay_log_recovery**

当 slave 从库宕机后，假如 relay log 损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的 relay log，并且重新从 master 上获取日志，这样就保证了 relay log 的完整性。默认情况下该功能是关闭的，将 relay_log_recovery 的值设置为 1 时，可在 slave 从库上开启该功能，建议开启。

**relay_log_space_limit**

防止中继日志写满磁盘，这里设置中继日志最大限额。但此设置存在主库崩溃，从库中继日志不全的情况，不到万不得已，**不推荐使用。**

**sync_relay_log**

这个参数和 Binlog 中的 `sync_binlog`作用相同。当设置为 1 时，slave 的 I/O 线程每次接收到 master 发送过来的 Binlog 日志都要写入系统缓冲区，然后刷入 relay log 中继日志里，这样是最安全的，因为在崩溃的时候，你最多会丢失一个事务，但会造成磁盘的大量 I/O。

当设置为 0 时，并不是马上就刷入中继日志里，而是由操作系统决定何时来写入，虽然安全性降低了，但减少了大量的磁盘 I/O 操作。这个值默认是 0，可动态修改，建议采用默认值。

**sync_relay_log_info**

当设置为 1 时，slave 的 I/O 线程每次接收到 master 发送过来的 Binlog 日志都要写入系统缓冲区，然后刷入 relay-log.info 里，这样是最安全的，因为在崩溃的时候，你最多会丢失一个事务，但会造成磁盘的大量 I/O。当设置为 0 时，并不是马上就刷入 relay-log.info 里，而是由操作系统决定何时来写入，虽然安全性降低了，但减少了大量的磁盘 I/O 操作。这个值默认是0，可动态修改，建议采用默认值。

##### 从节点 SQL 线程

SQL 线程负责读取 relay log 中的内容，解析成具体的操作并执行，最终保证主从数据的一致性。

对于每一个主从连接，都需要这三个进程来完成。当主节点有多个从节点时，主节点会为每一个当前连接的从节点建一个 log dump 进程，而每个从节点都有自己的 I/O 进程，SQL 进程。

从节点用两个线程将从主库拉取更新和执行分成独立的任务，这样在执行同步数据任务的时候，不会降低读操作的性能。比如，如果从节点没有运行，此时 I/O 进程可以很快从主节点获取更新，尽管 SQL 进程还没有执行。如果在 SQL 进程执行之前从节点服务停止，至少 I/O 进程已经从主节点拉取到了最新的变更并且保存在本地 relay log 中，当服务再次起来之后就可以完成数据的同步。

要实施复制，首先必须打开 Master 端的 Binlog 功能，否则无法实现。

因为整个复制过程实际上就是 Slave 从 Master 端获取该日志然后再在自己身上完全顺序的执行日志中所记录的各种操作。如下图所示：

![5](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1mvsnp5j31040iyad2.jpg)

##### 复制的基本过程

1. 在从节点上执行 `sart slave` 命令开启主从复制开关，开始进行主从复制。从节点上的 I/O 进程连接主节点，并请求从指定日志文件的指定位置（或者从最开始的日志）之后的日志内容。
2. 主节点接收到来自从节点的 I/O 请求后，通过负责复制的 I/O 进程（log Dump Thread）根据请求信息读取指定日志指定位置之后的日志信息，返回给从节点。返回信息中除了日志所包含的信息之外，还包括本次返回的信息的 Binlog file 以及 Binlog position（Binlog 下一个数据读取位置）。
3. 从节点的 I/O 进程接收到主节点发送过来的日志内容、日志文件及位置点后，将接收到的日志内容更新到本机的 relay log 文件（Mysql-relay-bin.xxx）的最末端，并将读取到的 Binlog文件名和位置保存到`master-info` 文件中，以便在下一次读取的时候能够清楚的告诉 Master ：“ 我需要从哪个 Binlog 的哪个位置开始往后的日志内容，请发给我”。
4. Slave 的 SQL 线程检测到relay log 中新增加了内容后，会将 relay log 的内容解析成在能够执行 SQL 语句，然后在本数据库中按照解析出来的顺序执行，并在 `relay log.info` 中记录当前应用中继日志的文件名和位置点。

#### MySQL 基于 Binlog 主从复制的模式介绍[#](https://www.cnblogs.com/rickiyang/p/13856388.html#552353404)

MySQL 主从复制默认是 **异步的模式**。MySQL增删改操作会全部记录在 Binlog 中，当 slave 节点连接 master 时，会主动从 master 处获取最新的 Binlog 文件。并把 Binlog 存储到本地的 relay log 中，然后去执行 relay log 的更新内容。

##### 异步模式 (async-mode)

异步模式如下图所示：

![6](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1mu02wbj31160kk0vj.jpg)

这种模式下，主节点不会主动推送数据到从节点，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主节点如果崩溃掉了，此时主节点上已经提交的事务可能并没有传到从节点上，如果此时，强行将从提升为主，可能导致新主节点上的数据不完整。

##### 半同步模式(semi-sync)

介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到 relay log 中才返回成功信息给客户端（只能保证主库的 Binlog 至少传输到了一个从节点上），否则需要等待直到超时时间然后切换成异步模式再提交。

![7](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjx1muypwdj31420jg41q.jpg)

相对于异步复制，半同步复制提高了数据的安全性，一定程度的保证了数据能成功备份到从库，同时它也造成了一定程度的延迟，但是比全同步模式延迟要低，这个延迟最少是一个 TCP/IP 往返的时间。所以，半同步复制最好在低延时的网络中使用。

半同步模式不是 MySQL 内置的，从 MySQL 5.5 开始集成，需要 master 和 slave 安装插件开启半同步模式。

##### 全同步模式

指当主库执行完一个事务，然后所有的从库都复制了该事务并成功执行完才返回成功信息给客户端。因为需要等待所有从库执行完该事务才能返回成功信息，所以全同步复制的性能必然会收到严重的影响。

#### Binlog 复制实战[#](https://www.cnblogs.com/rickiyang/p/13856388.html#822311245)

配置 my.cnf

```sql
Copy[mysqld]
log-bin
server-id
gtid_mode=off #禁掉 gtid
```

添加主从复制用户：

```sql
Copygrant replication slave on *.* to 'repl'@'%' identified by 'gtidUser';
flush privileges;
```

然后我们新增一个从库。

接着我们用命令行的方式来加载主库的 Binlog 到从库，这里可以设置指定的 binlog 文件和位移值。在从库执行以下命令：

```sql
Copymysql>change master to
master_host='192.168.199.117',
master_user='slave',
master_port=7000,
master_password='slavepass',
master_log_file='mysql-bin.000008',
master_log_pos=0;

mysql>start slave;
mysql>show slave status\G;
```

复制过程中如果出现代码性错误，个人根据错误日志来决定是否要跳过错误继续执行：

```sql
Copymysql>stop slave;
mysql>set global sql_slave_skip_counter=1;
```

#### 主从复制可能会出现的问题[#](https://www.cnblogs.com/rickiyang/p/13856388.html#3441388758)

**Slave 同步延迟**

因为 Slave 端是通过 I/O thread 单线程来实现数据解析入库；而 Master 端写 Binlog 由于是顺序写效率很高，当主库的 TPS 很高的时候，必然 Master 端的写效率要高过 Slave 端的读效率，这时候就有同步延迟的问题。

I/O Thread 的同步是基于库的，即同步几个库就会开启几个 I/O Thread。

可以通过 `show slave status` 命令查看 `Seconds_Behind_Master` 的值来看是否出现同步延迟，这个值代表主从同步延迟的时间，值越大说明延迟越严重。值为 0 为正常情况，正值表示已经出现延迟，数字越大从库落后主库越多。

基于 Binlog 的复制方式肯定有这种问题，MySQL 官方也意识到，单线程不如多线程强，所以在 MySQL 5.7 版本引入了基于组提交的并行复制（官方称为Enhanced Multi-threaded Slaves，即MTS），设置参数：

`slave_parallel_workers>0` 即可，并且 `global.slave_parallel_type＝‘LOGICAL_CLOCK’`，

即可支持一个 schema(库) 下，`slave_parallel_workers`个 worker 线程并发执行 relay log 中主库提交的事务。

**其核心思想：**

一个组提交的事务都是可以并行回放（配合binary log group commit）；

slave 机器的 relay log 中 last_committed 相同的事务（sequence_num不同）可以并发执行。其中，变量 `slave-parallel-type` 可以有两个值：

1. DATABASE 默认值，基于库的并行复制方式
2. LOGICAL_CLOCK，基于组提交的并行复制方式

MySQL 5.7 开启 MTS 很简单，只需要在 Slave 从数据库的 my.cnf 文件中如下配置即可:

```sql
Copy# slave
 slave-parallel-type=LOGICAL_CLOCK
 slave-parallel-workers=8        #一般建议设置4-8，太多的线程会增加线程之间的同步开销
 master_info_repository=TABLE
 relay_log_info_repository=TABLE
 relay_log_recovery=ON
```

当然多线程带来的并行复制方案也有很多实现难点，比如事务都是有序执行的，如果并行回放会不会存在执行数据错乱的问题。这些问题就不在本节解释，大家有兴趣可以继续深究。

#### 新一代主从复制模式 - GTID 复制模式[#](https://www.cnblogs.com/rickiyang/p/13856388.html#929452931)

在传统的复制里面，当发生故障，需要**主从切换**，需要找到 Binlog 和 位点信息，恢复完成数据之后将主节点指向新的主节点。在 MySQL 5.6 里面，提供了新的数据恢复思路，只需要知道主节点的 IP、端口以及账号密码就行，因为复制是自动的，MySQL 会通过内部机制 **GTID** 自动找点同步。

基于 GTID 的复制是 MySQL 5.6.5 后新增的复制方式。

**GTID (global transaction identifier)** 即全局事务 ID，一个事务对应一个 GTID，保证了在每个在主库上提交的事务在集群中有一个唯一的 ID。

##### GTID复制原理

在原来基于日志的复制中，从库需要告知主库要从哪个偏移量进行增量同步， 如果指定错误会造成数据的遗漏，从而造成数据的不一致。

而基于 GTID 的复制中，从库会告知主库已经执行的事务的 GTID 的值，然后主库会将所有未执行的事务的 GTID 的列表返回给从库，并且可以保证同一个事务只在指定的从库执行一次，**通过全局的事务 ID 确定从库要执行的事务的方式代替了以前需要用 Binlog 和 位点确定从库要执行的事务的方式**。

基于 GTID 的复制过程如下：

1. master 更新数据时，会在事务前产生 GTID，一同记录到 Binlog 日志中。
2. slave 端的 I/O 线程将变更的 Binlog，写入到本地的 relay log 中,读取值是根据`gitd_next变量`，告诉我们 slave 下一个执行哪个 GTID。
3. SQL 线程从 relay log 中获取 GTID，然后对比 slave 端的 Binlog 是否有记录。如果有记录，说明该 GTID 的事务已经执行，slave 会忽略。
4. 如果没有记录，slave 就会从 relay log 中执行该 GTID 的事务，并记录到 Binlog。
5. 在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有二级索引就用全部扫描。

##### GTID 组成

 GTID = source_id:transaction_id

`source_id` 正常即是 `server_uuid`，在第一次启动时生成(函数 `generate_server_uuid`)，并持久化到 `DATADIR/auto.cnf` 文件里。

`transaction_id` 是`顺序化的序列号`(sequence number)，在每台 MySQL 服务器上都是从 1 开始自增长的序列，是事务的唯一标识。

##### GTID 生成

GTID 的生成受 `gtid_next` 控制。

在 Master 上，`gtid_next` 是默认的 `AUTOMATIC`,即 GTID 在每次事务提交时自动生成。它从当前已执行的 GTID 集合(即 gtid_executed)中，找一个大于 0 的未使用的最小值作为下个事务 GTID。在实际的更新事务记录之前将 GTID 写入到 Binlog。

在 Slave 上，从 Binlog 先读取到主库的 GTID(即 set gtid_next 记录)，而后执行的事务采用该 GTID。

##### GTID 的好处

1. GTID 使用 `master_auto_position=1` 代替了 Binlog 的主从复制方案，相比 Binlog 方式更容易搭建主从复制。
2. GTID 方便实现主从之间的 failover（主从切换），不用一步一步的去定位 Binlog日志文件和查找 Binlog 的位点信息。

##### GTID 模式复制局限性

1. 在一个事务里面混合使用引擎，如 Innodb(支持事务)、MyISAM(不支持事务)， 造成多个 GTIDs 和同一个事务相关联出错。
2. `CREATE TABLE…..SELECT` 不能使用，该语句产生的两个 Event。 在某一情况会使用同一个 GTID(同一个 GTID 在 slave 只能被使用一次)：
   - event one：创建表语句 create table
   - event two ：插入数据语句 insert
3. `CREATE TEMPORARY TABLE and DROP TEMPORARY TABLE` 不能在事务内使用 (启用了 `–enforce-gtid-consistency` 参数)。
4. 使用 GTID 复制从库跳过错误时，不支持 `sql_slave_skip_counter` 参数的语法。

##### GTID 主从复制实战

1.Master 主数据库上的操作

在 my.cnf 文件中配置 GTID 主从复制

```sql
Copy[root@mysql-master ~]# cp /etc/my.cnf /etc/my.cnf.bak
[root@mysql-master ~]# >/etc/my.cnf
[root@mysql-master ~]# cat /etc/my.cnf
[mysqld]
datadir = /var/lib/mysql
socket = /var/lib/mysql/mysql.sock
  
symbolic-links = 0
  
log-error = /var/log/mysqld.log
pid-file = /var/run/mysqld/mysqld.pid
  
#GTID:
server_id = 1
gtid_mode = on
enforce_gtid_consistency = on
    
#binlog
log_bin = mysql-bin
log-slave-updates = 1
binlog_format = row
sync-master-info = 1
sync_binlog = 1
   
#relay log
skip_slave_start = 1
```

配置后，重启 MySQL 服务：

```sql
Copy[root@mysql-master ~]# systemctl restart mysqld
```

登录 MySQL，并查看 Master 状态， 发现多了一项 `Executed_Gtid_Set`：

```sql
Copymysql> show master status;
+-------------------+----------+--------------+------------------+-------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+-------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
  
mysql> show global variables like '%uuid%';
+---------------+--------------------------------------+
| Variable_name | Value                                |
+---------------+--------------------------------------+
| server_uuid   | 317e2aad-1565-11e9-9c2e-005056ac6820 |
+---------------+--------------------------------------+
1 row in set (0.00 sec)
```

查看确认 GTID 功能打开:

```sql
Copymysql> show global variables like '%gtid%';
+----------------------------------+-------+
| Variable_name                    | Value |
+----------------------------------+-------+
| binlog_gtid_simple_recovery      | ON    |
| enforce_gtid_consistency         | ON    |
| gtid_executed                    |       |
| gtid_executed_compression_period | 1000  |
| gtid_mode                        | ON    |
| gtid_owned                       |       |
| gtid_purged                      |       |
| session_track_gtids              | OFF   |
+----------------------------------+-------+
8 rows in set (0.00 sec)	
```

查看确认 Binlog 日志功能打开:

```sql
Copymysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```

授权 slave 复制用户，并刷新权限:

```sql
Copy
mysql> flush privileges;
Query OK, 0 rows affected (0.04 sec)
  
mysql> show grants for slave@'172.23.3.66';
+-------------------------------------------------------------------------------+
| Grants for slave@172.23.3.66                                                |
+-------------------------------------------------------------------------------+
| GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'172.23.3.66' |
+-------------------------------------------------------------------------------+
1 row in set (0.00 sec)
  
```

再次查看 master 状态:

```sql
Copymysql> show master status;
+-------------------+----------+--------------+------------------+------------------------------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                        |
+-------------------+----------+--------------+------------------+------------------------------------------+
| mysql-bin.000001 |      622 |              |                  | 317e2aad-1565-11e9-9c2e-005056ac6820:1-2 |
+-------------------+----------+--------------+------------------+------------------------------------------+
1 row in set (0.00 sec)
```

这里需要注意一下：
启动配置之前，同样需要对从服务器进行初始化。对从服务器初始化的方法基本和基于日志点是相同的，只不过在启动了 GTID 模式后，在备份中所记录的就不是备份时的二进制日志文件名和偏移量了，而是记录的是备份时最后的 GTID 值。
需要先在主数据库机器上把目标库备份一下，假设这里目标库是 slave_test：

```sql
Copymysql> CREATE DATABASE slave_test CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected (0.02 sec)
  
mysql> use slave_test;
Database changed
mysql> create table user (id int(10) PRIMARY KEY AUTO_INCREMENT,name varchar(50) NOT NULL);
Query OK, 0 rows affected (0.27 sec)
  
mysql> insert into slave_test.user values(1,"xiaoming"),(2,"xiaohong"),(3,"xiaolv");   
Query OK, 3 rows affected (0.06 sec)
Records: 3  Duplicates: 0  Warnings: 0
  
mysql> select * from slave_test.user;
+----+----------+
| id | name     |
+----+----------+
|  1 | xiaoming |
|  2 | xiaohong |
|  3 | xiaolv   |
+----+----------+
3 rows in set (0.00 sec)
```

把 slave_test 库备份出来:

```sql
Copy[root@mysql-master ~]# mysqldump --single-transaction --master-data=2 --triggers --routines --databases slave_test -uroot -p123456 > /root/user.sql
```

这里有个版本上的问题：

MySQL 5.6 使用 `mysqldump` 备份时，指定备份的具体库，使用 `--database`。

MySQL 5.7 使用 `mysqldump` 备份时，指定备份的具体库，使用`--databases。`

然后把备份的`/root/user.sql` 文件拷贝到 slave 从数据库服务器上。

```sql
Copy[root@mysql-master ~]# rsync -e "ssh -p20" -avpgolr /root/user.sql 
```

到这里主库的操作结束，包含 GTID 的备份数据已经 copy 到从库，下面来进行从库的操作。

2.从库操作

在 my.cnf 文件中配置 GTID 主从复制

与主服务器配置大概一致，除了 server_id 不一致外，从服务器还可以在配置文件里面添加`read_only＝on` ，使从服务器只能进行读取操作，此参数对超级用户无效，并且不会影响从服务器的复制。

```sql
Copy[root@mysql-slave1 ~]# >/etc/my.cnf
[root@mysql-slave1 ~]# vim /etc/my.cnf
[mysqld]
datadir = /var/lib/mysql
socket = /var/lib/mysql/mysql.sock
  
symbolic-links = 0
  
log-error = /var/log/mysqld.log
pid-file = /var/run/mysqld/mysqld.pid
  
#GTID:
server_id = 2
gtid_mode = on
enforce_gtid_consistency = on
    
#binlog
log_bin = mysql-bin
log-slave-updates = 1
binlog_format = row
sync-master-info = 1
sync_binlog = 1
   
#relay log
skip_slave_start = 1
read_only = on
```

配置完成后，重启mysql服务。

```sql
Copy[root@mysql-slave1 ~]# systemctl restart mysql
```

接着将主数据库目标库的备份数据 `user.sql`导入到从数据库里。

```sql
Copy[root@mysql-slave1 ~]# ls /root/user.sql
/root/user.sql
[root@mysql-slave1 ~]# mysql -p123456
.........
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
  
mysql> source /root/user.sql;
  
mysql> select * from slave.test;
+----+----------+
| id | name     |
+----+----------+
|  1 | xiaoming |
|  2 | xiaohong |
|  3 | xiaolv   |
+----+----------+
3 rows in set (0.00 sec)
```

在从数据库里，使用 `change master` 配置主从复制:

```sql
Copymysql> stop slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)
  
mysql> change master to master_host='172.23.3.66',master_user='slave1',master_password='123456',master_auto_position=1;
Query OK, 0 rows affected, 2 warnings (0.26 sec)
  
mysql> start slave;
Query OK, 0 rows affected (0.02 sec)
  
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.23.3.66
                  Master_User: slave1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 1357
               Relay_Log_File: mysql-slave1-relay-bin.000002
                Relay_Log_Pos: 417
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
................
................
            Executed_Gtid_Set: 317e2aad-1565-11e9-9c2e-005056ac6820:1-5
                Auto_Position: 1
  
```

由此，Master 和 Slave 节点已经配置了主从同步关系。接下来你可以自行在主库插入一条数据观察从库是否同步过来。

##### 使用 GTID 添加从库有两种方式

**直接同步主库所有GTID**

如果主库一开始就开启了 GTID，那么可以直接获取主库的所有GTID来同步至从库。但是如果主库 Binlog 日志太多，那么相应同步的时间也会变长。这种方式适用于小数据量的同步。

使用这种方式来同步相应的命令为：

```sql
Copymysql>change master to master_host='xxxxxxx',master_user='xxxxxx',master_password='xxxxx',MASTER_AUTO_POSITION=1;
mysql> start slave;
mysql> stop slave io_thread; #重启 io 线程,刷新状态
mysql> start slave io_thread;
```

**当使用 `MASTER_AUTO_POSITION` 参数的时候，`MASTER_LOG_FILE`，`MASTER_LOG_POS` 参数不能使用。**
**如果想要从 `GTID 配置回 pos`，再次执行这条语句，不过把 MASTER_AUTO_POSITION 置为 0。**

**通过设定范围进行同步**

通过指定 GTID 的范围，然后通过在 slave 设置 `@@GLOBAL.GTID_PURGED` 从而跳过备份包含的 GTID 。

这种方案适用于数据量比较大一次同步需要耗费巨量时间的数据。但同时也有操作复杂的问题，需要你记住每次同步的范围。

用这种方式来同步相应的命令为：

```sql
Copymysql>change master to master_host='xxxxxxx',master_user='xxxxxx',master_password='xxxxx',MASTER_LOG_POS='xxxx';
mysql> start slave;
mysql> stop slave io_thread; #重启 io 线程,刷新状态
mysql> start slave io_thread;
```

这里注意到我们的参数换了：`MASTER_LOG_POS`，该参数表示当前需要同步的 GTID 事务的起点值。

#### 总结[#](https://www.cnblogs.com/rickiyang/p/13856388.html#901928611)

本篇介绍主从复制的两种形式：基于 Binlog 和 位点信息的传统复制方式；基于 Binlog 和 GTID 的新式复制方式。现在很多公司可能还在使用 MySQL 5.6 的版本，所以 GTID不一定可以使用。

本文篇幅较长，大家可以简要查看原理。



## MySQL 学习总结 之 COMPACT 行格式的设计原理

## 一、回顾

MySQL 学习总结系列至此已经第七节了。

从大方向：我们已经学习了 MySQL 的架构设计、InnoDB 的架构设计。

从较为深入的：我们已经学习了 rodo log 和 binlog 配合的两阶段提交协议，了解 缓冲池的设计原理和支持高并发、动态调整的管理机制。

下面，我们将介绍数据行格式：数据是以什么格式存储在数据页中的。

## 二、行存储格式

InnoDB 储存引擎支持有四种行储存格式：COMPACT、Redundant、Dynamic 和 COMPRESSED。

**下面我们将重点介绍 COMPACT 行格式：**

COMPACT 行存储格式大概类似这样：

```csharp
变长字段的长度列表，null值列表，数据头，column01的值，column02的值，column0n的值......
```

ps：为了让磁盘空间得到最大的利用率，每个数据行都是紧紧地挨在一起的。

下面我们将详细介绍 COMPACT 行格式的各个知识点，当你学习完之后，你就晓得即使每行数据紧紧地挨在一起，MySQL 也能精准地将每行数据找出来~

## 三、变长字段如何存储？

#### 1、变长字段的存储问题

我们都知道，varchar 类型是变长的，例如 varchar(50)，那么这个字段值的长度范围：0 ~ 50 个字符。但是，不是每个字段值都刚好50个字符，肯定会有的长有的短。

那么，数据存储时，会按照字段定义时的最大长度来存储值吗？

必须不会的，如果都按照最大长度存储，当出现值不满 50个字符长度时，会浪费磁盘空间和内存空间。

为什么也浪费内存空间，数据不是存放在磁盘么？大家不会忘了缓冲池的作用了吧？哈哈，要记得缓冲池和磁盘数据交换的单位就是数据页而数据行是存放在数据页中的

#### 2、变长字段长度列表

InnoDB 中，利用 **变长字段长度列表** 来解决上面的问题：

1. 变长字段长度列表记录每一个变长字段值的长度，存储的长度是**十六进制**的。
2. 如果有多个变长字段，那么变长字段长度列表是按**逆序存储**的。

下面用一个例子来描述一下变长字段长度列表的使用原理：

```sql
-- 表结构
create table test(
	c1 varchar(10) comment '字段1-变长',
    c2 varchar(5) comment '字段2-变长',
    c3 varchar(20) comment '字段3-变长',
    c4 char(1) comment '字段4-定长',
    c5 char(1) comment '字段5-定长'
) ENGINE=InnoDB;
-- 一行数据
insert into test values('hello','ni','hao','a','a');
```

**我们来算一下他们的长度（十六进制）：**

1. hello 的长度为5，十六进制为 0x05
2. ni 的长度为2，十六进制为 0x02
3. hao 的长度为3，十六进制为 0x03

**那么，实际的存储格式是这样的：**

```css
0x03 0x02 0x05 null值列表 数据头 hello hi hao a a
```

## 四、NULL 值字段如何存储？

#### 1、可为 NULL 字段的存储问题

定义为 default NULL 的字段，值可空可不空。那如果字段值为 NULL，数据行里是怎样存储的呢？是直接存储“NULL”字段吗？

我们分析一下：

1. 如果是，那将会浪费磁盘空间，本来值就是 NULL 的，你现在给我搞了个四个字符大小的字符串。
2. 如果不是，那怎么识别这个字段是否是 NULL 呢？

#### 2、NULL值列表

InnoDB 中，利用 **NULL值列表** 来解决上面的问题：

1. NULL 值列表记录可为 NULL 的字段的情况。
2. 用二进制bit位来标识字段值是否为 NULL。1为 NULL，0 不为 NULL。
3. 如果有多个可为 NULL 的字段，那么 NULL 值列表也是按照**逆序存储**的。
4. 而且 NULL 值列表的位数必须是 **8bit 的N倍**。例如：列表仅仅只有4个bit，则往高位补0，补到 8个bit。

下面用一个例子来描述一下 NULL 值列表 的使用原理：

```sql
-- 表结构
create table test(
	c1 varchar(10) not null comment '字段1-变长',
    c2 varchar(5) comment '字段2-变长',
    c3 char(1) comment '字段3-变长',
    c4 varchar(30) comment '字段4-定长',
    c5 varchar(50) comment '字段5-定长'
) ENGINE=InnoDB;
-- 一行数据
insert into test values('howinfun',null,'m',null,'foshan');
```

**算一下变长字段的长度：**

1. howinfun 的长度为8，十六进制为 0x08
2. foshan 的长度为6，十六进制为 0x06

**统计一下值为 NULL 的字段：**

1. c2 字段为 NULL
2. c4 字段为 NULL

**那么，实际的存储格式是这样的：**

```undefined
0x06 0x08 00000101 数据头 howinfun m foshan
```

#### 3、采用 NULL值列表 和 直接存储“NULL”字符串相比，有多大的存储差距？

到此，我们就可以算一下这两种方案的存储差距有多大了。

1. 一个字节 8个bit，NULL 值列表用二进制 bit 位来标识字段值是否为 NULL；那么就是说，标识8个字段才占用一个字节。
2. 而如果用字符串的方式来存储，而一个"NULL"字符串足足用了四个字节（英文一个字符等于一个字节，中文一个字符等于两个字节），那么同样的8个字段就需要36个字节了。

这差距是非常明显的！

## 五、数据头

COMPACT 行格式中，除了 变长字段长度列表 和 NULL 值列表，就到数据头了。

数据头的大小为 40 个bit位。

下面介绍 40个 bit 分别都有什么信息。

| 名称         | 大小 (bit) | 描述                                                         |
| ------------ | ---------- | ------------------------------------------------------------ |
| 预留位1      | 1          | 没有使用                                                     |
| 预留位2      | 1          | 没有使用                                                     |
| delete_mask  | 1          | 标记该记录是否被删除                                         |
| min_rec_mask | 1          | B+树里每一层的非叶子节点里的最小值都有这个标记               |
| n_owned      | 4          | 表示当前记录拥有的记录数                                     |
| heap_no      | 13         | 表示当前记录在记录堆的位置信息                               |
| record_type  | 3          | 标识当前记录的类型：0代表的是普通类型，1代表的是B+树非叶子节点，2代表的是最小值数据，3代表的是最大值数据。 |
| next_record  | 16         | 表示下一条记录的相对位置                                     |

**那么，我们看一下，加上数据头的实际存储：**

```undefined
0x06 0x08 00000101 0000000000000000000010000000000000011001 howinfun m foshan
```

## 六、一行数据在磁盘是如何存储的

#### 1、字符集编码

上面，我们已经介绍了 COMPACT 行格式了，那么一行数据真正是如何存储的？

我们都知道，在建库和建表时，都可以指定字符集编码。所以，数据都会经过数据库指定的字符集编码后，再进行存储的。

下面用一个例子来描述一下使用原理：

```sql
-- 表结构
create table test(
	c1 varchar(10) not null comment '字段1',
    c2 varchar(5) comment '字段2',
    c3 char(1) comment '字段3',
    c4 varchar(30) comment '字段4',
    c5 varchar(50) comment '字段5'
)
-- 一行数据
insert into test values('howinfun',null,'m',null,'foshan');
```

假设编码后：

1. howinfun 编码后：61616161
2. m 编码后：62
3. foshan编码后：636363

**那么，实际的存储格式是这样的：**

```undefined
0x06 0x08 00000101 0000000000000000000010000000000000011001 61616161 62 636363
```

#### 2、隐藏字段

除了变长字段长度列表、NULL值列表、40个bit位的数据头和真实数据，其实还包含了一些隐藏字段：

1. DB_ROW_ID 字段：如果我们没有指定主键和unique key唯一索引的时候，他就内部自动加一个ROW_ID作为主键。
2. DB_TRX_ID 字段：事务 ID，标识这是哪个事务更新的数据
3. DB_ROLL_PTR 字段：回滚指针，用来进行事务回滚的

**加上隐藏字段后，上面的例子的实际存储可能就是：**

```x86asm
0x06 0x08 00000101 0000000000000000000010000000000000011001 00000000094C（DB_ROW_ID）00000000032D（DB_TRX_ID） EA000010078E（DB_ROL_PTR） 616161 636320 6262626262 
```

ps：括号里只是做说明用的，事实是不存在的。

#### 3、行溢出问题

数据页的默认大小是 16kb，但是某些字段的值可以远远大于 16kb。

例如变长字段类型 varchar(N)：N 最大可为 65532（65kb），这就远远大于 16kb。

当然了，还有 text 和 blog 字段，这些都是大字段，都可以超过 16kb。

如果一行数据的大小超过了 16kb，就会出现行溢出的现象。

**怎么解决？**

当一行数据超了 16kb，会在超了大小的那个字段中，可能仅仅包含他的一部分数据，然后同时包含一个20个字节的指针，指向存储了这行数据超了的部分的其他数据页。

# MySQL Innodb 数据页结构分析

页（Page）是 Innodb 存储引擎用于管理数据的最小磁盘单位。常见的页类型有数据页、Undo 页、系统页、事务数据页等，本文主要分析的是数据页。默认的页大小为 16KB，每个页中至少存储有 2 条或以上的行记录，本文主要分析的是页与行记录的数据结构，有关索引和 B-tree 的部分在后续文章中介绍。

下图是 Innodb 逻辑存储结构图，从上往下依次为：Tablespace、Segment、Extent、Page 以及 Row。本文关注的重点是 Page 和 Row 的数据结构。

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408160535653-284016311.jpg)

## Page 结构

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408162411775-1834436531.jpg)

上图为 Page 数据结构，File Header 字段用于记录 Page 的头信息，其中比较重要的是 FIL_PAGE_PREV 和 FIL_PAGE_NEXT 字段，通过这两个字段，我们可以找到该页的上一页和下一页，实际上所有页通过两个字段可以形成一条双向链表。Page Header 字段用于记录 Page 的状态信息。接下来的 Infimum 和 Supremum 是两个伪行记录，Infimum（下确界）记录比该页中任何主键值都要小的值，Supremum （上确界）记录比该页中任何主键值都要大的值，这个伪记录分别构成了页中记录的边界。

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408164025591-158405878.jpg)

User Records 中存放的是实际的数据行记录，具体的行记录结构将在本文的第二节中详细介绍。Free Space 中存放的是空闲空间，被删除的行记录会被记录成空闲空间。Page Directory 记录着与二叉查找相关的信息。File Trailer 存储用于检测数据完整性的校验和等数据。

## 行记录

Innodb 存储引擎提供了两种格式的行记录：Compact 和 Redundant。

### Compact 行记录

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408164927757-867511928.png)

变长字段长度列表：逆序记录每一个列的长度，如果列的长度小于 255 字节，则使用一个字节，否则使用 2 个字节。该字段的实际长度取决于列数和每一列的长度，因此是变长的。

NULL 标志位：一个字节，表示该行是否有 NULL 值（此处有疑问，8位，最多只能表示 8 列？）

记录头信息：五个字节，其中 next_record 记录了下一条记录的相对位置，一个页中的所有记录使用这个字段形成了一条单链表。

列数据部分：除了记录每一列对应的数据外，还有隐藏列，它们分别是 Transaction ID、Roll Pointer 以及 row_id（当没有指定主键）。

**注意**：此处需要注意固定长度 CHAR 数据类型和变长 VCHAR 数据类型在 Compact 记录下为 NULL 时不占用任何存储空间。

### Redundant 行记录

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408172411477-541622873.png)

字段长度偏移列表：与 Compact 中的变长字段长度列表相同的是它们都是按照列的逆序顺序设置值的，不同的是字段长度偏移列表记录的是偏移量，每一次都需要加上上一次的偏移，同时对于 CHAR 的 NULL 值，会直接按照最大空间记录，而对于 VCHAR 的 NULL 值不占用任何存储空间。

**注意**：此处需要注意 VCHAR 类型和 CHAR 类型在建表时传入的参数是字符长度而不是字节长度，实际的字节长度需要跟编码方式相关联，例如 UTF-8 一个中文字符需要 3 字节来表示，这样 CHAR(10) 以 UTF-8 来表示的话，它的字节长度在 10 - 30 之间。

### 行溢出

我们知道数据页的大小是 16KB，Innodb 存储引擎保证了每一页至少有两条记录，如果一页当中的记录过大，会截取前 768 个字节存入页中，其余的放入 BLOB Page。





# 原文链接：[提问式复习：图文回顾 redo log 相关知识](https://blog.csdn.net/Howinfun/article/details/120579196)

## 1、如何提升 redo日志 的写性能？

- 为了保证 redo日志 不丢失，会在磁盘中开辟一块空间将日志保存起来。但是这样会有一个问题，磁盘的读写性能非常的差。
- 所以 redo日志 和数据页一样，系统都是会分配一块连续的内存，来提升读写性能；数据页对应的是 buffer pool，而 redo日志 对应的是 log buffer。

> buffer pool可以利用「innodb_buffer_pool_size」指定总大小，利用「innodb_buffer_pool_instances」指定实例数，但是必须size大于等于1G才生效。
>
> log buffer 可利用「innodb_log_buffer_size」指定 log buffer 的大小；一片连续的内存空间会被划分为N个512字节大小的block。
>
> log file 可以利用「innodb_log_file_size」指定每个 log file 的大小，利用「innodb_log_files_in_group」指定一共多少个log file。

## 2、redo日志 何时写入log buffer？

- 对底层页面（可能是多个页面）进行一次原子性访问，等于一个MTR，即 Mini Transaction。一个 MTR对应一组 redo日志 。一个事务对应多个语句，一个语句对应多个个MTR，一个MTR对应一组redo日志，即多个 redo日志 。
- 在MTR结束后，会将一组 redo日志 写入到log buffer中。

详情可看下图：
![redo-lsn-offset](https://img-blog.csdnimg.cn/img_convert/c3891de81be76c83899b7f65fb88b14b.png)

## 3、log buffer 中的 redo日志 何时刷盘？

- 当 log buffer 已经被写入约一半左右，下次再写入 redo日志 时，需将 log buffer 的 redo日志 刷到磁盘文件中。
- 当事务结束时，需先将 log buffer 中，被修改的缓存页对应的 redo日志 刷回磁盘中。
- 后台线程刷，大概每隔一秒刷一次 log buffer 中的 redo日志 到磁盘中。
- 执行checkpoint。
- 正常关闭服务器。

## 4、我们都知道每次写入 redo日志 ，都是以组为单位，那么我们怎么知道哪些是一组？

- 在该组中的最后一条 redo日志 后边加上一条特殊类型的 redo日志 ，该类型名称为「MLOG_MULTI_REC_END」，type字段对应的十进制数字为31，该类型的 redo日志 结构很简单，只有一个type字段。

## 5、如何知道下一次redo日志改写到log buffer的哪个位置？

- buf_free全局变量，指向log buffer中下个写入的位置。

## 6、如何知道下次从log buffer的哪个位置开始刷入磁盘？

- buf_next_to_write全局变量，指向log buffer中下个刷回磁盘的位置。

## 7、如何定位 log buffer 中的 redo日志 对应哪些被修改的数据页；在被修改的数据页中，如何定位到对应的是哪些 redo日志 ？

- 修改的缓存页找到对应的 redo日志
  - lsn
    - 首先，出场一个变量，叫lsn，全称：log sequence number，日志序列号。它记录的是，redo日志 的总字节数，初始值为8704。当系统启动，初始化log buffer 时，lsn 值为 8704+12（一个log block header）=8716
    - 接着，log buffer 是由多个block组成的(可以理解为buffer pull的缓存页)，block由三部分组成，log block header（12个字节）、log block body、log block trailer（4个字节）。
    - 当第一个 redo日志 组，如「mt_1」准备被写入，并且一个block能容纳，此时lsn为 8704+12（一个log block header）=8716，假设「mt_1」一共100字节，那么「mt_1」写入后，lsn为8716+100=8816
    - 当第二个 redo日志 组，如「mt_2」准备被写入，并且需要跨block才能容纳，如跨一个（即包含一个log block header和一个log block trailer），开始写入前lsn：8816，假设「mt_2」一共1000个字节，那么「mt_2」写入后，lsn为8816+12（一个log header）+4（一个log tail）+1000=9832
  - flush和lsn
    - 当 MTR 结束时，会将被修改过的数据页对应的数据块放入 flush链表 的表头中，并且给两个参数赋值，分别是 old_modification 和 new_modification：old_m 赋值是 MTR 开始前的 lsn 值，而 new_m 赋值是 MTR 结束时的 lsn 值。
    - 如果一个 MTR 修改的数据页对应的控制块本来就在 flush链表 中，则不调整数据页对应的数据块的位置，只是修改 new_modification 的值，old_modification还 是保持第一次进入 flush链表 时 lsn 的值。
    - 就是说，在 flush链表 中，数据块是根据第一次修改的时间进行倒序排列的。
  - 通过上面，那么我们可以根据flush链表中，数据块的 old_modification 和 new_modification 找到对应的一组 redo日志 ，因为通过 lsn 可以定位到对应 redo日志 在磁盘文件中的偏移量（这个下面会讲解到）。
- redo日志 找到对应的缓存页面
  - redo日志 的通用结构是：type-spaceId ID-page Number-data，即我们可以根据 redo日志 的 space ID 和 page Number 即可找到对应的缓存页。
  - 顺带一提：在 InnoDB 中，有一个哈希表，key为表空间号+页号，value为缓存页地址。这样我们可以通过 space ID 和 page Number 快速定位到对应的缓存页。

## 8、我们知道可以利用 lsn 知道有多少字节数的 redo日志 写入到 log buffer 中，那么我们能有变量对应的知道有多少字节数的 redo日志 被刷入磁盘中吗？

- flushed_to_disk_lsn 全局变量，表示刷到磁盘的日志量。

## 9、lsn 和 log file 的偏移量怎么对得上么？

- lsn 初始值是 8704，随着 redo日志 的不断写入，lsn 不断增大。而 innodb 中，是利用 block 这个结构来存储 redo日志 （不管是 log buffer 还是 log file），而 block 包含三部分，上面已经提到。当 redo日志 不断写入，不断占用 block 的空间，那么 lsn 会增加对应的字节数，当然了，除了body、也算 header 和 trailer。
- log file 是由日志组组成，日志组最大设置100个文件数，每个日志文件也是由多个512字节的block镜像组成，日志组第一个日志文件前4个block镜像用于存储重要信息、如checkpoint等、即前2048个字节不用于存储 redo日志 ，即从2048个字节开始计算 redo日志 的存放量。
- log file 的 log file header 中有一个「LOG_HEADER_START_LSN」属性，标记本 redo日志 文件偏移量2048字节处对应的lsn值。

详情可看下图：
![redo-lsn-offset](https://img-blog.csdnimg.cn/img_convert/8a356a6682f60a7ad7a90586398e1379.png)

## 10、log buffer 中的 redo日志 真的会在事务结束时立马刷回到磁盘中吗？

- 默认是的，这里有一个参数控制：「innodb_flushing_log_at_trx_commit」，默认值是1
  - 0:事务提交，不会立马刷到磁盘中，依赖后台线程刷入，即如果此时MySQL或系统挂掉重启，无法恢复脏页
  - 1:事务提交，会立马将log buffer的 redo日志 刷回磁盘中
  - 2:事务提交，会立马将log buffer的 redo日志 刷到操作系统的缓存中，而不是刷到磁盘中；如果此时MySQL挂掉了，重启后不会影响恢复脏页，而如果是系统挂掉，就无力回天了。

## 11、log file 都是循环使用，即可以覆盖，那么怎么判断是否可以覆盖？

- log file 中可被覆盖，那么首要条件就是 redo日志 对应的脏页已经被刷到磁盘中。
- innodb 有个全局变量：checkpoint_lsn，它记录的是可被覆盖的 redo日志量。初始值就是lsn的初始值，8704。
  - 什么是 checkpoint？
    - 当有脏页被刷到磁盘时，首先在flus链表中拿到最旧的缓存页，即需要拿到链表尾部的控制块，然后拿到 old_modification 的值，然后将这个值赋值给 checkpoint_lsn，因为只要是小于 flush 链表中最旧的控制块的 old_modification 的 lsn，就代表可以被覆盖，毕竟对应的脏页已经被刷到磁盘中了。
    - 接着，将根据当前的 checkpoint_lsn 获取对应日志文件组的偏移量，记录为 checkpoint_offset，checkpoint_no 也需要加1，最后将三个信息记录在日志文件组的 checkpoint1 或 checkpoint2（checkpoint_no为奇数存1，否则存2）。
    - 上面两步称为执行一次checkpoint。
- 我们只需要从日志文件组中的 checkpoint1 和 checkpoint2 拿到信息，然后对比 checkpoint_no 看哪个是最新的，接着拿到checkpoint_lsn，那么 lsn 小于 checkpoint_lsn 的日志都可以被覆盖。

## 12、系统崩溃重启，如何利用 redo日志 进行恢复？

- redo日志 进行崩溃恢复主要是利用上面提到的 checkpoint_lsn，因为 checkpoint_lsn 表示可以覆盖的日志量，则表示 checkpoint_lsn 之前的 redo日志 对应的脏页都已经被刷回到磁盘中。
- 首先从 redo 日志组中拿到 checkpoint1 和 checkpoint2，接着判断谁的 checkpoint_no 大，大的就是最新的一次 checkpoint 执行。
- 接着拿到对应的 checkpoint_offset，那么 checkpoint_offset 后的 redo日志 都需要扫描一遍，然后根据 redo日志 的内容，对数据页进行恢复。

## 13、恢复是扫描一个 redo日志 ，就进行一次恢复吗？

- 问题：
  - 因为根据 redo日志 恢复数据页的变更，是直接更新磁盘中的数据页；扫描一个 redo日志 ，就进行一次恢复，如果存在多个 redo日志 记录同一个数据页的变更，并且不是连续的，那么会导致多次随机IO，性能会非常的差。
- 解决：
  - 所以会有一个哈希表，key为 space ID + page Number，value 为数据页地址。扫描 redo日志 时，会将同一个 space ID + page Number 的 redo日志 都放在同一个槽下。
  - 接着遍历哈希表，执行每一个 space ID + page Number 对应所有的 redo日志 。
- 好处：
  - 避免了多次的随机IO，提升恢复的速度。
  - 按顺序根据 redo日志 进行恢复，避免出现恢复的顺序问题。

详情可看下图：
![redo-恢复](https://img-blog.csdnimg.cn/img_convert/89a8c15a2a3b00a7e83beeb6011cb7e9.png)

## 14、恢复时，如何知道什么时候结束?

- 首先，我们知道，在日志组里，有多个block镜像，然后 redo日志 刷盘，是按顺序填入每个block的，只有前一个block填满了，才接着填下一个
- 接着，每个 block 的大小都是 512 个字节，包括 log block header、log block body 和 log block trailer。在block的页面结构中，log block header 头部有一个「LOG_BLOCK_HDR_DATA_LEN」的属性，该属性值记录了当前block里使用了多少字节的空间。对于被填满的block来说，该值永远为512。
- 最后，所以只管往后面一直扫，直到 log block header 中 「LOG_BLOCK_HDR_DATA_LEN」属性不是512的 block，那么就是恢复的终点了。

## 15、如何兼容脏页已经已经刷回磁盘，但是 redo日志 没有刷回磁盘的场景？

- 场景复现：
  - 当我们提交事务时，会根据参数「innodb_flush_at_trx_commit」来做下一步操作，如果是0或者2，那么此时的日志并没有刷回到磁盘中，而是留在log buffer中或操作系统缓存中。
  - 接着，如果有后台线程将 LRU 链表或 flush 链表的某些脏页刷回磁盘中，刷回后；但是此时对应的 redo日志 还停留在上面提到的两个地方，如果服务器宕机，那么对应的 redo日志 就会丢失了。
  - 因为刷 LRU 链表、flush 链表和刷 redo日志 的后台线程，往往都是不同的线程，无法知道对应的 redo日志 是否已经刷回去。
- 兼容：
  - 每个数据页都有一个称之为 File Header 的部分，在 File Header 里有一个称之为 FIL_PAGE_LSN 的属性，该属性记载了最近一次修改页面时对应的 lsn 值（其实就是页面控制块中的 newest_modification 值）。
  - 如果在做了某次 checkpoint 之后有脏页被刷新到磁盘中，那么该页对应的 FIL_PAGE_LSN 代表的 lsn 值肯定大于 checkpoint_lsn 的值，凡是符合这种情况的页面就不需要重复执行 lsn 值小于 FIL_PAGE_LSN 的 redo日志 了，

最后，祝大家国庆节快乐！



# 原文链接：[MySQL学习总结：提问式回顾 undo log 相关知识](https://blog.csdn.net/Howinfun/article/details/120601165?spm=1001.2014.3001.5501)

### 1、redo 日志支持恢复重做，那么如果是回滚事务中的操作呢，也会有什么日志支持么？

- 也回滚已有操作，那么就是想撤销，对应的有撤销日志，也叫做 undo log。
- undo 日志分为两大类：「TRX_UNDO_INSERT」和「TRX_UNDO_UPDATE」，undo 日志需根据大类分开存储，不能混淆。

> 「TRX_UNDO_INSERT」对应的是insert语句、「TRX_UNDO_UPDATE」对应的是update语句和delete语句

- 问题：那如何定位事务中对哪些记录做了改动？

  - 数据页记录中，有一个隐藏列「trx_id」，用于记录当前操作此记录的事务。
  - MySQL会在内存维护一个全局变量，专门为事务分配事务ID。
  - 每当需要为事务分配ID，则拿到上述全局变量，然后自增1。

  > 每当上述全局变量自增到256的倍数时，需要将此值刷新到磁盘中（系统空间表页号为5的页面的 Max Trx ID 属性中）。

- 记录修改了，如何定位到对应的 undo log？

  - 在记录中有一个隐藏列「roll_point」，它会指向对应的 undo 日志。

### 2、undo 日志都是存在哪些地方，事务对表进行编辑，怎么分配的？

- InnoDB支持128个回滚段，一个回滚段对应一个「Rollback Segment Header」页面。

- 回滚段分为两大类：第0号、第33～127号属于一类，0号存放在系统表空间、其他的可以放在系统表空间或者自己配置 undo 表空间，这类用于存放普通表改动对应的 undo 日志；第1～32号属于一类，存放在临时表空间中，这类用于存放临时表改动对应 undo 日志。

- InnoDB在系统表空间的5号页面的某个区域中，包含了128个8个字节大小的格子，用于保存128个「Rollback Segment Header」页面的地址。

  -「Rollback Segment Header」页面有一个重要部分是「TRX_RSEG_UNDO_SLOTS」：它表示各个 Undo 页面链表的 first undo page 的页号集合，也叫 undo slots 集合。

  - 因为一个页号占用4字节，而「TRX_RSEG_UNDO_SLOTS」一共是4096个字节，所以一共可以存储1024个lot。
  - 每个 slot 的初始值是 FIL_NULL，表示没有分配给其他事务使用。

- 当事务需要分配 undo 页面链表时，

  - 先到回滚段对应的两个 cached 链表找是否有可用的 undo 页面；

  > insert undo cache 链表和 update undo cache 链表。

  - 如果有则直接重用，否则需要回到「Rollback Segment Header」页中继续寻找；
  - 在「Rollback Segment Header」页中便利1024个 slot，看 slot 的值是否为 FIL_NULL，如果是的话，则申请一个 undo 页面作为该 undo 页面链表的 first undo page，接着将此页面的页号赋值给当前 slot。
  - 否则，表明已经有事务占用此 slot，需要继续往下寻找下一个 slot。
  - 如果到了最后一个回滚段的最后一个 slot，都没有找到可用的 slot，则给客户端返回异常：Too many active concurrent transactions。

大概如下图：
![undo](https://gitee.com/Howinfun/image/raw/master/mysql/undo.png)

### 3、通过上面，我们都知道「roll_point」可以定位到对应的 undo 日志，但 undo 日志也是保存在磁盘中的，那又是怎么定位的呢？

- 我们都知道聚簇索引以及二级索引，都是类型为「FIL_PAGE_INDEX」的页面；而 undo 日志也是存储在磁盘的，它对应的页面类型是「FIL_PAGE_UNDO_LOG」。
- roll_point 组成部分：
  - is_insert：是否是「TRX_UNDO_INSERT」大类的 undo 日志
  - rseg id：回滚段编号
  - page number：指针，指向 undo 日志所在页面的页号。
  - offset：指针，指向 undo 日志在页面中的偏移量。
- 所以，我们可以根据「roll_point」中的属性，找到对应的回滚段、接着找到对应的页面，最好定位到页面中的偏移量。

### 4、如果一个页面无法存储当前事务生成的日志，要多个页面才能完成，undo 日志间如何联系？

- undo 日志页面有一个特有的部分：「Undo Page Header」。

> 数据页都有一个共有的部分：「File Header」，页面间可利用链表完成联系，就是靠「File Header」 中的「FIL_PAGE_PREV」和「FIL_PAGE_NEXT」属性。
> 但这是页面之间的链表，如果要做到 undo 日志的链表，还需更细的连接信息。

- TRX_UNDO_PAGE_TYPE：上面提到的 und 日志分为两个大类：「TRX_UNDO_INSERT」和「TRX_UNDO_UPDATE」
- TRX_UNDO_PAGE_START：当前页面存储第一条 undo 日志的开始偏移量
- TRX_UNDO_PAGE_FREE：当前页面存储最后一条 undo 日志的结束偏移量
- TRX_UNDO_PAGE_NODE：
  - Prev Node Page Number：前一个节点的页号
  - Prev Node Offset：前一个节点页内的偏移量
  - Next Node Page Number：后一个节点的页号
  - Next Node Offset：后一个节点页内的偏移量
- 所以，可以利用每个 undo 日志页「Undo Page Header」的「TRX_UNDO_PAGE_NODE」属性来组成一个链表。

### 5、undo 日志也支持重用么？如果支持，如何覆盖 undo 日志？

- 重用条件：
  - 首要条件：undo 页面链表对应的事务已经提交
  - 第二：undo 页面链表只包含一个 undo 页面
  - 第三：该页面已使用的空间小于整个页面空间的3/4
- 重用策略：
  - insert undo 链表：因为对于新增记录，只要事务提交了，对应的 undo 日志就没啥用了，所以可以直接覆盖。
  - update undo 链表：对于更新/删除记录，即使提交了，也不能立马删除对应的 undo 日志，因为 MVCC 需要利用此链表做文章；所以只能在后面接着写入 undo 日志，即一个 undo 页面，写入多组 undo 日志。

### 6、如果 undo 日志支持重用，那怎么知道从哪里开始写入第二组 undo 日志？

- undo 日志的页面有一个非常重要的部分：Undo Log Header。
- 它包含：
  - TRX_UNDO_TRX_ID：本组 undo 日志对应的事务id
  - TRX_UNDO_TRX_NO：事务提顺序，事务提交后会生成一个序号；先提交的序号小。
  - TRX_UNDO_LOG_START：本组 undo 日志 中第一条 undo 日志在页面中的偏移量
  - TRX_UNDO_NEXT_LOG：下一组 undo 日志在页面中开始的偏移量（支持 undo 日志页面复用）
  - TRX_UNDO_PREV_LOG：上一组 undo 日志在页面中开始的偏移量（支持 undo 日志页面复用）
  - ......
- 因此，只需拿到「TRX_UNDO_NEXT_LOG」对应的偏移量，即可知道在哪里开始继续写入第二组 undo 日志。

### 7、insert语句和 undo 日志

- 插入一条类型为「TRX_UNDO_INSERT_REC」 的 undo 日志，用于支持回滚操作。
- undo 日志里最主要是记录了插入的记录的主键信息：<len,value>列表
  - len 为主键类型所占存储空间的长度，例如主键类型为int，占用4字节，那么len为4
  - value 为主键的真实值，例如 id 列为主键，插入记录的主键 id=2，那么value为2

### 8、delete语句和 undo 日志

- 删除阶段：
  - 插入一条类型为「TRX_UNDO_DEL_MARK_REC」的 undo 日志，用于支持回滚操作。
    - 将该记录的 trx_id 和 roll_point 旧值记录到 undo 日志对应的属性中。
  - delete_mark 阶段：将记录的「delete_flag」置为1，表示已经删除
    - 还是保留在正常记录链表中，保留这个中间状态是为了支持MVCC
    - 还会修改 trx_id、roll_point等隐藏列
  - purge阶段：当事务提交时，会有专门的线程真正删除此记录
    - 这里的真正删除指的是，将记录从正常记录链表中移除，加入到垃圾记录链表的表头。
      - 将记录的 next_record 指向 Page Header 中的 PAGE_FREE 属性
      - 将 Page Header 的 PAGE_FREE 属性执行此记录
    - 修改 Page Header 中的 PAGE_GARBAGE，表示页面中可重用的字节数量
    - purge 阶段不需要 undo 日志，因为此阶段在事务提交后执行。
- 扩展点：
  - 每当新插入一条记录，都会先判断垃圾链表的头节点代表的已删除记录的存储空间是否满足新纪录，如果满足，直接复用。
    - 疑问：如果新插入的记录一直都无法使用垃圾链表的头节点对应的存储空间，岂不是一直会存在碎片空间？下面第三点会解释！
  - 如果无法满足，则需要直接向页面申请新的空间来存储。
  - 但如果此时页面没有足够的空间来存储新纪录，那么就会判断「PAGE_GARBAGE」中的碎片空间和剩余的可用空间加起来是否能满足，如果可以，那么就会进行页面的重新组织
    - 开辟一个临时页面，把本页面的记录依次插入
    - 接着将临时页面复制到本页面，接着释放碎片空间。
    - 疑问：如果「PAGE_GARBAGE」中的碎片空间和剩余的可用空间加起来都不能满足呢？是不是会有页面分裂？
  - 否则，如果页面的碎片空间和剩余空间都不足以存放新纪录，那么只能进行页面分裂。

### 9、update语句和 undo 日志

- update 分两种情况：更新主键和不更新主键

- 不更新主键：也分为两种情况，一种是更新前后所占存储空间大小不变；另外一种是，更新前后所占存储空间变大或变小。

  - 就地更新：
    - 在原有记录更新
    - 插入一条类型为「TRX_UNDO_UPD_EXIST_REC」的 undo 日志，用于支持回滚操作。
      - 包含主键各列信息(<len,value>列表)、索引列各列信息(<pos,len,value>列表)、被更新的列更新前信息。
  - 先删除再插入：
    - 用户线程同步删除旧记录，更新相关的页面统计信息。
    - 在当前页申请新的空间插入一条变更后的记录，如果页面剩余的存储空间不足，则需要进行页面分裂。
    - 插入一条类型为「TRX_UNDO_UPD_EXIST_REC」的 undo 日志

- 更新主键：

  - 对旧id记录执行删除操作，注意：这里只是执行 delete_mark 阶段，避免其他事务无法访问此记录。

  > 插入一条类型为「TRX_UNDO_DEL_MARK_REC」的 undo 日志

  - 接着根据更新后的个列值创建一条新纪录，并插入到聚簇索引中。

  > 插入一条类型为「TRX_UNDO_INSERT_REC」 的 undo 日志







<!-- GFM-TOC -->
* [MySQL](#mysql)
    * [一、索引](#一索引)
        * [B+ Tree 原理](#b-tree-原理)
        * [MySQL 索引](#mysql-索引)
        * [索引优化](#索引优化)
        * [索引的优点](#索引的优点)
        * [索引的使用条件](#索引的使用条件)
    * [二、查询性能优化](#二查询性能优化)
        * [使用 Explain 进行分析](#使用-explain-进行分析)
        * [优化数据访问](#优化数据访问)
        * [重构查询方式](#重构查询方式)
    * [三、存储引擎](#三存储引擎)
        * [InnoDB](#innodb)
        * [MyISAM](#myisam)
        * [比较](#比较)
    * [四、数据类型](#四数据类型)
        * [整型](#整型)
        * [浮点数](#浮点数)
        * [字符串](#字符串)
        * [时间和日期](#时间和日期)
    * [五、切分](#五切分)
        * [水平切分](#水平切分)
        * [垂直切分](#垂直切分)
        * [Sharding 策略](#sharding-策略)
        * [Sharding 存在的问题](#sharding-存在的问题)
    * [六、复制](#六复制)
        * [主从复制](#主从复制)
        * [读写分离](#读写分离)
    * [参考资料](#参考资料)
    <!-- GFM-TOC -->


## 一、索引

### B+ Tree 原理

#### 1. 数据结构

B Tree 指的是 Balance Tree，也就是平衡树。平衡树是一颗查找树，并且所有叶子节点位于同一层。

B+ Tree 是基于 B Tree 和叶子节点顺序访问指针进行实现，它具有 B Tree 的平衡性，并且通过顺序访问指针来提高区间查询的性能。

在 B+ Tree 中，一个节点中的 key 从左到右非递减排列，如果某个指针的左右相邻 key 分别是 key<sub>i</sub> 和 key<sub>i+1</sub>，且不为 null，则该指针指向节点的所有 key 大于等于 key<sub>i</sub> 且小于等于 key<sub>i+1</sub>。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/33576849-9275-47bb-ada7-8ded5f5e7c73.png" width="350px"> </div><br>

#### 2. 操作

进行查找操作时，首先在根节点进行二分查找，找到一个 key 所在的指针，然后递归地在指针所指向的节点进行查找。直到查找到叶子节点，然后在叶子节点上进行二分查找，找出 key 所对应的 data。

插入删除操作会破坏平衡树的平衡性，因此在进行插入删除操作之后，需要对树进行分裂、合并、旋转等操作来维护平衡性。

#### 3. 与红黑树的比较

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库系统普遍采用 B+ Tree 作为索引结构，这是因为使用 B+ 树访问磁盘数据有更高的性能。

（一）B+ 树有更低的树高

平衡树的树高 O(h)=O(log<sub>d</sub>N)，其中 d 为每个节点的出度。红黑树的出度为 2，而 B+ Tree 的出度一般都非常大，所以红黑树的树高 h 很明显比 B+ Tree 大非常多。

（二）磁盘访问原理

操作系统一般将内存和磁盘分割成固定大小的块，每一块称为一页，内存与磁盘以页为单位交换数据。数据库系统将索引的一个节点的大小设置为页的大小，使得一次 I/O 就能完全载入一个节点。

如果数据不在同一个磁盘块上，那么通常需要移动制动手臂进行寻道，而制动手臂因为其物理结构导致了移动效率低下，从而增加磁盘数据读取时间。B+ 树相对于红黑树有更低的树高，进行寻道的次数与树高成正比，在同一个磁盘块上进行访问只需要很短的磁盘旋转时间，所以 B+ 树更适合磁盘数据的读取。

（三）磁盘预读特性

为了减少磁盘 I/O 操作，磁盘往往不是严格按需读取，而是每次都会预读。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道，并且只需要很短的磁盘旋转时间，速度会非常快。并且可以利用预读特性，相邻的节点也能够被预先载入。

### MySQL 索引

索引是在存储引擎层实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。

#### 1. B+Tree 索引

是大多数 MySQL 存储引擎的默认索引类型。

因为不再需要进行全表扫描，只需要对树进行搜索即可，所以查找速度快很多。

因为 B+ Tree 的有序性，所以除了用于查找，还可以用于排序和分组。

可以指定多个列作为索引列，多个索引列共同组成键。

适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于最左前缀查找。如果不是按照索引列的顺序进行查找，则无法使用索引。

InnoDB 的 B+Tree 索引分为主索引和辅助索引。主索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/45016e98-6879-4709-8569-262b2d6d60b9.png" width="350px"> </div><br>

辅助索引的叶子节点的 data 域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/7c349b91-050b-4d72-a7f8-ec86320307ea.png" width="350px"> </div><br>

#### 2. 哈希索引

哈希索引能以 O(1) 时间进行查找，但是失去了有序性：

- 无法用于排序与分组；
- 只支持精确查找，无法用于部分查找和范围查找。

InnoDB 存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。

#### 3. 全文索引

MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。

查找条件使用 MATCH AGAINST，而不是普通的 WHERE。

全文索引使用倒排索引实现，它记录着关键词到其所在文档的映射。

InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。

#### 4. 空间数据索引

MyISAM 存储引擎支持空间数据索引（R-Tree），可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。

### 索引优化

#### 1. 独立的列

在进行查询时，索引列不能是表达式的一部分，也不能是函数的参数，否则无法使用索引。

例如下面的查询不能使用 actor_id 列的索引：

```sql
SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;
```

#### 2. 多列索引

在需要使用多个列作为条件进行查询时，使用多列索引比使用多个单列索引性能更好。例如下面的语句中，最好把 actor_id 和 film_id 设置为多列索引。

```sql
SELECT film_id, actor_ id FROM sakila.film_actor
WHERE actor_id = 1 AND film_id = 1;
```

#### 3. 索引列的顺序

让选择性最强的索引列放在前面。

索引的选择性是指：不重复的索引值和记录总数的比值。最大值为 1，此时每个记录都有唯一的索引与其对应。选择性越高，每个记录的区分度越高，查询效率也越高。

例如下面显示的结果中 customer_id 的选择性比 staff_id 更高，因此最好把 customer_id 列放在多列索引的前面。

```sql
SELECT COUNT(DISTINCT staff_id)/COUNT(*) AS staff_id_selectivity,
COUNT(DISTINCT customer_id)/COUNT(*) AS customer_id_selectivity,
COUNT(*)
FROM payment;
```

```html
   staff_id_selectivity: 0.0001
customer_id_selectivity: 0.0373
               COUNT(*): 16049
```

#### 4. 前缀索引

对于 BLOB、TEXT 和 VARCHAR 类型的列，必须使用前缀索引，只索引开始的部分字符。

前缀长度的选取需要根据索引选择性来确定。

#### 5. 覆盖索引

索引包含所有需要查询的字段的值。

具有以下优点：

- 索引通常远小于数据行的大小，只读取索引能大大减少数据访问量。
- 一些存储引擎（例如 MyISAM）在内存中只缓存索引，而数据依赖于操作系统来缓存。因此，只访问索引可以不使用系统调用（通常比较费时）。
- 对于 InnoDB 引擎，若辅助索引能够覆盖查询，则无需访问主索引。

### 索引的优点

- 大大减少了服务器需要扫描的数据行数。

- 帮助服务器避免进行排序和分组，以及避免创建临时表（B+Tree 索引是有序的，可以用于 ORDER BY 和 GROUP BY 操作。临时表主要是在排序和分组过程中创建，不需要排序和分组，也就不需要创建临时表）。

- 将随机 I/O 变为顺序 I/O（B+Tree 索引是有序的，会将相邻的数据都存储在一起）。

### 索引的使用条件

- 对于非常小的表、大部分情况下简单的全表扫描比建立索引更高效；

- 对于中到大型的表，索引就非常有效；

- 但是对于特大型的表，建立和维护索引的代价将会随之增长。这种情况下，需要用到一种技术可以直接区分出需要查询的一组数据，而不是一条记录一条记录地匹配，例如可以使用分区技术。

## 二、查询性能优化

### 使用 Explain 进行分析

Explain 用来分析 SELECT 查询语句，开发人员可以通过分析 Explain 结果来优化查询语句。

比较重要的字段有：

- select_type : 查询类型，有简单查询、联合查询、子查询等
- key : 使用的索引
- rows : 扫描的行数

### 优化数据访问

#### 1. 减少请求的数据量

- 只返回必要的列：最好不要使用 SELECT * 语句。
- 只返回必要的行：使用 LIMIT 语句来限制返回的数据。
- 缓存重复查询的数据：使用缓存可以避免在数据库中进行查询，特别在要查询的数据经常被重复查询时，缓存带来的查询性能提升将会是非常明显的。

#### 2. 减少服务器端扫描的行数

最有效的方式是使用索引来覆盖查询。

### 重构查询方式

#### 1. 切分大查询

一个大查询如果一次性执行的话，可能一次锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但重要的查询。

```sql
DELETE FROM messages WHERE create < DATE_SUB(NOW(), INTERVAL 3 MONTH);
```

```sql
rows_affected = 0
do {
    rows_affected = do_query(
    "DELETE FROM messages WHERE create  < DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000")
} while rows_affected > 0
```

#### 2. 分解大连接查询

将一个大连接查询分解成对每一个表进行一次单表查询，然后在应用程序中进行关联，这样做的好处有：

- 让缓存更高效。对于连接查询，如果其中一个表发生变化，那么整个查询缓存就无法使用。而分解后的多个查询，即使其中一个表发生变化，对其它表的查询缓存依然可以使用。
- 分解成多个单表查询，这些单表查询的缓存结果更可能被其它查询使用到，从而减少冗余记录的查询。
- 减少锁竞争；
- 在应用层进行连接，可以更容易对数据库进行拆分，从而更容易做到高性能和可伸缩。
- 查询本身效率也可能会有所提升。例如下面的例子中，使用 IN() 代替连接查询，可以让 MySQL 按照 ID 顺序进行查询，这可能比随机的连接要更高效。

```sql
SELECT * FROM tag
JOIN tag_post ON tag_post.tag_id=tag.id
JOIN post ON tag_post.post_id=post.id
WHERE tag.tag='mysql';
```

```sql
SELECT * FROM tag WHERE tag='mysql';
SELECT * FROM tag_post WHERE tag_id=1234;
SELECT * FROM post WHERE post.id IN (123,456,567,9098,8904);
```

## 三、存储引擎

### InnoDB

是 MySQL 默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

实现了四个标准的隔离级别，默认级别是可重复读（REPEATABLE READ）。在可重复读隔离级别下，通过多版本并发控制（MVCC）+ Next-Key Locking 防止幻影读。

主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。

内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。

支持真正的在线热备份。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。

### MyISAM

设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。

提供了大量的特性，包括压缩表、空间数据索引等。

不支持事务。

不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。

可以手工或者自动执行检查和修复操作，但是和事务恢复以及崩溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。

如果指定了 DELAY_KEY_WRITE 选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大的提升写入性能，但是在数据库或者主机崩溃时会造成索引损坏，需要执行修复操作。

### 比较

- 事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。

- 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。

- 外键：InnoDB 支持外键。

- 备份：InnoDB 支持在线热备份。

- 崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。

- 其它特性：MyISAM 支持压缩表和空间数据索引。

## 四、数据类型

### 整型

TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT 分别使用 8, 16, 24, 32, 64 位存储空间，一般情况下越小的列越好。

INT(11) 中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。

### 浮点数

FLOAT 和 DOUBLE 为浮点类型，DECIMAL 为高精度小数类型。CPU 原生支持浮点运算，但是不支持 DECIMAl 类型的计算，因此 DECIMAL 的计算比浮点类型需要更高的代价。

FLOAT、DOUBLE 和 DECIMAL 都可以指定列宽，例如 DECIMAL(18, 9) 表示总共 18 位，取 9 位存储小数部分，剩下 9 位存储整数部分。

### 字符串

主要有 CHAR 和 VARCHAR 两种类型，一种是定长的，一种是变长的。

VARCHAR 这种变长类型能够节省空间，因为只需要存储必要的内容。但是在执行 UPDATE 时可能会使行变得比原来长，当超出一个页所能容纳的大小时，就要执行额外的操作。MyISAM 会将行拆成不同的片段存储，而 InnoDB 则需要分裂页来使行放进页内。

在进行存储和检索时，会保留 VARCHAR 末尾的空格，而会删除 CHAR 末尾的空格。

### 时间和日期

MySQL 提供了两种相似的日期时间类型：DATETIME 和 TIMESTAMP。

#### 1. DATETIME

能够保存从 1000 年到 9999 年的日期和时间，精度为秒，使用 8 字节的存储空间。

它与时区无关。

默认情况下，MySQL 以一种可排序的、无歧义的格式显示 DATETIME 值，例如“2008-01-16 22:37:08”，这是 ANSI 标准定义的日期和时间表示方法。

#### 2. TIMESTAMP

和 UNIX 时间戳相同，保存从 1970 年 1 月 1 日午夜（格林威治时间）以来的秒数，使用 4 个字节，只能表示从 1970 年到 2038 年。

它和时区有关，也就是说一个时间戳在不同的时区所代表的具体时间是不同的。

MySQL 提供了 FROM_UNIXTIME() 函数把 UNIX 时间戳转换为日期，并提供了 UNIX_TIMESTAMP() 函数把日期转换为 UNIX 时间戳。

默认情况下，如果插入时没有指定 TIMESTAMP 列的值，会将这个值设置为当前时间。

应该尽量使用 TIMESTAMP，因为它比 DATETIME 空间效率更高。

## 五、切分

### 水平切分

水平切分又称为 Sharding，它是将同一个表中的记录拆分到多个结构相同的表中。

当一个表的数据不断增多时，Sharding 是必然的选择，它可以将数据分布到集群的不同节点上，从而缓存单个数据库的压力。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/63c2909f-0c5f-496f-9fe5-ee9176b31aba.jpg" width=""> </div><br>

### 垂直切分

垂直切分是将一张表按列切分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不同的库中，例如将原来的电商数据库垂直切分成商品数据库、用户数据库等。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e130e5b8-b19a-4f1e-b860-223040525cf6.jpg" width=""> </div><br>

### Sharding 策略

- 哈希取模：hash(key) % N；
- 范围：可以是 ID 范围也可以是时间范围；
- 映射表：使用单独的一个数据库来存储映射关系。

### Sharding 存在的问题

#### 1. 事务问题

使用分布式事务来解决，比如 XA 接口。

#### 2. 连接

可以将原来的连接分解成多个单表查询，然后在用户程序中进行连接。

#### 3. ID 唯一性

- 使用全局唯一 ID（GUID）
- 为每个分片指定一个 ID 范围
- 分布式 ID 生成器 (如 Twitter 的 Snowflake 算法)

## 六、复制

### 主从复制

主要涉及三个线程：binlog 线程、I/O 线程和 SQL 线程。

-   **binlog 线程**  ：负责将主服务器上的数据更改写入二进制日志（Binary log）中。
-   **I/O 线程**  ：负责从主服务器上读取二进制日志，并写入从服务器的中继日志（Relay log）。
-   **SQL 线程**  ：负责读取中继日志，解析出主服务器已经执行的数据更改并在从服务器中重放（Replay）。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/master-slave.png" width=""> </div><br>

### 读写分离

主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。

读写分离能提高性能的原因在于：

- 主从服务器负责各自的读和写，极大程度缓解了锁的争用；
- 从服务器可以使用 MyISAM，提升查询性能以及节约系统开销；
- 增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/master-slave-proxy.png" width=""> </div><br>

## 参考资料

- BaronScbwartz, PeterZaitsev, VadimTkacbenko, 等. 高性能 MySQL[M]. 电子工业出版社, 2013.
- 姜承尧. MySQL 技术内幕: InnoDB 存储引擎 [M]. 机械工业出版社, 2011.
- [20+ 条 MySQL 性能优化的最佳经验](https://www.jfox.info/20-tiao-mysql-xing-nen-you-hua-de-zui-jia-jing-yan.html)
- [服务端指南 数据存储篇 | MySQL（09） 分库与分表带来的分布式困境与应对之策](http://blog.720ui.com/2017/mysql_core_09_multi_db_table2/ "服务端指南 数据存储篇 | MySQL（09） 分库与分表带来的分布式困境与应对之策")
- [How to create unique row ID in sharded databases?](https://stackoverflow.com/questions/788829/how-to-create-unique-row-id-in-sharded-databases)
- [SQL Azure Federation – Introduction](http://geekswithblogs.net/shaunxu/archive/2012/01/07/sql-azure-federation-ndash-introduction.aspx "Title of this entry.")
- [MySQL 索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)
- [MySQL 性能优化神器 Explain 使用分析](https://segmentfault.com/a/1190000008131735)
- [How Sharding Works](https://medium.com/@jeeyoungk/how-sharding-works-b4dec46b3f6)
- [大众点评订单系统分库分表实践](https://tech.meituan.com/dianping_order_db_sharding.html)
- [B + 树](https://zh.wikipedia.org/wiki/B%2B%E6%A0%91)
