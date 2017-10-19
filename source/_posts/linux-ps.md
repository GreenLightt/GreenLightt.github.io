﻿---
title: Linux 下进程查看命令 ps
date: 2017-10-12 00:12:00
description: 体验进程查看工具 ps
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

`ps` 命令是基本同时也是非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源；

参数挺多的，记录实际一些用法；

# 显示所有使用者的进程
命令：`ps auxf`
参数介绍：

- `a` : 显示其他用户启动的进程
- `u` : 启动这个进程的用户和它启动的时间，没有指定值，默认是 `root`; 会被 `a` 参数覆盖
- `x` : 显示没有控制终端的进程
- `f` : 全部列出

显示：
![此处输入链接的描述][1]



内容包括：

| USER | PID | %CPU | %MEM | VSZ | RSS | TTY | STAT | START | TIME | COMMAND |
|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|
| 用户名 | 进程`ID` | `cpu`占用率 | 内存占用率 | 使用虚存大小 | 实际内存的大小 | 关联的终端（`tty`） | 进程的状态 | 进程启动时间和日期 | 占用总`cpu`时间 | 正在执行的命令行命令 |

# 指定用户的进程
命令：`ps -fu rpc --forest`
参数介绍：

- `f` : 全部列出
- `u` : 指定用户名或者用户ID，没有指定值，默认是 `root`; 会被 `a` 参数覆盖
- `--forest` : 打印过程树，过程树显示系统中的进程如何相互链接; 父类被杀死的进程由`init`（或`systemd`）采用

显示：
![此处输入链接的描述][2]

# 指定用户组的进程
命令：`ps -fG daemon --forest`
参数介绍：

- `f` : 全部列出
- `G` : 指定组名或组ID
- `--forest` : 打印过程树，过程树显示系统中的进程如何相互链接; 父类被杀死的进程由`init`（或`systemd`）采用

显示：
![此处输入链接的描述][3]

# 指定 PID 的进程
命令：`ps -fp 1021`
参数介绍：

- `f` : 全部列出
- `p` : 指定进程ID，可选多个，以逗号隔开

显示：
![此处输入链接的描述][4]

# 指定 PPID 的子进程
命令：`ps -f --ppid 2`
参数介绍：

- `f` : 全部列出
- `--ppid` : 指定进程ID的子进程

显示：
![此处输入链接的描述][5]

# 指定 TTY 的进程
命令：`ps -ft tty1`
参数介绍：

- `f` : 全部列出
- `t` : 指定`tty`的进程

显示：
![此处输入链接的描述][6]

# 指定进程的所有线程
命令：`ps -fL -C httpd`
参数介绍：

- `f` : 全部列出
- `L` : 打印线程
- `C` : 指定进程名

显示：
![此处输入链接的描述][7]

内容包括：


| USER | PID | PPID | LWP | C | NLWP |
|: --- :|: --- :|: --- :|: --- :|: --- :|: --- :|
| 用户名 | 进程`ID` | 父进程`ID` | 线程 | 进程名 | 线程数 | 

# 根据CPU使用排序进程
静态命令：`ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head`
动态命令：`watch -n 1 'ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head'`
参数介绍：

- `e` : 显示所有进程，同`A`参数
- `o` : 自定义显示字段
- `--sort`：根据哪个字段排序



# 根据内存使用排序进程
静态命令：`ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head`
动态命令：`watch -n 1 'ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head'`
参数介绍：

- `e` : 显示所有进程，同`A`参数
- `o` : 自定义显示字段
- `--sort`：根据哪个字段排序

# 附录
## 进程状态

- `R`: 运行
- `S`: 睡眠
- `I`: 空闲
- `Z`: 僵死
- `D`: 不可中断
- `T`: 终止
- `P`: 等待交换页
- `W`: 无驻留页
- `X`: 死掉的进程
- `<`: 高优先级进程
- `N`: 低优先级进程
- `L`: 内存锁页
- `s`: 进程的领导者（在它之下有子进程）
- `+`: 位于后台的进程组
  
  


  [1]: http://owk2q4gs5.bkt.clouddn.com/ps-auxf.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/ps-fu.png
  [3]: http://owk2q4gs5.bkt.clouddn.com/ps-fG.png
  [4]: http://owk2q4gs5.bkt.clouddn.com/ps-fp.png
  [5]: http://owk2q4gs5.bkt.clouddn.com/ps-fppid.png
  [6]: http://owk2q4gs5.bkt.clouddn.com/ps-ft.png
  [7]: http://owk2q4gs5.bkt.clouddn.com/ps-fLC.png