﻿---
title: 练习 MongoDB 操作 —— 分片篇（五）
date: 2017-09-25 17:57:41
description: 学习 MongoDB 的分片功能
tags:
- MongoDB3.x
categories:
- NoSQL
---

分片（`sharding`）是指将数据拆分，将其分散存在不同的机器上的过程。在关系型数据库中，当一个表太大（超过几亿行数据）时，我们也有分表的做法，和这里的分片是类似的概念。

# 术语

- “片”：一个独立的`MongoDB`服务（即`mongod`服务进程，在开发测试环境中）或一个副本集（在生产环境中）。
- “片键”：在路由服务器上设置分片时，需要从开启分片的集合中选取一个键，用该键作为数据存放在哪个片的依据。这个键就称为“片键”。对于选择哪个键作为片键？有个原则就是，片键应该有较多变化的值。
- “配置服务器”：一个`mongod`服务进程，但这个数据库服务仅仅是为`mongos`路由服务提供配置信息存储的位置！启动`mongos`服务时，需要提供一个`mongod`服务器，以便路由服务访问或存储相关的配置信息。配置信息主要包括：分片与数据的对应关系！（配置服务器必须开启1个或则3个，开启2个则会报错（
- “`mongos`”服务：`MongoDB`自带的路由服务进程，它路由所有的客户端请求，并将各个片的结果进行汇聚返回。这个服务进程本身不会存储任何数据或配置信息（有时会缓存配置服务器的相关配置信息）。


# 示例
## 环境搭建
启动两台`mongodb`服务器（`27017`和`27018`），一台配置服务器（`27019`）和一个路由服务（`27020`）；

注意： `Mongodb3.4`版本开始要求配置服务器要是复制集

```
// 启动分片服务器，注意要加 --shardsvr
./bin/mongod --dbpath shard1/data --logpath shard1/log/0924.log --fork --port 27017 --shardsvr
./bin/mongod --dbpath shard2/data --logpath shard2/log/0924.log --fork --port 27018 --shardsvr

// 配置服务器， 注意要加 --configsvr
./bin/mongod --dbpath shard_router_server/data --logpath shard_router_server/log/0924.log --fork --port 27019 --configsvr --replSet config

// 启动路由服务
./bin/mongos --port 27020 --configdb config/localhost:27019 --logpath shard_router/log/0924.log --fork
```

进入路由服务，注册参与分片的节点

```
mongos> use admin
mongos> db.runCommand({"addshard" : "192.168.1.168:27017"});
mongos> db.runCommand({"addshard" : "192.168.1.168:27018"});
```

**查看分片信息**
```
mongos> sh.status()
```

**查看分片成员**
```
mongos> db.runCommand({listshards: 1})
// 或切换到 config 数据库，查看 shards 集合
mongos> use config
mongos> db.shards.find()
```

**删除分片**
```
mongos> db.runCommand({"removeshard":"192.168.1.168:27017"})
```
此时后台会自动开启分片数据库的数据迁移；然后需要手动迁移未开启分片数据库的数据；

手动迁移命令，可以参考
```
mongos> db.runCommand({ movePrimary: "school", to: "192.168.1.168:27018" })
```

## 开启分片
如果对应的数据库没有开启分片功能，则通过路由服务新增数据，是会报错的；

笔者开启`school`数据库的分片功能；
```
mongos> db.runCommand({"enablesharding" : "school"});
// 或
mongos> sh.enableSharding("school")
```
那么，如果 `school` 数据库下的集合没有开启分片功能，那么所有对`school`数据库下集合的操作都会在`school`数据库所在的分片上执行；

查看已经在片系统上的数据库列表：
```
mongos> db.databases.find()
```

**集合开启分片**
`school`数据库下的`demo`集合开启分片功能；
```
mongos> sh.shardCollection("school.demo", {"name":1})
```

开启分片的集合，是以 `chunks`（块） 为单位；块（`chunk`）是由多个`doc`组成的一个分组，在某个索引字段（片键）上是连续的，每个`chunk`的片键是有一定范围的。块的默认大小是 64 `MB`。有些chunk会非常大，包含的`doc`数量非常多，但是，在`MongoDB`看来，仍然是一个`chunk`，和没有任何`doc`的空`chunk`没有区别。均衡分发保证每个`shard`的`chunk`数量是大致相同的。因此，片键的选择直接影响分片的好坏。

## 测试分片效果
因为`chunk`默认大小是 64 `MB`（取值范围是 1 `MB` 到 1024 `MB`）， 不方便查看效果（`chunk`数目差异较大时的拆分与平衡）；

修改`chunk`块的大小：
```
mongos> use config
mongos> db.settings.save( { _id:"chunksize", value: 1 } )
```

此时批量向`school`数据库下的 `demo`插入 60W 条数据
```
mongos> use school
mongos> for(i=0;i<600000;i++){ db.demo.insert({"uid":i,"name":"zhanjindong"+i,"age":13,"data":new Date()}); }
```
通过命令`sh.status()`，可以实时查看各片上的`chunk`数目多少；

# 附录

## 参考阅读

- [悦光阴的博客](http://www.cnblogs.com/ljhdo/p/5016193.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [DrifterJ's Stash的博客](http://blog.csdn.net/drifterj/article/details/7934231)
