# Ladon Condition使用示例

生效条件（Condition）是一个Go函数，根据传入的上下文（context）在函数中实现自己的逻辑，最终返回true或false。如果返回true说明该条策略生效，如果返回false说明该条策略不生效。给策略添加一个condition，包括2个部分，一个键名和一个ladon.Condition的接口实现：

```go
// Condition either do or do not fulfill an access request.
type Condition interface {
    // GetName returns the condition's name.
    GetName() string

    // Fulfills returns true if the request is fulfilled by the condition.
    Fulfills(interface{}, *Request) bool
}
```

Condition接口实现了2个函数：GetName()返回Condition的名称，Fulfills()返回一个bool值，用来说明context是否满足condition，如果满足则策略生效，如果不满足则策略不生效。

Ladon内置了很多condition，如下是StringEqualCondition condition的实现：

```go
// StringEqualCondition is an exemplary condition.
type StringEqualCondition struct {
	Equals string `json:"equals"`
}

// Fulfills returns true if the given value is a string and is the
// same as in StringEqualCondition.Equals
func (c *StringEqualCondition) Fulfills(value interface{}, _ *ladon.Request) bool {
	s, ok := value.(string)

	return ok && s == c.Equals
}

// GetName returns the condition's name.
func (c *StringEqualCondition) GetName() string {
	return "StringEqualCondition"
}

// A policy with StringEqualCondition condition.
var pol = &ladon.DefaultPolicy{
	// ...
	Conditions: ladon.Conditions{
		"some-arbitrary-key": &StringEqualCondition{
			Equals: "the-value-should-be-this",
		},
	},
}
```

Fulfills()函数返回true，当给定的value是一个字符串，并且值等于the-value-should-be-this。我们在创建策略时，指定了一个key：some-arbitrary-key，其值为StringEqualCondition结构体类型的值，这样当ladon在进行授权时，会从context中获取key为some-arbitrary-key的值，然后在Fulfills()函数中，根据之前存储的StringEqualCondition内容和获取到的key的值，进行运算，最终返回true/false。

Policy的默认实现支持JSON un-/marshalling，该condition的JSON格式为：

```go
{
  "conditions": {
    "some-arbitrary-key": {
        "type": "StringEqualCondition",
        "options": {
            "equals": "the-value-should-be-this"
        }
    }
  }
}
```

返回参数意义为：
- type：StringEqualCondition.GetName()函数的返回值。
- options：options用来设置StringEqualCondition.Equals。

如果我们有策略的condition是上面的condition，当传入一下context时，Fullfills结果如下表所示：

| context                                                             | Fullfills结果 |
| ------------------------------------------------------------------- | ------------- |
| {"context":{"some-arbitrary-key":"the-value-should-be-this"}}       | true          |
| {"context":{"some-arbitrary-key":"some other value"}}               | false         |
| {"context":{"same value but other key":"the-value-should-be-this"}} | false         |


## 内置condition

Ladon内置了如下condition供我们直接使用：

### CIDRCondition

检查传入key值是否匹配condition所设定的CIDR，key值是一个remote ip。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
        "remoteIPAddress": &ladon.CIDRCondition{
            CIDR: "192.168.0.1/16",
        },
    },
}
```

### StringEqualCondition

检查传入的key值是否是字符串类型，并且等于condition所设定的值。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
        "some-arbitrary-key": &ladon.StringEqualCondition{
            Equals: "the-value-should-be-this",
        },
    },
}
```

context示例：

```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Context: ladon.Context{
         "some-arbitrary-key": "the-value-should-be-this",
    },
})
```

### BooleanCondition

检查传入的key值是否是bool类型，并且等于condition所设定的值。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
        "some-arbitrary-key": &ladon.BooleanCondition{
            BooleanValue: true,
        },
    },
}
```

context示例：

```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Context: ladon.Context{
        "some-arbitrary-key": true,
    },
})
```

当需要针对多个subject对资源进行动态的授权时，BooleanCondition就特别有用。例如：如果我们想授权某个用户只对其所拥有的资源有查看权限时，我们需要针对不同的人不同的资源都创建一个ladon策略。如果使用BooleanCondition condition我们就可以在运行时通过Fullfills()函数来判断策略是否生效。

### StringMatchCondition

检查传入的key值是否匹配condition指定的正则规则。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
      "some-arbitrary-key": &ladon.StringMatchCondition{
          Matches: "regex-pattern-here.+",
      },
  },
}
```

context示例：

```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Context: ladon.Context{
          "some-arbitrary-key": "regex-pattern-here111",
    },
})
```

### EqualsSubjectCondition

检查传入的key值是否匹配subject。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
        "some-arbitrary-key": &ladon.EqualsSubjectCondition{},
    },
}
```

context示例：

```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Subject: "peter",
    Context: ladon.Context{
         "some-arbitrary-key": "peter",
    },
})
```

### StringPairsEqualCondition

检查传入的key值是否是包含2个元素的数组，并且数组中的2个元素相等。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
        "some-arbitrary-key": &ladon.StringPairsEqualCondition{},
    },
}
```

context示例：

```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Context: ladon.Context{
         "some-arbitrary-key": [
             ["some-arbitrary-pair-value", "some-arbitrary-pair-value"],
             ["some-other-arbitrary-pair-value", "some-other-arbitrary-pair-value"],
         ],
    },
})
```

### ResourceContainsCondition

检查传入的key值是否出现在resource字符串中。condition Go格式为：

```go
var pol = &ladon.DefaultPolicy{
    Conditions: ladon.Conditions{
        "some-arbitrary-key": &ladon.ResourceContainsCondition{}
    },
}
```

context示例：

```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Resource: "rn:city:laholm:part:north"
    Context: ladon.Context{
      delimiter: ":",
      value: "part:north",
    },
})
```

Ladon还有很多其它的condition，更多的轻参考：https://github.com/ory/ladon。

**注意：**context需要指定delimiter来告诉ladon如何拆分resource。

## 自定义condition

我们还可以自定义condition，然后将自定义的condition添加到ladon.ConditionFactories中，例如：

```go
import "github.com/ory/ladon"

func main() {
    // ...

    ladon.ConditionFactories[new(CustomCondition).GetName()] = func() Condition {
        return new(CustomCondition)
    }

    // ...
}
```
自定义condition很简单，读者可以参考ory/ladon项目根目录下的condition_boolean.go文件，将BooleanCondition替换为CustomCondition就得到了一个简单的自定义condition。
