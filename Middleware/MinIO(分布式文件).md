本文参考[Minio官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.minio.io%2F)，使用细节里面说的很详细，本文主要讲解文档中较少涉及的Minio存储机制。以及我在使用中部署和使用Java SDK访问的过程。

# 简介

> Minio 是一个基于Apache License v2.0开源协议的`对象存储服务`。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量`非结构化`的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。
>  Minio是一个非常轻量的服务,可以很简单的和其他应用的结合，类似 NodeJS, Redis 或者 MySQL。

# 存储机制

> Minio使用纠删码`erasure code`和校验和`checksum`来保护数据免受硬件故障和无声数据损坏。 即便丢失一半数量（N/2）的硬盘，仍然可以恢复数据。

## 纠删码

> 纠删码是一种恢复丢失和损坏数据的数学算法，目前，纠删码技术在分布式存储系统中的应用主要有三类，阵列纠删码（Array Code: RAID5、RAID6等）、RS(Reed-Solomon)里德-所罗门类纠删码和LDPC(LowDensity Parity Check Code)低密度奇偶校验纠删码。Erasure Code是一种编码技术，它可以将n份原始数据，增加m份数据，并能通过n+m份中的任意n份数据，还原为原始数据。即如果有任意小于等于m份的数据失效，仍然能通过剩下的数据还原出来。

Minio采用Reed-Solomon code将对象拆分成N/2数据和N/2 奇偶校验块。因此下面主要讲解RS类纠删码。

### **RS code编码数据恢复原理：**

RS编码以word为编码和解码单位，大的数据块拆分到字长为w（取值一般为8或者16位）的word，然后对word进行编解码。 数据块的编码原理与word编码原理相同，后文中以word为例说明，变量Di, Ci将代表一个word。
 把输入数据视为向量D=(D1，D2，..., Dn）, 编码后数据视为向量（D1, D2,..., Dn, C1, C2,.., Cm)，RS编码可视为如下（图1）所示矩阵运算。
 图1最左边是编码矩阵（或称为生成矩阵、分布矩阵，Distribution Matrix），编码矩阵需要满足任意n*n子矩阵可逆。为方便数据存储，编码矩阵上部是单位阵（n行n列），下部是m行n列矩阵。下部矩阵可以选择范德蒙德矩阵或柯西矩阵。

![img](https:////upload-images.jianshu.io/upload_images/15296782-683a2bf10cf61f78.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

RS最多能容忍m个数据块被删除。 数据恢复的过程如下：
 （1）假设D1、D4、C2丢失，从编码矩阵中删掉丢失的数据块/编码块对应的行。（图2、3）
 （2）由于B' 是可逆的，记B'的逆矩阵为 (B'^-1)，则B' * (B'^-1) = I 单位矩阵。两边左乘B' 逆矩阵。 （图4、5）
 （3）得到如下原始数据D的计算公式 。

![img](https:////upload-images.jianshu.io/upload_images/15296782-ae5b18d3832ec889.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

（4）对D重新编码，可得到丢失的编码码

### 实践

Minio采用Reed-Solomon code将对象拆分成N/2数据和N/2 奇偶校验块。 这就意味着如果是12块盘，一个对象会被分成6个数据块、6个奇偶校验块，可以丢失任意6块盘（不管其是存放的数据块还是奇偶校验块），仍可以从剩下的盘中的数据进行恢复。

#### 以下是我在4个节点上部署的集群，部署代码如下面多节点部署所示。

1、我在minio服务中上传了一个hello.txt文件，内容为：hello,how are you?\n     共20个字符。
 2、然后在对应的bucket里会生成一个名为hello.txt的文件夹，文件夹里有2个文件，查看4个节点的part.1文件，会发现其中2个文件均分存放了原始数据hello,how are you?\n   ，另外2个文件是乱码，但是根据上述分析应该存的就是奇偶校验块，充当编码矩阵的作用。因此part.1文件是存储就删码的。
 (数据块和奇偶校验块所存储的节点**位置不是固定不变的**)

![img](https:////upload-images.jianshu.io/upload_images/15296782-f9ff94a928524cbd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

3、而另一个json文件中的内容如下，对比了4个节点xl.json文件的差别，发现只有index和checksum中的hash值不同。因此这个json文件是存储校验和的。保护数据免受无声数据损坏。

![img](https:////upload-images.jianshu.io/upload_images/15296782-8ba98729ea929e7b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 模拟数据丢失的情况

![img](https:////upload-images.jianshu.io/upload_images/15296782-0246233c0c982b39.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

新建一个happy.txt文件，上传至文件服务，在服务器的4个节点上可以看见该文件生成了一个part.1和xl.json文件。
 第一步删除005号服务器上的数据块，下载该文件，可读；
 第二步删除004号服务器上的数据块，下载该文件，可读；
 第二步删除003号服务器上的数据块，下载该文件，不可读，得到空白文件；因为丢失的硬盘数量大于N/2，不可恢复健康数据。

#### 模拟checksum丢失的情况

![img](https:////upload-images.jianshu.io/upload_images/15296782-850b09ad246ef395.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

新建一个sing.txt文件，上传至文件服务，在服务器的4个节点上可以看见该文件生成了一个part.1和xl.json文件。
 第一步删除005号服务器上的校验块，下载该文件，可读；
 第二步删除004号服务器上的校验块，下载该文件，可读；
 第二步删除003号服务器上的校验块，下载该文件，不可读，页面报异常，返回页面后该文件已不存在。
 再看服务器中但存储，文件名变为.CORRUPTED后缀。文件被损坏。

![img](https:////upload-images.jianshu.io/upload_images/15296782-4705289379682eab.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

单机Minio服务存在单点故障，相反，如果是一个N节点的分布式Minio,只要有N/2节点在线，你的数据就是安全的。不过你需要至少有N/2+1个节点 [Quorum](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fminio%2Fdsync%23lock-process) 来创建新的对象。
 例如，一个8节点的Minio集群，每个节点一块盘，就算4个节点宕机，这个集群仍然是可读的，不过你需要5个节点才能写数据。

# 部署

## 单节点

**（容器部署）**



```kotlin
docker pull minio/minio
 
#在Docker中运行Minio单点模式
docker run -p 9000:9000 -e MINIO_ACCESS_KEY=sunseaiot -e MINIO_SECRET_KEY=sunseaiot minio/minio server /data
 
#要创建具有永久存储的Minio容器，您需要将本地持久目录从主机操作系统映射到虚拟配置~/.minio 并导出/data目录
#建立外挂文件夹 /Users/hbl/dockersp/volume/minio/
docker run -p 9000:9000 -e MINIO_ACCESS_KEY=sunseaiot -e MINIO_SECRET_KEY=sunseaiot -v /Users/hbl/dockersp/volume/minio/data:/data -v /Users/hbl/dockersp/volume/minio/config:/root/.minio minio/minio server /data
```

## 多节点

> 分布式搭建的流程和单节点基本一样，Minio服务基于命令行传入的参数自动切换成单机模式还是分布式模式。
>
> 分布式Minio单租户存在最少4个盘最多16个盘的限制（受限于纠删码）。这种限制确保了Minio的简洁，同时仍拥有伸缩性。如果你需要搭建一个多租户环境，你可以轻松的使用编排工具（Kubernetes）来管理多个Minio实例。

### 纠删码 (多块硬盘 / 服务)

| 项目           | 参数    |
| -------------- | ------- |
| 最大驱动器数量 | 16      |
| 最小驱动器数量 | 4       |
| 读仲裁         | N / 2   |
| 写仲裁         | N / 2+1 |

**（多节点部署如果要使用容器需要用docker swarm 和K8s，文档中有介绍，本文重点不在此因此我直接在4个服务器上安装了Minio，直接运行即可。`Minio服务基于命令行传入的参数自动切换成单机模式还是分布式模式`，启动一个分布式Minio实例，你只需要把硬盘位置做为参数传给minio server命令即可，然后，你需要在所有其它节点运行同样的命令。）**

### 部署4主机，每机单块磁盘（drive）



```cpp
export MINIO_ACCESS_KEY=123456
export MINIO_SECRET_KEY=123456
./minio server http://192.168.8.110/export1 \
               http://192.168.8.111/export2 \
               http://192.168.8.112/export3 \
               http://192.168.8.113/export4
```

### 部署4主机，每机2块磁盘（drives）



```ruby
export MINIO_ACCESS_KEY=123456
export MINIO_SECRET_KEY=123456
./minio server http://192.168.8.110/export1 http://192.168.1.110/export2 \
               http://192.168.8.111/export1 http://192.168.1.111/export2 \          
               http://192.168.8.112/export1 http://192.168.1.112/export2 \
               http://192.168.8.113/export1 http://192.168.1.113/export2
```

### 后台运行

由于不是用docker部署的，所以需要将进程加入后台运行。使用nohup指令。



```cpp
export MINIO_ACCESS_KEY=SunseaIoT2018!
export MINIO_SECRET_KEY=SunseaIoT2018!
nohup ./minio server http://192.168.8.110/minio1 \
               http://192.168.8.111/minio2 \
               http://192.168.8.112/minio3 \
               http://192.168.8.113/minio4 >  out.file  2>&1  & 
```

# 使用

部署好Minio服务后可以通过浏览器访问。输入设置好的用户名和密码即可进行操作。

![img](https:////upload-images.jianshu.io/upload_images/15296782-38c26808256ee4bb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

# Java SDK访问Minio服务



```swift
package com.minio.client;

import io.minio.MinioClient;
import io.minio.errors.MinioException;
import lombok.extern.slf4j.Slf4j;
import org.xmlpull.v1.XmlPullParserException;

import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

@Slf4j
public class FileUploader {
    public static void main(String[] args) throws NoSuchAlgorithmException, IOException, InvalidKeyException, XmlPullParserException {
        try {
            MinioClient minioClient = new MinioClient("https://minio.sunseaiot.com", "sunseaiot", "sunseaiot",true);

            // 检查存储桶是否已经存在
            if(minioClient.bucketExists("ota")) {
                log.info("Bucket already exists.");
            } else {
                // 创建一个名为ota的存储桶
                minioClient.makeBucket("ota");
                log.info("create a new bucket.");
            }

            //获取下载文件的url，直接点击该url即可在浏览器中下载文件
            String url = minioClient.presignedGetObject("ota","hello.txt");
            log.info(url);

            //获取上传文件的url，这个url可以用Postman工具测试，在body里放入需要上传的文件即可
            String url2 = minioClient.presignedPutObject("ota","ubuntu.tar");
            log.info(url2);

            // 下载文件到本地
            minioClient.getObject("ota","hello.txt", "/Users/hbl/hello2.txt");
            log.info("get");

            // 使用putObject上传一个本地文件到存储桶中。
            minioClient.putObject("ota","tenant2/hello.txt", "/Users/hbl/hello.txt");
            log.info("/Users/hbl/hello.txt is successfully uploaded as hello.txt to `task1` bucket.");
        } catch(MinioException e) {
            log.error("Error occurred: " + e);
        }
    }
}
```

