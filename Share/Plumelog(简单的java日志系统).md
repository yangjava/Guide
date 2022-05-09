### 一.系统介绍

1. 无代码入侵的分布式日志系统，基于log4j、log4j2、logback搜集日志，设置链路ID，方便查询关联日志
2. 基于elasticsearch作为查询引擎
3. 高吞吐，查询效率高
4. 全程不占应用程序本地磁盘空间，免维护;对于项目透明，不影响项目本身运行
5. 无需修改老项目，引入直接使用，支持dubbo,支持springcloud

### 二.架构

- plumelog-core 核心组件包含日志搜集端，负责搜集日志并推送到kafka，redis等队列
- plumelog-server 负责把队列中的日志日志异步写入到elasticsearch
- plumelog-demo 基于springboot的使用案例
- plumelog-lite plumelog的嵌入式集成版本，免部署

### 三.使用方法

#### 使用前注意事项

- plumelog分三种启动模式，分别为redis,kafka,lite，外加嵌入式版本plumelog-lite
- lite模式，不依赖任何外部中间件直接启动使用，但是性能有限，一天10个G以内可以应付，还必须是SSD硬盘，适合管理系统类的小玩家
- redis,kafka模式可以集群分布式部署，适合大型玩家，互联网公司
- plumelog-lite plumelog的嵌入式集成版本，直接pom引用，嵌入在项目中，自带查询界面，适合单个独立小项目使用，外包软件的最佳伴侣