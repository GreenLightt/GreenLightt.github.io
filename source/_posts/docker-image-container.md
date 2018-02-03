---
title: Docker 仓库、镜像与容器
date: 2018-02-04 03:55:41
description: 整理 Docker 仓库、镜像与容器知识点
tags:
- Docker
categories:
copyright: false
---

# 镜像

## 查询

**列出机器上的所有镜像**

```
docker image ls
```

**查看镜像的详细信息**
`docker inspect` 命令返回的是一个 `json` 格式的消息，如果我们只要其中一项内容时，可以指定 `-f` 参数来指定；

```
docker inspect  ${IMAGE_ID}

docker inspect -f {{".Config.WorkingDir"}} bb3ac90ec48b
```

**从 `Docker Hub` 查找镜像**

```
docker search [OPTIONS] nginx
```

`OPTIONS` 说明

- `--automated` : 只列出 `automated build` 类型的镜像；
- `--no-trunc` :显示完整的镜像描述；
- `-s` :列出收藏数不小于指定值的镜像；

默认地输出结果将按照星级评价进行排序。官方的镜像说明是官方项目组创建和维护。 `automated` 资源允许用户验证镜像的来源和内容。


## 获取

**从 `Docker Hub` 中拉取或者更新指定镜像**
对于 `docker` 来说，如果不显式地指定 `Tag`，则会默认地选择 `latest` 标签，即下载仓库中最新版本的镜像；

```
docker pull [-a] name[:tag]
```

`-a` 参数拉取所有 `tagged` 镜像

## 增加

创建镜像的方法有三种：基于已有镜像的容器创建，基于本地模板导入，基于 `Dockerfile` 创建；

### 基于已有镜像的容器创建

```
docker commit [OPTIONS] container_id [Repository:[tag]] 
```

`OPTIONS` 说明

- `-a`， `--author=""`: 作者信息
- `-m`，`--message=""`: 提交信息
- `-p`，`--pause=true`: 提交时暂停容器运行

### 基于本地模板导入

也可以直接从一个操作系统模板文件导入一个镜像。推荐使用 [OpenVZ](https://download.openvz.org/template/precreated/) 提供的模板来创建。

比如，下载一个 `ubuntu-14.04` 的模板压缩后，可以使用以下命令导入：

```
sudo cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:14.04
```



## 删除

**删除一个镜像**

```
docker rmi ${IMAGE_ID}
```

**删除所有没有标签的镜像**

```
docker rmi $(docker images | grep “^” | awk “{print $3}”)
```

**删除所有的镜像**

```
docker rm $(docker ps -aq) 
```

**删除未使用的镜像**

```
docker rmi $(docker images --quiet --filter &quot;dangling=true&quot;)
```

## 存出和载入

可以使用 `docker save` 和 `docker load` 命令来存出和载入镜像。

如果要存出镜像到本地文件，可以使用 `docker save`命令。例如，存出本地的 `ubuntu:14.04` 镜像到 `ubuntu_14.04.tar`:

```
sudo docker save -o ubuntu_14.04.tar ubuntu:14.04
```


可以使用 `docker load` 从存出的本地文件中再导入到本地镜像库，例如从文件 `ubuntu_14.04.tar` 导入镜像到本地镜像列表

```
sudo docker load --input ubuntu_14.04.tar
# 或 sudo docker load < ubuntu_14.04.tar
```



# 容器

## 增

### 新建容器

可以使用 `docker create` 命令来创建容器， 例如

```
sudo docker create -it ubuntu:latest
```

使用 `docker create` 命令新建的容器处于停止状态，可以使用 `docker start`命令来启动。

### 新建并启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`stopped`）的容器重新启动。所需的命令主要为 `docker run`，等价于先执行 `docker create` 命令，再执行 `docker start` 命令；

当利用 `docker run` 来创建并启动容器时， `Docker` 在后台运行的标准操作包括：

1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载。
2. 利用镜像创建并启动一个容器。
3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层。
4. 从宿主机配置的网桥接口中桥接一个虚拟接口到容器中去。
5. 从地址池配置一个 `IP` 地址给容器。
6. 执行用户指定的应用程序。
7. 执行完毕后容器被终止。

下面的命令则启动一个 `bash` 终端，允许用户进行交互：

```
sudo docker run -t -i ubuntu:14.04 /bin/bash
```

用户可以按 `Ctrl + d` 或输入 `exit` 命令来退出容器；对于创建的容器，当使用 `exit` 命令退出之后，该容器就自动处于终止状态。


## 删除

**删除指定容器**

```
docker rm ${CID} 
```

**删除所有退出的容器**

```
docker rm -f $(docker ps -a | grep Exit | awk '{ print $1 }')
```

**删除所有的容器**

```
docker rm $(docker ps -aq)
```

## 查

**显示指定容器的IP**

```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID}
```

**查看Docker的底层信息,它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息**

```
docker inspect <container id/name>
```

**查看容器端口的映射情况**

```
docker port <container name/id> <port>
```


**显示正在运行的容器**

```
docker ps
```

**显示所有的容器**

```
docker ps -a
```

**显示所有退出状态为 1 的容器**

```
docker ps -a --filter &quot;exited=1&quot;
```

**查看容器内的标准输出**

```
docker logs ${CID}
```

## 运行

### 进入容器

**进入容器**

```
docker exec -it ${CID} bash
```

### 停止容器

**停止指定容器**

```
docker stop ${CID}
```

**停止所有正在运行的容器**

```
docker stop docker ps -q
```

## 导入与导出

导出容器是指导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态，可以使用 `docker export` 命令，该命令格式为 `docker export CONTAINER` 。

```
sudo docker export 943f73177d5c > test_for_run.tar
```

导出的文件又可以使用 `docker import ` 命令导入，成为镜像：

```
cat test_for_run.tar | sudo docker import - test/ubuntu:v1.0
```

实际上，既可以使用 `docker load` 命令来导入镜像存储文件到本地的镜像库，又可以使用 `docker import` 命令来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。

## 具体命令详参

### docker run

> docker run [OPTIONS] IMAGE [COMMAND] [ARG...]  

`OPTIONS` 说明：

- `-d`, `--detach=false`         指定容器运行于前台还是后台，默认为 `false `    
- `-i`, `--interactive=false`    打开 `STDIN`，用于控制台交互    
- `-t`, `--tty=false`           分配 `tty` 设备，该可以支持终端登录，默认为 `false`  - `-u`, `--user="" `             指定容器的用户    
- `-a`, `--attach=[]`            登录容器（必须是以 `docker run -d`启动的容器）  
- `-w`, `--workdir=""`          指定容器的工作目录   
- `-c`, `--cpu-shares=0`       设置容器 `CPU` 权重， 在 `CPU` 共享场景使用    
- `-e`, `--env=[] `              指定环境变量，容器中可以使用该环境变量    
- `-m`, `--memory="" `           指定容器的内存上限    
- `-P`, `--publish-all=false`    指定容器暴露的端口    
- `-p`, `--publish=[] `          指定容器暴露的端口   
- `-h`, `--hostname=""`          指定容器的主机名    
- `-v`, `--volume=[] `           给容器挂载存储卷，挂载到容器的某个目录    
- `--volumes-from=[]`          给容器挂载其他容器上的卷，挂载到容器的某个目录  
- `--cap-add=[]`              添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
- `--cap-drop=[]`            删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
- `--cidfile=""`               运行容器后，在指定文件中写入容器 `PID` 值，一种典型的监控系统用法    
- `--cpuset`=""                设置容器可以使用哪些`CPU`，此参数可以用来容器独占`CPU`    
- `--device=[]`                添加主机设备给容器，相当于设备直通    
- `--dns=[]`                   指定容器的 `dns` 服务器    
- `--dns-search=[]`            指定容器的 `dns` 搜索域名，写入到容器的 `/etc/resolv.conf` 文件    
- `--entrypoint=""`            覆盖 `image` 的入口点    
- `--env-file=[]`              指定环境变量文件，文件格式为每行一个环境变量    
- `--expose=[] `               指定容器暴露的端口，即修改镜像的暴露端口    
- `--link=[]`                  指定容器间的关联，使用其他容器的 `IP`、`env` 等信息    
- `--lxc-conf=[]`              指定容器的配置文件，只有在指定 `--exec-driver=lxc` 时使用    
- `--name=""`                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
- `--net="bridge"`             容器网络设置: 
    `bridge` 使用 `docker daemon` 指定的网桥       
    `host`    容器使用主机的网络    
    `container:NAME_or_ID` 使用其他容器的网路，共享IP和PORT等网络资源    
    `none` 容器使用自己的网络（类似 `--net=bridge`），但是不进行配置   
- `--privileged=false`         指定容器是否为特权容器，特权容器拥有所有的`capabilities  `  
- `--restart="no"`   指定容器停止后的重启策略: 
  `no`：容器退出时不重启    
  `on-failure`：容器故障退出（返回值非零）时重启   
  `always`：容器退出时总是重启    
- `--rm=false`                 指定容器停止后自动删除容器(不支持以 `docker run -d` 启动的容器)    
- `--sig-proxy=true`         设置由代理接受并处理信号，但是 `SIGCHLD`、`SIGSTOP` 和 `SIGKILL` 不能被代理   


# 仓库

仓库（`Repository`） 是集中存放镜像的地方。

一个容易与之混淆的概念是注册服务器（`Registry`） 。实际上注册服务器是存放仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。

## 创建私有仓库

安装好 `docker` 后，可以通过官方提供的 `registry` 镜像来简单搭建一套本地私有仓库环境：

```
docker run -d -p 5000:5000 registry
```

这将自动下载并启动一个 `registry` 容器，创建本地的私有仓库服务；

默认情况下，会将仓库创建在容器的 `/tmp/registry` 目录下。可以通过 `-v` 参数来将镜像文件存放在本地的指定路径上。例如下面的例子将上传的镜像放到 `/opt/data/registry` 目录：

```
docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry registry
```

## 使用私有仓库

假设当前私有仓库所有服务器的 `IP` 地址为 `172.16.1.68`； 

在当前服务器上查看已有的镜像：

```
sudo docker image ls
```

![][1]


使用 `docker tag` 命令将这个镜像标记为 `192.168.1.68:5000/redgo/php` （格式为 `docker tag IMAGE[:TAG] [REGISTORYHOST][USERNAME/]NAME[:TAG]`）

```
docker tag php:7.1.0-fpm 192.168.1.68:5000/redgo/php:7.1.0
```

使用 `docker push `上传标记的镜像

![此处输入图片的描述][2]

现在可以到任意一台能访问到 `192.168.1.68` 地址的机器去下载这个镜像了。


  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180204031130.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180204033525.png
