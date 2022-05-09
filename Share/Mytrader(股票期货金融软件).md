####  介绍

欢迎使用`mytrader`开源量化分析交易平台，我们致力于为量化交易、算法交易、程序化交易以及技术分析爱好者打造最极致的行情分析交易平台。

`mytrader`是一款基于[ZQDB](https://gitee.com/7thTool/zqdb/)构建的量化分析交易平台。

`mytrader`官网：[www.mytrader.org.cn](https://gitee.com/link?target=https%3A%2F%2Fwww.mytrader.org.cn%2F)

`mytrader`[下载安装](https://gitee.com/7thTool/mytrader/attach_files/948252/download/Setup.exe)(官方版本，无需搭建繁琐的开发环境，直接安装使用)

#### `mytrader`技术特性

##### 可视 行情/分析/交易/算法

`mytrader`强大的数据展示和管理能力让我们变得与众不同。

1. 您可以实时浏览和管理行情快照/Tick/K线/指标/筛选/排序/策略/算法以及交易数据。
2. 您还可以基于我们强大的可视化能力，以可视可信的方式验证您的交易策略。

##### 模块化 全方位定制

`mytrader`高度模块化的设计，让我们可以满足您的全方位定制开发需求。

1. 支持三方接入定制开发。
2. 支持计算模块定制开发。
3. 支持C/C++/Python/Excel/VBA/麦语言等定制开发自有的行情/交易/策略系统。
4. 支持GUI界面定制。

##### 无服务 实时推送/本地存储

`mytrader`全市场实时行情和交易数据推送，让我们可以在本地环境实时计算存储行情和交易数据，无需中间服务器处理。

1. 没有中间环节，您的交易策略总是快人一步。
2. 没有中间环节，您的数据全都存储在本地，这样您还可以收盘后执行全市场级别的复盘和模拟回测。

##### 可编程 C/C++/Python/Excel/VBA/麦语言

`mytrader`强大的计算引擎支持您使用C/C++/Python/Excel/VBA/麦语言等编写和调用指标/筛选/排序/策略/算法。

1. 指标：即您可以编写类似MA、MACD、KDJ等主图、辅图指标。
2. 筛选：即您可以编写选股算法，筛选出心仪的代码。
3. 排序：即您可以编写类似涨跌幅、涨跌速等排序算法。
4. 脚本：即您可以编写快速执行算法，以便人工盯盘时抓住时机。
5. 策略/算法：即您可以编写后台策略/算法，以实现策略/算法交易。

##### 多窗口/多策略

`mytrader`多窗口/多策略设计让您可以同时处理更多的事务。

1. 您可以同时打开多个行情交易界面，并将其拖放到不同的屏幕上，以实现跨屏分析交易。
2. 你可以同时运行多个策略/算法，以实现多策略/算法分析交易。

##### 安全 算法全部本地存储

`mytrader`将您的算法的存储和计算都放在本地，您的算法永远只属于您。

#### 如何使用`mytrader`

##### 人工盯盘

人工筛选->人工盯盘->人工交易

1. 选择交易范围，通过`mytrader`的筛选功能，锁定交易范围
2. 使用实时排序功能和技术分析画面实时盯盘
3. 手动交易或者自动化脚本交易

##### 自动化交易

人工筛选->策略盯盘->策略交易

1. 选择交易范围，通过`mytrader`的筛选功能，锁定交易范围
2. 通过`mytrader`的公式系统选择策略并运行
3. 策略自动化交易
4. `mytrader`点击策略可以查看策略实时运行状态

##### 算法编写/管理/回测

编写策略/管理策略/策略回测

1. 编写策略，通过`mytrader`或者三方编辑器编写算法
2. 管理策略，用户编写的Python策略默认存储在`mytrader`的`pycalc/src`目录，`mytrader`启动时会自动加载pycalc下的所有算法，用户可以使用`mytrader`查看算法、修改参数、运行算法等。
3. 策略回测，用户可以使用`mytrader`执行回测验证

#### 软件架构

![img](https://gitee.com/7thTool/mytrader/raw/master/assets/core.png)

#### 构建工具

1. Windows下相关依赖库都是基于VS2015下编译构建的，故自行构建需要使用VS2015或者更高版本，toolset=msvc-14.0

#### 构建依赖

1. [ZQDB](https://gitee.com/7thTool/zqdb)
2. `Python3.7`(ZQDB已有)，您也可以使用Anaconda
3. [wxWidgets3.1.5](https://gitee.com/link?target=https%3A%2F%2Fwww.wxwidgets.org),wxWidgets需编译成静态库，运行时选择MT/MTD
4. `protobuf`(ZQDB已有)
5. [boost](https://gitee.com/link?target=https%3A%2F%2Fwww.boost.org%2F)
6. `XUtil`(ZQDB子模块已有)([https://github.com/7thTool/XUtil.git](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2F7thTool%2FXUtil.git))
7. [CMake](https://gitee.com/link?target=https%3A%2F%2Fwww.cmake.org%2F)

#### 构建说明

1. 下载安装依赖
2. 下载`mytrader`：git clone https://gitee.com/7thTool/mytrader
3. 使用CMake gui构建，增加定义项：`CMAKE_PREFIX_PATH=/path/boost;/path/zqdb/3rd/x64-windows-static;/path/wxWidgets-3.1.5`，即增加三个依赖项（boost，zqdb自带的三方库，wxWidgets）的查找路径 ![img](https://gitee.com/7thTool/mytrader/raw/master/assets/cmake-gui.png)

#### 使用说明

1. `mymodule`是支持三方模块的封装
2. `myctp`是基于`mymodule`的`ctp`模块封装
3. `mytrader`目前支持`ctp`模块，未来会支持更多三方模块