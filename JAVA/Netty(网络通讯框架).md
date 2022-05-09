

# Netty源码

## Netty简介

### Netty是什么

大概用Netty的，无论新手还是老手，都知道它是一个“网络通讯框架”。所谓框架，基本上都是一个作用：基于底层API，提供更便捷的编程模型。那么”通讯框架”到底做了什么事情呢？回答这个问题并不太容易，我们不妨反过来看看，不使用netty，直接基于NIO编写网络程序，你需要做什么(以Server端TCP连接为例，这里我们使用Reactor模型)：

1. 监听端口，建立Socket连接
2. 建立线程，处理内容
   1. 读取Socket内容，并对协议进行解析
   2. 进行逻辑处理
   3. 回写响应内容
   4. 如果是多次交互的应用(SMTP、FTP)，则需要保持连接多进行几次交互
3. 关闭连接

建立线程是一个比较耗时的操作，同时维护线程本身也有一些开销，所以我们会需要多线程机制，幸好JDK已经有很方便的多线程框架了，这里我们不需要花很多心思。 此外，因为TCP连接的特性，我们还要使用连接池来进行管理：

1. 建立TCP连接是比较耗时的操作，对于频繁的通讯，保持连接效果更好
2. 对于并发请求，可能需要建立多个连接
3. 维护多个连接后，每次通讯，需要选择某一可用连接
4. 连接超时和关闭机制

想想就觉得很复杂了！实际上，基于NIO直接实现这部分东西，即使是老手也容易出现错误，而使用Netty之后，你只需要关注逻辑处理部分就可以了。

### 体验Netty

这里我们引用Netty的example包里的一个例子，一个简单的EchoServer，它接受客户端输入，并将输入原样返回。其主要代码如下：

```
public void run() {
// Configure the server.
ServerBootstrap bootstrap = new ServerBootstrap(
new NioServerSocketChannelFactory(
Executors.newCachedThreadPool(),
Executors.newCachedThreadPool()));

// Set up the pipeline factory.
bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
public ChannelPipeline getPipeline() throws Exception {
return Channels.pipeline(new EchoServerHandler());
}
});

// Bind and start to accept incoming connections.
bootstrap.bind(new InetSocketAddress(port));
}
```

这里`EchoServerHandler`是其业务逻辑的实现者，大致代码如下：

```
public class EchoServerHandler extends SimpleChannelUpstreamHandler {

    @Override
    public void messageReceived(
            ChannelHandlerContext ctx, MessageEvent e) {
        // Send back the received message to the remote peer.
        e.getChannel().write(e.getMessage());
    }
}
```

还是挺简单的，不是吗？

### Netty背后的事件驱动机制

完成了以上一段代码，我们算是与Netty进行了第一次亲密接触。如果想深入学习呢？

首先推荐Netty的官方User Guide：http://netty.io/3.7/guide/。其次，阅读源码是了解一个开源工具非常好的手段，但是Java世界的框架大多追求大而全，功能完备，如果逐个阅读，难免迷失方向，Netty也并不例外。相反，抓住几个重点对象，理解其领域概念及设计思想，从而理清其脉络，相当于打通了任督二脉，以后的阅读就不再困难了。

理解Netty的关键点在哪呢？我觉得，除了NIO的相关知识，另一个就是事件驱动的设计思想。什么叫事件驱动？我们回头看看`EchoServerHandler`的代码，其中的参数：`public void messageReceived(ChannelHandlerContext ctx, MessageEvent e)`，MessageEvent就是一个事件。这个事件携带了一些信息，例如这里`e.getMessage()`就是消息的内容，而`EchoServerHandler`则描述了处理这种事件的方式。一旦某个事件触发，相应的Handler则会被调用，并进行处理。这种事件机制在UI编程里广泛应用，而Netty则将其应用到了网络编程领域。

在Netty里，所有事件都来自`ChannelEvent`接口，这些事件涵盖监听端口、建立连接、读写数据等网络通讯的各个阶段。而事件的处理者就是`ChannelHandler`，这样，不但是业务逻辑，连网络通讯流程中底层的处理，都可以通过实现`ChannelHandler`来完成了。事实上，Netty内部的连接处理、协议编解码、超时等机制，都是通过handler完成的。当博主弄明白其中的奥妙时，不得不佩服这种设计！ 下图描述了Netty进行事件处理的流程。`Channel`是连接的通道，是ChannelEvent的产生者，而`ChannelPipeline`可以理解为ChannelHandler的集合。

## 开启Netty源码之门

理解了Netty的事件驱动机制，我们现在可以来研究Netty的各个模块了。Netty的包结构如下：

```
org
 └── jboss
 └── netty
 ├── bootstrap 配置并启动服务的类
 ├── buffer 缓冲相关类，对NIO Buffer做了一些封装
 ├── channel 核心部分，处理连接
 ├── container 连接其他容器的代码
 ├── example 使用示例
 ├── handler 基于handler的扩展部分，实现协议编解码等附加功能
 ├── logging 日志
 └── util 工具类
```

在这里面，`channel`和`handler`两部分比较复杂。我们不妨与Netty官方的结构图对照一下，来了解其功能。

![Netty](Netty.png)

具体的解释可以看这里：http://netty.io/3.7/guide/#architecture。图中可以看到，除了之前说到的事件驱动机制之外，Netty的核心功能还包括两部分：

- Zero-Copy-Capable Rich Byte Buffer 零拷贝的Buffer。为什么叫零拷贝？因为在数据传输时，最终处理的数据会需要对单个传输层的报文，进行组合或者拆分。NIO原生的ByteBuffer要做到这件事，需要对ByteBuffer内容进行拷贝，产生新的ByteBuffer，而Netty通过提供Composite(组合)和Slice(切分)两种Buffer来实现零拷贝。这部分代码在`org.jboss.netty.buffer`包中。
- Universal Communication API 统一的通讯API。因为Java的Old I/O和New I/O，使用了互不兼容的API，而Netty则提供了统一的API(`org.jboss.netty.channel.Channel`)来封装这两种I/O模型。这部分代码在`org.jboss.netty.channel`包中。

此外，Protocol Support功能通过handler机制实现。

### Netty源码解读（二）Netty中的buffer

感谢网友【**黄亿华】**投递本稿。

上一篇文章我们概要介绍了Netty的原理及结构，下面几篇文章我们开始对Netty的各个模块进行比较详细的分析。Netty的结构最底层是buffer模块，这部分也相对独立，我们就先从buffer讲起。

## What: buffer二三事

buffer中文名又叫缓冲区，按照维基百科的解释，是”在数据传输时，在内存里开辟的一块临时保存数据的区域”。它其实是一种化同步为异步的机制，可以解决数据传输的速率不对等以及不稳定的问题。

根据这个定义，我们可以知道涉及I/O(特别是I/O写)的地方，基本会有buffer的存在。就Java来说，我们非常熟悉的Old I/O–`InputStream`&`OutputStream`系列API，基本都是在内部使用到了buffer。Java课程老师就教过，o`utputStream.write()只将内容写入了buffer，`必须调用o`utputStream.flush()`，才能保证数据写入生效！

而NIO中则直接将buffer这个概念封装成了对象，其中最常用的大概是ByteBuffer了。于是使用方式变为了：将数据写入Buffer，flip()一下，然后将数据读出来。于是，buffer的概念更加深入人心了！

Netty中的buffer也不例外。不同的是，Netty的buffer专为网络通讯而生，所以它又叫ChannelBuffer(好吧其实没有什么因果关系…)。我们下面就来讲讲Netty中的buffer。当然，关于Netty，我们必须讲讲它的所谓”Zero-Copy-Capable”机制。

## When & Where: TCP/IP协议与buffer

TCP/IP协议是目前的主流网络协议。它是一个多层协议，最下层是物理层，最上层是应用层(HTTP协议等)，而在Java开发中，一般只接触TCP以上，即传输层和应用层的内容。这就是Netty的主要应用场景。

TCP报文有个比较大的特点，就是它传输的时候，会先把应用层的数据项拆开成字节，然后按照自己的传输需要，选择合适数量的字节进行传输。什么叫”自己的传输需要”？首先TCP包有最大长度限制，那么太大的数据项肯定是要拆开的。其次因为TCP以及下层协议会附加一些协议头信息，如果数据项太小，那么可能报文大部分都是没有价值的头信息，这样传输是很不划算的。因此有了收集一定数量的小数据，并打包传输的Nagle算法(这个东东在HTTP协议里会很讨厌，Netty里可以用setOption(“tcpNoDelay”, true)关掉它)。

这么说可能太抽象了一点，我们举个例子吧：

发送时，我们这样分3次写入(‘|’表示两个buffer的分隔):

```
   +-----+-----+-----+
   | ABC | DEF | GHI |
   +-----+-----+-----+
```

接收时，可能变成了这样:

```
   +----+-------+---+---+
   | AB | CDEFG | H | I |
   +----+-------+---+---+
```

很好懂吧？可是，说了这么多，跟buffer有个什么关系呢？别急，我们来看下面一部分。

## Why: buffer中的分层思想

我们先回到之前的`messageReceived`方法：

| `1`  | `public` `void` `messageReceived(` |
| ---- | ---------------------------------- |
|      |                                    |

| `2`  | `    ``ChannelHandlerContext ctx, MessageEvent e) {` |
| ---- | ---------------------------------------------------- |
|      |                                                      |

| `3`  | `  ``// Send back the received message to the remote peer.` |
| ---- | ----------------------------------------------------------- |
|      |                                                             |

| `4`  | `  ``transferredBytes.addAndGet(((ChannelBuffer) e.getMessage()).readableBytes());` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `5`  | `  ``e.getChannel().write(e.getMessage());` |
| ---- | ------------------------------------------- |
|      |                                             |

| `6`  | `}`  |
| ---- | ---- |
|      |      |

这里`MessageEvent.getMessage()`默认的返回值是一个`ChannelBuffer`。我们知道，业务中需要的”Message”，其实是一条应用层级别的完整消息，而一般的buffer工作在传输层，与”Message”是不能对应上的。那么这个ChannelBuffer是什么呢？

来一个官方给的图，我想这个答案就很明显了：

[![img](http://static.oschina.net/uploads/space/2013/0925/225747_kDAk_190591.png)](http://static.oschina.net/uploads/space/2013/0925/225747_kDAk_190591.png)

这里可以看到，TCP层HTTP报文被分成了两个ChannelBuffer，这两个Buffer对我们上层的逻辑(HTTP处理)是没有意义的。但是两个ChannelBuffer被组合起来，就成为了一个有意义的HTTP报文，这个报文对应的ChannelBuffer，才是能称之为”Message”的东西。这里用到了一个词”Virtual Buffer”，也就是所谓的”Zero-Copy-Capable Byte Buffer”了。是不是顿时觉得豁然开朗了？

我这里总结一下，**如果要说NIO的Buffer和Netty的ChannelBuffer最大的区别的话，就是前者仅仅是传输上的Buffer，而后者其实是传输Buffer和抽象后的逻辑Buffer的结合。**延伸开来说，NIO仅仅是一个网络传输框架，而Netty是一个网络应用框架，包括网络以及应用的分层结构。

当然，使用`ChannelBuffer`表示”Message”，不失为一个比较实用的方法，但是使用一个对象来表示解码后的Message可能更符合习惯一点。在Netty里，`MessageEvent.getMessage()`是可以存放一个POJO的，这样子抽象程度又高了一些，这个我们在以后讲到`ChannelPipeline`的时候会说到。

## How: Netty中的ChannelBuffer及实现

好了，终于来到了代码实现部分。之所以啰嗦了这么多，因为我觉得，关于”Zero-Copy-Capable Rich Byte Buffer”，理解为什么需要它，比理解它是怎么实现的，可能要更重要一点。

关于代码阅读，我想可能很多朋友跟我一样，喜欢”顺藤摸瓜”式读代码–找到一个入口，然后顺着查看它的调用，直到理解清楚。很幸运，`ChannelBuffers`(注意有s!)就是这样一根”藤”，它是所有ChannelBuffer实现类的入口，它提供了很多静态的工具方法来创建不同的Buffer，靠“顺藤摸瓜”式读代码方式，大致能把各种ChannelBuffer的实现类摸个遍。先列一下ChannelBuffer相关类图。

[![img](http://static.oschina.net/uploads/space/2013/0925/081551_v8pK_190591.png)](http://static.oschina.net/uploads/space/2013/0925/081551_v8pK_190591.png)

此外还有`WrappedChannelBuffer`系列也是继承自`AbstractChannelBuffer`，图放到了后面。

### ChannelBuffer中的readerIndex和writerIndex

Netty中的buffer是完全重新实现的，与NIO ByteBuffer与ByteBuffer不同的是，它内部保存了一个读指针readerIndex和一个写指针writerIndex，可以同时进行读和写，而不需要使用flip()进行读写切换。AbstactChannelBuffer类里面包含了主要的读写逻辑，贴一段代码，让大家能看的更明白一点：

| `01` | `public` `void` `writeByte(``int` `value) {` |
| ---- | -------------------------------------------- |
|      |                                              |

| `02` | `setByte(writerIndex ++, value);` |
| ---- | --------------------------------- |
|      |                                   |

| `03` | `}`  |
| ---- | ---- |
|      |      |

| `04` |      |
| ---- | ---- |
|      |      |

| `05` | `public` `byte` `readByte() {` |
| ---- | ------------------------------ |
|      |                                |

| `06` | `if` `(readerIndex == writerIndex) {` |
| ---- | ------------------------------------- |
|      |                                       |

| `07` | `throw` `new` `IndexOutOfBoundsException(``"Readable byte limit exceeded: "` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `08` | `+ readerIndex);` |
| ---- | ----------------- |
|      |                   |

| `09` | `}`  |
| ---- | ---- |
|      |      |

| `10` | `return` `getByte(readerIndex ++);` |
| ---- | ----------------------------------- |
|      |                                     |

| `11` | `}`  |
| ---- | ---- |
|      |      |

| `12` |      |
| ---- | ---- |
|      |      |

| `13` | `public` `int` `writableBytes() {` |
| ---- | ---------------------------------- |
|      |                                    |

| `14` | `return` `capacity() - writerIndex;` |
| ---- | ------------------------------------ |
|      |                                      |

| `15` | `}`  |
| ---- | ---- |
|      |      |

| `16` |      |
| ---- | ---- |
|      |      |

| `17` | `public` `int` `readableBytes() {` |
| ---- | ---------------------------------- |
|      |                                    |

| `18` | `return` `writerIndex - readerIndex;` |
| ---- | ------------------------------------- |
|      |                                       |

| `19` | `}`  |
| ---- | ---- |
|      |      |

这里readerIndex总是小于writerIndex。我觉得这样的方式非常自然，比单指针与flip()要更加好理解一些。AbstactChannelBuffer还有两个相应的mark指针`markedReaderIndex`和`markedWriterIndex`，跟NIO的原理一样，作标记用，这里不再赘述了。

### 字节序Endianness与HeapChannelBuffer

HeapChannelBuffer是最常用的Buffer，跟NIO HeapByteBuffer作用相当，其底层也是一个byte[]。

HeapChannelBuffer有两个子类：`BigEndianHeapChannelBuffer`和`LittleEndianHeapChannelBuffer。`这里有个很基础的概念：字节序(ByteOrder/Endianness)。字节序规定了多于一个字节的数字(int啊long什么的)，如何在内存中表示。BIG_ENDIAN(大端序)表示高位在前，按照大端序，整型数`12`会被存储为`0 0 0 12这样`四个字节，而LITTLE_ENDIAN则正好相反。可能搞C/C++的程序员对这个会比较熟悉，而Javaer则比较陌生一点，因为Java已经把内存给管理好了。但是在网络编程方面，根据协议的不同，不同的字节序也可能会被用到。目前大部分协议还是采用大端序，可参考[RFC1700](http://tools.ietf.org/html/rfc1700)。

了解了这些知识，我们也很容易就知道为什么会有`BigEndianHeapChannelBuffer`和`LittleEndianHeapChannelBuffer`了。

### DynamicChannelBuffer

DynamicChannelBuffer是一个很方便的Buffer，之所以叫Dynamic是因为它的长度会根据内容的长度来扩充，你可以像使用ArrayList一样，无须关心其容量。DynamicChannelBuffer实现自动扩容的核心在于`ensureWritableBytes`方法，算法很简单：在写入前做容量检查，容量不够时，新建一个容量x2的buffer，跟ArrayList的扩容是相同的。贴一段代码吧(为了代码易懂，这里我删掉了一些边界检查，只保留主逻辑)：

| `01` | `public` `void` `writeByte(``int` `value) {` |
| ---- | -------------------------------------------- |
|      |                                              |

| `02` | `  ``ensureWritableBytes(``1``);` |
| ---- | --------------------------------- |
|      |                                   |

| `03` | `  ``super``.writeByte(value);` |
| ---- | ------------------------------- |
|      |                                 |

| `04` | `}`  |
| ---- | ---- |
|      |      |

| `05` |      |
| ---- | ---- |
|      |      |

| `06` | `public` `void` `ensureWritableBytes(``int` `minWritableBytes) {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `07` | `  ``if` `(minWritableBytes <= writableBytes()) {` |
| ---- | -------------------------------------------------- |
|      |                                                    |

| `08` | `    ``return``;` |
| ---- | ----------------- |
|      |                   |

| `09` | `  ``}` |
| ---- | ------- |
|      |         |

| `10` |      |
| ---- | ---- |
|      |      |

| `11` | `  ``int` `newCapacity = capacity();` |
| ---- | ------------------------------------- |
|      |                                       |

| `12` | `  ``int` `minNewCapacity = writerIndex() + minWritableBytes;` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `13` | `  ``while` `(newCapacity < minNewCapacity) {` |
| ---- | ---------------------------------------------- |
|      |                                                |

| `14` | `    ``newCapacity <<= ``1``;` |
| ---- | ------------------------------ |
|      |                                |

| `15` | `  ``}` |
| ---- | ------- |
|      |         |

| `16` |      |
| ---- | ---- |
|      |      |

| `17` | `  ``ChannelBuffer newBuffer = factory().getBuffer(order(), newCapacity);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `18` | `  ``newBuffer.writeBytes(buffer, ``0``, writerIndex());` |
| ---- | --------------------------------------------------------- |
|      |                                                           |

| `19` | `  ``buffer = newBuffer;` |
| ---- | ------------------------- |
|      |                           |

| `20` | `}`  |
| ---- | ---- |
|      |      |

### CompositeChannelBuffer

`CompositeChannelBuffer`是由多个ChannelBuffer组合而成的，可以看做一个整体进行读写。这里有一个技巧：CompositeChannelBuffer并不会开辟新的内存并直接复制所有ChannelBuffer内容，而是直接保存了所有ChannelBuffer的引用，并在子ChannelBuffer里进行读写，从而实现了”Zero-Copy-Capable”。来段简略版的代码，应该更能说明其原理：

| `01` | `public` `class` `CompositeChannelBuffer{` |
| ---- | ------------------------------------------ |
|      |                                            |

| `02` |      |
| ---- | ---- |
|      |      |

| `03` | `  ``//components保存所有内部ChannelBuffer` |
| ---- | ------------------------------------------- |
|      |                                             |

| `04` | `  ``private` `ChannelBuffer[] components;` |
| ---- | ------------------------------------------- |
|      |                                             |

| `05` | `  ``//indices记录在整个CompositeChannelBuffer中，每个components的起始位置` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `06` | `  ``private` `int``[] indices;` |
| ---- | -------------------------------- |
|      |                                  |

| `07` | `  ``//缓存上一次读写的componentId` |
| ---- | ----------------------------------- |
|      |                                     |

| `08` | `  ``private` `int` `lastAccessedComponentId;` |
| ---- | ---------------------------------------------- |
|      |                                                |

| `09` |      |
| ---- | ---- |
|      |      |

| `10` | `  ``public` `byte` `getByte(``int` `index) {` |
| ---- | ---------------------------------------------- |
|      |                                                |

| `11` | `    ``//通过indices中记录的位置索引到对应第几个子Buffer` |
| ---- | --------------------------------------------------------- |
|      |                                                           |

| `12` | `    ``int` `componentId = componentId(index);` |
| ---- | ----------------------------------------------- |
|      |                                                 |

| `13` | `    ``return` `components[componentId].getByte(index - indices[componentId]);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `14` | `  ``}` |
| ---- | ------- |
|      |         |

| `15` |      |
| ---- | ---- |
|      |      |

| `16` | `  ``public` `void` `setByte(``int` `index, ``int` `value) {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `17` | `    ``int` `componentId = componentId(index);` |
| ---- | ----------------------------------------------- |
|      |                                                 |

| `18` | `    ``components[componentId].setByte(index - indices[componentId], value);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `19` | `  ``}` |
| ---- | ------- |
|      |         |

| `20` |      |
| ---- | ---- |
|      |      |

| `21` | `}`  |
| ---- | ---- |
|      |      |

查找componentId的算法再次不作介绍了，大家自己实现起来也不会太难。值得一提的是，基于ChannelBuffer连续读写的特性，使用了顺序查找(而不是二分查找)，并且用`lastAccessedComponentId`来进行缓存。

### ByteBufferBackedChannelBuffer

前面说ChannelBuffer是自己的实现的，其实只说对了一半。`ByteBufferBackedChannelBuffer`就是封装了NIO ByteBuffer的类，用于实现堆外内存的Buffer(使用NIO的`DirectByteBuffer`)。当然，其实它也可以放其他的ByteBuffer的实现类。代码实现就不说了，也没啥可说的。

### WrappedChannelBuffer

[![img](http://static.oschina.net/uploads/space/2013/0925/074748_oSkl_190591.png)](http://static.oschina.net/uploads/space/2013/0925/074748_oSkl_190591.png)

`WrappedChannelBuffer`都是几个对已有ChannelBuffer进行包装，完成特定功能的类。代码不贴了，实现都比较简单，列一下功能吧。

| 类名                    | 入口                                               | 功能                                                         |
| ----------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| SlicedChannelBuffer     | ChannelBuffer.slice() ChannelBuffer.slice(int,int) | 某个ChannelBuffer的一部分                                    |
| TruncatedChannelBuffer  | ChannelBuffer.slice() ChannelBuffer.slice(int,int) | 某个ChannelBuffer的一部分， 可以理解为其实位置为0的SlicedChannelBuffer |
| DuplicatedChannelBuffer | ChannelBuffer.duplicate()                          | 与某个ChannelBuffer使用同样的存储， 区别是有自己的index      |
| ReadOnlyChannelBuffer   | ChannelBuffers .unmodifiableBuffer(ChannelBuffer)  | 不可变的buffer                                               |

至此Netty 3.7的buffer部分我们基本了解了，相关内容还是比较简单的，也没有太多费脑细胞的地方。

Netty 4.0之后就不同了，ChannelBuffer改名ByteBuf，成为了单独项目buffer，并且为了性能优化，加入了BufferPool之类的机制，已经变得比较复杂了(本质倒没怎么变)。性能优化是个比较复杂的事情，研究源码时，建议先避开这些东西，了解其整体结构，等到需要深入时再对算法进行细致研究。举个例子，Netty4.0里为了优化，将Map换成了Java 8里6000行的[ConcurrentHashMapV8](https://github.com/netty/netty/blob/master/common/src/main/java/io/netty/util/internal/chmv8/ConcurrentHashMapV8.java)，你们感受一下…

下篇文章我们开始讲Channel。

### Netty源码解读（三）Channel与Pipeline

Channel是理解和使用Netty的核心。Channel的涉及内容较多，这里我使用由浅入深的介绍方法。在这篇文章中，我们主要介绍Channel部分中Pipeline实现机制。为了避免枯燥，借用一下《盗梦空间》的“梦境”概念，希望大家喜欢。



## 一层梦境：Channel实现概览

在Netty里，`Channel`是通讯的载体，而`ChannelHandler`负责Channel中的逻辑处理。

那么`ChannelPipeline`是什么呢？我觉得可以理解为ChannelHandler的容器：一个Channel包含一个ChannelPipeline，所有ChannelHandler都会注册到ChannelPipeline中，并按顺序组织起来。

在Netty中，`ChannelEvent`是数据或者状态的载体，例如传输的数据对应`MessageEvent`，状态的改变对应`ChannelStateEvent`。当对Channel进行操作时，会产生一个ChannelEvent，并发送到`ChannelPipeline`。ChannelPipeline会选择一个ChannelHandler进行处理。这个ChannelHandler处理之后，可能会产生新的ChannelEvent，并流转到下一个ChannelHandler。

[![channel pipeline](http://static.oschina.net/uploads/space/2013/0921/174032_18rb_190591.png)](http://static.oschina.net/uploads/space/2013/0921/174032_18rb_190591.png)

例如，一个数据最开始是一个`MessageEvent`，它附带了一个未解码的原始二进制消息`ChannelBuffer`，然后某个Handler将其解码成了一个数据对象，并生成了一个新的`MessageEvent`，并传递给下一步进行处理。

到了这里，可以看到，其实Channel的核心流程位于`ChannelPipeline`中。于是我们进入ChannelPipeline的深层梦境里，来看看它具体的实现。

## 二层梦境：ChannelPipeline的主流程

Netty的ChannelPipeline包含两条线路：Upstream和Downstream。Upstream对应上行，接收到的消息、被动的状态改变，都属于Upstream。Downstream则对应下行，发送的消息、主动的状态改变，都属于Downstream。`ChannelPipeline`接口包含了两个重要的方法:`sendUpstream(ChannelEvent e)`和`sendDownstream(ChannelEvent e)`，就分别对应了Upstream和Downstream。

对应的，ChannelPipeline里包含的ChannelHandler也包含两类：`ChannelUpstreamHandler`和`ChannelDownstreamHandler`。每条线路的Handler是互相独立的。它们都很简单的只包含一个方法：`ChannelUpstreamHandler.handleUpstream`和`ChannelDownstreamHandler.handleDownstream`。

Netty官方的javadoc里有一张图(`ChannelPipeline`接口里)，非常形象的说明了这个机制(我对原图进行了一点修改，加上了`ChannelSink`，因为我觉得这部分对理解代码流程会有些帮助)：

[![channel pipeline](http://static.oschina.net/uploads/space/2013/1109/075339_Kjw6_190591.png)](http://static.oschina.net/uploads/space/2013/1109/075339_Kjw6_190591.png)

什么叫`ChannelSink`呢？ChannelSink包含一个重要方法`ChannelSink.eventSunk`，可以接受任意ChannelEvent。“sink”的意思是”下沉”，那么”ChannelSink”好像可以理解为”Channel下沉的地方”？实际上，它的作用确实是这样，也可以换个说法：“处于末尾的万能Handler”。最初读到这里，也有些困惑，这么理解之后，就感觉简单许多。只有Downstream包含`ChannelSink`，这里会做一些建立连接、绑定端口等重要操作。为什么UploadStream没有ChannelSink呢？我只能认为，一方面，不符合”sink”的意义，另一方面，也没有什么处理好做的吧！

这里有个值得注意的地方：在一条“流”里，一个`ChannelEvent`并不会主动的”流”经所有的Handler，而是由上一个Handler显式的调用`ChannelPipeline.sendUp(Down)stream`产生，并交给下一个Handler处理。也就是说，每个Handler接收到一个ChannelEvent，并处理结束后，如果需要继续处理，那么它需要调用`sendUp(Down)stream`新发起一个事件。如果它不再发起事件，那么处理就到此结束，即使它后面仍然有Handler没有执行。这个机制可以保证最大的灵活性，当然对Handler的先后顺序也有了更严格的要求。

下面我们从代码层面来对这里面发生的事情进行深入分析，这部分涉及到一些细节，需要打开项目源码，对照来看，会比较有收获。

## 三层梦境：深入ChannelPipeline内部

### DefaultChannelPipeline的内部结构

`ChannelPipeline`的主要的实现代码在`DefaultChannelPipeline`类里。列一下DefaultChannelPipeline的主要字段：

| `1`  | `public` `class` `DefaultChannelPipeline ``implements` `ChannelPipeline {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `2`  |      |
| ---- | ---- |
|      |      |

| `3`  | `  ``private` `volatile` `Channel channel;` |
| ---- | ------------------------------------------- |
|      |                                             |

| `4`  | `  ``private` `volatile` `ChannelSink sink;` |
| ---- | -------------------------------------------- |
|      |                                              |

| `5`  | `  ``private` `volatile` `DefaultChannelHandlerContext head;` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `6`  | `  ``private` `volatile` `DefaultChannelHandlerContext tail;` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `7`  | `  ``private` `final` `Map&lt;String, DefaultChannelHandlerContext&gt; name2ctx =` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `8`  | `    ``new` `HashMap&lt;String, DefaultChannelHandlerContext&gt;(``4``);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `9`  | `}`  |
| ---- | ---- |
|      |      |

这里需要介绍一下`ChannelHandlerContext`这个接口。顾名思义，ChannelHandlerContext保存了Netty与Handler相关的的上下文信息。而咱们这里的`DefaultChannelHandlerContext`，则是对`ChannelHandler`的一个包装。一个`DefaultChannelHandlerContext`内部，除了包含一个`ChannelHandler`，还保存了”next”和”prev”两个指针，从而形成一个双向链表。

因此，在`DefaultChannelPipeline`中，我们看到的是对`DefaultChannelHandlerContext`的引用，而不是对`ChannelHandler`的直接引用。这里包含”head”和”tail”两个引用，分别指向链表的头和尾。而name2ctx则是一个按名字索引DefaultChannelHandlerContext用户的一个map，主要在按照名称删除或者添加ChannelHandler时使用。

### sendUpstream和sendDownstream

前面提到了，`ChannelPipeline`接口的两个重要的方法：`sendUpstream(ChannelEvent e)`和`sendDownstream(ChannelEvent e)`。所有事件的发起都是基于这两个方法进行的。`Channels`类有一系列`fireChannelBound`之类的`fireXXXX`方法，其实都是对这两个方法的facade包装。

下面来看一下这两个方法的实现。先看sendUpstream(对代码做了一些简化，保留主逻辑)：

| `01` | `public` `void` `sendUpstream(ChannelEvent e) {` |
| ---- | ------------------------------------------------ |
|      |                                                  |

| `02` | `  ``DefaultChannelHandlerContext head = getActualUpstreamContext(``this``.head);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `03` | `  ``head.getHandler().handleUpstream(head, e);` |
| ---- | ------------------------------------------------ |
|      |                                                  |

| `04` | `}`  |
| ---- | ---- |
|      |      |

| `05` |      |
| ---- | ---- |
|      |      |

| `06` | `private` `DefaultChannelHandlerContext getActualUpstreamContext(DefaultChannelHandlerContext ctx) {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `07` | `  ``DefaultChannelHandlerContext realCtx = ctx;` |
| ---- | ------------------------------------------------- |
|      |                                                   |

| `08` | `  ``while` `(!realCtx.canHandleUpstream()) {` |
| ---- | ---------------------------------------------- |
|      |                                                |

| `09` | `    ``realCtx = realCtx.next;` |
| ---- | ------------------------------- |
|      |                                 |

| `10` | `    ``if` `(realCtx == ``null``) {` |
| ---- | ------------------------------------ |
|      |                                      |

| `11` | `      ``return` `null``;` |
| ---- | -------------------------- |
|      |                            |

| `12` | `    ``}` |
| ---- | --------- |
|      |           |

| `13` | `  ``}` |
| ---- | ------- |
|      |         |

| `14` | `  ``return` `realCtx;` |
| ---- | ----------------------- |
|      |                         |

| `15` | `}`  |
| ---- | ---- |
|      |      |

这里最终调用了`ChannelUpstreamHandler.handleUpstream`来处理这个ChannelEvent。有意思的是，这里我们看不到任何”将Handler向后移一位”的操作，但是我们总不能每次都用同一个Handler来进行处理啊？实际上，我们更为常用的是`ChannelHandlerContext.handleUpstream`方法(实现是`DefaultChannelHandlerContext.sendUpstream`方法)：

| `1`  | `public` `void` `sendUpstream(ChannelEvent e) {` |
| ---- | ------------------------------------------------ |
|      |                                                  |

| `2`  | `  ``DefaultChannelHandlerContext next = getActualUpstreamContext(``this``.next);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `3`  | `  ``DefaultChannelPipeline.``this``.sendUpstream(next, e);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `4`  | `}`  |
| ---- | ---- |
|      |      |

可以看到，这里最终仍然调用了`ChannelPipeline.sendUpstream`方法，但是它会将Handler指针后移。

我们接下来看看`DefaultChannelHandlerContext.sendDownstream`:

| `01` | `public` `void` `sendDownstream(ChannelEvent e) {` |
| ---- | -------------------------------------------------- |
|      |                                                    |

| `02` | `  ``DefaultChannelHandlerContext prev = getActualDownstreamContext(``this``.prev);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `03` | `  ``if` `(prev == ``null``) {` |
| ---- | ------------------------------- |
|      |                                 |

| `04` | `    ``try` `{` |
| ---- | --------------- |
|      |                 |

| `05` | `      ``getSink().eventSunk(DefaultChannelPipeline.``this``, e);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `06` | `    ``} ``catch` `(Throwable t) {` |
| ---- | ----------------------------------- |
|      |                                     |

| `07` | `      ``notifyHandlerException(e, t);` |
| ---- | --------------------------------------- |
|      |                                         |

| `08` | `    ``}` |
| ---- | --------- |
|      |           |

| `09` | `  ``} ``else` `{` |
| ---- | ------------------ |
|      |                    |

| `10` | `    ``DefaultChannelPipeline.``this``.sendDownstream(prev, e);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `11` | `  ``}` |
| ---- | ------- |
|      |         |

| `12` | `}`  |
| ---- | ---- |
|      |      |

与sendUpstream好像不大相同哦？这里有两点：一是到达末尾时，就如梦境二所说，会调用ChannelSink进行处理；二是这里指针是往前移的，所以我们知道了：

UpstreamHandler是从前往后执行的，DownstreamHandler是从后往前执行的。在ChannelPipeline里添加时需要注意顺序了！

DefaultChannelPipeline里还有些机制，像添加/删除/替换Handler，以及`ChannelPipelineFactory`等，比较好理解，就不细说了。

## 回到现实：Pipeline解决的问题

好了，深入分析完代码，有点头晕了，我们回到最开始的地方，来想一想，Netty的Pipeline机制解决了什么问题？

我认为至少有两点：

一是提供了ChannelHandler的编程模型，基于ChannelHandler开发业务逻辑，基本不需要关心网络通讯方面的事情，专注于编码/解码/逻辑处理就可以了。Handler也是比较方便的开发模式，在很多框架中都有用到。

二是实现了所谓的”Universal Asynchronous API”。这也是Netty官方标榜的一个功能。用过OIO和NIO的都知道，这两套API风格相差极大，要从一个迁移到另一个成本是很大的。即使是NIO，异步和同步编程差距也很大。而Netty屏蔽了OIO和NIO的API差异，通过Channel提供对外接口，并通过ChannelPipeline将其连接起来，因此替换起来非常简单。

[![universal API](http://static.oschina.net/uploads/space/2013/1124/001528_TBb5_190591.jpg)](http://static.oschina.net/uploads/space/2013/1124/001528_TBb5_190591.jpg)

理清了ChannelPipeline的主流程，我们对Channel部分的大致结构算是弄清楚了。可是到了这里，我们依然对一个连接具体怎么处理没有什么概念。在下篇文章，我们会分析一下，在Netty中，究竟是如何处理连接的建立、数据的传输这些事情的。

### Netty源码解读（四）Netty与Reactor模式

[![Reactors](http://static.oschina.net/uploads/space/2014/0208/164000_EQQb_190591.jpg)](http://static.oschina.net/uploads/space/2014/0208/164000_EQQb_190591.jpg)

## 一：Netty、NIO、多线程？

时隔很久终于又更新了！之前一直迟迟未动也是因为积累不够，后面比较难下手。过年期间[@李林锋hw](http://weibo.com/lilinfeng)发布了一个[Netty5.0架构剖析和源码解读 ](http://vdisk.weibo.com/s/C9LV9iVqH13rW/1391437855)，看完也是收获不少。前面的文章我们分析了Netty的结构，这次咱们来分析最错综复杂的一部分-Netty中的多线程以及NIO的应用。

理清NIO与Netty的关系之前，我们必须先要来看看Reactor模式。Netty是一个典型的多线程的Reactor模式的使用，理解了这部分，在宏观上理解Netty的NIO及多线程部分就不会有什么困难了。

本篇文章依然针对Netty 3.7，不过因为也看过一点Netty 5的源码，所以会有一点介绍。



 

## 二：Reactor，反应堆还是核电站？

### 1、Reactor的由来

Reactor是一种广泛应用在服务器端开发的设计模式。Reactor中文大多译为“反应堆”，我当初接触这个概念的时候，就感觉很厉害，是不是它的原理就跟“核反应”差不多？后来才知道其实没有什么关系，从Reactor的兄弟“Proactor”（多译为前摄器）就能看得出来，这两个词的中文翻译其实都不是太好，不够形象。实际上，Reactor模式又有别名“Dispatcher”或者“Notifier”，我觉得这两个都更加能表明它的本质。

那么，Reactor模式究竟是个什么东西呢？这要从事件驱动的开发方式说起。我们知道，对于应用服务器，一个主要规律就是，CPU的处理速度是要远远快于IO速度的，如果CPU为了IO操作（例如从Socket读取一段数据）而阻塞显然是不划算的。好一点的方法是分为多进程或者线程去进行处理，但是这样会带来一些进程切换的开销，试想一个进程一个数据读了500ms，期间进程切换到它3次，但是CPU却什么都不能干，就这么切换走了，是不是也不划算？

这时先驱们找到了事件驱动，或者叫回调的方式，来完成这件事情。这种方式就是，应用业务向一个中间人注册一个回调（event handler），当IO就绪后，就这个中间人产生一个事件，并通知此handler进行处理。这种回调的方式，也体现了“好莱坞原则”（Hollywood principle）-“Don’t call us, we’ll call you”，在我们熟悉的IoC中也有用到。看来软件开发真是互通的！

好了，我们现在来看Reactor模式。在前面事件驱动的例子里有个问题：我们如何知道IO就绪这个事件，谁来充当这个中间人？Reactor模式的答案是：由一个不断等待和循环的单独进程（线程）来做这件事，它接受所有handler的注册，并负责先操作系统查询IO是否就绪，在就绪后就调用指定handler进行处理，这个角色的名字就叫做Reactor。

### 2、Reactor与NIO

Java中的NIO可以很好的和Reactor模式结合。关于NIO中的Reactor模式，我想没有什么资料能比Doug Lea大神（不知道Doug Lea？看看JDK集合包和并发包的作者吧）在[《Scalable IO in Java》](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)解释的更简洁和全面了。NIO中Reactor的核心是`Selector`，我写了一个简单的Reactor示例，这里我贴一个核心的Reactor的循环（这种循环结构又叫做`EventLoop`），剩余代码在[这里](https://github.com/code4craft/netty-learning/tree/master/learning-src/src/main/java/us/codecraft/netty/reactor)。

| `01` | `public` `void` `run() {` |
| ---- | ------------------------- |
|      |                           |

| `02` | `  ``try` `{` |
| ---- | ------------- |
|      |               |

| `03` | `    ``while` `(!Thread.interrupted()) {` |
| ---- | ----------------------------------------- |
|      |                                           |

| `04` | `      ``selector.select();` |
| ---- | ---------------------------- |
|      |                              |

| `05` | `      ``Set selected = selector.selectedKeys();` |
| ---- | ------------------------------------------------- |
|      |                                                   |

| `06` | `      ``Iterator it = selected.iterator();` |
| ---- | -------------------------------------------- |
|      |                                              |

| `07` | `      ``while` `(it.hasNext())` |
| ---- | -------------------------------- |
|      |                                  |

| `08` | `        ``dispatch((SelectionKey) (it.next()));` |
| ---- | ------------------------------------------------- |
|      |                                                   |

| `09` | `      ``selected.clear();` |
| ---- | --------------------------- |
|      |                             |

| `10` | `    ``}` |
| ---- | --------- |
|      |           |

| `11` | `  ``} ``catch` `(IOException ex) { ``/* ... */` |
| ---- | ------------------------------------------------ |
|      |                                                  |

| `12` | `  ``}` |
| ---- | ------- |
|      |         |

| `13` | `}`  |
| ---- | ---- |
|      |      |

### 3、与Reactor相关的其他概念

前面提到了Proactor模式，这又是什么呢？简单来说，Reactor模式里，操作系统只负责通知IO就绪，具体的IO操作（例如读写）仍然是要在业务进程里阻塞的去做的，而Proactor模式则更进一步，由操作系统将IO操作执行好（例如读取，会将数据直接读到内存buffer中），而handler只负责处理自己的逻辑，真正做到了IO与程序处理异步执行。所以我们一般又说Reactor是同步IO，Proactor是异步IO。

关于阻塞和非阻塞、异步和非异步，以及UNIX底层的机制，大家可以看看这篇文章[IO – 同步，异步，阻塞，非阻塞 （亡羊补牢篇）](http://blog.csdn.net/historyasamirror/article/details/5778378)，以及陶辉（《深入理解nginx》的作者）[《高性能网络编程》](http://www.oschina.net/(http://blog.csdn.net/russell_tao/article/details/17452997))的系列。

## 三：由Reactor出发来理解Netty

### 1、多线程下的Reactor

讲了一堆Reactor，我们回到Netty。在《Scalable IO in Java》中讲到了一种多线程下的Reactor模式。在这个模式里，mainReactor只有一个，负责响应client的连接请求，并建立连接，它使用一个NIO Selector；subReactor可以有一个或者多个，每个subReactor都会在一个独立线程中执行，并且维护一个独立的NIO Selector。

这样的好处很明显，因为subReactor也会执行一些比较耗时的IO操作，例如消息的读写，使用多个线程去执行，则更加有利于发挥CPU的运算能力，减少IO等待时间。

[![Multiple Reactors](http://static.oschina.net/uploads/space/2013/1125/130828_uKWD_190591.jpeg)](http://static.oschina.net/uploads/space/2013/1125/130828_uKWD_190591.jpeg)

### 2、Netty中的Reactor与NIO

好了，了解了多线程下的Reactor模式，我们来看看Netty吧（以下部分主要针对NIO，OIO部分更加简单一点，不重复介绍了）。Netty里对应mainReactor的角色叫做“Boss”，而对应subReactor的角色叫做”Worker”。Boss负责分配请求，Worker负责执行，好像也很贴切！以TCP的Server端为例，这两个对应的实现类分别为`NioServerBoss`和`NioWorker`（Server和Client的Worker没有区别，因为建立连接之后，双方就是对等的进行传输了）。

Netty 3.7中Reactor的EventLoop在`AbstractNioSelector.run()`中，它实现了`Runnable`接口。这个类是Netty NIO部分的核心。它的逻辑非常复杂，其中还包括一些对JDK Bug的处理（例如`rebuildSelector`），刚开始读的时候不需要深入那么细节。我精简了大部分代码，保留主干如下：

| `01` | `abstract` `class` `AbstractNioSelector ``implements` `NioSelector {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `02` |      |
| ---- | ---- |
|      |      |

| `03` | `  ``//NIO Selector` |
| ---- | -------------------- |
|      |                      |

| `04` | `  ``protected` `volatile` `Selector selector;` |
| ---- | ----------------------------------------------- |
|      |                                                 |

| `05` |      |
| ---- | ---- |
|      |      |

| `06` | `  ``//内部任务队列` |
| ---- | -------------------- |
|      |                      |

| `07` | `  ``private` `final` `Queue taskQueue = ``new` `ConcurrentLinkedQueue();` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `08` |      |
| ---- | ---- |
|      |      |

| `09` | `  ``//selector循环` |
| ---- | -------------------- |
|      |                      |

| `10` | `  ``public` `void` `run() {` |
| ---- | ----------------------------- |
|      |                               |

| `11` | `    ``for` `(;;) {` |
| ---- | -------------------- |
|      |                      |

| `12` | `      ``try` `{` |
| ---- | ----------------- |
|      |                   |

| `13` | `        ``//处理内部任务队列` |
| ---- | ------------------------------ |
|      |                                |

| `14` | `        ``processTaskQueue();` |
| ---- | ------------------------------- |
|      |                                 |

| `15` | `        ``//处理selector事件对应逻辑` |
| ---- | -------------------------------------- |
|      |                                        |

| `16` | `        ``process(selector);` |
| ---- | ------------------------------ |
|      |                                |

| `17` | `      ``} ``catch` `(Throwable t) {` |
| ---- | ------------------------------------- |
|      |                                       |

| `18` | `        ``try` `{` |
| ---- | ------------------- |
|      |                     |

| `19` | `          ``Thread.sleep(``1000``);` |
| ---- | ------------------------------------- |
|      |                                       |

| `20` | `        ``} ``catch` `(InterruptedException e) {` |
| ---- | -------------------------------------------------- |
|      |                                                    |

| `21` | `          ``// Ignore.` |
| ---- | ------------------------ |
|      |                          |

| `22` | `        ``}` |
| ---- | ------------- |
|      |               |

| `23` | `      ``}` |
| ---- | ----------- |
|      |             |

| `24` | `    ``}` |
| ---- | --------- |
|      |           |

| `25` | `  ``}` |
| ---- | ------- |
|      |         |

| `26` |      |
| ---- | ---- |
|      |      |

| `27` | `  ``private` `void` `processTaskQueue() {` |
| ---- | ------------------------------------------- |
|      |                                             |

| `28` | `    ``for` `(;;) {` |
| ---- | -------------------- |
|      |                      |

| `29` | `      ``final` `Runnable task = taskQueue.poll();` |
| ---- | --------------------------------------------------- |
|      |                                                     |

| `30` | `      ``if` `(task == ``null``) {` |
| ---- | ----------------------------------- |
|      |                                     |

| `31` | `        ``break``;` |
| ---- | -------------------- |
|      |                      |

| `32` | `      ``}` |
| ---- | ----------- |
|      |             |

| `33` | `      ``task.run();` |
| ---- | --------------------- |
|      |                       |

| `34` | `    ``}` |
| ---- | --------- |
|      |           |

| `35` | `  ``}` |
| ---- | ------- |
|      |         |

| `36` |      |
| ---- | ---- |
|      |      |

| `37` | `  ``protected` `abstract` `void` `process(Selector selector) ``throws` `IOException;` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

其中process是主要的处理事件的逻辑，例如在`AbstractNioWorker`中，处理逻辑如下：

| `01` | `protected` `void` `process(Selector selector) ``throws` `IOException {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `02` | `  ``Set selectedKeys = selector.selectedKeys();` |
| ---- | ------------------------------------------------- |
|      |                                                   |

| `03` | `  ``if` `(selectedKeys.isEmpty()) {` |
| ---- | ------------------------------------- |
|      |                                       |

| `04` | `    ``return``;` |
| ---- | ----------------- |
|      |                   |

| `05` | `  ``}` |
| ---- | ------- |
|      |         |

| `06` | `  ``for` `(Iterator i = selectedKeys.iterator(); i.hasNext();) {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `07` | `    ``SelectionKey k = i.next();` |
| ---- | ---------------------------------- |
|      |                                    |

| `08` | `    ``i.remove();` |
| ---- | ------------------- |
|      |                     |

| `09` | `    ``try` `{` |
| ---- | --------------- |
|      |                 |

| `10` | `      ``int` `readyOps = k.readyOps();` |
| ---- | ---------------------------------------- |
|      |                                          |

| `11` | `      ``if` `((readyOps & SelectionKey.OP_READ) != ``0` `|| readyOps == ``0``) {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `12` | `        ``if` `(!read(k)) {` |
| ---- | ----------------------------- |
|      |                               |

| `13` | `          ``// Connection already closed - no need to handle write.` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `14` | `          ``continue``;` |
| ---- | ------------------------- |
|      |                           |

| `15` | `        ``}` |
| ---- | ------------- |
|      |               |

| `16` | `      ``}` |
| ---- | ----------- |
|      |             |

| `17` | `      ``if` `((readyOps & SelectionKey.OP_WRITE) != ``0``) {` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `18` | `        ``writeFromSelectorLoop(k);` |
| ---- | ------------------------------------- |
|      |                                       |

| `19` | `      ``}` |
| ---- | ----------- |
|      |             |

| `20` | `    ``} ``catch` `(CancelledKeyException e) {` |
| ---- | ----------------------------------------------- |
|      |                                                 |

| `21` | `      ``close(k);` |
| ---- | ------------------- |
|      |                     |

| `22` | `    ``}` |
| ---- | --------- |
|      |           |

| `23` |      |
| ---- | ---- |
|      |      |

| `24` | `    ``if` `(cleanUpCancelledKeys()) {` |
| ---- | --------------------------------------- |
|      |                                         |

| `25` | `      ``break``; ``// break the loop to avoid ConcurrentModificationException` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `26` | `    ``}` |
| ---- | --------- |
|      |           |

| `27` | `  ``}` |
| ---- | ------- |
|      |         |

| `28` | `}`  |
| ---- | ---- |
|      |      |

这不就是第二部分提到的selector经典用法了么？

在4.0之后，作者觉得`NioSelector`这个叫法，以及区分`NioBoss`和`NioWorker`的做法稍微繁琐了点，干脆就将这些合并成了`NioEventLoop`，从此这两个角色就不做区分了。我倒是觉得新版本的会更优雅一点。

### 3、Netty中的多线程

下面我们来看Netty的多线程部分。一旦对应的Boss或者Worker启动，就会分配给它们一个线程去一直执行。对应的概念为`BossPool`和`WorkerPool`。对于每个`NioServerSocketChannel`，Boss的Reactor有一个线程，而Worker的线程数由Worker线程池大小决定，但是默认最大不会超过CPU核数*2，当然，这个参数可以通过`NioServerSocketChannelFactory`构造函数的参数来设置。

| `1`  | `public` `NioServerSocketChannelFactory(` |
| ---- | ----------------------------------------- |
|      |                                           |

| `2`  | `    ``Executor bossExecutor, Executor workerExecutor,` |
| ---- | ------------------------------------------------------- |
|      |                                                         |

| `3`  | `    ``int` `workerCount) {` |
| ---- | ---------------------------- |
|      |                              |

| `4`  | `  ``this``(bossExecutor, ``1``, workerExecutor, workerCount);` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| `5`  | `}`  |
| ---- | ---- |
|      |      |

最后我们比较关心一个问题，我们之前`ChannlePipeline`中的ChannleHandler是在哪个线程执行的呢？答案是在Worker线程里执行的，并且会阻塞Worker的EventLoop。例如，在`NioWorker`中，读取消息完毕之后，会触发`MessageReceived`事件，这会使得Pipeline中的handler都得到执行。

| `01` | `protected` `boolean` `read(SelectionKey k) {` |
| ---- | ---------------------------------------------- |
|      |                                                |

| `02` | `  ``....` |
| ---- | ---------- |
|      |            |

| `03` |      |
| ---- | ---- |
|      |      |

| `04` | `  ``if` `(readBytes > ``0``) {` |
| ---- | -------------------------------- |
|      |                                  |

| `05` | `    ``// Fire the event.` |
| ---- | -------------------------- |
|      |                            |

| `06` | `    ``fireMessageReceived(channel, buffer);` |
| ---- | --------------------------------------------- |
|      |                                               |

| `07` | `  ``}` |
| ---- | ------- |
|      |         |

| `08` |      |
| ---- | ---- |
|      |      |

| `09` | `  ``return` `true``;` |
| ---- | ---------------------- |
|      |                        |

| `10` | `}`  |
| ---- | ---- |
|      |      |

可以看到，对于处理事件较长的业务，并不太适合直接放到ChannelHandler中执行。那么怎么处理呢？我们在Handler部分会进行介绍。

最后附上项目github地址，欢迎交流：https://github.com/code4craft/netty-learning

参考资料：

- Scalable IO in Java http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf
- Netty5.0架构剖析和源码解读 http://vdisk.weibo.com/s/C9LV9iVqH13rW/1391437855
- Reactor pattern http://en.wikipedia.org/wiki/Reactor_pattern
- Reactor – An Object Behavioral Pattern for Demultiplexing and Dispatching Handles for Synchronous Events http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf
- 高性能网络编程6–reactor反应堆与定时器管理 http://blog.csdn.net/russell_tao/article/details/17452997
- IO – 同步，异步，阻塞，非阻塞 （亡羊补牢篇）http://blog.csdn.net/historyasamirror/article/details/5778378

题图来自：http://www.worldindustrialreporter.com/france-gives-green-light-to-tokamak-fusion-reactor/





# [Netty源码分析 （一）----- NioEventLoopGroup](https://www.cnblogs.com/java-chen-hao/p/11453562.html)

**正文**

提到Netty首当其冲被提起的肯定是支持它承受高并发的线程模型，说到线程模型就不得不提到`NioEventLoopGroup`这个线程池，接下来进入正题。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453562.html#_labelTop)

## 线程模型

首先来看一段Netty的使用示例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.wrh.server;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public final class SimpleServer {

    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new SimpleServerHandler())
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                        }
                    });

            ChannelFuture f = b.bind(8888).sync();

            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    private static class SimpleServerHandler extends ChannelInboundHandlerAdapter {
        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception {
            System.out.println("channelActive");
        }

        @Override
        public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
            System.out.println("channelRegistered");
        }

        @Override
        public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
            System.out.println("handlerAdded");
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面将分析第一、二行代码，看下NioEventLoopGroup类的构造函数干了些什么。其余的部分将在其他博文中分析。

```
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
```

从代码中可以看到这里使用了两个线程池`bossGroup`和`workerGroup`，那么为什么需要定义两个线程池呢？这就要说到Netty的线程模型了。

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190903181433857-1249885045.png)

 

 

Netty的线程模型被称为Reactor模型，具体如图所示，图上的mainReactor指的就是`bossGroup`，这个线程池处理客户端的连接请求，并将accept的连接注册到subReactor的其中一个线程上；图上的subReactor当然指的就是`workerGroup`，负责处理已建立的客户端通道上的数据读写；图上还有一块ThreadPool是具体的处理业务逻辑的线程池，一般情况下可以复用subReactor，比我的项目中就是这种用法，但官方建议处理一些较为耗时的业务时还是要使用单独的ThreadPool。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453562.html#_labelTop)

## NioEventLoopGroup构造函数

NioEventLoopGroup的构造函数的代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public NioEventLoopGroup() {
    this(0);
}

public NioEventLoopGroup(int nThreads) {
    this(nThreads, null);
}

public NioEventLoopGroup(int nThreads, ThreadFactory threadFactory) {
    this(nThreads, threadFactory, SelectorProvider.provider());
}

public NioEventLoopGroup(
        int nThreads, ThreadFactory threadFactory, final SelectorProvider selectorProvider) {
    super(nThreads, threadFactory, selectorProvider);
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

NioEventLoopGroup类中的构造函数最终都是调用的父类MultithreadEventLoopGroup如下的构造函数：

```
protected MultithreadEventLoopGroup(int nThreads, ThreadFactory threadFactory, Object... args) {
    super(nThreads == 0? DEFAULT_EVENT_LOOP_THREADS : nThreads, threadFactory, args);
}
```

从上面的构造函数可以得到 如果使用`EventLoopGroup workerGroup = new NioEventLoopGroup()`来创建对象，即不指定线程个数，则netty给我们使用默认的线程个数，如果指定则用我们指定的线程个数。

默认线程个数相关的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static {
    DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
            "io.netty.eventLoopThreads", Runtime.getRuntime().availableProcessors() * 2));

    if (logger.isDebugEnabled()) {
        logger.debug("-Dio.netty.eventLoopThreads: {}", DEFAULT_EVENT_LOOP_THREADS);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

而SystemPropertyUtil.getInt函数的功能为：得到系统属性中指定key(这里：key＝”io.netty.eventLoopThreads”)所对应的value，如果获取不到获取失败则返回默认值，这里的默认值为：cpu的核数的２倍。

结论：如果没有设置程序启动参数（或者说没有指定key=”io.netty.eventLoopThreads”的属性值），那么默认情况下线程的个数为cpu的核数乘以2。

继续看，由于MultithreadEventLoopGroup的构造函数是调用的是其父类MultithreadEventExecutorGroup的构造函数，因此，看下此类的构造函数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected MultithreadEventExecutorGroup(int nThreads, ThreadFactory threadFactory, Object... args) {
    if (nThreads <= 0) {
        throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
    }

    if (threadFactory == null) {
        threadFactory = newDefaultThreadFactory();
    }

    children = new SingleThreadEventExecutor[nThreads];
    //根据线程个数是否为2的幂次方，采用不同策略初始化chooser
    if (isPowerOfTwo(children.length)) {
        chooser = new PowerOfTwoEventExecutorChooser();
    } else {
        chooser = new GenericEventExecutorChooser();
    }
        //产生nTreads个NioEventLoop对象保存在children数组中
    for (int i = 0; i < nThreads; i ++) {
        boolean success = false;
        try {
            children[i] = newChild(threadFactory, args);
            success = true;
        } catch (Exception e) {
            // TODO: Think about if this is a good exception type
            throw new IllegalStateException("failed to create a child event loop", e);
        } finally {
                //如果newChild方法执行失败，则对前面执行new成功的几个NioEventLoop进行shutdown处理
            if (!success) {
                for (int j = 0; j < i; j ++) {
                    children[j].shutdownGracefully();
                }

                for (int j = 0; j < i; j ++) {
                    EventExecutor e = children[j];
                    try {
                        while (!e.isTerminated()) {
                            e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                        }
                    } catch (InterruptedException interrupted) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该构造函数干了如下三件事：

１、产生了一个线程工场：threadFactory = newDefaultThreadFactory();

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
MultithreadEventExecutorGroup.java
protected ThreadFactory newDefaultThreadFactory() {
    return new DefaultThreadFactory(getClass());//getClass()为：NioEventLoopGroup.class
}

DefaultThreadFactory.java    
public DefaultThreadFactory(Class<?> poolType) {
    this(poolType, false, Thread.NORM_PRIORITY);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

２、根据线程个数是否为2的幂次方，采用不同策略初始化chooser

```
private static boolean isPowerOfTwo(int val) {
    return (val & -val) == val;
}
```

3、 产生nTreads个NioEventLoop对象保存在children数组中 ，线程都是通过调用newChild方法来产生的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected EventExecutor newChild(
        ThreadFactory threadFactory, Object... args) throws Exception {
    return new NioEventLoop(this, threadFactory, (SelectorProvider) args[0]);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里传给NioEventLoop构造函数的参数为：NioEventLoopGroup、DefaultThreadFactory、SelectorProvider。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453562.html#_labelTop)

## NioEventLoop构造函数分析

既然上面提到来new一个NioEventLoop对象，下面我们就看下这个类以及其父类。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
NioEventLoop(NioEventLoopGroup parent, ThreadFactory threadFactory, SelectorProvider selectorProvider) {
    super(parent, threadFactory, false);
    if (selectorProvider == null) {
        throw new NullPointerException("selectorProvider");
    }
    provider = selectorProvider;
    selector = openSelector();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

继续看父类 SingleThreadEventLoop的构造函数

```
protected SingleThreadEventLoop(EventLoopGroup parent, ThreadFactory threadFactory, boolean addTaskWakesUp) {
    super(parent, threadFactory, addTaskWakesUp);
}
```

又是直接调用来父类SingleThreadEventExecutor的构造函数，继续看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected SingleThreadEventExecutor(
        EventExecutorGroup parent, ThreadFactory threadFactory, boolean addTaskWakesUp) {

    if (threadFactory == null) {
        throw new NullPointerException("threadFactory");
    }

    this.parent = parent;
    this.addTaskWakesUp = addTaskWakesUp;//false

    thread = threadFactory.newThread(new Runnable() {
        @Override
        public void run() {
            boolean success = false;
            updateLastExecutionTime();
            try {
            //调用NioEventLoop类的run方法
                SingleThreadEventExecutor.this.run();
                success = true;
            } catch (Throwable t) {
                logger.warn("Unexpected exception from an event executor: ", t);
            } finally {
                for (;;) {
                    int oldState = STATE_UPDATER.get(SingleThreadEventExecutor.this);
                    if (oldState >= ST_SHUTTING_DOWN || STATE_UPDATER.compareAndSet(
                            SingleThreadEventExecutor.this, oldState, ST_SHUTTING_DOWN)) {
                        break;
                    }
                }
                // Check if confirmShutdown() was called at the end of the loop.
                if (success && gracefulShutdownStartTime == 0) {
                    logger.error(
                            "Buggy " + EventExecutor.class.getSimpleName() + " implementation; " +
                            SingleThreadEventExecutor.class.getSimpleName() + ".confirmShutdown() must be called " +
                            "before run() implementation terminates.");
                }

                try {
                    // Run all remaining tasks and shutdown hooks.
                    for (;;) {
                        if (confirmShutdown()) {
                            break;
                        }
                    }
                } finally {
                    try {
                        cleanup();
                    } finally {
                        STATE_UPDATER.set(SingleThreadEventExecutor.this, ST_TERMINATED);
                        threadLock.release();
                        if (!taskQueue.isEmpty()) {
                            logger.warn(
                                    "An event executor terminated with " +
                                    "non-empty task queue (" + taskQueue.size() + ')');
                        }

                        terminationFuture.setSuccess(null);
                    }
                }
            }
        }
    });

    taskQueue = newTaskQueue();
} 
protected Queue<Runnable> newTaskQueue() {
    return new LinkedBlockingQueue<Runnable>();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

主要干如下两件事：

1、利用ThreadFactory创建来一个Thread，传入了一个Runnable对象，该Runnable重写的run代码比较长，不过重点仅仅是调用NioEventLoop类的run方法。

2、使用LinkedBlockingQueue类初始化taskQueue 。

其中，newThread方法的代码如下：

DefaultThreadFactory.java

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public Thread newThread(Runnable r) {
    Thread t = newThread(new DefaultRunnableDecorator(r), prefix + nextId.incrementAndGet());

    try {
    //判断是否是守护线程，并进行设置
        if (t.isDaemon()) {
            if (!daemon) {
                t.setDaemon(false);
            }
        } else {
            if (daemon) {
                t.setDaemon(true);
            }
        }
            //设置其优先级
        if (t.getPriority() != priority) {
            t.setPriority(priority);
        }
    } catch (Exception ignored) {
        // Doesn't matter even if failed to set.
    }
    return t;
}

protected Thread newThread(Runnable r, String name) {
    return new FastThreadLocalThread(r, name);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

FastThreadLocalThread.java

```
public FastThreadLocalThread(Runnable target, String name) {
    super(target, name);// FastThreadLocalThread extends Thread 
} 
```

到这里，可以看到底层还是借助于类似于Thread thread = new Thread(r)这种方式来创建线程。

关于NioEventLoop对象可以得到的点有，初始化了如下4个属性。

1、NioEventLoopGroup　（在父类SingleThreadEventExecutor中）

2、selector

3、provider

4、thread （在父类SingleThreadEventExecutor中）

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453562.html#_labelTop)

## 总结

关于NioEventLoopGroup，总结如下

１、 如果不指定线程数，则线程数为：CPU的核数*２

２、根据线程个数是否为2的幂次方，采用不同策略初始化chooser

３、产生nThreads个NioEventLoop对象保存在children数组中。

可以理解NioEventLoop就是一个线程，线程NioEventLoop中里面有如下几个属性：

1、NioEventLoopGroup　（在父类SingleThreadEventExecutor中）

2、selector

3、provider

4、thread （在父类SingleThreadEventExecutor中）

更通俗点就是：NioEventLoopGroup就是一个线程池，NioEventLoop就是一个线程。NioEventLoopGroup线程池中有N个NioEventLoop线程。

# [Netty源码分析 （二）----- ServerBootstrap](https://www.cnblogs.com/java-chen-hao/p/11459808.html)

**正文**

BootStrap在netty的应用程序中负责引导服务器和客户端。netty包含了两种不同类型的引导：
\1. 使用服务器的ServerBootStrap，用于接受客户端的连接以及为已接受的连接创建子通道。
\2. 用于客户端的BootStrap，不接受新的连接，并且是在父通道类完成一些操作。

一般服务端的代码如下所示：

**SimpleServer.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * Created by chenhao on 2019/9/4.
 */
public final class SimpleServer {

    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new SimpleServerHandler())
                    .childHandler(new SimpleServerInitializer())
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);

            ChannelFuture f = b.bind(8888).sync();

            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**SimpleServerHandler.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static class SimpleServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelActive");
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelRegistered");
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**SimpleServerInitializer.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class SimpleServerInitializer extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast("framer", new DelimiterBasedFrameDecoder(8192, Delimiters.lineDelimiter()));
        pipeline.addLast("decoder", new StringDecoder());
        pipeline.addLast("encoder", new StringEncoder());
        pipeline.addLast("handler", new SimpleChatServerHandler());

        System.out.println("SimpleChatClient:" + ch.remoteAddress()+"连接上");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在上篇博文（[Netty源码分析 （一）----- NioEventLoopGroup](https://www.cnblogs.com/java-chen-hao/p/11453562.html)）中 剖析了如下的两行代码内部的构造函数中干了些什么。

```
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
```

具体可以见上篇博文，对于如上的两行代码得到的结论是：

１、 如果不指定线程数，则线程数为：CPU的核数*２

２、根据线程个数是否为2的幂次方，采用不同策略初始化chooser

３、产生nThreads个NioEventLoop对象保存在children数组中。

可以理解NioEventLoop就是一个线程，线程NioEventLoop中里面有如下几个属性：

1、NioEventLoopGroup　（在父类SingleThreadEventExecutor中）

2、selector

3、provider

4、thread （在父类SingleThreadEventExecutor中）

更通俗点就是： NioEventLoopGroup就是一个线程池，NioEventLoop就是一个线程。NioEventLoopGroup线程池中有N个NioEventLoop线程。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11459808.html#_labelTop)

## ServerBootstrap类分析

本篇博文将分析如下几行代码里面做了些什么。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .handler(new SimpleServerHandler())
                .childHandler(new SimpleServerInitializer())
                .option(ChannelOption.SO_BACKLOG, 128)
                .childOption(ChannelOption.SO_KEEPALIVE, true);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ServerBootstrap类的继承结构如下：

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190904163641023-1618675023.png)

该类的参数，有必要列出：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final Map<ChannelOption<?>, Object> childOptions = new LinkedHashMap<ChannelOption<?>, Object>();
private final Map<AttributeKey<?>, Object> childAttrs = new LinkedHashMap<AttributeKey<?>, Object>();
private volatile EventLoopGroup childGroup;
private volatile ChannelHandler childHandler; 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

其父类AbstractBootstrap的参数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private volatile EventLoopGroup group;
private volatile ChannelFactory<? extends C> channelFactory;
private volatile SocketAddress localAddress;
private final Map<ChannelOption<?>, Object> options = new LinkedHashMap<ChannelOption<?>, Object>();
private final Map<AttributeKey<?>, Object> attrs = new LinkedHashMap<AttributeKey<?>, Object>();
private volatile ChannelHandler handler;  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面主要看下这个链式设置相关的参数。



### group(bossGroup, workerGroup)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) {
    super.group(parentGroup);
    if (childGroup == null) {
        throw new NullPointerException("childGroup");
    }
    if (this.childGroup != null) {
        throw new IllegalStateException("childGroup set already");
    }
    this.childGroup = childGroup;
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

即将workerGroup保存在 ServerBootstrap对象的childGroup属性上。 bossGroup保存在ServerBootstrap对象的group属性上



### channel(NioServerSocketChannel.class)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    return channelFactory(new BootstrapChannelFactory<C>(channelClass));
} 
public B channelFactory(ChannelFactory<? extends C> channelFactory) {
    if (channelFactory == null) {
        throw new NullPointerException("channelFactory");
    }
    if (this.channelFactory != null) {
        throw new IllegalStateException("channelFactory set already");
    }

    this.channelFactory = channelFactory;
    return (B) this;
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

函数功能：设置父类属性channelFactory 为： BootstrapChannelFactory类的对象。其中这里BootstrapChannelFactory对象中包括一个clazz属性为：NioServerSocketChannel.class，从如下该类的构造函数中可以明显的得到这一点。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static final class BootstrapChannelFactory<T extends Channel> implements ChannelFactory<T> {
    private final Class<? extends T> clazz;

    BootstrapChannelFactory(Class<? extends T> clazz) {
        this.clazz = clazz;
    }

    @Override
    public T newChannel() {
        try {
            return clazz.newInstance();
        } catch (Throwable t) {
            throw new ChannelException("Unable to create Channel from class " + clazz, t);
        }
    }

    @Override
    public String toString() {
        return StringUtil.simpleClassName(clazz) + ".class";
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

并且BootstrapChannelFactory中提供 newChannel()方法，我们可以看到 **clazz.newInstance()，**主要是通过反射来实例化NioServerSocketChannel.class



### handler(new SimpleServerHandler())

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public B handler(ChannelHandler handler) {
    if (handler == null) {
        throw new NullPointerException("handler");
    }
    this.handler = handler;
    return (B) this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注意：这里的handler函数的入参类是我们自己提供的。如下，后面的博文中将会分析这个handler将会在哪里以及何时被调用，这里只需要记住这一点即可

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static class SimpleServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelActive");
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelRegistered");
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### childHandler(new SimpleServerInitializer())

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ServerBootstrap childHandler(ChannelHandler childHandler) {
    if (childHandler == null) {
        throw new NullPointerException("childHandler");
    }
    this.childHandler = childHandler;
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

由最后一句可知，其实就是讲传入的childHandler赋值给ServerBootstrap的childHandler属性。

该函数的主要作用是设置channelHandler来处理客户端的请求的channel的IO。 这里我们一般都用ChannelInitializer这个类的实例或则继承自这个类的实例
这里我是通过新建类SimpleChatServerInitializer继承自ChannelInitializer。具体的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class SimpleChatServerInitializer extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast("framer", new DelimiterBasedFrameDecoder(8192, Delimiters.lineDelimiter()));
        pipeline.addLast("decoder", new StringDecoder());
        pipeline.addLast("encoder", new StringEncoder());
        pipeline.addLast("handler", new SimpleChatServerHandler());

        System.out.println("SimpleChatClient:" + ch.remoteAddress()+"连接上");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们再看看ChannelInitializer这个类的继承图可知ChannelInitializer其实就是继承自ChannelHandler的 

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190904164856411-1683048946.png)

 

可知，这个类其实就是往pipeline中添加了很多的channelHandler。



### 配置ServerBootstrap的option

这里调用的是父类的AbstractBootstrap的option()方法，源码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <T> B option(ChannelOption<T> option, T value) {
    if (option == null) {
        throw new NullPointerException("option");
    }
    if (value == null) {
        synchronized (options) {
            options.remove(option);
        }
    } else {
        synchronized (options) {
            options.put(option, value);
        }
    }
    return (B) this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

其中最重要的一行代码就是：
options.put(option, value);
这里用到了options这个参数，在AbstractBootstrap的定义如下：
private final Map<ChannelOption<?>, Object> options = new LinkedHashMap<ChannelOption<?>, Object>();
可知是私有变量，而且是一个Map集合。这个变量主要是设置TCP连接中的一些可选项,而且这些属性是作用于每一个连接到服务器被创建的channel。



### 配置ServerBootstrap的childOption

这里调用的是父类的ServerBootstrap的childOption()方法，源码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value) {
    if (childOption == null) {
        throw new NullPointerException("childOption");
    }
    if (value == null) {
        synchronized (childOptions) {
            childOptions.remove(childOption);
        }
    } else {
        synchronized (childOptions) {
            childOptions.put(childOption, value);
        }
    }
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个函数功能与option()函数几乎一样，唯一的区别是该属性设定只作用于被acceptor(也就是boss EventLoopGroup)接收之后的channel。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11459808.html#_labelTop)

## 总结

比较简单哈，主要是将我们提供的参数设置到其相应的对象属性中去了。 因为后面会用到如下的几个属性，因此最好知道下，这些属性是何时以及在那里赋值的。

1、group：workerGroup保存在 ServerBootstrap对象的childGroup属性上。 bossGroup保存在ServerBootstrap对象的group属性上

2、channelFactory：BootstrapChannelFactory类的对象（clazz属性为：NioServerSocketChannel.class）

3、handler：SimpleServerHandler

4、childHandler

5、option

6、childOption

# [Netty源码分析 （三）----- 服务端启动源码分析](https://www.cnblogs.com/java-chen-hao/p/11460395.html)

**正文**

本文接着前两篇文章来讲，主要讲服务端类剩下的部分，我们还是来先看看服务端的代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * Created by chenhao on 2019/9/4.
 */
public final class SimpleServer {

    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new SimpleServerHandler())
                    .childHandler(new SimpleServerInitializer())
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);

            ChannelFuture f = b.bind(8888).sync();

            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在前面两篇博文中从源码的角度分析了如下几行代码主要做了哪些工作。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .handler(new SimpleServerHandler())
        .childHandler(new SimpleServerInitializer())
        .option(ChannelOption.SO_BACKLOG, 128)
        .childOption(ChannelOption.SO_KEEPALIVE, true);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

本篇博文将从源码的角度分析`ChannelFuture f = b.bind(8888).sync()` 的内部实现。这样就完成了Netty服务器端启动过程的源码分析。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11460395.html#_labelTop)

## 源码分析ChannelFuture f = b.bind(8888).sync()

**AbstractBootstrap.java**

```
public ChannelFuture bind(int inetPort) {
    return bind(new InetSocketAddress(inetPort));
}
```

我们接着看重载的bind

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ChannelFuture bind(SocketAddress localAddress) {
    validate();//相关参数的检查
    if (localAddress == null) {
        throw new NullPointerException("localAddress");
    }
    return doBind(localAddress);//下面将分析
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该函数主要看两点：validate()和doBind(localAddress)



### validate()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//函数功能：检查相关参数是否设置了
@SuppressWarnings("unchecked")
public B validate() {
    if (group == null) {//这里的group指的是：b.group(bossGroup, workerGroup)代码中的bossGroup
        throw new IllegalStateException("group not set");
    }

    if (channelFactory == null) {
        throw new IllegalStateException("channel or channelFactory not set");
    }
    return (B) this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法主要检查了两个参数，一个是group，一个是channelFactory，在这里可以想一想这两个参数是在哪里以及何时被赋值的？答案是在如下代码块中被赋值的，其中是将bossGroup赋值给了group,将BootstrapChannelFactory赋值给了channelFactory.

```
ServerBootstrap b = new ServerBootstrap();
                b.group(bossGroup, workerGroup)
                        .channel(NioServerSocketChannel.class)
```



### doBind(localAddress)方法

doBind方法的源代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private ChannelFuture doBind(final SocketAddress localAddress) {
    final ChannelFuture regFuture = initAndRegister();//1
    final Channel channel = regFuture.channel();//2
    if (regFuture.cause() != null) {
        return regFuture;
    }

    final ChannelPromise promise;
    if (regFuture.isDone()) {
        promise = channel.newPromise();
        doBind0(regFuture, channel, localAddress, promise);
    } else {
        // Registration future is almost always fulfilled already, but just in case it's not.
        promise = new PendingRegistrationPromise(channel);
        regFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                doBind0(regFuture, channel, localAddress, promise);
            }
        });
    }

    return promise;
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

doBind这个函数是我们要分析的重点，这个函数的主要工作有如下几点：

1、通过initAndRegister()方法得到一个ChannelFuture的实例regFuture。

2、通过regFuture.cause()方法判断是否在执行initAndRegister方法时产生来异常。如果产生来异常，则直接返回，如果没有产生异常则进行第3步。

3、通过regFuture.isDone()来判断initAndRegister方法是否执行完毕，如果执行完毕来返回true，然后调用doBind0进行socket绑定。如果没有执行完毕则返回false进行第4步。

4、regFuture会添加一个ChannelFutureListener监听，当initAndRegister执行完成时，调用operationComplete方法并执行doBind0进行socket绑定。

第3、4点想干的事就是一个：调用doBind0方法进行socket绑定。

下面将分成4部分对每行代码具体做了哪些工作进行详细分析。



### initAndRegister()

该方法的具体代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final ChannelFuture initAndRegister() {
    //结论：这里的channel为一个NioServerSocketChannel对象，具体分析见后面
    final Channel channel = channelFactory().newChannel();//1
    try {
        init(channel);//2
    } catch (Throwable t) {
        channel.unsafe().closeForcibly();
        // as the Channel is not registered yet we need to force the usage of the GlobalEventExecutor
        return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
    }

    ChannelFuture regFuture = group().register(channel);//3
    if (regFuture.cause() != null) {
        if (channel.isRegistered()) {
            channel.close();
        } else {
            channel.unsafe().closeForcibly();
        }
    }
    return regFuture;
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过函数名以及内部调用的函数可以猜测该函数干了两件事情：

1、初始化一个Channel，要想初始化，肯定要先得到一个Channel。

```
final Channel channel = channelFactory().newChannel();//1
init(channel);//2
```

2、将Channel进行注册。

```
ChannelFuture regFuture = group().register(channel);//3
```

下面我们将分析这几行代码内部干来些什么。

#### final Channel channel = channelFactory().newChannel();

在上一篇文章中（[Netty源码分析 （二）----- ServerBootstrap](https://www.cnblogs.com/java-chen-hao/p/11459808.html)）分析中，我们知道b.channel(NioServerSocketChannel.class)的功能为：设置父类属性channelFactory 为： BootstrapChannelFactory类的对象。其中这里BootstrapChannelFactory对象中包括一个clazz属性为：**NioServerSocketChannel.class**

因此，final Channel channel = channelFactory().newChannel();就是调用的BootstrapChannelFactory类中的newChannel()方法，该方法的具体内容为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static final class BootstrapChannelFactory<T extends Channel> implements ChannelFactory<T> {
    private final Class<? extends T> clazz;

    BootstrapChannelFactory(Class<? extends T> clazz) {
        this.clazz = clazz;
    }

    @Override
    public T newChannel() {
        try {
            return clazz.newInstance();
        } catch (Throwable t) {
            throw new ChannelException("Unable to create Channel from class " + clazz, t);
        }
    }

    @Override
    public String toString() {
        return StringUtil.simpleClassName(clazz) + ".class";
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看到这个类，我们可以得到的结论：`final Channel channel = channelFactory().newChannel();`这行代码的作用为通过反射产生来一个NioServerSocketChannel类的实例。

#### **NioServerSocketChannel构造器**

下面将看下NioServerSocketChannel类的构造函数做了哪些工作。

NioServerSocketChannel类的继承体系结构如下：

**![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190904172804277-766349201.png)**

 

 

其无参构造函数如下：

```
public NioServerSocketChannel() {
    this(newSocket(DEFAULT_SELECTOR_PROVIDER));
}
```

无参构造函数中`SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider()`。

函数newSocket的功能为：利用SelectorProvider产生一个SocketChannelImpl对象。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static ServerSocketChannel newSocket(SelectorProvider provider) {
    try {
        return provider.openServerSocketChannel();
    } catch (IOException e) {
        throw new ChannelException(
                "Failed to open a server socket.", e);
    }
} 

public SocketChannel openSocketChannel() throws IOException {
    return new SocketChannelImpl(this);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

无参构造函数通过newSocket函数产生了一个SocketChannelImpl对象

然后调用了如下构造函数，我们继续看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public NioServerSocketChannel(ServerSocketChannel channel) {
    super(null, channel, SelectionKey.OP_ACCEPT);
    config = new NioServerSocketChannelConfig(this, javaChannel().socket());
} 
//父类AbstractNioMessageChannel的构造函数
protected AbstractNioMessageChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent, ch, readInterestOp);
}   
//父类 AbstractNioChannel的构造函数
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    this.readInterestOp = readInterestOp;//SelectionKey.OP_ACCEPT
    try {
        ch.configureBlocking(false);//设置当前的ServerSocketChannel为非阻塞的
    } catch (IOException e) {
        try {
            ch.close();
        } catch (IOException e2) {
            if (logger.isWarnEnabled()) {
                logger.warn(
                        "Failed to close a partially initialized socket.", e2);
            }
        }

        throw new ChannelException("Failed to enter non-blocking mode.", e);
    }
} 
//父类AbstractChannel的构造函数
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    unsafe = newUnsafe();
    pipeline = new DefaultChannelPipeline(this);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

new NioServerSocketChannel()产生一个实例对象时，调用上面这么多构造函数主要干了两件事情：

1、产生来一个SocketChannelImpl类的实例，设置到ch属性中，并设置为非阻塞的。

```
this.ch = ch;
ch.configureBlocking(false);
```

2、设置了config属性

```
config = new NioServerSocketChannelConfig(this, javaChannel().socket()
```

3、设置SelectionKey.OP_ACCEPT事件

```
this.readInterestOp = readInterestOp;//SelectionKey.OP_ACCEPT
```

4、设置unsafe属性

```
@Override
protected AbstractNioUnsafe newUnsafe() {
    return new NioMessageUnsafe();
}
```

主要作用为：用来负责底层的connect、register、read和write等操作。

5、设置pipeline属性

```
pipeline = new DefaultChannelPipeline(this);
```

每个Channel都有自己的pipeline，当有请求事件发生时，pipeline负责调用相应的hander进行处理。

这些属性在后面都会用到，至于NioServerSocketChannel 对象中的unsafe、pipeline属性的具体实现后面进行分析。

结论：**final Channel channel = channelFactory().newChannel();**这行代码的作用为通过反射产生来一个NioServerSocketChannel类的实例，其中这个NioServerSocketChannel类对象有这样几个属性：SocketChannel、NioServerSocketChannelConfig 、SelectionKey.OP_ACCEPT事件、NioMessageUnsafe、DefaultChannelPipeline



### init(channel)

init方法的具体代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
void init(Channel channel) throws Exception {
    //1、设置新接入channel的option
    final Map<ChannelOption<?>, Object> options = options();
    synchronized (options) {
        channel.config().setOptions(options);//NioServerSocketChannelConfig
    }
    //2、设置新接入channel的attr
    final Map<AttributeKey<?>, Object> attrs = attrs();
    synchronized (attrs) {
        for (Entry<AttributeKey<?>, Object> e: attrs.entrySet()) {
            @SuppressWarnings("unchecked")
            AttributeKey<Object> key = (AttributeKey<Object>) e.getKey();
            channel.attr(key).set(e.getValue());
        }
    }
    //3、设置handler到pipeline上
    ChannelPipeline p = channel.pipeline();
    if (handler() != null) {//这里的handler()返回的就是第二部分.handler(new SimpleServerHandler())所设置的SimpleServerHandler
        p.addLast(handler());
    }

    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    synchronized (childOptions) {
        currentChildOptions = childOptions.entrySet().toArray(newOptionArray(childOptions.size()));
    }
    synchronized (childAttrs) {
        currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(childAttrs.size()));
    }
    //p.addLast()向serverChannel的流水线处理器中加入了一个ServerBootstrapAcceptor，从名字上就可以看出来，这是一个接入器，专门接受新请求，把新的请求扔给某个事件循环器
    p.addLast(new ChannelInitializer<Channel>() {
        @Override
        public void initChannel(Channel ch) throws Exception {
            ch.pipeline().addLast(new ServerBootstrapAcceptor(
                    currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
        }
    });
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该函数的功能为：

1、设置channel的options

如果没有设置，则options为空，该属性在ServerBootstrap类中的定义如下

```
Map<ChannelOption<?>, Object> options = new LinkedHashMap<ChannelOption<?>, Object>();
```

options可能如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public <T> boolean setOption(ChannelOption<T> option, T value) {
    validate(option, value);

    if (option == CONNECT_TIMEOUT_MILLIS) {
        setConnectTimeoutMillis((Integer) value);
    } else if (option == MAX_MESSAGES_PER_READ) {
        setMaxMessagesPerRead((Integer) value);
    } else if (option == WRITE_SPIN_COUNT) {
        setWriteSpinCount((Integer) value);
    } else if (option == ALLOCATOR) {
        setAllocator((ByteBufAllocator) value);
    } else if (option == RCVBUF_ALLOCATOR) {
        setRecvByteBufAllocator((RecvByteBufAllocator) value);
    } else if (option == AUTO_READ) {
        setAutoRead((Boolean) value);
    } else if (option == AUTO_CLOSE) {
        setAutoClose((Boolean) value);
    } else if (option == WRITE_BUFFER_HIGH_WATER_MARK) {
        setWriteBufferHighWaterMark((Integer) value);
    } else if (option == WRITE_BUFFER_LOW_WATER_MARK) {
        setWriteBufferLowWaterMark((Integer) value);
    } else if (option == MESSAGE_SIZE_ESTIMATOR) {
        setMessageSizeEstimator((MessageSizeEstimator) value);
    } else {
        return false;
    }

    return true;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2、设置channel的attrs

如果没有设置，则attrs为空，该属性在ServerBootstrap类中的定义如下

```
private final Map<AttributeKey<?>, Object> attrs = new LinkedHashMap<AttributeKey<?>, Object>();
```

3、设置handler到channel的pipeline上

其中，这里的handler为：在博文（[Netty源码分析 （二）----- ServerBootstrap](https://www.cnblogs.com/java-chen-hao/p/11459808.html)）中分析的通过`b.handler(new SimpleServerHandler())`所设置的SimpleServerHandler对象

4、在pipeline上添加来一个ChannelInitializer对象，其中重写来initChannel方法。该方法通过p.addLast()向serverChannel的流水线处理器中加入了一个 ServerBootstrapAcceptor，
从名字上就可以看出来，这是一个接入器，专门接受新请求，把新的请求扔给某个事件循环器

看到这里，我们发现其实init只是初始化了一些基本的配置和属性，以及在pipeline上加入了一个接入器，用来专门接受新连接，并没有启动服务.



### group().register(channel)

回到 initAndRegister 方法中，继续看 config().group().register(channel) 这行代码，config 方法返回了 ServerBootstrapConfig，这个 ServerBootstrapConfig 调用了 group 方法，实际上就是 bossGroup。bossGroup 调用了 register 方法。

前面的分析我们知道group为：NioEvenLoopGroup，其继承MultithreadEventLoopGroup，该类中的register方法如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ChannelFuture register(Channel channel) {
    return next().register(channel);//调用了NioEvenLoop对象中的register方法,NioEventLoop extends SingleThreadEventLoop
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

next()方法的代码如下，其功能为选择下一个NioEventLoop对象。

```
@Override
public EventExecutor next() {
    return chooser.next();//调用MultithreadEventExecutorGroup中的next方法
} 
```

根据线程个数nThreads是否为2的幂次方来选择chooser，其中这两个chooser为： PowerOfTwoEventExecutorChooser、GenericEventExecutorChooser，这两个chooser功能都是一样，只是求余的方式不一样。

next()方法返回的是一个NioEvenLoop对象

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final class PowerOfTwoEventExecutorChooser implements EventExecutorChooser {
    @Override
    public EventExecutor next() {
        return children[childIndex.getAndIncrement() & children.length - 1];//利用2的N次方法的特点，使用&求余更快。
    }
}

private final class GenericEventExecutorChooser implements EventExecutorChooser {
    @Override
    public EventExecutor next() {
        return children[Math.abs(childIndex.getAndIncrement() % children.length)];
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结论：由于NioEventLoopGroup中维护着多个NioEventLoop，next方法回调用chooser策略找到下一个NioEventLoop，并执行该对象的register方法进行注册。

由于NioEventLoop extends SingleThreadEventLoop，NioEventLoop没有重写该方法，因此看 SingleThreadEventLoop类中的register方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ChannelFuture register(Channel channel) {
    return register(channel, new DefaultChannelPromise(channel, this));
}

@Override
public ChannelFuture register(final Channel channel, final ChannelPromise promise) {
    if (channel == null) {
        throw new NullPointerException("channel");
    }
    if (promise == null) {
        throw new NullPointerException("promise");
    }

    channel.unsafe().register(this, promise);
    return promise;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在本博文第1部分的NioServerSocketChannel实例化中设置来unsafe属性，具体是调用如下的方法来设置的，因此这里的`channel.unsafe()`就是NioMessageUnsafe实例。

```
@Override
protected AbstractNioUnsafe newUnsafe() {
    return new NioMessageUnsafe();
}
```

channel.unsafe().register(this, promise)这行代码调用的是AbstractUnsafe类中的register方法，具体代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void register(EventLoop eventLoop, final ChannelPromise promise) {
    if (eventLoop == null) {
        throw new NullPointerException("eventLoop");
    }
    //判断该channel是否已经被注册到EventLoop中
    if (isRegistered()) {
        promise.setFailure(new IllegalStateException("registered to an event loop already"));
        return;
    }
    if (!isCompatible(eventLoop)) {
        promise.setFailure(
                new IllegalStateException("incompatible event loop type: " + eventLoop.getClass().getName()));
        return;
    }

    //1 将eventLoop设置在NioServerSocketChannel上
    AbstractChannel.this.eventLoop = eventLoop;

    //判断当前线程是否为该EventLoop中拥有的线程，如果是，则直接注册，如果不是，则添加一个任务到该线程中
    if (eventLoop.inEventLoop()) {
        register0(promise);
    } else {
        try {
            eventLoop.execute(new OneTimeTask() { //重点
                @Override
                public void run() {
                    register0(promise);//分析
                }
            });
        } catch (Throwable t) {
            logger.warn(
                    "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                    AbstractChannel.this, t);
            closeForcibly();
            closeFuture.setClosed();
            safeSetFailure(promise, t);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的重点是register0(promise)方法。基本逻辑为：

1、通过调用eventLoop.inEventLoop()方法判断当前线程是否为该EventLoop中拥有的线程，如果是，则直接注册，如果不是，说明该EventLoop在等待并没有执行权，则进行第二步。

**AbstractEventExecutor.java**

```
@Override
public boolean inEventLoop() {
    return inEventLoop(Thread.currentThread());
}
```

**SingleThreadEventExecutor.java**

```
@Override
public boolean inEventLoop(Thread thread) {
    return thread == this.thread;
} 
```

2、既然该EventLoop中的线程此时没有执行权，但是我们可以提交一个任务到该线程中，等该EventLoop的线程有执行权的时候就自然而然的会执行此任务，而该任务负责调用register0方法，这样也就达到了调用register0方法的目的。

下面看register0这个方法，具体代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void register0(ChannelPromise promise) {
    try {
        // check if the channel is still open as it could be closed in the mean time when the register
        // call was outside of the eventLoop
        if (!promise.setUncancellable() || !ensureOpen(promise)) {
            return;
        }
        doRegister();
        registered = true;
        safeSetSuccess(promise);
        //执行完，控制台输出：channelRegistered
        pipeline.fireChannelRegistered();
        if (isActive()) { //分析
            pipeline.fireChannelActive();
        }
    } catch (Throwable t) {
        // Close the channel directly to avoid FD leak.
        closeForcibly();
        closeFuture.setClosed();
        safeSetFailure(promise, t);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在上面的代码中，是通过调用doRegister()方法完成NioServerSocketChannel的注册，该方法的具体代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
            selectionKey = javaChannel().register(eventLoop().selector, 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
                // Force the Selector to select now as the "canceled" SelectionKey may still be
                // cached and not removed because no Select.select(..) operation was called yet.
                eventLoop().selectNow();
                selected = true;
            } else {
                // We forced a select operation on the selector before but the SelectionKey is still cached
                // for whatever reason. JDK bug ?
                throw e;
            }
        }
    }
} 

protected SelectableChannel javaChannel() {
    return ch;
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在本博文的第1部分的NioServerSocketChannel的实例化分析中，我们知道这里的javaChannel()方法返回的ch为实例化NioServerSocketChannel时产生的一个SocketChannelImpl类的实例，并设置为非阻塞的，具体见本博文的第1部分。

**selectionKey = javaChannel().register(eventLoop().selector, 0, this);就完成了ServerSocketChannel注册到Selector中。**

回顾下，这里的eventLoop().selector是什么？答案是：KQueueSelectorImpl对象。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
NioEventLoop(NioEventLoopGroup parent, ThreadFactory threadFactory, SelectorProvider selectorProvider) {
    super(parent, threadFactory, false);
    if (selectorProvider == null) {
        throw new NullPointerException("selectorProvider");
    }
    provider = selectorProvider;
    selector = openSelector();
}

private Selector openSelector() {
    final Selector selector;
    try {
        selector = provider.openSelector();
    } catch (IOException e) {
        throw new ChannelException("failed to open a new selector", e);
    }
    //...省略了一部分代码
    return selector;
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ServerSocketChannel注册完之后，接着执行pipeline.fireChannelRegistered方法。

```
public final ChannelPipeline fireChannelRegistered() {
    AbstractChannelHandlerContext.invokeChannelRegistered(this.head);
    return this;
}
```

我们看到**invokeChannelRegistered(****this.head)**传的参数是head,这个head我们再下一篇文章中讲，继续往下看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void invokeChannelRegistered(final AbstractChannelHandlerContext next) {
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRegistered();
    } else {
        executor.execute(new Runnable() {
            public void run() {
                next.invokeChannelRegistered();
            }
        });
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

看**next.invokeChannelRegistered();**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeChannelRegistered() {
    if (this.invokeHandler()) {
        try {
            ((ChannelInboundHandler)this.handler()).channelRegistered(this);
        } catch (Throwable var2) {
            this.notifyHandlerException(var2);
        }
    } else {
        this.fireChannelRegistered();
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接着看看this.handler()，实际上就是head的handler()

```
public ChannelHandler handler() {
    return this;
}
```

返回的是this，那接着看 head中的**channelRegistered****(this****)**

```
public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
    DefaultChannelPipeline.this.invokeHandlerAddedIfNeeded();
    ctx.fireChannelRegistered();
}
```

继续看**ctx.fireChannelRegistered();**

```
public ChannelHandlerContext fireChannelRegistered() {
    invokeChannelRegistered(this.findContextInbound());
    return this;
}
```

我们看看**this.findContextInbound()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext findContextInbound() {
    AbstractChannelHandlerContext ctx = this;

    do {
        ctx = ctx.next;
    } while(!ctx.inbound);

    return ctx;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到 **ctx** **=** **ctx.next; 实际上是从head开始找，找到第一个 inbound 的hander**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void invokeChannelRegistered(final AbstractChannelHandlerContext next) {
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRegistered();
    } else {
        executor.execute(new Runnable() {
            public void run() {
                next.invokeChannelRegistered();
            }
        });
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**最后执行****next.invokeChannelRegistered();**

pipeline中维护了handler链表，还记得之前.handler(new SimpleServerHandler())初始化的handler在本博文的第1.2部分的分析中介绍了此handler被添加到此pipeline中了，通过遍历链表，执行InBound类型handler的channelRegistered方法

因此执行到这里，我们的控制台就回输出：channelRegistered，这行信息。

到这里，我们就将doBind方法final ChannelFuture regFuture = initAndRegister();给分析完了，得到的结论如下：

1、通过反射产生了一个NioServerSocketChannle对象。

2、完成了初始化

3、将NioServerSocketChannel进行了注册。

接下来我们分析doBind方法的剩余部分代码主要做了什么，

源代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private ChannelFuture doBind(final SocketAddress localAddress) {
    final ChannelFuture regFuture = initAndRegister();//1
    final Channel channel = regFuture.channel();//2
    if (regFuture.cause() != null) {
        return regFuture;
    }

    final ChannelPromise promise;
    if (regFuture.isDone()) {
        promise = channel.newPromise();
        doBind0(regFuture, channel, localAddress, promise);
    } else {
        // Registration future is almost always fulfilled already, but just in case it's not.
        promise = new PendingRegistrationPromise(channel);
        regFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                doBind0(regFuture, channel, localAddress, promise);
            }
        });
    }

    return promise;
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### doBind0(regFuture, channel, localAddress, promise);

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static void doBind0(
        final ChannelFuture regFuture, final Channel channel,
        final SocketAddress localAddress, final ChannelPromise promise) {

    // This method is invoked before channelRegistered() is triggered.  Give user handlers a chance to set up
    // the pipeline in its channelRegistered() implementation.
    channel.eventLoop().execute(new Runnable() {
        @Override
        public void run() {
            if (regFuture.isSuccess()) {
                channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
            } else {
                promise.setFailure(regFuture.cause());
            }
        }
    });
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该函数主要是提交了一个Runnable任务到NioEventLoop线程中来进行处理。，这里先看一下NioEventLoop类的execute方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void execute(Runnable task) {
    if (task == null) {
        throw new NullPointerException("task");
    }

    boolean inEventLoop = inEventLoop();//判断当前线程是否为该NioEventLoop所关联的线程，如果是，则添加任务到任务队列中，如果不是，则先启动线程，然后添加任务到任务队列中去
    if (inEventLoop) {
        addTask(task);
    } else {
        startThread();
        addTask(task);
        //如果
        if (isShutdown() && removeTask(task)) {
            reject();
        }
    }

    if (!addTaskWakesUp && wakesUpForTask(task)) {
        wakeup(inEventLoop);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

当提交的任务被线程执行后，则会执行channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE)这行代码，这行代码完成的功能为：实现channel与端口的绑定。

具体如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
AbstractChannel.java    

@Override
public ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise) {
    return pipeline.bind(localAddress, promise);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在该方法中直接调用了pipeline的bind方法，这里的pipeline时DefaultChannelPipeline的实例。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
DefaultChannelPipeline.java 

@Override
public ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise) {
    return tail.bind(localAddress, promise);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在上面方法中直接调用了TailContext实例tail的bind方法，tail在下一篇博文中有详细的介绍。继续看tail实例的bind方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
AbstractChannelHandlerContext.java   

@Override
public ChannelFuture bind(final SocketAddress localAddress, final ChannelPromise promise) {
    //...省略有效性检查

    final AbstractChannelHandlerContext next = findContextOutbound();//
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeBind(localAddress, promise);
    } else {
        safeExecute(executor, new OneTimeTask() {
            @Override
            public void run() {
                next.invokeBind(localAddress, promise);
            }
        }, promise, null);
    }

    return promise;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

此上面bind函数中的这行代码：final AbstractChannelHandlerContext next = findContextOutbound();所完成的任务就是在pipeline所持有的以AbstractChannelHandlerContext为节点的双向链表中从尾节点tail开始向前寻找第一个outbound=true的handler节点。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext findContextOutbound() {
    AbstractChannelHandlerContext ctx = this;
    do {
        ctx = ctx.prev;
    } while (!ctx.outbound);
    return ctx;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在 DefaultChannelPipeline 的构造器中, 会实例化两个对象: head 和 tail, 并形成了双向链表的头和尾。 head 是 HeadContext 的实例, 它实现了 ChannelOutboundHandler 接口和ChannelInboundHandler 接口, 并且它的 outbound 字段为 true.而tail 是 TailContext 的实例，它实现了ChannelInboundHandler 接口，并且其outbound 字段为 false，inbound 字段为true。 基于此在如上的bind函数中调用了 findContextOutbound方法 找到的 AbstractChannelHandlerContext 对象其实就是 head.

继续看，在pipelie的双向链表中找到第一个outbound=true的AbstractChannelHandlerContext节点head后，然后调用此节点的invokeConnect方法，该方法的代码如下:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeBind(SocketAddress localAddress, ChannelPromise promise) {
    try {
        ((ChannelOutboundHandler) handler()).bind(this, localAddress, promise);
    } catch (Throwable t) {
        notifyOutboundHandlerException(t, promise);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

HeadContext类中的handler()方法代码如下：

```
@Override
public ChannelHandler handler() {
    return this;
}
```

该方法返回的是其本身，这是因为HeadContext由于其继承AbstractChannelHandlerContext以及实现了ChannelHandler接口使其具有Context和Handler双重特性。

继续看，看HeadContext类中的bind方法，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void bind(
        ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise)
        throws Exception {
    unsafe.bind(localAddress, promise);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

unsafe这个字段是在HeadContext构造函数中被初始化的，如下：

```
HeadContext(DefaultChannelPipeline pipeline) {
    super(pipeline, null, HEAD_NAME, false, true);
    unsafe = pipeline.channel().unsafe();
}
```

而此构造函数中的pipeline.channel().unsafe()这行代码返回的就是在本博文前面研究NioServerSocketChannel这个类的构造函数中所初始化的一个实例，如下：

```
unsafe = newUnsafe();//newUnsafe()方法返回的是NioMessageUnsafe对象。  
```

接下来看NioMessageUnsafe类中的bind方法（准确来说：该方法在AbstractUnsafe中），该类bind具体方法代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void bind(final SocketAddress localAddress, final ChannelPromise promise) {
        //...省略了部分代码
    boolean wasActive = isActive();
    try {
        doBind(localAddress);//核心代码
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        closeIfClosed();
        return;
    }

    if (!wasActive && isActive()) {
        invokeLater(new OneTimeTask() {
            @Override
            public void run() {
                pipeline.fireChannelActive();
            }
        });
    }

    safeSetSuccess(promise);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的核心代码就是：doBind(localAddress);需要注意的是，此doBind方法是在NioServerSocketChannel类中的doBind方法，不是其他类中的。

NioServerSocketChannel类中的doBind方法代码如下：

```
@Override
protected void doBind(SocketAddress localAddress) throws Exception {
    javaChannel().socket().bind(localAddress, config.getBacklog());
}
```

上面方法中javaChannel()方法返回的是NioServerSocketChannel实例初始化时所产生的Java NIO ServerSocketChannel实例（更具体点为ServerSocketChannelImple实例）。 等价于语句serverSocketChannel.socket().bind(localAddress)完成了指定端口的绑定，这样就开始监听此端口。绑定端口成功后，是这里调用了我们自定义handler的channelActive方法，在绑定之前，isActive()方法返回false，绑定之后返回true。

```
@Override
public boolean isActive() {
    return javaChannel().socket().isBound();
}
```

这样，就进入了如下的if条件的代码块中

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
if (!wasActive && isActive()) {
    invokeLater(new OneTimeTask() {
        @Override
        public void run() {
            pipeline.fireChannelActive();
        }
    });
}    

private void invokeLater(Runnable task) {
    try {
            //省略了部分代码
        eventLoop().execute(task);
    } catch (RejectedExecutionException e) {
        logger.warn("Can't invoke task later as EventLoop rejected it", e);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

进而开始执行 pipeline.fireChannelActive();这行代码 ，这行代码的具体调用链如下所示：

**DefaultChannelPipeline.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ChannelPipeline fireChannelActive() {
    head.fireChannelActive();

    if (channel.config().isAutoRead()) {
        channel.read();
    }

    return this;
}

@Override
public ChannelHandlerContext fireChannelActive() {
    final AbstractChannelHandlerContext next = findContextInbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelActive();
    } else {
        executor.execute(new OneTimeTask() {
            @Override
            public void run() {
                next.invokeChannelActive();
            }
        });
    }
    return this;
}

private void invokeChannelActive() {
    try {
        ((ChannelInboundHandler) handler()).channelActive(this);
    } catch (Throwable t) {
        notifyHandlerException(t);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11460395.html#_labelTop)

## 总结

最后，我们来做下总结，netty启动一个服务所经过的流程
1.设置启动类参数，最重要的就是设置channel
2.创建server对应的channel，创建各大组件，包括ChannelConfig,ChannelId,ChannelPipeline,ChannelHandler,Unsafe等
3.init 初始化这个 NioServerSocketChannel，设置一些attr，option，以及设置子channel的attr，option，给server的channel添加新channel接入器，并触发addHandler事件

4.config().group().register(channel) 通过 ServerBootstrap 的 bossGroup 根据group长度取模得到NioEventLoop ，将 NioServerSocketChannel 注册到 NioEventLoop 中的 selector 上，然后触发 channelRegistered事件

5.调用到jdk底层做端口绑定，并触发active事件

# [Netty源码分析 （四）----- ChannelPipeline](https://www.cnblogs.com/java-chen-hao/p/11466291.html)

**正文**

netty在服务端端口绑定和新连接建立的过程中会建立相应的channel，而与channel的动作密切相关的是pipeline这个概念，pipeline像是可以看作是一条流水线，原始的原料(字节流)进来，经过加工，最后输出

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11466291.html#_labelTop)

## pipeline 初始化

在上一篇文章中，我们已经知道了创建`NioSocketChannel`的时候会将netty的核心组件创建出来

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905183812469-351234498.png)

pipeline是其中的一员，在下面这段代码中被创建

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected DefaultChannelPipeline newChannelPipeline() {
    return new DefaultChannelPipeline(this);
}
```

NioSocketChannel中保存了**pipeline**的引用

**DefaultChannelPipeline**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected DefaultChannelPipeline(Channel channel) {
    this.channel = ObjectUtil.checkNotNull(channel, "channel");
    tail = new TailContext(this);
    head = new HeadContext(this);

    head.next = tail;
    tail.prev = head;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

pipeline中保存了channel的引用，创建完pipeline之后，整个pipeline是这个样子的

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905184353031-260299727.png)

 

pipeline中的每个节点是一个`ChannelHandlerContext`对象，每个context节点保存了它包裹的执行器 `ChannelHandler` 执行操作所需要的上下文，其实就是pipeline，因为pipeline包含了channel的引用，可以拿到所有的context信息

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11466291.html#_labelTop)

## pipeline添加节点

下面是一段非常常见的客户端代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
bootstrap.childHandler(new ChannelInitializer<SocketChannel>() {
     @Override
     public void initChannel(SocketChannel ch) throws Exception {
         ChannelPipeline p = ch.pipeline();
         p.addLast(new Spliter())
         p.addLast(new Decoder());
         p.addLast(new BusinessHandler())
         p.addLast(new Encoder());
     }
});
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先，用一个spliter将来源TCP数据包拆包，然后将拆出来的包进行decoder，传入业务处理器BusinessHandler，业务处理完encoder，输出

整个pipeline结构如下

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905185109739-710843628.png)

 

我用两种颜色区分了一下pipeline中两种不同类型的节点，一个是 `ChannelInboundHandler`，处理inBound事件，最典型的就是读取数据流，加工处理；还有一种类型的Handler是 `ChannelOutboundHandler`, 处理outBound事件，比如当调用`writeAndFlush()`类方法时，就会经过该种类型的handler

不管是哪种类型的handler，其外层对象 `ChannelHandlerContext` 之间都是通过双向链表连接，而区分一个 `ChannelHandlerContext`到底是in还是out，在添加节点的时候我们就可以看到netty是怎么处理的

**DefaultChannelPipeline**

```
@Override
public final ChannelPipeline addLast(ChannelHandler... handlers) {
    return addLast(null, handlers);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final ChannelPipeline addLast(EventExecutorGroup executor, ChannelHandler... handlers) {
    for (ChannelHandler h: handlers) {
        addLast(executor, null, h);
    }
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        // 1.检查是否有重复handler
        checkMultiplicity(handler);
        // 2.创建节点
        newCtx = newContext(group, filterName(name, handler), handler);
        // 3.添加节点
        addLast0(newCtx);
    }
   
    // 4.回调用户方法
    callHandlerAdded0(handler);
    
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里简单地用`synchronized`方法是为了防止多线程并发操作pipeline底层的双向链表

我们还是逐步分析上面这段代码



### 检查是否有重复handler

在用户代码添加一条handler的时候，首先会查看该handler有没有添加过

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static void checkMultiplicity(ChannelHandler handler) {
    if (handler instanceof ChannelHandlerAdapter) {
        ChannelHandlerAdapter h = (ChannelHandlerAdapter) handler;
        if (!h.isSharable() && h.added) {
            throw new ChannelPipelineException(
                    h.getClass().getName() +
                    " is not a @Sharable handler, so can't be added or removed multiple times.");
        }
        h.added = true;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

netty使用一个成员变量`added`标识一个channel是否已经添加，上面这段代码很简单，如果当前要添加的Handler是非共享的，并且已经添加过，那就抛出异常，否则，标识该handler已经添加

由此可见，一个Handler如果是sharable的，就可以无限次被添加到pipeline中，我们客户端代码如果要让一个Handler被共用，只需要加一个@Sharable标注即可，如下

```
@Sharable
public class BusinessHandler {
    
}
```

而如果Handler是sharable的，一般就通过spring的注入的方式使用，不需要每次都new 一个

`isSharable()` 方法正是通过该Handler对应的类是否标注@Sharable来实现的

**ChannelHandlerAdapter**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean isSharable() {
   Class<?> clazz = getClass();
    Map<Class<?>, Boolean> cache = InternalThreadLocalMap.get().handlerSharableCache();
    Boolean sharable = cache.get(clazz);
    if (sharable == null) {
        sharable = clazz.isAnnotationPresent(Sharable.class);
        cache.put(clazz, sharable);
    }
    return sharable;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过反射判断是否有**Sharable.class注解**



### 创建节点

回到主流程，看创建上下文这段代码

```
newCtx = newContext(group, filterName(name, handler), handler);
```

这里我们需要先分析 `filterName(name, handler)` 这段代码，这个函数用于给handler创建一个唯一性的名字

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private String filterName(String name, ChannelHandler handler) {
    if (name == null) {
        return generateName(handler);
    }
    checkDuplicateName(name);
    return name;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

显然，我们传入的name为null，netty就给我们生成一个默认的name，否则，检查是否有重名，检查通过的话就返回

netty创建默认name的规则为 `简单类名#0`，下面我们来看些具体是怎么实现的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static final FastThreadLocal<Map<Class<?>, String>> nameCaches =
        new FastThreadLocal<Map<Class<?>, String>>() {
    @Override
    protected Map<Class<?>, String> initialValue() throws Exception {
        return new WeakHashMap<Class<?>, String>();
    }
};

private String generateName(ChannelHandler handler) {
    // 先查看缓存中是否有生成过默认name
    Map<Class<?>, String> cache = nameCaches.get();
    Class<?> handlerType = handler.getClass();
    String name = cache.get(handlerType);
    // 没有生成过，就生成一个默认name，加入缓存 
    if (name == null) {
        name = generateName0(handlerType);
        cache.put(handlerType, name);
    }

    // 生成完了，还要看默认name有没有冲突
    if (context0(name) != null) {
        String baseName = name.substring(0, name.length() - 1);
        for (int i = 1;; i ++) {
            String newName = baseName + i;
            if (context0(newName) == null) {
                name = newName;
                break;
            }
        }
    }
    return name;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

netty使用一个 `FastThreadLocal`(后面的文章会细说)变量来缓存Handler的类和默认名称的映射关系，在生成name的时候，首先查看缓存中有没有生成过默认name(`简单类名#0`)，如果没有生成，就调用`generateName0()`生成默认name，然后加入缓存

接下来还需要检查name是否和已有的name有冲突，调用`context0()`，查找pipeline里面有没有对应的context

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext context0(String name) {
    AbstractChannelHandlerContext context = head.next;
    while (context != tail) {
        if (context.name().equals(name)) {
            return context;
        }
        context = context.next;
    }
    return null;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`context0()`方法链表遍历每一个 `ChannelHandlerContext`，只要发现某个context的名字与待添加的name相同，就返回该context，最后抛出异常，可以看到，这个其实是一个线性搜索的过程

如果`context0(name) != null` 成立，说明现有的context里面已经有了一个默认name，那么就从 `简单类名#1` 往上一直找，直到找到一个唯一的name，比如`简单类名#3`

如果用户代码在添加Handler的时候指定了一个name，那么要做到事仅仅为检查一下是否有重复

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void checkDuplicateName(String name) {
    if (context0(name) != null) {
        throw new IllegalArgumentException("Duplicate handler name: " + name);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

处理完name之后，就进入到创建context的过程，由前面的调用链得知，`group`为null，因此`childExecutor(group)`也返回null

**DefaultChannelPipeline**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext newContext(EventExecutorGroup group, String name, ChannelHandler handler) {
    return new DefaultChannelHandlerContext(this, childExecutor(group), name, handler);
}

private EventExecutor childExecutor(EventExecutorGroup group) {
    if (group == null) {
        return null;
    }
    //..
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**DefaultChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
DefaultChannelHandlerContext(
        DefaultChannelPipeline pipeline, EventExecutor executor, String name, ChannelHandler handler) {
    super(pipeline, executor, name, isInbound(handler), isOutbound(handler));
    if (handler == null) {
        throw new NullPointerException("handler");
    }
    this.handler = handler;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

构造函数中，`DefaultChannelHandlerContext`将参数回传到父类，保存Handler的引用，进入到其父类

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
AbstractChannelHandlerContext(DefaultChannelPipeline pipeline, EventExecutor executor, String name,
                              boolean inbound, boolean outbound) {
    this.name = ObjectUtil.checkNotNull(name, "name");
    this.pipeline = pipeline;
    this.executor = executor;
    this.inbound = inbound;
    this.outbound = outbound;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

netty中用两个字段来表示这个`channelHandlerContext`属于`inBound`还是`outBound`，或者两者都是，两个boolean是通过下面两个小函数来判断(见上面一段代码)

**DefaultChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static boolean isInbound(ChannelHandler handler) {
    return handler instanceof ChannelInboundHandler;
}

private static boolean isOutbound(ChannelHandler handler) {
    return handler instanceof ChannelOutboundHandler;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过`instanceof`关键字根据接口类型来判断，因此，如果一个Handler实现了两类接口，那么他既是一个inBound类型的Handler，又是一个outBound类型的Handler，比如下面这个类

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905190321364-1063099644.png)

 

常用的，将decode操作和encode操作合并到一起的codec，一般会继承 `MessageToMessageCodec`，而`MessageToMessageCodec`就是继承`ChannelDuplexHandler`

**MessageToMessageCodec**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract class MessageToMessageCodec<INBOUND_IN, OUTBOUND_IN> extends ChannelDuplexHandler {

    protected abstract void encode(ChannelHandlerContext ctx, OUTBOUND_IN msg, List<Object> out)
        throws Exception;

    protected abstract void decode(ChannelHandlerContext ctx, INBOUND_IN msg, List<Object> out)
        throws Exception;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

context 创建完了之后，接下来终于要将创建完毕的context加入到pipeline中去了



### 添加节点

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void addLast0(AbstractChannelHandlerContext newCtx) {
    AbstractChannelHandlerContext prev = tail.prev;
    newCtx.prev = prev; // 1
    newCtx.next = tail; // 2
    prev.next = newCtx; // 3
    tail.prev = newCtx; // 4
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

用下面这幅图可见简单的表示这段过程，说白了，其实就是一个双向链表的插入操作

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905190450361-1239555047.png)

操作完毕，该context就加入到pipeline中

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905190522474-245029138.png)

到这里，pipeline添加节点的操作就完成了，你可以根据此思路掌握所有的addxxx()系列方法



### 回调用户方法

**AbstractChannelHandlerContext**

```
private void callHandlerAdded0(final AbstractChannelHandlerContext ctx) {
    ctx.handler().handlerAdded(ctx);
    ctx.setAddComplete();
}
```

到了第四步，pipeline中的新节点添加完成，于是便开始回调用户代码 `ctx.handler().handlerAdded(ctx);`，常见的用户代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class DemoHandler extends SimpleChannelInboundHandler<...> {
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        // 节点被添加完毕之后回调到此
        // do something
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接下来，设置该节点的状态

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final void setAddComplete() {
    for (;;) {
        int oldState = handlerState;
        if (oldState == REMOVE_COMPLETE || HANDLER_STATE_UPDATER.compareAndSet(this, oldState, ADD_COMPLETE)) {
            return;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

用cas修改节点的状态至：REMOVE_COMPLETE（说明该节点已经被移除） 或者 ADD_COMPLETE

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11466291.html#_labelTop)

## pipeline删除节点

netty 有个最大的特性之一就是Handler可插拔，做到动态编织pipeline，比如在首次建立连接的时候，需要通过进行权限认证，在认证通过之后，就可以将此context移除，下次pipeline在传播事件的时候就就不会调用到权限认证处理器

下面是权限认证Handler最简单的实现，第一个数据包传来的是认证信息，如果校验通过，就删除此Handler，否则，直接关闭连接

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class AuthHandler extends SimpleChannelInboundHandler<ByteBuf> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf data) throws Exception {
        if (verify(authDataPacket)) {
            ctx.pipeline().remove(this);
        } else {
            ctx.close();
        }
    }

    private boolean verify(ByteBuf byteBuf) {
        //...
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

重点就在 `ctx.pipeline().remove(this)` 这段代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final ChannelPipeline remove(ChannelHandler handler) {
    remove(getContextOrDie(handler));
    
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

remove操作相比add简单不少，分为三个步骤：

1.找到待删除的节点
2.调整双向链表指针删除
3.回调用户函数



### 找到待删除的节点

**DefaultChannelPipeline**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext getContextOrDie(ChannelHandler handler) {
    AbstractChannelHandlerContext ctx = (AbstractChannelHandlerContext) context(handler);
    if (ctx == null) {
        throw new NoSuchElementException(handler.getClass().getName());
    } else {
        return ctx;
    }
}

@Override
public final ChannelHandlerContext context(ChannelHandler handler) {
    if (handler == null) {
        throw new NullPointerException("handler");
    }

    AbstractChannelHandlerContext ctx = head.next;
    for (;;) {

        if (ctx == null) {
            return null;
        }

        if (ctx.handler() == handler) {
            return ctx;
        }

        ctx = ctx.next;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里为了找到Handler对应的context，照样是通过依次遍历双向链表的方式，直到某一个context的Handler和当前Handler相同，便找到了该节点



### 调整双向链表指针删除

**DefaultChannelPipeline**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext remove(final AbstractChannelHandlerContext ctx) {
    assert ctx != head && ctx != tail;

    synchronized (this) {
        // 2.调整双向链表指针删除
        remove0(ctx);
    }
    // 3.回调用户函数
    callHandlerRemoved0(ctx);
    return ctx;
}

private static void remove0(AbstractChannelHandlerContext ctx) {
    AbstractChannelHandlerContext prev = ctx.prev;
    AbstractChannelHandlerContext next = ctx.next;
    prev.next = next; // 1
    next.prev = prev; // 2
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

经历的过程要比添加节点要简单，可以用下面一幅图来表示

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905191000831-2042466351.png)

 

最后的结果为

 ![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190905191019693-1683770762.png)

结合这两幅图，可以很清晰地了解权限验证Handler的工作原理，另外，被删除的节点因为没有对象引用到，果过段时间就会被gc自动回收



### 回调用户函数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void callHandlerRemoved0(final AbstractChannelHandlerContext ctx) {
    try {
        ctx.handler().handlerRemoved(ctx);
    } finally {
        ctx.setRemoved();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

到了第三步，pipeline中的节点删除完成，于是便开始回调用户代码 `ctx.handler().handlerRemoved(ctx);`，常见的代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class DemoHandler extends SimpleChannelInboundHandler<...> {
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        // 节点被删除完毕之后回调到此，可做一些资源清理
        // do something
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后，将该节点的状态设置为removed

```
final void setRemoved() {
    handlerState = REMOVE_COMPLETE;
}
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11466291.html#_labelTop)

## 总结

1、在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应。

2、ChannelPipeline是一个维护了一个以 AbstractChannelHandlerContext 为节点的双向链表,其中此链表是 以head（HeadContext）作为头，以tail（TailContext）作为尾的双向链表.

3、pipeline中的每个节点包着具体的处理器`ChannelHandler`，节点根据`ChannelHandler`的类型是`ChannelInboundHandler`还是`ChannelOutboundHandler`来判断该节点属于in还是out或者两者都是

# [Netty源码分析 （五）----- 数据如何在 pipeline 中流动](https://www.cnblogs.com/java-chen-hao/p/11469098.html)

**正文**

在上一篇文章中，我们已经了解了pipeline在netty中所处的角色，像是一条流水线，控制着字节流的读写，本文，我们在这个基础上继续深挖pipeline在事件传播

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11469098.html#_labelTop)

## Unsafe

顾名思义，unsafe是不安全的意思，就是告诉你不要在应用程序里面直接使用Unsafe以及他的衍生类对象。

netty官方的解释如下

> Unsafe operations that should never be called from user-code. These methods are only provided to implement the actual transport, and must be invoked from an I/O thread

Unsafe 在Channel定义，属于Channel的内部类，表明Unsafe和Channel密切相关

下面是unsafe接口的所有方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
interface Unsafe {
   RecvByteBufAllocator.Handle recvBufAllocHandle();
   
   SocketAddress localAddress();
   SocketAddress remoteAddress();

   void register(EventLoop eventLoop, ChannelPromise promise);
   void bind(SocketAddress localAddress, ChannelPromise promise);
   void connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);
   void disconnect(ChannelPromise promise);
   void close(ChannelPromise promise);
   void closeForcibly();
   void beginRead();
   void write(Object msg, ChannelPromise promise);
   void flush();
   
   ChannelPromise voidPromise();
   ChannelOutboundBuffer outboundBuffer();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

按功能可以分为分配内存，Socket四元组信息，注册事件循环，绑定网卡端口，Socket的连接和关闭，Socket的读写，看的出来，这些操作都是和jdk底层相关



### Unsafe 继承结构

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190906094309381-1683823020.png)

 

 

`NioUnsafe` 在 `Unsafe`基础上增加了以下几个接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface NioUnsafe extends Unsafe {
    SelectableChannel ch();
    void finishConnect();
    void read();
    void forceFlush();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从增加的接口以及类名上来看，`NioUnsafe` 增加了可以访问底层jdk的`SelectableChannel`的功能，定义了从`SelectableChannel`读取数据的`read`方法



### Unsafe的分类

从以上继承结构来看，我们可以总结出两种类型的Unsafe分类，一个是与连接的字节数据读写相关的`NioByteUnsafe`，一个是与新连接建立操作相关的`NioMessageUnsafe`

**`NioByteUnsafe`中的读：委托到外部类NioSocketChannel**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected int doReadBytes(ByteBuf byteBuf) throws Exception {
    final RecvByteBufAllocator.Handle allocHandle = unsafe().recvBufAllocHandle();
    allocHandle.attemptedBytesRead(byteBuf.writableBytes());
    return byteBuf.writeBytes(javaChannel(), allocHandle.attemptedBytesRead());
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后一行已经与jdk底层以及netty中的ByteBuf相关，将jdk的 `SelectableChannel`的字节数据读取到netty的`ByteBuf`中

**`NioMessageUnsafe`中的读：委托到外部类NioSocketChannel**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = javaChannel().accept();

    if (ch != null) {
        buf.add(new NioSocketChannel(this, ch));
        return 1;
    }
    return 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`NioMessageUnsafe` 的读操作很简单，就是调用jdk的`accept()`方法，新建立一条连接

**`NioByteUnsafe`中的写：委托到外部类NioSocketChannel**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected int doWriteBytes(ByteBuf buf) throws Exception {
    final int expectedWrittenBytes = buf.readableBytes();
    return buf.readBytes(javaChannel(), expectedWrittenBytes);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后一行已经与jdk底层以及netty中的ByteBuf相关，将netty的`ByteBuf`中的字节数据写到jdk的 `SelectableChannel`中

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11469098.html#_labelTop)

## pipeline中的head

**NioEventLoop**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
     final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
     //新连接的已准备接入或者已存在的连接有数据可读
     if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
         unsafe.read();
     }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**NioByteUnsafe**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void read() {
    final ChannelConfig config = config();
    final ChannelPipeline pipeline = pipeline();
    // 创建ByteBuf分配器
    final ByteBufAllocator allocator = config.getAllocator();
    final RecvByteBufAllocator.Handle allocHandle = recvBufAllocHandle();
    allocHandle.reset(config);

    ByteBuf byteBuf = null;
    do {
        // 分配一个ByteBuf
        byteBuf = allocHandle.allocate(allocator);
        // 将数据读取到分配的ByteBuf中去
        allocHandle.lastBytesRead(doReadBytes(byteBuf));
        if (allocHandle.lastBytesRead() <= 0) {
            byteBuf.release();
            byteBuf = null;
            close = allocHandle.lastBytesRead() < 0;
            break;
        }

        // 触发事件，将会引发pipeline的读事件传播
        pipeline.fireChannelRead(byteBuf);
        byteBuf = null;
    } while (allocHandle.continueReading());
    pipeline.fireChannelReadComplete();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

同样，我抽出了核心代码，细枝末节先剪去，`NioByteUnsafe` 要做的事情可以简单地分为以下几个步骤

1. 拿到Channel的config之后拿到ByteBuf分配器，用分配器来分配一个ByteBuf，ByteBuf是netty里面的字节数据载体，后面读取的数据都读到这个对象里面
2. 将Channel中的数据读取到ByteBuf
3. 数据读完之后，调用 `pipeline.fireChannelRead(byteBuf);` 从head节点开始传播至整个pipeline
4. 最后调用fireChannelReadComplete();

这里，我们的重点其实就是 `pipeline.fireChannelRead(byteBuf);`

**DefaultChannelPipeline**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final AbstractChannelHandlerContext head;
//...
head = new HeadContext(this);

public final ChannelPipeline fireChannelRead(Object msg) {
    AbstractChannelHandlerContext.invokeChannelRead(head, msg);
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结合这幅图

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190906100619465-325910393.png)

 

 

可以看到，数据从head节点开始流入，在进行下一步之前，我们先把head节点的功能过一遍

**HeadContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final class HeadContext extends AbstractChannelHandlerContext
        implements ChannelOutboundHandler, ChannelInboundHandler {

    private final Unsafe unsafe;

    HeadContext(DefaultChannelPipeline pipeline) {
        super(pipeline, null, HEAD_NAME, false, true);
        unsafe = pipeline.channel().unsafe();
        setAddComplete();
    }

    @Override
    public ChannelHandler handler() {
        return this;
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        // NOOP
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        // NOOP
    }

    @Override
    public void bind(
            ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise)
            throws Exception {
        unsafe.bind(localAddress, promise);
    }

    @Override
    public void connect(
            ChannelHandlerContext ctx,
            SocketAddress remoteAddress, SocketAddress localAddress,
            ChannelPromise promise) throws Exception {
        unsafe.connect(remoteAddress, localAddress, promise);
    }

    @Override
    public void disconnect(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception {
        unsafe.disconnect(promise);
    }

    @Override
    public void close(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception {
        unsafe.close(promise);
    }

    @Override
    public void deregister(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception {
        unsafe.deregister(promise);
    }

    @Override
    public void read(ChannelHandlerContext ctx) {
        unsafe.beginRead();
    }

    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
        unsafe.write(msg, promise);
    }

    @Override
    public void flush(ChannelHandlerContext ctx) throws Exception {
        unsafe.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.fireExceptionCaught(cause);
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        invokeHandlerAddedIfNeeded();
        ctx.fireChannelRegistered();
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelUnregistered();

        // Remove all handlers sequentially if channel is closed and unregistered.
        if (!channel.isOpen()) {
            destroy();
        }
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelActive();

        readIfIsAutoRead();
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelInactive();
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ctx.fireChannelRead(msg);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelReadComplete();

        readIfIsAutoRead();
    }

    private void readIfIsAutoRead() {
        if (channel.config().isAutoRead()) {
            channel.read();
        }
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        ctx.fireUserEventTriggered(evt);
    }

    @Override
    public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelWritabilityChanged();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从head节点继承的两个接口看，TA既是一个ChannelHandlerContext，同时又属于inBound和outBound Handler

在传播读写事件的时候，head的功能只是简单地将事件传播下去，如`ctx.fireChannelRead(msg);`

在真正执行读写操作的时候，例如在调用`writeAndFlush()`等方法的时候，最终都会委托到unsafe执行，而当一次数据读完，`channelReadComplete`方法会被调用

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11469098.html#_labelTop)

## pipeline中的inBound事件传播

我们接着上面的 AbstractChannelHandlerContext.**invokeChannelRead(head, msg);** 这个静态方法看，参数传入了 head，我们知道入站数据都是从 head 开始的，以保证后面所有的 handler 都由机会处理数据流。

我们看看这个静态方法内部是怎么样的：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
    final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRead(m);
    } else {
        executor.execute(new Runnable() {
            public void run() {
                next.invokeChannelRead(m);
            }
        });
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

调用这个 Context （也就是 head） 的 invokeChannelRead 方法，并传入数据。我们再看看head中 invokeChannelRead 方法的实现，实际上是在headContext的父类AbstractChannelHandlerContext中：

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeChannelRead(Object msg) {
    if (invokeHandler()) {
        try {
            ((ChannelInboundHandler) handler()).channelRead(this, msg);
        } catch (Throwable t) {
            notifyHandlerException(t);
        }
    } else {
        fireChannelRead(msg);
    }
}

public ChannelHandler handler() {
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面 **`handler()`**`就是``headContext中的handler,也就是headContext自身，也就是调用 head 的 channelRead 方法。那么这个方法是怎么实现的呢？`

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    ctx.fireChannelRead(msg);
}
```

什么都没做，调用 Context 的 fire 系列方法，将请求转发给下一个节点。我们这里是 fireChannelRead 方法，注意，这里方法名字都挺像的。需要细心区分。下面我们看看 Context 的成员方法 fireChannelRead：

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ChannelHandlerContext fireChannelRead(final Object msg) {
    invokeChannelRead(findContextInbound(), msg);
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个是 head 的抽象父类 AbstractChannelHandlerContext 的实现，该方法再次调用了静态 fire 系列方法，但和上次不同的是，不再放入 head 参数了，而是使用 findContextInbound 方法的返回值。从这个方法的名字可以看出，是找到入站类型的 handler。我们看看方法实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext findContextInbound() {
    AbstractChannelHandlerContext ctx = this;
    do {
        ctx = ctx.next;
    } while (!ctx.inbound);
    return ctx;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法很简单，找到当前 Context 的 next 节点（inbound 类型的）并返回。这样就能将请求传递给后面的 inbound handler 了。我们来看看 **invokeChannelRead(findContextInbound(), msg);**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
    final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRead(m);
    } else {
        executor.execute(new Runnable() {
            public void run() {
                next.invokeChannelRead(m);
            }
        });
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面我们找到了next节点（inbound类型的），然后直接调用 **next.invokeChannelRead(m);**如果这个next是我们自定义的handler,此时我们自定义的handler的父类是AbstractChannelHandlerContext，则又回到了**AbstractChannelHandlerContext中实现的**invokeChannelRead，代码如下：

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeChannelRead(Object msg) {
    if (invokeHandler()) {
        try {
            ((ChannelInboundHandler) handler()).channelRead(this, msg);
        } catch (Throwable t) {
            notifyHandlerException(t);
        }
    } else {
        fireChannelRead(msg);
    }
}

public ChannelHandler handler() {
    return this;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

此时的handler()就是我们自定义的handler了，然后调用我们自定义handler中的 **channelRead****(****this****, msg);**

请求进来时，pipeline 会从 head 节点开始输送，通过配合 invoker 接口的 fire 系列方法，实现 Context 链在 pipeline 中的完美传递。最终到达我们自定义的 handler。

**注意：此时如果我们想继续向后传递该怎么办呢？我们前面说过，可以调用 Context 的 fire 系列方法，就像 head 的 channelRead 方法一样，调用 fire 系列方法，直接向后传递就 ok 了。**

**如果所有的handler都调用了fire系列方法，则会传递到最后一个inbound类型的handler，也就是——tail节点，那我们就来看看tail节点**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11469098.html#_labelTop)

## pipeline中的tail

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final class TailContext extends AbstractChannelHandlerContext implements ChannelInboundHandler {

    TailContext(DefaultChannelPipeline pipeline) {
        super(pipeline, null, TAIL_NAME, true, false);
        setAddComplete();
    }

    @Override
    public ChannelHandler handler() {
        return this;
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception { }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        // This may not be a configuration error and so don't log anything.
        // The event may be superfluous for the current pipeline configuration.
        ReferenceCountUtil.release(evt);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        onUnhandledInboundException(cause);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        onUnhandledInboundMessage(msg);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception { }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

正如我们前面所提到的，tail节点的大部分作用即终止事件的传播(方法体为空)

**channelRead**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void onUnhandledInboundMessage(Object msg) {
    try {
        logger.debug(
                "Discarded inbound message {} that reached at the tail of the pipeline. " +
                        "Please check your pipeline configuration.", msg);
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

tail节点在发现字节数据(ByteBuf)或者decoder之后的业务对象在pipeline流转过程中没有被消费，落到tail节点，tail节点就会给你发出一个警告，告诉你，我已经将你未处理的数据给丢掉了

总结一下，tail节点的作用就是结束事件传播，并且对一些重要的事件做一些善意提醒

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11469098.html#_labelTop)

## pipeline中的outBound事件传播

上一节中，我们在阐述tail节点的功能时，忽略了其父类`AbstractChannelHandlerContext`所具有的功能，这一节中，我们以最常见的writeAndFlush操作来看下pipeline中的outBound事件是如何向外传播的

典型的消息推送系统中，会有类似下面的一段代码

```
Channel channel = getChannel(userInfo);
channel.writeAndFlush(pushInfo);
```

这段代码的含义就是根据用户信息拿到对应的Channel，然后给用户推送消息，跟进 `channel.writeAndFlush`

**NioSocketChannel**

```
public ChannelFuture writeAndFlush(Object msg) {
    return pipeline.writeAndFlush(msg);
}
```

从pipeline开始往外传播

```
public final ChannelFuture writeAndFlush(Object msg) {
    return tail.writeAndFlush(msg);
}
```

Channel 中大部分outBound事件都是从tail开始往外传播, `writeAndFlush()`方法是tail继承而来的方法，我们跟进去

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ChannelFuture writeAndFlush(Object msg) {
    return writeAndFlush(msg, newPromise());
}

public ChannelFuture writeAndFlush(Object msg, ChannelPromise promise) {
    write(msg, true, promise);

    return promise;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void write(Object msg, boolean flush, ChannelPromise promise) {
    AbstractChannelHandlerContext next = findContextOutbound();
    final Object m = pipeline.touch(msg, next);
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        if (flush) {
            next.invokeWriteAndFlush(m, promise);
        } else {
            next.invokeWrite(m, promise);
        }
    } else {
        AbstractWriteTask task;
        if (flush) {
            task = WriteAndFlushTask.newInstance(next, m, promise);
        }  else {
            task = WriteTask.newInstance(next, m, promise);
        }
        safeExecute(executor, task, promise, m);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先调用`findContextOutbound()`方法找到下一个`outBound()`节点

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext findContextOutbound() {
    AbstractChannelHandlerContext ctx = this;
    do {
        ctx = ctx.prev;
    } while (!ctx.outbound);
    return ctx;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

找outBound节点的过程和找inBound节点类似，反方向遍历pipeline中的双向链表，直到第一个outBound节点`next`，然后调用`next.invokeWriteAndFlush(m, promise)`

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeWriteAndFlush(Object msg, ChannelPromise promise) {
    if (invokeHandler()) {
        invokeWrite0(msg, promise);
        invokeFlush0();
    } else {
        writeAndFlush(msg, promise);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

调用该节点的ChannelHandler的write方法，flush方法我们暂且忽略，后面会专门讲writeAndFlush的完整流程

**AbstractChannelHandlerContext**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeWrite0(Object msg, ChannelPromise promise) {
    try {
        ((ChannelOutboundHandler) handler()).write(this, msg, promise);
    } catch (Throwable t) {
        notifyOutboundHandlerException(t, promise);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到，数据开始出站，从后向前开始流动，和入站的方向是反的。那么最后会走到哪里呢，当然是走到 head 节点，因为 head 节点就是 outbound 类型的 handler。

**HeadContext**

```
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    unsafe.write(msg, promise);
}
```

调用了 底层的 unsafe 操作数据，这里，加深了我们对head节点的理解，即所有的数据写出都会经过head节点

**当执行完这个 write 方法后，方法开始退栈。逐步退到 unsafe 的 read 方法，回到最初开始的地方，然后继续调用 pipeline.fireChannelReadComplete() 方法**

**![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190906105147411-888783588.png)**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11469098.html#_labelTop)

## 总结

总结一下一个请求在 pipeline 中的流转过程：

1. 调用 pipeline 的 fire 系列方法，这些方法是接口 invoker 设计的，pipeline 实现了 invoker 的所有方法，inbound 事件从 head 开始流入，outbound 事件从 tail 开始流出。
2. pipeline 会将请求交给 Context，然后 Context 通过抽象父类 AbstractChannelHandlerContext 的 invoke 系列方法（静态和非静态的）配合 AbstractChannelHandlerContext 的 fire 系列方法再配合 findContextInbound 和 findContextOutbound 方法完成各个 Context 的数据流转。
3. 当入站过程中，调用 了出站的方法，那么请求就不会向后走了。后面的处理器将不会有任何作用。想继续相会传递就调用 Context 的 fire 系列方法，让 Netty 在内部帮你传递数据到下一个节点。如果你想在整个通道传递，就在 handler 中调用 channel 或者 pipeline 的对应方法，这两个方法会将数据从头到尾或者从尾到头的流转一遍。

# [Netty源码分析 （六）----- 客户端接入accept过程](https://www.cnblogs.com/java-chen-hao/p/11477358.html)

**正文**

通读本文，你会了解到
1.netty如何接受新的请求
2.netty如何给新请求分配reactor线程
3.netty如何给每个新连接增加ChannelHandler

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477358.html#_labelTop)

## netty中的reactor线程

netty中最核心的东西莫过于两种类型的reactor线程，可以看作netty中两种类型的发动机，驱动着netty整个框架的运转

一种类型的reactor线程是boos线程组，专门用来接受新的连接，然后封装成channel对象扔给worker线程组；还有一种类型的reactor线程是worker线程组，专门用来处理连接的读写

不管是boos线程还是worker线程，所做的事情均分为以下三个步骤

1. 轮询注册在selector上的IO事件
2. 处理IO事件
3. 执行异步task

对于boos线程来说，第一步轮询出来的基本都是 accept 事件，表示有新的连接，而worker线程轮询出来的基本都是read/write事件，表示网络的读写事件

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477358.html#_labelTop)

## 新连接的建立

简单来说，新连接的建立可以分为三个步骤
1.检测到有新的连接
2.将新的连接注册到worker线程组
3.注册新连接的读事件



### 检测到有新连接进入

我们已经知道，当服务端绑启动之后，服务端的channel已经注册到boos reactor线程中，reactor不断检测有新的事件，直到检测出有accept事件发生

**NioEventLoop.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final NioUnsafe unsafe = ch.unsafe();
    //检查该SelectionKey是否有效，如果无效，则关闭channel
    if (!k.isValid()) {
        // close the channel if the key is not valid anymore
        unsafe.close(unsafe.voidPromise());
        return;
    }

    try {
        int readyOps = k.readyOps();
        // Also check for readOps of 0 to workaround possible JDK bug which may otherwise lead
        // to a spin loop
        // 如果准备好READ或ACCEPT则触发unsafe.read() ,检查是否为0，如上面的源码英文注释所说：解决JDK可能会产生死循环的一个bug。
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
            if (!ch.isOpen()) {//如果已经关闭，则直接返回即可，不需要再处理该channel的其他事件
                // Connection already closed - no need to handle write.
                return;
            }
        }
        // 如果准备好了WRITE则将缓冲区中的数据发送出去，如果缓冲区中数据都发送完成，则清除之前关注的OP_WRITE标记
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
            ch.unsafe().forceFlush();
        }
        // 如果是OP_CONNECT，则需要移除OP_CONNECT否则Selector.select(timeout)将立即返回不会有任何阻塞，这样可能会出现cpu 100%
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
            // See https://github.com/netty/netty/issues/924
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);

            unsafe.finishConnect();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法主要是对SelectionKey k进行了检查，有如下几种不同的情况

1）OP_ACCEPT，接受客户端连接

2）OP_READ, 可读事件, 即 Channel 中收到了新数据可供上层读取。

3）OP_WRITE, 可写事件, 即上层可以向 Channel 写入数据。

4）OP_CONNECT, 连接建立事件, 即 TCP 连接已经建立, Channel 处于 active 状态。

本篇博文主要来看下当boss线程 selector检测到OP_ACCEPT事件时，内部干了些什么。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
    unsafe.read();
    if (!ch.isOpen()) {//如果已经关闭，则直接返回即可，不需要再处理该channel的其他事件
        // Connection already closed - no need to handle write.
        return;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

boos reactor线程已经轮询到 `SelectionKey.OP_ACCEPT` 事件，说明有新的连接进入，此时将调用channel的 `unsafe`来进行实际的操作,**此时的channel为** **NioServerSocketChannel，则unsafe为\**NioServerSocketChannel的属性NioMessageUnsafe\****

那么，我们进入到它的`read`方法，进入新连接处理的第二步



### 注册到reactor线程

**NioMessageUnsafe.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final List<Object> readBuf = new ArrayList<Object>();

public void read() {
    assert eventLoop().inEventLoop();
    final ChannelPipeline pipeline = pipeline();
    final RecvByteBufAllocator.Handle allocHandle = unsafe().recvBufAllocHandle();
    do {
        int localRead = doReadMessages(readBuf);
        if (localRead == 0) {
            break;
        }
        if (localRead < 0) {
            closed = true;
            break;
        }
    } while (allocHandle.continueReading());
    int size = readBuf.size();
    for (int i = 0; i < size; i ++) {
        pipeline.fireChannelRead(readBuf.get(i));
    }
    readBuf.clear();
    pipeline.fireChannelReadComplete();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

调用 `doReadMessages` 方法不断地读取消息，用 `readBuf` 作为容器，这里，其实可以猜到读取的是一个个连接，然后调用 `pipeline.fireChannelRead()`，将每条新连接经过一层服务端channel的洗礼，之后清理容器，触发 `pipeline.fireChannelReadComplete()`

```
下面我们具体看下这两个方法
```

1.doReadMessages(List)
2.pipeline.fireChannelRead(NioSocketChannel)



### doReadMessages()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = javaChannel().accept();

    try {
        if (ch != null) {
            buf.add(new NioSocketChannel(this, ch));
            return 1;
        }
    } catch (Throwable t) {
        logger.warn("Failed to create a new channel from an accepted socket.", t);

        try {
            ch.close();
        } catch (Throwable t2) {
            logger.warn("Failed to close a socket.", t2);
        }
    }

    return 0;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们终于窥探到netty调用jdk底层nio的边界 `javaChannel().accept();`，由于netty中reactor线程第一步就扫描到有accept事件发生，因此，这里的`accept`方法是立即返回的，返回jdk底层nio创建的一条channel

ServerSocketChannel有阻塞和非阻塞两种模式：

a、阻塞模式：ServerSocketChannel.accept() 方法监听新进来的连接，当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。阻塞模式下, accept()方法会一直阻塞到有新连接到达。

b、非阻塞模式：，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.

在NioServerSocketChannel的构造函数分析中，我们知道，**其通过ch.configureBlocking(false);语句设置当前的ServerSocketChannel为非阻塞的**。

netty将jdk的 `SocketChannel` 封装成自定义的 `NioSocketChannel`，加入到list里面，这样外层就可以遍历该list，做后续处理

从上一篇文章中，我们已经知道服务端的创建过程中会创建netty中一系列的核心组件，包括pipeline,unsafe等等，那么，接受一条新连接的时候是否也会创建这一系列的组件呢？

带着这个疑问，我们跟进去

**NioSocketChannel.java**

```
public NioSocketChannel(Channel parent, SocketChannel socket) {
    super(parent, socket);
    config = new NioSocketChannelConfig(this, socket.socket());
}
```

我们重点分析 `super(parent, socket)，NioSocketChannel的父类为 AbstractNioByteChannel`

**AbstractNioByteChannel.java**

```
protected AbstractNioByteChannel(Channel parent, SelectableChannel ch) {
    super(parent, ch, SelectionKey.OP_READ);
}
```

这里，我们看到jdk nio里面熟悉的影子—— `SelectionKey.OP_READ`，一般在原生的jdk nio编程中，也会注册这样一个事件，表示对channel的读感兴趣

我们继续往上，追踪到`AbstractNioByteChannel`的父类 `AbstractNioChannel`, 这里，我相信读了上一篇文章你对于这部分代码肯定是有印象的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    this.readInterestOp = readInterestOp;
    try {
        ch.configureBlocking(false);
    } catch (IOException e) {
        try {
            ch.close();
        } catch (IOException e2) {
            if (logger.isWarnEnabled()) {
                logger.warn(
                        "Failed to close a partially initialized socket.", e2);
            }
        }
        throw new ChannelException("Failed to enter non-blocking mode.", e);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在创建服务端channel的时候，最终也会进入到这个方法，`super(parent)`, 便是在`AbstractChannel`中创建一系列和该channel绑定的组件，如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected AbstractChannel(Channel parent) {
    this.parent = parent;
    id = newId();
    unsafe = newUnsafe();
    pipeline = newChannelPipeline();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

而这里的 `readInterestOp` 表示该channel关心的事件是 `SelectionKey.OP_READ`，后续会将该事件注册到selector，之后设置该通道为非阻塞模式，在channel中创建 **unsafe 和一条** **pipeline** 



### pipeline.fireChannelRead(NioSocketChannel)

对于 **`pipeline`**`我们前面已经了解过，在netty的各种类型的channel中，都会包含一个pipeline，字面意思是管道，我们可以理解为一条流水线工艺，流水线工艺有起点，有结束，中间还有各种各样的流水线关卡，一件物品，在流水线起点开始处理，经过各个流水线关卡的加工，最终到流水线结束`

对应到netty里面，流水线的开始就是`HeadContxt`，流水线的结束就是`TailConext`，`HeadContxt`中调用`Unsafe`做具体的操作，`TailConext`中用于向用户抛出pipeline中未处理异常以及对未处理消息的警告

通过前面的文章中，我们已经知道在服务端的channel初始化时，在pipeline中，已经自动添加了一个pipeline处理器 **`ServerBootstrapAcceptor`**, 并已经将用户代码中设置的一系列的参数传入了构造函数，接下来，我们就来看下`ServerBootstrapAcceptor`

**ServerBootstrapAcceptor.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static class ServerBootstrapAcceptor extends ChannelInboundHandlerAdapter {
    private final EventLoopGroup childGroup;
    private final ChannelHandler childHandler;
    private final Entry<ChannelOption<?>, Object>[] childOptions;
    private final Entry<AttributeKey<?>, Object>[] childAttrs;

    ServerBootstrapAcceptor(
            EventLoopGroup childGroup, ChannelHandler childHandler,
            Entry<ChannelOption<?>, Object>[] childOptions, Entry<AttributeKey<?>, Object>[] childAttrs) {
        this.childGroup = childGroup;
        this.childHandler = childHandler;
        this.childOptions = childOptions;
        this.childAttrs = childAttrs;
    }

    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        final Channel child = (Channel) msg;

        child.pipeline().addLast(childHandler);

        for (Entry<ChannelOption<?>, Object> e: childOptions) {
            try {
                if (!child.config().setOption((ChannelOption<Object>) e.getKey(), e.getValue())) {
                    logger.warn("Unknown channel option: " + e);
                }
            } catch (Throwable t) {
                logger.warn("Failed to set a channel option: " + child, t);
            }
        }

        for (Entry<AttributeKey<?>, Object> e: childAttrs) {
            child.attr((AttributeKey<Object>) e.getKey()).set(e.getValue());
        }

        try {
            childGroup.register(child).addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    if (!future.isSuccess()) {
                        forceClose(child, future.cause());
                    }
                }
            });
        } catch (Throwable t) {
            forceClose(child, t);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

前面的 `pipeline.fireChannelRead(NioSocketChannel);` 最终通过head->unsafe->ServerBootstrapAcceptor的调用链，调用到这里的 `ServerBootstrapAcceptor` 的`channelRead`方法，而 `channelRead` 一上来就把这里的msg强制转换为 `Channel`

然后，拿到该channel，也就是我们之前new出来的 `NioSocketChannel中`对应的pipeline，将用户代码中的 `childHandler`，添加到pipeline，这里的 `childHandler` 在用户代码中的体现为

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 .childHandler(new ChannelInitializer<SocketChannel>() {
     @Override
     public void initChannel(SocketChannel ch) throws Exception {
         ChannelPipeline p = ch.pipeline();
         p.addLast(new EchoServerHandler());
     }
 });
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

其实对应的是 `ChannelInitializer`，到了这里，`NioSocketChannel`中pipeline对应的处理器为 head->ChannelInitializer->tail，牢记，后面会再次提到！

接着，设置 `NioSocketChannel` 对应的 attr和option，然后进入到 `childGroup.register(child)`，这里的childGroup就是我们在启动代码中new出来的`NioEventLoopGroup`

我们进入到`NioEventLoopGroup`的`register`方法，代理到其父类`MultithreadEventLoopGroup`

**MultithreadEventLoopGroup.java**

```
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
```

这里又扯出来一个 next()方法，我们跟进去

**MultithreadEventLoopGroup.java**

```
@Override
public EventLoop next() {
    return (EventLoop) super.next();
}
```

回到其父类

**MultithreadEventExecutorGroup.java**

```
@Override
public EventExecutor next() {
    return chooser.next();
}
```

这里的chooser对应的类为 `EventExecutorChooser`，字面意思为事件执行器选择器，放到我们这里的上下文中的作用就是从worker reactor线程组中选择一个reactor线程

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface EventExecutorChooserFactory {

    /**
     * Returns a new {@link EventExecutorChooser}.
     */
    EventExecutorChooser newChooser(EventExecutor[] executors);

    /**
     * Chooses the next {@link EventExecutor} to use.
     */
    @UnstableApi
    interface EventExecutorChooser {

        /**
         * Returns the new {@link EventExecutor} to use.
         */
        EventExecutor next();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

chooser的实现有两种

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final class DefaultEventExecutorChooserFactory implements EventExecutorChooserFactory {

    public static final DefaultEventExecutorChooserFactory INSTANCE = new DefaultEventExecutorChooserFactory();

    private DefaultEventExecutorChooserFactory() { }

    @SuppressWarnings("unchecked")
    @Override
    public EventExecutorChooser newChooser(EventExecutor[] executors) {
        if (isPowerOfTwo(executors.length)) {
            return new PowerOfTowEventExecutorChooser(executors);
        } else {
            return new GenericEventExecutorChooser(executors);
        }
    }

    private static boolean isPowerOfTwo(int val) {
        return (val & -val) == val;
    }

    private static final class PowerOfTowEventExecutorChooser implements EventExecutorChooser {
        private final AtomicInteger idx = new AtomicInteger();
        private final EventExecutor[] executors;

        PowerOfTowEventExecutorChooser(EventExecutor[] executors) {
            this.executors = executors;
        }

        @Override
        public EventExecutor next() {
            return executors[idx.getAndIncrement() & executors.length - 1];
        }
    }

    private static final class GenericEventExecutorChooser implements EventExecutorChooser {
        private final AtomicInteger idx = new AtomicInteger();
        private final EventExecutor[] executors;

        GenericEventExecutorChooser(EventExecutor[] executors) {
            this.executors = executors;
        }

        @Override
        public EventExecutor next() {
            return executors[Math.abs(idx.getAndIncrement() % executors.length)];
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

默认情况下，chooser通过 `DefaultEventExecutorChooserFactory`被创建，在创建reactor线程选择器的时候，会判断reactor线程的个数，如果是2的幂，就创建`PowerOfTowEventExecutorChooser`，否则，创建`GenericEventExecutorChooser`

两种类型的选择器在选择reactor线程的时候，都是通过Round-Robin的方式选择reactor线程，唯一不同的是，`PowerOfTowEventExecutorChooser`是通过与运算，而`GenericEventExecutorChooser`是通过取余运算，与运算的效率要高于求余运算

选择完一个reactor线程，即 `NioEventLoop` 之后，我们回到注册的地方

```
public ChannelFuture register(Channel channel) {
    return next().register(channel);
}
```

**SingleThreadEventLoop.java**

```
@Override
public ChannelFuture register(Channel channel) {
    return register(new DefaultChannelPromise(channel, this));
}
```

其实，这里已经和服务端启动的过程一样了，可以参考我前面的文章

**AbstractNioChannel.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void register0(ChannelPromise promise) {
    boolean firstRegistration = neverRegistered;
    doRegister();
    neverRegistered = false;
    registered = true;

    pipeline.invokeHandlerAddedIfNeeded();

    safeSetSuccess(promise);
    pipeline.fireChannelRegistered();
    if (isActive()) {
        if (firstRegistration) {
            pipeline.fireChannelActive();
        } else if (config().isAutoRead()) {
            beginRead();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

和服务端启动过程一样，先是调用 `doRegister();`做真正的注册过程，如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
            selectionKey = javaChannel().register(eventLoop().selector, 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
                eventLoop().selectNow();
                selected = true;
            } else {
                throw e;
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

将该条channel绑定到一个`selector`上去，一个selector被一个reactor线程使用，后续该channel的事件轮询，以及事件处理，异步task执行都是由此reactor线程来负责

绑定完reactor线程之后，调用 `pipeline.invokeHandlerAddedIfNeeded()`

前面我们说到，到目前为止`NioSocketChannel` 的pipeline中有三个处理器，head->ChannelInitializer->tail，最终会调用到 `ChannelInitializer` 的 `handlerAdded` 方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
    if (ctx.channel().isRegistered()) {
        initChannel(ctx);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`handlerAdded`方法调用 `initChannel` 方法之后，调用`remove(ctx);`将自身删除，如下

**AbstractNioChannel.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
    if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) { 
        try {
            initChannel((C) ctx.channel());
        } catch (Throwable cause) {
            exceptionCaught(ctx, cause);
        } finally {
            remove(ctx);
        }
        return true;
    }
    return false;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

而这里的 `initChannel` 方法又是神马玩意？让我们回到用户方法，比如下面这段用户代码

**用户代码**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
 .option(ChannelOption.SO_BACKLOG, 100)
 .handler(new LoggingHandler(LogLevel.INFO))
 .childHandler(new ChannelInitializer<SocketChannel>() {
     @Override
     public void initChannel(SocketChannel ch) throws Exception {
         ChannelPipeline p = ch.pipeline();
         p.addLast(new LoggingHandler(LogLevel.INFO));
         p.addLast(new EchoServerHandler());
     }
 });
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

原来最终跑到我们自己的代码里去了啊！完了之后，`NioSocketChannel`绑定的pipeline的处理器就包括 head->LoggingHandler->EchoServerHandler->tail



### 注册读事件

接下来，我们还剩下这些代码没有分析完

**AbstractNioChannel.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void register0(ChannelPromise promise) {
    // ..
    pipeline.fireChannelRegistered();
    if (isActive()) {
        if (firstRegistration) {
            pipeline.fireChannelActive();
        } else if (config().isAutoRead()) {
            beginRead();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`pipeline.fireChannelRegistered();`，其实没有干啥有意义的事情，最终无非是再调用一下业务pipeline中每个处理器的 `ChannelHandlerAdded`方法处理下回调

`isActive()`在连接已经建立的情况下返回true，所以进入方法块，进入到 `pipeline.fireChannelActive();`在这里我详细步骤先省略，直接进入到关键环节

**AbstractNioChannel.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected void doBeginRead() throws Exception {
    // Channel.read() or ChannelHandlerContext.read() was called
    final SelectionKey selectionKey = this.selectionKey;
    if (!selectionKey.isValid()) {
        return;
    }

    readPending = true;

    final int interestOps = selectionKey.interestOps();
    if ((interestOps & readInterestOp) == 0) {
        selectionKey.interestOps(interestOps | readInterestOp);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里其实就是将 `SelectionKey.OP_READ`事件注册到selector中去，表示这条通道已经可以开始处理read事件了

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477358.html#_labelTop)

## 总结

至此，netty中关于新连接的处理已经向你展示完了，我们做下总结

1.boos reactor线程轮询到有新的连接进入
2.通过封装jdk底层的channel创建 `NioSocketChannel`以及一系列的netty核心组件
3.将该条连接通过chooser，选择一条worker reactor线程绑定上去
4.注册读事件，开始新连接的读写

# [Netty源码分析 （七）----- read过程 源码分析](https://www.cnblogs.com/java-chen-hao/p/11477384.html)

**正文**

在上一篇文章中，我们分析了processSelectedKey这个方法中的accept过程，本文将分析一下work线程中的read过程。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final NioUnsafe unsafe = ch.unsafe();
    //检查该SelectionKey是否有效，如果无效，则关闭channel
    if (!k.isValid()) {
        // close the channel if the key is not valid anymore
        unsafe.close(unsafe.voidPromise());
        return;
    }

    try {
        int readyOps = k.readyOps();
        // Also check for readOps of 0 to workaround possible JDK bug which may otherwise lead
        // to a spin loop
        // 如果准备好READ或ACCEPT则触发unsafe.read() ,检查是否为0，如上面的源码英文注释所说：解决JDK可能会产生死循环的一个bug。
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
            if (!ch.isOpen()) {//如果已经关闭，则直接返回即可，不需要再处理该channel的其他事件
                // Connection already closed - no need to handle write.
                return;
            }
        }
        // 如果准备好了WRITE则将缓冲区中的数据发送出去，如果缓冲区中数据都发送完成，则清除之前关注的OP_WRITE标记
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
            ch.unsafe().forceFlush();
        }
        // 如果是OP_CONNECT，则需要移除OP_CONNECT否则Selector.select(timeout)将立即返回不会有任何阻塞，这样可能会出现cpu 100%
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
            // See https://github.com/netty/netty/issues/924
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);

            unsafe.finishConnect();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法主要是对SelectionKey k进行了检查，有如下几种不同的情况

1）OP_ACCEPT，接受客户端连接

2）OP_READ, 可读事件, 即 Channel 中收到了新数据可供上层读取。

3）OP_WRITE, 可写事件, 即上层可以向 Channel 写入数据。

4）OP_CONNECT, 连接建立事件, 即 TCP 连接已经建立, Channel 处于 active 状态。

本篇博文主要来看下当work 线程 selector检测到OP_READ事件时，内部干了些什么。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
    unsafe.read();
    if (!ch.isOpen()) {//如果已经关闭，则直接返回即可，不需要再处理该channel的其他事件
        // Connection already closed - no need to handle write.
        return;
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从代码中可以看到，当selectionKey发生的事件是SelectionKey.OP_READ，执行unsafe的read方法。注意这里的unsafe是NioByteUnsafe的实例

为什么说这里的unsafe是NioByteUnsafe的实例呢？在上篇博文Netty源码分析：accept中我们知道Boss NioEventLoopGroup中的NioEventLoop只负责accpt客户端连接，然后将该客户端注册到Work NioEventLoopGroup中的NioEventLoop中，即最终是由work线程对应的selector来进行read等时间的监听，即work线程中的channel为SocketChannel，SocketChannel的unsafe就是NioByteUnsafe的实例

下面来看下NioByteUnsafe中的read方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
    public void read() {
        final ChannelConfig config = config();
        if (!config.isAutoRead() && !isReadPending()) {
            // ChannelConfig.setAutoRead(false) was called in the meantime
            removeReadOp();
            return;
        }

        final ChannelPipeline pipeline = pipeline();
        final ByteBufAllocator allocator = config.getAllocator();
        final int maxMessagesPerRead = config.getMaxMessagesPerRead();
        RecvByteBufAllocator.Handle allocHandle = this.allocHandle;
        if (allocHandle == null) {
            this.allocHandle = allocHandle = config.getRecvByteBufAllocator().newHandle();
        }

        ByteBuf byteBuf = null;
        int messages = 0;
        boolean close = false;
        try {
            int totalReadAmount = 0;
            boolean readPendingReset = false;
            do {
                //1、分配缓存
                byteBuf = allocHandle.allocate(allocator);
                int writable = byteBuf.writableBytes();//可写的字节容量
                //2、将socketChannel数据写入缓存
                int localReadAmount = doReadBytes(byteBuf);
                if (localReadAmount <= 0) {
                    // not was read release the buffer
                    byteBuf.release();
                    close = localReadAmount < 0;
                    break;
                }
                if (!readPendingReset) {
                    readPendingReset = true;
                    setReadPending(false);
                }
                //3、触发pipeline的ChannelRead事件来对byteBuf进行后续处理
                pipeline.fireChannelRead(byteBuf);
                byteBuf = null;

                if (totalReadAmount >= Integer.MAX_VALUE - localReadAmount) {
                    // Avoid overflow.
                    totalReadAmount = Integer.MAX_VALUE;
                    break;
                }

                totalReadAmount += localReadAmount;

                // stop reading
                if (!config.isAutoRead()) {
                    break;
                }

                if (localReadAmount < writable) {
                    // Read less than what the buffer can hold,
                    // which might mean we drained the recv buffer completely.
                    break;
                }
            } while (++ messages < maxMessagesPerRead);

            pipeline.fireChannelReadComplete();
            allocHandle.record(totalReadAmount);

            if (close) {
                closeOnRead(pipeline);
                close = false;
            }
        } catch (Throwable t) {
            handleReadException(pipeline, byteBuf, t, close);
        } finally {
            if (!config.isAutoRead() && !isReadPending()) {
                removeReadOp();
            }
        }
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面一一介绍比较重要的代码

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477384.html#_labelTop)

## allocHandler的实例化过程

allocHandle负责自适应调整当前缓存分配的大小，以防止缓存分配过多或过少，先看allocHandler的实例化过程

```
RecvByteBufAllocator.Handle allocHandle = this.allocHandle;
if (allocHandle == null) {
    this.allocHandle = allocHandle = config.getRecvByteBufAllocator().newHandle();
}
```

其中， `config.getRecvByteBufAllocator()`得到的是一个 AdaptiveRecvByteBufAllocator实例DEFAULT。

```
public static final AdaptiveRecvByteBufAllocator DEFAULT = new AdaptiveRecvByteBufAllocator();
```

而AdaptiveRecvByteBufAllocator中的newHandler()方法的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public Handle newHandle() {
    return new HandleImpl(minIndex, maxIndex, initial);
}

HandleImpl(int minIndex, int maxIndex, int initial) {
    this.minIndex = minIndex;
    this.maxIndex = maxIndex;

    index = getSizeTableIndex(initial);
    nextReceiveBufferSize = SIZE_TABLE[index];
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

其中，上面方法中所用到参数：minIndex maxIndex initial是什么意思呢？含义如下：minIndex是最小缓存在`SIZE_TABLE`中对应的下标。maxIndex是最大缓存在`SIZE_TABLE`中对应的下标，initial为初始化缓存大小。

AdaptiveRecvByteBufAllocator的相关常量字段

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class AdaptiveRecvByteBufAllocator implements RecvByteBufAllocator {

        static final int DEFAULT_MINIMUM = 64;
        static final int DEFAULT_INITIAL = 1024;
        static final int DEFAULT_MAXIMUM = 65536;

        private static final int INDEX_INCREMENT = 4;
        private static final int INDEX_DECREMENT = 1;

        private static final int[] SIZE_TABLE; 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面这些字段的具体含义说明如下：

1）、`SIZE_TABLE`：按照从小到大的顺序预先存储可以分配的缓存大小。 
从16开始，每次累加16，直到496，接着从512开始，每次增大一倍，直到溢出。SIZE_TABLE初始化过程如下。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static {
    List<Integer> sizeTable = new ArrayList<Integer>();
    for (int i = 16; i < 512; i += 16) {
        sizeTable.add(i);
    }

    for (int i = 512; i > 0; i <<= 1) {
        sizeTable.add(i);
    }

    SIZE_TABLE = new int[sizeTable.size()];
    for (int i = 0; i < SIZE_TABLE.length; i ++) {
        SIZE_TABLE[i] = sizeTable.get(i);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2）、DEFAULT_MINIMUM：最小缓存（64），在SIZE_TABLE中对应的下标为3。

3）、DEFAULT_MAXIMUM ：最大缓存（65536），在SIZE_TABLE中对应的下标为38。

4）、DEFAULT_INITIAL ：初始化缓存大小，第一次分配缓存时，由于没有上一次实际收到的字节数做参考，需要给一个默认初始值。

5）、INDEX_INCREMENT：上次预估缓存偏小，下次index的递增值。

6）、INDEX_DECREMENT ：上次预估缓存偏大，下次index的递减值。

构造函数：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AdaptiveRecvByteBufAllocator() {
    this(DEFAULT_MINIMUM, DEFAULT_INITIAL, DEFAULT_MAXIMUM);
}

public AdaptiveRecvByteBufAllocator(int minimum, int initial, int maximum) {
    if (minimum <= 0) {
        throw new IllegalArgumentException("minimum: " + minimum);
    }
    if (initial < minimum) {
        throw new IllegalArgumentException("initial: " + initial);
    }
    if (maximum < initial) {
        throw new IllegalArgumentException("maximum: " + maximum);
    }

    int minIndex = getSizeTableIndex(minimum);
    if (SIZE_TABLE[minIndex] < minimum) {
        this.minIndex = minIndex + 1;
    } else {
        this.minIndex = minIndex;
    }

    int maxIndex = getSizeTableIndex(maximum);
    if (SIZE_TABLE[maxIndex] > maximum) {
        this.maxIndex = maxIndex - 1;
    } else {
        this.maxIndex = maxIndex;
    }

    this.initial = initial;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该构造函数对参数进行了有效性检查，然后初始化了如下3个字段，这3个字段就是上面用于产生allocHandle对象所要用到的参数。

```
private final int minIndex;
private final int maxIndex;
private final int initial;
```

其中，getSizeTableIndex函数的代码如下，该函数的功能为：找到SIZE_TABLE中的元素刚好大于或等于size的位置。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static int getSizeTableIndex(final int size) {
    for (int low = 0, high = SIZE_TABLE.length - 1;;) {
        if (high < low) {
            return low;
        }
        if (high == low) {
            return high;
        }

        int mid = low + high >>> 1;
        int a = SIZE_TABLE[mid];
        int b = SIZE_TABLE[mid + 1];
        if (size > b) {
            low = mid + 1;
        } else if (size < a) {
            high = mid - 1;
        } else if (size == a) {
            return mid;
        } else { //这里的情况就是 a < size <= b 的情况
            return mid + 1;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477384.html#_labelTop)

## byteBuf = allocHandle.allocate(allocator);

申请一块指定大小的内存

**AdaptiveRecvByteBufAllocator#HandlerImpl**

```
@Override
public ByteBuf allocate(ByteBufAllocator alloc) {
    return alloc.ioBuffer(nextReceiveBufferSize);
}
```

直接调用了ioBuffer方法，继续看

**AbstractByteBufAllocator.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ByteBuf ioBuffer(int initialCapacity) {
    if (PlatformDependent.hasUnsafe()) {
        return directBuffer(initialCapacity);
    }
    return heapBuffer(initialCapacity);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ioBuffer函数中主要逻辑为：看平台是否支持unsafe，选择使用直接物理内存还是堆上内存。先看 heapBuffer

**AbstractByteBufAllocator.java** 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ByteBuf heapBuffer(int initialCapacity) {
    return heapBuffer(initialCapacity, Integer.MAX_VALUE);
}

@Override
public ByteBuf heapBuffer(int initialCapacity, int maxCapacity) {
    if (initialCapacity == 0 && maxCapacity == 0) {
        return emptyBuf;
    }
    validate(initialCapacity, maxCapacity);
    return newHeapBuffer(initialCapacity, maxCapacity);
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里的newHeapBuffer有两种实现：至于具体用哪一种，取决于我们对系统属性io.netty.allocator.type的设置，如果设置为： “pooled”，则缓存分配器就为：PooledByteBufAllocator，进而利用对象池技术进行内存分配。如果不设置或者设置为其他，则缓存分配器为：UnPooledByteBufAllocator，则直接返回一个UnpooledHeapByteBuf对象。

**UnpooledByteBufAllocator.java**

```
@Override
protected ByteBuf newHeapBuffer(int initialCapacity, int maxCapacity) {
    return new UnpooledHeapByteBuf(this, initialCapacity, maxCapacity);
}
```

**PooledByteBufAllocator.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected ByteBuf newHeapBuffer(int initialCapacity, int maxCapacity) {
    PoolThreadCache cache = threadCache.get();
    PoolArena<byte[]> heapArena = cache.heapArena;

    ByteBuf buf;
    if (heapArena != null) {
        buf = heapArena.allocate(cache, initialCapacity, maxCapacity);
    } else {
        buf = new UnpooledHeapByteBuf(this, initialCapacity, maxCapacity);
    }

    return toLeakAwareBuffer(buf);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

再看directBuffer

**AbstractByteBufAllocator.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public ByteBuf directBuffer(int initialCapacity) {
    return directBuffer(initialCapacity, Integer.MAX_VALUE);
}  

@Override
public ByteBuf directBuffer(int initialCapacity, int maxCapacity) {
    if (initialCapacity == 0 && maxCapacity == 0) {
        return emptyBuf;
    }
    validate(initialCapacity, maxCapacity);//参数的有效性检查
    return newDirectBuffer(initialCapacity, maxCapacity);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

与newHeapBuffer一样，这里的newDirectBuffer方法也有两种实现：至于具体用哪一种，取决于我们对系统属性io.netty.allocator.type的设置，如果设置为： “pooled”，则缓存分配器就为：PooledByteBufAllocator，进而利用对象池技术进行内存分配。如果不设置或者设置为其他，则缓存分配器为：UnPooledByteBufAllocator。这里主要看下UnpooledByteBufAllocator. newDirectBuffer的内部实现

**UnpooledByteBufAllocator.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected ByteBuf newDirectBuffer(int initialCapacity, int maxCapacity) {
    ByteBuf buf;
    if (PlatformDependent.hasUnsafe()) {
        buf = new UnpooledUnsafeDirectByteBuf(this, initialCapacity, maxCapacity);
    } else {
        buf = new UnpooledDirectByteBuf(this, initialCapacity, maxCapacity);
    }

    return toLeakAwareBuffer(buf);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

UnpooledUnsafeDirectByteBuf是如何实现缓存管理的？对Nio的ByteBuffer进行了封装，通过ByteBuffer的allocateDirect方法实现缓存的申请。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected UnpooledUnsafeDirectByteBuf(ByteBufAllocator alloc, int initialCapacity, int maxCapacity) {
    super(maxCapacity);
    //省略了部分参数检查的代码
    this.alloc = alloc;
    setByteBuffer(allocateDirect(initialCapacity));
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected ByteBuffer allocateDirect(int initialCapacity) {
    return ByteBuffer.allocateDirect(initialCapacity);
}

private void setByteBuffer(ByteBuffer buffer) {
    ByteBuffer oldBuffer = this.buffer;
    if (oldBuffer != null) {
        if (doNotFree) {
            doNotFree = false;
        } else {
            freeDirect(oldBuffer);
        }
    }

    this.buffer = buffer;
    memoryAddress = PlatformDependent.directBufferAddress(buffer);
    tmpNioBuf = null;
    capacity = buffer.remaining();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面代码的主要逻辑为：

1、先利用ByteBuffer的allocateDirect方法分配了大小为initialCapacity的缓存

2、然后判断将旧缓存给free掉

3、最后将新缓存赋给字段buffer上

其中：memoryAddress = PlatformDependent.directBufferAddress(buffer) 获取buffer的address字段值，指向缓存地址。
capacity = buffer.remaining() 获取缓存容量。

接下来看toLeakAwareBuffer(buf)方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected static ByteBuf toLeakAwareBuffer(ByteBuf buf) {
    ResourceLeak leak;
    switch (ResourceLeakDetector.getLevel()) {
        case SIMPLE:
            leak = AbstractByteBuf.leakDetector.open(buf);
            if (leak != null) {
                buf = new SimpleLeakAwareByteBuf(buf, leak);
            }
            break;
        case ADVANCED:
        case PARANOID:
            leak = AbstractByteBuf.leakDetector.open(buf);
            if (leak != null) {
                buf = new AdvancedLeakAwareByteBuf(buf, leak);
            }
            break;
    }
    return buf;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

方法toLeakAwareBuffer(buf)对申请的buf又进行了一次包装。

上面一长串的分析，得到了缓存后，回到AbstractNioByteChannel.read方法，继续看。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477384.html#_labelTop)

## doReadBytes方法

下面看下doReadBytes方法：将socketChannel数据写入缓存。

**NioSocketChannel.java**

```
@Override
protected int doReadBytes(ByteBuf byteBuf) throws Exception {
    return byteBuf.writeBytes(javaChannel(), byteBuf.writableBytes());
}
```

将Channel中的数据读入缓存byteBuf中。继续看

**WrappedByteBuf.java**

```
@Override
public int writeBytes(ScatteringByteChannel in, int length) throws IOException {
    return buf.writeBytes(in, length);
} 
```

**AbstractByteBuf.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public int writeBytes(ScatteringByteChannel in, int length) throws IOException {
    ensureAccessible();
    ensureWritable(length);
    int writtenBytes = setBytes(writerIndex, in, length);
    if (writtenBytes > 0) {
        writerIndex += writtenBytes;
    }
    return writtenBytes;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里的setBytes方法有不同的实现，这里看下UnpooledUnsafeDirectByteBuf的setBytes的实现。

**UnpooledUnsafeDirectByteBuf.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public int setBytes(int index, ScatteringByteChannel in, int length) throws IOException {
    ensureAccessible();
    ByteBuffer tmpBuf = internalNioBuffer();
    tmpBuf.clear().position(index).limit(index + length);
    try {
        return in.read(tmpBuf);
    } catch (ClosedChannelException ignored) {
        return -1;//当Channel 已经关闭，则返回-1.    
    }
} 

private ByteBuffer internalNioBuffer() {
    ByteBuffer tmpNioBuf = this.tmpNioBuf;
    if (tmpNioBuf == null) {
        this.tmpNioBuf = tmpNioBuf = buffer.duplicate();
    }
    return tmpNioBuf;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最终底层采用ByteBuffer实现read操作，无论是PooledByteBuf、还是UnpooledXXXBuf，里面都将底层数据结构BufBuffer/array转换为ByteBuffer 来实现read操作。即无论是UnPooledXXXBuf还是PooledXXXBuf里面都有一个ByteBuffer tmpNioBuf，这个tmpNioBuf才是真正用来存储从管道Channel中读取出的内容的。**到这里就完成了将channel的数据读入到了缓存Buf中。**

我们具体来看看 in.read(tmpBuf); FileChannel和SocketChannel的read最后都是依赖的IOUtil来实现，代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public int read(ByteBuffer dst) throws IOException {
    ensureOpen();
    if (!readable)
        throw new NonReadableChannelException();
    synchronized (positionLock) {
        int n = 0;
        int ti = -1;
        try {
            begin();
            ti = threads.add();
            if (!isOpen())
                return 0;
            do {
                n = IOUtil.read(fd, dst, -1, nd);
            } while ((n == IOStatus.INTERRUPTED) && isOpen());
            return IOStatus.normalize(n);
        } finally {
            threads.remove(ti);
            end(n > 0);
            assert IOStatus.check(n);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后目的就是将SocketChannel中的数据读出存放到**ByteBuffer dst**中**，**我们看看 IOUtil.read(fd, dst, -1, nd)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static int read(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
    if (var1.isReadOnly()) {
        throw new IllegalArgumentException("Read-only buffer");
    //如果最终承载数据的buffer是DirectBuffer，则直接将数据读入到堆外内存中
    } else if (var1 instanceof DirectBuffer) {
        return readIntoNativeBuffer(var0, var1, var2, var4);
    } else {
        // 分配临时的堆外内存
        ByteBuffer var5 = Util.getTemporaryDirectBuffer(var1.remaining());

        int var7;
        try {
            // Socket I/O 操作会将数据读入到堆外内存中
            int var6 = readIntoNativeBuffer(var0, var5, var2, var4);
            var5.flip();
            if (var6 > 0) {
                // 将堆外内存的数据拷贝到堆内存中（用户定义的缓存，在jvm中分配内存）
                var1.put(var5);
            }

            var7 = var6;
        } finally {
            // 里面会调用DirectBuffer.cleaner().clean()来释放临时的堆外内存
            Util.offerFirstTemporaryDirectBuffer(var5);
        }

        return var7;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过上述实现可以看出，基于channel的数据读取步骤如下：

1、如果缓存内存是DirectBuffer，就直接将Channel中的数据读取到堆外内存
2、如果缓存内存是堆内存，则先申请一块和缓存同大小的临时 DirectByteBuffer var5。
3、将内核缓存中的数据读到堆外缓存var5，底层由NativeDispatcher的read实现。
4、把堆外缓存var5的数据拷贝到堆内存var1（用户定义的缓存，在jvm中分配内存）。

5、会调用DirectBuffer.cleaner().clean()来释放创建的临时的堆外内存

如果AbstractNioByteChannel.read中第一步创建的是堆外内存，则会直接将数据读入到堆外内存，并不会先创建临时堆外内存，再将数据读入到堆外内存，最后将堆外内存拷贝到堆内存

简单的说，如果使用堆外内存，则只会复制一次数据，如果使用堆内存，则会复制两次数据

我们来看看readIntoNativeBuffer

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static int readIntoNativeBuffer(FileDescriptor filedescriptor, ByteBuffer bytebuffer, long l, NativeDispatcher nativedispatcher, Object obj)  throws IOException  {  
    int i = bytebuffer.position();  
    int j = bytebuffer.limit();  
    //如果断言开启，buffer的position大于limit，则抛出断言错误  
    if(!$assertionsDisabled && i > j)  
        throw new AssertionError();  
    //获取需要读的字节数  
    int k = i > j ? 0 : j - i;  
    if(k == 0)  
        return 0;  
    int i1 = 0;  
    //从输入流读取k个字节到buffer  
    if(l != -1L)  
        i1 = nativedispatcher.pread(filedescriptor, ((DirectBuffer)bytebuffer).address() + (long)i, k, l, obj);  
    else  
        i1 = nativedispatcher.read(filedescriptor, ((DirectBuffer)bytebuffer).address() + (long)i, k);  
    //重新定位buffer的position  
    if(i1 > 0)  
        bytebuffer.position(i + i1);  
    return i1;  
}  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个函数就是将内核缓冲区中的数据读取到堆外缓存DirectBuffer

回到AbstractNioByteChannel.read方法，继续看。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void read() {
        //...
        try {
            int totalReadAmount = 0;
            boolean readPendingReset = false;
            do {
                byteBuf = allocHandle.allocate(allocator);
                int writable = byteBuf.writableBytes();
                int localReadAmount = doReadBytes(byteBuf);
                if (localReadAmount <= 0) {
                    // not was read release the buffer
                    byteBuf.release();
                    close = localReadAmount < 0;
                    break;
                }
                if (!readPendingReset) {
                    readPendingReset = true;
                    setReadPending(false);
                }
                pipeline.fireChannelRead(byteBuf);
                byteBuf = null;

                if (totalReadAmount >= Integer.MAX_VALUE - localReadAmount) {
                    // Avoid overflow.
                    totalReadAmount = Integer.MAX_VALUE;
                    break;
                }

                totalReadAmount += localReadAmount;

                // stop reading
                if (!config.isAutoRead()) {
                    break;
                }

                if (localReadAmount < writable) {
                    // Read less than what the buffer can hold,
                    // which might mean we drained the recv buffer completely.
                    break;
                }
            } while (++ messages < maxMessagesPerRead);

            pipeline.fireChannelReadComplete();
            allocHandle.record(totalReadAmount);

            if (close) {
                closeOnRead(pipeline);
                close = false;
            }
        } catch (Throwable t) {
            handleReadException(pipeline, byteBuf, t, close);
        } finally {
            if (!config.isAutoRead() && !isReadPending()) {
                removeReadOp();
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

int localReadAmount = doReadBytes(byteBuf);
1、如果返回0，则表示没有读取到数据，则退出循环。
2、如果返回-1，表示对端已经关闭连接，则退出循环。
3、否则，表示读取到了数据，数据读入缓存后，触发pipeline的ChannelRead事件，byteBuf作为参数进行后续处理，这时自定义Inbound类型的handler就可以进行业务处理了。Pipeline的事件处理在我之前的博文中有详细的介绍。处理完成之后，再一次从Channel读取数据，直至退出循环。

4、循环次数超过maxMessagesPerRead时，即只能在管道中读取maxMessagesPerRead次数据，既是还没有读完也要退出。在上篇博文中，Boss线程接受客户端连接也用到了此变量，即当boss线程 selector检测到OP_ACCEPT事件后一次只能接受maxMessagesPerRead个客户端连接

# [Netty源码分析 （八）----- write过程 源码分析](https://www.cnblogs.com/java-chen-hao/p/11477385.html)

**正文**

上一篇文章主要讲了netty的read过程，本文主要分析一下write和writeAndFlush。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## 主要内容

本文分以下几个部分阐述一个java对象最后是如何转变成字节流，写到socket缓冲区中去的

1. pipeline中的标准链表结构
2. java对象编码过程
3. write：写队列
4. flush：刷新写队列
5. writeAndFlush: 写队列并刷新

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## pipeline中的标准链表结构

一个标准的pipeline链式结构如下

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907150343727-775140507.png)

数据从head节点流入，先拆包，然后解码成业务对象，最后经过业务Handler处理，调用write，将结果对象写出去。而写的过程先通过tail节点，然后通过encoder节点将对象编码成ByteBuf，最后将该ByteBuf对象传递到head节点，调用底层的Unsafe写到jdk底层管道

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## java对象编码过程

为什么我们在pipeline中添加了encoder节点，java对象就转换成netty可以处理的ByteBuf，写到管道里？

我们先看下调用`write`的code

**BusinessHandler**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void channelRead0(ChannelHandlerContext ctx, Request request) throws Exception {
    Response response = doBusiness(request);

    if (response != null) {
        ctx.channel().write(response);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

业务处理器接受到请求之后，做一些业务处理，返回一个`Response`，然后，response在pipeline中传递，落到 `Encoder`节点，我们来跟踪一下 **ctx.channel().write(response);**

```
public ChannelFuture write(Object msg) {
    return this.pipeline.write(msg);
}
```

调用了Channel中的pipeline中的write方法，我们接着看

```
public final ChannelFuture write(Object msg) {
    return this.tail.write(msg);
}
```

pipeline中有属性tail，调用tail中的write，由此我们知道write消息的时候，从tail开始，接着往下看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void write(Object msg, boolean flush, ChannelPromise promise) {
    AbstractChannelHandlerContext next = this.findContextOutbound();
    Object m = this.pipeline.touch(msg, next);
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        if (flush) {
            next.invokeWriteAndFlush(m, promise);
        } else {
            next.invokeWrite(m, promise);
        }
    } else {
        Object task;
        if (flush) {
            task = AbstractChannelHandlerContext.WriteAndFlushTask.newInstance(next, m, promise);
        } else {
            task = AbstractChannelHandlerContext.WriteTask.newInstance(next, m, promise);
        }

        safeExecute(executor, (Runnable)task, promise, m);
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

中间我省略了几个重载的方法，我们来看看第一行代码，**next =** **this****.findContextOutbound();**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private AbstractChannelHandlerContext findContextOutbound() {
    AbstractChannelHandlerContext ctx = this;

    do {
        ctx = ctx.prev;
    } while(!ctx.outbound);

    return ctx;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过 **ctx** **=** **ctx.prev; 我们知道从tail开始找到pipeline中的第一个****outbound的handler，然后调用** **invokeWrite(m, promise)，此时找到的第一个outbound的handler就是我们自定义的编码器Encoder**

我们接着看 next.invokeWrite(m, promise);

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeWrite(Object msg, ChannelPromise promise) {
    if (this.invokeHandler()) {
        this.invokeWrite0(msg, promise);
    } else {
        this.write(msg, promise);
    }

}
private void invokeWrite0(Object msg, ChannelPromise promise) {
    try {
        ((ChannelOutboundHandler)this.handler()).write(this, msg, promise);
    } catch (Throwable var4) {
        notifyOutboundHandlerException(var4, promise);
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

一路代码跟下来，我们可以知道是调用了第一个outBound类型的handler中的write方法，也就是第一个调用的是我们自定义**编码器Encoder的write方法**

我们来看看自定义Encoder

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Encoder extends MessageToByteEncoder<Response> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Response response, ByteBuf out) throws Exception {
        out.writeByte(response.getVersion());
        out.writeInt(4 + response.getData().length);
        out.writeBytes(response.getData());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

自定义Encoder继承 **MessageToByteEncoder ，并且重写了** encode方法，这就是编码器的核心，我们先来看 **MessageToByteEncoder**

```
public abstract class MessageToByteEncoder<I> extends ChannelOutboundHandlerAdapter {
```

我们看到 MessageToByteEncoder 继承了 **ChannelOutboundHandlerAdapter，说明了** Encoder 是一个 **Outbound的handler**

我们来看看 Encoder 的父类 MessageToByteEncoder中的write方法

**MessageToByteEncoder**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    ByteBuf buf = null;
    try {
        // 判断当前Handelr是否能处理写入的消息
        if (acceptOutboundMessage(msg)) {
            @SuppressWarnings("unchecked")
            // 强制换换
            I cast = (I) msg;
            // 分配一段ButeBuf
            buf = allocateBuffer(ctx, cast, preferDirect);
            try {
            // 调用encode，这里就调回到  `Encoder` 这个Handelr中    
                encode(ctx, cast, buf);
            } finally {
                // 既然自定义java对象转换成ByteBuf了，那么这个对象就已经无用了，释放掉
                // (当传入的msg类型是ByteBuf的时候，就不需要自己手动释放了)
                ReferenceCountUtil.release(cast);
            }
            // 如果buf中写入了数据，就把buf传到下一个节点
            if (buf.isReadable()) {
                ctx.write(buf, promise);
            } else {
            // 否则，释放buf，将空数据传到下一个节点    
                buf.release();
                ctx.write(Unpooled.EMPTY_BUFFER, promise);
            }
            buf = null;
        } else {
            // 如果当前节点不能处理传入的对象，直接扔给下一个节点处理
            ctx.write(msg, promise);
        }
    } catch (EncoderException e) {
        throw e;
    } catch (Throwable e) {
        throw new EncoderException(e);
    } finally {
        // 当buf在pipeline中处理完之后，释放
        if (buf != null) {
            buf.release();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里，我们详细阐述一下Encoder是如何处理传入的java对象的

1.判断当前Handler是否能处理写入的消息，如果能处理，进入下面的流程，否则，直接扔给下一个节点处理
2.将对象强制转换成`Encoder`可以处理的 `Response`对象
3.分配一个ByteBuf
4.调用encoder，即进入到 `Encoder` 的 `encode`方法，该方法是用户代码，用户将数据写入ByteBuf
5.既然自定义java对象转换成ByteBuf了，那么这个对象就已经无用了，释放掉，(当传入的msg类型是ByteBuf的时候，就不需要自己手动释放了)
6.如果buf中写入了数据，就把buf传到下一个节点，否则，释放buf，将空数据传到下一个节点
7.最后，当buf在pipeline中处理完之后，释放节点

总结一点就是，`Encoder`节点分配一个ByteBuf，调用`encode`方法，将java对象根据自定义协议写入到ByteBuf，然后再把ByteBuf传入到下一个节点，在我们的例子中，最终会传入到head节点，因为head节点是一个OutBount类型的handler

**HeadContext**

```
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
    unsafe.write(msg, promise);
}
```

这里的msg就是前面在`Encoder`节点中，载有java对象数据的自定义ByteBuf对象，进入下一节

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## write：写队列

我们来看看channel中unsafe的write方法，先来看看其中的一个属性

**AbstractUnsafe**

```
protected abstract class AbstractUnsafe implements Unsafe {
    private volatile ChannelOutboundBuffer outboundBuffer = new ChannelOutboundBuffer(AbstractChannel.this);
```

我们来看看 ChannelOutboundBuffer 这个类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final class ChannelOutboundBuffer {
    private final Channel channel;
    private ChannelOutboundBuffer.Entry flushedEntry;
    private ChannelOutboundBuffer.Entry unflushedEntry;
    private ChannelOutboundBuffer.Entry tailEntry;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ChannelOutboundBuffer内部维护了一个Entry链表，并使用Entry封装msg。其中的属性我们下面会详细讲

我们回到正题，接着看 unsafe.write(msg, promise);

**AbstractUnsafe**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
public final void write(Object msg, ChannelPromise promise) {
    assertEventLoop();

    ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;

    int size;
    try {
        msg = filterOutboundMessage(msg);
        size = pipeline.estimatorHandle().size(msg);
        if (size < 0) {
            size = 0;
        }
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        ReferenceCountUtil.release(msg);
        return;
    }

    outboundBuffer.addMessage(msg, size, promise);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1.调用 `filterOutboundMessage()` 方法，将待写入的对象过滤，把非`ByteBuf`对象和`FileRegion`过滤，把所有的非直接内存转换成直接内存`DirectBuffer`

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected final Object filterOutboundMessage(Object msg) {
    if (msg instanceof ByteBuf) {
        ByteBuf buf = (ByteBuf) msg;
        if (buf.isDirect()) {
            return msg;
        }

        return newDirectBuffer(buf);
    }

    if (msg instanceof FileRegion) {
        return msg;
    }

    throw new UnsupportedOperationException(
            "unsupported message type: " + StringUtil.simpleClassName(msg) + EXPECTED_TYPES);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2.接下来，估算出需要写入的ByteBuf的size
3.最后，调用 `ChannelOutboundBuffer` 的`addMessage(msg, size, promise)` 方法，所以，接下来，我们需要重点看一下这个方法干了什么事情

**ChannelOutboundBuffer**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void addMessage(Object msg, int size, ChannelPromise promise) {
    // 创建一个待写出的消息节点
    Entry entry = Entry.newInstance(msg, size, total(msg), promise);
    if (tailEntry == null) {
        flushedEntry = null;
        tailEntry = entry;
    } else {
        Entry tail = tailEntry;
        tail.next = entry;
        tailEntry = entry;
    }
    if (unflushedEntry == null) {
        unflushedEntry = entry;
    }

    incrementPendingOutboundBytes(size, false);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

想要理解上面这段代码，必须得掌握写缓存中的几个消息指针，如下图

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907155456857-699540739.png)

ChannelOutboundBuffer 里面的数据结构是一个单链表结构，每个节点是一个 `Entry`，`Entry` 里面包含了待写出`ByteBuf` 以及消息回调 `promise`，下面分别是三个指针的作用

1.flushedEntry 指针表示第一个被写到操作系统Socket缓冲区中的节点
2.unFlushedEntry 指针表示第一个未被写入到操作系统Socket缓冲区中的节点
3.tailEntry指针表示ChannelOutboundBuffer缓冲区的最后一个节点

初次调用 `addMessage` 之后，各个指针的情况为

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907155531356-1086381360.png)

`fushedEntry`指向空，`unFushedEntry`和 `tailEntry` 都指向新加入的节点

第二次调用 `addMessage`之后，各个指针的情况为

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907155558203-160912103.png)

第n次调用 `addMessage`之后，各个指针的情况为

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907155623379-631375004.png)

可以看到，调用n次`addMessage`，flushedEntry指针一直指向NULL，表示现在还未有节点需要写出到Socket缓冲区，而`unFushedEntry`之后有n个节点，表示当前还有n个节点尚未写出到Socket缓冲区中去

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## flush：刷新写队列

不管调用`channel.flush()`，还是`ctx.flush()`，最终都会落地到pipeline中的head节点

**HeadContext**

```
@Override
public void flush(ChannelHandlerContext ctx) throws Exception {
    unsafe.flush();
}
```

之后进入到`AbstractUnsafe`

**AbstractUnsafe**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final void flush() {
   assertEventLoop();

   ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
   if (outboundBuffer == null) {
       return;
   }

   outboundBuffer.addFlush();
   flush0();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

flush方法中，先调用 `outboundBuffer.addFlush();`

**ChannelOutboundBuffer**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void addFlush() {
    Entry entry = unflushedEntry;
    if (entry != null) {
        if (flushedEntry == null) {
            flushedEntry = entry;
        }
        do {
            flushed ++;
            if (!entry.promise.setUncancellable()) {
                int pending = entry.cancel();
                decrementPendingOutboundBytes(pending, false, true);
            }
            entry = entry.next;
        } while (entry != null);
        unflushedEntry = null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以结合前面的图来看，首先拿到 `unflushedEntry` 指针，然后将 `flushedEntry` 指向`unflushedEntry`所指向的节点，调用完毕之后，三个指针的情况如下所示

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907160323444-743199654.png)

 

相当于所有的节点都即将开始推送出去

接下来，调用 `flush0();`

**AbstractUnsafe**

```
protected void flush0() {
    doWrite(outboundBuffer);
}
```

发现这里的核心代码就一个 doWrite，继续跟

**AbstractNioByteChannel**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void doWrite(ChannelOutboundBuffer in) throws Exception {
    int writeSpinCount = -1;

    boolean setOpWrite = false;
    for (;;) {
        // 拿到第一个需要flush的节点的数据
        Object msg = in.current();

        if (msg instanceof ByteBuf) {
            // 强转为ByteBuf，若发现没有数据可读，直接删除该节点
            ByteBuf buf = (ByteBuf) msg;

            boolean done = false;
            long flushedAmount = 0;
            // 拿到自旋锁迭代次数
            if (writeSpinCount == -1) {
                writeSpinCount = config().getWriteSpinCount();
            }
            // 自旋，将当前节点写出
            for (int i = writeSpinCount - 1; i >= 0; i --) {
                int localFlushedAmount = doWriteBytes(buf);
                if (localFlushedAmount == 0) {
                    setOpWrite = true;
                    break;
                }

                flushedAmount += localFlushedAmount;
                if (!buf.isReadable()) {
                    done = true;
                    break;
                }
            }

            in.progress(flushedAmount);

            // 写完之后，将当前节点删除
            if (done) {
                in.remove();
            } else {
                break;
            }
        } 
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里略微有点复杂，我们分析一下

1.第一步，调用`current()`先拿到第一个需要flush的节点的数据

 **ChannelOutBoundBuffer**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object current() {
    Entry entry = flushedEntry;
    if (entry == null) {
        return null;
    }

    return entry.msg;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2.第二步,拿到自旋锁的迭代次数

```
if (writeSpinCount == -1) {
    writeSpinCount = config().getWriteSpinCount();
}
```

3.自旋的方式将ByteBuf写出到jdk nio的Channel

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
for (int i = writeSpinCount - 1; i >= 0; i --) {
    int localFlushedAmount = doWriteBytes(buf);
    if (localFlushedAmount == 0) {
        setOpWrite = true;
        break;
    }

    flushedAmount += localFlushedAmount;
    if (!buf.isReadable()) {
        done = true;
        break;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`doWriteBytes` 方法跟进去

```
protected int doWriteBytes(ByteBuf buf) throws Exception {
    final int expectedWrittenBytes = buf.readableBytes();
    return buf.readBytes(javaChannel(), expectedWrittenBytes);
}
```

我们发现，出现了 `javaChannel()`，表明已经进入到了jdk nio Channel的领域，我们来看看 buf.readBytes(javaChannel(), expectedWrittenBytes);

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public int readBytes(GatheringByteChannel out, int length) throws IOException {
    this.checkReadableBytes(length);
    int readBytes = this.getBytes(this.readerIndex, out, length);
    this.readerIndex += readBytes;
    return readBytes;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看关键代码 **this.getBytes(this.readerIndex, out, length)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private int getBytes(int index, GatheringByteChannel out, int length, boolean internal) throws IOException {
    this.checkIndex(index, length);
    if (length == 0) {
        return 0;
    } else {
        ByteBuffer tmpBuf;
        if (internal) {
            tmpBuf = this.internalNioBuffer();
        } else {
            tmpBuf = ((ByteBuffer)this.memory).duplicate();
        }

        index = this.idx(index);
        tmpBuf.clear().position(index).limit(index + length);
        //将tmpBuf中的数据写到out中
        return out.write(tmpBuf);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看out.write(tmpBuf)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public int write(ByteBuffer src) throws IOException {
    ensureOpen();
    if (!writable)
        throw new NonWritableChannelException();
    synchronized (positionLock) {
        int n = 0;
        int ti = -1;
        try {
            begin();
            ti = threads.add();
            if (!isOpen())
                return 0;
            do {
                n = IOUtil.write(fd, src, -1, nd);
            } while ((n == IOStatus.INTERRUPTED) && isOpen());
            return IOStatus.normalize(n);
        } finally {
            threads.remove(ti);
            end(n > 0);
            assert IOStatus.check(n);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

和read实现一样，SocketChannelImpl的write方法通过IOUtil的write实现：关键代码 **n** **= IOUtil.write(fd, src, -1****, nd);**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static int write(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
    //如果是DirectBuffer，直接写，将堆外缓存中的数据拷贝到内核缓存中进行发送
    if (var1 instanceof DirectBuffer) {
        return writeFromNativeBuffer(var0, var1, var2, var4);
    } else {
        //非DirectBuffer
        //获取已经读取到的位置
        int var5 = var1.position();
        //获取可以读到的位置
        int var6 = var1.limit();

        assert var5 <= var6;
        //申请一个原buffer可读大小的DirectByteBuffer
        int var7 = var5 <= var6 ? var6 - var5 : 0;
        ByteBuffer var8 = Util.getTemporaryDirectBuffer(var7);

        int var10;
        try {

            var8.put(var1);
            var8.flip();
            var1.position(var5);
            //通过DirectBuffer写，将堆外缓存的数据拷贝到内核缓存中进行发送
            int var9 = writeFromNativeBuffer(var0, var8, var2, var4);
            if (var9 > 0) {
                var1.position(var5 + var9);
            }

            var10 = var9;
        } finally {
            //回收分配的DirectByteBuffer
            Util.offerFirstTemporaryDirectBuffer(var8);
        }

        return var10;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

代码逻辑我们就不再讲了，代码注释已经很清楚了，这里我们关注一点，我们可以看看我们前面的一个方法 `filterOutboundMessage()`，将待写入的对象过滤，把非`ByteBuf`对象和`FileRegion`过滤，把所有的非直接内存转换成直接内存`DirectBuffer`

说明到了这一步所有的 var1 意境是直接内存`DirectBuffer，就不需要走到`else，`就不需要write两次了`

4.删除该节点

节点的数据已经写入完毕，接下来就需要删除该节点

**ChannelOutBoundBuffer**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean remove() {
    Entry e = flushedEntry;
    Object msg = e.msg;

    ChannelPromise promise = e.promise;
    int size = e.pendingSize;

    removeEntry(e);

    if (!e.cancelled) {
        ReferenceCountUtil.safeRelease(msg);
        safeSuccess(promise);
    }

    // recycle the entry
    e.recycle();

    return true;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先拿到当前被flush掉的节点(flushedEntry所指)，然后拿到该节点的回调对象 `ChannelPromise`, 调用 `removeEntry()`方法移除该节点

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void removeEntry(Entry e) {
    if (-- flushed == 0) {
        flushedEntry = null;
        if (e == tailEntry) {
            tailEntry = null;
            unflushedEntry = null;
        }
    } else {
        flushedEntry = e.next;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里的remove是逻辑移除，只是将flushedEntry指针移到下个节点，调用完毕之后，节点图示如下

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190907160953354-1709189976.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## writeAndFlush: 写队列并刷新

理解了write和flush这两个过程，`writeAndFlush` 也就不难了

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final ChannelFuture writeAndFlush(Object msg) {
    return tail.writeAndFlush(msg);
}

public ChannelFuture writeAndFlush(Object msg) {
    return writeAndFlush(msg, newPromise());
}

public ChannelFuture writeAndFlush(Object msg, ChannelPromise promise) {
    write(msg, true, promise);

    return promise;
}

private void write(Object msg, boolean flush, ChannelPromise promise) {
    AbstractChannelHandlerContext next = findContextOutbound();
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        if (flush) {
            next.invokeWriteAndFlush(m, promise);
        } else {
            next.invokeWrite(m, promise);
        }
    } 
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到，最终，通过一个boolean变量，表示是调用 `invokeWriteAndFlush`，还是 `invokeWrite`，`invokeWrite`便是我们上文中的`write`过程

```
private void invokeWriteAndFlush(Object msg, ChannelPromise promise) {
    invokeWrite0(msg, promise);
    invokeFlush0();
}
```

可以看到，最终调用的底层方法和单独调用 `write` 和 `flush` 是一样的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void invokeWrite(Object msg, ChannelPromise promise) {
        invokeWrite0(msg, promise);
}

private void invokeFlush(Object msg, ChannelPromise promise) {
        invokeFlush0(msg, promise);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

由此看来，`invokeWriteAndFlush`基本等价于`write`方法之后再来一次`flush`

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11477385.html#_labelTop)

## 总结

1.pipeline中的编码器原理是创建一个ByteBuf,将java对象转换为ByteBuf，然后再把ByteBuf继续向前传递
2.调用write方法并没有将数据写到Socket缓冲区中，而是写到了一个单向链表的数据结构中，flush才是真正的写出
3.writeAndFlush等价于先将数据写到netty的缓冲区，再将netty缓冲区中的数据写到Socket缓冲区中，写的过程与并发编程类似，用自旋锁保证写成功
4.netty中的缓冲区中的ByteBuf为DirectByteBuf

# [Netty源码分析 （九）----- 拆包器的奥秘](https://www.cnblogs.com/java-chen-hao/p/11436512.html)

**正文**

Netty 的解码器有很多种，比如基于长度的，基于分割符的，私有协议的。但是，总体的思路都是一致的。

拆包思路：当数据满足了 解码条件时，将其拆开。放到数组。然后发送到业务 handler 处理。

半包思路： 当读取的数据不够时，先存起来，直到满足解码条件后，放进数组。送到业务 handler 处理。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11436512.html#_labelTop)

## 拆包的原理

在没有netty的情况下，用户如果自己需要拆包，基本原理就是不断从TCP缓冲区中读取数据，每次读取完都需要判断是否是一个完整的数据包

1.如果当前读取的数据不足以拼接成一个完整的业务数据包，那就保留该数据，继续从tcp缓冲区中读取，直到得到一个完整的数据包
2.如果当前读到的数据加上已经读取的数据足够拼接成一个数据包，那就将已经读取的数据拼接上本次读取的数据，够成一个完整的业务数据包传递到业务逻辑，多余的数据仍然保留，以便和下次读到的数据尝试拼接

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11436512.html#_labelTop)

## netty中拆包的基类

netty 中的拆包也是如上这个原理，在每个SocketChannel中会一个 pipeline ，pipeline 内部会加入解码器，解码器都继承基类 `ByteToMessageDecoder，其`内部会有一个累加器，每次从当前SocketChannel读取到数据都会不断累加，然后尝试对累加到的数据进行拆包，拆成一个完整的业务数据包，下面我们先详细分析下这个类

看名字的意思是：将字节转换成消息的解码器。人如其名。而他本身也是一个入站 handler，所以，我们还是从他的 channelRead 方法入手。



### channelRead 方法

我们先看看基类中的属性，cumulation是此基类中的一个 ByteBuf 类型的累积区，每次从当前SocketChannel读取到数据都会不断累加，然后尝试对累加到的数据进行拆包，拆成一个完整的业务数据包，如果不够一个完整的数据包，则等待下一次从TCP的数据到来，继续累加到此cumulation中

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public abstract class ByteToMessageDecoder extends ChannelInboundHandlerAdapter {
    //累积区
    ByteBuf cumulation;
    private ByteToMessageDecoder.Cumulator cumulator;
    private boolean singleDecode;
    private boolean decodeWasNull;
    private boolean first;
    private int discardAfterReads;
    private int numReads;
    .
    .
    .
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
channelRead`方法是每次从TCP缓冲区读到数据都会调用的方法，触发点在`AbstractNioByteChannel`的`read`方法中，里面有个`while`循环不断读取，读取到一次就触发一次`channelRead
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 @Override
 2 public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
 3     if (msg instanceof ByteBuf) {
 4         // 从对象池中取出一个List
 5         CodecOutputList out = CodecOutputList.newInstance();
 6         try {
 7             ByteBuf data = (ByteBuf) msg;
 8             first = cumulation == null;
 9             if (first) {
10                 // 第一次解码
11                 cumulation = data;//直接赋值
12             } else {
13                  // 第二次解码，就将 data 向 cumulation 追加，并释放 data
14                 cumulation = cumulator.cumulate(ctx.alloc(), cumulation, data);
15             }
16             // 得到追加后的 cumulation 后，调用 decode 方法进行解码
17             // 主要目的是将累积区cumulation的内容 decode 到 out数组中
18             callDecode(ctx, cumulation, out);
19         } catch (DecoderException e) {
20             throw e;
21         } catch (Throwable t) {
22             throw new DecoderException(t);
23         } finally {
24             // 如果累计区没有可读字节了，有可能在上面callDecode方法中已经将cumulation全部读完了，此时writerIndex==readerIndex
25             // 每读一个字节，readerIndex会+1
26             if (cumulation != null && !cumulation.isReadable()) {
27                 // 将次数归零
28                 numReads = 0;
29                 // 释放累计区,因为累计区里面的字节都全部读完了
30                 cumulation.release();
31                 // 便于 gc
32                 cumulation = null;
33             // 如果超过了 16 次，还有字节没有读完，就将已经读过的数据丢弃，将 readIndex 归零。
34             } else if (++ numReads >= discardAfterReads) {
35                 numReads = 0;
36                 //将已经读过的数据丢弃，将 readIndex 归零。
37                 discardSomeReadBytes();
38             }
39 
40             int size = out.size();
41             decodeWasNull = !out.insertSinceRecycled();
42             //循环数组，向后面的 handler 发送数据
43             fireChannelRead(ctx, out, size);
44             out.recycle();
45         }
46     } else {
47         ctx.fireChannelRead(msg);
48     }
49 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. 从对象池中取出一个空的数组。
2. 判断成员变量是否是第一次使用，将 unsafe 中传递来的数据写入到这个 cumulation 累积区中。
3. 写到累积区后，在callDecode方法中调用子类的 decode 方法，尝试将累积区的内容解码，每成功解码一个，就调用后面节点的 channelRead 方法。若没有解码成功，什么都不做。
4. 如果累积区没有未读数据了，就释放累积区。
5. 如果还有未读数据，且解码超过了 16 次（默认），就对累积区进行压缩。将读取过的数据清空，也就是将 readIndex 设置为0.
6. 调用 fireChannelRead 方法，将数组中的元素发送到后面的 handler 中。
7. 将数组清空。并还给对象池。

下面来说说详细的步骤。

#### 写入累积区

如果当前累加器没有数据，就直接跳过内存拷贝，直接将字节容器的指针指向新读取的数据，否则，调用累加器累加数据至字节容器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ByteBuf data = (ByteBuf) msg;
first = cumulation == null;
if (first) {
    cumulation = data;
} else {
    cumulation = cumulator.cumulate(ctx.alloc(), cumulation, data);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看看构造方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected ByteToMessageDecoder() {
    this.cumulator = MERGE_CUMULATOR;
    this.discardAfterReads = 16;
    CodecUtil.ensureNotSharable(this);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到 this.cumulator = MERGE_CUMULATOR;，那我们接下来看看 MERGE_CUMULATOR

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static final ByteToMessageDecoder.Cumulator MERGE_CUMULATOR = new ByteToMessageDecoder.Cumulator() {
    public ByteBuf cumulate(ByteBufAllocator alloc, ByteBuf cumulation, ByteBuf in) {
        ByteBuf buffer;
        if (cumulation.writerIndex() <= cumulation.maxCapacity() - in.readableBytes() && cumulation.refCnt() <= 1) {
            buffer = cumulation;
        } else {
            buffer = ByteToMessageDecoder.expandCumulation(alloc, cumulation, in.readableBytes());
        }

        buffer.writeBytes(in);
        in.release();
        return buffer;
    }
};
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

MERGE_CUMULATOR是基类ByteToMessageDecoder中的一个静态常量，其重写了cumulate方法，下面我们看一下 `MERGE_CUMULATOR` 是如何将新读取到的数据累加到字节容器里的

netty 中ByteBuf的抽象，使得累加非常简单，通过一个简单的api调用 `buffer.writeBytes(in);` 便将新数据累加到字节容器中，为了防止字节容器大小不够，在累加之前还进行了扩容处理

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static ByteBuf expandCumulation(ByteBufAllocator alloc, ByteBuf cumulation, int readable) {
        ByteBuf oldCumulation = cumulation;
        cumulation = alloc.buffer(oldCumulation.readableBytes() + readable);
        cumulation.writeBytes(oldCumulation);
        oldCumulation.release();
        return cumulation;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

扩容也是一个内存拷贝操作，新增的大小即是新读取数据的大小

#### 将累加到的数据传递给业务进行拆包

当数据追加到累积区之后，需要调用 decode 方法进行解码，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean isReadable() {
    //写的坐标大于读的坐标则说明还有数据可读
    return this.writerIndex > this.readerIndex;
}
protected void callDecode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    // 如果累计区还有可读字节，循环解码，因为这里in有可能是粘包，即多次完整的数据包粘在一起，通过换行符连接
    // 下面的decode方法只能处理一个完整的数据包，所以这里循环处理粘包
    while (in.isReadable()) {
        int outSize = out.size();
        // 上次循环成功解码
        if (outSize > 0) {
            // 处理一个粘包就 调用一次后面的业务 handler 的  ChannelRead 方法
            fireChannelRead(ctx, out, outSize);
            // 将 size 置为0
            out.clear();//
            if (ctx.isRemoved()) {
                break;
            }
            outSize = 0;
        }
        // 得到可读字节数
        int oldInputLength = in.readableBytes();
        // 调用 decode 方法，将成功解码后的数据放入道 out 数组中
        decode(ctx, in, out);
        if (ctx.isRemoved()) {
            break;
        }
        if (outSize == out.size()) {
            if (oldInputLength == in.readableBytes()) {
                break;
            } else {
                continue;
            }
        }
        if (isSingleDecode()) {
            break;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看看 fireChannelRead

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static void fireChannelRead(ChannelHandlerContext ctx, List<Object> msgs, int numElements) {
    if (msgs instanceof CodecOutputList) {
        fireChannelRead(ctx, (CodecOutputList)msgs, numElements);
    } else {
        //将所有已解码的数据向下业务hadder传递
        for(int i = 0; i < numElements; ++i) {
            ctx.fireChannelRead(msgs.get(i));
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该方法主要逻辑：只要累积区还有未读数据，就循环进行读取。

1. 调用 decodeRemovalReentryProtection 方法，内部调用了子类重写的 decode 方法，很明显，这里是个模板模式。decode 方法的逻辑就是将累积区的内容按照约定进行解码，如果成功解码，就添加到数组中。同时该方法也会检查该 handler 的状态，如果被移除出 pipeline 了，就将累积区的内容直接刷新到后面的 handler 中。
2. 如果 Context 节点被移除了，直接结束循环。如果解码前的数组大小和解码后的数组大小相等，且累积区的可读字节数没有变化，说明此次读取什么都没做，就直接结束。如果字节数变化了，说明虽然数组没有增加，但确实在读取字节，就再继续读取。
3. 如果上面的判断过了，说明数组读到数据了，但如果累积区的 readIndex 没有变化，则抛出异常，说明没有读取数据，但数组却增加了，子类的操作是不对的。
4. 如果是个单次解码器，解码一次就直接结束了，如果数据包一次就解码完了，则下一次循环时 in.isReadable()就为false，因为 writerIndex = this.readerIndex 了

所以，这段代码的关键就是子类需要重写 decode 方法，将累积区的数据正确的解码并添加到数组中。每添加一次成功，就会调用 fireChannelRead 方法，将数组中的数据传递给后面的 handler。完成之后将数组的 size 设置为 0.

所以，如果你的业务 handler 在这个地方可能会被多次调用。也可能一次也不调用。取决于数组中的值。

解码器最主要的逻辑：

> 将 read 方法的数据读取到累积区，使用解码器解码累积区的数据，解码成功一个就放入到一个数组中，并将数组中的数据一次次的传递到后面的handler。

#### 清理字节容器

业务拆包完成之后，只是从累积区中取走了数据，但是这部分空间对于累积区来说依然保留着，而字节容器每次累加字节数据的时候都是将字节数据追加到尾部，如果不对累积区做清理，那么时间一长就会OOM，清理部分的代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
finally {
    // 如果累计区没有可读字节了，有可能在上面callDecode方法中已经将cumulation全部读完了，此时writerIndex==readerIndex
    // 每读一个字节，readerIndex会+1
    if (cumulation != null && !cumulation.isReadable()) {
        // 将次数归零
        numReads = 0;
        // 释放累计区,因为累计区里面的字节都全部读完了
        cumulation.release();
        // 便于 gc
        cumulation = null;
    // 如果超过了 16 次，还有字节没有读完，就将已经读过的数据丢弃，将 readIndex 归零。
    } else if (++ numReads >= discardAfterReads) {
        numReads = 0;
        //将已经读过的数据丢弃，将 readIndex 归零。
        discardSomeReadBytes();
    }

    int size = out.size();
    decodeWasNull = !out.insertSinceRecycled();
    //循环数组，向后面的 handler 发送数据
    fireChannelRead(ctx, out, size);
    out.recycle();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. 如果累积区没有可读数据了，将计数器归零，并释放累积区。
2. 如果不满足上面的条件，且计数器超过了 16 次，就压缩累积区的内容，压缩手段是删除已读的数据。将 readIndex 置为 0。还记得 ByteBuf 的指针结构吗？

![img](https://img2018.cnblogs.com/blog/1168971/201908/1168971-20190830180700717-733116891.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ByteBuf discardSomeReadBytes() {
    this.ensureAccessible();
    if (this.readerIndex == 0) {
        return this;
    } else if (this.readerIndex == this.writerIndex) {
        this.adjustMarkers(this.readerIndex);
        this.writerIndex = this.readerIndex = 0;
        return this;
    } else {
        //读指针超过了Buffer容量的一半时做清理工作
        if (this.readerIndex >= this.capacity() >>> 1) {
            //拷贝，从readerIndex开始，拷贝this.writerIndex - this.readerIndex 长度
            this.setBytes(0, this, this.readerIndex, this.writerIndex - this.readerIndex);
            //writerIndex=writerIndex-readerIndex
            this.writerIndex -= this.readerIndex;
            this.adjustMarkers(this.readerIndex);
            //将读指针重置为0
            this.readerIndex = 0;
        }

        return this;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到discardSomeReadBytes 主要是将未读的数据拷贝到原Buffer，重置 readerIndex 和 writerIndex 

我们看到最后还调用 fireChannelRead 方法，尝试将数组中的数据发送到后面的 handler。为什么要这么做。按道理，到这一步的时候，数组不可能是空，为什么这里还要这么谨慎的再发送一次？

如果是单次解码器，就需要发送了，因为单词解码器是不会在 callDecode 方法中发送的。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11436512.html#_labelTop)

## 总结

可以说，ByteToMessageDecoder 是解码器的核心所做，Netty 在这里使用了模板模式，留给子类扩展的方法就是 decode 方法。

主要逻辑就是将所有的数据全部放入累积区，子类从累积区取出数据进行解码后放入到一个 数组中，ByteToMessageDecoder 会循环数组调用后面的 handler 方法，将数据一帧帧的发送到业务 handler 。完成这个的解码逻辑。

使用这种方式，无论是粘包还是拆包，都可以完美的实现。

Netty 所有的解码器，都可以在此类上扩展，一切取决于 decode 的实现。只要遵守 ByteToMessageDecoder 的约定即可。

# [Netty源码分析 （十）----- 拆包器之LineBasedFrameDecoder](https://www.cnblogs.com/java-chen-hao/p/11445297.html)

**正文**

Netty 自带多个粘包拆包解码器。今天介绍 LineBasedFrameDecoder，换行符解码器。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11445297.html#_labelTop)

## 行拆包器

下面，以一个具体的例子来看看业netty自带的拆包器是如何来拆包的

这个类叫做 `LineBasedFrameDecoder`，基于行分隔符的拆包器，TA可以同时处理 `\n`以及`\r\n`两种类型的行分隔符，核心方法都在继承的 `decode` 方法中

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected final void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
    Object decoded = decode(ctx, in);
    if (decoded != null) {
        out.add(decoded);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

netty 中自带的拆包器都是如上这种模板，我们来看看decode(ctx, in);

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected Object decode(ChannelHandlerContext ctx, ByteBuf buffer) throws Exception {
    int eol = findEndOfLine(buffer);
    int length;
    int length;
    if (!this.discarding) {
        if (eol >= 0) {
            length = eol - buffer.readerIndex();
            int delimLength = buffer.getByte(eol) == '\r' ? 2 : 1;
            if (length > this.maxLength) {
                buffer.readerIndex(eol + delimLength);
                this.fail(ctx, length);
                return null;
            } else {
                ByteBuf frame;
                if (this.stripDelimiter) {
                    frame = buffer.readRetainedSlice(length);
                    buffer.skipBytes(delimLength);
                } else {
                    frame = buffer.readRetainedSlice(length + delimLength);
                }

                return frame;
            }
        } else {
            length = buffer.readableBytes();
            if (length > this.maxLength) {
                this.discardedBytes = length;
                buffer.readerIndex(buffer.writerIndex());
                this.discarding = true;
                if (this.failFast) {
                    this.fail(ctx, "over " + this.discardedBytes);
                }
            }

            return null;
        }
    } else {
        if (eol >= 0) {
            length = this.discardedBytes + eol - buffer.readerIndex();
            length = buffer.getByte(eol) == '\r' ? 2 : 1;
            buffer.readerIndex(eol + length);
            this.discardedBytes = 0;
            this.discarding = false;
            if (!this.failFast) {
                this.fail(ctx, length);
            }
        } else {
            this.discardedBytes += buffer.readableBytes();
            buffer.readerIndex(buffer.writerIndex());
        }

        return null;
    }
}

ByteProcessor FIND_LF = new IndexOfProcessor((byte) '\n');

private static int findEndOfLine(ByteBuf buffer) {
    int i = buffer.forEachByte(ByteProcessor.FIND_LF);
    if (i > 0 && buffer.getByte(i - 1) == '\r') {
        --i;
    }

    return i;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 找到换行符位置

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final int eol = findEndOfLine(buffer);

private static int findEndOfLine(final ByteBuf buffer) {
    int i = buffer.forEachByte(ByteProcessor.FIND_LF);
    if (i > 0 && buffer.getByte(i - 1) == '\r') {
        i--;
    }
    return i;
}

ByteProcessor FIND_LF = new IndexOfProcessor((byte) '\n');
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

for循环遍历，找到第一个 `\n` 的位置,如果`\n`前面的字符为`\r`，那就返回`\r`的位置



### 非discarding模式的处理

接下来，netty会判断，当前拆包是否属于丢弃模式，用一个成员变量来标识

```
private boolean discarding;
```

第一次拆包不在discarding模式

#### 非discarding模式下找到行分隔符的处理

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 1.计算分隔符和包长度
final ByteBuf frame;
final int length = eol - buffer.readerIndex();
final int delimLength = buffer.getByte(eol) == '\r'? 2 : 1;

// 丢弃异常数据
if (length > maxLength) {
    buffer.readerIndex(eol + delimLength);
    fail(ctx, length);
    return null;
}

// 取包的时候是否包括分隔符
if (stripDelimiter) {
    frame = buffer.readRetainedSlice(length);
    buffer.skipBytes(delimLength);
} else {
    frame = buffer.readRetainedSlice(length + delimLength);
}
return frame;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1.首先，新建一个帧，计算一下当前包的长度和分隔符的长度（因为有两种分隔符）
2.然后判断一下需要拆包的长度是否大于该拆包器允许的最大长度(`maxLength`)，这个参数在构造函数中被传递进来，如超出允许的最大长度，就将这段数据抛弃，返回null
3.最后，将一个完整的数据包取出，如果构造本解包器的时候指定 `stripDelimiter`为false，即解析出来的包包含分隔符，默认为不包含分隔符

#### 非discarding模式下未找到分隔符的处理

没有找到对应的行分隔符，说明字节容器没有足够的数据拼接成一个完整的业务数据包，进入如下流程处理

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final int length = buffer.readableBytes();
if (length > maxLength) {
    discardedBytes = length;
    buffer.readerIndex(buffer.writerIndex());
    discarding = true;
    if (failFast) {
        fail(ctx, "over " + discardedBytes);
    }
}
return null;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

首先取得当前字节容器的可读字节个数，接着，判断一下是否已经超过可允许的最大长度，如果没有超过，直接返回null，字节容器中的数据没有任何改变，否则，就需要进入丢弃模式

使用一个成员变量 `discardedBytes` 来表示已经丢弃了多少数据，然后将字节容器的读指针移到写指针，意味着丢弃这一部分数据，设置成员变量`discarding`为true表示当前处于丢弃模式。如果设置了`failFast`，那么直接抛出异常，默认情况下`failFast`为false，即安静得丢弃数据



### discarding模式

如果解包的时候处在discarding模式，也会有两种情况发生

#### discarding模式下找到行分隔符

在discarding模式下，如果找到分隔符，那可以将分隔符之前的都丢弃掉

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final int length = discardedBytes + eol - buffer.readerIndex();
final int delimLength = buffer.getByte(eol) == '\r'? 2 : 1;
buffer.readerIndex(eol + delimLength);
discardedBytes = 0;
discarding = false;
if (!failFast) {
    fail(ctx, length);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

计算出分隔符的长度之后，直接把分隔符之前的数据全部丢弃，当然丢弃的字符也包括分隔符，经过这么一次丢弃，后面就有可能是正常的数据包，下一次解包的时候就会进入正常的解包流程

#### discarding模式下未找到行分隔符

这种情况比较简单，因为当前还在丢弃模式，没有找到行分隔符意味着当前一个完整的数据包还没丢弃完，当前读取的数据是丢弃的一部分，所以直接丢弃

```
discardedBytes += buffer.readableBytes();
buffer.readerIndex(buffer.writerIndex());
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11445297.html#_labelTop)

## 特定分隔符拆包

这个类叫做 `DelimiterBasedFrameDecoder`，可以传递给TA一个分隔符列表，数据包会按照分隔符列表进行拆分，读者可以完全根据行拆包器的思路去分析这个`DelimiterBasedFrameDecoder`

# [Netty源码分析 （十一）----- 拆包器之LengthFieldBasedFrameDecoder](https://www.cnblogs.com/java-chen-hao/p/11571229.html)

**正文**

本篇文章主要是介绍使用LengthFieldBasedFrameDecoder解码器自定义协议。通常，协议的格式如下:

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190902155412644-364015440.png)

LengthFieldBasedFrameDecoder是netty解决拆包粘包问题的一个重要的类，主要结构就是header+body结构。我们只需要传入正确的参数就可以发送和接收正确的数据，那么重点就在于这几个参数的意义。下面我们就具体了解一下这几个参数的意义。先来看一下LengthFieldBasedFrameDecoder主要的构造方法：

```
public LengthFieldBasedFrameDecoder(
            int maxFrameLength,
            int lengthFieldOffset, int lengthFieldLength,
            int lengthAdjustment, int initialBytesToStrip)
```

那么这几个重要的参数如下：

- maxFrameLength：最大帧长度。也就是可以接收的数据的最大长度。如果超过，此次数据会被丢弃。
- lengthFieldOffset：长度域偏移。就是说数据开始的几个字节可能不是表示数据长度，需要后移几个字节才是长度域。
- lengthFieldLength：长度域字节数。用几个字节来表示数据长度。
- lengthAdjustment：数据长度修正。因为长度域指定的长度可以使header+body的整个长度，也可以只是body的长度。如果表示header+body的整个长度，那么我们需要修正数据长度。
- initialBytesToStrip：跳过的字节数。如果你需要接收header+body的所有数据，此值就是0，如果你只想接收body数据，那么需要跳过header所占用的字节数。

下面我们根据几个例子的使用来具体说明这几个参数的使用。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11571229.html#_labelTop)

## LengthFieldBasedFrameDecoder 的用法



### 需求1

长度域为2个字节，我们要求发送和接收的数据如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
     发送的数据 (14 bytes)          接收到数据 (14 bytes)
+--------+----------------+      +--------+----------------+
| Length | Actual Content |----->| Length | Actual Content |
|  12    | "HELLO, WORLD" |      |   12   | "HELLO, WORLD" |
+--------+----------------+      +--------+----------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

留心的你肯定发现了，长度域只是实际内容的长度，不包括长度域的长度。下面是参数的值：

- lengthFieldOffset=0：开始的2个字节就是长度域，所以不需要长度域偏移。
- lengthFieldLength=2：长度域2个字节。
- lengthAdjustment=0：数据长度修正为0，因为长度域只包含数据的长度，所以不需要修正。
- initialBytesToStrip=0：发送和接收的数据完全一致，所以不需要跳过任何字节。



### 需求2

长度域为2个字节，我们要求发送和接收的数据如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
   发送的数据 (14 bytes)        接收到数据 (12 bytes)
+--------+----------------+      +----------------+
| Length | Actual Content |----->| Actual Content |
|  12    | "HELLO, WORLD" |      | "HELLO, WORLD" |
+--------+----------------+      +----------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

参数值如下：

- lengthFieldOffset=0：开始的2个字节就是长度域，所以不需要长度域偏移。
- lengthFieldLength=2：长度域2个字节。
- lengthAdjustment=0：数据长度修正为0，因为长度域只包含数据的长度，所以不需要修正。
- initialBytesToStrip=2：我们发现接收的数据没有长度域的数据，所以要跳过长度域的2个字节。



### 需求3

长度域为2个字节，我们要求发送和接收的数据如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
+--------+----------------+      +--------+----------------+
| Length | Actual Content |----->| Length | Actual Content |
| 14     | "HELLO, WORLD" |      |  14    | "HELLO, WORLD" |
+--------+----------------+      +--------+----------------+  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

留心的你肯定又发现了，长度域表示的长度是总长度 也就是header+body的总长度。参数如下：

- lengthFieldOffset=0：开始的2个字节就是长度域，所以不需要长度域偏移。
- lengthFieldLength=2：长度域2个字节。
- lengthAdjustment=-2：因为长度域为总长度，所以我们需要修正数据长度，也就是减去2。
- initialBytesToStrip=0：我们发现接收的数据没有长度域的数据，所以要跳过长度域的2个字节。



### 需求4

长度域为2个字节，我们要求发送和接收的数据如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
   BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
+----------+----------+----------------+      +----------+----------+----------------+
| meta     |  Length  | Actual Content |----->| meta | Length | Actual Content |
|  0xCAFE  | 12       | "HELLO, WORLD" |      |  0xCAFE  | 12       | "HELLO, WORLD" |
+----------+----------+----------------+      +----------+----------+----------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们发现，数据的结构有点变化，变成了 meta+header+body的结构。meta一般表示元数据，魔数等。我们定义这里meta有三个字节。参数如下：

- lengthFieldOffset=3：开始的3个字节是meta，然后才是长度域，所以长度域偏移为3。
- lengthFieldLength=2：长度域2个字节。
- lengthAdjustment=0：长度域指定的长度位数据长度，所以数据长度不需要修正。
- initialBytesToStrip=0：发送和接收数据相同，不需要跳过数据。



### 需求5

长度域为2个字节，我们要求发送和接收的数据如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
+----------+----------+----------------+      +----------+----------+----------------+
|  Length  | meta     | Actual Content |----->| Length | meta | Actual Content |
|   12     |  0xCAFE  | "HELLO, WORLD" |      |    12    |  0xCAFE  | "HELLO, WORLD" |
+----------+----------+----------------+      +----------+----------+----------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们发现，数据的结构有点变化，变成了 header+meta+body的结构。meta一般表示元数据，魔数等。我们定义这里meta有三个字节。参数如下：

- lengthFieldOffset=0：开始的2个字节就是长度域，所以不需要长度域偏移。
- lengthFieldLength=2：长度域2个字节。
- lengthAdjustment=3：我们需要把meta+body当做body处理，所以数据长度需要加3。
- initialBytesToStrip=0：发送和接收数据相同，不需要跳过数据。



### 需求6

长度域为2个字节，我们要求发送和接收的数据如下所示：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    BEFORE DECODE (16 bytes)                    AFTER DECODE (13 bytes)
+------+--------+------+----------------+      +------+----------------+
| HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
| 0xCA | 0x000C | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
+------+--------+------+----------------+      +------+----------------+
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们发现，数据的结构有点变化，变成了 hdr1+header+hdr2+body的结构。我们定义这里hdr1和hdr2都只有1个字节。参数如下：

- lengthFieldOffset=1：开始的1个字节是长度域，所以需要设置长度域偏移为1。
- lengthFieldLength=2：长度域2个字节。
- lengthAdjustment=1：我们需要把hdr2+body当做body处理，所以数据长度需要加1。
- initialBytesToStrip=3：接收数据不包括hdr1和长度域相同，所以需要跳过3个字节。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11571229.html#_labelTop)

## LengthFieldBasedFrameDecoder 源码剖析



### 实现拆包抽象

在前面的文章中我们知道，具体的拆包协议只需要实现

```
void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) 
```

其中 in 表示目前为止还未拆的数据，拆完之后的包添加到 out这个list中即可实现包向下传递，第一层实现比较简单

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
protected final void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
    Object decoded = decode(ctx, in);
    if (decoded != null) {
        out.add(decoded);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

重载的protected函数decode做真正的拆包动作

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected Object decode(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
    if (this.discardingTooLongFrame) {
        long bytesToDiscard = this.bytesToDiscard;
        int localBytesToDiscard = (int)Math.min(bytesToDiscard, (long)in.readableBytes());
        in.skipBytes(localBytesToDiscard);
        bytesToDiscard -= (long)localBytesToDiscard;
        this.bytesToDiscard = bytesToDiscard;
        this.failIfNecessary(false);
    }

    // 如果当前可读字节还未达到长度长度域的偏移，那说明肯定是读不到长度域的，直接不读
    if (in.readableBytes() < this.lengthFieldEndOffset) {
        return null;
    } else {
        // 拿到长度域的实际字节偏移，就是长度域的开始下标
        // 这里就是需求4，开始的几个字节并不是长度域
        int actualLengthFieldOffset = in.readerIndex() + this.lengthFieldOffset;
        // 拿到实际的未调整过的包长度
        // 就是读取长度域的十进制值，最原始传过来的包的长度
        long frameLength = this.getUnadjustedFrameLength(in, actualLengthFieldOffset, this.lengthFieldLength, this.byteOrder);
        // 如果拿到的长度为负数，直接跳过长度域并抛出异常
        if (frameLength < 0L) {
            in.skipBytes(this.lengthFieldEndOffset);
            throw new CorruptedFrameException("negative pre-adjustment length field: " + frameLength);
        } else {
            // 调整包的长度
            frameLength += (long)(this.lengthAdjustment + this.lengthFieldEndOffset);
            // 整个数据包的长度还没有长度域长，直接抛出异常
            if (frameLength < (long)this.lengthFieldEndOffset) {
                in.skipBytes(this.lengthFieldEndOffset);
                throw new CorruptedFrameException("Adjusted frame length (" + frameLength + ") is less " + "than lengthFieldEndOffset: " + this.lengthFieldEndOffset);
            // 数据包长度超出最大包长度，进入丢弃模式
            } else if (frameLength > (long)this.maxFrameLength) {
                long discard = frameLength - (long)in.readableBytes();
                this.tooLongFrameLength = frameLength;
                if (discard < 0L) {
                    in.skipBytes((int)frameLength);
                } else {
                    this.discardingTooLongFrame = true;
                    this.bytesToDiscard = discard;
                    in.skipBytes(in.readableBytes());
                }

                this.failIfNecessary(true);
                return null;
            } else {
                int frameLengthInt = (int)frameLength;
                //当前可读的字节数小于包中的length，什么都不做，等待下一次解码
                if (in.readableBytes() < frameLengthInt) {
                    return null;
                //跳过的字节不能大于数据包的长度，否则就抛出 CorruptedFrameException 的异常
                } else if (this.initialBytesToStrip > frameLengthInt) {
                    in.skipBytes(frameLengthInt);
                    throw new CorruptedFrameException("Adjusted frame length (" + frameLength + ") is less " + "than initialBytesToStrip: " + this.initialBytesToStrip);
                } else {
                    //根据initialBytesToStrip的设置来跳过某些字节
                    in.skipBytes(this.initialBytesToStrip);
                    //拿到当前累积数据的读指针
                    int readerIndex = in.readerIndex();
                    //拿到待抽取数据包的实际长度
                    int actualFrameLength = frameLengthInt - this.initialBytesToStrip;
                    //进行抽取
                    ByteBuf frame = this.extractFrame(ctx, in, readerIndex, actualFrameLength);
                    //移动读指针
                    in.readerIndex(readerIndex + actualFrameLength);
                    return frame;
                }
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面分几个部分来分析一下这个重量级函数



### 获取frame长度

#### 获取需要待拆包的包大小

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 拿到长度域的实际字节偏移，就是长度域的开始下标
// 这里就是需求4，开始的几个字节并不是长度域
int actualLengthFieldOffset = in.readerIndex() + this.lengthFieldOffset;
// 拿到实际的未调整过的包长度
// 就是读取长度域的十进制值，最原始传过来的包的长度
long frameLength = this.getUnadjustedFrameLength(in, actualLengthFieldOffset, this.lengthFieldLength, this.byteOrder);
// 调整包的长度
frameLength += (long)(this.lengthAdjustment + this.lengthFieldEndOffset);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面这一段内容有个扩展点 getUnadjustedFrameLength，如果你的长度域代表的值表达的含义不是正常的int,short等基本类型，你可以重写这个函数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected long getUnadjustedFrameLength(ByteBuf buf, int offset, int length, ByteOrder order) {
    buf = buf.order(order);
    long frameLength;
    switch (length) {
    case 1:
        frameLength = buf.getUnsignedByte(offset);
        break;
    case 2:
        frameLength = buf.getUnsignedShort(offset);
        break;
    case 3:
        frameLength = buf.getUnsignedMedium(offset);
        break;
    case 4:
        frameLength = buf.getUnsignedInt(offset);
        break;
    case 8:
        frameLength = buf.getLong(offset);
        break;
    default:
        throw new DecoderException(
                "unsupported lengthFieldLength: " + lengthFieldLength + " (expected: 1, 2, 3, 4, or 8)");
    }
    return frameLength;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 跳过指定字节长度

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
int frameLengthInt = (int)frameLength;
//当前可读的字节数小于包中的length，什么都不做，等待下一次解码
if (in.readableBytes() < frameLengthInt) {
    return null;
//跳过的字节不能大于数据包的长度，否则就抛出 CorruptedFrameException 的异常
} else if (this.initialBytesToStrip > frameLengthInt) {
    in.skipBytes(frameLengthInt);
    throw new CorruptedFrameException("Adjusted frame length (" + frameLength + ") is less " + "than initialBytesToStrip: " + this.initialBytesToStrip);
}
//根据initialBytesToStrip的设置来跳过某些字节
in.skipBytes(this.initialBytesToStrip);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先验证当前是否已经读到足够的字节，如果读到了，在下一步抽取一个完整的数据包之前，需要根据initialBytesToStrip的设置来跳过某些字节(见文章开篇)，当然，跳过的字节不能大于数据包的长度，否则就抛出 CorruptedFrameException 的异常



### 抽取frame

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//根据initialBytesToStrip的设置来跳过某些字节
in.skipBytes(this.initialBytesToStrip);
//拿到当前累积数据的读指针
int readerIndex = in.readerIndex();
//拿到待抽取数据包的实际长度
int actualFrameLength = frameLengthInt - this.initialBytesToStrip;
//进行抽取
ByteBuf frame = this.extractFrame(ctx, in, readerIndex, actualFrameLength);
//移动读指针
in.readerIndex(readerIndex + actualFrameLength);
return frame;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

到了最后抽取数据包其实就很简单了，拿到当前累积数据的读指针，然后拿到待抽取数据包的实际长度进行抽取，抽取之后，移动读指针

```
protected ByteBuf extractFrame(ChannelHandlerContext ctx, ByteBuf buffer, int index, int length) {
    return buffer.retainedSlice(index, length);
}
```

抽取的过程是简单的调用了一下 ByteBuf 的retainedSliceapi，该api无内存copy开销

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11571229.html#_labelTop)

## 自定义解码器



### 协议实体的定义

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyProtocolBean {
    //类型  系统编号 0xA 表示A系统，0xB 表示B系统
    private byte type;

    //信息标志  0xA 表示心跳包    0xB 表示超时包  0xC 业务信息包
    private byte flag;

    //内容长度
    private int length;

    //内容
    private String content;

    //省略get/set
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 服务器端

#### 服务端的实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Server {

    private static final int MAX_FRAME_LENGTH = 1024 * 1024;  //最大长度
    private static final int LENGTH_FIELD_LENGTH = 4;  //长度字段所占的字节数
    private static final int LENGTH_FIELD_OFFSET = 2;  //长度偏移
    private static final int LENGTH_ADJUSTMENT = 0;
    private static final int INITIAL_BYTES_TO_STRIP = 0;

    private int port;

    public Server(int port) {
        this.port = port;
    }

    public void start(){
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap sbs = new ServerBootstrap().group(bossGroup,workerGroup).channel(NioServerSocketChannel.class).localAddress(new InetSocketAddress(port))
                    .childHandler(new ChannelInitializer<SocketChannel>() {

                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new MyProtocolDecoder(MAX_FRAME_LENGTH,LENGTH_FIELD_OFFSET,LENGTH_FIELD_LENGTH,LENGTH_ADJUSTMENT,INITIAL_BYTES_TO_STRIP,false));
                            ch.pipeline().addLast(new ServerHandler());
                        };

                    }).option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);
            // 绑定端口，开始接收进来的连接
            ChannelFuture future = sbs.bind(port).sync();

            System.out.println("Server start listen at " + port );
            future.channel().closeFuture().sync();
        } catch (Exception e) {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        } else {
            port = 8080;
        }
        new Server(port).start();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### `自定义解码器MyProtocolDecoder`

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyProtocolDecoder extends LengthFieldBasedFrameDecoder {

    private static final int HEADER_SIZE = 6;

    /**
     * @param maxFrameLength  帧的最大长度
     * @param lengthFieldOffset length字段偏移的地址
     * @param lengthFieldLength length字段所占的字节长
     * @param lengthAdjustment 修改帧数据长度字段中定义的值，可以为负数 因为有时候我们习惯把头部记入长度,若为负数,则说明要推后多少个字段
     * @param initialBytesToStrip 解析时候跳过多少个长度
     * @param failFast 为true，当frame长度超过maxFrameLength时立即报TooLongFrameException异常，为false，读取完整个帧再报异
     */

    public MyProtocolDecoder(int maxFrameLength, int lengthFieldOffset, int lengthFieldLength, int lengthAdjustment, int initialBytesToStrip, boolean failFast) {

        super(maxFrameLength, lengthFieldOffset, lengthFieldLength, lengthAdjustment, initialBytesToStrip, failFast);

    }

    @Override
    protected Object decode(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
        //在这里调用父类的方法,实现指得到想要的部分,我在这里全部都要,也可以只要body部分
        in = (ByteBuf) super.decode(ctx,in);  

        if(in == null){
            return null;
        }
        if(in.readableBytes()<HEADER_SIZE){
            throw new Exception("字节数不足");
        }
        //读取type字段
        byte type = in.readByte();
        //读取flag字段
        byte flag = in.readByte();
        //读取length字段
        int length = in.readInt();
        
        if(in.readableBytes()!=length){
            throw new Exception("标记的长度不符合实际长度");
        }
        //读取body
        byte []bytes = new byte[in.readableBytes()];
        in.readBytes(bytes);

        return new MyProtocolBean(type,flag,length,new String(bytes,"UTF-8"));

    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 服务端Hanlder

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class ServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        MyProtocolBean myProtocolBean = (MyProtocolBean)msg;  //直接转化成协议消息实体
        System.out.println(myProtocolBean.getContent());
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 客户端和客户端Handler

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Client {
    static final String HOST = System.getProperty("host", "127.0.0.1");
    static final int PORT = Integer.parseInt(System.getProperty("port", "8080"));
    static final int SIZE = Integer.parseInt(System.getProperty("size", "256"));

    public static void main(String[] args) throws Exception {

        // Configure the client.
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
                    .channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new MyProtocolEncoder());
                            ch.pipeline().addLast(new ClientHandler());
                        }
                    });

            ChannelFuture future = b.connect(HOST, PORT).sync();
            future.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 客户端编码器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyProtocolEncoder extends MessageToByteEncoder<MyProtocolBean> {

    @Override
    protected void encode(ChannelHandlerContext ctx, MyProtocolBean msg, ByteBuf out) throws Exception {
        if(msg == null){
            throw new Exception("msg is null");
        }
        out.writeByte(msg.getType());
        out.writeByte(msg.getFlag());
        out.writeInt(msg.getLength());
        out.writeBytes(msg.getContent().getBytes(Charset.forName("UTF-8")));
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 编码的时候,只需要按照定义的顺序依次写入到ByteBuf中.

#### 客户端Handler

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class ClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        super.channelRead(ctx, msg);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        MyProtocolBean myProtocolBean = new MyProtocolBean((byte)0xA, (byte)0xC, "Hello,Netty".length(), "Hello,Netty");
        ctx.writeAndFlush(myProtocolBean);

    }
}
```

# [Netty源码分析 （十二）----- 心跳服务之 IdleStateHandler 源码分析](https://www.cnblogs.com/java-chen-hao/p/11453198.html)

**正文**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453198.html#_labelTop)

## 什么是心跳机制？

心跳说的是在客户端和服务端在互相建立ESTABLISH状态的时候，如何通过发送一个最简单的包来保持连接的存活，还有监控另一边服务的可用性等。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453198.html#_labelTop)

## 心跳包的作用

- 保活
  Q：为什么说心跳机制能保持连接的存活，它是集群中或长连接中最为有效避免网络中断的一个重要的保障措施？
  A：之所以说是“避免网络中断的一个重要保障措施”，原因是：我们得知公网IP是一个宝贵的资源，一旦某一连接长时间的占用并且不发数据，这怎能对得起网络给此连接分配公网IP，这简直是对网络资源最大的浪费，所以基本上所有的NAT路由器都会定时的清除那些长时间没有数据传输的映射表项。一是回收IP资源，二是释放NAT路由器本身内存的资源，这样问题就来了，连接被从中间断开了，双发还都不晓得对方已经连通不了了，还会继续发数据，这样会有两个结果：a) 发方会收到NAT路由器的RST包，导致发方知道连接已中断；b) 发方没有收到任何NAT的回执，NAT只是简单的drop相应的数据包
  通常我们测试得出的是第二种情况会多些，就是客户端是不知道自己应经连接断开了，所以这时候心跳就可以和NAT建立关联了，只要我们在NAT认为合理连接的时间内发送心跳数据包，这样NAT会继续keep连接的IP映射表项不被移除，达到了连接不会被中断的目的。
- 检测另一端服务是否可用
  TCP的断开可能有时候是不能瞬时探知的，甚至是不能探知的，也可能有很长时间的延迟，如果前端没有正常的断开TCP连接，四次握手没有发起，服务端无从得知客户端的掉线，这个时候我们就需要心跳包来检测另一端服务是否还存活可用。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453198.html#_labelTop)

## 基于TCP的keepalive机制实现

基于TCP的keepalive机制，由具体的TCP协议栈来实现长连接的维持。如在netty中可以在创建channel的时候，指定SO_KEEPALIVE参数来实现：

![img](https://img2018.cnblogs.com/blog/1168971/201909/1168971-20190903144056718-721010082.png)

存在的问题：Netty只能控制SO_KEEPALIVE这个参数，其他参数，则需要从系统的sysctl中读取，其中比较关键的是tcp_keepalive_time，发送心跳包检测的时间间隔，默认为7200s，即空闲后，每2小时检测一次。如果客户端在这2小时内断开了，那么服务端也要维护这个连接2小时，浪费服务端资源；另外就是对于需要实时传输数据的场景，客户端断开了，服务端也要2小时后才能发现。服务端发送心跳检测，具体可能出现的情况如下：
（1）连接正常：客户端仍然存在，网络连接状况良好。此时客户端会返回一个 ACK 。 服务端接收到ACK后重置计时器，在2小时后再发送探测。如果2小时内连接上有数据传输，那么在该时间基础上向后推延2个小时；
（2）连接断开：客户端异常关闭，或是网络断开。在这两种情况下，客户端都不会响应。服务器没有收到对其发出探测的响应，并且在一定时间（系统默认为 1000 ms ）后重复发送 keep-alive packet ，并且重复发送一定次数。
（3）客户端曾经崩溃，但已经重启：这种情况下，服务器将会收到对其存活探测的响应，但该响应是一个复位，从而引起服务器对连接的终止。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/11453198.html#_labelTop)

## 基于Netty的IdleStateHandler实现



### 什么是 IdleStateHandler

当连接的空闲时间（读或者写）太长时，将会触发一个 IdleStateEvent 事件。然后，你可以通过你的 ChannelInboundHandler 中重写 userEventTrigged 方法来处理该事件。



### 如何使用？

IdleStateHandler 既是出站处理器也是入站处理器，继承了 ChannelDuplexHandler 。通常在 initChannel 方法中将 IdleStateHandler 添加到 pipeline 中。然后在自己的 handler 中重写 userEventTriggered 方法，当发生空闲事件（读或者写），就会触发这个方法，并传入具体事件。
这时，你可以通过 Context 对象尝试向目标 Socekt 写入数据，并设置一个 监听器，如果发送失败就关闭 Socket （Netty 准备了一个 `ChannelFutureListener.CLOSE_ON_FAILURE` 监听器用来实现关闭 Socket 逻辑）。
这样，就实现了一个简单的心跳服务。



### 源码分析

#### 构造方法

该类有 3 个构造方法，主要对一下 4 个属性赋值：

```
private final boolean observeOutput;// 是否考虑出站时较慢的情况。默认值是false（不考虑）。
private final long readerIdleTimeNanos; // 读事件空闲时间，0 则禁用事件
private final long writerIdleTimeNanos;// 写事件空闲时间，0 则禁用事件
private final long allIdleTimeNanos; //读或写空闲时间，0 则禁用事件
```

可以分别控制读，写，读写超时的时间，单位为秒，如果是0表示不检测，所以如果全是0，则相当于没添加这个IdleStateHandler，连接是个普通的短连接。

#### handlerAdded 方法

IdleStateHandler是在创建IdleStateHandler实例并添加到ChannelPipeline时添加定时任务来进行定时检测的，具体在initialize(ctx)方法实现；同时在从ChannelPipeline移除或Channel关闭时，移除这个定时检测，具体在destroy()实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
    if (ctx.channel().isActive() && ctx.channel().isRegistered()) {
        this.initialize(ctx);
    }

}

public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
    this.destroy();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### initialize

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void initialize(ChannelHandlerContext ctx) {
    switch (state) {
    case 1:
    case 2:
        return;
    }
    state = 1;
    initOutputChanged(ctx);

    lastReadTime = lastWriteTime = ticksInNanos();
    if (readerIdleTimeNanos > 0) {
      // 这里的 schedule 方法会调用 eventLoop 的 schedule 方法，将定时任务添加进队列中
        readerIdleTimeout = schedule(ctx, new ReaderIdleTimeoutTask(ctx),
                readerIdleTimeNanos, TimeUnit.NANOSECONDS);
    }
    if (writerIdleTimeNanos > 0) {
        writerIdleTimeout = schedule(ctx, new WriterIdleTimeoutTask(ctx),
                writerIdleTimeNanos, TimeUnit.NANOSECONDS);
    }
    if (allIdleTimeNanos > 0) {
        allIdleTimeout = schedule(ctx, new AllIdleTimeoutTask(ctx),
                allIdleTimeNanos, TimeUnit.NANOSECONDS);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

只要给定的参数大于0，就创建一个定时任务，每个事件都创建。同时，将 state 状态设置为 1，防止重复初始化。调用 initOutputChanged 方法，初始化 “监控出站数据属性”，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private void initOutputChanged(ChannelHandlerContext ctx) {
    if (observeOutput) {
        Channel channel = ctx.channel();
        Unsafe unsafe = channel.unsafe();
        ChannelOutboundBuffer buf = unsafe.outboundBuffer();
        // 记录了出站缓冲区相关的数据，buf 对象的 hash 码，和 buf 的剩余缓冲字节数
        if (buf != null) {
            lastMessageHashCode = System.identityHashCode(buf.current());
            lastPendingWriteBytes = buf.totalPendingWriteBytes();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 读事件的 run 方法

代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected void run(ChannelHandlerContext ctx) {
    long nextDelay = readerIdleTimeNanos;
    if (!reading) {
        nextDelay -= ticksInNanos() - lastReadTime;
    }

    if (nextDelay <= 0) {
        // Reader is idle - set a new timeout and notify the callback.
        // 用于取消任务 promise
        readerIdleTimeout = schedule(ctx, this, readerIdleTimeNanos, TimeUnit.NANOSECONDS);

        boolean first = firstReaderIdleEvent;
        firstReaderIdleEvent = false;

        try {
            // 再次提交任务
            IdleStateEvent event = newIdleStateEvent(IdleState.READER_IDLE, first);
            // 触发用户 handler use
            channelIdle(ctx, event);
        } catch (Throwable t) {
            ctx.fireExceptionCaught(t);
        }
    } else {
        // Read occurred before the timeout - set a new timeout with shorter delay.
        readerIdleTimeout = schedule(ctx, this, nextDelay, TimeUnit.NANOSECONDS);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

nextDelay的初始化值为超时秒数readerIdleTimeNanos，如果检测的时候没有正在读，且计算多久没读了：nextDelay -= 当前时间 - 上次读取时间，如果小于0，说明左边的readerIdleTimeNanos小于空闲时间（当前时间 - 上次读取时间）了，则超时了
则创建IdleStateEvent事件，IdleState枚举值为READER_IDLE，然后调用channelIdle方法分发给下一个ChannelInboundHandler，通常由用户自定义一个ChannelInboundHandler来捕获并处理

总的来说，每次读取操作都会记录一个时间，定时任务时间到了，会计算当前时间和最后一次读的时间的间隔，如果间隔超过了设置的时间，就触发 UserEventTriggered 方法。就是这么简单。

#### 写事件的 run 方法

写任务的逻辑基本和读任务的逻辑一样，唯一不同的就是有一个针对 出站较慢数据的判断。

```
if (hasOutputChanged(ctx, first)) {
   return;
}
```

如果这个方法返回 true，就不执行触发事件操作了，即使时间到了。看看该方法实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private boolean hasOutputChanged(ChannelHandlerContext ctx, boolean first) {
    if (observeOutput) {
        // 如果最后一次写的时间和上一次记录的时间不一样，说明写操作进行过了，则更新此值
        if (lastChangeCheckTimeStamp != lastWriteTime) {
            lastChangeCheckTimeStamp = lastWriteTime;
            // 但如果，在这个方法的调用间隙修改的，就仍然不触发事件
            if (!first) { // #firstWriterIdleEvent or #firstAllIdleEvent
                return true;
            }
        }
        Channel channel = ctx.channel();
        Unsafe unsafe = channel.unsafe();
        ChannelOutboundBuffer buf = unsafe.outboundBuffer();
        // 如果出站区有数据
        if (buf != null) {
            // 拿到出站缓冲区的 对象 hashcode
            int messageHashCode = System.identityHashCode(buf.current());
            // 拿到这个 缓冲区的 所有字节
            long pendingWriteBytes = buf.totalPendingWriteBytes();
            // 如果和之前的不相等，或者字节数不同，说明，输出有变化，将 "最后一个缓冲区引用" 和 “剩余字节数” 刷新
            if (messageHashCode != lastMessageHashCode || pendingWriteBytes != lastPendingWriteBytes) {
                lastMessageHashCode = messageHashCode;
                lastPendingWriteBytes = pendingWriteBytes;
                // 如果写操作没有进行过，则任务写的慢，不触发空闲事件
                if (!first) {
                    return true;
                }
            }
        }
    }
    return false;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. 如果用户没有设置了需要观察出站情况。就返回 false，继续执行事件。
2. 反之，继续向下， 如果最后一次写的时间和上一次记录的时间不一样，说明写操作刚刚做过了，则更新此值，但仍然需要判断这个 first 的值，如果这个值还是 false，说明在这个写事件是在两个方法调用间隙完成的 / 或者是第一次访问这个方法，就仍然不触发事件。
3. 如果不满足上面的条件，就取出缓冲区对象，如果缓冲区没对象了，说明没有发生写的很慢的事件，就触发空闲事件。反之，记录当前缓冲区对象的 hashcode 和 剩余字节数，再和之前的比较，如果任意一个不相等，说明数据在变化，或者说数据在慢慢的写出去。那么就更新这两个值，留在下一次判断。
4. 继续判断 first ，如果是 fasle，说明这是第二次调用，就不用触发空闲事件了。

#### 所有事件的 run 方法

这个类叫做 AllIdleTimeoutTask ，表示这个监控着所有的事件。当读写事件发生时，都会记录。代码逻辑和写事件的的基本一致，除了这里：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
long nextDelay = allIdleTimeNanos;
if (!reading) {
   // 当前时间减去 最后一次写或读 的时间 ，若大于0，说明超时了
   nextDelay -= ticksInNanos() - Math.max(lastReadTime, lastWriteTime);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里的时间计算是取读写事件中的最大值来的。然后像写事件一样，判断是否发生了写的慢的情况。最后调用 ctx.fireUserEventTriggered(evt) 方法。

通常这个使用的是最多的。构造方法一般是：

```
pipeline.addLast(new IdleStateHandler(0, 0, 30, TimeUnit.SECONDS));
```

读写都是 0 表示禁用，30 表示 30 秒内没有任务读写事件发生，就触发事件。注意，当不是 0 的时候，这三个任务会重叠。