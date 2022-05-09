# C 语言教程

![c-mini-logo](https://www.runoob.com/wp-content/uploads/2014/09/c-mini-logo.png)

C 语言是一种通用的、面向过程式的计算机程序设计语言。1972 年，为了移植与开发 UNIX 操作系统，丹尼斯·里奇在贝尔电话实验室设计开发了 C 语言。

C 语言是一种广泛使用的计算机语言，它与 Java 编程语言一样普及，二者在现代软件程序员之间都得到广泛使用。

当前最新的C语言标准为 C11 ，在它之前的C语言标准为 C99。

**[现在开始学习 C 编程！](https://www.runoob.com/cprogramming/c-intro.html)**

[C 在线工具](https://www.runoob.com/try/showc.php?filename=helloworld&language=c)

## 谁适合阅读本教程？

本教程是专门为需要从零开始了解 C 语言的软件程序员打造的。本教程将让您对 C 语言有足够的认识，从而提升您自己的专业知识水平。

## 阅读本教程前，您需要了解的知识：

在开始学习本教程之前，您需要对计算机编程术语有基本的了解。对任何一种编程语言有基本的了解将有助于您理解 C 语言编程概念，并有助于加快您的学习进度。

## 编译/执行 C 程序

## 实例

\#include <stdio.h>  int main() {    /* 我的第一个 C 程序 */    printf("Hello, World! \n");     return 0; }


[运行实例 »](https://www.runoob.com/try/showc.php?filename=helloworld&language=c)

**实例解析：**

- 所有的 C 语言程序都需要包含 **main()** 函数。 代码从 **main()** 函数开始执行。
- **/\* ... \*/** 用于注释说明。
- **printf()** 用于格式化输出到屏幕。**printf()** 函数在 **"stdio.h"** 头文件中声明。
- **stdio.h** 是一个头文件 (标准输入输出头文件) , **#include** 是一个预处理命令，用来引入头文件。 当编译器遇到 **printf()** 函数时，如果没有找到 **stdio.h** 头文件，会发生编译错误。
- **return 0;** 语句用于表示退出程序。

运行 **C** 程序，**main(int argc, char \*argv[])** 函数传参，**argc** 为参数个数，**argv** 是字符串数组， 下标从 **0** 开始，第一个存放的是可执行程序的文件名字，然后依次存放传入的参数，举个例子 **HelloWorld.c** ：

```
#include <stdio.h> 

int main(int argc, char *argv[])
{ 
    /* 我的第一个 C 程序 */ 
    printf("可执行程序 %s ,参数个数为[%d], 运行输出：[%s]\n",argv[0],argc,argv[1]); 
    return 0;
}
```

编译 **gcc HelloWorld.c**，得到可执行程序 **a.out**，运行程序：

```
./a.out Hello,World!
可执行程序 ./a.out ,参数个数为[2], 运行输出：[Hello,World!]
```

当是 **int main()** 时，**main()** 的返回值是 **int** 类型，所以是 **return 0;** 现在 **C** 标准里规定 **main()** 返回值必须为 **int**，所以必须写成是 **int main()**。

```
#include <stdio.h>
int main(){
    /* 我的第一个 C 程序 */
    printf("Hello, World!\n");
    return 0;
}
```

当是 **void main()** 时，**main()** 的返回值是空，所以可以不写或者是 **return;** 但这是以前的写法了，现在很少用 **void main()** 了，也不推荐大家这么用。写出来的目的就是给大家提个醒，别犯一些低级错误。

```
#include <stdio.h>
void main(){
    /* 我的第一个 C 程序 */
    printf("Hello, World!\n");
    return ;
}
```

**warning: implicitly declaring library function 'printf' with type'int (const char \*, ...)' [-Wimplicit-function-declaration]**

以上警告一般发生在没有包含头文件 **#include ** 就使用 **printf** 的情况

```
int main(){
    printf("Hello, World!\n");
    return 0;
}
```

这句话提示的意思是：**用类型“int (const char \*,...)”隐式声明了库函数printf。**

**该提示的背景知识：**因为 printf 这一类库函数太常用了，所以编译器在编译的时候，发现源文件中并没有声明 printf 这个函数就直接使用了它，那么编译器就会为 printf 函数隐式生成一个声明。你那个编译器提示你，它为printf生成的隐式声明是：**int printf(const char \*, ...)**

**该提示后果：**通常情况下，这个提示只是一个 warning，而不是 error，编译仍然可以通过。

**解决方案：**要消除这个提示，只要加上 printf 的声明就行了。printf 函数是一个库函数，它的声明在 stdio.h 文件里。所以，你只要在源文件中 **#include ** 就可以了。

```
#include<stdio.h>  

/*
这一句话是必须要的格式
stdio 表示系统文件库, 也可以声明其它的
.h  表示头文件,因为这些文件都是放在程序各文件的开头
#include 告诉预处理器将指定头文件的内容插入到预处理器命令的相应位  导入头文件的预编译指令
<> 表示系统自带的库
也可以写成" " 表示用户自定义的库
如果写成" "并且自定义的库里面没有这个文件系统会自动查找自带的库,如果还是没有报错
*/

int main() // 程序的入口
{ //程序从这里开始运行
/*
int 表示数字格式,返回一个数字
main()主函数 表示程序的入口  一个程序有且只能有一个main函数的存在
*/
printf("hello C"); //打印一个hello C
return 0; //返回一个整数0,因为它是int类型,所以只能返回整数
}  //程序从这里结束
```

# C 简介

C 语言是一种通用的高级语言，最初是由丹尼斯·里奇在贝尔实验室为开发 UNIX 操作系统而设计的。C 语言最开始是于 1972 年在 DEC PDP-11 计算机上被首次实现。

在 1978 年，布莱恩·柯林汉（Brian Kernighan）和丹尼斯·里奇（Dennis Ritchie）制作了 C 的第一个公开可用的描述，现在被称为 K&R 标准。

UNIX 操作系统，C编译器，和几乎所有的 UNIX 应用程序都是用 C 语言编写的。由于各种原因，C 语言现在已经成为一种广泛使用的专业语言。

- 易于学习。
- 结构化语言。
- 它产生高效率的程序。
- 它可以处理底层的活动。
- 它可以在多种计算机平台上编译。

## 关于 C

- C 语言是为了编写 UNIX 操作系统而被发明的。
- C 语言是以 B 语言为基础的，B 语言大概是在 1970 年被引进的。
- C 语言标准是于 1988 年由美国国家标准协会（ANSI，全称 American National Standard Institute）制定的。
- 截至 1973 年，UNIX 操作系统完全使用 C 语言编写。
- 目前，C 语言是最广泛使用的系统程序设计语言。
- 大多数先进的软件都是使用 C 语言实现的。
- 当今最流行的 Linux 操作系统和 RDBMS（Relational Database Management System：关系数据库管理系统） MySQL 都是使用 C 语言编写的。

## 为什么要使用 C？

C 语言最初是用于系统开发工作，特别是组成操作系统的程序。由于 C 语言所产生的代码运行速度与汇编语言编写的代码运行速度几乎一样，所以采用 C 语言作为系统开发语言。下面列举几个使用 C 的实例：

- 操作系统
- 语言编译器
- 汇编器
- 文本编辑器
- 打印机
- 网络驱动器
- 现代程序
- 数据库
- 语言解释器
- 实体工具

## C 程序

一个 C 语言程序，可以是 3 行，也可以是数百万行，它可以写在一个或多个扩展名为 **".c"** 的文本文件中，例如，*hello.c*。您可以使用 **"vi"**、**"vim"** 或任何其他文本编辑器来编写您的 C 语言程序。

本教程假定您已经知道如何编辑一个文本文件，以及如何在程序文件中编写源代码。

------

## C11

C11（也被称为C1X）指ISO标准ISO/IEC 9899:2011，是当前最新的C语言标准。在它之前的C语言标准为C99。

### 新特性

- 对齐处理（Alignment）的标准化（包括_Alignas标志符，alignof运算符，aligned_alloc函数以及<stdalign.h>头文件）。
- _Noreturn 函数标记，类似于 gcc 的 __attribute__((noreturn))。
- _Generic 关键字。
- 多线程（Multithreading）支持，包括：
  _Thread_local存储类型标识符，<threads.h>头文件，里面包含了线程的创建和管理函数。
  _Atomic类型修饰符和<stdatomic.h>头文件。
- 增强的Unicode的支持。基于C Unicode技术报告ISO/IEC TR 19769:2004，增强了对Unicode的支持。包括为UTF-16/UTF-32编码增加了char16_t和char32_t数据类型，提供了包含unicode字符串转换函数的头文件<uchar.h>。
- 删除了 gets() 函数，使用一个新的更安全的函数gets_s()替代。
- 增加了边界检查函数接口，定义了新的安全的函数，例如 fopen_s()，strcat_s() 等等。
- 增加了更多浮点处理宏(宏)。
- 匿名结构体/联合体支持。这个在gcc早已存在，C11将其引入标准。
- 静态断言（Static assertions），_Static_assert()，在解释 #if 和 #error 之后被处理。
- 新的 fopen() 模式，("…x")。类似 POSIX 中的 O_CREAT|O_EXCL，在文件锁中比较常用。
- 新增 quick_exit() 函数作为第三种终止程序的方式。当 exit()失败时可以做最少的清理工作。

# C 环境设置

## 本地环境设置

如果您想要设置 C 语言环境，您需要确保电脑上有以下两款可用的软件，文本编辑器和 C 编译器。

## 文本编辑器

这将用于输入您的程序。文本编辑器包括 Windows Notepad、OS Edit command、Brief、Epsilon、EMACS 和 vim/vi。

文本编辑器的名称和版本在不同的操作系统上可能会有所不同。例如，Notepad 通常用于 Windows 操作系统上，vim/vi 可用于 Windows 和 Linux/UNIX 操作系统上。

通过编辑器创建的文件通常称为源文件，源文件包含程序源代码。C 程序的源文件通常使用扩展名 "**.c**"。

在开始编程之前，请确保您有一个文本编辑器，且有足够的经验来编写一个计算机程序，然后把它保存在一个文件中，编译并执行它。

## C 编译器

写在源文件中的源代码是人类可读的源。它需要"编译"，转为机器语言，这样 CPU 可以按给定指令执行程序。

C 语言编译器用于把源代码编译成最终的可执行程序。这里假设您已经对编程语言编译器有基本的了解了。

最常用的免费可用的编译器是 GNU 的 C/C++ 编译器，如果您使用的是 HP 或 Solaris，则可以使用各自操作系统上的编译器。

以下部分将指导您如何在不同的操作系统上安装 GNU 的 C/C++ 编译器。这里同时提到 C/C++，主要是因为 GNU 的 gcc 编译器适合于 C 和 C++ 编程语言。

### UNIX/Linux 上的安装

如果您使用的是 **Linux 或 UNIX**，请在命令行使用下面的命令来检查您的系统上是否安装了 GCC：

```
$ gcc -v
```

如果您的计算机上已经安装了 GNU 编译器，则会显示如下消息：

```
Using built-in specs.
Target: i386-redhat-linux
Configured with: ../configure --prefix=/usr .......
Thread model: posix
gcc version 4.1.2 20080704 (Red Hat 4.1.2-46)
```

如果未安装 GCC，那么请按照 http://gcc.gnu.org/install/ 上的详细说明安装 GCC。

本教程是基于 Linux 编写的，所有给定的实例都已在 Cent OS Linux 系统上编译过。

### Mac OS 上的安装

如果您使用的是 Mac OS X，最快捷的获取 GCC 的方法是从苹果的网站上下载 Xcode 开发环境，并按照安装说明进行安装。一旦安装上 Xcode，您就能使用 GNU 编译器。

Xcode 目前可从 [developer.apple.com/technologies/tools/](http://developer.apple.com/technologies/tools/) 上下载。

### Windows 上的安装

为了在 Windows 上安装 GCC，您需要安装 MinGW。为了安装 MinGW，请访问 MinGW 的主页 [www.mingw.org](http://www.mingw.org/)，进入 MinGW 下载页面，下载最新版本的 MinGW 安装程序，命名格式为 MinGW-<version>.exe。

当安装 MinWG 时，您至少要安装 gcc-core、gcc-g++、binutils 和 MinGW runtime，但是一般情况下都会安装更多其他的项。

添加您安装的 MinGW 的 bin 子目录到您的 **PATH** 环境变量中，这样您就可以在命令行中通过简单的名称来指定这些工具。

当完成安装时，您可以从 Windows 命令行上运行 gcc、g++、ar、ranlib、dlltool 和其他一些 GNU 工具。

## Windows 环境下使用 GCC

MinGw 是 Minimal GNU on Windows 的缩写，允许在 GNU/Linux 和 Windows 平台生成本地的 Windows 程序而不需要第三方运行时库。本文主要介绍 MinGw 的安装和使用。

**(一)安装**

-  1.下载 min-gw 安装程序，链接为：http://sourceforge.net/projects/mingw/files/，下载 mingw-get-setup.exe (86.5 kB)
-  2.运行 mingw-get-setup.exe (86.5 kB) ,点击“运行”，continue等，注意记住安装的目录，如 **C:\MinGw**,下面修改环境变量时还会用到。
-  3.修改环境变量: 选择计算机—属性---高级系统设置---环境变量，在系统变量中找到 Path 变量，在后面加入 min-gw的安装目录，如 **C:\MinGw\bin**
- 4.在开始菜单中，点击“运行”，输入 **cmd**,打开命令行:输入 **mingw-get.exe**,如果弹出 MinGw installation manager 窗口，说明安装正常。此时，关闭 MinGw installation manager 窗口，否则接下来的步骤会报错
- 5.在cmd中输入命令 **mingw-get install gcc**,等待一会，gcc 就安装成功了。

如果想安装 g++,gdb,只要输入命令 **mingw-get install g++** 和 **mingw-get install gdb**

**(二)使用**

在 cmd 的当前工作目录写 C 程序 test.c：

```
# include <stdio.h>
int main()
{
    printf("%s\n","hello world");
    return 0;
}
```

在 cmd 中输入命令 **gcc test.c**

在当前目录下会生成 a.exe 的可执行文件，在 cmd 中输入 a.exe 就可以执行程序了。

如果想调试程序，可以输入 gdb a.exe

进入 gdb 的功能，使用 gdb 常用的命令就可以调试程序了。

Windows 环境变量的设置：

（1）将刚刚下载好的文件，解压到C盘根目录下，文件夹名称 MinGw；

（2）计算机——>(右键)属性——>高级系统设置——>环境变量——>系统变量，选中Path点击编辑，将MicGw文件下的bin目录路径复制出来，我这里是

C:\MinGW\bin，将路径复制到 Path 中，点击确定；注意点目录前后的分号，一定要有并且必须是英文半角。

（3）同上，新建一个系统变量 lib，对应 MicGw 下的lib 文件夹；新建一个系统变量 include，对应 MicGw 下的 include 文件夹；

到此为止，我们就算是搭建好C语言开发的基本环境了；

**安卓使用gcc**

c4droid是款Android设备上的C/C++程序编译器，默认以tcc(tiny c compiler)为编译器，可以选择安装gcc插件,选用gcc后，可以用sdl（简单直控媒体层库，需安装sdl plugin for c4droid）和qt（nokia官方开发库,需安装sdl plugin for c4droid）。也可以开发native android app（需安装sdl plugin for c4droid），就像google ndk一样。软件支持代码高亮，编译时间随cpu主频而定，主频越高编译越快。gcc插件版本5.00提供了示列程序，包含sdl，android native，qt和命令行测试程序等源码。

百度云链接：C4droid5.96汉化

https://pan.baidu.com/s/1kVn5x2Z

首先查看自己的手机类型

下载对应的软件

下载后

-  1.先安装C4droid和GCC、SDL开头的两个插件，注意选择汉化的版本哦
-  2、三个全部安装完成后，打开C4DROID，会有提示安装GCC插件，全选就可以
-  3.等待解压完成，打开设置切换编译器为gcc就可以了

安卓下安装GCC

Termux 下安装gcc。

termux是 可以在Android操作系统中模拟Linux环境的终端应用程序，可以直接安装在无root权限的安卓环境下。自动地安装了一个最小的基本系统——可以使用类似Debian系统阵营中的APT包管理器提供额外的软件包。

在Termux下可以直接安装 gcc 、Vim、nano、Python、w3m 甚至是 php + composer

在Termux中可以开启SSHD使用ssh客户端远程连接（包括在安卓本地使用类似ConnectBOT管理工具连接），可以做到和在类Unix系统中同样的使用gcc进行c语言文件的编译。

Termux官网 ： https://termux.com/

**gcc 进行 c 语言编译分为四个步骤：**

1.预处理，生成预编译文件（.i 文件）：

```
gcc –E hello.c –o hello.i
```

2.编译，生成汇编代码（.s 文件）：

```
gcc –S hello.i –o hello.s
```

3.汇编，生成目标文件（.o 文件）：

```
gcc –c hello.s –o hello.o
```

4.链接，生成可执行文件：

```
gcc hello.o –o hello
```

有时候，进行调试，可能会用到某个步骤哦

Win10下使用 vscode 编译 c 语言，安装好 MinGW 后，在里面找到 mingw32-gcc.bin, mingw32-gcc-g++.bin, 以及 mingw32-gdb.bin 第一个是 c 语言文件的编译器，第二个是 c++ 的，第三个是用来调试编译后文件的。然后设置好环境变量，编写好 .c 文件，在 vscode 中打开 .c 文件所在的文件夹（注意是文件夹），然后配置 launch.json 文件如下所示：

```
{

    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}/${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceRoot}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "E:\\MinGW\\bin\\gdb.exe",
            "preLaunchTask": "gcc",//tasks.json里面的名字
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
            
        }
    ]
}
```

tasks.json文件如下所示：

```
{

    "version": "2.0.0",
    "tasks": [
        {
            "label": "gcc",
            "type": "shell",
            "command": "gcc",
            "args": [
                "-g", "${file}","-o","${fileBasenameNoExtension}.exe"
            ],
            "group":{
                "kind": "build",
                "isDefault": true
            }
        }

    ]
}
```

# C 程序结构

在我们学习 C 语言的基本构建块之前，让我们先来看看一个最小的 C 程序结构，在接下来的章节中可以以此作为参考。

## C Hello World 实例

C 程序主要包括以下部分：

- 预处理器指令
- 函数
- 变量
- 语句 & 表达式
- 注释

让我们看一段简单的代码，可以输出单词 "Hello World"：

## 实例

\#include <stdio.h>  int main() {   /* 我的第一个 C 程序 */   printf("Hello, World! \n");      return 0; }

接下来我们讲解一下上面这段程序：

1. 程序的第一行 *#include * 是预处理器指令，告诉 C 编译器在实际编译之前要包含 stdio.h 文件。
2. 下一行 *int main()* 是主函数，程序从这里开始执行。
3. 下一行 /*...*/ 将会被编译器忽略，这里放置程序的注释内容。它们被称为程序的注释。
4. 下一行 *printf(...)* 是 C 中另一个可用的函数，会在屏幕上显示消息 "Hello, World!"。
5. 下一行 **return 0;** 终止 main() 函数，并返回值 0。

## 编译 & 执行 C 程序

接下来让我们看看如何把源代码保存在一个文件中，以及如何编译并运行它。下面是简单的步骤：

1. 打开一个文本编辑器，添加上述代码。
2. 保存文件为 *hello.c*。
3. 打开命令提示符，进入到保存文件所在的目录。
4. 键入 *gcc hello.c*，输入回车，编译代码。
5. 如果代码中没有错误，命令提示符会跳到下一行，并生成 *a.out* 可执行文件。
6. 现在，键入 *a.out* 来执行程序。
7. 您可以看到屏幕上显示 *"Hello World"*。

```
$ gcc hello.c
$ ./a.out
Hello, World!
```

请确保您的路径中已包含 gcc 编译器，并确保在包含源文件 hello.c 的目录中运行它。

如果是多个 c 代码的源码文件，编译方法如下：

```
$ gcc test1.c test2.c -o main.out
$ ./main.out
```

test1.c 与 test2.c 是两个源代码文件。

**gcc** 命令如果不指定目标文件名时默认生成的可执行文件名为 **a.out(linux)** 或 **a.exe(windows)**。

可用 **gcc [源文件名] -o [目标文件名]** 来指定目标文件路径及文件名。

例如，**windows** 系统上，**gcc hello.c -o target/hello** 会在 **target** 目录下生成 **hello.exe** 文件(Linux 系统生成 hello 可执行文件)，**target** 目录必须已存在，**[源文件名] 和 -o [目标文件名]** 的顺序可互换， **gcc -o target/hello hello.c** 依然有效。

因编译器的原因，生成的 **.exe** 文件打开时会一闪而过，从而观察不到其运行的结果，这是因为 **main()** 函数结束时，DOS 窗口会自动关闭。为了避免这个问题可在 **return 0;** 前加入 **system("pause");** 语句。

```
#include <stdio.h>
#include <stdlib.h> 
int main()
{
   /* 我的第一个 C 程序 */
   printf("Hello, World! \n");
   system("pause");      //暂停函数，请按任意键继续...
   return 0;
}
```

使用 **gcc hello.c -o hello** 命令时，可不添加目标路径，则 **gcc** 即在当前工作目录自动生成 **hello.exe** 文件。

**"引用(引用自楼上@军): 因编译器的原因，生成的 \*.exe 文件打开时会一闪而过，从而观察不到其运行的结果，这是因为 main() 函数结束时，DOS 窗口会自动关闭"，**如果不想使用 system("pause")函数，可以直接使用cmd运行编译的可执行文件：

# C 基本语法

我们已经看过 C 程序的基本结构，这将有助于我们理解 C 语言的其他基本的构建块。

## C 的令牌（Tokens）

C 程序由各种令牌组成，令牌可以是关键字、标识符、常量、字符串值，或者是一个符号。例如，下面的 C 语句包括五个令牌：

```
printf("Hello, World! \n");
```

这五个令牌分别是：

```
printf
(
"Hello, World! \n"
)
;
```

## 分号 ;

在 C 程序中，分号是语句结束符。也就是说，每个语句必须以分号结束。它表明一个逻辑实体的结束。

例如，下面是两个不同的语句：

```
printf("Hello, World! \n");
return 0;
```

## 注释



**C 语言有两种注释方式：**



```
// 单行注释
```

以 **//** 开始的单行注释，这种注释可以单独占一行。

```
/* 单行注释 */
/* 
 多行注释
 多行注释
 多行注释
 */
```

**/\* \*/** 这种格式的注释可以单行或多行。

您不能在注释内嵌套注释，注释也不能出现在字符串或字符值中。

## 标识符

C 标识符是用来标识变量、函数，或任何其他用户自定义项目的名称。一个标识符以字母 A-Z 或 a-z 或下划线 _ 开始，后跟零个或多个字母、下划线和数字（0-9）。

C 标识符内不允许出现标点字符，比如 @、$ 和 %。C 是**区分大小写**的编程语言。因此，在 C 中，*Manpower* 和 *manpower* 是两个不同的标识符。下面列出几个有效的标识符：

```
mohd       zara    abc   move_name  a_123
myname50   _temp   j     a23b9      retVal
```

## 关键字

下表列出了 C 中的保留字。这些保留字不能作为常量名、变量名或其他标识符名称。

| 关键字   | 说明                                             |
| :------- | :----------------------------------------------- |
| auto     | 声明自动变量                                     |
| break    | 跳出当前循环                                     |
| case     | 开关语句分支                                     |
| char     | 声明字符型变量或函数返回值类型                   |
| const    | 声明只读变量                                     |
| continue | 结束当前循环，开始下一轮循环                     |
| default  | 开关语句中的"其它"分支                           |
| do       | 循环语句的循环体                                 |
| double   | 声明双精度浮点型变量或函数返回值类型             |
| else     | 条件语句否定分支（与 if 连用）                   |
| enum     | 声明枚举类型                                     |
| extern   | 声明变量或函数是在其它文件或本文件的其他位置定义 |
| float    | 声明浮点型变量或函数返回值类型                   |
| for      | 一种循环语句                                     |
| goto     | 无条件跳转语句                                   |
| if       | 条件语句                                         |
| int      | 声明整型变量或函数                               |
| long     | 声明长整型变量或函数返回值类型                   |
| register | 声明寄存器变量                                   |
| return   | 子程序返回语句（可以带参数，也可不带参数）       |
| short    | 声明短整型变量或函数                             |
| signed   | 声明有符号类型变量或函数                         |
| sizeof   | 计算数据类型或变量长度（即所占字节数）           |
| static   | 声明静态变量                                     |
| struct   | 声明结构体类型                                   |
| switch   | 用于开关语句                                     |
| typedef  | 用以给数据类型取别名                             |
| unsigned | 声明无符号类型变量或函数                         |
| union    | 声明共用体类型                                   |
| void     | 声明函数无返回值或无参数，声明无类型指针         |
| volatile | 说明变量在程序执行中可被隐含地改变               |
| while    | 循环语句的循环条件                               |

### C99 新增关键字

| `_Bool` | `_Complex` | `_Imaginary` | `inline` | `restrict` |
| ------- | ---------- | ------------ | -------- | ---------- |
|         |            |              |          |            |

### C11 新增关键字

| `_Alignas`       | `_Alignof`      | `_Atomic` | `_Generic` | `_Noreturn` |
| ---------------- | --------------- | --------- | ---------- | ----------- |
| `_Static_assert` | `_Thread_local` |           |            |             |



## C 中的空格

只包含空格的行，被称为空白行，可能带有注释，C 编译器会完全忽略它。

在 C 中，空格用于描述空白符、制表符、换行符和注释。空格分隔语句的各个部分，让编译器能识别语句中的某个元素（比如 int）在哪里结束，下一个元素在哪里开始。因此，在下面的语句中：

```
int age;
```

在这里，int 和 age 之间必须至少有一个空格字符（通常是一个空白符），这样编译器才能够区分它们。另一方面，在下面的语句中：

```
fruit = apples + oranges;   // 获取水果的总数
```

fruit 和 =，或者 = 和 apples 之间的空格字符不是必需的，但是为了增强可读性，您可以根据需要适当增加一些空格。

**C关键字：**

按年份起始：

- **auto    break   case    char    const     continue  default   do** 

- **double  else    enum    extern  float      for       goto     if** 

- **int long register return short signed sizeof static** 

- **struct  switch   typedef  union   unsigned  void       volatile  while**

1999年12月16日，ISO推出了C99标准，该标准新增了5个C语言关键字：

- inline   restrict   _Bool  _Complex   _Imaginary

2011年12月8日，ISO发布C语言的新标准C11，该标准新增了7个C语言关键字：

- _Alignas  _Alignof  _Atomic  _Static_assert  _Noreturn  _Thread_local  _Generic  

**1、C语言的程序构成方式**

-  １）c语言程序由函数构成，每个函数可以实现一个或多个功能。
-  ２）一个正规程序可以有多个函数，但是有且只有一个主函数。
-  ３）函数只有在被调用的时候才执行，主函数由系统调用执行。
-  ４）函数的格式必须按照规范书写。
-  ５）C 语言程序文件的后缀为 .c

**2、一个简单的C程序**

在 Mac OS 系统上可以使用Xcdoe中创建 c 语言程序，打开 Xcode -- 点击 create a new xcode project -- 点击 os X-- 右边选择 commad line tool -- next -- product name 填项目名称，organization name 填公司名称，organization identifier 填公司网址的倒序。language选择 C。

简单打印一句话。

```
#include <stdio.h>

int main(int argc, const char * argv[]) {
    printf("Hello, World!\n");
    return 0;
}
```

## 关键字

**关键字：**C语言的关键字，是C语言保留的一些有特殊作用词语。一共有32个关键字，

关键字说明:

| 关键字   | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| auto     | 声明自动变量                                                 |
| double   | 声明双精度变量或函数                                         |
| typedef  | 用以给数据类型取别名                                         |
| register | 声明寄存器变量                                               |
| short    | 声明短整型变量或函数                                         |
| char     | 声明字符型变量或函数                                         |
| const    | 声明只读变量                                                 |
| static   | 声明静态变量int声明整型变量或函数                            |
| struct   | 声明结构体变量或函数                                         |
| unsigned | 声明无符号类型变量或函数                                     |
| volatile | 说明变量在程序执行中可被隐含地改变long声明长整型变量或函数   |
| union    | 声明共用数据类型                                             |
| signed   | 声明有符号类型变量或函数                                     |
| void     | 声明函数无返回值或无参数，声明无类型指针float声明浮点型变量或函数 |
| enum     | 声明枚举类型                                                 |
|          |                                                              |
| extern   | 声明变量是在其他文件正声明                                   |
| if       | 条件语句                                                     |
| else     | 条件语句否定分支（与 if 连用）                               |
| switch   | 用于开关语句                                                 |
| case     | 开关语句分支                                                 |
| for      | 一种循环语句do循环语句的循环体                               |
| while    | 循环语句的循环条件                                           |
| goto     | 无条件跳转语句                                               |
| continue | 结束当前循环，开始下一轮循环break跳出当前循环                |
| default  | 开关语句中的"其他"分支                                       |
| sizeof   | 计算数据类型长度                                             |
| return   | 子程序返回语句（可以带参数，也可不带参数）循环条件           |

## 标识符

**标识符：**在编程语言中，标识符是用户编程时使用的名字，变量、常量、函数、语句块都有名字。是用来标识某个实体的一个符号，是对变量名、函数名、标号和其他各种用户定义的对象命名。

C语言中标识符的命名规范：

-  1.标识符由字母、数字、下划线组成，并且首字母不能是数字。
-  2.不能把C的关键字作为用户的标识符，例如：if、for、while等。（注：标识符不能和C语言的关键字相同，也不能和用户自定义的函数或C语言库函数同名）
-  3.标识符长度是由机器上的编译系统决定的，一般的限制为8字符，(注：8字符长度限制是C89标准，C99标准已经扩充长度，其实大部分工业标准都更长)。
-  4.标识符对大小写敏感，即严格区分大小写。一般对变量名用小写，符号常量命名用大写。（注：C语言中字母是区分大小写的，因此score、Score、SCORE分别代表三个不同的标识符）
-  5.标识符命名应做到"见名知意"，例如，长度（外语：length），求和、总计（外语：sum），圆周率（外语：pi）

**C 语言有两种注释方式：**

-  （1）以 **//** 开始的单行注释，这种注释可以单独占一行，也可以出现在一行中其他内容的右侧，注意这种注释不能跨行。若注释内容一行内写不下可以在下一行重新用 **//** ，继续写注释。

- 

  （2）以 **/\*** 开始，以 ***/** 结束的块式注释。这种注释可以包含多行内容。编译系统在发现一个 **/\*** 后，会开始找注释结束符 ***/**，把两者间的内容作为注释。

  应注意的是在字符串中的和都不作为注释的开始。而是作为字符串的一部分。如：

  ```
  printf("//how do you do!\n");
  
  或
  
  printf("/*how do you do!*/\n);
  ```

  输出分别是：**//how do you do！**和 **/\*how do you do!\*/**

  # C 数据类型

  在 C 语言中，数据类型指的是用于声明不同类型的变量或函数的一个广泛的系统。变量的类型决定了变量存储占用的空间，以及如何解释存储的位模式。

  C 中的类型可分为以下几种：

  | 序号 | 类型与描述                                                   |
  | :--- | :----------------------------------------------------------- |
  | 1    | **基本类型：** 它们是算术类型，包括两种类型：整数类型和浮点类型。 |
  | 2    | **枚举类型：** 它们也是算术类型，被用来定义在程序中只能赋予其一定的离散整数值的变量。 |
  | 3    | **void 类型：** 类型说明符 *void* 表明没有可用的值。         |
  | 4    | **派生类型：** 它们包括：指针类型、数组类型、结构类型、共用体类型和函数类型。 |

  

  数组类型和结构类型统称为聚合类型。函数的类型指的是函数返回值的类型。在本章节接下来的部分我们将介绍基本类型，其他几种类型会在后边几个章节中进行讲解。

  ## 整数类型

  下表列出了关于标准整数类型的存储大小和值范围的细节：

  | 类型           | 存储大小    | 值范围                                               |
  | :------------- | :---------- | :--------------------------------------------------- |
  | char           | 1 字节      | -128 到 127 或 0 到 255                              |
  | unsigned char  | 1 字节      | 0 到 255                                             |
  | signed char    | 1 字节      | -128 到 127                                          |
  | int            | 2 或 4 字节 | -32,768 到 32,767 或 -2,147,483,648 到 2,147,483,647 |
  | unsigned int   | 2 或 4 字节 | 0 到 65,535 或 0 到 4,294,967,295                    |
  | short          | 2 字节      | -32,768 到 32,767                                    |
  | unsigned short | 2 字节      | 0 到 65,535                                          |
  | long           | 4 字节      | -2,147,483,648 到 2,147,483,647                      |
  | unsigned long  | 4 字节      | 0 到 4,294,967,295                                   |

  > 注意，各种类型的存储大小与系统位数有关，但目前通用的以64位系统为主。
  >
  > 以下列出了32位系统与64位系统的存储大小的差别（windows 相同）：
  >
  > ![img](https://www.runoob.com/wp-content/uploads/2014/09/32-64.jpg)

  为了得到某个类型或某个变量在特定平台上的准确大小，您可以使用 **sizeof** 运算符。表达式 *sizeof(type)* 得到对象或类型的存储字节大小。下面的实例演示了获取 int 类型的大小：

  ## 实例

  \#include <stdio.h> #include <limits.h>  int main() {   printf("int 存储大小 : %lu \n", sizeof(int));      return 0; }

  **%lu** 为 32 位无符号整数，详细说明查看 [C 库函数 - printf()](https://www.runoob.com/cprogramming/c-function-printf.html)。

  当您在 Linux 上编译并执行上面的程序时，它会产生下列结果：

  ```
  int 存储大小 : 4 
  ```

  ## 浮点类型

  下表列出了关于标准浮点类型的存储大小、值范围和精度的细节：

  | 类型        | 存储大小 | 值范围                 | 精度      |
  | :---------- | :------- | :--------------------- | :-------- |
  | float       | 4 字节   | 1.2E-38 到 3.4E+38     | 6 位小数  |
  | double      | 8 字节   | 2.3E-308 到 1.7E+308   | 15 位小数 |
  | long double | 16 字节  | 3.4E-4932 到 1.1E+4932 | 19 位小数 |

  头文件 float.h 定义了宏，在程序中可以使用这些值和其他有关实数二进制表示的细节。下面的实例将输出浮点类型占用的存储空间以及它的范围值：

  ## 实例

  \#include <stdio.h> #include <float.h>  int main() {   printf("float 存储最大字节数 : %lu \n", sizeof(float));   printf("float 最小值: %E\n", FLT_MIN );   printf("float 最大值: %E\n", FLT_MAX );   printf("精度值: %d\n", FLT_DIG );      return 0; }

  **%E** 为以指数形式输出单、双精度实数，详细说明查看 [C 库函数 - printf()](https://www.runoob.com/cprogramming/c-function-printf.html)。

  当您在 Linux 上编译并执行上面的程序时，它会产生下列结果：

  ```
  float 存储最大字节数 : 4 
  float 最小值: 1.175494E-38
  float 最大值: 3.402823E+38
  精度值: 6
  ```

  ## void 类型

  void 类型指定没有可用的值。它通常用于以下三种情况下：

  | 序号 | 类型与描述                                                   |
  | :--- | :----------------------------------------------------------- |
  | 1    | **函数返回为空** C 中有各种函数都不返回值，或者您可以说它们返回空。不返回值的函数的返回类型为空。例如 **void exit (int status);** |
  | 2    | **函数参数为空** C 中有各种函数不接受任何参数。不带参数的函数可以接受一个 void。例如 **int rand(void);** |
  | 3    | **指针指向 void** 类型为 void * 的指针代表对象的地址，而不是类型。例如，内存分配函数 **void \*malloc( size_t size );** 返回指向 void 的指针，可以转换为任何数据类型。 |

  如果现在您还是无法完全理解 void 类型，不用太担心，在后续的章节中我们将会详细讲解这些概念。

  常用基本数据类型占用空间（64位机器为例）

  -  char ： 1个字节
  -  int ：4个字节
  -  float：4个字节
  -  double：8个字节

  ## 基本类型书写

  整数

  

  -  a，默认为10进制 ，10 ，20。
  -  b，以0开头为8进制，045，021。
  -  c.，以0b开头为2进制，0b11101101。
  -  d，以0x开头为16进制，0x21458adf。

  **小数**

  单精度常量：2.3f 。

  双精度常量：2.3，默认为双精度。

  **字符型常量**

  用英文单引号括起来，只保存一个字符'a'、'b' 、'*' ，还有转义字符 '\n' 、'\t'。

  **字符串常量**

  用英文的双引号引起来 可以保存多个字符："abc"。

  1、数据类型转换：C 语言中如果一个表达式中含有不同类型的常量和变量，在计算时，会将它们自动转换为同一种类型；在 C 语言中也可以对数据类型进行强制转换；

  2、自动转换规则：

  -  a）浮点数赋给整型，该浮点数小数被舍去；
  -  b）整数赋给浮点型，数值不变，但是被存储到相应的浮点型变量中；

  3、强制类型转换形式: **(类型说明符)(表达式)**

  实例程序：

  ```
  #include<stdio.h>
  
  int main()
  {
      float f,x=3.6,y=5.2;
      int i=4,a,b;
      a=x+y;
      b=(int)(x+y);
      f=10/i;
      printf("a=%d,b=%d,f=%f,x=%f\n",a,b,f,x);
  }
  ```

  例中先计算 x+y 值为 8.8，然后赋值给 a，因为a为整型，所以自取整数部分8，a=8;

  接下来 b 把 x+y 强制转换为整型;

  最后 10/i 是两个整数相除，结果仍为整数 2，把 2 赋给浮点数 f;

  x 为浮点型直接输出。

  # C 变量

  变量其实只不过是程序可操作的存储区的名称。C 中每个变量都有特定的类型，类型决定了变量存储的大小和布局，该范围内的值都可以存储在内存中，运算符可应用于变量上。

  变量的名称可以由字母、数字和下划线字符组成。它必须以字母或下划线开头。大写字母和小写字母是不同的，因为 C 是大小写敏感的。基于前一章讲解的基本类型，有以下几种基本的变量类型：

  | 类型   | 描述                                                         |
  | :----- | :----------------------------------------------------------- |
  | char   | 通常是一个字节（八位）。这是一个整数类型。                   |
  | int    | 对机器而言，整数的最自然的大小。                             |
  | float  | 单精度浮点值。单精度是这样的格式，1位符号，8位指数，23位小数。![img](https://www.runoob.com/wp-content/uploads/2014/09/v2-749cc641eb4d5dafd085e8c23f8826aa_hd.png) |
  | double | 双精度浮点值。双精度是1位符号，11位指数，52位小数。![img](https://www.runoob.com/wp-content/uploads/2014/09/v2-48240f0e1e0dd33ec89100cbe2d30707_hd.png) |
  | void   | 表示类型的缺失。                                             |

  

  C 语言也允许定义各种其他类型的变量，比如枚举、指针、数组、结构、共用体等等，这将会在后续的章节中进行讲解，本章节我们先讲解基本变量类型。

  ## C 中的变量定义

  变量定义就是告诉编译器在何处创建变量的存储，以及如何创建变量的存储。变量定义指定一个数据类型，并包含了该类型的一个或多个变量的列表，如下所示：

  ```
  type variable_list;
  ```

  在这里，**type** 必须是一个有效的 C 数据类型，可以是 char、w_char、int、float、double 或任何用户自定义的对象，**variable_list** 可以由一个或多个标识符名称组成，多个标识符之间用逗号分隔。下面列出几个有效的声明：

  ```
  int    i, j, k;
  char   c, ch;
  float  f, salary;
  double d;
  ```

  行 **int i, j, k;** 声明并定义了变量 i、j 和 k，这指示编译器创建类型为 int 的名为 i、j、k 的变量。

  变量可以在声明的时候被初始化（指定一个初始值）。初始化器由一个等号，后跟一个常量表达式组成，如下所示：

  ```
  type variable_name = value;
  ```

  下面列举几个实例：

  ```
  extern int d = 3, f = 5;    // d 和 f 的声明与初始化
  int d = 3, f = 5;           // 定义并初始化 d 和 f
  byte z = 22;                // 定义并初始化 z
  char x = 'x';               // 变量 x 的值为 'x'
  ```

  不带初始化的定义：带有静态存储持续时间的变量会被隐式初始化为 NULL（所有字节的值都是 0），其他所有变量的初始值是未定义的。

  ## C 中的变量声明

  变量声明向编译器保证变量以指定的类型和名称存在，这样编译器在不需要知道变量完整细节的情况下也能继续进一步的编译。变量声明只在编译时有它的意义，在程序连接时编译器需要实际的变量声明。

  变量的声明有两种情况：

  - 1、一种是需要建立存储空间的。例如：int a 在声明的时候就已经建立了存储空间。
  - 2、另一种是不需要建立存储空间的，通过使用extern关键字声明变量名而不定义它。 例如：extern int a 其中变量 a 可以在别的文件中定义的。
  - 除非有extern关键字，否则都是变量的定义。

  ```
  extern int i; //声明，不是定义
  int i; //声明，也是定义
  ```

  ### 实例

  尝试下面的实例，其中，变量在头部就已经被声明，但是定义与初始化在主函数内：

  ## 实例

  \#include <stdio.h>  // 函数外定义变量 x 和 y int x; int y; int addtwonum() {    // 函数内声明变量 x 和 y 为外部变量    extern int x;    extern int y;    // 给外部变量（全局变量）x 和 y 赋值    x = 1;    y = 2;    return x+y; }  int main() {    int result;    // 调用函数 addtwonum    result = addtwonum();        printf("result 为: %d",result);    return 0; }

  当上面的代码被编译和执行时，它会产生下列结果：

  ```
  result 为: 3
  ```

  如果需要在一个源文件中引用另外一个源文件中定义的变量，我们只需在引用的文件中将变量加上 extern 关键字的声明即可。

  ## addtwonum.c 文件代码：

  \#include <stdio.h> /*外部变量声明*/ extern int x ; extern int y ; int addtwonum() {    return x+y; }

  ## test.c 文件代码：

  \#include <stdio.h>   /*定义两个全局变量*/ int x=1; int y=2; int addtwonum(); int main(void) {    int result;    result = addtwonum();    printf("result 为: %d\n",result);    return 0; }

  当上面的代码被编译和执行时，它会产生下列结果：

  ```
  $ gcc addtwonum.c test.c -o main
  $ ./main
  result 为: 3
  ```

  ## C 中的左值（Lvalues）和右值（Rvalues）

  C 中有两种类型的表达式：

  1. **左值（lvalue）：**指向内存位置的表达式被称为左值（lvalue）表达式。左值可以出现在赋值号的左边或右边。
  2. **右值（rvalue）：**术语右值（rvalue）指的是存储在内存中某些地址的数值。右值是不能对其进行赋值的表达式，也就是说，右值可以出现在赋值号的右边，但不能出现在赋值号的左边。

  变量是左值，因此可以出现在赋值号的左边。数值型的字面值是右值，因此不能被赋值，不能出现在赋值号的左边。下面是一个有效的语句：

  ```
  int g = 20;
  ```

  但是下面这个就不是一个有效的语句，会生成编译时错误：

  ```
  10 = 20;
  ```

```
extern int a;     // 声明一个全局变量 a

int a;            // 定义一个全局变量 a

extern int a =0;  // 定义一个全局变量 a 并给初值。一旦给予赋值，一定是定义，定义才会分配存储空间

int a =0;         //定义一个全局变量 a，并给初值
```

声明之后你不能直接使用这个变量，需要定义之后才能使用。

第四个等于第三个，都是定义一个可以被外部使用的全局变量，并给初值。

糊涂了吧，他们看上去可真像。但是定义只能出现在一处。也就是说，不管是 int a 还是 int a=0 都只能出现一次，而那个 extern int a 可以出现很多次。

当你要引用一个全局变量的时候，你就要声明 extern int a 这时候 extern 不能省略，因为省略了，就变成 int a 这是一个定义，不是声明。

总结：

-  1、 当需要保存数据的时候，需要lvalues。
-  2、 当需要读取数据的时候，需要rvalues。

lvalues 和 rvalues 角色的相互转换

1、 根据表达式的上下文情况，lvalues 在需要 rvalues 的地方会自动转换为 rvalues。例如:

```
int n;

int m;

m = n+2; // 这个表达式里 n 是 rvalues
```

2、 rvalues 永远不能转换为 lvalues

**变量的内存寻址(与系统有关)**

(1)内存寻址由大到小，优先分配内存地址比较大的字节给变量，所以说变量越先定义，内存地址就越大。 如下面代码，先定义变量 a,再定义变量 b,打印出 a 的地址是 0x7fff5fbff828,b 的值是 0x7fff5fbff824。a 的地址比 b 的地址大 4 字节。

(2)变量地址的获取方式：& 变量名。

(3)输出地址的方式：%p。

```
#include <stdio.h>  
  
int main()   
{  
      
    int a;  
    int b;  
  
    printf("a的地址是%p\nb的地址是%p\n",&a,&b);  
    return 0;  
     
}  
```

(4)一个变量一定要先初始化才可以使用，因为 c 语言中默认一个没有初始化的变量值是一个不可知的很大值。如下面所示，a 没有初始化，打印出 a 的值是 1606422582。

```
#include <stdio.h>  
  
int main()   
{  
      
    int a;  
  
    printf("a的值是%d\n",a);  
    return 0;  
     
}  
```

------

**Ethan,zhouyanchun16@163.com 的说明**

第一点和第四点有些小伙伴运行结果可能有差异：

**环境说明:**

-  OS：Ubuntu 16.10
-  gcc：6.2.0

实际执行结果发现，内存寻址由小到大,越先定义的值，内存地址越小。变量如果没有初始化，默认输出为 0。

-  1、 当需要保存数据的时候，需要lvalues。
-  2、 当需要读取数据的时候，需要rvalues。

**lvalues 和 rvalues 角色的相互转换**

1、 根据表达式的上下文情况，lvalues 在需要 rvalues 的地方会自动转换为 rvalues。例如:

```
int n;
int m;
m = n+2; // 这个表达式里 n 是 rvalues
```

2、 rvalues 永远不能转换为 lvalues

在 C 语言标准(C89)没有定义布尔类型，所以 C 语言判断真假时以 0 为假，非 0 为真。所以我们通常使用逻辑变量的做法：

```
//定义一个int类型变量，当变量值为0时表示false，值为1时表示true
int flag;
flag = 0;
//......
flag = 1;

if( flag )
{
//......
}
```

但这种做法不直观，而且没有明确 flag 一定是布尔值。所以我们又借助 C 语言的宏定义：

```
//宏定义布尔类型
#define BOOL int
#define TRUE 1
#define FALSE 0

//定义一个布尔变量
BOOL flag = FALSE;
```

这种方法虽然直观，但依然是换汤不换药，变量 flag 在编译器看来依然是 int 类型。

新版本总会改进一些不好的地方，所以在最新的 C 语言标准(C99)解决了布尔类型的问题。C99 提供了 _Bool 型，所以布尔类型可以声明为 _Bool flag。

_Bool 依然仍是整数类型，但与一般整型不同的是，_Bool 变量只能赋值为 0 或 1，非 0 的值都会被存储为 1。

C99还提供了一个头文件 <stdbool.h> 定义了 bool 代表 _Bool，true 代表 1，false 代表 0。只要导入 stdbool.h ，就能非常方便的操作布尔类型了。

```
//导入 stdbool.h 来使用布尔类型
#include <stdbool.h>
#include <stdio.h>

//计算n!,n的值在main中定义
int main(void)
{
    int n = 10;    //计算叠乘数
    int sum = 1; //用来存放叠乘的结果
    bool flag = false;    //叠乘标记
    
    int num = n;    //循环次数
    while( !flag )
    {
        sum = sum * (num--);
        //当num=1时结束循环
        if( num == 1)
        {
            flag = true;
        }
    }
    printf ("%d的叠乘值为 %d \n", n, sum);
    return 0;
}
```

**全局变量和局部变量在内存中的区别**

全局变量保存在内存的全局存储区中，占用静态的存储单元；局部变量保存在栈中，只有在所在函数被调用时才动态地为变量分配存储单元。

C语言经过编译之后将内存分为以下几个区域：

-  （1）栈（stack）：由编译器进行管理，自动分配和释放，存放函数调用过程中的各种参数、局部变量、返回值以及函数返回地址。操作方式类似数据结构中的栈。
-  （2）堆（heap）：用于程序动态申请分配和释放空间。C语言中的malloc和free，C++中的new和delete均是在堆中进行的。正常情况下，程序员申请的空间在使用结束后应该释放，若程序员没有释放空间，则程序结束时系统自动回收。注意：这里的“堆”并不是数据结构中的“堆”。
- （3）全局（静态）存储区：分为DATA段和BSS段。DATA段（全局初始化区）存放初始化的全局变量和静态变量；BSS段（全局未初始化区）存放未初始化的全局变量和静态变量。程序运行结束时自动释放。其中BBS段在程序执行之前会被系统自动清0，所以未初始化的全局变量和静态变量在程序执行之前已经为0。
-  （4）文字常量区：存放常量字符串。程序结束后由系统释放。
-  （5）程序代码区：存放程序的二进制代码。

显然，C语言中的全局变量和局部变量在内存中是有区别的。C语言中的全局变量包括外部变量和静态变量，均是保存在全局存储区中，占用永久性的存储单元；局部变量，即自动变量，保存在栈中，只有在所在函数被调用时才由系统动态在栈中分配临时性的存储单元。

有兴趣的读者可以运行下面的程序，分析一下运行结果。

```
#include <stdio.h>
#include <stdlib.h>
int k1 = 1;
int k2;
static int k3 = 2;
static int k4;
int main( )
{  staticint m1=2, m2;
    inti=1;
    char*p;
    charstr[10] = "hello";
    char*q = "hello";
    p= (char *)malloc( 100 );
    free(p);
    printf("栈区-变量地址  i：%p\n", &i);
    printf("                p：%p\n", &p);
    printf("              str：%p\n", str);
    printf("                q：%p\n", &q);
    printf("堆区地址-动态申请：%p\n", p);
    printf("全局外部有初值 k1：%p\n", &k1);
    printf("    外部无初值 k2：%p\n", &k2);
    printf("静态外部有初值 k3：%p\n", &k3);
    printf("    外静无初值 k4：%p\n", &k4);
    printf("  内静态有初值 m1：%p\n", &m1);
    printf("  内静态无初值 m2：%p\n", &m2);
    printf("文字常量地址    ：%p, %s\n",q, q);
    printf("程序区地址      ：%p\n",&main);
    return0;
}
```

变量应先定义再赋值，在一个表达式中，左值必须是变量，右值可以是变量，常量或者表达式。

```
#include<stdio.h>

int main()
{
    int a,b;
    a=(b=3);//注意左值 等同a=b=3,但是a=(a=b)=3是错误表示
    printf("%d\n",a);
    return 0;
}
```

**变量定义：**用于为变量分配存储空间，还可为变量指定初始值。程序中，变量有且仅有一个定义。

变量声明：用于向程序表明变量的类型和名字。

定义也是声明，extern 声明不是定义。

**定义也是声明：**当定义变量时我们声明了它的类型和名字。

**extern 声明不是定义：**通过使用 extern 关键字声明变量名而不定义它。

**[注意]**

变量在使用前就要被定义或者声明。

在一个程序中，变量只能定义一次，却可以声明多次。

定义分配存储空间，而声明不会。

# C 常量

常量是固定值，在程序执行期间不会改变。这些固定的值，又叫做**字面量**。

常量可以是任何的基本数据类型，比如整数常量、浮点常量、字符常量，或字符串字面值，也有枚举常量。

**常量**就像是常规的变量，只不过常量的值在定义后不能进行修改。

## 整数常量

整数常量可以是十进制、八进制或十六进制的常量。前缀指定基数：0x 或 0X 表示十六进制，0 表示八进制，不带前缀则默认表示十进制。

整数常量也可以带一个后缀，后缀是 U 和 L 的组合，U 表示无符号整数（unsigned），L 表示长整数（long）。后缀可以是大写，也可以是小写，U 和 L 的顺序任意。

下面列举几个整数常量的实例：

```
212         /* 合法的 */
215u        /* 合法的 */
0xFeeL      /* 合法的 */
078         /* 非法的：8 不是八进制的数字 */
032UU       /* 非法的：不能重复后缀 */
```

以下是各种类型的整数常量的实例：

```
85         /* 十进制 */
0213       /* 八进制 */
0x4b       /* 十六进制 */
30         /* 整数 */
30u        /* 无符号整数 */
30l        /* 长整数 */
30ul       /* 无符号长整数 */
```

## 浮点常量

浮点常量由整数部分、小数点、小数部分和指数部分组成。您可以使用小数形式或者指数形式来表示浮点常量。

当使用小数形式表示时，必须包含整数部分、小数部分，或同时包含两者。当使用指数形式表示时， 必须包含小数点、指数，或同时包含两者。带符号的指数是用 e 或 E 引入的。

下面列举几个浮点常量的实例：

```
3.14159       /* 合法的 */
314159E-5L    /* 合法的 */
510E          /* 非法的：不完整的指数 */
210f          /* 非法的：没有小数或指数 */
.e55          /* 非法的：缺少整数或分数 */
```

## 字符常量

字符常量是括在单引号中，例如，'x' 可以存储在 **char** 类型的简单变量中。

字符常量可以是一个普通的字符（例如 'x'）、一个转义序列（例如 '\t'），或一个通用的字符（例如 '\u02C0'）。

在 C 中，有一些特定的字符，当它们前面有反斜杠时，它们就具有特殊的含义，被用来表示如换行符（\n）或制表符（\t）等。下表列出了一些这样的转义序列码：

| 转义序列   | 含义                       |
| :--------- | :------------------------- |
| \\         | \ 字符                     |
| \'         | ' 字符                     |
| \"         | " 字符                     |
| \?         | ? 字符                     |
| \a         | 警报铃声                   |
| \b         | 退格键                     |
| \f         | 换页符                     |
| \n         | 换行符                     |
| \r         | 回车                       |
| \t         | 水平制表符                 |
| \v         | 垂直制表符                 |
| \ooo       | 一到三位的八进制数         |
| \xhh . . . | 一个或多个数字的十六进制数 |

下面的实例显示了一些转义序列字符：

## 实例

\#include <stdio.h>  int main() {   printf("Hello\tWorld\n\n");    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Hello   World
```

## 字符串常量

字符串字面值或常量是括在双引号 "" 中的。一个字符串包含类似于字符常量的字符：普通的字符、转义序列和通用的字符。

您可以使用空格做分隔符，把一个很长的字符串常量进行分行。

下面的实例显示了一些字符串常量。下面这三种形式所显示的字符串是相同的。

```
"hello, dear"

"hello, \

dear"

"hello, " "d" "ear"
```

## 定义常量

在 C 中，有两种简单的定义常量的方式：

1. 使用 **#define** 预处理器。
2. 使用 **const** 关键字。

### #define 预处理器

下面是使用 #define 预处理器定义常量的形式：

```
#define identifier value
```

具体请看下面的实例：

## 实例

\#include <stdio.h>  #define LENGTH 10    #define WIDTH  5 #define NEWLINE '\n'  int main() {    int area;       area = LENGTH * WIDTH;   printf("value of area : %d", area);   printf("%c", NEWLINE);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
value of area : 50
```

### const 关键字

您可以使用 **const** 前缀声明指定类型的常量，如下所示：

```
const type variable = value;
```

具体请看下面的实例：

## 实例

\#include <stdio.h>  int main() {   const int  LENGTH = 10;   const int  WIDTH  = 5;   const char NEWLINE = '\n';   int area;        area = LENGTH * WIDTH;   printf("value of area : %d", area);   printf("%c", NEWLINE);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
value of area : 50
```

请注意，把常量定义为大写字母形式，是一个很好的编程实践。

 **#define** 是宏定义，它不能定义常量，但宏定义可以实现在字面意义上和其它定义常量相同的功能，本质的区别就在于 **#define** 不为宏名分配内存，而 **const** 也不为常量分配内存，怎么回事呢，其实 **const** 并不是去定义一个常量，而是去改变一个变量的存储类，把该变量所占的内存变为只读！ 

反斜杠(**\**) 开头是叫转义序列(Escape Sequence)。

**\ooo** 是对用三位八进制数转义表示任意字符的形象化描述。

比如 **char ch = '\101';** 等价于 **char ch = 0101;** (以0开头的表示八进制）。

**\xhh** 里面是 x 是固定的，表示十六进制(hexadecimal)，h 也表示十六进制。

举例，**char ch = '\x41';** 就是用十六进制来表示，它与前面的 **\101** 是等价的。

可用如下代码证明它们等价：

```
#include <stdio.h>
  
int main(){
    printf("%c,%c,%c,%c", 0101, '\101', '\x41', 'A');
    return 0;
}
```

**const** 定义的是变量不是常量，只是这个变量的值不允许改变是常变量！带有类型。编译运行的时候起作用存在类型检查。

**define** 定义的是不带类型的常数，只进行简单的字符替换。在预编译的时候起作用，不存在类型检查。

## 1、两者的区别

**(1) 编译器处理方式不同**

- \#define 宏是在预处理阶段展开。
-  const 常量是编译运行阶段使用。

**(2) 类型和安全检查不同**

-  \#define 宏没有类型，不做任何类型检查，仅仅是展开。
-  const 常量有具体的类型，在编译阶段会执行类型检查。

**(3) 存储方式不同**

- \#define宏仅仅是展开，有多少地方使用，就展开多少次，不会分配内存。（宏定义不分配内存，变量定义分配内存。）
- const常量会在内存中分配(可以是堆中也可以是栈中)。

**(4) const 可以节省空间，避免不必要的内存分配。 例如：**

```
#define NUM 3.14159 //常量宏
const doulbe Num = 3.14159; //此时并未将Pi放入ROM中 ......
double i = Num; //此时为Pi分配内存，以后不再分配！
double I= NUM; //编译期间进行宏替换，分配内存
double j = Num; //没有内存分配
double J = NUM; //再进行宏替换，又一次分配内存！
```

const 定义常量从汇编的角度来看，只是给出了对应的内存地址，而不是象 #define 一样给出的是立即数，所以，const 定义的常量在程序运行过程中只有一份拷贝（因为是全局的只读变量，存在静态区），而 #define 定义的常量在内存中有若干个拷贝。

**(5) 提高了效率。 编译器通常不为普通const常量分配存储空间，而是将它们保存在符号表中，这使得它成为一个编译期间的常量，没有了存储与读内存的操作，使得它的效率也很高。**

**(6) 宏替换只作替换，不做计算，不做表达式求解;**

宏预编译时就替换了，程序运行时，并不分配内存。

define 注意“边缘效应”，例：**#define N 2+3**, N 的值是 5。

```
int a = N/2
```

在编译时我们预想 **a=2.5**，实际打印结果是 **3.5** 原因是在预处理阶段，编译器将 **a=N/2** 处理成 **a=2+3/2**，这就是 **define** 宏的边缘效应，所以我们应该写成 **#define N (2+3)**。

```
#include <stdio.h>

#define LENGTH 10+10
//正确写法 #define LENGTH (10+10)
#define WIDTH  5
#define NEWLINE '\n'
int main(){   
    int area;   
    area = LENGTH * WIDTH;   
    printf("value of area : %d", area);   
    printf("%c", NEWLINE);   
    return 0;
}
```

以上实例输出结果为：

```
value of area : 60
```

所以如果我们需要得到正确结果应该将 **#define LENGTH 10+10** 修改为 **#define LENGTH (10+10)**。

在 C 语言中，单引号与双引号是有很大区别的。

在 C 语言中没有专门的字符串类型，因此双引号内的字符串会被存储到一个数组中，这个字符串代表指向这个数组起始字符的指针；

而单引号中的内容是一个 char 类型，是一个字符，这个字符对应的是 ASCII 表中的序列值。

**四种进制说明：**

在二进制中只有 0、1 两种情况，你不会看到比 1 大的数字。二进制是逢 2 就进位，所有数字中没可能有 2 或大于 2 的数字，

在八进制中有 0、1、2、3、4、5、6、7这八种情况，你也不会看到比7大的数字。八进制是逢8就进位，所有数字中没可能有8或大于8的数字。

在十进制中有0、1、2、3、4、5、6、7、8、9这十种情况，你更不会看到比9大的数字。十进制是逢10就进位，所有数字中有0~9都有

在十六进制中有 0、1、2、3、4、5、6、7、8、9、A、B、C、D、E、F，其中 A 表示 10；B 表示 11；C 表示 12；D 表示 13；E 表示 14；F 表示 15。十六进制数字中含有 A~F 字母，它是 0~9+A~F

# C 存储类

存储类定义 C 程序中变量/函数的范围（可见性）和生命周期。这些说明符放置在它们所修饰的类型之前。下面列出 C 程序中可用的存储类：

- auto
- register
- static
- extern

## auto 存储类

**auto** 存储类是所有局部变量默认的存储类。

```
{
   int mount;
   auto int month;
}
```

上面的实例定义了两个带有相同存储类的变量，auto 只能用在函数内，即 auto 只能修饰局部变量。

## register 存储类

**register** 存储类用于定义存储在寄存器中而不是 RAM 中的局部变量。这意味着变量的最大尺寸等于寄存器的大小（通常是一个词），且不能对它应用一元的 '&' 运算符（因为它没有内存位置）。

```
{
   register int  miles;
}
```

寄存器只用于需要快速访问的变量，比如计数器。还应注意的是，定义 'register' 并不意味着变量将被存储在寄存器中，它意味着变量可能存储在寄存器中，这取决于硬件和实现的限制。

## static 存储类

**static** 存储类指示编译器在程序的生命周期内保持局部变量的存在，而不需要在每次它进入和离开作用域时进行创建和销毁。因此，使用 static 修饰局部变量可以在函数调用之间保持局部变量的值。

static 修饰符也可以应用于全局变量。当 static 修饰全局变量时，会使变量的作用域限制在声明它的文件内。

全局声明的一个 static 变量或方法可以被任何函数或方法调用，只要这些方法出现在跟 static 变量或方法同一个文件中。

以下实例演示了 static 修饰全局变量和局部变量的应用：

## 实例

\#include <stdio.h>  /* 函数声明 */ void func1(void);  static int count=10;        /* 全局变量 - static 是默认的 */  int main() {  while (count--) {      func1();  }  return 0; }  void func1(void) { /* 'thingy' 是 'func1' 的局部变量 - 只初始化一次 * 每次调用函数 'func1' 'thingy' 值不会被重置。 */                  static int thingy=5;  thingy++;  printf(" thingy 为 %d ， count 为 %d\n", thingy, count); }

实例中 count 作为全局变量可以在函数内使用，thingy 使用 static 修饰后，不会在每次调用时重置。

可能您现在还无法理解这个实例，因为我已经使用了函数和全局变量，这两个概念目前为止还没进行讲解。即使您现在不能完全理解，也没有关系，后续的章节我们会详细讲解。当上面的代码被编译和执行时，它会产生下列结果：

```
 thingy 为 6 ， count 为 9
 thingy 为 7 ， count 为 8
 thingy 为 8 ， count 为 7
 thingy 为 9 ， count 为 6
 thingy 为 10 ， count 为 5
 thingy 为 11 ， count 为 4
 thingy 为 12 ， count 为 3
 thingy 为 13 ， count 为 2
 thingy 为 14 ， count 为 1
 thingy 为 15 ， count 为 0
```

## extern 存储类

**extern** 存储类用于提供一个全局变量的引用，全局变量对所有的程序文件都是可见的。当您使用 **extern** 时，对于无法初始化的变量，会把变量名指向一个之前定义过的存储位置。

当您有多个文件且定义了一个可以在其他文件中使用的全局变量或函数时，可以在其他文件中使用 *extern* 来得到已定义的变量或函数的引用。可以这么理解，*extern* 是用来在另一个文件中声明一个全局变量或函数。

extern 修饰符通常用于当有两个或多个文件共享相同的全局变量或函数的时候，如下所示：

**第一个文件：main.c**

## 实例

\#include <stdio.h>  int count ; extern void write_extern();  int main() {   count = 5;   write_extern(); }

**第二个文件：support.c**

## 实例

\#include <stdio.h>  extern int count;  void write_extern(void) {   printf("count is %d\n", count); }

在这里，第二个文件中的 *extern* 关键字用于声明已经在第一个文件 main.c 中定义的 *count*。现在 ，编译这两个文件，如下所示：

```
 $ gcc main.c support.c
```

这会产生 **a.out** 可执行程序，当程序被执行时，它会产生下列结果：

```
count is 5
```

**auto** 是局部变量的默认存储类, 限定变量只能在函数内部使用；

**register** 代表了寄存器变量，不在内存中使用；

**static**是全局变量的默认存储类,表示变量在程序生命周期内可见；

**extern** 表示全局变量，即对程序内所有文件可见，类似于Java中的public关键字；

## C 语言中全局变量、局部变量、静态全局变量、静态局部变量的区别

**从作用域看：**

1、全局变量具有全局作用域。全局变量只需在一个源文件中定义，就可以作用于所有的源文件。当然，其他不包含全局变量定义的源文件需要用extern 关键字再次声明这个全局变量。

2、静态局部变量具有局部作用域，它只被初始化一次，自从第一次被初始化直到程序运行结束都一直存在，它和全局变量的区别在于全局变量对所有的函数都是可见的，而静态局部变量只对定义自己的函数体始终可见。

3、局部变量也只有局部作用域，它是自动对象（auto），它在程序运行期间不是一直存在，而是只在函数执行期间存在，函数的一次调用执行结束后，变量被撤销，其所占用的内存也被收回。

4、静态全局变量也具有全局作用域，它与全局变量的区别在于如果程序包含多个文件的话，它作用于定义它的文件里，不能作用到其它文件里，即被static关键字修饰过的变量具有文件作用域。这样即使两个不同的源文件都定义了相同名字的静态全局变量，它们也是不同的变量。

**从分配内存空间看：**

1、全局变量，静态局部变量，静态全局变量都在静态存储区分配空间，而局部变量在栈里分配空间

2、全局变量本身就是静态存储方式， 静态全局变量当然也是静态存储方式。这两者在存储方式上并无不同。这两者的区别虽在于非静态全局变量的作用域是整个源程序，当一个源程序由多个源文件组成时，非静态的全局变量在各个源文件中都是有效的。而静态全局变量则限制了其作用域，即只在定义该变量的源文件内有效，在同一源程序的其它源文件中不能使用它。由于静态全局变量的作用域局限于一个源文件内，只能为该源文件内的函数公用，因此可以避免在其它源文件中引起错误。

-  1)静态变量会被放在程序的静态数据存储区(全局可见)中，这样可以在下一次调用的时候还可以保持原来的赋值。这一点是它与堆栈变量和堆变量的区别。
-  2)变量用static告知编译器，自己仅仅在变量的作用范围内可见。这一点是它与全局变量的区别。

从以上分析可以看出， 把局部变量改变为静态变量后是改变了它的存储方式即改变了它的生存期。把全局变量改变为静态变量后是改变了它的作用域，限制了它的使用范围。因此static 这个说明符在不同的地方所起的作用是不同的。应予以注意。

**Tips:**

-  A.若全局变量仅在单个C文件中访问，则可以将这个变量修改为静态全局变量，以降低模块间的耦合度；
-  B.若全局变量仅由单个函数访问，则可以将这个变量改为该函数的静态局部变量，以降低模块间的耦合度；
-  C.设计和使用访问动态全局变量、静态全局变量、静态局部变量的函数时，需要考虑重入问题，因为他们都放在静态数据存储区，全局可见；
-  D.如果我们需要一个可重入的函数，那么，我们一定要避免函数中使用static变量(这样的函数被称为：带"内部存储器"功能的的函数)
-  E.函数中必须要使用static变量情况:比如当某函数的返回值为指针类型时，则必须是static的局部变量的地址作为返回值，若为auto类型，则返回为错指针。

**auto 普通局部栈变量**，是自动存储，这种对象会自动创建和销毁 ，建议这个变量要放在堆栈上面，调用函数时分配内存，函数结束时释放内存。一般隐藏auto默认为自动存储类别。我们程序都变量大多是自动变量。

实例 auto.c

```
#include <stdio.h>

int main(void)
{
    auto int i = 9; /* 声明局部变量的关键字是 auto; 因可以省略, 几乎没人使用 */   
    printf("%d\n", i);  
    getchar();
    return 0;
}
```

**Register变量**：动态和静态变量都是存放在内存中，程序中遇到该值时用控制器发指令将变量的值送到运算器中，需要存数再保存到内存中。如果频繁使用一个变量，比如一个函数体内的多次循环每次都引用该局部变量，我们则可以把局部变量的值放到CPU的寄存器中，叫寄存器变量。不需要多次到内存中存取提高效率。但是只能局部自动变量和形参可以做寄存器变量。在函数调用时占用一些寄存器，函数结束时释放。不同系统对register要求也不一样，比如对定义register变量个数，数据类型等限制，有的默认为自动变量处理。所以在程序一般也不用。

实例 register.c

```
#include <stdio.h>
#include <time.h>

#define TIME 1000000000
int m, n = TIME; /* 全局变量 */

int main(void)
{   
    time_t start, stop;
    register int a, b = TIME; /* 寄存器变量 */
    int x, y = TIME;          /* 一般变量   */

    time(&start);
    for (a = 0; a < b; a++);
    time(&stop);
    printf("寄存器变量用时: %ld 秒\n", stop - start);
    
    time(&start);
    for (x = 0; x < y; x++);
    time(&stop);
    printf("一般变量用时: %ld 秒\n", stop - start);
    
    time(&start);
    for (m = 0; m < n; m++);
    time(&stop);
    printf("全局变量用时: %ld 秒\n", stop - start);

    return 0;
}
```

输出结果：

```
寄存器变量用时: 1 秒
一般变量用时: 8 秒
全局变量用时: 9 秒
```

在 C 语言中，static 可以用来修饰局部变量，全局变量以及函数。在不同的情况下 static 的作用不尽相同。

**(1)修饰局部变量**

一般情况下，对于局部变量是存放在栈区的，并且局部变量的生命周期在该语句块执行结束时便结束了。但是如果用static进行修饰的话，该变量便存放在静态数据区，其生命周期一直持续到整个程序执行结束。但是在这里要注意的是，虽然用static对局部变量进行修饰过后，其生命周期以及存储空间发生了变化，但是其作用域并没有改变，其仍然是一个局部变量，作用域仅限于该语句块。

在用static修饰局部变量后，该变量只在初次运行时进行初始化工作，且只进行一次。

如:

```
#include<stdio.h>  
void fun()  
{   
    static int a=1; a++;   
    printf("%d\n",a);  
}  
int main(void)  
{   
    fun();   
    fun();   
    return 0;  
}  
```

程序执行结果为: 2 3

说明在第二次调用fun()函数时，a的值为2，并且没有进行初始化赋值，直接进行自增运算，所以得到的结果为3.

对于静态局部变量如果没有进行初始化的话，对于整形变量系统会自动对其赋值为0，对于字符数组，会自动赋值为'\0'.

**(2)修饰全局变量**

对于一个全局变量，它既可以在本源文件中被访问到，也可以在同一个工程的其它源文件中被访问(只需用extern进行声明即可)。

如:

```
//有file1.c  
int a=1;  
file2.c  
#include<stdio.h>  
extern int a;  
int main(void)  
{  
    printf("%d\",a);  
    return 0;  
} 
```

则执行结果为 1

但是如果在 file1.c 中把 int a=1 改为 static int a=1;

那么在file2.c是无法访问到变量a的。原因在于用static对全局变量进行修饰改变了其作用域的范围，由原来的整个工程可见变为本源文件可见。

**(3)修饰函数**

用static修饰函数的话，情况与修饰全局变量大同小异，就是改变了函数的作用域。

> 更多参考内容：[C 语言中 static 的作用](https://www.runoob.com/w3cnote/c-static-effect.html)

**static 的用法**

用来说明静态变量 ，如果是在函数的，那么其效果和全局变量类似 。但是，static定义的变量只能在当前 c 程序文件中使用，在另一个 c 代码里面 ， 即使使用 extern 关键词也不能访问这个static变量。如果实在函数内部定义的，那么这个变量只初始化一次，即使再次调用这个函数，这个static变量也不会再次初始化 ， 这个变量的取值就会一直保存着，也就是说，当你再次调用这个函数的时候，里面用到这个static变量时，就会发现，它还是上一次函数调用时的结果。

实例 static.c:

```
#include <stdio.h>

void varfunc(void)
{
    int var=0;
    static int static_var=0;
    printf("var equal: %d\n",var);
    printf("static_var equal: %d\n",static_var);
    printf("");
    var++;
    static_var++;
}

int main(int argc, char *argv[])
{
    int i;
    for(i=0;i<3;i++)
        varfunc();
    return 0;
}
```

运行结果如下：

```
var equal: 0
static_var equal: 0
var equal: 0
static_var equal: 1
var equal: 0
static_var equal: 2
```

由此可见，用static修饰的局部变量只初始化一次。

# C 运算符

运算符是一种告诉编译器执行特定的数学或逻辑操作的符号。C 语言内置了丰富的运算符，并提供了以下类型的运算符：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 位运算符
- 赋值运算符
- 杂项运算符

本章将逐一介绍算术运算符、关系运算符、逻辑运算符、位运算符、赋值运算符和其他运算符。

## 算术运算符

下表显示了 C 语言支持的所有算术运算符。假设变量 **A** 的值为 10，变量 **B** 的值为 20，则：

| 运算符 | 描述                             | 实例             |
| :----- | :------------------------------- | :--------------- |
| +      | 把两个操作数相加                 | A + B 将得到 30  |
| -      | 从第一个操作数中减去第二个操作数 | A - B 将得到 -10 |
| *      | 把两个操作数相乘                 | A * B 将得到 200 |
| /      | 分子除以分母                     | B / A 将得到 2   |
| %      | 取模运算符，整除后的余数         | B % A 将得到 0   |
| ++     | 自增运算符，整数值增加 1         | A++ 将得到 11    |
| --     | 自减运算符，整数值减少 1         | A-- 将得到 9     |

### 实例

请看下面的实例，了解 C 语言中所有可用的算术运算符：

## 实例

\#include <stdio.h>  int main() {   int a = 21;   int b = 10;   int c ;    c = a + b;   printf("Line 1 - c 的值是 %d\n", c );   c = a - b;   printf("Line 2 - c 的值是 %d\n", c );   c = a * b;   printf("Line 3 - c 的值是 %d\n", c );   c = a / b;   printf("Line 4 - c 的值是 %d\n", c );   c = a % b;   printf("Line 5 - c 的值是 %d\n", c );   c = a++;  // 赋值后再加 1 ，c 为 21，a 为 22   printf("Line 6 - c 的值是 %d\n", c );   c = a--;  // 赋值后再减 1 ，c 为 22 ，a 为 21   printf("Line 7 - c 的值是 %d\n", c );  }

当上面的代码被编译和执行时，它会产生下列结果：

```
Line 1 - c 的值是 31
Line 2 - c 的值是 11
Line 3 - c 的值是 210
Line 4 - c 的值是 2
Line 5 - c 的值是 1
Line 6 - c 的值是 21
Line 7 - c 的值是 22
```

以下实例演示了 a++ 与 ++a 的区别：

## 实例

\#include <stdio.h>  int main() {   int c;   int a = 10;   c = a++;    printf("先赋值后运算：\n");   printf("Line 1 - c 的值是 %d\n", c );   printf("Line 2 - a 的值是 %d\n", a );   a = 10;   c = a--;    printf("Line 3 - c 的值是 %d\n", c );   printf("Line 4 - a 的值是 %d\n", a );    printf("先运算后赋值：\n");   a = 10;   c = ++a;    printf("Line 5 - c 的值是 %d\n", c );   printf("Line 6 - a 的值是 %d\n", a );   a = 10;   c = --a;    printf("Line 7 - c 的值是 %d\n", c );   printf("Line 8 - a 的值是 %d\n", a );  }

以上程序执行输出结果为：

```
先赋值后运算：
Line 1 - c 的值是 10
Line 2 - a 的值是 11
Line 3 - c 的值是 10
Line 4 - a 的值是 9
先运算后赋值：
Line 5 - c 的值是 11
Line 6 - a 的值是 11
Line 7 - c 的值是 9
Line 8 - a 的值是 9
```

## 关系运算符

下表显示了 C 语言支持的所有关系运算符。假设变量 **A** 的值为 10，变量 **B** 的值为 20，则：

| 运算符 | 描述                                                         | 实例            |
| :----- | :----------------------------------------------------------- | :-------------- |
| ==     | 检查两个操作数的值是否相等，如果相等则条件为真。             | (A == B) 为假。 |
| !=     | 检查两个操作数的值是否相等，如果不相等则条件为真。           | (A != B) 为真。 |
| >      | 检查左操作数的值是否大于右操作数的值，如果是则条件为真。     | (A > B) 为假。  |
| <      | 检查左操作数的值是否小于右操作数的值，如果是则条件为真。     | (A < B) 为真。  |
| >=     | 检查左操作数的值是否大于或等于右操作数的值，如果是则条件为真。 | (A >= B) 为假。 |
| <=     | 检查左操作数的值是否小于或等于右操作数的值，如果是则条件为真。 | (A <= B) 为真。 |

### 实例

请看下面的实例，了解 C 语言中所有可用的关系运算符：

## 实例

\#include <stdio.h>  int main() {   int a = 21;   int b = 10;   int c ;    if( a == b )   {      printf("Line 1 - a 等于 b\n" );   }   else   {      printf("Line 1 - a 不等于 b\n" );   }   if ( a < b )   {      printf("Line 2 - a 小于 b\n" );   }   else   {      printf("Line 2 - a 不小于 b\n" );   }   if ( a > b )   {      printf("Line 3 - a 大于 b\n" );   }   else   {      printf("Line 3 - a 不大于 b\n" );   }   /* 改变 a 和 b 的值 */   a = 5;   b = 20;   if ( a <= b )   {      printf("Line 4 - a 小于或等于 b\n" );   }   if ( b >= a )   {      printf("Line 5 - b 大于或等于 a\n" );   } }

当上面的代码被编译和执行时，它会产生下列结果：

```
Line 1 - a 不等于 b
Line 2 - a 不小于 b
Line 3 - a 大于 b
Line 4 - a 小于或等于 b
Line 5 - b 大于或等于 a
```

## 逻辑运算符

下表显示了 C 语言支持的所有关系逻辑运算符。假设变量 **A** 的值为 1，变量 **B** 的值为 0，则：

| 运算符 | 描述                                                         | 实例              |
| :----- | :----------------------------------------------------------- | :---------------- |
| &&     | 称为逻辑与运算符。如果两个操作数都非零，则条件为真。         | (A && B) 为假。   |
| \|\|   | 称为逻辑或运算符。如果两个操作数中有任意一个非零，则条件为真。 | (A \|\| B) 为真。 |
| !      | 称为逻辑非运算符。用来逆转操作数的逻辑状态。如果条件为真则逻辑非运算符将使其为假。 | !(A && B) 为真。  |

### 实例

请看下面的实例，了解 C 语言中所有可用的逻辑运算符：

## 实例

\#include <stdio.h>  int main() {   int a = 5;   int b = 20;   int c ;    if ( a && b )   {      printf("Line 1 - 条件为真\n" );   }   if ( a || b )   {      printf("Line 2 - 条件为真\n" );   }   /* 改变 a 和 b 的值 */   a = 0;   b = 10;   if ( a && b )   {      printf("Line 3 - 条件为真\n" );   }   else   {      printf("Line 3 - 条件为假\n" );   }   if ( !(a && b) )   {      printf("Line 4 - 条件为真\n" );   } }

当上面的代码被编译和执行时，它会产生下列结果：

```
Line 1 - 条件为真
Line 2 - 条件为真
Line 3 - 条件为假
Line 4 - 条件为真
```

## 位运算符

位运算符作用于位，并逐位执行操作。&、 | 和 ^ 的真值表如下所示：

| p    | q    | p & q | p \| q | p ^ q |
| :--- | :--- | :---- | :----- | :---- |
| 0    | 0    | 0     | 0      | 0     |
| 0    | 1    | 0     | 1      | 1     |
| 1    | 1    | 1     | 1      | 0     |
| 1    | 0    | 0     | 1      | 1     |

假设如果 A = 60，且 B = 13，现在以二进制格式表示，它们如下所示：

A = 0011 1100

B = 0000 1101

\-----------------

A&B = 0000 1100

A|B = 0011 1101

A^B = 0011 0001

~A = 1100 0011

下表显示了 C 语言支持的位运算符。假设变量 **A** 的值为 60，变量 **B** 的值为 13，则：

| 运算符 | 描述                                                         | 实例                                                         |
| :----- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| &      | 按位与操作，按二进制位进行"与"运算。运算规则：`0&0=0;    0&1=0;     1&0=0;      1&1=1;` | (A & B) 将得到 12，即为 0000 1100                            |
| \|     | 按位或运算符，按二进制位进行"或"运算。运算规则：`0|0=0;    0|1=1;    1|0=1;     1|1=1;` | (A \| B) 将得到 61，即为 0011 1101                           |
| ^      | 异或运算符，按二进制位进行"异或"运算。运算规则：`0^0=0;    0^1=1;    1^0=1;   1^1=0;` | (A ^ B) 将得到 49，即为 0011 0001                            |
| ~      | 取反运算符，按二进制位进行"取反"运算。运算规则：`~1=0;    ~0=1;` | (~A ) 将得到 -61，即为 1100 0011，一个有符号二进制数的补码形式。 |
| <<     | 二进制左移运算符。将一个运算对象的各二进制位全部左移若干位（左边的二进制位丢弃，右边补0）。 | A << 2 将得到 240，即为 1111 0000                            |
| >>     | 二进制右移运算符。将一个数的各二进制位全部右移若干位，正数左补0，负数左补1，右边丢弃。 | A >> 2 将得到 15，即为 0000 1111                             |

### 实例

请看下面的实例，了解 C 语言中所有可用的位运算符：

## 实例

\#include <stdio.h>  int main() {    unsigned int a = 60;    /* 60 = 0011 1100 */     unsigned int b = 13;    /* 13 = 0000 1101 */   int c = 0;               c = a & b;       /* 12 = 0000 1100 */    printf("Line 1 - c 的值是 %d\n", c );    c = a | b;       /* 61 = 0011 1101 */   printf("Line 2 - c 的值是 %d\n", c );    c = a ^ b;       /* 49 = 0011 0001 */   printf("Line 3 - c 的值是 %d\n", c );    c = ~a;          /*-61 = 1100 0011 */   printf("Line 4 - c 的值是 %d\n", c );    c = a << 2;     /* 240 = 1111 0000 */   printf("Line 5 - c 的值是 %d\n", c );    c = a >> 2;     /* 15 = 0000 1111 */   printf("Line 6 - c 的值是 %d\n", c ); }

当上面的代码被编译和执行时，它会产生下列结果：

```
Line 1 - c 的值是 12
Line 2 - c 的值是 61
Line 3 - c 的值是 49
Line 4 - c 的值是 -61
Line 5 - c 的值是 240
Line 6 - c 的值是 15
```

## 赋值运算符

下表列出了 C 语言支持的赋值运算符：

| 运算符 | 描述                                                         | 实例                            |
| :----- | :----------------------------------------------------------- | :------------------------------ |
| =      | 简单的赋值运算符，把右边操作数的值赋给左边操作数             | C = A + B 将把 A + B 的值赋给 C |
| +=     | 加且赋值运算符，把右边操作数加上左边操作数的结果赋值给左边操作数 | C += A 相当于 C = C + A         |
| -=     | 减且赋值运算符，把左边操作数减去右边操作数的结果赋值给左边操作数 | C -= A 相当于 C = C - A         |
| *=     | 乘且赋值运算符，把右边操作数乘以左边操作数的结果赋值给左边操作数 | C *= A 相当于 C = C * A         |
| /=     | 除且赋值运算符，把左边操作数除以右边操作数的结果赋值给左边操作数 | C /= A 相当于 C = C / A         |
| %=     | 求模且赋值运算符，求两个操作数的模赋值给左边操作数           | C %= A 相当于 C = C % A         |
| <<=    | 左移且赋值运算符                                             | C <<= 2 等同于 C = C << 2       |
| >>=    | 右移且赋值运算符                                             | C >>= 2 等同于 C = C >> 2       |
| &=     | 按位与且赋值运算符                                           | C &= 2 等同于 C = C & 2         |
| ^=     | 按位异或且赋值运算符                                         | C ^= 2 等同于 C = C ^ 2         |
| \|=    | 按位或且赋值运算符                                           | C \|= 2 等同于 C = C \| 2       |

### 实例

请看下面的实例，了解 C 语言中所有可用的赋值运算符：

## 实例

\#include <stdio.h>  main() {   int a = 21;   int c ;    c =  a;   printf("Line 1 - =  运算符实例，c 的值 = %d\n", c );    c +=  a;   printf("Line 2 - += 运算符实例，c 的值 = %d\n", c );    c -=  a;   printf("Line 3 - -= 运算符实例，c 的值 = %d\n", c );    c *=  a;   printf("Line 4 - *= 运算符实例，c 的值 = %d\n", c );    c /=  a;   printf("Line 5 - /= 运算符实例，c 的值 = %d\n", c );    c  = 200;   c %=  a;   printf("Line 6 - %= 运算符实例，c 的值 = %d\n", c );    c <<=  2;   printf("Line 7 - <<= 运算符实例，c 的值 = %d\n", c );    c >>=  2;   printf("Line 8 - >>= 运算符实例，c 的值 = %d\n", c );    c &=  2;   printf("Line 9 - &= 运算符实例，c 的值 = %d\n", c );    c ^=  2;   printf("Line 10 - ^= 运算符实例，c 的值 = %d\n", c );    c |=  2;   printf("Line 11 - |= 运算符实例，c 的值 = %d\n", c );  }

当上面的代码被编译和执行时，它会产生下列结果：

```
Line 1 - =  运算符实例，c 的值 = 21
Line 2 - += 运算符实例，c 的值 = 42
Line 3 - -= 运算符实例，c 的值 = 21
Line 4 - *= 运算符实例，c 的值 = 441
Line 5 - /= 运算符实例，c 的值 = 21
Line 6 - %= 运算符实例，c 的值 = 11
Line 7 - <<= 运算符实例，c 的值 = 44
Line 8 - >>= 运算符实例，c 的值 = 11
Line 9 - &= 运算符实例，c 的值 = 2
Line 10 - ^= 运算符实例，c 的值 = 0
Line 11 - |= 运算符实例，c 的值 = 2
```

## 杂项运算符 ↦ sizeof & 三元

下表列出了 C 语言支持的其他一些重要的运算符，包括 **sizeof** 和 **? :**。

| 运算符   | 描述             | 实例                                 |
| :------- | :--------------- | :----------------------------------- |
| sizeof() | 返回变量的大小。 | sizeof(a) 将返回 4，其中 a 是整数。  |
| &        | 返回变量的地址。 | &a; 将给出变量的实际地址。           |
| *        | 指向一个变量。   | *a; 将指向一个变量。                 |
| ? :      | 条件表达式       | 如果条件为真 ? 则值为 X : 否则值为 Y |

### 实例

请看下面的实例，了解 C 语言中所有可用的杂项运算符：

## 实例

\#include <stdio.h>  int main() {   int a = 4;   short b;   double c;   int* ptr;    /* sizeof 运算符实例 */   printf("Line 1 - 变量 a 的大小 = %lu\n", sizeof(a) );   printf("Line 2 - 变量 b 的大小 = %lu\n", sizeof(b) );   printf("Line 3 - 变量 c 的大小 = %lu\n", sizeof(c) );    /* & 和 * 运算符实例 */   ptr = &a;    /* 'ptr' 现在包含 'a' 的地址 */   printf("a 的值是 %d\n", a);   printf("*ptr 是 %d\n", *ptr);    /* 三元运算符实例 */   a = 10;   b = (a == 1) ? 20: 30;   printf( "b 的值是 %d\n", b );    b = (a == 10) ? 20: 30;   printf( "b 的值是 %d\n", b ); }

当上面的代码被编译和执行时，它会产生下列结果：

```
Line 1 - 变量 a 的大小 = 4
Line 2 - 变量 b 的大小 = 2
Line 3 - 变量 c 的大小 = 8
a 的值是 4
*ptr 是 4
b 的值是 30
b 的值是 20
```

## C 中的运算符优先级

运算符的优先级确定表达式中项的组合。这会影响到一个表达式如何计算。某些运算符比其他运算符有更高的优先级，例如，乘除运算符具有比加减运算符更高的优先级。

例如 x = 7 + 3 * 2，在这里，x 被赋值为 13，而不是 20，因为运算符 * 具有比 + 更高的优先级，所以首先计算乘法 3*2，然后再加上 7。

下表将按运算符优先级从高到低列出各个运算符，具有较高优先级的运算符出现在表格的上面，具有较低优先级的运算符出现在表格的下面。在表达式中，较高优先级的运算符会优先被计算。

| 类别       | 运算符                            | 结合性   |
| :--------- | :-------------------------------- | :------- |
| 后缀       | () [] -> . ++ - -                 | 从左到右 |
| 一元       | + - ! ~ ++ - - (type)* & sizeof   | 从右到左 |
| 乘除       | * / %                             | 从左到右 |
| 加减       | + -                               | 从左到右 |
| 移位       | << >>                             | 从左到右 |
| 关系       | < <= > >=                         | 从左到右 |
| 相等       | == !=                             | 从左到右 |
| 位与 AND   | &                                 | 从左到右 |
| 位异或 XOR | ^                                 | 从左到右 |
| 位或 OR    | \|                                | 从左到右 |
| 逻辑与 AND | &&                                | 从左到右 |
| 逻辑或 OR  | \|\|                              | 从左到右 |
| 条件       | ?:                                | 从右到左 |
| 赋值       | = += -= *= /= %=>>= <<= &= ^= \|= | 从右到左 |
| 逗号       | ,                                 | 从左到右 |

### 实例

请看下面的实例，了解 C 语言中运算符的优先级：

## 实例

\#include <stdio.h>  main() {   int a = 20;   int b = 10;   int c = 15;   int d = 5;   int e;    e = (a + b) * c / d;      // ( 30 * 15 ) / 5   printf("(a + b) * c / d 的值是 %d\n",  e );    e = ((a + b) * c) / d;    // (30 * 15 ) / 5   printf("((a + b) * c) / d 的值是 %d\n" ,  e );    e = (a + b) * (c / d);   // (30) * (15/5)   printf("(a + b) * (c / d) 的值是 %d\n",  e );    e = a + (b * c) / d;     //  20 + (150/5)   printf("a + (b * c) / d 的值是 %d\n" ,  e );     return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
(a + b) * c / d 的值是 90
((a + b) * c) / d 的值是 90
(a + b) * (c / d) 的值是 90
a + (b * c) / d 的值是 50
```

利用异或 **^** 来交换两个数的值，而且不引入其他变量。

```
unsigned int a=60;  //0011 1100
unsigned int b=13;  //0000 1101
a=a^b;              //a=a^b=0011 0001
b=a^b;              //b=a^b=0011 1100  相当于b1=(a^b)^b
a=a^b;              //a=a^b=0000 1101  相当于a1=(a^b)^((a^b)^b)
```

实例

```
#include<stdio.h>

int main( )
{
    unsigned int a=60;         //0011 1100
    unsigned int b=13;         //0000 1101
    printf("a=%d,b=%d",a,b);   //输出a，b的值
    printf("\n");
    a=a^b;                     //a=a^b=0011 0001
    b=a^b;                     //b=a^b=0011 1100
    a=a^b;                     //a=a^b=0000 1101
    printf("a=%d,b=%d",a,b);   //输出a，b的值
}
```

结果：

```
a=60，b=13；
a=13，b=60；
```

利用位与 **&** 运算，判断一个整数是否是2的整数次幂。

二进制数的位权是以2为底的幂，如果一个整数 m 是 2 的 n 次幂，那么转换为二进制之后只有最高位为 1，其余位置为 0，再观察 m-1 转换为二进制后的形式以及 m&(m-1) 的结果，例如：

```
2 --> 0000 0010        1 --> 0000 0001        2&1 --> 0000 0010 & 0000 0001 = 0
4 --> 0000 0100        3 --> 0000 0011        4&3 --> 0000 0100 & 0000 0011 = 0
8 --> 0000 1000        7 --> 0000 0111        8&7 --> 0000 1000 & 0000 0111 = 0
```

可以看出所有的 1 完美的错过了，根据位与的特点可知 m&(m-1) 的结果为 0。

如果整数 m 不是 2 的 n 次幂，结果会怎样呢？例如 m=9 时：

```
9 --> 0000 1001        8 --> 0000 1000        9&8 --> 0000 1001 & 0000 1000 != 0
```

利用这一特点，即可判断一个整数是否是2的整数次幂。

示例：

```
int func(int num)
{
    return ((num > 0) && ((num & (num - 1)) == 0));//2的n次幂大于0
}
```

返回值为 1，则输入的正整数为 2 的整数次幂，返回值为 0 则不是。

**对取余运算的说明**

取余，也就是求余数，使用的运算符是 **%**。C 语言中的取余运算只能针对整数，也就是说，**%** 的两边都必须是整数，不能出现小数，否则编译器会报错。

另外，余数可以是正数也可以是负数，由 **%** 左边的整数决定：

-  如果 % 左边是正数，那么余数也是正数；
-  如果 % 左边是负数，那么余数也是负数；

请看下面的例子：

```
#include <stdio.h>

int main()
{
    printf( "100%%12=%d \n100%%-12=%d \n-100%%12=%d \n-100%%-12=%d \n", 100%12, 100%-12, -100%12, -100%-12 );
    return 0;
}
```

运行结果：

```
100%12=4
100%-12=4
-100%12=-4
-100%-12=-4 
```

对于 2 的幂指数的详细程序

```
#include <stdio.h>

int num;
int func(int num)
{
    if ((num>0)&&(num&(num-1))==0)
    {
        printf("%d是2的整数次幂",num);
    }
    else
    {
        printf("%d不是2的整数次幂",num);
    }
    return((num>0)&&(num&(num-1))==0);
}

int main()
{
    printf("请输入要查询的数\n");
    scanf("%d",&num);
    func(num);
}
```

**不同长度的数据进行位运算**

如果两个不同长度的数据进行位运算时，系统会将二者按右端对齐，然后进行位运算。

以“与”运算为例说明如下：我们知道在 C 语言中 long 型占 4 个字节，int 型占 2 个字节，如果一个 long 型数据与一个 int 型数据进行“与”运算，右端对齐后，左边不足的位依下面三种情况补足:

-  （1）如果整型数据为正数，左边补 16 个 0。
-  （2）如果整型数据为负数，左边补 16 个 1。
-  （3）如果整形数据为无符号数，左边也补 16 个 0。

如：**long a=123;int b=1;** 计算 **a & b**。

如：**long a=123;int b=-1;** 计算 **a & b**。

如：**long a=123; unsigned int b=1;** 计算 **a & b**。

**| 和 ||，& 和 && 的区别**

我们将 **||** 和 **&&** 定义为逻辑运算符，而 **|** 和 **&** 定义为位运算符。

**&&** 如果两个操作数都非零，则条件为真；

**||** 如果两个操作数中有任意一个非零，则条件为真。

**&** 按位与操作，按二进制位进行"**与**"运算。运算规则：（有 0 则为 0）

```
0&0=0;   
0&1=0;    
1&0=0;     
1&1=1;
```

**|** 按位或运算符，按二进制位进行"**或**"运算。运算规则：（有 1 则为 1）

```
0|0=0;   
0|1=1;   
1|0=1;    
1|1=1;
```

**那么，问题来了，在判断语句中，用 | 还是 ||，& 还是 &&？**

判断语句中为布尔类型，值只有 true 和 false（如果变量值为 0 就是 false，否则为 true）

```
举个例子，a=1 b=2
所以 a>0 这个值为true    b>1 这个值为true     b>2 这个值为 false
如 if(a>0&b>1)  我们可以得出 if(true&true)，条件成立（true不为0，所以true&true不为0）
如 if(a>0&&b>1)  我们可以得出 if(true&&true),条件成立（&&两边操作数都非零，所以条件成立）
如 if(b>2&a>0) 我们可以得出 if(false&true)，条件不成立（false为0，false&true为0，条件不成立）
如 if(b>2&&a>0) 我们可以得出 if(false&&a>0)，条件不成立（&&左侧为false，&&运算到此结束，条件不成立）
```

可以看出 & 和 && 在判断语句中都可以实现“和”这个功能，不过区别在于 & 两边都运算，而 && 先算 && 左侧，若左侧为 false 那么右侧就不运算了。因此从效率上来说，判断语句中推荐使用 &&（换句话就是逻辑运算就老老实实用逻辑运算符，不然它为啥叫逻辑运算符呢？）

而 | 和 || 的比较与上类似，不做赘述。

**a++ 与 ++a 区别在于一个后加，一个先加**。

a++ 输出 a 的值再自加，缓存 a 自加后的结果，用于下次进行与 a 相关的计算。

++a 则相当于 a+1。

```
#include <stdio.h>

int main(int argc, char **argv) {
    int a=100;
    int b=50;
//    b++;
    printf("%d,%d\n",a++,++a);
    printf("%d,%d",b++,++b);
    return 0;
}
```

运行后输出：

```
101,102
51,52
```

若不注释 **b++**：

```
int main(int argc, char **argv) {
    int a=100;
    int b=50;
    b++;
    printf("%d,%d\n",a++,++a);
    printf("%d,%d",b++,++b);
    return 0;
}
```

则结果为：

```
101,102
52,53
```

运算符优先级：

```
括号成员是老大;      // 括号运算符 []() 成员运算符.  ->

全体单目排老二;      // 所有的单目运算符比如++、 --、 +(正)、 -(负) 、指针运算*、&

乘除余三,加减四;    // 这个"余"是指取余运算即%

移位五，关系六;     // 移位运算符：<< >> ，关系：> < >= <= 等

等与不等排行七;     // 即 == 和 !=

位与异或和位或;     // 这几个都是位运算: 位与(&)异或(^)位或(|)    

"三分天下"八九十;  

逻辑与，逻辑或;    // 逻辑运算符: || 和 &&

十一十二紧挨着;    // 注意顺序: 优先级(||)  底于 优先级(&&) 

条件只比赋值高,    // 三目运算符优先级排到 13 位只比赋值运算符和 "," 高

逗号运算最低级!    //逗号运算符优先级最低 
```

# C 判断

判断结构要求程序员指定一个或多个要评估或测试的条件，以及条件为真时要执行的语句（必需的）和条件为假时要执行的语句（可选的）。

C 语言把任何**非零**和**非空**的值假定为 **true**，把**零**或 **null** 假定为 **false**。

下面是大多数编程语言中典型的判断结构的一般形式：

![C 中的判断语句](https://www.runoob.com/wp-content/uploads/2015/12/if.png)

## 判断语句

C 语言提供了以下类型的判断语句。点击链接查看每个语句的细节。

| 语句                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [if 语句](https://www.runoob.com/cprogramming/c-if.html)     | 一个 **if 语句** 由一个布尔表达式后跟一个或多个语句组成。    |
| [if...else 语句](https://www.runoob.com/cprogramming/c-if-else.html) | 一个 **if 语句** 后可跟一个可选的 **else 语句**，else 语句在布尔表达式为假时执行。 |
| [嵌套 if 语句](https://www.runoob.com/cprogramming/c-nested-if.html) | 您可以在一个 **if** 或 **else if** 语句内使用另一个 **if** 或 **else if** 语句。 |
| [switch 语句](https://www.runoob.com/cprogramming/c-switch.html) | 一个 **switch** 语句允许测试一个变量等于多个值时的情况。     |
| [嵌套 switch 语句](https://www.runoob.com/cprogramming/c-nested-switch.html) | 您可以在一个 **switch** 语句内使用另一个 **switch** 语句。   |



## ? : 运算符(三元运算符)

我们已经在前面的章节中讲解了 **条件运算符 ? :**，可以用来替代 **if...else** 语句。它的一般形式如下：

```
Exp1 ? Exp2 : Exp3;
```

其中，Exp1、Exp2 和 Exp3 是表达式。请注意，冒号的使用和位置。

? 表达式的值是由 Exp1 决定的。如果 Exp1 为真，则计算 Exp2 的值，结果即为整个 ? 表达式的值。如果 Exp1 为假，则计算 Exp3 的值，结果即为整个 ? 表达式的值。

![img](https://www.runoob.com/wp-content/uploads/2014/09/Conditional-Statement-in-C-Programming-Lanuage-Ternary-Operator.png)

### 实例

以下实例通过输入一个数字来判断它是否为奇数或偶数

## 实例

\#include<stdio.h>  int main() {    int num;     printf("输入一个数字 : ");    scanf("%d",&num);     (num%2==0)?printf("偶数"):printf("奇数"); }

**举一个三目（元）运算的例子：**

```
#include <stdio.h>

int A=10;
int B=20;
char buy;
int sum,number;

int main(){
    printf("以下是本店的商品及价格：\n A 商品每个十元；\n B 商品每个二十元；\n\n");
    printf("请输入你所需的产品(A 或 B):");
    scanf("%c",&buy);
    printf("请输入所需的数量：");
    scanf("%d",&number);
    sum=buy=='A'?A*number:B*number;
    printf("\n你所需要的%d个%c商品总共%d元。\n",number,buy,sum);
    return 0;
}
```

**switch语句一般形式：**

```
switch(表达式)
{
    case 常量表达式1:语句1;
    case 常量表达式2:语句2;
    ...
    default:语句n+1;
}
```

意思是先计算表达式的值，再逐个和 case 后的常量表达式比较，若不等则继续往下比较，若一直不等，则执行 default 后的语句；若等于某一个常量表达式，则从这个表达式后的语句开始执行，并执行后面所有 case 后的语句。

与 if 语句的不同：if 语句中若判断为真则只执行这个判断后的语句，执行完就跳出 if 语句，不会执行其他 if 语句；而 switch 语句不会在执行判断为真后的语句之后跳出循环，而是继续执行后面所有 case 语句。在每一 case 语句之后增加 break 语句，使每一次执行之后均可跳出 switch 语句，从而避免输出不应有的结果。

```
#include <stdio.h>

int main()
{
    int a;
    printf("input integer number: ");
    scanf("%d",&a);
    switch(a)
    {
        case 1:printf("Monday\n");
        break;
        case 2:printf("Tuesday\n");
        break;
        case 3:printf("Wednesday\n");
        break;
        case 4:printf("Thursday\n");
        break;
        case 5:printf("Friday\n");
        break;
        case 6:printf("Saturday\n");
        break;
        case 7:printf("Sunday\n");
        break;
        default:printf("error\n");
    }
}
```

# C 循环

有的时候，我们可能需要多次执行同一块代码。一般情况下，语句是按顺序执行的：函数中的第一个语句先执行，接着是第二个语句，依此类推。

编程语言提供了更为复杂执行路径的多种控制结构。

循环语句允许我们多次执行一个语句或语句组，下面是大多数编程语言中循环语句的流程图：

![循环结构](https://www.runoob.com/wp-content/uploads/2015/12/loop.png)

## 循环类型

C 语言提供了以下几种循环类型。点击链接查看每个类型的细节。

| 循环类型                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [while 循环](https://www.runoob.com/cprogramming/c-while-loop.html) | 当给定条件为真时，重复语句或语句组。它会在执行循环主体之前测试条件。 |
| [for 循环](https://www.runoob.com/cprogramming/c-for-loop.html) | 多次执行一个语句序列，简化管理循环变量的代码。               |
| [do...while 循环](https://www.runoob.com/cprogramming/c-do-while-loop.html) | 除了它是在循环主体结尾测试条件外，其他与 while 语句类似。    |
| [嵌套循环](https://www.runoob.com/cprogramming/c-nested-loops.html) | 您可以在 while、for 或 do..while 循环内使用一个或多个循环。  |



## 循环控制语句

循环控制语句改变你代码的执行顺序。通过它你可以实现代码的跳转。

C 提供了下列的循环控制语句。点击链接查看每个语句的细节。

| 控制语句                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [break 语句](https://www.runoob.com/cprogramming/c-break-statement.html) | 终止**循环**或 **switch** 语句，程序流将继续执行紧接着循环或 switch 的下一条语句。 |
| [continue 语句](https://www.runoob.com/cprogramming/c-continue-statement.html) | 告诉一个循环体立刻停止本次循环迭代，重新开始下次循环迭代。   |
| [goto 语句](https://www.runoob.com/cprogramming/c-goto-statement.html) | 将控制转移到被标记的语句。但是不建议在程序中使用 goto 语句。 |



## 无限循环

如果条件永远不为假，则循环将变成无限循环。**for** 循环在传统意义上可用于实现无限循环。由于构成循环的三个表达式中任何一个都不是必需的，您可以将某些条件表达式留空来构成一个无限循环。

## 实例

\#include <stdio.h>  int main () {   for( ; ; )   {      printf("该循环会永远执行下去！\n");   }   return 0; }

当条件表达式不存在时，它被假设为真。您也可以设置一个初始值和增量表达式，但是一般情况下，C 程序员偏向于使用 for(;;) 结构来表示一个无限循环。

**注意：**您可以按 **Ctrl + C** 键终止一个无限循环。

使用 while, for 分别输 1~100 以内的所有的奇数和偶数的和：

**使用 while：**

```
#include  <stdio.h>

int main(){
    int  sum=0;
    int  num=1;
    int  sum2=0;
    int  num2=2;
    while(num<100){
        sum=sum+num;
        num=num+2;
    }
    printf("奇数和为:%d\n",sum);

    while(num2<=100){
        sum2=sum2+num2;
        num2=num2+2;
    }
    printf("偶数和为：%d\n",sum2);
}
```

**使用 for:**

```
#include  <stdio.h>

int main(){

    int  sum=0;
    int sum2=0;
    int num,num2;
    for(num=1;num<100;num=num+2){
        sum=sum+num;
    }

    printf("奇数和%d\n",sum);
    for(num2=2;num2<=100;num2=num2+2){
        sum2=sum2+num2;
    }
    printf("偶数和%d\n",sum2);
}
```

用 do while 求算术平方根：

```
#include <stdio.h>

double DoSqrt(double z){
    double a=1;
    double b=0;
    double c=0;
    do{
        if(b*b<z){
            b+=a;
        }
        else{
            c=b;
            b-=a;
            a/=10;
        }
    }while(a>0.000001);

    return (b+c)/2;
}

int main(){
    double x, y;
    printf("请输入一个数字:");
    scanf("%lf", &x);
    if(x<0){
        printf("输入错误。");
    } else {
        y=DoSqrt(x);
        printf("%g 的平方根为: %g.\n", x, y);
    }

    int z=1;
    do{
        main();
        z++;
    }while(z>10);

    return 0;
}
```

我们看一个简单的列子，求100以内的素数。

```
#include<stdio.h>
#include<math.h>
int main(){
    int i,j;
    printf("100以内的素数有：\n");
    for(i=2;i<100;i++){
        for(j=2;j<sqrt(i);j++){
            if(i%j==0){
            break;
            }
        }
            if(j>sqrt(i)){
            printf("%d,\t",i);
            }
    }
}
```

第一个循环里面。i遍历从2到100以内的所有数字，第二个循环是在2到sqrt（n）之间看看是不是可以整除i。可以整除则不是素数，相反不可以整除，则是素数。

```
int fun(int n){
    for(i=2;i<=sqrt(p);i++){  
            if(x%i==0){
            return 0;  //不为素数
          }
            else{
            return 1; //为素数 
          }
        }
}
```

这个是求素数的关键代码。希望初学者要牢记于心，烂熟于心

求 100 以内偶数和奇数的和：

```
#include<stdio.h>

int main()
{
    int num=0,odd=0,even=0;
    for(;num<=100;num++)
    {
        if(num%2==0)
            even+=num;
        else
            odd+=num;
    }

    printf("Odd number sum is %d\n",odd);
    printf("Even number sum is %d\n",even);
    return 0;
}
```

用 continue 语句求输出 100~200 之间的不能被3整除的数。

```
#include<stdio.h>
int main()
{
    int n;
    for(n=100;n<=200;n++)
    {
        if(n%3==0)
            continue;
        printf("%d",n);
    }
    printf("\n");
    return 0;
}
```

一定要注意 break 语句与 continue 语句的区别，前者是结束整个循环过程，后者这是结束本次循环。应根据题目要求灵活使用。

# C 函数

函数是一组一起执行一个任务的语句。每个 C 程序都至少有一个函数，即主函数 **main()** ，所有简单的程序都可以定义其他额外的函数。

您可以把代码划分到不同的函数中。如何划分代码到不同的函数中是由您来决定的，但在逻辑上，划分通常是根据每个函数执行一个特定的任务来进行的。

函数**声明**告诉编译器函数的名称、返回类型和参数。函数**定义**提供了函数的实际主体。

C 标准库提供了大量的程序可以调用的内置函数。例如，函数 **strcat()** 用来连接两个字符串，函数 **memcpy()** 用来复制内存到另一个位置。

函数还有很多叫法，比如方法、子例程或程序，等等。

## 定义函数

C 语言中的函数定义的一般形式如下：

```
return_type function_name( parameter list )
{
   body of the function
}
```

在 C 语言中，函数由一个函数头和一个函数主体组成。下面列出一个函数的所有组成部分：

- **返回类型：**一个函数可以返回一个值。**return_type** 是函数返回的值的数据类型。有些函数执行所需的操作而不返回值，在这种情况下，return_type 是关键字 **void**。
- **函数名称：**这是函数的实际名称。函数名和参数列表一起构成了函数签名。
- **参数：**参数就像是占位符。当函数被调用时，您向参数传递一个值，这个值被称为实际参数。参数列表包括函数参数的类型、顺序、数量。参数是可选的，也就是说，函数可能不包含参数。
- **函数主体：**函数主体包含一组定义函数执行任务的语句。

## 实例

以下是 **max()** 函数的源代码。该函数有两个参数 num1 和 num2，会返回这两个数中较大的那个数：

/* 函数返回两个数中较大的那个数 */ int max(int num1, int num2)  {   /* 局部变量声明 */   int result;    if (num1 > num2)      result = num1;   else      result = num2;    return result;  }

## 函数声明

函数**声明**会告诉编译器函数名称及如何调用函数。函数的实际主体可以单独定义。

函数声明包括以下几个部分：

```
return_type function_name( parameter list );
```

针对上面定义的函数 max()，以下是函数声明：

```
int max(int num1, int num2);
```

在函数声明中，参数的名称并不重要，只有参数的类型是必需的，因此下面也是有效的声明：

```
int max(int, int);
```

当您在一个源文件中定义函数且在另一个文件中调用函数时，函数声明是必需的。在这种情况下，您应该在调用函数的文件顶部声明函数。

## 调用函数

创建 C 函数时，会定义函数做什么，然后通过调用函数来完成已定义的任务。

当程序调用函数时，程序控制权会转移给被调用的函数。被调用的函数执行已定义的任务，当函数的返回语句被执行时，或到达函数的结束括号时，会把程序控制权交还给主程序。

调用函数时，传递所需参数，如果函数返回一个值，则可以存储返回值。例如：

## 实例

\#include <stdio.h>  /* 函数声明 */ int max(int num1, int num2);  int main () {   /* 局部变量定义 */   int a = 100;   int b = 200;   int ret;    /* 调用函数来获取最大值 */   ret = max(a, b);    printf( "Max value is : %d\n", ret );    return 0; }  /* 函数返回两个数中较大的那个数 */ int max(int num1, int num2)  {   /* 局部变量声明 */   int result;    if (num1 > num2)      result = num1;   else      result = num2;    return result;  }

把 max() 函数和 main() 函数放一块，编译源代码。当运行最后的可执行文件时，会产生下列结果：

```
Max value is : 200
```

## 函数参数

如果函数要使用参数，则必须声明接受参数值的变量。这些变量称为函数的**形式参数**。

形式参数就像函数内的其他局部变量，在进入函数时被创建，退出函数时被销毁。

当调用函数时，有两种向函数传递参数的方式：

| 调用类型                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [传值调用](https://www.runoob.com/cprogramming/c-function-call-by-value.html) | 该方法把参数的实际值复制给函数的形式参数。在这种情况下，修改函数内的形式参数不会影响实际参数。 |
| [引用调用](https://www.runoob.com/cprogramming/c-function-call-by-pointer.html) | 通过指针传递方式，形参为指向实参地址的指针，当对形参的指向操作时，就相当于对实参本身进行的操作。 |

默认情况下，C 使用**传值调用**来传递参数。一般来说，这意味着函数内的代码不能改变用于调用函数的实际参数。

根据函数能否被其他源文件调用，将函数区分为内部函数和外部函数。

## 内部函数

如果一个函数只能被本文件中其他函数所调用，它称为内部函数。在定义内部函数时，在函数名和函数类型的前面加 static，即

```
static 类型名 函数名 （形参表）
```

例如，函数的首行：

```
static int max(int a,int b)
```

内部函数又称静态函数。使用内部函数，可以使函数的作用域只局限于所在文件。即使在不同的文件中有同名的内部函数，也互不干扰。提高了程序的可靠性。

## 外部函数

如果在定义函数时，在函数的首部的最左端加关键字 extern，则此函数是外部函数，可供其它文件调用。

如函数首部可以为

```
extern int max (int a,int b)
```

C 语言规定，如果在定义函数时省略 extern，则默认为外部函数。

在需要调用此函数的其他文件中，需要对此函数作声明（不要忘记，即使在本文件中调用一个函数，也要用函数原型来声明）。在对此函数作声明时，要加关键字 extern，表示该函数是在其他文件中定义的外部函数。

## 实例

以下实例通过多个文件的函数实现输入一串字符串，然后删除指定的字符：

file1.c(文件1)

```
#include <stdio.h>

static void delete_string(char str[],char ch);
int main()
{
    extern void enter(char str[]); // 对函数的声明
    extern void print(char str[]); // 对函数的声明
    char c,str[100];
    enter(str);
    scanf("%c",&c);
    delete_string(str,c);
    print(str);
    return 0;
}

static void delete_string(char str[],char ch)//内部函数
{
    int i,j;
    for(i=j=0;str[i]!='\0';i++)
    if(str[i]!=ch)
    str[j++]=str[i];
    str[j]='\0';
}
```

file2.c(文件2)

```
#include <stdio.h>

void enter(char str[100]) // 定义外部函数 enter
{
    fgets(str, 100, stdin); // 向字符数组输入字符串
}
```

file3.c(文件3)

```
#include <stdio.h>

void print(char str[]) // 定义外部函数 print
{
    printf("%s\n",str);
}
```

输入字符串"abcdef"，给字符数组 str，在输入要删去的字符'd'。 运行结果:

```
$ gcc file1.c file2.c file3.c 
$ ./a.out
abcdef                   # 输入的字符串
d                        # 要删除的字符
abcef                    # 删除后的字符串
```

**内联函数**

内联函数是指用inline关键字修饰的函数。在类内定义的函数被默认成内联函数。内联函数从源代码层看，有函数的结构，而在编译后，却不具备函数的性质。

内联扩展是用来**消除函数调用****时的时间开销**。它通常用于频繁执行的函数，对于小内存空间的函数非常受益。

使用内联函数的时候要注意：

- 递归函数不能定义为内联函数

- 内联函数一般适合于不存在while和switch等复杂的结构且只有1~5条语句的小函数上，否则编译系统将该函数视为普通函数。

- 内联函数只能先定义后使用，否则编译系统也会把它认为是普通函数。

- 对内联函数不能进行异常的接口声明。



**示例：**一个简单的交换函数

```
inline void swap(int *a, int *b)
{
    int t = *a;
    *a = *b;
    *b = t;
}
```

**关于 main 函数的参数**

在有些很专业的书会看到如下代码

```
int main( int argc, char *argv[] )
```

上面的代码中 main 函数带了参数。

但是有时又会看见main函数没有参数，如下：

```
int main()
```

**那么 main 函数到底有没有参数，有没有参数会不会有什么影响？**

main 函数其实与我们写的函数没有什么区别，它也会有自己的参数。

argc 和 argv 是 main 函数的形式参数。

这两个形式参数的类型是系统规定的。如果 main 函数要带参数，就是这两个类型的参数；否则main函数就没有参数。

变量名称argc和argv是常规的名称，当然也可以换成其他名称。在传入参数后main函数收到参数后就会做自己的事。那么，实际参数是如何传递给main函数的argc和argv的呢？我们知道，C程序在编译和链接后，都生成一个exe文件，执行该exe文件时，可以直接执行；也可以在命令行下带参数执行，命令行执行的形式为：可执行文件名称 参数1 参数2 ... ... 参数n。可执行文件名称和参数、参数之间均使用空格隔开。

如果按照这种方法执行，命令行字符串将作为实际参数传递给main函数。具体为：

-  (1) 可执行文件名称和所有参数的个数之和传递给 argc；
-  (2) 可执行文件名称（包括路径名称）作为一个字符串，首地址被赋给 argv[0]，参数1也作为一个字符串，首地址被赋给 argv[1]，... ...依次类推。

上面的是求两个数的最大值，我的是求三个数的最大值：

```
#include <stdio.h> 

int DoMax(int a, int b, int c){
    int max=a;
    if(b>max){
        max=b;
        if(c>max){
            max=c;
        }
    } else {
        if(c>max){
            max=c;
        }
    }
    return max;
}

int main(){
    int x, y, z, maxOne;
    printf("请输入三个数字（空格分隔）:");
    scanf("%d%d%d",&x,&y,&z);
    maxOne=DoMax(x, y, z);
    printf("\n");
    printf("最大数为:%d; \n",maxOne);
    return 0;
}
```

## 函数参数传递常用的三种方式

示例程序均以交换两个整数为例。

**1. 值传递**

```
#include <stdio.h>

void swap(int x, int y);
void swap(int x, int y)
{
    int temp;
    temp = x;
    x = y;
    y = temp;
}

int main( int argc, char *argv[] )
{
    int a = 5;
    int b = 10;
    swap(a, b); //调用交换函数
    printf("交换结果为 a = %d, b = %d\n",a,b);
    return 0;
}
```

由于值传递是单向传递，传递过程中只是改变了形参的数值，并未改变实参的数值，因此并不会改变a和b原有的值。

**2. 指针传递**

```
#include <stdio.h>

void swap(int *x, int *y);
void swap(int *x, int *y)
{
    int temp;
    temp = *x;
    *x = *y;
    *y = temp;
}

int main( int argc, char *argv[] )
{
    int a = 5;
    int b = 10;
    swap(&a, &b); //调用交换函数
    printf("交换结果为 a = %d, b = %d\n",a,b);
    return 0;
}
```

指针传递过程中，将a和b的地址分别传递给了x和y，在函数体内部改变了a、b所在地址的值，即交换了a、b的数值。

**3. 引用传递**

```
#include <stdio.h>

void swap(int &x, int &y);
void swap(int &x, int &y)
{
    int temp;
    temp = x;
    x = y;
    y = temp;
}

int main( int argc, char *argv[] )
{
    int a = 5;
    int b = 10;
    swap(a, b); //调用交换函数
    printf("交换结果为 a = %d, b = %d\n",a,b);
    return 0;
}
```

引用传递中，在调用swap(a, b);时函数会用a、b分别代替x、y，即x、y分别引用了a、b变量，这样函数体中实际参与运算的其实就是实参a、b本身，因此也能达到交换数值的目的。

**注：**严格来说，C语言中是没有引用传递，这是C++中语言特性，因此在.c文件中使用引用传递会导致程序编译出错。

函数声明和函数原型的参数名可以不一样，编译器他想知道的是函数参数的类型，与函数参数的名字没有关系，eg:

\#

```
include<stdio.h>

int sum(int c,int d); // 函数声明

int main(int argc ,char*argv[])
{
   int a=2,b=3;
   printf("输出结果为: %d \n",sum(a,b));
   return 0;
}

int sum(int a,int b)
{
    return a+b ;
}
```

甚至函数声明可以写成:

```
int sum(int ,int );
```

编译器只要检查到函数返回类型，名称和参数类型正确即可。

函数实现了代码的重用，大大简化缩短了程序员的工作量，更使得程序的可读性大大提高，函数的递归调用更是一种简化程序代码的方法；递归调用其实机是函数自己调用自己。

下面的实例是一个求1+2+3.....+n的递归实例：

```
#include <stdio.h>  //导如输入输出头文件

int sum(int n);//声明函数 
int main(){
    //主函数 
    int a=sum(4);
    printf("%d",a);
  return 0;
}
int sum(int n){
    //求和函数实现 
    //如果n为1，无需求和，直接返回1 
    if(n==1){
        return 1;
    }
    //如果n大于1，就返回n加上1+2+3.....+n-1的和 
    return n+sum(n-1);
}
```

可以发现，递归的代码很少，但是，递归也有缺点，递归占用的内存要比递推大，而且时间也要比递推长。

占位符就是先占住一个固定的位置，等着你再往里面添加内容的符号，广泛用于计算机中各类文档的编辑。

格式占位符(%)是在C/C++语言中格式输入函数，如 scanf、printf 等函数中使用。其意义就是起到格式占位的意思，表示在该位置有输入或者输出。

-  **%d, %i** 代表整数
-  **%f** 浮点
-  **%s** 字符串
-  **%c** char
-  **%p** 指针
-  **%fL** 长log
-  **%e** 科学计数
-  **%g** 小数或科学计数。
-  **%a,%A** 读入一个浮点值(仅C99有效)。
-  **%c** 读入一个字符。
-  **%d** 读入十进制整数。
-  **%i** 读入十进制，八进制，十六进制整数。
-  **%o** 读入八进制整数。
-  **%x,%X** 读入十六进制整数。
-  **%s** 读入一个字符串，遇空格、制表符或换行符结束。
-  **%f,%F,%e,%E,%g,%G** 用来输入实数，可以用小数形式或指数形式输入。
-  **%p** 读入一个指针。
-  **%u** 读入一个无符号十进制整数。
-  **%n** 至此已读入值的等价字符数。
-  **%[]** 扫描字符集合。
-  **%%** 读 % 符号

**实例：**

```
scanf("%d,%d,%d",&a,&b,&c); // 从键盘输入三个整数，用逗号分隔 　
scanf("%c", &s);   // 从键盘输入一个字符 　
scanf("%f", &f);   // 从键盘输入一个浮点型数据 　
printf("%d\n",a);  // 输出一个整数 　
printf("%f\n",b);  // 输出一个浮点数 　
printf("%s\n",c);  // 输出一个字符, 其中\n表示换行
```

**1、函数的调用**：由于程序是从上向下执行，所以函数要先声明，后调用。这种先后是文档中所处位置的先后，不是时间的先后。以下写法为正确的：

```
#include<stdio.h>

void f(void)  /*定义函数*/
{
    printf("我是一个函数，我将在主函数中输出\n");
}

int main(void)
{
    f();  /*调用函数*/
}
```

在上面这个实例中，在主函数中调用了函数 **f()**, 而函数的声明是在调用以前。

2、这种写法是错误的

```
#include<stdio.h>

int main(void)
{
    f();/*调用函数*/
}

void f(void)  /*定义函数*/
{
    printf("我是一个函数，我将在主函数中输出\n");
}
```

在上面这个实例中，在主函数中调用了函数 **f()**, 而函数的声明却在调用之后。由于函数执行时是从上往下执行的所以，这写法是错误的。

如果函数非要写在主函数之后可以在主函数之前加入一个函数的前置声明。

前置声明如下：

```
#include<stdio.h>

void f(void); /*前置声明*/
int main(void)  
{
    f();  /*调用函数*/
}

void f(void)  /*定义函数*/
{
    printf("我是一个函数，我一定要先声明后调用\n");
}
```

本质上说，C 里面所有的函数参数传递，都是值传递。

指针传递之所以能改变传递参数变量的值，是因为 swap 函数交换的不是传递进来的指针本身，而是指针指向的值。

```
void swap(int *x, int *y);
void swap(int *x, int *y){
    int temp;
    //这里操作的是指针指向的值 而不是指针
    temp = *x;
    *x = *y;
    *y = temp;
//   倘若直接交换指针 a、b的值不会交换
//    temp = x;
//    x = y;
//    y = temp;
}

int main( int argc, char *argv[] )
{
    int a = 5;
    int b = 10;
    swap(a, b); //调用交换函数
    printf("交换结果为 a = %d, b = %d\n",a,b);
    return 0;
}
```

引用传递也类似。

上面那个3个数最大值的太复杂，我这个好理解些:

```
#include <stdio.h>

int main()
{
    int a,b,c,max;
    printf("请输入三个数字（空格分隔）:");
    scanf("%d%d%d",&a,&b,&c);
    if((a>b)&&(a>c))
        max=a;
    else if((b>a)&&(b>c))
        max=b;
    else if((c>a)&&(c>a))
        max=c;
   printf("max=%d",max);
   return 0;
}
```

# C 作用域规则

任何一种编程中，作用域是程序中定义的变量所存在的区域，超过该区域变量就不能被访问。C 语言中有三个地方可以声明变量：

1. 在函数或块内部的**局部**变量
2. 在所有函数外部的**全局**变量
3. 在**形式**参数的函数参数定义中

让我们来看看什么是**局部**变量、**全局**变量和**形式**参数。

## 局部变量

在某个函数或块的内部声明的变量称为局部变量。它们只能被该函数或该代码块内部的语句使用。局部变量在函数外部是不可知的。下面是使用局部变量的实例。在这里，所有的变量 a、b 和 c 是 main() 函数的局部变量。

## 实例

\#include <stdio.h>  int main () {  /* 局部变量声明 */  int a, b;  int c;   /* 实际初始化 */  a = 10;  b = 20;  c = a + b;   printf ("value of a = %d, b = %d and c = %d\n", a, b, c);   return 0; }

## 全局变量

全局变量是定义在函数外部，通常是在程序的顶部。全局变量在整个程序生命周期内都是有效的，在任意的函数内部能访问全局变量。

全局变量可以被任何函数访问。也就是说，全局变量在声明后整个程序中都是可用的。下面是使用全局变量和局部变量的实例：

## 实例

\#include <stdio.h>  /* 全局变量声明 */ int g;  int main () {  /* 局部变量声明 */  int a, b;   /* 实际初始化 */  a = 10;  b = 20;  g = a + b;   printf ("value of a = %d, b = %d and g = %d\n", a, b, g);   return 0; }

在程序中，局部变量和全局变量的名称可以相同，但是在函数内，如果两个名字相同，会使用局部变量值，全局变量不会被使用。下面是一个实例：

在程序中，局部变量和全局变量的

## 实例

\#include <stdio.h>  /* 全局变量声明 */ int g = 20;  int main () {  /* 局部变量声明 */  int g = 10;   printf ("value of g = %d\n",  g);   return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
value of g = 10
```

## 形式参数

函数的参数，形式参数，被当作该函数内的局部变量，如果与全局变量同名它们会优先使用。下面是一个实例：

## 实例

\#include <stdio.h>  /* 全局变量声明 */ int a = 20;  int main () {  /* 在主函数中的局部变量声明 */  int a = 10;  int b = 20;  int c = 0;  int sum(int, int);   printf ("value of a in main() = %d\n",  a);  c = sum( a, b);  printf ("value of c in main() = %d\n",  c);   return 0; }  /* 添加两个整数的函数 */ int sum(int a, int b) {    printf ("value of a in sum() = %d\n",  a);    printf ("value of b in sum() = %d\n",  b);     return a + b; }

当上面的代码被编译和执行时，它会产生下列结果：

```
value of a in main() = 10
value of a in sum() = 10
value of b in sum() = 20
value of c in main() = 30
```

> **全局变量与局部变量在内存中的区别**：
>
> - 全局变量保存在内存的全局存储区中，占用静态的存储单元；
> - 局部变量保存在栈中，只有在所在函数被调用时才动态地为变量分配存储单元。
>
> 更多内容可参考：[C/C++ 中 static 的用法全局变量与局部变量](https://www.runoob.com/w3cnote/cpp-static-usage.html)

## 初始化局部变量和全局变量

当局部变量被定义时，系统不会对其初始化，您必须自行对其初始化。定义全局变量时，系统会自动对其初始化，如下所示：

| 数据类型 | 初始化默认值 |
| :------- | :----------- |
| int      | 0            |
| char     | '\0'         |
| float    | 0            |
| double   | 0            |
| pointer  | NULL         |

正确地初始化变量是一个良好的编程习惯，否则有时候程序可能会产生意想不到的结果，因为未初始化的变量会导致一些在内存位置中已经可用的垃圾值。

## C 的形参与实参

在 C 语言中，形参与实参虽然很简单，但是，是大家比较容易混淆的一个点，这里将为大家详细的讲解。

**概念：**从字面上理解，所谓形式参数即只只是声明了一个作为参数的变量，并未直接进行赋值使用，而实际参数则相反。

如下例

```
#include <stdio.h>

int test(int,int); // 形参，只声明

int main()
{
    int a,b;
    printf("%d",test(5,3)); // 实参，已赋值
}

int test(int a,int b) // 形参
{
    a=a+b;
    return a;
}
```

像上面在 test() 函数里只声明了最为参数的变量，而 main() 函数里则对它赋了值。

关于C语言形参与实参的区别，我就简单说几点：

实参可以是变量，变量与表达式。实参与形参。

实参与形参类型相同或赋值兼容

在调用函数过程中发生的实参与形参之间的数据传递，常称为“虚实结合”

1. 在定义函数中制定的形参，在没有出现函数调用时不占用内存中的存储单元。在函数调用时才分配内存
2. 将实参的值传递给形参
3. 在执行函数时，由于形参已经有值。可以用形参进行运算。
4. 通过return语句将函数值返回，若无返回值，则无return
5. 调用结束后，形参被释放掉，实参保留原值（单向传值）

# C 数组

C 语言支持**数组**数据结构，它可以存储一个固定大小的相同类型元素的顺序集合。数组是用来存储一系列数据，但它往往被认为是一系列相同类型的变量。

数组的声明并不是声明一个个单独的变量，比如 number0、number1、...、number99，而是声明一个数组变量，比如 numbers，然后使用 numbers[0]、numbers[1]、...、numbers[99] 来代表一个个单独的变量。数组中的特定元素可以通过索引访问。

所有的数组都是由连续的内存位置组成。最低的地址对应第一个元素，最高的地址对应最后一个元素。

![C 中的数组](https://www.runoob.com/wp-content/uploads/2014/08/arrays.jpg)

## 声明数组

在 C 中要声明一个数组，需要指定元素的类型和元素的数量，如下所示：

```
type arrayName [ arraySize ];
```

这叫做一维数组。**arraySize** 必须是一个大于零的整数常量，**type** 可以是任意有效的 C 数据类型。例如，要声明一个类型为 double 的包含 10 个元素的数组 **balance**，声明语句如下：

```
double balance[10];
```

现在 *balance* 是一个可用的数组，可以容纳 10 个类型为 double 的数字。

## 初始化数组

在 C 中，您可以逐个初始化数组，也可以使用一个初始化语句，如下所示：

```
double balance[5] = {1000.0, 2.0, 3.4, 7.0, 50.0};
```

大括号 { } 之间的值的数目不能大于我们在数组声明时在方括号 [ ] 中指定的元素数目。

如果您省略掉了数组的大小，数组的大小则为初始化时元素的个数。因此，如果：

```
double balance[] = {1000.0, 2.0, 3.4, 7.0, 50.0};
```

您将创建一个数组，它与前一个实例中所创建的数组是完全相同的。下面是一个为数组中某个元素赋值的实例：

```
balance[4] = 50.0;
```

上述的语句把数组中第五个元素的值赋为 50.0。所有的数组都是以 0 作为它们第一个元素的索引，也被称为基索引，数组的最后一个索引是数组的总大小减去 1。以下是上面所讨论的数组的的图形表示：

![数组表示](https://www.runoob.com/wp-content/uploads/2014/08/array_presentation.jpg)

数组元素可以通过数组名称加索引进行访问。元素的索引是放在方括号内，跟在数组名称的后边。例如：

```
double salary = balance[9];
```

上面的语句将把数组中第 10 个元素的值赋给 salary 变量。下面的实例使用了上述的三个概念，即，声明数组、数组赋值、访问数组：

## 实例

\#include <stdio.h>  int main () {   int n[ 10 ]; /* n 是一个包含 10 个整数的数组 */   int i,j;    /* 初始化数组元素 */            for ( i = 0; i < 10; i++ )   {      n[ i ] = i + 100; /* 设置元素 i 为 i + 100 */   }      /* 输出数组中每个元素的值 */   for (j = 0; j < 10; j++ )   {      printf("Element[%d] = %d\n", j, n[j] );   }    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Element[0] = 100
Element[1] = 101
Element[2] = 102
Element[3] = 103
Element[4] = 104
Element[5] = 105
Element[6] = 106
Element[7] = 107
Element[8] = 108
Element[9] = 109
```

## C 中数组详解

在 C 中，数组是非常重要的，我们需要了解更多有关数组的细节。下面列出了 C 程序员必须清楚的一些与数组相关的重要概念：

| 概念                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [多维数组](https://www.runoob.com/cprogramming/c-multi-dimensional-arrays.html) | C 支持多维数组。多维数组最简单的形式是二维数组。             |
| [传递数组给函数](https://www.runoob.com/cprogramming/c-passing-arrays-to-functions.html) | 您可以通过指定不带索引的数组名称来给函数传递一个指向数组的指针。 |
| [从函数返回数组](https://www.runoob.com/cprogramming/c-return-arrays-from-function.html) | C 允许从函数返回数组。                                       |
| [指向数组的指针](https://www.runoob.com/cprogramming/c-pointer-to-an-array.html) | 您可以通过指定不带索引的数组名称来生成一个指向数组中第一个元素的指针。 |

指针与数组的区别

```
char *str="Hello";
char arrgs[]="Hello";
```

字符数组:

```
char *p=(char*)&arrgs == char *p=arrgs ;
char *p=(char*)(&(arrgs[0])) == &arrgs == arrgs;
```

字符串指针:

```
char *p=(char*)&str != str; str==str[0] =='H'
```

得出结论字符数组的 首地址可以用 arrgs ，&arrgs,来表示并且它们相等，

但是指针&str和str是不同的,当遇到字符串指针时候要注意处理方式;

**一个小知识点：**

在我们没有明确数组的元素个数时，在程序中想知道数组单元个数可以使用 **sizeof(a)/sizeof(a[0])**, **sizeof(a)** 是得到数组 a 的大小，**sizeof(a[0])** 是得到数组 a 中单个元素的大小（因此可以不必要是a[0],a[i]都行），eg:

```
#include<stdio.h>

int main(int argc,char *grgv[])
{
    int a[]={1,2,3,4,5};
    int b;
    b=sizeof(a)/sizeof(a[0]);
    printf("数组元素个数为：%d",b);
    return 0; 
}
```

数组是一种一次申请多个变量的方法，数组的使用让我们可以在程序中保留多个变量的值，进行处理，例如给定n个学生的成绩，要求有多少学生超过了平均分,代码如下：

```
#include <stdio.h>  //导如输入输出头文件

int main(){
    //主函数 
    int cj[100]={0};//定义数组 
    int n;
    int count=0;//定义计数器，统计有多少人达到平均分 
    scanf("%d",&n);
    int i=0;
    //读入数据 
    for(i=0;i<n;i++){
        scanf("%d",&cj[i]);
    } 
    //对数据进行求和 
    double sum=0;
    for(i=0;i<n;i++){
        sum+=cj[i];
    } 
    //求平均分 
    double arg=sum/n;
    //判断有多少人达到平均分； 
    for(i=0;i<n;i++){
        if(cj[i]>arg){
            count++;
        }
    } 
    //输出平均分和人数 
    printf("平均分为：%0.2f\n超过平均分的人有：%d个\n",arg,count);
  return 0;
}
```

```
#include <stdio.h>

int main()
{
    int a[2] = {1,2};
    printf("a      = %d\n",a[0]);
    printf("*(a+0) = %d\n",*(a + 0));
    printf("a[1]   = %d\n",a[1]);
    printf("*a     = %d\n",*a);
    printf("*(a+1) = %d\n",*(a + 1));
    printf("\n");
    printf("a    的地址：%p\n",a);
    printf("(a+0)的地址：%p\n",(a + 0));
    printf("(a+1)的地址：%p\n",(a + 1));
    // %p 读入一个指针
    printf("\n");
    return 0;
}
```

输出结果：

```
a      = 1
*(a+0) = 1
a[1]   = 2
*a     = 1
*(a+1) = 2

a    的地址：0x7ffe9e227634
(a+0)的地址：0x7ffe9e227634
(a+1)的地址：0x7ffe9e227638
```

事实上 **a[0] 、a[1]...a[i]** 代表的都是值，**a、(a+0)、(a+1)、(a+i)** 代表的是地址；另外这里的 **a** 代表整个数组的首地址，相当于 **a[0]** 的地址，而这里 **(a+1)** 就代表的是 **a[0+1]** 的地址。

正如文章中提到的：所有的数组都是由连续的内存位置组成。最低的地址对应第一个元素，最高的地址对应最后一个元素，即是说 **(a+i)** 就代表的是 **a[0+i]** 的地址。

对于数组的初始化需要注意以下几点：

1) 可以只给部分元素赋值，当 **{ }** 中值的个数少于元素个数时，只给前面部分元素赋值。例如：

```
int a[10]={12, 19, 22 , 993, 344};
```

表示只给 **a[0]~a[4] 5** 个元素赋值，而后面 **5** 个元素自动初始化为 **0**。

当赋值的元素少于数组总体元素的时候，不同类型剩余的元素自动初始化值说明如下：

-  对于 short、int、long，就是整数 **0**；
-  对于 char，就是字符 **'\0'**；
-  对于 float、double，就是小数 **0.0**。

我们可以通过下面的形式将数组的所有元素初始化为 0：

```
int nums[10] = {0};
char str[10] = {0};
float scores[10] = {0.0};
```

由于剩余的元素会自动初始化为 0，所以只需要给第 0 个元素赋值为 0 即可。

2) 只能给元素逐个赋值，不能给数组整体赋值。例如给 10 个元素全部赋值为 1，只能写作：

```
int a[10] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
```

而不能写作：

```
int a[10] = 1;
```

**指针与数组名的区别**

指针：也是一个变量，存储的数据是地址。

数组名：代表的是该数组最开始的一个元素的地址。

```
int a[10];
int *p;
p = &a[0] // 可以写成 p = a;
```

- 对数组元素 a[i]的引用也可以写成*(a+i)这种形式。
- 赋值语句 p=&a[0] 也可以写成下列形式: p=a。
- p 是个指针，p[i]与*(p+i)是等价的。

区别：指针是一个变量，可以进行数值运算。数组名不是变量，不可以进行数值运算。

**数组和指针的关系**

1. 函数的参数中 数组类型参数 `int a[]`本质是指针 可以直接换成 `int *a`;
2. 可以用数组的运算符`[]`进行运算，而且可以通过`a[0]`就直接修改了外面的数组元素;
3. `size of(a) == sizeof(int *)` 所以函数内部没法用`sizeof`得到数组的长度 只能传一个`len`;

```
// 一个整型数组的指针，长度为 len， 得到数组中的最小值和最大值 ———— 从外部传入两个指针，从而将所求的两个结果保存带出来，得到多个返回值。
void minMax(int a[], int len, int *min, int *max) {
    int i;
    
    *min = *max = a[0]; //假定最大值 最小值相等 为a[0]  
    
    for(i= 1; i < len, i++) {
        if( a[i] < *min) {
          *min = a[i];
        }
        
        if(a[i] > *max) {
          *max = a[i];
        }
    }

}

int a[] = {1, 2, 3, 4, 5, 7, 8, 9, 15, 18, 25, 33};
int min, max;
minMax(a, sizeof(a)/ sizeof(a[0]), &min , &max );
printf( "min = %d, max = %d \n",  min,  max);
```

- 数组变量是特殊的指针，数组变量本身表达地址，int a[10];  int *p =a;
- 数组变量无需用取址符& ;
- 数组的元素表达的是变量，需要用&取地址,如 m = &a[0];
- `[]`运算符可以对数组做，也可以对指针做 `p[0] == a[0]`;
- `*`运算符可以对指针做，也可以对数组做 `*a = 25`, *a可以得到或者修改数组首个元素的值;
- 实际上数组是 const 的指针，所以不能被赋值。 如果 `int a[] = {1,2,5,7}`; `int b[]`，由于 `int b[]`实质上等价于 <===> int const *b;   b 是一个常数，不能被改变，初始化出来代表这个数组就不能再改变。 b = a; (错误) 数组变量之间是不可以这样互相赋值的。 而 int *q = a; 这样是可以的。

**数组赋值的区别:**

```
char a[]="runoob"; // 这样赋值之后在结尾会自动加上'\0'。
char a1[]={'r','u','n','o','o','b'}; // 这样赋值是整整好好的6个空间不会自动加上 <span class="marked">\0</span>。
```

所以比较的话，a 的长度比 a1 的要大。

数组初始化技巧: 将元素全部置零 **{0}**。

```
#include <stdio.h>

int main()
{
    double arr[10] = {0};

    for(int i=0; i<sizeof(arr)/sizeof(double); i++)
        printf("%d ", arr[i]);
    printf("\n\n");

    int a[3][4] = {0};
    for(int i=0; i<3; i++)
        {
        for(int j=0; j<4; j++)
                printf("%d ", a[i][j]);
        printf("\n");
        }
    
    return 0;
}
```

# C enum(枚举)

枚举是 C 语言中的一种基本数据类型，它可以让数据更简洁，更易读。

枚举语法定义格式为：

```
enum　枚举名　{枚举元素1,枚举元素2,……};
```

接下来我们举个例子，比如：一星期有 7 天，如果不用枚举，我们需要使用 #define 来为每个整数定义一个别名：

\#define MON  1 #define TUE  2 #define WED  3 #define THU  4 #define FRI  5 #define SAT  6 #define SUN  7

这个看起来代码量就比较多，接下来我们看看使用枚举的方式：

```
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
```

这样看起来是不是更简洁了。

**注意：**第一个枚举成员的默认值为整型的 0，后续枚举成员的值在前一个成员上加 1。我们在这个实例中把第一个枚举成员的值定义为 1，第二个就为 2，以此类推。

> 可以在定义枚举类型时改变枚举元素的值：
>
> ```
> enum season {spring, summer=3, autumn, winter};
> ```
>
> 没有指定值的枚举元素，其值为前一元素加 1。也就说 spring 的值为 0，summer 的值为 3，autumn 的值为 4，winter 的值为 5

### 枚举变量的定义

前面我们只是声明了枚举类型，接下来我们看看如何定义枚举变量。

我们可以通过以下三种方式来定义枚举变量

**1、先定义枚举类型，再定义枚举变量**

```
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
enum DAY day;
```

**2、定义枚举类型的同时定义枚举变量**

```
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
```

**3、省略枚举名称，直接定义枚举变量**

```
enum
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
```

## 实例

\#include<stdio.h>  enum DAY {      MON=1, TUE, WED, THU, FRI, SAT, SUN };  int main() {    enum DAY day;    day = WED;    printf("%d",day);    return 0; }

以上实例输出结果为：

```
3
```

在C 语言中，枚举类型是被当做 int 或者 unsigned int 类型来处理的，所以按照 C 语言规范是没有办法遍历枚举类型的。

不过在一些特殊的情况下，枚举类型必须连续是可以实现有条件的遍历。

以下实例使用 for 来遍历枚举的元素：

## 实例

\#include<stdio.h>  enum DAY {      MON=1, TUE, WED, THU, FRI, SAT, SUN } day; int main() {    // 遍历枚举元素    for (day = MON; day <= SUN; day++) {        printf("枚举元素：%d \n", day);    } }

以上实例输出结果为：

```
枚举元素：1 
枚举元素：2 
枚举元素：3 
枚举元素：4 
枚举元素：5 
枚举元素：6 
枚举元素：7
```

以下枚举类型不连续，这种枚举无法遍历。

```
enum
{
    ENUM_0,
    ENUM_10 = 10,
    ENUM_11
};
```

枚举在 switch 中的使用：

## 实例

\#include <stdio.h> #include <stdlib.h> int main() {     enum color { red=1, green, blue };     enum  color favorite_color;     /* ask user to choose color */    printf("请输入你喜欢的颜色: (1. red, 2. green, 3. blue): ");    scanf("%d", &favorite_color);     /* 输出结果 */    switch (favorite_color)    {    case red:        printf("你喜欢的颜色是红色");        break;    case green:        printf("你喜欢的颜色是绿色");        break;    case blue:        printf("你喜欢的颜色是蓝色");        break;    default:        printf("你没有选择你喜欢的颜色");    }     return 0; }

以上实例输出结果为：

```
请输入你喜欢的颜色: (1. red, 2. green, 3. blue): 1
你喜欢的颜色是红色
```

### 将整数转换为枚举

以下实例将整数转换为枚举：

## 实例

\#include <stdio.h> #include <stdlib.h>  int main() {     enum day    {        saturday,        sunday,        monday,        tuesday,        wednesday,        thursday,        friday    } workday;     int a = 1;    enum day weekend;    weekend = ( enum day ) a;  //类型转换    //weekend = a; //错误    printf("weekend:%d",weekend);    return 0; }

以上实例输出结果为：

```
weekend:1
```

用 for 用来遍历枚举元素根本是不可行的，直接上代码：

```
#include<stdio.h>
enum DAY{MON=1, TUE, WED, THU=7, FRI, SAT, SUN};

int main()
{
        enum DAY day;
        for(day=MON;day<=SUN;day++)
        {
                printf("day=%d\n",day);
        }
        return 0;
}
```

得到的结果：

```
day=1
day=2
day=3
day=4
day=5
day=6
day=7
day=8
day=9
day=10
```

所以用来遍历是不可行的，它只是给 day 赋值了一个整数类型的值。

枚举其实可以直接使用，上代码：

```
#include <stdio.h>
#include <stdlib.h>

enum {
 Q,W,E=4,R
};

int main()
{

   printf("枚举值QWER分别是: %d , %d , %d , %d",Q,W,E,R);
   
   return 0;
}
```

输出:

```
枚举值QWER分别是: 0 , 1 , 4 , 5
```

在 C 语言中，枚举类型是被当做 int 或者 unsigned int 类型来处理的，所以按照 C 语言规范是没有办法遍历枚举类型的。

不过在一些特殊的情况下，枚举类型必须连续是可以实现有条件的遍历。

以下枚举类型不连续，这种枚举无法遍历。

```
enum
{
    ENUM_0,
    ENUM_10 = 10,
    ENUM_11
};
```

[一楼的同学](https://www.runoob.com/cprogramming/c-enum.html#comment-34407)你的枚举类型不连续。。。

```
#include<stdio.h>
enum DAY{MON=1, TUE, WED, THU=7, FRI, SAT, SUN};

int main()
{
        enum DAY day;
        for(day=MON;day<=SUN;day++)
        {
                printf("day=%d\n",day);
        }
        return 0;
}
#include<stdio.h>
enum DAY{MON=1, TUE=2, WED=3, THU=7, FRI=8, SAT=9, SUN=10};

int main()
{
        enum DAY day;
        for(day=MON;day<=SUN;day++)
        {
                printf("day=%d\n",day);
        }
        return 0;
}
#include<stdio.h>
enum DAY{MON=1, TUE, WED, THU=7, FRI, SAT, SUN};

int main()
{
        enum DAY day;
        for(day=1;day<=10;day++)
        {
                printf("day=%d\n",day);
        }
        return 0;
}
```

这 3 个是等价的，不明白你们说的遍历是否可行是什么意思但是。

```
enum
{
    ENUM_0,
    ENUM_10 = 10,
    ENUM_11
};
```

这个是没错的，0 10 11 不会报错。我说错了发邮件联系我 **1958380208@qq.com**。

**关于遍历问题的讨论：**

首先大家要清楚在 c 语言中枚举元素的数据类型是被当成了 int 或者 unsigned int 型，不会是其他数据类型，如浮点型。简单而言，每个枚举元素可以看作为一个整形变量的宏定义。所以无论是哪种形式的遍历，大家都可以看成是对整形变量的操作。至于作者说的条件的遍历情况无非是恰好满足了遍历时 枚举变量的值 恰好是和定义好的枚举类型中的值一样罢了。可能说得有点抽象，一个栗子帮助大家理解：

一、

```
#include <stdio.h>

int main(void)
{
    enum MONTH{A=1,B,C,D,F=10};    //注意B的值是在A加1，C是B加1....所以D你知道吧
    enum MONTH month=A;
    for(month=A;month<F;month++)
    printf("the value of month is :%d  ",month);
    return 0;
}
```

二、

```
#include <stdio.h>

int main(void)
{
    int month=1;
    int F=10;
    for(month=1;month<F;month++)
    printf("the value of month is :%d  ",month);
    return 0;
}
```

上面两种情况是一样的，希望帮到大家。

枚举类型连续，遍历时要加类型转换:

```
#include<stdio.h>

enum DAY
{
    MON=1, TUE, WED, THU, FRI, SAT, SUN
} weekend;

int main()
{
    weekend=MON;
    printf("day is %d\n",weekend);
    // 遍历枚举元素
    while(weekend<=SUN)
    {
        printf("枚举元素：%d \n",weekend);  //类型扎UN哈UN
        weekend=(enum DAY)(weekend+1);
    }
}
```

# C 指针

学习 C 语言的指针既简单又有趣。通过指针，可以简化一些 C 编程任务的执行，还有一些任务，如动态内存分配，没有指针是无法执行的。所以，想要成为一名优秀的 C 程序员，学习指针是很有必要的。

正如您所知道的，每一个变量都有一个内存位置，每一个内存位置都定义了可使用连字号（&）运算符访问的地址，它表示了在内存中的一个地址。请看下面的实例，它将输出定义的变量地址：

## 实例

\#include <stdio.h>  int main () {   int  var1;   char var2[10];    printf("var1 变量的地址： %p\n", &var1  );   printf("var2 变量的地址： %p\n", &var2  );    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
var1 变量的地址： 0x7fff5cc109d4
var2 变量的地址： 0x7fff5cc109de
```

通过上面的实例，我们了解了什么是内存地址以及如何访问它。接下来让我们看看什么是指针。

## 什么是指针？

**指针**是一个变量，其值为另一个变量的地址，即，内存位置的直接地址。就像其他变量或常量一样，您必须在使用指针存储其他变量地址之前，对其进行声明。指针变量声明的一般形式为：

```
type *var-name;
```

在这里，**type** 是指针的基类型，它必须是一个有效的 C 数据类型，**var-name** 是指针变量的名称。用来声明指针的星号 * 与乘法中使用的星号是相同的。但是，在这个语句中，星号是用来指定一个变量是指针。以下是有效的指针声明：

int    *ip;    /* 一个整型的指针 */ double *dp;    /* 一个 double 型的指针 */ float  *fp;    /* 一个浮点型的指针 */ char   *ch;     /* 一个字符型的指针 */

所有实际数据类型，不管是整型、浮点型、字符型，还是其他的数据类型，对应指针的值的类型都是一样的，都是一个代表内存地址的长的十六进制数。

不同数据类型的指针之间唯一的不同是，指针所指向的变量或常量的数据类型不同。

## 如何使用指针？

使用指针时会频繁进行以下几个操作：定义一个指针变量、把变量地址赋值给指针、访问指针变量中可用地址的值。这些是通过使用一元运算符 ***** 来返回位于操作数所指定地址的变量的值。下面的实例涉及到了这些操作：

## 实例

\#include <stdio.h>  int main () {   int  var = 20;   /* 实际变量的声明 */   int  *ip;        /* 指针变量的声明 */    ip = &var;  /* 在指针变量中存储 var 的地址 */    printf("Address of var variable: %p\n", &var  );    /* 在指针变量中存储的地址 */   printf("Address stored in ip variable: %p\n", ip );    /* 使用指针访问值 */   printf("Value of *ip variable: %d\n", *ip );    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Address of var variable: bffd8b3c
Address stored in ip variable: bffd8b3c
Value of *ip variable: 20
```

## C 中的 NULL 指针

在变量声明的时候，如果没有确切的地址可以赋值，为指针变量赋一个 NULL 值是一个良好的编程习惯。赋为 NULL 值的指针被称为**空**指针。

NULL 指针是一个定义在标准库中的值为零的常量。请看下面的程序：

## 实例

\#include <stdio.h>  int main () {   int  *ptr = NULL;    printf("ptr 的地址是 %p\n", ptr  );    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
ptr 的地址是 0x0
```

在大多数的操作系统上，程序不允许访问地址为 0 的内存，因为该内存是操作系统保留的。然而，内存地址 0 有特别重要的意义，它表明该指针不指向一个可访问的内存位置。但按照惯例，如果指针包含空值（零值），则假定它不指向任何东西。

如需检查一个空指针，您可以使用 if 语句，如下所示：

if(ptr)     /* 如果 p 非空，则完成 */ if(!ptr)    /* 如果 p 为空，则完成 */

## C 指针详解

在 C 中，有很多指针相关的概念，这些概念都很简单，但是都很重要。下面列出了 C 程序员必须清楚的一些与指针相关的重要概念：

| 概念                                                         | 描述                                                     |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| [指针的算术运算](https://www.runoob.com/cprogramming/c-pointer-arithmetic.html) | 可以对指针进行四种算术运算：++、--、+、-                 |
| [指针数组](https://www.runoob.com/cprogramming/c-array-of-pointers.html) | 可以定义用来存储指针的数组。                             |
| [指向指针的指针](https://www.runoob.com/cprogramming/c-pointer-to-pointer.html) | C 允许指向指针的指针。                                   |
| [传递指针给函数](https://www.runoob.com/cprogramming/c-passing-pointers-to-functions.html) | 通过引用或地址传递参数，使传递的参数在调用函数中被改变。 |
| [从函数返回指针](https://www.runoob.com/cprogramming/c-return-pointer-from-functions.html) | C 允许函数返回指针到局部变量、静态变量和动态内存分配。   |

指针是一个变量，其值为另一个变量的地址，即，内存位置的直接地址。就像其他变量或常量一样，您必须在使用指针存储其他变量地址之前，对其进行声明。

要理解指针就要先理解计算机的内存。计算机内存会被划分为按顺序编号的内存单元。每个变量都是存储在内存单元中的，称之为地址。

```
#include <stdio.h>

int main ()
{
    int var = 20;   /* 实际变量的声明 此时的 VAR 这个变量是存在某个地址的，地址对应某个内存单元，该单元中存储了数据20 */
    int *ip;         /* 指针变量的声明 定义了一个指针 即一个内存单元的地址变量 */

    ip = &var;      /* 在指针变量中存储 var 的地址 就是将地址值赋值给指针这个变量*/

    /* 在指针变量中存储的地址 利用&符号直接输出了var所存储的数据的内存单元的地址*/
    printf("Address of var variable: %p\n", &var );
    
    /* 在指针变量中存储的地址 ip代表的是这个赋值到的地址的值 所以输出的是地址值 */
    printf("Address stored in ip variable: %p\n", ip );
    
    /* 使用指针访问值 *ip代表的是定义到这个内存单元之后，内存单元中所存储的数据的值也就是将20赋值给var中20这个值 */
    printf("Value of *ip variable: %d\n", *ip );

    return 0;
}
```

指针是一个变量，所以可以使用任何合法的变量名。在大多数的操作系统上，程序不允许访问地址为 0 的内存，因为该内存是操作系统保留的。

然而，内存地址 0 有特别重要的意义，它表明该指针不指向一个可访问的内存位置。

但按照惯例，如果指针包含空值（零值），则假定它不指向任何东西。

所有指针在创建时都要初始化，如果不知道他指向什么就将 0 赋值给他。必须初始化指针，没有被初始化的指针被称为失控指针(野指针）。

```
#include <stdio.h>

int main ()
{
    int *p = 0;
    int a ;
    p = &a;
    printf ("输入一个数字\n");
    scanf ("%d",p);
    printf("%d\n",*p);
}
```

实例定义了变量 a 和指针变量 p。**p = &a;**表示指针变量指向了变量 a，p 中存放的地址为 a 的地址 &a，*p 所指的是 p 中存放的地址 a 内存单元中的值。

```
/*按照偏移值访问函数形参内容实验*/
//二级指针
void Pros(char* a,int b,int e,char et)
{
    char **p=&a;
    //a==*p
    printf("%p %p %p %p \n%p\n",&a,p,a,*p,&b);
    printf("%s\n",*p);
    p++;
    printf("%d\n",*p);
    p++;
    printf("%d\n",*p);
    p++;
    printf("%c\n",*p);
    return;
}

//一级指针访问
void Test(char* a,int b)
{
    char *p=(char*)&a;
    //a!=*p;
    //printf("%p %p %p %p\n",&a,p,a,*p);
    //printf("%p\n",&b);
    //得出结果一级指针自加+1 二级指针自按照元素内容大小自加
    //printf("%d %p\n",*(++p),p);
    //printf("%d %p\n",*(p+8),p+8);
    //a=a[0]一个printf函数以'\0'结束
    //此时p=&a把元素首地址给了p或者说a只记录一个元素首地址的地址
    //同等汇编语句 a：db 'Hello' b:db '16'
    //所以 p=&a != p=a ;
    /*
    char *a="Hello";
    char *b=(char*)&a;
    printf("%p %p %p %p",&a,b,a,&(a[0]));
    */
    //printf("%c %p %p\n",*a,a,&(a[0]));
    //printf("%c %p %p\n",*(a+1),a+1,&(a[1]));
    printf("%c\n",*(*(char**)p));
    //if p=a; *p=a;
    p=a;
    printf("%s",p);
    return;
}

int main()
{
    //Pros("Hello",5,66666,'a');
    Test("Hello",16);
    //指针转换问题
    /*
    char *a="Hello";//&a变量里面存储着a所指向的变量地址
    //char **p=&a;
    char *b=(char*)&a;
    char **p=&a;
    printf("%p %p %p %p\n",&a,b,a,*b);
    printf("%p %c\n",&(*a),*(&(*(a+1))));
    printf("%p %c\n",a,*a);//此时a->H，*a=H;
    printf("%p %c\n",(*p),*(*p));
    //p=&a,*p=a所指向的第一个元素的地址还需要一解才能访问正确数据
    //所以1级指针需要解2次 所以进行强制转换
    printf("%c \n",*(*(char**)b));
    //原试解 现在b=&a,*b= &a->a所以如果此时想正确访问H必须在解
    */
    return 1;
}
```

**指针的一些复杂说明：**

-  **int p;** -- 这是一个普通的整型变量
-  **int \*p;** -- 首先从 p 处开始,先与*结合,所以说明 p 是一个指针, 然后再与 int 结合, 说明指针所指向的内容的类型为 int 型。所以 p 是一个返回整型数据的指针。
-  **int p[3]** -- 首先从 p 处开始,先与[] 结合,说明 p 是一个数组, 然后与 int 结合, 说明数组里的元素是整型的, 所以 p 是一个由整型数据组成的数组。
-  **int \*p[3];** -- 首先从 p 处开始, 先与 [] 结合, 因为其优先级比 * 高,所以 p 是一个数组, 然后再与 * 结合, 说明数组里的元素是指针类型, 然后再与 int 结合, 说明指针所指向的内容的类型是整型的, 所以 p 是一个由返回整型数据的指针所组成的数组。
-  **int (\*p)[3];** -- 首先从 p 处开始, 先与 * 结合,说明 p 是一个指针然后再与 [] 结合(与"()"这步可以忽略,只是为了改变优先级), 说明指针所指向的内容是一个数组, 然后再与int 结合, 说明数组里的元素是整型的。所以 p 是一个指向由整型数据组成的数组的指针。
-  **int \**p;** -- 首先从 p 开始, 先与 * 结合, 说是 p 是一个指针, 然后再与 * 结合, 说明指针所指向的元素是指针, 然后再与 int 结合, 说明该指针所指向的元素是整型数据。由于二级指针以及更高级的指针极少用在复杂的类型中, 所以后面更复杂的类型我们就不考虑多级指针了, 最多只考虑一级指针。
-  **int p(int);** -- 从 p 处起,先与 () 结合, 说明 p 是一个函数, 然后进入 () 里分析, 说明该函数有一个整型变量的参数, 然后再与外面的 int 结合, 说明函数的返回值是一个整型数据。
-  **int (\*p)(int);** -- 从 p 处开始, 先与指针结合, 说明 p 是一个指针, 然后与()结合, 说明指针指向的是一个函数, 然后再与()里的 int 结合, 说明函数有一个int 型的参数, 再与最外层的 int 结合, 说明函数的返回类型是整型, 所以 p 是一个指向有一个整型参数且返回类型为整型的函数的指针。
-  **int \*(\*p(int))[3];** -- 可以先跳过, 不看这个类型, 过于复杂从 p 开始,先与 () 结合, 说明 p 是一个函数, 然后进入 () 里面, 与 int 结合, 说明函数有一个整型变量参数, 然后再与外面的 * 结合, 说明函数返回的是一个指针, 然后到最外面一层, 先与[]结合, 说明返回的指针指向的是一个数组, 然后再与 * 结合, 说明数组里的元素是指针, 然后再与 int 结合, 说明指针指向的内容是整型数据。所以 p 是一个参数为一个整数据且返回一个指向由整型指针变量组成的数组的指针变量的函数。

指针实例说明：

```
int board[8][8];    /* int 数组的数组 */ 
int ** ptr;         /* 指向 int 指针的指针 */
int * risks[10];    /* 具有 10 个元素的数组, 每个元素是一个指向 int 的指针 */
int (* rusks) [10];  /* 一个指针, 指向具有 10 个元素的 int 数组 */
int * oof[3][4];    /* 一个 3 x 4 的数组, 每个元素是一个指向 int 的指针 */ 
int (* uuf) [3][4]; /* 一个指针, 指向 3 X 4 的 int 数组 */
int (* uof[3]) [4]; /* 一个具有 3 个元素的数组, 每个元素是一个指向具有 4 个元素的int 数组的指针 */ 
```

**指向函数的指针**

代码和数据是一样的，都需要占据一定内存，那当然也会有一个基地址，所以我们可以定义一个指针来指向这个基地址，这就是所谓的函数指针。

假设有函数：

```
double func(int a,char c);
double (*p)(int a,char c);
p=&func;
```

即可以定义一个函数指针。

**调用函数**

```
double s1=func(100,'x');
double s2=(*p)(100,'x');
```

上面两个语句是等价的。s

函数指针形式的传递，实质却是地址传递的一个例子：

```
#include <stdio.h>

void func1(int *a, int **b);

void func1(int *a, int **b)
{
   (*a)++;
   (*b)++;//这里虽然传进来的是指针的形式，但其实是指针c的地址，
          //可以认为这里本质还是值传递，只不过这个值是地址值
}

int main()
{
    int a[2] = {10, 20};
    int *b = &a[0];
    int *c = a+1;
    int **d = &c;
    
    func1(b, d);
    printf("a[0] = %d   a[1] = %d\n", a[0], a[1]);

    return 0;
}
```

执行结果：**a[0] = 11 a[1] = 20**

由上可知，虽然传递参数时，是以指针形式进行的，但有时候会发现其实还是值传递，是地址值的传递，特别是在多维数组进行参数传递的时候，特别容易出现这种情况。

**数组指针**

给定义好的数组中赋值时, 指针可以通过调整地址给数组赋值。

例: 创建一个包含 3 个元素的一维数组, 并给它赋值。

```
int* array0 = (int*)malloc(sizeof(int) *3);
for(int i=0; i<3; i++){
    scanf("%d", array0+i);
}
```

2016 年接触 C 语言 ，当年愣是看不懂！尤其是指针，到现在 2 年多了，有点小理解。回顾一下，一说到指针就会说到地址的概念，所以跟指针有关的概念，跟 C 的内存管理结合起来理解会更好点，下面说说如何更好的理解指针。

为什么叫指针，指针其实是一种很形象的比喻，下面说说我的个人理解。

int 变量存的是 int 型的值，char 变量存的是 char 型的值，而指针，它是一种特殊的变量，存的是内存地址，按照这个模板可以把它理解为：**“内存地址变量” 存的是 “内存地址”，等价于：“指针变量” 存的是 “内存地址”**

操作系统进行资源调度时，会根据这些变量存的地址去请求和使用那个地址代表的内存区域，这就仿佛像是这个变量存的地址指向了某片内存，人们用 “指针” 来统称所谓的 “内存地址变量”

因此，**任何跟指针有关的概念，都可以联系内存地址加以理解**，二者必然有联系，数组与指针，函数与指针，都是如此。

-  **内存是线性的**，内存以地址空间的形式呈现给我们看的，所以可以说所谓的地址空间也是线性的，指针存放的是内存地址，所以你可以对地址做 ++，或者 -- 这样的运算。
-  **两个指针不赋 NULL，是坏习惯**
-  **初始化指针不赋 NULL**，因为这样的指针会指向一片未知的区域，这样的指针不是空指针，但指向一片访问受限制的内存区域，你无法使用它，这样的情况下的指针，业界给了它一个形象的名字：“野指针”，而且难以调试，在许多编译器单步 debug 会出现奇怪的错误，但经常看见的 "Segmentation Fault" 这样的错误，实测当代码多的时候，这是一个非常蛋疼的错误，野指针就是成因之一，所以看到这样的错误，首先是想想，是否有某些指针没有初始化引起的
-  **free() 后指针不赋 NULL**，为指针分配内存后，指针便可以指向一片合法可使用的内存，但使用 free() 释放那片内存时，指针依旧存放着那片内存的地址，也就是依旧指向那片内存，但这片内存已经释放，不可访问，这时若不小心使用了这个指针，便会内存错误，又是会有奇怪的 bug ，代码几百行多点就会难以调试，业界给这样的指针也有个统称：“悬空指针”，为了避免这种蛋疼的情况出现，一定要释放内存后，给指向这片内存的指针，都赋值为 NULL，从中也可以看出，free() 这个函数释放内存时跟指向这片内存的指针并没有什么卵关系，不会连着把指针一起搞定掉的！ **珍爱生命，远离 "野指针" 与 "悬空指针" ！**
-  **多级指针**，指向指针的指针，有时人们也管它叫多维指针。既然指针变量是一个变量，指针变量能存变量的内存的地址。

像 int * 存 int 型变量的地址，char * 存 char 型的地址，那指针理所当然可以存指针变量的地址啊。

例如，int ** 存 int * 的地址，int *** 存 int ** 的地址。

这就是一个二级指针存一级指针的地址，三级指针存二级指针的地址，人们把这样的过程叫指向指针的指针，但其实也就是一个上一级的指针存了下一级的指针的地址而已。

因此，像上面说的，你存了它的地址，你就是指向它，所以：

-  二级指针存一级指针的地址，那么可以说二级指针指向一级指针
-  三级指针存二级指针的地址，那么可以说二级指针指向一级指针
-  多级指针用处多多， 这里暂不举例详细说明。

个人认为指针可以说是 C 的最伟大的特性，通过这样的一个模型可以形象地管理部分内存！

## 函数指针

函数指针是指向函数的指针变量。

通常我们说的指针变量是指向一个整型、字符型或数组等变量，而函数指针是指向函数。

函数指针可以像一般函数一样，用于调用函数、传递参数。

函数指针变量的声明：

```
typedef int (*fun_ptr)(int,int); // 声明一个指向同样参数、返回值的函数指针类型
```

### 实例

以下实例声明了函数指针变量 p，指向函数 max：

## 实例

\#include <stdio.h>  int max(int x, int y) {    return x > y ? x : y; }  int main(void) {    /* p 是函数指针 */    int (* p)(int, int) = & max; // &可以省略    int a, b, c, d;     printf("请输入三个数字:");    scanf("%d %d %d", & a, & b, & c);     /* 与直接调用函数等价，d = max(max(a, b), c) */    d = p(p(a, b), c);      printf("最大的数字是: %d\n", d);     return 0; }

编译执行，输出结果如下：

```
请输入三个数字:1 2 3
最大的数字是: 3
```

------

## 回调函数

### 函数指针作为某个函数的参数



函数指针变量可以作为某个函数的参数来使用的，回调函数就是一个通过函数指针调用的函数。

简单讲：回调函数是由别人的函数执行时调用你实现的函数。

> 以下是来自知乎作者常溪玲的解说：
>
> 你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里，你的电话号码就叫回调函数，你把电话留给店员就叫登记回调函数，店里后来有货了叫做触发了回调关联的事件，店员给你打电话叫做调用回调函数，你到店里去取货叫做响应回调事件。

### 实例

实例中 populate_array 函数定义了三个参数，其中第三个参数是函数的指针，通过该函数来设置数组的值。

实例中我们定义了回调函数 getNextRandomValue，它返回一个随机值，它作为一个函数指针传递给 populate_array 函数。

populate_array 将调用 10 次回调函数，并将回调函数的返回值赋值给数组。

## 实例

\#include <stdlib.h>   #include <stdio.h>  // 回调函数 void populate_array(int *array, size_t arraySize, int (*getNextValue)(void)) {    for (size_t i=0; i<arraySize; i++)        array[i] = getNextValue(); }  // 获取随机值 int getNextRandomValue(void) {    return rand(); }  int main(void) {    int myarray[10];    populate_array(myarray, 10, getNextRandomValue);    for(int i = 0; i < 10; i++) {        printf("%d ", myarray[i]);    }    printf("\n");    return 0; }

编译执行，输出结果如下：

```
16807 282475249 1622650073 984943658 1144108930 470211272 101027544 1457850878 1458777923 2007237709 
```

有关于 **size_t**:

size_t 是一种数据类型，近似于无符号整型，但容量范围一般大于 int 和 unsigned。这里使用 size_t 是为了保证 arraysize 变量能够有足够大的容量来储存可能大的数组。

size_t 类型在C语言标准库函数原型使用的很多，数值范围一般是要大于int和unsigned.

但凡不涉及负值范围的表示size取值的，都可以用size_t；比如array[size_t]。

size_t 在stddef.h头文件中定义。

在其他常见的宏定义以及函数中常用到有：

1，sizeof运算符返回的结果是size_t类型；

2，void *malloc(size_t size)...

更标准写法 GCC 直接能过，指针变量用指针变量的方式处理。

```
#include <stdlib.h>
#include <stdio.h>

// 回调函数
void populate_array(int *array, int arraySize, int (*getNextValue)(void))
{
    int i;
    for(i=0; i<arraySize; i++){
        *(array+i) = getNextValue();
    }
}

// 获取随机值
int getNextRandomValue(void)
{
    return rand();
}

int main(void)
{
    int myarray[10];
    int i;
    populate_array(myarray, 10, getNextRandomValue);
    for(i=0; i<10; i++) {
        printf("%d ", myarray[i]);
    }
    printf("\n");
    return 0;
}
```

**引用这段话：**你到一个商店买东西，刚好你要的东西没有货，于是你在店员那里留下了你的电话，过了几天店里有货了，店员就打了你的电话，然后你接到电话后就到店里去取了货。在这个例子里，你的电话号码就叫回调函数，你把电话留给店员就叫登记回调函数，店里后来有货了叫做触发了回调关联的事件，店员给你打电话叫做调用回调函数，你到店里去取货叫做响应回调事件。

**可以这样理解，号码就是回调函数的名称，手机就是函数指针，有手机有了号码，就可以进行回调。**

补充下带参的回调函数：

```
#include <stdio.h>

int sum(int a,int b){
    return a+b;
}

//sum2回调sum函数
int sum2(int num,int (*sum)(int,int),int a,int b){
    return  num * sum(a,b);
}

int main()
{
    int (* p)(int a,int b)=sum;

    printf("SUM=%d\n",sum(1,2));
    printf("SUM2= %d\n",sum2(2,sum,1,2));
    return 0;
}
```

**关于回调函数的一点思考**

在 A 函数中将参数与 C 函数传递给 B 函数，B函数调用 C 函数；B 函数调用的动作称为回调，C 函数称为回调函数。

举栗

```
#include <stdio.h>

// 回调函数a
void callback_a(int var_a){    
    printf("Call callback_a, var: %d\n\n", var_a);
}

// 回调函数b
void callback_b(int var_b){    
    printf("Call callback_b, var: %d\n\n", var_b);
}

// 回调动作
void callback_act(int x, void (*callback) (int var)){    
    printf("Call callback_act, var_x: %d\n\n", x);    
    callback(x);
}


void main(){    
    int a = 1, b = 2;    
    callback_act(a, callback_a);    
    callback_act(b, callback_b);    
    printf("Main program has done.");
}
```

执行结果：

```
Call callback_act, var_x: 1
Call callback_a, var: 1
Call callback_act, var_x: 2
Call callback_b, var: 2
Main program has done.
```

仔细想想回调函数的作用完全可以通过普通函数来达到，其实回调函数最大的意义在于解耦，降低了代码之间的耦合度。

# C 字符串

在 C 语言中，字符串实际上是使用 **null** 字符 '\0' 终止的一维字符数组。因此，一个以 null 结尾的字符串，包含了组成字符串的字符。

下面的声明和初始化创建了一个 "Hello" 字符串。由于在数组的末尾存储了空字符，所以字符数组的大小比单词 "Hello" 的字符数多一个。

```
char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

依据数组初始化规则，您可以把上面的语句写成以下语句：

```
char greeting[] = "Hello";
```

以下是 C/C++ 中定义的字符串的内存表示：

![C/C++ 中的字符串表示](https://www.runoob.com/wp-content/uploads/2014/08/string_representation.jpg)

其实，您不需要把 *null* 字符放在字符串常量的末尾。C 编译器会在初始化数组时，自动把 '\0' 放在字符串的末尾。让我们尝试输出上面的字符串：

## 实例

\#include <stdio.h>  int main () {   char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};    printf("Greeting message: %s\n", greeting );    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Greeting message: Hello
```

C 中有大量操作字符串的函数：

| 序号 | 函数 & 目的                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **strcpy(s1, s2);** 复制字符串 s2 到字符串 s1。              |
| 2    | **strcat(s1, s2);** 连接字符串 s2 到字符串 s1 的末尾。       |
| 3    | **strlen(s1);** 返回字符串 s1 的长度。                       |
| 4    | **strcmp(s1, s2);** 如果 s1 和 s2 是相同的，则返回 0；如果 s1<s2 则返回小于 0；如果 s1>s2 则返回大于 0。 |
| 5    | **strchr(s1, ch);** 返回一个指针，指向字符串 s1 中字符 ch 的第一次出现的位置。 |
| 6    | **strstr(s1, s2);** 返回一个指针，指向字符串 s1 中字符串 s2 的第一次出现的位置。 |

下面的实例使用了上述的一些函数：

## 实例

\#include <stdio.h> #include <string.h>  int main () {   char str1[12] = "Hello";   char str2[12] = "World";   char str3[12];   int  len ;    /* 复制 str1 到 str3 */   strcpy(str3, str1);   printf("strcpy( str3, str1) :  %s\n", str3 );    /* 连接 str1 和 str2 */   strcat( str1, str2);   printf("strcat( str1, str2):   %s\n", str1 );    /* 连接后，str1 的总长度 */   len = strlen(str1);   printf("strlen(str1) :  %d\n", len );    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
strcpy( str3, str1) :  Hello
strcat( str1, str2):   HelloWorld
strlen(str1) :  10
```

您可以在 C 标准库中找到更多字符串相关的函数。

本节涉及函数英文全称

strcmp: string compare 

strcat: string catenate 

strcpy: string copy 

strlen: string length 

strlwr: string lowercase 

strupr: string upercase

字符串在以如下输入进行初始化的时候需要对 **\0** 特别注意：

```
char greeting[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

如果没有在字符数组最后增加 **\0** 的话输出结果有误：

```
// 初始化字符串
char greeting[5] = { 'H', 'e', 'l', 'l', 'o' };
printf("Greeting message: %s\n", greeting);
```

输出结果:

```
Greeting message: Hello烫烫烫?侵7(?╔?╚╔╔
```

在使用不定长数组初始化字符串时默认结尾为 **\0**

```
char greeting[] = "Hello";
printf("Greeting message: %s, greeting[] Length: %d\n", greeting, sizeof(greeting));
```

输出结果:

```
Greeting message: Hello, greeting[] Length: 6
```

**strlen 与 sizeof的区别：**

strlen 是函数，sizeof 是运算操作符，二者得到的结果类型为 size_t，即 unsigned int 类型。

sizeof 计算的是变量的大小，不受字符 **\0**

而 strlen 计算的是字符串的长度，以 **\0** 作为长度判定依据。

> 更多参考内容：
>
> -  [利用 strlen 和 sizeof 求取字符串长度注意事项](https://www.runoob.com/w3cnote/strlen-and-sizeof.html)
> -  [C++ sizeof 与 strlen的区别](https://www.runoob.com/note/27755)

**'a'** 表示是一个字符，**"a"** 表示一个字符串相当于 **'a'+'\0'**;

**''** 里面只能放一个字符;

**""** 里面表示是字符串系统自动会在串末尾补一个 0。

1、字符串的本质是以 **\0** 结束的字符数组。

2、字符串的字面量是常量，比如"hello world"。

3、字符串的定义与初始化。

**字符数组：**

1）数组形式

```
char hi[] = {'h','e','l','l','o','\0'};
```

2）直接赋首地址：

```
char hi[50] ="hello" ;
```

3）省略长度

```
char hi[] = "hello";
char hi[] = { "hello" };
```

字符指针：(注意指针不能直接赋给数组)

```
char *hi = "hello";
```

4、字符串遍历：

```
char hi[] = "hello";
```

for 遍历：

```
for(i==0, i<6,i++)
{
    printf("%c",hi[i]);
}
```

直接输出：

```
printf(hi);
```

字符串格式（%s）输出：

```
printf("%s",hi);
```

# C 结构体

C 数组允许定义可存储相同类型数据项的变量，**结构**是 C 编程中另一种用户自定义的可用的数据类型，它允许您存储不同类型的数据项。

结构用于表示一条记录，假设您想要跟踪图书馆中书本的动态，您可能需要跟踪每本书的下列属性：

- Title
- Author
- Subject
- Book ID

## 定义结构

为了定义结构，您必须使用 **struct** 语句。struct 语句定义了一个包含多个成员的新的数据类型，struct 语句的格式如下：

struct tag {     member-list    member-list     member-list      ... } variable-list ;

**tag** 是结构体标签。

**member-list** 是标准的变量定义，比如 int i; 或者 float f，或者其他有效的变量定义。

**variable-list** 结构变量，定义在结构的末尾，最后一个分号之前，您可以指定一个或多个结构变量。下面是声明 Book 结构的方式：

struct Books {   char  title[50];   char  author[50];   char  subject[100];   int   book_id; } book;

在一般情况下，**tag、member-list、variable-list** 这 3 部分至少要出现 2 个。以下为实例：

//此声明声明了拥有3个成员的结构体，分别为整型的a，字符型的b和双精度的c //同时又声明了结构体变量s1 //这个结构体并没有标明其标签 struct  {    int a;    char b;    double c; } s1;  //此声明声明了拥有3个成员的结构体，分别为整型的a，字符型的b和双精度的c //结构体的标签被命名为SIMPLE,没有声明变量 struct SIMPLE {    int a;    char b;    double c; }; //用SIMPLE标签的结构体，另外声明了变量t1、t2、t3 struct SIMPLE t1, t2[20], *t3;  //也可以用typedef创建新类型 typedef struct {    int a;    char b;    double c;  } Simple2; //现在可以用Simple2作为类型声明新的结构体变量 Simple2 u1, u2[20], *u3;

在上面的声明中，第一个和第二声明被编译器当作两个完全不同的类型，即使他们的成员列表是一样的，如果令 t3=&s1，则是非法的。

结构体的成员可以包含其他结构体，也可以包含指向自己结构体类型的指针，而通常这种指针的应用是为了实现一些更高级的数据结构如链表和树等。

//此结构体的声明包含了其他的结构体 struct COMPLEX {    char string[100];    struct SIMPLE a; };  //此结构体的声明包含了指向自己类型的指针 struct NODE {    char string[100];    struct NODE *next_node; };

如果两个结构体互相包含，则需要对其中一个结构体进行不完整声明，如下所示：

struct B;    //对结构体B进行不完整声明  //结构体A中包含指向结构体B的指针 struct A {    struct B *partner;    //other members; };  //结构体B中包含指向结构体A的指针，在A声明完后，B也随之进行声明 struct B {    struct A *partner;    //other members; };

------

## 结构体变量的初始化

和其它类型变量一样，对结构体变量可以在定义时指定初始值。

## 实例

\#include <stdio.h>  struct Books {   char  title[50];   char  author[50];   char  subject[100];   int   book_id; } book = {"C 语言", "RUNOOB", "编程语言", 123456};  int main() {    printf("title : %s\nauthor: %s\nsubject: %s\nbook_id: %d\n", book.title, book.author, book.subject, book.book_id); }

执行输出结果为：

```
title : C 语言
author: RUNOOB
subject: 编程语言
book_id: 123456
```

## 访问结构成员

为了访问结构的成员，我们使用**成员访问运算符（.）**。成员访问运算符是结构变量名称和我们要访问的结构成员之间的一个句号。您可以使用 **struct** 关键字来定义结构类型的变量。下面的实例演示了结构的用法：

## 实例

\#include <stdio.h> #include <string.h>  struct Books {   char  title[50];   char  author[50];   char  subject[100];   int   book_id; };  int main( ) {   struct Books Book1;        /* 声明 Book1，类型为 Books */   struct Books Book2;        /* 声明 Book2，类型为 Books */    /* Book1 详述 */   strcpy( Book1.title, "C Programming");   strcpy( Book1.author, "Nuha Ali");    strcpy( Book1.subject, "C Programming Tutorial");   Book1.book_id = 6495407;    /* Book2 详述 */   strcpy( Book2.title, "Telecom Billing");   strcpy( Book2.author, "Zara Ali");   strcpy( Book2.subject, "Telecom Billing Tutorial");   Book2.book_id = 6495700;    /* 输出 Book1 信息 */   printf( "Book 1 title : %s\n", Book1.title);   printf( "Book 1 author : %s\n", Book1.author);   printf( "Book 1 subject : %s\n", Book1.subject);   printf( "Book 1 book_id : %d\n", Book1.book_id);    /* 输出 Book2 信息 */   printf( "Book 2 title : %s\n", Book2.title);   printf( "Book 2 author : %s\n", Book2.author);   printf( "Book 2 subject : %s\n", Book2.subject);   printf( "Book 2 book_id : %d\n", Book2.book_id);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Book 1 title : C Programming
Book 1 author : Nuha Ali
Book 1 subject : C Programming Tutorial
Book 1 book_id : 6495407
Book 2 title : Telecom Billing
Book 2 author : Zara Ali
Book 2 subject : Telecom Billing Tutorial
Book 2 book_id : 6495700
```

## 结构作为函数参数

您可以把结构作为函数参数，传参方式与其他类型的变量或指针类似。您可以使用上面实例中的方式来访问结构变量：

## 实例

\#include <stdio.h> #include <string.h>  struct Books {   char  title[50];   char  author[50];   char  subject[100];   int   book_id; };  /* 函数声明 */ void printBook( struct Books book ); int main( ) {   struct Books Book1;        /* 声明 Book1，类型为 Books */   struct Books Book2;        /* 声明 Book2，类型为 Books */    /* Book1 详述 */   strcpy( Book1.title, "C Programming");   strcpy( Book1.author, "Nuha Ali");    strcpy( Book1.subject, "C Programming Tutorial");   Book1.book_id = 6495407;    /* Book2 详述 */   strcpy( Book2.title, "Telecom Billing");   strcpy( Book2.author, "Zara Ali");   strcpy( Book2.subject, "Telecom Billing Tutorial");   Book2.book_id = 6495700;    /* 输出 Book1 信息 */   printBook( Book1 );    /* 输出 Book2 信息 */   printBook( Book2 );    return 0; } void printBook( struct Books book ) {   printf( "Book title : %s\n", book.title);   printf( "Book author : %s\n", book.author);   printf( "Book subject : %s\n", book.subject);   printf( "Book book_id : %d\n", book.book_id); }

当上面的代码被编译和执行时，它会产生下列结果：

```
Book title : C Programming
Book author : Nuha Ali
Book subject : C Programming Tutorial
Book book_id : 6495407
Book title : Telecom Billing
Book author : Zara Ali
Book subject : Telecom Billing Tutorial
Book book_id : 6495700
```

## 指向结构的指针

您可以定义指向结构的指针，方式与定义指向其他类型变量的指针相似，如下所示：

```
struct Books *struct_pointer;
```

现在，您可以在上述定义的指针变量中存储结构变量的地址。为了查找结构变量的地址，请把 & 运算符放在结构名称的前面，如下所示：

```
struct_pointer = &Book1;
```

为了使用指向该结构的指针访问结构的成员，您必须使用 -> 运算符，如下所示：

```
struct_pointer->title;
```

让我们使用结构指针来重写上面的实例，这将有助于您理解结构指针的概念：

## 实例

\#include <stdio.h> #include <string.h>  struct Books {   char  title[50];   char  author[50];   char  subject[100];   int   book_id; };  /* 函数声明 */ void printBook( struct Books *book ); int main( ) {   struct Books Book1;        /* 声明 Book1，类型为 Books */   struct Books Book2;        /* 声明 Book2，类型为 Books */    /* Book1 详述 */   strcpy( Book1.title, "C Programming");   strcpy( Book1.author, "Nuha Ali");    strcpy( Book1.subject, "C Programming Tutorial");   Book1.book_id = 6495407;    /* Book2 详述 */   strcpy( Book2.title, "Telecom Billing");   strcpy( Book2.author, "Zara Ali");   strcpy( Book2.subject, "Telecom Billing Tutorial");   Book2.book_id = 6495700;    /* 通过传 Book1 的地址来输出 Book1 信息 */   printBook( &Book1 );    /* 通过传 Book2 的地址来输出 Book2 信息 */   printBook( &Book2 );    return 0; } void printBook( struct Books *book ) {   printf( "Book title : %s\n", book->title);   printf( "Book author : %s\n", book->author);   printf( "Book subject : %s\n", book->subject);   printf( "Book book_id : %d\n", book->book_id); }

当上面的代码被编译和执行时，它会产生下列结果：

```
Book title : C Programming
Book author : Nuha Ali
Book subject : C Programming Tutorial
Book book_id : 6495407
Book title : Telecom Billing
Book author : Zara Ali
Book subject : Telecom Billing Tutorial
Book book_id : 6495700
```

## 位域

有些信息在存储时，并不需要占用一个完整的字节，而只需占几个或一个二进制位。例如在存放一个开关量时，只有 0 和 1 两种状态，用 1 位二进位即可。为了节省存储空间，并使处理简便，C 语言又提供了一种数据结构，称为"位域"或"位段"。

所谓"位域"是把一个字节中的二进位划分为几个不同的区域，并说明每个区域的位数。每个域有一个域名，允许在程序中按域名进行操作。这样就可以把几个不同的对象用一个字节的二进制位域来表示。

典型的实例：

- 用 1 位二进位存放一个开关量时，只有 0 和 1 两种状态。
- 读取外部文件格式——可以读取非标准的文件格式。例如：9 位的整数。

### 位域的定义和位域变量的说明

位域定义与结构定义相仿，其形式为：

```
struct 位域结构名 
{

 位域列表

};
```

其中位域列表的形式为：

```
类型说明符 位域名: 位域长度 
```

例如：

struct bs{    int a:8;    int b:2;    int c:6; }data;

说明 data 为 bs 变量，共占两个字节。其中位域 a 占 8 位，位域 b 占 2 位，位域 c 占 6 位。

让我们再来看一个实例：

struct packed_struct {  unsigned int f1:1;  unsigned int f2:1;  unsigned int f3:1;  unsigned int f4:1;  unsigned int type:4;  unsigned int my_int:9; } pack;

在这里，packed_struct 包含了 6 个成员：四个 1 位的标识符 f1..f4、一个 4 位的 type 和一个 9 位的 my_int。

**对于位域的定义尚有以下几点说明：**

- 一个位域存储在同一个字节中，如一个字节所剩空间不够存放另一位域时，则会从下一单元起存放该位域。也可以有意使某位域从下一单元开始。例如：

  struct bs{    unsigned a:4;    unsigned  :4;    /* 空域 */    unsigned b:4;    /* 从下一单元开始存放 */    unsigned c:4 }

  在这个位域定义中，a 占第一字节的 4 位，后 4 位填 0 表示不使用，b 从第二字节开始，占用 4 位，c 占用 4 位。

- 由于位域不允许跨两个字节，因此位域的长度不能大于一个字节的长度，也就是说不能超过8位二进位。如果最大长度大于计算机的整数字长，一些编译器可能会允许域的内存重叠，另外一些编译器可能会把大于一个域的部分存储在下一个字中。

- 位域可以是无名位域，这时它只用来作填充或调整位置。无名的位域是不能使用的。例如：

  struct k{    int a:1;    int  :2;    /* 该 2 位不能使用 */    int b:3;    int c:2; };

从以上分析可以看出，位域在本质上就是一种结构类型，不过其成员是按二进位分配的。

### 位域的使用

位域的使用和结构成员的使用相同，其一般形式为：

```
位域变量名.位域名
位域变量名->位域名
```

位域允许用各种格式输出。

请看下面的实例：

## 实例

main(){    struct bs{        unsigned a:1;        unsigned b:3;        unsigned c:4;    } bit,*pbit;    bit.a=1;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */    bit.b=7;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */    bit.c=15;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */    printf("%d,%d,%d\n",bit.a,bit.b,bit.c);    /* 以整型量格式输出三个域的内容 */    pbit=&bit;    /* 把位域变量 bit 的地址送给指针变量 pbit */    pbit->a=0;    /* 用指针方式给位域 a 重新赋值，赋为 0 */    pbit->b&=3;    /* 使用了复合的位运算符 "&="，相当于：pbit->b=pbit->b&3，位域 b 中原有值为 7，与 3 作按位与运算的结果为 3（111&011=011，十进制值为 3） */    pbit->c|=1;    /* 使用了复合位运算符"|="，相当于：pbit->c=pbit->c|1，其结果为 15 */    printf("%d,%d,%d\n",pbit->a,pbit->b,pbit->c);    /* 用指针方式输出了这三个域的值 */ }

上例程序中定义了位域结构 bs，三个位域为 a、b、c。说明了 bs 类型的变量 bit 和指向 bs 类型的指针变量 pbit。这表示位域也是可以使用指针的。

结构体中成员变量分配的空间是按照成员变量中占用空间最大的来作为分配单位,同样成员变量的存储空间也是不能跨分配单位的,如果当前的空间不足,则会存储到下一个分配单位中。

```
#include <stdio.h>

typedef struct
{
    unsigned char a;
    unsigned int  b;
    unsigned char c;
} debug_size1_t;
typedef struct
{
    unsigned char a;
    unsigned char b;
    unsigned int  c;
} debug_size2_t;

int main(void)
{
    printf("debug_size1_t size=%lu,debug_size2_t size=%lu\r\n", sizeof(debug_size1_t), sizeof(debug_size2_t));
    return 0;
}
```

编译执行输出结果：

```
debug_size1_t size=12,debug_size2_t size=8
```

结构体占用存储空间,以32位机为例

-  1.debug_size1_t 存储空间分布为a(1byte)+空闲(3byte)+b(4byte)+c(1byte)+空闲(3byte)=12(byte)。
-  1.debug_size2_t 存储空间分布为a(1byte)+b(1byte)+空闲(2byte)+c(4byte)=8(byte)。

**结构体数组:**

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAXTITL 41
#define MAXAUTL 31
#define MAVXBKS 100

char * s_gets(char * st, int n);
struct book {
    char title[MAXTITL];
    char author[MAXAUTL];
    float value;
};

int main()
{
    struct book library[MAVXBKS];    //book类型的结构体数组
    int i;
    int index;
    printf("请问你要录入多少本书的信息\n");
    do
    {
        scanf("%d", &index);
    } while (index > MAVXBKS);
    getchar();
    for (i = 0; i < index; i++)
        {
        printf("请输入第%d本书的名称:\n",i+1);
        s_gets(library[i].title, MAXTITL);
        printf("输入其作者的名字:\n");
        s_gets(library[i].author, MAXAUTL);
        printf("请输入书本的价格:\n");
        scanf("%f", &library[i].value);
        getchar();
    }
    for (i = 0; i < index; i++)
    {
            printf("%d\t%s  是  %s 写的 定价为%f元\n", i,library[i].title, library[i].author, library[i].value);
    }
    system("pause");
    return 0;
}

char * s_gets(char * st, int n)
{
    char * ret_val;
    char * find;
    ret_val = fgets(st, n, stdin);
    if (ret_val)
    {
        find = strchr(st, '\n');    //查找换行符
        if (find)    //  查找地址不为空
            *find = '\0';    //在此处放入一个空字符
        else
            while (getchar() != '\n')
        continue;    //处理剩余字符
    }
    return ret_val;
}
```

可以在声明结构体时初始化结构体变量:

```
#include <stdio.h>

int main(void)
{
    struct Student {
        char name[50];
        int gender;
        int age;
    } student2 = {"张三",0,30};
    struct Student student1;
    printf("name:\n");
    scanf("%s",student1.name);
    printf("gender:\n");
    scanf("%d",&student1.gender);
    printf("age:\n");
    scanf("%d",&student1.age);
    printf("student1 >>name = %s, gender = %d, age = %d\n", student1.name, student1.gender, student1.age);
    printf("student2 >>name = %s, gender = %d, age = %d\n", student2.name, student2.gender, student2.age);
}
```

## 结构体数组

一个结构体变量中可以存放一组数据（如一个学生的学号，姓名，成绩等数据）。如果有10个学生的数据需要参加运算，显然应该用数组，这就是结构体数组。结构体数组与以前介绍过的数据值型数组不同之处在于每个数组元素都一个结构体类型的数据，它们分别包括各个成员（分量）项。

### 定义结构体数组

和定义结构体变量的方法相仿，只需说明其为数组即可。

```
struct student
{
    int num;
    char name[20];
    char sex;
    int age;
    float score;
    char addr[30];
};
struct student stu[3];
```

以上定义了一个数组 stu，其元素为 struct student 类型数据，数组有 3 个元素。也可以直接定义一个结构体数组。如：

```
struct student
{
    int num;
    ....

}stu[3];

或

struct

{
    int num;
    　...
}stu[3];
```

### 结构体数组的初始化

与其它类型数组一样，对结构体数组可以初始化如：

```
struct student
{
    int mum;
    char name[20];
    char sex;
    int age;
    float score;
    char addr[30];
}stu[3] = {{10101,"Li Lin", 'M', 18, 87.5, "103 Beijing Road"},
            {10101,"Li Lin", 'M', 18, 87.5, "103 Beijing Road"},
            {10101,"Li Lin", 'M', 18, 87.5, "103 Beijing Road"}};
```

定义数组 stu 时，元素个数可以不指定，即写成以下形式：

```
stu[] = {{...},{...},{...}};
```

编译时，系统会根据给出初值的结构体常量的个数来确定数组元素的个数。

当然，数组的初始化也可以用以下形式：

```
struct student
{
    int num;
    ...
};
struct student stu[] = {{...},{...},{...}};
```

即先声明结构体类型，然后定义数组为该结构体类型，在定义数组时初始化。

从以上可以看到，结构体数组初始化的一般形式是在定义数组的后面加上：

### 结构体数组应用举例

下面例子说明结构体数组的定义和引用。

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

struct person
{
    char name[20];
    int count;

}leader[3] = {{"Li", 0},
            　{"Zhang", 0},
            　{"Fun", 0}};

void main()
{
    int i, j;
    char leader_name[20];
    for(i = 1; i<= 10;i++)
    {
        scanf("%s", leader_name);
        for(j=0;j<3;j++)
            if(strcmp(leader_name, leader[j].name) == 0)
                leader[j].count ++;
    }
    printf("\n");
    for(i=0;i<3;i++)
        printf("%5s: %d\n", leader[i].name, leader[i].count);
    system("pause");
}
```

运行结果如下：

```
LI
Li
Fun
Zhang
Zhang
Fun
Li
Fun
Zhang
Li

   Li: 3
Zhang: 3
  Fun: 3
```

> 更多内容参考：[C 结构体详解](https://www.runoob.com/w3cnote/c-structures-intro.html)

使用结构数组存储书名/作者，结构体指针访问成员时，也可以对指针解引用再访问，如：***struct_pointer.title;**(上面范例)。

```
#include<stdio.h>
#include<string.h>

char * s_gets(char*st, int n);

#define MAXTITL 40
#define MAXAUTL 40
#define MAXBKS 100  // 书籍的最大数量

struct book {   //简历 book 模板
    char title[MAXTITL];
    char author[MAXAUTL];
    float value;
};

int main(void)
{
    struct book library[MAXBKS];  //book 结构类型数组
    int count = 0;
    int index;
    printf("请输入书名：\n");
    printf("按下 [enter] 键结束输入。\n");
    while (count < MAXBKS && s_gets(library[count].title, MAXTITL) != NULL && library[count].title[0] != '\0')
    {
        printf("请输入作者：\n");
        s_gets(library[count].author, MAXAUTL);
        printf("请输入价格：\n");
        scanf("%f", &library[count++].value);
        while (getchar() != '\n')
            continue;  //清理输入行
        if (count < MAXBKS)
            printf("输入下一本书。\n");
    }
    if (count > 0)  // 如果数组内有存书籍
    {
        printf("书的列表:\n");
        for (index = 0; index < count; index++)  // 遍历已存入的书籍,
            printf("%s - %s:$%.2f\n", library[index].title, library[index].author, library[index].value); // 将内容打印出来
    }
    else
        printf("没有书。\n");  // 否则就打印没书
    return 0;
}

char * s_gets(char * st, int n)  //输入文本(作家)函数
{
    char * ret_val;
    char * find;
    ret_val = fgets(st, n, stdin);
    if (ret_val)
    {
        find = strchr(st, '\n');  //查找换行符
        if (find)  //如果地址不是NULL
            *find = '\0';  //在此处放置一个空字符
        else
            while (getchar() != '\n')
                continue;
    }
    return ret_val;
}
```

**对一楼所讲内容的补充：结构体内存大小对齐原则**

1. 结构体变量的首地址能够被其最宽基本类型成员的大小所整除。
2. 结构体每个成员相对于结构体首地址的偏移量(offset)都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节(internal adding)。即结构体成员的末地址减去结构体首地址(第一个结构体成员的首地址)得到的偏移量都要是对应成员大小的整数倍。
3. 结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要编译器会在成员末尾加上填充字节。

# C 共用体

**共用体**是一种特殊的数据类型，允许您在相同的内存位置存储不同的数据类型。您可以定义一个带有多成员的共用体，但是任何时候只能有一个成员带有值。共用体提供了一种使用相同的内存位置的有效方式。

## 定义共用体

为了定义共用体，您必须使用 **union** 语句，方式与定义结构类似。union 语句定义了一个新的数据类型，带有多个成员。union 语句的格式如下：

union [union tag] {   member definition;   member definition;   ...   member definition; } [one or more union variables];

**union tag** 是可选的，每个 member definition 是标准的变量定义，比如 int i; 或者 float f; 或者其他有效的变量定义。在共用体定义的末尾，最后一个分号之前，您可以指定一个或多个共用体变量，这是可选的。下面定义一个名为 Data 的共用体类型，有三个成员 i、f 和 str：

union Data {   int i;   float f;   char  str[20]; } data;

现在，**Data** 类型的变量可以存储一个整数、一个浮点数，或者一个字符串。这意味着一个变量（相同的内存位置）可以存储多个多种类型的数据。您可以根据需要在一个共用体内使用任何内置的或者用户自定义的数据类型。

共用体占用的内存应足够存储共用体中最大的成员。例如，在上面的实例中，Data 将占用 20 个字节的内存空间，因为在各个成员中，字符串所占用的空间是最大的。下面的实例将显示上面的共用体占用的总内存大小：

## 实例

\#include <stdio.h> #include <string.h>  union Data {   int i;   float f;   char  str[20]; };  int main( ) {   union Data data;            printf( "Memory size occupied by data : %d\n", sizeof(data));    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Memory size occupied by data : 20
```

## 访问共用体成员

为了访问共用体的成员，我们使用**成员访问运算符（.）**。成员访问运算符是共用体变量名称和我们要访问的共用体成员之间的一个句号。您可以使用 **union** 关键字来定义共用体类型的变量。下面的实例演示了共用体的用法：

## 实例

\#include <stdio.h> #include <string.h>  union Data {   int i;   float f;   char  str[20]; };  int main( ) {   union Data data;            data.i = 10;   data.f = 220.5;   strcpy( data.str, "C Programming");    printf( "data.i : %d\n", data.i);   printf( "data.f : %f\n", data.f);   printf( "data.str : %s\n", data.str);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
data.i : 1917853763
data.f : 4122360580327794860452759994368.000000
data.str : C Programming
```

在这里，我们可以看到共用体的 **i** 和 **f** 成员的值有损坏，因为最后赋给变量的值占用了内存位置，这也是 **str** 成员能够完好输出的原因。现在让我们再来看一个相同的实例，这次我们在同一时间只使用一个变量，这也演示了使用共用体的主要目的：

## 实例

\#include <stdio.h> #include <string.h>  union Data {   int i;   float f;   char  str[20]; };  int main( ) {   union Data data;            data.i = 10;   printf( "data.i : %d\n", data.i);      data.f = 220.5;   printf( "data.f : %f\n", data.f);      strcpy( data.str, "C Programming");   printf( "data.str : %s\n", data.str);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
data.i : 10
data.f : 220.500000
data.str : C Programming
```

在这里，所有的成员都能完好输出，因为同一时间只用到一个成员。

**结构体与共用体**

结构体变量所占内存长度是其中最大字段大小的整数倍（参考：[结构体大小的计算](https://www.runoob.com/w3cnote/struct-size.html)）。

共用体变量所占的内存长度等于最长的成员变量的长度。例如，教程中定义的共用体Data各占20个字节（因为char str[20]变量占20个字节）,而不是各占4+4+20=28个字节。

```
union Data
{
   int i;
   float f;
   char  str[20];
} data;  
```

## 共用体作用

节省内存，有两个很长的数据结构，不会同时使用，比如一个表示老师，一个表示学生，如果要统计教师和学生的情况用结构体的话就有点浪费了！用共用体的话，只占用最长的那个数据结构所占用的空间，就足够了！

## 共用体应用场景

通信中的数据包会用到共用体:因为不知道对方会发一个什么包过来，用共用体的话就很简单了，定义几种格式的包，收到包之后就可以直接根据包的格式取出数据。

```
union Data{
    int i;
    float f;
    char str[9];
    double d;
}data;
```

共用体所占内存**并非是**9个char，即9个字节，**而是double的两倍，即16个字节，**而且每次输出都是它前面离它最近的值

编程时经常会需要判断机器是大端机还是小端机，此时使用union就非常方便：

```
union
{
    char str;
    int data;
};
data=0x01020304;
if(str==0x01)
{
    cout<< "此机器是大端！"<<endl;
}
else if(str==0x04){
    cout<<"此机器是小端！"<<endl;
}
else{
    cout <<" 暂无法判断此机器类型！"<<endl;
}
```

注：大端机高位存在低位，小端机反之

看了前面大家的内存占用大小计算，都没有分析到点，下面给出部分概念：

- **位：**"位(bit)"是电子计算机中最小的数据单位。每一位的状态只能是0或1。

- **字节：**8个二进制位构成1个"字节(Byte)"，它是存储空间的基本计量单位。1个字节可以储存1个英文字母或者半个汉字，换句话说，1个汉字占据2个字节的存储空间。

- **字：**"字"由若干个字节构成，字的位数叫做字长，不同档次的机器有不同的字长。例如一台8位机，它的1个字就等于1个字节，字长为8位。如果是一台16位机，那么，它的1个字就由2个字节构成，字长为16位。字是计算机进行数据处理和运算的单位。
- 一般的计算机都已经到了64位机 也就是说 一个基本单位就是64位，也就是8字节了。这样再综合上面的分析就不难看出，结构体，共用体，位域的定义中，按顺序分配内存，下一个字段所占大小如果超出了上一个字段占的内存单元剩余部分，那么它会重新申请下一个内存单元，而上一个多出部分将空着。

关于3楼：

原文内容：

“”共用体所占内存**并非是**9个char，即9个字节，**而是double的两倍，即16个字节，**而且每次输出都是它前面离它最近的值“”

补充说明：

这是因为字节对齐



关于5楼：

原文内容：

**“字节：**8个二进制位构成1个"字节(Byte)"，它是存储空间的基本计量单位。1个字节可以储存1个英文字母或者半个汉字，换句话说，1个汉字占据2个字节的存储空间。”

补充说明：

一个汉字占几个字节的存储空间，要看汉字的编码格式；

前不久测出的一个汉字占3个字节的存储空间；还没确定编码格式是UTF-8还是GB213

# C 位域

如果程序的结构中包含多个开关量，只有 TRUE/FALSE 变量，如下：

```
struct
{
  unsigned int widthValidated;
  unsigned int heightValidated;
} status;
```

这种结构需要 8 字节的内存空间，但在实际上，在每个变量中，我们只存储 0 或 1。在这种情况下，C 语言提供了一种更好的利用内存空间的方式。如果您在结构内使用这样的变量，您可以定义变量的宽度来告诉编译器，您将只使用这些字节。例如，上面的结构可以重写成：

```
struct
{
  unsigned int widthValidated : 1;
  unsigned int heightValidated : 1;
} status;
```

现在，上面的结构中，status 变量将占用 4 个字节的内存空间，但是只有 2 位被用来存储值。如果您用了 32 个变量，每一个变量宽度为 1 位，那么 status 结构将使用 4 个字节，但只要您再多用一个变量，如果使用了 33 个变量，那么它将分配内存的下一段来存储第 33 个变量，这个时候就开始使用 8 个字节。让我们看看下面的实例来理解这个概念：

## 实例

\#include <stdio.h> #include <string.h>  /* 定义简单的结构 */ struct {  unsigned int widthValidated;  unsigned int heightValidated; } status1;  /* 定义位域结构 */ struct {  unsigned int widthValidated : 1;  unsigned int heightValidated : 1; } status2;  int main( ) {   printf( "Memory size occupied by status1 : %d\n", sizeof(status1));   printf( "Memory size occupied by status2 : %d\n", sizeof(status2));    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
Memory size occupied by status1 : 8
Memory size occupied by status2 : 4
```

## 位域声明

在结构内声明位域的形式如下：

```
struct
{
  type [member_name] : width ;
};
```

下面是有关位域中变量元素的描述：

| 元素        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| type        | 整数类型，决定了如何解释位域的值。类型可以是整型、有符号整型、无符号整型。 |
| member_name | 位域的名称。                                                 |
| width       | 位域中位的数量。宽度必须小于或等于指定类型的位宽度。         |

带有预定义宽度的变量被称为**位域**。位域可以存储多于 1 位的数，例如，需要一个变量来存储从 0 到 7 的值，您可以定义一个宽度为 3 位的位域，如下：

```
struct
{
  unsigned int age : 3;
} Age;
```

上面的结构定义指示 C 编译器，age 变量将只使用 3 位来存储这个值，如果您试图使用超过 3 位，则无法完成。让我们来看下面的实例：

## 实例

\#include <stdio.h> #include <string.h>  struct {  unsigned int age : 3; } Age;  int main( ) {   Age.age = 4;   printf( "Sizeof( Age ) : %d\n", sizeof(Age) );   printf( "Age.age : %d\n", Age.age );    Age.age = 7;   printf( "Age.age : %d\n", Age.age );    Age.age = 8; // 二进制表示为 1000 有四位，超出   printf( "Age.age : %d\n", Age.age );    return 0; }

当上面的代码被编译时，它会带有警告，当上面的代码被执行时，它会产生下列结果：

```
Sizeof( Age ) : 4
Age.age : 4
Age.age : 7
Age.age : 0
```

文中例子解析：

```
struct 
{
    unsigned int age : 3;
} Age;

/*age 变量将只使用 3 位来存储这个值，如果您试图使用超过 3 位，则无法完成*/
Age.age = 4;
printf("Sizeof( Age ) : %d\n", sizeof(Age));
printf("Age.age : %d\n", Age.age);

// 二进制表示为 111 有三位，达到最大值
Age.age = 7;
printf("Age.age : %d\n", Age.age);

// 二进制表示为 1000 有四位，超出
Age.age = 8;
printf("Age.age : %d\n", Age.age);
```

**如果超出范围，则直接丢掉了，存不进去。**

**结构体内存分配原则**

**原则一**：结构体中元素按照定义顺序存放到内存中，但并不是紧密排列。从结构体存储的首地址开始 ，每一个元素存入内存中时，它都会认为内存是以自己的宽度来划分空间的，因此元素存放的位置一定会在自己大小的整数倍上开始。

**原则二**： 在原则一的基础上，检查计算出的存储单元是否为所有元素中最宽的元素长度的整数倍。若是，则结束；否则，将其补齐为它的整数倍。

测试实例：

```
#include <stdio.h>

typedef struct t1{
    char x;
    int y;
    double z;
}T1;

typedef struct t2{
    char x;
    double z;
    int y;
}T2;

int main(int argc, char* argv[])
{
    printf("sizeof(T1) = %lu\n", sizeof(T1));
    printf("sizeof(T2) = %lu\n", sizeof(T2));

    return 0;
}
```

输出：

```
sizeof(T1) = 16
sizeof(T2) = 24
```

**解析**

```
sizeof(T1.x) = sizeof(T2.x) = 1; 
sizeof(T1.y) = sizeof(T2.y) = 4; 
sizeof(T1.z) = sizeof(T2.z) = 8;
```

**T1**: 若从第 0 个字节开始分配内存，则 T1.x 存入第 0 字节，T1.y 占 4 个字节，由于第一的 4 字节已有数据，所以 T1.y 存入第 4-7 个字节，T1.z 占 8 个字节，由于第一个 8 字节已有数据，所以 T1.z 存入 8-15 个字节。共占有 16 个字节。

**T2**: 若从第 0 个字节开始分配内存，则 T1.x 存入第 0 字节，T1.z 占 8 个字节，由于第一的 8 字节已有数据，所以 T1.z 存入第 8-15 个字节，T1.y 占 4 个字节，由于前四个 4 字节已有数据，所以 T1.z 存入 16-19 个字节。共占有 20 个字节。此时所占字节不是最宽元素（double 长度为 8）的整数倍，因此将其补齐到 8 的整数倍，最终结果为 24。

**补充楼上 \**@karma\**，位域的内存大小测试**

```
// 位域内存测试
#include <stdio.h>
struct ONE_BYTE
{
    unsigned char _bool : 1;
    unsigned char del_flag : 1;
    unsigned char status : 4;
} one_byte;

struct TWO_BYTE
{
    unsigned char ccc1 : 4;
    unsigned char ccc2 : 4;
    unsigned char ccc3 : 4;
    unsigned char ccc4 : 4;
} two_byte;

struct THREE_BYTE
{
    unsigned char ccc1 : 4;
    unsigned char ccc2 : 4;
    unsigned char ccc3 : 4;
    unsigned char ccc4 : 4;
    unsigned char ccc5 : 4;
} three_byte;

struct FOUR_BYTE
{
    unsigned int ccc1 : 16;
    unsigned int ccc2 : 16;
} four_byte;


struct EIGHT_BYTE
{
    unsigned char ccc1 : 1;
    unsigned int ccc2 : 1;
} eight_byte;

int main(int argc, char const *argv[])
{
    printf("sizeof one_byte is : %lu\n", sizeof(one_byte));
    printf("sizeof two_byte is : %lu\n", sizeof(two_byte));
    printf("sizeof three_byte is : %lu\n", sizeof(three_byte));
    printf("sizeof four_byte is : %lu\n", sizeof(four_byte));
    printf("sizeof eight_byte is : %lu\n", sizeof(eight_byte));
    return 0;
}
```

输出结果为：

```
sizeof one_byte is : 1B
sizeof two_byte is : 2B
sizeof three_byte is : 3B
sizeof four_byte is : 4B
sizeof eight_byte is : 4B
```

由输出，可以验证以下结论：

（1）结构体内存分配原则：

- 原则一：结构体中元素按照定义顺序存放到内存中，但并不是紧密排列。从结构体存储的首地址开始 ，每一个元素存入内存中时，它都会认为内存是以自己的宽度来划分空间的，因此元素存放的位置一定会在自己大小的整数倍上开始。
- 原则二： 在原则一的基础上，检查计算出的存储单元是否为所有元素中最宽的元素长度的整数倍。若是，则结束；否则，将其补齐为它的整数倍。

（2）定义位域时，各个成员的类型最好保持一致，比如都用char，或都用int，不要混合使用，这样才能达到节省内存空间的目的。

```
// 二进制表示为 1000 有四位，超出
Age.age = 8;
printf("Age.age : %d\n", Age.age);
```

超出范围并不是直接丢弃，而是保留对应的 3 位的值。

比如 8 是 00001000，按照位域，对应 3 位的值是 000，所以打印结果是 0；

但是 9 是 00001001，按照位域，对应 3 位的值是 001，所以打印结果是 1；

同理 10 是 00001010,按照位域，对应 3 位的值是 010，所以打印结果是 2；

```
Age.age = 9; // 二进制表示为 1001 有四位，超出
printf( "Age.age : %d\n", Age.age );
   
Age.age = 10; // 二进制表示为 1010 有四位，超出
printf( "Age.age : %d\n", Age.age );
// 以下是 4,7,8,9,10 的打印结果
Sizeof( Age ) : 4
Age.age : 4
Age.age : 7
Age.age : 0
Age.age : 1
Age.age : 2
```

# C typedef

C 语言提供了 **typedef** 关键字，您可以使用它来为类型取一个新的名字。下面的实例为单字节数字定义了一个术语 **BYTE**：

```
typedef unsigned char BYTE;
```

在这个类型定义之后，标识符 BYTE 可作为类型 **unsigned char** 的缩写，例如：

```
BYTE  b1, b2;
```

按照惯例，定义时会大写字母，以便提醒用户类型名称是一个象征性的缩写，但您也可以使用小写字母，如下：

```
typedef unsigned char byte;
```

您也可以使用 **typedef** 来为用户自定义的数据类型取一个新的名字。例如，您可以对结构体使用 typedef 来定义一个新的数据类型名字，然后使用这个新的数据类型来直接定义结构变量，如下：

## 实例

\#include <stdio.h> #include <string.h>  typedef struct Books {   char  title[50];   char  author[50];   char  subject[100];   int   book_id; } Book;  int main( ) {   Book book;    strcpy( book.title, "C 教程");   strcpy( book.author, "Runoob");    strcpy( book.subject, "编程语言");   book.book_id = 12345;    printf( "书标题 : %s\n", book.title);   printf( "书作者 : %s\n", book.author);   printf( "书类目 : %s\n", book.subject);   printf( "书 ID : %d\n", book.book_id);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
书标题 : C 教程
书作者 : Runoob
书类目 : 编程语言
书 ID : 12345
```

## typedef vs #define

**#define** 是 C 指令，用于为各种数据类型定义别名，与 **typedef** 类似，但是它们有以下几点不同：

- **typedef** 仅限于为类型定义符号名称，**#define** 不仅可以为类型定义别名，也能为数值定义别名，比如您可以定义 1 为 ONE。
- **typedef** 是由编译器执行解释的，**#define** 语句是由预编译器进行处理的。

下面是 #define 的最简单的用法：

## 实例

\#include <stdio.h>  #define TRUE  1 #define FALSE 0  int main( ) {   printf( "TRUE 的值: %d\n", TRUE);   printf( "FALSE 的值: %d\n", FALSE);    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
TRUE 的值: 1
FALSE 的值: 0
```

**typedef 与 #define 的区别**

（1）#define可以使用其他类型说明符对宏类型名进行扩展，但对 typedef 所定义的类型名却不能这样做。例如：

```
#define INTERGE int;
unsigned INTERGE n;  //没问题
typedef int INTERGE;
unsigned INTERGE n;  //错误，不能在 INTERGE 前面添加 unsigned
```

（2） 在连续定义几个变量的时候，typedef 能够保证定义的所有变量均为同一类型，而 #define 则无法保证。例如：

```
#define PTR_INT int *
PTR_INT p1, p2;        //p1、p2 类型不相同，宏展开后变为int *p1, p2;
typedef int * PTR_INT
PTR_INT p1, p2;        //p1、p2 类型相同，它们都是指向 int 类型的指针。
```

**typedef 与 #define 比较**

typdef 的一些特性与 define 的功能重合。例如：

```
#define BYTE unsigned char
```

这是预处理器用 BYTE 替换 unsigned char。

但也有 #define 没有的功能，例如：

```
typedef char * STRING;
```

编译器把 STRING 解释为一个类型的表示符，该类型指向 char。因此：

```
STRING name, sign;
```

相当于：

```
char * name , * sign;  
```

但是，如果这样假设：

```
#define STRING char *
```

然后，下面的声明：

```
STRING name, sign;
```

将被翻译成：

```
char * name, sign;
```

这导致 name 才是指针。

简而言之，**#define** 只是字面上的替换，由预处理器执行，**#define A B** 相当于打开编辑器的替换功能，把所有的 B 替换成 A。

与 #define 不同，typedef 具有以下三个特点：

-  1.typedef 给出的符号名称仅限于对类型，而不是对值。
-  2.typedef 的解释由编译器，而不是预处理器执行。并不是简单的文本替换。
-  3.虽然范围有限，但是在其受限范围内 typedef 比 #define 灵活。

用 **typedef** 为数组去别名：

```
typedef int A[6];
```

表示用 **A** 代替 **int [6]**。

即：**A a;** 等于 **int a[6];**

**typedef 还有一个作用，就是为复杂的声明定义一个新的简单的别名。用在回调函数中特别好用：**

\1. 原声明：**int \*(\*a[5])(int, char\*);**

在这里，变量名为 **a**，直接用一个新别名 **pFun** 替换 **a** 就可以了：

```
typedef int *(*pFun)(int, char*);
```

于是，原声明的最简化版：

```
pFun a[5];
```

\2. 原声明：**void (\*b[10]) (void (\*)());**

这里，变量名为 b，先替换右边部分括号里的，pFunParam 为别名一：

```
typedef void (*pFunParam)();
```

再替换左边的变量 **b**，**pFunx** 为别名二：

```
typedef void (*pFunx)(pFunParam);
```

于是，原声明的最简化版：

```
pFunx b[10];
```

其实，可以这样理解:

```
typedef int *(*pFun)(int, char*); 
```

由 **typedef** 定义的函数 **pFun**，为一个新的类型，所以这个新的类型可以像 **int** 一样定义变量，于是，**pFun a[5];** 就定义了 **int \*(\*a[5])(int, char\*);**

所以我们可以用来定义回调函数，特别好用。

另外，也要注意，typedef 在语法上是一个存储类的关键字（如 auto、extern、mutable、static、register 等一样），虽然它并不真正影响对象的存储特性，如：

```
typedef static int INT2; // 不可行
```

编译将失败，会提示“指定了一个以上的存储类”。

# C 输入 & 输出

当我们提到**输入**时，这意味着要向程序填充一些数据。输入可以是以文件的形式或从命令行中进行。C 语言提供了一系列内置的函数来读取给定的输入，并根据需要填充到程序中。

当我们提到**输出**时，这意味着要在屏幕上、打印机上或任意文件中显示一些数据。C 语言提供了一系列内置的函数来输出数据到计算机屏幕上和保存数据到文本文件或二进制文件中。

## 标准文件

C 语言把所有的设备都当作文件。所以设备（比如显示器）被处理的方式与文件相同。以下三个文件会在程序执行时自动打开，以便访问键盘和屏幕。

| 标准文件 | 文件指针 | 设备     |
| :------- | :------- | :------- |
| 标准输入 | stdin    | 键盘     |
| 标准输出 | stdout   | 屏幕     |
| 标准错误 | stderr   | 您的屏幕 |

文件指针是访问文件的方式，本节将讲解如何从屏幕读取值以及如何把结果输出到屏幕上。

C 语言中的 I/O (输入/输出) 通常使用 printf() 和 scanf() 两个函数。

scanf() 函数用于从标准输入（键盘）读取并格式化， printf() 函数发送格式化输出到标准输出（屏幕）。

## 实例

\#include <stdio.h>      // 执行 printf() 函数需要该库 int main() {    printf("菜鸟教程");  //显示引号中的内容    return 0; }

编译以上程序，输出结果为：

```
菜鸟教程
```

**实例解析：**

- 所有的 C 语言程序都需要包含 **main()** 函数。 代码从 **main()** 函数开始执行。
- **printf()** 用于格式化输出到屏幕。**printf()** 函数在 **"stdio.h"** 头文件中声明。
- **stdio.h** 是一个头文件 (标准输入输出头文件) and **#include** 是一个预处理命令，用来引入头文件。 当编译器遇到 **printf()** 函数时，如果没有找到 **stdio.h** 头文件，会发生编译错误。
- **return 0;** 语句用于表示退出程序。

## %d 格式化输出整数

\#include <stdio.h> int main() {    int testInteger = 5;    printf("Number = %d", testInteger);    return 0; }

编译以上程序，输出结果为：

```
Number = 5
```

在 printf() 函数的引号中使用 "%d" (整型) 来匹配整型变量 testInteger 并输出到屏幕。

## %f 格式化输出浮点型数据

\#include <stdio.h> int main() {    float f;    printf("Enter a number: ");    // %f 匹配浮点型数据    scanf("%f",&f);    printf("Value = %f", f);    return 0; }

## getchar() & putchar() 函数

**int getchar(void)** 函数从屏幕读取下一个可用的字符，并把它返回为一个整数。这个函数在同一个时间内只会读取一个单一的字符。您可以在循环内使用这个方法，以便从屏幕上读取多个字符。

**int putchar(int c)** 函数把字符输出到屏幕上，并返回相同的字符。这个函数在同一个时间内只会输出一个单一的字符。您可以在循环内使用这个方法，以便在屏幕上输出多个字符。

请看下面的实例：

## 实例

\#include <stdio.h>  int main( ) {   int c;    printf( "Enter a value :");   c = getchar( );    printf( "\nYou entered: ");   putchar( c );   printf( "\n");   return 0; }

当上面的代码被编译和执行时，它会等待您输入一些文本，当您输入一个文本并按下回车键时，程序会继续并只会读取一个单一的字符，显示如下：

```
$./a.out
Enter a value :runoob

You entered: r
```

## gets() & puts() 函数

**char \*gets(char \*s)** 函数从 **stdin** 读取一行到 **s** 所指向的缓冲区，直到一个终止符或 EOF。

**int puts(const char \*s)** 函数把字符串 s 和一个尾随的换行符写入到 **stdout**。

## 实例

\#include <stdio.h>  int main( ) {   char str[100];    printf( "Enter a value :");   gets( str );    printf( "\nYou entered: ");   puts( str );   return 0; }

当上面的代码被编译和执行时，它会等待您输入一些文本，当您输入一个文本并按下回车键时，程序会继续并读取一整行直到该行结束，显示如下：

```
$./a.out
Enter a value :runoob

You entered: runoob
```

## scanf() 和 printf() 函数

**int scanf(const char \*format, ...)** 函数从标准输入流 **stdin** 读取输入，并根据提供的 **format** 来浏览输入。

**int printf(const char \*format, ...)** 函数把输出写入到标准输出流 **stdout** ，并根据提供的格式产生输出。

**format** 可以是一个简单的常量字符串，但是您可以分别指定 %s、%d、%c、%f 等来输出或读取字符串、整数、字符或浮点数。还有许多其他可用的格式选项，可以根据需要使用。如需了解完整的细节，可以查看这些函数的参考手册。现在让我们通过下面这个简单的实例来加深理解：

## 实例

\#include <stdio.h> int main( ) {    char str[100];   int i;    printf( "Enter a value :");   scanf("%s %d", str, &i);    printf( "\nYou entered: %s %d ", str, i);   printf("\n");   return 0; }

当上面的代码被编译和执行时，它会等待您输入一些文本，当您输入一个文本并按下回车键时，程序会继续并读取输入，显示如下：

```
$./a.out
Enter a value :runoob 123

You entered: runoob 123 
```

在这里，应当指出的是，scanf() 期待输入的格式与您给出的 %s 和 %d 相同，这意味着您必须提供有效的输入，比如 "string integer"，如果您提供的是 "string string" 或 "integer integer"，它会被认为是错误的输入。另外，在读取字符串时，只要遇到一个空格，scanf() 就会停止读取，所以 "this is test" 对 scanf() 来说是三个字符串。

linux系统下需要这样编译：不支持 gets 与 puts, 需要用 fgets 和 fputs。

将以下代码放到 test.c 文件：

```
#include <stdio.h>

int main()
{
    char c[100];
    printf("Enter a value:");
    fgets( c,100,stdin );

    printf("\nyou entered:");
    fputs( c,stdout );

    return 0;

}
```

编译执行以上代码，输出结果为：

```
# gcc test.c 
# ./a.out 
Enter a value:runoob

you entered:runoob
```

## gets()与fgets()

### gets()

gets函数原型：char*gets(char*buffer);//读取字符到数组：gets(str);str为数组名。

gets函数功能：从键盘上输入字符，直至接受到换行符或EOF时停止，并将读取的结果存放在buffer指针所指向的字符数组中。

读取的换行符被转换为null值，做为字符数组的最后一个字符，来结束字符串。

**注意：**gets函数由于没有指定输入字符大小，所以会无限读取，一旦输入的字符大于数组长度，就会发生内存越界，

从而造成程序崩溃或其他数据的错误。

### fgets()

fgets函数原型：char *fgets(char *s, int n, FILE *stream);//我们平时可以这么使用：fgets(str, sizeof(str), stdin);

其中str为数组首地址，sizeof(str)为数组大小，stdin表示我们从键盘输入数据。

fgets函数功能：从文件指针stream中读取字符，存到以s为起始地址的空间里，直到读完N-1个字符，或者读完一行。

**注意：**调用fgets函数时，最多只能读入n-1个字符。读入结束后，系统将自动在最后加'\0'，并以str作为函数值返回。

借用教程实例，我把char str[100] 改为 char str[5]



```
#include <stdio.h>

int main( )
{
    char str[5];

    printf( "Enter a value :");
    gets( str );

    printf( "\nYou entered: ");
    puts( str );
    return 0;
}
```

如果输入123(长度小于5)结果为：

```
Enter a value :123

You entered: 123
```

如果输入123456789(长度大于5)结果为：

```
Enter a value :123456789

You entered: 123456789
```

虽然正常显示了，但是系统提示程序崩溃了

如果不能正确使用gets()函数，带来的危害是很大的，就如上面我们看到的，输入字符串的长度大于缓冲区长度时，并没有截断，原样输出了读入的字符串，造成程序崩溃。



考虑到程序安全性和健壮性，建议用fgets()来代替gets()。如：



```
#include <stdio.h>

int main( )
{
    char str[5];

    printf( "Enter a value :");
    fgets( str,5,stdin );      //fgets()函数;

    printf( "\nYou entered: ");
    puts( str );
    return 0;
}
```

## Windows、Unix、Mac不同操作系统的换行问题 回车符\r和换行符\n

**一、概念：**

**换行符‘\n’**和**回车符‘\r’**

（1）换行符就是另起一行  --- '**\n**' 10 换行（newline）

（2）回车符就是回到一行的开头 --- '**\r**' 13 回车（return）

所以我们平时编写文件的回车符应该确切来说叫做**回车换行符** 

**CR: 回车(Carriage Return) \rLF: 换行(Line Feed) \n**

**二、应用：**

（1）在微软的MS-DOS和Windows中，使用“回车CR('\r')”和“换行LF('\n')”两个字符作为换行符;

（2）Windows系统里面，每行结尾是 回车+换行(CR+LF)，即“\r\n”；

（3）Unix系统里，每行结尾只有 换行LF，即“\n”；

（4）Mac系统里，每行结尾是 回车CR 即'\r'。

Mac OS 9 以及之前的系统的换行符是 CR，从 Mac OS X （后来改名为“OS X”）开始的换行符是 LF即‘\n'，和Unix/Linux统一了。

**三、影响：**

（1）一个直接后果是，Unix/Mac系统下的文件在Windows里打开的话，所有文字会变成一行；

（2）而Windows里的文件在Unix/Mac下打开的话，在每行的结尾可能会多出一个^M符号。

（3）Linux保存的文件在windows上用记事本看的话会出现黑点。

**四、可以相互转换**：

在linux下，命令unix2dos 是把linux文件格式转换成windows文件格式，命令dos2unix 是把windows格式转换成linux文件格式。

在不同平台间使用FTP软件传送文件时, 在ascii文本模式传输模式下, 一些FTP客户端程序会自动对换行格式进行转换. 经过这种传输的文件字节数可能会发生变化.

 如果你不想ftp修改原文件, 可以使用bin模式(二进制模式)传输文本。

一个程序在windows上运行就生成CR/LF换行格式的文本文件，而在Linux上运行就生成LF格式换行的文本文件。

学 C 语言的时候，字符输入曾经困扰过我，例如这段代码：

```
int i;
char c;
scanf("%d%c", &i,&c);
```

这时候变量 c 中存储的往往不是你想输入的字符，而是一个空格，然后我们又会这样来写：

```
int i;
char c;
scanf("%d", &i);
scanf("%c", &c);
```

这时候，我们发现，根本没有输入字符C的机会，这是为什么？因为输入流是有缓冲区的，我们输入的字符存储在那，然后再赋值给我们的变量。我们可以这样改：

```
int i;
char c;
scanf("%d", &i);
while((c=getchar())==' ' || c=='\n');
c = getchar();
```

这个办法是一直读取，读到没有空格和换行就跳出循环，但是有一个更好的解决办法；

```
int i;
char c;
scanf("%d%[^' '^'\n']", &i, &c);
```

这是用正则表达来控制输入格式为非空格非换行。

在进行输出时，若要用到用来输出实数的 **f** 格式符（以小数形式输出），有以下几种用法：

**1、基本型，用 \**%f\****

不指定输出类型的长度，用系统根据情况决定，一般是实数中的整数部分全部输出，小数部分输出六位。例：

```
#include<stdio.h>
int main()
{
    double a=1.0;
    printf("%f\n",a/3);
    return 0;
}
```

运行结果：**0.333333**

**2、指定数据宽度和小数位数，用 \**%m.nf\****

例：将上个程序的双精度变量 a 输出 15 位小数，用 **%20.15f** 的格式声明，指定输出的数据占 20 列，其中包括 15 位小数。改动上面程序如下：

```
#include<stdio.h>
int main()
{
    double a=1.0;
    printf("%20.15f\n",a/3);
    return 0;
}
```

运行结果：  **0.333333333333333**

注意在 0 的前面有 3 个空格，且双精度数只保证 15 位有效数字的准确性。

**3、输出的数据相左对齐，用 \**%-m.nf\****

在 **m.n** 前加一个负号，其作用与 **%m.nf** 形式作用基本相同，但当数据长度不长过 **m** 时，数据向左靠，右端补空格。

有关 **putchar** 的一个程序。

先后输出BOY三个字符。

例子1：

```
#include<stdio.h>
int main()
{
    char a='B',b='O',c='Y';
    putchar(a);
    putchar(b);
    putchar(c);
    putchar('\n');
    return 0;
}
```

例子2：

```
#include<stdio.h>
int main()
{
    int a=66,b=79,c=89;
    putchar(a);
    putchar(b);
    putchar(c);
    putchar('\n');
    return 0;
}
```

两个运行结果相同，因为将一个字符赋给字符变量和将字符的 ASCII 代码赋给字符变量作用是完全相同的，但要注意其值必须在字符的 ASCII 代码范围内。

scanf 是有返回值的，例如：

```
#include <stdio.h>
#include <stdlib.h>
int main()
{
    int a;
    int b;
    int c;
    printf("请输入三个整数：");
    int x=scanf("%d%d%d",&a,&b,&c);
    printf("d%\n%d\n",a,x);
}
```

测试输出：

```
$ ./a.out 
请输入三个整数：1 2 3
1
3
$ ./a.out 
请输入三个整数：5 6 d
5
2
```

- 1、scanf() 函数有返回值且类型 int 型，当发生错误时立刻返回 EOF。
-  2、scanf() 函数返回的值为：正确按指定格式输入变量的个数；也即能正确接收到值的变量个数。

从上边的例子中可以得到验证，这里用变量 x 接收 scanf() 函数的返回值，并输出显示出来。

在输入时注意格式对应：

```
#include <stdio.h>
int main()
{
    int a;
    float x;
    char c1;
    scanf("a=%d",&a);
    scanf("x=%f",&x);
    scanf("c1=%c",&c1);
    printf("a=%d,x=%f,c1=%c",a,x,c1);
    return 0;
}
```

若在输入时用错空格键或者换行符，则会出现错误:

```
a=1 x=1.2 c1=3
```

上述输入只能输出 a=1 因为空格键取代了 x 的位置 输入完 x=1.2 后空格键有取代了应该输入 c1 的位置。

正确的输入应为:

```
a=1x=1.2c1=3
```

<

strong>gets() & puts() 函数

使用 gets() 会报警告：

```
warning: this program uses gets(), which is unsafe.
```

gets() 不安全是因为未指定缓冲区大小。可以使用 fgets();

```
char* fgets(char *buf, int bufsize, FILE *stream);
```

-  **buf**：字符型指针，指向用来存储所得数据的地址。
-  **bufsize**：整形数据，指明缓冲区的大小，拷贝到 buf 地址的最大字符数量。
-  **stream**：指明输入流的 FILE 对象的指针，stdin 可以作为参数，表示从标准输入读取。

返回值：成功，则函数返回 buf。

如果当尝试读取一个字符时遇到了文件结尾，则 eof 被置位(feof)，如果还没有成功读入任何一个字符就遇到了文件结尾，那么就会返回 null，buff 中的内容保持不变。如果读取错误发生，那么 error indicator(ferror) 被置位，还是返回 null。

# C 文件读写

上一章我们讲解了 C 语言处理的标准输入和输出设备。本章我们将介绍 C 程序员如何创建、打开、关闭文本文件或二进制文件。

一个文件，无论它是文本文件还是二进制文件，都是代表了一系列的字节。C 语言不仅提供了访问顶层的函数，也提供了底层（OS）调用来处理存储设备上的文件。本章将讲解文件管理的重要调用。

## 打开文件

您可以使用 **fopen( )** 函数来创建一个新的文件或者打开一个已有的文件，这个调用会初始化类型 **FILE** 的一个对象，类型 **FILE** 包含了所有用来控制流的必要的信息。下面是这个函数调用的原型：

```
FILE *fopen( const char * filename, const char * mode );
```

在这里，**filename** 是字符串，用来命名文件，访问模式 **mode** 的值可以是下列值中的一个：

| 模式 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| r    | 打开一个已有的文本文件，允许读取文件。                       |
| w    | 打开一个文本文件，允许写入文件。如果文件不存在，则会创建一个新文件。在这里，您的程序会从文件的开头写入内容。如果文件存在，则该会被截断为零长度，重新写入。 |
| a    | 打开一个文本文件，以追加模式写入文件。如果文件不存在，则会创建一个新文件。在这里，您的程序会在已有的文件内容中追加内容。 |
| r+   | 打开一个文本文件，允许读写文件。                             |
| w+   | 打开一个文本文件，允许读写文件。如果文件已存在，则文件会被截断为零长度，如果文件不存在，则会创建一个新文件。 |
| a+   | 打开一个文本文件，允许读写文件。如果文件不存在，则会创建一个新文件。读取会从文件的开头开始，写入则只能是追加模式。 |

如果处理的是二进制文件，则需使用下面的访问模式来取代上面的访问模式：

```
"rb", "wb", "ab", "rb+", "r+b", "wb+", "w+b", "ab+", "a+b"
```

## 关闭文件

为了关闭文件，请使用 fclose( ) 函数。函数的原型如下：

```
 int fclose( FILE *fp );
```

如果成功关闭文件，**fclose( )** 函数返回零，如果关闭文件时发生错误，函数返回 **EOF**。这个函数实际上，会清空缓冲区中的数据，关闭文件，并释放用于该文件的所有内存。EOF 是一个定义在头文件 **stdio.h** 中的常量。

C 标准库提供了各种函数来按字符或者以固定长度字符串的形式读写文件。

## 写入文件

下面是把字符写入到流中的最简单的函数：

```
int fputc( int c, FILE *fp );
```

函数 **fputc()** 把参数 c 的字符值写入到 fp 所指向的输出流中。如果写入成功，它会返回写入的字符，如果发生错误，则会返回 **EOF**。您可以使用下面的函数来把一个以 null 结尾的字符串写入到流中：

```
int fputs( const char *s, FILE *fp );
```

函数 **fputs()** 把字符串 **s** 写入到 fp 所指向的输出流中。如果写入成功，它会返回一个非负值，如果发生错误，则会返回 **EOF**。您也可以使用 **int fprintf(FILE \*fp,const char \*format, ...)** 函数来写把一个字符串写入到文件中。尝试下面的实例：

> **注意：**请确保您有可用的 **tmp** 目录，如果不存在该目录，则需要在您的计算机上先创建该目录。
>
> **/tmp** 一般是 Linux 系统上的临时目录，如果你在 Windows 系统上运行，则需要修改为本地环境中已存在的目录，例如: **C:\tmp**、**D:\tmp**等。

## 实例

\#include <stdio.h>  int main() {   FILE *fp = NULL;    fp = fopen("/tmp/test.txt", "w+");   fprintf(fp, "This is testing for fprintf...\n");   fputs("This is testing for fputs...\n", fp);   fclose(fp); }

当上面的代码被编译和执行时，它会在 /tmp 目录中创建一个新的文件 **test.txt**，并使用两个不同的函数写入两行。接下来让我们来读取这个文件。

## 读取文件

下面是从文件读取单个字符的最简单的函数：

```
int fgetc( FILE * fp );
```

**fgetc()** 函数从 fp 所指向的输入文件中读取一个字符。返回值是读取的字符，如果发生错误则返回 **EOF**。下面的函数允许您从流中读取一个字符串：

```
char *fgets( char *buf, int n, FILE *fp );
```

函数 **fgets()** 从 fp 所指向的输入流中读取 n - 1 个字符。它会把读取的字符串复制到缓冲区 **buf**，并在最后追加一个 **null** 字符来终止字符串。

如果这个函数在读取最后一个字符之前就遇到一个换行符 '\n' 或文件的末尾 EOF，则只会返回读取到的字符，包括换行符。您也可以使用 **int fscanf(FILE \*fp, const char \*format, ...)** 函数来从文件中读取字符串，但是在遇到第一个空格字符时，它会停止读取。

## 实例

\#include <stdio.h>  int main() {   FILE *fp = NULL;   char buff[255];    fp = fopen("/tmp/test.txt", "r");   fscanf(fp, "%s", buff);   printf("1: %s\n", buff );    fgets(buff, 255, (FILE*)fp);   printf("2: %s\n", buff );      fgets(buff, 255, (FILE*)fp);   printf("3: %s\n", buff );   fclose(fp);  }

当上面的代码被编译和执行时，它会读取上一部分创建的文件，产生下列结果：

```
1: This
2: is testing for fprintf...

3: This is testing for fputs...
```

首先，**fscanf()** 方法只读取了 **This**，因为它在后边遇到了一个空格。其次，调用 **fgets()** 读取剩余的部分，直到行尾。最后，调用 **fgets()** 完整地读取第二行。

## 二进制 I/O 函数

下面两个函数用于二进制输入和输出：

size_t fread(void *ptr, size_t size_of_elements,              size_t number_of_elements, FILE *a_file);               size_t fwrite(const void *ptr, size_t size_of_elements,              size_t number_of_elements, FILE *a_file);

这两个函数都是用于存储块的读写 - 通常是数组或结构体。

fseek 可以移动文件指针到指定位置读,或插入写:

```
int fseek(FILE *stream, long offset, int whence);
```

fseek 设置当前读写点到 offset 处, whence 可以是 SEEK_SET,SEEK_CUR,SEEK_END 这些值决定是从文件头、当前点和文件尾计算偏移量 offset。

你可以定义一个文件指针 **FILE \*fp**,当你打开一个文件时，文件指针指向开头，你要指到多少个字节，只要控制偏移量就好，例如, 相对当前位置往后移动一个字节：**fseek(fp,1,SEEK_CUR);** 中间的值就是偏移量。 如果你要往前移动一个字节，直接改为负值就可以：**fseek(fp,-1,SEEK_CUR)**。

执行以下实例前，确保当前目录下 **test.txt** 文件已创建：

```
#include <stdio.h>

int main(){   
    FILE *fp = NULL;
    fp = fopen("test.txt", "r+");  // 确保 test.txt 文件已创建
    fprintf(fp, "This is testing for fprintf...\n");   
    fseek(fp, 10, SEEK_SET);
    if (fputc(65,fp) == EOF) {
        printf("fputc fail");   
    }   
    fclose(fp);
}
```

执行结束后，打开 test.txt 文件：

```
This is teAting for fprintf...
```

**注意：** 只有用 **r+** 模式打开文件才能插入内容，**w** 或 **w+** 模式都会清空掉原来文件的内容再来写，**a** 或 **a+** 模式即总会在文件最尾添加内容，哪怕用 fseek() 移动了文件指针位置。

在 Visual Studio 2015 开发环境中运行下列代码, 会提示 fopen 函数不安全:

```
#include <stdio.h>
int main() {
    // 申明文件指针
    FILE *filePointer = NULL; 
    // 以追加模式打开文件 test.txt, 其中路径参数用正斜杠("/")
    filePointer = fopen("C:/Users/Administrator/Desktop/test.txt", "a+");
    // 在test.txt文件末尾追加写入
    fputs("The text was added to the file by executing these codes.", filepointer);
    // 关闭文件
    fclose(filepointer);
    return 0;
}
```

错误提示:

**错误 C4996 'fopen': This function or variable may be unsafe. Consider using fopen_s instead. To disable deprecation, use _CRT_SECURE_NO_WARNINGS. See online help for details.**

解决办法，在代码首行写上:

```
#define _CRT_SECURE_NO_WARNINGS
```

定义宏忽略警告, 即可忽略安全警告, 生成运行。

# C 预处理器

**C 预处理器**不是编译器的组成部分，但是它是编译过程中一个单独的步骤。简言之，C 预处理器只不过是一个文本替换工具而已，它们会指示编译器在实际编译之前完成所需的预处理。我们将把 C 预处理器（C Preprocessor）简写为 CPP。

所有的预处理器命令都是以井号（#）开头。它必须是第一个非空字符，为了增强可读性，预处理器指令应从第一列开始。下面列出了所有重要的预处理器指令：

| 指令     | 描述                                                        |
| :------- | :---------------------------------------------------------- |
| #define  | 定义宏                                                      |
| #include | 包含一个源代码文件                                          |
| #undef   | 取消已定义的宏                                              |
| #ifdef   | 如果宏已经定义，则返回真                                    |
| #ifndef  | 如果宏没有定义，则返回真                                    |
| #if      | 如果给定条件为真，则编译下面代码                            |
| #else    | #if 的替代方案                                              |
| #elif    | 如果前面的 #if 给定条件不为真，当前条件为真，则编译下面代码 |
| #endif   | 结束一个 #if……#else 条件编译块                              |
| #error   | 当遇到标准错误时，输出错误消息                              |
| #pragma  | 使用标准化方法，向编译器发布特殊的命令到编译器中            |



## 预处理器实例

分析下面的实例来理解不同的指令。

```
#define MAX_ARRAY_LENGTH 20
```

这个指令告诉 CPP 把所有的 MAX_ARRAY_LENGTH 替换为 20。使用 *#define* 定义常量来增强可读性。

```
#include <stdio.h>
#include "myheader.h"
```

这些指令告诉 CPP 从**系统库**中获取 stdio.h，并添加文本到当前的源文件中。下一行告诉 CPP 从本地目录中获取 **myheader.h**，并添加内容到当前的源文件中。

```
#undef  FILE_SIZE
#define FILE_SIZE 42
```

这个指令告诉 CPP 取消已定义的 FILE_SIZE，并定义它为 42。

```
#ifndef MESSAGE
   #define MESSAGE "You wish!"
#endif
```

这个指令告诉 CPP 只有当 MESSAGE 未定义时，才定义 MESSAGE。

```
#ifdef DEBUG
   /* Your debugging statements here */
#endif
```

这个指令告诉 CPP 如果定义了 DEBUG，则执行处理语句。在编译时，如果您向 gcc 编译器传递了 *-DDEBUG* 开关量，这个指令就非常有用。它定义了 DEBUG，您可以在编译期间随时开启或关闭调试。

## 预定义宏

ANSI C 定义了许多宏。在编程中您可以使用这些宏，但是不能直接修改这些预定义的宏。

| 宏       | 描述                                                |
| :------- | :-------------------------------------------------- |
| __DATE__ | 当前日期，一个以 "MMM DD YYYY" 格式表示的字符常量。 |
| __TIME__ | 当前时间，一个以 "HH:MM:SS" 格式表示的字符常量。    |
| __FILE__ | 这会包含当前文件名，一个字符串常量。                |
| __LINE__ | 这会包含当前行号，一个十进制常量。                  |
| __STDC__ | 当编译器以 ANSI 标准编译时，则定义为 1。            |

让我们来尝试下面的实例：

```
#include <stdio.h>

main()
{
   printf("File :%s\n", __FILE__ );
   printf("Date :%s\n", __DATE__ );
   printf("Time :%s\n", __TIME__ );
   printf("Line :%d\n", __LINE__ );
   printf("ANSI :%d\n", __STDC__ );

}
```

当上面的代码（在文件 **test.c** 中）被编译和执行时，它会产生下列结果：

```
File :test.c
Date :Jun 2 2012
Time :03:36:24
Line :8
ANSI :1
```

## 预处理器运算符

C 预处理器提供了下列的运算符来帮助您创建宏：

##### 宏延续运算符（\）

一个宏通常写在一个单行上。但是如果宏太长，一个单行容纳不下，则使用宏延续运算符（\）。例如：

```
#define  message_for(a, b)  \
    printf(#a " and " #b ": We love you!\n")
```

##### 字符串常量化运算符（#）

在宏定义中，当需要把一个宏的参数转换为字符串常量时，则使用字符串常量化运算符（#）。在宏中使用的该运算符有一个特定的参数或参数列表。例如：

```
#include <stdio.h>

#define  message_for(a, b)  \
    printf(#a " and " #b ": We love you!\n")

int main(void)
{
   message_for(Carole, Debra);
   return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
Carole and Debra: We love you!
```

##### 标记粘贴运算符（##）

宏定义内的标记粘贴运算符（##）会合并两个参数。它允许在宏定义中两个独立的标记被合并为一个标记。例如：

```
#include <stdio.h>

#define tokenpaster(n) printf ("token" #n " = %d", token##n)

int main(void)
{
   int token34 = 40;
   
   tokenpaster(34);
   return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
token34 = 40
```

这是怎么发生的，因为这个实例会从编译器产生下列的实际输出：

```
printf ("token34 = %d", token34);
```

这个实例演示了 token##n 会连接到 token34 中，在这里，我们使用了**字符串常量化运算符（#）**和**标记粘贴运算符（##）**。

##### defined() 运算符

预处理器 **defined** 运算符是用在常量表达式中的，用来确定一个标识符是否已经使用 #define 定义过。如果指定的标识符已定义，则值为真（非零）。如果指定的标识符未定义，则值为假（零）。下面的实例演示了 defined() 运算符的用法：

```
#include <stdio.h>

#if !defined (MESSAGE)
   #define MESSAGE "You wish!"
#endif

int main(void)
{
   printf("Here is the message: %s\n", MESSAGE);  
   return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
Here is the message: You wish!
```

## 参数化的宏

CPP 一个强大的功能是可以使用参数化的宏来模拟函数。例如，下面的代码是计算一个数的平方：

```
int square(int x) {
   return x * x;
}
```

我们可以使用宏重写上面的代码，如下：

```
#define square(x) ((x) * (x))
```

在使用带有参数的宏之前，必须使用 **#define** 指令定义。参数列表是括在圆括号内，且必须紧跟在宏名称的后边。宏名称和左圆括号之间不允许有空格。例如：

```
#include <stdio.h>

#define MAX(x,y) ((x) > (y) ? (x) : (y))

int main(void)
{
   printf("Max between 20 and 10 is %d\n", MAX(10, 20));  
   return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
Max between 20 and 10 is 20
```

使用#define含参时，参数括号很重要，如上例中省略括号会导致运算错误：

```
#include <stdio.h>

#define square(x) ((x) * (x))

#define square_1(x) (x * x)

int main(void)
{
   printf("square 5+4 is %d\n", square(5+4));  
   printf("square_1 5+4 is %d\n", square_1(5+4)); 
   return 0;
}
```

输出结果为：

```
square 5+4 is 81
square_1 5+4 is 29
```

原因:

```
square   等价于   （5+4）*（5+4）=81
square_1 等价于   5+4*5+4=29
```

用#define宏定义将a,b交换，不使用中间变量，两种方法实现swap(x,y);

```
#include <stdio.h>
#define MAX(x,y) ((x>y)?(x):(y))
#define SWAP1(x,y) {x=x+y;y=x-y;x=x-y;}
#define SWAP2(x,y) {x=x^y;y=x^y;x=x^y;}

int main()
{
    int a,b;
    scanf("%d %d",&a,&b);
    printf("Max number is:%d\n",MAX(a,b));
    printf("交换前：x=%d,y=%d\n",a,b);
    SWAP1(a,b);
    printf("交换后：x=%d,y=%d\n",a,b);
    SWAP2(a,b);
    printf("再次交换后：x=%d,y=%d\n",a,b);
    return 0;
}
```

输出结果为：

```
2 4
Max number is:4
交换前：x=2,y=4
交换后：x=4,y=2
再次交换后：x=2,y=4
```

 回复楼上，不建议使用异或运算实现 swap，具体可参看链接：[C 关于使用异或运算交换两数的值](https://www.runoob.com/w3cnote/c-swap-data.html) 

对于字符常量化宏一例中，可以将宏做成函数形式，如下更改，给 **printf****{}****;****message_for(Carole, Debra)****;**

```
#include <stdio.h>

#define  message_for(a, b)  \
    {printf(#a " and " #b ": We love you!\n");}

int main(void)
{
   message_for(Carole, Debra)
   return 0;
}
```

# C 头文件

头文件是扩展名为 **.h** 的文件，包含了 C 函数声明和宏定义，被多个源文件中引用共享。有两种类型的头文件：程序员编写的头文件和编译器自带的头文件。

在程序中要使用头文件，需要使用 C 预处理指令 **#include** 来引用它。前面我们已经看过 **stdio.h** 头文件，它是编译器自带的头文件。

引用头文件相当于复制头文件的内容，但是我们不会直接在源文件中复制头文件的内容，因为这么做很容易出错，特别在程序是由多个源文件组成的时候。

A simple practice in C 或 C++ 程序中，建议把所有的常量、宏、系统全局变量和函数原型写在头文件中，在需要的时候随时引用这些头文件。

## 引用头文件的语法

使用预处理指令 **#include** 可以引用用户和系统头文件。它的形式有以下两种：

```
#include <file>
```

这种形式用于引用系统头文件。它在系统目录的标准列表中搜索名为 file 的文件。在编译源代码时，您可以通过 -I 选项把目录前置在该列表前。

```
#include "file"
```

这种形式用于引用用户头文件。它在包含当前文件的目录中搜索名为 file 的文件。在编译源代码时，您可以通过 -I 选项把目录前置在该列表前。

## 引用头文件的操作

**#include** 指令会指示 C 预处理器浏览指定的文件作为输入。预处理器的输出包含了已经生成的输出，被引用文件生成的输出以及 **#include** 指令之后的文本输出。例如，如果您有一个头文件 header.h，如下：

```
char *test (void);
```

和一个使用了头文件的主程序 *program.c*，如下：

```
int x;
#include "header.h"

int main (void)
{
   puts (test ());
}
```

编译器会看到如下的代码信息：

```
int x;
char *test (void);

int main (void)
{
   puts (test ());
}
```

## 只引用一次头文件

如果一个头文件被引用两次，编译器会处理两次头文件的内容，这将产生错误。为了防止这种情况，标准的做法是把文件的整个内容放在条件编译语句中，如下：

```
#ifndef HEADER_FILE
#define HEADER_FILE

the entire header file file

#endif
```

这种结构就是通常所说的包装器 **#ifndef**。当再次引用头文件时，条件为假，因为 HEADER_FILE 已定义。此时，预处理器会跳过文件的整个内容，编译器会忽略它。

## 有条件引用

有时需要从多个不同的头文件中选择一个引用到程序中。例如，需要指定在不同的操作系统上使用的配置参数。您可以通过一系列条件来实现这点，如下：

```
#if SYSTEM_1
   # include "system_1.h"
#elif SYSTEM_2
   # include "system_2.h"
#elif SYSTEM_3
   ...
#endif
```

但是如果头文件比较多的时候，这么做是很不妥当的，预处理器使用宏来定义头文件的名称。这就是所谓的**有条件引用**。它不是用头文件的名称作为 **#include** 的直接参数，您只需要使用宏名称代替即可：

```
 #define SYSTEM_H "system_1.h"
 ...
 #include SYSTEM_H
```

SYSTEM_H 会扩展，预处理器会查找 system_1.h，就像 **#include** 最初编写的那样。SYSTEM_H 可通过 -D 选项被您的 Makefile 定义。

在有多个 **.h** 文件和多个 **.c** 文件的时候，往往我们会用一个 **global.h** 的头文件来包括所有的 **.h** 文件，然后在除 **global.h** 文件外的头文件中 包含 **global.h** 就可以实现所有头文件的包含，同时不会乱。方便在各个文件里面调用其他文件的函数或者变量。

```
#ifndef _GLOBAL_H
#define _GLOBAL_H
#include <fstream>
#include <iostream>
#include <math.h>
#include <Config.h>
```

**C 语言中 include <> 与include "" 的区别?**

**#include < >** 引用的是编译器的类库路径里面的头文件。

**#include " "** 引用的是你程序目录的相对路径中的头文件，如果在程序目录没有找到引用的头文件则到编译器的类库路径的目录下找该头文件。

# C 强制类型转换

强制类型转换是把变量从一种类型转换为另一种数据类型。例如，如果您想存储一个 long 类型的值到一个简单的整型中，您需要把 long 类型强制转换为 int 类型。您可以使用**强制类型转换运算符**来把值显式地从一种类型转换为另一种类型，如下所示：

```
(type_name) expression
```

请看下面的实例，使用强制类型转换运算符把一个整数变量除以另一个整数变量，得到一个浮点数：

## 实例

\#include <stdio.h>  int main() {   int sum = 17, count = 5;   double mean;    mean = (double) sum / count;   printf("Value of mean : %f\n", mean );  }

当上面的代码被编译和执行时，它会产生下列结果：

```
Value of mean : 3.400000
```

这里要注意的是强制类型转换运算符的优先级大于除法，因此 **sum** 的值首先被转换为 **double** 型，然后除以 count，得到一个类型为 double 的值。

类型转换可以是隐式的，由编译器自动执行，也可以是显式的，通过使用**强制类型转换运算符**来指定。在编程时，有需要类型转换的时候都用上强制类型转换运算符，是一种良好的编程习惯。

## 整数提升

整数提升是指把小于 **int** 或 **unsigned int** 的整数类型转换为 **int** 或 **unsigned int** 的过程。请看下面的实例，在 int 中添加一个字符：

## 实例

\#include <stdio.h>  int main() {   int  i = 17;   char c = 'c'; /* ascii 值是 99 */   int sum;    sum = i + c;   printf("Value of sum : %d\n", sum );  }

当上面的代码被编译和执行时，它会产生下列结果：

```
Value of sum : 116
```

在这里，sum 的值为 116，因为编译器进行了整数提升，在执行实际加法运算时，把 'c' 的值转换为对应的 ascii 值。

## 常用的算术转换

**常用的算术转换**是隐式地把值强制转换为相同的类型。编译器首先执行**整数提升**，如果操作数类型不同，则它们会被转换为下列层次中出现的最高层次的类型：

![Usual Arithmetic Conversion](https://www.runoob.com/wp-content/uploads/2014/08/usual_arithmetic_conversion.png)

## 实例

\#include <stdio.h>  int main() {   int  i = 17;   char c = 'c'; /* ascii 值是 99 */   float sum;    sum = i + c;   printf("Value of sum : %f\n", sum );  }

当上面的代码被编译和执行时，它会产生下列结果：

```
Value of sum : 116.000000
```

在这里，c 首先被转换为整数，但是由于最后的值是 float 型的，所以会应用常用的算术转换，编译器会把 i 和 c 转换为浮点型，并把它们相加得到一个浮点数。如果一个运算符两边的运算数类型不同，先要将其转换为相同的类型，即较低类型转换为较高类型，然后再参加运算，转换规则如下图所示。

![img](https://www.runoob.com/wp-content/uploads/2014/09/img1.png)

# C 错误处理

C 语言不提供对错误处理的直接支持，但是作为一种系统编程语言，它以返回值的形式允许您访问底层数据。在发生错误时，大多数的 C 或 UNIX 函数调用返回 1 或 NULL，同时会设置一个错误代码 **errno**，该错误代码是全局变量，表示在函数调用期间发生了错误。您可以在 errno.h 头文件中找到各种各样的错误代码。

所以，C 程序员可以通过检查返回值，然后根据返回值决定采取哪种适当的动作。开发人员应该在程序初始化时，把 errno 设置为 0，这是一种良好的编程习惯。0 值表示程序中没有错误。

## errno、perror() 和 strerror()

C 语言提供了 **perror()** 和 **strerror()** 函数来显示与 **errno** 相关的文本消息。

- **perror()** 函数显示您传给它的字符串，后跟一个冒号、一个空格和当前 errno 值的文本表示形式。
- **strerror()** 函数，返回一个指针，指针指向当前 errno 值的文本表示形式。

让我们来模拟一种错误情况，尝试打开一个不存在的文件。您可以使用多种方式来输出错误消息，在这里我们使用函数来演示用法。另外有一点需要注意，您应该使用 **stderr** 文件流来输出所有的错误。

## 实例

\#include <stdio.h> #include <errno.h> #include <string.h>  extern int errno ;  int main () {   FILE * pf;   int errnum;   pf = fopen ("unexist.txt", "rb");   if (pf == NULL)   {      errnum = errno;      fprintf(stderr, "错误号: %d\n", errno);      perror("通过 perror 输出错误");      fprintf(stderr, "打开文件错误: %s\n", strerror( errnum ));   }   else   {      fclose (pf);   }   return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
错误号: 2
通过 perror 输出错误: No such file or directory
打开文件错误: No such file or directory
```

## 被零除的错误

在进行除法运算时，如果不检查除数是否为零，则会导致一个运行时错误。

为了避免这种情况发生，下面的代码在进行除法运算前会先检查除数是否为零：

## 实例

\#include <stdio.h> #include <stdlib.h>  main() {   int dividend = 20;   int divisor = 0;   int quotient;    if( divisor == 0){      fprintf(stderr, "除数为 0 退出运行...\n");      exit(-1);   }   quotient = dividend / divisor;   fprintf(stderr, "quotient 变量的值为 : %d\n", quotient );    exit(0); }

当上面的代码被编译和执行时，它会产生下列结果：

```
除数为 0 退出运行...
```

## 程序退出状态

通常情况下，程序成功执行完一个操作正常退出的时候会带有值 EXIT_SUCCESS。在这里，EXIT_SUCCESS 是宏，它被定义为 0。

如果程序中存在一种错误情况，当您退出程序时，会带有状态值 EXIT_FAILURE，被定义为 -1。所以，上面的程序可以写成：

## 实例

\#include <stdio.h> #include <stdlib.h>  main() {   int dividend = 20;   int divisor = 5;   int quotient;    if( divisor == 0){      fprintf(stderr, "除数为 0 退出运行...\n");      exit(EXIT_FAILURE);   }   quotient = dividend / divisor;   fprintf(stderr, "quotient 变量的值为: %d\n", quotient );    exit(EXIT_SUCCESS); }

当上面的代码被编译和执行时，它会产生下列结果：

```
quotient 变量的值为 : 4
```

# C 递归

递归指的是在函数的定义中使用函数自身的方法。

> 举个例子：
> 从前有座山，山里有座庙，庙里有个老和尚，正在给小和尚讲故事呢！故事是什么呢？"从前有座山，山里有座庙，庙里有个老和尚，正在给小和尚讲故事呢！故事是什么呢？'从前有座山，山里有座庙，庙里有个老和尚，正在给小和尚讲故事呢！故事是什么呢？……'"

语法格式如下：

void recursion() {   statements;   ... ... ...   recursion(); /* 函数调用自身 */   ... ... ... }  int main() {   recursion(); }

流程图：

![img](https://www.runoob.com/wp-content/uploads/2014/09/recursion_0.png)

C 语言支持递归，即一个函数可以调用其自身。但在使用递归时，程序员需要注意定义一个从函数退出的条件，否则会进入死循环。

递归函数在解决许多数学问题上起了至关重要的作用，比如计算一个数的阶乘、生成斐波那契数列，等等。

## 数的阶乘

下面的实例使用递归函数计算一个给定的数的阶乘：

## 实例

\#include <stdio.h>  double factorial(unsigned int i) {   if(i <= 1)   {      return 1;   }   return i * factorial(i - 1); } int  main() {    int i = 15;    printf("%d 的阶乘为 %f\n", i, factorial(i));    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
15 的阶乘为 1307674368000.000000
```

## 斐波那契数列

下面的实例使用递归函数生成一个给定的数的斐波那契数列：

## 实例

\#include <stdio.h>  int fibonaci(int i) {   if(i == 0)   {      return 0;   }   if(i == 1)   {      return 1;   }   return fibonaci(i-1) + fibonaci(i-2); }  int  main() {    int i;    for (i = 0; i < 10; i++)    {       printf("%d\t\n", fibonaci(i));    }    return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```
0    
1    
1    
2    
3    
5    
8    
13    
21    
34
```

 递归是一个简洁的概念，同时也是一种很有用的手段。但是，使用递归是要付出代价的。与直接的语句(如while循环)相比，递归函数会耗费更多的运行时间，并且要占用大量的栈空间。递归函数每次调用自身时，都需要把它的状态存到栈中，以便在它调用完自身后，程序可以返回到它原来的状态。未经精心设计的递归函数总是会带来麻烦。 

采用递归方法来解决问题，必须符合以下三个条件：

1、可以把要解决的问题转化为一个新问题，而这个新的问题的解决方法仍与原来的解决方法相同，只是所处理的对象有规律地递增或递减。

说明：解决问题的方法相同，调用函数的参数每次不同（有规律的递增或递减），如果没有规律也就不能适用递归调用。

2、可以应用这个转化过程使问题得到解决。

说明：使用其他的办法比较麻烦或很难解决，而使用递归的方法可以很好地解决问题。

3、必定要有一个明确的结束递归的条件。

说明：一定要能够在适当的地方结束递归调用。不然可能导致系统崩溃。

# C 可变参数

有时，您可能会碰到这样的情况，您希望函数带有可变数量的参数，而不是预定义数量的参数。C 语言为这种情况提供了一个解决方案，它允许您定义一个函数，能根据具体的需求接受可变数量的参数。下面的实例演示了这种函数的定义。

int func(int, ... )  {   .   .   . }  int main() {   func(2, 2, 3);   func(3, 2, 3, 4); }

请注意，函数 **func()** 最后一个参数写成省略号，即三个点号（**...**），省略号之前的那个参数是 **int**，代表了要传递的可变参数的总数。为了使用这个功能，您需要使用 **stdarg.h** 头文件，该文件提供了实现可变参数功能的函数和宏。具体步骤如下：

- 定义一个函数，最后一个参数为省略号，省略号前面可以设置自定义参数。
- 在函数定义中创建一个 **va_list** 类型变量，该类型是在 stdarg.h 头文件中定义的。
- 使用 **int** 参数和 **va_start** 宏来初始化 **va_list** 变量为一个参数列表。宏 va_start 是在 stdarg.h 头文件中定义的。
- 使用 **va_arg** 宏和 **va_list** 变量来访问参数列表中的每个项。
- 使用宏 **va_end** 来清理赋予 **va_list** 变量的内存。

现在让我们按照上面的步骤，来编写一个带有可变数量参数的函数，并返回它们的平均值：

## 实例

\#include <stdio.h> #include <stdarg.h>  double average(int num,...) {     va_list valist;    double sum = 0.0;    int i;     /* 为 num 个参数初始化 valist */    va_start(valist, num);     /* 访问所有赋给 valist 的参数 */    for (i = 0; i < num; i++)    {       sum += va_arg(valist, int);    }    /* 清理为 valist 保留的内存 */    va_end(valist);     return sum/num; }  int main() {   printf("Average of 2, 3, 4, 5 = %f\n", average(4, 2,3,4,5));   printf("Average of 5, 10, 15 = %f\n", average(3, 5,10,15)); }

当上面的代码被编译和执行时，它会产生下列结果。应该指出的是，函数 **average()** 被调用两次，每次第一个参数都是表示被传的可变参数的总数。省略号被用来传递可变数量的参数。

```
Average of 2, 3, 4, 5 = 3.500000
Average of 5, 10, 15 = 10.000000
```

**一、可变参数**

```
#include <stdio.h>

void debug_arg(unsigned int num, ...) 
{
    unsigned int i = 0;
    unsigned int *addr = &num;
    for (i = 0; i <= num; i++) 
    {
        /* *(addr + i) 从左往右依次取出传递进来的参数,类似于出栈过程 */
        printf("i=%d,value=%d\r\n", i, *(addr + i));
    }
}
int main(void)
{
    debug_arg(3, 66, 88, 666);
    return 0;
}
```

可变参数的工作原理,以32位机为例:

-  1.函数参数的传递存储在栈中,从右至左压入栈中,压栈过程为递减。
-  2.参数的传递以4字节对齐,float/double这里不讨论。

```
// 64 位机器用 8 字节对齐, 32 位 4 位对齐
#ifdef X64
#defin t long long
#else
#define t int
#endif

//VA_LIST套宏中可以使用，用来改变INTSIZEOF中t的类型
//固定参数详见
void test(int a, double b, char* c)
{
    char *p = (char*)&a;
    //因为&a = void 类型 需要转换,void * =&a 不需要转换但是使用时要转换
    printf("%p %p %p\n", &a, &b, &c);
    //观察地址变化
    printf("%p %s",(p+8),*(char**)(p+8+8));//64位机器时加8内存大小8字节对齐
    return;
}

//可变参数实验
void test1(char* s,char *st,...)
{
    char *ppt =(char*)&s;
    //printf("%p %p %p %p,",ppt,&s,&st,(char*)ppt+8);
    printf("%p %p %p %p\n", ppt, &s, &st, ppt + 8);
    printf("%s\n", *(char**)(ppt+4));
    printf(" %d\n",*(int*)(ppt + 4+4));//当是X64就加8 X86就加4因为内存对齐规则
    return;
}

int main()
{
    char *p = "Hello world";
    test1("111","eee",45234,23);
    //test(2, 2.2, "Hello world");x
    void *s = &p;
    printf("%s", *(char**)s);
    return 0;
}
```

C 函数要在程序中用到以下这些宏:

```
void va_start( va_list arg_ptr, prev_param ); 
type va_arg( va_list arg_ptr, type ); 
void va_end( va_list arg_ptr );
```

**va_list**: 用来保存宏va_start、va_arg和va_end所需信息的一种类型。为了访问变长参数列表中的参数，必须声明 va_list 类型的一个对象，定义： **typedef char \* va_list;**

**va_start**: 访问变长参数列表中的参数之前使用的宏，它初始化用 va_list 声明的对象，初始化结果供宏 va_arg 和 va_end 使用；

**va_arg**: 展开成一个表达式的宏，该表达式具有变长参数列表中下一个参数的值和类型。每次调用 va_arg 都会修改用 va_list 声明的对象，从而使该对象指向参数列表中的下一个参数；

**va_end**: 该宏使程序能够从变长参数列表用宏 va_start 引用的函数中正常返回。

**va** 在这里是 variable-argument(可变参数) 的意思。

这些宏定义在 stdarg.h 中，所以用到可变参数的程序应该包含这个头文件。

下面我们写一个简单的可变参数的函数，改函数至少有一个整数参数，第二个参数也是整数，是可选的。函数只是打印这两个参数的值。

```
#include <stdio.h>;  
#include <string.h>;  
#include <stdarg.h>;  

/* ANSI标准形式的声明方式，括号内的省略号表示可选参数 */  
int demo(char *msg, ... )  
{  
    va_list argp;                    /* 定义保存函数参数的结构 */  
    int argno = 0;                    /* 纪录参数个数 */  
    char *para;                        /* 存放取出的字符串参数 */                                      
    va_start( argp, msg );          /* argp指向传入的第一个可选参数，      msg是最后一个确定的参数 */  
    
    while (1) 
    {  
        para = va_arg( argp, char *);                 /*      取出当前的参数，类型为char *. */  
        if ( strcmp( para, "/0") == 0 )  
                                                      /* 采用空串指示参数输入结束 */  
            break;  
        printf("Parameter #%d is: %s/n", argno, para);  
        argno++;  
    }  
    va_end( argp );                                   /* 将argp置为NULL */  
    return 0;  
}

int main( void )  
{  
    demo("DEMO", "This", "is", "a", "demo!" ,"333333", "/0");  
}    
```

从这个函数的实现可以看到,我们使用可变参数应该有以下步骤:

-  1)首先在函数里定义一个 va_list 型的变量,这里是 arg_ptr，这个变量是指向参数的指针。
-  2)然后用 va_start 宏初始化变量 arg_ptr，这个宏的第二个参数是第一个可变参数的前一个参数，是一个固定的参数。
-  3)然后用 va_arg 返回可变的参数，并赋值给整数 j。va_arg 的第二个参数是你要返回的参数的类型，这里是int型。
-  4)最后用 va_end 宏结束可变参数的获取。然后你就可以在函数里使用第二个参数了。如果函数有多个可变参数的，依次调用 va_arg 获取各个参数。

# C 内存管理

本章将讲解 C 中的动态内存管理。C 语言为内存的分配和管理提供了几个函数。这些函数可以在 **** 头文件中找到。

| 序号 | 函数和描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | **void \*calloc(int num, int size);** 在内存中动态地分配 num 个长度为 size 的连续空间，并将每一个字节都初始化为 0。所以它的结果是分配了 num*size 个字节长度的内存空间，并且每个字节的值都是0。 |
| 2    | **void free(void \*address);** 该函数释放 address 所指向的内存块,释放的是动态分配的内存空间。 |
| 3    | **void \*malloc(int num);** 在堆区分配一块指定大小的内存空间，用来存放数据。这块内存空间在函数执行完成后不会被初始化，它们的值是未知的。 |
| 4    | **void \*realloc(void \*address, int newsize);** 该函数重新分配内存，把内存扩展到 **newsize**。 |

**注意：**void * 类型表示未确定类型的指针。C、C++ 规定 void * 类型可以通过类型转换强制转换为任何其它类型的指针。



## 动态分配内存

编程时，如果您预先知道数组的大小，那么定义数组时就比较容易。例如，一个存储人名的数组，它最多容纳 100 个字符，所以您可以定义数组，如下所示：

```
char name[100];
```

但是，如果您预先不知道需要存储的文本长度，例如您向存储有关一个主题的详细描述。在这里，我们需要定义一个指针，该指针指向未定义所需内存大小的字符，后续再根据需求来分配内存，如下所示：

## 实例

\#include <stdio.h> #include <stdlib.h> #include <string.h>  int main() {   char name[100];   char *description;    strcpy(name, "Zara Ali");    /* 动态分配内存 */   description = (char *)malloc( 200 * sizeof(char) );   if( description == NULL )   {      fprintf(stderr, "Error - unable to allocate required memory\n");   }   else   {      strcpy( description, "Zara ali a DPS student in class 10th");   }   printf("Name = %s\n", name );   printf("Description: %s\n", description ); }

当上面的代码被编译和执行时，它会产生下列结果：

```
Name = Zara Ali
Description: Zara ali a DPS student in class 10th
```

上面的程序也可以使用 **calloc()** 来编写，只需要把 malloc 替换为 calloc 即可，如下所示：

```
calloc(200, sizeof(char));
```

当动态分配内存时，您有完全控制权，可以传递任何大小的值。而那些预先定义了大小的数组，一旦定义则无法改变大小。

## 重新调整内存的大小和释放内存

当程序退出时，操作系统会自动释放所有分配给程序的内存，但是，建议您在不需要内存时，都应该调用函数 **free()** 来释放内存。

或者，您可以通过调用函数 **realloc()** 来增加或减少已分配的内存块的大小。让我们使用 realloc() 和 free() 函数，再次查看上面的实例：

## 实例

\#include <stdio.h> #include <stdlib.h> #include <string.h>  int main() {   char name[100];   char *description;    strcpy(name, "Zara Ali");    /* 动态分配内存 */   description = (char *)malloc( 30 * sizeof(char) );   if( description == NULL )   {      fprintf(stderr, "Error - unable to allocate required memory\n");   }   else   {      strcpy( description, "Zara ali a DPS student.");   }   /* 假设您想要存储更大的描述信息 */   description = (char *) realloc( description, 100 * sizeof(char) );   if( description == NULL )   {      fprintf(stderr, "Error - unable to allocate required memory\n");   }   else   {      strcat( description, "She is in class 10th");   }      printf("Name = %s\n", name );   printf("Description: %s\n", description );    /* 使用 free() 函数释放内存 */   free(description); }

当上面的代码被编译和执行时，它会产生下列结果：

```
Name = Zara Ali
Description: Zara ali a DPS student.She is in class 10th
```

您可以尝试一下不重新分配额外的内存，strcat() 函数会生成一个错误，因为存储 description 时可用的内存不足。

对于 void 指针，GNU 认为 **void \*** 和 **char \*** 一样，所以以下写法是正确的:

```
description = malloc( 200 * sizeof(char) );
```

但按照 ANSI(American National Standards Institute) 标准，需要对 void 指针进行强制转换，如下:

```
description = (char *)malloc( 200 * sizeof(char) );
```

同时，按照 ANSI(American National Standards Institute) 标准，不能对 void 指针进行算法操作:

```
void * pvoid;
pvoid++; //ANSI：错误
pvoid += 1; //ANSI：错误
// ANSI标准之所以这样认定，是因为它坚持：进行算法操作的指针必须是确定知道其指向数据类型大小的。

int *pint;
pint++; //ANSI：正确
```

> 更多内容参考：[C 语言中 void* 详解及应用](https://www.runoob.com/w3cnote/c-void-intro.html)

对于我们手动分配的内存，在 C 语言中是不用强制转换类型的。

```
description = malloc( 200 * sizeof(char) ); // C 语言正确。
description = malloc( 200 * sizeof(char) ); // C++ 错误
```

但是 C++ 是强制要求的，不然会报错。

动态可变长的结构体：

```
typedef struct
{
  int id;
  char name[0];
}stu_t;
```

定义该结构体，只占用4字节的内存，name不占用内存。

```
stu_t *s = NULL;    //定义一个结构体指针
s = malloc(sizeof(*s) + 100);//sizeof(*s)获取的是4，但加上了100，4字节给id成员使用，100字节是属于name成员的
s->id = 1010;
strcpy(s->name,"hello");
```

注意：一个结构体中只能有一个可变长的成员，并且该成员必须是最后一个成员。

# C 命令行参数

执行程序时，可以从命令行传值给 C 程序。这些值被称为**命令行参数**，它们对程序很重要，特别是当您想从外部控制程序，而不是在代码内对这些值进行硬编码时，就显得尤为重要了。

命令行参数是使用 main() 函数参数来处理的，其中，**argc** 是指传入参数的个数，**argv[]** 是一个指针数组，指向传递给程序的每个参数。下面是一个简单的实例，检查命令行是否有提供参数，并根据参数执行相应的动作：

```
#include <stdio.h>

int main( int argc, char *argv[] )  
{
   if( argc == 2 )
   {
      printf("The argument supplied is %s\n", argv[1]);
   }
   else if( argc > 2 )
   {
      printf("Too many arguments supplied.\n");
   }
   else
   {
      printf("One argument expected.\n");
   }
}
```

使用一个参数，编译并执行上面的代码，它会产生下列结果：

```
$./a.out testing
The argument supplied is testing
```

使用两个参数，编译并执行上面的代码，它会产生下列结果：

```
$./a.out testing1 testing2
Too many arguments supplied.
```

不传任何参数，编译并执行上面的代码，它会产生下列结果：

```
$./a.out
One argument expected
```

应当指出的是，**argv[0]** 存储程序的名称，**argv[1]** 是一个指向第一个命令行参数的指针，*argv[n] 是最后一个参数。如果没有提供任何参数，argc 将为 1，否则，如果传递了一个参数，**argc** 将被设置为 2。

多个命令行参数之间用空格分隔，但是如果参数本身带有空格，那么传递参数的时候应把参数放置在双引号 "" 或单引号 '' 内部。让我们重新编写上面的实例，有一个空间，那么你可以通过这样的观点，把它们放在双引号或单引号""""。让我们重新编写上面的实例，向程序传递一个放置在双引号内部的命令行参数：

```
#include <stdio.h>

int main( int argc, char *argv[] )  
{
   printf("Program name %s\n", argv[0]);
 
   if( argc == 2 )
   {
      printf("The argument supplied is %s\n", argv[1]);
   }
   else if( argc > 2 )
   {
      printf("Too many arguments supplied.\n");
   }
   else
   {
      printf("One argument expected.\n");
   }
}
```

使用一个用空格分隔的简单参数，参数括在双引号中，编译并执行上面的代码，它会产生下列结果：

```
$./a.out "testing1 testing2"

Progranm name ./a.out
The argument supplied is testing1 testing2
```

main 的两个参数的参数名如下:

```
int main( int argc, char *argv[] )
```

并不一定这样写，只是约定俗成罢了。但是亦可以写成下面这样:

```
int main( int test_argc, char *test_argv[] )  
```

任意你喜欢的名字。

但是大部分人还是写成开头那样的，如下：

```
int main( int argc, char *argv[] )  
```

# C 排序算法

### 冒泡排序

冒泡排序（英语：Bubble Sort）是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序（如从大到小、首字母从A到Z）错误就把他们交换过来。

过程演示：

![img](https://www.runoob.com/wp-content/uploads/2018/09/Bubble_sort_animation.gif)

## 实例

\#include <stdio.h> void bubble_sort(int arr[], int len) {    int i, j, temp;    for (i = 0; i < len - 1; i++)        for (j = 0; j < len - 1 - i; j++)            if (arr[j] > arr[j + 1]) {                temp = arr[j];                arr[j] = arr[j + 1];                arr[j + 1] = temp;            } } int main() {    int arr[] = { 22, 34, 3, 32, 82, 55, 89, 50, 37, 5, 64, 35, 9, 70 };    int len = (int) sizeof(arr) / sizeof(*arr);    bubble_sort(arr, len);    int i;    for (i = 0; i < len; i++)        printf("%d ", arr[i]);    return 0; }

### 选择排序

选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

过程演示：

![img](https://www.runoob.com/wp-content/uploads/2018/09/Selection_sort_animation.gif)

![img](https://www.runoob.com/wp-content/uploads/2018/09/Selection-Sort-Animation.gif)

## 实例

void swap(int *a,int *b) //交換兩個變數 {    int temp = *a;    *a = *b;    *b = temp; } void selection_sort(int arr[], int len)  {    int i,j;     for (i = 0 ; i < len - 1 ; i++)     {        int min = i;        for (j = i + 1; j < len; j++)     //走訪未排序的元素            if (arr[j] < arr[min])    //找到目前最小值                min = j;    //紀錄最小值           swap(&arr[min], &arr[i]);    //做交換    } }

### 插入排序

插入排序（英语：Insertion Sort）是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通常采用in-place排序（即只需用到 {\displaystyle O(1)} {\displaystyle O(1)}的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后

挪位，为最新元素提供插入空间。



过程演示：

![img](https://www.runoob.com/wp-content/uploads/2018/09/Insertion_sort_animation.gif)

## 实例

void insertion_sort(int arr[], int len){    int i,j,temp;    for (i=1;i<len;i++){            temp = arr[i];            for (j=i;j>0 && arr[j-1]>temp;j--)                    arr[j] = arr[j-1];            arr[j] = temp;    } }

### 希尔排序

希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。希尔排序是非稳定排序算法。

希尔排序是基于插入排序的以下两点性质而提出改进方法的：

- 插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率
- 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位

过程演示：

![img](https://www.runoob.com/wp-content/uploads/2018/09/Sorting_shellsort_anim.gif)

## 实例

void shell_sort(int arr[], int len) {    int gap, i, j;    int temp;    for (gap = len >> 1; gap > 0; gap = gap >> 1)        for (i = gap; i < len; i++) {            temp = arr[i];            for (j = i - gap; j >= 0 && arr[j] > temp; j -= gap)                arr[j + gap] = arr[j];            arr[j + gap] = temp;        } }

### 归并排序

把数据分为两段，从两段中逐个选最小的元素移入新数据段的末尾。

可从上到下或从下到上进行。

过程演示：

![img](https://www.runoob.com/wp-content/uploads/2018/09/Merge_sort_animation2.gif)

![img](https://www.runoob.com/wp-content/uploads/2018/09/Merge-sort-example-300px.gif)

## 迭代法

int min(int x, int y) {    return x < y ? x : y; } void merge_sort(int arr[], int len) {    int* a = arr;    int* b = (int*) malloc(len * sizeof(int));    int seg, start;    for (seg = 1; seg < len; seg += seg) {        for (start = 0; start < len; start += seg + seg) {            int low = start, mid = min(start + seg, len), high = min(start + seg + seg, len);            int k = low;            int start1 = low, end1 = mid;            int start2 = mid, end2 = high;            while (start1 < end1 && start2 < end2)                b[k++] = a[start1] < a[start2] ? a[start1++] : a[start2++];            while (start1 < end1)                b[k++] = a[start1++];            while (start2 < end2)                b[k++] = a[start2++];        }        int* temp = a;        a = b;        b = temp;    }    if (a != arr) {        int i;        for (i = 0; i < len; i++)            b[i] = a[i];        b = a;    }    free(b); }

## 递归法

void merge_sort_recursive(int arr[], int reg[], int start, int end) {    if (start >= end)        return;    int len = end - start, mid = (len >> 1) + start;    int start1 = start, end1 = mid;    int start2 = mid + 1, end2 = end;    merge_sort_recursive(arr, reg, start1, end1);    merge_sort_recursive(arr, reg, start2, end2);    int k = start;    while (start1 <= end1 && start2 <= end2)        reg[k++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];    while (start1 <= end1)        reg[k++] = arr[start1++];    while (start2 <= end2)        reg[k++] = arr[start2++];    for (k = start; k <= end; k++)        arr[k] = reg[k]; } void merge_sort(int arr[], const int len) {    int reg[len];    merge_sort_recursive(arr, reg, 0, len - 1); }

### 快速排序

在区间中随机挑选一个元素作基准，将小于基准的元素放在基准之前，大于基准的元素放在基准之后，再分别对小数区与大数区进行排序。

过程演示：

![img](https://www.runoob.com/wp-content/uploads/2018/09/Sorting_quicksort_anim.gif)

## 迭代法

typedef struct _Range {    int start, end; } Range; Range new_Range(int s, int e) {    Range r;    r.start = s;    r.end = e;    return r; } void swap(int *x, int *y) {    int t = *x;    *x = *y;    *y = t; } void quick_sort(int arr[], const int len) {    if (len <= 0)        return; // 避免len等於負值時引發段錯誤（Segment Fault）    // r[]模擬列表,p為數量,r[p++]為push,r[--p]為pop且取得元素    Range r[len];    int p = 0;    r[p++] = new_Range(0, len - 1);    while (p) {        Range range = r[--p];        if (range.start >= range.end)            continue;        int mid = arr[(range.start + range.end) / 2]; // 選取中間點為基準點        int left = range.start, right = range.end;        do        {            while (arr[left] < mid) ++left;   // 檢測基準點左側是否符合要求            while (arr[right] > mid) --right; //檢測基準點右側是否符合要求             if (left <= right)            {                swap(&arr[left],&arr[right]);                left++;right--;               // 移動指針以繼續            }        } while (left <= right);         if (range.start < right) r[p++] = new_Range(range.start, right);        if (range.end > left) r[p++] = new_Range(left, range.end);    } }

## 递归法

void swap(int *x, int *y) {    int t = *x;    *x = *y;    *y = t; } void quick_sort_recursive(int arr[], int start, int end) {    if (start >= end)        return;    int mid = arr[end];    int left = start, right = end - 1;    while (left < right) {        while (arr[left] < mid && left < right)            left++;        while (arr[right] >= mid && left < right)            right--;        swap(&arr[left], &arr[right]);    }    if (arr[left] >= arr[end])        swap(&arr[left], &arr[end]);    else        left++;    if (left)        quick_sort_recursive(arr, start, left - 1);    quick_sort_recursive(arr, left + 1, end); } void quick_sort(int arr[], int len) {    quick_sort_recursive(arr, 0, len - 1); }

\1. 希尔排序缩小递增量必须是要互质的。

\2. 快速排序可以不用交换中间值。

以下代码仅供参考：

```
void Array_Map_Sort_Quickly_Extrem(int* Array, int start, int end)
{
    int i=start;
    int j=end;
    int Pivot = Array[end];
    if(start<end)
    {
        while(i<j)
        {
            while(i<j &&Array[i]<=Pivot) i++;//Note: i choose the end as parameter
            Array[j]=Array[i];
            while(i<j &&Array[j]>=Pivot) j--;
            Array[i]=Array[j];
        }
        Array[i]= Pivot;
    }
    else 
        return;
    Array_Map_Sort_Quickly_Extrem(Array,start,i-1);
    Array_Map_Sort_Quickly_Extrem(Array,i+1,end);
}
```

**有种排序叫做猴子排序（Bogo Monkey）：**

-  1、检查是否排好
-  2、打乱
-  3、检查是否排好
-  4、打乱
-  5、……

如果数据稍多的话，几乎是不可能排序好的。

排序代码：

```
#include <time.h>
#include <stdlib.h>
#include <stdbool.h>

void swap(int* x, int* y){
  //交换
  int temporary = *x;
  *x = *y;
  *y = temporary;
}
void randomize(int arr[], int length){
  //打乱数组
  for(int i = 0; i < length; i++){
    srand(time(NULL)+i);//引入i增加随机性
    if(rand()%2) swap(&arr[i],&arr[i+1]);
  }
  //printf("!");//记录打乱次数
}
bool isSorted(int arr[], int length){
  for(int i = 0; i < length; i++) if(arr[i]>=arr[i+1]) return false;
  return true;
}
void bogoSort(int array[], int length){
  while(!isSorted(array,length)) randomize(array,length);
}
```

Demo：

```
#include <stdio.h>
int main(){
  int numbers[] = {20,9,233,0,-23,7,1,666,4,345,63,45,2,45};
  bogoSort(numbers,14);//也可以改成更小
  for(int i = 0; i < 14; i++) printf("%d,",numbers[i]);
}
```







***\*So you think you know C?\****

A lot of programmers claim they know C. Well, it has the most famous syntax, it has been there for 44 years, and it’s not cluttered with obscure features. It’s easy!

I mean, it’s easy to **claim** that you know C. You probably learned it in college or on the go, you probably had some experience with it, you probably think that you know it through and through because there’s not much to know. Well, there is. C is not that simple.

If you think it is — take this test. It only has 5 questions. Every question is basically the same: what the return value would be? And each question has a choice of four answers, of which one and only one is right.

***\*1\****

struct S {

 int i;

 char c;

} s;

 

int main(void) {

 return sizeof(*(&s));

}

  

窗体顶端

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps578A.tmp.png) A. 4
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps579A.tmp.png) B. 5
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps579B.tmp.png) C. 8
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps579C.tmp.png) D. I don't know.

窗体底端

***\*2\****

int main(void) {

 char a = 0;

 short int b = 0;

 return sizeof(b) == sizeof(a+b);

}

  

窗体顶端

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57AD.tmp.png) A. 0
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57AE.tmp.png) B. 1
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57AF.tmp.png) C. 2
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57B0.tmp.png) D. I don't know.

窗体底端

***\*3\****

int main(void) {

 char a = ' ' * 13;

 return a;

}

  

窗体顶端

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57C0.tmp.png) A. 416
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57C1.tmp.png) B. 160
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57C2.tmp.png) C. -96
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57D3.tmp.png) D. I don't know.

窗体底端

***\*4\****

int main(void) {

 int i = 16;

 return (((((i >= i) << i) >> i) <= i));

}

  

窗体顶端

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57D4.tmp.png) A. 0
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57D5.tmp.png) B. 1
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57D6.tmp.png) C. 16
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57E7.tmp.png) D. I don't know.

窗体底端

***\*5\****

int main(void) {

 int i = 0;

 return i++ + ++i;

}

  

窗体顶端

![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57E8.tmp.png) A. 1
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57E9.tmp.png) B. 2
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57F9.tmp.png) C. 3
![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57FA.tmp.png) D. I don't know.

窗体底端

That's it.

So, the right answers are:

|      | A    | B    | C    | D    |
| ---- | ---- | ---- | ---- | ---- |
| 1    |      |      |      | ✔    |
| 2    |      |      |      | ✔    |
| 3    |      |      |      | ✔    |
| 4    |      |      |      | ✔    |
| 5    |      |      |      | ✔    |

You scored 5 points, congratulations! You know exactly what you don't know.

And yes, the right answer to every question is “I don’t know”.

Let’s untangle them one by one now.

***\*First one\**** is actually about structure padding. C compiler knows that storing unaligned data in RAM may be costly, so it pads your data for you. If you have 5 bytes of data in a structure, it will probably make it 8. Or 16. Or 6. Or whatever it wants. There are extensions like GCC attributes aligned and packed that let you get some control over this process, but they are non-standard. C itself does not define padding attributes, so the right answer is: “I don’t know”.

The ***\*second one\**** is about integer promotion. It’s only reasonable that the type of short int and an expression with the largest integer being short int would be the same. But the reasonable doesn’t mean right for C. There is the rule that every integer expression gets promoted to int. Actually, it’s much more complicated than that. Take a peek at the standard, you’ll enjoy it.

But even so, we don’t compare types, we compare sizes. And the standard only guarantees that the range of short int should not exceed int. They ranges may be equal, and the size of the **short int** may even be grates than the size of **int** because of the padding bits. So the right answer is: “I don’t know”.

The ***\*third one\**** is all about dark corners. Starting from that neither integer overflows, nor char type sign are defined by the standard. First one is undefined behavior, the second is implementation specific. But even more, the size of the **char** type itself is not specified in bits either. There were platforms where it was 6 bits (remember trigraphs?), and there are platforms where all five integer types are 32 bits. Even the value of the space character is implementation defined. Without all these details specified, every speculation about the result is invalid, so the answer is: “I don’t know”.

The ***\*fourth one\**** looks tricky, but it’s not that hard in retrospective since you already know that int size is not directly specified in the standard. It can easily be 16 bits, then the very first operation will cause the over-shift and that’s plain undefined behavior. It’s not C fault, on some platforms, it is even undefined in assembly, so the compiler simply can’t give you valid guarantees without eating up a lot of performance.

So once again “I don’t know” is the right answer.

And the ***\*last one\**** is classic. The order of operand evaluation for +is not specified, and i++ and ++i alter their operand. It might work just like you expect on one platform and might fail easily on the other. Usually, it just evaluates to **2**, so you get used to it until one day it doesn't. That’s the problem with unspecified things. When you meet one, the right answer is always: “I don’t know”.

***\*P. S.\****

And at this point, I only have to apologize. The test is clearly provocative and may even be a little offensive. I’m sorry if it causes any aggravation.

The thing is, I learned C in roughly 1998, and for the whole 15 years thought that I’m good at it. It was my language of choice in college, and I’ve done some successful projects in C on my first full-time job, and even then, when I was mostly working with C++, I thought of it as over-bloated C.

The pivoting moment came in 2013, when I’ve got myself involved with some safety-critical PLC programming. It was a research project in nuclear power plant automation, where absolutely no underspecification was tolerable. I had to learn that, while I did know a lot about C programming, the absolute majority of what I knew was false. And I had to learn it the hard way too.

Eventually, I had to learn to rely on the standard instead of folklore; to trust measurements and not presumptions; to take “things that simply work” skeptically, — I had to learn an engineering attitude. This is what matters the most, not some particular WAT anecdotes.

I only hope this little test would help someone like myself from the past to learn this attitude in some 15 minutes and not 15 years.

| ![img](file:///C:\Users\DELL\AppData\Local\Temp\ksohtml\wps57FB.tmp.png) | ← there's more. | + [Github](https://github.com/akalenuk/wordsandbuttons) & [Twitter](https://twitter.com/wordsandbuttons) & [Hacker News](https://news.ycombinator.com/from?site=wordsandbuttons.online) |
| ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ |
|                                                              |                 |                                                              |

 