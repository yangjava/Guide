# IDEA

**JetBrains 破解许可服务器**：https://www.licensez.com

## agent调试

首先把agent打成jar包

然后在主项目中，导入agent。要使用 IntelliJ IDEA 的菜单 File / New / Module 或 File / New / Module from Existing Sources ，保证主项目和 Agent 项目平级。

在VM options里填写：-javaagent: /agent路径/agent.jar

如果有多个参数要填，空格隔开就行了

然后debug模式就可以同时调试主程序和agent了

## 常见设置

### 显示工具条

**效果图**
![显示工具条](F:/lemon-guide-main/images/Others/显示工具条.png)
**设置方法**

- 标注1：View–>Toolbar
- 标注2：View–>Tool Buttons



### 设置鼠标悬浮提示

**效果图**
![设置鼠标悬浮提示](F:/lemon-guide-main/images/Others/设置鼠标悬浮提示.png)

**设置方法**
File–>settings–>Editor–>General–>勾选Show quick documentation…
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030154227362.png)



### 显示方法分隔符

**效果图**
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030154728569.png)
**设置方法**

File–>settings–>Editor–>Appearance–>勾选

![在这里插入图片描述](F:/lemon-guide-main/images/Others/2018103015481187.png)



### 忽略大小写提示

**效果图**
备注：idea的默认设置是严格区分大小写提示的，例如输入string不会提示String，不方便编码
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030155133360.png)
**设置方法**
File–>settings–>Editor–>General -->Code Completion -->
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030155413727.png)



### 主题设置

**效果图**
备注：有黑白两种风格
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030155545483.png)
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030155612301.png)
**设置方法**
File–>settings–>Appearance & Behavior–>Appearance–>
![在这里插入图片描述](F:/lemon-guide-main/images/Others/2018103015572874.png)



### 护眼主题设置

**效果图**
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190110100508868.png)
**设置方法**

如果想将编辑页面变换主题，可以去设置里面调节背景颜色
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190110100622631.png)

如果需要很好看的编码风格，这里有很多主题
http://color-themes.com/?view=index&layout=Generic&order=popular&search=&page=1
点击相应主题，往下滑点击按钮
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190110100734530.png)
下载下来有很多Jar包
![在这里插入图片描述](F:/lemon-guide-main/images/Others/2019011010115881.png)
![在这里插入图片描述](F:/lemon-guide-main/images/Others/2019011010123767.png)

在上面的位置选择导入jar包，然后重启idea生效，重启之后去设置

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190110101907801.png)



### 自动导入包

**效果图**
备注：默认情况是需要手动导入包的，比如我们需要导入Map类，那么需要手动导入，如果不需要使用了，删除了Map的实例，导入的包也需要手动删除，设置了这个功能这个就不需要手动了，自动帮你实现自动导入包和去包，不方便截图，效果请亲测~
**设置方法**
File–>settings–>Editor–>general–>Auto Import–>

![在这里插入图片描述](F:/lemon-guide-main/images/Others/2018103015593523.png)



### 单行显示多个Tabs

**效果图**
默认是显示单排的Tabs:
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030160351154.png)
单行显示多个Tabs:

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030160604417.png)
**设置方法**
File–>settings–>Editor–>General -->Editor Tabs–>去掉√
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030160533499.png)



### 设置字体

**效果图**
备注：默认安装启动Idea字体很小，看着不习惯，需要调整字体大小与字体（有需要可以调整）
**设置方法**
File–>settings–>Editor–>Font–>
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030161017921.png)



### 配置类文档注释信息和方法注释模版

**效果图**
备注：团队开发时方便追究责任与管理查看
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030161142910.png)
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030161254216.png)
**设置方法**
https://blog.csdn.net/zeal9s/article/details/83514565



### 水平或者垂直显示代码

**效果图**
备注：Eclipse如果需要对比代码，只需要拖动Tabs即可，但是idea要设置
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030162041400.png)
**设置方法**
鼠标右击Tabs
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030161922248.png)



### 更换快捷键

**效果图**
备注：从Eclipse移植到idea编码，好多快捷键不一致，导致编写效率降低，现在我们来更换一下快捷键
**设置方法**

- 方法一：

File–>Setting–>
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030165223996.png)

例如设置成Eclipse的，设置好了之后可以ctrl+d删除单行代码（idea是ctrl+y）

- 方法二：设置模板
- File–>Setting–>
  ![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030165549295.png)
- 方法三：

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030165842703.png)
以ctrl+o重写方法为例

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181030170008544.png)



### 注释去掉斜体

**效果图**
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181031135509461.png)
**设置方法**
File–>settings–>Editor–>
![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181031135416445.png)

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20181031135540101.png)



### 代码检测警告提示等级设置

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190316140152621.png)
强烈建议，不要给关掉，不要嫌弃麻烦，他的提示都是对你好，帮助你提高你的代码质量，很有帮助的。



### 项目目录相关–折叠空包

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190316140238852.png)



### 窗口复位

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190316140505814.png)
这个就是当你把窗口忽然间搞得乱七八糟的时候，还可以挽回，就是直接restore一下，就好啦。



### 查看本地代码历史

![在这里插入图片描述](F:/lemon-guide-main/images/Others/20190316140617590.png)



## 常用插件

### .ignore

生成各种ignore文件，一键创建git ignore文件的模板，免得自己去写，如下图。

地址：https://plugins.jetbrains.com/plugin/7495--ignore

![ignore](F:/lemon-guide-main/images/Others/ignore.gif)



### Lombok

支持lombok的各种注解，从此不用写getter setter这些 可以把注解还原为原本的java代码 非常方便。

地址：https://plugins.jetbrains.com/plugin/6317-lombok-plugin

![Lombok](F:/lemon-guide-main/images/Others/Lombok.gif)



### GsonFormat

一键根据json文本生成java类  非常方便。

地址：https://plugins.jetbrains.com/plugin/7654-gsonformat

![GsonFormat](F:/lemon-guide-main/images/Others/GsonFormat.gif)



### GenerateAllSetter

一键调用一个对象的所有set方法并且赋予默认值 在对象字段多的时候非常方便。

地址：https://plugins.jetbrains.com/plugin/9360-generateallsetter

![GenerateAllSetter](F:/lemon-guide-main/images/Others/GenerateAllSetter.gif)



### GenerateSerialVersionUID

`Alt + Insert` 生成`serialVersionUID`

![GenerateSerialVersionUID](F:/lemon-guide-main/images/Others/GenerateSerialVersionUID.gif)



### MyBatisCodeHelper-Pro

MybatisCodeHelperPro 是一款IDEA下全方位支持Mybatis的插件 大部分功能是免费的。

① 接口与xml 互相跳转 高清图标 更改图标 使用快捷键跳转

![MyBatisCodeHelper-Pro](F:/lemon-guide-main/images/Others/MyBatisCodeHelper-Pro.gif)

② 一键添加param注解

![addParamAnnotation](F:/lemon-guide-main/images/Others/addParamAnnotation.gif)



### Maven Helper

一键查看maven依赖，查看冲突的依赖，一键进行exclude依赖，对于大型项目 非常方便。

地址：https://plugins.jetbrains.com/plugin/7179-maven-helper

#### 前言

一般用这款插件来查看[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)的依赖树。在不使用此插件的情况下，要想查看maven的依赖树就要使用Maven命令`maven dependency:tree`来查看依赖。想要查看是否有依赖冲突也可以使用`mvn dependency:tree -Dverbose -Dincludes=<groupId>:<artifactId>`只查看关心的[jar](https://so.csdn.net/so/search?q=jar&spm=1001.2101.3001.7020)包，但是这样还是需要我执行命令，并且当项目比较复杂的时候，这个过程是比较漫长的。maven helper就能很好的解决这个问题。

一旦安装了Maven Helper插件，只要打开pom文件，就可以打开该pom文件的Dependency Analyzer视图（在文件打开之后，文件下面会多出这样一个tab），进入Dependency Analyzer视图之后有三个查看选项，分别是：

- Conflicts(冲突)；
- All Dependencies as List(列表形式查看所有依赖)；
- All Dependencies as Tree(树结构查看所有依赖)。

并且这个页面还支持搜索。很方便！并且使用该插件还能快速的执行maven命令。

#### 安装：

`File-->setting--->Plugins--->`在搜索框中填写Maven Helper然后搜索，单击`Install`按钮进行安装，装完重启IDE。

#### 使用

当Maven Helper 插件安装成功后，打开项目中的pom文件，下面就会多出一个试图

> 切换到此试图即可进行相应操作：
>
> 1. Conflicts（查看冲突）
> 2. All Dependencies as List（列表形式查看所有依赖）
> 3. All Dependencies as Tree（树形式查看所有依赖）

> 当前界面上还提供搜索功能，方便使用。

打开pom文件，并可以切换tab，简单使用，如下图

#### 冲突jar包的解决

切换到maven 依赖视图选择冲突选项，如果有冲突，在左下面区域会有红色显示。

解决冲突，右键单击红色区域，弹出菜单选择Exclude命令，对冲突进行排除。

![Maven-Helper](F:/lemon-guide-main/images/Others/Maven-Helper.gif)



### Rainbow Brackets

代码作色工具（Rainbow Brackets）插件可以实现配对括号相同颜色，并且实现选中区域代码高亮的功能。

- 高亮效果快捷键
  - mac: command+鼠标右键单击
  - windows: ctrl+鼠标右键单击

- 选中部分外暗淡效果快捷键：alt+鼠标右键单击

地址：https://plugins.jetbrains.com/plugin/10080-rainbow-brackets

![Rainbow-Brackets](F:/lemon-guide-main/images/Others/Rainbow-Brackets.gif)

事实上，代码作色之后，可以非常方便我们阅读。类似的工具还有：Grep Console 来自定义设置控制台输出颜色等。



### HighlightBracketPair

自动化高亮显示光标所在代码块对应的括号，可以定制颜色和形状。

![HighlightBracketPair](F:/lemon-guide-main/images/Others/HighlightBracketPair.gif)
![HighlightBracketPair-set](F:/lemon-guide-main/images/Others/HighlightBracketPair-set.jpg)



### CodeGlance

在编辑区的右侧显示的代码地图。

![CodeGlance](F:/lemon-guide-main/images/Others/CodeGlance.png)



### Alibaba Java Coding Guidelines

阿里巴巴出品的java代码规范插件（p3c）。可以扫描整个项目找到不规范的地方 并且大部分可以自动修复。Alibaba Java Code Guidelines 插件实现了开发手册中的的 53 条规则，大部分基于 PMD 实现，其中有 4 条规则基于 IDEA 实现，并且基于 IDEA Inspection 实现了实时检测功能。部分规则实现了 Quick Fix 功能。目前，插件检测有两种模式：实时检测、手动触发。

地址：https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines



### Key promoter X

Key Promoter X 是一个**快捷键提示插件**，如果鼠标操作是能够用快捷键替代，Key Promoter X 会提示可以用什么快捷键替代。详细使用文档。

地址：https://plugins.jetbrains.com/plugin/9792-key-promoter-x

![key-promoter-x](F:/lemon-guide-main/images/Others/key-promoter-x.gif)


### Translation

最好用的翻译插件，功能很强大，界面很漂亮。

地址：https://plugins.jetbrains.com/plugin/8579-translation

![Translation](F:/lemon-guide-main/images/Others/Translation.gif)



### SequenceDiagram

时序图生成工具。有时我们需要梳理业务逻辑或者阅读源码。从中，我们需要了解整个调用链路，反向生成 UML 的时序图是强需求。其中，SequenceDiagram 插件是一个非常棒的插件。

地址：https://plugins.jetbrains.com/plugin/8286-sequencediagram

![SequenceDiagram](F:/lemon-guide-main/images/Others/SequenceDiagram.gif)



### CamelCase

命名风格转换插件，可以在 kebab-case，SNAKE_CASE，PascalCase，camelCase，snake_case 和 空格风格之间切换。快捷键苹果为 **⇧+⌥+ U** ，windows 下为 **Shift + Alt +U**。

![CamelCase](F:/lemon-guide-main/images/Others/CamelCase.gif)



###  MybatisX

**Mybatis-plus** 团队为 **Mybatis** 开发的插件，提供了 **Mapper** 接口和 **XML**之间的跳转和自动生成模版的功能。另外这个名字是我起的，嘿嘿！

![MybatisX](F:/lemon-guide-main/images/Others/MybatisX.gif)





### Git Commit Template

老是有人吐槽你提交的 **Git** 不规范？你可以试试这个插件。它提供了很好的 **Git** 格式化模版，你可以按照实际情况格式化你的提交信息。

![Git-Commit-Template](F:/lemon-guide-main/images/Others/Git-Commit-Template.jpg)



### Extra Icons

最后是一个美化插件，为一些文件类型提供官方没有的图标。来看看效果吧。

![Extra-Icons](F:/lemon-guide-main/images/Others/Extra-Icons.jpg)



### RestfulToolkit

开发时经常会根据 URI 的部分信息来查找对应的Controller 中方法，RestfulToolkit提供了一套 RESTful 服务开发辅助工具集。

地址：https://plugins.jetbrains.com/plugin/10292-restfultoolkit

![RestfulToolkit](F:/lemon-guide-main/images/Others/iuYryuy.jpg)

### Grep Console

### EasyCode

### CheckStyle

### Java Stream Debugger

https://www.e-learn.cn/topic/3624009

### 字节码神器jclasslib

github 地址**：https://github.com/ingokegel/jclasslib**

> **jclasslib bytecode viewer is a tool that visualizes all aspects of compiled Java class files and the contained bytecode.**

jclasslib bytecode viewer 是一个可以可视化已编译Java类文件和所包含的字节码的工具。 另外，它还提供一个库，可以让开发人员读写Java类文件和字节码。

使用时直接选择 View --> Show Bytecode With jclasslib

可以查看基本信息、常量池、接口、属性、函数等信息。

主要优点：

1 不需要使用javap指令，使用简单

2 点击字节码指令可以跳转到 java虚拟机规范对应的章节。

比如我们想了解 putstatic 的含义，可以点击该指令

自动通过浏览器打开虚拟机规范并定位到该指令位置，非常方便。

### API工具

**Restful Fast Request** 是idea版Postman。它是一个强大的restful api工具包插件，可以根据已有的方法帮助您快速生成url和params。 Restful Fast Request = API调试工具 + API管理工具 + API搜索工具。 它有一个漂亮的界面来完成请求、检查服务器响应、存储你的api请求和导出api请求。插件帮助你在IDEA界面内更快更高效得调试你的API。

插件最大的优势是在于集成了开发软件IntelliJ IDEA，开发者只需要一门心思的使用IDEA，摆脱了不停切换软件的烦恼，并且插件是直接跟你开发的API代码挂钩，可以相互跳转。我不需要在postman中看到一个API想调试，但我又要去IDEA中翻阅许久找代码。

## 功能插件

### FindBugs-IDEA

检测代码中可能的bug及不规范的位置，检测的模式相比p3c更多，写完代码后检测下 避免低级bug，强烈建议用一下，一不小心就发现很多老代码的bug。

地址：https://plugins.jetbrains.com/plugin/3847-findbugs-idea

### VisualVM Launcher

运行java程序的时候启动visualvm，方便查看jvm的情况 比如堆内存大小的分配，某个对象占用了多大的内存，jvm调优必备工具。

地址：https://plugins.jetbrains.com/plugin/7115-visualvm-launcher



### MyBatis Log Plugin

Mybatis现在是java中操作数据库的首选，在开发的时候，我们都会把Mybatis的脚本直接输出在console中，但是默认的情况下，输出的脚本不是一个可以直接执行的。如果我们想直接执行，还需要在手动转化一下。MyBatis Log Plugin 这款插件是直接将Mybatis执行的sql脚本显示出来，无需处理，可以直接复制出来执行的，如图：

![img](F:/lemon-guide-main/images/Others/v2-2e63a1d6ddf2782ab8a0cda4d3e41502_hd.jpg)



### Activate-Power-Mode

根据Atom的插件activate-power-mode（或Power mode II）的效果移植到IDEA上。

![8](F:/lemon-guide-main/images/Others/intellij-idea-zhuangbi-top-5-5.gif)



### Background Image Plus

可设置idea背景图片的插件，不但可以设置固体的图片，还可以设置一段时间后随机变化背景图片，以及设置图片的透明度等等。

![img](F:/lemon-guide-main/images/Others/772743-20181027200424736-854569575.png)

 

### Material Theme UI

这是一款主题插件，可以让你的ide的图标变漂亮，配色搭配的很到位，还可以切换不同的颜色，甚至可以自定义颜色。默认的配色就很漂亮了，如果需要修改配色，可以在工具栏中Tools->Material Theme然后修改配色等。

![tools](F:/lemon-guide-main/images/Others/material-theme-ui-tools.png)

![](F:/lemon-guide-main/images/Others/oceanic.png)



## 集成插件

### 集成JIRA

Jira是一个广泛使用的项目与事务跟踪工具，被广泛应用于缺陷跟踪、客户服务、需求收集、流程审批、任务跟踪、项目跟踪和敏捷管理等工作领域。idea可以很好的跟它集成，参考下图：

File -> Settings ->Task -> Servers 点击右侧上面的+号，选择JIRA，然后输入JIRA的Server地址，用户名、密码即可

![IDEA-JIRA-1](F:/lemon-guide-main/images/Others/IDEA-JIRA-1.png)

然后打开Open Task界面

![IDEA-JIRA-2](F:/lemon-guide-main/images/Others/IDEA-JIRA-2.png)

如果JIRA中有分配给你的Task，idea能自动列出来

![IDEA-JIRA-3](F:/lemon-guide-main/images/Others/IDEA-JIRA-3.png)

代码修改后，向svn提交时，会自动与该任务关联

![IDEA-JIRA-4](F:/lemon-guide-main/images/Others/IDEA-JIRA-4.png)

将每次提交的代码修改与JIRA上的TASK关联后，有什么好处呢？我们每天可能要写很多代码，修复若干bug，日子久了以后，谁也不记得当初为了修复某个bug做了哪些修改，只要你按上面的操作正确提交，idea都会帮你记着这些细节：

![IDEA-JIRA-5](F:/lemon-guide-main/images/Others/IDEA-JIRA-5.png)

如上图，选择最近提交的TASK列表，选择Switch to，idea就会自动打开该TASK关联的源代码，并定位到修改过的代码行。当然如果该TASK已经Close了，也可以选择Remove将其清空。



### UML类图插件

idea已经集成了该功能，只是默认没打开，仍然打开Settings界面，定位到Plugins，输入UML，参考下图：

![img](F:/lemon-guide-main/images/Others/IDEA-UML-1.png)

 确认UML 这个勾已经勾上了，然后点击Apply，重启idea，然后仍然找一个java类文件，右击Diagram

![img](F:/lemon-guide-main/images/Others/IDEA-UML-2.png)

然后，就自个儿爽去吧

![img](F:/lemon-guide-main/images/Others/IDEA-UML-3.png)



### 集成SSH

java项目经常会在linux上部署，每次要切换到SecureCRT这类终端工具未免太麻烦，idea也想到了这一点:

![img](F:/lemon-guide-main/images/Others/IDEA-SSH-1.png)

然后填入IP、用户名、密码啥的

![img](F:/lemon-guide-main/images/Others/IDEA-SSH-2.png)

点击OK，就能连接上linux了

![img](F:/lemon-guide-main/images/Others/IDEA-SSH-3.png)

注：如果有中文乱码问题，可以在Settings里调整编码为utf-8

![img](F:/lemon-guide-main/images/Others/IDEA-SSH-4.png)

 

### 集成FTP

![img](F:/lemon-guide-main/images/Others/IDEA-FTP-1.png)

点击上图中的...，添加一个Remote Host

![img](F:/lemon-guide-main/images/Others/IDEA-FTP-2.png)

填写ftp的IP、用户名、密码，根路径啥的，然后点击Test FTP Connection，正常的话，应该能连接，如果连接不通，点击Advanced Options，参考下图调整下连接选项

![img](F:/lemon-guide-main/images/Others/IDEA-FTP-3.png)

配置了FTP连接后，在提交代码时，可以选择提交完成后将代码自动上传到ftp服务器

![img](F:/lemon-guide-main/images/Others/IDEA-FTP-4.png)

 

### Database管理工具

先看效果吧：

![img](F:/lemon-guide-main/images/Others/IDEA-Database-1.png)

有了这个，再也不羡慕vs.net的db管理功能了。配置也很简单，就是点击+号，增加一个Data Source即可

![img](F:/lemon-guide-main/images/Others/IDEA-Database-2.png)

唯一要注意的是，intellij idea不带数据库驱动，所以在上图中，要手动指定db driver的jar包路径。



## Debug

### 弹框/显示

勾选Show debug window on breakpoint，则请求进入到断点后自动激活Debug窗口：

![img](F:/lemon-guide-main/images/Others/856154-20170905111655647-1134637623.png)

如果IDEA底部没有显示工具栏或状态栏，可以在View里打开，显示出工具栏会方便我们使用：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905112617351-1554043487.png)



### 快捷键

在菜单栏Run里有调试对应的功能，同时可以查看对应的快捷键：　　![img](F:/lemon-guide-main/images/Others/856154-20170905124338444-556465721.png)

 首先说第一组按钮，共8个按钮，从左到右依次如下：

　　　　![img](F:/lemon-guide-main/images/Others/856154-20170905134837851-1615718043.png)

- **Show Execution Point (Alt + F10)**：如果你的光标在其它行或其它页面，点击这个按钮可跳转到当前代码执行的行
- **Step Over (F8)**：步过，一行一行地往下走，如果这一行上有方法不会进入方法
- **Step Into (F7)**：步入，如果当前行有方法，可以进入方法内部，一般用于进入自定义方法内
- **Force Step Into (Alt + Shift + F7)**：强制步入，能进入任何方法，查看底层源码的时候可以用这个进入官方类库的方法
- **Step Out (Shift + F8)**：步出，从步入的方法内退出到方法调用处，此时方法已执行完毕，只是还没有完成赋值
- **Drop Frame (默认无)**：回退断点，后面章节详细说明
- **Run to Cursor (Alt + F9)**：运行到光标处，你可以将光标定位到你需要查看的那一行，然后使用这个功能，代码会运行至光标行，而不需要打断点
- **Evaluate Expression (Alt + F8)**：计算表达式，后面章节详细说明

第二组按钮，共7个按钮，从上到下依次如下：

 　　　　![img](F:/lemon-guide-main/images/Others/856154-20170905134011101-1824595229.png) 

- **Rerun 'xxxx'**：重新运行程序，会关闭服务后重新启动程序

- **Update 'tech' application (Ctrl + F5)**：更新程序，一般在你的代码有改动后可执行这个功能
- **Resume Program (F9)**：恢复程序，比如，你在第20行和25行有两个断点，当前运行至第20行，按F9，则运行到下一个断点(即第25行)，再按F9，则运行完整个流程，因为后面已经没有断点了
- **Pause Program**：暂停程序，启用Debug。目前没发现具体用法
- **Stop 'xxx' (Ctrl + F2)**：连续按两下，关闭程序。有时候你会发现关闭服务再启动时，报端口被占用，这是因为没完全关闭服务的原因，你就需要查杀所有JVM进程了
- **View Breakpoints (Ctrl + Shift + F8)**：查看所有断点，后面章节会涉及到
- **Mute Breakpoints**：哑的断点，选择这个后，所有断点变为灰色，断点失效，按F9则可以直接运行完程序。再次点击，断点变为红色，有效


### 变量查看

在IDEA中，参数所在行后面会显示当前变量的值：　　![img](F:/lemon-guide-main/images/Others/856154-20170905154209179-9123997.png) 

光标悬停到参数上，显示当前变量信息：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905154425772-770303651.png)

![img](F:/lemon-guide-main/images/Others/856154-20170905154724866-160919363.png) 

在Variables里查看，这里显示当前方法里的所有变量：

 　　![img](F:/lemon-guide-main/images/Others/856154-20170905155339491-1166069157.png)

在Watches里，点击New Watch，输入需要查看的变量。或者可以从Variables里拖到Watche里查看：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905160057038-750351531.png)

如果你发现你没有Watches，可能在下图所在的地方：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905160433710-2004658473.png)

![img](F:/lemon-guide-main/images/Others/856154-20170905160515538-1647769062.png) 



### 计算表达式

　　![img](F:/lemon-guide-main/images/Others/856154-20170905160826444-1625048711.png)

![img](F:/lemon-guide-main/images/Others/856154-20170905161614694-93470669.png) 

设置变量，在计算表达式的框里，可以改变变量的值，这样有时候就能很方便我们去调试各种值的情况了不是：

![img](F:/lemon-guide-main/images/Others/856154-20170905162404288-824548249.png) 



### 智能步入

一行代码里有好几个方法，怎么只选择某一个方法进入。之前提到过使用Step Into (Alt + F7) 或者 Force Step Into (Alt + Shift + F7)进入到方法内部，但这两个操作会根据方法调用顺序依次进入，这比较麻烦。

　　那么智能步入就很方便了，智能步入，这个功能在Run里可以看到，Smart Step Into (Shift + F7)：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905152523304-803289488.png)

　　按Shift + F7，会自动定位到当前断点行，并列出需要进入的方法，如图5.2，点击方法进入方法内部。

　如果只有一个方法，则直接进入，类似Force Step Into。

　　![img](F:/lemon-guide-main/images/Others/856154-20170905163730929-1374653206.png) [图5.2]



### 断点条件设置

通过设置断点条件，在满足条件时，才停在断点处，否则直接运行：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905165253944-1162138475.png)  [图6.1]

点击View Breakpoints (Ctrl + Shift + F8)，查看所有断点。Java Line Breakpoints 显示了所有的断点，在右边勾选Condition，设置断点的条件。

- 勾选Log message to console，则会将当前断点行输出到控制台
- 勾选Evaluate and log，可以在执行这行代码是计算表达式的值，并将结果输出到控制台

![img](F:/lemon-guide-main/images/Others/856154-20170905170655163-1805982960.png)

　　![img](F:/lemon-guide-main/images/Others/856154-20170905170947257-1667065155.png)



### 异常断点

通过设置异常断点，在程序中出现需要拦截的异常时，会自动定位到异常行。点击+号添加Java Exception Breakpoints，添加异常断点。然后输入需要断点的异常类，如图6.7，之后可以在Java Exception Breakpoints里看到添加的异常断点。我这里添加了一个NullPointerException异常断点，出现空指针异常后，自动定位在空指针异常行：

　　![img](F:/lemon-guide-main/images/Others/856154-20170905200131851-150143203.png)

![img](F:/lemon-guide-main/images/Others/856154-20170905200305147-527881101.png) 　![img](F:/lemon-guide-main/images/Others/856154-20170905200726069-688175303.png) 



### 多线程调试

　　一般情况下我们调试的时候是在一个线程中的，一步一步往下走。但有时候你会发现在Debug的时候，想发起另外一个请求都无法进行了？那是因为IDEA在Debug时默认阻塞级别是ALL，会阻塞其它线程，只有在当前调试线程走完时才会走其它线程。可以在View Breakpoints里选择Thread，如图下图，然后点击Make Default设置为默认选项：

![img](F:/lemon-guide-main/images/Others/856154-20170905204329757-1196950664.png) 

　　切换线程，在下图中Frames的下拉列表里，可以切换当前的线程，如下我这里有两个Debug的线程，切换另外一个则进入另一个Debug的线程：![img](F:/lemon-guide-main/images/Others/856154-20170905205012663-56609868.png) 



### 回退断点

在调试的时候，想要重新走一下流程而不用再次发起一个请求？

所谓的断点回退，其实就是回退到上一个方法调用的开始处，在IDEA里测试无法一行一行地回退或回到到上一个断点处，而是回到上一个方法。回退的方式有两种：

​	方法一：是Drop Frame按钮，按调用的方法逐步回退，包括三方类库的其它方法(取消Show All Frames按钮会显示三方类库的方法，如图8.3)。

![img](F:/lemon-guide-main/images/Others/856154-20170905211428554-1617570377.png)

　　方法二：在调用栈方法上选择要回退的方法，右键选择Drop Frame(图8.4)，回退到该方法的上一个方法调用处，此时再按F9(Resume Program)，可以看到程序进入到该方法的断点处了。

　　![img](F:/lemon-guide-main/images/Others/856154-20170905212138101-113776159.png)

但有一点需要注意，断点回退只能重新走一下流程，之前的某些参数/数据的状态已经改变了的是无法回退到之前的状态的，如对象、集合、更新了数据库数据等等。



### 中断Debug

想要在Debug的时候，中断请求，不要再走剩余的流程了？

　　![img](F:/lemon-guide-main/images/Others/856154-20170905213656241-1998475384.png)

 

## 常用设置


### UID

快捷键：`ALT` + `INS`

### 折叠空包

![clipboard.png](F:/lemon-guide-main/images/Others/bV13Sa)

![clipboard.png](F:/lemon-guide-main/images/Others/bV13RH)



### 显示边沟图标

Editor→General→Gutter Icons

### 设置自动import包

可选，对于不能import \*的要求的，建议不要用这个：

[![img](F:/lemon-guide-main/images/Others/417876-20171119103804546-819180423.png)](https://images2017.cnblogs.com/blog/417876/201711/417876-20171119103804546-819180423.png)

如果非要用这个自动导入却不想导入\*的，可以通过配置这个来解决：

**![img](F:/lemon-guide-main/images/Others/417876-20171128105146972-1080966086.png)**

调整import包导入的顺序，保持和Eclipse一致：![img](F:/lemon-guide-main/images/Others/417876-20171129081632175-614878351.png)



### 右下角显示内存

[![img](F:/lemon-guide-main/images/Others/417876-20171119103943062-564729605.png)](https://images2017.cnblogs.com/blog/417876/201711/417876-20171119103943062-564729605.png)

点击右下角可以回收内存。



### 自定义Javadoc注释

@date可能不是标准的Javadoc，但是在业界标准来说，这个已经成为Javadoc必备的注释，因为大多数人都用这个来标注日期。建议：注释不要太个性，比如自定义类说明，日期时间字段等等；尽量保持统一的代码风格，建议参考阿里巴巴Java开发手册：

[![img](F:/lemon-guide-main/images/Others/417876-20171119150102656-1550889934.png)](https://images2017.cnblogs.com/blog/417876/201711/417876-20171119150102656-1550889934.png)



### 鼠标放上去自动显示文档

![设置功能](F:/lemon-guide-main/images/Others/201705221495449669133450.jpg)

![演示效果](F:/lemon-guide-main/images/Others/201705221495449833127277.png)



## 快捷键

Mac 键盘符号和修饰键说明

- `⌘` ——> `Command`
- `⇧` ——> `Shift`
- `⌥` ——> `Option`
- `⌃` ——> `Control`
- `↩︎` ——> `Return/Enter`
- `⌫` ——> `Delete`
- `⌦` ——> `向前删除键(Fn + Delete)`
- `↑` ——> `上箭头`
- `↓` ——> `下箭头`
- `←` ——> `左箭头`
- `→` ——> `右箭头`
- `⇞` ——> `Page Up(Fn + ↑)`
- `⇟` ——> `Page Down(Fn + ↓)`
- `⇥` ——> `右制表符(Tab键)`
- `⇤` ——> `左制表符(Shift + Tab)`
- `⎋` ——> `Escape(Esc)`
- `End` ——> `Fn + →`
- `Home` ——> `Fn + ←`

### Editing(编辑)

| 快捷键                                          | 作用                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `Control + Space`                               | 基本的代码补全（补全任何类、方法、变量）                     |
| `Control + Shift + Space`                       | 智能代码补全（过滤器方法列表和变量的预期类型）               |
| `Command + Shift + Enter`                       | 自动结束代码，行末自动添加分号                               |
| `Command + P`                                   | 显示方法的参数信息                                           |
| `Control + J`                                   | 快速查看文档                                                 |
| `Shift + F1`                                    | 查看外部文档（在某些代码上会触发打开浏览器显示相关文档）     |
| `Command` + 鼠标放在代码上                      | 显示代码简要信息                                             |
| `Command + F1`                                  | 在错误或警告处显示具体描述信息                               |
| `Command + N`, `Control + Enter`, `Control + N` | 生成代码（`getter`、`setter`、`hashCode`、`equals`、`toString`、构造函数等） |
| `Control + O`                                   | 覆盖方法（重写父类方法）                                     |
| `Control + I`                                   | 实现方法（实现接口中的方法）                                 |
| `Command + Option + T`                          | 包围代码（使用`if...else`、`try...catch`、`for`、`synchronized`等包围选中的代码） |
| `Command + /`                                   | 注释 / 取消注释与行注释                                      |
| `Command + Option + /`                          | 注释 / 取消注释与块注释                                      |
| `Option` + 方向键上                             | 连续选中代码块                                               |
| `Option` + 方向键下                             | 减少当前选中的代码块                                         |
| `Control + Shift + Q`                           | 显示上下文信息                                               |
| `Option + Enter`                                | 显示意向动作和快速修复代码                                   |
| `Command + Option + L`                          | 格式化代码                                                   |
| `Control + Option + O`                          | 优化 import                                                  |
| `Control + Option + I`                          | 自动缩进线                                                   |
| `Tab / Shift + Tab`                             | 缩进代码 / 反缩进代码                                        |
| `Command + X`                                   | 剪切当前行或选定的块到剪贴板                                 |
| `Command + C`                                   | 复制当前行或选定的块到剪贴板                                 |
| `Command + V`                                   | 从剪贴板粘贴                                                 |
| `Command + Shift + V`                           | 从最近的缓冲区粘贴                                           |
| `Command + D`                                   | 复制当前行或选定的块                                         |
| `Command + Delete`                              | 删除当前行或选定的块的行                                     |
| `Control + Shift + J`                           | 智能的将代码拼接成一行                                       |
| `Command + Enter`                               | 智能的拆分拼接的行                                           |
| `Shift + Enter`                                 | 开始新的一行                                                 |
| `Command + Shift + U`                           | 大小写切换                                                   |
| `Command + Shift + ]` / `Command + Shift + [`   | 选择直到代码块结束 / 开始                                    |
| `Option + Fn + Delete`                          | 删除到单词的末尾                                             |
| `Option + Delete`                               | 删除到单词的开头                                             |
| `Command` + 加号 / `Command` + 减号             | 展开 / 折叠代码块                                            |
| `Command + Shift` + 加号                        | 展开所以代码块                                               |
| `Command + Shift` + 减号                        | 折叠所有代码块                                               |
| `Command + W`                                   | 关闭活动的编辑器选项卡                                       |



### Search/Replace(查询/替换)

| 快捷键                | 作用                                                      |
| --------------------- | --------------------------------------------------------- |
| `Double Shift`        | 查询任何东西                                              |
| `Command + F`         | 文件内查找                                                |
| `Command + G`         | 查找模式下，向下查找                                      |
| `Command + Shift + G` | 查找模式下，向上查找                                      |
| `Command + R`         | 文件内替换                                                |
| `Command + Shift + F` | 全局查找（根据路径）                                      |
| `Command + Shift + R` | 全局替换（根据路径）                                      |
| `Command + Shift + S` | 查询结构（Ultimate Edition 版专用，需要在 Keymap 中设置） |
| `Command + Shift + M` | 替换结构（Ultimate Edition 版专用，需要在 Keymap 中设置） |



### Usage Search(使用查询)

| 快捷键                         | 作用                              |
| ------------------------------ | --------------------------------- |
| `Option + F7` / `Command + F7` | 在文件中查找用法 / 在类中查找用法 |
| `Command + Shift + F7`         | 在文件中突出显示的用法            |
| `Command + Option + F7`        | 显示用法                          |



### Compile and Run(编译和运行)

| 快捷键                                       | 作用                       |
| -------------------------------------------- | -------------------------- |
| `Command + F9`                               | 编译 Project               |
| `Command + Shift + F9`                       | 编译选择的文件、包或模块   |
| `Control + Option + R`                       | 弹出 Run 的可选择菜单      |
| `Control + Option + D`                       | 弹出 Debug 的可选择菜单    |
| `Control + R`                                | 运行                       |
| `Control + D`                                | 调试                       |
| `Control + Shift + R`, `Control + Shift + D` | 从编辑器运行上下文环境配置 |



### Debugging(调试)

| 快捷键                 | 作用                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `F8`                   | 进入下一步，如果当前行断点是一个方法，则不进入当前方法体内   |
| `F7`                   | 进入下一步，如果当前行断点是一个方法，则进入当前方法体内，如果该方法体还有方法，则不会进入该内嵌的方法中 |
| `Shift + F7`           | 智能步入，断点所在行上有多个方法调用，会弹出进入哪个方法     |
| `Shift + F8`           | 跳出                                                         |
| `Option + F9`          | 运行到光标处，如果光标前有其他断点会进入到该断点             |
| `Option + F8`          | 计算表达式（可以更改变量值使其生效）                         |
| `Command + Option + R` | 恢复程序运行，如果该断点下面代码还有断点则停在下一个断点上   |
| `Command + F8`         | 切换断点（若光标当前行有断点则取消断点，没有则加上断点）     |
| `Command + Shift + F8` | 查看断点信息                                                 |



### Navigation（导航）

| 快捷键                                                       | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Command + O`                                                | 查找类文件                                                   |
| `Command + Shift + O`                                        | 查找所有类型文件、打开文件、打开目录，打开目录需要在输入的内容前面或后面加一个反斜杠`/` |
| `Command + Option + O`                                       | 前往指定的变量 / 方法                                        |
| `Control` + 方向键左 / `Control` + 方向键右                  | 左右切换打开的编辑 tab 页                                    |
| `F12`                                                        | 返回到前一个工具窗口                                         |
| `Esc`                                                        | 从工具窗口进入代码文件窗口                                   |
| `Shift + Esc`                                                | 隐藏当前或最后一个活动的窗口，且光标进入代码文件窗口         |
| `Command + Shift + F4`                                       | 关闭活动 `run/messages/find/... tab`                         |
| `Command + L`                                                | 在当前文件跳转到某一行的指定处                               |
| `Command + E`                                                | 显示最近打开的文件记录列表                                   |
| `Option` + 方向键左 / `Option` + 方向键右                    | 光标跳转到当前单词 / 中文句的左 / 右侧开头位置               |
| `Command + Option` + 方向键左 / `Command + Option` + 方向键右 | 退回 / 前进到上一个操作的地方                                |
| `Command + Shift + Delete`                                   | 跳转到最后一个编辑的地方                                     |
| `Option + F1`                                                | 显示当前文件选择目标弹出层，弹出层中有很多目标可以进行选择(如在代码编辑窗口可以选择显示该文件的 Finder) |
| `Command + B` / `Command` + 鼠标点击                         | 进入光标所在的方法/变量的接口或是定义处                      |
| `Command + Option + B`                                       | 跳转到实现处，在某个调用的方法名上使用会跳到具体的实现处，可以跳过接口 |
| `Option + Space`, `Command + Y`                              | 快速打开光标所在方法、类的定义                               |
| `Control + Shift + B`                                        | 跳转到类型声明处                                             |
| `Command + U`                                                | 前往当前光标所在方法的父类的方法 / 接口定义                  |
| `Control` + 方向键下 / `Control` + 方向键上                  | 当前光标跳转到当前文件的前一个 / 后一个方法名位置            |
| `Command + ]` / `Command + [`                                | 移动光标到当前所在代码的花括号开始 / 结束位置                |
| `Command + F12`                                              | 弹出当前文件结构层，可以在弹出的层上直接输入进行筛选（可用于搜索类中的方法） |
| `Control + H`                                                | 显示当前类的层次结构                                         |
| `Command + Shift + H`                                        | 显示方法层次结构                                             |
| `Control + Option + H`                                       | 显示调用层次结构                                             |
| `F2` / `Shift + F2`                                          | 跳转到下一个 / 上一个突出错误或警告的位置                    |
| `F4` / `Command` + 方向键下                                  | 编辑 / 查看代码源                                            |
| `Option + Home`                                              | 显示到当前文件的导航条                                       |
| `F3`                                                         | 选中文件 / 文件夹 / 代码行，添加 / 取消书签                  |
| `Option + F3`                                                | 选中文件 / 文件夹/代码行，使用助记符添加 / 取消书签          |
| `Control + 0`…`Control + 9`                                  | 定位到对应数值的书签位置                                     |
| `Command + F3`                                               | 显示所有书签                                                 |



### Refactoring（重构）

| 快捷键                 | 作用                               |
| ---------------------- | ---------------------------------- |
| `F5`                   | 复制文件到指定目录                 |
| `F6`                   | 移动文件到指定目录                 |
| `Command + Delete`     | 在文件上为安全删除文件，弹出确认框 |
| `Shift + F6`           | 重命名文件                         |
| `Command + F6`         | 更改签名                           |
| `Command + Option + N` | 一致性                             |
| `Command + Option + M` | 将选中的代码提取为方法             |
| `Command + Option + V` | 提取变量                           |
| `Command + Option + F` | 提取字段                           |
| `Command + Option + C` | 提取常量                           |
| `Command + Option + P` | 提取参数                           |



### VCS / Local History（版本控制 / 本地历史记录）

| 快捷键               | 作用                       |
| -------------------- | -------------------------- |
| `Command + K`        | 提交代码到版本控制器       |
| `Command + T`        | 从版本控制器更新代码       |
| `Option + Shift + C` | 查看最近的变更记录         |
| `Control + C`        | 快速弹出版本控制器操作面板 |



### Live Templates（动态代码模板）

| 快捷键                 | 作用                                           |
| ---------------------- | ---------------------------------------------- |
| `Command + Option + J` | 弹出模板选择窗口，将选定的代码使用动态模板包住 |
| `Command + J`          | 插入自定义动态代码模板                         |



### General（通用）

| 快捷键                      | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `Command + 1`…`Command + 9` | 打开相应编号的工具窗口                                       |
| `Command + S`               | 保存所有                                                     |
| `Command + Option + Y`      | 同步、刷新                                                   |
| `Control + Command + F`     | 切换全屏模式                                                 |
| `Command + Shift + F12`     | 切换最大化编辑器                                             |
| `Option + Shift + F`        | 添加到收藏夹                                                 |
| `Option + Shift + I`        | 检查当前文件与当前的配置文件                                 |
| Control + `                 | 快速切换当前的 scheme（切换主题、代码样式等）                |
| `Command + ,`               | 打开 IDEA 系统设置                                           |
| `Command + ;`               | 打开项目结构对话框                                           |
| `Shift + Command + A`       | 查找动作（可设置相关选项）                                   |
| `Control + Shift + Tab`     | 编辑窗口标签和工具窗口之间切换（如果在切换的过程加按上 delete，则是关闭对应选中的窗口） |

## 问题

**IDEA 告警：Library source does not match the bytecode for class**

IntelliJ IDEA代码编码区提示库源不匹配字节码

IDEA 没有问题，你的的依赖项或本地 Maven 缓存也没有问题，它可以正确识别不匹配。

解决方案
由于使用lombok插件会造成编写的Java文件和编译后的class上有差别，所以IDEA打开时看到的是Maven打包时用的源码，而IDEA会自动匹配与.class反编译后的源代码，造成不匹配的提示。

解决方法可以说是没有的！忽略它吧！
