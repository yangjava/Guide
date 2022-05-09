BUILD

这个目录在本系列的上篇文章中我们仔细看过，内含各种平台的编译脚本，这里就不仔细说了。

client

这个目录下有如下比较让人眼熟的文件: mysql.cc, mysqlcheck.c, mysqladmin.cc,

mysqlshow.c，等等，如果你编译一下就会发现那些眼熟的程序也出现了，比如mysql。明白了吧，这个目录就是那些客户端程序所在的目录。这个

目录的内容也比较少，而且也不是我们阅读的重点。

storage

这个目录包含了所谓的Mysql存储引擎 (storage

engine)。存储引擎是数据库系统的核心，封装了数据库文件的操作，是数据库系统是否强大最重要的因素。Mysql实现了一个抽象接口层，叫做

handler(sql/handler.h)，其中定义了接口函数，比如：ha_open, ha_index_end,

ha_create等等，存储引擎需要实现这些接口才能被系统使用。这个接口定义超级复杂，有900多行

:-(，不过我们暂时知道它是干什么的就好了，没必要深究每行代码。对于具体每种引擎的特点，我推荐大家去看mysql的在线文档:

http://dev.mysql.com/doc/refman/5.1/en/storage-engines.html

应该能看到如下的目录:

* innobase, innodb的目录，当前最流行的存储引擎

* myisam, 最早的Mysql存储引擎,一直到innodb出现以前，使用最广的引擎。

* heap, 基于内存的存储引擎

* federated, 一个比较新的存储引擎

* example, csv，这几个大家可以作为自己写存储引擎时的参考实现，比较容易读懂

mysys

包含了对于系统调用的封装，用以方便实现跨平台。大家看看文件名就大概知道是什么情况了。

sql

这个目录是另外一个大块头，你应该会看到mysqld.cc，没错，这里就是数据库主程序mysqld所在的地方。大部分的系统流程都发生在这里。你还能

看到sql_insert.cc, sql_update.cc,

sql_select.cc，等等，分别实现了对应的SQL命令。后面我们还要经常提到这个目录下的文件。

大概有如下及部分:

SQL解析器代码: sql_lex.cc, sql_yacc.yy, sql_yacc.cc, sql_parse.cc等，实现了对SQL语句的解析操作。

"handler"代码: handle.cc, handler.h，定义了存储引擎的接口。

"item"代码：item_func.cc, item_create.cc，定义了SQL解析后的各个部分。

SQL语句执行代码: sql_update.cc, sql_insert.cc sql_select.cc, sql_show.cc,

sql_load.cc，执行SQL对应的语句。当你要看"SELECT ..."的执行的时候，直接到sql_select.cc去看就OK了。

辅助代码: net_serv.cc实现网络操作

还有其他很多代码。

vio

封装了virtual IO接口，主要是封装了各种协议的网络操作。

plugin

插件的目录，目前有一个全文搜索插件(只能用在myisam存储引擎)。

libmysqld

Mysql连接库源代码。 开源函数库目录 ，和所有的开源项目一样，Mysql也使用了一些开源的库，在其代码库中我们能看到dbug、pstack、strings、 zlib等。

多说无益，主要是对于mysql的代码目录有个概念，要找的时候也有个方向。万一要找某个东西找不到了就只能grep了...

picked from: http://blog.csdn.net/saylerboxer/article/details/6989222

目录清单

Bdb 伯克利DB表引擎

BUILD 构建工程的脚本

Client 客户端

Cmd-line-utils 命令行工具

Config 构建工程所需的一些文件

Dbug Fred Fish的调试库

Docs 文档文件夹

Extra 一些相对独立的次要的工具

Heap HEAP表引擎

Include 头文件

Innobase INNODB表引擎

Libmysql 动态库

Libmysql_r 为了构建线程安全的libmysql库

Libmysqld 服务器作为一个嵌入式的库

Man 用户手册

Myisam MyISAM表引擎

Myisammrg MyISAM Merge表引擎

Mysql-test mysqld的测试单元

Mysys MySQL的系统库

Ndb Mysql集群

Netware Mysql网络版本相关文件

NEW-RPM 部署时存放RPM

Os2 针对OS/2操作系统的底层函数

Pstack 进行堆栈

Regex 正则表达式库(包括扩展的正则表达式函数)

SCCS 源码控制系统(不是源码的一部分)

Scripts 批量SQL脚本，如初始化库脚本

Server-tools 管理工具

Sql 处理SQL命令；Mysql的核心

Sql-bench Mysql的标准检查程序

Sql-common 一些sql文件夹相关的C文件

SSL 安全套接字层

Strings 字符串函数库

Support-files 用于在不同系统上构建Mysql的文件

Tests 包含Perl和C的测试

Tools

Vio 虚拟I/O库

Zlib 数据压缩库，用于WINDOWS

----------------------------------------------------------------------------------------------------------------

下面给出几个比较重要的目录清单：

文件清单

目录名 文件名 注释

----------------------------------------------------------------------------------------------------------------

Client

get_password.c 命令行输入密码

Mysql.cc MySQL命令行工具

Mysqladmin.cc 数据库weihu

Mysqldump.c 将表的内容以SQL语句输出，即逻辑备份

Mysqlimport.c 文本文件数据导入表中

Mysqlmanager-pwgen.c 密码生成

Mysqlshow.c 显示数据库，表和列

Mysqltest.c 被mysql测试单元使用的测试程序

----------------------------------------------------------------------------------------------------------------

MYSYS

Array.c 动态数组

Charset.c 动态字符集，默认字符集

Charset-def.c 包含客户端使用的字符集

Checksum.c 为内存块计算校验和，用于pack_isam

Default.c 从*.cnf和*.ini文件中查找默认配置项

Default_modify.c 编辑可选项

Errors.c 英文错误文本

Hash.c hash查找、比较、释放函数

List.c 双向链表

Make-conf.c 创建*.conf文件

Md5.c MD5算法

Mf_brkhant.c

Mf_cache.c 打开临时文件,并使用io_cache进行缓存

Mf_driname.c 解析，转换路径名

Mf_fn_ext.c 获取文件名的后缀

Mf_format.c 格式化文件名

Mf_getdate 获取日期：

yyyy-mm-dd hh:mm:ss format

mf_iocache.c 缓存I/O

mf_iocaches.c 多键值缓存

mf_loadpath.c 获取全路径名

mf_pack.c 创建需要的压缩/非压缩文件名

mf_path.c 决定是否程序可以找到文件

mf_qsort.c 快速排序

mf_qsort2.c 快速排序2

mf_radix.c 基数排序

mf_soundex.c 探测算法(EDN NOV 14, 1985)

mf_strip.c 去字符串结尾空格

mf_tempdir.c 临时文件夹的创建、查找、删除

mf_tempfile.c 临时文件的创建

mf_unixpath.c 转化文件名为UNIX风格

mf_util.c 常用函数

mf_wcomp.c 使用通配符比较

mf_wfile.c 通配符查找文件

mulalloc.c 同时分配多个指针

my_access.c 检查文件或路径是否合法

my_aes.c AES加密算法

my_alarm.c 警报相关

my_alloc.c 同时分配临时结果集缓存

my_append.c 一个文件到另一个

my_bit.c 除法使用，位运算

my_bitmap.c 位图

my_chsize.c 填充或截断一个文件

my_clock.c 时钟函数

my_compress.c 压缩

my_copy.c 拷贝文件

my_crc32.c

my_create.c 创建文件

my_delete.c 删除文件

my_div.c 获取文件名

my_dup.c 打开复制文件

my_error.c 错误码

my_file.c

my_fopen.c 打开文件

my_fstream.c 文件流读/写

my_gethostbyname.c 获取主机名

my_gethwaddr.c 获取硬件地址

my_getopt.c 查找生效的选项

my_getsystime.c time of day

my_getwd.c 获取工作目录

my_handler.c

my_init.c 初始化变量和函数

my_largepage.c 获取OS的分页大小

my_lib.c 比较/转化目录名和文件名

my_lock.c 锁住文件

my_lockmem.c 分配一块被锁住的内存

my_lread.c 读取文件到内存

my_lwrite.c 内存写入文件

my_malloc.c 分配内存

my_messnc.c 标准输出上输出消息

my_mkdir.c 创建目录

my_mmap.c 内存映射

my_net.c net函数

my_netware.c Mysql网络版

my_once.c 一次分配，永不free

my_open.c 打开一个文件

my_os2cond.c 操作系统cond的简单实现

my_os2dirsrch.c 模拟Win32目录查询

my_os2dlfcn.c 模拟UNIX动态装载

my_os2file64.c 文件64位设置

my_os2mutex.c 互斥量

my_os2thread.c 线程

my_os2tls.c 线程本地存储

my_port.c

my_pthread.c 线程的封装

my_quick.c 读/写

my_read.c 从文件读bytes

my_realloc.c 重新分配内存

my_redel.c 重命名和删除文件

my_seek.c 查找

my_semaphore.c 信号量

my_sleep.c 睡眠等待

my_static.c 静态变量

my_symlink.c 读取符号链接

my_symlink2.c 2

my_sync.c 同步内存和文件

my_thr_init.c 初始化/分配线程变量

my_wincond.c

my_windac.c WINDOWS NT/2000自主访问控制

my_winsem.c 模拟线程

my_winthread.c 模拟线程

my_write.c 写文件

ptr_cmp.c 字节流比较函数

queue,c 优先级队列

raid2.c 支持RAID

rijndael.c AES加密算法

safemalloc.c 安全的malloc

sha1.c sha1哈希加密算法

string.c 字符串函数

testhash.c 测试哈希函数(独立程序)

test_charset 测试字符集(独立)

thr_lock.c 读写锁

thr_mutex.c 互斥量

thr_rwlock.c 同步读写锁

tree.c 二叉树

typelib.c 字符串中匹配字串

SQL

derror.cc 读取独立于语言的信息文件

Des_key_file.cc 加载DES密钥

Discover.cc frm文件的查找

Field.cc 存储列信息

Filed_conv.cc 拷贝字段信息

Filesort.cc 结果集排序(内存或临时文件)

Frm_crypt.cc get_crypt_from_frm

Gen_lex_hash.cc 查找、排列SQL关键字

Gstream.c GIS

Handler.cc 函数句柄

Hash_filo.cc 静态大小HASH表，

以FIFO方式存储主机名、IP表

Ha_berkeley.cc BDB的句柄

Ha_innodb.cc INNODB句柄

Hostname.cc 根据IP获取hostname

Init.cc 初始化和unireg相关的函数

item.cc  item函数

item_buff.cc item的保存和比较的缓存

item_cmpfunc.cc 比较函数的定义

item_create.cc 创建一个item

item_func.cc 数字函数

item_geofunc.cc 集合函数

item_row.cc 记录项比较

item_strfunc.cc 字符串函数

item_subselect.cc 子查询

item_sum.cc 集函数(SUM,AVG...)

item_timefunc.cc 时间日期函数

item_uniq.cc  空文件

Key.cc 创建KEY以及比较

Lock.cc 锁

Log.cc 日志

log_event.cc 日志事件

Matherr.c 处理溢出

mf_iocache.cc 顺序读写的缓存

Mysqld.cc main，处理信号和连接

mf_decimal.cc decimal类型

my_lock.c

net_serv.cc socket数据包的解析

nt_servc.cc NT服务

opt_range.cc KEY排序

opt_sum.cc 集函数优化

parse_file.cc frm解析

Password.c 密码检查

Procedure.cc

Protocol.cc 数据包打包发送给客户端

protocol_cursor.cc 存储返送数据

Records.cc 读取记录集

repl_failsafe.cc

set_var.cc 设置、读取用户变量

Slave.cc slave节点

Sp.cc 存储过程和存储函数

sp_cache.cc

sp_head.cc

sp_pcontext.cc

sp_rcontext.cc

Spatial.cc 集合函数，点线面

Sql_acl.cc ACL

sql_analyse.cc

sql_base.cc 基础函数

sql_cache.cc 查询缓存

sql_client.cc

sql_crypt.cc 加解密

sql_db.cc 创建、删除DB

sql_delete.cc DELETE语句

sql_derived.cc 派生表

sql_do.cc DO

sql_error.cc  错误和警告

sql_handler.cc

sql_help.cc HELP

sql_insert.cc INSERT

sql_lex.cc 词法分析

sql_list.cc

sql_load.cc LOAD DATA 语句

sql_manager.cc 维护工作

sql_map.cc  内存映射

sql_olap.cc

sql_parse.cc 解析语句

sql_prepare.cc

sql_rename.cc 重命名table名

sql_repl.cc 复制

sql_select.cc SELECT和JOIN优化

sql_show.cc SHOW

sql_state.c 错误号和状态的映射

sql_string.cc

sql_table.cc DROP TABLE、ALTER TABLE

sql_trigger.cc 触发器

sql_udf.cc 用户自定义函数

sql_union.cc UNION操作符

sql_update.cc UPDATE

sql_view.cc 视图

Stacktrace.c 显示堆栈(LINUX/INTEL ONLY)

Strfunc.cc

Table.cc 表元数据获取(FRM)

thr_malloc.cc

Time.cc

Uniques.cc 副本的快速删除

Unireg.cc 创建一个FRM
————————————————
版权声明：本文为CSDN博主「我会笑你一辈子的」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_42713608/article/details/113270820

**主要模块及数据流**
经过多年的发展，mysql的主要模块已经稳定，基本不会有大的修改。本文将对MySQL的整体架构及重要目录进行讲述。

1. 1. 源码结构（MySQL-5.5.0-m2）

      - BUILD: 内含在各个平台、各种编译器下进行编译的脚本。如compile-pentium-debug表示在pentium架构上进行编译的脚本。
      - Client: 客户端工具，如mysql, mysqladmin之类。
      - Cmd-line-utils: readline, libedit工具。
      - Config: 给aclocal使用的配置文件。
      - Dbug: 提供一些调试用的宏定义。
      - Extra: 提供innochecksum，resolveip等额外的小工具。
      - Include: 包含的头文件
      - Libmysql: 库文件，生产libmysqlclient.so。
      - Libmysql_r: 线程安全的库文件，生成libmysqlclient_r.so。
      - Libservices: 5.5.0中新加的目录，实现了打印功能。
      - Man: 手册页。
      - Mysql-test: mysqld的测试工具一套。
      - Mysys: 为跨平台计，MySQL自己实现了一套常用的数据结构和算法，如string, hash等。
      - Netware: 在netware平台上进行编译时需要的工具和库。
      - Plugin: mysql以插件形式实现的部分功能。
      - Pstack: 异步栈追踪工具。
      - Regex: 正则表达式工具。
      - Scripts: 提供脚本工具，如mysql_install_db等
      - Sql: mysql主要代码，将会生成mysqld文件。
      - Sql-bench: 一些评测代码。
      - Sql-common: 存放部分服务器端和客户端都会用到的代码。
      - Storage: 存储引擎所在目录，如myisam, innodb, ndb等。
      - Strings: string库。
      - Support-files: my.cnf示例配置文件。
      - Tests: 测试文件所在目录。
      - Unittest: 单元测试。
      - Vio: virtual io系统，是对network io的封装。
      - Win: 给windows平台提供的编译环境。
      - Zip: zip库工具

   2. 主要数据结构

      - THD 线程描述符(sql/sql_class.h)

      - 包含处理用户请求时需要的相关数据，每个连接会有一个线程来处理，在一些高层函数中，此数据结构常被当作第一个参数传递。
        `NET net; // 客户连接描述符Protocol *protocol; // 当前的协议Protocol_text protocol_text; // 普通协议Protocol_binary protocol_binary; // 二进制协议HASH user_vars; //用户变量的hash值String packet; // 网络IO时所用的缓存String convert_buffer; // 字符集转换所用的缓存struct sockaddr_in remote; //客户端socket地址`

        THR_LOCK_INFO lock_info; // 当前线程的锁信息
        THR_LOCK_OWNER main_lock_id; // 在旧版的查询中使用
        THR_LOCK_OWNER *lock_id; //若非main_lock_id, 指向游标的lock_id
        pthread_mutex_t LOCK_thd_data; //thd的mutex锁，保护THD数据（thd->query, thd->query_length）不会被其余线程访问到。

        Statement_map stmt_map; //prepared statements和stored routines 会被重复利用
        int insert(THD *thd, Statement *statement); // statement的hash容器
        class Statement::
        LEX_STRING name;
        LEX *lex; //语法树描述符
        bool set_db(const char *new_db, size_t new_db_len)
        void set_query(char *query_arg, uint32 query_length_arg);
        {
        pthread_mutex_lock(&LOCK_thd_data);
        set_query_inner(query_arg, query_length_arg);
        pthread_mutex_unlock(&LOCK_thd_data);
        }

      - NET 网络连接描述符（sql/mysql_com.h）

      - 网络连接描述符，对内部数据包进行了封装，是client和server之间的通信协议。
        `Vio *vio; //底层的网络I/O socket描述符unsigned char *buff,*buff_end,*write_pos,*read_pos; //缓存相关unsigned long remain_in_buf,length, buf_length, where_b;unsigned long max_packet,max_packet_size; //当前值;最大值unsigned int pkt_nr,compress_pkt_nr; //当前（未）压缩包的顺序值my_bool compress; //是否压缩unsigned int write_timeout, read_timeout, retry_count; //最大等待时间unsigned int *return_status; //thd中的服务器状态unsigned char reading_or_writing;unsigned int last_errno; //返回给客户端的错误号unsigned char error;`

      - TABLE 数据库表描述符（sql/table.h）

      - 数据库表描述符，分成TABLE和TABLE_SHARE两部分。
        `handler *file; //指向这张表在storage engine中的handler的指针THD *in_use;Field **field;uchar *record[2];uchar *write_row_record;uchar *insert_values;key_map covering_keys;key_map quick_keys, merge_keys;key_map keys_in_use_for_query;key_map keys_in_use_for_group_by;key_map keys_in_use_for_order_by;KEY *key_info;`

        HASH name_hash; //数据域名字的hash值
        MEM_ROOT mem_root; //内存块
        LEX_STRING db;
        LEX_STRING table_name;
        LEX_STRING table_cache_key;
        enum db_type db_type //当前表的storage engine类型
        enum row_type row_type //当前记录是定长还是变长
        uint primary_key;
        uint next_number_index; //自动增长key的值
        bool is_view ;
        bool crashed;

      - FIELD 字段描述符（sql/field.h）

      - 域描述符，是各种字段的抽象基类。
        `uchar *ptr; // 记录中数据域的位置uchar *null_ptr; // 记录 null_bit 位置的byteTABLE *table; // 指向表的指针TABLE *orig_table; // 指向原表的指针const char **table_name, *field_name;LEX_STRING comment;key_map key_start, part_of_key, part_of_key_not_clustered;key_map part_of_sortkey;enum utype { NONE,DATE,SHIELD,NOEMPTY,CASEUP,PNR,BGNR,PGNR,YES,NO,REL,CHECK,EMPTY,UNKNOWN_FIELD,CASEDN,NEXT_NUMBER,INTERVAL_FIELD,BIT_FIELD, TIMESTAMP_OLD_FIELD, CAPITALIZE, BLOB_FIELD,TIMESTAMP_DN_FIELD, TIMESTAMP_UN_FIELD, TIMESTAMP_DNUN_FIELD};…..virtual int store(const char *to, uint length,CHARSET_INFO *cs)=0;inline String *val_str(String *str) { return val_str(str, str); }`

      - Utility API Calls 各种API

      - 各种核心的工具，例如内存分配，字符串操作或文件管理。标准C库中的函数只使用了很少一部分，C++中的函数基本没用。
        `void *my_malloc(size_t size, myf my_flags) //对malloc的封装size_t my_write(File Filedes, const uchar *Buffer, size_t Count, myf MyFlags) //对write的封装`

      - Preprocessor Macros 处理器宏

      - Mysql中使用了大量的C预编译，随编译参数的不同最终代码也不同。
        `#define max(a, b) ((a) > (b) ? (a) : (b)) //得出两数中的大者`

        do 
        { 
        char compile_time_assert[(X) ? 1 : -1] 
        __attribute__ ((unused)); 
        } while(0)
        使用gcc的attribute属性指导编译器行为
        • Global variables 全局变量
        • configuration settings
        • server status information
        • various data structures shared among threads
        主要包括一些全局的设置，服务器信息和部分线程间共享的数据结构。
        `struct system_status_var global_status_var; //全局的状态信息struct system_variables global_system_variables; //全局系统变量`

   3. 主要调用流程

      1. MySQL启动

      2. 主要代码在sql/mysqld.cc中，精简后的代码如下：
         `int main(int argc, char **argv) //标准入口函数MY_INIT(argv[0]); //调用mysys/My_init.c->my_init()，初始化mysql内部的系统库logger.init_base(); //初始化日志功能init_common_variables(MYSQL_CONFIG_NAME,argc, argv, load_default_groups) //调用load_defaults(conf_file_name, groups, &argc, &argv)，读取配置信息user_info= check_user(mysqld_user);//检测启动时的用户选项set_user(mysqld_user, user_info);//设置以该用户运行init_server_components();//初始化内部的一些组件，如table_cache, query_cache等。network_init();//初始化网络模块，创建socket监听start_signal_handler();// 创建pid文件mysql_rm_tmp_tables() || acl_init(opt_noacl)//删除tmp_table并初始化数据库级别的权限。init_status_vars(); // 初始化mysql中的status变量start_handle_manager();//创建manager线程handle_connections_sockets();//主要处理函数，处理新的连接并创建新的线程处理之`

      3. 监听接收链接

      4. 主要代码在sql/mysqld.cc中，精简后的代码如下：
         `THD *thd;FD_SET(ip_sock,&clientFDs); //客户端socketwhile (!abort_loop)readFDs=clientFDs;if (select((int) max_used_connection,&readFDs,0,0,0) error && net->vio != 0 &&!(thd->killed == THD::KILL_CONNECTION)){if(do_command(thd)) //处理客户端发出的命令break;}end_connection(thd);}`

      5. 预处理连接

      6. `thread_count++;//增加当前连接的线程thread_scheduler.add_connection(thd);for (;;) {lex_start(thd);login_connection(thd); // 认证prepare_new_connection_state(thd); //初始化thd描述符while(!net->error && net->vio != 0 &&!(thd->killed == THD::KILL_CONNECTION)){if(do_command(thd)) //处理客户端发出的命令break;}end_connection(thd);}`

      7. 处理

      8. do_command在sql/sql_parse.cc中：读取客户端传递的命令并分发。
         `NET *net= &thd->net;packet_length= my_net_read(net);packet= (char*) net->read_pos;command= (enum enum_server_command) (uchar) packet[0]; //从net结构中获取命令dispatch_command(command, thd, packet+1, (uint) (packet_length-1));//分发命令在dispatch_command函数中，根据命令的类型进行分发。thd->command=command;switch( command ) {case COM_INIT_DB: ...;case COM_TABLE_DUMP: ...;case COM_CHANGE_USER: ...;….case COM_QUERY: //如果是查询语句{alloc_query(thd, packet, packet_length)//thd->set_query(query, packet_length);mysql_parse(thd, thd->query(), thd->query_length(), &end_of_stmt);// 解析查询语句….}`
         在mysql_parse函数中，
         `lex_start(thd);if (query_cache_send_result_to_client(thd, (char*) inBuf, length) sql_command`
         在mysql_execute_command中，根据命令类型，转到相应的执行函数。
         `switch (lex->sql_command) {LEX *lex= thd->lex;TABLE_LIST *all_tables;case SQLCOM_SELECT:check_table_access(thd, lex->exchange ? SELECT_ACL | FILE_ACL : SELECT_ACL, all_tables, UINT_MAX, FALSE); //检查用户权限execute_sqlcom_select(thd, all_tables); //执行select命令break;`

         case SQLCOM_INSERT:
         { res= insert_precheck(thd, all_tables) //rights
         mysql_insert(thd, all_tables, lex->field_list, lex->many_values,
         lex->update_list, lex->value_list,
         lex->duplicates, lex->ignore);
         break;
         在execute_sqlcom_select函数中，
         `res= open_and_lock_tables(thd, all_tables)//directly and indirectlyres= handle_select(thd, lex, result, 0);`
         handle_select在sql_select.cc中，调用mysql_select ，在mysql_select中，
         `join->prepare();//Prepare of whole select (including sub queries in future).join->optimize();//global select optimisation.join->exec();//`
         在mysql_insert函数中，
         `open_and_lock_tables(thd, table_list)mysql_prepare_insert(); //prepare item in INSERT statmentwhile ((values= its++))write_record(thd, table ,&info);//写入新的数据`
         在write_record函数中，
         `table->file->ha_write_row(table->record[0])ha_write_row在Handler.cc中，只是一个接口write_row(buf); //调用表存储所用的引擎`

   4. 当客户端链接上mysql服务端时，系统为其分配一个链接描述符thd，用以描述客户端的所有信息，将作为参数在各个模块之间传递。一个典型的客户端查询在MySQL的主要模块之间的调用关系如下图所示：

      

      当mysql启动完毕后，调用handle_connection_sockets等待客户端连接。当客户端连接上服务器时，服务处理函数将接受连 接，会为其创建链接线程，并进行认证。如认证通过，每个连接线程将会被分配到一个线程描述符thd，可能是新创建的，也可能是从 cached_thread线程池中复用的。该描述符包含了客户端输入的所有信息，如查询语句等。服务器端会层层解析命令，根据命令类型的不同，转到相应 的sql执行函数，进而给传递给下层的存储引擎模块，处理磁盘上的数据库文件，最后将结果返回。执行完毕后thd将被加入cached_thread中。

# mysql源码结构介绍

[![img](https://upload.jianshu.io/users/upload_avatars/12069313/ffd1209f-e78c-443e-aaa6-65585d5946ce.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/07f3d21a61d7)

[ermaot](https://www.jianshu.com/u/07f3d21a61d7)关注

2020.03.11 18:57:37字数 2,031阅读 771

mysql源码非常庞大，直接去看肯定毫无头绪。至少需要知道哪个目录是做什么的，才能有一定的条理。现在对mysql的源码结构做初步介绍

## 目录结构

==来自622463 MySQL运维内参：MySQL、Galera、Inception核心原理与最佳实践==

| 文件夹          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| BUILD           | 里面包含各个平台、各种编译器下进行编译的脚本                 |
| CMakeLists. txt | CMake入口编译文件                                            |
| client          | 客户端工具，所有的客户端工具都在这里，比如 mysql、 mysqlbinlog、 mysqladmin mysqldump等cmake为 CMake编译服务的，这里定义了很多在 CMake编译时使用的方法或变量 |
| cmd-line-utils  | 一些小工具                                                   |
| config.h.cmake  | 用于生成编译时配置头文件的 cmake文件                         |
| dbug            | 提供一些调试用的宏定义，可以很好地跟踪数据库执行到的执行函数、运行栈桢等信息，可以定位一些问题 |
| extra           | 包含了用来做网络消息认证的SSL包，并提供了 comp-err， resolver等一些小工具 |
| include         | MySQL代码包含的所有头文件，这里不包括存储引擎的头文件        |
| libbinlogevents | MySQL57版本开始新增的、用于解析 Binlog的lib服务              |
| libmysql        | 用来创建嵌入式系统的 MySQL客户端程序API                      |
| libmysqld       | MySQL服务器的核心级API文件，也用来开发嵌入式系统             |
| mysql-test      | mysqld的测试工具                                             |
| mysys           | MySQL自己实现的一些常用的数据结构和算法，比如aray、list和hash，以及些区分不同底层操作系统平台的函数封装，比如 my_file、my_fopen等函数，这类型的函数都以my开头 |
| mysys_ssl       | MySQL中SSL相关的服务                                         |
| plugin          | 包括一些系统内置的插件，比如auth、 password_validation等，同时包含了可动态载入的插件，比如 fulltext、 semisync等 |
| regex           | 一些关于正则表达式的算法                                     |
| scripts         | 实现包含一些系统工具脚本，比如 mysql_install_db、 mysqld_safe及 mysql_multi等 |
| sql             | MySQL服务器主要代码，这里包含了main函数（ maIn cc），将会生成 mysqld可执行文件 |
| sql-common      | 存放部分服务器端和客户端都会用到的代码                       |
| storage         | 所有存储引擎的源代码都在这个目录中，文件夹名一般就是其存储引擎的名称，包括 innobase、 myisam、 blackhole、ndb及 perfschema等 |
| strings         | 包含了很多关于字符串处理的函数，比如 strmov、 strapped及 my_atof等函数 |
| support- files  | my.cnf示例配置文件及编译所需的一些工具                       |
| unittest        | 单元测试文件目录                                             |
| vio             | 虚拟网络IO处理系统，是对不同平台或不同协议的网络通信API的封装 |
| win             | 在 windows平台编译所需的文件和一些说明                       |
| zlib            | zlib压缩算法库（GNU）                                        |

## 主要关键目录

1. BUILD
2. client
3. storage
4. mysys
5. sql
6. vio
   本部分来自《MySQL核心内幕-祝定泽》

#### BUILD

- BUILD是编译和安装脚本目录
- 比如目录下的 compile-pentium64-debug 文件，调用了SETUP.sh 和 FINISH.sh



```bash
path=`dirname $0`
set -- "$@" --with-debug=full
. "$path/SETUP.sh"

extra_flags="$pentium64_cflags $debug_cflags"
extra_configs="$pentium_configs $debug_configs $static_link"

extra_configs="$extra_configs "
CC="$CC --pipe"
. "$path/FINISH.sh"
```

#### client

- 在这里,你可以找到亲切熟悉的mysql、mysqladmin、 mysqlshow 等常用命令和客户端工具的源代码

| 大小   | 文件名       | 注释                                      |
| ------ | ------------ | ----------------------------------------- |
| 137966 | mysql.cc     | MySQL客户端                               |
| 37139  | mysqladmin.c | mysqladmin工具,mysqladmin用于服务器的运作 |
| 24631  | mysqlshow.c  | mysqlshow,显示数据库、表和列等            |

- MySQL的绝大部分代码是由C或者C++写成,大部分代码都是以.c、.cc 或h结尾的
- 还包括密码确认功能get_password.c、SSL连接可行性检查等功能

#### storage

- 该目录存放MySQL存储引擎的代码

#### mysys

- mysys代表MySQL system library,是MySQL的库函数文件。库函数是一些预先编译好的函数的集合。这些库文件编译以后在Windows上是.lib 和.dll文件,在Linux和Unix上是.so和.a文件
- mysys目录其实就是个大杂烩,包含了各种各样的功能库文件,包括==文件打开、数据读写、内存分配、OS/2系统特别优化、线程控制、权限控制、Raid Table、动态字符串处理、队列算法、网络传输协议、初始化函数、错误处理、平衡二叉树算法、符号连接处理、唯一临时文件名生成、hash函数、排序算法、压缩传输协议==等

#### sql

- MySQL源代码中需要经常变化的目录之一,另外大部分已有bug也来自于该目录中的文件。20MB左右的代码量,说明它是MySQL服务器内核最为核心和重要的目录
- [线程、查询解析和查询优化器和存储引擎接口)的大部分篇幅也用于描述这个目录中的文件
- 包含mysqld.cc MySQL main函数
  -还包括各类SQL语句的解析/实现

| 文件        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| sql_lex.cc  | 词法解析模块 lex是词法分析程序的自动生成工具。它输入描述构词规则的一系列正则表达式,然后构建有穷自动机和这个有穷自动机的一个驱动程序,进而生成词法分析程序 |
| sql_yacc.yy | 语法解析模块 yacc(Yet Another Compiler Compiler),是Unix/Linux上用来生成编译器的编译器 |

- ==这两个文件决定了MySQL如何解析输入的字符流和SQL语句,并最终获得解析树==
- 存储引擎接口模块的代码/storage下的各存储引擎目录中,存在的是各类存储引擎的实现代码(ha_innodb、ha_myisam等类),而/sql下存放的是处理接口handler,handler中有很多虚函数，需要其子类来实现



```csharp
virtual void change_table_ptr (TABLE *table_arg,TABLE_SHARE *share)
virtual double scan_time ()
virtual double read_time (uint index, uint ranges, ha_rows rows)
virtual double index__only_read_.t ime (uint keynr, double records)
```

- 各种SQL语句的执行代码也可以在sql目录中找到:

| 文件          | 说明          |
| ------------- | ------------- |
| sql_delete.cc | delete语句    |
| sql_do.cc     | 操作语句      |
| sq1_help.cc   | help,帮助语句 |
| sql_insert.cc | 插入语句      |
| sql_select.cc | select语句    |
| sql_.show. cC | show语句      |
| sql_update.cc | update操作    |

这类文件常以sql开始对文件命名。以此类推,insert 语句可查看sql_insert.cc

- MySQL语句的SQL函数代码同样也在sql目录下。MySQL将UNION和ROLLUP等操作,看作内部函数:

| 文件         | 说明                                       |
| ------------ | ------------------------------------------ |
| sql_string.c | 处理string的各种函数                       |
| sql_olap.c   | 处理olap的各种函数,目前对于MySQL只是rollup |

#### vio

VIO意指Virtual I/0,主要用来处理各种网络协议的IO。Virtual I/O使得各种模块的网络协议能够无缝地调用I/O功能.

## mysql执行流

用户执行SELECT语句后,MySQL 的执行流：

1. 第一步,客户端应用程序mysql, 发送了一个SQL语句
2. MySQL 服务器通过vio模块收到了网络传输过来的SQL语句
3. 下一步,Lex和YACC将SQL语句解析并生成语法树,该语法树最终由底层sql目录中sql select.cc执行
4. 独立的handler.cc接受到最后的请求,并执行这个语句
5. Handler 依靠storage目录下的具体存储引擎代码和函数读取数据

## 开源模块

1. dbug：Fred Fish提供，编译时使用with-debug会显示dbug输出
2. pstack：显示进程的stack信息
3. regex：Henry Spencer提供，执行正则匹配函数REGEXP时，会需要
4. strings
5. zlib

## 操作系统相关代码目录

1. netware
2. win

## 源码例解

#### sql_delete.cc

- sql_delete.cc 是delete语句代码，delete和truncate语句会调用（==包括多表删除==（==额外写一篇关于多表删除==））



```php
/*实现MySQL DELETE语句与其他的SQL语句,如INSERT、UPDATE一样,dispatch_command()函数会调用这些函数*/
bool mysql_delete(THD *thd, TABLE_LIST *table_list, COND *conds,SQL_LIST *order, ha_rows limit, ulonglong opt ions,bool reset_auto_increment)
{
bool will_batch;
int error, 1oc_error;
/*


/*下面这个函数为DELETE语句准备Item类.
参数说明:
mysql_prepare_delete()
thd -线程描述符
table_list -全局或本地表的列表
conds -条件参数
*/
int mysq1_prepare_delete(THD *thd, TABLE_LIST *tab1e_list, Item **conds)
Item *fake_conds = 0;
SELECT_LEX *select_1ex = &thd->1ex->select_1ex;
DBUG_ENTER( "mysq1_prepare_delete") ;
List<Item> all_fields; /* 将数据表的所有列放到Item对象的容器*/
if (thd->lex->current_select->select_limit)
tha->lex->set_stmt_unsafe() ; /* 在基于语句复制功能中,delete .... limit语句是不安全的,因为行顺序是未定的*/
thd->set_current_stmt_binlog_row_based_if_mixed(); /* 如果复制功能是基于混合模式,那么我们将delete操作转化为基于行的复制*/
}
```

==这里关于delete 和truncate 额外写一篇==



```dart
/***************************************************
TRUNCATE表
********************************************************************* /
/*
某些存储引擎,如InnoDB的存储引擎,无法支持表重建,故此对这类表的truncate操作是按照一行一行地刪除
*/

static bool mysq1_truncate_by_delete (THD *thd, TABLE LIST *table__list)
bool error, save_binlog_row_based = thd->current_stmt_binlog_row_based;
DBUG_ENTER("mysql_truncate_by_delete") ;
table_list->lock_type= TL_WRITE;
mysq1_init_select (thd->lex) ;
thd->clear_current_stmt_binlog_row_based() ;
error =mysq1_delete(thd,table_list, NULL, NULL,HA_POS_ERROR, LL(0), TRUE);
ha_autocommit_or_rollback(thd, error) ;
end_trans(thd, error ? ROLLBACK : COMMIT); //事务处理
tha->current_stmt_binlog_row_based =save_binlog_row_based; / /基于语句的二进制日志
DBUG_RETURN (error) ;
```

所有行删除操作被优化成自动转化为表重建，即使MYD和MYI表损坏也如此

在下列情况中,经常需要设置dont_ send_ ok:

我们不想发送OK给客户端(don't want an ok to be sent to the end user)

我们不想truncate命令被日志记录下来
*/



```csharp
bool mysq1_truncate(THD *thd, TABLE LIST *table_list, bool dont_send_ok){
HA_CREATE_INFO create_info;
char path[FN_REFLEN]; //记录表在文件系统中的文件地址
TABLE *table;
bool error;
uint path_length;
DBUG_ENTER( "mysql_truncate") ;
bzerol(char*) &create__info, sizeof (create_info));
/*如果是临时表,则关闭这个临时表并重建该表*/
if (!dont_send ok && (table= find temporary_table(thd, table_list))){
handlerton *table_type = table->s->db_type();
TABLE_SHARE *share = table->s;
if (!ha_check_storage_engine_flag(table_type, HTON_CAN_RECREATE) )
goto trunc_by_del;
```

**Mysql源码结构**

目录清单

目录名 注释

Bdb 伯克利DB表引擎

BUILD 构建工程的脚本

Client 客户端

Cmd-line-utils 命令行工具

Config 构建工程所需的一些文件

Dbug Fred Fish的调试库

Docs 文档文件夹

Extra 一些相对独立的次要的工具

Heap HEAP表引擎

Include 头文件

Innobase INNODB表引擎

Libmysql 动态库

Libmysql_r 为了构建线程安全的libmysql库

Libmysqld 服务器作为一个嵌入式的库

Man 用户手册

Myisam MyISAM表引擎

Myisammrg MyISAM Merge表引擎

Mysql-test mysqld的测试单元

Mysys MySQL的系统库

Ndb Mysql集群

Netware Mysql网络版本相关文件

NEW-RPM 部署时存放RPM

Os2 针对OS/2操作系统的底层函数

Pstack 进行堆栈

Regex 正则表达式库（包括扩展的正则表达式函数）

SCCS 源码控制系统（不是源码的一部分）

Scripts 批量SQL脚本，如初始化库脚本

Server-tools 管理工具

Sql 处理SQL命令；Mysql的核心

Sql-bench Mysql的标准检查程序

Sql-common 一些sql文件夹相关的C文件

SSL 安全套接字层

Strings 字符串函数库

Support-files 用于在不同系统上构建Mysql的文件

Tests 包含Perl和C的测试

Tools

Vio 虚拟I/O库

Zlib 数据压缩库，用于WINDOWS

下面给出几个比较重要的目录清单：

文件清单

目录名 文件名 注释

Client

get_password.c 命令行输入密码

Mysql.cc MySQL命令行工具

Mysqladmin.cc 数据库weihu

Mysqldump.c 将表的内容以SQL语句输出，即逻辑备份

Mysqlimport.c 文本文件数据导入表中

Mysqlmanager-pwgen.c 密码生成

Mysqlshow.c 显示数据库，表和列

Mysqltest.c 被mysql测试单元使用的测试程序

\----------------------------------------------------------------------------------------------------------------

MYSYS

Array.c 动态数组

Charset.c 动态字符集，默认字符集

Charset-def.c 包含客户端使用的字符集

Checksum.c 为内存块计算校验和，用于pack_isam

Default.c 从*.cnf和*.ini文件中查找默认配置项

Default_modify.c 编辑可选项

Errors.c 英文错误文本

Hash.c hash查找、比较、释放函数

List.c 双向链表

Make-conf.c 创建*.conf文件

Md5.c MD5算法

Mf_brkhant.c

Mf_cache.c 打开临时文件,并使用io_cache进行缓存

Mf_driname.c 解析，转换路径名

Mf_fn_ext.c 获取文件名的后缀

Mf_format.c 格式化文件名

Mf_getdate 获取日期：

yyyy-mm-dd hh:mm:ss format

mf_iocache.c 缓存I/O

mf_iocaches.c 多键值缓存

mf_loadpath.c 获取全路径名

mf_pack.c 创建需要的压缩/非压缩文件名

mf_path.c 决定是否程序可以找到文件

mf_qsort.c 快速排序

mf_qsort2.c 快速排序2

mf_radix.c 基数排序

mf_soundex.c 探测算法（EDN NOV 14, 1985）

mf_strip.c 去字符串结尾空格

mf_tempdir.c 临时文件夹的创建、查找、删除

mf_tempfile.c 临时文件的创建

mf_unixpath.c 转化文件名为UNIX风格

mf_util.c 常用函数

mf_wcomp.c 使用通配符比较

mf_wfile.c 通配符查找文件

mulalloc.c 同时分配多个指针

my_access.c 检查文件或路径是否合法

my_aes.c AES加密算法

my_alarm.c 警报相关

my_alloc.c 同时分配临时结果集缓存

my_append.c 一个文件到另一个

my_bit.c 除法使用，位运算

my_bitmap.c 位图

my_chsize.c 填充或截断一个文件

my_clock.c 时钟函数

my_compress.c 压缩

my_copy.c 拷贝文件

my_crc32.c

my_create.c 创建文件

my_delete.c 删除文件

my_div.c 获取文件名

my_dup.c 打开复制文件

my_error.c 错误码

my_file.c

my_fopen.c 打开文件

my_fstream.c 文件流读/写

my_gethostbyname.c 获取主机名

my_gethwaddr.c 获取硬件地址

my_getopt.c 查找生效的选项

my_getsystime.c time of day

my_getwd.c 获取工作目录

my_handler.c

my_init.c 初始化变量和函数

my_largepage.c 获取OS的分页大小

my_lib.c 比较/转化目录名和文件名

my_lock.c 锁住文件

my_lockmem.c 分配一块被锁住的内存

my_lread.c 读取文件到内存

my_lwrite.c 内存写入文件

my_malloc.c 分配内存

my_messnc.c 标准输出上输出消息

my_mkdir.c 创建目录

my_mmap.c 内存映射

my_net.c net函数

my_netware.c Mysql网络版
my_once.c 一次分配，永不free

my_open.c 打开一个文件

my_os2cond.c 操作系统cond的简单实现

my_os2dirsrch.c 模拟Win32目录查询

my_os2dlfcn.c 模拟UNIX动态装载

my_os2file64.c 文件64位设置

my_os2mutex.c 互斥量

my_os2thread.c 线程

my_os2tls.c 线程本地存储

my_port.c

my_pthread.c 线程的封装

my_quick.c 读/写

my_read.c 从文件读bytes

my_realloc.c 重新分配内存

my_redel.c 重命名和删除文件

my_seek.c 查找

my_semaphore.c 信号量

my_sleep.c 睡眠等待

my_static.c 静态变量

my_symlink.c 读取符号链接

my_symlink2.c 2

my_sync.c 同步内存和文件

my_thr_init.c 初始化/分配线程变量

my_wincond.c

my_windac.c WINDOWS NT/2000自主访问控制

my_winsem.c 模拟线程

my_winthread.c 模拟线程

my_write.c 写文件

ptr_cmp.c 字节流比较函数

queue,c 优先级队列

raid2.c 支持RAID

rijndael.c AES加密算法

safemalloc.c 安全的malloc

sha1.c sha1哈希加密算法

string.c 字符串函数

testhash.c 测试哈希函数（独立程序）

test_charset 测试字符集(独立)

thr_lock.c 读写锁

thr_mutex.c 互斥量

thr_rwlock.c 同步读写锁

tree.c 二叉树

typelib.c 字符串中匹配字串

SQL
derror.cc 读取独立于语言的信息文件

Des_key_file.cc 加载DES密钥

Discover.cc frm文件的查找

Field.cc 存储列信息

Filed_conv.cc 拷贝字段信息

Filesort.cc 结果集排序（内存或临时文件）

Frm_crypt.cc get_crypt_from_frm

Gen_lex_hash.cc 查找、排列SQL关键字

Gstream.c GIS

Handler.cc 函数句柄

Hash_filo.cc 静态大小HASH表，

以FIFO方式存储主机名、IP表

Ha_berkeley.cc BDB的句柄

Ha_innodb.cc INNODB句柄

Hostname.cc 根据IP获取hostname

Init.cc 初始化和unireg相关的函数

item.cc item函数

item_buff.cc item的保存和比较的缓存

item_cmpfunc.cc 比较函数的定义

item_create.cc 创建一个item

item_func.cc 数字函数

item_geofunc.cc 集合函数

item_row.cc 记录项比较

item_strfunc.cc 字符串函数

item_subselect.cc 子查询

item_sum.cc 集函数（SUM,AVG...）

item_timefunc.cc 时间日期函数

item_uniq.cc 空文件

Key.cc 创建KEY以及比较

Lock.cc 锁

Log.cc 日志

log_event.cc 日志事件

Matherr.c 处理溢出

mf_iocache.cc 顺序读写的缓存

Mysqld.cc main，处理信号和连接

mf_decimal.cc decimal类型

my_lock.c

net_serv.cc socket数据包的解析

nt_servc.cc NT服务

opt_range.cc KEY排序

opt_sum.cc 集函数优化

parse_file.cc frm解析

Password.c 密码检查

Procedure.cc

Protocol.cc 数据包打包发送给客户端

protocol_cursor.cc 存储返送数据

Records.cc 读取记录集

repl_failsafe.cc

set_var.cc 设置、读取用户变量

Slave.cc slave节点

Sp.cc 存储过程和存储函数

sp_cache.cc

sp_head.cc

sp_pcontext.cc

sp_rcontext.cc

Spatial.cc 集合函数，点线面

Sql_acl.cc ACL

sql_analyse.cc

sql_base.cc 基础函数

sql_cache.cc 查询缓存

sql_client.cc

sql_crypt.cc 加解密

sql_db.cc 创建、删除DB

sql_delete.cc DELETE语句

sql_derived.cc 派生表

sql_do.cc DO

sql_error.cc 错误和警告

sql_handler.cc

sql_help.cc HELP

sql_insert.cc INSERT

sql_lex.cc 词法分析

sql_list.cc

sql_load.cc LOAD DATA 语句

sql_manager.cc 维护工作

sql_map.cc 内存映射

sql_olap.cc

sql_parse.cc 解析语句

sql_prepare.cc

sql_rename.cc 重命名table名

sql_repl.cc 复制

sql_select.cc SELECT和JOIN优化

sql_show.cc SHOW

sql_state.c 错误号和状态的映射

sql_string.cc

sql_table.cc DROP TABLE、ALTER TABLE

sql_trigger.cc 触发器

sql_udf.cc 用户自定义函数

sql_union.cc UNION操作符

sql_update.cc UPDATE

sql_view.cc 视图

Stacktrace.c 显示堆栈（LINUX/INTEL ONLY）

Strfunc.cc

Table.cc 表元数据获取（FRM）

thr_malloc.cc

Time.cc

Uniques.cc 副本的快速删除

Unireg.cc 创建一个FRM