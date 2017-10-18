---
title: PHP 扩展 Mongodb 的 FAQ
date: 2017-10-18 13:10:41
description: 记录 PHP 扩展 Mongodb 会遇到的问题
tags:
- PHP-Extensions
categories:
- PHP
copyright: false
---

# mongo.native_long 配置项
笔者在 `Laravel`框架，读取 `mongodb` 数据，其中字段是 `Int64` 的这些数据，在`php`内存中显示为浮点型，然后过滤筛选之类的就出了问题；而在测试机上部署相同的代码，一切显示正常，最后发现是 `mongo.native_long`配置项不同；

| 本地环境 | 测试机环境 |
|: --- :|: --- :|
| ![此处输入链接的描述][1] | ![此处输入链接的描述][2] |



  [1]: http://owk2q4gs5.bkt.clouddn.com/%E6%9C%AC%E5%9C%B0mongo%E7%8E%AF%E5%A2%83.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/%E6%B5%8B%E8%AF%95%E6%9C%BA%20mongo%E7%8E%AF%E5%A2%83.png
  
解决方法，在代码公共引用处添加一行 `ini_set('mongo.native_long', 1); `
这样，`php` 不会将 `mongo` 数据库中的 `Int64` 字段转为 浮点型；

## php 与 mongo 数字类型转换规律

![](http://owk2q4gs5.bkt.clouddn.com/php-extension-mongodb-conversation.png)

参考阅读

- [64-bit integers in MongoDB](https://derickrethans.nl/64bit-ints-in-mongodb.html)