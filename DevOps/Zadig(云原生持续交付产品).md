# Zadig

## 什么是 Zadig

Zadig 是 KodeRover 公司基于 Kubernetes 自主设计、研发的开源分布式持续交付 (Continuous Delivery) 产品，为开发者提供云原生运行环境，支持开发者本地联调、微服务并行构建和部署、集成测试等。Zadig 内置了面向 Kubernetes、Helm、云主机/物理机、大体量微服务等复杂业务场景的最佳实践，为工程师一键生成自动化工作流 (workflow)。Zadig 不改变现有习惯和流程，几乎兼容所有软件架构，无缝集成 GitHub/GitLab、Jenkins、多家云厂商等，运维成本极低。

我们的目标是通过云原生技术的运用和工程产品赋能，打造极致、高效、愉悦的开发者工作体验，让工程师成为企业创新的核心引擎。

![Zadig-Business-Architecture](png\Zadig\Zadig-Business-Architecture.jpg)

## 核心能力

- **高并发的工作流**

  系统经过简单配置，即可自动生成高并发工作流，高效执行构建、部署、测试等任务。这一设计解决了微服务架构下带来的多服务交付效率低下的问题。

- **以服务为核心的环境**

  只需一套服务配置，即可在几分钟创建多套数据隔离的测试环境，为开发者日常调试、集成测试验证、产品演示提供强力支撑。现有环境无需迁移，一键托管即可轻松浏览、调试环境中的所有服务。

- **无侵入的自动化测试**

  便捷对接已有的自动化测试框架，通过 GitHub/GitLab WebHook 自动构建、部署、测试。通过办公 IM 机器人为开发者提供第一时间质量反馈，有效实现“测试左移”，充分体现测试价值。

- **开发本地联调 CLI**

  开发可以本地编辑代码，一键构建、部署到联调环境，无需处理复杂且繁琐的工作流程，省出宝贵时间去创造更多产品价值。

## 为谁服务

- #### 开发工程师

  - 基于代码合并请求级别的持续集成，并获得单元测试，代码扫描，耗时和通过率的质量反馈和改进建议
  - 定时器/Webhook 支持自动触发执行工作流，更新环境，运行自动化测试，获得详细的质量结果反馈
  - 一键生成独立环境，可直连容器云环境进行开发、调试、自测

- #### 测试（开发）工程师

  - 测试任务维护和管理，通过工作流执行自动化功能测试，获得相应测试报告
  - 管理/执行一个项目的交付工作流任务，成功执行后，进行版本交付，并获得交付版本的服务配置、镜像信息、代码信息、测试结果等

- #### DevOps（运维） 工程师

  - 一个项目可以实现完整的容器化环境管理、交付管理
  - 根据 Branch/Tag 执行发布工作流，版本交付完整信息数据流，不限于需求/代码/工作流/配置的 changelog

- #### 项目管理/产品管理/工程效率管理人员

  - 可以随时将新功能展示给内外部用户 POC
  - 实现对不同客户进行内部交付版本管理和检索
  - 可查看各团队持续集成、持续交付、持续部署等行业效能 DevOps 指标

## 功能介绍

具有产品持续交付、持续测试、持续追踪的全流程能力，包括以下核心功能：

- 项目：工作流、环境、服务、构建、测试、版本管理
- 测试中心：自动化测试管理
- 交付中心：版本管理、交付物追踪
- 数据视图：数据概览、效能洞察 - 构建效能、测试效能、部署效能
- 集成管理：GitHub/GitLab/Gerrit/CodeHub 集成、SSO/LDAP/AD 账号系统集成、Jenkins/Jira 集成、软件包管理、构建镜像管理
- 基础设施：镜像仓库、对象存储、Helm 仓库、集群管理、主机管理
- 系统配置：RBAC 权限、操作日志、公告管理

## 项目

Zadig 中的项目包括工作流、环境、服务、构建、测试、版本等资源，用户在项目中可以进行服务开发、服务部署、集成测试、版本发布等操作。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#工作流)工作流

典型的软件开发过程一般包括以下几个步骤：

> 编写代码 -> 构建 -> 部署 -> 测试 -> 发布

工作流就是 Zadig 平台对这样一个开发流程的实现，通过工作流来更新环境中的服务或者配置。

### [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#工作流组成)工作流组成

Zadig 工作流简化示意图如下所示：

![工作流基本流程](https://docs.koderover.com/zadig/assets/img/workflow_basic.af2c57d6.png)

目前工作流基本组成部分有：

- 构建：拉取代码，执行构建
- 部署：将构建产物部署到测试环境中
- 测试：执行自动化测试，对部署结果进行验证
- 分发：完成测试验证后，将构建产物分发到待发布的仓库

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#环境)环境

Zadig 环境是一组服务集合及其配置、运行环境的总称，与 Kubernetes 的 NameSpace 一对一的对应关系，使用一套服务模板可以创建多套环境。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#服务)服务

Zadig 中的服务可以理解为一组 Kubernetes 资源，包括 Ingress、Service、Deployment/Statefulset、ConfigMap 等，也可以是一个完整的 Helm Chart 或者云主机/物理机服务，成功部署后可对外提供服务能力。

### [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#服务组件)服务组件

服务组件是 Zadig 中可被更新的最小单元，是使用 Kubernetes 作为基础设施的项目中的概念。一个服务中可包括多个服务组件。不同项目中的服务组件信息如下表：

| 项目类型            | 服务组件来源                                            | 服务组件名称                 | 服务组件镜像信息          |
| ------------------- | ------------------------------------------------------- | ---------------------------- | ------------------------- |
| K8s YAML 项目       | 服务 K8s YAML 配置中的 Deployment/Statefulset、Job 资源 | 对应资源的 container 名称    | 对应资源的 container 镜像 |
| K8s Helm Chart 项目 | 服务 Helm Chart values.yaml 文件                        | values.yaml 文件中的镜像名称 | values.yaml 文件中的镜像  |
| K8s 托管项目        | 被托管服务实例中的 Deployment/Statefulset、Job 资源     | 对应资源的 container 名称    | 对应资源的 container 镜像 |



服务组件是服务构建配置中的一部分。为服务组件配置构建后，运行工作流时可选择对应的服务组件对其进行更新。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#构建)构建

Zadig 构建属于服务配置的一部分，同时在工作流运行阶段会被调用，与服务是一对多的对应关系，即一套构建可以支持多个服务共享。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#测试)测试

Zadig 测试属于项目的资源，同时也可以作为一个非必要阶段在工作流中调用，支持跨项目。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/concepts/#版本管理)版本管理

Zadig 版本是一个完整的可靠交付物，比如 Helm Chart，或 K8s YAML 完整配置文件。

## 第一种模式：All in One 一键安装

以 root 用户登录 Linux 主机执行以下命令：

```bash
export IP=<IP> # 主机 IP，用于访问 Zadig 系统
export PORT=<PORT> # 随机填写 30000 - 32767 区间的任一端口，如果安装过程中，发现端口占用，换一个端口再尝试
curl -SsL https://github.com/koderover/zadig/releases/latest/download/all_in_one_install_quickstart.sh | bash
```

1
2
3

若网络状况欠佳，也可使用官方脚本安装:

```bash
export IP=<IP> # 主机 IP，用于访问 Zadig 系统
export PORT=<PORT> # 随机填写 30000 - 32767 区间的任一端口，如果安装过程中，发现端口占用，换一个端口再尝试
curl -SsL https://download.koderover.com/install?type=all-in-one | bash
```

1
2
3

安装完毕后，通过 **http://<IP>:<PORT>** 即可访问 Zadig 系统。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/try-out-install/#第二种模式-基于现有-kubernetes-安装)第二种模式：基于现有 Kubernetes 安装

前提

Kubernetes 集群版本在 1.16.0 及以上

以集群管理员身份，执行以下命令：

```bash
export IP=<IP> # 集群任一节点公网 IP，用于访问 Zadig 系统
export PORT=<PORT> # 随机填写 30000 - 32767 区间的任一端口，如果安装过程中，发现端口占用，换一个端口再尝试
curl -SsL https://github.com/koderover/zadig/releases/latest/download/install.sh | bash
```

1
2
3

若网络状况欠佳，也可使用官方脚本安装:

```bash
export IP=<IP> # 集群任一节点公网 IP，用于访问 Zadig 系统
export PORT=<PORT> # 随机填写 30000 - 32767 区间的任一端口，如果安装过程中，发现端口占用，换一个端口再尝试
curl -SsL https://download.koderover.com/install?type=standard| bash
```



安装完毕后，通过 **http://<IP>:<PORT>** 即可访问 Zadig 系统。

## [#](https://docs.koderover.com/zadig/v1.11.0/quick-start/try-out-install/#第三种模式-基于-helm-命令安装)第三种模式：基于 Helm 命令安装