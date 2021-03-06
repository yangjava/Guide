# 关于 GreatSQL

------

GreatSQL是源于Percona Server的分支版本，除了Percona Server已有的稳定可靠、高效、管理更方便等优势外，特别是进一步提升了MGR（MySQL Group Replication）的性能及可靠性，以及众多bug修复。此外，GreatSQL还合并了由华为鲲鹏计算团队贡献的两个Patch，分别针对OLTP和OLAP两种业务场景，尤其是InnoDB并行查询特性，TPC-H测试中平均提升聚合分析型SQL性能15倍，最高提升40多倍，特别适用于周期性数据汇总报表之类的SAP、财务统计等业务。

GreatSQL可以作为MySQL或Percona Server的可选替代方案，用于线上生产环境。

GreatSQL完全免费并兼容MySQL或Percona Server。

# 下载GreatSQL

------

[戳此下载GreatSQL](https://gitee.com/GreatSQL/GreatSQL/releases)

# 版本特性

------

GreatSQL除了提升MGR性能及可靠性，还引入InnoDB事务锁优化及并行查询优化等特性，以及众多BUG修复。 选用GreatSQl主要有以下几点优势：

- 提升MGR模式下的大事务并发性能及稳定性
- 改进MGR的GC及流控算法，以及减少每次发送数据量，避免性能抖动
- 在MGR集群AFTER模式下，解决了节点加入集群时容易出错的问题
- 在MGR集群AFTER模式下，强一致性采用多数派原则，以适应网络分区的场景
- 当MGR节点崩溃时，能更快发现节点异常状态，有效减少切主和异常节点的等待时间
- 优化InnoDB事务锁机制，在高并发场景中有效提升事务并发性能至少10%以上
- 实现InnoDB并行查询机制，极大提升聚合查询效率，TPC-H测试中，最高可提升40多倍，平均提升15倍。特别适用于周期性数据汇总报表之类的SAP、财务统计等业务
- 修复了MGR模式下可能导致数据丢失、性能抖动、节点加入恢复极慢等多个缺陷或BUG

# 注意事项

------

运行GreatSQL可能需要依赖jemalloc库，因此请先先安装上

```
yum -y install jemalloc jemalloc-devel
```

也可以把自行安装的lib库so文件路径加到系统配置文件中，例如：

```
[root@greatdb]# cat /etc/ld.so.conf
/usr/local/lib64/
```

而后执行下面的操作加载libjemalloc库，并确认是否已存在

```
[root@greatdb]# ldconfig

[root@greatdb]# ldconfig -p | grep libjemalloc
        libjemalloc.so.1 (libc6,x86-64) => /usr/local/lib64/libjemalloc.so.1
        libjemalloc.so (libc6,x86-64) => /usr/local/lib64/libjemalloc.so
```

my.cnf配置文件可参考[这份样例](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/my.cnf-example)。

就可以正常启动GreatSQL服务了。

# 版本历史

------

- [GreatSQL 更新说明 8.0.25(2021-8-26)](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/relnotes/changes-greatsql-8-0-25-20210820.md)
- [GreatSQL 更新说明 8.0.22 v20210410(2021-4-10)](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/relnotes/changes-greatsql-8-0-22-v20210410.md)
- [GreatSQL 更新说明 8.0.22(2021-4-1)](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/relnotes/changes-greatsql-8-0-22.md)

# 使用文档

------

- [在Linux下源码编译安装GreatSQL](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/build-greatsql-with-source.md)
- [利用Ansible安装GreatSQL并构建MGR集群](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/install-greatsql-with-ansible.md)
- [在Docker中部署GreatSQL并构建MGR集群](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/install-greatsql-with-docker.md)
- [MGR优化配置参考](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/mgr-best-options-ref.md)
- [InnoDB并行查询优化参考](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/innodb-parallel-execute.md)
- [利用GreatSQL部署MGR集群](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/using-greatsql-to-build-mgr-and-node-manage.md)
- [MySQL InnoDB Cluster+GreatSQL部署MGR集群](https://gitee.com/GreatSQL/GreatSQL-Doc/blob/master/docs/mysql-innodb-cluster-with-greatsql.md)
- [MySQL MGR专栏文章](https://gitee.com/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fmp%2Fhomepage%3F__biz%3DMjM5NzAzMTY4NQ%3D%3D%26hid%3D16%26sn%3D9d3d21966d850dcf158e5b676d9060ed%26scene%3D18%23wechat_redirect)

# 相关资源

- [GreatSQL-Docker](https://gitee.com/GreatSQL/GreatSQL-Docker)，在Docker中运行GreatSQL。
- [GreatSQL-Ansible](https://gitee.com/GreatSQL/GreatSQL-Ansible)，利用ansible一键安装GreatSQL并完成MGR集群部署。