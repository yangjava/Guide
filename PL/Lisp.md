Lisp是一种计算机编程语言，由约翰·麦卡锡（John McCarthy）于1958年发明。“Lisp”这个名称是“List Processing”的缩写，具有表处理的意思...以上大概是公认的比较权威的介绍，网络上很多资料大概都是这么讲的。从这些信息中可以看出，Lisp应该是一种和C、Java类似的计算机编程语言，是一种和中文、英文等所谓自然语言不同的、用来给计算机编程的语言，这或许是大部分人对它的印象。然而，在学习Lisp的过程中，感受到了很多与编程语言常识相背的地方，产生了一些感想，本文就表达一些自己对Lisp的一己之见。

## 1. Lisp语言和方言

如果特指一种语言的话，“Lisp语言”指的应该1958那时John McCarthy发明的那个版本。但是，那个版本的Lisp应该已经停留在书本里了，接下来的几十年里，Lisp快速发展、演变，现代的各种Lisp跟最初的那个版本已经不能称为同一种语言了，只是看上去有点像，都是用包含括号的S-表达式来表示的，但是语言特性和实现已经是天差地别了。那么，现在所说的“Lisp语言”用来指代包含最初的Lisp及其衍生出来的各种方言（例如：Common Lisp，Scheme，Emacs Lisp）在内的一类语言比较合适。

你可能从未听过计算机方言，毕竟没有“C++方言”或者“Java方言”这样的说法，那是因为Java的某一版本明确定义了这个语言这个版本的所有语法，开发者并不能直接修改语法或添加语法。但是，Lisp语言由于提供了扩展词法、语法的功能（例如 Common Lisp 的宏和读取宏），使得Lisp语言的使用者可以给自己手头的Lisp实现添加很多自己想要的语言特性，这样就形成了大量方言，有一些可能连名字都没有。

Common Lisp 和 Scheme 是现在相对主流的两大Lisp方言，各自有着诸多的实现和衍生出来的方言。其余还有 Emacs Lisp、AutoLisp 也仍保持活跃，不过只适合特定环境特定场景，Emacs Lisp 仅用于Emacs编辑器中，而AutoLisp则内嵌于AutoCAD中。还有一些方言用于特定科研领域，在此不多介绍了。

虽说 Common Lisp 和 Scheme 是两大主流Lisp方言，但是并非具体的Lisp实现，仅仅是一种规范，而对应的具体实现也并非只有规范中规定的语言特性和API，通常都包含各自的扩展。下面列出一些常见实现和对应的官方主页（或项目地址）。个人比较推荐 SBCL 和 ChezScheme。

**Common Lisp实现**：

| 名称                          | 官方主页                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| Steel Bank Common Lisp (SBCL) | [http://www.sbcl.org/](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.sbcl.org%2F) |
| GNU CLISP                     | [https://clisp.sourceforge.io/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fclisp.sourceforge.io%2F) |
| Embeddable Common-Lisp (ECL)  | [https://ecl.common-lisp.dev/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fecl.common-lisp.dev%2F) |
| Armed Bear Common Lisp (ABCL) | [https://www.abcl.org/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.abcl.org%2F) |

**Scheme实现**：

| 名称        | 官方主页                                                     |
| ----------- | ------------------------------------------------------------ |
| Chez Scheme | [https://scheme.com/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fscheme.com%2F) 或 [https://cisco.github.io/ChezScheme/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fcisco.github.io%2FChezScheme%2F) |
| Racket      | [https://www.racket-lang.org/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.racket-lang.org%2F) |
| GNU Guile   | [http://www.gnu.org/software/guile/](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.gnu.org%2Fsoftware%2Fguile%2F) |

## 2. Lisp的程序

谈论一门编程语言，没有程序示例，讲得再多也会显得空洞抽象。这一节就展示一下Lisp的程序大概是什么样子的。

首先，我们来看几个 hello world 例子：

- 示例 1. 打印5行helloworld

> 英语：`print 5 lines of "hello world"`

> Lisp：`(print 5 line "hello world")` 或 `(打印 5 行 "hello world")`

- 示例 2. 字符串"hello world"

> 英语：`the string "hello world"`

> Lisp：`(the string "hello world")` 或 `#T(字符串 "hello world")`

- 示例 3. John 说："hello world"

> 英语：`"hello world," John said`

> Lisp：`(say by John "hello world")` 或 `#P(John say "hello world")` 或 `(John 说 "hello world")`

看到这里，或许有人已经把上面的例子在Lisp实现中试了下，然后遇到了一堆错误。并不是我捉弄大家，上面那些确实可以作为Lisp语言的程序，不过缺少对应宏或操作符的实现。好了，言归正传，下面附上能运行几个简单示例（Common Lisp 实现）：

```Lisp
(print "hello world") ;打印"hello world"

(list 1 2 3 4)        ;创建一个列表，包含元素1、2、3、4

(defun test-fun (a b) ;定义函数test-fun执行加法
  (+ a b))

(test-fun 1 2)        ;用1和2来调用函数test-fun

(dolist (i (list 1 2 3 4)) ;对列表进行遍历并打印
  (print i))
```

我们可以看到上述的Lisp程序都有个普遍的特征：都是 `(xxx ......)` 这样的括号包含符号的形式。这里重申下，Lisp是表处理（List Processing）的缩写，而这里括号包起来的就是表，例如，程序`(list 1 2 3 4)`返回的结果`(1 2 3 4)` 就是一个表，包含元素1、2、3、4，而空表就是`()`。（可能有人注意到`#P(...)`还有`#Y[...]`这类表达方式，这是 Common Lisp 提供的读取器宏的机制，由读取器将其转换为由表组成的Lisp程序，可以用来实现一些便利的语法糖，甚至内嵌类似JSON的语法：`#J{ "id" : "XX123" , "type" : 2 }`。）

Lisp的程序本身就是由这样的表组成，用这样的表组织起来的内容基本可以成为Lisp程序。如果懂一些解析器和解释器相关的知识，借助Lisp实现提供的扩展机制，还可以为Lisp加入其他代码组织形式。所以，综上所述，Lisp的代码形式是多变的，限制程序形式的有时候是开发者的想象力和习惯。

接下来附上两段贴近应用的Lisp程序。

首先是一段 Common Lisp 程序，这是借助 SxQL 实现数据库查询的示例，也是基于宏的DSL实现（懂SQL的应该都理解这段程序做了什么事吧）：

```Lisp
(select (:title :author :year)
  (from :books)
  (where (:and (:>= :year 1995)
               (:< :year 2010)))
  (order-by (:desc :year)))
```

下面这段是《Essentials of Programming Languages》书中作者给出的习题参考答案（中文是我额外添加的注释），用Scheme实现，用来构建一个LET语言的解析器：

```Scheme
(define the-lexical-spec                            ;定义词法规范
  '((whitespace (whitespace) skip)                  ;空白字符的词法（引用Scheme的空白字符）和对应操作，skip表示跳过，什么都不做
    (comment ("%" (arbno (not #\newline))) skip)    ;注释的词法（%开头直到行末）和对应操作，arbno类似正则里的“*”，表示“任意个”
    (identifier                                     ;标识符的词法（字母开头加上任意个字母数字符号组合）和对应操作，symbol表示保留为符号
      (letter (arbno (or letter digit "_" "-" "?")))
      symbol)
    (number (digit (arbno digit)) number)           ;数字的词法（1个以上的数字）和对应操作，number表示保留为数字
    (number ("-" digit (arbno digit)) number)))     ;数字的词法（-开头加1个以上的数字）和对应操作，number表示保留为数字

(define the-grammar                                 ;定义语法规范
  '((program (expression) a-program)                ;程序由表达式组成
    (expression (number) const-exp)
    (expression                                     ;表达式可以是“-(1,2)”这样的表达式
      ("-" "(" expression "," expression ")")
      diff-exp)
    (expression                                     ;表达式可以是“zero?(a)”这样的表达式
      ("zero?" "(" expression ")")
      zero?-exp)
    (expression                                     ;表达式可以是“if ... then ... else”这样的表达式
      ("if" expression "then" expression "else" expression)
      if-exp)
    (expression (identifier) var-exp)               ;表达式可以是一个标识符
    (expression                                     ;表达式可以是“let a  = 1 in ...”这样的表达式
      ("let" identifier "=" expression "in" expression)
      let-exp)))

;;;;;;;;;;;;;;;; sllgen boilerplate ;;;;;;;;;;;;;;;;
(sllgen:make-define-datatypes the-lexical-spec the-grammar) ;根据词法和语法规范生成对应数据结构

(define scan&parse
  (sllgen:make-string-parser the-lexical-spec the-grammar)) ;根据词法和语法规范生成对应解析器
```

Lisp的语法是可扩展的，有着动态性极强的宏机制，因此Lisp语言可以自我扩展，甚至在运行时扩展语法，诞生新的语言。另外，Lisp语言编写的代码由表（list）组成（例如，`(+ 1 2 3 4)`，表示对1、2、3、4求和），而表又是Lisp语言中的基本结构之一（例如，`'(+ 1 2 3 4)`或`(list '+ '1 '2 '3 '4)`，表示包含+、1、2、3、4五个元素的表），因此使用Lisp语言操纵修改Lisp语言编写的代码十分地便捷，生成Lisp代码也很简单（例如，`(setf (first '(+ 1 2 3 4)) '-)`，把表中的第一个元素'+'换成'-'，程序就从累加变成了累减）。正因为Lisp语言中数据和代码的高度一致性，写Lisp的人很多都能自己实现Lisp的解释器和编译器。

## 3. Lisp的特点

分析一门语言，不可避免地需要去分析这个语言与其他语言相比的优点和缺点，但是优缺点是容易受到应用场景和开发者能力的影响，可能是“甲之蜜糖，乙之砒霜”的情况。因此，这里还是列举一些Lisp语言的特点，结合开发场景对其使用中可能造成的利弊来展开讨论。

说到特点，可能理解为“特有的点”，Lisp是一门古老的语言，存在已经超过了半个世纪，这段时间里，很多原来Lisp特有的语言特性都被其他语言所接受并吸收，甚至成为业界标配（比如函数递归调用），又从别的语言吸收到很多现代的语言特性（比如面向对象编程）。如果只谈论特有的点，那么就略显片面，所以还是将Lisp作为一门语言的各个方面的特点都说明下。（由于Lisp的各个方言之间有着很大区别，这里暂且谈论所有方言中都有的特点。Common Lisp 和 Scheme 中都有很多值得一提的点，但是还是另写长文来介绍比较好）

### 3.1 语法简单

很少有语言具有比Lisp更简单的语法，正如SICP书中开头提到的“所有的形式化性质可以在一小时内讲完，然后就不用去关注语法细节了（因为根本就没有）”，尤其是Scheme（Common Lisp又稍微复杂点）。做SICP的习题时，大概用了300行不到的代码就实现了Scheme基本语法的解释。现在有很多《xx入门》、《xx天学习xxx》这样的书，基本都两三百页以上，如果哪天有《Scheme入门》之类的，估计100页都撑不满。SICP有400页，已经把简单的Scheme编译器解释器都讲完了。

对于开发者学习而言，这样的简单性极大地缩短了学习时间，把更多的精力放到高级特性和程序设计上。对于语言处理程序（编译器、解释器）而言，简化了解析过程，使得编写语言处理程序也变得简单。

但是这样的简单性带来的也不全然是好处，主要问题还是代码的肉眼解析难度。举个例子说，

```JavaScript
fun1(1,2);    //1

if (type == 0){ //2
    Console.log("...");//... 3
    return 0;          //... 3
}
```

在上述JavaScript代码中不需要仔细看每个字符就可以知道1处是函数调用，2处是if分支，3处是代码块，只要眼睛一瞥。把上面代码换成Common Lisp代码如下：

```Lisp
(fun1 1 2)

(if (= type 0)
  (progn
    (print "...")
    (return 0)))
```

不知道`fun1`是一个函数，还是一个宏或者特殊操作符；看不出`if`是一个特殊操作符，看不出条件和分支；也看不出`(progn ...)`引入了一个类似代码块的形式。更不可能通过简单一瞥看出程序的大体结构，必须观察到字符级别。因为Lisp的代码都是S-表达式，有时候无法在词法和语法层面判断出一个符号到底是函数还是宏，因此编辑器也无法做到像Java那样完美的语法高亮，给工程管理带来了一些难度。

### 3.2 实现和运行原理多样化

介绍一门编程语言必然涉及语言的底层实现/运行原理，例如，C语言可以使用GCC编译器编译为对应平台机器码执行，python语言可以由python解释器来解释执行，而Java语言可以通过Java编译器编译为Java字节码然后由Java虚拟机解释执行。每个语言也都有自己的官方实现和较为具体的规范。

Lisp并没有所谓的官方实现，Common Lisp 和 Scheme 都是如此，而语言规范也仅仅是语言的规范，没有指定具体的解释行为和编译行为（但还是指定了一些相关的API）。事实上，Lisp语言的运行好像包含了编程语言的所有方式：clisp、guile等大部分实现可以在命令行以repl交互式的形式使用；而SBCL和ChezScheme可以将Lisp编译成本地机器码来执行；ECL可以嵌入到C/C++程序中，clisp可以把代码编译成对应字节码并解释执行字节码；ABCL内嵌到Java程序中运行于Java虚拟机之上，等等。和大部分使用软件实现的语言不同，在20世纪80年代还有用来直接执行Lisp的硬件设备--Lisp机器。那是一种在硬件上实现Lisp程序的解释器的机器，在那个年代曾风靡一时，现在可能只能在博物馆里看到了。

而且，Lisp对编译期和运行期也没有严格的限制，不一定非得编译成独立文件执行，也不一定非得解释执行。在一段 Common Lisp 程序中，甚至可以做下面这些事：

```Lisp
(setf a 1 
      fun-name 'add1)                          ;设置符号a为1，函数名fun-name为add1
(setf exp1 (list '+ 'a 1))                     ;生成表达式(+ a 1)，并设置给符号exp1
(eval exp1)                                    ;对符号exp1的值进行解释求值
(setf fun-def-exp `(defun ,fun-name (a) ,exp1));生成函数定义表达式(defun add1 (a) (+ a 1))
(eval fun-def-exp)                             ;解释并求值上面函数的定义
(compile fun-name)                             ;编译该函数
(funcall fun-name 2)                           ;调用该函数
```

这样多样的实现使得使用者可以根据场景找到并选择适合自己的实现：编程成本地机器码的SBCL和ChezScheme适合关注性能的场合；具有良好交互式体验的clisp可以作为命令行的REPL；精简的ABCL和ECL可以分别内嵌到Java和C/C++的程序中作为内置脚本引擎来用，等等。丰富的运行方式也使得在程序运行时生成、编译并加载代码成为可能，让开发者不再受制于程序的执行方式。

但是，“多样化”常常和“分裂”挂钩，各种实现都包含自己的扩展功能，甚至扩展语法（例如，Racket都快扩展成另一种语言了）。按照语言规范写的程序在各个实现上移植性和向后兼容性都不错，但是如果使用了一些实现扩展的功能，那么这个程序的移植性就受影响。例如，日本的 Common Lisp 开发者深町英太郎开发的名为[woo](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Ffukamachi%2Fwoo)的http服务器有着极为优异的性能，但是用到了SBCL特有的功能而无法在其他 Common Lisp 实现上使用。

### 3.3 自动内存管理（GC）

即便在这个GC满大街的时代，这也是一个不想多说但又不得不说的特点...

Lisp是历史上第一个采用垃圾回收的语言，这在那个年代是突破性的（“Lisp之父”同时也是“GC之父”）。这种内存管理方式增强了内存安全性，减少了开发者管理内存带来的心智压力，也使得语言变得更加简单，特定算法还可以提高内存的连续性。相比于C语言开发者排查内存泄漏的问题，C++开发者折腾各种引用、指针，还有Rust开发者跟编译器斗智斗勇，使用带垃圾回收的语言进行开发省心很多。Lisp强大的表达能力、自由的风格加上自动内存管理使得开发者可以把精力完全投入到程序设计中而不必过多关注内存问题。夸张的说，这是一个可以直接提升开发者幸福感的特性。

不过垃圾回收机制的弊端同样成为Lisp的弊端，最大暂停时间的问题使得无法应用于实时性要求较高的场景，而堆使用效率降低也导致无法运行于资源紧张的低端嵌入式设备上。另外，距离第一个垃圾回收算法的提出已经超过半个世纪了，但为应用场景选择合适的垃圾回收算法，调整GC参数，并编写和垃圾回收算法相契合的程序，仍然是一门高超且稀有的手艺。

### 3.4 First-class Function和函数式编程

Lisp是一个提供 First-class Function（这里暂且翻译为“第一类函数”）的语言，因此也是一个支持函数式编程的语言。这里的第一类函数具有如下性质：

1. 可以用变量命名
2. 可以提供给过程作为参数
3. 可以由过程作为结果返回
4. 可以包含在数据结构中

用JavaScript程序表示就是：

```JavaScript
var f = (a) => a + 1                //1
(function (a) {
  a()
})(() => console.log("hello"))      //2
function test (x){
  return (a) => a + x               //3
}
[1, 2, () => console.log("hello")]  //4
```

近似的 Scheme 程序是：

```Scheme
(define f (lambda (a) (+ a 1)))      ;1
((lambda (a) (a)) 
  (lambda () (printf "hello")))      ;2
(define test (lambda (x)
  (lambda (a) (+ a x))))             ;3
(list 1 (lambda () (printf "hello")));4
```

如上所述，在Lisp里函数使用起来就跟数字、字符串一样，这为函数式编程提供了基础。与Java、Python这些“加入”了函数式特性的语言相比，具有更好的体验。

不过Lisp是建议/支持使用函数式编程，而非强制使用，也没有过多强调纯函数。它并非所谓的“函数式语言”，确切说应该是“多范式语言”，除了函数式编程外，不管是面向对象编程、面向语言编程、逻辑式编程等都是可以实现的（如果做不到，那就扩展语法来做）。因此，语言提供的基础特性里缺少一些先进的/激进的函数式特性和功能模块（例如，不可变数据结构和惰性求值），只能自己去扩展实现或找一些开源实现。

### 3.5 代码就是数据

在大部分流行的编程语言中，编写的程序可以构建出各种各样复杂的数据结构和计算过程，但是构建出这些东西的代码本身处于特殊的地位，甚至是只读的、受保护的，例如Java和C中访问受限的方法区和代码区。而且大部分语言因为复杂的语法使得经过解析后的代码结构也非常复杂。这些原因也增加了这些语言的代码生成程序和代码处理工具的开发难度。但是在Lisp中，这个问题得到了很好的解决。Lisp程序本身就是由Lisp基础列表结构组织起来，非常贴近抽象语法树本身；另外，Lisp对符号数据的支持也使得代码中的标识符、关键字可以简单的表示。这两点极大地简化的Lisp代码的操作过程。

举例来说：

```Lisp
(list 1 2 3 4)        ;1. 得到列表(1 2 3 4)
(quote (list 1 2 3 4));2. 得到列表(LIST 1 2 3 4)
'(list 1 2 3 4)       ;3. 得到列表(LIST 1 2 3 4)
```

上面这段 Lisp 代码中，1处代码是一个函数调用，构建出对应数据的列表；2处的通过`quote`操作得到上面代码的表结构数据（列表中“LIST”就是一个符号）；3处的代码是2处的简写。这样我们就可以轻而易举的做到下面这些事：

```CommonLisp
(let ((exp '(list 1 2 3 4)))    ;将一个表达式作为列表数据绑定给符号exp
  (print (length (cdr exp)))    ;获取表达式的参数数量
  (setf exp (cons '+ (cdr exp)));将表达式的操作替换为“+”
  (eval exp))                   ;对新的表达式进行求值
```

就像 John McCarthy 所说的：“LISP程序作为LISP数据的表示，可以被对象程序操作。这就防止了系统程序员和应用程序程序员之间的分离。”在其他编程语言中，开发语言的人和使用语言之间所面对的数据、所拥有的权限有着天壤之别；在Lisp中，这两种人并没有那么大的差别，所面对的都是Lisp的表结构。

### 3.6 自由扩展

要给程序添加新功能时，C语言添加新的结构体和函数，Java添加新的类，各个语言都都有自己的一套方案，甚至一些多范式语言提供了多套方案（例如，C++和Scala）。但是，在追求简洁、高度抽象的表达时，语言的语法都会成为一道屏障，将开发者限制在一个圈子里，或者迫使开发者拐弯抹角地来实现想要的功能，面对复杂的现实，只好“适当的领域使用适当的语言”。

相比之下，Lisp也有自己的方案，但是Lisp并不将开发者限定在“自己的方案”里，它给开发者提供了各种扩展自身的机制（如：Common Lisp里的宏和元对象协议）。例如，下面这段代码利用Common Lisp 的读取器宏来给Common Lisp加入了类似切片的语法：

```Lisp
(set-macro-character #\] (get-macro-character #\)))
(set-dispatch-macro-character #\# #\[
  #'(lambda (stream char1 char2)
      (declare (ignore char1 char2))
      (let ((accum nil)
            (pair (read-delimited-list #\] stream t)))
        (do ((i (ceiling (car pair)) (1+ i)))
            ((> i (floor (cadr pair)))
             (list 'quote (nreverse accum)))
          (push i accum)))))

#[2 7] ;产生 (2 3 4 5 6 7)
```

其他语言的表达遇到不适用的场景，那么你就“另请高明”吧；在Lisp的表达遇到不适用的场景，那你就让Lisp来适合这个场景。这也使得Lisp受很多崇尚自由的黑客的欢迎。就如 John McCarthy 所说的的那样：“每个人都可以“改进”自己的LISP，而这些“改进”中有许多已经发展成为对语言的改进。”一个“精通”Java的人不一定能实现Java，但是一个“熟悉”Lisp的人一般都能实现一个可运行的Lisp。

自由促进了多样化，也促进了分裂。各种方言、各种程序设计方法的实现方式层出不穷，想要给Lisp加入面向对象编程、逻辑式编程就和上学时的课后练习一样，一天开发几百行代码就解决了。虽然这让开发者变得更加“独立”，但也使得社区不像其他语言的社区那么热闹，联系那么紧密。

另外，这样的自由也为项目失控埋下隐患。有一句玩笑话说“Lisp语言跑着跑着跑成了另一门语言，Lisp程序跑着跑着跑成了另一个程序”，这样的动态性如果在可控制范围内那么会是一个巨大的优点，但是一旦失控很可能是致命的。Lisp开发者如果不自律，完全放飞自我，容易写出高度抽象又毫无章法的程序，这样排查问题时远比其他语言的情况更痛苦。

## 4. Lisp相关的一些建议

最后，如果有对Lisp产生兴趣的朋友，给一些建议（真正的一己之见）。

Lisp适合具有以下特征的人群学习（适合人群）：

1. 希望开阔自己的眼界（建议把SICP或PAIP读了，可能会有从井底之蛙进入外太空的感觉）。
2. 对计算机科学（尤其是PL领域）感兴趣（可以找到很多教材和论文，Lisp在工程领域的书比较少）。
3. 不想写代码，想生成代码（学习Lisp可以加深用程序处理代码和生成代码的理解）。
4. 想要实现编程语言（Lisp是捷径，虽然现在大量语言解释器都用C语言实现，但借助C语言来学习解释器是一条漫长且坎坷的路）。

但是如果包含以下特征，还是不建议花费时间学习Lisp（不适合人群）：

1. 期望直接得到项目解决方案或直接应用到工作中去（Lisp的三方库缺乏，中文资料少）；
2. 对编程语言有宗教式的崇拜（因崇拜而导致不重视Lisp语言设计上的缺点是很危险的）；
3. 时间、精力过于宝贵的人（Lisp的很多好书都是需要花大量时间钻研、做习题甚至啃生肉的，修行道路更长，天花板更高）。

推荐几本相关读物：

1. 《ANSI Common Lisp》Paul Graham 著，适合 Common Lisp 入门。电子版链接：[https://acl.readthedocs.io/en/latest/zhCN/](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Facl.readthedocs.io%2Fen%2Flatest%2FzhCN%2F)
2. 《实用Common Lisp》（英文名《Practical Common Lisp》），适合 Common Lisp 入门。
3. 《程序设计方法》（英文名《How to Design Programs》，简称HTDP），适合Scheme（确切说是Racket）入门、程序设计入门，也是很多世界一流名校的教材。
4. 《On Lisp》Paul Graham 著，适合 CommonLisp 宏进阶学习，里面包含很多宏的实践。
5. Daniel P. Friedman的《The Little 。。。》系列，有好几本，包含Scheme的入门和一些实践，范围包括函数式编程、逻辑式编程、类型系统、程序验证等等，风格深入浅出。
6. 《计算机程序的构造和解释》（英文名《Structure and Interpretation of Computer Programs》，简称SICP），是一本魔法书，适合进阶。
7. 《Essentials of Programming Languages》（中文意思是编程语言概要，简称EOPL），也是Daniel P. Friedman所著，适合PL（Programming Language）入门，可以加深对Lisp或者说对编程语言的理解。
8. 《Common Lisp the Language, 2nd Edition》Common Lisp的权威之作，适合完整学习以掌握 Common Lisp 的全貌，也适合查阅。
9. 《Paradigms of Artificial Intelligence Programming: Case Studies in Common Lisp》（中文意思是人工智能编程范例（Common Lisp案例研究），简称PAIP），Peter Norvig著，适合Common Lisp进阶，包含面比较广，涉及编程语言、自然语言处理、逻辑编程、搜索等等等等。