# Shell

Linux Shell常用shell命令

## 文件、目录操作命令

1、ls命令

功能：显示文件和目录的信息

ls　以默认方式显示当前目录文件列表

ls -a 显示所有文件包括隐藏文件

ls -l 显示文件属性，包括大小，日期，符号连接，是否可读写及是否可执行

ls -lh 显示文件的大小，以容易理解的格式印出文件大小 (例如 1K 234M2G)

ls -lt 显示文件，按照修改时间排序

2、cd命令

功能：改名目录

cd dir　切换到当前目录下的dir目录

cd /　切换到根目录

cd ..　切换到到上一级目录

cd ../..　切换到上二级目录

cd ~　切换到用户目录，比如是root用户，则切换到/root下

3、cp命令

功能：copy文件

cp source target　将文件source复制为target

cp /root /source.　将/root下的文件source复制到当前目录

cp –av soure_dir target_dir　将整个目录复制，两目录完全一样

4、rm命令

功能：删除文件或目录

rm file　删除某一个文件

rm -f file 删除时候不进行提示。可以于r参数配合使用

rm -rf dir　删除当前目录下叫dir的整个目录

5、mv命令

功能：将文件移动走，或者改名，在uinx下面没有改名的命令，如果想改名，可以使用该命令

mv source target　将文件source更名为target

6、diff

功能：比较文件内容

diff dir1 dir2　比较目录1与目录2的文件列表是否相同，但不比较文件的实际内容，不同则列出

diff file1 file2　比较文件1与文件2的内容是否相同，如果是文本格式的文件，则将不相同的内容显示，如果是二进制代码则只表示两个文件是不同的

comm file1 file2　比较文件，显示两个文件不相同的内容

7、ln命令

功能：建立链接。windows的快捷方式就是根据链接的原理来做的

ln source_path target_path 硬连接

ln -s source_path target_path 软连接

 

## 查看文件内容命令

cat命令

显示文件的内容，和DOS的type相同

cat file　

2、more命令

功能：分页显示命令

more　file

more命令也可以通过管道符(|)与其他的命令一起使用,例如：

ps ux|more

ls|more

3、tail 命令

功能：显示文件的最后几行

tail -n 100 aaa.txt 显示文件aaa.txt文件的最后100行

4、vi命令

vi file　编辑文件file

vi 原基本使用及命令：

输入命令的方式为先按[ESC]键，然后输入:w(写入文件),:w!(不询问方式写入文件）,:wq保存并退出,:q退出,q!不保存退出

5、touch命令

功能：创建一个空文件

touch aaa.txt 创建一个空文件，文件名为aaa.txt

三、基本系统命令

1、man命令

功能：查看某个命令的帮助，如果你不知道某个命令的用法不懂，可以问他，他知道就回告诉你

例如：

man ls 显示ls命令的帮助内容

2、w命令

功能：显示登录用户的详细信息

例如：

Sarge:~# w

22:06:51 up 43 min, 1 user, load average: 0.00, 0.00, 0.00

USER   TTY    FROM         LOGIN@  IDLE  JCPU  PCPU WHAT

zhoulj  pts/0  10.140.0.109   21:24  0.00s 0.85s 0.09s sshd: zhoulj [priv]

3、who命令

功能：显示登录用户

例如：

Sarge:~# who

zhoulj  pts/0     Mar 13 21:24 (10.140.0.109)

4、last命令

功能：查看最近那些用户登录系统

例如：

Sarge:~# last

zhoulj  pts/0     10.140.0.109   Mon Mar 13 21:24  still logged in  

reboot  system boot 2.6.8-2-386    Mon Mar 13 21:23      (00:43)  

zhoulj  pts/0     10.140.0.105   Sun Mar 12 22:51 - down  (00:00)  

zhoulj  pts/0     10.140.0.105   Sun Mar 12 22:51 - 22:51 (00:00)  

root   tty1                 Sun Mar 12 22:50 - down  (00:01)  

root   tty1                 Sun Mar 12 22:46 - 22:48 (00:02)  

root   tty1                 Sun Mar 12 22:43 - 22:46 (00:02)  

reboot  system boot 2.6.8-2-386    Mon Mar 13 06:34      (-7:-41)  

wtmp begins Mon Mar 13 06:34:11 2006

5、date命令

功能：系统日期设定

date　显示当前日期时间

date -s 20:30:30　设置系统时间为20:30:30

date -s 2002-3-5　设置系统时期为2003-3-5

date -s "060520 06:00:00"　设置系统时期为2006年5月20日6点整。

6、clock命令

功能：时钟设置

clock –r　对系统Bios中读取时间参数

clock –w　将系统时间(如由date设置的时间)写入Bios

7、uname命令

功能：查看系统版本

uname -R　显示操作系统内核的version

例如：

Sarge:~# uname -a

Linux Sarge 2.6.8-2-386 #1 Tue Aug 16 12:46:35 UTC 2005 i686 GNU/Linux

8、关闭和重新启动系统命令

reboot　 重新启动计算机

shutdown -r now 重新启动计算机，停止服务后重新启动计算机

shutdown -h now 关闭计算机，停止服务后再关闭系统

halt  关闭计算机

一般用shutdown -r now,在重启系统是，关闭相关服务，shutdown -h now也是如此。

9、su命令

功能：切换用户

su - 切换到root用户

su - zhoulj 切换到zhoulj用户，

注意：- ，他很关键，使用-，将使用用户的环境变量

四、监视系统状态命令

1、top命令

功能：查看系统cpu、内存等使用情况

2、free命令

功能：查看内存和swap分区使用情况

例如：

Sarge:~# free -tm

​          total    used    free   shared  buffers   cached

Mem:       187      42     145      0      6      16

-/+ buffers/cache:      19     167

Swap:      243      0     243

Total:      430      42     388

3、uptime

功能：现在的时间 ，系统开机运转到现在经过的时间，连线的使用者数量，最近一分钟，五分钟和十五分钟的系统负载

例如：

Sarge:~# uptime

21:54:46 up 31 min, 1 user, load average: 0.00, 0.00, 0.00

4、vmstat命令

功能：监视虚拟内存使用情况

例如：

\# vmstat

procs              memory    swap      io   system      cpu

r b  swpd  free  buff cache  si  so  bi  bo  in  cs us sy id wa

1 0    0 63704  8100 32272  0  0   8   3 103  17 0 1 98 1

5、ps命令

功能：显示进程信息

ps ux 显示当前用户的进程

ps uxwww 显示当前用户的进程的详细信息

ps aux 显示所有用户的进程

ps ef 显示系统所有进程信息

6、kill命令

功能：干掉某个进程，进程号可以通过ps命令得到

kill -9 1001　将进程编号为1001的程序干掉

kill all -9 apache　将所有名字为apapche的程序杀死，kill不是万能的，对僵死的程序则无效。

五、磁盘操作命令

1、df命令

功能：检查文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

参数 功能

-a 列出全部目录

-Ta 列出全部目录，并且显示文件类型

-B 显示块信息

-i 以i节点列出全部目录

-h 按照日常习惯显示（如：1K、100M、20G）

-x [filesystype] 不显示[filesystype]

例如：

\# df -Th

Filesystem  Type  Size Used Avail Use% Mounted on

/dev/sda1   ext3  265M  64M 187M 26% /

tmpfs     tmpfs   94M   0  94M  0% /dev/shm

/dev/sda6   ext3  714M 8.1M 667M  2% /home

/dev/sda8   ext3  956M 215M 691M 24% /usr

/dev/sda7   ext3  714M  57M 619M  9% /var

2、du命令

功能：检测一个目录和（递归地）所有它的子目录中的文件占用的磁盘空间。

参数 功能

-s [dirName] 显示目录占用总空间

-sk [dirName] 显示目录占用总空间，以k为单位

-sb [dirName] 显示目录占用总空间，以b为单位

-sm [dirName] 显示目录占用总空间，以m为单位

-sc [dirName] 显示目录占用总空间，加上目录统计

-sh [dirName] 只统计目录大小

例如：

\# du -sh /etc

1.3M   /etc

3、mount命令

功能：使用mount命令就可在Linux中挂载各种文件系统。

格式：mount -t 设备名 挂载点

（1）、mount /dev/sda1 /mnt/filetest

mount -t vfat /dev/hda /mnt/fatfile

mount -t ntfs /dev/hda /mnt/ntfsfile

mount -t iso9660 /dev/cdrom /mnt/cdrom

mount -o 设备名 挂载点

（2）、使用usb设备

modprobe usb-storage

mkdir /mnt/usb

mount -t auto /dev/sdx1 /mnt/usb

umount /mnt/usb

4、mkswap命令

功能：使用mkswap命令可以创建swap空间，如：

debian:~# mkswap -c /dev/hda4

debian:~# swapon /dev/hda4    #启用新创建的swap空间，停用可使用swapoff命令

5、fdisk命令

功能：对磁盘进行分区

fdisk /dev/xxx 格式化xxx设备(xxx是指磁盘驱动器的名字，例如hdb，sdc)

fdisk -l 显示磁盘的分区表

6、mkfs命令

功能：格式化文件系统，可以指定文件系统的类型，如ext2、ext3、fat、ntfs等

格式1：mkfs.ext3 options /dev/xxx

格式2：mkfs -t ext2 options /dev/xxx

参数  功能

-b 块大小

-i  节点大写

-m  预留管理空间大小

例如：

debian:~#mkfs.ext3 /dev/sdb1

7、e2fsck命令

功能：磁盘检测

e2fsck /dev/hda1　检查/dev/hda1是否有文件系统错误，提示修复方式

e2fsck -p /dev/hda1　检查/dev/hda1是否有错误，如果有则自动修复

e2fsck -y /dev/hda1　检查错误，所有提问均于yes方式执行

e2fsck -c /dev/hda1　检查磁盘是否有坏区

8、tune2fs命令

功能：调整ext2/ext3文件的参数

参数 功能

-l 查看文件系统信息

-c 设置强制自检的挂载次数

-i 设置强制自检的间隔时间，单位天

-m 保留块的百分比

-j 将ext2文件系统转换成ext3格式

\# tune2fs -l /dev/sda1

9、dd命令

功能：功能：把指定的输入文件拷贝到指定的输出文件中，并且在拷贝过程中可以进行格式转换。

跟DOS下的diskcopy命令的作用类似。

dd if=/dev/fd0 of=floppy.img　将软盘的内容复制成一个镜像

dd if=floppy.img of=/dev/fd0　将一个镜像的内容复制到软盘，做驱动盘的时候经常用。

六、用户和组相关命令

1、groupadd命令

功能：添加组

groupadd test1 添加test1组

groupadd -g 1111 test2 添加test2组，组id为1111

2、useradd命令

功能：添加用户

useradd user1 添加用户user1，home为/home/user1，组为user1

useradd -g test1 -m -d /home/test1 test1 添加用户test1，home为/home/test1，组为test1

user list　显示已登陆的用户列表

3、passwd命令

功能：更改用户密码

passwd user1　修改用户user1的密码

passwd -d root　将root用户的密码删除

4、userdel命令

功能：删除用户

userdel user1　删除user1用户

5、chown命令

功能：改变文件或目录的所有者

chown user1 /dir　将/dir目录设置为user1所有

chown -R user1.user1 /dir　将/dir目录下所有文件和目录，设置为user1所有,组为user1。-R递归到下面的每个文件和目录

6、chgrp命令

功能：改变文件或目录的所有组

chgrp user1 /dir　将/dir目录设置为user1所有

7、chmod命令

功能：改变用户的权限

chmod a+x file　将file文件设置为可执行，脚本类文件一定要这样设置一个，否则得用bash file才能执行

chmod 666 file　将文件file设置为可读写

chmod 750 file 将文件file设置为，所有者为完全权限，同组可以读和执行，其他无权限

8、id命令

功能：显示用户的信息，包括uid、gid等

\# id zhoulj

uid=500(zhoulj) gid=500(zhoulj) groups=500(zhoulj)

9、finger命令

功能：显示用的信息

注意：debian下没有该命令。

\# finger zhoulj

Login: zhoulj                  Name:

Directory: /home/zhoulj           Shell: /bin/bash

On since Sun May 21 07:59 (CST) on pts/0 from 192.168.1.4

No mail.

No Plan.

七、压缩命令

1、gzip格式命令

功能：压缩文件，gz格式的

注意：生成的文件会把源文件覆盖

gzip -v 压缩文件，并且显示进度

-d 解压缩

gnuzip -f 解压缩

例如：

\# gzip a.sh

\#ll

-rwxr-xr-x  1 root   root       71 12月 18 21:08 a.sh.gz

\# gzip -d a.sh.gz

\#ll

-rwxr-xr-x  1 root   root       48 12月 18 21:08 a.sh

2、zip格式命令

功能：压缩和解压缩zip命令

zip  

unzip  

例如：

\# zip a.sh.zip a.sh

 adding: a.sh (stored 0%)

\# ll

-rw-r--r--  1 root   root      188 5月 21 10:37 a.sh.zip

\# unzip a.sh.zip

Archive: a.sh.zip

replace a.sh? [y]es, [n]o, [A]ll, [N]one, [r]ename: r

new name: a1.sh

extracting: a1.sh            

\# ll

-rwxr-xr-x  1 root   root       48 12月 18 21:08 a1.sh

3、bzip2根式命令

功能：bzip2格式压缩命令，

注意：生成的文件会把源文件覆盖

bzip2  

bunzip2

例如：

\# bzip2 a.sh

\# ll

-rwxr-xr-x  1 root   root       85 12月 18 21:08 a.sh.bz2

\# bunzip2 a.sh.bz2

\# ll

-rwxr-xr-x  1 root   root       48 12月 18 21:08 a.sh

4、tar命令

功能：归档、压缩等，比较重要，会经常使用。

-cvf  压缩文件或目录

-xvf   解压缩文件或目录

-zcvf  压缩文件或，格式tar.gz

-zxvf  解压缩文件或，格式tar.gz

-zcvf   压缩文件或，格式tgz

-zxvf   解压缩文件或，格式tgz

举例:

\# tar cvf abc.tar *.sh

\# tar xvf abc.tar

\# tar czvf abc.tar.gz *.sh

\# ll

-rw-r--r--  1 root   root     20480 5月 21 10:50 abc.tar

-rw-r--r--  1 root   root      1223 5月 21 10:53 abc.tar.gz

\# tar xzvf abc.tar.gz

 

八、网络相关命令

1、ifconfig命令

功能：显示修改网卡的信息

ifconfig 显示网络信息

ifconfig eth0 显示eth0网络信息

修改网络信息：

ifconfig eth0 192.168.1.1 netmask 255.255.255.0 设置网卡1的地址192.168.1.1，掩码为255.255.255.0

ifconfig eth0:1 192.168.1.2　  捆绑网卡1的第二个地址为192.168.1.2

ifconfig eth0:x 192.168.1.n　  捆绑网卡1的第n个地址为192.168.1.n

例如：

\# ifconfig eth0:1 192.168.1.11

\# ifconfig

eth0    Link encap:Ethernet HWaddr 00:0C:29:06:9C:24 

​      inet addr:192.168.1.5 Bcast:192.168.1.255 Mask:255.255.255.0

​      UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1

​      RX packets:4220 errors:0 dropped:0 overruns:0 frame:0

​      TX packets:3586 errors:0 dropped:0 overruns:0 carrier:0

​      collisions:0 txqueuelen:1000

​      RX bytes:342493 (334.4 Kb) TX bytes:469020 (458.0 Kb)

​      Interrupt:9 Base address:0x1400

eth0:1  Link encap:Ethernet HWaddr 00:0C:29:06:9C:24 

​      inet addr:192.168.1.11 Bcast:192.168.1.255 Mask:255.255.255.0

​      UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1

​      Interrupt:9 Base address:0x1400

2、route命令

功能：显示当前路由设置情况

route 显示当前路由设置情况，比较慢一般不用。

route add -net 10.0.0.0 netmask 255.255.0.0 gw 192.168.1.254 添加静态路由

route del -net 10.0.0.0 netmask 255.255.0.0 gw 192.168.1.254 添加静态路由

route add default gw 192.168.1.1 metric1　  设置192.168.1.1为默认的路由

route del default　    将默认的路由删除

举例：

\# route add -net 10.0.0.0 netmask 255.255.0.0 gw 192.168.1.254

\# netstat -nr

Kernel IP routing table

Destination   Gateway      Genmask      Flags  MSS Window irtt Iface

192.168.1.0   0.0.0.0      255.255.255.0  U      0 0      0 eth0

10.0.0.0     192.168.1.254  255.255.0.0   UG     0 0      0 eth0

169.254.0.0   0.0.0.0      255.255.0.0   U      0 0      0 eth0

0.0.0.0      192.168.1.254  0.0.0.0      UG     0 0      0 eth0

\# route del -net 10.0.0.0 netmask 255.255.0.0 gw 192.168.1.254  

\# netstat -nr

Kernel IP routing table

Destination   Gateway      Genmask      Flags  MSS Window irtt Iface

192.168.1.0   0.0.0.0      255.255.255.0  U      0 0      0 eth0

169.254.0.0   0.0.0.0      255.255.0.0   U      0 0      0 eth0

0.0.0.0      192.168.1.254  0.0.0.0      UG     0 0      0 eth0

3、netstat命令

功能：显示网络状态

netstat -an 查看网络端口信息

netstat -nr 查看路由表信息，比route快多了，

4、启动网络的命令

redhat族的命令:

/etc/init.d/network

debian命令:

/etc/init.d/networking

例如：

/etc/init.d/network stop 停止网络，

/etc/init.d/network start 启动网络，

5、手工修改网络配置

(1)、debian系统

配置文件位置为：/etc/network/interfaces

\# The loopback network interface

auto lo

iface lo inet loopback

\# The primary network interface

auto eth0 eth1

iface eth0 inet static

​     address 10.4.5.6

​     netmask 255.255.255.0

​     network 10.4.5.0

​     broadcast 10.4.5.255

iface eth1 inet static

​     address 219.25.5.60

​     netmask 255.255.255.192

​     network 219.25.5.0

​     broadcast 219.25.5.63

​     gateway 219.25.5.30

修改后保存配置后，运行

/etc/init.d/networking restart

网络配置就改变了

(2)、redhat系统

配置文件位置为：/etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0

BOOTPROTO=static

BROADCAST=192.168.1.255

IPADDR=192.168.1.5

NETMASK=255.255.255.0

NETWORK=192.168.1.0

GATEWAY=192.168.1.254

ONBOOT=yes

TYPE=Ethernet

修改后保存配置后，运行

/etc/init.d/network restart

或者

service network restart

网络配置就改变了。

默认DNS的文件的位置为：/etc/resolv.conf 

\#cat /etc/resolv.conf

search test.com.cn

nameserver 192.168.1.11

6、网络排错

(1)、ping命令

功能：不说了，不知道就用干这行了。

ping

(2)、traceroute命令

功能：路由跟踪

traceroute

traceroute 207.68.173.7

(3)、nslookup命令

功能：域名解析排错

例如：

$ nslookup

Note: nslookup is deprecated and may be removed from future releases.

Consider using the `dig' or `host' programs instead. Run nslookup with

the `-sil[ent]' option to prevent this message from appearing.

\>

Server:      192.168.1.11

Address:     192.168.1.11#53

Non-authoritative answer:

Name:  

Address: 202.118.66.66

\> server 202.118.66.6

Default server: 202.118.66.6

Address: 202.118.66.6#53

\>

Server:      202.118.66.6

Address:     202.118.66.6#53

Non-authoritative answer:  canonical name =

.

Name:  

Address: 202.108.22.5

九、其他命令

1、ssh命令

功能：远程登陆到其他UNIX主机

ssh -l user1 192.168.1.2 使用用户名user1登陆到192.168.1.2

ssh

  使用用户名user1登陆到192.168.1.2

2、scp命令

功能：安全copy

例如：

scp abc.tar.gz

:~ 将本地的abc.tar.gz 复制到 192.168.1.5的user1用户的根(/home/user1)下。

3、telnet命令

功能：登陆到远程主机

例如：

telnet 192.168.1.5