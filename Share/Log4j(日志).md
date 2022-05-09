# Log4j手册

## 第一章	概述

### 1.1	网址

http://logging.apache.org/log4j/

### 1.2	简介

Log4j是Apache的一个开源Java项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

### 1.3	历史

| 时间      | 事件                                                         |
| --------- | ------------------------------------------------------------ |
| 1996      | 始于E.U. SEMPER （安全电子市场为欧洲）跟踪API的项目。        |
| ...       | Log4j 是基于Java开发的日志框架，其作者`Ceki Gülcü`将Log4j捐献给了Apache软件基金会，使之成为了Apache日志服务的一个子项目。 |
| 2015年9月 | Apache软件基金业宣布，Log4j不在维护，建议所有相关项目升级到Log4j2。 |

### 1.4	特性

- 线程安全
- 支持多输出源
- 支持日志级别
- 配置简单
- 输出格式可控
- 开源协议
- 速度快

### 1.5    缺点

​	日志记录确实也有它的缺点。它可以减缓的应用程序。如果太详细，它可能会导致滚动失明。为了减轻这些影响，log4j被设计为是可靠，快速和可扩展。

## 第二章	使用

### 2.1	Maven依赖

```xml
<!-- 加入log4j支持 -->
<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.17</version>
</dependency>
```

### 2.2	配置文件

```properties
#日志配置
log4j.rootLogger = DEBUG,stdout,file
 
#控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.Threshold=DEBUG
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss}[ %p ]%m%n
 
#所有文件输出
log4j.appender.file = org.apache.log4j.FileAppender
log4j.appender.file.File = D:/logs/log.log
log4j.appender.file.Encoding=UTF-8
log4j.appender.file.name = fileLogDemo
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss} [%c {Num}] [%l] [ %t:%r ] - [ %p ]  %m%n
log4j.appender.file.append = true
```

### 2.3	输出日志

```java
import org.apache.log4j.Logger;

public class log4jTest {
    //获取日志记录器Logger，名字为本类类名
    private static Logger logger = Logger.getLogger(log4jTest.class);
    public static void main(String[] args) {
            // 记录debug级别的信息
            logger.debug("log4j日志输出：This is debug message.");
            // 记录info级别的信息
            logger.info("log4j日志输出：This is info message.");
            // 记录error级别的信息
            logger.error("log4j日志输出：This is error message."); 
    }
}
```

### 2.4	配置说明

log4j包含三个组件,分别是 Logger(记录器)、Appender(输出目的地)、Layout(日志布局)。

可分别简单理解为"日志类别"、"日志要输出的地方"和"日志以何种形式输出"。

#### 2.4.1	配置Logger记录器

```
log4j.rootLogger = [ level ] , appenderName, appenderName, …
```

level表示日志记录的优先级，分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者你定义的级别。

Log4j建议使用四个级别，优先级从高到低分别是ERROR、WARN、INFO、DEBUG。

通过在这里定义的级别，你可以控制到应用程序中相应级别的日志信息的开关。比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。

appenderName就是指日志输出的目的。你可以灵活地定义日志输出，也可以同时指定多个输出目的地。

Log4j配置文件实现了输出到控制台、文件、回滚文件、发送日志邮件、输出到数据库日志表、自定义标签等全套功能。

#### 2.4.2	配置Appender输出目的地

**Appender：日志输出器，配置日志的输出级别、输出位置等，包括以下几类：**

- ConsoleAppender: 日志输出到控制台；
- FileAppender：输出到文件；
- RollingFileAppender：输出到文件，文件达到一定阈值时，自动备份日志文件;
- DailyRollingFileAppender：可定期备份日志文件，默认一天一个文件，也可设置为每分钟一个、每小时一个；
- WriterAppender：可自定义日志输出位置

**配置日志信息输出目的地**：

```
org.apache.log4j.ConsoleAppender（控制台），  
org.apache.log4j.FileAppender（文件），  
org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），  
org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件），  
org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）
```

#### 2.4.3	配置layout日志布局

**配置日志信息的格式**

```
org.apache.log4j.HTMLLayout（HTML表格形式）
org.apache.log4j.SimpleLayout（简单格式的日志，只包括日志信息的级别和指定的信息字符串 ，如:DEBUG - Hello）
org.apache.log4j.TTCCLayout（日志的格式包括日志产生的时间、线程、类别等等信息）
org.apache.log4j.PatternLayout（灵活地自定义日志格式）
```

使用org.apache.log4j.PatternLayout来自定义信息格式时，可以使用ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss} [%c {Num}] [%l] [ %t:%r ] - [ %p ] %m%n 来格式化信息

```
%c 输出所属类的全名，可写为 %c{Num} ,Num类名输出的范围 如："com.sun.aaa.classB",%C{2}将使日志输出输出范围为：aaa.classB
%d 输出日志时间其格式为，一般使用格式 %d{yyyy-MM-dd HH:mm:ss， SSS}
%l 输出日志事件发生位置，包括类目名、发生线程，在代码中的行数
%n 换行符
%m 输出代码指定信息，如info(“message”),输出message
%p 输出日志的优先级，即 (FATAL， ERROR，WARN， INFO，DEBUG or custom)
%r 输出从启动到显示该条日志信息所耗费的时间（毫秒数）
%t 输出产生该日志事件的线程名
%F java 源文件名
%L java 源码行数
%M java 方法名
可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如：  
1)%20c：指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，默认的情况下右对齐。  
2)%-20c:指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，"-"号指定左对齐。  
3)%.30c:指定输出category的名称，最大的宽度是30，如果category的名称大于30的话，就会将左边多出的字符截掉，但小于30的话也不会有空格。  
4)%20.30c:如果category的名称小于20就补空格，并且右对齐，如果其名称长于30字符，就从左边较远输出的字符截掉。
```

## 第三章	架构

### 3.1	组成

Log4j中有三个主要组成部分：

| 组成          | 作用                                   |
| ------------- | -------------------------------------- |
| **loggers**   | 负责捕获记录信息。                     |
| **appenders** | 负责发布日志信息，以不同的首选目的地。 |
| **layouts**   | 负责格式化不同风格的日志信息。         |

### 3.2	对象

#### Logger对象：

顶级层的Logger，它提供Logger对象。Logger对象负责捕获日志信息及它们存储在一个空间的层次结构。

#### Appender对象：

Appender对象负责发布日志信息，以不同的首选目的地，如数据库，文件，控制台，UNIX系统日志等。

#### Level对象：

级别对象定义的任何记录信息的粒度和优先级。有记录的七个级别在API中定义：OFF, DEBUG, INFO, ERROR, WARN, FATAL 和 ALL

#### Filter对象：

过滤对象用于分析日志信息及是否应记录或不用这些信息做出进一步的决定。

​	一个appender对象可以有与之关联的几个Filter对象。如果日志记录信息传递给特定Appender对象，都和特定Appender相关的Filter对象批准的日志信息，然后才能发布到所连接的目的地。

## 第四章	配置

### 4.1	日志级别

每个Logger都被了一个日志级别（log level），用来控制日志信息的输出。日志级别从高到低分为：

| Level | 描述                                               |
| ----- | -------------------------------------------------- |
| OFF   | 最高等级，用于关闭所有日志记录。                   |
| FATAL | 指出每个严重的错误事件将会导致应用程序的退出。     |
| ERROR | 指出虽然发生错误事件，但仍然不影响系统的继续运行。 |
| WARN  | 表明会出现潜在的错误情形。                         |
| INFO  | 一般和在粗粒度级别上，强调应用程序的运行全程。     |
| DEBUG | 一般用于细粒度级别上，对调试应用程序非常有帮助。   |
| ALL   | 最低等级，用于打开所有日志记录。                   |
| TRACE | 指定细粒度比DEBUG更低的信息事件                    |

上面这些级别是定义在org.apache.log4j.Level类中。

Log4j只建议使用4个级别，优先级从低到高分别是DEBUG， INFO， WARN， ERROR。

通过使用日志级别，可以控制应用程序中相应级别日志信息的输出。例如，如果使用b了info级别，则应用程序中所有低于info级别的日志信息(如debug)将不会被打印出来。

## 第五章	源码

### 5.1	核心类

- Logger 用于对日志记录行为的抽象，提供记录不同级别日志的统一接口；

```
public class Logger extends Category
{
// Logger继承Category，Category也是一种日志类
}
```

- Level对日志级别的抽象；

```
public class Level extends Priority implements Serializable
{
// 该类封装一系列日志等级的名字和数字，然后内容封装多个等级的相关枚举
public final static int INFO_INT = 20000;
 
 
private static final String INFO_NAME = "INFO";
	
final static public Level INFO = new Level(INFO_INT, INFO_NAME, 6);
}

```

- Appender是对记录日志形式的抽象，标示了日志打印的目的地；

```
public interface Appender
{
// Appender抽象成了接口，然后主要的实现是WriterAppender，常用的ConsoleAppender，FileAppender都继承了该类。
// 实际编码中经常会遇到DailyRollingFileAppender，RollingFileAppender都继承于FileAppender。
}
```

- Layout是对日志行格式的抽象；

```
public abstract class Layout implements OptionHandler
{
// Layout抽象成一个模板，比较常用的PatternLayout，HTMLLayout都是该类子类
}
```

- LoggingEvent是对一次日志记录过程中所需要的信息的抽象,可以理解成一个上下文；

```
public class LoggingEvent implements java.io.Serializable
{
// 该类定义了一堆堆属性，封装了所有的日志信息。
}
```

+ LoggerRepository是Logger实例的容器

```
public interface LoggerRepository
{
// 常见的Hierarchy就是该接口实现，里面封装了框架一堆默认配置，还有Logger工厂。
// 可以理解该类就是事件源，该类内部封装了以系列的事件
}
```

+ ObjectRender是对日志实例的解析接口，它们主要提供了一种扩展支持。

```
public interface ObjectRenderer
{
 
	/**
	 * @创建时间： 2016年2月25日
	 * @相关参数： @param o
	 * @相关参数： @return
	 * @功能描述： 解析日志对象，默认实现返回toString()
	 */
	public String doRender(Object o);
}

```

　　整个日志打印的过程可以理解为Loger拿着LoggingEvent去找Appender， 让Appender按照Layout的形式将日志打印到指定的位置。 而Level起的啥作用呢？ Logger和Appender都是有原则的不能说你让我打印我就打印，必须满足我的规则我才给你打印，这里的规则就是指Level（出来混都是要讲原则的）。

### 5.2	工作流程

先简述下流程：

入口：Logger log = Logger.getLogger(AppTest.class);
 Logger.getLogger(Class clazz);实际上启动LoggerManager日志管理器，该管理器主要工作是加载log4j配置文件，并进行初始化；
配置文件解析，初始化rootLogger，添加Appender，Layout等配置；
注册logger实例到LoggerRepository，更新层级节点关系；
执行日志记录行为，debug？info？error？
判别级别后，让所有日志输出器执行输出；
输出前会进行过滤和样式渲染；
指定形式输出。


## 附录

### 全面的配置文件

Log4j配置文件实现了输出到控制台、文件、回滚文件、发送日志邮件、输出到数据库日志表、自定义标签等全套功能。

```properties
log4j.rootLogger=DEBUG,console,dailyFile,im
log4j.additivity.org.apache=true
# 控制台(console)
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.ImmediateFlush=true
log4j.appender.console.Target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

# 日志文件(logFile)
log4j.appender.logFile=org.apache.log4j.FileAppender
log4j.appender.logFile.Threshold=DEBUG
log4j.appender.logFile.ImmediateFlush=true
log4j.appender.logFile.Append=true
log4j.appender.logFile.File=D:/logs/log.log4j
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 回滚文件(rollingFile)
log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.Threshold=DEBUG
log4j.appender.rollingFile.ImmediateFlush=true
log4j.appender.rollingFile.Append=true
log4j.appender.rollingFile.File=D:/logs/log.log4j
log4j.appender.rollingFile.MaxFileSize=200KB
log4j.appender.rollingFile.MaxBackupIndex=50
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 定期回滚日志文件(dailyFile)
log4j.appender.dailyFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.dailyFile.Threshold=DEBUG
log4j.appender.dailyFile.ImmediateFlush=true
log4j.appender.dailyFile.Append=true
log4j.appender.dailyFile.File=D:/logs/log.log4j
log4j.appender.dailyFile.DatePattern='.'yyyy-MM-dd
log4j.appender.dailyFile.layout=org.apache.log4j.PatternLayout
log4j.appender.dailyFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于socket
log4j.appender.socket=org.apache.log4j.RollingFileAppender
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.Port=5001
log4j.appender.socket.LocationInfo=true
# Set up for Log Factor 5
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# Log Factor 5 Appender
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000
# 发送日志到指定邮件
log4j.appender.mail=org.apache.log4j.net.SMTPAppender
log4j.appender.mail.Threshold=FATAL
log4j.appender.mail.BufferSize=10
log4j.appender.mail.From = xxx@mail.com
log4j.appender.mail.SMTPHost=mail.com
log4j.appender.mail.Subject=Log4J Message
log4j.appender.mail.To= xxx@mail.com
log4j.appender.mail.layout=org.apache.log4j.PatternLayout
log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于数据库
log4j.appender.database=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.database.URL=jdbc:mysql://localhost:3306/test
log4j.appender.database.driver=com.mysql.jdbc.Driver
log4j.appender.database.user=root
log4j.appender.database.password=
log4j.appender.database.sql=INSERT INTO LOG4J (Message) VALUES('=[%-5p] %d(%r) --> [%t] %l: %m %x %n')
log4j.appender.database.layout=org.apache.log4j.PatternLayout
log4j.appender.database.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

# 自定义Appender
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
log4j.appender.im.host = mail.cybercorlin.net
log4j.appender.im.username = username
log4j.appender.im.password = password
log4j.appender.im.recipient = corlin@cybercorlin.net
log4j.appender.im.layout=org.apache.log4j.PatternLayout
log4j.appender.im.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
```



