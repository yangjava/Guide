# XXL-Job

XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

## Features

- 1、简单：支持通过Web页面对任务进行CRUD操作，操作简单，一分钟上手；
- 2、动态：支持动态修改任务状态、启动/停止任务，以及终止运行中任务，即时生效；
- 3、调度中心HA（中心式）：调度采用中心式设计，“调度中心”自研调度组件并支持集群部署，可保证调度中心HA；
- 4、执行器HA（分布式）：任务分布式执行，任务"执行器"支持集群部署，可保证任务执行HA；
- 5、注册中心: 执行器会周期性自动注册任务, 调度中心将会自动发现注册的任务并触发执行。同时，也支持手动录入执行器地址；
- 6、弹性扩容缩容：一旦有新执行器机器上线或者下线，下次调度时将会重新分配任务；
- 7、触发策略：提供丰富的任务触发策略，包括：Cron触发、固定间隔触发、固定延时触发、API（事件）触发、人工触发、父子任务触发；
- 8、调度过期策略：调度中心错过调度时间的补偿处理策略，包括：忽略、立即补偿触发一次等；
- 9、阻塞处理策略：调度过于密集执行器来不及处理时的处理策略，策略包括：单机串行（默认）、丢弃后续调度、覆盖之前调度；
- 10、任务超时控制：支持自定义任务超时时间，任务运行超时将会主动中断任务；
- 11、任务失败重试：支持自定义任务失败重试次数，当任务失败时将会按照预设的失败重试次数主动进行重试；其中分片任务支持分片粒度的失败重试；
- 12、任务失败告警；默认提供邮件方式失败告警，同时预留扩展接口，可方便的扩展短信、钉钉等告警方式；
- 13、路由策略：执行器集群部署时提供丰富的路由策略，包括：第一个、最后一个、轮询、随机、一致性HASH、最不经常使用、最近最久未使用、故障转移、忙碌转移等；
- 14、分片广播任务：执行器集群部署时，任务路由策略选择"分片广播"情况下，一次任务调度将会广播触发集群中所有执行器执行一次任务，可根据分片参数开发分片任务；
- 15、动态分片：分片广播任务以执行器为维度进行分片，支持动态扩容执行器集群从而动态增加分片数量，协同进行业务处理；在进行大数据量业务操作时可显著提升任务处理能力和速度。
- 16、故障转移：任务路由策略选择"故障转移"情况下，如果执行器集群中某一台机器故障，将会自动Failover切换到一台正常的执行器发送调度请求。
- 17、任务进度监控：支持实时监控任务进度；
- 18、Rolling实时日志：支持在线查看调度结果，并且支持以Rolling方式实时查看执行器输出的完整的执行日志；
- 19、GLUE：提供Web IDE，支持在线开发任务逻辑代码，动态发布，实时编译生效，省略部署上线的过程。支持30个版本的历史版本回溯。
- 20、脚本任务：支持以GLUE模式开发和运行脚本任务，包括Shell、Python、NodeJS、PHP、PowerShell等类型脚本;
- 21、命令行任务：原生提供通用命令行任务Handler（Bean任务，"CommandJobHandler"）；业务方只需要提供命令行即可；
- 22、任务依赖：支持配置子任务依赖，当父任务执行结束且执行成功后将会主动触发一次子任务的执行, 多个子任务用逗号分隔；
- 23、一致性：“调度中心”通过DB锁保证集群分布式调度的一致性, 一次任务调度只会触发一次执行；
- 24、自定义任务参数：支持在线配置调度任务入参，即时生效；
- 25、调度线程池：调度系统多线程触发调度运行，确保调度精确执行，不被堵塞；
- 26、数据加密：调度中心和执行器之间的通讯进行数据加密，提升调度信息安全性；
- 27、邮件报警：任务失败时支持邮件报警，支持配置多邮件地址群发报警邮件；
- 28、推送maven中央仓库: 将会把最新稳定版推送到maven中央仓库, 方便用户接入和使用;
- 29、运行报表：支持实时查看运行数据，如任务数量、调度次数、执行器数量等；以及调度报表，如调度日期分布图，调度成功分布图等；
- 30、全异步：任务调度流程全异步化设计实现，如异步调度、异步运行、异步回调等，有效对密集调度进行流量削峰，理论上支持任意时长任务的运行；
- 31、跨语言：调度中心与执行器提供语言无关的 RESTful API 服务，第三方任意语言可据此对接调度中心或者实现执行器。除此之外，还提供了 “多任务模式”和“httpJobHandler”等其他跨语言方案；
- 32、国际化：调度中心支持国际化设置，提供中文、英文两种可选语言，默认为中文；
- 33、容器化：提供官方docker镜像，并实时更新推送dockerhub，进一步实现产品开箱即用；
- 34、线程池隔离：调度线程池进行隔离拆分，慢任务自动降级进入"Slow"线程池，避免耗尽调度线程，提高系统稳定性；
- 35、用户管理：支持在线管理系统用户，存在管理员、普通用户两种角色；
- 36、权限控制：执行器维度进行权限控制，管理员拥有全量权限，普通用户需要分配执行器权限后才允许相关操作；





# xxl-job

### 系统说明

#### 安装

安装部署参考文档：[分布式任务调度平台xxl-job](http://www.xuxueli.com/xxl-job/#/?id=《分布式任务调度平台xxl-job》)

#### 功能

定时调度、服务解耦、灵活控制跑批时间（停止、开启、重新设定时间、手动触发）

XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用

#### 概念

执行器列表：一个执行器是一个项目

任务：一个任务是一个项目中的 JobHandler

一个xxl-job服务可以有多个执行器（项目），一个项目下可以有多个任务（JobHandler），他们是如何关联的？

**页面操作：**

1. 在管理平台可以新增执行器（项目）
2. 在任务列表可以指定执行器（项目）下新增多个任务（JobHandler）

**代码操作：**

1. 项目配置中增加 xxl.job.executor.appname = "执行器名称"
2. 在实现类中增加 @JobHandler(value="xxl-job-demo") 注解，并继承 IJobHandler

#### 架构图

![img](xxljob.png)

### 抛出疑问

1. 调度中心启动过程？
2. 执行器启动过程？
3. 执行器如何注册到调度中心？
4. 调度中心怎么调用执行器？
5. 集群调度时如何控制一个任务在该时刻不会重复执行
6. 集群部署应该注意什么？

xxljob的源码解析
xxl-job的源码主要分为两大部分，xxl-job-core和xxl-job-admin。其中xxl-job-core是它交互的核心，而xxl-job-admin则是后台管理的一些接口。本文主要分析xxl-job-core设计的核心思想。

设计思想：xxl-job通过在调度中心操作各个执行器的调度方法，使用http作为rpc,调度执行执行器的业务，从而使得业务和调度相对隔离。

### 系统分析

#### 执行器依赖jar包

com.xuxueli:xxl-job-core:2.1.0

com.xuxueli:xxl-registry-client:1.0.2

com.xuxueli:xxl-rpc-core:1.4.1

#### 调度中心启动过程

```java
// 1. 加载 XxlJobAdminConfig，adminConfig = this
XxlJobAdminConfig.java

// 启动过程代码
@Component
public class XxlJobScheduler implements InitializingBean, DisposableBean {
	private static final Logger logger = LoggerFactory.getLogger(XxlJobScheduler.class);

	@Override
	public void afterPropertiesSet() throws Exception {
		// init i18n
		initI18n();

		// admin registry monitor run
        // 2. 启动注册监控器（将注册到register表中的IP加载到group表）/ 30执行一次
		JobRegistryMonitorHelper.getInstance().start();

		// admin monitor run
        // 3. 启动失败日志监控器（失败重试，失败邮件发送）
		JobFailMonitorHelper.getInstance().start();

		// admin-server
        // 4. 初始化RPC服务
		initRpcProvider();

		// start-schedule
        // 5. 启动定时任务调度器（执行任务，缓存任务）
		JobScheduleHelper.getInstance().start();

		logger.info(">>>>>>>>> init xxl-job admin success.");
	}
	
	......
}
```

#### 执行器启动过程

```java
@Override
public void start() throws Exception {

	// init JobHandler Repository
    // 将执行 JobHandler 注册到缓存中 jobHandlerRepository（ConcurrentMap）
	initJobHandlerRepository(applicationContext);

	// refresh GlueFactory
    // 刷新GLUE
	GlueFactory.refreshInstance(1);

	// super start
    // 核心启动项
	super.start();
}

public void start() throws Exception {
    // 初始化日志路径 
    // private static String logBasePath = "/data/applogs/xxl-job/jobhandler";
	XxlJobFileAppender.initLogPath(this.logPath);
    // 初始化注册中心列表 （把注册地址放到 List）
	this.initAdminBizList(this.adminAddresses, this.accessToken);
    // 启动日志文件清理线程 （一天清理一次）
    // 每天清理一次过期日志，配置参数必须大于3才有效
	JobLogFileCleanThread.getInstance().start((long)this.logRetentionDays);
    // 开启触发器回调线程
	TriggerCallbackThread.getInstance().start();
    // 指定端口
	this.port = this.port > 0 ? this.port : NetUtil.findAvailablePort(9999);
    // 指定IP
	this.ip = this.ip != null && this.ip.trim().length() > 0 ? this.ip : IpUtil.getIp();
    // 初始化RPC 将执行器注册到调度中心 30秒一次
	this.initRpcProvider(this.ip, this.port, this.appName, this.accessToken);
}
```

#### 执行器注册到调度中心

执行器

```java
// 注册执行器入口
XxlJobExecutor.java->initRpcProvider()->xxlRpcProviderFactory.start();

// 开启注册
XxlRpcProviderFactory.java->start();

// 执行注册
ExecutorRegistryThread.java->start();
// RPC 注册代码
for (AdminBiz adminBiz: XxlJobExecutor.getAdminBizList()) {
	try {
		ReturnT<String> registryResult = adminBiz.registry(registryParam);
		if (registryResult!=null && ReturnT.SUCCESS_CODE == registryResult.getCode()) {
			registryResult = ReturnT.SUCCESS;
			logger.debug(">>>>>>>>>>> xxl-job registry success, registryParam:{}, registryResult:{}", new Object[]{registryParam, registryResult});
			break;
		} else {
			logger.info(">>>>>>>>>>> xxl-job registry fail, registryParam:{}, registryResult:{}", new Object[]{registryParam, registryResult});
		}
	} catch (Exception e) {
		logger.info(">>>>>>>>>>> xxl-job registry error, registryParam:{}", registryParam, e);
	}

}
```

调度中心

```java
// RPC 注册服务
AdminBizImpl.java->registry();
```

#### 调度中心调用执行器

```java
/* 调度中心执行步骤 */
// 1. 调用执行器
XxlJobTrigger.java->runExecutor();

// 2. 获取执行器
XxlJobScheduler.java->getExecutorBiz();

// 3. 调用
ExecutorBizImpl.java->run();

/* 执行器执行步骤 */
// 1. 执行器接口
ExecutorBiz.java->run();

// 2. 执行器实现
ExecutorBizImpl.java->run();

// 3. 把jobInfo 从 jobThreadRepository (ConcurrentMap) 中获取一个新线程，并开启新线程
XxlJobExecutor.java->registJobThread();

// 4. 保存到当前线程队列
JobThread.java->pushTriggerQueue();

// 5. 执行
JobThread.java->handler.execute(triggerParam.getExecutorParams());
```

### 调度中心（Admin）

实现 org.springframework.beans.factory.InitializingBean类，重写 afterPropertiesSet 方法，在初始化bean的时候都会执行该方法

DisposableBean spring停止时执行

**结束加载项**

1. 停止定时任务调度器（中断scheduleThread，中断ringThread）
2. 停止触发线程池（JobTriggerPoolHelper）
3. 停止注册监控器（registryThread）
4. 停止失败日志监控器（monitorThread）
5. 停止RPC服务（stopRpcProvider）

#### 手动执行方式

JobInfoController.java

```java
@RequestMapping("/trigger")
@ResponseBody
//@PermissionLimit(limit = false)
public ReturnT<String> triggerJob(int id, String executorParam) {
    // force cover job param
    if (executorParam == null) {
        executorParam = "";
    }

    JobTriggerPoolHelper.trigger(id, TriggerTypeEnum.MANUAL, -1, null, executorParam);
    return ReturnT.SUCCESS;
}
```

#### 定时调度策略

调度策略执行图

![img](https://img2018.cnblogs.com/blog/1072053/201909/1072053-20190920094826086-2070926333.png)

调度策略源码

```java
JobScheduleHelper.java->start();
```

#### 路由策略

##### 第一个

固定选择第一个机器

```java
ExecutorRouteFirst.java->route();
```

##### 最后一个

固定选择最后一个机器

```java
ExecutorRouteLast.java->route();
```

##### 轮询

随机选择在线的机器

```java
ExecutorRouteRound.java->route();

private static int count(int jobId) {
	// cache clear
	if (System.currentTimeMillis() > CACHE_VALID_TIME) {
		routeCountEachJob.clear();
		CACHE_VALID_TIME = System.currentTimeMillis() + 1000*60*60*24;
	}

	// count++
	Integer count = routeCountEachJob.get(jobId);
	count = (count==null || count>1000000)?(new Random().nextInt(100)):++count;  // 初始化时主动Random一次，缓解首次压力
	routeCountEachJob.put(jobId, count);
	return count;
}
```

##### 随机

随机获取地址列表中的一个

```java
ExecutorRouteRandom.java->route();
```

##### 一致性HASH

一个job通过hash算法固定使用一台机器，且所有任务均匀散列在不同机器

```java
ExecutorRouteConsistentHash.java->route();

public String hashJob(int jobId, List<String> addressList) {

	// ------A1------A2-------A3------
	// -----------J1------------------
	TreeMap<Long, String> addressRing = new TreeMap<Long, String>();
	for (String address: addressList) {
		for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
			long addressHash = hash("SHARD-" + address + "-NODE-" + i);
			addressRing.put(addressHash, address);
		}
	}

	long jobHash = hash(String.valueOf(jobId));
    // 取出键值 >= jobHash
	SortedMap<Long, String> lastRing = addressRing.tailMap(jobHash);
	if (!lastRing.isEmpty()) {
		return lastRing.get(lastRing.firstKey());
	}
	return addressRing.firstEntry().getValue();
}
```

##### 最不经常使用

使用频率最低的机器优先被选举
把地址列表加入到内存中，等下次执行时剔除无效的地址，判断地址列表中执行次数最少的地址取出
频率、次数

```java
ExecutorRouteLFU.java->route();

public String route(int jobId, List<String> addressList) {

	// cache clear
	if (System.currentTimeMillis() > CACHE_VALID_TIME) {
		jobLfuMap.clear();
		CACHE_VALID_TIME = System.currentTimeMillis() + 1000*60*60*24;
	}

	// lfu item init
	HashMap<String, Integer> lfuItemMap = jobLfuMap.get(jobId);     // Key排序可以用TreeMap+构造入参Compare；Value排序暂时只能通过ArrayList；
	if (lfuItemMap == null) {
		lfuItemMap = new HashMap<String, Integer>();
		jobLfuMap.putIfAbsent(jobId, lfuItemMap);   // 避免重复覆盖
	}

	// put new
	for (String address: addressList) {
		if (!lfuItemMap.containsKey(address) || lfuItemMap.get(address) >1000000 ) {
            // 0-n随机数，包括0不包括n
			lfuItemMap.put(address, new Random().nextInt(addressList.size()));  // 初始化时主动Random一次，缓解首次压力
		}
	}
	// remove old
	List<String> delKeys = new ArrayList<>();
	for (String existKey: lfuItemMap.keySet()) {
		if (!addressList.contains(existKey)) {
			delKeys.add(existKey);
		}
	}
	if (delKeys.size() > 0) {
		for (String delKey: delKeys) {
			lfuItemMap.remove(delKey);
		}
	}
    
    /*********************** 优化 START ***********************/
    // 优化  remove old部分
    Iterator<String> iterable = lfuItemMap.keySet().iterator();
    while (iterable.hasNext()) {
        String address = iterable.next();
        if (!addressList.contains(address)) {
            iterable.remove();
        }
    }
    /*********************** 优化 START ***********************/

	// load least userd count address
    // 从小到大排序
	List<Map.Entry<String, Integer>> lfuItemList = new ArrayList<Map.Entry<String, Integer>>(lfuItemMap.entrySet());
	Collections.sort(lfuItemList, new Comparator<Map.Entry<String, Integer>>() {
		@Override
		public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
			return o1.getValue().compareTo(o2.getValue());
		}
	});

	Map.Entry<String, Integer> addressItem = lfuItemList.get(0);
	String minAddress = addressItem.getKey();
	addressItem.setValue(addressItem.getValue() + 1);

	return addressItem.getKey();
}
```

##### 最近最久未使用

最久未使用的机器优先被选举
用链表的方式存储地址，第一个地址使用后下次该任务过来使用第二个地址，依次类推（PS：有点类似轮询策略）
与轮询策略的区别：

1. 轮询策略是第一次随机找一台机器执行，后续执行会将索引加1取余
2. 轮询策略依赖 addressList 的顺序，如果这个顺序变了，索引到下一次的机器可能不是期望的顺序
3. LRU算法第一次执行会把所有地址加载进来并缓存，从第一个地址开始执行，即使 addressList 地址顺序变了也不影响
   次数

```java
ExecutorRouteLRU.java->route();

public String route(int jobId, List<String> addressList) {

	// cache clear
	if (System.currentTimeMillis() > CACHE_VALID_TIME) {
		jobLRUMap.clear();
		CACHE_VALID_TIME = System.currentTimeMillis() + 1000*60*60*24;
	}

	// init lru
	LinkedHashMap<String, String> lruItem = jobLRUMap.get(jobId);
	if (lruItem == null) {
		/**
		 * LinkedHashMap
		 *      a、accessOrder：ture=访问顺序排序（get/put时排序）；false=插入顺序排期；
		 *      b、removeEldestEntry：新增元素时将会调用，返回true时会删除最老元素；可封装LinkedHashMap并重写该方法，比如定义最大容量，超出是返回true即可实现固定长度的LRU算法；
		 */
		lruItem = new LinkedHashMap<String, String>(16, 0.75f, true);
		jobLRUMap.putIfAbsent(jobId, lruItem);
	}

    /*********************** 举个例子 START ***********************/
    // 如果accessOrder为true的话，则会把访问过的元素放在链表后面，放置顺序是访问的顺序 
	// 如果accessOrder为flase的话，则按插入顺序来遍历
    LinkedHashMap<String, String> lruItem = new LinkedHashMap<String, String>(16, 0.75f, true);
        jobLRUMap.putIfAbsent(1, lruItem);
        lruItem.put("192.168.0.1", "192.168.0.1");
        lruItem.put("192.168.0.2", "192.168.0.2");
        lruItem.put("192.168.0.3", "192.168.0.3");
        String eldestKey = lruItem.entrySet().iterator().next().getKey();
        String eldestValue = lruItem.get(eldestKey);
        System.out.println(eldestValue + ": " + lruItem);
        eldestKey = lruItem.entrySet().iterator().next().getKey();
        eldestValue = lruItem.get(eldestKey);
        System.out.println(eldestValue + ": " + lruItem);
    
    // 输出结果：
    192.168.0.1: {192.168.0.2=192.168.0.2, 192.168.0.3=192.168.0.3, 192.168.0.1=192.168.0.1}
192.168.0.2: {192.168.0.3=192.168.0.3, 192.168.0.1=192.168.0.1, 192.168.0.2=192.168.0.2}
    /*********************** 举个例子 END ***********************/
    
	// put new
	for (String address: addressList) {
		if (!lruItem.containsKey(address)) {
			lruItem.put(address, address);
		}
	}
	// remove old
	List<String> delKeys = new ArrayList<>();
	for (String existKey: lruItem.keySet()) {
		if (!addressList.contains(existKey)) {
			delKeys.add(existKey);
		}
	}
	if (delKeys.size() > 0) {
		for (String delKey: delKeys) {
			lruItem.remove(delKey);
		}
	}

	// load
	String eldestKey = lruItem.entrySet().iterator().next().getKey();
	String eldestValue = lruItem.get(eldestKey);
	return eldestValue;
}
```

##### 故障转移

按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度

```java
ExecutorRouteFailover.java->route();
```

##### 忙碌转移

按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度

```java
ExecutorRouteBusyover.java->route();
```

##### 分片广播

广播触发对应集群中所有机器执行一次任务，同时传递分片参数；可根据分片参数开发分片任务

#### 阻塞处理策略

为了解决执行线程因并发问题、执行效率慢、任务多等原因而做的一种线程处理机制，主要包括 串行、丢弃后续调度、覆盖之前调度，一般常用策略是串行机制

```java
ExecutorBlockStrategyEnum.java

SERIAL_EXECUTION("Serial execution"), // 串行
DISCARD_LATER("Discard Later"), // 丢弃后续调度
COVER_EARLY("Cover Early"); // 覆盖之前调度

ExecutorBizImpl.java->run();

// executor block strategy
if (jobThread != null) {
	ExecutorBlockStrategyEnum blockStrategy = ExecutorBlockStrategyEnum.match(triggerParam.getExecutorBlockStrategy(), null);
	if (ExecutorBlockStrategyEnum.DISCARD_LATER == blockStrategy) {
		// discard when running
		if (jobThread.isRunningOrHasQueue()) {
			return new ReturnT<String>(ReturnT.FAIL_CODE, "block strategy effect："+ExecutorBlockStrategyEnum.DISCARD_LATER.getTitle());
		}
	} else if (ExecutorBlockStrategyEnum.COVER_EARLY == blockStrategy) {
		// kill running jobThread
		if (jobThread.isRunningOrHasQueue()) {
			removeOldReason = "block strategy effect：" + ExecutorBlockStrategyEnum.COVER_EARLY.getTitle();

			jobThread = null;
		}
	} else {
		// just queue trigger
	}
}
```

##### 单机串行

对当前线程不做任何处理，并在当前线程的队列里增加一个执行任务

##### 丢弃后续调度

如果当前线程阻塞，后续任务不再执行，直接返回失败

##### 覆盖之前调度

创建一个移除原因，新建一个线程去执行后续任务

#### 运行模式

```java
ExecutorBizImpl.java->run();
```

##### BEAN

java里的bean对象

##### GLUE(Java)

利用java的反射机制，通过代码字符串生成实体类

```java
IJobHandler originJobHandler = GlueFactory.getInstance().loadNewInstance(triggerParam.getGlueSource());

GroovyClassLoader
```

##### GLUE(Shell Python PHP Nodejs PowerShell)

按照文件命名规则创建一个执行脚本文件和一个日志输出文件，通过脚本执行器执行

#### 失败重试次数

任务失败后记录到 xxl_job_log 中，由失败监控线程查询处理失败的任务且失败次数大于0，继续执行

#### 任务超时时间

把超时时间给 triggerParam 触发参数，在调用执行器的任务时超时时间，有点类似HttpClient的超时时间

### 执行器（Exector）

1. 注册自己的机器地址

2. 注册项目中的 JobHandler

3. 提供被调度中心调用的接口

   ```java
   public interface ExecutorBiz {
   
       /**
        * 供调度中心检测机器是否存活
        *
        * beat
        * @return
        */
       public ReturnT<String> beat();
   
       /**
        * 供调度中心检测机器是否空闲
        *
        * @param jobId
        * @return
        */
       public ReturnT<String> idleBeat(int jobId);
   
       /**
        * kill
        * @param jobId
        * @return
        */
       public ReturnT<String> kill(int jobId);
   
       /**
        * log
        * @param logDateTim
        * @param logId
        * @param fromLineNum
        * @return
        */
       public ReturnT<LogResult> log(long logDateTim, long logId, int fromLineNum);
   
       /**
        * 执行触发器
        * 
        * @param triggerParam
        * @return
        */
       public ReturnT<String> run(TriggerParam triggerParam);
   
   }
   ```

### 总结

#### 学到了什么

1. 算法（LFU、LRU、轮询等）
2. JDK动态代理对象（详细研究）
3. 用到了Netty（详细研究）
4. FutureTask
5. GroovyClassLoader