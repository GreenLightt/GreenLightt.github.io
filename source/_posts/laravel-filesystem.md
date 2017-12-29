---
title: Laravel 用法之 FileSystem 模块
date: 2017-12-25 17:39:41
description: 转载 Laravel 学院的文件系统相关文章并略加精简
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

本文转载 [[ Laravel 5.1 文档 ] 服务 —— 文件系统/云存储](http://laravelacademy.org/post/200.html#ipt_kb_toc_200_8)

官方 `API` 地址 [https://laravel.com/api/5.1/Illuminate/Filesystem.html](https://laravel.com/api/5.1/Illuminate/Filesystem.html)

基于`Frank de Jonge` 的PHP包 `Flysystem`，`Laravel` 提供了强大的文件系统抽象。Laravel文件系统集成提供了使用驱动处理本地文件系统的简单使用，这些驱动包括 `Amazon S3`，以及 `Rackspace` 云存储。此外在这些存储选项间切换非常简单，因为对每个系统而言，`API` 是一样的。

# 基本使用
## 获取硬盘实例
获取默认的驱动
```
Storage::getDefaultDriver()
```
获取指定驱动，比如获取本地文件驱动
```
$disk = Storage::disk('local')
```

## 获取文件
`exists` 方法用于判断给定文件是否存在于磁盘上：

```
$exists = Storage::disk('s3')->exists('file.jpg');
```

`get`方法用于获取给定文件的内容，该方法将会返回该文件的原生字符串内容：

```
$contents = Storage::get('file.jpg');
```

`size`方法以字节方式返回文件大小：

```
$size = Storage::size('file1.jpg');
```

`lastModified` 方法以 `UNIX` 时间戳格式返回文件最后一次修改时间：

```
$time = Storage::lastModified('file1.jpg');
```


## 存储文件
`put` 方法用于存储文件到磁盘。可以传递一个`PHP`资源到`put`方法，该方法将会使用`Flysystem`底层的流支持。在处理大文件的时候推荐使用文件流：

```
Storage::put('file.jpg', $contents);
Storage::put('file.jpg', $resource);
```

`copy` 方法将磁盘中已存在的文件从一个地方拷贝到另一个地方：

```
Storage::copy('old/file1.jpg', 'new/file1.jpg');
```

`move` 方法将磁盘中已存在的文件从一定地方移到到另一个地方：

```
Storage::move('old/file1.jpg', 'new/file1.jpg');
```

`prepend` 和 `append` 方法允许你轻松插入内容到文件开头/结尾：

```
Storage::prepend('file.log', 'Prepended Text');
Storage::append('file.log', 'Appended Text');
```



## 删除文件
`delete` 方法接收单个文件名或多个文件数组并将其从磁盘移除：

```
Storage::delete('file.jpg');
Storage::delete(['file1.jpg', 'file2.jpg']);
```

## 目录
**获取一个目录下的所有文件**
`files` 方法返回给定目录下的所有文件数组，如果你想要获取给定目录下包含子目录的所有文件列表，可以使用 `allFiles` 方法：

```
$files = Storage::files($directory);
$files = Storage::allFiles($directory);
```

**获取一个目录下的所有子目录**
`directories` 方法返回给定目录下所有目录数组，此外，可以使用 `allDirectories` 方法获取嵌套的所有子目录数组：

```
$directories = Storage::directories($directory);
// 递归...
$directories = Storage::allDirectories($directory);
```

**创建目录**
`makeDirectory` 方法将会创建给定目录，包含子目录（递归）：

```
Storage::makeDirectory($directory);
```

**删除目录**
`deleteDirectory` 方法用于移除目录，包括该目录下的所有文件

```
Storage::deleteDirectory($directory);
```





