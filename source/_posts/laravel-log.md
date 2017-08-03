---
title: Laravel 大将之 日志 模块
date: 2017-06-01 12:31:38
description: 介绍如何使用日志功能，手动记录信息
tags:
- Laravel-5.4
categories:
- Laravel
---

<!-- toc -->

# 简介
`Laravel`的日志模块位于`Illuminate/Log`文件夹下；通过封装`Monolog`插件提供日志服务；

# 配置
## 配置文件
在`config/app.php`文件中关于日志模块的配置项有两个，分别如下：

 ```php
 // 日志模式，可选择参数有 "single", "daily", "syslog", "errorlog"
'log' => env('APP_LOG', 'single'),
 // 错误等级，为 "RFC 5424" 中定义的八种日志级
'log_level' => env('APP_LOG_LEVEL', 'debug'),
 ```

1. 日志模式不同的参数值有不同的含义：

 - single
  所有日志信息都会输出到`storage/log/laravel.log`文件中
 - daily
  每天的日志信息都会输出到`storage/log`文件夹下的日志文件中，日志文件名会包含当天的年月日信息；
 - syslog
  日志信息输出到系统的日志文件中；比如，笔者是`centos`系统，日志信息写到了`/var/log/message`文件中
 - errorlog
  相当于调用`PHP`的`error_log`语句，没有写入到文件

2. 八种日志错误等级分别为：

| 错误等级 | 描述 | 整型值 | 应用场景 | 
|: --- :|: --- :|: --- :|: --- :|
| debug      | 代码引用为 "MonologLogger::DEBUG" | 100 | 紧急，如系统挂掉 |
| info       | 代码引用为 "MonologLogger::INFO" |  200 | 需要立即采取行动，如数据库异常等 |
| notice     | 代码引用为 "MonologLogger::NOTICE" | 250 | 严重问题，如异常 | 
| warning    | 代码引用为 "MonologLogger::WARNING" | 300 | 运行时错误，不需要立即处理但需要被记录和监控|
| error      | 代码引用为 "MonologLogger::ERROR" | 400 | 警告但不是错误，比如使用了被废弃的API |
| critical   | 代码引用为 "MonologLogger::CRITICAL" | 500 | 普通但值得注意的事件 |
| alert      | 代码引用为 "MonologLogger::ALERT" | 550 | 感兴趣的事件，比如登录、退出 |
| emergency  | 代码引用为 "MonologLogger::EMERGENCY" | 600 | 详细的调试信息 |
 如果`log_level`的值为`critical`, 则日志只会输出`alert`和`emergency`这两种错误等级的信息；

## 自定义配置
如果你想要在应用中完全控制`Monolog`的配置，可以使用应用的`configureMonologUsing`方法；
在`bootstrap/app.php`文件返回`$app`变量之前调用该方法;
比如，使用`daily`日志模式，但日志文件名想自定义，可以如下：
```
$app->configureMonologUsing(function(Monolog\Logger $log) {
    $filename = storage_path('/logs/' . php_sapi_name() . '.log');
    $handler = new Monolog\Handler\RotatingFileHandler($filename, 0, \Monolog\Logger::DEBUG, true, 0664);
    $handler->setFormatter(new \Monolog\Formatter\LineFormatter(null, null, true, true));  
    $log->pushHandler($handler);                                                                    
});
```

# 使用
## 日志记录
1. 创建日志器实例

    直接创建`Writer`实例，日志器实例还需要设置日志模式与错误等级;
    ```
$writer = new Illuminate\Log\Writer(new Monolog\Logger('channel_name'));
$writer->useErrorLog('info');
    ```
    或者通过全局帮助函数，从服务容器获取实例;日志模式与错误等级已按照配置文件中的参数设置好了；
    
    ```
        $writer = app('log');
    ```
    还有一种直接通过门面模式`\Log`;比如要输出`debug`级信息，执行`\Log::debug('xxxx'); `
    
2. 输出各种类型信息

 ```
 $writer->emergency('消息内容', ['上下文环境，比如用户ID' => '这里显示用户ID的值，比如78']);
 $writer->alert('消息内容');
 $writer->critical('消息内容');
 $writer->error('消息内容');
 $writer->warning('消息内容');
 $writer->notice('消息内容');
 $writer->info('消息内容');
 $writer->debug('消息内容');
 
 $writer->log('emergency', '消息内容'， ['上下文' => '环境']);
 $writer->log('alert', '消息内容'， ['上下文' => '环境']);
 ......
 ```

## 事件监听
每次执行日志输出信息时，如果日志器包含事件分发器（即`event dispatcher`），则会触发`\Illuminate\Log\Events\MessageLogged::class`这个事件；

`\Illuminate\Log\Events\MessageLogged::class`这个事件的构造函数如下：
```
    public function __construct($level, $message, array $context = []) {
        $this->level = $level;
        $this->message = $message; 
        $this->context = $context;
    }
```

首先如何为日志器设置事件分发器，有两处，第一处是构造函数的第二个参数，可以传入事件分发器实例；第二处是通过调用`setEventDispatcher`方法；Laravel在服务容器注册绑定的日志器，带有事件分发器；

现在，需要为日志写入事件安排一些监听器，通过调用`listen`方法，比如:

```
$dispatcher = $writer->getEventDispatcher();
$dispatcher->listen(\Illuminate\Log\Events\MessageLogged::class, function ($event) {
        // 监听器的内容
});
```
