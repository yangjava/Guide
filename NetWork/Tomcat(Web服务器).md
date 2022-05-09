# [Tomcat源码分析 （一）----- 手写一个web服务器](https://www.cnblogs.com/java-chen-hao/p/11316521.html)

**正文**

作为后端开发人员，在实际的工作中我们会非常高频地使用到web服务器。而tomcat作为web服务器领域中举足轻重的一个web框架，又是不能不学习和了解的。

tomcat其实是一个web框架，那么其内部是怎么实现的呢？如果不用tomcat我们能自己实现一个web服务器吗？

首先，tomcat内部的实现是非常复杂的，也有非常多的各类组件，我们在后续章节会深入地了解。
其次，本章我们将自己实现一个web服务器的。

下面我们就自己来实现一个看看。(【注】：参考了《How tomcat works》这本书)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316521.html#_labelTop)

## http协议简介

http是一种协议（超文本传输协议），允许web服务器和浏览器通过Internet来发送和接受数据，是一种请求/响应协议。http底层使用TCP来进行通信。目前，http已经迭代到了2.x版本，从最初的0.9、1.0、1.1到现在的2.x，每个迭代都加了很多功能。

在http中，始终都是客户端发起一个请求，服务器接受到请求之后，然后处理逻辑，处理完成之后再发送响应数据，客户端收到响应数据，然后请求结束。在这个过程中，客户端和服务器都可以对建立的连接进行中断操作。比如可以通过浏览器的停止按钮。

### http协议-请求

一个http协议的请求包含三部分：

1. 方法 URI 协议/版本
2. 请求的头部
3. 主体内容

举个例子

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
POST /examples/default.jsp HTTP/1.1
Accept: text/plain; text/html
Accept-Language: en-gb
Connection: Keep-Alive
Host: localhost
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 98)
Content-Length: 33
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate

lastName=Franks&firstName=Michael
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

数据的第一行包括：方法、URI、协议和版本。在这个例子里，方法为`POST`，URI为`/examples/default.jsp`，协议为`HTTP/1.1`，协议版本号为`1.1`。他们之间通过空格来分离。
请求头部从第二行开始，使用英文冒号（:）来分离键和值。
请求头部和主体内容之间通过空行来分离，例子中的请求体为表单数据。

### http协议-响应

类似于http协议的请求，响应也包含三个部分。

1. 协议 状态 状态描述
2. 响应的头部
3. 主体内容

举个例子

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
HTTP/1.1 200 OK
Server: Microsoft-IIS/4.0
Date: Mon, 5 Jan 2004 13:13:33 GMT
Content-Type: text/html
Last-Modified: Mon, 5 Jan 2004 13:13:12 GMT
Content-Length: 112

<html>
<head>
<title>HTTP Response Example</title> </head>
<body>
Welcome to Brainy Software
</body>
</html>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

第一行，`HTTP/1.1 200 OK`表示协议、状态和状态描述。
之后表示响应头部。
响应头部和主体内容之间使用空行来分离。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316521.html#_labelTop)

## Socket

Socket，又叫套接字，是网络连接的一个端点（end point）。套接字允许应用程序从网络中读取和写入数据。两个不同计算机的不同进程之间可以通过连接来发送和接受数据。A应用要向B应用发送数据，A应用需要知道B应用所在的IP地址和B应用开放的套接字端口。java里面使用`java.net.Socket`来表示一个套接字。

`java.net.Socket`最常用的一个构造方法为：`public Socket(String host, int port);`，host表示主机名或ip地址，port表示套接字端口。我们来看一个例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Socket socket = new Socket("127.0.0.1", "8080");
OutputStream os = socket.getOutputStream(); 
boolean autoflush = true;
PrintWriter out = new PrintWriter( socket.getOutputStream(), autoflush);
BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputstream()));

// send an HTTP request to the web server 
out.println("GET /index.jsp HTTP/1.1"); 
out.println("Host: localhost:8080"); 
out.println("Connection: Close");
out.println();

// read the response
boolean loop = true;
StringBuffer sb = new StringBuffer(8096); 
while (loop) {
    if (in.ready()) { 
        int i=0;
        while (i != -1) {
            i = in.read();
            sb.append((char) i); 
        }
        loop = false;
    }
    Thread.currentThread().sleep(50L);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这儿通过`socket.getOutputStream()`来发送数据，使用`socket.getInputstream()`来读取数据。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316521.html#_labelTop)

## ServerSocket

Socket表示一个客户端套接字，任何时候如果你想发送或接受数据，都需要构造创建一个Socket。现在假如我们需要一个服务器端的应用程序，我们需要额外考虑更多的东西。因为服务器需要随时待命，它不清楚什么时候一个客户端会连接到它。在java里面，我们可以通过`java.net.ServerSocket`来表示一个服务器套接字。
ServerSocket和Socket不同，它需要等待来自客户端的连接。一旦有客户端和其建立了连接，ServerSocket需要创建一个Socket来和客户端进行通信。
ServerSocket有很多的构造方法，我们拿其中的一个来举例子。

```
public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException;
new ServerSocket(8080, 1, InetAddress.getByName("127.0.0.1"));
```

1. port表示端口
2. backlog表示队列的长度
3. bindAddr表示地址

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316521.html#_labelTop)

## HttpServer

我们这儿还是看一个例子。

HttpServer表示一个服务器端入口，提供了一个main方法，并一直在8080端口等待，直到客户端建立一个连接。这时，服务器通过生成一个Socket来对此连接进行处理。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class HttpServer {

  /** WEB_ROOT is the directory where our HTML and other files reside.
   *  For this package, WEB_ROOT is the "webroot" directory under the working
   *  directory.
   *  The working directory is the location in the file system
   *  from where the java command was invoked.
   */
  public static final String WEB_ROOT =
    System.getProperty("user.dir") + File.separator  + "webroot";

  // shutdown command
  private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";

  // the shutdown command received
  private boolean shutdown = false;

  public static void main(String[] args) {
    HttpServer server = new HttpServer();
    server.await();
  }

  public void await() {
    ServerSocket serverSocket = null;
    int port = 8080;
    try {
      serverSocket =  new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
    }
    catch (IOException e) {
      e.printStackTrace();
      System.exit(1);
    }

    // Loop waiting for a request
    while (!shutdown) {
      Socket socket = null;
      InputStream input = null;
      OutputStream output = null;
      try {
        socket = serverSocket.accept();
        input = socket.getInputStream();
        output = socket.getOutputStream();

        // create Request object and parse
        Request request = new Request(input);
        request.parse();

        // create Response object
        Response response = new Response(output);
        response.setRequest(request);
        response.sendStaticResource();

        // Close the socket
        socket.close();

        //check if the previous URI is a shutdown command
        shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
      }
      catch (Exception e) {
        e.printStackTrace();
        continue;
      }
    }
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Request对象主要完成几件事情

- 解析请求数据
- 解析uri（请求数据第一行）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Request {

  private InputStream input;
  private String uri;

  public Request(InputStream input) {
    this.input = input;
  }

  public void parse() {
    // Read a set of characters from the socket
    StringBuffer request = new StringBuffer(2048);
    int i;
    byte[] buffer = new byte[2048];
    try {
      i = input.read(buffer);
    }
    catch (IOException e) {
      e.printStackTrace();
      i = -1;
    }
    for (int j=0; j<i; j++) {
      request.append((char) buffer[j]);
    }
    System.out.print(request.toString());
    uri = parseUri(request.toString());
  }

  private String parseUri(String requestString) {
    int index1, index2;
    index1 = requestString.indexOf(' ');
    if (index1 != -1) {
      index2 = requestString.indexOf(' ', index1 + 1);
      if (index2 > index1)
        return requestString.substring(index1 + 1, index2);
    }
    return null;
  }

  public String getUri() {
    return uri;
  }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Response主要是向客户端发送文件内容（如果请求的uri指向的文件存在）。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Response {

  private static final int BUFFER_SIZE = 1024;
  Request request;
  OutputStream output;

  public Response(OutputStream output) {
    this.output = output;
  }

  public void setRequest(Request request) {
    this.request = request;
  }

  public void sendStaticResource() throws IOException {
    byte[] bytes = new byte[BUFFER_SIZE];
    FileInputStream fis = null;
    try {
      File file = new File(HttpServer.WEB_ROOT, request.getUri());
      if (file.exists()) {
        fis = new FileInputStream(file);
        int ch = fis.read(bytes, 0, BUFFER_SIZE);
        while (ch!=-1) {
          output.write(bytes, 0, ch);
          ch = fis.read(bytes, 0, BUFFER_SIZE);
        }
      }
      else {
        // file not found
        String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
          "Content-Type: text/html\r\n" +
          "Content-Length: 23\r\n" +
          "\r\n" +
          "<h1>File Not Found</h1>";
        output.write(errorMessage.getBytes());
      }
    }
    catch (Exception e) {
      // thrown if cannot instantiate a File object
      System.out.println(e.toString() );
    }
    finally {
      if (fis!=null)
        fis.close();
    }
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316521.html#_labelTop)

## 总结

在看了上面的例子之后，我们惊奇地发现，在Java里面实现一个web服务器真容易，代码也非常简单和清晰！

既然我们能很简单地实现web服务器，为啥我们还需要tomcat呢？它又给我们带来了哪些组件和特性呢，它又是怎么组装这些组件的呢，后续章节我们将逐层分析。

这是我们后面将要分析的内容，让我们拭目以待！

# [Tomcat源码分析 （二）----- Tomcat整体架构及组件](https://www.cnblogs.com/java-chen-hao/p/11316795.html)

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316795.html#_labelTop)

## 前言

Tomcat的前身为Catalina，而Catalina又是一个轻量级的Servlet容器。在美国，catalina是一个很美的小岛。所以Tomcat作者的寓意可能是想把Tomcat设计成一个优雅美丽且轻量级的web服务器。Tomcat从4.x版本开始除了作为支持Servlet的容器外，额外加入了很多的功能，比如：jsp、el、naming等等，所以说Tomcat不仅仅是Catalina。

既然Tomcat首先是一个Servlet容器，我们应该更多的关心Servlet。

那么，什么是Servlet呢？

在互联网兴起之初，当时的Sun公司（后面被Oracle收购）已然看到了这次机遇，于是设计出了Applet来对Web应用的支持。不过事实却并不是预期那么得好，Sun悲催地发现Applet并没有给业界带来多大的影响。经过反思，Sun就想既然机遇出现了，市场前景也非常不错，总不能白白放弃了呀，怎么办呢？于是又投入精力去搞一套规范出来，这时Servlet诞生了！

**所谓Servlet，其实就是Sun为了让Java能实现动态可交互的网页，从而进入Web编程领域而制定的一套标准！**

一个Servlet主要做下面三件事情：

1. 创建并填充Request对象，包括：URI、参数、method、请求头信息、请求体信息等
2. 创建Response对象
3. 执行业务逻辑，将结果通过Response的输出流输出到客户端

Servlet没有main方法，所以，如果要执行，则需要在一个`容器`里面才能执行，这个容器就是为了支持Servlet的功能而存在，Tomcat其实就是一个Servlet容器的实现。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316795.html#_labelTop)

## 整体架构图

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807172526067-353517570.png)

从上图我们看出，最核心的两个组件--连接器（Connector）和容器（Container）起到`心脏`的作用，他们至关重要！他们的作用如下：

> 1、Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化;
> 2、Container用于封装和管理Servlet，以及具体处理Request请求；

一个Tomcat中只有一个Server，一个Server可以包含多个Service，一个Service只有一个Container，但是可以有多个Connectors，这是因为一个服务可以有多个连接，如同时提供Http和Https链接，也可以提供向相同协议不同端口的连接,示意图如下（Engine、Host、Context下边会说到）：

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807172958351-1132740453.png)

多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了！所以整个 Tomcat 的生命周期由 Server 控制。

 另外，上述的包含关系或者说是父子关系，都可以在tomcat的conf目录下的`server.xml`配置文件中看出

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807173324906-2029196119.png)

上边的配置文件，还可以通过下边的一张结构图更清楚的理解：

 ![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807173511076-1097002698.png)

下面我们逐一来分析各个组件的功能：

1. `Server`表示服务器，提供了一种优雅的方式来启动和停止整个系统，不必单独启停连接器和容器

2. `Service`表示服务，`Server`可以运行多个服务。比如一个Tomcat里面可运行订单服务、支付服务、用户服务等等

3. 每个`Service`可包含`多个Connector`和`一个Container`。因为每个服务允许同时支持多种协议，但是每种协议最终执行的Servlet却是相同的

4. `Connector`表示连接器，比如一个服务可以同时支持AJP协议、Http协议和Https协议，每种协议可使用一种连接器来支持

5. ```
   Container
   ```

   表示容器，可以看做Servlet容器

   - `Engine` -- 引擎
   - `Host` -- 主机
   - `Context` -- 上下文
   - `Wrapper` -- 包装器

6. Service服务之下还有各种

   ```
   支撑组件
   ```

   ，下面简单罗列一下这些组件

   - `Manager` -- 管理器，用于管理会话Session
   - `Logger` -- 日志器，用于管理日志
   - `Loader` -- 加载器，和类加载有关，只会开放给Context所使用
   - `Pipeline` -- 管道组件，配合Valve实现过滤器功能
   - `Valve` -- 阀门组件，配合Pipeline实现过滤器功能
   - `Realm` -- 认证授权组件

除了连接器和容器，管道组件和阀门组件也很关键，我们通过一张图来看看这两个组件

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807174018910-1559786217.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316795.html#_labelTop)

## Connector和Container的微妙关系

由上述内容我们大致可以知道一个请求发送到Tomcat之后，首先经过Service然后会交给我们的Connector，Connector用于接收请求并将接收的请求封装为Request和Response来具体处理，Request和Response封装完之后再交由Container进行处理，Container处理完请求之后再返回给Connector，最后在由Connector通过Socket将处理的结果返回给客户端，这样整个请求的就处理完了！

Connector最底层使用的是Socket来进行连接的，Request和Response是按照HTTP协议来封装的，所以Connector同时需要实现TCP/IP协议和HTTP协议！

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316795.html#_labelTop)

## Connector架构分析

Connector用于接受请求并将请求封装成Request和Response，然后交给Container进行处理，Container处理完之后在交给Connector返回给客户端。

因此，我们可以把Connector分为四个方面进行理解：

（1）Connector如何接受请求的？
（2）如何将请求封装成Request和Response的？
（3）封装完之后的Request和Response如何交给Container进行处理的？

首先看一下Connector的结构图，如下所示：

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807174351299-521376669.png)

Connector就是使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型，比如：Http11Protocol使用的是普通Socket来连接的，Http11NioProtocol使用的是NioSocket来连接的。

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

（1）Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request，Adapter用于将Request交给Container进行具体的处理。

（2）Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。

（3）Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316795.html#_labelTop)

## Container如何处理请求的

Container处理请求是使用Pipeline-Valve管道来处理的！（Valve是阀门之意）

Pipeline-Valve是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将处理后的请求返回，再让下一个处理着继续处理。

但是！Pipeline-Valve使用的责任链模式和普通的责任链模式有些不同！区别主要有以下两点：

（1）每个Pipeline都有特定的Valve，而且是在管道的最后一个执行，这个Valve叫做BaseValve，BaseValve是不可删除的；

（2）在上层容器的管道的BaseValve中会调用下层容器的管道。

我们知道Container包含四个子容器，而这四个子容器对应的BaseValve分别在：StandardEngineValve、StandardHostValve、StandardContextValve、StandardWrapperValve。

Pipeline的处理流程图如下：

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807174820042-2008919568.png)

（1）Connector在接收到请求后会首先调用最顶层容器的Pipeline来处理，这里的最顶层容器的Pipeline就是EnginePipeline（Engine的管道）；

（2）在Engine的管道中依次会执行EngineValve1、EngineValve2等等，最后会执行StandardEngineValve，在StandardEngineValve中会调用Host管道，然后再依次执行Host的HostValve1、HostValve2等，最后在执行StandardHostValve，然后再依次调用Context的管道和Wrapper的管道，最后执行到StandardWrapperValve。

（3）当执行到StandardWrapperValve的时候，会在StandardWrapperValve中创建FilterChain，并调用其doFilter方法来处理请求，这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet，其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理！

（4）当所有的Pipeline-Valve都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给Connector了，Connector在通过Socket的方式将结果返回给客户端。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316795.html#_labelTop)

## 总结

好了，我们已经从整体上看到了Tomcat的结构，但是对于每个组件我们并没有详细分析。后续章节我们会从几个方面来学习Tomcat：

1. 逐一分析各个组件
2. 通过断点的方式来跟踪Tomcat代码中的一次完整请求

# [Tomcat源码分析 （三）----- 生命周期机制 Lifecycle](https://www.cnblogs.com/java-chen-hao/p/11316902.html)

**正文**

Tomcat里面有各种各样的组件，每个组件各司其职，组件之间又相互协作共同完成web服务器这样的工程。在这些组件之上，Lifecycle（生命周期机制）至关重要！在学习各个组件之前，我们需要看看Lifecycle是什么以及能做什么？实现原理又是怎样的？

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316902.html#_labelTop)

## 什么是Lifecycle?

Lifecycle，其实就是一个状态机，对组件的由生到死状态的管理。

- 当组件在`STARTING_PREP`、`STARTING`或`STARTED`时，调用`start()`方法没有任何效果
- 当组件在`NEW`状态时，调用`start()`方法会导致`init()`方法被立刻执行，随后`start()`方法被执行
- 当组件在`STOPPING_PREP`、`STOPPING`或`STOPPED`时，调用`stop()`方法没有任何效果
- 当一个组件在`NEW`状态时，调用`stop()`方法会将组件状态变更为`STOPPED`，比较典型的场景就是组件启动失败，其子组件还没有启动。当一个组件停止的时候，它将尝试停止它下面的所有子组件，即使子组件还没有启动。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316902.html#_labelTop)

## Lifecycle方法

我们看看Lifecycle有哪些方法，如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface Lifecycle {
    // 添加监听器
    public void addLifecycleListener(LifecycleListener listener);
    // 获取所以监听器
    public LifecycleListener[] findLifecycleListeners();
    // 移除某个监听器
    public void removeLifecycleListener(LifecycleListener listener);
    // 初始化方法
    public void init() throws LifecycleException;
    // 启动方法
    public void start() throws LifecycleException;
    // 停止方法，和start对应
    public void stop() throws LifecycleException;
    // 销毁方法，和init对应
    public void destroy() throws LifecycleException;
    // 获取生命周期状态
    public LifecycleState getState();
    // 获取字符串类型的生命周期状态
    public String getStateName();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316902.html#_labelTop)

## LifecycleBase

`LifecycleBase`是`Lifecycle`的基本实现。我们逐一来看Lifecycle的各个方法。



### 增加、删除和获取监听器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final List<LifecycleListener> lifecycleListeners = new CopyOnWriteArrayList<>();

@Override
public void addLifecycleListener(LifecycleListener listener) {
    lifecycleListeners.add(listener);
}
@Override
public LifecycleListener[] findLifecycleListeners() {
    return lifecycleListeners.toArray(new LifecycleListener[0]);
}
@Override
public void removeLifecycleListener(LifecycleListener listener) {
    lifecycleListeners.remove(listener);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. 生命周期监听器保存在一个线程安全的List中，`CopyOnWriteArrayList`。所以add和remove都是直接调用此List的相应方法。
2. findLifecycleListeners返回的是一个数组，为了线程安全，所以这儿会生成一个新数组。



### init()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final synchronized void init() throws LifecycleException {
    // 非NEW状态，不允许调用init()方法
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }

    try {
        // 初始化逻辑之前，先将状态变更为`INITIALIZING`
        setStateInternal(LifecycleState.INITIALIZING, null, false);
        // 初始化，该方法为一个abstract方法，需要组件自行实现
        initInternal();
        // 初始化完成之后，状态变更为`INITIALIZED`
        setStateInternal(LifecycleState.INITIALIZED, null, false);
    } catch (Throwable t) {
        // 初始化的过程中，可能会有异常抛出，这时需要捕获异常，并将状态变更为`FAILED`
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.initFail",toString()), t);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`setStateInternal`方法用于维护状态，同时在状态转换成功之后触发事件。为了状态的可见性，所以state声明为volatile类型的。

```
private volatile LifecycleState state = LifecycleState.NEW;。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private synchronized void setStateInternal(LifecycleState state,
        Object data, boolean check) throws LifecycleException {
    if (log.isDebugEnabled()) {
        log.debug(sm.getString("lifecycleBase.setState", this, state));
    }

    // 是否校验状态
    if (check) {
        // Must have been triggered by one of the abstract methods (assume
        // code in this class is correct)
        // null is never a valid state
        // state不允许为null
        if (state == null) {
            invalidTransition("null");
            // Unreachable code - here to stop eclipse complaining about
            // a possible NPE further down the method
            return;
        }

        // Any method can transition to failed
        // startInternal() permits STARTING_PREP to STARTING
        // stopInternal() permits STOPPING_PREP to STOPPING and FAILED to
        // STOPPING
        if (!(state == LifecycleState.FAILED ||
                (this.state == LifecycleState.STARTING_PREP &&
                        state == LifecycleState.STARTING) ||
                (this.state == LifecycleState.STOPPING_PREP &&
                        state == LifecycleState.STOPPING) ||
                (this.state == LifecycleState.FAILED &&
                        state == LifecycleState.STOPPING))) {
            // No other transition permitted
            invalidTransition(state.name());
        }
    }

    // 设置状态
    this.state = state;
    // 触发事件
    String lifecycleEvent = state.getLifecycleEvent();
    if (lifecycleEvent != null) {
        fireLifecycleEvent(lifecycleEvent, data);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看看`fireLifecycleEvent`方法，

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void fireLifecycleEvent(String type, Object data) {
    // 事件监听,观察者模式的另一种方式
    LifecycleEvent event = new LifecycleEvent(lifecycle, type, data);
    LifecycleListener interested[] = listeners;// 监听器数组 关注 事件(启动或者关闭事件)
    // 循环通知所有生命周期时间侦听器
    for (int i = 0; i < interested.length; i++)
        // 每个监听器都有自己的逻辑
        interested[i].lifecycleEvent(event);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 首先, 创建一个事件对象, 然通知所有的监听器发生了该事件.并做响应.



### start()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final synchronized void start() throws LifecycleException {
    // `STARTING_PREP`、`STARTING`和`STARTED时，将忽略start()逻辑
    if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
            LifecycleState.STARTED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
        } else if (log.isInfoEnabled()) {
            log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
        }

        return;
    }

    // `NEW`状态时，执行init()方法
    if (state.equals(LifecycleState.NEW)) {
        init();
    }

    // `FAILED`状态时，执行stop()方法
    else if (state.equals(LifecycleState.FAILED)) {
        stop();
    }

    // 不是`INITIALIZED`和`STOPPED`时，则说明是非法的操作
    else if (!state.equals(LifecycleState.INITIALIZED) &&
            !state.equals(LifecycleState.STOPPED)) {
        invalidTransition(Lifecycle.BEFORE_START_EVENT);
    }

    try {
        // start前的状态设置
        setStateInternal(LifecycleState.STARTING_PREP, null, false);
        // start逻辑，抽象方法，由组件自行实现
        startInternal();
        // start过程中，可能因为某些原因失败，这时需要stop操作
        if (state.equals(LifecycleState.FAILED)) {
            // This is a 'controlled' failure. The component put itself into the
            // FAILED state so call stop() to complete the clean-up.
            stop();
        } else if (!state.equals(LifecycleState.STARTING)) {
            // Shouldn't be necessary but acts as a check that sub-classes are
            // doing what they are supposed to.
            invalidTransition(Lifecycle.AFTER_START_EVENT);
        } else {
            // 设置状态为STARTED
            setStateInternal(LifecycleState.STARTED, null, false);
        }
    } catch (Throwable t) {
        // This is an 'uncontrolled' failure so put the component into the
        // FAILED state and throw an exception.
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(sm.getString("lifecycleBase.startFail", toString()), t);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### stop()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final synchronized void stop() throws LifecycleException {
    // `STOPPING_PREP`、`STOPPING`和STOPPED时，将忽略stop()的执行
    if (LifecycleState.STOPPING_PREP.equals(state) || LifecycleState.STOPPING.equals(state) ||
            LifecycleState.STOPPED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyStopped", toString()), e);
        } else if (log.isInfoEnabled()) {
            log.info(sm.getString("lifecycleBase.alreadyStopped", toString()));
        }

        return;
    }

    // `NEW`状态时，直接将状态变更为`STOPPED`
    if (state.equals(LifecycleState.NEW)) {
        state = LifecycleState.STOPPED;
        return;
    }

    // stop()的执行，必须要是`STARTED`和`FAILED`
    if (!state.equals(LifecycleState.STARTED) && !state.equals(LifecycleState.FAILED)) {
        invalidTransition(Lifecycle.BEFORE_STOP_EVENT);
    }

    try {
        // `FAILED`时，直接触发BEFORE_STOP_EVENT事件
        if (state.equals(LifecycleState.FAILED)) {
            // Don't transition to STOPPING_PREP as that would briefly mark the
            // component as available but do ensure the BEFORE_STOP_EVENT is
            // fired
            fireLifecycleEvent(BEFORE_STOP_EVENT, null);
        } else {
            // 设置状态为STOPPING_PREP
            setStateInternal(LifecycleState.STOPPING_PREP, null, false);
        }

        // stop逻辑，抽象方法，组件自行实现
        stopInternal();

        // Shouldn't be necessary but acts as a check that sub-classes are
        // doing what they are supposed to.
        if (!state.equals(LifecycleState.STOPPING) && !state.equals(LifecycleState.FAILED)) {
            invalidTransition(Lifecycle.AFTER_STOP_EVENT);
        }
        // 设置状态为STOPPED
        setStateInternal(LifecycleState.STOPPED, null, false);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(sm.getString("lifecycleBase.stopFail",toString()), t);
    } finally {
        if (this instanceof Lifecycle.SingleUse) {
            // Complete stop process first
            setStateInternal(LifecycleState.STOPPED, null, false);
            destroy();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### destroy()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final synchronized void destroy() throws LifecycleException {
    // `FAILED`状态时，直接触发stop()逻辑
    if (LifecycleState.FAILED.equals(state)) {
        try {
            // Triggers clean-up
            stop();
        } catch (LifecycleException e) {
            // Just log. Still want to destroy.
            log.warn(sm.getString(
                    "lifecycleBase.destroyStopFail", toString()), e);
        }
    }

    // `DESTROYING`和`DESTROYED`时，忽略destroy的执行
    if (LifecycleState.DESTROYING.equals(state) ||
            LifecycleState.DESTROYED.equals(state)) {

        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyDestroyed", toString()), e);
        } else if (log.isInfoEnabled() && !(this instanceof Lifecycle.SingleUse)) {
            // Rather than have every component that might need to call
            // destroy() check for SingleUse, don't log an info message if
            // multiple calls are made to destroy()
            log.info(sm.getString("lifecycleBase.alreadyDestroyed", toString()));
        }

        return;
    }

    // 非法状态判断
    if (!state.equals(LifecycleState.STOPPED) &&
            !state.equals(LifecycleState.FAILED) &&
            !state.equals(LifecycleState.NEW) &&
            !state.equals(LifecycleState.INITIALIZED)) {
        invalidTransition(Lifecycle.BEFORE_DESTROY_EVENT);
    }

    try {
        // destroy前状态设置
        setStateInternal(LifecycleState.DESTROYING, null, false);
       // 抽象方法，组件自行实现
        destroyInternal();
        // destroy后状态设置
        setStateInternal(LifecycleState.DESTROYED, null, false);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.destroyFail",toString()), t);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 模板方法

从上述源码看得出来，`LifecycleBase`是使用了状态机+模板模式来实现的。模板方法有下面这几个：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 初始化方法
protected abstract void initInternal() throws LifecycleException;
// 启动方法
protected abstract void startInternal() throws LifecycleException;
// 停止方法
protected abstract void stopInternal() throws LifecycleException;
// 销毁方法
protected abstract void destroyInternal() throws LifecycleException;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316902.html#_labelTop)

## 总结

Lifecycle其实非常简单，代码也不复杂，但是剖析其实现对于我们理解组件的生命周期有很大的帮助，也有助于我们对设计模式的回顾。

# [Tomcat源码分析 （四）----- Pipeline和Valve](https://www.cnblogs.com/java-chen-hao/p/11341478.html)

**正文**

在 [Tomcat源码分析 （二）----- Tomcat整体架构及组件](https://www.cnblogs.com/java-chen-hao/p/11316795.html) 中我们简单分析了一下Pipeline和Valve，并给出了整体的结构图。而这一节，我们将详细分析Tomcat里面的源码。

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190812173223071-2090840547.png)

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341478.html#_labelTop)

## Valve

`Valve`作为一个个基础的阀门，扮演着业务实际执行者的角色。我们看看`Valve`这个接口有哪些方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface Valve {
    // 获取下一个阀门
    public Valve getNext();
    // 设置下一个阀门
    public void setNext(Valve valve);
    // 后台执行逻辑，主要在类加载上下文中使用到
    public void backgroundProcess();
    // 执行业务逻辑
    public void invoke(Request request, Response response)
        throws IOException, ServletException;
    // 是否异步执行
    public boolean isAsyncSupported();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### Contained

`ValveBase`、`Pipeline`及其他相关组件都实现了`Contained`接口，我们看看这个接口有哪些方法。很简单，就是get/set容器操作。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface Contained {

    /**
     * Get the {@link Container} with which this instance is associated.
     *
     * @return The Container with which this instance is associated or
     *         <code>null</code> if not associated with a Container
     */
    Container getContainer();


    /**
     * Set the <code>Container</code> with which this instance is associated.
     *
     * @param container The Container instance with which this instance is to
     *  be associated, or <code>null</code> to disassociate this instance
     *  from any Container
     */
    void setContainer(Container container);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### ValveBase

从Valve的类层次结构，我们发现几乎所有Valve都继承了`ValveBase`这个抽象类，所以这儿我们需要分析一下它。

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190812173531865-1318894649.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract class ValveBase extends LifecycleMBeanBase implements Contained, Valve {
    // 国际化管理器，可以支持多国语言
    protected static final StringManager sm = StringManager.getManager(ValveBase.class);

    //------------------------------------------------------ Instance Variables

    // 无参构造方法，默认不支持异步
    public ValveBase() {
        this(false);
    }
    // 有参构造方法，可传入异步支持标记
    public ValveBase(boolean asyncSupported) {
        this.asyncSupported = asyncSupported;
    }


    //------------------------------------------------------ Instance Variables

    // 异步标记
    protected boolean asyncSupported;
    // 所属容器
    protected Container container = null;
    // 容器日志组件对象
    protected Log containerLog = null;
    // 下一个阀门
    protected Valve next = null;


    //-------------------------------------------------------------- Properties

    // 获取所属容器
    @Override
    public Container getContainer() {
        return container;
    }
    // 设置所属容器
    @Override
    public void setContainer(Container container) {
        this.container = container;
    }
    // 是否异步执行
    @Override
    public boolean isAsyncSupported() {
        return asyncSupported;
    }
    // 设置是否异步执行
    public void setAsyncSupported(boolean asyncSupported) {
        this.asyncSupported = asyncSupported;
    }
    // 获取下一个待执行的阀门
    @Override
    public Valve getNext() {
        return next;
    }
    // 设置下一个待执行的阀门
    @Override
    public void setNext(Valve valve) {
        this.next = valve;
    }


    //---------------------------------------------------------- Public Methods

    // 后台执行，子类实现
    @Override
    public void backgroundProcess() {
        // NOOP by default
    }
    // 初始化逻辑
    @Override
    protected void initInternal() throws LifecycleException {
        super.initInternal();
        // 设置容器日志组件对象到当前阀门的containerLog属性
        containerLog = getContainer().getLogger();
    }
    // 启动逻辑
    @Override
    protected synchronized void startInternal() throws LifecycleException {
        setState(LifecycleState.STARTING);
    }
    // 停止逻辑
    @Override
    protected synchronized void stopInternal() throws LifecycleException {
        setState(LifecycleState.STOPPING);
    }
    // 重写toString，格式为[${containerName}]
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder(this.getClass().getName());
        sb.append('[');
        if (container == null) {
            sb.append("Container is null");
        } else {
            sb.append(container.getName());
        }
        sb.append(']');
        return sb.toString();
    }


    // -------------------- JMX and Registration  --------------------

    // 设置获取MBean对象的keyProperties，格式如：a=b,c=d,e=f...
    @Override
    public String getObjectNameKeyProperties() {
        StringBuilder name = new StringBuilder("type=Valve");

        Container container = getContainer();

        name.append(container.getMBeanKeyProperties());

        int seq = 0;

        // Pipeline may not be present in unit testing
        Pipeline p = container.getPipeline();
        if (p != null) {
            for (Valve valve : p.getValves()) {
                // Skip null valves
                if (valve == null) {
                    continue;
                }
                // Only compare valves in pipeline until we find this valve
                if (valve == this) {
                    break;
                }
                if (valve.getClass() == this.getClass()) {
                    // Duplicate valve earlier in pipeline
                    // increment sequence number
                    seq ++;
                }
            }
        }

        if (seq > 0) {
            name.append(",seq=");
            name.append(seq);
        }

        String className = this.getClass().getName();
        int period = className.lastIndexOf('.');
        if (period >= 0) {
            className = className.substring(period + 1);
        }
        name.append(",name=");
        name.append(className);

        return name.toString();
    }
    // 获取所属域，从container获取
    @Override
    public String getDomainInternal() {
        Container c = getContainer();
        if (c == null) {
            return null;
        } else {
            return c.getDomain();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341478.html#_labelTop)

## Pipeline

`Pipeline`作为一个管道，我们可以简单认为是一个Valve的集合，内部会对这个集合进行遍历，调用每个元素的业务逻辑方法`invoke()`。

是不是这样呢？我们还是分析一下源码，先看看接口定义。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface Pipeline {
    // ------------------------------------------------------------- Properties

    // 获取基本阀门
    public Valve getBasic();
    // 设置基本阀门
    public void setBasic(Valve valve);

    // --------------------------------------------------------- Public Methods

    // 添加阀门
    public void addValve(Valve valve);
    // 获取阀门数组
    public Valve[] getValves();
    // 删除阀门
    public void removeValve(Valve valve);
    // 获取首个阀门
    public Valve getFirst();
    // 管道内所有阀门是否异步执行
    public boolean isAsyncSupported();
    // 获取管道所属的容器
    public Container getContainer();
    // 设置管道所属的容器
    public void setContainer(Container container);
    // 查找非异步执行的所有阀门，并放置到result参数中，所以result不允许为null
    public void findNonAsyncValves(Set<String> result);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### StandardPipeline

接着我们分析一下`Pipeline`唯一的实现`StandardPipeline`。代码很长，但是都很简单。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class StandardPipeline extends LifecycleBase
        implements Pipeline, Contained {

    private static final Log log = LogFactory.getLog(StandardPipeline.class);

    // ----------------------------------------------------------- Constructors

    // 构造一个没有所属容器的管道
    public StandardPipeline() {
        this(null);
    }

    // 构造一个有所属容器的管道
    public StandardPipeline(Container container) {
        super();
        setContainer(container);
    }

    // ----------------------------------------------------- Instance Variables

    /**
     * 基本阀门，最后执行的阀门
     */
    protected Valve basic = null;

    /**
     * 管道所属的容器
     */
    protected Container container = null;

    /**
     * 管道里面的首个执行的阀门
     */
    protected Valve first = null;

    // --------------------------------------------------------- Public Methods

    // 是否异步执行，如果一个阀门都没有，或者所有阀门都是异步执行的，才返回true
    @Override
    public boolean isAsyncSupported() {
        Valve valve = (first!=null)?first:basic;
        boolean supported = true;
        while (supported && valve!=null) {
            supported = supported & valve.isAsyncSupported();
            valve = valve.getNext();
        }
        return supported;
    }

    // 查找所有未异步执行的阀门
    @Override
    public void findNonAsyncValves(Set<String> result) {
        Valve valve = (first!=null) ? first : basic;
        while (valve != null) {
            if (!valve.isAsyncSupported()) {
                result.add(valve.getClass().getName());
            }
            valve = valve.getNext();
        }
    }

    // ------------------------------------------------------ Contained Methods

    // 获取所属容器
    @Override
    public Container getContainer() {
        return (this.container);
    }

    // 设置所属容器
    @Override
    public void setContainer(Container container) {
        this.container = container;
    }

    // 初始化逻辑，默认没有任何逻辑
    @Override
    protected void initInternal() {
        // NOOP
    }

    // 开始逻辑，调用所有阀门的start方法
    @Override
    protected synchronized void startInternal() throws LifecycleException {
        // Start the Valves in our pipeline (including the basic), if any
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof Lifecycle)
                ((Lifecycle) current).start();
            current = current.getNext();
        }

        setState(LifecycleState.STARTING);
    }

    // 停止逻辑，调用所有阀门的stop方法
    @Override
    protected synchronized void stopInternal() throws LifecycleException {
        setState(LifecycleState.STOPPING);

        // Stop the Valves in our pipeline (including the basic), if any
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof Lifecycle)
                ((Lifecycle) current).stop();
            current = current.getNext();
        }
    }

    // 销毁逻辑，移掉所有阀门，调用removeValve方法
    @Override
    protected void destroyInternal() {
        Valve[] valves = getValves();
        for (Valve valve : valves) {
            removeValve(valve);
        }
    }

    /**
     * 重新toString方法
     */
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("Pipeline[");
        sb.append(container);
        sb.append(']');
        return sb.toString();
    }

    // ------------------------------------------------------- Pipeline Methods

    // 获取基础阀门
    @Override
    public Valve getBasic() {
        return (this.basic);
    }

    // 设置基础阀门
    @Override
    public void setBasic(Valve valve) {
        // Change components if necessary
        Valve oldBasic = this.basic;
        if (oldBasic == valve)
            return;

        // Stop the old component if necessary
        // 老的基础阀门会被调用stop方法且所属容器置为null
        if (oldBasic != null) {
            if (getState().isAvailable() && (oldBasic instanceof Lifecycle)) {
                try {
                    ((Lifecycle) oldBasic).stop();
                } catch (LifecycleException e) {
                    log.error("StandardPipeline.setBasic: stop", e);
                }
            }
            if (oldBasic instanceof Contained) {
                try {
                    ((Contained) oldBasic).setContainer(null);
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                }
            }
        }

        // Start the new component if necessary
        // 新的阀门会设置所属容器，并调用start方法
        if (valve == null)
            return;
        if (valve instanceof Contained) {
            ((Contained) valve).setContainer(this.container);
        }
        if (getState().isAvailable() && valve instanceof Lifecycle) {
            try {
                ((Lifecycle) valve).start();
            } catch (LifecycleException e) {
                log.error("StandardPipeline.setBasic: start", e);
                return;
            }
        }

        // Update the pipeline
        // 替换pipeline中的基础阀门，就是讲基础阀门的前一个阀门的next指向当前阀门
        Valve current = first;
        while (current != null) {
            if (current.getNext() == oldBasic) {
                current.setNext(valve);
                break;
            }
            current = current.getNext();
        }

        this.basic = valve;
    }

    // 添加阀门
    @Override
    public void addValve(Valve valve) {
        // Validate that we can add this Valve
        // 设置所属容器
        if (valve instanceof Contained)
            ((Contained) valve).setContainer(this.container);

        // Start the new component if necessary
        // 调用阀门的start方法
        if (getState().isAvailable()) {
            if (valve instanceof Lifecycle) {
                try {
                    ((Lifecycle) valve).start();
                } catch (LifecycleException e) {
                    log.error("StandardPipeline.addValve: start: ", e);
                }
            }
        }

        // Add this Valve to the set associated with this Pipeline
        // 设置阀门，将阀门添加到基础阀门的前一个
        if (first == null) {
            first = valve;
            valve.setNext(basic);
        } else {
            Valve current = first;
            while (current != null) {
                if (current.getNext() == basic) {
                    current.setNext(valve);
                    valve.setNext(basic);
                    break;
                }
                current = current.getNext();
            }
        }

        container.fireContainerEvent(Container.ADD_VALVE_EVENT, valve);
    }

    // 获取阀门数组
    @Override
    public Valve[] getValves() {
        ArrayList<Valve> valveList = new ArrayList<>();
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            valveList.add(current);
            current = current.getNext();
        }

        return valveList.toArray(new Valve[0]);
    }

    // JMX方法，在此忽略
    public ObjectName[] getValveObjectNames() {
        ArrayList<ObjectName> valveList = new ArrayList<>();
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof JmxEnabled) {
                valveList.add(((JmxEnabled) current).getObjectName());
            }
            current = current.getNext();
        }

        return valveList.toArray(new ObjectName[0]);
    }

    // 移除阀门
    @Override
    public void removeValve(Valve valve) {
        Valve current;
        if(first == valve) {
            // 如果待移出的阀门是首个阀门，则首个阀门的下一个阀门变成首个阀门
            first = first.getNext();
            current = null;
        } else {
            current = first;
        }
        // 遍历阀门集合，并进行移除
        while (current != null) {
            if (current.getNext() == valve) {
                current.setNext(valve.getNext());
                break;
            }
            current = current.getNext();
        }

        if (first == basic) first = null;

        // 设置阀门所属容器为null
        if (valve instanceof Contained)
            ((Contained) valve).setContainer(null);

        // 调用待移除阀门的stop方法和destroy方法，并触发移除阀门事件
        if (valve instanceof Lifecycle) {
            // Stop this valve if necessary
            if (getState().isAvailable()) {
                try {
                    ((Lifecycle) valve).stop();
                } catch (LifecycleException e) {
                    log.error("StandardPipeline.removeValve: stop: ", e);
                }
            }
            try {
                ((Lifecycle) valve).destroy();
            } catch (LifecycleException e) {
                log.error("StandardPipeline.removeValve: destroy: ", e);
            }
        }

        container.fireContainerEvent(Container.REMOVE_VALVE_EVENT, valve);
    }

    // 获取首个阀门，如果阀门列表为null，返回基础阀门
    @Override
    public Valve getFirst() {
        if (first != null) {
            return first;
        }
        return basic;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341478.html#_labelTop)

## 总结

通过上面的代码分析，我们发现了几个关键的设计模式：

1. 模板方法模式，父类定义框架，子类实现
2. 责任链模式，就是这儿的管道/阀门的实现方式，每个阀门维护一个next属性指向下一个阀门

# [Tomcat源码分析 （五）----- Tomcat 类加载器](https://www.cnblogs.com/java-chen-hao/p/11341004.html)

**正文**

在研究tomcat 类加载之前，我们复习一下或者说巩固一下java 默认的类加载器。楼主以前对类加载也是懵懵懂懂，借此机会，也好好复习一下。

楼主翻开了神书《深入理解Java虚拟机》第二版，p227, 关于类加载器的部分。请看：

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341004.html#_labelTop)

## 什么是类加载机制？

> Java虚拟机把描述类的数据从Class文件加载进内存，并对数据进行校验，转换解析和初始化，最终形成可以呗虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

> 虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这动作的代码模块成为“类加载器”。



### 类与类加载器的关系

> 类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远远不限于类加载阶段。对于任意一个类，都需要由**加载他的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性**，每一个类加载器，都拥有一个独立的类命名空间。这句话可以表达的更通俗一些：比较两个类是否“相等”，`只有在这两个类是由同一个类加载器加载的前提下才有意义`，否则，即使这两个类来自同一个Class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这个两个类就必定不相等。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341004.html#_labelTop)

## 什么是双亲委任模型？

1. 从Java虚拟机的角度来说，只存在两种不同类加载器：一种是**启动类加载器(Bootstrap ClassLoader)**，这个类加载器使用C++语言实现（只限HotSpot），是虚拟机自身的一部分；另一种就是所有其他的类加载器，这些类加载器都由Java语言实现，独立于虚拟机外部，并且全都继承自抽象类`java.lang.ClassLoader`.
2. 从Java开发人员的角度来看，类加载还可以划分的更细致一些，绝大部分Java程序员都会使用以下3种系统提供的类加载器：
   - 启动类加载器（Bootstrap ClassLoader）：这个类加载器复杂将存放在 JAVA_HOME/lib 目录中的，或者被-Xbootclasspath 参数所指定的路径种的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录下也不会重载）。
   - 扩展类加载器（Extension ClassLoader）：这个类加载器由sun.misc.Launcher$ExtClassLoader实现，它负责夹杂JAVA_HOME/lib/ext 目录下的，或者被java.ext.dirs 系统变量所指定的路径种的所有类库。开发者可以直接使用扩展类加载器。
   - 应用程序类加载器（Application ClassLoader）：这个类加载器由sun.misc.Launcher$AppClassLoader 实现。由于这个类加载器是ClassLoader 种的getSystemClassLoader方法的返回值，所以也成为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库。开发者可以直接使用这个类加载器，如果应用中没有定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

这些类加载器之间的关系一般如下图所示：

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190812164438786-348100315.png)

图中各个类加载器之间的关系成为 **类加载器的双亲委派模型（Parents Dlegation Mode）**。双亲委派模型要求除了顶层的启动类加载器之外，**其余的类加载器都应当由自己的父类加载器加载**，这里类加载器之间的父子关系一般不会以继承的关系来实现，而是都使用组合关系来复用父加载器的代码。

类加载器的双亲委派模型在JDK1.2 期间被引入并被广泛应用于之后的所有Java程序中，但他并不是个强制性的约束模型，而是Java设计者推荐给开发者的一种类加载器实现方式。

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，他首先不会自己去尝试加载这个类，而是把这个请求委派父类加载器去完成。每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个请求（他的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。



###  为什么要这么做呢？

 如果没有使用双亲委派模型，由各个类加载器自行加载的话，如果用户自己编写了一个称为java.lang.Object的类，并放在程序的ClassPath中，那系统将会出现多个不同的Object类， Java类型体系中最基础的行为就无法保证。应用程序也将会变得一片混乱。



### 双亲委任模型时如何实现的？

非常简单：所有的代码都在java.lang.ClassLoader中的loadClass方法之中，代码如下：

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190812165703474-1342686202.png)

先检查是否已经被加载过，若没有加载则调用父加载器的loadClass方法， 如父加载器为空则默认使用启动类加载器作为父加载器。如果父类加载失败，抛出ClassNotFoundException 异常后，再调用自己的findClass方法进行加载。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341004.html#_labelTop)

## 如何破坏双亲委任模型？

双亲委任模型不是一个强制性的约束模型，而是一个建议型的类加载器实现方式。在Java的世界中大部分的类加载器都遵循者模型，但也有例外，到目前为止，双亲委派模型有过3次大规模的“被破坏”的情况。
**第一次**：在双亲委派模型出现之前-----即JDK1.2发布之前。
**第二次**：是这个模型自身的缺陷导致的。我们说，双亲委派模型很好的解决了各个类加载器的基础类的统一问题（越基础的类由越上层的加载器进行加载），基础类之所以称为“基础”，是因为它们总是作为被用户代码调用的API， 但没有绝对，**如果基础类调用会用户的代码**怎么办呢？

这不是没有可能的。一个典型的例子就是JNDI服务，JNDI现在已经是Java的标准服务，它的代码由启动类加载器去加载（在JDK1.3时就放进去的rt.jar）,但它需要调用由独立厂商实现并部署在应用程序的ClassPath下的JNDI接口提供者（SPI， Service Provider Interface）的代码，但启动类加载器不可能“认识“这些代码啊。因为这些类不在rt.jar中，但是启动类加载器又需要加载。怎么办呢？

为了解决这个问题，Java设计团队只好引入了一个不太优雅的设计：**线程上下文类加载器（Thread Context ClassLoader）**。这个类加载器可以通过java.lang.Thread类的setContextClassLoader方法进行设置。如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过多的话，那这个类加载器默认即使应用程序类加载器。

嘿嘿，有了线程上下文加载器，JNDI服务使用这个线程上下文加载器去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载的动作，这种行为实际上就是打通了双亲委派模型的层次结构来逆向使用类加载器，实际上已经违背了双亲委派模型的一般性原则。但这无可奈何，Java中所有涉及SPI的加载动作基本胜都采用这种方式。例如JNDI，**JDBC**，JCE，JAXB，JBI等。

**第三次**：为了实现热插拔，热部署，模块化，意思是添加一个功能或减去一个功能不用重启，只需要把这模块连同类加载器一起换掉就实现了代码的热替换。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341004.html#_labelTop)

## Tomcat 的类加载器是怎么设计的？

首先，我们来问个问题：

### Tomcat 如果使用默认的类加载机制行不行？

我们思考一下：Tomcat是个web容器， 那么它要解决什么问题：

1. 一个web容器可能需要部署两个应用程序，不同的应用程序可能会依赖**同一个第三方类库的不同版本**，不能要求同一个类库在同一个服务器只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离。
2. 部署在同一个web容器中相同的类库**相同的版本可以共享**。否则，如果服务器有10个应用程序，那么要有10份相同的类库加载进虚拟机，这是扯淡的。
3. **web容器也有自己依赖的类库**，不能于应用程序的类库混淆。基于安全考虑，应该让容器的类库和程序的类库隔离开来。
4. web容器要支持jsp的修改，我们知道，jsp 文件最终也是要编译成class文件才能在虚拟机中运行，但程序运行后修改jsp已经是司空见惯的事情，否则要你何用？ 所以，web容器需要支持 jsp 修改后不用重启。

 

再看看我们的问题：Tomcat 如果使用默认的类加载机制行不行？
答案是不行的。为什么？我们看，第一个问题，如果使用默认的类加载器机制，那么是无法加载两个相同类库的不同版本的，默认的类加器是不管你是什么版本的，只在乎你的全限定类名，并且只有一份。第二个问题，默认的类加载器是能够实现的，因为他的职责就是保证唯一性。第三个问题和第一个问题一样。我们再看第四个问题，我们想我们要怎么实现jsp文件的热修改（楼主起的名字），jsp 文件其实也就是class文件，那么如果修改了，但类名还是一样，类加载器会直接取方法区中已经存在的，修改后的jsp是不会重新加载的。那么怎么办呢？我们可以直接卸载掉这jsp文件的类加载器，所以你应该想到了，每个jsp文件对应一个唯一的类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器。重新创建类加载器，重新加载jsp文件。



### Tomcat 如何实现自己独特的类加载机制？

我们看看他们的设计图：

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190812170624553-784647721.png)

我们在这张图中看到很多类加载器，除了Jdk自带的类加载器，我们尤其关心Tomcat自身持有的类加载器。仔细一点我们很容易发现：Catalina类加载器和Shared类加载器，他们并不是父子关系，而是兄弟关系。为啥这样设计，我们得分析一下每个类加载器的用途，才能知晓。

1. Common类加载器，负责加载Tomcat和Web应用都复用的类
2. Catalina类加载器，负责加载Tomcat专用的类，而这些被加载的类在Web应用中将不可见
3. Shared类加载器，负责加载Tomcat下所有的Web应用程序都复用的类，而这些被加载的类在Tomcat中将不可见
4. WebApp类加载器，负责加载具体的某个Web应用程序所使用到的类，而这些被加载的类在Tomcat和其他的Web应用程序都将不可见
5. Jsp类加载器，每个jsp页面一个类加载器，不同的jsp页面有不同的类加载器，方便实现jsp页面的热插拔

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341004.html#_labelTop)

## 源码阅读

Tomcat启动的入口在`Bootstrap的main()方法`。`main()`方法执行前，必然先执行其`static{}`块。所以我们首先分析`static{}`块，然后分析`main()`方法



### Bootstrap.static{}

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static {
    // 获取用户目录
    // Will always be non-null
    String userDir = System.getProperty("user.dir");

    // 第一步，从环境变量中获取catalina.home，在没有获取到的时候将执行后面的获取操作
    // Home first
    String home = System.getProperty(Globals.CATALINA_HOME_PROP);
    File homeFile = null;

    if (home != null) {
        File f = new File(home);
        try {
            homeFile = f.getCanonicalFile();
        } catch (IOException ioe) {
            homeFile = f.getAbsoluteFile();
        }
    }

    // 第二步，在第一步没获取的时候，从bootstrap.jar所在目录的上一级目录获取
    if (homeFile == null) {
        // First fall-back. See if current directory is a bin directory
        // in a normal Tomcat install
        File bootstrapJar = new File(userDir, "bootstrap.jar");

        if (bootstrapJar.exists()) {
            File f = new File(userDir, "..");
            try {
                homeFile = f.getCanonicalFile();
            } catch (IOException ioe) {
                homeFile = f.getAbsoluteFile();
            }
        }
    }

    // 第三步，第二步中的bootstrap.jar可能不存在，这时我们直接把user.dir作为我们的home目录
    if (homeFile == null) {
        // Second fall-back. Use current directory
        File f = new File(userDir);
        try {
            homeFile = f.getCanonicalFile();
        } catch (IOException ioe) {
            homeFile = f.getAbsoluteFile();
        }
    }

    // 重新设置catalinaHome属性
    catalinaHomeFile = homeFile;
    System.setProperty(
            Globals.CATALINA_HOME_PROP, catalinaHomeFile.getPath());

    // 接下来获取CATALINA_BASE（从系统变量中获取），若不存在，则将CATALINA_BASE保持和CATALINA_HOME相同
    // Then base
    String base = System.getProperty(Globals.CATALINA_BASE_PROP);
    if (base == null) {
        catalinaBaseFile = catalinaHomeFile;
    } else {
        File baseFile = new File(base);
        try {
            baseFile = baseFile.getCanonicalFile();
        } catch (IOException ioe) {
            baseFile = baseFile.getAbsoluteFile();
        }
        catalinaBaseFile = baseFile;
    }
   // 重新设置catalinaBase属性
    System.setProperty(
            Globals.CATALINA_BASE_PROP, catalinaBaseFile.getPath());
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们把代码中的注释搬下来总结一下：

1. 获取用户目录
2. 第一步，从环境变量中获取catalina.home，在没有获取到的时候将执行后面的获取操作
3. 第二步，在第一步没获取的时候，从bootstrap.jar所在目录的上一级目录获取
4. 第三步，第二步中的bootstrap.jar可能不存在，这时我们直接把user.dir作为我们的home目录
5. 重新设置catalinaHome属性
6. 接下来获取CATALINA_BASE（从系统变量中获取），若不存在，则将CATALINA_BASE保持和CATALINA_HOME相同
7. 重新设置catalinaBase属性

简单总结一下，就是加载并设置catalinaHome和catalinaBase相关的信息，以备后续使用。



### main()

main方法大体分成两块，一块为**init**，另一块为**load+start**。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String args[]) {
    // 第一块，main方法第一次执行的时候，daemon肯定为null，所以直接new了一个Bootstrap对象，然后执行其init()方法
    if (daemon == null) {
        // Don't set daemon until init() has completed
        Bootstrap bootstrap = new Bootstrap();
        try {
            bootstrap.init();
        } catch (Throwable t) {
            handleThrowable(t);
            t.printStackTrace();
            return;
        }
        // daemon守护对象设置为bootstrap
        daemon = bootstrap;
    } else {
        // When running as a service the call to stop will be on a new
        // thread so make sure the correct class loader is used to prevent
        // a range of class not found exceptions.
        Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
    }

    // 第二块，执行守护对象的load方法和start方法
    try {
        String command = "start";
        if (args.length > 0) {
            command = args[args.length - 1];
        }

        if (command.equals("startd")) {
            args[args.length - 1] = "start";
            daemon.load(args);
            daemon.start();
        } else if (command.equals("stopd")) {
            args[args.length - 1] = "stop";
            daemon.stop();
        } else if (command.equals("start")) {
            daemon.setAwait(true);
            daemon.load(args);
            daemon.start();
            if (null == daemon.getServer()) {
                System.exit(1);
            }
        } else if (command.equals("stop")) {
            daemon.stopServer(args);
        } else if (command.equals("configtest")) {
            daemon.load(args);
            if (null == daemon.getServer()) {
                System.exit(1);
            }
            System.exit(0);
        } else {
            log.warn("Bootstrap: command \"" + command + "\" does not exist.");
        }
    } catch (Throwable t) {
        // Unwrap the Exception for clearer error reporting
        if (t instanceof InvocationTargetException &&
                t.getCause() != null) {
            t = t.getCause();
        }
        handleThrowable(t);
        t.printStackTrace();
        System.exit(1);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们点到`init()`里面去看看~

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void init() throws Exception {
    // 非常关键的地方，初始化类加载器s，后面我们会详细具体地分析这个方法
    initClassLoaders();

    // 设置上下文类加载器为catalinaLoader，这个类加载器负责加载Tomcat专用的类
    Thread.currentThread().setContextClassLoader(catalinaLoader);
    // 暂时略过，后面会讲
    SecurityClassLoad.securityClassLoad(catalinaLoader);

    // 使用catalinaLoader加载我们的Catalina类
    // Load our startup class and call its process() method
    if (log.isDebugEnabled())
        log.debug("Loading startup class");
    Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
    Object startupInstance = startupClass.getConstructor().newInstance();

    // 设置Catalina类的parentClassLoader属性为sharedLoader
    // Set the shared extensions class loader
    if (log.isDebugEnabled())
        log.debug("Setting startup class properties");
    String methodName = "setParentClassLoader";
    Class<?> paramTypes[] = new Class[1];
    paramTypes[0] = Class.forName("java.lang.ClassLoader");
    Object paramValues[] = new Object[1];
    paramValues[0] = sharedLoader;
    Method method =
        startupInstance.getClass().getMethod(methodName, paramTypes);
    method.invoke(startupInstance, paramValues);

    // catalina守护对象为刚才使用catalinaLoader加载类、并初始化出来的Catalina对象
    catalinaDaemon = startupInstance;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

关键的方法`initClassLoaders`，这个方法负责初始化Tomcat的类加载器。通过这个方法，我们很容易验证我们上一小节提到的Tomcat类加载图。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void initClassLoaders() {
    try {
        // 创建commonLoader，如果未创建成果的话，则使用应用程序类加载器作为commonLoader
        commonLoader = createClassLoader("common", null);
        if( commonLoader == null ) {
            // no config file, default to this loader - we might be in a 'single' env.
            commonLoader=this.getClass().getClassLoader();
        }
        // 创建catalinaLoader，父类加载器为commonLoader
        catalinaLoader = createClassLoader("server", commonLoader);
        // 创建sharedLoader，父类加载器为commonLoader
        sharedLoader = createClassLoader("shared", commonLoader);
    } catch (Throwable t) {
        // 如果创建的过程中出现异常了，日志记录完成之后直接系统退出
        handleThrowable(t);
        log.error("Class loader creation threw exception", t);
        System.exit(1);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

所有的类加载器的创建都使用到了方法`createClassLoader`，所以，我们进一步分析一下这个方法。`createClassLoader`用到了CatalinaProperties.getProperty("xxx")方法，这个方法用于从`conf/catalina.properties`文件获取属性值。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private ClassLoader createClassLoader(String name, ClassLoader parent)
    throws Exception {
    // 获取类加载器待加载的位置，如果为空，则不需要加载特定的位置，使用父类加载返回回去。
    String value = CatalinaProperties.getProperty(name + ".loader");
    if ((value == null) || (value.equals("")))
        return parent;
    // 替换属性变量，比如：${catalina.base}、${catalina.home}
    value = replace(value);

    List<Repository> repositories = new ArrayList<>();

   // 解析属性路径变量为仓库路径数组
    String[] repositoryPaths = getPaths(value);

    // 对每个仓库路径进行repositories设置。我们可以把repositories看成一个个待加载的位置对象，可以是一个classes目录，一个jar文件目录等等
    for (String repository : repositoryPaths) {
        // Check for a JAR URL repository
        try {
            @SuppressWarnings("unused")
            URL url = new URL(repository);
            repositories.add(
                    new Repository(repository, RepositoryType.URL));
            continue;
        } catch (MalformedURLException e) {
            // Ignore
        }

        // Local repository
        if (repository.endsWith("*.jar")) {
            repository = repository.substring
                (0, repository.length() - "*.jar".length());
            repositories.add(
                    new Repository(repository, RepositoryType.GLOB));
        } else if (repository.endsWith(".jar")) {
            repositories.add(
                    new Repository(repository, RepositoryType.JAR));
        } else {
            repositories.add(
                    new Repository(repository, RepositoryType.DIR));
        }
    }
    // 使用类加载器工厂创建一个类加载器
    return ClassLoaderFactory.createClassLoader(repositories, parent);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来分析一下`ClassLoaderFactory.createClassLoader`--类加载器工厂创建类加载器。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static ClassLoader createClassLoader(List<Repository> repositories,
                                            final ClassLoader parent)
    throws Exception {

    if (log.isDebugEnabled())
        log.debug("Creating new class loader");

    // Construct the "class path" for this class loader
    Set<URL> set = new LinkedHashSet<>();
    // 遍历repositories，对每个repository进行类型判断，并生成URL，每个URL我们都要校验其有效性，有效的URL我们会放到URL集合中
    if (repositories != null) {
        for (Repository repository : repositories)  {
            if (repository.getType() == RepositoryType.URL) {
                URL url = buildClassLoaderUrl(repository.getLocation());
                if (log.isDebugEnabled())
                    log.debug("  Including URL " + url);
                set.add(url);
            } else if (repository.getType() == RepositoryType.DIR) {
                File directory = new File(repository.getLocation());
                directory = directory.getCanonicalFile();
                if (!validateFile(directory, RepositoryType.DIR)) {
                    continue;
                }
                URL url = buildClassLoaderUrl(directory);
                if (log.isDebugEnabled())
                    log.debug("  Including directory " + url);
                set.add(url);
            } else if (repository.getType() == RepositoryType.JAR) {
                File file=new File(repository.getLocation());
                file = file.getCanonicalFile();
                if (!validateFile(file, RepositoryType.JAR)) {
                    continue;
                }
                URL url = buildClassLoaderUrl(file);
                if (log.isDebugEnabled())
                    log.debug("  Including jar file " + url);
                set.add(url);
            } else if (repository.getType() == RepositoryType.GLOB) {
                File directory=new File(repository.getLocation());
                directory = directory.getCanonicalFile();
                if (!validateFile(directory, RepositoryType.GLOB)) {
                    continue;
                }
                if (log.isDebugEnabled())
                    log.debug("  Including directory glob "
                        + directory.getAbsolutePath());
                String filenames[] = directory.list();
                if (filenames == null) {
                    continue;
                }
                for (int j = 0; j < filenames.length; j++) {
                    String filename = filenames[j].toLowerCase(Locale.ENGLISH);
                    if (!filename.endsWith(".jar"))
                        continue;
                    File file = new File(directory, filenames[j]);
                    file = file.getCanonicalFile();
                    if (!validateFile(file, RepositoryType.JAR)) {
                        continue;
                    }
                    if (log.isDebugEnabled())
                        log.debug("    Including glob jar file "
                            + file.getAbsolutePath());
                    URL url = buildClassLoaderUrl(file);
                    set.add(url);
                }
            }
        }
    }

    // Construct the class loader itself
    final URL[] array = set.toArray(new URL[set.size()]);
    if (log.isDebugEnabled())
        for (int i = 0; i < array.length; i++) {
            log.debug("  location " + i + " is " + array[i]);
        }

    // 从这儿看，最终所有的类加载器都是URLClassLoader的对象~~
    return AccessController.doPrivileged(
            new PrivilegedAction<URLClassLoader>() {
                @Override
                public URLClassLoader run() {
                    if (parent == null)
                        return new URLClassLoader(array);
                    else
                        return new URLClassLoader(array, parent);
                }
            });
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们已经对initClassLoaders分析完了，接下来分析`SecurityClassLoad.securityClassLoad`，我们看看里面做了什么事情

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void securityClassLoad(ClassLoader loader) throws Exception {
    securityClassLoad(loader, true);
}

static void securityClassLoad(ClassLoader loader, boolean requireSecurityManager) throws Exception {

    if (requireSecurityManager && System.getSecurityManager() == null) {
        return;
    }

    loadCorePackage(loader);
    loadCoyotePackage(loader);
    loadLoaderPackage(loader);
    loadRealmPackage(loader);
    loadServletsPackage(loader);
    loadSessionPackage(loader);
    loadUtilPackage(loader);
    loadValvesPackage(loader);
    loadJavaxPackage(loader);
    loadConnectorPackage(loader);
    loadTomcatPackage(loader);
}

 private static final void loadCorePackage(ClassLoader loader) throws Exception {
    final String basePackage = "org.apache.catalina.core.";
    loader.loadClass(basePackage + "AccessLogAdapter");
    loader.loadClass(basePackage + "ApplicationContextFacade$PrivilegedExecuteMethod");
    loader.loadClass(basePackage + "ApplicationDispatcher$PrivilegedForward");
    loader.loadClass(basePackage + "ApplicationDispatcher$PrivilegedInclude");
    loader.loadClass(basePackage + "ApplicationPushBuilder");
    loader.loadClass(basePackage + "AsyncContextImpl");
    loader.loadClass(basePackage + "AsyncContextImpl$AsyncRunnable");
    loader.loadClass(basePackage + "AsyncContextImpl$DebugException");
    loader.loadClass(basePackage + "AsyncListenerWrapper");
    loader.loadClass(basePackage + "ContainerBase$PrivilegedAddChild");
    loadAnonymousInnerClasses(loader, basePackage + "DefaultInstanceManager");
    loader.loadClass(basePackage + "DefaultInstanceManager$AnnotationCacheEntry");
    loader.loadClass(basePackage + "DefaultInstanceManager$AnnotationCacheEntryType");
    loader.loadClass(basePackage + "ApplicationHttpRequest$AttributeNamesEnumerator");
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这儿其实就是使用catalinaLoader加载tomcat源代码里面的各个专用类。我们大致罗列一下待加载的类所在的package：

1. org.apache.catalina.core.*
2. org.apache.coyote.*
3. org.apache.catalina.loader.*
4. org.apache.catalina.realm.*
5. org.apache.catalina.servlets.*
6. org.apache.catalina.session.*
7. org.apache.catalina.util.*
8. org.apache.catalina.valves.*
9. javax.servlet.http.Cookie
10. org.apache.catalina.connector.*
11. org.apache.tomcat.*

好了，至此我们已经分析完了init里面涉及到的几个关键方法



### WebApp类加载器

到这儿，我们隐隐感觉到少分析了点什么！没错，就是WebApp类加载器。整个启动过程分析下来，我们仍然没有看到这个类加载器。它又是在哪儿出现的呢？

我们知道WebApp类加载器是Web应用私有的，而每个Web应用其实算是一个Context，那么我们通过Context的实现类应该可以发现。在Tomcat中，Context的默认实现为`StandardContext`，我们看看这个类的`startInternal()方法`，在这儿我们发现了我们感兴趣的WebApp类加载器。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected synchronized void startInternal() throws LifecycleException {
    if (getLoader() == null) {
        WebappLoader webappLoader = new WebappLoader(getParentClassLoader());
        webappLoader.setDelegate(getDelegate());
        setLoader(webappLoader);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

入口代码非常简单，就是webappLoader不存在的时候创建一个，并调用`setLoader`方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11341004.html#_labelTop)

## 总结

我们终于完整地分析完了Tomcat的整个启动过程+类加载过程。也了解并学习了Tomcat不同的类加载机制是为什么要这样设计，带来的附加作用又是怎样的。

# [Tomcat源码分析 （六）----- Tomcat 启动过程(一)](https://www.cnblogs.com/java-chen-hao/p/11344968.html)

**正文**

说到Tomcat的启动，我们都知道，我们每次需要运行tomcat/bin/startup.sh这个脚本，而这个脚本的内容到底是什么呢？我们来看看。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344968.html#_labelTop)

## 启动脚本



### startup.sh 脚本

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#!/bin/sh
os400=false
case "`uname`" in
OS400*) os400=true;;
esac

# resolve links - $0 may be a softlink
PRG="$0"

while [ -h "$PRG" ] ; do
  ls=`ls -ld "$PRG"`
  link=`expr "$ls" : '.*-> \(.*\)$'`
  if expr "$link" : '/.*' > /dev/null; then
    PRG="$link"
  else
    PRG=`dirname "$PRG"`/"$link"
  fi
done

PRGDIR=`dirname "$PRG"`
EXECUTABLE=catalina.sh

# Check that target executable exists
if $os400; then
  # -x will Only work on the os400 if the files are:
  # 1. owned by the user
  # 2. owned by the PRIMARY group of the user
  # this will not work if the user belongs in secondary groups
  eval
else
  if [ ! -x "$PRGDIR"/"$EXECUTABLE" ]; then
    echo "Cannot find $PRGDIR/$EXECUTABLE"
    echo "The file is absent or does not have execute permission"
    echo "This file is needed to run this program"
    exit 1
  fi
fi

exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看这脚本。该脚本中有2个重要的变量：

1. PRGDIR：表示当前脚本所在的路径
2. EXECUTABLE：catalina.sh 脚本名称
   其中最关键的一行代码就是 `exec "$PRGDIR"/"$EXECUTABLE" start "$@"`，表示执行了脚本catalina.sh，参数是start。

### catalina.sh 脚本

然后我们看看catalina.sh 脚本中的实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
elif [ "$1" = "start" ] ; then

  if [ ! -z "$CATALINA_PID" ]; then
    if [ -f "$CATALINA_PID" ]; then
      if [ -s "$CATALINA_PID" ]; then
        echo "Existing PID file found during start."
        if [ -r "$CATALINA_PID" ]; then
          PID=`cat "$CATALINA_PID"`
          ps -p $PID >/dev/null 2>&1
          if [ $? -eq 0 ] ; then
            echo "Tomcat appears to still be running with PID $PID. Start aborted."
            echo "If the following process is not a Tomcat process, remove the PID file and try again:"
            ps -f -p $PID
            exit 1
          else
            echo "Removing/clearing stale PID file."
            rm -f "$CATALINA_PID" >/dev/null 2>&1
            if [ $? != 0 ]; then
              if [ -w "$CATALINA_PID" ]; then
                cat /dev/null > "$CATALINA_PID"
              else
                echo "Unable to remove or clear stale PID file. Start aborted."
                exit 1
              fi
            fi
          fi
        else
          echo "Unable to read PID file. Start aborted."
          exit 1
        fi
      else
        rm -f "$CATALINA_PID" >/dev/null 2>&1
        if [ $? != 0 ]; then
          if [ ! -w "$CATALINA_PID" ]; then
            echo "Unable to remove or write to empty PID file. Start aborted."
            exit 1
          fi
        fi
      fi
    fi
  fi

  shift
  touch "$CATALINA_OUT"
  if [ "$1" = "-security" ] ; then
    if [ $have_tty -eq 1 ]; then
      echo "Using Security Manager"
    fi
    shift
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -classpath "\"$CLASSPATH\"" \
      -Djava.security.manager \
      -Djava.security.policy=="\"$CATALINA_BASE/conf/catalina.policy\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

  else
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -classpath "\"$CLASSPATH\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

  fi

  if [ ! -z "$CATALINA_PID" ]; then
    echo $! > "$CATALINA_PID"
  fi

  echo "Tomcat started."
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该脚本很长，但我们只关心我们感兴趣的：如果参数是 `start`， 那么执行这里的逻辑，关键再最后一行执行了 `org.apache.catalina.startup.Bootstrap "$@" start`， 也就是说，执行了我们熟悉的main方法，并且携带了start 参数，那么我们就来看Bootstrap 的main方法是如何实现的。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344968.html#_labelTop)

## Bootstrap.main

首先我们启动 main 方法:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static void main(String args[]) {
    System.err.println("Have fun and Enjoy! cxs");

    // daemon 就是 bootstrap
    if (daemon == null) {
        Bootstrap bootstrap = new Bootstrap();
        try {
            //类加载机制我们前面已经讲过，在这里就不在重复了
            bootstrap.init();
        } catch (Throwable t) {
            handleThrowable(t);
            t.printStackTrace();
            return;
        }
        daemon = bootstrap;
    } else {
        Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
    }
    try {
        // 命令
        String command = "start";
        // 如果命令行中输入了参数
        if (args.length > 0) {
            // 命令 = 最后一个命令
            command = args[args.length - 1];
        }
        // 如果命令是启动
        if (command.equals("startd")) {
            args[args.length - 1] = "start";
            daemon.load(args);
            daemon.start();
        }
        // 如果命令是停止了
        else if (command.equals("stopd")) {
            args[args.length - 1] = "stop";
            daemon.stop();
        }
        // 如果命令是启动
        else if (command.equals("start")) {
            daemon.setAwait(true);// bootstrap 和 Catalina 一脉相连, 这里设置, 方法内部设置 Catalina 实例setAwait方法
            daemon.load(args);// args 为 空,方法内部调用 Catalina 的 load 方法.
            daemon.start();// 相同, 反射调用 Catalina 的 start 方法 ,至此,启动结束
        } else if (command.equals("stop")) {
            daemon.stopServer(args);
        } else if (command.equals("configtest")) {
            daemon.load(args);
            if (null==daemon.getServer()) {
                System.exit(1);
            }
            System.exit(0);
        } else {
            log.warn("Bootstrap: command \"" + command + "\" does not exist.");
        }
    } catch (Throwable t) {
        // Unwrap the Exception for clearer error reporting
        if (t instanceof InvocationTargetException &&
                t.getCause() != null) {
            t = t.getCause();
        }
        handleThrowable(t);
        t.printStackTrace();
        System.exit(1);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看bootstrap.init();的部分代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void init() throws Exception {

    // 类加载机制我们前面已经讲过，在这里就不在重复了
    initClassLoaders();

    Thread.currentThread().setContextClassLoader(catalinaLoader);
    SecurityClassLoad.securityClassLoad(catalinaLoader);

    // 反射方法实例化Catalina
    Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
    Object startupInstance = startupClass.getConstructor().newInstance();

   
    String methodName = "setParentClassLoader";
    Class<?> paramTypes[] = new Class[1];
    paramTypes[0] = Class.forName("java.lang.ClassLoader");
    Object paramValues[] = new Object[1];
    paramValues[0] = sharedLoader;
    Method method =
        startupInstance.getClass().getMethod(methodName, paramTypes);
    method.invoke(startupInstance, paramValues);

    // 引用Catalina实例
    catalinaDaemon = startupInstance;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们可以看到是通过反射实例化Catalina类，并将实例引用赋值给catalinaDaemon,接着我们看看**daemon.load(args);**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void load(String[] arguments)
    throws Exception {

    // Call the load() method
    String methodName = "load";
    Object param[];
    Class<?> paramTypes[];
    if (arguments==null || arguments.length==0) {
        paramTypes = null;
        param = null;
    } else {
        paramTypes = new Class[1];
        paramTypes[0] = arguments.getClass();
        param = new Object[1];
        param[0] = arguments;
    }
    Method method =
        catalinaDaemon.getClass().getMethod(methodName, paramTypes);
    if (log.isDebugEnabled())
        log.debug("Calling startup class " + method);
    //通过反射调用Catalina的load()方法
    method.invoke(catalinaDaemon, param);

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### Catalina.load

我们可以看到daemon.load(args)实际上就是通过反射调用Catalina的load()方法.那么我们进入 Catalina 类的 load 方法看看:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void load() {

    initDirs();

    // 初始化jmx的环境变量
    initNaming();

    // Create and execute our Digester
    // 定义解析server.xml的配置，告诉Digester哪个xml标签应该解析成什么类
    Digester digester = createStartDigester();

    InputSource inputSource = null;
    InputStream inputStream = null;
    File file = null;
    try {

      // 首先尝试加载conf/server.xml，省略部分代码......
      // 如果不存在conf/server.xml，则加载server-embed.xml(该xml在catalina.jar中)，省略部分代码......
      // 如果还是加载不到xml，则直接return，省略部分代码......

      try {
          inputSource.setByteStream(inputStream);

          // 把Catalina作为一个顶级实例
          digester.push(this);

          // 解析过程会实例化各个组件，比如Server、Container、Connector等
          digester.parse(inputSource);
      } catch (SAXParseException spe) {
          // 处理异常......
      }
    } finally {
        // 关闭IO流......
    }

    // 给Server设置catalina信息
    getServer().setCatalina(this);
    getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
    getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());

    // Stream redirection
    initStreams();

    // 调用Lifecycle的init阶段
    try {
        getServer().init();
    } catch (LifecycleException e) {
        // ......
    }

    // ......

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### Server初始化

可以看到, 这里有一个我们今天感兴趣的方法, getServer.init(), 这个方法看名字是启动 Server 的初始化, 而 Server 是我们上面图中最外层的容器. 因此, 我们去看看该方法, 也就是LifecycleBase.init() 方法. 该方法是一个模板方法, 只是定义了一个算法的骨架, 将一些细节算法放在子类中去实现.我们看看该方法:

**LifecycleBase.init()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final synchronized void init() throws LifecycleException {
    // 1
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }
    // 2
    setStateInternal(LifecycleState.INITIALIZING, null, false);

    try {
        // 模板方法
        /**
         * 采用模板方法模式来对所有支持生命周期管理的组件的生命周期各个阶段进行了总体管理，
         * 每个需要生命周期管理的组件只需要继承这个基类，
         * 然后覆盖对应的钩子方法即可完成相应的声明周期阶段的管理工作
         */
        initInternal();
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.initFail",toString()), t);
    }

    // 3
    setStateInternal(LifecycleState.INITIALIZED, null, false);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`Server`的实现类为`StandardServer`，我们分析一下**`StandardServer.initInternal()`**方法。该方法用于对`Server`进行初始化，关键的地方就是代码最后对services的循环操作，对每个service调用init方法。
【注】：这儿我们只粘贴出这部分代码。

**StandardServer.initInternal()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void initInternal() throws LifecycleException {
    super.initInternal();

    // Initialize our defined Services
    for (int i = 0; i < services.length; i++) {
        services[i].init();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

调用Service子容器的init方法，让Service组件完成初始化，注意：在同一个Server下面，可能存在多个Service组件.



### Service初始化

StandardService和StandardServer都是继承至LifecycleMBeanBase，因此公共的初始化逻辑都是一样的，这里不做过多介绍，我们直接看下initInternal

***\*StandardService.\**initInternal()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void initInternal() throws LifecycleException {

    // 往jmx中注册自己
    super.initInternal();

    // 初始化Engine
    if (engine != null) {
        engine.init();
    }

    // 存在Executor线程池，则进行初始化，默认是没有的
    for (Executor executor : findExecutors()) {
        if (executor instanceof JmxEnabled) {
            ((JmxEnabled) executor).setDomain(getDomain());
        }
        executor.init();
    }

    mapperListener.init();

    // 初始化Connector，而Connector又会对ProtocolHandler进行初始化，开启应用端口的监听,
    synchronized (connectorsLock) {
        for (Connector connector : connectors) {
            try {
                connector.init();
            } catch (Exception e) {
                // 省略部分代码，logger and throw exception
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. 首先，往jmx中注册StandardService
2. 初始化Engine，而Engine初始化过程中会去初始化Realm(权限相关的组件)
3. 如果存在Executor线程池，还会进行init操作，这个Excecutor是tomcat的接口，继承至java.util.concurrent.Executor、org.apache.catalina.Lifecycle
4. 初始化Connector连接器，默认有http1.1、ajp连接器，而这个Connector初始化过程，又会对ProtocolHandler进行初始化，开启应用端口的监听，后面会详细分析



### Engine初始化

StandardEngine初始化的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void initInternal() throws LifecycleException {
    getRealm();
    super.initInternal();
}

public Realm getRealm() {
    Realm configured = super.getRealm();
    if (configured == null) {
        configured = new NullRealm();
        this.setRealm(configured);
    }
    return configured;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

StandardEngine继承至ContainerBase，而ContainerBase重写了initInternal()方法，用于初始化start、stop线程池，这个线程池有以下特点：
\1. core线程和max是相等的，默认为1
\2. 允许core线程在超时未获取到任务时退出线程
\3. 线程获取任务的超时时间是10s，也就是说所有的线程(包括core线程)，超过10s未获取到任务，那么这个线程就会被销毁

这么做的初衷是什么呢？因为这个线程池只需要在容器启动和停止的时候发挥作用，没必要时时刻刻处理任务队列

ContainerBase的代码如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 默认是1个线程
private int startStopThreads = 1;
protected ThreadPoolExecutor startStopExecutor;

@Override
protected void initInternal() throws LifecycleException {
    BlockingQueue<Runnable> startStopQueue = new LinkedBlockingQueue<>();
    startStopExecutor = new ThreadPoolExecutor(
            getStartStopThreadsInternal(),
            getStartStopThreadsInternal(), 10, TimeUnit.SECONDS,
            startStopQueue,
            new StartStopThreadFactory(getName() + "-startStop-"));
    // 允许core线程超时未获取任务时退出
    startStopExecutor.allowCoreThreadTimeOut(true);
    super.initInternal();
}

private int getStartStopThreadsInternal() {
    int result = getStartStopThreads();

    if (result > 0) {
        return result;
    }
    result = Runtime.getRuntime().availableProcessors() + result;
    if (result < 1) {
        result = 1;
    }
    return result;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个startStopExecutor线程池有什么用呢？

1. 在start的时候，如果发现有子容器，则会把子容器的start操作放在线程池中进行处理
2. 在stop的时候，也会把stop操作放在线程池中处理

在前面的文章中我们介绍了Container组件，StandardEngine作为顶层容器，它的直接子容器是StardandHost，但是对StandardEngine的代码分析，我们并没有发现它会对子容器StardandHost进行初始化操作，StandardEngine不按照套路出牌，而是把初始化过程放在start阶段。个人认为Host、Context、Wrapper这些容器和具体的webapp应用相关联了，初始化过程会更加耗时，因此在start阶段用多线程完成初始化以及start生命周期，否则，像顶层的Server、Service等组件需要等待Host、Context、Wrapper完成初始化才能结束初始化流程，整个初始化过程是具有传递性的



### Connector初始化

Connector初始化会在后面有专门的Connector文章讲解

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344968.html#_labelTop)

## 总结

至此，整个初始化过程便告一段落。整个初始化过程，由parent组件控制child组件的初始化，一层层往下传递，直到最后全部初始化OK。下图描述了整体的传递流程 

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190813115051025-679728263.png)

默认情况下，Server只有一个Service组件，Service组件先后对Engine、Connector进行初始化。而Engine组件并不会在初始化阶段对子容器进行初始化，Host、Context、Wrapper容器的初始化是在start阶段完成的。tomcat默认会启用HTTP1.1和AJP的Connector连接器，这两种协议默认使用Http11NioProtocol、AJPNioProtocol进行处理

# [Tomcat源码分析 （七）----- Tomcat 启动过程(二)](https://www.cnblogs.com/java-chen-hao/p/11344993.html)

**正文**

在上一篇文章中，我们分析了tomcat的初始化过程，是由Bootstrap反射调用Catalina的load方法完成tomcat的初始化，包括server.xml的解析、实例化各大组件、初始化组件等逻辑。那么tomcat又是如何启动webapp应用，又是如何加载应用程序的ServletContextListener，以及Servlet呢？我们将在这篇文章进行分析

我们先来看下整体的启动逻辑，tomcat由上往下，挨个启动各个组件： 

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190813114654656-549909215.png)

 我们接着上一篇文章来分析，上一篇文章我们分析完了Catalina.load(),这篇文章来看看daemon.start();

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Bootstrap



### daemon.start()

启动过程和初始化一样，由Bootstrap反射调用Catalina的start方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void start()
    throws Exception {
    if( catalinaDaemon==null ) init();

    Method method = catalinaDaemon.getClass().getMethod("start", (Class [] )null);
    method.invoke(catalinaDaemon, (Object [])null);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Catalina

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void start() {

    if (getServer() == null) {
        load();
    }

    if (getServer() == null) {
        log.fatal("Cannot start server. Server instance is not configured.");
        return;
    }

    long t1 = System.nanoTime();

    // Start the new server
    try {
        //调用Server的start方法，启动Server组件 
        getServer().start();
    } catch (LifecycleException e) {
        log.fatal(sm.getString("catalina.serverStartFail"), e);
        try {
            getServer().destroy();
        } catch (LifecycleException e1) {
            log.debug("destroy() failed for failed Server ", e1);
        }
        return;
    }

    long t2 = System.nanoTime();
    if(log.isInfoEnabled()) {
        log.info("Server startup in " + ((t2 - t1) / 1000000) + " ms");
    }

    // Register shutdown hook
    // 注册勾子，用于安全关闭tomcat
    if (useShutdownHook) {
        if (shutdownHook == null) {
            shutdownHook = new CatalinaShutdownHook();
        }
        Runtime.getRuntime().addShutdownHook(shutdownHook);

        // If JULI is being used, disable JULI's shutdown hook since
        // shutdown hooks run in parallel and log messages may be lost
        // if JULI's hook completes before the CatalinaShutdownHook()
        LogManager logManager = LogManager.getLogManager();
        if (logManager instanceof ClassLoaderLogManager) {
            ((ClassLoaderLogManager) logManager).setUseShutdownHook(
                    false);
        }
    }

    // Bootstrap中会设置await为true，其目的在于让tomcat在shutdown端口阻塞监听关闭命令
    if (await) {
        await();
        stop();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Server

在前面的Lifecycle文章中，我们介绍了StandardServer重写了startInternal方法，完成自己的逻辑



### StandardServer.startInternal

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void startInternal() throws LifecycleException {

    fireLifecycleEvent(CONFIGURE_START_EVENT, null);
    setState(LifecycleState.STARTING);

    globalNamingResources.start();

    // Start our defined Services
    synchronized (servicesLock) {
        for (int i = 0; i < services.length; i++) {
            services[i].start();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先是由LifecycleBase统一发出STARTING_PREP事件，StandardServer额外还会发出CONFIGURE_START_EVENT、STARTING事件，用于通知LifecycleListener在启动前做一些准备工作，比如NamingContextListener会处理CONFIGURE_START_EVENT事件，实例化tomcat相关的上下文，以及ContextResource资源

接着，启动Service组件，这一块的逻辑将在下面进行详细分析，最后由LifecycleBase发出STARTED事件，完成start

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Service

StandardService的start代码如下所示：
\1. 启动Engine，Engine的child容器都会被启动，webapp的部署会在这个步骤完成；
\2. 启动Executor，这是tomcat用Lifecycle封装的线程池，继承至java.util.concurrent.Executor以及tomcat的Lifecycle接口
\3. 启动Connector组件，由Connector完成Endpoint的启动，这个时候意味着tomcat可以对外提供请求服务了



### StandardService.startInternal

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void startInternal() throws LifecycleException {

    setState(LifecycleState.STARTING);

    // 启动Engine
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }

    // 启动Executor线程池
    synchronized (executors) {
        for (Executor executor: executors) {
            executor.start();
        }
    }

    // 启动MapperListener
    mapperListener.start();

    // 启动Connector
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            try {
                // If it has already failed, don't try and start it
                if (connector.getState() != LifecycleState.FAILED) {
                    connector.start();
                }
            } catch (Exception e) {
                // logger......
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Engine

Engine的标准实现为`org.apache.catalina.core.StandardEngine`。我们先来看看构造函数。其主要职责为：使用默认的基础阀门创建标准Engine组件。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * Create a new StandardEngine component with the default basic Valve.
 */
public StandardEngine() {
    super();
    pipeline.setBasic(new StandardEngineValve());
    /* Set the jmvRoute using the system property jvmRoute */
    try {
        setJvmRoute(System.getProperty("jvmRoute"));
    } catch(Exception ex) {
        log.warn(sm.getString("standardEngine.jvmRouteFail"));
    }
    // By default, the engine will hold the reloading thread
    backgroundProcessorDelay = 10;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看StandardEngine.startInternal

**StandardEngine.startInternal**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected synchronized void startInternal() throws LifecycleException {

    // Log our server identification information
    if(log.isInfoEnabled())
        log.info( "Starting Servlet Engine: " + ServerInfo.getServerInfo());

    // Standard container startup
    super.startInternal();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

StandardEngine、StandardHost、StandardContext、StandardWrapper各个容器存在父子关系，一个父容器包含多个子容器，并且一个子容器对应一个父容器。Engine是顶层父容器，它不存在父容器。各个组件的包含关系如下图所示，默认情况下，StandardEngine只有一个子容器StandardHost，一个StandardContext对应一个webapp应用，而一个StandardWrapper对应一个webapp里面的一个 Servlet

**![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190813153846385-434701125.png)**

StandardEngine、StandardHost、StandardContext、StandardWrapper都是继承至ContainerBase，各个容器的启动，都是由父容器调用子容器的start方法，也就是说由StandardEngine启动StandardHost，再StandardHost启动StandardContext，以此类推。

由于它们都是继续至ContainerBase，当调用 start 启动Container容器时，首先会执行 ContainerBase 的 start 方法，它会寻找子容器，并且在线程池中启动子容器，StandardEngine也不例外。



### ContainerBase

ContainerBase的startInternal方法如下所示，主要分为以下3个步骤：
\1. 启动子容器
\2. 启动Pipeline，并且发出STARTING事件
\3. 如果backgroundProcessorDelay参数 >= 0，则开启ContainerBackgroundProcessor线程，用于调用子容器的backgroundProcess

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected synchronized void startInternal() throws LifecycleException {
    // 省略若干代码......

    // 把子容器的启动步骤放在线程中处理，默认情况下线程池只有一个线程处理任务队列
    Container children[] = findChildren();
    List<Future<Void>> results = new ArrayList<>();
    for (int i = 0; i < children.length; i++) {
        results.add(startStopExecutor.submit(new StartChild(children[i])));
    }

    // 阻塞当前线程，直到子容器start完成
    boolean fail = false;
    for (Future<Void> result : results) {
        try {
            result.get();
        } catch (Exception e) {
            log.error(sm.getString("containerBase.threadedStartFailed"), e);
            fail = true;
        }
    }

    // 启用Pipeline
    if (pipeline instanceof Lifecycle)
        ((Lifecycle) pipeline).start();
    setState(LifecycleState.STARTING);

    // 开启ContainerBackgroundProcessor线程用于调用子容器的backgroundProcess方法，默认情况下backgroundProcessorDelay=-1，不会启用该线程
    threadStart();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ContainerBase会把StartChild任务丢给线程池处理，得到Future，并且会遍历所有的Future进行阻塞result.get()，这个操作是将异步启动转同步，子容器启动完成才会继续运行。我们再来看看submit到线程池的StartChild任务，它实现了java.util.concurrent.Callable接口，在call里面完成子容器的start动作

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static class StartChild implements Callable<Void> {

    private Container child;

    public StartChild(Container child) {
        this.child = child;
    }

    @Override
    public Void call() throws LifecycleException {
        child.start();
        return null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 启动Pipeline

默认使用 StandardPipeline 实现类，它也是一个Lifecycle。在容器启动的时候，StandardPipeline 会遍历 Valve 链表，如果 Valve 是 Lifecycle 的子类，则会调用其 start 方法启动 Valve 组件，代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class StandardPipeline extends LifecycleBase
        implements Pipeline, Contained {

    // 省略若干代码......

    protected synchronized void startInternal() throws LifecycleException {

        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof Lifecycle)
                ((Lifecycle) current).start();
            current = current.getNext();
        }

        setState(LifecycleState.STARTING);
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Host

分析Host的时候，我们从Host的构造函数入手，该方法主要是设置基础阀门。

```
public StandardHost() {
    super();
    pipeline.setBasic(new StandardHostValve());
}
```

**StandardEngine.startInternal** 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected synchronized void startInternal() throws LifecycleException {

    // errorValve默认使用org.apache.catalina.valves.ErrorReportValve
    String errorValve = getErrorReportValveClass();
    if ((errorValve != null) && (!errorValve.equals(""))) {
        try {
            boolean found = false;

            // 如果所有的阀门中已经存在这个实例，则不进行处理，否则添加到  Pipeline 中
            Valve[] valves = getPipeline().getValves();
            for (Valve valve : valves) {
                if (errorValve.equals(valve.getClass().getName())) {
                    found = true;
                    break;
                }
            }

            // 如果未找到则添加到 Pipeline 中，注意是添加到 basic valve 的前面
            // 默认情况下，first valve 是 AccessLogValve，basic 是 StandardHostValve
            if(!found) {
                Valve valve =
                    (Valve) Class.forName(errorValve).getConstructor().newInstance();
                getPipeline().addValve(valve);
            }
        } catch (Throwable t) {
            // 处理异常，省略......
        }
    }

    // 调用父类 ContainerBase，完成统一的启动动作
    super.startInternal();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

StandardHost Pipeline 包含的 Valve 组件：
\1. basic：org.apache.catalina.core.StandardHostValve
\2. first：org.apache.catalina.valves.AccessLogValve

需要注意的是，在往 Pipeline 中添加 Valve 阀门时，是添加到 first 后面，basic 前面

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Context

接下来我们分析一下Context的实现`org.apache.catalina.core.StandardContext`。

先来看看构造方法，该方法用于设置`Context.pipeline`的基础阀门。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public StandardContext() {
    super();
    pipeline.setBasic(new StandardContextValve());
    broadcaster = new NotificationBroadcasterSupport();
    // Set defaults
    if (!Globals.STRICT_SERVLET_COMPLIANCE) {
        // Strict servlet compliance requires all extension mapped servlets
        // to be checked against welcome files
        resourceOnlyServlets.add("jsp");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

启动方法和上面的容器启动方法类似，我们就不再赘述了

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## Wrapper

Wrapper是一个Servlet的包装，我们先来看看构造方法。主要作用就是设置基础阀门`StandardWrapperValve`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public StandardWrapper() {
    super();
    swValve=new StandardWrapperValve();
    pipeline.setBasic(swValve);
    broadcaster = new NotificationBroadcasterSupport();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里每个容器中的pipeline设置的StandardEngineValve、StandardHostValve、StandardContextValve、StandardWrapperValve是有大用处的，后面我们会在Http请求过程中详细讲解。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11344993.html#_labelTop)

## 总结

至此，整个启动过程便告一段落。整个启动过程程，由parent组件控制child组件的启动，一层层往下传递，直到最后全部启动完成。

# [Tomcat源码分析 （八）----- HTTP请求处理过程（一）](https://www.cnblogs.com/java-chen-hao/p/11305207.html)

**正文**

终于进行到`Connector`的分析阶段了，这也是Tomcat里面最复杂的一块功能了。`Connector`中文名为`连接器`，既然是连接器，它肯定会连接某些东西，连接些什么呢？

> `Connector`用于接受请求并将请求封装成Request和Response，然后交给`Container`进行处理，`Container`处理完之后再交给`Connector`返回给客户端。

要理解`Connector`，我们需要问自己4个问题。

- （1）`Connector`如何接受请求的？
- （2）如何将请求封装成Request和Response的？
- （3）封装完之后的Request和Response如何交给`Container`进行处理的？
- （4）`Container`处理完之后如何交给`Connector`并返回给客户端的？

先来一张`Connector`的整体结构图

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190805194836476-1284200425.png)

【注意】：不同的协议、不同的通信方式，`ProtocolHandler`会有不同的实现。在Tomcat8.5中，`ProtocolHandler`的类继承层级如下图所示。

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190805194923329-458434509.png)

 

针对上述的类继承层级图，我们做如下说明：

1. ajp和http11是两种不同的协议
2. nio、nio2和apr是不同的通信方式
3. 协议和通信方式可以相互组合。

`ProtocolHandler`包含三个部件：`Endpoint`、`Processor`、`Adapter`。

1. `Endpoint`用来处理底层Socket的网络连接，`Processor`用于将`Endpoint`接收到的Socket封装成Request，`Adapter`用于将Request交给Container进行具体的处理。
2. `Endpoint`由于是处理底层的Socket网络连接，因此`Endpoint`是用来实现`TCP/IP协议`的，而`Processor`用来实现`HTTP协议`的，`Adapter`将请求适配到Servlet容器进行具体的处理。
3. `Endpoint`的抽象实现类AbstractEndpoint里面定义了`Acceptor`和`AsyncTimeout`两个内部类和一个`Handler接口`。`Acceptor`用于监听请求，`AsyncTimeout`用于检查异步Request的超时，`Handler`用于处理接收到的Socket，在内部调用`Processor`进行处理。

至此，我们已经明白了问题（1）、（2）和（3）。至于（4），当我们了解了Container自然就明白了，前面章节内容已经详细分析过了。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11305207.html#_labelTop)

## Connector源码分析入口

 我们在`Service`标准实现`StandardService`的源码中发现，其`init()`、`start()`、`stop()`和`destroy()`方法分别会对Connectors的同名方法进行调用。而一个`Service`对应着多个`Connector`。

### Service.init()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void initInternal() throws LifecycleException {
    super.initInternal();

    if (engine != null) {
        engine.init();
    }

    // Initialize any Executors
    for (Executor executor : findExecutors()) {
        if (executor instanceof JmxEnabled) {
            ((JmxEnabled) executor).setDomain(getDomain());
        }
        executor.init();
    }

    // Initialize mapper listener
    mapperListener.init();

    // Initialize our defined Connectors
    synchronized (connectorsLock) {
        for (Connector connector : connectors) {
            try {
                connector.init();
            } catch (Exception e) {
                String message = sm.getString(
                        "standardService.connector.initFailed", connector);
                log.error(message, e);

                if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"))
                    throw new LifecycleException(message);
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### Service.start()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void startInternal() throws LifecycleException {
    if(log.isInfoEnabled())
        log.info(sm.getString("standardService.start.name", this.name));
    setState(LifecycleState.STARTING);

    // Start our defined Container first
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }

    synchronized (executors) {
        for (Executor executor: executors) {
            executor.start();
        }
    }

    mapperListener.start();

    // Start our defined Connectors second
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            try {
                // If it has already failed, don't try and start it
                if (connector.getState() != LifecycleState.FAILED) {
                    connector.start();
                }
            } catch (Exception e) {
                log.error(sm.getString(
                        "standardService.connector.startFailed",
                        connector), e);
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们知道`Connector`实现了`Lifecycle`接口，所以它是一个`生命周期组件`。所以`Connector`的启动逻辑入口在于`init()`和`start()`。

### Connector构造方法

在分析之前，我们看看`server.xml`，该文件已经体现出了tomcat中各个组件的大体结构。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在这个文件中，我们看到一个`Connector`有几个关键属性，`port`和`protocol`是其中的两个。`server.xml`默认支持两种协议：`HTTP/1.1`和`AJP/1.3`。其中`HTTP/1.1`用于支持http1.1协议，而`AJP/1.3`用于支持对apache服务器的通信。

接下来我们看看构造方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Connector() {
    this(null); // 1. 无参构造方法，传入参数为空协议，会默认使用`HTTP/1.1`
}

public Connector(String protocol) {
    setProtocol(protocol);
    // Instantiate protocol handler
    // 5. 使用protocolHandler的类名构造ProtocolHandler的实例
    ProtocolHandler p = null;
    try {
        Class<?> clazz = Class.forName(protocolHandlerClassName);
        p = (ProtocolHandler) clazz.getConstructor().newInstance();
    } catch (Exception e) {
        log.error(sm.getString(
                "coyoteConnector.protocolHandlerInstantiationFailed"), e);
    } finally {
        this.protocolHandler = p;
    }

    if (Globals.STRICT_SERVLET_COMPLIANCE) {
        uriCharset = StandardCharsets.ISO_8859_1;
    } else {
        uriCharset = StandardCharsets.UTF_8;
    }
}

@Deprecated
public void setProtocol(String protocol) {
    boolean aprConnector = AprLifecycleListener.isAprAvailable() &&
            AprLifecycleListener.getUseAprConnector();

    // 2. `HTTP/1.1`或`null`，protocolHandler使用`org.apache.coyote.http11.Http11NioProtocol`，不考虑apr
    if ("HTTP/1.1".equals(protocol) || protocol == null) {
        if (aprConnector) {
            setProtocolHandlerClassName("org.apache.coyote.http11.Http11AprProtocol");
        } else {
            setProtocolHandlerClassName("org.apache.coyote.http11.Http11NioProtocol");
        }
    }
    // 3. `AJP/1.3`，protocolHandler使用`org.apache.coyote.ajp.AjpNioProtocol`，不考虑apr
    else if ("AJP/1.3".equals(protocol)) {
        if (aprConnector) {
            setProtocolHandlerClassName("org.apache.coyote.ajp.AjpAprProtocol");
        } else {
            setProtocolHandlerClassName("org.apache.coyote.ajp.AjpNioProtocol");
        }
    }
    // 4. 其他情况，使用传入的protocol作为protocolHandler的类名
    else {
        setProtocolHandlerClassName(protocol);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从上面的代码我们看到构造方法主要做了下面几件事情：

1. 无参构造方法，传入参数为空协议，会默认使用`HTTP/1.1`
2. `HTTP/1.1`或`null`，protocolHandler使用`org.apache.coyote.http11.Http11NioProtocol`，不考虑apr
3. `AJP/1.3`，protocolHandler使用`org.apache.coyote.ajp.AjpNioProtocol`，不考虑apr
4. 其他情况，使用传入的protocol作为protocolHandler的类名
5. 使用protocolHandler的类名构造ProtocolHandler的实例

### Connector.init()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void initInternal() throws LifecycleException {
    super.initInternal();

    // Initialize adapter
    // 1. 初始化adapter
    adapter = new CoyoteAdapter(this);
    protocolHandler.setAdapter(adapter);

    // Make sure parseBodyMethodsSet has a default
    // 2. 设置接受body的method列表，默认为POST
    if (null == parseBodyMethodsSet) {
        setParseBodyMethods(getParseBodyMethods());
    }

    if (protocolHandler.isAprRequired() && !AprLifecycleListener.isAprAvailable()) {
        throw new LifecycleException(sm.getString("coyoteConnector.protocolHandlerNoApr",
                getProtocolHandlerClassName()));
    }
    if (AprLifecycleListener.isAprAvailable() && AprLifecycleListener.getUseOpenSSL() &&
            protocolHandler instanceof AbstractHttp11JsseProtocol) {
        AbstractHttp11JsseProtocol<?> jsseProtocolHandler =
                (AbstractHttp11JsseProtocol<?>) protocolHandler;
        if (jsseProtocolHandler.isSSLEnabled() &&
                jsseProtocolHandler.getSslImplementationName() == null) {
            // OpenSSL is compatible with the JSSE configuration, so use it if APR is available
            jsseProtocolHandler.setSslImplementationName(OpenSSLImplementation.class.getName());
        }
    }

    // 3. 初始化protocolHandler
    try {
        protocolHandler.init();
    } catch (Exception e) {
        throw new LifecycleException(
                sm.getString("coyoteConnector.protocolHandlerInitializationFailed"), e);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`init()`方法做了3件事情

1. 初始化adapter
2. 设置接受body的method列表，默认为POST
3. 初始化protocolHandler

从`ProtocolHandler类继承层级`我们知道`ProtocolHandler`的子类都必须实现`AbstractProtocol`抽象类，而`protocolHandler.init();`的逻辑代码正是在这个抽象类里面。我们来分析一下。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void init() throws Exception {
    if (getLog().isInfoEnabled()) {
        getLog().info(sm.getString("abstractProtocolHandler.init", getName()));
    }

    if (oname == null) {
        // Component not pre-registered so register it
        oname = createObjectName();
        if (oname != null) {
            Registry.getRegistry(null, null).registerComponent(this, oname, null);
        }
    }

    if (this.domain != null) {
        rgOname = new ObjectName(domain + ":type=GlobalRequestProcessor,name=" + getName());
        Registry.getRegistry(null, null).registerComponent(
                getHandler().getGlobal(), rgOname, null);
    }

    // 1. 设置endpoint的名字，默认为：http-nio-{port}
    String endpointName = getName();
    endpoint.setName(endpointName.substring(1, endpointName.length()-1));
    endpoint.setDomain(domain);
    
    // 2. 初始化endpoint
    endpoint.init();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们接着分析一下`Endpoint.init()`里面又做了什么。该方法位于`AbstactEndpoint`抽象类，该类是基于模板方法模式实现的，主要调用了子类的`bind()`方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract void bind() throws Exception;
public abstract void unbind() throws Exception;
public abstract void startInternal() throws Exception;
public abstract void stopInternal() throws Exception;

public void init() throws Exception {
    // 执行bind()方法
    if (bindOnInit) {
        bind();
        bindState = BindState.BOUND_ON_INIT;
    }
    if (this.domain != null) {
        // Register endpoint (as ThreadPool - historical name)
        oname = new ObjectName(domain + ":type=ThreadPool,name=\"" + getName() + "\"");
        Registry.getRegistry(null, null).registerComponent(this, oname, null);

        ObjectName socketPropertiesOname = new ObjectName(domain +
                ":type=ThreadPool,name=\"" + getName() + "\",subType=SocketProperties");
        socketProperties.setObjectName(socketPropertiesOname);
        Registry.getRegistry(null, null).registerComponent(socketProperties, socketPropertiesOname, null);

        for (SSLHostConfig sslHostConfig : findSslHostConfigs()) {
            registerJmx(sslHostConfig);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

继续分析`bind()`方法，我们终于看到了我们想要看的东西了。关键的代码在于`serverSock.socket().bind(addr,getAcceptCount());`，用于绑定`ServerSocket`到指定的IP和端口。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void bind() throws Exception {

    if (!getUseInheritedChannel()) {
        serverSock = ServerSocketChannel.open();
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = (getAddress()!=null?new InetSocketAddress(getAddress(),getPort()):new InetSocketAddress(getPort()));
        //绑定ServerSocket到指定的IP和端口
        serverSock.socket().bind(addr,getAcceptCount());
    } else {
        // Retrieve the channel provided by the OS
        Channel ic = System.inheritedChannel();
        if (ic instanceof ServerSocketChannel) {
            serverSock = (ServerSocketChannel) ic;
        }
        if (serverSock == null) {
            throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
        }
    }

    serverSock.configureBlocking(true); //mimic APR behavior

    // Initialize thread count defaults for acceptor, poller
    if (acceptorThreadCount == 0) {
        // FIXME: Doesn't seem to work that well with multiple accept threads
        acceptorThreadCount = 1;
    }
    if (pollerThreadCount <= 0) {
        //minimum one poller thread
        pollerThreadCount = 1;
    }
    setStopLatch(new CountDownLatch(pollerThreadCount));

    // Initialize SSL if needed
    initialiseSsl();

    selectorPool.open();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

好了，我们已经分析完了`init()`方法，接下来我们分析`start()`方法。关键代码就一行，调用`ProtocolHandler.start()`方法。

### Connector.start()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void startInternal() throws LifecycleException {

    // Validate settings before starting
    if (getPort() < 0) {
        throw new LifecycleException(sm.getString(
                "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
    }

    setState(LifecycleState.STARTING);

    try {
        protocolHandler.start();
    } catch (Exception e) {
        throw new LifecycleException(
                sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们深入`ProtocolHandler.start()`方法。

1. 调用`Endpoint.start()`方法
2. 开启异步超时线程，线程执行单元为`Asynctimeout`

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void start() throws Exception {
    if (getLog().isInfoEnabled()) {
        getLog().info(sm.getString("abstractProtocolHandler.start", getName()));
    }

    // 1. 调用`Endpoint.start()`方法
    endpoint.start();

    // Start async timeout thread
    // 2. 开启异步超时线程，线程执行单元为`Asynctimeout`
    asyncTimeout = new AsyncTimeout();
    Thread timeoutThread = new Thread(asyncTimeout, getNameInternal() + "-AsyncTimeout");
    int priority = endpoint.getThreadPriority();
    if (priority < Thread.MIN_PRIORITY || priority > Thread.MAX_PRIORITY) {
        priority = Thread.NORM_PRIORITY;
    }
    timeoutThread.setPriority(priority);
    timeoutThread.setDaemon(true);
    timeoutThread.start();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这儿我们重点关注`Endpoint.start()`方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final void start() throws Exception {
    // 1. `bind()`已经在`init()`中分析过了
    if (bindState == BindState.UNBOUND) {
        bind();
        bindState = BindState.BOUND_ON_START;
    }
    startInternal();
}

@Override
public void startInternal() throws Exception {
    if (!running) {
        running = true;
        paused = false;

        processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                socketProperties.getProcessorCache());
        eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                        socketProperties.getEventCache());
        nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                socketProperties.getBufferPool());

        // Create worker collection
        // 2. 创建工作者线程池
        if ( getExecutor() == null ) {
            createExecutor();
        }
        
        // 3. 初始化连接latch，用于限制请求的并发量
        initializeConnectionLatch();

        // Start poller threads
        // 4. 开启poller线程。poller用于对接受者线程生产的消息（或事件）进行处理，poller最终调用的是Handler的代码
        pollers = new Poller[getPollerThreadCount()];
        for (int i=0; i<pollers.length; i++) {
            pollers[i] = new Poller();
            Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
            pollerThread.setPriority(threadPriority);
            pollerThread.setDaemon(true);
            pollerThread.start();
        }
        // 5. 开启acceptor线程
        startAcceptorThreads();
    }
}

protected final void startAcceptorThreads() {
    int count = getAcceptorThreadCount();
    acceptors = new Acceptor[count];

    for (int i = 0; i < count; i++) {
        acceptors[i] = createAcceptor();
        String threadName = getName() + "-Acceptor-" + i;
        acceptors[i].setThreadName(threadName);
        Thread t = new Thread(acceptors[i], threadName);
        t.setPriority(getAcceptorThreadPriority());
        t.setDaemon(getDaemon());
        t.start();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. `bind()`已经在`init()`中分析过了
2. 创建工作者线程池
3. 初始化连接latch，用于限制请求的并发量
4. 创建轮询Poller线程。poller用于对接受者线程生产的消息（或事件）进行处理，poller最终调用的是Handler的代码
5. 创建Acceptor线程

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11305207.html#_labelTop)

## Connector请求逻辑

分析完了`Connector`的启动逻辑之后，我们就需要进一步分析一下http的请求逻辑，当请求从客户端发起之后，需要经过哪些操作才能真正地得到执行？



### Acceptor

Acceptor线程主要用于监听套接字，将已连接套接字转给Poller线程。Acceptor线程数由AbstracEndPoint的acceptorThreadCount成员变量控制，默认值为1

AbstractEndpoint.Acceptor是AbstractEndpoint类的静态抽象类，实现了Runnable接口，部分代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract static class Acceptor implements Runnable {
    public enum AcceptorState {
        NEW, RUNNING, PAUSED, ENDED
    }

    protected volatile AcceptorState state = AcceptorState.NEW;
    public final AcceptorState getState() {
        return state;
    }

    private String threadName;
    protected final void setThreadName(final String threadName) {
        this.threadName = threadName;
    }
    protected final String getThreadName() {
        return threadName;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

NioEndpoint的Acceptor成员内部类继承了AbstractEndpoint.Acceptor：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected class Acceptor extends AbstractEndpoint.Acceptor {
    @Override
    public void run() {
        int errorDelay = 0;

        // Loop until we receive a shutdown command
        while (running) {

            // Loop if endpoint is paused
            // 1. 运行过程中，如果`Endpoint`暂停了，则`Acceptor`进行自旋（间隔50毫秒） `       
            while (paused && running) {
                state = AcceptorState.PAUSED;
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    // Ignore
                }
            }
            // 2. 如果`Endpoint`终止运行了，则`Acceptor`也会终止
            if (!running) {
                break;
            }
            state = AcceptorState.RUNNING;

            try {
                //if we have reached max connections, wait
                // 3. 如果请求达到了最大连接数，则wait直到连接数降下来
                countUpOrAwaitConnection();

                SocketChannel socket = null;
                try {
                    // Accept the next incoming connection from the server
                    // socket
                    // 4. 接受下一次连接的socket
                    socket = serverSock.accept();
                } catch (IOException ioe) {
                    // We didn't get a socket
                    countDownConnection();
                    if (running) {
                        // Introduce delay if necessary
                        errorDelay = handleExceptionWithDelay(errorDelay);
                        // re-throw
                        throw ioe;
                    } else {
                        break;
                    }
                }
                // Successful accept, reset the error delay
                errorDelay = 0;

                // Configure the socket
                if (running && !paused) {
                    // setSocketOptions() will hand the socket off to
                    // an appropriate processor if successful
                    // 5. `setSocketOptions()`这儿是关键，会将socket以事件的方式传递给poller
                    if (!setSocketOptions(socket)) {
                        closeSocket(socket);
                    }
                } else {
                    closeSocket(socket);
                }
            } catch (Throwable t) {
                ExceptionUtils.handleThrowable(t);
                log.error(sm.getString("endpoint.accept.fail"), t);
            }
        }
        state = AcceptorState.ENDED;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从以上代码可以看到：

- countUpOrAwaitConnection函数检查当前最大连接数，若未达到maxConnections则加一，否则等待；
- socket = serverSock.accept()这一行中的serverSock正是NioEndpoint的bind函数中打开的ServerSocketChannel。为了引用这个变量，NioEndpoint的Acceptor类是成员而不再是静态类；
- setSocketOptions函数调用上的注释表明该函数将已连接套接字交给Poller线程处理。

setSocketOptions方法接着处理已连接套接字：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected boolean setSocketOptions(SocketChannel socket) {
    // Process the connection
    try {
        //disable blocking, APR style, we are gonna be polling it
        socket.configureBlocking(false);
        Socket sock = socket.socket();
        socketProperties.setProperties(sock);

        NioChannel channel = nioChannels.pop();
        if (channel == null) {
            SocketBufferHandler bufhandler = new SocketBufferHandler(
                    socketProperties.getAppReadBufSize(),
                    socketProperties.getAppWriteBufSize(),
                    socketProperties.getDirectBuffer());
            if (isSSLEnabled()) {
                channel = new SecureNioChannel(socket, bufhandler, selectorPool, this);
            } else {
                channel = new NioChannel(socket, bufhandler);
            }
        } else {
            channel.setIOChannel(socket);
            channel.reset();
        }
        // 将channel注册到poller，注意关键的两个方法，`getPoller0()`和`Poller.register()`
        getPoller0().register(channel);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        try {
            log.error("",t);
        } catch (Throwable tt) {
            ExceptionUtils.handleThrowable(tt);
        }
        // Tell to close the socket
        return false;
    }
    return true;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 从NioChannel栈中出栈一个，若能重用（即不为null）则重用对象，否则新建一个NioChannel对象；
- getPoller0方法利用轮转法选择一个Poller线程，利用Poller类的register方法将上述NioChannel对象注册到该Poller线程上；
- 若成功转给Poller线程该函数返回true，否则返回false。返回false后，Acceptor类的closeSocket函数会关闭通道和底层Socket连接并将当前最大连接数减一。

### Poller

Poller线程主要用于以较少的资源轮询已连接套接字以保持连接，当数据可用时转给工作线程。

Poller线程数由NioEndPoint的pollerThreadCount成员变量控制，默认值为2与可用处理器数二者之间的较小值。
Poller实现了Runnable接口，可以看到构造函数为每个Poller打开了一个新的Selector。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Poller implements Runnable {
    private Selector selector;
    private final SynchronizedQueue<PollerEvent> events =
            new SynchronizedQueue<>();
    // 省略一些代码
    public Poller() throws IOException {
        this.selector = Selector.open();
    }

    public Selector getSelector() { return selector;}
    // 省略一些代码
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

将channel注册到poller，注意关键的两个方法，`getPoller0()`和`Poller.register()`。先来分析一下`getPoller0()`，该方法比较关键的一个地方就是`以取模的方式`对poller数量进行轮询获取。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * The socket poller.
 */
private Poller[] pollers = null;
private AtomicInteger pollerRotater = new AtomicInteger(0);
/**
 * Return an available poller in true round robin fashion.
 *
 * @return The next poller in sequence
 */
public Poller getPoller0() {
    int idx = Math.abs(pollerRotater.incrementAndGet()) % pollers.length;
    return pollers[idx];
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接下来我们分析一下`Poller.register()`方法。因为`Poller`维持了一个`events同步队列`，所以`Acceptor`接受到的channel会放在这个队列里面，放置的代码为`events.offer(event);`

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Poller implements Runnable {

    private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

    /**
     * Registers a newly created socket with the poller.
     *
     * @param socket    The newly created socket
     */
    public void register(final NioChannel socket) {
        socket.setPoller(this);
        NioSocketWrapper ka = new NioSocketWrapper(socket, NioEndpoint.this);
        socket.setSocketWrapper(ka);
        ka.setPoller(this);
        ka.setReadTimeout(getSocketProperties().getSoTimeout());
        ka.setWriteTimeout(getSocketProperties().getSoTimeout());
        ka.setKeepAliveLeft(NioEndpoint.this.getMaxKeepAliveRequests());
        ka.setSecure(isSSLEnabled());
        ka.setReadTimeout(getConnectionTimeout());
        ka.setWriteTimeout(getConnectionTimeout());
        PollerEvent r = eventCache.pop();
        ka.interestOps(SelectionKey.OP_READ);//this is what OP_REGISTER turns into.
        if ( r==null) r = new PollerEvent(socket,ka,OP_REGISTER);
        else r.reset(socket,ka,OP_REGISTER);
        addEvent(r);
    }

    private void addEvent(PollerEvent event) {
        events.offer(event);
        if ( wakeupCounter.incrementAndGet() == 0 ) selector.wakeup();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### PollerEvent

接下来看一下PollerEvent，PollerEvent实现了Runnable接口，用来表示一个轮询事件，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static class PollerEvent implements Runnable {
    private NioChannel socket;
    private int interestOps;
    private NioSocketWrapper socketWrapper;

    public PollerEvent(NioChannel ch, NioSocketWrapper w, int intOps) {
        reset(ch, w, intOps);
    }

    public void reset(NioChannel ch, NioSocketWrapper w, int intOps) {
        socket = ch;
        interestOps = intOps;
        socketWrapper = w;
    }

    public void reset() {
        reset(null, null, 0);
    }

    @Override
    public void run() {
        if (interestOps == OP_REGISTER) {
            try {
                socket.getIOChannel().register(
                        socket.getPoller().getSelector(), SelectionKey.OP_READ, socketWrapper);
            } catch (Exception x) {
                log.error(sm.getString("endpoint.nio.registerFail"), x);
            }
        } else {
            final SelectionKey key = socket.getIOChannel().keyFor(socket.getPoller().getSelector());
            try {
                if (key == null) {
                    socket.socketWrapper.getEndpoint().countDownConnection();
                    ((NioSocketWrapper) socket.socketWrapper).closed = true;
                } else {
                    final NioSocketWrapper socketWrapper = (NioSocketWrapper) key.attachment();
                    if (socketWrapper != null) {
                        //we are registering the key to start with, reset the fairness counter.
                        int ops = key.interestOps() | interestOps;
                        socketWrapper.interestOps(ops);
                        key.interestOps(ops);
                    } else {
                        socket.getPoller().cancelledKey(key);
                    }
                }
            } catch (CancelledKeyException ckx) {
                try {
                    socket.getPoller().cancelledKey(key);
                } catch (Exception ignore) {}
            }
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在run函数中：

- 若感兴趣集是自定义的OP_REGISTER，则说明该事件表示的已连接套接字通道尚未被轮询线程处理过，那么将该通道注册到Poller线程的Selector上，感兴趣集是OP_READ，通道注册的附件是一个NioSocketWrapper对象。从Poller的register方法添加事件即是这样的过程；
- 否则获得已连接套接字通道注册到Poller线程的Selector上的SelectionKey，为key添加新的感兴趣集。

### 重访Poller

上文提到Poller类实现了Runnable接口，其重写的run方法如下所示。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean events() {
    boolean result = false;
    PollerEvent pe = null;
    for (int i = 0, size = events.size(); i < size && (pe = events.poll()) != null; i++ ) {
        result = true;
        try {
            //直接调用run方法
            pe.run();
            pe.reset();
            if (running && !paused) {
                eventCache.push(pe);
            }
        } catch ( Throwable x ) {
            log.error("",x);
        }
    }
    return result;
}

@Override
public void run() {
    // Loop until destroy() is called
    while (true) {
        boolean hasEvents = false;

        try {
            if (!close) {
                /执行PollerEvent的run方法
                hasEvents = events();
                if (wakeupCounter.getAndSet(-1) > 0) {
                    //if we are here, means we have other stuff to do
                    //do a non blocking select
                    keyCount = selector.selectNow();
                } else {
                    keyCount = selector.select(selectorTimeout);
                }
                wakeupCounter.set(0);
            }
            if (close) {
                events();
                timeout(0, false);
                try {
                    selector.close();
                } catch (IOException ioe) {
                    log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                }
                break;
            }
        } catch (Throwable x) {
            ExceptionUtils.handleThrowable(x);
            log.error("",x);
            continue;
        }
        //either we timed out or we woke up, process events first
        if ( keyCount == 0 ) hasEvents = (hasEvents | events());

        // 获取当前选择器中所有注册的“选择键(已就绪的监听事件)”
        Iterator<SelectionKey> iterator =
            keyCount > 0 ? selector.selectedKeys().iterator() : null;
        // Walk through the collection of ready keys and dispatch
        // any active event.
        // 对已经准备好的key进行处理
        while (iterator != null && iterator.hasNext()) {
            SelectionKey sk = iterator.next();
            NioSocketWrapper attachment = (NioSocketWrapper)sk.attachment();
            // Attachment may be null if another thread has called
            // cancelledKey()
            if (attachment == null) {
                iterator.remove();
            } else {
                iterator.remove();
                // 真正处理key的地方
                processKey(sk, attachment);
            }
        }//while

        //process timeouts
        timeout(keyCount,hasEvents);
    }//while

    getStopLatch().countDown();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 若队列里有元素则会先把队列里的事件均执行一遍，PollerEvent的run方法会将通道注册到Poller的Selector上；
- 对select返回的SelectionKey进行处理，由于在PollerEvent中注册通道时带上了NioSocketWrapper附件，因此这里可以用SelectionKey的attachment方法得到，接着调用processKey去处理已连接套接字通道。

我们接着分析`processKey()`，该方法又会根据key的类型，来分别处理读和写。

1. 处理读事件，比如生成Request对象
2. 处理写事件，比如将生成的Response对象通过socket写回客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void processKey(SelectionKey sk, NioSocketWrapper attachment) {
    try {
        if ( close ) {
            cancelledKey(sk);
        } else if ( sk.isValid() && attachment != null ) {
            if (sk.isReadable() || sk.isWritable() ) {
                if ( attachment.getSendfileData() != null ) {
                    processSendfile(sk,attachment, false);
                } else {
                    unreg(sk, attachment, sk.readyOps());
                    boolean closeSocket = false;
                    // 1. 处理读事件，比如生成Request对象
                    // Read goes before write
                    if (sk.isReadable()) {
                        if (!processSocket(attachment, SocketEvent.OPEN_READ, true)) {
                            closeSocket = true;
                        }
                    }
                    // 2. 处理写事件，比如将生成的Response对象通过socket写回客户端
                    if (!closeSocket && sk.isWritable()) {
                        if (!processSocket(attachment, SocketEvent.OPEN_WRITE, true)) {
                            closeSocket = true;
                        }
                    }
                    if (closeSocket) {
                        cancelledKey(sk);
                    }
                }
            }
        } else {
            //invalid key
            cancelledKey(sk);
        }
    } catch ( CancelledKeyException ckx ) {
        cancelledKey(sk);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        log.error("",t);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们继续来分析方法`processSocket()`。

1. 从`processorCache`里面拿一个`Processor`来处理socket，`Processor`的实现为`SocketProcessor`
2. 将`Processor`放到工作线程池中执行

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
        SocketEvent event, boolean dispatch) {
    try {
        if (socketWrapper == null) {
            return false;
        }
        // 1. 从`processorCache`里面拿一个`Processor`来处理socket，`Processor`的实现为`SocketProcessor`
        SocketProcessorBase<S> sc = processorCache.pop();
        if (sc == null) {
            sc = createSocketProcessor(socketWrapper, event);
        } else {
            sc.reset(socketWrapper, event);
        }
        // 2. 将`Processor`放到工作线程池中执行
        Executor executor = getExecutor();
        if (dispatch && executor != null) {
            executor.execute(sc);
        } else {
            sc.run();
        }
    } catch (RejectedExecutionException ree) {
        getLog().warn(sm.getString("endpoint.executor.fail", socketWrapper) , ree);
        return false;
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        // This means we got an OOM or similar creating a thread, or that
        // the pool and its queue are full
        getLog().error(sm.getString("endpoint.process.fail"), t);
        return false;
    }
    return true;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

dispatch参数表示是否要在另外的线程中处理，上文processKey各处传递的参数都是true。

- dispatch为true且工作线程池存在时会执行executor.execute(sc)，之后是由工作线程池处理已连接套接字；
- 否则继续由Poller线程自己处理已连接套接字。

AbstractEndPoint类的createSocketProcessor是抽象方法，NioEndPoint类实现了它：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected SocketProcessorBase<NioChannel> createSocketProcessor(
        SocketWrapperBase<NioChannel> socketWrapper, SocketEvent event) {
    return new SocketProcessor(socketWrapper, event);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接着我们分析`SocketProcessor.doRun()`方法（`SocketProcessor.run()`方法最终调用此方法）。该方法将处理逻辑交给`Handler`处理，当event为null时，则表明是一个`OPEN_READ`事件。

该类的注释说明SocketProcessor与Worker的作用等价。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * This class is the equivalent of the Worker, but will simply use in an
 * external Executor thread pool.
 */
protected class SocketProcessor extends SocketProcessorBase<NioChannel> {

    public SocketProcessor(SocketWrapperBase<NioChannel> socketWrapper, SocketEvent event) {
        super(socketWrapper, event);
    }

    @Override
    protected void doRun() {
        NioChannel socket = socketWrapper.getSocket();
        SelectionKey key = socket.getIOChannel().keyFor(socket.getPoller().getSelector());

        try {
            int handshake = -1;

            try {
                if (key != null) {
                    if (socket.isHandshakeComplete()) {
                        // No TLS handshaking required. Let the handler
                        // process this socket / event combination.
                        handshake = 0;
                    } else if (event == SocketEvent.STOP || event == SocketEvent.DISCONNECT ||
                            event == SocketEvent.ERROR) {
                        // Unable to complete the TLS handshake. Treat it as
                        // if the handshake failed.
                        handshake = -1;
                    } else {
                        handshake = socket.handshake(key.isReadable(), key.isWritable());
                        // The handshake process reads/writes from/to the
                        // socket. status may therefore be OPEN_WRITE once
                        // the handshake completes. However, the handshake
                        // happens when the socket is opened so the status
                        // must always be OPEN_READ after it completes. It
                        // is OK to always set this as it is only used if
                        // the handshake completes.
                        event = SocketEvent.OPEN_READ;
                    }
                }
            } catch (IOException x) {
                handshake = -1;
                if (log.isDebugEnabled()) log.debug("Error during SSL handshake",x);
            } catch (CancelledKeyException ckx) {
                handshake = -1;
            }
            if (handshake == 0) {
                SocketState state = SocketState.OPEN;
                // Process the request from this socket
                // 将处理逻辑交给`Handler`处理，当event为null时，则表明是一个`OPEN_READ`事件
                if (event == null) {
                    state = getHandler().process(socketWrapper, SocketEvent.OPEN_READ);
                } else {
                    state = getHandler().process(socketWrapper, event);
                }
                if (state == SocketState.CLOSED) {
                    close(socket, key);
                }
            } else if (handshake == -1 ) {
                close(socket, key);
            } else if (handshake == SelectionKey.OP_READ){
                socketWrapper.registerReadInterest();
            } else if (handshake == SelectionKey.OP_WRITE){
                socketWrapper.registerWriteInterest();
            }
        } catch (CancelledKeyException cx) {
            socket.getPoller().cancelledKey(key);
        } catch (VirtualMachineError vme) {
            ExceptionUtils.handleThrowable(vme);
        } catch (Throwable t) {
            log.error("", t);
            socket.getPoller().cancelledKey(key);
        } finally {
            socketWrapper = null;
            event = null;
            //return to cache
            if (running && !paused) {
                processorCache.push(this);
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Handler`的关键方法是`process(),虽然这个方法有很多条件分支，但是逻辑却非常清楚，主要是调用Processor.process()方法。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public SocketState process(SocketWrapperBase<S> wrapper, SocketEvent status) {
    try {
     
        if (processor == null) {
            processor = getProtocol().createProcessor();
            register(processor);
        }

        processor.setSslSupport(
                wrapper.getSslSupport(getProtocol().getClientCertProvider()));

        // Associate the processor with the connection
        connections.put(socket, processor);

        SocketState state = SocketState.CLOSED;
        do {
            // 关键的代码，终于找到你了
            state = processor.process(wrapper, status);

        } while ( state == SocketState.UPGRADING);
        return state;
    } 
    catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        // any other exception or error is odd. Here we log it
        // with "ERROR" level, so it will show up even on
        // less-than-verbose logs.
        getLog().error(sm.getString("abstractConnectionHandler.error"), e);
    } finally {
        ContainerThreadMarker.clear();
    }

    // Make sure socket/processor is removed from the list of current
    // connections
    connections.remove(socket);
    release(processor);
    return SocketState.CLOSED;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### Processor

**createProcessor** 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected Http11Processor createProcessor() {                          
    // 构建 Http11Processor
    Http11Processor processor = new Http11Processor(
            proto.getMaxHttpHeaderSize(), (JIoEndpoint)proto.endpoint, // 1. http header 的最大尺寸
            proto.getMaxTrailerSize(),proto.getMaxExtensionSize());
    processor.setAdapter(proto.getAdapter());
    // 2. 默认的 KeepAlive 情况下, 每个 Socket 处理的最多的 请求次数
    processor.setMaxKeepAliveRequests(proto.getMaxKeepAliveRequests());
    // 3. 开启 KeepAlive 的 Timeout
    processor.setKeepAliveTimeout(proto.getKeepAliveTimeout());      
    // 4. http 当遇到文件上传时的 默认超时时间 (300 * 1000)    
    processor.setConnectionUploadTimeout(
            proto.getConnectionUploadTimeout());                      
    processor.setDisableUploadTimeout(proto.getDisableUploadTimeout());
    // 5. 当 http 请求的 body size超过这个值时, 通过 gzip 进行压缩
    processor.setCompressionMinSize(proto.getCompressionMinSize());  
    // 6. http 请求是否开启 compression 处理    
    processor.setCompression(proto.getCompression());                  
    processor.setNoCompressionUserAgents(proto.getNoCompressionUserAgents());
    // 7. http body里面的内容是 "text/html,text/xml,text/plain" 才会进行 压缩处理
    processor.setCompressableMimeTypes(proto.getCompressableMimeTypes());
    processor.setRestrictedUserAgents(proto.getRestrictedUserAgents());
    // 8. socket 的 buffer, 默认 9000
    processor.setSocketBuffer(proto.getSocketBuffer());       
    // 9. 最大的 Post 处理尺寸的大小 4 * 1000    
    processor.setMaxSavePostSize(proto.getMaxSavePostSize());          
    processor.setServer(proto.getServer());
    processor.setDisableKeepAlivePercentage(
            proto.getDisableKeepAlivePercentage());                    
    register(processor);                                               
    return processor;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这儿我们主要关注的是`Processor`对于读的操作，也只有一行代码。调用`service()`方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract class AbstractProcessorLight implements Processor {

    @Override
    public SocketState process(SocketWrapperBase<?> socketWrapper, SocketEvent status)
            throws IOException {

        SocketState state = SocketState.CLOSED;
        Iterator<DispatchType> dispatches = null;
        do {
            if (dispatches != null) {
                DispatchType nextDispatch = dispatches.next();
                state = dispatch(nextDispatch.getSocketStatus());
            } else if (status == SocketEvent.DISCONNECT) {
                // Do nothing here, just wait for it to get recycled
            } else if (isAsync() || isUpgrade() || state == SocketState.ASYNC_END) {
                state = dispatch(status);
                if (state == SocketState.OPEN) {
                    // There may be pipe-lined data to read. If the data isn't
                    // processed now, execution will exit this loop and call
                    // release() which will recycle the processor (and input
                    // buffer) deleting any pipe-lined data. To avoid this,
                    // process it now.
                    state = service(socketWrapper);
                }
            } else if (status == SocketEvent.OPEN_WRITE) {
                // Extra write event likely after async, ignore
                state = SocketState.LONG;
            } else if (status == SocketEvent.OPEN_READ){
                // 调用`service()`方法
                state = service(socketWrapper);
            } else {
                // Default to closing the socket if the SocketEvent passed in
                // is not consistent with the current state of the Processor
                state = SocketState.CLOSED;
            }

            if (getLog().isDebugEnabled()) {
                getLog().debug("Socket: [" + socketWrapper +
                        "], Status in: [" + status +
                        "], State out: [" + state + "]");
            }

            if (state != SocketState.CLOSED && isAsync()) {
                state = asyncPostProcess();
                if (getLog().isDebugEnabled()) {
                    getLog().debug("Socket: [" + socketWrapper +
                            "], State after async post processing: [" + state + "]");
                }
            }

            if (dispatches == null || !dispatches.hasNext()) {
                // Only returns non-null iterator if there are
                // dispatches to process.
                dispatches = getIteratorAndClearDispatches();
            }
        } while (state == SocketState.ASYNC_END ||
                dispatches != null && state != SocketState.CLOSED);

        return state;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`Processor.service()`方法比较重要的地方就两点。该方法非常得长，也超过了200行，在此我们不再拷贝此方法的代码。

1. 生成Request和Response对象
2. 调用`Adapter.service()`方法，将生成的Request和Response对象传进去

### Adapter

`Adapter`用于连接`Connector`和`Container`，起到承上启下的作用。`Processor`会调用`Adapter.service()`方法。我们来分析一下，主要做了下面几件事情：

1. 根据coyote框架的request和response对象，生成connector的request和response对象（是HttpServletRequest和HttpServletResponse的封装）
2. 补充header
3. 解析请求，该方法会出现代理服务器、设置必要的header等操作
4. 真正进入容器的地方，调用Engine容器下pipeline的阀门
5. 通过request.finishRequest 与 response.finishResponse(刷OutputBuffer中的数据到浏览器) 来完成整个请求

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void service(org.apache.coyote.Request req, org.apache.coyote.Response res)
        throws Exception {

    // 1. 根据coyote框架的request和response对象，生成connector的request和response对象（是HttpServletRequest和HttpServletResponse的封装）
    Request request = (Request) req.getNote(ADAPTER_NOTES);
    Response response = (Response) res.getNote(ADAPTER_NOTES);

    if (request == null) {
        // Create objects
        request = connector.createRequest();
        request.setCoyoteRequest(req);
        response = connector.createResponse();
        response.setCoyoteResponse(res);

        // Link objects
        request.setResponse(response);
        response.setRequest(request);

        // Set as notes
        req.setNote(ADAPTER_NOTES, request);
        res.setNote(ADAPTER_NOTES, response);

        // Set query string encoding
        req.getParameters().setQueryStringCharset(connector.getURICharset());
    }

    // 2. 补充header
    if (connector.getXpoweredBy()) {
        response.addHeader("X-Powered-By", POWERED_BY);
    }

    boolean async = false;
    boolean postParseSuccess = false;

    req.getRequestProcessor().setWorkerThreadName(THREAD_NAME.get());

    try {
        // Parse and set Catalina and configuration specific
        // request parameters
        // 3. 解析请求，该方法会出现代理服务器、设置必要的header等操作
        // 用来处理请求映射 (获取 host, context, wrapper, URI 后面的参数的解析, sessionId )
        postParseSuccess = postParseRequest(req, request, res, response);
        if (postParseSuccess) {
            //check valves if we support async
            request.setAsyncSupported(
                    connector.getService().getContainer().getPipeline().isAsyncSupported());
            // Calling the container
            // 4. 真正进入容器的地方，调用Engine容器下pipeline的阀门
            connector.getService().getContainer().getPipeline().getFirst().invoke(
                    request, response);
        }
        if (request.isAsync()) {
            async = true;
            ReadListener readListener = req.getReadListener();
            if (readListener != null && request.isFinished()) {
                // Possible the all data may have been read during service()
                // method so this needs to be checked here
                ClassLoader oldCL = null;
                try {
                    oldCL = request.getContext().bind(false, null);
                    if (req.sendAllDataReadEvent()) {
                        req.getReadListener().onAllDataRead();
                    }
                } finally {
                    request.getContext().unbind(false, oldCL);
                }
            }

            Throwable throwable =
                    (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);

            // If an async request was started, is not going to end once
            // this container thread finishes and an error occurred, trigger
            // the async error process
            if (!request.isAsyncCompleting() && throwable != null) {
                request.getAsyncContextInternal().setErrorState(throwable, true);
            }
        } else {
            //5. 通过request.finishRequest 与 response.finishResponse(刷OutputBuffer中的数据到浏览器) 来完成整个请求
            request.finishRequest();
            //将 org.apache.catalina.connector.Response对应的 OutputBuffer 中的数据 刷到 org.apache.coyote.Response 对应的 InternalOutputBuffer 中, 并且最终调用 socket对应的 outputStream 将数据刷出去( 这里会组装 Http Response 中的 header 与 body 里面的数据, 并且刷到远端 )
            response.finishResponse();
        }

    } catch (IOException e) {
        // Ignore
    } finally {
        AtomicBoolean error = new AtomicBoolean(false);
        res.action(ActionCode.IS_ERROR, error);

        if (request.isAsyncCompleting() && error.get()) {
            // Connection will be forcibly closed which will prevent
            // completion happening at the usual point. Need to trigger
            // call to onComplete() here.
            res.action(ActionCode.ASYNC_POST_PROCESS,  null);
            async = false;
        }

        // Access log
        if (!async && postParseSuccess) {
            // Log only if processing was invoked.
            // If postParseRequest() failed, it has already logged it.
            Context context = request.getContext();
            // If the context is null, it is likely that the endpoint was
            // shutdown, this connection closed and the request recycled in
            // a different thread. That thread will have updated the access
            // log so it is OK not to update the access log here in that
            // case.
            if (context != null) {
                context.logAccess(request, response,
                        System.currentTimeMillis() - req.getStartTime(), false);
            }
        }

        req.getRequestProcessor().setWorkerThreadName(null);

        // Recycle the wrapper request and response
        if (!async) {
            request.recycle();
            response.recycle();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 请求预处理

postParseRequest方法对请求做预处理，如对路径去除分号表示的路径参数、进行URI解码、规格化（点号和两点号）

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected boolean postParseRequest(org.apache.coyote.Request req, Request request,
        org.apache.coyote.Response res, Response response) throws IOException, ServletException {
    // 省略部分代码
    MessageBytes decodedURI = req.decodedURI();

    if (undecodedURI.getType() == MessageBytes.T_BYTES) {
        // Copy the raw URI to the decodedURI
        decodedURI.duplicate(undecodedURI);

        // Parse the path parameters. This will:
        //   - strip out the path parameters
        //   - convert the decodedURI to bytes
        parsePathParameters(req, request);

        // URI decoding
        // %xx decoding of the URL
        try {
            req.getURLDecoder().convert(decodedURI, false);
        } catch (IOException ioe) {
            res.setStatus(400);
            res.setMessage("Invalid URI: " + ioe.getMessage());
            connector.getService().getContainer().logAccess(
                    request, response, 0, true);
            return false;
        }
        // Normalization
        if (!normalize(req.decodedURI())) {
            res.setStatus(400);
            res.setMessage("Invalid URI");
            connector.getService().getContainer().logAccess(
                    request, response, 0, true);
            return false;
        }
        // Character decoding
        convertURI(decodedURI, request);
        // Check that the URI is still normalized
        if (!checkNormalize(req.decodedURI())) {
            res.setStatus(400);
            res.setMessage("Invalid URI character encoding");
            connector.getService().getContainer().logAccess(
                    request, response, 0, true);
            return false;
        }
    } else {
        /* The URI is chars or String, and has been sent using an in-memory
            * protocol handler. The following assumptions are made:
            * - req.requestURI() has been set to the 'original' non-decoded,
            *   non-normalized URI
            * - req.decodedURI() has been set to the decoded, normalized form
            *   of req.requestURI()
            */
        decodedURI.toChars();
        // Remove all path parameters; any needed path parameter should be set
        // using the request object rather than passing it in the URL
        CharChunk uriCC = decodedURI.getCharChunk();
        int semicolon = uriCC.indexOf(';');
        if (semicolon > 0) {
            decodedURI.setChars
                (uriCC.getBuffer(), uriCC.getStart(), semicolon);
        }
    }

    // Request mapping.
    MessageBytes serverName;
    if (connector.getUseIPVHosts()) {
        serverName = req.localName();
        if (serverName.isNull()) {
            // well, they did ask for it
            res.action(ActionCode.REQ_LOCAL_NAME_ATTRIBUTE, null);
        }
    } else {
        serverName = req.serverName();
    }

    // Version for the second mapping loop and
    // Context that we expect to get for that version
    String version = null;
    Context versionContext = null;
    boolean mapRequired = true;

    while (mapRequired) {
        // This will map the the latest version by default
        connector.getService().getMapper().map(serverName, decodedURI,
                version, request.getMappingData());
        // 省略部分代码
    }
    // 省略部分代码
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

以MessageBytes的类型是T_BYTES为例：

- parsePathParameters方法去除URI中分号表示的路径参数；
- req.getURLDecoder()得到一个UDecoder实例，它的convert方法对URI解码，这里的解码只是移除百分号，计算百分号后两位的十六进制数字值以替代原来的三位百分号编码；
- normalize方法规格化URI，解释路径中的“.”和“..”；
- convertURI方法利用Connector的uriEncoding属性将URI的字节转换为字符表示；
- 注意connector.getService().getMapper().map(serverName, decodedURI, version, request.getMappingData()) 这行，之前Service启动时MapperListener注册了该Service内的各Host和Context。根据URI选择Context时，Mapper的map方法采用的是convertURI方法解码后的URI与每个Context的路径去比较

### 容器处理

如果请求可以被传给容器的Pipeline即当postParseRequest方法返回true时，则由容器继续处理，在service方法中有connector.getService().getContainer().getPipeline().getFirst().invoke(request, response)这一行：

- Connector调用getService返回StandardService；
- StandardService调用getContainer返回StandardEngine；
- StandardEngine调用getPipeline返回与其关联的StandardPipeline；

 后续处理流程请看下一篇文章

**正文**

我们接着上一篇文章的容器处理来讲，当postParseRequest方法返回true时，则由容器继续处理，在service方法中有**connector.getService().getContainer().getPipeline().getFirst().invoke(request, response)**这一行：

- Connector调用getService()返回StandardService；
- StandardService调用getContainer返回StandardEngine；
- StandardEngine调用getPipeline返回与其关联的StandardPipeline；

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11309722.html#_labelTop)

## Engine处理请求

我们在前面的文章中讲过**`StandardEngine`**的构造函数为自己的Pipeline添加了基本阀**`StandardEngineValve`**，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public StandardEngine() {
    super();
    pipeline.setBasic(new StandardEngineValve());
    try {
        setJvmRoute(System.getProperty("jvmRoute"));
    } catch(Exception ex) {
        log.warn(sm.getString("standardEngine.jvmRouteFail"));
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接下来我们看看`StandardEngineValve`的`invoke()`方法。该方法主要是选择合适的Host，然后调用Host中pipeline的第一个`Valve的invoke()`方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final void invoke(Request request, Response response)
    throws IOException, ServletException {

    // Select the Host to be used for this Request
    Host host = request.getHost();
    if (host == null) {
        response.sendError
            (HttpServletResponse.SC_BAD_REQUEST,
             sm.getString("standardEngine.noHost",
                          request.getServerName()));
        return;
    }
    if (request.isAsyncSupported()) {
        request.setAsyncSupported(host.getPipeline().isAsyncSupported());
    }

    // Ask this Host to process this request
    host.getPipeline().getFirst().invoke(request, response);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法很简单，校验该Engline 容器是否含有Host容器，如果不存在，返回400错误，否则继续执行 `host.getPipeline().getFirst().invoke(request, response)`，可以看到 Host 容器先获取自己的管道，再获取第一个阀门，我们再看看该阀门的 invoke 方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11309722.html#_labelTop)

## Host处理请求

分析Host的时候，我们从Host的构造函数入手，该方法主要是设置基础阀门。

```
public StandardHost() {
    super();
    pipeline.setBasic(new StandardHostValve());
}
```

StandardPipeline调用getFirst得到第一个阀去处理请求，由于基本阀是最后一个，所以最后会由基本阀去处理请求。

StandardHost的Pipeline里面一定有 ErrorReportValve 与 StandardHostValve两个Valve，ErrorReportValve主要是检测 Http 请求过程中是否出现过什么异常, 有异常的话, 直接拼装 html 页面, 输出到客户端。

我们看看ErrorReportValve的invoke方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void invoke(Request request, Response response)
    throws IOException, ServletException {
    // Perform the request
    // 1. 先将 请求转发给下一个 Valve
    getNext().invoke(request, response);  
    // 2. 这里的 isCommitted 表明, 请求是正常处理结束    
    if (response.isCommitted()) {               
        return;
    }
    // 3. 判断请求过程中是否有异常发生
    Throwable throwable = (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
    if (request.isAsyncStarted() && ((response.getStatus() < 400 &&
            throwable == null) || request.isAsyncDispatching())) {
        return;
    }
    if (throwable != null) {
        // The response is an error
        response.setError();
        // Reset the response (if possible)
        try {
            // 4. 重置 response 里面的数据(此时 Response 里面可能有些数据)
            response.reset();                  
        } catch (IllegalStateException e) {
            // Ignore
        }
         // 5. 这就是我们常看到的 500 错误码
        response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
    }
    response.setSuspended(false);
    try {
        // 6. 这里就是将 异常的堆栈信息组合成 html 页面, 输出到前台        
        report(request, response, throwable);                                   
    } catch (Throwable tt) {
        ExceptionUtils.handleThrowable(tt);
    }
    if (request.isAsyncStarted()) {          
        // 7. 若是异步请求的话, 设置对应的 complete (对应的是 异步 Servlet)                   
        request.getAsyncContext().complete();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法首先执行了下个阀门的 invoke 方法。然后根据返回的Request 属性设置一些错误信息。那么下个阀门是谁呢？其实就是基础阀门了：StandardHostValve，该阀门的 invoke 的方法是如何实现的呢？

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void invoke(Request request, Response response)
    throws IOException, ServletException {

    // Select the Context to be used for this Request
    Context context = request.getContext();
    if (context == null) {
        response.sendError
            (HttpServletResponse.SC_INTERNAL_SERVER_ERROR,
             sm.getString("standardHost.noContext"));
        return;
    }

    // Bind the context CL to the current thread
    if( context.getLoader() != null ) {
        // Not started - it should check for availability first
        // This should eventually move to Engine, it's generic.
        if (Globals.IS_SECURITY_ENABLED) {
            PrivilegedAction<Void> pa = new PrivilegedSetTccl(
                    context.getLoader().getClassLoader());
            AccessController.doPrivileged(pa);                
        } else {
            Thread.currentThread().setContextClassLoader
                    (context.getLoader().getClassLoader());
        }
    }
    if (request.isAsyncSupported()) {
        request.setAsyncSupported(context.getPipeline().isAsyncSupported());
    }

    // Don't fire listeners during async processing
    // If a request init listener throws an exception, the request is
    // aborted
    boolean asyncAtStart = request.isAsync(); 
    // An async error page may dispatch to another resource. This flag helps
    // ensure an infinite error handling loop is not entered
    boolean errorAtStart = response.isError();
    if (asyncAtStart || context.fireRequestInitEvent(request)) {

        // Ask this Context to process this request
        try {
            context.getPipeline().getFirst().invoke(request, response);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            if (errorAtStart) {
                container.getLogger().error("Exception Processing " +
                        request.getRequestURI(), t);
            } else {
                request.setAttribute(RequestDispatcher.ERROR_EXCEPTION, t);
                throwable(request, response, t);
            }
        }

        // If the request was async at the start and an error occurred then
        // the async error handling will kick-in and that will fire the
        // request destroyed event *after* the error handling has taken
        // place
        if (!(request.isAsync() || (asyncAtStart &&
                request.getAttribute(
                        RequestDispatcher.ERROR_EXCEPTION) != null))) {
            // Protect against NPEs if context was destroyed during a
            // long running request.
            if (context.getState().isAvailable()) {
                if (!errorAtStart) {
                    // Error page processing
                    response.setSuspended(false);

                    Throwable t = (Throwable) request.getAttribute(
                            RequestDispatcher.ERROR_EXCEPTION);

                    if (t != null) {
                        throwable(request, response, t);
                    } else {
                        status(request, response);
                    }
                }

                context.fireRequestDestroyEvent(request);
            }
        }
    }

    // Access a session (if present) to update last accessed time, based on a
    // strict interpretation of the specification
    if (ACCESS_SESSION) {
        request.getSession(false);
    }

    // Restore the context classloader
    if (Globals.IS_SECURITY_ENABLED) {
        PrivilegedAction<Void> pa = new PrivilegedSetTccl(
                StandardHostValve.class.getClassLoader());
        AccessController.doPrivileged(pa);                
    } else {
        Thread.currentThread().setContextClassLoader
                (StandardHostValve.class.getClassLoader());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先校验了Request 是否存在 Context，其实在执行 CoyoteAdapter.postParseRequest 方法的时候就设置了，如果Context 不存在，就返回500，接着还是老套路：context.getPipeline().getFirst().invoke，该管道获取的是基础阀门：StandardContextValve，我们还是关注他的 invoke 方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11309722.html#_labelTop)

## Context处理请求

接着Context会去处理请求，同理，StandardContextValve的invoke方法会被调用：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void invoke(Request request, Response response)
    throws IOException, ServletException {
    // Disallow any direct access to resources under WEB-INF or META-INF
    MessageBytes requestPathMB = request.getRequestPathMB();
    if ((requestPathMB.startsWithIgnoreCase("/META-INF/", 0))
            || (requestPathMB.equalsIgnoreCase("/META-INF"))
            || (requestPathMB.startsWithIgnoreCase("/WEB-INF/", 0))
            || (requestPathMB.equalsIgnoreCase("/WEB-INF"))) {
        response.sendError(HttpServletResponse.SC_NOT_FOUND);
        return;
    }

    // Select the Wrapper to be used for this Request
    Wrapper wrapper = request.getWrapper();
    if (wrapper == null || wrapper.isUnavailable()) {
        response.sendError(HttpServletResponse.SC_NOT_FOUND);
        return;
    }

    // Acknowledge the request
    try {
        response.sendAcknowledgement();
    } catch (IOException ioe) {
        container.getLogger().error(sm.getString(
                "standardContextValve.acknowledgeException"), ioe);
        request.setAttribute(RequestDispatcher.ERROR_EXCEPTION, ioe);
        response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        return;
    }

    if (request.isAsyncSupported()) {
        request.setAsyncSupported(wrapper.getPipeline().isAsyncSupported());
    }
    wrapper.getPipeline().getFirst().invoke(request, response);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11309722.html#_labelTop)

## Wrapper处理请求

Wrapper是一个Servlet的包装，我们先来看看构造方法。主要作用就是设置基础阀门`StandardWrapperValve`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public StandardWrapper() {
    super();
    swValve=new StandardWrapperValve();
    pipeline.setBasic(swValve);
    broadcaster = new NotificationBroadcasterSupport();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接下来我们看看`StandardWrapperValve`的`invoke()`方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void invoke(Request request, Response response)
    throws IOException, ServletException {

    // Initialize local variables we may need
    boolean unavailable = false;
    Throwable throwable = null;
    // This should be a Request attribute...
    long t1=System.currentTimeMillis();
    requestCount.incrementAndGet();
    StandardWrapper wrapper = (StandardWrapper) getContainer();
    Servlet servlet = null;
    Context context = (Context) wrapper.getParent();

    // Check for the application being marked unavailable
    if (!context.getState().isAvailable()) {
        response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                       sm.getString("standardContext.isUnavailable"));
        unavailable = true;
    }

    // Check for the servlet being marked unavailable
    if (!unavailable && wrapper.isUnavailable()) {
        container.getLogger().info(sm.getString("standardWrapper.isUnavailable",
                wrapper.getName()));
        long available = wrapper.getAvailable();
        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            response.setDateHeader("Retry-After", available);
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                    sm.getString("standardWrapper.isUnavailable",
                            wrapper.getName()));
        } else if (available == Long.MAX_VALUE) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND,
                    sm.getString("standardWrapper.notFound",
                            wrapper.getName()));
        }
        unavailable = true;
    }

    // Allocate a servlet instance to process this request
    try {
        // 关键点1：这儿调用Wrapper的allocate()方法分配一个Servlet实例
        if (!unavailable) {
            servlet = wrapper.allocate();
        }
    } catch (UnavailableException e) {
        container.getLogger().error(
                sm.getString("standardWrapper.allocateException",
                        wrapper.getName()), e);
        long available = wrapper.getAvailable();
        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            response.setDateHeader("Retry-After", available);
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                       sm.getString("standardWrapper.isUnavailable",
                                    wrapper.getName()));
        } else if (available == Long.MAX_VALUE) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND,
                       sm.getString("standardWrapper.notFound",
                                    wrapper.getName()));
        }
    } catch (ServletException e) {
        container.getLogger().error(sm.getString("standardWrapper.allocateException",
                         wrapper.getName()), StandardWrapper.getRootCause(e));
        throwable = e;
        exception(request, response, e);
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString("standardWrapper.allocateException",
                         wrapper.getName()), e);
        throwable = e;
        exception(request, response, e);
        servlet = null;
    }

    MessageBytes requestPathMB = request.getRequestPathMB();
    DispatcherType dispatcherType = DispatcherType.REQUEST;
    if (request.getDispatcherType()==DispatcherType.ASYNC) dispatcherType = DispatcherType.ASYNC;
    request.setAttribute(Globals.DISPATCHER_TYPE_ATTR,dispatcherType);
    request.setAttribute(Globals.DISPATCHER_REQUEST_PATH_ATTR,
            requestPathMB);
    // Create the filter chain for this request
    // 关键点2，创建过滤器链，类似于Pipeline的功能
    ApplicationFilterChain filterChain =
            ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);

    // Call the filter chain for this request
    // NOTE: This also calls the servlet's service() method
    try {
        if ((servlet != null) && (filterChain != null)) {
            // Swallow output if needed
            if (context.getSwallowOutput()) {
                try {
                    SystemLogHandler.startCapture();
                    if (request.isAsyncDispatching()) {
                        request.getAsyncContextInternal().doInternalDispatch();
                    } else {
                        // 关键点3，调用过滤器链的doFilter，最终会调用到Servlet的service方法
                        filterChain.doFilter(request.getRequest(),
                                response.getResponse());
                    }
                } finally {
                    String log = SystemLogHandler.stopCapture();
                    if (log != null && log.length() > 0) {
                        context.getLogger().info(log);
                    }
                }
            } else {
                if (request.isAsyncDispatching()) {
                    request.getAsyncContextInternal().doInternalDispatch();
                } else {
                    // 关键点3，调用过滤器链的doFilter，最终会调用到Servlet的service方法
                    filterChain.doFilter
                        (request.getRequest(), response.getResponse());
                }
            }

        }
    } catch (ClientAbortException e) {
        throwable = e;
        exception(request, response, e);
    } catch (IOException e) {
        container.getLogger().error(sm.getString(
                "standardWrapper.serviceException", wrapper.getName(),
                context.getName()), e);
        throwable = e;
        exception(request, response, e);
    } catch (UnavailableException e) {
        container.getLogger().error(sm.getString(
                "standardWrapper.serviceException", wrapper.getName(),
                context.getName()), e);
        //            throwable = e;
        //            exception(request, response, e);
        wrapper.unavailable(e);
        long available = wrapper.getAvailable();
        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            response.setDateHeader("Retry-After", available);
            response.sendError(HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                       sm.getString("standardWrapper.isUnavailable",
                                    wrapper.getName()));
        } else if (available == Long.MAX_VALUE) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND,
                        sm.getString("standardWrapper.notFound",
                                    wrapper.getName()));
        }
        // Do not save exception in 'throwable', because we
        // do not want to do exception(request, response, e) processing
    } catch (ServletException e) {
        Throwable rootCause = StandardWrapper.getRootCause(e);
        if (!(rootCause instanceof ClientAbortException)) {
            container.getLogger().error(sm.getString(
                    "standardWrapper.serviceExceptionRoot",
                    wrapper.getName(), context.getName(), e.getMessage()),
                    rootCause);
        }
        throwable = e;
        exception(request, response, e);
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString(
                "standardWrapper.serviceException", wrapper.getName(),
                context.getName()), e);
        throwable = e;
        exception(request, response, e);
    }

    // Release the filter chain (if any) for this request
    // 关键点4，释放掉过滤器链及其相关资源
    if (filterChain != null) {
        filterChain.release();
    }

    // 关键点5，释放掉Servlet及相关资源
    // Deallocate the allocated servlet instance
    try {
        if (servlet != null) {
            wrapper.deallocate(servlet);
        }
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString("standardWrapper.deallocateException",
                         wrapper.getName()), e);
        if (throwable == null) {
            throwable = e;
            exception(request, response, e);
        }
    }

    // If this servlet has been marked permanently unavailable,
    // unload it and release this instance
    // 关键点6，如果servlet被标记为永远不可达，则需要卸载掉它，并释放这个servlet实例
    try {
        if ((servlet != null) &&
            (wrapper.getAvailable() == Long.MAX_VALUE)) {
            wrapper.unload();
        }
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        container.getLogger().error(sm.getString("standardWrapper.unloadException",
                         wrapper.getName()), e);
        if (throwable == null) {
            throwable = e;
            exception(request, response, e);
        }
    }
    long t2=System.currentTimeMillis();

    long time=t2-t1;
    processingTime += time;
    if( time > maxTime) maxTime=time;
    if( time < minTime) minTime=time;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过阅读源码，我们发现了几个关键点。现罗列如下，后面我们会逐一分析这些关键点相关的源码。

1. 关键点1：这儿调用Wrapper的allocate()方法分配一个Servlet实例
2. 关键点2，创建过滤器链，类似于Pipeline的功能
3. 关键点3，调用过滤器链的doFilter，最终会调用到Servlet的service方法
4. 关键点4，释放掉过滤器链及其相关资源
5. 关键点5，释放掉Servlet及相关资源
6. 关键点6，如果servlet被标记为永远不可达，则需要卸载掉它，并释放这个servlet实例



### 关键点1 - Wrapper分配Servlet实例

我们来分析一下Wrapper.allocate()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public Servlet allocate() throws ServletException {

    // If we are currently unloading this servlet, throw an exception
    // 卸载过程中，不能分配Servlet
    if (unloading) {
        throw new ServletException(sm.getString("standardWrapper.unloading", getName()));
    }

    boolean newInstance = false;

    // If not SingleThreadedModel, return the same instance every time
    // 如果Wrapper没有实现SingleThreadedModel，则每次都会返回同一个Servlet
    if (!singleThreadModel) {
        // Load and initialize our instance if necessary
        // 实例为null或者实例还未初始化，使用synchronized来保证并发时的原子性
        if (instance == null || !instanceInitialized) {
            synchronized (this) {
                if (instance == null) {
                    try {
                        if (log.isDebugEnabled()) {
                            log.debug("Allocating non-STM instance");
                        }

                        // Note: We don't know if the Servlet implements
                        // SingleThreadModel until we have loaded it.
                        // 加载Servlet
                        instance = loadServlet();
                        newInstance = true;
                        if (!singleThreadModel) {
                            // For non-STM, increment here to prevent a race
                            // condition with unload. Bug 43683, test case
                            // #3
                            countAllocated.incrementAndGet();
                        }
                    } catch (ServletException e) {
                        throw e;
                    } catch (Throwable e) {
                        ExceptionUtils.handleThrowable(e);
                        throw new ServletException(sm.getString("standardWrapper.allocate"), e);
                    }
                }
                // 初始化Servlet
                if (!instanceInitialized) {
                    initServlet(instance);
                }
            }
        }

        if (singleThreadModel) {
            if (newInstance) {
                // Have to do this outside of the sync above to prevent a
                // possible deadlock
                synchronized (instancePool) {
                    instancePool.push(instance);
                    nInstances++;
                }
            }
        }
        // 非单线程模型，直接返回已经创建的Servlet，也就是说，这种情况下只会创建一个Servlet
        else {
            if (log.isTraceEnabled()) {
                log.trace("  Returning non-STM instance");
            }
            // For new instances, count will have been incremented at the
            // time of creation
            if (!newInstance) {
                countAllocated.incrementAndGet();
            }
            return instance;
        }
    }

    // 如果是单线程模式，则使用servlet对象池技术来加载多个Servlet
    synchronized (instancePool) {
        while (countAllocated.get() >= nInstances) {
            // Allocate a new instance if possible, or else wait
            if (nInstances < maxInstances) {
                try {
                    instancePool.push(loadServlet());
                    nInstances++;
                } catch (ServletException e) {
                    throw e;
                } catch (Throwable e) {
                    ExceptionUtils.handleThrowable(e);
                    throw new ServletException(sm.getString("standardWrapper.allocate"), e);
                }
            } else {
                try {
                    instancePool.wait();
                } catch (InterruptedException e) {
                    // Ignore
                }
            }
        }
        if (log.isTraceEnabled()) {
            log.trace("  Returning allocated STM instance");
        }
        countAllocated.incrementAndGet();
        return instancePool.pop();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

总结下来，注意以下几点即可：

1. 卸载过程中，不能分配Servlet
2. 如果不是单线程模式，则每次都会返回同一个Servlet（默认Servlet实现方式）
3. `Servlet`实例为`null`或者`Servlet`实例还`未初始化`，使用synchronized来保证并发时的原子性
4. 如果是单线程模式，则使用servlet对象池技术来加载多个Servlet

接下来我们看看`loadServlet()`方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public synchronized Servlet loadServlet() throws ServletException {

    // Nothing to do if we already have an instance or an instance pool
    if (!singleThreadModel && (instance != null))
        return instance;

    PrintStream out = System.out;
    if (swallowOutput) {
        SystemLogHandler.startCapture();
    }

    Servlet servlet;
    try {
        long t1=System.currentTimeMillis();
        // Complain if no servlet class has been specified
        if (servletClass == null) {
            unavailable(null);
            throw new ServletException
                (sm.getString("standardWrapper.notClass", getName()));
        }

        // 关键的地方，就是通过实例管理器，创建Servlet实例，而实例管理器是通过特殊的类加载器来加载给定的类
        InstanceManager instanceManager = ((StandardContext)getParent()).getInstanceManager();
        try {
            servlet = (Servlet) instanceManager.newInstance(servletClass);
        } catch (ClassCastException e) {
            unavailable(null);
            // Restore the context ClassLoader
            throw new ServletException
                (sm.getString("standardWrapper.notServlet", servletClass), e);
        } catch (Throwable e) {
            e = ExceptionUtils.unwrapInvocationTargetException(e);
            ExceptionUtils.handleThrowable(e);
            unavailable(null);

            // Added extra log statement for Bugzilla 36630:
            // https://bz.apache.org/bugzilla/show_bug.cgi?id=36630
            if(log.isDebugEnabled()) {
                log.debug(sm.getString("standardWrapper.instantiate", servletClass), e);
            }

            // Restore the context ClassLoader
            throw new ServletException
                (sm.getString("standardWrapper.instantiate", servletClass), e);
        }

        if (multipartConfigElement == null) {
            MultipartConfig annotation =
                    servlet.getClass().getAnnotation(MultipartConfig.class);
            if (annotation != null) {
                multipartConfigElement =
                        new MultipartConfigElement(annotation);
            }
        }

        // Special handling for ContainerServlet instances
        // Note: The InstanceManager checks if the application is permitted
        //       to load ContainerServlets
        if (servlet instanceof ContainerServlet) {
            ((ContainerServlet) servlet).setWrapper(this);
        }

        classLoadTime=(int) (System.currentTimeMillis() -t1);

        if (servlet instanceof SingleThreadModel) {
            if (instancePool == null) {
                instancePool = new Stack<>();
            }
            singleThreadModel = true;
        }

        // 调用Servlet的init方法
        initServlet(servlet);

        fireContainerEvent("load", this);

        loadTime=System.currentTimeMillis() -t1;
    } finally {
        if (swallowOutput) {
            String log = SystemLogHandler.stopCapture();
            if (log != null && log.length() > 0) {
                if (getServletContext() != null) {
                    getServletContext().log(log);
                } else {
                    out.println(log);
                }
            }
        }
    }
    return servlet;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

关键的地方有两个：

1. 通过实例管理器，创建Servlet实例，而实例管理器是通过特殊的类加载器来加载给定的类
2. 调用Servlet的init方法



### 关键点2 - 创建过滤器链

创建过滤器链是调用的`org.apache.catalina.core.ApplicationFilterFactory`的`createFilterChain()`方法。我们来分析一下这个方法。该方法需要注意的地方已经在代码的comments里面说明了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static ApplicationFilterChain createFilterChain(ServletRequest request,
        Wrapper wrapper, Servlet servlet) {

    // If there is no servlet to execute, return null
    if (servlet == null)
        return null;

    // Create and initialize a filter chain object
    // 1. 如果加密打开了，则可能会多次调用这个方法
    // 2. 为了避免重复生成filterChain对象，所以会将filterChain对象放在Request里面进行缓存
    ApplicationFilterChain filterChain = null;
    if (request instanceof Request) {
        Request req = (Request) request;
        if (Globals.IS_SECURITY_ENABLED) {
            // Security: Do not recycle
            filterChain = new ApplicationFilterChain();
        } else {
            filterChain = (ApplicationFilterChain) req.getFilterChain();
            if (filterChain == null) {
                filterChain = new ApplicationFilterChain();
                req.setFilterChain(filterChain);
            }
        }
    } else {
        // Request dispatcher in use
        filterChain = new ApplicationFilterChain();
    }

    filterChain.setServlet(servlet);
    filterChain.setServletSupportsAsync(wrapper.isAsyncSupported());

    // Acquire the filter mappings for this Context
    StandardContext context = (StandardContext) wrapper.getParent();
    // 从这儿看出过滤器链对象里面的元素是根据Context里面的filterMaps来生成的
    FilterMap filterMaps[] = context.findFilterMaps();

    // If there are no filter mappings, we are done
    if ((filterMaps == null) || (filterMaps.length == 0))
        return (filterChain);

    // Acquire the information we will need to match filter mappings
    DispatcherType dispatcher =
            (DispatcherType) request.getAttribute(Globals.DISPATCHER_TYPE_ATTR);

    String requestPath = null;
    Object attribute = request.getAttribute(Globals.DISPATCHER_REQUEST_PATH_ATTR);
    if (attribute != null){
        requestPath = attribute.toString();
    }

    String servletName = wrapper.getName();

    // Add the relevant path-mapped filters to this filter chain
    // 类型和路径都匹配的情况下，将context.filterConfig放到过滤器链里面
    for (int i = 0; i < filterMaps.length; i++) {
        if (!matchDispatcher(filterMaps[i] ,dispatcher)) {
            continue;
        }
        if (!matchFiltersURL(filterMaps[i], requestPath))
            continue;
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
            context.findFilterConfig(filterMaps[i].getFilterName());
        if (filterConfig == null) {
            // FIXME - log configuration problem
            continue;
        }
        filterChain.addFilter(filterConfig);
    }

    // Add filters that match on servlet name second
    // 类型和servlet名称都匹配的情况下，将context.filterConfig放到过滤器链里面
    for (int i = 0; i < filterMaps.length; i++) {
        if (!matchDispatcher(filterMaps[i] ,dispatcher)) {
            continue;
        }
        if (!matchFiltersServlet(filterMaps[i], servletName))
            continue;
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
            context.findFilterConfig(filterMaps[i].getFilterName());
        if (filterConfig == null) {
            // FIXME - log configuration problem
            continue;
        }
        filterChain.addFilter(filterConfig);
    }

    // Return the completed filter chain
    return filterChain;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 关键点3 - 调用过滤器链的doFilter

ApplicationFilterChain类的doFilter函数代码如下,它会将处理委托给internalDoFilter函数。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void doFilter(ServletRequest request, ServletResponse response)
    throws IOException, ServletException {

    if( Globals.IS_SECURITY_ENABLED ) {
        final ServletRequest req = request;
        final ServletResponse res = response;
        try {
            java.security.AccessController.doPrivileged(
                new java.security.PrivilegedExceptionAction<Void>() {
                    @Override
                    public Void run()
                        throws ServletException, IOException {
                        internalDoFilter(req,res);
                        return null;
                    }
                }
            );
        } catch( PrivilegedActionException pe) {
            Exception e = pe.getException();
            if (e instanceof ServletException)
                throw (ServletException) e;
            else if (e instanceof IOException)
                throw (IOException) e;
            else if (e instanceof RuntimeException)
                throw (RuntimeException) e;
            else
                throw new ServletException(e.getMessage(), e);
        }
    } else {
        internalDoFilter(request,response);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ApplicationFilterChain类的internalDoFilter函数代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 1. `internalDoFilter`方法通过pos和n来调用过滤器链里面的每个过滤器。pos表示当前的过滤器下标，n表示总的过滤器数量
// 2. `internalDoFilter`方法最终会调用servlet.service()方法
private void internalDoFilter(ServletRequest request,
                              ServletResponse response)
    throws IOException, ServletException {

    // Call the next filter if there is one
    // 1. 当pos小于n时, 则执行Filter
    if (pos < n) {
        // 2. 得到 过滤器 Filter，执行一次post++
        ApplicationFilterConfig filterConfig = filters[pos++];
        try {
            Filter filter = filterConfig.getFilter();

            if (request.isAsyncSupported() && "false".equalsIgnoreCase(
                    filterConfig.getFilterDef().getAsyncSupported())) {
                request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR, Boolean.FALSE);
            }
            if( Globals.IS_SECURITY_ENABLED ) {
                final ServletRequest req = request;
                final ServletResponse res = response;
                Principal principal =
                    ((HttpServletRequest) req).getUserPrincipal();

                Object[] args = new Object[]{req, res, this};
                SecurityUtil.doAsPrivilege ("doFilter", filter, classType, args, principal);
            } else {
                // 4. 这里的 filter 的执行 有点递归的感觉, 通过 pos 来控制从 filterChain 里面拿出那个 filter 来进行操作
                // 这里把this（filterChain）传到自定义filter里面，我们自定义的filter，会重写doFilter，在这里会被调用，doFilter里面会执行业务逻辑，如果执行业务逻辑成功，则会调用 filterChain.doFilter(servletRequest, servletResponse); ，filterChain就是这里传过去的this；如果业务逻辑执行失败，则return，filterChain终止，后面的servlet.service(request, response)也不会执行了
                // 所以在 Filter 里面所调用 return, 则会终止 Filter 的调用, 而下面的 Servlet.service 更本就没有调用到
                filter.doFilter(request, response, this);
            }
        } catch (IOException | ServletException | RuntimeException e) {
            throw e;
        } catch (Throwable e) {
            e = ExceptionUtils.unwrapInvocationTargetException(e);
            ExceptionUtils.handleThrowable(e);
            throw new ServletException(sm.getString("filterChain.filter"), e);
        }
        return;
    }

    // We fell off the end of the chain -- call the servlet instance
    try {
        if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
            lastServicedRequest.set(request);
            lastServicedResponse.set(response);
        }

        if (request.isAsyncSupported() && !servletSupportsAsync) {
            request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR,
                    Boolean.FALSE);
        }
        // Use potentially wrapped request from this point
        if ((request instanceof HttpServletRequest) &&
                (response instanceof HttpServletResponse) &&
                Globals.IS_SECURITY_ENABLED ) {
            final ServletRequest req = request;
            final ServletResponse res = response;
            Principal principal =
                ((HttpServletRequest) req).getUserPrincipal();
            Object[] args = new Object[]{req, res};
            SecurityUtil.doAsPrivilege("service",
                                       servlet,
                                       classTypeUsedInService,
                                       args,
                                       principal);
        } else {
            //当pos等于n时，过滤器都执行完毕，终于执行了熟悉的servlet.service(request, response)方法。
            servlet.service(request, response);
        }
    } catch (IOException | ServletException | RuntimeException e) {
        throw e;
    } catch (Throwable e) {
        e = ExceptionUtils.unwrapInvocationTargetException(e);
        ExceptionUtils.handleThrowable(e);
        throw new ServletException(sm.getString("filterChain.servlet"), e);
    } finally {
        if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
            lastServicedRequest.set(null);
            lastServicedResponse.set(null);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

自定义Filter

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@WebFilter(urlPatterns = "/*", filterName = "myfilter")
public class FileterController implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("Filter初始化中");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        System.out.println("登录逻辑");
        if("登录失败"){
            response.getWriter().write("登录失败");
            //后面的拦截器和servlet都不会执行了
            return;
        }
        //登录成功，执行下一个过滤器
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("Filter销毁中");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- **pos和n是ApplicationFilterChain的成员变量，分别表示过滤器链的当前位置和过滤器总数，所以当pos小于n时，会不断执行ApplicationFilterChain的doFilter方法；**
- **当pos等于n时，过滤器都执行完毕，终于执行了熟悉的servlet.service(request, response)方法。**

# [Tomcat源码分析 （十）----- 彻底理解 Session机制](https://www.cnblogs.com/java-chen-hao/p/11316172.html)

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316172.html#_labelTop)

## Tomcat Session 概述

首先 HTTP 是一个无状态的协议, 这意味着每次发起的HTTP请求, 都是一个全新的请求(与上个请求没有任何联系, 服务端不会保留上个请求的任何信息), 而 Session 的出现就是为了解决这个问题, 将 Client 端的每次请求都关联起来, 要实现 Session 机制 通常通过 Cookie(cookie 里面保存统一标识符号), URI 附加参数, 或者就是SSL (就是SSL 中的各种属性作为一个Client请求的唯一标识), 而在初始化 ApplicationContext 指定默认的Session追踪机制(URL + COOKIE), 若 Connector 配置了 SSLEnabled, 则将通过 SSL 追踪Session的模式也加入追踪机制里面 (将 ApplicationContext.populateSessionTrackingModes()方法)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316172.html#_labelTop)

## Cookie 概述

Cookie 是在Http传输中存在于Header中的一小撮文本信息(KV), 每次浏览器都会将服务端发送给自己的Cookie信息返回发送给服务端(PS: Cookie的内容存储在浏览器端); 有了这种技术服务端就知道这次请求是谁发送过来的(比如我们这里的Session, 就是基于在Http传输中, 在Cookie里面加入一个全局唯一的标识符号JsessionId来区分是哪个用户的请求)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316172.html#_labelTop)

## Tomcat 中 Cookie 的解析

在 Tomcat 8.0.5 中 Cookie 的解析是通过内部的函数 processCookies() 来进行操作的(其实就是将Http header 的内容直接赋值给 Cookie 对象, Cookie在Header中找name是"Cookie"的数据, 拿出来进行解析), 我们这里主要从 jsessionid 的角度来看一下整个过程是如何触发的, 我们直接看函数 CoyoteAdapter.postParseRequest() 中解析 jsessionId 那部分

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 尝试从 URL, Cookie, SSL 回话中获取请求的 ID, 并将 mapRequired 设置为 false
String sessionID = null;
// 1. 是否支持通过 URI 尾缀 JSessionId 的方式来追踪 Session 的变化 (默认是支持的)
if (request.getServletContext().getEffectiveSessionTrackingModes().contains(SessionTrackingMode.URL)) {
    // 2. 从 URI 尾缀的参数中拿取 jsessionId 的数据 (SessionConfig.getSessionUriParamName 是获取对应cookie的名字, 默认 jsessionId, 可以在 web.xml 里面进行定义)
    sessionID = request.getPathParameter( SessionConfig.getSessionUriParamName(request.getContext()));
    if (sessionID != null) { 
        // 3. 若从 URI 里面拿取了 jsessionId, 则直接进行赋值给 request
        request.setRequestedSessionId(sessionID);
        request.setRequestedSessionURL(true);
    }
}

// Look for session ID in cookies and SSL session
// 4. 通过 cookie 里面获取 JSessionId 的值
parseSessionCookiesId(req, request);   
// 5. 在 SSL 模式下获取 JSessionId 的值                             
parseSessionSslId(request);                                         

/**
 * Parse session id in URL.
 */
protected void parseSessionCookiesId(org.apache.coyote.Request req, Request request) {

    // If session tracking via cookies has been disabled for the current
    // context, don't go looking for a session ID in a cookie as a cookie
    // from a parent context with a session ID may be present which would
    // overwrite the valid session ID encoded in the URL
    Context context = request.getMappingData().context;
    // 1. Tomcat 是否支持 通过 cookie 机制 跟踪 session
    if (context != null && !context.getServletContext()
            .getEffectiveSessionTrackingModes().contains(
                    SessionTrackingMode.COOKIE)) {                      
        return;
    }

    // Parse session id from cookies
     // 2. 获取 Cookie的实际引用对象 (PS: 这里还没有触发 Cookie 解析, 也就是 serverCookies 里面是空数据, 数据还只是存储在 http header 里面)
    Cookies serverCookies = req.getCookies(); 
    // 3. 就在这里出发了 Cookie 解析Header里面的数据 (PS: 其实就是 轮训查找 Header 里面那个 name 是 Cookie 的数据, 拿出来进行解析)    
    int count = serverCookies.getCookieCount();                         
    if (count <= 0) {
        return;
    }

    // 4. 获取 sessionId 的名称 JSessionId
    String sessionCookieName = SessionConfig.getSessionCookieName(context); 

    for (int i = 0; i < count; i++) {
        // 5. 轮询所有解析出来的 Cookie
        ServerCookie scookie = serverCookies.getCookie(i);      
        // 6. 比较 Cookie 的名称是否是 jsessionId        
        if (scookie.getName().equals(sessionCookieName)) {              
            logger.info("scookie.getName().equals(sessionCookieName)");
            logger.info("Arrays.asList(Thread.currentThread().getStackTrace()):" + Arrays.asList(Thread.currentThread().getStackTrace()));
            // Override anything requested in the URL
            // 7. 是否 jsessionId 还没有解析 (并且只将第一个解析成功的值 set 进去)
            if (!request.isRequestedSessionIdFromCookie()) {            
                // Accept only the first session id cookie
                // 8. 将MessageBytes转成 char
                convertMB(scookie.getValue());        
                // 9. 设置 jsessionId 的值                
                request.setRequestedSessionId(scookie.getValue().toString());
                request.setRequestedSessionCookie(true);
                request.setRequestedSessionURL(false);
                if (log.isDebugEnabled()) {
                    log.debug(" Requested cookie session id is " +
                        request.getRequestedSessionId());
                }
            } else {
                // 10. 若 Cookie 里面存在好几个 jsessionid, 则进行覆盖 set 值
                if (!request.isRequestedSessionIdValid()) {             
                    // Replace the session id until one is valid
                    convertMB(scookie.getValue());
                    request.setRequestedSessionId
                        (scookie.getValue().toString());
                }
            }
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的步骤其实就是依次从 URI, Cookie, SSL 里面进行 jsessionId 的解析, 其中从Cookie里面进行解析是最常用的, 而且 就这个Tomcat版本里面, 从cookie里面解析 jsessionid 藏得比较深, 是由 Cookie.getCookieCount() 来进行触发的, 整个解析的过程其实就是将线程 header 里面的数据依次遍历, 找到 name="Cookie"的数据,拿出来解析字符串(这里就不再叙述了); 程序到这里其实若客户端传 jsessionId 的话, 则服务端已经将其解析出来, 并且set到Request对象里面了, 但是 Session 对象还没有触发创建, 最多也就是查找一下 jsessionId 对应的 Session 在 Manager 里面是否存在。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316172.html#_labelTop)

## tomcat session 设计分析

tomcat session 组件图如下所示，其中 `Context` 对应一个 webapp 应用，每个 webapp 有多个 `HttpSessionListener`， 并且每个应用的 session 是独立管理的，而 session 的创建、销毁由 `Manager` 组件完成，它内部维护了 N 个 `Session` 实例对象。在前面的文章中，我们分析了 `Context` 组件，它的默认实现是 `StandardContext`，它与 `Manager` 是一对一的关系，`Manager` 创建、销毁会话时，需要借助 `StandardContext` 获取 `HttpSessionListener` 列表并进行事件通知，而 `StandardContext` 的后台线程会对 `Manager` 进行过期 Session 的清理工作

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807160517191-830922781.png)

`org.apache.catalina.Manager` 接口的主要方法如下所示，它提供了 `Context`、`org.apache.catalina.SessionIdGenerator`的 getter/setter 接口，以及创建、添加、移除、查找、遍历 `Session` 的 API 接口，此外还提供了 `Session` 持久化的接口（load/unload） 用于加载/卸载会话信息，当然持久化要看不同的实现类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface Manager {
    public Context getContext();
    public void setContext(Context context);
    public SessionIdGenerator getSessionIdGenerator();
    public void setSessionIdGenerator(SessionIdGenerator sessionIdGenerator);
    public void add(Session session);
    public void addPropertyChangeListener(PropertyChangeListener listener);
    public void changeSessionId(Session session);
    public void changeSessionId(Session session, String newId);
    public Session createEmptySession();
    public Session createSession(String sessionId);
    public Session findSession(String id) throws IOException;
    public Session[] findSessions();
    public void remove(Session session);
    public void remove(Session session, boolean update);
    public void removePropertyChangeListener(PropertyChangeListener listener);
    public void unload() throws IOException;
    public void backgroundProcess();
    public boolean willAttributeDistribute(String name, Object value);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

tomcat8.5 提供了 4 种实现，默认使用 `StandardManager`，tomcat 还提供了集群会话的解决方案，但是在实际项目中很少运用

- StandardManager：Manager 默认实现，在内存中管理 session，宕机将导致 session 丢失；但是当调用 Lifecycle 的 start/stop 接口时，将采用 jdk 序列化保存 Session 信息，因此当 tomcat 发现某个应用的文件有变更进行 reload 操作时，这种情况下不会丢失 Session 信息
- DeltaManager：增量 Session 管理器，用于Tomcat集群的会话管理器，某个节点变更 Session 信息都会同步到集群中的所有节点，这样可以保证 Session 信息的实时性，但是这样会带来较大的网络开销
- BackupManager：用于 Tomcat 集群的会话管理器，与DeltaManager不同的是，某个节点变更 Session 信息的改变只会同步给集群中的另一个 backup 节点
- PersistentManager：当会话长时间空闲时，将会把 Session 信息写入磁盘，从而限制内存中的活动会话数量；此外，它还支持容错，会定期将内存中的 Session 信息备份到磁盘

 我们来看下 `StandardManager` 的类图，它也是个 `Lifecycle` 组件，并且 `ManagerBase` 实现了主要的逻辑。

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807160844084-1975582431.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316172.html#_labelTop)

## Tomcat 中 Session 的创建

经过上面的Cookie解析, 则若存在jsessionId的话, 则已经set到Request里面了, 那Session又是何时触发创建的呢? 主要还是代码 **request.getSession()**, 看代码:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class SessionExample extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response)
        throws IOException, ServletException  {
        HttpSession session = request.getSession();
        // other code......
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看getSession():

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 获取 request 对应的 session
public HttpSession getSession() {
    // 这里就是 通过 managerBase.sessions 获取 Session
    Session session = doGetSession(true); 
    if (session == null) {
        return null;
    }
    return session.getSession();
}

// create 代表是否创建 StandardSession
protected Session doGetSession(boolean create) {              

    // There cannot be a session if no context has been assigned yet
    // 1. 检验 StandardContext
    if (context == null) {
        return (null);                                           
    }

    // Return the current session if it exists and is valid
     // 2. 校验 Session 的有效性
    if ((session != null) && !session.isValid()) {              
        session = null;
    }
    if (session != null) {
        return (session);
    }

    // Return the requested session if it exists and is valid
    Manager manager = null;
    if (context != null) {
        //拿到StandardContext 中对应的StandardManager，Context与 Manager 是一对一的关系
        manager = context.getManager();
    }
    if (manager == null)
     {
        return (null);      // Sessions are not supported
    }
    if (requestedSessionId != null) {
        try {        
            // 3. 通过 managerBase.sessions 获取 Session
            // 4. 通过客户端的 sessionId 从 managerBase.sessions 来获取 Session 对象
            session = manager.findSession(requestedSessionId);   
        } catch (IOException e) {
            session = null;
        }
         // 5. 判断 session 是否有效
        if ((session != null) && !session.isValid()) {          
            session = null;
        }
        if (session != null) {
            // 6. session access +1
            session.access();                                    
            return (session);
        }
    }

    // Create a new session if requested and the response is not committed
    // 7. 根据标识是否创建 StandardSession ( false 直接返回)
    if (!create) {
        return (null);                                           
    }
    // 当前的 Context 是否支持通过 cookie 的方式来追踪 Session
    if ((context != null) && (response != null) && context.getServletContext().getEffectiveSessionTrackingModes().contains(SessionTrackingMode.COOKIE) && response.getResponse().isCommitted()) {
        throw new IllegalStateException
          (sm.getString("coyoteRequest.sessionCreateCommitted"));
    }

    // Attempt to reuse session id if one was submitted in a cookie
    // Do not reuse the session id if it is from a URL, to prevent possible
    // phishing attacks
    // Use the SSL session ID if one is present.
    // 8. 到这里其实是没有找到 session, 直接创建 Session 出来
    if (("/".equals(context.getSessionCookiePath()) && isRequestedSessionIdFromCookie()) || requestedSessionSSL ) {
        session = manager.createSession(getRequestedSessionId()); // 9. 从客户端读取 sessionID, 并且根据这个 sessionId 创建 Session
    } else {
        session = manager.createSession(null);
    }

    // Creating a new session cookie based on that session
    if ((session != null) && (getContext() != null)&& getContext().getServletContext().getEffectiveSessionTrackingModes().contains(SessionTrackingMode.COOKIE)) {
        // 10. 根据 sessionId 来创建一个 Cookie
        Cookie cookie = ApplicationSessionCookieConfig.createSessionCookie(context, session.getIdInternal(), isSecure());
        // 11. 最后在响应体中写入 cookie
        response.addSessionCookieInternal(cookie);              
    }

    if (session == null) {
        return null;
    }
    // 12. session access 计数器 + 1
    session.access();                                          
    return session;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看看 **manager.createSession(null****);**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract class ManagerBase extends LifecycleMBeanBase implements Manager {
    //Manager管理着当前Context的所有session
    protected Map<String, Session> sessions = new ConcurrentHashMap<>();
    @Override
    public Session findSession(String id) throws IOException {
        if (id == null) {
            return null;
        }
        //通过JssionId获取session
        return sessions.get(id);
    }
    
    public Session createSession(String sessionId) {
        // 1. 判断 单节点的 Session 个数是否超过限制
        if ((maxActiveSessions >= 0) && (getActiveSessions() >= maxActiveSessions)) {      
            rejectedSessions++;
            throw new TooManyActiveSessionsException(
                    sm.getString("managerBase.createSession.ise"),
                    maxActiveSessions);
        }

        // Recycle or create a Session instance
        // 创建一个 空的 session
        // 2. 创建 Session
        Session session = createEmptySession();                     

        // Initialize the properties of the new session and return it
        // 初始化空 session 的属性
        session.setNew(true);
        session.setValid(true);
        session.setCreationTime(System.currentTimeMillis());
        // 3. StandardSession 最大的默认 Session 激活时间
        session.setMaxInactiveInterval(this.maxInactiveInterval); 
        String id = sessionId;
        // 若没有从 client 端读取到 jsessionId
        if (id == null) {      
            // 4. 生成 sessionId (这里通过随机数来生成)    
            id = generateSessionId();                              
        }
        //这里会将session存入Map<String, Session> sessions = new ConcurrentHashMap<>();
        session.setId(id);
        sessionCounter++;

        SessionTiming timing = new SessionTiming(session.getCreationTime(), 0);
        synchronized (sessionCreationTiming) {
            // 5. 每次创建 Session 都会创建一个 SessionTiming, 并且 push 到 链表 sessionCreationTiming 的最后
            sessionCreationTiming.add(timing); 
            // 6. 并且将 链表 最前面的节点删除        
            sessionCreationTiming.poll();                         
        }      
        // 那这个 sessionCreationTiming 是什么作用呢, 其实 sessionCreationTiming 是用来统计 Session的新建及失效的频率 (好像Zookeeper 里面也有这个的统计方式)    
        return (session);
    }
    
    @Override
    public void add(Session session) {
        //将创建的Seesion存入Map<String, Session> sessions = new ConcurrentHashMap<>();
        sessions.put(session.getIdInternal(), session);
        int size = getActiveSessions();
        if( size > maxActive ) {
            synchronized(maxActiveUpdateLock) {
                if( size > maxActive ) {
                    maxActive = size;
                }
            }
        }
    }
}

@Override
public void setId(String id) {
    setId(id, true);
}

@Override
public void setId(String id, boolean notify) {

    if ((this.id != null) && (manager != null))
        manager.remove(this);

    this.id = id;

    if (manager != null)
        manager.add(this);

    if (notify) {
        tellNew();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

其主要的步骤就是:

```
1. 若 request.Session != null, 则直接返回 (说明同一时刻之前有其他线程创建了Session, 并且赋值给了 request)
2. 若 requestedSessionId != null, 则直接通过 manager 来进行查找一下, 并且判断是否有效
3. 调用 manager.createSession 来创建对应的Session，并将Session存入Manager的Map中
4. 根据 SessionId 来创建 Cookie, 并且将 Cookie 放到 Response 里面
5. 直接返回 Session
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11316172.html#_labelTop)

## Session清理



### Background 线程

前面我们分析了 Session 的创建过程，而 Session 会话是有时效性的，下面我们来看下 tomcat 是如何进行失效检查的。在分析之前，我们先回顾下 `Container` 容器的 Background 线程。

tomcat 所有容器组件，都是继承至 `ContainerBase` 的，包括 `StandardEngine`、`StandardHost`、`StandardContext`、`StandardWrapper`，而 `ContainerBase` 在启动的时候，如果 `backgroundProcessorDelay` 参数大于 0 则会开启 `ContainerBackgroundProcessor` 后台线程，调用自己以及子容器的 `backgroundProcess` 进行一些后台逻辑的处理，和 `Lifecycle` 一样，这个动作是具有传递性的，也就

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190807162732848-246474422.png)

关键代码如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ContainerBase.java

protected synchronized void startInternal() throws LifecycleException {
    // other code......
    // 开启ContainerBackgroundProcessor线程用于处理子容器，默认情况下backgroundProcessorDelay=-1，不会启用该线程
    threadStart();
}

protected class ContainerBackgroundProcessor implements Runnable {
    public void run() {
        // threadDone 是 volatile 变量，由外面的容器控制
        while (!threadDone) {
            try {
                Thread.sleep(backgroundProcessorDelay * 1000L);
            } catch (InterruptedException e) {
                // Ignore
            }
            if (!threadDone) {
                processChildren(ContainerBase.this);
            }
        }
    }

    protected void processChildren(Container container) {
        container.backgroundProcess();
        Container[] children = container.findChildren();
        for (int i = 0; i < children.length; i++) {
            // 如果子容器的 backgroundProcessorDelay 参数小于0，则递归处理子容器
            // 因为如果该值大于0，说明子容器自己开启了线程处理，因此父容器不需要再做处理
            if (children[i].getBackgroundProcessorDelay() <= 0) {
                processChildren(children[i]);
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### Session 检查

`backgroundProcessorDelay` 参数默认值为 `-1`，单位为秒，即默认不启用后台线程，而 tomcat 的 Container 容器需要开启线程处理一些后台任务，比如监听 jsp 变更、tomcat 配置变动、Session 过期等等，因此 `StandardEngine` 在构造方法中便将 `backgroundProcessorDelay` 参数设为 10（当然可以在 `server.xml` 中指定该参数），即每隔 10s 执行一次。那么这个线程怎么控制生命周期呢？我们注意到 `ContainerBase` 有个 `threadDone` 变量，用 `volatile` 修饰，如果调用 Container 容器的 stop 方法该值便会赋值为 false，那么该后台线程也会退出循环，从而结束生命周期。另外，有个地方需要注意下，父容器在处理子容器的后台任务时，需要判断子容器的 `backgroundProcessorDelay` 值，只有当其小于等于 0 才进行处理，因为如果该值大于0，子容器自己会开启线程自行处理，这时候父容器就不需要再做处理了

前面分析了容器的后台线程是如何调度的，下面我们重点来看看 webapp 这一层，以及 `StandardManager` 是如何清理过期会话的。`StandardContext` 重写了 `backgroundProcess` 方法，除了对子容器进行处理之外，还会对一些缓存信息进行清理，关键代码如下所示：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
StandardContext.java

@Override
public void backgroundProcess() {
    if (!getState().isAvailable())
        return;
    // 热加载 class，或者 jsp
    Loader loader = getLoader();
    if (loader != null) {
        loader.backgroundProcess();
    }
    // 清理过期Session
    Manager manager = getManager();
    if (manager != null) {
        manager.backgroundProcess();
    }
    // 清理资源文件的缓存
    WebResourceRoot resources = getResources();
    if (resources != null) {
        resources.backgroundProcess();
    }
    // 清理对象或class信息缓存
    InstanceManager instanceManager = getInstanceManager();
    if (instanceManager instanceof DefaultInstanceManager) {
        ((DefaultInstanceManager)instanceManager).backgroundProcess();
    }
    // 调用子容器的 backgroundProcess 任务
    super.backgroundProcess();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`StandardContext` 重写了 `backgroundProcess` 方法，在调用子容器的后台任务之前，还会调用 `Loader`、`Manager`、`WebResourceRoot`、`InstanceManager` 的后台任务，这里我们只关心 `Manager` 的后台任务。弄清楚了 `StandardManager` 的来龙去脉之后，我们接下来分析下具体的逻辑。

`StandardManager` 继承至 `ManagerBase`，它实现了主要的逻辑，关于 Session 清理的代码如下所示。backgroundProcess 默认是每隔10s调用一次，但是在 `ManagerBase` 做了取模处理，默认情况下是 60s 进行一次 Session 清理。tomcat 对 Session 的清理并没有引入时间轮，因为对 Session 的时效性要求没有那么精确，而且除了通知 `SessionListener`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ManagerBase.java

public void backgroundProcess() {
    // processExpiresFrequency 默认值为 6，而backgroundProcess默认每隔10s调用一次，也就是说除了任务执行的耗时，每隔 60s 执行一次
    count = (count + 1) % processExpiresFrequency;
    if (count == 0) // 默认每隔 60s 执行一次 Session 清理
        processExpires();
}

/**
 * 单线程处理，不存在线程安全问题
 */
public void processExpires() {
    long timeNow = System.currentTimeMillis();
    Session sessions[] = findSessions();    // 获取所有的 Session
    int expireHere = 0 ;
    for (int i = 0; i < sessions.length; i++) {
        // Session 的过期是在 isValid() 里面处理的
        if (sessions[i]!=null && !sessions[i].isValid()) {
            expireHere++;
        }
    }
    long timeEnd = System.currentTimeMillis();
    // 记录下处理时间
    processingTime += ( timeEnd - timeNow );
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 清理过期 Session

在上面的代码，我们并没有看到太多的过期处理，只是调用了 `sessions[i].isValid()`，原来清理动作都在这个方法里面处理的，相当的隐晦。在 `StandardSession#isValid()` 方法中，如果 `now - thisAccessedTime >= maxInactiveInterval`则判定当前 Session 过期了，而这个 `thisAccessedTime` 参数在每次访问都会进行更新

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean isValid() {
    // other code......
    // 如果指定了最大不活跃时间，才会进行清理，这个时间是 Context.getSessionTimeout()，默认是30分钟
    if (maxInactiveInterval > 0) {
        int timeIdle = (int) (getIdleTimeInternal() / 1000L);
        if (timeIdle >= maxInactiveInterval) {
            expire(true);
        }
    }
    return this.isValid;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

而 `expire` 方法处理的逻辑较繁锁，下面我用伪代码简单地描述下核心的逻辑，由于这个步骤可能会有多线程进行操作，因此使用 `synchronized` 对当前 Session 对象加锁，还做了双重校验，避免重复处理过期 Session。它还会向 Container 容器发出事件通知，还会调用 `HttpSessionListener` 进行事件通知，这个也就是我们 web 应用开发的 `HttpSessionListener` 了。由于 `Manager` 中维护了 `Session` 对象，因此还要将其从 `Manager` 移除。Session 最重要的功能就是存储数据了，可能存在强引用，而导致 Session 无法被 gc 回收，因此还要移除内部的 key/value 数据。由此可见，tomcat 编码的严谨性了，稍有不慎将可能出现并发问题，以及出现内存泄露

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void expire(boolean notify) {
    //1、校验 isValid 值，如果为 false 直接返回，说明已经被销毁了
    synchronized (this) {   // 加锁
        //2、双重校验 isValid 值，避免并发问题
        Context context = manager.getContext();
        if (notify) {   
            Object listeners[] = context.getApplicationLifecycleListeners();
            HttpSessionEvent event = new HttpSessionEvent(getSession());
            for (int i = 0; i < listeners.length; i++) {
            //3、判断是否为 HttpSessionListener，不是则继续循环
            //4、向容器发出Destory事件，并调用 HttpSessionListener.sessionDestroyed() 进行通知
            context.fireContainerEvent("beforeSessionDestroyed", listener);
            listener.sessionDestroyed(event);
            context.fireContainerEvent("afterSessionDestroyed", listener);
        }
        //5、从 manager 中移除该  session
        //6、向 tomcat 的 SessionListener 发出事件通知，非 HttpSessionListener
        //7、清除内部的 key/value，避免因为强引用而导致无法回收 Session 对象
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

由前面的分析可知，tomcat 会根据时间戳清理过期 Session，那么 tomcat 又是如何更新这个时间戳呢？ tomcat 在处理完请求之后，会对 `Request` 对象进行回收，并且会对 Session 信息进行清理，而这个时候会更新 `thisAccessedTime`、`lastAccessedTime` 时间戳。此外，我们通过调用 `request.getSession()` 这个 API 时，在返回 Session 时会调用 `Session#access()` 方法，也会更新 `thisAccessedTime` 时间戳。这样一来，每次请求都会更新时间戳，可以保证 Session 的鲜活时间。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
org.apache.catalina.connector.Request.java

protected void recycleSessionInfo() {
    if (session != null) {  
        session.endAccess();    // 更新时间戳
    }
    // 回收 Request 对象的内部信息
    session = null;
    requestedSessionCookie = false;
    requestedSessionId = null;
    requestedSessionURL = false;
    requestedSessionSSL = false;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**org.apache.catalina.session.StandardSession.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void endAccess() {
    isNew = false;
    if (LAST_ACCESS_AT_START) {     // 可以通过系统参数改变该值，默认为false
        this.lastAccessedTime = this.thisAccessedTime;
        this.thisAccessedTime = System.currentTimeMillis();
    } else {
        this.thisAccessedTime = System.currentTimeMillis();
        this.lastAccessedTime = this.thisAccessedTime;
    }
}

public void access() {
    this.thisAccessedTime = System.currentTimeMillis();
}
```