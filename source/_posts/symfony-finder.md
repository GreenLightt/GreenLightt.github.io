---
title: Symfony 组件之 Finder
date: 2017-11-27 00:12:41
description: 总结 Finder 的 API 及用法
tags:
- Symfony
categories:
copyright: false
---

`Finder` 组件用来遍历文件或目录；

[github的仓库地址链接](https://github.com/symfony/finder/tree/2.0)

# Method
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| `create` |  | 创建新的 `Finder` 对象  |
| `directories` | | 约束只匹配目录 |
| `files` | | 约束只匹配文件 |
| `depth` | `$level`| 针对目录或文件的目录范围或深度(0, 1, 2...) |
| `date` | `$date` | 针对文件的修改时间的约束<br/>比如 `'>= 2005-10-15'`、`since yesterday` |
| `name` | `$pattern` | 针对文件名的匹配约束<br/>比如 `'*.php'`、`'/\.php$/'` |
| `notName` | `$pattern` | 针对文件名的不匹配约束<br/>比如 `'*.php'`、`'/\.php$/'`|
| `contains` | `$pattern` | 针对文件内容的匹配约束<br/>比如 `'Lorem ipsum'`、`'/Lorem ipsum/i'` |
| `notContains` | `$pattern` | 针对文件内容的不匹配约束<br/>比如 `'Lorem ipsum'`、`'/Lorem ipsum/i'` |
| `path` | `$pattern` | 针对文件名路径的匹配约束<br/>比如 `'some/special/dir'` |
| `notPath` | `$pattern` | 针对文件名路径的不匹配约束<br/>比如 `'some/special/dir'` |
| `size` | `$size` | 针对文件大小的匹配约束<br/>比如 `'> 10K'`、`'<= 1Ki'`|
| `exclude` | `$dirs` | 排除掉哪些目录 |
| `ignoreDotFiles` | `$ignoreDotFiles` | 参数决定是否忽略隐藏文件（即以 `.` 开始的文件）|
| `ignoreVCS` | `$ignoreVCS` | 布尔参数决定是否忽略`.svn`、`_svn`、<br/>`CVS`、`_darcs`、` .arch-params`、`.monotone`、`.bzr`、`.git`、`.hg` 这些 `VCS` 格式|
| `addVCSPattern` | `$pattern`| 添加 `VCS` 模式类别，参数为数组 |
| `sort` | `\Closure $closure` | 根据匿名函数排序 |
| `sortByName` | | 根据文件名或目录名排序 |
| `sortByType` | | 根据文件类型排序 |
| `sortByAccessedTime` | | 根据文件的访问时间排序|
| `sortByChangedTime` | | 根据文件`inode`的修改时间排序 |
| `sortByModifiedTime` | | 根据文件的修改时间排序 |
| `filter` | | 根据匿名函数筛选 |
| `followLinks` | | `Forces the following of symlinks.` |
| `ignoreUnreadableDirs` | `$ignore` | 布尔参数决定是否忽略不可读的目录 |
| `in` | `$dirs` | 约束匹配文件在哪些目录中 |
| `getIterator` | | 获取当前 `Finder` 迭代器 |
| `append` | `$iterator` | `Appends an existing set of files/directories to the finder` |
| `count` | | 获取当前 `Finder` 迭代器匹配结果的数目 |
| `searchInDirectory` | `$dir` | |

# 官方的 Demo
Demo1
```
use Symfony\Component\Finder\Finder;

$finder = new Finder();

$iterator = $finder
  ->files()
  ->name('*.php')
  ->depth(0)
  ->size('>= 1K')
  ->in(__DIR__);

foreach ($iterator as $file) {
    print $file->getRealpath()."\n";
}
```

Demo2
```
$s3 = new \Zend_Service_Amazon_S3($key, $secret);
$s3->registerStreamWrapper("s3");

$finder = new Finder();
$finder->name('photos*')->size('< 100K')->date('since 1 hour ago');
foreach ($finder->in('s3://bucket-name') as $file) {
    print $file->getFilename()."\n";
}
```

上面 `Demo` 中的 `foreach` 循环中的 `$file` 是 `"Symfony\Component\Finder\SplFileInfo"` 
类实例，该类提供 `getRelativePathname` 方法来获取文件名；
