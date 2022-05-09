### 软件简介

[如何使用英特尔 ®oneAPI 工具实现 PyTorch 优化，直播火热报名中 >>>>>> ![img](https://www.oschina.net/img/hot3.png)](https://uao.so/pct46bd2f2)

Dolphins-DevOPS 是元豚科技开发的 DevOPS-K8S 服务自动化部署平台。

**主要特性**

- 云配置自动化：基于 Java 定时任务接平台任务，自动化部署 K8S 托管集群，并打通 K8S Client，构建容器周边的服务（Jenkins、Sonar、堡垒机、容器安全检测、漏洞扫描等）
- 服务配置打通：CI/CD 逻辑的打通，平台可以进行业务自动化发布。
- 部署支付：打通微信支付逻辑。

**目标**

- 满足云上基础服务的配置，如 VPC、网络、CFS 之类的基础服务。
- 创建 K8S 集群门槛降低，让人人都能用 K8S 集群。
- 满足自动化部署的需求。
- 满足业务部署的需求，安全基础服务的部署能力。

**开发技术栈**

- 后端：Java Spring boot、模块化
- 工具端：Python、Golang
- 前端：Vue
- 服务部署：基于 K8S 容器化部署

### DEMO

主界面:

![img](https://static.oschina.net/uploads/space/2022/0430/083449_kdy1_5430600.jpg)

工程分析页面:

![img](https://static.oschina.net/uploads/space/2022/0430/083507_U8lu_5430600.jpg)

发布页面:

![img](https://static.oschina.net/uploads/space/2022/0430/083522_gj1M_5430600.jpg)

### 备注

- 这个版本并不完善，但是基本的部署、发布跑通了，还需要继续优化迭代。
- 每次阿里云、腾讯云改变规则时，创建集群会有一定概率失败，腾讯云百分百成功，阿里云失败概率极高。