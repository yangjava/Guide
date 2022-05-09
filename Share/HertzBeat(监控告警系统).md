[HertzBeat赫兹跳动](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fdromara%2Fhertzbeat) 是由[Dromara](https://gitee.com/link?target=https%3A%2F%2Fdromara.org)孵化，[TanCloud](https://gitee.com/link?target=https%3A%2F%2Ftancloud.cn)开源的一个支持网站，API，PING，端口，数据库，全站等监控类型，支持阈值告警，告警通知(邮箱，webhook，钉钉，企业微信，飞书机器人)，拥有易用友好的可视化操作界面的开源监控告警项目。  

**官网: [hertzbeat.com](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fhertzbeat.com%2F) | [tancloud.cn](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftancloud.cn%2F)**

此升级版本包含了大量特性与修复，包括用户急需的账户用户配置，丰富了主流第三方告警通知(企业微信机器人，钉钉机器人，飞书机器人)，更好看的邮件模版，自定义邮件服务器等，欢迎使用。

版本特性：

1. 告警通知：集成飞书官方WebHook实现推送告警信息 #PR9 由 @learning-code 贡献 thanks
2. 告警通知：实现企业微信WebHook告警信息推送 #PR8 由 @learning-code[ ](https://gitee.com/learning-code)贡献 thanks
3. 告警通知：告警邮件通知模版优化 由 @learning-code[ ](https://gitee.com/learning-code)贡献 thanks
4. 告警通知：集成钉钉群机器人实现推送告警信息
5. 账户：暴露支持YML文件配置登陆用户账户信息
6. 支持自定义邮件服务器
7. 新增帮助中心，监控告警等功能使用过程中的帮助文档. [https://tancloud.cn/docs/help/guide](https://gitee.com/link?target=https%3A%2F%2Ftancloud.cn%2Fdocs%2Fhelp%2Fguide)
8. DOC其它文档更新，本地启动帮助
9. 新LOGO更新
10. 监控采集间隔时间放开为7天
11. 新增controller接口入参限定修饰符 由 @learning-code[ ](https://gitee.com/learning-code)贡献 thanks

BUG修复

1. 监控host参数修复校验.
2. fixBug自定义邮件服务器未生效
3. 邮件页面优化，fix告警级别未转译
4. fix监控删除后告警定义关联未删除
5. 调整jvm启动内存大小,fixOOM
6. fixbug重启后状态异常监控无法触发恢复告警
7. fix pmd error
8. bugfix告警设置确定后异常,按钮还在旋转
9. fix多余租户ID依赖
10. fix receiver的email类型错误，调整弹出框大小
11. fixbug告警定义关联监控不存在时异常

欢迎在线试用 [https://console.tancloud.cn](https://gitee.com/link?target=https%3A%2F%2Fconsole.tancloud.cn)

版本升级注意⚠️

1.0-beta2升级上来，MYSQL的数据库需执行。
ALTER TABLE alert_define_monitor_bind DROP monitor_name;

1.0-beta2,1.0-beta3升级上来，MYSQL的数据库需执行。
ALTER TABLE notice_receiver ADD access_token varchar(255);

\-----------------------

[HertzBeat赫兹节拍](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fusthe%2Fhertzbeat) 是由[TanCloud](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Ftancloud.cn%2F)开源的一个支持网站，API，PING，端口，数据库等监控类型，拥有易用友好的可视化操作界面的开源监控告警项目。
我们也提供了对应的 **[SAAS版本监控云](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fconsole.tancloud.cn%2F)**，中小团队和个人无需再为了监控自己的网站资源，而去部署一套繁琐的监控系统，**[登陆即可免费开始](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fconsole.tancloud.cn%2F)**。
HertzBeat 支持[自定义监控](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fhertzbeat.com%2Fdocs%2Fadvanced%2Fextend-point) ,只用通过配置YML文件我们就可以自定义需要的监控类型和指标，来满足常见的个性化需求。
HertzBeat 模块化，`manager, collector, scheduler, warehouse, alerter` 各个模块解耦合，方便理解与定制开发。
HertzBeat 支持更自由化的告警配置(计算表达式)，支持告警通知，告警模版
欢迎登陆 HertzBeat 的 [云环境TanCloud](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fconsole.tancloud.cn%2F) 试用发现更多。
我们正在快速迭代中，欢迎参与加入一起共建项目开源生态。

`HertzBeat`的多类型支持，易扩展，低耦合，希望能帮助开发者和中小团队快速搭建自有监控系统。

### 开箱即用

中小团队和个人无需再为了监控自己的网站资源，而去部署一套繁琐的监控系统。往往有时候，那套监控系统比自身网站消耗的资源还大。**TANCLOUD** 提供**SAAS**云版本，[**注册登录**](https://www.console.tancloud.cn/)即可开始您的服务监控。
安全是最重要的，我们对账户密钥和监控密钥全链路加密。

### 多支持与自定义

TANCLOUD目前支持对网站，API，PING连通性，端口可用性，MYSQL数据库等的监控，不久我们将兼容 prometheus 协议，提供更多的监控类型和性能指标。
我们提供了更自由化的阈值告警配置，支持邮箱，短信，webhook，钉钉，企业微信，飞书机器人等告警通知。
不同团队的监控需求千变万化，我们提供[**自定义监控**](https://tancloud.cn/docs/advanced/extend-point)，仅需配置YML就能快速接入监控系统。

### 拥抱开源

TANCLOUD监控系统代码开源，非常欢迎任何对此有兴趣的同学参与中来，我们一起进步，丰富的资源文档正在完善中。
中二的我们相信开源改变世界！
[**HertzBeat Github 代码仓库**](https://github.com/dromara/hertzbeat)
[**HertzBeat Gitee 代码仓库**](https://gitee.com/dromara/hertzbeat)

# HertzBeat赫兹跳动

> 易用友好的高性能监控告警系统。

![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/web-monitor.svg) ![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/ping-connect.svg) ![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/port-available.svg) ![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/database-monitor.svg) ![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/custom-monitor.svg) ![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/threshold.svg) ![tan-cloud](https://cdn.jsdelivr.net/gh/dromara/hertzbeat@gh-pages/img/badge/alert.svg)

## 📫 前言[](https://tancloud.cn/docs/#-前言)

> 毕业后投入很多业余时间也做了一些开源项目,[Sureness](https://github.com/dromara/sureness) [Bootshiro](https://gitee.com/tomsun28/bootshiro) [Issues-translate-action](https://github.com/usthe/issues-translate-action) , 当时上班有空就回答网友问题，下班回家写开源代码，远程帮人看问题，还总感觉时间不够用，当时想如果不去上班能做自己热爱的该多好。
> 年轻就要折腾，何况还是自己很想做的。于是乎，21年底我放弃激励裸辞开始全职开源了(这里感谢老婆大人的全力支持)，也是第一次全职创业。 自己在APM领域做了多年，当然这次创业加开源的方向也就是老本行APM监控系统，我们开发一个支持多种监控指标(更多监控类型指标正在适配中)，拥有自定义监控，支持阈值告警通知等功能，面向开发者友好的开源监控项目-HertzBeat赫兹跳动。 想到很多开发者和团队拥有云上资源，可能只需要使用监控服务而并不想部署监控系统，我们也提供了可以直接登录使用的SAAS云监控版本-[TanCloud探云](https://console.tancloud.cn/)。
> 希望大家多多支持点赞，非常感谢。

## 🎡 介绍[](https://tancloud.cn/docs/#-介绍)

> [HertzBeat赫兹跳动](https://github.com/dromara/hertzbeat) 是由[TanCloud](https://tancloud.cn/)开源的一个支持网站，API，PING，端口，数据库等监控类型，拥有易用友好的可视化操作界面的开源监控告警项目。
> 当然，我们也提供了对应的[SAAS云监控版本](https://console.tancloud.cn/)，中小团队和个人无需再为了监控自己的网站资源，而去部署一套繁琐的监控系统，[登录即可免费开始](https://console.tancloud.cn/)监控之旅。
> HertzBeat 支持自定义监控，只用通过配置YML文件我们就可以自定义需要的监控类型和指标，来满足常见的个性化需求。 HertzBeat 模块化，`manager, collector, scheduler, warehouse, alerter` 各个模块解耦合，方便理解与定制开发。
> HertzBeat 支持更自由化的告警配置(计算表达式)，支持告警通知，告警模版
> 欢迎登录 HertzBeat 的 [云环境TanCloud](https://console.tancloud.cn/) 试用发现更多。
> 我们正在快速迭代中，欢迎参与加入共建项目开源生态。

> `HertzBeat`的多类型支持，易扩展，低耦合，希望能帮助开发者和中小团队快速搭建自有监控系统。

## 🥐 模块[](https://tancloud.cn/docs/#-模块)

- [manager](https://github.com/dromara/hertzbeat/tree/master/manager)

   

  提供监控管理,系统管理基础服务

  > 提供对监控的管理，监控应用配置的管理，系统用户租户后台管理等。

- [collector](https://github.com/dromara/hertzbeat/tree/master/collector)

   

  提供监控数据采集服务

  > 使用通用协议远程采集获取对端指标数据。

- [scheduler](https://github.com/dromara/hertzbeat/tree/master/scheduler)

   

  提供监控任务调度服务

  > 采集任务管理，一次性任务和周期性任务的调度分发。

- [warehouse](https://github.com/dromara/hertzbeat/tree/master/warehouse)

   

  提供监控数据仓储服务

  > 采集指标结果数据管理，数据落盘，查询，计算统计。

- [alerter](https://github.com/dromara/hertzbeat/tree/master/alerter)

   

  提供告警服务

  > 告警计算触发，监控状态联动，告警配置，告警通知。

- [web-app](https://github.com/dromara/hertzbeat/tree/master/web-app)

   

  提供可视化控制台页面

  > 监控告警系统可视化控制台前端(angular+ts+zorro)

![hertzBeat](https://tancloud.gd2.qingstor.com/img/docs/hertzbeat-stru.svg)

[Edit this page](https://github.com/dromara/hertzbeat/edit/master/home/docs/introduce.md)