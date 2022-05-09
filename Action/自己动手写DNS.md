# 自己动手实现DNS协议

## 1. 主要内容

本文的主要内容：

1. 介绍DNS是干什么的；
2. 介绍DNS是如何工作的；
3. 介绍DNS请求与响应的消息格式；
4. 编程实现一个简单的DNS服务器；

## 2. DNS是啥

关于DNS是啥，想必学过计算机网络的应该都知道，它是Domain Name System的简写，中文翻译过来就是域名系统，是用来将主机名转换为ip的。事实上，除了进行主机名到IP地址的转换外，DNS通常还提供主机名到以下几项的转换服务：

1. 主机命名（host aloasing）。有着复杂**规范主机名（canonical hostname）**的主机可能有一个或多个别名，通常规范主机名较复杂，而别名让人更容易记忆。应用程序可以调用DNS来获得主机别名对应的规范主机名，以及主机的ip地址。
2. 邮件服务器别名（mail server aliasing）。DNS也能完成邮件服务器别名到其规范主机名以及ip地址的转换。
3. 负载均衡（load distribution）。DNS可用于冗余的服务器之间进行负载均衡。一个繁忙的站点，如abc.com，可能被冗余部署在多台具有不同ip的服务器上。在该情况下，在DNS数据库中，该主机名可能对应着一个ip集合，但应用程序调用DNS来获取该主机名对应的ip时，DNS通过某种算法从该主机名对应的ip集合中，挑选出某一ip进行响应。

问：为什么会有DNS，或者说为什么要弄出两种方式（主机名和IP地址）来标识一台主机呢？

答：这是因为主机名便于人的记忆，而IP地址便于计算机网络设备的处理，于是需要DNS来做前者到后者的转换。

## 3. DNS工作原理

DNS实际上是由一个分层的DNS服务器实现的**分布式数据库**和一个让主机能够查询分布式数据库的**应用层协议**组成。因此，要了解DNS的工作原理，需要从以上两个方便入手。

### 3.1 DNS的分布式架构

先来了解DNS的分布式架构。

DNS服务器根据域名命名空间（domian name space）组织成如下图所示的树形结构（当然，只给出部分DNS服务器，只为显示出DNS服务器的层次结构）：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/%E9%80%89%E5%8C%BA_010.jpg)

在图中，根节点代表的是根DNS服务器，因特网上共有13台，编号从A到M；根DNS服务器之下的一层被称为顶级DNS服务器；再往下一层被称为权威DNS服务器。

当一个应用要通过DNS来查询某个主机名，比如www.google.com的ip时，粗略地说，查询过程是这样的：它先与根服务器之一联系，根服务器根据顶级域名com，会响应命名空间为com的顶级域服务器的ip；于是该应用接着向com顶级域服务器发出请求，com顶级域服务器会响应命名空间为google.com的权威DNS服务器的ip地址；最后该应用将请求命名空间为google.com的权威DNS服务器，该权威DNS服务器会响应主机名为www.google.com的ip。

实际上，除了上图层次结构中所展示的DNS外，还有一类与我们接触更为密切的DNS服务器，它们是本地DNS服务器，我们经常在电脑上配置的DNS服务器通常就是此类。它们一般由某公司，某大学，或某居民区提供，比如Google提供的DNS服务器8.8.8.8；比如常被人诟病的114.114.114.114等。

加入了本地DNS的查询过程跟之前的查询过程基本上是一致的，查询流程如下图所示：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/%E9%80%89%E5%8C%BA_011.jpg)

在实际工作中，DNS服务器是带缓存的。即DNS服务器在每次收到DNS请求时，都会先查询自身数据库包括缓存中有无要查询的主机名的ip，若有且没有过期，则直接响应该ip，否则才会按上图流程进行查询；而服务器在每次收到响应信息后，都会将响应信息缓存起来；

### 3.2 DNS应用层协议

#### 3.2.1 DNS资源记录

在介绍DNS层协议之前，先了解一下DNS服务器存储的**资源记录（Resource Records，RRs）**，一条资源记录(RR)记载着一个映射关系。每条RR通常包含如下表所示的一些信息：

|   字段   |       含义        |
| :------: | :---------------: |
|   NAME   |       名字        |
|   TYPE   |       类型        |
|  CLASS   |        类         |
|   TTL    |     生存时间      |
| RDLENGTH | RDATA所占的字节数 |
|  RDATA   |       数据        |

NAME和RDATA表示的含义根据TYPE的取值不同而不同，常见的：

1. 若TYPE=A，则name是主机名，value是其对应的ip；
2. 若TYPE=NS，则name是一个域，value是一个权威DNS服务器的主机名。该记录表示name域的域名解析将由value主机名对应的DNS服务器来做；
3. 若TYPE=CNAME，则value是别名为name的主机对应的规范主机名；
4. 若TYPE=MX，则value是别名为name的邮件服务器的规范主机名；
5. ……

TYPE实际上还有其他类型，所有可能的type及其约定的数值表示如下：

| TYPE  | value |                 meaning                  |
| :---: | :---: | :--------------------------------------: |
|   A   |   1   |              a host address              |
|  NS   |   2   |       an authoritative name server       |
|  MD   |   3   |  a mail destination (Obsolete - use MX)  |
|  MF   |   4   |   a mail forwarder (Obsolete - use MX)   |
| CNAME |   5   |     the canonical name for an alias      |
|  SOA  |   6   |  marks the start of a zone of authority  |
|  MB   |   7   |   a mailbox domain name (EXPERIMENTAL)   |
|  MG   |   8   |    a mail group member (EXPERIMENTAL)    |
|  MR   |   9   | a mail rename domain name (EXPERIMENTAL) |
| NULL  |  10   |         a null RR (EXPERIMENTAL)         |
|  WKS  |  11   |     a well known service description     |
|  PTR  |  12   |          a domain name pointer           |
| HINFO |  13   |             host information             |
| MINFO |  14   |     mailbox or mail list information     |
|  MX   |  15   |              mail exchange               |
|  TXT  |  16   |               text strings               |

#### 3.2.2 整体及Header部分

下面介绍第二个方面，DNS协议。

DNS请求与响应的格式是一致的，其整体分为Header、Question、Answer、Authority、Additional5部分，如下图所示：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/2017-04-15%2003-02-39%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

Header部分是一定有的，长度固定为12个字节；其余4部分可能有也可能没有，并且长度也不一定，这个在Header部分中有指明。Header的结构如下：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/2017-04-15%2003-06-31%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

下面说明一下各个字段的含义:

1. ID：占16位。该值由发出DNS请求的程序生成，DNS服务器在响应时会使用该ID，这样便于请求程序区分不同的DNS响应。
2. QR：占1位。指示该消息是请求还是响应。0表示请求；1表示响应。
3. OPCODE：占4位。指示请求的类型，有请求发起者设定，响应消息中复用该值。0表示标准查询；1表示反转查询；2表示服务器状态查询。3~15目前保留，以备将来使用。
4. AA（Authoritative Answer，权威应答）：占1位。表示响应的服务器是否是权威DNS服务器。只在响应消息中有效。
5. TC（TrunCation，截断）：占1位。指示消息是否因为传输大小限制而被截断。
6. RD（Recursion Desired，期望递归）：占1位。该值在请求消息中被设置，响应消息复用该值。如果被设置，表示希望服务器递归查询。但服务器不一定支持递归查询。
7. RA（Recursion Available，递归可用性）：占1位。该值在响应消息中被设置或被清除，以表明服务器是否支持递归查询。
8. Z：占3位。保留备用。
9. RCODE（Response code）：占4位。该值在响应消息中被设置。取值及含义如下：
   - 0：No error condition，没有错误条件；
   - 1：Format error，请求格式有误，服务器无法解析请求；
   - 2：Server failure，服务器出错。
   - 3：Name Error，只在权威DNS服务器的响应中有意义，表示请求中的域名不存在。
   - 4：Not Implemented，服务器不支持该请求类型。
   - 5：Refused，服务器拒绝执行请求操作。
   - 6~15：保留备用。
10. QDCOUNT：占16位（无符号）。指明Question部分的包含的实体数量。
11. ANCOUNT：占16位（无符号）。指明Answer部分的包含的RR（Resource Record）数量。
12. NSCOUNT：占16位（无符号）。指明Authority部分的包含的RR（Resource Record）数量。
13. ARCOUNT：占16位（无符号）。指明Additional部分的包含的RR（Resource Record）数量。

#### 3.2.3 Question部分

Question部分的每一个实体的格式如下图所示：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/2017-04-15%2003-08-09%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

1. QNAME：字节数不定，以0x00作为结束符。表示查询的主机名。注意：众所周知，主机名被"."号分割成了多段标签。在QNAME中，每段标签前面加一个数字，表示接下来标签的长度。比如：api.sina.com.cn表示成QNAME时，会在"api"前面加上一个字节0x03，"sina"前面加上一个字节0x04，"com"前面加上一个字节0x03，而"cn"前面加上一个字节0x02；
2. QTYPE：占2个字节。表示RR类型，见以上RR介绍；
3. QCLASS：占2个字节。表示RR分类，见以上RR介绍。

#### 3.2.4 Answer、Authority、Additional部分

Answer、Authority、Additional部分格式一致，每部分都由若干实体组成，每个实体即为一条RR，之前有过介绍，格式如下图所示：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/2017-04-15%2003-11-32%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

1. NAME：长度不定，可能是真正的数据，也有可能是指针（其值表示的是真正的数据在整个数据中的字节索引数），还有可能是二者的混合（以指针结尾）。若是真正的数据，会以0x00结尾；若是指针，指针占2个字节，第一个字节的高2位为11。
2. TYPE：占2个字节。表示RR的类型，如A、CNAME、NS等，见以上RR介绍；
3. CLASS：占2个字节。表示RR的分类，见以上RR介绍；
4. TTL：占4个字节。表示RR生命周期，即RR缓存时长，单位是秒；
5. RDLENGTH：占2个字节。指定RDATA字段的字节数；
6. RDATA：即之前介绍的value，含义与TYPE有关，见以上RR介绍。

DNS协议是工作在应用层的，运输层依赖的是UDP协议。下面尝试使用Python3.6来实现一个简单的DNS服务器。

在此之前先用[Wireshark](https://www.wireshark.org/)抓一下DNS包，验证一下上面的DNS协议的格式，也便于之后的实现。Wireshark的用法就不做介绍了，相信装好随便点点就知道怎么用了。先打开监听，添加过滤条件，然后用nslookup命令发送一个DNS包，比如我们尝试查询www.baidu.com的ip：

```shell
nslookup www.baidu.com
```

然后可以在Wireshark中看到如下图所示的请求数据包：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/%E9%80%89%E5%8C%BA_022.jpg)

响应数据如下图所示：

![img](https://blog-1251252991.cos.ap-chengdu.myqcloud.com/%E9%80%89%E5%8C%BA_023.jpg)

## 4. 实现一个简单的DNS服务器

下面用Python来实现一个非常简单的DNS服务器。

### 4.1 代理功能

首先，它应该具有最基本的“代理”功能，即我们的DNS服务器在接到DNS请求后，直接将请求转发到某DNS服务器（如114.114.114.114）上，然后再将那台DNS的响应结果返回给DNS客户端：

```python
import threading
import socket
import socketserver
 
 
class Handler(socketserver.BaseRequestHandler):
    def handle(self):
        request_data = self.request[0]
        # 将请求转发到 114 DNS
        redirect_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        redirect_socket.sendto(request_data, ('114.114.114.114', 53))
        response_data, address = redirect_socket.recvfrom(1024)
	
        # 将114响应响应给客户
        client_socket = self.request[1]
        client_socket.sendto(response_data, self.client_address)
 
 
class Server(socketserver.ThreadingMixIn, socketserver.UDPServer):
    pass
 
 
if __name__ == "__main__":
    # 一下ip需换成自己电脑的ip
    server = Server(('172.16.42.254', 53), Handler)
    with server:
        server_thread = threading.Thread(target=server.serve_forever)
        server_thread.daemon = True
        server_thread.start()
        print('The DNS server is running at 172.16.42.254...')
        server_thread.join()
```

现在我们的DNS服务器就可以进行转发工作了。运行以上程序（需root权限），然后用nsloop命令，向我们的服务器发送DNS请求，一切OK：

```shell
$ nslookup baidu.com 172.16.42.254
Server:		172.16.42.254
Address:	172.16.42.254#53
 
Non-authoritative answer:
Name:	baidu.com
Address: 123.125.114.144
Name:	baidu.com
Address: 180.149.132.47
Name:	baidu.com
Address: 111.13.101.208
Name:	baidu.com
Address: 220.181.57.217
```

### 4.2 缓存功能

如果仅仅做一下代理转发，那也太无聊了。现在我们再加上缓存功能，即它能够将其他DNS服务器的响应结果缓存起来。当收到请求时，若请求主机号在缓存中，且没有过期，则直接响应缓存结果；否则进行上一功能中的操作。这一功能的关键在于对DNS消息的解析，代码如下：

```python
class Message:
    u"""All communications inside of the domain protocol are carried in a single format called a message"""
 
    def __init__(self, header, question=None, answer=None, authority=None, additional=None):
        self.header = header
        self.question = question
        self.answer = answer
        self.authority = authority
        self.additional = additional
 
    @classmethod
    def from_bytes(cls, data):
        scanner = Scanner(data)
        # 读取header
        header = dict()
        header['ID'] = scanner.next_bytes(2)
        header['QR'] = scanner.next_bits(1)
        header['OPCODE'] = scanner.next_bits(4)
        header['AA'] = scanner.next_bits(1)
        header['TC'] = scanner.next_bits(1)
        header['RD'] = scanner.next_bits(1)
        header['RA'] = scanner.next_bits(1)
        header['Z'] = scanner.next_bits(3)
        header['RCODE'] = scanner.next_bits(4)
        header['QDCOUNT'] = scanner.next_bytes(2)
        header['ANCOUNT'] = scanner.next_bytes(2)
        header['NSCOUNT'] = scanner.next_bytes(2)
        header['ARCOUNT'] = scanner.next_bytes(2)
        print('header:', header)
        # 读取question
        questions = list()
        for _ in range(header['QDCOUNT']):
            question = dict()
            question['QNAME'] = scanner.next_bytes_until(lambda current, _: current == 0)
            scanner.next_bytes(1)  # 跳过0
            question['QTYPE'] = scanner.next_bytes(2)
            question['QCLASS'] = scanner.next_bytes(2)
            questions.append(question)
        print('questions:', questions)
        message = Message(header)
        # 读取answer、authority、additional
        rrs = list()
        for i in range(header['ANCOUNT'] + header['NSCOUNT'] + header['ARCOUNT']):
            rr = dict()
            rr['NAME'] = cls.handle_compression(scanner)
            rr['TYPE'] = scanner.next_bytes(2)
            rr['CLASS'] = scanner.next_bytes(2)
            rr['TTL'] = scanner.next_bytes(4)
            rr['RDLENGTH'] = scanner.next_bytes(2)
            # 处理data
            if rr['TYPE'] == 1:  # A记录
                r_data = scanner.next_bytes(rr['RDLENGTH'], False)
                rr['RDATA'] = reduce(lambda x, y: y if (len(x) == 0) else x + '.' + y,
                                     map(lambda num: str(num), r_data))
            elif rr['TYPE'] == 2 or rr['TYPE'] == 5:  # NS与CNAME记录
                rr['RDATA'] = cls.handle_compression(scanner, rr['RDLENGTH'])
            rrs.append(rr)
        answer, authority, additional = list(), list(), list()
        for i, rr in enumerate(rrs):
            if i < header['ANCOUNT']:
                answer.append(rr)
            elif i < header['ANCOUNT'] + header['NSCOUNT']:
                authority.append(rr)
            else:
                additional.append(rr)
        print('answer:', answer)
        print('authority:', authority)
        print('additional:', additional)
        return message
 
    @classmethod
    def handle_compression(cls, scanner, length=float("inf")):
        """
        The compression scheme allows a domain name in a message to be represented as either:
            - a pointer
            - a sequence of labels ending in a zero octet
            - a sequence of labels ending with a pointer
        """
        byte = scanner.next_bytes()
        if byte >> 6 == 3:  # a pointer
            pointer = (byte & 0x3F << 8) + scanner.next_bytes()
            return cls.handle_compression(Scanner(scanner.data, pointer))
        data = scanner.next_bytes_until(lambda current, offset: current == 0 or current >> 6 == 3 or offset > length)
        if scanner.next_bytes(move=False) == 0:  # a sequence of labels ending in a zero octet
            scanner.next_bytes()
            return data
        # a sequence of labels ending with a pointer
        result = data + '.' + cls.handle_compression(Scanner(scanner.data, *scanner.position()))
        scanner.next_bytes(2)  # 跳过2个字节的指针
        return result
```

其中用到了一个自定义的Scanner类，用来帮助我们从bytes中按字节或位读取数据，其定义如下：

```python
class Scanner:
    """scan bytes"""
    __mark_offset_byte, __mark_offset_bit = 0, 0
 
    def __init__(self, data: bytes, offset_byte=0, offset_bit=0):
        self.data = data
        self.__offset_byte = offset_byte
        self.__offset_bit = offset_bit
 
    def next_bits(self, n=1):
        if n > (len(self.data) - self.__offset_byte) * 8 - self.__offset_bit:
            raise RuntimeError('剩余数据不足{}位'.format(n))
        if n > 8 - self.__offset_bit:
            raise RuntimeError('不能跨字节读取读取位')
        result = self.data[self.__offset_byte] >> 8 - self.__offset_bit - n & (1 << n) - 1
        self.__offset_bit += n
        if self.__offset_bit == 8:
            self.__offset_bit = 0
            self.__offset_byte += 1
        return result
 
    def next_bytes(self, n=1, convert=True, move=True):
        if not self.__offset_bit == 0:
            raise RuntimeError('当前字节不完整，请先读取完当前字节的所有位')
        if n > len(self.data) - self.__offset_byte:
            raise RuntimeError('剩余数据不足{}字节'.format(n))
        result = self.data[self.__offset_byte: self.__offset_byte + n]
        if move:
            self.__offset_byte += n
        if convert:
            result = int.from_bytes(result, 'big')
        return result
 
    def next_bytes_until(self, stop, convert=True):
        if not self.__offset_bit == 0:
            raise RuntimeError('当前字节不完整，请先读取完当前字节的所有位')
        end = self.__offset_byte
        while not stop(self.data[end], end - self.__offset_byte):
            end += 1
        result = self.data[self.__offset_byte: end]
        self.__offset_byte = end
        if convert:
            if result:
                result = reduce(lambda x, y: y if (x == '.') else x + y,
                                map(lambda x: chr(x) if (31 < x < 127) else '.', result))
            else:
                result = ''
        return result
```

然后需要做的是当收到114 DNS服务器的响应消息后，将消息缓存起来：

```python
# 缓存响应结果
message = Message.from_bytes(response_data)
message.save()
```

以上save方法就是将message中包含的各条RR保存起来，可以直接用一个集合来保存，也可以保存在一些专业的缓存设施中，比如redis。需要注意的是TTL的处理，若用redis缓存，它自带了TTL功能，可以直接使用。若是自己实现的，需要在保存的时候记录当前的时间，以便取出的时候能够判断是否过期。这些应该很容易实现，但是本人比较懒，这里就不写了……

### 4.3 添加记录功能

最后，它需要具备能够读取我们自定义的记录，并将记录加入缓存。。这个也不想写了……

另外，Message类还应该有一个`to_bytes`方法，它能将一个Message对象转换为bytes对象，用于将 从缓存中取出的数据（即RR记录）转换为bytes，返回给用户。这个其实就是`from_bytes`的逆过程，但实现起来应该比`from_bytes`简单许多，因为你可以不使用指针来压缩数据，这样处理起来就没什么难度了。同样不想写了……

最后稍微做一下测试，算是做个结束：

```python
if __name__ == "__main__":
    server = Server('172.16.42.254')
    server.start()
```

使用nsloop发送DNS请求到我们自己写的服务器上，响应结果如下：

```yaml
$ nslookup api.sina.com.cn 172.16.42.254
Server:		172.16.42.254
Address:	172.16.42.254#53
 
Non-authoritative answer:
api.sina.com.cn	canonical name = common6.dpool.sina.com.cn.
Name:	common6.dpool.sina.com.cn
Address: 123.126.56.253
```

在运行的控制台中，打印出了从114 DNS返回的数据的解析结果：

```shell
$ sudo python3 dns.py 
The DNS server is running at 172.16.42.254...
header: {'ID': 25835, 'QR': 1, 'OPCODE': 0, 'AA': 0, 'TC': 0, 'RD': 1, 'RA': 1, 'Z': 0, 'RCODE': 0, 'QDCOUNT': 1, 'ANCOUNT': 2, 'NSCOUNT': 4, 'ARCOUNT': 4}
questions: [{'QNAME': 'api.sina.com.cn', 'QTYPE': 1, 'QCLASS': 1}]
answer: [{'NAME': 'api.sina.com.cn', 'TYPE': 5, 'CLASS': 1, 'TTL': 56, 'RDLENGTH': 16, 'RDATA': 'common6.dpool.sina.com.cn'}, {'NAME': 'common6.dpool.sina.com.cn', 'TYPE': 1, 'CLASS': 1, 'TTL': 34, 'RDLENGTH': 4, 'RDATA': '123.126.56.253'}]
authority: [{'NAME': 'dpool.sina.com.cn', 'TYPE': 2, 'CLASS': 1, 'TTL': 26753, 'RDLENGTH': 6, 'RDATA': 'ns1.sina.com.cn'}, {'NAME': 'dpool.sina.com.cn', 'TYPE': 2, 'CLASS': 1, 'TTL': 26753, 'RDLENGTH': 6, 'RDATA': 'ns3.sina.com.cn'}, {'NAME': 'dpool.sina.com.cn', 'TYPE': 2, 'CLASS': 1, 'TTL': 26753, 'RDLENGTH': 6, 'RDATA': 'ns2.sina.com.cn'}, {'NAME': 'dpool.sina.com.cn', 'TYPE': 2, 'CLASS': 1, 'TTL': 26753, 'RDLENGTH': 6, 'RDATA': 'ns4.sina.com.cn'}]
additional: [{'NAME': 'ns1.sina.com.cn', 'TYPE': 1, 'CLASS': 1, 'TTL': 26674, 'RDLENGTH': 4, 'RDATA': '202.106.184.166'}, {'NAME': 'ns2.sina.com.cn', 'TYPE': 1, 'CLASS': 1, 'TTL': 26652, 'RDLENGTH': 4, 'RDATA': '61.172.201.254'}, {'NAME': 'ns3.sina.com.cn', 'TYPE': 1, 'CLASS': 1, 'TTL': 26509, 'RDLENGTH': 4, 'RDATA': '123.125.29.99'}, {'NAME': 'ns4.sina.com.cn', 'TYPE': 1, 'CLASS': 1, 'TTL': 26497, 'RDLENGTH': 4, 'RDATA': '121.14.1.22'}]
```

以上完整代码，见[这里](https://github.com/derker94/dns-server/blob/master/dns.py)

## 5. 参考资料

- 计算机网络，自顶向下方法（原书第6版）/（美）库罗斯（Kurose，J.F.），（美）罗斯（Ross，K.W.）著；陈鸣译.——北京：机械工业出版社，2014.9
- [RFC 1034](https://datatracker.ietf.org/doc/html/rfc1034)
- [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035)
- [BlackHole](https://github.com/code4craft/blackhole)
- [结合Wireshark分析DNS 协议](http://blog.csdn.net/hunanchenxingyu/article/details/21488291)