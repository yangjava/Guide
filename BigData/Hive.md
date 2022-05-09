# 概述

1. Hive是Apache提供的一套基于Hadoop的**数据仓库**的管理工具。提供了读写以及管理数据的功能
2. 提供了**类SQL**语言来操作Hadoop，在底层将类SQL转换为MapReduce来执行 ，所以Hive的执行效率相对比较低，适合于**离线批处理**
3. 在安装Hive之前，需要先安装Hadoop
4. 在Hive中，每一个database/table在HDFS上对应一个目录
5. Hive中没有主键的概念
6. Hive在建表的时候，需要指定字段之间的间隔字符。一张表一旦建好，间隔字符就不能改变
7. into表示向表中追加数据，overwrite表示将表中的数据覆盖掉



# 安装

1. 安装JDK
2. 安装Hadoop
3. 配置JDK和Hadoop的环境变量
4. 下载Hive安装包
5. 解压Hive
6. 启动Hadoop的HDFS和Yarn
7. 进入Hive的bin目录，通过`sh hive`启动Hive

[TOC]



# 基础指令整理

| 命令                                                         | 作用                                         | 额外说明                                                     |
| ------------------------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------ |
| show databases;                                              | 查看都有哪些数据库                           |                                                              |
| create database park;                                        | 创建park数据库                               | 创建的数据库，实际是在Hadoop的HDFS文件系统里创建一个目录节点，统一存在： /user/hive/warehouse 目录下 |
| use park;                                                    | 进入park数据库                               |                                                              |
| show tables;                                                 | 查看当前数据库下所有表                       |                                                              |
| create table stu   (id int,name string);                     | 创建stu表，以及相关的两个字段                | hive里，表示字符串用的是string，不用char和varchar    所创建的表，也是HDFS里的一个目录节点 |
| insert into stu values(1,'zhang')                            | 向stu表插入数据                              | HDFS不支持数据的修改和删除，因此已经插入的数据不能够再进行任何的改动    在Hadoop2.0版本后支持了数据追加。实际上，insert into 语句执行的是追加操作    hive支持查询，行级别的插入。不支持行级别的删除和修改    hive的操作实际是执行一个job任务，调用的是Hadoop的MR    插入完数据之后，发现HDFS stu目录节点下多了一个文件，文件里存了插入的数据，因此，hive存储的数据，是通过HDFS的文件来存储的。 |
| select * from stu                                            | 查看表数据                                   | 也可以根据字段来查询，比如select id from stu                 |
| drop table stu                                               | 删除表                                       |                                                              |
| select * from stu                                            | 查询stu表数据                                |                                                              |
| load data local inpath '/home/software/1.txt' into table stu; | 通过加载文件数据到指定的表里                 | 在执行完这个指令之后，发现hdfs stu目录下多了一个1.txt文件。由此可见，hive的工作原理实际上就是在管理hdfs上的文件，把文件里数据抽象成二维表结构，然后提供hql语句供程序员查询文件数据    可以做这样的实验：不通过load 指令，而通过插件向stu目录下再上传一个文件，看下hive是否能将数据管理到stu表里。 |
| create table stu1(id int,name string) row format delimited fields  terminated by '   '; | 创建stu1表，并指定分割符 空格。              |                                                              |
| desc stu                                                     | 查看 stu表结构                               |                                                              |
| create table stu2 like stu                                   | 创建一张stu2表，表结构和stu表结构相同        | like只复制表结构，不复制数据                                 |
| insert overwrite table stu2   select * from stu              | 把stu表数据插入到stu2表中                    |                                                              |
| insert  overwrite local directory '/home/stu' row format delimited fields terminated by ' ' select * from  stu; | 将stu表中查询的数据写到本地的/home/stu目录下 |                                                              |
| insert  overwrite directory '/stu' row format delimited fields terminated by '   ' select  * from stu; | 将stu表中查询的数据写到HDFS的stu目录下       |                                                              |
| from stu insert overwrite table stu1 select * insert overwrite table stu2 select *; | 将stu表中查询的数据写到stu1以及stu2两张表中  |                                                              |
| alter table stu rename to  stu2                              | 为表stu重命名为stu2                          |                                                              |
| alter table stu  add columns (age int);                      | 为表stu增加一个列字段age，类型为int          |                                                              |
| exit                                                         | 退出hive                                     |                                                              |



# 基本命令

1. 将`/home/hivedata/person.txt`文件加载到Hive的person表中

   ```mysql
   load data local inpath '/home/hivedata/person.txt' into table person;
   ```

2. 建表的时候指定字段之间的间隔字符是：**空格**

   ```mysql
   create table person (id int,name string,age int) row format delimited fields terminated by ' ';
   ```

3. 建立一张和person结构一样的表p2

   ```mysql
   create table p2 like person
   ```

4. 将person表中的`age>=14`的数据查询出来放到p2中

   ```mysql
   insert into table p2 select * from person where age>=14;
   ```

5. 将person表中`age>=14`的数据查询出来放到p2中，同时将`id<3`的数据查询出来放到p3表中

   ```mysql
   from person insert into table p2 select * where age>=14
   	insert into table p3 select * where id<3
   ```

6. 将person表中的数据查询出来写到本地目录中，并且在本地目录中，字段之间需要用`,`作为间隔

   ```mysql
   insert overwrite local directory '/home/hivedata' row format delimited fields terminated by ','
   	select * from person where id>=3
   ```

7. 将person表中的数据查询出来放到HDFS的指定目录下

   ```mysql
   insert overwrite directory '/person' row format delimited fields terminated by ' '
   	select * form person;
   ```

8. 将person表改名为p1

   ```mysql
   alter table person rename to p1;
   ```

9. 向p1表中添加一个字段

   ```mysql
   alter table p1 add columns (gender string);
   ```

   



# 常见表结构

## 内部表和外部表

1. 内部表：

   - 在Hive中手动创建，手动添加数据的表
   - 再删除内部表的时候，在HDFS上对应的目录也会被删除

2. 外部表：

   - 在Hive中建立的管理HDFS中已经存在的数据的表

   - 在删除外部表的时候，在HDFS上对应的目录不会被删除

   > 外部表的建表语句：关键字：`external`

   ```mysql
   create external table 
   	score (name string, chinese int, math int, english int) 
   	row format delimited fields terminated by ' ' 
   	location '/score'; 
   ```

   

## 分区表

> 分区表的作用是对数据进行分类

1. 建表语句

   ```mysql
   create table cities(id int, name string) 
       partitioned by(province string) 
       row format delimited fields terminated by ' '; 
   ```

2. 加载数据

   ```mysql
   load data local inpath '/home/hivedata/hebei.txt' into table cities partition(province='hebei');
   load data local inpath '/home/hivedata/jiangsu.txt' into table cities partition (province= 'jiangsu'); 
   ```

3. ==每一个分区在HDFS上对应一个目录==

   ![](https://note.youdao.com/yws/api/personal/file/BA69020BF27641569084AF3181221740?method=download&shareKey=12f195dc36ae7d44bac781b290e12f13)

4. 在建立分区表的时候，分区字段在要求在原始数据中不能存在，而是在加载数据的时候手动指定

5. 如果手动在HDFS上新建了一个目录作为分区，那么需要在Hive中添加这个分区

   ```mysql
   alter table cities add partition (province='shandong') 
   	location '/user/hive/warehouse/hivedemo.db/cities/province=shandong'; 
   ```

   或者利用下述命令来修复分区。但是这个命令不稳定，有时候会修复失败

   ```mysql
   msck repair table cities; 
   ```

6. 删除分区

   ```mssql
   alter table cities drop partition(province='guangdong'); 
   ```

7. 修改分区名

   ```mysql
   alter table cities partition(province='jiangxi') rename to partition(province='jiangsu'); 
   ```

8. 在分区表中

   - 如果按照分区字段进行查询，会大幅度的提高查询效率，因为此时只需要查询对应目录即可，而不需要将整个表全部查询
   - 如果查询的时候进行了跨分区的查询，此时查询效率反而会变低，因为需要读取多个文件

9. 动态分区：将数据从未分区的表中向已分区的表中存放

   - 建表管理源数据

     ```mysql
     create table c_tmp(cid int, cname string, cpro string)
     	row format delimited fields terminated by ' '; 
     ```

   - 加载数据

     ```mysql
     load data local inpath '/home/hivedata/city.txt' into table c_tmp; 
     ```

   - 开启动态分区，动态分区默认是严格模式，不开启

     ```mysql
     set hive.exec.dynamic.partition.mode=nonstrict; 
     ```

   - 动态分区

     ```mysql
     insert into table cities partition(province) 
     	select cid, cname, cpro from c_tmp distribute by(cpro); 
     ```

10. 在Hive中，分区字段可以有多个，往往是用于对数据进行==多级分区==的。写在前面的分区表示大类，包含后面的分区

    ```mysql
    create table product(id int, name string) 
    	partitioned by(kind string, subkind string, grandkind string) 
    	row format delimited fields terminated by ' ';
    
    load data local inpath '/home/hivedata/shoes.txt'into table product 
    	partition (kind='clothes',subkind='shoes',grandkind='male');  
    ```

    ![](https://note.youdao.com/yws/api/personal/file/DD273DA9E09D4D54AFCE87E1CDC4A213?method=download&shareKey=02322682ed42d64abc1dd6f43fa91f50)



## 分桶表

1. 作用：对数据进行抽样
2. 分桶表的特点在于：不支持load加载方式，而是需要从其他表中查询然后放到分桶表中

> 案例

- 建表：表示根据name字段来进行分桶，将数据分到6个桶中

  > 先计算name字段值的哈希吗，然后利用这个哈希吗对桶进行取余，取余完成之后决定放到哪个桶中

  ```mysql
  create table score_sample(name string, chinese int, math int, english int) 
      clustered by (name) into 6 buckets 
      row format delimited fields terminated by ' '; 
  ```

- 开启分桶机制

  ```mysql
  set hive.enforce.bucketing = true; 
  ```

- 创建外部表

  ```mysql
  create external table score(name string, chinese int, math int, english int) 
  	row format delimited fields terminated by ' ' location '/score';
  ```

- 将数据从`score`表中查询出来放到`score_sample`表中，这个过程会自动进行分桶

  ```mysql
  insert overwrite table score_sample select * from score;  
  ```

  > 分桶表文件名从0开始

  ![](https://note.youdao.com/yws/api/personal/file/BC8566364638418EA8FE7172047FDAF2?method=download&shareKey=ef2057d02d7339611b87dc21354975d3)

- 抽样：`bucket x out of y`---x表示起始桶编号（图片的000001_0，编号是2），y表示抽样步长

  > `bucket 1 out of 3`表示从第一个桶（对应文件名为0）中抽取，每隔3个桶抽一次

  ```mysql
  select * from score_sample tablesample (bucket 1 out of 3 on name); 
  ```

  





# SerDe机制

1. SerDe（Serializar/Deserializar）是Hive提供的一套序列化/反序列化机制
2. 实际开发中，SerDe机制专门用于处理不规则的数据

> 案例：把Tomcat日志整理到表中

1. 日志信息

   ```
   192.168.120.23 -- [30/Apr/2018:20:25:32 +0800] "GET /asf.avi HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:32 +0800] "GET /bupper.png HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:32 +0800] "GET /bupper.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /bg-button HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /bbutton.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /asf.jpg HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /tomcat.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /tomcat.png HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /tbutton.png HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /tinput.png HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:33 +0800] "GET /tbg.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /tomcat.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /bg.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /bg-button.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /bg-input.css HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /bd-input.png HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /bg-input.png HTTP/1.1" 304 -
   192.168.120.23 -- [30/Apr/2018:20:25:34 +0800] "GET /music.mp3 HTTP/1.1" 304 -
   ```

   

2. 建表

   ```mysql
   create table logs(ip string, time string, timezone string, request_way string, resource string, protocols string, statusid int) 
       row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe' 
       with serdeproperties ("input.regex" = "(.*) \-\- \\[(.*) (.*)\\] \"(.*) (.*) (.*)\" ([0-9]+) \-") 
       stored as textfile; 
   ```

   

3. 加载数据

   ```mysql
   load data local inpath '/home/hivedata/logs.txt' into table logs; 
   ```

   



# join

1. 在Hive中，提供了inner/left/right/full outer/left semi join 查询

2. 如果join的时候，没有明确指定形式，那么默认就是inner join

3. `products p left semi join orders o`表示查询products表中有哪些数据在orders表中出现过

   

> 案例

1. 建立外部表orders

   ```mysql
   create external table orders(orderid int, orderdate string, proid int, num int) 
   	row format delimited fields terminated by ' ' location '/order';
   ```

2. 建立外部表products

   ```mysql
   create external table products (proid int, name string, price double) 
   	row format delimited fields terminated by ' ' location '/product';
   ```

3. 关联查询

   ```mysql
   select * from products p left semi join orders o on p.proid = o.proid;
   ```

   





# beeline

1. beeline是Hive提供的一种用于远程连接Hive的方式，其提供了表格形式(类似MySQL)来进行展现

2. 如果使用这种方式，需要先后台启动`hiveserver2`

   `sh hive --service hiveserver2 &`

3. 远程连接：`beeline -u jdbc:hive2://hadoop01:10000/hivedemo`

   - 注意：此种方式的情况下，默认是不对用户名和密码进行验证的
   - 指定验证用户：`beeline -u jdbc:hive2://hadoop01:10000/hivedemo -n root`

   





# 视图

1. 在一个表中包含了很多字段，但是每一个字段的使用频率并不相同，此时可以考虑将使用频率高的字段抽取出来形成一个子表，又希望子表能与主表自动关联，此时这个子表可以以视图的形式来体现
2. **虚拟视图**：抽取出来的视图放到了内存中
3. **物化视图**：抽取出来的视图放到了磁盘中
4. Hive中只支持虚拟视图，不支持物化视图
5. **视图封装的select语句在封装时并没有执行，在第一次使用视图的时候才会触发select**

> 案例

- 建立视图

  ```mysql
  create view o_view as select orderid, num from orders;
  ```

- 查询语句

  ```mysql
  select orderid from o_view;
  ```

- 删除视图

  ```mysql
  drop view o_view;
  ```

  





# 索引

1. 在数据库中，因为在建表的时候存在主键，所以会自动根据主键建立索引。在Hive中，没有主键的概念，所以也不会自动建立索引
2. 在Hive中可以针对任何一个字段来建立索引，而且一张表可以存放多个索引

> 案例

- 建立索引表

  ```mysql
  create index oin on table orders(orderid) 
      as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'  
      with deferred rebuild in table o_index;
  ```

- 手动建立索引

  ```mysql
  alter index oin on orders rebuild;
  ```

- 删除索引

  ```mysql
  drop index oin on orders;
  ```


[TOC]



# 数据类型

## 基本类型

| Hive中的类型 | Java中的类型 |
| ------------ | ------------ |
| tinyint      | byte         |
| smallint     | short        |
| int          | int          |
| bigint       | long         |
| boolean      | boolean      |
| float        | float        |
| double       | double       |
| string       | String       |
| timestamp    | TimeStamp    |
| binary       | byte[]       |



## 复杂类型

### array类型

> 数组类型，对应了Java中的数组或者集合

1. 原始数据

   ```
   2,4,6,2,5  3,2,4,5
   32,48,23,52  54,23,12,57,95
   122,23,53,4,4,23  64,23,86,4,84,4
   27,48 90,12,44
   ```

2. 建表

   ```mysql
   create table nums(na1 array<int>, na2 array<int>) 
   	row format delimited fields terminated by ' ' 
   	collection items terminated by ','; 
   ```

3. 加载数据

   ```mysql
   load data local inpath '/home/hivedata/nums.txt' into table nums; 
   ```

4. 查询非空数据

   ```mysql
   select na1[5] from nums where na1[5] is not null; 
   ```

   

### map类型

> 映射类型，对应了Java中的Map类型

1. 原始数据

   ```
   1 Amy:female
   2 Sam:male
   3 Alex:male
   4 Lily:female
   5 Jack:male
   ```

2. 建表语句

   ```mysql
   create table infos (id int, info map<string,string>) 
       row format delimited fields terminated by ' ' 
       map keys terminated by ':'; 
   ```

3. 加载数据

   ```mysql
   load data local inpath '/home/hivedata/info.txt' into table infos; 
   ```

4. 查询非空值

   ```mysql
   select info['Sam'] from infos where info['Sam'] is not null; 
   ```

   



### struct类型

> 结构体类型，对应了Java中的对象

1. 建表

   ```mysql
   create external table orders(o struct<orderId:int, orderDate:string, productId:int, num:int>) 
       row format delimited collection items terminated by ' ' 
       location '/order'; 
   ```

2. 查询语句

   ```mysql
   select o.orderid from orders; 
   ```

   



# 运算符

## 关系运算符

| 运算符         | 类型         | 说明                                                         |
| -------------- | ------------ | ------------------------------------------------------------ |
| A = B          | 所有原始类型 | 如果A与B相等，返回true，否则返回false                        |
| A ==  B        | 无           | 失败，因为无效的语法。  SQL使用”=”，不使用”==”。             |
| A  <> B        | 所有原始类型 | 如果A不等于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL”。 |
| A  < B         | 所有原始类型 | 如果A小于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL”。 |
| A  <= B        | 所有原始类型 | 如果A小于等于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL”。 |
| A  > B         | 所有原始类型 | 如果A大于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL”。 |
| A  >= B        | 所有原始类型 | 如果A大于等于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL”。 |
| A IS  NULL     | 所有类型     | 如果A值为”NULL”，返回TRUE,否则返回FALSE                      |
| A IS  NOT NULL | 所有类型     | 如果A值不为”NULL”，返回TRUE,否则返回FALSE                    |
| A  LIKE B      | 字符串       | 如果A或B值为”NULL”，结果返回”NULL”。字符串A与B通过sql进行匹配，如果相符返回TRUE，不符返回FALSE。B字符串中  的”_”代表任一字符，”%”则代表多个任意字符。例如： (‘foobar’ like ‘foo’)返回FALSE，（ ‘foobar’ like  ‘foo_ _ _’或者 ‘foobar’ like ‘foo%’)则返回TURE |
| A  RLIKE B     | 字符串       | 如果A或B值为”NULL”，结果返回”NULL”。字符串A与B通过java进行匹配，如果相符返回TRUE，不符返回FALSE。例如：（  ‘foobar’ rlike ‘foo’）返回FALSE，（’foobar’ rlike ‘^f.*r$’ ）返回TRUE。 |
| A  REGEXP B    | 字符串       | 与RLIKE相同。                                                |



## 算数运算符

| 运算符 | 类型         | 说明                                                         |
| ------ | ------------ | ------------------------------------------------------------ |
| A +  B | 所有数字类型 | A和B相加。结果的与操作数值有共同类型。例如每一个整数是一个浮点数，浮点数包含整数。所以，一个浮点数和一个整数相加结果也是一个浮点数。 |
| A –  B | 所有数字类型 | A和B相减。结果的与操作数值有共同类型。                       |
| A *  B | 所有数字类型 | A和B相乘，结果的与操作数值有共同类型。需要说明的是，如果乘法造成溢出，将选择更高的类型。 |
| A /  B | 所有数字类型 | A和B相除，结果是一个double（双精度）类型的结果。             |
| A %  B | 所有数字类型 | A除以B余数与操作数值有共同类型。                             |
| A  & B | 所有数字类型 | 运算符查看两个参数的二进制表示法的值，并执行按位”与”操作。两个表达式的一位均为1时，则结果的该位为  1。否则，结果的该位为 0。 |
| A\|B   | 所有数字类型 | 运算符查看两个参数的二进制表示法的值，并执行按位”或”操作。只要任一表达式的一位为  1，则结果的该位为 1。否则，结果的该位为 0。 |
| A ^  B | 所有数字类型 | 运算符查看两个参数的二进制表示法的值，并执行按位”异或”操作。当且仅当只有一个表达式的某位上为  1 时，结果的该位才为 1。否则结果的该位为 0。 |
| ~A     | 所有数字类型 | 对一个表达式执行按位”非”（取反）。                           |



## 逻辑运算符

| 运算符   | 类型   | 说明                                                         |
| -------- | ------ | ------------------------------------------------------------ |
| A  AND B | 布尔值 | A和B同时正确时,返回TRUE,否则FALSE。如果A或B值为NULL，返回NULL。 |
| A  && B  | 布尔值 | 与”A  AND B”相同                                             |
| A OR  B  | 布尔值 | A或B正确,或两者同时正确返返回TRUE,否则FALSE。如果A和B值同时为NULL，返回NULL。 |
| A \|  B  | 布尔值 | 与”A  OR B”相同                                              |
| NOT  A   | 布尔值 | 如果A为NULL或错误的时候返回TURE，否则返回FALSE。             |
| ! A      | 布尔值 | 与”NOT  A”相同                                               |



# 函数

## 数学函数

| 返回类型    | 函数                                               | 说明                                                         |
| ----------- | -------------------------------------------------- | ------------------------------------------------------------ |
| BIGINT      | round(double  a)                                   | 四舍五入                                                     |
| DOUBLE      | round(double  a, int d)                            | 小数部分d位之后数字四舍五入，例如round(21.263,2),返回21.26   |
| BIGINT      | floor(double  a)                                   | 对给定数据进行向下舍入最接近的整数。例如floor(21.2),返回21。 |
| BIGINT      | ceil(double  a), ceiling(double a)                 | 将参数向上舍入为最接近的整数。例如ceil(21.2),返回23.         |
| double      | rand(),  rand(int seed)                            | 返回大于或等于0且小于1的平均分布随机数（依重新计算而变）     |
| double      | exp(double  a)                                     | 返回e的n次方                                                 |
| double      | ln(double  a)                                      | 返回给定数值的自然对数                                       |
| double      | log10(double  a)                                   | 返回给定数值的以10为底自然对数                               |
| double      | log2(double  a)                                    | 返回给定数值的以2为底自然对数                                |
| double      | log(double  base, double a)                        | 返回给定底数及指数返回自然对数                               |
| double      | pow(double  a, double p) power(double a, double p) | 返回某数的乘幂                                               |
| double      | sqrt(double  a)                                    | 返回数值的平方根                                             |
| string      | bin(BIGINT  a)                                     | 返回二进制格式                                               |
| string      | hex(BIGINT  a) hex(string a)                       | 将整数或字符转换为十六进制格式                               |
| string      | unhex(string  a)                                   | 十六进制字符转换由数字表示的字符。                           |
| string      | conv(BIGINT  num, int from_base, int to_base)      | 将指定数值，由原来的度量体系转换为指定的试题体系。例如CONV(‘a’,16,2),返回 |
| double      | abs(double  a)                                     | 取绝对值                                                     |
| int  double | pmod(int  a, int b) pmod(double a, double b)       | 返回a除b的余数的绝对值                                       |
| double      | sin(double  a)                                     | 返回给定角度的正弦值                                         |
| double      | asin(double  a)                                    | 返回x的反正弦，即是X。如果X是在-1到1的正弦值，返回NULL。     |
| double      | cos(double  a)                                     | 返回余弦                                                     |
| double      | acos(double  a)                                    | 返回X的反余弦，即余弦是X，，如果-1<=  A <= 1，否则返回null.  |
| int  double | positive(int  a) positive(double a)                | 返回A的值，例如positive(2)，返回2。                          |
| int  double | negative(int  a) negative(double a)                | 返回A的相反数，例如negative(2),返回-2。                      |



## 类型转换函数

| 返回类型     | 函数                     | 说明                                                         |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| 指定  “type” | `cast(expr  as <type>) ` | 类型转换。例如将字符”1″转换为整数:cast(’1′  as bigint)，如果转换失败返回NULL。 |



## 日期函数

| 返回类型 | 函数                                             | 说明                                                         |
| -------- | ------------------------------------------------ | ------------------------------------------------------------ |
| string   | from_unixtime(bigint  unixtime[, string format]) | UNIX_TIMESTAMP参数表示返回一个值’YYYY-  MM – DD  HH：MM：SS’或YYYYMMDDHHMMSS.uuuuuu格式，这取决于是否是在一个字符串或数字语境中使用的功能。该值表示在当前的时区。 |
| bigint   | unix_timestamp()                                 | 如果不带参数的调用，返回一个Unix时间戳（从’1970-  01 – 0100:00:00′到现在的UTC秒数）为无符号整数。 |
| bigint   | unix_timestamp(string  date)                     | 指定日期参数调用UNIX_TIMESTAMP（），它返回参数值’1970-  01 – 0100:00:00′到指定日期的秒数。 |
| bigint   | unix_timestamp(string  date, string pattern)     | 指定时间输入格式，返回到1970年秒数：unix_timestamp(’2009-03-20′,  ‘yyyy-MM-dd’) = 1237532400 |
| string   | to_date(string  timestamp)                       | 返回时间中的年月日：  to_date(“1970-01-01 00:00:00″) = “1970-01-01″ |
| string   | to_dates(string  date)                           | 给定一个日期date，返回一个天数（0年以来的天数）              |
| int      | year(string  date)                               | 返回指定时间的年份，范围在1000到9999，或为”零”日期的0。      |
| int      | month(string  date)                              | 返回指定时间的月份，范围为1至12月，或0一个月的一部分，如’0000-00-00′或’2008-00-00′的日期。 |
| int      | day(string  date) dayofmonth(date)               | 返回指定时间的日期                                           |
| int      | hour(string  date)                               | 返回指定时间的小时，范围为0到23。                            |
| int      | minute(string  date)                             | 返回指定时间的分钟，范围为0到59。                            |
| int      | second(string  date)                             | 返回指定时间的秒，范围为0到59。                              |
| int      | weekofyear(string  date)                         | 返回指定日期所在一年中的星期号，范围为0到53。                |
| int      | datediff(string  enddate, string startdate)      | 两个时间参数的日期之差。                                     |
| int      | date_add(string  startdate, int days)            | 给定时间，在此基础上加上指定的时间段。                       |
| int      | date_sub(string  startdate, int days)            | 给定时间，在此基础上减去指定的时间段。                       |



## 条件函数

| 返回类型 | 函数                                                        | 说明                                                         |
| -------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| T        | if(boolean  testCondition, T valueTrue, T valueFalseOrNull) | 判断是否满足条件，如果满足返回一个值，如果不满足则返回另一个值。 |
| T        | COALESCE(T  v1, T v2, …)                                    | 返回一组数据中，第一个不为NULL的值，如果均为NULL,返回NULL。  |
| T        | CASE  a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END         | 当a=b时,返回c；当a=d时，返回e，否则返回f。                   |
| T        | CASE  WHEN a THEN b [WHEN c THEN d]* [ELSE e] END           | 当值为a时返回b,当值为c时返回d。否则返回e。                   |



## 字符串函数

| 返回类型                     | 函数                                                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| int                          | length(string  A)                                            | 返回字符串的长度                                             |
| string                       | reverse(string  A)                                           | 返回倒序字符串                                               |
| string                       | concat(string  A, string B…)                                 | 连接多个字符串，合并为一个字符串，可以接受任意数量的输入字符串 |
| string                       | concat_ws(string  SEP, string A, string B…)                  | 链接多个字符串，字符串之间以指定的分隔符分开。               |
| string                       | substr(string  A, int start) substring(string A, int start)  | 从文本字符串中指定的起始位置后的字符。                       |
| string                       | substr(string  A, int start, int len) substring(string A, int start, int len) | 从文本字符串中指定的位置指定长度的字符。                     |
| string                       | upper(string  A) ucase(string A)                             | 将文本字符串转换成字母全部大写形式                           |
| string                       | lower(string  A) lcase(string A)                             | 将文本字符串转换成字母全部小写形式                           |
| string                       | trim(string  A)                                              | 删除字符串两端的空格，字符之间的空格保留                     |
| string                       | ltrim(string  A)                                             | 删除字符串左边的空格，其他的空格保留                         |
| string                       | rtrim(string  A)                                             | 删除字符串右边的空格，其他的空格保留                         |
| string                       | regexp_replace(string  A, string B, string C)                | 字符串A中的B字符被C字符替代                                  |
| string                       | regexp_extract(string  subject, string pattern, int index)   | 通过下标返回正则表达式指定的部分。regexp_extract(‘foothebar’,  ‘foo(.*?)(bar)’, 2) returns ‘bar.’ |
| string                       | parse_url(string  urlString, string partToExtract [, string keyToExtract]) | 返回URL指定的部分。parse_url(‘http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1′,  ‘HOST’) 返回：’facebook.com’ |
| string                       | get_json_object(string  json_string, string path)            | select  a.timestamp, get_json_object(a.appevents, ‘$.eventid’),  get_json_object(a.appenvets, ‘$.eventname’) from log a; |
| string                       | space(int  n)                                                | 返回指定数量的空格                                           |
| string                       | repeat(string  str, int n)                                   | 重复N次字符串                                                |
| int                          | ascii(string  str)                                           | 返回字符串中首字符的数字值                                   |
| string                       | lpad(string  str, int len, string pad)                       | 返回指定长度的字符串，给定字符串长度小于指定长度时，由指定字符从左侧填补。 |
| string                       | rpad(string  str, int len, string pad)                       | 返回指定长度的字符串，给定字符串长度小于指定长度时，由指定字符从右侧填补。 |
| array                        | split(string  str, string pat)                               | 将字符串转换为数组。                                         |
| int                          | find_in_set(string  str, string strList)                     | 返回字符串str第一次在strlist出现的位置。如果任一参数为NULL,返回NULL；如果第一个参数包含逗号，返回0。 |
| array<array<string>>         | sentences(string  str, string lang, string locale)           | 将字符串中内容按语句分组，每个单词间以逗号分隔，最后返回数组。  例如sentences(‘Hello there! How are you?’) 返回：( (“Hello”, “there”), (“How”,  “are”, “you”) ) |
| array<struct<string,double>> | ngrams(array<array<string>>,  int N, int K, int pf)          | SELECT  ngrams(sentences(lower(tweet)), 2, 100 [, 1000]) FROM twitter; |
| array<struct<string,double>> | context_ngrams(array<array<string>>,  array<string>, int K, int pf) | SELECT  context_ngrams(sentences(lower(tweet)), array(null,null), 100, [, 1000]) FROM  twitter; |



## 聚合函数

| 返回类型                 | 函数                                                         | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| bigint                   | count(*)  , count(expr), count(DISTINCT expr[, expr_., expr_.]) | 返回记录条数。                                               |
| double                   | sum(col),  sum(DISTINCT col)                                 | 求和                                                         |
| double                   | avg(col),  avg(DISTINCT col)                                 | 求平均值                                                     |
| double                   | min(col)                                                     | 返回指定列中最小值                                           |
| double                   | max(col)                                                     | 返回指定列中最大值                                           |
| double                   | var_pop(col)                                                 | 返回指定列的方差                                             |
| double                   | var_samp(col)                                                | 返回指定列的样本方差                                         |
| double                   | stddev_pop(col)                                              | 返回指定列的偏差                                             |
| double                   | stddev_samp(col)                                             | 返回指定列的样本偏差                                         |
| double                   | covar_pop(col1,  col2)                                       | 两列数值协方差                                               |
| double                   | covar_samp(col1,  col2)                                      | 两列数值样本协方差                                           |
| double                   | corr(col1,  col2)                                            | 返回两列数值的相关系数                                       |
| double                   | percentile(col,  p)                                          | 返回数值区域的百分比数值点。0<=P<=1,否则返回NULL,不支持浮点型数值。 |
| array<double>            | percentile(col,  array(p~1,,\ [, p,,2,,]…))                  | 返回数值区域的一组百分比值分别对应的数值点。0<=P<=1,否则返回NULL,不支持浮点型数值。 |
| double                   | percentile_approx(col,  p[, B])                              | Returns  an approximate p^th^ percentile of a numeric column (including floating point  types) in the group. The B parameter controls approximation accuracy at the  cost of memory. Higher values yield better approximations, and the default is  10,000. When the number of distinct values in col is smaller than B, this  gives an exact percentile value. |
| array<double>            | percentile_approx(col,  array(p~1,, [, p,,2_]…) [, B])       | Same  as above, but accepts and returns an array of percentile values instead of a  single one. |
| array<struct\{‘x’,'y’\}> | histogram_numeric(col,  b)                                   | Computes  a histogram of a numeric column in the group using b non-uniformly spaced  bins. The output is an array of size b of double-valued (x,y) coordinates  that represent the bin centers and heights |
| array                    | collect_set(col)                                             | 返回无重复记录                                               |



# explode函数

1. explode命令可以将行数据，按照指定规则切分出多行
2. 用explode做行切分，注意表里只有一列，并且行数据是string类型，因为只有字符类型才能切分

> 案例：统计单词个数

- 原始数据

  ```
  hello tom hello bob
  hello joy
  hello rose
  hello joy
  hello jerry
  hello tom
  hello rose
  hello joy
  ```

- 将原始数据上传至HDFS，并创建对应外部表

  ```mysql
  create external table words(warr array<string>) 
      row format delimited collection items terminated by ' ' 
      location '/words';
  ```

- 通过explode指令做切分

  ```mysql
  select w, count(w) from 
  (select explode(warr) w from words)ws 
  group by(w);
  ```

- 结果

  ```
  hello 9
  tom 2
  bob 1
  joy 3
  rose 2
  jerry 1
  ```

  

# UDF - 自定义函数

> User Define Function

1. 导入依赖：

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.apache.hive</groupId>
           <artifactId>hive-exec</artifactId>
           <version>1.2.0</version>
       </dependency>
       <dependency>
           <groupId>org.apache.hive</groupId>
           <artifactId>hive-common</artifactId>
           <version>1.2.0</version>
       </dependency>
   </dependencies>
   ```

2. 自定义一个类继承`UDF`，实现`evaluate()`方法，这个方法返回值和参数类型由业务场景决定

   ```java
   package cn.tedu.hive;
   
   import org.apache.hadoop.hive.ql.exec.UDF;
   
   // abc 5 -> abcabcabcabcabc
   public class Repeat extends UDF {
   
       // Hive自动去寻找这个evaluate函数
       // 根据场景要求决定这个函数的参数和返回值类型
       public String evaluate(String str, int n) {
           if (n < 1)
               throw new IllegalArgumentException("个数至少为1！！！");
           StringBuffer sb = new StringBuffer();
           for (int i = 0; i < n; i++) {
               sb.append(str);
           }
           return sb.toString();
       }
   }
   ```

   

3. 打成jar包放到Linux上，位置随意：`/home/hivedata`

4. 格式统一：`zip -d H_Hive.jar 'META-INF/.SF' 'META-INF/.RSA' 'META-INF/*SF'`

5. 在Hive中

   1. 添加jar包：`add jar /home/hivedata/H_Hive.jar`

   2. 创建临时函数，给函数起名，并且绑定这个函数对应的类

      ```mysql
      create temporary function repeatstring as 'cn.tedu.hive.Repeat';
      ```

   3. 调用函数：

      ```mysql
      select repeatstring ("abc",5)
      ```

   4. 结果：`abcabcabc`


   [TOC]

   # 元数据

   1. 在Hive中：database名，table名，字段名，分区名，索引名，视图名等这些都是==元数据==（描述其他数据的数据）
   2. 在Hive中，元数据是存储在关系型数据库中。目前为止，Hive的元数据的数据库只支持DerBy和MySQL，如果不指定，则默认使用Hive中自带的DerBy
   3. DerBy是单连接数据库，不符合大数据的并发场景，所以需要将DerBy改为MySQL

   

   # 更换Hive数据库

   ## MySQL的安装

   ### 如果已经安装MySQL

   1. 编辑`my.cnf`

      ```sh
      vim /usr/my.cnf
      
      # 添加如下内容
      [client]
      default-character-set=utf8
      [mysql]
      default-character-set=utf8
      [mysqld]
      character_set_server=utf8
      sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
      ```

   2. 重启MySQL：`service mysql restart`

   

   ### 全新安装

   1. 查看是否有残缺的MySQL

      ```sh
      rpm -qa | grep mysql
      ```

   2. 删除残缺的MySQL

      ```sh
      rpm -ev --nodeps mysql-libs-5.1.73-8.el6_8.x86_64
      rpm -ev --nodeps tcl-mysqltcl-3.052-1.el6.x86_64
      ....
      ```

   3. 新添MySQL用户和用户组

      ```sh
      groupadd mysql
      useradd -r -g mysql mysql
      ```

   4. 下载MySQL安装包

      ```sh
      cd /home/software
      wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/MySQL-client-5.6.29-1.linux_glibc2.5.x86_64.rpm
      wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/MySQL-server-5.6.29-1.linux_glibc2.5.x86_64.rpm
      ```

   5. 安装MySQL

      ```sh
      rpm -ivh MySQL-server-5.6.29-1.linux_glibc2.5.x86_64.rpm 
      rpm -ivh MySQL-client-5.6.29-1.linux_glibc2.5.x86_64.rpm 
      ```

   6. 编辑`my.cnf`

      ```sh
      vim /usr/my.cnf
      
      # 添加如下内容
      [client]
      default-character-set=utf8
      [mysql]
      default-character-set=utf8
      [mysqld]
      character_set_server=utf8
      sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
      ```

   7. 添加到开机启动

      ```sh
      cp /usr/share/mysql/mysql.server /etc/init.d/mysql
      ```

   8. 重启MySQL

      ```sh
      service mysql restart
      ```

   9. 查看MySQL的密码

      ```sh
      cat /root/.mysql_secret
      ```

   10. 修改密码

       ```sh
       mysqladmin -u root -p password root
       ```

   

   

   > 如果之前安装过MySQL，需要删除感觉才能重新安装

   `rpm qa | grep mysql`---如果出现结果，删除：`rpm -ev --nodeps xxx`

   `rpm qa | grep MySQL`---如果出现结果，删除

   检查以下目录中是否有和MySQL相关的内容，如果有，则移除掉

   ```sh
   rm -rf * /usr/bin/mysql*
   rm -rf * /usr/sbin/mysql*
   rm -rf * /var/lib/mysql*
   rm -rf * /usr/share/info/mysql*
   rm -rf * /usr/share/man/mysql*
   rm -rf * /usr/include/mysql*
   rm -rf * /usr/lib/mysql*
   rm -rf * /usr/share/mysql*
   rm -rf /root/.mysql_secret
   ```

   全部移除之后，重新安装MySQL

   

   ## Hive的MySQL配置

   1. 下载MySQL的驱动包

      ```sh
      cd /home/software/apache-hive-1.2.0-bin/lib
      wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/mysql-connector-java-5.1.38-bin.jar
      ```

   2. 在Hive的conf目录下，新建并编辑`hive-site.xml`

      ```xml
      <configuration>
          <property>
              <name>javax.jdo.option.ConnectionURL</name>
              <value>jdbc:mysql://hadoop01:3306/hive?createDatabaseIfNotExist=true</value>
          </property>
          <property>
              <name>javax.jdo.option.ConnectionDriverName</name>
              <value>com.mysql.jdbc.Driver</value>
          </property>
          <property>
              <name>javax.jdo.option.ConnectionUserName</name>
              <value>root</value>
          </property>
          <property>
              <name>javax.jdo.option.ConnectionPassword</name>
              <value>root</value>
          </property>
      </configuration>
      ```

   3. 进入MySQL，执行以下语句

      ```mysql
      grant all privileges on *.* to 'root'@'hadoop01' identified by 'root' with grant option;
      
      grant all on *.* to 'root'@'%' identified by 'root';
      
      flush privileges;
      
      create database hive character set latin1;
      ```

   4. 进入Hive的bin目录，启动Hive

      ```sh
      sh hive
      ```

   5. 登录MySQL，查看mysql的hive数据库:`use hive`

      ![](https://note.youdao.com/yws/api/personal/file/2BA6CD273EF546628FCE9ED86B0F87FE?method=download&shareKey=928388d75326468899619d856679c898)

   

   > 如果启动失败，从第三个错误开始看，出现：`READ COMMITTED or READ UNCOMMITTED`，解决方案👇👇

   1. `vim /usr/my.cnf`
   2. 添加：`binlog_format=mixed`
   3. 重启MySQL：`service mysql restart`

[TOC]

# 体系架构

![](https://note.youdao.com/yws/api/personal/file/E18F08B76D3F4DBFBF81F866FF83B4E7?method=download&shareKey=70d8f4cb46de94c8d07922dd2bc4980c)



1. **用户接口主要有三个：CLI，JDBC 和 WUI**

   - CLI，最常用的模式。实际上在`>hive` 命令行下操作时，就是利用CLI用户接口
    - JDBC，通过java代码操作，需要启动hiveserver，然后连接操作

2. **Metastore**

   Hive将元数据存储在数据库中，如mysql、derby。Hive中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等

3. **解释器（complier)、优化器(optimizer)、执行器(executor)组件**

   这三个组件用于：HQL语句从词法分析、语法分析、编译、优化以及查询计划的生成。生成的查询计划存储在HDFS中，并在随后有MapReduce调用执行

4. **Hadoop**

   Hive的数据存储在HDFS中，大部分的查询、计算由MapReduce完成



# Hive执行流程

![](https://note.youdao.com/yws/api/personal/file/EA29B9431DC84C4383EC63A644D999EC?method=download&shareKey=386f6b4a26eb1d13288d0e0b02c3bafb)

> Driver：负责对外交互，对内协调
>
> Compiler：负责将SQL转化为MapReduce
>
> ExecutionEngine：负责和Yarn进行交互



1. 通过客户端提交一条Hql语句

2. 通过complier（编译组件）对Hql进行词法分析、语法分析。在这一步，编译器要知道此hql语句到底要操作哪张表

3. 去元数据库找表信息

4. 得到信息

5. complier编译器提交Hql语句分析方案

6. 执行流程

   1. executor 执行器收到方案后，执行方案（DDL过程）。在这里注意，执行器在执行方案时，会进行判断：如果当前方案不涉及到MR组件，比如为表添加分区信息、比如字符串操作等，比如简单的查询操作等，此时就会直接和元数据库交互，然后去HDFS上去找具体数据；如果方案需要转换成MR      job，则会将job 提交给Hadoop的JobTracker
   2. MR job完成，并且将运行结果写入到HDFS上
   3. 执行器和HDFS交互，获取结果文件信息

[TOC]



# 优化

## map side join

如果a join b，且a表较小，b表较大，那么可以考虑将a表放到内存中，然后专心处理b表，如果处理过程中需要a表的数据，那么从内存中将a表的数据找出来，而不需要从HDFS上再读取数据；

默认小表的大小不过25M；要求小表 join 大表，不能是大表 join 小表。

通过`set hive.auto.convert.join=true;`



## join

如果出现join和where，先where减少整体数据量，然后再进行join

优化前：

```mysql
select m.cid,u.id 
from order m join customer u 
on m.cid=u.id 
where m.dt=’20160801’; 
```

优化后：

```mysql
elect m.cid,u.id 
from (select cid from order where dt=’20160801’)m 
join customer u 
on m.cid = u.id
```



## group by

将相同的值放到一组，因为数据本身有倾斜特性(因为数据本身不均匀)，就会导致有的组数据多，有的组数据少，分到数据少的组的ReduceTask执行的就快；分到数据多的组的ReduceTask执行的就慢，这就导致了数据倾斜。

通过`set hive.groupby.skewindata=true;`来启动二阶段聚合



## distinct和聚合函数

count属于聚合函数，当聚合函数和distinct同时出现的时候，可以考虑先用多个ReduceTask去重，最后再利用一个ReduceTask来聚合

优化前：`select count(distinct id )from tablename`

优化后：`select count(*) from (select distinct id from tablename)tmp; `



1. 优化前

   - 由于对id=引入了distinct操作，所以在Map阶段无法利用combine对输出结果去消重，必须将id作为key输出
   - 在reduce阶段再对来自于不同的MapTask的结果进行消重，计入最终统计值
   - 由于ReduceTask的数量默认为1，所以导致MapTask的所有结果都只能由这一个ReduceTask处理，这就使得ReduceTask的执行效率成为整个任务的瓶颈
   - 虽然在使用hive的时候可以通过set     mapred.reduce.tasks设置ReduceTask的数量，但是Hive在处理COUNT这种“全聚合(full aggregates)”计算时，它会忽略用户指定的Reduce     Task数，而强制使用1

   ![](https://note.youdao.com/yws/api/personal/file/49A1B29A16684A7D953AC0CB769C1E31?method=download&shareKey=ea3d62c8667a426a2814ba65dff11629)

2. 优化后

   - 利用Hive对嵌套语句的支持，将原来一个MapReduce作业转换为两个作业：在第一阶段选出全部的非重复id，在第二阶段再对这些已消重的id进行计数
   - 在第一阶段我们可以通过增大Reduce的并发数，并发处理Map输出
   - 在第二阶段，由于id已经消重，因此COUNT(*)操作在Map阶段不需要输出原id数据，只输出一个合并后的计数即可。这样即使第二阶段Hive强制指定一个Reduce     Task，极少量的Map输出数据也不会使单一的Reduce Task成为瓶颈
   - 这一优化使得在同样的运行环境下，优化后的语句执行只需要原语句20%左右的时间

   ![](https://note.youdao.com/yws/api/personal/file/C8D08DEFBD4442CDA2ED7F54579B84DD?method=download&shareKey=0ee90e6ed382be03b8657f237a7a2a76)



## 整理切片个数

如果数据字段比较多处理逻辑比较复杂，可以考虑将切片调小增多任务数；

如果数据字段少处理逻辑比较简单，可以考虑将切片调大减少任务数



## JVM重用

每一个MapTask或者ReduceTask都会启动和关闭一次JVM子进程，那么如果一个节点接受到大量的MapTask或者ReduceTask就需要频繁的开启和关闭JVM子进程。

可以考虑JVM重用，即JVM子进程启动之后执行多个MapTask或者ReduceTask再结束，而不是执行一次就关闭，这样子可以减少JVM子进程频繁的开启和关闭，节省资源

[TOC]



# 概述

1. Sqoop是Apache提供的工具，用于HDFS和关系型数据库之间数据的导出和导入
2. 可以从HDFS导出数据到关系型数据库，也可以从关系型数据库导入数据到HDFS



# 安装和使用

1. 准备Sqoop安装包，官网地址：http://sqoop.apache.org
2. 配置jdk环境变量和Hadoop的环境变量。因为Sqoop在使用时回去找环境变量对应的路径，从而进行完整的工作
3. 解压Sqoop的安装包
4. 在关系型数据库中新建一个数据库`sqoop`，对应的表`order`
5. 需要将连接的数据库**驱动包**下载到Sqoop的**lib**目录下
6. 在Sqoop的**bin**目录下进行命令操作



# 基础指令

| 说明                     | 指令示例                                                     |
| ------------------------ | ------------------------------------------------------------ |
| 查看mysql所有数据库      | sh sqoop list-databases --connect  jdbc:mysql://192.168.150.138:3306/  -username root -password root |
| 查看指定数据库下的所有表 | sh  sqoop list-tables --connect jdbc:mysql://hadoop02:3306/hive -username root  -password root |
| 关系型数据库 ->hdfs      | 实现步骤：     先在mysql数据库的test数据下建立一张tabx表，并插入测试数据    <br />建表：create table tabx (id int,name varchar(20));  <br />插入：insert into tabx (id,name) values  (1,'aaa'),(2,'bbb'),(3,'ccc')，(1,'ddd'),(2,'eee'),(3,'fff');     <br />进入到sqoop的bin目录下，执行导入语句：<br />sh sqoop import --connect jdbc:mysql://192.168.150.138:3306/test   --username root --password root --table tabx --target-dir '/sqoop/tabx'       --fields-terminated-by '\|' -m 1; |
| hdfs ->关系型数据库      | 执行：sh sqoop export  --connect jdbc:mysql://192.168.150.138:3306/test --username root --password root  --export-dir '/sqoop/tabx/part-m-00000' --table taby -m  1 --fields-terminated-by '\|'     注：sqoop只能导出数据，不能自动建表。所以在导出之前，要现在mysql数据库里建好对应的表 |
| sh sqoop import -help    | 查看import的帮助指令                                         |



# 案例

> 将HDFS的`/order/order.txt`导入数据库

1. 在MySQL创建库和表

   ```mysql
   create database sqoop
   create table orders(oid int,odate varchar(25),pid int,num int);
   ```

2. 在sqoop的bin目录下执行命令

   ```sh
   sh sqoop export --connect jdbc:mysql://hadoop01:3306/sqoopdemo --username root --password root --export-dir '/order/order.txt' table orders -m 1 --fields-terminated-by ' '
   ```

3. 查看MySQL中数据库信息

   ```mysql
   select * from orders;
   ```

   

   [TOC]

   # 数据库和数据仓库的区别

   |          | 数据库                                                       | 数据仓库                                                   |
   | -------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
   | 数据量   | <=  GB                                                       | >=  TB                                                     |
   | 来源     | 来源相对比较单一，绑定某一个应用                             | 数据来源多样，可以来源于日志、爬虫、网页埋点、数据库等     |
   | 种类     | 结构化数据                                                   | 结构化、半结构化、非结构化数据                             |
   | 冗余     | 精简，避免冗余                                               | 人为的制造冗余 - 副本                                      |
   | 操作     | 提供完整的增删改查                                           | 一般只提供增和查的方法                                     |
   | 事务     | 强调事务(ACID)                                               | 弱事务甚至没有事务                                         |
   | 系统     | OLTP  - Online **Transaction** Process -  联机**事务**处理系统 | OLAP  - Online **Analysis** Process - 联机**分析**处理系统 |
   | 结果对象 | 程序员、DBA等技术人员                                        | 市场、客户、领导等非技术人员                               |
   | 场景     | 功能、业务                                                   | 决策                                                       |





![](https://note.youdao.com/yws/api/personal/file/40469FD1C6AC4E978BE7FA39F78DAB44?method=download&shareKey=67e0a151f50544ed602cb4c546fa80b1)

