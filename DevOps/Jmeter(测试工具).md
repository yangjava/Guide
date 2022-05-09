# Jmeter

## Jmeter概述

### JMeter介绍

   Apache JMeter是100%纯JAVA桌面应用程序，被设计为用于测试客户端/服务端结构的软件(例如web应用程序)。它可以用来测试静态和动态资源的性能，例如：静态文件，Java Servlet,CGI Scripts,Java Object,数据库和FTP服务器等等。JMeter可用于模拟大量负载来测试一台服务器，网络或者对象的健壮性或者分析不同负载下的整体性能。
   同时，JMeter可以帮助你对你的应用程序进行回归测试。通过你创建的测试脚本和assertions来验证你的程序返回了所期待的值。为了更高的适应性，JMeter允许你使用正则表达式来创建这些assertions.

 Jmeter是一款优秀的开源测试工具， 是每个资深测试工程师，必须掌握的测试工具，熟练使用Jmeter能大大提高工作效率。

熟练使用Jmeter后， 能用Jmeter搞定的事情，你就不会使用LoadRunner了。

### 如何学好Jmeter

如果你用Jmeter去对Web进行功能测试，或者性能测试。 你必须熟练HTTP协议，才能学好Jmeter。 否则你很难理解Jmeter中得概念。

### JMeter缺点

　　使用JMeter无法验证JS程序，也无法验证页面UI，所以要须要和Selenium配合来完成Web2.0应用的测试。 

## Jmeter  下载和运行

官方网站：http://jmeter.apache.org/

#### **1）JMeter安装**

1. http://jmeter.apache.org/下载最新版本的JMeter，解压文件到任意目录

2. 安装JDK，配置环境变量JAVA_HOME.

3. 系统要求：JMeter2.11 需要JDK1.6以上的版本支持运行

4. JMeter可以运行在如下操作系统上：Unix，Windows和Open VMS.

5. 避免在一个有空格的路径安装JMeter，这将导致远程测试出现问题。
6. Jmeter 是支持中文的， 启动Jmeter 后， 点击 Options -> Choose Language  来选择语言



#### **2）JMeter插件安装**

1. 插件下载地址： http://jmeter-plugins.org/downloads/all/

2. 插件下载后解压：找到JMeterPlugins-Extras.jar,把JMeterPlugins-Extras.jar放到apache-jmeter-2.12\lib\ext目录。

#### 3）JMeter 运行

1. 进入bin目录运行jmeter.bat启动jmeter

  注意：打开的时候会有两个窗口，JMeter的命令窗口和JMeter的图形操作界面，不可以关闭命令窗口。

2. JMeter的classpath:

  如果你想添加其他JAR文件到JMeter的classpath中，你必须复制他们到lib目录中;

  如果你开发了一个JMeter特定组件或有效的jar文件，复制到lib目录下的ext目录中。

3. 打开之后显示的是中文，如果你想使用其他语言，比如英文，那么通过菜单选项->选择语言->英文即可，当然转为中文也是同样操作。

## 实际测试的例子

目标： 获取城市的天气数据：

**第一步**

 发送request 获取城市的城市代号

http://toy1.weather.com.cn/search?cityname=上海 

从这个请求的response 中获取到上海的城市代码. 比如:

上海的地区代码是101020100

上海动物园的地区代码是:  10102010016A

**第二步**

 发送request 到： http://www.weather.com.cn/weather2d/101020100.shtml  可以得到该城市的天气数据

### 第一步： 新建一个Thread Group

必须新建一个Thread Group,  jmeter的所有任务都必须由线程处理，所有任务都必须在线程组下面创建。

### 第二步：新建一个 HTTP Request

比如我要发送一个Get 方法的http 请求: http://toy1.weather.com.cn/search?cityname=上海 

### 第三步 添加HTTP Head Manager

选中上一步新建的HTTP request. 右键，新建一个Http Header manager. 添加一个header

### 第四步: 添加View Results Tree

View Results Tree 是用来看运行的结果的

### 第五步：运行测试,查看结果

到这里。 我们已经成功运行起来了。

### 第六步:添加Assertion和Assert Results

选择HTTP Request, 右键 Add-> Assertions -> Response Assertion.  添加 Patterns To Test

然后添加一个Assetion Results 用来查看Assertion执行的结果. 

选中Thread Group 右键  Add -> Listener -> Assertion Results. 

运行后， 如果HTTP Response中没有包含期待的字符串。 那么test 就会Fail. 

### 第7步: 使用用户自定义变量

我们还可以在Jmeter中定义变量。 比如我定义一个变量叫 city.   使用它的时候用  ${city}

添加一个 User Defined Variables.  选中Thread Group: 右键 Add -> Config Element -> User Defined Variables.

我们添加一个变量: city

然后在Http Request中使用这个变量

### 第八步：关联

所谓关联， 就是第二个Requst, 使用第一个Request中的数据

我们需要在第一个Http Requst 中新建一个正则表达式，把Response的值提取到变量中，提供给别的Http Request 使用

选择第一个Http Request, 右键 Add -> Post Processors -> Regular Expresstion Extractor

现在新建第二个Http Request,   发送到： http://www.weather.com.cn/weather2d/${citycode}.html 

${citycode} 中的数据， 是从Regular Expression Extractor 中取来的

## JMeter与LoadRunner比较

JMeter 是一款开源(有着典型开源工具特点：**界面不美观**)测试工具，虽然与LoadRunner相比有很多不足，比如：它结果分析能力没有LoadRunner详细；很它的优点也有很多：

-  开源，他是一款开源的免费软件，使用它你不需要支付任何费用，
-  小巧，相比LR的庞大（最新LR11将近4GB），它非常小巧，不需要安装，但需要JDK环境，因为它是使用java开发的工具。
-  功能强大，jmeter设计之初只是一个简单的web性能测试工具，但经过不段的更新扩展，现在可以完成数据库、FTP、LDAP、WebService等方面的测试。因为它的开源性，当然你也可以根据自己的需求扩展它的功能。

 两者最大的区别：jmeter不支持IP欺骗，而LR支持。

## JMeter 测试计划元件

打开Jmeter页面：包括测试计划+工作台。

#### **Test Plan (测试计划)**：

用来描述一个性能测试，包含与本次性能测试所有相关的功能。也就说本的性能测试的所有内容是于基于一个计划的。

右键单击“测试计划”弹出菜单：

**注意**：

“函数测试模式”复选框，如果被选择，它会使Jmeter记录来自服务器返回的每个取样的数据。如果你在测试监听器中选择一个文件，这个数据将被写入文件。如果你尝试一个较小的测试来保证Jmeter配置正确并且你的服务器正在返回期望的结果，这是很有用的。这样做的后果就是这个文件会快速的增大，并且Jmeter的效率会影响。

如果不记录数据到文件，这个选项就没有不同了。

 

#### **Threads （Users）线程 用户**

虽然有三个添加线程组的选项，名字不一样， 创建之后，其界面是完全一样的。之前的版本只有一个线程组的名字。现在多一个setUp theread Group 与terDown Thread Group

1) setup thread group 

一种特殊类型的ThreadGroup的，可用于执行预测试操作。这些线程的行为完全像一个正常的线程组元件。不同的是，这些类型的线程执行测试前进行定期线程组的执行。

setUp Thread Group类似于lr的init.可用于执行预测试操作。

2) teardown thread group. 

一种特殊类型的ThreadGroup的，可用于执行测试后动作。这些线程的行为完全像一个正常的线程组元件。不同的是，这些类型的线程执行测试结束后执行定期的线程组。

tearDown Thread Group类似于lr的end.可用于执行测试后动作。

3) thread group(线程组).

   这个就是我们通常添加运行的线程。通俗的讲一个线程组,，可以看做一个虚拟用户组，线程组中的每个线程都可以理解为一个虚拟用户。线程组中包含的线程数量在测试执行过程中是不会发生改变的。

线程组：

　　名称：就如字面意思，起个有意义的名字就行

　　注释：

　　线程数：这里选择5

　　Ramp-Up Period：单位是秒，默认时间是1秒。它指定了启动所有线程所花费的时间，比如，当前的设定表示“在5秒内启动5个线程，每个线程的间隔时间为1秒”。如果你需要Jmeter立即启动所有线程，将此设定为0即可

　　循环次数：表示每个线程执行多少次请求。

 

#### **测试片段（Test Fragment）**

   测试片段元素是控制器上的一个种特殊的线程组，它在测试树上与线程组处于一个层级。它与线程组有所不同，因为它不被执行，除非它是一个模块控制器或者是被控制器所引用时才会被执行。

控制器

JMeter有两种类型的控制器：取样器（sample）和逻辑控制器（Logic Controller），用这些原件来驱动处理一个测试。

#### **取样器（Sampler）**

  取样器（Sampler）是性能测试中向服务器发送请求，记录响应信息，记录响应时间的最小单元，JMeter 原生支持多种不同的sampler ， 如 HTTP Request Sampler 、 FTP Request Sampler 、TCP Request Sampler 、 JDBC Request Sampler 等，每一种不同类型的 sampler 可以根据设置的参数向服务器发出不同类型的请求。

  在Jmeter的所有Sampler中，Java Request Sampler与BeanShell Requst Sampler是两种特殊的可定制的Sampler.

 

#### **逻辑控制器（Logic Controller）**

  逻辑控制器，包括两类无件，一类是用于控制test plan 中 sampler 节点发送请求的逻辑顺序的控制器，常用的有 如果（If）控制器 、 switch Controller 、Runtime Controller、循环控制器等。另一类是用来组织可控制 sampler 来节点的， 如 事务控制器、吞吐量控制器。

#### **配置元件（Config Element）**

  配置元件（config element）用于提供对静态数据配置的支持。CSV Data Set config 可以将本地数据文件形成数据池 （Data Pool），而对应于HTTP Request Sampler和 TCP Request Sampler等类型的配制无件则可以修改 Sampler的默认数据。

　　例如，HTTP Cookie Manager 可以用于对 HTTP Request Sampler 的 cookie 进行管理。

　　　　　HTTP 请求默认值不会触发Jmeter发送http请求，而只是定义HTTP请求的默认属性。

#### **定时器（Timer）**

  定时器（Timer）用于操作之间设置等待时间，等待时间是性能测试中常用的控制客户端QPS的手段。类似于LoadRunner里面的“思考时间”。 JMeter 定义了Bean Shell Timer、Constant Throughput Timer、固定定时器等不同类型的Timer。

 

#### **前置处理器（Per Processors）**

  前置处理器用于在实际的请求发出之前对即将发出的请求进行特殊处理。例如，HTTP URL重写修复符则可以实现URL重写，当RUL中有sessionID 一类的session信息时，可以通过该处理器填充发出请求的实际的sessionID 。

 

#### **后置处理器（Post Processors）**

  后置处理器是用于对Sampler 发出请求后得到的服务器响应进行处理。一般用来提取响应中的特定数据（类似LoadRunner测试工具中的关联概念）。例如，XPath Extractor 则可以用于提取响应数据中通过给定XPath 值获得的数据;正则表达式提取器，则可以提取响应数据中通过正则表达式获得的数据。

 

#### **断言（Assertions）**

断言用于检查测试中得到的相应数据等是否符合预期，断言一般用来设置检查点，用以保证性能测试过程中的数据交互是否与预期一致。

 

#### **监听器（Listener）**

这个监听器可不是用来监听系统资源的元件。它是用来对测试结果数据进行处理和可视化展示的一系列元件。 图形结果、查看结果树、聚合报告、用表格察看结果都是我们经常用到的元件。

#### **工作台**

在测试中我们可能需要暂时更改一些组件，可以把一些需要更改的组件保存在工作台中，测试完成后再恢复，但是切记：不能退出jmeter.一旦退出jmeter，工作台中的内容就会消失。

**工作台－非测试元件－Property Display，**此元件相当于是jmeter.properties的GUI。

**六、帮助**

http://jmeter.apache.org/usermanual/component_reference.html

最好的帮助是：**菜单－“帮助”－“帮助”。**

到此，我们已经简单了解了jmeter的基本组成原件，我们后序的测试工作也就是使用这些元件来完成测试任务。

参考文档：

聊聊ab、wrk、JMeter、Locust这些压测工具的并发模型差别 https://www.cnblogs.com/ScarecrowAnBird/p/14411611.html

