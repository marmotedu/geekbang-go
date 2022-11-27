## 如何使用 Pflag 给应用添加命令行标识

Go 应用开发中，经常需要给开发的组件加上各种启动参数来配置服务进程，影响服务的行为。像 kube-apiserver 就有多达 200 多个启动参数，而且这些参数的类型各不相同（例如：`string`、`int`、`ip` 类型等），使用方式也不相同（例如：需要支持 `--` 长选项，`-` 短选项等），所以我们需要一个强大的命令行参数解析工具。当前业界最受欢迎的命令行参数解析工具是 [pflag](https://github.com/spf13/pflag)。

pflag 功能强大，那么如何使用呢？接下来，我们就来看下具体如何使用 pflag 来解析命令行参数。

## Pflag 使用介绍

pflag 是标准库 flag 包的一个替代品，其设计的目的就是取代标准库 flag 包，pflag 实现了 POSIX/GNU 风格的命令行参数，同时 pflag 完全兼容 flag，并且提供了更为强大的特性。

pflag 具有如下特性：

* 支持丰富的参数类型，例如除了支持基本类型、切片类型外，还支持高级类型：`count`、`duration`、`ip`、`ipmask`、`map` 等；
* 兼容标准 flag 包的 `Flag` 和 `FlagSet`；
* 支持更高级的功能，比如：`shorthand`、`deprecated`、`hidden` 等。

pflag 非常适合用来构建大型项目，一些耳熟能详的开源项目都是用 pflag 来进行命令行参数解析，例如：kubernetes、istio、helm、docker、etcd 等。

接下来，我们就来介绍下如何使用 pflag。pflag 主要是通过创建 `Flag` 和 `FlagSet` 来使用的。先来看下 `Flag`。

## pflag 包 `Flag` 定义

pflag 可以对命令行参数进行处理，一个命令行参数在 pflag 包中会解析为一个 `Flag` 类型的变量，`Flag` 是一个结构体，定义如下：

```go
type Flag struct {
    Name                string // flag 长选项的名称
    Shorthand           string // flag 短选项的名称，一个缩写的字符
    Usage               string // flag 的使用文本
    Value               Value  // flag 的值
    DefValue            string // flag 的默认值
    Changed             bool // 记录 flag 的值是否有被设置过
    NoOptDefVal         string // 当 flag 出现在命令行，但是没有指定选项值时的默认值
    Deprecated          string // 记录该 flag 是否被放弃
    Hidden              bool // 如果值为 true，则从 help / usage 输出信息中隐藏该 flag
    ShorthandDeprecated string // 如果 flag 的短选项被废弃，当使用 flag 的短选项时打印该信息
    Annotations         map[string][]string // 给 flag 设置注解
}
```

`Flag` 的值是一个 `Value` 类型的接口，`Value` 定义如下：

```go
type Value interface {
    String() string // 将 flag 类型的值转换为 string 类型的值，并返回 string 的内容
    Set(string) error // 将 string 类型的值转换为 flag 类型的值，转换失败报错
    Type() string // 返回 flag 的类型，例如：string、int、ip等
}
```

通过将 flag 的值抽象成一个 interface 接口，可以使我们自定义 flag 类型。

## pflag 包 `FlagSet` 定义

pflag 除了支持单个的 `Flag` 之外，还支持 `FlagSet`。`FlagSet` 是一些预先定义好的 `Flag` 的集和，几乎所有的 pflag 操作，都需要借助 `FlagSet` 提供的方法来完成。在实际开发中，我们可以使用 2 种方法来获取并使用 `FlagSet`：

* 调用 `NewFlagSet` 创建一个 `FlagSet`。
* 使用 pflag 包定义的全局 `FlagSet`: `CommandLine`。实际上 `CommandLine` 也是由 `NewFlagSet` 函数创建的。

先来看下，自定义 `FlagSet`。如下是一个自定义 `FlagSet` 的示例：

```go
var version bool
flagSet := pflag.NewFlagSet("test", pflag.ContinueOnError)
flagSet.BoolVar(&version, "version", true, "Print version information and quit.")
```

我们可以通过定义一个新的 `FlagSet` 来定义命令及其子命令的 `Flag`。

再来看下，使用全局 `FlagSet`。如下是一个使用全局 `FlagSet` 的示例：

```go
import (
    "github.com/spf13/pflag"
)

pflag.BoolVarP(&version, "version", "v", true, "Print version information and quit.")
```

`pflag.BoolVarP` 函数定义如下：

```go
func BoolVarP(p *bool, name, shorthand string, value bool, usage string) {
    flag := CommandLine.VarPF(newBoolValue(value, p), name, shorthand, usage)
    flag.NoOptDefVal = "true"
}
```

可以看到 `pflag.BoolVarP` 最终调用了 `CommandLine`，`CommandLine` 是一个包级别的变量，定义为：

```go
// CommandLine is the default set of command-line flags, parsed from os.Args.
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```

在一些不需要定义子命令的命令行工具中，可以直接使用全局的 `FlagSet`，更加简单方便。

## pflag 使用方法

上面，我们介绍了使用 pflag 包的 2 个核心结构体。接下来，我来详细介绍下 pflag 的常见使用方法。pflag 有很多强大的功能，一些常见的使用方法如下。大概有以下 11 种常见的使用方法。

1) 兼容标准库的 flag 包

如果以前用的是标准库中的 flag 包，在替换为 pflag 时，大部分情况下只需要将导入的 pflag 包重命名为 flag 即可：

```go
import flag "github.com/spf13/pflag"
```

2) 支持多种命令行参数定义方式

pflag 支持以下 4 种命令行参数定义方式：

* 支持长选项、默认值和使用文本，并将标志的值存储在指针中

```go
var name = pflag.String("name", "colin", "Input Your Name")
```

* 支持长选项、短选项、默认值和使用文本，并将标志的值存储在指针中

```go
var name = pflag.StringP("name", "n", "colin", "Input Your Name")
```

* 支持长选项、默认值和使用文本，并将标志的值绑定到变量

```go
var name string
pflag.StringVar(&name, "name", "colin", "Input Your Name")
```

* 支持长选项、短选项、默认值和使用文本，并将标志的值绑定到变量

```go
var name string
pflag.StringVarP(&name, "name", "n","colin", "Input Your Name")
```

上面的函数命名是有规则的：

* 函数名带 `Var` 说明是将标志的值绑定到变量，否则是将标志的值存储在指针中。
* 函数名带 `P` 说明支持短选项，否则不支持短选项。
3) 使用 `Get<Type>` 获取参数的值

可以使用 `Get<Type>` 来获取标志的值，`<Type>` 代表 pflag 所支持的类型。例如：有一个 `pflag.FlagSet`，带有一个名为 `flagname` 的 `int` 类型的标志，可以使用 `GetInt()` 来获取 `int` 值。需要注意 `flagname` 必须存在且必须是 `int`，例如：

```go
i, err := flagset.GetInt("flagname")
```

4) 获取非选项参数

代码示例如下：

```go
package main

import (
    "fmt"

    "github.com/spf13/pflag"
)

var (
    flagvar = pflag.Int("flagname", 1234, "help message for flagname")
)

func main() {
    pflag.Parse()

    fmt.Printf("argument number is: %v\n", pflag.NArg())
    fmt.Printf("argument list is: %v\n", pflag.Args())
    fmt.Printf("the first argument is: %v\n", pflag.Arg(0))
}
```

执行上述代码，输出如下：

```bash
$ go run example1.go arg1 arg2
argument number is: 2
argument list is: [arg1 arg2]
the first argument is: arg1
```

在定义完标志之后，可以调用 `pflag.Parse()` 来解析定义的标志。解析后，可通过 `pflag.Args()` 返回所有的非选项参数，通过 `pflag.Arg(i)` 返回第 `i` 个非选项参数。参数下标 `0` 到 `pflag.NArg() - 1`。

5) 指定了选项但是没指定选项值时的默认值

创建一个 flag 后，可以为这个 flag 设置 `pflag.NoOptDefVal`。如果一个 flag 具有 `NoOptDefVal`，并且该 flag 在命令行上没有设置这个 flag 的值，则该标志将设置为 `NoOptDefVal` 指定的值。例如：

```go
var ip = flag.IntP("flagname", "f", 1234, "help message")
flag.Lookup("flagname").NoOptDefVal = "4321"
```

上面的代码会产生结果如下表所示：

|**命令行参数**|**解析结果**|
|:----|:----|
| --flagname=1357 | ip=1357  |
| --flagname      | ip=4321  |
| [nothing]       | ip=1234  |

6) 弃用标志或者标志的简写

可以弃用标志或者标志的简写。弃用的标志/简写在帮助文本中会被隐藏，并在使用不推荐的标志/简写时打印正确的用法提示，使用方法如下：

弃用名为 `logmode` 的标志，并告知用户应该使用哪个标志代替

```go
// deprecate a flag by specifying its name and a usage message
pflag.CommandLine.MarkDeprecated("logmode", "please use --log-mode instead")
```
这样隐藏了帮助文本中的 `logmode`，并且当使用 `logmode` 时打印了 `Flag --logmode has been deprecated, please use --log-mode instead`。

7) 保留名为 `port` 的标志，但是弃用它的简写形式

```go
pflag.IntVarP(&port, "port", "P", 3306, "MySQL service host port.")

// deprecate a flag shorthand by specifying its flag name and a usage message
pflag.CommandLine.MarkShorthandDeprecated("port", "please use --port only")
```

这样隐藏了帮助文本中的简写 `P`，并且当使用简写 `P` 时打印了 `Flag shorthand -P has been deprecated, please use --port only。usage message` 在此处必不可少，并且不应为空。

8) 隐藏标志

可以将 flag 标记为隐藏的，这意味着它仍将正常运行，但不会显示在 usage/help 文本中。例如：隐藏名为 `secretFlag` 的标志，只在内部使用，并且不希望它显示在帮助文本中或者使用文本中，代码如下：

```go
// hide a flag by specifying its name
pflag.CommandLine.MarkHidden("secretFlag")
```

9) 禁用 flag 的 help 和 usage 的排序

pflag 允许你禁用 flag 的 help 和 usage 的排序，例如：

```go
pflag.CommandLine.BoolP("verbose", "v", false, "verbose output")
pflag.CommandLine.String("coolflag", "yeaah", "it's really cool flag")
pflag.CommandLine.Int("usefulflag", 777, "sometimes it's very useful")
pflag.CommandLine.SortFlags = false
pflag.CommandLine.PrintDefaults()
```

上述代码输出如下：

```bash
-v, --verbose           verbose output
     --coolflag string   it's really cool flag (default "yeaah")
     --usefulflag int    sometimes it's very useful (default 777)
```

10) 支持 Go 标准库 flag 包定义的 flag

为了支持使用 Go 的 flag 包定义的 flag，必须将它们添加到 pflag 标志集中。这通常是支持第三方依赖项定义的标志所必需的(例如 golang/glog)。例如：将 Go flags 添加到 `CommandLine` 标志集中：

```go
package main
import (
	goflag "flag"
	flag "github.com/spf13/pflag"
)

var ip *int = flag.Int("flagname", 1234, "help message for flagname")

func main() {
	flag.CommandLine.AddGoFlagSet(goflag.CommandLine)
	flag.Parse()
}
```

11) 自定义标志名

可以设置一个自定义标志名的 normalization function。它允许标志在代码中创建时以及在命令行上使用某些需要 normalized 的形式时，标记名都会发生 normalized。normalized 形式一般用于比较。下面是使用自定义 normalized 函数的两个示例。

* 使 `-`，`_` 和 `.` 在标志中等效。比如 `--my-flag` 等于 `--my_flag` 等于 `--my.flag`

```go
func wordSepNormalizeFunc(f *pflag.FlagSet, name string) pflag.NormalizedName {
	from := []string{"-", "_"}
	to := "."
	for _, sep := range from {
		name = strings.Replace(name, sep, to, -1)
	}
	return pflag.NormalizedName(name)
}

myFlagSet.SetNormalizeFunc(wordSepNormalizeFunc)
```

* 给两个标志设置别名。例如 `--old-flag-name` 等于 `--new-flag-name`

```go
func aliasNormalizeFunc(f *pflag.FlagSet, name string) pflag.NormalizedName {
	switch name {
	case "old-flag-name":
		name = "new-flag-name"
		break
	}
	return pflag.NormalizedName(name)
}

myFlagSet.SetNormalizeFunc(aliasNormalizeFunc)
```
