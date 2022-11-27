## 现代化的命令行框架：Cobra 全解

[Cobra](https://github.com/spf13/cobra) 是一个可以创建强大的现代 CLI 应用程序的库，它还提供了一个可以生成应用和命令文件的程序的命令行工具：`cobra-cli`。有许多大型项目都是用 cobra 来构建他们的应用程序，例如：kubernetes、Docker、Etcd、Rkt、Hugo 等。Cobra 具有很多特性，一些核心特性如下：

* 可以构建基于子命令的 CLI，并支持支持嵌套子命令。例如：`app server`，`app fetch`。
* 可以通过 `cobra-cli init appname` & `cobra-cli add cmdname` 轻松生成应用和子命令。
* 智能化命令建议 (app srver... did you mean app server?)。
* 自动生成命令和标志的 help 文本，并能自动识别 `-h` ， `--help` 等标志。
* 自动为你的应用程序生成 bash、zsh、fish 和 powershell 自动补全脚本。
* 支持命令别名、自定义帮助、自定义用法等。
* 可以与 viper、pflag 紧密集成，用于构建 12-factor 应用程序。

Cobra 建立在 commands、arguments 和 flags 结构之上。commands 代表命令，arguments 代表非选项参数，flags 代表选项参数（也叫标志）。一个好的应用程序应该是易懂的，用户可以清晰的知道如何去使用这个应用程序。应用程序通常遵循如下模式：`APPNAME VERB NOUN --ADJECTIVE` 或者 `APPNAME COMMAND ARG --FLAG`，例如：

```go
git clone URL --bare # clone 是一个命令，URL 是一个非选项参数，bare 是一个选项参数
```

这里，`VERB` 代表动词，`NOUN` 代码名词，`ADJECTIVE` 代表形容词。

### `cobra-cli` 命令安装

Cobra 提供了一个 `cobra-cli` 命令，用来初始化一个应用程序并为其添加命令，方便我们开发基于 Cobra 的应用。`cobra-cli` 命令安装方法如下：

```go
$ go install github.com/spf13/cobra-cli@latest
```

`cobra-cli` 命令提供了 4 个子命令：

* `init`：初始化一个 cobra 应用程序；
* `add`：给通过 cobra init 创建的应用程序添加子命令；
* `completion`：为指定的 shell 生成命令自动补全脚本；
* `help`：打印任意命令的帮助信息。

`cobra-cli` 命令还提供了一些全局的参数：

* `-a`, `--author`：指定 Copyright 版权声明中的作者；
* `--config`：指定 cobra 配置文件的路径；
* `-l`, `--license`：指定生成的应用程序所使用的开源协议，内置的有：GPLv2, GPLv3, LGPL, AGPL, MIT, 2-Clause BSD or 3-Clause BSD；
* `--viper`：使用 `viper` 作为命令行参数解析工具，默认为 `true`。

### Cobra 使用方法

> 提示：请确保 Go 版本 >= `1.18`。

在构建 cobra 应用时，我们可以自行组织代码目录结构，但 cobra 建议如下目录结构：

```go
▾ appName/
    ▾ cmd/
        add.go
        your.go
        commands.go
        here.go
      main.go
```

`main.go` 文件目的只有一个：初始化 cobra 应用：

```go
package main

import (
  "{pathToYourApp}/cmd"
)

func main() {
  cmd.Execute()
}
```

### 使用 `cobra-cli` 命令生成应用程序并添加子命令

我们可以选择使用 `cobra-cli` 命令行工具，来快速生成一个应用程序，并为其添加子命令，然后基于生成的代码进行二次开发，提高开发效率，具体步骤如下：

1) 生成应用程序

可以使用 `cobra-cli init` 命令初始化一个应用程序，然后我们就可以基于这个 Demo 程序做二次开发，提高开发效率。如下命令可以初始化一个新的应用程序：

```bash
$ mkdir -p cobrademo && cd cobrademo && go mod init
$ cobra-cli init --license=MIT --viper
$ ls
cmd  go.mod  go.sum  LICENSE  main.go
```

> 提示：如果遇到错误 `Error: invalid character '{' after top-level value)'}'`，可参考：https://github.com/spf13/cobra-cli/issues/26。

当一个应用程序被初始化之后，就可以给这个应用程序添加一些命令：

```bash
$ cobra-cli add serve
$ cobra-cli add config
$ cobra-cli add create -p 'configCmd' # 此命令的父命令的变量名（默认为 'rootCmd'）
$ ls cmd/
config.go  create.go  root.go  serve.go
```

执行 `cobra-cli add` 之后，会在 `cmd` 目录下生成命令源码文件。`cobra-cli add` 不仅可以添加命令，也可以添加子命令，例如上面的例子，通过 `cobra-cli add create -p 'configCmd'` 给 `config` 命令添加了 `create` 子命令，`-p` 指定子命令的父命令：`<父命令>Cmd`。

2) 编译并执行

在生成完命令后，可以直接执行 `go build` 命令编译应用程序：

```bash
$ go build -v .
$ go work use .
$ ./cobrademo -h
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobrademo [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  config      A brief description of your command
  help        Help about any command
  serve       A brief description of your command

Flags:
      --config string   config file (default is $HOME/.cobrademo.yaml)
  -h, --help            help for cobrademo
  -t, --toggle          Help message for toggle

Use "cobrademo [command] --help" for more information about a command.

$ ./cobrademo config -h
......
Usage:
  cobrademo config [flags]
  cobrademo config [command]

Available Commands:
  create      A brief description of your command

Flags:
  -h, --help   help for config

Global Flags:
      --config string   config file (default is $HOME/.cobrademo.yaml)

Use "cobrademo config [command] --help" for more information about a command.
```

这里需要注意：命令名称要是 `camelCase` 格式，而不是 `snake_case` / `snake-case` 格式，如果不是驼峰格式，cobra 会报错。

3) 配置 cobra

cobra 在生成应用程序时，也会在当前目录下生成 `LICENSE` 文件，并且会在生成的 Go 源码文件中，添加 LICENSE Header，LICENSE 和 LICENSE Header 的内容可以通过 cobra 配置文件进行配置，默认的配置文件为：`~/.cobra.yaml`，例如：

```yaml
author: Steve Francia <spf@spf13.com>
year: 2020
license:
  header: This file is part of CLI application foo.
  text: |
    {{ .copyright }}

    This is my license. There are many like it, but this one is mine.
    My license is my best friend. It is my life. I must master it as I must
    master my life.
```

在如上例子中，`{{ .copyright }}` 的具体内容会根据 `author` 和 `year` 生成，根据此配置生成的 LICENSE 文件内容为：

```plain
Copyright © 2020 Steve Francia <spf@spf13.com>
 
This is my license. There are many like it, but this one is mine.
My license is my best friend. It is my life. I must master it as I must
master my life.
 
LICENSE Header为 ：
/*
Copyright © 2020 Steve Francia <spf@spf13.com>
This file is part of CLI application foo.
*/
```

我们也可以使用内建的 licenses，内建的 licenses 有：GPLv2, GPLv3, LGPL, AGPL, MIT, 2-Clause BSD or 3-Clause BSD。例如，我们使用 MIT license：

```bash
$ cobra-cli init --license=MIT
```

### 使用 cobra 库创建命令

如果要用 cobra 库编码实现一个应用程序，需要首选创建一个空的 `main.go` 文件和一个 rootCmd 文件，之后可以根据需要添加其它命令。具体步骤如下；

1) 创建 rootCmd

```bash
$ mkdir -p cobrademolib && cd cobrademolib
```

通常情况下，我们会将 `rootCmd` 放在文件 `cmd/root.go` 中：

```go
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
	Use:   "hugo",
	Short: "Hugo is a very fast static site generator",
	Long: `A Fast and Flexible Static Site Generator built with
         love by spf13 and friends in Go.
				 Complete documentation is available at http://hugo.spf13.com`,
	Run: func(cmd *cobra.Command, args []string) {
		// Do Stuff Here
	},
}

func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

还可以在 `init()` 函数中定义标志和处理配置，例如：`cmd/helper.go`：

```go
package cmd

import (
	"fmt"
	"os"

	homedir "github.com/mitchellh/go-homedir"
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

var (
	cfgFile     string
	projectBase string
	userLicense string
)

func init() {
	cobra.OnInitialize(initConfig)
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra.yaml)")
	rootCmd.PersistentFlags().StringVarP(&projectBase, "projectbase", "b", "", "base project directory eg. github.com/spf13/")
	rootCmd.PersistentFlags().StringP("author", "a", "YOUR NAME", "Author name for copyright attribution")
	rootCmd.PersistentFlags().StringVarP(&userLicense, "license", "l", "", "Name of license for the project (can provide `licensetext` in config)")
	rootCmd.PersistentFlags().Bool("viper", true, "Use Viper for configuration")
	viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
	viper.BindPFlag("projectbase", rootCmd.PersistentFlags().Lookup("projectbase"))
	viper.BindPFlag("useViper", rootCmd.PersistentFlags().Lookup("viper"))
	viper.SetDefault("author", "NAME HERE <EMAIL ADDRESS>")
	viper.SetDefault("license", "apache")
}

func initConfig() {
	// Don't forget to read config either from cfgFile or from home directory!
	if cfgFile != "" {
		// Use config file from the flag.
		viper.SetConfigFile(cfgFile)
	} else {
		// Find home directory.
		home, err := homedir.Dir()
		if err != nil {
			fmt.Println(err)
			os.Exit(1)
		}

		// Search config in home directory with name ".cobra" (without extension).
		viper.AddConfigPath(home)
		viper.SetConfigName(".cobra")
	}

	if err := viper.ReadInConfig(); err != nil {
		fmt.Println("Can't read config:", err)
		os.Exit(1)
	}
}
```

2) 创建 `main.go`

我们还需要一个 main 函数来 调用 rootCmd，通常我们会创建一个 `main.go` 文件，在 `main.go` 中调用 `rootCmd.Execute()` 来执行命令：

```go
package main

import (
  "{pathToYourApp}/cmd"
)

func main() {
  cmd.Execute()
}
```

需要注意，`main.go` 中不建议放很多代码，通常只需要调用 `cmd.Execute()` 即可。

3) 添加命令

除了 `rootCmd`，我们还可以调用 `AddCommand` 添加其它命令，通常情况下，我们会把其它命令的源码文件放在 `cmd/` 目录下，例如，我们添加一个 `version` 命令，可以创建 `cmd/version.go` 文件，内容为：

```go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

func init() {
	rootCmd.AddCommand(versionCmd)
}

var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "Print the version number of Hugo",
	Long:  `All software has versions. This is Hugo's`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("Hugo Static Site Generator v0.9 -- HEAD")
	},
}
```

本示例中，我们通过调用 `rootCmd.AddCommand(versionCmd)` 给 `rootCmd` 命令添加了一个 `versionCmd` 命令。

4) 编译并运行

替换 `main.go` 中 `{pathToYourApp}` 为对应的路径，例如本示例中 `pathToYourApp` 为 `github.com/nosbelm/miniblogdemo/cobrademolib`。

```plain
$ go mod init
$ go work use .
$ go mod tidy
$ go build -v .
$ ./cobrademolib -h
```

通过步骤 1、2、3 我们就成功创建和添加了 cobra 应用程序和其命令。

### 使用标志

cobra 可以跟 pflag 结合使用，实现强大的标志功能。使用步骤如下：

1) 使用持久化的标志

标志可以是“持久的”，这意味着该标志可用于它所分配的命令以及该命令下的每个子命令。可以在 `rootCmd` 上定义持久标志：

```go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```

2) 使用本地标志

也可以分配一个本地标志，本地标志只能在其所绑定的命令上使用：

```go
rootCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

`--source` 标志只能在 `rootCmd` 上引用，而不能在 `rootCmd` 的子命令上引用。

3) 将标志绑定到 viper

我们可以将标志绑定到 viper，这样就可以使用 `viper.Get()` 获取标志的值。

```go
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```

4) 设置标志为必选

默认情况下，标志是可选的，我们也可以设置标志位必选，当设置标志位必选，但是没有提供标志时，cobra 会报错。

```go
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region")
```

5) 非选项参数验证

在使用命令的过程中，经常会传入非选项参数，并且需要对这些非选项参数进行验证，cobra 提供了机制来对非选项参数进行验证。可以使用 `Command` 的 `Args` 字段来验证非选项参数。cobra 也内置了一些验证函数，具体见下表：

|函数|描述|
|:----|:----|
|NoArgs|如果存在任何非选项参数，该命令将报错|
|ArbitraryArgs|该命令将接受任何非选项参数|
|OnlyValidArgs|如果有任何非选项参数不在 `Command` 的 `ValidArgs` 字段中，该命令将报错|
|MinimumNArgs(int)|如果没有至少 N 个非选项参数，该命令将报错|
|MaximumNArgs(int)|如果有多于 N 个非选项参数，该命令将报错|
|ExactArgs(int)|如果非选项参数个数不为 N，该命令将报错|
|ExactValidArgs(int)|如果非选项参数的个数不为 N，或者非选项参数不在 `Command` 的 `ValidArgs` 字段中，该命令将报错|
|RangeArgs(min, max)|如果非选项参数的个数不在 `min` 和 `max` 之间，该命令将报错|


使用自定义验证函数，示例如下：

```go
var cmd = &cobra.Command{
  Short: "hello",
  Args: cobra.MinimumNArgs(1), // 使用内置的验证函数
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```

也可以自定义验证函数，示例如下：

```go
var cmd = &cobra.Command{
  Short: "hello",
  // Args: cobra.MinimumNArgs(10), // 使用内置的验证函数
  Args: func(cmd *cobra.Command, args []string) error { // 自定义验证函数
    if len(args) < 1 {
      return errors.New("requires at least one arg")
    }
    if myapp.IsValidColor(args[0]) {
      return nil
    }
    return fmt.Errorf("invalid color specified: %s", args[0])
  },
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```

### Help命令

在使用应用程序时，我们需要知道该应用程序的调用方法，所以需要有一个 Help 命令或者选项参数，Cobra 的强大之处也在于所有我们需要的功能 cobra 都已经帮我们实现好了。在用 cobra 构建应用程序时，cobra 会自动为应用程序添加一个帮助命令，当用户运行 `app help` 时会调用此方法。此外，当 `help` 的输入为其它命令时，会打印该命令的用法，比如有一个叫做 `create` 的命令，当调用 `app help create` 时，会打印 `create` 的帮助信息。cobra 也会给每个命令自动添加 `--help` 标志。例如：

```bash
$ cobra-cli help
Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra-cli [command]

Available Commands:
  add         Add a command to a Cobra Application
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra-cli
  -l, --license string   name of license for the project
      --viper            use Viper for configuration

Use "cobra-cli [command] --help" for more information about a command.
```

我们也可以定义自己的 `help` 命令。使用如下函数，可以定义 `help` 命令：

```go
cmd.SetHelpCommand(cmd *Command)
cmd.SetHelpFunc(f func(*Command, []string))
cmd.SetHelpTemplate(s string)
```

### 使用信息

当用户提供无效标志或无效命令时，cobra 会打印出 usage 信息。例如：

```bash
$ cobra-cli --invalid
Error: unknown flag: --invalid
Usage:
  cobra-cli [command]

Available Commands:
  add         Add a command to a Cobra Application
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra-cli
  -l, --license string   name of license for the project
      --viper            use Viper for configuration

Use "cobra-cli [command] --help" for more information about a command.

```

像 `help` 一样，我们也可以自定义 usage，通过如下看函数可以自定义 usage：

```go
cmd.SetUsageFunc(f func(*Command) error)
cmd.SetUsageTemplate(s string)
```

### version 标志

如果在 `rootCmd` 命令上设置了 `Version` 字段，Cobra 会添加持久的 `--version` 标志。运行应用程序时，指定了 `--version` 标志，应用程序会使用 `Version` 模板将版本打印到 stdout。可以使用 `cmd.SetVersionTemplate(s string)` 函数自定义 `Version` 模板。

### PreRun and PostRun Hooks

在运行 `Run` 函数时我们可以运行一些钩子函数，比如 `PersistentPreRun` 和 `PreRun` 函数在 `Run` 函数之前执行。`PersistentPostRun` 和 `PostRun` 在 `Run` 函数之后执行。如果子命令没有指定 `Persistent*Run` 函数，则子命令将会继承父命令的 `Persistent*Run` 函数。这些函数的运行顺序如下：

1. `PersistentPreRun`
2. `PreRun`
3. `Run`
4. `PostRun`
5. `PersistentPostRun`

注意父级的 `PreRun` 只会在父级命令运行时调用，子命令时不会调用的。

下面是使用所有这些函数的两个命令的示例。执行子命令时，它将运行 `rootCmd` 命令的 `PersistentPreRun`，但不运行 `rootCmd` 命令的 `PersistentPostRun`：

```go
package main

import (
  "fmt"

  "github.com/spf13/cobra"
)

func main() {

  var rootCmd = &cobra.Command{
    Use:   "root [sub]",
    Short: "My root command",
    PersistentPreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPreRun with args: %v\n", args)
    },
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPostRun with args: %v\n", args)
    },
  }

  var subCmd = &cobra.Command{
    Use:   "sub [no options!]",
    Short: "My subcommand",
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PersistentPostRun with args: %v\n", args)
    },
  }

  rootCmd.AddCommand(subCmd)

  rootCmd.SetArgs([]string{""})
  rootCmd.Execute()
  fmt.Println()
  rootCmd.SetArgs([]string{"sub", "arg1", "arg2"})
  rootCmd.Execute()
}
```
执行后，输出如下：

```bash
Inside rootCmd PersistentPreRun with args: []
Inside rootCmd PreRun with args: []
Inside rootCmd Run with args: []
Inside rootCmd PostRun with args: []
Inside rootCmd PersistentPostRun with args: []

Inside rootCmd PersistentPreRun with args: [arg1 arg2]
Inside subCmd PreRun with args: [arg1 arg2]
Inside subCmd Run with args: [arg1 arg2]
Inside subCmd PostRun with args: [arg1 arg2]
Inside subCmd PersistentPostRun with args: [arg1 arg2]
```

### 命令建议

Cobra 还有很多其它有用的特性，比如当我们输入的命令有误时，cobra 会根据注册的命令，推算出可能的命令。例如，当 `unknown command` 错误发生时，Cobra 将自动打印建议的命令:

```bash
$ hugo srever
Error: unknown command "srever" for "hugo"

Did you mean this?
        server

Run 'hugo --help' for usage.
```

根据注册的每个子命令自动建议并使用 Levenshtein distance 实现。每个匹配最小距离为 2（忽略大小写）的注册命令将显示为建议。如果需要在命令中禁用建议或调整字符串距离，可以使用：

```go
command.DisableSuggestions = true
```

或者：

```go
command.SuggestionsMinimumDistance = 1
```

需要注意，Levenshtein distance（编辑距离）是针对二个字符串（例如英文字）的差异程度的量化量测，量测方式是看至少需要多少次的处理才能将一个字符串变成另一个字符串。

还可以使用 `SuggestFor` 属性显式设置要为其指定命令的名称，例如：

```go
// configCmd represents the config command
var configCmd = &cobra.Command{
    Use:   "config",
    Short: "A brief description of your command",
    Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
    SuggestFor: []string{"cfg", "conf"},
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("config called")
    },
}
```

执行 `newApp cfg`：

```bash
$ ./newApp cfg
Error: unknown command "cfg" for "newApp"

Did you mean this?
	config

Run 'newApp --help' for usage.
unknown command "cfg" for "newApp"

Did you mean this?
	config
```
