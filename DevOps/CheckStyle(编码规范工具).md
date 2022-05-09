# CheckStyle

## 简介

​      CheckStyle是SourceForge下的一个项目，提供了一个帮助JAVA开发人员遵守某些编码规范的工具。它能够自动化代码规范检查过程，从而使得开发人员从这项重要，但是枯燥的任务中解脱出来。

## 安装

​      安装有两种方式，和SVN类似，即离线安装和在线安装。接下来分别介绍这两种安装方式

### 离线安装

去官网下载CheckStyle插件，官网地址：https://sourceforge.net/projects/eclipse-cs/files/Eclipse%20Checkstyle%20Plug-in/
在某个目录（D:\Java）创建checkstyle\eclipse子目录，将之前下载的插件下的features和plugins两个文件下拷贝到该目录下（D:\Java\checkstyle\eclipse）
接着在eclipse的links目录下新建checkstyle.xml文件，在该文件中添加一行写上需要链接的插件地址，例如：path=D:/Java/checkstyle
注：此种方式便于管理外部Eclipse插件

### 在线安装

打开eclipse，点击“help->Install New Software…”
单击“Add”，输入Name：CheckStyle和Location：http://eclipse-cs.sourceforge.net/update/
单击“OK”，出来两个选项，勾选上然后单击“Next”
出现安装详情，单击“Next”，遇到需要同意协议的，选择同意，直到最后
安装过程中，会有安全提示窗口，点击“OK”
重启后就可以使用了

## 使用

 选择某个项目，点击右键选择“Checkstyle”，会出现如下菜单：

Ⅰ Configure project(s) from blueprint：指定工程加入Checkstyle检查
Ⅱ Activate Checkstyle：激活Checkstyle
Ⅲ Deactivate Checkstyle：取消激活Checkstyle
Ⅴ Check code with Checkstyle：使用Checkstyle检查代码
Ⅳ Clear Checkstyle violations：去掉Checkstyle的检查提示

      选择“Check code with Checkstyle”对所选中的项目进行检查，检查后对有问题的类会使用警告或错误标识。参考提示信息如下：


Checkstyle默认换号是两个空格，我们在项目中一般都是一个tab或四个空格（这里注意，一个tab在eclipse中默认是8个空格），此时就需要我们自定义检查内容

      接下分别介绍Checkstyle中错误信息的含义和自定义检查内容

## 常见提示速查

Checkstyle常见错误和警告提示见下表所示：

错误提示	错误说明
missing a javadoc comment	缺少注释
Line longer than X characters	行长度超过X个字符(包括空格)
Return count is X(max allowed 3)	一个方法内的返回数量是X(最大值只能为3)
Nested if-else depth is X(max allowed is 3)	最大的if-else嵌套层数为X(最大只能为3)
Array brackets at illegal position	数组的方括号“[]”的位置不正确（检查数组类型的定义是String[] args，而不是String args[]）
Line matchs the illegal pattern ‘System.out.println’	本行包含System.out.println语句
ctor def modifier at indentation level 8 not at corrent indentation 4	缩进不正确，一般是因为没有在Eclipse中使用4个空格代替tab键引起
‘static’ modifier out of order with the JLS suggestions	static修饰符没有按照JLS的建议来排序(eg.写成public final static…应该改成public static final)
Name ‘X’ must match pattern ‘^[A-Z][A-Z0-9][_A-Z0-9+]$’(正则表达式)	名称不符合正则表达式’^[A-Z][A-Z0-9][_A-Z0-9+]$’(即为大写字母，数字、下划线等)。一般在静态变量没有大写时提示，包名不是全部消息时提示，类名不是大写开头时提示，方法名不是小写开头时提示
Variable access definition in wrong order	变量定义顺序不正确(例如在类成员变量定义时，将private类型的变量定义在public类型的变量之前)
Static variable definition in wrong order	静态变量定义顺序不正确(例如在构造函数之后定义静态变量)
Instance variable definition in wrong order	成员变量定义顺序不正确(例如在构造函数之后定义成员变量)
X is a magic number	X是一个魔术数字(非0、1、2的数字)
if construct must use ‘{}’	if结构必须使用’{}’
Got an exception - Unexpected character 0xfffd in identifier	因为没有设置checkstyle配置文件的charset为UTF-8，而类文件使用UTF-8编码，并且含有中文
“{” should be on the previous line	“{” 应该位于前一行
Methods is missing a javadoc comment	方法前面缺少javadoc注释
Expected @throws tag for “Exception”	在注释中希望有@throws的说明
“.” Is preceeded with whitespace	“.” 前面不能有空格
“.” Is followed by whitespace	“.” 后面不能有空格
“=” is not preceeded with whitespace“=”	“=” 前面缺少空格
“=” is not followed with whitespace	“=” 后面缺少空格
“}” should be on the same line	“}” 应该与下条语句位于同一行
Unused @param tag for “unused”	没有参数“unused”，不需注释
Variable “X” missing javadoc	变量“X”缺少javadoc注释
Line contains a tab character	行含有”tab” 字符
Redundant “Public” modifier	冗余的“public” modifier
final modifier out of order with the JSL suggestion	final修饰符的顺序错误
Avoid using the “.*” form of import	Import格式避免使用“.*”
Redundant import from the same package	从同一个包中Import内容
Unused import-X Import	import的X类没有被使用
Duplicate import to line X	重复Import同一个内容
Import from illegal package	从非法包中 Import内容
“while” construct must use “{}”	“while” 语句缺少“{}”
Variable “X” must be private and have accessor method	变量“X”应该是private的，并且有调用它的方法
Variable “X” must match pattern “^[a-z][a-zA-Z0-9]*$”	变量“X”不符合命名规则“^[a-z][a-zA-Z0-9]*$”
“(” is followed by whitespace	“(” 后面不能有空格
“)” is proceeded by whitespace	“)” 前面不能有空格
五、自定义检查内容
      Checkstyle提供的默认检查不一定就适用于我们的开发习惯，因此我们可以根据自己的开发习惯指定Checkstyle检查的格式。下面给出制定Checkstyle检查示例

<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
          "-//Puppy Crawl//DTD Check Configuration 1.2//EN"
          "http://www.puppycrawl.com/dtds/configuration_1_2.dtd">

<module name="Checker">
    <property name="severity" value="warning" />
    <module name="StrictDuplicateCode">
        <property name="charset" value="utf-8" />
    </module>

    <module name="TreeWalker">
        <!-- javadoc的检查 -->
        <!-- 检查所有的interface和class -->
        <module name="JavadocType" />
    
        <!-- 命名方面的检查 -->
        <!-- 局部的final变量，包括catch中的参数的检查 -->
        <module name="LocalFinalVariableName" />
        <!-- 局部的非final型的变量，包括catch中的参数的检查 -->
        <module name="LocalVariableName" />
        <!-- 包名的检查（只允许小写字母） -->
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$" />
        </module>
        <!-- 仅仅是static型的变量（不包括static final型）的检查 -->
        <module name="StaticVariableName" />
        <!-- 类型(Class或Interface)名的检查 -->
        <module name="TypeName" />
        <!-- 非static型变量的检查 -->
        <module name="MemberName" />
        <!-- 方法名的检查 -->
        <module name="MethodName" />
        <!-- 方法的参数名 -->
        <module name="ParameterName " />
        <!-- 常量名的检查 -->
        <module name="ConstantName" />
        <!-- 没用的import检查，比如：
        1.没有被用到
        2.重复的
        3.import java.lang的
        4.import 与该类在同一个package的 -->
        <module name="UnusedImports" />
    
        <!-- 长度方面的检查 -->
        <!-- 文件长度不超过1500行 -->
        <module name="FileLength">
            <property name="max" value="1500" />
        </module>
        <!-- 每行不超过120个字 -->
        <module name="LineLength">
            <property name="max" value="120" />
        </module>
        <!-- 方法不超过150行 -->
        <module name="MethodLength">
            <property name="tokens" value="METHOD_DEF" />
            <property name="max" value="150" />
        </module>
        <!-- 方法的参数个数不超过5个。 并且不对构造方法进行检查 -->
        <module name="ParameterNumber">
            <property name="max" value="5" />
            <property name="tokens" value="METHOD_DEF" />
        </module>
    
        <!-- 空格检查 -->
        <!-- 允许方法名后紧跟左边圆括号"(" -->
        <module name="MethodParamPad" />
        <!-- 在类型转换时，不允许左圆括号右边有空格，也不允许与右圆括号左边有空格 -->
        <module name="TypecastParenPad" />
    
        <!-- 关键字 -->
        <!-- 每个关键字都有正确的出现顺序。比如 public static final XXX 是对一个常量的声明。如果使用 static public 
            final 就是错误的 -->
        <module name="ModifierOrder" />
        <!-- 多余的关键字 -->
        <module name="RedundantModifier" />
    
        <!-- 对区域的检查 -->
        <!-- 不能出现空白区域 -->
        <module name="EmptyBlock" />
        <!-- 所有区域都要使用大括号 -->
        <module name="NeedBraces" />
        <!-- 多余的括号 -->
        <module name="AvoidNestedBlocks">
            <property name="allowInSwitchCase" value="true" />
        </module>
    
        <!-- 编码方面的检查 -->
        <!-- 不许出现空语句 -->
        <module name="EmptyStatement" />
        <!-- 不允许魔法数 -->
        <module name="MagicNumber">
            <property name="tokens" value="NUM_DOUBLE, NUM_INT" />
        </module>
        <!-- 多余的throw -->
        <module name="RedundantThrows" />
        <!-- String的比较不能用!= 和 == -->
        <module name="StringLiteralEquality" />
        <!-- if最多嵌套3层 -->
        <module name="NestedIfDepth">
            <property name="max" value="3" />
        </module>
        <!-- try最多被嵌套2层 -->
        <module name="NestedTryDepth">
            <property name="max" value="2" />
        </module>
        <!-- clone方法必须调用了super.clone() -->
        <module name="SuperClone" />
        <!-- finalize 必须调用了super.finalize() -->
        <module name="SuperFinalize" />
        <!-- 不能catch java.lang.Exception -->
        <module name="IllegalCatch">
            <property name="illegalClassNames" value="java.lang.Exception" />
        </module>
        <!-- 确保一个类有package声明 -->
        <module name="PackageDeclaration" />
        <!-- 一个方法中最多有10个return -->
        <module name="ReturnCount">
            <property name="max" value="10" />
            <property name="format" value="^$" />
        </module>
        <!-- 根据 Sun 编码规范， class 或 interface 中的顺序如下： 
        1.class 声明。首先是 public, 然后是protected, 然后是 package level （不包括access modifier ） 最后是private . (多个class放在一个java文件中的情况） 
        2.变量声明。 首先是 public, 然后是protected然后是 package level （不包括access modifier ） 最后是private. (多个class放在一个java文件中的情况） 
        3.构造函数 
        4.方法 -->
        <module name="DeclarationOrder" />
        <!-- 同一行不能有多个声明 -->
        <module name="MultipleVariableDeclarations" />
        <!-- 不必要的圆括号 -->
        <module name="UnnecessaryParentheses" />
    
        <!-- 杂项 -->
        <!-- 禁止使用System.out.println -->
        <module name="GenericIllegalRegexp">
            <property name="format" value="System\.out\.println" />
            <property name="ignoreComments" value="true" />
        </module>
        <!-- 检查并确保所有的常量中的L都是大写的。因为小写的字母l跟数字1太象了 -->
        <module name="UpperEll" />
        <!-- 检查数组类型的定义是String[] args，而不是String args[] -->
        <module name="ArrayTypeStyle" />
        <!-- 检查java代码的缩进 默认配置：基本缩进 4个空格，新行的大括号：0。新行的case 4个空格 -->
        <module name="Indentation" />
    </module>
</module>        
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146

CrapeMyrtles
关注

————————————————
版权声明：本文为CSDN博主「CrapeMyrtles」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/cainiao_ACCP/article/details/77443925