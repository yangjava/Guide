# Hutool

**A set of tools that keep Java sweet.**

## 简介

Hutool是一个小而全的Java工具类库，通过静态方法封装，降低相关API的学习成本，提高工作效率，使Java拥有函数式语言般的优雅，让Java语言也可以“甜甜的”。

Hutool中的工具方法来自每个用户的精雕细琢，它涵盖了Java开发底层代码中的方方面面，它既是大型项目开发中解决小问题的利器，也是小型项目中的效率担当；

- Web开发
- 与其它框架无耦合
- 高度可替换

## Hutool名称的由来

Hutool = Hu + tool，是原公司项目底层代码剥离后的开源库，“Hu”是公司名称的表示，tool表示工具。Hutool谐音“糊涂”，一方面简洁易懂，一方面寓意“难得糊涂”。

## 设计哲学

Hutool的设计思想是尽量减少重复的定义，让项目中的util这个package尽量少，总的来说有如下的几个思想：

- 方法优先于对象
- 自动识别优于用户定义
- 便捷性与灵活性并存
- 适配与兼容
- 可选依赖原则
- 无侵入原则

## 安装

Maven:在项目的pom.xml的dependencies中加入以下内容:

```
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.0.M3</version>
</dependency>
```

Gradle:

```
implementation 'cn.hutool:hutool-all:5.8.0.M3'
```

## Hutool 是什么

Hutool是一个Java工具包类库，对文件、流、加密解密、转码、正则、线程、XML等JDK方法进行封装，组成各种Util工具类

### 日期工具

通过DateUtil类，提供高度便捷的日期访问、处理和转换方式。

### HTTP客户端

通过HttpUtil对HTTP客户端的封装，实现便捷的HTTP请求，并简化文件上传操作。

### 转换工具

通过Convert类中的相应静态方法，提供一整套的类型转换解决方案，并通过ConverterRegistry工厂类自定义转换。

### 配置文件工具（SETTING）

通过Setting对象，提供兼容Properties文件的更加强大的配置文件工具，用于解决中文、分组等JDK配置文件存在的诸多问题。

### 日志工具

Hutool的日志功能，通过抽象Log接口，提供对Slf4j、LogBack、Log4j、JDK-Logging的全面兼容支持。

### JDBC工具类（DB模块）

通过db模块，提供对MySQL、Oracle等关系型数据库的JDBC封装，借助ActiveRecord思想，大大简化数据库操作。

## 包含组件

一个Java基础工具类，对文件、流、加密解密、转码、正则、线程、XML等JDK方法进行封装，组成各种Util工具类，同时提供以下组件：

| 模块               | 介绍                                                         |
| ------------------ | ------------------------------------------------------------ |
| hutool-aop         | JDK动态代理封装，提供非IOC下的切面支持                       |
| hutool-bloomFilter | 布隆过滤，提供一些Hash算法的布隆过滤                         |
| hutool-cache       | 简单缓存实现                                                 |
| hutool-core        | 核心，包括Bean操作、日期、各种Util等                         |
| hutool-cron        | 定时任务模块，提供类Crontab表达式的定时任务                  |
| hutool-crypto      | 加密解密模块，提供对称、非对称和摘要算法封装                 |
| hutool-db          | JDBC封装后的数据操作，基于ActiveRecord思想                   |
| hutool-dfa         | 基于DFA模型的多关键字查找                                    |
| hutool-extra       | 扩展模块，对第三方封装（模板引擎、邮件、Servlet、二维码、Emoji、FTP、分词等） |
| hutool-http        | 基于HttpUrlConnection的Http客户端封装                        |
| hutool-log         | 自动识别日志实现的日志门面                                   |
| hutool-script      | 脚本执行封装，例如Javascript                                 |
| hutool-setting     | 功能更强大的Setting配置文件和Properties封装                  |
| hutool-system      | 系统参数调用封装（JVM信息等）                                |
| hutool-json        | JSON实现                                                     |
| hutool-captcha     | 图片验证码实现                                               |
| hutool-poi         | 针对POI中Excel和Word的封装                                   |
| hutool-socket      | 基于Java的NIO和AIO的Socket封装                               |
| hutool-jwt         | JSON Web Token (JWT)封装实现                                 |

可以根据需求对每个模块单独引入，也可以通过引入`hutool-all`方式引入所有模块。