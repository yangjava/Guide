**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10688617.html#_labelTop)

## Servlet简介



### Servlet定义

Servlet是一个Java应用程序，运行在服务器端，用来处理客户端请求并作出响应的程序。



### Servlet的特点

（1）Servlet对像，由Servlet容器（Tomcat）创建。

（2）Servlet是一个接口：位于javax.servlet包中。

（3）service方法用于接收用户的请求并返回响应。

（4）用户访问时多次被执行（可以统计网站的访问量）。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10688617.html#_labelTop)

## Servlet底层原理



### Servlet

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package javax.servlet;

import java.io.IOException;

public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们可以看出Servlet实际上是一个接口，其中包含init、getServletConfig、service、getServletInfo、destroy几个接口方法。

#### 实现Servlet接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package cn.chenhao;

import java.io.IOException;

import javax.servlet.Servlet;
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

public class FirstServlet implements Servlet {
    /**init方法*/
    @Override
    public void init(ServletConfig paramServletConfig) throws ServletException {
    }

    /**getServletConfig方法*/
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    /**service方法*/
    @Override
    public void service(ServletRequest paramServletRequest,
            ServletResponse paramServletResponse) throws ServletException,
            IOException {
    }

    /**getServletInfo方法*/
    @Override
    public String getServletInfo() {
        return null;
    }

    /**destroy方法*/
    @Override
    public void destroy() {
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们需要实现servlet接口里面所有的方法，这对于我们开发来说是很繁琐的。


Servlet 生命周期的方法: 以下方法都是由 Serlvet 容器负责调用.
1). 构造器: 只被调用一次. 只有第一次请求 Servlet 时, 创建 Servlet 的实例. 调用构造器. **这说明 Serlvet 的单实例的!**
2). init 方法: 只被调用一次. 在创建好实例后立即被调用. 用于初始化当前 Servlet.
3). service: 被多次调用. 每次请求都会调用 service 方法. 实际用于响应请求的.
4). destroy: 只被调用一次. 在当前 Servlet 所在的 WEB 应用被卸载前调用. 用于释放当前 Servlet 所占用的资源.

下面两个方法是开发者调用
1). getServletConfig: 返回一个ServletConfig对象，其中包含这个servlet初始化和启动参数.
2). getServletinfo: 返回有关servlet的信息，如作者、版本和版权.

#### **在 web.xml 文件中配置和映射这个 Servlet**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!-- 配置和映射 Servlet -->
<servlet>
    <!-- Servlet 注册的名字 -->
    <servlet-name>helloServlet</servlet-name>
    <!-- Servlet 的全类名 -->
    <servlet-class>com.atguigu.javaweb.HelloServlet</servlet-class>
</servlet>

<servlet-mapping>
    <!-- 需要和某一个 servlet 节点的 serlvet-name 子节点的文本节点一致 -->
    <servlet-name>helloServlet</servlet-name>
    <!-- 映射具体的访问路径: / 代表当前 WEB 应用的根目录. -->
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190411111053747-132608403.png)

 

#### load-on-startup 参数

1). 配置在 servlet 节点中:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<servlet>
    <!-- Servlet 注册的名字 -->
    <servlet-name>secondServlet</servlet-name>
    <!-- Servlet 的全类名 -->
    <servlet-class>com.atguigu.javaweb.SecondServlet</servlet-class>
    <!-- 可以指定 Servlet 被创建的时机 -->
    <load-on-startup>2</load-on-startup>
</servlet>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2). load-on-startup: 可以指定 Serlvet 被创建的时机. 若为负数, 则在第一次请求时被创建.若为 0 或正数, 则在当前 WEB 应用被Serlvet 容器加载时创建实例, 且数组越小越早被创建.

#### 关于 serlvet-mapping

1). 同一个Servlet可以被映射到多个URL上，即多个 <servlet-mapping> 元素的<servlet-name>子元素的设置值可以是同一个
Servlet的注册名。

2). 在Servlet映射到的URL中也可以使用 * 通配符，但是只能有两种固定的格式：
一种格式是“*.扩展名”，另一种格式是以正斜杠（/）开头并以“/*”结尾。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<servlet-mapping>
    <servlet-name>secondServlet</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>

OR

<servlet-mapping>
    <servlet-name>secondServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注意: 以下的既带 / 又带扩展名的不合法. 

```
<servlet-mapping>
    <servlet-name>secondServlet</servlet-name>
    <url-pattern>/*.action</url-pattern>
</servlet-mapping>
```

#### Servlet生命周期

实例化：在第一次访问或启动tomcat时，tomcat会调用此无参构造方法实例化servlet。

初始化：tomcat在实例化此servlet后，会立即调用init方法初始化servlet。

就绪：容器收到请求后调用servlet的service方法来处理请求。

销毁：容器依据自身算法删除servlet对象，删除前会调用destory方法

其中实例化，初始化，销毁只会执行一次，service方法执行多次，默认情况下servlet是在第一次接受到用户请求的情况下才会实例化，可以在web.xml中的<servlet><servlet>标签内添加一个<load-on-startup>1<load-on-startup>配置，此时在启动tomcat时会创建servlet实例。

#### ServletConfig

封装了 Serlvet 的配置信息, 并且可以获取 ServletContext 对象

1). 配置 Serlvet 的初始化参数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<servlet>
    <servlet-name>helloServlet</servlet-name>
    <servlet-class>com.atguigu.javaweb.HelloServlet</servlet-class>
    
    <!-- 配置 Serlvet 的初始化参数。 且节点必须在 load-on-startup 节点的前面 -->
    <init-param>
        <!-- 参数名 -->
        <param-name>user</param-name>
        <!-- 参数值 -->
        <param-value>root</param-value>
    </init-param>
    
    <init-param>
        <param-name>password</param-name>
        <param-value>1230</param-value>
    </init-param>
    
    <load-on-startup>-1</load-on-startup>
    
</servlet>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2). 获取初始化参数: 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
getInitParameter(String name): 获取指定参数名的初始化参数
getInitParameterNames(): 获取参数名组成的 Enumeration 对象. 

String user = servletConfig.getInitParameter("user");
System.out.println("user： " + user);

Enumeration<String> names = servletConfig.getInitParameterNames();
while(names.hasMoreElements()){
    String name = names.nextElement();
    String value = servletConfig.getInitParameter(name);
    System.out.println("^^" + name + ": " + value);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### ServletContext

1). 可以由 SerlvetConfig 获取:

```java
ServletContext servletContext = servletConfig.getServletContext();
```

2). 该对象代表当前 WEB 应用: 可以认为 SerlvetContext 是当前 WEB 应用的一个大管家. 可以从中获取到当前 WEB 应用的各个方面的信息.

①. 获取当前 WEB 应用的初始化参数

设置初始化参数: 可以为所有的 Servlet 所获取, 而 Servlet 的初始化参数只用那个 Serlvet 可以获取.

```xml
<!-- 配置当前 WEB 应用的初始化参数 -->
<context-param>
　　<param-name>driver</param-name>
　　<param-value>com.mysql.jdbc.Driver</param-value>
</context-param>
```

代码:

```java
ServletContext servletContext = servletConfig.getServletContext();
String driver = servletContext.getInitParameter("driver");
System.out.println("driver:" + driver);

Enumeration<String> names2 = servletContext.getInitParameterNames();
while(names2.hasMoreElements()){
　　String name = names2.nextElement();
　　System.out.println("-->" + name); 
}
```

②. 获取当前 WEB 应用的某一个文件在服务器上的绝对路径, 而不是部署前的路径

```java
String realPath = servletContext.getRealPath("/note.txt");
```

③. 获取当前 WEB 应用的名称: 

```java
String contextPath = servletContext.getContextPath();
```

④. 获取当前 WEB 应用的某一个文件对应的输入流. 

```java
InputStream is2 = servletContext.getResourceAsStream("/WEB-INF/classes/jdbc.properties");
```

#### 如何在 Serlvet 中获取请求信息

**1). Servlet 的 service() 方法用于应答请求: 因为每次请求都会调用 service() 方法**

```java
public void service(ServletRequest request, ServletResponse response)throws ServletException, IOException
```

ServletRequest: 封装了请求信息. 可以从中获取到任何的请求信息.
ServletResponse: 封装了响应信息, 如果想给用户什么响应, 具体可以使用该接口的方法实现.

这两个接口的实现类都是服务器给予实现的, 并在服务器调用 service 方法时传入.

**2). ServletRequest: 封装了请求信息. 可以从中获取到任何的请求信息.**

①. 获取请求参数:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
> String getParameter(String name): 根据请求参数的名字, 返回参数值. 

若请求参数有多个值(例如 checkbox), 该方法只能获取到第一个提交的值.

> String[] getParameterValues(String name): 根据请求参数的名字, 返回请求参数对应的字符串数组.

> Enumeration getParameterNames(): 返回参数名对应的 Enumeration 对象, 
类似于 ServletConfig(或 ServletContext) 的 getInitParameterNames() 方法.

> Map getParameterMap(): 返回请求参数的键值对: key: 参数名, value: 参数值, String 数组类型.
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

②. 获取请求的 URI:

```
HttpServletRequest httpServletRequest = (HttpServletRequest) request;
    
String requestURI = httpServletRequest.getRequestURI();
System.out.println(requestURI); //  /day_29/loginServlet
```

③. 获取请求方式: 

```
String method = httpServletRequest.getMethod();
System.out.println(method); //GET
```

④. 若是一个 GET 请求, 获取请求参数对应的那个字符串, 即 ? 后的那个字符串. 

```
String queryString = httpServletRequest.getQueryString();
System.out.println(queryString); //user=atguigu&password=123456&interesting=game&interesting=party&interesting=shopping
```

⑤. 获取请求的 Serlvet 的映射路径 

```
String servletPath = httpServletRequest.getServletPath();
System.out.println(servletPath);  //  /loginServlet
```

 

**3). HttpServletRequest: 是 SerlvetRequest 的子接口. 针对于 HTTP 请求所定义. 里边包含了大量获取 HTTP 请求相关的方法.** 

**4). ServletResponse: 封装了响应信息, 如果想给用户什么响应, 具体可以使用该接口的方法实现.** 

　　①. *getWriter(): 返回 PrintWriter 对象. 调用该对象的 print() 方法, 将把 print() 中的参数直接打印到客户的浏览器上. 

　　②. 设置响应的内容类型: response.setContentType("application/msword");

　　③. void sendRedirect(String location): 请求的重定向. (此方法为 HttpServletResponse 中定义.)

 



### GenericServlet

1). 是一个 Serlvet. 是 Servlet 接口和 ServletConfig 接口的实现类. 但是一个抽象类. 其中的 service 方法为抽象方法

2). 如果新建的 Servlet 程序直接继承 GenericSerlvet 会使开发更简洁.

3). 具体实现:

　　①. 在 GenericServlet 中声明了一个 SerlvetConfig 类型的成员变量, 在 init(ServletConfig) 方法中对其进行了初始化
　　②. 利用 servletConfig 成员变量的方法实现了 ServletConfig 接口的方法
　　③. 还定义了一个 init() 方法, 在 init(SerlvetConfig) 方法中对其进行调用, 子类可以直接覆盖 init() 在其中实现对 Servlet 的初始化.
　　④. 不建议直接覆盖 init(ServletConfig), 因为如果忘记编写 super.init(config); 而还是用了 SerlvetConfig 接口的方法,则会出现空指针异常.
　　⑤. 新建的 init(){} 并非 Serlvet 的生命周期方法. 而 init(ServletConfig) 是生命周期相关的方法.

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract class GenericServlet implements Servlet, ServletConfig {

    /** 以下方法为 Servlet 接口的方法 **/
    @Override
    public void destroy() {}

    @Override
    public ServletConfig getServletConfig() {
        return servletConfig;
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    private ServletConfig servletConfig;
    
    @Override
    public void init(ServletConfig arg0) throws ServletException {
        this.servletConfig = arg0;
        init();
    }

    public void init() throws ServletException{}

     /**【注意】唯独service方法没有被实现，还是一个抽象方法，这个service方法必须我们自己去重写*/
    @Override
    public abstract void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException;

    /** 以下方法为 ServletConfig 接口的方法 **/
    @Override
    public String getInitParameter(String arg0) {
        return servletConfig.getInitParameter(arg0);
    }

    @Override
    public Enumeration getInitParameterNames() {
        return servletConfig.getInitParameterNames();
    }

    @Override
    public ServletContext getServletContext() {
        return servletConfig.getServletContext();
    }

    @Override
    public String getServletName() {
        return servletConfig.getServletName();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【注意】唯独service方法没有被实现，还是一个抽象方法，这个service方法必须我们自己去重写。
```



### HttpServlet

　　1). 是一个 Servlet, 继承自 GenericServlet. 针对于 HTTP 协议所定制.

　　2). 在 service() 方法中直接把 ServletReuqest 和 ServletResponse 转为 HttpServletRequest 和 HttpServletResponse.并调用了重载的 service(HttpServletRequest, HttpServletResponse)

　　  在 service(HttpServletRequest, HttpServletResponse) 获取了请求方式: request.getMethod(). 根据请求方式有创建了doXxx() 方法(xxx 为具体的请求方式, 比如 doGet, doPost)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
 public void service(ServletRequest req, ServletResponse res)
    throws ServletException, IOException {

    HttpServletRequest  request;
    HttpServletResponse response;
    
    try {
        request = (HttpServletRequest) req;
        response = (HttpServletResponse) res;
    } catch (ClassCastException e) {
        throw new ServletException("non-HTTP request or response");
    }
    service(request, response);
}

public void service(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    //1. 获取请求方式.
    String method = request.getMethod();
    
    //2. 根据请求方式再调用对应的处理方法
    if("GET".equalsIgnoreCase(method)){
        doGet(request, response);
    }else if("POST".equalsIgnoreCase(method)){
        doPost(request, response);
    }
}

public void doPost(HttpServletRequest request, HttpServletResponse response) 
        throws ServletException, IOException{
    // TODO Auto-generated method stub
    
}

public void doGet(HttpServletRequest request, HttpServletResponse response) 
        throws ServletException, IOException {
    // TODO Auto-generated method stub
    
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

3). 实际开发中, 直接继承 HttpServlet, 并根据请求方式复写 doXxx() 方法即可.

4). 好处: 直接由针对性的覆盖 doXxx() 方法; 直接使用 HttpServletRequest 和 HttpServletResponse, 不再需要强转.

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10688617.html#_labelTop)

## Servlet的生命周期



### Servlet生命周期图

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190411123348548-937154503.png)



### Servlet生命周期简述

（1）加载和实例化

　　当Servlet容器启动或客户端发送一个请求时，Servlet容器会查找内存中是否存在该Servlet实例，若存在，则直接读取该实例响应请求；如果不存在，就创建一个Servlet实例。

（2） 初始化

　　实例化后，Servlet容器将调用Servlet的init()方法进行初始化（一些准备工作或资源预加载工作）。

（3）服务

　　初始化后，Servlet处于能响应请求的就绪状态。当接收到客户端请求时，调用service()的方法处理客户端请求，HttpServlet的service()方法会根据不同的请求 转调不同的doXxx()方法,比如 doGet, doPost。

（4）销毁

　　当Servlet容器关闭时，Servlet实例也随时销毁。其间，Servlet容器会调用Servlet 的destroy()方法去判断该Servlet是否应当被释放（或回收资源）。

 

其中实例化，初始化，销毁只会执行一次，service方法执行多次，默认情况下servlet是在第一次接受到用户请求的情况下才会实例化，可以在web.xml中的<servlet><servlet>标签内添加一个<load-on-startup>1<load-on-startup>配置，此时在启动tomcat时会创建servlet实例。

# [HttpServletRequest 接口、HttpServletResponse 接口、请求转发与重定向](https://www.cnblogs.com/java-chen-hao/p/10729903.html)



**目录**

- HttpServletRequest 接口
  - [获取请求参数](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_label0_0)
  - [在请求域中保存数据](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_label0_1)
- [HttpServletResponse 接口](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_label1)
- 请求转发与重定向
  - [请求转发：](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_label2_0)
  - [请求重定向：](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_label2_1)
  - [转发与重定向的区别：](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_label2_2)

 

**正文**

上篇文章我们讲了servlet的基本原理，这章将讲一下剩余的部分。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_labelTop)

## HttpServletRequest 接口

　　该接口是 ServletRequest 接口的子接口，封装了 HTTP 请求的相关信息，由 Servlet 容器创建其实现类对象并传入 service(ServletRequest req, ServletResponse res)方法中。我们请求的详细信息都可以通过 HttpServletRequest 接口的实现类对象获取。这个实现类对象一般都是容器创建的，我们不需要管理。

**HttpServletRequest 主要功能**



### 获取请求参数

1）什么是请求参数？

请求参数就是浏览器向服务器提交的数据

2）浏览器向服务器如何发送数据

　　a）附在 url 后面，如：http://localhost:8989/MyServlet/MyHttpServlet?userId=20

　　b）通过表单提交

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<form action="MyHttpServlet" method="post">
    你喜欢的足球队<br /><br />
    巴西<input type="checkbox" name="soccerTeam" value="Brazil" /> 
    德国<input type="checkbox" name="soccerTeam" value="German" />
    荷兰<input type="checkbox" name="soccerTeam" value="Holland" />
    <input type="submit" value="提交" />
</form>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

3）使用HttpServletRequest对象获取请求参数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
　　//一个name对应一个值
　　String userId = request.getParameter("userId"); 　　System.out.println("userId="+userId);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //一个name对应一组值
    String[] soccerTeams = request.getParameterValues("soccerTeam");
    for(int i = 0; i < soccerTeams.length; i++){
        System.out.println("team "+i+"="+soccerTeams[i]);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 在请求域中保存数据

数据保存在请求域中，可以转发到其他Servlet或者jsp页面，这些Servlet或者jsp页面就会 从请求中再取出数据

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //将数据保存到request对象的属性域中request.setAttribute("attrName", "attrValueInRequest");
    //两个Servlet要想共享request对象中的数据，必须是转发的关系
    request.getRequestDispatcher("/ReceiveServlet").forward(request, response);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //从request属性域中获取数据
    Object attribute = request.getAttribute("attrName"); 
    System.out.println("attrValue="+attribute);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_labelTop)

## **HttpServletResponse** 接口

HttpServletResponse 是 ServletResponse 接口的子接口，封装了 HTTP 响应的相关信息，由

Servlet 容器创建其实现类对象并传入 service(ServletRequest req, ServletResponse res)方法中。主要功能：

1）使用 PrintWriter 对象向浏览器输出数据

```
//通过PrintWriter对象向浏览器端发送响应信息
PrintWriter writer = res.getWriter();
writer.write("Servlet response"); writer.close();
```

2）实现请求重定向

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10729903.html#_labelTop)

## 请求转发与重定向

请求转发和重定向是 web 应用页面跳转的主要手段，应用十分广泛，所以我们一定要搞清楚他们的区别。



### 请求转发：

　　1）第一个 Servlet 接收到了浏览器端的请求，进行了一定的处理，然后没有立即对请求进行响应，而是将请求“交给下一个 Servlet”继续处理，下一个 Servlet 处理完成之后对浏览器进行了响应。在服务器内部将请求“交给”其它组件继续处理就是请求的转发。对浏览器来说，一共只发了一次请求，服务器内部进行的“转发”浏览器感觉不到，同时浏览器地址栏中的地址不会变成“下一个 Servlet”的虚拟路径。

　　2）在转发的情况下，两个 Servlet 可以共享 Request 对象中保存的数据

　　3）转发的情况下，可以访问 WEB-INF 下的资源

　　4）当需要将后台获取的数据传送到 JSP 上显示的时候，就可以先将数据存放到 Request 对象中，再转发到 JSP 从属性域中获取。此时由于是“转发”，所以它们二者共享 Request 对象中的数据。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class ForwardServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("forwardServlet  doGet");

        //请求的转发
        //1.调用HTTPServletRequest 的getRequestDispatcher（）方法获取RequestDispatcher对象
        //调用getRequestDispatcher（）方法时需要传入转发的地址
        String path = "testServlet";

        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/"+path);
        //2.调用调用HTTPServletRequest 的forward（request，response）进行请求的转发
        requestDispatcher.forward(request,response);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 请求重定向：

　　1）第一个 Servlet 接收到了浏览器端的请求，进行了一定的处理，然后给浏览器一个特殊的响应消息，这个特殊的响应消息会通知浏览器去访问另外一个资源，这个动作是服务器和浏览器自动完成的，但是在浏览器地址栏里面能够看到地址的改变，会变成下一个资源的地址。

　　2）对浏览器来说，一共发送两个请求，所以用户是能够感知到变化的。

　　3）在重定向的情况下，不能共享 Request 对象中保存的数据。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("RedirectServlet doGet");

        //执行请求的重定向,直接调用reponse.sendRedirect（path）方法
        //path为重定向的地址
        String path = "testServlet";
        response.sendRedirect(path);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 转发与重定向的区别：

|                | 转发                        | 重定向                |
| -------------- | --------------------------- | --------------------- |
| 浏览器地址栏   | 不会变化                    | 会变化                |
| Request        | 同一个请求                  | 两次请求              |
| API            | Request 对象                | Response 对象         |
| 位置           | 服务器内部完成              | 浏览器完成            |
| WEB-INF        | 可以访问                    | 不能访问              |
| 共享请求域数据 | 可以共享                    | 不可以共享            |
| 目标资源       | 必须是当前 Web 应用中的资源 | 不局限于当前 Web 应用 |

 


图解转发和重定向

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190418152756629-1036111198.png)

 [JSP总结](https://www.cnblogs.com/java-chen-hao/p/10730424.html)



**目录**

- [背景](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label0)
- [jsp 简介](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label1)
- [jsp 运行原理](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label2)
- jsp 基本语法
  - [jsp 模板元素](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label3_0)
  - [jsp 表达式](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label3_1)
  - [jsp 脚本片段](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label3_2)
-  jsp 指令
  - [ 指令简介](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label4_0)
  - [page 指令和 include 指令](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label4_1)
- jsp 标签
  - [概述](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label5_0)
  - [标签](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label5_1)
  - [标签](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label5_2)
  - [ 标签](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label5_3)
- jsp 九大隐含对象
  - [PageContext pageContext](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_0)
  - [HttpServletRequest request](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_1)
  - [HttpSession session](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_2)
  - [ServletContext application](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_3)
  - [ 四个域对象的比较](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_4)
  - [HttpServletResponse response](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_5)
  - [ServletConfig config](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_6)
  - [Throwable exception](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_7)
  - [JspWriter out](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_8)
  - [Object page](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_label6_9)

 

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

## 背景

Servlet 可以通过转发或重定向跳转到某个 HTML 文档。但 HTML 文档中的内容不受 Servlet 的控制。比如登录失败时，跳转回登录表单页面无法显示诸如“用户名或密码不正确”的错误消息，所以我们目前采用的办法是跳转到一个错误信息页面。如果通过 Servlet 逐行输出响应信息则会非常繁琐。

|      | Servlet                            | HTML               |
| ---- | ---------------------------------- | ------------------ |
| 长处 | 接收请求参数，访问域对象，转发页面 | 以友好方式显示数据 |
| 短处 | 以友好方式显示数据                 | 动态显示数据       |

Servlet 和 HTML 二者的长处结合起来就诞生了 JSP 。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

## jsp 简介

Java Server Page

　　1） JSP 的本质是一个 Servlet，Servlet 能做的事情 JSP 都能做。

　　2） JSP 能够以 HTML 页面的方式呈现数据，是一个可以嵌入 Java 代码的 HTML。

　　3） JSP 不同于 HTML，不能使用浏览器直接打开，而必须运行在 Servlet 容器中。

　　4） JSP 可以放置在 WEB 应用程序中的除了 WEB-INF 及其子目录外的其他任何目录中,JSP 页面的访问路径与普通 HTML 页面的访问路径形式也完全一样。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

## **jsp** 运行原理

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190418160521692-195073158.png)

 

每个JSP 页面在第一次被访问时, JSP 引擎将它翻译成一个 Servlet 源程序, 接着再把这个 Servlet 源程序编译成 Servlet 的class类文件.然后再由WEB容器（Servlet引擎）像调用普通Servlet程序一样的方式来装载和解释执行这个由JSP页面翻译成的Servlet程序。

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

## **jsp** 基本语法



### **jsp** 模板元素

　　1） JSP 页面中的静态 HTML 内容称之为 JSP 模版元素，在静态的 HTML 内容之中可以嵌套 JSP的其他各种元素来产生动态内容和执行业务逻辑。

　　2） JSP 模版元素定义了网页的基本骨架，即定义了页面的结构和外观



### **jsp** 表达式

　　1） JSP 表达式（expression）提供了将一个 Java 变量或表达式的计算结果输出到客户端的简化方式，它将要输出的变量或表达式直接封装在<%= 和 %>之中

　　　　举例：Current time: <%= new java.util.Date() %>

　　2） JSP 表达式中的变量或表达式的计算结果将被转换成一个字符串，然后被插入到整个 JSP页面输出结果的相应位置处。

　　3） JSP 表达式中的变量或表达式后面不能有分号（;），JSP 表达式被翻译成 Servlet 程序中的一条 out.print(…)语句。



### **jsp** 脚本片段

　　1）JSP 脚本片断（scriptlet）是指嵌套在<% 和 %>之中的一条或多条 Java 程序代码。

　　2）在 JSP 脚本片断中，可以定义变量、执行基本的程序运算、调用其他 Java 类、访问数据库、访问文件系统等普通 Java 程序所能实现的功能。

　　3）在 JSP 脚本片断可以直接使用 JSP 提供的隐含对象来完成 WEB 应用程序特有的功能。

　　4） JSP 脚本片断中的 Java 代码将被原封不动地搬移进由 JSP 页面所翻译成的 Servlet 的 _jspService()方法中。所以，JSP 脚本片断之中只能是符合 Java 语法要求的程序代码，除此之外的任何文本、HTML 标记、其他 JSP 元素都必须在脚本片断之外编写。

　　5） JSP 脚本片断中的 Java 代码必须严格遵循 Java 语法，例如，每条命令执行语句后面必须用分号（;）结束。

　　6）在一个 JSP 页面中可以有多个脚本片断（每个脚本片断代码嵌套在各自独立的一对<% 和 %>之间），在两个或多个脚本片断之间可以嵌入文本、HTML 标记和其他 JSP 元素。

举例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<%
　　int x = 3;
%>
<p>这是一个 HTML 段落</p>
<%
　　out.println(x);
%>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

7）多个脚本片断中的代码可以相互访问，犹如将所有的代码放在一对<%%>之中的情况。 举例：上面的 JSP 内容与下面的 JSP 内容具有同样的运行效果

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<p>这是一个 HTML 段落</p>
<%
    int x = 3; 
    out.println(x);
%>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

举例 1：

| JSP 代码                                                     | _jspService()方法中的代码                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <%**for** (**int** i=1; i<5; i++) {%><h<%=i%>>[www.atguigu.com](http://www.atguigu.com/)</h<%=i%>><%}%> | for (int i=1; i<5; i++) {out.write("\r\n"); out.write("\t\t<h"); out.print(i);out.write(">[www.atguigu.com](http://www.atguigu.com/)</h"); out.print(i);out.write(">\r\n");out.write("\t");} |

举例 2：

| JSP 代码                                                   | _jspService()方法中的代码                                    |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| <% **if**(age>65){ %>可以退休<%}**else**{ %>不能退休<%} %> | if(age>65){out.write("\r\n");out.write("\t\t 可以退休\r\n"); out.write("\t");}else{out.write("\r\n"); out.write("\t\t 不能退休\r\n");out.write("\t");} |

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

##  **jsp** 指令



###  指令简介

　　1） JSP 指令（directive）是为 JSP 引擎而设计的，它们并不直接产生任何可见输出，而只是告诉引擎如何处理 JSP 页面中的其余部分。

　　2） JSP 指令的基本语法格式：

　　<%@ 指令名 属性名="值" %>

　　举例：<%@ page contentType="text/html;charset=gb2312"%>

　　注意：属性名部分是大小写敏感的

　　3）在目前的 JSP2.0 中，定义了 page、include 和 taglib 这三种指令，每种指令中又都定义了一些各自的属性。如果要在一个 JSP 页面中设置同一条指令的多个属性，可以使用多条指令语句单独设置每个属性，也可以使用同一条指令语句设置该指令的多个属性。

　　第一种方式：

　　<%@ page contentType="text/html;charset=gb2312"%>

　　<%@ page import="java.util.Date"%>

第二种方式：

<%@ page contentType="text/html;charset=gb2312" import="java.util.Date"%>



### **page** 指令和 **include** 指令

　　**1） page 指令**

　　page 指令用于定义 JSP 页面的各种属性，无论 page 指令出现在 JSP 页面中的什么地方，它作用的都是整个 JSP 页面。为了保持程序的可读性和遵循良好的编程习惯， **page** 指令最好是放在整个 **JSP** 页面的起始位置

　　

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<%@ page
    [ language="java" ]
    [ extends="package.class" ]
    [ import="{package.class | package.*}, ..." ] [ session="true | false" ]
    [ buffer="none | 8kb | sizekb" ] [ autoFlush="true | false" ]
    [ isThreadSafe="true | false" ] [ info="text" ]
    [ errorPage="relative_url" ] [ isErrorPage="true | false" ]
    [contentType="mimeType [ ;charset=characterSet ]" | "text/html ; charset=ISO-8859-1" ] [ pageEncoding="characterSet | ISO-8859-1" ]
    [ isELIgnored="true | false" ]
%>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[1]import 属性：指定 JSP 页面转换成 Servlet 时应该导入的包。
[2]pageEncoding 属性：设置 JSP 页面翻译成 Servlet 源文件时使用的字符集。
[3]contentType 属性：设置 Content-Type 响应报头，标明即将发送到客户程序的文档的 MIME 类型以及浏览器对响应内容的解码字符集。
[4]errorPage 属性：指定当前 JSP 抛出异常时的转发页面。
[5]isErrorPage 属性：指定当前页面是不是一个显示错误消息的页面，如果是，则会自动创建 exception 对象，否则就不会创建 exception 对象。
[6]session 属性：控制页面是否参与 HTTP 会话，其本质是要不要自动创建session 隐含对象以供使用。
[7]isELIgnored 属性：指定当前页面是否忽略 EL 表达式，如果忽略，EL 表达式的内容将会原封不动的输出到浏览器端。

 

**errorPage 和 isErrorPage:** 

errorPage 指定若当前页面出现错误的实际响应页面时什么. 其中 / 表示的是当前 WEB 应用的根目录.
<%@ page errorPage="/error.jsp" %>

在响应 error.jsp 时, JSP 引擎使用的请求转发的方式.

isErrorPage 指定当前页面是否为错误处理页面, 可以说明当前页面是否可以使用 exception 隐藏变量. 需要注意的是: 若指定
isErrorPage="true", 并使用 exception 的方法了, 一般不建议能够直接访问该页面.

如何使客户不能直接访问某一个页面呢 ? 对于 Tomcat 服务器而言, WEB-INF 下的文件是不能通过在浏览器中直接输入地址的方式
来访问的. 但通过请求的转发是可以的!

还可以在 web.xml 文件中配置错误页面:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<error-page>
<!-- 指定出错的代码: 404 没有指定的资源, 500 内部错误. -->
    <error-code>404</error-code>
    <!-- 指定响应页面的位置 -->
    <location>/WEB-INF/error.jsp</location>
</error-page>
  
<error-page>
    <!-- 指定异常的类型 -->
    <exception-type>java.lang.ArithmeticException</exception-type>
    <location>/WEB-INF/error.jsp</location>
</error-page>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**2） include 指令**

**include** 指令用于通知 **JSP** 引擎在翻译当前 **JSP** 页面时将其他文件中的内容合并进当前 **JSP** 页面转换成的 **Servlet** 源文件中，这种在源文件级别进行引入的方式称之为静态引入，当前 **JSP** 页面与静态引入的页面紧密结合为一个 **Servlet**。

语法：

<%@ include file="relativeURL"%>

其中的 file 属性用于指定被引入文件的相对路径。

 

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

## **jsp** 标签



### 概述

　　1） JSP 还提供了一种称之为 Action 的元素，在 JSP 页面中使用 Action 元素可以完成各种通用的 JSP 页面功能，也可以实现一些处理复杂业务逻辑的专用功能。

　　2） Action 元素采用 XML 元素的语法格式，即每个 Action 元素在 JSP 页面中都以 XML标签的形式出现。

　　3） JSP 规范中定义了一些标准的 Action 元素，这些元素的标签名都以 jsp 作为前缀，并且全部采用小写，例如，<jsp:include>、<jsp:forward>等等。



### **<jsp:include>**标签

　　1）<jsp:include>标签用于把另外一个资源的输出内容插入进当前 JSP 页面的输出内容之中，这种在 JSP 页面执行时的引入方式称之为动态引入。

　　2）语法：

　　　　<jsp:include page="relativeURL | <%=expression%>" flush="true|false" />

　　3）<jsp:include>标签与 include 指令的比较

　　　　○ @include 指令在翻译“主体”代码时起作用。相当于把“被包含”的 JSP 代码复制到“主体”JSP 文件中，生成一个合并之后的 Servlet 源文件，所以二者在代码中不能使用相同的变量名等——凡是放在一起会冲突的内容都不被允许。

　　　　○ <jsp:include>标签会被翻译为 JspRuntimeLibrary 类的 include()方法，“被包含”的 JSP 页面会单独翻译、编译；每次“主体”JSP 被请求时，都会先执行“被包含”的JSP，将执行结果合并到 HTML 代码中，一起发送到浏览器端。所以使用<jsp:include>标签时，“被包含”的 JSP 中的代码不会和“主体”JSP 冲突。

　　　　○ 对比

|                                          | @include 指令          | <jsp:include>标签       |
| ---------------------------------------- | ---------------------- | ----------------------- |
| 特点                                     | 静态包含               | 动态包含                |
| 语法的基本形式                           | <%@ include file=”…”%> | <jsp:include page=”…”/> |
| 包含动作发生的时机                       | 翻译期间               | 请求期间                |
| 生成 Servlet 源文件                      | 一个                   | 多个                    |
| 合并方式                                 | 代码复制               | 合并运行结果            |
| 包含的内容                               | 文件实际内容           | 页面输出结果            |
| 代码冲突                                 | 有可能                 | 不可能                  |
| 被包含页面中可否设置影响主页面的响应报头 | 可以                   | 不可以                  |



### **<jsp:forward>**标签

　　1）<jsp:forward>标签用于把请求转发给另外一个资源

　　2）语法：

　　　　<jsp:forward page="relativeURL|<%=expression%>" />

　　　　page  属性用于指定请求转发到的资源的相对路径，它也可以通过执行一个表达式来获得。



###  **<jsp:param>**标签

　　1）当使用<jsp:include>和<jsp:forward>标签引入或将请求转发给的资源是一个能动态执 行的程序时，例如 Servlet 和 JSP 页面，那么，还可以使用<jsp:param>标签向这个程序传递请求参数。

　　语法 1：

```
<jsp:include page="relativeURL | <%=expression%>">
　　<jsp:param name="parameterName" value="parameterValue|<%= expression %>" />
</jsp:include>
```

　　语法 2：

```
<jsp:forward page="relativeURL | <%=expression%>">
    <jsp:param name="parameterName" value="parameterValue|<%= expression %>" />
</jsp:include>
```

　　2）<jsp:param>标签的name属性用于指定参数名，value属性用于指定参数值。在<jsp:include>和<jsp:forward>标签中可以使用多个<jsp:param>标签来传递多个参数。

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10730424.html#_labelTop)

## jsp 九大隐含对象

在 JSP 页面上编写 Java 代码时，有九个可以直接使用的内置对象。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
PageContext pageContext 
HttpServletRequest request 
HttpSession session 
ServletContext application 
HttpServletResponse response 
ServletConfig config 
Throwable exception 
JspWriter out
Object page
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

为什么可以在页面使用它们，因为我们发现，页面是在service方法中进行解析的。而service方法在解析页面之前申明了。在页面设置为 isErrorPage=”true”的时候，exception对象就会显示

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190418170406526-402217535.png)



### PageContext pageContext

**页面的上下文, 是 PageContext 的一个对象.** 可以从该对象中获取到其他 8 个隐含对象. 也可以从中获取到当前页面的其他信息.

1） 获取其它隐含对象

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
getException 方法返回 exception 隐式对象
getPage 方法返回 page 隐式对象
getRequest 方法返回 request 隐式对象
getResponse 方法返回 response 隐式对象
getServletConfig 方法返回 config 隐式对象
getServletContext 方法返回 application 隐式对象
getSession 方法返回 session 隐式对象
getOut 方法返回 out 隐式对象
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1） 作为域对象

可以设置、获取属性值

```
public void setAttribute(java.lang.String name,java.lang.Object value) 
public java.lang.Object getAttribute(java.lang.String name)
public void removeAttribute(java.lang.String name)
```

3） 访问其它属性域

```
public java.lang.Object getAttribute(java.lang.String name,int scope)
public void setAttribute(java.lang.String name, java.lang.Object value,int scope)
public void removeAttribute(java.lang.String name,int scope)
```

**属性的作用范围仅限于当前 JSP 页面**



### HttpServletRequest request

域对象，可以存取属性值，用来在域中共享。**HttpServletRequest 的一个对象。**

```
public void setAttribute(java.lang.String name,java.lang.Object value) public java.lang.Object getAttribute(java.lang.String name)
public void removeAttribute(java.lang.String name)
```

**属性的作用范围仅限于同一个请求.** 



### HttpSession session

域对象，可以存取属性值，用来在域中共享。

**代表浏览器和服务器的一次会话, 是 HttpSession 的一个对象.**

属性的作用范围限于一次会话: 浏览器打开直到关闭称之为一次会话(在此期间会话不失效)



### ServletContext application

域对象，可以存取属性值，用来在域中共享。

**代表当前 WEB 应用. 是 ServletContext 对象.**

**属性的作用范围限于当前 WEB 应用. 是范围最大的属性作用范围, 只要在一处设置属性, 在其他各处的 JSP 或 Servlet 中都可以获取到.** 

 

pageContext, request, session, application 这四个对象也称之为域对象.



###  四个域对象的比较

| 域对象      | 作用范围      | 起始时间     | 结束时间     |
| ----------- | ------------- | ------------ | ------------ |
| pageContext | 当前 JSP 页面 | 页面加载     | 离开页面     |
| request     | 同一个请求    | 收到请求     | 响应         |
| session     | 同一个会话    | 开始会话     | 结束会话     |
| application | 当前 Web 应用 | Web 应用加载 | Web 应用卸载 |

pageContext, request, session, application(对属性的作用域的范围从小到大)



### HttpServletResponse response

HttpServletResponse 的一个对象(在 JSP 页面中几乎不会调用 response 的任何方法.)



### ServletConfig config

当前 JSP 对应的 Servlet 的 ServletConfig 对象(几乎不使用). 



### **Throwable** **exception**

exception 对象：封装了当前 JSP 页面捕获到的异常信息

**在声明了 page 指令的 isErrorPage="true" 时, 才可以使用.** 



### **JspWriter out**

JspWriter 对象. 调用 out.println() 可以直接把字符串打印到浏览器上.



### Object page

指向当前 JSP 对应的 Servlet 对象的引用, 但为 Object 类型, 只能调用 Object 类的方法(几乎不使用) 

# [一文带你超详细了解Cookie](https://www.cnblogs.com/java-chen-hao/p/10774541.html)



**目录**

- cookie 简介
  - [什么是 cookie](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_label0_0)
  - [庐山真面目](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_label0_1)
  - [cookie 原理](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_label0_2)
- Cookie 的使用
  - [创建对象](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_label1_0)
  - [设置 cookie](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_label1_1)
  - [读取 cookie](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_label1_2)

 

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_labelTop)

## cookie 简介



### 什么是 **cookie**

cookie，有时我们也用其复数形式 cookies，是服务端保存在浏览器端的数据片段。以 key/value的形式进行保存。每次请求的时候，请求头会自动包含本网站此目录下的 cookie 数据。网站经常使用这个技术来识别用户是否登陆等功能。

简单的说，cookie 就是服务端留给计算机用户浏览器端的小文件。

- HTTP  是无状态协议，服务器不能记录浏览器的访问状态，也就是说服务器不能区分中两次请求是否由一个客户端发出。这样的设计严重阻碍的 Web 程序的设计。如：在我们进行网购时，买了一条裤子，又买了一个手机。由于 http 协议是无状态的，如果不通过其他手段，服务器是不能知道用户到底买了什么。而 Cookie 就是解决方案之一。
- Cookie 实际上就是服务器保存在浏览器上的一段信息。浏览器有了 Cookie 之后，每次向服务器发送请求时都会同时将该信息发送给服务器，服务器收到请求后，就可以根据  该信息处理请求。
- 例如：我们上文说的网上商城，当用户向购物车中添加一个商品时，服务器会将这个条信息封装成一个 Cookie 发送给浏览器，浏览器收到 Cookie，会将它保存在内存中(注意这里的内存是本机内存，而不是服务器内存)，那之后每次向服务器发送请求，浏览器都会携带该 Cookie，而服务器就可以通过读取 Cookie 来判断用户到底买了哪些商品。当用户进行结账操作时，服务器就可以根据 Cookie 的信息来做结算。

- Cookie 的用途： 网上商城的购物车 保持用户登录状态

- Cookie 的缺点

　　　　　Cookie 做为请求或响应报文发送，无形中增加了网络流量。

　　　　　Cookie 是明文传送的安全性差。

　　　　　Cookie 中保存数据是不稳定的，用户可以随时清理 cookie,各个浏览器对 Cookie 有限制，使用上有局限



### 庐山真面目

chrome 的 cookie 位置：

C:\Users\lfy\AppData\Local\Google\Chrome\User Data\Default\Cookies

ie 中 cookie 位 置 ： C:\Users\lfy\AppData\Local\Microsoft\Windows\InetCache 点击设置->查看对象即可

chrome 中查看 cookie

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190426151300997-1501762655.png)

cookie 如上图所示

从上图可以看出 cookie 是键值对的形式，有过期时间（Max-Age，session 表示在这个会话期内有效）。



### **cookie** 原理

1）总的来看 Cookie 像是服务器发给浏览器的一张“会员卡”，浏览器每次向服务器发送请求时都会带着这张“会员卡”，当服务器看到这张“会员卡”时就可以识别浏览器的身份。实际上这个所谓的“会员卡”就是服务器发送的一个响应头：

 ![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190426151455827-928438614.png)

2）如图 Set-Cookie 这个响应头就是服务器在向服务器发“会员卡”，这个响应头的名字是 Set-Cookie ， 后 边 JSESSIONID=95A92EC1D7CCB4ADFC24584CB316382E 和 Path=/Test_cookie，是两组键值对的结构就是服务器为这个“会员卡”设置的信息。浏览器收到该信息后就会将它保存到内存或硬盘中。

3）当浏览器再次向服务器发送请求时就会携带这个 Cookie 信息：

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190426151537849-1237729577.png)

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190426151556496-1144075767.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10774541.html#_labelTop)

## **Cookie** 的使用



### 创建对象

cookie 是由服务端创建的，由浏览器端保存的。所以创建对象我们应该在服务端创建 cookie,cookie 的创建方法：

1）创建一个 CookieServlet

在 Servlet 的 doPost()方法中编写如下代码：

```
//创建一个Cookie对象
Cookie cookie = new Cookie("username", "zhangsan");
//将Cookie对象放入response对象中response.addCookie(cookie);
```

 

2）在浏览器中访问该 Servlet，会发现响应头中出现如下内容： Set-Cookie: username=zhangsan

如此就成功的向浏览器设置了一个 Cookie，当我们在刷新页面时会发现浏览器的请求头中出现如下代码：

Cookie: username=zhangsan

3）同样我们还可以同时设置多个 Cookie：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//创建一个Cookie对象
Cookie cookie1 = new Cookie("username", "zhangsan"); 
Cookie cookie2 = new Cookie("password", "123456"); 
Cookie cookie3 = new Cookie("age", "20");
//将Cookie对象放入response对象中
response.addCookie(cookie1);
response.addCookie(cookie2);
response.addCookie(cookie3);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

浏览器会按以下形式发送 Cookie：

Cookie: username=zhangsan; password=123456; age=20

4）设置 Cookie 就是两个步骤： 创建 Cookie 对象

将 Cookie 对象加入到 response 中



### 设置 **cookie**

#### **cookie** 的有效时间

1）经过上边的介绍我们已经知道 Cookie  是存储在浏览器中的，但是可想而知一般情况下浏览器不可能永远保存一个 Cookie，一来是占用硬盘空间，再来一个 Cookie 可能只在某一时刻有用没必要长久保存。

2） 所以我们还需要为 Cookie 设置一个有效时间。

3）通过 Cookie 对象的 setMaxAge()可以设置 Cookie 的有效时间。

其中 setMaxAge()接收一个 int 型的参数，来设置有效时间。参数主要有一下四种情况：

- 设置为 0，setMaxAge(0)

　　　　Cookie 立即失效，下次浏览器发送请求将不会在携带该 Cookie

- 设置大于 0，setMaxAge(60)

　　　　表示有效的秒数 60 就代表 60 秒即 1 分钟，也就是 Cookie 在 1 分钟后失效。

- 设置小于 0，setMaxAge(-1)

　　　　设置为负数表示当前会话有效。也就是关闭浏览器后 Cookie 失效

- 不设置

　　　　如果不设置失效时间，则默认当前会话有效。

#### **cookie** 的路径

1） Cookie 的路径指告诉浏览器访问那些地址时该携带该 Cookie，我们知道浏览器会保存很多不同网站的 Cookie，比如百度的 Cookie，新浪的 Cookie，腾讯的 Cookie 等等。那我们不可能访问百度的时候携带新浪的 Cookie，也不可能访问每个网站时都带上所有的 Cookie 这是不现实的，所以往往我们还需要为 Cookie 设置一个 Path 属性，来告诉浏览器何时携带该 Cookie。

2）我们同过 Cookie 的 setPath()来设置路径，这个路径是由浏览器来解析的所以/代表服务器的根目录。

如：设置为 /项目名/路径 cookie.setPath(“/项目名/路径”),这样设置只有访问“/项目名/路径”下的的资源才会携带 Cookie

如：/项目名/路径/1.jsp 、/项目名/路径/hello/2.jsp 等

如果不设置，默认会在访问“/项目名”下的资源时携带如：“/项目名/index.jsp” 、 “/项目名/hello/index.jsp”

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Cookie cookie = new Cookie("username", "abc"); cookie.setMaxAge(60*60*24);//秒为单位,一天后过期
cookie.setPath(getServletContext().getContextPath()+"/"); resp.addCookie(cookie); resp.sendRedirect(getServletContext().getContextPath()+"/index.jsp");
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 读取 **cookie**

通过以上步骤，我们将 cookie 保存到了浏览器端。那么我们如何读取 cookie 中的值呢。分析：

cookie 被设置进入浏览器后，每次请求都会携带 cookie 的值，所以我们需要从 request 中取出 cookie 进行解析。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//从request中获取所有cookie
Cookie[] cookies = request.getCookies();
//遍历cookie
for(Cookie c:cookies){
　　String cName = c.getName();//获取cookie名String cValue = c.getValue();//获取cookie值
　　System.out.println("cookie：" + cName + "=" +cValue);
}
```

# [干货，一文带你超详细了解Session的原理及应用](https://www.cnblogs.com/java-chen-hao/p/10774858.html)



**目录**

- [session 简介](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label0)
- HttpSession 的生命周期
  - [什么时候创建 HttpSession 对象](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label1_0)
  - [page 指令的 session=“false“ 表示什么意思？](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label1_1)
  - [在 Serlvet 中如何获取 HttpSession 对象？](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label1_2)
  - [什么时候销毁 HttpSession 对象](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label1_3)
- session 使用
  - [Session 时 效](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label2_0)
  - [URL 重写](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label2_1)
  - [Session 的活化和钝化](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label2_2)
- 表单重复提交问题
  - [什么是表单重复提交？](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label3_0)
  - [几种重复提交](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_label3_1)

 

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_labelTop)

## **session** 简介

session 是我们 jsp 九大隐含对象的一个对象。

session 称作域对象，他的作用是保存一些信息，而 session 这个域对象是一次会话期间使用同一个对象。所以这个对象可以用来保存共享数据。

- 使用 Cookie 有一个非常大的局限，就是如果 Cookie 很多，则无形的增加了客户端与服务端的数据传输量。而且由于浏览器对 Cookie 数量的限制，注定我们不能再 Cookie 中保存过多的信息，于是 Session 出现。
- Session 的作用就是在服务器端保存一些用户的数据，然后传递给用户一个名字为JSESSIONID 的 Cookie，这个 JESSIONID 对应这个服务器中的一个 Session 对象，通过它就可以获取到保存用户信息的 Session。

session 是基于 cookie 的。

在用户第一次使用 session 的时候（访问 jsp 页面会获取 session，所以一般访问 index.jsp 就算是第一次使用 session 了），服务器会为用户创建一个 session 域对象。使用 jsessionid 和这个对象关联，这个对象在整个用户会话期间使用。响应体增加 set-cookie:jsessionid=xxx 的项。用户下次以后的请求都会携带 jsessionid 这个参数，我们使用 request.getSession()的时候，就会使用 jsessionid 取出 session 对象。

session 原理图：

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190426160703288-530962273.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_labelTop)

## **HttpSession 的生命周期**



### 什么时候创建 HttpSession 对象

①. 对于 JSP: 是否浏览器访问服务端的任何一个 JSP, 服务器都会立即创建一个 HttpSession 对象呢？
不一定。

- 若当前的 JSP 是客户端访问的当前 WEB 应用的第一个资源，且 JSP 的 page 指定的 session 属性值为 false,则服务器就不会为 JSP 创建一个 HttpSession 对象;
- 若当前 JSP 不是客户端访问的当前 WEB 应用的第一个资源，且其他页面已经创建一个 HttpSession 对象，则服务器也不会为当前 JSP 页面创建一个 HttpSession 对象，而会把和当前会话关联的那个 HttpSession 对象返回给当前的 JSP 页面.

②. 对于 Serlvet: 若 Serlvet 是客户端访问的第一个 WEB 应用的资源,则只有调用了 request.getSession() 或 request.getSession(true) 才会创建 HttpSession 对象



### page 指令的 session=“false“ 表示什么意思？

当前 JSP 页面禁用 session 隐含变量！但可以使用其他的显式的 HttpSession 对象



### 在 Serlvet 中如何获取 HttpSession 对象？

```
request.getSession(boolean create): 
```

create 为 false, 若没有和当前 JSP 页面关联的 HttpSession 对象, 则返回 null; 若有, 则返回 true
create 为 true, 一定返回一个 HttpSession 对象. 若没有和当前 JSP 页面关联的 HttpSession 对象, 则服务器创建一个新的HttpSession 对象返回, 若有, 直接返回关联的. 
request.getSession(): 等同于 request.getSession(true)



### 什么时候销毁 HttpSession 对象

①. 直接调用 HttpSession 的 invalidate() 方法: 该方法使 HttpSession 失效

②. 服务器卸载了当前 WEB 应用.

③. 超出 HttpSession 的过期时间.

④. 并不是关闭了浏览器就销毁了 HttpSession.

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_labelTop)

## **session** 使用

获取 session 对象

```
HttpSession session = request.getSession();
```

session 是我们的四大域对象之一。用来保存数据。常用的方法

```
session.setAttribute("user", new Object()); session.getAttribute("user");
session.setMaxInactiveInterval(60*60*24);//秒为单位
session.invalidate();//使 session 不可用
```



### **Session** 时 效

①、基本原则

Session 对象在服务器端不能长期保存，它是有时间限制的，超过一定时间没有被访问过的 Session 对象就应该释放掉，以节约内存。所以 Session 的有效时间并不是从创建对象开始计时，到指定时间后释放——而是从最后一次被访问开始计时，统计其“空闲” 的时间。

②、默认设置

在全局 web.xml 中能够找到如下配置：

```
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

③、手工设置

```
session.setMaxInactiveInterval(int seconds) 
session.getMaxInactiveInterval()
```

④、强制失效 

```
session.invalidate()
```

⑤、可以使 Session 对象释放的情况

Session 对象空闲时间达到了目标设置的最大值，自动释放

Session 对象被强制失效

Web 应用卸载服务器进程停止



### **URL** 重写

在整个会话控制技术体系中，保持 JSESSIONID 的值主要通过 Cookie 实现。但 Cookie 在浏览器端可能会被禁用，所以我们还需要一些备用的技术手段，例如：URL 重写。

1）URL 重写其实就是将 JSESSIONID 的值以固定格式附着在 URL 地址后面，以实现保持

JSESSIONID，进而保持会话状态。这个固定格式是：URL;jsessionid=xxxxxxxxx

例如：

```
targetServlet;jsessionid=F9C893D3E77E3E8329FF6BD9B7A09957
```

2） 实 现 方 式 ：

```
response.encodeURL(String)
response.encodeRedirectURL(String)
```

例如：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//1.获取Session对象
HttpSession session = request.getSession();

//2.创建目标URL地址字符串
String url = "targetServlet";

//3.在目标URL地址字符串后面附加JSESSIONID的值
url = response.encodeURL(url);

//4.重定向到目标资源
response.sendRedirect(url);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### **Session** 的活化和钝化

Session 机制很好的解决了 Cookie 的不足，但是当访问应用的用户很多时，服务器上就会创建非常多的 Session 对象，如果不对这些 Session 对象进行处理，那么在 Session 失效之前，这些 Session 一直都会在服务器的内存中存在。那么就，就出现了 Session 活化和钝化的机制。

1）Session 钝化：

Session 在一段时间内没有被使用时，会将当前存在的 Session 对象序列化到磁盘上，而不 再 占 用 内 存 空 间 。                    

2）Session 活化：

Session 被钝化后，服务器再次调用 Session 对象时，将 Session 对象由磁盘中加载到内存中使用。

如果希望 Session 域中的对象也能够随 Session 钝化过程一起序列化到磁盘上，则对象的实现类也必须实现 java.io.Serializable 接口。不仅如此，如果对象中还包含其他对象的引用，则被关联的对象也必须支持序列化，否则会抛出异常：java.io.NotSerializableException

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10774858.html#_labelTop)

## 表单重复提交问题



### 什么是表单重复提交？

同一个表单中的数据内容多次提交到服务器。 危害：

服务器重复处理信息，负担加重。

如果是保存数据可能导致保存多份相同数据。



### 几种重复提交

1）提交完表单后，直接刷新页面，会再次提交。

\- 根本原因：Servlet 处理完请求以后，直接转发到目标页面。

\- 这样整一个业务，只发送了一次请求，那么当你在浏览器中点击刷新按钮或者狂按 f5，会一直都会刷新之前的请求

解决方案：使用重定向跳转到目标页面

2）提交表单后，由于网速差等原因，服务器还未返回结果，连续点击提交按钮，会重 复提交。

\- 根本原因：按钮可以多次点击

\- 解决方案：通过 js，使得按钮只能提交一次。

```
$(“#form1”).submit(function(){
    $(“#sub_btn”).prop(“disabled”,true);
})
```

3）表单提交后，点击浏览器回退按钮，不刷新页面，点击提交按钮再次提交表单

\- 根本原因：服务器并不能识别请求是否重复。

\- 解决方案：使用 token 机制。

1、页面生成时，产生一个唯一的 token 值。将此值放入 session

2、表单提交时，带上这个 token 值。

3、服务端验证 token 值存在，则提交表单，然后移除此值。验证 token 不存在，说明是之前验证过一次被移除了，所以是重复请求。不予处理

原理：

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190426161940597-621205882.png)

代码：

jsp 页面

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<%
　　String token = System.currentTimeMillis() + ""; 
　　request.getSession().setAttribute(token, "");
%>
<div>
　　<h1>测试表单重复提交</h1>
　　<form action="login" method="get">
　　　　用户名：<input name="username" type="text"/>
　　　　密码：<input name="password" type="password">
　　　　<input name="token" value="<%=token%>">
　　　　<input type="submit">
　　</form>
　　<hr>
</div>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Servlet

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    HttpSession session = request.getSession(); 
    String token = request.getParameter("token"); 
    Object attribute = session.getAttribute(token);
    response.setContentType("text/html;charset=UTF-8");
    if(attribute!=null){ 
        session.removeAttribute(token);
        response.getWriter().write("请求成功！");
    }else{
        response.getWriter().write("请不要重复请求！");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 其实防止重复提交的核心就是让服务器有一个字段能来识别此次请求是否已经执行。 这个字段需要页面传递过来，因为只要回退回去的页面，字段都是一致的。不会变化， 通过这个特性我们想到了 token 机制来防止重复提交

# [干货，一文带你超详细了解 Filter 的原理及应用](https://www.cnblogs.com/java-chen-hao/p/10785405.html)



**目录**

- [提出问题](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label0)
- Filter 简介
  - [什么是 filter](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label1_0)
  - [filter 的运行原理是什么](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label1_1)
- Filter-helloword
  - [filter 的生命周期](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label2_0)
  - [filter 放行请求](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label2_1)
  - [filter 拦截原理](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label2_2)
- [FilterChain](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label3)
- [FilterConfig](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label4)
- [Filter 的 url-pattern](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label5)
- [多 Filter 执行顺序](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label6)
- HttpServletWrapper 和 HttpServletResponseWrapper
  - [定义](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label7_0)
  - [作用](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label7_1)
  - [使用](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_label7_2)

 

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## 提出问题

1、我们在访问后台很多页面时都需要登录，只有登录的用户才能查看这些页面，我们需要  在每次请求的时候都检查用户是否登陆，这样做很麻烦，有没有一种方法可以在我们请求之  前就帮我们做这些事情。有！

2、我们 web 应用经常会接收中文字符，由于可能导致中文乱码，我们每次都需要在方法的开始使用 request.setCharacterEncoding(“utf-8”)；能不能在我们要获取参数值直接就可以自己设置好编码呀。能！

这种问题的解决方法我们想到了一种办法。那就是在每次请求之前我们先将它拦截起来，当  我们设置好一切东西的时候，再将请求放行。类似与我们地铁站的检票系统。每个人进站的  时候必须刷卡，扣完钱后才可以进站坐车。

web 中也有这个机制，我们叫做过滤器。就是我们接下来学习的 filter

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## **Filter** 简介



### 什么是 **filter**

1） Filter（过滤器） 的基本功能是对 Servlet 容器调用 Servlet (JSP)的过程进行拦截， 从而在 Servlet 处理请求前和 Servlet 响应请求后实现一些特殊的功能。

2） 在 Servlet API 中定义了三个接口类来开供开发人员编写 Filter 程序： Filter,FilterChain, FilterConfig

3） Filter 程序是一个实现了 Filter 接口的 Java 类，与 Servlet 程序相似，它由 Servlet容器进行调用和执行

4） Filter 程序需要在 web.xml 文件中进行注册和设置它所能拦截的资源：Filter 程序可以拦截 Jsp, Servlet, 静态图片文件和静态 html 文件



### **filter** 的运行原理是什么

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190428172348894-460305252.png)

这个 Servlet 过滤器就是我们的 filter

1）当在 web.xml 中注册了一个 Filter 来对某个 Servlet 程序进行拦截处理时，这个Filter 就成了 Tomcat 与该 Servlet 程序的通信线路上的一道关卡，该 Filter 可以对Servlet 容器发送给 Servlet 程序的请求和 Servlet 程序回送给 Servlet 容器的响应进行拦截，可以决定是否将请求继续传递给 Servlet 程序，以及对请求和相应信息是否进行修改

2）在一个 web 应用程序中可以注册多个 Filter 程序，每个 Filter 程序都可以对一个或一组 Servlet 程序进行拦截。

3）若有多个 Filter 程序对某个 Servlet 程序的访问过程进行拦截，当针对该 Servlet 的访问请求到达时，web 容器将把这多个 Filter 程序组合成一个 Filter 链(过滤器链)。Filter 链中各个 Filter 的拦截顺序与它们在应用程序的 web.xml 中映射的顺序一致

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## Filter-helloword

**Hello-World**

filter 编写三步骤：

1、创建 filter 实现类，实现 filter 接口

2、编写 web.xml 配置文件，配置 filter 的信息

3、运行项目，可以看到 filter 起作用了

代码：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//1、filter 实现类
public class MyFirstFilter implements Filter{ 

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("初始化方法");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,FilterChain chain) throws
    IOException, ServletException {
        System.out.println("dofilter方法");
    }

    @Override
    public void destroy() {
        System.out.println("销毁方法...");
    }

}

//2、web.xml 配置
<filter>
    <filter-name>MyFirstFilter</filter-name>
    <filter-class>com.atguigu.filter.MyFirstFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>MyFirstFilter</filter-name>
    <url-pattern>/index.jsp</url-pattern>
</filter-mapping>

//3、运行程序，发现 index.jsp 页面不显示了，后台输出“dofilter 方法”，说明我们写的 filter 执行了。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### **filter** 的生命周期

1）在服务器启动时，filter 被创建并初始化，执行 init()方法。

2）请求通过 filter 时执行 doFilter 方法。

3）服务器停止时，调用 destroy 方法。



### **filter** 放行请求

我们发现，刚才的 filter 配置好后，index.jsp 页面没法访问了，访问这个页面的时候 filter的 dofilter 方法被调用了。说明 dofilter 这个方法拦截了我们的请求。

我们如何显示页面呢。也就是如何将请求放行呢。我们观察发现有个 filterChain 被传入到这个方法里面了。filterChain 里面有个 doFilter()方法。放行请求只需要调用 filterChain 的 dofilter 方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException,ServletException {
    System.out.println("dofilter方法");
    chain.doFilter(request, response);//放行请求
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### **filter** 拦截原理

我们在 chain.doFilter(request, response);方法后也写一句话，System.out.println

(“doFilter 方法执行后…”)，在 index.jsp 页面也写上 jsp 脚本片段，输出我是 jsp 页面。运行程序发现控制台输出了这几句话：

```
dofilter 方法… 我是 jsp 页面
dofilter 方法后…
我们不难发现 filter 的运行流程
```

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190428173119102-1765956214.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## FilterChain

#### doFilter(ServletRequest request, ServletResponse response, FilterChain chain)

在 doFilter 执行之前，由容器将 filterChain 对象传入方法。调用此对象的.doFilter()方法可以将请求放行，实际上是执行过滤器链中的下一个 doFilter 方法，但是如果只有一个过滤器，则为放行。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## **FilterConfig**

FilterConfig 类似 ServletConfig，是 filter 的配置信息对象。FilterConfig 对象具有以下方法。

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190428173251756-1363824625.png)

getFilterName():获取当前 filter 的名字。获取的是在 web.xml 中配置的 filter-name 的值

getInitParameter(String name):获取 filter 的初始化参数。在 web.xml 中配置

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190428173315757-1400108751.png)

getInitParameterNames():获取 filter 初始化参数名的集合。

getServletContext():获取当前 web 工程的 ServletContext 对象。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## **Filter** 的 **url-pattern**

url-pattern 是配置 filter 过滤哪些请求的。主要有以下几种配置：

web.xml 中配置的/都是以当前项目路径为根路径的

1）精确匹配：

/index.jsp/user/login 会在请求/index.jsp、/user/login 的时候执行过滤方法

2）路径匹配：

/user/* /* 凡是路径为/user/下的所有请求都会被拦截，/*表示拦截系统的所有请求，包括静态资源文件。

3）扩展匹配：

*.jsp *.action 凡是后缀名为.jsp .action 的请求都会被拦截。

注意：/login/*.jsp 这种写法是错误的，只能是上述三种的任意一种形式。不能组合新形式。

*jsp 也是错误的，扩展匹配必须是后缀名

4）多重 url-pattern 配置

上面的三种形式比较有局限性，但是 url-pattern 可以配置多个，这样这三种组合基本就能解决所有问题了

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## 多 **Filter** 执行顺序

如果同一个资源有多个 filter 都对其拦截，则拦截的顺序是按照 web.xml 中配置的顺序进行的

执行流程图如下

![img](https://img2018.cnblogs.com/blog/1168971/201904/1168971-20190428173439110-1109481745.png)

 

请求总是在处理之后再回来执行 doFilter 之后的方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10785405.html#_labelTop)

## HttpServletWrapper 和 HttpServletResponseWrapper



### 定义

Servlet API 中提供了一个 HttpServletRequestWrapper 类来包装原始的 request 对象,HttpServletRequestWrapper 类实现了 HttpServletRequest 接口中的所有方法, 这些方法的内部实现都是仅仅调用了一下所包装的的 request 对象的对应方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//包装类实现 ServletRequest 接口. 
public class ServletRequestWrapper implements ServletRequest {

    //被包装的那个 ServletRequest 对象
    private ServletRequest request;
    
    //构造器传入 ServletRequest 实现类对象
    public ServletRequestWrapper(ServletRequest request) {
        if (request == null) {
            throw new IllegalArgumentException("Request cannot be null");   
        }
        this.request = request;
    }

    //具体实现 ServletRequest 的方法: 调用被包装的那个成员变量的方法实现。 
    public Object getAttribute(String name) {
        return this.request.getAttribute(name);
    }

    public Enumeration getAttributeNames() {
        return this.request.getAttributeNames();
    }    
    
    //...    
}    
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

相类似 Servlet API 也提供了一个 HttpServletResponseWrapper 类来包装原始的 response 对象



### 作用

用于对 HttpServletRequest 或 HttpServletResponse 的某一个方法进行修改或增强.

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyHttpServletRequest extends HttpServletRequestWrapper{

    public MyHttpServletRequest(HttpServletRequest request) {
        super(request);
    }
    
    @Override
    public String getParameter(String name) {
        String val = super.getParameter(name);
        if(val != null && val.contains(" fuck ")){
            val = val.replace("fuck", "****");
        }
        return val;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 使用

在 Filter 中, 利用 MyHttpServletRequest 替换传入的 HttpServletRequest

```
HttpServletRequest req = new MyHttpServletRequest(request);
filterChain.doFilter(req, response);
```

此时到达目标 Servlet 或 JSP 的 HttpServletRequest 实际上是 MyHttpServletRequest