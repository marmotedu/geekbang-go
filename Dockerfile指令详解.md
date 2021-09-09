## Dockerfile指令详解

Dockerfile中包含了大量的指令，这些指令完成的功能，使用的格式都不同。这里，我详细介绍下这些指令。

1) `FROM`

格式：`FROM <image>`或`FROM <image>:<tag>`

`FROM`指令的功能是为后面的指令提供基础镜像，因此，Dockerfile必须以`FROM`指令作为第一条非注释指令。我们可以根据需要，选择任何有效的镜像作为基础镜像。

可以在一个Dockerfile文件中，使用多个 `FROM` 指令，来构建多个镜像。每个镜像构建完成之后，Docker会打印出该镜像的ID。如果 `FROM` 指令中没有指定镜像的tag，则默认tag是 `latest` 。如果镜像不存在，则报错退出。

2) `MAINTAINER`

格式：`MAINTAINER <name> <Email>`

用来指定维护该Dockerfile的作者信息。

3) `ENV`

`ENV`指令有两种格式：

* `ENV <key> <value>`
* `ENV <key1>=<value1> <key2>=<value2>`

`ENV`指令用来在镜像创建出来的容器中声明环境变量，被声明的环境变量可被后面的特定指令，例如：`ENV`、`ADD`、`COPY`、`WORKDIR`、`EXPOSE`、`VOLUME`、`USER`使用。其他指令引用格式为：`$variableName`或`${variableName}`。可以使用斜杠 `\` 来转义环境变量，例如：`\$test`或者`\${test}`，这样二者将会被分别转换为`$test`和`${test}`字符串，而不是环境变量所保存的值。这里要注意，`ONBUILD`指令不支持环境替换。

4) `WORKDIR`

格式：`WORKDIR <工作目录路径>`

该指令用于指定当前的工作目录，使用该命令后，接下来每一层的工作目录都会切换到指定的目录（除非重新使用 `WORKDIR` 切换）。

`WORKDIR` 的路径始终使用绝对路径。这可以保证指令的准确和可靠。 同时，使用 `WORKDIR` 来替代`RUN cd ... && do-something` 这样难以维护的指令。

5) `COPY`

格式：`COPY <src> <dest>`

`COPY` 指令复制本机上的 `<src>` 文件或目录到镜像的 `<dest>` 路径下。若`<dest>`或`<src>`以反斜杠`/`结尾则说明其指向的是目录，否则指向文件。

`<src>`可以是多个文件或目录，但必须是上下文根目录中的相对路径。不能使用形如 `COPY ../filename /filename`这样的指令。此外，`<src>`支持使用通配符指向所有匹配通配符的文件或目录，例如，`COPY iam* /opt/`表示添加所有以 `iam` 开头的文件到目录`/opt/`中。

`<dest>`可以是文件或目录，但必须是镜像中的绝对路径或者相对于`WORKDIR`的相对路径。 若`<dest>`是一个文件，则`<src>`的内容会被复制到`<dest>`中；否则`<src>`指向的文件或目录中的内容会被复制到`<dest>`目录中。当`<src>`指定多个源时，`<dest>`必须是目录。如果`<dest>`在镜像中不存在，则会被自动创建。

6) `ADD`

格式：`ADD <src> <dest>`

`ADD`与`COPY`指令具有相似的功能，都支持复制本地文件到镜像的功能。但`ADD`指令还支持其他功能。在 `ADD` 指令中，`<src>`可以是一个网络文件的下载地址， 比如 `ADD http://example.com/iamctl.yaml /`会在镜像中创建文件`/iamctl.yaml`。

`<src>`还可以是一个压缩归档文件，该文件在复制到容器时会被解压提取，例如`ADD iam.tar.xz /`。但是若URL中的文件为归档文件，则不会被解压提取。

在编写Dockerfile时，推荐使用`COPY`，因为`COPY`只支持本地文件，它比 `ADD` 更加透明。

7) `RUN`

`RUN`指令有两种格式：

* `RUN <command>` (shell格式)
* `RUN ["executable", "param1", "param2"]` (exec格式，推荐使用)

`RUN`指令会在前一条命令创建出的镜像的基础上创建一个容器，并在容器中运行命令，在命令结束运行后提交容器为新镜像，新镜像被Dockerfile中的下一条指令使用。

当使用shell格式时，命令通过`/bin/sh -c`运行。当使用exec格式时，命令是直接运行的，即不通过shell来运行命令。这里要注意，exec格式中的参数会以 `JSON` 数组的格式被Docker解析，所以参数必须使用双引号而不是单引号。

因为exec格式不会在shell中执行，所以环境变量不会被替换。比如，执行`RUN ["echo", "$USER"]`指令时，`$USER`不会做变量替换。如果希望运行shell程序，指令可以写成 `RUN ["/bin/bash", "-c", "echo", "$USER"]`。

8) `CMD`

`CMD`指令有3种格式:

* `CMD <command>` (shell格式)
* `CMD ["executable", "param1", "param2"]` (exec格式，推荐使用)
* `CMD ["param1", "param2"]` (为`ENTRYPOINT`指令提供参数)

`CMD`指令提供容器运行时的默认命令或参数，一个Dockerfile中可以有多条`CMD`指令，但只有最后一条`CMD`指令有效。`CMD ["param1", "param2"]`格式用来跟`ENTRYPOINT`指令配合使用，`CMD`指令中的参数会添加到`ENTRYPOINT`指令中。当使用shell和exec格式时，命令在容器中的运行方式与`RUN`指令相同。如果在执行`docker run` 时指定了命令行参数，则会覆盖`CMD`指令中的命令。

9) `ENTRYPOINT`

`ENTRYPOINT`指令有两种格式：

* `ENTRYPOINT <command>` (shell格式)
* `ENTRYPOINT ["executable", "param1", "param2"]` (exec格式，推荐格式)

`ENTRYPOINT`指令和`CMD`指令类似，都可以让容器在每次启动时执行指定的命令，但二者又有不同。`ENTRYPOINT`只能是命令 ，而`CMD`可以是参数，也可以是指令。另外，`docker run`命令行参数可以覆盖`CMD`，但不能覆盖`ENTRYPOINT`。

当使用Shell格式时，`ENTRYPOINT`指令会忽略任何`CMD`指令和`docker run`命令的参数，并且会运行在`bin/sh -c`中。

推荐使用exec格式。使用exec格式时，`docker run`传入的命令行参数会覆盖`CMD`指令的内容并且追加到`ENTRYPOINT`指令的参数中。

一个Dockerfile中可以有多条`ENTRYPOINT`指令，但只有最后一条`ENTRYPOINT`指令有效。

10) `ONBUILD`

格式：`ONBUILD [INSTRUCTION]`

`ONBUILD` 指令后面跟的是其它指令，例如  `RUN` ,  `COPY` 等。这些指令，在当前镜像构建时不会被执行，当以当前镜像为基础镜像，构建下一级镜像时才会被执行。

这里要注意，使用包含`ONBUILD`指令的Dockerfile，构建的镜像应该打上特殊的标签，比如：`python:3.0-onbuild`。另外，在`ONBUILD`指令中使用`ADD`或`COPY`指令时要特别注意。假如新的构建过程的上下文中缺失了被添加的资源，则新的构建过程会失败。我们可以通过给`ONBUILD`镜像添加特殊标签，来提示编写Dockerfile的开发人员要特别注意。

11) `VOLUME`

`VOLUME`指令有两种格式：

* `VOLUMN ["<路径1>", "路径2"...]`
* `VOLUMN <路径>`

为了防止容器内的重要数据因为容器重启而丢失，且避免容器不断变大，应该将容器内的某些目录挂载到宿主机上。在Dockerfile中，我们可以通过 `VOLUME` 指令事先将某些目录挂载为匿名卷，这样在执行`docker run`时，如果没有指定 `-v` 选项，则默认会将`VOLUMN`指定的目录挂载为匿名卷 。

12) `EXPOSE`

格式：`EXPOSE <端口1> [<端口2> ...]`

`EXPOSE`指定了该镜像生成容器时提供服务的端口（默认对外不暴露端口），可以配合`docker run -P`使用，也可以配合`docker run --net=host`使用。

13) `LABEL`

格式：`LABEL <key>=<value> <key>=<value> <key>=<value> ...`

`LABEL` 指令用来给镜像添加一些元数据（metadata）。

14) `USER`

格式：`USER <用户名>[:<用户组>]`

`USER`指令设定一个用户或者用户ID，在执行`RUN`、`CMD`、`ENTRYPOINT`等指令时指定以那个用户得身份去执行。如果容器中的服务不需要以特权身份来运行，则可以使用 `USER` 指令，切换到非root用户，以此来增加安全性。

15) `ARG`

格式：`ARG <参数名>[=<默认值>]`

与`ENV`作用一致，但作用域不相同。通过`ARG`指定的环境变量仅在`docker build`的过程中有效，构建好的镜像内不存在此变量。虽然构建好的镜像内不存在此变量，但是使用`docker history`可以查看到。所以，敏感数据不建议使用`ARG`。

可以在`docker build`时用`--build-arg <参数名>=<值>`来覆盖`ARG`指定的参数值 。

16) `HEALTHCHECK`

`HEALTHCHECK`指令有两种格式：

* `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
* `HEALTHCHECK NONE`：用来屏蔽掉已有的健康检查指令。

定时检测容器进程是否健康，可以设置检查的时间间隔、超时时间、重试次数、以及用于判断是否健康的指令。其中，`CMD`指定的`<命令>`的返回值决定了该次健康检查是否成功：`0` 成功；`1` 失败；`2` 保留。

`HEALTHCHECK`支持下列指令：

* --interval=<时间间隔>：设置健康检查的时间间隔，默认是 30s。
* --timeout=<超时时长>：设置单次健康检查的超时时间，默认是 30s。
* --retries=<重试次数>：如果健康检查失败，重试多少次，默认是 3 次。如果连续 3 次健康检查都失败，容器的 STATUS 就会显示 unhealthy。

例如：

```dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/list*
HEALTHCHECK --interval=5s --timeout=3s\
    CMD curl -fs http://localhost/ || exit 1
```

上面的Dockerfile中，使用`curl -fs http://localhost/ || exit 1`作为健康检查命令 ，每`5`秒检查一次容器是否健康，如果健康检查命令超过`3`秒没响应就视为检查失败。

17) `STOPSIGNAL`

格式：`STOPSIGNAL signal`

`STOPSIGNAL`指令指定了容器退出时，发送给容器的系统调用信号。
