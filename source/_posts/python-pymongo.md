---
title: Python 包之 pymongo
date: 2017-11-13 16:14:41
description: 记录 python 操作 mongodb 包 pymongo 的常用命令
tags:
- Python-Packages
categories:
- Python
copyright: false
---

详细命令及说明，点击[官网指南](http://api.mongodb.com/python/current/tutorial.html)；

# 常用命令
包引入
```
import pymongo
from pymongo import MongoClient
# 使用 ID 查找需要 ObjectID
from bson.objectid import ObjectId
```

## 连接
```
client= MongoClient('localhost',27017)
或
client = MongoClient('mongodb://localhost:27017/')
```



# 参考阅读

- [python操作mongodb之基础操作](https://www.cnblogs.com/similarface/p/5608987.html)




