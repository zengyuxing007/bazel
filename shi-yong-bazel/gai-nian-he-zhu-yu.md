* 介绍 Introduction
* 工作区, 包和目标  Workspace. Packages and Targets

  * 工作区\(Workspace\)
  * 包\(Packages\)
  * 目标\(Targets\)
  * 标签\(Labels\)
  * 标签规范
  * 规则\(Rules\)

* BUILD文件

  * 核心构建语言
  * 声明构建规则

* 构建规则的类型

* 依赖

  * 实际依赖和声明依赖

  * 依赖类型

  * 使用标签引用目录

# 介绍

Bazel从一个叫做工作区的目录中的源码构建软件, 工作区中的源文件组织在一个嵌套的层次结构中，其中每个包是一个包含一组相关源文件和一个BUILD文件的目录。 BUILD文件指定了从源文件构建出什么 样的软件.

# 工作区, 包和目标\(Workspace, Packages, Targets\)

## 工作区

一个工作区就是一个在你的文件系统中的目录, 它包含了你要构建的软件的源文件, 和包含构建输出目录的链接符号. 每一个工作区包含 了一个叫做WORKSPACE的文本文件, WORKSPACE文件可以是空的, 或者包含了需要的外部依赖的引用.  你可以在构建百科中查看工作区规则 .

## 包

在工作区中,最主要的代码组织单元就是包.  包就是一组相关的文件和它们之间依赖关系的一种规范.

包被定义为一个包含了名叫BUILD文件的目录. 它在工作区目录之下. 包中包含 其目录中的所有文件,以及它下面的所有子目录, 但是其中也包含了BUILD文件的文件夹除外.

例如, 在下面的目录树中, 有两个包: my/app 和子包 my/app/tests. 注意: my/app/data不是一个包, 它是属于包my/app的一个目录

```
src/my/app/BUILD
src/my/app/app.cc
src/my/app/data/input.txt
src/my/app/tests/BUILD
src/my/app/tests/test.cc
```

## 目标

包是一个容器, 组成包的元素叫做目标\(targets\).  大多数目标\(targets\) 属于 文件和规则 . 此外,还有另外一种目标: 包组\(package groups\), 这种目标非常少.

目标层次

文件进一步划分为两种,. 源文件通常由程序员编写并且要提交到代码库中的. 生成的文件, 有时被称做衍生文件, 是由构建工具根据指定的规则生成, 不会被提交到代码库中.

第二种目标就是规则, 规则定义了一系列输入和输出文件之间的关系, 其中最重要的一步就是从输入构建输出.  输出的规则通常是生成文件. 输入的规则一般是源文件, 但有时候输出生成的文件也可以做为一个规则的输入; 所以输出的规则可能是另一个规则的输入. 允许定义一个很长的规则链.

大多数情况下, 一个规则的输入是源文件, 还是生成的文件并不重要, 重要的是文件的内容. 这样使得使用生成的文件替代复杂的源文件变得容易. 例如在某些场景:场当手动维护高度结构化的文件变成一种负担时,我们可能会用程序导出这种文件, 这种文件对使用的人来说不需要修改.  相反如果需要修改可以用源文件轻松的替换生成的文件.

输入的规则可能包含了其它的规则.  这种关系的确切含往往相当复杂. 但直觉上比较简单: C++库的规则A可能用另外一个C++库B的规则作为输入, 这种依赖的影响是在编译期B的头文件对A可用; B的符在链接期间对A可用; B的运行时数据在执行期间对A可用.

对于所有规则, 一个规则生成的文件总是属于和规则相同的包, 不可能将文件生成到另一个包中.  一个包中规则的输入来自另一个包中的规则的情况也不常见.

包组 是用来限制某些规则的访问权限的一组包, 包组使用`package_group`函数定义.它有两个属性:包的列表及其名称.  引用它们的唯一方法是通过规则的`visibility`属性, 或者 package函数的 `defalut_visibility` 属性. 它们不生成或者使用文件, 更多信息,请参考构建百科章节.

## 标签

所有的目标都属于一个包, 包的名称叫做标签. 一个经典的标签的规范形式如下所示:

```
//my/app/main:app_binary
```

每个标签有两部分组成: 包的名称\(my/app/main\)和目标名称\(app\_binary\). 每一个标签唯一标识一个目标.  标签有时以另外一种形式出现, 当冒号省略时. 认为目标的名称与包名的最后一部分相同, 所以下面两个标签是相等的:

```
//my/app
//my/app:app
```

诸如 //my/app 之类的短格式的标签 不能和包名混淆.  标签以 // 开头, 但是包不是. 因此, my/app 是一个包, 并且包含了 //my/app . \(一个常见的误解是//my/app指的是一个包, 或者一个包中的所有目标. 都不对\)

在一个BUILD文件中, 标签中的包名部分可能会省略, 冒号也可能会省略.  所以对于包my/app的BUILD文件的标签\(例如: //my/app:app\)来说, 以下几种 "相对的" 的标签来说是相等的:

```
//my/app:app
//my/app
:app
app
```

\(It is a matter of convention that the colon is omitted for files, but retained for rules, but it is not otherwise significant.\)

类似地，在BUILD文件中，属于该包的文件可能被相对于包目录的未修饰的名称引用:

```
generate.cc
testdata/input.txt
```

对于其它包或者命令行来说, 这些文件目标必须通过完整的标签名引用, 例如:

```
//my/app:generate.cc
```

相对标签不能用于指向其它包中的目标; 在这种情况下必须要指定完整的包名. 例如: 有两个包`my/app`和包`my/app/testdata`\(每个包都有它自已的BUILD文件\), 第二个包中有一个名叫`testdepot.zip`的文件. 这里`//my/app:BUILD`有两种方式引用这个文件\(一个对的,一个错的\):

```
testdata/testdepot.zip  # Wrong: testdata is a different package.
//my/app/testdata:testdepot.zip   # Right.
```

如果错误的使用标签引用`testdepot.zip`文件, 比如 //my/app:testdata/testdept.zip或者//my:app/testdata/testdepot.zip, 构建工具会报"crosses a package boundary"的错误. 因此您应该通过将冒号放在包含最内层的BUILD文件的目录之后来更正标签, 比如: `//my/app/testdata:testdepot.zip`.

## 标签规范

标签的语法是有意设计成的严格的，以便禁止出现对shell有特殊意义的元字符。 这有助于避免非预期的引用问题，并且操作标签时可以更容易地操作构建工具和编写脚本，例如Bazel查询语言。 标签中禁止以下所有内容：任何类型的空白，大括号，括号或括号; 通配符，如 \*;  shell元字符，如＆，\|; 这个列表不全面; 具体细节如下。

#### 目标名称, //... :** **tartget-name

目标名称是目标在包内的名称. 规则的名称是BUILD文件中规则声明中的name参数的值；文件的名称是相对于包含BUILD文件的目录的路径名. 目标名称必须完全由`a-z, A-Z, 0-9` 的字符和标点符号 `_/.+-=~`组成. 不能使用相对路径`..`引用其它包的文件；使用 `//packagename:file`代替. 文件名必须是正常形式的相对路径名，这意味着它们既不能以斜线开头也不能以斜线结尾（例如:`/foo`和`foo/`被禁止），也不得包含多个连续的斜杠作为路径分隔符（例如: `foo// bar`）。类似地，上级引用（`..`）和当前目录引用（`./`）被禁止。 这个规则的唯一例外是目标名称可能完全包含 “`.`” 。

虽然通常使用`/`以文件目标的名称，我们建议您避免在规则名称中使用`/`。 特别是当使用标签的简写形式时，可能会使阅读器混淆。 标签`//foo/bar/wiz`永远是`//foo/bar/wiz:wiz`的缩写，即使没有的包`foo/bar/wiz`; 它不会引用`// foo:bar/wiz`，即使该目标存在。

然而，在某些情况下，使用斜杠是方便的，有时甚至是必要的。 例如，某些规则的名称必须与其主文件源文件匹配，这些文件可能位于该包的子目录中。

#### 包名   //package-name : ...

包名是包含了BUILD文件的目录的目录名, 相对于源代码的顶级目录, 例如 `my/app`. 包名必须由 `A-Z, a-z, 0-9, '/','-','.'`和`'_'`组成, 不能以斜杠开头.

对于具有对其模块系统（例如Java）有意义的目录结构的语言，重要的是选择作为该语言的有效标识符的目录名称。

虽然Bazel允许在构建根目录下使用一个包（例如：`//:foo`），但是这并不建议这样做，项目应该尝试使用更多描述性命名的包。

包名称可能不包含子字符串`//`，也不能以斜线结尾。

## 规则

规则指定了输入和输出之间的关系, 还有构建输出的步骤.规则可以是许多不同种类或类别之一，它们生成编译的可执行文件和库，测试可执行文件和其他支持的输出，如“构建百科”中所述。

每个规则都有一个名字, 使用字符串类型的`name`属性指定. 该名称必须是一个语法有效的目标名称，如上所述. 在某些情况下，名称有些随意，更有趣的是规则生成的文件的名称; 这是真实的genrules。 在其他情况下，对于`* _binary`和`* _test`规则, 名称是重要的，例如，规则名称决定了构建的可执行文件的名称。

每个规则都有一组属性; 给定规则的适用属性以及每个属性的含义和语义是规则类的函数; 有关支持的规则及其相应属性的完整列表，请参阅构建百科。每个属性都有一个名称和一个类型。 属性可以包含的全部类型是：整数，标签，标签列表，字符串，字符串列表，输出标签，输出标签列表。 在每个规则中都不需要指定所有属性。 因此，属性从键（名称）形成可选的类型值的字典。

许多规则中存在的`srcs`属性属于“标签列表”类型。 其值（如果存在）是标签列表，每个标签是作为此规则输入的目标的名称。

许多规则中存在的outs属性属于"输出标签列表"类型 , 它和`srcs`属性相似，但在两个重要方面有所不同。 首先，由于规则的输出属于与规则本身相同的包，因此输出标签不能包含包组件; 它们必须是上面显示的“相对”形式之一。 其次，（普通）标签属性暗示的关系与输出标签所暗示的关系相反：规则取决于其`srcs`，而规则取决于它的出口。 因此，两种类型的标签属性将方向分配给目标之间的边缘，产生依赖图。

下图表示构建依赖关系图的示例片段，并说明：文件（圆圈）和规则（框）; 从生成的文件到规则的依赖关系; 从规则到文件的依赖关系，从规则到其他规则。 通常，依赖箭头表示为从目标指向其先决条件。

`(注:此处所说的图,官方文档上也没有)`

源文件，规则和生成的文件。

这个针对目标的非循环图被称为“目标图”或“构建依赖图”，并且是Bazel查询工具运行的域。

## BUILD文件

前面的部分描述了包，目标和标签以及构建依赖关系图。 在本节中，我们将介绍用于定义包的具体语法。

根据定义，每个包都包含一个BUILD文件，这是一个用构建语言编写的简短程序。 大多数BUILD文件似乎只是一系列构建规则的声明; 确实，在编写BUILD文件时，强烈地鼓励声明方式。

然而，构建语言实际上是一种命令式语言，而BUILD文件被解释为一个连续的语句列表。 构建规则函数（如cc\_library）是其作用在构建工具中创建抽象构建规则的过程。

BUILD文件的具体语法是Python的一个子集。 最初的语法是Python的，但经验表明，用户很少使用Python的功能，而且当它们这样做时，往往会导致复杂而脆弱的BUILD文件。 在许多情况下，使用这些特征是不必要的，并且可以通过使用外部程序来实现相同的结果，例如， 通过genrule构建规则。

至关重要的是，构建语言中的程序无法执行任意I / O（尽管很多用户尝试！）。 这种不变量使得BUILD文件的解释成为封闭的，即仅依赖于一组已知的输入，这对于确保构建是可重现的必需的。

## 核心构建语言

Lexemes：核心语言的词法语法是Python 2.6的严格子集，我们建议读者参考Python规范以了解详细信息。 不支持的Python的词法功能包括：字符串文字中的浮点文字，十六进制和Unicode转义。

BUILD文件应仅使用ASCII字符，尽管技术上它们使用Latin-1字符集进行解释。 使用`coding：`声明编码是禁止的。

语法：核心语言的语法如下所示，使用EBNF符号。 使用Python定义的优先级解决歧义。

```
file_input ::= (simple_stmt? '\n')*

simple_stmt ::= small_stmt (';' small_stmt)* ';'?

small_stmt ::= expr
             | assign_stmt

assign_stmt ::= IDENTIFIER '=' expr

expr ::= INTEGER
       | STRING+
       | IDENTIFIER
       | IDENTIFIER '(' arg_list? ')'
       | expr '.' IDENTIFIER
       | expr '.' IDENTIFIER '(' arg_list? ')'
       | '[' expr_list? ']'
       | '[' expr ('for' IDENTIFIER 'in' expr)+ ']'
       | '(' expr_list? ')'
       | '{' dict_entry_list? '}'
       | '{' dict_entry ('for' IDENTIFIER 'in' expr)+ '}'
       | expr '+' expr
       | expr '-' expr
       | expr '%' expr
       | '-' expr
       | expr '[' expr? ':' expr? ']'
       | expr '[' expr ']'

expr_list ::= (expr ',')* expr ','?

dict_entry_list ::= (dict_entry ',')* dict_entry ','?

dict_entry ::= expr ':' expr

arg_list ::= (arg ',')* arg ','?

arg ::= IDENTIFIER '=' expr
      | expr
```

对于核心语言的每个表达式，语义与相应的Python语义相同，但在以下情况下除外：不支持二进制`％`运算符的某些重载。 只支持`int％int`和`str％tuple`表单。 只能使用`％`s和`％`d格式说明符;`％（var）s`是非法的。

缺少了许多Python的特性：流程控制（循环，条件，异常），基本数据类型（浮点数，大整数），导入和模块系统，支持类的定义，还有一些Python的内置函数。 功能定义和语句只允许在扩展文件（.bzl）中使用。 库部分记录了可用的功能。

## 声明构建规则

构建语言是一种必要的语言，因此一般来说，顺序是重要的：例如,在使用变量之前必须定义变量. 大多数BUILD文件由声明的构建规则组成, 并且这些语句的相对顺序是无关紧要的; 重要的是声明了哪些规则, 和规则的值. 所以在简单的BUILD文件中,规则的声明可以自由重新排序,而不会改变形为.

鼓厉BUILD文件的作者使用注释来扫述每个构建目标的作用.无论是为了公开使用, 还是帮助其它使用者或者维护者了解构建的功能. 通过在文件顶部添加 `#Description:注释内容`的方式描述.

支持`＃...`的Python注释语法。 三个引号的字符串文字可能会跨多行，并可用于多行注释。

## 构建规则的类型

大多数的构建规则是相似的, 按照语言分组. 例如 `cc_binary`_, _`cc_library` 和 cc\_test是用来构建c++二进制程序,库和测试的规则.其他语言使用相同的命名方案，使用 不同的前缀，例如 java\_ \* 是用来构建Java的规则。 这些功能全部记录在"构建百科"中。

* \* \_binary规则以给定语言构建可执行程序。 构建后，可执行文件将输出到构建工具的二进制输出目录中，并以规则标签的相应名称存在，因此`//my:program`将出现在（例如）`$(BINDIR)/my/program）`中。

此类规则还会创建一个包含属于该规则的数据属性中提及的所有文件的runfiles目录，或其传递依赖关系的任何规则; 这组文件聚集在一起，方便部署到生产中。

* \* \_test规则是一个\* \_binary规则的专用化，用于自动化测试。 测试只是成功返回零的程序。

像二进制文件一样，测试也有runfiles树，其下的文件是测试在运行时合法打开的唯一文件。 例如，程序`cc_test（name ='x'，data = ['// foo：bar']）`可以打开并读取.

`$TEST_SRCDIR/workspace/foo/bar`执行期间。 （每个编程语言都有自己的实用功能，用于访问$ TEST\_SRCDIR的值，但它们都等价于直接使用环境变量。）如果在远程测试主机上执行该操作，不遵守该规则将导致测试失败。

* \* \_library规则以给定的编程语言指定单独编译的模块。 库可以依赖于其他库，二进制和测试可以依赖于库，具有预期的单独编译行为。

## 依赖

如果A在构建或执行时间需要B，则目标A依赖于目标B。这样在目标与目标之间产生了一种非循环的图, 我们称之为依赖图. 目标的直接依赖关系是依赖图中长度为1的路径可达到的其他目标。 目标的传递依赖关系是通过图形通过任何长度的路径依赖于其的目标。

实际上，在构建的上下文中，有两个依赖关系图: 实际依赖关系图和已声明的依赖关系图。 大多数情况下，这两个图形非常相似，不需要进行区分，但下面的讨论是有用的。

### 实际依赖和声明依赖

目标X实际上依赖于目标Y，必须存在，并且是最新的，以便正确构建X。 “构建”过程包括生成文件，处理，编译，链接，归档，压缩，执行或在构建过程中常规发生的任何其他任务。

如果X的包中存在从X到Y的依赖关系，则目标X具有对目标Y的声明依赖性。

对于正确的构建，实际依赖关系的图A必须是声明的依赖关系图D的子图。也就是说，A中每对直接连接的节点x - &gt; y也必须直接连接在D.我们说D 是A.的过度近似\(_overapproximation_\)。

