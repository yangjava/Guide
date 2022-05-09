# [Solon 1.6.30 发布，更现代感的应用开发框架](https://www.oschina.net/news/185941/solon-1-6-30-released)

来源: 投稿

作者: [林西东](https://my.oschina.net/noear)

2022-03-11

[ 0](https://www.oschina.net/news/185941/solon-1-6-30-released#comments)

### 相对于 Spring Boot 和 Spring Cloud 的项目

- 启动快 5 ～ 10 倍
- qps 高 2～ 3 倍
- 运行时内存节省 1/3 ~ 1/2
- 打包可以缩小到 1/2 ~ 1/10（比如，90Mb 的变成了 9Mb）

### 关于 Solon

Solon 是一个更现代感的应用开发框架，轻量、开放生态型的。支持 Web、Data、Job、Remoting、Cloud 等任何开发场景。

- 强调，**克制 + 简洁 + 开放 + 生态的原则**
- 力求，**更小、更少、更快、更自由的体验**

目前有近**130**个生态插件，含盖了日常开发的各种需求。

### 本次主要更新

- 插件 solon.boot.smarthttp，升级 smart-http-server 到：1.1.12
- 插件 solon.boot.jlhttp ，增加 maxHeaderSize, maxBodySize 设置支持
- 插件 mybatisplus-solon-plugin 更名为：mybatis-plus-solon-plugin（旧的仍保留）
- 新增 solon.boot.jetty.add.servlet 插件
- 提取 solon boot 相关的公共配置，独立为 solon.boot 模块
- 优化 mybatis-solon-plugin 关于事务的对接适配

### 进一步了解 Solon

- [《想法与架构笔记》](https://my.oschina.net/noear/blog/4980834)
- [《生态预览》](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fsolon.noear.org%2Farticle%2Ffamily-preview)
- [《与 Spring Boot 的区别？》](https://my.oschina.net/noear/blog/4863844)
- [《与 Spring Cloud 的区别？》](https://my.oschina.net/noear/blog/5039169)