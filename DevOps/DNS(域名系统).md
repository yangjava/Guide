# DNS

## 简介

DNS：Domain Name Service 域名解析服务，工作在应用层，是互联网的一项服务。它作为将域名和IP地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。DNS监听在TCP和UDP端口53。

**FQDN**：全称域名，如 `www.example.com`

- `www`是主机名
- `example.com`是域名

**实现名称到IP解析的有三种方式**：

- 分散式解决方案：/etc/hosts，不易管理
- 集中式解决方案：NIC，如果主机太多，对单一服务的压力太大
- 分布式解决方案：DNS，解决了上俩个方式的不足

**权威的DNS服务器**：记录主机名到IP的DNS服务器叫做权威的DNS服务器

**主从DNS服务器**：主服务器记录发生变化，会同步到从服务器，（主从复制），实现容错机制

- 序列号：解析库版本号，主服务器解析库变化时，其序列递增
- 刷新时间间隔：从服务器从主服务器请求同步解析的时间间隔
- 重试时间间隔：从服务器请求同步失败时，再次尝试时间间隔
- 过期时长：从服务器联系不到主服务器时，多久后停止服务

**DNS的查询类型**：

- 递归查询：服务器从根开始去找一级一级的找到目标主机，直到找到，负责到底
- 迭代查询：根服务器给了主机所在的域的IP，并没有直接给出结果

**DNS的解析类型**：正反向解析是两个不同的名称空间，是两棵不同的解析树

- 正向解析：FQDN --> IP
- 反向解析：IP ---> FQDN 如邮件服务器需要用到反向解析技术

**DNS服务器的类型**：

- 主服务器：管理和维护所负责解析的域内解析库的服务器
- 从服务器：从主服务器或从服务器“复制”（区域传输）解析库副本
- 缓存服务器

### DNS的工作原理

1. 客户端向离它最近的DNS服务器发起了查询请求，一般是由运营商提供
2. 如果代理DNS服务器有记录则直接可以返回给客户端；如果没有记录则去根DNS服务器请求，根DNS并不会存储所以的主机名对应IP的记录，它只会记录它的子域的IP，例如`.com`等后缀的域，代理DNS服务器会拿到`.com`域的DNS服务器IP
3. 然后再将请求发往`.com.`域的DNS服务器，如果还是没有找到主机，则再往它的下一级找，直到找到具体的主机，把IP返回给客户端，同时代理DNS服务器也会缓存一份到本地

一次完整的查询请求经过的流程：Client -->hosts文件 -->DNS Service Local Cache --> DNS Server (recursion) --> Server Cache --> iteration(迭代) --> 根--> 顶级域名DNS-->二级域名DNS…

### DNS域名

- 根域：目前有13个根集群服务器，美国10台，日本1台，荷兰1台，瑞典1台
- 一级域名：Top Level Domain: tld com, edu, mil, gov, net, org, int,arpa 组织域、国家域(.cn, .ca, .hk, .tw)、反向域 等
- 二级域名
- 三级域名
- 最多127级域名

ICANN（The Internet Corporation for Assigned Names and Numbers）互联网名称与数字地址分配机构，负责在全球范围内对互联网通用顶级域名（gTLD）以及国家和地区顶级域名（ccTLD）系统的管理、以及根服务器系统的管理

### DNS服务管理工具

dig：只用于测试dns系统，不会查询hosts文件进行解析

- -x IP ：测试反向解析
- -t axfr ZONE_NAME @SERVER ：模拟区域传送
- -t NS . @a.root-servers.net ：查询所有的根DNS服务器

host

- -t：指定查询记录类型
- host www.dongfei.tech 192.168.0.7 ：向192.168.0.7查询www.dongfei.tech这个域名

rndc

- reload: 重载主配置文件和区域解析库文件
- reload zonename: 重载区域解析库文件
- retransfer zonename: 手动启动区域传送，而不管序列号是否增加
- notify zonename: 重新对区域传送发通知
- reconfig: 重载主配置文件
- querylog: 开启或关闭查询日志文件/var/log/message
- trace: 递增debug一个级别
- trace LEVEL: 指定使用的级别
- notrace：将调试级别设置为 0
- flush：清空DNS服务器的所有缓存记录

named-checkconf：检查配置文件的语法

named-checkzone "dongfei.com" /var/named/dongfei.com.zone ：查询区域数据库文件的语法

# 二、DNS服务

DNS的实现：bind（Bekerley Internat Name Domain ） ，由 ISC （www.isc.org） 维护，本章所有配置实例的bind版本为 ：`bind-9.9.4-61.el7.x86_64`

软件包名：`bind`

服务名：`named`

提供的服务：`DNS域名解析`

主配置文件：`/etc/named.conf`

```perl
options {  #全局选项
    listen-on port 53 { 127.0.0.1; };  //默认监听本机的53号端口，如果没有其他需求则注释掉
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    allow-query     { localhost; };  //授权为指定的主机解析，默认只为本机解析，需要注释掉
    recursion yes|no;  //递归查询，默认开启
    dnssec-enable yes;  //sec功能，与安全加密传输相关的选项，如果要做转发，需要关闭此选项
    dnssec-validation yes;  //同上一条，需要关闭
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";
    allow-transfer { none; }：允许区域传送的主机；白名单，默认开启，建议关闭
};

logging {  //日志子系统配置
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {  //根区域定义，如果自己要做根服务器的话需要注释掉
    type hint;
    file "named.ca";
};

include "/etc/named.rfc1912.zones";  //区域定义信息放在此文件
include "/etc/named.root.key";
```

DNS解析数据库文件存放位置：`/var/named/`

区域定义：`/etc/named.rfc1912.zones`

```go
zone "test.com" IN { //internation 记录
    type {master|slave|hint|forward};   //类型，主服务器
    file "test.com.zone";  //区域数据库文件
};  
```

区域文件模板：`/etc/named.rfc1912.zones`

```powershell
# cat /var/named/named.localhost 
$TTL 1D  #默认的TTL值
@       IN SOA  @ rname.invalid. (  						  #SOA记录
                                        0       ; serial		#序列号
                                        1D      ; refresh		#主从复制的时间间隔
                                        1H      ; retry			#如果主从复制失败重试时间
                                        1W      ; expire		#失效时间
                                        3H )    ; minimum  		#否定答案的TTL值
        NS      @			#名字服务器记录
        A       127.0.0.1	 #正向解析记录
        AAAA    ::1			 #IPv6的正向解析记录
```

#### 区域解析数据库文件：

资源记录：Resource Record 简称 RR

 **语法格式**：name [TTL] IN rr_type value

- name：当前区域的名字，列如 `www.example.com.`，注意：最后的 "." 一定要加，如果不加则会把你的域名当成区域名字，再在后边加上你的默认域名。 “@”：表示当前域名的名字

TTL：例如`$TTL 1D`的意思是默认的TTL值为1天

> TTL(Time- To-Live)，简单的说它表示一条域名解析记录在DNS服务器上缓存时间.当各地的DNS服务器接受到解析请求时，就会向域名指定的DNS服务器发出解析请求从而获得解析记录；在获得这个记录之后，记录会在DNS服务器中保存一段时间，这段时间内如果再接到这个域名的解析请求，DNS服务器将不再向DNS服务器发出请求，而是直接返回刚才获得的记录；而这个记录在DNS服务器上保留的时间，就是TTL值。

> 同一个名字可以通过多条记录定义多个不同的值；此时DNS服务器会以轮询方式响应

> 同一个值也可能有多个不同的定义名字；通过多个不同的名字指向同一个值进行定义；此仅表示通过多个不同的名字可以找到同一个主机

- IN：关键字

##### 记录类型

- A ：正向解析，名字到IP

```php
主机名（简称只写主机名，如www）	A	ip地址

@   A   192.168.0.12  		//不需要输入www，直接输入域名则可访问此网站
*   A   192.168.0.12	  //泛域名解析，只要请求的是这个域，但是没有匹配的主机，则应答此条记录
$GENERATE 1-254 websvr$ A 192.168.0.$   //循环，表示websvr1 A 192.168.0.1 到 websvr254 A 192.168.0.254的254条记录
```

- AAAA：正向解析，名字到IPV6
- PTR：反向解析

```basic
12  PTR websrv.dongfei.com.
14  PTR web2srv.dongfei.com.
100 PTR mailsrv.dongfei.com.
```

- NS：名字服务器记录，记录谁是这个域的DNS服务器，包括主服务器和从服务器

```less
		NS	dns  //表示192.168.0.1这台主机是这个域的dns服务器
dns		A	192.168.0.1
```

- CNAME：别名记录

```less
websrv	A	192.168.0.10
websrv	A	192.168.0.11
websrv	A	192.168.0.12
www		CNAME	websrv   //访问www.xxxx.com的时候则代表访问192.168.0.10-12，DNS会做轮调应答，实现负载均衡的功能
```

- MX：邮件记录

```less
@       MX  10  mailsrv1
@       MX  20  mailsrv2
mailsrv1    A   192.168.0.100
mailsrv2    A   192.168.0.200
```

- SOA：起始授权记录，记录域中的主DNS服务器，一个区域解析库中只能有且仅能有一个SOA记录，必须位于解析库的第一条记录

> SOA记录的格式：域名称 IN SOA 主DNS主机名 域管理员邮箱 （序列号 主从同步的时间间隔 同步失败的尝试时间间隔 联系不上主DNS多长时间后失效 域名无法找到，在多长时间内不会再去查询）

```x86asm
@ IN    SOA dns1.dongfei.com. admin.dongfei.com. ( 1 1D 1H 1W 3H )
```

[回到顶部](https://www.cnblogs.com/L-dongf/p/9118111.html#_labelTop)

### 实例1：配置DNS主服务器的正向解析

 1）安装bind

```shell
# yum install bind
# systemctl start named
```

 2）修改主配置文件，将下边俩条注释掉

```perl
# vim /etc/named.conf
//  listen-on port 53 { 127.0.0.1; };  //监听本机的所有地址
//  allow-query     { localhost; };  //允许给所有客户端解析
```

 3）修改区域数据文件

```bash
# vim /etc/named.rfc1912.zones
zone "dongfei.com" IN {  	//internation 记录
    type master;  			//类型，主服务器
    file "dongfei.com.zone";  //区域数据库文件，指向/var/named/dongfei.com.zone
};
```

 4）新建区域解析数据库文件，/var/named/dongfei.com.zone

```swift
# vim /var/named/dongfei.com.zone
$TTL 1D  //代表全局的TTL值
@ IN    SOA dns1.dongfei.com. admin.dongfei.com. ( 1 1D 1H 1W 3H ) //SOA记录，按格式写
        NS  dns1			//NS记录，域中有几个DNS服务器都要写出来
dns1    A   192.168.0.7		 //NDS服务器的A记录，0.7是本机的IP
www     A   192.168.0.6		 //一条正向解析记录，这里的0.6是web服务器

# chgrp named /var/named/dongfei.com.zone  //切记，修改文件的权限和所属组，保证named进程有此文件的可读属性
# chmod 640 /var/named/dongfei.com.zone
# ll /var/named/dongfei.com.zone
-rw-r-----. 1 root named 120 Jun  1 19:39 /var/named/dongfei.com.zone
```

 5）测试

```cpp
# named-checkconf  //检查主配置文件
# named-checkzone "dongfei.com" /var/named/dongfei.com.zone  //检查
# rndc reload  //重载配置文件
# dig www.dongfei.com @192.168.0.7  //在客户端上使用dig命令测试，如果返回有以下值则说明成功
;; ANSWER SECTION:
www.dongfei.com.        86400   IN      A       192.168.0.6
```

[回到顶部](https://www.cnblogs.com/L-dongf/p/9118111.html#_labelTop)

### 实例2：配置DNS主服务器的反向解析

 1）在/etc/name.rfc1912.zone中加入

```go
zone "0.168.192.in-addr.arpa" IN {  //地址需要反着写，与正向解析不同，是另一颗树
    type master;
    file "192.168.0.zone";
};
```

 2）创建区域数据文件

```shell
# vim /var/named/192.168.0.zone
$TTL 1D
@ IN    SOA dns.dongfei.com. admin.dongfei.com. ( 1 1D 1H 1W 3H )
                        NS  dns.dongfei.com.
dns.dongfei.com         A   192.168.0.7
100                     PTR mail.dongfei.com.

# chgrp named /var/named/192.168.0.zone
# chmod 640 /var/named/192.168.0.zone
# rndc reload
```

 3）重载配置文件，测试

```cpp
# vim /etc/resolv.conf
nameserver 192.168.0.7  //在客户端将DNS服务器指向我们配置的DNS服务器
# dig -x 192.168.0.100  //测试，如果出现以下信息则表示成功
;; ANSWER SECTION:
100.0.168.192.in-addr.arpa. 86400 IN    PTR     mail.dongfei.com.
```

[回到顶部](https://www.cnblogs.com/L-dongf/p/9118111.html#_labelTop)

### 实例3：DNS服务的动态更新

 1）打开允许指定主机更新数据库

```fsharp
zone "dongfei.com" IN {
    type master;
    file "dongfei.com.zone";
    allow-update { 192.168.0.6; };  //允许192.168.0.6远程更新数据库
};
```

 2）放开数据库文件夹和文件的权限

```shell
# chmod 770 /var/named/
# ll -d /var/named/
drwxrwx---. 5 root named 173 Jun  1 20:55 /var/named/
# rndc reload
```

 3）在客户端测试，上传一条更新记录

```markdown
# nsupdate 
> server 192.168.0.7
> zone dongfei.com
> update add ftp.dongfei.com 86400 IN A 192.168.0.101
> send
> quit
# dig ftp.dongfei.com @192.168.0.7  // 出现以下信息表示成功
;; ANSWER SECTION:
ftp.dongfei.com.        86400   IN      A       192.168.0.101
```

> 这时我们再回来看DNS服务器的/var/named/文件夹下出现了一个 `dongfei.com.zone.jnl` 的文件，这个文件是更新数据库的日志文件，它不会立即同步到区域数据中库文件中，而是会先存放到日志文件中，过一会儿再向数据库文件中同步。

`# named-journalprint /var/named/dongfei.com.zone.jnl` 查看日志文件

# 三、DNS的主从复制

DNS服务一般需要一台主，俩台从，如果主DNS服务器出现故障后可以向从DNS服务器请求解析；客户端需要将主DNS设置为主DNS服务器，将从DNS服务器设置为备DNS服务器

### 实例3：配置主从DNS服务

> 192.168.0.7 为主DNS服务器
>
> 192.168.0.11 为从DNS服务器
>
> 192.168.0.6 为客户端

 1）主服务器配置

```python
# vim /etc/named.conf
//  listen-on port 53 { 127.0.0.1; };
//  allow-query     { localhost; };
    allow-transfer { 192.168.0.11; };  //只允许从DNS服务器同步区域数据库

# vim /etc/named.rfc1912.zones
zone "dongfei.com" IN {
    type master;
    file "dongfei.com.zone";
};

# vim /var/named/dongfei.com.zone
$TTL 86400  ; 1 day
@ IN SOA    dns1.dongfei.com. admin.dongfei.com. (
                2          ; serial
                86400      ; refresh (1 day)
                3600       ; retry (1 hour)
                604800     ; expire (1 week)
                10800      ; minimum (3 hours)
                )
            NS  dns1
            NS  dns2		 //将从DNS服务器的NS记录添加到此
dns1        A   192.168.0.7
dns2        A   192.168.0.11  //将从DNS服务器的A记录添加到此

ftp         A   192.168.0.101
www         A   192.168.0.6
@           MX  10  mail
mail        A   192.168.0.100
```

 2）配置从DNS服务器

```cpp
# vim /etc/named.conf
//  listen-on port 53 { 127.0.0.1; };
//  allow-query     { localhost; };
    allow-transfer  { none; };  //为了数据安全，不允许任何主机从从DNS服务器上拉取区域数据

# vim /etc/named.rfc1912.zones
zone "dongfei.com" IN {
    type slave;  //类型为从服务器
    masters { 192.168.0.7; };  //主DNS服务器的IP
    file "slaves/dongfei.com.zone.slave";  //数据库文件保存到 /var/named/slaves/ 文件夹下，名字叫dongfei.com.zone.slave
};

# systemctl restart named //重启服务
# ls /var/named/slaves/ //可以看到数据库文件则证明已经同步成功
dongfei.com.zone.slave
```

 3）在客户端测试

```yaml
# vim /etc/resolv.conf  //修改DNS配置文件
; generated by /sbin/dhclient-script
nameserver 192.168.0.7
nameserver 192.168.0.11

# dig www.dongfei.com
;; ANSWER SECTION:
www.dongfei.com.        86400   IN      A       192.168.0.6
;; SERVER: 192.168.0.7#53(192.168.0.7)
```

> 主服务器可以正常解析，接下来我们将主机模拟故障，比如把网络断掉

```less
# dig www.dongfei.com  //再次在客户端测试，发现现在已经是从服务响应我们的解析请求了
;; ANSWER SECTION:
www.dongfei.com.        86400   IN      A       192.168.0.6
;; SERVER: 192.168.0.11#53(192.168.0.11) //192.168.0.11是从服务器地址
```

> dig -t axfr magedu.com @192.168.0.7 手动抓取区域记录

- 注意：
  1. 用户查询是走的UDP的53端口
  2. 主从复制的时候需要放开tcp和udp 的53号端口
  3. 需要的防火墙开放tcp和udp的53号端口

# 四、DNS子域委派

在互联网中我们的单个DNS服务无法去存储所有主机的域名到IP的记录，比如根域，它只是将来至查询`.com` 的请求委派给`.com`域的DNS服务器，来自`.org`的查询交给`.org`域的DNS服务器。具体怎么实现配置，那我们来一起研究吧

[回到顶部](https://www.cnblogs.com/L-dongf/p/9118111.html#_labelTop)

### 实例4：配置子域委派的实现

> 环境：
>
>  父：192.168.0.7，dongfei.com
>
>  子：192.168.0.11，bj.dongfei.com

 1）配置父域

```python
# vim /etc/named.conf
//  listen-on port 53 { 127.0.0.1; };
//  allow-query     { localhost; };
    dnssec-enable no;
    dnssec-validation no;

# vim /etc/named.rfc1912.zones
zone "dongfei.com" IN {
    type master;
    file "dongfei.com.zone";
};

# vim /var/named/dongfei.com.zone
$TTL 86400  ; 1 day
@ IN SOA    dns1.dongfei.com. admin.dongfei.com. (
                2          ; serial
                86400      ; refresh (1 day)
                3600       ; retry (1 hour)
                604800     ; expire (1 week)
                10800      ; minimum (3 hours)
                )
            NS  dns1
bj          NS  dns2.dongfei.com.  //将bj域的请求委派给dns2来处理
dns1        A   192.168.0.7
dns2        A   192.168.0.11

# chgrp named /var/named/dongfei.com.zone
# chmod 640 /var/named/dongfei.com.zone
# rndc reload
```

 2）配置子域

```shell
# vim /etc/named.conf
//  listen-on port 53 { 127.0.0.1; };
//  allow-query     { localhost; };

# vim /etc/named.rfc1912.zones
zone "bj.dongfei.com" IN {
    type master;
    file "bj.dongfei.com.zone";
};

# vim /var/named/bj.dongfei.com.zone
$TTL 1D
@ IN SOA dns1.bj.dongfei.com. admin.bj.dongfei.com. ( 1 1D 1H 1W 3H )
        NS  dns1
dns1    A   192.168.0.11
www     A   192.168.0.6

# chgrp named /var/named/bj.dongfei.com.zone
# chmod 640 /var/named/bj.dongfei.com.zone
# rndc reload
```

 3）在客户端测试

```mipsasm
# dig www.bj.dongfei.com @192.168.0.7
;; ANSWER SECTION:
www.bj.dongfei.com.     86385   IN      A       192.168.0.6
```

# 五、DNS转发

- 1. 全局转发: 对非本机所负责解析区域的请求，全转发给指定的服务器
- 1. 特定区域转发：仅转发对特定的区域的请求，比全局转发优先级高

> 注意：被转发的服务器需要能够为请求者做递归，否则转发请求不予进行；关闭dnssec功能（dnssec-enable no; dnssec-validation no; ）

转发器类型：

1. first模式：优先先转发到目标DNS服务器，如果区域不存在，再转发到根上去查询
2. onyl模式：只转发到目标DNS服务器，如果目标主机没有此查询的信息则转发失败

[回到顶部](https://www.cnblogs.com/L-dongf/p/9118111.html#_labelTop)

### 实例5：配置DNS转发（全局转发）

> 192.168.0.11 为转发DNS服务器
>
> 192.168.0.7 为目标DNS服务器

 1）配置转发DNS服务器

```perl
# vim /etc/named.conf
options {
//  listen-on port 53 { 127.0.0.1; };
//  allow-query     { localhost; };
    recursion yes;  //开启递归查询
    dnssec-enable no;
    dnssec-validation no;
    forward only;  //only模式
    forwarders { 192.168.0.7; };  //目标DNS服务器IP
};

# rndc reload //重载配置文件
```

 2）配置目标DNS服务器

```python
# vim /etc/named.conf
//  listen-on port 53 { 127.0.0.1; };
//  allow-transfer { 192.168.0.11; };
    recursion no;  //关闭递归查询
    dnssec-enable no;
    dnssec-validation no;

# vim /etc/named.rfc1912.zones
zone "dongfei.com" IN {
    type master;
    file "dongfei.com.zone";
};

# vim /var/named/dongfei.com.zone
$TTL 86400  ; 1 day
@ IN SOA    dns1.dongfei.com. admin.dongfei.com. (
                2          ; serial
                86400      ; refresh (1 day)
                3600       ; retry (1 hour)
                604800     ; expire (1 week)
                10800      ; minimum (3 hours)
                )
            NS  dns1
dns1        A   192.168.0.7
webs        A   192.168.0.6
webs        A   192.168.0.5
www         CNAME   webs
@           MX  10 mail
mail        A   192.168.0.100
ftp         A   192.168.0.101

# rndc reload
```

 3）在客户端测试

```less
# dig www.dongfei.com @192.168.0.11
;; ANSWER SECTION:
www.dongfei.com.        86369   IN      CNAME   webs.dongfei.com.
webs.dongfei.com.       86369   IN      A       192.168.0.6
webs.dongfei.com.       86369   IN      A       192.168.0.5
;; SERVER: 192.168.0.11#53(192.168.0.11)  //真正的域名解析信息在192.168.0.7上，这里由0.11代为去查询
```

> 如果对单个域进行转发则把配置写到区域配置文件中即可
>
> ```haskell
> # vim /etc/named.rfc1912.zones
> zone "dongfei.com" {
>    type forward;
>    forward only;
>    forwarders { 192.168.0.7; };
> };
> ```

# 六、智能DNS

**在**互联网上各个地区的网络站点分布到各个地区，这时就需要按地区为当地地区的客户解析到当地的站点服务器，比如在北京和在上海打开同一个网站显示的信息是不同的；这就需要用到智能DNS解析的技术。

**电**商站点或者视频站点，这些站点需要快速响应客户的请求，不可能将服务器搭建到一个地区，而是需要分布到各个省市，在每个地方有缓存服务器，这就是CDN: Content Delivery Network内容分发网络的工作，一般由单独的CDN公司搭建机房服务于各大电商视频等站点。

接下来，我们一起研究一下如何实现智能DNS解析吧

## ACL规则

acl: 把一个或多个地址归并为一个集合，并通过一个统一的名称调用；只能先定义，后使用，因此一般定义在配置文件中，处于options的前面

格式：

```bash
acl acl_name { 
  ip; 
  net/prelen; 
  …… 
}; 
```

> bind有四个内置的acl：
>
>  none: 没有一个主机
>
>  any: 任意主机
>
>  localhost: 本机
>
>  localnet: 本机的IP同掩码运算后得到的网络地址

访问控制的指令：

- allow-query {}： 允许查询的主机；白名单
- allow-transfer {}：允许区域传送的主机；白名单
- allow-recursion {}: 允许递归的主机,建议全局使用
- allow-update {}: 允许更新区域数据库中的内容

## view 视图

view:视图：实现智能DNS

- 一个bind服务器可定义多个view，每个view中可定义一个或多个zone
- 每个view用来匹配一组客户端
- 多个view内可能需要对同一个区域进行解析，但使用不同的区域解析库文件

> 注意：
> (1) 一旦启用了view，所有的zone都只能定义在view中
> (2) 仅在允许递归请求的客户端所在view中定义根区域
> (3) 客户端请求到达时，是自上而下检查每个view所服务的客户端列表

[回到顶部](https://www.cnblogs.com/L-dongf/p/9118111.html#_labelTop)

### 实例6：ACL和view实现智能DNS

> 在一台主机上有俩张网卡，配置俩个网段来模拟来自不同地区的客户
>
>  192.168.0.7/24
>
>  172.20.111.236/16

 1）添加ACL和视图

```fsharp
# vim /etc/named.conf
acl bjnet {  //注意，acl要写在options前边，而且要注意acl的匹配顺序关系，至上而下
    192.168.0.0/24;
};
acl shnet {
    172.20.0.0/16;
};
acl othernet {
    any;
};

options {
//  listen-on port 53 { 127.0.0.1; };
//  allow-query     { localhost; };
};

view bjview {
    match-clients {bjnet;};
    include "/etc/named.rfc1912.zones.beijing";
};
view shview {
    match-clients {shnet;};
    include "/etc/named.rfc1912.zones.shanghai";
};
view otherview {
    match-clients {othernet;};
    include "/etc/named.rfc1912.zones";
};

include "/etc/named.root.key";
```

> 注意：要将默认配置文件中的根区域配置放到/etc/named.rfc1912.zones文件中

 2）创建各地区区域配置文件

```haskell
# vim /etc/named.rfc1912.zones.beijing 
zone "dongfei.com" {
    type master;
    file "dongfei.com.zones.beijing";
};
# vim /etc/named.rfc1912.zones.shanghai
zone "dongfei.com" {
    type master;
    file "dongfei.com.zones.shanghai";
};
```

 3）配置解析数据库文件

```x86asm
# vim /var/named/dongfei.com.zones.beijing
$TTL 1D
@ IN SOA dns1.dongfei.com. admin.dongfei.com. ( 1 1D 1H 1W 3H )
        NS  dns1
dns1    A   192.168.0.7
www     A   192.168.0.1
# vim /var/named/dongfei.com.zones.shanghai
$TTL 1D
@ IN SOA dns1.dongfei.com. admin.dongfei.com. ( 1 1D 1H 1W 3H )
        NS  dns1
dns1    A   172.20.111.236
www     A   172.20.111.1
```

 4）在客户端测试

```yaml
# dig www.dongfei.com @192.168.0.7
;; ANSWER SECTION:
www.dongfei.com.        86400   IN      A       192.168.0.1
;; SERVER: 192.168.0.7#53(192.168.0.7)

# dig www.dongfei.com @172.20.111.236
;; ANSWER SECTION:
www.dongfei.com.        86400   IN      A       172.20.111.1
;; SERVER: 172.20.111.236#53(172.20.111.236)
```

> ###### 从上边的测试结果，从不同IP段查询同一个域名得到的结果却不一样，从而可以实现按地区来智能解析

# 七、模拟搭建互联网名称解析服务架构

![img](https://images2018.cnblogs.com/blog/1331725/201806/1331725-20180602202619213-738091695.png)

 1）**192.168.0.1** : web

```shell
# echo -e web1.dongfei.com\n\<h1\>hello web1\</h1\> > /var/www/html/index.html
# service httpd start
# curl 192.168.0.1
# web1.dongfei.comn<h1>hello web1</h1>
```

 2）**192.168.0.2** : web2

```css
# echo -e web2.dongfei.com\n\<h2\>hello web1\</h1\> > /var/www/html/index.html
# service httpd start
# curl 192.168.0.2
web2.dongfei.comn<h2>hello web2</h1>
```

 3）**192.168.0.3** ：dns1，配置dongfei.com域的主DNS服务器

```perl
# vim /etc/named.conf
options {
//  listen-on port 53 { 127.0.0.1; };  //注释掉
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
//  allow-query     { localhost; };  //注释掉
    allow-transfer { 192.168.0.4; };  //只允许192.168.0.4的主机，也就是从DNS来拉取区域解析数据库
    recursion yes;
    dnssec-enable yes;
    dnssec-validation yes;
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};
# vim /etc/named.rfc1912.zones
zone "dongfei.com" IN {
    type master;
    file "dongfei.com.zone";
};
# vim /var/named/dongfei.com.zone
$TTL 1D
@ IN SOA dns1.dongfei.com. admin.dongfei.com. ( 1 1D 1H 1W 3H )
        NS  dns1
        NS  dns2
dns1    A   192.168.0.3
dns2    A   192.168.0.4

webs    A   192.168.0.1
webs    A   192.168.0.2
www     CNAME   webs
# chgrp named /var/named/dongfei.com.zone
# chmod 640 /var/named/dongfei.com.zone
# named-checkconf
# named-checkzone "dongfei.com" /var/named/dongfei.com.zone
# service named start
# dig www.dongfei.com @192.168.0.3
;; ANSWER SECTION:
www.dongfei.com.        86400   IN      CNAME   webs.dongfei.com.
webs.dongfei.com.       86400   IN      A       192.168.0.1
webs.dongfei.com.       86400   IN      A       192.168.0.2
```

 4）**192.168.0.4** ：dns2，配置dongfei.com域的从DNS服务器

```perl
# vim /etc/named.conf 
options {
//  listen-on port 53 { 127.0.0.1; };  //注释掉
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
//  allow-query     { localhost; };  //注释掉
    allow-transfer { none; };  //不对任何主机做区域传送
    recursion yes;
    dnssec-enable yes;
    dnssec-validation yes;
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};
# vim /etc/named.rfc1912.zones
zone "dongfei.com" {
    type slave;  //类型为从服务器
    masters { 192.168.0.3; };  //指向谁是我的主服务器
    file "slaves/dongfei.com.zone.slave";  //解析数据库存放位置
};
# named-checkconf 
# service named start
# ls -l /var/named/slaves/  //查看一下有没有数据库文件，有则说明同步成功
-rw-r--r--. 1 named named 417 May 19 22:09 dongfei.com.zone.slave
```

 5）**192.168.0.5** ：com. 配置子域委派

```perl
# vim /etc/named.conf
options {
//  listen-on port 53 { 127.0.0.1; };
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
//  allow-query     { localhost; };
    recursion yes;
    dnssec-enable yes;
    dnssec-validation yes;
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};
# vim /etc/named.rfc1912.zones
zone "com" IN {
    type master;
    file "com.zone";
};
# vim /var/named/com.zone
@ IN SOA dns1.com. admin.com. ( 1 1D 1H 1W 3H )
        NS  dns1
dongfei NS  dns2.com.  //子域委派给192.168.0.3和192.168.0.4
dns1    A   192.168.0.5
dns2    A   192.168.0.3
dns2    A   192.168.0.4
# chgrp named /var/named/com.zone
# chmod 640 /var/named/com.zone
# named-checkconf
# service named start
# dig www.dongfei.com @192.168.0.5
www.dongfei.com.        86387   IN      CNAME   webs.dongfei.com.
webs.dongfei.com.       86387   IN      A       192.168.0.1
webs.dongfei.com.       86387   IN      A       192.168.0.2
;; AUTHORITY SECTION:
dongfei.com.            86400   IN      NS      dns2.com.
;; ADDITIONAL SECTION:
dns2.com.               86400   IN      A       192.168.0.4
dns2.com.               86400   IN      A       192.168.0.3
;; SERVER: 192.168.0.5#53(192.168.0.5)
```

 6）**192.168.0.6** ：根域配置

```perl
# vim /etc/named.conf
options {
//  listen-on port 53 { 127.0.0.1; };
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
//  allow-query     { localhost; };
    recursion yes;
    dnssec-enable no;
    dnssec-validation no;
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};
# vim /etc/named.rfc1912.zones
zone "." {
    type master;
    file "root.zone";
};
# vim /var/named/root.zone 
$TTL 1D
@ IN SOA dns1. admin. ( 1 1D 2H 3D 1H )
        NS  dns1
com     NS  dns2
dns1    A   192.168.0.6
dns2    A   192.168.0.5
# chgrp named /var/named/root.zone
# chmod 640 /var/named/root.zone
# service named start
# dig www.dongfei.com @127.0.0.1
;; ANSWER SECTION:
www.dongfei.com.        86177   IN      CNAME   webs.dongfei.com.
webs.dongfei.com.       86177   IN      A       192.168.0.1
webs.dongfei.com.       86177   IN      A       192.168.0.2
```

 7）**192.168.0.7** ：缓存DNS服务器配置

```perl
# vim /etc/named.conf
options {
//  listen-on port 53 { 127.0.0.1; };
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
//  allow-query     { localhost; };
    recursion yes;
    dnssec-enable no;
    dnssec-validation no;
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};  

zone "." IN {
    type hint;
    file "named.ca";
};
# vim /var/named/named.ca 
.                        3600000      NS    A.ROOT-SERVERS.NET.
A.ROOT-SERVERS.NET.      3600000      A     192.168.0.6  //将根服务器指向我们自己搭建的根DNS服务器
# service named start
```

 8）**192.168.0.8** ：在客户端测试

```cpp
# vim /etc/resolv.conf
nameserver 192.168.0.7  //将自己的DNS服务器指向缓存服务器
# dig www.dongfei.com  //出现以下信息则说明成功
;; ANSWER SECTION:
www.dongfei.com.        86319   IN      CNAME   webs.dongfei.com.
webs.dongfei.com.       86319   IN      A       192.168.0.1
webs.dongfei.com.       86319   IN      A       192.168.0.2
;; SERVER: 192.168.0.7#53(192.168.0.7)
```

 到此为止，我们可以访问以下www.dongfei.com，看看是否可以正常解析

```css
[root@client ~]# curl www.dongfei.com 
web2.dongfei.comn<h2>hello web2</h1>
[root@client ~]# curl www.dongfei.com 
web1.dongfei.comn<h1>hello web1</h1>
[root@client ~]# curl www.dongfei.com 
web2.dongfei.comn<h2>hello web2</h1>
[root@client ~]# curl www.dongfei.com 
web1.dongfei.comn<h1>hello web1</h1>
```

 从测试结果看来，不仅可以正常解析，还实现了DNS负载均衡的功能。



DNS技术和NAT技术详解

一.DNS(Domain Name System)

1.什么是DNS

\2. 了解域名

3.域名解析过程

4.使用dig工具分析DNS过程

5.浏览器输入URL后发生什么事？

二.ICMP协议

1.ICMP功能

2.ICMP报文格式

3.ping命令

4.traceroute命令

3.NAT技术

1.为什么要要NAT技术

\2. NAT技术IP地址的替换过程

3.NAPT技术

4.NAT技术的缺陷

5.NAT-PT（NAPT-PT）

6.NAT和代理服务器

一.DNS(Domain Name System)

1.什么是DNS

DNS是一套从域名到IP的映射系统。

TCP/IP中使用IP地址和端口号来确定网络上的一台主机的一个程序，但是IP地址不方便记忆。于是人们发明了一种叫主机名的东西是一个字符串，并且使用hosts文件来描述主机名和IP地址的关系。 用户可以简单的输入主机名xxxx，这样就可以简单的得到主机名和IP地址的映射关系，它存储在hosts文件中。最初通过互连网信息中心(SRI-NIC)来管理这个hosts文件的：

 

如果一个新计算机要接入网络, 或者某个计算机IP变更, 都需要到信息中心申请变更hosts文件。

其他计算机也需要定期下载更新新版本的hosts文件才能正确上网。

但是这样做太过麻烦了，所以就产生了域名解析系统DNS：

 

一个组织的系统管理机构，维护系统内的每个主机的IP和主机名的对应关系

如果新计算机接入网络，将这个信息注册到数据库中

用户输入域名的时候，会自动查询DNS服务器，由DNS服务器检索数据库, 得到对应的IP地址。

注：我们的计算机上仍然保留了hosts文件,在域名解析的过程中仍然会优先查找hosts文件的内容。

 

 

\2. 了解域名

主域名是用来识别主机名称和主机所属的组织机构的一种分层结构的名称。

例如：www.baidu.com(域名使用.连接)

 

com： 一级域名，表示这是一个企业域名。同级的还有 "net"(网络提供商)，"org"(非盈利组织) 等。

baidu: 二级域名, 公司名。

www: 只是一种习惯用法，之前人们在使用域名时，往往命名成类似于ftp.xxx.xxx/www.xxx.xxx这样的格式来表示主机支持的协议。

3.域名解析过程

客户机提出域名解析请求,并将该请求发送给本地的域名服务器。

当本地的域名服务器收到请求后,就先查询本地的缓存,如果有该纪录项,则本地的域名服务器就直接把查询的结果返回。

如果本地的缓存中没有该纪录,则本地域名服务器就直接把请求发给根域名服务器,然后根域名服务器再返回给本地域名服务器一个所查询域(根的子域)的主域名服务器的地址。

本地服务器再向上一步返回的域名服务器发送请求,然后接受请求的服务器查询自己的缓存,如果没有该纪录,则返回相关的下级的域名服务器的地址。

重复第四步,直到找到正确的纪录。

本地域名服务器把返回的结果保存到缓存,以备下一次使用,同时还将结果返回给客户机

4.使用dig工具分析DNS过程

安装dig工具：

 

yum install bind-utils

1

使用dig查看域名解析过程：

 

dig www.baidu.com

1

如下图：

 

 

5.浏览器输入URL后发生什么事？

详细介绍：

https://blog.csdn.net/wuhenliushui/article/details/20038819/

 

二.ICMP协议

ICMP协议是一个 网络层协议 一个新搭建好的网络，往往需要先进行一个简单的测试，来验证网络是否畅通。但是IP协议并不提供可靠传输，如果丢包了IP协议并不能通知传输层是否丢包以及丢包的原因。

 

1.ICMP功能

确认IP包是否成功到达目标地址

通知在发送过程中IP包被丢弃的原因

ICMP也是基于IP协议工作的，但是它并不是传输层的功能, 因此人们仍然把它归结为网络层协议。

ICMP只能搭配IPv4使用. 如果是IPv6的情况下, 需要是用ICMPv6>

2.ICMP报文格式

|----------------|---------------|----------------------------------|

|    类型   |    代码   |        检验和       |

|----------------|---------------|----------------------------------|

|           不同的类型和代码存在不同的内容           |

\---------------------------------------------------------------------

1

2

3

4

5

ICMP报文一般分为两类：

 

一类是通知出错原因

一类是用于诊断查询

3.ping命令

ping命令不光能验证网络的连通性, 同时也会统计响应时间和TTL(IP包中的Time To Live, 生存周期)。

ping命令会先发送一个 ICMP Echo Request给对端，对端接收到之后, 会返回一个ICMP Echo Reply。

ping 的是域名，而不是url。一个域名可以通过DNS解析成IP地址。

 

telnet是23端口, ssh是22端口, 那么ping是什么端口?

答：ping命令基于ICMP, 是在网络层. 而端口号, 是传输层的内容. 在ICMP中根本就不关注端口号这样的信息。

4.traceroute命令

traceroute (Windows 系统下是tracert) 命令利用ICMP协议定位您的计算机和目标计算机之间的所有路由器。TTL值可以反映数据包经过的路由器或网关的数量，通过操纵独立ICMP 呼叫报文的TTL 值和观察该报文被抛弃的返回信息，traceroute命令能够遍历到数据包传输路径上的所有路由器。

 

 

3.NAT技术

1.为什么要要NAT技术

IPv4的地址不充足的问题对于计算机网络的发展是一个很大的问题，NAT技术是当前解决IP地址不够用的主要手段，它是路由器的一个重要功能。

 

NAT能够将私有IP对外通信时转为全局IP。也就是就是一种将私有IP和全局IP相互转化的技术方法。

很多学校、家庭、公司内部采用每个终端设置私有IP, 而在路由器或必要的服务器上设置全局IP。

全局IP地址要求唯一，但是私有IP不需要。在不同的局域网中出现相同的私有IP是完全不影响的。

\2. NAT技术IP地址的替换过程

 

 

NAT路由器将源地址从10.0.0.10替换成全局的IP 202.244.174.37

NAT路由器收到外部的数据时, 又会把目标IP从202.244.174.37替换回10.0.0.10

在NAT路由器内部, 有一张自动生成的用于地址转换的表

当 10.0.0.10第一次向163.221.120.9 发送数据时就会生成表中的映射关系

3.NAPT技术

如果局域网内, 有多个主机都访问同一个外网服务器， 那么对于服务器返回的数据中, 目的IP都是相同的。 那么NAT路由器如何判定将这个数据包转发给哪个局域网的主机? NAPT技术使用IP+Port来解决这个问题。

 

 

在使用TCP或UDP的通信当中，只有目标地址、源地址、目标端口、源端口以及协议类型（TCP还是UDP）五项内容都一致时才被认为是同一个通信连接。此时所使用的正是NAPT。

 

这种转换表在NAT路由器上自动生成。例如，在TCP情况下，建立TCP连接首次握手时的SYN包一经发出，就会生成这个表。而后又随着收到关闭连接时发出FIN包的确认应答从表中被删除。

 

4.NAT技术的缺陷

无法从NAT外部向内部服务器建立连接。

转换表的生成和销毁都需要额外开销。

通信过程中一旦NAT设备异常，即使存在备份, 所有的TCP连接也都会断开。

5.NAT-PT（NAPT-PT）

现在很多互联网服务都基于IPv4。如果这些服务不能做到IPv6中也能正常使用的话，搭建IPv6网络环境的有时也就无从谈起。 为此，就产生了NAT-PT（NAPT-PT）规范，PT是Protocol Translation的缩写。NAT-PT是将IPv6的首部转换为IPv4的首部的一种技术。有了这种技术，那些只有IPv6地址的主机也就能够与IPv4地址的其他主机进行通信了。

 

 

6.NAT和代理服务器

路由器往往都具备NAT设备的功能，通过NAT设备进行中转，完成子网设备和其他子网设备的通信过程。代理服务器看起来和NAT设备有一点像， 客户端像代理服务器发送请求, 代理服务器将请求转发给真正要请求的服务器，服务器返回结果后代理服务器又把结果回传给客户端。

 

NAT和代理服务器的区别：

 

NAT设备是网络基础设备之一，解决的是IP不足的问题，而代理服务器则是更贴近具体应用, 比如通过代理服务器进行翻墙，另外像迅游这样的加速器, 也是使用代理服务器。

NAT是工作在网络层直接对IP地址进行替换。代理服务器往往工作在应用层。

NAT一般在局域网的出口部署，代理服务器可以在局域网做也可以在广域网做也可以跨网。

NAT一般集成在防火墙，路由器等硬件设备上。代理服务器则是一个软件程序, 需要部署在服务器上。

代理服务器的应用：

 

翻墙: 广域网中的代理。

负载均衡: 局域网中的代理。

正向代理与反向代理的区别：

 

花王尿不湿是一个很经典的尿不湿品牌, 产自日本. 我自己去日本买尿不湿比较不方便, 但是可以让我在日本工作的表姐去超市买了快递给我. 此时超市看到的买家是我表 姐, 我的表姐就是 “正向代理”; 后来找我表姐买尿不湿的人太多了, 我表姐觉得天天去超市太麻烦, 干脆去超市买了一大批尿不湿屯在家里, 如果有人 来找她代购, 就直接把屯在家里的货发出去, 而不必再去超超市. 此时我表姐就是 “反向代理” 。

 

正向代理用于请求的转发(例如借助代理绕过反爬虫)

反向代理往往作为一个缓存

\--------------------- 

作者：Hansionz 

来源：CSDN 

原文：https://blog.csdn.net/hansionz/article/details/86570290 

版权声明：本文为博主原创文章，转载请附上博文链接！

　　本文将详细讲解如何用go语言一步一步实现dns域名解析的过程，并简单介绍点dns有关的知识，直接开始正题吧。

　　首先我们要了解dns解析的过程，没有了解的请看这里[DNS入门（转）](http://www.cnblogs.com/chase-wind/p/6803696.html)很详细。扫盲结束后，我们需要了解下dns报文格式，知道了报文的格式是怎样的，才可以写代码构造dns请求包：   

　　dns请求和应答都是用相同的报文格式，分成5个段（有的报文段在不同的情况下可能为空），如下：      

　　![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps8115.tmp.jpg)

　　Header段是报文的头部，它定义了报文是请求还是应答，也定义了其他段是否需要存在，以及是标准查询还是其他。   

　　Header包含如下字段：

　　![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps8126.tmp.jpg)    

　　各字段分别解释如下：

　　ID:请求客户端设置的16位标示，服务器给出应答的时候会带相同的标示字段回来，这样请求客户端就可以区分不同的请求应答了。

　　QR:1个比特位用来区分是请求（0）还是应答（1）。

　　OPCODE:4个比特位用来设置查询的种类，应答的时候会带相同值，可用的值如下： 0 标准查询 (QUERY) 1 反向查询 (IQUERY) 2 服务器状态查询 (STATUS) 3-15保留值，暂时未使用

　　AA:授权应答(Authoritative Answer) - 这个比特位在应答的时候才有意义，指出给出应答的服务器是查询域名的授权解析服务器。注意因为别名的存在，应答可能存在多个主域名，这个AA位对应请求名，或者应答中的第一个主域名。

　　TC:截断(TrunCation) - 用来指出报文比允许的长度还要长，导致被截断。  

　　RD:期望递归(Recursion Desired) - 这个比特位被请求设置，应答的时候使用的相同的值返回。如果设置了RD，就建议域名服务器进行递归解析，递归查询的支持是可选的。  

　　RA:支持递归(Recursion Available) - 这个比特位在应答中设置或取消，用来代表服务器是否支持递归查询。  

　　Z:保留值，暂时未使用。在所有的请求和应答报文中必须置为0。  

　　RCODE:应答码(Response code) - 这4个比特位在应答报文中设置，代表的含义如下：

　　　　0 没有错误。

　　　　1 报文格式错误(Format error) - 服务器不能理解请求的报文。

　　　　2 服务器失败(Server failure) - 因为服务器的原因导致没办法处理这个请求。

　　　　3 名字错误(Name Error) - 只有对授权域名解析服务器有意义，指出解析的域名不存在。

　　　　4 没有实现(Not Implemented) - 域名服务器不支持查询类型。

　　　　5 拒绝(Refused) - 服务器由于设置的策略拒绝给出应答。比如，服务器不希望对某些请求者给出应答，或者服务器不希望进行某些操作（比如区域传送zone transfer）。

　　　　6-15 保留值，暂时未使用。

　　QDCOUNT 无符号16位整数表示报文请求段中的问题记录数。

　　ANCOUNT 无符号16位整数表示报文回答段中的回答记录数。

　　NSCOUNT 无符号16位整数表示报文授权段中的授权记录数。

　　ARCOUNT 无符号16位整数表示报文附加段中的附加记录数。

　　根据这些，dns头部的数据结构可以定义如下：

　　type dnsHeader struct {

  　 Id                 uint16

 　　 Bits                uint16

 　  Qdcount, Ancount, Nscount, Arcount uint16

　　}

　　构造头部信息我们主要处理Bits，可以直接根据需求对相应位置值，也可以定义好每一个字段，通过移位的方式然后相加构造请求的头部各个字段。推荐后一种方法，这样就有：

  　　header.Bits = QR<<15 + OperationCode<<11 + AuthoritativeAnswer<<10 + Truncation<<9 + RecursionDesired<<8 + RecursionAvailable<<7 + ResponseCode

  其他的头部信息就比较简单了：

　　requestHeader := dnsHeader{

​    Id:    0x0010,

​    Qdcount: 1,

​    Ancount: 0,

​    Nscount: 0,

​    Arcount: 0,

　　}

　　报文头搞定后，接下来就是查询问题Question：

　　Question段描述了查询的问题，包括查询类型(QTYPE)，查询类(QCLASS)，以及查询的域名(QNAME)。字段含义如下  QNAME：域名被编码为一些labels序列，每个labels包含一个字节表示后续字符串长度，以及这个字符串，以0长度和空字符串来表示域名结束。注意这个字段可能为奇数字节，不需要进行边界填充对齐。  QTYPE：2个字节表示查询类型，.取值可以为任何可用的类型值，以及通配码来表示所有的资源记录。  QCLASS：2个字节表示查询的协议类，比如，IN代表Internet。所以我们直接定义查询的结构体如下：

　　type dnsQuery struct {

  　　QuestionType  uint16

  　　QuestionClass uint16

　　}

查询的域名不定义在查询的结构体中，由函数接收参数的方式接收。

　　剩下的3个段包含相同的格式:一系列可能为空的资源记录(RRs)。Answer段包含回答问题的RRs；授权段包含授权域名服务器的RRs；附加段包含和请求相关的，但是不是必须回答的RRs。而在发送请求的时候，我们是发起请求方，所以这些字段放空就好。

完整代码：

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps8127.tmp.png) 

// 002 project main.go

package main

 

import (

  "bytes"

  "encoding/binary"

  "fmt"

  "net"

  "strings"

  "time"

)

 

type dnsHeader struct {

  Id                 uint16

  Bits                uint16

  Qdcount, Ancount, Nscount, Arcount uint16

}

 

func (header *dnsHeader) SetFlag(QR uint16, OperationCode uint16, AuthoritativeAnswer uint16, Truncation uint16, RecursionDesired uint16, RecursionAvailable uint16, ResponseCode uint16) {

  header.Bits = QR<<15 + OperationCode<<11 + AuthoritativeAnswer<<10 + Truncation<<9 + RecursionDesired<<8 + RecursionAvailable<<7 + ResponseCode

}

 

type dnsQuery struct {

  QuestionType  uint16

  QuestionClass uint16

}

 

func ParseDomainName(domain string) []byte {

  var (

​    buffer  bytes.Buffer

​    segments []string = strings.Split(domain, ".")

  )

  for _, seg := range segments {

​    binary.Write(&buffer, binary.BigEndian, byte(len(seg)))

​    binary.Write(&buffer, binary.BigEndian, []byte(seg))

  }

  binary.Write(&buffer, binary.BigEndian, byte(0x00))

 

  return buffer.Bytes()

}

func Send(dnsServer, domain string) ([]byte, int, time.Duration) {

  requestHeader := dnsHeader{

​    Id:    0x0010,

​    Qdcount: 1,

​    Ancount: 0,

​    Nscount: 0,

​    Arcount: 0,

  }

  requestHeader.SetFlag(0, 0, 0, 0, 1, 0, 0)

 

  requestQuery := dnsQuery{

​    QuestionType:  1,

​    QuestionClass: 1,

  }

 

  var (

​    conn  net.Conn

​    err   error

​    buffer bytes.Buffer

  )

 

  if conn, err = net.Dial("udp", dnsServer); err != nil {

​    fmt.Println(err.Error())

​    return make([]byte, 0), 0, 0

  }

  defer conn.Close()

 

  binary.Write(&buffer, binary.BigEndian, requestHeader)

  binary.Write(&buffer, binary.BigEndian, ParseDomainName(domain))

  binary.Write(&buffer, binary.BigEndian, requestQuery)

 

  buf := make([]byte, 1024)

  t1 := time.Now()

  if _, err := conn.Write(buffer.Bytes()); err != nil {

​    fmt.Println(err.Error())

​    return make([]byte, 0), 0, 0

  }

  length, err := conn.Read(buf)

  t := time.Now().Sub(t1)

  return buf, length, t

}

func main() {

  remsg, n, _ := Send("114.114.114.114:53", "www.baidu.com")

  fmt.Println(remsg, n)

}

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps8128.tmp.png) 

 

抓个包看看：

 ![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps8139.tmp.jpg)

这是发出去的，看看详细的Questions信息：

 ![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps813A.tmp.jpg)

我们设置的请求类型是1,class是1，意味着是请求A记录，class IN。下一节我们在来讨论下如何处理服务器端响应的内容。

　上一节已经讲了如何构造dns请求包的情况，这一节接着上一节的情况，谈谈dns查询报文中的问题部分。问题部分中每个问题的格式如下：

　　![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps813B.tmp.png)

　　

　　查询名是要查找的名字，它是一个或者多个标识符的序列。每个标识符以首字母字节的计数值来说明随后标识符的字节长度，每个查询名以最后字节为0结束，长度为0的标识符是根标识符。具体情况我们抓个包看看：

　　![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps814B.tmp.jpg)

　　我们看到请求的名字是www.baidu.com发出的包的内容是下面的红线标识的部分，要查找的名字被转变成了3www5baidu3com这种的格式，所以我们在构造dns查询请求包的时候，需要把查询的名字格式改改：

　　var (

   　　 buffer  bytes.Buffer

  　　  segments []string = strings.Split(domain, ".")

 　　 )

  　 for _, seg := range segments {

  　   binary.Write(&buffer, binary.BigEndian, byte(len(seg)))

   　　 binary.Write(&buffer, binary.BigEndian, []byte(seg))

  　　}

  　 binary.Write(&buffer, binary.BigEndian, byte(0x00))

  　 return buffer.Bytes()

　　完整的代码请移步上一节。查询报文中的问题部分就简单介绍到这里。

前面说了构造请求发送报文，接下来我们好好研究下如何解析服务器端发回来的应答信息。

首先还是用前面的程序代码发一个请求，用抓包工具看看应答的内容有哪些：

 ![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps814C.tmp.jpg)

　　截图的第一部分是返回信息的统计，表明这个返回的包数据包含一个问题，5个权威应答，5个附加信息。第二部分是问题的内容，第三部分是权威应答的内容，第四部分是附加信息的内容。再往下面就是接收到的原始数据的展示，这里需要提及的一点就是为了减小报文，域名系统使用一种压缩方法来消除报文中域名的重复。使用这种方法，后面重复出现的域名或者labels被替换为指向之前出现位置的指针。    

　　指针占用2个字节，格式如下：   

0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5   

+--+--+--+--+--+--+--+--+--+--+--+--+--   

| 1 1|        OFFSET          |   

+--+--+--+--+--+--+--+--+--+--+--+--+--     

　　前两个比特位都为1。因为lablels限制为不多于63个字节，所以label的前两位一定为0，这样就可以让指针与label进行区分。(10 和 01 组合保留，以便日后使用) 。偏移值(OFFSET)表示从报文开始的字节指针。偏移量为0表示ID字段的第一个字节。    

　　压缩方法让报文中的域名成为：    

　　　　- 以0结尾的labels序列    

　　　　- 一个指针    

　　　　- 指针结尾的labels序列    

　　不理解的往下面看，我选中的这部分是权威记录中的一条信息，对应的是下面的选中部分：

 ![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps815D.tmp.jpg)

　　前两个字节’c0 0c’就是之前出现baidu.com位置的指针，’00 02’表明返回的类型是NS，’00 01’表明是Internet协议，’00 03 a3 00’是TTL,’00 06’是后面跟着的内容的长度，告诉你后面的6个字节是返回给你的ns信息，你往后读6个字节第一条返回的权威记录就结束了，看看后面6个字节都是什么：’03 64 6e 73 c0 c0’这个的意思翻译过来就是：3dns+一个指针（指向的内容是baidu.com），所以我们就可以解析出baidu.com的一条ns记录是dns.baidu.com。后面的类似，就不多说了。

原理性的东西分析透彻了，下一节再来聊聊怎样用代码去实现。

 