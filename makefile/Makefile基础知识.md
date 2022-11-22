## Makefile简介

考虑到一些同学可能不了解Makefile，这里我先来简单介绍下。

简单来说，Makefile是一个工程文件的编译规则，描述了整个工程的编译和链接等规则，这些规则里包含了这些内容：

- 工程中的哪些源文件需要编译，以及如何编译；
- 需要创建哪些库文件，以及如何创建；
- 如何最终生成我们想要的可执行文件。

通过编写Makefile规则，我们可以使上述功能自动化，极大提高工程的编译效率。同时，借助Makefile的其他功能，我们也能完成项目管理的自动化。

在实际使用过程中，一般是先编写一个Makefile文件，告诉整个项目的编译规则，然后通过Linux make命令来解析该Makefile文件，实现项目编译、管理的自动化。

默认情况下，make命令会在当前目录下按如下顺序查找Makefile文件：“GNUmakefile”、“makefile”、“Makefile”的文件，一旦找到，就开始读取这个文件并执行。

这里，**我建议使用“Makefile”文件**名，因为这个文件名第一个字符大写，这样有一种显目的感觉。还有一些make只对全小写的“makefile”文件名敏感。大多数的make都支持“makefile”和“Makefile”这两种默认文件名。make也支持`-f`和`--file`参数来指定其它文件名，比如：`make -f golang.mk`或者`make --file golang.mk`。

## 如何学习Makefile

上面，我们简单介绍了Makefile。那么如何学习Makefile呢？要回答这个问题，我们先来看一下Makefile的组成部分，Makefile脚本文件内容由以下三部分组成：，

- 一系列规则来指定源文件编译的先后顺序。规则是makefile中的重要概念，它一般由目标、依赖和命令组成。
- 特定的语法规则，支持变量、函数和函数调用等。
- 操作系统中的各种命令。

学习Makefile，其实也就是对这3个部分的学习，分别对应于：

- Makefile规则
- Makefile语法
- Shell脚本、Linux命令等

其中Shell脚本、Linux命令这些不是本专栏的核心，所以不做介绍。

当然了，只学习Makefile规则和语法是不够的，这一讲，我还会将我开发中的一些Makefile编写经验分享给你。最后，我也会给你展示一个大型Go项目中Makefile通常需要实现的功能，来让你感受下Makefile强大的项目管理能力。

## Makefile规则介绍

学习Makefile最核心的是要学习Makefile的规则，Makefile这么受欢迎的核心原因也是因为Makefile规则。

我们先来看下Makefile的规则。

### 规则语法

Makefile的规则语法如下：

```makefile
target ...: prerequisites ...
    command
	  ...
	  ...
```
- **target：** 可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。可使用通配符，当有多个目标时，目标之间用空格分隔。
- **prerequisites：** 生成该target所需要的依赖项，当有多个依赖项时，依赖项之间用空格分隔。
- **command：** 该target要执行的命令（任意的shell命令）。
    - 在执行command之前，默认会先打印出该命令，然后再输出命令的结果；如果不想打印出命令，可在各个command前加上`@`。
    - command可以为多条，可以分行写，但每行都要以tab键开始。另外，如果后一条命令依赖前一条命令，则这两条命令需要写在同一行，并用分号进行分隔。
    - 如果要忽略命令的出错，需要在各个command之前加上减号`-`。

**只要targets不存在或prerequisites中有一个以上的文件比targets文件新，command所定义的命令就会被执行。command会产生我们需要的文件或执行我们期望的操作。**

为了加深你的理解，这里我们通过一个例子来说明Makefile的规则：

1) 先编写一个hello.c文件

```c++
#include <stdio.h>
int main()
{
  printf("Hello World!\n");
  return 0;
}
```

2) 当前目录下编写Makefile文件

```makefile
hello: hello.o
	gcc -o hello hello.o
			    
hello.o: hello.c
	gcc -c hello.c
			    
clean:
	rm hello.o
```

3) 执行make，产生可执行文件

```shell
$ make
gcc -c hello.c
gcc -o hello hello.o
$ ls
hello  hello.c  hello.o  Makefile
```

以上示例的Makefile文件有2个target：hello、hello.o，每个target都指定了构建command。当执行make命令时，发现hello、hello.o文件不存在，就会执行command命令生成target。

4) 不更新任何文件，再次执行make

```shell
$ make
make: 'hello' is up to date.
```

当target存在，并且prerequisites都不比target新时，不会执行对应的command。

5) 更新hello.c，并再次执行make

```shell
$ touch hello.c
$ make
gcc -c hello.c
gcc -o hello hello.o
```

当target存在，但prerequisites比target新时，会重新执行对应的command。

6) 清理编译中间文件

Makefile一般都会有一个clean伪目标，用来清理编译中间产物，或者对源码目录做一些定制化的清理：

```shell
$ make clean
rm hello.o
```

我们可以在规则中使用通配符，make 支持三个通配符：*，?和~，例如：

```makefile
objects = *.o
print: *.c
    rm *.c
```

### 伪目标

这里我们重点介绍下Makefile的伪目标，Makefile的管理能力都是通过伪目标来实现的，要执行的功能在Makefile中以伪目标的形式存在。

在上面的Makefile示例中，我们定义了一个clean目标，这个其实是一个伪目标，也就是说我们不会为该目标生成任何文件。因为伪目标不是文件，make 无法生成它的依赖关系和决定是否要执行它，通常我们需要显式地指明这个目标为伪目标。为了避免和文件重名，在Makefile中可以使用`.PHONY`来标识一个目标为伪目标：

```makefile
.PHONY: clean
clean:
    rm hello.o
```

伪目标可以有依赖文件，也可以作为“默认目标”，例如：

```makefile
.PHONY: all
all: lint test build
```

因为伪目标总是会被执行，所以其依赖总是会被决议，通过这种方式，可以达到同时执行所有依赖项的目的。

### order-only依赖

在上面介绍的规则中，只要当prerequisites中有任何文件发生改变时就会重新构造target，但是有时候我们希望只有当prerequisites中的部分文件改变时才重新构造target，这时可以通过order-only prerequisites实现。



order-only prerequisites形式如下：

```makefile
targets : normal-prerequisites | order-only-prerequisites
    command
    ...
    ...
```

在上面的规则中，只有第一次构造targets时才会使用order-only-prerequisites，后面即使order-only-prerequisites发生改变，也不会重新构造targets，而只有normal-prerequisites中的文件发生改变时才重新构造targets。符号|后面的prerequisites即是order-only-prerequisites。

### 引入其它Makefile

在之前的规范介绍中，我们介绍过Makefile要结构化、层次化，这可以通过在项目根目录下的Makefile中引入其他Makefile来实现。

在Makefile中，我们可以通过关键字include，把别的makefile包含进来，类似于C语言的`#include`，被包含的文件会插入在当前的位置。include用法为：`include <filename>`，示例如下：

```makefile
include scripts/make-rules/common.mk
include scripts/make-rules/golang.mk
```

include也可以包含通配符：`include scripts/make-rules/*`。make会按如下顺序找寻makefile文件：

1. 如果是绝对/相对路径，则直接根据路径include进来。
2. 如果make执行时，有`-I`或`--include-dir`参数，那么make就会在这个参数所指定的目录下去找。
3. 如果目录`<prefix>/include`（一般是：`/usr/local/bin`或`/usr/include`）存在的话，make也会去找。

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号`-`。如：`-include <filename>`。

至此，我们介绍了Makefile的规则。接下来，我们再来看下Makefile中的核心语法。

## Makefile语法概览

因为Makefile语法比较多，本讲只介绍Makefile的核心语法，以及 IAM项目Makefile用到的语法。包括：命令、变量、条件语句和函数。Makefile没有太多复杂的语法，掌握了这些知识点之后，再加以融会贯通，就会写出非常复杂，功能强大的Makefile文件。

### 命令

Makefile支持Linux命令，调用方式跟你在Linux系统下调用命令的方式基本一致。默认情况下，make会把其正在执行的命令输出到当前屏幕上，但我们可以通过在命令前加`@`符号禁止make输出当前正在执行的命令，如下Makefile：

```makefile
.PHONY: test    
test:                      
    echo "hello world"
```

执行make命令：

```shell
$ make test
echo "hello world"
hello world
```

可以看到make输出了其所执行的命令。很多时候，我们不需要这样的提示，我们更想看的是命令产生的日志，而不是执行的命令，可以通过在命令行前加`@`禁止make输出所执行的命令，例如：

```makefile
.PHONY: test
test:
    @echo "hello world"
```
再次执行make命令：
```shell
$ make test
hello world
```
可以看到make只是执行了命令，并没有打印命令本身，make输出清晰了很多。
这里，我建议在命令前都加`@`符号，禁止打印命令本身，这样可以使你的Makefile输出易于阅读的、有用的信息。

默认情况下，每条命令执行完make就会检查其返回码，如果返回成功（返回码为0），make就执行下一条指令。如果返回失败（返回码非0），make就会终止当前命令。很多时候，命令出错，我们并不想终止，比如：删除一个不存在的文件。可以通过在命令行前加-符号，来让make忽略命令的出错，比如

```makefile
clean:
    -rm hello.o
```

### 变量赋值

Makefile使用最频繁的语法可能就是变量赋值了，Makefile支持变量赋值、多行变量和环境变量。另外，Makefile还内置了一些特殊变量和自动化变量。

先来看下最基本的**变量赋值**功能。

Makefile也可以像其它语言一样支持变量。在使用变量时，会像shell变量一样，原地展开，然后再执行替换后的内容。可以通过变量声明来声明一个变量，变量在声明时需要赋予一个初值，比如：`ROOT_PACKAGE=github.com/marmotedu/iam`，引用变量时可以通过`$()`或者`${}`方式引用，建议用`$()`方式引用变量：`$(ROOT_PACKAGE)`，也建议整个makefile的变量引用方式要保持一致。变量会像bash变量一样，在使用它的地方展开。比如：

```makefile
GO=go
build:
    $(GO) build -v .
```

展开后为：

```makefile
GO=go
build:
    go build -v .
```

Makefile中一共有4种变量赋值方法。

1) **=** 最基本的赋值方法。

例如：

```makefile
BASE_IMAGE = alpine:3.10
```

使用=进行赋值时要注意，如下的情况：

```makefile
A = a
B = $(A) b
A = c
```
B最后的值为：c b，而不是a b。也就是说，在用变量给变量赋值时，右边变量的取值取的是最终的变量值。

2) **:=** 直接赋值，赋予当前位置的值。

例如：

```makefile
A = a
B := $(A) b
A = c
```
B最后的值为：a b。通过:=可以避免=赋值带来的一些潜在的不一致。

3) **?=** 表示如果该变量没有被赋值，则赋予等号后的值。

例如：

```makefile
PLATFORMS ?= linux_amd64 linux_arm64
```

4) **+=** 表示将等号后面的值添加到前面的变量上。

例如：

```makefile
MAKEFLAGS += --no-print-directory
```

Makefile还支持**多行变量**。可以通过define关键字，设置多行变量，变量中允许换行。定义方式为：

```makefile
define 变量名
变量内容
...
endef
```

变量的内容可以包含函数、命令、文字或是其它变量。例如我们可以定义一个USAGE_OPTIONS变量：

```makefile
define USAGE_OPTIONS    
    
Options:    
  DEBUG        Whether to generate debug symbols. Default is 0.    
  BINS         The binaries to build. Default is all of cmd.    
  ...
  V            Set to 1 enable verbose build. Default is 0.    
endef   
```

Makefile还支**持环境变量**。

Makefile也像Linux一样支持环境变量，在Makefile中有2种环境变量：Makefile预定义的环境变量和自定义的环境变量。

其中自定义的环境变量可以覆盖Makefile预定义的环境变量。默认情况下Makefile中定义的环境变量只在当前Makefile有效，如果想向下层传递（Makefile中调用另一个Makefile），需要使用export关键字来声明，如下声明了一个环境变量，并可以在下层Makefile中使用：

```makefile
...
export USAGE_OPTIONS
...
```


此外，Makefile还支持2种内置的变量：特殊变量和自动化变量。

**特殊变量**是make提前定义好的，可以在makefile中直接引用，特殊变量列表如下：

|变量|含义|
|:----|:----|
|MAKE|当前make解释器的文件名|
|MAKECMDGOALS|命令行中指定的目标名（make的命令行参数）|
|CURDIR|当前make解释器的工作目录|
|MAKE_VERSION|当前make解释器的版本|
|MAKEFILE_LIST|make所需要处理的makefile文件列表，当前makefile的文件名总是位于列表的最后，文件名之间以空格进行分隔|
|.DEFAULT_GOAL|指定如果在命令行中未指定目标，应该构建哪个目标，即使这个目标不是在第一行|
|.VARIABLES|所有已经定义的变量名列表（预定义变量和自定义变量）|
|.FEATURES|列出本版本支持的功能，以空格隔开|
|.INCLUDE_DIRS|make查询makefile的路径，以空格隔开|


Makefile还支持**自动化变量**。自动化变量可以提高我们编写Makefile的效率和质量。

在Makefile的模式规则中，目标和依赖文件都是一系例的文件，那么我们如何书写一个命令来完成从不同的依赖文件生成相对应的目标呢？ 自动化变量就是完成这个功能的，所谓自动化变量，就是这种变量会把模式中所定义的一系列的文件自动地挨个取出，直至所有的符合模式的文件都取完了。这种自动化变量只应出现在规则的命令中。Makefile中支持的自动变量见下表。

|变量|含义|
|:----|:----|
|$@|表示规则中的目标文件集。在模式规则中，如果有多个目标，那么$@就是匹配于目标中模式定义的集合|
|$%|仅当目标是函数库文件中，表示规则中的目标成员名。例如，如果一个目标是foo.a(bar.o)，那么$% 就是bar.o ，$@就是foo.a。如果目标不是函数库文件（Unix下是.a ，Windows下是.lib），那么，其值为空|
|$<|依赖目标中的第一个目标名字。如果依赖目标是以模式（即%）定义的，那么$<将是符合模式的一系列的文件集。注意，其是一个一个取出来的|
|$?|所有比目标新的依赖目标的集合，以空格分隔|
|$^|所有的依赖目标的集合，以空格分隔。如果在依赖目标中有多个重复的，那个这个变量会去除重复的依赖目标，只保留一份|
|$+|这个变量很像$^ ，也是所有依赖目标的集合，只是它不去除重复的依赖目标|
|$\||所有的order-only依赖目标的集合，以空格分割|
|$*|这个变量表示目标模式中%及其之前的部分。如果目标是dir/a.foo.b，并且目标的模式是a.%.b，那么，$*的值就是dir/a.foo。这个变量对于构造有关联的文件名是比较有较。如果目标中没有模式的定义，那么$*也就不能被推导出，但是，如果目标文件的后缀是make所识别的，那么$*就是除了后缀的那一部分。例如：如果目标是foo.c ，因为.c是make所能识别的后缀名，所以$*的值就是foo。这个特性是GNU make的，很有可能不兼容于其它版本的make，所以你应该尽量避免使用$*，除非是在隐含规则或是静态模式中。如果目标中的后缀是make所不能识别的，那么$*就是空值|


上面这些自动化变量中`$*`使用的最多的。

### 条件语句

Makefile也支持条件语句，先来看一个示例，下面的例子判断变量`ROOT_PACKAGE`是否为空，如果为空，则输出错误信息，不为空则打印变量值：

```makefile
ifeq ($(ROOT_PACKAGE),)    
$(error the variable ROOT_PACKAGE must be set prior to including golang.mk)    
else    
$(info the value of ROOT_PACKAGE is $(ROOT_PACKAGE))    
endif
```

条件语句的语法为：

```makefile
# if ... 
<conditional-directive>
<text-if-true>
endif
# if ... else ...
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

例如：

```makefile
ifeq 条件表达式
...
else
...
endif
```

ifeq表示条件语句的开始，并指定一个条件表达式，表达式包含两个参数，以逗号分隔，表达式以圆括号括起。else表示条件表达式为假的情况。endif表示一个条件语句的结束，任何一个条件表达式都应该以endif结束。<conditional-directive>表示条件关键字，有4个关键字：ifeq、ifneq、ifdef、ifndef。为了加深你的理解，我们分别来看下这4个关键字的例子。

1) ifeq

```makefile
ifeq (<arg1>, <arg2>)
ifeq '<arg1>' '<arg2>'
ifeq "<arg1>" "<arg2>"
ifeq "<arg1>" '<arg2>'
ifeq '<arg1>' "<arg2>"
```

比较arg1和arg2的值是否相同，如果相同则为真。也可以用make函数/变量替代arg1或arg2，例如：ifeq ($(origin ROOT_DIR),undefined)或($(ROOT_PACKAGE),)。origin函数在函数一节中会介绍到。

2) ifneq

```makefile
ifneq (<arg1>, <arg2>)
ifneq '<arg1>' '<arg2>'
ifneq "<arg1>" "<arg2>"
ifneq "<arg1>" '<arg2>'
ifneq '<arg1>' "<arg2>"
```

比较arg1和arg2的值是否不同，如果不同则为真。

3) ifdef

```makefile
ifdef <variable-name>
```
如果<variable-name>值非空，则表达式为真，否则为假。<variable-name>也可以是函数的返回值
4) ifndef

```makefile
ifndef <variable-name>
```

如果<variable-name>值为空，则表达式为真，否则为假。<variable-name>也可以是函数的返回值。

### 函数

Makefile同样也支持函数。函数语法包括定义语法和调用语法。我们分别来看下。

**先来看下自定义函数。** make解释器提供了一系列的函数供Makefile调用，这些函数是Makefile的预定义函数，Makefile也支持自定义函数，我们可以通过define关键字来自定义一个函数。自定义函数的语法为：

```makefile
define 函数名
函数体
endef
```

例如，如下是一个自定义函数：

```makefile
define Foo
    @echo "my name is $(0)"
    @echo "param is $(1)"
endef
```

define本质上是定义一个多行变量，可以在call的作用下当作函数来使用，在其它位置使用只能作为多行变量的使用，例如：

```makefile
var := $(call Foo)   
new := $(Foo)
```

自定义函数是一种过程调用，没有任何的返回值。可以使用自定义函数来定义命令的集和，并应用在规则中。

**再来看下预定义函数。** make编译器也定义了很多函数，这些函数叫做预定义函数，调用语法和变量类似，语法为：

```makefile
$(<function> <arguments>)
```
或者
```makefile
${<function> <arguments>}
```

`<function>`是函数名，`<arguments>`是函数参数，参数间以逗号（,）分割。函数的参数也可以为变量。

我们来看一个例子：

```makefile
PLATFORM = linux_amd64
GOOS := $(word 1, $(subst _, ,$(PLATFORM)))
```

上面例子用到了2个函数：word和subst。word函数有2个参数：1和subst函数的输出。subst函数将PLATFORM变量值中的_替换成空格（替换后的PLATFORM值为：linux amd64）。word函数取linux amd64字符串中的第一个单词。所以最后GOOS的值为：linux。

Makefile预定义函数能够帮助我们实现很多强大的功能，在我们编写Makefile的过程中，如果有功能需求，可以优先使用这些函数。如果你想使用这些函数，那就需要知道有哪些函数，以及它们实现的功能。常用的函数，罗列如下：

|函数名|功能描述|
|:----|:----|
|$(origin <variable>)|告诉变量的“出生情况”，有如下返回值：<br><ul><li>undefined：<variable> 从来没有定义过</li><li>default：<variable> 是一个默认的定义</li><li>environment：<variable> 是一个环境变量</li><li>file：<variable> 这个变量被定义在 Makefile中</li><li>command line：<variable> 这个变量是被命令行定义的</li><li>override：<variable> 是被 override 指示符重新定义的</li><li>automatic：<variable> 是一个命令运行中的自动化变量</li>|
|$(addsuffix <suffix>,<names...>)|把后缀<suffix>加到<names>中的每个单词后面，并返回加过后缀的文件名序列。|
|$(addprefix <prefix>,<names...>)|把前缀<prefix>加到<names>中的每个单词后面，并返回加过前缀的文件名序列。|
|$(wildcard <pattern>)|扩展通配符，例如：$(wildcard ${ROOT_DIR}/build/docker/*)|
|$(word <n>,<text>)|取字符串<text>中第<n>个单词（从一开始），并返回字符串<text>中第<n>个单词。如 <n>比<text>中的单词数要大，那么返回空字符串|
|$(subst <from>,<to>,<text>)|把字串 <text> 中的 <from> 字符串替换成 <to>，并返回被替换后的字符串|
|$(eval <text>)|将<text>的内容将作为makefile的一部分而被make解析和执行。|
|$(firstword <text>)|取字符串 <text> 中的第一个单词，并返回字符串 <text> 的第一个单词|
|$(lastword <text>)|取字符串 <text> 中的最后一个单词，并返回字符串 <text> 的最后一个单词|
|$(abspath <text>)|将<text>中的各路径转换成绝对路径，并将转换后的结果返回|
|$(shell cat foo)|执行操作系统命令，并返回操作结果|
|$(info <text ...>)|输出一段信息|
|$(warning <text ...>)|出一段警告信息，而 make 继续执行|
|$(error <text ...>)|产生一个致命的错误，<text ...> 是错误信息|
|$(filter <pattern...>,<text>)|以<pattern>模式过滤<text>字符串中的单词，保留符合模式<pattern>的单词。可以有多个模式。返回符合模式<pattern>的字串|
|$(filter-out <pattern...>,<text>)|以<pattern>模式过滤<text>字符串中的单词，去除符合模式<pattern>的单词。可以有多个模式，并返回不符合模式<pattern>的字串|
|$(dir <names...>)|从文件名序列 <names> 中取出目录部分。目录部分是指最后一个反斜杠（/ ）之前的部分。如果没有反斜杠，那么返回 ./|
|$(notdir <names...>)|从文件名序列<names>中取出非目录部分。非目录部分是指最後一个反斜杠（/）之后的部分。返回文件名序列<names>的非目录部分。|
|$(strip <string>)|去掉<string>字串中开头和结尾的空字符，并返回去掉空格后的字符串|
|$(suffix <names...>)|从文件名序列<names>中取出各个文件名的后缀。返回文件名序列<names>的后缀序列，如果文件没有后缀，则返回空字串。|
|$(foreach <var>,<list>,<text>)|把参数<list>中的单词逐一取出放到参数<var>所指定的变量中，然后再执行<text>所包含的表达式。每一次 <text>会返回一个字符串，循环过程中<text>的所返回的每个字符串会以空格分隔，最后当整个循环结束时，<text>所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。|
