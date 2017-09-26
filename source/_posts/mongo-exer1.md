﻿---
title: 练习 MongoDB 操作 —— 数据操作（一）
date: 2017-09-14 17:57:41
description: 通过实例 Demo 学习数据的增删改查操作
tags:
- MongoDB3.x
categories:
- NoSQL
---

本文的目标是通过大量的示例，来更好的理解如果在`Mongodb`中进行数据操作；

**初入客户端**
刚利用 `mongod`命令进入客户端环境，此时对数据库一无所知；

举目四望，想知道现在有哪些数据库，
```
show dbs;
```

因为是新装的`mongodb`环境，所以只看到了`admin`和`local`两个默认就存在的数据库；目光慢慢收回，那么当前是处于哪个数据库上呢？
```
db;
```
通过上述这个命令，不仅可以知道当前在哪个数据库上；
现在切换到`admin`数据库上，转一圈；
```
use admin;
```

**数据库**
这时候，笔者想要创建自己应用的数据库`school`, 用来存放一些班级学生信息；
```
use school;
```
`use`命令：如果数据库不存在，则创建数据库，否则切换到指定数据库；

突然发现刚才敲命令，写错了，写成了`use school1`；这时候，希望删除`school1`这个数据库，就切换到该数据库下，再键入删除命令；
```
use school1;
db.dropDatabase();
```

**集合**
`Mongodb`中的集合相当于`Mysql`中的表；

作为一名优秀的“校长”，能适应高信息化社会发展，笔者需要为学校下的各个年级、班级建立集合；创建集合可以是显式的，也可以是隐式的；

通过`show tables`，看到数据库下没有任何集合；笔者显式地创建“一年级一班的”集合；
```
db.createCollection("grade_1_1");
```
再次通过`show tables`就可以看到列表中有`grade_1_1`这个集合；

当然，也可以隐式地创建，当为集合插入数据，集合不存在，这时候集合会自动创建；现在，不存在`grade_1_2`“一年级二班”这个集合，执行下面语句，为“一年级二班”加入一个学生；
```
db.grade_1_2.insert({"name": 'zhangsan', "age": '7', "sex": "0"});
```
通过`show tables`就可以看到`grade_1_2`这个集合了；

因为一些特殊原因，要解散一年级二班，那笔者这儿就不用继续维护`grade_1_2`集合，

```
db.grade_1_2.drop();
```

## 练习增查


1. 清空上面的`school`数据库

 ```
 use school;
 db.dropDatabase();
 use school;
 show tables;
 ```
2. 创建一年级的3个班，并随机添加 10 名学生；

 ```
    for(grade_index in (grade = ['grade_1_1', 'grade_1_2', 'grade_1_3'])) {
        for (var i = 1; i <= 10; i++) {
            db[grade[grade_index]].insert({
                "name": "zhangsan" + i,
                "sex": Math.round(Math.random() * 10) % 2,
                "age": Math.round(Math.random() * 6) + 3,
                "hobby": []
            });
        }
    }
 ```
3. 查看一年级二班`grade_1_2`中的所有学生

 ```
 db.getCollection('grade_1_2').find({})
 ```
 
4. 查看一年级二班`grade_1_2`中所有年龄是 4 岁的学生

     ```
     db.getCollection('grade_1_2').find({"age": 4})
     ```
    查看一年级二班`grade_1_2`中所有年龄大于 4 岁的学生

    ```
    db.getCollection('grade_1_2').find({"age": {$gt: 4}})
    ```
    查看一年级二班`grade_1_2`中所有年龄大于 4 岁并且小于 7 岁的学生
 ```
 db.getCollection('grade_1_2').find({"age": {$gt: 4, $lt: 7}})
 ```
    查看一年级二班`grade_1_2`中所有年龄大于 4 岁并且性别值为`0`的学生
 ```
 db.getCollection('grade_1_2').find({"age": {$gt: 4}, "sex": 0})
 ```
 查看一年级二班`grade_1_2`中所有年龄小于 4 岁并且大于 7 岁的学生
 ```
 db.getCollection('grade_1_2').find({$or: [{"age": {$lt: 4}}, {"age": {$gt: 6}}]})
 ```
 
5. 查看一年级二班`grade_1_2`中所有年龄是 4 岁或 6 岁的学生

 ```
 db.getCollection('grade_1_2').find({"age": {$in: [4, 6]}})
 ```

6. 查看一年级二班`grade_1_2`中所有姓名带`zhangsan1`的学生

 ```
 db.getCollection('grade_1_2').find({"name": {$regex: "zhangsan1"}})
 ```
    查看一年级二班`grade_1_2`中所有姓名带`zhangsan1`和`zhangsan2`的学生
 ```
 db.getCollection('grade_1_2').find({"name": {
    $in: [new RegExp(""zhangsan1"), new RegExp(""zhangsan2")]
 }})
 ```

7. 查看一年级二班`grade_1_2`中所有兴趣爱好有三项的学生

 ```
 db.getCollection('grade_1_2').find({"hobby": {$size: 3}})
 ```
     查看一年级二班`grade_1_2`中所有兴趣爱好包括画画的学生
 
 ```
 db.getCollection('grade_1_2').find({"hobby": "drawing"})
 ```
     查看一年级二班`grade_1_2`中所有兴趣爱好既包括画画又包括跳舞的学生
 
 ```
 db.getCollection('grade_1_2').find({"hobby": {$all: ["drawing", "dance"]}})
 ```
8. 查看一年级二班`grade_1_2`中所有兴趣爱好有三项的学生的学生数目
 
 ```
 db.getCollection('grade_1_2').find({"hobby": {$size: 3}}).count()
 ```
 
9. 查看一年级二班的第二位学生

 ```
 db.getCollection('grade_1_2').find({}).limit(1).skip(1)
 ```
10. 查看一年级二班的学生，按年纪升序
 
 ```
 db.getCollection('grade_1_2').find({}).sort({"age": 1})
 ```
     查看一年级二班的学生，按年纪降序
 
 ```
 db.getCollection('grade_1_2').find({}).sort({"age": -1})
 ```
 
11. 查看一年级二班的学生，年龄值有哪些

 ```
 db.getCollection('grade_1_2').distinct('age')
 ```
     查看一年级二班的学生，兴趣覆盖范围有哪些
 
 ```
 db.getCollection('grade_1_2').distinct('hobby')
 ```
     查看一年级二班的学生，男生（`sex`为 0）年龄值有哪些
 
 ```
  db.getCollection('grade_1_2').distinct('age', {"sex": 0})
 ```

## 练习删除
1. 一年级二班`grade_1_2`， 删除所有 4 岁的学生
 
 ```
 db.getCollection('grade_1_2').remove({"age": 4})
 ```
 
2. 一年级二班`grade_1_2`， 删除第一位 6 岁的学生
 
 ```
 db.getCollection('grade_1_2').remove({"age": 6}, {justOne: 1})
 ```
 

## 练习修改
1. 一年级二班`grade_1_2`中，修改名为`zhangsan7`的学生，年龄为 8 岁，兴趣爱好为 跳舞和画画；

 ```
 db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$set: {"age": 8, "hobby": ["dance", "drawing"]}})
 ```
     一年级二班`grade_1_2`中，追加zhangsan7`学生兴趣爱好唱歌；

 ```
 db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$push: {"hobby": "sing"}})
 ```
     一年级二班`grade_1_2`中，追加zhangsan7`学生兴趣爱好吹牛和打篮球；
 
 ```
 db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$push: {"hobby": {$each: ["brag", "play_basketball"]}}})
 
 ```
     一年级二班`grade_1_2`中，追加`zhangsan7`学生兴趣爱好唱歌和打篮球，要保证`hobby`数组不重复；
 
 ```
  db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$addToSet: {"hobby": {$each: ["sing1", "play_basketball"]}}})
 ```
 
3. 新学年，给一年级二班所有学生的年龄都增加一岁

 ```
  db.getCollection('grade_1_2').update({}, {$inc: {"age": 1}}, {multi: true})
 ```
4. 一年级二班`grade_1_2`中，删除`zhangsan7`学生的`sex`属性

 ```
  db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$unset: {"sex": 1}})
 ```
5. 一年级二班`grade_1_2`中，删除`zhangsan7`学生的`hobby`数组中的头元素

 ```
  db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$pop: {"hobby": -1}})
 ```
     一年级二班`grade_1_2`中，删除`zhangsan7`学生的`hobby`数组中的尾元素
 
 ```
  db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$pop: {"hobby": 1}})
 ```
     一年级二班`grade_1_2`中，删除`zhangsan7`学生的`hobby`数组中的`sing`元素
 
 ```
  db.getCollection('grade_1_2').update({"name": "zhangsan7"}, {$pull: {"hobby": "sing"}})
 ```
 
## 练习分组
新建一个集合`grade_1_4`，记录一年级四班在期中考试时的成绩；
```
    for (var i = 1; i <= 10; i++) {
        db.grade_1_4.insert({
            "name": "zhangsan" + i,
            "sex": Math.round(Math.random() * 10) % 2,
            "age": Math.round(Math.random() * 6) + 3,
            "score": {
	            "chinese": 60 + Math.round(Math.random() * 40),
		        "math": 60 + Math.round(Math.random() * 40),
		        "english": 60 + Math.round(Math.random() * 40)
		    }
        });
    }

```

1. 统计每名学生在考试中的总分

 ```
    db.grade_1_4.group({
        key: {"name": 1},
        cond: {},
        reduce: function(curr, result) {
            result.total += curr.score.chinese + curr.score.math + curr.score.english;
        },
        initial: { total : 0 }
    })
 ```
 
2. 统计每名男生在考试中的总分

 ```
    db.grade_1_4.group({
        key: {"name": 1},
        cond: {"sex": 0},
        reduce: function(curr, result) {
            result.total += curr.score.chinese + curr.score.math + curr.score.english;
        },
        initial: { total : 0 }
    })
 ```
 
3. 统计每名男生在考试中的总分及平均分

 ```
    db.grade_1_4.group({
        key: {"name": 1},
        cond: {"sex": 0},
        reduce: function(curr, result) {
            result.total += curr.score.chinese + curr.score.math + curr.score.english;
        },
        initial: { total : 0 },
        finalize: function(item) {
            item.avg = (item.total / 3).toFixed(2);
            return item;
        }
    })
 ```
 
## 练习聚合
1. 根据姓名分组， 并统计人数

 ```
    db.getCollection('grade_1_4').aggregate([
        {$group: {_id: "$name", num: {$sum: 1}}}
    ])
 ```
     根据姓名分组， 并统计人数，过滤人数大于 1 的学生
 
 ```
    db.getCollection('grade_1_4').aggregate([
        {$group: {_id: "$name", num: {$sum: 1}}}, 
        {$match: {num: {$gt: 1}}}
    ])
 ```
 
2. 统计每名学生在考试中的总分

 ```
    db.getCollection('grade_1_4').aggregate([
        {$group: {_id: "$name", score: {$sum: {$sum: ["$score.chinese", "$score.math", "$score.english"]}}}}
])
 ```

2. 统计每名男生在考试中的总分

 ```
 db.getCollection('grade_1_4').aggregate([
    {$match: {sex: 0}},
    {$group: {_id: "$name", score: {$sum: {$sum: ["$score.chinese", "$score.math", "$score.english"]}}}}
])
 ```
    统计每名男生在考试中的总分， 总分降序
 
 ```
    db.getCollection('grade_1_4').aggregate([
        {$match: {sex: 0}},
        {$group: {_id: "$name", score: {$sum: {$sum: ["$score.chinese", "$score.math", "$score.english"]}}}},
        {$sort: {score: 1}}
])
 ```

 
## 练习权限

### 创建用户
> 要让权限生效，需要`mongo`服务器启动时添加`--auth`选项；

创建用户`school_admin`，只能对`school`数据库进行读写操作；

```
use school;

db.createUser({
    user: "school_admin",
    pwd: "school_admin",
    roles: [{role: "readWrite", db: "school"}]
})
```

关于第三个参数角色，看下表：

| 角色名 | 描述 |
|: --- :|: --- :|
| Read | 允许用户读取指定数据库 |
| readWrite | 允许用户读写指定数据库 |
| dbAdmin | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问`system.profile` |
| userAdmin | 允许用户向`system.users`集合写入，可以找指定数据库里创建、删除和管理用户 |
| clusterAdmin | 只在`admin`数据库中可用，赋予用户所有分片和复制集相关函数的管理权限 |
| readAnyDatabase | 只在`admin`数据库中可用，赋予用户所有数据库的读权限 |
| readWriteAnyDatabase | 只在`admin`数据库中可用，赋予用户所有数据库的读写权限 |
| userAdminAnyDatabase  | 只在`admin`数据库中可用，赋予用户所有数据库的`userAdmin`权限 |
| dbAdminAnyDatabase  | 只在`admin`数据库中可用，赋予用户所有数据库的`dbAdmin`权限 |
| root | 只在`admin`数据库中可用。超级账号，超级权限 |


### 验证用户
如果未通过验证，进行查询，

会得到如下的提示：
```
Error: error: {
	"ok" : 0,
	"errmsg" : "not authorized on school to execute command { find: \"grade_1_2\", filter: {} }",
	"code" : 13,
	"codeName" : "Unauthorized"
}
```

如果执行验证代码：**注意，要在注册时所在的数据库中验证**

```
use school;

db.auth('用户名', '密码')
```

### 查看所有用户
```
db.getUsers()
```

### 删除用户
先移到用户注册的数据库，然后移除指定用户 `school_admin`
```
db.dropUser('school_admin')
```

> 关于用户的注册位置，笔者的理解是在`admin`下创建其他数据库的管理者，再由这些管理者在对应的数据库下创建“可读”或“可写”的用户；
