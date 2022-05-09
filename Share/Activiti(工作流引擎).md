## 工作流

### 工作流简介

- 工作流(Workflow):

   

  工作流就是通过计算机技术对业务流程进行自动化管理。实现多个参与者按照预定的流程去自动执行业务流程。

  - **定义:** 通过计算机对业务流程自动化执行管理
  - **主要解决的是:** 使在多个参与者之间按照某种预定义的规则自动进行传递文档,信息或任务的过程.从而实现某个预期的业务目标,或者促使此目标的实现

- 工作流管理系统的目标:

  - 管理工作的流程以确保工作在正确的时间被期望的人员所执行
  - 在自动化进行的业务过程中插入人工的执行和干预

- 工作流框架:

  - **Activiti**,JBPM,OSWorkFlow,WorkFlow
  - 工作流框架底层需要有数据库提供支持



### 工作流术语



### 工作流引擎

- **ProcessEngine对象:** 这是Activiti工作的核心.负责生成流程运行时的各种实例及数据,监控和管理流程的运行



### BPM

- 业务流程管理:
  - 是一种以规范化的构造**端到端**的卓越业务流程为中心,以持续的提高组织业务绩效为目的的系统化方法
  - 常见商业管理教育如EMBA,MBA等均将BPM包含在内



### BPMN

- 业务流程[建模](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.zhihu.com%2Fsearch%3Fq%3D%E5%BB%BA%E6%A8%A1%26search_source%3DEntity%26hybrid_search_source%3DEntity%26hybrid_search_extra%3D%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A440721749%7D)与标注:
  - 这些图如何组合成一个业务流程图(Business Process Diagram)
  - 讨论BPMN的各种的用途:包括以何种精度来影响一个流程图中的模型
  - BPMN作为一个标准的价值
  - BPMN未来发展的远景



### 流对象

一个业务流程图有三个流对象的核心元素

- 事件
  - 一个事件用圆圈来描述,表示一个业务流程期间发生的东西
  - 事件影响流程的流动.一般有一个原因(触发器)或一个影响(结果)
  - 基于它们对流程的影响,有三种事件:**开始事件,中间事件,终止事件**

![img](https://pic1.zhimg.com/80/v2-ddef01655e8314c730b8cff87d3c0574_720w.jpg)

- 活动
  - 用圆角矩形表示,一个流程由一个活动或多个活动组成

![img](https://pic4.zhimg.com/80/v2-99d2cab4e9d258cd2e24caf7e70e3a53_720w.jpg)

- 条件
  - 条件用菱形表示,用于控制序列流的分支与合并。
  - 可以作为选择,包括路径的分支与合并
  - 内部的标记会给出控制流的类型



## Activiti开源工作流框架



### Activiti简介

- Activiti是一个开源的工作流引擎,它实现了BPMN 2.0规范,可以发布设计好的流程定义,并通过api进行流程调度
- Activiti 作为一个遵从 Apache 许可的工作流和业务流程管理开源平台,其核心是基于Java的超快速,超稳定的 BPMN2.0 流程引擎,强调流程服务的可嵌入性和可扩展性,同时更加强调面向业务人员
- Activiti 流程引擎重点关注在系统开发的易用性和轻量性上.每一项BPM业务功能Activiti流程引擎都以服务的形式提供给开发人员.通过使用这些服务,开发人员能够构建出功能丰富,轻便且高效的BPM应用程序



### Activiti服务结构

- **Activiti系统服务结构图**

![img](https://pic2.zhimg.com/80/v2-9a2a145fe3d5d4241bb0313683256cd1_720w.jpg)

- 核心类:

  - **ProcessEngine:** 流程引擎的抽象,可以通过此类获取需要的所有服务

- 服务类:

  - XxxService:

     

    通过ProcessEngine获取,Activiti将不同生命周期的服务封装在不同Service中,包括定义,部署,运行.通过服务类可获取相关生命周期中的服务信息

    - RepositoryService
      - **Repository Service提供了对repository的存取服务**
      - Activiti中每一个不同版本的业务流程的定义都需要使用一些**定义文件,部署文件和支持数据**(例如BPMN2.0XML文件,表单定义文件,流程定义图像文件等),这些文件都存储在Activiti内建的Repository中
    - RuntimeService
      - Runtime Service提供了**启动流程,查询流程实例,设置获取流程实例变量**等功能.此外它还提供了**对流程部署,流程定义和流程实例的存取服务**
    - TaskService
      - Task Service提供了对用户Task和Form相关的操作.它提供了**运行时任务查询,领取,完成,删除以及变量设置**等功能
    - HistoryService
      - History Service用于**获取正在运行或已经完成的流程实例的信息,**与Runtime Service中获取的流程信息不同,**历史信息包含已经持久化存储的永久信息，并已经被针对查询优化**
    - FormService
      - 使用Form Service可以**存取启动和完成任务所需的表单数据并且根据需要来渲染表单**
      - Activiti中的流程和状态Task均可以关联业务相关的数据
    - IdentityService
      - Identity Service提供了**对Activiti系统中的用户和组的管理功能**
      - Activiti中内置了用户以及组管理的功能,必须使用这些用户和组的信息才能获取到相应的Task
    - ManagementService
      - Management Service提供了**对Activiti流程引擎的管理和维护功能**
      - 这些功能不在工作流驱动的应用程序中使用，主要用于 Activiti 系统的日常维护

- 核心业务对象:

  - **org.activiti.engine.impl.persistence.entity**包下的类,包括Task,ProcessInstance,Execution等
  - 根据不同职责实现相应接口的方法(如需要持久化则继承PersistentObject接口),与传统的实体类不同



### Activiti组件

- Activiti上下文组件Context:

   

  用来保存生命周期比较长,全局性的信息,类似Application.主要包括如下三类:

  - **CommandContext:** 命令上下文-保存每个命令必要的资源,如持久化需要的session
  - **ProcessEngineConfigurationImpl:** 流程引擎相关配置信息-整个引擎全局的配置信息.如数据源DataSource等.该对象为单例,在流程引擎创建的时候初始化
  - **ExecutionContext:** 持有ExecutionEntity对象

- 持久化组件:

  - Activiti使用mybatis作OR映射,并在此基础上增加设计了自己的持久化框架
  - 在流程引擎创建时初始化,顶层接口Session,SessionFactory
  - Session有两个实现类:
    - **DbSqlSession:** 负责sql表达式的执行
    - **AbstractManager:** 负责对象的持久化操作
  - SessionFactory有两个实现类:
    - **DbSqlSessionFactory:** 负责DbSqlSession相关操作
    - **GenericManagerFactory:** 负责AbstractManager相关操作

- Event-Listener组件:

  - Activiti允许客户代码介入流程执行,提供了事件监听组件
  - 监听的事件类型:
    - **TaskListener**
    - **JavaDelegate**
    - **Expression**
    - **ExecutionListener**
    - ProcessEngineConfigurationImpl持有DelegateInterceptor的某个实例,方便调用handleInvocation

- Cache组件

  - DbSqlSession中有cache的实现
  - Activiti基于List和Map来做缓存:如查询时先查缓存,没有则直接查询并放入缓存

- 异步执行组件

  - Activiti可以执行任务,JobExecutor为其核心类,JobExecutor包含三个主要属性:
    - **JobAcquisitionThread**
    - **BlockingQueue**
    - **ThreadPoolExecutor**
    - 方法ProcessEngines在引擎启动时调用JobExecutor.start,JobAcquisitionThread 线程即开始工作，其run方法不断循环执行AcquiredJobs中的job，执行一次后线程等待一定时间直到超时或者JobExecutor.jobWasAdded方法，因为有新任务而被调用。



### 流程虚拟机PVM

- 流程虚拟机API暴露了流程虚拟机的POJO核心,流程虚拟机API描述了一个工作流流程必备的组件,这些组件包括:
  - **PvmProcessDefinition:** 流程的定义,形象点说就是用户画的那个图.静态含义
  - **PvmProcessInstance:** 流程实例,用户发起的某个PvmProcessDefinition的一个实例.动态含义
  - **PvmActivity:** 流程中的一个节点
  - **PvmTransition:** 衔接各个节点之间的路径,形象点说就是图中各个节点之间的连接线
  - **PvmEvent:** 流程执行过程中触发的事件



### Activiti架构

![img](https://pic3.zhimg.com/80/v2-3f3a759ba496a8d5c33c15362686fe26_720w.jpg)

- Activiti Engine:
  - **最核心的模块**
  - 提供针对BPMN 2.0规范的解析,执行,创建,管理(任务,流程实例),查询历史记录并根据结果生成报表
- Activiti Modeler:
  - **模型设计器**
  - 适用于业务人员把需求转换为规范流程定义
- Activiti Designer:
  - 功能和Activiti Modeler类似,同样提供了基于BPMN 2.0规范的可视化设计功能,但是目前还没有完全支持BPMN规范的定义
  - 可以把业务需求人员用Signavio设计的流程定义(XML格式)导入到Designer中,从而让开发人员将其进一步加工成为可以运行的流程定义
- Activiti Explorer:
  - 可以用来管理仓库,用户,组,启动流程,任务办理等
  - 此组件使用REST风格API,提供一个基础的设计模型.如果业务简单,也可以直接使用无需开发.还可以作为后台管理员的流程、任务管理系统使用
- Activiti REST:
  - 提供RESTful风格的服务
  - 允许客户端以JSON的方式与引擎的REST API交互
  - 通用的协议具有跨平台,跨语言的特性



### Activiti数据库支持

- Activiti的后台由有数据库的支持
- 所有的表都以ACT_开头
- 第二部分是表示表的用途的两个字母标识
- 用途也和服务的API对应

```text
ACT_RE_* : 'RE'表示repository. 这个前缀的表包含了流程定义和流程静态资源(图片,规则...)
ACT_RU_* : 'RU'表示runtime.这些运行时的表,  包含流程实例,任务,变量,异步任务,等运行中的数据.
			Activiti只在流程实例执行过程中保存这些数据,在流程结束时就会删除这些记录.这样运行时表可以一直很小速度很快
ACT_ID_* : 'ID'表示identity.这些表包含身份信息,  比如用户,组...
ACT_HI_* : 'HI'表示history.这些表包含历史数据,  比如历史流程实例,变量,任务...
ACT_GE_* :通用数据. 用于不同场景下,  如存放资源文件
```

- **资源库流程规则表** (ACT_RE_*:'RE’表示repository. 这个前缀的表包含了流程定义和流程静态资源(图片,规则…))

```text
act_re_deployment  		部署信息表
act_re_model			流程设计模型部署表
act_re_procdef			流程定义数据表
```

- **运行时数据库表** (ACT_RU_*:'RU’表示runtime.这些运行时的表, 包含流程实例,任务,变量,异步任务,等运行中的数据.Activiti只在流程实例执行过程中保存这些数据,在流程结束时就会删除这些记录.这样运行时表可以一直很小速度很快)

```text
act_ru_execution 		运行时流程执行实例表
act_ru_identitylink		运行时流程人员表,主要存储任务节点与参与者的相关信息
act_ru_task				运行时任务节点表
act_ru_variable			运行时流程变量数据表
```

- **组织机构表** (ACT_ID_* : 'ID’表示identity.这些表包含身份信息, 比如用户,组…)

```text
act_id_group			用户组信息表
act_id_info				用户扩展信息表
act_id_membership		用户与用户组对应信息表
act_id_user				用户信息表
```

这四张表很常见,基本的组织机构管理,关于用户认证方面建议还是自己开发一套,组件自带的功能太简单,使用中有很多需求难以满足

- **历史数据库表** (ACT_HI_*:'HI’表示history.这些表包含历史数据, 比如历史流程实例,变量,任务…)

```text
act_hi_actinst 			历史节点表
act_hi_attachment		历史附件表
act_hi_comment			历史意见表
act_hi_identitylink		历史流程人员表
act_hi_detail			历史详情表，提供历史变量的查询
act_hi_procinst			历史流程实例表
act_hi_taskinst			历史任务实例表
act_hi_varinst			历史变量表
```

- **组织机构表** (ACT_GE_*:通用数据. 用于不同场景下, 如存放资源文件)

```text
act_ge_bytearray		二进制数据表
act_ge_property			属性数据表存储整个流程引擎级别的数据,初始化表结构时,会默认插入三条记录
```



### Activiti配置文件

- activiti.cfg.xml:

   

  Activiti核心配置文件，配置流程引擎创建工具的基本参数和数据库连接池参数

  - **定义数据库配置参数**
  - **配置连接池参数**



### Activiti特点



### 数据持久化

- Activiti的设计思想是简洁,快速
- 瓶颈体现在和数据库交换数据的过程中,针对这一点Activiti选择了使MyBatis,从而可以通过最优的SQL语句执行Command,仅凭如此就能让引擎在速度上保持最高的性能



### 引擎service接口

- Activiti流程引擎重点关注在系统开发的易用性和轻量性上,每一项BPM业务功能Activiti流程引擎都以服务的形式提供给开发人员,通过使用这些服务,开发人员能够构建出功能丰富,轻便且高效的BPM应用程序
- **activiti.cfg.xml**文件为**核心配置文件**,该配置文件集成在Spring的IOC容器当中,可以产生**ProcessEngineConfiguration**对象,这个对象就是**流程引擎的配置对象**
- **ProcessEngine**对象为**流程引擎对象**,该对象是工作流业务系统的核心,所有的业务操作都是由这个对象所派生出来的对象实现
- Activiti引擎提供了七大Service接口,均通过ProcessEngine获取,并且支持链式API编程风格



### 流程设计器

- 基于Web的Activiti Modeler流程设计器
- IDEA的actiBPM插件



### 原生支持Spring

- Activiti原生支持Spring,可以很轻松地进行Spring集成,非常方便管理事务和解析表达式(Expression)



### 分离运行时与历史数据

- Activiti继承自jBPM4,在表结构设计方面也遵循运行时与历史数据的分离
- 这样的设计可以快速读取运行时数据,仅当需要查询历史数据时再从专门的历史数据表中读取.这种设计方式可以大幅提高数据的存取效率,尤其是当数据日积月累时依然能够快速反应

# activiti-数据库设计

在整个流程的生命周期中，会产生流程流转的相关数据，Activiti流程引擎为我们提供了整套数据的存储方案，设计了不同类型的表来保存整个流程生命周期的数据。

 Activiti流程引擎的数据表分5大类，每一类的数据表均有不同的职责。

例如运行时数据表，专门用来记录流程运行时所产生的数据；

身份数据表，专门保存身份数据，包括用户、用户组、用户和用户组关系等。

Activiti为这些不同类别的数据表制定了规范的命名，不同职责的数据表均可以通过命名来体现。

例如运行时数据表会以 ACT_RU作为开头，历史数据表以ACT_HI作为开头。

本文数据版本为Activiti6.0，这个版本的较之前的版本有部分变化，在使用过程注意对应版本号，Activiti6.0中加入了基于DMN规范的规则引擎，本文不讲述关于规则引擎的数据表。

## 1.通用数据表

1.1 资源表
1.2 属性表

## 2.流程存储表

2.1 部署数据表
2.2 流程定义表

## 3.身份数据表

3.1 用户表
3.2 用户账号（信息）表
3.3 用户组表
3.4 关系表

## 4.运行时数据表

4.1 流程实例（执行流）表
4.2 流程任务表
4.3 流程参数表
4.4 流程与身份关系表
4.5 工作数据表
4.6 事件描述表

## 5.历史数据表

5.1 流程实例表
5.2 流程明细表
5.3 历史任务表和历史行为表
5.4 附件表和评论表

1.通用数据表
通用数据表用于存放一些通用的数据，这些表本身不关心特定的流程或者业务，只用于存放这些业务或者流程所使用的通用资源。它们可以独立存在于流程引擎或者应用系统中，其他类型的数据表有些地方会使用这些表中的资源数据。通用数据表有两张，都以ACT_GE开头，GE是单词general的缩写。



1.1 资源表
表ACT_GE_BYTEARRAY用于保存与流程引擎相关的资源，只要调用了Activiti存储服务的APl，涉及的资源均会被转换为byte数组保存到这个表中。在资源表中设计了一个BYTE字段，用来保存资源的内容，因此理论上可以用于保存任何类型的资源（文件或者其他来源的输入流）。一般情况下， Activit使用这个表来保存字符串、流程文件的内容、流程图片内容等。ACT_GE_BYTEARRAY表主要包含如下字段。

ID_：主键，类型为 varchar(64)。
REV_：数据版本，Activiti为一些有可能会被频繁修改的数据表加入该字段，用来表示该数据被操作的次数。
NAME_：资源名称，类型为varchar(255)。
BYTES_：资源内容，数据类型为longblob，最大可存4GB数据。
DEPLOYMENT_ID_：一次部署可以添加多个资源，该字段与部署表ACT_RE_DEPLOYMEN的主键相关联。
GENERATED_：是否由Activiti自动产生的资源，0表示false，1为true。

1.2 属性表
Activiti将全部的属性抽象为key-value键值对，每个属性都有名称和值，使用ACT_GE_PROPERTY来保存这些属性，该表有以下三个字段。

NAME_：属性名称，类型为varchar（64）。
VALUE_：属性值，varchar（300）。
REV_：数据版本，Activiti为一些有可能会被频繁修改的数据表加入该字段，用来表示该数据被操作的次数。

2.流程存储表
要想在Activiti使用流程，必须先将流程文件部署到流程引擎中，生成部署信息和流程定义信息。流程存储表就是用来保存流程文件的部署信息和流程定义这类数据。流程存储表有两张，都以ACT_RE开头，GE是单词repository的缩写。

2.1 部署数据表
在Activit中，一次部署可以添加多个资源，资源会被保存到资源表中（ACT_GE_BTYEARRAY），而对于部署，则部署信息会被保存到部署表中，部署表名称为ACT_RE_DEPLOYMENT，该表主要包含以下字段。

NAME_：部署的名称，可以调用Activiti的流程存储API来设置，类型为varchar(255)。
CATEGORY_：部署的分类，可以调用Activiti的流程存储API来设置，类型为varchar(255)。
KEY_：可以为流程部署设置一个关键字。
DEPLOY_TIME_：部署时间，类型为timestamp。
TENANT_ID：Activiti租户，该值主要用于记录启动的流程实例归属于哪个系统。Activiti5.15版本中增加了多租户的概念，该功能主要用于一个或者多个的引擎数据共享在一个数据库的使用场景，但是他们使用的数据库为同一个。因此操作的时候需要区分这些数据（部署的流程资源）的来源，以方便程序后续的处理。只需要将TENANT_ID理解为一个标记即可，其本身也没有更多的含义，仅仅是标记而已。
2.2 流程定义表
Activiti在部署添加资源时，如果发布部署的文件是流程文件（.bpmn或者BPMN.20.xml），则除了会解析这些流程文件，将内容保存到资源表外，还会解析流程文件的内容，形成特定的流程定义数据，写入流程定义表（ACT_RE_PROCDEF）中。ACT_RE_PROCDEF表主要包含以下字段。

ID_：主键，类型为varchar(64)，一般格式是ID串:VERSION:KEY，当这种格式的总长度超过64位时，只取前面的ID串，ID串在配置流程引擎时可以自定义配置，但是不能超过64位。
CATEGORY_：流程定义的分类，读取流程XML文件中的targetNamespace值。
NAME_：流程定义名称，读取流程文件中process元素的name属性。
KEY_：流程定义的key，读取流程文件中process元素的id属性，类型为varchar(255)。
DEPLOYMENT_ID_：流程定义对应的部署数据ID。
RESOURCE_NAME_：流程定义对应的资源名称，一般为流程文件的相对路径。
DGRM_RERSOURCE_NAME_：流程定义对应的流程图资源名称。
SUSPENSION_STATE_：表示流程定义的状态是激活还是中止，激活状态时该字段值为1，中止时字段值为2。如果流程定义被设置为中止状态，就将不能启动流程。
3.身份数据表
Activiti的提供了存储身份数据的表，但是这些表并没有与流程相关的数据去关联，可以独立于流程引擎而存在。身份数据有四张，表名称以ACT_ID开头，ID是单词identity的缩写。

3.1 用户表
流程引擎用户的信息被保存在ACT_ID_USER表中，该表有以下几个字段。

FIRST_：人名。
LAST_：姓氏。
EMAIL_：用户邮箱。
PWD_：用户密码，IdentityService中提供检查密码的API。
PICTURE ID：用户图片，对应资源表中的ID。
3.2 用户账号（信息）表
Activiti将用户、用户账号和用户信息分为三种数据，其中用户表保存用户的数据，而用户账号和用户信息，则被保存到ACT_ID_INFO表中，该表有以下字段。

USER_ID_：对应用户表的数据ID，但没有强制做外键关联。
TYPE_：信息类型，当前可以设置用户账号（ account）、用户信息（ userinfo）和NULL三种值。
KEY_：数据的键，可以根据该键来查找用户信息的值。
VALUE_：数据的值，类型为 varchar(255)。
PASSWORD_：用户账号的密码字段，不过当前版本的Activiti并没有使用该字段。
PARENT_ID_：该信息的父信息ID，如果一条数据设置了父信息ID，则表示该数据是用户账号（信息）的明细数据，例如一个账号有激活日期，那么激活日期就是该账号的明细数据，此处使用了自关联来实现。
3.3 用户组表
使用ACT_ID_GROUP表来保存用户组的数据，该表有以下几个字段

NAME_：用户组名称。
TYPE_：用户组类型，类型不由Activiti提供，Activiti会根据该字段的值进行查询。
3.4 关系表
一个用户组下有多个用户，一个用户可以属于不同的用户组，那么这种多对多的关系，就使用关系表来进行描述，关系表为ACT_ID_MEMBERSHIP，只有两个字段。

USER_ID_：_用户ID，不能为NULL。
GROUP_ID_：用户组ID，不能为NULL。
需要注意的是， ACT_ID_MEMBERSHIP的两个字段均做了外键约束，写入该表的数据必须要有用户和用户组数据与之关联。
4.运行时数据表
运行时数据表用来保存流程在运行过程中所产生的数据，例如流程实例、执行流、任务等。运行时数据表名称以ACT_RU开头，RU是单词runtime的缩写。

4.1 流程实例（执行流）表
流程启动后，会产生一个流程实例，同时会产生相应的执行流，流程实例和执行流数据均被保存在ACT_RU_EXECUTION表中，如果一个流程实例只有一条执行流，那么该表中只产生一条数据，该数据既表示执行流，也表示流程实例。ACT_RU_EXECUTION表有以下字段。

PROC_INST_ID_：流程实例ID，一个流程实例有可能会产生多个执行流，该字段表示执行流所属的流程实例。
BUSINESS_KEY_：启动流程时指定的业务主键。
PARENT_ID_：父执行流的ID，一个流程实例有可能会产生子执行流，该字段保存父执行流ID。
PROC_DEF_ID_：流程定义数据的ID。
ACT_ID_：当前执行流行为的ID，ID在流程文件中定义。
IS_ACTIVE_：该执行流是否活跃的标识。
IS_CONCURRENT_：执行流是否正在并行。
SUSPENSION_STATE_：标识流程的中断状态。
4.2 流程任务表
流程在运行过程中所产生的任务数据保存在ACT_RU_TASK表中，该表主要有如下字段。

EXECUTION_ID_：任务所在的执行流ID。
PROC_INST_ID_：对应的流程实例ID。
PROC_DEF_ID_：对应流程定义数据的ID。
NAME_：任务名称，在流程文件中定义。
DESCRIPTION_：任务描述，在流程文件中配置。
TASK_DEF_KEY_：任务定义的ID值，在流程文件中定义。
OWNER_：任务拥有人。
ASSIGNEE_：被指派执行该任务的人。
PRIORITY_：任务优先级数值。
DUE_DATE_：任务预订日期，类型为datetime。
4.3 流程参数表
Activiti提供了ACT_RU_VARIABLE表来存放流程中的参数，这类参数包括流程实例参数、执行流参数和任务参数，参数有可能会有多种类型，因此该表使用多个字段来存放参数值。ACT_RU_VARIABLE表主要有以下字段。

TYPE_：参数类型，该字段值可以为“boolean”、“bytes”、“serializable”、“date”、“double”、“Integer”、“jpa-entity”、“long”、“null”、“short”、“string”，这些字段值均为Activiti提供，还可以通过扩展来自定义参数类型。
NAME_：参数名称。
EXECUTION_ID_：该参数对应的执行ID，可以为null。
PROC_INST_ID_：该参数对应的流程实例ID，可以为null。
TASK_ID_：如果该参数是任务参数，就需要设置任务ID
BYTEARRAY_ID_：如果参数值是序列化对象，那么可以将该对象作为资源保存到资源表中，该字段保存资源表中数据的ID
DOUBLE_：参数类型为double的话，则值会保存到该字段中。
LONG_：参数类型为long的话，则值会保存到该字段中。
TEXT_：用于保存文本类型的参数值，该字段为varchar类型，长度为4000字节。
TEXT2_：与TEXT字段一样，用于保存文本类型的参数值。
4.4 流程与身份关系表
用户组和用户之间的关系，使用ACT_ID_MEMBERSHIP表保存，用户或者用户组与流程数据之间的关系，则使用ACT_RU_IDENTITYLINK表进行保存，相比于ACT_ID_MEMBERSHIP表， ACT_RU_IDENTITYLINK表的字段更多一些。

GROUP_ID_：该关系数据中的用户组ID。
TYPE_：该关系数据的类型，这里说明一下流程的人员参与角色：
assignee ：签收者，签收人或者被委托。
candidate：候选者，当一个节点被指定为用户组，或者被指定用户去处理，则TYPE_的类型为candidate，指的是该任务需要先被组用户或者指定用户签收认领。
owner：拥有者，只有在任务被委托出去后，它才有意义，它表示任务实际的拥有者。
starter：启动者，流程实例的发起者。
participant：参与者，包含查阅。
USER_ID_：用户ID。
TASK_ID_：任务ID。
PROC_DEF ID_：流程定义ID。
4.5 工作数据表
在流程执行的过程中，会有一些工作需要定时或者重复执行，这类工作数据被保存到工作表中，Activiti提供了四个工作表用于保存不同的工作数据。

ACT_RU_JOB_：一般工作表。
ACT_RU_DEADLETTER_JOB_：无法执行工作表，用于存放无法执行的工作。
ACT_RU_SUSPENDED_JOB_：中断工作表，中断工作产生后，会将工作保存到该表中。
ACT_RU_TIMER_JOB_：定时器工作表，用于存放定时器工作。
4.6 事件描述表
如果流程到达某类事件节点，Activiti会往ACT_RU_EVENT_SUBSCR表中加入事件描述数据，这些事件描述数据将会决定流程事件的触发。ACT_RU_EVENT_SUBSCR表有如下字段。

EVENT_TYPE_：事件类型，不同的事件会产生不同类型的事件描述，并不是所有的事件都会产生事件描述。
EVENT_NAME _：事件名称，在流程文件中定义。
EXECUTION_ID_：事件所在的执行流ID。
PROC_INST_ID_：事件所在的流程实例ID。
ACTIVITY_ID_：具体事件的ID，在流程文件中定义。
CONFIGURATION_：事件的配置属性，该字段中有可能存放流程定义ID、执行流ID、或者其他数据。
5.历史数据表
历史数据表就好像流程引擎的日志表，操作过的流程元素将会被记录到历史表中。历史数据表名称以ACT_HI开尖，“HI”是单词history的缩写。

5.1 流程实例表
流程实例的历史数据会被保存到ACT_HI_PR0CINST表中，流程启动的时候，就会将流程实例的数据写入到ACT_HI_PROCINST表中。历史流程实例表还会额外多记录流程的开始活动ID、结束活动ID等信息。

START_ACT_ID_：开始活动的ID，一般是流程开始时间的ID，在流程文件中定义。
END_ACT_ID_：流程最后一个活动的，一般是流程开始时间的ID，在流程文件中定义。
DELETE_REASON_：该流程实例被删除的原因。
该表的其他字段含义与运行时的流程实例表字段类似，在此不再赞述。
5.2 流程明细表
流程明细表（ACT_HI_DETAIL）会记录流程执行过程中的参数或者表单数据，由于在流程执行过程中，会产生大量这类数据，因此默认情况下，Activiti不会保存流程明细数据，除非将流程引擎的历史数据(history)配置为full。

5.3 历史任务表和历史行为表
当流程到达某个任务节点时，就会向历史任务表(ACT_HI_TASKINST)中写入历史任务数据，该表与运行时的任务表类似。历史行为表(ACT_HI_ACTINST)会记录每一个流程活动的实例，一个流程活动将会被记录为一条数据，根据该表可以追踪最完整的流程信息。

5.4 附件表和评论表
使用任务服务（Taskservice）的API可以添加附件和评论，这些附件和评论的数据将会 被保存到ACT_HI_ATTACHMENT和ACT_HI_COMMENT表中。ACT_HI_ATTACHMENT有如下字段。

USER_ID_：附件对应的用户ID， 可以力NULL。
NAME_：附件名称。
DESCRIPTION_：附件描述。
TYPE_：附件类型。
TASK_ID_：该附件对应的任务ID。
PROC_INST_ID_：对应的流程实例ID。
URL_：连接到该附件的URL。
CONTENT_ID_：附件内容ID， 附件的内容将会被保存到资源表中，该字段记录资源数据ID。
ACT_HI_COMMENT表实际不只保存评论数据，它还会保存某些事件数据，但它的表名为COMMENT， 因此更倾向把它叫作评论表，该表有如下字段。
TYPE_：评论的类型，可以设值为"event"或者"00-001"， 表示事件记录数据或者评论数据。
TIME_：数据产生的时间。
USER_ID_：产生评论数据的用户ID。
TASK_ID_：该评论数据的任务ID。
PROC_INST ID_：数据对应的流程实例ID。
ACTION_：该评论数据的操作标识。
MESSAGE_：该评论数据的信息。
FULL_MSG_：该字段同样记录评论数据的信息。
虽然附件表和评论表的命名遵守历史数据表的命名规范（以ACT_HI开头)，但是可以调用其他服务组件的API来往这两个表中写入数据，以笔者的理解，历史数据表实际上保存的是那种一经写入，就很少会发生变化（结构性变化）的数据。

# 1 Activiti数据库表结构

## 1.1   数据库表名说明

  Activiti工作流总共包含23张数据表，所有的表名默认以“**ACT_**”开头。

并且表名的第二部分用两个字母表明表的用例，而这个用例也基本上跟Service API匹配。

u **ACT_GE_\*** : “GE”代表“General”（通用），用在各种情况下；

u **ACT_HI_\*** : “HI”代表“History”（历史），这些表中保存的都是历史数据，比如执行过的流程实例、变量、任务，等等。Activit默认提供了4种历史级别：

Ø **none**: 不保存任何历史记录，可以提高系统性能；

Ø **activity**：保存所有的流程实例、任务、活动信息；

Ø **audit**：也是Activiti的**默认**级别，保存所有的流程实例、任务、活动、表单属性；

Ø **full**：最完整的历史记录，除了包含**audit**级别的信息之外还能保存详细，例如：流程变量。

对于几种级别根据对功能的要求选择，如果需要日后跟踪详细可以开启**full**。

 

u **ACT_ID_\*** : “ID”代表“Identity”（身份），这些表中保存的都是身份信息，如用户和组以及两者之间的关系。如果Activiti被集成在某一系统当中的话，这些表可以不用，可以直接使用现有系统中的用户或组信息；

u **ACT_RE_\*** : “RE”代表“Repository”（仓库），这些表中保存一些‘静态’信息，如流程定义和流程资源（如图片、规则等）；

u **ACT_RU_\*** : “RU”代表“Runtime”（运行时），这些表中保存一些流程实例、用户任务、变量等的运行时数据。Activiti只保存流程实例在执行过程中的运行时数据，并且当流程结束后会立即移除这些数据，这是为了保证运行时表尽量的小并运行的足够快；

 

## 1.2   数据库表结构

### 1.2.1  Activiti数据表清单:

| **表分类**                                                   | **表名**                                                     | **解释**                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------ |
| 一般数据                                                     | [ACT_GE_BYTEARRAY](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_GE_BYTEARRAY) | 通用的流程定义和流程资源 |
| [ACT_GE_PROPERTY](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_GE_PROPERTY) | 系统相关属性                                                 |                          |
| 流程历史记录                                                 | [ACT_HI_ACTINST](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_ACTINST) | 历史的流程实例           |
| [ACT_HI_ATTACHMENT](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_ATTACHMENT) | 历史的流程附件                                               |                          |
| [ACT_HI_COMMENT](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_COMMENT) | 历史的说明性信息                                             |                          |
| [ACT_HI_DETAIL](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_DETAIL) | 历史的流程运行中的细节信息                                   |                          |
| [ACT_HI_IDENTITYLINK](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_IDENTITYLINK) | 历史的流程运行过程中用户关系                                 |                          |
| [ACT_HI_PROCINST](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_PROCINST) | 历史的流程实例                                               |                          |
| [ACT_HI_TASKINST](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_TASKINST) | 历史的任务实例                                               |                          |
| [ACT_HI_VARINST](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_HI_VARINST) | 历史的流程运行中的变量信息                                   |                          |
| 用户用户组表                                                 | [ACT_ID_GROUP](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_ID_GROUP) | 身份信息-组信息          |
| [ACT_ID_INFO](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_ID_INFO) | 身份信息-组信息                                              |                          |
| [ACT_ID_MEMBERSHIP](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_ID_MEMBERSHIP) | 身份信息-用户和组关系的中间表                                |                          |
| [ACT_ID_USER](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_ID_USER) | 身份信息-用户信息                                            |                          |
| 流程定义表                                                   | [ACT_RE_DEPLOYMENT](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RE_DEPLOYMENT) | 部署单元信息             |
| [ACT_RE_MODEL](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RE_MODEL) | 模型信息                                                     |                          |
| [ACT_RE_PROCDEF](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RE_PROCDEF) | 已部署的流程定义                                             |                          |
| 运行实例表                                                   | [ACT_RU_EVENT_SUBSCR](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RU_EVENT_SUBSCR) | 运行时事件               |
| [ACT_RU_EXECUTION](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RU_EXECUTION) | 运行时流程执行实例                                           |                          |
| [ACT_RU_IDENTITYLINK](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RU_IDENTITYLINK) | 运行时用户关系信息                                           |                          |
| [ACT_RU_JOB](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RU_JOB) | 运行时作业                                                   |                          |
| [ACT_RU_TASK](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RU_TASK) | 运行时任务                                                   |                          |
| [ACT_RU_VARIABLE](file:///C:/Users/Administrator/Desktop/Activiti工作流数据库表结构.docx#LOCAL_1-ACT_RU_VARIABLE) | 运行时变量表                                                 |                          |

 

### 1.2.2表名:ACT_GE_BYTEARRAY（通用的流程定义和流程资源）

用来保存部署文件的大文本数据。

保存流程定义图片和xml、Serializable(序列化)的变量,即保存所有二进制数据，特别注意类路径部署时候，不要把svn等隐藏文件或者其他与流程无关的文件也一起部署到该表中，会造成一些错误（可能导致流程定义无法删除）。

| **ACT_GE_BYTEARRAY(act_ge_bytearray)** |                                   |                              |              |                                       |          |            |              |
| -------------------------------------- | --------------------------------- | ---------------------------- | ------------ | ------------------------------------- | -------- | ---------- | ------------ |
| **是否主键**                           | **字段名**                        | **字段描述**                 | **数据类型** | **可空**                              | **约束** | **缺省值** | **取值说明** |
| 是                                     | ID_                               | 主键ID，资源文件编号，自增长 | VARCHAR(64)  |                                       |          |            |              |
| REV_                                   | 版本号                            | INT(11)                      | 是           | Version                               |          |            |              |
| NAME_                                  | 部署的文件名称，                  | VARCHAR(255)                 | 是           | mail.bpmn、mail.png 、mail.bpmn20.xml |          |            |              |
| DEPLOYMENT_ID_                         | 来自于父表ACT_RE_DEPLOYMENT的主键 | VARCHAR(64)                  | 是           | 部署的ID                              |          |            |              |
| BYTES_                                 | 大文本类型，存储文本字节流        | LONGBLOB                     | 是           |                                       |          |            |              |
| GENERATED_                             | 是否是引擎生成。                  | TINYINT(4)                   | 是           | 0为用户生成1为Activiti生成            |          |            |              |

 

 

### 1.2.3  表名:ACT_GE_PROPERTY（系统相关属性）

属性数据表。存储这个流程引擎级别的数据。

| **ACT_GE_PROPERTY(act_ge_property)** |            |              |              |          |                                       |          |            |              |
| ------------------------------------ | ---------- | ------------ | ------------ | -------- | ------------------------------------- | -------- | ---------- | ------------ |
| **是否主键**                         | **字段名** | **字段描述** | **数据类型** | **长度** | **可空**                              | **约束** | **缺省值** | **取值说明** |
| 是                                   | NAME_      | 属性名称     | VARCHAR(64)  | 64       | schema.versionschema.historynext.dbid |          |            |              |
| VALUE_                               | 属性值     | VARCHAR(300) | 300          | 是       | 5.*create(5.*)                        |          |            |              |
| REV_INT                              | 版本号     | INT(11)      | 11           | 是       |                                       |          |            |              |

 

### 1.2.4表名:ACT_HI_ACTINST（历史节点表）

历史活动信息。这里记录流程流转过的所有节点，与HI_TASKINST不同的是，taskinst只记录usertask内容。

| **ACT_HI_ACTINST(act_hi_actinst)** |                |              |              |                                         |          |              |
| ---------------------------------- | -------------- | ------------ | ------------ | --------------------------------------- | -------- | ------------ |
| **是否主键**                       | **字段名**     | **字段描述** | **数据类型** | **可空**                                | **约束** | **取值说明** |
| 是                                 | ID_            | ID_          | VARCHAR(64)  |                                         |          |              |
| PROC_DEF_ID_                       | 流程定义ID     | VARCHAR(64)  |              |                                         |          |              |
| PROC_INST_ID_                      | 流程实例ID     | VARCHAR(64)  |              |                                         |          |              |
| EXECUTION_ID_                      | 流程执行ID     | VARCHAR(64)  |              |                                         |          |              |
| ACT_ID_                            | 活动ID         | VARCHAR(255) |              | 节点定义ID                              |          |              |
| TASK_ID_                           | 任务ID         | VARCHAR(64)  | 是           | 任务实例ID 其他节点类型实例ID在这里为空 |          |              |
| CALL_PROC_INST_ID_                 | 请求流程实例ID | VARCHAR(64)  | 是           | 调用外部流程的流程实例ID'               |          |              |
| ACT_NAME_                          | 活动名称       | VARCHAR(255) | 是           | 节点定义名称                            |          |              |
| ACT_TYPE_                          | 活动类型       | VARCHAR(255) |              | 如startEvent、userTask                  |          |              |
| ASSIGNEE_                          | 代理人员       | VARCHAR(64)  | 是           | 节点签收人                              |          |              |
| START_TIME_                        | 开始时间       | DATETIME     |              | 2013-09-15 11:30:00                     |          |              |
| END_TIME_                          | 结束时间       | DATETIME     | 是           | 2013-09-15 11:30:00                     |          |              |
| DURATION_                          | 时长，耗时     | BIGINT(20)   | 是           | 毫秒值                                  |          |              |

 

 

### 1.2.5  表名:ACT_HI_ATTACHMENT（附件信息）

| **ACT_HI_ATTACHMENT(act_hi_attachment)** |                  |               |              |          |                      |          |            |              |
| ---------------------------------------- | ---------------- | ------------- | ------------ | -------- | -------------------- | -------- | ---------- | ------------ |
| **是否主键**                             | **字段名**       | **字段描述**  | **数据类型** | **长度** | **可空**             | **约束** | **缺省值** | **取值说明** |
| 是                                       | ID_              | ID_           | VARCHAR(64)  | 64       | 主键ID               |          |            |              |
| REV_                                     | REV_             | INT(11)       | 11           | 是       | Version              |          |            |              |
| USER_ID_                                 | 用户id           | VARCHAR(255)  | 255          | 是       | 用户ID               |          |            |              |
| NAME_                                    | 名称             | VARCHAR(255)  | 255          | 是       | 附件名称             |          |            |              |
| DESCRIPTION_                             | 描述             | VARCHAR(4000) | 4000         | 是       | 描述                 |          |            |              |
| TYPE_                                    | 类型             | VARCHAR(255)  | 255          | 是       | 附件类型             |          |            |              |
| TASK_ID_                                 | 任务Id           | VARCHAR(64)   | 64           | 是       | 节点实例ID           |          |            |              |
| PROC_INST_ID_                            | 流程实例ID       | VARCHAR(64)   | 64           | 是       | 流程实例ID           |          |            |              |
| URL_                                     | 连接             | VARCHAR(4000) | 4000         | 是       | 附件地址             |          |            |              |
| CONTENT_ID_                              | 内容Id字节表的ID | VARCHAR(64)   | 64           | 是       | ACT_GE_BYTEARRAY的ID |          |            |              |

 

 

### 1.2.6  表名:ACT_HI_COMMENT（历史审批意见表）

| **ACT_HI_COMMENT(act_hi_comment)** |                                       |               |              |          |                                                              |          |            |              |
| ---------------------------------- | ------------------------------------- | ------------- | ------------ | -------- | ------------------------------------------------------------ | -------- | ---------- | ------------ |
| **是否主键**                       | **字段名**                            | **字段描述**  | **数据类型** | **长度** | **可空**                                                     | **约束** | **缺省值** | **取值说明** |
| 是                                 | ID_                                   | ID_           | VARCHAR(64)  | 64       | 主键ID                                                       |          |            |              |
| TYPE_                              | 意见记录类型，为comment时，为处理意见 | VARCHAR(255)  | 255          | 是       | 类型：event（事件）comment（意见）                           |          |            |              |
| TIME_                              | 记录时间                              | DATETIME      | 填写时间     |          |                                                              |          |            |              |
| USER_ID_                           | 用户Id                                | VARCHAR(255)  | 255          | 是       | 填写人                                                       |          |            |              |
| TASK_ID_                           | 任务Id                                | VARCHAR(64)   | 64           | 是       | 节点实例ID                                                   |          |            |              |
| PROC_INST_ID_                      | 流程实例Id                            | VARCHAR(64)   | 64           | 是       | 流程实例ID                                                   |          |            |              |
| ACTION_                            | 行为类型。为addcomment时，为处理意见  | VARCHAR(255)  | 255          | 是       | 值为下列内容中的一种：　　　　AddUserLink、DeleteUserLink、AddGroupLink、DeleteGroupLink、AddComment、AddAttachment、DeleteAttachment |          |            |              |
| MESSAGE_                           | 处理意见                              | VARCHAR(4000) | 4000         | 是       | 用于存放流程产生的信息，比如审批意见                         |          |            |              |
| FULL_MSG_                          | 全部消息                              | LONGBLOB      | 是           |          |                                                              |          |            |              |

 

 

### 1.2.7表名:ACT_HI_DETAIL（历史详细信息）

历史详情表：流程中产生的变量详细，包括控制流程流转的变量，业务表单中填写的流程需要用到的变量等。

| **ACT_HI_DETAIL(act_hi_detail)** |            |               |              |                                                |                                                     |          |            |              |
| -------------------------------- | ---------- | ------------- | ------------ | ---------------------------------------------- | --------------------------------------------------- | -------- | ---------- | ------------ |
| **是否主键**                     | **字段名** | **字段描述**  | **数据类型** | **长度**                                       | **可空**                                            | **约束** | **缺省值** | **取值说明** |
| 是                               | ID_        | ID_           | VARCHAR(64)  | 64                                             | 主键                                                |          |            |              |
| TYPE_                            | 数据类型   | VARCHAR(255)  | 255          | 类型:FormProperty, //表单VariableUpdate //参数 |                                                     |          |            |              |
| PROC_INST_ID_                    | 流程实例ID | VARCHAR(64)   | 64           | 是                                             | 流程实例ID                                          |          |            |              |
| EXECUTION_ID_                    | 执行实例Id | VARCHAR(64)   | 64           | 是                                             | 执行实例ID                                          |          |            |              |
| TASK_ID_                         | 任务Id     | VARCHAR(64)   | 64           | 是                                             | 任务实例ID                                          |          |            |              |
| ACT_INST_ID_                     | 活动实例Id | VARCHAR(64)   | 64           | 是                                             | ACT_HI_ACTINST表的ID                                |          |            |              |
| NAME_                            | 名称       | VARCHAR(255)  | 255          | 名称                                           |                                                     |          |            |              |
| VAR_TYPE_                        | 变量类型   | VARCHAR(255)  | 255          | 是                                             | 参见VAR_TYPE_类型说明                               |          |            |              |
| REV_                             | REV_       | INT(11)       | 11           | 是                                             | Version                                             |          |            |              |
| TIME_                            | 创建时间   | DATETIME      | 创建时间     |                                                |                                                     |          |            |              |
| BYTEARRAY_ID_                    | 字节数组Id | VARCHAR(64)   | 64           | 是                                             | ACT_GE_BYTEARRAY表的ID                              |          |            |              |
| DOUBLE_                          | DOUBLE_    | DOUBLE        | 是           | 存储变量类型为Double                           |                                                     |          |            |              |
| LONG_                            | LONG_      | BIGINT(20)    | 20           | 是                                             | 存储变量类型为long                                  |          |            |              |
| TEXT_                            | 值         | VARCHAR(4000) | 4000         | 是                                             | 存储变量值类型为String                              |          |            |              |
| TEXT2_                           | 值2        | VARCHAR(4000) | 4000         | 是                                             | 此处存储的是JPA持久化对象时，才会有值。此值为对象ID |          |            |              |

 

备注：VAR_TYPE_类型说明: jpa-entity、boolean、bytes、serializable(可序列化)、自定义type(根据你自身配置)、 CustomVariableType、date、double、integer、long、null、short、string

 

### 1.2.8  表名:ACT_HI_IDENTITYLINK （历史流程人员表）

任务参与者数据表。主要存储历史节点参与者的信息。

| **ACT_HI_IDENTITYLINK(act_hi_identitylink)** |            |              |              |          |                                                              |          |            |              |
| -------------------------------------------- | ---------- | ------------ | ------------ | -------- | ------------------------------------------------------------ | -------- | ---------- | ------------ |
| **是否主键**                                 | **字段名** | **字段描述** | **数据类型** | **长度** | **可空**                                                     | **约束** | **缺省值** | **取值说明** |
| 是                                           | ID_        | ID_          | VARCHAR(64)  | 64       | ID_                                                          |          |            |              |
| GROUP_ID_                                    | 用户组ID   | VARCHAR(255) | 255          | 是       | 组ID                                                         |          |            |              |
| TYPE_                                        | 用户组类型 | VARCHAR(255) | 255          | 是       | 类型，主要分为以下几种：assignee、candidate、owner、starter 、participant |          |            |              |
| USER_ID_                                     | 用户ID     | VARCHAR(255) | 255          | 是       | 用户ID                                                       |          |            |              |
| TASK_ID_                                     | 任务Id     | VARCHAR(64)  | 64           | 是       | 节点实例ID                                                   |          |            |              |
| PROC_INST_ID_                                | 流程实例Id | VARCHAR(64)  | 64           | 是       | 流程实例ID                                                   |          |            |              |

 

### 1.2.9  表名:ACT_HI_PROCINST（历史流程实例信息）核心表

| **ACT_HI_PROCINST(act_hi_procinst)** |                |               |              |          |          |          |            |          |
| ------------------------------------ | -------------- | ------------- | ------------ | -------- | -------- | -------- | ---------- | -------- |
| **是否主键**                         | **字段名**     | **字段描述**  | **数据类型** | **长度** | **可空** | **约束** | **缺省值** | **备注** |
| 是                                   | ID_            | ID_           | VARCHAR(64)  | 64       |          |          |            |          |
| PROC_INST_ID_                        | 流程实例ID     | VARCHAR(64)   | 64           |          |          |          |            |          |
| BUSINESS_KEY_                        | 业务Key        | VARCHAR(255)  | 255          | 是       |          |          |            |          |
| PROC_DEF_ID_                         | 流程定义Id     | VARCHAR(64)   | 64           |          |          |          |            |          |
| START_TIME_                          | 开始时间       | DATETIME      |              |          |          |          |            |          |
| END_TIME_                            | 结束时间       | DATETIME      | 是           |          |          |          |            |          |
| DURATION_                            | 时长           | BIGINT(20)    | 20           | 是       |          |          |            |          |
| START_USER_ID_                       | 发起人员Id     | VARCHAR(255)  | 255          | 是       |          |          |            |          |
| START_ACT_ID_                        | 开始节点       | VARCHAR(255)  | 255          | 是       |          |          |            |          |
| END_ACT_ID_                          | 结束节点       | VARCHAR(255)  | 255          | 是       |          |          |            |          |
| SUPER_PROCESS_INSTANCE_ID_           | 超级流程实例Id | VARCHAR(64)   | 64           | 是       |          |          |            |          |
| DELETE_REASON_                       | 删除理由       | VARCHAR(4000) | 4000         | 是       |          |          |            |          |

 

### 1.2.10 表名:ACT_HI_TASKINST（历史任务流程实例信息）核心表

| **ACT_HI_TASKINST(act_hi_taskinst)** |                         |               |              |                                      |                                        |          |            |          |
| ------------------------------------ | ----------------------- | ------------- | ------------ | ------------------------------------ | -------------------------------------- | -------- | ---------- | -------- |
| **是否主键**                         | **字段名**              | **字段描述**  | **数据类型** | **长度**                             | **可空**                               | **约束** | **缺省值** | **备注** |
| 是                                   | ID_                     | ID_           | VARCHAR(64)  | 64                                   | 主键ID                                 |          |            |          |
| PROC_DEF_ID_                         | 流程定义Id              | VARCHAR(64)   | 64           | 是                                   | 流程定义ID                             |          |            |          |
| TASK_DEF_KEY_                        | 任务定义Key             | VARCHAR(255)  | 255          | 是                                   | 节点定义ID                             |          |            |          |
| PROC_INST_ID_                        | 流程实例ID              | VARCHAR(64)   | 64           | 是                                   | 流程实例ID                             |          |            |          |
| EXECUTION_ID_                        | 执行ID                  | VARCHAR(64)   | 64           | 是                                   | 执行实例ID                             |          |            |          |
| NAME_                                | 名称                    | VARCHAR(255)  | 255          | 是                                   | 名称                                   |          |            |          |
| PARENT_TASK_ID_                      | 父任务iD                | VARCHAR(64)   | 64           | 是                                   | 父节点实例ID                           |          |            |          |
| DESCRIPTION_                         | 描述                    | VARCHAR(4000) | 4000         | 是                                   | 描述                                   |          |            |          |
| OWNER_                               | 实际签收人 任务的拥有者 | VARCHAR(255)  | 255          | 是                                   | 签收人（默认为空，只有在委托时才有值） |          |            |          |
| ASSIGNEE_                            | 代理人                  | VARCHAR(255)  | 255          | 是                                   | 签收人或被委托                         |          |            |          |
| START_TIME_                          | 开始时间                | DATETIME      | 开始时间     |                                      |                                        |          |            |          |
| CLAIM_TIME_                          | 提醒时间                | DATETIME      | 是           | 提醒时间                             |                                        |          |            |          |
| END_TIME_                            | 结束时间                | DATETIME      | 是           | 结束时间                             |                                        |          |            |          |
| DURATION_                            | 时长                    | BIGINT(20)    | 20           | 是                                   | 耗时                                   |          |            |          |
| DELETE_REASON_                       | 删除理由                | VARCHAR(4000) | 4000         | 是                                   | 删除原因(completed,deleted)            |          |            |          |
| PRIORITY_                            | 优先级                  | INT(11)       | 11           | 是                                   | 优先级别                               |          |            |          |
| DUE_DATE_                            | 应完成时间              | DATETIME      | 是           | 过期时间，表明任务应在多长时间内完成 |                                        |          |            |          |
| FORM_KEY_                            | 表单key                 | VARCHAR(255)  | 255          | 是                                   | desinger节点定义的form_key属性         |          |            |          |

 

### 1.2.11 表名:ACT_HI_VARINST（历史变量信息）

| **ACT_HI_VARINST(act_hi_varinst)** |            |               |              |                          |                                                              |          |            |          |
| ---------------------------------- | ---------- | ------------- | ------------ | ------------------------ | ------------------------------------------------------------ | -------- | ---------- | -------- |
| **是否主键**                       | **字段名** | **字段描述**  | **数据类型** | **长度**                 | **可空**                                                     | **约束** | **缺省值** | **备注** |
| 是                                 | ID_        | ID_           | VARCHAR(64)  | 64                       | ID_                                                          |          |            |          |
| PROC_INST_ID_                      | 流程实例ID | VARCHAR(64)   | 64           | 是                       | 流程实例ID                                                   |          |            |          |
| EXECUTION_ID_                      | 执行ID     | VARCHAR(64)   | 64           | 是                       | 执行实例ID                                                   |          |            |          |
| TASK_ID_                           | 任务Id     | VARCHAR(64)   | 64           | 是                       | 任务实例ID                                                   |          |            |          |
| NAME_                              | 名称       | VARCHAR(255)  | 255          | 参数名称(英文)           |                                                              |          |            |          |
| VAR_TYPE_                          | 变量类型   | VARCHAR(100)  | 100          | 是                       | 参见VAR_TYPE_类型说明                                        |          |            |          |
| REV_                               | REV_       | INT(11)       | 11           | 是                       | Version                                                      |          |            |          |
| BYTEARRAY_ID_                      | 字节数组ID | VARCHAR(64)   | 64           | 是                       | ACT_GE_BYTEARRAY表的主键                                     |          |            |          |
| DOUBLE_                            | DOUBLE_    | DOUBLE        | 是           | 存储DoubleType类型的数据 |                                                              |          |            |          |
| LONG_                              | LONG_      | BIGINT(20)    | 20           | 是                       | 存储LongType类型的数据                                       |          |            |          |
| TEXT_                              | TEXT_      | VARCHAR(4000) | 4000         | 是                       | 存储变量值类型为String，如此处存储持久化对象时，值jpa对象的class |          |            |          |
| TEXT2_                             | TEXT2_     | VARCHAR(4000) | 4000         | 是                       | 此处存储的是JPA持久化对象时，才会有值。此值为对象ID          |          |            |          |

 

### 1.2.12    表名:ACT_ID_GROUP（用户组表）

用来存储用户组信息。

| **ACT_ID_GROUP(act_id_group)** |                |              |              |          |          |          |            |          |
| ------------------------------ | -------------- | ------------ | ------------ | -------- | -------- | -------- | ---------- | -------- |
| **是否主键**                   | **字段名**     | **字段描述** | **数据类型** | **长度** | **可空** | **约束** | **缺省值** | **备注** |
| 是                             | ID_            | 用户组ID     | VARCHAR(64)  | 64       |          |          |            |          |
| REV_                           | 版本号         | INT(11)      | 11           | 是       |          |          |            |          |
| NAME_                          | 用户组描述信息 | VARCHAR(255) | 255          | 是       |          |          |            |          |
| TYPE_                          | 用户组类型     | VARCHAR(255) | 255          | 是       |          |          |            |          |

 

### 1.2.13 表名:ACT_ID_INFO（用户扩展信息表）

用户扩展信息表。目前该表未用到。

| **ACT_ID_INFO(act_id_info)** |               |              |              |          |          |          |            |          |
| ---------------------------- | ------------- | ------------ | ------------ | -------- | -------- | -------- | ---------- | -------- |
| **是否主键**                 | **字段名**    | **字段描述** | **数据类型** | **长度** | **可空** | **约束** | **缺省值** | **备注** |
| 是                           | ID_           | VARCHAR(64)  | 64           |          |          |          |            |          |
| REV_                         | 版本号        | INT(11)      | 11           | 是       |          |          |            |          |
| USER_ID_                     | 用户ID        | VARCHAR(64)  | 64           | 是       |          |          |            |          |
| TYPE_                        | 类型          | VARCHAR(64)  | 64           | 是       |          |          |            |          |
| KEY_                         | formINPut名称 | VARCHAR(255) | 255          | 是       |          |          |            |          |
| VALUE_                       | 值            | VARCHAR(255) | 255          | 是       |          |          |            |          |
| PASSWORD_                    | 密码          | LONGBLOB     | 是           |          |          |          |            |          |
| PARENT_ID_                   | 父节点        | VARCHAR(255) | 255          | 是       |          |          |            |          |

 

### 1.2.14    表名:ACT_ID_MEMBERSHIP（用户用户组关联表）

用来保存用户的分组信息

| **ACT_ID_MEMBERSHIP(act_id_membership)** |            |              |              |          |          |          |            |          |
| ---------------------------------------- | ---------- | ------------ | ------------ | -------- | -------- | -------- | ---------- | -------- |
| **是否主键**                             | **字段名** | **字段描述** | **数据类型** | **长度** | **可空** | **约束** | **缺省值** | **备注** |
| 是                                       | USER_ID_   | 用户Id       | VARCHAR(64)  | 64       |          |          |            |          |
| 是                                       | GROUP_ID_  | 用户组Id     | VARCHAR(64)  | 64       |          |          |            |          |

 

### 1.2.15 表名:ACT_ID_USER（用户信息表）

| **ACT_ID_USER(act_id_user)** |            |              |              |          |          |          |            |          |
| ---------------------------- | ---------- | ------------ | ------------ | -------- | -------- | -------- | ---------- | -------- |
| **是否主键**                 | **字段名** | **字段描述** | **数据类型** | **长度** | **可空** | **约束** | **缺省值** | **备注** |
| 是                           | ID_        | ID_          | VARCHAR(64)  | 64       |          |          |            |          |
| REV_                         | 版本号     | INT(11)      | 11           | 是       |          |          |            |          |
| FIRST_                       | 用户名称   | VARCHAR(255) | 255          | 是       |          |          |            |          |
| LAST_                        | 用户姓氏   | VARCHAR(255) | 255          | 是       |          |          |            |          |
| EMAIL_                       | 邮箱       | VARCHAR(255) | 255          | 是       |          |          |            |          |
| PWD_                         | 密码       | VARCHAR(255) | 255          | 是       |          |          |            |          |
| PICTURE_ID_                  | 头像Id     | VARCHAR(64)  | 64           | 是       |          |          |            |          |

 

 

### 1.2.16 表名:ACT_RE_DEPLOYMENT（部署信息表）

用来存储部署时需要持久化保存下来的信息

| **ACT_RE_DEPLOYMENT(act_re_deployment)** |              |                  |                   |          |          |          |                                                      |          |
| ---------------------------------------- | ------------ | ---------------- | ----------------- | -------- | -------- | -------- | ---------------------------------------------------- | -------- |
| **是否主键**                             | **字段名**   | **字段描述**     | **数据类型**      | **长度** | **可空** | **约束** | **缺省值**                                           | **备注** |
| 是                                       | ID_          | 部署编号，自增长 | VARCHAR(64)       | 64       |          |          |                                                      |          |
| NAME_                                    | 部署包的名称 | VARCHAR(255)     | 255               | 是       |          |          |                                                      |          |
| CATEGORY_                                | 类型         | VARCHAR(255)     | 255               | 是       |          |          |                                                      |          |
|                                          | TENANT_ID_   | 租户             | VARCHAR(255)      | 255      | 是       |          | 多租户通常是在软件需要为多个不同组织服务时产生的概念 |          |
| DEPLOY_TIME_                             | 部署时间     | TIMESTAMP        | CURRENT_TIMESTAMP |          |          |          |                                                      |          |

 

### 1.2.17 表名:ACT_RE_MODEL(流程设计模型表)

  创建流程的设计模型时，保存在该数据表中。

| **ACT_RE_MODEL(act_re_model)** |                                                              |               |              |              |                                                  |          |            |          |
| ------------------------------ | ------------------------------------------------------------ | ------------- | ------------ | ------------ | ------------------------------------------------ | -------- | ---------- | -------- |
| **是否主键**                   | **字段名**                                                   | **字段描述**  | **数据类型** | **长度**     | **可空**                                         | **约束** | **缺省值** | **备注** |
| 是                             | ID_                                                          | **ID_**       | VARCHAR(64)  | 64           | ID_                                              |          |            |          |
| REV_                           | INT(11)                                                      | 11            | 是           | 乐观锁       |                                                  |          |            |          |
| NAME_                          | **模型的名称：****比如：收文管理**                           | VARCHAR(255)  | 255          | 是           | 名称                                             |          |            |          |
| KEY_                           | **模型的关键字，流程引擎用到。****比如：FTOA_SWGL**          | VARCHAR(255)  | 255          | 是           | 分类，例如：http://www.mossle.com/docs/activiti/ |          |            |          |
| CATEGORY_                      | **类型，用户自己对流程模型的分类。**                         | VARCHAR(255)  | 255          | 是           | 分类                                             |          |            |          |
| CREATE_TIME_                   | **创建时间**                                                 | TIMESTAMP     | 是           | 创建时间     |                                                  |          |            |          |
| LAST_UPDATE_TIME_              | **最后修改时间**                                             | TIMESTAMP     | 是           | 最新修改时间 |                                                  |          |            |          |
| VERSION_                       | **版本，从1开始。**                                          | INT(11)       | 11           | 是           | 版本                                             |          |            |          |
| META_INFO_                     | **数据源信息，比如：****{"name":"FTOA_SWGL","revision":1,"description":"丰台财政局OA，收文管理流程"}** | VARCHAR(4000) | 4000         | 是           | 以json格式保存流程定义的信息                     |          |            |          |
| DEPLOYMENT_ID_                 | **部署ID**                                                   | VARCHAR(64)   | 64           | 是           | 部署ID                                           |          |            |          |
| EDITOR_SOURCE_VALUE_ID_        | **编辑源值ID**                                               | VARCHAR(64)   | 64           | 是           | 是 ACT_GE_BYTEARRAY 表中的ID_值。                |          |            |          |
| EDITOR_SOURCE_EXTRA_VALUE_ID_  | **编辑源额外值ID（外键****ACT_GE_BYTEARRAY ）**              | VARCHAR(64)   | 64           | 是           | 是 ACT_GE_BYTEARRAY 表中的ID_值。                |          |            |          |
|                                | TENANT_ID_                                                   | 租户          | VARCHAR(255) | 255          | 是                                               |          |            |          |

 

### 1.2.18 表名:ACT_RE_PROCDEF（流程定义：解析表）

流程解析表，解析成功了，在该表保存一条记录。业务流程定义数据表

| **ACT_RE_PROCDEF(act_re_procdef)** |                                                            |                                                |              |            |                                  |          |          |          |
| ---------------------------------- | ---------------------------------------------------------- | ---------------------------------------------- | ------------ | ---------- | -------------------------------- | -------- | -------- | -------- |
| **是否主键**                       | **字段名**                                                 | **字段描述**                                   | **数据类型** | **长度**   | **可空**                         | **约束** | **缺省** | **备注** |
| 是                                 | ID_                                                        | 流程ID，由“流程编号：流程版本号：自增长ID”组成 | VARCHAR(64)  | 64         | ID_                              |          |          |          |
| REV_                               | 版本号                                                     | INT(11)                                        | 11           | 是         | 乐观锁                           |          |          |          |
| CATEGORY_                          | 流程命名空间（该编号就是流程文件targetNamespace的属性值）  | VARCHAR(255)                                   | 255          | 是         | 流程定义的Namespace就是类别      |          |          |          |
| NAME_                              | 流程名称（该编号就是流程文件process元素的name属性值）      | VARCHAR(255)                                   | 255          | 是         | 名称                             |          |          |          |
| KEY_                               | 流程编号（该编号就是流程文件process元素的id属性值）        | VARCHAR(255)                                   | 255          | 流程定义ID |                                  |          |          |          |
| VERSION_                           | 流程版本号（由程序控制，新增即为1，修改后依次加1来完成的） | INT(11)                                        | 11           | 版本       |                                  |          |          |          |
| DEPLOYMENT_ID_                     | 部署编号                                                   | VARCHAR(64)                                    | 64           | 是         | 部署表ID                         |          |          |          |
| RESOURCE_NAME_                     | 资源文件名称                                               | VARCHAR(4000)                                  | 4000         | 是         | 流程bpmn文件名称                 |          |          |          |
| DGRM_RESOURCE_NAME_                | 图片资源文件名称                                           | VARCHAR(4000)                                  | 4000         | 是         | png流程图片名称                  |          |          |          |
| DESCRIPTION_                       | 描述信息                                                   | VARCHAR(4000)                                  | 4000         | 是         | 描述                             |          |          |          |
| HAS_START_FORM_KEY_                | 是否从key启动                                              | TINYINT(4)                                     | 4            | 是         | start节点是否存在formKey 0否 1是 |          |          |          |
| SUSPENSION_STATE_                  | 是否挂起                                                   | INT(11)                                        | 11           | 是         | 1激活 2挂起                      |          |          |          |

注：此表和ACT_RE_DEPLOYMENT是多对一的关系，即，一个部署的bar包里可能包含多个流程定义文件，每个流程定义文件都会有一条记录在ACT_RE_PROCDEF表内，每个流程定义的数据，都会对于ACT_GE_BYTEARRAY表内的一个资源文件和PNG图片文件。和ACT_GE_BYTEARRAY的关联是通过程序用ACT_GE_BYTEARRAY.NAME与ACT_RE_PROCDEF.NAME_完成的，在[数据库](http://lib.csdn.net/base/mysql)表结构中没有体现。

 

### 1.2.19 表名:ACT_RU_EVENT_SUBSCR(运行时事件)

| **ACT_RU_EVENT_SUBSCR(act_ru_event_subscr)** |            |              |                   |          |          |          |            |          |
| -------------------------------------------- | ---------- | ------------ | ----------------- | -------- | -------- | -------- | ---------- | -------- |
| **是否主键**                                 | **字段名** | **字段描述** | **数据类型**      | **长度** | **可空** | **约束** | **缺省值** | **备注** |
| 是                                           | ID_        | ID           | VARCHAR(64)       | 64       |          |          |            |          |
| REV_                                         | 版本号     | INT(11)      | 11                | 是       |          |          |            |          |
| EVENT_TYPE_                                  | 事件类型   | VARCHAR(255) | 255               |          |          |          |            |          |
| EVENT_NAME_                                  | 事件名称   | VARCHAR(255) | 255               | 是       |          |          |            |          |
| EXECUTION_ID_                                | 流程执行ID | VARCHAR(64)  | 64                | 是       |          |          |            |          |
| PROC_INST_ID_                                | 流程实例ID | VARCHAR(64)  | 64                | 是       |          |          |            |          |
| ACTIVITY_ID_                                 | 活动ID     | VARCHAR(64)  | 64                | 是       |          |          |            |          |
| CONFIGURATION_                               | 配置信息   | VARCHAR(255) | 255               | 是       |          |          |            |          |
| CREATED_                                     | 创建时间   | TIMESTAMP    | CURRENT_TIMESTAMP |          |          |          |            |          |

 

### 1.2.20    表名:ACT_RU_EXECUTION（运行时流程执行实例）

核心，我的代办任务查询表

| **ACT_RU_EXECUTION(act_ru_execution)** |               |              |              |          |                                |          |            |          |
| -------------------------------------- | ------------- | ------------ | ------------ | -------- | ------------------------------ | -------- | ---------- | -------- |
| **是否主键**                           | **字段名**    | **字段描述** | **数据类型** | **长度** | **可空**                       | **约束** | **缺省值** | **备注** |
| 是                                     | ID_           | ID_          | VARCHAR(64)  | 64       | ID_                            |          |            |          |
| REV_                                   | 版本号        | INT(11)      | 11           | 是       | 乐观锁                         |          |            |          |
| PROC_INST_ID_                          | 流程实例编号  | VARCHAR(64)  | 64           | 是       | 流程实例ID                     |          |            |          |
| BUSINESS_KEY_                          | 业务编号      | VARCHAR(255) | 255          | 是       | 业务主键ID                     |          |            |          |
| PARENT_ID_                             | 父执行流程    | VARCHAR(64)  | 64           | 是       | 父节点实例ID                   |          |            |          |
| PROC_DEF_ID_                           | 流程定义Id    | VARCHAR(64)  | 64           | 是       | 流程定义ID                     |          |            |          |
| SUPER_EXEC_                            | VARCHAR(64)   | 64           | 是           |          |                                |          |            |          |
| ACT_ID_                                | 实例id        | VARCHAR(255) | 255          | 是       | 节点实例ID即ACT_HI_ACTINST中ID |          |            |          |
| IS_ACTIVE_                             | 激活状态      | TINYINT(4)   | 4            | 是       | 是否存活                       |          |            |          |
| IS_CONCURRENT_                         | 并发状态      | TINYINT(4)   | 4            | 是       | 是否为并行(true/false）        |          |            |          |
| IS_SCOPE_                              |               | TINYINT(4)   | 4            | 是       |                                |          |            |          |
| IS_EVENT_SCOPE_                        |               | TINYINT(4)   | 4            | 是       |                                |          |            |          |
| SUSPENSION_STATE_                      | 暂停状态_     | INT(11)      | 11           | 是       | 挂起状态  1激活 2挂起          |          |            |          |
| CACHED_ENT_STATE_                      | 缓存结束状态_ | INT(11)      | 11           | 是       |                                |          |            |          |

 

 

### 1.2.21 表名:ACT_RU_IDENTITYLINK（身份联系）

主要存储当前节点参与者的信息,任务参与者数据表。

| **ACT_RU_IDENTITYLINK(act_ru_identitylink)** |            |              |              |          |                                                              |          |            |              |
| -------------------------------------------- | ---------- | ------------ | ------------ | -------- | ------------------------------------------------------------ | -------- | ---------- | ------------ |
| **是否主键**                                 | **字段名** | **字段描述** | **数据类型** | **长度** | **可空**                                                     | **约束** | **缺省值** | **取值说明** |
| 是                                           | ID_        | ID_          | VARCHAR(64)  | 64       |                                                              |          |            |              |
| REV_                                         | 版本号     | INT(11)      | 11           | 是       |                                                              |          |            |              |
| GROUP_ID_                                    | 用户组ＩＤ | VARCHAR(255) | 255          | 是       |                                                              |          |            |              |
| TYPE_                                        | 用户组类型 | VARCHAR(255) | 255          | 是       | 主要分为以下几种：assignee、candidate、owner、starter、participant。即：受让人,候选人,所有者、起动器、参与者 |          |            |              |
| USER_ID_                                     | 用户ID     | VARCHAR(255) | 255          | 是       |                                                              |          |            |              |
| TASK_ID_                                     | 任务Id     | VARCHAR(64)  | 64           | 是       |                                                              |          |            |              |
| PROC_INST_ID_                                | 流程实例ID | VARCHAR(64)  | 64           | 是       |                                                              |          |            |              |
| PROC_DEF_ID_                                 | 流程定义Id | VARCHAR(64)  | 64           | 是       |                                                              |          |            |              |

 

 

### 1.2.22 表名:ACT_RU_JOB（运行中的任务）

运行时定时任务数据表

| **ACT_RU_JOB(act_ru_job)** |                      |               |              |              |            |          |            |              |
| -------------------------- | -------------------- | ------------- | ------------ | ------------ | ---------- | -------- | ---------- | ------------ |
| **是否主键**               | **字段名**           | **字段描述**  | **数据类型** | **长度**     | **可空**   | **约束** | **缺省值** | **取值说明** |
| 是                         | ID_                  | ID_           | VARCHAR(64)  | 64           | 标识       |          |            |              |
| REV_                       | 版本号               | INT(11)       | 11           | 是           | 版本       |          |            |              |
| TYPE_                      | TYPE_                | VARCHAR(255)  | 255          | 类型         |            |          |            |              |
| LOCK_EXP_TIME_             | LOCK_EXP_TIME_       | TIMESTAMP     | 是           | 锁定释放时间 |            |          |            |              |
| LOCK_OWNER_                | LOCK_OWNER_          | VARCHAR(255)  | 255          | 是           | 挂起者     |          |            |              |
| EXCLUSIVE_                 | EXCLUSIVE_           | TINYINT(1)    | 1            | 是           |            |          |            |              |
| EXECUTION_ID_              | EXECUTION_ID_        | VARCHAR(64)   | 64           | 是           | 执行实例ID |          |            |              |
| PROCESS_INSTANCE_ID_       | PROCESS_INSTANCE_ID_ | VARCHAR(64)   | 64           | 是           | 流程实例ID |          |            |              |
| PROC_DEF_ID_               | PROC_DEF_ID_         | VARCHAR(64)   | 64           | 是           | 流程定义ID |          |            |              |
| RETRIES_                   | RETRIES_             | INT(11)       | 11           | 是           |            |          |            |              |
| EXCEPTION_STACK_ID_        | EXCEPTION_STACK_ID_  | VARCHAR(64)   | 64           | 是           | 异常信息ID |          |            |              |
| EXCEPTION_MSG_             | EXCEPTION_MSG_       | VARCHAR(4000) | 4000         | 是           | 异常信息   |          |            |              |
| DUEDATE_                   | DUEDATE_             | TIMESTAMP     | 是           | 到期时间     |            |          |            |              |
| REPEAT_                    | REPEAT_              | VARCHAR(255)  | 255          | 是           | 重复       |          |            |              |
| HANDLER_TYPE_              | HANDLER_TYPE_        | VARCHAR(255)  | 255          | 是           | 处理类型   |          |            |              |
| HANDLER_CFG_               | HANDLER_CFG_         | VARCHAR(4000) | 4000         | 是           | 标识       |          |            |              |

 

 

 

 

### 1.2.23 表名:ACT_RU_TASK(运行时任务数据表)

（执行中实时任务）代办任务查询表

| **ACT_RU_TASK(act_ru_task)** |                                 |               |                             |          |                                                              |          |            |              |
| ---------------------------- | ------------------------------- | ------------- | --------------------------- | -------- | ------------------------------------------------------------ | -------- | ---------- | ------------ |
| **是否主键**                 | **字段名**                      | **字段描述**  | **数据类型**                | **长度** | **可空**                                                     | **约束** | **缺省值** | **取值说明** |
| 是                           | ID_                             | ID_           | VARCHAR(64)                 | 64       | ID_                                                          |          |            |              |
| REV_                         | 版本号                          | INT(11)       | 11                          | 是       | 乐观锁                                                       |          |            |              |
| EXECUTION_ID_                | 实例id（外键EXECUTION_ID_）     | VARCHAR(64)   | 64                          | 是       | 执行实例ID                                                   |          |            |              |
| PROC_INST_ID_                | 流程实例ID（外键PROC_INST_ID_） | VARCHAR(64)   | 64                          | 是       | 流程实例ID                                                   |          |            |              |
| PROC_DEF_ID_                 | 流程定义ID                      | VARCHAR(64)   | 64                          | 是       | 流程定义ID                                                   |          |            |              |
| NAME_                        | 任务名称                        | VARCHAR(255)  | 255                         | 是       | 节点定义名称                                                 |          |            |              |
| PARENT_TASK_ID_              | 父节任务ID                      | VARCHAR(64)   | 64                          | 是       | 父节点实例ID                                                 |          |            |              |
| DESCRIPTION_                 | 任务描述                        | VARCHAR(4000) | 4000                        | 是       | 节点定义描述                                                 |          |            |              |
| TASK_DEF_KEY_                | 任务定义key                     | VARCHAR(255)  | 255                         | 是       | 任务定义的ID                                                 |          |            |              |
| OWNER_                       | 所属人(老板)                    | VARCHAR(255)  | 255                         | 是       | 拥有者（一般情况下为空，只有在委托时才有值）                 |          |            |              |
| ASSIGNEE_                    | 代理人员(受让人)                | VARCHAR(255)  | 255                         | 是       | 签收人或委托人                                               |          |            |              |
| DELEGATION_                  | 代理团                          | VARCHAR(64)   | 64                          | 是       | 委托类型，DelegationState分为两种：PENDING，RESOLVED。如无委托则为空 |          |            |              |
| PRIORITY_                    | 优先权                          | INT(11)       | 11                          | 是       | 优先级别，默认为：50                                         |          |            |              |
| CREATE_TIME_                 | 创建时间                        | TIMESTAMP     | 创建时间，CURRENT_TIMESTAMP |          |                                                              |          |            |              |
| DUE_DATE_                    | 执行时间                        | DATETIME      | 是                          | 耗时     |                                                              |          |            |              |
| SUSPENSION_STATE_            | 暂停状态                        | INT(11)       | 11                          | 是       | 1代表激活 2代表挂起                                          |          |            |              |

 

 

### 1.2.24 表名:ACT_RU_VARIABLE(运行时流程变量数据表)

| **ACT_RU_VARIABLE(act_ru_variable)** |            |               |              |                       |                                                              |          |            |          |
| ------------------------------------ | ---------- | ------------- | ------------ | --------------------- | ------------------------------------------------------------ | -------- | ---------- | -------- |
| **是否主键**                         | **字段名** | **字段描述**  | **数据类型** | **长度**              | **可空**                                                     | **约束** | **缺省值** | **备注** |
| 是                                   | ID_        | ID_           | VARCHAR(64)  | 64                    | 主键标识                                                     |          |            |          |
| REV_                                 | 版本号     | INT(11)       | 11           | 是                    | 乐观锁                                                       |          |            |          |
| TYPE                                 | 编码类型   | VARCHAR(255)  | 255          | 参见VAR_TYPE_类型说明 |                                                              |          |            |          |
| NAME_                                | 变量名称   | VARCHAR(255)  | 255          | 变量名称              |                                                              |          |            |          |
| EXECUTION_ID_                        | 执行实例ID | VARCHAR(64)   | 64           | 是                    | 执行的ID                                                     |          |            |          |
| PROC_INST_ID_                        | 流程实例Id | VARCHAR(64)   | 64           | 是                    | 流程实例ID                                                   |          |            |          |
| TASK_ID_                             | 任务id     | VARCHAR(64)   | 64           | 是                    | 节点实例ID(Local）                                           |          |            |          |
| BYTEARRAY_ID_                        | 字节组ID   | VARCHAR(64)   | 64           | 是                    | 字节表的ID（ACT_GE_BYTEARRAY）                               |          |            |          |
| DOUBLE_                              | DOUBLE_    | DOUBLE        | 是           | 存储变量类型为Double  |                                                              |          |            |          |
| LONG_                                | LONG_      | BIGINT(20)    | 20           | 是                    | 存储变量类型为long                                           |          |            |          |
| TEXT_                                | TEXT_      | VARCHAR(4000) | 4000         | 是                    | 存储变量值类型为String如此处存储持久化对象时，值jpa对象的class |          |            |          |
| TEXT2_                               | TEXT2_     | VARCHAR(4000) | 4000         | 是                    | 此处存储的是JPA持久化对象时，才会有值。此值为对象ID          |          |            |          |

 

 

# 2 Activiti中主要对象的关系

本节主要介绍在工作流中出现的几个对象及其之间的关系，以及在Activiti中各个对象是如何关联的。

在开始之前先看看下图，对整个对象结构有个了解，再结合实例详细介绍理解。

图1.Activiti中几个对象之间的关系

我们模拟一个请假的流程进行分析介绍，该流程主要包含以下几个步骤：

u 员工申请请假

u 部门领导审批

u 人事审批

u 员工销假

 

**ProcessInstance对象**

员工开始申请请假流程，通过runtimeService.startProcessInstance()方法启动，引擎会创建一个流程实例（ProcessInstance）。

简单来说流程实例就是根据一次（一条）业务数据用流程驱动的入口，两者之间是**一对一**的关系。流程引擎会创建一条数据到**ACT_RU_EXECUTION**表，同时也会根据**history**的级别决定是否查询相同的历史数据到**ACT_HI_PROCINST**表。

启动完流程之后业务和流程已经建立了关联关系，第一步结束。

启动流程和业务关联区别：

u 对于**自定义表单**来说启动的时候会传入**businessKey**作为业务和流程的关联属性

u 对于**动态表单**来说不需要使用**businessKey**关联，因为所有的数据都保存在引擎的表中

u 对于**外部表单**来说**businessKey**是可选的，但是一般不会为空，和自定义表单类似

 

**Execution对象**

对于初学者来说，最难理解的地方就是ProcessInstance与Execution之间的关系，要分两种情况说明。Execution的含义就是一个流程实例（ProcessInstance）具体要执行的过程对象。

不过在说明之前先声明两者的对象映射关系：

ProcessInstance（**1**）→ Execution(**N**)，（其中**N**>=1）。

**1) 值相等的情况：**

除了在流程中启动的子流程之外，流程启动之后在表**ACT_RU_EXECUTION**中的字段**ID_**和**PROC_INST_ID**_字段值是相同的。

图2.ID_和PROC_INST_ID_相等

**2) 值不相等的情况：**

不相等的情况目前只会出现在子流程中（包含：嵌套、引入），例如一个购物流程中除了下单、出库节点之外可能还有一个付款子流程，在实际企业应用中付款流程通常是作为公用的，所以使用子流程作为主流程（购物流程）的一部分。

当任务到达子流程时引擎会自动创建一个付款流程，但是这个流程有一个特殊的地方，在数据库可以直观体现，如下图。

图3.ID_和PROC_INST_ID_不相等

上图中有两条数据，第二条数据（嵌入的子流程）的**PARENT_ID_**等于第一条数据的**ID_**和**PROC_INST_ID_**，并且两条数据的**PROC_INST_ID_**相同。

上图中还有一点特殊的地方，字段**IS_ACTIVE_**的值分别是0和1，说明正在执行子流程主流程挂起。

 

**Task对象**

前面说了ProcessInstance和业务是一对一关联的，和业务数据最亲密；而Task则和用户最亲密的（UserTask），用户每天的待办事项就是一个个的Task对象。

从图1中看得出Execution和Task是一对一关系，Task可以是任何类型的Task实现，可以是用户任务（UserTask）、[Java](http://lib.csdn.net/base/java)服务（JavaServiceTask）等，在实际流程运行中只不过面向对象不同，用户任务(UserTask)需要有人为参与完成（complete），Java服务需要由系统自动执行（execution）。

图4. 表ACT_RU_TASK

Task是在流程定义中看到的最大单位，每当一个Task完成的时候引擎会把当前的任务移动到历史中，然后插入下一个任务插入到表**ACT_RU_TASK**中。结合请假流程来说就是让用户点击“完成”按钮提交当前任务是的动作，引擎自动根据任务的顺序流或者排他分支判断走向。

 

**HistoryActivity（历史活动）**

图5. 表ACT_HI_ACTINST

 

Activity包含了流程中所有的活动数据，例如开始事件（图5表中的第1条数据）、各种分支（排他分支、并行分支等，图5表中的第2条数据）、以及刚刚提到的Task执行记录（如图5表中的第3、4条数据）。

有些人认为Activity和Task是多对一关系，其实不是，从上图中可以看出来根本没有Task相关的字段。

结合请假流程来说，如Task中提到的当完成流程的时候所有下一步要执行的任务（包括各种分支）都会创建一个Activity记录到数据库中。例如领导审核节点点击“同意”按钮就会流转到人事审批节点，如果“驳回”那就流转到调整请假内容节点，每一次操作的Task背后实际记录更详细的活动（Activity）。