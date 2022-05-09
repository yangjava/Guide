# Kubernetes1--Kubeadm安装

Kubeadm安装Kubernetes环境总结:

Kubernetes服务架构采用Mater-Slave，不考虑高可用，一般采用一个Master节点和多个Slave节点，这里采用1Master和2Slave节点

首先这里采用Vmware来安装三台Linux操作系统，网络设置为NAT模式

操作系统centos7,内核版本：





Master和Slave均要操作

1.配置好各节点hosts文件

K8smaster    192.168.3.3

K8snode1    192.168.3.4

K8snode2    192.168.3.5

2.关闭系统防火墙

Systemctl  stop    firewalld.service

Systemctl  disable  firewalld.service



3.关闭Selinux

Vim  /etc/selinux/config

SELINUX=disabled

SELinux配置文件/etc/selinux/config控制系统下一次启动过程中载入哪个策略，以及系统运行在哪个模式下。



4.关闭swap

Swapoff  -a



5.配置系统内核参数使流过网桥的流量也进入iptables/netfilter框架中

Vim 在/etc/sysctl.conf中添加以下配置：

net.bridge.bridge-nf-call-iptables = 1

net.bridge.bridge-nf-call-ip6tables = 1



sysctl –p

重启Linux

6.配置阿里K8S YUM源

cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]

name=Kubernetes

baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=0

EOF



yum -y install epel-release

EPEL，即Extra Packages for Enterprise Linux的简称，是为企业级Linux提供的一组高质量的额外软件包，包括但不限于Red Hat Enterprise Linux (RHEL), CentOS and Scientific Linux (SL), Oracle Enterprise Linux (OEL)，使用docker之前安装EPEL源。
如果安装失败，也可以手动安装：

wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm

rpm -vih epel-release-7-11.noarch.rpm



安装epel源遇到了很多错，可以看一下后面的解决过程

yum clean all

yum makecache

7.安装kubeadm和相关工具包

不能下载以下软件，可以配置一下阿里源仓库

https://blog.csdn.net/aiming66/article/details/78879996

yum -y install docker kubelet kubeadm kubectl kubernetes-cni



8.启动Docker与kubelet服务

systemctl enable docker && systemctl start docker

systemctl enable kubelet



操作过程注意查看日志  tail -f /var/log/messages

kubelet报错：



添加如下：

KUBELET_EXTRA_ARGS=--runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice



9.配置加速镜像

sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'

{undefined

  "registry-mirrors": ["https://99b9e5v9.mirror.aliyuncs.com"]

}

EOF



不同docker版本可能配置信息不同，配置文件 /etc/sysconfig/docker



sudo systemctl daemon-reload

sudo systemctl restart docker

10.下载K8S相关镜像

这里下载相关镜像版本应与kubeadm版本一致，这里采用kubeadm  1.12.0

#!/bin/bash
images=(kube-proxy-amd64:v1.12.0-rc.1 
        kube-scheduler-amd64:v1.12.0-rc.1 
        kube-controller-manager-amd64:v1.12.0-rc.1 
        kube-apiserver-amd64:v1.12.0-rc.1
        etcd-amd64:3.1.12 
        coredns:1.2.2
        pause-amd64:3.1 
        kubernetes-dashboard-amd64:v1.8.3 
        k8s-dns-sidecar-amd64:1.14.8 
        k8s-dns-kube-dns-amd64:1.14.8
        k8s-dns-dnsmasq-nanny-amd64:1.14.8)
for imageName in ${images[@]} ; do
    docker pull keveon/$imageName
done

docker pull quay.io/coreos/flannel:v0.9.1-amd64

docker tag keveon/kube-proxy-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-proxy:v1.12.0-rc.1
docker rmi keveon/kube-proxy-amd64:v1.12.0-rc.1

docker tag keveon/kube-scheduler-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-scheduler:v1.12.0-rc.1
docker rmi keveon/kube-scheduler-amd64:v1.12.0-rc.1

docker tag keveon/kube-controller-manager-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1
docker rmi keveon/kube-controller-manager-amd64:v1.12.0-rc.1

docker tag keveon/kube-apiserver-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-apiserver:v1.12.0-rc.1
docker rmi keveon/kube-apiserver-amd64:v1.12.0-rc.1

docker tag keveon/etcd-amd64:3.1.12 k8s.gcr.io/etcd:3.2.24
docker rmi keveon/etcd-amd64:3.1.12

docker tag keveon/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2
docker rmi keveon/coredns:1.2.2

docker tag keveon/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker rmi keveon/pause-amd64:3.1
下面是1.10版本的下载方式，各镜像变化方式蛮大的。

#!/bin/bash

images=(kube-proxy-amd64:v1.10.0 kube-scheduler-amd64:v1.10.0 kube-controller-manager-amd64:v1.10.0 kube-apiserver-amd64:v1.10.0 etcd-amd64:3.1.12 pause-amd64:3.1 kubernetes-dashboard-amd64:v1.8.3 k8s-dns-sidecar-amd64:1.14.8 k8s-dns-kube-dns-amd64:1.14.8 k8s-dns-dnsmasq-nanny-amd64:1.14.8)

for imageName in ${images[@]} ; do

    docker pull keveon/$imageName
    
    docker tag keveon/$imageName k8s.gcr.io/$imageName
    
    docker rmi keveon/$imageName

done



下载结果如下：



关于images镜像版本，下载地址：

https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=docker.io%2Fkeveon&starCount=0

镜像版本应与docker  kubelet   kubeadm 版本保持一致





所以此处改为下载v1.12.0版本镜像

11.master节点操作如下

初始化安装K8S Master

kubeadm init --kubernetes-version=v1.12.0-rc.1 --pod-network-cidr=10.244.0.0/16

版本要与下载的镜像版本一致，由于kubeadm现在是1.12.0版本，镜像的名字需要修改如下：

镜像仓库位置在：

https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=docker.io%2Fkeveon&starCount=0

这里需要考虑kubeadm的安装版本，可以查看日志tail -f /var/log/messages来查看需要容器版本有哪些，利用docker  tag修改如下：





配置kubectl认证信息

mkdir -p $HOME/.kube

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config



master节点安装flannel网络：

12.安装flannel网络

mkdir -p /etc/cni/net.d/

cat <<EOF> /etc/cni/net.d/10-flannel.conf

{
"name": "cbr0",
"type": "flannel",
"delegate": {
"isDefaultGateway": true
}
}

EOF

{
"name":"cbr0"
"type":"flannel"
"delegate":{"isDefaultGateway":true}
}




mkdir /usr/share/oci-umount/oci-umount.d -p

mkdir /run/flannel/

cat <<EOF> /run/flannel/subnet.env

FLANNEL_NETWORK=10.244.0.0/16

FLANNEL_SUBNET=10.244.1.0/24

FLANNEL_MTU=1450

FLANNEL_IPMASQ=true

EOF



确保配置文件配置正确

 /etc/cni/net.d/10-flannel.conf

/run/flannel/subnet.env

查看镜像是否已经下载：



kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml



13.Node节点：

Node节点镜像准备如下：



执行1-10步骤

配置flannel网络文件：

 /etc/cni/net.d/10-flannel.conf

/run/flannel/subnet.env

准备flannel镜像：



加入master节点

kubeadm join 

其他操作

master重启

kubeadm reset

swapoff –a

kubeadm init --kubernetes-version=v1.12.0-rc.1 --pod-network-cidr=10.244.0.0/16

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config

配置/etc/cni/net.d/10-flannel.conf文件

{
  "name": "cbr0",
  "type": "flannel",
  "delegate": {
    "isDefaultGateway": true
  }
}
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

node重启

kubeadm reset

swapoff –a

kubeadm join

kubectl管理：

# 查看节点状态

kubectl get nodes

kubectl describe node k8snode1

# 查看pods状态

kubectl get pods --all-namespaces

# 查看K8S集群状态

kubectl get cs

查看资源：

kubectl get pods

删除某个pod:

kubectl delete pod pod-name

DNS服务搭建





配置好各主机/etc/hosts

验证DNS服务：



kubectl run curl --image=radial/busyboxplus:curl -i --tty

nslookup kubernetes.default

kubectl delete deploy curl

安装epel引发的血案：
安装epel 

yum install epel-release

或者

wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm

rpm -vih epel-release-7-11.noarch.rpm

其中有两台安装成功，一台安装失败，报错如下：



查了一下，解决办法大致如下，修改epel.repo文件或者配置DNS=8.8.8.8 仍然报错



后来又查了一下，说是更换源：

#中科大镜像源 高质量源

rpm -Uvh  http://mirrors.ustc.edu.cn/centos/7.0.1406/extras/x86_64/Packages/epel-release-7-5.noarch.rpm

#浙大源 

rpm-Uvh http://mirrors.zju.edu.cn/epel/7/x86_64/e/epel-release-7-5.noarch.rpm

#上海交大源

rpm -Uvh  http://ftp.sjtu.edu.cn/fedora/epel/7/x86_64/e/epel-release-7-5.noarch.rpm

#还有东北的活雷锋东软的源

http://mirrors.neusoft.edu.cn/epel/7/x86_64/e/epel-release-7-5.noarch.rpm

#sohu镜像源

rpm -Uvh  http://mirrors.sohu.com/fedora-epel/7/x86_64/e/epel-release-7-2.noarch.rpm

清华大学：

https://mirrors.tuna.tsinghua.edu.cn

阿里源：

http://mirrors.aliyun.com

将epel.repo  baseurl更换为上海交大源：



还是报错：



一直显示这个文件import失败，于是我看了一下这个文件：



浏览器是可以访问到的，说明文件源没有问题。服务器访问一下：



发现没有内容输出，换了一台发现有内容输出：



擦，好像是这台服务器的网络配置有问题或者是防火墙有问题可能过滤拦截了。

在另外一台上使用wget下载该配置文件： 可以正常下载



在本台上下载，发现下载文件竟然是登录网关页面的html代码，擦，果然是网络的问题。



Master节点报错如下：



后来发现quay.io/coreos/flannel:v0.9.1-amd64这个镜像没有下载下来

执行这一步：

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

看一下上述文件的内容;



其中要使用镜像quay.io/coreos/flannel:v0.9.1-amd64

下载images镜像脚本以及tag修改如下：

#!/bin/bash
images=(kube-proxy-amd64:v1.12.0-rc.1 
        kube-scheduler-amd64:v1.12.0-rc.1 
        kube-controller-manager-amd64:v1.12.0-rc.1 
        kube-apiserver-amd64:v1.12.0-rc.1
        etcd-amd64:3.1.12 
        coredns:1.2.2
        pause-amd64:3.1 
        kubernetes-dashboard-amd64:v1.8.3 
        k8s-dns-sidecar-amd64:1.14.8 
        k8s-dns-kube-dns-amd64:1.14.8
        k8s-dns-dnsmasq-nanny-amd64:1.14.8)
for imageName in ${images[@]} ; do
    docker pull keveon/$imageName
done

docker pull quay.io/coreos/flannel:v0.9.1-amd64

docker tag keveon/kube-proxy-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-proxy:v1.12.0-rc.1
docker rmi keveon/kube-proxy-amd64:v1.12.0-rc.1

docker tag keveon/kube-scheduler-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-scheduler:v1.12.0-rc.1
docker rmi keveon/kube-scheduler-amd64:v1.12.0-rc.1

docker tag keveon/kube-controller-manager-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1
docker rmi keveon/kube-controller-manager-amd64:v1.12.0-rc.1

docker tag keveon/kube-apiserver-amd64:v1.12.0-rc.1 k8s.gcr.io/kube-apiserver:v1.12.0-rc.1
docker rmi keveon/kube-apiserver-amd64:v1.12.0-rc.1

docker tag keveon/etcd-amd64:3.1.12 k8s.gcr.io/etcd:3.2.24
docker rmi keveon/etcd-amd64:3.1.12

docker tag keveon/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2
docker rmi keveon/coredns:1.2.2

docker tag keveon/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker rmi keveon/pause-amd64:3.1

修改文件内容： /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/ --cni-bin-dir=/opt/cni/bin""

# Kubernetes2--k8s资源调度

研究一下Kubernetes的资源调度器的实现原理以及大神们的改进。

k8s的基本架构如下：



Scheduler调度器做为Kubernetes三大核心组件之一， 承载着整个集群资源的调度功能，其根据特定调度算法和策略，将Pod调度到最优工作节点上，从而更合理与充分的利用集群计算资源。其作用是根据特定的调度算法和策略将Pod调度到指定的计算节点（Node）上，其做为单独的程序运行，启动之后会一直监听API Server，获取PodSpec.NodeName为空的Pod，对每个Pod都会创建一个绑定。默认情况下，k8s的调度器采用扩散策略，将同一集群内部的pod对象调度到不同的Node节点，以保证资源的均衡利用。



首先用户通过 Kubernetes 客户端 Kubectl 提交创建 Pod 的 Yaml 的文件，向Kubernetes 系统发起资源请求，该资源请求被提交到 
Kubernetes 系统中，用户通过命令行工具 Kubectl 向 Kubernetes 集群即 APIServer 用 的方式发送“POST”请求，即创建 Pod 的请求。
 APIServer 接收到请求后把创建 Pod 的信息存储到 Etcd 中，从集群运行那一刻起，资源调度系统 Scheduler 就会定时去监控 APIServer
通过 APIServer 得到创建 Pod 的信息，Scheduler 采用 watch 机制，一旦 Etcd 存储 Pod 信息成功便会立即通知APIServer，APIServer会立即把Pod创建的消息通知Scheduler，Scheduler发现 Pod 的属性中 Dest Node 为空时（Dest Node=””）便会立即触发调度流程进行调度。

Kubernetes的调度器以插件化形式实现的，调度器的源码位于kubernetes/plugin/中

kubernetes/pkg/scheduler/ 
`-- scheduler               //调度相关的具体实现              
|-- algorithm
|   |-- predicates       //节点筛选策略
|   `-- priorities         //节点打分策略
|       `-- util
|-- algorithmprovider
|   `-- defaults          //定义默认的调度器  
Scheduler创建和运行的过程， 对应的代码在kubernetes/pkg/scheduler//scheduler.go。调度器是Kubernetes容器集群管理系统中加载并运行的调度程序，负责收集、统计分析容器集群管理系统中所有Node的资源使用情况，然后以此为依据将新建的Pod发送到优先级最高的可用Node上去建立。 

具体的调度过程，一般如下：

首先，客户端通过API Server的REST API/kubectl/helm创建pod/service/deployment/job等，支持类型主要为JSON/YAML/helm tgz。
接下来，API Server收到用户请求，存储到相关数据到etcd。
调度器通过API Server查看未调度（bind）的Pod列表，循环遍历地为每个Pod分配节点，尝试为Pod分配节点。调度过程分为2个阶段：
第一阶段：预选过程，过滤节点，调度器用一组规则过滤掉不符合要求的主机。比如Pod指定了所需要的资源量，那么可用资源比Pod需要的资源量少的主机会被过滤掉。
第二阶段：优选过程，节点优先级打分，对第一步筛选出的符合要求的主机进行打分，在主机打分阶段，调度器会考虑一些整体优化策略，比如把容一个Replication Controller的副本分布到不同的主机上，使用最低负载的主机等。
选择主机：选择打分最高的节点，进行binding操作，结果存储到etcd中。
所选节点对于的kubelet根据调度结果执行Pod创建操作。
预选过程：  源码位置kubernetes/pkg/scheduler/algorithm/predicates/predicates.go

predicatesOrdering = []string{CheckNodeConditionPred, CheckNodeUnschedulablePred,
		GeneralPred, HostNamePred, PodFitsHostPortsPred,
		MatchNodeSelectorPred, PodFitsResourcesPred, NoDiskConflictPred,
		PodToleratesNodeTaintsPred, PodToleratesNodeNoExecuteTaintsPred, CheckNodeLabelPresencePred,
		CheckServiceAffinityPred, MaxEBSVolumeCountPred, MaxGCEPDVolumeCountPred, MaxCSIVolumeCountPred,
		MaxAzureDiskVolumeCountPred, CheckVolumeBindingPred, NoVolumeZoneConflictPred,
		CheckNodeMemoryPressurePred, CheckNodePIDPressurePred, CheckNodeDiskPressurePred, MatchInterPodAffinityPred}
主要包括主机的目录，端口，cpu内存资源等条件进行物理主机的筛选，筛选出可以进行调度的物理机节点，然后进行优选环节，通过某种策略进行可用节点的评分，最终选出最优节点。

优选过程： 源码位置 kubernetes/pkg/scheduler/algorithm/priorities/

用一组优先级函数处理每一个通过预选的节点，每一个优先级函数会返回一个0-10的分数，分数越高表示节点越优， 同时每一个函数也会对应一个表示权重的值。最终主机的得分用以下公式计算得出：
               finalScoreNode = (weight1 * priorityFunc1) + (weight2 * priorityFunc2) + … + (weightn * priorityFuncn)

优先级策略：

LeastRequestedPriority：节点的优先级就由节点空闲资源与节点总容量的比值，即由（总容量-节点上Pod的容量总和-新Pod的容量）/总容量）来决定。CPU和内存具有相同权重，资源空闲比越高的节点得分越高。

                 cpu((capacity – sum(requested)) * 10 / capacity) + memory((capacity – sum(requested)) * 10 / capacity) / 2

BalancedResourceAllocation：CPU和内存使用率越接近的节点权重越高，该策略不能单独使用，必须和LeastRequestedPriority组合使用，尽量选择在部署Pod后各项资源更均衡的机器。如果请求的资源（CPU或者内存）大于节点的capacity，那么该节点永远不会被调度到。

InterPodAffinityPriority：通过迭代 weightedPodAffinityTerm 的元素计算和，并且如果对该节点满足相应的PodAffinityTerm，则将 “weight” 加到和中，具有最高和的节点是最优选的。 `

SelectorSpreadPriority：为了更好的容灾，对同属于一个service、replication controller或者replica的多个Pod副本，尽量调度到多个不同的节点上。如果指定了区域，调度器则会尽量把Pod分散在不同区域的不同节点上。当一个Pod的被调度时，会先查找Pod对于的service或者replication controller，然后查找service或replication controller中已存在的Pod，运行Pod越少的节点的得分越高。

NodeAffinityPriority：亲和性机制。Node Selectors（调度时将pod限定在指定节点上），支持多种操作符（In, NotIn, Exists, DoesNotExist, Gt, Lt），而不限于对节点labels的精确匹配。另外支持两种类型的选择器，一种是“hard（requiredDuringSchedulingIgnoredDuringExecution）”选择器，它保证所选的主机必须满足所有Pod对主机的规则要求。这种选择器更像是之前的nodeselector，在nodeselector的基础上增加了更合适的表现语法。另一种是“soft（preferresDuringSchedulingIgnoredDuringExecution）”选择器，它作为对调度器的提示，调度器会尽量但不保证满足NodeSelector的所有要求。

NodePreferAvoidPodsPriority（权重1W）：如果 节点的 Anotation 没有设置 key-value:scheduler. alpha.kubernetes.io/ preferAvoidPods = "..."，则节点对该 policy 的得分就是10分，加上权重10000，那么该node对该policy的得分至少10W分。如果Node的Anotation设置了，scheduler.alpha.kubernetes.io/preferAvoidPods = "..." ，如果该 pod 对应的 Controller 是 ReplicationController 或 ReplicaSet，则该 node 对该 policy 的得分就是0分。

TaintTolerationPriority : 使用 Pod 中 tolerationList 与 节点 Taint 进行匹配，配对成功的项越多，则得分越低。

ImageLocalityPriority：根据Node上是否存在一个pod的容器运行所需镜像大小对优先级打分，分值为0-10。遍历全部Node，如果某个Node上pod容器所需的镜像一个都不存在，分值为0；如果Node上存在Pod容器部分所需镜像，则根据这些镜像的大小来决定分值，镜像越大，分值就越高；如果Node上存在pod所需全部镜像，分值为10。

EqualPriority : EqualPriority 是一个优先级函数，它给予所有节点相等权重。

MostRequestedPriority : 在 ClusterAutoscalerProvider 中，替换 LeastRequestedPriority，给使用多资源的节点，更高的优先级。计算公式为：(cpu(10 sum(requested) / capacity) + memory(10 sum(requested) / capacity)) / 2

自定义调度：

kubernetes/pkg/scheduler/algorithmprovider/defaults/defaults.go

自定义预选以及优选策略，scheduler在启动的时候可以通过 --policy-config-file参数可以指定调度策略文件。

"kind" : "Policy",
"apiVersion" : "v1",
"predicates" : [
    {"name" : "PodFitsHostPorts"},
    {"name" : "PodFitsResources"},
    {"name" : "NoDiskConflict"},
    {"name" : "NoVolumeZoneConflict"},
    {"name" : "MatchNodeSelector"},
    {"name" : "HostName"}
    ],
"priorities" : [
    {"name" : "LeastRequestedPriority", "weight" : 1},
    {"name" : "BalancedResourceAllocation", "weight" : 1},
    {"name" : "ServiceSpreadingPriority", "weight" : 1},
    {"name" : "EqualPriority", "weight" : 1}
    ],
"hardPodAffinitySymmetricWeight" : 10 
自定义Priority和Predicate

自定义Priority的实现过程

1.接口位置：kubernetes/pkg/scheduler/algorithm/types.go

// PriorityFunction is a function that computes scores for all nodes.
// DEPRECATED
// Use Map-Reduce pattern for priority functions.
type PriorityFunction func(pod *v1.Pod, nodeNameToInfo map[string]*schedulercache.NodeInfo, nodes []*v1.Node) (schedulerapi.HostPriorityList, error)
2.在kubernetes/pkg/scheduler/algorithm/predicates/predicates.go文件中编写对象是实现上述接口



3.注册自定义过滤函数   kubernetes/pkg/scheduler/algorithmprovider/defaults/defaults.go



4.--policy-config-file将自定义Priority加载进来即可

自定义调度器：

利用Pod的spec.schedulername字段来指定调度器，默认为default

#!/bin/bash
SERVER='localhost:8001'
while true;
do
for PODNAME in $(kubectl --server $SERVER get pods -o json | jq '.items[] | select(.spec.schedulerName == "my-scheduler") | select(.spec.nodeName == null) | .metadata.name' | tr -d '"')
;
do
    NODES=($(kubectl --server $SERVER get nodes -o json | jq '.items[].metadata.name' | tr -d '"'))
    NUMNODES=${#NODES[@]}
    CHOSEN=${NODES[$[ $RANDOM % $NUMNODES ]]}
    curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind": "Binding", "metadata": {"name": "'$PODNAME'"}, "target": {"apiVersion": "v1", "kind"
: "Node", "name": "'$CHOSEN'"}}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
    echo "Assigned $PODNAME to $CHOSEN"
done
sleep 1
done 
Pod优先级（Priority）和抢占（Preemption）

pod priority指的是Pod的优先级，高优先级的Pod会优先被调度，或者在资源不足低情况牺牲低优先级的Pod，以便于重要的Pod能够得到资源部署.

定义PriorityClass对象：

apiVersion: scheduling.k8s.io/v1alpha1
kind: PriorityClass
metadata:
name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
在Pod的spec. priorityClassName中指定已定义的PriorityClass名称

apiVersion: v1
kind: Pod
metadata:
name: nginx
labels:
env: test
spec:
containers:
- name: nginx
image: nginx
imagePullPolicy: IfNotPresent
priorityClassName: high-priority
当节点没有足够的资源供调度器调度Pod、导致Pod处于pending时，抢占（preemption）逻辑会被触发。Preemption会尝试从一个节点删除低优先级的Pod，从而释放资源使高优先级的Pod得到节点资源进行部署。


参考地址：

http://dockone.io/article/2885

https://github.com/kubernetes/kubernetes/blob/master

1.基于SLA驱动的资源动态调度算法
将应用分为不同类型，将不同应用调度到不同资源状态节点上，减少应用因资源不足带来的问题，根据SLA协议实时监控应用资源使用状况，动态调整应用资源占用率，提高资源使用率。

SLA协议：Service-Level Agreement的缩写，意思是服务等级协议。是关于网络服务供应商和客户间的一份合同，其中定义了服务类型、服务质量和客户付款等术语。

监控容器在一段时间内资源使用率数据，使用拟合函数来拟合数据（机器学习回归），通过拟合函数来计算容器虚拟资源到达使用率上限的时间，根据时间对容器进行排序。

1. tj时刻容器i的cpu使用率cij, 内存使用率mij

    

2.线性回归得到cpu使用率与时间的拟合函数Ztjc  预测容器icpu使用率到达上限Ciu的时间ticu,令Ztjc=Ciu,解得ticu

3.根据每个容器的ticu升序排列，得到排序方案Sc，容器i在排序方案中的序列号为Sic

4.求得内存使用率与时间的拟合函数Ytjm    预测内存使用达到上限Miu的时间timu 令 Ytjm=Miu  得到timu

5.根据每个容器的timu升序排列，得到排序方案Sm,容器i在排序方案中的序列号为Sim

6.对于排序方案Sc, Sm进行整合：



7.考虑SLA处理问题的优先级pre



S为最终排序，每次选择S中第一位作为资源调度的目标

8.根据容器资源使用率的下限，确定容器资源是否可以回收

一段时间内cpu的平均使用率c`i  内存平均使用率m`i

cid  mid为cpu,内存使用率下限   ciu  cmu  cpu内存使用率上限

Ci已经分配的资源   deltaC 要回收的资源



物理机节点K可分配的资源

若任务所需资源小于物理机节点的资源，则执行节点内调度，否则执行节点间调度。

总结一下，利用容器的cpu,内存的预测情况以及SLA协议来对容器进行排序，根据资源使用率上下限进行节点内以及节点间的调度

参考论文：

[1]边俊峰. 基于Docker的资源调度及应用容器集群管理系统设计与实现[D].山东大学,2017.
2.基于Kubernetes-on-EGO的两级资源调度器
目前的 Kubernetes 调度器是单进程的同步调度，当系统调度任务大量增加时，调度器会发生阻塞延时，不能满足调度效率高的要求；调度计划是不对外暴露的，不能满足大规模数据中心中用户自己通过配置调配资源的问题。第一个问题应该是存在的，第二个问题现在应该支持了吧，需要研究一下。

单体调度：

单体式调度指调度器是单一进程运行在某台主机上，调度器经过内置的算法将任务分派给集群中的某台主机，所有任务都服从一个调度器的安排，这种架构简单，但是这种调度器存在着缺陷，对于长时间运行的服务如 Web 服务和批处理任务如 Map Reduce 任务没有区分调度；应对不同场景，调度器应有不同的调度逻辑；调度器是单进程设计，在集群规模增大调度任务增多时，调度会导致队列阻塞。

二级调度;

二级调度是指资源调度和任务调度分离，这样任务调度逻辑就可以更加灵活，根据不同的应用需求配置不同的调度逻辑，集群之间资源还可以保证处于共享状态。Mesos 使用二级调度架构，在 Mesos 中资源会主动提供给上层应用层调度，在二级调度逻辑，待调度的任务跟资源管理器交互，资源管理器主动给任务分配动态资源。用户能够灵活使用调度策略。二级调度也存在问题：应用层调度隐藏了全局资源调度，调度器无法看到全局资源的配置情况，只看到资源管理器提供的资源，因此会导致资源调度不合理，降低资源的使用效率。



首先用户创建 Pod，向系统提交创建 Pod 的请求，Kubernetes 组件 APIServer接受用户的资源请求，将资源请求的信息逻辑层存储到 
Etcd 中保存，二级调度系统的上层调度也就是一级调度首先接受并初始处理用户的资源请求，比如将用户的资源格式化为调度系统所能识别的资源格式，这里会将 Memory 的资源请求除以 1000，并进行取整，以此来增大资源的利用率；上次调度接受用户请求并进行处理后，接下来进入下层的二级调度系统，首先登陆到 EGO 的系统中，登陆成功则进行 Client 的信息注册，进而进入最主要的资源分配阶段，首先进行资源分配，调度系统查找资源池中可用的资源，查找到可用的资源则进入到构建调度结果阶段，将可用的资源信息，创建 
Pod 的信息，用户注册的 Client 信息组织到一起，而后调度系统读取调度结果，最后将调度结果返回给整个云平台系统，本次任务调度资源分配结束，调度系统成功为用户分配到可用的资源。接下来就是Kubernetes 系统得知调度结束，使用系统分配的资源为用户的任务启动服务。



k8s自身的调度系统是接受待调度任务，然后根据待调度任务资源情况以及物理机节点情况来进行进行节点选择，最终执行任务调度。二级调度系统是将资源分配与任务调度分离，资源调度系统为任务分配资源，根据分配结果再进行任务调度。通过将资源调度和任务调度分离，使得资源调度提高调度效率，下层任务调度可以灵活的接入用户定义的任务调度算法和调度机制，这使得任务调度逻辑不仅可以根据不同应用要求而进行裁剪，而且保留了在集群之间共享资源的可能性。

参考论文：

[1]台慧敏. 基于Kubernetes-on-EGO的两级资源调度器的设计与实现[D].西安电子科技大学,2017.
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84422613

# Kubernetes4--动态调度

基于Kubernetes的容器云平台资源调度策略
调度，给定的N种系统资源，给定一系列任务列表，如何分配各种资源，使得系统的总资源利用达到某种目标，或者资源利用率最高，资源利用率最高等。容器与物理机的调度，主要是物理机有一定的各种资源限额，容器服务有各种不同的资源需求，如何分配这些容器服务分配到指定的物理机节点实现系统功能，以达到系统的资源利用等目标。

调度思想大致分为两种  扩散以及贪心。

扩散：尽量将服务调度到不同的集群节点上以此来保证资源的均衡利用率，避免单机故障的风险

贪心：尽量将所有服务调度到同一节点上，提高资源的利用率

集群资源调度系统分类
集中式调度系统

Swarm Borg 集群使用同一的调度算法进行调度，调度任务不可以并行执行，调度信息需要存储在存储器，任务较重延时较高。Scheduler 支持 Random、Spread 和 Binpack 三种调度策略，其中 Random 策略随机选择目标节点进行 Docker 调度；Spread 和 
Binpack 则根据节点可用的 CPU、存资源量和节点上当前运行的 Docker 数量做出调度决策，Spread 将 Docker 分散调度到各个节点上，以提高系统的整体负载均衡；Binpack 与 Spread 相反，将 Docker调度到负载最高的节点上，提高集群中节点承载容器的密度。

​                                      

两层式调度系统

Memos  YARN 资源管理系统负责资源的统一分配和利用，任务调度系统可以执行不同的调度策略来并行调度任务

​                                           

Mesos Master实现资源的统一分配和管理，调度框架可以并行调度任务。其使用DRF算法来实现对不同类型作业的不同资源需求公平分配。该算法的核心思想是在资源类型多样的环境下，一个作业的资源分配量由作业的主导份额资源决定，主导份额资源是作业请求的各类型资源中占据资源量最大的一种资源。

​                                            

共享式调度系统

Kubernetes 支持使用不同的调度框架执行不同类型的任务，它的核心是共享状态。为了提高调度系统的并发性和可扩展性，共享式调度系统使用乐观锁进行并发控制，集群的相关信息都增加了版本号，在提交的时候与当前数据的版本号进行对照，若提交的状态信息版本号比当前信息的版本号高，则允许此次提交，否则决绝提交请求，这样虽然会增加资源请求的冲突几率，但是实际应用证明，系统的整体性能并不会因为这些冲突而整体下降。

k8s的Scheduler调度思想：
通过调度策略（算法）依次为待调度 Pod 队列的每一个 Pod 从可用 Node 队列中选择一个最合适的作为宿主机。目标节点上的 Kublet 进程通过 API  Server 监听到 Kubernetes 调度器发出的 Pod 绑定事件，然后从 etcd 中获取对应的 Pod 资源描述文件，下载 Image 镜像，启动容器，挂载持久化存储系统。

​                          

kubernetes主要使用Pod来作为实际的服务对象，其Pod调度流程如下：

​            

k8s的默认调度策略分析：

​                           

默认调度过程如下：

首先，客户端通过API Server的REST API/kubectl/helm创建pod/service/deployment/job等，支持类型主要为JSON/YAML/helm tgz。
接下来，API Server收到用户请求，存储到相关数据到etcd。
调度器通过API Server查看未调度（bind）的Pod列表，循环遍历地为每个Pod分配节点，尝试为Pod分配节点。调度过程分为2个阶段：
第一阶段：预选过程，过滤节点，调度器用一组规则过滤掉不符合要求的主机。比如Pod指定了所需要的资源量，那么可用资源比Pod需要的资源量少的主机会被过滤掉。
第二阶段：优选过程，节点优先级打分，对第一步筛选出的符合要求的主机进行打分，在主机打分阶段，调度器会考虑一些整体优化策略，比如把容一个Replication Controller的副本分布到不同的主机上，使用最低负载的主机等。
选择主机：选择打分最高的节点，进行binding操作，结果存储到etcd中。
所选节点对于的kubelet根据调度结果执行Pod创建操作。
k8s的调度优化
k8s的默认指标使用物理机的CPU以及内存等指标来计算物理节点的虚拟性能状态，将Pod对象调度到某个最合适的Node物理节点

物理节点性能指标优化：

可以考虑物理机的cpu，内存，IO，网络，响应时间，链接数等系统资源

论文中主要使用CPU，内存，镜像下载速度，数据传输速度四个指标
       镜像下载速度：宿主机需要到 Pod 资源描述文件指定的网络地址下载 Pod 中所有容器的镜像文件，宿主机与镜像存储系统之间的网络传输速度直接关系到 Pod 的启动速度的快慢；这个问题可以通过搭建私有镜像仓库来解决，但是集群规模过大还是会存在的。
      数据传输速度：Pod 中数据是临时的，当 Pod 销毁时，其中的数据会丢失，所以 Pod 需要通过数据卷的方式将数据持久化。因此 Pod 在启动运行后，Pod 还需要挂载持久化存储系统进行数据的存取，宿主机与持久化存储之间的数据传输速度会直接影响 Pod中运行的应用的 IO 速度。

用户绑定策略：

论文中提出了各种绑定策略，即指定Pod到某个确定的Node,现在版本应该已经支持很好,可以使用Label标签实现

抢占式调度策略：

现在版本支持Pod优先级抢占，通过PriorityClass来实现同一个Node节点内部的Pod对象抢占。根据 Pod 中运行的作业类型判定各个 Pod 的优先级，对于高优先级的 Pod 可以抢占低优先级 Pod 的资源。Pod priority指的是Pod的优先级，高优先级的Pod会优先被调度，或者在资源不足低情况牺牲低优先级的Pod，以便于重要的Pod能够得到资源部署.

定义PriorityClass对象：

apiVersion: scheduling.k8s.io/v1alpha1
kind: PriorityClass
metadata:
name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
在Pod的spec. priorityClassName中指定已定义的PriorityClass名称

apiVersion: v1
kind: Pod
metadata:
name: nginx
labels:
env: test
spec:
containers:
- name: nginx
image: nginx
imagePullPolicy: IfNotPresent
priorityClassName: high-priority
当节点没有足够的资源供调度器调度Pod、导致Pod处于pending时，抢占（preemption）逻辑会被触发。Preemption会尝试从一个节点删除低优先级的Pod，从而释放资源使高优先级的Pod得到节点资源进行部署。

论文中按照Pod的重启策略来定义pod优先级，Always>OnFailure>Never

利用容器资源请求量使用优先级队列来进行排序   OnFailure队列   Never队列

Always可以抢占Never，OnFailure资源，依次回收低优先级的Pod对象资源直到宿主机资源满足Always的资源请求

OnFailure可以抢占Never资源  

​                                                    

动态负载均衡策略
依据Node节点的衡量指标，定量度量Node节点性能情况，建立高负载队列以及低负载队列，根据系统的实施运行情况来不断调整不同Node节点之间的容器对象，实现资源的均衡利用，主要是扩散思想。

优选策略的设计

假定服务器集群N台，其物理集群定义如下：

​                                                     

各物理节点的资源规格，定义各物理节点的性能度量指标：

​                                                                

其中          ci, mi分别为cpu以及内存的资源配额。

各物理节点的规格系数向量：

​                                                               

其中        分别为cpu以及内存的规格系数  例cpu2核，内存4G  其表示规格系数  2   4

第i个物理节点的资源规格：

​                                                        

集群的资源规格系数：

​                                                        

综合考虑 CPU、内存、镜像网络下载速度、持久化存储网络传输速度四方面因素来决定 Pod 的调度目的地。

计算集群的cpu负载均值，及集群的cpu利用率的加权平均值：

                                                                        为第i个节点的cpu规格系数

集群的内存负载均值：

​                                                               

集群镜像网络的传输均值：

​                                                                

集群数据网络的传输均值：

​                                                                   

计算第i个物理节点的各种指标得分：

​                                             

 

​                                     

 

利用均值与节点值的比值作为节点的衡量分数，当该比值越大时表示该节点越低于平均性能，即节点的性能越优。节点相对于集群整体的相对负载情况。

各种指标的权值因子：

​                                                                          

该物理节点的性能综合得分：各种指标的加权平均值取对数

​                                                   

值大于零表示，节点的负载低于平均值   小于零表示节点的负载高于平均值

负载队列的实现

建立高负载队列以及低负载队列，得分大于 0 的表示综合负载相对于集群较空闲，存储在低负载队列，小于 0 表示综合负载相对于集群较重，存储在高负载队列。通过周期检测集群的负载情况，将负载较高的节点的一些 Pod 迁移到负载较低的节点上，以保证集群的负载均衡。

利用二叉堆来实现，高负载队列使用最大堆，堆顶元素即为负载最大的Node节点；低负载队列使用最小堆，堆顶元素即为负载最小的Node节点。根据调节阈值，将高负载队列上的容器服务动态调度到低负载队列上，以保证集群的整体资源利用的均衡。

动态控制流程

​                                                         

总体调度算法的设计：

​                                        

 动态调度的具体实现：

​                                      

算法的负载均衡度的衡量：使用Node集群得分的标准差

​                                                                   

总结一下：

同一个Node节点内部实行抢占式调度  根据pod重启策略来定制优先级  优先级高可以抢占优先级低的资源

不同Node节点执行动态调度，衡量每个Node节点分数，建立高负载队列以及低负载队列，然后将高负载队列的pod对象迁移到低负载队列以此来实现资源的均衡利用。

参考论文如下，这篇论文写的真好，学习了。

[1]唐瑞. 基于Kubernetes的容器云平台资源调度策略研究[D].电子科技大学,2017.
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84471489

# Kubernetes5--弹性伸缩1

参考论文：

[1]杨茂. 基于Kubernetes的容器自动伸缩技术的研究[D].西安邮电大学,2018.

Kubernetes的弹性伸缩技术



        k8s提供了Service虚拟服务概念，对应于实际的服务提供者Pod集群，可以使用ReplicationController来实现Service对应一个Pod集群，服务入口对应于Service端口，通过Service负载到Pod集群，同时利用HPA策略来实现对于Pod集群数量的动态调整，扩容或者缩容。

k8s的HPA策略分析：

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

利用cpu利用率与预定的cpu利用率来进行伸缩



扩容阶段问题：

Kubernetes 自动伸缩策略属于一种响应式缩，其实质是根据 Pod 副本当前负载情况计算伸缩量。对于扩容阶段，当 Pod 副本负载过高触发 HPA 进行扩容时，HPA 计算出期望副本数并通知副本控制器，副本控制器根据期望值计算出需要创建 Pod 的数量，并交给相应组件执行具体的创建工作。因此从 Pod 副本负载过高触发扩容到新副本分担服务请求存在一定响应延迟。



在扩容时需要将 Pod 初始化的时间考虑进去，在负载高峰到来之前做好扩容准备，减少用户的请求响应时间，从而保障服务质量的稳性。

缩容阶段问题：

自动缩容的目的在于当应用 Pod 副本集负载较低时，通过删除冗余副本提高副本集的整体资源利用率，降低应用的部署成本。

缩容策略不仅要计算删除的副本量，还要考虑如何在副本集中选择合适的副本作为待删副本。

首先根据副本集中各副本所处状态的不同赋予不同的优先级，根据优先级对副本进行排序，最后选择优先级最小的副本作为待删副本。

一方面可能导致集群节点资源分配不平衡，造成部分节点 Pod 聚集，节点上的 Pod 相互之间竞争资源导致执行效率下降。部分节点 
Pod 分布很少，节点上的资源处于闲置状态，造成资源浪费。另一方面删除 Pod 可能会造成节点上各维度资源使用不平衡，产生资源碎片问题。不同类型 Pod 对节点上不同维度资源需求存在很大的差异性。如果节点在删除某一 Pod 副本后剩余的各维度资源很难满足其他类型的 Pod 需求，而被删除类型的 Pod又长时间得不到扩容，或者该类型的 Pod 以后都不会再出现，那么该节点剩余的资源将长时间不能再被再次利用，产生资源碎片。对于长期运行的集群而言，资源碎片的累积会导致集群的整体资源利用率的降低。

基于负载预测的扩容策略：

预测式伸缩通过预测副本集未来的负载状况，在负载高峰到来之前完成扩容，从而避免因 Pod 初始化过程带来的响应延迟，保障了副本集的负载始终处于目标状态。

负载预测是通过总结已有的历史负载数据来对未来数据进行预测。历史负载数据是一段按照其发生时间排列成的数列，属于一种典型的时间序列数据。可以利用时间序列预测算法进行预测。此外负载的变化在不同时间会有不同的表现，因此对于负载数据的预测既要考虑其稳定性也要考虑其突发性。
指数平滑法进行负载预测：

一种短期时间序列的预测方法，具有计算量少、预测精度高的优点，同时该方法能充分利用全部历史数据，并且按照“重近轻远”的原则进
行加权平均和修匀数据，能够较好地将时间序列所包含的历史规律挖掘出来。
{yt}原始时间序列，对原始时间序列进行一次指数平滑：



进行二次指数平滑：



预测方程：





灰色预测模型：

灰色系统理论以“部分样本信息已知，部分样本信息未知”的“小样本”来进行预测的算法模型，该模型主要通过对部分未知信息的生成来开发和提取有价值的信息，从而实现对系统运行规律的正确认识和确切描述。基于灰色系统理论 GM(1,1)模型的预测方法称为灰色预测方法。灰色预测方法对于样本量小、变化规律不明显的样本有很好的适应性，同时灰色预测法的计算量小而且其定量分析结果和定性分析结果保持一致。

样本数据：



一次累加序列：





紧邻均值序列：





白花方程：



设 最小二乘法参数求解：





计算X1预测值：



对于X0进行还原：



精度检验：



基于灰色-指数平滑组合模型的负载预测:

利用指数平滑预测负载变化情况，总结之前的负载突变情况序列进行灰色建模，用突变情况的突变预测值取代相应平滑预测值作为最终结果返回，根据指数平滑法预测值与真实负载值相对误差来判断负载数据是否属于异常负载



扩容实现：



监控模块heapster+cadvisor来实现：



预测模块实现：



基于集群负载均衡的缩容策略：

在缩容时预先计算删除副本后所在节点的资源利用率，并将其作为该副本的资源权值。



基于节点多维度资源平衡的缩容策略

如果节点在删除某一 Pod 副本后造成某一维度上的资源剩余量难以满足其他类型的 Pod 需求，那么即使其他维度资源有很大剩余量，该节点都难以部署新 Pod。如果被删除类型的 Pod 长时间不扩容甚至该类型的 Pod 以后都不会再部署到集群中，那么节点的这部分剩余资源将很难再被重新利用。这些难以满足部署需求的资源碎片如果大量积累，将会造成集群资源的极大浪费。因此在删除 Pod时需要考虑到节点内各维度资源的平衡，节点各维度资源越平衡，其产生资源碎片的可能性往往越小。



总结：

弹性伸缩：

基于负载预测提前进行集群扩容，进而降低集群响应时间

基于负载均衡以及节点多维度资源平衡进行缩容，保证服务质量，减少集群数量，降低资源消耗。
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84528572

# Kubernetes6--资源调度

参考论文：

[1]杜军. 基于Kubernetes的云端资源调度器改进[D].浙江大学,2016.
k8s主要应用Pod这一概念进行应用部署和操作，以后是pod对象的创建时序图，pod对象的创建效率直接影响系统的整体性能。



优先级分类与抢占式调度：

按照Pod对象的重启策略分为低优先级以及高优先级

利用双端循环队列，维护队首指针以及队尾指针

Pod对象重启策略为Always加入到队首，优先调度

Pod对象重启策略为OnFailure Never加入到队尾  如果Pod.spec.Host值为非空，说明直接已经调度过需要重新调度，则加入到队首，优先调度

资源配额动态调整调度：

资源申请配额   资源实际使用  利用率较低

动态调整Pod资源的配额,例如内存以及cpu等，pod对象实际使用内存超过申请额度，会被杀死。在此只动态调整cpu的额度。

利用linux  CGroup技术来实现资源配额限制

支持动态调度和负载均衡的云资源管理平台D-Cloud的设计与实现

资源动态调度：

参考论文：[1]杨鹏飞. 基于Kubernetes的资源动态调度的研究与实现[D].浙江大学,2017.

主要对容器层资源（内存使用量、ＣＰＵ使用率、网络带宽和磁盘１０等资源），使用预测模型预测未来一段时间的资源使用量，并
基于预测数据对资源动态调度、实例自动伸缩和考虑资源敏感程度的负载均衡策略等方面。

资源使用量预测模型：

ＡＲＩＭＡ模型也称为差分自回归移动平均模型，能够运用于随机行时间序列预测，在时间序列分析方式中，被认为是最高级、最有效的分析方法。

ＡＲＩＭＡ模型的思想是：在时间序列中，单个的值虽然具有随机性和不确定性，但整个时间序列却有规律可寻，可以用ＡＲＩＭＡ模型进行类似的描述。

ＡＲＩＭＡ模型本质上是三种模型的组合，分别是：

用于处理线性问题的ＡＲ（ｐ）模型（自回归模型）

加入平均滑动的ＭＡ（ｑ）模型（移动平均模型）

针对平稳时间序列的ＡＲＭＡ（ｐ，ｑ）模型（自回归移动平均模型）

AR模型：某一时刻的值yt与过去p个时刻有关

     零均值的白噪声

MA模型：某一时刻的值yt与过去p个时刻的残差序列加权平均值组成



ARMA模型：时间序列对于任意时刻ｔ的值ｘ不仅和历史值有关，还跟外部的干扰有关，并且两者还存在一定的依存关系，在进行数据拟合时，为了保证模型有更大的灵活性，在模型中增加了移动平均部分，并结合自回归模型功能组合成自回归移动平均模型。





ARIMA+RBF模型：

时间序列可以分解成线性关系和非线性关系


用ARIMA对数据进行线性建模，得到预测值Lt,预测值与实际值的误差为：



误差序列隐含了非线性关系。

ＲＢＰ网络模型对误差值序列建模进行数据训练，最终来逼近时间序列中的非线性关系

误差值预测序列：





监控模块：



资源动态调度模块：



总结：

提供对于资源的预测算法

Pod调度到Node节点时，需要考虑多种因素，改进调度器

弹性伸缩的设计
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84561247

# Kubernetes7--Dashboard安装

系统已经搭建好1Master4Node节点，现在安装一下Dashboard面板

镜像仓库可以参考如下：

https://hub.docker.com/r/keveon/kubernetes-dashboard-amd64/tags/

Master节点

下载最新的dashboard镜像

docker pull keveon/kubernetes-dashboard-amd64:v1.10.0

将镜像名字改一下：

docker tag docker.io/keveon/kubernetes-dashboard-amd64:v1.10.0 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
docker rmi docker.io/keveon/kubernetes-dashboard-amd64:v1.10.0

kubectl get pods --all-namespaces



研究一下部署文件：

https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

k8s的RBAC权限控制体系：

https://github.com/kubernetes/dashboard/wiki/Creating-sample-user

安装如下，参考地址：

https://github.com/kubernetes/dashboard/wiki/Installation#recommended-setup

dashboard安装可以采用自己申请证书，使用https服务方式：

https://github.com/kubernetes/dashboard/wiki/Installation#recommended-setup

可以使用自带签名证书https服务，通过kubecatl proxy来访问

这里通过采用http服务来构建，没有安全认证，不安全

获取yaml文件：

wget  https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/alternative/kubernetes-dashboard.yaml

修改service为NodePort方式暴露30002端口，以提供外网访问：



运行文件



显示权限不够：



看一下RABC（角色访问控制机制）：

RBAC
K8S 1.6引进，是让用户能够访问 k8S API 资源的授权方式【不授权就没有资格访问K8S的资源】
用户
K8S有两种用户：User和Service Account。其中，User给人用，Service Account给进程用，让进程有相关权限。如Dashboard就是一个进程，我们就可以创建一个Service Account给它
角色
Role是一系列权限的集合，例如一个Role可包含读取和列出 Pod的权限【 ClusterRole 和 Role 类似，其权限范围是整个集群】
角色绑定
RoleBinding把角色映射到用户，从而让这些用户拥有该角色的权限【ClusterRoleBinding 和RoleBinding 类似，可让用户拥有 ClusterRole 的权限】
Secret
Secret是一个包含少量敏感信息如密码，令牌，或秘钥的对象。把这些信息保存在 Secret对象中，可以在这些信息被使用时加以控制，并可以降低信息泄露的风险
看一下上述配置文件的RABC设置信息：

声明了一个ServiceAccount:



声明了一个Role:



进行角色绑定：



kubernetes-dashboard-minimal角色权限最低，无法显示页面信息，现在更换角色：
将角色更换为cluster-admin角色，进行clusterRoleBinding



重新启动服务：kubectl apply



可以正常访问

参考地址：

https://github.com/kubernetes/dashboard/releases

https://github.com/kubernetes/dashboard

https://github.com/kubernetes/dashboard/wiki/Installation#recommended-setup

https://blog.csdn.net/java_zyq/article/details/82178152

https://www.centos.bz/2018/07/kubernetes%E7%9A%84dashboard%E7%99%BB%E5%BD%95%E6%96%B9%E5%BC%8F/
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84632628

# Kubernetes8--Centos基础镜像使用

目前已经搭建好k8s集群，一个Master节点，4个Node节点。内存资源总共约600G,需要进行虚拟化，资源的隔离与限制方便使用。

使用Centos基础镜像虚拟出若干linux镜像,方便服务部署等使用。

Centos基础镜像位置：

https://hub.docker.com/_/centos/

拉取镜像：

docker pull centos



基础镜像的使用：



基础镜像安装ssh服务：





启动sshd服务：



生成ssh验证文件：

[root@46506dfd57bb /]# ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ""
Generating public/private rsa key pair.
Your identification has been saved in /etc/ssh/ssh_host_rsa_key.
Your public key has been saved in /etc/ssh/ssh_host_rsa_key.pub.
The key fingerprint is:
SHA256:C+AfCEsdndDOzDBYCLZsgKfIp+zQ6Rppude+pCC14mc root@46506dfd57bb
The key's randomart image is:
+---[RSA 2048]----+
|+o +++ .         |
|= =.o.+          |
|o*o oB           |
|+o = o=          |
|..=.o o S        |
|.=+. . o .       |
|B=. ... .        |
|+++Eo.           |
|.++..o.          |
+----[SHA256]-----+
[root@46506dfd57bb /]# ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ""
Generating public/private ecdsa key pair.
Your identification has been saved in /etc/ssh/ssh_host_ecdsa_key.
Your public key has been saved in /etc/ssh/ssh_host_ecdsa_key.pub.
The key fingerprint is:
SHA256:qu+Qlnr6gSAyr/9A+NBAd53ZvHXoGjIq76ot8I51rKg root@46506dfd57bb
The key's randomart image is:
+---[ECDSA 256]---+
| . . .. =   .    |
|. . .  + o o .   |
|.         + .    |
| +     o o .     |
|B o   . S o      |
|+B + + . .       |
|..* @ .          |
| *o*.=           |
|E+OB*+o          |
+----[SHA256]-----+
[root@46506dfd57bb /]# ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""
Generating public/private ed25519 key pair.
Your identification has been saved in /etc/ssh/ssh_host_ed25519_key.
Your public key has been saved in /etc/ssh/ssh_host_ed25519_key.pub.
The key fingerprint is:
SHA256:gwRq6UtfORwQ1Rb2YtBYngD/v7AhmYoZNtuN6ZdGoLQ root@46506dfd57bb
The key's randomart image is:
+--[ED25519 256]--+
|    =++=+.       |
|   o +.=+o       |
|  +   +.= .      |
| o. .o * .       |
| .oo .* S        |
| .Eo ..+ o       |
|  = ..+.o .      |
| . B =+. + .     |
|  +.*o. . .      |
+----[SHA256]-----+
修改/etc/ssh/sshd_config  中：

将配置文件中原本UsePAM yes换成UsePAM no

允许远程root登录

[root@46506dfd57bb ssh]# vi sshd_config 
[root@46506dfd57bb ssh]# passwd root
Changing password for user root.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
退出容器，将容器提交为新基础镜像：

[root@xjs-dn04 ~]# docker commit 465 docker.io/chenwenkai123456/centos_ssh
sha256:119bc03d6dc591a65455bbfe62d5a05452f76ba8486ca7be130c24cb67f44f2d
[root@xjs-dn04 ~]# docker images
REPOSITORY                                                           TAG                 IMAGE ID            CREATED             SIZE
docker.io/chenwenkai123456/centos_ssh                                latest              119bc03d6dc5        3 seconds ago       305 MB
在docker hub中申请一个账号，用来存储镜像，也可以搭建自己的私有仓库。https://hub.docker.com/

[root@xjs-dn04 ~]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: chenwenkai123456
Password: 
Login Succeeded
镜像上传到docker hub中：

[root@xjs-dn04 ~]# docker push docker.io/chenwenkai123456/centos_ssh:latest
The push refers to a repository [docker.io/chenwenkai123456/centos_ssh]
ae1efa863646: Pushed 
f972d139738d: Pushed 
latest: digest: sha256:2716209fc110e41460a0377673c0c495041294ccc386b8cd7baf00a077feb0d6 size: 741
此时可以使用docker  pull命令来拉取刚才的镜像。

启动新容器：

[root@xjs-dn04 ~]# docker run -d -p 2222:22 docker.io/chenwenkai123456/centos_ssh:latest /usr/sbin/sshd -D
4aa4b16063de1074a4d9c875a9b35511d91de3cefec173251fa6b4b694a4520f
[root@xjs-dn04 ~]# docker ps
CONTAINER ID        IMAGE                                                                                                                                        COMMAND                  CREATED             STATUS              PORTS                  NAMES
4aa4b16063de        docker.io/chenwenkai123456/centos_ssh:latest                                                                                                 "/usr/sbin/sshd -D"      3 seconds ago       Up 3 seconds        0.0.0.0:2222->22/tcp   peaceful_banach
登录测试ssh:

[root@xjs-dn04 ~]# ssh root@127.0.0.1 -p 2222
root@127.0.0.1's password: 
[root@4aa4b16063de ~]# uname -r
3.10.0-327.el7.x86_64
[root@4aa4b16063de ~]# hostname
4aa4b16063de
服务器的内网ip为192.168.1.16  外网ip为10.3.10.149   通过防火墙NAT映射实现端口流量转发

10.3.10.149:2222-------192.168.1.16:2222

在16服务器使用ssh,结果出现内网可以ssh登录，外网不可以ssh登录，这里外网实际上也是公司的内网。

ssh root@10.3.10.149 -p 2222

ssh root@192.168.1.16 -p 2222

服务器的网络

内网到外网：  192.168.1.16---192.168.1.1--10.3.10.138（防火墙IP）---10.3.10.98(专线网关)---专线上网

外网到内网：   外网IP---公司网关---10.3.10.254----10.3.10.138(NAT映射)---192.168.1.16

在16服务器内部无法使用外网ip登录ssh,结果在自己电脑上可以登陆：



电脑IP   10.3.27.50 ---10.3.27.254----10.3.10. 254---10.3.10.138---192.168.1.16

现在由内向外以及由外向内使用两条不同线路，因此网络可能有点错乱。

docker run --name centos-ssh -d -p 2222:22 docker.io/chenwenkai123456/centos_ssh:latest /usr/sbin/sshd -D

docker run --name centos-ssh -d -p 2222:22 docker.io/chenwenkai123456/centos-ssh:v1 /usr/sbin/sshd -D

现在根据以上镜像来使用k8s部署启动：

定义Pod以及Service对象：

[root@Ac-private-1 centos]# cat centos-rc.yaml 
---
kind: ReplicationController
apiVersion: v1
metadata:
  name: centos-ssh
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: centos-ssh
    spec:
      containers:
      - name: centos-ssh
        image: docker.io/chenwenkai123456/centos-ssh:v1
        ports:
        - containerPort: 22
          protocol: TCP
        command: ["/usr/sbin/sshd","-D"]
[root@Ac-private-1 centos]# cat centos-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: centos-svc
spec:
  ports:
  - name: centos-svc
    port: 22
    targetPort: 22
    nodePort: 30005
    selector:
    app: centos-ssh
    type: NodePort 
    结果发现ssh登录失败，好奇怪。



使用直接docker  run centos镜像可以ssh成功，使用k8s  yaml方式部署失败，可能是yaml配置有问题。

总感觉是公司网络ip产生冲突，内部IP使用dhcp来分配，但是服务器IP是static的，这样可能会冲突，当从外网访问内网时，可能会冲突。

更换镜像： 直接支持ssh登录

docker pull kinogmt/centos-ssh

启动容器：

docker run --name ct -p 22222:22 -d docker.io/kinogmt/centos-ssh

内网可以登陆，外网不可以：

[root@xjs-dn04 k8s]# ssh root@192.168.1.16 -p 22222
root@192.168.1.16's password: 
[root@9f77646b277e ~]# exit
logout
Connection to 192.168.1.16 closed.
[root@xjs-dn04 k8s]# ssh root@10.3.10.149 -p 22222
ssh: connect to host 10.3.10.149 port 22222: Connection refused
感觉是IP冲突

使用k8s更换镜像来部署，发现内外网均不可以，k8s部署方式配置不正确。

仍要解决的问题：

ssh登录

内存，cpu，硬盘等资源的隔离与配额限制

nodeSelector将pod调度到指定node

使用deplyment来优化k8s部署方式

参考链接：

https://blog.csdn.net/u012767761/article/details/78107870

https://www.jianshu.com/p/e38c05cf076a?spm=a2c4e.11153940.blogcont508898.10.43bd25101X52jj
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84746457

# Kubernetes10--API访问控制

使用API来访问k8s集群，方便使用第三方客户端来访问和管理k8s集群资源，在此准备使用Java语言来封装API使用。

1.API Server 配置文件解析


如上图所示，k8s集群中主要使用Api Server来作为系统通信的中心，因此使用api相当于主要与api server来进行信息交互。

在此之前使用kubeadm方式来部署k8s集群，查看一下apiserver的yaml文件：

master节点    /etc/kubernetes/manifests/kube-apiserver.yaml

- command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
    - --advertise-address=192.168.1.15
    - --allow-privileged=true
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    认证模式： authorization-mode=Node,RBAC

apiserver 接口地址： secure-port=6443

安全模式，https服务：证书tls-cert-file=/etc/kubernetes/pki/apiserver.crt   密钥tls-private-key-file=/etc/kubernetes/pki/apiserver.key

开启了boostrap-token认证服务   enable-bootstrap-token-auth=true，后面主要使用该方式进行认证

非安全访问关闭：  insecure-port=0  如果开启不安全  这里由于关闭了http服务，如果开启需要重启apiserver不安全。

默认情况下，Kubernetes API Server在2个端口上提供HTTP：

Localhost Port：
 - is intended for testing and bootstrap, and for other components of the master node
   (scheduler, controller-manager) to talk to the API
 - no TLS
 - default is port 8080, change with `--insecure-port` flag.
 - defaults IP is localhost, change with `--insecure-bind-address` flag.
 - request **bypasses** authentication and authorization modules.
 - request handled by admission control module(s).
 - protected by need to have host access
Secure Port：
 - use whenever possible
 - uses TLS.  Set cert with `--tls-cert-file` and key with `--tls-private-key-file` flag.
 - default is port 6443, change with `--secure-port` flag.
 - default IP is first non-localhost network interface, change with `--bind-address` flag.
 - request handled by authentication and authorization modules.
 - request handled by admission control module(s).
 - authentication and authorisation modules run.
本地主机端口的话只能在集群内部进行访问，kube-proxy可以开启一个代理，但是不能保持长久连接，因此这里使用https访问，使用token认证。

2.API 访问控制


k8s中使用者有普通用户以及Service Account两种，需要进行认证，授权以及访问控制等步骤。

由Kubernetes管理的Service Accounts （服务账户）和（Users Accounts） 普通账户。普通账户是假定被外部或独立服务管理的，由管理员分配keys。在Kubernetes中不能通过API调用将普通用户添加到集群中。相比之下，Service Accounts是由Kubernetes API管理的帐户。它们被绑定到特定的命名空间，并由APIserver自动创建或通过API调用手动创建。Service Accounts与存储为Secrets的一组证书相关联，这些凭据被挂载到pod中，以便集群进程与Kubernetes API通信。

认证：

开启TLS时，所有的请求首先需要认证。Kubernetes支持多种认证机制，并支持同时开启多个认证插件（只要有一个认证通过即可）。如果认证成功，则用户的username会传入授权模块做进一步授权验证；对于认证失败的请求则返回HTTP 401。集群管理器将apiserver配置为运行一个或多个认证器模块。认证模块包含客户端证书，密码、Plain Tokens、Bootstrap Tokens、JWT Tokens（used for service accounts）。可以指定多个认证模块，每个认证模块都会按顺序进行，直到其中一个成功。

授权：

请求必须包含请求者的用户名，请求的操作以及受该操作影响的对象，如果策略已经声明用户具有完成请求的权限，则该请求将被授权。

Kubernetes支持多种授权模块，如ABAC模式、RBAC模式和Webhook模式。当创建集群时，会配置在API Server中使用的授权模块。如果配置了多个授权模块，Kubernetes会检查每个模块，当通过其中任何模块授权请求，则授权成功，如果所有模块都拒绝了该请求，则授权失败（HTTP 403）。

访问控制：

准入控制（Admission Control）用来对请求做进一步的验证或添加默认参数，除了授权模块可用的属性外，准入控制模块还可以访问正在创建或更新对象内容。也可以可以配置多个准入控制器，每个都会按顺序调用。

3. k8s  Bootstrap Tokens 认证
Kubernetes用户可以使用client certificates、bearer tokens、authenticating proxy、HTTP basic auth等认证插件来验证API请求

访问api可以使用普通用户来使用Static Token File认证，这样需要配置api server并且需要重启apiserver 且不支持动态改查。

https://blog.csdn.net/zhd930818/article/details/80026150

在此主要使用Bootstrap Tokens来进行认证：

功能目前为Alpha级别Kubernetes包括一个dynamically-managed的Bearer token类型，称为Bootstrap Token。这些token作为Secret存储在kube-system namespace中，可以动态管理和创建。Controller Manager包含一个TokenCleaner Controller，如果到期时可以删除Bootstrap Tokens Controller。

https://blog.csdn.net/u013073961/article/details/80045927

创建一个Service Account  命名空间 kube-system   

名字admin    角色  ClusterRole   角色绑定  ClusterRoleBinding

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
生成account:

kubectl create -f admin-token.yaml

查看kube-system令牌，具体令牌介绍http://docs.kubernetes.org.cn/713.html

kubectl get secret -n kube-system



第一个即为刚才创建的令牌：

kubectl describe secret admin-token-tddhx -ns=kube-system



注意token参数，以后api访问需要将token参数至于http  header参数中

4.Bootstrap Tokens访问API Server
curl -k https://192.168.1.15:6443/api/v1/services --header "Authorization: Bearer $token"

-k, --insecure   Allow connections to SSL sites without certs (H)



接下来研究一下：

授权  访问控制原理

api增删改查以及传参等问题

pod node等关键对象的信息获取

 

参考地址：

k8s英文api:

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/

k8s中文说明：

http://docs.kubernetes.org.cn/31.html#Alpha

k8s  java  client  Fabric8:

https://github.com/kubernetes-client/java

其他参考：

https://blog.csdn.net/u013073961/article/details/80045927
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84768856

# Kubernetes11--Java访问apiserver

想实现Java访问k8s  apiserver接口服务，目前主要Java client 有基于Jersey以及基于Fabric8

https://jersey.github.io/

https://github.com/fabric8io/kubernetes-client

Jersey需要自己实现各种接口调用，Fabric8已经实现主要的api调用，研究了一下Fabric8，决定还是自己模拟https请求来调用k8s服务接口，现成的框架尽管便于使用，但是有时候不便于扩展，还有k8s更新太快，现有框架对于功能可能不支持，因此这里使用org.apache.http包来模拟https请求，访问k8s服务。

k8s  apiserver服务端口   https://10.3.10.144:6443   内网IP  这里验证方式使用Bootstrap Token请求响应头方式，已经在上一篇给出Service Account的配置以及访问方式。

由于使用https通信，首先下载apiserver的证书到jdk中证书信息中。

1.apiserver证书下载
点击证书信息  复制到文件，下载到本地   命名为k8s.cer





2.将k8s证书导入到jdk中
cd $JAVA_HOME/jre/lib/security

keytool -import -file C:\Users\chen\Desktop\k8s.cer -keystore cacerts -alias k8s -trustcacerts

输入口令   changeit



查看口令：



具体密钥库的管理命令可以查看keytool指令

首先保证当前操作者用户对于jdk所在的文件有读写权限，win10权限验证很严格





尽量到jdk中secure目录下进行操作，keystore路径中不能有空格，否则：



3.使用httpclient访问
之前访问利用curl来完成：

curl -k https://192.168.1.15:6443/api/v1/services --header "Authorization: Bearer $token”

现在使用httpclient来模拟上述请求

构建sslclient,信任所有证书：

private static CloseableHttpClient createSSLClientDefault() {
        SSLContext sslContext;
        try {
            sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy() {
                //信任所有
                @Override
                public boolean isTrusted(X509Certificate[] xcs, String string){
                    return true;
                }
            }).build();
            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, SSLConnectionSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);            
            return HttpClients.custom().setSSLSocketFactory(sslsf).build();
        } catch (KeyStoreException ex) {
            Logger.getLogger(HttpUtils.class.getName()).log(Level.SEVERE, null, ex);
        } catch (NoSuchAlgorithmException ex) {
            Logger.getLogger(HttpUtils.class.getName()).log(Level.SEVERE, null, ex);
        } catch (KeyManagementException ex) {
            Logger.getLogger(HttpUtils.class.getName()).log(Level.SEVERE, null, ex);
        }
        return HttpClients.createDefault();
    }
构建http  get请求方法，来执行get请求：

public static String get(String url, Map<String, String> param,
            Map<String, String> headers) {
        HttpClient httpClient = getClient();
        // HttpClient httpClient = new DefaultHttpClient();
        HttpGet get = null;
        get = new HttpGet(url);
        appendHeaders(get, headers);
        appendParams(get, param);     
        try {
            HttpResponse httpResponse = httpClient.execute(get);
            if (httpResponse.getStatusLine().getStatusCode() == 200) {
                HttpEntity entity = httpResponse.getEntity();
                return EntityUtils.toString(entity);
            } else {
                httpResponse.getEntity().getContent().close();
                //System.out.println("return status code:" + httpResponse.getStatusLine().getStatusCode());
            }
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        long e = System.currentTimeMillis();
        return null;
    }

    private static HttpClient getClient() {
//        HttpClient hc = clients.get();
//        if (hc == null) {
//            hc = ThreadSafeHttpclientGetter.getNewInstance(1000, 5000, 40000);
//            clients.set(hc);
//        }
//        return hc;
    	return createSSLClientDefault();
    }
构建main方法进行初步测试，

public static void main( String[] args ){
    	String url = "https://10.3.10.144:6443/api/v1/services";
    	Map<String, String> param = new HashMap<>();
    	Map<String, String> headers = new HashMap<>();
    	headers.put("Authorization", ConfigUtils.getBootstrapTokenSecret());
        String result = HttpUtils.get(url, param, headers);
    	System.out.println(result);
    }
在请求头中添加Authorization中这一项，内容是Bootstrap_Token_Secret,之前生成的Service Account  Secret

执行结果：返回所有的Services的列表



参考地址：

请求头地址：http://tools.jb51.net/table/http_header

github:  https://github.com/chen1234567/k8s-java
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84788467

# Kubernetes12--Pod对象相关API

学习有关Pod对象的相关api调用以及返回json数据格式

api文档：https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/#

k8s的api更新很快，注意版本，以1.12为例进行详解：

K8S API概览
资源对象

name	decription
Workloads	Workloads are objects you use to manage and run your containers on the cluster.
Discovery & LB	Discovery & LB resources are objects you use to "stitch" your workloads together into an externally accessible, load-balanced Service.
Config & Storage	Config & Storage resources are objects you use to inject initialization data into your applications, and to persist data that is external to your container.
Cluster	Cluster resources objects define how the cluster itself is configured; these are typically used only by cluster operators.
Metadata	Metadata resources are objects you use to configure the behavior of other resources within the cluster, such as HorizontalPodAutoscaler for scaling workloads.
主要介绍了集群的主要对象以及作用，主要学习Pod, Node等关键对象的API

资源对象的主要组件：

name	decription
Resource ObjectMeta	Resource ObjectMeta: This is metadata about the resource, such as its name, type, api version, annotations, and labels. This contains fields that maybe updated both by the end user and the system (e.g. annotations)
ResourceSpec	
ResourceSpec: This is defined by the user and describes the desired state of system. Fill this in when creating or updating an object.

ResourceStatus	
ResourceStatus: This is filled in by the server and reports the current state of the system. In most cases, users don't need to change this.

重点是Spec部分，主要介绍了定制资源对象的主要API

资源对象操作：

name	describe
Create	Create operations will create the resource in the storage backend. After a resource is create the system will apply the desired state.
Update	Replace	Replacing a resource object will update the resource by replacing the existing spec with the provided one. For read-then-write operations this is safe because an optimistic lock failure will occur if the resource was modified between the read and write
Patch	Patch will apply a change to a specific field. How the change is merged is defined per field. Lists may either be replaced or merged. Merging lists will not preserve ordering.
Patches will never cause optimistic locking failures, and the last write will win
Read	Get	Get will retrieve a specific resource object by name.
List	List will retrieve all resource objects of a specific type within a namespace, and the results can be restricted to resources matching a selector query
Watch	Watch will stream results for an object(s) as it is updated. Similar to a callback, watch is used to respond to resource changes.
Delete	Rollback	Rollback a PodTemplate to a previous version
Read / Write Scale	Read or Update the number of replicas for the given resource
Read / Write Status	Read or Update the Status for a resource object.
Pod对象API
与Pod有关的对象类：

这一部分类之间关联关系复杂，以后慢慢研究

给Node节点创建label  kubectl label nodes <node-name> <label-key>=<label-value>
查看labels  kubectl get nodes --show-labels

API调用实例
如图所示，现在default namespace 启动了三个Pod  busybox   centos  tomcat



列出某个命名空间下所有Pod对象：

api/v1/namespaces/default/pods

{"kind":"PodList","apiVersion":"v1","metadata":{"selfLink":"/api/v1/namespaces/default/pods/","resourceVersion":"857561"},"items":[{"metadata":{"name":"busybox-68f48b9c57-887mw","generateName":"busybox-68f48b9c57-","namespace":"default","selfLink":"/api/v1/namespaces/default/pods/busybox-68f48b9c57-887mw","uid":"658c990f-f76e-11e8-953c-a0a33bc4c0eb","resourceVersion":"689769","creationTimestamp":"2018-12-04T02:43:35Z","labels":{"pod-template-hash":"68f48b9c57","run":"busybox"},"ownerReferences":[{"apiVersion":"apps/v1","kind":"ReplicaSet","name":"busybox-68f48b9c57","uid":"d3cfd753-f76d-11e8-953c-a0a33bc4c0eb","controller":true,"blockOwnerDeletion":true}]},"spec":{"volumes":[{"name":"default-token-fw46g","secret":{"secretName":"default-token-fw46g","defaultMode":420}}],"containers":[{"name":"busybox","image":"busybox","args":["/bin/sh"],"resources":{},"volumeMounts":[{"name":"default-token-fw46g","readOnly":true,"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount"}],"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"Always","stdin":true,"tty":true}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","serviceAccountName":"default","serviceAccount":"default","nodeName":"ac-private-2","securityContext":{},"schedulerName":"default-scheduler","tolerations":[{"key":"node.kubernetes.io/not-ready","operator":"Exists","effect":"NoExecute","tolerationSeconds":300},{"key":"node.kubernetes.io/unreachable","operator":"Exists","effect":"NoExecute","tolerationSeconds":300}],"priority":0},"status":{"phase":"Running","conditions":[{"type":"Initialized","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-04T02:43:35Z"},{"type":"Ready","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-04T02:47:14Z"},{"type":"ContainersReady","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-04T02:47:14Z"},{"type":"PodScheduled","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-04T02:43:35Z"}],"hostIP":"192.168.1.19","podIP":"10.244.1.85","startTime":"2018-12-04T02:43:35Z","containerStatuses":[{"name":"busybox","state":{"running":{"startedAt":"2018-12-04T02:47:13Z"}},"lastState":{},"ready":true,"restartCount":0,"image":"docker.io/busybox:latest","imageID":"docker-pullable://docker.io/busybox@sha256:2a03a6059f21e150ae84b0973863609494aad70f0a80eaeb64bddd8d92465812","containerID":"docker://710df306fe639930c9646c663714d50d1249b83452e94106fcb4fc6d83cdb50d"}],"qosClass":"BestEffort"}},{"metadata":{"name":"centos-ssh","namespace":"default","selfLink":"/api/v1/namespaces/default/pods/centos-ssh","uid":"79ebcd22-f84e-11e8-953c-a0a33bc4c0eb","resourceVersion":"843562","creationTimestamp":"2018-12-05T05:27:37Z","labels":{"name":"centos-ssh"}},"spec":{"volumes":[{"name":"data","hostPath":{"path":"/dev/sdk","type":""}},{"name":"default-token-fw46g","secret":{"secretName":"default-token-fw46g","defaultMode":420}}],"containers":[{"name":"centos-ssh","image":"docker.io/chenwenkai123456/centos-ssh:v1","command":["/usr/sbin/sshd","-D"],"ports":[{"hostPort":30015,"containerPort":22,"protocol":"TCP"}],"resources":{"limits":{"cpu":"4","memory":"8Gi"},"requests":{"cpu":"2","memory":"4Gi"}},"volumeMounts":[{"name":"data","mountPath":"/data"},{"name":"default-token-fw46g","readOnly":true,"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount"}],"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"IfNotPresent"}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","serviceAccountName":"default","serviceAccount":"default","nodeName":"xjs-dn04","securityContext":{},"affinity":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"type","operator":"In","values":["node4"]}]}]}}},"schedulerName":"default-scheduler","tolerations":[{"key":"node.kubernetes.io/not-ready","operator":"Exists","effect":"NoExecute","tolerationSeconds":300},{"key":"node.kubernetes.io/unreachable","operator":"Exists","effect":"NoExecute","tolerationSeconds":300}],"priority":0},"status":{"phase":"Running","conditions":[{"type":"Initialized","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:37Z"},{"type":"Ready","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:40Z"},{"type":"ContainersReady","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:40Z"},{"type":"PodScheduled","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:37Z"}],"hostIP":"192.168.1.16","podIP":"10.244.4.21","startTime":"2018-12-05T05:27:37Z","containerStatuses":[{"name":"centos-ssh","state":{"running":{"startedAt":"2018-12-05T05:27:39Z"}},"lastState":{},"ready":true,"restartCount":0,"image":"docker.io/chenwenkai123456/centos-ssh:v1","imageID":"docker-pullable://docker.io/chenwenkai123456/centos-ssh@sha256:ae6b8990ff39d54179443faace89fb2f26605bc76e56d435d2f804da099d7f83","containerID":"docker://6feaa58048bc482ce7e08b4344adae6acec2bd6bcacc563c6140ce26e295e17a"}],"qosClass":"Burstable"}},{"metadata":{"name":"tomcat-t6q74","generateName":"tomcat-","namespace":"default","selfLink":"/api/v1/namespaces/default/pods/tomcat-t6q74","uid":"aacb0cc9-f850-11e8-953c-a0a33bc4c0eb","resourceVersion":"845174","creationTimestamp":"2018-12-05T05:43:18Z","labels":{"app":"tomcat"},"ownerReferences":[{"apiVersion":"v1","kind":"ReplicationController","name":"tomcat","uid":"aac9e59d-f850-11e8-953c-a0a33bc4c0eb","controller":true,"blockOwnerDeletion":true}]},"spec":{"volumes":[{"name":"default-token-fw46g","secret":{"secretName":"default-token-fw46g","defaultMode":420}}],"containers":[{"name":"tomcat","image":"docker.io/tomcat","ports":[{"containerPort":8080,"protocol":"TCP"}],"resources":{},"volumeMounts":[{"name":"default-token-fw46g","readOnly":true,"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount"}],"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"Always"}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","serviceAccountName":"default","serviceAccount":"default","nodeName":"ac-private-2","securityContext":{},"affinity":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"type","operator":"In","values":["node1"]}]}]}}},"schedulerName":"default-scheduler","tolerations":[{"key":"node.kubernetes.io/not-ready","operator":"Exists","effect":"NoExecute","tolerationSeconds":300},{"key":"node.kubernetes.io/unreachable","operator":"Exists","effect":"NoExecute","tolerationSeconds":300}],"priority":0},"status":{"phase":"Running","conditions":[{"type":"Initialized","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:43:18Z"},{"type":"Ready","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:43:56Z"},{"type":"ContainersReady","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:43:56Z"},{"type":"PodScheduled","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:43:18Z"}],"hostIP":"192.168.1.19","podIP":"10.244.1.86","startTime":"2018-12-05T05:43:18Z","containerStatuses":[{"name":"tomcat","state":{"running":{"startedAt":"2018-12-05T05:43:56Z"}},"lastState":{},"ready":true,"restartCount":0,"image":"docker.io/tomcat:latest","imageID":"docker-pullable://docker.io/tomcat@sha256:e394c2f94eee344300e68d7183f3f19d272316f9691b280ac0e3911ea610e590","containerID":"docker://1b5cc80c30d58942d1e6431a156f3a58cd00293dcac0025ec6fc84f474d26050"}],"qosClass":"BestEffort"}}]}

获取某个Pod对象的信息：

api/v1/namespaces/default/pods/centos-ssh

{"kind":"Pod","apiVersion":"v1","metadata":{"name":"centos-ssh","namespace":"default","selfLink":"/api/v1/namespaces/default/pods/centos-ssh","uid":"79ebcd22-f84e-11e8-953c-a0a33bc4c0eb","resourceVersion":"843562","creationTimestamp":"2018-12-05T05:27:37Z","labels":{"name":"centos-ssh"}},"spec":{"volumes":[{"name":"data","hostPath":{"path":"/dev/sdk","type":""}},{"name":"default-token-fw46g","secret":{"secretName":"default-token-fw46g","defaultMode":420}}],"containers":[{"name":"centos-ssh","image":"docker.io/chenwenkai123456/centos-ssh:v1","command":["/usr/sbin/sshd","-D"],"ports":[{"hostPort":30015,"containerPort":22,"protocol":"TCP"}],"resources":{"limits":{"cpu":"4","memory":"8Gi"},"requests":{"cpu":"2","memory":"4Gi"}},"volumeMounts":[{"name":"data","mountPath":"/data"},{"name":"default-token-fw46g","readOnly":true,"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount"}],"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"IfNotPresent"}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","serviceAccountName":"default","serviceAccount":"default","nodeName":"xjs-dn04","securityContext":{},"affinity":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"type","operator":"In","values":["node4"]}]}]}}},"schedulerName":"default-scheduler","tolerations":[{"key":"node.kubernetes.io/not-ready","operator":"Exists","effect":"NoExecute","tolerationSeconds":300},{"key":"node.kubernetes.io/unreachable","operator":"Exists","effect":"NoExecute","tolerationSeconds":300}],"priority":0},"status":{"phase":"Running","conditions":[{"type":"Initialized","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:37Z"},{"type":"Ready","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:40Z"},{"type":"ContainersReady","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:40Z"},{"type":"PodScheduled","status":"True","lastProbeTime":null,"lastTransitionTime":"2018-12-05T05:27:37Z"}],"hostIP":"192.168.1.16","podIP":"10.244.4.21","startTime":"2018-12-05T05:27:37Z","containerStatuses":[{"name":"centos-ssh","state":{"running":{"startedAt":"2018-12-05T05:27:39Z"}},"lastState":{},"ready":true,"restartCount":0,"image":"docker.io/chenwenkai123456/centos-ssh:v1","imageID":"docker-pullable://docker.io/chenwenkai123456/centos-ssh@sha256:ae6b8990ff39d54179443faace89fb2f26605bc76e56d435d2f804da099d7f83","containerID":"docker://6feaa58048bc482ce7e08b4344adae6acec2bd6bcacc563c6140ce26e295e17a"}],"qosClass":"Burstable"}}
获取所有Node物理节点列表：

api/v1/nodes

{"kind":"NodeList","apiVersion":"v1","metadata":{"selfLink":"/api/v1/nodes","resourceVersion":"857998"},"items":[{"metadata":{"name":"ac-private-0","selfLink":"/api/v1/nodes/ac-private-0","uid":"7f4e158c-f399-11e8-953c-a0a33bc4c0eb","resourceVersion":"857991","creationTimestamp":"2018-11-29T05:42:02Z","labels":{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/hostname":"ac-private-0","type":"node2"},"annotations":{"flannel.alpha.coreos.com/backend-data":"{\"VtepMAC\":\"ea:fa:d4:6d:e1:e3\"}","flannel.alpha.coreos.com/backend-type":"vxlan","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.1.18","kubeadm.alpha.kubernetes.io/cri-socket":"/var/run/dockershim.sock","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"}},"spec":{"podCIDR":"10.244.2.0/24"},"status":{"capacity":{"cpu":"32","ephemeral-storage":"211338860Ki","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131455684Ki","pods":"110"},"allocatable":{"cpu":"32","ephemeral-storage":"194769893054","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131353284Ki","pods":"110"},"conditions":[{"type":"OutOfDisk","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletHasSufficientDisk","message":"kubelet has sufficient disk space available"},{"type":"MemoryPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletHasSufficientMemory","message":"kubelet has sufficient memory available"},{"type":"DiskPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletHasNoDiskPressure","message":"kubelet has no disk pressure"},{"type":"PIDPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T05:42:02Z","reason":"KubeletHasSufficientPID","message":"kubelet has sufficient PID available"},{"type":"Ready","status":"True","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletReady","message":"kubelet is posting ready status"}],"addresses":[{"type":"InternalIP","address":"192.168.1.18"},{"type":"Hostname","address":"ac-private-0"}],"daemonEndpoints":{"kubeletEndpoint":{"Port":10250}},"nodeInfo":{"machineID":"802b88242a474039b190efc9ce1e95c0","systemUUID":"F65FD9B1-2C7C-E711-8E6A-BC3F8F8906EF","bootID":"bd8e7ee1-4be5-4d22-8d99-9eeb2845ea3c","kernelVersion":"3.10.0-327.el7.x86_64","osImage":"CentOS Linux 7 (Core)","containerRuntimeVersion":"docker://1.13.1","kubeletVersion":"v1.12.3","kubeProxyVersion":"v1.12.3","operatingSystem":"linux","architecture":"amd64"},"images":[{"names":["docker.io/tomcat@sha256:e394c2f94eee344300e68d7183f3f19d272316f9691b280ac0e3911ea610e590","docker.io/tomcat:latest"],"sizeBytes":462725827},{"names":["docker.io/chenwenkai123456/centos-ssh@sha256:ae6b8990ff39d54179443faace89fb2f26605bc76e56d435d2f804da099d7f83","docker.io/chenwenkai123456/centos-ssh:v1"],"sizeBytes":305338283},{"names":["k8s.gcr.io/kube-apiserver:v1.12.0-rc.1"],"sizeBytes":193937288},{"names":["k8s.gcr.io/etcd:3.2.24"],"sizeBytes":193214599},{"names":["k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1"],"sizeBytes":164993346},{"names":["k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0"],"sizeBytes":122460923},{"names":["k8s.gcr.io/kube-proxy:v1.12.0-rc.1"],"sizeBytes":96583047},{"names":["k8s.gcr.io/kube-scheduler:v1.12.0-rc.1"],"sizeBytes":58295858},{"names":["quay.io/coreos/flannel@sha256:056cf57fd3bbe7264c0be1a3b34ec2e289b33e51c70f332f4e88aa83970ad891","quay.io/coreos/flannel:v0.9.1-amd64"],"sizeBytes":51338831},{"names":["docker.io/keveon/k8s-dns-kube-dns-amd64@sha256:6d8e0da4fb46e9ea2034a3f4cab0e095618a2ead78720c12e791342738e5f85d","docker.io/keveon/k8s-dns-kube-dns-amd64:1.14.8"],"sizeBytes":50456751},{"names":["docker.io/keveon/k8s-dns-sidecar-amd64@sha256:23df717980b4aa08d2da6c4cfa327f1b730d92ec9cf740959d2d5911830d82fb","docker.io/keveon/k8s-dns-sidecar-amd64:1.14.8"],"sizeBytes":42210862},{"names":["docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64@sha256:93c827f018cf3322f1ff2aa80324a0306048b0a69bc274e423071fb0d2d29d8b","docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64:1.14.8"],"sizeBytes":40951779},{"names":["k8s.gcr.io/coredns:1.2.2"],"sizeBytes":39220458},{"names":["docker.io/busybox@sha256:2a03a6059f21e150ae84b0973863609494aad70f0a80eaeb64bddd8d92465812","docker.io/busybox:latest"],"sizeBytes":1154353},{"names":["k8s.gcr.io/pause:3.1"],"sizeBytes":742472}]}},{"metadata":{"name":"ac-private-1","selfLink":"/api/v1/nodes/ac-private-1","uid":"af3aa33b-f385-11e8-953c-a0a33bc4c0eb","resourceVersion":"857988","creationTimestamp":"2018-11-29T03:20:13Z","labels":{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/hostname":"ac-private-1","node-role.kubernetes.io/master":"","type":"master"},"annotations":{"flannel.alpha.coreos.com/backend-data":"{\"VtepMAC\":\"22:67:2b:9d:c5:46\"}","flannel.alpha.coreos.com/backend-type":"vxlan","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.1.15","kubeadm.alpha.kubernetes.io/cri-socket":"/var/run/dockershim.sock","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"}},"spec":{"podCIDR":"10.244.0.0/24","taints":[{"key":"node-role.kubernetes.io/master","effect":"NoSchedule"}]},"status":{"capacity":{"cpu":"32","ephemeral-storage":"211338860Ki","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131455684Ki","pods":"110"},"allocatable":{"cpu":"32","ephemeral-storage":"194769893054","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131353284Ki","pods":"110"},"conditions":[{"type":"OutOfDisk","status":"False","lastHeartbeatTime":"2018-12-05T07:57:15Z","lastTransitionTime":"2018-11-29T03:20:03Z","reason":"KubeletHasSufficientDisk","message":"kubelet has sufficient disk space available"},{"type":"MemoryPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:15Z","lastTransitionTime":"2018-11-29T03:20:03Z","reason":"KubeletHasSufficientMemory","message":"kubelet has sufficient memory available"},{"type":"DiskPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:15Z","lastTransitionTime":"2018-11-29T03:20:03Z","reason":"KubeletHasNoDiskPressure","message":"kubelet has no disk pressure"},{"type":"PIDPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:15Z","lastTransitionTime":"2018-11-29T03:20:03Z","reason":"KubeletHasSufficientPID","message":"kubelet has sufficient PID available"},{"type":"Ready","status":"True","lastHeartbeatTime":"2018-12-05T07:57:15Z","lastTransitionTime":"2018-11-29T03:32:23Z","reason":"KubeletReady","message":"kubelet is posting ready status"}],"addresses":[{"type":"InternalIP","address":"192.168.1.15"},{"type":"Hostname","address":"ac-private-1"}],"daemonEndpoints":{"kubeletEndpoint":{"Port":10250}},"nodeInfo":{"machineID":"b2df1d64f0744a5e8296ca5dac633f5d","systemUUID":"68BCF705-2D7C-E711-8CB0-BC3F8F8905AE","bootID":"7fe0bed5-dc91-4a68-bc6f-225b5970cdae","kernelVersion":"3.10.0-327.el7.x86_64","osImage":"CentOS Linux 7 (Core)","containerRuntimeVersion":"docker://1.13.1","kubeletVersion":"v1.12.2","kubeProxyVersion":"v1.12.2","operatingSystem":"linux","architecture":"amd64"},"images":[{"names":["k8s.gcr.io/kube-apiserver:v1.12.0-rc.1"],"sizeBytes":193937288},{"names":["k8s.gcr.io/etcd:3.2.24"],"sizeBytes":193214599},{"names":["k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1"],"sizeBytes":164993346},{"names":["k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0"],"sizeBytes":122460923},{"names":["k8s.gcr.io/kube-proxy:v1.12.0-rc.1"],"sizeBytes":96583047},{"names":["k8s.gcr.io/kube-scheduler:v1.12.0-rc.1"],"sizeBytes":58295858},{"names":["quay.io/coreos/flannel@sha256:056cf57fd3bbe7264c0be1a3b34ec2e289b33e51c70f332f4e88aa83970ad891","quay.io/coreos/flannel:v0.9.1-amd64"],"sizeBytes":51338831},{"names":["k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8"],"sizeBytes":50456751},{"names":["k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8"],"sizeBytes":42210862},{"names":["k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8"],"sizeBytes":40951779},{"names":["k8s.gcr.io/coredns:1.2.2"],"sizeBytes":39220458},{"names":["k8s.gcr.io/pause:3.1"],"sizeBytes":742472}]}},{"metadata":{"name":"ac-private-2","selfLink":"/api/v1/nodes/ac-private-2","uid":"63a10798-f396-11e8-953c-a0a33bc4c0eb","resourceVersion":"857983","creationTimestamp":"2018-11-29T05:19:47Z","labels":{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/hostname":"ac-private-2","type":"node1"},"annotations":{"flannel.alpha.coreos.com/backend-data":"{\"VtepMAC\":\"36:00:06:6e:25:32\"}","flannel.alpha.coreos.com/backend-type":"vxlan","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.1.19","kubeadm.alpha.kubernetes.io/cri-socket":"/var/run/dockershim.sock","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"}},"spec":{"podCIDR":"10.244.1.0/24"},"status":{"capacity":{"cpu":"32","ephemeral-storage":"211338860Ki","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131455684Ki","pods":"110"},"allocatable":{"cpu":"32","ephemeral-storage":"194769893054","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131353284Ki","pods":"110"},"conditions":[{"type":"OutOfDisk","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T05:19:47Z","reason":"KubeletHasSufficientDisk","message":"kubelet has sufficient disk space available"},{"type":"MemoryPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T05:19:47Z","reason":"KubeletHasSufficientMemory","message":"kubelet has sufficient memory available"},{"type":"DiskPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T05:19:47Z","reason":"KubeletHasNoDiskPressure","message":"kubelet has no disk pressure"},{"type":"PIDPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T05:19:47Z","reason":"KubeletHasSufficientPID","message":"kubelet has sufficient PID available"},{"type":"Ready","status":"True","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T05:19:47Z","reason":"KubeletReady","message":"kubelet is posting ready status"}],"addresses":[{"type":"InternalIP","address":"192.168.1.19"},{"type":"Hostname","address":"ac-private-2"}],"daemonEndpoints":{"kubeletEndpoint":{"Port":10250}},"nodeInfo":{"machineID":"68c868b5739149b0bd014cd97cf7342f","systemUUID":"EEAFC7F8-2C7C-E711-B736-BC3F8F8906B3","bootID":"1bf420f2-b895-4130-9f9c-4c6577bc63f5","kernelVersion":"3.10.0-327.el7.x86_64","osImage":"CentOS Linux 7 (Core)","containerRuntimeVersion":"docker://1.13.1","kubeletVersion":"v1.12.2","kubeProxyVersion":"v1.12.2","operatingSystem":"linux","architecture":"amd64"},"images":[{"names":["docker.io/zerosre/kubernetes-heapster-canary@sha256:5eab4ad3d6fb0f4e97fbc3525b5feab3f93f900f63574e4e76a51105d186c4ef","docker.io/zerosre/kubernetes-heapster-canary:latest"],"sizeBytes":971277014},{"names":["docker.io/tomcat@sha256:e394c2f94eee344300e68d7183f3f19d272316f9691b280ac0e3911ea610e590","docker.io/tomcat:latest"],"sizeBytes":462725827},{"names":["docker.io/zerosre/gcr.io-google_containers-heapster_grafana-v3.1.1@sha256:94dfb1fc71792cf3dd8277ac60f306f4b54ff2c6c3996f1fd5bd93439671db5b","docker.io/zerosre/gcr.io-google_containers-heapster_grafana-v3.1.1:latest"],"sizeBytes":279315095},{"names":["docker.io/kubernetes/heapster@sha256:db8dc92b04f8818a30a4655bfc42e3f4eaca5157bcbfa3d0b834d196762d1926","docker.io/kubernetes/heapster:v0.10.0"],"sizeBytes":211314293},{"names":["k8s.gcr.io/kube-apiserver:v1.12.0-rc.1"],"sizeBytes":193937288},{"names":["k8s.gcr.io/etcd:3.2.24"],"sizeBytes":193214599},{"names":["k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1"],"sizeBytes":164993346},{"names":["k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0"],"sizeBytes":122460923},{"names":["k8s.gcr.io/kube-proxy:v1.12.0-rc.1"],"sizeBytes":96583047},{"names":["k8s.gcr.io/kube-scheduler:v1.12.0-rc.1"],"sizeBytes":58295858},{"names":["quay.io/coreos/flannel@sha256:056cf57fd3bbe7264c0be1a3b34ec2e289b33e51c70f332f4e88aa83970ad891","quay.io/coreos/flannel:v0.9.1-amd64"],"sizeBytes":51338831},{"names":["k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.8"],"sizeBytes":50456751},{"names":["k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.8"],"sizeBytes":42210862},{"names":["k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.8"],"sizeBytes":40951779},{"names":["k8s.gcr.io/coredns:1.2.2"],"sizeBytes":39220458},{"names":["k8s.gcr.io/heapster-influxdb-amd64:v1.3.3"],"sizeBytes":11585302},{"names":["docker.io/busybox@sha256:2a03a6059f21e150ae84b0973863609494aad70f0a80eaeb64bddd8d92465812","docker.io/busybox:latest"],"sizeBytes":1154353},{"names":["k8s.gcr.io/pause:3.1"],"sizeBytes":742472}]}},{"metadata":{"name":"xjs-dn02","selfLink":"/api/v1/nodes/xjs-dn02","uid":"0643ac8c-f3a0-11e8-953c-a0a33bc4c0eb","resourceVersion":"857984","creationTimestamp":"2018-11-29T06:28:46Z","labels":{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/hostname":"xjs-dn02","type":"node3"},"annotations":{"flannel.alpha.coreos.com/backend-data":"{\"VtepMAC\":\"c2:88:2e:59:da:47\"}","flannel.alpha.coreos.com/backend-type":"vxlan","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.1.14","kubeadm.alpha.kubernetes.io/cri-socket":"/var/run/dockershim.sock","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"}},"spec":{"podCIDR":"10.244.3.0/24"},"status":{"capacity":{"cpu":"32","ephemeral-storage":"211338860Ki","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131455684Ki","pods":"110"},"allocatable":{"cpu":"32","ephemeral-storage":"194769893054","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131353284Ki","pods":"110"},"conditions":[{"type":"OutOfDisk","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T06:28:46Z","reason":"KubeletHasSufficientDisk","message":"kubelet has sufficient disk space available"},{"type":"MemoryPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T06:28:46Z","reason":"KubeletHasSufficientMemory","message":"kubelet has sufficient memory available"},{"type":"DiskPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T06:28:46Z","reason":"KubeletHasNoDiskPressure","message":"kubelet has no disk pressure"},{"type":"PIDPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T06:28:46Z","reason":"KubeletHasSufficientPID","message":"kubelet has sufficient PID available"},{"type":"Ready","status":"True","lastHeartbeatTime":"2018-12-05T07:57:14Z","lastTransitionTime":"2018-11-29T06:28:46Z","reason":"KubeletReady","message":"kubelet is posting ready status"}],"addresses":[{"type":"InternalIP","address":"192.168.1.14"},{"type":"Hostname","address":"xjs-dn02"}],"daemonEndpoints":{"kubeletEndpoint":{"Port":10250}},"nodeInfo":{"machineID":"a5a773f762db43c4ab3296e99cdcaa1d","systemUUID":"74D10CDA-2C7C-E711-903E-BC3F8F8905AB","bootID":"78410d78-b362-4f5b-a77c-2e3799dc6fbd","kernelVersion":"3.10.0-327.el7.x86_64","osImage":"CentOS Linux 7 (Core)","containerRuntimeVersion":"docker://1.13.1","kubeletVersion":"v1.12.3","kubeProxyVersion":"v1.12.3","operatingSystem":"linux","architecture":"amd64"},"images":[{"names":["docker.io/chenwenkai123456/centos-ssh@sha256:ae6b8990ff39d54179443faace89fb2f26605bc76e56d435d2f804da099d7f83","docker.io/chenwenkai123456/centos-ssh:v1"],"sizeBytes":305338283},{"names":["k8s.gcr.io/kube-apiserver:v1.12.0-rc.1"],"sizeBytes":193937288},{"names":["k8s.gcr.io/etcd:3.2.24"],"sizeBytes":193214599},{"names":["k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1"],"sizeBytes":164993346},{"names":["k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0"],"sizeBytes":122460923},{"names":["k8s.gcr.io/kube-proxy:v1.12.0-rc.1"],"sizeBytes":96583047},{"names":["k8s.gcr.io/kube-scheduler:v1.12.0-rc.1"],"sizeBytes":58295858},{"names":["quay.io/coreos/flannel@sha256:056cf57fd3bbe7264c0be1a3b34ec2e289b33e51c70f332f4e88aa83970ad891","quay.io/coreos/flannel:v0.9.1-amd64"],"sizeBytes":51338831},{"names":["docker.io/keveon/k8s-dns-kube-dns-amd64@sha256:6d8e0da4fb46e9ea2034a3f4cab0e095618a2ead78720c12e791342738e5f85d","docker.io/keveon/k8s-dns-kube-dns-amd64:1.14.8"],"sizeBytes":50456751},{"names":["docker.io/keveon/k8s-dns-sidecar-amd64@sha256:23df717980b4aa08d2da6c4cfa327f1b730d92ec9cf740959d2d5911830d82fb","docker.io/keveon/k8s-dns-sidecar-amd64:1.14.8"],"sizeBytes":42210862},{"names":["docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64@sha256:93c827f018cf3322f1ff2aa80324a0306048b0a69bc274e423071fb0d2d29d8b","docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64:1.14.8"],"sizeBytes":40951779},{"names":["k8s.gcr.io/coredns:1.2.2"],"sizeBytes":39220458},{"names":["k8s.gcr.io/pause:3.1"],"sizeBytes":742472}]}},{"metadata":{"name":"xjs-dn04","selfLink":"/api/v1/nodes/xjs-dn04","uid":"6b3bef8a-f3a0-11e8-953c-a0a33bc4c0eb","resourceVersion":"857992","creationTimestamp":"2018-11-29T06:31:35Z","labels":{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/hostname":"xjs-dn04","type":"node4"},"annotations":{"flannel.alpha.coreos.com/backend-data":"{\"VtepMAC\":\"3a:8d:17:87:97:12\"}","flannel.alpha.coreos.com/backend-type":"vxlan","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.1.16","kubeadm.alpha.kubernetes.io/cri-socket":"/var/run/dockershim.sock","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"}},"spec":{"podCIDR":"10.244.4.0/24"},"status":{"capacity":{"cpu":"32","ephemeral-storage":"211338860Ki","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131455684Ki","pods":"110"},"allocatable":{"cpu":"32","ephemeral-storage":"194769893054","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131353284Ki","pods":"110"},"conditions":[{"type":"OutOfDisk","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T06:31:35Z","reason":"KubeletHasSufficientDisk","message":"kubelet has sufficient disk space available"},{"type":"MemoryPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T06:31:35Z","reason":"KubeletHasSufficientMemory","message":"kubelet has sufficient memory available"},{"type":"DiskPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T06:31:35Z","reason":"KubeletHasNoDiskPressure","message":"kubelet has no disk pressure"},{"type":"PIDPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T06:31:35Z","reason":"KubeletHasSufficientPID","message":"kubelet has sufficient PID available"},{"type":"Ready","status":"True","lastHeartbeatTime":"2018-12-05T07:57:19Z","lastTransitionTime":"2018-11-29T06:31:35Z","reason":"KubeletReady","message":"kubelet is posting ready status"}],"addresses":[{"type":"InternalIP","address":"192.168.1.16"},{"type":"Hostname","address":"xjs-dn04"}],"daemonEndpoints":{"kubeletEndpoint":{"Port":10250}},"nodeInfo":{"machineID":"b993a6bb441c48d395be6d805102b997","systemUUID":"7CD61657-2A7C-E711-9805-BC3F8F8905B1","bootID":"0c53ce4d-eda0-43e6-9e4c-0cd81a52d866","kernelVersion":"3.10.0-327.el7.x86_64","osImage":"CentOS Linux 7 (Core)","containerRuntimeVersion":"docker://1.13.1","kubeletVersion":"v1.12.3","kubeProxyVersion":"v1.12.3","operatingSystem":"linux","architecture":"amd64"},"images":[{"names":["docker.io/kinogmt/centos-ssh@sha256:acdf7798259f859a8e713f2f88511eef5e0f94cb2d2dec078a1fe169a8aacf63","docker.io/kinogmt/centos-ssh:latest"],"sizeBytes":773130464},{"names":["docker.io/chenwenkai123456/centos-ssh@sha256:ae6b8990ff39d54179443faace89fb2f26605bc76e56d435d2f804da099d7f83","docker.io/chenwenkai123456/centos-ssh:v1"],"sizeBytes":305338283},{"names":["docker.io/zerosre/gcr.io-google_containers-heapster_grafana-v3.1.1@sha256:94dfb1fc71792cf3dd8277ac60f306f4b54ff2c6c3996f1fd5bd93439671db5b","docker.io/zerosre/gcr.io-google_containers-heapster_grafana-v3.1.1:latest"],"sizeBytes":279315095},{"names":["docker.io/centos@sha256:67dad89757a55bfdfabec8abd0e22f8c7c12a1856514726470228063ed86593b","docker.io/centos:latest"],"sizeBytes":200375261},{"names":["k8s.gcr.io/kube-apiserver:v1.12.0-rc.1"],"sizeBytes":193937288},{"names":["k8s.gcr.io/etcd:3.2.24"],"sizeBytes":193214599},{"names":["k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1"],"sizeBytes":164993346},{"names":["k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0"],"sizeBytes":122460923},{"names":["k8s.gcr.io/kube-proxy:v1.12.0-rc.1"],"sizeBytes":96583047},{"names":["k8s.gcr.io/kube-scheduler:v1.12.0-rc.1"],"sizeBytes":58295858},{"names":["quay.io/coreos/flannel@sha256:056cf57fd3bbe7264c0be1a3b34ec2e289b33e51c70f332f4e88aa83970ad891","quay.io/coreos/flannel:v0.9.1-amd64"],"sizeBytes":51338831},{"names":["docker.io/keveon/k8s-dns-kube-dns-amd64@sha256:6d8e0da4fb46e9ea2034a3f4cab0e095618a2ead78720c12e791342738e5f85d","docker.io/keveon/k8s-dns-kube-dns-amd64:1.14.8"],"sizeBytes":50456751},{"names":["docker.io/keveon/k8s-dns-sidecar-amd64@sha256:23df717980b4aa08d2da6c4cfa327f1b730d92ec9cf740959d2d5911830d82fb","docker.io/keveon/k8s-dns-sidecar-amd64:1.14.8"],"sizeBytes":42210862},{"names":["docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64@sha256:93c827f018cf3322f1ff2aa80324a0306048b0a69bc274e423071fb0d2d29d8b","docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64:1.14.8"],"sizeBytes":40951779},{"names":["k8s.gcr.io/coredns:1.2.2"],"sizeBytes":39220458},{"names":["k8s.gcr.io/pause:3.1"],"sizeBytes":742472}]}}]}
获取某个Node节点物理列表：

api/v1/nodes/ac-private-0

{"kind":"Node","apiVersion":"v1","metadata":{"name":"ac-private-0","selfLink":"/api/v1/nodes/ac-private-0","uid":"7f4e158c-f399-11e8-953c-a0a33bc4c0eb","resourceVersion":"858215","creationTimestamp":"2018-11-29T05:42:02Z","labels":{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/hostname":"ac-private-0","type":"node2"},"annotations":{"flannel.alpha.coreos.com/backend-data":"{\"VtepMAC\":\"ea:fa:d4:6d:e1:e3\"}","flannel.alpha.coreos.com/backend-type":"vxlan","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.1.18","kubeadm.alpha.kubernetes.io/cri-socket":"/var/run/dockershim.sock","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"}},"spec":{"podCIDR":"10.244.2.0/24"},"status":{"capacity":{"cpu":"32","ephemeral-storage":"211338860Ki","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131455684Ki","pods":"110"},"allocatable":{"cpu":"32","ephemeral-storage":"194769893054","hugepages-1Gi":"0","hugepages-2Mi":"0","memory":"131353284Ki","pods":"110"},"conditions":[{"type":"OutOfDisk","status":"False","lastHeartbeatTime":"2018-12-05T07:59:39Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletHasSufficientDisk","message":"kubelet has sufficient disk space available"},{"type":"MemoryPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:59:39Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletHasSufficientMemory","message":"kubelet has sufficient memory available"},{"type":"DiskPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:59:39Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletHasNoDiskPressure","message":"kubelet has no disk pressure"},{"type":"PIDPressure","status":"False","lastHeartbeatTime":"2018-12-05T07:59:39Z","lastTransitionTime":"2018-11-29T05:42:02Z","reason":"KubeletHasSufficientPID","message":"kubelet has sufficient PID available"},{"type":"Ready","status":"True","lastHeartbeatTime":"2018-12-05T07:59:39Z","lastTransitionTime":"2018-11-29T05:55:48Z","reason":"KubeletReady","message":"kubelet is posting ready status"}],"addresses":[{"type":"InternalIP","address":"192.168.1.18"},{"type":"Hostname","address":"ac-private-0"}],"daemonEndpoints":{"kubeletEndpoint":{"Port":10250}},"nodeInfo":{"machineID":"802b88242a474039b190efc9ce1e95c0","systemUUID":"F65FD9B1-2C7C-E711-8E6A-BC3F8F8906EF","bootID":"bd8e7ee1-4be5-4d22-8d99-9eeb2845ea3c","kernelVersion":"3.10.0-327.el7.x86_64","osImage":"CentOS Linux 7 (Core)","containerRuntimeVersion":"docker://1.13.1","kubeletVersion":"v1.12.3","kubeProxyVersion":"v1.12.3","operatingSystem":"linux","architecture":"amd64"},"images":[{"names":["docker.io/tomcat@sha256:e394c2f94eee344300e68d7183f3f19d272316f9691b280ac0e3911ea610e590","docker.io/tomcat:latest"],"sizeBytes":462725827},{"names":["docker.io/chenwenkai123456/centos-ssh@sha256:ae6b8990ff39d54179443faace89fb2f26605bc76e56d435d2f804da099d7f83","docker.io/chenwenkai123456/centos-ssh:v1"],"sizeBytes":305338283},{"names":["k8s.gcr.io/kube-apiserver:v1.12.0-rc.1"],"sizeBytes":193937288},{"names":["k8s.gcr.io/etcd:3.2.24"],"sizeBytes":193214599},{"names":["k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1"],"sizeBytes":164993346},{"names":["k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0"],"sizeBytes":122460923},{"names":["k8s.gcr.io/kube-proxy:v1.12.0-rc.1"],"sizeBytes":96583047},{"names":["k8s.gcr.io/kube-scheduler:v1.12.0-rc.1"],"sizeBytes":58295858},{"names":["quay.io/coreos/flannel@sha256:056cf57fd3bbe7264c0be1a3b34ec2e289b33e51c70f332f4e88aa83970ad891","quay.io/coreos/flannel:v0.9.1-amd64"],"sizeBytes":51338831},{"names":["docker.io/keveon/k8s-dns-kube-dns-amd64@sha256:6d8e0da4fb46e9ea2034a3f4cab0e095618a2ead78720c12e791342738e5f85d","docker.io/keveon/k8s-dns-kube-dns-amd64:1.14.8"],"sizeBytes":50456751},{"names":["docker.io/keveon/k8s-dns-sidecar-amd64@sha256:23df717980b4aa08d2da6c4cfa327f1b730d92ec9cf740959d2d5911830d82fb","docker.io/keveon/k8s-dns-sidecar-amd64:1.14.8"],"sizeBytes":42210862},{"names":["docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64@sha256:93c827f018cf3322f1ff2aa80324a0306048b0a69bc274e423071fb0d2d29d8b","docker.io/keveon/k8s-dns-dnsmasq-nanny-amd64:1.14.8"],"sizeBytes":40951779},{"names":["k8s.gcr.io/coredns:1.2.2"],"sizeBytes":39220458},{"names":["docker.io/busybox@sha256:2a03a6059f21e150ae84b0973863609494aad70f0a80eaeb64bddd8d92465812","docker.io/busybox:latest"],"sizeBytes":1154353},{"names":["k8s.gcr.io/pause:3.1"],"sizeBytes":742472}]}}
可以获取到该物理节点的内存以及cpu资源信息。
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84823560

# Kubernetes13--Metrics API

参考地址：https://github.com/kubernetes-incubator/metrics-server/

metrics-server： 它是一种API Server，提供了核心的Metrics API，就像k8s组件kube-apiserver提供了很多API群组一样，但它不是k8s组成部分，而是托管运行在k8s之上的Pod。为了让用户无缝的使用metrics-server当中的API，还需要把这类自定义的API，通过聚合器聚合到核心API组里，然后可以把此API当作是核心API的一部分，通过kubectl api-versions可直接查看。metrics-server收集指标数据的方式是从各节点上kubelet提供的Summary API 即10250端口收集数据，收集Node和Pod核心资源指标数据，主要是内存和cpu方面的使用情况，并将收集的信息存储在内存中。

原理图如下：



Metrics Server安装
准备metrics-server部署文件：

https://github.com/kubernetes-incubator/metrics-server/tree/master/deploy/1.8%2B

其中metrics-server-deployment.yaml需要使用k8s.gcr.io/metrics-server-amd64:v0.3.1镜像，该镜像无法直接下载，到docker hub中寻找替代镜像rancher/metrics-server-amd64

下载rancher/metrics-server-amd64基础镜像



修改metrics-server-deployment.yaml镜像名称：docker.io/rancher/metrics-server-amd64:v0.3.1

部署文件详解如下：

声明一个ClusterRole:

[root@Ac-private-1 metrics]# cat aggregated-metrics-reader.yaml 
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:aggregated-metrics-reader
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
  声明RoleBinding:

root@Ac-private-1 metrics]# cat auth-reader.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
  进行角色绑定：

[root@Ac-private-1 metrics]# cat auth-delegator.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
  声明读取资源的角色：ClusterRole  ClusterRoleBinding

root@Ac-private-1 metrics]# cat resource-reader.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "extensions"
  resources:
  - deployments
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
  声明APIService

[root@Ac-private-1 metrics]# cat metrics-apiservice.yaml 
---
apiVersion: apiregistration.k8s.io/v1beta1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
进行Deployment 注意修改image以及添加启动命令，禁止https验证

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: docker.io/rancher/metrics-server-amd64:v0.3.1
        imagePullPolicy: Always
        command:
        - /metrics-server
        - --metric-resolution=30s
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
声明Service

[root@Ac-private-1 metrics]# cat metrics-server-service.yaml 
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    kubernetes.io/name: "Metrics-server"
spec:
  selector:
    k8s-app: metrics-server
  ports:
  - port: 443
    protocol: TCP
    targetPort: 443
    执行部署文件：



访问路径：

获取所有Node资源状态

curl -k https://192.168.1.15:6443/apis/metrics.k8s.io/v1beta1/nodes

{"kind":"NodeMetricsList","apiVersion":"metrics.k8s.io/v1beta1","metadata":{"selfLink":"/apis/metrics.k8s.io/v1beta1/nodes"},"items":[{"metadata":{"name":"ac-private-1","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/ac-private-1","creationTimestamp":"2018-12-06T02:09:50Z"},"timestamp":"2018-12-06T02:09:15Z","window":"30s","usage":{"cpu":"139173580n","memory":"14264488Ki"}},{"metadata":{"name":"ac-private-2","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/ac-private-2","creationTimestamp":"2018-12-06T02:09:50Z"},"timestamp":"2018-12-06T02:09:15Z","window":"30s","usage":{"cpu":"21085203n","memory":"3846036Ki"}},{"metadata":{"name":"ac-private-0","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/ac-private-0","creationTimestamp":"2018-12-06T02:09:50Z"},"timestamp":"2018-12-06T02:09:18Z","window":"30s","usage":{"cpu":"73255790n","memory":"12064132Ki"}},{"metadata":{"name":"xjs-dn02","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/xjs-dn02","creationTimestamp":"2018-12-06T02:09:50Z"},"timestamp":"2018-12-06T02:09:18Z","window":"30s","usage":{"cpu":"93832180n","memory":"10894344Ki"}},{"metadata":{"name":"xjs-dn04","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/xjs-dn04","creationTimestamp":"2018-12-06T02:09:50Z"},"timestamp":"2018-12-06T02:09:20Z","window":"30s","usage":{"cpu":"329854983n","memory":"10906516Ki"}}]}
获取所有pods资源状态：

https://192.168.1.15:6443/apis/metrics.k8s.io/v1beta1/namespaces/default/pods

{"kind":"PodMetricsList","apiVersion":"metrics.k8s.io/v1beta1","metadata":{"selfLink":"/apis/metrics.k8s.io/v1beta1/namespaces/default/pods"},"items":[{"metadata":{"name":"tomcat-t6q74","namespace":"default","selfLink":"/apis/metrics.k8s.io/v1beta1/namespaces/default/pods/tomcat-t6q74","creationTimestamp":"2018-12-06T02:21:56Z"},"timestamp":"2018-12-06T02:21:43Z","window":"30s","containers":[{"name":"tomcat","usage":{"cpu":"259106n","memory":"650528Ki"}}]},{"metadata":{"name":"centos-ssh","namespace":"default","selfLink":"/apis/metrics.k8s.io/v1beta1/namespaces/default/pods/centos-ssh","creationTimestamp":"2018-12-06T02:21:56Z"},"timestamp":"2018-12-06T02:21:23Z","window":"30s","containers":[{"name":"centos-ssh","usage":{"cpu":"0","memory":"1032Ki"}}]},{"metadata":{"name":"busybox-68f48b9c57-887mw","namespace":"default","selfLink":"/apis/metrics.k8s.io/v1beta1/namespaces/default/pods/busybox-68f48b9c57-887mw","creationTimestamp":"2018-12-06T02:21:56Z"},"timestamp":"2018-12-06T02:21:42Z","window":"30s","containers":[{"name":"busybox","usage":{"cpu":"0","memory":"60Ki"}}]}]}
获取某个Node节点资源：

https://192.168.1.15:6443/apis/metrics.k8s.io/v1beta1/nodes/ac-private-2

{"kind":"NodeMetrics","apiVersion":"metrics.k8s.io/v1beta1","metadata":{"name":"ac-private-2","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/ac-private-2","creationTimestamp":"2018-12-06T02:26:04Z"},"timestamp":"2018-12-06T02:25:46Z","window":"30s","usage":{"cpu":"22436949n","memory":"3846476Ki"}}
获取某个Pod资源：

https://192.168.1.15:6443/apis/metrics.k8s.io/v1beta1/namespaces/default/pods/tomcat-t6q74

{"kind":"PodMetrics","apiVersion":"metrics.k8s.io/v1beta1","metadata":{"name":"tomcat-t6q74","namespace":"default","selfLink":"/apis/metrics.k8s.io/v1beta1/namespaces/default/pods/tomcat-t6q74","creationTimestamp":"2018-12-06T02:24:30Z"},"timestamp":"2018-12-06T02:24:15Z","window":"30s","containers":[{"name":"tomcat","usage":{"cpu":"222623n","memory":"650528Ki"}}]}
metric可以获取Node以及Pod的资源情况 这里只包含cpu以及内存使用。
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84839055

# Kubernetes14--Kubeadm详解

之前用Kubeadm安装k8s集群遇到了好多问题，后来想扩展CoreDns，Metrics-server等功能发现对于内部机理不太清楚，因此把kubeadm的内部机理学习一下。

kubeadm的常见命令：



主要学习一下init初始化过程：

使用kubeadm来部署集群的一般步骤：

kubeadm init
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f <network-of-choice.yaml>
kubeadm join --token <token> <master-ip>:<master-port>
默认的配置文件
/etc/kubernetes/为k8s集群的应用路径

/etc/kubernetes/manifests存放静态Pod组件的yaml模板文件

etcd  apiserver  controller-manager  scheduler



etcd.yaml文件存放etcd数据库的配置yaml：

[root@Ac-private-1 manifests]# cat etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://127.0.0.1:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://127.0.0.1:2380
    - --initial-cluster=ac-private-1=https://127.0.0.1:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379
    - --listen-peer-urls=https://127.0.0.1:2380
    - --name=ac-private-1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.2.24
    imagePullPolicy: IfNotPresent
    livenessProbe:
      exec:
        command:
        - /bin/sh
        - -ec
        - ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt
          --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
          get foo
          failureThreshold: 8
          initialDelaySeconds: 15
          timeoutSeconds: 15
        name: etcd
        resources: {}
        volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
    hostNetwork: true
    priorityClassName: system-cluster-critical
    volumes:
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
      status: {}
      apiserver的yaml模板文件：

[root@Ac-private-1 manifests]# cat kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
    - --advertise-address=192.168.1.15
    - --allow-privileged=true
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.12.0-rc.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.1.15
        path: /healthz
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-apiserver
    resources:
      requests:
        cpu: 250m
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    hostNetwork: true
    priorityClassName: system-cluster-critical
    volumes:
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
      status: {}
      controller-manager yaml模板配置文件

[root@Ac-private-1 manifests]# cat kube-controller-manager.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --address=127.0.0.1
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --node-cidr-mask-size=24
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --use-service-account-credentials=true
    image: k8s.gcr.io/kube-controller-manager:v1.12.0-rc.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10252
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-controller-manager
    resources:
      requests:
        cpu: 200m
    volumeMounts:
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/kubernetes/controller-manager.conf
      name: kubeconfig
      readOnly: true
    - mountPath: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      name: flexvolume-dir
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    hostNetwork: true
    priorityClassName: system-cluster-critical
    volumes:
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/kubernetes/controller-manager.conf
      type: FileOrCreate
    name: kubeconfig
  - hostPath:
      path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      type: DirectoryOrCreate
    name: flexvolume-dir
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
      status: {}
      scheduler yaml文件：

[root@Ac-private-1 manifests]# cat kube-scheduler.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: k8s.gcr.io/kube-scheduler:v1.12.0-rc.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10251
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
    hostNetwork: true
    priorityClassName: system-cluster-critical
    volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
      status: {}
      /etc/kubernetes存放以上组件的配置文件   主要用于认证

admin.conf  for the cluster admin and kubeadm itself

kubelet.conf (bootstrap-kubelet.conf during TLS bootstrap) 

controller-manager.conf

scheduler.conf

/etc/kubernetes/pki主要存放证书以及密钥



kubeadm init流程
1.Preflight checks

在安装之前进行一系列检查，可以通过--ignore-preflight-errors跳过该步骤。具体检查如下：

检查k8s版本与kubeadm版本

[warning] If the Kubernetes version to use (specified with the --kubernetes-version flag) is at least one minor version higher than the kubeadm CLI version.
Kubernetes system requirements:
if running on linux:
检查内核版本
[error] if not Kernel 3.10+ or 4+ with specific KernelSpec
控制组功能
[error] if required cgroups subsystem aren’t in set up
if using docker:
[warning/error] if Docker service does not exist, if it is disabled, if it is not active.
安装docker环境
[error] if Docker endpoint does not exist or does not work
[warning] if docker version >17.03
If using other cri engine:
[error] if crictl socket does not answer
操作者为root
[error] if user is not root
[error] if the machine hostname is not a valid DNS subdomain
[warning] if the host name cannot be reached via network lookup
kubelet与kubeadm要相符
[error] if kubelet version is lower that the minimum kubelet version supported by kubeadm (current minor -1)
[error] if kubelet version is at least one minor higher than the required controlplane version (unsupported version skew)
[warning] if kubelet service does not exist or if it is disabled
[warning] if firewalld is active
[error] if API server bindPort or ports 10250/10251/10252 are used
配置文件路径
[Error] if /etc/kubernetes/manifest folder already exists and it is not empty
网络流量转发
[Error] if /proc/sys/net/bridge/bridge-nf-call-iptables file does not exist/does not contain 1
[Error] if advertise address is ipv6 and /proc/sys/net/bridge/bridge-nf-call-ip6tables does not exist/does not contain 1.
关闭swap功能
[Error] if swap is on
[Error] if ip, iptables, mount, nsenter commands are not present in the command path
[warning] if ebtables, ethtool, socat, tc, touch, crictl commands are not present in the command path
[warning] if extra arg flags for API server, controller manager, scheduler contains some invalid options
[warning] if connection to https://API.AdvertiseAddress:API.BindPort goes through proxy
[warning] if connection to services subnet goes through proxy (only first address checked)
[warning] if connection to Pods subnet goes through proxy (only first address checked)
etcd数据库检查
If external etcd is provided:
[Error] if etcd version less than 3.0.14
[Error] if etcd certificates or keys are specified, but not provided
If external etcd is NOT provided (and thus local etcd will be installed):
[Error] if ports 2379 is used
[Error] if Etcd.DataDir folder already exists and it is not empty
If authorization mode is ABAC:
[Error] if abac_policy.json does not exist
If authorization mode is WebHook
[Error] if webhook_authz.conf does not exist
2.Generate the necessary certificates

产生一系列证书以及密钥信息，存放在 /etc/kubernetes/pki目录，可以通过--cert-dir来修改 

k8s集群的证书以及密钥文件：ca.crt     ca.key

apiserver的证书以及密钥：apiserver.crt    apiserver.key

apiserver client连接kubelet的证书以及密钥： apiserver-kubelet-client.crt    apiserver-kubelet-client.key

signing ServiceAccount Token   sa.pub   sa.key

front proxy   front-proxy-ca.crt    front-proxy-ca.key

A client cert for the front proxy client   front-proxy-client.crt   front-proxy-client.key

3.Generate kubeconfig files for control plane components

A kubeconfig file for kubelet to use, /etc/kubernetes/kubelet.conf

A kubeconfig file for controller-manager, /etc/kubernetes/controller-manager.conf

A kubeconfig file for scheduler, /etc/kubernetes/scheduler.conf

4.Generate static Pod manifests for control plane components

主要产生apiserver  controller-manager  scheduler的yaml模板配置文件

writes static Pod manifest files for control plane components to /etc/kubernetes/manifests

apiserver的配置信息：

apiserver-advertise-address and apiserver-bind-port  ip以及端口

--insecure-port=0  to avoid insecure connections to the api server

--enable-bootstrap-token-auth=true to enable the BootstrapTokenAuthenticator authentication module. 

--allow-privileged to true (required e.g. by kube proxy)

--requestheader-client-ca-file to front-proxy-ca.crt

--enable-admission-plugins

开启API聚合功能：

--requestheader-username-headers=X-Remote-User

--requestheader-group-headers=X-Remote-Group

--requestheader-extra-headers-prefix=X-Remote-Extra-

--requestheader-allowed-names=front-proxy-client

5.Generate static Pod manifest for local etcd

产生etcd yaml模板文件

6.Optional Dynamic Kublet Configuration

Kubelet组件运行在Node节点上，维持运行中的Pods以及提供kuberntes运行时环境，主要完成以下使命： １．监视分配给该Node节点的pods ２．挂载pod所需要的volumes ３．下载pod的secret ４．通过docker/rkt来运行pod中的容器 ５．周期的执行pod中为容器定义的liveness探针 ６．上报pod的状态给系统的其他组件 ７．上报Node的状态

/var/lib/kubelet/config/init/kubelet

/etc/systemd/system/kubelet.service.d/10-kubeadm.conf   --dynamic-config-dir=/var/lib/kubelet/config/dynamic



7.Wait for the control plane to come up

kubeadm relies on the kubelet to pull the control plane images and run them properly as static Pods.下载镜像以及启动Pod

kubeadm waits until localhost:6443/healthz returns ok这一节点耗时较长

8.Write base kubelet configuration

If kubeadm is invoked with --feature-gates=DynamicKubeletConfig:

Write the kubelet base configuration into the kubelet-base-config-v1.9 ConfigMap in the kube-system namespace

Creates RBAC rules for granting read access to that ConfigMap to all bootstrap tokens and all kubelet instances (that is system:bootstrappers:kubeadm:default-node-token and system:nodes groups)

Enable the dynamic kubelet configuration feature for the initial control-plane node by pointing Node.spec.configSource to the newly-created ConfigMap

如果启动该选项，记录kubelet的配置文件到ConfigMap中  创建一个可以读取该配置的RBAC，允许动态该配置

9.Save the kubeadm ClusterConfiguration in a ConfigMap for later reference

This will ensure that kubeadm actions executed in future (e.g kubeadm upgrade) will be able to determine the actual/current cluster state and make new decisions based on that data

10.Mark master

As soon as the control plane is available, kubeadm executes following actions:

Label the master with node-role.kubernetes.io/master=""
Taints the master with node-role.kubernetes.io/master:NoSchedule
给Master打标签master 设置master不调度pod对象

11.Configure TLS-Bootstrapping for node joining

Create a bootstrap token

Allow joining nodes to call CSR API

Setup auto approval for new bootstrap tokens

Setup nodes certificate rotation with auto approval

Create the public cluster-info ConfigMap

12.Install addons

安装kube-proxy以及DNS

13.Optional self-hosting

使用kubeadm需要安装的镜像： k8s.gcr.io仓库访问不到，可以下载其他镜像，然后利用docker tag修改镜像标签



kubeadm  init启动流程很复杂，先粗略学习一下，具体每个细节在以后过程中慢慢研究。

参考链接：

https://kubernetes.io/docs/reference/setup-tools/kubeadm/implementation-details/
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84852152

# Kubernetes15--Custom-Metrics自定义监控指标

Metric可以实现对于pod以及node对象cpu,内存指标的监控，要想获取更多的监控指标，可以使用custom-metrics自定义监控指标。

部署prometheus监控系统，prometheus采集其它各种指标，使用组件kube-state-metrics，将prometheus的metrics数据格式转换成k8s API接口能识别的格式，转换以后，用Kubernetes aggregator在主API服务器中注册，以便直接通过/apis/来访问。

Custom Metrics 部署流程

1. node-exporter：prometheus的agent端，收集Node级别的监控数据。

2. prometheus：监控服务端，从node-exporter拉数据并存储为时序数据。

3. kube-state-metrics： 将prometheus中可以用PromQL查询到的指标数据转换成k8s对应的数据格式，即转换成【Custerom Metrics API】接口格式的数据，但是它不能聚合进apiserver中的功能。

4. k8s-prometheus-adpater：聚合apiserver，即提供了一个apiserver【cuester-metrics-api】，自定义APIServer通常都要通过Kubernetes aggregator聚合到apiserver。

1.创建monitor命名空间：

[root@Ac-private-1 custom-metrics]# cat monitor-namespace.yaml 
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
kubectl apply -f monitor-namespace.yaml



2.部署node-exporter服务：

node-exporter：prometheus的agent端，收集Node级别的监控数据

https://github.com/mgxian/k8s-monitor/blob/master/node_exporter.yaml

---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    k8s-app: node-exporter
spec:
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - containerPort: 9100
          protocol: TCP
          name: http
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: monitoring
spec:
  ports:
  - name: http
    port: 9100
    nodePort: 31672
    protocol: TCP
    type: NodePort
    selector:
    k8s-app: node-exporter


端口映射31672---9100，宿主机暴露了31672端口：

curl http://192.168.1.16:31672/metrics

可以访问到宿主机的性能指标数据：cpu,mem,fs,net等指标数据

3.部署prometheus

prometheus：监控服务端，从node-exporter拉数据并存储为时序数据

https://github.com/stefanprodan/k8s-prom-hpa/tree/master/prometheus



rbac.yaml配置文件：

[root@Ac-private-1 prometheus]# cat prometheus-rbac.yaml 
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
  svc.yaml配置文件

[root@Ac-private-1 prometheus]# cat prometheus-svc.yaml 
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
spec:
  type: NodePort
  ports:
    - port: 9090
      targetPort: 9090
      nodePort: 31190
      protocol: TCP
  selector:
    app: prometheus
dep.yaml配置文件：

[root@Ac-private-1 prometheus]# cat prometheus-dep.yaml 
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
      annotations:
        prometheus.io/scrape: 'false'
    spec:
      serviceAccountName: prometheus
      containers:
      - name: prometheus
        image: prom/prometheus:v2.1.0
        imagePullPolicy: Always
        command:
          - prometheus
          - --config.file=/etc/prometheus/prometheus.yml
          - --storage.tsdb.retention=1h
        ports:
        - containerPort: 9090
          protocol: TCP
        resources:
          limits:
            memory: 2Gi
        volumeMounts:
        - mountPath: /etc/prometheus/prometheus.yml
          name: prometheus-config
          subPath: prometheus.yml
      volumes:
        - name: prometheus-config
          configMap:
            name: prometheus-config
            items:
              - key: prometheus.yml
                path: prometheus.yml
                mode: 0644
cfg.yaml配置文件：

[root@Ac-private-1 prometheus]# cat prometheus-cfg.yaml
---
kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    app: prometheus
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    # A scrape configuration for running Prometheus on a Kubernetes cluster.
    # This uses separate scrape configs for cluster components (i.e. API server, node)
    # and services to allow each to use different authentication configs.
    #
    # Kubernetes labels will be added as Prometheus labels on metrics via the
    # `labelmap` relabeling action.
    #
    # If you are using Kubernetes 1.7.2 or earlier, please take note of the comments
    # for the kubernetes-cadvisor job; you will need to edit or remove this job.
    # Scrape config for API servers.
    #
    # Kubernetes exposes API servers as endpoints to the default/kubernetes
    # service so this uses `endpoints` role and uses relabelling to only keep
    # the endpoints associated with the default/kubernetes service using the
    # default named port `https`. This works for single API server deployments as
    # well as HA API server deployments.
    global:
      scrape_interval: 15s
      scrape_timeout: 10s
      evaluation_interval: 1m
    scrape_configs:
    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      # Default to scraping over https. If required, just disable this or change to
      # `http`.
      scheme: https
      # This TLS & bearer token file config is used to connect to the actual scrape
      # endpoints for cluster components. This is separate to discovery auth
      # configuration because discovery & scraping are two separate concerns in
      # Prometheus. The discovery auth config is automatic if Prometheus runs inside
      # the cluster. Otherwise, more config options have to be provided within the
      # <kubernetes_sd_config>.
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        # If your node certificates are self-signed or use a different CA to the
        # master CA, then disable certificate verification below. Note that
        # certificate verification is an integral part of a secure infrastructure
        # so this should only be disabled in a controlled environment. You can
        # disable certificate verification by uncommenting the line below.
        #
        # insecure_skip_verify: true
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      # Keep only the default/kubernetes service endpoints for the https port. This
      # will add targets for each API server which Kubernetes adds an endpoint to
      # the default/kubernetes service.
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
    - job_name: 'kubernetes-nodes'
      # Default to scraping over https. If required, just disable this or change to
      # `http`.
      scheme: https
      # This TLS & bearer token file config is used to connect to the actual scrape
      # endpoints for cluster components. This is separate to discovery auth
      # configuration because discovery & scraping are two separate concerns in
      # Prometheus. The discovery auth config is automatic if Prometheus runs inside
      # the cluster. Otherwise, more config options have to be provided within the
      # <kubernetes_sd_config>.
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: 192.168.1.15:6443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics
    # Scrape config for Kubelet cAdvisor.
    #
    # This is required for Kubernetes 1.7.3 and later, where cAdvisor metrics
    # (those whose names begin with 'container_') have been removed from the
    # Kubelet metrics endpoint.  This job scrapes the cAdvisor endpoint to
    # retrieve those metrics.
    #
    # In Kubernetes 1.7.0-1.7.2, these metrics are only exposed on the cAdvisor
    # HTTP endpoint; use "replacement: /api/v1/nodes/${1}:4194/proxy/metrics"
    # in that case (and ensure cAdvisor's HTTP server hasn't been disabled with
    # the --cadvisor-port=0 Kubelet flag).
    #
    # This job is not necessary and should be removed in Kubernetes 1.6 and
    # earlier versions, or it will cause the metrics to be scraped twice.
    - job_name: 'kubernetes-cadvisor'
      # Default to scraping over https. If required, just disable this or change to
      # `http`.
      scheme: https
      # This TLS & bearer token file config is used to connect to the actual scrape
      # endpoints for cluster components. This is separate to discovery auth
      # configuration because discovery & scraping are two separate concerns in
      # Prometheus. The discovery auth config is automatic if Prometheus runs inside
      # the cluster. Otherwise, more config options have to be provided within the
      # <kubernetes_sd_config>.
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: 192.168.1.15:6443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
    # Scrape config for service endpoints.
    #
    # The relabeling allows the actual service scrape endpoint to be configured
    # via the following annotations:
    #
    # * `prometheus.io/scrape`: Only scrape services that have a value of `true`
    # * `prometheus.io/scheme`: If the metrics endpoint is secured then you will need
    # to set this to `https` & most likely set the `tls_config` of the scrape config.
    # * `prometheus.io/path`: If the metrics path is not `/metrics` override this.
    # * `prometheus.io/port`: If the metrics are exposed on a different port to the
    # service then set this appropriately.
    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name
    # Example scrape config for pods
    #
    # The relabeling allows the actual pod scrape endpoint to be configured via the
    # following annotations:
    #
    # * `prometheus.io/scrape`: Only scrape pods that have a value of `true`
    # * `prometheus.io/path`: If the metrics path is not `/metrics` override this.
    # * `prometheus.io/port`: Scrape the pod on the indicated port instead of the
    # pod's declared ports (default is a port-free target if none are declared).
    - job_name: 'kubernetes-pods'
      # if you want to use metrics on jobs, set the below field to
      # true to prevent Prometheus from setting the `job` label
      # automatically.
      honor_labels: false
      kubernetes_sd_configs:
      - role: pod
      # skip verification so you can do HTTPS to pods
      tls_config:
        insecure_skip_verify: true
      # make sure your labels are in order
      relabel_configs:
      # these labels tell Prometheus to automatically attach source
      # pod and namespace information to each collected sample, so
      # that they'll be exposed in the custom metrics API automatically.
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: pod
      # these labels tell Prometheus to look for
      # prometheus.io/{scrape,path,port} annotations to configure
      # how to scrape
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (.+)
    - job_name: 'kubernetes-nodes-physical'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - source_labels: [__meta_kubernetes_role]
        action: replace
        target_label: kubernetes_role
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:31672'
        target_label: __address__
cfg中定义了prometheus中数据来源，其中Node物理节点数据由node-exporter来收集，容器数据使用Cadvisor来收集，注意端口号以及IP的对应性，保证可以正确访问到数据源



服务对外暴露31190端口，可以图形化访问服务以及各种性能指标

以下是prometheus接入数据源的信息



访问容器的IO状态



访问Node的IO状态



4.部署kube-state-metrics

将prometheus中可以用PromQL查询到的指标数据转换成k8s对应的数据格式，即转换成【Custerom Metrics API】接口格式的数据

https://github.com/mgxian/k8s-monitor/blob/master/kube-state-metrics.yaml



 

5.部署组件k8s-prometheus-adapter

https://github.com/stefanprodan/k8s-prom-hpa/tree/master/custom-metrics-api



6.Grafana图形化显示Promethues数据

部署Grafana

kubectl apply -f grafana.yaml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: daocloud.io/liukuan73/grafana:5.0.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /var
          name: grafana-storage
        env:
        - name: INFLUXDB_HOST
          value: monitoring-influxdb
        - name: GF_SERVER_HTTP_PORT
          value: "3000"
          # The following env variables are required to make Grafana accessible via
          # the kubernetes api-server proxy. On production clusters, we recommend
          # removing these env variables, setup auth for grafana, and expose the grafana
          # service using a LoadBalancer or a public IP.
        - name: GF_AUTH_BASIC_ENABLED
          value: "false"
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "true"
        - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          value: Admin
        - name: GF_SERVER_ROOT_URL
          # If you're only using the API Server proxy, set this value instead:
          # value: /api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
          value: /
      volumes:
      - name: grafana-storage
        emptyDir: {}
      #nodeSelector:
        #node-role.kubernetes.io/master: "true"
      #tolerations:
      #- key: "node-role.kubernetes.io/master"
        #effect: "NoSchedule"
---
apiVersion: v1
kind: Service
metadata:
  labels:
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-grafana
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/tcp-probe: 'true'
    prometheus.io/tcp-probe-port: '80'
  name: monitoring-grafana
  namespace: kube-system
spec:
  type: NodePort
  # In a production setup, we recommend accessing Grafana through an external Loadbalancer
  # or through a public IP.
  # type: LoadBalancer
  # You could also use NodePort to expose the service at a randomly-generated port
  # type: NodePort
  ports:
  - port: 80
    targetPort: 3000
    nodePort: 30007
    selector:
    k8s-app: grafana
    访问Grafanaweb页面    http://nodeIp:30007

    ​                                           

添加数据源：

​                                               

添加显示模板，模板文件可以参考官网https://grafana.com/dashboards

​                               

下载模板json文件，上传至grafana服务端

​                                  

查看效果：

​                                             

参考链接：

https://github.com/kubernetes-incubator/custom-metrics-apiserver/blob/master/docs/getting-started.md

https://github.com/stefanprodan/k8s-prom-hpa/tree/master/prometheus

https://cloud.tencent.com/info/d735182c2b8aa8380b5752dc5d2c972c.html

https://github.com/prometheus/node_exporter

https://www.kubernetes.org.cn/4438.html

https://blog.csdn.net/sinat_35930259/article/details/80456470

https://github.com/liukuan73/kubernetes-addons/blob/master/monitor/prometheus%2Bgrafana/node-exporter-daemonset.yaml

https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/prometheus

http://blog.51cto.com/newfly/2299768

https://blog.csdn.net/liukuan73/article/details/78881008

https://github.com/liukuan73/kubernetes-addons/blob/master/monitor
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84857773

# Kubernetes16--kube-scheduler源码1--Cobra.Command启动过程

进行k8s的源码解读，使用Windows环境，首先windows配置GO语言环境，IED使用IntelliJ IDEA，配置go语言开发环境。

下载k8s源码：

git clone https://github.com/kubernetes/kubernetes.git

GOPATH路径：F:\Go\src

工程路径: F:\Go\src\k8s.io\kubernetes

kube-scheduler程序入口：k8s.io/kubernetes/cmd/kube-scheduler/scheduler.go  main

scheduler插件式调度器：k8s.io/kubernetes/pkg/scheduler/scheduler.go

kube-scheduler程序入口：

func main() {
	rand.Seed(time.Now().UnixNano())

	command := app.NewSchedulerCommand()
	 
	// TODO: once we switch everything over to Cobra commands, we can go back to calling
	// utilflag.InitFlags() (by removing its pflag.Parse() call). For now, we have to set the
	// normalize func and add the go flag set by hand.
	pflag.CommandLine.SetNormalizeFunc(utilflag.WordSepNormalizeFunc)
	// utilflag.InitFlags()
	logs.InitLogs()
	defer logs.FlushLogs()
	 
	if err := command.Execute(); err != nil {
		fmt.Fprintf(os.Stderr, "%v\n", err)
		os.Exit(1)
	}
}
声明一个Command,然后执行Execute()方法来启动kube-scheduler,其中cobra是一个命令行启动框架。Cobra is both a library for creating powerful modern CLI applications as well as a program to generate applications and command files.https://github.com/spf13/cobra。

NewSchedulerCommand产生一个Command对象，使用一些默认的参数

// NewSchedulerCommand creates a *cobra.Command object with default parameters
func NewSchedulerCommand() *cobra.Command {
	opts, err := options.NewOptions()
	if err != nil {
		klog.Fatalf("unable to initialize command options: %v", err)
	}

	cmd := &cobra.Command{
		Use: "kube-scheduler",
		Long: `The Kubernetes scheduler is a policy-rich, topology-aware,
workload-specific function that significantly impacts availability, performance,
and capacity. The scheduler needs to take into account individual and collective
resource requirements, quality of service requirements, hardware/software/policy
constraints, affinity and anti-affinity specifications, data locality, inter-workload
interference, deadlines, and so on. Workload-specific requirements will be exposed
through the API as necessary.`,
		Run: func(cmd *cobra.Command, args []string) {
			if err := runCommand(cmd, args, opts); err != nil {
				fmt.Fprintf(os.Stderr, "%v\n", err)
				os.Exit(1)
			}
		},
	}
	fs := cmd.Flags()
	namedFlagSets := opts.Flags()
	verflag.AddFlags(namedFlagSets.FlagSet("global"))
	globalflag.AddGlobalFlags(namedFlagSets.FlagSet("global"), cmd.Name())
	for _, f := range namedFlagSets.FlagSets {
		fs.AddFlagSet(f)
	}

	usageFmt := "Usage:\n  %s\n"
	cols, _, _ := apiserverflag.TerminalSize(cmd.OutOrStdout())
	cmd.SetUsageFunc(func(cmd *cobra.Command) error {
		fmt.Fprintf(cmd.OutOrStderr(), usageFmt, cmd.UseLine())
		apiserverflag.PrintSections(cmd.OutOrStderr(), namedFlagSets, cols)
		return nil
	})
	cmd.SetHelpFunc(func(cmd *cobra.Command, args []string) {
		fmt.Fprintf(cmd.OutOrStdout(), "%s\n\n"+usageFmt, cmd.Long, cmd.UseLine())
		apiserverflag.PrintSections(cmd.OutOrStdout(), namedFlagSets, cols)
	})
	cmd.MarkFlagFilename("config", "yaml", "yml", "json")
	 
	return cmd
}
创建一个Options对象：opts, err := options.NewOptions()

// NewOptions returns default scheduler app options.
func NewOptions() (*Options, error) {
	cfg, err := newDefaultComponentConfig()
	if err != nil {
		return nil, err
	}

	hhost, hport, err := splitHostIntPort(cfg.HealthzBindAddress)
	if err != nil {
		return nil, err
	}
	 
	o := &Options{
		ComponentConfig: *cfg,
		SecureServing:   apiserveroptions.NewSecureServingOptions().WithLoopback(),
		CombinedInsecureServing: &CombinedInsecureServingOptions{
			Healthz: (&apiserveroptions.DeprecatedInsecureServingOptions{
				BindNetwork: "tcp",
			}).WithLoopback(),
			Metrics: (&apiserveroptions.DeprecatedInsecureServingOptions{
				BindNetwork: "tcp",
			}).WithLoopback(),
			BindPort:    hport,
			BindAddress: hhost,
		},
		Authentication: apiserveroptions.NewDelegatingAuthenticationOptions(),
		Authorization:  apiserveroptions.NewDelegatingAuthorizationOptions(),
		Deprecated: &DeprecatedOptions{
			UseLegacyPolicyConfig:    false,
			PolicyConfigMapNamespace: metav1.NamespaceSystem,
		},
	}
	 
	o.Authentication.TolerateInClusterLookupFailure = true
	o.Authentication.RemoteKubeConfigFileOptional = true
	o.Authorization.RemoteKubeConfigFileOptional = true
	o.Authorization.AlwaysAllowPaths = []string{"/healthz"}
	 
	// Set the PairName but leave certificate directory blank to generate in-memory by default
	o.SecureServing.ServerCert.CertDirectory = ""
	o.SecureServing.ServerCert.PairName = "kube-scheduler"
	o.SecureServing.BindPort = ports.KubeSchedulerPort
	 
	return o, nil
}
以下newDefaultComponentConfig()一个默认的组件配置，返回一个KubeSchedulerConfiguration对象
func newDefaultComponentConfig() (*kubeschedulerconfig.KubeSchedulerConfiguration, error) {
	cfgv1alpha1 := kubeschedulerconfigv1alpha1.KubeSchedulerConfiguration{}
	kubeschedulerscheme.Scheme.Default(&cfgv1alpha1)
	cfg := kubeschedulerconfig.KubeSchedulerConfiguration{}
	if err := kubeschedulerscheme.Scheme.Convert(&cfgv1alpha1, &cfg, nil); err != nil {
		return nil, err
	}
	return &cfg, nil
}
看一下KubeSchedulerConfiguration对象配置

// KubeSchedulerConfiguration configures a scheduler
type KubeSchedulerConfiguration struct {
	metav1.TypeMeta

	// SchedulerName is name of the scheduler, used to select which pods
	// will be processed by this scheduler, based on pod's "spec.SchedulerName".
	SchedulerName string
	// AlgorithmSource specifies the scheduler algorithm source.
	AlgorithmSource SchedulerAlgorithmSource
	// RequiredDuringScheduling affinity is not symmetric, but there is an implicit PreferredDuringScheduling affinity rule
	// corresponding to every RequiredDuringScheduling affinity rule.
	// HardPodAffinitySymmetricWeight represents the weight of implicit PreferredDuringScheduling affinity rule, in the range 0-100.
	HardPodAffinitySymmetricWeight int32
	 
	// LeaderElection defines the configuration of leader election client.
	LeaderElection KubeSchedulerLeaderElectionConfiguration
	 
	// ClientConnection specifies the kubeconfig file and client connection
	// settings for the proxy server to use when communicating with the apiserver.
	ClientConnection apimachineryconfig.ClientConnectionConfiguration
	// HealthzBindAddress is the IP address and port for the health check server to serve on,
	// defaulting to 0.0.0.0:10251
	HealthzBindAddress string
	// MetricsBindAddress is the IP address and port for the metrics server to
	// serve on, defaulting to 0.0.0.0:10251.
	MetricsBindAddress string
	 
	// DebuggingConfiguration holds configuration for Debugging related features
	// TODO: We might wanna make this a substruct like Debugging apiserverconfig.DebuggingConfiguration
	apiserverconfig.DebuggingConfiguration
	 
	// DisablePreemption disables the pod preemption feature.
	DisablePreemption bool
	 
	// PercentageOfNodeToScore is the percentage of all nodes that once found feasible
	// for running a pod, the scheduler stops its search for more feasible nodes in
	// the cluster. This helps improve scheduler's performance. Scheduler always tries to find
	// at least "minFeasibleNodesToFind" feasible nodes no matter what the value of this flag is.
	// Example: if the cluster size is 500 nodes and the value of this flag is 30,
	// then scheduler stops finding further feasible nodes once it finds 150 feasible ones.
	// When the value is 0, default percentage (50%) of the nodes will be scored.
	PercentageOfNodesToScore int32
	 
	// DEPRECATED.
	// Indicate the "all topologies" set for empty topologyKey when it's used for PreferredDuringScheduling pod anti-affinity.
	FailureDomains string
	 
	// Duration to wait for a binding operation to complete before timing out
	// Value must be non-negative integer. The value zero indicates no waiting.
	// If this value is nil, the default value will be used.
	BindTimeoutSeconds *int64
}
SchedulerName调度器名字 定义Pod的对象时可以指定调度pod对象的调度器spec.SchedulerName
PercentageOfNodesToScore 计算Node分数的比分比，调度pod时并不需要计算所有Node节点
BindTimeoutSeconds绑定时间过期时间
AlgorithmSource定义调度算法来源，可以来自于file以及configmap等
// SchedulerAlgorithmSource is the source of a scheduler algorithm. One source
// field must be specified, and source fields are mutually exclusive.
type SchedulerAlgorithmSource struct {
	// Policy is a policy based algorithm source.
	Policy *SchedulerPolicySource
	// Provider is the name of a scheduling algorithm provider to use.
	Provider *string
}

// SchedulerPolicySource configures a means to obtain a scheduler Policy. One
// source field must be specified, and source fields are mutually exclusive.
type SchedulerPolicySource struct {
	// File is a file policy source.
	File *SchedulerPolicyFileSource
	// ConfigMap is a config map policy source.
	ConfigMap *SchedulerPolicyConfigMapSource
}

// SchedulerPolicyFileSource is a policy serialized to disk and accessed via
// path.
type SchedulerPolicyFileSource struct {
	// Path is the location of a serialized policy.
	Path string
}

// SchedulerPolicyConfigMapSource is a policy serialized into a config map value
// under the SchedulerPolicyConfigMapKey key.
type SchedulerPolicyConfigMapSource struct {
	// Namespace is the namespace of the policy config map.
	Namespace string
	// Name is the name of hte policy config map.
	Name string
}
看一下默认的配置：k8s.io/kube-scheduler/config/v1alpha1

// KubeSchedulerConfiguration configures a scheduler
type KubeSchedulerConfiguration struct {
	metav1.TypeMeta `json:",inline"`
	SchedulerName string `json:"schedulerName"`
	AlgorithmSource SchedulerAlgorithmSource `json:"algorithmSource"`
	HardPodAffinitySymmetricWeight int32 `json:"hardPodAffinitySymmetricWeight"`
	LeaderElection KubeSchedulerLeaderElectionConfiguration `json:"leaderElection"`
	ClientConnection apimachineryconfigv1alpha1.ClientConnectionConfiguration `json:"clientConnection"`
	HealthzBindAddress string `json:"healthzBindAddress"`
	MetricsBindAddress string `json:"metricsBindAddress"`
	apiserverconfigv1alpha1.DebuggingConfiguration `json:",inline"`
	DisablePreemption bool `json:"disablePreemption"`
	PercentageOfNodesToScore int32 `json:"percentageOfNodesToScore"`
	FailureDomains string `json:"failureDomains"`
	BindTimeoutSeconds *int64 `json:"bindTimeoutSeconds"`
}
// SchedulerDefaultProviderName defines the default provider names
SchedulerDefaultProviderName = "DefaultProvider"使用默认的调度器
以下是kube-scheduler启动时设置参数



总结：

利用默认配置以及命令行参数逐次构建Cobra.Command对象，然后启动kube-scheduelr执行调度过程

KubeSchedulerConfiguration---Options---cobra.Command
参考链接：

https://blog.csdn.net/skytoup/article/details/50063037

https://www.cnblogs.com/zuochuang/p/9105642.html

http://www.cnblogs.com/cloudgeek/p/9939233.html
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84950889

# Kubernetes17--kube-scheduler源码2--Scheduler构造

kube-scehduler启动，使用默认的调度算法，调度器监听apiserver,寻找待调度的Pod对象以及可用Node节点，执行调度算法，将Pod调度到某一Node，然后回写apiserver。流程如下：

​                                    

调度器代码位置：\k8s.io\kubernetes\pkg\scheduler\scheduler.go

研究一下Scheduler对象的产生

核心对象介绍：
Scheduler 

观测未调度的Pods，给Pod指定合适的Node节点，将调度结果写回apiserver  可知其内部由一个Config对象构成

type Scheduler struct {
	config *factory.Config
}
schedulerOptions 

定义调度器的一些配置参数

type schedulerOptions struct {
	schedulerName                  string
	hardPodAffinitySymmetricWeight int32
	enableEquivalenceClassCache    bool
	disablePreemption              bool
	percentageOfNodesToScore       int32
	bindTimeoutSeconds             int64
}
默认的配置参数

var defaultSchedulerOptions = schedulerOptions{
	schedulerName:                  v1.DefaultSchedulerName,
	hardPodAffinitySymmetricWeight: v1.DefaultHardPodAffinitySymmetricWeight,
	enableEquivalenceClassCache:    false,
	disablePreemption:              false,
	percentageOfNodesToScore:       schedulerapi.DefaultPercentageOfNodesToScore,
	bindTimeoutSeconds:             BindTimeoutSeconds,
}
Config 

调度器参数配置对象  Config对象非常重要，直接组成Scheduler对象

type Config struct {
	SchedulerCache schedulerinternalcache.Cache
	Ecache     *equivalence.Cache
	NodeLister algorithm.NodeLister
	Algorithm  algorithm.ScheduleAlgorithm
	GetBinder  func(pod *v1.Pod) Binder
	PodConditionUpdater PodConditionUpdater
	PodPreemptor PodPreemptor
	PluginSet pluginsv1alpha1.PluginSet
	NextPod func() *v1.Pod
	WaitForCacheSync func() bool
	Error func(*v1.Pod, error)
	Recorder record.EventRecorder
	StopEverything <-chan struct{}
	VolumeBinder *volumebinder.VolumeBinder
	DisablePreemption bool
	SchedulingQueue internalqueue.SchedulingQueue
}
ConfigFactoryArgs   

定义产生Configurator的参数

// ConfigFactoryArgs is a set arguments passed to NewConfigFactory.
type ConfigFactoryArgs struct {
	SchedulerName                  string
	Client                         clientset.Interface
	NodeInformer                   coreinformers.NodeInformer
	PodInformer                    coreinformers.PodInformer
	PvInformer                     coreinformers.PersistentVolumeInformer
	PvcInformer                    coreinformers.PersistentVolumeClaimInformer
	ReplicationControllerInformer  coreinformers.ReplicationControllerInformer
	ReplicaSetInformer             appsinformers.ReplicaSetInformer
	StatefulSetInformer            appsinformers.StatefulSetInformer
	ServiceInformer                coreinformers.ServiceInformer
	PdbInformer                    policyinformers.PodDisruptionBudgetInformer
	StorageClassInformer           storageinformers.StorageClassInformer
	HardPodAffinitySymmetricWeight int32
	EnableEquivalenceClassCache    bool
	DisablePreemption              bool
	PercentageOfNodesToScore       int32
	BindTimeoutSeconds             int64
	StopCh                         <-chan struct{}
}
Configurator 

定义产生Config以及调度器等一些方法

type Configurator interface {
	// Exposed for testing
	GetHardPodAffinitySymmetricWeight() int32
	// Exposed for testing
	MakeDefaultErrorFunc(backoff *util.PodBackoff, podQueue internalqueue.SchedulingQueue) func(pod *v1.Pod, err error)
	// Predicate related accessors to be exposed for use by k8s.io/autoscaler/cluster-autoscaler
	GetPredicateMetadataProducer() (algorithm.PredicateMetadataProducer, error)
	GetPredicates(predicateKeys sets.String) (map[string]algorithm.FitPredicate, error)
	// Needs to be exposed for things like integration tests where we want to make fake nodes.
	GetNodeLister() corelisters.NodeLister
	// Exposed for testing
	GetClient() clientset.Interface
	// Exposed for testing
	GetScheduledPodLister() corelisters.PodLister
	Create() (*Config, error)
	CreateFromProvider(providerName string) (*Config, error)
	CreateFromConfig(policy schedulerapi.Policy) (*Config, error)
	CreateFromKeys(predicateKeys, priorityKeys sets.String, extenders []algorithm.SchedulerExtender) (*Config, error)
}
configFactory

Configurator的默认实现

Scheduler的产生
func New(client clientset.Interface,
	nodeInformer coreinformers.NodeInformer,
	podInformer coreinformers.PodInformer,
	pvInformer coreinformers.PersistentVolumeInformer,
	pvcInformer coreinformers.PersistentVolumeClaimInformer,
	replicationControllerInformer coreinformers.ReplicationControllerInformer,
	replicaSetInformer appsinformers.ReplicaSetInformer,
	statefulSetInformer appsinformers.StatefulSetInformer,
	serviceInformer coreinformers.ServiceInformer,
	pdbInformer policyinformers.PodDisruptionBudgetInformer,
	storageClassInformer storageinformers.StorageClassInformer,
	recorder record.EventRecorder,
	schedulerAlgorithmSource kubeschedulerconfig.SchedulerAlgorithmSource,
	stopCh <-chan struct{},
	opts ...func(o *schedulerOptions)) (*Scheduler, error) {

	options := defaultSchedulerOptions
	for _, opt := range opts {
		opt(&options)
	}
	 
	// Set up the configurator which can create schedulers from configs.
	configurator := factory.NewConfigFactory(&factory.ConfigFactoryArgs{
		SchedulerName:                  options.schedulerName,
		Client:                         client,
		NodeInformer:                   nodeInformer,
		PodInformer:                    podInformer,
		PvInformer:                     pvInformer,
		PvcInformer:                    pvcInformer,
		ReplicationControllerInformer:  replicationControllerInformer,
		ReplicaSetInformer:             replicaSetInformer,
		StatefulSetInformer:            statefulSetInformer,
		ServiceInformer:                serviceInformer,
		PdbInformer:                    pdbInformer,
		StorageClassInformer:           storageClassInformer,
		HardPodAffinitySymmetricWeight: options.hardPodAffinitySymmetricWeight,
		EnableEquivalenceClassCache:    options.enableEquivalenceClassCache,
		DisablePreemption:              options.disablePreemption,
		PercentageOfNodesToScore:       options.percentageOfNodesToScore,
		BindTimeoutSeconds:             options.bindTimeoutSeconds,
	})
	var config *factory.Config
	source := schedulerAlgorithmSource
	switch {
	case source.Provider != nil:
		// Create the config from a named algorithm provider.
		sc, err := configurator.CreateFromProvider(*source.Provider)
		if err != nil {
			return nil, fmt.Errorf("couldn't create scheduler using provider %q: %v", *source.Provider, err)
		}
		config = sc
	case source.Policy != nil:
		// Create the config from a user specified policy source.
		policy := &schedulerapi.Policy{}
		switch {
		case source.Policy.File != nil:
			if err := initPolicyFromFile(source.Policy.File.Path, policy); err != nil {
				return nil, err
			}
		case source.Policy.ConfigMap != nil:
			if err := initPolicyFromConfigMap(client, source.Policy.ConfigMap, policy); err != nil {
				return nil, err
			}
		}
		sc, err := configurator.CreateFromConfig(*policy)
		if err != nil {
			return nil, fmt.Errorf("couldn't create scheduler from policy: %v", err)
		}
		config = sc
	default:
		return nil, fmt.Errorf("unsupported algorithm source: %v", source)
	}
	// Additional tweaks to the config produced by the configurator.
	config.Recorder = recorder
	config.DisablePreemption = options.disablePreemption
	config.StopEverything = stopCh
	// Create the scheduler.
	sched := NewFromConfig(config)
	return sched, nil
}
由以上可知，Scheduler的产生由四个过程：

1.传入配置参数以及SchedulerOptions，构建ConfigFactoryArgs对象；

2.由factory.NewConfigFactory产生Configurator对象；

3.依据指定的调度算法源，由Configurator产生Config对象;

4.由Config对象构造Scheduler对象

NewConfigFactory实现ConfigFactoryArgs构造Configurator
调度器算法源对象决定调度器怎么产生，具体来说，来源有Policy以及Provider,  其中Policy由File以及ConfigMap两种方式

type SchedulerAlgorithmSource struct {
	Policy *SchedulerPolicySource
	Provider *string
}
type SchedulerPolicySource struct {
	File *SchedulerPolicyFileSource
	ConfigMap *SchedulerPolicyConfigMapSource
}
由Provider产生Config对象

case source.Provider != nil:
		// Create the config from a named algorithm provider.
		sc, err := configurator.CreateFromProvider(*source.Provider)
		if err != nil {
			return nil, fmt.Errorf("couldn't create scheduler using provider %q: %v", *source.Provider, err)
		}
		config = sc
configurator.CreateFromProvider(*source.Provider)方法
// Creates a scheduler from the name of a registered algorithm provider.
func (c *configFactory) CreateFromProvider(providerName string) (*Config, error) {
	klog.V(2).Infof("Creating scheduler from algorithm provider '%v'", providerName)
	provider, err := GetAlgorithmProvider(providerName)
	if err != nil {
		return nil, err
	}
	return c.CreateFromKeys(provider.FitPredicateKeys, provider.PriorityFunctionKeys, []algorithm.SchedulerExtender{})
}
GetAlgorithmProvider(providerName)方法  
// GetAlgorithmProvider should not be used to modify providers. It is publicly visible for testing.
func GetAlgorithmProvider(name string) (*AlgorithmProviderConfig, error) {
	schedulerFactoryMutex.Lock()
	defer schedulerFactoryMutex.Unlock()

	provider, ok := algorithmProviderMap[name]
	if !ok {
		return nil, fmt.Errorf("plugin %q has not been registered", name)
	}
	 
	return &provider, nil
}
algorithmProviderMap = make(map[string]AlgorithmProviderConfig) 存放调度算法名称以及对应配置的关系map 其中AlgorithmProviderConfig存放了预选以及优选策略
// AlgorithmProviderConfig is used to store the configuration of algorithm providers.
type AlgorithmProviderConfig struct {
	FitPredicateKeys     sets.String
	PriorityFunctionKeys sets.String
}
根据预选以及优选策略，产生Scheduler配置对象
CreateFromKeys(provider.FitPredicateKeys, provider.PriorityFunctionKeys, []algorithm.SchedulerExtender{})方法
// Creates a scheduler from a set of registered fit predicate keys and priority keys.
func (c *configFactory) CreateFromKeys(predicateKeys, priorityKeys sets.String, extenders []algorithm.SchedulerExtender) (*Config, error) {
	klog.V(2).Infof("Creating scheduler with fit predicates '%v' and priority functions '%v'", predicateKeys, priorityKeys)

	if c.GetHardPodAffinitySymmetricWeight() < 1 || c.GetHardPodAffinitySymmetricWeight() > 100 {
		return nil, fmt.Errorf("invalid hardPodAffinitySymmetricWeight: %d, must be in the range 1-100", c.GetHardPodAffinitySymmetricWeight())
	}
	 
	predicateFuncs, err := c.GetPredicates(predicateKeys)
	if err != nil {
		return nil, err
	}
	 
	priorityConfigs, err := c.GetPriorityFunctionConfigs(priorityKeys)
	if err != nil {
		return nil, err
	}
	 
	priorityMetaProducer, err := c.GetPriorityMetadataProducer()
	if err != nil {
		return nil, err
	}
	 
	predicateMetaProducer, err := c.GetPredicateMetadataProducer()
	if err != nil {
		return nil, err
	}
	 
	// TODO(bsalamat): the default registrar should be able to process config files.
	c.pluginSet = plugins.NewDefaultPluginSet(pluginsv1alpha1.NewPluginContext(), &c.schedulerCache)
	 
	// Init equivalence class cache
	if c.enableEquivalenceClassCache {
		c.equivalencePodCache = equivalence.NewCache(predicates.Ordering())
		klog.Info("Created equivalence class cache")
	}
	 
	algo := core.NewGenericScheduler(
		c.schedulerCache,
		c.equivalencePodCache,
		c.podQueue,
		predicateFuncs,
		predicateMetaProducer,
		priorityConfigs,
		priorityMetaProducer,
		c.pluginSet,
		extenders,
		c.volumeBinder,
		c.pVCLister,
		c.pdbLister,
		c.alwaysCheckAllPredicates,
		c.disablePreemption,
		c.percentageOfNodesToScore,
	)
	 
	podBackoff := util.CreateDefaultPodBackoff()
	return &Config{
		SchedulerCache: c.schedulerCache,
		Ecache:         c.equivalencePodCache,
		// The scheduler only needs to consider schedulable nodes.
		NodeLister:          &nodeLister{c.nodeLister},
		Algorithm:           algo,
		GetBinder:           c.getBinderFunc(extenders),
		PodConditionUpdater: &podConditionUpdater{c.client},
		PodPreemptor:        &podPreemptor{c.client},
		PluginSet:           c.pluginSet,
		WaitForCacheSync: func() bool {
			return cache.WaitForCacheSync(c.StopEverything, c.scheduledPodsHasSynced)
		},
		NextPod: func() *v1.Pod {
			return c.getNextPod()
		},
		Error:           c.MakeDefaultErrorFunc(podBackoff, c.podQueue),
		StopEverything:  c.StopEverything,
		VolumeBinder:    c.volumeBinder,
		SchedulingQueue: c.podQueue,
	}, nil
}
用File对象来构造Policy对象

case source.Policy.File != nil:
			if err := initPolicyFromFile(source.Policy.File.Path, policy); err != nil {
				return nil, err
			}
// initPolicyFromFile initialize policy from file
func initPolicyFromFile(policyFile string, policy *schedulerapi.Policy) error {
	// Use a policy serialized in a file.
	_, err := os.Stat(policyFile)
	if err != nil {
		return fmt.Errorf("missing policy config file %s", policyFile)
	}
	data, err := ioutil.ReadFile(policyFile)
	if err != nil {
		return fmt.Errorf("couldn't read policy config: %v", err)
	}
	err = runtime.DecodeInto(latestschedulerapi.Codec, []byte(data), policy)
	if err != nil {
		return fmt.Errorf("invalid policy: %v", err)
	}
	return nil
}
用ConfigMap来构造Policy对象

case source.Policy.ConfigMap != nil:
			if err := initPolicyFromConfigMap(client, source.Policy.ConfigMap, policy); err != nil {
				return nil, err
			}
		}
// initPolicyFromConfigMap initialize policy from configMap
func initPolicyFromConfigMap(client clientset.Interface, policyRef *kubeschedulerconfig.SchedulerPolicyConfigMapSource, policy *schedulerapi.Policy) error {
	// Use a policy serialized in a config map value.
	policyConfigMap, err := client.CoreV1().ConfigMaps(policyRef.Namespace).Get(policyRef.Name, metav1.GetOptions{})
	if err != nil {
		return fmt.Errorf("couldn't get policy config map %s/%s: %v", policyRef.Namespace, policyRef.Name, err)
	}
	data, found := policyConfigMap.Data[kubeschedulerconfig.SchedulerPolicyConfigMapKey]
	if !found {
		return fmt.Errorf("missing policy config map value at key %q", kubeschedulerconfig.SchedulerPolicyConfigMapKey)
	}
	err = runtime.DecodeInto(latestschedulerapi.Codec, []byte(data), policy)
	if err != nil {
		return fmt.Errorf("invalid policy: %v", err)
	}
	return nil
}
由Policy对象产生Config对象

// Creates a scheduler from the configuration file
func (c *configFactory) CreateFromConfig(policy schedulerapi.Policy) (*Config, error) {
	klog.V(2).Infof("Creating scheduler from configuration: %v", policy)

	// validate the policy configuration
	if err := validation.ValidatePolicy(policy); err != nil {
		return nil, err
	}
	 
	predicateKeys := sets.NewString()
	if policy.Predicates == nil {
		klog.V(2).Infof("Using predicates from algorithm provider '%v'", DefaultProvider)
		provider, err := GetAlgorithmProvider(DefaultProvider)
		if err != nil {
			return nil, err
		}
		predicateKeys = provider.FitPredicateKeys
	} else {
		for _, predicate := range policy.Predicates {
			klog.V(2).Infof("Registering predicate: %s", predicate.Name)
			predicateKeys.Insert(RegisterCustomFitPredicate(predicate))
		}
	}
	 
	priorityKeys := sets.NewString()
	if policy.Priorities == nil {
		klog.V(2).Infof("Using priorities from algorithm provider '%v'", DefaultProvider)
		provider, err := GetAlgorithmProvider(DefaultProvider)
		if err != nil {
			return nil, err
		}
		priorityKeys = provider.PriorityFunctionKeys
	} else {
		for _, priority := range policy.Priorities {
			klog.V(2).Infof("Registering priority: %s", priority.Name)
			priorityKeys.Insert(RegisterCustomPriorityFunction(priority))
		}
	}
	 
	var extenders []algorithm.SchedulerExtender
	if len(policy.ExtenderConfigs) != 0 {
		ignoredExtendedResources := sets.NewString()
		for ii := range policy.ExtenderConfigs {
			klog.V(2).Infof("Creating extender with config %+v", policy.ExtenderConfigs[ii])
			extender, err := core.NewHTTPExtender(&policy.ExtenderConfigs[ii])
			if err != nil {
				return nil, err
			}
			extenders = append(extenders, extender)
			for _, r := range policy.ExtenderConfigs[ii].ManagedResources {
				if r.IgnoredByScheduler {
					ignoredExtendedResources.Insert(string(r.Name))
				}
			}
		}
		predicates.RegisterPredicateMetadataProducerWithExtendedResourceOptions(ignoredExtendedResources)
	}
	// Providing HardPodAffinitySymmetricWeight in the policy config is the new and preferred way of providing the value.
	// Give it higher precedence than scheduler CLI configuration when it is provided.
	if policy.HardPodAffinitySymmetricWeight != 0 {
		c.hardPodAffinitySymmetricWeight = policy.HardPodAffinitySymmetricWeight
	}
	// When AlwaysCheckAllPredicates is set to true, scheduler checks all the configured
	// predicates even after one or more of them fails.
	if policy.AlwaysCheckAllPredicates {
		c.alwaysCheckAllPredicates = policy.AlwaysCheckAllPredicates
	}
	 
	return c.CreateFromKeys(predicateKeys, priorityKeys, extenders)
}
由Config对象产生Scheduler对象：

// NewFromConfig returns a new scheduler using the provided Config.
func NewFromConfig(config *factory.Config) *Scheduler {
	metrics.Register()
	return &Scheduler{
		config: config,
	}
}
总结：
Scheduler由以下方法产生：需要传入一系列参数以及schedulerOpitons对象；

func New(client clientset.Interface,
	nodeInformer coreinformers.NodeInformer,
	podInformer coreinformers.PodInformer,
	pvInformer coreinformers.PersistentVolumeInformer,
	pvcInformer coreinformers.PersistentVolumeClaimInformer,
	replicationControllerInformer coreinformers.ReplicationControllerInformer,
	replicaSetInformer appsinformers.ReplicaSetInformer,
	statefulSetInformer appsinformers.StatefulSetInformer,
	serviceInformer coreinformers.ServiceInformer,
	pdbInformer policyinformers.PodDisruptionBudgetInformer,
	storageClassInformer storageinformers.StorageClassInformer,
	recorder record.EventRecorder,
	schedulerAlgorithmSource kubeschedulerconfig.SchedulerAlgorithmSource,
	stopCh <-chan struct{},
	opts ...func(o *schedulerOptions)) (*Scheduler, error)
由一系列参数构建ConfigFactoryArgs对象；

&factory.ConfigFactoryArgs{
		SchedulerName:                  options.schedulerName,
		Client:                         client,
		NodeInformer:                   nodeInformer,
		PodInformer:                    podInformer,
		PvInformer:                     pvInformer,
		PvcInformer:                    pvcInformer,
		ReplicationControllerInformer:  replicationControllerInformer,
		ReplicaSetInformer:             replicaSetInformer,
		StatefulSetInformer:            statefulSetInformer,
		ServiceInformer:                serviceInformer,
		PdbInformer:                    pdbInformer,
		StorageClassInformer:           storageClassInformer,
		HardPodAffinitySymmetricWeight: options.hardPodAffinitySymmetricWeight,
		EnableEquivalenceClassCache:    options.enableEquivalenceClassCache,
		DisablePreemption:              options.disablePreemption,
		PercentageOfNodesToScore:       options.percentageOfNodesToScore,
		BindTimeoutSeconds:             options.bindTimeoutSeconds,
	}
由ConfigFactoryArgs对象构建Configurator对象；

func NewConfigFactory(args *ConfigFactoryArgs) Configurator
由schedulerAlgorithmSource中指定的方式来产生Config对象；

 由Provider直接产生Config

func (c *configFactory) CreateFromProvider(providerName string) (*Config, error)
由File,ConfigMap产生Policy，由Policy产生Config

func initPolicyFromFile(policyFile string, policy *schedulerapi.Policy) error
func initPolicyFromConfigMap(client clientset.Interface, policyRef *kubeschedulerconfig.SchedulerPolicyConfigMapSource, policy *schedulerapi.Policy) error
func (c *configFactory) CreateFromConfig(policy schedulerapi.Policy) (*Config, error)
由Config对象来产生Scheduler对象

func NewFromConfig(config *factory.Config) *Scheduler
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84954560

# Kubernetes18--kube-scheduler源码3--Scheduler执行过程

由New方法产生Scheduler对象，研究其执行过程

Scheduler的启动函数

func (sched *Scheduler) Run() {
	if !sched.config.WaitForCacheSync() {
		return
	}

	go wait.Until(sched.scheduleOne, 0, sched.config.StopEverything)
}
首先等待缓存同步  WaitForCacheSync waits for scheduler cache to populate  WaitForCacheSync方法是Config里面一个接口方法，CreateFromKeys中提供了一个默认的实现

WaitForCacheSync: func() bool {
			return cache.WaitForCacheSync(c.StopEverything, c.scheduledPodsHasSynced)
		}
func WaitForCacheSync(stopCh <-chan struct{}, cacheSyncs ...InformerSynced) bool {
	err := wait.PollUntil(syncedPollPeriod,
		func() (bool, error) {
			for _, syncFunc := range cacheSyncs {
				if !syncFunc() {
					return false, nil
				}
			}
			return true, nil
		},
		stopCh)
	if err != nil {
		klog.V(2).Infof("stop requested")
		return false
	}

	klog.V(4).Infof("caches populated")
	return true
}
调度流程的核心  scheduleOne方法
第一步获取待调度的Pod对象，如果待调度Pod为空，或者待调度的Pod已经删除，则直接返回

pod := sched.config.NextPod()
	// pod could be nil when schedulerQueue is closed
	if pod == nil {
		return
	}
	if pod.DeletionTimestamp != nil {
		sched.config.Recorder.Eventf(pod, v1.EventTypeWarning, "FailedScheduling", "skip schedule deleting pod: %v/%v", pod.Namespace, pod.Name)
		klog.V(3).Infof("Skip schedule deleting pod: %v/%v", pod.Namespace, pod.Name)
		return
	}
NextPod实现

NextPod: func() *v1.Pod {
			return c.getNextPod()
		}
func (c *configFactory) getNextPod() *v1.Pod {
	pod, err := c.podQueue.Pop()
	if err == nil {
		klog.V(4).Infof("About to try and schedule pod %v/%v", pod.Namespace, pod.Name)
		return pod
	}
	klog.Errorf("Error while retrieving next pod from scheduling queue: %v", err)
	return nil
}
configFactory中podQueue队列

// queue for pods that need scheduling
	podQueue internalqueue.SchedulingQueue
SchedulingQueue解析
SchedulingQueue一个用来保存待调度的Pod对象的队列，类似于FIFO以及Heap. SchedulingQueue is an interface for a queue to store pods waiting to be scheduled.The interface follows a pattern similar to cache.FIFO and cache.Heap and makes it easy to use those data structures as a SchedulingQueue.

type SchedulingQueue interface {
	Add(pod *v1.Pod) error
	AddIfNotPresent(pod *v1.Pod) error
	AddUnschedulableIfNotPresent(pod *v1.Pod) error
	// Pop removes the head of the queue and returns it. It blocks if the
	// queue is empty and waits until a new item is added to the queue.
	Pop() (*v1.Pod, error)
	Update(oldPod, newPod *v1.Pod) error
	Delete(pod *v1.Pod) error
	MoveAllToActiveQueue()
	AssignedPodAdded(pod *v1.Pod)
	AssignedPodUpdated(pod *v1.Pod)
	WaitingPodsForNode(nodeName string) []*v1.Pod
	WaitingPods() []*v1.Pod
	// Close closes the SchedulingQueue so that the goroutine which is
	// waiting to pop items can exit gracefully.
	Close()
	// DeleteNominatedPodIfExists deletes nominatedPod from internal cache
	DeleteNominatedPodIfExists(pod *v1.Pod)
}
SchedulingQueue构造方法，如果Pod启动优先级则使用优先级队列，没有优先级则使用FIFO

// NewSchedulingQueue initializes a new scheduling queue. If pod priority is
// enabled a priority queue is returned. If it is disabled, a FIFO is returned.
func NewSchedulingQueue(stop <-chan struct{}) SchedulingQueue {
	if util.PodPriorityEnabled() {
		return NewPriorityQueue(stop)
	}
	return NewFIFO()
}
优先级队列生成：

// NewPriorityQueue creates a PriorityQueue object.
func NewPriorityQueue(stop <-chan struct{}) *PriorityQueue {
	return NewPriorityQueueWithClock(stop, util.RealClock{})
}

// NewPriorityQueueWithClock creates a PriorityQueue which uses the passed clock for time.
func NewPriorityQueueWithClock(stop <-chan struct{}, clock util.Clock) *PriorityQueue {
	pq := &PriorityQueue{
		clock:          clock,
		stop:           stop,
		podBackoff:     util.CreatePodBackoffWithClock(1*time.Second, 10*time.Second, clock),
		activeQ:        util.NewHeap(cache.MetaNamespaceKeyFunc, activeQComp),
		unschedulableQ: newUnschedulablePodsMap(),
		nominatedPods:  map[string][]*v1.Pod{},
	}
	pq.cond.L = &pq.lock
	pq.podBackoffQ = util.NewHeap(cache.MetaNamespaceKeyFunc, pq.podsCompareBackoffCompleted)

	pq.run()
	 
	return pq
}
FIFO队列生成：

// NewFIFO creates a FIFO object.
func NewFIFO() *FIFO {
	return &FIFO{FIFO: cache.NewFIFO(cache.MetaNamespaceKeyFunc)}
}
Pop方法实现

func (f *FIFO) Pop() (*v1.Pod, error) {
	result, err := f.FIFO.Pop(func(obj interface{}) error { return nil })
	if err == cache.FIFOClosedError {
		return nil, fmt.Errorf(queueClosed)
	}
	return result.(*v1.Pod), err
}
为Pod选定Node主机
// Synchronously attempt to find a fit for the pod.
	start := time.Now()
	suggestedHost, err := sched.schedule(pod)
	if err != nil {
		// schedule() may have failed because the pod would not fit on any host, so we try to
		// preempt, with the expectation that the next time the pod is tried for scheduling it
		// will fit due to the preemption. It is also possible that a different pod will schedule
		// into the resources that were preempted, but this is harmless.
		if fitError, ok := err.(*core.FitError); ok {
			preemptionStartTime := time.Now()
			sched.preempt(pod, fitError)
			metrics.PreemptionAttempts.Inc()
			metrics.SchedulingAlgorithmPremptionEvaluationDuration.Observe(metrics.SinceInMicroseconds(preemptionStartTime))
			metrics.SchedulingLatency.WithLabelValues(metrics.PreemptionEvaluation).Observe(metrics.SinceInSeconds(preemptionStartTime))
			// Pod did not fit anywhere, so it is counted as a failure. If preemption
			// succeeds, the pod should get counted as a success the next time we try to
			// schedule it. (hopefully)
			metrics.PodScheduleFailures.Inc()
		} else {
			klog.Errorf("error selecting node for pod: %v", err)
			metrics.PodScheduleErrors.Inc()
		}
		return
	}
schedule方法

// schedule implements the scheduling algorithm and returns the suggested host.
func (sched *Scheduler) schedule(pod *v1.Pod) (string, error) {
	host, err := sched.config.Algorithm.Schedule(pod, sched.config.NodeLister)
	if err != nil {
		pod = pod.DeepCopy()
		sched.recordSchedulingFailure(pod, err, v1.PodReasonUnschedulable, err.Error())
		return "", err
	}
	return host, err
}
ScheduleAlgorithm定义了调度pod到node的方法，Schedule方法为Pod选定一个Node

// ScheduleAlgorithm is an interface implemented by things that know how to schedule pods
// onto machines.
type ScheduleAlgorithm interface {
	Schedule(*v1.Pod, NodeLister) (selectedMachine string, err error)
	// Preempt receives scheduling errors for a pod and tries to create room for
	// the pod by preempting lower priority pods if possible.
	// It returns the node where preemption happened, a list of preempted pods, a
	// list of pods whose nominated node name should be removed, and error if any.
	Preempt(*v1.Pod, NodeLister, error) (selectedNode *v1.Node, preemptedPods []*v1.Pod, cleanupNominatedPods []*v1.Pod, err error)
	// Predicates() returns a pointer to a map of predicate functions. This is
	// exposed for testing.
	Predicates() map[string]FitPredicate
	// Prioritizers returns a slice of priority config. This is exposed for
	// testing.
	Prioritizers() []PriorityConfig
}
如果选定失败，记录Pod调度失败

// recordFailedSchedulingEvent records an event for the pod that indicates the
// pod has failed to schedule.
// NOTE: This function modifies "pod". "pod" should be copied before being passed.
func (sched *Scheduler) recordSchedulingFailure(pod *v1.Pod, err error, reason string, message string) {
	sched.config.Error(pod, err)
	sched.config.Recorder.Event(pod, v1.EventTypeWarning, "FailedScheduling", message)
	sched.config.PodConditionUpdater.Update(pod, &v1.PodCondition{
		Type:    v1.PodScheduled,
		Status:  v1.ConditionFalse,
		Reason:  reason,
		Message: err.Error(),
	})
}
suggestedHost, err := sched.schedule(pod)如果执行失败，则取代当前待调度的Pod对象

if err != nil {
		// schedule() may have failed because the pod would not fit on any host, so we try to
		// preempt, with the expectation that the next time the pod is tried for scheduling it
		// will fit due to the preemption. It is also possible that a different pod will schedule
		// into the resources that were preempted, but this is harmless.
		if fitError, ok := err.(*core.FitError); ok {
			preemptionStartTime := time.Now()
			sched.preempt(pod, fitError)
			metrics.PreemptionAttempts.Inc()
			metrics.SchedulingAlgorithmPremptionEvaluationDuration.Observe(metrics.SinceInMicroseconds(preemptionStartTime))
			metrics.SchedulingLatency.WithLabelValues(metrics.PreemptionEvaluation).Observe(metrics.SinceInSeconds(preemptionStartTime))
			// Pod did not fit anywhere, so it is counted as a failure. If preemption
			// succeeds, the pod should get counted as a success the next time we try to
			// schedule it. (hopefully)
			metrics.PodScheduleFailures.Inc()
		} else {
			klog.Errorf("error selecting node for pod: %v", err)
			metrics.PodScheduleErrors.Inc()
		}
preempt方法  保存调度失败的Pod,选取下一个低优先级的Pod

// preempt tries to create room for a pod that has failed to schedule, by preempting lower priority pods if possible.
// If it succeeds, it adds the name of the node where preemption has happened to the pod annotations.
// It returns the node name and an error if any.
func (sched *Scheduler) preempt(preemptor *v1.Pod, scheduleErr error) (string, error) {
	if !util.PodPriorityEnabled() || sched.config.DisablePreemption {
		klog.V(3).Infof("Pod priority feature is not enabled or preemption is disabled by scheduler configuration." +
			" No preemption is performed.")
		return "", nil
	}
	preemptor, err := sched.config.PodPreemptor.GetUpdatedPod(preemptor)
	if err != nil {
		klog.Errorf("Error getting the updated preemptor pod object: %v", err)
		return "", err
	}

	node, victims, nominatedPodsToClear, err := sched.config.Algorithm.Preempt(preemptor, sched.config.NodeLister, scheduleErr)
	metrics.PreemptionVictims.Set(float64(len(victims)))
	if err != nil {
		klog.Errorf("Error preempting victims to make room for %v/%v.", preemptor.Namespace, preemptor.Name)
		return "", err
	}
	var nodeName = ""
	if node != nil {
		nodeName = node.Name
		err = sched.config.PodPreemptor.SetNominatedNodeName(preemptor, nodeName)
		if err != nil {
			klog.Errorf("Error in preemption process. Cannot update pod %v/%v annotations: %v", preemptor.Namespace, preemptor.Name, err)
			return "", err
		}
		for _, victim := range victims {
			if err := sched.config.PodPreemptor.DeletePod(victim); err != nil {
				klog.Errorf("Error preempting pod %v/%v: %v", victim.Namespace, victim.Name, err)
				return "", err
			}
			sched.config.Recorder.Eventf(victim, v1.EventTypeNormal, "Preempted", "by %v/%v on node %v", preemptor.Namespace, preemptor.Name, nodeName)
		}
	}
	// Clearing nominated pods should happen outside of "if node != nil". Node could
	// be nil when a pod with nominated node name is eligible to preempt again,
	// but preemption logic does not find any node for it. In that case Preempt()
	// function of generic_scheduler.go returns the pod itself for removal of the annotation.
	for _, p := range nominatedPodsToClear {
		rErr := sched.config.PodPreemptor.RemoveNominatedNodeName(p)
		if rErr != nil {
			klog.Errorf("Cannot remove nominated node annotation of pod: %v", rErr)
			// We do not return as this error is not critical.
		}
	}
	return nodeName, err
}
PodPreemptor解析  定义了修改Pod以及修改Node标签方法

// PodPreemptor has methods needed to delete a pod and to update
// annotations of the preemptor pod.
type PodPreemptor interface {
	GetUpdatedPod(pod *v1.Pod) (*v1.Pod, error)
	DeletePod(pod *v1.Pod) error
	SetNominatedNodeName(pod *v1.Pod, nominatedNode string) error
	RemoveNominatedNodeName(pod *v1.Pod) error
}
具体实现

type podPreemptor struct {
	Client clientset.Interface
}

func (p *podPreemptor) GetUpdatedPod(pod *v1.Pod) (*v1.Pod, error) {
	return p.Client.CoreV1().Pods(pod.Namespace).Get(pod.Name, metav1.GetOptions{})
}

func (p *podPreemptor) DeletePod(pod *v1.Pod) error {
	return p.Client.CoreV1().Pods(pod.Namespace).Delete(pod.Name, &metav1.DeleteOptions{})
}

func (p *podPreemptor) SetNominatedNodeName(pod *v1.Pod, nominatedNodeName string) error {
	podCopy := pod.DeepCopy()
	podCopy.Status.NominatedNodeName = nominatedNodeName
	_, err := p.Client.CoreV1().Pods(pod.Namespace).UpdateStatus(podCopy)
	return err
}

func (p *podPreemptor) RemoveNominatedNodeName(pod *v1.Pod) error {
	if len(pod.Status.NominatedNodeName) == 0 {
		return nil
	}
	return p.SetNominatedNodeName(pod, "")
}
如果schedule方法执行成功，为当前Pod返回一个合适的Node,则下一步执行assumeVolumes方法，在分配Pod之前首先执行目录卷的分配

// Tell the cache to assume that a pod now is running on a given node, even though it hasn't been bound yet.
	// This allows us to keep scheduling without waiting on binding to occur.
	assumedPod := pod.DeepCopy()

	// Assume volumes first before assuming the pod.
	//
	// If all volumes are completely bound, then allBound is true and binding will be skipped.
	//
	// Otherwise, binding of volumes is started after the pod is assumed, but before pod binding.
	//
	// This function modifies 'assumedPod' if volume binding is required.
	allBound, err := sched.assumeVolumes(assumedPod, suggestedHost)
	if err != nil {
		klog.Errorf("error assuming volumes: %v", err)
		metrics.PodScheduleErrors.Inc()
		return
	}
分配目录卷方法

// assumeVolumes will update the volume cache with the chosen bindings
//
// This function modifies assumed if volume binding is required.
func (sched *Scheduler) assumeVolumes(assumed *v1.Pod, host string) (allBound bool, err error) {
	if utilfeature.DefaultFeatureGate.Enabled(features.VolumeScheduling) {
		allBound, err = sched.config.VolumeBinder.Binder.AssumePodVolumes(assumed, host)
		if err != nil {
			sched.recordSchedulingFailure(assumed, err, SchedulerError,
				fmt.Sprintf("AssumePodVolumes failed: %v", err))
		}
		// Invalidate ecache because assumed volumes could have affected the cached
		// pvs for other pods
		if sched.config.Ecache != nil {
			invalidPredicates := sets.NewString(predicates.CheckVolumeBindingPred)
			sched.config.Ecache.InvalidatePredicates(invalidPredicates)
		}
	}
	return
}
// Run "reserve" plugins.
	for _, pl := range plugins.ReservePlugins() {
		if err := pl.Reserve(plugins, assumedPod, suggestedHost); err != nil {
			klog.Errorf("error while running %v reserve plugin for pod %v: %v", pl.Name(), assumedPod.Name, err)
			sched.recordSchedulingFailure(assumedPod, err, SchedulerError,
				fmt.Sprintf("reserve plugin %v failed", pl.Name()))
			metrics.PodScheduleErrors.Inc()
			return
		}
	}
分配方法assume  修改Pod Spec.NodeName标签为NodeName

// assume modifies `assumedPod` by setting NodeName=suggestedHost
	err = sched.assume(assumedPod, suggestedHost)
	if err != nil {
		klog.Errorf("error assuming pod: %v", err)
		metrics.PodScheduleErrors.Inc()
		return
	}
// assume signals to the cache that a pod is already in the cache, so that binding can be asynchronous.
// assume modifies `assumed`.
func (sched *Scheduler) assume(assumed *v1.Pod, host string) error {
	// Optimistically assume that the binding will succeed and send it to apiserver
	// in the background.
	// If the binding fails, scheduler will release resources allocated to assumed pod
	// immediately.
	assumed.Spec.NodeName = host
	// NOTE: Updates must be written to scheduler cache before invalidating
	// equivalence cache, because we could snapshot equivalence cache after the
	// invalidation and then snapshot the cache itself. If the cache is
	// snapshotted before updates are written, we would update equivalence
	// cache with stale information which is based on snapshot of old cache.
	if err := sched.config.SchedulerCache.AssumePod(assumed); err != nil {
		klog.Errorf("scheduler cache AssumePod failed: %v", err)

		// This is most probably result of a BUG in retrying logic.
		// We report an error here so that pod scheduling can be retried.
		// This relies on the fact that Error will check if the pod has been bound
		// to a node and if so will not add it back to the unscheduled pods queue
		// (otherwise this would cause an infinite loop).
		sched.recordSchedulingFailure(assumed, err, SchedulerError,
			fmt.Sprintf("AssumePod failed: %v", err))
		return err
	}
	// if "assumed" is a nominated pod, we should remove it from internal cache
	if sched.config.SchedulingQueue != nil {
		sched.config.SchedulingQueue.DeleteNominatedPodIfExists(assumed)
	}
	 
	// Optimistically assume that the binding will succeed, so we need to invalidate affected
	// predicates in equivalence cache.
	// If the binding fails, these invalidated item will not break anything.
	if sched.config.Ecache != nil {
		sched.config.Ecache.InvalidateCachedPredicateItemForPodAdd(assumed, host)
	}
	return nil
}
异步执行绑定过程

// bind the pod to its host asynchronously (we can do this b/c of the assumption step above).
	go func() {
		// Bind volumes first before Pod
		if !allBound {
			err := sched.bindVolumes(assumedPod)
			if err != nil {
				klog.Errorf("error binding volumes: %v", err)
				metrics.PodScheduleErrors.Inc()
				return
			}
		}

		// Run "prebind" plugins.
		for _, pl := range plugins.PrebindPlugins() {
			approved, err := pl.Prebind(plugins, assumedPod, suggestedHost)
			if err != nil {
				approved = false
				klog.Errorf("error while running %v prebind plugin for pod %v: %v", pl.Name(), assumedPod.Name, err)
				metrics.PodScheduleErrors.Inc()
			}
			if !approved {
				sched.Cache().ForgetPod(assumedPod)
				var reason string
				if err == nil {
					msg := fmt.Sprintf("prebind plugin %v rejected pod %v.", pl.Name(), assumedPod.Name)
					klog.V(4).Infof(msg)
					err = errors.New(msg)
					reason = v1.PodReasonUnschedulable
				} else {
					reason = SchedulerError
				}
				sched.recordSchedulingFailure(assumedPod, err, reason, err.Error())
				return
			}
		}
	 
		err := sched.bind(assumedPod, &v1.Binding{
			ObjectMeta: metav1.ObjectMeta{Namespace: assumedPod.Namespace, Name: assumedPod.Name, UID: assumedPod.UID},
			Target: v1.ObjectReference{
				Kind: "Node",
				Name: suggestedHost,
			},
		})
		metrics.E2eSchedulingLatency.Observe(metrics.SinceInMicroseconds(start))
		if err != nil {
			klog.Errorf("error binding pod: %v", err)
			metrics.PodScheduleErrors.Inc()
		} else {
			metrics.PodScheduleSuccesses.Inc()
		}
	}()
bindVolumes方法  执行目录绑定

// bindVolumes will make the API update with the assumed bindings and wait until
// the PV controller has completely finished the binding operation.

// If binding errors, times out or gets undone, then an error will be returned to
// retry scheduling.
func (sched *Scheduler) bindVolumes(assumed *v1.Pod) error {
	klog.V(5).Infof("Trying to bind volumes for pod \"%v/%v\"", assumed.Namespace, assumed.Name)
	err := sched.config.VolumeBinder.Binder.BindPodVolumes(assumed)
	if err != nil {
		klog.V(1).Infof("Failed to bind volumes for pod \"%v/%v\": %v", assumed.Namespace, assumed.Name, err)

		// Unassume the Pod and retry scheduling
		if forgetErr := sched.config.SchedulerCache.ForgetPod(assumed); forgetErr != nil {
			klog.Errorf("scheduler cache ForgetPod failed: %v", forgetErr)
		}
	 
		// Volumes may be bound by PV controller asynchronously, we must clear
		// stale pod binding cache.
		sched.config.VolumeBinder.DeletePodBindings(assumed)
	 
		sched.recordSchedulingFailure(assumed, err, "VolumeBindingFailed", err.Error())
		return err
	}
	 
	klog.V(5).Infof("Success binding volumes for pod \"%v/%v\"", assumed.Namespace, assumed.Name)
	return nil
}
执行prebind插件方法

// Run "prebind" plugins.
		for _, pl := range plugins.PrebindPlugins() {
			approved, err := pl.Prebind(plugins, assumedPod, suggestedHost)
			if err != nil {
				approved = false
				klog.Errorf("error while running %v prebind plugin for pod %v: %v", pl.Name(), assumedPod.Name, err)
				metrics.PodScheduleErrors.Inc()
			}
			if !approved {
				sched.Cache().ForgetPod(assumedPod)
				var reason string
				if err == nil {
					msg := fmt.Sprintf("prebind plugin %v rejected pod %v.", pl.Name(), assumedPod.Name)
					klog.V(4).Infof(msg)
					err = errors.New(msg)
					reason = v1.PodReasonUnschedulable
				} else {
					reason = SchedulerError
				}
				sched.recordSchedulingFailure(assumedPod, err, reason, err.Error())
				return
			}
		}
执行绑定过程bind：

err := sched.bind(assumedPod, &v1.Binding{
			ObjectMeta: metav1.ObjectMeta{Namespace: assumedPod.Namespace, Name: assumedPod.Name, UID: assumedPod.UID},
			Target: v1.ObjectReference{
				Kind: "Node",
				Name: suggestedHost,
			},
		})
bind方法

// bind binds a pod to a given node defined in a binding object.  We expect this to run asynchronously, so we
// handle binding metrics internally.
func (sched *Scheduler) bind(assumed *v1.Pod, b *v1.Binding) error {
	bindingStart := time.Now()
	// If binding succeeded then PodScheduled condition will be updated in apiserver so that
	// it's atomic with setting host.
	err := sched.config.GetBinder(assumed).Bind(b)
	if finErr := sched.config.SchedulerCache.FinishBinding(assumed); finErr != nil {
		klog.Errorf("scheduler cache FinishBinding failed: %v", finErr)
	}
	if err != nil {
		klog.V(1).Infof("Failed to bind pod: %v/%v", assumed.Namespace, assumed.Name)
		if err := sched.config.SchedulerCache.ForgetPod(assumed); err != nil {
			klog.Errorf("scheduler cache ForgetPod failed: %v", err)
		}
		sched.recordSchedulingFailure(assumed, err, SchedulerError,
			fmt.Sprintf("Binding rejected: %v", err))
		return err
	}

	metrics.BindingLatency.Observe(metrics.SinceInMicroseconds(bindingStart))
	metrics.SchedulingLatency.WithLabelValues(metrics.Binding).Observe(metrics.SinceInSeconds(bindingStart))
	sched.config.Recorder.Eventf(assumed, v1.EventTypeNormal, "Scheduled", "Successfully assigned %v/%v to %v", assumed.Namespace, assumed.Name, b.Target.Name)
	return nil
}
Bind方法

// Bind just does a POST binding RPC.
func (b *binder) Bind(binding *v1.Binding) error {
	klog.V(3).Infof("Attempting to bind %v to %v", binding.Name, binding.Target.Name)
	return b.Client.CoreV1().Pods(binding.Namespace).Bind(binding)
}
总结
scheduler的调度执行流程  代码位置\kubernetes\pkg\scheduler\scheduler.go

待调度的Pod对象组成一个队列SchedulingQueue，FIFO或者PriorityQueue

待调度的Node对象  List<Node>列表

NodeLister algorithm.NodeLister

nodes, err := nodeLister.List()

具体调度流程如下：

1.入口函数                           func (sched *Scheduler) Run()

2.调度一个Pod                    func (sched *Scheduler) scheduleOne()

3.获取一个待调度的Pod      pod := sched.config.NextPod()

4.为该Pod对象分配一个合适的Node    suggestedHost, err := sched.schedule(pod)

5.复制当前Pod              assumedPod := pod.DeepCopy()

6.分配目录卷                 allBound, err := sched.assumeVolumes(assumedPod, suggestedHost)

7.分配主机                     err = sched.assume(assumedPod, suggestedHost)

8.绑定目录卷                 err := sched.bindVolumes(assumedPod)

9.绑定主机                     func (sched *Scheduler) bind(assumed *v1.Pod, b *v1.Binding) error

核心为步骤4 ： 

sched.schedule(pod)

func (sched *Scheduler) schedule(pod *v1.Pod) (string, error)

host, err := sched.config.Algorithm.Schedule(pod, sched.config.NodeLister)

其中ScheduleAlgorithm中Schedule(*v1.Pod, NodeLister) (selectedMachine string, err error)方法由插件式算法定义。

接下来分析一下kube-scheduler如何注入插件式算法ScheduleAlgorithm，以及具体的schedule方法执行过程。
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84971344

# Kubernetes19--kube-scheduler源码4--Scheduler算法注册

研究一下kube-scheduler是如何构建Scheduler对象以及如何进行算法插件式注入

代码入口kubernetes\cmd\kube-scheduler\scheduler.go

首先构建SchedulerCommand对象

command := app.NewSchedulerCommand()
接下来执行runCommand

runCommand(cmd, args, opts)
根据配置信息启动Scheduler

Run(cc, stopCh)
构建scheduler对象

// Create the scheduler.
	sched, err := scheduler.New(cc.Client,
		cc.InformerFactory.Core().V1().Nodes(),
		cc.PodInformer,
		cc.InformerFactory.Core().V1().PersistentVolumes(),
		cc.InformerFactory.Core().V1().PersistentVolumeClaims(),
		cc.InformerFactory.Core().V1().ReplicationControllers(),
		cc.InformerFactory.Apps().V1().ReplicaSets(),
		cc.InformerFactory.Apps().V1().StatefulSets(),
		cc.InformerFactory.Core().V1().Services(),
		cc.InformerFactory.Policy().V1beta1().PodDisruptionBudgets(),
		storageClassInformer,
		cc.Recorder,
		cc.ComponentConfig.AlgorithmSource,
		stopCh,
		scheduler.WithName(cc.ComponentConfig.SchedulerName),
		scheduler.WithHardPodAffinitySymmetricWeight(cc.ComponentConfig.HardPodAffinitySymmetricWeight),
		scheduler.WithEquivalenceClassCacheEnabled(cc.ComponentConfig.EnableContentionProfiling),
		scheduler.WithPreemptionDisabled(cc.ComponentConfig.DisablePreemption),
		scheduler.WithPercentageOfNodesToScore(cc.ComponentConfig.PercentageOfNodesToScore),
		scheduler.WithBindTimeoutSeconds(*cc.ComponentConfig.BindTimeoutSeconds))
执行run方法

// Prepare a reusable runCommand function.
	run := func(ctx context.Context) {
		sched.Run()
		<-ctx.Done()
	}
接下来就是Scheduler的启动函数

func (sched *Scheduler) Run() {
	if !sched.config.WaitForCacheSync() {
		return
	}
	go wait.Until(sched.scheduleOne, 0, sched.config.StopEverything)
}
研究一下Run(cc, stopCh)中配置参数cc的由来：

func runCommand(cmd *cobra.Command, args []string, opts *options.Options) error

由Command对象以及命令行参数构建Config对象

c, err := opts.Config()

// Config return a scheduler config object
func (o *Options) Config() (*schedulerappconfig.Config, error) {
	if o.SecureServing != nil {
		if err := o.SecureServing.MaybeDefaultWithSelfSignedCerts("localhost", nil, []net.IP{net.ParseIP("127.0.0.1")}); err != nil {
			return nil, fmt.Errorf("error creating self-signed certificates: %v", err)
		}
	}

	c := &schedulerappconfig.Config{}
	if err := o.ApplyTo(c); err != nil {
		return nil, err
	}
	 
	// Prepare kube clients.
	client, leaderElectionClient, eventClient, err := createClients(c.ComponentConfig.ClientConnection, o.Master, c.ComponentConfig.LeaderElection.RenewDeadline.Duration)
	if err != nil {
		return nil, err
	}
	 
	// Prepare event clients.
	eventBroadcaster := record.NewBroadcaster()
	recorder := eventBroadcaster.NewRecorder(legacyscheme.Scheme, corev1.EventSource{Component: c.ComponentConfig.SchedulerName})
	 
	// Set up leader election if enabled.
	var leaderElectionConfig *leaderelection.LeaderElectionConfig
	if c.ComponentConfig.LeaderElection.LeaderElect {
		leaderElectionConfig, err = makeLeaderElectionConfig(c.ComponentConfig.LeaderElection, leaderElectionClient, recorder)
		if err != nil {
			return nil, err
		}
	}
	 
	c.Client = client
	c.InformerFactory = informers.NewSharedInformerFactory(client, 0)
	c.PodInformer = factory.NewPodInformer(client, 0)
	c.EventClient = eventClient
	c.Recorder = recorder
	c.Broadcaster = eventBroadcaster
	c.LeaderElection = leaderElectionConfig
	 
	return c, nil
}
cc := c.Complete()

// Complete fills in any fields not set that are required to have valid data. It's mutating the receiver.
func (c *Config) Complete() CompletedConfig {
	cc := completedConfig{c}

	if c.InsecureServing != nil {
		c.InsecureServing.Name = "healthz"
	}
	if c.InsecureMetricsServing != nil {
		c.InsecureMetricsServing.Name = "metrics"
	}
	 
	apiserver.AuthorizeClientBearerToken(c.LoopbackClientConfig, &c.Authentication, &c.Authorization)
	 
	return CompletedConfig{&cc}
}
algorithmprovider.ApplyFeatureGates()

Run(cc, stopCh)

Config对象 
Scheduler上下文配置

// Config has all the context to run a Scheduler
type Config struct {
	// config is the scheduler server's configuration object.
	ComponentConfig kubeschedulerconfig.KubeSchedulerConfiguration
	// LoopbackClientConfig is a config for a privileged loopback connection
	LoopbackClientConfig *restclient.Config
	InsecureServing        *apiserver.DeprecatedInsecureServingInfo // nil will disable serving on an insecure port
	InsecureMetricsServing *apiserver.DeprecatedInsecureServingInfo // non-nil if metrics should be served independently
	Authentication         apiserver.AuthenticationInfo
	Authorization          apiserver.AuthorizationInfo
	SecureServing          *apiserver.SecureServingInfo
	Client          clientset.Interface
	InformerFactory informers.SharedInformerFactory
	PodInformer     coreinformers.PodInformer
	EventClient     v1core.EventsGetter
	Recorder        record.EventRecorder
	Broadcaster     record.EventBroadcaster
	// LeaderElection is optional.
	LeaderElection *leaderelection.LeaderElectionConfig
}
scheduler server配置信息

// KubeSchedulerConfiguration configures a scheduler
type KubeSchedulerConfiguration struct {
	metav1.TypeMeta

	// SchedulerName is name of the scheduler, used to select which pods
	// will be processed by this scheduler, based on pod's "spec.SchedulerName".
	SchedulerName string
	// AlgorithmSource specifies the scheduler algorithm source.
	AlgorithmSource SchedulerAlgorithmSource
	// RequiredDuringScheduling affinity is not symmetric, but there is an implicit PreferredDuringScheduling affinity rule
	// corresponding to every RequiredDuringScheduling affinity rule.
	// HardPodAffinitySymmetricWeight represents the weight of implicit PreferredDuringScheduling affinity rule, in the range 0-100.
	HardPodAffinitySymmetricWeight int32
	 
	// LeaderElection defines the configuration of leader election client.
	LeaderElection KubeSchedulerLeaderElectionConfiguration
	 
	// ClientConnection specifies the kubeconfig file and client connection
	// settings for the proxy server to use when communicating with the apiserver.
	ClientConnection apimachineryconfig.ClientConnectionConfiguration
	// HealthzBindAddress is the IP address and port for the health check server to serve on,
	// defaulting to 0.0.0.0:10251
	HealthzBindAddress string
	// MetricsBindAddress is the IP address and port for the metrics server to
	// serve on, defaulting to 0.0.0.0:10251.
	MetricsBindAddress string
	 
	// DebuggingConfiguration holds configuration for Debugging related features
	// TODO: We might wanna make this a substruct like Debugging apiserverconfig.DebuggingConfiguration
	apiserverconfig.DebuggingConfiguration
	 
	// DisablePreemption disables the pod preemption feature.
	DisablePreemption bool
	 
	// PercentageOfNodeToScore is the percentage of all nodes that once found feasible
	// for running a pod, the scheduler stops its search for more feasible nodes in
	// the cluster. This helps improve scheduler's performance. Scheduler always tries to find
	// at least "minFeasibleNodesToFind" feasible nodes no matter what the value of this flag is.
	// Example: if the cluster size is 500 nodes and the value of this flag is 30,
	// then scheduler stops finding further feasible nodes once it finds 150 feasible ones.
	// When the value is 0, default percentage (50%) of the nodes will be scored.
	PercentageOfNodesToScore int32
	 
	// DEPRECATED.
	// Indicate the "all topologies" set for empty topologyKey when it's used for PreferredDuringScheduling pod anti-affinity.
	FailureDomains string
	 
	// Duration to wait for a binding operation to complete before timing out
	// Value must be non-negative integer. The value zero indicates no waiting.
	// If this value is nil, the default value will be used.
	BindTimeoutSeconds *int64
}
默认算法的设定过程：kubernetes\pkg\scheduler\apis\config\v1alpha1\register.go

func init() {
	// We only register manually written functions here. The registration of the
	// generated functions takes place in the generated files. The separation
	// makes the code compile even when the generated files are missing.
		localSchemeBuilder.Register(addDefaultingFuncs)
}
func addDefaultingFuncs(scheme *runtime.Scheme) error {
	return RegisterDefaults(scheme)
}
func RegisterDefaults(scheme *runtime.Scheme) error {
	scheme.AddTypeDefaultingFunc(&v1alpha1.KubeSchedulerConfiguration{}, func(obj interface{}) {
		SetObjectDefaults_KubeSchedulerConfiguration(obj.(*v1alpha1.KubeSchedulerConfiguration))
	})
	return nil
}
func SetObjectDefaults_KubeSchedulerConfiguration(in *v1alpha1.KubeSchedulerConfiguration) {
	SetDefaults_KubeSchedulerConfiguration(in)
}
// SetDefaults_KubeSchedulerConfiguration sets additional defaults
func SetDefaults_KubeSchedulerConfiguration(obj *kubescedulerconfigv1alpha1.KubeSchedulerConfiguration) {
	if len(obj.SchedulerName) == 0 {
		obj.SchedulerName = api.DefaultSchedulerName
	}

	if obj.HardPodAffinitySymmetricWeight == 0 {
		obj.HardPodAffinitySymmetricWeight = api.DefaultHardPodAffinitySymmetricWeight
	}
	 
	if obj.AlgorithmSource.Policy == nil &&
		(obj.AlgorithmSource.Provider == nil || len(*obj.AlgorithmSource.Provider) == 0) {
		val := kubescedulerconfigv1alpha1.SchedulerDefaultProviderName
		obj.AlgorithmSource.Provider = &val
	}
	 
	if policy := obj.AlgorithmSource.Policy; policy != nil {
		if policy.ConfigMap != nil && len(policy.ConfigMap.Namespace) == 0 {
			obj.AlgorithmSource.Policy.ConfigMap.Namespace = api.NamespaceSystem
		}
	}
	 
	if host, port, err := net.SplitHostPort(obj.HealthzBindAddress); err == nil {
		if len(host) == 0 {
			host = "0.0.0.0"
		}
		obj.HealthzBindAddress = net.JoinHostPort(host, port)
	} else {
		obj.HealthzBindAddress = net.JoinHostPort("0.0.0.0", strconv.Itoa(ports.InsecureSchedulerPort))
	}
	 
	if host, port, err := net.SplitHostPort(obj.MetricsBindAddress); err == nil {
		if len(host) == 0 {
			host = "0.0.0.0"
		}
		obj.MetricsBindAddress = net.JoinHostPort(host, port)
	} else {
		obj.MetricsBindAddress = net.JoinHostPort("0.0.0.0", strconv.Itoa(ports.InsecureSchedulerPort))
	}
	 
	if len(obj.LeaderElection.LockObjectNamespace) == 0 {
		obj.LeaderElection.LockObjectNamespace = kubescedulerconfigv1alpha1.SchedulerDefaultLockObjectNamespace
	}
	if len(obj.LeaderElection.LockObjectName) == 0 {
		obj.LeaderElection.LockObjectName = kubescedulerconfigv1alpha1.SchedulerDefaultLockObjectName
	}
	 
	if obj.PercentageOfNodesToScore == 0 {
		// by default, stop finding feasible nodes once the number of feasible nodes is 50% of the cluster.
		obj.PercentageOfNodesToScore = 50
	}
	 
	if len(obj.FailureDomains) == 0 {
		obj.FailureDomains = kubeletapis.DefaultFailureDomains
	}
	 
	if len(obj.ClientConnection.ContentType) == 0 {
		obj.ClientConnection.ContentType = "application/vnd.kubernetes.protobuf"
	}
	// Scheduler has an opinion about QPS/Burst, setting specific defaults for itself, instead of generic settings.
	if obj.ClientConnection.QPS == 0.0 {
		obj.ClientConnection.QPS = 50.0
	}
	if obj.ClientConnection.Burst == 0 {
		obj.ClientConnection.Burst = 100
	}
	 
	// Use the default LeaderElectionConfiguration options
	apiserverconfigv1alpha1.RecommendedDefaultLeaderElectionConfiguration(&obj.LeaderElection.LeaderElectionConfiguration)
	 
	if obj.BindTimeoutSeconds == nil {
		defaultBindTimeoutSeconds := int64(600)
		obj.BindTimeoutSeconds = &defaultBindTimeoutSeconds
	}
}
可知默认调度算法是由register.go初始化完成的。

调度器名字

if len(obj.SchedulerName) == 0 {
		obj.SchedulerName = api.DefaultSchedulerName
	}
const (
	// "default-scheduler" is the name of default scheduler.
	DefaultSchedulerName = "default-scheduler"

	// RequiredDuringScheduling affinity is not symmetric, but there is an implicit PreferredDuringScheduling affinity rule
	// corresponding to every RequiredDuringScheduling affinity rule.
	// When the --hard-pod-affinity-weight scheduler flag is not specified,
	// DefaultHardPodAffinityWeight defines the weight of the implicit PreferredDuringScheduling affinity rule.
	DefaultHardPodAffinitySymmetricWeight int32 = 1
)
Provider提供者

if obj.AlgorithmSource.Policy == nil &&
		(obj.AlgorithmSource.Provider == nil || len(*obj.AlgorithmSource.Provider) == 0) {
		val := kubescedulerconfigv1alpha1.SchedulerDefaultProviderName
		obj.AlgorithmSource.Provider = &val
	}
const (
	// SchedulerDefaultLockObjectNamespace defines default scheduler lock object namespace ("kube-system")
    SchedulerDefaultLockObjectNamespace string = metav1.NamespaceSystem

	// SchedulerDefaultLockObjectName defines default scheduler lock object name ("kube-scheduler")
	SchedulerDefaultLockObjectName = "kube-scheduler"
	 
	// SchedulerDefaultProviderName defines the default provider names
	SchedulerDefaultProviderName = "DefaultProvider"
)
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84975886

# Kubernetes20--kube-scheduler源码5--Scheduler算法分析

kube-scheduler启动默认使用DefaultProvider算法，研究一下DefaultProvider的Schedule过程。

默认使用DefaultSchedulerName调度器  SchedulerDefaultProviderName调度算法

if len(obj.SchedulerName) == 0 {
		obj.SchedulerName = api.DefaultSchedulerName
	}

	if obj.HardPodAffinitySymmetricWeight == 0 {
		obj.HardPodAffinitySymmetricWeight = api.DefaultHardPodAffinitySymmetricWeight
	}
	 
	if obj.AlgorithmSource.Policy == nil &&
		(obj.AlgorithmSource.Provider == nil || len(*obj.AlgorithmSource.Provider) == 0) {
		val := kubescedulerconfigv1alpha1.SchedulerDefaultProviderName
		obj.AlgorithmSource.Provider = &val
	}
看一下默认调度算法的初始化过程  kubernetes/pkg/scheduler/algorithmprovider/defaults/defaults.go

注册预选工厂

factory.RegisterPredicateMetadataProducerFactory(
		func(args factory.PluginFactoryArgs) algorithm.PredicateMetadataProducer {
			return predicates.NewPredicateMetadataFactory(args.PodLister)
		})
注册优选工厂

factory.RegisterPriorityMetadataProducerFactory(
		func(args factory.PluginFactoryArgs) algorithm.PriorityMetadataProducer {
			return priorities.NewPriorityMetadataFactory(args.ServiceLister, args.ControllerLister, args.ReplicaSetLister, args.StatefulSetLister)
		})
注册默认的预选以及优选策略

registerAlgorithmProvider(defaultPredicates(), defaultPriorities())
注册一系列预选策略：

// PodFitsPorts has been replaced by PodFitsHostPorts for better user understanding.
	// For backwards compatibility with 1.0, PodFitsPorts is registered as well.
	factory.RegisterFitPredicate("PodFitsPorts", predicates.PodFitsHostPorts)
	// Fit is defined based on the absence of port conflicts.
	// This predicate is actually a default predicate, because it is invoked from
	// predicates.GeneralPredicates()
	factory.RegisterFitPredicate(predicates.PodFitsHostPortsPred, predicates.PodFitsHostPorts)
	// Fit is determined by resource availability.
	// This predicate is actually a default predicate, because it is invoked from
	// predicates.GeneralPredicates()
	factory.RegisterFitPredicate(predicates.PodFitsResourcesPred, predicates.PodFitsResources)
	// Fit is determined by the presence of the Host parameter and a string match
	// This predicate is actually a default predicate, because it is invoked from
	// predicates.GeneralPredicates()
	factory.RegisterFitPredicate(predicates.HostNamePred, predicates.PodFitsHost)
	// Fit is determined by node selector query.
	factory.RegisterFitPredicate(predicates.MatchNodeSelectorPred, predicates.PodMatchNodeSelector)
使同一个Service的不同Pod尽量分配到不同Node

factory.RegisterPriorityConfigFactory(
		"ServiceSpreadingPriority",
		factory.PriorityConfigFactory{
			MapReduceFunction: func(args factory.PluginFactoryArgs) (algorithm.PriorityMapFunction, algorithm.PriorityReduceFunction) {
				return priorities.NewSelectorSpreadPriority(args.ServiceLister, algorithm.EmptyControllerLister{}, algorithm.EmptyReplicaSetLister{}, algorithm.EmptyStatefulSetLister{})
			},
			Weight: 1,
		},
	)
注册优先级函数

// EqualPriority is a prioritizer function that gives an equal weight of one to all nodes
	// Register the priority function so that its available
	// but do not include it as part of the default priorities
	factory.RegisterPriorityFunction2("EqualPriority", core.EqualPriorityMap, nil, 1)
	// Optional, cluster-autoscaler friendly priority function - give used nodes higher priority.
	factory.RegisterPriorityFunction2("MostRequestedPriority", priorities.MostRequestedPriorityMap, nil, 1)
	factory.RegisterPriorityFunction2(
		"RequestedToCapacityRatioPriority",
		priorities.RequestedToCapacityRatioResourceAllocationPriorityDefault().PriorityMap,
		nil,
		1)
这里有个factory概念，研究一下  kubernetes/pkg/scheduler/factory/plugins.go

主要变量

var (
	schedulerFactoryMutex sync.Mutex

	// maps that hold registered algorithm types
	fitPredicateMap        = make(map[string]FitPredicateFactory)
	mandatoryFitPredicates = sets.NewString()
	priorityFunctionMap    = make(map[string]PriorityConfigFactory)
	algorithmProviderMap   = make(map[string]AlgorithmProviderConfig)
	 
	// Registered metadata producers
	priorityMetadataProducer  PriorityMetadataProducerFactory
	predicateMetadataProducer PredicateMetadataProducerFactory
)
预选工厂map

fitPredicateMap        = make(map[string]FitPredicateFactory)
由给定的参数产生一个预选策略

// FitPredicateFactory produces a FitPredicate from the given args.
type FitPredicateFactory func(PluginFactoryArgs) algorithm.FitPredicate
PluginFactoryArgs参数结构体

// PluginFactoryArgs are passed to all plugin factory functions.
type PluginFactoryArgs struct {
	PodLister                      algorithm.PodLister
	ServiceLister                  algorithm.ServiceLister
	ControllerLister               algorithm.ControllerLister
	ReplicaSetLister               algorithm.ReplicaSetLister
	StatefulSetLister              algorithm.StatefulSetLister
	NodeLister                     algorithm.NodeLister
	PDBLister                      algorithm.PDBLister
	NodeInfo                       predicates.NodeInfo
	PVInfo                         predicates.PersistentVolumeInfo
	PVCInfo                        predicates.PersistentVolumeClaimInfo
	StorageClassInfo               predicates.StorageClassInfo
	VolumeBinder                   *volumebinder.VolumeBinder
	HardPodAffinitySymmetricWeight int32
}
判断一个Pod是否适合一个Node

// FitPredicate is a function that indicates if a pod fits into an existing node.
// The failure information is given by the error.
type FitPredicate func(pod *v1.Pod, meta PredicateMetadata, nodeInfo *schedulercache.NodeInfo) (bool, []PredicateFailureReason, error)
优先级函数map

priorityFunctionMap    = make(map[string]PriorityConfigFactory)
// PriorityConfigFactory produces a PriorityConfig from the given function and weight
type PriorityConfigFactory struct {
	Function          PriorityFunctionFactory
	MapReduceFunction PriorityFunctionFactory2
	Weight            int
}
// PriorityFunctionFactory2 produces map & reduce priority functions
// from a given args.
// FIXME: Rename to PriorityFunctionFactory.
type PriorityFunctionFactory2 func(PluginFactoryArgs) (algorithm.PriorityMapFunction, algorithm.PriorityReduceFunction)
计算每一个Node分数以及所有Node最终分数

// PriorityMapFunction is a function that computes per-node results for a given node.
// TODO: Figure out the exact API of this method.
// TODO: Change interface{} to a specific type.
type PriorityMapFunction func(pod *v1.Pod, meta interface{}, nodeInfo *schedulercache.NodeInfo) (schedulerapi.HostPriority, error)

// PriorityReduceFunction is a function that aggregated per-node results and computes
 final scores for all nodes.
// TODO: Figure out the exact API of this method.
// TODO: Change interface{} to a specific type.
type PriorityReduceFunction func(pod *v1.Pod, meta interface{}, nodeNameToInfo map[string]*schedulercache.NodeInfo, result schedulerapi.HostPriorityList) error
algorithmProviderMap

algorithmProviderMap   = make(map[string]AlgorithmProviderConfig)
算法提供者Config,保存一系列预选以及优先级函数关键字

// AlgorithmProviderConfig is used to store the configuration of algorithm providers.
type AlgorithmProviderConfig struct {
	FitPredicateKeys     sets.String
	PriorityFunctionKeys sets.String
}
PredicateMetadataProducerFactory--PredicateMetadataProducer--PredicateMetadata

predicateMetadataProducer PredicateMetadataProducerFactory
// PredicateMetadataProducerFactory produces PredicateMetadataProducer from the given args.
type PredicateMetadataProducerFactory func(PluginFactoryArgs) algorithm.PredicateMetadataProducer
// PredicateMetadataProducer is a function that computes predicate metadata for a given pod.
type PredicateMetadataProducer func(pod *v1.Pod, nodeNameToInfo map[string]*schedulercache.NodeInfo) PredicateMetadata
// PredicateMetadata interface represents anything that can access a predicate metadata.
type PredicateMetadata interface {
	ShallowCopy() PredicateMetadata
	AddPod(addedPod *v1.Pod, nodeInfo *schedulercache.NodeInfo) error
	RemovePod(deletedPod *v1.Pod) error
}
PriorityMetadataProducerFactory--PriorityMetadataProducer

priorityMetadataProducer  PriorityMetadataProducerFactory
// PriorityMetadataProducerFactory produces PriorityMetadataProducer from the given args.
type PriorityMetadataProducerFactory func(PluginFactoryArgs) algorithm.PriorityMetadataProducer
// PriorityMetadataProducer is a function that computes metadata for a given pod. This
// is now used for only for priority functions. For predicates please use PredicateMetadataProducer.
type PriorityMetadataProducer func(pod *v1.Pod, nodeNameToInfo map[string]*schedulercache.NodeInfo) interface{}
注册算法提供者

func registerAlgorithmProvider(predSet, priSet sets.String) {
	// Registers algorithm providers. By default we use 'DefaultProvider', but user can specify one to be used
	// by specifying flag.
	factory.RegisterAlgorithmProvider(factory.DefaultProvider, predSet, priSet)
	// Cluster autoscaler friendly scheduling algorithm.
	factory.RegisterAlgorithmProvider(ClusterAutoscalerProvider, predSet,
		copyAndReplace(priSet, "LeastRequestedPriority", "MostRequestedPriority"))
}
需要提供算法名称以及相对应的预选以及优选策略关键字

// RegisterAlgorithmProvider registers a new algorithm provider with the algorithm registry. This should
// be called from the init function in a provider plugin.
func RegisterAlgorithmProvider(name string, predicateKeys, priorityKeys sets.String) string {
	schedulerFactoryMutex.Lock()
	defer schedulerFactoryMutex.Unlock()
	validateAlgorithmNameOrDie(name)
	algorithmProviderMap[name] = AlgorithmProviderConfig{
		FitPredicateKeys:     predicateKeys,
		PriorityFunctionKeys: priorityKeys,
	}
	return name
}
注册预选策略流程

factory.RegisterFitPredicate(predicates.PodFitsHostPortsPred, predicates.PodFitsHostPorts)
// RegisterFitPredicate registers a fit predicate with the algorithm
// registry. Returns the name with which the predicate was registered.
func RegisterFitPredicate(name string, predicate algorithm.FitPredicate) string {
	return RegisterFitPredicateFactory(name, func(PluginFactoryArgs) algorithm.FitPredicate { return predicate })
}
// RegisterFitPredicateFactory registers a fit predicate factory with the
// algorithm registry. Returns the name with which the predicate was registered.
func RegisterFitPredicateFactory(name string, predicateFactory FitPredicateFactory) string {
	schedulerFactoryMutex.Lock()
	defer schedulerFactoryMutex.Unlock()
	validateAlgorithmNameOrDie(name)
	fitPredicateMap[name] = predicateFactory
	return name
}
注册优先级函数流程：

factory.RegisterPriorityFunction2("MostRequestedPriority", priorities.MostRequestedPriorityMap, nil, 1)
// RegisterPriorityFunction2 registers a priority function with the algorithm registry. Returns the name,
// with which the function was registered.
// FIXME: Rename to PriorityFunctionFactory.
func RegisterPriorityFunction2(
	name string,
	mapFunction algorithm.PriorityMapFunction,
	reduceFunction algorithm.PriorityReduceFunction,
	weight int) string {
	return RegisterPriorityConfigFactory(name, PriorityConfigFactory{
		MapReduceFunction: func(PluginFactoryArgs) (algorithm.PriorityMapFunction, algorithm.PriorityReduceFunction) {
			return mapFunction, reduceFunction
		},
		Weight: weight,
	})
}
// RegisterPriorityConfigFactory registers a priority config factory with its name.
func RegisterPriorityConfigFactory(name string, pcf PriorityConfigFactory) string {
	schedulerFactoryMutex.Lock()
	defer schedulerFactoryMutex.Unlock()
	validateAlgorithmNameOrDie(name)
	priorityFunctionMap[name] = pcf
	return name
}
默认的预选策略：

func defaultPredicates() sets.String {
	return sets.NewString(
		// Fit is determined by volume zone requirements.
		factory.RegisterFitPredicateFactory(
			predicates.NoVolumeZoneConflictPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewVolumeZonePredicate(args.PVInfo, args.PVCInfo, args.StorageClassInfo)
			},
		),
		// Fit is determined by whether or not there would be too many AWS EBS volumes attached to the node
		factory.RegisterFitPredicateFactory(
			predicates.MaxEBSVolumeCountPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewMaxPDVolumeCountPredicate(predicates.EBSVolumeFilterType, args.PVInfo, args.PVCInfo)
			},
		),
		// Fit is determined by whether or not there would be too many GCE PD volumes attached to the node
		factory.RegisterFitPredicateFactory(
			predicates.MaxGCEPDVolumeCountPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewMaxPDVolumeCountPredicate(predicates.GCEPDVolumeFilterType, args.PVInfo, args.PVCInfo)
			},
		),
		// Fit is determined by whether or not there would be too many Azure Disk volumes attached to the node
		factory.RegisterFitPredicateFactory(
			predicates.MaxAzureDiskVolumeCountPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewMaxPDVolumeCountPredicate(predicates.AzureDiskVolumeFilterType, args.PVInfo, args.PVCInfo)
			},
		),
		factory.RegisterFitPredicateFactory(
			predicates.MaxCSIVolumeCountPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewCSIMaxVolumeLimitPredicate(args.PVInfo, args.PVCInfo)
			},
		),
		// Fit is determined by inter-pod affinity.
		factory.RegisterFitPredicateFactory(
			predicates.MatchInterPodAffinityPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewPodAffinityPredicate(args.NodeInfo, args.PodLister)
			},
		),

		// Fit is determined by non-conflicting disk volumes.
		factory.RegisterFitPredicate(predicates.NoDiskConflictPred, predicates.NoDiskConflict),
	 
		// GeneralPredicates are the predicates that are enforced by all Kubernetes components
		// (e.g. kubelet and all schedulers)
		factory.RegisterFitPredicate(predicates.GeneralPred, predicates.GeneralPredicates),
	 
		// Fit is determined by node memory pressure condition.
		factory.RegisterFitPredicate(predicates.CheckNodeMemoryPressurePred, predicates.CheckNodeMemoryPressurePredicate),
	 
		// Fit is determined by node disk pressure condition.
		factory.RegisterFitPredicate(predicates.CheckNodeDiskPressurePred, predicates.CheckNodeDiskPressurePredicate),
	 
		// Fit is determined by node pid pressure condition.
		factory.RegisterFitPredicate(predicates.CheckNodePIDPressurePred, predicates.CheckNodePIDPressurePredicate),
	 
		// Fit is determined by node conditions: not ready, network unavailable or out of disk.
		factory.RegisterMandatoryFitPredicate(predicates.CheckNodeConditionPred, predicates.CheckNodeConditionPredicate),
	 
		// Fit is determined based on whether a pod can tolerate all of the node's taints
		factory.RegisterFitPredicate(predicates.PodToleratesNodeTaintsPred, predicates.PodToleratesNodeTaints),
	 
		// Fit is determined by volume topology requirements.
		factory.RegisterFitPredicateFactory(
			predicates.CheckVolumeBindingPred,
			func(args factory.PluginFactoryArgs) algorithm.FitPredicate {
				return predicates.NewVolumeBindingPredicate(args.VolumeBinder)
			},
		),
	)
}
默认的优先级函数：

func defaultPriorities() sets.String {
	return sets.NewString(
		// spreads pods by minimizing the number of pods (belonging to the same service or replication controller) on the same node.
		factory.RegisterPriorityConfigFactory(
			"SelectorSpreadPriority",
			factory.PriorityConfigFactory{
				MapReduceFunction: func(args factory.PluginFactoryArgs) (algorithm.PriorityMapFunction, algorithm.PriorityReduceFunction) {
					return priorities.NewSelectorSpreadPriority(args.ServiceLister, args.ControllerLister, args.ReplicaSetLister, args.StatefulSetLister)
				},
				Weight: 1,
			},
		),
		// pods should be placed in the same topological domain (e.g. same node, same rack, same zone, same power domain, etc.)
		// as some other pods, or, conversely, should not be placed in the same topological domain as some other pods.
		factory.RegisterPriorityConfigFactory(
			"InterPodAffinityPriority",
			factory.PriorityConfigFactory{
				Function: func(args factory.PluginFactoryArgs) algorithm.PriorityFunction {
					return priorities.NewInterPodAffinityPriority(args.NodeInfo, args.NodeLister, args.PodLister, args.HardPodAffinitySymmetricWeight)
				},
				Weight: 1,
			},
		),

		// Prioritize nodes by least requested utilization.
		factory.RegisterPriorityFunction2("LeastRequestedPriority", priorities.LeastRequestedPriorityMap, nil, 1),
	 
		// Prioritizes nodes to help achieve balanced resource usage
		factory.RegisterPriorityFunction2("BalancedResourceAllocation", priorities.BalancedResourceAllocationMap, nil, 1),
	 
		// Set this weight large enough to override all other priority functions.
		// TODO: Figure out a better way to do this, maybe at same time as fixing #24720.
		factory.RegisterPriorityFunction2("NodePreferAvoidPodsPriority", priorities.CalculateNodePreferAvoidPodsPriorityMap, nil, 10000),
	 
		// Prioritizes nodes that have labels matching NodeAffinity
		factory.RegisterPriorityFunction2("NodeAffinityPriority", priorities.CalculateNodeAffinityPriorityMap, priorities.CalculateNodeAffinityPriorityReduce, 1),
	 
		// Prioritizes nodes that marked with taint which pod can tolerate.
		factory.RegisterPriorityFunction2("TaintTolerationPriority", priorities.ComputeTaintTolerationPriorityMap, priorities.ComputeTaintTolerationPriorityReduce, 1),
	 
		// ImageLocalityPriority prioritizes nodes that have images requested by the pod present.
		factory.RegisterPriorityFunction2("ImageLocalityPriority", priorities.ImageLocalityPriorityMap, nil, 1),
	)
}
默认的预选以及优选策略：

预选：

NoVolumeZoneConflictPred MaxEBSVolumeCountPred MaxGCEPDVolumeCountPred MaxAzureDiskVolumeCountPred

MaxCSIVolumeCountPred MatchInterPodAffinityPred NoDiskConflictPred GeneralPred CheckNodeMemoryPressurePred

CheckNodeDiskPressurePred CheckNodePIDPressurePred CheckNodeConditionPred PodToleratesNodeTaintsPred

CheckVolumeBindingPred

PodFitsHostPortsPred  PodFitsResourcesPred HostNamePred MatchNodeSelectorPred

优选：

ServiceSpreadingPriority EqualPriority MostRequestedPriority RequestedToCapacityRatioPriority SelectorSpreadPriority

InterPodAffinityPriority LeastRequestedPriority BalancedResourceAllocation NodePreferAvoidPodsPriority

NodeAffinityPriority TaintTolerationPriority ImageLocalityPriority

接下来研究一下预选以及优先级函数的实现
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84982807

# Kubernetes21--kube-scheduler源码6--GenericScheduler分析

kube-scheduler调度核心函数：kubernetes/pkg/scheduler/scheduler.go

func (sched *Scheduler) scheduleOne()
为一Pod选定合适的Node节点核心函数:

host, err := sched.config.Algorithm.Schedule(pod, sched.config.NodeLister)
其中ScheudlerAlgorithm为调度算法接口

type ScheduleAlgorithm interface {
	Schedule(*v1.Pod, NodeLister) (selectedMachine string, err error)
	// Preempt receives scheduling errors for a pod and tries to create room for
	// the pod by preempting lower priority pods if possible.
	// It returns the node where preemption happened, a list of preempted pods, a
	// list of pods whose nominated node name should be removed, and error if any.
	Preempt(*v1.Pod, NodeLister, error) (selectedNode *v1.Node, preemptedPods []*v1.Pod, cleanupNominatedPods []*v1.Pod, err error)
	// Predicates() returns a pointer to a map of predicate functions. This is
	// exposed for testing.
	Predicates() map[string]FitPredicate
	// Prioritizers returns a slice of priority config. This is exposed for
	// testing.
	Prioritizers() []PriorityConfig
}
   核心为Schedule方法

Schedule(*v1.Pod, NodeLister) (selectedMachine string, err error)
在创建Scheduler.Config时，最终调用方法

func (c *configFactory) CreateFromKeys(predicateKeys, priorityKeys sets.String, extenders []algorithm.SchedulerExtender) (*Config, error)
其中定义了GenericScheduler

algo := core.NewGenericScheduler(
		c.schedulerCache,
		c.equivalencePodCache,
		c.podQueue,
		predicateFuncs,
		predicateMetaProducer,
		priorityConfigs,
		priorityMetaProducer,
		c.pluginSet,
		extenders,
		c.volumeBinder,
		c.pVCLister,
		c.pdbLister,
		c.alwaysCheckAllPredicates,
		c.disablePreemption,
		c.percentageOfNodesToScore,
	)
可知GenericScheduler为代码默认的通用调度器，研究一下。

代码位置：kubernetes/pkg/scheduler/core/generic_scheduler.go

首先研究一下NewGenericScheduler方法

// NewGenericScheduler creates a genericScheduler object.
func NewGenericScheduler(
	cache schedulerinternalcache.Cache,
	eCache *equivalence.Cache,
	podQueue internalqueue.SchedulingQueue,
	predicates map[string]algorithm.FitPredicate,
	predicateMetaProducer algorithm.PredicateMetadataProducer,
	prioritizers []algorithm.PriorityConfig,
	priorityMetaProducer algorithm.PriorityMetadataProducer,
	pluginSet pluginsv1alpha1.PluginSet,
	extenders []algorithm.SchedulerExtender,
	volumeBinder *volumebinder.VolumeBinder,
	pvcLister corelisters.PersistentVolumeClaimLister,
	pdbLister algorithm.PDBLister,
	alwaysCheckAllPredicates bool,
	disablePreemption bool,
	percentageOfNodesToScore int32,
) algorithm.ScheduleAlgorithm {
	return &genericScheduler{
		cache:                    cache,
		equivalenceCache:         eCache,
		schedulingQueue:          podQueue,
		predicates:               predicates,
		predicateMetaProducer:    predicateMetaProducer,
		prioritizers:             prioritizers,
		priorityMetaProducer:     priorityMetaProducer,
		pluginSet:                pluginSet,
		extenders:                extenders,
		cachedNodeInfoMap:        make(map[string]*schedulercache.NodeInfo),
		volumeBinder:             volumeBinder,
		pvcLister:                pvcLister,
		pdbLister:                pdbLister,
		alwaysCheckAllPredicates: alwaysCheckAllPredicates,
		disablePreemption:        disablePreemption,
		percentageOfNodesToScore: percentageOfNodesToScore,
	}
}
返回类型为genericScheduler

type genericScheduler struct {
	cache                    schedulerinternalcache.Cache
	equivalenceCache         *equivalence.Cache
	schedulingQueue          internalqueue.SchedulingQueue
	predicates               map[string]algorithm.FitPredicate
	priorityMetaProducer     algorithm.PriorityMetadataProducer
	predicateMetaProducer    algorithm.PredicateMetadataProducer
	prioritizers             []algorithm.PriorityConfig
	pluginSet                pluginsv1alpha1.PluginSet
	extenders                []algorithm.SchedulerExtender
	lastNodeIndex            uint64
	alwaysCheckAllPredicates bool
	cachedNodeInfoMap        map[string]*schedulercache.NodeInfo
	volumeBinder             *volumebinder.VolumeBinder
	pvcLister                corelisters.PersistentVolumeClaimLister
	pdbLister                algorithm.PDBLister
	disablePreemption        bool
	percentageOfNodesToScore int32
}
核心方法Schedule
// Schedule tries to schedule the given pod to one of the nodes in the node list.
// If it succeeds, it will return the name of the node.
// If it fails, it will return a FitError error with reasons.
func (g *genericScheduler) Schedule(pod *v1.Pod, nodeLister algorithm.NodeLister) (string, error) {
	trace := utiltrace.New(fmt.Sprintf("Scheduling %s/%s", pod.Namespace, pod.Name))
	defer trace.LogIfLong(100 * time.Millisecond)

	if err := podPassesBasicChecks(pod, g.pvcLister); err != nil {
		return "", err
	}
	 
	nodes, err := nodeLister.List()
	if err != nil {
		return "", err
	}
	if len(nodes) == 0 {
		return "", ErrNoNodesAvailable
	}
	 
	err = g.snapshot()
	if err != nil {
		return "", err
	}
	 
	trace.Step("Computing predicates")
	startPredicateEvalTime := time.Now()
	filteredNodes, failedPredicateMap, err := g.findNodesThatFit(pod, nodes)
	if err != nil {
		return "", err
	}
	 
	if len(filteredNodes) == 0 {
		return "", &FitError{
			Pod:              pod,
			NumAllNodes:      len(nodes),
			FailedPredicates: failedPredicateMap,
		}
	}
	metrics.SchedulingAlgorithmPredicateEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPredicateEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PredicateEvaluation).Observe(metrics.SinceInSeconds(startPredicateEvalTime))
	 
	trace.Step("Prioritizing")
	startPriorityEvalTime := time.Now()
	// When only one node after predicate, just use it.
	if len(filteredNodes) == 1 {
		metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
		return filteredNodes[0].Name, nil
	}
	 
	metaPrioritiesInterface := g.priorityMetaProducer(pod, g.cachedNodeInfoMap)
	priorityList, err := PrioritizeNodes(pod, g.cachedNodeInfoMap, metaPrioritiesInterface, g.prioritizers, filteredNodes, g.extenders)
	if err != nil {
		return "", err
	}
	metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PriorityEvaluation).Observe(metrics.SinceInSeconds(startPriorityEvalTime))
	 
	trace.Step("Selecting host")
	return g.selectHost(priorityList)
}
1.基本检查判断Pod是否可以调度

if err := podPassesBasicChecks(pod, g.pvcLister); err != nil {
		return "", err
	}
// podPassesBasicChecks makes sanity checks on the pod if it can be scheduled.
func podPassesBasicChecks(pod *v1.Pod, pvcLister corelisters.PersistentVolumeClaimLister) error {
	// Check PVCs used by the pod
	namespace := pod.Namespace
	manifest := &(pod.Spec)
	for i := range manifest.Volumes {
		volume := &manifest.Volumes[i]
		if volume.PersistentVolumeClaim == nil {
			// Volume is not a PVC, ignore
			continue
		}
		pvcName := volume.PersistentVolumeClaim.ClaimName
		pvc, err := pvcLister.PersistentVolumeClaims(namespace).Get(pvcName)
		if err != nil {
			// The error has already enough context ("persistentvolumeclaim "myclaim" not found")
			return err
		}

		if pvc.DeletionTimestamp != nil {
			return fmt.Errorf("persistentvolumeclaim %q is being deleted", pvc.Name)
		}
	}
	 
	return nil
}
2.获取Node列表并进行一系列检查

nodes, err := nodeLister.List()
	if err != nil {
		return "", err
	}
	if len(nodes) == 0 {
		return "", ErrNoNodesAvailable
	}

	err = g.snapshot()
	if err != nil {
		return "", err
	}
3.进行预选

trace.Step("Computing predicates")
	startPredicateEvalTime := time.Now()
	filteredNodes, failedPredicateMap, err := g.findNodesThatFit(pod, nodes)
	if err != nil {
		return "", err
	}

	if len(filteredNodes) == 0 {
		return "", &FitError{
			Pod:              pod,
			NumAllNodes:      len(nodes),
			FailedPredicates: failedPredicateMap,
		}
	}
	metrics.SchedulingAlgorithmPredicateEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPredicateEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PredicateEvaluation).Observe(metrics.SinceInSeconds(startPredicateEvalTime))
4.优选过程

如果预选只有一个节点则直接返回，如果有多个则进行优选过程

trace.Step("Prioritizing")
	startPriorityEvalTime := time.Now()
	// When only one node after predicate, just use it.
	if len(filteredNodes) == 1 {
		metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
		return filteredNodes[0].Name, nil
	}

	metaPrioritiesInterface := g.priorityMetaProducer(pod, g.cachedNodeInfoMap)
	priorityList, err := PrioritizeNodes(pod, g.cachedNodeInfoMap, metaPrioritiesInterface, g.prioritizers, filteredNodes, g.extenders)
	if err != nil {
		return "", err
	}
	metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PriorityEvaluation).Observe(metrics.SinceInSeconds(startPriorityEvalTime))
5.选择最后的主机Node

trace.Step("Selecting host")
	return g.selectHost(priorityList)
预选过程
通过给定的预选函数，筛选Node节点，给出符合预选条件的Node节点列表

filteredNodes, failedPredicateMap, err := g.findNodesThatFit(pod, nodes)
结果返回预选符合Node filtered列表，失败匹配的预选策略Map,如果返回预选策略为空，则直接返回所有Node列表

 获取Node列表  

nodes, err := nodeLister.List()
获取Node总数

allNodes := int32(g.cache.NodeTree().NumNodes())
		numNodesToFind := g.numFeasibleNodesToFind(allNodes)
可知这里有一个缓存用来缓存Node以及Pod信息

kubernetes/pkg/scheduler/internal/cache/interface.go
// Cache collects pods' information and provides node-level aggregated information.
// It's intended for generic scheduler to do efficient lookup.
// Cache's operations are pod centric. It does incremental updates based on pod events.
// Pod events are sent via network. We don't have guaranteed delivery of all events:
// We use Reflector to list and watch from remote.
// Reflector might be slow and do a relist, which would lead to missing events.
状态机：  Pod的状态变化如下

//   +-------------------------------------------+  +----+
//   |                            Add            |  |    |
//   |                                           |  |    | Update
//   +      Assume                Add            v  v    |
//Initial +--------> Assumed +------------+---> Added <--+
//   ^                +   +               |       +
//   |                |   |               |       |
//   |                |   |           Add |       | Remove
//   |                |   |               |       |
//   |                |   |               +       |
//   +----------------+   +-----------> Expired   +----> Deleted
//         Forget             Expire
// Note that an assumed pod can expire, because if we haven't received Add event notifying us
// for a while, there might be some problems and we shouldn't keep the pod in cache anymore.
//
// Note that "Initial", "Expired", and "Deleted" pods do not actually exist in cache.
// Based on existing use cases, we are making the following assumptions:
// - No pod would be assumed twice
// - A pod could be added without going through scheduler. In this case, we will see Add but not Assume event.
// - If a pod wasn't added, it wouldn't be removed or updated.
// - Both "Expired" and "Deleted" are valid end states. In case of some problems, e.g. network issue,
//   a pod might have changed its state (e.g. added and deleted) without delivering notification to the cache.
具体实现位置：kubernetes/pkg/scheduler/internal/cache/cache.go

NodeTree数据结构：

// NodeTree is a tree-like data structure that holds node names in each zone. Zone names are
// keys to "NodeTree.tree" and values of "NodeTree.tree" are arrays of node names.
type NodeTree struct {
	tree      map[string]*nodeArray // a map from zone (region-zone) to an array of nodes in the zone.
	zones     []string              // a list of all the zones in the tree (keys)
	zoneIndex int
	numNodes  int
	mu        sync.RWMutex
}

// nodeArray is a struct that has nodes that are in a zone.
// We use a slice (as opposed to a set/map) to store the nodes because iterating over the nodes is
// a lot more frequent than searching them by name.
type nodeArray struct {
	nodes     []string
	lastIndex int
}
获取NodeTree的所有Node节点

// NumNodes returns the number of nodes.
func (nt *NodeTree) NumNodes() int {
	nt.mu.RLock()
	defer nt.mu.RUnlock()
	return nt.numNodes
}
获取所有Node节点总数

allNodes := int32(g.cache.NodeTree().NumNodes())
两个变量

每一轮循环最小的打分Node节点数minFeasibleNodesToFind

const (
	// minFeasibleNodesToFind is the minimum number of nodes that would be scored
	// in each scheduling cycle. This is a semi-arbitrary value to ensure that a
	// certain minimum of nodes are checked for feasibility. This in turn helps
	// ensure a minimum level of spreading.
	minFeasibleNodesToFind = 100
)
percentageOfNodesToScore  最多的打分Node百分比，计算所有Node开销太大

根据以上变量计算应该查找的Node节点列表

// numFeasibleNodesToFind returns the number of feasible nodes that once found, the scheduler stops
// its search for more feasible nodes.
func (g *genericScheduler) numFeasibleNodesToFind(numAllNodes int32) int32 {
	if numAllNodes < minFeasibleNodesToFind || g.percentageOfNodesToScore <= 0 ||
		g.percentageOfNodesToScore >= 100 {
		return numAllNodes
	}
	numNodes := numAllNodes * g.percentageOfNodesToScore / 100
	if numNodes < minFeasibleNodesToFind {
		return minFeasibleNodesToFind
	}
	return numNodes
}
构建预选元数据工厂

meta := g.predicateMetaProducer(pod, g.cachedNodeInfoMap)
开始检查每一个Node节点是否符合预选策略

checkNode := func(i int) {
			var nodeCache *equivalence.NodeCache
			nodeName := g.cache.NodeTree().Next()
			if g.equivalenceCache != nil {
				nodeCache = g.equivalenceCache.LoadNodeCache(nodeName)
			}
			fits, failedPredicates, err := podFitsOnNode(
				pod,
				meta,
				g.cachedNodeInfoMap[nodeName],
				g.predicates,
				nodeCache,
				g.schedulingQueue,
				g.alwaysCheckAllPredicates,
				equivClass,
			)
			if err != nil {
				predicateResultLock.Lock()
				errs[err.Error()]++
				predicateResultLock.Unlock()
				return
			}
			if fits {
				length := atomic.AddInt32(&filteredLen, 1)
				if length > numNodesToFind {
					cancel()
					atomic.AddInt32(&filteredLen, -1)
				} else {
					filtered[length-1] = g.cachedNodeInfoMap[nodeName].Node()
				}
			} else {
				predicateResultLock.Lock()
				failedPredicateMap[nodeName] = failedPredicates
				predicateResultLock.Unlock()
			}
		}
核心方法podFitsOnNode

说明如下：

// podFitsOnNode checks whether a node given by NodeInfo satisfies the given predicate functions.
// For given pod, podFitsOnNode will check if any equivalent pod exists and try to reuse its cached
// predicate results as possible.
// This function is called from two different places: Schedule and Preempt.
// When it is called from Schedule, we want to test whether the pod is schedulable
// on the node with all the existing pods on the node plus higher and equal priority
// pods nominated to run on the node.
// When it is called from Preempt, we should remove the victims of preemption and
// add the nominated pods. Removal of the victims is done by SelectVictimsOnNode().
// It removes victims from meta and NodeInfo before calling this function.
func podFitsOnNode(
	pod *v1.Pod,
	meta algorithm.PredicateMetadata,
	info *schedulercache.NodeInfo,
	predicateFuncs map[string]algorithm.FitPredicate,
	nodeCache *equivalence.NodeCache,
	queue internalqueue.SchedulingQueue,
	alwaysCheckAllPredicates bool,
	equivClass *equivalence.Class,
) (bool, []algorithm.PredicateFailureReason, error) {
	var (
		eCacheAvailable  bool
		failedPredicates []algorithm.PredicateFailureReason
	)

	podsAdded := false
	// We run predicates twice in some cases. If the node has greater or equal priority
	// nominated pods, we run them when those pods are added to meta and nodeInfo.
	// If all predicates succeed in this pass, we run them again when these
	// nominated pods are not added. This second pass is necessary because some
	// predicates such as inter-pod affinity may not pass without the nominated pods.
	// If there are no nominated pods for the node or if the first run of the
	// predicates fail, we don't run the second pass.
	// We consider only equal or higher priority pods in the first pass, because
	// those are the current "pod" must yield to them and not take a space opened
	// for running them. It is ok if the current "pod" take resources freed for
	// lower priority pods.
	// Requiring that the new pod is schedulable in both circumstances ensures that
	// we are making a conservative decision: predicates like resources and inter-pod
	// anti-affinity are more likely to fail when the nominated pods are treated
	// as running, while predicates like pod affinity are more likely to fail when
	// the nominated pods are treated as not running. We can't just assume the
	// nominated pods are running because they are not running right now and in fact,
	// they may end up getting scheduled to a different node.
	for i := 0; i < 2; i++ {
		metaToUse := meta
		nodeInfoToUse := info
		if i == 0 {
			podsAdded, metaToUse, nodeInfoToUse = addNominatedPods(pod, meta, info, queue)
		} else if !podsAdded || len(failedPredicates) != 0 {
			break
		}
		// Bypass eCache if node has any nominated pods.
		// TODO(bsalamat): consider using eCache and adding proper eCache invalidations
		// when pods are nominated or their nominations change.
		eCacheAvailable = equivClass != nil && nodeCache != nil && !podsAdded
		for predicateID, predicateKey := range predicates.Ordering() {
			var (
				fit     bool
				reasons []algorithm.PredicateFailureReason
				err     error
			)
			//TODO (yastij) : compute average predicate restrictiveness to export it as Prometheus metric
			if predicate, exist := predicateFuncs[predicateKey]; exist {
				if eCacheAvailable {
					fit, reasons, err = nodeCache.RunPredicate(predicate, predicateKey, predicateID, pod, metaToUse, nodeInfoToUse, equivClass)
				} else {
					fit, reasons, err = predicate(pod, metaToUse, nodeInfoToUse)
				}
				if err != nil {
					return false, []algorithm.PredicateFailureReason{}, err
				}
	 
				if !fit {
					// eCache is available and valid, and predicates result is unfit, record the fail reasons
					failedPredicates = append(failedPredicates, reasons...)
					// if alwaysCheckAllPredicates is false, short circuit all predicates when one predicate fails.
					if !alwaysCheckAllPredicates {
						klog.V(5).Infoln("since alwaysCheckAllPredicates has not been set, the predicate " +
							"evaluation is short circuited and there are chances " +
							"of other predicates failing as well.")
						break
					}
				}
			}
		}
	}
	 
	return len(failedPredicates) == 0, failedPredicates, nil
}
该预选过程要执行两次，第一次当选定Node时，该Node节点可能有更高优先级的Pod待调度，若这些Pod调度成功，则根据一些亲和反亲和策略等该Pod可能已经不适合该Node节点，因此需要再一次预选

缓存预选结果，下一次执行避免重复计算

fit, reasons, err = nodeCache.RunPredicate(predicate, predicateKey, predicateID, pod, metaToUse, nodeInfoToUse, equivClass)
预选匹配

fit, reasons, err = predicate(pod, metaToUse, nodeInfoToUse)
predicate具体实现在kubernetes/pkg/scheduler/algorithm/predicates/predicates.go

优选过程
metaPrioritiesInterface := g.priorityMetaProducer(pod, g.cachedNodeInfoMap)
	priorityList, err := PrioritizeNodes(pod, g.cachedNodeInfoMap, metaPrioritiesInterface, g.prioritizers, filteredNodes, g.extenders)
	if err != nil {
		return "", err
	}
	metrics.SchedulingAlgorithmPriorityEvaluationDuration.Observe(metrics.SinceInMicroseconds(startPriorityEvalTime))
	metrics.SchedulingLatency.WithLabelValues(metrics.PriorityEvaluation).Observe(metrics.SinceInSeconds(startPriorityEvalTime))
// PrioritizeNodes prioritizes the nodes by running the individual priority functions in parallel.
// Each priority function is expected to set a score of 0-10
// 0 is the lowest priority score (least preferred node) and 10 is the highest
// Each priority function can also have its own weight
// The node scores returned by the priority function are multiplied by the weights to get weighted scores
// All scores are finally combined (added) to get the total weighted scores of all nodes
每一个优先级函数0---10，以及响应权重，最终返回权重加权和，每一个优先级函数并行计算

func PrioritizeNodes(
	pod *v1.Pod,
	nodeNameToInfo map[string]*schedulercache.NodeInfo,
	meta interface{},
	priorityConfigs []algorithm.PriorityConfig,
	nodes []*v1.Node,
	extenders []algorithm.SchedulerExtender,
) (schedulerapi.HostPriorityList, error) {
	// If no priority configs are provided, then the EqualPriority function is applied
	// This is required to generate the priority list in the required format
	if len(priorityConfigs) == 0 && len(extenders) == 0 {
		result := make(schedulerapi.HostPriorityList, 0, len(nodes))
		for i := range nodes {
			hostPriority, err := EqualPriorityMap(pod, meta, nodeNameToInfo[nodes[i].Name])
			if err != nil {
				return nil, err
			}
			result = append(result, hostPriority)
		}
		return result, nil
	}

	var (
		mu   = sync.Mutex{}
		wg   = sync.WaitGroup{}
		errs []error
	)
	appendError := func(err error) {
		mu.Lock()
		defer mu.Unlock()
		errs = append(errs, err)
	}
	 
	results := make([]schedulerapi.HostPriorityList, len(priorityConfigs), len(priorityConfigs))
	 
	// DEPRECATED: we can remove this when all priorityConfigs implement the
	// Map-Reduce pattern.
	for i := range priorityConfigs {
		if priorityConfigs[i].Function != nil {
			wg.Add(1)
			go func(index int) {
				defer wg.Done()
				var err error
				results[index], err = priorityConfigs[index].Function(pod, nodeNameToInfo, nodes)
				if err != nil {
					appendError(err)
				}
			}(i)
		} else {
			results[i] = make(schedulerapi.HostPriorityList, len(nodes))
		}
	}
	 
	workqueue.ParallelizeUntil(context.TODO(), 16, len(nodes), func(index int) {
		nodeInfo := nodeNameToInfo[nodes[index].Name]
		for i := range priorityConfigs {
			if priorityConfigs[i].Function != nil {
				continue
			}
	 
			var err error
			results[i][index], err = priorityConfigs[i].Map(pod, meta, nodeInfo)
			if err != nil {
				appendError(err)
				results[i][index].Host = nodes[index].Name
			}
		}
	})
	 
	for i := range priorityConfigs {
		if priorityConfigs[i].Reduce == nil {
			continue
		}
		wg.Add(1)
		go func(index int) {
			defer wg.Done()
			if err := priorityConfigs[index].Reduce(pod, meta, nodeNameToInfo, results[index]); err != nil {
				appendError(err)
			}
			if klog.V(10) {
				for _, hostPriority := range results[index] {
					klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), hostPriority.Host, priorityConfigs[index].Name, hostPriority.Score)
				}
			}
		}(i)
	}
	// Wait for all computations to be finished.
	wg.Wait()
	if len(errs) != 0 {
		return schedulerapi.HostPriorityList{}, errors.NewAggregate(errs)
	}
	 
	// Summarize all scores.
	result := make(schedulerapi.HostPriorityList, 0, len(nodes))
	 
	for i := range nodes {
		result = append(result, schedulerapi.HostPriority{Host: nodes[i].Name, Score: 0})
		for j := range priorityConfigs {
			result[i].Score += results[j][i].Score * priorityConfigs[j].Weight
		}
	}
	 
	if len(extenders) != 0 && nodes != nil {
		combinedScores := make(map[string]int, len(nodeNameToInfo))
		for i := range extenders {
			if !extenders[i].IsInterested(pod) {
				continue
			}
			wg.Add(1)
			go func(extIndex int) {
				defer wg.Done()
				prioritizedList, weight, err := extenders[extIndex].Prioritize(pod, nodes)
				if err != nil {
					// Prioritization errors from extender can be ignored, let k8s/other extenders determine the priorities
					return
				}
				mu.Lock()
				for i := range *prioritizedList {
					host, score := (*prioritizedList)[i].Host, (*prioritizedList)[i].Score
					if klog.V(10) {
						klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), host, extenders[extIndex].Name(), score)
					}
					combinedScores[host] += score * weight
				}
				mu.Unlock()
			}(i)
		}
		// wait for all go routines to finish
		wg.Wait()
		for i := range result {
			result[i].Score += combinedScores[result[i].Host]
		}
	}
	 
	if klog.V(10) {
		for i := range result {
			klog.Infof("Host %s => Score %d", result[i].Host, result[i].Score)
		}
	}
	return result, nil
}
如果没有优先级函数使用均衡权重优先级函数EqualPriority function

// If no priority configs are provided, then the EqualPriority function is applied
	// This is required to generate the priority list in the required format
	if len(priorityConfigs) == 0 && len(extenders) == 0 {
		result := make(schedulerapi.HostPriorityList, 0, len(nodes))
		for i := range nodes {
			hostPriority, err := EqualPriorityMap(pod, meta, nodeNameToInfo[nodes[i].Name])
			if err != nil {
				return nil, err
			}
			result = append(result, hostPriority)
		}
		return result, nil
	}
// EqualPriorityMap is a prioritizer function that gives an equal weight of one to all nodes
func EqualPriorityMap(_ *v1.Pod, _ interface{}, nodeInfo *schedulercache.NodeInfo) (schedulerapi.HostPriority, error) {
	node := nodeInfo.Node()
	if node == nil {
		return schedulerapi.HostPriority{}, fmt.Errorf("node not found")
	}
	return schedulerapi.HostPriority{
		Host:  node.Name,
		Score: 1,
	}, nil
}
初始版本循环计算优先级函数

// DEPRECATED: we can remove this when all priorityConfigs implement the
	// Map-Reduce pattern.
		for i := range priorityConfigs {
		if priorityConfigs[i].Function != nil {
			wg.Add(1)
			go func(index int) {
				defer wg.Done()
				var err error
				results[index], err = priorityConfigs[index].Function(pod, nodeNameToInfo, nodes)
				if err != nil {
					appendError(err)
			}
			}(i)
		} else {
			results[i] = make(schedulerapi.HostPriorityList, len(nodes))
		}
	}
以后会修改为map-reduce版本

map过程

workqueue.ParallelizeUntil(context.TODO(), 16, len(nodes), func(index int) {
		nodeInfo := nodeNameToInfo[nodes[index].Name]
		for i := range priorityConfigs {
			if priorityConfigs[i].Function != nil {
				continue
			}

			var err error
			results[i][index], err = priorityConfigs[i].Map(pod, meta, nodeInfo)
			if err != nil {
				appendError(err)
				results[i][index].Host = nodes[index].Name
			}
		}
	})
// PriorityMapFunction is a function that computes per-node results for a given node.
// TODO: Figure out the exact API of this method.
// TODO: Change interface{} to a specific type.
type PriorityMapFunction func(pod *v1.Pod, meta interface{}, nodeInfo *schedulercache.NodeInfo) (schedulerapi.HostPriority, error)
reduce过程

for i := range priorityConfigs {
		if priorityConfigs[i].Reduce == nil {
			continue
		}
		wg.Add(1)
		go func(index int) {
			defer wg.Done()
			if err := priorityConfigs[index].Reduce(pod, meta, nodeNameToInfo, results[index]); err != nil {
				appendError(err)
			}
			if klog.V(10) {
				for _, hostPriority := range results[index] {
					klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), hostPriority.Host, priorityConfigs[index].Name, hostPriority.Score)
				}
			}
		}(i)
	}
// PriorityReduceFunction is a function that aggregated per-node results and computes
 final scores for all nodes.
// TODO: Figure out the exact API of this method.
// TODO: Change interface{} to a specific type.
type PriorityReduceFunction func(pod *v1.Pod, meta interface{}, nodeNameToInfo map[string]*schedulercache.NodeInfo, result schedulerapi.HostPriorityList) error
计算每个节点的总分数

// Summarize all scores.
	result := make(schedulerapi.HostPriorityList, 0, len(nodes))

	for i := range nodes {
		result = append(result, schedulerapi.HostPriority{Host: nodes[i].Name, Score: 0})
		for j := range priorityConfigs {
			result[i].Score += results[j][i].Score * priorityConfigs[j].Weight
		}
	}
// PriorityConfig is a config used for a priority function.
type PriorityConfig struct {
	Name   string
	Map    PriorityMapFunction
	Reduce PriorityReduceFunction
	// TODO: Remove it after migrating all functions to
	// Map-Reduce pattern.
	Function PriorityFunction
	Weight   int
}
SchedulerExtender对于优先级分数的扩展

if len(extenders) != 0 && nodes != nil {
		combinedScores := make(map[string]int, len(nodeNameToInfo))
		for i := range extenders {
			if !extenders[i].IsInterested(pod) {
				continue
			}
			wg.Add(1)
			go func(extIndex int) {
				defer wg.Done()
				prioritizedList, weight, err := extenders[extIndex].Prioritize(pod, nodes)
				if err != nil {
					// Prioritization errors from extender can be ignored, let k8s/other extenders determine the priorities
					return
				}
				mu.Lock()
				for i := range *prioritizedList {
					host, score := (*prioritizedList)[i].Host, (*prioritizedList)[i].Score
					if klog.V(10) {
						klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), host, extenders[extIndex].Name(), score)
					}
					combinedScores[host] += score * weight
				}
				mu.Unlock()
			}(i)
		}
		// wait for all go routines to finish
		wg.Wait()
		for i := range result {
			result[i].Score += combinedScores[result[i].Host]
		}
	}
选择最高的分数Node节点

// selectHost takes a prioritized list of nodes and then picks one
// in a round-robin manner from the nodes that had the highest score.
func (g *genericScheduler) selectHost(priorityList schedulerapi.HostPriorityList) (string, error) {
	if len(priorityList) == 0 {
		return "", fmt.Errorf("empty priorityList")
	}

	maxScores := findMaxScores(priorityList)
	ix := int(g.lastNodeIndex % uint64(len(maxScores)))
	g.lastNodeIndex++
	 
	return priorityList[maxScores[ix]].Host, nil
}
// findMaxScores returns the indexes of nodes in the "priorityList" that has the highest "Score".
func findMaxScores(priorityList schedulerapi.HostPriorityList) []int {
	maxScoreIndexes := make([]int, 0, len(priorityList)/2)
	maxScore := priorityList[0].Score
	for i, hp := range priorityList {
		if hp.Score > maxScore {
			maxScore = hp.Score
			maxScoreIndexes = maxScoreIndexes[:0]
			maxScoreIndexes = append(maxScoreIndexes, i)
		} else if hp.Score == maxScore {
			maxScoreIndexes = append(maxScoreIndexes, i)
		}
	}
	return maxScoreIndexes
}
总结：
schedule方法执行流程

func (g *genericScheduler) Schedule(pod *v1.Pod, nodeLister algorithm.NodeLister) (string, error)

基本检查判断是否该Pod可以被调度

podPassesBasicChecks(pod, g.pvcLister)

获取Node列表

nodes, err := nodeLister.List()

预选

filteredNodes, failedPredicateMap, err := g.findNodesThatFit(pod, nodes)

优选

priorityList, err := PrioritizeNodes(pod, g.cachedNodeInfoMap, metaPrioritiesInterface, g.prioritizers, filteredNodes, g.extenders)

选择得分最高的Node节点

g.selectHost(priorityList)
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84986854

# Kubernetes22--kube-scheduler源码7--预选过程分析

kubernetes/pkg/scheduler/core/generic_scheduler.go

从一系列Node列表中为某一Pod选择最合适的Node节点

func (g *genericScheduler) Schedule(pod *v1.Pod, nodeLister algorithm.NodeLister) (string, error)
预选过程入口

func (g *genericScheduler) findNodesThatFit(pod *v1.Pod, nodes []*v1.Node) ([]*v1.Node, FailedPredicateMap, error)
在一系列预选策略下判断某一Node是否满足该Pod对象预选条件

func podFitsOnNode(
	pod *v1.Pod,
	meta algorithm.PredicateMetadata,
	info *schedulercache.NodeInfo,
	predicateFuncs map[string]algorithm.FitPredicate,
	nodeCache *equivalence.NodeCache,
	queue internalqueue.SchedulingQueue,
	alwaysCheckAllPredicates bool,
	equivClass *equivalence.Class,
) (bool, []algorithm.PredicateFailureReason, error)
接下来调用

type FitPredicate func(pod *v1.Pod, meta PredicateMetadata, nodeInfo *schedulercache.NodeInfo) (bool, []PredicateFailureReason, error)
FitPredicate具体实现： kubernetes/pkg/scheduler/algorithm/predicates/predicates.go

默认支持预选策略

var (
	predicatesOrdering = []string{CheckNodeConditionPred, CheckNodeUnschedulablePred,
		GeneralPred, HostNamePred, PodFitsHostPortsPred,
		MatchNodeSelectorPred, PodFitsResourcesPred, NoDiskConflictPred,
		PodToleratesNodeTaintsPred, PodToleratesNodeNoExecuteTaintsPred, CheckNodeLabelPresencePred,
		CheckServiceAffinityPred, MaxEBSVolumeCountPred, MaxGCEPDVolumeCountPred, MaxCSIVolumeCountPred,
		MaxAzureDiskVolumeCountPred, CheckVolumeBindingPred, NoVolumeZoneConflictPred,
		CheckNodeMemoryPressurePred, CheckNodePIDPressurePred, CheckNodeDiskPressurePred, MatchInterPodAffinityPred}
)
defaultprovider初始化时定义了四种预选策略

// Fit is defined based on the absence of port conflicts.
	// This predicate is actually a default predicate, because it is invoked from
	// predicates.GeneralPredicates()
	factory.RegisterFitPredicate(predicates.PodFitsHostPortsPred, predicates.PodFitsHostPorts)
	// Fit is determined by resource availability.
	// This predicate is actually a default predicate, because it is invoked from
	// predicates.GeneralPredicates()
	factory.RegisterFitPredicate(predicates.PodFitsResourcesPred, predicates.PodFitsResources)
	// Fit is determined by the presence of the Host parameter and a string match
	// This predicate is actually a default predicate, because it is invoked from
	// predicates.GeneralPredicates()
	factory.RegisterFitPredicate(predicates.HostNamePred, predicates.PodFitsHost)
	// Fit is determined by node selector query.
	factory.RegisterFitPredicate(predicates.MatchNodeSelectorPred, predicates.PodMatchNodeSelector)
端口是否冲突PodFitsHostPorts

// PodFitsHostPorts checks if a node has free ports for the requested pod ports.
func PodFitsHostPorts(pod *v1.Pod, meta algorithm.PredicateMetadata, nodeInfo *schedulercache.NodeInfo) (bool, []algorithm.PredicateFailureReason, error) {
	var wantPorts []*v1.ContainerPort
	if predicateMeta, ok := meta.(*predicateMetadata); ok {
		wantPorts = predicateMeta.podPorts
	} else {
		// We couldn't parse metadata - fallback to computing it.
		wantPorts = schedutil.GetContainerPorts(pod)
	}
	if len(wantPorts) == 0 {
		return true, nil, nil
	}

	existingPorts := nodeInfo.UsedPorts()
	 
	// try to see whether existingPorts and  wantPorts will conflict or not
	if portsConflict(existingPorts, wantPorts) {
		return false, []algorithm.PredicateFailureReason{ErrPodNotFitsHostPorts}, nil
	}
	 
	return true, nil, nil
}
// GetContainerPorts returns the used host ports of Pods: if 'port' was used, a 'port:true' pair
// will be in the result; but it does not resolve port conflict.
func GetContainerPorts(pods ...*v1.Pod) []*v1.ContainerPort {
	var ports []*v1.ContainerPort
	for _, pod := range pods {
		for j := range pod.Spec.Containers {
			container := &pod.Spec.Containers[j]
			for k := range container.Ports {
				ports = append(ports, &container.Ports[k])
			}
		}
	}
	return ports
}
资源是否满足PodFitsResources

// PodFitsResources checks if a node has sufficient resources, such as cpu, memory, gpu, opaque int resources etc to run a pod.
// First return value indicates whether a node has sufficient resources to run a pod while the second return value indicates the
// predicate failure reasons if the node has insufficient resources to run the pod.
func PodFitsResources(pod *v1.Pod, meta algorithm.PredicateMetadata, nodeInfo *schedulercache.NodeInfo) (bool, []algorithm.PredicateFailureReason, error) {
	node := nodeInfo.Node()
	if node == nil {
		return false, nil, fmt.Errorf("node not found")
	}

	var predicateFails []algorithm.PredicateFailureReason
	allowedPodNumber := nodeInfo.AllowedPodNumber()
	if len(nodeInfo.Pods())+1 > allowedPodNumber {
		predicateFails = append(predicateFails, NewInsufficientResourceError(v1.ResourcePods, 1, int64(len(nodeInfo.Pods())), int64(allowedPodNumber)))
	}
	 
	// No extended resources should be ignored by default.
	ignoredExtendedResources := sets.NewString()
	 
	var podRequest *schedulercache.Resource
	if predicateMeta, ok := meta.(*predicateMetadata); ok {
		podRequest = predicateMeta.podRequest
		if predicateMeta.ignoredExtendedResources != nil {
			ignoredExtendedResources = predicateMeta.ignoredExtendedResources
		}
	} else {
		// We couldn't parse metadata - fallback to computing it.
		podRequest = GetResourceRequest(pod)
	}
	if podRequest.MilliCPU == 0 &&
		podRequest.Memory == 0 &&
		podRequest.EphemeralStorage == 0 &&
		len(podRequest.ScalarResources) == 0 {
		return len(predicateFails) == 0, predicateFails, nil
	}
	 
	allocatable := nodeInfo.AllocatableResource()
	if allocatable.MilliCPU < podRequest.MilliCPU+nodeInfo.RequestedResource().MilliCPU {
		predicateFails = append(predicateFails, NewInsufficientResourceError(v1.ResourceCPU, podRequest.MilliCPU, nodeInfo.RequestedResource().MilliCPU, allocatable.MilliCPU))
	}
	if allocatable.Memory < podRequest.Memory+nodeInfo.RequestedResource().Memory {
		predicateFails = append(predicateFails, NewInsufficientResourceError(v1.ResourceMemory, podRequest.Memory, nodeInfo.RequestedResource().Memory, allocatable.Memory))
	}
	if allocatable.EphemeralStorage < podRequest.EphemeralStorage+nodeInfo.RequestedResource().EphemeralStorage {
		predicateFails = append(predicateFails, NewInsufficientResourceError(v1.ResourceEphemeralStorage, podRequest.EphemeralStorage, nodeInfo.RequestedResource().EphemeralStorage, allocatable.EphemeralStorage))
	}
	 
	for rName, rQuant := range podRequest.ScalarResources {
		if v1helper.IsExtendedResourceName(rName) {
			// If this resource is one of the extended resources that should be
			// ignored, we will skip checking it.
			if ignoredExtendedResources.Has(string(rName)) {
				continue
			}
		}
		if allocatable.ScalarResources[rName] < rQuant+nodeInfo.RequestedResource().ScalarResources[rName] {
			predicateFails = append(predicateFails, NewInsufficientResourceError(rName, podRequest.ScalarResources[rName], nodeInfo.RequestedResource().ScalarResources[rName], allocatable.ScalarResources[rName]))
		}
	}
	 
	if klog.V(10) {
		if len(predicateFails) == 0 {
			// We explicitly don't do klog.V(10).Infof() to avoid computing all the parameters if this is
			// not logged. There is visible performance gain from it.
			klog.Infof("Schedule Pod %+v on Node %+v is allowed, Node is running only %v out of %v Pods.",
				podName(pod), node.Name, len(nodeInfo.Pods()), allowedPodNumber)
		}
	}
	return len(predicateFails) == 0, predicateFails, nil
}
主机名是否符合PodFitsHost

// PodFitsHost checks if a pod spec node name matches the current node.
func PodFitsHost(pod *v1.Pod, meta algorithm.PredicateMetadata, nodeInfo *schedulercache.NodeInfo) (bool, []algorithm.PredicateFailureReason, error) {
	if len(pod.Spec.NodeName) == 0 {
		return true, nil, nil
	}
	node := nodeInfo.Node()
	if node == nil {
		return false, nil, fmt.Errorf("node not found")
	}
	if pod.Spec.NodeName == node.Name {
		return true, nil, nil
	}
	return false, []algorithm.PredicateFailureReason{ErrPodNotMatchHostName}, nil
}
nodeSelector标签是否符合

// PodMatchNodeSelector checks if a pod node selector matches the node label.
func PodMatchNodeSelector(pod *v1.Pod, meta algorithm.PredicateMetadata, nodeInfo *schedulercache.NodeInfo) (bool, []algorithm.PredicateFailureReason, error) {
	node := nodeInfo.Node()
	if node == nil {
		return false, nil, fmt.Errorf("node not found")
	}
	if podMatchesNodeSelectorAndAffinityTerms(pod, node) {
		return true, nil, nil
	}
	return false, []algorithm.PredicateFailureReason{ErrNodeSelectorNotMatch}, nil
}
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84990159

# Kubernetes23--kube-scheduler源码8--优选过程分析

kubernetes/pkg/scheduler/core/generic_scheduler.go

优选过程分析
优选函数入口

priorityList, err := PrioritizeNodes(pod, g.cachedNodeInfoMap, metaPrioritiesInterface, g.prioritizers, filteredNodes, g.extenders)
函数定义

func PrioritizeNodes(
	pod *v1.Pod,
	nodeNameToInfo map[string]*schedulercache.NodeInfo,
	meta interface{},
	priorityConfigs []algorithm.PriorityConfig,
	nodes []*v1.Node,
	extenders []algorithm.SchedulerExtender,
) (schedulerapi.HostPriorityList, error)
函数说明

// PrioritizeNodes prioritizes the nodes by running the individual priority functions in parallel.
// Each priority function is expected to set a score of 0-10
// 0 is the lowest priority score (least preferred node) and 10 is the highest
// Each priority function can also have its own weight
// The node scores returned by the priority function are multiplied by the weights to get weighted scores
// All scores are finally combined (added) to get the total weighted scores of all nodes
每一个优先级函数返回0--10，并且有相应权重，优先级函数并行计算，最终使用加权和作为最终Node节点得分数，返回优先级Node列表

func PrioritizeNodes(
	pod *v1.Pod,
	nodeNameToInfo map[string]*schedulercache.NodeInfo,
	meta interface{},
	priorityConfigs []algorithm.PriorityConfig,
	nodes []*v1.Node,
	extenders []algorithm.SchedulerExtender,
) (schedulerapi.HostPriorityList, error) {
	// If no priority configs are provided, then the EqualPriority function is applied
	// This is required to generate the priority list in the required format
	if len(priorityConfigs) == 0 && len(extenders) == 0 {
		result := make(schedulerapi.HostPriorityList, 0, len(nodes))
		for i := range nodes {
			hostPriority, err := EqualPriorityMap(pod, meta, nodeNameToInfo[nodes[i].Name])
			if err != nil {
				return nil, err
			}
			result = append(result, hostPriority)
		}
		return result, nil
	}

	var (
		mu   = sync.Mutex{}
		wg   = sync.WaitGroup{}
		errs []error
	)
	appendError := func(err error) {
		mu.Lock()
		defer mu.Unlock()
		errs = append(errs, err)
	}
	 
	results := make([]schedulerapi.HostPriorityList, len(priorityConfigs), len(priorityConfigs))
	 
	// DEPRECATED: we can remove this when all priorityConfigs implement the
	// Map-Reduce pattern.
		for i := range priorityConfigs {
		if priorityConfigs[i].Function != nil {
			wg.Add(1)
			go func(index int) {
				defer wg.Done()
				var err error
				results[index], err = priorityConfigs[index].Function(pod, nodeNameToInfo, nodes)
				if err != nil {
					appendError(err)
			}
			}(i)
		} else {
			results[i] = make(schedulerapi.HostPriorityList, len(nodes))
		}
	}
	 
	workqueue.ParallelizeUntil(context.TODO(), 16, len(nodes), func(index int) {
		nodeInfo := nodeNameToInfo[nodes[index].Name]
		for i := range priorityConfigs {
			if priorityConfigs[i].Function != nil {
				continue
			}
	 
			var err error
			results[i][index], err = priorityConfigs[i].Map(pod, meta, nodeInfo)
			if err != nil {
				appendError(err)
				results[i][index].Host = nodes[index].Name
			}
		}
	})
	 
	for i := range priorityConfigs {
		if priorityConfigs[i].Reduce == nil {
			continue
		}
		wg.Add(1)
		go func(index int) {
			defer wg.Done()
			if err := priorityConfigs[index].Reduce(pod, meta, nodeNameToInfo, results[index]); err != nil {
				appendError(err)
			}
			if klog.V(10) {
				for _, hostPriority := range results[index] {
					klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), hostPriority.Host, priorityConfigs[index].Name, hostPriority.Score)
				}
			}
		}(i)
	}
	// Wait for all computations to be finished.
	wg.Wait()
	if len(errs) != 0 {
		return schedulerapi.HostPriorityList{}, errors.NewAggregate(errs)
	}
	 
	// Summarize all scores.
	result := make(schedulerapi.HostPriorityList, 0, len(nodes))
	 
	for i := range nodes {
		result = append(result, schedulerapi.HostPriority{Host: nodes[i].Name, Score: 0})
		for j := range priorityConfigs {
			result[i].Score += results[j][i].Score * priorityConfigs[j].Weight
		}
	}
	 
	if len(extenders) != 0 && nodes != nil {
		combinedScores := make(map[string]int, len(nodeNameToInfo))
		for i := range extenders {
			if !extenders[i].IsInterested(pod) {
				continue
			}
			wg.Add(1)
			go func(extIndex int) {
				defer wg.Done()
				prioritizedList, weight, err := extenders[extIndex].Prioritize(pod, nodes)
				if err != nil {
					// Prioritization errors from extender can be ignored, let k8s/other extenders determine the priorities
					return
				}
				mu.Lock()
				for i := range *prioritizedList {
					host, score := (*prioritizedList)[i].Host, (*prioritizedList)[i].Score
					if klog.V(10) {
						klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), host, extenders[extIndex].Name(), score)
					}
					combinedScores[host] += score * weight
				}
				mu.Unlock()
			}(i)
		}
		// wait for all go routines to finish
		wg.Wait()
		for i := range result {
			result[i].Score += combinedScores[result[i].Host]
		}
	}
	 
	if klog.V(10) {
		for i := range result {
			klog.Infof("Host %s => Score %d", result[i].Host, result[i].Score)
		}
	}
	return result, nil
}
如果没有优先级配置，则执行相等权重函数来处理

// If no priority configs are provided, then the EqualPriority function is applied
	// This is required to generate the priority list in the required format
	if len(priorityConfigs) == 0 && len(extenders) == 0 {
		result := make(schedulerapi.HostPriorityList, 0, len(nodes))
		for i := range nodes {
			hostPriority, err := EqualPriorityMap(pod, meta, nodeNameToInfo[nodes[i].Name])
			if err != nil {
				return nil, err
			}
			result = append(result, hostPriority)
		}
		return result, nil
	}
// EqualPriorityMap is a prioritizer function that gives an equal weight of one to all nodes
func EqualPriorityMap(_ *v1.Pod, _ interface{}, nodeInfo *schedulercache.NodeInfo) (schedulerapi.HostPriority, error) {
	node := nodeInfo.Node()
	if node == nil {
		return schedulerapi.HostPriority{}, fmt.Errorf("node not found")
	}
	return schedulerapi.HostPriority{
		Host:  node.Name,
		Score: 1,
	}, nil
}
可知最终返回的HostPriority得分默认均为1

计算所有Node节点在该优先级函数的得分，旧版本使用普通循环方式加异步执行

// DEPRECATED: we can remove this when all priorityConfigs implement the
	// Map-Reduce pattern.
		for i := range priorityConfigs {
		if priorityConfigs[i].Function != nil {
			wg.Add(1)
			go func(index int) {
				defer wg.Done()
				var err error
				results[index], err = priorityConfigs[index].Function(pod, nodeNameToInfo, nodes)
				if err != nil {
					appendError(err)
			}
			}(i)
		} else {
			results[i] = make(schedulerapi.HostPriorityList, len(nodes))
		}
	}
// PriorityFunction is a function that computes scores for all nodes.
// DEPRECATED
// Use Map-Reduce pattern for priority functions.
type PriorityFunction func(pod *v1.Pod, nodeNameToInfo map[string]*schedulercache.NodeInfo, nodes []*v1.Node) (schedulerapi.HostPriorityList, error)
新版本使用Map-Reduce模式来实现  map过程启动16个进程

workqueue.ParallelizeUntil(context.TODO(), 16, len(nodes), func(index int) {
		nodeInfo := nodeNameToInfo[nodes[index].Name]
		for i := range priorityConfigs {
			if priorityConfigs[i].Function != nil {
				continue
			}

			var err error
			results[i][index], err = priorityConfigs[i].Map(pod, meta, nodeInfo)
			if err != nil {
				appendError(err)
				results[i][index].Host = nodes[index].Name
			}
		}
	})
	 
	for i := range priorityConfigs {
		if priorityConfigs[i].Reduce == nil {
			continue
		}
		wg.Add(1)
		go func(index int) {
			defer wg.Done()
			if err := priorityConfigs[index].Reduce(pod, meta, nodeNameToInfo, results[index]); err != nil {
				appendError(err)
			}
			if klog.V(10) {
				for _, hostPriority := range results[index] {
					klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), hostPriority.Host, priorityConfigs[index].Name, hostPriority.Score)
				}
			}
		}(i)
	}
map以及reduce函数定义如下

// PriorityMapFunction is a function that computes per-node results for a given node.
// TODO: Figure out the exact API of this method.
// TODO: Change interface{} to a specific type.
type PriorityMapFunction func(pod *v1.Pod, meta interface{}, nodeInfo *schedulercache.NodeInfo) (schedulerapi.HostPriority, error)

// PriorityReduceFunction is a function that aggregated per-node results and computes
 final scores for all nodes.
// TODO: Figure out the exact API of this method.
// TODO: Change interface{} to a specific type.
type PriorityReduceFunction func(pod *v1.Pod, meta interface{}, nodeNameToInfo map[string]*schedulercache.NodeInfo, result schedulerapi.HostPriorityList) error
等到计算结束，汇总计算每一个Node的总得分

// Wait for all computations to be finished.
	wg.Wait()
	if len(errs) != 0 {
		return schedulerapi.HostPriorityList{}, errors.NewAggregate(errs)
	}

	// Summarize all scores.
	result := make(schedulerapi.HostPriorityList, 0, len(nodes))
	 
	for i := range nodes {
		result = append(result, schedulerapi.HostPriority{Host: nodes[i].Name, Score: 0})
		for j := range priorityConfigs {
			result[i].Score += results[j][i].Score * priorityConfigs[j].Weight
		}
	}
对于算法的扩展功能计算实现SchedulerExtender

if len(extenders) != 0 && nodes != nil {
		combinedScores := make(map[string]int, len(nodeNameToInfo))
		for i := range extenders {
			if !extenders[i].IsInterested(pod) {
				continue
			}
			wg.Add(1)
			go func(extIndex int) {
				defer wg.Done()
				prioritizedList, weight, err := extenders[extIndex].Prioritize(pod, nodes)
				if err != nil {
					// Prioritization errors from extender can be ignored, let k8s/other extenders determine the priorities
					return
				}
				mu.Lock()
				for i := range *prioritizedList {
					host, score := (*prioritizedList)[i].Host, (*prioritizedList)[i].Score
					if klog.V(10) {
						klog.Infof("%v -> %v: %v, Score: (%d)", util.GetPodFullName(pod), host, extenders[extIndex].Name(), score)
					}
					combinedScores[host] += score * weight
				}
				mu.Unlock()
			}(i)
		}
		// wait for all go routines to finish
		wg.Wait()
		for i := range result {
			result[i].Score += combinedScores[result[i].Host]
		}
	}
SchedulerExtender接口以后需要研究一下

优先级函数实现分析
优先级函数实现位置  kubernetes/pkg/scheduler/algorithm/priorities

1.BalancedResourceAllocation   资源均衡利用  计算cpu，mem, volumn利用率的方差

// BalancedResourceAllocationMap favors nodes with balanced resource usage rate.
// BalancedResourceAllocationMap should **NOT** be used alone, and **MUST** be used together
// with LeastRequestedPriority. It calculates the difference between the cpu and memory fraction
// of capacity, and prioritizes the host based on how close the two metrics are to each other.
balancedResourcePriority = &ResourceAllocationPriority{"BalancedResourceAllocation", balancedResourceScorer}
具体实现

func balancedResourceScorer(requested, allocable *schedulercache.Resource, includeVolumes bool, requestedVolumes int, allocatableVolumes int) int64 {
	cpuFraction := fractionOfCapacity(requested.MilliCPU, allocable.MilliCPU)
	memoryFraction := fractionOfCapacity(requested.Memory, allocable.Memory)
	// This to find a node which has most balanced CPU, memory and volume usage.
	if includeVolumes && utilfeature.DefaultFeatureGate.Enabled(features.BalanceAttachedNodeVolumes) && allocatableVolumes > 0 {
		volumeFraction := float64(requestedVolumes) / float64(allocatableVolumes)
			if cpuFraction >= 1 || memoryFraction >= 1 || volumeFraction >= 1 {
			// if requested >= capacity, the corresponding host should never be preferred.
			return 0
		}
		// Compute variance for all the three fractions.
		mean := (cpuFraction + memoryFraction + volumeFraction) / float64(3)
		variance := float64((((cpuFraction - mean) * (cpuFraction - mean)) + ((memoryFraction - mean) * (memoryFraction - mean)) + ((volumeFraction - mean) * (volumeFraction - mean))) / float64(3))
		// Since the variance is between positive fractions, it will be positive fraction. 1-variance lets the
		// score to be higher for node which has least variance and multiplying it with 10 provides the scaling
		//		// factor needed.
		return int64((1 - variance) * float64(schedulerapi.MaxPriority))
	}

	if cpuFraction >= 1 || memoryFraction >= 1 {
		// if requested >= capacity, the corresponding host should never be preferred.
		return 0
	}
	// Upper and lower boundary of difference between cpuFraction and memoryFraction are -1 and 1
	// respectively. Multiplying the absolute value of the difference by 10 scales the value to
	// 0-10 with 0 representing well balanced allocation and 10 poorly balanced. Subtracting it from
	// 10 leads to the score which also scales from 0 to 10 while 10 representing well balanced.
	diff := math.Abs(cpuFraction - memoryFraction)
	return int64((1 - diff) * float64(schedulerapi.MaxPriority))
}

func fractionOfCapacity(requested, capacity int64) float64 {
	if capacity == 0 {
		return 1
	}
	return float64(requested) / float64(capacity)
}
分为两种模式，第一种计算目录使用率，第二种只计算cpu以及内存使用率。计算各种指标的使用率，计算均值，计算方差

cpuFraction := fractionOfCapacity(requested.MilliCPU, allocable.MilliCPU)

memoryFraction := fractionOfCapacity(requested.Memory, allocable.Memory)

volumeFraction := float64(requestedVolumes) / float64(allocatableVolumes)

mean := (cpuFraction + memoryFraction + volumeFraction) / float64(3)
variance := float64((((cpuFraction - mean) * (cpuFraction - mean)) + ((memoryFraction - mean) * (memoryFraction - mean)) + ((volumeFraction - mean) * (volumeFraction - mean))) / float64(3))

return int64((1 - variance) * float64(schedulerapi.MaxPriority))

2.ImageLocalityPriority  根据需要的镜像在Node节点已经存在的数量，因为镜像如果不存在则需要到仓库拉取，这样时间较长

// ImageLocalityPriorityMap is a priority function that favors nodes that already have requested pod container's images.
// It will detect whether the requested images are present on a node, and then calculate a score ranging from 0 to 10
// based on the total size of those images.
// - If none of the images are present, this node will be given the lowest priority.
// - If some of the images are present on a node, the larger their sizes' sum, the higher the node's priority.
func ImageLocalityPriorityMap(pod *v1.Pod, meta interface{}, nodeInfo *schedulercache.NodeInfo) (schedulerapi.HostPriority, error) {
	node := nodeInfo.Node()
	if node == nil {
		return schedulerapi.HostPriority{}, fmt.Errorf("node not found")
	}

	var score int
	if priorityMeta, ok := meta.(*priorityMetadata); ok {
		score = calculatePriority(sumImageScores(nodeInfo, pod.Spec.Containers, priorityMeta.totalNumNodes))
	} else {
		// if we are not able to parse priority meta data, skip this priority
		score = 0
	}
	 
	return schedulerapi.HostPriority{
		Host:  node.Name,
		Score: score,
	}, nil
}
计算一个镜像的得分

spread := float64(imageState.NumNodes) / float64(totalNumNodes)
int64(float64(imageState.Size) * spread)

计算一个Node节点所有镜像的得分

sum += scaledImageScore(state, totalNumNodes)

转换得分到区间0--10

int(int64(schedulerapi.MaxPriority) * (sumScores - minThreshold) / (maxThreshold - minThreshold))

3.LeastResourceAllocation  cpu以及内存的平均cpu利用率

leastResourcePriority = &ResourceAllocationPriority{"LeastResourceAllocation", leastResourceScorer}
func leastResourceScorer(requested, allocable *schedulercache.Resource, includeVolumes bool, requestedVolumes int, allocatableVolumes int) int64 {
	return (leastRequestedScore(requested.MilliCPU, allocable.MilliCPU) +
		leastRequestedScore(requested.Memory, allocable.Memory)) / 2
}
计算可用的资源利用率

func leastRequestedScore(requested, capacity int64) int64 {
	if capacity == 0 {
		return 0
	}
	if requested > capacity {
		return 0
	}

	return ((capacity - requested) * int64(schedulerapi.MaxPriority)) / capacity
}
计算资源可用率

((capacity - requested) * int64(schedulerapi.MaxPriority)) / capacity

计算cpu以及内存可用利用率均值

(leastRequestedScore(requested.MilliCPU, allocable.MilliCPU) + leastRequestedScore(requested.Memory, allocable.Memory)) / 2

其余优先级函数可以参考如下：

kubernetes/pkg/scheduler/algorithm/priorities

————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84997564

# Kubernetes24--弹性伸缩2

结合论文研究一下云平台中弹性伸缩技术，k8s中提供了HPA策略来实现弹性扩容，基于负载预测的弹性伸缩技术。

HPA核心代码

kubernetes/pkg/controller/podautoscaler/horizontal.go

​                     

       伸缩理论关注的问题主要是在面临超出现有集群最大承载能力的时候，如何通过调整集群的规模以提高集群的承载能力，从而保证用户体验和系统服务的稳定性，同时在集群负载很低的时候，尽可能的减少闲置服务器带来的资源浪费。资源的伸缩性是指通过增加 CPU、内存等硬件资源的投入来提升软件效率以达到更高的系统性能。平常所说的集群伸缩方法大多数是指资源的伸缩性，而资源的伸缩性又可以划分为两个子类。从伸缩的方向划分，分为纵向的伸缩和横向的伸缩。
       纵向伸缩是指通过提升系统当前各个节点的处理能力来达到提升系统整体处理能力的伸缩方法。提高各节点的处理能力具体来说包括服务器升级现有的配置，例如更换主频更高、多核的处理器，更换容量更大的内存条，配置读写速度更快的硬盘、甚至替换为更高端、更强劲的处理器等。
      横向伸缩是单纯地通过增加节点的数量来提升系统整体的处理能力。横向伸缩的优点在于当每台服务器成本比较低的情况下，可以很容易地搭建起一个系统性能有保障的集群，相比纵向伸缩集群，这个集群面对增加的用户量或者数据量带来的性能瓶颈可以处理得更灵活、游刃有余，并且能够很好的减少由于单台服务器出现故障而对于系统整体带来的影响。

当前阶段常用的策略算法有：
 1.单指标非预测性算法
        常用的有 CPU 利用率为指标，不考虑多条历史记录，只关注当前的负载，直接将历史记录中的最后一条数据(可认为是当前负载)作为扩容指数输出。这种算法简单有效，根据实时的负载大小与阈值进行对比，超过了则扩容，否则保持原样。因此这种算法虽然简单，却是目前工业界普遍采用的。缺点：容易受噪音干扰；反应滞后，不具备预测性；无法预测到 CPU 之外的因素（内存等）引起的负载过重情况。
 2.多指标自回归算法
自回归模型一般用于统计和信号处理，是一种经常被用来对各种自然现象进行建模和预测的随机过程。Web 应用的负载变化虽然不是自然现象，但其多变且难以准确预测的特征在某种程度上具有一定的随机性，因此可以用自回归模型对负载进行建模和预测。将多个负载指标数据经过处理，求出加权负载平均值作为指标，将之用自回归算法处理得出扩容指数。缺点：该方法可以实现预测，且考虑了多种指标的影响。但是没有考虑到 Web 服务访问暗含的规律性，预测性不够高。
Web负载分析
        为了能够准确地预测 Web 集群负载，需要对负载进行收集分析，从而了解负载的特性，并针对这些特性来选择最适合的方法来进行平台负载预测。负载的几个特性，主要有以下六个方面 
1.  负载有很大的波动性，这说明有必要通过预测算法来改善负载相关问题。
2.  方差和最大值之类的差异量数 1 与平均值是正相关的，平均负载高的机器也有较大的方差和最大值，这种关联表明对高负载量的机器进行预测更有价值。
3.  负载相对来说比较复杂，有时多峰分布的负载不能根据一般的分布曲线来预测。
4.  通过负载时间序列分析可以知道，负载与时间有很强的关联，也就是说上一时间段的负载对下一时间段的负载有很大影响。所以对于负载预测来说，线性模型也许是比较适合的预测模型。
5.  负载具有自相似性，其赫斯特指数范围是0.63 至 0.97，已经接近赫斯特指数的上限 2。通过这个结果可知负载按照时间以复杂的方式变化着，并存在长期依赖性。
6.  负载的变化在不同时间段显示出不同特性。有时负载在一段较长时间内非常稳定，但是会忽然有一个较大的波动，这种波动可能是因为新的任务被创建或销毁等，针对这种突发波动，线性模型可能需要自适应并进行修改。

利用GBDT算法，选取cpu,内存，请求响应时间作为指标，结合时间序列来进行负载预测以实现弹性伸缩。

​                                                             

自动扩展在以往研究中有多种实现方式，比如基于阈值的策略、增强型学习、队列理论、控制理论以及时间序列分析等。其中，基于阈值的策略在目前的云中实现得较为普遍。时间序列预测分析方法有自回归预测模型、指数预测模型和人工神经网络模型。

自回归预测模型。自回归模型是用过去若干时刻的变量的线性组合来预测以后某时刻随机变量的线性回归模型
指数平滑预测模型。指数平滑法是一种特殊的加权平均法，对本期观察值和本期预测值赋予不同的权重，求得下一期预测值的方法
BP 神经网络预测模型。BP 神经网络的核心是 BP算法：将正确的结果和产生的结果进行比较，得到误差，再逆推对神经网中的权重进行反馈修正，从而来完成学习的过程，这种向后反馈的学习机制被称为BP神经网络的反馈机制

​                                             

       在应用系统面临高并发、大流量的突发式请求时，监控系统会监测到应用容器的 CPU、内存等资源使用量迅速上升，对应的策略就是通过设置告警阈值来触发弹性伸缩机制。以递增步长的节奏进行扩容；峰值过后，以固定步长的节奏进行缩容。由弹性伸缩模块启动或删除的应用容器，在Etcd中有对应的键值变化。Confd 检测到这种变化后，就会动态生成 Haproxy 的配置文件并重新加载，之后就会将前端的用户请求分流到后端的应用容器上，提供具体的服务，实现应用系统的负载均衡。此动态负载均衡弹性伸缩系统依据应用容器的负载状态，来提供对应的处理能力，按需供给，在保证系统的稳定运行的前提下，将系统开销降低到最低。

基于灰色模型的云资源动态伸缩功能

​                                                  

灰色预测通过鉴别系统因素之间发展趋势的差异程度，对原始数据进行生成处理，寻找系统变动的规律，生成有较强规律性 的 数 据 序 列
，然 后 建 立 相 应 的 微 分 方 程 模型，从而预测事物未来发展趋势。灰色预测的数据是通过生成数据模型得到的预测值的逆处理结果。灰色预测以灰色模型为基础。

容器化工作流系统

​                                 

基于负载预测的弹性伸缩服务模型，设定性能指标，构建模型预测负载值，提前进行弹性伸缩服务，其中常用的预测模型由自回归模型，指数预测模型，BP神经网络，GBDT预测以及灰度模型等，其根本是回归问题，可以采用机器学习等方法来提前预测。

参考论文

[1]徐建中,王俊,周迅钊,徐雷.一种基于预测的云计算的弹性伸缩策略[J].计算机与数字工程,2018,46(06):1160-1162+1231.

[1]王强,王瑞刚,周德永.基于Docker的动态负载均衡弹性伸缩系统[J].计算机与数字工程,2018,46(06):1140-1144+1159.

[1]李正寅. 服务创新平台中基于Docker的弹性负载均衡功能的设计与实现[D].北京邮电大学,2018.

[1]刘彪,王宝生,邓文平.云环境下支持弹性伸缩的容器化工作流框架[J/OL].计算机工程:1-12[2018-12-14].https://doi.org/10.19678/j.issn.1000-3428.0049811.

[1]王天泽.基于灰色模型的云资源动态伸缩功能研究[J].软件导刊,2018,17(04):131-134.

[1]陈金光. 基于阿里云的Kubernetes容器云平台的设计与实现[D].浙江大学,2018.

[1]王晓钰. 基于云平台可弹性扩缩的Web应用系统的研究与实现[D].北京邮电大学,2018.

[1]刘锦福. 基于Docker的直播云平台弹性调度系统设计及实现[D].北京邮电大学,2018.

[1]梅荣.基于云计算的弹性负载均衡服务研究[J].中国公共安全,2018(01):194-199.

[1]王晓钰,吴伟明,谷勇浩.基于云平台的弹性Web集群扩缩容机制的研究[J].软件,2017,38(11):24-28.

[1]杨若琪.云计算中弹性伸缩负载预测算法的研究和改进[J].电子制作,2017(16):40-41+59

[1]张淼. 面向云服务的弹性调度算法的研究与实现[D].哈尔滨工业大学,2017.
————————————————
版权声明：本文为CSDN博主「暗夜猎手-大魔王」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u014106644/article/details/84999086