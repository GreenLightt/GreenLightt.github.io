---
title: Mysql 慢查询
date: 2018-01-21 21:31:41
description: 整理 Mysql 慢查询知识点
tags:
- Mysql
categories:
---


# 简介

开启慢查询日志，可以让 `MySQL` 记录下查询超过指定时间的语句，通过定位分析性能的瓶颈，才能更好的优化数据库系统的性能。

# 参数说明

| 字段 | 描述  |
|: --- :|: --- :|
| `slow_query_log`  | 慢查询开启状态 |
| `slow_query_log_file`|  慢查询日志存放的位置（这个目录需要 `MySQL` 的运行帐号的可写权限，一般设置为 `MySQL` 的数据存放目录） | 
| `long_query_time` | 查询超过多少秒才记录| 

# 命令

## 查看

查看慢查询相关参数

```
show variables like 'slow_query%';

+---------------------------+----------------------------------+
| Variable_name             | Value                            |
+---------------------------+----------------------------------+
| slow_query_log            | OFF                              |
| slow_query_log_file       | /mysql/data/localhost-slow.log   |
+---------------------------+----------------------------------+

mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

## 设置

### 变量设置

```
# 将 slow_query_log 全局变量设置为“ON”状态
mysql> set global slow_query_log='ON';

# 设置慢查询日志存放的位置
mysql> set global slow_query_log_file='/usr/local/mysql/data/slow.log';

# 查询超过1秒就记录
mysql> set global long_query_time=1;
```

### 配置设置

修改配置文件 `my.cnf`，在 `[mysqld]` 下的下方加入

```
[mysqld]
slow_query_log = ON
slow_query_log_file = /usr/local/mysql/data/slow.log
long_query_time = 1
```

重启 `mysql` 服务器


## 测试

执行一条慢查询 `SQL` 语句

```
mysql> select sleep(2);
```

查看是否生成慢查询日志

```
ls /usr/local/mysql/data/slow.log
```



本文转自 [成九云笔记 - MySQL慢查询（一） - 开启慢查询](http://www.cnblogs.com/luyucheng/p/6265594.html)

