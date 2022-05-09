# VMware

## 简介

VMware是一个虚拟PC的软件，可以在现有的操作系统中虚拟出一个新的硬件环境，相当于模拟出一台新的PC，以此来实现在一台机器上真正同时运行两个独立的操作系统。

VMware 官网: http://www.vmware.com

特点:

  1.不需要分区或重新开机就能在同一台PC上使用两种以上的操作系统。

  2.本机系统可以与[虚拟机](https://so.csdn.net/so/search?q=虚拟机&spm=1001.2101.3001.7020)系统网络通信。

  3.可以设定并且随时修改虚拟机操作系统的硬件环境。

注：虚拟机占用内存不能超过真实机内存的一半。

快照:把你当前操作系统的状态保存下来，任何时候把操作系统弄坏了，可以使用快照来恢复到记录快照那个时间的状态。

建议VMware配置（真实机）:

  cpu: 建议主频为1GHz以上

  内存: 建议1GB以上（若是1GB，则虚拟机最大分配512MB大小）

  硬盘: 建议分区空闲空间8GB以上

  若在虚拟机中安装操作系统，则虚拟机内存最少需要628MB，即真实机内存至少2GB。

## 虚拟机使用

## 网络配置

### **Bridged（桥接模式）**

桥接模式相当于虚拟机和主机在同一个真实网段，VMWare充当一个集线器功能（一根网线连到主机相连的路由器上），所以如果电脑换了内网，静态分配的ip要更改。图如下：

![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419143437270-462551669.png)

### **NAT（网络地址转换模式）**

NAT模式和桥接模式一样可以上网，只不过，虚拟机会虚拟出一个内网，主机和虚拟机都在这个虚拟的局域网中。NAT中VMWare相当于交换机（产生一个局域网，在这个局域网中分别给主机和虚拟机分配ip地址）

**![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419143720854-1271541753.png)**

步骤：

1.设置VMVare的默认网关（相当于我们设置路由器）: 
编辑->虚拟网络编辑器->更改设置->选中VM8>点击NAT设置，设置默认网关为192.168.182.2。

![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419145040228-1331049611.png)![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419145433514-976861705.png)

2.设置主机ip地址，点击VMnet8，设置ip地址为192.168.182.1，网关为上面设置的网关。

![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419150244761-104929934.png)

![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419145656726-751051047.png)

3.设置linux虚拟机上的网络配置，界面化同上。

### IP配置

#### 未安装系统

 ![img](https://img2020.cnblogs.com/blog/1208477/202012/1208477-20201218135525376-1790028470.png)

 

#### 已安装系统

ifcfg-ens33原文件如下，此时为NAT模式下的DHCP

![img](https://img2020.cnblogs.com/blog/1208477/202010/1208477-20201025175946067-2099571204.png)

 改为静态IP：

如果是NAT要去虚拟网络编辑器中查看NAT设置中的网关IP

```
cd  /etc/sysconfig/network-scripts/     //进入到网络适配器文件夹中
mv ifcfg-ethXXX ifcfg-eth0     //名字改为ifcfg-eth0
vi  ifcfg-eth0    //编辑文件

TYPE=Ethernet 
DEFROUTE=yes 
PEERDNS=yes 
PEERROUTES=yes 
IPV4_FAILURE_FATAL=no 
IPV6INIT=yes 
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes 
IPV6_PEERDNS=yes 
IPV6_PEERROUTES=yes 
IPV6_FAILURE_FATAL=no 
NAME=eth0
#UUID（Universally Unique Identifier）是系统层面的全局唯一标识符号，Mac地址以及IP地址是网络层面的标识号；
#两台不同的Linux系统拥有相同的UUID并不影响系统的使用以及系统之间的通信；
#可以通过命令uuidgen ens33生成新的uuid#和DEVICE一样也可以不写,DEVICE="ens33"可以不写，但一定不要写DEVICE="eth0"
UUID=ae0965e7-22b9-45aa-8ec9-3f0a20a85d11 

ONBOOT=yes  #开启自动启用网络连接,这个一定要改
IPADDR=192.168.182.3  #设置IP地址 
NETMASK=255.255.225.0  #设置子网掩码 
GATEWAY=192.168.182.2  #设置网关 
DNS1=61.147.37.1  #设置主DNS 
DNS2=8.8.8.8  #设置备DNS 
BOOTPROTO=static  #启用静态IP地址 ,默认为dhcp

:wq!  #保存退出 

service network restart  #重启网络，本文环境为centos7

ping www.baidu.com  #测试网络是否正常

ip addr  #查看IP地址
```

改BOOTPROTO和NAME，新增IP网关DNS等配置

![img](https://img2020.cnblogs.com/blog/1208477/202010/1208477-20201025182720952-1754423781.png)

测试下OK

![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419153003040-1182862366.png)

### failed to start lsb:bring up/down networking

ip addr查看mac地址（ link/ether后面的为mac地址），然后在ifcfg-eth0中配置

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0  #修改配置文件

#添加如下配置，这里要写上你的MAC地址
HWADDR=00:0c:bd:05:4e:cc
```

然后关闭NetworkManager

```
systemctl stop NetworkManager
systemctl disable NetworkManager

#重启计算机（推荐）
#systemctl restart network.service
#service network restart
```

### ping通局域网但是ping不通外网

去掉配置NETWORK=yes即可，不知道为啥CentOS7.8加上去之后只能ping的通同一网段的，其他网段的和外网都ping不通。

### 下载ifconfig

ping通网络之后可以下载ifconfig命令

```
yum provides ifconfig    #查看哪个包提供了ifconfig命令,显示net-tools
yum install net-tools    #安装提供ifconfig的包
```

### **Host-Only（仅主机模式）**

主机模式和NAT模式很相似，只不过不能上网，相当于VMware虚拟一个局域网，但是这个局域网没有连互联网。

![img](https://images2018.cnblogs.com/blog/1208477/201804/1208477-20180419144541754-1780033276.png)

参考[文章](https://www.linuxidc.com/Linux/2016-09/135521.htm)

虚拟机安装好后用xshell直接拖拽传递文件的话要执行以下命令

yum install lrzsz

## 快照使用

多重快照功能简介： 

快照的含义：对某一个特定文件系统在某一个特定时间内的一个具有只读属性的镜像。当你需要重复的返回到某一系统状态，又不想创建多个虚拟机的时候，就可以使用快照功能。其实，快照并不是VMware Workstation 5的新功能。早在VMware Workstation 4的时代，就已经支持快照功能了。但是VMware Workstation 4只能生成一个快照，也就是说，VMware Workstation 4创建的虚拟机要只有两个状态：当前状态和快照状态。使用起来还不够灵活。与之相比，VMware Workstation 5及其它升级版本的多重快照功能真的是很强大。 新的多重快照功能，可以针对一台虚拟机创建两个以上的快照，这就意味着我们可以针对不同时刻的系统环境作成多个快照，毫无限制的往返于任何快照之间。与此同时快照管理器，形象的提供了VMware多个快照镜像间的关系。树状的结构使我们能够轻松的浏览和使用生成的快照。那么新的快照功能究竟能给我们带来什么呢？其实，多重快照不只是简单的保存了虚拟机的多个状态，通过建立多个快照，可以为不同的工作保存多个状态，并且不相互影响。例如，当我们在虚拟机上做实验或是作[测试](http://lib.csdn.net/base/softwaretest)时，难免碰到一些不熟悉的地方，此时做个快照，备份一下当前的系统状态，一旦操作错误，可以很快还原到出错前的状态，完成实验，最终避免一步的失误导致重新开始整个实验或测试的后果。 

 

创建快照具体操作步骤： 

1、 启动一个虚拟机，在菜单中单击VM展开Snapshot（快照），单击Take Snapshot... （丛当前状态创建快照） 

2、 在“创建快照”窗口中填入快照的名字和注释，单击“OK”。 

## 克隆

**什么是克隆？** 

说过了快照，那么，什么又是虚拟机的克隆呢？在VMware软件中，克隆和快照功能很相像，但又不同，稍不注意就会混淆。一个虚拟机的克隆就是原始虚拟机全部状态的一个拷贝，或者说一个镜像。克隆的过程并不影响原始虚拟机，克隆的操作一但完成，克隆的虚拟机就可以脱离原始虚拟机独立存在，而且在克隆的虚拟机中和原始虚拟机中的操作是相对独立的，不相互影响。克隆过程中，VMware会生成和原始虚拟机不同的MAC地址和UUID，这就允许克隆的虚拟机和原始虚拟机在同一网络中出现，并且不会产生任何冲突。 VMware支持两种类型的克隆： 完整克隆 链接克隆 

 

一个完整克隆是和原始虚拟机完全独立的一个拷贝，它不和原始虚拟机共享任何资源。 可以脱离原始虚拟机独立使用。 

一个链接克隆需要和原始虚拟机共享同一虚拟磁盘文件，不能脱离原始虚拟机独立运行。但采用共享磁盘文件却大大缩短了创建克隆虚拟机的时间，同时还节省了宝贵的物理磁盘空间。通过链接克隆，可以轻松的为不同的任务创建一个独立的虚拟机。 

 

**创建克隆的虚拟机**： 

1、 打开一个虚拟机，单击“Clone this virtual machine（克隆这个虚拟机）”按钮。 

注意：克隆虚拟机只能在虚拟机未启动的状态下进行。 

2、 在克隆虚拟机创建向导页上，单击“下一步”。 

3、 选择从当前状态或是某一快照创建克隆。 

可以看到，克隆过程既可以按照虚拟机当前的状态来操作，也可以对已经存在的克隆的镜像或快照的镜像来操作。 

4、 在克隆类型选择页面上，可以选择创建的克隆虚拟机的类型“linked clone（联系克隆）”或“full clone（全面克隆）”。一个连接的克隆指向原始的虚拟机，占用很少的磁盘空间，但必须依托于原始的虚拟机，不能够脱离原始虚拟机独立运行。一个完整 

的克隆提供原始虚拟机当前状态的一个副本，可以独立的运行，但是占用很多的磁盘空间。 

此处我们选择“Create a linked clone（创建链接的克隆）”，单击“下一步”。 

5、 在新虚拟机名页面上填入克隆的虚拟机的名称，并确定新虚拟机的安装位置。 

6、 单击完成，完成新克隆的建立。同样的方法，我们可以建立出多个虚拟机的克隆。 

 

**快照与克隆的区别**： 

说了这么多，为了让大家更清晰的理解快照与克隆的区别，我们不妨作一张表，总结一下。 

 

​                  **快照**                            **克隆** 

创建时间：         不限                            虚拟机关机时才可以 

创建数量：         不限                            不限 

占用磁盘空间：      由创建的数量决定                较小 由创建的数量决定，完整克隆较大 

用途：             保存虚拟机某一时刻状态            分发创建的虚拟机 

是否独立：         不能脱离原始虚拟机独立运行        链接克隆：部分脱离 完整克隆：完全脱离 

能否同时使用：      不能                            克隆的虚拟机可以和原始虚拟机同时使用 

是否网络使用：      不能                            生成和原始虚拟机不同的MAC地址和UUID，网络中可以同时使用 

 

**镜像的管理：** 

无论是快照还是克隆，都是对虚拟机的一个状态生成了一个镜像，不同的是这个镜像是作为虚拟机的一部分存在还是作为独立的部分存在。总之，我们可以通过vmware创建多个镜像，用以保存虚拟机不同时期状态。这么多的镜像我们如何管理呢？下面就通过快照管理器来看看我们的成果吧。 

vmware提供了一个管理镜像和快照的快照管理器。在快照管理器中，快照树形象的显示出当前多个快照的层次结构。单击管理其中任何一个镜像，都可以为这个镜像起一个形象的名字，写些必要的注释，还能够删除快照，也能够基于选中的快照创建出一份新的克隆。有了快照管理器，快照的管理也就容易了。 



## Vmware上安装Linux（centos7）

参考资料：https://www.cnblogs.com/yunwangjun-python-520/p/11288690.html

### 准备安装文件（vmware && centos7 镜像）

1、下载 vmware workstations ：链接: https://pan.baidu.com/s/1GscfXnkgzOvVO9889_n8-Q 提取码: a7jm ，也可以自行在网上下载。

然后一路下一步安装就可以了。附激活码：CG54H-D8D0H-H8DHY-C6X7X-N2KG6   如果不行的话，百度下也有很多，找个能用的就OK。

直接在centos 官网上下载就可以，地址：https://www.centos.org/download/，选择minimal iOS直接下载。

### 开始安装centos 7

1、打开vmware workstations，文件->新建虚拟机，出现如下界面，选择“自定义（高级）”选项，下一步继续：

2、此步骤默认，下一步继续：

3、在出现下面界面，选中“稍后安装操作系统”选项，下一步继续：

4、在出现如下界面，客户机操作系统选择“linux”，版本选择“CentOS 7 64位”，下一步继续：

5、出现如下界面，输入自定义虚拟机名称，虚拟机名称随便写，第二个输入框指定虚拟机位置，可以自己选择位置，也可以默认，然后下一步继续：

6、出现下面界面，选择处理器数量和每个处理器核心数量，这里分别是2和4，下一步继续：

7、出现如下界面，指定虚拟机占用内存大小，这里是2048M，下一步继续：

8、出现如下界面，选择网络连接类型，这里选择“使用桥接网络”，各位安装虚拟机过程根据需要自行选择，安装向导中已经针对各种模式进行了比较规范的说明，这里补充说明如下：

1）使用桥接网络：虚拟机ip与本机在同一网段，本机与虚拟机可以通过ip互通，本机联网状态下虚拟机即可联网，同时虚拟机与本网段内其他主机可以互通，这种模式常用于服务器环境架构中。

2）使用网络地址转换（NAT）：虚拟机可以联网，与本机互通，与本机网段内其他主机不通。

3）使用仅主机模式网络：虚拟机不能联网，与本机互通，与本机网段内其他主机不通。

下一步继续：

9、默认，下一步继续：

10、默认、下一步继续：

11、默认，下一步继续：

12、出现下面界面，输入虚拟机磁盘大小，默认20g一般不够使用，建议设置略大一些，这里设置虚拟机磁盘大小为80G，下一步继续：

13、默认，下一步继续：

14、默认、点击“完成”结束虚拟机创建：

15、退出安装向导后，我们可以在虚拟机管理界面左侧栏看到刚刚创建的虚拟机，右侧栏可以看到虚拟机详细配置信息：

16、上图界面中点击“编辑虚拟机设置”选项，出现如下界面：

17、上图中需要指定“CD/DVD(IDE)”安装镜像，移除“USB控制器”和“打印机”等不需要的设备，然后点击确定，按照上述设置后界面如下图所示：

 18、点击开启虚拟机进入CentOS7操作系统安装过程：

19、虚拟机控制台出现界面，鼠标点击虚拟机窗口内，使用上下键选择Install CentOS liunx 7，点击回车键继续：

20、根据提示点击回车键继续：

21、如下界面默认选择English，点击Continue继续：

22、CentOS7安装配置主要界面如下图所示，根据界面展示，这里对以下3个部分配置进行说明：

Localization和software部分不需要进行任何设置，其中需要注意的是sofrware selection选项，这里本次采用默认值（即最小化安装，这种安装的linux系统不包含图形界面）安装，至于其他组件，待后期使用通过yum安装即可。

如上图，system部分需要必须规划配置的是图中红色部分选项，即磁盘分区规划，另外可以在安装过程中修改network & host name选项中修改主机名（默认主机名为localhost.localdomain）。具体配置过程如下：

点击“installation destination”，进入如下界面，选中80g硬盘，下来滚动条到最后，选中“i will configure partitioning”，即自定义磁盘分区，最后点击左上角done进行磁盘分区规划：

23、CentOS7划分磁盘，进入到磁盘划分页面分配好磁盘大小后，点击done完成分区规划，在弹出对话框中点击“accept changs”：

24、完成磁盘规划后，点击下图红框部分，修改操作系统主机名，这里修改为luohua（如第二图所示），然后点击done完成主机名配置，返回主配置界面：

25、在下图中，其实从第24步配置开始我们就可以发现右下角“begin installtion”按钮已经从原本的灰色变成蓝色，这说明已经可以进行操作系统安装工作了，点击“begin installtion”进行操作系统安装过程。

26、在下图用户设置中需要做的仅是修改root用户密码，点击“root password”，设置密码，如果密码安全度不高，比如我这里的密码为“oracle”，那么可能需要点击2次确定才可以。当root密码设置成功再次返回安装界面时我们可以发现之前user setting界面红色警告消失了，对比下面图1和图3：

27、在下图，操作系统安装已经完成，点击reboot重启操作系统。

28、使用root用户登录（即root/oracle），修改IP地址（vi /etc/sysconfig/network-scripts/ifcfg-ens32）：

按字符键“i”进入编辑模式，修改/etc/sysconfig/network-scripts/ifcfg-ens32文件内容如下：

按“esc”键后，输入:wq回车，完成配置文件编辑。

输入：service network restart命令重启网卡，生效刚刚修改ip地址，ping www.baidu.com测试网络连通性。

注：centos 7 已经取消了 ifconfig 的查看IP方式，可以使用命令： ip addr  来查看

好了，到这一步，CentOS7操作系统安装成功了。

随笔 - 96 文章 - 2 评论 - 386 阅读 - 76万

## VMware中CentOS设置静态IP

因为之前搭建的MongoDB分片没有采用副本集，最近现网压力较大，所以准备研究一下，于是在自己电脑的虚拟机中搭建环境，但是发现之前VMware设置的是DHCP，所以每次重新resume后虚拟机中IP都变了，导致之前已经搭建好的mongodb环境老是出问题又要重新搭建很麻烦，所以设置一下静态静态IP，步骤很简单：

**首先关闭VMware的DHCP**：

Edit->Virtual Network Editor

![img](https://images0.cnblogs.com/blog/288950/201308/10182500-b2ce89e6e7d94f929fcc97b6aac01ac9.png)

选择VMnet8，去掉Use local DHCP service to distribute IP address to VMs选项。点击NAT Settings查看一下GATEWAY地址：

![img](https://images0.cnblogs.com/blog/288950/201308/10182802-2f014494e8bb4366a40dcfbc85c1b203.png)

点击OK就可以了。

 

**设置CentOS静态IP：**

涉及到三个配置文件，分别是：

```
/etc/sysconfig/network
/etc/sysconfig/network-scripts/ifcfg-eth0
/etc/resolv.conf
```

 首先修改/etc/sysconfig/network如下：

```
NETWORKING=yes
HOSTNAME=localhost.localdomain
GATEWAY=192.168.129.2
```

指定网关地址。

然后修改/etc/sysconfig/network-scripts/ifcfg-eth0：

```
DEVICE="eth0"
#BOOTPROTO="dhcp"
BOOTPROTO="static"
IPADDR=192.168.129.129
NETMASK=255.255.255.0
HWADDR="00:0C:29:56:8F:AD"
IPV6INIT="no"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID="ba48a4c0-f33d-4e05-98bd-248b01691c20"
DNS1=192.168.129.2
```

**注意**：这里DNS1是必须要设置的否则无法进行域名解析。

最后配置下/etc/resolv.conf：

```
nameserver 192.168.129.2
```

其实这一步可以省掉，上面设置了DNS Server的地址后系统会自动修改这个配置文件。

 

这样很简单几个步骤后虚拟机的IP就一直是192.168.129.129了。