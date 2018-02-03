---
title: Docker 安装
date: 2018-02-03 18:45:41
description: 整理 Docker 安装步骤
tags:
- Docker
categories:
copyright: false
---

本笔记基于 `Centos 7` 服务器；

# Docker
## 安装
### Linux

`yum` 的仓库中有一个很旧的 `Docker` 包, 现在 `Docker` 官方已经将 `Docker` 更名为 `docker-engine`。使用下边的命令设置最新稳定版的 `docker` 仓库

```
yum-config-manager --add-repo https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo
```
如果不存在 `yum-config-manager` 命令，请执行 `yum -y install yum-utils`；

更新 `yum` 源

```
yum makecache fast
```

安装最新的 `docker`

```
yum -y install docker-engine
```
  
注意：`Docker 1.11.1` 版本要求 `Linux` 内核要求达到 `3.10.0`，可以通过命令 `cat /proc/version` 查看当前 `Linux`系统版本；


## 服务的启动与关闭
```
service docker start
```

如果想设置开机启动，可键入命令`chkconfig docker on`；

对应地，如果想要关闭服务

```
service docker stop
```

## 加速
使用 `DaoCloud ` 加速镜像获取，具体命令参考[DaoCloud 网页 - 加速器](http://www.daocloud.io/mirror#accelerator-doc)；

## 卸载
### yum 卸载
如果是通过 `yum` 方式安装，执行以下步骤

1. 找到安装包
 
  ```
  yum list installed | grep docker
  ```
2. 移出安装包
 
  ```
  yum -y remove  docker.x86_64
  ```
3. 删除镜像/容器

  ```
  rm -rf /var/lib/docker
  ```






# 参考阅读

- [CentOS7下安装docker](https://www.cnblogs.com/baolong/p/6526591.html)
- [玩转Docker镜像](http://blog.daocloud.io/how-to-master-docker-image/)




