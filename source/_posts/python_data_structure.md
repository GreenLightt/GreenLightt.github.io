---
title: Python 数据结构
date: 2017-09-29 23:55:41
description: 总结 Python 数据结构
tags:
categories:
- Python
copyright: false
---

**各数据结构与字符串之间的转换**

**列表**
```
fruit_list = ['apple','banana','orange']
fruit_str = str(fruit_list) # 列表转字符串
fruit_list = eval(fruit_str) # 字符串转列表
```

**元组**
```
fruits = ('apple','banana','orange') 
fruit_str = fruits.__str__()  # 元组转字符串
fruits = eval(fruit_str) # 字符串转元组
```

**集合**
```
set1 = set(['apple', 'orange', 'apple', 'pear', 'orange', 'banana'])
set1_str = str(set1) # 集合转字符串
set1 = eval(set1_str) # 字符串转集合
```

**字典**
```
defalut_str = "{'a':1 ,'b':2}"
dict1 = eval(defalut_str) # 字符串转字典
dict1_str = str(dict1)  # 字典转字符串
```

![此处输入链接的描述][1]

# 列表

## 初始化
逗号分隔值，方括号列表。一个列表中的项不一定是同个数据类型；
```
list = ['physics', 'chemistry', 1997, 2000];
```
定义空的 `list`
```
list = [];
```

## 增

| 描述 | 运算 |
|: --- :|: --- :|
|添加元素到指定下标位置 |`list.insert(index, item)`|
|添加元素到末尾 | `list.append(item)` 相当于 `a[len(list):] = [item]`|
|添加一个列表 | `list.extend(NList)` 相当于 `a[len(list):] = NList`|

## 删

| 描述 | 运算 |
|: --- :|: --- :|
|删除列表末尾的元素 |`list.pop()`|
|删除列表指定下标位置的元素 |`list.pop(index)` 相当于 `del list[index]`|
|**VE** 删除链表中值为 `x` 的第一个元素 |`list.remove(x)`|

## 改

| 描述 | 运算 |
|: --- :|: --- :|
|修改列表指定下标位置的元素| `list[index] = item`|

## 查

| 描述 | 运算 |
|: --- :|: --- :|
|读取列表中第一个元素| `list[0]`|
|读取列表中第最后一个元素| `list[-1]`|
|读取列表中第二个到第四个元素| `list[1:5]`|
|**VE** 返回链表中第一个值为 `x` 的元素的索引 |`list.index(x)`|
|**V** 返回 `x` 在链表中出现的次数| `list.count(x)`|
|获取元素个数| `len(list)`|
|**V** 获取元素最大值| `max(list)`|
|**V** 获取元素最小值| `min(list)`|


## 排序

| 描述 | 运算 |
|: --- :|: --- :|
|原列表进行升序| `list.sort()`|
|原列表进行降序| `list.sort(reverse=True)`|
|反向列表中的元素| `list.reverse()`|
|不影响原列表，返回升序列表| `sorted(list)`|
|不影响原列表，返回降序列表| `sorted(list, reverse=True)`|

## 比较

| 描述 | 运算 |
|: --- :|: --- :|
|比较两个列表的大小| `cmp(list1, list2)`|

# 元组

## 初始化

元组中只包含一个元素时，需要在元素后面添加逗号来消除歧义
```
tuple1 = ('physics', 'chemistry', 1997, 2000);
tuple2 = (1, 2, 3, 4, 5 );
tuple3 = "a", "b", "c", "d";
tuple4 = ();
tuple5 = (50,);
```
## 删

删除整个元组

```
tup = ('physics', 'chemistry', 1997, 2000);

del tup;
```

## 改

元组中的元素值是不允许修改的，但可以对元组进行连接组合

```
tuple6 = (12, 34.56) + ('abc', 'xyz');
```

## 查
| 描述 | 运算 |
|: --- :|: --- :|
|读取元组中第一个元素 |`tuple1[0]`|
|读取元组中第最后一个元素| `tuple1[-1]`|
|读取元组中第二个到第四个元素 |`tuple1[1:5]`|
|获取元素个数| `len(tuple)`|
|获取元素最大值| `max(tuple)`|
|获取元素最小值| `min(tuple)`|

## 比较
| 描述 | 运算 |
|: --- :|: --- :|
|比较两个元组的大小| `cmp(tuple1, tuple2)`|


# 集合

## 初始化
集合是一个无序不重复元素的集; 大括号或 `set()` 函数可以用来创建集合。注意：想要创建空集合，你必须使用 `set()` 而不是 `{}`

```
set1 = set(['apple', 'orange', 'apple', 'pear', 'orange', 'banana'])

set2 = {'apple', 'orange', 'banana', 'pear'}
```

## 增
| 描述 | 运算 |
|: --- :|: --- :|
|增加一个元素| `set1.add(item1)`|
|增加多个元素| `set1.update(item1, item2, item3)`|

## 删
| 描述 | 运算 |
|: --- :|: --- :|
|清空集合 |`set1.clear()`|
|删除一个元素| `set1.discard(item1)`|
|**E** 删除一个元素| `set1.remove(item1)`|
|**E** 随机删除并返回一个元素| `set.pop()`|

## 查
| 描述 | 运算 |
|: --- :|: --- :|
|查看集合的长度| `len(set1)`|

## 其他操作

| 描述 | 运算 |
|: --- :|: --- :|
|`t` 和 `s` 的并集 |`t` &#124; `s` 或 `s.union(t)  `|
|`t` 和 `s` 的交集 |`t & s  ` 或 `s.intersection(t) `|
|差集（项在 `t` 中，但不在 `s` 中）| `t – s` 或 `t.difference(s)  `|
|对称差集（项在 `t` 或 `s` 中，但不会同时出现在二者中）|`t ^ s` 或 `s.symmetric_difference(t)  `|
| `s` 是 `t` 的子集 | `s <= t` 或 `s.issubset(t) ` 或 `t.issuperset(s) ` |


# 字典
## 初始化
```
dict1 = {};

dict2 = {'jack': 4098, 'sape': 4139};

dict3 = dict([('sape', 4139), ('guido', 4127), ('jack', 4098)]);
dict4 = dict(sape=4139, guido=4127, jack=4098);
```
## 增
| 描述 | 操作 |
|: --- :|: --- :|
|添加另一个字典的内容 |`dict1.update(dict2)`|
|添加键值对 |`dict[key] = value`|

## 删
| 描述 | 操作 |
|: --- :|: --- :|
|随机返回并删除字典中的一对键和值 |`dict.popitem()`|
|删除字典给定键 `key` 所对应的值，返回值为被删除的值。<br>`key`值必须给出。否则，返回`default`值 |`dict.pop(key[, default])`|
|根据键名删除元素| `del dict1[key]`|
|清空字典 |`dict1.clear()`|

## 改
| 描述 | 操作 |
|: --- :|: --- :|
|修改指定键名的值| `dict[key] = value`|

## 查
| 描述 | 操作 |
|: --- :|: --- :|
|**E** 根据键名访问键值 |`dict[key]` 或 `dict.get(key, default=None)`|
|判断字典是否存在键名 |`dict.has_key(key)`|
|以列表返回所有的键名 |`dict.keys()`|
|以列表返回所有的键值 |`dict.values()`|
|查看字典的数目 |`len(dict)`|

## 比较
| 描述 | 操作 |
|: --- :|: --- :|
| 比较两个字典的大小 | `cmp(dict1, dict2)` |


  [1]: http://owk2q4gs5.bkt.clouddn.com/python_data.png
