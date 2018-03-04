---
title: Docker 多容器搭建 lnmp 环境
date: 2018-03-04 15:34:00
description: Docker 多容器搭建 lnmp 环境
tags:
- Docker
categories:
copyright: false
---

笔者是在 `centos 7` 系统上进行部署操作；


## 准备

从 `Docker Hub` 查找与 `nginx` 相关镜像

```
docker search nginx
```

下拉最新版的 `nginx` 镜像，位置放在 `/var/lib/docker` 下

```
docker pull nginx
```


下拉 `php7.1` 版本

```
docker pull php:7.1.0-fpm
```

下拉 `mysql` 最新版本

```
docker pull mysql
```

## mysql

**创建用于挂载的目录**

```
# 用于挂载 mysql 数据文件
mkdir -p /home/docker_lnmp/mysql/data

# #用于挂载 mysql 配置文件
mkdir -p /home/docker_lnmp/mysql/conf.d
```

**使用镜像创建容器**

先创建一个简单的容器，然后拷贝出配置文件，删除这个简单的容器；（如果有 `mysql` 的配置文件，此步可跳过）

```
# 创建简单容器，返回 容器ID
docker run --name mysql_180227_latest -p 3306:3306  -e MYSQL_ROOT_PASSWORD=123456 -d mysql
# 复制配置目录
docker cp mysql_180227_latest:/etc/mysql/conf.d /home/docker_lnmp/mysql/
# 停止并删除简单容器
docker stop mysql_180227_latest 
docker rm mysql_180227_latest
```

创建正式容器

```
docker run \
    --name mysql_180227_latest \
    -p 3306:3306 \
    -v /home/docker_lnmp/mysql/data:/var/lib/mysql \
    -v /home/docker_lnmp/mysql/conf.d:/etc/mysql/conf.d \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -d \
    mysql
```

## php

**创建用于挂载的目录**

```
# 用于挂载 php 配置目录
mkdir /home/docker_lnmp/php/conf.d
# 用于挂载 php fpm 配置目录
mkdir /home/docker_lnmp/php/php-fpm.d
# 用于挂载 php 日志
mkdir /home/docker_lnmp/php/logs

# 用于挂载 nginx 项目文件
mkdir -p /home/docker_lnmp/nginx/www
```

**使用镜像创建容器**

先创建一个简单的容器，然后拷贝出配置文件，删除这个简单的容器；（如果有 `php` 的配置文件，此步可跳过）

```
# 创建简单容器，返回 容器ID
docker run --name php_180227_7.1.0 -p 9000:9000  -d php:7.1.0-fpm
# 复制配置目录
docker cp php_180227_7.1.0:/usr/local/etc/php-fpm.d /home/docker_lnmp/php/
# 停止并删除简单容器
docker stop php_180227_7.1.0 
docker rm php_180227_7.1.0
```

创建正式容器

```
docker run \
    --name php_180227_7.1.0 \
    -p 9000:9000 \
    --link mysql_180227_latest:mysql_180227_latest \
    -e TIMEZONE=Asia/Shanghai \
    -v /home/docker_lnmp/php/php-fpm.d:/usr/local/etc/php-fpm.d \
    -v /home/docker_lnmp/php/conf.d:/usr/local/etc/php/conf.d \
    -v /home/docker_lnmp/nginx/www:/var/www/html \
    -d \
    php:7.1.0-fpm
```

## nginx

**创建用于挂载的目录**

```
# 用于挂载 nginx 项目文件
mkdir -p /home/docker_lnmp/nginx/www
# 用于挂载 nginx 配置文件
mkdir -p /home/docker_lnmp/nginx/conf.d
# 用于挂载 nginx 日志文件
mkdir -p /home/docker_lnmp/nginx/logs
```

**使用镜像创建容器**

先创建一个简单的容器，然后拷贝出配置文件，删除这个简单的容器；（如果有 `nginx` 的配置文件，此步可跳过）

```
# 创建简单容器，返回 容器ID
docker run --name nginx_180227_latest -p 80:80  -d nginx
# 复制配置目录
docker cp nginx_180227_latest:/etc/nginx/conf.d /home/docker_lnmp/nginx/
# 停止并删除简单容器
docker stop nginx_180227_latest 
docker rm nginx_180227_latest
```

创建正式容器

```
docker run \
    --name nginx_180227_latest \
    -p 80:80 \
    --link php_180227_7.1.0:php_180227_7.1.0 \
    -v /home/docker_lnmp/nginx/www:/usr/share/nginx/html \
    -v /home/docker_lnmp/nginx/conf.d:/etc/nginx/conf.d \
    -v /home/docker_lnmp/nginx/logs:/var/log/nginx \
    -d \
    nginx
```
修改宿主机上 `nginx` 配置文件 `conf.d/default.conf`

```
server {
    listen       80;
    server_name  localhost;
    root /usr/share/nginx/html/;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_pass   php_180227_7.1.0:9000;
        fastcgi_index  index.php;
	# 这里的不能写 $document_root 要写 php 的文件位置
        fastcgi_param  SCRIPT_FILENAME  /var/www/html$fastcgi_script_name;
        include        fastcgi_params;
    }

    location ~ /\.ht {
        deny  all;
    }
}
```

如果配置文件有修改，需要重启容器生效：

```
docker restart nginx_180227_latest
```

## 心得

1. `docker pull ` 下来的镜像中基于的系统是 `Redhat 4.8` ，系统很老，老的好处在于镜像小而快；
2. 整体操作步骤多；










