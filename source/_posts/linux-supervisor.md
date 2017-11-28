---
title: Linux 后台进程管理利器 supervisor
date: 2017-11-28 15:59:00
description: 总结 supervisor 功能
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

`Linux` 的后台进程运行有好几种方法，例如 `nohup`，`screen` 等，但是，如果是一个服务程序，要可靠地在后台运行，我们就需要把它做成 `daemon`，最好还能监控进程状态，在意外结束时能自动重启。

`supervisor` 就是用 `Python` 开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台 `daemon`，并监控进程状态，异常退出时能自动重启。

官方文档参考 [http://supervisord.org/index.html](http://supervisord.org/index.html)

# 安装 supervisor
在 `centos` 上通过 `yum` 安装

```
yum -y install supervisor
```
`supervisor` 安装完成后会生成三个执行程序：`supervisortd`、`supervisorctl`、`/etc/supervisord.conf`，分别是 `supervisor` 的守护进程服务（用于接收进程管理命令）、客户端（用于和守护进程通信，发送管理进程的指令）、生成初始配置文件程序。

# 配置文件
## supervisord 配置
```
[supervisord]
http_port=/var/tmp/supervisor.sock ; UNIX socket 文件，supervisorctl 会使用
;http_port=127.0.0.1:9001     ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
;sockchmod=0700               ; socket 文件的 mode，默认是 0700
;sockchown=nobody.nogroup     ; socket 文件的 owner，格式： uid:gid
;umask=022                   ; (process file creation umask;default 022)
logfile=/var/log/supervisor/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
pidfile=/var/run/supervisord.pid ; pid 文件
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200

;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;http_username=user          ; 登录管理后台的用户名
;http_password=123           ; 登录管理后台的密码
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;user=chrism                 ; (default is current user, required if root)
;directory=/tmp              ; (default is not to cd during start)
;environment=KEY=value       ; (key value pairs to add to environment)

[supervisorctl]
serverurl=unix:///var/tmp/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")

; 包含其他的配置文件
[include]
files = relative/directory/*.ini    ; 可以是 *.conf 或 *.ini
```

## program 配置
可以把所有配置项都写到 `supervisord.conf` 文件里，但并不推荐这样做，而是通过 `include` 的方式把不同的程序（组）写到不同的配置文件里。

为了举例，我们新建一个目录 `/etc/supervisor/` 用于存放这些配置文件，相应的，把 `/etc/supervisord.conf` 里 `include` 部分的的配置修改一下：

```
[include]
files = /etc/supervisor/*.conf
```
假设有个用 `Python` 和 `Flask` 框架编写的用户中心系统，取名 `usercenter`，用 `gunicorn` (http://gunicorn.org/) 做 `web` 服务器。项目代码位于 `/home/leon/projects/usercenter`，`gunicorn` 配置文件为 `gunicorn.py`，`WSGI callable` 是 `wsgi.py` 里的 `app` 属性。所以直接在命令行启动的方式可能是这样的：

```
cd /home/leon/projects/usercenter
gunicorn -c gunicorn.py wsgi:app
```

现在编写一份配置文件来管理这个进程（需要注意：用 `supervisord` 管理时，`gunicorn` 的 `daemon` 选项需要设置为 `False`）
```
[program:usercenter]
directory = /home/leon/projects/usercenter ; 程序的启动目录
command = gunicorn -c gunicorn.py wsgi:app  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
autorestart = true   ; 程序异常退出后自动重启
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = leon          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /data/logs/usercenter_stdout.log
 
; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```


此举例部分参考 [使用 supervisor 管理进程](http://www.ttlsa.com/linux/using-supervisor-control-program/)；


# 启动关闭
## supervisor 服务器端
**启动 supervisor 服务**

```
supervisord -c /etc/supervisord.conf
```


**停止 supervisor 服务**

```
kill -9 $(ps -ef|grep supervisor | awk '{print $2}')
或
kill -9 `ps -ef|grep supervisor | awk '{print $2}'`
```

## supervisorctl 
`Supervisorctl` 是 ` supervisord` 的一个命令行客户端工具，启动时需要指定与 `supervisord` 使用同一份配置文件，否则与 `supervisord` 一样按照顺序查找配置文件。

```
supervisorctl -c /etc/supervisord.conf
```
上面这个命令会进入 supervisorctl 的 shell 界面，然后可以执行不同的命令了：

```
> status                # 查看程序状态
> stop usercenter       # 关闭 usercenter 程序
> start usercenter      # 启动 usercenter 程序
> restart usercenter    # 重启 usercenter 程序
> reread                # 读取有更新（增加）的配置文件，不会启动新添加的程序
> update                # 重启配置文件修改过的程序
```

上面这些命令都有相应的输出，除了进入 `supervisorctl` 的 `shell` 界面，也可以直接在 `bash` 终端运行：

```
$ supervisorctl status
$ supervisorctl stop usercenter
$ supervisorctl start usercenter
$ supervisorctl restart usercenter
$ supervisorctl reread
$ supervisorctl update
```



