---
title: Docker Dockerfile
date: 2018-02-06 23:08:41
description: 整理 Dockerfile 书写要求
tags:
- Docker
categories:
copyright: false
---

`Dockerfile` 是一个文本格式的配置文件，用户可以使用 `Dockerfile` 快速创建自定义的镜像。

# 基本结构

`Dockerfile` 由一行行命令语句组成，并且支持 `#` 开头的注释行。

一般而言，`Dockerfile` 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

# 指令

指令的一般格式为 `INSTRUCTION arguments`，指令包括 `FROM`、`MAINTAINER`、`RUN`等。下面分别介绍。

## FROM

> 格式为： `FROM <image>` 或 `FROM <image>:<tag>`

第一条指令必须为 `FROM` 指令。并且， 如果在同一个 `Dockerfile` 中创建多个镜像时，可以使用多个 `FROM` 指令（每个镜像一次）。


## MAINTAINER

> 格式为： `MAINTAINER <name>`，指定维护者信息

## ENV

> 格式为： `ENV <key> <value>`

指定一个环境变量，会被后续 `RUN` 指定使用，并在容器运行时保持。 例如：

![此处输入图片的描述][1]

## RUN

> 格式为：`RUN <command>` 或 `RUN ["executable"， "param1"， "param2"]`

前者将在 `shell` 终端中运行命令，即 `/bin/sh -c` ；后者则使用 `exec` 执行；指定使用其他终端可以通过第二种方式实现，例如 `RUN  ["/bin/bash", "-c", "echo hello"]`

每条 `RUN` 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 `\` 来换行。

## CMD

> 支持三种格式
1. `CMD ["executable", "param1", "param2"]` 使用 `exec` 执行，推荐方式
2. `CMD command param1 param2` 在 `/bin/bash` 中执行，提供给需要交互的应用
3. `CMD ["param1", "param2"]` 提供给 `ENTRYPOINT` 的默认参数

指定启动容器时执行的命令。每个 `Dockerfile` 只能有一条 `CMD` 命令。如果指定了多条命令，则只有最后一条会被执行。

如果用户启动容器时候，指定了运行的命令，则会覆盖 `CMD` 指定的命令。

## ENTRYPOINT

> 有两种格式
1. `ENTRYPOINT ["executable", "param1", "param2"]`
2. `ENTRYPOINT command param1 param2`

配置容器启动后执行的命令，并且不可被`docker run` 提供的参数覆盖；

每个 `Dockerfile` 中只能有一个 `ENTRYPOINT` ，当指定多个 `ENTRYPOINT` 时，只有最后一个生效；

## EXPOSE

> 格式为： `EXPOSE <port> [<port>...]`

例如：

```
EXPOSE 22 80 8443
```

告诉 `Docker` 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 `-P` 。`Docker` 主机会自动分配一个端口转发到指定的端口；使用 `-p` ，则可以具体指定哪个本地端口映射过来；



## ADD

> 格式为： `ADD <src> <dest>`

该命令将复制指定的 `<src>` 到容器中的 `<dest>`。其中，`<src>`可以是 `Dockerfile` 所在目录的一个相对路径（文件或目录）；也可以是一个 `URL`；还可以是一个 `tar` 文件（自动解压为目录）。

## COPY

> 格式为： `COPY <src> <dest>`

复制本地主机的 `<src>` （为 `Dockerfile` 所在目录的相对路径，文件或目录）为容器中的 `<dest>`。目标路径不存在时，会自动创建。

当使用本地目录为源目录时，推荐使用 `COPY`。


## VOLUME

> 格式为： `VOLUME ["/data"]`

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等；

## USER

> 格式为： `USER daemon`

指定运行容器时的用户名或 `UID`，后续的 `RUN` 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，使用 `RUN groupadd -r postgres && useradd -r -g postgres postgres`。要临时获取管理员权限可以使用 `gosu`，而不推荐 `sudo`。

## WORKDIR

> 格式为： `WORKDIR /path/to/workdir`

为后续的 `RUN`、`CMD`、`ENTRYPOINT`指令配置工作目录。

可以使用多个 `WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径，例如

```
WORKDIR /a
WORKDIR b
WORKDIR c
```
则最终路径为 `/a/b/c`；

## ONBUILD

> 格式为： `ONBUILD [INSTRUCTION]`

配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令。例如， `Dockerfile` 使用如下的内容创建了镜像 `image-A`

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```


如果基于 `image-A` 创建新的镜像时，新的 `Dockerfile` 中使用 `FROM image-A` 指定基础镜像时，会自动执行 `ONBUILD` 指令内容，等价于在后面添加了两条指令。

```
FROM image-A
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
# Automatically run the following

```

使用 `ONBUILD` 指令的镜像，推荐在标签中注明，例如 `ruby:1.9-onbuild`；

# 创建镜像

编写完成 `Dockerfile` 之后，可以通过 `docker build` 命令来创建镜像。

基本的格式为 `docker build [选项] 路径`， 该命令将读取指定路径下（包括子目录）的 `Dockerfile` ，并将该路径下所有内容发送给 `Docker` 服务端，由服务端来创建镜像。因此一般建议放置 `Dockerfile` 的目录为空目录。

另外，可以通过 `.dockerignore` 文件（每一行添加一条匹配模式） 来让 `Docker` 忽略路径下的目录和文件。

要指定镜像的标签信息，可以通过 `-t` 选项。

例如，指定 `Dockerfile` 所在路径为 `/tmp/docker_build/` ， 并且希望生成镜像标签为 `build_repo/first_image` ， 可以使用下面的命令：

```
sudo docker build -t build_repo/first_image /tmp/docker_build/
```

  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180205223156.png
