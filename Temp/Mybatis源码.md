# Mybatis源码

## 简介

MyBatis是现在最流行的java持久层框架之一，其通过XML配置的方式消除了绝大部分JDBC重复代码以及参数的设置，结果集的映射。

#### 那么我们如何学习Mybatis的源码呢?

爱因斯坦说过：“提出一个问题，往往比解决一个问题更重要，因为解决一个问题也许仅仅是一个数学上或实验上的技能而已，而提出新的问题、新的可能性，从新的角度去看旧的问题，都需要有创新性的想像力，而且标志着科学的真正的进步。”

对于Mybatis框架而言，我们把自己当成一个问题制造机，不断得提出问题，然后看看Mybatis是如何解决的?

那么下面就这么开始?

#### 我们为什么要使用Mybatis框架?

假设我们现在没用Mybatis框架，那么我们要操作一个数据库需要如下操作

```
package com.demo;

import com.demo.domain.User;

import java.sql.*;

public class JdbcDemo {


    public static void main(String[] args) {

        // 配置信息
        String driveName = "com.mysql.cj.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/mybatis?serverTimezone=GMT%2B8";
        String user = "root";
        String pass = "root";

        /**
         * 加载数据库驱动
         * 获取数据库链接
         * 获取预处理statement
         * 发送SQL命令
         * 获取结果集
         * 释放资源
         */
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            Class.forName(driveName);
            connection = DriverManager.getConnection(url, user, pass);
            String sql = "select * from user where id = ?";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setInt(1, 1);
            resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                User user1 = new User();
                user1.setId(resultSet.getInt("id"));
                user1.setUserName(resultSet.getString("user_name"));
                System.out.println(user1);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //释放资源
            if (resultSet != null) {
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (preparedStatement != null) {
                try {
                    preparedStatement.close();
                } catch (SQLException e) {

                    e.printStackTrace();
                }
            }
            if (connection != null) {
                try {
                    connection.close();
                } catch (SQLException e) {

                    e.printStackTrace();
                }
            }

        }
    }
}

```

原始的Jdbc操作我们可以看到很多问题如下

1.数据库频繁创建连接和释放连接，造成数据库资源浪费，影响数据库的性能

2.手写SQL语句，如果sql语句修改，需要重新编译java代码，不利于系统维护

3.参数需要手动设置，难以维护

4.返回结果需要手动遍历，难以管理

#### 那么Mybatis是如何处理以上问题的呢?

1.使用SqlMapConfig.xml配置数据链接池，使用数据库连接池管理数据库链接

2.将Sql语句维护在XXXXmapper.xml文件中

3.Mybatis自动将java对象映射至sql语句，通过statement中的**parameterType**定义输入参数的类型。

4.Mybatis自动将sql执行结果映射至java对象，通过statement中的**resultType**定义输出结果的类型。

#### 了解了Mybatis的问题，我们产生了更多的疑惑?

1.我们如何使用Mybatis

## 入门

#### 我们如何使用Mybatis?

要使用Mybatis，我们首先要去它的官网研究下

https://mybatis.org/mybatis-3/zh/getting-started.html

然后我们看到它的入门案例

我们先简单的配置一个mybatis数据库，创建一个简单的user表

```
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `user_name` varchar(10) COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```

然后去使用Mybatis

#### maven下载

```
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

#### 配置mybatis-3-config.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!--   配置环境-->
    <environments default="mysql">
        <!--配置mysql的环境-->
        <environment id="mysql">
            <!--配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置连接池-->
            <dataSource type="POOLED">
                <!--配置连接数据库的4个基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=UTF-8&amp;zeroDateTimeBehavior=convertToNull&amp;allowMultiQueries=true&amp;rewriteBatchedStatements=true&amp;serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
           </dataSource>
        </environment>
    </environments>

    <!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

#### 配置UserMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.demo.mapper.UserMapper">
    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.demo.domain.User">
        <id column="id" property="id" javaType="java.lang.Integer" jdbcType="INTEGER"/>
        <id column="user_name" property="userName" javaType="java.lang.String" jdbcType="VARCHAR"/>
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, user_name
    </sql>

    <select id="findAll" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from user
    </select>
</mapper>
```

简单的使用Mybatis

```
package com.demo;

import com.demo.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;
import java.util.List;

public class MybatisDemo {

    public static void main(String[] args) throws Exception {
        String resource = "mybatis-config.xml";
        //1.流形式读取mybatis配置文件
        InputStream stream = Resources.getResourceAsStream(resource);
        //2.通过配置文件创建SqlSessionFactory
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(stream);
        //3.通过SqlSessionFactory创建sqlSession
        SqlSession session = sessionFactory.openSession();
        //4.通过SqlSession执行Sql语句获取结果
        List<User> userList = session.selectList("com.demo.mapper.UserMapper.findAll");
        userList.stream().forEach(e-> System.out.println(e));
    }
}

```

从上面的代码中，我们可以看出Mybatis首先要加载配置信息，那么问题来了

#### Mybatis是如何加载和使用配置信息的呢?

从上述代码可以看出，SqlSessionFactoryBuilder是一个关键的入口类，其中承担了mybatis配置文件的加载，解析，内部构建等职责。我们就从SqlSessionFactoryBuilder开始研究

## 配置Configuration

### SqlSessionFactoryBuilder

SqlSessionFactoryBuilder是SqlSessionFactory的构造器,主要作用是创建SqlSessionFactory

源码如下

```
public class SqlSessionFactoryBuilder {

  public SqlSessionFactory build(Reader reader) {
    return build(reader, null, null);
  }

  public SqlSessionFactory build(Reader reader, String environment) {
    return build(reader, environment, null);
  }

  public SqlSessionFactory build(Reader reader, Properties properties) {
    return build(reader, null, properties);
  }

  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }

  public SqlSessionFactory build(InputStream inputStream) {
    return build(inputStream, null, null);
  }

  public SqlSessionFactory build(InputStream inputStream, String environment) {
    return build(inputStream, environment, null);
  }

  public SqlSessionFactory build(InputStream inputStream, Properties properties) {
    return build(inputStream, null, properties);
  }

  public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        inputStream.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }

  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }

}
```

SqlSessionFactoryBuilder提供了根据字节流、字符流以及直接使用org.apache.ibatis.session.Configuration配置类三种途径的读取配置信息方式，首先都是将XML配置文件构建为Configuration配置类XMLConfiguration对象；然后根据XMLConfiguration对象的parse方法创建了一个Configuration对象，根据传入的Configuration参数新建了一个DefaultSqlSessionFactory对象返回；很显然DefaultSqlSessionFactory是SqlSessionFactory接口的默认实现类

### XMLConfigBuilder

SqlSessionFactoryBuilder使用了XMLConfigBuilder作为解析器。

XMLConfigBuilder类位于Mybatis包的org.apache.ibatis.builder.xml目录下，继承于BaseBuilder类

XMLConfigBuilder是关于mybatis的XML配置的构造类，负责构造mybatis的XML配置的。

XMLConfigBuilder共有四个属性，代码如下：

```
public class XMLConfigBuilder extends BaseBuilder {
  //解析标识,因为Configuration是全局变量，只需要解析创建一次即可，true表示已经解析创建过,false则表示没有
  private boolean parsed;
  private final XPathParser parser;
  //环境参数
  private String environment;
  private final ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();
  ...
```



XMLConfigBuilder以及解析Mapper文件的XMLMapperBuilder都继承于BaseBuilder。他们对于XML文件本身技术上的加载和解析都委托给了XPathParser，最终用的是jdk自带的xml解析器而非第三方比如dom4j，底层使用了xpath方式进行节点解析。new XPathParser(reader, true, props, new XMLMapperEntityResolver())的参数含义分别是Reader，是否进行DTD 校验，属性配置，XML实体节点解析器。

XMLConfigBuilder创建完成之后，SqlSessionFactoryBuild调用parser.parse()创建Configuration。所有，真正Configuration构建逻辑就在XMLConfigBuilder.parse()里面，如下所示：

```
  public Configuration parse() {
  //判断Configuration是否解析过，Configuration是全局变量，只需要解析创建一次即可
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    //mybatis配置文件解析的主流程
    // 解析XML配置的configuration节点的内容，得到XNode对象
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }

```

parse的作用是解析mybatis-config.xml的configuration节点的内容，然后挨个赋值给configuration对象的属性；

首先判断有没有解析过配置文件，只有没有解析过才允许解析。其中调用了parser.evalNode(“/configuration”)返回根节点的org.apache.ibatis.parsing.XNode表示，XNode里面主要把关键的节点属性和占位符变量结构化出来

如下所示：

```
  private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

下面我们来看下mybatis-3-config.dtd的声明

```
<?xml version="1.0" encoding="UTF-8" ?>
<!--

       Copyright 2009-2016 the original author or authors.

       Licensed under the Apache License, Version 2.0 (the "License");
       you may not use this file except in compliance with the License.
       You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

       Unless required by applicable law or agreed to in writing, software
       distributed under the License is distributed on an "AS IS" BASIS,
       WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
       See the License for the specific language governing permissions and
       limitations under the License.

-->
<!ELEMENT configuration (properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?)>

<!ELEMENT databaseIdProvider (property*)>
<!ATTLIST databaseIdProvider
type CDATA #REQUIRED
>

<!ELEMENT properties (property*)>
<!ATTLIST properties
resource CDATA #IMPLIED
url CDATA #IMPLIED
>

<!ELEMENT property EMPTY>
<!ATTLIST property
name CDATA #REQUIRED
value CDATA #REQUIRED
>

<!ELEMENT settings (setting+)>

<!ELEMENT setting EMPTY>
<!ATTLIST setting
name CDATA #REQUIRED
value CDATA #REQUIRED
>

<!ELEMENT typeAliases (typeAlias*,package*)>

<!ELEMENT typeAlias EMPTY>
<!ATTLIST typeAlias
type CDATA #REQUIRED
alias CDATA #IMPLIED
>

<!ELEMENT typeHandlers (typeHandler*,package*)>

<!ELEMENT typeHandler EMPTY>
<!ATTLIST typeHandler
javaType CDATA #IMPLIED
jdbcType CDATA #IMPLIED
handler CDATA #REQUIRED
>

<!ELEMENT objectFactory (property*)>
<!ATTLIST objectFactory
type CDATA #REQUIRED
>

<!ELEMENT objectWrapperFactory EMPTY>
<!ATTLIST objectWrapperFactory
type CDATA #REQUIRED
>

<!ELEMENT reflectorFactory EMPTY>
<!ATTLIST reflectorFactory
type CDATA #REQUIRED
>

<!ELEMENT plugins (plugin+)>

<!ELEMENT plugin (property*)>
<!ATTLIST plugin
interceptor CDATA #REQUIRED
>

<!ELEMENT environments (environment+)>
<!ATTLIST environments
default CDATA #REQUIRED
>

<!ELEMENT environment (transactionManager,dataSource)>
<!ATTLIST environment
id CDATA #REQUIRED
>

<!ELEMENT transactionManager (property*)>
<!ATTLIST transactionManager
type CDATA #REQUIRED
>

<!ELEMENT dataSource (property*)>
<!ATTLIST dataSource
type CDATA #REQUIRED
>

<!ELEMENT mappers (mapper*,package*)>

<!ELEMENT mapper EMPTY>
<!ATTLIST mapper
resource CDATA #IMPLIED
url CDATA #IMPLIED
class CDATA #IMPLIED
>

<!ELEMENT package EMPTY>
<!ATTLIST package
name CDATA #REQUIRED
>
```

mybatis-config文件最多有11个配置项，分别是properties?, settings?, typeAliases?, typeHandlers?, objectFactory?, objectWrapperFactory?, reflectorFactory?, plugins?, environments?, databaseIdProvider?, mappers?

所有的配置最后保存到org.apache.ibatis.session.Configuration

下面我们来看下Configuration

### Configuration

```
public class Configuration {

  protected Environment environment;
  // 允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为false。默认为false
  protected boolean safeRowBoundsEnabled;
  // 允许在嵌套语句中使用分页（ResultHandler）。如果允许使用则设置为false。
  protected boolean safeResultHandlerEnabled = true;
  // 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。默认false
  protected boolean mapUnderscoreToCamelCase;
  // 当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属性会按需加载。默认值false (true in ≤3.4.1)
  protected boolean aggressiveLazyLoading;
  // 是否允许单一语句返回多结果集（需要兼容驱动）。
  protected boolean multipleResultSetsEnabled = true;

  // 允许 JDBC 支持自动生成主键，需要驱动兼容。这就是insert时获取mysql自增主键/oracle sequence的开关。注：一般来说,这是希望的结果,应该默认值为true比较合适。
  protected boolean useGeneratedKeys;

  // 使用列标签代替列名,一般来说,这是希望的结果
  protected boolean useColumnLabel = true;

  // 是否启用缓存
  protected boolean cacheEnabled = true;

  // 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的。
  protected boolean callSettersOnNulls;

  // 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的工程必须采用Java 8编译，并且加上-parameters选项。（从3.4.1开始）
  protected boolean useActualParamName = true;

  //当返回行的所有列都是空时，MyBatis默认返回null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集 (i.e. collectioin and association)。（从3.4.2开始） 注：这里应该拆分为两个参数比较合适, 一个用于结果集，一个用于单记录。通常来说，我们会希望结果集不是null，单记录仍然是null
  protected boolean returnInstanceForEmptyRow;

  // 指定 MyBatis 增加到日志名称的前缀。
  protected String logPrefix;

  // 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。一般建议指定为slf4j或log4j
  protected Class <? extends Log> logImpl;

  // 指定VFS的实现, VFS是mybatis提供的用于访问AS内资源的一个简便接口
  protected Class <? extends VFS> vfsImpl;

  // MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。
  protected LocalCacheScope localCacheScope = LocalCacheScope.SESSION;

  // 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。
  protected JdbcType jdbcTypeForNull = JdbcType.OTHER;

  // 指定对象的哪个方法触发一次延迟加载。
  protected Set<String> lazyLoadTriggerMethods = new HashSet<String>(Arrays.asList(new String[] { "equals", "clone", "hashCode", "toString" }));

  // 设置超时时间，它决定驱动等待数据库响应的秒数。默认不超时
  protected Integer defaultStatementTimeout;

  // 为驱动的结果集设置默认获取数量。
  protected Integer defaultFetchSize;

  // SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。
  protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE;

  // 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。
  protected AutoMappingBehavior autoMappingBehavior = AutoMappingBehavior.PARTIAL;

  // 指定发现自动映射目标未知列（或者未知属性类型）的行为。这个值应该设置为WARNING比较合适
  protected AutoMappingUnknownColumnBehavior autoMappingUnknownColumnBehavior = AutoMappingUnknownColumnBehavior.NONE;

  // settings下的properties属性
  protected Properties variables = new Properties();

  // 默认的反射器工厂,用于操作属性、构造器方便
  protected ReflectorFactory reflectorFactory = new DefaultReflectorFactory();

  // 对象工厂, 所有的类resultMap类都需要依赖于对象工厂来实例化
  protected ObjectFactory objectFactory = new DefaultObjectFactory();

  // 对象包装器工厂,主要用来在创建非原生对象,比如增加了某些监控或者特殊属性的代理类
  protected ObjectWrapperFactory objectWrapperFactory = new DefaultObjectWrapperFactory();

  // 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态。
  protected boolean lazyLoadingEnabled = false;

  // 指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。MyBatis 3.3+使用JAVASSIST
  protected ProxyFactory proxyFactory = new JavassistProxyFactory(); // #224 Using internal Javassist instead of OGNL

  // MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。
  protected String databaseId;

  /**
   * Configuration factory class.
   * Used to create Configuration for loading deserialized unread properties.
   * 指定一个提供Configuration实例的类. 这个被返回的Configuration实例是用来加载被反序列化对象的懒加载属性值. 这个类必须包含一个签名方法static Configuration getConfiguration(). (从 3.2.3 版本开始)
   */
  protected Class<?> configurationFactory;

  protected final MapperRegistry mapperRegistry = new MapperRegistry(this);

  // mybatis插件列表
  protected final InterceptorChain interceptorChain = new InterceptorChain();
  protected final TypeHandlerRegistry typeHandlerRegistry = new TypeHandlerRegistry();

  // 类型注册器, 用于在执行sql语句的出入参映射以及mybatis-config文件里的各种配置比如<transactionManager type="JDBC"/><dataSource type="POOLED">时使用简写, 后面会详细解释
  protected final TypeAliasRegistry typeAliasRegistry = new TypeAliasRegistry();
  protected final LanguageDriverRegistry languageRegistry = new LanguageDriverRegistry();

  protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection");
  protected final Map<String, Cache> caches = new StrictMap<Cache>("Caches collection");
  protected final Map<String, ResultMap> resultMaps = new StrictMap<ResultMap>("Result Maps collection");
  protected final Map<String, ParameterMap> parameterMaps = new StrictMap<ParameterMap>("Parameter Maps collection");
  protected final Map<String, KeyGenerator> keyGenerators = new StrictMap<KeyGenerator>("Key Generators collection");

  protected final Set<String> loadedResources = new HashSet<String>();
  protected final Map<String, XNode> sqlFragments = new StrictMap<XNode>("XML fragments parsed from previous mappers");

  protected final Collection<XMLStatementBuilder> incompleteStatements = new LinkedList<XMLStatementBuilder>();
  protected final Collection<CacheRefResolver> incompleteCacheRefs = new LinkedList<CacheRefResolver>();
  protected final Collection<ResultMapResolver> incompleteResultMaps = new LinkedList<ResultMapResolver>();
  protected final Collection<MethodResolver> incompleteMethods = new LinkedList<MethodResolver>();

  /*
   * A map holds cache-ref relationship. The key is the namespace that
   * references a cache bound to another namespace and the value is the
   * namespace which the actual cache is bound to.
   */
  protected final Map<String, String> cacheRefMap = new HashMap<String, String>();

  public Configuration(Environment environment) {
    this();
    this.environment = environment;
  }

  public Configuration() {
    // 内置别名注册
    typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
    typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);

    typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
    typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
    typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);

    typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
    typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
    typeAliasRegistry.registerAlias("LRU", LruCache.class);
    typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
    typeAliasRegistry.registerAlias("WEAK", WeakCache.class);

    typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);

    typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
    typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);

    typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
    typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
    typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
    typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
    typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
    typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
    typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);

    typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
    typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);

    languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
    languageRegistry.register(RawLanguageDriver.class);
  }

  ... 省去不必要的getter/setter

  public Class<? extends VFS> getVfsImpl() {
    return this.vfsImpl;
  }

  public void setVfsImpl(Class<? extends VFS> vfsImpl) {
    if (vfsImpl != null) {
      this.vfsImpl = vfsImpl;
      VFS.addImplClass(this.vfsImpl);
    }
  }

  public ProxyFactory getProxyFactory() {
    return proxyFactory;
  }

  public void setProxyFactory(ProxyFactory proxyFactory) {
    if (proxyFactory == null) {
      proxyFactory = new JavassistProxyFactory();
    }
    this.proxyFactory = proxyFactory;
  }

  /**
   * @since 3.2.2
   */
  public List<Interceptor> getInterceptors() {
    return interceptorChain.getInterceptors();
  }

  public LanguageDriverRegistry getLanguageRegistry() {
    return languageRegistry;
  }

  public void setDefaultScriptingLanguage(Class<?> driver) {
    if (driver == null) {
      driver = XMLLanguageDriver.class;
    }
    getLanguageRegistry().setDefaultDriverClass(driver);
  }

  public LanguageDriver getDefaultScriptingLanguageInstance() {
    return languageRegistry.getDefaultDriver();
  }

  /** @deprecated Use {@link #getDefaultScriptingLanguageInstance()} */
  @Deprecated
  public LanguageDriver getDefaultScriptingLanuageInstance() {
    return getDefaultScriptingLanguageInstance();
  }

  public MetaObject newMetaObject(Object object) {
    return MetaObject.forObject(object, objectFactory, objectWrapperFactory, reflectorFactory);
  }

  public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
  }

  public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }

  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
  }

  public Executor newExecutor(Transaction transaction) {
    return newExecutor(transaction, defaultExecutorType);
  }

  public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }

  public void addKeyGenerator(String id, KeyGenerator keyGenerator) {
    keyGenerators.put(id, keyGenerator);
  }

  public Collection<String> getKeyGeneratorNames() {
    return keyGenerators.keySet();
  }

  public Collection<KeyGenerator> getKeyGenerators() {
    return keyGenerators.values();
  }

  public KeyGenerator getKeyGenerator(String id) {
    return keyGenerators.get(id);
  }

  public boolean hasKeyGenerator(String id) {
    return keyGenerators.containsKey(id);
  }

  public void addCache(Cache cache) {
    caches.put(cache.getId(), cache);
  }

  public Collection<String> getCacheNames() {
    return caches.keySet();
  }

  public Collection<Cache> getCaches() {
    return caches.values();
  }

  public Cache getCache(String id) {
    return caches.get(id);
  }

  public boolean hasCache(String id) {
    return caches.containsKey(id);
  }

  public void addResultMap(ResultMap rm) {
    resultMaps.put(rm.getId(), rm);
    checkLocallyForDiscriminatedNestedResultMaps(rm);
    checkGloballyForDiscriminatedNestedResultMaps(rm);
  }

  public Collection<String> getResultMapNames() {
    return resultMaps.keySet();
  }

  public Collection<ResultMap> getResultMaps() {
    return resultMaps.values();
  }

  public ResultMap getResultMap(String id) {
    return resultMaps.get(id);
  }

  public boolean hasResultMap(String id) {
    return resultMaps.containsKey(id);
  }

  public void addParameterMap(ParameterMap pm) {
    parameterMaps.put(pm.getId(), pm);
  }

  public Collection<String> getParameterMapNames() {
    return parameterMaps.keySet();
  }

  public Collection<ParameterMap> getParameterMaps() {
    return parameterMaps.values();
  }

  public ParameterMap getParameterMap(String id) {
    return parameterMaps.get(id);
  }

  public boolean hasParameterMap(String id) {
    return parameterMaps.containsKey(id);
  }

  public void addMappedStatement(MappedStatement ms) {
    mappedStatements.put(ms.getId(), ms);
  }

  public Collection<String> getMappedStatementNames() {
    buildAllStatements();
    return mappedStatements.keySet();
  }

  public Collection<MappedStatement> getMappedStatements() {
    buildAllStatements();
    return mappedStatements.values();
  }

  public Collection<XMLStatementBuilder> getIncompleteStatements() {
    return incompleteStatements;
  }

  public void addIncompleteStatement(XMLStatementBuilder incompleteStatement) {
    incompleteStatements.add(incompleteStatement);
  }

  public Collection<CacheRefResolver> getIncompleteCacheRefs() {
    return incompleteCacheRefs;
  }

  public void addIncompleteCacheRef(CacheRefResolver incompleteCacheRef) {
    incompleteCacheRefs.add(incompleteCacheRef);
  }

  public Collection<ResultMapResolver> getIncompleteResultMaps() {
    return incompleteResultMaps;
  }

  public void addIncompleteResultMap(ResultMapResolver resultMapResolver) {
    incompleteResultMaps.add(resultMapResolver);
  }

  public void addIncompleteMethod(MethodResolver builder) {
    incompleteMethods.add(builder);
  }

  public Collection<MethodResolver> getIncompleteMethods() {
    return incompleteMethods;
  }

  public MappedStatement getMappedStatement(String id) {
    return this.getMappedStatement(id, true);
  }

  public MappedStatement getMappedStatement(String id, boolean validateIncompleteStatements) {
    if (validateIncompleteStatements) {
      buildAllStatements();
    }
    return mappedStatements.get(id);
  }

  public Map<String, XNode> getSqlFragments() {
    return sqlFragments;
  }

  public void addInterceptor(Interceptor interceptor) {
    interceptorChain.addInterceptor(interceptor);
  }

  public void addMappers(String packageName, Class<?> superType) {
    mapperRegistry.addMappers(packageName, superType);
  }

  public void addMappers(String packageName) {
    mapperRegistry.addMappers(packageName);
  }

  public <T> void addMapper(Class<T> type) {
    mapperRegistry.addMapper(type);
  }

  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
  }

  public boolean hasMapper(Class<?> type) {
    return mapperRegistry.hasMapper(type);
  }

  public boolean hasStatement(String statementName) {
    return hasStatement(statementName, true);
  }

  public boolean hasStatement(String statementName, boolean validateIncompleteStatements) {
    if (validateIncompleteStatements) {
      buildAllStatements();
    }
    return mappedStatements.containsKey(statementName);
  }

  public void addCacheRef(String namespace, String referencedNamespace) {
    cacheRefMap.put(namespace, referencedNamespace);
  }

  /*
   * Parses all the unprocessed statement nodes in the cache. It is recommended
   * to call this method once all the mappers are added as it provides fail-fast
   * statement validation.
   */
  protected void buildAllStatements() {
    if (!incompleteResultMaps.isEmpty()) {
      synchronized (incompleteResultMaps) {
        // This always throws a BuilderException.
        incompleteResultMaps.iterator().next().resolve();
      }
    }
    if (!incompleteCacheRefs.isEmpty()) {
      synchronized (incompleteCacheRefs) {
        // This always throws a BuilderException.
        incompleteCacheRefs.iterator().next().resolveCacheRef();
      }
    }
    if (!incompleteStatements.isEmpty()) {
      synchronized (incompleteStatements) {
        // This always throws a BuilderException.
        incompleteStatements.iterator().next().parseStatementNode();
      }
    }
    if (!incompleteMethods.isEmpty()) {
      synchronized (incompleteMethods) {
        // This always throws a BuilderException.
        incompleteMethods.iterator().next().resolve();
      }
    }
  }

  /*
   * Extracts namespace from fully qualified statement id.
   *
   * @param statementId
   * @return namespace or null when id does not contain period.
   */
  protected String extractNamespace(String statementId) {
    int lastPeriod = statementId.lastIndexOf('.');
    return lastPeriod > 0 ? statementId.substring(0, lastPeriod) : null;
  }

  // Slow but a one time cost. A better solution is welcome.
  protected void checkGloballyForDiscriminatedNestedResultMaps(ResultMap rm) {
    if (rm.hasNestedResultMaps()) {
      for (Map.Entry<String, ResultMap> entry : resultMaps.entrySet()) {
        Object value = entry.getValue();
        if (value instanceof ResultMap) {
          ResultMap entryResultMap = (ResultMap) value;
          if (!entryResultMap.hasNestedResultMaps() && entryResultMap.getDiscriminator() != null) {
            Collection<String> discriminatedResultMapNames = entryResultMap.getDiscriminator().getDiscriminatorMap().values();
            if (discriminatedResultMapNames.contains(rm.getId())) {
              entryResultMap.forceNestedResultMaps();
            }
          }
        }
      }
    }
  }

  // Slow but a one time cost. A better solution is welcome.
  protected void checkLocallyForDiscriminatedNestedResultMaps(ResultMap rm) {
    if (!rm.hasNestedResultMaps() && rm.getDiscriminator() != null) {
      for (Map.Entry<String, String> entry : rm.getDiscriminator().getDiscriminatorMap().entrySet()) {
        String discriminatedResultMapName = entry.getValue();
        if (hasResultMap(discriminatedResultMapName)) {
          ResultMap discriminatedResultMap = resultMaps.get(discriminatedResultMapName);
          if (discriminatedResultMap.hasNestedResultMaps()) {
            rm.forceNestedResultMaps();
            break;
          }
        }
      }
    }
  }

  protected static class StrictMap<V> extends HashMap<String, V> {

    private static final long serialVersionUID = -4950446264854982944L;
    private final String name;

    public StrictMap(String name, int initialCapacity, float loadFactor) {
      super(initialCapacity, loadFactor);
      this.name = name;
    }

    public StrictMap(String name, int initialCapacity) {
      super(initialCapacity);
      this.name = name;
    }

    public StrictMap(String name) {
      super();
      this.name = name;
    }

    public StrictMap(String name, Map<String, ? extends V> m) {
      super(m);
      this.name = name;
    }

    @SuppressWarnings("unchecked")
    public V put(String key, V value) {
      if (containsKey(key)) {
        throw new IllegalArgumentException(name + " already contains value for " + key);
      }
      if (key.contains(".")) {
        final String shortKey = getShortName(key);
        if (super.get(shortKey) == null) {
          super.put(shortKey, value);
        } else {
          super.put(shortKey, (V) new Ambiguity(shortKey));
        }
      }
      return super.put(key, value);
    }

    public V get(Object key) {
      V value = super.get(key);
      if (value == null) {
        throw new IllegalArgumentException(name + " does not contain value for " + key);
      }
      if (value instanceof Ambiguity) {
        throw new IllegalArgumentException(((Ambiguity) value).getSubject() + " is ambiguous in " + name
            + " (try using the full name including the namespace, or rename one of the entries)");
      }
      return value;
    }

    private String getShortName(String key) {
      final String[] keyParts = key.split("\\.");
      return keyParts[keyParts.length - 1];
    }

    protected static class Ambiguity {
      final private String subject;

      public Ambiguity(String subject) {
        this.subject = subject;
      }

      public String getSubject() {
        return subject;
      }
    }
  }
}
```

mybatis中所有环境配置、resultMap集合、sql语句集合、插件列表、缓存、加载的xml列表、类型别名、类型处理器等全部都维护在Configuration中。Configuration中包含了一个内部静态类StrictMap，它继承于HashMap，对HashMap的装饰在于增加了put时防重复的处理，get时取不到值时候的异常处理，这样核心应用层就不需要额外关心各种对象异常处理,简化应用层逻辑。

下面看下配置项解析

#### properties

```
private void propertiesElement(XNode context) throws Exception {
  if (context != null) {
    Properties defaults = context.getChildrenAsProperties();
    String resource = context.getStringAttribute("resource");
    String url = context.getStringAttribute("url");
    if (resource != null && url != null) {
      throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
    }
    if (resource != null) {
      defaults.putAll(Resources.getResourceAsProperties(resource));
    } else if (url != null) {
      defaults.putAll(Resources.getUrlAsProperties(url));
    }
    Properties vars = configuration.getVariables();
    if (vars != null) {
      defaults.putAll(vars);
    }
    parser.setVariables(defaults);
    configuration.setVariables(defaults);
  }
}
```

总体逻辑比较简单，首先加载properties节点下的property属性

```
<properties resource="com/mybatis/config.properties">
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</properties>
```

然后从url或resource加载配置文件，都先和configuration.variables合并，然后赋值到XMLConfigBuilder.parser和BaseBuilder.configuration。此时开始所有的属性就可以在随后的整个配置文件中使用了。

#### settings

```
  private Properties settingsAsProperties(XNode context) {
    if (context == null) {
      return new Properties();
    }
    Properties props = context.getChildrenAsProperties();
    // Check that all settings are known to the configuration class
    MetaClass metaConfig = MetaClass.forClass(Configuration.class, localReflectorFactory);
    for (Object key : props.keySet()) {
      if (!metaConfig.hasSetter(String.valueOf(key))) {
        throw new BuilderException("The setting " + key + " is not known.  Make sure you spelled it correctly (case sensitive).");
      }
    }
    return props;
  }
```

首先加载settings下面的setting节点为property，然后检查所有属性,确保它们都在Configuration中已定义，而非未知的设置。注：MetaClass是一个保存对象定义比如getter/setter/构造器等的元数据类,localReflectorFactory则是mybatis提供的默认反射工厂实现，这个ReflectorFactory主要采用了工厂类，其内部使用的Reflector采用了facade设计模式，简化反射的使用。

#### loadCustomVfs

VFS主要用来加载容器内的各种资源，比如jar或者class文件。mybatis提供了2个实现 JBoss6VFS 和 DefaultVFS，并提供了用户扩展点，用于自定义VFS实现，加载顺序是自定义VFS实现 > 默认VFS实现 取第一个加载成功的，默认情况下会先加载JBoss6VFS，如果classpath下找不到jboss的vfs实现才会加载默认VFS实现，启动打印的日志如下：
　　org.apache.ibatis.io.VFS.getClass(VFS.java:111) Class not found: org.jboss.vfs.VFS
　　org.apache.ibatis.io.JBoss6VFS.setInvalid(JBoss6VFS.java:142) JBoss 6 VFS API is not available in this environment.
　　org.apache.ibatis.io.VFS.getClass(VFS.java:111) Class not found: org.jboss.vfs.VirtualFile
　　org.apache.ibatis.io.VFS$VFSHolder.createVFS(VFS.java:63) VFS implementation org.apache.ibatis.io.JBoss6VFS is not valid in this environment.
　　org.apache.ibatis.io.VFS$VFSHolder.createVFS(VFS.java:77) Using VFS adapter org.apache.ibatis.io.DefaultVFS

#### typeAliases

```
  private void typeAliasesElement(XNode parent) {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          String typeAliasPackage = child.getStringAttribute("name");
          configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
        } else {
          String alias = child.getStringAttribute("alias");
          String type = child.getStringAttribute("type");
          try {
            Class<?> clazz = Resources.classForName(type);
            if (alias == null) {
              typeAliasRegistry.registerAlias(clazz);
            } else {
              typeAliasRegistry.registerAlias(alias, clazz);
            }
          } catch (ClassNotFoundException e) {
            throw new BuilderException("Error registering typeAlias for '" + alias + "'. Cause: " + e, e);
          }
        }
      }
    }
  }
```

从上述代码可以看出，mybatis主要提供两种类型的别名设置，具体类的别名以及包的别名设置。类型别名是为 Java 类型设置一个短的名字，存在的意义仅在于用来减少类完全限定名的冗余。所有的别名，无论是内置的还是自定义的，都一开始被保存在configuration.typeAliasRegistry中了，这样就可以确保任何时候使用别名和FQN的效果是一样的。

#### plugins

mybatis调用pluginElement(root.evalNode(“plugins”));加载mybatis插件，最常用的插件应该算是分页插件PageHelper了，再比如druid连接池提供的各种监控、拦截、预发检查功能，在使用其它连接池比如dbcp的时候，在不修改连接池源码的情况下，就可以借助mybatis的插件体系实现。

```
  private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        String interceptor = child.getStringAttribute("interceptor");
        Properties properties = child.getChildrenAsProperties();
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
        interceptorInstance.setProperties(properties);
        configuration.addInterceptor(interceptorInstance);
      }
    }
  }
```

插件在具体实现的时候，采用的是拦截器模式，要注册为mybatis插件，必须实现org.apache.ibatis.plugin.Interceptor接口，每个插件可以有自己的属性。interceptor属性值既可以完整的类名，也可以是别名，只要别名在typealias中存在即可，如果启动时无法解析，会抛出ClassNotFound异常。实例化插件后，将设置插件的属性赋值给插件实现类的属性字段。

#### objectFactory

什么是对象工厂？MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂DefaultObjectFactory做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。

```
  private void objectFactoryElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      Properties properties = context.getChildrenAsProperties();
      ObjectFactory factory = (ObjectFactory) resolveClass(type).newInstance();
      factory.setProperties(properties);
      configuration.setObjectFactory(factory);
    }
  }
```

#### objectWrapperFactory

对象包装器工厂主要用来包装返回result对象，比如说可以用来设置某些敏感字段脱敏或者加密等。默认对象包装器工厂是DefaultObjectWrapperFactory，也就是不使用包装器工厂。

```
  private void objectWrapperFactoryElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      ObjectWrapperFactory factory = (ObjectWrapperFactory) resolveClass(type).newInstance();
      configuration.setObjectWrapperFactory(factory);
    }
  }
```

#### reflectorFactoryElement

因为加载配置文件中的各种插件类等等，为了提供更好的灵活性，mybatis支持用户自定义反射工厂，不过总体来说，用的不多，要实现反射工厂，只要实现ReflectorFactory接口即可。默认的反射工厂是DefaultReflectorFactory。

```
  private void reflectorFactoryElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      ReflectorFactory factory = (ReflectorFactory) resolveClass(type).newInstance();
      configuration.setReflectorFactory(factory);
    }
  }
```

#### environmentsElement

环境可以说是mybatis-config配置文件中最重要的部分，它类似于spring和maven里面的profile，允许给开发、生产环境同时配置不同的environment，根据不同的环境加载不同的配置，这也是常见的做法，如果在SqlSessionFactoryBuilder调用期间没有传递使用哪个环境的话，默认会使用一个名为default”的环境。找到对应的environment之后，就可以加载事务管理器和数据源了。事务管理器和数据源类型这里都用到了类型别名，JDBC/POOLED都是在mybatis内置提供的，在Configuration构造器执行期间注册到TypeAliasRegister。
　　mybatis内置提供JDBC和MANAGED两种事务管理方式，前者主要用于简单JDBC模式，后者主要用于容器管理事务，一般使用JDBC事务管理方式。mybatis内置提供JNDI、POOLED、UNPOOLED三种数据源工厂，一般情况下使用POOLED数据源。

```
 private void environmentsElement(XNode context) throws Exception {
    if (context != null) {
      if (environment == null) {
        environment = context.getStringAttribute("default");
      }
      for (XNode child : context.getChildren()) {
        String id = child.getStringAttribute("id");
        if (isSpecifiedEnvironment(id)) {
          TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
          DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
          DataSource dataSource = dsFactory.getDataSource();
          Environment.Builder environmentBuilder = new Environment.Builder(id)
              .transactionFactory(txFactory)
              .dataSource(dataSource);
          configuration.setEnvironment(environmentBuilder.build());
        }
      }
    }
  }
```

#### databaseIdProviderElement

MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。 MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库 databaseId 属性的所有语句。 如果同时找到带有 databaseId 和不带 databaseId 的相同语句，则后者会被舍弃。

```
  private void databaseIdProviderElement(XNode context) throws Exception {
    DatabaseIdProvider databaseIdProvider = null;
    if (context != null) {
      String type = context.getStringAttribute("type");
      // awful patch to keep backward compatibility
      if ("VENDOR".equals(type)) {
        type = "DB_VENDOR";
      }
      Properties properties = context.getChildrenAsProperties();
      databaseIdProvider = (DatabaseIdProvider) resolveClass(type).newInstance();
      databaseIdProvider.setProperties(properties);
    }
    Environment environment = configuration.getEnvironment();
    if (environment != null && databaseIdProvider != null) {
      String databaseId = databaseIdProvider.getDatabaseId(environment.getDataSource());
      configuration.setDatabaseId(databaseId);
    }
  }
```

#### typeHandlerElement

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。

为了简化使用，mybatis在初始化TypeHandlerRegistry期间，自动注册了大部分的常用的类型处理器比如字符串、数字、日期等。对于非标准的类型，用户可以自定义类型处理器来处理。要实现一个自定义类型处理器，只要实现 org.apache.ibatis.type.TypeHandler 接口， 或继承一个实用类 org.apache.ibatis.type.BaseTypeHandler， 并将它映射到一个 JDBC 类型即可。

```
  private void typeHandlerElement(XNode parent) {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          String typeHandlerPackage = child.getStringAttribute("name");
          typeHandlerRegistry.register(typeHandlerPackage);
        } else {
          String javaTypeName = child.getStringAttribute("javaType");
          String jdbcTypeName = child.getStringAttribute("jdbcType");
          String handlerTypeName = child.getStringAttribute("handler");
          Class<?> javaTypeClass = resolveClass(javaTypeName);
          JdbcType jdbcType = resolveJdbcType(jdbcTypeName);
          Class<?> typeHandlerClass = resolveClass(handlerTypeName);
          if (javaTypeClass != null) {
            if (jdbcType == null) {
              typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);
            } else {
              typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);
            }
          } else {
            typeHandlerRegistry.register(typeHandlerClass);
          }
        }
      }
    }
  }
```

#### mappers

mapper文件是mybatis框架的核心之处，所有的用户sql语句都编写在mapper文件中，所以理解mapper文件对于所有的开发人员来说都是必备的要求。

```
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        // 如果要同时使用package自动扫描和通过mapper明确指定要加载的mapper，一定要确保package自动扫描的范围不包含明确指定的mapper，否则在通过package扫描的interface的时候，尝试加载对应xml文件的loadXmlResource()的逻辑中出现判重出错，报org.apache.ibatis.binding.BindingException异常，即使xml文件中包含的内容和mapper接口中包含的语句不重复也会出错，包括加载mapper接口时自动加载的xml mapper也一样会出错。
        if ("package".equals(child.getName())) {
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

mybatis提供了两类配置mapper的方法，第一类是使用package自动搜索的模式，这样指定package下所有接口都会被注册为mapper，例如：

```
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

另外一类是明确指定mapper，这又可以通过resource、url或者class进行细分。

```
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

需要注意的是，如果要同时使用package自动扫描和通过mapper明确指定要加载的mapper，则必须先声明mapper，然后声明package，否则DTD校验会失败。同时一定要确保package自动扫描的范围不包含明确指定的mapper，否则在通过package扫描的interface的时候，尝试加载对应xml文件的loadXmlResource()的逻辑中出现判重出错，报org.apache.ibatis.binding.BindingException异常。

　　对于通过package加载的mapper文件，调用mapperRegistry.addMappers(packageName);进行加载，其核心逻辑在org.apache.ibatis.binding.MapperRegistry中，对于每个找到的接口或者mapper文件，最后调用用XMLMapperBuilder进行具体解析。
　　对于明确指定的mapper文件或者mapper接口，则主要使用XMLMapperBuilder进行具体解析。

## 映射mapper

### MapperAnnotationBuilder注解解析

首先加载mapper映射文件

```
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

我们先来看通过package自动搜索加载的方式，它的范围由addMappers的参数packageName指定的包名以及父类superType确定

```
  /**
   * @since 3.2.2
   */
  public void addMappers(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    for (Class<?> mapperClass : mapperSet) {
      addMapper(mapperClass);
    }
  }

  /**
   * @since 3.2.2
   */
  public void addMappers(String packageName) {
    addMappers(packageName, Object.class);
  }
```

如何addMapper

```
public <T> void addMapper(Class<T> type) {
// 对于mybatis mapper接口文件，必须是interface，不能是class
  if (type.isInterface()) {
  // 判重，确保只会加载一次不会被覆盖
    if (hasMapper(type)) {
      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
    }
    boolean loadCompleted = false;
    try {
    // 为mapper接口创建一个MapperProxyFactory代理
      knownMappers.put(type, new MapperProxyFactory<>(type));
      // It's important that the type is added before the parser is run
      // otherwise the binding may automatically be attempted by the
      // mapper parser. If the type is already known, it won't try.
      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
      parser.parse();
      loadCompleted = true;
    } finally {
      if (!loadCompleted) {
      //剔除解析出现异常的接口
        knownMappers.remove(type);
      }
    }
  }
}
```

knownMappers是MapperRegistry的主要字段，维护了Mapper接口和代理类的映射关系,key是mapper接口类，value是MapperProxyFactory，其定义如下：

```
public class MapperRegistry {

  private final Configuration config;
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();

  public MapperRegistry(Configuration config) {
    this.config = config;
  }

  @SuppressWarnings("unchecked")
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }
  ...
  
```

从定义看出，MapperProxyFactory主要是维护mapper接口的方法与对应mapper文件中具体CRUD节点的关联关系。其中每个Method与对应MapperMethod维护在一起。MapperMethod是mapper中具体映射语句节点的内部表示。

从源码可看出MapperRegistry就是Mapper的注册中心，有两个属性一个是全局配置Configuration还有一个是已经加载过的mapper集合 knownMappers

新增一个mapper的方法就是addMapper，就是向knownsMappers集合中put一条新的mapper记录，key就是mapper的类名全称，value是这个mapper的代理工厂；

分析到这里，发现Configuration对象初始化的时候会解析所有的xml文件中配置的所有mapper接口，并添加到Configuration的mapper集合knowMappers中，但是貌似还没有MappedStatement的影子，也没有看到哪里解析了mapper.xml配置。

首先为mapper接口创建MapperProxyFactory，然后创建MapperAnnotationBuilder进行具体的解析，MapperAnnotationBuilder在解析前的构造器中完成了下列工作：

```
  static {
    SQL_ANNOTATION_TYPES.add(Select.class);
    SQL_ANNOTATION_TYPES.add(Insert.class);
    SQL_ANNOTATION_TYPES.add(Update.class);
    SQL_ANNOTATION_TYPES.add(Delete.class);

    SQL_PROVIDER_ANNOTATION_TYPES.add(SelectProvider.class);
    SQL_PROVIDER_ANNOTATION_TYPES.add(InsertProvider.class);
    SQL_PROVIDER_ANNOTATION_TYPES.add(UpdateProvider.class);
    SQL_PROVIDER_ANNOTATION_TYPES.add(DeleteProvider.class);
  }

  public MapperAnnotationBuilder(Configuration configuration, Class<?> type) {
    String resource = type.getName().replace('.', '/') + ".java (best guess)";
    this.assistant = new MapperBuilderAssistant(configuration, resource);
    this.configuration = configuration;
    this.type = type;
  }
```

其中的MapperBuilderAssistant和XMLConfigBuilder一样，都是继承于BaseBuilder。Select.class/Insert.class等注解指示该方法对应的真实sql语句类型分别是select/insert。
SelectProvider.class/InsertProvider.class主要用于动态SQL，它们允许你指定一个类名和一个方法在具体执行时返回要运行的SQL语句。MyBatis会实例化这个类，然后执行指定的方法。

MapperBuilderAssistant初始化完成之后，就调用build.parse()进行具体的mapper接口文件加载与解析，如下所示：

```
  public void parse() {
    String resource = type.toString();
    //首先根据mapper接口的字符串表示判断是否已经加载,避免重复加载,正常情况下应该都没有加载
    if (!configuration.isResourceLoaded(resource)) {
      loadXmlResource();
      configuration.addLoadedResource(resource);
      // 每个mapper文件自成一个namespace，通常自动匹配就是这么来的，约定俗成代替人工设置最简化常见的开发
      assistant.setCurrentNamespace(type.getName());
      parseCache();
      parseCacheRef();
      Method[] methods = type.getMethods();
      for (Method method : methods) {
        try {
          // issue #237
          if (!method.isBridge()) {
            parseStatement(method);
          }
        } catch (IncompleteElementException e) {
          configuration.addIncompleteMethod(new MethodResolver(this, method));
        }
      }
    }
    parsePendingMethods();
  }
```

loadXmlResource

```
  private void loadXmlResource() {
    // Spring may not know the real resource name so we check a flag
    // to prevent loading again a resource twice
    // this flag is set at XMLMapperBuilder#bindMapperForNamespace
    if (!configuration.isResourceLoaded("namespace:" + type.getName())) {
      String xmlResource = type.getName().replace('.', '/') + ".xml";
      // #1347
      InputStream inputStream = type.getResourceAsStream("/" + xmlResource);
      if (inputStream == null) {
        // Search XML mapper that is not in the module but in the classpath.
        try {
          inputStream = Resources.getResourceAsStream(type.getClassLoader(), xmlResource);
        } catch (IOException e2) {
          // ignore, resource is not required
        }
      }
      if (inputStream != null) {
        XMLMapperBuilder xmlParser = new XMLMapperBuilder(inputStream, assistant.getConfiguration(), xmlResource, configuration.getSqlFragments(), type.getName());
        xmlParser.parse();
      }
    }
  }
```

根据package自动搜索加载的时候，约定俗称从classpath下加载接口的完整名，比如org.mybatis.example.mapper.BlogMapper，就加载org/mybatis/example/mapper/BlogMapper.xml。对于从package和class进来的mapper，如果找不到对应的文件，就忽略，因为这种情况下是允许SQL语句作为注解打在接口上的，所以xml文件不是必须的，而对于直接声明的xml mapper文件，如果找不到的话会抛出IOException异常而终止，这在使用注解模式的时候需要注意。加载到对应的mapper.xml文件后，调用XMLMapperBuilder进行解析。在创建XMLMapperBuilder时，我们发现用到了configuration.getSqlFragments()，这就是我们在mapper文件中经常使用的可以被包含在其他语句中的SQL片段，但是我们并没有初始化过，所以很有可能它是在解析过程中动态添加的，创建了XMLMapperBuilder之后，在调用其parse()接口进行具体xml的解析，这和mybatis-config的逻辑基本上是一致的思路。再来看XMLMapperBuilder的初始化逻辑：

### XMLMapperBuilder解析XML

Mapper文件的解析主要由XMLMapperBuilder类完成

```
  public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
      configurationElement(parser.evalNode("/mapper"));
      configuration.addLoadedResource(resource);
      bindMapperForNamespace();
    }

    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
  }
```

解析mapper的核心又在configurationElement中

```
  private void configurationElement(XNode context) {
    try {
      String namespace = context.getStringAttribute("namespace");
      if (namespace == null || namespace.equals("")) {
        throw new BuilderException("Mapper's namespace cannot be empty");
      }
      builderAssistant.setCurrentNamespace(namespace);
      cacheRefElement(context.evalNode("cache-ref"));
      cacheElement(context.evalNode("cache"));
      parameterMapElement(context.evalNodes("/mapper/parameterMap"));
      resultMapElements(context.evalNodes("/mapper/resultMap"));
      sqlElement(context.evalNodes("/mapper/sql"));
      buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
    }
  }
```

#### cache-ref

解析缓存参照cache-ref。参照缓存顾名思义，就是共用其他缓存的设置。

```
  private void cacheRefElement(XNode context) {
    if (context != null) {
      configuration.addCacheRef(builderAssistant.getCurrentNamespace(), context.getStringAttribute("namespace"));
      CacheRefResolver cacheRefResolver = new CacheRefResolver(builderAssistant, context.getStringAttribute("namespace"));
      try {
        cacheRefResolver.resolveCacheRef();
      } catch (IncompleteElementException e) {
        configuration.addIncompleteCacheRef(cacheRefResolver);
      }
    }
  }
```

缓存参考因为通过namespace指向其他的缓存。所以会出现第一次解析的时候指向的缓存还不存在的情况，所以需要在所有的mapper文件加载完成后进行二次处理，不仅仅是缓存参考，其他的CRUD也一样。所以在XMLMapperBuilder.configuration中有很多的incompleteXXX，这种设计模式类似于JVM GC中的mark and sweep，标记、然后处理。所以当捕获到IncompleteElementException异常时，没有终止执行，而是将指向的缓存不存在的cacheRefResolver添加到configuration.incompleteCacheRef中。

#### cache

```
  private void cacheElement(XNode context) {
    if (context != null) {
      String type = context.getStringAttribute("type", "PERPETUAL");
      Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
      String eviction = context.getStringAttribute("eviction", "LRU");
      Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
      Long flushInterval = context.getLongAttribute("flushInterval");
      Integer size = context.getIntAttribute("size");
      boolean readWrite = !context.getBooleanAttribute("readOnly", false);
      boolean blocking = context.getBooleanAttribute("blocking", false);
      Properties props = context.getChildrenAsProperties();
      builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
    }
  }
```

默认情况下，mybatis使用的是永久缓存PerpetualCache，读取或设置各个属性默认值之后，调用builderAssistant.useNewCache构建缓存，其中的CacheBuilder使用了build模式（在effective里面，建议有4个以上可选属性时，应该为对象提供一个builder便于使用），只要实现org.apache.ibatis.cache.Cache接口，就是合法的mybatis缓存。

#### parameterMap

```
  private void parameterMapElement(List<XNode> list) {
    for (XNode parameterMapNode : list) {
      String id = parameterMapNode.getStringAttribute("id");
      String type = parameterMapNode.getStringAttribute("type");
      Class<?> parameterClass = resolveClass(type);
      List<XNode> parameterNodes = parameterMapNode.evalNodes("parameter");
      List<ParameterMapping> parameterMappings = new ArrayList<>();
      for (XNode parameterNode : parameterNodes) {
        String property = parameterNode.getStringAttribute("property");
        String javaType = parameterNode.getStringAttribute("javaType");
        String jdbcType = parameterNode.getStringAttribute("jdbcType");
        String resultMap = parameterNode.getStringAttribute("resultMap");
        String mode = parameterNode.getStringAttribute("mode");
        String typeHandler = parameterNode.getStringAttribute("typeHandler");
        Integer numericScale = parameterNode.getIntAttribute("numericScale");
        ParameterMode modeEnum = resolveParameterMode(mode);
        Class<?> javaTypeClass = resolveClass(javaType);
        JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);
        Class<? extends TypeHandler<?>> typeHandlerClass = resolveClass(typeHandler);
        ParameterMapping parameterMapping = builderAssistant.buildParameterMapping(parameterClass, property, javaTypeClass, jdbcTypeEnum, resultMap, modeEnum, typeHandlerClass, numericScale);
        parameterMappings.add(parameterMapping);
      }
      builderAssistant.addParameterMap(id, parameterClass, parameterMappings);
    }
  }
```

目前已经不推荐使用参数映射，而是直接使用内联参数。

#### resultMap

结果集映射早期版本可以说是用的最多的辅助节点了，不过有了mapUnderscoreToCamelCase属性之后，如果命名规范控制做的好的话，resultMap也是可以省略的。每个mapper文件可以有多个结果集映射。

```
 private void resultMapElements(List<XNode> list) throws Exception {
    for (XNode resultMapNode : list) {
      try {
        resultMapElement(resultMapNode);
      } catch (IncompleteElementException e) {
        // ignore, it will be retried
        // 在内部实现中将未完成的元素添加到configuration.incomplete中了
      }
    }
  }

  private ResultMap resultMapElement(XNode resultMapNode) throws Exception {
    return resultMapElement(resultMapNode, Collections.<ResultMapping> emptyList());
  }

  private ResultMap resultMapElement(XNode resultMapNode, List<ResultMapping> additionalResultMappings) throws Exception {
    ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());
    String id = resultMapNode.getStringAttribute("id",
        resultMapNode.getValueBasedIdentifier());
    String type = resultMapNode.getStringAttribute("type",
        resultMapNode.getStringAttribute("ofType",
            resultMapNode.getStringAttribute("resultType",
                resultMapNode.getStringAttribute("javaType"))));
    String extend = resultMapNode.getStringAttribute("extends");
    Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");
    Class<?> typeClass = resolveClass(type);
    Discriminator discriminator = null;
    List<ResultMapping> resultMappings = new ArrayList<ResultMapping>();
    resultMappings.addAll(additionalResultMappings);
    List<XNode> resultChildren = resultMapNode.getChildren();
    for (XNode resultChild : resultChildren) {
      if ("constructor".equals(resultChild.getName())) {
        processConstructorElement(resultChild, typeClass, resultMappings);
      } else if ("discriminator".equals(resultChild.getName())) {
        discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
      } else {
        List<ResultFlag> flags = new ArrayList<ResultFlag>();
        if ("id".equals(resultChild.getName())) {
          flags.add(ResultFlag.ID);
        }
        resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));
      }
    }
    ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);
    try {
      return resultMapResolver.resolve();
    } catch (IncompleteElementException  e) {
      configuration.addIncompleteResultMap(resultMapResolver);
      throw e;
    }
  }

  private void processConstructorElement(XNode resultChild, Class<?> resultType, List<ResultMapping> resultMappings) throws Exception {
    List<XNode> argChildren = resultChild.getChildren();
    for (XNode argChild : argChildren) {
      List<ResultFlag> flags = new ArrayList<ResultFlag>();
      flags.add(ResultFlag.CONSTRUCTOR);
      if ("idArg".equals(argChild.getName())) {
        flags.add(ResultFlag.ID);
      }
      resultMappings.add(buildResultMappingFromContext(argChild, resultType, flags));
    }
  }

  private Discriminator processDiscriminatorElement(XNode context, Class<?> resultType, List<ResultMapping> resultMappings) throws Exception {
    String column = context.getStringAttribute("column");
    String javaType = context.getStringAttribute("javaType");
    String jdbcType = context.getStringAttribute("jdbcType");
    String typeHandler = context.getStringAttribute("typeHandler");
    Class<?> javaTypeClass = resolveClass(javaType);
    @SuppressWarnings("unchecked")
    Class<? extends TypeHandler<?>> typeHandlerClass = (Class<? extends TypeHandler<?>>) resolveClass(typeHandler);
    JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);
    Map<String, String> discriminatorMap = new HashMap<String, String>();
    for (XNode caseChild : context.getChildren()) {
      String value = caseChild.getStringAttribute("value");
      String resultMap = caseChild.getStringAttribute("resultMap", processNestedResultMappings(caseChild, resultMappings));
      discriminatorMap.put(value, resultMap);
    }
    return builderAssistant.buildDiscriminator(resultType, column, javaTypeClass, jdbcTypeEnum, typeHandlerClass, discriminatorMap);
  }
```

我们先来看下ResultMapping的定义：

```
public class ResultMapping {

  private Configuration configuration;
  private String property;
  private String column;
  private Class<?> javaType;
  private JdbcType jdbcType;
  private TypeHandler<?> typeHandler;
  private String nestedResultMapId;
  private String nestedQueryId;
  private Set<String> notNullColumns;
  private String columnPrefix;
  private List<ResultFlag> flags;
  private List<ResultMapping> composites;
  private String resultSet;
  private String foreignColumn;
  private boolean lazy;
  ...
}
```

总体逻辑是先解析resultMap节点本身，然后解析子节点构造器，鉴别器discriminator，id。最后组装成真正的resultMappings。

构造器的解析比较简单，除了遍历构造参数外，还可以构造器参数的ID也识别出来。最后调用buildResultMappingFromContext建立具体的resultMap。buildResultMappingFromContext是个公共工具方法，会被反复使用，我们来看下它的具体实现

```
private ResultMapping buildResultMappingFromContext(XNode context, Class<?> resultType, List<ResultFlag> flags) throws Exception {
    String property;
    if (flags.contains(ResultFlag.CONSTRUCTOR)) {
      property = context.getStringAttribute("name");
    } else {
      property = context.getStringAttribute("property");
    }
    String column = context.getStringAttribute("column");
    String javaType = context.getStringAttribute("javaType");
    String jdbcType = context.getStringAttribute("jdbcType");
    String nestedSelect = context.getStringAttribute("select");
    // resultMap中可以包含association或collection复合类型,这些复合类型可以使用外部定义的公用resultMap或者内嵌resultMap, 所以这里的处理逻辑是如果有resultMap就获取resultMap,如果没有,那就动态生成一个。如果自动生成的话，他的resultMap id通过调用XNode.getValueBasedIdentifier()来获得
    String nestedResultMap = context.getStringAttribute("resultMap",
        processNestedResultMappings(context, Collections.<ResultMapping> emptyList()));
    String notNullColumn = context.getStringAttribute("notNullColumn");
    String columnPrefix = context.getStringAttribute("columnPrefix");
    String typeHandler = context.getStringAttribute("typeHandler");
    String resultSet = context.getStringAttribute("resultSet");
    String foreignColumn = context.getStringAttribute("foreignColumn");
    boolean lazy = "lazy".equals(context.getStringAttribute("fetchType", configuration.isLazyLoadingEnabled() ? "lazy" : "eager"));
    Class<?> javaTypeClass = resolveClass(javaType);
    @SuppressWarnings("unchecked")
    Class<? extends TypeHandler<?>> typeHandlerClass = (Class<? extends TypeHandler<?>>) resolveClass(typeHandler);
    JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);
    return builderAssistant.buildResultMapping(resultType, property, column, javaTypeClass, jdbcTypeEnum, nestedSelect, nestedResultMap, notNullColumn, columnPrefix, typeHandlerClass, flags, resultSet, foreignColumn, lazy);
  }
```

上述过程主要用于获取各个属性，其中唯一值得注意的是processNestedResultMappings，它用于解析包含的association或collection复合类型,这些复合类型可以使用外部定义的公用resultMap或者内嵌resultMap, 所以这里的处理逻辑是如果是外部resultMap就获取对应resultMap的名称,如果没有,那就动态生成一个。如果自动生成的话，其resultMap id通过调用XNode.getValueBasedIdentifier()来获得。由于colletion和association、discriminator里面还可以包含复合类型，所以将进行递归解析直到所有的子元素都为基本列位置，它在使用层面的目的在于将关系模型映射为对象树模型。

esultMap.Builder.build()进行最后的resultMap构建。build应该来说是真正创建根resultMap对象的逻辑入口。我们先看下ResultMap的定义：

```
public class ResultMap {
  private Configuration configuration;

  // resultMap的id属性
  private String id;

  // resultMap的type属性,有可能是alias
  private Class<?> type;

  // resultMap下的所有节点
  private List<ResultMapping> resultMappings;

  // resultMap下的id节点比如<id property="id" column="user_id" />
  private List<ResultMapping> idResultMappings;

  // resultMap下的构造器节点<constructor>
  private List<ResultMapping> constructorResultMappings;

  // resultMap下的property节点比如<result property="password" column="hashed_password"/>
  private List<ResultMapping> propertyResultMappings;

  //映射的列名
  private Set<String> mappedColumns;

  // 映射的javaBean属性名,所有映射不管是id、构造器或者普通的
  private Set<String> mappedProperties;

  // 鉴别器
  private Discriminator discriminator;

  // 是否有嵌套的resultMap比如association或者collection
  private boolean hasNestedResultMaps;

  // 是否有嵌套的查询,也就是select属性
  private boolean hasNestedQueries;

  // autoMapping属性,这个属性会覆盖全局的属性autoMappingBehavior
  private Boolean autoMapping;
  ...
}
其中定义了节点下所有的子元素,以及必要的辅助属性，包括最重要的各种ResultMapping。
再来看下build的实现：

    public ResultMap build() {
      if (resultMap.id == null) {
        throw new IllegalArgumentException("ResultMaps must have an id");
      }
      resultMap.mappedColumns = new HashSet<String>();
      resultMap.mappedProperties = new HashSet<String>();
      resultMap.idResultMappings = new ArrayList<ResultMapping>();
      resultMap.constructorResultMappings = new ArrayList<ResultMapping>();
      resultMap.propertyResultMappings = new ArrayList<ResultMapping>();
      final List<String> constructorArgNames = new ArrayList<String>();
      for (ResultMapping resultMapping : resultMap.resultMappings) {
        // 判断是否有嵌套查询, nestedQueryId是在buildResultMappingFromContext的时候通过读取节点的select属性得到的
        resultMap.hasNestedQueries = resultMap.hasNestedQueries || resultMapping.getNestedQueryId() != null;

        // 判断是否嵌套了association或者collection, nestedResultMapId是在buildResultMappingFromContext的时候通过读取节点的resultMap属性得到的或者内嵌resultMap的时候自动计算得到的。注：这里的resultSet没有地方set进来,DTD中也没有看到，不确定是不是有意预留的，但是association/collection的子元素中倒是有声明
        resultMap.hasNestedResultMaps = resultMap.hasNestedResultMaps || (resultMapping.getNestedResultMapId() != null && resultMapping.getResultSet() == null);

        // 获取column属性, 包括复合列，复合列是在org.apache.ibatis.builder.MapperBuilderAssistant.parseCompositeColumnName(String)中解析的。所有的数据库列都被按顺序添加到resultMap.mappedColumns中
        final String column = resultMapping.getColumn();
        if (column != null) {
          resultMap.mappedColumns.add(column.toUpperCase(Locale.ENGLISH));
        } else if (resultMapping.isCompositeResult()) {
          for (ResultMapping compositeResultMapping : resultMapping.getComposites()) {
            final String compositeColumn = compositeResultMapping.getColumn();
            if (compositeColumn != null) {
              resultMap.mappedColumns.add(compositeColumn.toUpperCase(Locale.ENGLISH));
            }
          }
        }

        // 所有映射的属性都被按顺序添加到resultMap.mappedProperties中,ID单独存储
        final String property = resultMapping.getProperty();
        if(property != null) {
          resultMap.mappedProperties.add(property);
        }

        // 所有映射的构造器被按顺序添加到resultMap.constructorResultMappings 
        // 如果本元素具有CONSTRUCTOR标记,则添加到构造函数参数列表,否则添加到普通属性映射列表resultMap.propertyResultMappings
        if (resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR)) {
          resultMap.constructorResultMappings.add(resultMapping);
          if (resultMapping.getProperty() != null) {
            constructorArgNames.add(resultMapping.getProperty());
          }
        } else {
          resultMap.propertyResultMappings.add(resultMapping);
        }

        // 如果本元素具有ID标记, 则添加到ID映射列表resultMap.idResultMappings
        if (resultMapping.getFlags().contains(ResultFlag.ID)) {
          resultMap.idResultMappings.add(resultMapping);
        }
      }

      // 如果没有声明ID属性,就把所有属性都作为ID属性
      if (resultMap.idResultMappings.isEmpty()) {
        resultMap.idResultMappings.addAll(resultMap.resultMappings);
      }

      // 根据声明的构造器参数名和类型,反射声明的类,检查其中是否包含对应参数名和类型的构造器,如果不存在匹配的构造器,就抛出运行时异常,这是为了确保运行时不会出现异常
      if (!constructorArgNames.isEmpty()) {
        final List<String> actualArgNames = argNamesOfMatchingConstructor(constructorArgNames);
        if (actualArgNames == null) {
          throw new BuilderException("Error in result map '" + resultMap.id
              + "'. Failed to find a constructor in '"
              + resultMap.getType().getName() + "' by arg names " + constructorArgNames
              + ". There might be more info in debug log.");
        }
        //构造器参数排序
        Collections.sort(resultMap.constructorResultMappings, new Comparator<ResultMapping>() {
          @Override
          public int compare(ResultMapping o1, ResultMapping o2) {
            int paramIdx1 = actualArgNames.indexOf(o1.getProperty());
            int paramIdx2 = actualArgNames.indexOf(o2.getProperty());
            return paramIdx1 - paramIdx2;
          }
        });
      }
      // lock down collections
      // 为了避免用于无意或者有意事后修改resultMap的内部结构, 克隆一个不可修改的集合提供给用户
      resultMap.resultMappings = Collections.unmodifiableList(resultMap.resultMappings);
      resultMap.idResultMappings = Collections.unmodifiableList(resultMap.idResultMappings);
      resultMap.constructorResultMappings = Collections.unmodifiableList(resultMap.constructorResultMappings);
      resultMap.propertyResultMappings = Collections.unmodifiableList(resultMap.propertyResultMappings);
      resultMap.mappedColumns = Collections.unmodifiableSet(resultMap.mappedColumns);
      return resultMap;
    }

    private List<String> argNamesOfMatchingConstructor(List<String> constructorArgNames) {
      Constructor<?>[] constructors = resultMap.type.getDeclaredConstructors();
      for (Constructor<?> constructor : constructors) {
        Class<?>[] paramTypes = constructor.getParameterTypes();
        if (constructorArgNames.size() == paramTypes.length) {
          List<String> paramNames = getArgNames(constructor);
          if (constructorArgNames.containsAll(paramNames)
              && argTypesMatch(constructorArgNames, paramTypes, paramNames)) {
            return paramNames;
          }
        }
      }
      return null;
    }

    private boolean argTypesMatch(final List<String> constructorArgNames,
        Class<?>[] paramTypes, List<String> paramNames) {
      for (int i = 0; i < constructorArgNames.size(); i++) {
        Class<?> actualType = paramTypes[paramNames.indexOf(constructorArgNames.get(i))];
        Class<?> specifiedType = resultMap.constructorResultMappings.get(i).getJavaType();
        if (!actualType.equals(specifiedType)) {
          if (log.isDebugEnabled()) {
            log.debug("While building result map '" + resultMap.id
                + "', found a constructor with arg names " + constructorArgNames
                + ", but the type of '" + constructorArgNames.get(i)
                + "' did not match. Specified: [" + specifiedType.getName() + "] Declared: ["
                + actualType.getName() + "]");
          }
          return false;
        }
      }
      return true;
    }

    private List<String> getArgNames(Constructor<?> constructor) {
      List<String> paramNames = new ArrayList<String>();
      List<String> actualParamNames = null;
      final Annotation[][] paramAnnotations = constructor.getParameterAnnotations();
      int paramCount = paramAnnotations.length;
      for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
        String name = null;
        for (Annotation annotation : paramAnnotations[paramIndex]) {
          if (annotation instanceof Param) {
            name = ((Param) annotation).value();
            break;
          }
        }
        if (name == null && resultMap.configuration.isUseActualParamName() && Jdk.parameterExists) {
          if (actualParamNames == null) {
            actualParamNames = ParamNameUtil.getParamNames(constructor);
          }
          if (actualParamNames.size() > paramIndex) {
            name = actualParamNames.get(paramIndex);
          }
        }
        paramNames.add(name != null ? name : "arg" + paramIndex);
      }
      return paramNames;
    }
  }
```

ResultMap构建完成之后，添加到Configuration的resultMaps中。

## SQL的解析

CRUD是SQL的核心部分，也是mybatis针对用户提供的最有价值的部分，所以研究源码有必要对CRUD的完整实现进行分析。不理解这一部分的核心实现，就谈不上对mybatis的实现有深刻理解。主要是从下面开始

```
buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
```

具体代码

```
  private void configurationElement(XNode context) {
    try {
      String namespace = context.getStringAttribute("namespace");
      if (namespace == null || namespace.equals("")) {
        throw new BuilderException("Mapper's namespace cannot be empty");
      }
      builderAssistant.setCurrentNamespace(namespace);
      cacheRefElement(context.evalNode("cache-ref"));
      cacheElement(context.evalNode("cache"));
      parameterMapElement(context.evalNodes("/mapper/parameterMap"));
      resultMapElements(context.evalNodes("/mapper/resultMap"));
      sqlElement(context.evalNodes("/mapper/sql"));
      buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
    }
  }
```

crud的解析从buildStatementFromContext(context.evalNodes(“select|insert|update|delete”));语句开始，透过调用调用链，我们可以得知SQL语句的解析主要在XMLStatementBuilder中实现。

```
  private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
      buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
  }

  private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
      final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
      try {
        statementParser.parseStatementNode();
      } catch (IncompleteElementException e) {
        configuration.addIncompleteStatement(statementParser);
      }
    }
  }
```

在看具体实现之前，我们先大概看下DTD的定义（因为INSERT/UPDATE/DELETE属于一种类型，所以我们以INSERT为例），这样我们就知道整体关系和脉络：

```
<!ELEMENT select (#PCDATA | include | trim | where | set | foreach | choose | if | bind)*>
<!ATTLIST select
id CDATA #REQUIRED
parameterMap CDATA #IMPLIED
parameterType CDATA #IMPLIED
resultMap CDATA #IMPLIED
resultType CDATA #IMPLIED
resultSetType (FORWARD_ONLY | SCROLL_INSENSITIVE | SCROLL_SENSITIVE) #IMPLIED
statementType (STATEMENT|PREPARED|CALLABLE) #IMPLIED
fetchSize CDATA #IMPLIED
timeout CDATA #IMPLIED
flushCache (true|false) #IMPLIED
useCache (true|false) #IMPLIED
databaseId CDATA #IMPLIED
lang CDATA #IMPLIED
resultOrdered (true|false) #IMPLIED
resultSets CDATA #IMPLIED 
>

<!ELEMENT insert (#PCDATA | selectKey | include | trim | where | set | foreach | choose | if | bind)*>
<!ATTLIST insert
id CDATA #REQUIRED
parameterMap CDATA #IMPLIED
parameterType CDATA #IMPLIED
timeout CDATA #IMPLIED
flushCache (true|false) #IMPLIED
statementType (STATEMENT|PREPARED|CALLABLE) #IMPLIED
keyProperty CDATA #IMPLIED
useGeneratedKeys (true|false) #IMPLIED
keyColumn CDATA #IMPLIED
databaseId CDATA #IMPLIED
lang CDATA #IMPLIED
>

<!ELEMENT selectKey (#PCDATA | include | trim | where | set | foreach | choose | if | bind)*>
<!ATTLIST selectKey
resultType CDATA #IMPLIED
statementType (STATEMENT|PREPARED|CALLABLE) #IMPLIED
keyProperty CDATA #IMPLIED
keyColumn CDATA #IMPLIED
order (BEFORE|AFTER) #IMPLIED
databaseId CDATA #IMPLIED
>
```

我们先来看statementParser.parseStatementNode()的实现主体部分。

```
 public void parseStatementNode() {
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");

    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
      return;
    }

    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultType = context.getStringAttribute("resultType");

    // MyBatis 从 3.2 开始支持可插拔的脚本语言，因此你可以在插入一种语言的驱动（language driver）之后来写基于这种语言的动态 SQL 查询。
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    Class<?> resultTypeClass = resolveClass(resultType);

    // 结果集的类型，FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）。
    String resultSetType = context.getStringAttribute("resultSetType");

    // 解析crud语句的类型，mybatis目前支持三种,prepare、硬编码、以及存储过程调用
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);

    String nodeName = context.getNode().getNodeName();

    // 解析SQL命令类型，目前主要有UNKNOWN, INSERT, UPDATE, DELETE, SELECT, FLUSH
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;

    // insert/delete/update后是否刷新缓存
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);

    // select是否使用缓存
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);

    // 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：false。我猜测这个属性为true的意思是查询的结果字段根据定义的嵌套resultMap进行了排序，后面在分析sql执行源码的时候，我们会具体看到他到底是干吗用的
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    // 解析语句中包含的sql片段，也就是
    // <select id="select" resultType="map">
    //  select
    //      field1, field2, field3
    //      <include refid="someinclude"></include>
    // </select>

    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    // Parse selectKey after includes and remove them.
    processSelectKeyNodes(id, parameterTypeClass, langDriver);

    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
    String resultSets = context.getStringAttribute("resultSets");
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    KeyGenerator keyGenerator;
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
      keyGenerator = configuration.getKeyGenerator(keyStatementId);
    } else {
      keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
          configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
          ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered, 
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
  }
```

从DTD可以看出，sql/crud元素里面可以包含这些元素：#PCDATA(文本节点) | include | trim | where | set | foreach | choose | if | bind，除了文本节点外，其他类型的节点都可以嵌套，我们看下解析包含的sql片段的逻辑。

我们来看下MappedStatement的具体构造以及代表SelectKey的sqlSource是如何组装成MappedStatement的。

MappedStatement类位于mybatis包的org.apache.ibatis.mapping目录下，是一个final类型也就是说实例化之后就不允许改变

MappedStatement对象对应Mapper.xml配置文件中的一个select/update/insert/delete节点，描述的就是一条SQL语句，属性如下：

```
public final class MappedStatement {

 private String resource;//mapper配置文件名，如：UserMapper.xml
  private Configuration configuration;//全局配置
  private String id;//节点的id属性加命名空间,如：com.lucky.mybatis.dao.UserMapper.selectByExample
  private Integer fetchSize;
  private Integer timeout;//超时时间
  private StatementType statementType;//操作SQL的对象的类型
  private ResultSetType resultSetType;//结果类型
  private SqlSource sqlSource;//sql语句
  private Cache cache;//缓存
  private ParameterMap parameterMap;
  private List<ResultMap> resultMaps;
  private boolean flushCacheRequired;
  private boolean useCache;//是否使用缓存，默认为true
  private boolean resultOrdered;//结果是否排序
  private SqlCommandType sqlCommandType;//sql语句的类型，如select、update、delete、insert
  private KeyGenerator keyGenerator;
  private String[] keyProperties;
  private String[] keyColumns;
  private boolean hasNestedResultMaps;
  private String databaseId;//数据库ID
  private Log statementLog;
  private LanguageDriver lang;
  private String[] resultSets;
  ...
}
对于每个语句而言，在运行时都需要知道结果映射，是否使用缓存，语句类型，sql文本，超时时间，是否自动生成key等等。为了方便运行时的使用以及高效率，MappedStatement被设计为直接包含了所有这些属性。

  public MappedStatement addMappedStatement(
      String id,
      SqlSource sqlSource,
      StatementType statementType,
      SqlCommandType sqlCommandType,
      Integer fetchSize,
      Integer timeout,
      String parameterMap,
      Class<?> parameterType,
      String resultMap,
      Class<?> resultType,
      ResultSetType resultSetType,
      boolean flushCache,
      boolean useCache,
      boolean resultOrdered,
      KeyGenerator keyGenerator,
      String keyProperty,
      String keyColumn,
      String databaseId,
      LanguageDriver lang,
      String resultSets) {

    if (unresolvedCacheRef) {
      throw new IncompleteElementException("Cache-ref not yet resolved");
    }

    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;

    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        .resource(resource)
        .fetchSize(fetchSize)
        .timeout(timeout)
        .statementType(statementType)
        .keyGenerator(keyGenerator)
        .keyProperty(keyProperty)
        .keyColumn(keyColumn)
        .databaseId(databaseId)
        .lang(lang)
        .resultOrdered(resultOrdered)
        .resultSets(resultSets)
        .resultMaps(getStatementResultMaps(resultMap, resultType, id))
        .resultSetType(resultSetType)
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
        .useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);

    // 创建语句参数映射
    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
      statementBuilder.parameterMap(statementParameterMap);
    }

    MappedStatement statement = statementBuilder.build();
    configuration.addMappedStatement(statement);
    return statement;
  }
```

其中StatementType指操作SQL对象的类型，是个枚举类型，值分别为：

STATEMENT（直接操作SQL，不进行预编译），

PREPARED（预处理参数，进行预编译，获取数据），

CALLABLE（执行存储过程）

ResultSetType指返回结果集的类型，也是个枚举类型，值分别为：

FORWARD_ONLY：结果集的游标只能向下滚动

SCROLL_INSENSITIVE：结果集的游标可以上下移动，当数据库变化时当前结果集不变

SCROLL_SENSITIVE：结果集客自由滚动，数据库变化时当前结果集同步改变

言归正传，现在我们知道一个MappedStatement对象对应一个mapper.xml中的一个SQL节点，而Mapper.xml文件是初始化Configuration对象的时候进行解析加载的，则说明MappedStatement对象就是在初始化Configuration对象的时候创建的，并且是final类型不可更改。

## MyBatis 四大核心对象

ParameterHandler：处理SQL的参数对象

ResultSetHandler：处理SQL的返回结果集

StatementHandler：数据库的处理对象，用于执行SQL语句

Executor：MyBatis的执行器，用于执行增删改查操作

### ParameterHandler参数处理器

ParameterHandler接口是参数处理器，位于mybatis包的org.apache.ibatis.executor.parameter下，源码如下：

```
public interface ParameterHandler {

Object getParameterObject();//获取参数

void setParameters(PreparedStatement ps)//设置参数
throws SQLException;

}
```

可见ParameterHandler接口只有简单的两个方法，一个是获取参数一个是设置参数。ParameterHandler接口默认实现类是DefaultParameterHandler，主要源码如下：

```
public class DefaultParameterHandler implements ParameterHandler {

private final TypeHandlerRegistry typeHandlerRegistry;

private final MappedStatement mappedStatement;
private final Object parameterObject;
private BoundSql boundSql;
private Configuration configuration;

public DefaultParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
this.mappedStatement = mappedStatement;
this.configuration = mappedStatement.getConfiguration();
this.typeHandlerRegistry = mappedStatement.getConfiguration().getTypeHandlerRegistry();
this.parameterObject = parameterObject;//设置参数
this.boundSql = boundSql;
}

@Override
public Object getParameterObject() {
return parameterObject;//返回参数
}

@Override
public void setParameters(PreparedStatement ps) {
ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();//获取所有参数，ParameterMapping是jdbc和java类型之间的对应关系
if (parameterMappings != null) {
//遍历所有参数，将java 类型设置成jdbc类型
for (int i = 0; i < parameterMappings.size(); i++) {
ParameterMapping parameterMapping = parameterMappings.get(i);
if (parameterMapping.getMode() != ParameterMode.OUT) {
Object value;
String propertyName = parameterMapping.getProperty();
if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
value = boundSql.getAdditionalParameter(propertyName);
} else if (parameterObject == null) {
value = null;
} else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
value = parameterObject;
} else {
MetaObject metaObject = configuration.newMetaObject(parameterObject);
value = metaObject.getValue(propertyName);
}
TypeHandler typeHandler = parameterMapping.getTypeHandler();
JdbcType jdbcType = parameterMapping.getJdbcType();
if (value == null && jdbcType == null) {
jdbcType = configuration.getJdbcTypeForNull();
}
try {
typeHandler.setParameter(ps, i + 1, value, jdbcType);
} catch (TypeException e) {
throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
} catch (SQLException e) {
throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
}
}
}
}
}

}
```

而ParameterHandler的初始化同样也是在Configuration中实现的，代码如下：

```
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
   ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
   parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
   return parameterHandler;
 }
```

### StatementHandler 语句处理器

接口的作用是statement处理器，位于mybatis包的org.apache.ibatis.executor.statement目录下，源码如下：

```
package org.apache.ibatis.executor.statement;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;

import org.apache.ibatis.executor.parameter.ParameterHandler;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.session.ResultHandler;

public interface StatementHandler {

  //sql预编译，构建Statement对象
  Statement prepare(Connection connection)
      throws SQLException;

  //对prepare方法构建的预编译的SQL进行参数的设置
  void parameterize(Statement statement)
      throws SQLException;

  //批量处理
  void batch(Statement statement)
      throws SQLException;

  //执行预编译后的SQL--update语句
  int update(Statement statement)
      throws SQLException;

  //执行预编译后的SQL--select语句
  <E> List<E> query(Statement statement, ResultHandler resultHandler)
      throws SQLException;

  //获取SQL封装类BoundSql对象
  BoundSql getBoundSql();

  //获取参数处理器对象
  ParameterHandler getParameterHandler();

}
```

可见StatementHandler的作用就是先通过prepare方法构建一个Statement对象，然后再调用其他方法对Statement对象进行处理

StatementHandler和Executor类似，StatementHandler也有两个实现类，BaseStatementHandler和RoutingStatementHandler

而BaseStatement又有三个子类实现它的抽象方法，下面再挨个分析，先看最简单的BaseStatementHandler

**BaseStatementHandler解析**

BaseStatementHandler是一个抽象父类，有三个子类继承于它，源码如下：

```
package org.apache.ibatis.executor.statement;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

import org.apache.ibatis.executor.ErrorContext;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.executor.ExecutorException;
import org.apache.ibatis.executor.keygen.KeyGenerator;
import org.apache.ibatis.executor.parameter.ParameterHandler;
import org.apache.ibatis.executor.resultset.ResultSetHandler;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.reflection.factory.ObjectFactory;
import org.apache.ibatis.session.Configuration;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;
import org.apache.ibatis.type.TypeHandlerRegistry;

public abstract class BaseStatementHandler implements StatementHandler {

  protected final Configuration configuration;//全局配置
  protected final ObjectFactory objectFactory;//对象工厂
  protected final TypeHandlerRegistry typeHandlerRegistry;
  protected final ResultSetHandler resultSetHandler;//结果集处理器
  protected final ParameterHandler parameterHandler;//参数处理器

  protected final Executor executor;//执行器
  protected final MappedStatement mappedStatement;//mapper的SQL对象
  protected final RowBounds rowBounds;//分页参数

  protected BoundSql boundSql;//sql封装对象

  //构造方法
  protected BaseStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    this.configuration = mappedStatement.getConfiguration();
    this.executor = executor;
    this.mappedStatement = mappedStatement;
    this.rowBounds = rowBounds;

    this.typeHandlerRegistry = configuration.getTypeHandlerRegistry();
    this.objectFactory = configuration.getObjectFactory();

    if (boundSql == null) { // issue #435, get the key before calculating the statement
      generateKeys(parameterObject);
      boundSql = mappedStatement.getBoundSql(parameterObject);
    }

    this.boundSql = boundSql;

    this.parameterHandler = configuration.newParameterHandler(mappedStatement, parameterObject, boundSql);
    this.resultSetHandler = configuration.newResultSetHandler(executor, mappedStatement, rowBounds, parameterHandler, resultHandler, boundSql);
  }

  //返回boundSql
  public BoundSql getBoundSql() {
    return boundSql;
  }

  //返回parameterHandler
  public ParameterHandler getParameterHandler() {
    return parameterHandler;
  }

  //预编译SQL语句
  public Statement prepare(Connection connection) throws SQLException {
    ErrorContext.instance().sql(boundSql.getSql());
    Statement statement = null;
    try {
      statement = instantiateStatement(connection);//调用抽象方法构建statement对象是，但是没有具体实现，而是交给其子类去实现
      setStatementTimeout(statement);//设置statement超时时间
      setFetchSize(statement);//设置statement的fetchSize
      return statement;
    } catch (SQLException e) {
      closeStatement(statement);
      throw e;
    } catch (Exception e) {
      closeStatement(statement);
      throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
    }
  }

  protected abstract Statement instantiateStatement(Connection connection) throws SQLException;

  //给statement对象设置timeout
  protected void setStatementTimeout(Statement stmt) throws SQLException {
    Integer timeout = mappedStatement.getTimeout();
    Integer defaultTimeout = configuration.getDefaultStatementTimeout();
    if (timeout != null) {
      stmt.setQueryTimeout(timeout);
    } else if (defaultTimeout != null) {
      stmt.setQueryTimeout(defaultTimeout);
    }
  }

  //给statement对象设置fetchSize
  protected void setFetchSize(Statement stmt) throws SQLException {
    Integer fetchSize = mappedStatement.getFetchSize();
    if (fetchSize != null) {
      stmt.setFetchSize(fetchSize);
    }
  }

  //关闭statement
  protected void closeStatement(Statement statement) {
    try {
      if (statement != null) {
        statement.close();
      }
    } catch (SQLException e) {
      //ignore
    }
  }

   //根据参数对象生成key
  protected void generateKeys(Object parameter) {
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    ErrorContext.instance().store();
    keyGenerator.processBefore(executor, mappedStatement, null, parameter);
    ErrorContext.instance().recall();
  }

}
```

可以看出BaseStatementHandler只是实现了StatementHandler的三个方法，其中getBoundSql和getParameterHandler方法只是返回了由构造方法初始化的boundSql和parameterHandler属性，

而prepare方法是用于构建Statement对象的，但是BaseStatementHandler只是调用了自身的抽象方法instantiateStatement来创建，然后对statement对象进行其他处理，但是创建的过程则没有实现，而是交给了其子类去实现。

BaseStatementHandler有三个子类，分别为：
SimpleStatememtHandler:最简单的StatementHandler，处理不带参数运行的SQL
PreparedStatementHandler:预处理Statement的handler，处理带参数允许的SQL
CallableStatementHandler：存储过程的Statement的handler，处理存储过程SQL

先来看最简单的SimpleStatementHandler，它继承于BaseStatementHandler，所以它需要实现StatementHandler的接口，还需要重写父类BaseStatementHandler的抽象方法，源码如下：

```
package org.apache.ibatis.executor.statement;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;

import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.executor.keygen.Jdbc3KeyGenerator;
import org.apache.ibatis.executor.keygen.KeyGenerator;
import org.apache.ibatis.executor.keygen.SelectKeyGenerator;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;

public class SimpleStatementHandler extends BaseStatementHandler {

  //构造方法执行父类的构造方法
  public SimpleStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    super(executor, mappedStatement, parameter, rowBounds, resultHandler, boundSql);
  }

  //执行update操作
  public int update(Statement statement)
      throws SQLException {
    String sql = boundSql.getSql();
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    int rows;
    //最终都是执行statement的getUpdateCount方法
    if (keyGenerator instanceof Jdbc3KeyGenerator) {
      statement.execute(sql, Statement.RETURN_GENERATED_KEYS);
      rows = statement.getUpdateCount();
      keyGenerator.processAfter(executor, mappedStatement, statement, parameterObject);
    } else if (keyGenerator instanceof SelectKeyGenerator) {
      statement.execute(sql);
      rows = statement.getUpdateCount();
      keyGenerator.processAfter(executor, mappedStatement, statement, parameterObject);
    } else {
      statement.execute(sql);
      rows = statement.getUpdateCount();
    }
    return rows;
  }

  //给statement添加批量处理sql语句
  public void batch(Statement statement)
      throws SQLException {
    String sql = boundSql.getSql();
    statement.addBatch(sql);
  }

  //执行查询语句
  public <E> List<E> query(Statement statement, ResultHandler resultHandler)
      throws SQLException {
    String sql = boundSql.getSql();
    statement.execute(sql);//statement.execute方法执行sql语句
    return resultSetHandler.<E>handleResultSets(statement);
  }

  //构造Statement对象
  protected Statement instantiateStatement(Connection connection) throws SQLException {
     //通过Connection来create一个Statement对象
    if (mappedStatement.getResultSetType() != null) {
      return connection.createStatement(mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    } else {
      return connection.createStatement();
    }
  }

  //由于SimpleStatementHandler是处理没有参数的SQL，所以参数设置的方法无需任何处理
  public void parameterize(Statement statement) throws SQLException {
    // N/A
  }

}
```

可以看出主要是通过Connection创建一个Statement对象，然后通过调用Statement的execute方法执行sql语句

再看下PreparedStatementHandler，源码如下

```
package org.apache.ibatis.executor.statement;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;

import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.executor.keygen.Jdbc3KeyGenerator;
import org.apache.ibatis.executor.keygen.KeyGenerator;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;

public class PreparedStatementHandler extends BaseStatementHandler {

  //执行父类构造方法
  public PreparedStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    super(executor, mappedStatement, parameter, rowBounds, resultHandler, boundSql);
  }

  //执行update操作
  public int update(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    int rows = ps.getUpdateCount();
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
    return rows;
  }

  //给preparedStatement对象添加批量处理sql语句
  public void batch(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.addBatch();
  }

  //执行preparedStatement的查询语句
  public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    return resultSetHandler.<E> handleResultSets(ps);
  }

  //构建Statement的子类PreparedStatement对象
  protected Statement instantiateStatement(Connection connection) throws SQLException {
    String sql = boundSql.getSql();
    if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
      String[] keyColumnNames = mappedStatement.getKeyColumns();
      if (keyColumnNames == null) {
        return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
      } else {
        return connection.prepareStatement(sql, keyColumnNames);
      }
    } else if (mappedStatement.getResultSetType() != null) {
      return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    } else {
      return connection.prepareStatement(sql);
    }
  }

  //给Statement对象设置参数
  public void parameterize(Statement statement) throws SQLException {
    parameterHandler.setParameters((PreparedStatement) statement);
  }

}
```

再看下RoutingStatementHandler，这个相当于一个StatementHandler的路由器，本身不实现任何功能，只是根据传入的参数来选择调用哪个类型的StatementHandler来处理，代码如下：

```
public class RoutingStatementHandler implements StatementHandler {

private final StatementHandler delegate;

public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {

switch (ms.getStatementType()) {
case STATEMENT:
delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
break;
case PREPARED:
delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
break;
case CALLABLE:
delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
break;
default:
throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
}

}

@Override
public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
return delegate.prepare(connection, transactionTimeout);
}

@Override
public void parameterize(Statement statement) throws SQLException {
delegate.parameterize(statement);
}

@Override
public void batch(Statement statement) throws SQLException {
delegate.batch(statement);
}

@Override
public int update(Statement statement) throws SQLException {
return delegate.update(statement);
}

@Override
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
return delegate.<E>query(statement, resultHandler);
}

@Override
public <E> Cursor<E> queryCursor(Statement statement) throws SQLException {
return delegate.queryCursor(statement);
}

@Override
public BoundSql getBoundSql() {
return delegate.getBoundSql();
}

@Override
public ParameterHandler getParameterHandler() {
return delegate.getParameterHandler();
}
}
```

可见RoutingStatementHandler的作用就是在构造的时候根据Statement类型来创建不同的处理器，然后调用对应的处理器来进行操作。而在StatementHandler在初始化的时候，都是通过RoutingStatementHandler来进行路由的，Configuration中的创建StatementHandler代码如下：

```
 public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
return statementHandler;
}
```

再看看SimpleStatementHandler是如何执行sql语句的：

```
@Override
protected Statement instantiateStatement(Connection connection) throws SQLException {
if (mappedStatement.getResultSetType() != null) {
return connection.createStatement(mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
} else {
return connection.createStatement();
}
}
```

然后通过Statement对象执行sql语句，最后通过ResultSetHandler对象来处理执行后的结果。

### ResultSetHandler结果集处理器

说完了StatementHandler和ParameterHandler，接下来就需要对查询的结果进行处理了，而对于sql结果的处理是由ResultSetHandler处理的，ResultHandler位于mybatis包的

org.apache.ibatis.executor.resultset下，源码如下：

```
public interface ResultSetHandler {

  <E> List<E> handleResultSets(Statement stmt) throws SQLException;

  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;

  void handleOutputParameters(CallableStatement cs) throws SQLException;

}
```

前面两个方法是处理Statement执行后的结果集，而后面一个方法是处理存储过程执行后的输出参数。本文主要分析处理Statement执行结果的第一个方法，ResultSetHandler默认实现类是

DefaultResultSetHandler。和StatementHandler、ParameterHandler一样也是通过Configuration进行初始化的，代码如下：

```
public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }
```

DefaultResultSetHandler处理Statement的执行结果方法代码如下：

```
@Override
  public List<Object> handleResultSets(Statement stmt) throws SQLException {
    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

    final List<Object> multipleResults = new ArrayList<Object>();//定义返回结果的List

    int resultSetCount = 0;//定义结果长度
    ResultSetWrapper rsw = getFirstResultSet(stmt);//获取第一个结果集

    //获取结果集合
    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    //一般结果集合只有1个
    int resultMapCount = resultMaps.size();
    //
    validateResultMapsCount(rsw, resultMapCount);
    while (rsw != null && resultMapCount > resultSetCount) {
      //获取第一个结果集合
      ResultMap resultMap = resultMaps.get(resultSetCount);
      //处理结果映射，将数据存放到list中
      handleResultSet(rsw, resultMap, multipleResults, null);
      //取下一个结果集合重复操作
      rsw = getNextResultSet(stmt);
      cleanUpAfterHandlingResultSet();
      resultSetCount++;
    }

    String[] resultSets = mappedStatement.getResultSets();
    if (resultSets != null) {
      while (rsw != null && resultSetCount < resultSets.length) {
        ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
        if (parentMapping != null) {
          String nestedResultMapId = parentMapping.getNestedResultMapId();
          ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
          handleResultSet(rsw, resultMap, null, parentMapping);
        }
        rsw = getNextResultSet(stmt);
        cleanUpAfterHandlingResultSet();
        resultSetCount++;
      }
    }

    return collapseSingleResultList(multipleResults);
  }
```

### Executor执行器

从前面分析我们知道了sql的具体执行是通过调用SqlSession接口的对应的方法去执行的，而SqlSession最终都是通过调用了自己的Executor对象的query和update去执行的。本文就分析下sql的执行器-----Executor

Executor是mybatis的sql执行器，SqlSession是面向程序的，而Executor则就是面向数据库的，先看下Executor接口的方法有哪些，源码如下：

```
public interface Executor {

  ResultHandler NO_RESULT_HANDLER = null;

  int update(MappedStatement ms, Object parameter) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;

  <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException;

  List<BatchResult> flushStatements() throws SQLException;

  void commit(boolean required) throws SQLException;

  void rollback(boolean required) throws SQLException;

  CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql);

  boolean isCached(MappedStatement ms, CacheKey key);

  void clearLocalCache();

  void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType);

  Transaction getTransaction();

  void close(boolean forceRollback);

  boolean isClosed();

  void setExecutorWrapper(Executor executor);
```

和SqlSession一样定义了各种各样的sql执行的方法，有查询的query方法，有更新的update方法，以及和事务有关的commit方法和rollback方法等，接下来就以query方法为例，看下具体是如何执行的。

Executor接口共有两个实现类，分别是BaseExecutor和CachingExecutor，CachingExecutor是缓存执行器，后面会提到，现在先看下BaseExecutor

BaseExecutor的属性有：

```
protected Transaction transaction;//事务
  protected Executor wrapper;//执行器包装者

  protected ConcurrentLinkedQueue<DeferredLoad> deferredLoads;//线程安全队列
  protected PerpetualCache localCache;//本地缓存
  protected PerpetualCache localOutputParameterCache;
  protected Configuration configuration;

  protected int queryStack = 0;//查询次数栈
  private boolean closed;//是否已关闭(回滚的时候会被关闭)
```

再看下BaseExecutor执行query方法的源码：

```
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
      ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
      if (closed) {//判断执行器是否已关闭
        throw new ExecutorException("Executor was closed.");
      }
      //如果查询次数栈为0并且MappedStatement可以清除缓存,则清除本地缓存
      if (queryStack == 0 && ms.isFlushCacheRequired()) {
        clearLocalCache();
      }
      List<E> list;
      try {
        queryStack++;//查询次数+1
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;//从缓存中根据缓存key查询是否有缓存
        if (list != null) {
          //如果缓存中有数据,则处理缓存
          handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
          //如果缓存中没有数据,则从数据库查询数据
          list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
      } finally {
          //查询次数-1
        queryStack--;
      }
      if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
          deferredLoad.load();
        }
        // issue #601
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
          // issue #482
          clearLocalCache();
        }
      }
      return list;
    }
```

可以看出BaseExecutor会优先从缓存中查询数据，如果缓存不为空再从数据库数据。在这里有一个queryStack会进行自增自减，它的作用是什么呢？

先看下如果没有缓存的话，BaseExecutor是怎么从数据库查询数据的：

```
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);//执行doQuery方法
    } finally {
      localCache.removeObject(key);
    }
    localCache.putObject(key, list);//将查询结果放入缓存
    if (ms.getStatementType() == StatementType.CALLABLE) {//如果callable类型查询
      localOutputParameterCache.putObject(key, parameter);//将参数放入缓存中
    }
    return list;
  }
```

可见该方法是调用了doQuery方法从数据库查询了数据，然后将查询的结果及查询用的参数放入了缓存中，而doQuery方法是BaseExecutor中的抽象方法，具体的实现是由BaseExecutor的子类进行实现

```
protected abstract int doUpdate(MappedStatement ms, Object parameter)
      throws SQLException;

  protected abstract List<BatchResult> doFlushStatements(boolean isRollback)
      throws SQLException;

  protected abstract <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)
      throws SQLException;

  protected abstract <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql)
      throws SQLException;

  protected void closeStatement(Statement statement) {
    if (statement != null) {
      try {
        statement.close();
      } catch (SQLException e) {
        // ignore
      }
    }
  }
```

BaseExecutor共有SimpleExecutor、ReuseExecutor、BatchExecutor以及ClosedExecutor四个子类，这个后面再分析，现在我们以及知道了SqlSession是调用了Executor的方法来执行sql，而Executor的默认实现类的BaseExecutor，而BaseExecutor又是调用了其子类的方法。而BaseExecutor则对查询的结果进行了缓存处理以及查询的时候会从缓存中进行查询。

从上述BaseExecutor的定义可以看出：

1. 执行器在特定的事务上下文下执行；
2. 具有本地缓存和本地出参缓存（任何时候，只要事务提交或者回滚或者执行update或者查询时设定了刷新缓存，都会清空本地缓存和本地出参缓存）；
3. 具有延迟加载任务；

　　BaseExecutor实现了大部分通用功能本地缓存管理、事务提交、回滚、超时设置、延迟加载等，但是将下列4个方法留给了具体的子类实现：

```
 protected abstract int doUpdate(MappedStatement ms, Object parameter)
      throws SQLException;

  protected abstract List<BatchResult> doFlushStatements(boolean isRollback)
      throws SQLException;

  protected abstract <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)
      throws SQLException;

  protected abstract <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql)
      throws SQLException;
```

从功能上来说，这三种执行器的差别在于：

- ExecutorType.SIMPLE：这个执行器类型不做特殊的事情。它为每个语句的每次执行创建一个新的预处理语句。
- ExecutorType.REUSE：这个执行器类型会复用预处理语句。
- ExecutorType.BATCH：这个执行器会批量执行所有更新语句，也就是jdbc addBatch API的facade模式。
  　　所以这三种类型的执行器可以说时应用于不同的负载场景下，除了SIMPLE类型外，另外两种要求对系统有较好的架构设计，当然也提供了更多的回报。

#### SIMPLE执行器

```
public class SimpleExecutor extends BaseExecutor {

  public SimpleExecutor(Configuration configuration, Transaction transaction) {
    super(configuration, transaction);
  }

  @Override
  public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.update(stmt);
    } finally {
      closeStatement(stmt);
    }
  }

  @Override
  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }

  @Override
  protected <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql) throws SQLException {
    Configuration configuration = ms.getConfiguration();
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, null, boundSql);
    Statement stmt = prepareStatement(handler, ms.getStatementLog());
    return handler.<E>queryCursor(stmt);
  }

  @Override
  public List<BatchResult> doFlushStatements(boolean isRollback) throws SQLException {
    return Collections.emptyList();
  }

  private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout());
    handler.parameterize(stmt);
    return stmt;
  }

}
```

#### REUSE执行器

```
public class ReuseExecutor extends BaseExecutor {

  private final Map<String, Statement> statementMap = new HashMap<String, Statement>();

  public ReuseExecutor(Configuration configuration, Transaction transaction) {
    super(configuration, transaction);
  }

  @Override
  public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Configuration configuration = ms.getConfiguration();
    StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
    Statement stmt = prepareStatement(handler, ms.getStatementLog());
    return handler.update(stmt);
  }

  @Override
  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Configuration configuration = ms.getConfiguration();
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    Statement stmt = prepareStatement(handler, ms.getStatementLog());
    return handler.<E>query(stmt, resultHandler);
  }

  @Override
  protected <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql) throws SQLException {
    Configuration configuration = ms.getConfiguration();
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, null, boundSql);
    Statement stmt = prepareStatement(handler, ms.getStatementLog());
    return handler.<E>queryCursor(stmt);
  }

  @Override
  public List<BatchResult> doFlushStatements(boolean isRollback) throws SQLException {
    for (Statement stmt : statementMap.values()) {
      closeStatement(stmt);
    }
    statementMap.clear();
    return Collections.emptyList();
  }

  private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    BoundSql boundSql = handler.getBoundSql();
    String sql = boundSql.getSql();
    if (hasStatementFor(sql)) {
      stmt = getStatement(sql);
      applyTransactionTimeout(stmt);
    } else {
      Connection connection = getConnection(statementLog);
      stmt = handler.prepare(connection, transaction.getTimeout());
      putStatement(sql, stmt);
    }
    handler.parameterize(stmt);
    return stmt;
  }

  private boolean hasStatementFor(String sql) {
    try {
      return statementMap.keySet().contains(sql) && !statementMap.get(sql).getConnection().isClosed();
    } catch (SQLException e) {
      return false;
    }
  }

  private Statement getStatement(String s) {
    return statementMap.get(s);
  }

  private void putStatement(String sql, Statement stmt) {
    statementMap.put(sql, stmt);
  }

}
```

从实现可以看出，REUSE和SIMPLE在doUpdate/doQuery上有个差别，不再是每执行一个语句就close掉了，而是尽可能的根据SQL文本进行缓存并重用，但是由于数据库服务器端通常对每个连接以及全局的语句(oracle称为游标)handler的数量有限制，oracle中是open_cursors参数控制，mysql中是mysql_stmt_close参数控制，这就会导致如果sql都是靠if各种拼接出来，日积月累可能会导致数据库资源耗尽。其是否有足够价值，视创建Statement语句消耗的资源占整体资源的比例、以及一共有多少完全不同的Statement数量而定，一般来说，纯粹的OLTP且非自动生成的sqlmap，它会比SIMPLE执行器更好。

#### BATCH执行器

BATCH执行器的实现代码如下：

```
public class BatchExecutor extends BaseExecutor {

  public static final int BATCH_UPDATE_RETURN_VALUE = Integer.MIN_VALUE + 1002;
  // 存储在一个事务中的批量DML的语句列表
  private final List<Statement> statementList = new ArrayList<Statement>();
  // 存放DML语句对应的参数对象,包括自动/手工生成的key
  private final List<BatchResult> batchResultList = new ArrayList<BatchResult>();

  // 最新提交执行的SQL语句
  private String currentSql;

  // 最新提交执行的语句
  private MappedStatement currentStatement;

  public BatchExecutor(Configuration configuration, Transaction transaction) {
    super(configuration, transaction);
  }

  @Override
  public int doUpdate(MappedStatement ms, Object parameterObject) throws SQLException {
    final Configuration configuration = ms.getConfiguration();
    final StatementHandler handler = configuration.newStatementHandler(this, ms, parameterObject, RowBounds.DEFAULT, null, null);
    final BoundSql boundSql = handler.getBoundSql();
    final String sql = boundSql.getSql();
    final Statement stmt;
    // 如果最新执行的一条语句和前面一条语句相同,就不创建新的语句了，直接用缓存的语句，只是把参数对象添加到该语句对应的BatchResult中
    // 否则的话，无论是否在未提交之前，还有pending的语句，都新插入一条语句到list中
    if (sql.equals(currentSql) && ms.equals(currentStatement)) {
      int last = statementList.size() - 1;
      stmt = statementList.get(last);
      applyTransactionTimeout(stmt);
     handler.parameterize(stmt);//fix Issues 322
      BatchResult batchResult = batchResultList.get(last);
      batchResult.addParameterObject(parameterObject);
    } else {
      Connection connection = getConnection(ms.getStatementLog());
      stmt = handler.prepare(connection, transaction.getTimeout());
      handler.parameterize(stmt);    //fix Issues 322
      currentSql = sql;
      currentStatement = ms;
      statementList.add(stmt);
      batchResultList.add(new BatchResult(ms, sql, parameterObject));
    }
  // handler.parameterize(stmt);
    // 调用jdbc的addBatch方法
    handler.batch(stmt);
    return BATCH_UPDATE_RETURN_VALUE;
  }

  @Override
  public <E> List<E> doQuery(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)
      throws SQLException {
    Statement stmt = null;
    try {
      flushStatements();
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameterObject, rowBounds, resultHandler, boundSql);
      Connection connection = getConnection(ms.getStatementLog());
      stmt = handler.prepare(connection, transaction.getTimeout());
      handler.parameterize(stmt);
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }

  @Override
  protected <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql) throws SQLException {
    flushStatements();
    Configuration configuration = ms.getConfiguration();
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, null, boundSql);
    Connection connection = getConnection(ms.getStatementLog());
    Statement stmt = handler.prepare(connection, transaction.getTimeout());
    handler.parameterize(stmt);
    return handler.<E>queryCursor(stmt);
  }

  @Override
  public List<BatchResult> doFlushStatements(boolean isRollback) throws SQLException {
    try {
      List<BatchResult> results = new ArrayList<BatchResult>();
      if (isRollback) {
        return Collections.emptyList();
      }
      for (int i = 0, n = statementList.size(); i < n; i++) {
        Statement stmt = statementList.get(i);
        applyTransactionTimeout(stmt);
        BatchResult batchResult = batchResultList.get(i);
        try {
          batchResult.setUpdateCounts(stmt.executeBatch());
          MappedStatement ms = batchResult.getMappedStatement();
          List<Object> parameterObjects = batchResult.getParameterObjects();
          KeyGenerator keyGenerator = ms.getKeyGenerator();
          if (Jdbc3KeyGenerator.class.equals(keyGenerator.getClass())) {
            Jdbc3KeyGenerator jdbc3KeyGenerator = (Jdbc3KeyGenerator) keyGenerator;
            jdbc3KeyGenerator.processBatch(ms, stmt, parameterObjects);
          } else if (!NoKeyGenerator.class.equals(keyGenerator.getClass())) { //issue #141
            for (Object parameter : parameterObjects) {
              keyGenerator.processAfter(this, ms, stmt, parameter);
            }
          }
          // Close statement to close cursor #1109
          closeStatement(stmt);
        } catch (BatchUpdateException e) {
          StringBuilder message = new StringBuilder();
          message.append(batchResult.getMappedStatement().getId())
              .append(" (batch index #")
              .append(i + 1)
              .append(")")
              .append(" failed.");
          if (i > 0) {
            message.append(" ")
                .append(i)
                .append(" prior sub executor(s) completed successfully, but will be rolled back.");
          }
          throw new BatchExecutorException(message.toString(), e, results, batchResult);
        }
        results.add(batchResult);
      }
      return results;
    } finally {
      for (Statement stmt : statementList) {
        closeStatement(stmt);
      }
      currentSql = null;
      statementList.clear();
      batchResultList.clear();
    }
  }

}
```

批量执行器是JDBC Statement.addBatch的实现,对于批量insert而言比如导入大量数据的ETL,驱动器如果支持的话，能够大幅度的提高DML语句的性能

## 事务实现

mybatis的事务管理模式分为两种，自动提交和手工提交，DefaultSqlSessionFactory的openSession中重载中，提供了一个参数用于控制是否自动提交事务，该参数最终被传递给 java.sql.Connection.setAutoCommit()方法用于控制是否自动提交事务(默认情况下，连接是自动提交的)，如下所示：

```
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      final Executor executor = configuration.newExecutor(tx, execType);
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

如上所示，返回的事务传递给了执行器，因为执行器是在事务上下文中执行，所以对于自动提交模式，实际上mybatis不需要去关心。只有非自动管理模式，mybatis才需要关心事务。对于非自动提交模式，通过sqlSession.commit()或sqlSession.rollback()发起，在进行提交或者回滚的时候会调用isCommitOrRollbackRequired判断是否应该提交或者回滚事务，如下所示：

```
 private boolean isCommitOrRollbackRequired(boolean force) {
    return (!autoCommit && dirty) || force;
  }
```

只有非自动提交模式且执行过DML操作或者设置强制提交才会认为应该进行事务提交或者回滚操作。
　　对于不同的执行器，在提交和回滚执行的逻辑不一样，因为每个执行器在一级、二级、语句缓存上的差异：

- 对于简单执行器，除了清空一级缓存外，什么都不做；
- 对于REUSE执行器，关闭每个缓存的Statement以释放服务器端语句处理器，然后清空缓存的语句；
- 对于批量处理器，则执行每个批处理语句的executeBatch()方法以便真正执行语句，然后关闭Statement；

　　上述逻辑执行完成后，会执行提交/回滚操作。对于缓存执行器，在提交/回滚完成之后，会将TransactionCache中的entriesMissedInCache和entriesToAddOnCommit列表分别移动到语句对应的二级缓存中或清空掉。

## 缓存

只要实现org.apache.ibatis.cache.Cache接口的任何类都可以当做缓存，Cache接口很简单：

```
public interface Cache {

  /**
   * @return The identifier of this cache
   */
  String getId();

  /**
   * @param key Can be any object but usually it is a {@link CacheKey}
   * @param value The result of a select.
   */
  void putObject(Object key, Object value);

  /**
   * @param key The key
   * @return The object stored in the cache.
   */
  Object getObject(Object key);

  /**
   * As of 3.3.0 this method is only called during a rollback 
   * for any previous value that was missing in the cache.
   * This lets any blocking cache to release the lock that 
   * may have previously put on the key.
   * A blocking cache puts a lock when a value is null 
   * and releases it when the value is back again.
   * This way other threads will wait for the value to be 
   * available instead of hitting the database.
   *
   * 
   * @param key The key
   * @return Not used
   */
  Object removeObject(Object key);

  /**
   * Clears this cache instance
   */  
  void clear();

  /**
   * Optional. This method is not called by the core.
   * 
   * @return The number of elements stored in the cache (not its capacity).
   */
  int getSize();

  /** 
   * Optional. As of 3.2.6 this method is no longer called by the core.
   *  
   * Any locking needed by the cache must be provided internally by the cache provider.
   * 
   * @return A ReadWriteLock 
   */
  ReadWriteLock getReadWriteLock();

}
```

mybatis提供了基本实现org.apache.ibatis.cache.impl.PerpetualCache，内部采用原始HashMap实现。第二个需要知道的方面是mybatis有一级缓存和二级缓存。

一级缓存是SqlSession级别的缓存，不同SqlSession之间的缓存数据区域（HashMap）是互相不影响，MyBatis默认支持一级缓存，不需要任何的配置，默认情况下(一级缓存的有效范围可通过参数localCacheScope参数修改，取值为SESSION或者STATEMENT)，在一个SqlSession的查询期间，只要没有发生commit/rollback或者调用close()方法，那么mybatis就会先根据当前执行语句的CacheKey到一级缓存中查找，如果找到了就直接返回，不到数据库中执行。其实现在代码BaseExecutor.query()中，如下所示：

```
@Override
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
      clearLocalCache();
    }
    List<E> list;
    try {
      queryStack++;
      // 如果在一级缓存中就直接获取
      ==list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }==
    } finally {
      queryStack--;
    }
    if (queryStack == 0) {
      for (DeferredLoad deferredLoad : deferredLoads) {
        deferredLoad.load();
      }
      // issue #601
      deferredLoads.clear();
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        // issue #482
        // 如果设置了一级缓存是STATEMENT级别而非默认的SESSION级别，一级缓存就去掉了
        clearLocalCache();
      }
    }
    return list;
  }
```

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession。二级缓存默认不启用，需要通过在Mapper中明确设置cache，它的实现在CachingExecutor的query()方法中，如下所示：

```
  @Override
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    Cache cache = ms.getCache();
    if (cache != null) {
      flushCacheIfRequired(ms);
      if (ms.isUseCache() && resultHandler == null) {
        ensureNoOutParams(ms, parameterObject, boundSql);
        @SuppressWarnings("unchecked")
        // 如果二级缓存中找到了记录就直接返回,否则到DB查询后进行缓存
        List<E> list = (List<E>) tcm.getObject(cache, key);
        if (list == null) {
          list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          tcm.putObject(cache, key, list); // issue #578 and #116
        }
        return list;
      }
    }
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }
```

　　在mybatis的缓存实现中，缓存键CacheKey的格式为：cacheKey=ID + offset + limit + sql + parameterValues + environmentId。对于本书例子中的语句，其CacheKey为：

> -1445574094:212285810:org.mybatis.internal.example.mapper.UserMapper.getUser:0:2147483647:select lfPartyId,partyName from LfParty where partyName = ?　AND partyName like ? and lfPartyId in ( ?, ?):p2:p2:1:2:development

- 对于一级缓存，commit/rollback都会清空一级缓存。
- 对于二级缓存，DML操作或者显示设置语句层面的flushCache属性都会使得二级缓存失效。

　　在二级缓存容器的具体回收策略实现上，有下列几种：

- LRU – 最近最少使用的：移除最长时间不被使用的对象，也是默认的选项，其实现类是org.apache.ibatis.cache.decorators.LruCache。
- FIFO – 先进先出：按对象进入缓存的顺序来移除它们，其实现类是org.apache.ibatis.cache.decorators.FifoCache。
- SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象，其实现类是org.apache.ibatis.cache.decorators.SoftCache。
- WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象，其实现类是org.apache.ibatis.cache.decorators.WeakCache。

　　在缓存的设计上，Mybatis的所有Cache算法都是基于装饰器/Composite模式对PerpetualCache扩展增加功能。

　　对于模块化微服务系统来说，应该来说mybatis的一二级缓存对业务数据都不适合，尤其是对于OLTP系统来说，CRM/BI这些不算，如果要求数据非常精确的话，也不是特别合适。对这些要求数据准确的系统来说，尽可能只使用mybatis的ORM特性比较靠谱。但是有一部分数据如果前期没有很少的设计缓存的话，是很有价值的，比如说对于一些配置类数据比如数据字典、系统参数、业务配置项等很少变化的数据。