---
title: 详解 PHP 服务器变量 $_SERVER
date: 2017-10-21 15:23:38
description: 总结 $_SERVER 的变量含义
tags:
categories:
- PHP
---


# 客户端
浏览当前页面的用户的 `IP` 地址。
```
[REMOTE_ADDR] => 192.168.1.5 
```
用户机器上连接到 `Web` 服务器所使用的端口号。 
```
[REMOTE_PORT] => 51666 
```

# 请求
通信协议
```
[REQUEST_SCHEME] => http 
```
请求页面时通信协议的名称和版本。
```
[SERVER_PROTOCOL] => HTTP/1.1 
```
访问页面使用的请求方法；例如，“`GET`”, “`HEAD`”，“`POST`”，“`PUT`”。 
```
[REQUEST_METHOD] => GET 
```
`query string`（查询字符串），如果有的话，通过它进行页面访问。
```
[QUERY_STRING] => 
```
当前请求头中 `Accept:` 项的内容，如果存在的话。
```
[HTTP_ACCEPT] => text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8 
```
当前请求头中 `User-Agent:` 项的内容，如果存在的话。该字符串表明了访问该页面的用户代理的信息。除此之外，你可以通过 `get_browser()` 来使用该值，从而定制页面输出以便适应用户代理的性能。 
```
[HTTP_USER_AGENT] => Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.91 Safari/537.36 
```
当前请求头中 `Host:` 项的内容，如果存在的话。
```
[HTTP_HOST] => 192.168.1.168 
```
当前请求头中 `Connection:` 项的内容
```
[HTTP_CONNECTION] => keep-alive 
```
当前请求头中 `Accept-Language:` 项的内容
```
[HTTP_ACCEPT_LANGUAGE] => zh-CN,zh;q=0.8 
```
当前请求头中 `Accept-Encoding:` 项的内容
```
[HTTP_ACCEPT_ENCODING] => gzip, deflate 
```
表示浏览器可读懂服务器发过来的请求。
```
[HTTP_UPGRADE_INSECURE_REQUESTS] => 1 
```
表示浏览器是否会缓存这个页面信息。
```
[HTTP_CACHE_CONTROL] => max-age=0 
```


# 服务器端
当前运行脚本所在的服务器的主机名。如果脚本运行于虚拟主机中，该名称是由那个虚拟主机所设置的值决定。
```
[SERVER_NAME] => localhost 
```

当前运行脚本所在的服务器的 `IP` 地址
```
[SERVER_ADDR] => 192.168.1.168 
```
`Web` 服务器使用的端口。默认值为 “80”。如果使用 `SSL` 安全连接，则这个值为用户设置的 `HTTP` 端口。
```
[SERVER_PORT] => 80 
```
服务器标识字符串，在响应请求时的头信息中给出。 
```
[SERVER_SOFTWARE] => nginx/1.13.5 
```
服务器使用的 `CGI` 规范的版本
```
[GATEWAY_INTERFACE] => CGI/1.1 
```
当前运行脚本所在的文档根目录。在服务器配置文件中定义。
```
[DOCUMENT_ROOT] => /usr/local/nginx/html 
```
`URI` 用来指定要访问的页面。例如 “/index.html”。
```
[REQUEST_URI] => /phpinfo.php 
```
当前执行脚本的文件名，与 `document root` 有关。
```
[PHP_SELF] => /phpinfo.php 
```
包含当前脚本的路径。这在页面需要指向自己时非常有用。`__FILE__` 常量包含当前脚本(例如包含文件)的完整路径和文件名。
```
[SCRIPT_NAME] => /phpinfo.php 
```
当前执行脚本的绝对路径。
```
[SCRIPT_FILENAME] => /usr/local/nginx/html/phpinfo.php 
```
请求开始时的时间戳，微秒级别的精准度。 自 PHP 5.4.0 开始生效。
```
[REQUEST_TIME_FLOAT] => 1508427782.9947 
```
请求开始时的时间戳。从 PHP 5.1.0 起可用。
```
[REQUEST_TIME] => 1508427782 
```


# 待确定

```
[HOSTNAME] => Redgo 
```
```
[USER] => root 
```
```
[HOME] => /root 
```
```
[SHELL] => /bin/bash 
```
```
[LANG] => en_US.UTF-8 
```
```
[PATH] => /usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/root/bin 
```
```
[OLDPWD] => /usr/local/nginx 
```
```
[PWD] => /usr/local/nginx/html 
```
```
[TERM] => screen-256color 
```
```
[TMUX] => /tmp/tmux-0/default,1690,1 
```
```
[TMUX_PANE] => %2 
```
```
[LS_COLORS] => rs=0:di=38;5;27:ln=38;5;51:mh=44;38;5;15:pi=40;38;5;11:so=38;5;13:do=38;5;5:bd=48;5;232;38;5;11:cd=48;5;232;38;5;3:or=48;5;232;38;5;9:mi=05;48;5;232;38;5;15:su=48;5;196;38;5;15:sg=48;5;11;38;5;16:ca=48;5;196;38;5;226:tw=48;5;10;38;5;16:ow=48;5;10;38;5;21:st=48;5;21;38;5;15:ex=38;5;34:*.tar=38;5;9:*.tgz=38;5;9:*.arj=38;5;9:*.taz=38;5;9:*.lzh=38;5;9:*.lzma=38;5;9:*.tlz=38;5;9:*.txz=38;5;9:*.zip=38;5;9:*.z=38;5;9:*.Z=38;5;9:*.dz=38;5;9:*.gz=38;5;9:*.lz=38;5;9:*.xz=38;5;9:*.bz2=38;5;9:*.tbz=38;5;9:*.tbz2=38;5;9:*.bz=38;5;9:*.tz=38;5;9:*.deb=38;5;9:*.rpm=38;5;9:*.jar=38;5;9:*.rar=38;5;9:*.ace=38;5;9:*.zoo=38;5;9:*.cpio=38;5;9:*.7z=38;5;9:*.rz=38;5;9:*.jpg=38;5;13:*.jpeg=38;5;13:*.gif=38;5;13:*.bmp=38;5;13:*.pbm=38;5;13:*.pgm=38;5;13:*.ppm=38;5;13:*.tga=38;5;13:*.xbm=38;5;13:*.xpm=38;5;13:*.tif=38;5;13:*.tiff=38;5;13:*.png=38;5;13:*.svg=38;5;13:*.svgz=38;5;13:*.mng=38;5;13:*.pcx=38;5;13:*.mov=38;5;13:*.mpg=38;5;13:*.mpeg=38;5;13:*.m2v=38;5;13:*.mkv=38;5;13:*.ogm=38;5;13:*.mp4=38;5;13:*.m4v=38;5;13:*.mp4v=38;5;13:*.vob=38;5;13:*.qt=38;5;13:*.nuv=38;5;13:*.wmv=38;5;13:*.asf=38;5;13:*.rm=38;5;13:*.rmvb=38;5;13:*.flc=38;5;13:*.avi=38;5;13:*.fli=38;5;13:*.flv=38;5;13:*.gl=38;5;13:*.dl=38;5;13:*.xcf=38;5;13:*.xwd=38;5;13:*.yuv=38;5;13:*.cgm=38;5;13:*.emf=38;5;13:*.axv=38;5;13:*.anx=38;5;13:*.ogv=38;5;13:*.ogx=38;5;13:*.aac=38;5;45:*.au=38;5;45:*.flac=38;5;45:*.mid=38;5;45:*.midi=38;5;45:*.mka=38;5;45:*.mp3=38;5;45:*.mpc=38;5;45:*.ogg=38;5;45:*.ra=38;5;45:*.wav=38;5;45:*.axa=38;5;45:*.oga=38;5;45:*.spx=38;5;45:*.xspf=38;5;45: 
```
```
[SHLVL] => 2 
```
```
[LOGNAME] => root 
```
```
[LESSOPEN] => ||/usr/bin/lesspipe.sh %s 
```
```
[G_BROKEN_FILENAMES] => 1 
```
```
[_] => /usr/local/php7/bin/php-cgi 
```
```
[FCGI_ROLE] => RESPONDER 
```
```
[REDIRECT_STATUS] => 200 
```
```
[DOCUMENT_URI] => /phpinfo.php 
```
```
[CONTENT_LENGTH] => 
```
```
[CONTENT_TYPE] => 
```
```
[MAIL] => /var/spool/mail/root 
```
```
[QTDIR] => /usr/lib64/qt-3.3 
```
```
[QTINC] => /usr/lib64/qt-3.3/include 
```
```
[QTLIB] => /usr/lib64/qt-3.3/lib 
```
```
[SELINUX_ROLE_REQUESTED] => 
```
```
[SELINUX_USE_CURRENT_RANGE] => 
```
```
[SELINUX_LEVEL_REQUESTED] => 
```
```
[HISTCONTROL] => ignoredups 
```
```
[HISTSIZE] => 1000 
```
```
[CVS_RSH] => ssh 
```
```
[SSH_CLIENT] => 192.168.1.5 49775 22 
```
```
[SSH_CONNECTION] => 192.168.1.5 51100 192.168.1.168 22 
```
```
[SSH_TTY] => /dev/pts/0 
```