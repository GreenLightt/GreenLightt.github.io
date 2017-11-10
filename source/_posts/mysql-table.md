---
title: Mysql 数据库和表操作
date: 2017-11-10 17:01:41
description: 整理 Mysql 数据库和表操作命令
tags:
- Mysql
categories:
---

# 表
## 增
示例 `demo`

```
CREATE TABLE `【表名字】` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `dot_id` int(11) NOT NULL DEFAULT '2',
  `uid` int(11) NOT NULL,
  `shelf_id` int(11) NOT NULL,
  `shelf_code` varchar(16) NOT NULL,
  `floor_id` int(11) NOT NULL,
  `shelf_trading_id` varchar(25) NOT NULL,
  `type` int(2) NOT NULL DEFAULT '1',
  `default_num` int(11) DEFAULT '10',
  `relative_num` int(11) NOT NULL DEFAULT '0',
  `fill_num` int(11) NOT NULL DEFAULT '0' COMMENT '填充数量',
  `timestamp_flag` varchar(12) DEFAULT NULL,
  `created_at` datetime DEFAULT CURRENT_TIMESTAMP,
  `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

如果有索引，在创建表的语法里写法如下：
```
KEY `INDEX_SHELF_TRADINGID_SHELFCODE` (`shelf_code`,`shelf_trading_id`)
```
如果是唯一索引：
```
UNIQUE KEY `INDEX_UNIQUE` (`id`)
```

## 删
删除表结构及数据：

```
DROP TABLE IF EXISTS `【表名字】`;
```

不删除表结构，只清空数据：

```
DELETE FROM 【表名字】;
# 或
TRUNCATE TABLE 【表名字】;
```
效率上 `truncate` 比 `delete` 快，但 `truncate` 删除后不记录 `mysql` 日志，不可以恢复数据。

## 改
### 删除列

```
ALTER TABLE 【表名字】 DROP 【列名称】
```

### 增加列
新增列
```
ALTER TABLE 【表名字】 ADD 【列名称】 INT
# 等价于
ALTER TABLE shelf_deal_log ADD system_num INT DEFAULT NULL;
```
新增列，非空，并带说明
```
ALTER TABLE 【表名字】 ADD 【列名称】 INT NOT NULL  COMMENT '注释说明'
```

新增一个 `datetime` 列，设置默认值为创建时的当前时间；
```
ALTER TABLE 【表名字】 ADD 【列名称】 DATETIME DEFAULT CURRENT_TIMESTAMP;
```

新增一个 `datetime` 列，设置默认值为修改时的当前时间，修改时时间会更新；

```
ALTER TABLE 【表名字】 ADD 【列名称】 DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;
```


### 修改列的类型信息
```
ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称（这里可以用和原来列同名即可）】 BIGINT NOT NULL  COMMENT '注释说明'
```
### 重命名列
```
ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称】 BIGINT NOT NULL  COMMENT '注释说明'
```

### 重命名表

```
ALTER TABLE 【表名字】 RENAME 【表新名字】
```

### 删除表中主键
```
Alter TABLE 【表名字】 drop primary key
```

### 替换主键

```
ALTER TABLE 【表名字】 DROP PRIMARY KEY ,ADD PRIMARY KEY (【新主键名称，有多个，逗号隔开】);
```

### 添加索引
```
ALTER TABLE 【表名字】 ADD INDEX 【索引名】 (【索引字段，有多个，逗号隔开】);
```

### 添加唯一限制条件索引
```
ALTER TABLE 【表名字】 ADD UNIQUE 【索引名】 (【索引字段】);
```

### 删除索引
```
ALTER TABLE 【表名字】 DROP INDEX 【索引名】;
```

## 查
查看当前数据库下所有的表：

```
SHOW TABLES;
```

查询表的字段信息：

```
DESC 【表名字】;
```

查看表字段详细信息；

```
SHOW FULL COLUMNS FROM 	【表名字】;
```


# 数据库
查找当前服务器上存在哪些数据库；

```
SHOW DATABASES;
```

使用指定数据库，比如使用 `antenna` 数据库；

```
USE antenna;
```

创建数据库，比如新创建的数据库名称叫 `hello`;

```
CREATE DATABASE hello;
```




