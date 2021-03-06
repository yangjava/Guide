# Mybatis

**MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings. MyBatis eliminates almost all of the JDBC code and manual setting of parameters and retrieval of results. MyBatis can use simple XML or Annotations for configuration and map primitives, Map interfaces and Java POJOs (Plain Old Java Objects) to database records.**

官网：MyBatis是一个支持普通SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis消除了几乎所有的JDBC代码和参数的手工设置以及对结果集的检索封装。MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录。

## 简介

MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。

github地址：https://github.com/mybatis/mybatis-3/releases

 1）MyBATIS 目前提供了三种语言实现的版本，包括：Java、.NET以及Ruby。（我主要学习java，就讲java的使用）
 2）它提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。
 3）mybatis与hibernate的对比？

  mybatis提供一种“半自动化”的ORM实现。
  这里的“半自动化”，是相对Hibernate等提供了全面的数据库封装机制的“全自动化”ORM实现而言，“全自动”ORM实现了POJO和数据库表之间的映射，以及 SQL 的自动生成和执行。

  而mybatis的着力点，则在于POJO与SQL之间的映射关系。

# 简单例子

# 1. jdbc编程步骤

1. 加载数据库驱动
2. 创建并获取数据库链接
3. 创建jdbc statement对象
4. 设置sql语句
5. 设置sql语句中的参数(使用preparedStatement)
6. 通过statement执行sql并获取结果
7. 对sql执行结果进行解析处理
8. 释放资源(resultSet、preparedstatement、connection)

# 2. 问题总结

1.数据库连接，使用时就创建，不使用立即释放，对数据库进行频繁连接开启和关闭，造成数据库资源浪费，影响数据库性能。

设想：使用数据库连接池管理数据库连接。

2.将sql语句硬编码到java代码中，如果sql语句修改，需要重新编译java代码，不利于系统维护。

设想：将sql语句配置在xml配置文件中，即使sql变化，不需要对java代码进行重新编译。


3.向preparedStatement中设置参数，对占位符号位置和设置参数值，硬编码在java代码中，不利于系统维护。

设想：将sql语句及占位符号和参数全部配置在xml中。

4.从resultSet中遍历结果集数据时，存在硬编码，将获取表的字段进行硬编码，不利于系统维护。

设想：将查询的结果集，自动映射成java对象。

# 3. 参考代码

```java
package com.iot.mybatis.jdbc;

//import java.sql.*;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Created by Administrator on 2016/2/21.
 */
public class JdbcTest {
    public static void main(String[] args) {
        //数据库连接
        Connection connection = null;
        //预编译的Statement，使用预编译的Statement提高数据库性能
        PreparedStatement preparedStatement = null;
        //结果集
        ResultSet resultSet = null;

        try {
            //加载数据库驱动
            Class.forName("com.mysql.jdbc.Driver");

            //通过驱动管理类获取数据库链接
            connection =  DriverManager.getConnection("jdbc:mysql://120.25.162.238:3306/mybatis001?characterEncoding=utf-8", "root", "123");
            //定义sql语句 ?表示占位符
            String sql = "select * from user where username = ?";
            //获取预处理statement
            preparedStatement = connection.prepareStatement(sql);
            //设置参数，第一个参数为sql语句中参数的序号（从1开始），第二个参数为设置的参数值
            preparedStatement.setString(1, "王五");
            //向数据库发出sql执行查询，查询出结果集
            resultSet =  preparedStatement.executeQuery();
            //遍历查询结果集
            while(resultSet.next()){
                System.out.println(resultSet.getString("id")+"  "+resultSet.getString("username"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            //释放资源
            if(resultSet!=null){
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
            if(preparedStatement!=null){
                try {
                    preparedStatement.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
            if(connection!=null){
                try {
                    connection.close();
                } catch (SQLException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }

        }

    }

}

```



# 2. 框架原理

mybatis框架

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601112152.png)

# 3. mybatis框架执行过程

1、配置mybatis的配置文件，SqlMapConfig.xml（名称不固定）

2、通过配置文件，加载mybatis运行环境，创建SqlSessionFactory会话工厂(SqlSessionFactory在实际使用时按单例方式)

3、通过SqlSessionFactory创建SqlSession。SqlSession是一个面向用户接口（提供操作数据库方法），实现对象是线程不安全的，建议sqlSession应用场合在方法体内。

4、调用sqlSession的方法去操作数据。如果需要提交事务，需要执行SqlSession的commit()方法。

5、释放资源，关闭SqlSession

# 4. mybatis开发dao的方法

1.原始dao 的方法

- 需要程序员编写dao接口和实现类
- 需要在dao实现类中注入一个SqlSessionFactory工厂

2.mapper代理开发方法（建议使用）

只需要程序员编写mapper接口（就是dao接口）。
程序员在编写mapper.xml(映射文件)和mapper.java需要遵循一个开发规范：

- mapper.xml中namespace就是mapper.java的类全路径。
- mapper.xml中statement的id和mapper.java中方法名一致。
- mapper.xml中statement的parameterType指定输入参数的类型和mapper.java的方法输入参数类型一致
- mapper.xml中statement的resultType指定输出结果的类型和mapper.java的方法返回值类型一致。


SqlMapConfig.xml配置文件：可以配置properties属性、别名、mapper加载。

# 5. 输入映射和输出映射

- 输入映射：
  - parameterType：指定输入参数类型可以简单类型、pojo、hashmap。
  - 对于综合查询，建议parameterType使用包装的pojo，有利于系统扩展。

- 输出映射：
  - resultType：查询到的列名和resultType指定的pojo的属性名一致，才能映射成功。
  - reusltMap：可以通过resultMap 完成一些高级映射。如果查询到的列名和映射的pojo的属性名不一致时，通过resultMap设置列名和属性名之间的对应关系（映射关系）。可以完成映射。
    - 高级映射：
      将关联查询的列映射到一个pojo属性中。（一对一）
      将关联查询的列映射到一个List<pojo>中。（一对多）

# 6. 动态sql

- 动态sql：（重点）
  - if判断（掌握）
  - where
  - foreach
  - sql片段（掌握）

# 1. 目录结构

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601112421.jpg)

# 2. 数据库表的设计

~~~sql
/*
 Navicat Premium Data Transfer

 Source Server         : Mysql
 Source Server Type    : MySQL
 Source Server Version : 50726
 Source Host           : localhost:3306
 Source Schema         : mybatis

 Target Server Type    : MySQL
 Target Server Version : 50726
 File Encoding         : 65001

 Date: 03/11/2019 12:01:59
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `birthday` date NULL DEFAULT NULL,
  `sex` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `address` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 36 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;

~~~

# 3. 配置文件

## 3.1 pom.xml

~~~xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
  </dependencies>
~~~

## 3.2 log4j.properties

```properties
# Global logging configuration
# 开发环境下，日志级别要设置成DEBUG或者ERROR
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

## 3.3 SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理-->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="" />
            </dataSource>
        </environment>
    </environments>
    <!--加载映射文件-->
    <mappers>
        <mapper resource="sqlmap/User.xml"/>
    </mappers>
</configuration>
```

# 4. User.java

~~~java
import java.util.Date;

public class User {
    //用户po
    //属性名和数据库字段名对应
    private int id;
    private String username;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
~~~


# 5. User.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    命名空间，作用为，对sql进行分类化管理，理解sql隔离，
    注意：使用mapper代理方法开发，namespace有特殊作用
-->
<mapper namespace="test">
    <!--在映射文件中配置sql-->
    <!--
        findUserById
        通过select执行数据库查询
        id：标识映射文件的sql
        将sql语句封装到mappedStatement对象中，所以将id称为Statement的id
        #{}:表示一个占位符
        parameterType:指定输入参数的类型
        #{id}：其中的id表示接收输入的参数，参数的名称就是id，如果输入参数类型为简单类型，那么#{}中的参数可以任意，可以是value或其他
        resultType：指定sql输出结果的所映射的Java对象类型，select指定的resultType表示将单条记录映射成的Java对象
    -->
    <select id="findUserById" parameterType="int" resultType="cn.edu.wtu.po.User">
        select * from user where id=#{VALUE }
    </select>




    <!--
        findUserByName
        ${}:表示拼接字符串，将接收到的sql不加任何修饰拼接在sql语句里
        使用${}拼接sql，可能会引起sql注入,一般不建议使用
        ${value}：接收参数的内容，如果传入的的是简单类型，${}中只能使用value
    -->
    <select id="findUserByName" parameterType="java.lang.String" resultType="cn.edu.wtu.po.User">
        select * from user WHERE username LIKE '%${value}%'
    </select>



    <!--
        添加用户
        parameterType:指定参数类型为pojo类型
        #{}中指定pojo的属性名，接收到的pojo对象的属性值，mybatis通过OGNL获取对象的值
        SELECT LAST_INSERT_ID():得到刚刚insert进去的记录的主键值，只适用于主键自增
        非主键自的则需要使用uuid()来实现,表的id类型也得设置为tring(详见下面的注释)
        keyProperty：将查询到的主键值设置到SparameterType指定的对象的哪个属性
        order:SELECT LAST_INSERT_ID()执行顺序，相当于insert语句来说它的实现顺序

    -->
    <insert id="insertUser" parameterType="cn.edu.wtu.po.User">
        <!--uuid()-->
        <!--
            <selectKey keyProperty="id" order="AFTER" resultType="java.lang.String">
              SELECT uuid()
            </selectKey>
            insert into user (id,username,birthday,sex,address) value(#{id},#{username},#{birthday},#{sex},#{address})
        -->
        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT LAST_INSERT_ID()
        </selectKey>
        insert into user (username,birthday,sex,address) value(#{username},#{birthday},#{sex},#{address})
    </insert>


    <delete id="deleteUser" parameterType="java.lang.Integer">
        delete from user where id=#{id}
    </delete>

    <update id="updateUser" parameterType="cn.edu.wtu.po.User">
        UPDATE user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
    </update>
</mapper>
```


# 6. MybatisFirst.java

~~~java
public class MybatisFirst {
    /**
     * 根据id查询用户信息，得到一条记录
     * @throws IOException
     */
    @Test
    public void findUserByIdTest() throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream is = Resources.getResourceAsStream(resource);
        // 创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //通过SqlSession操作数据库
        //第一个参数：映射文件中的statement的id，等于namespace+"."+statement的id
        //第二个参数：指定和映射文件中所匹配的所有parameterType的类型
        //sqlSession.selectOne()的结果是映射文件中所匹配的resultType类型的对象
        User user = sqlSession.selectOne("test.findUserById",1);
        System.out.println(user);
        // 释放资源
        sqlSession.close();
    }

    @Test
    public void findUserByName() throws IOException {
        // mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream is = Resources.getResourceAsStream(resource);
        // 创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        // 通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<User> list= sqlSession.selectList("test.findUserByName","小明");
        System.out.println(list);
        sqlSession.close();
    }

    /*
        小结：
        selectOne和selectList：

        selectOne表示查询出一条记录进行映射。如果使用selectOne可以实现使用selectList也可以实现（list中只有一个对象）。
        selectList表示查询出一个列表（多条记录）进行映射。如果使用selectList查询多条记录，不能使用selectOne。
        如果使用selectOne报错：
        org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 4

     */

    @Test
    public void insertUserTest() throws IOException {
        //mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        User user = new User();
        user.setUsername("宋江涛");
        user.setBirthday(new Date());
        user.setSex("男");
        user.setAddress("山西");
        //list中的user和映射文件User.xml中的resultType的类型一直
        sqlSession.insert("test.insertUser",user);
        //提交事务
        sqlSession.commit();
        //获取主键
        System.out.println(user.getId());
        sqlSession.close();
    }

    /**
     * 删除用户
     * @throws IOException
     */
    @Test
    public void deleteUserTest() throws IOException {
        //mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //list中的user和映射文件User.xml中的resultType的类型一直
        sqlSession.delete("test.deleteUser",30);
        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }

    /**
     * 更新用户
     * @throws IOException
     */
    @Test
    public void updateUserTest() throws IOException {
        //mybatis配置文件
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //创建会话工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //通过工厂得到SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        User user = new User();
        user.setId(27);
        user.setUsername("宋江涛new2");
        user.setBirthday(new Date());
        user.setSex("男");
        user.setAddress("山西太原new");
        //list中的user和映射文件User.xml中的resultType的类型一直
        sqlSession.update("test.updateUser",user);
        //提交事务
        sqlSession.commit();
        sqlSession.close();
    }
}
~~~

# 7. 总结

- `parameterType`

在映射文件中通过parameterType指定输入参数的类型


- `resultType`

在映射文件中通过resultType指定输出结果的类型


- `#{}`和`${}`

`#{}`表示一个占位符号;

`${}`表示一个拼接符号，会引起sql注入，所以不建议使用


- `selectOne`和`selectList`

`selectOne`表示查询一条记录进行映射，使用`selectList`也可以使用，只不过只有一个对象

`selectList`表示查询出一个列表(参数记录)进行映射，不能够使用`selectOne`查，不然会报下面的错:

```
org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 3
```

# 1. SqlSession使用范围

- SqlSessionFactoryBuilder

通过`SqlSessionFactoryBuilder`创建会话工厂`SqlSessionFactory`将`SqlSessionFactoryBuilder`当成一个工具类使用即可，不需要使用单例管理`SqlSessionFactoryBuilder`。在需要创建`SqlSessionFactory`时候，只需要new一次`SqlSessionFactoryBuilder`即可。


- `SqlSessionFactory`

通过`SqlSessionFactory`创建`SqlSession`，使用单例模式管理`sqlSessionFactory`（工厂一旦创建，使用一个实例）。将来mybatis和spring整合后，使用单例模式管理`sqlSessionFactory`。


- `SqlSession`

`SqlSession`是一个面向用户（程序员）的接口。SqlSession中提供了很多操作数据库的方法：如：`selectOne`(返回单个对象)、`selectList`（返回单个或多个对象）。

`SqlSession`是线程不安全的，在`SqlSesion`实现类中除了有接口中的方法（操作数据库的方法）还有数据域属性。

`SqlSession`最佳应用场合在方法体内，定义成局部变量使用。


# 2. 项目结构

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601112709.jpg)

# 3. 原始dao方法

## 3.1 UserDao.java

~~~java
package cn.edu.wtu.dao;

import cn.edu.wtu.po.User;

/**
 * @Package cn.edu.wtu.dao
 * @InterfaceName UserDao
 * @Description TODO
 * @Date 19/11/3 13:48
 * @Author LIM
 * @Version V1.0
 */
public interface UserDAO {
    /**
     * dao原始开发
     * 根据id查询用户信息
     * @param id
     * @return
     * @throws Exception
     */
    public User findUserById(int id) throws Exception;

    /**
     * 添加用户
     * @param user
     * @throws Exception
     */
    public void insertUser(User user) throws Exception;

    /**
     * 删除用户
     * @param id
     * @throws Exception
     */
    public void deleteUser(int id) throws Exception;
}
~~~

## 3.2 UserDaoImpl.java

~~~java
package cn.edu.wtu.dao;

import cn.edu.wtu.po.User;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

/**
 * @Package cn.edu.wtu.dao
 * @ClassName UserDAOImpl
 * @Description TODO
 * @Date 19/11/3 13:48
 * @Author LIM
 * @Version V1.0
 */
public class UserDAOImpl<id> implements UserDAO {
  /** 原生态的dao
   * 需要向dao实现类里注入SqlSessionFactory
   * 通过构造方法
   */
  private SqlSessionFactory sqlSessionFactory;

  public UserDAOImpl(SqlSessionFactory sqlSessionFactory) {
    this.sqlSessionFactory = sqlSessionFactory;
  }

  @Override
  public User findUserById(int id) throws Exception {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    User user = sqlSession.selectOne("test.findUserById", id);
    // 关闭
    sqlSession.close();
    return user;
  }

  @Override
  public void insertUser(User user) throws Exception {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    sqlSession.insert("test.insertUser", user);
    // 提交
    sqlSession.commit();
    // 关闭
    sqlSession.close();
  }

  @Override
  public void deleteUser(int id) throws Exception {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    sqlSession.insert("test.deleteUser", id);
    sqlSession.commit();
    sqlSession.close();
  }
}
~~~

## 3.3 UserDaoImplTest.java

~~~java
package cn.edu.wtu.dao;

import cn.edu.wtu.po.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;

public class UserDAOImplTest {
  private SqlSessionFactory sqlSessionFactory;

  @Before
  public void setUp() throws IOException {
    InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
    // 创建会话工厂
    sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
  }

  @Test
  public void testFindUserById() throws Exception {
    User user = new UserDAOImpl<>(sqlSessionFactory).findUserById(1);
    System.out.println(user.toString());
  }

  @Test
  public void testInsertUser() throws Exception {
    User user = new User();
    user.setId(100);
    user.setUsername("张宇");
    user.setSex("2");
    user.setBirthday(new Date());
    user.setAddress("襄阳");
    new UserDAOImpl<>(sqlSessionFactory).insertUser(user);
  }

  @Test
  public void testDeleteUser() throws Exception {
    new UserDAOImpl<>(sqlSessionFactory).deleteUser(25);
  }
}
~~~

# 4. Mybatis的mapper接口（相当于dao接口）代理开发方法

## 4.1 UserMapper.java

~~~java
package cn.edu.wtu.mapper;

import cn.edu.wtu.po.User;

import java.util.List;

/**
 * @Package cn.edu.wtu.mapper
 * @InterfaceName UserMapper
 * @Description TODO
 * @Date 19/11/3 13:51
 * @Author LIM
 * @Version V1.0
 */
public interface UserMapper {
  /** mapper代理开发和dao开发对比 mapper接口,相当于dao接口，
   * mybatis可以自动生成mapper接口实现类的代理对象
   */

    /**
     * 根据id查询用户信息
     * @param id
     * @return
     * @throws Exception
     */
  public User findUserById(int id) throws Exception;

    /**
     * 根据用户名查询用户列表
     * @param name
     * @return
     * @throws Exception
     */
  public List<User> findUserByName(String name) throws Exception;

    /**
     * 添加用户
     * @param user
     * @throws Exception
     */
    public void insertUser(User user) throws Exception;

    /**
     * 删除用户
     * @param id
     * @throws Exception
     */
    public void deleteUser(int id) throws Exception;

    /**
     * 更新用户
     * @param user
     * @throws Exception
     */
    public void updateUser(User user) throws Exception;
}
~~~

## 4.2 UserMapper.xml

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.edu.wtu.mapper.UserMapper">

    <select id="findUserById" parameterType="int" resultType="cn.edu.wtu.po.User">
        select * from user where id=#{VALUE}
    </select>

    <select id="findUserByName" parameterType="java.lang.String" resultType="cn.edu.wtu.po.User">
        select * from user WHERE username LIKE '%${value}%'
    </select>

    <insert id="insertUser" parameterType="cn.edu.wtu.po.User">
        <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
            SELECT LAST_INSERT_ID()
        </selectKey>
        insert into user(username,birthday,sex,address) value(#{username},#{birthday},#{sex},#{address})
    </insert>

    <delete id="deleteUser" parameterType="cn.edu.wtu.po.User">
        delete from user where id = #{id}
    </delete>

    <update id="updateUser" parameterType="cn.edu.wtu.po.User">
        UPDATE user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
    </update>
</mapper>
~~~

## 4.3 在测试之前需要在SqlMapConfig.xml中加载mapper.xml这个映射文件

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125321.jpg)

## 4.4 UserMapperTest.java

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601125443.jpg)

~~~java
package cn.edu.wtu.mapper;

import cn.edu.wtu.po.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import java.io.InputStream;
import java.util.Date;
import java.util.List;

/**
 * @Package cn.edu.wtu.mapper
 * @ClassName UserMapperTest
 * @Description TODO
 * @Date 19/11/3 13:51
 * @Author LIM
 * @Version V1.0
 */
public class UserMapperTest {
    /**
     * mapper测试
     */
    private SqlSessionFactory sqlSessionFactory;
    @Before
    public void setUp() throws Exception {
        // 得到配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建会话工厂
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
    }

    @Test
    public void testFindUserById() throws Exception {
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象,mybatis自动调用
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = userMapper.findUserById(1);
        System.out.println(user.toString());
    }

    @Test
    public void testFindUserByName() throws Exception {
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象,mybatis自动调用
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = userMapper.findUserByName("小明");
        System.out.println(list.toString());
    }

    @Test
    public void testInsertUser() throws Exception {
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象,mybatis自动调用
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setUsername("张思");
        user.setSex("1");
        user.setBirthday(new Date());
        user.setAddress("武汉江夏区");
        userMapper.insertUser(user);
    }

    @Test
    public void testDeleteUser() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象,mybatis自动调用
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        userMapper.deleteUser(28);
    }

    @Test
    public void testUpdateUser() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建UserMapper对象,mybatis自动调用
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setUsername("张于");
        user.setSex("1");
        user.setBirthday(new Date());
        user.setAddress("武汉江夏区");
        userMapper.insertUser(user);
        userMapper.updateUser(user);
    }
}
~~~

# 5. 总结

## 5.1 原始dao开发问题

1.dao接口实现类方法中存在大量模板方法，设想能否将这些代码提取出来，大大减轻程序员的工作量。

2.调用sqlsession方法时将statement的id硬编码了

3.调用sqlsession方法时传入的变量，由于sqlsession方法使用泛型，即使变量类型传入错误，在编译阶段也不报错，不利于程序员开发。

## 5.2 mapper开发

* 只需要编写两个文件，mapper.java,mapper.xml。即可，不需要类来继承它

* mapper开发只需要遵守几个规范即可

  * 在mapper.xml中namespace等于mapper接口地址
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125557.jpg)

  * mapper.java接口中的方法名和mapper.xml中statement的id一致
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125628.jpg)

    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125736.jpg)

  * mapper.java接口中的方法输入参数类型和mapper.xml中statement的parameterType指定的类型一致。
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125805.jpg)

    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125845.jpg)

  * mapper.java接口中的方法返回值类型和mapper.xml中statement的resultType指定的类型一致。

    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601125917.jpg)

     

     ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130003.jpg)

* 其实，以上开发规范主要是对下边的代码进行统一生成：

  ~~~java
    User user = sqlSession.selectOne("test.findUserById", id);
    sqlSession.insert("test.insertUser", user);
    ......
  ~~~


# 6. 一些问题总结

- 代理对象内部调用`selectOne`或`selectList`
  - 如果mapper方法返回单个pojo对象（非集合对象），代理对象内部通过selectOne查询数据库。
  - 如果mapper方法返回集合对象，代理对象内部通过selectList查询数据库。


- mapper接口方法参数只能有一个是否影响系统开发

mapper接口方法参数只能有一个，系统是否不利于扩展维护?系统框架中，dao层的代码是被业务层公用的。即使mapper接口只有一个参数，可以使用包装类型的pojo满足不同的业务方法的需求。

注意：持久层方法的参数可以包装类型、map...等，service方法中建议不要使用包装类型（不利于业务层的可扩展）。

# 1. SqlMapConfig.xml中配置的内容和顺序

- properties（属性）
- settings（全局配置参数）
- **typeAliases（类型别名）**
- typeHandlers（类型处理器）
- *objectFactory（对象工厂）*
- *plugins（插件）*
- environments（环境集合属性对象）
  - environment（环境子属性对象）
    - transactionManager（事务管理）
    - dataSource（数据源）
- **mappers（映射器）**


(注：粗体是重点，斜体不常用)


# 2. properties（属性）

  * 将数据库连接的参数单独配置在，db.properties中，只需要在SqlMapConfig.xml中加载db.properties的属性值。在SqlMapConfig.xml中就不需要对数据库连接参数硬编码。

  * 好处：方便对参数进行统一管理，其它xml可以引用该db.properties

  * 特性： MyBatis 将按照下面的顺序来加载属性：

    * 在 properties 元素体内定义的属性首先被读取。
    * 然后会读取properties 元素中resource或 url 加载的属性，它会覆盖已读取的同名属性。
    * 最后读取parameterType传递的属性，它会覆盖已读取的同名属性。

  * 建议：

    * 不要在SqlMapConfig.xml的properties元素体内添加任何属性值，只将属性值定义在properties文件中。

    * 在properties文件中定义属性名要有一定的特殊性，如XXXXX.XXXXX.XXXX

      ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130131.jpg)

      

      ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130213.jpg)


# 3. settings（全局配置参数）

mybatis框架在运行时可以调整一些运行参数。比如：开启二级缓存、开启延迟加载。。全局参数将会影响mybatis的运行行为。具体如下：
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601130256.jpg)

# 4. typeAliases（类型别名）(重点)

  * 单个定义
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130401.jpg)
  * 批量定义（常用）
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130434.jpg)
    这样在其他地方就可以使用，例如：
    ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601130558.jpg)
* mybatis默认支持的别名
  ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601130711.jpg)

# 5. typeHandlers（类型处理器）

  * mybatis中通过typeHandlers完成jdbc类型和java类型的转换。通常情况下，mybatis提供的类型处理器满足日常需要，不需要自定义.
  * mybatis支持的类型处理器 
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130804.jpg)

# 6. objectFactory（对象工厂）

# 7. plugins（插件）

# 8. environments（环境集合属性对象）

  * environment（环境子属性对象）
    * transactionManager（事务管理）
    * dataSource（数据源）

# 9. mappers（映射器）

  * 通过resource
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131109.jpg)
  * 通过class
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131140.jpg)
  * 通过package(推荐使用)
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131232.jpg)

# 1. Mybatis输入映射（掌握）

* 通过parameterType指定输入参数的类型，类型可以是简单类型、hashmap、pojo的包装类型
  * 传递pojo的包装对象
    * 需求：完成用户信息的综合查询，需要传入查询条件很复杂（可能包括用户信息、其它信息，比如商品、订单的）
    * 针对上边需求，建议使用自定义的包装类型的pojo。在包装类型的pojo中将复杂的查询条件包装进去。 

## 1.1 目录结构

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131354.jpg)

## 1.2 UserCustom.java

~~~java
public class UserCustom extends User{
// 可扩展用户信息
}
~~~

## 1.3 UserQueryVo.java

~~~java
public class UserQueryVo {
// 这里包装所需的查询条件

/**
* 用户查询
*/
private UserCustom userCustom;

public UserCustom getUserCustom() {
    return userCustom;
}

public void setUserCustom(UserCustom userCustom) {
    this.userCustom = userCustom;
}

//可包装其他的查询条件,订单,商品......
}
~~~

## 1.4 UserMapper.java

~~~java
/**
* 用户综合查询
* @param userQueryVo
* @return UserCustom
* @throws Exception
*/
public UserCustom findUserList(UserQueryVo userQueryVo) throws Exception;
~~~

## 1.5 UserMapper.xml中配置新的查询

~~~xml
<!--用户综合查询-->
<select id="findUserList" parameterType="cn.edu.wtu.po.UserQueryVo" resultType="cn.edu.wtu.po.UserCustom">
    select *from user where user.sex=#{userCustom.sex} and
    user.username like "%${userCustom.username}%";
</select>
~~~

注意不要将`#{userCustom.sex}`中的`userCustom`写成`UserCustom`,前者指属性名(由于使用IDE提示自动补全，所以只是把类型名首字母小写了)，后者指类型名，这里是`UserQueryVo`类中的`userCustom`属性，是**属性名**。写错会报如下异常：

```
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'UserCustom' in 'class com.iot.mybatis.po.UserQueryVo'
### Cause: org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'UserCustom' in 'class com.iot.mybatis.po.UserQueryVo'
```

## 1.6 UserMapperTest.java中新增测试

~~~java
/**
* 用户综合查询
* @throws Exception
*/
@Test
public void testFindUserList() throws Exception{
    // 创建会话
    SqlSession sqlSession = sqlSessionFactory.openSession();
    // 创建代理对象
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    // 创建包对象,设置查询条件
    UserQueryVo userQueryVo = new UserQueryVo();
    UserCustom userCustom = new UserCustom();

    userCustom.setSex("1");
    userCustom.setUsername("张三");

    userQueryVo.setUserCustom(userCustom);

    UserCustom userCustom1 = userMapper.findUserList(userQueryVo);
    List<UserCustom> list = new ArrayList<>();
    list.add(userCustom1);
    System.out.println(list);
~~~

# 2. Mybatis输出映射（掌握）

## 2.1 resultType

  * 使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功
  * 如果查询出来的列名和pojo中的属性名全部不一致，没有创建pojo对象。
  * 只要查询出来的列名和pojo中的属性有一个一致，就会创建pojo对象

### 2.1.1 resultType的输出简单类型

- UserMapper.java

```java
  /**
   * 用户综合查询个数
   * @param userQueryVo
   * @return 用户个数
   * @throws Exception
   */
  public int findUserCount(UserQueryVo userQueryVo) throws Exception;
```

- UserMapper.xml

```xml
 <!-- 用户信息综合查询总数
        parameterType：指定输入类型和findUserList一样
        resultType：输出结果类型
    -->
    <select id="findUserCount" parameterType="com.iot.mybatis.po.UserQueryVo" resultType="int">
        SELECT count(*) FROM user WHERE user.sex=#{userCustom.sex} AND user.username LIKE '%${userCustom.username}%'
    </select>
```

- UserMapperTest.java

```java
/**
     * 用户综合查询 用户个数
     * @throws Exception
     */
    @Test
    public void testFindUserCount() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建代理对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        // 创建包装对象,设置查询条件
        UserQueryVo userQueryVo = new UserQueryVo();
        UserCustom userCustom = new UserCustom();

        userCustom.setUsername("1");
        userCustom.setUsername("陈晓明");
        userQueryVo.setUserCustom(userCustom);

        int count = userMapper.findUserCount(userQueryVo);

        System.out.println(count);
    }
```

- 小结

查询出来的结果集只有一行且一列，可以使用简单类型进行输出映射。


### 2.1.2 resultType的输出pojo对象和pojo列表

**不管是输出的pojo单个对象还是一个列表（list中包括pojo），在UserMapper.xml中`resultType`指定的类型是一样的。**

在mapper.java指定的方法返回值类型不一样：

- 输出单个pojo对象，方法返回值是单个对象类型

```java
//根据id查询用户信息
public User findUserById(int id) throws Exception;
```

- 输出pojo对象list，方法返回值是List\<Pojo>

```java
//根据用户名列查询用户列表
public List<User> findUserByName(String name) throws Exception;
```


**生成的动态代理对象中是根据mapper方法的返回值类型确定是调用`selectOne`(返回单个对象调用)还是`selectList` （返回集合对象调用 ）.**

## 2.2 resultMap

如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。

1.定义resultMap

2.使用resultMap作为statement的输出映射类型

- 定义reusltMap

```xml
    <!-- 定义resultMap
	将id_,username_ 和User类中的属性作映射

	type：resultMap最终映射的java对象类型,可以使用别名
	id：对resultMap的唯一标识
	 -->
	 <resultMap  id="userResultMap" type="cn.edu.wtu.po.Userr">
	 	<!-- id表示查询结果集中唯一标识 
	 	column：查询出来的列名
	 	property：type指定的pojo类型中的属性名
	 	最终resultMap对column和property作一个映射关系 （对应关系）
	 	-->
	 	<id column="id_" property="id"/>
	 	<!-- 
	 	result：对普通名映射定义
	 	column：查询出来的列名
	 	property：type指定的pojo类型中的属性名
	 	最终resultMap对column和property作一个映射关系 （对应关系）
	 	 -->
	 	<result column="username_" property="username"/>
	 
	 </resultMap>
```

- 使用resultMap作为statement的输出映射类型

```xml
<!-- 使用resultMap进行输出映射
        resultMap：指定定义的resultMap的id，如果这个resultMap在其它的mapper文件，前边需要加namespace
        -->
    <select id="findUserByIdResultMap" parameterType="int" resultMap="userResultMap">
        SELECT id id_,username username_ FROM USER WHERE id=#{value}
    </select>

```

- UserMapper.java

```java
//根据id查询用户信息，使用resultMap输出
public User findUserByIdResultMap(int id) throws Exception;
```

- 测试代码

```java
@Test
public void testFindUserByIdResultMap() throws Exception {

	SqlSession sqlSession = sqlSessionFactory.openSession();

	//创建UserMapper对象，mybatis自动生成mapper代理对象
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

	//调用userMapper的方法

	User user = userMapper.findUserByIdResultMap(1);

	System.out.println(user);


}
```


## 2.3 总结

* 使用resultType进行输出映射，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。
* 如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。

# 1. 什么是动态sql？

mybatis核心:对sql语句进行灵活操作，通过表达式进行判断，对sql进行灵活拼接、组装。

# 2. if判断

## 2.1 UserMapper.xml

```java
<!--用户综合查询-->
    <select id="findUserList" parameterType="cn.edu.wtu.po.UserQueryVo" resultType="cn.edu.wtu.po.UserCustom">
        select * from user
        <where>
            <if test="userCustom!=null">
            <if test="userCustom.sex!=null and userCustom.sex!=''">
                and user.sex = #{userCustom.sex}
            </if>
            <if test="userCustom.username!=null and userCustom.username!=''">
                and user.username LIKE '%${userCustom.username}%'
            </if>
            </if>
        </where>
    </select>
    <!--用户综合查询总数-->
    <select id="findUserCount" parameterType="cn.edu.wtu.po.UserQueryVo" resultType="java.lang.Integer">
                SELECT count(*) FROM user
                <where>
                    <if test="userCustom!=null">
                        <if test="userCustom.sex!=null and userCustom.sex!=''">
                            and user.sex = #{userCustom.sex}
                        </if>
                        <if test="userCustom.username!=null and userCustom.username!=''">
                            and user.username LIKE '%${userCustom.username}%'
                        </if>
                    </if>
                </where>
    </select>
```

## 2.2 测试结果

### 1.注释掉`testFindUserList()`方法中的`userCustom.setUsername("张三");`

UserMapperTest.java

```java
//由于这里使用动态sql，如果不设置某个值，条件不会拼接在sql中
userCustom.setSex("1");
// userCustom.setUsername("晓明");
userQueryVo.setUserCustom(userCustom);
```

输出

```
DEBUG [main] - Created connection 1293618474.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4d1b0d2a]
DEBUG [main] - ==>  Preparing: select * from user WHERE user.sex = ? 
DEBUG [main] - ==> Parameters: 1(String)
DEBUG [main] - <==      Total: 9
[id:10
username:张三
sex:1
birthday:Wed Nov 06 00:00:00 CST 2019
address:北京市, id:22
username:张晓明
sex:1
birthday:Tue Nov 26 00:00:00 CST 2019
address:天津, id:24
username:陈晓明
sex:1
birthday:Tue Apr 04 00:00:00 CST 2017
address:重庆, id:26
username:小明
sex:1
birthday:Tue Jul 24 00:00:00 CST 2018
address:武汉, id:28
username:李梅梅
sex:1
birthday:Wed Jul 26 00:00:00 CST 2000
address:西安, id:32
username:江江
sex:1
birthday:Wed Jul 05 00:00:00 CST 2017
address:宁夏, id:33
username:红红
sex:1
birthday:Tue Jun 20 00:00:00 CST 2017
address:广州, id:34
username:木木
sex:1
birthday:Tue Apr 26 00:00:00 CST 2016
address:河北, id:37
username:陈晓明
sex:1
birthday:Wed Nov 27 00:00:00 CST 2019
address:null]
```

可以看到sql语句为`reparing: SELECT * FROM user WHERE user.sex=? `，没有username的部分


### 2.`userQueryVo`设为null,则`userCustom`为null

```java
//List<UserCustom> list = userMapper.findUserList(userQueryVo);
List<UserCustom> list = userMapper.findUserList(null);
```

输出

```
DEBUG [main] - Created connection 1859039536.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@6eceb130]
DEBUG [main] - ==>  Preparing: select * from user 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 15
[id:1
username:王五
sex:2
birthday:Tue Nov 05 00:00:00 CST 2019
address:null, id:10
username:张三
sex:1
birthday:Wed Nov 06 00:00:00 CST 2019
address:北京市, id:22
username:张晓明
sex:1
birthday:Tue Nov 26 00:00:00 CST 2019
address:天津, id:24
username:陈晓明
sex:1
birthday:Tue Apr 04 00:00:00 CST 2017
address:重庆, id:26
username:小明
sex:1
birthday:Tue Jul 24 00:00:00 CST 2018
address:武汉, id:27
username:宋江涛new2
sex:男
birthday:Sun Nov 03 00:00:00 CST 2019
address:山西太原new, id:28
username:李梅梅
sex:1
birthday:Wed Jul 26 00:00:00 CST 2000
address:西安, id:29
username:丽丽
sex:2
birthday:Tue Oct 29 00:00:00 CST 2019
address:新疆, id:31
username:涵涵
sex:2
birthday:Tue Jun 12 00:00:00 CST 2018
address:西藏, id:32
username:江江
sex:1
birthday:Wed Jul 05 00:00:00 CST 2017
address:宁夏, id:33
username:红红
sex:1
birthday:Tue Jun 20 00:00:00 CST 2017
address:广州, id:34
username:木木
sex:1
birthday:Tue Apr 26 00:00:00 CST 2016
address:河北, id:35
username:宋江涛
sex:男
birthday:Sun Nov 03 00:00:00 CST 2019
address:山西, id:36
username:张宇
sex:2
birthday:Sun Nov 03 00:00:00 CST 2019
address:襄阳, id:37
username:陈晓明
sex:1
birthday:Wed Nov 27 00:00:00 CST 2019
address:null]
```

可以看到sql语句变为了`SELECT * FROM user`

# 3. sql片段(重点)

将上边实现的动态sql判断代码块抽取出来，组成一个sql片段。其它的statement中就可以引用sql片段。


## 3.1 定义sql片段

```xml
<!-- 定义sql片段
id：sql片段的唯 一标识

经验：是基于单表来定义sql片段，这样话这个sql片段可重用性才高
在sql片段中不要包括 where
 -->
<sql id="query_user_where">
    <if test="userCustom!=null">
        <if test="userCustom.sex!=null and userCustom.sex!=''">
            AND user.sex = #{userCustom.sex}
        </if>
        <if test="userCustom.username!=null and userCustom.username!=''">
            AND user.username LIKE '%${userCustom.username}%'
        </if>
    </if>
</sql>
```

## 3.2 引用sql片段

```xml
<!-- 用户信息综合查询
    #{userCustom.sex}:取出pojo包装对象中性别值
    ${userCustom.username}：取出pojo包装对象中用户名称
 -->
<select id="findUserList" parameterType="com.iot.mybatis.po.UserQueryVo"
        resultType="com.iot.mybatis.po.UserCustom">
    SELECT * FROM user
    <!--  where 可以自动去掉条件中的第一个and -->
    <where>
        <!-- 引用sql片段 的id，如果refid指定的id不在本mapper文件中，需要前边加namespace -->
        <include refid="query_user_where"></include>
        <!-- 在这里还可以引用其它的sql片段  -->
    </where>
</select>
```

# 4. foreach标签

向sql传递数组或List，mybatis使用foreach解析

在用户查询列表和查询总数的statement中增加多个id输入查询。两种方法，sql语句如下：

- `SELECT * FROM USER WHERE id=1 OR id=10 OR id=16`
- `SELECT * FROM USER WHERE id IN(1,10,16)`

一个使用OR,一个使用IN


## 4.1 在输入参数类型中添加`List<Integer> ids`传入多个id

```java
public class UserQueryVo {

    //传入多个id
   private List<Integer> ids;

    public List<Integer> getIds() {
        return ids;
    }

    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
}
```

## 4.2 修改mapper.xml

```xml
 select * from user
 <where>
    <if test="ids!=null">
        <!-- 使用 foreach遍历传入ids
        collection：指定输入 对象中集合属性
        item：每个遍历生成对象中
        open：开始遍历时拼接的串
        close：结束遍历时拼接的串
        separator：遍历的两个对象中需要拼接的串
        -->
        <!-- 使用实现下边的sql拼接：
        AND (id=1 OR id=10 OR id=16)
        -->
        <foreach collection="ids" item="user_id" open="AND (" close=")" separator="or">
            <!-- 每个遍历需要拼接的串 -->
            id=#{user_id}
        </foreach>
        
        <!-- 实现  “ and id IN(1,10,16)”拼接 -->
        <!-- <foreach collection="ids" item="user_id" open="and id IN(" close=")" separator=",">
            每个遍历需要拼接的串
            #{user_id}
        </foreach> -->
    </if>
</where>
```


## 4.3 测试代码

在`testFindUserList`中加入

```java
//传入多个id
List<Integer> ids = new ArrayList<Integer>();
ids.add(1);
ids.add(10);
ids.add(16);
//将ids通过userQueryVo传入statement中
userQueryVo.setIds(ids);
```

#	1. 数据模型分析思路

- 每张表记录的数据内容
  分模块对每张表记录的内容进行熟悉，相当于你学习系统需求（功能）的过程。

- 每张表重要的字段设置
  非空字段、外键字段

- 数据库级别表与表之间的关系
  外键关系

- 表与表之间的业务关系
  在分析表与表之间的业务关系时一定要建立在某个业务意义基础上去分析。

# 2. 数据模型分析

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131659.png)

- 用户表user：记录了购买商品的用户信息
- 订单表orders：记录了用户所创建的订单（购买商品的订单）
- 订单明细表orderdetail：记录了订单的详细信息即购买商品的信息
- 商品表items：记录了商品信息


## 2.1 表与表之间的业务关系：

在分析表与表之间的业务关系时需要建立在某个业务意义基础上去分析。先分析数据级别之间有关系的表之间的业务关系：
	

### 1. usre和orders：

user--->orders：一个用户可以创建多个订单，一对多
orders--->user：一个订单只由一个用户创建，一对一

### 2. orders和orderdetail：

orders--->orderdetail：一个订单可以包括多个订单明细，因为一个订单可以购买多个商品，每个商品的购买信息在orderdetail记录，一对多关系

orderdetail---> orders：一个订单明细只能包括在一个订单中，一对一


### 3. orderdetail和itesm：

orderdetail--->itesms：一个订单明细只对应一个商品信息，一对一

items---> orderdetail:一个商品可以包括在多个订单明细 ，一对多

再分析数据库级别没有关系的表之间是否有业务关系：

### 4. orders和items：

orders和items之间可以通过orderdetail表建立关系。

# 3. 订单商品数据模型建表sql

```sql
/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
/*Table structure for table `items` */

CREATE TABLE `items` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL COMMENT '商品名称',
  `price` float(10,1) NOT NULL COMMENT '商品定价',
  `detail` text COMMENT '商品描述',
  `pic` varchar(64) DEFAULT NULL COMMENT '商品图片',
  `createtime` datetime NOT NULL COMMENT '生产日期',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Table structure for table `orderdetail` */

CREATE TABLE `orderdetail` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `orders_id` int(11) NOT NULL COMMENT '订单id',
  `items_id` int(11) NOT NULL COMMENT '商品id',
  `items_num` int(11) DEFAULT NULL COMMENT '商品购买数量',
  PRIMARY KEY (`id`),
  KEY `FK_orderdetail_1` (`orders_id`),
  KEY `FK_orderdetail_2` (`items_id`),
  CONSTRAINT `FK_orderdetail_1` FOREIGN KEY (`orders_id`) REFERENCES `orders` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `FK_orderdetail_2` FOREIGN KEY (`items_id`) REFERENCES `items` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;

/*Table structure for table `orders` */

CREATE TABLE `orders` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL COMMENT '下单用户id',
  `number` varchar(32) NOT NULL COMMENT '订单号',
  `createtime` datetime NOT NULL COMMENT '创建订单时间',
  `note` varchar(100) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`),
  KEY `FK_orders_1` (`user_id`),
  CONSTRAINT `FK_orders_id` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

/*Table structure for table `user` */

CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` date DEFAULT NULL COMMENT '生日',
  `sex` char(1) DEFAULT NULL COMMENT '性别',
  `address` varchar(256) DEFAULT NULL COMMENT '地址',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=27 DEFAULT CHARSET=utf8;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

```

测试数据

```sql
/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
/*Data for the table `items` */

insert  into `items`(`id`,`name`,`price`,`detail`,`pic`,`createtime`) values (1,'台式机',3000.0,'该电脑质量非常好！！！！',NULL,'2015-02-03 13:22:53'),(2,'笔记本',6000.0,'笔记本性能好，质量好！！！！！',NULL,'2015-02-09 13:22:57'),(3,'背包',200.0,'名牌背包，容量大质量好！！！！',NULL,'2015-02-06 13:23:02');

/*Data for the table `orderdetail` */

insert  into `orderdetail`(`id`,`orders_id`,`items_id`,`items_num`) values (1,3,1,1),(2,3,2,3),(3,4,3,4),(4,4,2,3);

/*Data for the table `orders` */

insert  into `orders`(`id`,`user_id`,`number`,`createtime`,`note`) values (3,1,'1000010','2015-02-04 13:22:35',NULL),(4,1,'1000011','2015-02-03 13:22:41',NULL),(5,10,'1000012','2015-02-12 16:13:23',NULL);

/*Data for the table `user` */

insert  into `user`(`id`,`username`,`birthday`,`sex`,`address`) values (1,'王五',NULL,'2',NULL),(10,'张三','2014-07-10','1','北京市'),(16,'张小明',NULL,'1','河南郑州'),(22,'陈小明',NULL,'1','河南郑州'),(24,'张三丰',NULL,'1','河南郑州'),(25,'陈小明',NULL,'1','河南郑州'),(26,'王五',NULL,NULL,NULL);

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

# 1. 一对一

## 1.1. resultType实现

- sql语句


确定查询的主表：订单表

确定查询的关联表：用户表

关联查询使用内连接？还是外连接？

由于orders表中有一个外键（user_id），通过外键关联查询用户表只能查询出一条记录，可以使用内连接。

```sql
SELECT 
  orders.*,
  USER.username,
  USER.sex,
  USER.address 
FROM
  orders,
  USER 
WHERE orders.user_id = user.id
```

- 创建pojo

将上边sql查询的结果映射到pojo中，pojo中必须包括所有查询列名。

原始的Orders.java不能映射全部字段，需要新创建的pojo。

创建一个pojo继承包括查询字段较多的po类。

对应数据表的几个pojo类(Items,Orderdetail,Orders)就是把该类的属性名设为和数据表列字段名相同，并为这些属性添加getter和setter，在这里就不贴代码了，只贴出对应于关联查询的自定义pojo类`OrdersCustom`的代码

```java
package cn.edu.wtu.po;

/**
 * @Package cn.edu.wtu.po
 * @ClassName OrdersCustom
 * @Description 通过此类映射订单和用户查询的结果，让此类继承包括 字段较多的pojo类
 * @Date 19/11/10 11:17
 * @Author LIM
 * @Version V1.0
 */
public class OrdersCustom extends Orders{
    private String username;
    private String sex;
    private String address;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "OrdersCustom{" +
                "username='" + username + '\'' +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```



- OrderMapper.xml

```xml
 <!-- 查询订单关联查询用户信息 -->
<select id="findOrdersUser"  resultType="com.iot.mybatis.po.OrdersCustom">
  SELECT
      orders.*,
      user.username,
      user.sex,
      user.address
    FROM
      orders,
      user
    WHERE orders.user_id = user.id
</select>
```


- OrdeMapper.java

```java
//查询订单关联查询用户信息
public List<OrdersCustom> findOrdersUser()throws Exception;
}
```



## 1.2. resultMap实现

使用resultMap将查询结果中的订单信息映射到Orders对象中，在orders类中添加User属性，将关联查询出来的用户信息映射到orders对象中的user属性中。

- 定义resultMap


```xml
   <!-- 订单查询关联用户的resultMap
    将整个查询的结果映射到cn.edu.wtu.po.Orders中
    -->
    <resultMap id="OrdersUsersResultMap" type="cn.edu.wtu.po.Orders">
        <!-- 配置映射的订单信息 -->
        <!-- id：指定查询列中的唯一标识，订单信息的中的唯 一标识，如果有多个列组成唯一标识，配置多个id
            column：订单信息的唯一标识列
            property：订单信息的唯一标识列所映射到Orders中哪个属性
          -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="createtime" property="createtime"/>
        <result column="number" property="number"/>
        <result column="note" property="note"/>
        <!-- 配置映射的关联的用户信息 -->
        <!-- association：用于映射关联查询单个对象的信息
        property：要将关联查询的用户信息映射到Orders中哪个属性
         -->
        <association property="user" javaType="cn.edu.wtu.po.User">
            <!-- id：关联查询用户的唯一标识
            column：指定唯 一标识用户信息的列
            javaType：映射到user的哪个属性
             -->
            <id column="user_id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
            <result column="address" property="address"/>
        </association>
    </resultMap>
```


- statement定义


```xml
<!-- 查询订单关联查询用户信息 -->
<select id="findOrdersUserResultMap" resultMap="OrdersUserResultMap">
    SELECT
    orders.*,
    user.username,
    user.sex,
    user.address
    FROM
    orders,
    user
    WHERE orders.user_id = user.id
</select>
```

- OrderMapper.java

```java
//查询订单关联查询用户使用resultMap
public List<Orders> findOrdersUserResultMap()throws Exception;
```

- 测试代码

```java
@Test
public void testFindOrdersUserResultMap() throws Exception {

	SqlSession sqlSession = sqlSessionFactory.openSession();
	// 创建代理对象
	OrdersMapperCustom ordersMapperCustom = sqlSession
			.getMapper(OrdersMapperCustom.class);

	// 调用maper的方法
	List<Orders> list = ordersMapperCustom.findOrdersUserResultMap();

	System.out.println(list);

	sqlSession.close();
}
```

## 1.3. resultType和resultMap实现一对一查询小结

实现一对一查询：

- resultType：使用resultType实现较为简单，如果pojo中没有包括查询出来的列名，需要增加列名对应的属性，即可完成映射。如果没有查询结果的特殊要求建议使用resultType。
- resultMap：需要单独定义resultMap，实现有点麻烦，如果对查询结果有特殊的要求，使用resultMap可以完成将关联查询映射pojo的属性中。
- resultMap可以实现延迟加载，resultType无法实现延迟加载。

# 2. 一对多

## 2.1. 需求

查询订单及订单明细的信息。(根据数据库模型分析的结果来查询)
使用resultMap
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601132022.png)

## 2.2 要求

对orders映射不能出现重复记录。

## 2.3. 解决思路

1. 在orders.java类中添加List<>, orderDetails属性。
2. 最终会将订单信息映射到orders中，订单所对应的订单明细映射到orders中的orderDetails属性中。
3. 映射成的orders记录数为两条（orders信息不重复）
4. 每个orders中的orderDetails属性存储了该订单所对应的订单明细

## 2.4. resultMap

```xml
<!-- 订单及订单明细的resultMap
    使用extends继承，不用在中配置订单信息和用户信息的映射
-->
<resultMap id="OrdersAndOrderDetailResultMap" type="cn.edu.wtu.po.Orders" extends="OrdersUsersResultMap">
        <!-- 订单信息 -->
        <!-- 用户信息 -->
        <!-- 使用extends继承，不用在中配置订单信息和用户信息的映射 -->
        <!-- 订单明细信息
        一个订单关联查询出了多条明细，要使用collection进行映射
        collection：对关联查询到多条记录映射到集合对象中
        property：将关联查询到多条记录映射到cn.edu.wtu.po.Orders哪个属性
        ofType：指定映射到list集合属性中pojo的类型
         -->
        <collection property="orderDetails" ofType="cn.edu.wtu.po.OrderDetail">
            <!-- id：订单明细唯 一标识
            property:要将订单明细的唯 一标识 映射到cn.edu.wtu.po.OrderDetail的哪个属性
              -->
            <id column="orderDetail_id" property="id"/>
            <result column="items_id" property="itemsId"/>
            <result column="order_id" property="ordersId"/>
            <result column="items_num" property="itemsNum"/>
        </collection>
</resultMap>
```

## 2.5. OrderMapper.xml

```xml
<!-- 查询订单关联查询用户及订单明细，使用resultMap -->
    <select id="findOrdersAndOrderDetailResultMap" resultMap="OrdersAndOrderDetailResultMap">
        SELECT
          orders.*,
          user.username,
          user.sex,
          user.address,
          orderdetail.id orderdetail_id,
          orderdetail.items_id,
          orderdetail.items_num,
          orderdetail.orders_id
        FROM
          orders,
          user,
          orderdetail
        WHERE orders.user_id = user.id AND orderdetail.orders_id=orders.id;
    </select>
```

## 2.6. OrderMapper.java

```java
/**
     * 查询订单关联查询用户及订单明细，使用resultMap
     * @return Orders的列表
     * @throws Exception
     */
    public List<Orders> findOrdersAndOrderDetailResultMap() throws Exception;
```

## 2.7. 测试

```java
@Test
    public void testFindOrdersAndOrderDetailResultMap() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建代理对象
        OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);

        List<Orders> lists = orderMapper.findOrdersAndOrderDetailResultMap();

        for (Orders list:lists){
            System.out.println(list.toString());
        }
    }
```

## 2.8. 小结

mybatis使用resultMap的collection对关联查询的多条记录映射到一个list集合属性中。

使用resultType实现：将订单明细映射到orders中的orderdetails中，需要自己处理，使用双重循环遍历，去掉重复记录，将订单明细放在orderdetails中。


另外，下面这篇文章对一对多的resultMap机制解释的很清楚：

> [MyBatis：一对多表关系详解(从案例中解析)](http://blog.csdn.net/xzm_rainbow/article/details/15336933)

# 3. 多对多

## 3.1 需求

查询用户及用户购买商品信息。

查询主表是：用户表

关联表：由于用户和商品没有直接关联，通过订单和订单明细进行关联，所以关联表：orders、orderdetail、items

## 3.2 sql

```sql
SELECT 
  orders.*,
  user.username,
  user.sex,
  user.address,
  orderdetail.id orderdetail_id,
  orderdetail.items_id,
  orderdetail.items_num,
  orderdetail.orders_id,
  items.name items_name,
  items.detail items_detail,
  items.price items_price
FROM
  orders,
  user,
  orderdetail,
  items
WHERE orders.user_id = user.id AND orderdetail.orders_id=orders.id AND orderdetail.items_id = items.id
```

## 3.3 映射思路

1. 将用户信息映射到user中。

2. 在user类中添加订单列表属性`List<Orders> orderslist`，将用户创建的订单映射到orderslist

3. 在Orders中添加订单明细列表属性`List<OrderDetail>orderdetials`，将订单的明细映射到orderdetials

4. 在OrderDetail中添加`Items`属性，将订单明细所对应的商品映射到Items

## 3.4 resultMap

```xml
<!-- 查询用户及购买的商品 -->
    <resultMap id="UserAndItemsResultMap" type="cn.edu.wtu.po.User">
        <!-- 用户信息 -->
        <id column="user_id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <result column="address" property="address"/>
        <!-- 订单信息
        一个用户对应多个订单，使用collection映射
         -->
        <collection property="ordersList" ofType="cn.edu.wtu.po.Orders">
            <id column="id" property="id"/>
            <result column="user_id" property="userId"/>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
            <result column="note" property="note"/>
            <!-- 订单明细
             一个订单包括 多个明细
             -->
            <collection property="orderDetails" ofType="cn.edu.wtu.po.OrderDetail">
                <id column="orderDetail_id" property="id"/>
                <result column="items_id" property="itemsId"/>
                <result column="items_num" property="itemsNum"/>
                <result column="orders_id" property="ordersId"/>
                <!-- 商品信息
                 一个订单明细对应一个商品
                 -->
                <association property="items" javaType="cn.edu.wtu.po.Items">
                    <id column="items_id" property="id"/>
                    <result column="items_name" property="name"/>
                    <result column="items_detail" property="detail"/>
                    <result column="items_price" property="price"/>
                </association>
            </collection>
        </collection>
    </resultMap>
```

## 3.5 OrderMapper.xml

```xml
<!-- 查询用户及购买的商品信息，使用resultMap -->
    <select id="findUserAndItemsResultMap" resultMap="UserAndItemsResultMap">
        SELECT
        orders.*,
        USER.username,
        USER.sex,
        USER.address,
        orderdetail.id orderdetail_id,
        orderdetail.items_id,
        orderdetail.items_num,
        orderdetail.orders_id,
        items.name items_name,
        items.detail items_detail,
        items.price items_price
        FROM
        orders,
        USER,
        orderdetail,
        items
        WHERE orders.user_id = user.id AND orderdetail.orders_id=orders.id AND orderdetail.items_id = items.id
    </select>
```

## 3.6 OrderMapper.java

```java
/**
     * 查询用户购买商品信息
     * @return User的列表
     * @throws Exception
     */
    public List<User>  findUserAndItemsResultMap()throws Exception;
```

## 3.7 测试

```java
@Test
    public void testFindUserAndItemsResultMap() throws Exception{
        // 创建会话
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建代理对象
        OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);

        List<User> lists = orderMapper.findUserAndItemsResultMap();

        for (User list:lists){
            System.out.println(list.toString());
        }
    }
```

## 3.8 多对多查询总结

将查询用户购买的商品信息明细清单，（用户名、用户地址、购买商品名称、购买商品时间、购买商品数量）

针对上边的需求就使用resultType将查询到的记录映射到一个扩展的pojo中，很简单实现明细清单的功能。

一对多是多对多的特例，如下需求：

查询用户购买的商品信息，用户和商品的关系是多对多关系。

- 需求1：

查询字段：用户账号、用户名称、用户性别、商品名称、商品价格(最常见)

企业开发中常见明细列表，用户购买商品明细列表，

使用resultType将上边查询列映射到pojo输出。

- 需求2：

查询字段：用户账号、用户名称、购买商品数量、商品明细（鼠标移上显示明细）

使用resultMap将用户购买的商品明细列表映射到user对象中。

总结：

使用resultMap是针对那些对查询结果映射有特殊要求的功能，比如特殊要求映射成list中包括多个list。



# 4. 总结

## 4.1 resultType

   - 作用：将查询结果按照sql列名pojo属性名一致性映射到pojo中。
   - 场合：常见一些明细记录的展示，比如用户购买商品明细，将关联查询信息全部展示在页面时，此时可直接使用resultType将每一条记录映射到pojo中，在前端页面遍历list（list中是pojo）即可。

## 4.2 resultMap

使用association和collection完成一对一和一对多高级映射（对结果有特殊的映射要求）。

### 4.2.1 association(一对一)

- 作用：将关联查询信息映射到一个pojo对象中。
- 场合：为了方便查询关联信息可以使用association将关联订单信息映射为用户对象的pojo属性中，比如：查询订单及关联用户信息。

使用resultType无法将查询结果映射到pojo对象的pojo属性中，根据对结果集查询遍历的需要选择使用resultType还是resultMap。
	

### 4.2.2 collection(一对多)

- 作用：将关联查询信息映射到一个list集合中。
- 场合：为了方便查询遍历关联信息可以使用collection将关联信息映射到list集合中，比如：查询用户权限范围模块及模块下的菜单，可使用collection将模块映射到模块list中，将菜单列表映射到模块对象的菜单list属性中，这样的作的目的也是方便对查询结果集进行遍历查询。如果使用resultType无法将查询结果映射到list集合中。

resultMap可以实现高级映射（使用`association`、`collection`实现一对一及一对多映射），`association`、`collection`具备延迟加载功能。

延迟加载：先从单表查询、需要时再从关联表去关联查询，大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。


需求：

如果查询订单并且关联查询用户信息。如果先查询订单信息即可满足要求，当我们需要查询用户信息时再查询用户信息。把对用户信息的按需去查询就是延迟加载。

# 1. 使用association实现延迟加载

- mapper.xml

需要定义两个mapper的方法对应的statement。

1.只查询订单信息

`SELECT * FROM orders`

在查询订单的statement中使用association去延迟加载（执行）下边的satatement(关联查询用户信息)

```xml
<!-- 查询订单关联查询用户，用户信息需要延迟加载 -->
<select id="findOrdersUserLazyLoading" resultMap="OrdersUserLazyLoadingResultMap">
    SELECT * FROM orders
</select>
```

2.关联查询用户信息

通过上边查询到的订单信息中user_id去关联查询用户信息,使用UserMapper.xml中的findUserById

```xml
<select id="findUserById" parameterType="int" resultType="cn.edu.wtu.po.User">
    SELECT * FROM  user  WHERE id=#{value}
</select>
```

上边先去执行findOrdersUserLazyLoading，当需要去查询用户的时候再去执行findUserById，通过resultMap的定义将延迟加载执行配置起来。


- 延迟加载resultMap

```xml
<!-- 延迟加载的resultMap -->
<resultMap type="cn.edu.wtu.po.Orders" id="OrdersUserLazyLoadingResultMap">
    <!--对订单信息进行映射配置  -->
    <id column="id" property="id"/>
    <result column="user_id" property="userId"/>
    <result column="number" property="number"/>
    <result column="createtime" property="createtime"/>
    <result column="note" property="note"/>
    <!-- 实现对用户信息进行延迟加载
    select：指定延迟加载需要执行的statement的id（是根据user_id查询用户信息的statement）
    要使用OrderMapper.xml中findUserById完成根据用户id(user_id)用户信息的查询，如果findUserById不在本mapper中需要前边加namespace
    column：订单信息中关联用户信息查询的列，是user_id
    关联查询的sql理解为：
    SELECT orders.*,
    (SELECT username FROM USER WHERE orders.user_id = user.id)username,
    (SELECT sex FROM USER WHERE orders.user_id = user.id)sex
     FROM orders
     -->
    <association property="user"  javaType="cn.edu.wtu.po.User"
                 select="cn.edu.wtu.mapper.OrderMapper.findUserById"
                 column="user_id">
     <!-- 实现对用户信息进行延迟加载 -->

    </association>

</resultMap>
```

**与非延迟加载的主要区别就在`association`标签属性多了`select`和`column`**

```xml
<association property="user"  javaType="cn.edu.wtu.po.User"
             select="cn.edu.wtu.mapper.OrderMapper.findUserById"
             column="user_id">
```

- OrderMapper.java

```java
//查询订单关联查询用户，用户信息是延迟加载
public List<Orders> findOrdersUserLazyLoading()throws Exception;
```



- 测试思路
  - 执行上边mapper方法(`findOrdersUserLazyLoading`)，内部去调用`cn.edu.wtu.mapper.OrdersMapperCustom`中的`findOrdersUserLazyLoading`只查询orders信息（单表）。
   - 在程序中去遍历上一步骤查询出的List<Orders>，当我们调用Orders中的getUser方法时，开始进行延迟加载。
   - 延迟加载，去调用OrderMapper.xml中findUserbyId这个方法获取用户信息。

- 延迟加载配置

mybatis默认没有开启延迟加载，需要在SqlMapConfig.xml中setting配置。

在mybatis核心配置文件中配置：lazyLoadingEnabled、aggressiveLazyLoading


| 设置项                | 描述                                                         | 允许值     | 默认值 |
| :-------------------- | :----------------------------------------------------------- | :--------- | :----- |
| lazyLoadingEnabled    | 全局性设置懒加载。如果设为‘false’，则所有相关联的都会被初始化加载 | true/false | false  |
| aggressiveLazyLoading | 当设置为‘true’的时候，懒加载的对象可能被任何懒属性全部加载。否则，每个属性都按需加载。 | true/false | true   |


在SqlMapConfig.xml中配置：

```xml
<settings>
    <!-- 打开延迟加载 的开关 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 将积极加载改为消极加载即按需要加载 -->
    <setting name="aggressiveLazyLoading" value="false"/>
    <!-- 开启二级缓存 -->
   <!-- <setting name="cacheEnabled" value="true"/>-->
</settings>
```

- 测试代码

```java
// 查询订单关联查询用户，用户信息使用延迟加载
@Test
public void testFindOrdersUserLazyLoading() throws Exception {
	SqlSession sqlSession = sqlSessionFactory.openSession();// 创建代理对象
	OrdersMapperCustom ordersMapperCustom = sqlSession
			.getMapper(OrdersMapperCustom.class);
	// 查询订单信息（单表）
	List<Orders> list = ordersMapperCustom.findOrdersUserLazyLoading();

	// 遍历上边的订单列表
	for (Orders orders : list) {
		// 执行getUser()去查询用户信息，这里实现按需加载
		User user = orders.getUser();
		System.out.println(user);
	}
}
```


# 2. 延迟加载思考

不使用mybatis提供的association及collection中的延迟加载功能，如何实现延迟加载？？

实现方法如下：

定义两个mapper方法：

- 查询订单列表
- 根据用户id查询用户信息

实现思路：

先去查询第一个mapper方法，获取订单信息列表；在程序中（service），按需去调用第二个mapper方法去查询用户信息。

总之，使用延迟加载方法，先去查询简单的sql（最好单表，也可以关联查询），再去按需要加载关联查询的其它信息。

# 1. 查询缓存

mybatis提供查询缓存，用于减轻数据压力，提高数据库性能。

mybaits提供一级缓存，和二级缓存。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601132621.png)

一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

为什么要用缓存？

如果缓存中有数据就不用从数据库中获取，大大提高系统性能。


# 2. 一级缓存

## 2.1 一级缓存工作原理

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601132658.png)

第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。

如果sqlSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。


## 2.2 一级缓存测试

mybatis默认支持一级缓存，不需要在配置文件去配置。

按照上边一级缓存原理步骤去测试。

测试代码

```java
// 一级缓存测试
@Test
public void testCache1() throws Exception {
	SqlSession sqlSession = sqlSessionFactory.openSession();// 创建代理对象
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

	// 下边查询使用一个SqlSession
	// 第一次发起请求，查询id为1的用户
	User user1 = userMapper.findUserById(1);
	System.out.println(user1);

	// 如果sqlSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

	// 更新user1的信息
	// user1.setUsername("测试用户22");
	// userMapper.updateUser(user1);
	// //执行commit操作去清空缓存
	// sqlSession.commit();

	// 第二次发起请求，查询id为1的用户
	User user2 = userMapper.findUserById(1);
	System.out.println(user2);

	sqlSession.close();

}
```



1.不执行更新操作，输出:

```
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Created connection 110771485.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User [id=1, username=王五, sex=2, birthday=null, address=null]
User [id=1, username=王五, sex=2, birthday=null, address=null]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - Returned connection 110771485 to pool.
```

2.取消测试代码中更新的的注释，输出：

```
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Created connection 110771485.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User [id=1, username=王五, sex=2, birthday=null, address=null]
DEBUG [main] - ==>  Preparing: update user set username=?,birthday=?,sex=?,address=? where id=? 
DEBUG [main] - ==> Parameters: 测试用户22(String), null, 2(String), null, 1(Integer)
DEBUG [main] - <==    Updates: 1
DEBUG [main] - Committing JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User [id=1, username=测试用户22, sex=2, birthday=null, address=null]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@69a3d1d]
DEBUG [main] - Returned connection 110771485 to pool.
```


## 2.3 一级缓存应用

正式开发，是将mybatis和spring进行整合开发，事务控制在service中。

一个service方法中包括 很多mapper方法调用。

```
service{
	//开始执行时，开启事务，创建SqlSession对象
	//第一次调用mapper的方法findUserById(1)
	
	//第二次调用mapper的方法findUserById(1)，从一级缓存中取数据
	//方法结束，sqlSession关闭
}
```

如果是执行两次service调用查询相同的用户信息，不走一级缓存，因为session方法结束，sqlSession就关闭，一级缓存就清空。

# 3. 二级缓存

## 3.1 二级缓存原理


![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601132750.png)

首先开启mybatis的二级缓存.

sqlSession1去查询用户id为1的用户信息，查询到用户信息会将查询数据存储到二级缓存中。

如果SqlSession3去执行相同mapper下sql，执行commit提交，清空该mapper下的二级缓存区域的数据。

sqlSession2去查询用户id为1的用户信息，去缓存中找是否存在数据，如果存在直接从缓存中取出数据。

二级缓存与一级缓存区别，**二级缓存的范围更大，多个sqlSession可以共享一个UserMapper的二级缓存区域**。

UserMapper有一个二级缓存区域（按namespace分），其它mapper也有自己的二级缓存区域（按namespace分）。每一个namespace的mapper都有一个二级缓存区域，两个mapper的namespace如果相同，这两个mapper执行sql查询到数据将存在相同的二级缓存区域中。


## 3.2 开启二级缓存

mybaits的二级缓存是mapper范围级别，除了在SqlMapConfig.xml设置二级缓存的总开关，还要在具体的mapper.xml中开启二级缓存。

在核心配置文件SqlMapConfig.xml中加入`<setting name="cacheEnabled" value="true"/>`

| 设置项       | 描述                                              | 允许值     | 默认值 |
| :----------- | :------------------------------------------------ | :--------- | :----- |
| cacheEnabled | 对在此配置文件下的所有cache 进行全局性开/关设置。 | true/false | true   |

```xml
<!-- 开启二级缓存 -->
<setting name="cacheEnabled" value="true"/>
```

在UserMapper.xml中开启二缓存，UserMapper.xml下的sql执行完成会存储到它的缓存区域（HashMap）。

```xml
<mapper namespace="cn.edu.wtu.mapper.OrderMapper">
<!-- 开启本mapper的namespace下的二级缓存-->
<cache />

...

</mapper>
```


## 3.3 调用pojo类实现序列化接口

```java
public class User implements Serializable{
    ....
}
```

为了将缓存数据取出执行反序列化操作，因为二级缓存数据存储介质多种多样，不一定在内存。

## 3.4 测试方法

```java
// 二级缓存测试
@Test
public void testCache2() throws Exception {
	SqlSession sqlSession1 = sqlSessionFactory.openSession();
	SqlSession sqlSession2 = sqlSessionFactory.openSession();
	SqlSession sqlSession3 = sqlSessionFactory.openSession();
	// 创建代理对象
	UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);
	// 第一次发起请求，查询id为1的用户
	User user1 = userMapper1.findUserById(1);
	System.out.println(user1);

	//这里执行关闭操作，将sqlsession中的数据写到二级缓存区域
	sqlSession1.close();


//		//使用sqlSession3执行commit()操作
//		UserMapper userMapper3 = sqlSession3.getMapper(UserMapper.class);
//		User user  = userMapper3.findUserById(1);
//		user.setUsername("张明明");
//		userMapper3.updateUser(user);
//		//执行提交，清空UserMapper下边的二级缓存
//		sqlSession3.commit();
//		sqlSession3.close();



	UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
	// 第二次发起请求，查询id为1的用户
	User user2 = userMapper2.findUserById(1);
	System.out.println(user2);

	sqlSession2.close();
}
```



1.无更新，输出

```
DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.0
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Created connection 103887628.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User [id=1, username=测试用户22, sex=2, birthday=null, address=null]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Returned connection 103887628 to pool.
DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.5
User [id=1, username=测试用户22, sex=2, birthday=null, address=null]
```

2.有更新，输出


```
DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.0
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Created connection 103887628.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User [id=1, username=测试用户22, sex=2, birthday=null, address=null]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Returned connection 103887628 to pool.
DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.5
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Checked out connection 103887628 from pool.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - ==>  Preparing: update user set username=?,birthday=?,sex=?,address=? where id=? 
DEBUG [main] - ==> Parameters: 张明明(String), null, 2(String), null, 1(Integer)
DEBUG [main] - <==    Updates: 1
DEBUG [main] - Committing JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Returned connection 103887628 to pool.
DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.3333333333333333
DEBUG [main] - Opening JDBC Connection
DEBUG [main] - Checked out connection 103887628 from pool.
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User [id=1, username=张明明, sex=2, birthday=null, address=null]
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]
DEBUG [main] - Returned connection 103887628 to pool.
```


## 3.5 useCache配置

在statement中设置`useCache=false`可以禁用当前select语句的二级缓存，即每次查询都会发出sql去查询，默认情况是true，即该sql使用二级缓存。

`<select id="findOrderListResultMap" resultMap="ordersUserMap" useCache="false">`

总结：针对每次查询都需要最新的数据sql，要设置成useCache=false，禁用二级缓存。



## 3.6 刷新缓存（就是清空缓存）

刷新缓存就是清空缓存。在mapper的同一个namespace中，如果有其它insert、update、delete操作数据后需要刷新缓存，如果不执行刷新缓存会出现脏读。

 设置statement配置中的`flushCache="true"`属性，默认情况下为true即刷新缓存，如果改成false则不会刷新。使用缓存时如果手动修改数据库表中的查询数据会出现脏读。如下：

`<insert id="insertUser" parameterType="cn.edu.wtu.po.User" flushCache="true">`

总结：一般下执行完commit操作都需要刷新缓存，`flushCache=true`表示刷新缓存，这样可以避免数据库脏读。

## 3.7 应用场景和局限性

- 应用场景

对于访问多的查询请求且用户对查询结果实时性要求不高，此时可采用mybatis二级缓存技术降低数据库访问量，提高访问速度，业务场景比如：耗时较高的统计分析sql、电话账单查询sql等。

实现方法如下：通过设置刷新间隔时间，由mybatis每隔一段时间自动清空缓存，根据数据变化频率设置缓存刷新间隔flushInterval，比如设置为30分钟、60分钟、24小时等，根据需求而定。


- 局限性

mybatis二级缓存对细粒度的数据级别的缓存实现不好，比如如下需求：对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次都能查询最新的商品信息，此时如果使用mybatis的二级缓存就无法实现当一个商品变化时只刷新该商品的缓存信息而不刷新其它商品的信息，因为mybaits的二级缓存区域以mapper为单位划分，当一个商品信息变化会将所有商品信息的缓存数据全部清空。解决此类问题需要在业务层根据需求对数据有针对性缓存。

**ehcache是一个分布式缓存框架**

# 1. 分布缓存

我们系统为了提高系统并发，性能、一般对系统进行分布式部署（集群部署方式）

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601132939.png)


不使用分布缓存，缓存的数据在各各服务单独存储，不方便系统开发。所以要使用分布式缓存对缓存数据进行集中管理。

mybatis无法实现分布式缓存，需要和其它分布式缓存框架进行整合。

# 2. 整合方法(掌握)

mybatis提供了一个`cache`接口，如果要实现自己的缓存逻辑，实现`cache`接口开发即可。

mybatis和ehcache整合，mybatis和ehcache整合包中提供了一个cache接口的实现类。


```java
package org.apache.ibatis.cache;

import java.util.concurrent.locks.ReadWriteLock;

/**
 * SPI for cache providers.
 * 
 * One instance of cache will be created for each namespace.
 * 
 * The cache implementation must have a constructor that receives the cache id as an String parameter.
 * 
 * MyBatis will pass the namespace as id to the constructor.
 * 
 * <pre>
 * public MyCache(final String id) {
 *  if (id == null) {
 *    throw new IllegalArgumentException("Cache instances require an ID");
 *  }
 *  this.id = id;
 *  initialize();
 * }
 * </pre>
 *
 * @author Clinton Begin
 */

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
   * Optional. It is not called by the core.
   * 
   * @param key The key
   * @return The object that was removed
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


mybatis默认实现cache类是：

```java
package org.apache.ibatis.cache.impl;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;

import org.apache.ibatis.cache.Cache;
import org.apache.ibatis.cache.CacheException;

/**
 * @author Clinton Begin
 */
public class PerpetualCache implements Cache {

  private String id;

  private Map<Object, Object> cache = new HashMap<Object, Object>();

  public PerpetualCache(String id) {
    this.id = id;
  }

  public String getId() {
    return id;
  }

  public int getSize() {
    return cache.size();
  }

  public void putObject(Object key, Object value) {
    cache.put(key, value);
  }

  public Object getObject(Object key) {
    return cache.get(key);
  }

  public Object removeObject(Object key) {
    return cache.remove(key);
  }

  public void clear() {
    cache.clear();
  }

  public ReadWriteLock getReadWriteLock() {
    return null;
  }

  public boolean equals(Object o) {
    if (getId() == null) throw new CacheException("Cache instances require an ID.");
    if (this == o) return true;
    if (!(o instanceof Cache)) return false;

    Cache otherCache = (Cache) o;
    return getId().equals(otherCache.getId());
  }

  public int hashCode() {
    if (getId() == null) throw new CacheException("Cache instances require an ID.");
    return getId().hashCode();
  }

}
```

## 2.1 整合ehcache

- 加入ehcache包
  - ehcache-core-2.6.5.jar
  - mybatis-ehcache-1.0.2.jar

配置mapper中`cache`中的`type`为ehcache对cache接口的实现类型

```xml
 <!-- 开启本mapper的namespace下的二级缓存
    type：指定cache接口的实现类的类型，mybatis默认使用PerpetualCache
    要和ehcache整合，需要配置type为ehcache实现cache接口的类型
    <cache />
    -->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

## 2.2 加入ehcache的配置文件

在classpath下配置ehcache.xml


```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	<diskStore path="F:\develop\ehcache" />
	<defaultCache 
		maxElementsInMemory="1000" 
		maxElementsOnDisk="10000000"
		eternal="false" 
		overflowToDisk="false" 
		timeToIdleSeconds="120"
		timeToLiveSeconds="120" 
		diskExpiryThreadIntervalSeconds="120"
		memoryStoreEvictionPolicy="LRU">
	</defaultCache>
</ehcache>
```

# Mybatis源码

## 简介

MyBatis 是一款优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集

Mybatis可以将Sql语句配置在XML文件中，避免将Sql语句硬编码在Java类中。与JDBC相比：

1. Mybatis通过参数映射方式，可以将参数灵活的配置在SQL语句中的配置文件中，避免在Java类中配置参数（JDBC）
2. Mybatis通过输出映射机制，将结果集的检索自动映射成相应的Java对象，避免对结果集手工检索（JDBC）
3. Mybatis可以通过Xml配置文件对数据库连接进行管理。

## 架构

我们把Mybatis的功能架构分为三层：

(1)API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。

(2)数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。

(3)基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

![Mybatis](F:\work\openGuide\Middleware\Mybatis.png)

## 结构

MyBatis整个项目的包结构如下：

```
.
└── org
    └── apache
        └── ibatis
            ├── annotations (注解)
            ├── binding (绑定)
            ├── builder(构建)
            	├──annotation
            	├──xml
            ├── cache  (缓存)
            	├──decorators
            	├──impl
            ├── cursor
            ├── datasource (数据源)
            ├──  --		org.apache.ibatis.datasource.jndi
            ├──  --		org.apache.ibatis.datasource.pooled
            ├──  --		org.apache.ibatis.datasource.unpooled
            ├── exceptions (异常)
            ├── executor (执行器)
            	├──keygen
            	├──loader
            		├──cglib
            		├──javassist
            	├──parameter
            	├──result
            	├──resultset
            	├──statement
            	--	    org.apache.ibatis.executor.Executor
            ├── io (通过类加载器在jar包中寻找一个package下满足条件(比如某个接口的子类)的所有类)
            ├── jdbc (jdbc工具)
            ├── logging (日志)
			├──  --		org.apache.ibatis.logging.commons
			├──  --		org.apache.ibatis.logging.jdbc
			├──  --     org.apache.ibatis.logging.jdk14 
			├──  --     org.apache.ibatis.logging.log4j
			├──  --     org.apache.ibatis.logging.log4j2
			├──  --     org.apache.ibatis.logging.nologging
			├──  --     org.apache.ibatis.logging.slf4j
			├──  --     org.apache.ibatis.logging.stdout
            ├── mapping (映射)
            ├── parsing (解析,xml解析，${} 格式的字符串解析)
            ├── plugin  (插件)
            ├── reflection (反射)
            ├── scripting (脚本)
            ├── session (会话)
            	├──defaults
            ├──  --     org.apache.ibatis.session.SqlSession
            ├── transaction (事务)
            ├──  --     org.apache.ibatis.transaction.jdbc
            ├──  --     org.apache.ibatis.transaction.managed
            └── type (类型处理器,实现java和jdbc中的类型之间转换)
```

- SqlSession 作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
- Executor MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
- StatementHandler 封装了JDBC Statement操作，负责对JDBCstatement的操作，如设置参数、将Statement结果集转换成List集合。
- ParameterHandler 负责对用户传递的参数转换成JDBC Statement 所需要的参数
- ResultSetHandler *负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
- TypeHandler 负责java数据类型和jdbc数据类型之间的映射和转换
- MappedStatement MappedStatement维护了一条<select|update|delete|insert>节点的封
- SqlSource 负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
- BoundSql 表示动态生成的SQL语句以及相应的参数信息
- Configuration MyBatis所有的配置信息都维持在Configuration对象之中

## 流程

**(1)加载配置并初始化**

- 触发条件：加载配置文件
- 配置来源于两个地方，一处是配置文件，一处是Java代码的注解，将SQL的配置信息加载成为一个个MappedStatement对象（包括了传入参数映射配置、执行的SQL语句、结果映射配置），存储在内存中。

**(2)接收调用请求**

- 触发条件：调用Mybatis提供的API
- 传入参数：为SQL的ID和传入参数对象
- 处理过程：将请求传递给下层的请求处理层进行处理。

**(3)处理操作请求 触发条件：API接口层传递请求过来**　

- 传入参数：为SQL的ID和传入参数对象
- 处理过程：

　　(A)根据SQL的ID查找对应的MappedStatement对象。
　　(B)根据传入参数对象解析MappedStatement对象，得到最终要执行的SQL和执行传入参数。
　　(C)获取数据库连接，根据得到的最终SQL语句和执行传入参数到数据库执行，并得到执行结果。
　　(D)根据MappedStatement对象中的结果映射配置对得到的执行结果进行转换处理，并得到最终的处理结果。
　　(E)释放连接资源。

**(4)返回处理结果将最终的处理结果返回。**

## 核心类

#### SqlSessionFactoryBuilder

每一个MyBatis的应用程序的入口是SqlSessionFactoryBuilder。

它的作用是通过XML配置文件创建Configuration对象（当然也可以在程序中自行创建），然后通过build方法创建SqlSessionFactory对象。没有必要每次访问Mybatis就创建一次SqlSessionFactoryBuilder，通常的做法是创建一个全局的对象就可以了

#### SqlSessionFactory

由SqlSessionFactoryBuilder创建

它的主要功能是创建SqlSession对象，和SqlSessionFactoryBuilder对象一样，没有必要每次访问Mybatis就创建一次SqlSessionFactory，通常的做法是创建一个全局的对象就可以了。SqlSessionFactory对象一个必要的属性是Configuration对象，它是保存Mybatis全局配置的一个配置对象，通常由SqlSessionFactoryBuilder从XML配置文件创建

#### SqlSession

SqlSession对象的主要功能是完成一次数据库的访问和结果的映射，它类似于数据库的session概念，由于不是线程安全的，所以SqlSession对象的作用域需限制方法内。SqlSession的默认实现类是DefaultSqlSession，它有两个必须配置的属性：Configuration和Executor。Configuration前文已经描述这里不再多说。SqlSession对数据库的操作都是通过Executor来完成的

#### Executor

Executor对象在创建Configuration对象的时候创建，并且缓存在Configuration对象里。Executor对象的主要功能是调用StatementHandler访问数据库，并将查询结果存入缓存中（如果配置了缓存的话）。

#### StatementHandler

StatementHandler是真正访问数据库的地方，并调用ResultSetHandler处理查询结果。

#### ResultSetHandler

处理查询结果。

## 源码

### 源码--MyBatis HelloWorld

源码分析之前先搭一个mybatis的demo，这个在看源码的时候能起到了很大的作用，因为在看源码的时候，会恍然大悟，为什么要这么配置，为什么要这么写。（老鸟可以跳过这篇）

#### 创建maven项目

**pom.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mybatis.chenhao</groupId>
    <artifactId>mybatisDemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <!-- mybatis版本号 -->
        <mybatis.version>3.4.2</mybatis.version>
    </properties>
    <dependencies>

        <!--mybatis依赖 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <!-- mysql驱动包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.44</version>
        </dependency>

    </dependencies>

</project>
```

#### 创建mybatis的配置文件

**mybatis-config.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 引入外部配置文件 -->
    <properties resource="db.properties"></properties>
    <environments default="default">
        <environment id="default">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper class="mapper.DemoMapper"></mapper>
    </mappers>
</configuration>
```

**db.properties**

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8
jdbc.username=chenhao
jdbc.password=123456
```



#### entity和mapper

**Employee**

```
package entity;

/***
 *
 *@Author ChenHao
 *@Description:
 *@Date: Created in 14:58 2019/10/26
 *@Modified By:
 *
 */
public class Employee {
    int id;
    String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

**EmployeeMapper**

```
package mapper;

import entity.Employee;
import java.util.List;

/***
 *
 *@Author ChenHao
 *@Description:
 *@Date: Created in 14:58 2019/10/26
 *@Modified By:
 *
 */
public interface EmployeeMapper {
    List<Employee> getAll();
}
```

**EmployeeMapper.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper.EmployeeMapper">

    <resultMap id="baseMap" type="entity.Employee">
        <result property="id" column="id" jdbcType="INTEGER"></result>
        <result property="name" column="name" jdbcType="VARCHAR"></result>

    </resultMap>
    <select id="getAll" resultMap="baseMap">
        select * from employee
    </select>
</mapper>
```

#### 测试

```
public static void main(String[] args) throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
         EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
         List<Employee> all = employeeMapper.getAll();
         for (Employee item : all)
            System.out.println(item);
    } finally {
        sqlSession.close();
    }
}
```

测试结果：

```
Employee{id=1, name='name1'}
Employee{id=2, name='name2'}
Employee{id=3, name='name3'}
```

好了，MyBatis HelloWorld我们已经搭建完了，后面的源码分析文章我们将以这个为基础来分析

### 源码--Configuration的创建过程

我们使用mybatis操作数据库都是通过SqlSession的API调用，而创建SqlSession是通过SqlSessionFactory。下面我们就看看SqlSessionFactory的创建过程。

## 配置文件解析入口

我们看看第一篇文章中的测试方法

```
 1 public static void main(String[] args) throws IOException {
 2     String resource = "mybatis-config.xml";
 3     InputStream inputStream = Resources.getResourceAsStream(resource);
 4     SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
 5     SqlSession sqlSession = sqlSessionFactory.openSession();
 6     try {
 7         Employee employeeMapper = sqlSession.getMapper(Employee.class);
 8          List<Employee> all = employeeMapper.getAll();
 9          for (Employee item : all)
10             System.out.println(item);
11     } finally {
12         sqlSession.close();
13     }
14 }
```

首先，我们使用 MyBatis 提供的工具类 Resources 加载配置文件，得到一个输入流。然后再通过 SqlSessionFactoryBuilder 对象的`build`方法构建 SqlSessionFactory 对象。所以这里的 build 方法是我们分析配置文件解析过程的入口方法。我们看看build里面是代码：

```
public SqlSessionFactory build(InputStream inputStream) {
    // 调用重载方法
    return build(inputStream, null, null);
}

public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
        // 创建配置文件解析器
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        // 调用 parser.parse() 方法解析配置文件，生成 Configuration 对象
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
    // 创建 DefaultSqlSessionFactory，将解析配置文件后生成的Configuration传入
    return new DefaultSqlSessionFactory(config);
}
```

SqlSessionFactory是通过SqlSessionFactoryBuilder的build方法创建的，build方法内部是通过一个XMLConfigBuilder对象解析mybatis-config.xml文件生成一个Configuration对象。XMLConfigBuilder从名字可以看出是解析Mybatis配置文件的，其实它是继承了一个父类BaseBuilder，其每一个子类多是以XMLXXXXXBuilder命名的，也就是其子类都对应解析一种xml文件或xml文件中一种元素。

![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191026165910820-1306401062.png)

我们看看XMLConfigBuilder的构造方法：

```
private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
}
```

可以看到调用了父类的构造方法，并传入一个new Configuration()对象，这个对象也就是最终的Mybatis配置对象

我们先来看看其基类BaseBuilder

```
public abstract class BaseBuilder {
  protected final Configuration configuration;
  protected final TypeAliasRegistry typeAliasRegistry;
  protected final TypeHandlerRegistry typeHandlerRegistry;
 
  public BaseBuilder(Configuration configuration) {
    this.configuration = configuration;
    this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
    this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
  }
  ....
}
```

BaseBuilder中只有三个成员变量,而typeAliasRegistry和typeHandlerRegistry都是直接从Configuration的成员变量获得的，接着我们看看Configuration这个类

Configuration类位于mybatis包的org.apache.ibatis.session目录下，其属性就是对应于mybatis的全局配置文件mybatis-config.xml的配置，将XML配置中的内容解析赋值到Configuration对象中。

由于XML配置项有很多，所以Configuration类的属性也很多。先来看下Configuration对于的XML配置文件示例：

```
<?xml version="1.0" encoding="UTF-8"?>    

<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">    
<!-- 全局配置顶级节点 -->
<configuration>    
     
     <!-- 属性配置,读取properties中的配置文件 -->
    <properties resource="db.propertis">
       <property name="XXX" value="XXX"/>
    </properties>
    
    <!-- 类型别名 -->
    <typeAliases>
       <!-- 在用到User类型的时候,可以直接使用别名,不需要输入User类的全部路径 -->
       <typeAlias type="com.luck.codehelp.entity.User" alias="user"/>
    </typeAliases>

    <!-- 类型处理器 -->
    <typeHandlers>
        <!-- 类型处理器的作用是完成JDBC类型和java类型的转换,mybatis默认已经由了很多类型处理器,正常无需自定义-->
    </typeHandlers>
    
    <!-- 对象工厂 -->
    <!-- mybatis创建结果对象的新实例时,会通过对象工厂来完成，mybatis有默认的对象工厂，正常无需配置 -->
    <objectFactory type=""></objectFactory>
    
    <!-- 插件 -->
    <plugins>
        <!-- 可以自定义拦截器通过plugin标签加入 -->
       <plugin interceptor="com.lucky.interceptor.MyPlugin"></plugin>
    </plugins>
    
    <!-- 全局配置参数 -->
    <settings>   
        <setting name="cacheEnabled" value="false" />   
        <setting name="useGeneratedKeys" value="true" /><!-- 是否自动生成主键 -->
        <setting name="defaultExecutorType" value="REUSE" />   
        <setting name="lazyLoadingEnabled" value="true"/><!-- 延迟加载标识 -->
        <setting name="aggressiveLazyLoading" value="true"/><!--有延迟加载属性的对象是否延迟加载 -->
        <setting name="multipleResultSetsEnabled" value="true"/><!-- 是否允许单个语句返回多个结果集 -->
        <setting name="useColumnLabel" value="true"/><!-- 使用列标签而不是列名 -->
        <setting name="autoMappingBehavior" value="PARTIAL"/><!-- 指定mybatis如何自动映射列到字段属性;NONE:自动映射;PARTIAL:只会映射结果没有嵌套的结果;FULL:可以映射任何复杂的结果 -->
        <setting name="defaultExecutorType" value="SIMPLE"/><!-- 默认执行器类型 -->
        <setting name="defaultFetchSize" value=""/>
        <setting name="defaultStatementTimeout" value="5"/><!-- 驱动等待数据库相应的超时时间 ,单位是秒-->
        <setting name="safeRowBoundsEnabled" value="false"/><!-- 是否允许使用嵌套语句RowBounds -->
        <setting name="safeResultHandlerEnabled" value="true"/>
        <setting name="mapUnderscoreToCamelCase" value="false"/><!-- 下划线列名是否自动映射到驼峰属性：如user_id映射到userId -->
        <setting name="localCacheScope" value="SESSION"/><!-- 本地缓存（session是会话级别） -->
        <setting name="jdbcTypeForNull" value="OTHER"/><!-- 数据为空值时,没有特定的JDBC类型的参数的JDBC类型 -->
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/><!-- 指定触发延迟加载的对象的方法 -->
        <setting name="callSettersOnNulls" value="false"/><!--如果setter方法或map的put方法，如果检索到的值为null时，数据是否有用  -->
        <setting name="logPrefix" value="XXXX"/><!-- mybatis日志文件前缀字符串 -->
        <setting name="logImpl" value="SLF4J"/><!-- mybatis日志的实现类 -->
        <setting name="proxyFactory" value="CGLIB"/><!-- mybatis代理工具 -->
    </settings>  

    <!-- 环境配置集合 -->
    <environments default="development">    
        <environment id="development">    
            <transactionManager type="JDBC"/><!-- 事务管理器 -->
            <dataSource type="POOLED"><!-- 数据库连接池 -->    
                <property name="driver" value="com.mysql.jdbc.Driver" />    
                <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8" />    
                <property name="username" value="root" />    
                <property name="password" value="root" />    
            </dataSource>    
        </environment>    
    </environments>    
    
    <!-- mapper文件映射配置 -->
    <mappers>    
        <mapper resource="com/luck/codehelp/mapper/UserMapper.xml"/>    
    </mappers>    
</configuration>
```

而对于XML的配置，Configuration类的属性是和XML配置对应的。Configuration类属性如下：

```
public class Configuration {
  protected Environment environment;//运行环境

  protected boolean safeRowBoundsEnabled = false;
  protected boolean safeResultHandlerEnabled = true;
  protected boolean mapUnderscoreToCamelCase = false;
  protected boolean aggressiveLazyLoading = true; //true：有延迟加载属性的对象被调用时完全加载任意属性;false：每个属性按需要加载
  protected boolean multipleResultSetsEnabled = true;//是否允许多种结果集从一个单独的语句中返回
  protected boolean useGeneratedKeys = false;//是否支持自动生成主键
  protected boolean useColumnLabel = true;//是否使用列标签
  protected boolean cacheEnabled = true;//是否使用缓存标识
  protected boolean callSettersOnNulls = false;//
  protected boolean useActualParamName = true;

  protected String logPrefix;
  protected Class <? extends Log> logImpl;
  protected Class <? extends VFS> vfsImpl;
  protected LocalCacheScope localCacheScope = LocalCacheScope.SESSION;
  protected JdbcType jdbcTypeForNull = JdbcType.OTHER;
  protected Set<String> lazyLoadTriggerMethods = new HashSet<String>(Arrays.asList(new String[] { "equals", "clone", "hashCode", "toString" }));
  protected Integer defaultStatementTimeout;
  protected Integer defaultFetchSize;
  protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE;
  protected AutoMappingBehavior autoMappingBehavior = AutoMappingBehavior.PARTIAL;//指定mybatis如果自动映射列到字段和属性,PARTIAL会自动映射简单的没有嵌套的结果,FULL会自动映射任意复杂的结果
  protected AutoMappingUnknownColumnBehavior autoMappingUnknownColumnBehavior = AutoMappingUnknownColumnBehavior.NONE;

  protected Properties variables = new Properties();
  protected ReflectorFactory reflectorFactory = new DefaultReflectorFactory();
  protected ObjectFactory objectFactory = new DefaultObjectFactory();
  protected ObjectWrapperFactory objectWrapperFactory = new DefaultObjectWrapperFactory();

  protected boolean lazyLoadingEnabled = false;//是否延时加载，false则表示所有关联对象即使加载,true表示延时加载
  protected ProxyFactory proxyFactory = new JavassistProxyFactory(); // #224 Using internal Javassist instead of OGNL

  protected String databaseId;

  protected Class<?> configurationFactory;

  protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
  protected final InterceptorChain interceptorChain = new InterceptorChain();
  protected final TypeHandlerRegistry typeHandlerRegistry = new TypeHandlerRegistry();
  protected final TypeAliasRegistry typeAliasRegistry = new TypeAliasRegistry();
  protected final LanguageDriverRegistry languageRegistry = new LanguageDriverRegistry();

  protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection");
  protected final Map<String, Cache> caches = new StrictMap<Cache>("Caches collection");
  protected final Map<String, ResultMap> resultMaps = new StrictMap<ResultMap>("Result Maps collection");
  protected final Map<String, ParameterMap> parameterMaps = new StrictMap<ParameterMap>("Parameter Maps collection");
  protected final Map<String, KeyGenerator> keyGenerators = new StrictMap<KeyGenerator>("Key Generators collection");

  protected final Set<String> loadedResources = new HashSet<String>(); //已经加载过的resource(mapper)
  protected final Map<String, XNode> sqlFragments = new StrictMap<XNode>("XML fragments parsed from previous mappers");

  protected final Collection<XMLStatementBuilder> incompleteStatements = new LinkedList<XMLStatementBuilder>();
  protected final Collection<CacheRefResolver> incompleteCacheRefs = new LinkedList<CacheRefResolver>();
  protected final Collection<ResultMapResolver> incompleteResultMaps = new LinkedList<ResultMapResolver>();
  protected final Collection<MethodResolver> incompleteMethods = new LinkedList<MethodResolver>();

  protected final Map<String, String> cacheRefMap = new HashMap<String, String>();
  
  //其他方法略
}
```

加载的过程是SqlSessionFactoryBuilder根据xml配置的文件流，通过XMLConfigBuilder的parse方法进行解析得到一个Configuration对象，我们再看看其构造函数

```
 1 public Configuration() {
 2     this.safeRowBoundsEnabled = false;
 3     this.safeResultHandlerEnabled = true;
 4     this.mapUnderscoreToCamelCase = false;
 5     this.aggressiveLazyLoading = true;
 6     this.multipleResultSetsEnabled = true;
 7     this.useGeneratedKeys = false;
 8     this.useColumnLabel = true;
 9     this.cacheEnabled = true;
10     this.callSettersOnNulls = false;
11     this.localCacheScope = LocalCacheScope.SESSION;
12     this.jdbcTypeForNull = JdbcType.OTHER;
13     this.lazyLoadTriggerMethods = new HashSet(Arrays.asList("equals", "clone", "hashCode", "toString"));
14     this.defaultExecutorType = ExecutorType.SIMPLE;
15     this.autoMappingBehavior = AutoMappingBehavior.PARTIAL;
16     this.autoMappingUnknownColumnBehavior = AutoMappingUnknownColumnBehavior.NONE;
17     this.variables = new Properties();
18     this.reflectorFactory = new DefaultReflectorFactory();
19     this.objectFactory = new DefaultObjectFactory();
20     this.objectWrapperFactory = new DefaultObjectWrapperFactory();
21     this.mapperRegistry = new MapperRegistry(this);
22     this.lazyLoadingEnabled = false;
23     this.proxyFactory = new JavassistProxyFactory();
24     this.interceptorChain = new InterceptorChain();
25     this.typeHandlerRegistry = new TypeHandlerRegistry();
26     this.typeAliasRegistry = new TypeAliasRegistry();
27     this.languageRegistry = new LanguageDriverRegistry();
28     this.mappedStatements = new Configuration.StrictMap("Mapped Statements collection");
29     this.caches = new Configuration.StrictMap("Caches collection");
30     this.resultMaps = new Configuration.StrictMap("Result Maps collection");
31     this.parameterMaps = new Configuration.StrictMap("Parameter Maps collection");
32     this.keyGenerators = new Configuration.StrictMap("Key Generators collection");
33     this.loadedResources = new HashSet();
34     this.sqlFragments = new Configuration.StrictMap("XML fragments parsed from previous mappers");
35     this.incompleteStatements = new LinkedList();
36     this.incompleteCacheRefs = new LinkedList();
37     this.incompleteResultMaps = new LinkedList();
38     this.incompleteMethods = new LinkedList();
39     this.cacheRefMap = new HashMap();
40     this.typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
41     this.typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
42     this.typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
43     this.typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
44     this.typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
45     this.typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
46     this.typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
47     this.typeAliasRegistry.registerAlias("LRU", LruCache.class);
48     this.typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
49     this.typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
50     this.typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);
51     this.typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
52     this.typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);
53     this.typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
54     this.typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
55     this.typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
56     this.typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
57     this.typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
58     this.typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
59     this.typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);
60     this.typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
61     this.typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);
62     this.languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
63     this.languageRegistry.register(RawLanguageDriver.class);
64 }
```

我们看到第26行**this.typeAliasRegistry = new TypeAliasRegistry();，并且第40到61行向** **typeAliasRegistry 注册了很多别名，我们看看\**TypeAliasRegistry\****

```
public class TypeAliasRegistry {
    private final Map<String, Class<?>> TYPE_ALIASES = new HashMap();

    public TypeAliasRegistry() {
        this.registerAlias("string", String.class);
        this.registerAlias("byte", Byte.class);
        this.registerAlias("long", Long.class);
        this.registerAlias("short", Short.class);
        this.registerAlias("int", Integer.class);
        this.registerAlias("integer", Integer.class);
        this.registerAlias("double", Double.class);
        this.registerAlias("float", Float.class);
        this.registerAlias("boolean", Boolean.class);
        this.registerAlias("byte[]", Byte[].class);
        this.registerAlias("long[]", Long[].class);
        this.registerAlias("short[]", Short[].class);
        this.registerAlias("int[]", Integer[].class);
        this.registerAlias("integer[]", Integer[].class);
        this.registerAlias("double[]", Double[].class);
        this.registerAlias("float[]", Float[].class);
        this.registerAlias("boolean[]", Boolean[].class);
        this.registerAlias("_byte", Byte.TYPE);
        this.registerAlias("_long", Long.TYPE);
        this.registerAlias("_short", Short.TYPE);
        this.registerAlias("_int", Integer.TYPE);
        this.registerAlias("_integer", Integer.TYPE);
        this.registerAlias("_double", Double.TYPE);
        this.registerAlias("_float", Float.TYPE);
        this.registerAlias("_boolean", Boolean.TYPE);
        this.registerAlias("_byte[]", byte[].class);
        this.registerAlias("_long[]", long[].class);
        this.registerAlias("_short[]", short[].class);
        this.registerAlias("_int[]", int[].class);
        this.registerAlias("_integer[]", int[].class);
        this.registerAlias("_double[]", double[].class);
        this.registerAlias("_float[]", float[].class);
        this.registerAlias("_boolean[]", boolean[].class);
        this.registerAlias("date", Date.class);
        this.registerAlias("decimal", BigDecimal.class);
        this.registerAlias("bigdecimal", BigDecimal.class);
        this.registerAlias("biginteger", BigInteger.class);
        this.registerAlias("object", Object.class);
        this.registerAlias("date[]", Date[].class);
        this.registerAlias("decimal[]", BigDecimal[].class);
        this.registerAlias("bigdecimal[]", BigDecimal[].class);
        this.registerAlias("biginteger[]", BigInteger[].class);
        this.registerAlias("object[]", Object[].class);
        this.registerAlias("map", Map.class);
        this.registerAlias("hashmap", HashMap.class);
        this.registerAlias("list", List.class);
        this.registerAlias("arraylist", ArrayList.class);
        this.registerAlias("collection", Collection.class);
        this.registerAlias("iterator", Iterator.class);
        this.registerAlias("ResultSet", ResultSet.class);
    }

    public void registerAliases(String packageName) {
        this.registerAliases(packageName, Object.class);
    }
    //略
}
```

其实TypeAliasRegistry里面有一个**HashMap，**并且在TypeAliasRegistry的构造器中注册很多别名到这个hashMap中，好了，到现在我们只是创建了一个 XMLConfigBuilder，在其构造器中我们创建了一个 Configuration 对象，接下来我们看看将mybatis-config.xml解析成Configuration中对应的属性，也就是**parser.parse()**方法：

**XMLConfigBuilder**

```
1 public Configuration parse() {
2     if (parsed) {
3         throw new BuilderException("Each XMLConfigBuilder can only be used once.");
4     }
5     parsed = true;
6     // 解析配置
7     parseConfiguration(parser.evalNode("/configuration"));
8     return configuration;
9 }
```

我们看看第7行，注意一个 xpath 表达式 - `/configuration`。这个表达式代表的是 MyBatis 的`<configuration/>`标签，这里选中这个标签，并传递给`parseConfiguration`方法。我们继续跟下去。

```
private void parseConfiguration(XNode root) {
    try {
        // 解析 properties 配置
        propertiesElement(root.evalNode("properties"));

        // 解析 settings 配置，并将其转换为 Properties 对象
        Properties settings = settingsAsProperties(root.evalNode("settings"));

        // 加载 vfs
        loadCustomVfs(settings);

        // 解析 typeAliases 配置
        typeAliasesElement(root.evalNode("typeAliases"));

        // 解析 plugins 配置
        pluginElement(root.evalNode("plugins"));

        // 解析 objectFactory 配置
        objectFactoryElement(root.evalNode("objectFactory"));

        // 解析 objectWrapperFactory 配置
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));

        // 解析 reflectorFactory 配置
        reflectorFactoryElement(root.evalNode("reflectorFactory"));

        // settings 中的信息设置到 Configuration 对象中
        settingsElement(settings);

        // 解析 environments 配置
        environmentsElement(root.evalNode("environments"));

        // 解析 databaseIdProvider，获取并设置 databaseId 到 Configuration 对象
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));

        // 解析 typeHandlers 配置
        typeHandlerElement(root.evalNode("typeHandlers"));

        // 解析 mappers 配置
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```

## 解析 properties 配置

先来看一下 properties 节点的配置内容。如下：

```
<properties resource="db.properties">
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</properties>
```

我为 properties 节点配置了一个 resource 属性，以及两个子节点。接着我们看看propertiesElement的逻辑

```
private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
        // 解析 propertis 的子节点，并将这些节点内容转换为属性对象 Properties
        Properties defaults = context.getChildrenAsProperties();
        // 获取 propertis 节点中的 resource 和 url 属性值
        String resource = context.getStringAttribute("resource");
        String url = context.getStringAttribute("url");

        // 两者都不用空，则抛出异常
        if (resource != null && url != null) {
            throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
        }
        if (resource != null) {
            // 从文件系统中加载并解析属性文件
            defaults.putAll(Resources.getResourceAsProperties(resource));
        } else if (url != null) {
            // 通过 url 加载并解析属性文件
            defaults.putAll(Resources.getUrlAsProperties(url));
        }
        Properties vars = configuration.getVariables();
        if (vars != null) {
            defaults.putAll(vars);
        }
        parser.setVariables(defaults);
        // 将属性值设置到 configuration 中
        configuration.setVariables(defaults);
    }
}

public Properties getChildrenAsProperties() {
    //创建一个Properties对象
    Properties properties = new Properties();
    // 获取并遍历子节点
    for (XNode child : getChildren()) {
        // 获取 property 节点的 name 和 value 属性
        String name = child.getStringAttribute("name");
        String value = child.getStringAttribute("value");
        if (name != null && value != null) {
            // 设置属性到属性对象中
            properties.setProperty(name, value);
        }
    }
    return properties;
}

// -☆- XNode
public List<XNode> getChildren() {
    List<XNode> children = new ArrayList<XNode>();
    // 获取子节点列表
    NodeList nodeList = node.getChildNodes();
    if (nodeList != null) {
        for (int i = 0, n = nodeList.getLength(); i < n; i++) {
            Node node = nodeList.item(i);
            if (node.getNodeType() == Node.ELEMENT_NODE) {
                children.add(new XNode(xpathParser, node, variables));
            }
        }
    }
    return children;
}
```

解析properties主要分三个步骤：

1. 解析 properties 节点的子节点，并将解析结果设置到 Properties 对象中。
2. 从文件系统或通过网络读取属性配置，这取决于 properties 节点的 resource 和 url 是否为空。
3. 将解析出的属性对象设置到 XPathParser 和 Configuration 对象中。

需要注意的是，propertiesElement 方法是先解析 properties 节点的子节点内容，后再从文件系统或者网络读取属性配置，并将所有的属性及属性值都放入到 defaults 属性对象中。这就会存在同名属性覆盖的问题，也就是从文件系统，或者网络上读取到的属性及属性值会覆盖掉 properties 子节点中同名的属性和及值。

## 解析 settings 配置



### settings 节点的解析过程

下面先来看一个settings比较简单的配置，如下：

```
<settings>
    <setting name="cacheEnabled" value="true"/>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="autoMappingBehavior" value="PARTIAL"/>
</settings>
```

接着来看看settingsAsProperties

```
private Properties settingsAsProperties(XNode context) {
    if (context == null) {
        return new Properties();
    }
    // 获取 settings 子节点中的内容，解析成Properties，getChildrenAsProperties 方法前面已分析过
    Properties props = context.getChildrenAsProperties();

    // 创建 Configuration 类的“元信息”对象
    MetaClass metaConfig = MetaClass.forClass(Configuration.class, localReflectorFactory);
    for (Object key : props.keySet()) {
        // 检测 Configuration 中是否存在相关属性，不存在则抛出异常
        if (!metaConfig.hasSetter(String.valueOf(key))) {
            throw new BuilderException("The setting " + key + " is not known.  Make sure you spelled it correctly (case sensitive).");
        }
    }
    return props;
}
```



### 设置 settings 配置到 Configuration 中

接着我们看看将 settings 配置设置到 Configuration 对象中的过程。如下：

```
private void settingsElement(Properties props) throws Exception {
    // 设置 autoMappingBehavior 属性，默认值为 PARTIAL
    configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
    configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
    // 设置 cacheEnabled 属性，默认值为 true
    configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));

    // 省略部分代码

    // 解析默认的枚举处理器
    Class<? extends TypeHandler> typeHandler = (Class<? extends TypeHandler>)resolveClass(props.getProperty("defaultEnumTypeHandler"));
    // 设置默认枚举处理器
    configuration.setDefaultEnumTypeHandler(typeHandler);
    configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
    configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));
    
    // 省略部分代码
}
```

上面代码处理调用 Configuration 的 setter 方法

## 解析 typeAliases 配置

在 MyBatis 中，可以为我们自己写的有些类定义一个别名。这样在使用的时候，我们只需要输入别名即可，无需再把全限定的类名写出来。在 MyBatis 中，我们有两种方式进行别名配置。第一种是仅配置包名，让 MyBatis 去扫描包中的类型，并根据类型得到相应的别名

```
<typeAliases>
    <package name="com.mybatis.model"/>
</typeAliases>
```

第二种方式是通过手动的方式，明确为某个类型配置别名。这种方式的配置如下：

```
<typeAliases>
    <typeAlias alias="employe" type="com.mybatis.model.Employe" />
    <typeAlias type="com.mybatis.model.User" />
</typeAliases>
```

下面我们来看一下两种不同的别名配置是怎样解析的。代码如下：

**XMLConfigBuilder**

```
private void typeAliasesElement(XNode parent) {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            // 从指定的包中解析别名和类型的映射
            if ("package".equals(child.getName())) {
                String typeAliasPackage = child.getStringAttribute("name");
                configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
                
            // 从 typeAlias 节点中解析别名和类型的映射
            } else {
                // 获取 alias 和 type 属性值，alias 不是必填项，可为空
                String alias = child.getStringAttribute("alias");
                String type = child.getStringAttribute("type");
                try {
                    // 加载 type 对应的类型
                    Class<?> clazz = Resources.classForName(type);

                    // 注册别名到类型的映射
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

我们看到通过包扫描和手动注册时通过子节点名称是否package来判断的



### 从 typeAlias 节点中解析并注册别名

在别名的配置中，`type`属性是必须要配置的，而`alias`属性则不是必须的。

```
private final Map<String, Class<?>> TYPE_ALIASES = new HashMap<String, Class<?>>();

public void registerAlias(Class<?> type) {
    // 获取全路径类名的简称
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
        // 从注解中取出别名
        alias = aliasAnnotation.value();
    }
    // 调用重载方法注册别名和类型映射
    registerAlias(alias, type);
}

public void registerAlias(String alias, Class<?> value) {
    if (alias == null) {
        throw new TypeException("The parameter alias cannot be null");
    }
    // 将别名转成小写
    String key = alias.toLowerCase(Locale.ENGLISH);
    /*
     * 如果 TYPE_ALIASES 中存在了某个类型映射，这里判断当前类型与映射中的类型是否一致，
     * 不一致则抛出异常，不允许一个别名对应两种类型
     */
    if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null && !TYPE_ALIASES.get(key).equals(value)) {
        throw new TypeException(
            "The alias '" + alias + "' is already mapped to the value '" + TYPE_ALIASES.get(key).getName() + "'.");
    }
    // 缓存别名到类型映射
    TYPE_ALIASES.put(key, value);
}
```

若用户为明确配置 alias 属性，MyBatis 会使用类名的小写形式作为别名。比如，全限定类名com.mybatis.model.User的别名为user。若类中有@Alias注解，则从注解中取值作为别名。



### 从指定的包中解析并注册别名

```
public void registerAliases(String packageName) {
    registerAliases(packageName, Object.class);
}

public void registerAliases(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    /*
     * 查找包下的父类为 Object.class 的类。
     * 查找完成后，查找结果将会被缓存到resolverUtil的内部集合中。
     */ 
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    // 获取查找结果
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for (Class<?> type : typeSet) {
        // 忽略匿名类，接口，内部类
        if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
            // 为类型注册别名 
            registerAlias(type);
        }
    }
}
```

主要分为两个步骤：

1. 查找指定包下的所有类
2. 遍历查找到的类型集合，为每个类型注册别名

我们看看查找指定包下的所有类

```
private Set<Class<? extends T>> matches = new HashSet();

public ResolverUtil<T> find(ResolverUtil.Test test, String packageName) {
    //将包名转换成文件路径
    String path = this.getPackagePath(packageName);

    try {
        //通过 VFS（虚拟文件系统）获取指定包下的所有文件的路径名，比如com/mybatis/model/Employe.class
        List<String> children = VFS.getInstance().list(path);
        Iterator i$ = children.iterator();

        while(i$.hasNext()) {
            String child = (String)i$.next();
            //以.class结尾的文件就加入到Set集合中
            if (child.endsWith(".class")) {
                this.addIfMatching(test, child);
            }
        }
    } catch (IOException var7) {
        log.error("Could not read package: " + packageName, var7);
    }

    return this;
}

protected String getPackagePath(String packageName) {
    //将包名转换成文件路径
    return packageName == null ? null : packageName.replace('.', '/');
}

protected void addIfMatching(ResolverUtil.Test test, String fqn) {
    try {
        //将路径名转成全限定的类名，通过类加载器加载类名，比如com.mybatis.model.Employe.class
        String externalName = fqn.substring(0, fqn.indexOf(46)).replace('/', '.');
        ClassLoader loader = this.getClassLoader();
        if (log.isDebugEnabled()) {
            log.debug("Checking to see if class " + externalName + " matches criteria [" + test + "]");
        }

        Class<?> type = loader.loadClass(externalName);
        if (test.matches(type)) {
            //加入到matches集合中
            this.matches.add(type);
        }
    } catch (Throwable var6) {
        log.warn("Could not examine class '" + fqn + "'" + " due to a " + var6.getClass().getName() + " with message: " + var6.getMessage());
    }

}
```

主要有以下几步：

1. 通过 VFS（虚拟文件系统）获取指定包下的所有文件的路径名，比如 **com/mybatis/model/Employe.class**
2. 筛选以`.class`结尾的文件名
3. 将路径名转成全限定的类名，通过类加载器加载类名
4. 对类型进行匹配，若符合匹配规则，则将其放入内部集合中

这里我们要注意，在前面我们分析Configuration对象的创建时，就已经默认注册了很多别名，可以回到文章开头看看。

## 解析 plugins 配置

插件是 MyBatis 提供的一个拓展机制，通过插件机制我们可在 SQL 执行过程中的某些点上做一些自定义操作。比喻分页插件，在SQL执行之前动态拼接语句，我们后面会单独来讲插件机制，先来了解插件的配置。如下：

```
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="helperDialect" value="mysql"/>
    </plugin>
</plugins>
```

解析过程分析如下：

```
private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            String interceptor = child.getStringAttribute("interceptor");
            // 获取配置信息
            Properties properties = child.getChildrenAsProperties();
            // 解析拦截器的类型，并创建拦截器
            Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
            // 设置属性
            interceptorInstance.setProperties(properties);
            // 添加拦截器到 Configuration 中
            configuration.addInterceptor(interceptorInstance);
        }
    }
}
```

首先是获取配置，然后再解析拦截器类型，并实例化拦截器。最后向拦截器中设置属性，并将拦截器添加到 Configuration 中。

```
private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            String interceptor = child.getStringAttribute("interceptor");
            // 获取配置信息
            Properties properties = child.getChildrenAsProperties();
            // 解析拦截器的类型，并实例化拦截器
            Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
            // 设置属性
            interceptorInstance.setProperties(properties);
            // 添加拦截器到 Configuration 中
            configuration.addInterceptor(interceptorInstance);
        }
    }
}

public void addInterceptor(Interceptor interceptor) {
    //添加到Configuration的interceptorChain属性中
    this.interceptorChain.addInterceptor(interceptor);
}
```

我们来看看InterceptorChain

```
public class InterceptorChain {
    private final List<Interceptor> interceptors = new ArrayList();

    public InterceptorChain() {
    }

    public Object pluginAll(Object target) {
        Interceptor interceptor;
        for(Iterator i$ = this.interceptors.iterator(); i$.hasNext(); target = interceptor.plugin(target)) {
            interceptor = (Interceptor)i$.next();
        }

        return target;
    }

    public void addInterceptor(Interceptor interceptor) {
        this.interceptors.add(interceptor);
    }

    public List<Interceptor> getInterceptors() {
        return Collections.unmodifiableList(this.interceptors);
    }
}
```

实际上是一个 interceptors 集合，关于插件的原理我们后面再讲。

## 解析 environments 配置

在 MyBatis 中，事务管理器和数据源是配置在 environments 中的。它们的配置大致如下：

```
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </dataSource>
    </environment>
</environments>
```

我们来看看environmentsElement方法

```
private void environmentsElement(XNode context) throws Exception {
    if (context != null) {
        if (environment == null) {
            // 获取 default 属性
            environment = context.getStringAttribute("default");
        }
        for (XNode child : context.getChildren()) {
            // 获取 id 属性
            String id = child.getStringAttribute("id");
            /*
             * 检测当前 environment 节点的 id 与其父节点 environments 的属性 default 
             * 内容是否一致，一致则返回 true，否则返回 false
             * 将其default属性值与子元素environment的id属性值相等的子元素设置为当前使用的Environment对象
             */
            if (isSpecifiedEnvironment(id)) {
                // 将environment中的transactionManager标签转换为TransactionFactory对象
                TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                // 将environment中的dataSource标签转换为DataSourceFactory对象
                DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                // 创建 DataSource 对象
                DataSource dataSource = dsFactory.getDataSource();
                Environment.Builder environmentBuilder = new Environment.Builder(id)
                    .transactionFactory(txFactory)
                    .dataSource(dataSource);
                // 构建 Environment 对象，并设置到 configuration 中
                configuration.setEnvironment(environmentBuilder.build());
            }
        }
    }
}
```

看看TransactionFactory和 DataSourceFactory的获取

```
private TransactionFactory transactionManagerElement(XNode context) throws Exception {
    if (context != null) {
        String type = context.getStringAttribute("type");
        Properties props = context.getChildrenAsProperties();
        //通过别名获取Class,并实例化
        TransactionFactory factory = (TransactionFactory)this.resolveClass(type).newInstance();
        factory.setProperties(props);
        return factory;
    } else {
        throw new BuilderException("Environment declaration requires a TransactionFactory.");
    }
}

private DataSourceFactory dataSourceElement(XNode context) throws Exception {
    if (context != null) {
        String type = context.getStringAttribute("type");
        //通过别名获取Class,并实例化
        Properties props = context.getChildrenAsProperties();
        DataSourceFactory factory = (DataSourceFactory)this.resolveClass(type).newInstance();
        factory.setProperties(props);
        return factory;
    } else {
        throw new BuilderException("Environment declaration requires a DataSourceFactory.");
    }
}
```

**<transactionManager type="JDBC"/>**中type有"JDBC"、"MANAGED"这两种配置，而我们前面Configuration中默认注册的别名中有对应的JdbcTransactionFactory.class、ManagedTransactionFactory.class这两个TransactionFactory

**<dataSource type="POOLED">**中type有"JNDI"、"POOLED"、"UNPOOLED"这三种配置，默认注册的别名中有对应的JndiDataSourceFactory.class、PooledDataSourceFactory.class、UnpooledDataSourceFactory.class这三个DataSourceFactory

而我们的environment配置中**transactionManager type="JDBC"和\**dataSource type="POOLED"，则生成的\*\*transactionManager为JdbcTransactionFactory，DataSourceFactory为PooledDataSourceFactory\*\**\***

我们来看看PooledDataSourceFactory和UnpooledDataSourceFactory

```
public class UnpooledDataSourceFactory implements DataSourceFactory {
    private static final String DRIVER_PROPERTY_PREFIX = "driver.";
    private static final int DRIVER_PROPERTY_PREFIX_LENGTH = "driver.".length();
    //创建UnpooledDataSource实例
    protected DataSource dataSource = new UnpooledDataSource();
    
    public DataSource getDataSource() {
        return this.dataSource;
    }
    //略
}
 
//继承UnpooledDataSourceFactory
public class PooledDataSourceFactory extends UnpooledDataSourceFactory {
    public PooledDataSourceFactory() {
        //创建PooledDataSource实例
        this.dataSource = new PooledDataSource();
    }
}
```

我们发现 UnpooledDataSourceFactory 创建的dataSource是 UnpooledDataSource，PooledDataSourceFactory创建的 dataSource是PooledDataSource

## 解析 mappers 配置

mapperElement方法会将mapper标签内的元素转换成MapperProxyFactory产生的代理类，和与mapper.xml文件的绑定，我们下一篇文章会详解介绍这个方法

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

## 创建DefaultSqlSessionFactory

到此为止XMLConfigBuilder的parse方法中的重要步骤都过了一遍了，然后返回的就是一个完整的Configuration对象了，最后通过SqlSessionFactoryBuilder的build的重载方法创建了一个SqlSessionFactory实例DefaultSqlSessionFactory，我们来看看

```
public SqlSessionFactory build(Configuration config) {
    //创建DefaultSqlSessionFactory实例
    return new DefaultSqlSessionFactory(config);
}

public class DefaultSqlSessionFactory implements SqlSessionFactory {
    private final Configuration configuration;

    //只是将configuration设置为其属性
    public DefaultSqlSessionFactory(Configuration configuration) {
        this.configuration = configuration;
    }
    
    //略
}
```

# [Mybaits 源码解析 （三）----- Mapper映射的解析过程](https://www.cnblogs.com/java-chen-hao/p/11743442.html)

**正文**

上一篇我们讲解到mapperElement方法用来解析mapper，我们这篇文章具体来看看mapper.xml的解析过程

## mappers配置方式

mappers 标签下有许多 mapper 标签，每一个 mapper 标签中配置的都是一个独立的映射配置文件的路径，配置方式有以下几种。



### 接口信息进行配置



```
<mappers>
    <mapper class="org.mybatis.mappers.UserMapper"/>
    <mapper class="org.mybatis.mappers.ProductMapper"/>
    <mapper class="org.mybatis.mappers.ManagerMapper"/>
</mappers>
```

**注意：**这种方式必须保证接口名（例如UserMapper）和xml名（UserMapper.xml）相同，还必须在同一个包中。因为是通过获取mapper中的class属性，拼接上.xml来读取UserMapper.xml，如果xml文件名不同或者不在同一个包中是无法读取到xml的。



### 相对路径进行配置



```
<mappers>
    <mapper resource="org/mybatis/mappers/UserMapper.xml"/>
    <mapper resource="org/mybatis/mappers/ProductMapper.xml"/>
    <mapper resource="org/mybatis/mappers/ManagerMapper.xml"/>
</mappers>
```



**注意：**这种方式不用保证同接口同包同名。但是要保证xml中的namespase和对应的接口名相同。



### 绝对路径进行配置



```
<mappers>
    <mapper url="file:///var/mappers/UserMapper.xml"/>
    <mapper url="file:///var/mappers/ProductMapper.xml"/>
    <mapper url="file:///var/mappers/ManagerMapper.xml"/>
</mappers>
```



### 接口所在包进行配置

```
<mappers>
    <package name="org.mybatis.mappers"/>
</mappers>
```

这种方式和第一种方式要求一致，保证接口名（例如UserMapper）和xml名（UserMapper.xml）相同，还必须在同一个包中。

**注意：**以上所有的配置都要保证xml中的namespase和对应的接口名相同。

我们以packae属性为例详细分析一下:

## mappers解析入口方法

接上一篇文章最后部分，我们来看看mapperElement方法：

```
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            //包扫描的形式
            if ("package".equals(child.getName())) {
                // 获取 <package> 节点中的 name 属性
                String mapperPackage = child.getStringAttribute("name");
                // 从指定包中查找 所有的 mapper 接口，并根据 mapper 接口解析映射配置
                configuration.addMappers(mapperPackage);
            } else {
                // 获取 resource/url/class 等属性
                String resource = child.getStringAttribute("resource");
                String url = child.getStringAttribute("url");
                String mapperClass = child.getStringAttribute("class");

                // resource 不为空，且其他两者为空，则从指定路径中加载配置
                if (resource != null && url == null && mapperClass == null) {
                    ErrorContext.instance().resource(resource);
                    InputStream inputStream = Resources.getResourceAsStream(resource);
                    XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                    // 解析映射文件
                    mapperParser.parse();
                // url 不为空，且其他两者为空，则通过 url 加载配置
                } else if (resource == null && url != null && mapperClass == null) {
                    ErrorContext.instance().resource(url);
                    InputStream inputStream = Resources.getUrlAsStream(url);
                    XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                    // 解析映射文件
                    mapperParser.parse();
                // mapperClass 不为空，且其他两者为空，则通过 mapperClass 解析映射配置
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



在 MyBatis 中，共有四种加载映射文件或信息的方式。第一种是从文件系统中加载映射文件；第二种是通过 URL 的方式加载和解析映射文件；第三种是通过 mapper 接口加载映射信息，映射信息可以配置在注解中，也可以配置在映射文件中。最后一种是通过包扫描的方式获取到某个包下的所有类，并使用第三种方式为每个类解析映射信息。

我们先看下以packae扫描的形式，看下configuration.addMappers(mapperPackage)方法

```
public void addMappers(String packageName) {
    mapperRegistry.addMappers(packageName);
}
```

我们看一下MapperRegistry的addMappers方法:



```
 1 public void addMappers(String packageName) {
 2     //传入包名和Object.class类型
 3     this.addMappers(packageName, Object.class);
 4 }
 5 
 6 public void addMappers(String packageName, Class<?> superType) {
 7     ResolverUtil<Class<?>> resolverUtil = new ResolverUtil();
 8     /*
 9      * 查找包下的父类为 Object.class 的类。
10      * 查找完成后，查找结果将会被缓存到resolverUtil的内部集合中。上一篇文章我们已经看过这部分的源码，不再累述了
11      */ 
12     resolverUtil.find(new IsA(superType), packageName);
13     // 获取查找结果
14     Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
15     Iterator i$ = mapperSet.iterator();
16 
17     while(i$.hasNext()) {
18         Class<?> mapperClass = (Class)i$.next();
19         //我们具体看这个方法
20         this.addMapper(mapperClass);
21     }
22 
23 }
```



其实就是通过 VFS（虚拟文件系统）获取指定包下的所有文件的Class，也就是所有的Mapper接口，然后遍历每个Mapper接口进行解析，接下来就和第一种配置方式（接口信息进行配置）一样的流程了，接下来我们来看看 基于 XML 的映射文件的解析过程，可以看到先创建一个XMLMapperBuilder，再调用其parse()方法：



```
 1 public void parse() {
 2     // 检测映射文件是否已经被解析过
 3     if (!configuration.isResourceLoaded(resource)) {
 4         // 解析 mapper 节点
 5         configurationElement(parser.evalNode("/mapper"));
 6         // 添加资源路径到“已解析资源集合”中
 7         configuration.addLoadedResource(resource);
 8         // 通过命名空间绑定 Mapper 接口
 9         bindMapperForNamespace();
10     }
11 
12     parsePendingResultMaps();
13     parsePendingCacheRefs();
14     parsePendingStatements();
15 }
```



我们重点关注第5行和第9行的逻辑，也就是**configurationElement和****bindMapperForNamespace方法**



## 解析映射文件

在 MyBatis 映射文件中，可以配置多种节点。比如 <cache>，<resultMap>，<sql> 以及 <select | insert | update | delete> 等。下面我们来看一个映射文件配置示例。



```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper.EmployeeMapper">
    <cache/>
    
    <resultMap id="baseMap" type="entity.Employee">
        <result property="id" column="id" jdbcType="INTEGER"></result>
        <result property="name" column="name" jdbcType="VARCHAR"></result>
    </resultMap>
    
    <sql id="table">
        employee
    </sql>
    
    <select id="getAll" resultMap="baseMap">
        select * from  <include refid="table"/>  WHERE id = #{id}
    </select>
    
    <!-- <insert|update|delete/> -->
</mapper>
```



接着来看看configurationElement解析mapper.xml中的内容。



```
 1 private void configurationElement(XNode context) {
 2     try {
 3         // 获取 mapper 命名空间，如 mapper.EmployeeMapper
 4         String namespace = context.getStringAttribute("namespace");
 5         if (namespace == null || namespace.equals("")) {
 6             throw new BuilderException("Mapper's namespace cannot be empty");
 7         }
 8 
 9         // 设置命名空间到 builderAssistant 中
10         builderAssistant.setCurrentNamespace(namespace);
11 
12         // 解析 <cache-ref> 节点
13         cacheRefElement(context.evalNode("cache-ref"));
14 
15         // 解析 <cache> 节点
16         cacheElement(context.evalNode("cache"));
17 
18         // 已废弃配置，这里不做分析
19         parameterMapElement(context.evalNodes("/mapper/parameterMap"));
20 
21         // 解析 <resultMap> 节点
22         resultMapElements(context.evalNodes("/mapper/resultMap"));
23 
24         // 解析 <sql> 节点
25         sqlElement(context.evalNodes("/mapper/sql"));
26 
27         // 解析 <select>、<insert>、<update>、<delete> 节点
28         buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
29     } catch (Exception e) {
30         throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
31     }
32 }
```



接下来我们就对其中关键的方法进行详细分析



### 解析 cache 节点

MyBatis 提供了一、二级缓存，其中一级缓存是 SqlSession 级别的，默认为开启状态。二级缓存配置在映射文件中，使用者需要显示配置才能开启。如下：

```
<cache/>
```

也可以使用第三方缓存

```
<cache type="org.mybatis.caches.redis.RedisCache"/>
```

其中有一些属性可以选择

```
<cache eviction="LRU"  flushInterval="60000"  size="512" readOnly="true"/>
```

1. 根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”
2. 缓存的容量为 512 个对象引用
3. 缓存每隔60秒刷新一次
4. 缓存返回的对象是写安全的，即在外部修改对象不会影响到缓存内部存储对象

下面我们来分析一下缓存配置的解析逻辑，如下：

```
private void cacheElement(XNode context) throws Exception {
    if (context != null) {
        // 获取type属性，如果type没有指定就用默认的PERPETUAL(早已经注册过的别名的PerpetualCache)
        String type = context.getStringAttribute("type", "PERPETUAL");
        // 根据type从早已经注册的别名中获取对应的Class,PERPETUAL对应的Class是PerpetualCache.class
        // 如果我们写了type属性，如type="org.mybatis.caches.redis.RedisCache"，这里将会得到RedisCache.class
        Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
        //获取淘汰方式，默认为LRU(早已经注册过的别名的LruCache)，最近最少使用到的先淘汰
        String eviction = context.getStringAttribute("eviction", "LRU");
        Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
        Long flushInterval = context.getLongAttribute("flushInterval");
        Integer size = context.getIntAttribute("size");
        boolean readWrite = !context.getBooleanAttribute("readOnly", false);
        boolean blocking = context.getBooleanAttribute("blocking", false);

        // 获取子节点配置
        Properties props = context.getChildrenAsProperties();

        // 构建缓存对象
        builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
    }
}

public <T> Class<T> resolveAlias(String string) {
    try {
        if (string == null) {
            return null;
        } else {
            // 转换成小写
            String key = string.toLowerCase(Locale.ENGLISH);
            Class value;
            // 如果没有设置type属性，则这里传过来的是PERPETUAL，能从别名缓存中获取到PerpetualCache.class
            if (this.TYPE_ALIASES.containsKey(key)) {
                value = (Class)this.TYPE_ALIASES.get(key);
            } else {
                //如果是设置了自定义的type，则在别名缓存中是获取不到的，直接通过类加载，加载自定义的type，如RedisCache.class
                value = Resources.classForName(string);
            }

            return value;
        }
    } catch (ClassNotFoundException var4) {
        throw new TypeException("Could not resolve type alias '" + string + "'.  Cause: " + var4, var4);
    }
}
```



缓存的构建封装在 BuilderAssistant 类的 useNewCache 方法中,我们来看看

```
public Cache useNewCache(Class<? extends Cache> typeClass,
    Class<? extends Cache> evictionClass,Long flushInterval,
    Integer size,boolean readWrite,boolean blocking,Properties props) {

    // 使用建造模式构建缓存实例
    Cache cache = new CacheBuilder(currentNamespace)
        .implementation(valueOrDefault(typeClass, PerpetualCache.class))
        .addDecorator(valueOrDefault(evictionClass, LruCache.class))
        .clearInterval(flushInterval)
        .size(size)
        .readWrite(readWrite)
        .blocking(blocking)
        .properties(props)
        .build();

    // 添加缓存到 Configuration 对象中
    configuration.addCache(cache);

    // 设置 currentCache 属性，即当前使用的缓存
    currentCache = cache;
    return cache;
}
```



上面使用了建造模式构建 Cache 实例，我们跟下去看看。



```
public Cache build() {
    // 设置默认的缓存类型（PerpetualCache）和缓存装饰器（LruCache）
    setDefaultImplementations();

    // 通过反射创建缓存
    Cache cache = newBaseCacheInstance(implementation, id);
    setCacheProperties(cache);
    // 仅对内置缓存 PerpetualCache 应用装饰器
    if (PerpetualCache.class.equals(cache.getClass())) {
        // 遍历装饰器集合，应用装饰器
        for (Class<? extends Cache> decorator : decorators) {
            // 通过反射创建装饰器实例
            cache = newCacheDecoratorInstance(decorator, cache);
            // 设置属性值到缓存实例中
            setCacheProperties(cache);
        }
        // 应用标准的装饰器，比如 LoggingCache、SynchronizedCache
        cache = setStandardDecorators(cache);
    } else if (!LoggingCache.class.isAssignableFrom(cache.getClass())) {
        // 应用具有日志功能的缓存装饰器
        cache = new LoggingCache(cache);
    }
    return cache;
}

private void setDefaultImplementations() {
    if (this.implementation == null) {
        //设置默认缓存类型为PerpetualCache
        this.implementation = PerpetualCache.class;
        if (this.decorators.isEmpty()) {
            this.decorators.add(LruCache.class);
        }
    }
}

private Cache newBaseCacheInstance(Class<? extends Cache> cacheClass, String id) {
    //获取构造器
    Constructor cacheConstructor = this.getBaseCacheConstructor(cacheClass);

    try {
        //通过构造器实例化Cache
        return (Cache)cacheConstructor.newInstance(id);
    } catch (Exception var5) {
        throw new CacheException("Could not instantiate cache implementation (" + cacheClass + "). Cause: " + var5, var5);
    }
}
```



如上就创建好了一个Cache的实例，然后把它添加到Configuration中，并且设置到currentCache属性中，这个属性后面还要使用，也就是Cache实例后面还要使用，我们后面再看。



### 解析 resultMap 节点

resultMap 主要用于映射结果。通过 resultMap 和自动映射，可以让 MyBatis 帮助我们完成 ResultSet → Object 的映射。下面开始分析 resultMap 配置的解析过程。



```
private void resultMapElements(List<XNode> list) throws Exception {
    // 遍历 <resultMap> 节点列表
    for (XNode resultMapNode : list) {
        try {
            // 解析 resultMap 节点
            resultMapElement(resultMapNode);
        } catch (IncompleteElementException e) {
        }
    }
}

private ResultMap resultMapElement(XNode resultMapNode) throws Exception {
    return resultMapElement(resultMapNode, Collections.<ResultMapping>emptyList());
}

private ResultMap resultMapElement(XNode resultMapNode, List<ResultMapping> additionalResultMappings) throws Exception {
    ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());

    // 获取 id 和 type 属性
    String id = resultMapNode.getStringAttribute("id", resultMapNode.getValueBasedIdentifier());
    String type = resultMapNode.getStringAttribute("type",
        resultMapNode.getStringAttribute("ofType",
            resultMapNode.getStringAttribute("resultType",
                resultMapNode.getStringAttribute("javaType"))));
    // 获取 extends 和 autoMapping
    String extend = resultMapNode.getStringAttribute("extends");
    Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");

    // 获取 type 属性对应的类型
    Class<?> typeClass = resolveClass(type);
    Discriminator discriminator = null;
    //创建ResultMapping集合，对应resultMap子节点的id和result节点
    List<ResultMapping> resultMappings = new ArrayList<ResultMapping>();
    resultMappings.addAll(additionalResultMappings);

    // 获取并遍历 <resultMap> 的子节点列表
    List<XNode> resultChildren = resultMapNode.getChildren();
    for (XNode resultChild : resultChildren) {
        if ("constructor".equals(resultChild.getName())) {
            processConstructorElement(resultChild, typeClass, resultMappings);
        } else if ("discriminator".equals(resultChild.getName())) {
            discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
        } else {

            List<ResultFlag> flags = new ArrayList<ResultFlag>();
            if ("id".equals(resultChild.getName())) {
                // 添加 ID 到 flags 集合中
                flags.add(ResultFlag.ID);
            }
            // 解析 id 和 result 节点，将id或result节点生成相应的 ResultMapping，将ResultMapping添加到resultMappings集合中
            resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));
        }
    }
    //创建ResultMapResolver对象
    ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend,
        discriminator, resultMappings, autoMapping);
    try {
        // 根据前面获取到的信息构建 ResultMap 对象
        return resultMapResolver.resolve();
    } catch (IncompleteElementException e) {
        configuration.addIncompleteResultMap(resultMapResolver);
        throw e;
    }
}
```



**解析 id 和 result 节点**

在 <resultMap> 节点中，子节点 <id> 和 <result> 都是常规配置，比较常见。我们来看看解析过程



```
private ResultMapping buildResultMappingFromContext(XNode context, Class<?> resultType, List<ResultFlag> flags) throws Exception {
    String property;
    // 根据节点类型获取 name 或 property 属性
    if (flags.contains(ResultFlag.CONSTRUCTOR)) {
        property = context.getStringAttribute("name");
    } else {
        property = context.getStringAttribute("property");
    }

    // 获取其他各种属性
    String column = context.getStringAttribute("column");
    String javaType = context.getStringAttribute("javaType");
    String jdbcType = context.getStringAttribute("jdbcType");
    String nestedSelect = context.getStringAttribute("select");
    
    /*
     * 解析 resultMap 属性，该属性出现在 <association> 和 <collection> 节点中。
     * 若这两个节点不包含 resultMap 属性，则调用 processNestedResultMappings 方法,递归调用resultMapElement解析<association> 和 <collection>的嵌套节点，生成resultMap，并返回resultMap.getId();
     * 如果包含resultMap属性，则直接获取其属性值，这个属性值对应一个resultMap节点
     */
    String nestedResultMap = context.getStringAttribute("resultMap", processNestedResultMappings(context, Collections.<ResultMapping>emptyList()));
    
    String notNullColumn = context.getStringAttribute("notNullColumn");
    String columnPrefix = context.getStringAttribute("columnPrefix");
    String typeHandler = context.getStringAttribute("typeHandler");
    String resultSet = context.getStringAttribute("resultSet");
    String foreignColumn = context.getStringAttribute("foreignColumn");
    boolean lazy = "lazy".equals(context.getStringAttribute("fetchType", configuration.isLazyLoadingEnabled() ? "lazy" : "eager"));

    Class<?> javaTypeClass = resolveClass(javaType);
    Class<? extends TypeHandler<?>> typeHandlerClass = (Class<? extends TypeHandler<?>>) resolveClass(typeHandler);
    JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);

    // 构建 ResultMapping 对象
    return builderAssistant.buildResultMapping(resultType, property, column, javaTypeClass, jdbcTypeEnum, nestedSelect,
        nestedResultMap, notNullColumn, columnPrefix, typeHandlerClass, flags, resultSet, foreignColumn, lazy);
}
```



看processNestedResultMappings解析<association> 和 <collection> 节点中的子节点，并返回ResultMap.id



```
private String processNestedResultMappings(XNode context, List<ResultMapping> resultMappings) throws Exception {
    if (("association".equals(context.getName()) || "collection".equals(context.getName()) || "case".equals(context.getName())) && context.getStringAttribute("select") == null) {
        ResultMap resultMap = this.resultMapElement(context, resultMappings);
        return resultMap.getId();
    } else {
        return null;
    }
}
```



**只要此节点是**（**association或者****collection）并且select为空,就说明是嵌套查询，那如果select不为空呢？那说明是延迟加载此节点的信息，并不属于嵌套查询，但是有可能有多个\**association或者\*\*collection，有一个设置为延迟加载也就是select属性不为空，有一个没有设置延迟加载，那说明resultMap中有嵌套查询的ResultMapping，也有延迟加载的ResultMapping，这个在后面结果集映射时会用到。\*\**\***

下面以 <association> 节点为例，演示该节点的两种配置方式，分别如下：

第一种配置方式是通过 resultMap 属性引用其他的 <resultMap> 节点，配置如下：



```
<resultMap id="articleResult" type="Article">
    <id property="id" column="id"/>
    <result property="title" column="article_title"/>
    <!-- 引用 authorResult，此时为嵌套查询 -->
    <association property="article_author" column="article_author_id" javaType="Author" resultMap="authorResult"/>
    <!-- 引用 authorResult，此时为延迟查询 -->
    <association property="article_author" column="article_author_id" javaType="Author" select="authorResult"/>
</resultMap>

<resultMap id="authorResult" type="Author">
    <id property="id" column="author_id"/>
    <result property="name" column="author_name"/>
</resultMap>
```



第二种配置方式是采取 resultMap 嵌套的方式进行配置，如下：



```
<resultMap id="articleResult" type="Article">
    <id property="id" column="id"/>
    <result property="title" column="article_title"/>
    <!-- resultMap 嵌套 -->
    <association property="article_author" javaType="Author">
        <id property="id" column="author_id"/>
        <result property="name" column="author_name"/>
    </association>
</resultMap>
```



第二种配置，<association> 的子节点是一些结果映射配置，这些结果配置最终也会被解析成 ResultMap。

下面分析 ResultMapping 的构建过程。



```
public ResultMapping buildResultMapping(Class<?> resultType, String property, String column, Class<?> javaType,JdbcType jdbcType, 
    String nestedSelect, String nestedResultMap, String notNullColumn, String columnPrefix,Class<? extends TypeHandler<?>> typeHandler, 
    List<ResultFlag> flags, String resultSet, String foreignColumn, boolean lazy) {

    // resultType：即 <resultMap type="xxx"/> 中的 type 属性
    // property：即 <result property="xxx"/> 中的 property 属性
    Class<?> javaTypeClass = resolveResultJavaType(resultType, property, javaType);

    TypeHandler<?> typeHandlerInstance = resolveTypeHandler(javaTypeClass, typeHandler);

    List<ResultMapping> composites = parseCompositeColumnName(column);

    // 通过建造模式构建 ResultMapping
    return new ResultMapping.Builder(configuration, property, column, javaTypeClass)
        .jdbcType(jdbcType)
        .nestedQueryId(applyCurrentNamespace(nestedSelect, true))
        .nestedResultMapId(applyCurrentNamespace(nestedResultMap, true))
        .resultSet(resultSet)
        .typeHandler(typeHandlerInstance)
        .flags(flags == null ? new ArrayList<ResultFlag>() : flags)
        .composites(composites)
        .notNullColumns(parseMultipleColumnNames(notNullColumn))
        .columnPrefix(columnPrefix)
        .foreignColumn(foreignColumn)
        .lazy(lazy)
        .build();
}

private Class<?> resolveResultJavaType(Class<?> resultType, String property, Class<?> javaType) {
    if (javaType == null && property != null) {
        try {
            //获取ResultMap中的type属性的元类，如<resultMap id="user" type="java.model.User"/> 中User的元类
            MetaClass metaResultType = MetaClass.forClass(resultType, this.configuration.getReflectorFactory());
            //<result property="name" javaType="String"/>,如果result中没有设置javaType，则获取元类属性对那个的类型
            javaType = metaResultType.getSetterType(property);
        } catch (Exception var5) {
            ;
        }
    }

    if (javaType == null) {
        javaType = Object.class;
    }

    return javaType;
}
    
public ResultMapping build() {
    resultMapping.flags = Collections.unmodifiableList(resultMapping.flags);
    resultMapping.composites = Collections.unmodifiableList(resultMapping.composites);
    resolveTypeHandler();
    validate();
    return resultMapping;
}
```



我们来看看ResultMapping类



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

    ResultMapping() {
    }
    //略
}
```



我们看到ResultMapping中有属性**nestedResultMapId表示嵌套查询和****nestedQueryId表示延迟查询**

ResultMapping就是和ResultMap中子节点id和result对应

```
<id column="wi_id" jdbcType="INTEGER"  property="id" />
<result column="warrant_no" jdbcType="String"  jdbcType="CHAR" property="warrantNo" />
```

**ResultMap 对象构建**

前面的分析我们知道了<id>，<result> 等节点最终都被解析成了 ResultMapping。并且封装到了**resultMappings集合中，**紧接着要做的事情是构建 ResultMap，关键代码在**resultMapResolver.resolve()：**



```
public ResultMap resolve() {
    return assistant.addResultMap(this.id, this.type, this.extend, this.discriminator, this.resultMappings, this.autoMapping);
}

public ResultMap addResultMap(
    String id, Class<?> type, String extend, Discriminator discriminator,
    List<ResultMapping> resultMappings, Boolean autoMapping) {
    
    // 为 ResultMap 的 id 和 extend 属性值拼接命名空间
    id = applyCurrentNamespace(id, false);
    extend = applyCurrentNamespace(extend, true);

    if (extend != null) {
        if (!configuration.hasResultMap(extend)) {
            throw new IncompleteElementException("Could not find a parent resultmap with id '" + extend + "'");
        }
        ResultMap resultMap = configuration.getResultMap(extend);
        List<ResultMapping> extendedResultMappings = new ArrayList<ResultMapping>(resultMap.getResultMappings());
        extendedResultMappings.removeAll(resultMappings);
        
        boolean declaresConstructor = false;
        for (ResultMapping resultMapping : resultMappings) {
            if (resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR)) {
                declaresConstructor = true;
                break;
            }
        }
        
        if (declaresConstructor) {
            Iterator<ResultMapping> extendedResultMappingsIter = extendedResultMappings.iterator();
            while (extendedResultMappingsIter.hasNext()) {
                if (extendedResultMappingsIter.next().getFlags().contains(ResultFlag.CONSTRUCTOR)) {
                    extendedResultMappingsIter.remove();
                }
            }
        }
        resultMappings.addAll(extendedResultMappings);
    }

    // 构建 ResultMap
    ResultMap resultMap = new ResultMap.Builder(configuration, id, type, resultMappings, autoMapping)
        .discriminator(discriminator)
        .build();
    // 将创建好的ResultMap加入configuration中
    configuration.addResultMap(resultMap);
    return resultMap;
}
```



我们先看看**ResultMap**



```
public class ResultMap {
    private String id;
    private Class<?> type;
    private List<ResultMapping> resultMappings;
    //用于存储 <id> 节点对应的 ResultMapping 对象
    private List<ResultMapping> idResultMappings;
    private List<ResultMapping> constructorResultMappings;
    //用于存储 <id> 和 <result> 节点对应的 ResultMapping 对象
    private List<ResultMapping> propertyResultMappings;
    //用于存储 所有<id>、<result> 节点 column 属性
    private Set<String> mappedColumns;
    private Discriminator discriminator;
    private boolean hasNestedResultMaps;
    private boolean hasNestedQueries;
    private Boolean autoMapping;

    private ResultMap() {
    }
    //略
}
```



再来看看通过建造模式构建 ResultMap 实例



```
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
        /*
         * resultMapping.getNestedQueryId()不为空，表示当前resultMap是中有需要延迟查询的属性
         * resultMapping.getNestedResultMapId()不为空，表示当前resultMap是一个嵌套查询
         * 有可能当前ResultMapp既是一个嵌套查询，又存在延迟查询的属性
         */
        resultMap.hasNestedQueries = resultMap.hasNestedQueries || resultMapping.getNestedQueryId() != null;
        resultMap.hasNestedResultMaps =  resultMap.hasNestedResultMaps || (resultMapping.getNestedResultMapId() != null && resultMapping.getResultSet() == null);

        final String column = resultMapping.getColumn();
        if (column != null) {
            // 将 colum 转换成大写，并添加到 mappedColumns 集合中
            resultMap.mappedColumns.add(column.toUpperCase(Locale.ENGLISH));
        } else if (resultMapping.isCompositeResult()) {
            for (ResultMapping compositeResultMapping : resultMapping.getComposites()) {
                final String compositeColumn = compositeResultMapping.getColumn();
                if (compositeColumn != null) {
                    resultMap.mappedColumns.add(compositeColumn.toUpperCase(Locale.ENGLISH));
                }
            }
        }

        // 添加属性 property 到 mappedProperties 集合中
        final String property = resultMapping.getProperty();
        if (property != null) {
            resultMap.mappedProperties.add(property);
        }

        if (resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR)) {
            resultMap.constructorResultMappings.add(resultMapping);
            if (resultMapping.getProperty() != null) {
                constructorArgNames.add(resultMapping.getProperty());
            }
        } else {
            // 添加 resultMapping 到 propertyResultMappings 中
            resultMap.propertyResultMappings.add(resultMapping);
        }

        if (resultMapping.getFlags().contains(ResultFlag.ID)) {
            // 添加 resultMapping 到 idResultMappings 中
            resultMap.idResultMappings.add(resultMapping);
        }
    }
    if (resultMap.idResultMappings.isEmpty()) {
        resultMap.idResultMappings.addAll(resultMap.resultMappings);
    }
    if (!constructorArgNames.isEmpty()) {
        final List<String> actualArgNames = argNamesOfMatchingConstructor(constructorArgNames);
        if (actualArgNames == null) {
            throw new BuilderException("Error in result map '" + resultMap.id
                + "'. Failed to find a constructor in '"
                + resultMap.getType().getName() + "' by arg names " + constructorArgNames
                + ". There might be more info in debug log.");
        }
        Collections.sort(resultMap.constructorResultMappings, new Comparator<ResultMapping>() {
            @Override
            public int compare(ResultMapping o1, ResultMapping o2) {
                int paramIdx1 = actualArgNames.indexOf(o1.getProperty());
                int paramIdx2 = actualArgNames.indexOf(o2.getProperty());
                return paramIdx1 - paramIdx2;
            }
        });
    }

    // 将以下这些集合变为不可修改集合
    resultMap.resultMappings = Collections.unmodifiableList(resultMap.resultMappings);
    resultMap.idResultMappings = Collections.unmodifiableList(resultMap.idResultMappings);
    resultMap.constructorResultMappings = Collections.unmodifiableList(resultMap.constructorResultMappings);
    resultMap.propertyResultMappings = Collections.unmodifiableList(resultMap.propertyResultMappings);
    resultMap.mappedColumns = Collections.unmodifiableSet(resultMap.mappedColumns);
    return resultMap;
}
```



主要做的事情就是将 ResultMapping 实例及属性分别存储到不同的集合中。



### 解析 sql 节点

<sql> 节点用来定义一些可重用的 SQL 语句片段，比如表名，或表的列名等。在映射文件中，我们可以通过 <include> 节点引用 <sql> 节点定义的内容。



```
<sql id="table">
    user
</sql>

<select id="findOne" resultType="Article">
    SELECT * FROM <include refid="table"/> WHERE id = #{id}
</select>
```



下面分析一下 sql 节点的解析过程，如下：



```
private void sqlElement(List<XNode> list) throws Exception {
    if (configuration.getDatabaseId() != null) {
        // 调用 sqlElement 解析 <sql> 节点
        sqlElement(list, configuration.getDatabaseId());
    }

    // 再次调用 sqlElement，不同的是，这次调用，该方法的第二个参数为 null
    sqlElement(list, null);
}

private void sqlElement(List<XNode> list, String requiredDatabaseId) throws Exception {
    for (XNode context : list) {
        // 获取 id 和 databaseId 属性
        String databaseId = context.getStringAttribute("databaseId");
        String id = context.getStringAttribute("id");

        // id = currentNamespace + "." + id
        id = builderAssistant.applyCurrentNamespace(id, false);

        // 检测当前 databaseId 和 requiredDatabaseId 是否一致
        if (databaseIdMatchesCurrent(id, databaseId, requiredDatabaseId)) {
            // 将 <id, XNode> 键值对缓存到XMLMapperBuilder对象的 sqlFragments 属性中，以供后面的sql语句使用
            sqlFragments.put(id, context);
        }
    }
}
```





### 解析select|insert|update|delete节点

 <select>、<insert>、<update> 以及 <delete> 等节点统称为 SQL 语句节点，其解析过程在buildStatementFromContext方法中：




```
private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
        // 调用重载方法构建 Statement
        buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
}

private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
        // 创建 XMLStatementBuilder 建造类
        final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
        try {
            /*
             * 解析sql节点，将其封装到 Statement 对象中，并将解析结果存储到 configuration 的 mappedStatements 集合中
             */
            statementParser.parseStatementNode();
        } catch (IncompleteElementException e) {
            configuration.addIncompleteStatement(statementParser);
        }
    }
}
```



我们继续看 statementParser.parseStatementNode();



```
public void parseStatementNode() {
    // 获取 id 和 databaseId 属性
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");

    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
        return;
    }

    // 获取各种属性
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultType = context.getStringAttribute("resultType");
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    // 通过别名解析 resultType 对应的类型
    Class<?> resultTypeClass = resolveClass(resultType);
    String resultSetType = context.getStringAttribute("resultSetType");
    
    // 解析 Statement 类型，默认为 PREPARED
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    
    // 解析 ResultSetType
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);

    // 获取节点的名称，比如 <select> 节点名称为 select
    String nodeName = context.getNode().getNodeName();
    // 根据节点名称解析 SqlCommandType
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // 解析 <include> 节点
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    processSelectKeyNodes(id, parameterTypeClass, langDriver);

    // 解析 SQL 语句
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
            configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType)) ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

    /*
     * 构建 MappedStatement 对象，并将该对象存储到 Configuration 的 mappedStatements 集合中
     */
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered,
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
```

我们主要来分析下面几个重要的方法：

1. 解析 <include> 节点
2. 解析 SQL，获取 SqlSource
3. 构建 MappedStatement 实例

**解析 <include> 节点**

先来看一个include的例子



```
<mapper namespace="java.mybaits.dao.UserMapper">
    <sql id="table">
        user
    </sql>

    <select id="findOne" resultType="User">
        SELECT  * FROM  <include refid="table"/> WHERE id = #{id}
    </select>
</mapper>
```



<include> 节点的解析逻辑封装在 applyIncludes 中，该方法的代码如下：



```
public void applyIncludes(Node source) {
    Properties variablesContext = new Properties();
    Properties configurationVariables = configuration.getVariables();
    if (configurationVariables != null) {
        // 将 configurationVariables 中的数据添加到 variablesContext 中
        variablesContext.putAll(configurationVariables);
    }

    // 调用重载方法处理 <include> 节点
    applyIncludes(source, variablesContext, false);
}
```



继续看 applyIncludes 方法



```
private void applyIncludes(Node source, final Properties variablesContext, boolean included) {

    // 第一个条件分支
    if (source.getNodeName().equals("include")) {

        //获取 <sql> 节点。
        Node toInclude = findSqlFragment(getStringAttribute(source, "refid"), variablesContext);

        Properties toIncludeContext = getVariablesContext(source, variablesContext);

        applyIncludes(toInclude, toIncludeContext, true);

        if (toInclude.getOwnerDocument() != source.getOwnerDocument()) {
            toInclude = source.getOwnerDocument().importNode(toInclude, true);
        }
        // 将 <select>节点中的 <include> 节点替换为 <sql> 节点
        source.getParentNode().replaceChild(toInclude, source);
        while (toInclude.hasChildNodes()) {
            // 将 <sql> 中的内容插入到 <sql> 节点之前
            toInclude.getParentNode().insertBefore(toInclude.getFirstChild(), toInclude);
        }

        /*
         * 前面已经将 <sql> 节点的内容插入到 dom 中了，
         * 现在不需要 <sql> 节点了，这里将该节点从 dom 中移除
         */
        toInclude.getParentNode().removeChild(toInclude);

    // 第二个条件分支
    } else if (source.getNodeType() == Node.ELEMENT_NODE) {
        if (included && !variablesContext.isEmpty()) {
            NamedNodeMap attributes = source.getAttributes();
            for (int i = 0; i < attributes.getLength(); i++) {
                Node attr = attributes.item(i);
                // 将 source 节点属性中的占位符 ${} 替换成具体的属性值
                attr.setNodeValue(PropertyParser.parse(attr.getNodeValue(), variablesContext));
            }
        }
        
        NodeList children = source.getChildNodes();
        for (int i = 0; i < children.getLength(); i++) {
            // 递归调用
            applyIncludes(children.item(i), variablesContext, included);
        }
        
    // 第三个条件分支
    } else if (included && source.getNodeType() == Node.TEXT_NODE && !variablesContext.isEmpty()) {
        // 将文本（text）节点中的属性占位符 ${} 替换成具体的属性值
        source.setNodeValue(PropertyParser.parse(source.getNodeValue(), variablesContext));
    }
}
```



我们先来看一下 applyIncludes 方法第一次被调用时的状态，source为<select> 节点，节点类型：ELEMENT_NODE，此时会进入第二个分支，获取到获取 <select> 子节点列表，遍历子节点列表，将子节点作为参数，进行递归调用applyIncludes ，此时可获取到的子节点如下：

| 编号 | 子节点                   | 类型         | 描述     |
| ---- | ------------------------ | ------------ | -------- |
| 1    | SELECT * FROM            | TEXT_NODE    | 文本节点 |
| 2    | <include refid="table"/> | ELEMENT_NODE | 普通节点 |
| 3    | WHERE id = #{id}         | TEXT_NODE    | 文本节点 |

接下来要做的事情是遍历列表，然后将子节点作为参数进行递归调用。第一个子节点调用applyIncludes方法，source为 SELECT * FROM 节点，节点类型：TEXT_NODE，进入分支三，没有${}，不会替换，则节点一结束返回，什么都没有做。第二个节点调用applyIncludes方法，此时source为 <include refid="table"/>节点，节点类型：ELEMENT_NODE，进入分支一，通过refid找到 sql 节点，也就是toInclude节点，然后执行source.getParentNode().replaceChild(toInclude, source);，直接将<include refid="table"/>节点的父节点，也就是<select> 节点中的当前<include >节点替换成 <sql> 节点，然后调用toInclude.getParentNode().insertBefore(toInclude.getFirstChild(), toInclude);，将 <sql> 中的内容插入到 <sql> 节点之前，也就是将user插入到 <sql> 节点之前，现在不需要 <sql> 节点了，最后将该节点从 dom 中移除

**创建SqlSource**

创建SqlSource在createSqlSource方法中



```
public SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) {
    XMLScriptBuilder builder = new XMLScriptBuilder(configuration, script, parameterType);
    return builder.parseScriptNode();
}

// -☆- XMLScriptBuilder
public SqlSource parseScriptNode() {
    // 解析 SQL 语句节点
    MixedSqlNode rootSqlNode = parseDynamicTags(context);
    SqlSource sqlSource = null;
    // 根据 isDynamic 状态创建不同的 SqlSource
    if (isDynamic) {
        sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
    } else {
        sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);
    }
    return sqlSource;
}
```



继续跟进parseDynamicTags



```
/** 该方法用于初始化 nodeHandlerMap 集合，该集合后面会用到 */
private void initNodeHandlerMap() {
    nodeHandlerMap.put("trim", new TrimHandler());
    nodeHandlerMap.put("where", new WhereHandler());
    nodeHandlerMap.put("set", new SetHandler());
    nodeHandlerMap.put("foreach", new ForEachHandler());
    nodeHandlerMap.put("if", new IfHandler());
    nodeHandlerMap.put("choose", new ChooseHandler());
    nodeHandlerMap.put("when", new IfHandler());
    nodeHandlerMap.put("otherwise", new OtherwiseHandler());
    nodeHandlerMap.put("bind", new BindHandler());
}
    
protected MixedSqlNode parseDynamicTags(XNode node) {
    List<SqlNode> contents = new ArrayList<SqlNode>();
    NodeList children = node.getNode().getChildNodes();
    // 遍历子节点
    for (int i = 0; i < children.getLength(); i++) {
        XNode child = node.newXNode(children.item(i));
        //如果节点是TEXT_NODE类型
        if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE || child.getNode().getNodeType() == Node.TEXT_NODE) {
            // 获取文本内容
            String data = child.getStringBody("");
            TextSqlNode textSqlNode = new TextSqlNode(data);
            // 若文本中包含 ${} 占位符，会被认为是动态节点
            if (textSqlNode.isDynamic()) {
                contents.add(textSqlNode);
                // 设置 isDynamic 为 true
                isDynamic = true;
            } else {
                // 创建 StaticTextSqlNode
                contents.add(new StaticTextSqlNode(data));
            }

        // child 节点是 ELEMENT_NODE 类型，比如 <if>、<where> 等
        } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) {
            // 获取节点名称，比如 if、where、trim 等
            String nodeName = child.getNode().getNodeName();
            // 根据节点名称获取 NodeHandler，也就是上面注册的nodeHandlerMap
            NodeHandler handler = nodeHandlerMap.get(nodeName);
            if (handler == null) {
                throw new BuilderException("Unknown element <" + nodeName + "> in SQL statement.");
            }
            // 处理 child 节点，生成相应的 SqlNode
            handler.handleNode(child, contents);

            // 设置 isDynamic 为 true
            isDynamic = true;
        }
    }
    return new MixedSqlNode(contents);
}
```



对于if、trim、where等这些动态节点，是通过对应的handler来解析的，如下

```
handler.handleNode(child, contents);
```

该代码用于处理动态 SQL 节点，并生成相应的 SqlNode。下面来简单分析一下 WhereHandler 的代码。



```
/** 定义在 XMLScriptBuilder 中 */
private class WhereHandler implements NodeHandler {

    public WhereHandler() {
    }

    @Override
    public void handleNode(XNode nodeToHandle, List<SqlNode> targetContents) {
        // 调用 parseDynamicTags 解析 <where> 节点
        MixedSqlNode mixedSqlNode = parseDynamicTags(nodeToHandle);
        // 创建 WhereSqlNode
        WhereSqlNode where = new WhereSqlNode(configuration, mixedSqlNode);
        // 添加到 targetContents
        targetContents.add(where);
    }
}
```



我们已经将 XML 配置解析了 SqlSource，下面我们看看MappedStatement的构建。

**构建MappedStatement**

SQL 语句节点可以定义很多属性，这些属性和属性值最终存储在 MappedStatement 中。



```
public MappedStatement addMappedStatement(
    String id, SqlSource sqlSource, StatementType statementType, 
    SqlCommandType sqlCommandType,Integer fetchSize, Integer timeout, 
    String parameterMap, Class<?> parameterType,String resultMap, 
    Class<?> resultType, ResultSetType resultSetType, boolean flushCache,
    boolean useCache, boolean resultOrdered, KeyGenerator keyGenerator, 
    String keyProperty,String keyColumn, String databaseId, 
    LanguageDriver lang, String resultSets) {

    if (unresolvedCacheRef) {
        throw new IncompleteElementException("Cache-ref not yet resolved");
    }
　　// 拼接上命名空间，如 <select id="findOne" resultType="User">，则id=java.mybaits.dao.UserMapper.findOne
    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;

    // 创建建造器，设置各种属性
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        .resource(resource).fetchSize(fetchSize).timeout(timeout)
        .statementType(statementType).keyGenerator(keyGenerator)
        .keyProperty(keyProperty).keyColumn(keyColumn).databaseId(databaseId)
        .lang(lang).resultOrdered(resultOrdered).resultSets(resultSets)
        .resultMaps(getStatementResultMaps(resultMap, resultType, id))
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
        .resultSetType(resultSetType).useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);//这里用到了前面解析<cache>节点时创建的Cache对象，设置到MappedStatement对象里面的cache属性中

    // 获取或创建 ParameterMap
    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
        statementBuilder.parameterMap(statementParameterMap);
    }

    // 构建 MappedStatement
    MappedStatement statement = statementBuilder.build();
    // 添加 MappedStatement 到 configuration 的 mappedStatements 集合中
    // 通过UserMapper代理对象调用findOne方法时，就可以拼接UserMapper接口名java.mybaits.dao.UserMapper和findOne方法找到id=java.mybaits.dao.UserMapper的MappedStatement，然后执行对应的sql语句
    configuration.addMappedStatement(statement);
    return statement;
}
```



这里我们要注意，MappedStatement对象中有一个cache属性，将前面解析<cache>节点时创建的Cache对象，设置到MappedStatement对象里面的cache属性中，以备后面二级缓存使用，我们后面专门来讲这一块。

我们还要注意一个地方，.resultMaps(getStatementResultMaps(resultMap, resultType, id))，设置MappedStatement的resultMaps，我们来看看是怎么获取resultMap的



```
private List<ResultMap> getStatementResultMaps(String resultMap, Class<?> resultType, String statementId) {
    //拼接上当前nameSpace
    resultMap = this.applyCurrentNamespace(resultMap, true);
    //创建一个集合
    List<ResultMap> resultMaps = new ArrayList();
    if (resultMap != null) {
        //通过,分隔字符串，一般resultMap只会是一个，不会使用逗号
        String[] resultMapNames = resultMap.split(",");
        String[] arr$ = resultMapNames;
        int len$ = resultMapNames.length;

        for(int i$ = 0; i$ < len$; ++i$) {
            String resultMapName = arr$[i$];

            try {
                //从configuration中通过resultMapName获取ResultMap对象加入到resultMaps中
                resultMaps.add(this.configuration.getResultMap(resultMapName.trim()));
            } catch (IllegalArgumentException var11) {
                throw new IncompleteElementException("Could not find result map " + resultMapName, var11);
            }
        }
    } else if (resultType != null) {
        ResultMap inlineResultMap = (new org.apache.ibatis.mapping.ResultMap.Builder(this.configuration, statementId + "-Inline", resultType, new ArrayList(), (Boolean)null)).build();
        resultMaps.add(inlineResultMap);
    }

    return resultMaps;
}
```



从configuration中获取到ResultMap并设置到MappedStatement中，当查询结束后，就可以拿到ResultMap进行结果映射，这个在后面讲



### Mapper 接口绑定

映射文件解析完成后，我们需要通过命名空间将绑定 mapper 接口，看看具体绑定的啥



```
private void bindMapperForNamespace() {
    // 获取映射文件的命名空间
    String namespace = builderAssistant.getCurrentNamespace();
    if (namespace != null) {
        Class<?> boundType = null;
        try {
            // 根据命名空间解析 mapper 类型
            boundType = Resources.classForName(namespace);
        } catch (ClassNotFoundException e) {
        }
        if (boundType != null) {
            // 检测当前 mapper 类是否被绑定过
            if (!configuration.hasMapper(boundType)) {
                configuration.addLoadedResource("namespace:" + namespace);
                // 绑定 mapper 类
                configuration.addMapper(boundType);
            }
        }
    }
}

// Configuration
public <T> void addMapper(Class<T> type) {
    // 通过 MapperRegistry 绑定 mapper 类
    mapperRegistry.addMapper(type);
}

// MapperRegistry
public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
        if (hasMapper(type)) {
            throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
        }
        boolean loadCompleted = false;
        try {
            /*
             * 将 type 和 MapperProxyFactory 进行绑定，MapperProxyFactory 可为 mapper 接口生成代理类
             */
            knownMappers.put(type, new MapperProxyFactory<T>(type));
            
            MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
            // 解析注解中的信息
            parser.parse();
            loadCompleted = true;
        } finally {
            if (!loadCompleted) {
                knownMappers.remove(type);
            }
        }
    }
}
```



其实就是获取当前映射文件的命名空间，并获取其Class，也就是获取每个Mapper接口，然后为每个Mapper接口创建一个代理类工厂，new MapperProxyFactory<T>(type)，并放进 knownMappers 这个HashMap中，我们来看看这个MapperProxyFactory



```
public class MapperProxyFactory<T> {
    //存放Mapper接口Class
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap();

    public MapperProxyFactory(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    public Class<T> getMapperInterface() {
        return this.mapperInterface;
    }

    public Map<Method, MapperMethod> getMethodCache() {
        return this.methodCache;
    }

    protected T newInstance(MapperProxy<T> mapperProxy) {
        //生成mapperInterface的代理类
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
    }

    public T newInstance(SqlSession sqlSession) {
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }
}
```



这一块我们后面文章再来看是如何调用的。

# [Mybaits 源码解析 （四）----- SqlSession的创建过程](https://www.cnblogs.com/java-chen-hao/p/11743506.html)

**正文**

SqlSession是mybatis的核心接口之一，是myabtis接口层的主要组成部分，对外提供了mybatis常用的api。myabtis提供了两个SqlSesion接口的实现，常用的实现类是DefaultSqlSession。它相当于一个数据库连接对象，在一个SqlSession中可以执行多条SQL语句。



## 创建SqlSession

前面的两篇文章我们已经得到了`SqlSessionFactory`，那么`SqlSession`将由`SqlSessionFactory`进行创建。

```
SqlSession sqlSession=sqlSessionFactory.openSession();
```

我们就来看看这个`SqlSessionFactory`的 `openSession`方法是如何创建SqlSession对象的。根据上面的分析，这里的`SqlSessionFactory`类型对象其实是一个`DefaultSqlSessionFactory`对象，因此，需要到`DefaultSqlSessionFactory`类中去看`openSession`方法。



```
  @Override
  public SqlSession openSession() {
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
  }
```



调用了**openSessionFromDataSource**方法，并且第一个参数获取了默认的执行器类型，第二个参数为null,第三个参数为false,看看这个默认的执行器类型是啥

![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191028105214615-2132752424.png) ![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191028110352230-347735037.png)

默认的执行器类型SIMPLE，我们跟进**openSessionFromDataSource**方法



```
/**
 * ExecutorType 指定Executor的类型，分为三种：SIMPLE, REUSE, BATCH，默认使用的是SIMPLE
 * TransactionIsolationLevel 指定事务隔离级别，使用null,则表示使用数据库默认的事务隔离界别
 * autoCommit 是否自动提交，传过来的参数为false，表示不自动提交
 */
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
        // 获取配置中的环境信息，包括了数据源信息、事务等
        final Environment environment = configuration.getEnvironment();
        // 创建事务工厂
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
        // 创建事务，配置事务属性
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        // 创建Executor，即执行器
        // 它是真正用来Java和数据库交互操作的类，后面会展开说。
        final Executor executor = configuration.newExecutor(tx, execType);
        // 创建DefaultSqlSession对象返回，其实现了SqlSession接口
        return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
        closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```



主要包含以下几个步骤：

1. 首先从configuration获取Environment对象，里面主要包含了DataSource和TransactionFactory对象
2. 创建TransactionFactory
3. 创建Transaction
4. 从configuration获取Executor
5. 构造DefaultSqlSession对象

 我们先来看看常规的environment配置



```
//配置environment环境
<environments default="development">
    <environment id="development">
        /** 事务配置 type= JDBC、MANAGED 
         *  1.JDBC:这个配置直接简单使用了JDBC的提交和回滚设置。它依赖于从数据源得到的连接来管理事务范围。
         *  2.MANAGED:这个配置几乎没做什么。它从来不提交或回滚一个连接。
         */
        <transactionManager type="JDBC" />
        /** 数据源类型：type = UNPOOLED、POOLED、JNDI 
         *  1.UNPOOLED：这个数据源的实现是每次被请求时简单打开和关闭连接。
         *  2.POOLED：这是JDBC连接对象的数据源连接池的实现。 
         *  3.JNDI：这个数据源的实现是为了使用如Spring或应用服务器这类的容器
         */
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.jdbc.Driver" />
            <property name="url" value="jdbc:mysql://localhost:3306/xhm" />
            <property name="username" value="root" />
            <property name="password" value="root" />
            //默认连接事务隔离级别
            <property name="defaultTransactionIsolationLevel" value=""/> 
        </dataSource>
    </environment>
</environments>
```



还记得前面文章是怎么解析environments的吗，[Mybaits 源码解析 （二）----- 根据配置文件创建SqlSessionFactory（Configuration的创建过程）](https://www.cnblogs.com/java-chen-hao/p/11743430.html)，我们简单的回顾一下



```
private void environmentsElement(XNode context) throws Exception {
    if (context != null) {
        if (environment == null) {
            // 获取 default 属性
            environment = context.getStringAttribute("default");
        }
        for (XNode child : context.getChildren()) {
            // 获取 id 属性
            String id = child.getStringAttribute("id");
            /*
             * 检测当前 environment 节点的 id 与其父节点 environments 的属性 default 
             * 内容是否一致，一致则返回 true，否则返回 false
             * 将其default属性值与子元素environment的id属性值相等的子元素设置为当前使用的Environment对象
             */
            if (isSpecifiedEnvironment(id)) {
                // 将environment中的transactionManager标签转换为TransactionFactory对象
                TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                // 将environment中的dataSource标签转换为DataSourceFactory对象
                DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                // 创建 DataSource 对象
                DataSource dataSource = dsFactory.getDataSource();
                Environment.Builder environmentBuilder = new Environment.Builder(id)
                    .transactionFactory(txFactory)
                    .dataSource(dataSource);
                // 构建 Environment 对象，并设置到 configuration 中
                configuration.setEnvironment(environmentBuilder.build());
            }
        }
    }
}

private TransactionFactory transactionManagerElement(XNode context) throws Exception {
    if (context != null) {
        String type = context.getStringAttribute("type");
        Properties props = context.getChildrenAsProperties();
        //通过别名获取Class,并实例化
        TransactionFactory factory = (TransactionFactory)this.resolveClass(type).newInstance();
        factory.setProperties(props);
        return factory;
    } else {
        throw new BuilderException("Environment declaration requires a TransactionFactory.");
    }
}

private DataSourceFactory dataSourceElement(XNode context) throws Exception {
    if (context != null) {
        String type = context.getStringAttribute("type");
        //通过别名获取Class,并实例化
        Properties props = context.getChildrenAsProperties();
        DataSourceFactory factory = (DataSourceFactory)this.resolveClass(type).newInstance();
        factory.setProperties(props);
        return factory;
    } else {
        throw new BuilderException("Environment declaration requires a DataSourceFactory.");
    }
}
```



### 获取TransactionFactory

我们的environment配置中transactionManager type="JDBC"和dataSource type="POOLED"，则生成的**transactionManager为JdbcTransactionFactory，DataSourceFactory为PooledDataSourceFactory**

我们回到openSessionFromDataSource，接着看看**getTransactionFactoryFromEnvironment方法**



```
    private TransactionFactory getTransactionFactoryFromEnvironment(Environment environment) {
        return (TransactionFactory)(environment != null && environment.getTransactionFactory() != null ? environment.getTransactionFactory() : new ManagedTransactionFactory());
    }
```



### 创建Transaction

很明显 environment.getTransactionFactory() 就是**JdbcTransactionFactory，**看看这个工厂是如何创建Transaction的

```
public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
    return new JdbcTransaction(ds, level, autoCommit);
}
```

直接通过工厂方法创建了一个JdbcTransaction对象，并传参DataSource ，事务隔离级别null，自动提交false三个参数，我们来看看JdbcTransaction



```
public class JdbcTransaction implements Transaction {
    //数据库连接对象
    protected Connection connection;
    //数据库DataSource
    protected DataSource dataSource;
    //数据库隔离级别
    protected TransactionIsolationLevel level;
    //是否自动提交
    protected boolean autoCommmit;

    public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
        //设置dataSource和隔离级别，是否自动提交属性
        //这里隔离级别传过来的是null,表示使用数据库默认隔离级别，自动提交为false，表示不自动提交
        this.dataSource = ds;
        this.level = desiredLevel;
        this.autoCommmit = desiredAutoCommit;
    }

     public Connection getConnection() throws SQLException {
        if (this.connection == null) {
            this.openConnection();
        }

        return this.connection;
    }

    //提交功能是通过Connection去完成的
    public void commit() throws SQLException {
        if (this.connection != null && !this.connection.getAutoCommit()) {
            if (log.isDebugEnabled()) {
                log.debug("Committing JDBC Connection [" + this.connection + "]");
            }

            this.connection.commit();
        }

    }

    //回滚功能是通过Connection去完成的
    public void rollback() throws SQLException {
        if (this.connection != null && !this.connection.getAutoCommit()) {
            if (log.isDebugEnabled()) {
                log.debug("Rolling back JDBC Connection [" + this.connection + "]");
            }

            this.connection.rollback();
        }

    }

    //关闭功能是通过Connection去完成的
    public void close() throws SQLException {
        if (this.connection != null) {
            this.resetAutoCommit();
            if (log.isDebugEnabled()) {
                log.debug("Closing JDBC Connection [" + this.connection + "]");
            }

            this.connection.close();
        }

    }
    
    //获取连接是通过dataSource来完成的
    protected void openConnection() throws SQLException {
        if (log.isDebugEnabled()) {
            log.debug("Opening JDBC Connection");
        }

        this.connection = this.dataSource.getConnection();
        if (this.level != null) {
            this.connection.setTransactionIsolation(this.level.getLevel());
        }

        this.setDesiredAutoCommit(this.autoCommmit);
    }
}
```



JdbcTransaction主要维护了一个默认autoCommit为false的Connection对象，对事物的提交，回滚，关闭等都是接见通过Connection完成的。

### 创建Executor



```
//创建一个执行器，默认是SIMPLE
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    //根据executorType来创建相应的执行器,Configuration默认是SIMPLE
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      //创建SimpleExecutor实例，并且包含Configuration和transaction属性
      executor = new SimpleExecutor(this, transaction);
    }
    
    //如果要求缓存，生成另一种CachingExecutor,装饰者模式,默认都是返回CachingExecutor
    /**
     * 二级缓存开关配置示例
     * <settings>
     *   <setting name="cacheEnabled" value="true"/>
     * </settings>
     */
    if (cacheEnabled) {
      //CachingExecutor使用装饰器模式，将executor的功能添加上了二级缓存的功能，二级缓存会单独文章来讲
      executor = new CachingExecutor(executor);
    }
    //此处调用插件,通过插件可以改变Executor行为，此处我们后面单独文章讲
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```



executor包含了Configuration和刚刚创建的Transaction，默认的执行器为SimpleExecutor，如果开启了二级缓存(默认开启)，则CachingExecutor会包装SimpleExecutor，然后依次调用拦截器的plugin方法返回一个被代理过的Executor对象。

CachingExecutor 对象里面包含了刚创建的**SimpleExecutor**，后面文章我们会及具体讲这个类



```
public class CachingExecutor implements Executor {
    private Executor delegate;
    private TransactionalCacheManager tcm = new TransactionalCacheManager();

    public CachingExecutor(Executor delegate) {
        this.delegate = delegate;
        delegate.setExecutorWrapper(this);
    }
    //略
}
```



### 构造DefaultSqlSession对象

```
new DefaultSqlSession(this.configuration, executor, autoCommit);
```

传参configuration和刚生成的executor，我们来简单看看



```
public class DefaultSqlSession implements SqlSession {

  /**
   * mybatis全局配置新
   */
  private final Configuration configuration;
  /**
   * SQL执行器
   */
  private final Executor executor;

  /**
   * 是否自动提交
   */
  private final boolean autoCommit;

  private List<Cursor<?>> cursorList;
  
  public DefaultSqlSession(Configuration configuration, Executor executor, boolean autoCommit) {
        this.configuration = configuration;
        this.executor = executor;
        this.dirty = false;
        this.autoCommit = autoCommit;
  }
  
  @Override
  public <T> T selectOne(String statement) {
    return this.<T>selectOne(statement, null);
  }

  @Override
  public <T> T selectOne(String statement, Object parameter) {
    // Popular vote was to return null on 0 results and throw exception on too many.
    List<T> list = this.<T>selectList(statement, parameter);
    if (list.size() == 1) {
      return list.get(0);
    } else if (list.size() > 1) {
      throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
    } else {
      return null;
    }
  }
  @Override
  public <E> List<E> selectList(String statement) {
    return this.selectList(statement, null);
  }

  @Override
  public <E> List<E> selectList(String statement, Object parameter) {
    return this.selectList(statement, parameter, RowBounds.DEFAULT);
  }

  @Override
  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      MappedStatement ms = configuration.getMappedStatement(statement);
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
  
  //略....update等方法
}
```



SqlSession的所有查询接口最后都归结位Exector的方法调用。后面文章我们来分析其调用流程

# [Mybaits 源码解析 （五）----- Mapper接口底层原理（为什么Mapper不用写实现类就能访问到数据库？）](https://www.cnblogs.com/java-chen-hao/p/11752084.html)

**正文**

刚开始使用Mybaits的同学有没有这样的疑惑，为什么我们没有编写Mapper的实现类，却能调用Mapper的方法呢？本篇文章我带大家一起来解决这个疑问

上一篇文章我们获取到了DefaultSqlSession，接着我们来看第一篇文章测试用例后面的代码

```
EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
List<Employee> allEmployees = employeeMapper.getAll();
```



## 为 Mapper 接口创建代理对象

我们先从 DefaultSqlSession 的 getMapper 方法开始看起，如下：



```
 1 public <T> T getMapper(Class<T> type) {
 2     return configuration.<T>getMapper(type, this);
 3 }
 4 
 5 // Configuration
 6 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
 7     return mapperRegistry.getMapper(type, sqlSession);
 8 }
 9 
10 // MapperRegistry
11 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
12     // 从 knownMappers 中获取与 type 对应的 MapperProxyFactory
13     final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
14     if (mapperProxyFactory == null) {
15         throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
16     }
17     try {
18         // 创建代理对象
19         return mapperProxyFactory.newInstance(sqlSession);
20     } catch (Exception e) {
21         throw new BindingException("Error getting mapper instance. Cause: " + e, e);
22     }
23 }
```



这里最重要就是两行代码，第13行和第19行，我们接下来就分析这两行代码



### **获取MapperProxyFactory**

根据名称看，可以理解为Mapper代理的创建工厂，是不是Mapper的代理对象由它创建呢？我们先来回顾一下knownMappers 集合中的元素是何时存入的。这要在我前面的文章中找答案，MyBatis 在解析配置文件的 <mappers> 节点的过程中，会调用 MapperRegistry 的 addMapper 方法将 Class 到 MapperProxyFactory 对象的映射关系存入到 knownMappers。有兴趣的同学可以看看我之前的文章，我们来回顾一下源码：



```
private void bindMapperForNamespace() {
    // 获取映射文件的命名空间
    String namespace = builderAssistant.getCurrentNamespace();
    if (namespace != null) {
        Class<?> boundType = null;
        try {
            // 根据命名空间解析 mapper 类型
            boundType = Resources.classForName(namespace);
        } catch (ClassNotFoundException e) {
        }
        if (boundType != null) {
            // 检测当前 mapper 类是否被绑定过
            if (!configuration.hasMapper(boundType)) {
                configuration.addLoadedResource("namespace:" + namespace);
                // 绑定 mapper 类
                configuration.addMapper(boundType);
            }
        }
    }
}

// Configuration
public <T> void addMapper(Class<T> type) {
    // 通过 MapperRegistry 绑定 mapper 类
    mapperRegistry.addMapper(type);
}

// MapperRegistry
public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
        if (hasMapper(type)) {
            throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
        }
        boolean loadCompleted = false;
        try {
            /*
             * 将 type 和 MapperProxyFactory 进行绑定，MapperProxyFactory 可为 mapper 接口生成代理类
             */
            knownMappers.put(type, new MapperProxyFactory<T>(type));
            
            MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
            // 解析注解中的信息
            parser.parse();
            loadCompleted = true;
        } finally {
            if (!loadCompleted) {
                knownMappers.remove(type);
            }
        }
    }
}
```



在解析Mapper.xml的最后阶段，获取到Mapper.xml的namespace，然后利用反射，获取到namespace的Class,并创建一个**MapperProxyFactory的实例，namespace的Class作为参数，最后将namespace的Class为key，\**MapperProxyFactory的实例为value存入\******knownMappers。**

**注意，我们这里是通过****映射文件的命名空间的Class当做\**knownMappers的Key。然后我们看看\****getMapper方法的13行，是通过参数Employee.class也就是Mapper接口的Class来获取***\*MapperProxyFactory，所以我们明白了为什么要求xml配置中的namespace要和和对应的Mapper接口的全限定名了\****



### 生成代理对象

我们看第19行代码 **return mapperProxyFactory.newInstance(sqlSession);，**很明显是调用了MapperProxyFactory的一个工厂方法，我们跟进去看看



```
public class MapperProxyFactory<T> {
    //存放Mapper接口Class
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap();

    public MapperProxyFactory(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    public Class<T> getMapperInterface() {
        return this.mapperInterface;
    }

    public Map<Method, MapperMethod> getMethodCache() {
        return this.methodCache;
    }

    protected T newInstance(MapperProxy<T> mapperProxy) {
        //生成mapperInterface的代理类
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
    }

    public T newInstance(SqlSession sqlSession) {
         /*
         * 创建 MapperProxy 对象，MapperProxy 实现了 InvocationHandler 接口，代理逻辑封装在此类中
         * 将sqlSession传入MapperProxy对象中，第二个参数是Mapper的接口，并不是其实现类
         */
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }
}
```



上面的代码首先创建了一个 MapperProxy 对象，该对象实现了 InvocationHandler 接口。然后将对象作为参数传给重载方法，并在重载方法中调用 JDK 动态代理接口为 Mapper接口 生成代理对象。

这里要注意一点，MapperProxy这个InvocationHandler 创建的时候，传入的参数并不是Mapper接口的实现类，我们以前是怎么创建JDK动态代理的？先创建一个接口，然后再创建一个接口的实现类，最后创建一个InvocationHandler并将实现类传入其中作为目标类，创建接口的代理类，然后调用代理类方法时会回调InvocationHandler的invoke方法，最后在invoke方法中调用目标类的方法，但是我们这里调用Mapper接口代理类的方法时，需要调用其实现类的方法吗？不需要，我们需要调用对应的配置文件的SQL，所以这里并不需要传入Mapper的实现类到MapperProxy中，那Mapper接口的代理对象是如何调用对应配置文件的SQL呢？下面我们来看看。



## Mapper代理类如何执行SQL？

上面一节中我们已经获取到了EmployeeMapper的代理类，并且其InvocationHandler为MapperProxy，那我们接着看Mapper接口方法的调用

```
List<Employee> allEmployees = employeeMapper.getAll();
```

知道JDK动态代理的同学都知道，调用代理类的方法，最后都会回调到InvocationHandler的Invoke方法，那我们来看看这个InvocationHandler（MapperProxy）



```
public class MapperProxy<T> implements InvocationHandler, Serializable {
    private final SqlSession sqlSession;
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache;

    public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
        this.sqlSession = sqlSession;
        this.mapperInterface = mapperInterface;
        this.methodCache = methodCache;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 如果方法是定义在 Object 类中的，则直接调用
        if (Object.class.equals(method.getDeclaringClass())) {
            try {
                return method.invoke(this, args);
            } catch (Throwable var5) {
                throw ExceptionUtil.unwrapThrowable(var5);
            }
        } else {
            // 从缓存中获取 MapperMethod 对象，若缓存未命中，则创建 MapperMethod 对象
            MapperMethod mapperMethod = this.cachedMapperMethod(method);
            // 调用 execute 方法执行 SQL
            return mapperMethod.execute(this.sqlSession, args);
        }
    }

    private MapperMethod cachedMapperMethod(Method method) {
        MapperMethod mapperMethod = (MapperMethod)this.methodCache.get(method);
        if (mapperMethod == null) {
            //创建一个MapperMethod，参数为mapperInterface和method还有Configuration
            mapperMethod = new MapperMethod(this.mapperInterface, method, this.sqlSession.getConfiguration());
            this.methodCache.put(method, mapperMethod);
        }

        return mapperMethod;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，回调函数**invoke**逻辑会首先检测被拦截的方法是不是定义在 Object 中的，比如 equals、hashCode 方法等。对于这类方法，直接执行即可。紧接着从缓存中获取或者创建 MapperMethod 对象，然后通过该对象中的 execute 方法执行 SQL。我们先来看看如何创建MapperMethod



### 创建 MapperMethod 对象

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MapperMethod {

    //包含SQL相关信息，比喻MappedStatement的id属性，（mapper.EmployeeMapper.getAll）
    private final SqlCommand command;
    //包含了关于执行的Mapper方法的参数类型和返回类型。
    private final MethodSignature method;

    public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
        // 创建 SqlCommand 对象，该对象包含一些和 SQL 相关的信息
        this.command = new SqlCommand(config, mapperInterface, method);
        // 创建 MethodSignature 对象，从类名中可知，该对象包含了被拦截方法的一些信息
        this.method = new MethodSignature(config, mapperInterface, method);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

MapperMethod包含SqlCommand 和MethodSignature 对象，我们来看看其创建过程

**① 创建 SqlCommand 对象**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static class SqlCommand {
    //name为MappedStatement的id，也就是namespace.methodName（mapper.EmployeeMapper.getAll）
    private final String name;
    //SQL的类型，如insert，delete，update
    private final SqlCommandType type;

    public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
        //拼接Mapper接口名和方法名，（mapper.EmployeeMapper.getAll）
        String statementName = mapperInterface.getName() + "." + method.getName();
        MappedStatement ms = null;
        //检测configuration是否有key为mapper.EmployeeMapper.getAll的MappedStatement
        if (configuration.hasStatement(statementName)) {
            //获取MappedStatement
            ms = configuration.getMappedStatement(statementName);
        } else if (!mapperInterface.equals(method.getDeclaringClass())) {
            String parentStatementName = method.getDeclaringClass().getName() + "." + method.getName();
            if (configuration.hasStatement(parentStatementName)) {
                ms = configuration.getMappedStatement(parentStatementName);
            }
        }
        
        // 检测当前方法是否有对应的 MappedStatement
        if (ms == null) {
            if (method.getAnnotation(Flush.class) != null) {
                name = null;
                type = SqlCommandType.FLUSH;
            } else {
                throw new BindingException("Invalid bound statement (not found): "
                    + mapperInterface.getName() + "." + methodName);
            }
        } else {
            // 设置 name 和 type 变量
            name = ms.getId();
            type = ms.getSqlCommandType();
            if (type == SqlCommandType.UNKNOWN) {
                throw new BindingException("Unknown execution method for: " + name);
            }
        }
    }
}

public boolean hasStatement(String statementName, boolean validateIncompleteStatements) {
    //检测configuration是否有key为statementName的MappedStatement
    return this.mappedStatements.containsKey(statementName);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过拼接接口名和方法名，在configuration获取对应的MappedStatement，并设置设置 name 和 type 变量，代码很简单

**② 创建 MethodSignature 对象**

**MethodSignature** 包含了被拦截方法的一些信息，如目标方法的返回类型，目标方法的参数列表信息等。下面，我们来看一下 MethodSignature 的构造方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static class MethodSignature {

    private final boolean returnsMany;
    private final boolean returnsMap;
    private final boolean returnsVoid;
    private final boolean returnsCursor;
    private final Class<?> returnType;
    private final String mapKey;
    private final Integer resultHandlerIndex;
    private final Integer rowBoundsIndex;
    private final ParamNameResolver paramNameResolver;

    public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {

        // 通过反射解析方法返回类型
        Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);
        if (resolvedReturnType instanceof Class<?>) {
            this.returnType = (Class<?>) resolvedReturnType;
        } else if (resolvedReturnType instanceof ParameterizedType) {
            this.returnType = (Class<?>) ((ParameterizedType) resolvedReturnType).getRawType();
        } else {
            this.returnType = method.getReturnType();
        }
        
        // 检测返回值类型是否是 void、集合或数组、Cursor、Map 等
        this.returnsVoid = void.class.equals(this.returnType);
        this.returnsMany = configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray();
        this.returnsCursor = Cursor.class.equals(this.returnType);
        // 解析 @MapKey 注解，获取注解内容
        this.mapKey = getMapKey(method);
        this.returnsMap = this.mapKey != null;
        /*
         * 获取 RowBounds 参数在参数列表中的位置，如果参数列表中
         * 包含多个 RowBounds 参数，此方法会抛出异常
         */ 
        this.rowBoundsIndex = getUniqueParamIndex(method, RowBounds.class);
        // 获取 ResultHandler 参数在参数列表中的位置
        this.resultHandlerIndex = getUniqueParamIndex(method, ResultHandler.class);
        // 解析参数列表
        this.paramNameResolver = new ParamNameResolver(configuration, method);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 执行 execute 方法

前面已经分析了 MapperMethod 的初始化过程，现在 MapperMethod 创建好了。那么，接下来要做的事情是调用 MapperMethod 的 execute 方法，执行 SQL。传递参数sqlSession和method的运行参数args

```
return mapperMethod.execute(this.sqlSession, args);
```

我们去MapperMethod 的execute方法中看看

**MapperMethod**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    
    // 根据 SQL 类型执行相应的数据库操作
    switch (command.getType()) {
        case INSERT: {
            // 对用户传入的参数进行转换，下同
            Object param = method.convertArgsToSqlCommandParam(args);
            // 执行插入操作，rowCountResult 方法用于处理返回值
            result = rowCountResult(sqlSession.insert(command.getName(), param));
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            // 执行更新操作
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            // 执行删除操作
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            // 根据目标方法的返回类型进行相应的查询操作
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                // 执行查询操作，并返回多个结果 
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                // 执行查询操作，并将结果封装在 Map 中返回
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                // 执行查询操作，并返回一个 Cursor 对象
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                // 执行查询操作，并返回一个结果
                result = sqlSession.selectOne(command.getName(), param);
            }
            break;
        case FLUSH:
            // 执行刷新操作
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    return result;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，execute 方法主要由一个 switch 语句组成，用于根据 SQL 类型执行相应的数据库操作。我们先来看看是参数的处理方法convertArgsToSqlCommandParam是如何将方法参数数组转化成Map的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object convertArgsToSqlCommandParam(Object[] args) {
    return paramNameResolver.getNamedParams(args);
}

public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    if (args == null || paramCount == 0) {
        return null;
    } else if (!hasParamAnnotation && paramCount == 1) {
        return args[names.firstKey()];
    } else {
        //创建一个Map，key为method的参数名，值为method的运行时参数值
        final Map<String, Object> param = new ParamMap<Object>();
        int i = 0;
        for (Map.Entry<Integer, String> entry : names.entrySet()) {
            // 添加 <参数名, 参数值> 键值对到 param 中
            param.put(entry.getValue(), args[entry.getKey()]);
            final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
            if (!names.containsValue(genericParamName)) {
                param.put(genericParamName, args[entry.getKey()]);
            }
            i++;
        }
        return param;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到，将Object[] args转化成了一个Map<参数名, 参数值> ，接着我们就可以看查询过程分析了，如下

```
// 执行查询操作，并返回一个结果
result = sqlSession.selectOne(command.getName(), param);
```

我们看到是通过sqlSession来执行查询的，并且传入的参数为command.getName()和param，也就是**namespace.methodName（mapper.EmployeeMapper.getAll）和方法的运行参数。**

查询操作我们下一篇文章单独来讲

# [Mybaits 源码解析 （六）----- Select 语句的执行过程分析（上篇）](https://www.cnblogs.com/java-chen-hao/p/11754184.html)

**正文**

上一篇我们分析了Mapper接口代理类的生成，本篇接着分析是如何调用到XML中的SQL

我们回顾一下MapperMethod 的execute方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    
    // 根据 SQL 类型执行相应的数据库操作
    switch (command.getType()) {
        case INSERT: {
            // 对用户传入的参数进行转换，下同
            Object param = method.convertArgsToSqlCommandParam(args);
            // 执行插入操作，rowCountResult 方法用于处理返回值
            result = rowCountResult(sqlSession.insert(command.getName(), param));
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            // 执行更新操作
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            // 执行删除操作
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            // 根据目标方法的返回类型进行相应的查询操作
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                // 执行查询操作，并返回多个结果 
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                // 执行查询操作，并将结果封装在 Map 中返回
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                // 执行查询操作，并返回一个 Cursor 对象
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                // 执行查询操作，并返回一个结果
                result = sqlSession.selectOne(command.getName(), param);
            }
            break;
        case FLUSH:
            // 执行刷新操作
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    return result;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11754184.html#_labelTop)

## selectOne 方法分析

本节选择分析 selectOne 方法，主要是因为 selectOne 在内部会调用 selectList 方法。同时分析 selectOne 方法等同于分析 selectList 方法。代码如下

```
// 执行查询操作，并返回一个结果
result = sqlSession.selectOne(command.getName(), param);
```

我们看到是通过sqlSession来执行查询的，并且传入的参数为command.getName()和param，也就是namespace.methodName（mapper.EmployeeMapper.getAll）和方法的运行参数。我们知道了，所有的数据库操作都是交给sqlSession来执行的，那我们就来看看sqlSession的方法

**DefaultSqlSession**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <T> T selectOne(String statement, Object parameter) {
    // 调用 selectList 获取结果
    List<T> list = this.<T>selectList(statement, parameter);
    if (list.size() == 1) {
        // 返回结果
        return list.get(0);
    } else if (list.size() > 1) {
        // 如果查询结果大于1则抛出异常
        throw new TooManyResultsException(
            "Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
    } else {
        return null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，selectOne 方法在内部调用 selectList 了方法，并取 selectList 返回值的第1个元素作为自己的返回值。如果 selectList 返回的列表元素大于1，则抛出异常。下面我们来看看 selectList 方法的实现。

**DefaultSqlSession**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final Executor executor;
public <E> List<E> selectList(String statement, Object parameter) {
    // 调用重载方法
    return this.selectList(statement, parameter, RowBounds.DEFAULT);
}

public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
        // 通过MappedStatement的Id获取 MappedStatement
        MappedStatement ms = configuration.getMappedStatement(statement);
        // 调用 Executor 实现类中的 query 方法
        return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们之前创建**DefaultSqlSession**的时候，是创建了一个Executor的实例作为其属性的，我们看到**通过MappedStatement的Id获取 MappedStatement后，就交由Executor去执行了**

我们回顾一下前面的文章，Executor的创建过程，代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//创建一个执行器，默认是SIMPLE
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    //根据executorType来创建相应的执行器,Configuration默认是SIMPLE
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      //创建SimpleExecutor实例，并且包含Configuration和transaction属性
      executor = new SimpleExecutor(this, transaction);
    }
    
    //如果要求缓存，生成另一种CachingExecutor,装饰者模式,默认都是返回CachingExecutor
    /**
     * 二级缓存开关配置示例
     * <settings>
     *   <setting name="cacheEnabled" value="true"/>
     * </settings>
     */
    if (cacheEnabled) {
      //CachingExecutor使用装饰器模式，将executor的功能添加上了二级缓存的功能，二级缓存会单独文章来讲
      executor = new CachingExecutor(executor);
    }
    //此处调用插件,通过插件可以改变Executor行为，此处我们后面单独文章讲
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

executor包含了Configuration和Transaction，默认的执行器为SimpleExecutor，如果开启了二级缓存(默认开启)，则CachingExecutor会包装SimpleExecutor，那么我们该看CachingExecutor的**query**方法了

**CachingExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    // 获取 BoundSql
    BoundSql boundSql = ms.getBoundSql(parameterObject);
   // 创建 CacheKey
    CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    // 调用重载方法
    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的代码用于获取 BoundSql 对象，创建 CacheKey 对象，然后再将这两个对象传给重载方法。CacheKey 以及接下来即将出现的一二级缓存将会独立成文进行分析。



### 获取 BoundSql

我们先来看看获取BoundSql

```
// 获取 BoundSql
BoundSql boundSql = ms.getBoundSql(parameterObject);
```

调用了MappedStatement的getBoundSql方法，并将运行时参数传入其中，我们大概的猜一下，这里是不是拼接SQL语句呢，并将运行时参数设置到SQL语句中？

我们都知道 SQL 是配置在映射文件中的，但由于映射文件中的 SQL 可能会包含占位符 #{}，以及动态 SQL 标签，比如 <if>、<where> 等。因此，我们并不能直接使用映射文件中配置的 SQL。MyBatis 会将映射文件中的 SQL 解析成一组 SQL 片段。我们需要对这一组片段进行解析，从每个片段对象中获取相应的内容。然后将这些内容组合起来即可得到一个完成的 SQL 语句，这个完整的 SQL 以及其他的一些信息最终会存储在 BoundSql 对象中。下面我们来看一下 BoundSql 类的成员变量信息，如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final String sql;
private final List<ParameterMapping> parameterMappings;
private final Object parameterObject;
private final Map<String, Object> additionalParameters;
private final MetaObject metaParameters;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面用一个表格列举各个成员变量的含义。

| 变量名               | 类型       | 用途                                                         |
| -------------------- | ---------- | ------------------------------------------------------------ |
| sql                  | String     | 一个完整的 SQL 语句，可能会包含问号 ? 占位符                 |
| parameterMappings    | List       | 参数映射列表，SQL 中的每个 #{xxx} 占位符都会被解析成相应的 ParameterMapping 对象 |
| parameterObject      | Object     | 运行时参数，即用户传入的参数，比如 Article 对象，或是其他的参数 |
| additionalParameters | Map        | 附加参数集合，用于存储一些额外的信息，比如 datebaseId 等     |
| metaParameters       | MetaObject | additionalParameters 的元信息对象                            |

接下来我们接着MappedStatement 的 getBoundSql 方法，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public BoundSql getBoundSql(Object parameterObject) {

    // 调用 sqlSource 的 getBoundSql 获取 BoundSql，把method运行时参数传进去
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);return boundSql;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

MappedStatement 的 getBoundSql 在内部调用了 SqlSource 实现类的 getBoundSql 方法，并把method运行时参数传进去，SqlSource 是一个接口，它有如下几个实现类：

- DynamicSqlSource
- RawSqlSource
- StaticSqlSource
- ProviderSqlSource
- VelocitySqlSource

当 SQL 配置中包含 `${}`（不是 #{}）占位符，或者包含 <if>、<where> 等标签时，会被认为是动态 SQL，此时使用 DynamicSqlSource 存储 SQL 片段。否则，使用 RawSqlSource 存储 SQL 配置信息。我们来看看DynamicSqlSource的**getBoundSql**

**DynamicSqlSource**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public BoundSql getBoundSql(Object parameterObject) {
    // 创建 DynamicContext
    DynamicContext context = new DynamicContext(configuration, parameterObject);

    // 解析 SQL 片段，并将解析结果存储到 DynamicContext 中，这里会将${}替换成method对应的运行时参数，也会解析<if><where>等SqlNode
    rootSqlNode.apply(context);
    
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();
    /*
     * 构建 StaticSqlSource，在此过程中将 sql 语句中的占位符 #{} 替换为问号 ?，
     * 并为每个占位符构建相应的 ParameterMapping
     */
    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());
    
 // 调用 StaticSqlSource 的 getBoundSql 获取 BoundSql
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);

    // 将 DynamicContext 的 ContextMap 中的内容拷贝到 BoundSql 中
    for (Map.Entry<String, Object> entry : context.getBindings().entrySet()) {
        boundSql.setAdditionalParameter(entry.getKey(), entry.getValue());
    }
    return boundSql;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法由数个步骤组成，这里总结一下：

1. 创建 DynamicContext
2. 解析 SQL 片段，并将解析结果存储到 DynamicContext 中
3. 解析 SQL 语句，并构建 StaticSqlSource
4. 调用 StaticSqlSource 的 getBoundSql 获取 BoundSql
5. 将 DynamicContext 的 ContextMap 中的内容拷贝到 BoundSql

**DynamicContext**

DynamicContext 是 SQL 语句构建的上下文，每个 SQL 片段解析完成后，都会将解析结果存入 DynamicContext 中。待所有的 SQL 片段解析完毕后，一条完整的 SQL 语句就会出现在 DynamicContext 对象中。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class DynamicContext {

    public static final String PARAMETER_OBJECT_KEY = "_parameter";
    public static final String DATABASE_ID_KEY = "_databaseId";

    //bindings 则用于存储一些额外的信息，比如运行时参数
    private final ContextMap bindings;
    //sqlBuilder 变量用于存放 SQL 片段的解析结果
    private final StringBuilder sqlBuilder = new StringBuilder();

    public DynamicContext(Configuration configuration, Object parameterObject) {
        // 创建 ContextMap,并将运行时参数放入ContextMap中
        if (parameterObject != null && !(parameterObject instanceof Map)) {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            bindings = new ContextMap(metaObject);
        } else {
            bindings = new ContextMap(null);
        }

        // 存放运行时参数 parameterObject 以及 databaseId
        bindings.put(PARAMETER_OBJECT_KEY, parameterObject);
        bindings.put(DATABASE_ID_KEY, configuration.getDatabaseId());
    }

    
    public void bind(String name, Object value) {
        this.bindings.put(name, value);
    }

    //拼接Sql片段
    public void appendSql(String sql) {
        this.sqlBuilder.append(sql);
        this.sqlBuilder.append(" ");
    }
    
    //得到sql字符串
    public String getSql() {
        return this.sqlBuilder.toString().trim();
    }

    //继承HashMap
    static class ContextMap extends HashMap<String, Object> {

        private MetaObject parameterMetaObject;

        public ContextMap(MetaObject parameterMetaObject) {
            this.parameterMetaObject = parameterMetaObject;
        }

        @Override
        public Object get(Object key) {
            String strKey = (String) key;
            // 检查是否包含 strKey，若包含则直接返回
            if (super.containsKey(strKey)) {
                return super.get(strKey);
            }

            if (parameterMetaObject != null) {
                // 从运行时参数中查找结果，这里会在${name}解析时，通过name获取运行时参数值，替换掉${name}字符串
                return parameterMetaObject.getValue(strKey);
            }

            return null;
        }
    }
    // 省略部分代码
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**解析 SQL 片段**

接着我们来看看解析SQL片段的逻辑

```
rootSqlNode.apply(context);
```

对于一个包含了 ${} 占位符，或 <if>、<where> 等标签的 SQL，在解析的过程中，会被分解成多个片段。每个片段都有对应的类型，每种类型的片段都有不同的解析逻辑。在源码中，片段这个概念等价于 sql 节点，即 SqlNode。

StaticTextSqlNode 用于存储静态文本，TextSqlNode 用于存储带有 ${} 占位符的文本，IfSqlNode 则用于存储 <if> 节点的内容。MixedSqlNode 内部维护了一个 SqlNode 集合，用于存储各种各样的 SqlNode。接下来，我将会对 MixedSqlNode 、StaticTextSqlNode、TextSqlNode、IfSqlNode、WhereSqlNode 以及 TrimSqlNode 等进行分析

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MixedSqlNode implements SqlNode {
    private final List<SqlNode> contents;

    public MixedSqlNode(List<SqlNode> contents) {
        this.contents = contents;
    }

    @Override
    public boolean apply(DynamicContext context) {
        // 遍历 SqlNode 集合
        for (SqlNode sqlNode : contents) {
            // 调用 salNode 对象本身的 apply 方法解析 sql
            sqlNode.apply(context);
        }
        return true;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

MixedSqlNode 可以看做是 SqlNode 实现类对象的容器，凡是实现了 SqlNode 接口的类都可以存储到 MixedSqlNode 中，包括它自己。MixedSqlNode 解析方法 apply 逻辑比较简单，即遍历 SqlNode 集合，并调用其他 SqlNode实现类对象的 apply 方法解析 sql。

StaticTextSqlNode

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class StaticTextSqlNode implements SqlNode {

    private final String text;

    public StaticTextSqlNode(String text) {
        this.text = text;
    }

    @Override
    public boolean apply(DynamicContext context) {
        //直接拼接当前sql片段的文本到DynamicContext的sqlBuilder中
        context.appendSql(text);
        return true;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

StaticTextSqlNode 用于存储静态文本，直接将其存储的 SQL 的文本值拼接到 DynamicContext 的**sqlBuilder**中即可。下面分析一下 TextSqlNode。

TextSqlNode

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class TextSqlNode implements SqlNode {

    private final String text;
    private final Pattern injectionFilter;

    @Override
    public boolean apply(DynamicContext context) {
        // 创建 ${} 占位符解析器
        GenericTokenParser parser = createParser(new BindingTokenParser(context, injectionFilter));
        // 解析 ${} 占位符，通过ONGL 从用户传入的参数中获取结果，替换text中的${} 占位符
        // 并将解析结果的文本拼接到DynamicContext的sqlBuilder中
        context.appendSql(parser.parse(text));
        return true;
    }

    private GenericTokenParser createParser(TokenHandler handler) {
        // 创建占位符解析器
        return new GenericTokenParser("${", "}", handler);
    }

    private static class BindingTokenParser implements TokenHandler {

        private DynamicContext context;
        private Pattern injectionFilter;

        public BindingTokenParser(DynamicContext context, Pattern injectionFilter) {
            this.context = context;
            this.injectionFilter = injectionFilter;
        }

        @Override
        public String handleToken(String content) {
            Object parameter = context.getBindings().get("_parameter");
            if (parameter == null) {
                context.getBindings().put("value", null);
            } else if (SimpleTypeRegistry.isSimpleType(parameter.getClass())) {
                context.getBindings().put("value", parameter);
            }
            // 通过 ONGL 从用户传入的参数中获取结果
            Object value = OgnlCache.getValue(content, context.getBindings());
            String srtValue = (value == null ? "" : String.valueOf(value));
            // 通过正则表达式检测 srtValue 有效性
            checkInjection(srtValue);
            return srtValue;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

GenericTokenParser 是一个通用的标记解析器，用于解析形如 ${name}，#{id} 等标记。此时是解析 ${name}的形式，从运行时参数的Map中获取到key为name的值，直接用运行时参数替换掉 ${name}字符串，将替换后的text字符串拼接到DynamicContext的sqlBuilder中

举个例子吧，比喻我们有如下SQL

```
SELECT * FROM user WHERE name = '${name}' and id= ${id}
```

假如我们传的参数 Map中name值为 chenhao,id为1，那么该 SQL 最终会被解析成如下的结果：

```
SELECT * FROM user WHERE name = 'chenhao' and id= 1
```

很明显这种直接拼接值很容易造成SQL注入，假如我们传入的参数为name值为 chenhao'; DROP TABLE user;# ，解析得到的结果为

```
SELECT * FROM user WHERE name = 'chenhao'; DROP TABLE user;#'
```

由于传入的参数没有经过转义，最终导致了一条 SQL 被恶意参数拼接成了两条 SQL。这就是为什么我们不应该在 SQL 语句中是用 ${} 占位符，风险太大。接着我们来看看IfSqlNode

IfSqlNode

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class IfSqlNode implements SqlNode {

    private final ExpressionEvaluator evaluator;
    private final String test;
    private final SqlNode contents;

    public IfSqlNode(SqlNode contents, String test) {
        this.test = test;
        this.contents = contents;
        this.evaluator = new ExpressionEvaluator();
    }

    @Override
    public boolean apply(DynamicContext context) {
        // 通过 ONGL 评估 test 表达式的结果
        if (evaluator.evaluateBoolean(test, context.getBindings())) {
            // 若 test 表达式中的条件成立，则调用其子节点节点的 apply 方法进行解析
            // 如果是静态SQL节点，则会直接拼接到DynamicContext中
            contents.apply(context);
            return true;
        }
        return false;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

IfSqlNode 对应的是 <if test='xxx'> 节点，首先是通过 ONGL 检测 test 表达式是否为 true，如果为 true，则调用其子节点的 apply 方法继续进行解析。如果子节点是静态SQL节点，则子节点的文本值会直接拼接到DynamicContext中

好了，其他的SqlNode我就不一一分析了，大家有兴趣的可以去看看

**解析 #{} 占位符**

经过前面的解析，我们已经能从 DynamicContext 获取到完整的 SQL 语句了。但这并不意味着解析过程就结束了，因为当前的 SQL 语句中还有一种占位符没有处理，即 #{}。与 ${} 占位符的处理方式不同，MyBatis 并不会直接将 #{} 占位符替换为相应的参数值，而是将其替换成**？**。其解析是在如下代码中实现的

```
SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());
```

我们看到将前面解析过的sql字符串和运行时参数的Map作为参数，我们来看看parse方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public SqlSource parse(String originalSql, Class<?> parameterType, Map<String, Object> additionalParameters) {
    // 创建 #{} 占位符处理器
    ParameterMappingTokenHandler handler = new ParameterMappingTokenHandler(configuration, parameterType, additionalParameters);
    // 创建 #{} 占位符解析器
    GenericTokenParser parser = new GenericTokenParser("#{", "}", handler);
    // 解析 #{} 占位符，并返回解析结果字符串
    String sql = parser.parse(originalSql);
    // 封装解析结果到 StaticSqlSource 中，并返回,因为所有的动态参数都已经解析了，可以封装成一个静态的SqlSource
    return new StaticSqlSource(configuration, sql, handler.getParameterMappings());
}

public String handleToken(String content) {
    // 获取 content 的对应的 ParameterMapping
    parameterMappings.add(buildParameterMapping(content));
    // 返回 ?
    return "?";
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到将Sql中的 #{} 占位符替换成**"?"，并且将对应的参数转化成ParameterMapping 对象**，通过buildParameterMapping 完成,最后创建一个**StaticSqlSource，**将sql字符串和**ParameterMappings为参数传入，返回这个\**StaticSqlSource\****

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private ParameterMapping buildParameterMapping(String content) {
    /*
     * 将#{xxx} 占位符中的内容解析成 Map。
     *   #{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}
     *      上面占位符中的内容最终会被解析成如下的结果：
     *  {
     *      "property": "age",
     *      "typeHandler": "MyTypeHandler", 
     *      "jdbcType": "NUMERIC", 
     *      "javaType": "int"
     *  }
     */
    Map<String, String> propertiesMap = parseParameterMapping(content);
    String property = propertiesMap.get("property");
    Class<?> propertyType;
    // metaParameters 为 DynamicContext 成员变量 bindings 的元信息对象
    if (metaParameters.hasGetter(property)) {
        propertyType = metaParameters.getGetterType(property);
    
    /*
     * parameterType 是运行时参数的类型。如果用户传入的是单个参数，比如 Employe 对象，此时 
     * parameterType 为 Employe.class。如果用户传入的多个参数，比如 [id = 1, author = "chenhao"]，
     * MyBatis 会使用 ParamMap 封装这些参数，此时 parameterType 为 ParamMap.class。
     */
    } else if (typeHandlerRegistry.hasTypeHandler(parameterType)) {
        propertyType = parameterType;
    } else if (JdbcType.CURSOR.name().equals(propertiesMap.get("jdbcType"))) {
        propertyType = java.sql.ResultSet.class;
    } else if (property == null || Map.class.isAssignableFrom(parameterType)) {
        propertyType = Object.class;
    } else {
        /*
         * 代码逻辑走到此分支中，表明 parameterType 是一个自定义的类，
         * 比如 Employe，此时为该类创建一个元信息对象
         */
        MetaClass metaClass = MetaClass.forClass(parameterType, configuration.getReflectorFactory());
        // 检测参数对象有没有与 property 想对应的 getter 方法
        if (metaClass.hasGetter(property)) {
            // 获取成员变量的类型
            propertyType = metaClass.getGetterType(property);
        } else {
            propertyType = Object.class;
        }
    }
    
    ParameterMapping.Builder builder = new ParameterMapping.Builder(configuration, property, propertyType);
    
    // 将 propertyType 赋值给 javaType
    Class<?> javaType = propertyType;
    String typeHandlerAlias = null;
    
    // 遍历 propertiesMap
    for (Map.Entry<String, String> entry : propertiesMap.entrySet()) {
        String name = entry.getKey();
        String value = entry.getValue();
        if ("javaType".equals(name)) {
            // 如果用户明确配置了 javaType，则以用户的配置为准
            javaType = resolveClass(value);
            builder.javaType(javaType);
        } else if ("jdbcType".equals(name)) {
            // 解析 jdbcType
            builder.jdbcType(resolveJdbcType(value));
        } else if ("mode".equals(name)) {...} 
        else if ("numericScale".equals(name)) {...} 
        else if ("resultMap".equals(name)) {...} 
        else if ("typeHandler".equals(name)) {
            typeHandlerAlias = value;    
        } 
        else if ("jdbcTypeName".equals(name)) {...} 
        else if ("property".equals(name)) {...} 
        else if ("expression".equals(name)) {
            throw new BuilderException("Expression based parameters are not supported yet");
        } else {
            throw new BuilderException("An invalid property '" + name + "' was found in mapping #{" + content
                + "}.  Valid properties are " + parameterProperties);
        }
    }
    if (typeHandlerAlias != null) {
        builder.typeHandler(resolveTypeHandler(javaType, typeHandlerAlias));
    }
    
    // 构建 ParameterMapping 对象
    return builder.build();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

SQL 中的 #{name, ...} 占位符被替换成了问号 ?。#{name, ...} 也被解析成了一个 ParameterMapping 对象。我们再来看一下 StaticSqlSource 的创建过程。如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class StaticSqlSource implements SqlSource {

    private final String sql;
    private final List<ParameterMapping> parameterMappings;
    private final Configuration configuration;

    public StaticSqlSource(Configuration configuration, String sql) {
        this(configuration, sql, null);
    }

    public StaticSqlSource(Configuration configuration, String sql, List<ParameterMapping> parameterMappings) {
        this.sql = sql;
        this.parameterMappings = parameterMappings;
        this.configuration = configuration;
    }

    @Override
    public BoundSql getBoundSql(Object parameterObject) {
        // 创建 BoundSql 对象
        return new BoundSql(configuration, sql, parameterMappings, parameterObject);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后我们通过创建的StaticSqlSource就可以获取BoundSql对象了，并传入运行时参数

```
BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
```

也就是调用上面创建的StaticSqlSource 中的getBoundSql方法，这是简单的 **return new** **BoundSql(configuration, sql, parameterMappings, parameterObject);** ，接着看看BoundSql

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class BoundSql {
    private String sql;
    private List<ParameterMapping> parameterMappings;
   private Object parameterObject;
    private Map<String, Object> additionalParameters;
    private MetaObject metaParameters;

    public BoundSql(Configuration configuration, String sql, List<ParameterMapping> parameterMappings, Object parameterObject) {
        this.sql = sql;
        this.parameterMappings = parameterMappings;
        this.parameterObject = parameterObject;
        this.additionalParameters = new HashMap();
        this.metaParameters = configuration.newMetaObject(this.additionalParameters);
    }

    public String getSql() {
        return this.sql;
    }
    //略
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到只是做简单的赋值。BoundSql中包含了sql，#{}解析成的parameterMappings，还有运行时参数parameterObject。好了，SQL解析我们就介绍这么多。我们先回顾一下我们代码是从哪里开始的

**CachingExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
2     // 获取 BoundSql
3     BoundSql boundSql = ms.getBoundSql(parameterObject);
4    // 创建 CacheKey
5     CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
6     // 调用重载方法
7     return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，我们刚才都是分析的第三行代码，获取到了**BoundSql，**CacheKey 和二级缓存有关，我们留在下一篇文章单独来讲，接着我们看第七行重载方法 **query**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    // 从 MappedStatement 中获取缓存
    Cache cache = ms.getCache();
    // 若映射文件中未配置缓存或参照缓存，此时 cache = null
    if (cache != null) {
        flushCacheIfRequired(ms);
        if (ms.isUseCache() && resultHandler == null) {
            ensureNoOutParams(ms, boundSql);
            List<E> list = (List<E>) tcm.getObject(cache, key);
            if (list == null) {
                // 若缓存未命中，则调用被装饰类的 query 方法，也就是SimpleExecutor的query方法
                list = delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                tcm.putObject(cache, key, list); // issue #578 and #116
            }
            return list;
        }
    }
    // 调用被装饰类的 query 方法,这里的delegate我们知道应该是SimpleExecutor
    return delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的代码涉及到了二级缓存，若二级缓存为空，或未命中，则调用被装饰类的 query 方法。被装饰类为SimpleExecutor，而SimpleExecutor继承BaseExecutor，那我们来看看 BaseExecutor 的query方法

**BaseExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
        clearLocalCache();
    }
    List<E> list;
    try {
        queryStack++;
        // 从一级缓存中获取缓存项，一级缓存我们也下一篇文章单独讲
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
            handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
            // 一级缓存未命中，则从数据库中查询
            list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
    } finally {
        queryStack--;
    }
    if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
            deferredLoad.load();
        }
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
            clearLocalCache();
        }
    }
    return list;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从一级缓存中查找查询结果。若缓存未命中，再向数据库进行查询。至此我们明白了一级二级缓存的大概思路，先从二级缓存中查找，若未命中二级缓存，再从一级缓存中查找，若未命中一级缓存，再从数据库查询数据，那我们来看看是怎么从数据库查询的

**BaseExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds,
    ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    // 向缓存中存储一个占位符
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
        // 调用 doQuery 进行查询
        list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
        // 移除占位符
        localCache.removeObject(key);
    }
    // 缓存查询结果
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
        localOutputParameterCache.putObject(key, parameter);
    }
    return list;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

调用了**doQuery**方法进行查询，最后将查询结果放入一级缓存，我们来看看doQuery,在SimpleExecutor中

**SimpleExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
        Configuration configuration = ms.getConfiguration();
        // 创建 StatementHandler
        StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
        // 创建 Statement
        stmt = prepareStatement(handler, ms.getStatementLog());
        // 执行查询操作
        return handler.<E>query(stmt, resultHandler);
    } finally {
        // 关闭 Statement
        closeStatement(stmt);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们先来看看第一步创建**StatementHandler** 



### 创建StatementHandler 

StatementHandler有什么作用呢？通过这个对象获取Statement对象，然后填充运行时参数，最后调用query完成查询。我们来看看其创建过程

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement,
    Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    // 创建具有路由功能的 StatementHandler
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    // 应用插件到 StatementHandler 上
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看看RoutingStatementHandler的构造方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class RoutingStatementHandler implements StatementHandler {

    private final StatementHandler delegate;

    public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds,
        ResultHandler resultHandler, BoundSql boundSql) {

        // 根据 StatementType 创建不同的 StatementHandler 
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
    
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

RoutingStatementHandler 的构造方法会根据 MappedStatement 中的 statementType 变量创建不同的 StatementHandler 实现类。那statementType 是什么呢？我们还要回顾一下MappedStatement 的创建过程

![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191029105527512-170019357.png)

 

我们看到statementType 的默认类型为**PREPARED，这里将会创建****PreparedStatementHandler。**

接着我们看下面一行代码prepareStatement,



### 创建 Statement

创建 Statement 在 stmt = prepareStatement(handler, ms.getStatementLog()); 这句代码，那我们跟进去看看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    // 获取数据库连接
    Connection connection = getConnection(statementLog);
    // 创建 Statement，
    stmt = handler.prepare(connection, transaction.getTimeout());
  // 为 Statement 设置参数
    handler.parameterize(stmt);
    return stmt;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在上面的代码中我们终于看到了和jdbc相关的内容了，创建完**Statement，最后就可以****执行查询操作了。**由于篇幅的原因，我们留在下一篇文章再来详细讲解

# [Mybaits 源码解析 （七）----- Select 语句的执行过程分析（下篇）](https://www.cnblogs.com/java-chen-hao/p/11758412.html)

**正文**

我们上篇文章讲到了查询方法里面的doQuery方法，这里面就是调用JDBC的API了，其中的逻辑比较复杂，我们这边文章来讲，先看看我们上篇文章分析的地方

**SimpleExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
 2     Statement stmt = null;
 3     try {
 4         Configuration configuration = ms.getConfiguration();
 5         // 创建 StatementHandler
 6         StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
 7         // 创建 Statement
 8         stmt = prepareStatement(handler, ms.getStatementLog());
 9         // 执行查询操作
10         return handler.<E>query(stmt, resultHandler);
11     } finally {
12         // 关闭 Statement
13         closeStatement(stmt);
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上篇文章我们分析完了第6行代码，在第6行处我们创建了一个**PreparedStatementHandler，**我们要接着第8行代码开始分析，也就是创建 Statement，先不忙着分析，我们先来回顾一下 ，我们以前是怎么使用jdbc的

**jdbc**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Login {
    /**
     *    第一步，加载驱动，创建数据库的连接
     *    第二步，编写sql
     *    第三步，需要对sql进行预编译
     *    第四步，向sql里面设置参数
     *    第五步，执行sql
     *    第六步，释放资源 
     * @throws Exception 
     */
     
    public static final String URL = "jdbc:mysql://localhost:3306/chenhao";
    public static final String USER = "liulx";
    public static final String PASSWORD = "123456";
    public static void main(String[] args) throws Exception {
        login("lucy","123");
    }
    
    public static void login(String username , String password) throws Exception{
        Connection conn = null; 
        PreparedStatement psmt = null;
        ResultSet rs = null;
        try {
            //加载驱动程序
            Class.forName("com.mysql.jdbc.Driver");
            //获得数据库连接
            conn = DriverManager.getConnection(URL, USER, PASSWORD);
            //编写sql
            String sql = "select * from user where name =? and password = ?";//问号相当于一个占位符
            //对sql进行预编译
            psmt = conn.prepareStatement(sql);
            //设置参数
            psmt.setString(1, username);
            psmt.setString(2, password);
            //执行sql ,返回一个结果集
            rs = psmt.executeQuery();
            //输出结果
            while(rs.next()){
                System.out.println(rs.getString("user_name")+" 年龄："+rs.getInt("age"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            //释放资源
            conn.close();
            psmt.close();
            rs.close();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面代码中注释已经很清楚了，我们来看看mybatis中是怎么和数据库打交道的。

**SimpleExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    // 获取数据库连接
    Connection connection = getConnection(statementLog);
   // 创建 Statement，
    stmt = handler.prepare(connection, transaction.getTimeout());
   // 为 Statement 设置参数
    handler.parameterize(stmt);
    return stmt;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在上面的代码中我们终于看到了和jdbc相关的内容了，大概分为下面三个步骤：

1. 获取数据库连接
2. 创建PreparedStatement
3. 为PreparedStatement设置运行时参数

我们先来看看获取数据库连接，跟进代码看看

**BaseExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected Connection getConnection(Log statementLog) throws SQLException {
    //通过transaction来获取Connection
    Connection connection = this.transaction.getConnection();
    return statementLog.isDebugEnabled() ? ConnectionLogger.newInstance(connection, statementLog, this.queryStack) : connection;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到是通过**Executor中的**transaction属性来获取Connection，那我们就先来看看transaction，根据前面的文章中的配置**` ``<``transactionManager` `type="jdbc"/>，`**则MyBatis会创建一个**JdbcTransactionFactory.class** 实例，Executor中的transaction是一个**JdbcTransaction.class** 实例，其实现Transaction接口，那我们先来看看Transaction

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11758412.html#_labelTop)

## **JdbcTransaction**

我们先来看看其接口Transaction



### **Transaction**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface Transaction {
    //获取数据库连接
    Connection getConnection() throws SQLException;
    //提交事务
    void commit() throws SQLException;
    //回滚事务
    void rollback() throws SQLException;
    //关闭事务
    void close() throws SQLException;
    //获取超时时间
    Integer getTimeout() throws SQLException;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接着我们看看其实现类**JdbcTransaction**



### **JdbcTransaction**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class JdbcTransaction implements Transaction {
  
  private static final Log log = LogFactory.getLog(JdbcTransaction.class);
  
  //数据库连接
  protected Connection connection;
  //数据源信息
  protected DataSource dataSource;
  //隔离级别
  protected TransactionIsolationLevel level;
  //是否为自动提交
  protected boolean autoCommmit;
  
  public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
    dataSource = ds;
    level = desiredLevel;
    autoCommmit = desiredAutoCommit;
  }
  
  public JdbcTransaction(Connection connection) {
    this.connection = connection;
  }
  
  public Connection getConnection() throws SQLException {
    //如果事务中不存在connection，则获取一个connection并放入connection属性中
    //第一次肯定为空
    if (connection == null) {
      openConnection();
    }
    //如果事务中已经存在connection，则直接返回这个connection
    return connection;
  }
  
    /**
     * commit()功能 
     * @throws SQLException
     */
  public void commit() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Committing JDBC Connection [" + connection + "]");
      }
      //使用connection的commit()
      connection.commit();
    }
  }
  
    /**
     * rollback()功能 
     * @throws SQLException
     */
  public void rollback() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Rolling back JDBC Connection [" + connection + "]");
      }
      //使用connection的rollback()
      connection.rollback();
    }
  }
  
    /**
     * close()功能 
     * @throws SQLException
     */
  public void close() throws SQLException {
    if (connection != null) {
      resetAutoCommit();
      if (log.isDebugEnabled()) {
        log.debug("Closing JDBC Connection [" + connection + "]");
      }
      //使用connection的close()
      connection.close();
    }
  }
  
  protected void openConnection() throws SQLException {
    if (log.isDebugEnabled()) {
      log.debug("Opening JDBC Connection");
    }
    //通过dataSource来获取connection，并设置到transaction的connection属性中
    connection = dataSource.getConnection();
   if (level != null) {
      //通过connection设置事务的隔离级别
      connection.setTransactionIsolation(level.getLevel());
    }
    //设置事务是否自动提交
    setDesiredAutoCommit(autoCommmit);
  }
  
  protected void setDesiredAutoCommit(boolean desiredAutoCommit) {
    try {
        if (this.connection.getAutoCommit() != desiredAutoCommit) {
            if (log.isDebugEnabled()) {
                log.debug("Setting autocommit to " + desiredAutoCommit + " on JDBC Connection [" + this.connection + "]");
            }
            //通过connection设置事务是否自动提交
            this.connection.setAutoCommit(desiredAutoCommit);
        }

    } catch (SQLException var3) {
        throw new TransactionException("Error configuring AutoCommit.  Your driver may not support getAutoCommit() or setAutoCommit(). Requested setting: " + desiredAutoCommit + ".  Cause: " + var3, var3);
    }
  }
  
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到JdbcTransaction中有一个**Connection属性和dataSource属性，使用****connection来进行提交、回滚、关闭等操作，也就是说JdbcTransaction其实只是在jdbc的connection上面封装了一下，实际使用的其实还是jdbc的事务。**我们看看getConnection()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//数据库连接
protected Connection connection;
//数据源信息
protected DataSource dataSource;

public Connection getConnection() throws SQLException {
//如果事务中不存在connection，则获取一个connection并放入connection属性中
//第一次肯定为空
if (connection == null) {
  openConnection();
}
//如果事务中已经存在connection，则直接返回这个connection
return connection;
}

protected void openConnection() throws SQLException {
if (log.isDebugEnabled()) {
  log.debug("Opening JDBC Connection");
}
//通过dataSource来获取connection，并设置到transaction的connection属性中
connection = dataSource.getConnection();
if (level != null) {
  //通过connection设置事务的隔离级别
  connection.setTransactionIsolation(level.getLevel());
}
//设置事务是否自动提交
setDesiredAutoCommit(autoCommmit);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先是判断当前事务中是否存在connection，如果存在，则直接返回connection，如果不存在则通过dataSource来获取connection，这里我们明白了一点，如果当前事务没有关闭，也就是没有释放connection，那么在同一个Transaction中使用的是同一个connection,我们再来想想，**transaction是SimpleExecutor中的属性，\**SimpleExecutor又是SqlSession中的属性，那我们可以这样说，同一个\*\*\*\*SqlSession中只有一个\*\*\*\*SimpleExecutor，\*\*\*\*\*\*\*\*\*\*\*\*SimpleExecutor中有一个\*\*Transaction，\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*Transaction有一个connection。我们来看看如下例子\*\*\*\*\*\*\*\*
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    //创建一个SqlSession
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
         EmployeeMapper employeeMapper = sqlSession.getMapper(Employee.class);
         UserMapper userMapper = sqlSession.getMapper(User.class);
         List<Employee> allEmployee = employeeMapper.getAll();
         List<User> allUser = userMapper.getAll();
         Employee employee = employeeMapper.getOne();
    } finally {
        sqlSession.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**我们看到同一个sqlSession可以获取多个Mapper代理对象，则多个Mapper代理对象中的sqlSession引用应该是同一个，那么多个Mapper代理对象调用方法应该是同一个Connection，直到调用close(),所以说我们的\**sqlSession是线程不安全的，如果所有的业务都使用一个\*\*\*\*sqlSession，那\*\*Connection也是同一个，一个业务执行完了就将其关闭，那其他的业务还没执行完呢。大家明白了吗？\*\*\*\*\*\**\***我们回归到源码，connection = dataSource.getConnection();，最终还是调用dataSource来获取连接，那我们是不是要来看看dataSource呢？

我们还是从前面的配置文件来看**<dataSource type="UNPOOLED|POOLED">，这里有\**UNPOOLED和POOLED两种\**DataSource，一种是使用连接池，一种是普通的\**DataSource，\*\*\*\*UNPOOLED将会创将new UnpooledDataSource()实例，\*\*\*\*POOLED将会\*\*\*\*\*\*\*\*new pooledDataSource()实例，都实现\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*DataSource接口，\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****那我们先来看看DataSource接口

**DataSource**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface DataSource  extends CommonDataSource,Wrapper {
  //获取数据库连接
  Connection getConnection() throws SQLException;

  Connection getConnection(String username, String password)
    throws SQLException;

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

很简单，只有一个获取数据库连接的接口，那我们来看看其实现类

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11758412.html#_labelTop)

## UnpooledDataSource

UnpooledDataSource，从名称上即可知道，该种数据源不具有池化特性。该种数据源每次会返回一个新的数据库连接，而非复用旧的连接。其核心的方法有三个，分别如下：

1. initializeDriver - 初始化数据库驱动
2. doGetConnection - 获取数据连接
3. configureConnection - 配置数据库连接



### 初始化数据库驱动

看下我们上面使用JDBC的例子，在执行 SQL 之前，通常都是先获取数据库连接。一般步骤都是加载数据库驱动，然后通过 DriverManager 获取数据库连接。UnpooledDataSource 也是使用 JDBC 访问数据库的，因此它获取数据库连接的过程一样

**UnpooledDataSource**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class UnpooledDataSource implements DataSource {
    private ClassLoader driverClassLoader;
    private Properties driverProperties;
    private static Map<String, Driver> registeredDrivers = new ConcurrentHashMap();
    private String driver;
    private String url;
    private String username;
    private String password;
    private Boolean autoCommit;
    private Integer defaultTransactionIsolationLevel;

    public UnpooledDataSource() {
    }

    public UnpooledDataSource(String driver, String url, String username, String password) {
        this.driver = driver;
        this.url = url;
        this.username = username;
        this.password = password;
    }

    private synchronized void initializeDriver() throws SQLException {
        // 检测当前 driver 对应的驱动实例是否已经注册
        if (!registeredDrivers.containsKey(driver)) {
            Class<?> driverType;
            try {
                // 加载驱动类型
                if (driverClassLoader != null) {
                    // 使用 driverClassLoader 加载驱动
                    driverType = Class.forName(driver, true, driverClassLoader);
                } else {
                    // 通过其他 ClassLoader 加载驱动
                    driverType = Resources.classForName(driver);
                }

                // 通过反射创建驱动实例
                Driver driverInstance = (Driver) driverType.newInstance();
                /*
                 * 注册驱动，注意这里是将 Driver 代理类 DriverProxy 对象注册到 DriverManager 中的，而非 Driver 对象本身。
                 */
                DriverManager.registerDriver(new DriverProxy(driverInstance));
                // 缓存驱动类名和实例，防止多次注册
                registeredDrivers.put(driver, driverInstance);
            } catch (Exception e) {
                throw new SQLException("Error setting driver on UnpooledDataSource. Cause: " + e);
            }
        }
    }
    //略...
}

//DriverManager
private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<DriverInfo>();
public static synchronized void registerDriver(java.sql.Driver driver)
    throws SQLException {

    if(driver != null) {
        registeredDrivers.addIfAbsent(new DriverInfo(driver));
    } else {
        // This is for compatibility with the original DriverManager
        throw new NullPointerException();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
通过反射机制加载驱动Driver，并将其注册到DriverManager中的一个常量集合中，供后面获取连接时使用，为什么这里是一个List呢？我们实际开发中有可能使用到了多种数据库类型，如Mysql、Oracle等，其驱动都是不同的，不同的数据源获取连接时使用的是不同的驱动。
在我们使用JDBC的时候，也没有通过DriverManager.registerDriver(new DriverProxy(driverInstance));去注册Driver啊，如果我们使用的是Mysql数据源，那我们来看Class.forName("com.mysql.jdbc.Driver");这句代码发生了什么
Class.forName主要是做了什么呢？它主要是要求JVM查找并装载指定的类。这样我们的类com.mysql.jdbc.Driver就被装载进来了。而且在类被装载进JVM的时候，它的静态方法就会被执行。我们来看com.mysql.jdbc.Driver的实现代码。在它的实现里有这么一段代码：
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static {  
    try {  
        java.sql.DriverManager.registerDriver(new Driver());  
    } catch (SQLException E) {  
        throw new RuntimeException("Can't register driver!");  
    }  
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 很明显，这里使用了DriverManager并将该类给注册上去了。所以，对于任何实现前面Driver接口的类，只要在他们被装载进JVM的时候注册DriverManager就可以实现被后续程序使用。

作为那些被加载的Driver实现，他们本身在被装载时会在执行的static代码段里通过调用DriverManager.registerDriver()来把自身注册到DriverManager的registeredDrivers列表中。这样后面就可以通过得到的Driver来取得连接了。



### 获取数据库连接

在上面例子中使用 JDBC 时，我们都是通过 DriverManager 的接口方法获取数据库连接。我们来看看UnpooledDataSource是如何获取的。

**UnpooledDataSource**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Connection getConnection() throws SQLException {
    return doGetConnection(username, password);
}
    
private Connection doGetConnection(String username, String password) throws SQLException {
    Properties props = new Properties();
    if (driverProperties != null) {
        props.putAll(driverProperties);
    }
    if (username != null) {
        // 存储 user 配置
        props.setProperty("user", username);
    }
    if (password != null) {
        // 存储 password 配置
        props.setProperty("password", password);
    }
    // 调用重载方法
    return doGetConnection(props);
}

private Connection doGetConnection(Properties properties) throws SQLException {
    // 初始化驱动,我们上一节已经讲过了，只用初始化一次
    initializeDriver();
    // 获取连接
    Connection connection = DriverManager.getConnection(url, properties);
    // 配置连接，包括自动提交以及事务等级
    configureConnection(connection);
    return connection;
}

private void configureConnection(Connection conn) throws SQLException {
    if (autoCommit != null && autoCommit != conn.getAutoCommit()) {
        // 设置自动提交
        conn.setAutoCommit(autoCommit);
    }
    if (defaultTransactionIsolationLevel != null) {
        // 设置事务隔离级别
        conn.setTransactionIsolation(defaultTransactionIsolationLevel);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面方法将一些配置信息放入到 Properties 对象中，然后将数据库连接和 Properties 对象传给 DriverManager 的 getConnection 方法即可获取到数据库连接。我们来看看是怎么获取数据库连接的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static Connection getConnection(String url, java.util.Properties info, Class<?> caller) throws SQLException {
    // 获取类加载器
    ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;
    synchronized(DriverManager.class) {
      if (callerCL == null) {
        callerCL = Thread.currentThread().getContextClassLoader();
      }
    }
    // 此处省略部分代码 
    // 这里遍历的是在registerDriver(Driver driver)方法中注册的驱动对象
    // 每个DriverInfo包含了驱动对象和其信息
    for(DriverInfo aDriver : registeredDrivers) {

      // 判断是否为当前线程类加载器加载的驱动类
      if(isDriverAllowed(aDriver.driver, callerCL)) {
        try {
          println("trying " + aDriver.driver.getClass().getName());

          // 获取连接对象，这里调用了Driver的父类的方法
          // 如果这里有多个DriverInfo，比喻Mysql和Oracle的Driver都注册registeredDrivers了
          // 这里所有的Driver都会尝试使用url和info去连接，哪个连接上了就返回
          // 会不会所有的都会连接上呢？不会，因为url的写法不同，不同的Driver会判断url是否适合当前驱动
          Connection con = aDriver.driver.connect(url, info);
          if (con != null) {
            // 打印连接成功信息
            println("getConnection returning " + aDriver.driver.getClass().getName());
            // 返回连接对像
            return (con);
          }
        } catch (SQLException ex) {
          if (reason == null) {
            reason = ex;
          }
        }
      } else {
        println("    skipping: " + aDriver.getClass().getName());
      }
    }  
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

代码中循环所有注册的驱动，然后通过驱动进行连接，所有的驱动都会尝试连接，但是不同的驱动，连接的URL是不同的，如Mysql的url是**jdbc:mysql://localhost:3306/chenhao，以\**jdbc:mysql://开头，则其Mysql的驱动肯定会判断获取连接的url符合，Oracle的也类似，我们来看看Mysql的驱动获取连接\****

***\*![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191029164438415-949838728.png)\****

由于篇幅原因，我这里就不分析了，大家有兴趣的可以看看，最后由URL对应的驱动获取到Connection返回，好了我们再来看看下一种DataSource

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11758412.html#_labelTop)

## PooledDataSource

PooledDataSource 内部实现了连接池功能，用于复用数据库连接。因此，从效率上来说，PooledDataSource 要高于 UnpooledDataSource。但是最终获取Connection还是通过UnpooledDataSource，只不过PooledDataSource 提供一个存储Connection的功能。



### 辅助类介绍

PooledDataSource 需要借助两个辅助类帮其完成功能，这两个辅助类分别是 PoolState 和 PooledConnection。PoolState 用于记录连接池运行时的状态，比如连接获取次数，无效连接数量等。同时 PoolState 内部定义了两个 PooledConnection 集合，用于存储空闲连接和活跃连接。PooledConnection 内部定义了一个 Connection 类型的变量，用于指向真实的数据库连接。以及一个 Connection 的代理类，用于对部分方法调用进行拦截。至于为什么要拦截，随后将进行分析。除此之外，PooledConnection 内部也定义了一些字段，用于记录数据库连接的一些运行时状态。接下来，我们来看一下 PooledConnection 的定义。

**PooledConnection**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class PooledConnection implements InvocationHandler {

    private static final String CLOSE = "close";
    private static final Class<?>[] IFACES = new Class<?>[]{Connection.class};

    private final int hashCode;
    private final PooledDataSource dataSource;
    // 真实的数据库连接
    private final Connection realConnection;
    // 数据库连接代理
    private final Connection proxyConnection;
    
    // 从连接池中取出连接时的时间戳
    private long checkoutTimestamp;
    // 数据库连接创建时间
    private long createdTimestamp;
    // 数据库连接最后使用时间
    private long lastUsedTimestamp;
    // connectionTypeCode = (url + username + password).hashCode()
    private int connectionTypeCode;
    // 表示连接是否有效
    private boolean valid;

    public PooledConnection(Connection connection, PooledDataSource dataSource) {
        this.hashCode = connection.hashCode();
        this.realConnection = connection;
        this.dataSource = dataSource;
        this.createdTimestamp = System.currentTimeMillis();
        this.lastUsedTimestamp = System.currentTimeMillis();
        this.valid = true;
        // 创建 Connection 的代理类对象
        this.proxyConnection = (Connection) Proxy.newProxyInstance(Connection.class.getClassLoader(), IFACES, this);
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {...}
    
    // 省略部分代码
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面再来看看 PoolState 的定义。

**PoolState** 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class PoolState {

    protected PooledDataSource dataSource;

    // 空闲连接列表
    protected final List<PooledConnection> idleConnections = new ArrayList<PooledConnection>();
    // 活跃连接列表
    protected final List<PooledConnection> activeConnections = new ArrayList<PooledConnection>();
    // 从连接池中获取连接的次数
    protected long requestCount = 0;
    // 请求连接总耗时（单位：毫秒）
    protected long accumulatedRequestTime = 0;
    // 连接执行时间总耗时
    protected long accumulatedCheckoutTime = 0;
    // 执行时间超时的连接数
    protected long claimedOverdueConnectionCount = 0;
    // 超时时间累加值
    protected long accumulatedCheckoutTimeOfOverdueConnections = 0;
    // 等待时间累加值
    protected long accumulatedWaitTime = 0;
    // 等待次数
    protected long hadToWaitCount = 0;
    // 无效连接数
    protected long badConnectionCount = 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

大家记住上面的空闲连接列表和活跃连接列表



### 获取连接

前面已经说过，PooledDataSource 会将用过的连接进行回收，以便可以复用连接。因此从 PooledDataSource 获取连接时，如果空闲链接列表里有连接时，可直接取用。那如果没有空闲连接怎么办呢？此时有两种解决办法，要么创建新连接，要么等待其他连接完成任务。

**PooledDataSource**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class PooledDataSource implements DataSource {
    private static final Log log = LogFactory.getLog(PooledDataSource.class);
    //这里有辅助类PoolState
    private final PoolState state = new PoolState(this);
    //还有一个UnpooledDataSource属性，其实真正获取Connection是由UnpooledDataSource来完成的
    private final UnpooledDataSource dataSource;
    protected int poolMaximumActiveConnections = 10;
    protected int poolMaximumIdleConnections = 5;
    protected int poolMaximumCheckoutTime = 20000;
    protected int poolTimeToWait = 20000;
    protected String poolPingQuery = "NO PING QUERY SET";
    protected boolean poolPingEnabled = false;
    protected int poolPingConnectionsNotUsedFor = 0;
    private int expectedConnectionTypeCode;
    
    public PooledDataSource() {
        this.dataSource = new UnpooledDataSource();
    }
    
    public PooledDataSource(String driver, String url, String username, String password) {
        //构造器中创建UnpooledDataSource对象
        this.dataSource = new UnpooledDataSource(driver, url, username, password);
    }
    
    public Connection getConnection() throws SQLException {
        return this.popConnection(this.dataSource.getUsername(), this.dataSource.getPassword()).getProxyConnection();
    }
    
    private PooledConnection popConnection(String username, String password) throws SQLException {
        boolean countedWait = false;
        PooledConnection conn = null;
        long t = System.currentTimeMillis();
        int localBadConnectionCount = 0;

        while (conn == null) {
            synchronized (state) {
                // 检测空闲连接集合（idleConnections）是否为空
                if (!state.idleConnections.isEmpty()) {
                    // idleConnections 不为空，表示有空闲连接可以使用，直接从空闲连接集合中取出一个连接
                    conn = state.idleConnections.remove(0);
                } else {
                    /*
                     * 暂无空闲连接可用，但如果活跃连接数还未超出限制
                     *（poolMaximumActiveConnections），则可创建新的连接
                     */
                    if (state.activeConnections.size() < poolMaximumActiveConnections) {
                        // 创建新连接，看到没，还是通过dataSource获取连接，也就是UnpooledDataSource获取连接
                        conn = new PooledConnection(dataSource.getConnection(), this);
                    } else {    // 连接池已满，不能创建新连接
                        // 取出运行时间最长的连接
                        PooledConnection oldestActiveConnection = state.activeConnections.get(0);
                        // 获取运行时长
                        long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
                        // 检测运行时长是否超出限制，即超时
                        if (longestCheckoutTime > poolMaximumCheckoutTime) {
                            // 累加超时相关的统计字段
                            state.claimedOverdueConnectionCount++;
                            state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
                            state.accumulatedCheckoutTime += longestCheckoutTime;

                            // 从活跃连接集合中移除超时连接
                            state.activeConnections.remove(oldestActiveConnection);
                            // 若连接未设置自动提交，此处进行回滚操作
                            if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                                try {
                                    oldestActiveConnection.getRealConnection().rollback();
                                } catch (SQLException e) {...}
                            }
                            /*
                             * 创建一个新的 PooledConnection，注意，
                             * 此处复用 oldestActiveConnection 的 realConnection 变量
                             */
                            conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
                            /*
                             * 复用 oldestActiveConnection 的一些信息，注意 PooledConnection 中的 
                             * createdTimestamp 用于记录 Connection 的创建时间，而非 PooledConnection 
                             * 的创建时间。所以这里要复用原连接的时间信息。
                             */
                            conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
                            conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());

                            // 设置连接为无效状态
                            oldestActiveConnection.invalidate();
                            
                        } else {// 运行时间最长的连接并未超时
                            try {
                                if (!countedWait) {
                                    state.hadToWaitCount++;
                                    countedWait = true;
                                }
                                long wt = System.currentTimeMillis();
                                // 当前线程进入等待状态
                                state.wait(poolTimeToWait);
                                state.accumulatedWaitTime += System.currentTimeMillis() - wt;
                            } catch (InterruptedException e) {
                                break;
                            }
                        }
                    }
                }
                if (conn != null) {
                    if (conn.isValid()) {
                        if (!conn.getRealConnection().getAutoCommit()) {
                            // 进行回滚操作
                            conn.getRealConnection().rollback();
                        }
                        conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));
                        // 设置统计字段
                        conn.setCheckoutTimestamp(System.currentTimeMillis());
                        conn.setLastUsedTimestamp(System.currentTimeMillis());
                        state.activeConnections.add(conn);
                        state.requestCount++;
                        state.accumulatedRequestTime += System.currentTimeMillis() - t;
                    } else {
                        // 连接无效，此时累加无效连接相关的统计字段
                        state.badConnectionCount++;
                        localBadConnectionCount++;
                        conn = null;
                        if (localBadConnectionCount > (poolMaximumIdleConnections
                            + poolMaximumLocalBadConnectionTolerance)) {
                            throw new SQLException(...);
                        }
                    }
                }
            }

        }
        if (conn == null) {
            throw new SQLException(...);
        }

        return conn;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从连接池中获取连接首先会遇到两种情况：

1. 连接池中有空闲连接
2. 连接池中无空闲连接

对于第一种情况，把连接取出返回即可。对于第二种情况，则要进行细分，会有如下的情况。

1. 活跃连接数没有超出最大活跃连接数
2. 活跃连接数超出最大活跃连接数

对于上面两种情况，第一种情况比较好处理，直接创建新的连接即可。至于第二种情况，需要再次进行细分。

1. 活跃连接的运行时间超出限制，即超时了
2. 活跃连接未超时

对于第一种情况，我们直接将超时连接强行中断，并进行回滚，然后复用部分字段重新创建 PooledConnection 即可。对于第二种情况，目前没有更好的处理方式了，只能等待了。



### 回收连接

相比于获取连接，回收连接的逻辑要简单的多。回收连接成功与否只取决于空闲连接集合的状态，所需处理情况很少，因此比较简单。

我们还是来看看

```
public Connection getConnection() throws SQLException {
    return this.popConnection(this.dataSource.getUsername(), this.dataSource.getPassword()).getProxyConnection();
}
```

返回的是PooledConnection的一个代理类，为什么不直接使用PooledConnection的realConnection呢？我们可以看下PooledConnection这个类

```
class PooledConnection implements InvocationHandler {
```

 很熟悉是吧，标准的代理类用法，看下其invoke方法

**PooledConnection**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String methodName = method.getName();
    // 重点在这里，如果调用了其close方法，则实际执行的是将连接放回连接池的操作
    if (CLOSE.hashCode() == methodName.hashCode() && CLOSE.equals(methodName)) {
        dataSource.pushConnection(this);
        return null;
    } else {
        try {
            if (!Object.class.equals(method.getDeclaringClass())) {
                // issue #579 toString() should never fail
                // throw an SQLException instead of a Runtime
                checkConnection();
            }
            // 其他的操作都交给realConnection执行
            return method.invoke(realConnection, args);
        } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

那我们来看看**pushConnection**做了什么

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void pushConnection(PooledConnection conn) throws SQLException {
    synchronized (state) {
        // 从活跃连接池中移除连接
        state.activeConnections.remove(conn);
        if (conn.isValid()) {
            // 空闲连接集合未满
            if (state.idleConnections.size() < poolMaximumIdleConnections
                && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {
                state.accumulatedCheckoutTime += conn.getCheckoutTime();

                // 回滚未提交的事务
                if (!conn.getRealConnection().getAutoCommit()) {
                    conn.getRealConnection().rollback();
                }

                // 创建新的 PooledConnection
                PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);
                state.idleConnections.add(newConn);
                // 复用时间信息
                newConn.setCreatedTimestamp(conn.getCreatedTimestamp());
                newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());

                // 将原连接置为无效状态
                conn.invalidate();

                // 通知等待的线程
                state.notifyAll();
                
            } else {// 空闲连接集合已满
                state.accumulatedCheckoutTime += conn.getCheckoutTime();
                // 回滚未提交的事务
                if (!conn.getRealConnection().getAutoCommit()) {
                    conn.getRealConnection().rollback();
                }

                // 关闭数据库连接
                conn.getRealConnection().close();
                conn.invalidate();
            }
        } else {
            state.badConnectionCount++;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先将连接从活跃连接集合中移除，如果空闲集合未满，此时复用原连接的字段信息创建新的连接，并将其放入空闲集合中即可；若空闲集合已满，此时无需回收连接，直接关闭即可。

连接池总觉得很神秘，但仔细分析完其代码之后，也就没那么神秘了，就是将连接使用完之后放到一个集合中，下面再获取连接的时候首先从这个集合中获取。 还有PooledConnection的代理模式的使用，值得我们学习

好了，我们已经获取到了数据库连接，接下来要创建**PrepareStatement**了，我们上面JDBC的例子是怎么获取的？ **psmt = conn.prepareStatement(sql);，直接通过Connection来获取，并且把sql传进去了**，我们看看Mybaits中是怎么创建PrepareStatement的

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11758412.html#_labelTop)

## ***\*创建PreparedStatement\**** 

**PreparedStatementHandler**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
stmt = handler.prepare(connection, transaction.getTimeout());

public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
    Statement statement = null;
    try {
        // 创建 Statement
        statement = instantiateStatement(connection);
        // 设置超时和 FetchSize
        setStatementTimeout(statement, transactionTimeout);
        setFetchSize(statement);
        return statement;
    } catch (SQLException e) {
        closeStatement(statement);
        throw e;
    } catch (Exception e) {
        closeStatement(statement);
        throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
    }
}

protected Statement instantiateStatement(Connection connection) throws SQLException {
    //获取sql字符串，比如"select * from user where id= ?"
    String sql = boundSql.getSql();
    // 根据条件调用不同的 prepareStatement 方法创建 PreparedStatement
    if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
        String[] keyColumnNames = mappedStatement.getKeyColumns();
        if (keyColumnNames == null) {
            //通过connection获取Statement，将sql语句传进去
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
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看到没和jdbc的形式一模一样，我们具体来看看**connection.prepareStatement**做了什么

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
 2        
 3     boolean canServerPrepare = true;
 4 
 5     String nativeSql = getProcessEscapeCodesForPrepStmts() ? nativeSQL(sql) : sql;
 6 
 7     if (this.useServerPreparedStmts && getEmulateUnsupportedPstmts()) {
 8         canServerPrepare = canHandleAsServerPreparedStatement(nativeSql);
 9     }
10 
11     if (this.useServerPreparedStmts && getEmulateUnsupportedPstmts()) {
12         canServerPrepare = canHandleAsServerPreparedStatement(nativeSql);
13     }
14 
15     if (this.useServerPreparedStmts && canServerPrepare) {
16         if (this.getCachePreparedStatements()) {
17             ......
18         } else {
19             try {
20                 //这里使用的是ServerPreparedStatement创建PreparedStatement
21                 pStmt = ServerPreparedStatement.getInstance(getMultiHostSafeProxy(), nativeSql, this.database, resultSetType, resultSetConcurrency);
22 
23                 pStmt.setResultSetType(resultSetType);
24                 pStmt.setResultSetConcurrency(resultSetConcurrency);
25             } catch (SQLException sqlEx) {
26                 // Punt, if necessary
27                 if (getEmulateUnsupportedPstmts()) {
28                     pStmt = (PreparedStatement) clientPrepareStatement(nativeSql, resultSetType, resultSetConcurrency, false);
29                 } else {
30                     throw sqlEx;
31                 }
32             }
33         }
34     } else {
35         pStmt = (PreparedStatement) clientPrepareStatement(nativeSql, resultSetType, resultSetConcurrency, false);
36     }
37 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们只用看最关键的第21行代码，使用**ServerPreparedStatement的getInstance返回一个**PreparedStatement，其实本质上**ServerPreparedStatement继承了PreparedStatement对象**，我们看看其构造方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected ServerPreparedStatement(ConnectionImpl conn, String sql, String catalog, int resultSetType, int resultSetConcurrency) throws SQLException {
    //略...

    try {
        this.serverPrepare(sql);
    } catch (SQLException var10) {
        this.realClose(false, true);
        throw var10;
    } catch (Exception var11) {
        this.realClose(false, true);
        SQLException sqlEx = SQLError.createSQLException(var11.toString(), "S1000", this.getExceptionInterceptor());
        sqlEx.initCause(var11);
        throw sqlEx;
    }
    //略...

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

继续调用**this****.serverPrepare(sql);**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class ServerPreparedStatement extends PreparedStatement {
    //存放运行时参数的数组
    private ServerPreparedStatement.BindValue[] parameterBindings;
    //服务器预编译好的sql语句返回的serverStatementId
    private long serverStatementId;
    private void serverPrepare(String sql) throws SQLException {
        synchronized(this.connection.getMutex()) {
            MysqlIO mysql = this.connection.getIO();
            try {
                //向sql服务器发送了一条PREPARE指令
                Buffer prepareResultPacket = mysql.sendCommand(MysqlDefs.COM_PREPARE, sql, (Buffer)null, false, characterEncoding, 0);
                //记录下了预编译好的sql语句所对应的serverStatementId
                this.serverStatementId = prepareResultPacket.readLong();
                this.fieldCount = prepareResultPacket.readInt();
                //获取参数个数，比喻 select * from user where id= ？and name = ？，其中有两个？，则这里返回的参数个数应该为2
                this.parameterCount = prepareResultPacket.readInt();
                this.parameterBindings = new ServerPreparedStatement.BindValue[this.parameterCount];

                for(int i = 0; i < this.parameterCount; ++i) {
                    //根据参数个数，初始化数组
                    this.parameterBindings[i] = new ServerPreparedStatement.BindValue();
                }

            } catch (SQLException var16) {
                throw sqlEx;
            } finally {
                this.connection.getIO().clearInputStream();
            }

        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ServerPreparedStatement继承PreparedStatement，ServerPreparedStatement初始化的时候就向sql服务器发送了一条PREPARE指令，把SQL语句传到mysql服务器，如select * from user where id= ？and name = ？，mysql服务器会对sql进行编译，并保存在服务器，返回预编译语句对应的id，并保存在
ServerPreparedStatement中，同时创建BindValue[] parameterBindings数组，后面设置参数就直接添加到此数组中。好了，此时我们创建了一个ServerPreparedStatement并返回，下面就是设置运行时参数了
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11758412.html#_labelTop)

## 设置运行时参数到 SQL 中



### 我们已经获取到了PreparedStatement，接下来就是将运行时参数设置到PreparedStatement中，如下代码

```
handler.parameterize(stmt);
```

JDBC是怎么设置的呢？我们看看上面的例子，很简单吧

```
psmt = conn.prepareStatement(sql);
//设置参数
psmt.setString(1, username);
psmt.setString(2, password);
```

我们来看看parameterize方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void parameterize(Statement statement) throws SQLException {
    // 通过参数处理器 ParameterHandler 设置运行时参数到 PreparedStatement 中
    parameterHandler.setParameters((PreparedStatement) statement);
}

public class DefaultParameterHandler implements ParameterHandler {
    private final TypeHandlerRegistry typeHandlerRegistry;
    private final MappedStatement mappedStatement;
    private final Object parameterObject;
    private final BoundSql boundSql;
    private final Configuration configuration;

    public void setParameters(PreparedStatement ps) {
        /*
         * 从 BoundSql 中获取 ParameterMapping 列表，每个 ParameterMapping 与原始 SQL 中的 #{xxx} 占位符一一对应
         */
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        if (parameterMappings != null) {
            for (int i = 0; i < parameterMappings.size(); i++) {
                ParameterMapping parameterMapping = parameterMappings.get(i);
                if (parameterMapping.getMode() != ParameterMode.OUT) {
                    Object value;
                    // 获取属性名
                    String propertyName = parameterMapping.getProperty();
                    if (boundSql.hasAdditionalParameter(propertyName)) {
                        value = boundSql.getAdditionalParameter(propertyName);
                    } else if (parameterObject == null) {
                        value = null;
                    } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                        value = parameterObject;
                    } else {
                        // 为用户传入的参数 parameterObject 创建元信息对象
                        MetaObject metaObject = configuration.newMetaObject(parameterObject);
                        // 从用户传入的参数中获取 propertyName 对应的值
                        value = metaObject.getValue(propertyName);
                    }

                    TypeHandler typeHandler = parameterMapping.getTypeHandler();
                    JdbcType jdbcType = parameterMapping.getJdbcType();
                    if (value == null && jdbcType == null) {
                        jdbcType = configuration.getJdbcTypeForNull();
                    }
                    try {
                        // 由类型处理器 typeHandler 向 ParameterHandler 设置参数
                        typeHandler.setParameter(ps, i + 1, value, jdbcType);
                    } catch (TypeException e) {
                        throw new TypeException(...);
                    } catch (SQLException e) {
                        throw new TypeException(...);
                    }
                }
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先从**boundSql中获取****parameterMappings 集合，这块大家可以看看我前面的文章，然后遍历获取** **parameterMapping中的****propertyName ，如**#{name} 中的name,然后从运行时参数**parameterObject中获取name对应的参数值，最后设置到**PreparedStatement 中，我们主要来看是如何设置参数的。也就是

typeHandler.setParameter(ps, i + 1, value, jdbcType);，这句代码最终会向我们例子中一样执行，如下

```
public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
}
```

还记得我们的PreparedStatement是什么吗？是**ServerPreparedStatement，那我们就来看看\**ServerPreparedStatement的\******setString方法**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void setString(int parameterIndex, String x) throws SQLException {
    this.checkClosed();
    if (x == null) {
        this.setNull(parameterIndex, 1);
    } else {
        //根据参数下标从parameterBindings数组总获取BindValue
        ServerPreparedStatement.BindValue binding = this.getBinding(parameterIndex, false);
        this.setType(binding, this.stringTypeCode);
        //设置参数值
        binding.value = x;
        binding.isNull = false;
        binding.isLongData = false;
    }

}

protected ServerPreparedStatement.BindValue getBinding(int parameterIndex, boolean forLongData) throws SQLException {
    this.checkClosed();
    if (this.parameterBindings.length == 0) {
        throw SQLError.createSQLException(Messages.getString("ServerPreparedStatement.8"), "S1009", this.getExceptionInterceptor());
    } else {
        --parameterIndex;
        if (parameterIndex >= 0 && parameterIndex < this.parameterBindings.length) {
            if (this.parameterBindings[parameterIndex] == null) {
                this.parameterBindings[parameterIndex] = new ServerPreparedStatement.BindValue();
            } else if (this.parameterBindings[parameterIndex].isLongData && !forLongData) {
                this.detectedLongParameterSwitch = true;
            }

            this.parameterBindings[parameterIndex].isSet = true;
            this.parameterBindings[parameterIndex].boundBeforeExecutionNum = (long)this.numberOfExecutions;
            //根据参数下标从parameterBindings数组总获取BindValue
            return this.parameterBindings[parameterIndex];
        } else {
            throw SQLError.createSQLException(Messages.getString("ServerPreparedStatement.9") + (parameterIndex + 1) + Messages.getString("ServerPreparedStatement.10") + this.parameterBindings.length, "S1009", this.getExceptionInterceptor());
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
就是根据参数下标从ServerPreparedStatement的参数数组parameterBindings中获取BindValue对象，然后设置值，好了现在ServerPreparedStatement包含了预编译SQL语句的Id和参数数组，最后一步便是执行SQL了。
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11758412.html#_labelTop)

## 执行查询

执行查询操作就是我们文章开头的最后一行代码，如下

```
return handler.<E>query(stmt, resultHandler);
```

我们来看看query是怎么做的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    PreparedStatement ps = (PreparedStatement)statement;
    //直接执行ServerPreparedStatement的execute方法
    ps.execute();
    return this.resultSetHandler.handleResultSets(ps);
}

public boolean execute() throws SQLException {
    this.checkClosed();
    ConnectionImpl locallyScopedConn = this.connection;
    if (!this.checkReadOnlySafeStatement()) {
        throw SQLError.createSQLException(Messages.getString("PreparedStatement.20") + Messages.getString("PreparedStatement.21"), "S1009", this.getExceptionInterceptor());
    } else {
        ResultSetInternalMethods rs = null;
        CachedResultSetMetaData cachedMetadata = null;
        synchronized(locallyScopedConn.getMutex()) {
            //略....
            rs = this.executeInternal(rowLimit, sendPacket, doStreaming, this.firstCharOfStmt == 'S', metadataFromCache, false);
            //略....
        }

        return rs != null && rs.reallyResult();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

省略了很多代码，只看最关键的**executeInternal**

**ServerPreparedStatement**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected ResultSetInternalMethods executeInternal(int maxRowsToRetrieve, Buffer sendPacket, boolean createStreamingResultSet, boolean queryIsSelectOnly, Field[] metadataFromCache, boolean isBatch) throws SQLException {
    try {
        return this.serverExecute(maxRowsToRetrieve, createStreamingResultSet, metadataFromCache);
    } catch (SQLException var11) {
        throw sqlEx;
    } 
}

private ResultSetInternalMethods serverExecute(int maxRowsToRetrieve, boolean createStreamingResultSet, Field[] metadataFromCache) throws SQLException {
    synchronized(this.connection.getMutex()) {
        //略....
        MysqlIO mysql = this.connection.getIO();
        Buffer packet = mysql.getSharedSendPacket();
        packet.clear();
        packet.writeByte((byte)MysqlDefs.COM_EXECUTE);
        //将该语句对应的id写入数据包
        packet.writeLong(this.serverStatementId);

        int i;
        //将对应的参数写入数据包
        for(i = 0; i < this.parameterCount; ++i) {
            if (!this.parameterBindings[i].isLongData) {
                if (!this.parameterBindings[i].isNull) {
                    this.storeBinding(packet, this.parameterBindings[i], mysql);
                } else {
                    nullBitsBuffer[i / 8] = (byte)(nullBitsBuffer[i / 8] | 1 << (i & 7));
                }
            }
        }
        //发送数据包,表示执行id对应的预编译sql
        Buffer resultPacket = mysql.sendCommand(MysqlDefs.COM_EXECUTE, (String)null, packet, false, (String)null, 0);
        //略....
        ResultSetImpl rs = mysql.readAllResults(this,  this.resultSetType,  resultPacket, true, (long)this.fieldCount, metadataFromCache);
        //返回结果
        return rs;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ServerPreparedStatement在记录下serverStatementId后，对于相同SQL模板的操作，每次只是发送serverStatementId和对应的参数，省去了编译sql的过程。 至此我们的已经从数据库拿到了查询结果，但是结果是ResultSetImpl类型，我们还需要将返回结果转化成我们的java对象呢，留在下一篇来讲吧

# [Mybaits 源码解析 （八）----- 结果集 ResultSet 自动映射成实体类对象（上篇）](https://www.cnblogs.com/java-chen-hao/p/11760777.html)

**正文**

上一篇文章我们已经将SQL发送到了数据库，并返回了ResultSet，接下来就是将结果集 ResultSet 自动映射成实体类对象。这样使用者就无需再手动操作结果集，并将数据填充到实体类对象中。这可大大降低开发的工作量，提高工作效率。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11760777.html#_labelTop)

## 映射结果入口

我们来看看上次看源码的位置

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    PreparedStatement ps = (PreparedStatement)statement;
    //执行数据库SQL
    ps.execute();
    //进行resultSet自动映射
    return this.resultSetHandler.handleResultSets(ps);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结果集的处理入口方法是 handleResultSets

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public List<Object> handleResultSets(Statement stmt) throws SQLException {
    
    final List<Object> multipleResults = new ArrayList<Object>();

    int resultSetCount = 0;
    //获取第一个ResultSet,通常只会有一个
    ResultSetWrapper rsw = getFirstResultSet(stmt);
    //从配置中读取对应的ResultMap，通常也只会有一个，设置多个是通过逗号来分隔，我们平时有这样设置吗？
    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    int resultMapCount = resultMaps.size();
    validateResultMapsCount(rsw, resultMapCount);

    while (rsw != null && resultMapCount > resultSetCount) {
        ResultMap resultMap = resultMaps.get(resultSetCount);
        // 处理结果集
        handleResultSet(rsw, resultMap, multipleResults, null);
        rsw = getNextResultSet(stmt);
        cleanUpAfterHandlingResultSet();
        resultSetCount++;
    }

    // 以下逻辑均与多结果集有关，就不分析了，代码省略
    String[] resultSets = mappedStatement.getResultSets();
    if (resultSets != null) {...}

    return collapseSingleResultList(multipleResults);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在实际运行过程中，通常情况下一个Sql语句只返回一个结果集，对多个结果集的情况不做分析 。实际很少用到。继续看handleResultSet方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void handleResultSet(ResultSetWrapper rsw, ResultMap resultMap, List<Object> multipleResults, ResultMapping parentMapping) throws SQLException {
    try {
        if (parentMapping != null) {
            handleRowValues(rsw, resultMap, null, RowBounds.DEFAULT, parentMapping);
        } else {
            if (resultHandler == null) {
                // 创建默认的结果处理器
                DefaultResultHandler defaultResultHandler = new DefaultResultHandler(objectFactory);
                // 处理结果集的行数据
                handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);
                // 将结果加入multipleResults中
                multipleResults.add(defaultResultHandler.getResultList());
            } else {
                handleRowValues(rsw, resultMap, resultHandler, rowBounds, null);
            }
        }
    } finally {
        closeResultSet(rsw.getResultSet());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过handleRowValues 映射ResultSet结果，最后映射的结果会在defaultResultHandler的ResultList集合中，最后将结果加入到multipleResults中就可以返回了，我们继续跟进**handleRowValues这个核心方法**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler,
        RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
    if (resultMap.hasNestedResultMaps()) {
        ensureNoRowBounds();
        checkResultHandler();
        // 处理嵌套映射，关于嵌套映射我们下一篇文章单独分析
        handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
    } else {
        // 处理简单映射，本文先只分析简单映射
        handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们可以通过resultMap.hasNestedResultMaps()知道查询语句是否是嵌套查询，如果resultMap中包含<association> 和 <collection>且其select属性不为空，则为嵌套查询，大家可以看看我第三篇文章关于[解析 resultMap 节点](https://www.cnblogs.com/java-chen-hao/p/11743442.html#_label2_1)。本文先分析简单的映射

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap,
        ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {

    DefaultResultContext<Object> resultContext = new DefaultResultContext<Object>();
    // 根据 RowBounds 定位到指定行记录
    skipRows(rsw.getResultSet(), rowBounds);
    // ResultSet是一个集合，很有可能我们查询的就是一个List，这就就每条数据遍历处理
    while (shouldProcessMoreRows(resultContext, rowBounds) && rsw.getResultSet().next()) {
        ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(rsw.getResultSet(), resultMap, null);
        // 从 resultSet 中获取结果
        Object rowValue = getRowValue(rsw, discriminatedResultMap);
        // 存储结果到resultHandler的ResultList，最后ResultList加入multipleResults中返回
        storeObject(resultHandler, resultContext, rowValue, parentMapping, rsw.getResultSet());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们查询的结果很有可能是一个集合，所以这里要遍历集合，每条结果单独进行映射，最后映射的结果加入到**resultHandler的ResultList**

**MyBatis 默认提供了 RowBounds 用于分页，从上面的代码中可以看出，这并非是一个高效的分页方式，是查出所有的数据，进行内存分页。除了使用 RowBounds，还可以使用一些第三方分页插件进行分页。**我们后面文章来讲，我们来看关键代码getRowValue，处理一行数据

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap) throws SQLException {
    // 这个Map是用来存储延迟加载的BountSql的，我们下面来看
    final ResultLoaderMap lazyLoader = new ResultLoaderMap();
 // 创建实体类对象，比如 Employ 对象
    Object rowValue = createResultObject(rsw, resultMap, lazyLoader, null);
    if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
        final MetaObject metaObject = configuration.newMetaObject(rowValue);
        boolean foundValues = this.useConstructorMappings;
        
        if (shouldApplyAutomaticMappings(resultMap, false)) {
            //自动映射,结果集中有的column，但resultMap中并没有配置  
            foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, null) || foundValues;
        }
      // 根据 <resultMap> 节点中配置的映射关系进行映射
        foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, null) || foundValues;
        foundValues = lazyLoader.size() > 0 || foundValues;
        rowValue = foundValues || configuration.isReturnInstanceForEmptyRow() ? rowValue : null;
    }
    return rowValue;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

重要的逻辑已经注释出来了。分别如下：

1. 创建实体类对象
2. 自动映射结果集中有的column，但resultMap中并没有配置
3. 根据 <resultMap> 节点中配置的映射关系进行映射

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11760777.html#_labelTop)

## 创建实体类对象

我们想将查询结果映射成实体类对象，第一步当然是要创建实体类对象了，下面我们来看一下 MyBatis 创建实体类对象的过程。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {

    this.useConstructorMappings = false;
    final List<Class<?>> constructorArgTypes = new ArrayList<Class<?>>();
    final List<Object> constructorArgs = new ArrayList<Object>();

    // 调用重载方法创建实体类对象
    Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);
    if (resultObject != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
        final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
        for (ResultMapping propertyMapping : propertyMappings) {
            // 如果开启了延迟加载，则为 resultObject 生成代理类，如果仅仅是配置的关联查询，没有开启延迟加载，是不会创建代理类
            if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {
                /*
                 * 创建代理类，默认使用 Javassist 框架生成代理类。
                 * 由于实体类通常不会实现接口，所以不能使用 JDK 动态代理 API 为实体类生成代理。
                 * 并且将lazyLoader传进去了
                 */
                resultObject = configuration.getProxyFactory()
                    .createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
                break;
            }
        }
    }
    this.useConstructorMappings =
        resultObject != null && !constructorArgTypes.isEmpty();
    return resultObject;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们先来看 createResultObject 重载方法的逻辑

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix) throws SQLException {

    final Class<?> resultType = resultMap.getType();
    final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
    final List<ResultMapping> constructorMappings = resultMap.getConstructorResultMappings();

    if (hasTypeHandlerForResultObject(rsw, resultType)) {
        return createPrimitiveResultObject(rsw, resultMap, columnPrefix);
    } else if (!constructorMappings.isEmpty()) {
        return createParameterizedResultObject(rsw, resultType, constructorMappings, constructorArgTypes, constructorArgs, columnPrefix);
    } else if (resultType.isInterface() || metaType.hasDefaultConstructor()) {
        // 通过 ObjectFactory 调用目标类的默认构造方法创建实例
        return objectFactory.create(resultType);
    } else if (shouldApplyAutomaticMappings(resultMap, false)) {
        return createByConstructorSignature(rsw, resultType, constructorArgTypes, constructorArgs, columnPrefix);
    }
    throw new ExecutorException("Do not know how to create an instance of " + resultType);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

一般情况下，MyBatis 会通过 ObjectFactory 调用默认构造方法创建实体类对象。看看是如何创建的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    Class<?> classToCreate = this.resolveInterface(type);
    return this.instantiateClass(classToCreate, constructorArgTypes, constructorArgs);
}

<T> T instantiateClass(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    try {
        Constructor constructor;
        if (constructorArgTypes != null && constructorArgs != null) {
            constructor = type.getDeclaredConstructor((Class[])constructorArgTypes.toArray(new Class[constructorArgTypes.size()]));
            if (!constructor.isAccessible()) {
                constructor.setAccessible(true);
            }

            return constructor.newInstance(constructorArgs.toArray(new Object[constructorArgs.size()]));
        } else {
            //通过反射获取构造器
            constructor = type.getDeclaredConstructor();
            if (!constructor.isAccessible()) {
                constructor.setAccessible(true);
            }
            //通过构造器来实例化对象
            return constructor.newInstance();
        }
    } catch (Exception var9) {
        throw new ReflectionException("Error instantiating " + type + " with invalid types (" + argTypes + ") or values (" + argValues + "). Cause: " + var9, var9);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

很简单，就是通过反射创建对象

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11760777.html#_labelTop)

## 结果集映射

映射结果集分为两种情况：一种是自动映射(结果集有但在resultMap里没有配置的字段)，在实际应用中，都会使用自动映射，减少配置的工作。自动映射在Mybatis中也是默认开启的。第二种是映射ResultMap中配置的，我们分这两者映射来看



### 自动映射

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private boolean applyAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, String columnPrefix) throws SQLException {

    // 获取 UnMappedColumnAutoMapping 列表
    List<UnMappedColumnAutoMapping> autoMapping = createAutomaticMappings(rsw, resultMap, metaObject, columnPrefix);
    boolean foundValues = false;
    if (!autoMapping.isEmpty()) {
        for (UnMappedColumnAutoMapping mapping : autoMapping) {
            // 通过 TypeHandler 从结果集中获取指定列的数据
            final Object value = mapping.typeHandler.getResult(rsw.getResultSet(), mapping.column);
            if (value != null) {
                foundValues = true;
            }
            if (value != null || (configuration.isCallSettersOnNulls() && !mapping.primitive)) {
                // 通过元信息对象设置 value 到实体类对象的指定字段上
                metaObject.setValue(mapping.property, value);
            }
        }
    }
    return foundValues;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先是获取 UnMappedColumnAutoMapping 集合，然后遍历该集合，并通过 TypeHandler 从结果集中获取数据，最后再将获取到的数据设置到实体类对象中。

UnMappedColumnAutoMapping 用于记录未配置在 <resultMap> 节点中的映射关系。它的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static class UnMappedColumnAutoMapping {

    private final String column;
    private final String property;
    private final TypeHandler<?> typeHandler;
    private final boolean primitive;

    public UnMappedColumnAutoMapping(String column, String property, TypeHandler<?> typeHandler, boolean primitive) {
        this.column = column;
        this.property = property;
        this.typeHandler = typeHandler;
        this.primitive = primitive;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

仅用于记录映射关系。下面看一下获取 UnMappedColumnAutoMapping 集合的过程，如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private List<UnMappedColumnAutoMapping> createAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, String columnPrefix) throws SQLException {

    final String mapKey = resultMap.getId() + ":" + columnPrefix;
    // 从缓存中获取 UnMappedColumnAutoMapping 列表
    List<UnMappedColumnAutoMapping> autoMapping = autoMappingsCache.get(mapKey);
    // 缓存未命中
    if (autoMapping == null) {
        autoMapping = new ArrayList<UnMappedColumnAutoMapping>();
        // 从 ResultSetWrapper 中获取未配置在 <resultMap> 中的列名
        final List<String> unmappedColumnNames = rsw.getUnmappedColumnNames(resultMap, columnPrefix);
        for (String columnName : unmappedColumnNames) {
            String propertyName = columnName;
            if (columnPrefix != null && !columnPrefix.isEmpty()) {
                if (columnName.toUpperCase(Locale.ENGLISH).startsWith(columnPrefix)) {
                    propertyName = columnName.substring(columnPrefix.length());
                } else {
                    continue;
                }
            }
            // 将下划线形式的列名转成驼峰式，比如 AUTHOR_NAME -> authorName
            final String property = metaObject.findProperty(propertyName, configuration.isMapUnderscoreToCamelCase());
            if (property != null && metaObject.hasSetter(property)) {
                // 检测当前属性是否存在于 resultMap 中
                if (resultMap.getMappedProperties().contains(property)) {
                    continue;
                }
                // 获取属性对于的类型
                final Class<?> propertyType = metaObject.getSetterType(property);
                if (typeHandlerRegistry.hasTypeHandler(propertyType, rsw.getJdbcType(columnName))) {
                    final TypeHandler<?> typeHandler = rsw.getTypeHandler(propertyType, columnName);
                    // 封装上面获取到的信息到 UnMappedColumnAutoMapping 对象中
                    autoMapping.add(new UnMappedColumnAutoMapping(columnName, property, typeHandler, propertyType.isPrimitive()));
                } else {
                    configuration.getAutoMappingUnknownColumnBehavior()
                        .doAction(mappedStatement, columnName, property, propertyType);
                }
            } else {
                configuration.getAutoMappingUnknownColumnBehavior()
                    .doAction(mappedStatement, columnName, (property != null) ? property : propertyName, null);
            }
        }
        // 写入缓存
        autoMappingsCache.put(mapKey, autoMapping);
    }
    return autoMapping;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先来看看从 ResultSetWrapper 中获取未配置在 <resultMap> 中的列名

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public List<String> getUnmappedColumnNames(ResultMap resultMap, String columnPrefix) throws SQLException {
    List<String> unMappedColumnNames = unMappedColumnNamesMap.get(getMapKey(resultMap, columnPrefix));
    if (unMappedColumnNames == null) {
        // 加载已映射与未映射列名
        loadMappedAndUnmappedColumnNames(resultMap, columnPrefix);
        // 获取未映射列名
        unMappedColumnNames = unMappedColumnNamesMap.get(getMapKey(resultMap, columnPrefix));
    }
    return unMappedColumnNames;
}

private void loadMappedAndUnmappedColumnNames(ResultMap resultMap, String columnPrefix) throws SQLException {
    List<String> mappedColumnNames = new ArrayList<String>();
    List<String> unmappedColumnNames = new ArrayList<String>();
    final String upperColumnPrefix = columnPrefix == null ? null : columnPrefix.toUpperCase(Locale.ENGLISH);
    // 获取 <resultMap> 中配置的所有列名
    final Set<String> mappedColumns = prependPrefixes(resultMap.getMappedColumns(), upperColumnPrefix);
    /*
     * 遍历 columnNames，columnNames 是 ResultSetWrapper 的成员变量，保存了当前结果集中的所有列名
     * 这里是通过ResultSet中的所有列名来获取没有在resultMap中配置的列名
     * 意思是后面进行自动赋值时，只赋值查出来的列名
     */
    for (String columnName : columnNames) {
        final String upperColumnName = columnName.toUpperCase(Locale.ENGLISH);
        // 检测已映射列名集合中是否包含当前列名
        if (mappedColumns.contains(upperColumnName)) {
            mappedColumnNames.add(upperColumnName);
        } else {
            // 将列名存入 unmappedColumnNames 中
            unmappedColumnNames.add(columnName);
        }
    }
    // 缓存列名集合
    mappedColumnNamesMap.put(getMapKey(resultMap, columnPrefix), mappedColumnNames);
    unMappedColumnNamesMap.put(getMapKey(resultMap, columnPrefix), unmappedColumnNames);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先是从当前数据集中获取列名集合，然后获取 <resultMap> 中配置的列名集合。之后遍历数据集中的列名集合，并判断列名是否被配置在了 <resultMap> 节点中。若配置了，则表明该列名已有映射关系，此时该列名存入 mappedColumnNames 中。若未配置，则表明列名未与实体类的某个字段形成映射关系，此时该列名存入 unmappedColumnNames 中。



### 映射result节点

接下来分析一下 MyBatis 是如何将结果集中的数据填充到已配置ResultMap映射的实体类字段中的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private boolean applyPropertyMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject,ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {
    
    // 获取已映射的列名
    final List<String> mappedColumnNames = rsw.getMappedColumnNames(resultMap, columnPrefix);
    boolean foundValues = false;
    // 获取 ResultMapping集合
    final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
    // 所有的ResultMapping遍历进行映射
    for (ResultMapping propertyMapping : propertyMappings) {
        String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
        if (propertyMapping.getNestedResultMapId() != null) {
            column = null;
        }
        if (propertyMapping.isCompositeResult()
            || (column != null && mappedColumnNames.contains(column.toUpperCase(Locale.ENGLISH)))
            || propertyMapping.getResultSet() != null) {
            
            // 从结果集中获取指定列的数据
            Object value = getPropertyMappingValue(rsw.getResultSet(), metaObject, propertyMapping, lazyLoader, columnPrefix);
            
            final String property = propertyMapping.getProperty();
            if (property == null) {
                continue;

            // 若获取到的值为 DEFERED，则延迟加载该值
            } else if (value == DEFERED) {
                foundValues = true;
                continue;
            }
            if (value != null) {
                foundValues = true;
            }
            if (value != null || (configuration.isCallSettersOnNulls() && !metaObject.getSetterType(property).isPrimitive())) {
                // 将获取到的值设置到实体类对象中
                metaObject.setValue(property, value);
            }
        }
    }
    return foundValues;
}

private Object getPropertyMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping,ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {

    if (propertyMapping.getNestedQueryId() != null) {
        // 获取关联查询结果
        return getNestedQueryMappingValue(rs, metaResultObject, propertyMapping, lazyLoader, columnPrefix);
    } else if (propertyMapping.getResultSet() != null) {
        addPendingChildRelation(rs, metaResultObject, propertyMapping);
        return DEFERED;
    } else {
        final TypeHandler<?> typeHandler = propertyMapping.getTypeHandler();
        final String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
        // 从 ResultSet 中获取指定列的值
        return typeHandler.getResult(rs, column);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从 ResultMap 获取映射对象 ResultMapping 集合。然后遍历 ResultMapping 集合，再此过程中调用 getPropertyMappingValue 获取指定指定列的数据，最后将获取到的数据设置到实体类对象中。

这里和自动映射有一点不同，**自动映射是从直接从ResultSet 中获取指定列的值，但是通过ResultMap多了一种情况，那就是关联查询，也可以说是延迟查询，此关联查询如果没有配置延迟加载，那么就要获取关联查询的值，如果配置了延迟加载，则返回DEFERED**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11760777.html#_labelTop)

## 关联查询与延迟加载

我们的查询经常会碰到一对一，一对多的情况，通常我们可以用一条 SQL 进行多表查询完成任务。当然我们也可以使用关联查询，将一条 SQL 拆成两条去完成查询任务。MyBatis 提供了两个标签用于支持一对一和一对多的使用场景，分别是 <association> 和 <collection>。下面我来演示一下如何使用 <association> 完成一对一的关联查询。先来看看实体类的定义：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/** 作者类 */
public class Author {
    private Integer id;
    private String name;
    private Integer age;
    private Integer sex;
    private String email;
    
    // 省略 getter/setter
}

/** 文章类 */
public class Article {
    private Integer id;
    private String title;
    // 一对一关系
    private Author author;
    private String content;
    private Date createTime;
    
    // 省略 getter/setter
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接下来看一下 Mapper 接口与映射文件的定义。

```
public interface ArticleDao {
    Article findOne(@Param("id") int id);
    Author findAuthor(@Param("id") int authorId);
}
```

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<mapper namespace="xyz.coolblog.dao.ArticleDao">
    <resultMap id="articleResult" type="Article">
        <result property="createTime" column="create_time"/>
        //column 属性值仅包含列信息，参数类型为 author_id 列对应的类型，这里为 Integer
        //意思是将author_id做为参数传给关联的查询语句findAuthor
        <association property="author" column="author_id" javaType="Author" select="findAuthor"/>
    </resultMap>

    <select id="findOne" resultMap="articleResult">
        SELECT
            id, author_id, title, content, create_time
        FROM
            article
        WHERE
            id = #{id}
    </select>

    <select id="findAuthor" resultType="Author">
        SELECT
            id, name, age, sex, email
        FROM
            author
        WHERE
            id = #{id}
    </select>
</mapper>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**开启延迟加载**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!-- 开启延迟加载 -->
<setting name="lazyLoadingEnabled" value="true"/>
<!-- 关闭积极的加载策略 -->
<setting name="aggressiveLazyLoading" value="false"/>
<!-- 延迟加载的触发方法 -->
<setting name="lazyLoadTriggerMethods" value="equals,hashCode"/>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**此时association节点使用了select指向另外一个查询语句，并且将 author_id作为参数传给关联查询的语句**

**此时如果不开启延迟加载，那么会生成两条SQL，先执行findOne，然后通过findOne的返回结果做为参数，执行findAuthor语句，并将结果设置到****author属性**

**如果开启了延迟加载呢？那么只会执行findOne一条SQL，当调用article.getAuthor()方法时，才会去执行findAuthor进行查询，我们下面来看看是如何实现的**

我们还是要从上面映射result节点说起

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object getPropertyMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping,ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {

    if (propertyMapping.getNestedQueryId() != null) {
        // 获取关联查询结果
        return getNestedQueryMappingValue(rs, metaResultObject, propertyMapping, lazyLoader, columnPrefix);
    } else if (propertyMapping.getResultSet() != null) {
        addPendingChildRelation(rs, metaResultObject, propertyMapping);
        return DEFERED;
    } else {
        final TypeHandler<?> typeHandler = propertyMapping.getTypeHandler();
        final String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
        // 从 ResultSet 中获取指定列的值
        return typeHandler.getResult(rs, column);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到，如果ResultMapping设置了关联查询，也就是**association或者collection配置了select，那么就要通过关联语句来查询结果，并设置到实体类对象的属性中了。如果没配置select，那就简单，直接从ResultSet中通过列名获取结果。**那我们来看看getNestedQueryMappingValue

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object getNestedQueryMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {

    // 获取关联查询 id，id = 命名空间 + <association> 的 select 属性值
    final String nestedQueryId = propertyMapping.getNestedQueryId();
    final String property = propertyMapping.getProperty();
    // 根据 nestedQueryId 获取关联的 MappedStatement
    final MappedStatement nestedQuery = configuration.getMappedStatement(nestedQueryId);
    //获取关联查询MappedStatement的参数类型
    final Class<?> nestedQueryParameterType = nestedQuery.getParameterMap().getType();
    /*
     * 生成关联查询语句参数对象，参数类型可能是一些包装类，Map 或是自定义的实体类，
     * 具体类型取决于配置信息。以上面的例子为基础，下面分析不同配置对参数类型的影响：
     *   1. <association column="author_id"> 
     *      column 属性值仅包含列信息，参数类型为 author_id 列对应的类型，这里为 Integer
     * 
     *   2. <association column="{id=author_id, name=title}"> 
     *      column 属性值包含了属性名与列名的复合信息，MyBatis 会根据列名从 ResultSet 中
     *      获取列数据，并将列数据设置到实体类对象的指定属性中，比如：
     *          Author{id=1, name="陈浩"}
     *      或是以键值对 <属性, 列数据> 的形式，将两者存入 Map 中。比如：
     *          {"id": 1, "name": "陈浩"}
     *
     *      至于参数类型到底为实体类还是 Map，取决于关联查询语句的配置信息。比如：
     *          <select id="findAuthor">  ->  参数类型为 Map
     *          <select id="findAuthor" parameterType="Author"> -> 参数类型为实体类
     */
    final Object nestedQueryParameterObject = prepareParameterForNestedQuery(rs, propertyMapping, nestedQueryParameterType, columnPrefix);
    Object value = null;
    if (nestedQueryParameterObject != null) {
        // 获取 BoundSql，这里设置了运行时参数，所以这里是能直接执行的
        final BoundSql nestedBoundSql = nestedQuery.getBoundSql(nestedQueryParameterObject);
        final CacheKey key = executor.createCacheKey(nestedQuery, nestedQueryParameterObject, RowBounds.DEFAULT, nestedBoundSql);
        final Class<?> targetType = propertyMapping.getJavaType();

        if (executor.isCached(nestedQuery, key)) {
            executor.deferLoad(nestedQuery, metaResultObject, property, key, targetType);
            value = DEFERED;
        } else {
            // 创建结果加载器
            final ResultLoader resultLoader = new ResultLoader(configuration, executor, nestedQuery, nestedQueryParameterObject, targetType, key, nestedBoundSql);
            // 检测当前属性是否需要延迟加载
            if (propertyMapping.isLazy()) {
                // 添加延迟加载相关的对象到 loaderMap 集合中
                lazyLoader.addLoader(property, metaResultObject, resultLoader);
                value = DEFERED;
            } else {
                // 直接执行关联查询
                // 如果只是配置关联查询，但是没有开启懒加载，则直接执行关联查询，并返回结果，设置到实体类对象的属性中
                value = resultLoader.loadResult();
            }
        }
    }
    return value;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面先来总结一下该方法的逻辑：

1. 根据 nestedQueryId 获取 MappedStatement
2. 生成参数对象
3. 获取 BoundSql
4. 创建结果加载器 ResultLoader
5. 检测当前属性是否需要进行延迟加载，若需要，则添加延迟加载相关的对象到 loaderMap 集合中
6. 如不需要延迟加载，则直接通过结果加载器加载结果

以上流程中针对一级缓存的检查是十分有必要的，若缓存命中，可直接取用结果，无需再在执行关联查询 SQL。若缓存未命中，接下来就要按部就班执行延迟加载相关逻辑

我们来看一下添加延迟加载相关对象到 loaderMap 集合中的逻辑，如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void addLoader(String property, MetaObject metaResultObject, ResultLoader resultLoader) {
    // 将属性名转为大写
    String upperFirst = getUppercaseFirstProperty(property);
    if (!upperFirst.equalsIgnoreCase(property) && loaderMap.containsKey(upperFirst)) {
        throw new ExecutorException("Nested lazy loaded result property '" + property +
                                    "' for query id '" + resultLoader.mappedStatement.getId() +
                                    " already exists in the result map. The leftmost property of all lazy loaded properties must be unique within a result map.");
    }
    // 创建 LoadPair，并将 <大写属性名，LoadPair对象> 键值对添加到 loaderMap 中
    loaderMap.put(upperFirst, new LoadPair(property, metaResultObject, resultLoader));
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们再来回顾一下文章开始的创建实体类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {

    this.useConstructorMappings = false;
    final List<Class<?>> constructorArgTypes = new ArrayList<Class<?>>();
    final List<Object> constructorArgs = new ArrayList<Object>();

    // 调用重载方法创建实体类对象
    Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);
    if (resultObject != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
        final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
        for (ResultMapping propertyMapping : propertyMappings) {
            // 如果开启了延迟加载，则为 resultObject 生成代理类，如果仅仅是配置的关联查询，没有开启延迟加载，是不会创建代理类
            if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {
                /*
                 * 创建代理类，默认使用 Javassist 框架生成代理类。
                 * 由于实体类通常不会实现接口，所以不能使用 JDK 动态代理 API 为实体类生成代理。
                 * 并且将lazyLoader传进去了
                 */
                resultObject = configuration.getProxyFactory()
                    .createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
                break;
            }
        }
    }
    this.useConstructorMappings =
        resultObject != null && !constructorArgTypes.isEmpty();
    return resultObject;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果开启了延迟加载，并且有关联查询，此时是要创建一个代理对象的，**将上面存放BondSql的lazyLoader和创建的目标对象****resultObject 作为参数传进去。**

Mybatis提供了两个实现类CglibProxyFactory和JavassistProxyFactory，分别基于org.javassist:javassist和cglib:cglib进行实现。createProxy方法就是实现懒加载逻辑的核心方法，也是我们分析的目标。



### CglibProxyFactory

CglibProxyFactory基于cglib动态代理模式，**通过继承父类的方式生成动态代理类。**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public Object createProxy(Object target, ResultLoaderMap lazyLoader, Configuration configuration, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
  return EnhancedResultObjectProxyImpl.createProxy(target, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
}

public static Object createProxy(Object target, ResultLoaderMap lazyLoader, Configuration configuration, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
  final Class<?> type = target.getClass();
  EnhancedResultObjectProxyImpl callback = new EnhancedResultObjectProxyImpl(type, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
  //由CglibProxyFactory生成对象
  Object enhanced = crateProxy(type, callback, constructorArgTypes, constructorArgs);
  //复制属性
  PropertyCopier.copyBeanProperties(type, target, enhanced);
  return enhanced;
}

static Object crateProxy(Class<?> type, Callback callback, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
  Enhancer enhancer = new Enhancer();
  enhancer.setCallback(callback);
  //设置父类对象
  enhancer.setSuperclass(type);
  try {
    type.getDeclaredMethod(WRITE_REPLACE_METHOD);
    // ObjectOutputStream will call writeReplace of objects returned by writeReplace
    if (log.isDebugEnabled()) {
      log.debug(WRITE_REPLACE_METHOD + " method was found on bean " + type + ", make sure it returns this");
    }
  } catch (NoSuchMethodException e) {
    enhancer.setInterfaces(new Class[]{WriteReplaceInterface.class});
  } catch (SecurityException e) {
    // nothing to do here
  }
  Object enhanced;
  if (constructorArgTypes.isEmpty()) {
    enhanced = enhancer.create();
  } else {
    Class<?>[] typesArray = constructorArgTypes.toArray(new Class[constructorArgTypes.size()]);
    Object[] valuesArray = constructorArgs.toArray(new Object[constructorArgs.size()]);
    enhanced = enhancer.create(typesArray, valuesArray);
  }
  return enhanced;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 可以看到，初始化Enhancer，并调用构造方法，生成对象。从`enhancer.setSuperclass(type);`也能看出cglib采用的是继承父类的方式。

**EnhancedResultObjectProxyImpl**

EnhancedResultObjectProxyImpl实现了MethodInterceptor接口，此接口是Cglib拦截目标对象方法的入口，对目标对象方法的调用都会通过此接口的intercept的方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public Object intercept(Object enhanced, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
  final String methodName = method.getName();
  try {
    synchronized (lazyLoader) {
      if (WRITE_REPLACE_METHOD.equals(methodName)) {
        Object original;
        if (constructorArgTypes.isEmpty()) {
          original = objectFactory.create(type);
        } else {
          original = objectFactory.create(type, constructorArgTypes, constructorArgs);
        }
        PropertyCopier.copyBeanProperties(type, enhanced, original);
        if (lazyLoader.size() > 0) {
          return new CglibSerialStateHolder(original, lazyLoader.getProperties(), objectFactory, constructorArgTypes, constructorArgs);
        } else {
          return original;
        }
      } else {
        if (lazyLoader.size() > 0 && !FINALIZE_METHOD.equals(methodName)) {
        /*
         * 如果 aggressive 为 true，或触发方法（比如 equals，hashCode 等）被调用，
         * 则加载所有的所有延迟加载的数据
         */
          if (aggressive || lazyLoadTriggerMethods.contains(methodName)) {
            lazyLoader.loadAll();
          } else if (PropertyNamer.isSetter(methodName)) {
            // 如果使用者显示调用了 setter 方法，则将相应的延迟加载类从 loaderMap 中移除
            final String property = PropertyNamer.methodToProperty(methodName);
            lazyLoader.remove(property);
          // 检测使用者是否调用 getter 方法
          } else if (PropertyNamer.isGetter(methodName)) {
            final String property = PropertyNamer.methodToProperty(methodName);
            if (lazyLoader.hasLoader(property)) {
              // 执行延迟加载逻辑
              lazyLoader.load(property);
            }
          }
        }
      }
    }
    //执行原方法（即父类方法）
    return methodProxy.invokeSuper(enhanced, args);
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

完整的代码

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

如上，代理方法首先会检查 aggressive 是否为 true，如果不满足，再去检查 lazyLoadTriggerMethods 是否包含当前方法名。这里两个条件只要一个为 true，当前实体类中所有需要延迟加载。aggressive 和 lazyLoadTriggerMethods 两个变量的值取决于下面的配置。

```
<setting name="aggressiveLazyLoading" value="false"/>
<setting name="lazyLoadTriggerMethods" value="equals,hashCode"/>
```

然后代理逻辑会检查使用者是不是调用了实体类的 setter 方法，如果调用了，就将该属性对应的 LoadPair 从 loaderMap 中移除。为什么要这么做呢？答案是：使用者既然手动调用 setter 方法，说明使用者想自定义某个属性的值。此时，延迟加载逻辑不应该再修改该属性的值，所以这里从 loaderMap 中移除属性对于的 LoadPair。

最后如果使用者调用的是某个属性的 **getter 方法**，且该属性配置了延迟加载，此时延迟加载逻辑就会被触发。那接下来，我们来看看延迟加载逻辑是怎样实现的的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean load(String property) throws SQLException {
    // 从 loaderMap 中移除 property 所对应的 LoadPair
    LoadPair pair = loaderMap.remove(property.toUpperCase(Locale.ENGLISH));
    if (pair != null) {
        // 加载结果
        pair.load();
        return true;
    }
    return false;
}

public void load(final Object userObject) throws SQLException {
    /*
     * 调用 ResultLoader 的 loadResult 方法加载结果，
     * 并通过 metaResultObject 设置结果到实体类对象中
     */
    this.metaResultObject.setValue(property, this.resultLoader.loadResult());
}

public Object loadResult() throws SQLException {
    // 执行关联查询
    List<Object> list = selectList();
    // 抽取结果
    resultObject = resultExtractor.extractObjectFromList(list, targetType);
    return resultObject;
}

private <E> List<E> selectList() throws SQLException {
    Executor localExecutor = executor;
    if (Thread.currentThread().getId() != this.creatorThreadId || localExecutor.isClosed()) {
        localExecutor = newExecutor();
    }
    try {
        // 通过 Executor 就行查询，这个之前已经分析过了
        // 这里的parameterObject和boundSql就是我们之前存放在LoadPair中的，现在直接拿来执行了
        return localExecutor.<E>query(mappedStatement, parameterObject, RowBounds.DEFAULT,
                                      Executor.NO_RESULT_HANDLER, cacheKey, boundSql);
    } finally {
        if (localExecutor != executor) {
            localExecutor.close(false);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

好了，延迟加载我们基本已经讲清楚了，我们介绍一下另外的一种代理方式



### JavassistProxyFactory

JavassistProxyFactory使用的是javassist方式，直接修改class文件的字节码格式。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import java.lang.reflect.Method;
import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

import javassist.util.proxy.MethodHandler;
import javassist.util.proxy.Proxy;
import javassist.util.proxy.ProxyFactory;

import org.apache.ibatis.executor.ExecutorException;
import org.apache.ibatis.executor.loader.AbstractEnhancedDeserializationProxy;
import org.apache.ibatis.executor.loader.AbstractSerialStateHolder;
import org.apache.ibatis.executor.loader.ResultLoaderMap;
import org.apache.ibatis.executor.loader.WriteReplaceInterface;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.logging.Log;
import org.apache.ibatis.logging.LogFactory;
import org.apache.ibatis.reflection.ExceptionUtil;
import org.apache.ibatis.reflection.factory.ObjectFactory;
import org.apache.ibatis.reflection.property.PropertyCopier;
import org.apache.ibatis.reflection.property.PropertyNamer;
import org.apache.ibatis.session.Configuration;

/**JavassistProxy字节码生成代理
 * 1.创建一个代理对象然后将目标对象的值赋值给代理对象，这个代理对象是可以实现其他的接口
 * 2. JavassistProxyFactory实现ProxyFactory接口createProxy(创建代理对象的方法)
 * @author Eduardo Macarron
 */
public class JavassistProxyFactory implements org.apache.ibatis.executor.loader.ProxyFactory {

  /**
   * finalize方法（垃圾回收）
   */
  private static final String FINALIZE_METHOD = "finalize";

  /**
   * writeReplace(序列化写出方法）
   */
  private static final String WRITE_REPLACE_METHOD = "writeReplace";

  /**
   * 加载ProxyFactory, 也就是JavassistProxy的入口
   */
  public JavassistProxyFactory() {
    try {
      Resources.classForName("javassist.util.proxy.ProxyFactory");
    } catch (Throwable e) {
      throw new IllegalStateException("Cannot enable lazy loading because Javassist is not available. Add Javassist to your classpath.", e);
    }
  }

  /**
   * 创建代理
   * @param target 目标对象
   * @param lazyLoader 延迟加载Map集合（那些属性是需要延迟加载的）
   * @param configuration 配置类
   * @param objectFactory 对象工厂
   * @param constructorArgTypes 构造函数类型[]
   * @param constructorArgs  构造函数的值[]
   * @return
   */
  @Override
  public Object createProxy(Object target, ResultLoaderMap lazyLoader, Configuration configuration, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    return EnhancedResultObjectProxyImpl.createProxy(target, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
  }

  public Object createDeserializationProxy(Object target, Map<String, ResultLoaderMap.LoadPair> unloadedProperties, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    return EnhancedDeserializationProxyImpl.createProxy(target, unloadedProperties, objectFactory, constructorArgTypes, constructorArgs);
  }

  @Override
  public void setProperties(Properties properties) {
      // Not Implemented
  }

  /**
   * 获取代理对象， 也就是说在执行方法之前首先调用MethodHanlder的invoke方法
   * @param type 目标类型
   * @param callback 回调对象
   * @param constructorArgTypes 构造函数类型数组
   * @param constructorArgs 构造函数值的数组
   * @return
   */
  static Object crateProxy(Class<?> type, MethodHandler callback, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    // 创建一个代理工厂类
    // 配置超类
    ProxyFactory enhancer = new ProxyFactory();
    enhancer.setSuperclass(type);
    //判断是否有writeReplace方法，如果没有将这个代理对象实现WriteReplaceInterface接口，这个接口只有一个writeReplace方法
    try {
      type.getDeclaredMethod(WRITE_REPLACE_METHOD);
      // ObjectOutputStream will call writeReplace of objects returned by writeReplace
      if (LogHolder.log.isDebugEnabled()) {
        LogHolder.log.debug(WRITE_REPLACE_METHOD + " method was found on bean " + type + ", make sure it returns this");
      }
    } catch (NoSuchMethodException e) {
      enhancer.setInterfaces(new Class[]{WriteReplaceInterface.class});
    } catch (SecurityException e) {
      // nothing to do here
    }

    Object enhanced;
    Class<?>[] typesArray = constructorArgTypes.toArray(new Class[constructorArgTypes.size()]);
    Object[] valuesArray = constructorArgs.toArray(new Object[constructorArgs.size()]);
    try {
      // 根据构造函数创建一个代理对象
      enhanced = enhancer.create(typesArray, valuesArray);
    } catch (Exception e) {
      throw new ExecutorException("Error creating lazy proxy.  Cause: " + e, e);
    }
    // 设置回调对象
    ((Proxy) enhanced).setHandler(callback);
    return enhanced;
  }

  /**
   * 实现Javassist的MethodHandler接口， 相对于Cglib的MethodInterceptor
   * 他们接口的方法名也是不一样的，Javassist的是invoke, 而cglib是intercept，叫法不同，实现功能是一样的
   */
  private static class EnhancedResultObjectProxyImpl implements MethodHandler {

    /**
     * 目标类型
     */
    private final Class<?> type;
    /**
     * 延迟加载Map集合
     */
    private final ResultLoaderMap lazyLoader;

    /**
     * 是否配置延迟加载
     */
    private final boolean aggressive;

    /**
     * 延迟加载触发的方法
     */
    private final Set<String> lazyLoadTriggerMethods;

    /**
     * 对象工厂
     */
    private final ObjectFactory objectFactory;

    /**
     * 构造函数类型数组
     */
    private final List<Class<?>> constructorArgTypes;

    /**
     * 构造函数类型的值数组
     */
    private final List<Object> constructorArgs;

    /**
     * 构造函数私有化了
     * @param type
     * @param lazyLoader
     * @param configuration
     * @param objectFactory
     * @param constructorArgTypes
     * @param constructorArgs
     */
    private EnhancedResultObjectProxyImpl(Class<?> type, ResultLoaderMap lazyLoader, Configuration configuration, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      this.type = type;
      this.lazyLoader = lazyLoader;
      this.aggressive = configuration.isAggressiveLazyLoading();
      this.lazyLoadTriggerMethods = configuration.getLazyLoadTriggerMethods();
      this.objectFactory = objectFactory;
      this.constructorArgTypes = constructorArgTypes;
      this.constructorArgs = constructorArgs;
    }

    public static Object createProxy(Object target, ResultLoaderMap lazyLoader, Configuration configuration, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      // 获取目标类型
      // 创建一个EnhancedResultObjectProxyImpl对象，回调对象
      final Class<?> type = target.getClass();
      EnhancedResultObjectProxyImpl callback = new EnhancedResultObjectProxyImpl(type, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
      Object enhanced = crateProxy(type, callback, constructorArgTypes, constructorArgs);
      PropertyCopier.copyBeanProperties(type, target, enhanced);
      return enhanced;
    }

    /**
     * 回调方法
     * @param enhanced 代理对象
     * @param method 方法
     * @param methodProxy 代理方法
     * @param args 入参
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object enhanced, Method method, Method methodProxy, Object[] args) throws Throwable {
      //获取方法名称
      final String methodName = method.getName();
      try {
        synchronized (lazyLoader) {
          if (WRITE_REPLACE_METHOD.equals(methodName)) {
            //如果方法是writeReplace
            Object original;
            if (constructorArgTypes.isEmpty()) {
              original = objectFactory.create(type);
            } else {
              original = objectFactory.create(type, constructorArgTypes, constructorArgs);
            }
            PropertyCopier.copyBeanProperties(type, enhanced, original);
            if (lazyLoader.size() > 0) {
              return new JavassistSerialStateHolder(original, lazyLoader.getProperties(), objectFactory, constructorArgTypes, constructorArgs);
            } else {
              return original;
            }
          } else {
            //不是writeReplace方法
            // 延迟加载长度大于0， 且不是finalize方法
            // configuration配置延迟加载参数，延迟加载触发的方法包含这个方法
            // 延迟加载所有数据
            if (lazyLoader.size() > 0 && !FINALIZE_METHOD.equals(methodName)) {
              if (aggressive || lazyLoadTriggerMethods.contains(methodName)) {
                lazyLoader.loadAll();
              } else if (PropertyNamer.isSetter(methodName)) {
                final String property = PropertyNamer.methodToProperty(methodName);
                lazyLoader.remove(property);
              } else if (PropertyNamer.isGetter(methodName)) {
                final String property = PropertyNamer.methodToProperty(methodName);
                if (lazyLoader.hasLoader(property)) {
                  lazyLoader.load(property);
                }
              }
            }
          }
        }
        return methodProxy.invoke(enhanced, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
  }

  private static class EnhancedDeserializationProxyImpl extends AbstractEnhancedDeserializationProxy implements MethodHandler {

    private EnhancedDeserializationProxyImpl(Class<?> type, Map<String, ResultLoaderMap.LoadPair> unloadedProperties, ObjectFactory objectFactory,
            List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      super(type, unloadedProperties, objectFactory, constructorArgTypes, constructorArgs);
    }

    public static Object createProxy(Object target, Map<String, ResultLoaderMap.LoadPair> unloadedProperties, ObjectFactory objectFactory,
            List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      final Class<?> type = target.getClass();
      EnhancedDeserializationProxyImpl callback = new EnhancedDeserializationProxyImpl(type, unloadedProperties, objectFactory, constructorArgTypes, constructorArgs);
      Object enhanced = crateProxy(type, callback, constructorArgTypes, constructorArgs);
      PropertyCopier.copyBeanProperties(type, target, enhanced);
      return enhanced;
    }

    @Override
    public Object invoke(Object enhanced, Method method, Method methodProxy, Object[] args) throws Throwable {
      final Object o = super.invoke(enhanced, method, args);
      return o instanceof AbstractSerialStateHolder ? o : methodProxy.invoke(o, args);
    }

    @Override
    protected AbstractSerialStateHolder newSerialStateHolder(Object userBean, Map<String, ResultLoaderMap.LoadPair> unloadedProperties, ObjectFactory objectFactory,
            List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      return new JavassistSerialStateHolder(userBean, unloadedProperties, objectFactory, constructorArgTypes, constructorArgs);
    }
  }

  private static class LogHolder {
    private static final Log log = LogFactory.getLog(JavassistProxyFactory.class);
  }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注释已经很清楚了，我就不累述了

# [Mybaits 源码解析 （九）----- 一级缓存和二级缓存源码分析](https://www.cnblogs.com/java-chen-hao/p/11770064.html)

**正文**

像Mybatis、Hibernate这样的ORM框架，封装了JDBC的大部分操作，极大的简化了我们对数据库的操作。

在实际项目中，我们发现在一个事务中查询同样的语句两次的时候，第二次没有进行数据库查询，直接返回了结果，实际这种情况我们就可以称为缓存。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11770064.html#_labelTop)

## Mybatis的缓存级别



###  一级缓存

- MyBatis的一级查询缓存（也叫作本地缓存）是基于org.apache.ibatis.cache.impl.PerpetualCache 类的 HashMap本地缓存，其作用域是SqlSession，myBatis 默认一级查询缓存是开启状态，且不能关闭。
- 在同一个SqlSession中两次执行相同的 sql查询语句，第一次执行完毕后，会将查询结果写入到缓存中，第二次会从缓存中直接获取数据，而不再到数据库中进行查询，这样就减少了数据库的访问，从而提高查询效率。
- 基于PerpetualCache 的 HashMap本地缓存，其存储作用域为 Session，PerpetualCache 对象是在SqlSession中的Executor的localcache属性当中存放，当 Session flush 或 close 之后，该Session中的所有 Cache 就将清空。

### 二级缓存

- 二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap存储，不同在于其存储作用域为 Mapper(Namespace)，每个Mapper中有一个Cache对象，存放在Configration中，并且将其放进当前Mapper的所有MappedStatement当中，并且可自定义存储源，如 Ehcache。

- Mapper级别缓存，定义在Mapper文件的<cache>标签并需要开启此缓存

用下面这张图描述一级缓存和二级缓存的关系。

![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191031112956850-1740606388.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11770064.html#_labelTop)

## CacheKey

在 MyBatis 中，引入缓存的目的是为提高查询效率，降低数据库压力。既然 MyBatis 引入了缓存，那么大家思考过缓存中的 key 和 value 的值分别是什么吗？大家可能很容易能回答出 value 的内容，不就是 SQL 的查询结果吗。那 key 是什么呢？是字符串，还是其他什么对象？如果是字符串的话，那么大家首先能想到的是用 SQL 语句作为 key。但这是不对的，比如：

```
SELECT * FROM user where id > ?
```

id > 1 和 id > 10 查出来的结果可能是不同的，所以我们不能简单的使用 SQL 语句作为 key。从这里可以看出来，运行时参数将会影响查询结果，因此我们的 key 应该涵盖运行时参数。除此之外呢，如果进行分页查询也会导致查询结果不同，因此 key 也应该涵盖分页参数。综上，我们不能使用简单的 SQL 语句作为 key。应该考虑使用一种复合对象，能涵盖可影响查询结果的因子。在 MyBatis 中，这种复合对象就是 CacheKey。下面来看一下它的定义。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class CacheKey implements Cloneable, Serializable {

    private static final int DEFAULT_MULTIPLYER = 37;
    private static final int DEFAULT_HASHCODE = 17;

    // 乘子，默认为37
    private final int multiplier;
    // CacheKey 的 hashCode，综合了各种影响因子
    private int hashcode;
    // 校验和
    private long checksum;
    // 影响因子个数
    private int count;
    // 影响因子集合
    private List<Object> updateList;
    
    public CacheKey() {
        this.hashcode = DEFAULT_HASHCODE;
        this.multiplier = DEFAULT_MULTIPLYER;
        this.count = 0;
        this.updateList = new ArrayList<Object>();
    }
    
    /** 每当执行更新操作时，表示有新的影响因子参与计算 
     *  当不断有新的影响因子参与计算时，hashcode 和 checksum 将会变得愈发复杂和随机。这样可降低冲突率，使 CacheKey 可在缓存中更均匀的分布。
     */
    public void update(Object object) {
            int baseHashCode = object == null ? 1 : ArrayUtil.hashCode(object);
        // 自增 count
        count++;
        // 计算校验和
        checksum += baseHashCode;
        // 更新 baseHashCode
        baseHashCode *= count;

        // 计算 hashCode
        hashcode = multiplier * hashcode + baseHashCode;

        // 保存影响因子
        updateList.add(object);
    }
    
    /**
     *  CacheKey 最终要作为键存入 HashMap，因此它需要覆盖 equals 和 hashCode 方法
     */
    public boolean equals(Object object) {
        // 检测是否为同一个对象
        if (this == object) {
            return true;
        }
        // 检测 object 是否为 CacheKey
        if (!(object instanceof CacheKey)) {
            return false;
        }
        final CacheKey cacheKey = (CacheKey) object;

        // 检测 hashCode 是否相等
        if (hashcode != cacheKey.hashcode) {
            return false;
        }
        // 检测校验和是否相同
        if (checksum != cacheKey.checksum) {
            return false;
        }
        // 检测 coutn 是否相同
        if (count != cacheKey.count) {
            return false;
        }

        // 如果上面的检测都通过了，下面分别对每个影响因子进行比较
        for (int i = 0; i < updateList.size(); i++) {
            Object thisObject = updateList.get(i);
            Object thatObject = cacheKey.updateList.get(i);
            if (!ArrayUtil.equals(thisObject, thatObject)) {
                return false;
            }
        }
        return true;
    }

    public int hashCode() {
        // 返回 hashcode 变量
        return hashcode;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

当不断有新的影响因子参与计算时，hashcode 和 checksum 将会变得愈发复杂和随机。这样可降低冲突率，使 CacheKey 可在缓存中更均匀的分布。CacheKey 最终要作为键存入 HashMap，因此它需要覆盖 equals 和 hashCode 方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11770064.html#_labelTop)

## 一级缓存源码解析



### 一级缓存的测试

**同一个session查询**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    SqlSession session = sqlSessionFactory.openSession();
    try {
        Blog blog = (Blog)session.selectOne("queryById",1);
        Blog blog2 = (Blog)session.selectOne("queryById",1);
    } finally {
        session.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结论：只有一个DB查询

**两个session分别查询**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    SqlSession session = sqlSessionFactory.openSession();
    SqlSession session1 = sqlSessionFactory.openSession();
    try {
        Blog blog = (Blog)session.selectOne("queryById",17);
        Blog blog2 = (Blog)session1.selectOne("queryById",17);
    } finally {
        session.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 结论：进行了两次DB查询

**同一个session，进行update之后再次查询**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    SqlSession session = sqlSessionFactory.openSession();
    try {
        Blog blog = (Blog)session.selectOne("queryById",17);
        blog.setName("llll");
        session.update("updateBlog",blog);
        
        Blog blog2 = (Blog)session.selectOne("queryById",17);
    } finally {
        session.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结论：进行了两次DB查询

总结：在一级缓存中，同一个SqlSession下，查询语句相同的SQL会被缓存，如果执行增删改操作之后，该缓存就会被删除



### 创建缓存对象**PerpetualCache**

我们来回顾一下创建SqlSession的过程

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
SqlSession session = sessionFactory.openSession();

public SqlSession openSession() {
    return this.openSessionFromDataSource(this.configuration.getDefaultExecutorType(), (TransactionIsolationLevel)null, false);
}

private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;

    DefaultSqlSession var8;
    try {
        Environment environment = this.configuration.getEnvironment();
        TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        //创建SQL执行器
        Executor executor = this.configuration.newExecutor(tx, execType);
        var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
    } catch (Exception var12) {
        this.closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
    } finally {
        ErrorContext.instance().reset();
    }

    return var8;
}

public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? this.defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Object executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        //默认创建SimpleExecutor
        executor = new SimpleExecutor(this, transaction);
    }

    if (this.cacheEnabled) {
        //开启二级缓存就会用CachingExecutor装饰SimpleExecutor
        executor = new CachingExecutor((Executor)executor);
    }

    Executor executor = (Executor)this.interceptorChain.pluginAll(executor);
    return executor;
}

public SimpleExecutor(Configuration configuration, Transaction transaction) {
    super(configuration, transaction);
}

protected BaseExecutor(Configuration configuration, Transaction transaction) {
    this.transaction = transaction;
    this.deferredLoads = new ConcurrentLinkedQueue();
    //创建一个缓存对象，PerpetualCache并不是线程安全的
    //但SqlSession和Executor对象在通常情况下只能有一个线程访问，而且访问完成之后马上销毁。也就是session.close();
    this.localCache = new PerpetualCache("LocalCache");
    this.localOutputParameterCache = new PerpetualCache("LocalOutputParameterCache");
    this.closed = false;
    this.configuration = configuration;
    this.wrapper = this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我只是简单的贴了代码，大家可以看我之前的博客，我们可以看到DefaultSqlSession中有SimpleExecutor对象，SimpleExecutor对象中有一个PerpetualCache，一级缓存的数据就是存储在PerpetualCache对象中，SqlSession关闭的时候会清空PerpetualCache



### 一级缓存实现

再来看BaseExecutor中的query方法是怎么实现一级缓存的，executor默认实现为CachingExecutor

**CachingExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameter);
    //利用sql和执行的参数生成一个key，如果同一sql不同的执行参数的话，将会生成不同的key
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}

@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
    throws SQLException {
    // 这里是二级缓存的查询，我们暂且不看
    Cache cache = ms.getCache();
    if (cache != null) {
        flushCacheIfRequired(ms);
        if (ms.isUseCache() && resultHandler == null) {
            ensureNoOutParams(ms, parameterObject, boundSql);
            @SuppressWarnings("unchecked")
            List<E> list = (List<E>) tcm.getObject(cache, key);
            if (list == null) {
                list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                tcm.putObject(cache, key, list); // issue #578 and #116
            }
            return list;
        }
    }
    
    // 直接来到这里
    // 实现为BaseExecutor.query()
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，在访问一级缓存之前，MyBatis 首先会调用 createCacheKey 方法创建 CacheKey。下面我们来看一下 createCacheKey 方法的逻辑：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    // 创建 CacheKey 对象
    CacheKey cacheKey = new CacheKey();
    // 将 MappedStatement 的 id 作为影响因子进行计算
    cacheKey.update(ms.getId());
    // RowBounds 用于分页查询，下面将它的两个字段作为影响因子进行计算
    cacheKey.update(rowBounds.getOffset());
    cacheKey.update(rowBounds.getLimit());
    // 获取 sql 语句，并进行计算
    cacheKey.update(boundSql.getSql());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();
    for (ParameterMapping parameterMapping : parameterMappings) {
        if (parameterMapping.getMode() != ParameterMode.OUT) {
            // 运行时参数
            Object value;    
            // 当前大段代码用于获取 SQL 中的占位符 #{xxx} 对应的运行时参数，
            // 前文有类似分析，这里忽略了
            String propertyName = parameterMapping.getProperty();
            if (boundSql.hasAdditionalParameter(propertyName)) {
                value = boundSql.getAdditionalParameter(propertyName);
            } else if (parameterObject == null) {
                value = null;
            } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                value = parameterObject;
            } else {
                MetaObject metaObject = configuration.newMetaObject(parameterObject);
                value = metaObject.getValue(propertyName);
            }
            
            // 让运行时参数参与计算
            cacheKey.update(value);
        }
    }
    if (configuration.getEnvironment() != null) {
        // 获取 Environment id 遍历，并让其参与计算
        cacheKey.update(configuration.getEnvironment().getId());
    }
    return cacheKey;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，在计算 CacheKey 的过程中，有很多影响因子参与了计算。比如 MappedStatement 的 id 字段，SQL 语句，分页参数，运行时变量，Environment 的 id 字段等。通过让这些影响因子参与计算，可以很好的区分不同查询请求。所以，我们可以简单的把 CacheKey 看做是一个查询请求的 id。有了 CacheKey，我们就可以使用它读写缓存了。

**SimpleExecutor(BaseExecutor)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@SuppressWarnings("unchecked")
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
        // 看这里，先从localCache中获取对应CacheKey的结果值
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
            handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
            // 如果缓存中没有值，则从DB中查询
            list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
    } finally {
        queryStack--;
    }
    if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
            deferredLoad.load();
        }
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
            clearLocalCache();
        }
    }
    return list;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**BaseExecutor.queryFromDatabase()**

我们先来看下这种缓存中没有值的情况，看一下查询后的结果是如何被放置到缓存中的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
        // 1.执行查询，获取list
        list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
        localCache.removeObject(key);
    }
    // 2.将查询后的结果放置到localCache中，key就是我们刚才封装的CacheKey，value就是从DB中查询到的list
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
        localOutputParameterCache.putObject(key, parameter);
    }
    return list;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看 **localCache.putObject(key, list);**



### PerpetualCache

PerpetualCache 是一级缓存使用的缓存类，内部使用了 HashMap 实现缓存功能。它的源码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class PerpetualCache implements Cache {

    private final String id;

    private Map<Object, Object> cache = new HashMap<Object, Object>();

    public PerpetualCache(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public int getSize() {
        return cache.size();
    }

    @Override
    public void putObject(Object key, Object value) {
        // 存储键值对到 HashMap
        cache.put(key, value);
    }

    @Override
    public Object getObject(Object key) {
        // 查找缓存项
        return cache.get(key);
    }

    @Override
    public Object removeObject(Object key) {
        // 移除缓存项
        return cache.remove(key);
    }

    @Override
    public void clear() {
        cache.clear();
    }
    
    // 省略部分代码
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

总结：可以看到localCache本质上就是一个Map，key为我们的CacheKey，value为我们的结果值，是不是很简单，只是封装了一个**Map而已。**



### 清除缓存

**SqlSession.update（）**

当我们进行更新操作时，会执行如下代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    //每次执行update/insert/delete语句时都会清除一级缓存。
    clearLocalCache();
    // 然后再进行更新操作
    return doUpdate(ms, parameter);
}
 
@Override
public void clearLocalCache() {
    if (!closed) {
        // 直接将Map清空
        localCache.clear();
        localOutputParameterCache.clear();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**session.close();**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//DefaultSqlSession
public void close() {
    try {
        this.executor.close(this.isCommitOrRollbackRequired(false));
        this.closeCursors();
        this.dirty = false;
    } finally {
        ErrorContext.instance().reset();
    }

}

//BaseExecutor
public void close(boolean forceRollback) {
    try {
        try {
            this.rollback(forceRollback);
        } finally {
            if (this.transaction != null) {
                this.transaction.close();
            }

        }
    } catch (SQLException var11) {
        log.warn("Unexpected exception on closing transaction.  Cause: " + var11);
    } finally {
        this.transaction = null;
        this.deferredLoads = null;
        this.localCache = null;
        this.localOutputParameterCache = null;
        this.closed = true;
    }

}

public void rollback(boolean required) throws SQLException {
    if (!this.closed) {
        try {
            this.clearLocalCache();
            this.flushStatements(true);
        } finally {
            if (required) {
                this.transaction.rollback();
            }

        }
    }

}

public void clearLocalCache() {
    if (!this.closed) {
        // 直接将Map清空
        this.localCache.clear();
        this.localOutputParameterCache.clear();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

当关闭SqlSession时，也会清楚SqlSession中的一级缓存



### **总结**

1. 一级缓存只在同一个SqlSession中共享数据
2. 在同一个SqlSession对象执行相同的sql并参数也要相同，缓存才有效。
3. 如果在SqlSession中执行update/insert/detete语句或者**session.close();**的话，SqlSession中的executor对象会将一级缓存清空。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11770064.html#_labelTop)

## 二级缓存源码解析

二级缓存构建在一级缓存之上，在收到查询请求时，MyBatis 首先会查询二级缓存。若二级缓存未命中，再去查询一级缓存。与一级缓存不同，二级缓存和具体的命名空间绑定，一个Mapper中有一个Cache，相同Mapper中的MappedStatement公用一个Cache，一级缓存则是和 SqlSession 绑定。一级缓存不存在并发问题二级缓存可在多个命名空间间共享，这种情况下，会存在并发问题，比喻多个不同的SqlSession 会同时执行相同的SQL语句，参数也相同，那么CacheKey是相同的，就会造成多个线程并发访问相同CacheKey的值，下面首先来看一下访问二级缓存的逻辑。



### 二级缓存的测试

二级缓存需要在Mapper.xml中配置<cache/>标签

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="mybatis.BlogMapper">
    <select id="queryById" parameterType="int" resultType="jdbc.Blog">
        select * from blog where id = #{id}
    </select>
    <update id="updateBlog" parameterType="jdbc.Blog">
        update Blog set name = #{name},url = #{url} where id=#{id}
    </update>
    <!-- 开启BlogMapper二级缓存 -->
    <cache/>
</mapper>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**不同的session进行相同的查询**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    SqlSession session = sqlSessionFactory.openSession();
    SqlSession session1 = sqlSessionFactory.openSession();
    try {
        Blog blog = (Blog)session.selectOne("queryById",17);
        Blog blog2 = (Blog)session1.selectOne("queryById",17);
    } finally {
        session.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结论：执行两次DB查询

**第一个session查询完成之后，手动提交，在执行第二个session查询**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    SqlSession session = sqlSessionFactory.openSession();
    SqlSession session1 = sqlSessionFactory.openSession();
    try {
        Blog blog = (Blog)session.selectOne("queryById",17);
        session.commit();
 
        Blog blog2 = (Blog)session1.selectOne("queryById",17);
    } finally {
        session.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结论：执行一次DB查询

**第一个session查询完成之后，手动关闭，在执行第二个session查询**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String[] args) {
    SqlSession session = sqlSessionFactory.openSession();
    SqlSession session1 = sqlSessionFactory.openSession();
    try {
        Blog blog = (Blog)session.selectOne("queryById",17);
        session.close();
 
        Blog blog2 = (Blog)session1.selectOne("queryById",17);
    } finally {
        session.close();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结论：执行一次DB查询

**总结：二级缓存的生效必须在session提交或关闭之后才会生效**



### 标签<cache/>的解析

按照之前的对Mybatis的分析，对blog.xml的解析工作主要交给XMLConfigBuilder.parse()方法来实现的

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

我们来看看解析Mapper.xml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// XMLMapperBuilder.parse()
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
        // 解析mapper属性
        configurationElement(parser.evalNode("/mapper"));
        configuration.addLoadedResource(resource);
        bindMapperForNamespace();
    }
 
    parsePendingResultMaps();
    parsePendingChacheRefs();
    parsePendingStatements();
}
 
// configurationElement()
private void configurationElement(XNode context) {
    try {
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.equals("")) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }
        builderAssistant.setCurrentNamespace(namespace);
        cacheRefElement(context.evalNode("cache-ref"));
        // 最终在这里看到了关于cache属性的处理
        cacheElement(context.evalNode("cache"));
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        sqlElement(context.evalNodes("/mapper/sql"));
        // 这里会将生成的Cache包装到对应的MappedStatement
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
    }
}
 
// cacheElement()
private void cacheElement(XNode context) throws Exception {
    if (context != null) {
        //解析<cache/>标签的type属性，这里我们可以自定义cache的实现类，比如redisCache，如果没有自定义，这里使用和一级缓存相同的PERPETUAL
        String type = context.getStringAttribute("type", "PERPETUAL");
        Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
        String eviction = context.getStringAttribute("eviction", "LRU");
        Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
        Long flushInterval = context.getLongAttribute("flushInterval");
        Integer size = context.getIntAttribute("size");
        boolean readWrite = !context.getBooleanAttribute("readOnly", false);
        boolean blocking = context.getBooleanAttribute("blocking", false);
        Properties props = context.getChildrenAsProperties();
        // 构建Cache对象
        builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先来看看是如何构建Cache对象的

**MapperBuilderAssistant.useNewCache()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Cache useNewCache(Class<? extends Cache> typeClass,
                         Class<? extends Cache> evictionClass,
                         Long flushInterval,
                         Integer size,
                         boolean readWrite,
                         boolean blocking,
                         Properties props) {
    // 1.生成Cache对象
    Cache cache = new CacheBuilder(currentNamespace)
         //这里如果我们定义了<cache/>中的type，就使用自定义的Cache,否则使用和一级缓存相同的PerpetualCache
        .implementation(valueOrDefault(typeClass, PerpetualCache.class))
        .addDecorator(valueOrDefault(evictionClass, LruCache.class))
        .clearInterval(flushInterval)
        .size(size)
        .readWrite(readWrite)
        .blocking(blocking)
        .properties(props)
        .build();
    // 2.添加到Configuration中
    configuration.addCache(cache);
    // 3.并将cache赋值给MapperBuilderAssistant.currentCache
    currentCache = cache;
    return cache;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到一个Mapper.xml只会解析一次<cache/>标签，也就是只创建一次Cache对象，放进configuration中，并将cache赋值给MapperBuilderAssistant.currentCache



### buildStatementFromContext(context.evalNodes("select|insert|update|delete"));将Cache包装到MappedStatement

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// buildStatementFromContext()
private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
        buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
}
 
//buildStatementFromContext()
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
        final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
        try {
            // 每一条执行语句转换成一个MappedStatement
            statementParser.parseStatementNode();
        } catch (IncompleteElementException e) {
            configuration.addIncompleteStatement(statementParser);
        }
    }
}
 
// XMLStatementBuilder.parseStatementNode();
public void parseStatementNode() {
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");
    ...
 
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultType = context.getStringAttribute("resultType");
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);
 
    ...
    // 创建MappedStatement对象
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
                                        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
                                        resultSetTypeEnum, flushCache, useCache, resultOrdered, 
                                        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
 
// builderAssistant.addMappedStatement()
public MappedStatement addMappedStatement(
    String id,
    ...) {
 
    if (unresolvedCacheRef) {
        throw new IncompleteElementException("Cache-ref not yet resolved");
    }
 
    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    //创建MappedStatement对象
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        ...
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
        .useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);// 在这里将之前生成的Cache封装到MappedStatement
 
    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
        statementBuilder.parameterMap(statementParameterMap);
    }
 
    MappedStatement statement = statementBuilder.build();
    configuration.addMappedStatement(statement);
    return statement;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到将Mapper中创建的Cache对象，加入到了每个MappedStatement对象中，也就是同一个Mapper中所有的MappedStatement 中的cache属性引用是同一个

有关于<cache/>标签的解析就到这了。



### 查询源码分析

**CachingExecutor**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// CachingExecutor
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameterObject);
    // 创建 CacheKey
    CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}

public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
    throws SQLException {
    // 从 MappedStatement 中获取 Cache，注意这里的 Cache 是从MappedStatement中获取的
    // 也就是我们上面解析Mapper中<cache/>标签中创建的，它保存在Configration中
    // 我们在上面解析blog.xml时分析过每一个MappedStatement都有一个Cache对象，就是这里
    Cache cache = ms.getCache();
    // 如果配置文件中没有配置 <cache>，则 cache 为空
    if (cache != null) {
        //如果需要刷新缓存的话就刷新：flushCache="true"
        flushCacheIfRequired(ms);
        if (ms.isUseCache() && resultHandler == null) {
            ensureNoOutParams(ms, boundSql);
            // 访问二级缓存
            List<E> list = (List<E>) tcm.getObject(cache, key);
            // 缓存未命中
            if (list == null) {
                // 如果没有值，则执行查询，这个查询实际也是先走一级缓存查询，一级缓存也没有的话，则进行DB查询
                list = delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                // 缓存查询结果
                tcm.putObject(cache, key, list);
            }
            return list;
        }
    }
    return delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果设置了**flushCache="true"，则每次查询都会刷新缓存**

```
<!-- 执行此语句清空缓存 -->
<select id="getAll" resultType="entity.TDemo" useCache="true" flushCache="true" >
    select * from t_demo
</select>
```

如上，注意二级缓存是从 MappedStatement 中获取的。由于 MappedStatement 存在于全局配置中，可以多个 CachingExecutor 获取到，这样就会出现线程安全问题。除此之外，若不加以控制，多个事务共用一个缓存实例，会导致脏读问题。至于脏读问题，需要借助其他类来处理，也就是上面代码中 tcm 变量对应的类型。下面分析一下。

**TransactionalCacheManager**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/** 事务缓存管理器 */
public class TransactionalCacheManager {

    // Cache 与 TransactionalCache 的映射关系表
    private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();

    public void clear(Cache cache) {
        // 获取 TransactionalCache 对象，并调用该对象的 clear 方法，下同
        getTransactionalCache(cache).clear();
    }

    public Object getObject(Cache cache, CacheKey key) {
        // 直接从TransactionalCache中获取缓存
        return getTransactionalCache(cache).getObject(key);
    }

    public void putObject(Cache cache, CacheKey key, Object value) {
        // 直接存入TransactionalCache的缓存中
        getTransactionalCache(cache).putObject(key, value);
    }

    public void commit() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.commit();
        }
    }

    public void rollback() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.rollback();
        }
    }

    private TransactionalCache getTransactionalCache(Cache cache) {
        // 从映射表中获取 TransactionalCache
        TransactionalCache txCache = transactionalCaches.get(cache);
        if (txCache == null) {
            // TransactionalCache 也是一种装饰类，为 Cache 增加事务功能
            // 创建一个新的TransactionalCache，并将真正的Cache对象存进去
            txCache = new TransactionalCache(cache);
            transactionalCaches.put(cache, txCache);
        }
        return txCache;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

TransactionalCacheManager 内部维护了 Cache 实例与 TransactionalCache 实例间的映射关系，该类也仅负责维护两者的映射关系，真正做事的还是 TransactionalCache。TransactionalCache 是一种缓存装饰器，可以为 Cache 实例增加事务功能。我在之前提到的脏读问题正是由该类进行处理的。下面分析一下该类的逻辑。

**TransactionalCache**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class TransactionalCache implements Cache {
    //真正的缓存对象，和上面的Map<Cache, TransactionalCache>中的Cache是同一个
    private final Cache delegate;
    private boolean clearOnCommit;
    // 在事务被提交前，所有从数据库中查询的结果将缓存在此集合中
    private final Map<Object, Object> entriesToAddOnCommit;
    // 在事务被提交前，当缓存未命中时，CacheKey 将会被存储在此集合中
    private final Set<Object> entriesMissedInCache;


    @Override
    public Object getObject(Object key) {
        // 查询的时候是直接从delegate中去查询的，也就是从真正的缓存对象中查询
        Object object = delegate.getObject(key);
        if (object == null) {
            // 缓存未命中，则将 key 存入到 entriesMissedInCache 中
            entriesMissedInCache.add(key);
        }

        if (clearOnCommit) {
            return null;
        } else {
            return object;
        }
    }

    @Override
    public void putObject(Object key, Object object) {
        // 将键值对存入到 entriesToAddOnCommit 这个Map中中，而非真实的缓存对象 delegate 中
        entriesToAddOnCommit.put(key, object);
    }

    @Override
    public Object removeObject(Object key) {
        return null;
    }

    @Override
    public void clear() {
        clearOnCommit = true;
        // 清空 entriesToAddOnCommit，但不清空 delegate 缓存
        entriesToAddOnCommit.clear();
    }

    public void commit() {
        // 根据 clearOnCommit 的值决定是否清空 delegate
        if (clearOnCommit) {
            delegate.clear();
        }
        
        // 刷新未缓存的结果到 delegate 缓存中
        flushPendingEntries();
        // 重置 entriesToAddOnCommit 和 entriesMissedInCache
        reset();
    }

    public void rollback() {
        unlockMissedEntries();
        reset();
    }

    private void reset() {
        clearOnCommit = false;
        // 清空集合
        entriesToAddOnCommit.clear();
        entriesMissedInCache.clear();
    }

    private void flushPendingEntries() {
        for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
            // 将 entriesToAddOnCommit 中的内容转存到 delegate 中
            delegate.putObject(entry.getKey(), entry.getValue());
        }
        for (Object entry : entriesMissedInCache) {
            if (!entriesToAddOnCommit.containsKey(entry)) {
                // 存入空值
                delegate.putObject(entry, null);
            }
        }
    }

    private void unlockMissedEntries() {
        for (Object entry : entriesMissedInCache) {
            try {
                // 调用 removeObject 进行解锁
                delegate.removeObject(entry);
            } catch (Exception e) {
                log.warn("...");
            }
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

存储二级缓存对象的时候是放到了TransactionalCache.entriesToAddOnCommit这个map中，但是每次查询的时候是直接从TransactionalCache.delegate中去查询的，所以这个二级缓存查询数据库后，设置缓存值是没有立刻生效的，主要是因为直接存到 delegate 会导致脏数据问题。



### 为何只有SqlSession提交或关闭之后二级缓存才会生效？

那我们来看下SqlSession.commit()方法做了什么

**SqlSession**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void commit(boolean force) {
    try {
        // 主要是这句
        executor.commit(isCommitOrRollbackRequired(force));
        dirty = false;
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
 
// CachingExecutor.commit()
@Override
public void commit(boolean required) throws SQLException {
    delegate.commit(required);
    tcm.commit();// 在这里
}
 
// TransactionalCacheManager.commit()
public void commit() {
    for (TransactionalCache txCache : transactionalCaches.values()) {
        txCache.commit();// 在这里
    }
}
 
// TransactionalCache.commit()
public void commit() {
    if (clearOnCommit) {
        delegate.clear();
    }
    flushPendingEntries();//这一句
    reset();
}
 
// TransactionalCache.flushPendingEntries()
private void flushPendingEntries() {
    for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
        // 在这里真正的将entriesToAddOnCommit的对象逐个添加到delegate中，只有这时，二级缓存才真正的生效
        delegate.putObject(entry.getKey(), entry.getValue());
    }
    for (Object entry : entriesMissedInCache) {
        if (!entriesToAddOnCommit.containsKey(entry)) {
            delegate.putObject(entry, null);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果从数据库查询到的数据直接存到 delegate 会导致脏数据问题。下面通过一张图演示一下脏数据问题发生的过程，假设两个线程开启两个不同的事务，它们的执行过程如下：

![img](https://img2018.cnblogs.com/blog/1168971/201910/1168971-20191031155354463-1986976124.png)

如上图，时刻2，事务 A 对记录 A 进行了更新。时刻3，事务 A 从数据库查询记录 A，并将记录 A 写入缓存中。时刻4，事务 B 查询记录 A，由于缓存中存在记录 A，事务 B 直接从缓存中取数据。这个时候，脏数据问题就发生了。事务 B 在事务 A 未提交情况下，读取到了事务 A 所修改的记录。为了解决这个问题，我们可以为每个事务引入一个独立的缓存。查询数据时，仍从 delegate 缓存（以下统称为共享缓存）中查询。若缓存未命中，则查询数据库。存储查询结果时，并不直接存储查询结果到共享缓存中，而是先存储到事务缓存中，也就是 entriesToAddOnCommit 集合。当事务提交时，再将事务缓存中的缓存项转存到共享缓存中。这样，事务 B 只能在事务 A 提交后，才能读取到事务 A 所做的修改，解决了脏读问题。



### 二级缓存的刷新

我们来看看SqlSession的更新操作

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public int update(String statement, Object parameter) {
    int var4;
    try {
        this.dirty = true;
        MappedStatement ms = this.configuration.getMappedStatement(statement);
        var4 = this.executor.update(ms, this.wrapCollection(parameter));
    } catch (Exception var8) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + var8, var8);
    } finally {
        ErrorContext.instance().reset();
    }

    return var4;
}

public int update(MappedStatement ms, Object parameterObject) throws SQLException {
    this.flushCacheIfRequired(ms);
    return this.delegate.update(ms, parameterObject);
}

private void flushCacheIfRequired(MappedStatement ms) {
    //获取MappedStatement对应的Cache，进行清空
    Cache cache = ms.getCache();
    //SQL需设置flushCache="true" 才会执行清空
    if (cache != null && ms.isFlushCacheRequired()) {
  this.tcm.clear(cache);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

MyBatis二级缓存只适用于不常进行增、删、改的数据，比如国家行政区省市区街道数据。一但数据变更，MyBatis会清空缓存。因此二级缓存不适用于经常进行更新的数据。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11770064.html#_labelTop)

## 使用redis存储二级缓存

通过上面代码分析，我们知道二级缓存默认和一级缓存都是使用的PerpetualCache存储结果，一级缓存只要SQLSession关闭就会清空，其内部使用HashMap实现，所以二级缓存无法实现分布式，并且服务器重启后就没有缓存了。此时就需要引入第三方缓存中间件，将缓存的值存到外部，如redis和ehcache

修改mapper.xml中的配置。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.tyb.saas.common.dal.dao.AreaDefaultMapper">
 
    <!--
    flushInterval（清空缓存的时间间隔）: 单位毫秒，可以被设置为任意的正整数。
        默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新。
    size（引用数目）: 可以被设置为任意正整数，要记住你缓存的对象数目和你运行环境的可用内存资源数目。默认值是1024。
    readOnly（只读）:属性可以被设置为true或false。只读的缓存会给所有调用者返回缓存对象的相同实例。
        因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是false。
    eviction（回收策略）: 默认的是 LRU:
        1.LRU – 最近最少使用的:移除最长时间不被使用的对象。
        2.FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
        3.SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
        4.WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。
    blocking（是否使用阻塞缓存）: 默认为false，当指定为true时将采用BlockingCache进行封装，blocking，阻塞的意思，
        使用BlockingCache会在查询缓存时锁住对应的Key，如果缓存命中了则会释放对应的锁，否则会在查询数据库以后再释放锁，
        这样可以阻止并发情况下多个线程同时查询数据，详情可参考BlockingCache的源码。
    type（缓存类）：可指定使用的缓存类，mybatis默认使用HashMap进行缓存,这里引用第三方中间件进行缓存
    -->
    <cache type="org.mybatis.caches.redis.RedisCache" blocking="false"
           flushInterval="0" readOnly="true" size="1024" eviction="FIFO"/>
 
    <!--
        useCache（是否使用缓存）：默认true使用缓存
    -->
    <select id="find" parameterType="map" resultType="com.chenhao.model.User" useCache="true">
        SELECT * FROM user
    </select>
 
</mapper>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**依然很简单， RedisCache 在保存缓存数据和获取缓存数据时，使用了Java的序列化和反序列化，因此需要保证被缓存的对象必须实现Serializable接口。**

也可以自己实现cache



### 实现自己的cache

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.chenhao.mybatis.cache;

import org.apache.ibatis.cache.Cache;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @author chenhao
 * @date 2019/10/31.
 */
public class RedisCache implements Cache {

    private final String id;

    private static ValueOperations<String, Object> valueOs;

    private static RedisTemplate<String, String> template;


    public static void setValueOs(ValueOperations<String, Object> valueOs) {
        RedisCache.valueOs = valueOs;
    }

    public static void setTemplate(RedisTemplate<String, String> template) {
        RedisCache.template = template;
    }

    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();


    public RedisCache(String id) {
        if (id == null) {
            throw new IllegalArgumentException("Cache instances require an ID");
        }
        this.id = id;
    }

    @Override
    public String getId() {
        return this.id;
    }

    @Override
    public void putObject(Object key, Object value) {
        valueOs.set(key.toString(), value, 10, TimeUnit.MINUTES);
    }

    @Override
    public Object getObject(Object key) {
        return valueOs.get(key.toString());
    }

    @Override
    public Object removeObject(Object key) {
        valueOs.set(key.toString(), "", 0, TimeUnit.MINUTES);
        return key;
    }

    @Override
    public void clear() {
        template.getConnectionFactory().getConnection().flushDb();
    }

    @Override
    public int getSize() {
        return template.getConnectionFactory().getConnection().dbSize().intValue();
    }

    @Override
    public ReadWriteLock getReadWriteLock() {
        return this.readWriteLock;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Mapper中配置自己实现的Cache

```
<cache type="com.chenhao.mybatis.cache.RedisCache"/> 
```

# [Mybaits 源码解析 （十）----- Spring-Mybatis框架使用与源码解析](https://www.cnblogs.com/java-chen-hao/p/11833780.html)

**正文**

在前面几篇文章中我们主要分析了Mybatis的单独使用，在实际在常规项目开发中，大部分都会使用mybatis与Spring结合起来使用，毕竟现在不用Spring开发的项目实在太少了。本篇文章便来介绍下Mybatis如何与Spring结合起来使用，并介绍下其源码是如何实现的。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11833780.html#_labelTop)

## Spring-Mybatis使用



### 添加maven依赖

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.8.RELEASE</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.2</version>
</dependency>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 在src/main/resources下添加mybatis-config.xml文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <typeAlias alias="User" type="com.chenhao.bean.User" />
    </typeAliases>
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <property name="helperDialect" value="mysql"/>
        </plugin>
    </plugins>

</configuration>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 在src/main/resources/mapper路径下添加User.xml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    
<mapper namespace="com.chenhao.mapper.UserMapper">
    <select id="getUser" parameterType="int"
        resultType="com.chenhao.bean.User">
        SELECT *
        FROM USER
        WHERE id = #{id}
    </select>
</mapper>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 在src/main/resources/路径下添加beans.xml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/test"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
 
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="configLocation" value="classpath:mybatis-config.xml"></property>
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath:mapper/*.xml" />
    </bean>
    
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.chenhao.mapper" />
    </bean>
 
</beans>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 注解的方式

- 以上分析都是在spring的XML配置文件applicationContext.xml进行配置的，mybatis-spring也提供了基于注解的方式来配置sqlSessionFactory和Mapper接口。
- sqlSessionFactory主要是在@Configuration注解的配置类中使用@Bean注解的名为sqlSessionFactory的方法来配置；
- Mapper接口主要是通过在@Configuration注解的配置类中结合@MapperScan注解来指定需要扫描获取mapper接口的包。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Configuration
@MapperScan("com.chenhao.mapper")
public class AppConfig {

  @Bean
  public DataSource dataSource() {
     return new EmbeddedDatabaseBuilder()
            .addScript("schema.sql")
            .build();
  }
 
  @Bean
  public DataSourceTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource());
  }
 
  @Bean
  public SqlSessionFactory sqlSessionFactory() throws Exception {
    //创建SqlSessionFactoryBean对象
    SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    //设置数据源
    sessionFactory.setDataSource(dataSource());
    //设置Mapper.xml路径
    sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
    // 设置MyBatis分页插件
    PageInterceptor pageInterceptor = new PageInterceptor();
    Properties properties = new Properties();
    properties.setProperty("helperDialect", "mysql");
    pageInterceptor.setProperties(properties);
    sessionFactory.setPlugins(new Interceptor[]{pageInterceptor});
    return sessionFactory.getObject();
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11833780.html#_labelTop)

## 对照Spring-Mybatis的方式，也就是对照beans.xml文件来看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis-config.xml"></property>
    <property name="dataSource" ref="dataSource" />
    <property name="mapperLocations" value="classpath:mapper/*.xml" />
</bean>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

就对应着SqlSessionFactory的生成，类似于原生Mybatis使用时的以下代码

```
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build( Resources.getResourceAsStream("mybatis-config.xml"));
```

而UserMapper代理对象的获取，是通过扫描的形式获取，也就是**MapperScannerConfigurer**这个类

```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.chenhao.mapper" />
</bean>
```

对应着Mapper接口的获取，类似于原生Mybatis使用时的以下代码：

```
SqlSession session = sqlSessionFactory.openSession();
UserMapper mapper = session.getMapper(UserMapper.class);
```

接着我们就可以在Service中直接从Spring的BeanFactory中获取了，如下

![img](https://img2018.cnblogs.com/blog/1168971/201911/1168971-20191101175402333-1796850150.png)

**所以我们现在就主要分析下在Spring中是如何生成SqlSessionFactory和Mapper接口的**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11833780.html#_labelTop)

## SqlSessionFactoryBean的设计与实现



### 大体思路

- mybatis-spring为了实现spring对mybatis的整合，即将mybatis的相关组件作为spring的IOC容器的bean来管理，使用了spring的FactoryBean接口来对mybatis的相关组件进行包装。spring的IOC容器在启动加载时，如果发现某个bean实现了FactoryBean接口，则会调用该bean的getObject方法，获取实际的bean对象注册到IOC容器，其中FactoryBean接口提供了getObject方法的声明，从而统一spring的IOC容器的行为。
- SqlSessionFactory作为mybatis的启动组件，在mybatis-spring中提供了SqlSessionFactoryBean来进行包装，所以在spring项目中整合mybatis，首先需要在spring的配置，如XML配置文件applicationContext.xml中，配置SqlSessionFactoryBean来引入SqlSessionFactory，即在spring项目启动时能加载并创建SqlSessionFactory对象，然后注册到spring的IOC容器中，从而可以直接在应用代码中注入使用或者作为属性，注入到mybatis的其他组件对应的bean对象。在applicationContext.xml的配置如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
       // 数据源
       <property name="dataSource" ref="dataSource" />
       // mapper.xml的资源文件,也就是SQL文件
       <property name="mapperLocations" value="classpath:mybatis/mapper/**/*.xml" />
       //mybatis配置mybatisConfig.xml的资源文件
       <property name="configLocation" value="classpath:mybatis/mybitas-config.xml" />
</bean>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 接口设计与实现

SqlSessionFactory的接口设计如下：实现了spring提供的**FactoryBean，InitializingBean和ApplicationListener这三个接口，在内部封装了mybatis的相关组件作为内部属性，如mybatisConfig.xml配置资源文件引用，mapper.xml配置资源文件引用，以及SqlSessionFactoryBuilder构造器和SqlSessionFactory引用。**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 解析mybatisConfig.xml文件和mapper.xml，设置数据源和所使用的事务管理机制，将这些封装到Configuration对象
// 使用Configuration对象作为构造参数，创建SqlSessionFactory对象，其中SqlSessionFactory为单例bean，最后将SqlSessionFactory单例对象注册到spring容器。
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {

  private static final Logger LOGGER = LoggerFactory.getLogger(SqlSessionFactoryBean.class);

  // mybatis配置mybatisConfig.xml的资源文件
  private Resource configLocation;

  //解析完mybatisConfig.xml后生成Configuration对象
  private Configuration configuration;

  // mapper.xml的资源文件
  private Resource[] mapperLocations;

  // 数据源
  private DataSource dataSource;

  // 事务管理，mybatis接入spring的一个重要原因也是可以直接使用spring提供的事务管理
  private TransactionFactory transactionFactory;

  private Properties configurationProperties;

  // mybatis的SqlSessionFactoryBuidler和SqlSessionFactory
  private SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();

  private SqlSessionFactory sqlSessionFactory;
  
  
  // 实现FactoryBean的getObject方法
  @Override
  public SqlSessionFactory getObject() throws Exception {
  
    //...

  }
  
  // 实现InitializingBean的
  @Override
  public void afterPropertiesSet() throws Exception {
  
    //...
    
  }
  // 为单例
  public boolean isSingleton() {
    return true;
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们重点关注FactoryBean，InitializingBean这两个接口，spring的IOC容器在加载创建SqlSessionFactoryBean的bean对象实例时，会调用InitializingBean的afterPropertiesSet方法进行对该bean对象进行相关初始化处理。



### InitializingBean的afterPropertiesSet方法

大家最好看一下我前面关于Spring源码的文章，有Bean的生命周期详细源码分析，我们现在简单回顾一下，在getBean()时initializeBean方法中调用InitializingBean的afterPropertiesSet，而在前一步操作populateBean中，以及将该bean对象实例的属性设值好了，InitializingBean的afterPropertiesSet进行一些后置处理。此时我们要注意，**populateBean方法已经将**SqlSessionFactoryBean对象的属性进行赋值了，也就是xml中property配置的dataSource，mapperLocations，configLocation这三个属性已经在SqlSessionFactoryBean对象的属性进行赋值了，后面调用afterPropertiesSet时直接可以使用这三个配置的值了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// bean对象实例创建的核心实现方法
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
        throws BeanCreationException {

    // 使用构造函数或者工厂方法来创建bean对象实例
    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    
    ...

    // 初始化bean对象实例，包括属性赋值，初始化方法，BeanPostProcessor的执行
    // Initialize the bean instance.
    Object exposedObject = bean;
    try {

        // 1. InstantiationAwareBeanPostProcessor执行:
        //     (1). 调用InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation,
        //  (2). 调用InstantiationAwareBeanPostProcessor的postProcessProperties和postProcessPropertyValues
        // 2. bean对象的属性赋值
        populateBean(beanName, mbd, instanceWrapper);

        // 1. Aware接口的方法调用
        // 2. BeanPostProcess执行：调用BeanPostProcessor的postProcessBeforeInitialization
        // 3. 调用init-method：首先InitializingBean的afterPropertiesSet，然后应用配置的init-method
        // 4. BeanPostProcess执行：调用BeanPostProcessor的postProcessAfterInitialization
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    

    // Register bean as disposable.
    try {
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    }
    catch (BeanDefinitionValidationException ex) {
        throw new BeanCreationException(
                mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
    }

    return exposedObject;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如上，在**populateBean阶段，**dataSource，mapperLocations，configLocation这三个属性已经在SqlSessionFactoryBean对象的属性进行赋值了，调用afterPropertiesSet时直接可以使用这三个配置的值了。那我们来接着看看afterPropertiesSet方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void afterPropertiesSet() throws Exception {
    notNull(dataSource, "Property 'dataSource' is required");
    notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
    state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
              "Property 'configuration' and 'configLocation' can not specified with together");

    // 创建sqlSessionFactory
    this.sqlSessionFactory = buildSqlSessionFactory();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

SqlSessionFactoryBean的afterPropertiesSet方法实现如下：调用buildSqlSessionFactory方法创建用于注册到spring的IOC容器的sqlSessionFactory对象。我们接着来看看**buildSqlSessionFactory**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected SqlSessionFactory buildSqlSessionFactory() throws IOException {

    // 配置类
   Configuration configuration;
    // 解析mybatis-Config.xml文件，
    // 将相关配置信息保存到configuration
   XMLConfigBuilder xmlConfigBuilder = null;
   if (this.configuration != null) {
     configuration = this.configuration;
     if (configuration.getVariables() == null) {
       configuration.setVariables(this.configurationProperties);
     } else if (this.configurationProperties != null) {
       configuration.getVariables().putAll(this.configurationProperties);
     }
    //资源文件不为空
   } else if (this.configLocation != null) {
     //根据configLocation创建xmlConfigBuilder，XMLConfigBuilder构造器中会创建Configuration对象
     xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);
     //将XMLConfigBuilder构造器中创建的Configuration对象直接赋值给configuration属性
     configuration = xmlConfigBuilder.getConfiguration();
   } 
   
    //略....

   if (xmlConfigBuilder != null) {
     try {
       //解析mybatis-Config.xml文件，并将相关配置信息保存到configuration
       xmlConfigBuilder.parse();
       if (LOGGER.isDebugEnabled()) {
         LOGGER.debug("Parsed configuration file: '" + this.configLocation + "'");
       }
     } catch (Exception ex) {
       throw new NestedIOException("Failed to parse config resource: " + this.configLocation, ex);
     }
   }
    
   if (this.transactionFactory == null) {
     //事务默认采用SpringManagedTransaction，这一块非常重要，我将在后买你单独写一篇文章讲解Mybatis和Spring事务的关系
     this.transactionFactory = new SpringManagedTransactionFactory();
   }
    // 为sqlSessionFactory绑定事务管理器和数据源
    // 这样sqlSessionFactory在创建sqlSession的时候可以通过该事务管理器获取jdbc连接，从而执行SQL
   configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
    // 解析mapper.xml
   if (!isEmpty(this.mapperLocations)) {
     for (Resource mapperLocation : this.mapperLocations) {
       if (mapperLocation == null) {
         continue;
       }
       try {
         // 解析mapper.xml文件，并注册到configuration对象的mapperRegistry
         XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
             configuration, mapperLocation.toString(), configuration.getSqlFragments());
         xmlMapperBuilder.parse();
       } catch (Exception e) {
         throw new NestedIOException("Failed to parse mapping resource: '" + mapperLocation + "'", e);
       } finally {
         ErrorContext.instance().reset();
       }

       if (LOGGER.isDebugEnabled()) {
         LOGGER.debug("Parsed mapper file: '" + mapperLocation + "'");
       }
     }
   } else {
     if (LOGGER.isDebugEnabled()) {
       LOGGER.debug("Property 'mapperLocations' was not specified or no matching resources found");
     }
   }

    // 将Configuration对象实例作为参数，
    // 调用sqlSessionFactoryBuilder创建sqlSessionFactory对象实例
   return this.sqlSessionFactoryBuilder.build(configuration);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

buildSqlSessionFactory的核心逻辑：解析mybatis配置文件mybatisConfig.xml和mapper配置文件mapper.xml并封装到Configuration对象中，最后调用mybatis的sqlSessionFactoryBuilder来创建SqlSessionFactory对象。这一点相当于前面介绍的原生的mybatis的初始化过程。另外，当**配置中未指定事务时，mybatis-spring默认采用SpringManagedTransaction，这一点非常重要，请大家先在心里做好准备**。此时SqlSessionFactory已经创建好了，并且赋值到了SqlSessionFactoryBean的sqlSessionFactory属性中。



### FactoryBean的getObject方法定义

**FactoryBean：创建某个类的对象实例的工厂。**

spring的IOC容器在启动，创建好bean对象实例后，会检查这个bean对象是否实现了FactoryBean接口，如果是，则调用该bean对象的getObject方法，在getObject方法中实现创建并返回实际需要的bean对象实例，然后将该实际需要的bean对象实例注册到spring容器；如果不是则直接将该bean对象实例注册到spring容器。

SqlSessionFactoryBean的getObject方法实现如下：由于spring在创建SqlSessionFactoryBean自身的bean对象时，已经调用了InitializingBean的afterPropertiesSet方法创建了sqlSessionFactory对象，故可以直接返回sqlSessionFactory对象给spring的IOC容器，从而完成sqlSessionFactory的bean对象的注册，之后可以直接在应用代码注入或者spring在创建其他bean对象时，依赖注入sqlSessionFactory对象。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public SqlSessionFactory getObject() throws Exception {
    if (this.sqlSessionFactory == null) {
      afterPropertiesSet();
    }
    // 直接返回sqlSessionFactory对象
    // 单例对象，由所有mapper共享
    return this.sqlSessionFactory;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11833780.html#_labelTop)

## 总结

由以上分析可知，spring在加载创建SqlSessionFactoryBean的bean对象实例时，调用SqlSessionFactoryBean的afterPropertiesSet方法完成了sqlSessionFactory对象实例的创建；在将SqlSessionFactoryBean对象实例注册到spring的IOC容器时，发现SqlSessionFactoryBean实现了FactoryBean接口，故不是SqlSessionFactoryBean对象实例自身需要注册到spring的IOC容器，而是SqlSessionFactoryBean的getObject方法的返回值对应的对象需要注册到spring的IOC容器，而这个返回值就是SqlSessionFactory对象，故完成了将sqlSessionFactory对象实例注册到spring的IOC容器。创建Mapper的代理对象我们下一篇文章再讲

# [Mybaits 源码解析 （十一）----- @MapperScan将Mapper接口生成代理注入到Spring-静态代理和动态代理结合使用](https://www.cnblogs.com/java-chen-hao/p/11839958.html)

**正文**

上一篇文章我们讲了SqlSessionFactoryBean，通过这个FactoryBean创建SqlSessionFactory并注册进Spring容器，这篇文章我们就讲剩下的部分，通过MapperScannerConfigurer将Mapper接口生成代理注入到Spring

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11839958.html#_labelTop)

## 扫描Mapper接口

我们上一篇文章介绍了扫描Mapper接口有两种方式，一种是通过bean.xml注册MapperScannerConfigurer对象，一种是通过@MapperScan("com.chenhao.mapper")注解的方式，如下

方式一：

```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.chenhao.mapper" />
</bean>
```

方式二：

```
@Configuration
@MapperScan("com.chenhao.mapper")
public class AppConfig {
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11839958.html#_labelTop)

## @MapperScan

我们来看看**@MapperScan**这个注解

```
@Import(MapperScannerRegistrar.class)
public @interface MapperScan {
```

MapperScan使用`@Import`将`MapperScannerRegistrar`导入。



### MapperScannerRegistrar

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MapperScannerRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware {
 2   private ResourceLoader resourceLoader;
 3   @Override
 4   public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
 5     // 获取MapperScan 注解，如@MapperScan("com.chenhao.mapper")
 6     AnnotationAttributes annoAttrs = AnnotationAttributes.fromMap(importingClassMetadata.getAnnotationAttributes(MapperScan.class.getName()));
 7     // 创建路径扫描器，下面的一大段都是将MapperScan 中的属性设置到ClassPathMapperScanner ，做的就是一个set操作
 8     ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
 9     // this check is needed in Spring 3.1
10     if (resourceLoader != null) {
11       // 设置资源加载器，作用：扫描指定包下的class文件。
12       scanner.setResourceLoader(resourceLoader);
13     }
14     Class<? extends Annotation> annotationClass = annoAttrs.getClass("annotationClass");
15     if (!Annotation.class.equals(annotationClass)) {
16       scanner.setAnnotationClass(annotationClass);
17     }
18     Class<?> markerInterface = annoAttrs.getClass("markerInterface");
19     if (!Class.class.equals(markerInterface)) {
20       scanner.setMarkerInterface(markerInterface);
21     }
22     Class<? extends BeanNameGenerator> generatorClass = annoAttrs.getClass("nameGenerator");
23     if (!BeanNameGenerator.class.equals(generatorClass)) {
24       scanner.setBeanNameGenerator(BeanUtils.instantiateClass(generatorClass));
25     }
26     Class<? extends MapperFactoryBean> mapperFactoryBeanClass = annoAttrs.getClass("factoryBean");
27     if (!MapperFactoryBean.class.equals(mapperFactoryBeanClass)) {
28       scanner.setMapperFactoryBean(BeanUtils.instantiateClass(mapperFactoryBeanClass));
29     }
30     scanner.setSqlSessionTemplateBeanName(annoAttrs.getString("sqlSessionTemplateRef"));
31     //设置SqlSessionFactory的名称
32     scanner.setSqlSessionFactoryBeanName(annoAttrs.getString("sqlSessionFactoryRef"));
33     List<String> basePackages = new ArrayList<String>();
34     //获取配置的包路径，如com.chenhao.mapper
35     for (String pkg : annoAttrs.getStringArray("value")) {
36       if (StringUtils.hasText(pkg)) {
37         basePackages.add(pkg);
38       }
39     }
40     for (String pkg : annoAttrs.getStringArray("basePackages")) {
41       if (StringUtils.hasText(pkg)) {
42         basePackages.add(pkg);
43       }
44     }
45     for (Class<?> clazz : annoAttrs.getClassArray("basePackageClasses")) {
46       basePackages.add(ClassUtils.getPackageName(clazz));
47     }
48     // 注册过滤器，作用：什么类型的Mapper将会留下来。
49     scanner.registerFilters();
50     // 扫描指定包
51     scanner.doScan(StringUtils.toStringArray(basePackages));
52   }
53 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### ClassPathMapperScanner

接着我们来看看扫描过程 **scanner.doScan(StringUtils.toStringArray(basePackages));**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 @Override
 2 public Set<BeanDefinitionHolder> doScan(String... basePackages) {
 3     //调用父类进行扫描，并将basePackages下的class都封装成BeanDefinitionHolder，并注册进Spring容器的BeanDefinition
 4     Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
 5  
 6     if (beanDefinitions.isEmpty()) {
 7       logger.warn("No MyBatis mapper was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.");
 8     } else {
 9       //继续对beanDefinitions做处理，额外设置一些属性
10       processBeanDefinitions(beanDefinitions);
11     }
12  
13     return beanDefinitions;
14 }
15   
16 protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
17     Assert.notEmpty(basePackages, "At least one base package must be specified");
18     Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
19     //遍历basePackages进行扫描
20     for (String basePackage : basePackages) {
21         //找出匹配的类
22         Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
23         for (BeanDefinition candidate : candidates) {
24             ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
25             candidate.setScope(scopeMetadata.getScopeName());
26             String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
27             if (candidate instanceof AbstractBeanDefinition) {
28                 postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
29             }
30             if (candidate instanceof AnnotatedBeanDefinition) {
31                 AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
32             }
33             if (checkCandidate(beanName, candidate)) {
34                 //封装成BeanDefinitionHolder 对象
35                 BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
36                 definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
37                 beanDefinitions.add(definitionHolder);
38                 //将BeanDefinition对象注入spring的BeanDefinitionMap中，后续getBean时，就是从BeanDefinitionMap获取对应的BeanDefinition对象，取出其属性进行实例化Bean
39                 registerBeanDefinition(definitionHolder, this.registry);
40             }
41         }
42     }
43     return beanDefinitions;
44 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们重点看下第4行和第10行代码，第4行是调用父类的doScan方法，获取basePackages下的所有Class，并将其生成**BeanDefinition，**注入spring的**BeanDefini**tionMap中，也就是Class的描述类，在调用getBean方法时，获取BeanDefinition进行实例化。此时，所有的Mapper接口已经被生成了BeanDefinition。接着我们看下第10行，对生成的BeanDefinition做一些额外的处理。



### **processBeanDefinitions**

上面BeanDefinition已经注入进Spring容器了，接着我们看对BeanDefinition进行哪些额外的处理

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void processBeanDefinitions(Set<BeanDefinitionHolder> beanDefinitions) {
 2     GenericBeanDefinition definition;
 3     for (BeanDefinitionHolder holder : beanDefinitions) {
 4       definition = (GenericBeanDefinition) holder.getBeanDefinition();
 5  
 6       if (logger.isDebugEnabled()) {
 7         logger.debug("Creating MapperFactoryBean with name '" + holder.getBeanName() 
 8           + "' and '" + definition.getBeanClassName() + "' mapperInterface");
 9       }
10  
11       // 设置definition构造器的输入参数为definition.getBeanClassName()，这里就是com.chenhao.mapper.UserMapper
12       // 那么在getBean实例化时，通过反射调用构造器实例化时要将这个参数传进去
13       definition.getConstructorArgumentValues().addGenericArgumentValue(definition.getBeanClassName())
14       // 修改definition对应的Class
15       // 看过Spring源码的都知道，getBean()返回的就是BeanDefinitionHolder中beanClass属性对应的实例
16       // 所以我们后面ac.getBean(UserMapper.class)的返回值也就是MapperFactoryBean的实例
17       // 但是最终被注入到Spring容器的对象的并不是MapperFactoryBean的实例，根据名字看，我们就知道MapperFactoryBean实现了FactoryBean接口
18       // 那么最终注入Spring容器的必定是从MapperFactoryBean的实例的getObject()方法中返回
19       definition.setBeanClass(this.mapperFactoryBean.getClass());
20  
21       definition.getPropertyValues().add("addToConfig", this.addToConfig);
22  
23       boolean explicitFactoryUsed = false;
24       if (StringUtils.hasText(this.sqlSessionFactoryBeanName)) {
25         definition.getPropertyValues().add("sqlSessionFactory", new RuntimeBeanReference(this.sqlSessionFactoryBeanName));
26         explicitFactoryUsed = true;
27       } else if (this.sqlSessionFactory != null) {
28         //往definition设置属性值sqlSessionFactory，那么在MapperFactoryBean实例化后，进行属性赋值时populateBean(beanName, mbd, instanceWrapper);，会通过反射调用sqlSessionFactory的set方法进行赋值
29         //也就是在MapperFactoryBean创建实例后，要调用setSqlSessionFactory方法将sqlSessionFactory注入进MapperFactoryBean实例
30         definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);
31         explicitFactoryUsed = true;
32       }
33  
34       if (StringUtils.hasText(this.sqlSessionTemplateBeanName)) {
35         if (explicitFactoryUsed) {
36           logger.warn("Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
37         }
38         definition.getPropertyValues().add("sqlSessionTemplate", new RuntimeBeanReference(this.sqlSessionTemplateBeanName));
39         explicitFactoryUsed = true;
40       } else if (this.sqlSessionTemplate != null) {
41         if (explicitFactoryUsed) {
42           logger.warn("Cannot use both: sqlSessionTemplate and sqlSessionFactory together. sqlSessionFactory is ignored.");
43         }
44         definition.getPropertyValues().add("sqlSessionTemplate", this.sqlSessionTemplate);
45         explicitFactoryUsed = true;
46       }
47  
48       if (!explicitFactoryUsed) {
49         if (logger.isDebugEnabled()) {
50           logger.debug("Enabling autowire by type for MapperFactoryBean with name '" + holder.getBeanName() + "'.");
51         }
52         definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
53       }
54     }
55 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看第19行，将**definitio**n的beanClass属性设置为MapperFactoryBean.class，我们知道，在getBean的时候，会通过反射创建Bean的实例，也就是创建beanClass的实例，如下Spring的getBean的部分代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
    // 没有覆盖
    // 直接使用反射实例化即可
    if (!bd.hasMethodOverrides()) {
        // 重新检测获取下构造函数
        // 该构造函数是经过前面 N 多复杂过程确认的构造函数
        Constructor<?> constructorToUse;
        synchronized (bd.constructorArgumentLock) {
            // 获取已经解析的构造函数
            constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
            // 如果为 null，从 class 中解析获取，并设置
            if (constructorToUse == null) {
                final Class<?> clazz = bd.getBeanClass();
                if (clazz.isInterface()) {
                    throw new BeanInstantiationException(clazz, "Specified class is an interface");
                }
                try {
                    if (System.getSecurityManager() != null) {
                        constructorToUse = AccessController.doPrivileged(
                                (PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
                    }
                    else {
                        //利用反射获取构造器
                        constructorToUse =  clazz.getDeclaredConstructor();
                    }
                    bd.resolvedConstructorOrFactoryMethod = constructorToUse;
                }
                catch (Throwable ex) {
                    throw new BeanInstantiationException(clazz, "No default constructor found", ex);
                }
            }
        }

        // 通过BeanUtils直接使用构造器对象实例化bean
        return BeanUtils.instantiateClass(constructorToUse);
    }
    else {
        // 生成CGLIB创建的子类对象
        return instantiateWithMethodInjection(bd, beanName, owner);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看到没，是通过**bd.getBeanClass();从**BeanDefinition中获取beanClass属性，然后通过反射实例化Bean，如上，所有的Mapper接口扫描封装成的BeanDefinition的beanClass都设置成了MapperFactoryBean，我们知道在Spring初始化的最后，会获取所有的BeanDefinition，并通过getBean创建所有的实例注入进Spring容器，那么意思就是说，在getBean时，创建的的所有Mapper对象是MapperFactoryBean这个对象了。

**我们看下第13行处，设置了BeanDefinition构造器参数，那么当getBean中通过构造器创建实例时，需要将设置的构造器参数definition.getBeanClassName()，这里就是com.chenhao.mapper.UserMapper传进去。**

还有一个点要注意，在第30行处，往BeanDefinition的PropertyValues设置了sqlSessionFactory，那么在创建完MapperFactoryBean的实例后，会对MapperFactoryBean进行属性赋值，也就是Spring创建Bean的这句代码，**populateBean(beanName, mbd, instanceWrapper);**，这里会通过反射调用MapperFactoryBean的**setSqlSessionFactory方法将sqlSessionFactory注入进MapperFactoryBean实例，所以我们接下来重点看的就是****MapperFactoryBean这个对象了。**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11839958.html#_labelTop)

## MapperFactoryBean

接下来我们看最重要的一个类MapperFactoryBean

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//继承SqlSessionDaoSupport、实现FactoryBean，那么最终注入Spring容器的对象要从getObject()中取
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {
    private Class<T> mapperInterface;
    private boolean addToConfig = true;

    public MapperFactoryBean() {
    }

    //构造器，我们上一节中在BeanDefinition中已经设置了构造器输入参数
    //所以在通过反射调用构造器实例化时，会获取在BeanDefinition设置的构造器输入参数
    //也就是对应得每个Mapper接口Class
    public MapperFactoryBean(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    protected void checkDaoConfig() {
        super.checkDaoConfig();
        Assert.notNull(this.mapperInterface, "Property 'mapperInterface' is required");
        Configuration configuration = this.getSqlSession().getConfiguration();
        if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
            try {
                configuration.addMapper(this.mapperInterface);
            } catch (Exception var6) {
                this.logger.error("Error while adding the mapper '" + this.mapperInterface + "' to configuration.", var6);
                throw new IllegalArgumentException(var6);
            } finally {
                ErrorContext.instance().reset();
            }
        }

    }
    //最终注入Spring容器的就是这里的返回对象
    public T getObject() throws Exception {
        //获取父类setSqlSessionFactory方法中创建的SqlSessionTemplate
        //通过SqlSessionTemplate获取mapperInterface的代理类
        //我们例子中就是通过SqlSessionTemplate获取com.chenhao.mapper.UserMapper的代理类
        //获取到Mapper接口的代理类后，就把这个Mapper的代理类对象注入Spring容器
        return this.getSqlSession().getMapper(this.mapperInterface);
    }

    public Class<T> getObjectType() {
        return this.mapperInterface;
    }

    public boolean isSingleton() {
        return true;
    }

    public void setMapperInterface(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    public Class<T> getMapperInterface() {
        return this.mapperInterface;
    }

    public void setAddToConfig(boolean addToConfig) {
        this.addToConfig = addToConfig;
    }

    public boolean isAddToConfig() {
        return this.addToConfig;
    }
}


public abstract class SqlSessionDaoSupport extends DaoSupport {
    private SqlSession sqlSession;
    private boolean externalSqlSession;

    public SqlSessionDaoSupport() {
    }
    //还记得上一节中我们往BeanDefinition中设置的sqlSessionFactory这个属性吗？
    //在实例化MapperFactoryBean后，进行属性赋值时，就会通过反射调用setSqlSessionFactory
    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
        if (!this.externalSqlSession) {
            //创建一个SqlSessionTemplate并赋值给sqlSession
            this.sqlSession = new SqlSessionTemplate(sqlSessionFactory);
        }
    }

    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSession = sqlSessionTemplate;
        this.externalSqlSession = true;
    }

    public SqlSession getSqlSession() {
        return this.sqlSession;
    }

    protected void checkDaoConfig() {
        Assert.notNull(this.sqlSession, "Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到MapperFactoryBean **extends SqlSessionDaoSupport implements FactoryBean，那么getBean获取的对象是从其getObject()中获取，并且MapperFactoryBean是一个单例，那么其中的属性****SqlSessionTemplate对象也是一个单例，全局唯一，供所有的Mapper代理类使用。**

这里我大概讲一下getBean时，这个类的过程：

1、MapperFactoryBean通过反射调用构造器实例化出一个对象，并且通过上一节中definition.getConstructorArgumentValues().addGenericArgumentValue(definition.getBeanClassName())设置的构造器参数，在构造器实例化时，传入Mapper接口的Class,并设置为MapperFactoryBean的mapperInterface属性。

2、进行属性赋值，通过上一节中definition.getPropertyValues().add("sqlSessionFactory", this.sqlSessionFactory);设置的属性值，在populateBean属性赋值过程中通过反射调用setSqlSessionFactory方法，并创建SqlSessionTemplate对象设置到sqlSession属性中。

3、由于MapperFactoryBean实现了FactoryBean，最终注册进Spring容器的对象是从getObject()方法中取，接着获取SqlSessionTemplate这个SqlSession调用getMapper(this.mapperInterface);生成Mapper接口的代理对象，将Mapper接口的代理对象注册进Spring容器

至此，所有com.chenhao.mapper中的Mapper接口都生成了代理类，并注入到Spring容器了。接着我们就可以在Service中直接从Spring的BeanFactory中获取了，如下

![img](https://img2018.cnblogs.com/i-beta/1168971/201911/1168971-20191102151929081-185356019.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11839958.html#_labelTop)

## SqlSessionTemplate

还记得我们前面分析Mybatis源码时，获取的SqlSession实例是什么吗？我们简单回顾一下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * ExecutorType 指定Executor的类型，分为三种：SIMPLE, REUSE, BATCH，默认使用的是SIMPLE
 * TransactionIsolationLevel 指定事务隔离级别，使用null,则表示使用数据库默认的事务隔离界别
 * autoCommit 是否自动提交，传过来的参数为false，表示不自动提交
 */
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
        // 获取配置中的环境信息，包括了数据源信息、事务等
        final Environment environment = configuration.getEnvironment();
        // 创建事务工厂
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
        // 创建事务，配置事务属性
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        // 创建Executor，即执行器
        // 它是真正用来Java和数据库交互操作的类，后面会展开说。
        final Executor executor = configuration.newExecutor(tx, execType);
        // 创建DefaultSqlSession对象返回，其实现了SqlSession接口
        return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
        closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

大家应该还有印象，就是上面的DefaultSqlSession，那上一节的SqlSessionTemplate是什么鬼？？？我们来看看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 实现SqlSession接口，单例、线程安全，使用spring的事务管理器的sqlSession，
// 具体的SqlSession的功能，则是通过内部包含的sqlSessionProxy来来实现，这也是静态代理的一种实现。
// 同时内部的sqlSessionProxy实现InvocationHandler接口，则是动态代理的一种实现，而线程安全也是在这里实现的。
// 注意mybatis默认的sqlSession不是线程安全的，需要每个线程有一个单例的对象实例。
// SqlSession的主要作用是提供SQL操作的API，执行指定的SQL语句，mapper需要依赖SqlSession来执行其方法对应的SQL。
public class SqlSessionTemplate implements SqlSession, DisposableBean {
    private final SqlSessionFactory sqlSessionFactory;
    private final ExecutorType executorType;
    // 一个代理类，由于SqlSessionTemplate为单例的，被所有mapper，所有线程共享，
    // 所以sqlSessionProxy要保证这些mapper中方法调用的线程安全特性：
    // sqlSessionProxy的实现方式主要为实现了InvocationHandler接口实现了动态代理，
    // 由动态代理的知识可知，InvocationHandler的invoke方法会拦截所有mapper的所有方法调用，
    // 故这里的实现方式是在invoke方法内部创建一个sqlSession局部变量，从而实现了每个mapper的每个方法调用都使用
    private final SqlSession sqlSessionProxy;
    private final PersistenceExceptionTranslator exceptionTranslator;

    public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        this(sqlSessionFactory, sqlSessionFactory.getConfiguration().getDefaultExecutorType());
    }

    public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType) {
        this(sqlSessionFactory, executorType, new MyBatisExceptionTranslator(sqlSessionFactory.getConfiguration().getEnvironment().getDataSource(), true));
    }

    public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
        Assert.notNull(sqlSessionFactory, "Property 'sqlSessionFactory' is required");
        Assert.notNull(executorType, "Property 'executorType' is required");
        this.sqlSessionFactory = sqlSessionFactory;
        this.executorType = executorType;
        this.exceptionTranslator = exceptionTranslator;
        this.sqlSessionProxy = (SqlSession)Proxy.newProxyInstance(SqlSessionFactory.class.getClassLoader(), new Class[]{SqlSession.class}, new SqlSessionTemplate.SqlSessionInterceptor());
    }

    public SqlSessionFactory getSqlSessionFactory() {
        return this.sqlSessionFactory;
    }

    public ExecutorType getExecutorType() {
        return this.executorType;
    }

    public PersistenceExceptionTranslator getPersistenceExceptionTranslator() {
        return this.exceptionTranslator;
    }

    public <T> T selectOne(String statement) {
        //由真实的对象sqlSessionProxy执行查询
        return this.sqlSessionProxy.selectOne(statement);
    }

    public <T> T selectOne(String statement, Object parameter) {
        //由真实的对象sqlSessionProxy执行查询
        return this.sqlSessionProxy.selectOne(statement, parameter);
    }

    public <K, V> Map<K, V> selectMap(String statement, String mapKey) {
        //由真实的对象sqlSessionProxy执行查询
        return this.sqlSessionProxy.selectMap(statement, mapKey);
    }

    public <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey) {
        //由真实的对象sqlSessionProxy执行查询
        return this.sqlSessionProxy.selectMap(statement, parameter, mapKey);
    }

    public <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowBounds) {
        //由真实的对象sqlSessionProxy执行查询
        return this.sqlSessionProxy.selectMap(statement, parameter, mapKey, rowBounds);
    }

    public <T> Cursor<T> selectCursor(String statement) {
        return this.sqlSessionProxy.selectCursor(statement);
    }

    public <T> Cursor<T> selectCursor(String statement, Object parameter) {
        return this.sqlSessionProxy.selectCursor(statement, parameter);
    }

    public <T> Cursor<T> selectCursor(String statement, Object parameter, RowBounds rowBounds) {
        return this.sqlSessionProxy.selectCursor(statement, parameter, rowBounds);
    }

    public <E> List<E> selectList(String statement) {
        return this.sqlSessionProxy.selectList(statement);
    }

    public <E> List<E> selectList(String statement, Object parameter) {
        return this.sqlSessionProxy.selectList(statement, parameter);
    }

    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        return this.sqlSessionProxy.selectList(statement, parameter, rowBounds);
    }

    public void select(String statement, ResultHandler handler) {
        this.sqlSessionProxy.select(statement, handler);
    }

    public void select(String statement, Object parameter, ResultHandler handler) {
        this.sqlSessionProxy.select(statement, parameter, handler);
    }

    public void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {
        this.sqlSessionProxy.select(statement, parameter, rowBounds, handler);
    }

    public int insert(String statement) {
        return this.sqlSessionProxy.insert(statement);
    }

    public int insert(String statement, Object parameter) {
        return this.sqlSessionProxy.insert(statement, parameter);
    }

    public int update(String statement) {
        return this.sqlSessionProxy.update(statement);
    }

    public int update(String statement, Object parameter) {
        return this.sqlSessionProxy.update(statement, parameter);
    }

    public int delete(String statement) {
        return this.sqlSessionProxy.delete(statement);
    }

    public int delete(String statement, Object parameter) {
        return this.sqlSessionProxy.delete(statement, parameter);
    }

    public <T> T getMapper(Class<T> type) {
        return this.getConfiguration().getMapper(type, this);
    }

    public void commit() {
        throw new UnsupportedOperationException("Manual commit is not allowed over a Spring managed SqlSession");
    }

    public void commit(boolean force) {
        throw new UnsupportedOperationException("Manual commit is not allowed over a Spring managed SqlSession");
    }

    public void rollback() {
        throw new UnsupportedOperationException("Manual rollback is not allowed over a Spring managed SqlSession");
    }

    public void rollback(boolean force) {
        throw new UnsupportedOperationException("Manual rollback is not allowed over a Spring managed SqlSession");
    }

    public void close() {
        throw new UnsupportedOperationException("Manual close is not allowed over a Spring managed SqlSession");
    }

    public void clearCache() {
        this.sqlSessionProxy.clearCache();
    }

    public Configuration getConfiguration() {
        return this.sqlSessionFactory.getConfiguration();
    }

    public Connection getConnection() {
        return this.sqlSessionProxy.getConnection();
    }

    public List<BatchResult> flushStatements() {
        return this.sqlSessionProxy.flushStatements();
    }

    public void destroy() throws Exception {
    }

    private class SqlSessionInterceptor implements InvocationHandler {
        private SqlSessionInterceptor() {
        }

        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            SqlSession sqlSession = SqlSessionUtils.getSqlSession(SqlSessionTemplate.this.sqlSessionFactory, SqlSessionTemplate.this.executorType, SqlSessionTemplate.this.exceptionTranslator);

            Object unwrapped;
            try {
                Object result = method.invoke(sqlSession, args);
                if (!SqlSessionUtils.isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
                    sqlSession.commit(true);
                }

                unwrapped = result;
            } catch (Throwable var11) {
                unwrapped = ExceptionUtil.unwrapThrowable(var11);
                if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
                    SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
                    sqlSession = null;
                    Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException)unwrapped);
                    if (translated != null) {
                        unwrapped = translated;
                    }
                }

                throw (Throwable)unwrapped;
            } finally {
                if (sqlSession != null) {
                    SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
                }

            }

            return unwrapped;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到SqlSessionTemplate实现了**SqlSession接口，那么Mapper代理类中执行所有的数据库操作，都是通过SqlSessionTemplate来执行，如上我们看到所有的数据库操作都由****对象sqlSessionProxy执行查询**



### 静态代理的使用

SqlSessionTemplate在内部访问数据库时，其实是委派给**sqlSessionProxy**来执行数据库操作的，SqlSessionTemplate不是自身重新实现了一套mybatis数据库访问的逻辑。

SqlSessionTemplate通过静态代理机制来提供SqlSession接口的行为，即实现SqlSession接口来获取SqlSession的所有方法；SqlSessionTemplate的定义如下：标准的静态代理实现模式，即实现SqlSession接口并在内部包含一个SqlSession接口实现类引用sqlSessionProxy。那我们就要看看sqlSessionProxy这个SqlSession，我们先来看看SqlSessionTemplate的构造方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
        Assert.notNull(sqlSessionFactory, "Property 'sqlSessionFactory' is required");
        Assert.notNull(executorType, "Property 'executorType' is required");
        this.sqlSessionFactory = sqlSessionFactory;
        this.executorType = executorType;
        this.exceptionTranslator = exceptionTranslator;
        this.sqlSessionProxy = (SqlSession)Proxy.newProxyInstance(SqlSessionFactory.class.getClassLoader(), new Class[]{SqlSession.class}, new SqlSessionTemplate.SqlSessionInterceptor());
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 动态代理的使用

不是吧，又使用了动态代理？？真够曲折的，那我们接着看 new SqlSessionTemplate.SqlSessionInterceptor() 这个InvocationHandler

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private class SqlSessionInterceptor implements InvocationHandler {
    //很奇怪，这里并没有真实目标对象？
    private SqlSessionInterceptor() {
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 获取一个sqlSession来执行proxy的method对应的SQL,
        // 每次调用都获取创建一个sqlSession线程局部变量，故不同线程相互不影响，在这里实现了SqlSessionTemplate的线程安全性
        SqlSession sqlSession = SqlSessionUtils.getSqlSession(SqlSessionTemplate.this.sqlSessionFactory, SqlSessionTemplate.this.executorType, SqlSessionTemplate.this.exceptionTranslator);

        Object unwrapped;
        try {
            //直接通过新创建的SqlSession反射调用method
            //这也就解释了为什么不需要目标类属性了，这里每次都会创建一个
            Object result = method.invoke(sqlSession, args);
            // 如果当前操作没有在一个Spring事务中，则手动commit一下
            // 如果当前业务没有使用@Transation,那么每次执行了Mapper接口的方法直接commit
            // 还记得我们前面讲的Mybatis的一级缓存吗，这里一级缓存不能起作用了，因为每执行一个Mapper的方法，sqlSession都提交了
            // sqlSession提交，会清空一级缓存
            if (!SqlSessionUtils.isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
                sqlSession.commit(true);
            }

            unwrapped = result;
        } catch (Throwable var11) {
            unwrapped = ExceptionUtil.unwrapThrowable(var11);
            if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
                SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
                sqlSession = null;
                Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException)unwrapped);
                if (translated != null) {
                    unwrapped = translated;
                }
            }

            throw (Throwable)unwrapped;
        } finally {
            if (sqlSession != null) {
                SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
            }

        }
        return unwrapped;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里大概讲一下Mapper代理类调用方法执行逻辑：

1、SqlSessionTemplate生成Mapper代理类时，将本身传进去做为Mapper代理类的属性，调用Mapper代理类的方法时，最后会通过SqlSession类执行，也就是调用SqlSessionTemplate中的方法。

2、SqlSessionTemplate中操作数据库的方法中又交给了**sqlSessionProxy**这个代理类去执行，那么每次执行的方法都会回调其SqlSessionInterceptor这个InvocationHandler的invoke方法

3、在invoke方法中，为每个线程创建一个新的SqlSession，并通过反射调用SqlSession的method。这里sqlSession是一个线程局部变量，不同线程相互不影响，实现了SqlSessionTemplate的线程安全性

4、如果当前操作并没有在Spring事务中，那么每次执行一个方法，都会提交，相当于数据库的事务自动提交，Mysql的一级缓存也将不可用

接下来我们还要看一个地方，invoke中是如何创建**SqlSession的？**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
    Assert.notNull(sessionFactory, "No SqlSessionFactory specified");
    Assert.notNull(executorType, "No ExecutorType specified");
    //通过TransactionSynchronizationManager内部的ThreadLocal中获取
    SqlSessionHolder holder = (SqlSessionHolder)TransactionSynchronizationManager.getResource(sessionFactory);
    SqlSession session = sessionHolder(executorType, holder);
    if(session != null) {
        return session;
    } else {
        if(LOGGER.isDebugEnabled()) {
            LOGGER.debug("Creating a new SqlSession");
        }
        //这里我们知道实际上是创建了一个DefaultSqlSession
        session = sessionFactory.openSession(executorType);
        //将创建的SqlSession对象放入TransactionSynchronizationManager内部的ThreadLocal中
        registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);
        return session;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过**sessionFactory.openSession(executorType)实际创建的****SqlSession还是****DefaultSqlSession。**如果读过我前面Spring源码的朋友，肯定知道TransactionSynchronizationManager这个类，其内部维护了一个**ThreadLocal的**Map，这里同一线程创建了**SqlSession后放入\**ThreadLocal中，同一线程中其他Mapper接口调用方法时，将会直接从\*\*ThreadLocal中获取。\*\**\***



# [Mybaits 源码解析 （十二）----- Mybatis的事务如何被Spring管理？Mybatis和Spring事务中用的Connection是同一个吗？](https://www.cnblogs.com/java-chen-hao/p/11839993.html)

**正文**

不知道一些同学有没有这种疑问，为什么Mybtis中要配置dataSource，Spring的事务中也要配置dataSource？那么Mybatis和Spring事务中用的Connection是同一个吗？我们常用配置如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!--会话工厂 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>

<!--spring事务管理 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource" />
</bean>

<!--使用注释事务 -->
<tx:annotation-driven  transaction-manager="transactionManager" />
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看到没，**sqlSessionFactory中配置了****dataSource，****transactionManager也配置了****dataSource，我们来回忆一下****SqlSessionFactoryBean这个类**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 protected SqlSessionFactory buildSqlSessionFactory() throws IOException {
 2 
 3     // 配置类
 4    Configuration configuration;
 5     // 解析mybatis-Config.xml文件，
 6     // 将相关配置信息保存到configuration
 7    XMLConfigBuilder xmlConfigBuilder = null;
 8    if (this.configuration != null) {
 9      configuration = this.configuration;
10      if (configuration.getVariables() == null) {
11        configuration.setVariables(this.configurationProperties);
12      } else if (this.configurationProperties != null) {
13        configuration.getVariables().putAll(this.configurationProperties);
14      }
15     //资源文件不为空
16    } else if (this.configLocation != null) {
17      //根据configLocation创建xmlConfigBuilder，XMLConfigBuilder构造器中会创建Configuration对象
18      xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);
19      //将XMLConfigBuilder构造器中创建的Configuration对象直接赋值给configuration属性
20      configuration = xmlConfigBuilder.getConfiguration();
21    } 
22    
23     //略....
24 
25    if (xmlConfigBuilder != null) {
26      try {
27        //解析mybatis-Config.xml文件，并将相关配置信息保存到configuration
28        xmlConfigBuilder.parse();
29        if (LOGGER.isDebugEnabled()) {
30          LOGGER.debug("Parsed configuration file: '" + this.configLocation + "'");
31        }
32      } catch (Exception ex) {
33        throw new NestedIOException("Failed to parse config resource: " + this.configLocation, ex);
34      }
35    }
36     
37    if (this.transactionFactory == null) {
38      //事务默认采用SpringManagedTransaction，这一块非常重要
39      this.transactionFactory = new SpringManagedTransactionFactory();
40    }
41     // 为sqlSessionFactory绑定事务管理器和数据源
42     // 这样sqlSessionFactory在创建sqlSession的时候可以通过该事务管理器获取jdbc连接，从而执行SQL
43    configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
44     // 解析mapper.xml
45    if (!isEmpty(this.mapperLocations)) {
46      for (Resource mapperLocation : this.mapperLocations) {
47        if (mapperLocation == null) {
48          continue;
49        }
50        try {
51          // 解析mapper.xml文件，并注册到configuration对象的mapperRegistry
52          XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
53              configuration, mapperLocation.toString(), configuration.getSqlFragments());
54          xmlMapperBuilder.parse();
55        } catch (Exception e) {
56          throw new NestedIOException("Failed to parse mapping resource: '" + mapperLocation + "'", e);
57        } finally {
58          ErrorContext.instance().reset();
59        }
60 
61        if (LOGGER.isDebugEnabled()) {
62          LOGGER.debug("Parsed mapper file: '" + mapperLocation + "'");
63        }
64      }
65    } else {
66      if (LOGGER.isDebugEnabled()) {
67        LOGGER.debug("Property 'mapperLocations' was not specified or no matching resources found");
68      }
69    }
70 
71     // 将Configuration对象实例作为参数，
72     // 调用sqlSessionFactoryBuilder创建sqlSessionFactory对象实例
73    return this.sqlSessionFactoryBuilder.build(configuration);
74 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看第39行，Mybatis集成Spring后，默认使用的transactionFactory是SpringManagedTransactionFactory，那我们就来看看其获取Transaction的方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private SqlSession openSessionFromConnection(ExecutorType execType, Connection connection) {
    try {
      boolean autoCommit;
      try {
        autoCommit = connection.getAutoCommit();
      } catch (SQLException e) {
        // Failover to true, as most poor drivers
        // or databases won't support transactions
        autoCommit = true;
      }      
      //从configuration中取出environment对象
      final Environment environment = configuration.getEnvironment();
      //从environment中取出TransactionFactory
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      //创建Transaction
      final Transaction tx = transactionFactory.newTransaction(connection);
      //创建包含事务操作的执行器
      final Executor executor = configuration.newExecutor(tx, execType);
      //构建包含执行器的SqlSession
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
}

private TransactionFactory getTransactionFactoryFromEnvironment(Environment environment) {
    if (environment == null || environment.getTransactionFactory() == null) {
      return new ManagedTransactionFactory();
    }
    //这里返回SpringManagedTransactionFactory
    return environment.getTransactionFactory();
}

@Override
public Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit) {
    //创建SpringManagedTransaction
    return new SpringManagedTransaction(dataSource);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11839993.html#_labelTop)

## SpringManagedTransaction

也就是说mybatis的执行事务的事务管理器就切换成了SpringManagedTransaction，下面我们再去看看SpringManagedTransactionFactory类的源码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class SpringManagedTransaction implements Transaction {
    private static final Log LOGGER = LogFactory.getLog(SpringManagedTransaction.class);
    private final DataSource dataSource;
    private Connection connection;
    private boolean isConnectionTransactional;
    private boolean autoCommit;

    public SpringManagedTransaction(DataSource dataSource) {
        Assert.notNull(dataSource, "No DataSource specified");
        this.dataSource = dataSource;
    }

    public Connection getConnection() throws SQLException {
        if (this.connection == null) {
            this.openConnection();
        }

        return this.connection;
    }

    private void openConnection() throws SQLException {
        //通过DataSourceUtils获取connection，这里和JdbcTransaction不一样
        this.connection = DataSourceUtils.getConnection(this.dataSource);
        this.autoCommit = this.connection.getAutoCommit();
        this.isConnectionTransactional = DataSourceUtils.isConnectionTransactional(this.connection, this.dataSource);
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("JDBC Connection [" + this.connection + "] will" + (this.isConnectionTransactional ? " " : " not ") + "be managed by Spring");
        }

    }

    public void commit() throws SQLException {
        if (this.connection != null && !this.isConnectionTransactional && !this.autoCommit) {
            if (LOGGER.isDebugEnabled()) {
                LOGGER.debug("Committing JDBC Connection [" + this.connection + "]");
            }
            //通过connection提交，这里和JdbcTransaction一样
            this.connection.commit();
        }

    }

    public void rollback() throws SQLException {
        if (this.connection != null && !this.isConnectionTransactional && !this.autoCommit) {
            if (LOGGER.isDebugEnabled()) {
                LOGGER.debug("Rolling back JDBC Connection [" + this.connection + "]");
            }
            //通过connection回滚，这里和JdbcTransaction一样
            this.connection.rollback();
        }

    }

    public void close() throws SQLException {
        DataSourceUtils.releaseConnection(this.connection, this.dataSource);
    }

    public Integer getTimeout() throws SQLException {
        ConnectionHolder holder = (ConnectionHolder)TransactionSynchronizationManager.getResource(this.dataSource);
        return holder != null && holder.hasTimeout() ? holder.getTimeToLiveInSeconds() : null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### org.springframework.jdbc.datasource.DataSourceUtils#getConnection

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static Connection getConnection(DataSource dataSource) throws CannotGetJdbcConnectionException {
    try {
        return doGetConnection(dataSource);
    }
    catch (SQLException ex) {
        throw new CannotGetJdbcConnectionException("Could not get JDBC Connection", ex);
    }
}

public static Connection doGetConnection(DataSource dataSource) throws SQLException {
    Assert.notNull(dataSource, "No DataSource specified");
    //TransactionSynchronizationManager重点！！！有没有很熟悉的感觉？？
    //还记得我们前面Spring事务源码的分析吗？@Transaction会创建Connection，并放入ThreadLocal中
    //这里从ThreadLocal中获取ConnectionHolder
    ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(dataSource);
    if (conHolder == null || !conHolder.hasConnection() && !conHolder.isSynchronizedWithTransaction()) {
        logger.debug("Fetching JDBC Connection from DataSource");
        //如果没有使用@Transaction，那调用Mapper接口方法时，也是通过Spring的方法获取Connection
        Connection con = fetchConnection(dataSource);
        if (TransactionSynchronizationManager.isSynchronizationActive()) {
            logger.debug("Registering transaction synchronization for JDBC Connection");
            ConnectionHolder holderToUse = conHolder;
            if (conHolder == null) {
                holderToUse = new ConnectionHolder(con);
            } else {
                conHolder.setConnection(con);
            }

            holderToUse.requested();
            TransactionSynchronizationManager.registerSynchronization(new DataSourceUtils.ConnectionSynchronization(holderToUse, dataSource));
            holderToUse.setSynchronizedWithTransaction(true);
            if (holderToUse != conHolder) {
                //将获取到的ConnectionHolder放入ThreadLocal中，那么当前线程调用下一个接口，下一个接口使用了Spring事务，那Spring事务也可以直接取到Mybatis创建的Connection
                //通过ThreadLocal保证了同一线程中Spring事务使用的Connection和Mapper代理类使用的Connection是同一个
                TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
            }
        }

        return con;
    } else {
        conHolder.requested();
        if (!conHolder.hasConnection()) {
            logger.debug("Fetching resumed JDBC Connection from DataSource");
            conHolder.setConnection(fetchConnection(dataSource));
        }

        //所以如果我们业务代码使用了@Transaction注解，在Spring中就已经通过dataSource创建了一个Connection并放入ThreadLocal中
        //那么当Mapper代理对象调用方法时，通过SqlSession的SpringManagedTransaction获取连接时，就直接获取到了当前线程中Spring事务创建的Connection并返回
        return conHolder.getConnection();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

想看怎么获取connHolder 



### org.springframework.transaction.support.TransactionSynchronizationManager#getResource

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//保存数据库连接的ThreadLocal
private static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal<>("Transactional resources");
@Nullable
public static Object getResource(Object key) {
    Object actualKey = TransactionSynchronizationUtils.unwrapResourceIfNecessary(key);
    //获取ConnectionHolder
    Object value = doGetResource(actualKey);
    ....
    return value;
}

@Nullable
private static Object doGetResource(Object actualKey) {
    /**
     * 从threadlocal <Map<Object, Object>>中取出来当前线程绑定的map
     * map里面存的是<dataSource,ConnectionHolder>
     */
    Map<Object, Object> map = resources.get();
    if (map == null) {
        return null;
    }
    //map中取出来对应dataSource的ConnectionHolder
    Object value = map.get(actualKey);
    // Transparently remove ResourceHolder that was marked as void...
    if (value instanceof ResourceHolder && ((ResourceHolder) value).isVoid()) {
        map.remove(actualKey);
        // Remove entire ThreadLocal if empty...
        if (map.isEmpty()) {
            resources.remove();
        }
        value = null;
    }
    return value;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到直接从ThreadLocal中取出来的conn,而spring自己的事务也是操作的这个ThreadLocal中的conn来进行事务的开启和回滚，由此我们知道了在同一线程中Spring事务中的Connection和Mybaits中Mapper代理对象中操作数据库的Connection是同一个，当取出来的conn为空时候,调用org.springframework.jdbc.datasource.DataSourceUtils#fetchConnection获取，然后把从数据源取出来的连接返回

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static Connection fetchConnection(DataSource dataSource) throws SQLException {
    //从数据源取出来conn
    Connection con = dataSource.getConnection();
    if (con == null) {
        throw new IllegalStateException("DataSource returned null from getConnection(): " + dataSource);
    }
    return con;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们再来回顾一下上篇文章中的SqlSessionInterceptor

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private class SqlSessionInterceptor implements InvocationHandler {
 2     private SqlSessionInterceptor() {
 3     }
 4 
 5     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
 6         SqlSession sqlSession = SqlSessionUtils.getSqlSession(SqlSessionTemplate.this.sqlSessionFactory, SqlSessionTemplate.this.executorType, SqlSessionTemplate.this.exceptionTranslator);
 7 
 8         Object unwrapped;
 9         try {
10             Object result = method.invoke(sqlSession, args);
11             // 如果当前操作没有在一个Spring事务中，则手动commit一下
12             // 如果当前业务没有使用@Transation,那么每次执行了Mapper接口的方法直接commit
13             // 还记得我们前面讲的Mybatis的一级缓存吗，这里一级缓存不能起作用了，因为每执行一个Mapper的方法，sqlSession都提交了
14             // sqlSession提交，会清空一级缓存
15             if (!SqlSessionUtils.isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
16                 sqlSession.commit(true);
17             }
18 
19             unwrapped = result;
20         } catch (Throwable var11) {
21             unwrapped = ExceptionUtil.unwrapThrowable(var11);
22             if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
23                 SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
24                 sqlSession = null;
25                 Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException)unwrapped);
26                 if (translated != null) {
27                     unwrapped = translated;
28                 }
29             }
30 
31             throw (Throwable)unwrapped;
32         } finally {
33             if (sqlSession != null) {
34                 SqlSessionUtils.closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
35             }
36 
37         }
38         return unwrapped;
39     }
40 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看第15和16行，如果我们没有使用@Transation，Mapper方法执行完后，sqlSession将会提交，也就是说通过org.springframework.jdbc.datasource.DataSourceUtils#fetchConnection获取到的Connection将会commit，相当于Connection是自动提交的，也就是说如果不使用@Transation，Mybatis将没有事务可言。

Mybatis和Spring整合后SpringManagedTransaction和Spring的Transaction的关系：

- 如果开启Spring事务，则先有Spring的Transaction，然后mybatis创建sqlSession时，会创建SpringManagedTransaction并加入sqlSession中，SpringManagedTransaction中的connection会从Spring的Transaction创建的Connection并放入ThreadLocal中获取
- 如果没有开启Spring事务或者第一个方法没有事务后面的方法有事务，则SpringManagedTransaction创建Connection并放入ThreadLocal中

spring结合mybatis后mybaits一级缓存失效分为两种情况：

- 如果没有开启事务，每一次sql都是用的新的SqlSession，这时mybatis的一级缓存是失效的。
- 如果有事务，同一个事务中相同的查询使用的相同的SqlSessioon，此时一级缓存是生效的。

如果使用了@Transation呢？那在调用Mapper代理类的方法之前就已经通过Spring的事务生成了Connection并放入ThreadLocal，并且设置事务不自动提交，当前线程多个Mapper代理对象调用数据库操作方法时，将从ThreadLocal获取Spring创建的connection,在所有的Mapper方法调用完后，Spring事务提交或者回滚，到此mybatis的事务是怎么被spring管理的就显而易见了

还有文章开头的问题，为什么Mybtis中要配置dataSource，Spring的事务中也要配置dataSource？

因为Spring事务在没调用Mapper方法之前就需要开一个Connection，并设置事务不自动提交，那么transactionManager中自然要配置dataSource。那如果我们的Service没有用到Spring事务呢，难道就不需要获取数据库连接了吗？当然不是，此时通过SpringManagedTransaction调用org.springframework.jdbc.datasource.DataSourceUtils#getConnection#fetchConnection方法获取，并将dataSource作为参数传进去，实际上获取的Connection都是通过dataSource来获取的。

**目录**



））））））））））））））））））））））））））））））））））））））））））

### API 接口层

##### 会话 SqlSession

### 核心处理层

##### 配置初始化

以 SqlSessionFactoryBuilder 去创建 SqlSessionFactory,  那么，我们就先从SqlSessionFactoryBuilder入手， 咱们先看看源码是怎么实现的

```
public class SqlSessionFactoryBuilder {

  //Reader读取mybatis配置文件，传入构造方法
  //除了Reader外，其实还有对应的inputStream作为参数的构造方法，
  //这也体现了mybatis配置的灵活性
  public SqlSessionFactory build(Reader reader) {
    return build(reader, null, null);
  }

  public SqlSessionFactory build(Reader reader, String environment) {
    return build(reader, environment, null);
  }

  //mybatis配置文件 + properties, 此时mybatis配置文件中可以不配置properties，也能使用${}形式
  public SqlSessionFactory build(Reader reader, Properties properties) {
    return build(reader, null, properties);
  }

  //通过XMLConfigBuilder解析mybatis配置，然后创建SqlSessionFactory对象
  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      //下面看看这个方法的源码
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

  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }

}
```

通过源码，我们可以看到SqlSessionFactoryBuilder 通过XMLConfigBuilder 去解析我们传入的mybatis的配置文件， 下面就接着看看 XMLConfigBuilder 部分源码：

```
/**
 * mybatis 配置文件解析
 */
public class XMLConfigBuilder extends BaseBuilder {
  public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
    this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
  }

  private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
  }

  //外部调用此方法对mybatis配置文件进行解析
  public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    //从根节点configuration
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }

  //此方法就是解析configuration节点下的子节点
  //由此也可看出，我们在configuration下面能配置的节点为以下10个节点
  private void parseConfiguration(XNode root) {
    try {
      propertiesElement(root.evalNode("properties")); //issue #117 read properties first
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      settingsElement(root.evalNode("settings"));
      environmentsElement(root.evalNode("environments")); // read it after objectFactory and objectWrapperFactory issue #631
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
}
```

通过以上源码，我们就能看出，在mybatis的配置文件中：

1. configuration节点为根节点。

2. 在configuration节点之下，我们可以配置10个子节点， 分别为：properties、typeAliases、plugins、objectFactory、objectWrapperFactory、settings、environments、databaseIdProvider、typeHandlers、mappers。

为了让大家能够更好地阅读mybatis源码，我先简单的给大家示例一下properties的使用方法。　

```
<configuration>
<!-- 方法一： 从外部指定properties配置文件, 除了使用resource属性指定外，还可通过url属性指定url
  <properties resource="dbConfig.properties"></properties>
  -->
  <!-- 方法二： 直接配置为xml -->
  <properties>
      <property name="driver" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
      <property name="username" value="root"/>
      <property name="password" value="root"/>
  </properties>
```

那么，我要是 两种方法都同时用了，那么哪种方法优先？

　　当以上两种方法都xml配置优先， 外部指定properties配置其次。至于为什么，接下来的源码分析会提到，请留意一下。

　　再看一下envirements元素节点的使用方法吧：

```
<environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
          <!--
          如果上面没有指定数据库配置的properties文件，那么此处可以这样直接配置 
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
         -->
         
         <!-- 上面指定了数据库配置文件， 配置文件里面也是对应的这四个属性 -->
         <property name="driver" value="${driver}"/>
         <property name="url" value="${url}"/>
         <property name="username" value="${username}"/>
         <property name="password" value="${password}"/>
         
      </dataSource>
    </environment>
    
    <!-- 我再指定一个environment -->
    <environment id="test">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <!-- 与上面的url不一样 -->
        <property name="url" value="jdbc:mysql://localhost:3306/demo"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
    
  </environments>
```

environments元素节点可以配置多个environment子节点， 怎么理解呢？ 

　　假如我们系统的开发环境和正式环境所用的数据库不一样（这是肯定的）， 那么可以设置两个environment, 两个id分别对应开发环境（dev）和正式环境（final），那么通过配置environments的default属性就能选择对应的environment了， 例如，我将environments的deault属性的值配置为dev, 那么就会选择dev的environment。 至于这个是怎么实现的， 下面源码就会讲。

　　好啦，上面简单给大家介绍了一下properties 和 environments 的配置， 接下来就正式开始看源码了：

　　上次我们说过mybatis 是通过XMLConfigBuilder这个类在解析mybatis配置文件的，那么本次就接着看看XMLConfigBuilder对于properties和environments的解析：

```
public class XMLConfigBuilder extends BaseBuilder {

    private boolean parsed;
    //xml解析器
    private XPathParser parser;
    private String environment;

    //上次说到这个方法是在解析mybatis配置文件中能配置的元素节点
    //今天首先要看的就是properties节点和environments节点
    private void parseConfiguration(XNode root) {
        try {
          //解析properties元素
          propertiesElement(root.evalNode("properties")); //issue #117 read properties first
          typeAliasesElement(root.evalNode("typeAliases"));
          pluginElement(root.evalNode("plugins"));
          objectFactoryElement(root.evalNode("objectFactory"));
          objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
          settingsElement(root.evalNode("settings"));
          //解析environments元素
          environmentsElement(root.evalNode("environments")); // read it after objectFactory and objectWrapperFactory issue #631
          databaseIdProviderElement(root.evalNode("databaseIdProvider"));
          typeHandlerElement(root.evalNode("typeHandlers"));
          mapperElement(root.evalNode("mappers"));
        } catch (Exception e) {
          throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
        }
    }


    //下面就看看解析properties的具体方法
    private void propertiesElement(XNode context) throws Exception {
        if (context != null) {
          //将子节点的 name 以及value属性set进properties对象
          //这儿可以注意一下顺序，xml配置优先， 外部指定properties配置其次
          Properties defaults = context.getChildrenAsProperties();
          //获取properties节点上 resource属性的值
          String resource = context.getStringAttribute("resource");
          //获取properties节点上 url属性的值, resource和url不能同时配置
          String url = context.getStringAttribute("url");
          if (resource != null && url != null) {
            throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
          }
          //把解析出的properties文件set进Properties对象
          if (resource != null) {
            defaults.putAll(Resources.getResourceAsProperties(resource));
          } else if (url != null) {
            defaults.putAll(Resources.getUrlAsProperties(url));
          }
          //将configuration对象中已配置的Properties属性与刚刚解析的融合
          //configuration这个对象会装载所解析mybatis配置文件的所有节点元素，以后也会频频提到这个对象
          //既然configuration对象用有一系列的get/set方法， 那是否就标志着我们可以使用java代码直接配置？
          //答案是肯定的， 不过使用配置文件进行配置，优势不言而喻
          Properties vars = configuration.getVariables();
          if (vars != null) {
            defaults.putAll(vars);
          }
          //把装有解析配置propertis对象set进解析器， 因为后面可能会用到
          parser.setVariables(defaults);
          //set进configuration对象
          configuration.setVariables(defaults);
        }
    }

    //下面再看看解析enviroments元素节点的方法
    private void environmentsElement(XNode context) throws Exception {
        if (context != null) {
            if (environment == null) {
                //解析environments节点的default属性的值
                //例如: <environments default="development">
                environment = context.getStringAttribute("default");
            }
            //递归解析environments子节点
            for (XNode child : context.getChildren()) {
                //<environment id="development">, 只有enviroment节点有id属性，那么这个属性有何作用？
                //environments 节点下可以拥有多个 environment子节点
                //类似于这样： <environments default="development"><environment id="development">...</environment><environment id="test">...</environments>
                //意思就是我们可以对应多个环境，比如开发环境，测试环境等， 由environments的default属性去选择对应的enviroment
                String id = child.getStringAttribute("id");
                //isSpecial就是根据由environments的default属性去选择对应的enviroment
                if (isSpecifiedEnvironment(id)) {
                    //事务， mybatis有两种：JDBC 和 MANAGED, 配置为JDBC则直接使用JDBC的事务，配置为MANAGED则是将事务托管给容器，
                    TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                    //enviroment节点下面就是dataSource节点了，解析dataSource节点（下面会贴出解析dataSource的具体方法）
                    DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                    DataSource dataSource = dsFactory.getDataSource();
                    Environment.Builder environmentBuilder = new Environment.Builder(id)
                          .transactionFactory(txFactory)
                          .dataSource(dataSource);
                    //老规矩，会将dataSource设置进configuration对象
                    configuration.setEnvironment(environmentBuilder.build());
                }
            }
        }
    }

    //下面看看dataSource的解析方法
    private DataSourceFactory dataSourceElement(XNode context) throws Exception {
        if (context != null) {
            //dataSource的连接池
            String type = context.getStringAttribute("type");
            //子节点 name, value属性set进一个properties对象
            Properties props = context.getChildrenAsProperties();
            //创建dataSourceFactory
            DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();
            factory.setProperties(props);
            return factory;
        }
        throw new BuilderException("Environment declaration requires a DataSourceFactory.");
    }
}
```

通过以上对mybatis源码的解读，相信大家对mybatis的配置又有了一个深入的认识。

　　还有一个问题， 上面我们看到，在配置dataSource的时候使用了 ${driver} 这种表达式， 这种形式是怎么解析的？其实，是通过PropertyParser这个类解析：

```
/**
 * 这个类解析${}这种形式的表达式
 */
public class PropertyParser {

  public static String parse(String string, Properties variables) {
    VariableTokenHandler handler = new VariableTokenHandler(variables);
    GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
    return parser.parse(string);
  }

  private static class VariableTokenHandler implements TokenHandler {
    private Properties variables;

    public VariableTokenHandler(Properties variables) {
      this.variables = variables;
    }

    public String handleToken(String content) {
      if (variables != null && variables.containsKey(content)) {
        return variables.getProperty(content);
      }
      return "${" + content + "}";
    }
  }
}
```

好啦，以上就是对于properties 和 environments元素节点的分析，比较重要的都在对于源码的注释中标出。

例如： 我们在使用 com.demo.entity. UserEntity 的时候，我们可以直接配置一个别名user, 这样以后在配置文件中要使用到com.demo.entity. UserEntity的时候，直接使用User即可。

　　就以上例为例，我们来实现一下，看看typeAliases的配置方法：　

```
<configuration>
    <typeAliases>
      <!--
      通过package, 可以直接指定package的名字， mybatis会自动扫描你指定包下面的javabean,
      并且默认设置一个别名，默认的名字为： javabean 的首字母小写的非限定类名来作为它的别名。
      也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user) 
      <package name="com.dy.entity"/>
       -->
      <typeAlias alias="UserEntity" type="com.dy.entity.User"/>
  </typeAliases>
  
  ......
  
</configuration>
```

先从对xml的解析讲起：

```
/**
 * 解析typeAliases节点
 */
private void typeAliasesElement(XNode parent) {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        //如果子节点是package, 那么就获取package节点的name属性， mybatis会扫描指定的package
        if ("package".equals(child.getName())) {
          String typeAliasPackage = child.getStringAttribute("name");
          //TypeAliasRegistry 负责管理别名， 这儿就是通过TypeAliasRegistry 进行别名注册， 下面就会看看TypeAliasRegistry源码
          configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
        } else {
          //如果子节点是typeAlias节点，那么就获取alias属性和type的属性值
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

重要的源码在这儿

```
public class TypeAliasRegistry {
  
  //这就是核心所在啊， 原来别名就仅仅通过一个HashMap来实现， key为别名， value就是别名对应的类型（class对象）
  private final Map<String, Class<?>> TYPE_ALIASES = new HashMap<String, Class<?>>();

  /**
   * 以下就是mybatis默认为我们注册的别名
   */
  public TypeAliasRegistry() {
    registerAlias("string", String.class);

    registerAlias("byte", Byte.class);
    registerAlias("long", Long.class);
    registerAlias("short", Short.class);
    registerAlias("int", Integer.class);
    registerAlias("integer", Integer.class);
    registerAlias("double", Double.class);
    registerAlias("float", Float.class);
    registerAlias("boolean", Boolean.class);

    registerAlias("byte[]", Byte[].class);
    registerAlias("long[]", Long[].class);
    registerAlias("short[]", Short[].class);
    registerAlias("int[]", Integer[].class);
    registerAlias("integer[]", Integer[].class);
    registerAlias("double[]", Double[].class);
    registerAlias("float[]", Float[].class);
    registerAlias("boolean[]", Boolean[].class);

    registerAlias("_byte", byte.class);
    registerAlias("_long", long.class);
    registerAlias("_short", short.class);
    registerAlias("_int", int.class);
    registerAlias("_integer", int.class);
    registerAlias("_double", double.class);
    registerAlias("_float", float.class);
    registerAlias("_boolean", boolean.class);

    registerAlias("_byte[]", byte[].class);
    registerAlias("_long[]", long[].class);
    registerAlias("_short[]", short[].class);
    registerAlias("_int[]", int[].class);
    registerAlias("_integer[]", int[].class);
    registerAlias("_double[]", double[].class);
    registerAlias("_float[]", float[].class);
    registerAlias("_boolean[]", boolean[].class);

    registerAlias("date", Date.class);
    registerAlias("decimal", BigDecimal.class);
    registerAlias("bigdecimal", BigDecimal.class);
    registerAlias("biginteger", BigInteger.class);
    registerAlias("object", Object.class);

    registerAlias("date[]", Date[].class);
    registerAlias("decimal[]", BigDecimal[].class);
    registerAlias("bigdecimal[]", BigDecimal[].class);
    registerAlias("biginteger[]", BigInteger[].class);
    registerAlias("object[]", Object[].class);

    registerAlias("map", Map.class);
    registerAlias("hashmap", HashMap.class);
    registerAlias("list", List.class);
    registerAlias("arraylist", ArrayList.class);
    registerAlias("collection", Collection.class);
    registerAlias("iterator", Iterator.class);

    registerAlias("ResultSet", ResultSet.class);
  }

  
  /**
   * 处理别名， 直接从保存有别名的hashMap中取出即可
   */
  @SuppressWarnings("unchecked")
  public <T> Class<T> resolveAlias(String string) {
    try {
      if (string == null) return null;
      String key = string.toLowerCase(Locale.ENGLISH); // issue #748
      Class<T> value;
      if (TYPE_ALIASES.containsKey(key)) {
        value = (Class<T>) TYPE_ALIASES.get(key);
      } else {
        value = (Class<T>) Resources.classForName(string);
      }
      return value;
    } catch (ClassNotFoundException e) {
      throw new TypeException("Could not resolve type alias '" + string + "'.  Cause: " + e, e);
    }
  }
  
  /**
   * 配置文件中配置为package的时候， 会调用此方法，根据配置的报名去扫描javabean ，然后自动注册别名
   * 默认会使用 Bean 的首字母小写的非限定类名来作为它的别名
   * 也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user)
   */
  public void registerAliases(String packageName){
    registerAliases(packageName, Object.class);
  }

  public void registerAliases(String packageName, Class<?> superType){
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for(Class<?> type : typeSet){
      // Ignore inner classes and interfaces (including package-info.java)
      // Skip also inner classes. See issue #6
      if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
        registerAlias(type);
      }
    }
  }

  public void registerAlias(Class<?> type) {
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
      alias = aliasAnnotation.value();
    } 
    registerAlias(alias, type);
  }

  //这就是注册别名的本质方法， 其实就是向保存别名的hashMap新增值而已， 呵呵， 别名的实现太简单了，对吧
  public void registerAlias(String alias, Class<?> value) {
    if (alias == null) throw new TypeException("The parameter alias cannot be null");
    String key = alias.toLowerCase(Locale.ENGLISH); // issue #748
    if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null && !TYPE_ALIASES.get(key).equals(value)) {
      throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + TYPE_ALIASES.get(key).getName() + "'.");
    }
    TYPE_ALIASES.put(key, value);
  }

  public void registerAlias(String alias, String value) {
    try {
      registerAlias(alias, Resources.classForName(value));
    } catch (ClassNotFoundException e) {
      throw new TypeException("Error registering type alias "+alias+" for "+value+". Cause: " + e, e);
    }
  }
  
  /**
   * 获取保存别名的HashMap, Configuration对象持有对TypeAliasRegistry的引用，因此，如果需要，我们可以通过Configuration对象获取
   */
  public Map<String, Class<?>> getTypeAliases() {
    return Collections.unmodifiableMap(TYPE_ALIASES);
  }

}
```

由源码可见，设置别名的原理就这么简单，Mybatis默认给我们设置了不少别名，在上面代码中都可以见到。

Mybatis中的TypeHandler是什么？

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成 Java 类型。Mybatis默认为我们实现了许多TypeHandler, 当我们没有配置指定TypeHandler时，Mybatis会根据参数或者返回结果的不同，默认为我们选择合适的TypeHandler处理。

那么，Mybatis为我们实现了哪些TypeHandler呢?  我们怎么自定义实现一个TypeHandler ?  这些都会在接下来的mybatis的源码中看到。

在看源码之前，还是像之前一样，先看看怎么配置吧？

```
<configuration>
    <typeHandlers>
      <!-- 
          当配置package的时候，mybatis会去配置的package扫描TypeHandler
          <package name="com.dy.demo"/>
       -->
      
      <!-- handler属性直接配置我们要指定的TypeHandler -->
      <typeHandler handler=""/>
      
      <!-- javaType 配置java类型，例如String, 如果配上javaType, 那么指定的typeHandler就只作用于指定的类型 -->
      <typeHandler javaType="" handler=""/>
      
      <!-- jdbcType 配置数据库基本数据类型，例如varchar, 如果配上jdbcType, 那么指定的typeHandler就只作用于指定的类型  -->
      <typeHandler jdbcType="" handler=""/>
      
      <!-- 也可两者都配置 -->
      <typeHandler javaType="" jdbcType="" handler=""/>
      
  </typeHandlers>
  
  ......
  
</configuration>
```

先从对xml的解析讲起

```
/**
 * 解析typeHandlers节点
 */
private void typeHandlerElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        //子节点为package时，获取其name属性的值，然后自动扫描package下的自定义typeHandler
        if ("package".equals(child.getName())) {
          String typeHandlerPackage = child.getStringAttribute("name");
          typeHandlerRegistry.register(typeHandlerPackage);
        } else {
          //子节点为typeHandler时， 可以指定javaType属性， 也可以指定jdbcType, 也可两者都指定
          //javaType 是指定java类型
          //jdbcType 是指定jdbc类型（数据库类型： 如varchar）
          String javaTypeName = child.getStringAttribute("javaType");
          String jdbcTypeName = child.getStringAttribute("jdbcType");
          //handler就是我们配置的typeHandler
          String handlerTypeName = child.getStringAttribute("handler");
          //resolveClass方法就是我们上篇文章所讲的TypeAliasRegistry里面处理别名的方法
          Class<?> javaTypeClass = resolveClass(javaTypeName);
          //JdbcType是一个枚举类型，resolveJdbcType方法是在获取枚举类型的值
          JdbcType jdbcType = resolveJdbcType(jdbcTypeName);
          Class<?> typeHandlerClass = resolveClass(handlerTypeName);
          //注册typeHandler, typeHandler通过TypeHandlerRegistry这个类管理
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

接下来看看TypeHandler的管理注册类：

```
/**
 * typeHandler注册管理类
 */
public final class TypeHandlerRegistry {

  //源码一上来，二话不说，几个大大的HashMap就出现，这不又跟上次讲的typeAliases的注册类似么

  //基本数据类型与其包装类
  private static final Map<Class<?>, Class<?>> reversePrimitiveMap = new HashMap<Class<?>, Class<?>>() {
    private static final long serialVersionUID = 1L;
    {
      put(Byte.class, byte.class);
      put(Short.class, short.class);
      put(Integer.class, int.class);
      put(Long.class, long.class);
      put(Float.class, float.class);
      put(Double.class, double.class);
      put(Boolean.class, boolean.class);
      put(Character.class, char.class);
    }
  };

  //这几个MAP不用说就知道存的是什么东西吧，命名的好处
  private final Map<JdbcType, TypeHandler<?>> JDBC_TYPE_HANDLER_MAP = new EnumMap<JdbcType, TypeHandler<?>>(JdbcType.class);
  private final Map<Type, Map<JdbcType, TypeHandler<?>>> TYPE_HANDLER_MAP = new HashMap<Type, Map<JdbcType, TypeHandler<?>>>();
  private final TypeHandler<Object> UNKNOWN_TYPE_HANDLER = new UnknownTypeHandler(this);
  private final Map<Class<?>, TypeHandler<?>> ALL_TYPE_HANDLERS_MAP = new HashMap<Class<?>, TypeHandler<?>>();

  //就像上篇文章讲的typeAliases一样，mybatis也默认给我们注册了不少的typeHandler
  //具体如下
  public TypeHandlerRegistry() {
    register(Boolean.class, new BooleanTypeHandler());
    register(boolean.class, new BooleanTypeHandler());
    register(JdbcType.BOOLEAN, new BooleanTypeHandler());
    register(JdbcType.BIT, new BooleanTypeHandler());

    register(Byte.class, new ByteTypeHandler());
    register(byte.class, new ByteTypeHandler());
    register(JdbcType.TINYINT, new ByteTypeHandler());

    register(Short.class, new ShortTypeHandler());
    register(short.class, new ShortTypeHandler());
    register(JdbcType.SMALLINT, new ShortTypeHandler());

    register(Integer.class, new IntegerTypeHandler());
    register(int.class, new IntegerTypeHandler());
    register(JdbcType.INTEGER, new IntegerTypeHandler());

    register(Long.class, new LongTypeHandler());
    register(long.class, new LongTypeHandler());

    register(Float.class, new FloatTypeHandler());
    register(float.class, new FloatTypeHandler());
    register(JdbcType.FLOAT, new FloatTypeHandler());

    register(Double.class, new DoubleTypeHandler());
    register(double.class, new DoubleTypeHandler());
    register(JdbcType.DOUBLE, new DoubleTypeHandler());

    register(String.class, new StringTypeHandler());
    register(String.class, JdbcType.CHAR, new StringTypeHandler());
    register(String.class, JdbcType.CLOB, new ClobTypeHandler());
    register(String.class, JdbcType.VARCHAR, new StringTypeHandler());
    register(String.class, JdbcType.LONGVARCHAR, new ClobTypeHandler());
    register(String.class, JdbcType.NVARCHAR, new NStringTypeHandler());
    register(String.class, JdbcType.NCHAR, new NStringTypeHandler());
    register(String.class, JdbcType.NCLOB, new NClobTypeHandler());
    register(JdbcType.CHAR, new StringTypeHandler());
    register(JdbcType.VARCHAR, new StringTypeHandler());
    register(JdbcType.CLOB, new ClobTypeHandler());
    register(JdbcType.LONGVARCHAR, new ClobTypeHandler());
    register(JdbcType.NVARCHAR, new NStringTypeHandler());
    register(JdbcType.NCHAR, new NStringTypeHandler());
    register(JdbcType.NCLOB, new NClobTypeHandler());

    register(Object.class, JdbcType.ARRAY, new ArrayTypeHandler());
    register(JdbcType.ARRAY, new ArrayTypeHandler());

    register(BigInteger.class, new BigIntegerTypeHandler());
    register(JdbcType.BIGINT, new LongTypeHandler());

    register(BigDecimal.class, new BigDecimalTypeHandler());
    register(JdbcType.REAL, new BigDecimalTypeHandler());
    register(JdbcType.DECIMAL, new BigDecimalTypeHandler());
    register(JdbcType.NUMERIC, new BigDecimalTypeHandler());

    register(Byte[].class, new ByteObjectArrayTypeHandler());
    register(Byte[].class, JdbcType.BLOB, new BlobByteObjectArrayTypeHandler());
    register(Byte[].class, JdbcType.LONGVARBINARY, new BlobByteObjectArrayTypeHandler());
    register(byte[].class, new ByteArrayTypeHandler());
    register(byte[].class, JdbcType.BLOB, new BlobTypeHandler());
    register(byte[].class, JdbcType.LONGVARBINARY, new BlobTypeHandler());
    register(JdbcType.LONGVARBINARY, new BlobTypeHandler());
    register(JdbcType.BLOB, new BlobTypeHandler());

    register(Object.class, UNKNOWN_TYPE_HANDLER);
    register(Object.class, JdbcType.OTHER, UNKNOWN_TYPE_HANDLER);
    register(JdbcType.OTHER, UNKNOWN_TYPE_HANDLER);

    register(Date.class, new DateTypeHandler());
    register(Date.class, JdbcType.DATE, new DateOnlyTypeHandler());
    register(Date.class, JdbcType.TIME, new TimeOnlyTypeHandler());
    register(JdbcType.TIMESTAMP, new DateTypeHandler());
    register(JdbcType.DATE, new DateOnlyTypeHandler());
    register(JdbcType.TIME, new TimeOnlyTypeHandler());

    register(java.sql.Date.class, new SqlDateTypeHandler());
    register(java.sql.Time.class, new SqlTimeTypeHandler());
    register(java.sql.Timestamp.class, new SqlTimestampTypeHandler());

    // issue #273
    register(Character.class, new CharacterTypeHandler());
    register(char.class, new CharacterTypeHandler());
  }

  public boolean hasTypeHandler(Class<?> javaType) {
    return hasTypeHandler(javaType, null);
  }

  public boolean hasTypeHandler(TypeReference<?> javaTypeReference) {
    return hasTypeHandler(javaTypeReference, null);
  }

  public boolean hasTypeHandler(Class<?> javaType, JdbcType jdbcType) {
    return javaType != null && getTypeHandler((Type) javaType, jdbcType) != null;
  }

  public boolean hasTypeHandler(TypeReference<?> javaTypeReference, JdbcType jdbcType) {
    return javaTypeReference != null && getTypeHandler(javaTypeReference, jdbcType) != null;
  }

  public TypeHandler<?> getMappingTypeHandler(Class<? extends TypeHandler<?>> handlerType) {
    return ALL_TYPE_HANDLERS_MAP.get(handlerType);
  }

  public <T> TypeHandler<T> getTypeHandler(Class<T> type) {
    return getTypeHandler((Type) type, null);
  }

  public <T> TypeHandler<T> getTypeHandler(TypeReference<T> javaTypeReference) {
    return getTypeHandler(javaTypeReference, null);
  }

  public TypeHandler<?> getTypeHandler(JdbcType jdbcType) {
    return JDBC_TYPE_HANDLER_MAP.get(jdbcType);
  }

  public <T> TypeHandler<T> getTypeHandler(Class<T> type, JdbcType jdbcType) {
    return getTypeHandler((Type) type, jdbcType);
  }

  public <T> TypeHandler<T> getTypeHandler(TypeReference<T> javaTypeReference, JdbcType jdbcType) {
    return getTypeHandler(javaTypeReference.getRawType(), jdbcType);
  }

  private <T> TypeHandler<T> getTypeHandler(Type type, JdbcType jdbcType) {
    Map<JdbcType, TypeHandler<?>> jdbcHandlerMap = TYPE_HANDLER_MAP.get(type);
    TypeHandler<?> handler = null;
    if (jdbcHandlerMap != null) {
      handler = jdbcHandlerMap.get(jdbcType);
      if (handler == null) {
        handler = jdbcHandlerMap.get(null);
      }
    }
    if (handler == null && type != null && type instanceof Class && Enum.class.isAssignableFrom((Class<?>) type)) {
      handler = new EnumTypeHandler((Class<?>) type);
    }
    @SuppressWarnings("unchecked")
    // type drives generics here
    TypeHandler<T> returned = (TypeHandler<T>) handler;
    return returned;
  }

  public TypeHandler<Object> getUnknownTypeHandler() {
    return UNKNOWN_TYPE_HANDLER;
  }

  public void register(JdbcType jdbcType, TypeHandler<?> handler) {
    JDBC_TYPE_HANDLER_MAP.put(jdbcType, handler);
  }

  //
  // REGISTER INSTANCE
  //

  /**
   * 只配置了typeHandler, 没有配置jdbcType 或者javaType
   */
  @SuppressWarnings("unchecked")
  public <T> void register(TypeHandler<T> typeHandler) {
    boolean mappedTypeFound = false;
    //在自定义typeHandler的时候，可以加上注解MappedTypes 去指定关联的javaType
    //因此，此处需要扫描MappedTypes注解
    MappedTypes mappedTypes = typeHandler.getClass().getAnnotation(MappedTypes.class);
    if (mappedTypes != null) {
      for (Class<?> handledType : mappedTypes.value()) {
        register(handledType, typeHandler);
        mappedTypeFound = true;
      }
    }
    // @since 3.1.0 - try to auto-discover the mapped type
    if (!mappedTypeFound && typeHandler instanceof TypeReference) {
      try {
        TypeReference<T> typeReference = (TypeReference<T>) typeHandler;
        register(typeReference.getRawType(), typeHandler);
        mappedTypeFound = true;
      } catch (Throwable t) {
        // maybe users define the TypeReference with a different type and are not assignable, so just ignore it
      }
    }
    if (!mappedTypeFound) {
      register((Class<T>) null, typeHandler);
    }
  }

  /**
   * 配置了typeHandlerhe和javaType
   */
  public <T> void register(Class<T> javaType, TypeHandler<? extends T> typeHandler) {
    register((Type) javaType, typeHandler);
  }

  private <T> void register(Type javaType, TypeHandler<? extends T> typeHandler) {
    //扫描注解MappedJdbcTypes
    MappedJdbcTypes mappedJdbcTypes = typeHandler.getClass().getAnnotation(MappedJdbcTypes.class);
    if (mappedJdbcTypes != null) {
      for (JdbcType handledJdbcType : mappedJdbcTypes.value()) {
        register(javaType, handledJdbcType, typeHandler);
      }
      if (mappedJdbcTypes.includeNullJdbcType()) {
        register(javaType, null, typeHandler);
      }
    } else {
      register(javaType, null, typeHandler);
    }
  }

  public <T> void register(TypeReference<T> javaTypeReference, TypeHandler<? extends T> handler) {
    register(javaTypeReference.getRawType(), handler);
  }

  /**
   * typeHandlerhe、javaType、jdbcType都配置了
   */
  public <T> void register(Class<T> type, JdbcType jdbcType, TypeHandler<? extends T> handler) {
    register((Type) type, jdbcType, handler);
  }

  /**
   * 注册typeHandler的核心方法
   * 就是向Map新增数据而已
   */
  private void register(Type javaType, JdbcType jdbcType, TypeHandler<?> handler) {
    if (javaType != null) {
      Map<JdbcType, TypeHandler<?>> map = TYPE_HANDLER_MAP.get(javaType);
      if (map == null) {
        map = new HashMap<JdbcType, TypeHandler<?>>();
        TYPE_HANDLER_MAP.put(javaType, map);
      }
      map.put(jdbcType, handler);
      if (reversePrimitiveMap.containsKey(javaType)) {
        register(reversePrimitiveMap.get(javaType), jdbcType, handler);
      }
    }
    ALL_TYPE_HANDLERS_MAP.put(handler.getClass(), handler);
  }

  //
  // REGISTER CLASS
  //

  // Only handler type

  public void register(Class<?> typeHandlerClass) {
    boolean mappedTypeFound = false;
    MappedTypes mappedTypes = typeHandlerClass.getAnnotation(MappedTypes.class);
    if (mappedTypes != null) {
      for (Class<?> javaTypeClass : mappedTypes.value()) {
        register(javaTypeClass, typeHandlerClass);
        mappedTypeFound = true;
      }
    }
    if (!mappedTypeFound) {
      register(getInstance(null, typeHandlerClass));
    }
  }

  // java type + handler type

  public void register(Class<?> javaTypeClass, Class<?> typeHandlerClass) {
    register(javaTypeClass, getInstance(javaTypeClass, typeHandlerClass));
  }

  // java type + jdbc type + handler type

  public void register(Class<?> javaTypeClass, JdbcType jdbcType, Class<?> typeHandlerClass) {
    register(javaTypeClass, jdbcType, getInstance(javaTypeClass, typeHandlerClass));
  }

  // Construct a handler (used also from Builders)

  @SuppressWarnings("unchecked")
  public <T> TypeHandler<T> getInstance(Class<?> javaTypeClass, Class<?> typeHandlerClass) {
    if (javaTypeClass != null) {
      try {
        Constructor<?> c = typeHandlerClass.getConstructor(Class.class);
        return (TypeHandler<T>) c.newInstance(javaTypeClass);
      } catch (NoSuchMethodException ignored) {
        // ignored
      } catch (Exception e) {
        throw new TypeException("Failed invoking constructor for handler " + typeHandlerClass, e);
      }
    }
    try {
      Constructor<?> c = typeHandlerClass.getConstructor();
      return (TypeHandler<T>) c.newInstance();
    } catch (Exception e) {
      throw new TypeException("Unable to find a usable constructor for " + typeHandlerClass, e);
    }
  }

 
  /**
   * 根据指定的pacakge去扫描自定义的typeHander，然后注册
   */
  public void register(String packageName) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(TypeHandler.class), packageName);
    Set<Class<? extends Class<?>>> handlerSet = resolverUtil.getClasses();
    for (Class<?> type : handlerSet) {
      //Ignore inner classes and interfaces (including package-info.java) and abstract classes
      if (!type.isAnonymousClass() && !type.isInterface() && !Modifier.isAbstract(type.getModifiers())) {
        register(type);
      }
    }
  }
  
  // get information
  
  /**
   * 通过configuration对象可以获取已注册的所有typeHandler
   */
  public Collection<TypeHandler<?>> getTypeHandlers() {
    return Collections.unmodifiableCollection(ALL_TYPE_HANDLERS_MAP.values());
  }
  
}
```

由源码可以看到， mybatis为我们实现了那么多TypeHandler,  随便打开一个TypeHandler，看其源码，都可以看到，它继承自一个抽象类：BaseTypeHandler， 那么我们是不是也能通过继承BaseTypeHandler，从而实现自定义的TypeHandler ? 答案是肯定的， 那么现在下面就为大家演示一下自定义TypeHandler:

```
@MappedJdbcTypes(JdbcType.VARCHAR)  
//此处如果不用注解指定jdbcType, 那么，就可以在配置文件中通过"jdbcType"属性指定， 同理， javaType 也可通过 @MappedTypes指定
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

然后，就该配置我们的自定义TypeHandler了：

```
<configuration>
  <typeHandlers>
      <!-- 由于自定义的TypeHandler在定义时已经通过注解指定了jdbcType, 所以此处不用再配置jdbcType -->
      <typeHandler handler="ExampleTypeHandler"/>
  </typeHandlers>
  
  ......
  
</configuration>
```

也就是说，我们在自定义TypeHandler的时候，可以在TypeHandler通过@MappedJdbcTypes指定jdbcType, 通过 @MappedTypes 指定javaType, 如果没有使用注解指定，那么我们就需要在配置文件中配置。

那么，接下来，就简单介绍一下这几个objectFactory、databaseIdProvider、plugins、mappers配置的作用吧：

1、objectFactory是干什么的？ 需要配置吗？

　　MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。默认情况下，我们不需要配置，mybatis会调用默认实现的objectFactory。 除非我们要自定义ObjectFactory的实现， 那么我们才需要去手动配置。

　　那么怎么自定义实现ObjectFactory？ 怎么配置呢？

　　自定义ObjectFactory只需要去继承DefaultObjectFactory（是ObjectFactory接口的实现类），并重写其方法即可。具体的，本处不多说，后面再具体讲解。

　　写好了ObjectFactory, 仅需做如下配置： 

```
<configuration>
    ......
    <objectFactory type="org.mybatis.example.ExampleObjectFactory">
        <property name="someProperty" value="100"/>
    </objectFactory>
    ......
  </configuration
```

2、plugin有何作用？ 需要配置吗？

　　plugins 是一个可选配置。mybatis中的plugin其实就是个interceptor， 它可以拦截Executor 、ParameterHandler 、ResultSetHandler 、StatementHandler 的部分方法，处理我们自己的逻辑。Executor就是真正执行sql语句的东西， ParameterHandler 是处理我们传入参数的，还记得前面讲TypeHandler的时候提到过，mybatis默认帮我们实现了不少的typeHandler, 当我们不显示配置typeHandler的时候，mybatis会根据参数类型自动选择合适的typeHandler执行，其实就是ParameterHandler 在选择。ResultSetHandler 就是处理返回结果的。

　  怎么自定义plugin ? 怎么配置？

　  要自定义一个plugin, 需要去实现Interceptor接口， 这儿不细说， 后面实战部分会详细讲解。定义好之后，配置如下：

```
<configuration>
    ......
    <plugins>
      <plugin interceptor="org.mybatis.example.ExamplePlugin">
        <property name="someProperty" value="100"/>
      </plugin>
    </plugins>
    ......
  </configuration>
```

3、mappers, 这下引出mybatis的核心之一了，mappers作用 ? 需要配置吗？

　　mappers 节点下，配置我们的mapper映射文件， 所谓的mapper映射文件，就是让mybatis 用来建立数据表和javabean映射的一个桥梁。在我们实际开发中，通常一个mapper文件对应一个dao接口， 这个mapper可以看做是dao的实现。所以,mappers必须配置。

　　那么怎么配置呢？

```
<configuration>
    ......
    <mappers>
      <!-- 第一种方式：通过resource指定 -->
    <mapper resource="com/dy/dao/userDao.xml"/>
    
     <!-- 第二种方式， 通过class指定接口，进而将接口与对应的xml文件形成映射关系
             不过，使用这种方式必须保证 接口与mapper文件同名(不区分大小写)， 
             我这儿接口是UserDao,那么意味着mapper文件为UserDao.xml 
     <mapper class="com.dy.dao.UserDao"/>
      -->
      
      <!-- 第三种方式，直接指定包，自动扫描，与方法二同理 
      <package name="com.dy.dao"/>
      -->
      <!-- 第四种方式：通过url指定mapper文件位置
      <mapper url="file://........"/>
       -->
  </mappers>
    ......
  </configuration>
```

这几个节点的解析源码，与之前提到的那些节点的解析类似

```
/**
 * objectFactory 节点解析
 */
private void objectFactoryElement(XNode context) throws Exception {
    if (context != null) {
      //读取type属性的值， 接下来进行实例化ObjectFactory, 并set进 configuration
      //到此，简单讲一下configuration这个对象，其实它里面主要保存的都是mybatis的配置
      String type = context.getStringAttribute("type");
      //读取propertie的值， 根据需要可以配置， mybatis默认实现的objectFactory没有使用properties
      Properties properties = context.getChildrenAsProperties();
      
      ObjectFactory factory = (ObjectFactory) resolveClass(type).newInstance();
      factory.setProperties(properties);
      configuration.setObjectFactory(factory);
    }
 }
 
  
  /**
   * plugins 节点解析
   */
  private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        String interceptor = child.getStringAttribute("interceptor");
        Properties properties = child.getChildrenAsProperties();
        //由此可见，我们在定义一个interceptor的时候，需要去实现Interceptor, 这儿先不具体讲，以后会详细讲解
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
        interceptorInstance.setProperties(properties);
        configuration.addInterceptor(interceptorInstance);
      }
    }
  }
  
  /**
   * mappers 节点解析
   * 这是mybatis的核心之一，这儿先简单介绍，在接下来的文章会对它进行分析
   */
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          //如果mappers节点的子节点是package, 那么就扫描package下的文件, 注入进configuration
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          //resource, url, class 三选一
          
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            //mapper映射文件都是通过XMLMapperBuilder解析
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

mapper映射文件的配置， 这是mybatis的核心之一，一定要学好。在mapper文件中，以mapper作为根节点，其下面可以配置的元素节点有： select, insert, update, delete, cache, cache-ref, resultMap, sql 。

本篇文章将简单介绍 insert, update, delete 的配置及使用，以后会对mybatis的源码进行深入讲解。

相信，看到insert, update, delete， 我们就知道其作用了，顾名思义嘛，myabtis 作为持久层框架，必须要对CRUD啊。

好啦，咱们就先来看看 insert, update, delete 怎么配置， 能配置哪些元素吧：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<!-- mapper 为根元素节点， 一个namespace对应一个dao -->
<mapper namespace="com.dy.dao.UserDao">

    <insert
      <!-- 1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致 -->
      
      id="insertUser"
      
      <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
      
      parameterType="com.demo.User"
      
      <!-- 3. flushCache （可选配置，默认配置为true）
        将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句） -->
      
      flushCache="true"
      
      <!-- 4. statementType （可选配置，默认配置为PREPARED）
        STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 -->
      
      statementType="PREPARED"
      
      <!-- 5. keyProperty (可选配置， 默认为unset)
        （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      
      keyProperty=""
      
      <!-- 6. keyColumn     (可选配置)
        （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      
      keyColumn=""
      
      <!-- 7. useGeneratedKeys (可选配置， 默认为false)
        （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。  -->
      
      useGeneratedKeys="false"
      
      <!-- 8. timeout  (可选配置， 默认为unset, 依赖驱动)
        这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 -->
      timeout="20">

    <update
      id="updateUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">

    <delete
      id="deleteUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">
</mapper>
```

以上就是一个模板配置， 哪些是必要配置，哪些是根据自己实际需求，看一眼就知道了。

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<mapper namespace="com.dy.dao.UserDao">
   
   <!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
           insert into user(id, name, password, age, deleteFlag) 
               values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
   </insert>
   
   <!-- 对应userDao中的updateUser方法 -->
   <update id="updateUser" parameterType="com.dy.entity.User">
           update user set name = #{name}, password = #{password}, age = #{age}, deleteFlag = #{deleteFlag}
               where id = #{id};
   </update>
    
   <!-- 对应userDao中的deleteUser 方法 --> 
   <delete id="deleteUser" parameterType="com.dy.entity.User">
           delete from user where id = #{id};
   </delete>
</mapper>
```

这样，一个简单的映射关系就建立了。仔细观察上面parameterType,  "com.dy.entity.User"，包名要是再长点呢，每次都这样写，写得蛋疼了。别忘了之前讲的 typeAliases（别名）， 那么这个地方，用上别名，岂不是技能跟蛋疼的长长的包名说拜拜了。好啦，咱们配上别名，在哪儿配？ 当然是在mybatis 的全局配置文件（我这儿名字是mybatis-conf.xml）， 不要认为是在mapper的配置文件里面配置哈。

mybatis-conf.xml:

```
<typeAliases>
      <!--
      通过package, 可以直接指定package的名字， mybatis会自动扫描你指定包下面的javabean,
      并且默认设置一个别名，默认的名字为： javabean 的首字母小写的非限定类名来作为它的别名。
      也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user) 
      <package name="com.dy.entity"/>
       -->
      <typeAlias alias="user" type="com.dy.entity.User"/>
  </typeAliases>
```

这样，一个别名就取好了，咱们可以把上面的 com.dy.entity.User 都直接改为user 了。 这多方便呀！

我这儿数据库用的是mysql, 我把user表的主键id 设置了自动增长， 以上代码运行正常， 那么问题来了（当然，我不是要问学挖掘机哪家强），我要是换成oracle数据库怎么办？ oracle 可是不支持id自增长啊？ 怎么办？请看下面：

```
<!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
           <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id 
           
        <selectKey resultType="int" order="BEFORE" keyProperty="id">
              select seq_user_id.nextval as id from dual
        </selectKey>
        
        -->   
           insert into user(id, name, password, age, deleteFlag) 
               values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
   </insert>
```

同理，如果我们在使用mysql的时候，想在数据插入后返回插入的id, 我们也可以使用 selectKey 这个元素：

```
<!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
           <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id 
           
        <selectKey resultType="int" order="BEFORE" keyProperty="id">
              select seq_user_id.nextval as id from dual
        </selectKey>
        
        --> 
        
        <!-- mysql插入数据后，获取id -->
        <selectKey keyProperty="id" resultType="int" order="AFTER" >
               SELECT LAST_INSERT_ID() as id
           </selectKey>
          
           insert into user(id, name, password, age, deleteFlag) 
               values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
   </insert>
```

这儿，我们就简单提一下 <selectKey> 这个元素节点吧:

```
<selectKey
        <!-- selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
        keyProperty="id"
        <!-- 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 -->
        resultType="int"
        <!-- 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 -->
        order="BEFORE"
        <!-- 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 -->
        statementType="PREPARED">
```

select无疑是我们最常用，也是最复杂的，mybatis通过resultMap能帮助我们很好地进行高级映射。下面就开始看看select 以及 resultMap的用法：

```
<select
        <!--  1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致  -->
     
     id="selectPerson"
     
     <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
     parameterType="int"
     
     <!-- 3. resultType (resultType 与 resultMap 二选一配置)
         resultType用以指定返回类型，指定的类型可以是基本类型，可以是java容器，也可以是javabean -->
     resultType="hashmap"
     
     <!-- 4. resultMap (resultType 与 resultMap 二选一配置)
         resultMap用于引用我们通过 resultMap标签定义的映射类型，这也是mybatis组件高级复杂映射的关键 -->
     resultMap="personResultMap"
     
     <!-- 5. flushCache (可选配置)
         将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false -->
     flushCache="false"
     
     <!-- 6. useCache (可选配置)
         将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true -->
     useCache="true"
     
     <!-- 7. timeout (可选配置) 
         这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）-->
     timeout="10000"
     
     <!-- 8. fetchSize (可选配置) 
         这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动)-->
     fetchSize="256"
     
     <!-- 9. statementType (可选配置) 
         STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED-->
     statementType="PREPARED"
     
     <!-- 10. resultSetType (可选配置) 
         FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）-->
     resultSetType="FORWARD_ONLY">
```

 不过有个问题值得思考：在mybatis中如何处理这种一对多， 甚至于多对多，一对一的关系呢？

这儿，就不得不提到 resultMap 这个东西， mybatis的resultMap功能可谓十分强大，能够处理复杂的关系映射， 那么resultMap 该怎么配置呢？ 别急，这就来了：

```
<!-- 
        1.type 对应类型，可以是javabean, 也可以是其它
        2.id 必须唯一， 用于标示这个resultMap的唯一性，在使用resultMap的时候，就是通过id指定
     -->
    <resultMap type="" id="">
    
        <!-- id, 唯一性，注意啦，这个id用于标示这个javabean对象的唯一性， 不一定会是数据库的主键（不要把它理解为数据库对应表的主键） 
            property属性对应javabean的属性名，column对应数据库表的列名
            （这样，当javabean的属性与数据库对应表的列名不一致的时候，就能通过指定这个保持正常映射了）
        -->
        <id property="" column=""/>
        
        <!-- result与id相比， 对应普通属性 -->    
        <result property="" column=""/>
        
        <!-- 
            constructor对应javabean中的构造方法
         -->
        <constructor>
            <!-- idArg 对应构造方法中的id参数 -->
            <idArg column=""/>
            <!-- arg 对应构造方法中的普通参数 -->
            <arg column=""/>
        </constructor>
        
        <!-- 
            collection，对应javabean中容器类型, 是实现一对多的关键 
            property 为javabean中容器对应字段名
            column 为体现在数据库中列名
            ofType 就是指定javabean中容器指定的类型
        -->
        <collection property="" column="" ofType=""></collection>
        
        <!-- 
            association 为关联关系，是实现N对一的关键。
            property 为javabean中容器对应字段名
            column 为体现在数据库中列名
            javaType 指定关联的类型
         -->
        <association property="" column="" javaType=""></association>
    </resultMap>
```

本文将介绍mybatis强大的动态SQL。

那么，问题来了： 什么是动态SQL? 动态SQL有什么作用？

　　传统的使用JDBC的方法，相信大家在组合复杂的的SQL语句的时候，需要去拼接，稍不注意哪怕少了个空格，都会导致错误。Mybatis的动态SQL功能正是为了解决这种问题， 其通过 if, choose, when, otherwise, trim, where, set, foreach标签，可组合成非常灵活的SQL语句，从而提高开发人员的效率。下面就去感受Mybatis动态SQL的魅力吧：

**1. if:   你们能判断，我也能判断！**

作为程序猿，谁不懂 if !  在mybatis中也能用 if 啦：

```
<select id="findUserById" resultType="user">
           select * from user where 
           <if test="id != null">
               id=#{id}
           </if>
            and deleteFlag=0;
</select>
```

上面例子： 如果传入的id 不为空， 那么才会SQL才拼接id = #{id}。 这个相信大家看一样就能明白，不多说。

细心的人会发现一个问题：“你这不对啊！ 要是你传入的id为null,  那么你这最终的SQL语句不就成了 select * from user where and deleteFlag=0,  这语句有问题！”

是啊，这时候，mybatis的 where 标签就该隆重登场啦：

**2. where, 有了我，SQL语句拼接条件神马的都是浮云！**

咱们通过where改造一下上面的例子：

```
<select id="findUserById" resultType="user">
           select * from user            <where>
               <if test="id != null">
                   id=#{id}
               </if>
               and deleteFlag=0;
           </where>
 </select>
```

有些人就要问了： “你这都是些什么玩意儿！ 跟上面的相比， 不就是多了个where标签嘛！ 那这个还会不会出现  select * from user where and deleteFlag=0 ？”

的确，从表面上来看，就是多了个where标签而已， 不过实质上， mybatis是对它做了处理，当它遇到AND或者OR这些，它知道怎么处理。其实我们可以通过 trim 标签去自定义这种处理规则。

**3. trim :  我的地盘，我做主！**

上面的where标签，其实用trim 可以表示如下：

```
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
```

它的意思就是： 当WHERE后紧随AND或则OR的时候，就去除AND或者OR。 除了WHERE以外， 其实还有一个比较经典的实现，那就是SET。

**4. set:  信我，不出错！**

```
<update id="updateUser" parameterType="com.dy.entity.User">
           update user set 
           <if test="name != null">
               name = #{name},
           </if> 
           <if test="password != null">
               password = #{password},
           </if> 
           <if test="age != null">
               age = #{age}
           </if> 
           <where>
               <if test="id != null">
                   id = #{id}
               </if>
               and deleteFlag = 0;
           </where>
</update>
```

问题又来了： “如果我只有name不为null,  那么这SQL不就成了 update set name = #{name}, where ........ ?  你那name后面那逗号会导致出错啊！”

是的，这时候，就可以用mybatis为我们提供的set 标签了。下面是通过set标签改造后：

```
<update id="updateUser" parameterType="com.dy.entity.User">
           update user        <set>
          <if test="name != null">name = #{name},</if> 
             <if test="password != null">password = #{password},</if> 
             <if test="age != null">age = #{age},</if> 
        </set>
           <where>
               <if test="id != null">
                   id = #{id}
               </if>
               and deleteFlag = 0;
           </where>
</update>
```

这个用trim 可表示为：

```
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

WHERE是使用的 prefixOverrides（前缀）， SET是使用的 suffixOverrides （后缀）， 看明白了吧！

**5. foreach:  你有for, 我有foreach, 不要以为就你才屌！**

java中有for, 可通过for循环， 同样， mybatis中有foreach, 可通过它实现循环，循环的对象当然主要是java容器和数组。

```
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

将一个 List 实例或者数组作为参数对象传给 MyBatis，当这么做的时候，MyBatis 会自动将它包装在一个 Map 中并以名称为键。List 实例将会以“list”作为键，而数组实例的键将是“array”。同样， 当循环的对象为map的时候，index其实就是map的key。

**6. choose:  我选择了你，你选择了我！**

Java中有switch,  mybatis有choose。

```
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

以上例子中： 当title和author都不为null的时候， 那么选择二选一（前者优先）， 如果都为null, 那么就选择 otherwise中的， 如果tilte和author只有一个不为null, 那么就选择不为null的那个。

纵观mybatis的动态SQL， 强大而简单， 相信大家简单看一下就能使用了。

好啦，本次就写到这！下篇文章将结合mybatis的源码分析一次sql语句执行的整个过程。

##### SQL 执行流程

SqlSessionFactory 与 SqlSession.

　　通过前面的章节对于mybatis 的介绍及使用，大家都能体会到SqlSession的重要性了吧， 没错，从表面上来看，咱们都是通过SqlSession去执行sql语句（注意：是从表面看，实际的待会儿就会讲）。那么咱们就先看看是怎么获取SqlSession的吧：

首先，SqlSessionFactoryBuilder去读取mybatis的配置文件，然后build一个DefaultSqlSessionFactory。源码如下：

```
/**
   * 一系列的构造方法最终都会调用本方法（配置文件为Reader时会调用本方法，还有一个InputStream方法与此对应）
   * @param reader
   * @param environment
   * @param properties
   * @return
   */
  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      //通过XMLConfigBuilder解析配置文件，解析的配置相关信息都会封装为一个Configuration对象
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      //这儿创建DefaultSessionFactory对象
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

  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```

当我们获取到SqlSessionFactory之后，就可以通过SqlSessionFactory去获取SqlSession对象。源码如下：

```
/**
   * 通常一系列openSession方法最终都会调用本方法
   * @param execType 
   * @param level
   * @param autoCommit
   * @return
   */
  private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      //通过Confuguration对象去获取Mybatis相关配置信息, Environment对象包含了数据源和事务的配置
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      //之前说了，从表面上来看，咱们是用sqlSession在执行sql语句， 实际呢，其实是通过excutor执行， excutor是对于Statement的封装
      final Executor executor = configuration.newExecutor(tx, execType);
      //关键看这儿，创建了一个DefaultSqlSession对象
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

通过以上步骤，咱们已经得到SqlSession对象了。接下来就是该干嘛干嘛去了（话说还能干嘛，当然是执行sql语句咯）。看了上面，咱们也回想一下之前写的Demo, 

```
SqlSessionFactory sessionFactory = null;  
String resource = "mybatis-conf.xml";  
try {
     //SqlSessionFactoryBuilder读取配置文件
    sessionFactory = new SqlSessionFactoryBuilder().build(Resources  
              .getResourceAsReader(resource));
} catch (IOException e) {  
    e.printStackTrace();  
}    //通过SqlSessionFactory获取SqlSessionSqlSession sqlSession = sessionFactory.openSession();
```

还真这么一回事儿，对吧！ 

SqlSession咱们也拿到了，咱们可以调用SqlSession中一系列的select...,  insert..., update..., delete...方法轻松自如的进行CRUD操作了。 就这样？ 那咱配置的映射文件去哪儿了？  别急， 咱们接着往下看：

利器之MapperProxy:

在mybatis中，通过MapperProxy动态代理咱们的dao， 也就是说， 当咱们执行自己写的dao里面的方法的时候，其实是对应的mapperProxy在代理。那么，咱们就看看怎么获取MapperProxy对象吧：

通过SqlSession从Configuration中获取。源码如下：

```
/**
   * 什么都不做，直接去configuration中找， 哥就是这么任性
   */
  @Override
  public <T> T getMapper(Class<T> type) {
    return configuration.<T>getMapper(type, this);
  }
```

SqlSession把包袱甩给了Configuration, 接下来就看看Configuration。源码如下：

```
/**
   * 烫手的山芋，俺不要，你找mapperRegistry去要
   * @param type
   * @param sqlSession
   * @return
   */
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
  }
```

Configuration不要这烫手的山芋，接着甩给了MapperRegistry， 那咱看看MapperRegistry。 源码如下：

```
/**
   * 烂活净让我来做了，没法了，下面没人了，我不做谁来做
   * @param type
   * @param sqlSession
   * @return
   */
  @SuppressWarnings("unchecked")
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    //能偷懒的就偷懒，俺把粗活交给MapperProxyFactory去做
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      //关键在这儿
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }
```

MapperProxyFactory是个苦B的人，粗活最终交给它去做了。咱们看看源码：

```
/**
   * 别人虐我千百遍，我待别人如初恋
   * @param mapperProxy
   * @return
   */
  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    //动态代理我们写的dao接口
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
  
  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }
```

通过以上的动态代理，咱们就可以方便地使用dao接口啦， 就像之前咱们写的demo那样：

```
 UserDao userMapper = sqlSession.getMapper(UserDao.class);  
 User insertUser = new User();
```

这下方便多了吧， 呵呵， 貌似mybatis的源码就这么一回事儿啊。

别急，还没完， 咱们还没看具体是怎么执行sql语句的呢。

接下来，咱们才要真正去看sql的执行过程了。

上面，咱们拿到了MapperProxy, 每个MapperProxy对应一个dao接口， 那么咱们在使用的时候，MapperProxy是怎么做的呢？ 源码奉上：

```
/**
   * MapperProxy在执行时会触发此方法
   */
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    //二话不说，主要交给MapperMethod自己去管
    return mapperMethod.execute(sqlSession, args);
  }
```

MapperMethod

```
/**
   * 看着代码不少，不过其实就是先判断CRUD类型，然后根据类型去选择到底执行sqlSession中的哪个方法，绕了一圈，又转回sqlSession了
   * @param sqlSession
   * @param args
   * @return
   */
  public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    if (SqlCommandType.INSERT == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
    } else if (SqlCommandType.UPDATE == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
    } else if (SqlCommandType.DELETE == command.getType()) {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
    } else if (SqlCommandType.SELECT == command.getType()) {
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
      }
    } else {
      throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName() 
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }
```

既然又回到SqlSession了， 那么咱们就看看SqlSession的CRUD方法了，为了省事，还是就选择其中的一个方法来做分析吧。这儿，咱们选择了selectList方法：

```
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      MappedStatement ms = configuration.getMappedStatement(statement);
      //CRUD实际上是交给Excetor去处理， excutor其实也只是穿了个马甲而已，小样，别以为穿个马甲我就不认识你嘞！
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

然后，通过一层一层的调用，最终会来到doQuery方法， 这儿咱们就随便找个Excutor看看doQuery方法的实现吧，我这儿选择了SimpleExecutor:

```
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      stmt = prepareStatement(handler, ms.getStatementLog());
      //StatementHandler封装了Statement, 让 StatementHandler 去处理
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }
```

接下来，咱们看看StatementHandler 的一个实现类 PreparedStatementHandler（这也是我们最常用的，封装的是PreparedStatement）, 看看它使怎么去处理的：

```
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
     //到此，原形毕露， PreparedStatement, 这个大家都已经滚瓜烂熟了吧
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    //结果交给了ResultSetHandler 去处理
    return resultSetHandler.<E> handleResultSets(ps);
  }
```

到此， 一次sql的执行流程就完了。 我这儿仅抛砖引玉，建议有兴趣的去看看Mybatis3的源码。

##### 参数设置

Mybatis框架极大地简化了ORM，让使用者自定义[sql](https://so.csdn.net/so/search?q=sql&spm=1001.2101.3001.7020)语句，再将查询结果映射到Java类中，其中很关键的一部分就是，将用户写的sql语句（以xml或者注解形式）解析成数据库可执行的sql语句。

本文将会介绍这整个的解析过程。

本文将以一下的注解定义为例，剖析整个解析过程。

```
@Select({"<script>",
            "SELECT account",
            "FROM user",
            "WHERE id IN",
            "<foreach item='item' index='index' collection='list'",
            "open='(' separator=',' close=')'>",
            "#{item}",
            "</foreach>",
            "</script>"})
    List<String> selectAccountsByIds(@Param("list") int[] ids);
```

Sql解析其实分成了两部分，第一部分是将注解/xml中定义的sql语句转化成内存中的MappedStatement，也就是将script部分转化成内存中的MappedStatement，这部分发生在Mybatis初始化的时候，第二部分则是根据MappedStatement以及传入的参数，生成可以直接执行的sql语句，这部分发生在mapper函数被调用的时候。
从注解/xml 定义到MappedStatement

首先，我们要明白什么是MappedStatement，先来看看MappedStatment的类定义。

```
public final class MappedStatement {

  private String resource; //来源，一般为文件名或者是注解的类名
  private Configuration configuration; //Mybatis的全局唯一Configuration
  private String id; //标志符，可以用于缓存
  private Integer fetchSize; //每次需要查询的行数（可选）
  private Integer timeout;//超时时间
  private StatementType statementType;//语句类型，决定最后将使用Statement, PreparedStatement还是CallableStatement进行查询
  private ResultSetType resultSetType;//结果集的读取类型，与java.sql.ResultSet中的类型对应。
  private SqlSource sqlSource;//Mybatis中的sqlSource，保存了初次解析的结果
  private Cache cache;//缓存空间
  private ParameterMap parameterMap;//保存了方法参数与sql语句中的参数对应关系
  private List<ResultMap> resultMaps;//可选，定义结果集与Java类型中的字段映射关系
  private boolean flushCacheRequired;//是否立即写入
  private boolean useCache;//是否使用缓存
  private boolean resultOrdered;//可选，默认为false
  private SqlCommandType sqlCommandType;//Sql执行类型（增、删、改、查）
  private KeyGenerator keyGenerator;//可选，键生成器
  private String[] keyProperties;//可选，作为键的属性
  private String[] keyColumns;//可选，键的列
  private boolean hasNestedResultMaps;//是否有嵌套的映射关系
  private String databaseId;//数据库的id
  private Log statementLog;//logger
  private LanguageDriver lang;//解析器
  private String[] resultSets;//可选，数据集的名称
 }
```

可以说，MappedStatement里面包含了Mybatis有关Sql执行的所有特性，在这里我们能够找到Mybatis在Sql执行层面支持哪些特性，比如说可以支持定制化的数据库列到Java属性的映射，定制化的主键生成器。

接下来，我们看一些具体的处理逻辑，也是比较有意思的部分。

```
@Override
  public SqlSource createSqlSource(Configuration configuration, String script, Class<?> parameterType) {
    // issue #3
    if (script.startsWith("<script>")) {
      XPathParser parser = new XPathParser(script, false, configuration.getVariables(), new XMLMapperEntityResolver());
      return createSqlSource(configuration, parser.evalNode("/script"), parameterType);
    } else {
      // issue #127
      script = PropertyParser.parse(script, configuration.getVariables());
      TextSqlNode textSqlNode = new TextSqlNode(script);
      if (textSqlNode.isDynamic()) {
        return new DynamicSqlSource(configuration, textSqlNode);
      } else {
        return new RawSqlSource(configuration, script, parameterType);
      }
    }
  }

```

在XMLLanguageDriver中，真正发生了解析我们定义在XML／注解中的语句，这里的解析分成了两部分，第一部分，初始化一个Parser，第二部分，对Parser解析出来的节点再进行下一步解析。

而最后生成的SqlSource也分成了两个种，分别为RawSqlSource以及DynamicSqlSource。这样做的好处是，RawSqlSource可以在参数替换完成后直接执行，更加简单高效。

接下来再来看看解析具体Node的方法，`XMLScriptBuilder`

```
List<SqlNode> parseDynamicTags(XNode node) {
    List<SqlNode> contents = new ArrayList<SqlNode>();
    NodeList children = node.getNode().getChildNodes();
    for (int i = 0; i < children.getLength(); i++) {
      XNode child = node.newXNode(children.item(i));
      if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE || child.getNode().getNodeType() == Node.TEXT_NODE) {
        String data = child.getStringBody("");
        TextSqlNode textSqlNode = new TextSqlNode(data);
        if (textSqlNode.isDynamic()) {
          contents.add(textSqlNode);
          isDynamic = true;
        } else {
          contents.add(new StaticTextSqlNode(data));
        }
      } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) { // issue #628
        String nodeName = child.getNode().getNodeName();
        //使用具体的Handler进行处理
        NodeHandler handler = nodeHandlers(nodeName);
        if (handler == null) {
          throw new BuilderException("Unknown element <" + nodeName + "> in SQL statement.");
        }
        handler.handleNode(child, contents);
        isDynamic = true;
      }
    }
    return contents;
  }
```

所以，由上可知，最后我们写在Sql定义中的</foreach>这样的元素，会由实际的handler（ForEachHandler）来进行处理，并且最后也会生成特定的SqlNode用于动态Sql的生成。这样就确保了Mybatis对于元素可扩展性的支持，同时保证了Sql的动态化。

以上，就是从我们的定义，到MappedStatement的整个过程，在这个过程里面，结构化的XML语言被转化成了具体内存里面的一个个SqlNode，而像ResultMap这样的注解，则被转化成了MappedStatement里面其它的具体属性。

接下来，我们将会使用MappedStatement，进行可执行的Sql生成。

从MappedStatement到可执行的Sql
从MappedStatement到可执行的Sql部分相对来说就比较简单了，里面只涉及到从RawSqlSource/DynamicSqlSource到StaticSqlSource的转换。
而在`DynamicSqlSource`中具体代码如下：

```
@Override
  public BoundSql getBoundSql(Object parameterObject) {
    DynamicContext context = new DynamicContext(configuration, parameterObject);
    //首先，通过node的apply方法，动态生成占位符，以及相应的参数映射关系
    rootSqlNode.apply(context);
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();
    //然后，再使用SqlSourceBuilder转化成staticSqlSource，这里也会对参数映射关系进行解析
    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());
    //这里的bouldSql中的sql就已经是可执行的sql了
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    for (Map.Entry<String, Object> entry : context.getBindings().entrySet()) {
      boundSql.setAdditionalParameter(entry.getKey(), entry.getValue());
    }
    return boundSql;
  }

```

在这里，我们能够看到之前由XMLNode转化而成的SqlNode起作用了，通过其中的Apply方法，会根据参数的长度来添加动态的?占位符。

而返回的StaticSqlSource将会交由Executor进行包含参数设定，执行结构映射成Java实例在内的一系列操作，这个将会在后续的文章中继续解析。
通过MethodSignature将参数转化为Map

```
public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    if (args == null || paramCount == 0) {
      return null;
    } else if (!hasParamAnnotation && paramCount == 1) {
      return args[names.firstKey()];
    } else {
      final Map<String, Object> param = new ParamMap<Object>();
      int i = 0;
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
      //首先，根据@param中注解的值，将相关参数添加到Map中
        param.put(entry.getValue(), args[entry.getKey()]);
        // 然后，再将参数的通用名，也添加到Map中
        final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
```

通过以上的转换之后，我们在后续的函数调用中，就可以以键值对的形式来获取方法参数的值了。

DynamicSqlSource中的参数解析
在上一片文章中，我们提到过，SqlNode中的apply方法，会将我们定义在注解中的Sql，转化成带有占位符的Sql。而实际上，除了Sql的转化之外，它还会向DynamicContext中添加Binding（参数值与对应键的绑定）。

`ForeachSqlNode`中的`apply`方法

```
public boolean apply(DynamicContext context) {
    Map<String, Object> bindings = context.getBindings();
    //首先，通过解析器，将参数中的值解析成为一个可遍历的集合。
    //初始状态下，bindins中只有_parameter -> { 'list' -> [1,2]}
    //提取出来后的集合为[1,2]
    final Iterable<?> iterable = evaluator.evaluateIterable(collectionExpression, bindings);
    if (!iterable.iterator().hasNext()) {
      return true;
    }
    boolean first = true;
    applyOpen(context);
    int i = 0;
    for (Object o : iterable) {
      DynamicContext oldContext = context;
      if (first) {
        context = new PrefixedContext(context, "");
      } else if (separator != null) {
        context = new PrefixedContext(context, separator);
      } else {
          context = new PrefixedContext(context, "");
      }
      int uniqueNumber = context.getUniqueNumber();
      // Issue #709 
      if (o instanceof Map.Entry) {
        @SuppressWarnings("unchecked") 
        Map.Entry<Object, Object> mapEntry = (Map.Entry<Object, Object>) o;
        applyIndex(context, mapEntry.getKey(), uniqueNumber);
        applyItem(context, mapEntry.getValue(), uniqueNumber);
      } else {
        //将对应的binding添加到DynamicContext中。
        //第一次遍历时，增加到binding 为 { '__frch_index_0' -> 0 }
        applyIndex(context, i, uniqueNumber);
        //第一次遍历时，增加到binding 为 { '__frch_item_0' -> 1 }
        applyItem(context, o, uniqueNumber);
      }
      contents.apply(new FilteredDynamicContext(configuration, context, index, item, uniqueNumber));
      if (first) {
        first = !((PrefixedContext) context).isPrefixApplied();
      }
      context = oldContext;
      i++;
    }
    applyClose(context);
    return true;
  }
```

当SqlNode解析完成后，我们得到的DynamicContext中Sql如下：

```
 SELECT account FROM user WHERE id IN  (   #{__frch_item_0}  ,  #{__frch_item_1}  )   
```

而在DynamicContext中的binding，则如下所示：

```
'__frch_index_0' -> 0,
'__frch_item_0' -> 1,
'__frch_index_1' -> 1,
'__frch_item_0' -> 2,
'_parameter' -> { 'list' -> [1,2] , 'param1' ->[1,2] },
'_databaseID' -> null
```

SqlSourceBuilder中获取ParameterMapping

虽然在DynamicContext中已经有了Bindings，但是Mybatis并不会直接使用这些binding进行查询。它会从含有占位符的语句中提取ParameterMapping关系，然后再根据ParameterMapping来对参数进行设置。

```
public SqlSource parse(String originalSql, Class<?> parameterType, Map<String, Object> additionalParameters) {
    //ParameterMappingTokenHandler，在查找到特定的token之后，对token进行处理，并且返回处理后的字符串
    ParameterMappingTokenHandler handler = new ParameterMappingTokenHandler(configuration, parameterType, additionalParameters);
    //只负责找到被#{}包围的字符串，然后交由tokenHandler进行处理
    GenericTokenParser parser = new GenericTokenParser("#{", "}", handler);
    String sql = parser.parse(originalSql);
    return new StaticSqlSource(configuration, sql, handler.getParameterMappings());
  }
```

ParameterMappingTokenHandler

```
public String handleToken(String content) {
      //首先，使用token中的标志符构造ParameterMapping，然后返回"?"
      parameterMappings.add(buildParameterMapping(content));
      return "?";
    }

```

而在这一次处理之后，我们就能够能到可以被直接执行的Sql了。

```
SELECT account FROM user WHERE id IN  (   ?  ,  ?  )
```

同时，也生成了对应的ParameterMapping（暂时忽略一些其它属性）

```
[
{ property: '__frch_item_0'},
{ property: '__frch_item_1'}
]
```

最后，在构造BoundSql时，Mybatis还会做如下的事情：

```
//将默认的参数，也作为BoundSql的默认参数
BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    //在context中生成的binding，则会做为额外的参数，也传给boundSql
    for (Map.Entry<String, Object> entry : context.getBindings().entrySet()) {
      boundSql.setAdditionalParameter(entry.getKey(), entry.getValue());
    }
    return boundSql;
```

参数的使用——DefaultParameterHandler

在这个类里面，我们只需要关注一个方法`setParameters`。

```
public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
      //parameterMapping是根据Sql中占位符逐个生成的，因此数组中的顺序也等同于sql中对应参数的顺序，直接进行遍历以及参数设置即可。
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
          String propertyName = parameterMapping.getProperty();
          if (boundSql.hasAdditionalParameter(propertyName)) { 
          // 首先，从额外的参数中获取参数值，获取顺序很重要，这个与Mybatis中的issue #448有关
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } else {
            //然后，再从parameterObject中进行获取。
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          JdbcType jdbcType = parameterMapping.getJdbcType();
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
            //考虑到Java类型与Mysql类型还有一个映射关系，所以使用typeHandler进行处理
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
```

##### 主键生成

正常情况下insert语句不会返回插入后所生成的主键，返回的是受影响的记录数。如果在插入一条数据后希望能得到数据库生成的主键(无论是oracle的自增序列，还是mysql的自增主键)，则可以使用KeyGenerator接口。

使用方式为在`<insert/>`或`<update/>`标签上配置useGeneratedKeys属性，关于useGeneratedKeys、keyProperty和keyColumn属性，官方文档对其的描述为：

| **useGeneratedKeys** | **（仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。** |
| -------------------- | ------------------------------------------------------------ |
| keyProperty          | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| keyColumn            | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |

Mapper.xml

```
<mapper namespace="userMapper">
    <insert id="addUser" parameterType="userScope" useGeneratedKeys="true" keyProperty="id">
        insert user(user_name,pass_word,address) values (#{userName},#{passWord},#{address})
    </insert>
    <insert id="insertSelectKey" parameterType="userScope">
        <selectKey keyProperty="id" resultType="int">
            select LAST_INSERT_ID()
        </selectKey>
        insert user(user_name,pass_word,address) values (#{userName},#{passWord},#{address})
    </insert>
</mapper>
```

上面写了两种方式用来获取自增长主键。一种是使用 ***\*useGemeratedKeys\****，一种是使用 ***\*<selectKey/>\****的方式。这两种方式获取自增长主键的ID的方式是一样的。都是从参数对象中获取自增长主键值。下面我们来根据源码来看一下MyBatis是怎么做的。

PreparedStatementHandler

```
 public int update(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();//这里是调用的JDBC4PreparedStatement执行sql
    int rows = ps.getUpdateCount();//获取受影响的行数，也就是调用插入操作的时候返回的真正值。
    Object parameterObject = boundSql.getParameterObject();//获取参数对象
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();//判断是那种KeyGenerator
    keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);//根据KeyGenerator去set主键值
    return rows;
  }
```

KeyGenerator接口的定义如下：

```
public interface KeyGenerator {
  //在sql执行之前
  void processBefore(Executor executor, MappedStatement ms, Statement stmt, Object parameter);
  //在sql执行之后
  void processAfter(Executor executor, MappedStatement ms, Statement stmt, Object parameter);
}
```

KeyGenerator有三个实现类：NoKeyGenerator、Jdbc3KeyGenerator、SelectKeyGenerator。

NoKeyGenerator为空实现，没有设置返回自增长主键值的插入操作

Jdbc3KeyGenerator用于在执行sql的时候取回生成的id，并将其保存在用户传入的实参中，

SelectKeyGenerator用于当前使用的数据库不支持主键自增的场景，用户自己编写sql语句来生成对应的自增id。

下面我们来分析Jdbc3KeyGenerator这个类。

```
 public void processBatch(MappedStatement ms, Statement stmt, Collection<Object> parameters) {
    ResultSet rs = null;
    try {
      rs = stmt.getGeneratedKeys();//获取执行结果值
      final Configuration configuration = ms.getConfiguration();
      final TypeHandlerRegistry typeHandlerRegistry = configuration.getTypeHandlerRegistry();
      final String[] keyProperties = ms.getKeyProperties();//获取设置的keyProperty
      final ResultSetMetaData rsmd = rs.getMetaData();
      TypeHandler<?>[] typeHandlers = null;
      if (keyProperties != null && rsmd.getColumnCount() >= keyProperties.length) {//如果设置了keyProperty，则在参数对象中放入处理之后的结果值
        for (Object parameter : parameters) {
          // there should be one row for each statement (also one for each parameter)
          if (!rs.next()) {
            break;
          }
          final MetaObject metaParam = configuration.newMetaObject(parameter);
          if (typeHandlers == null) {
            typeHandlers = getTypeHandlers(typeHandlerRegistry, metaParam, keyProperties, rsmd);//keyProperty设置的字段类型的处理器
          }
          populateKeys(rs, metaParam, keyProperties, typeHandlers);//在参数对象中设置keyProperty的字段值
        }
      }


 private void populateKeys(ResultSet rs, MetaObject metaParam, String[] keyProperties, TypeHandler<?>[] typeHandlers) throws SQLException {
    for (int i = 0; i < keyProperties.length; i++) {
      TypeHandler<?> th = typeHandlers[i];
      if (th != null) {
        Object value = th.getResult(rs, i + 1);
        metaParam.setValue(keyProperties[i], value);//在参数对象中设置keyProperty的字段值
      }
    }
  }


public void setValue(String name, Object value) {
    PropertyTokenizer prop = new PropertyTokenizer(name);
    if (prop.hasNext()) {
      MetaObject metaValue = metaObjectForProperty(prop.getIndexedName());
      if (metaValue == SystemMetaObject.NULL_META_OBJECT) {
        if (value == null && prop.getChildren() != null) {
          // don't instantiate child path if value is null
          return;
        } else {
          metaValue = objectWrapper.instantiatePropertyValue(name, prop, objectFactory);
        }
      }
      metaValue.setValue(prop.getChildren(), value);
    } else {
      objectWrapper.set(prop, value);//在参数对象中设置keyProperty的字段值（根据反射来操作的）
    }
  }

```

MyBatis3.4.0之后已经支持批量插入并获取自增主键值了。例子如下：

```
    public void addUserBatch(List<UserScope> list) {
 
        this.getSqlSession().insert("userMapper.addUserBatch",list);
        System.out.println(Arrays.toString(list.toArray()));
    }


    <insert id="addUserBatch" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="id" keyColumn="id">
      insert user(user_name,pass_word,address) values
      <foreach collection="list" item="item" separator=",">
          ( #{item.userName},#{item.passWord},#{item.address} )
      </foreach>
    </insert>

```

##### 结果集映射

##### 延迟加载

##### 插件体系

### 基础模块层

##### 解析器模块

##### 反射模块

##### 异常模块

##### 数据源模块

##### 事务模块

##### 缓存模块

##### 类型模块

##### IO 模块

##### 日志模块

##### 注解模块

##### Binding 模块

