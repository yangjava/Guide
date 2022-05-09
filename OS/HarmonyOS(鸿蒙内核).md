# 鸿蒙内核源码分析

### **主流站点覆盖.定期同步更新**

- 鸿蒙研究站

- - (国内) | [http://weharmonyos.com](https://link.zhihu.com/?target=http%3A//weharmonyos.com/)
  - (国外) | [https://weharmony.github.io](https://link.zhihu.com/?target=https%3A//weharmony.github.io/)

- oschina | [https://my.oschina.net/weharmony](https://link.zhihu.com/?target=https%3A//my.oschina.net/weharmony)

- 博客园 | [https://www.cnblogs.com/weharmony/](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/weharmony/)

- 知乎 | https://www.zhihu.com/people/weharmonyos

- csdn | [https://blog.csdn.net/kuangyufei](https://link.zhihu.com/?target=https%3A//blog.csdn.net/kuangyufei)

- 51cto | [https://harmonyos.51cto.com/column/34](https://link.zhihu.com/?target=https%3A//harmonyos.51cto.com/column/34)

- 掘金 | [https://juejin.cn/user/756888642000808](https://link.zhihu.com/?target=https%3A//juejin.cn/user/756888642000808)

### **百篇博客分析.深挖内核地基**

- 给鸿蒙内核源码加注释过程中，整理出以下文章。内容立足源码，常以生活场景打比方尽可能多的将内核知识点置入某种场景，具有画面感，容易理解记忆。说别人能听得懂的话很重要! 百篇博客绝不是百度教条式的在说一堆诘屈聱牙的概念，那没什么意思。更希望让内核变得栩栩如生，倍感亲切.确实有难度，自不量力，但已经出发，回头已是不可能的了。　
- 与代码有bug需不断debug一样，文章和注解内容会存在不少错漏之处，请多包涵，但会反复修正，持续更新，v**.xx 代表文章序号和修改的次数，精雕细琢，言简意赅，力求打造精品内容。

按时间顺序:

- [v01.12 鸿蒙内核源码分析(双向链表) | 谁是内核最重要结构体](https://zhuanlan.zhihu.com/p/418766952)
- [v02.06 鸿蒙内核源码分析(进程管理) | 谁在管理内核资源](https://zhuanlan.zhihu.com/p/418943425)
- [v03.06 鸿蒙内核源码分析(时钟任务) | 调度的源动力从哪来](https://zhuanlan.zhihu.com/p/418985180)
- [v04.03 鸿蒙内核源码分析(任务调度) | 内核调度的单元是谁](https://zhuanlan.zhihu.com/p/418985797)
- [v05.05 鸿蒙内核源码分析(任务管理) | 如何管理任务池](https://zhuanlan.zhihu.com/p/418986396)
- [v06.03 鸿蒙内核源码分析(调度队列) | 内核调度也需要排队](https://zhuanlan.zhihu.com/p/418990908)
- [v07.08 鸿蒙内核源码分析(调度机制) | 任务是如何被调度执行的](https://zhuanlan.zhihu.com/p/418991375)
- [v08.03 鸿蒙内核源码分析(总目录) | 百万汉字注解 百篇博客分析](https://zhuanlan.zhihu.com/p/418991797)
- [v09.04 鸿蒙内核源码分析(调度故事) | 用故事说内核调度](https://zhuanlan.zhihu.com/p/418995590)
- [v10.03 鸿蒙内核源码分析(内存主奴) | 皇上和奴才如何相处](https://zhuanlan.zhihu.com/p/418991797)
- [v11.03 鸿蒙内核源码分析(内存分配) | 内存有哪些分配方式](https://zhuanlan.zhihu.com/p/419012859)
- [v12.04 鸿蒙内核源码分析(内存管理) | 虚拟内存全景图是怎样的](https://zhuanlan.zhihu.com/p/419013610)
- [v13.05 鸿蒙内核源码分析(源码注释) | 每天死磕一点点](https://zhuanlan.zhihu.com/p/419013894)
- [v14.02 鸿蒙内核源码分析(内存汇编) | 谁是虚拟内存实现的基础](https://zhuanlan.zhihu.com/p/419015360)
- [v15.03 鸿蒙内核源码分析(内存映射) | 映射真是个好东西](https://zhuanlan.zhihu.com/p/419016858)
- [v16.02 鸿蒙内核源码分析(内存规则) | 内存管理到底在管什么](https://zhuanlan.zhihu.com/p/419018875)
- [v17.04 鸿蒙内核源码分析(物理内存) | 怎么管理物理内存](https://zhuanlan.zhihu.com/p/419024737)
- [v18.02 鸿蒙内核源码分析(源码结构) | 内核文件各自含义](https://zhuanlan.zhihu.com/p/419931729)
- [v19.04 鸿蒙内核源码分析(位图管理) | 特节俭的苦命孩子](https://zhuanlan.zhihu.com/p/419932963)
- [v20.03 鸿蒙内核源码分析(用栈方式) | 谁来提供程序运行场地](https://zhuanlan.zhihu.com/p/419933219)
- [v21.07 鸿蒙内核源码分析(线程概念) | 是谁在不断的折腾CPU](https://zhuanlan.zhihu.com/p/419933484)
- [v22.03 鸿蒙内核源码分析(汇编基础) | CPU上班也要打卡](https://zhuanlan.zhihu.com/p/419996530)
- [v23.04 鸿蒙内核源码分析(汇编传参) | 如何传递复杂的参数](https://zhuanlan.zhihu.com/p/419996585)
- [v24.03 鸿蒙内核源码分析(进程概念) | 如何更好的理解进程](https://zhuanlan.zhihu.com/p/419996612)
- [v25.05 鸿蒙内核源码分析(并发并行) | 听过无数遍的两个概念](https://zhuanlan.zhihu.com/p/419996671)
- [v26.08 鸿蒙内核源码分析(自旋锁) | 当立贞节牌坊的好同志](https://zhuanlan.zhihu.com/p/419996832)
- [v27.05 鸿蒙内核源码分析(互斥锁) | 同样是锁它确更丰满](https://zhuanlan.zhihu.com/p/419996896)
- [v28.04 鸿蒙内核源码分析(进程通讯) | 九种进程间通讯方式速揽](https://zhuanlan.zhihu.com/p/419997190)
- [v29.05 鸿蒙内核源码分析(信号量) | 谁在解决任务间的同步](https://zhuanlan.zhihu.com/p/419997238)
- [v30.07 鸿蒙内核源码分析(事件控制) | 多对多任务如何同步](https://zhuanlan.zhihu.com/p/419997266)
- [v31.02 鸿蒙内核源码分析(定时器) | 内核最高级任务竟是它](https://zhuanlan.zhihu.com/p/418991797)
- [v32.03 鸿蒙内核源码分析(CPU) | 整个内核是一个死循环](https://zhuanlan.zhihu.com/p/420181108)
- [v33.03 鸿蒙内核源码分析(消息队列) | 进程间如何异步传递大数据](https://zhuanlan.zhihu.com/p/420203967)
- [v34.04 鸿蒙内核源码分析(原子操作) | 谁在为完整性保驾护航](https://zhuanlan.zhihu.com/p/418991797)
- [v35.03 鸿蒙内核源码分析(时间管理) | 内核基本时间单位是谁](https://zhuanlan.zhihu.com/p/418991797)
- [v36.05 鸿蒙内核源码分析(工作模式) | 程序界的韦小宝是谁](https://zhuanlan.zhihu.com/p/420407093)
- [v37.06 鸿蒙内核源码分析(系统调用) | 开发者永远的口头禅](https://zhuanlan.zhihu.com/p/420407155)
- [v38.06 鸿蒙内核源码分析(寄存器) | 讲真 全宇宙只佩服它](https://zhuanlan.zhihu.com/p/420407203)
- [v39.06 鸿蒙内核源码分析(异常接管) | 社会很单纯 复杂的是人](https://zhuanlan.zhihu.com/p/420407271)
- [v40.03 鸿蒙内核源码分析(汇编汇总) | 汇编可爱如邻家女孩](https://zhuanlan.zhihu.com/p/418991797)
- [v41.03 鸿蒙内核源码分析(任务切换) | 看汇编如何切换任务](https://zhuanlan.zhihu.com/p/418991797)
- [v42.05 鸿蒙内核源码分析(中断切换) | 系统因中断活力四射](https://zhuanlan.zhihu.com/p/418991797)
- [v43.05 鸿蒙内核源码分析(中断概念) | 海公公的日常工作](https://zhuanlan.zhihu.com/p/418991797)
- [v44.04 鸿蒙内核源码分析(中断管理) | 没中断太可怕](https://zhuanlan.zhihu.com/p/418991797)
- [v45.05 鸿蒙内核源码分析(Fork) | 一次调用 两次返回](https://zhuanlan.zhihu.com/p/418991797)
- [v46.05 鸿蒙内核源码分析(特殊进程) | 老鼠生儿会打洞](https://zhuanlan.zhihu.com/p/418991797)
- [v47.02 鸿蒙内核源码分析(进程回收) | 临终托孤的短命娃](https://zhuanlan.zhihu.com/p/418991797)
- [v48.05 鸿蒙内核源码分析(信号生产) | 年过半百 活力十足](https://zhuanlan.zhihu.com/p/418991797)
- [v49.03 鸿蒙内核源码分析(信号消费) | 谁让CPU连续四次换栈运行](https://zhuanlan.zhihu.com/p/418991797)
- [v50.03 鸿蒙内核源码分析(编译环境) | 编译鸿蒙防掉坑指南](https://zhuanlan.zhihu.com/p/418991797)
- [v51.04 鸿蒙内核源码分析(ELF格式) | 应用程序入口并非main](https://zhuanlan.zhihu.com/p/418991797)
- [v52.05 鸿蒙内核源码分析(静态站点) | 五一哪也没去在干这事](https://zhuanlan.zhihu.com/p/418991797)
- [v53.03 鸿蒙内核源码分析(ELF解析) | 敢忘了她姐俩你就不是银](https://zhuanlan.zhihu.com/p/418991797)
- [v54.04 鸿蒙内核源码分析(静态链接) | 一个小项目看中间过程](https://zhuanlan.zhihu.com/p/418991797)
- [v55.04 鸿蒙内核源码分析(重定位) | 与国际接轨的对外发言人](https://zhuanlan.zhihu.com/p/418991797)
- [v56.05 鸿蒙内核源码分析(进程映像) | 程序是如何被加载运行的](https://zhuanlan.zhihu.com/p/418991797)
- [v57.02 鸿蒙内核源码分析(编译过程) | 简单案例说透中间过程](https://zhuanlan.zhihu.com/p/418991797)
- [v58.03 鸿蒙内核源码分析(环境脚本) | 编译鸿蒙原来很简单](https://zhuanlan.zhihu.com/p/418991797)
- [v59.04 鸿蒙内核源码分析(构建工具) | 顺瓜摸藤调试构建过程](https://zhuanlan.zhihu.com/p/418991797)
- [v60.04 鸿蒙内核源码分析(gn应用) | 如何构建鸿蒙系统](https://zhuanlan.zhihu.com/p/418991797)
- [v61.03 鸿蒙内核源码分析(忍者ninja) | 忍者的特点就是一个字](https://zhuanlan.zhihu.com/p/418991797)
- [v62.02 鸿蒙内核源码分析(文件概念) | 为什么说一切皆是文件](https://zhuanlan.zhihu.com/p/418991797)
- [v63.04 鸿蒙内核源码分析(文件系统) | 用图书管理说文件系统](https://zhuanlan.zhihu.com/p/418991797)
- [v64.06 鸿蒙内核源码分析(索引节点) | 谁是文件系统最重要的概念](https://zhuanlan.zhihu.com/p/418991797)
- [v65.05 鸿蒙内核源码分析(挂载目录) | 为何文件系统需要挂载](https://zhuanlan.zhihu.com/p/418991797)
- [v66.07 鸿蒙内核源码分析(根文件系统) | 谁先挂到/谁就是根根](https://zhuanlan.zhihu.com/p/418991797)
- [v67.03 鸿蒙内核源码分析(字符设备) | 绝大多数设备都是这类](https://zhuanlan.zhihu.com/p/418991797)
- [v68.02 鸿蒙内核源码分析(VFS) | 文件系统是个大家庭](https://zhuanlan.zhihu.com/p/418991797)
- [v69.04 鸿蒙内核源码分析(文件句柄) | 你为什么叫句柄](https://zhuanlan.zhihu.com/p/418991797)
- [v70.05 鸿蒙内核源码分析(管道文件) | 如何降低数据流动成本](https://zhuanlan.zhihu.com/p/418991797)
- [v71.03 鸿蒙内核源码分析(Shell编辑) | 两个任务 三个阶段](https://zhuanlan.zhihu.com/p/418991797)
- [v72.01 鸿蒙内核源码分析(Shell解析) | 应用窥伺内核的窗口](https://zhuanlan.zhihu.com/p/418991797)

按功能模块:

- 前因后果 >> [总目录](https://zhuanlan.zhihu.com/p/418991797) | [调度故事](https://zhuanlan.zhihu.com/p/418995590) | [内存主奴](https://zhuanlan.zhihu.com/p/418991797) | [源码注释](https://zhuanlan.zhihu.com/p/419013894) | [源码结构](https://zhuanlan.zhihu.com/p/419931729) | [静态站点](https://zhuanlan.zhihu.com/p/418991797) |
- 基础工具 >> [双向链表](https://zhuanlan.zhihu.com/p/418766952) | [位图管理](https://zhuanlan.zhihu.com/p/419932963) | [用栈方式](https://zhuanlan.zhihu.com/p/419933219) | [定时器](https://zhuanlan.zhihu.com/p/418991797) | [原子操作](https://zhuanlan.zhihu.com/p/418991797) | [时间管理](https://zhuanlan.zhihu.com/p/418991797) |
- 加载运行 >> [ELF格式](https://zhuanlan.zhihu.com/p/418991797) | [ELF解析](https://zhuanlan.zhihu.com/p/418991797) | [静态链接](https://zhuanlan.zhihu.com/p/418991797) | [重定位](https://zhuanlan.zhihu.com/p/418991797) | [进程映像](https://zhuanlan.zhihu.com/p/418991797) |
- 进程管理 >> [进程管理](https://zhuanlan.zhihu.com/p/418943425) | [进程概念](https://zhuanlan.zhihu.com/p/419996612) | [Fork](https://zhuanlan.zhihu.com/p/418991797) | [特殊进程](https://zhuanlan.zhihu.com/p/418991797) | [进程回收](https://zhuanlan.zhihu.com/p/418991797) | [信号生产](https://zhuanlan.zhihu.com/p/418991797) | [信号消费](https://zhuanlan.zhihu.com/p/418991797) | [Shell编辑](https://zhuanlan.zhihu.com/p/418991797) | [Shell解析](https://zhuanlan.zhihu.com/p/418991797) |
- 编译构建 >> [编译环境](https://zhuanlan.zhihu.com/p/418991797) | [编译过程](https://zhuanlan.zhihu.com/p/418991797) | [环境脚本](https://zhuanlan.zhihu.com/p/418991797) | [构建工具](https://zhuanlan.zhihu.com/p/418991797) | [gn应用](https://zhuanlan.zhihu.com/p/418991797) | [忍者ninja](https://zhuanlan.zhihu.com/p/418991797) |
- 进程通讯 >> [自旋锁](https://zhuanlan.zhihu.com/p/419996832) | [互斥锁](https://zhuanlan.zhihu.com/p/419996896) | [进程通讯](https://zhuanlan.zhihu.com/p/419997190) | [信号量](https://zhuanlan.zhihu.com/p/419997238) | [事件控制](https://zhuanlan.zhihu.com/p/419997266) | [消息队列](https://zhuanlan.zhihu.com/p/420203967) |
- 内存管理 >> [内存分配](https://zhuanlan.zhihu.com/p/419012859) | [内存管理](https://zhuanlan.zhihu.com/p/419013610) | [内存汇编](https://zhuanlan.zhihu.com/p/419015360) | [内存映射](https://zhuanlan.zhihu.com/p/419016858) | [内存规则](https://zhuanlan.zhihu.com/p/419018875) | [物理内存](https://zhuanlan.zhihu.com/p/419024737) |
- 任务管理 >> [时钟任务](https://zhuanlan.zhihu.com/p/418985180) | [任务调度](https://zhuanlan.zhihu.com/p/418985797) | [任务管理](https://zhuanlan.zhihu.com/p/418986396) | [调度队列](https://zhuanlan.zhihu.com/p/418990908) | [调度机制](https://zhuanlan.zhihu.com/p/418991375) | [线程概念](https://zhuanlan.zhihu.com/p/419933484) | [并发并行](https://zhuanlan.zhihu.com/p/419996671) | [CPU](https://zhuanlan.zhihu.com/p/420181108) | [系统调用](https://zhuanlan.zhihu.com/p/420407155) | [任务切换](https://zhuanlan.zhihu.com/p/418991797) |
- 文件系统 >> [文件概念](https://zhuanlan.zhihu.com/p/418991797) | [文件系统](https://zhuanlan.zhihu.com/p/418991797) | [索引节点](https://zhuanlan.zhihu.com/p/418991797) | [挂载目录](https://zhuanlan.zhihu.com/p/418991797) | [根文件系统](https://zhuanlan.zhihu.com/p/418991797) | [字符设备](https://zhuanlan.zhihu.com/p/418991797) | [VFS](https://zhuanlan.zhihu.com/p/418991797) | [文件句柄](https://zhuanlan.zhihu.com/p/418991797) | [管道文件](https://zhuanlan.zhihu.com/p/418991797) |
- 硬件架构 >> [汇编基础](https://zhuanlan.zhihu.com/p/419996530) | [汇编传参](https://zhuanlan.zhihu.com/p/419996585) | [工作模式](https://zhuanlan.zhihu.com/p/420407093) | [寄存器](https://zhuanlan.zhihu.com/p/420407203) | [异常接管](https://zhuanlan.zhihu.com/p/420407271) | [汇编汇总](https://zhuanlan.zhihu.com/p/418991797) | [中断切换](https://zhuanlan.zhihu.com/p/418991797) | [中断概念](https://zhuanlan.zhihu.com/p/418991797) | [中断管理](https://zhuanlan.zhihu.com/p/418991797) |

### **百万汉字注解.精读内核源码**

四大码仓同步注解内核源码 | Gitee仓 >> [https://gitee.com/weharmony/kernel_liteos_a_note](https://link.zhihu.com/?target=https%3A//gitee.com/weharmony/kernel_liteos_a_note)

鸿蒙研究站( weharmonyos ) | 每天死磕一点点，原创不易，欢迎转载，请注明出处。若能支持点赞则更佳，感谢每一份支持。