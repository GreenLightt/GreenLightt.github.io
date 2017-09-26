﻿---
title: 练习 MongoDB 操作 —— 复制集篇（四）
date: 2017-09-24 17:57:41
description: 学习 MongoDB 的复制集功能
tags:
- MongoDB3.x
categories:
- NoSQL
---

复制集（`Replication Set`），也叫副本集；作用是把一份数据同时保存在多台服务器上，保证数据的安全，不发生丢失；

# 示例

## 启动服务器
通过指定 `replSet`选项启动三台`Mongo`服务器，端口号是 27017， 27018， 27019；指定加入的复制集名称为 `demo`；

```
./bin/mongod --dbpath master/data --logpath master/log/0923.log --port 27017 --fork --replSet demo

./bin/mongod --dbpath slave1/data --logpath slave1/log/0923.log --port 27018 --fork --replSet demo

./bin/mongod --dbpath slave2/data --logpath slave2/log/0923.log --port 27019 --fork --replSet demo
```

此时，在三台服务器上分别键入`rs.status()`命令，均报“未初始化”的错误；

启动服务器的另外一种方式是通过配置文件启动；举个例子，这里的`27018`以配置文件启动；

配置文件如下所示：
```
 dbpath=/home/mongodb/slave1/data             # 数据存放目录
 logpath=/home/mongodb/slave1/log/0923.log    # 日志存放路径
 pidfilepath=/home/mongodb/slave1/slave1.pid  # 进程文件，方便停止mongodb
 directoryperdb=false      # 为每一个数据库按照数据库名建立文件夹存放
 logappend=true                               # 以追加的方式记录日志
 replSet=demo                                 # Replication Set 的名称
 #bind_ip=127.0.0.1                           # 限制只允许某一特定IP来访问，逗号隔开
 port=27018                                   # 进程所使用的端口号，默认为27017
 oplogSize=10000          # mongodb操作日志文件的最大大小。单位为Mb，默认为硬盘剩余空间的5%
 fork=true                                    # 以后台方式运行进程                  
 noprealloc=false                             # 不预先分配存储                      
 smallfiles=false                             # 当提示空间不够时添加此参数
```

## 初始化
登入任意一台机器的 `MongoDB` 执行，因为是全新的复制集，所以可以任意进入一台执行；要是一台有数据，则需要在有数据上执行；要多台有数据则不能初始化。

笔者选择在`27017`这台`MongoDB`服务器进行初始化

```
rs.initiate({
    _id:'demo',  // 复制集名称
    members:    // 复制集服务器列表
    [
        {
	    _id:0, // 服务器的唯一 ID
	    host:'192.168.1.168:27017' // 服务器的地址
        },
        {
	    _id:1,
	    host:'192.168.1.168:27018'
        },
        {
	    _id:2,
	    host:'192.168.1.168:27019'
        },
    ]
});
```
这样，三台服务器都加入了`demo`复制集；此时主节点`27017`可以进行读写，从节点`27018`和`27019`不可以读写；**所有的写操作只能在主节点上进行；**

那如何才能在从节点`27018`上读数据呢？在`27018`上键入`rs.slaveOk()`， 这样就可以进行读操作；

注：可以通过`rs.status()`查看复制集状态；通过`rs.config()`查看复制集的配置信息；通过`db.isMaster()`查看是否是主节点信息；

## 添加从节点
先启动想要添加的从节点服务器，笔者是本地的`27020`服务器，同时指定相要加入的复制集名称；

```
./bin/mongod --dbpath slave3/data --logpath slave3/log/0923.log --port 27020 --fork --replSet demo
```

主节点`27017`配置添加：

```
rs.add('192.168.1.168:27020');
```

此时，通过键入`rs.status()`可以看到`members`成员有四位，包括这台新的`27020`从节点服务器；

如果想要移除`27020`从节点服务器，即可成功从`demon`复制集中移除该服务器；

```
rs.remove('192.168.1.168:27020');
```

## 模拟主服务器故障
目前环境：

- `27017`: `Primary`主服务器
- `27018`: `SECONDARY`从服务器
- `27019`: `SECONDARY`从服务器

在系统命令行上，手动`drop`掉主服务器

```
lsof -i:27017 | sed '1d' | while read line
do
    echo $line | awk '{print $2}' | xargs kill -9
done
```
或者在主服务器上执行下面的语句
```
db.shutdownServer()
```

此时，在通过`rs.status()`查看环境

- `27017`: 离线（`not reachable/healthy`）
- `27018`: `SECONDARY`从服务器
- `27019`: `Primary`主服务器

这里，为什么在主服务器`27017`服务器挂了之后，会选择`27019`做新的主服务器呢，这是由`MongoDB`服务器内部的多数投票算法，即每台服务器会为`secondary`从节点服务器进行投票，票数多的从节点服务器会成为新的`Primary`服务器；

笔者这里很幸运，两台从节点服务器`27018`和`27019`都选择了`27019`作为新的从节点服务器；另外一种可能是 `27018`和`27019`各得一票，这样会导致无法选举出新`Primary`服务器; 所以建议复制集的服务器数目为奇数，如果碰巧是偶数，可以添加一台仲裁节点服务器；

## 添加仲裁节点
仲裁节点是一种特殊的节点，它本身并不存储数据，主要的作用是决定哪一个从节点在主节点挂掉之后提升为主节点，所以客户端不需要连接此节点。

仲裁节点服务器启动，

```
./bin/mongod --dbpath arbiter/data --logpath arbiter/log/0923.log --port 27021 --fork --replSet demo
```

主服务器的配置上添加仲裁节点信息：

```
demo:PRIMARY> rs.addArb('192.168.1.168:27021')
```

## 手动设置`Primay`服务器
 将`27017`服务器重启；
 
 此时，在通过`rs.status()`查看环境

- `27017`: `SECONDARY`从服务器
- `27018`: `SECONDARY`从服务器
- `27019`: `Primary`主服务器
- `27021`: `Arbiter`仲裁服务器

如何能将当前的服务器从`27019`切换回`27017`服务器呢？

通过修改`priority`的值来实现（默认的优先级是1（0-100），`priority`的值设的越大，就优先成为主）；

在`27019`主节点上执行
```
Primary > config=rs.conf()
Primary > config.members[0].priority = 3
Primary > rs.reconfig(config)
```

注意：第2步`members`大括号里面的成员和`_id`是没有关系的，而是`rs.conf`查出来节点的数值的顺序；

# 原理
主节点的操作记录成为`oplog`（`operation log`）。 `oplog`存储在一个系统数据库`local`的集合`oplog.rs`中，这个集合的每个文档都代表主节点上执行的一个操作。我们重新向主数据库服务器中插入一条数据，然后查看这个集合可以看到：

```
use local
db.getCollection('oplog.rs').find({})
```

> 文档中的字段含义：

> - `ts`：8字节的时间戳，由4字节 `unix timestamp` + 4字节自增计数表示
> - `h`： 未知
> - `v`： 未知
> - `op`：操作类型，`i`表示`insert`；`u`表示`update`；`d`表示`delete`；`c`表示`db cmd`；`n`表示`no op`, 空操作，其会定期执行以确保时效性；
> - `ns`：操作所在的`namespace`
> - `o`：操作所对应的document,即当前操作的内容（比如更新操作时要更新的的字段和值）

从服务器会定期从主服务器中获取oplog记录，然后在本机上执行！

对于存储`oplog`的集合，`MongoDB`采用的是固定集合，也就是说随着操作过多，新的操作会覆盖旧的操作！这样做也是有道理的，不然，这个集合占用的空间就无法估算了！我们在启动服务时，可以通过选项`--oplogSize`来指定这个集合的大小，单位是`MB`，在`Windows`平台下，默认`MongoDB`会使用数据库安装分区可用空间的5%作为这个集合的大小！



# 附录
## 属性说明

|  ' | 成为`Primary` | 对客户端可见 | 参与同步 | 延迟同步 | 复制数据 |
|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|
| `Primary`    | √| √ | √|  | √|
| `Secondary`  |  | √ | √|  | √|
| `Hidden`     |  |   | √|  | √|
| `Delayed`    |  | √ | √| √| √|
| `Arbiters`   |  |   | √|  | |
| `Non-Voting` | √| √ |  |  | √|

## 参考阅读

- [DrifterJ's Stash的博客](http://blog.csdn.net/drifterj/article/details/7908926)
