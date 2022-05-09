[TOC]





# 简介

## 概述

1. HBase是Apache提供的一个==基于Hadoop的、开源的、有版本的、分布式的、可扩展的、能够存储大量数据的非关系型数据库==
2. HBase是Doug Cutting在Yahoo工作期间，根据Google的论文《The BigTable》来实现的，所以HBase和BigTable的设计思想和实现原理一模一样，只是BigTable是用C语言实现的，而HBase使用Java实现的
3. 区别于传统数据库的行存储，HBase面向列进行存储，底层基于Key-Value结构存储
4. HBase能够提供低延迟的数据查询能力，其原因是底层充分利用了缓存机制以及复杂的数据结构和算法来实现





## 行存储和列存储

1. HBase允许对数据进行==随机==且实时的读取大量数据 - billions of rows X millions of columns

2. HBase借鉴了列存储的思想，但是HBase并不完全是列存储

   ![](https://note.youdao.com/yws/api/personal/file/B5BB29BA20F740CEA368C36408D395BE?method=download&shareKey=c2719be52573445910bb3ded36623c60)

3. HBase中不支持SQL语句，==提供了一套全新的操作指令==

4. HBase不强调列，列可以动态增删

5. HBase适合存储稀疏数据，例如半结构化数据，也可以存储结构化数据



## 特点

1. 分布式架构：HBase是通过集群来存储数据的，数据最终要落地到HDFS上
2. 是一种NoSQL的非关系型数据库，不符合关系型数据库的范式
3. 面向列存储，底层基于Key-Value结构
4. 适合存储半结构化、非结构化的数据
5. 适合存储稀疏的数据，空的数据不占用空间
6. 提供实时的增删改查的能力，但是不提供严格的事务机制，只能在行级别提供事务



## 关系型非关系型数据库

1. **传统关系型数据库的缺陷：**

   1. 高并发读写的瓶颈：Web      2.0网站要根据用户个性化信息来实时生成动态页面和提供动态信息，所以基本上无法使用静态化技术，因此数据库并发负载非常高，可能峰值会达到每秒上万次读写请求。关系型数据库应付上万次SQL查询还勉强顶得住，但是应付上万次SQL写数据请求，硬盘I/O却无法承受。其实对于普通的BBS网站，往往也存在相对高并发写请求的需求，例如，人人网的实时统计在线用户状态，记录热门帖子的点击次数，投票计数等，这是一个相当普遍的业务需求
   2. 可扩展性的限制：在基于Web的架构中，数据库是最难以进行横向扩展的，当应用系统的用户量和访问量与日俱增时，数据库系统却无法像Web Server和App Server那样简单地通过添加更多的硬件和服务节点来扩展性能和负载能力。对于很多需要提供24小时不间断服务的网站来说，对数据库系统进行升级和扩展是非常痛苦的事情，往往需要停机维护和数据迁移，而不能通过横向添加节点的方式实现无缝扩展
   3. 事务一致性的负面影响：事务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。保证数据库一致性是指当事务完成时，必须使所有数据都具有一致的状态。在关系型数据库中，所有的规则必须应用到事务的修改上，以便维护所有数据的完整性，这随之而来的是性能的大幅度下降。很多Web系统并不需要严格的数据库事务，对读一致性的要求很低，有些场合对写一致性要求也不高。因此数据库事务管理成了高负载下的一个沉重负担
   4. 复杂SQL查询的弱化：任何大数据量的Web系统都非常忌讳几个大表间的关联查询，以及复杂的数据分析类型的SQL查询，特别是SNS类型的网站，从需求以及产品设计角度就避免了这种情况的产生。更多的情况往往只是单表的主键查询，以及单表的简单条件分页查询，SQL的功能被极大地弱化了，所以这部分功能不能得到充分发挥

2. **NoSQL数据库的优势:**

   1. 扩展性强：NoSQL数据库种类繁多，但是一个共同的特点就是去掉关系型数据库的关系特性，数据之间是弱关系，非常容易扩展。一般来说，NoSql数据库的数据结构都是Key-Value字典式存储结构。例如，HBase、Cassandra等系统的水平扩展性能非常优越，非常容易实现支撑数据从TB到PB级别的过渡
   2. 并发性能好：NoSQL数据库具有非常良好的读写性能，尤其在大数据量下，同样表现优秀。当然这需要有优秀的数据结构和算法做支撑
   3. 数据模型灵活：NoSQL无须事先为要存储的数据建立字段，随时可以存储自定义的数据格式。而在关系型数据库中，增删字段是一件非常麻烦的事情。对于数据量非常大的表，增加字段简直就是一场噩梦。NoSQL允许使用者随时随地添加字段，并且字段类型可以是任意格式。



# 基本概念

## 概述

> HBase以表的形式存储数据。
>
> 表有行和列族组成。列族划分为若干个列

![](https://note.youdao.com/yws/api/personal/file/185A48BDD9D044E781FE3DFAD294DDFA?method=download&shareKey=b2c73113062f55dfb689581793559e51)

## Row Key - 行键

1. HBase本质上也是一种Key-Value存储系统。Key相当于RowKey，Value相当于列族数据的集合

2. 与NoSQL数据库们一样，RowKey是用索引记录的主键

3. 访问HBase Table中的行，只有三种方式：

   - 通过单个Row Key访问
   - 通过Row Key的Range
   - 全表扫描

4. RowKey行键可以是任意字符串，在HBase内部，RowKey保存为字节数组

   > 字符串最大长度：64KB，实际应用中长度一般为10-100byte

5. 存储时，数据按照RowKey的==字典==排序存储。设计Key时，要充分考虑排序存储这个特性，将经常一起读取的行存储放到一起（位置相关性）



## Column Family - 列族（列簇）

1. HBase表中的每个列，都归属与某个列族

2. 列族是表的Schema的一部分（列不是），==列族必须在使用表之前定义==

3. 列名都以列族作为前缀。

   - 例如：`courses:history`，`courses:math`。都属于`courses`这列族

4. 访问控制、磁盘和内存的使用统计都是在列族层面进行的。

   > 实际应用中，列族上的控制权限能帮助管理不同类型的应用：允许一些应用可以添加新的基本数据、一些应用可以读取基本数据并创建继承的列族、一些应用则只允许浏览数据（甚至可能因为隐私的原因不能浏览所有数据）



## Cell(单元)与时间戳

1. 由`{rowkey，column(=<family> + <label>),version}`唯一确定一个单元

2. Cell中的数据是没有类型的，全部是字节码形式存储的

3. 每个Cell都保存着同一份数据的多个版本，版本通过时间戳来索引

4. 时间戳的类型是64位整形。

   > 时间戳可以由HBase在数据写入时自动赋值，此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显式赋值

5. 如果应用程序要避免数据版本冲突，就必须自己生成具有唯一性的时间戳

6. 每个Cell中，不同版本的数据按照时间倒序排序，及最新的数据排在最前面

7. 为了避免数据存在多版本造成的管理(存储、索引)负担，HBase提供了两种数据版本回收方式

   - 保存数据的最后n个版本
   - 保存最近一段时间内的版本（比如最近7天）

   > 用户可以针对每个列族进行设置



## Version - 版本

1. 每一个数据都附带一个时间戳，这个时间戳称之为是数据的版本

2. 如果不指定，那么HBase表只对外开放一个版本的数据，即最新的版本

3. 如果需要获取多个版本的数据，那么需要在建表的时候指定版本开放的数量

   - 建表：

     ```mysql
     create 'person', {NAME=>'basic', VERSIONS=>3}, {NAME=>'info', VERSIONS=>5}
     ```

   - 查询：

     ```mysql
     get 'person', 'p1', {COLUMN=>'basic:age', VERSIONS=>3}
     ```

     

## namespace - 名称空间

1. `namespace`类似于数据库的`database`，作用是用于区分同名表
2. HBase启动的时候，自带了2个名称空间：`default`和`hbase`
3. 如果建表的时候没有指定namespace，那么默认放在default下





# 基本指令

| 指令                                                         | 解释                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| status                                                       | 查看HBase的运行状态                                          |
| version                                                      | 查看HBase的版本                                              |
| whoami                                                       | 查看当前用户                                                 |
| create  'person', {NAME=>'basic'},{NAME=>'info'},{NAME=>'other'}  <br />或者：  <br />create  'person', 'basic', 'info', 'other' | 建立了一个person表，包含basic、info和other三个列族           |
| put  'person', 'p1', 'basic:name', 'Helen'                   | 向person表中的basic列族的name列添加一个行键为p1的值为Helen的数据 |
| get  'person', 'p1'                                          | 查看person表中行键为p1对应的值                               |
| get  'person', 'p1', {COLUMN=>['basic','info']}  <br />或者：  <br />get  'person', 'p1', 'basic', 'info' | 查看person表中行键为p1对应的basic列族和info列族的值          |
| get  'person', 'p1', {COLUMN=>['basic:name', 'info:height']}  <br />或者：  <br />get  'person', 'p1', 'basic:name', 'info:height' | 查看指定列的值                                               |
| scan  'person'                                               | 查询整个表                                                   |
| scan  'person', {COLUMNS=>['basic', 'info']}                 | 查询person表中basic列族和info列族的值                        |
| scan  'person', {COLUMNS=>['basic:name', 'info:weight']}     | 查询person表中basic列族name列和info列族weight列的值          |
| delete  'person', 'p1', 'other:addr'                         | 删除person表中p1行键对应的other列族addr列的值                |
| deleteall  'person', 'p1'                                    | 删除一行                                                     |
| truncate  'person'                                           | 摧毁重建表                                                   |
| describe  'person'  <br />或者：  <br />desc  'person'       | 描述这个表的结构                                             |
| list                                                         | 查看所有名称空间下的表                                       |
| disable  'person'                                            | 禁用表                                                       |
| drop  'person'                                               | 删除表                                                       |
| create  'person', {NAME=>'basic', VERSIONS=>3}, {NAME=>'info',  VERSIONS=>5} | 建立一个person表包含basic列族和info列族，其中basic列族允许对外提供3个版本的数据，info列族允许对外提供5个版本的数据 |
| get  'person', 'p1', {COLUMN=>'basic:age', VERSIONS=>3}      | 获取basic列族age列最近3个版本的数据                          |
| exists  'student'                                            | 判断一个表是否存在                                           |
| create_namespace  'hbasedemo'                                | 创建名称空间hbasedemo                                        |
| create  'hbasedemo:person', 'basic', 'info'                  | 在hbasedemo下新建了一张person表                              |
| list_namespace_tables  'hbasedemo'                           | 查看指定名称空间下的表                                       |
| drop_namespace  'hbasedemo'                                  | 删除指定的名称空间，要求这个空间下没有表                     |



# 单机安装

> 不依赖于Hadoop的HDFS，配置完即可使用
>
> 好处是便于测试；坏处是不具备分布式存储数据的能力

1. 安装JDK

2. 上传或者下载HBASE的安装包

3. 解压安装包：tar -xvf hbase-0.98.17-hadoop2-bin.tar.gz

4. 进入HBASE安装目录下的子目录conf：cd hbase-0.98.17-hadoop2/conf

5. 修改配置文件hbase-site.xml：vim hbase-site.xml

6. 添加如下配置：

   ```xaml
   <!--指定HBASE的数据存储目录-->
   <property>
       <name>hbase.rootdir</name>
       <value>file:///home/software/hbase/tmp</value>
   </property>
   ```

7. 进入HBASE的安装目录的子目录bin下：cd ../bin

8. 启动服务器端，执行sh start-hbase.sh

   > 启动完成之后可以通过jps命令查看是否有HMaster进程

9. 启动客户端 ./hbase shell





# 伪分布式安装

1. 安装JDK

2. 安装Hadoop的伪分布式或者完全分布式集群

3. 上传或者下载HBASE的安装包

4. 解压安装包：`tar -xvf hbase-0.98.17-hadoop2-bin.tar.gz`

5. 进入HBASE安装目录下的子目录conf：`cd hbase-0.98.17-hadoop2/conf`

6. 修改conf/hbase-env.sh：`vim hbase-env.sh`

7. 添加JAVA_HOME：`export JAVA_HOME=JDK的实际安装路径`

8. 重新生效：`source hbase-env.sh`

9. 修改配置文件hbase-site.xml：`vim hbase-site.xml`

10. 配置使用hdfs:

    ```xml
    <property>
    <name>hbase.rootdir</name>
        <value>hdfs://hadoop01:9000/hbase</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    ```

11. 启动Hadoop。如果是使用的Hadoop完全分布式集群，则还需要启动Zookeeper

12. 进入HBASE的安装目录的子目录bin下：`cd ../bin`

13. 启动服务器端，执行`sh start-hbase.sh`
    启动完成之后可以通过jps命令查看是否有HMaster进程

14. 启动客户端 `./hbase shell`







# ==完全分布式安装==

1. 安装和配置：Hadoop+JDK+ZooKeeper

2. 三台云主机开启ZooKeeper

   ```sh
   cd /home/software/zookeeper-3.4.8/bin
   sh zkServer.sh start	#启动Zookeeper
   sh zkServer.sh status	#查看Zookeeper状态
   ```

   > 开启成功后，3个节点中，有2个follower和1和leader

3. 在第一个节点启动Hadoop

   ```sh
   start-all.sh
   ```

4. 在第一个节点下载HBase安装包并解压

   ```sh
   cd /home/software/
   #下载
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/hbase-1.3.1-bin.tar.gz
   #解压
   tar -xvf hbase-1.3.1-bin.tar.gz
   ```

5. 修改HBase目录下的`/conf/hbase-env.sh`

   ```sh
   #修改JAVA_HOME：
   export JAVA_HOME=xxxx
   #修改Zookeeper和Hbase的协调模式，hbase默认使用自带的zookeeper，如果需要使用外部zookeeper，需要先关闭：
   export HBASE_MANAGES_ZK=false
   ```

   将`export HBASE_MASTER_OPTS`和`export HBASE_REGIONSERVER_OPTS`注释掉

   保存，重新生效：`source hbase-env.sh`

6. 修改HBase目录下的`/conf/hbase-site.xml`

   > 在`<configuration>`标签中添加如下内容

   ```xml
   <property>
       <name>hbase.rootdir</name>
       <value>hdfs://hadoop01:9000/hbase</value>
   </property>
   <property>
       <name>hbase.cluster.distributed</name>
       <value>true</value>
   </property>
   <property>
       <name>hbase.zookeeper.quorum</name>
       <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
   </property>
   ```

7. 编辑`/conf/regionservers`

   > 添加云主机的名字

   ```
   hadoop01
   hadoop02
   hadoop03
   ```

8. 远程拷贝到`Hadoop02`,`Hadoop03`主机上

   ```sh
   cd /home/software
   
   scp -r hbase-1.3.1 root@hadoop02:/home/software/
   scp -r hbase-1.3.1 root@hadoop03:/home/software/
   ```

9. 进入HBase的bin目录，启动HBase

   ```sh
   cd /home/software/hbase-1.3.1/bin
   
   sh start-hbase.sh
   ```

   > 启动后，该节点会出现HMaster、HRegionServer两个进程
   >
   > 其他两个节点都有HRegionServer进程

10. 在HBase的bin目录，启动HBase的客户端

    ```sh
    sh hbase shell
    ```



> 1. 可以通过浏览器访问：http://xxxx:60010访问web界面，管理HBase
> 2. bin目录执行：`stop-hbase.sh`关闭HMaster
> 3. bin目录执行：`sh hbase-daemon.sh stop regionserver`关闭RegionServer



# 配置Maven

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase</artifactId>
        <version>1.3.1</version>
        <type>pom</type>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-common</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-server</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency>
</dependencies>
```



# API操作

## 创建表

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.junit.Test;

import java.io.IOException;

public class HBaseDemo {

    // 创建表
    @Test
    public void createTable() throws IOException {

        // create 'student', 'basic', 'info'
        // 获取HBase的配置
        Configuration conf = HBaseConfiguration.create();
        // 通过Zookeeper连接HBase，需要设置Zookeeper的连接地址
        conf.set("hbase.zookeeper.quorum", "10.42.0.33:2181,10.42.24.13:2181,10.42.104.102:2181");
        // 获取表的管理权
        HBaseAdmin admin = new HBaseAdmin(conf);
        // 构建一个表描述器
        HTableDescriptor table = new HTableDescriptor(TableName.valueOf("student"));
        // 构建列描述器
        HColumnDescriptor cf1 = new HColumnDescriptor("basic");
        HColumnDescriptor cf2 = new HColumnDescriptor("info");
        // 添加列族
        table.addFamily(cf1);
        table.addFamily(cf2);
        // 建表
        admin.createTable(table);
        // 关闭管理权
        admin.close();
    }

}

```

> 执行成功后，在HBase中通过list查看，会发现多了一张`student`表



## 添加/修改数据

```java
// 添加数据/修改数据
@Test
public void putData() throws IOException {

    // put 'student', 'p1', 'basic:name', 'Helen'
    // put 'student', 'p1', 'info:addr', 'beijing'
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置Zookeeper的连接地址
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    // 连接表
    HTable table = new HTable(conf, "student");
    // 封装Put对象
    // 参数表示的是行键
    Put put = new Put("s1".getBytes());
    // family - 列族
    // qualifier - 列
    // value - 数据值
    put.addColumn("basic".getBytes(), "name".getBytes(), "Helen".getBytes());
    put.addColumn("info".getBytes(), "addr".getBytes(), "beijing".getBytes());
    // 添加数据
    table.put(put);
    // 关闭连接
    table.close();
}
```



## 百万条数据写入

```java
// 测试：添加百万条数据
// 花费时间：17536 ~ 17.5s
@Test
public void putMillionData() throws IOException {

    long begin = System.currentTimeMillis();
    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    List<Put> list = new ArrayList<>();
    for (int i = 0; i < 1000000; i++) {
        Put put = new Put(("s" + i).getBytes());
        put.addColumn("basic".getBytes(), "id".getBytes(), ("no" + i).getBytes());
        list.add(put);
        // 每1w条数据，就向HBase表中添加一次
        if (list.size() >= 10000) {
            table.put(list);
            list.clear();
        }
    }
    table.close();
    long end = System.currentTimeMillis();
    System.err.println(end - begin);
}
```



## 获取数据

```java
// 查询数据
@Test
public void getData() throws IOException {

    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    // 封装Get对象
    Get get = new Get("s1".getBytes());
    get.addColumn("basic".getBytes(), "name".getBytes());
    // 查询数据，将查询的结果封装到Result中
    Result r = table.get(get);
    byte[] value = r.getValue("basic".getBytes(), "name".getBytes());
    System.err.println(new String(value));
    table.close();
}
```



## 获取结果集

```java
// 遍历数据
@Test
public void scanData() throws IOException {

    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    // 封装Scan对象
    // Scan scan = new Scan();
    // 从指定行键开始打印到最后
    // Scan scan = new Scan("s9".getBytes());
    // 指定范围
    Scan scan = new Scan("s90".getBytes(), "s91".getBytes());
    // 遍历数据，将结果封装成一个ResultScanner - 结果扫描器返回
    ResultScanner rs = table.getScanner(scan);
    Iterator<Result> it = rs.iterator();
    while (it.hasNext()) {
        Result r = it.next();
        System.err.println(new String(r.getValue("basic".getBytes(), "id".getBytes())));
    }
    table.close();
}
```



## 删除数据

```java
// 删除数据
@Test
public void deleteData() throws IOException {
    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    HTable table = new HTable(conf, "student");
    // 封装成Delete对象
    Delete del = new Delete("s1".getBytes());
    del.addColumn("info".getBytes(), "addr".getBytes());
    // 删除数据
    table.delete(del);
    // 关闭连接
    table.close();
}
```



## 删除表

```java
// 删除表
@Test
public void dropTable() throws IOException {
    Configuration conf = HBaseConfiguration.create();
    conf.set("hbase.zookeeper.quorum", "10.9.162.133:2181,10.9.152.65:2181,10.9.130.83:2181");
    // 获取管理权
    HBaseAdmin admin = new HBaseAdmin(conf);
    // 禁用表
    admin.disableTable("student");
    // 删除表
    admin.deleteTable("student");
    // 关流
    admin.close();
}
```



## 正则过滤器

```java
/*
 * 正则过滤器
 */
@Test
public void scanData() throws Exception {
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置zookeeper的地址
    conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
    // 获取表
    HTable table = new HTable(conf, "student");
    // 获取迭代器
    Scan scan = new Scan();
    // 正则过滤器，匹配行键含3的行数据
    Filter filter = new RowFilter(CompareOp.EQUAL, new RegexStringComparator(".*3.*"));
    // 加入过滤器
    scan.setFilter(filter);
    ResultScanner scanner = table.getScanner(scan);
    // 获取结果的迭代器
    Iterator<Result> it = scanner.iterator();
    while (it.hasNext()) {
        Result result = it.next();
        // 通过result对象获取指定列族的列的数据
        byte[] value = result.getValue("basic".getBytes(), "id".getBytes());
        System.out.println(new String(value));
    }
    scanner.close();
    table.close();
}
```



## 行键过滤器

```java
@Test
public void rowKeyCompare() throws Exception {
	// 获取配置
	Configuration conf = HBaseConfiguration.create();
	// 设置zookeeper的地址
	conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
	// 获取表
	HTable table = new HTable(conf, "student");
	Scan scan = new Scan();
	// 行键比较过滤器，下例是匹配小于或等于指定行键的行数据
	Filter filter = new RowFilter(CompareOp.LESS_OR_EQUAL, new BinaryComparator("100".getBytes()));
	scan.setFilter(filter);
	ResultScanner scanner = table.getScanner(scan);
	Iterator<Result> it = scanner.iterator();
	while (it.hasNext()) {
		Result result = it.next();
		byte[] value = result.getValue("basic".getBytes(), "id".getBytes());
		System.out.println(new String(value));
	}
	scanner.close();
	table.close();
}
```



## 行键前缀过滤器

```java
@Test
public void prefix() throws Exception {
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置zookeeper的地址
    conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
    // 获取表
    HTable table = new HTable(conf, "student");
    Scan scan = new Scan();
    // 行键前缀过滤器
    Filter filter = new PrefixFilter("3".getBytes());
    scan.setFilter(filter);
    ResultScanner scanner = table.getScanner(scan);
    Iterator<Result> it = scanner.iterator();
    while (it.hasNext()) {
        Result result = it.next();
        byte[] value = result.getValue("basic".getBytes(), "id".getBytes());
        System.out.println(new String(value));
    }
    scanner.close();
    table.close();
}
```



## 列值过滤器

```java
@Test
public void colScan() throws Exception {
    // 获取配置
    Configuration conf = HBaseConfiguration.create();
    // 设置zookeeper的地址
    conf.set("hbase.zookeeper.quorum", "192.168.232.129:2181,192.168.232.130:2181,192.168.232.131:2181");
    // 获取表
    HTable table = new HTable(conf, "student");
    Scan scan = new Scan();
    // --列值过滤器
    Filter filter = new SingleColumnValueFilter("basic".getBytes(), "name".getBytes(), CompareOp.EQUAL,
                                                "rose".getBytes());
    scan.setFilter(filter);
    ResultScanner scanner = table.getScanner(scan);
    // --获取结果的迭代器
    Iterator<Result> it = scanner.iterator();
    while (it.hasNext()) {
        Result result = it.next();
        byte[] name = result.getValue("basic".getBytes(), "name".getBytes());
        byte[] age = result.getValue("basic".getBytes(), "age".getBytes());
        System.out.println(new String(name) + ":" + new String(age));
    }
    scanner.close();
    table.close();
}
```



# 物理存储原理

## 概述

1. HBase里的一个Table在行的方向上分割为一个或者多个HRegion，即HBase中一个表的数据会被划分成一个或者很多的HRegion，每一个HRegion会被存储在一个节点上。这样做的目的是为了能做到数据的分布式存储

2. HRegion可以动态扩展，并且HBase保证HRegion的负载均衡

3. HRegion实际上是行键排序后，按规则分割的连续的存储空间

   ![](https://note.youdao.com/yws/api/personal/file/88266D0247D74299843CA4C4ABE6F7A0?method=download&shareKey=a23bb2160aae712653da06e8912b3a82)

4. 一张HBase表，可能有多个HRegion，每个HRegion达到一定大小时，进行分裂

   > ==大小默认为：10G==

   ![](https://note.youdao.com/yws/api/personal/file/5E224C3D0C7148A5954E27C4603E6ACC?method=download&shareKey=b9955354fea5c226961ffd8076803051)



## 拆分过程

1. HRegion是按照大小分割的，每个表一开始只有一个HRegion，随着数据不断插入表，HRegion不断增大，当增大到一个阈值时，HRegion就会等分出两个等大的HRegion。因此随着Table中的行不断增多，就会有越来越多的HRegion

2. 按照现在主流硬件的配置，每个HRegion的大小可以是1~20GB，这个大小由`hbase.hregion.max.filesize`指定，默认为10GB

3. HRegion的拆分和转移是由HBase的`HMaster`进程自动完成的，用户感知不到

4. **HRegion是HBase中分布式存储和负载均衡的最小单元**

   ![](https://note.youdao.com/yws/api/personal/file/01137F1311254AD390BF27A0BFAFFF47?method=download&shareKey=a985bb14267580375a22ad1927bca16c)

5. **HRegion虽然是分布式存储的最小单元，但并不是存储的最小单元**

6. HRegion由一个或者多个`HStore`组成，每个`HStore`保存一个列族（columns family）

7. 每个`HStore`又由【一个`memStore`】以及【0个或者多个`StoreFile`】组成，`StoreFile`以`HFile`格式保存在HDFS上

   > `memStore`：写缓存，默认大小128MB

8. **总结：HRegion是分布式存储的最小单位，StoreFile(HFile)是存储的最小单位**

   ![](https://note.youdao.com/yws/api/personal/file/0788B6BA03B8438E9149D7E89488BF2F?method=download&shareKey=2c6caeb30dffe941ccf8a0292a41f0c5)



# 系统架构

## HBase基于Hadoop架构图

![](https://note.youdao.com/yws/api/personal/file/389EC4F3FC6F4C74B0A5DF8A3AE6F035?method=download&shareKey=f5afe693bf4072f5ff9fecd668fbc8e9)



## HBase架构组成

> HBase采用Master/Slave（主从）架构搭建集群，它隶属于Hadoop生态系统，由一下类型节点组成

1. HMaster节点：主节点：Active；从节点：Backup

   - **管理HRegionServer，实现其负载均衡**。保证每一个HRegionServer上的数据量差别不大
   - **管理和分配HRegion**。比如在HRegion分裂时分配新的HRegion；在HRegionServer退出时迁移其内的HRegion到其他HRegionServer上
   - **实现DDL操作**。（Data Definition Language），namespace和Table的增删改，column family的增删改等
   - **管理namespace和table的元数据**。（实际存储在HDFS上）
   - **权限控制（ACL）**

2. HRegionServer节点

   - 存放和管理本地HRegion

   - 读写HDFS，管理Table中的数据

   - Client直接通过HRegionServer读写数据

     > 从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后读取数据

3. ZooKeeper集群

   - 存放整个HBase集群的元数据、集群的状态信息以及RS服务器的运行状态
   - 实现HMaster主备节点的failover



> HBase的数据存储于HDFS中，==RegionServer和DataNode一般会放在相同的Server上==**实现数据的本地化(避免或减少数据在网络中的传输，节省带宽)**



![](https://note.youdao.com/yws/api/personal/file/259A3636982F4D54BD240C54A870F648?method=download&shareKey=e07f961613b74125ff9e04b634c839c4)



![](https://note.youdao.com/yws/api/personal/file/D0C4BC8711B54E41AC0A29FA61CE0A3F?method=download&shareKey=10fe951b02d5fce0521dda086dd2bd15)





> HBase Client通过RPC方式和HMaster、HRegionServer通信；
>
> 一个HRegionServer可以存放1000个HRegion（1000个数字的由来是来自于Google的Bigtable论文）；
>
> 底层Table数据存储于HDFS中，而HRegion所处理的数据尽量和数据所在的DataNode在一起，实现数据的本地化；**数据本地化并不是总能实现，比如在HRegion移动(如因Split)时，需要等下一次Compact才能继续回到本地化**



# 架构原理详解

## HRegion

![](https://note.youdao.com/yws/api/personal/file/DE224EB7D51B4FBA97DFE6745A6AEF95?method=download&shareKey=30c66066818e92edb8ed2e9a27be1936)



1. HBase使用RowKey将表水平切割成多个HRegion
2. 从HMaster的角度看，每个HRegion都记录了它的StartKey和EndKey
3. ==由于RowKey是排序的，因而Client可以通过HMaster快速的定位每个RowKey在哪个HRegion==
4. HRegion由HMaster分配到响应的HRegionServer中，然后由HRegionServer负责HRegion的启动、管理、与Client的通信以及负责数据的读取（使用HDFS）
5. 每个HRegionServer可以同时管理1000个左右的HRegion



## HMaster

![](https://note.youdao.com/yws/api/personal/file/1B381D5EF2F94ABFA41DBD950E9F9E12?method=download&shareKey=bcae5ded868f6b0b03c8be007bfa0d1a)



1. HMaster没有单点故障问题，因为HBase集群可以启动多个HMaster

2. 通过ZooKeeper的Master Election机制保证同时只有一个HMaster处于Active状态，其他的HMaster则处于热备份状态

3. 一般情况下会启动两个HMaster，BackUp的HMaster会定期的和Active HMaster通信以获取其最新状态，从而保证它是实时更新的，因而如果启动了多个HMaster反而增加了Active HMaster的负担

4. HMaster主要用于HRegion的分配和管理，DDL的实现等，既它主要有两方面的职责：

   > DDL：Data Definition Language，既Table的新建、删除、修改等

   1. 协调HRegionServer：

      - 启动时HRegion的分配，以及负载均衡和修复时HRegion的重新分配
      - 监控集群中所有HRegionServer的状态(通过Heartbeat和监听ZooKeeper中的状态)

   2. Admin职能

      - 管理创建、删除、修改Table的操作



## ZooKeeper - 协调者

![](https://note.youdao.com/yws/api/personal/file/3008CFB308BF46F3BB235B7277DF7554?method=download&shareKey=30c79b6dc7cc5b7fa99ee4474eb567d2)



1. ZooKeeper为HBase集群提供协调服务，它管理着HMaster和HRegionServer的状态(available/alive等)

2. ZooKeeper协调集群所有节点的共享信息，在HMaster和HRegionServer连接到ZooKeeper后创建Ephemeral( 临时）节点，并使用Heartbeat机制维持这个节点的存活状态，如果某个Ephemeral节点实效，则HMaster会收到通知，并做相应的处理

3. 当HMaster宕机的时候实现HMaster之间的failover(失败恢复）

4. 在HRegionServer宕机时通知给HMaster，从而对宕机的HRegionServer中的HRegion集合的修复(将它们分配给其他的HRegionServer)

   ![](https://note.youdao.com/yws/api/personal/file/D1B97AEE263A455EBBD0CDCAA5F22638?method=download&shareKey=e13aa3194d913f720c0617d7047bacaa)

5. 另外，HMaster通过监听ZooKeeper中的Ephemeral节点(默认：/hbase/rs/*)来监控HRegionServer的加入和宕机。

   - 在第一个HMaster连接到ZooKeeper时会创建Ephemeral节点(默认：/hbasae/master)来表示Active的HMaster
   - 其后加进来的HMaster则监听该Ephemeral节点，如果当前Active的HMaster宕机，则该节点消失，因而其他HMaster得到通知，而将自身转换成Active的HMaster，在变为Active的HMaster之前，它会创建在/hbase/back-masters/下创建自己的Ephemeral节点。



## HRegionServer

![](https://note.youdao.com/yws/api/personal/file/ED96BAD7AEDC49308581992AA3BA05A1?method=download&shareKey=a650197820cac342ef80b14a4f15bcc9)

> 参考图片2：

![](https://note.youdao.com/yws/api/personal/file/90BBE0BCD9BE47B7A9484B3A21BBBF18?method=download&shareKey=683bfcee06550ebe7988ce551ac78852)



1. HRegionServer一般和DataNode在同一台机器上运行，**实现数据的本地性**

2. HRegionServer存储和管理多个HRegion，由WAL(HLog)、BlockCache、Region组成：

   1. **WAL即Write Ahead Log**

      - 作用：记录写操作，保证数据的可靠性（不丢失）
      - 当HRegionSever接收到写请求之后，会先将这个写请求记录到WAL中，如果记录成功，则将数据更新到对应HRegion中HStore中的memStore中
      - 在HBase0.94版本之前，WAL只允许串行写；从HBase0.94版本开始，引入了NIO的Channel机制，允许对WAL进行并行写
      - 当WAL达到一定条件的时候，会被自动清理
   2. **BlockCache是一个读缓存**
      - 读缓存，维系在内存中，默认是128M
      - 采用了"局部性"原理 - 本质上就是一个"猜"的过程，提高"猜"的命中率：
        - **时间局部性**：在HBase中，如果一条数据被读取过，那么会认为这条数据再次被读取的概率要高于其他没有被读取过的数据，那么会将这条数据放到读缓存中
        - **空间局部性**：在HBase中，如果一条数据被读取过，那么会认为与这条数据相邻的数据被读取的概率要高于其他不相邻的数据，那么会将这条数据相邻的数据放到读缓存中
      - 采用了LRU策略：抛弃最长时间不用的数据
   3. **HRegion是一个Table中的一个Region在一个HRegionServer中的表达**
      1. 一个HRegion包含1到多个HStore，每一个HStore对应一个列族
      2. 一个HStore中包含1个memStore以及0到多个HFile/StoreFile
      3. memStore：写缓存，将数据记录到内存中，默认大小是128M，当memStore达到一定条件的时候，冲刷出一个HFile
      4. HFile：0到多个。HFile落地到HDFS上 -> HBase中的数据最终写到HFile中，HFile存储在DataNode上
      5. memStore的冲刷条件：
         - 当memStore用满之后，会自动冲刷出一个HFil
         - 当WAL的大小达到1G的时候，会冲刷这个HRegionServer上所有的memStore，同时WAL会滚动产生一个新的WAL，原来的WAL就会变成OLD_WAL，OLD_WAL会被清理掉
         - 当所有的memStore所占用的内存之和/总内存>0.35，那么会自动冲刷几个最大的memStore - 这种方案会产生大量的小文件

3. 因为Hbase的HFile是存到HDFS上，所以Hbase实际上是具备数据的副本冗余机制的



## HBase的第一次读写

> 在HBase 0.96以前，HBase有两个特殊的Table：`-ROOT-`和`.meta.`

![](https://note.youdao.com/yws/api/personal/file/25C31B7608974144865BAD3FF60F704A?method=download&shareKey=c433b4d6a3ea777463e92ba83e3075c0)

1. `-ROOT-` 表的存储位置存储在ZooKeeper，它存储了`.META.`表的RegionInfo信息
2. `.META.` 表则存储了用户定义的Table的RegionInfo信息
3. 第一次访问Table时
   - 首先从ZooKeeper中读取`-ROOT-`表所在`HRegionServer`；
   - 然后从该HRegionServer中根据请求的TableName，RowKey读取`.META.`表所在HRegionServer；
   - 最后从该HRegionServer中读取`.META.`表的内容而获取此次请求需要访问的HRegion所在的位置，然后访问该HRegionSever获取请求的数据，这需要三次请求才能找到用户Table所在的位置，**然后第四次请求开始获取真正的数据**。
   - 当然为了提升性能，客户端会缓存`-ROOT- Table`位置以及`-ROOT-/.META. Table`的内容

![](https://note.youdao.com/yws/api/personal/file/E1583765D65741BEAA405909AF490EA2?method=download&shareKey=c0c026525ac00d4d9ffa51d05b07c74c)





> 在HBase 0.96及之后，去掉了`-ROOT- table`

![](https://note.youdao.com/yws/api/personal/file/67A711B3E22D4C26B4936FBAF53FF2FD?method=download&shareKey=65dceac422886f180479636514db5362)

1. 只剩下这个特殊的目录表叫做**Meta Table(**hbase:meta)，它存储了集群中所有用户HRegion的位置信息，而ZooKeeper的节点中(`/hbase/meta-region-server`)存储的则直接是这个`Meta Table`的位置，并且这个`Meta Table`如以前的`-ROOT- Table`一样是不可split的。**这样，客户端在第一次访问用户Table的流程是：**

   1. 从ZooKeeper(`/hbase/meta-region-server`)中获取`hbase:meta`的位置（HRegionServer的位置），缓存该位置信息
   2. 从HRegionServer中查询用户Table对应请求的RowKey所在的HRegionServer，**缓存该位置信息**
   3. 从查询到HRegionServer中读取Row

2. 从这个过程中，客户端会缓存这些位置信息，然而第二步它只是缓存当前RowKey对应的HRegion的位置，因而如果下一个要查的RowKey不在同一个HRegion中，则需要继续查询`hbase:meta`所在的HRegion，**然而随着时间的推移，客户端缓存的位置信息越来越多，以至于不需要再次查找`hbase:meta Table`的信息**

3. **当某个HRegion因为宕机或Split被移动，此时需要重新查询并且更新缓存。**

   ![](https://note.youdao.com/yws/api/personal/file/9BDAC7916F8244F6ADE533D93F3118D2?method=download&shareKey=69ecdc9e9a5320bd7f6563db5e04e6fb)

# 写流程

1. 当HRegionServer收到写请求的时候，会默认先将这个写操作记录到WAL中，然后会将数据更新到memStore中

2. 数据在memStore中会进行排序：行键字典升序-> 列族名字典升序 -> 列名字典升序 -> 时间戳倒叙

3. 当memStore达到一定条件之后，会自动进行冲刷，冲刷出HFile

   > 因为数据在memStore中已经排序，所以HFile是局部有序整体无序的，各个HFile把持其实行键和结束行键，最后落地到HDFS上

   ![](https://note.youdao.com/yws/api/personal/file/635C4EE02FCE48779ACAEA041A704C18?method=download&shareKey=336079847c21a94a97e847c108effee5)

4. HFile的格式V1：

   ![](https://note.youdao.com/yws/api/personal/file/E146B74C114D41D49AB601816CEF3B16?method=download&shareKey=3e40c371c7f396a7f4027735e2e2031c)

   - DataBlock：存储数据

     - DataBlock是HBase中数据存储的最小单位

     - DataBlock默认大小是64KB。小的DataBlock利于查询(get)，大的DataBlock利于遍历(scan)

     - BlockCache的空间局部性是以DataBlock为单位存储：只要DataBlock中有一条数据被读取，那么整个DataBlock都会放到BlockCache中

     - 每一个DataBlock都包含1个Magic和多个KeyValue

       > **Magice**：魔术，本质上是一个随机数。**作用**：对数据进行校验，只要改变过数据，这个随机数就会发生变化
       >
       > **KeyValue**：用于存储数据
       >
       > ![](https://note.youdao.com/yws/api/personal/file/25041F414F0344478E39BE19F38F65BE?method=download&shareKey=b71bfddea308aeb0f5e1c7a903958794)

   - MetaBlock：存储元数据，绝大部分HFile不包含这一部分，只有`.meta.`文件中会包含

   - FileInfo：对文件信息的描述

   - DataIndex：记录DataIBlock在文件中的存储位置

   - MetaIndex：记录MetaIBlock在文件中的存储位置

   - Trailer：在文件末尾，固定4个字节大小，其中前2个字节记录DataIndex的起始位置，后2个字节记录MetaIndex的起始位置

5. HFile在第二个版本中新添了==布隆过滤器==

   ![](https://note.youdao.com/yws/api/personal/file/B5DB4F235A104C9D890922789C6E0D44?method=download&shareKey=e28faca694438c832fe77b81403eb3fa)



# 读流程

1. 当HRegionServer接收到读请求之后，会先从BlockCache中读取
   - 如果BlockCache读不到，则会从memStore中读取
   - 如果memStore也都不到，需要从HFile中读取
2. 会现根据HFile的行键范围进行筛选，筛选完成之后会再利用布隆过滤器来进行筛选，被筛掉的文件中一定是没有要查询的数据的，但是剩下的文件中不能保证一定有要查询的数据



# Compaction机制

1. HBase提供了2种Compaction机制

   - minor compact

     > 将相邻的几个小的HFile合并成一个大的HFile，如果HFile本身很大就不合并

   - major compact

     > 将所有的HFile，不管大小，合并成一个HFile。合并完成之后存在一个HFile

2. 在HBase中，如果不指定，默认使用`minor compact`

   > 实际开发中，一般会在周末的凌晨进行`major compact`
   >
   > `major compact`合并一般是几个小时

3. 在合并过程中，会舍弃被标记删除的数据或者过时的数据

   > 例如一个数据指定保留3个版本的数据，但是在合并过程中发现5个版本，就会舍弃最早的2个版本的数据

