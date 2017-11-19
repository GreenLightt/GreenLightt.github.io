---
title: Cenos7 网络配置、换源、更新 yum
date: 2017-11-19 15:07:00
description: 整理 Cenos7 网络配置、换源、更新 yum
tags:
- Centos7
categories:
- Linux
copyright: false
---

`Centos 7` 使用 `ip` 命令替换 `ifconfig`；键入命令，查看当前网络配置情况

```
 > ip addr
```

![此处输入链接的描述][1]

从图片可以发现，保存网卡 `ip` 信息的配置文件名也由以前的 `ifcfg-eth0` 变成了 `ifcfg-enp0s3`;

**动态IP**
通过编辑 `/etc/sysconfig/network-scripts/ifcfg-enp0s3`

```
BOOTPROTO=dhcp
......
ONBOOT=yes
```
重启网络服务  `service network restart`

**固定IP**
编辑 `/etc/sysconfig/network-scripts/ifcfg-enp0s3`
```
BOOTPROTO=static

ONBOOT=yes
IPADDR=192.168.1.68
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
BROADCAST=192.168.1.255

```
编辑 `/etc/resolv.conf`
```
nameserver 192.168.1.1
search bogon
```

重启网络服务  `service network restart`

# 附录
## 换源
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
wget -P /etc/yum.repos.d http://mirrors.163.com/.help/CentOS6-Base-163.repo 
mv /etc/yum.repos.d/CentOS6-Base-163.repo /etc/yum.repos.d/CentOS-Base.repo
```
## yum 更新
```
# 安装epel, epel是免费开源发行软件包版本库
yum install -y epel-release
# 更新 yum
yum -y update
```


  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171119144133.png
