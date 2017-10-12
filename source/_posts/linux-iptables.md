---
title: Linux 下防火墙工具 iptables
date: 2017-09-28 16:27:00
description: 总结 iptables 的使用
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

# 简介 

`iptables`命令是`Linux上`常用的防火墙软件，是`netfilter`项目的一部分。可以直接配置，也可以通过许多前端和图形界面配置。

`iptables`可以用来过滤流入流出主机的数据包，其中一个应用场景是禁止异常`IP`地址访问`Web`暴露的端口，阻止`Dos`攻击；

![此处输入链接的描述][1]

由上表可以看到，`iptables`有三条链，`Chain INPUT`、`Chain FORWARD`和`Chain OUTPUT`；这三条链的默认策略都是`Policy`；

- `Chain INPUT`：负责过滤所有进入主机的数据包(最主要)
- `Chain FORWARD`：负责流经主机的数据包
- `Chain OUTPUT`：处理所有源地址都是本机地址的数据包（也就是主机发出去的数据包）

>  链（`chains`）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一 条或数条规则。当一个数据包到达一个链时，`iptables`就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据该条规则所定义的方法处理该数据包；否则`iptables`将继续检查下一条规则，如果该数据包不符合链中任一条规则，`iptables`就会根据该链预先定 义的默认策略来处理数据包。**所以，规则的次序非常关键，谁的规则越严格，应该放的越靠前，因为检查规则的时候，是按照从上往下的方式进行检查的。**

# 语法

> iptables [-t 表名] 命令选项 ［链名］ ［条件匹配］ ［-j 目标动作或跳转］

说明，

- `-t`参数：指定`iptables`命令所操作的表，有三种值`filter`、`nat`、`manage`；
- 命令选项：指定管理`iptables规则`的方式（比如：插入、增加、删除、查看等）；
- 链名：`INPUT`、`FORWARD`、`OUTPUT`
- 条件匹配：通过给定参数及值的方式，指定对符合什么样条件的数据包进行处理；
- `-j`参数：目标动作或跳转，用于指定数据包的处理方式，值有`ACCEPT`（允许）、`DROP`（丢弃）、`REJECT`（拒绝）、`REDIRECT`（跳转）；

## `Command`命令
1. 链管理命令

  - `-P`：设置指定链的默认策略（设定默认门是关着的还是开着的）；默认策略一般只有两种：`ACCEPT`和`DROP`； 例如 ` iptables -P INPUT (DROP|ACCEPT) `； 
  - `-N`： 新建一条用户自己定义的规则链，例如 ` iptables -N my_chain `； 
  - `-F`： 清空规则链，例如 ` iptables -F my_chain `；
  - `-X`： 删除用户自定义的空链，例如 ` iptables -X my_chain `；
  - `-E`： 给用户自定义的链重命名，例如 ` iptables -E my_chain my_new_chain `；    
2. 规则管理命令

  - `-A`：追加，在当前链的最后新增一个规则
  - `-I num`: 插入，把当前规则插入为第几条，不加参数，默认加在第一条。例如：`-I 3`表示插入为第三条；
  - `-R num`：`Replays`替换/修改第几条规则；格式：`iptables -R 3 …………`；
  - `-D num`：删除，明确指定删除第几条规则；
3. 查看管理命令

  - `-L` 列出（`list`）指定链中所有的规则进行查看
  - `-n`：以数字的方式显示`ip`，它会将`ip`直接显示出来，如果不加`-n`，则会将`ip`反向解析成主机名。
  - `-v`：显示详细信息
  - `-vv`
  - `-vvv`：越多越详细
  - `--line-numbers` : 显示规则的行号
  - `-t nat`：显示所有的关卡的信息

## 匹配参数
### 通用匹配

- `-s`：指定作为源地址匹配，这里不能指定主机名称，必须是`IP`（`IP` | `IP/MASK` | `0.0.0.0/0.0.0.0`），而且地址可以取反，加一个“!”表示除了哪个`IP`之外；
- `-d`：表示匹配目标地址
- `-p`：用于匹配协议的（这里的协议通常有3种，`TCP`/`UDP`/`ICMP`）
- `-i eth0`：从这块网卡流入的数据，流入一般用在`INPUT`和`PREROUTING`上
- `-o eth0`：从这块网卡流出的数据，流出一般用在`OUTPUT`和`POSTROUTING`上

### 扩展匹配

1. 隐含扩展：对协议的扩展

    - `-p tcp`：TCP协议的扩展。一般有三种扩展
	    - `--dport XX-XX`：指定目标端口,不能指定多个非连续端口,只能指定单个端口，比如`--dport 21`  或者 `--dport 21-23` (此时表示21,22,23)
	    - `--sport`：指定源端口
	    - `--tcp-fiags`：TCP的标志位（`SYN`，`ACK`，`FIN`，`PSH`，`RST`，`URG`）
    - `-p udp`：`UDP`协议的扩展
        - `--dport`
        - `--sport`
    - `-p icmp`：`icmp`数据报文的扩展
        - `--icmp-type`：
		    `echo-request`（请求回显)，一般用 8 来表示，所以 `--icmp-type 8` 匹配请求回显数据包
		    `echo-reply`（响应的数据包)，一般用 0 来表示

2. 显示扩展
  扩展各种模块；`-m multiport`表示启用多端口扩展，之后我们就可以启用比如` --dports 21,23,80`；

# 常用命令

## 永久关闭与开启防火墙
关闭 `chkconfig iptables off`
开启 `chkconfig iptables on`

## 查看防火墙状态
```
service iptables status
```

# 参考阅读

- [永志●哥德的博客 -  iptables详解](http://www.cnblogs.com/metoy/p/4320813.html)
    可以从该博客扩展阅读一些 `Demon`示例
- [hhktony 的博客 -  iptables详解](http://blog.chinaunix.net/uid-26495963-id-3279216.html)
    可以从该博客扩展阅读一些原理和对`state`概念的理解

  [1]: http://owk2q4gs5.bkt.clouddn.com/iptables.png
