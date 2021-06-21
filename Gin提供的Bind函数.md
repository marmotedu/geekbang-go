# Gin提供的Bind函数

Gint提供了一些函数，可以绑定以下5种类别的HTTP参数：

- 路径参数。
- 查询字符串参数。
- 表单参数。
- HTTP头参数。
- 消息体参数。

## 1. 路径参数

1) 获取指定的路径参数参数
Param(key string) string：如果指定的参数存在则返回其值，不存在则返回空字符串`""`。

2) 绑定路径参数
- ShouldBindUri(obj interface{}) error：绑定路径参数到传入的结构体指针中，绑定出错时，返回错误内容。
- BindUri(obj interface{}) error：绑定路径参数到传入的结构体指针中，绑定出错时，会终止请求，并返回HTTP 400错误。

## 2. 查询字符串参数

1) 获取指定的查询字符串参数
- Query(key string) string：如果指定的参数key存在则返回其值，不存在则返回空字符串""。
- DefaultQuery(key, defaultValue string) string：如果指定的key存在则返回其值，不存在则返回指定的默认字符串defaultValue。
- GetQuery(key string) (string, bool)：同Query()，但是当key存在时，会额外返回一个true值，当key不存在时，会额外返回一个false值。
- QueryArray(key string) []string：返回指定key的slice，例如：GET /?name=name1&name=name2，则返回：["name1",:"name2"]，如果key不存在，则返回[]string{}。
- GetQueryArray(key string) ([]string, bool)：同QueryArray()，但是当key存在时，会额外返回一个true值，当key不存在时，会额外返回一个false值。
- QueryMap(key string) map[string]string：返回key的map值。例如：POST /post?ids[a]=1234&ids[b]=hello，使用QueryMap返回的map值为：ids: map[b:hello a:1234]。
- GetQueryMap(key string) (map[string]string, bool)：同QueryMap，但是当key存在时，会额外返回一个true值，当key不存在时，会额外返回一个false值。

2) 绑定查询字符串参数
- ShouldBindQuery(obj interface{}) error：绑定查询字符串参数到传入的结构体指针中，绑定出错时，返回错误内容。
- BindQuery(obj interface{}) error：绑定查询字符串参数到传入的结构体指针中，绑定出错时，会终止请求，并返回HTTP 400错误。

## 3. 表单参数

获取指定的表单参数：
- PostForm(key string) string：如果指定的参数key存在则返回其值，不存在则返回空字符串""。
- DefaultPostForm(key, defaultValue string)：如果指定的key存在则返回其值，不存在则返回指定的默认字符串defaultValue。
- GetPostForm(key string) (string, bool)：同PostForm()，但是当key存在时，会额外返回一个true值，当key不存在时，会额外返回一个false值。
- PostFormArray(key string) []string：返回指定key的slice，如果key不存在，则返回[]string{}。
- GetPostFormArray(key string) ([]string, bool)：同PostFormArray()，但是当key存在时，会额外返回一个true值，当key不存在时，会额外返回一个false值。
- PostFormMap(key string) map[string]string：返回key的map值
- GetPostFormMap(key string) (map[string]string, bool)：同PostFormMap，但是当key存在时，会额外返回一个true值，当key不存在时，会额外返回一个false值。

## 4. HTTP头参数

1) 获取指定的HTTP头参数
GetHeader(key string) string：如果指定的参数存在则返回其值，不存在则返回空字符串`""`。

2) 绑定HTTP头参数
- ShouldBindHeader(obj interface{}) error：绑定HTTP头参数到传入的结构体指针中，绑定出错时，返回错误内容。
- BindHeader(obj interface{}) error：绑定HTTP头参数到传入的结构体指针中，绑定出错时，会终止请求，并返回HTTP 400错误。

## 5. 消息体参数

- ShouldBind(obj interface{}) error

检查Content-Type并选择适配的绑定引擎，不同的Content-Type，选择的绑定引擎不同，Content-Type当前支持application/json、application/xml。

该函数会解析HTTP请求Body，如果Content-Type为application/json，则把body视为json格式，然后解析body字符串到传入的结构体指针中，如果绑定失败，返回错误内容。

- Bind(obj interface{}) error：跟ShouldBind()函数相似，但是当绑定失败时，会终止HTTP请求，并设置Content-Type="text/plain"，返回HTTP 400错误。
- BindJSON(obj interface{}) error：MustBindWith(obj, binding.JSON)函数的简单封装。
- ShouldBindJSON(obj interface{}) error：ShouldBindWith(obj, binding.JSON) 函数的简单封装。
- BindXML(obj interface{}) error：MustBindWith(obj, binding.XML) 函数的简单封装。
- ShouldBindXML(obj interface{}) error：ShouldBindWith(obj, binding.XML) 函数的简单封装。
- BindYAML(obj interface{}) error：MustBindWith(obj, binding.YAML) 函数的简单封装。
- ShouldBindYAML(obj interface{}) error：ShouldBindWith(obj, binding.YAML) 函数的简单封装。
- ShouldBindBodyWith(obj interface{}, bb binding.BindingBody) (err error)：类似于ShouldBindWith()函数，但是会把body存储在context中，方便以后再次使用，该函数在绑定前，会读取body，如果后面不需要再使用body，则可以直接调用ShouldBindWith()函数完成绑定功能，这样可以提高性能。

**助记：**ShouldBindXXX函数在绑定出错时，返回错误内容。BindXXX函数在绑定出错时，会终止请求，并返回HTTP 400错误。
