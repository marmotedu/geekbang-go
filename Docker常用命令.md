## Docker常用命令

### 1. 辅助指令

```bash
docker version       | 显示 Docker 版本信息
docker info          | 显示 Docker 系统信息（包括镜像和容器数）
docker --help        | 帮助
```

### 2. 镜像指令

```bash
docker images [OPTIONS]            | 列出本机上的镜像（-a: 列出本地所有镜像、-q: 只显示镜像id）
docker search [OPTIONS] xxx镜像名  | 搜索镜像名
docker pull xxx镜像名字            | 拉取某镜像
docker rmi xxx镜像id/镜像名        | 删除某镜像
```

### 3. 容器指令

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]  | 启动容器
（-i:以交互模式运行容器;-t:为容器重新分配一个伪输入终端;-p:指定端口映射;-d:后台运行容器）
docker ps [OPTIONS]                            | 列出当前所有正在运行的容器
（-a:列出当前所有正在运行的容器+历史上运行过的;-l:显示最近创建的容器）
exit                                           | 交互模式中容器停止并退出交互
ctrl+P+Q                                       | 交互模式中容器不停止退出交互
docker start 容器id/容器名                     | 启动容器
docker restart 容器id/容器名                   | 重启容器
docker stop 容器id/容器名                      | 停止容器
docker kill 容器id/容器名                      | 强制停止容器
docker rm 容器id                               | 删除已停止的容器
docker rm -f $(docker ps -a -q)                | 删除全部容器
docker logs -f -t --tail 容器id                | 查看容器日志
docker top 容器id                              | 查看容器运行的进程
docker inspect 容器id                          | 查看容器内部信息
docker exec -it 容器id bashShell               | 进入容器执行指令
docker attach 容器id                           | 进入容器交互执行指令
docker cp 容器id:容器内路径 目的主机路径       | 从容器内拷贝文件到主机
```
