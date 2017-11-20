---
title: Superset 介绍与环境搭建
date: 2017-11-21 19:56:00
description: Superset 介绍与环境搭建
tags:
- Superset
categories:
copyright: false
---

# 介绍
`Superset` 是一个数据挖掘和可视化的 `Web` 应用；

`Superset` 提供以下功能:

- 快速创建可交互的、直观形象的数据集合
- 有丰富的可视化方法来分析数据，且具有灵活的扩展能力
- 简单, 代码自由, 可对暴露在仪表盘上的数据进行切片操作； 仪表盘和图表是更深层次分析的起点；
- 丰富的在线 `SQL` 编辑器，以及一个简单的工作流来创建任何结果集的可视化。
- 具有可扩展的、高粒度的安全模型，可以用复杂规则来控制访问权限。目前支持主要的认证提供商：`DB`、`OpenID`、`LDAP`、`OAuth`、和 `Flask AppBuiler` 的 `REMOTE_USER`
- 使用简单的语法，就可以控制数据在 `UI` 中的展现方式
- 支持大多数 `SQL` 数据库
- 与 `Druid` 深度结合，可快速的分析大数据
- 配置缓存来快速加载仪表盘

有效链接：

- [superset 官网](http://airbnb.io/projects/superset/)
- [superset 官网上的文档](https://superset.incubator.apache.org/installation.html)
- [github - superset的仓库](https://github.com/apache/incubator-superset)

`Superset`的技术组成：后端是基于`Python`，采用了 `Flask`、`Pandas`、`SqlAlchemy`框架等；前端用了用到了`npm`、`react`、`webpack`；

# 安装
笔者基于`Centos 6.8` 系统安装；

**安装系统环境依赖**
```
yum -y upgrade python-setuptools
yum -y install gcc gcc-c++ libffi-devel python-devel python-pip python-wheel openssl-devel libsasl2-devel openldap-devel
```

**建立 `python` 虚拟环境**
```
pip install virtualenv
virtualenv venv
. ./venv/bin/activate    # 激活，进入虚拟环境
```
要退出虚拟环境 `deactivate`

**准备 `python` 环境与依赖**
```
pip install --upgrade setuptools pip
```

**安装与初始化**
```
# 安装 superset；从清华的源获取
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple  superset

# 创建一个 admin 用户 (你会被提示输入 username first 和 lastname 在输入密码前)
fabmanager create-admin --app superset

# 初始化数据库
superset db upgrade

# 加载一些例子数据
superset load_examples

# 初始化，创建默认的角色和权限
superset init

# 启动 web 服务器，端口号 8088， 可以用 -p 参数指定其它端口号
superset runserver

# 要开启一个开发服务器，使用 -d 参数
# superset runserver -d
```

# 入门操作
## 连接 mysql

**准备**

`superset` 默认安装是没有准备连接 `mysql` 的环境，因为需要提前准备好连接 `mysql` 的一些工具包及驱动

```
pip install pymysql

yum -y install mysql-devel
pip install mysql-python
```

**新建 mysql 的数据库连接**
“数据库” 》 “数据源” 》添加新记录
 
数据库名 ： 起一个有标识度，自己能分辨的名称
`SQLAlchemy URI` ： 例如 `mysql+mysqldb://root:111111@192.168.1.168/test?charset=utf8`
在SQL工具箱中公开：勾选则可以在 `sql 工具箱`中操作；

点击保存；








