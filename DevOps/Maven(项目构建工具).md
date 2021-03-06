# Maven

## 概念

1. Maven是目前市场上最流行的包管理工具、项目构建工具。

2. 通过maven可以管理整个项目从创建、开发到编译、测试、打包、发布的整个流程，进行标准化开发。

   > 特别是通过依赖机制可以优雅的解决项目开发中包的依赖问题，大大简化了项目开发、管理流程。

3. Maven基于项目对象模型（POM）概念，利用==中央信息片段==管理一个项目的构建，生成，报告等等步骤；是目前最主流的==项目构建工具==。

4. maven并不是市场上唯一的项目构建工具，但是是目前最流行的项目构建工具。

   常见的项目管理工具

   | ANT    | 最早的项目构建工具之一，目前很少再使用                       |
   | ------ | ------------------------------------------------------------ |
   | Maven  | 目前最主流的项目构建工具，使用非常广泛                       |
   | Gradle | 目前比较新颖的项目构建工具，相对Maven在进行大项目管理时性能更好 |

## 功能

* 构建
* 文档生成
* 报告
* 依赖
* SCMs
* 发布
* 分发
* 邮件列表

介绍
----

**Maven**是基于项目对象模型（POM），可以通过一小段描述信息来管理项目的构建、报告和文档的软件项目管理工具。

1. bin目录是包含mvn的运行脚本
2. boot目录包含一个类加载器的框架，maven使用它加载自己的类库
3. conf配置文件
4. lib包含maven运行时的依赖类库

* 项目对象模型POM：，xml文件，项目基本信息，描述项目构建过程，声明项目依赖。
* 构件：任何一个依赖、插件或者项目构建输出，都可以称之为构件。
* 插件Plugin：maven是依赖插件完成任务的，项目管理工具。
* 仓库Repository：maven仓库能够帮我们管理构件

环境变量的配置
---------

### Maven下载和安装

下载地址：`http://maven.apache.org/download.cgi`

配置环境变量

- 配置`MAVEN_HOME`环境变量指向maven的安装目录

​        MAVEN_HOME=C:\Program Files\apache-maven-3.5.0-bin\apache-maven-3.5.0

- 配置`PATH`环境变量指向maven安装目录的`bin`目录

​       path=%MAVEN_HOME%\bin

### 配置maven

- maven的核心配置文件是`conf/settings.xml`
- 在正式使用maven之前需要配置这个文件
- 主要需要指定**本地库**和**镜像库**的地址

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

    <!--本地库配置，默认在 ${user.home}/.m2/repository-->
    <!-- localRepository
       | The path to the local repository maven will use to store artifacts.
       |
       | Default: ${user.home}/.m2/repository
      <localRepository>/path/to/local/repo</localRepository>
      -->
    <localRepository>D:/mvn_repo</localRepository>

    <pluginGroups>
    </pluginGroups>

    <proxies>
    </proxies>

    <servers>
    </servers>

    <!--配置镜像库-->
    <mirrors>
        <!--配置阿里镜像库-->   
        <mirror>
            <id>nexus-aliyun</id>
            <mirrorOf>central</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </mirror>
    </mirrors>
   
</settings>
```

### Maven的项目结构 ###

创建项目

> 命令：`mvn archetype:generate`

进入要创建项目的目录，执行命令：`mvn archetype:generate`

提示要求选择创建项目的程序骨架

- 普通Java项目:`maven-archetype-quickstart`

    ```
    7: internal -> org.apache.maven.archetypes:maven-archetype-quickstart (An archet
    ype which contains a sample Maven project.)
    ```

- 普通web项目:`maven-archetype-webapp`

```
10: internal -> org.apache.maven.archetypes:maven-archetype-webapp (An archetype
 which contains a sample Maven Webapp project.)
```

创建出的项目结构

```cmd
|-src
	|-main
		|-java --> 主程序代码
		|-resources --> 主程序的资源文件(主要是配置文件)
	|-test
		|-java -->测试代码
		|-resources -->测试程序的资源文件(主要是配置文件)
|-target-->目标文件夹，src/main和src/test下的资源处理过后产生的文件存放在此处
|-pom.xml -->当前项目的maven核心配置文件
```

常用命令
------

### maven快速创建项目骨架目录  ###

```cmd
mvn archetype:generate 按照提示进行选择
mvn archetype:generate  -DgroupId=com.imooc.maven   -DartifactId=
  maven-service   -Dversion=1.0.0SNAPSHOT   -Dpackage=com.imooc.maven.demo
	1. -DgroupId=组织名，公司网址反写+项目名
	2. -DartifactId=项目名+模块名
	3. -Dversion=版本号
	4. -Dpackage=代码所存在的包名
```

### Maven命令

```cmd
mvn -v 		查看 maven 版本	 
mvn archetype:generate -->创建项目
配置pom文件，声明需要的依赖
mvn compile --->项目编译，此时会自动导入pom文件中声明的依赖jar

while 开发未完成:
	进行开发...
	mvn clean -->项目清理,删除 target
	mvn compile -->项目编译，将写好的代码编译到target
	mvn test -->项目测试
	...

mvn package -->项目打包
mvn install -->安装 jar 包到本地仓库
mvn deploy -->项目发布	 
```

#### 编译

> 命令：`mvn compile`

   会自动导入`pom`文件中指定的依赖。

  会将`src/main`目录下的源码和资源编译后存放到`target/classes`下。

**注意**：`test`文件夹下的所有内容在编译，打包，安装过程都不参加，但是会参加测试过程。

#### 测试

> 命令：`mvn test`

通过此命令可以执行test文件夹下的测试用的内容，实现项目测试

首先会将`src/test`目录下的源码和资源编译后存放到`target/test-classes`下，之后执行其中的测试代码，输出测试结果到控制台，同时测试结果保存一份到`target/surefire-reports`中

#### 清理

> 命令：`mvn clean`

清理mvn命令，此命令可以清除target文件夹

在其他mvn命令执行之前，通常都建议大家先执行一次mvn clean，这样可以将之前其他操作产生的结果清除，防止对本次执行的命令产生影响。

#### 打包

> 命令：`mvn package`

1. 打包命令，会将编译完成的资源打包成指定格式(jar包/war包)

2. 具体怎样打包取决于pom.xml文件中的配置

   ```xml
   <packaging>jar</packaging>
   ```

3. `mvn package`命令同时==隐含了编译 测试 打包过程==

4. 建议在执行此命令之前最好先执行一次`mvn clean`防止之前其他命令产生target内容影响此次命令执行的结果。

#### 安装

> 命令：`mvn install`

1. maven项目安装，这会将打包好的包及其相关的资源文件存放到本地库中由maven进行管理，成为了maven所管理的一个资源。
2. `maven install`命令会隐含进行编译 测试 打包 安装
3. 建议在执行此命令之前最好先执行一次`mvn clean` 防止之前其他命令产生 的target影响此次命令执行的结果。

#### 发布

> 命令：`mvn deploy`

1. maven项目发布，将maven本地库中管理的资源发布到远程库中。
2. 但是无论是中央库还是镜像库都不允许随意上传部署，所以无法在中央库和镜像库中实现这个过程。
3. ==但是如果是自己搭建的私服，是可以通过这个过程完成资源发布的==。

### maven快速创建项目骨架目录  ###

**两种方式：**

```cmd
1.  mvn archetype:generate 按照提示进行选择
2.  mvn archetype:generate  -DgroupId=com.imooc.maven   -DartifactId=
  maven-service   -Dversion=1.0.0SNAPSHOT   -Dpackage=com.imooc.maven.demo
	1. -DgroupId=组织名，公司网址反写+项目名
	2. -DartifactId=项目名+模块名
	3. -Dversion=版本号
	4. -Dpackage=代码所存在的包名
```
Maven中的坐标
--------------

> 在maven库中管理着大量的资源，如何唯一的标识这些资源是一个基本的问题。

maven中是通过资源的坐标地址来解决这个问题的。

```xml
<groupId>org.springframework</groupId>
<artifactId>spring-beans</artifactId>
<version>4.3.7.RLELEASE</version>
```

其中

| **标签**       | **作用**                                     |
| -------------- | -------------------------------------------- |
| `<groupId>`    | 指定项目名称（公司名字+项目名）              |
| `<artifactId>` | 指项目下某一模块名称(项目名+模块名jar包名称) |
| `<version>`    | 指版本信息                                   |

****

## 库（repository）

> maven使用库的概念来管理项目资源。
>
> maven库又分为本地库和远程库，远程库可以细分为中央库、镜像库(代理库)和私服。

1. **中央库**

   指的是maven官方管理维护的库，是全世界最大的maven仓库，管理着大量的资源。

2. **镜像库（代理库）**

   为了分摊中央库的访问压力、为了使全世界不同地区的用户都可以有较好的下载体验，除了中央库，全世界范围内还有多镜像库存在，镜像库可以认为是对中央库全部或部分资源的拷贝，全世界开发者可以选择去连接速度最优的镜像库获取资源。

   ==国内比较知名的maven镜像库有网易镜像库和阿里镜像库==。

3. **私服**

   公司或个人也可以利用maven的机制搭建在一定范围内使用的类似中央库的库，在一定范围内管理项目资源，这样的库只在一定范围内起作用，且不一定和中央库互通，这样的库称之为私服库。

4. **本地库**

   在当前机器内部保存资源的库。



> maven在工作时优先从本地库寻找资源，如果找不到就去从配置的镜像库或私服或中央库中自动下载资源，下载的资源保存在本地库中，以便于重复使用。

maven生命周期
---------

完整的项目构建过程包括：
**清理、编译、测试、打包、集成测试、验证、部署**

**maven三套独立的生命周期**

```cmd
	clean 	清理项目
			1.pre-clean	执行清理前的工作
			2.clean		清理上一次构建生成的所有文件
			3.post-clean 	执行清理后的文件

	default 构建项目（最核心）
			compile test package install

	site 	生成项目站点
			1. pre-site 	在生成项目站点前要完成的工作
			2. site 	生成项目的站点文档
			3. post-site	在生成项目站点后要完成的工作
			4. site-deploy	发布生成的站点到服务器上
```

maven中pom.xml常见元素介绍
---------------
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!--指定了当前pom的版本-->
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tiakon.maven.demo</groupId>
    <artifactId>HoictasStudio-MavenDemo01</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--
        第一个0表示大版本号
        第二个0表示分支版本号
        第三个0表示小版本号
        0.0.1
        snapshot    快照
        alpha       内部测试
        beta        公测
        Release     稳定
        GA          正式发布
    -->
    <!--
        打包方式:默认是jar,可选war、zip、pom
        <packaging></packaging>
    -->

    <!--指定编码格式-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!--
        项目名
        <name></name>
        项目地址
        <url></url>
        项目描述
        <description></description>
        开发人员列表
        <developers></developers>
        许可证信息
        <licenses></licenses>
        组织信息
        <organization></organization>
    -->
```


```xml
<!--依赖列表-->
<dependencies>
    <!--依赖项-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>RELEASE</version>
        <!--<type></type>-->
        <!--依赖范围-->
        <!--<scope></scope>-->
        <!--设置依赖是否可选（默认）false-->
        <!--<optional></optional>-->
        <!--排斥依赖传递列表-->
        <!--
                <exclusions>
                    <exclusion>
                    </exclusion>
                </exclusions>
            -->
    </dependency>
</dependencies>
<!--依赖的管理，作用主要定义在父模块中，对子模块进行管理-->
<!--
        <dependencyManagement>
            <dependencies>

            </dependencies>
        </dependencyManagement>
    -->
<!--对构件的行为提供相应的支持-->
<build>
    <!--插件列表-->
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.0.1</version>

            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>
                            jar-no-fork
                        </goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
<!--通常用于子模块对父模块pom的继承-->
<!--<parent></parent>-->
<!--用来聚合运行Maven项目，指定多个模块一起编译-->
<!--
        <modules>
            <module></module>
        </modules>
    -->

</project>
```
Maven的依赖范围
----------

scope属性

> 在项目中导入的依赖并不一定在项目全生命周期中都要用，此时可以通过scope属性指定依赖应用的范围。

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.8.1</version>
    <scope>test</scope>
</dependency>
```

| 属性值   | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| compile  | ==编译范围==<br />scope的默认值就是compile。  在编译,测试,打包,安装,发布全部生命周期都存在的依赖资源 |
| test     | ==测试范围==  <br />运行测试代码时,才加载的依赖资源,打包,安装,发布都不参加. |
| runtime  | ==运行时范围==<br />和compile的唯一区别,就是不参加编译;  例如JDBC可以不写代码编译,但是必须在运行,打包安装其他阶段参加. |
| provided | ==在编译阶段使用,但是运行,打包安装都不参加==<br />官方给了一个案例:servlet-api,编辑servlet,web应用等代码使用的内容,必须使用provided;(web应用,在tomcat中运行),如果将servlet-api依赖资源打包到war包,扔到tomcat执行会出现冲突;编写任何web应用时,使用到servlet-api的资源,必须添加provided的范围; |
| system   | ==系统范围==<br />  在当前项目的环境中存在需要使用的jar包资源,maven没有提供groupId artifactId version,可以使用system指定从本地路径进行加载  <br />`<dependency>` <br />        `<groupId>cn.tedu</groupId> `<br />       `<artifactId>wotongzhuo</artifactId>`<br />       `<version>1.0</version> ` <br />       `<scope>system</scope> `<br />       `<systemPath>D:\\piaoqian\\xxw\\xxw.jar</systemPath>`<br />  `</dependency> ` |

依赖冲突
------

```cmd
1.短路优先:

	C->B->A->X1(jar)
	C->B->X2(jar)

【C依赖B,B依赖A,A和B都包含同一个不同版本的Jar,则取B的依赖版本。（c的pom.xml中不必注明jar坐标）】

2.先声明先优先

	如果路径相同长度相同，则谁先声明，先解析谁。

【C依赖A和B,A和B都包含同一个不同版本的Jar,谁依赖在前取谁的依赖版本。】
```

exclusions移除依赖传递

> 在项目中导入A依赖，单A本身又依赖B和C,B又依赖的D，则Maven会智能的自动导入B、C和D，这样依赖就被传递了开来，这个过程就称之为依赖的传递。
>
> 依赖传递是一个非常优良的特性，可以省去maven使用过程中的大量配置细节，但这种智能的自动导入过程偶尔也会造成包引入的冲突，造成程序运行出错。此时可以通过配置`exclusions`来打断依赖的传递来解决问题。
>
> 简单来说配置的`exclustions`就是在告诉maven在导入某个包的过程中，指定的依赖传递不要自动导入。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.7.RELEASE</version>
    <exclusions>
        <exclusion>
            <!-- 坐标指定移除内容 -->
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.3.7.RELEASE</version>
        </exclusion>
    </exclusions>
</dependency>
```

聚合与继承
-------

### 聚合 ###

```xml
<packaging>pom</packaging>
<modules>
    <module>../HoictasStudio-MavenDemo01</module>
    <module>../HoictasStudio-MavenDemo02</module>
    <module>../HoictasStudio-MavenDemo03</module>
</modules>
```

假设在**HoictasStudio-MavenParent**模块中添如以上代码，输入`clean install`命令后，即可同时安装多个jar到本地仓库中

```markdown
    [INFO] HoictasStudio-MavenDemo01 .......................... SUCCESS [  4.618 s]
    [INFO] HoictasStudio-MavenDemo02 .......................... SUCCESS [  0.828 s]
    [INFO] HoictasStudio-MavenDemo03 .......................... SUCCESS [  0.923 s]
    [INFO] HoictasStudio-MavenParent .......................... SUCCESS [  0.021 s]
```


### 继承 ###

**根据官方文档说明继承会根据父模块与子模块的包含与否，对pom.xml的写法则有两种。**

#### 第一种写法 ####

假设我们有两个模块，前一个叫 `com.mycompany.app:my-app:1`，后一个叫`com.mycompany.app:my-module:1`。

my-app的pom文件为：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

my-module的pom文件为：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```

我们指定如下项目结构：

```xml
	.
	 |-- my-module
	 |   `-- pom.xml
	 `-- pom.xml
```

那么，我们需要`my-module`去继承`my-app`，则需要在`my-module`的pom文件中添加以下代码：

```xml
	<project>
	  <parent>
	    <groupId>com.mycompany.app</groupId>
	    <artifactId>my-app</artifactId>
	    <version>1</version>
	  </parent>
	  <modelVersion>4.0.0</modelVersion>
	  <groupId>com.mycompany.app</groupId>
	  <artifactId>my-module</artifactId>
	  <version>1</version>
	</project>
```

#### 第二种写法 ####

```cmd
However, that would work if the parent project was already installed inour local repository or was in that specific 
directory structure (parent pom.xml is one directory higher than that of the module's pom.xml). But what if the parent 
is not yet installed and if the directory structure is
.
```
 	 |-- my-module
 	 |   `-- pom.xml
 	 `-- parent
 	     `-- pom.xml	

上一段话摘自官网对继承的介绍，就是说如果你的父模块已在本地安装或者父模块不包含子模块，目录级别甚至是
比子模块的还要高，就在第一种写法上添加`<relativePath>`标签。
​	

```xml
	<project>
		  <parent>
		    <groupId>com.mycompany.app</groupId>
		    <artifactId>my-app</artifactId>
		    <version>1</version>
		    <relativePath>../parent/pom.xml</relativePath>
		  </parent>
		  <modelVersion>4.0.0</modelVersion>
		  <artifactId>my-module</artifactId>
	</project>
```



笔者在看视频时就发现，当父模块与子模块处于同一级别时，在按照视频中的写法（第一种写法）test时就会报错，
而此时的情况是不包含子模块，所以应该在`<parent>`标签中添加`<relativePath>`标签即可测试通过。

## 生命周期

| 阶段	|处理	|描述|
|-|-|-|-|
| 验证 validate	|验证项目	|验证项目是否正确且所有必须信息是可用的|
| 编译 compile	|执行编译	|源代码编译在此阶段完成|
| 测试 Test	|测试	|使用适当的单元测试框架（例如JUnit）运行测试。|
| 包装 package	|打包	|创建JAR/WAR包如在 pom.xml 中定义提及的包|
| 检查 verify	|检查	|对集成测试的结果进行检查，以保证质量达标|
| 安装 install	|安装	|安装打包的项目到本地仓库，以供其他项目使用|
| 部署 deploy	|部署	|拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程|

#### clean生命周期：清理项目，包含三个phase。

1. pre-clean：执行清理前需要完成的工作
2. clean：清理上一次构建生成的文件
3. post-clean：执行清理后需要完成的工作

#### default生命周期：构建项目，重要的phase如下。

1. validate：验证工程是否正确，所有需要的资源是否可用。
1. compile：编译项目的源代码。  
1. test：使用合适的单元测试框架来测试已编译的源代码。这些测试不需要已打包和布署。
1. Package：把已编译的代码打包成可发布的格式，比如jar。
1. integration-test：如有需要，将包处理和发布到一个能够进行集成测试的环境。
1. verify：运行所有检查，验证包是否有效且达到质量标准。
1. install：把包安装到maven本地仓库，可以被其他工程作为依赖来使用。
1. Deploy：在集成或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享。

#### site生命周期：建立和发布项目站点，phase如下

1. pre-site：生成项目站点之前需要完成的工作
2. site：生成项目站点文档
3. post-site：生成项目站点之后需要完成的工作
4. site-deploy：将项目站点发布到服务器

## 插件信息

> 可以额外的增强maven的功能，实现一些特殊效果

1. main插件

   > 用来指定打包出来的jar中的入口方法

   ```xml
   <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-jar-plugin</artifactId>
       <configuration>
           <archive>
               <manifest>
                   <addClasspath>true</addClasspath>
                   <mainClass>cn.tedu.App</mainClass> <!-- 此处为主入口-->
               </manifest>
           </archive>
       </configuration>
   </plugin>
   ```

   

2. 打源码插件

   > 用来额外打包源代码

   ```xml
   <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-source-plugin</artifactId>
       <configuration>
           <attach>true</attach>
       </configuration>
       <executions>
           <execution>
               <phase>compile</phase>
               <goals>
                   <goal>jar</goal>
               </goals>
           </execution>
       </executions>
   </plugin>
   ```

   

常见Maven命令

````
mvn  package -D maven.test.skip=true -D maven.javadoc.skip=true 
````

