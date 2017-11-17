---
title: Python 包之pymysql 
date: 2017-11-17 00:14:41
description: 记录 python 操作 mysql 包 pymysql 的常用命令
tags:
- Python-Packages
categories:
- Python
copyright: false
---

详细文档见[官方](http://pymysql.readthedocs.io/en/latest/)；

# 安装
```
pip install pymysql
```

# 使用
## 执行 Mysql 语句
```
# -*- coding: UTF-8 -*-

import pymysql

# 创建连接
conn = pymysql.connect(host='192.168.1.168', port=3306, user='root', passwd='', db='test', charset='utf8')

# 创建游标
cursor = conn.cursor()

# 执行 查 SQL，并返回受影响的行数
#effect_row = cursor.execute("select * from hardware_device_infos")

# 执行 改 SQL，并返回受影响的行数
#effect_row = cursor.execute("update hardware_device_infos set number = 'DYY' where name = %s", ('电源',))

# 执行 增 SQL，并返回受影响行数, 执行多次
#effect_row = cursor.executemany("insert into hardware_device_infos(name, number) values(%s,%s)", [("显示器1","XSQ1"), ("显示器2","XSQ2")])

# 提交，不然无法保存新建或者修改的数据
conn.commit()

# 关闭游标
cursor.close()
# 关闭连接
conn.close()
```

## 获取查询数据
```
# -*- coding: UTF-8 -*-

import pymysql

# 创建连接
conn = pymysql.connect(host='192.168.1.168', port=3306, user='root', passwd='', db='test', charset='utf8')

# 创建游标
cursor = conn.cursor()

# 执行 查 SQL，并返回受影响的行数
effect_row = cursor.execute("select * from hardware_device_infos")

# 获取剩余结果的第一行数据，形如 (第一列的值，第二列的值...)
row_1 = cursor.fetchone()

# 获取剩余结果前 3 行数据 ((第一列的值，第二列的值...), (第一列的值，第二列的值...), (第一列的值，第二列的值...))
row_2 = cursor.fetchmany(3)
 
# 获取剩余结果所有数据 ((第一列的值，第二列的值...), (第一列的值，第二列的值...), (第一列的值，第二列的值...), ...)
row_3 = cursor.fetchall()

# 关闭游标
cursor.close()
# 关闭连接
conn.close()
```

1. 操作都是靠游标，在 `fetch` 数据时按照顺序进行，可以使用 `cursor.scroll(num,mode)` 来移动游标位置，如：

  ```
  cursor.scroll(1, mode='relative') # 相对当前位置移动
  cursor.scroll(2, mode='absolute') # 相对绝对位置移动
  ```
2. 默认获取的数据是元祖类型，如果想要字典类型的数据

  ```
  cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
  ```

## 获取新创建数据自增 ID
```
# -*- coding: UTF-8 -*-

import pymysql

# 创建连接
conn = pymysql.connect(host='192.168.1.168', port=3306, user='root', passwd='', db='test', charset='utf8')

# 创建游标
cursor = conn.cursor()

# 执行 增 SQL，并返回受影响行数, 执行多次
cursor.executemany("insert into hardware_device_infos(name, number) values(%s,%s)", [("显示器1","XSQ1"), ("显示器2","XSQ2")])

# 提交，不然无法保存新建或者修改的数据
conn.commit()

# 关闭游标
cursor.close()
# 关闭连接
conn.close()

# 获取最新的自增id    
print cursor.lastrowid
```

## 使用 with 简化连接过程
```
# -*- coding: UTF-8 -*-

import pymysql
import contextlib

#定义上下文管理器，连接后自动关闭连接
@contextlib.contextmanager
def mysql(host='192.168.1.168', port=3306, user='root', passwd='111111', db='test', charset='utf8'):
    # 创建连接
    conn = pymysql.connect(host=host, port=port, user=user, passwd=passwd, db=db, charset=charset)
    # 创建游标
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    try:
        yield cursor
    finally:
        # 提交，不然无法保存新建或者修改的数据
        conn.commit()
        # 关闭游标
        cursor.close()
        # 关闭连接
        conn.close()

# 执行sql
with mysql() as cursor:
    # 执行 查 SQL，并返回受影响的行数
    cursor.execute("select * from hardware_device_infos")
    # 获取第一行数据
    row_1 = cursor.fetchone()
    print row_1
```

# 参考阅读
- [Python中操作mysql的pymysql模块详解](https://www.cnblogs.com/wt11/p/6141225.html)

