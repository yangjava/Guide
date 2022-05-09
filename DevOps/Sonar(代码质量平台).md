# sonar

**SonarQube empowers all developers to write cleaner and safer code.**

官网介绍：SonarQube允许所有开发人员编写更干净、更安全的代码。

### 简介

SonarQube是一个用于管理代码质量的开放平台，可以快速的定位代码中潜在的或者明显的错误。目前支持Java,C#,C/C++,Python,PL/SQL,Cobol,JavaScrip,Groovy等二十几种编程语言的代码质量管理与检测。

**主要特点：**

- 代码覆盖：通过单元测试，将会显示哪行代码被选中
- 改善编码规则
- 搜寻编码规则：按照名字，插件，激活级别和类别进行查询
- 项目搜寻：按照项目的名字进行查询
- 对比数据：比较同一张表中的任何测量的趋势　

### 七个维度检测代码质量

复杂度分布(Complexity):代码复杂度过高将难以理解
重复代码(Duplications):程序中包含大量复制、粘贴的代码而导致代码臃肿，sonar可以展示源码中重复严重的地方
单元测试统计(Unit Tests):统计并展示单元测试覆盖率，开发或测试可以清楚测试代码的覆盖情况
代码规则检查(Coding Rules):通过Findbugs,PMD,CheckStyle等检查代码是否符合规范
注释率(Comments):若代码注释过少，特别是人员变动后，其他人接手比较难接手；若过多，又不利于阅读
潜在的Bug(Potential Bugs):通过Findbugs,PMD,CheckStyle等检测潜在的bug
结构与设计(Architecture & Design):找出循环，展示包与包、类与类之间的依赖、检查程序之间耦合度

### 灵活的配置方式

用户本地使用IDE的插件进行代码分析
用户上传到源代码版本控制服务器
持续集成，使用Sonar Scanner进行扫描
将扫描结果上传到SonarQube服务器
SonarQube server将结果写入db
用户通过web ui查看扫描结果
SonarQube导出结果到其他需要的服务

　　Sonar是一个用于代码质量管理的开源平台，用于管理代码的质量，通过插件形式可以支持二十几种语言的代码质量检测，通过多个维度的检查了快速定位代码中潜在的或者明显的错误。

### 2. SonarQube与Sonar

　　SonarQube是sonar的服务端，相当于一个web服务器中的tomcat，用来发布应用，在线浏览分析等。

## 2.安装

### 1.下载sonarqubexxx.zip并且解压即可:

　　下载地址:http://www.sonarqube.org/downloads/

下载完成后解压后点击StartSonar.bat启动即可，如下：

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318103452658-616425909.png)

 

 **http访问9000端口出现下面则证明安装成功。 (如果需要修改端口等信息修改sonarqube-6.7.6\conf\sonar.properties即可)**

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318103550037-1304972033.png)

### 2.配置数据库

\1. 在mysql数据库新建一个库的名称为sonar

\2. 修改sonar/conf/sonar.properties的db信息:

　　不用放置驱动包，也不用创建表。

```
sonar.jdbc.username=root
sonar.jdbc.password=123456
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

 3.重启sonarQube会自动建表。

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318105547688-478220513.png)

 

4.接下来访问9000端口然后进行登录即可。默认创建的用户名和密码都是admin。可以在system选项卡看到系统信息

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318105842769-282731632.png)

##  3.使用

###  1.下载sonar-scanner:(这个工具是对源码进行扫描，并将结果保存到数据库以便用上面的sonarqube进行分析)

　　下载地址:  https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner

###  2.配置mysql信息

 　\sonar\sonar-scanner-3.3.0.1492-windows\conf\sonar-scanner.properties文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#Configure here general information about the environment, such as SonarQube server connection details for example
#No information about specific project should appear here

#----- Default SonarQube server
#sonar.host.url=http://localhost:9000

#----- Default source code encoding
#sonar.sourceEncoding=UTF-8

sonar.jdbc.username=root
sonar.jdbc.password=123456
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 3.配置环境变量 并测试

 　path中增加如下变量:  E:\sonar\sonar-scanner-3.3.0.1492-windows\bin

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
C:\Users\Administrator>sonar-scanner --version
INFO: Scanner configuration file: E:\sonar\sonar-scanner-3.3.0.1492-windows\bin\..\conf\sonar-scanner.properties
INFO: Project root configuration file: NONE
INFO: SonarQube Scanner 3.3.0.1492
INFO: Java 1.8.0_121 Oracle Corporation (64-bit)
INFO: Windows 10 10.0 amd64
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

###  4.打开要进行代码分析的项目根目录，新建sonar-project.properties文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# must be unique in a given SonarQube instance
sonar.projectKey=my:project
# this is the name displayed in the SonarQube UI
sonar.projectName=springboot-ssm
sonar.projectVersion=1.0
 
# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# Since SonarQube 4.2, this property is optional if sonar.modules is set. 
# If not set, SonarQube starts looking for source code from the directory containing 
# the sonar-project.properties file.
sonar.sources=src/main/java
sonar.java.binaries=./target/classes
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 解释：projectName是项目名字，sources是源文件所在的目录;sonar.java.binaries是项目编译后的class文件的目录。

### 5.设置成功后，启动sonarqube服务

### 6.开始分析:

　　cmd窗口进入到项目的根路径，执行下面命令即可:

```
E:\xiangmu\springboot-ssm>sonar-scanner
```

如下:

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318112305133-264354812.png)

### 7.访问9000端口查看分析结果

项目分析概要图:

 ![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318112438058-1559656633.png)

 

 查看存在的bug:

 ![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318112514230-1402261684.png)

### 8.选择一个bug进行查看(可以看到与git也进行了集成，可以看到编写人与时间以及bug的原因)

 ![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318112753644-1699744096.png)

 

　　此工具还分析出一些异常信息的记录等，如下:

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318113107039-1835933964.png)

### 9. 解决下面的bug

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318133523806-1207402709.png)

**代码修改为下面即可:(上面逻辑应该是没有错，只是在多次改变引用的情况下被检测为bug)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    @Override
    public Token addOrUpdateToken(String username) {
        // 1.判断是否有对应的token,如果有的话更新时间，没有的话就创建token并且返回
        Token token = findTokenByUsername(username);
        // 1.1创建token并返回
        if (token == null) {
            return generateAndSaveTokenByUserName(username);
        }

        // 1.2根据失效时间更新且返回token
        return updateTokenByTokenLoseTime(token);
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 10.解决异常处理的bug

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318134247451-189414300.png)

**代码修改为:**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    @Scheduled(fixedRate = 10000)
    public void cron() {
        try {
            Thread.sleep(2000);
            System.out.println("spring anno task execute times " + count++);
        } catch (InterruptedException e) {
            System.err.println("InterruptedException " + e);
            Thread.currentThread().interrupt();
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### 11.程序中故意写一个空指针异常看是否可以检测出来 

```
        String string = null;
        if(string.equals("xxx")){
            System.out.println("xxx");
        }
```

结果:

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318132354834-1749249515.png)

 

###  **更多的学习参考官网:**

　　https://docs.sonarqube.org/display/SCAN/Advanced+SonarQube+Scanner+Usages

  https://www.sonarqube.org/

### 补充:sonarqube汉化

- 到https://docs.sonarqube.org/display/PLUG/Plugin+Library 网站搜索 **chinese pack**

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318121457617-1947216161.png)

 

- **然后点击到对应的git地址https://github.com/SonarQubeCommunity/sonar-l10n-zh/releases下载对应版本的jar包，下载之后放到sonar\sonarqube-6.7.6\extensions\plugins目录下面重启即可，如下:**

　　![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318121610510-2112052728.png)

- **再次通过9000端口访问发现变为中文版:**

![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190318121936490-1704668695.png)

 

## 4. 注册为windows服务　

　　sonar自带的有注册与删除服务的方法，采用wrapper注册为服务，如下已管理员方式运行InstallNTService.bat即可:

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418101530636-1002845058.png)

 

　　注册为服务之后我这里启动服务报错不能正常启动服务，到 %sonar%/logs/sonar.log中查看原因如下:

```
Launching a JVM...
Unable to execute Java command.  系统找不到指定的文件。 (0x2)
```

 

**解决办法：修改%sonar%/conf/wrapper.conf中java的路径为绝对路径**

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418101747143-1125195183.png)

 

补充:sonar-scanner的配置也可以进行分模块配置，比如我想检测一个web项目的所有文件(包括Java、JSP、JS、Html、XML)，如下:

\0. 如果检测html和JSP需要下载sonar-html-plugin-3.1.0.1615.jar插件置于sonarqube-6.7.6\extensions\plugins目录下，而且html的language为web，jsp的language为jsp

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418151551031-1296915628.png)

1.项目结构如下:

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418151051228-1707803460.png)

2.sonar-project.properties配置文件如下:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# must be unique in a given SonarQube instance
sonar.projectKey=my:project
# this is the name displayed in the SonarQube UI
sonar.projectName=sonarTest
sonar.projectVersion=1.0
sonar.modules=java-module,javascript-module,xml-module,jsp-module,html-module
sonar.sourceEncoding=UTF-8

# Java module 
java-module.sonar.projectName=Java Module 
java-module.sonar.language=java 
java-module.sonar.projectBaseDir=.
java-module.sonar.sources=src
#ignore files and directory
java-module.sonar.exclusions=src/cn/qlq/test2/**,src/cn/qlq/test3.java
sonar.java.binaries=./build

# JavaScript module 
javascript-module.sonar.projectName=JavaScript Module 
javascript-module.sonar.projectBaseDir=.
javascript-module.sonar.language=js 
javascript-module.sonar.sources=WebContent

# Jsp module 
jsp-module.sonar.projectName=Jsp Module 
jsp-module.sonar.projectBaseDir=.
jsp-module.sonar.language=jsp
jsp-module.sonar.sources=WebContent

# Html module 
html-module.sonar.projectName=Html Module 
html-module.sonar.projectBaseDir=.
html-module.sonar.language=web
html-module.sonar.sources=WebContent

#Xml module 
xml-module.sonar.projectName=Xml Module 
xml-module.sonar.projectBaseDir=.
xml-module.sonar.language=xml
xml-module.sonar.sources=WebContent
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

3.测试结果从web上访问如下:

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418151351649-923120715.png)

 

补充:sonar集成阿里的p3c规则

1.下载插件 

　　到https://github.com/mrprince/sonar-p3c-pmd/releases 下载jar包即可

2.jar放在sonarqube-6.7.6\extensions\plugins目录下

3.重启sonarqube

4.到网页规则搜索p3c，如下

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418163139307-2100616910.png)

\5. 创建规则，不用选文件，如下：

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418163307373-2013566337.png)

6.接下来激活p3c规则 (选择上面的创建的p3c，同时搜索未激活的p3c，然后激活即可。如果p3c也需要即可sonar的自带规则就不要加搜索条件，选择所有的规则)

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418163456698-1576294876.png)

7.查看p3c激活的规则 (48条)

 ![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418163618099-340003382.png)

\8.  到质量配置设为java默认规则即可

![img](https://img2018.cnblogs.com/blog/1196212/201904/1196212-20190418163713077-240502751.png)

9.简单的测试

　　sonar自带的与p3c规则最明显的区别是:

　　　　 str.equals("xxx")在p3c会被检测，在自带规则不会被检测到。

 　　　System.out.print... 会被自带规则检测到，p3c不会检测到。