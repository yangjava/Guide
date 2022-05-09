DBSyncer是一款开源的数据同步中间件，提供Mysql、Oracle、SqlServer、Elasticsearch(ES)、Kafka、SQL(Mysql/Oracle/SqlServer)等同步场景。支持上传插件自定义同步转换业务，提供监控全量和增量数据统计图、应用性能预警等。

> 特点

- 组合驱动，自定义库同步到库组合，关系型数据库与非关系型之间组合，任意搭配表同步映射关系
- 实时监控，驱动全量或增量实时同步运行状态、结果、同步日志和系统日志
- 开发插件，自定义转化同步逻辑

## 🌈应用场景

| 连接器    | 数据源                    | 目标源 | 支持版本(包含以下) |
| --------- | ------------------------- | ------ | ------------------ |
| Mysql     | ✔                         | ✔      | 5.7.19以上         |
| Oracle    | ✔                         | ✔      | 10g以上            |
| SqlServer | ✔                         | ✔      | 2008以上           |
| ES        | ✔                         | ✔      | 6.X以上            |
| Kafka     | 开发中                    | ✔      | 2.10-0.9.0.0以上   |
| SQL       | ✔                         |        |                    |
| 最近计划  | PostgreSQL(设计中)、Redis |        |                    |

## 📦安装配置

#### 步骤

1. 安装[JDK 1.8](https://gitee.com/link?target=https%3A%2F%2Fwww.oracle.com%2Fjava%2Ftechnologies%2Fjdk8-downloads.html)（省略详细）
2. 下载安装包[DBSyncer-1.0.0-Beta.zip](https://gitee.com/ghi/dbsyncer/releases)（也可手动编译）
3. 解压安装包，Window执行bin/startup.bat，Linux执行bin/startup.sh
4. 打开浏览器访问：[http://127.0.0.1:18686](https://gitee.com/link?target=http%3A%2F%2F127.0.0.1%3A18686)
5. 账号和密码：admin/admin

#### 增量同步配置（源库）

##### Mysql

- Dump Binlog二进制日志。Master同步Slave, 创建IO线程读取数据，写入relaylog，基于消息订阅捕获增量数据。
- 配置

> 修改my.ini文件

```
#服务唯一ID
server_id=1
log-bin=mysql_bin
binlog-format=ROW
max_binlog_cache_size = 256M
max_binlog_size = 512M
expire_logs_days = 7
#监听同步的库, 多个库使用英文逗号“,”拼接
replicate-do-db=test
```

##### Oracle

- CDN注册订阅。监听增删改事件，得到rowid，根据rowid执行SQL查询，得到变化数据。
- 配置

> 授予账号监听权限, 同时要求目标源表必须定义一个长度为18的varchar字段，通过接收rowid值实现增删改操作。

```
grant change notification to 你的账号
```

##### SqlServer

- SQL Server 2008提供了内建的方法变更数据捕获（Change Data Capture 即CDC）以实现异步跟踪用户表的数据修改。
- 配置

> 要求2008版本以上, 启动代理服务（Agent服务）, 连接账号具有 sysadmin 固定服务器角色或 db_owner 固定数据库角色的成员身份。对于所有其他用户，具有源表SELECT 权限；如果已定义捕获实例的访问控制角色，则还要求具有该数据库角色的成员身份。

##### ES

- 定时获取增量数据。
- 配置

> 账号具有访问权限。

##### 日志

> 建议Mysql和SqlServer都使用日志

![日志](https://images.gitee.com/uploads/images/2021/0906/181036_1f9a9e78_376718.png)

##### 定时

> 假设源表数据格式

![表数据格式](https://images.gitee.com/uploads/images/2021/0903/004406_68ef9bb4_376718.png) ![定时和过滤条件](https://images.gitee.com/uploads/images/2021/0903/004807_07cdf2b7_376718.png)

## ✨预览

### 驱动管理

![连接器和驱动](https://images.gitee.com/uploads/images/2021/0903/003755_01016fc1_376718.png)

### 驱动详情

![驱动详情](https://images.gitee.com/uploads/images/2021/0903/004031_a571f6b5_376718.png)

### 驱动表字段关系配置

![驱动表字段关系配置](https://images.gitee.com/uploads/images/2021/0903/004106_26399534_376718.png)

### 监控

![监控](https://images.gitee.com/uploads/images/2021/0728/000645_35a544b3_376718.png)

### 上传插件

![上传插件](https://images.gitee.com/uploads/images/2021/0806/232643_9b1f3f64_376718.png)

## 🔗开发依赖

- [JDK - 1.8.0_40](https://gitee.com/link?target=https%3A%2F%2Fwww.oracle.com%2Fjava%2Ftechnologies%2Fjdk8-downloads.html)（推荐版本以上）
- [Maven - 3.3.9](https://gitee.com/link?target=https%3A%2F%2Fdlcdn.apache.org%2Fmaven%2Fmaven-3%2F)（推荐版本以上）