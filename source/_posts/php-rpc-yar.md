---
title: 并行的 PHP RPC 框架 Yar
date: 2018-02-06 11:23:38
description: 通过 Yar 框架的小示例入门 Rpc；
tags:
- RPC
categories:
- PHP
---

# 介绍

`RPC`（`Remote Procedure Call Protocol`）远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。`RPC` 不依赖于具体的网络传输协议，`tcp`、`udp` 等都可以。由于存在各式各样的变换和细节差异，相应的 `rpc` 也派生出了各式远程过程通信协议。`RPC` 是跨语言的通信标准，`SUN` 和微软都有其实现，比如`RMI` 可以被看作 `SUN` 对 `RPC` 的 `Java` 版本( 实现)，而微软的 `DCOM` 就是建立在 `ORPC` 协议之上。一言以蔽之，`RPC` 是协议，而无论是 `SUN` 的 `RMI` 还是微软的 `DCOM` 都是对该协议的不同实现，二者都为编程人员提供了应用PRC技术的程序接口（`API``）。

# 安装

**`pecl` 安装**

`yar` 是一个 `pecl` 扩展，因此可以直接 `install` :

```
pecl install yar
```

**自行编译**

```
$/path/to/phpize
$./configure --with-php-config=/path/to/php-config/
$make && make install
```

**Install Yar with msgpack**

首先应该安装 `msgpack` 扩展 

```
pecl install msgpack
```

或者，也可以获取 `github` 源代码 [ https://github.com/msgpack/msgpack-php](https://github.com/msgpack/msgpack-php)

```
$phpize
$configure --with-php-config=/path/to/php-config/ --enable-msgpack
$make && make install
```

## 配置项

- `yar.timeout`    处理超时，默认为 5000 (ms)
- `yar.connect_timeout` 连接超时，默认为 1000 (ms)
- `yar.packager` //设置 `yar` 的打包工具，默认是 "`php`", 当以 `--enable-msgpack` 构建扩展，则默认值为 `msgpack`, 它的值可以为  "`php`", "`json`", "`msgpack`"
- `yar.debug` 默认是 `Off`， 打开的时候, `Yar` 会把请求过程的日志都打印出来(到 `stderr`) 
- `yar.expose_info` 默认是 `On`, 如果关闭, 则当通过浏览器访问 `Server` 的时候, 不会出现 `Server Info` 信息
- `yar.content_type` 默认是 "`application/octet-stream`"
- `yar.allow_persistent` 默认是 `Off`


# 使用

## 常量

- `YAR_VERSION` ： `YAR` 的当前版本
- `YAR_OPT_PACKAGER`
- `YAR_OPT_PERSISTENT`
- `YAR_OPT_TIMEOUT`
- `YAR_OPT_CONNECT_TIMEOUT`
- `YAR_OPT_HEADER`  


## 简单示例

**`Server` 端**
服务器端提供算术服务

```
<?php

/* 假设这个页面的访问路径是: http://example.com/operator.php */

class Operator {

    /**
     * Add two operands
     * @param interge 
     * @return interge
     */
    public function add($a, $b) {
        return $this->_add($a, $b);
    }

    /**
     * Sub 
     */
    public function sub($a, $b) {
        return $a - $b;
    }

    /**
     * Mul
     */
    public function mul($a, $b) {
        return $a * $b;
    }

    /**
     * Protected methods will not be exposed
     * @param interge 
     * @return interge
     */
    protected function _add($a, $b) {
        return $a + $b;
    }
}

$server = new Yar_Server(new Operator());
$server->handle();
```

通过 `Get` 请求， 从浏览器端访问，可以看到自动生成的 `API` 文档

![此处输入图片的描述][1]

**`Client` 端调用**

```
<?php
$client = new yar_client("http://example.com/operator.php");

/* call directly */
var_dump($client->add(1, 2));

/* call via call */
var_dump($client->call("add", array(3, 2)));


/* __add can not be called */
var_dump($client->_add(1, 2));
```
输出如下：

```
int(3)
int(5)
PHP Fatal error:  Uncaught exception 'Yar_Server_Exception' with message 'call to api Operator::_add() failed' in *
```

**`Client` 端异步调用**

```
<?php
function callback($ret, $callinfo) {
    echo $callinfo['method'] , " result: ", $ret , "\n";
}

/* 注册一个异步调用 */
Yar_Concurrent_Client::call("http://example.com/operator.php", "add", array(1, 2), "callback");
Yar_Concurrent_Client::call("http://example.com/operator.php", "sub", array(2, 1), "callback");
Yar_Concurrent_Client::call("http://example.com/operator.php", "mul", array(2, 2), "callback");

/* 发送所有注册的调用, 等待返回, 返回后Yar会调用callback回掉函数 */
Yar_Concurrent_Client::loop();

/* 重置call ,否则上面的call会调用*/
Yar_Concurrent_Client::reset();
Yar_Concurrent_Client::call("http://example.com/operator.php", "sub", array(6, 7), "callback");
Yar_Concurrent_Client::loop();
```

# 参考阅读

- [PHP 官网关于 yar 的文档](http://php.net/manual/zh/book.yar.php)
- [Yar – 并行的RPC框架(Concurrent RPC framework)](http://www.laruence.com/2012/09/15/2779.html)
- [为什么需要 RPC，而不是简单的 HTTP 接口](https://www.cnblogs.com/winner-0715/p/5847638.html)


  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180206102730.png
