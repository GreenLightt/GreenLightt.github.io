---
title: PHP 配置管理扩展 Yaconf
date: 2018-02-05 22:21:38
description: 整理记录 yaconf
tags:
- PHP-Extensions
categories:
- PHP
---

# 介绍

`Yaconf` 是一个专门用来作配置管理的扩展，由 `Laurance` 编写并开源；

`Yaconf` 有以下的特性：

- 轻量，快速
- 常驻内存; 在 `PHP` 启动的时候, 处理所有的要处理的配置, 然后这些配置就会常驻内存, 随着 `PHP` 的生命周期存亡. 避免了每次请求的时候解析配置文件
- 支持丰富的配置类型, 包括字符串, 数组, 分节, 分节继承, 并且还可以在配置中直接写 `PHP` 的常量和环境变量
- 修改后自动 `reload`
- `api` 简单

# 安装

`Yaconf` 是一个 `PECL`扩展，所以可以通过如下命令轻易安装：

```
$pecl install yaconf
```

也可以，自行编译：

```
$ /path/to/php7/bin/phpize
$ ./configure --with-php-config=/path/to/php7/bin/php-config
$ make && make install
```

## 配置项

有两个运行时参数需要在 `php.ini` 中配置

`yaconf.directory`

配置文件目录, 这个配置不能通过 `ini_set ` 指定, 因为必须在 `PHP` 启动的时候就确定好.

`yaconf.check_delay`

多久(秒)检测一次文件变动, 如果是 0 就是不检测, 也就是说如果是 0 的时候, 文件变更只能通过重启 `PHP` 重新加载 





# 使用

`Yaconf` 采用`ini`文件作为配置文件, 这是因为作者觉得 `ini` 是最适合做配置文件的, `key`-`value` 格式, 清晰可读。

简单的配置写起来如下(以下全部假设 `ini` 文件的名字是 `test`):

```
    foo="bar"
    phpversion=PHP_VERSION
    env=${HOME}
```

如上所示, 对于一般的配置我们都用引号引起来. 而对于没有引起来的, 会尝试以 `PHP` 的常量做解释, 也就是说我们可以直接在配置里面写 `PHP` 的常量.

另外你也看到了, 我们可以直接在配置中写环境变量, 比如上面的 `env` :

```
    Yaconf::get("test.env"); //test是配置文件名字
    # 输出 /root
```

如上面所示, 你可以看到, 假设对于 `foo` 的值, 你可以通过如下代码访问:

```
    Yaconf::get("test.foo"); //test是配置文件名字
```

`Yaconf` 也**支持数组类型的配置**, 写法如下:

```
    arr[]=1
    arr.1=2
```

如上面所示你可以直接使用 `foo[]` 这种形式来定义数组, 也可以使用 `arr.1`, ` arr.2` 来指定 `key` 定义.
对于数组的第二个值, 你可以通过如下代码访问:

```
    Yaconf::get("test.arr.1"); //test是配置文件名字
```

或者你可以通过获取整个 `arr` 数组之后访问:

```
    $arr = Yaconf::get("test.arr"); //test是配置文件名字
    echo $arr[1];
```

`Yaconf` 也**支持 `map` 类型的配置**, 写法如下:

```
    map.foo=bar
    map.bar=foo
     
    ;你可以使用分号来写注释
    map2.foo.name=yaconf
    map2.foo.year=2015
```

对于 `map2` 的 `foo` 子 `map` 的 `name` 值可以通过如下形式访问:

```
Yaconf::get("test.map2.foo.name"); //test是配置文件名字
```

并且, 配置文件还可以**分节, 和分节继承**:

```
    [parent]
    parent="base"
    children="NULL"
     
    [children : parent]
    children="children"
```

请注意配置的分节继承的语法 `children:(冒号)parent`, 这的意思是 `children` 节继承全部 `base` 的配置项. 然后你在 `children` 节里面定义的和 `parent` 节中同名的配置, 会覆盖掉 `parent` 中定义的内容。

对于 `chidlren` 节的 `children` 配置的值可以通过如下形式访问:

```
Yaconf::get("test.children.children"); //test是配置文件名字
```
