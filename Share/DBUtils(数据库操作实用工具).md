# DbUtils手册

[TOC]



## 简介

### 概述

Apache Commons DbUtils是 Java 编程中的数据库操作实用工具，小巧简单实用。 

DBUtils 封装了对 JDBC 的操作，简化了 JDBC 操作，可以少写代码。 

### 优点

#### 无资源泄漏

DBUtils类确保不会发生资源泄漏。

#### 清理和清除代码

DBUtils类提供干净清晰的代码来执行数据库操作，而无需编写任何清理或资源泄漏防护代码。

#### Bean映射

DBUtils类支持从结果集中自动填充javabeans。

### 设计原则

#### 小

DBUtils库的体积很小，只有较少的类，因此易于理解和使用。

#### 透明

DBUtils库在后台没有做太多工作，它只需查询并执行。

#### 快速

DBUtils库类不会创建许多背景对象，并且在数据库操作执行中速度非常快。

## 入门

### POM

```xml
<!-- https://mvnrepository.com/artifact/commons-dbutils/commons-dbutils -->
<dependency>
   <groupId>commons-dbutils</groupId>
   <artifactId>commons-dbutils</artifactId>
   <version>1.7</version>
</dependency>
```

### 依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.9</version>
</dependency>
```

### 数据准备

创建一张数据表

**现有一张** **User** **表，其表结构如下：**

| id   | name | age  | email         |
| ---- | ---- | ---- | ------------- |
| 1    | 张三 | 18   | test1@163.com |
| 2    | 李四 | 20   | test2@163.com |
| 3    | 王五 | 28   | test3@163.com |
| 4    | 侯六 | 21   | test4@163.com |
| 5    | 杨七 | 24   | test5@163.com |

#### 数据库脚本

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
	id BIGINT(20) NOT NULL COMMENT '主键ID',
	name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
	age INT(11) NULL DEFAULT NULL COMMENT '年龄',
	email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY (id)
);

```

#### 数据库 Data

```sql
DELETE FROM user;

INSERT INTO user (id, name, age, email) VALUES
(1, '张三', 18, 'test1@163.com'),
(2, '李四', 20, 'test2@163.com'),
(3, '王五', 28, 'test3@163.com'),
(4, '侯六', 21, 'test4@163.com'),
(5, '杨七', 24, 'test5@163.com');

```

### 示例代码

User对象

```java
package com.demo.dbutils;

public class User {

    private Long id;

    private String name;

    private Integer age;

    private String email;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", email='" + email + '\'' +
                '}';
    }
}
```

执行JDBC应用代码

```java
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class HelloWorld {

    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

    private static final String URL = "jdbc:mysql://localhost:3306/test";

    private static final String USER = "root";

    private static final String PASSWORD = "root";


    public static void main(String[] args) throws SQLException {

        QueryRunner queryRunner = new QueryRunner();
        // 注册驱动
        DbUtils.loadDriver(JDBC_DRIVER);
        //加载连接
        Connection connection = DriverManager.getConnection(URL, USER, PASSWORD);
        try {
            // 执行查询
            User user = queryRunner.query(connection, "SELECT * FROM user WHERE id=?",
                    new BeanHandler<User>(User.class), "1");
            //从结果集中提取数据
            System.out.println(user.toString());
        } finally {
            //清理环境
            DbUtils.close(connection);
        }
    }

}
```

执行上面的代码，它会产生以下结果

```json
User{id=1, name='张三', age=18, email='test1@163.com'}
```

### 流程解析

构建JDBC应用程序涉及以下步骤 

| 步骤                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **注册JDBC驱动程序**   | 需要初始化驱动程序，以便可以打开与数据库的通信通道。         |
| **打开连接**           | 需要使用`DriverManager.getConnection()`方法创建一个`Connection`对象，该对象表示与数据库的物理连接。 |
| **执行查询**           | 需要使用类型为`Statement`的对象来构建和提交SQL语句到数据库。 |
| **从结果集中提取数据** | 要求您使用适当的`ResultSet.getXXX()`方法从结果集中检索数据。 |
| **清理环境**           | 需要显式关闭所有数据库资源而不依赖于JVM的垃圾收集。          |

## 基本用法CRUD

### 创建(Create)

在DBUtils中，使用`Insert`语句来创建记录。

#### 语法

```java
 String sql="INSERT INTO user (id, name, age, email) VALUES(?,?,?,?)";

            int total = queryRunner.update(connection, sql,8,"冯八", "15","test8@163.com");
```

#### 示例代码

```java
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Create {


    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

    private static final String URL = "jdbc:mysql://localhost:3306/test";

    private static final String USER = "root";

    private static final String PASSWORD = "root";


    public static void main(String[] args) throws SQLException {

        QueryRunner queryRunner = new QueryRunner();
        // 注册驱动
        DbUtils.loadDriver(JDBC_DRIVER);
        //加载连接
        Connection connection = DriverManager.getConnection(URL, USER, PASSWORD);
        try {
            // 执行查询
            String sql="INSERT INTO user (id, name, age, email) VALUES(?,?,?,?)";

            int total = queryRunner.update(connection, sql,8,"冯八", "15","test8@163.com");

            //从结果集中提取数据
           System.out.println(total+" records inserted");
        } finally {
            //清理环境
            DbUtils.close(connection);
        }
    }
}

```

结果如下

```
1 records inserted
```

### 读取(Read)

在DBUtils的中，使用`query`查询来读取数据库表中的记录。

#### 语法

```
 User user = queryRunner.query(connection, "SELECT * FROM user WHERE id=?",
                    new BeanHandler<User>(User.class), "1");
```

#### 示例代码

```sql
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Read {

    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

    private static final String URL = "jdbc:mysql://localhost:3306/test";

    private static final String USER = "root";

    private static final String PASSWORD = "root";


    public static void main(String[] args) throws SQLException {

        QueryRunner queryRunner = new QueryRunner();
        // 注册驱动
        DbUtils.loadDriver(JDBC_DRIVER);
        //加载连接
        Connection connection = DriverManager.getConnection(URL, USER, PASSWORD);
        try {
            // 执行查询
            User user = queryRunner.query(connection, "SELECT * FROM user WHERE id=?",
                    new BeanHandler<User>(User.class), "1");
            //从结果集中提取数据
            System.out.println(user.toString()+" records read ");
        } finally {
            //清理环境
            DbUtils.close(connection);
        }
    }


}

```

执行结果

```
User{id=1, name='张三', age=18, email='test1@163.com'} records read 
```

### 更新(Update)

在DBUtils的中，使用`update`查询来读取数据库表中的记录。

#### 语法

```
    String sql="Update user set age = ? where id =?";

    int total = queryRunner.update(connection, sql,8,1);
```

#### 示例代码

```sql
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Update {


    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

    private static final String URL = "jdbc:mysql://localhost:3306/test";

    private static final String USER = "root";

    private static final String PASSWORD = "root";


    public static void main(String[] args) throws SQLException {

        QueryRunner queryRunner = new QueryRunner();
        // 注册驱动
        DbUtils.loadDriver(JDBC_DRIVER);
        //加载连接
        Connection connection = DriverManager.getConnection(URL, USER, PASSWORD);
        try {
            // 执行查询
            String sql="Update user set age = ? where id =?";

            int total = queryRunner.update(connection, sql,8,1);

            //从结果集中提取数据
            System.out.println(total+" records updated");
        } finally {
            //清理环境
            DbUtils.close(connection);
        }
    }
}

```

### 删除(Delete)

在DBUtils的中，使用`update`查询来读取数据库表中的记录。

#### 语法

```
 String sql="DELETE FROM user where id =?";

 int total = queryRunner.update(connection, sql,8);
```

#### 示例代码

```java
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Delete {


    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

    private static final String URL = "jdbc:mysql://localhost:3306/test";

    private static final String USER = "root";

    private static final String PASSWORD = "root";


    public static void main(String[] args) throws SQLException {

        QueryRunner queryRunner = new QueryRunner();
        // 注册驱动
        DbUtils.loadDriver(JDBC_DRIVER);
        //加载连接
        Connection connection = DriverManager.getConnection(URL, USER, PASSWORD);
        try {
            // 执行查询
            String sql="DELETE FROM user where id =?";

            int total = queryRunner.update(connection, sql,8);

            //从结果集中提取数据
            System.out.println(total+" records deleted");
        } finally {
            //清理环境
            DbUtils.close(connection);
        }
    }
}

```

结果如下

```
1 records deleted
```

## 核心

**QueryRunner**中提供对sql语句操作的API

**ResultSetHandler**接口，用于定义select操作后，怎样封装结果集

**DBUtils**类，它就是一个工具类，定义了关闭资源与事务处理的方法

### QueryRunner

`org.apache.commons.dbutils.QueryRunner`类是DBUtils库中的中心类。 

它执行带有可插入策略的SQL查询来处理`ResultSets`。 

这个类是线程安全的。

```
QueryRunner(DataSourcr ds)
提供数据源（连接池），DbUtils底层自动维护连接connection

update（String sql，Obj...params）
执行更新数据

query（String sql，ResultSetHandler<T>rsh,Object...panrams)
执行查询

```

### ResultSetHandler

`org.apache.commons.dbutils.ResultSetHandler`接口负责将ResultSets转换为对象。

```
ArrayHandler：
适合取1条记录，把结果集中的第一行数据转成对象数组。

ArrayListHandler：
适合取多条记录，把结果集中的每一行数据都转成一个对象数组，再存放到List中。
 
BeanHandler：
将结果集中的第一行数据封装到一个对应的JavaBean实例中（把每条记录封装成对象，适合取一条记录）

BeanListHandler：
将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里。

MapHandler：
将结果集中的第一行数据封装到一个Map里，key是列名，value就是对应的值。

MapListHandler：
将结果集中的每一行数据都封装到一个Map里，然后再存放到List

ColumnListHandler：
将结果集中某一列的数据存放到List中。

KeyedHandler(name)：
将结果集中的每一行数据都封装到一个Map里(List<Map>)，再把这些map再存到一个map里，其key为指定的列。

ScalarHandler:将结果集第一行的某一列放到某个对象中。

```

### BeanHandler

`org.apache.commons.dbutils.BeanHandler`是`ResultSetHandler`接口的实现，负责将第一个`ResultSet`行转换为`JavaBean`。 

这个类是线程安全的。

### BeanListHandler

`org.apache.commons.dbutils.BeanListHandler`是`ResultSetHandler`接口的实现，负责将`ResultSet`行转换为Java Bean列表。 

### ArrayListHandler

`org.apache.commons.dbutils.ArrayListHandler`是`ResultSetHandler`接口的实现，负责将`ResultSet`行转换为`object[]`。 这个类是线程安全的。

### MapListHandler

`org.apache.commons.dbutils.MapListHandler`是`ResultSetHandler`接口的实现，负责将`ResultSet`行转换为Maps列表。 这个类是线程安全的。

## 高级

### 自定义处理程序

可以通过实现`ResultSetHandler`接口或扩展任何现有的`ResultSetHandler`实现来创建自己的自定义处理程序。 如果用户要想自定义业务返回类型

```
public class CustomerResultSetHandler <T> implements ResultSetHandler<T> {
    @Override
    public T handle(ResultSet rs) throws SQLException {
        return null;
    }

}
```

示例代码

```java
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class UserHandlerQuery {

    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

    private static final String URL = "jdbc:mysql://localhost:3306/test";

    private static final String USER = "root";

    private static final String PASSWORD = "root";


    public static void main(String[] args) throws SQLException {

        QueryRunner queryRunner = new QueryRunner();
        // 注册驱动
        DbUtils.loadDriver(JDBC_DRIVER);
        //加载连接
        Connection connection = DriverManager.getConnection(URL, USER, PASSWORD);
        try {
            // 执行查询
            User user = queryRunner.query(connection, "SELECT * FROM user WHERE id=?",
                    new UserHandler(), "1");
            //从结果集中提取数据
            System.out.println(user.toString());
        } finally {
            //清理环境
            DbUtils.close(connection);
        }
    }

}

```

结果如下

```
User{id=1, name='张三', age=8, email='张三@163.com'}
```

### 自定义行处理器

如果数据库表中的列名和等价的javabean对象名称不相似，那么我们可以通过使用自定义的`BasicRowProcessor`对象来映射它们。

```
package com.demo.dbutils;

import org.apache.commons.dbutils.BasicRowProcessor;
import org.apache.commons.dbutils.BeanProcessor;
import org.apache.commons.dbutils.handlers.BeanHandler;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

public class UserRowHandler extends BeanHandler<User> {

    public UserRowHandler() {
        super(User.class, new BasicRowProcessor(new BeanProcessor(mapColumnsToFields())));
    }

    @Override
    public User handle(ResultSet rs) throws SQLException {
        User user = super.handle(rs);
        user.setEmail(user.getName() +"@163.com");
        return user;
    }

    public static Map<String, String> mapColumnsToFields() {
        Map<String, String> columnsToFieldsMap = new HashMap<>();
        columnsToFieldsMap.put("ID", "id");
        columnsToFieldsMap.put("AGE", "age");
        return columnsToFieldsMap;
    }

}
```

### 使用DataSource

到目前为止，我们在使用QueryRunner时都是使用`Connection`对象。 也可以使用数据源。

#### 使用dbcp数据库连接池

Maven

```xml
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
```

它自己会依赖

```xml
<dependency>
      <groupId>commons-pool</groupId>
      <artifactId>commons-pool</artifactId>
      <version>1.5.4</version>
</dependency>
```

示例代码

```java
package com.demo.dbutils;

import org.apache.commons.dbutils.DbUtils;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;

import java.sql.SQLException;

public class DataSourceQuery {

    public static void main(String[] args) throws SQLException {
        QueryRunner queryRunner = new QueryRunner(CustomDataSource.getInstance());
        // 执行查询
        User user = queryRunner.query("SELECT * FROM user WHERE id=?",
                new BeanHandler<User>(User.class), "1");
        //从结果集中提取数据
        System.out.println(user.toString());
    }
}

```

#### 使用c3p0数据库连接池

```xml
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.1</version>
</dependency>
```

### 大字典clob

```
public void testClob() throws Exception {
    QueryRunner queryRunner = new QueryRunner(JDBCUtil.getDataSource());
    String sql = "insert into clob values(?)";
    File file = new File("c:/a.txt");
    Long l = file.length();
    char[] buffer = new char[l.intValue()];
    FileReader reader = new FileReader(file);
    reader.read(buffer);
    SerialClob clob = new SerialClob(buffer);
    queryRunner.update(sql, clob);
}

```

二进制blob

```
public void testBlob() throws Exception{
    QueryRunner queryRunner = new QueryRunner(JDBCUtil.getDataSource());
    String sql = "insert into blob values(?)";
    File file = new File("c:/a.jpg");
    Long l = file.length();
    byte[] buffer = new byte[l.intValue()];
    FileInputStream input = new FileInputStream(file);
    input.read(buffer);
    SerialBlob blob = new SerialBlob(buffer);
    queryRunner.update(sql,blob);
}

```

## 事务操作

#### 事务案例

使用ThreadLocal类，传递事务连接

```java
public class JdbcKit {

    /**
     * 获取连接池
     */
    public static ComboPooledDataSource ds = new ComboPooledDataSource();
    /**
     * 线程共享变量
     */
    public static ThreadLocal<Connection> container = new ThreadLocal<Connection>();

    /**
     * 获取线程变量
     *
     * @return
     */
    public static ThreadLocal<Connection> getContainer() {
        return container;
    }
  public static DataSource getDataSource() {
        return ds;
    }

    /**
     * 开启事务
     */
    public static void startTransaction() {
        Connection conn = container.get();
        if (conn == null) {
            conn = getConnection();
            container.set(conn);
        }
        try {
            conn.setAutoCommit(false);
        } catch (SQLException e) {
            throw new RuntimeException(e.getMessage(), e);
        }
    }

    /**
     * 提交事务
     */
    public static void commit() {
        Connection conn = container.get();
        if (conn != null) {
            try {
                conn.commit();
            } catch (SQLException e) {
                throw new RuntimeException(e.getMessage(), e);
            }
        }
    }

    /**
     * 回滚事务
     */
    public static void rollback() {
        Connection conn = container.get();
        if (conn != null) {
            try {
                conn.rollback();
            } catch (SQLException e) {
                throw new RuntimeException(e.getMessage(), e);
            }
        }
    }

    /**
     * 关闭连接
     */
    public static void close() {
        Connection conn = container.get();
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                throw new RuntimeException(e.getMessage(), e);
            } finally {
                container.remove();
            }
        }
    }


    /**
     * 获取数据库连接
     *
     * @return
     */
    public static Connection getConnection() {
        try {
            return ds.getConnection();
        } catch (Exception e) {
            throw new RuntimeException();
        }
    }


}

```

#### 模拟银行卡转账

##### 账户模型

```
public class Account {

    private Integer id;

    private BigDecimal money;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public BigDecimal getMoney() {
        return money;
    }

    public void setMoney(BigDecimal money) {
        this.money = money;
    }
}

```

##### 账户持久化

```
public class AccountDAO {

    /**
     * 连接
     */
    private Connection conn;

    public AccountDAO() {
        this.conn = JdbcKit.getContainer().get();
    }


    /**
     * 查询账户ID
     *
     * @param id
     * @return
     */
    public Account findAccount(int id) {
        //处理事务 ，无参构造
        QueryRunner runner = new QueryRunner();
        String sql = "select * from account where id=?";
        Object[] params = {id};
        try {
            //附加连接 处理者为BeanHandler
            return (Account) runner.query(conn, sql, params, new
                    BeanHandler(Account.class));
        } catch (Exception e) {
            throw new RuntimeException(e.getMessage(), e);
        }
    }

    /**
     * 更新账户
     *
     * @param a
     */
    public void updateAccount(Account a) {
        QueryRunner runner = new QueryRunner();
        String sql = "update account set money=? where id=?";
        Object[] params = {a.getMoney(), a.getId()};
        try {
            runner.update(conn, sql, params);
        } catch (SQLException e) {
            throw new RuntimeException(e.getMessage(), e);
        }
    }
}

```

##### 账户转账操作

```java
public class AccountService {

    /**
     * 转账方法
     *
     * @param fromID 原始账户
     * @param toID   目标账户
     * @param money  转入多少钱
     */

    public void trafferAccount(int fromID, int toID, BigDecimal money) {
        try {
            JdbcKit.startTransaction();
            AccountDAO dao = new AccountDAO();
            Account from = dao.findAccount(fromID);
            Account to = dao.findAccount(toID);
            from.setMoney(from.getMoney().subtract(money));
            to.setMoney(to.getMoney().add(money));
            dao.updateAccount(from);
            dao.updateAccount(to);
            JdbcKit.commit();
        } catch (Exception e) {
            JdbcKit.rollback();
            throw new RuntimeException();
        } finally {
            JdbcKit.close();
        }
    }
}

```

