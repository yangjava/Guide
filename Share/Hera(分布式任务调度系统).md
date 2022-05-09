# 介绍文章

[操作文档](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fscxwhite%2Fhera%2Fblob%2Fhera-master%2Fhera-admin%2Fsrc%2Fmain%2Fresources%2Fstatic%2Fhelp%2Fhelp.md)

[赫拉(hera)分布式任务调度系统之操作文档](https://gitee.com/link?target=https%3A%2F%2Fscx-white.blog.csdn.net%2Farticle%2Fdetails%2F102571798)

[赫拉(hera)分布式任务调度系统之架构，基本功能(一)](https://gitee.com/link?target=https%3A%2F%2Fblog.csdn.net%2Fsu20145104009%2Farticle%2Fdetails%2F85124746)

[赫拉(hera)分布式任务调度系统之项目启动(二)](https://gitee.com/link?target=https%3A%2F%2Fblog.csdn.net%2Fsu20145104009%2Farticle%2Fdetails%2F85161711)

[赫拉(hera)分布式任务调度系统之开发中心(三)](https://gitee.com/link?target=https%3A%2F%2Fblog.csdn.net%2Fsu20145104009%2Farticle%2Fdetails%2F85336364)

[赫拉(hera)分布式任务调度系统之版本(四)](https://gitee.com/link?target=https%3A%2F%2Fblog.csdn.net%2Fsu20145104009%2Farticle%2Fdetails%2F85778303)

[赫拉(hera)分布式任务调度系统之Q&A(五)](https://gitee.com/link?target=https%3A%2F%2Fblog.csdn.net%2Fsu20145104009%2Farticle%2Fdetails%2F86076137)

## 前言

在大数据平台，随着业务发展，每天承载着成千上万的ETL任务调度，这些任务集中在hive,shell脚本调度。怎么样让大量的ETL任务准确的完成调度而不出现问题，甚至在任务调度执行中出现错误的情况下，任务能够完成自我恢复甚至执行错误告警与完整的日志查询。`hera`任务调度系统就是在这种背景下衍生的一款分布式调度系统。随着hera集群动态扩展，可以承载成千上万的任务调度。它是一款原生的分布式任务调度，可以快速的添加部署`wokrer`节点，动态扩展集群规模。支持`shell,hive,spark`脚本调度,可以动态的扩展支持`python`等服务器端脚本调度。

> hera分布式任务调度系统是根据前阿里开源调度系统(`zeus`)进行的二次开发，其中zeus大概在2014年开源，开源后却并未进行维护。我公司(二维火)2015年引进了zeus任务调度系统，一直使用至今年11月份，在我们部门乃至整个公司发挥着不可替代的作用。在我使用zeus的这一年多，不得不承认它的强大，只要集群规模于配置适度，他可以承担数万乃至十万甚至更高的数量级的任务调度。但是由于zeus代码是未维护的，前端更是使用GWT技术，难于在`zeus`上面进行维护。我与另外一个小伙伴(花名：凌霄，现在阿里淘宝部门)于今年三月份开始重写`zeus`，改名赫拉(hera)

```
***项目地址：git@github.com:scxwhite/hera.git ***
```

## 架构

`hera`系统只是负责调度以及辅助的系统，具体的计算还是要落在`hadoop、hive、yarn、spark`等集群中去。所以此时又一个硬性要求，如果要执行`hadoop，hive，spark`等任务，我们的`hera`系统的`worker`一定要部署在这些集群某些机器之上。如果仅仅是`shell`,那么也至少需要`linux`系统。对于`windows`系统，可以把自己作为`master`进行调试。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191213100911982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zY3gtd2hpdGUuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

`hera`系统本身严格的遵从主从架构模式，由主节点充当着任务调度触发与任务分发器，从节点作为具体的任务执行器.架构图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191213100937780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zY3gtd2hpdGUuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70) `hera` 在 `2.4` 版本以上也支持了`emr` 集群，即允许任务执行在阿里云、亚马逊的 `emr` 机器之上，架构图如下： ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191114114902720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zY3gtd2hpdGUuYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

## 功能

![具体功能](https://img-blog.csdnimg.cn/20200922110012179.png)

- 支持任务的定时调度、依赖调度、手动调度、手动恢复、超级恢复、重跑历史
- 支持丰富的任务类型：`shell,hive,python,spark-sql,java`
- 可视化的任务`DAG`图展示，任务的执行严格按照任务的依赖关系执行
- 某个任务的上、下游执行状况查看，通过任务依赖图可以清楚的判断当前任务为何还未执行，删除该任务会影响那些任务。
- 支持上传文件到`hdfs`，支持使用`hdfs`文件资源
- 支持日志的实时滚动
- 支持任务失败自动恢复
- 实现集群HA，机器宕机环境实现机器断线重连与心跳恢复与`hera`集群`HA`，节点单点故障环境下任务自动恢复，`master`断开，`worker`抢占`master`
- 支持对`master/work` 负载，内存，进程，`cpu`信息的可视化查看
- 支持正在等待执行的任务，每个`worker`上正在执行的任务信息的可视化查看
- 支持实时运行的任务，失败任务，成功任务，任务耗时`top10`的可视化查看
- 支持历史执行任务信息的折线图查看 具体到某天的总运行次数，总失败次数，总成功次数，总任务数，总失败任务数，总成功任务数
- 支持关注自己的任务，自动调度执行失败时会向负责人发送邮件
- 对外提供`API`，开放系统任务调度触发接口，便于对接其它需要使用hera的系统
- 组下任务总览、组下任务失败、组下任务正在运行
- 支持`map-reduce`任务和`yarn`任务的实时取消。
- 支持任务超时提醒
- 支持用户与组的概念
- 支持任务操作历史记录查看与恢复
- 支持任务告警定位到个人
- 告警类型支持邮箱以及自定义的钉钉、企业微信、短信、电话等
- 支持任务各种条件的模糊搜索
- 支持阿里云emr的自动创建、销毁
- 支持亚马逊emr的自动创建、销毁、弹性伸缩
- （还有更多，等待大家探索）