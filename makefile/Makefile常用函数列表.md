## Makefile常用函数列表

|函数名|功能描述|
|:----|:----|
|$(origin <variable>)|告诉变量的“出生情况”，有如下返回值：<br><ul><li>undefined：<variable> 从来没有定义过</li><li>default：<variable> 是一个默认的定义</li><li>environment：<variable> 是一个环境变量</li><li>file：<variable> 这个变量被定义在 Makefile中</li><li>command line：<variable> 这个变量是被命令行定义的</li><li>override：<variable> 是被 override 指示符重新定义的</li><li>automatic：<variable> 是一个命令运行中的自动化变量</li>|
|$(addsuffix <suffix>,<names...>)|把后缀<suffix>加到<names>中的每个单词后面，并返回加过后缀的文件名序列。|
|$(addprefix <prefix>,<names...>)|把前缀<prefix>加到<names>中的每个单词前面，并返回加过前缀的文件名序列。|
|$(wildcard <pattern>)|扩展通配符，例如：$(wildcard ${ROOT_DIR}/build/docker/\*)|
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
|$(dir <names...>)|从文件名序列<names>中取出目录部分。目录部分是指最后一个反斜杠（/）之前的部分。返回文件名序列<names>的目录部分。|
|$(notdir <names...>)|从文件名序列<names>中取出非目录部分。非目录部分是指最後一个反斜杠（/）之后的部分。返回文件名序列<names>的非目录部分。|
|$(strip <string>)|去掉<string>字串中开头和结尾的空字符，并返回去掉空格后的字符串|
|$(suffix <names...>)|从文件名序列<names>中取出各个文件名的后缀。返回文件名序列<names>的后缀序列，如果文件没有后缀，则返回空字串。|
|$(foreach <variable>,<list>,<text>)|把参数<list>中的单词逐一取出放到参数<variable>所指定的变量中，然后再执行<text>所包含的表达式。每一次 <text>会返回一个字符串，循环过程中<text>的所返回的每个字符串会以空格分隔，最后当整个循环结束时，<text>所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。|
