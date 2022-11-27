## 配置解析神器：Viper 全解

几乎所有的后端服务都需要一些配置项来配置我们的服务，一些小型的项目，配置不是很多，可以选择只通过命令行参数来传递配置，但是大型项目配置很多，通过命令行参数传递就变的很麻烦，不好维护，标准的解决方案是将这些配置信息保存在配置文件中，由程序启动时加载和解析。所以对于一个稍微大型点的系统，配置文件加载和解析就是一个刚需。Go 生态中有很多包可以加载并解析配置，目前最受欢迎的是 Viper 包。

[Viper](https://github.com/spf13/viper) 是 Go 应用程序现代化、完整的解决方案，能够处理不同格式的配置文件，让我们在构建现代应用程序时，不必担心配置文件格式。Viper 也能够满足我们对应用配置的各种需求。Viper有 很多特性，其中一些比较重要的特性如下：

* 支持默认配置。
* 支持从 `json`、`toml`、`yaml`、`yml`、`properties`、`props`、`prop`、`hcl`、`dotenv`、`env` 格式的文件中读取数据。
* 实时监控和重新读取配置文件（可选）。
* 支持从环境变量中读取配置。
* 支持从远程配置系统（etcd 或 Consul）读取并监控配置变化。
* 从命令行参数读取配置。
* 支持从 buffer 中读取配置。
* 可以显式的给配置项设置值。

Viper 可以从不同的位置读取配置，不同位置的配置具有不同的优先级，高优先的配置会覆盖低优先级相同的配置，按优先级从高到低排列如下：

1. 通过 `viper.Set` 函数显试设置的配置
2. 命令行参数
3. 环境变量
4. 配置文件
5. key/value 存储
6. 默认值

Viper 因为其强大的特性，越来越多的优秀项目开始使用 Viper 作为其配置解析工具，例如：Hugo、Docker Notary、Clairctl、Mercure 等。

这里需要注意，Viper 配置键不区分大小写。

### Viper 使用方法

Viper 有很多功能，这里选择一些常用的、重要的功能来讲解。在使用 viper 的过程中，最重要的2类功能就是读入配置和读取配置，Viper 提供不同的方式来读入配置和读取配置。

读入配置，将配置读入到 viper 中，有如下读入方式：

* 可以设置默认的配置文件名。
* 读取配置文件。
* 监听和重新读取配置文件。
* 从 `io.Reader` 读取配置。
* 从环境变量读取。
* 从命令行标志读取。
* 从远程 Key/Value 存储读取。
读取配置，从 viper 中读取配置到应用程序中，viper 提供如下函数，来读取配置：

* `Get(key string) interface{}`
* `Get<Type>(key string) <Type>`
* `AllSettings() map[string]interface{}`

### 读入配置

1) 设置默认值

一个好的配置系统应该支持默认值。viper 支持对 key 设置默认值，当没有通过配置文件，环境变量，远程配置或命令行标志设置 key 时，设置默认值通常是很有用的，可以使程序即使在没有明确指定配置时，也能够正常运行。例如：

```go
viper.SetDefault("ContentDir", "content")
viper.SetDefault("LayoutDir", "layouts")
viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
```

2) 读取配置文件

viper 可以读取配置文件来解析配置，支持 `json`、`toml`、`yaml`、`yml`、`properties`、`props`、`prop`、`hcl`、`dotenv`、`env` 格式的配置文件。Viper 可以搜索多个路径，但目前单个 Viper 实例仅支持单个配置文件。Viper 不默认任何配置搜索路径，将默认决策留给应用程序。

以下是如何使用 Viper 搜索和读取配置文件的示例：

```go
package main

import (
	"fmt"

	"github.com/spf13/pflag"
	"github.com/spf13/viper"
)

var (
	cfg  = pflag.StringP("config", "c", "", "Configuration file.")
	help = pflag.BoolP("help", "h", false, "Show this help message.")
)

func main() {
	pflag.Parse()
	if *help {
		pflag.Usage()
		return
	}

  // 从配置文件中读取配置
	if *cfg != "" {
		viper.SetConfigFile(*cfg)   // 指定配置文件名
		viper.SetConfigType("yaml") // 如果配置文件名中没有文件扩展名，则需要指定配置文件的格式，告诉 viper 以何种格式解析文件
	} else {
		viper.AddConfigPath(".")          // 把当前目录加入到配置文件的搜索路径中
		viper.AddConfigPath("$HOME/.iam") // 配置文件搜索路径，可以设置多个配置文件搜索路径
		viper.SetConfigName("config")     // 配置文件名称（没有文件扩展名）
	}

	if err := viper.ReadInConfig(); err != nil { // 读取配置文件。如果指定了配置文件名，则使用指定的配置文件，否则在注册的搜索路径中搜索
		panic(fmt.Errorf("Fatal error config file: %s \n", err))
	}

	fmt.Printf("Used configuration file is: %s\n", viper.ConfigFileUsed())
}
```

在加载配置文件出错时，你可以像下面这样处理找不到配置文件的特定情况：

```go
if err := viper.ReadInConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        // 配置文件未找到错误；如果需要可以忽略
    } else {
        // 配置文件被找到，但产生了另外的错误
    }
}

// 配置文件找到并成功解析
```

viper 支持设置多个配置文件搜索路径，需要注意添加搜索路径的顺序，viper 会根据添加的路径顺序搜索配置文件，如果找到则停止搜索。如果调用 `SetConfigFile` 直接指定了配置文件名，并且配置文件名没有文件扩展名时，需要显试指定配置文件的格式，以使 viper 能够正确解析配置文件。

如果通过搜索的方式查找配置文件，则需要注意 `SetConfigName` 设置的配置文件名是不带扩展名的，在搜索时 viper 会在文件名之后追加文件扩展名，并尝试搜索所有支持的扩展类型。比如，如果我们通过 `SetConfigName` 设置了配置文件名为 `config`，则 viper 会在注册的搜索路径中，依次搜索：`config.json`、`config.toml`、`config.yaml`、`config.yml`、`config.properties`、`config.props`、`config.prop`、`config.hcl`、`config.dotenv`、`config.env`。

3) 写入配置文件

读取配置文件很有用，但有时候我们可能需要将程序中当前的配置保存起来，方便后续使用或者 debug，viper 提供了一系列的函数，可以让我们把当前的配置保存到文件中，viper 提供了如下函数来保存配置：

* `WriteConfig`：保存当前的配置到 viper 当前使用的配置文件中，如果配置文件不存在会报错，如果配置文件存在则覆盖当前的配置文件。
* `SafeWriteConfig`：保存当前的配置到 viper 当前使用的配置文件中，如果配置文件不存在会报错，如果配置文件存在则返回 `file exists` 错误。
* `WriteConfigAs`：保存当前的配置到指定的文件中，如果文件不存在则新建，如果文件存在则会覆盖文件。
* `SafeWriteConfigAs`：保存当前的配置到指定的文件中，如果文件不存在则新建，如果文件存在则返回 `file exists` 错误。

根据经验，标记为 `Safe` 的所有方法都不会覆盖任何文件，而是直接创建（如果不存在），而默认行为是创建或截断。

一个小示例：

```go
viper.WriteConfig()
viper.SafeWriteConfig()
viper.WriteConfigAs("config.running.yaml")
viper.SafeWriteConfigAs("config.running.yaml")
```

4) 监听和重新读取配置文件

Viper 支持在运行时让应用程序实时读取配置文件，也就是热加载配置。可以通过 `WatchConfig` 函数热加载配置。在调用 `WatchConfig` 函数之前，请确保已经添加了配置文件的搜索路径。可选地，可以为 Viper 提供一个回调函数，以便在每次发生更改时运行。

示例：

```go
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
   // 配置文件发生变更之后会调用的回调函数
	fmt.Println("Config file changed:", e.Name)
})
```

5) 设置配置值

我们可以通过 `viper.Set()` 函数来显试设置配置：

```go
viper.Set("user.username", "colin")
```

6) 注册和使用别名

别名允许多个键引用单个值。示例：

```go
viper.RegisterAlias("loud", "Verbose")
viper.Set("verbose", true) // 效果等同于下面一行代码
viper.Set("loud", true)   // 效果等同于上面一行代码

viper.GetBool("loud") // true
viper.GetBool("verbose") // true
```

7) 使用环境变量

viper 还支持环境变量，通过如下 5 个函数来支持环境变量：

* `AutomaticEnv()`
* `BindEnv(input ...string) error`
* `SetEnvPrefix(in string)`
* `SetEnvKeyReplacer(r *strings.Replacer)`
* `AllowEmptyEnv(allowEmptyEnv bool)`

这里要注意：viper 读取环境变量是区分大小写的。viper 提供了一种机制来确保 `ENV` 变量是唯一的。通过使用 `SetEnvPrefix`，可以告诉 Viper 在读取环境变量时使用前缀。`BindEnv` 和 `AutomaticEnv` 都将使用此前缀。比如，我们设置了 `viper.SetEnvPrefix("VIPER")`，当使用 `viper.Get("apiversion")` 时，实际读取的环境变量是 `VIPER_APIVERSION`。

`BindEnv` 需要一个或两个参数。第一个参数是键名，第二个是环境变量的名称，环境变量的名称区分大小写。如果未提供 `ENV` 变量名，则viper将假定ENV变量名为： `环境变量前缀_键名全大写` ，例如：前缀为VIPER，key为username，则ENV变量名为： `VIPER_USERNAME` 。当显式提供 `ENV` 变量名（第二个参数）时，它不会自动添加前缀。例如，如果第二个参数是 `id`，Viper 将查找环境变量 `ID`。

在使用 `ENV` 变量时，需要注意的一件重要事情是，每次访问该值时都将读取它。Viper在调用 `BindEnv` 时不固定该值。

还有一个魔法函数 `SetEnvKeyReplacer`，`SetEnvKeyReplacer` 允许你使用 `strings.Replacer` 对象来重写 `Env` 键。如果你想在 `Get()` 调用中使用 `-` 或者 `.` ，但希望你的环境变量使用 `_` 分隔符，可以通过 `SetEnvKeyReplacer` 来实现。比如，我们设置了环境变量 `USER_SECRET_KEY=bVix2WBv0VPfrDrvlLWrhEdzjLpPCNYb`，但我们想用 `viper.Get("user.secret-key")`，我们调用函数：

```go
viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_", "-", "_"))
```

上面的代码，在调用 `viper.Get()` 函数时，会用 `_` 替换 `.` 和 `-` 。默认情况下，空环境变量被认为是未设置的，并将返回到下一个配置源。若要将空环境变量视为已设置，可以使用 `AllowEmptyEnv` 方法。使用环境变量示例如下：

```go
// 使用环境变量
os.Setenv("VIPER_USER_SECRET_ID", "QLdywI2MrmDVjSSv6e95weNRvmteRjfKAuNV")
os.Setenv("VIPER_USER_SECRET_KEY", "bVix2WBv0VPfrDrvlLWrhEdzjLpPCNYb")

viper.AutomaticEnv() // 读取环境变量
viper.SetEnvPrefix("VIPER") // 设置环境变量前缀：VIPER_，如果是 viper，将自动转变为大写。
viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_", "-", "_")) // 将 viper.Get(key) key 字符串中 '.' 和 '-' 替换为 '_'
viper.BindEnv("user.secret-key")
viper.BindEnv("user.secret-id", "USER_SECRET_ID") // 绑定环境变量名到 key
```

8) 使用标志

viper 支持 pflag 包，能够绑定 key 到 flag。与 `BindEnv` 类似，在调用绑定方法时，不会设置该值。但在访问它时会设置。对于单个标志，可以调用 `BindPFlag()` 进行绑定：

```go
viper.BindPFlag("token", pflag.Lookup("token")) // 绑定单个标志
```
还可以绑定一组现有的pflags（`pflag.FlagSet`）：

```go
viper.BindPFlags(pflag.CommandLine)             //绑定标志集
```

### 读取配置

viper 提供了如下方法来读取配置：

- `Get(key string) interface{}`

- `Get<Type>(key string) <Type>`

- `AllSettings() map[string]interface{}`

- `IsSet(key string) bool`

每一个 `Get` 方法在找不到值的时候都会返回零值。为了检查给定的键是否存在，可以使用 `IsSet()` 方法。`<Type>` 可以是 viper 支持的类型首字母大写：`Bool`、`Float64`、`Int`、`IntSlice`、`String`、`StringMap`、`StringMapString`、`StringSlice`、`Time`、`Duration`。例如：`GetInt()`。

读取配置具体使用方法如下：

1) 访问嵌套的键

例如：加载下面的 JSON 文件：

```json
{
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}
```

Viper可以通过传入 `.` 分隔的路径来访问嵌套字段：

```go
viper.GetString("datastore.metric.host") // (返回 "127.0.0.1")
```

如果 `datastore.metric` 被直接赋值覆盖（被 flag，环境变量，`set()` 方法等等），那么 `datastore.metric` 的所有子键都将变为未定义状态，它们被高优先级配置级别覆盖了。

如果存在与分隔的键路径匹配的键，则直接返回其值。例如：

```go
{
    "datastore.metric.host": "0.0.0.0",
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}
```

通过 `viper.GetString` 获取值：

```go
viper.GetString("datastore.metric.host") // 返回 "0.0.0.0"
```

2) 提取子树

例如：viper 加载了如下配置：

```yaml
app:
  cache1:
    max-items: 100
    item-size: 64
  cache2:
    max-items: 200
    item-size: 80
```

可以通过 `viper.Sub` 提取子树：

```go
subv := viper.Sub("app.cache1")
```

`subv` 现在就代表：

```bash
max-items: 100
item-size: 64
```

3) 反序列化

viper 可以支持将所有或特定的值解析到结构体、map 等。可以通过 2 个函数来实现：

* `Unmarshal(rawVal interface{}) error`
* `UnmarshalKey(key string, rawVal interface{}) error`

一个示例：

```go
type config struct {
	Port int
	Name string
	PathMap string `mapstructure:"path_map"`
}

var C config

err := viper.Unmarshal(&C)
if err != nil {
	t.Fatalf("unable to decode into struct, %v", err)
}
```

如果想要解析那些键本身就包含 `.` (默认的键分隔符）的配置，则需要修改分隔符：

```go
v := viper.NewWithOptions(viper.KeyDelimiter("::"))

v.SetDefault("chart::values", map[string]interface{}{
    "ingress": map[string]interface{}{
        "annotations": map[string]interface{}{
            "traefik.frontend.rule.type":                 "PathPrefix",
            "traefik.ingress.kubernetes.io/ssl-redirect": "true",
        },
    },
})

type config struct {
	Chart struct{
        Values map[string]interface{}
    }
}

var C config

v.Unmarshal(&C)
```

Viper 在后台使用 `github.com/mitchellh/mapstructure` 来解析值，其默认情况下使用 `mapstructure tags`。当我们需要将 viper 读取的配置反序列到我们定义的结构体变量中时，一定要使用 mapstructure tags。

4) 序列化成字符串

有时候我们需要将 viper 中保存的所有设置序列化到一个字符串中，而不是将它们写入到一个文件中，示例如下：

```go
import (
    yaml "gopkg.in/yaml.v2"
    // ...
)

func yamlStringSettings() string {
    c := viper.AllSettings()
    bs, err := yaml.Marshal(c)
    if err != nil {
        log.Fatalf("unable to marshal config to YAML: %v", err)
    }
    return string(bs)
}
```
