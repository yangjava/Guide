Markdown手册

[TOC]

## Markdown概述

### 概述

Markdown 语言在 2004 由约翰·格鲁伯（英语：John Gruber）创建。

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。 

Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。 

Markdown 编写的文档后缀为 .md 或者 .markdown。 

### 用途

Markdown 能被使用来撰写电子书，如：Gitbook。 

当前许多网站都广泛使用 Markdown 来撰写帮助文档或是用于论坛上发表消息。例如：GitHub、简书、reddit、

Diaspora、Stack Exchange、OpenStreetMap 、SourceForge等。 

## Markdown 常用语法

### 常用语法

最常见的Markdown格式选项和键盘快捷键:

| 输出后的效果 | Markdown         | 快捷键             |
| :----------- | :--------------- | :----------------- |
| Bold         | **text**         | Ctrl/⌘ + B         |
| *Emphasize*  | *text*           | Ctrl/⌘ + I         |
| Link         | [title](http://) | Ctrl/⌘ + K         |
| Inline Code  | `code`           | Ctrl/⌘ + Shift + K |
| Image        | ![alt](http://)  | Ctrl/⌘ + Shift + I |
| List         | * item           | Ctrl + L           |
| Blockquote   | > quote          | Ctrl + Q           |
| H1           | # Heading        |                    |
| H2           | ## Heading       | Ctrl/⌘ + H         |
| H3           | ### Heading      | Ctrl/⌘ + H (x2)    |

### Markdown 标题

标题能显示出文章的结构。

行首插入1-6个 # ，每增加一个 # 表示更深入层次的内容，对应到标题的深度由 1-6 阶。

#### `#`    标题

使用 # 号可表示 1-6 级标题，一级标题对应一个 # 号，二级标题对应两个 # 号，以此类推。

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

#### `=`    一级标题

```
我展示的是一级标题
=================
```

#### `-`    二级标题

```
我展示的是二级标题
-----------------
```

### Markdown 文本样式

#### 段落

Markdown 段落没有特殊的格式，直接编写文字就好，`段落的换行是使用两个以上空格加上回车`。 

当然也可以在段落后面使用一个空行来表示重新开始一个段落。 

#### 字体

Markdown 可以使用以下几种字体：

##### 斜体

##### `*`    斜体文本

```
*斜体文本*
```

*斜体文本*

##### `_`    斜体文本

```
_斜体文本_
```

_斜体文本_

##### 加粗 

##### `**`    粗体文本

```
**粗体文本**
```

**粗体文本**

##### `__`    粗体文本

```
__粗体文本__
```

__粗体文本__

##### 高亮

##### `==`   高亮文本

```
==高亮文本==  
```

==高亮文本== 

##### 粗斜体

##### `***`   粗斜体

```
***粗斜体文本***
```

***粗斜体文本***

##### `___`  粗斜体

```
___粗斜体文本___
```

___粗斜体文本___

#### 分隔线

你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。下面每种写法都可以建立分隔线：

##### `*` 分割线

```
*********
```

##### `-`  分割线

```
----------
```

#### 删除线

如果段落上的文字要添加删除线，只需要在文字的两端加上两个波浪线 ~~ 即可，实例如下：

##### ` ~~`删除线

```
~~删除线文本~~
```

~~删除线文本~~

#### 下划线

下划线可以通过 HTML 的 `<u> `标签来实现：

##### `u` 下划线

```
<u>下划线文本</u>
```

<u>下划线文本</u>

#### 脚注

脚注是对文本的补充说明。

##### `[^脚注]` 脚注

Markdown 脚注的格式如下:

```
[^要注明的文本]: 这是一段脚注
```

创建脚注 [^脚注]。

[^脚注]: 这是一段脚注！！！

## Markdown 列表

Markdown 支持有序列表和无序列表。

### 无序列表

无序列表使用星号(*)、加号(+)或是减号(-)作为列表标记：

#### `* `  无序列表

```
* 第一项
* 第二项
* 第三项
```

* 第一项
* 第二项
* 第三项

#### `+  `  无序列表

```
+ 第一项
+ 第二项
+ 第三项
```

+ 第一项
+ 第二项
+ 第三项

#### `_ `   无序列表

```
- 第一项
- 第二项
- 第三项
```

- 第一项
- 第二项
- 第三项

### 有序列表

有序列表使用数字并加上 . 号来表示

#### ` 数字.` 有序列表

```
1. 第一项
2. 第二项
3. 第三项
```

第一项

第二项

第三项

### 列表嵌套

列表嵌套只需在子列表中的选项添加四个空格即可：

#### `四个空格` 列表嵌套

```
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
```
第一项：
- 第一项嵌套的第一个元素
- 第一项嵌套的第二个元素

第二项：
- 第二项嵌套的第一个元素
- 第二项嵌套的第二个元素

## Markdown 区块

### 区块引用

Markdown 区块引用是在段落开头使用 > 符号 ，然后后面紧跟一个**空格**符号：

#### `> `    区块

```
> 区块引用
> 这是一个区块
> 这是一个区块
```
> 区块引用
> 这是一个区块
> 这是一个区块

另外区块是可以嵌套的，一个 > 符号是最外层，两个 > 符号是第一层嵌套，以此类推退：

#### `>>`  区块嵌套

```
> 最外层
> > 第一层嵌套
> > > 第二层嵌套
```

> 最外层
> > 第一层嵌套
> >
> > > 第二层嵌套                          

### 区块中使用列表

区块中使用列表实例如下：

#### `> 数字.` 区块列表

```
> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项
```

> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项

### 列表中使用区块

如果要在列表项目内放进区块，那么就需要在 > 前添加四个空格的缩进。

#### `四个空格>` 列表区块

区块中使用列表实例如下：

```
* 第一项
    > 这是一个区块
    > 这是一个区块
* 第二项
```

* 第一项
    > 这是一个区块
    > 这是一个区块

* 第二项

## Markdown 代码

### 单行代码

#### `` 单行代码

如果是段落上的一个函数或片段的代码可以用反引号把它包起来（`），例如：

```
`printf()` 函数
```

`printf()` 函数

### 代码区块

####   ```  代码区块

代码区块使用 **4 个空格**或者一个**制表符（Tab 键）**。

你也可以用 ``` 包裹一段代码，并指定一种语言（也可以不指定）：

  ```
​```javascript
$(document).ready(function () {
    alert('Hello World !');
});
​```
  ```

## Markdown 链接

### 链接

链接使用方法如下：

#### `[title](http://) ` 链接

```
[链接名称](链接地址)

这是一个链接 [百度](https://www.baidu.com)
```

这是一个链接 [百度](https://www.baidu.com)

#### `<http://>`  链接

直接使用链接地址： 

```
<链接地址>
<https://www.baidu.com>
```

### 高级链接

```
链接也可以用变量来代替，文档末尾附带变量地址：
这个链接用 1 作为网址变量 [百度][1]
这个链接用 runoob 作为网址变量 [百度][baidu]
然后在文档的结尾为变量赋值（网址）

  [1]: http://www.baidu.com/
  [baidu]: http://www.baidu.com/
```

显示结果如下：

链接也可以用变量来代替，文档末尾附带变量地址：
这个链接用 1 作为网址变量 [百度][1]
这个链接用 runoob 作为网址变量 [百度][baidu]
然后在文档的结尾为变量赋值（网址）

[1]: http://www.baidu.com/
[baidu]: http://www.baidu.com/

## Markdown 图片

### 图片

Markdown 图片语法格式如下：

` ![alt](http://)`  图片

```
![alt 属性文本](图片地址)

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png)
```

` ![alt](http:// "title")`  图片

```
![alt 属性文本](图片地址 "可选标题")

![RUNOOB 图标](http://static.runoob.com/images/runoob-logo.png "RUNOOB")
```

- 开头一个感叹号 !
- 接着一个方括号，里面放上图片的替代文字
- 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的 'title' 属性的文字。

当然，你也可以像网址那样对图片网址使用变量:

```
这个链接用 1 作为网址变量 [RUNOOB][1].
然后在文档的结尾位变量赋值（网址）

[1]: http://static.runoob.com/images/runoob-logo.png
```

Markdown 还没有办法指定图片的高度与宽度，如果你需要的话，你可以使用普通的 `<img>` 标签。

```
<img src="http://static.runoob.com/images/runoob-logo.png" width="50%">
```

## Markdown 表格

### 表格

Markdown 制作表格使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行。

语法格式如下：

```
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |
```

| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |
| 单元格 | 单元格 |

对齐方式

**我们可以设置表格的对齐方式：**

- -: 设置内容和标题栏居右对齐。
- :- 设置内容和标题栏居左对齐。
- :-: 设置内容和标题栏居中对齐。

实例如下：

```
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
```

## Markdown 高级

### 支持的 HTML 元素

不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。

目前支持的 HTML 元素有：`<kbd> <b> <i> <em> <sup> <sub> <br>`等 ，如：

```
使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑
```

### 转义

Markdown 使用了很多特殊符号来表示特定的意义，如果需要显示特定的符号则需要使用转义字符，Markdown 使用反斜杠转义特殊字符：

```
**文本加粗** 
\*\* 正常显示星号 \*\*
```

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号： 

```
\   反斜线
`   反引号
*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号
```

### 公式

当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。提交后，问答和文章页会根据需要加载 Mathjax 对数学公式进行渲染。如：

```
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$

大括号

```
方法一：

$$ f(x)=\left\{
\begin{aligned}
x & = & \cos(t) \\
y & = & \sin(t) \\
z & = & \frac xy
\end{aligned}
\right.
$$

方法二：
$$ F^{HLLC}=\left\{
\begin{array}{rcl}
F_L       &      & {0      <      S_L}\\
F^*_L     &      & {S_L \leq 0 < S_M}\\
F^*_R     &      & {S_M \leq 0 < S_R}\\
F_R       &      & {S_R \leq 0}
\end{array} \right. $$

方法三:
$$f(x)=
\begin{cases}
0& \text{x=0}\\
1& \text{x!=0}
\end{cases}$$
```

```
基本数据类型数值型字符型布尔型基本数据类型= \begin{cases} 数值型& \\ 字符型& \\ 布尔型  \end{cases}
```

$$基本数据类型= \begin{cases} 数值型& \\ 字符型& \\ 布尔型  \end{cases}$$

## Markdown图表

### 横向流程图

```
​```mermaid
graph LR
A[方形] -->B(圆角)
    B --> C{条件a}
    C -->|a=1| D[结果1]
    C -->|a=2| E[结果2]
    F[横向流程图]
​```
```

```mermaid
graph LR
A[方形] -->B(圆角)
    B --> C{条件a}
    C -->|a=1| D[结果1]
    C -->|a=2| E[结果2]
    F[横向流程图]
```
### 竖向流程图

```
​```mermaid
graph TD
A[方形] --> B(圆角)
    B --> C{条件a}
    C --> |a=1| D[结果1]
    C --> |a=2| E[结果2]
    F[竖向流程图]
​```
```



```mermaid
graph TD
A[方形] --> B(圆角)
    B --> C{条件a}
    C --> |a=1| D[结果1]
    C --> |a=2| E[结果2]
    F[竖向流程图]
```

### 竖向标准流程图

```
​```flow
st=>start: 开始框
op=>operation: 处理框
cond=>condition: 判断框(是或否?)
sub1=>subroutine: 子流程
io=>inputoutput: 输入输出框
e=>end: 结束框
st->op->cond
cond(yes)->io->e
cond(no)->sub1(right)->op
​```
```

```flow
st=>start: 开始框
op=>operation: 处理框
cond=>condition: 判断框(是或否?)
sub1=>subroutine: 子流程
io=>inputoutput: 输入输出框
e=>end: 结束框
st->op->cond
cond(yes)->io->e
cond(no)->sub1(right)->op
```
### 横向标准流程图

```
​```flow
st=>start: 开始框
op=>operation: 处理框
cond=>condition: 判断框(是或否?)
sub1=>subroutine: 子流程
io=>inputoutput: 输入输出框
e=>end: 结束框
st(right)->op(right)->cond
cond(yes)->io(bottom)->e
cond(no)->sub1(right)->op
​```
```

```flow
st=>start: 开始框
op=>operation: 处理框
cond=>condition: 判断框(是或否?)
sub1=>subroutine: 子流程
io=>inputoutput: 输入输出框
e=>end: 结束框
st(right)->op(right)->cond
cond(yes)->io(bottom)->e
cond(no)->sub1(right)->op
```
### UML时序图

```
​```sequence
对象A->对象B: 对象B你好吗?（请求）
Note right of 对象B: 对象B的描述
Note left of 对象A: 对象A的描述(提示)
对象B-->对象A: 我很好(响应)
对象A->对象B: 你真的好吗？
​```
```

```sequence
对象A->对象B: 对象B你好吗?（请求）
Note right of 对象B: 对象B的描述
Note left of 对象A: 对象A的描述(提示)
对象B-->对象A: 我很好(响应)
对象A->对象B: 你真的好吗？
```
### UML时序图复杂

```
​```sequence
Title: 标题：复杂使用
对象A->对象B: 对象B你好吗?（请求）
Note right of 对象B: 对象B的描述
Note left of 对象A: 对象A的描述(提示)
对象B-->对象A: 我很好(响应)
对象B->小三: 你好吗
小三-->>对象A: 对象B找我了
对象A->对象B: 你真的好吗？
Note over 小三,对象B: 我们是朋友
participant C
Note right of C: 没人陪我玩
​```
```

```sequence
Title: 标题：复杂使用
对象A->对象B: 对象B你好吗?（请求）
Note right of 对象B: 对象B的描述
Note left of 对象A: 对象A的描述(提示)
对象B-->对象A: 我很好(响应)
对象B->小三: 你好吗
小三-->>对象A: 对象B找我了
对象A->对象B: 你真的好吗？
Note over 小三,对象B: 我们是朋友
participant C
Note right of C: 没人陪我玩
```

## 编辑器

常用的Markdown 编辑器

### OSX

VSCode

Atom

Byword

Mou

Typora

MacDown

RStudio

### Linux

VSCode

Atom

Typora

ReText

UberWriter

RStudio

### Windows

VSCode

Atom

CuteMarkEd

MarkdownPad2

Miu

Typora

RStudio

### iOS

Byword

### 浏览器插件

MaDo (Chrome)

Marxico（Chrome）

### 高级应用

Sublime Text 3 + MarkdownEditing / 教程

# 附录

## 编辑器问题

关于Typora的markdown编辑器行间距过大、不一致的问题

解决方案：
在经过查阅资料发现，在Typora中
shift + enter       单回车 
enter                   双回车 