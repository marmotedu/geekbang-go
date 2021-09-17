## Delve使用详解

[Delve](https://github.com/go-delve/delve)是一个简单、功能齐全的Go代码调试工具，使用方式与GDB比较类似，也是目前最受欢迎的Go代码调试工具。

### 安装delve工具

要使用 delve 调试工具，首先需要安装 delve ，安装方法如下：

```bash
$ go install github.com/go-delve/delve/cmd/dlv@latest
```

delve通过`dlv`命令行工具来进行调试。安装后执行 `dlv version` ，如果能够正确输出版本号，说明安装成功：
```bash
$ dlv version
Delve Debugger
Version: 1.7.1
Build: $Id: 3bde2354aafb5a4043fd59838842c4cd4a8b6f0b $
```

安装完成后，你可以执行`dlv --help`来查看`dlv`支持的命令和功能：

```bash
$ dlv --help
...

Usage:
  dlv [command]

Available Commands:
  attach      连接到正在运行的流程并开始调试.
  connect     连接到无头调试服务器.
  core        检查核心转储.
  dap         通过DAP协议，启动无头 TCP 服务器通信.
  debug       编译并开始调试当前目录下的主包或指定的包.
  exec        执行预编译的二进制文件，并开始调试会话.
  help        帮助信息
  run         弃用的命令。使用'debug'替代它.
  test        编译测试二进制文件并开始调试程序.
  trace       编译并开始跟踪程序.
  version     打印版本信息.

Flags:
      ...

Use "dlv [command] --help" for more information about a command.
```

### 进入调试模式

使用`dlv`调试程序，首先需要进入到调试模式中。`dlv`支持四种方式进入调试模式：

1) dlv attach

`dlv attach`可以用来调试一个正在运行的程序。执行以下命令进入调试模式：

```bash
$ dlv attach $PID  # PID 是需要调试程序的进程 ID
```

2) dlv debug

`dlv debug`可以通过源码文件进入调试模式。`dlv debug`会先编译Go源文件，并在当前目录下生成一个名为`__debug_bin`的可执行二进制文件，接着进入调试模式。执行以下命令进入调试模式：

```bash
$ dlv debug main.go
Type 'help' for list of commands.
(dlv)
```

退出调试模式时，`__debug_bin`文件会被自动被删除。`dlv debug`是最简单的模式，常见于调试小型程序。

3) dlv exec


`dlv exec`可以通过一个二进制文件进入调试模式。执行以下命令进入调试模式：

```bash
$ dlv exec /opt/iam/bin/iam-apiserver  ## 直接启动调试
$ dlv exec /opt/iam/bin/iam-apiserver -- --config=/etc/iam/iam-apiserver.yaml # `--`后的内容为`/opt/iam/bin/iam-apiserver`的命令行参数
```

如果希望二进制文件被调试，在编译二进制文件时需要关闭内联优化：

```bash
$ go build -gcflags=all="-N -l"
```

如果不希望二进制文件被调试，则可以使用以下编译选项：

```bash
$ go build -ldflags "-s -w" # -s: 去掉符号信息；-w: 去掉 DWARF 调试信息。
```

4) dlv core

`dlv core`使用core文件启动调试，这种方式可以找出可执行文件core的原因，具体执行命令如下：
`
```bash
$ dlv core <executable-file> <core-file>
```

### 调试命令

进入调试模式之后，我们就可以利用调试模式中提供的丰富的调试命令来调试程序。`dlv`提供了以下调试命令：

```bash
(dlv) help
The following commands are available:

Running the program:
    call ------------------------ 恢复进程，注入一个函数调用(还在实验阶段!!)
    continue (alias: c) --------- 运行到断点或程序终止.
    next (alias: n) ------------- 转到下一个源码行.
    rebuild --------------------- 重建目标可执行文件并重新启动它。 如果可执行文件不是由 delve 构建的，则不起作用.
    restart (alias: r) ---------- 重启进程.
    step (alias: s) ------------- 单步执行程序.
    step-instruction (alias: si)  单步执行一条cpu指令.
    stepout (alias: so) --------- 跳出当前函数.

Manipulating breakpoints:
    break (alias: b) ------- 设置断点.
    breakpoints (alias: bp)  输出活动断点的信息.
    clear ------------------ 删除断点.
    clearall --------------- 删除多个断点.
    condition (alias: cond)  设置断点条件.
    on --------------------- 在命中断点时执行命令.
    toggle ----------------- 打开或关闭断点.
    trace (alias: t) ------- 设置跟踪点.
    watch ------------------ 设置观察点.

Viewing program variables and memory:
    args ----------------- 打印函数参数.
    display -------------- 每次程序停止时打印表达式的值.
    examinemem (alias: x)  检查给定地址处的原始内存.
    locals --------------- 打印局部变量.
    print (alias: p) ----- 计算一个表达式.
    regs ----------------- 打印CPU寄存器的内容.
    set ------------------ 更改变量的值.
    vars ----------------- 打印包变量.
    whatis --------------- 打印表达式的类型.

Listing and switching between threads and goroutines:
    goroutine (alias: gr) -- 显示或更改当前goroutine
    goroutines (alias: grs)  列举程序goroutines.
    thread (alias: tr) ----- 切换到指定的线程.
    threads ---------------- 打印每个跟踪线程的信息.

Viewing the call stack and selecting frames:
    deferred --------- 在延迟调用的上下文中执行命令.
    down ------------- 将当前帧向下移动.
    frame ------------ 设置当前帧，或在不同的帧上执行命令.
    stack (alias: bt)  打印堆栈跟踪信息.
    up --------------- 向上移动当前帧.

Other commands:
    config --------------------- 修改配置参数.
    disassemble (alias: disass)  反汇编程序.
    dump ----------------------- 从当前进程状态创建核心转储
    edit (alias: ed) ----------- 在$DELVE_EDITOR或$EDITOR中打开你所在的位置
    exit (alias: quit | q) ----- 退出调试器.
    funcs ---------------------- 打印函数列表.
    help (alias: h) ------------ 打印帮助信息.
    libraries ------------------ 列出加载的动态库
    list (alias: ls | l) ------- 显示源代码.
    source --------------------- 执行包含delve命令列表的文件
    sources -------------------- 打印源文件列表.
    types ---------------------- 打印类型列表

Type help followed by a command for full documentation.
```

`dlv`支持的调试命令有很多，常用的调试命令有：break、print、next、continue、args、locals、list、breakpoints、restart、exit。


### 使用delve调试代码

假设我们启动`/opt/iam/bin/iam-apiserver -- --config=/etc/iam/iam-apiserver.yaml`报以下错误：

```bash
...
2021-09-17 20:23:26.475	INFO	apiserver	server/genericapiserver.go:88	GET    /debug/pprof/threadcreate --> github.com/gin-contrib/pprof.pprofHandler.func1 (5 handlers)
2021-09-17 20:23:26.475	INFO	apiserver	server/genericapiserver.go:88	GET    /version --> github.com/marmotedu/iam/internal/pkg/server.(*GenericAPIServer).InstallAPIs.func2 (5 handlers)
2021-09-17 20:23:26.481	INFO	apiserver	gorm@v1.21.12/gorm.go:202	mysql/mysql.go:75[error] failed to initialize database,got error commands out of sync. Did you run multiple statements at once?
2021-09-17 20:23:26.481	FATAL	apiserver	apiserver/server.go:139	Failed to get cache instance: got nil cache server
github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:139
github.com/marmotedu/iam/internal/apiserver.createAPIServer
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:66
github.com/marmotedu/iam/internal/apiserver.Run
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/run.go:11
github.com/marmotedu/iam/internal/apiserver.run.func1
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/app.go:46
github.com/marmotedu/iam/pkg/app.(*App).runCommand
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/pkg/app/app.go:278
github.com/spf13/cobra.(*Command).execute
	/data/mod/github.com/spf13/cobra@v1.2.1/command.go:856
github.com/spf13/cobra.(*Command).ExecuteC
	/data/mod/github.com/spf13/cobra@v1.2.1/command.go:974
github.com/spf13/cobra.(*Command).Execute
	/data/mod/github.com/spf13/cobra@v1.2.1/command.go:902
github.com/marmotedu/iam/pkg/app.(*App).Run
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/pkg/app/app.go:233
main.main
	/home/colin/workspace/golang/src/github.com/marmotedu/iam/cmd/iam-apiserver/apiserver.go:24
runtime.main
	/home/colin/go/go1.17/src/runtime/proc.go:255
```

现在需要通过`dlv`工具找到故障原因。具体调试步骤如下：

1) 进入调试模式

这里选择使用`dlv exec`的方式进入到调试模式：

```bash
$ dlv exec /opt/iam/bin/iam-apiserver -- --config=/etc/iam/iam-apiserver.yaml
Type 'help' for list of commands.
(dlv)
```

2) 在关键位置打断点

我们在报错栈中最靠近报错信息的位置打上断点：`/home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:139`。

```bash
(dlv) b /home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:139
Breakpoint 1 set at 0xd24de2 for github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:139
```

3) 启动程序

执行`dlv exec`命令后，其实程序还没有真正运行，接着需要你通过`restart`命令运行程序：

```bash
$ dlv exec /opt/iam/bin/iam-apiserver -- --config=/etc/iam/iam-apiserver.yaml
Type 'help' for list of commands.
(dlv) b /home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:139
Breakpoint 1 set at 0xd24de2 for github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:139
(dlv) r
Process restarted with PID 30155
```

4) 继续执行到断点处

程序启动后，需要你执行`continue`运行到断点处：

```go
(dlv) c
...
2021-09-17 20:31:12.815	INFO	apiserver	gorm@v1.21.12/gorm.go:202	mysql/mysql.go:75[error] failed to initialize database,got error commands out of sync. Did you run multiple statements at once?
> github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:139 (hits goroutine(1):1 total:1) (PC: 0xd24de2)
Warning: debugging optimized function
   134:		storeIns, _ := mysql.GetMySQLFactoryOr(c.mysqlOptions)
   135:		// storeIns, _ := etcd.GetEtcdFactoryOr(c.etcdOptions, nil)
   136:		store.SetClient(storeIns)
   137:		cacheIns, err := cachev1.GetCacheInsOr(storeIns)
   138:		if err != nil {
=> 139:			log.Fatalf("Failed to get cache instance: %s", err.Error())
   140:		}
   141:
   142:		pb.RegisterCacheServer(grpcServer, cacheIns)
   143:
   144:		reflection.Register(grpcServer)
(dlv)
```

通过第`139`行的代码，我们发现日志报错是因为`cachev1.GetCacheInsOr`返回了错误信息。我们可以在`cachev1.GetCacheInsOr`处打断点，继续排障。

5) 重新打断点继续排障

在`cachev1.GetCacheInsOr`处打断点，重新执行程序，并进入到`cachev1.GetCacheInsOr`函数中，查找报错的原因：

```go
(dlv) b /home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:137
Breakpoint 2 set at 0xd24dd3 for github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:137
(dlv) r
Process restarted with PID 5700
(dlv) c
...
> github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:137 (hits goroutine(1):1 total:1) (PC: 0xd24dd3)
Warning: debugging optimized function
   132:		grpcServer := grpc.NewServer(opts...)
   133:
   134:		storeIns, _ := mysql.GetMySQLFactoryOr(c.mysqlOptions)
   135:		// storeIns, _ := etcd.GetEtcdFactoryOr(c.etcdOptions, nil)
   136:		store.SetClient(storeIns)
=> 137:		cacheIns, err := cachev1.GetCacheInsOr(storeIns)
   138:		if err != nil {
   139:			log.Fatalf("Failed to get cache instance: %s", err.Error())
   140:		}
   141:
   142:		pb.RegisterCacheServer(grpcServer, cacheIns)
(dlv) s
> github.com/marmotedu/iam/internal/apiserver/controller/v1/cache.GetCacheInsOr() ./internal/apiserver/controller/v1/cache/cache.go:34 (PC: 0xd20982)
Warning: debugging optimized function
    29:		once        sync.Once
    30:	)
    31:
    32:	// GetCacheInsOr return cache server instance with given factory.
    33:	func GetCacheInsOr(store store.Factory) (*Cache, error) {
=>  34:		if store != nil {
    35:			once.Do(func() {
    36:				cacheServer = &Cache{store}
    37:			})
    38:		}
    39:
(dlv) s
> github.com/marmotedu/iam/internal/apiserver/controller/v1/cache.GetCacheInsOr() ./internal/apiserver/controller/v1/cache/cache.go:40 (PC: 0xd209c7)
Warning: debugging optimized function
    35:			once.Do(func() {
    36:				cacheServer = &Cache{store}
    37:			})
    38:		}
    39:
=>  40:		if cacheServer == nil {
    41:			return nil, fmt.Errorf("got nil cache server")
    42:		}
    43:
    44:		return cacheServer, nil
    45:	}
```

打上断点后，可以重新启动程序，并运行到断点处。在断点处执行`step`可以进入到`cachev1.GetCacheInsOr`函数中。

通过执行`cachev1.GetCacheInsOr`中的代码，我们发现报错原因是因为`cacheServer == nil`，`cacheServer == nil`又是因为`store == nil`导致`cacheServer`没有被初始化。store是由`GetMySQLFactoryOr`函数返回的，所以我们继续打断点，继续排障。

6) 重新打断点继续排障

在`GetMySQLFactoryOr`所在的位置`.../internal/apiserver/server.go:134`处打断点，重新执行程序，继续排障：

```
(dlv) b /home/colin/workspace/golang/src/github.com/marmotedu/iam/internal/apiserver/server.go:134
Breakpoint 3 set at 0xd24d96 for github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:134
(dlv) r
Process restarted with PID 9472
(dlv) c
...
> github.com/marmotedu/iam/internal/apiserver.(*completedExtraConfig).New() ./internal/apiserver/server.go:134 (hits goroutine(1):1 total:1) (PC: 0xd24d96)
Warning: debugging optimized function
   129:			log.Fatalf("Failed to generate credentials %s", err.Error())
   130:		}
   131:		opts := []grpc.ServerOption{grpc.MaxRecvMsgSize(c.MaxMsgSize), grpc.Creds(creds)}
   132:		grpcServer := grpc.NewServer(opts...)
   133:
=> 134:		storeIns, _ := mysql.GetMySQLFactoryOr(c.mysqlOptions)
   135:		// storeIns, _ := etcd.GetEtcdFactoryOr(c.etcdOptions, nil)
   136:		store.SetClient(storeIns)
   137:		cacheIns, err := cachev1.GetCacheInsOr(storeIns)
   138:		if err != nil {
   139:			log.Fatalf("Failed to get cache instance: %s", err.Error())
(dlv) s
> github.com/marmotedu/iam/internal/apiserver/store/mysql.GetMySQLFactoryOr() ./internal/apiserver/store/mysql/mysql.go:56 (PC: 0xd16aaf)
Warning: debugging optimized function
    51:		mysqlFactory store.Factory
    52:		once         sync.Once
    53:	)
    54:
    55:	// GetMySQLFactoryOr create mysql factory with the given config.
=>  56:	func GetMySQLFactoryOr(opts *genericoptions.MySQLOptions) (store.Factory, error) {
    57:		if opts == nil && mysqlFactory == nil {
    58:			return nil, fmt.Errorf("failed to get mysql store fatory")
    59:		}
    60:
    61:		var err error
(dlv) print opts
*github.com/marmotedu/iam/internal/pkg/options.MySQLOptions {
	Host: "127.0.0.1:3309",
	Username: "iam",
	Password: "iam59!z$",
	Database: "iam",
	MaxIdleConnections: 100,
	MaxOpenConnections: 100,
	MaxConnectionLifeTime: golang.org/x/net/http2.prefaceTimeout (10000000000),
	LogLevel: 4,}
```

上面的调试步骤中，我们打印了`options.MySQLOptions`类型的变量`opts`的值：

```go
 {
	Host: "127.0.0.1:3309",
	Username: "iam",
	Password: "iam59!z$",
	Database: "iam",
	MaxIdleConnections: 100,
	MaxOpenConnections: 100,
	MaxConnectionLifeTime: golang.org/x/net/http2.prefaceTimeout (10000000000),
	LogLevel: 4,}
```

很明显，Host的地址是错误的，应该是`"127.0.0.1:3306"`。所以，这里我们就找到了程序报错的根本原因：MySQL的端口配置错了。

找到问题的根因之后，就可以修改`/etc/iam/iam-apiserver.yaml`配置文件中的`mysql.host`配置项的值为：`127.0.0.1:3306`。重新启动`iam-apiserver`，发现服务成功运行。


delve是一款非常强大的调试工具，本文只用了其中一小部分功能，更多的delve功能需要你解锁。在你能够熟练使用delve工具之后，相信delve能够极大的提高你的排障效率。
