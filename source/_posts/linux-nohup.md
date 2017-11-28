---
title: Linux 不挂断运行命令工具 nohup
date: 2017-11-28 18:22:00
description: 总结 nohup 的用法
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

`nohup` 命令运行由 `Command` 参数和任何相关的 `Arg` 参数指定的命令，忽略所有挂断（`SIGHUP`）信号。在注销后使用 `nohup` 命令运行后台中的程序。要运行后台中的 `nohup` 命令，添加 `&` （ 表示“`and`”的符号）到命令的尾部。

如果不将 `nohup` 命令的输出重定向，**输出将附加到当前目录的 `nohup.out` 文件中**。如果当前目录的 `nohup.out` 文件不可写，输出重定向到 `$HOME/nohup.out`  文件中。如果没有文件能创建或打开以用于追加，那么 `Command` 参数指定的命令不可调用。如果标准错误是一个终端，那么把指定的命令写给标准错误的所有输出作为标准输出重定向到相同的文件描述符。

比如要执行 `laravel` 的队列监听：如果 `session` 会话关闭，则监听会断开；

```
php artisan queue:listen
```

`session` 会话意外关闭，还想继续执行监听，可以执行：

```
nohup php artisan queue:listen
```

`session` 会话意外关闭，还想继续执行监听，同时不阻塞命令行输入，执行：

```
nohup php artisan queue:listen &
```



