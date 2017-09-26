---
title: 练习 MongoDB 操作 —— 备份篇（三）
date: 2017-09-23 17:57:41
description: 学习 MongoDB 的备份与导入导出功能
tags:
- MongoDB3.x
categories:
- NoSQL
---

# 导入与导入
导入与导出是针对集合，对集合上的文档数据经过"查询条件"后导出；
## 导出
`MongoDB`的导出是利用`mongoexport`命令；同时列举常用的参数：

- `-h`：数据库宿主机的`IP`
- `-u`：数据库用户名
- `-p`：数据库密码
- `-d`：数据库名字
- `-c`：集合的名字
- `-f`：导出的列名
- `-q`：导出数据的过滤条件
- `-o`：导出文件的目录及文件名（/xx/xx/xx.json）
- `--type`：`json` 或 `csv`（默认是 `json`）

示例：

1. 导出本地`Mongodb`服务器上`school`数据库`grade_1_5`集合上的数据（必须指定集合名）；默认导出的文件是`json`格式；

 ```
 mongoexport -d school -c grade_1_5 -o /tmp/school.json
 ```
2. 导出本地`Mongodb`服务器上`school`数据库`grade_1_5`集合上的数据（必须指定集合名），`csv`格式的文件(`csv`文件必须指定导出哪些列)

 ```
 mongoexport -d school -c grade_1_5 -o /tmp/school.csv --type=csv -f name,sex,age
 ```
 > `mongoexport`不能导出文档中的数组信息；导出 `csv` 文件的好处在于可以导入`mysql`；
 
3. 导出本地`Mongodb`服务器上`school`数据库`grade_1_5`集合上的数据（必须指定集合名）；只导出 `sex` 为 1 的文档；

  ```
  mongoexport -d school -c grade_1_5 -o /tmp/school.json -q "{"sex": 1}"
  ```

## 导入
`MongoDB`的导出是利用`mongoimport`命令；同时列举常用的参数：

- `--host`：数据库宿主机的`IP`
- `--port`：端口号
- `-d`： 待导入的数据库
- `-c`： 待导入的表
- `--type`： `json` 或 `csv`（默认是 `json`）
- `--file`： `./xx/xx.json`

示例：

1. 把之前导出的`school.csv`文件，导入到本地`Mongodb`中`school`数据库的`grade_1_6`集合

 ```
 mongoimport -d school -c grade_1_6 --file /tmp/school.json
 ```
 
# 备份与恢复
备份与恢复主要面向数据库，也可以对集合进行这类操作；

## 备份
`MongoDB`的备份是利用`mongodump`命令；同时列举常用的参数：

- `-h`：`MongDB`所在服务器地址，例如：`127.0.0.1`，当然也可以指定端口号：`127.0.0.1:27017`
- `-d`：需要备份的数据库实例，例如：`school`、`test`
- `-c`：需要备份的集合名
- `-o`：备份的数据存放位置，例如：`/home/mongodb/dump`

示例：

1. 备份本地`MongoDb`的`school`数据库，数据存放在`/home/mongodb/dump`

 ```
 mongodump -d school -o /home/mongodb/dump
 ```
2. 备份本地`MongoDb`的`school`数据库中的`grade_1_5`集合，数据存放在`/home/mongodb/dump`
 
 ```
 mongodump -d school -c grade_1_5 -o /home/mongodb/dump
 ```
## 恢复
`MongoDB`的恢复是利用`mongorestore`命令；同时列举常用的参数：

- `-h`：`MongDB`所在服务器地址，例如：`127.0.0.1`，当然也可以指定端口号：`127.0.0.1:27017`
- `-d`：需要备份的数据库实例，例如：`school`、`test`
- `-c`：需要备份的集合名
- `--drop` : 恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用!

示例：

1. 恢复本地`MongoDb`的`school`数据库中的`grade_1_5`集合

 ```
 ./bin/mongorestore -d school -c grade_1_5 /home/mongodb/dump/school/grade_1_5.bson
 ```
