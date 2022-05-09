# Flink

## 概述

**Flink是分布式、高性能、随时可以用以及准确的流处理应用程序打造的开源流处理框架**。

Apache Flink是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。Flink被设计在所有常见的集群环境中运行，以内存执行速度和任意规模来执行计算。

按照Apache官方的介绍，Flink是一个对有界和无界数据流进行状态计算的分布式处理引擎和框架。通俗地讲，Flink就是一个流计算框架，主要用来处理流式数据。其起源于2010年德国研究基金会资助的科研项目“Stratosphere”，2014年3月成为Apache孵化项目，12月即成为Apache顶级项目。Flinken在德语里是敏捷的意思，意指快速精巧。其代码主要是由 Java 实现，部分代码由 Scala实现。Flink既可以处理有界的批量数据集，也可以处理无界的实时流数据，为批处理和流处理提供了统一编程模型。

在这里，我们解释了 Flink 架构的重要方面。

## Flink基本架构

### JobManager和TaskManager

#### JobManager

也称之为Master用于协调分布式执行，它们用来调度task协调检查点，协调失败时恢复等。Flink运行时至少存在一个master处理器，如果配置高可用模式则会存在多个master处理器，其中有一个是leader其他都是standby。

#### TaskManager

也称之为Worker，用于执行一个dataflow的task(或者特殊的subtask)、数据缓冲和datastream的交换，Flink运行时至少会存在一个worker处理器。
Master和Worker处理器可以直接在物理机上启动或者通过相Yarn这样的资源调度框架。
Worker连接到Master，告知自身的可用性进而获得任务分配。

### 无界数据流和有界数据流

#### 无界数据流

无界数据流有一个开始但是没有结束，它们不会在生成时终止 并提供数据，必须连续处理无界流，就是说必须在获取后立即处理event。处理无界数据通常要求以特定顺序获取event，以便能够推断结果完整性，无界流的处理称为流处理。

#### 有界数据流

有界数据流有明确定义的开始和结束。可以在执行任何计算之前通过获取所有数据来处理有界流，处理有界流不需要有序获取，因为可以始终对有界数据尽心排序，有界流的处理也称为批处理。
批处理的特点是有界、持久、大量，批处理非常适合需要访问全套记录才能完成的计算工作，一般要用于离线统计。流处理的特点是无界、实时、流处理方式无需针对整个暑假集执行操作，而是对通过系统传输的每个数据项执行操作，一般用于实时统计。

Spark中批处理由SparkSql实现，流处理由SparkStreaming实现。
那么Flink如何同时实现批处理和流处理的呢？？Flink将批处理(即处理有限的静态数据)视作一种特殊的流处理。

Flink是一个面向分布式数据流处理和批量数据处理的开源计算平台，能够基于同一个Flink运行时，提供支持流处理和批处理两种类型应用的功能。流处理一般需要支持低延迟、Exactly-once保证，而批处理需要支持高吞吐、高效处理。

Flink完全支持流处理就是说作为流处理看待时输入数据流是无界的，批处理被作为一种特殊的流处理只是它的输入数据流被定义为有界的。

## 数据流编程模型

![flink-level](F:\work\openGuide\BigData\flink-level.png)

大多数应用并不需要上述的底层抽象，而是针对核心API进行编程，比如DataStream API(有界或无界数据流)以及DataSet API(有界数据集)。可以在表与DataStream/DataSet之间无缝切换，以允许程序将Table API与DataStream以及DataSet混合使用。

## Flink运行架构



## Flink安装部署

Apache Flink 是一个分布式系统，需要计算资源才能执行应用程序。Flink 集成了所有常见的集群资源管理器，如[Hadoop YARN](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html)、[Apache Mesos](https://mesos.apache.org/)和[Kubernetes](https://kubernetes.io/) ，但也可以设置为作为独立集群运行。

### 本地模式

本地模式即在linux服务器直接解压flink二进制包就可以使用，不用修改任何参数，用于一些简单测试场景。

#### 下载安装包

直接在[Flink官网](https://flink.apache.org/downloads.html)下载安装包，为[flink-1.11.1-bin-scala_2.11.tgz](https://www.apache.org/dyn/closer.lua/flink/flink-1.11.2/flink-1.11.2-bin-scala_2.11.tgz)

#### 上传并解压至linux

```shell
[root@vm1 myapp]# pwd
/usr/local/myapp

[root@vm1 myapp]# ll
总用量 435772
-rw-r--r--.  1 root root  255546057 2月  8 02:29 flink-1.11.1-bin-scala_2.11.tgz
```

解压到指定目录

```shell
[root@vm1 myapp]# tar -zxvf flink-1.11.1-bin-scala_2.11.tgz  -C /usr/local/myapp/flink/
```

#### 启动Flink

注意运行之前确保机器上已经安装了JDK1.8或以上版本，并配置了JAVA_HOME环境变量。

```shell
[root@vm1 ~]# java -version
java version "1.8.0_261"
Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
```

进入flink目录执行启动命令

```shell
[root@vm1 ~]# cd /usr/local/myapp/flink/flink-1.11.1/
[root@vm1 flink-1.11.1]# bin/start-cluster.sh 
[root@vm1 flink-1.11.1]# jps
3577 Jps
3242 StandaloneSessionClusterEntrypoint
3549 TaskManagerRunner
```

执行Jps查看java进程，可以看到Flink相关进程已经启动。可以通过浏览器访问Flink的Web界面[http://vm1:8081](http://vm1:8081/)

能在本机浏览器访问上述页面的前提是Windows系统的hosts文件配了vm1这台服务器的主机名和IP的映射关系，并且linux服务器的防火墙已关闭。

#### 关闭防火墙

查看linux防火墙状态

```
[root@vm1 ~]# systemctl status firewalld
```

临时关闭防火墙

```
[root@vm1 ~]# systemctl stop firewalld
```

永久关闭防火墙

```
[root@vm1 ~]# systemctl disable firewalld
```

关闭Flink

执行`bin/stop-cluster.sh`

### 集群模式

集群环境适合在生产环境下面使用，且需要修改对应的配置参数。Flink提供了多种集群模式，我们这里主要介绍standalone和Flink on Yarn两种模式。

#### Standalone模式

Standalone是Flink的独立集群部署模式，不依赖任何其它第三方软件或库。如果想搭建一套独立的Flink集群，不依赖其它组件可以使用这种模式。搭建一个标准的Flink集群，需要准备3台Linux机器。

##### Linux机器规划

| 节点类型 | 主机名 | IP              |
| -------- | ------ | --------------- |
| Master   | vm1    | 192.168.174.136 |
| Slave    | vm2    | 192.168.174.137 |
| Slave    | vm3    | 192.168.174.138 |

在Flink集群中，Master节点上会运行JobManager(StandaloneSessionClusterEntrypoint)进程，Slave节点上会运行TaskManager(TaskManagerRunner)进程。

集群中Linux节点都要配置JAVA_HOME，并且节点之间需要设置ssh免密码登录，至少保证Master节点可以免密码登录到其他两个Slave节点，linux防火墙也需关闭。

##### 设置免密登录

1）先在每一台机器设置本机免密登录自身

```shell
[root@vm1 ~]# ssh-keygen -t rsa
[root@vm1 ~]#  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

在本机执行ssh登录自身，不提示输入密码则表明配置成功

```shell
[root@vm1 ~]# ssh vm1
Last login: Tue Sep 29 22:23:39 2020 from vm1
```

在其它机器vm2、vm3执行同样的操作:

ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

ssh vm2

ssh vm3

2）设置vm1免密登录其它机器

把vm1的公钥文件拷贝到其它机器vm2、vm3上

```shell
[root@vm1 ~]# scp ~/.ssh/id_rsa.pub root@vm2:~/
[root@vm1 ~]# scp ~/.ssh/id_rsa.pub root@vm3:~/
```

登录到vm2、vm3，把vm1的公钥文件追加到自己的授权文件中

```shell
[root@vm2 ~]# cat ~/id_rsa.pub  >> ~/.ssh/authorized_keys 
[root@vm3 ~]# cat ~/id_rsa.pub  >> ~/.ssh/authorized_keys 
```

如果提示没有 ~/.ssh/authorized_keys目录则可以在这台机器上执行ssh-keygen -t rsa。不建议手动创建.ssh目录！

验证在vm1上ssh登录vm2、vm3是否无需密码，不需要密码则配置成功！

```shell
[root@vm1 ~]# ssh vm2
Last login: Mon Sep 28 22:31:22 2020 from 192.168.174.133

[root@vm1 ~]# ssh vm3
Last login: Tue Sep 29 22:35:25 2020 from vm1
```

执行`exit`退回到本机

```shell
[root@vm3 ~]# exit
logout
Connection to vm3 closed.
[root@vm1 ~]# 
```

3）同样方式设置其它机器之间的免密登录

在vm2、vm3上执行同样的步骤

把vm2的公钥文件拷贝到vm1、vm3

```shell
[root@vm2 ~]# scp ~/.ssh/id_rsa.pub root@vm1:~/
[root@vm2 ~]# scp ~/.ssh/id_rsa.pub root@vm3:~/
[root@vm1 ~]#  cat ~/id_rsa.pub  >> ~/.ssh/authorized_keys 
[root@vm3 ~]#  cat ~/id_rsa.pub  >> ~/.ssh/authorized_keys 
```

把vm3的公钥文件拷贝到vm1、vm2

```shell
[root@vm3 ~]# scp ~/.ssh/id_rsa.pub root@vm1:~/
[root@vm3 ~]# scp ~/.ssh/id_rsa.pub root@vm2:~/
[root@vm1 ~]#  cat ~/id_rsa.pub  >> ~/.ssh/authorized_keys 
[root@vm2 ~]#  cat ~/id_rsa.pub  >> ~/.ssh/authorized_keys 
```

4）验证ssh免密码登录

```shell
[root@vm2 ~]# ssh vm1
[root@vm2 ~]# ssh vm3
[root@vm3 ~]# ssh vm1
[root@vm3 ~]# ssh vm2
```

##### 设置主机时间同步

如果集群内节点时间相差太大的话，会导致集群服务异常，所以需要保证集群内各节点时间一致。

执行命令`yum install -y ntpdate`安装ntpdate

执行命令`ntpdate -u ntp.sjtu.edu.cn` 同步时间

##### Flink安装步骤

下列步骤都是先在Master机器上操作，再拷贝到其它机器(确保每台机器都安装了jdk)

1. 解压Flink安装包

```
[root@vm1 myapp]# tar -zxvf flink-1.11.1-bin-scala_2.11.tgz -C /usr/local/myapp/flink/
```

1. 修改Flink的配置文件flink-1.11.1/conf/flink-conf.yaml

把jobmanager.rpc.address配置的参数值改为vm1

```
jobmanager.rpc.address: vm1
```

1. 修改Flink的配置文件flink-1.11.1/conf/workers

```shell
[root@vm1 conf]# vim workers 
vm2
vm3
```

1. 将vm1这台机器上修改后的flink-1.11.1目录复制到其他两个Slave节点

```shell
scp -rq /usr/local/myapp/flink vm2:/usr/local/myapp/
scp -rq /usr/local/myapp/flink vm3:/usr/local/myapp/
```

1. 在vm1这台机器上启动Flink集群服务

执行这一步时确保各个服务器防火墙已关闭

进入flink目录/flink-1.11.1/bin执行start-cluster.sh

```shell
[root@vm1 ~]# cd /usr/local/myapp/flink/flink-1.11.1/
[root@vm1 flink-1.11.1]# bin/start-cluster.sh 
Starting cluster.
Starting standalonesession daemon on host vm1.
Starting taskexecutor daemon on host vm2.
Starting taskexecutor daemon on host vm3.
```

1. 查看vm1、vm2和vm3这3个节点上的进程信息

```shell
[root@vm1 flink-1.11.1]# jps
4983 StandaloneSessionClusterEntrypoint
5048 Jps

[root@vm2 ~]# jps
4122 TaskManagerRunner
4175 Jps

[root@vm3 ~]# jps
4101 Jps
4059 TaskManagerRunner
```

1. 查看Flink Web UI界面，访问http://vm1:8081

8）提交任务执行

[root@vm1 flink-1.11.1]# bin/flink run ./examples/batch/WordCount.jar

提交任务可以在任意一台flink客户端服务器提交，本例中在vm1、vm2、vm3都可以

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-AJKInteJ-1603003404554)(image-20201017200539366.png)]

1. 停止flink集群

```
bin/stop-cluster.sh
```

10）单独启动、停止进程

手工启动、停止主进程StandaloneSessionClusterEntrypoint

```shell
[root@vm1 flink-1.11.1]# bin/jobmanager.sh start
[root@vm1 flink-1.11.1]# bin/jobmanager.sh stop
```

手工启动、停止TaskManagerRunner(常用于向集群中添加新的slave节点)

```shell
[root@vm1 flink-1.11.1]# bin/taskmanager.sh start
[root@vm1 flink-1.11.1]# bin/taskmanager.sh stop
```

#### Flink on YARN 模式

Flink on Yarn模式使用YARN 作为任务调度系统，即在YARN上启动运行flink。好处是能够充分利用集群资源，提高服务器的利用率。这种模式的前提是要有一个Hadoop集群，并且只需公用一套hadoop集群就可以执行MapReduce和Spark以及Flink任务，非常方便。因此需要先搭建一个hadoop集群。

##### Hadoop集群搭建

1）下载并解压到指定目录

从[官网](https://hadoop.apache.org/releases.html)下载Hadoop二进制包，上传到linux服务器，并解压到指定目录。

```shell
[root@vm1 ~]# tar -zxvf hadoop-2.9.2.tar.gz -C /usr/local/myapp/hadoop/
```

2）配置环境变量

vim /etc/profile

```shell
export HADOOP_HOME=/usr/local/myapp/hadoop/hadoop-2.9.2/
export PATH=$PATH:$HADOOP_HOME/bin
```

执行hadoop version查看版本号

```shell
[root@vm1 hadoop]# source /etc/profile
[root@vm1 hadoop]# hadoop version
Hadoop 2.9.2
```

3）修改hadoop-env.sh文件

修改配置export JAVA_HOME=${JAVA_HOME}指定JAVA_HOME路径:

```shell
export JAVA_HOME=/usr/local/myapp/jdk/jdk1.8.0_261/
```

同时指定Hadoop日志路径，先创建好目录：

```shell
[root@vm1]# mkdir -p /data/hadoop_repo/logs/hadoop
```

再配置HADOOP_LOG_DIR

```shell
export HADOOP_LOG_DIR=/data/hadoop_repo/logs/hadoop
```

4）修改yarn-env.sh文件

指定JAVA_HOME路径

```shell
export JAVA_HOME=/usr/local/myapp/jdk/jdk1.8.0_261/
```

指定YARN日志目录：

```shell
[root@vm1 ~]# mkdir -p /data/hadoop_repo/logs/yarn
```

```shell
export YARN_LOG_DIR=/data/hadoop_repo/logs/yarn
```

4）修改core-site.xml

配置NameNode的地址fs.defaultFS、Hadoop临时目录hadoop.tmp.dir

NameNode和DataNode的数据文件都会存在临时目录下的对应子目录下

```xml
<configuration>
<property>
   <name>fs.defaultFS</name>
   <value>hdfs://vm1:9000</value>
 </property>
 <property>
   <name>hadoop.tmp.dir</name>
   <value>/data/hadoop_repo</value>
 </property>
</configuration>
```

6）修改hdfs-site.xml

dfs.namenode.secondary.http-address指定secondaryNameNode的http地址，本例设置vm2机器为SecondaryNameNode

```xml
<configuration>
 <property>
   <name>dfs.replication</name>
   <value>2</value>
 </property>
 <property>
   <name>dfs.namenode.secondary.http-address</name>
   <value>vm2:50090</value>
 </property>
</configuration>
```

7）修改yarn-site.xml

yarn.resourcemanager.hostname指定resourcemanager的服务器地址，本例设置vm1机器为hadoop主节点

```xml
<configuration>
<property>
   <name>yarn.nodemanager.aux-services</name>
   <value>mapreduce_shuffle</value>
</property>
<property>
   <name>yarn.resourcemanager.hostname</name>
   <value>vm1</value>
</property>
</configuration>
```

8）修改mapred-site.xml

[root@vm1 hadoop]# mv mapred-site.xml.template mapred-site.xml

```xml
<configuration>
<property>
   <name>mapreduce.framework.name</name>
   <value>yarn</value>
</property>
</configuration>
```

mapreduce.framework.name设置使用yarn运行mapreduce程序

9） 配置slaves

设置vm2、vm3为Hadoop副节点

[root@vm1 hadoop]# vim slaves

```shell
vm2
vm3
```

10）设置免密码登录

免密配置参考前文 设置服务器间相互免密登录

11）拷贝hadoop到其它机器

将在vm1上配置好的Hadoop目录拷贝到其它服务器

```shell
[root@vm1 hadoop]# scp -r /usr/local/myapp/hadoop/ vm2:/usr/local/myapp/
[root@vm1 hadoop]# scp -r /usr/local/myapp/hadoop/ vm3:/usr/local/myapp/
```

12）格式化HDFS

在Hadoop集群主节点vm1上执行格式化命令

```shell
[root@vm1 bin]# pwd
/usr/local/myapp/hadoop/hadoop-2.9.2/bin
[root@vm1 bin]# hdfs namenode -format
```

如果要重新格式化NameNode，则需要先将原来NameNode和DataNode下的文件全部删除，否则报错。NameNode和DataNode所在目录在`core-site.xml`中`hadoop.tmp.dir`、`dfs.namenode.name.dir`、`dfs.datanode.data.dir`属性配置

13）启动集群

直接启动全部进程

```
[root@vm1 hadoop-2.9.2]# sbin/start-all.sh
```

也可以单独启动HDFS

```
sbin/start-dfs.sh
```

也可以单独启动YARN

```
sbin/start-yarn.sh
```

14）查看web页面

要在本地机器http访问虚拟机先关闭linux防火墙，关闭linux防火墙请参照前文

查看HDFS Web页面：

http://vm1:50070/

查看YARN Web 页面：

http://vm1:8088/cluster

15）查看各个节点进程

```shell
[root@vm1 ~]# jps
5026 ResourceManager
5918 Jps
5503 NameNode

[root@vm2 ~]# jps
52512 NodeManager
52824 Jps
52377 DataNode
52441 SecondaryNameNode

[root@vm3 ~]# jps
52307 DataNode
52380 NodeManager
52655 Jps
```

16）停止Hadoop集群

```
[root@vm1 hadoop-2.9.2]# sbin/stop-all.sh
```

Hadoop集群搭建完成后就可以在Yarn上运行Flink了！

##### Flink on Yarn的两种方式

第1种：在YARN中预先初始化一个Flink集群，占用YARN中固定的资源。该Flink集群常驻YARN 中，所有的Flink任务都提交到这里。这种方式的缺点在于不管有没有Flink任务执行，Flink集群都会独占系统资源，除非手动停止。如果YARN中给Flink集群分配的资源耗尽，只能等待YARN中的一个作业执行完成释放资源才能正常提交下一个Flink作业。

第2种：每次提交Flink任务时单独向YARN申请资源，即每次都在YARN上创建一个新的Flink集群，任务执行完成后Flink集群终止，不再占用机器资源。这样不同的Flink任务之间相互独立互不影响。这种方式能够使得资源利用最大化，适合长时间、大规模计算任务。

下面分别介绍2种方式的具体步骤。

###### 第1种方式

不管是哪种方式，都要先运行Hadoop集群

1）启动Hadoop集群

```
[root@vm1 hadoop-2.9.2]# sbin/start-all.sh
```

2）将flink依赖的hadoop相关jar包拷贝到flink目录

```shell
[root@vm1]# cp /usr/local/myapp/hadoop/hadoop-2.9.2/share/hadoop/yarn/hadoop-yarn-api-2.9.2.jar /usr/local/myapp/flink/flink-1.11.1/lib
[root@vm1]# cp /usr/local/myapp/hadoop/hadoop-2.9.2/share/hadoop/yarn/sources/hadoop-yarn-api-2.9.2-sources.jar /usr/local/myapp/flink/flink-1.11.1/lib
```

还需要 flink-shaded-hadoop-2-uber-2.8.3-10.0.jar ，可以从maven仓库下载并放到flink的lib目录下。

3）创建并启动flink集群

在flink的安装目录下执行

```shell
bin/yarn-session.sh -n 2 -jm 512 -tm 512 -d
```

这种方式创建的是一个一直运行的flink集群，也称为flink yarn-session

创建成功后，可以访问hadoop任务页面，查看是否有flink任务成功运行：http://vm1:8088/cluster

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-bIcGDTS0-1603003404558)(image-20201015212535158.png)]

创建成功后，flink控制台会输出web页面的访问地址，可以在web页面查看flink任务执行情况：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-91J9xKue-1603003404560)(image-20201015213139655.png)]

控制台输出http://vm2:43243 可以认为flink的Jobmanager进程就运行在vm2上，且端口是43243。指定host、port提交flink任务时可以使用这个地址+端口

4）附着到flink集群

创建flink集群后会有对应的applicationId，因此执行flink任务时也可以附着到已存在的、正在运行的flink集群

```shell
#附着到指定flink集群
[root@vm1 flink-1.11.1]# bin/yarn-session.sh -id application_1602852161124_0001
```

applicationId参数是上一步创建flink集群时对应的applicationId

5） 提交flink任务

可以运行flink自带的wordcount样例：

```
[root@vm1 flink-1.11.1]# bin/flink run ./examples/batch/WordCount.jar
```

在flink web页面 http://vm2:43243/ 可以看到运行记录：

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ZYwxhNTz-1603003404561)(image-20201015213038724.png)]

可以通过-input和-output来手动指定输入数据目录和输出数据目录:

-input hdfs://vm1:9000/words
-output hdfs://vm1:9000/wordcount-result.txt

###### 第2种方式

这种方式很简单，就是在提交flink任务时同时创建flink集群

```
[root@vm1 flink-1.11.1]# bin/flink run -m yarn-cluster -yjm 1024 ./examples/batch/WordCount.jar
```

需要在执行上述命令的机器(即flink客户端)上配置环境变量YARN_CONF_DIR、HADOOP_CONF_DIR或者HADOOP_HOME环境变量，Flink会通过这个环境变量来读取YARN和HDFS的配置信息。

如果报下列错，则需要禁用hadoop虚拟内存检查：

```shell
Diagnostics from YARN: Application application_1602852161124_0004 failed 1 times (global limit =2; local limit is =1) due to AM Container for appattempt_1602852161124_0004_000001 exited with  exitCode: -103
Failing this attempt.Diagnostics: [2020-10-16 23:35:56.735]Container [pid=6890,containerID=container_1602852161124_0004_01_000001] is running beyond virtual memory limits. Current usage: 105.8 MB of 1 GB physical memory used; 2.2 GB of 2.1 GB virtual memory used. Killing container.
```

修改所有hadoop机器(所有 nodemanager)的文件$HADOOP_HOME/etc/hadoop/yarn-site.xml

```shell
<property>  
    <name>yarn.nodemanager.vmem-check-enabled</name>  
    <value>false</value>  
</property>
```

重启hadoop集群再次运行

```shell
[root@vm1 hadoop-2.9.2]# sbin/stop-all.sh
[root@vm1 hadoop-2.9.2]# sbin/start-all.sh
[root@vm1 flink-1.11.1]# bin/flink run  -m yarn-cluster  -yjm 1024 ./examples/batch/WordCount.jar
```

任务成功执行，控制台输出如下。可以使用控制台输出的web页面地址vm3:44429查看任务。不过这种模式下任务执行完成后Flink集群即终止，所以输入地址vm3:44429时可能看不到结果，因为此时任务可能执行完了，flink集群终止，页面也访问不了了。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-XsBPibbF-1603003404563)(image-20201016000427565.png)]

上述Flink On Yarn的2种方式案例中分别使用了两个命令：yarn-session.sh 和 flink run

yarn-session.sh 可以用来在Yarn上创建并启动一个flink集群，可以通过如下命令查看常用参数：

```
[root@vm1 flink-1.11.1]# bin/yarn-session.sh -h
```

-n :表示分配的容器数量，即TaskManager的数量

-jm:设置jobManagerMemory，即JobManager的内存，单位MB

-tm:设置taskManagerMemory ，即TaskManager的内存，单位MB

-d: 设置运行模式为detached，即后台独立运行

-nm：设置在YARN上运行的应用的name（名字）

-id: 指定任务在YARN集群上的applicationId ,附着到后台独立运行的yarn session中

flink run命令既可以提交任务到Flink集群中执行，也可以在提交任务时创建一个新的flink集群，可以通过如下命令查看常用参数：

```
[root@vm1 flink-1.11.1]# bin/flink run -h
```

-m: 指定主节点(JobManger)的地址，在此命令中指定的JobManger地址优先于配置文件中的

-c: 指定jar包的入口类，此参数在jar 包名称之前

-p:指定任务并行度，同样覆盖配置文件中的值

flink run使用举例：

1）提交并执行flink任务，默认查找当前YARN集群中已有的yarn-session的JobManager

```shell
[root@vm1 flink-1.11.1]# bin/flink run ./examples/batch/WordCount.jar -input hdfs://vm1:9000/hello.txt -output hdfs://vm1:9000/result_hello
```

2）提交flink任务时显式指定JobManager的的host的port，该域名和端口是创建flink集群时控制台输出的

```shell
[root@vm1 flink-1.11.1]# bin/flink run -m vm3:39921 ./examples/batch/WordCount.jar  -input hdfs://vm1:9000/hello.txt -output hdfs://vm1:9000/result_hello
```

3）在YARN中启动一个新的Flink集群，并提交任务

```shell
[root@vm1 flink-1.11.1]# bin/flink run  -m yarn-cluster  -yjm 1024 ./examples/batch/WordCount.jar -input hdfs://vm1:9000/hello.txt -output hdfs://vm1:9000/result_hello
```

#### Flink on Yarn集群HA

Flink on Yarn模式的HA利用的是YARN的任务恢复机制。Flink on Yarn模式依赖hadoop集群，这里可以使用前文中的hadoop集群。这种模式下的HA虽然依赖YARN的任务恢复机制，但是Flink任务在恢复时，需要依赖检查点产生的快照。快照虽然存储在HDFS上，但是其元数据保存在zk中，所以也需要一个zk集群，使用前文配置好的zk集群即可。

## 快速入门案例

### 流式处理

POM文件

```

    <properties>
        <hadoop.hdfs.version>2.10.1</hadoop.hdfs.version>
        <!--        <hadoop.hdfs.version>2.7.1</hadoop.hdfs.version>-->
        <flink.version>1.11.2</flink.version>
        <scala.binary.version>2.11</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>

    <dependencyManagement>

        <dependencies>

            <dependency>
                <groupId>com.lmax</groupId>
                <artifactId>disruptor</artifactId>
                <version>3.4.2</version>
            </dependency>

            <!-- 需要去掉其中的低版本依赖，部分api的使用方法返回值不一致，需要版本一致-->
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-yarn_${scala.binary.version}</artifactId>
                <version>${flink.version}</version>
            </dependency>

            <!--  flink相关包-->
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-java</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-clients_${scala.binary.version}</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-table-api-java-bridge_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-table-planner-blink_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-streaming-scala_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
                <version>${flink.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-connector-elasticsearch6_2.11</artifactId>
                <version>${flink.version}</version>
                <!--            <scope>provided</scope>-->
            </dependency>
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-connector-filesystem_2.12</artifactId>
                <version>${flink.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
                <version>${flink.version}</version>
            </dependency>

            <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>6.0.6</version>
            </dependency>

            <!-- https://mvnrepository.com/artifact/org.apache.bahir/flink-connector-redis -->
            <dependency>
                <groupId>org.apache.bahir</groupId>
                <artifactId>flink-connector-redis_2.11</artifactId>
                <version>1.0</version>
            </dependency>

            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-json</artifactId>
                <version>${flink.version}</version>
            </dependency>


            <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
                <version>3.3.0</version>
            </dependency>

            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-high-level-client</artifactId>
                <version>6.6.1</version>
            </dependency>

            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
                <version>1.2.74</version>
            </dependency>

            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-runtime-web_${scala.binary.version}</artifactId>
                <version>${flink.version}</version>
            </dependency>


            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>${slf4j.version}</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
                <version>${slf4j.version}</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-to-slf4j</artifactId>
                <version>2.14.0</version>
                <scope>provided</scope>
            </dependency>

            <dependency>
                <groupId>org.apache.hadoop</groupId>
                <artifactId>hadoop-client</artifactId>
                <version>3.1.3</version>
                <scope>provided</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <!--指定打的jar包使用的jdk版本-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

最简单的例子WindowWordCount

```
package com.flink;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

public class WindowWordCount {

    public static void main(String[] args) throws Exception {

        ExecutionEnvironment executionEnvironment =ExecutionEnvironment.getExecutionEnvironment();
        String path="E:\\flink\\flink-wordcount\\src\\main\\resources\\helloword.txt";
        DataSet<String> stringDataSource = executionEnvironment
                .readTextFile(path);
        DataSet<Tuple2<String, Integer>> set =
                stringDataSource.flatMap(new myFlatMap())
                        .groupBy(0)
                        .sum(1);
        set.print();
    }


    public  static  class myFlatMap implements FlatMapFunction<String, Tuple2<String,Integer>> {
        @Override
        public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
            String [] words=  s.split(" ");
            for ( String word: words) {
                collector.collect(new Tuple2<>(word,1));
            }
        }
    }
}


```

### 批处理

### 提交flink集群运行

#### flink代码打成jar包

## Flink核心API讲解

## 1.Environment

Flink Job在提交执行计算时，需要首先建立和Flink框架之间的联系，也就指的是当前的flink运行环境，只有获取了环境信息，才能将task调度到不同的taskManager执行。而这个环境对象的获取方式相对比较简单

```
// 批处理环境
val env = ExecutionEnvironment.getExecutionEnvironment
// 流式数据处理环境
val env = StreamExecutionEnvironment.getExecutionEnvironment
```

## 2.Source

Flink框架可以从不同的来源获取数据，将数据提交给框架进行处理, 我们将获取数据的来源称之为数据源.

Flink框架可以从不同的来源获取数据，将数据提交给框架进行处理, 我们将获取数据的来源称之为数据源.

### 2.1.从集合读取数据

一般情况下，可以将数据临时存储到内存中，形成特殊的数据结构后，作为数据源使用。这里的数据结构采用集合类型是比较普遍的

```

```

### 2.2从文件中读取数据

```
```

### 2.3 kafka读取数据

Kafka作为消息传输队列，是一个分布式的，高吞吐量，易于扩展地基于主题发布/订阅的消息系统。在现今企业级开发中，Kafka 和 Flink成为构建一个实时的数据处理系统的首选

#### 引入kafka连接器的依赖

```
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```

### 2.4 自定义数据源

大多数情况下，前面的数据源已经能够满足需要，但是难免会存在特殊情况的场合，所以flink也提供了能自定义数据源的方式

## 3.Transform

在Spark中，算子分为转换算子和行动算子，转换算子的作用可以通过算子方法的调用将一个RDD转换另外一个RDD，Flink中也存在同样的操作，可以将一个数据流转换为其他的数据流。

转换过程中，数据流的类型也会发生变化，那么到底Flink支持什么样的数据类型呢，其实我们常用的数据类型，Flink都是支持的。比如：Long, String, Integer, Int, 元组，样例类，List, Map等。

### 3.1 map

- 映射：将数据流中的数据进行转换, 形成新的数据流，消费一个元素并产出一个元素
- 参数：Scala匿名函数或MapFunction
- 返回：DataStream

#### 3.1.1 MapFunction

Flink为每一个算子的参数都至少提供了Scala匿名函数和函数类两种的方式，其中如果使用函数类作为参数的话，需要让自定义函数继承指定的父类或实现特定的接口。例如：MapFunction

#### 3.1.2 RichMapFunction

所有Flink函数类都有其Rich版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。也有意味着提供了更多的，更丰富的功能。例如：RichMapFunction

Rich Function有一个生命周期的概念。典型的生命周期方法有：

- open()方法是rich function的初始化方法，当一个算子例如map或者filter被调 用之前open()会被调用
- close()方法是生命周期中的最后一个调用的方法，做一些清理工作
- getRuntimeContext()方法提供了函数的RuntimeContext的一些信息，例如函数执行     的并行度，任务的名字，以及state状态

#### 3.1.3 flatMap

- 扁平映射：将数据流中的整体拆分成一个一个的个体使用，消费一个元素并产生零到多个元素
- 参数：Scala匿名函数或FlatMapFunction
- 返回：DataStream

### 3.2. filter

- 过滤：根据指定的规则将满足条件（true）的数据保留，不满足条件(false)的数据丢弃
- 参数：Scala匿名函数或FilterFunction
- 返回：DataStream

### 3.3 keyBy

在Spark中有一个GroupBy的算子，用于根据指定的规则将数据进行分组，在flink中也有类似的功能，那就是keyBy，根据指定的key对数据进行分流

- 分流：根据指定的Key将元素发送到不同的分区，相同的Key会被分到一个分区（这里分区指的就是下游算子多个并行节点的其中一个）。keyBy()是通过哈希来分区的

- 参数：Scala匿名函数或POJO属性或元组索引，不能使用数组
- 返回：KeyedStream

### 3.4 shuffle

- 打乱重组（洗牌）：将数据按照均匀分布打散到下游
- 参数：无
- 返回：DataStream

### 3.5. split

在某些情况下，我们需要将数据流根据某些特征拆分成两个或者多个数据流，给不同数据流增加标记以便于从流中取出。

### 3.6 select

将数据流进行切分后，如何从流中将不同的标记取出呢，这时就需要使用select算子了。

### 3.7 connect

在某些情况下，我们需要将两个不同来源的数据流进行连接，实现数据匹配，比如订单支付和第三方交易信息，这两个信息的数据就来自于不同数据源，连接后，将订单支付和第三方交易信息进行对账，此时，才能算真正的支付完成。

Flink中的connect算子可以连接两个保持他们类型的数据流，两个数据流被Connect之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。

### 3.8 union

对两个或者两个以上的DataStream进行union操作，产生一个包含所有DataStream元素的新DataStream

connect与 union 区别：

1. union之前两个流的类型必须是一样，connect可以不一样
2. connect只能操作两个流，union可以操作多个。

### 3.9 Operator

Flink作为计算框架，主要应用于数据计算处理上， 所以在keyBy对数据进行分流后，可以对数据进行相应的统计分析

#### 3.9.1 滚动聚合算子（Rolling Aggregation）

这些算子可以针对KeyedStream的每一个支流做聚合。执行完成后，会将聚合的结果合成一个流返回，所以结果都是DataStream

#### 3.9.2 reduce

一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。

### 3.9.3 process

Flink在数据流通过keyBy进行分流处理后，如果想要处理过程中获取环境相关信息，可以采用process算子自定义实现 1)继承KeyedProcessFunction抽象类，并定义泛型：`[KEY, IN, OUT]`

## 4.Sink

Sink有下沉的意思，在Flink中所谓的Sink其实可以表示为将数据存储起来的意思，也可以将范围扩大，表示将处理完的数据发送到指定的存储系统的输出操作

之前我们一直在使用的print方法其实就是一种Sink。

之前我们一直在使用的print方法其实就是一种Sink。

```
  @PublicEvolving
    public DataStreamSink<T> print(String sinkIdentifier) {
        PrintSinkFunction<T> printFunction = new PrintSinkFunction(sinkIdentifier, false);
        return this.addSink(printFunction).name("Print to Std. Out");
    }
```



### Flink处理流程与核心概念

Flink中有对数据处理的操作进行抽象，称为Transformation Operator，而对于整个Dataflow的开始和结束分别对应着Source Operator和Sink Operator。

### DataStream API(流处理)

### DataSet API(批处理)

## window、watermark

# 基于 Flink SQL CDC 的实时数据同步方案

Flink 1.11 引入了 Flink SQL CDC，CDC 能给我们数据和业务间能带来什么变化？本文由 Apache Flink PMC，阿里巴巴技术专家伍翀 (云邪）分享，内容将从传统的数据同步方案，基于 Flink CDC 同步的解决方案以及更多的应用场景和 CDC 未来开发规划等方面进行介绍和演示。

1、传统数据同步方案
2、基于 Flink SQL CDC 的数据同步方案（Demo）
3、Flink SQL CDC 的更多应用场景
4、Flink SQL CDC 的未来规划

传统的数据同步方案与 Flink SQL CDC 解决方案
业务系统经常会遇到需要更新数据到多个存储的需求。例如：一个订单系统刚刚开始只需要写入数据库即可完成业务使用。某天 BI 团队期望对数据库做全文索引，于是我们同时要写多一份数据到 ES 中，改造后一段时间，又有需求需要写入到 Redis 缓存中。



很明显这种模式是不可持续发展的，这种双写到各个数据存储系统中可能导致不可维护和扩展，数据一致性问题等，需要引入分布式事务，成本和复杂度也随之增加。我们可以通过 CDC（Change Data Capture）工具进行解除耦合，同步到下游需要同步的存储系统。通过这种方式提高系统的稳健性，也方便后续的维护。



Flink SQL CDC 数据同步与原理解析

CDC 全称是 Change Data Capture ，它是一个比较广义的概念，只要能捕获变更的数据，我们都可以称为 CDC 。业界主要有基于查询的 CDC 和基于日志的 CDC ，可以从下面表格对比他们功能和差异点。



经过以上对比，我们可以发现基于日志 CDC 有以下这几种优势：

能够捕获所有数据的变化，捕获完整的变更记录。在异地容灾，数据备份等场景中得到广泛应用，如果是基于查询的 CDC 有可能导致两次查询的中间一部分数据丢失
每次 DML 操作均有记录无需像查询 CDC 这样发起全表扫描进行过滤，拥有更高的效率和性能，具有低延迟，不增加数据库负载的优势
无需入侵业务，业务解耦，无需更改业务模型
捕获删除事件和捕获旧记录的状态，在查询 CDC 中，周期的查询无法感知中间数据是否删除


基于日志的 CDC 方案介绍

从 ETL 的角度进行分析，一般采集的都是业务库数据，这里使用 MySQL 作为需要采集的数据库，通过 Debezium 把 MySQL Binlog 进行采集后发送至 Kafka 消息队列，然后对接一些实时计算引擎或者 APP 进行消费后把数据传输入 OLAP 系统或者其他存储介质。

Flink 希望打通更多数据源，发挥完整的计算能力。我们生产中主要来源于业务日志和数据库日志，Flink 在业务日志的支持上已经非常完善，但是在数据库日志支持方面在 Flink 1.11 前还属于一片空白，这就是为什么要集成 CDC 的原因之一。

Flink SQL 内部支持了完整的 changelog 机制，所以 Flink 对接 CDC 数据只需要把CDC 数据转换成 Flink 认识的数据，所以在 Flink 1.11 里面重构了 TableSource 接口，以便更好支持和集成 CDC。





重构后的 TableSource 输出的都是 RowData 数据结构，代表了一行的数据。在RowData 上面会有一个元数据的信息，我们称为 RowKind 。RowKind 里面包括了插入、更新前、更新后、删除，这样和数据库里面的 binlog 概念十分类似。通过 Debezium 采集的 JSON 格式，包含了旧数据和新数据行以及原数据信息，op 的 u表示是 update 更新操作标识符，ts_ms 表示同步的时间戳。因此，对接 Debezium JSON 的数据，其实就是将这种原始的 JSON 数据转换成 Flink 认识的 RowData。

选择 Flink 作为 ETL 工具

当选择 Flink 作为 ETL 工具时，在数据同步场景，如下图同步结构：



通过 Debezium 订阅业务库 MySQL 的 Binlog 传输至 Kafka ，Flink 通过创建 Kafka 表指定 format 格式为 debezium-json ，然后通过 Flink 进行计算后或者直接插入到其他外部数据存储系统，例如图中的 Elasticsearch 和 PostgreSQL。



但是这个架构有个缺点，我们可以看到采集端组件过多导致维护繁杂，这时候就会想是否可以用 Flink SQL 直接对接 MySQL 的 binlog 数据呢，有没可以替代的方案呢？

答案是有的！经过改进后结构如下图：



社区开发了 flink-cdc-connectors 组件，这是一个可以直接从 MySQL、PostgreSQL 等数据库直接读取全量数据和增量变更数据的 source 组件。

flink-cdc-connectors 可以用来替换 Debezium+Kafka 的数据采集模块，从而实现 Flink SQL 采集+计算+传输（ETL）一体化，这样做的优点有以下：

开箱即用，简单易上手
减少维护的组件，简化实时链路，减轻部署成本
减小端到端延迟
Flink 自身支持 Exactly Once 的读取和计算
数据不落地，减少存储成本
支持全量和增量流式读取
binlog 采集位点可回溯
基于 Flink SQL CDC 的数据同步方案实践
下面给大家带来 3 个关于 Flink SQL + CDC 在实际场景中使用较多的案例。在完成实验时候，你需要 Docker、MySQL、Elasticsearch 等组件，具体请参考每个案例参考文档。

案例 1 : Flink SQL CDC + JDBC Connector

这个案例通过订阅我们订单表（事实表）数据，通过 Debezium 将 MySQL Binlog 发送至 Kafka，通过维表 Join 和 ETL 操作把结果输出至下游的 PG 数据库。



案例 2 : CDC Streaming ETL

模拟电商公司的订单表和物流表，需要对订单数据进行统计分析，对于不同的信息需要进行关联后续形成订单的大宽表后，交给下游的业务方使用 ES 做数据分析，这个案例演示了如何只依赖 Flink 不依赖其他组件，借助 Flink 强大的计算能力实时把 Binlog 的数据流关联一次并同步至 ES 。



例如如下的这段 Flink SQL 代码就能完成实时同步 MySQL 中 orders 表的全量+增量数据的目的。

CREATE TABLE orders (
  order_id INT,
  order_date TIMESTAMP(0),
  customer_name STRING,
  price DECIMAL(10, 5),
  product_id INT,
  order_status BOOLEAN
) WITH (
  'connector' = 'mysql-cdc',
  'hostname' = 'localhost',
  'port' = '3306',
  'username' = 'root',
  'password' = '123456',
  'database-name' = 'mydb',
  'table-name' = 'orders'
);

SELECT * FROM orders
为了让读者更好地上手和理解，我们还提供了 docker-compose 的测试环境。

案例 3 : Streaming Changes to Kafka

下面案例就是对 GMV 进行天级别的全站统计。包含插入/更新/删除，只有付款的订单才能计算进入 GMV ，观察 GMV 值的变化。



Flink SQL CDC 的更多应用场景
Flink SQL CDC 不仅可以灵活地应用于实时数据同步场景中，还可以打通更多的场景提供给用户选择。

Flink 在数据同步场景中的灵活定位

如果你已经有 Debezium/Canal + Kafka 的采集层 (E)，可以使用 Flink 作为计算层 (T) 和传输层 (L)
也可以用 Flink 替代 Debezium/Canal ，由 Flink 直接同步变更数据到 Kafka，Flink 统一 ETL 流程
如果不需要 Kafka 数据缓存，可以由 Flink 直接同步变更数据到目的地，Flink 统一 ETL 流程
Flink SQL CDC : 打通更多场景

实时数据同步，数据备份，数据迁移，数仓构建
优势：丰富的上下游（E & L），强大的计算（T），易用的 API（SQL），流式计算低延迟
数据库之上的实时物化视图、流式数据分析
索引构建和实时维护
业务 cache 刷新
审计跟踪
微服务的解耦，读写分离
基于 CDC 的维表关联
下面介绍一下为何用 CDC 的维表关联会比基于查询的维表查询快。

基于查询的维表关联



目前维表查询的方式主要是通过 Join 的方式，数据从消息队列进来后通过向数据库发起 IO 的请求，由数据库把结果返回后合并再输出到下游，但是这个过程无可避免的产生了 IO 和网络通信的消耗，导致吞吐量无法进一步提升，就算使用一些缓存机制，但是因为缓存更新不及时可能会导致精确性也没那么高。

基于 CDC 的维表关联



我们可以通过 CDC 把维表的数据导入到维表 Join 的状态里面，在这个 State 里面因为它是一个分布式的 State ，里面保存了 Database 里面实时的数据库维表镜像，当消息队列数据过来时候无需再次查询远程的数据库了，直接查询本地磁盘的 State ，避免了 IO 操作，实现了低延迟、高吞吐，更精准。

Tips：目前此功能在 1.12 版本的规划中，具体进度请关注 FLIP-132 。

未来规划
FLIP-132 ：Temporal Table DDL（基于 CDC 的维表关联）
Upsert 数据输出到 Kafka
更多的 CDC formats 支持（debezium-avro, OGG, Maxwell）
批模式支持处理 CDC 数据
flink-cdc-connectors 支持更多数据库
总结
本文通过对比传统的数据同步方案与 Flink SQL CDC 方案分享了 Flink CDC 的优势，与此同时介绍了 CDC 分为日志型和查询型各自的实现原理。后续案例也演示了关于 Debezium 订阅 MySQL Binlog 的场景介绍，以及如何通过 flink-cdc-connectors 实现技术整合替代订阅组件。除此之外，还详细讲解了 Flink CDC 在数据同步、物化视图、多机房备份等的场景，并重点讲解了社区未来规划的基于 CDC 维表关联对比传统维表关联的优势以及 CDC 组件工作。

希望通过这次分享，大家对 Flink SQL CDC 能有全新的认识和了解，在未来实际生产开发中，期望 Flink CDC 能带来更多开发的便捷和更丰富的使用场景。

Q & A
1、GROUP BY 结果如何写到 Kafka ？

因为 group by 的结果是一个更新的结果，目前无法写入 append only 的消息队列中里面去。更新的结果写入 Kafka 中将在 1.12 版本中原生地支持。在 1.11 版本中，可以通过 flink-cdc-connectors 项目提供的 changelog-json format 来实现该功能。

2、CDC 是否需要保证顺序化消费？

是的，数据同步到 kafka ，首先需要 kafka 在分区中保证有序，同一个 key 的变更数据需要打入到同一个 kafka 的分区里面。这样 flink 读取的时候才能保证顺序。
