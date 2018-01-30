---
title: PHP SPL 标准库
date: 2018-01-30 21:56:38
description: 提炼总结 SPL 相关知识点
tags:
categories:
- PHP
---


`SPL` 是用于解决典型问题(`standard problems`)的一组接口与类的集合。 

# 数据结构

## 双向链表

双链表 (`DLL`) 是一个链接到两个方向的节点列表。当底层结构是 `DLL` 时, 迭代器的操作、对两端的访问、节点的添加或删除都具有 `O(1)` 的开销。因此, 它为栈和队列提供了一个合适的实现。

![此处输入图片的描述][1]

- 双向链表的数据结构 [SplDoublyLinkedList](http://php.net/manual/zh/class.spldoublylinkedlist.php)
  - 基于双向链表实现的栈 [SplStack](http://php.net/manual/zh/class.splstack.php)
  - 基于双向链表实现的队列 [SplQueue](http://php.net/manual/zh/class.splqueue.php)

  [1]: http://owk2q4gs5.bkt.clouddn.com/timg.jpg
  
## 堆

最大（最小）堆是一棵每一个节点的键值都不小于（大于）其孩子（如果存在）的键值的树。大顶堆是一棵完全二叉树，同时也是一棵最大树。小顶堆是一棵完全完全二叉树，同时也是一棵最小树。


- 堆数据结构 [SplHeap](http://php.net/manual/zh/class.splheap.php)
  - 大顶堆 [SplMaxHeap](http://php.net/manual/zh/class.splmaxheap.php)
  - 小顶堆 [SplMinHeap](http://php.net/manual/zh/class.splminheap.php)
  
基于堆继承，特定于队列优先级场景的[SplPriorityQueue](http://php.net/manual/zh/class.splpriorityqueue.php)

## 数组

数组是以连续方式存储数据的结构, 可通过索引进行访问。不要将它们与 `php` 数组混淆: `php` 数组实际上是按照有序的列表实现的。 

- 固定大小的数组 [SplFixedArray](http://php.net/manual/zh/class.splfixedarray.php)


## 对象集

SPL 提供了从对象到数据的映射。此映射也可用作对象集。

- 对象集 [SplObjectStorage](http://php.net/manual/zh/class.splobjectstorage.php)


# 迭代器

`SPL` 提供一系列迭代器以遍历不同的对象。


- [`ArrayIterator`](http://php.net/manual/zh/class.arrayiterator.php)  遍历数组和对象
    - [`RecursiveArrayIterator`](http://php.net/manual/zh/class.recursivearrayiterator.php) 可递归遍历数组和对象
- [`EmptyIterator`](http://php.net/manual/zh/class.emptyiterator.php) 空迭代器
- [`IteratorIterator`](http://php.net/manual/zh/class.iteratoriterator.php) 迭代器的装饰器
    - [`AppendIterator`](http://php.net/manual/zh/class.appenditerator.php) 这个迭代器能陆续遍历几个迭代器 
    - [`CachingIterator`](http://php.net/manual/zh/class.cachingiterator.php) 支持对另一个迭代器的缓存迭代。
        - [`RecursiveCachingIterator`](http://php.net/manual/zh/class.recursivecachingiterator.php)  支持对另一个可递归迭代器的递归缓存迭代
    - [`FilterIterator`](http://php.net/manual/zh/class.filteriterator.php) 过滤元素的迭代器
        - [`CallbackFilterIterator`](http://php.net/manual/zh/class.callbackfilteriterator.php) 回调函数过滤元素的迭代器
            - [`RecursiveCallbackFilterIterator`](http://php.net/manual/zh/class.recursivecallbackfilteriterator.php) 可递归地回调函数过滤元素的迭代器
        - [`RecursiveFilterIterator`](http://php.net/manual/zh/class.recursivefilteriterator.php)  与 `RecursiveIteratorIterator` 结合使用，可以进行递归过滤
            - [`ParentIterator`](http://php.net/manual/zh/class.parentiterator.php) 筛选出有子元素的迭代器
        - [`RegexIterator`](http://php.net/manual/zh/class.regexiterator.php) 正则迭代器
            - [`RecursiveRegexIterator`](http://php.net/manual/zh/class.recursiveregexiterator.php) 可递归的正则迭代器
        - [`InfiniteIterator`](http://php.net/manual/zh/class.infiniteiterator.php) 无限循环遍历
        - [`LimitIterator`](http://php.net/manual/zh/class.limititerator.php) 允许遍历一个 `Iterator` 的限定子集的元素. 
        - [`NoRewindIterator`](http://php.net/manual/zh/class.norewinditerator.php) 不可再 `rewind` 遍历
- [`MultiplerIterator`](http://php.net/manual/zh/class.multipleiterator.php) 并列迭代多个迭代器
- [`RecursiveIteratorIterator`](http://php.net/manual/zh/class.recursiveiteratoriterator.php) 可递归的迭代器
    - [`RecursiveTreeIterator`](http://php.net/manual/zh/class.recursivetreeiterator.php) 允许遍历一个递归迭代器来生成一个`ASCII`图形树。
- [`DirectoryIterator`](http://php.net/manual/zh/class.directoryiterator.php) 目录迭代器
    - [`FilesystemIterator`](http://php.net/manual/zh/class.filesystemiterator.php) 对目录迭代器的二次封装
        - [`GlobIterator`](http://php.net/manual/zh/class.globiterator.php) 遍历一个文件系统行为类似于 `glob()`. 即路径名带 `*`
        - [`RecursiveDirectoryIterator`](http://php.net/manual/zh/class.recursivedirectoryiterator.php) 可递归目录迭代器
    

# 接口

## `Countable`

类实现 `Countable` 可被用于 `count()` 函数. 

```
Countable {
    /* 方法 */
    abstract public int count ( void )
}
```

## `OuterIterator`

类实现 `OuterIterator` 可被用于迭代迭代器；

```
OuterIterator extends Iterator {
    /* 方法 */
    public Iterator getInnerIterator ( void )
    /* 继承的方法 */
    abstract public mixed Iterator::current ( void )
    abstract public scalar Iterator::key ( void )
    abstract public void Iterator::next ( void )
    abstract public void Iterator::rewind ( void )
    abstract public boolean Iterator::valid ( void )
}
```

## `RecursiveIterator`

类实现 `RecursiveIterator` 可被用于递归迭代迭代器；

```
RecursiveIterator extends Iterator {
    /* 方法 */
    public RecursiveIterator getChildren ( void )
    public bool hasChildren ( void )
    /* 继承的方法 */
    abstract public mixed Iterator::current ( void )
    abstract public scalar Iterator::key ( void )
    abstract public void Iterator::next ( void )
    abstract public void Iterator::rewind ( void )
    abstract public boolean Iterator::valid ( void )
}
```

## `SeekableIterator`

可以根据下标搜索

```
SeekableIterator extends Iterator {
    /* 方法 */
    abstract public void seek ( int $position )
    /* 继承的方法 */
    abstract public mixed Iterator::current ( void )
    abstract public scalar Iterator::key ( void )
    abstract public void Iterator::next ( void )
    abstract public void Iterator::rewind ( void )
    abstract public boolean Iterator::valid ( void )
}
```

## `SplSubject` 与 `SplObserver`

`SplSubject` 与 [`SplObserver`](http://php.net/manual/zh/class.splobserver.php) 一起使用实现观察者模式；

```
SplSubject {
    /* 方法 */
    abstract public void attach ( SplObserver $observer )
    abstract public void detach ( SplObserver $observer )
    abstract public void notify ( void )
}

SplObserver {
    /* 方法 */
    abstract public void update ( SplSubject $subject )
}
```


# 异常

- `LogicException` (`extends Exception`) 应用程序编写有错误时会抛出此异常
    - `BadFunctionCallException` 找不到方法
        - `BadMethodCallException` 找不到函数
    - `DomainException` 值不符合有效数据域
    - `InvalidArgumentException` 无效参数
    - `LengthException` 长度异常
    - `OutOfRangeException` 请求非法索引（重点在索引 `index`）
- `RuntimeException` (`extends Exception`) 运程过程中发现的异常
    - `OutOfBoundsException` 键值找不到（重点在 `key`）
    - `OverflowException` 向一个装满元素的容器再添加元素时，报此异常
    - `RangeException` 
      程序执行期间，抛出的异常表示范围错误。通常这意味着除了在溢出之外，还有一个算术错误。这是 `DomainException` 的运行时版本。
    - `UnderflowException` 在一个空容器执行无效操作，比如移除元素
    - `UnexpectedValueException` 不是预期值异常

# SPL 函数

| 函数名 | 描述 |
|: --- :|: --- :|
| `class_implements` | 返回指定的类实现的所有接口  |
| `class_parents` |  返回指定类的父类 |
| `class_uses` |  返回指定类用到的 `trait` |
| `iterator_apply` | 为迭代器中每个元素调用一个用户自定义函数 |
| `iterator_count` | 计算迭代器中元素的个数 |
| `iterator_to_array` | 将迭代器中的元素拷贝到数组 |
| `spl_classes` |  返回所有可用的 `SPL` 类 |
| `spl_object_hash` | 返回指定对象的 `hash id`  |
| `spl_object_id` | 为指定对象生成独一无二的 `id`， 类似于 `spl_object_hash` |
| `spl_autoload_functions` | 返回所有已注册的 `__autoload()` 函数 |
| `spl_autoload_register` |  注册给定的函数作为 `__autoload` 的实现 |
| `spl_autoload_unregister` | 注销已注册的 `__autoload()` 函数 |
| `spl_autoload` |  `__autoload()` 函数的默认实现 |
| `spl_autoload_extensions` | 注册并返回 `spl_autoload` 函数使用的默认文件扩展名。 |
| `spl_autoload_call` | 尝试调用所有已注册的 `__autoload()` 函数来装载请求类 |


# 文件处理

## `SplFileInfo`

`SplFileInfo` 类提供了针对单个文件的高级面向对象接口。

获取文件的基本信息；比如文件名、路径、访问时间、修改时间；

## `SplFileObject`

继承了 `SplFileInfo` 类，同时实现迭代器相关功能；

可以对文件内容进行迭代读取，和重写，不支持 `append` 追加 ；


## `SplTempFileObject`

继承了 `SplFileObject` 类；临时文件；
