# Docker

**Build safer, share wider, run faster: New updates to our product subscriptions.**

## Docker概述

### 什么是 Docker

Docker 是使用最广泛的开源容器引擎，它彻底释放了计算虚拟化的威力，极大提高了应用的运行效率，降低了云计算资源供应的成本！ 使用 Docker，可以让应用的部署、测试和分发都变得前所未有的高效和轻松！

Docker 使用 Google 公司推出的 Go 语言 进行开发实现，基于 Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得 Docker 技术比虚拟机技术更为轻便、快捷。

### 为什么要用 Docker

① **更高效的利用系统资源**

由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker 对系统资源的利用率更高。

② **更快速的启动时间**

Docker 容器应用，由于直接运行于宿主内核，无需启动完整的操作系统，因此可以做到秒级、甚至毫秒级的启动时间。

③ **一致的运行环境**

Docker 的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性。

④ **持续交付和部署**

使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。一次创建或配置，可以在任意地方正常运行。

⑤ **更轻松的迁移**

Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker 可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。

### Docker 基本组成

① **镜像(Images)**

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

Docker 设计时，充分利用 Union FS 的技术，将其设计为分层存储的架构，Docker 镜像由多层文件系统联合组成。镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

② **容器(Container)**

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为容器存储层。容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。
按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用数据卷（Volume）、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储)发生读写，其性能和稳定性更高。
数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器可以随意删除、重新 run ，数据却不会丢失。

③ **镜像仓库(Registry)**

镜像仓库是一个集中的存储、分发镜像的服务。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。
通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

最常使用的 Registry 公开服务是官方的 Docker Hub，这也是默认的 Registry，并拥有大量的高质量的官方镜像。用户还可以在本地搭建私有 Docker Registry。Docker 官方提供了 Docker Registry 镜像，可以直接使用做为私有 Registry 服务。

| 概念                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |

## Docker 安装

Docker 版本包含 社区版和企业版，我们日常使用社区版足够。

Docker 社区版各个环境的安装参考官方文档：https://docs.docker.com/install/。

后面的学习在 Linux CentOS7 上测试。

### CentOS7 安装步骤

CentOS 安装参考官方文档：https://docs.docker.com/install/linux/docker-ce/centos/

① 卸载旧版本

```
# yum remove docker docker-common docker-selinux
```

② 安装依赖包

```
# yum install -y yum-utils device-mapper-persistent-data lvm2
```

③ 安装 Docker 软件包源

```
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

④ 安装 Docker CE

```
# yum install docker-ce
```

⑤ 启动 Docker 服务

```
# systemctl start docker
```

⑥ 设置开机启动

```
# systemctl enable docker
```

⑦ 验证安装是否成功

```
# docker -v
# docker info
```

### Docker 命令

通过 **--help** 参数可以看到 docker 提供了哪些命令，可以看到 docker 的用法是 **docker [选项] 命令 。**

命令有两种形式，Management Commands 是子命令形式，每个命令下还有子命令；Commands 是直接命令，相当于子命令的简化形式。

```
[root@localhost usr]# docker --help

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  app*        Docker App (Docker Inc., v0.9.1-beta3)
  builder     Manage builds
  buildx*     Docker Buildx (Docker Inc., v0.8.0-docker)
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  scan*       Docker Scan (Docker Inc., v0.17.0)
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

To get more help with docker, check out our guides at https://docs.docker.com/go/guides/

```

继续查看 Management Commands 有哪些子命令，例如查看 image 的子命令。docker image ls 等同于 docker images，docker image pull 等同于 docker pull。

```
[root@localhost usr]# docker image --help

Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.

```

### Docker 和 VM 比较

Docker 工作原理

Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。容器，是一个运行时环境，就是我们前面说到的集装箱。

为什么 Docker 运行速度远大于 VM?

1. Docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

2. Docker利用的是宿主机的内核,而不需要CentOS。因此,当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。从而避免加载操作系统内核这个比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载CentOS,整个新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,因此新建一个docker容器只需要几秒钟。

## Docker Hello World

通过输入docker pull hello-world来拉取hello-world镜像

```
[root@localhost usr]# docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
Digest: sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38
Status: Image is up to date for hello-world:latest
docker.io/library/hello-world:latest
```

这样我们就从仓库拉取到了HelloWorld的镜像，接下来我们来运行一下，通过输入docker run hello-world

```
[root@localhost usr]# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

若是出现了上图的内容则说明hello-world运行成功。

## 镜像管理

### 镜像简介

镜像包含了一个软件的运行环境，是一个不包含Linux内核而又精简的Linux操作系统，一个镜像可以创建N个容器。

镜像是一个分层存储的架构，由多层文件系统联合组成。镜像构建时，会一层层构建，前一层是后一层的基础。

从下载过程中可以看到，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。

```
[root@localhost usr]# docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
a4b007099961: Pull complete 
e2b610d88fd9: Pull complete 
38567843b438: Pull complete 
5fc423bf9558: Pull complete 
aa8241dfe828: Pull complete 
cc662311610e: Pull complete 
9832d1192cf2: Pull complete 
f2aa1710465f: Pull complete 
4a2d5722b8f3: Pull complete 
3a246e8d7cac: Pull complete 
2f834692d7cc: Pull complete 
a37409568022: Pull complete 
Digest: sha256:b2ae0f527005d99bacdf3a220958ed171e1eb0676377174f0323e0a10912408a
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest
```

通过 **docker history <ID/NAME>** 查看镜像中各层内容及大小，每层会对应着 Dockerfile 中的一条指令。

```
[root@localhost usr]# docker history mysql
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
562c9bc24a08   5 days ago   /bin/sh -c #(nop)  CMD ["mysqld"]               0B        
<missing>      5 days ago   /bin/sh -c #(nop)  EXPOSE 3306 33060            0B        
<missing>      5 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      5 days ago   /bin/sh -c ln -s usr/local/bin/docker-entryp…   34B       
<missing>      5 days ago   /bin/sh -c #(nop) COPY file:e9a583a365264f0f…   13.5kB    
<missing>      5 days ago   /bin/sh -c #(nop) COPY dir:2e040acc386ebd23b…   1.12kB    
<missing>      5 days ago   /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B        
<missing>      5 days ago   /bin/sh -c {   echo mysql-community-server m…   384MB     
<missing>      5 days ago   /bin/sh -c echo 'deb [ signed-by=/etc/apt/ke…   97B       
<missing>      5 days ago   /bin/sh -c #(nop)  ENV MYSQL_VERSION=8.0.28-…   0B        
<missing>      5 days ago   /bin/sh -c #(nop)  ENV MYSQL_MAJOR=8.0          0B        
<missing>      5 days ago   /bin/sh -c set -eux;  key='859BE8D7C586F5384…   2.29kB    
<missing>      5 days ago   /bin/sh -c set -eux;  apt-get update;  apt-g…   53.6MB    
<missing>      5 days ago   /bin/sh -c mkdir /docker-entrypoint-initdb.d    0B        
<missing>      5 days ago   /bin/sh -c set -eux;  savedAptMark="$(apt-ma…   4.06MB    
<missing>      5 days ago   /bin/sh -c #(nop)  ENV GOSU_VERSION=1.14        0B        
<missing>      5 days ago   /bin/sh -c apt-get update && apt-get install…   9.34MB    
<missing>      5 days ago   /bin/sh -c groupadd -r mysql && useradd -r -…   329kB     
<missing>      6 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      6 days ago   /bin/sh -c #(nop) ADD file:7f5787c324936e09d…   69.3MB  
```

由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

```
[root@localhost usr]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        latest    562c9bc24a08   5 days ago   521MB

```

### 镜像原理

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

类似于花卷这种常见的早餐,文件系统可以通过一层一层的嵌套,对外暴露统一的"表面层"来供使用者操作

> 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

**平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？**

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。

### 镜像管理

Docker 运行容器前需要本地存在对应的镜像，如果镜像不存在本地，Docker 会从镜像仓库下载，默认是 Docker Hub 公共注册服务器中的仓库。

Docker Hub 是由 Docker 公司负责维护的公共注册中心，包含大量的优质容器镜像，Docker 工具默认从这个公共镜像库下载镜像。下载的镜像如何使用可以参考官方文档。

地址：https://hub.docker.com/explore

如果从 Docker Hub 下载镜像非常缓慢，可以先配置镜像加速器，

参考：https://www.daocloud.io/mirror

Linux下通过以下命令配置镜像站：

```
# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

# systemctl restart docker
```

#### ① 搜索镜像 

**docker search <NAME> [选项]**

```
[root@localhost usr]# docker search mysql
NAME                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                            MySQL is a widely used, open-source relation…   12289     [OK]       
mariadb                          MariaDB Server is a high performing open sou…   4726      [OK]       
mysql/mysql-server               Optimized MySQL Server Docker images. Create…   911                  [OK]
percona                          Percona Server is a fork of the MySQL relati…   572       [OK]       
phpmyadmin                       phpMyAdmin - A web interface for MySQL and M…   476       [OK]       
mysql/mysql-cluster              Experimental MySQL Cluster Docker images. Cr…   93                   
centos/mysql-57-centos7          MySQL 5.7 SQL database server                   92                   
bitnami/mysql                    Bitnami MySQL Docker Image                      67                   [OK]
databack/mysql-backup            Back up mysql databases to... anywhere!         58                   
ubuntu/mysql                     MySQL open source fast, stable, multi-thread…   25                   
circleci/mysql                   MySQL is a widely used, open-source relation…   25                   
mysql/mysql-router               MySQL Router provides transparent routing be…   23                   
centos/mysql-56-centos7          MySQL 5.6 SQL database server                   22                   
google/mysql                     MySQL server for Google Compute Engine          20                   [OK]
vmware/harbor-db                 Mysql container for Harbor                      10                   
mysqlboy/mydumper                mydumper for mysql logcial backups              3                    
mysqlboy/docker-mydumper         docker-mydumper containerizes MySQL logical …   3                    
bitnami/mysqld-exporter                                                          2                    
ibmcom/mysql-s390x               Docker image for mysql-s390x                    1                    
mirantis/mysql                                                                   0                    
mysqlboy/elasticsearch                                                           0                    
mysqleatmydata/mysql-eatmydata                                                   0                    
ibmcom/tidb-ppc64le              TiDB is a distributed NewSQL database compat…   0                    
mysql/mysql-operator             MySQL Operator for Kubernetes                   0                    
cimg/mysql                                                                       0     
```

各个选项说明:

- **NAME:** 镜像仓库源的名称
- **DESCRIPTION:** 镜像的描述
- **OFFICIAL:** 是否 docker 官方发布
- **stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思。
- **AUTOMATED:** 自动构建。 



#### ② 下载镜像 

**docker pull [选项] [Docker Registry地址]<仓库名>:<标签>**

Docker Registry地址：地址的格式一般是 <域名/IP>[:端口号] 。默认地址是Docker Hub。

仓库名：仓库名是两段式名称，既 <用户名>/<软件名> 。对于 Docker Hub，如果不给出用户名，则默认为 library ，也就是官方镜像。

```
[root@localhost usr]# docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
Digest: sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38
Status: Image is up to date for hello-world:latest
docker.io/library/hello-world:latest

```

#### ③ 列出本地镜像 

**docker images [选项]**

```
[root@localhost usr]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         latest    562c9bc24a08   5 days ago     521MB
hello-world   latest    feb5d9fea6a5   6 months ago   13.3kB
```

各个选项说明:

- **REPOSITORY：**表示镜像的仓库源
- **TAG：**镜像的标签
- **IMAGE ID：**镜像ID
- **CREATED：**镜像创建时间
- **SIZE：**镜像大小



同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如 ubuntu 仓库源里，有 15.10、14.04 等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。

所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下：

```
[root@localhost usr]# docker run -t -i ubuntu:15.10 /bin/bash 
```

参数说明：

- **-i**: 交互式操作。
- **-t**: 终端。
- **ubuntu:15.10**: 这是指用 ubuntu 15.10 版本镜像为基础来启动容器。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像。



查看虚悬镜像（镜像既没有仓库名，也没有标签，显示为 <none>）：**docker images -f dangling=true**

默认的 docker images 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像：**docker images -a**

只列出镜像ID：**docker images -q**

列出部分镜像：**docker images redis** 

以特定格式显示：**docker images --format "{{.ID}}: {{.Repository}}"**

#### ④ 给镜像打 Tag 

**docker tag <IMAGE ID> [<\**用户名\**>/]<镜像名>:<标签>**

镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。

我们可以使用 docker tag 命令，为镜像添加一个新的标签。

```
[root@localhost usr]# docker tag 860c279d2fec root/centos:dev
```

docker tag 镜像ID，这里是 860c279d2fec ,用户名称、镜像源名(repository name)和新的标签名(tag)。

使用 docker images 命令可以看到，ID为860c279d2fec的镜像多一个标签。

#### ⑤ 删除本地镜像 

**docker rmi [选项] <镜像1> [<镜像2> ...]**

<镜像> 可以是镜像短 ID、镜像长 ID、镜像名或者镜像摘要。

删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的Untagged 的信息。

镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。镜像的多层结构让镜像复用变得非常容易，因此很有可能某个其它镜像正依赖于当前镜像的某一层。这种情况，不会触发删除该层的行为。直到没有任何层依赖当前层时，才会真实的删除当前层。

除了镜像依赖以外，还需要注意的是容器对镜像的依赖。如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。如果这些容器是不需要的，应该先将它们删除，然后再来删除镜像。

#### ⑥ 批量删除镜像

删除所有虚悬镜像：**docker rmi $(docker images -q -f dangling=true)**

删除所有仓库名为 redis 的镜像：**docker rmi $(docker images -q redis)**

删除所有在 mysql:8.0 之前的镜像：**docker rmi $(docker images -q -f before=mysql:8.0)**

#### ⑦ 导出镜像

**docker save -o <镜像文件> <镜像>**

可以将一个镜像完整的导出，就可以传输到其它地方去使用。

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190910000642896-1030526149.png)

#### ⑧ 导入镜像

**docker load -i <镜像文件>**

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190910000534924-1667611011.png)

####  ⑨更新镜像

更新镜像之前，我们需要使用镜像来创建一个容器。

更新镜像之前，我们需要使用镜像来创建一个容器。

```
[root@localhost usr]# docker run -t -i ubuntu:15.10 /bin/bash
```

在运行的容器内使用 **apt-get update** 命令进行更新。

在完成操作之后，输入 exit 命令来退出这个容器。

此时 ID 为 e218edb10161 的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit 来提交容器副本。

```
[root@localhost usr]# docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
```

各个参数说明：

- **-m:** 提交的描述信息
- **-a:** 指定镜像作者
- **e218edb10161：**容器 ID
- **runoob/ubuntu:v2:** 指定要创建的目标镜像名

#### ⑩构建镜像

我们使用命令 **docker build** ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。

第一条FROM，指定使用哪个镜像源

RUN 指令告诉docker 在镜像内执行命令，安装了什么。。。

然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。

## 容器管理

### 创建容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（stopped）的容器重新启动。Docker 容器非常轻量级，很多时候可以随时删除和新创建容器。

创建容器的主要命令为 **docker run (或 docker container run)**，常用参数如下：

```
[root@localhost usr]# docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cgroupns string                Cgroup namespace to use (host|private)
                                       'host':    Run the container in the Docker host's cgroup namespace
                                       'private': Run the container in its own private cgroup namespace
                                       '':        Use the cgroup namespace as configured by the
                                                  default-cgroupns-mode option on the daemon (default)
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --pull string                    Pull image before running ("always"|"missing"|"never") (default "missing")
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container

```

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190923230033784-1475011422.png)

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

① 创建并进入容器：**docker run -ti <IMAGE ID OR NAME> /bin/bash**

创建容器，通过 **-ti** 参数分配一个 bash 终端，并进入容器内执行命令。退出容器时容器就被终止了。

```
[root@localhost usr]# docker run -ti mysql /bin/bash
root@1f4533013531:/# ls
bin  boot  dev	docker-entrypoint-initdb.d  entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@1f4533013531:/# exit
exit
[root@localhost usr]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                      PORTS     NAMES
1f4533013531   mysql     "docker-entrypoint.s…"   43 seconds ago   Exited (0) 18 seconds ago             epic_lichterman


```

② 容器后台运行：**docker run -d <IMAGE ID OR NAME>**

```
[root@localhost usr]# docker run -d mysql
2f40507d0192409e7b0f0fe15b7862938c8814103c611740abe35c5f36b5f608
[root@localhost usr]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                      PORTS     NAMES
2f40507d0192   mysql     "docker-entrypoint.s…"   9 seconds ago        Exited (1) 8 seconds ago              blissful_sinoussi
1f4533013531   mysql     "docker-entrypoint.s…"   About a minute ago   Exited (0) 54 seconds ago             epic_lichterman

```

容器是否会长久运行，是和 docker run 指定的命令有关，即容器内是否有前台进程在运行，和 -d 参数无关。一个容器必须有一个进程来守护容器才会在后台长久运行。

例如通过 -d 参数后台运行 centos，容器创建完成后将退出。而通过 -ti 分配一个伪终端后，容器内将有一个 bash 终端守护容器，容器不会退出。

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190923233721960-1995535823.png)

③ 进入容器：**docker exec -ti <CONTAINER ID> bash**

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190923234704353-500720325.png)

④ 指定端口：**docker run -d -p <宿主机端口>:<容器内端口> <IMAGE ID>**

通过 -p 参数指定宿主机与容器内的端口映射，如果不指定宿主机端口，将会随机映射一个宿主机端口。映射好端口后，宿主机外部就可以通过该端口访问容器了。

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190923235027844-222049269.png)

⑤ 宿主机重启时自动重启容器：**docker run -d --restart always <IMAGE ID>** 

宿主机重启时，docker 容器默认不会自动启动，可以通过 --restart=always 设置自动重启。

### 容器资源限制

容器资源限制主要是对内存的限制和使用CPU数量的限制，常用选项如下：

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191002145017354-1453708331.png)

① 使用例子

1) 限制容器最多使用500M内存和100M的Swap，并禁用OOM Killer。注意 --memory-swap 的值为 --memory 的值加上 Swap 的值。

docker run -d --name nginx_1 --memory="500m" --memory-swap="600m" --oom-kill-disable nginx

如果 memory 小于等于 memory-swap，表示不使用Swap；如果 memory-swap="-1" 表示可以无限使用Swap；如果不设置 memory-swap，其值默认为 memory 的2倍。

2) 允许容器最多使用一个半的CPU：docker run -d --name nginx_2 --cpus="1.5" nginx

3) 允许容器最多使用 50% 的CPU：docker run -d --name nginx_3 --cpus=".5" nginx

② 查看容器资源使用统计：**docker stats <CONTAINER ID>**

通过 docker stats 可以实时查看容器资源使用情况。如果不对容器内存或CPU加以限制，将默认使用物理机所有内存和CPU。通常情况下，为了安全一定要限制容器内存和CPU，否则容器内程序遭到攻击，可能无限占用物理机的内存和CPU。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191002153256322-386222621.png)



### 容器常用命令

① 停止容器：**docker stop <CONTAINER ID>**

② 重启容器：**docker start \**<CONTAINER ID>\****

③ 删除容器

删除已停止的容器：**docker rm <CONTAINER ID>**

强制删除运行中的容器：**docker rm -f <CONTAINER ID>**

批量删除Exited(0)状态的容器：**docker rm $(docker ps -aq -f exited=0)**

批量删除所有容器：**docker rm -f $(docker ps -aq)**

④ 查看容器日志：**docker logs <CONTAINER ID OR NAMES>**

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190923235530512-618197819.png)

⑤ 查看容器详细信息：**docker inspect <CONTAINER ID>**

⑥ 列出或指定容器端口映射：**docker port <CONTAINER ID>**

⑦ 显示一个容器运行的进程：**docker top <CONTAINER ID>**

⑧ 拷贝文件/文件夹到容器：**docker cp <dir> <CONTAINER ID>:<dir>**

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191002161821752-1503549770.png)

⑨ 查看容器与镜像的差异：**docker diff <CONTAINER ID>**

![img](https://img2018.cnblogs.com/blog/856154/201909/856154-20190923224554682-2090054521.png)

## 数据卷

### 简介

需求:

- Docker 可以将运行的环境打包形成容器运行，但是我们对 Docker 容器的数据的要求希望是持久化的
- 容器之间希望共享数据

Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来， 那么当容器删除后，数据自然也就没有了。

为了能保存数据在docker中我们使用数据卷。

类似 Redis里面的rdb和aof文件或者我们平时使用的移动硬盘

容器删除后，里面的数据将一同被删除，因此需要将容器内经常变动的数据存储在容器之外，这样删除容器之后数据依然存在。

### 数据卷的用处

数据卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：

数据卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止

Docker 提供三种方式将数据从宿主机挂载到容器中：

- **volumes**：由 Docker 管理的数据卷，是宿主机文件系统的一部分(/var/lib/docker/volumes)。是保存数据的最佳方式。
- **bind mounts**：将宿主机上的文件或目录挂载到容器中，通常在容器需要使用宿主机上的目录或文件时使用，比如搜集宿主机的信息、挂载宿主机上的 maven 仓库等。
- **tmpfs**：挂载存储在主机系统的内存中，而不会写入主机的文件系统。如果不希望将数据持久存储在任何位置，可以使用 tmpfs，同时避免写入容器可写层提高性能。这种方式使用比较少。



### Volume

① 管理卷

创建数据卷：**docker volume create <volume_name>**

查看数据卷列表：**docker volume ls**

查看数据卷详细信息：**docker volume inspect <volume_name>**

创建的数据卷默认在宿主机的 /var/lib/docker/volumes/ 下

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006175104915-175131327.png)

② 创建容器时指定数据卷

\1) 通过 --mount 方式：**--mount src=<数据卷名称>,dst=<容器内的数据目录>，注意逗号之间不能有空格**

docker run -d --name nginx_1 --mount src=nginx_html,dst=/usr/share/nginx/html nginx

\2) 通过 -v 方式：**-v <数据卷名称>:<容器内的数据目录>**

docker run -d --name nginx_2 -v nginx_html:/usr/share/nginx/html nginx

以上两种方式，--mount 更加通用直观，-v 是老语法的方式。如果对应的数据卷不存在，将自动创建数据卷。如果挂载目标在容器中非空，则该目录现有内容会自动同步到数据卷中。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006175210780-649821198.png)

③ 删除数据卷：**docker volume rm <volume_name>**

数据卷是独立于容器的生命周期的，删除容器是不会删除数据卷的，除非删除数据卷，删除数据卷之后数据也就丢失了。

如果数据卷正在被某个容器使用，将不能被删除，需要先删除使用此数据卷的所有容器之后才能删除数据卷。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006173817305-2022720420.png)

④ Volume 特点及使用场景

- 多个容器可以同时挂载相同的卷，可用于多个运行容器之间共享数据。
- 当容器停止或被删除后，数据卷依然存在。当明确删除卷时，卷才会被删除。
- 可以将容器的数据存储在远程主机或其它存储上。
- 将数据从一台 docker 主机迁移到另一台主机时，先停止容器，然后备份卷的目录 /var/lib/docker/volumes



### Bind Mounts

① 创建容器时绑定数据卷

\1) 通过 --mount 方式：**--mount type=bind,src=<宿主机目录>,dst=<容器内的数据目录>，注意逗号之间不能有空格**

docker run -d --name nginx_1 --mount type=bind,src=/data/nginx/html,dst=/usr/share/nginx/html nginx

\2) 通过 -v 方式：**-v <宿主机目录>:\**<容器内的数据目录>\****

docker run -d --name nginx_2 -v /data/nginx/html:/usr/share/nginx/html nginx

以上两种方式，如果源文件/目录不存在，不会自动创建容器，会抛出错误。与 volume 方式相比，如果挂载目标在容器中非空，则该目录现有内容将被隐藏，可以理解成使用宿主机的目录覆盖了容器中的目录，新增的数据会同步。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006174708341-358044453.png)

② Bind Mounts 特点及使用场景

- 从主机共享配置文件到容器。默认情况下，挂载主机 /etc/resolv.conf 到每个容器，提供 DNS 解析。
- 在 docker 主机上的开发环境和容器之间共享源代码。例如，可以将 maven target 目录挂载到容器中，每次在 docker 主机上构建maven项目时，容器都可以访问构建好的项目包。
- 当 docker 主机的文件或目
- 录结构保证与容器所需的绑定挂载一致时，例如容器中需要统计主机的一些信息，可以直接将主机的某些目录直接挂载给容器使用。

## 容器网络

### Docker 网络模式

① bridge：**--net=bridge**

默认网络，docker 启动后会创建一个 docker0 网桥，默认创建的容器会添加到这个网桥中。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006221018676-1826142752.png)

创建容器时不指定网络模式，默认使用 bridge 方式，通过容器网络配置可以看到会默认分配一个 docker0 的内网IP。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006221712371-558945729.png)

② host：**--net=host**

容器不会获得一个独立的 network namespace，而是与主机共用一个。这就意味着容器不会有自己的网卡信息，而是使用宿主机的。容器除了网络，其它都是隔离的。

可以看到 host 网络模式下，容器网络配置与宿主机是一样的。那么容器内应用的端口将占用宿主机的端口。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006222034377-323969518.png)

③ none：**--net=none**

获取独立的 network namespace，但不为容器进行任何网络配置，需要我们手动配置。应用场景比较少。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006222522643-1364582786.png)

④ container：**--net=container:<CONTAINER NAME/ID>**

与指定的容器使用同一个 network namespace，具有同样的网络配置信息，两个容器除了网络，其它都是隔离的。

\1) 创建容器，并映射 9090 端口到容器的 80 端口，进入容器内，通过 netstat -antp 可以看到没有端口连接信息

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006223802000-1773597709.png)

\2) 创建 nginx 容器，并使用 net_container 容器的网络。可以看到 net_container 容器内已经在监听 nginx 的80端口了，而且通过 映射的 9090 端口可以访问到 nginx 服务。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191006224235577-1982026757.png)

⑤ 自定义网络

与默认的bridge原理一样，但自定义网络具备内部DNS发现，可以通过容器名或者主机名进行容器之间网络通信

首先创建一个自定义网络，创建两个容器并加入到自定义网络，在容器中就可以互相连通。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191007123036413-300458064.png)



### 容器网络访问原理

当 Docker 启动时，会自动在主机上创建一个 docker0 虚拟网桥（其上有一个 docker0 内部接口），实际上是Linux 的一个 bridge，可以理解为一个软件交换机。它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。

每次创建一个新容器的时候，Docker 从可用的地址段中选择一个空闲的 IP 地址分配给容器的 eth0 端口。使用本地主机上 docker0 接口的 IP 作为所有容器的默认网关。

当创建一个 Docker 容器的时候，同时会创建一对 veth pair 接口（当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包）。这对接口一端在容器内，即 eth0 ；另一端在本地并被挂载到 docker0 网桥，名称以 veth 开头。通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。Docker 就创建了在主机和所有容器之间的一个虚拟共享网络。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191007124456783-38819526.png)

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191007125216709-1729870172.png)

## 制作镜像

### Dockerfile

镜像是多层存储，每一层是在前一层的基础上进行的修改；Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

我们所使用的镜像基本都是来自于 Docker Hub 的镜像，直接使用这些镜像是可以满足一定的需求，而当这些镜像无法直接满足需求时，我们就需要定制这些镜像。

编写 Dockerfile 时，主要会用到如下一些指令：

- FROM : 基础镜像，当前新镜像是基于哪个镜像的

- MAINTAINER : 镜像维护者的姓名和邮箱地址

- RUN : 容器构建时需要运行的命令

- EXPOSE : 当前容器对外暴露出的端口

- WORKDIR : 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

- ENV : 用来在构建镜像过程中设置环境变量

  ENV MY_PATH /usr/mytest 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样； 也可以在其它指令中直接使用这些环境变量，

  比如：WORKDIR $MY_PATH

- UER : 为RUN CMD ENTRYPOINT 执行命令指定运行用户

- ADD : 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包

- COPY : 类似ADD，拷贝文件和目录到镜像中。 将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

- VOLUME : 容器数据卷，用于数据保存和持久化工作

- CMD : 指定一个容器启动时要运行的命令

  注意: Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

- ENTRYPOINT : 指定一个容器启动时要运行的命令;ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数,但是不会被 docker run 后面的参数替换,而是追加

- ONBUILD : 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

Dockerfile 中每一个指令都会建立一层，在其上执行后面的命令，执行结束后， commit 这一层的修改，构成新的镜像。对于有些指令，尽量将多个指令合并成一个指令，否则容易产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。而且Union FS 是有最大层数限制的，比如 AUFS不得超过 127 层。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

**① FROM**

一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

格式：FROM <镜像名称>[:<TAG>]

Docker 还存在一个特殊的镜像，名为 scratch 。这个镜像并不实际存在，它表示一个空白的镜像。如果以 scratch 为基础镜像的话，意味着不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

**② RUN**

RUN 指令是用来执行命令行命令的。由于命令行的强大能力， RUN 指令在定制镜像时是最常用的指令之一。

多个 RUN 指令尽量合并成一个，使用 && 将各个所需命令串联起来。Dockerfile 支持 Shell 类的行尾添加 \ 的命令换行方式，以及行首 # 进行注释的格式。

格式：

- shell 格式： RUN <命令> ，就像直接在命令行中输入的命令一样。
- exec 格式： RUN ["可执行文件", "参数1", "参数2"] ，这更像是函数调用中的格式。

**③ COPY**

COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。<源路径> 可以是多个，甚至可以是通配符。<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径。

使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。

格式：

- COPY <源路径>,... <目标路径>
- COPY ["<源路径1>",... "<目标路径>"]

**④ ENV**

设置环境变量，无论是后面的其它指令，如 RUN ，还是运行时的应用，都可以直接使用这里定义的环境变量。

格式：

- ENV <key> <value>
- ENV <key1>=<value1> <key2>=<value2>...

**⑤ USER**

USER 用于切换到指定用户，这个用户必须是事先建立好的，否则无法切换。

格式： USER <用户名>

**⑥ EXPOSE**

EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。

格式：EXPOSE <端口1> [<端口2>...]

**⑦ HEALTHCHECK**

HEALTHCHECK 指令是告诉 Docker 应该如何判断容器的状态是否正常。通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

当在一个镜像指定了 HEALTHCHECK 指令后，用其启动容器，初始状态会为 starting ，在 HEALTHCHECK 指令检查成功后变为 healthy ，如果连续一定次数失败，则会变为 unhealthy

格式：

- HEALTHCHECK [选项] CMD <命令> ：设置检查容器健康状况的命令，CMD 命令的返回值决定了该次健康检查的成功与否： 0 - 成功； 1 - 失败；。
- HEALTHCHECK NONE ：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK 支持下列选项：

- --interval=<间隔> ：两次健康检查的间隔，默认为 30 秒；
- --timeout=<时长> ：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
- --retries=<次数> ：当连续失败指定次数后，则将容器状态视为unhealthy ，默认 3 次。

**⑧ WORKDIR**

使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，该目录需要已经存在， WORKDIR 并不会帮你建立目录。

在Dockerfile 中，两行 RUN 命令的执行环境是不同的，是两个完全不同的容器。每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令。

格式：WORKDIR <工作目录路径> 

**⑨ VOLUME**

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

格式：

- VOLUME ["<路径1>", "<路径2>"...]
- VOLUME <路径>

这里的路径会在运行时自动挂载为匿名卷，任何向此路径中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。

**⑩ CMD**

CMD 指令用于指定默认的容器主进程的启动命令。

格式：

- shell 格式： CMD <命令>
- exec 格式： CMD ["可执行文件", "参数1", "参数2"...]

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，跟在镜像名后面的是 command ，运行时会替换 CMD 的默认值；例如：docker run -ti nginx /bin/bash，"/bin/bash" 就是替换命令。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON数组，一定要使用双引号，而不要使用单引号。如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式执行，如：CMD echo $HOME 会被包装为 CMD [ "sh", "-c", "echo $HOME" ]，实际就是一个 shell 进程。

注意：Docker 不是虚拟机，容器中的应用都应该以前台执行。对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出。启动程序时，应要求其以前台形式运行。

**⑪ ENTRYPOINT**

ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。 ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。

格式：

- shell 格式： ENTRYPOINT <命令>
- exec 格式： ENTRYPOINT ["可执行文件", "参数1", "参数2"...]

当指定了 ENTRYPOINT 后， CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令。ENTRYPOINT 中的参数始终会被使用，而 CMD 的额外参数可以在容器启动时动态替换掉。

```
例如：
ENTRYPOINT ["/bin/echo", "hello"]  

容器通过 docker run -ti <image> 启动时，输出为：hello

容器通过 docker run -ti <image> docker 启动时，输出为：hello docker

将Dockerfile修改为：

ENTRYPOINT ["/bin/echo", "hello"]  
CMD ["world"]

容器通过 docker run -ti <image> 启动时，输出为：hello world

容器通过 docker run -ti <image> docker 启动时，输出为：hello docker
```

### Docker执行Dockerfile的大致流程

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

总结:

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

- Dockerfile是软件的原材料
- Docker镜像是软件的交付品
- Docker容器则可以认为是软件的运行态。 Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

### 构建基础镜像

以构建 nginx 基础镜像为例看如何使用 Dockerfile 构建镜像。

① 编写 Dockerfile

```
# 基于 centos:8 构建基础镜像
FROM centos:8
# 作者
MAINTAINER jiangzhou.bo@vip.163.com
# 安装编译依赖的 gcc 等环境，注意最后清除安装缓存以减小镜像体积
RUN yum install -y gcc gcc-c++ make \
    openssl-devel pcre-devel gd-devel \
    iproute net-tools telnet wget curl \
    && yum clean all \
    && rm -rf /var/cache/yum/*
# 编译安装 nginx
RUN wget http://nginx.org/download/nginx-1.17.4.tar.gz \
    && tar zxf nginx-1.17.4.tar.gz \
    && cd nginx-1.17.4 \
    && ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module \
    && make && make install \
    && rm -rf /usr/local/nginx/html/* \
    && echo "hello nginx !" >> /usr/local/nginx/html/index.html \
    && cd / && rm -rf nginx-1.17.4*
# 加入 nginx 环境变量
ENV PATH $PATH:/usr/local/nginx/sbin
# 拷贝上下文目录下的配置文件
COPY ./nginx.conf /usr/local/nginx/conf/nginx.conf
# 设置工作目录
WORKDIR /usr/local/nginx
# 暴露端口
EXPOSE 80
# daemon off: 要求 nginx 以前台进行运行
CMD ["nginx", "-g", "daemon off;"]
```

② 构建镜像

命令：**docker build -t <镜像名称>:<TAG> \**[-f <Dockerfile 路径>]\** <上下文目录>**

- **上下文目录**：镜像构建的上下文。Docker 是 C/S 结构，docker 命令是客户端工具，一切命令都是使用的远程调用形式在服务端(Docker 引擎)完成。当构建的时候，用户指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。这样在 COPY 文件的时候，实际上是复制上下文路径下的文件。一般来说，应该将 Dockerfile 以及所需文件置于一个空目录下，并指定这个空目录为上下文目录。
- Dockerfile 路径：可选，缺省时会寻找当前目录下的 Dockerfile 文件，如果是其它名称的 Dockerfile，可以使用 -f 参数指定文件路径。

在 nginx 目录下包含了构建需要的 Dockerfile 文件及配置文件，docker build 执行时，可以清晰地看到镜像的构建过程。构建成功后，就可以在本地看到已经构建好的镜像。

```
docker build 命令用于使用 Dockerfile 创建镜像。

语法
docker build [OPTIONS] PATH | URL | -
OPTIONS说明：

--build-arg=[] :设置镜像创建时的变量；

--cpu-shares :设置 cpu 使用权重；

--cpu-period :限制 CPU CFS周期；

--cpu-quota :限制 CPU CFS配额；

--cpuset-cpus :指定使用的CPU id；

--cpuset-mems :指定使用的内存 id；

--disable-content-trust :忽略校验，默认开启；

-f :指定要使用的Dockerfile路径；

--force-rm :设置镜像过程中删除中间容器；

--isolation :使用容器隔离技术；

--label=[] :设置镜像使用的元数据；

-m :设置内存最大值；

--memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；

--no-cache :创建镜像的过程不使用缓存；

--pull :尝试去更新镜像的新版本；

--quiet, -q :安静模式，成功后只输出镜像 ID；

--rm :设置镜像成功后删除中间容器；

--shm-size :设置/dev/shm的大小，默认值是64M；

--ulimit :Ulimit配置。

--squash :将 Dockerfile 中所有的操作压缩为一层。

--tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。

--network: 默认 default。在构建期间设置RUN指令的网络模式
```

使用当前目录的 Dockerfile 创建镜像

```
docker build -t mysql .
```

使用URL **github.com/creack/docker-firefox** 的 Dockerfile 创建镜像。

```
docker build github.com/creack/docker-firefox
```

也可以通过 -f Dockerfile 文件的位置：

```
docker build -f Dockerfile .
```

在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：

```
Error response from daemon: dockerfile parse error line 1: unknown instruction: ://GITHUB.COM/ELASTIC/ELASTICSEARCH-DOCKER
```

基于构建的镜像启用容器并测试

③ 构建应用镜像

构件好基础镜像之后，以这个基础镜像来构建应用镜像。

首先编写 Dockerfile：将项目下的 html 目录拷贝到 nginx 目录下

```
FROM bojiangzhou/nginx:v1
COPY ./html /usr/local/nginx/html
```

项目镜像构建成功后，运行镜像，可以看到内容已经被替换了。我们构建好的项目镜像就可以传给其他用户去使用了。

## 镜像仓库

### Habor

镜像仓库是集中存放镜像的地方。目前 Docker 官方维护了一个公共仓库 Docker Hub，大部分需求，都可以通过在 Docker Hub 中直接下载镜像来实现。

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。docker-registry 是官方提供的工具，可以用于构建私有的镜像仓库。

Habor 是由 VMWare 公司开源的容器镜像仓库，Habor 在 docker-registry 上进行了相应的企业级扩展，从而获得了更加广泛的应用。这章学习如何使用 Habor 搭建本地私有仓库。

Harbor 安装及配置要求参考：[Installation and Configuration Guide](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)

Harbor 使用文档参考：[User Guide](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md)

安装 Harbor 前，需确保硬件条件至少 2核4G内存40G的硬盘，本机需已安装 docker、docker-compose、openssl。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020175857146-278088255.png)

### 安装 Docker Compose

Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速在集群中部署分布式应用。使用 Docker Compose 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具。它允许用户通过一个单独的 docker-compose.yml 模板文件来定义一组相关联的应用容器为一个项目。Harbor 也是基于 docker compose 编排部署的。

① 安装 Docker Compose 可以通过下面命令自动下载适应版本的 Compose，并为安装脚本添加执行权限

```
# curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
```

② 检测安装是否成功

```
# docker-compose -v
```

### 安装 Harbor

① 首先从 github 下载二进制包，选择一个最新版本安装：https://github.com/goharbor/harbor/releases

```
# wget https://storage.googleapis.com/harbor-releases/release-1.9.0/harbor-offline-installer-v1.9.1.tgz
```

② 解压

```
# tar zxvf harbor-offline-installer-v1.9.1.tgz
# cd harbor
```

③ 修改配置文件

```
# vim harbor.yml
```

主要修改 hostname 为可访问的域名或IP、修改管理员密码

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020201918076-1520108744.png)

④ 安装

```
# ./prepare
# ./install.sh
```

Harbor 安装成功后，实际上就是通过 docker compose 启动了相关组件的容器

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020202737427-1592916381.png)

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020205806054-458219658.png)

⑤ Harbor 生命周期管理

harbor 基于 docker compose 编排部署，因此可以使用 docker-compose 命令来管理 harbor 的生命周期。

首先切到 harbor 根目录下

停止 harbor：**docker-compose stop**

重启 harbor：**docker-compose start**

修改配置：

```
# #先停止 harbor
# docker-compose down -v
# #修改 harbor.yml
# vim harbor.yml
# #运行 prepare 填充配置
# ./prepare
# #启动harbor
# docker-compose up -d
```

### Harbor 简单使用

① 访问 Harbor

访问配置的 hostname 进入 harbor 管理页面。具体如何使用可参考官方文档：[User Guide](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md)

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020204507210-1425348155.png)

通过新建项目创建一个项目(bojiangzhou)用于测试。公开项目不需登录就可以下载镜像，私有的需要登录；上传镜像则都需要登录才能上传镜像。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020213418601-800905329.png)

 推送镜像的命令格式可以参考如下：

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020213524755-1880208073.png)

② 创建用户

若想上传镜像，需要用户登录，首先在用户管理创建用户：

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020213930603-2047571943.png)

接着在项目中添加用户成员：

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020214048824-640916042.png)

③ 前置配置

上传镜像之前，首先需要将 harbor 仓库配置为 docker 授信的仓库，否则将被拒绝连接：

```
vim /etc/docker/daemon.json
# 配置如下内容
{
    "registry-mirrors": ["http://f1361db2.m.daocloud.io"],
    "insecure-registries": ["192.168.31.54"]
}
```

之后重启 docker 和 harbor：

```
#systemctl restart docker

# cd /docker-test/harbor
# docker-compose up -d
```

④ 镜像仓库登录退出

登录：**docker login registry**

退出：**docker logout \**registry\****

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020214333689-1399842223.png)

⑤ 上传镜像

首先需要给镜像打 Tag：**docker tag SOURCE_IMAGE[:TAG] 192.168.31.54/bojiangzhou/IMAGE[:TAG]**

接着推送镜像：**docker push 192.168.31.54/bojiangzhou/IMAGE[:TAG]**

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020214817968-1820509767.png)

可以看到镜像已经推送到镜像仓库了：

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020214912044-215991698.png)

下载仓库镜像：**docker pull 192.168.31.54/bojiangzhou/nginx-app:1.0.0**

## Docker Compose

### Compose 简介

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

Compose 使用的三个步骤：

- 使用 Dockerfile 定义应用程序的环境。
- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

docker-compose.yml 的配置案例如下（配置参数参考下文）：

```
# yaml 配置实例
version: '3'
services:
  web:
    build: .
    ports:
   - "5000:5000"
    volumes:
   - .:/code
    - logvolume01:/var/log
    links:
   - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

### Compose 安装

Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：https://github.com/docker/compose/releases。

运行以下命令以下载 Docker Compose 的当前稳定版本：

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

### 常用参数

```
    version           # 指定 compose 文件的版本
    services          # 定义所有的 service 信息, services 下面的第一级别的 key 既是一个 service 的名称

        build                 # 指定包含构建上下文的路径, 或作为一个对象，该对象具有 context 和指定的 dockerfile 文件以及 args 参数值
            context               # context: 指定 Dockerfile 文件所在的路径
            dockerfile            # dockerfile: 指定 context 指定的目录下面的 Dockerfile 的名称(默认为 Dockerfile)
            args                  # args: Dockerfile 在 build 过程中需要的参数 (等同于 docker container build --build-arg 的作用)
            cache_from            # v3.2中新增的参数, 指定缓存的镜像列表 (等同于 docker container build --cache_from 的作用)
            labels                # v3.3中新增的参数, 设置镜像的元数据 (等同于 docker container build --labels 的作用)
            shm_size              # v3.5中新增的参数, 设置容器 /dev/shm 分区的大小 (等同于 docker container build --shm-size 的作用)

        command               # 覆盖容器启动后默认执行的命令, 支持 shell 格式和 [] 格式

        configs               # 不知道怎么用

        cgroup_parent         # 不知道怎么用

        container_name        # 指定容器的名称 (等同于 docker run --name 的作用)

        credential_spec       # 不知道怎么用

        deploy                # v3 版本以上, 指定与部署和运行服务相关的配置, deploy 部分是 docker stack 使用的, docker stack 依赖 docker swarm
            endpoint_mode         # v3.3 版本中新增的功能, 指定服务暴露的方式
                vip                   # Docker 为该服务分配了一个虚拟 IP(VIP), 作为客户端的访问服务的地址
                dnsrr                 # DNS轮询, Docker 为该服务设置 DNS 条目, 使得服务名称的 DNS 查询返回一个 IP 地址列表, 客户端直接访问其中的一个地址
            labels                # 指定服务的标签，这些标签仅在服务上设置
            mode                  # 指定 deploy 的模式
                global                # 每个集群节点都只有一个容器
                replicated            # 用户可以指定集群中容器的数量(默认)
            placement             # 不知道怎么用
            replicas              # deploy 的 mode 为 replicated 时, 指定容器副本的数量
            resources             # 资源限制
                limits                # 设置容器的资源限制
                    cpus: "0.5"           # 设置该容器最多只能使用 50% 的 CPU 
                    memory: 50M           # 设置该容器最多只能使用 50M 的内存空间 
                reservations          # 设置为容器预留的系统资源(随时可用)
                    cpus: "0.2"           # 为该容器保留 20% 的 CPU
                    memory: 20M           # 为该容器保留 20M 的内存空间
            restart_policy        # 定义容器重启策略, 用于代替 restart 参数
                condition             # 定义容器重启策略(接受三个参数)
                    none                  # 不尝试重启
                    on-failure            # 只有当容器内部应用程序出现问题才会重启
                    any                   # 无论如何都会尝试重启(默认)
                delay                 # 尝试重启的间隔时间(默认为 0s)
                max_attempts          # 尝试重启次数(默认一直尝试重启)
                window                # 检查重启是否成功之前的等待时间(即如果容器启动了, 隔多少秒之后去检测容器是否正常, 默认 0s)
            update_config         # 用于配置滚动更新配置
                parallelism           # 一次性更新的容器数量
                delay                 # 更新一组容器之间的间隔时间
                failure_action        # 定义更新失败的策略
                    continue              # 继续更新
                    rollback              # 回滚更新
                    pause                 # 暂停更新(默认)
                monitor               # 每次更新后的持续时间以监视更新是否失败(单位: ns|us|ms|s|m|h) (默认为0)
                max_failure_ratio     # 回滚期间容忍的失败率(默认值为0)
                order                 # v3.4 版本中新增的参数, 回滚期间的操作顺序
                    stop-first            #旧任务在启动新任务之前停止(默认)
                    start-first           #首先启动新任务, 并且正在运行的任务暂时重叠
            rollback_config       # v3.7 版本中新增的参数, 用于定义在 update_config 更新失败的回滚策略
                parallelism           # 一次回滚的容器数, 如果设置为0, 则所有容器同时回滚
                delay                 # 每个组回滚之间的时间间隔(默认为0)
                failure_action        # 定义回滚失败的策略
                    continue              # 继续回滚
                    pause                 # 暂停回滚
                monitor               # 每次回滚任务后的持续时间以监视失败(单位: ns|us|ms|s|m|h) (默认为0)
                max_failure_ratio     # 回滚期间容忍的失败率(默认值0)
                order                 # 回滚期间的操作顺序
                    stop-first            # 旧任务在启动新任务之前停止(默认)
                    start-first           # 首先启动新任务, 并且正在运行的任务暂时重叠

            注意：
                支持 docker-compose up 和 docker-compose run 但不支持 docker stack deploy 的子选项
                security_opt  container_name  devices  tmpfs  stop_signal  links    cgroup_parent
                network_mode  external_links  restart  build  userns_mode  sysctls

        devices               # 指定设备映射列表 (等同于 docker run --device 的作用)

        depends_on            # 定义容器启动顺序 (此选项解决了容器之间的依赖关系， 此选项在 v3 版本中 使用 swarm 部署时将忽略该选项)
            示例：
                docker-compose up 以依赖顺序启动服务，下面例子中 redis 和 db 服务在 web 启动前启动
                默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系

                version: '3'
                services:
                    web:
                        build: .
                        depends_on:
                            - db      
                            - redis  
                    redis:
                        image: redis
                    db:
                        image: postgres                             

        dns                   # 设置 DNS 地址(等同于 docker run --dns 的作用)

        dns_search            # 设置 DNS 搜索域(等同于 docker run --dns-search 的作用)

        tmpfs                 # v2 版本以上, 挂载目录到容器中, 作为容器的临时文件系统(等同于 docker run --tmpfs 的作用, 在使用 swarm 部署时将忽略该选项)

        entrypoint            # 覆盖容器的默认 entrypoint 指令 (等同于 docker run --entrypoint 的作用)

        env_file              # 从指定文件中读取变量设置为容器中的环境变量, 可以是单个值或者一个文件列表, 如果多个文件中的变量重名则后面的变量覆盖前面的变量, environment 的值覆盖 env_file 的值
            文件格式：
                RACK_ENV=development 

        environment           # 设置环境变量， environment 的值可以覆盖 env_file 的值 (等同于 docker run --env 的作用)

        expose                # 暴露端口, 但是不能和宿主机建立映射关系, 类似于 Dockerfile 的 EXPOSE 指令

        external_links        # 连接不在 docker-compose.yml 中定义的容器或者不在 compose 管理的容器(docker run 启动的容器, 在 v3 版本中使用 swarm 部署时将忽略该选项)

        extra_hosts           # 添加 host 记录到容器中的 /etc/hosts 中 (等同于 docker run --add-host 的作用)

        healthcheck           # v2.1 以上版本, 定义容器健康状态检查, 类似于 Dockerfile 的 HEALTHCHECK 指令
            test                  # 检查容器检查状态的命令, 该选项必须是一个字符串或者列表, 第一项必须是 NONE, CMD 或 CMD-SHELL, 如果其是一个字符串则相当于 CMD-SHELL 加该字符串
                NONE                  # 禁用容器的健康状态检测
                CMD                   # test: ["CMD", "curl", "-f", "http://localhost"]
                CMD-SHELL             # test: ["CMD-SHELL", "curl -f http://localhost || exit 1"] 或者　test: curl -f https://localhost || exit 1
            interval: 1m30s       # 每次检查之间的间隔时间
            timeout: 10s          # 运行命令的超时时间
            retries: 3            # 重试次数
            start_period: 40s     # v3.4 以上新增的选项, 定义容器启动时间间隔
            disable: true         # true 或 false, 表示是否禁用健康状态检测和　test: NONE 相同

        image                 # 指定 docker 镜像, 可以是远程仓库镜像、本地镜像

        init                  # v3.7 中新增的参数, true 或 false 表示是否在容器中运行一个 init, 它接收信号并传递给进程

        isolation             # 隔离容器技术, 在 Linux 中仅支持 default 值

        labels                # 使用 Docker 标签将元数据添加到容器, 与 Dockerfile 中的 LABELS 类似

        links                 # 链接到其它服务中的容器, 该选项是 docker 历史遗留的选项, 目前已被用户自定义网络名称空间取代, 最终有可能被废弃 (在使用 swarm 部署时将忽略该选项)

        logging               # 设置容器日志服务
            driver                # 指定日志记录驱动程序, 默认 json-file (等同于 docker run --log-driver 的作用)
            options               # 指定日志的相关参数 (等同于 docker run --log-opt 的作用)
                max-size              # 设置单个日志文件的大小, 当到达这个值后会进行日志滚动操作
                max-file              # 日志文件保留的数量

        network_mode          # 指定网络模式 (等同于 docker run --net 的作用, 在使用 swarm 部署时将忽略该选项，主机模式如 network_mode: "host" )         

        networks              # 将容器加入指定网络 (等同于 docker network connect 的作用), networks 可以位于 compose 文件顶级键和 services 键的二级键
            aliases               # 同一网络上的容器可以使用服务名称或别名连接到其中一个服务的容器
            ipv4_address      # IP V4 格式
            ipv6_address      # IP V6 格式

            示例:
                version: '3.7'
                services: 
                    test: 
                        image: nginx:1.14-alpine
                        container_name: mynginx
                        command: ifconfig
                        networks: 
                            app_net:                                # 调用下面 networks 定义的 app_net 网络
                            ipv4_address: 172.16.238.10
                networks:
                    app_net:
                        driver: bridge
                        ipam:
                            driver: default
                            config:
                                - subnet: 172.16.238.0/24

        pid: 'host'           # 共享宿主机的 进程空间(PID)

        ports                 # 建立宿主机和容器之间的端口映射关系, ports 支持两种语法格式
            SHORT 语法格式示例:
                - "3000"                            # 暴露容器的 3000 端口, 宿主机的端口由 docker 随机映射一个没有被占用的端口
                - "3000-3005"                       # 暴露容器的 3000 到 3005 端口, 宿主机的端口由 docker 随机映射没有被占用的端口
                - "8000:8000"                       # 容器的 8000 端口和宿主机的 8000 端口建立映射关系
                - "9090-9091:8080-8081"
                - "127.0.0.1:8001:8001"             # 指定映射宿主机的指定地址的
                - "127.0.0.1:5000-5010:5000-5010"   
                - "6060:6060/udp"                   # 指定协议

            LONG 语法格式示例:(v3.2 新增的语法格式)
                ports:
                    - target: 80                    # 容器端口
                      published: 8080               # 宿主机端口
                      protocol: tcp                 # 协议类型
                      mode: host                    # host 在每个节点上发布主机端口,  ingress 对于群模式端口进行负载均衡

        secrets               # 不知道怎么用

        security_opt          # 为每个容器覆盖默认的标签 (在使用 swarm 部署时将忽略该选项)

        stop_grace_period     # 指定在发送了 SIGTERM 信号之后, 容器等待多少秒之后退出(默认 10s)

        stop_signal           # 指定停止容器发送的信号 (默认为 SIGTERM 相当于 kill PID; SIGKILL 相当于 kill -9 PID; 在使用 swarm 部署时将忽略该选项)

        sysctls               # 设置容器中的内核参数 (在使用 swarm 部署时将忽略该选项)

        ulimits               # 设置容器的 limit

        userns_mode           # 如果Docker守护程序配置了用户名称空间, 则禁用此服务的用户名称空间 (在使用 swarm 部署时将忽略该选项)

        volumes               # 定义容器和宿主机的卷映射关系, 其和 networks 一样可以位于 services 键的二级键和 compose 顶级键, 如果需要跨服务间使用则在顶级键定义, 在 services 中引用
            SHORT 语法格式示例:
                volumes:
                    - /var/lib/mysql                # 映射容器内的 /var/lib/mysql 到宿主机的一个随机目录中
                    - /opt/data:/var/lib/mysql      # 映射容器内的 /var/lib/mysql 到宿主机的 /opt/data
                    - ./cache:/tmp/cache            # 映射容器内的 /var/lib/mysql 到宿主机 compose 文件所在的位置
                    - ~/configs:/etc/configs/:ro    # 映射容器宿主机的目录到容器中去, 权限只读
                    - datavolume:/var/lib/mysql     # datavolume 为 volumes 顶级键定义的目录, 在此处直接调用

            LONG 语法格式示例:(v3.2 新增的语法格式)
                version: "3.2"
                services:
                    web:
                        image: nginx:alpine
                        ports:
                            - "80:80"
                        volumes:
                            - type: volume                  # mount 的类型, 必须是 bind、volume 或 tmpfs
                                source: mydata              # 宿主机目录
                                target: /data               # 容器目录
                                volume:                     # 配置额外的选项, 其 key 必须和 type 的值相同
                                    nocopy: true                # volume 额外的选项, 在创建卷时禁用从容器复制数据
                            - type: bind                    # volume 模式只指定容器路径即可, 宿主机路径随机生成; bind 需要指定容器和数据机的映射路径
                                source: ./static
                                target: /opt/app/static
                                read_only: true             # 设置文件系统为只读文件系统
                volumes:
                    mydata:                                 # 定义在 volume, 可在所有服务中调用

        restart               # 定义容器重启策略(在使用 swarm 部署时将忽略该选项, 在 swarm 使用 restart_policy 代替 restart)
            no                    # 禁止自动重启容器(默认)
            always                # 无论如何容器都会重启
            on-failure            # 当出现 on-failure 报错时, 容器重新启动

        其他选项：
            domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir
            上面这些选项都只接受单个值和 docker run 的对应参数类似

        对于值为时间的可接受的值：
            2.5s
            10s
            1m30s
            2h32m
            5h34m56s
            时间单位: us, ms, s, m， h
        对于值为大小的可接受的值：
            2b
            1024kb
            2048k
            300m
            1gb
            单位: b, k, m, g 或者 kb, mb, gb
    networks          # 定义 networks 信息
        driver                # 指定网络模式, 大多数情况下, 它 bridge 于单个主机和 overlay Swarm 上
            bridge                # Docker 默认使用 bridge 连接单个主机上的网络
            overlay               # overlay 驱动程序创建一个跨多个节点命名的网络
            host                  # 共享主机网络名称空间(等同于 docker run --net=host)
            none                  # 等同于 docker run --net=none
        driver_opts           # v3.2以上版本, 传递给驱动程序的参数, 这些参数取决于驱动程序
        attachable            # driver 为 overlay 时使用, 如果设置为 true 则除了服务之外，独立容器也可以附加到该网络; 如果独立容器连接到该网络，则它可以与其他 Docker 守护进程连接到的该网络的服务和独立容器进行通信
        ipam                  # 自定义 IPAM 配置. 这是一个具有多个属性的对象, 每个属性都是可选的
            driver                # IPAM 驱动程序, bridge 或者 default
            config                # 配置项
                subnet                # CIDR格式的子网，表示该网络的网段
        external              # 外部网络, 如果设置为 true 则 docker-compose up 不会尝试创建它, 如果它不存在则引发错误
        name                  # v3.5 以上版本, 为此网络设置名称
```

文件格式示例

```
    version: "3"
    services:
      redis:
        image: redis:alpine
        ports:
          - "6379"
        networks:
          - frontend
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
      db:
        image: postgres:9.4
        volumes:
          - db-data:/var/lib/postgresql/data
        networks:
          - backend
        deploy:
          placement:
            constraints: [node.role == manager]
```



## 图形化管理

Portainer 是一个开源、轻量级 Docker 管理用户页面，基于 Docker API，管理 Docker 主机。一般单机下不会使用图形管理页面，这里简单了解下就好。

### 安装 Portainer

① 创建数据卷：docker volume create portainer_volume

② 启动 Portainer 容器：docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_volume:/data portainer/portainer



### 访问 Portainer

访问 ip:9000，即可进入 Portainer 管理页面，首先需要创建一个管理用户

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020224032167-156230665.png)

接着连接本机

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020224620389-1112771799.png)

连接本地进入后就可以进行容器、镜像等的管理了。

![img](https://img2018.cnblogs.com/blog/856154/201910/856154-20191020224646240-69261385.png)

 

# Docker

## 安装

```shell
# 检查服务器的内核版本，必须是3.10及以上：
uname -r
# 安装docker
yum install docker
# 输入y确认
# 启动docker
systemctl start docker
# 查看docker的版本：
[root@izwz9ib5he33fx3jnuis2xz ~]# docker -v
Docker version 1.13.1, build 94f4240/1.13.1
# 设置开机启动docker
systemctl enable docker
# 停止docker
systemctl stop docker
```



## 常用命令

### 容器信息

```shell
# 查看docker容器版本
docker version
# 查看docker容器信息
docker info
# 查看docker容器帮助
docker --help
```



### 镜像操作

提示：对于镜像的操作可使用镜像名、镜像长ID和短ID。

#### 镜像查看

```shell
# 列出本地images
docker images
# 含中间映像层
docker images -a
# 只显示镜像ID
docker images -q
# 含中间映像层
docker images -qa
# 显示镜像摘要信息(DIGEST列)
docker images --digests
# 显示镜像完整信息
docker images --no-trunc
# 显示指定镜像的历史创建；参数：-H 镜像大小和日期，默认为true；--no-trunc  显示完整的提交记录；-q  仅列出提交记录ID
docker history -H redis
```



#### 镜像搜索

```shell
# 搜索仓库MySQL镜像
docker search mysql
# --filter=stars=600：只显示 starts>=600 的镜像
docker search --filter=stars=600 mysql
# --no-trunc 显示镜像完整 DESCRIPTION 描述
docker search --no-trunc mysql
# --automated ：只列出 AUTOMATED=OK 的镜像
docker search  --automated mysql
```



#### 镜像下载

```shell
# 下载Redis官方最新镜像，相当于：docker pull redis:latest
docker pull redis
# 下载仓库所有Redis镜像
docker pull -a redis
# 下载私人仓库镜像
docker pull bitnami/redis
```



#### 镜像删除

```shell
# 单个镜像删除，相当于：docker rmi redis:latest
docker rmi redis
# 强制删除(针对基于镜像有运行的容器进程)
docker rmi -f redis
# 多个镜像删除，不同镜像间以空格间隔
docker rmi -f redis tomcat nginx
# 删除本地全部镜像
docker rmi -f $(docker images -q)
```



#### 镜像构建

```shell
# 编写dockerfile
cd /docker/dockerfile
vim mycentos
# 构建docker镜像
docker build -f /docker/dockerfile/mycentos -t mycentos:1.1
```



### 容器操作

提示：对于容器的操作可使用CONTAINER ID 或 NAMES。

#### 容器启动

```shell
# 新建并启动容器，参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称
docker run -i -t --name mycentos
# 后台启动容器，参数：-d  已守护方式启动容器
docker run -d mycentos
```

注意：此时使用"docker ps -a"会发现容器已经退出。这是docker的机制：要使Docker容器后台运行，就必须有一个前台进程。解决方案：将你要运行的程序以前台进程的形式运行。

```shell
# 启动一个或多个已经被停止的容器
docker start redis
# 重启容器
docker restart redis
```



#### 容器进程

```shell
# top支持 ps 命令参数，格式：docker top [OPTIONS] CONTAINER [ps OPTIONS]
# 列出redis容器中运行进程
docker top redis
# 查看所有运行容器的进程信息
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done
```

#### 查看容器细节

docker inspect

json格式存储的容器详细细节



#### 容器日志

```shell
$ docker logs [OPTIONS] CONTAINER
  Options:
        --details          显示更多的信息
    -f, --follow           跟踪实时日志
        --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
        --tail string      从日志末尾显示多少行日志， 默认是all
    -t, --timestamps  显示时间戳
        --until string    显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）

# 查看redis容器日志，默认参数
docker logs rabbitmq
# 查看redis容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；
docker logs -f -t --tail=20 redis
# 查看容器redis从2019年05月21日后的最新10条日志。
docker logs --since="2019-05-21" --tail=10 redis

# 查看指定时间后的日志，只显示最后100行
$ docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID
# 查看最近30分钟的日志
$ docker logs --since 30m CONTAINER_ID
# 查看某时间之后的日志
$ docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID
# 查看某时间段日志
$ docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
# 查看最后100行，并过滤关键词Exception
$ docker logs -f --tail=100 CONTAINER_ID | grep "Exception"
```



#### 容器的进入与退出

```shell
# 使用run方式在创建时进入
docker run -it centos /bin/bash
# 关闭容器并退出
exit
# 仅退出容器，不关闭
快捷键：Ctrl + P + Q
# 直接进入centos 容器启动命令的终端，不会启动新进程，多个attach连接共享容器屏幕，参数：--sig-proxy=false  确保CTRL-D或CTRL-C不会关闭容器
docker attach --sig-proxy=false centos 
# 在 centos 容器中打开新的交互模式终端，可以启动新进程，参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端
docker exec -i -t  centos /bin/bash
# 以交互模式在容器中执行命令，结果返回到当前终端屏幕
docker exec -i -t centos ls -l /tmp
# 以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
docker exec -d centos  touch cache.txt 
```



#### 查看容器

```shell
# 查看正在运行的容器
docker ps
# 查看正在运行的容器的ID
docker ps -q
# 查看正在运行+历史运行过的容器
docker ps -a
# 显示运行容器总文件大小
docker ps -s

# 显示最近创建容器
docker ps -l
# 显示最近创建的3个容器
docker ps -n 3
# 不截断输出
docker ps --no-trunc 

# 获取镜像redis的元信息
docker inspect redis
# 获取正在运行的容器redis的 IP
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis
```



#### 容器的停止与删除

```shell
#停止一个运行中的容器
docker stop redis
#杀掉一个运行中的容器
docker kill redis
#删除一个已停止的容器
docker rm redis
#删除一个运行中的容器
docker rm -f redis
#删除多个容器
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
# -l 移除容器间的网络连接，连接名为 db
docker rm -l db 
# -v 删除容器，并删除容器挂载的数据卷
docker rm -v redis
```



#### 生成镜像

```shell
# 基于当前redis容器创建一个新的镜像；参数：-a 提交的镜像作者；-c 使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
docker commit -a="DeepInThought" -m="my redis" [redis容器ID]  myredis:v1.1
```



#### 容器与主机间的数据拷贝

```shell
# 将rabbitmq容器中的文件copy至本地路径
docker cp rabbitmq:/[container_path] [local_path]
# 将主机文件copy至rabbitmq容器
docker cp [local_path] rabbitmq:/[container_path]/
# 将主机文件copy至rabbitmq容器，目录重命名为[container_path]（注意与非重命名copy的区别）
docker cp [local_path] rabbitmq:/[container_path]
```



## 其它操作

### 清理环境

```shell
# 查询容器列表
docker ps -a
# 停用容器 
sudo docker stop [CONTAINER ID] 
# 删除容器
sudo docker rm  [CONTAINER ID]
# 删除镜像
sudo docker rmi [Image ID]
# 检查是否被删除
sudo docker images

# 停止所有容器
docker stop $(docker ps -a -q)
# 删除所有容器
docker rm $(docker ps -a -q)
# 删除所有镜像
docker rmi $(docker images -q)
```



### 文件拷贝

```shell
# 从主机复制到容器
sudo docker cp host_path containerID:container_path
# 从容器复制到主机
sudo docker cp containerID:container_path host_path
```

## Docker安装的脚本

```
#!/bin/bash
# 移除掉旧的版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

# 删除所有旧的数据
sudo rm -rf /var/lib/docker

#  安装依赖包
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# 添加源，使用了阿里云镜像
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 配置缓存
sudo yum makecache fast

# 安装最新稳定版本的docker
sudo yum install -y docker-ce

# 配置镜像加速器
#sudo mkdir -p /etc/docker
#sudo tee /etc/docker/daemon.json <<-'EOF'
#{
# "registry-mirrors": ["http://hub-mirror.c.163.com"]
#}
#EOF

# 启动docker引擎并设置开机启动
sudo systemctl start docker
sudo systemctl enable docker

# 配置当前用户对docker的执行权限
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker
```

将文件报错为docker.sh

```
chmod 777 docker.sh
```

执行脚本 ./docker.sh

遇到的问题

```
Could not retrieve mirrorlist http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os&infra=stock error was
```

解决ping 不通问题

```
[root@seckillmysql ~]# vi /etc/resolv.conf
//增加这两行
nameserver 223.5.5.5
nameserver 223.6.6.6
```

## docker创建Alpine镜像

### 什么是Alpine

Alpine Linux 是一个社区开发的面向安全应用的轻量级Linux发行版，适合用来做[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)镜像、路由器、防火墙、VPNs、VoIP 盒子 以及服务器的操作系统，基于musl libc和Busybox，镜像大小只有5M，并且还提供了包管理工具apk查询和安装软件包。

### 获取Alpine镜像

```
$  docker search alpine
NAME                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
alpine                                 A minimal Docker image based on Alpine Linux…   5587                [OK]                
mhart/alpine-node                      Minimal Node.js built on Alpine Linux           439                                     
anapsix/alpine-java                    Oracle Java 8 (and 7) with GLIBC 2.28 over A…   421                                     [OK]
frolvlad/alpine-glibc                  Alpine Docker image with glibc (~12MB)          210                                     [OK]
gliderlabs/alpine                      Image based on Alpine Linux will help you wi…   180                                     
mvertes/alpine-mongo                   light MongoDB container                         105                                     [OK]
alpine/git                             A  simple git container running in alpine li…   97                                      [OK]
yobasystems/alpine-mariadb             MariaDB running on Alpine Linux [docker] [am…   46                                      [OK]
kiasaki/alpine-postgres                PostgreSQL docker image based on Alpine Linux   44                                      [OK]
alpine/socat                           Run socat command in alpine container           36                                      [OK]
davidcaste/alpine-tomcat               Apache Tomcat 7/8 using Oracle Java 7/8 with…   36                                      [OK]
zzrot/alpine-caddy                     Caddy Server Docker Container running on Alp…   35                                      [OK]
easypi/alpine-arm                      AlpineLinux for RaspberryPi                     32                                      
jfloff/alpine-python                   A small, more complete, Python Docker image …   26                                      [OK]
byrnedo/alpine-curl                    Alpine linux with curl installed and set as …   26                                      [OK]
hermsi/alpine-sshd                     Dockerize your OpenSSH-server with rsync and…   23                                      [OK]
etopian/alpine-php-wordpress           Alpine WordPress Nginx PHP-FPM WP-CLI           21                                      [OK]
hermsi/alpine-fpm-php                  Dockerize your FPM PHP 7.4 upon a lightweigh…   18                                      [OK]
bashell/alpine-bash                    Alpine Linux with /bin/bash as a default she…   13                                      [OK]
zenika/alpine-chrome                   Chrome running in headless mode in a tiny Al…   13                                      [OK]
davidcaste/alpine-java-unlimited-jce   Oracle Java 8 (and 7) with GLIBC 2.21 over A…   13                                      [OK]
spotify/alpine                         Alpine image with `bash` and `curl`.            9                                       [OK]
tenstartups/alpine                     Alpine linux base docker image with useful p…   8                                       [OK]
rawmind/alpine-traefik                 This image is the traefik base. It comes fro…   5                                       [OK]
hermsi/alpine-varnish                  Dockerize Varnish upon a lightweight alpine-…   1                                       [OK]

```

### 获取Alpine镜像

 docker pull 方法

```

docker pull alpine:latest 
latest: Pulling from library/alpine
9d48c3bd43c5: Pull complete 
Digest: sha256:72c42ed48c3a2db31b7dafe17d275b634664a708d901ec9fd57b1529280f01fb
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

dockerfile 文件

```

mkdir alpine && cd alpine
 
touch Dockerfile
'''
#escape=
#This docker file uses alpine:latest image
#VERSION 1.0
#Author: Swift
#e-mail: ilyzhaoxin@sina.com
#DateTime: 2019-08-27 21:15
from alpine:latest
RUN apk add --no-cache mysql-client
ENTRYPOINT ['mysql']
'''
 
docker build .
```

## Docker常用的脚本

### Git

```
FROM alpine/git

MAINTAINER yang <yang@163.com>

# 使用阿里云镜像
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# Install base packages
RUN apk --no-cache update && \
    apk --no-cache upgrade && \
    apk --no-cache add curl bash tzdata tar unzip && \ 
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    rm -fr /tmp/* /var/cache/apk/*

# Define bash as default command
CMD ["/bin/bash"]
```

### Maven

```
FROM maven:3.6-jdk-8-alpine

MAINTAINER yang <yang@163.com>

ARG LOCAL_MAVEN_MIRROR=http://maven.aliyun.com/nexus/content/groups/public/

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

ENV MAVEN_SETING_FILE=/usr/share/maven/conf/settings.xml

VOLUME /var/maven/repository

# Install base packages
RUN apk --no-cache update && \
    apk --no-cache upgrade && \
    apk --no-cache add tzdata unzip git && \ 
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    rm -fr /tmp/* /var/cache/apk/*

COPY settings.xml ${MAVEN_SETING_FILE}
# override default maven docker configuration
RUN cat /usr/share/maven/conf/settings.xml > /usr/share/maven/ref/settings-docker.xml
RUN mkdir -p /root/.m2 && cp /usr/share/maven/conf/settings.xml /root/.m2/settings.xml
```

### Mysql5.7

```
FROM mysql:5.7.33

# 维护者信息
MAINTAINER yang <yang@163.com>

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
COPY buster.sources.list /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y curl git unzip vim wget && \
    apt-get install -y locales kde-l10n-zhcn && \ 
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN sed -i 's/# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/g' /etc/locale.gen

RUN locale && locale-gen "zh_CN.UTF-8"
       
# Set environment variables.
ENV LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN:zh:en_US:en \
    LC_ALL=zh_CN.UTF-8 \
    TZ=Asia/Shanghai \
    DEBIAN_FRONTEND="noninteractive" \
    TERM=xterm

RUN ln -fs /usr/share/zoneinfo/$TZ /etc/localtime && \
            echo $TZ > /etc/timezone && \        
            dpkg-reconfigure --frontend noninteractive tzdata && \
            dpkg-reconfigure --frontend noninteractive locales

COPY my.cnf /etc/mysql/my.cnf
RUN mkdir -p /etc/mysql/mysql-my.conf.d/

EXPOSE 3306
CMD ["mysqld"]
```

### Elasticsearch

```
# https://github.com/elastic/elasticsearch-docker
FROM docker.elastic.co/elasticsearch/elasticsearch:5.3.0

USER root

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# Install base packages
RUN apk --no-cache update && \
    apk --no-cache add curl bash tzdata tar unzip && \ 
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    echo -ne "Alpine Linux 3.4.4 image. (`uname -rsv`)\n" >> /root/.built && \
    rm -fr /tmp/* /var/cache/apk/*

USER elasticsearch

CMD ["/bin/bash", "bin/es-docker"]

EXPOSE 9200 9300
```

### Skywalking

**安装服务端：**这里介绍服务端的两种存储等式，一种是默认的H2存储，即数据存储在内存中，一种是使用elasticsearch存储，

**默认H2存储**

输入以下命令，并耐心待下载。

```
sudo docker run --name skywalking -d -p 1234:1234 -p 11800:11800 -p 12800:12800 --restart always apache/skywalking-oap-server
```

**elasticsearch存储**

安装ElasticSearch

```
sudo docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 --restart always -e "discovery.type=single-node" elasticsearch:6.7.2
```

安装 ElasticSearch管理界面elasticsearch-hq

```
sudo docker run -d --name elastic-hq -p 5000:5000 --restart always elastichq/elasticsearch-hq
```

输入以下命令，并等待下载

```
sudo docker run --name skywalking -d -p 1234:1234 -p 11800:11800 -p 12800:12800 --restart always --link elasticsearch:elasticsearch -e SW_STORAGE=elasticsearch -e SW_STORAGE_ES_CLUSTER_NODES=elasticsearch:9200 apache/skywalking-oap-server 
```

**安装管理界面：**

输入以下命令，并等待下载安装。

```
sudo docker run --name skywalking-ui -d -p 8080:8080 --link skywalking:skywalking -e SW_OAP_ADDRESS=skywalking:12800 --restart always apache/skywalking-ui
```

**访问管理界验证安装结果**

在浏览器里面输入http://192.168.150.134:8080,出现了如下界面，到此Skywalking的安装就大功告成了。

Skywalking8.6.0 脚本

```
# skywalking-oap-server

docker run \
--name skywalking-oap \
--restart always \
-p 11800:11800 \
-p 12800:12800 -d \
--privileged=true \
-e TZ=Asia/Shanghai \
-e SW_STORAGE=elasticsearch7 \
-e SW_STORAGE_ES_CLUSTER_NODES=10.7.1.39:9200 \
-e SW_NAMESPACE=qjd-sw-online \
-v /etc/localtime:/etc/localtime:ro \
apache/skywalking-oap-server:8.6.0-es7

# skywalking-ui

docker run \
--name skywalking-ui \
--restart always \
-p 8081:8080 -d \
--privileged=true \
--link skywalking-oap:skywalking-oap \
-e TZ=Asia/Shanghai \
-e SW_OAP_ADDRESS=http://10.1.67.88:12800 \
-v /etc/localtime:/etc/localtime:ro \
apache/skywalking-ui:8.6.0
```

## Docker 资源

### Docker 资源

- Docker 官方主页: [https://www.docker.com](https://www.docker.com/)
- Docker 官方博客: https://blog.docker.com/
- Docker 官方文档: https://docs.docker.com/
- Docker Store: [https://store.docker.com](https://store.docker.com/)
- Docker Cloud: [https://cloud.docker.com](https://cloud.docker.com/)
- Docker Hub: [https://hub.docker.com](https://hub.docker.com/)
- Docker 的源代码仓库: https://github.com/moby/moby
- Docker 发布版本历史: https://docs.docker.com/release-notes/
- Docker 常见问题: https://docs.docker.com/engine/faq/
- Docker 远端应用 API: https://docs.docker.com/develop/sdk/

### Docker 国内镜像

阿里云的加速器：https://help.aliyun.com/document_detail/60750.html

网易加速器：http://hub-mirror.c.163.com

官方中国加速器：https://registry.docker-cn.com

ustc 的镜像：https://docker.mirrors.ustc.edu.cn

daocloud：https://www.daocloud.io/mirror#accelerator-doc（注册后使用）

## 问题解决

#### 终端提示Unable to find image hello-world:latest locally的问题

意思是docker在本地没有找到hello-world镜像，也没有从docker仓库中拉取到镜像。

这是因为docker服务器在国外，基于网速与“和谐墙”的问题，所以我们在国内操作国外镜像可能无法正常拉取，这需要我们为docker设置国内的阿里云镜像加速器。

 解决办法
1. 创建文件daemon.json文件

  ```
  touch  /etc/docker/daemon.json
  ```

2. 配置文件/etc/docker/daemon.json,添加阿里云镜像

​       国产镜像： https://registry.docker-cn.com

  ```
  { 
  "registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"] 
  }
  ```

3. 重启docker服务

  ```
  #systemctl daemon-reload
  #systemctl restart docker
  #sudo systemctl status docker
  ```

  ### loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)

说明docker就没有启动成功

解决方法：

vi /etc/docker/daemon.json

```
写入指定参数：
{ "storage-driver": "devicemapper" }
```

重启docker服务

```
systemctl restart docker
```

#### Docker Application Container Engine Loaded

```
修改docker.service启动信息
# 修改docker.service启动信息
vim /usr/lib/systemd/system/docker.service
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock
修改daemon.json
 
复制代码
#修改daemon.json
vim /etc/docker/daemon.json为 daemon.conf
```

