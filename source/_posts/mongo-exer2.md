﻿---
title: 练习 MongoDB 操作 —— 索引篇（二） 
date: 2017-09-20 17:57:41
description: 学习 MongoDB 的索引知识
tags:
- MongoDB3.x
categories:
- NoSQL
---

本文围绕索引、游标两部分进行探索，对`MongoDB`数据库的索引部分有一个大概的了解；
 
# 索引
索引通常能够极大的提高查询的效率，如果没有索引，`MongoDB`在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构。索引，从创建的形式上可以分为`普通索引`、`复合索引`、`数组索引`、`子文档索引`；

应该了解的索引限制，

- 开销：每个索引占据一定的存储空间，在进行插入，更新和删除操作时也需要对索引进行操作。所以，如果你很少对集合进行读取操作，建议不使用索引；
- 内存（RAM）使用：由于索引是存储在内存中,你应该确保该索引的大小不超过内存的限制。如果索引的大小大于内存的限制，`MongoDB`会删除一些索引，这将导致性能下降。
- 最大范围：
    （1）集合中索引不能超过 64 个；
    （2）索引名的长度不能超过 125 个字符；
    （3）一个复合索引最多可以有 31 个字段
- 索引不能被以下的查询使用；可以调用`explain()`方法查看是否使用索引；
    （1）正则表达式及非操作符，如 `$nin`， `$not`， 等
    （2）算术运算符，如 `$mod` 等
    （3）`$where` 子句

## 操作
### 创建索引

为集合`grade_1_4`根据性别和年龄创建复合索引：

```
db.getCollection('grade_1_4').ensureIndex({"sex": 1, "age": 1})
```

会得到结果：

```
{
    "v" : 2,                  // 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本
	"key" : {
		"sex" : 1,
		"age" : 1
	},
	"name" : "sex_1_age_1",
	"ns" : "school.grade_1_4"
}
```
> 说明：基于`sex`和`age`的查询将会用到该复合索引，或者是基于`sex`的查询也会用到该索引，但是只是基于`age`的查询将不会用到该复合索引。因此可以说，如果想用到复合索引，必须在查询条件中包含复合索引中的前`N`个索引列。然而如果查询条件中的键值顺序和复合索引中的创建顺序不一致的话，`MongoDB`可以智能的帮助我们调整该顺序，以便使复合索引可以为查询所用

如果分别为性别和年龄创建索引：

```
db.getCollection('grade_1_4').ensureIndex({"sex": 1})
db.getCollection('grade_1_4').ensureIndex({"age": 1})
```

会得到这样的结果：

```
{
	"v" : 2,
	"key" : {
		"sex" : 1
	},
	"name" : "sex_1",
	"ns" : "school.grade_1_4"
},
{
	"v" : 2,
	"key" : {
    	"age" : 1
    },
    "name" : "age_1",
    "ns" : "school.grade_1_4"
}

```
    

### 查看索引
查看集合`grade_1_4`已经创建的索引规则：

```
db.getCollection('grade_1_4').getIndexes()
```

查看集合`grade_1_4`索引占用内存空间的大小，单位是字节：

```
db.getCollection('grade_1_4').totalIndexSize()
```

### 删除索引
删除集合`grade_1_4`中`name`为`sex_1`的索引规则：

```
db.getCollection('grade_1_4').dropIndex({"sex": 1})
```

## 唯一索引
创建唯一索引与普通索引的区别在于，多一个可选参数 `unique`；举个例子，根据姓名创建唯一索引：
```
db.getCollection('grade_1_4').ensureIndex({"name":1}, {"unique":true})
```

创建唯一索引能够保证每条记录的`name`字段值是不重复的；

当一个文档以唯一索引的方式保存到集合中去的时候，任何缺失的索引字段都会以`null`值代替，因此，不能在唯一索引上同时插入两条缺省的记录。

假设集合已经存在一些记录，在些基础上创建唯一索引；此时，对这些已经存在的记录，如果索引项值是存在重复的，则创建索引时会报错；如果一定要在这样的键上创建唯一索引，那么系统将保存第一条记录，剩下的记录会被删除，结合`dropDups`参数使用（此参数只能在`mongodb 3.0`版本之前使用）：

```
db.getCollection('grade_1_4').ensureIndex({"name":1}, {unique:true, dropDups: true})
```

## 稀疏索引
稀疏索引的创建和完全索引的创建没有什么不同。使用稀疏索引进行查询的时候，某些由于缺失了字段的文档记录可能不会被返回，这是由于稀疏索引子返回被索引了的字段。

示例：
```
> db.people.ensureIndex({title:1},{sparse:true}) //在title字段上建立稀疏索引
> db.people.save({name:"Jim"})
> db.people.save({name:"yang",title:"prince"})
> db.people.find();
{ "_id" : ObjectId("4e244dc5cac1e3490b9033d7"), "name" : "Jim" }
{ "_id" : ObjectId("4e244debcac1e3490b9033d8"), "name" : "yang", "title" : "prince" }

> db.people.find().sort({title:1})//自有包含有索引字段的记录才被返回
{ "_id" : ObjectId("4e244debcac1e3490b9033d8"), "name" : "yang", "title" : "prince" }

> db.people.dropIndex({title:1})//删除稀疏索引之后，所有的记录均显示
{ "nIndexesWas" : 2, "ok" : 1 }

> db.people.find().sort({title:1})
{ "_id" : ObjectId("4e244dc5cac1e3490b9033d7"), "name" : "Jim" }
{ "_id" : ObjectId("4e244debcac1e3490b9033d8"), "name" : "yang", "title" : "prince" }
```

## 性能示例
创建三十万条数据

```
for (var i = 1; i <= 300000; i++) {
    db.getCollection('grade_1_5').insert({
       "name": "zhangsan" + i,
       "sex": Math.round(Math.random() * 10) % 2,
       "age": Math.round(Math.random() * 6) + 3
   });
}
```

根据姓名查一些特定的人

```
db.getCollection('grade_1_5').find(
    {
        $or: [{"name":"zhangsan10000"},{"name":"zhangsan14000"},{"name":"zhangsan9000"},{"name":"zhangsan23000"},{"name":"zhangsan24050"},
              {"name":"zhangsan12000"},{"name":"zhangsan14300"},{"name":"zhangsan9300"},{"name":"zhangsan23300"},{"name":"zhangsan24350"},
              {"name":"zhangsan11100"},{"name":"zhangsan15200"},{"name":"zhangsan8100"},{"name":"zhangsan22100"},{"name":"zhangsan26150"},
              {"name":"zhangsan10200"},{"name":"zhangsan14020"},{"name":"zhangsan9020"},{"name":"zhangsan23020"},{"name":"zhangsan24070"},
              {"name":"zhangsan10300"},{"name":"zhangsan14030"},{"name":"zhangsan9030"},{"name":"zhangsan23030"},{"name":"zhangsan24080"}]
    }
)
```

对姓名建立索引，会占用`5873664`字节

```
db.getCollection('grade_1_5').ensureIndex({"name": 1})
```

对于上面的命令，通过调整顺序，观察时间，对性能有一些大概的了解；

| 行为 | 创建-查询-建索引 | 创建-建索引-查询 | 建索引-创建-查询 |
|: --- :|: --- :|: --- :|: --- :|
| 创建时间 | 150.957s | 150.957s| 159.967s |
| 查询时间 | 1.024s | 0.005s| 0.005s |
| 建立索引时间 | 0.527s | 0.527s | 0.009s |

# 游标
直接对一个集合调用`find()`方法时，我们会发现，如果查询结果超过二十条，只会返回二十条的结果，这是因为`Mongodb`会自动递归`find()`返回的是游标。

```
var cursor = db.getCollection('grade_1_4').find({});
```
执行上述命令时，`shell`并不会真正地访问数据库，而是等待开始要求获得结果的时候才向数据库发送查询请求。

此时可以对这个游标进行各种设置，然后调用游标的`hashNext()`或`next()`方法，这样就会真正访问数据库，这是一个懒加载的过程，如下

```
var cursor = db.getCollection('grade_1_4').find({});

while(cursor.hasNext()){  
    var doc = cursor.next();  
    // do stuff with doc  
};  
```

可以基于游标，进行**limit、skip、sort**操作，一般应用于分页场景；

- `limit`：限制游标返回的数量，指定了上限；
- `skip` ：忽略前面的部分文档，如果文档总数量小于忽略的数量，则返回空集合；
- `sort` ：对得到的子集合进行排序，可以按照多个键进行正反排序；

上述三个命令可以进行链式操作；

# 附录
以下是文章会用到的参考，及有意义的扩展阅读；

- [DrifterJ's Stash的博客 - 学习MongoDB--（4-3）：MongoDB查询(游标使用) ](http://blog.csdn.net/drifterj/article/details/7848809)
- [MongoDB使用小结：一些不常见的经验分享](http://www.cnblogs.com/cswuyg/p/4355948.html)

