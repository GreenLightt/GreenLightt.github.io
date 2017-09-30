---
title: 搭建 WNMP 环境
date: 2017-09-30 13:46:41
description: 在 Windows 上搭建 nginx + php + mysql 环境
tags:
categories:
- PHP
copyright: false
---

本文是在`Win10`上进行搭建`PHP` + `Nginx` + `Mysql`环境的指导性文章；

# PHP
[PHP 官方下载地址](http://windows.php.net/download#php-7.1)

下地地址会出现 `Thread Safe` 和 `Non Thread Safe` 两个版本，选择哪种版本参考以下：

- 采用`IIS` + `ISAPI`的话，  就使用 `TS` 版本
- 采用`IIS` + `FastCGI`的话，就使用 `NTS` 版本

## 配置
1. 进入 `php` 的解压目录，将 `php.ini-production` 文件复制一份，重新命名为 `php.ini`
2. 修改 `php.ini`
  取消下列设置的注释 

  ```
  extension_dir = "ext"
  
  extension=php_curl.dll
  extension=php_gd2.dll
  extension=php_mbstring.dll
  extension=php_mysqli.dll
  extension=php_pdo_mysql.dll
  extension=php_pdo_odbc.dll
  extension=php_xmlrpc.dll
  
  enable_dl = On
  cgi.fix_pathinfo=1
  cgi.force_redirect = 0
  fastcgi.impersonate = 1
  cgi.rfc2616_headers = 1
  
  ```
  配置 `session`功能
  ```
  session.save_path = "D:/php_session"
  ```
  配置文件上传功能
  ```
  upload_tmp_dir = "D:/php_upload"
  ```
  修改时区信息
  ```
  date.timezone = Asia/Shanghai
  ```
3. 加入全局环境变量；将 `php` 当下的目录以及 `php\ext` 的目录放置到系统环境变量中的 `PATH`中去。（环境变量的设置路径： `win + x`键 > 系统信息 > 高级系统设置 > 环境变量 > 系统变量，找到 `Path` ）

# Nginx
[Nginx 官方下载地址](http://nginx.org/en/download.html)

将下载的文件移到自定义的文件夹，解压，双击`nginx.exe`即可运行 `nginx`（默认为 `80`端口）；

## Nginx 启用 PHP
修改 `config.php`;

先将前面的“`#`”去掉，同样将`root  html;`改为 `root  D:\Nginx\nginx\html;`。
再把“`/scripts`”改为“`$document_root`”(重要)，这里的“`$document_root`”就是指前面“`root`”所指的站点路径，这是改完后的：

```
        location ~ \.php$ {
            root           D:/Nginx/nginx/html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  D:/Nginx/nginx/html/$fastcgi_script_name;
            include        fastcgi_params;
        }
```

## 验证
任务管理器结束`nginx.exe`进程； 

启动内置 `fastcgi` 管理器 `php-cgi.exe -b 127.0.0.1:9000 -c D:\Nginx\php\php.ini `

再运行`nginx.exe`；

通过浏览器访问 `http://127.0.0.1/phpinfo.php`；

## 优化：一键启动与关闭
目前每次启动`nginx`，都需要开`cmd`，执行`php-cgi.exe -b 127.0.0.1:9000 -c D:\Nginx\php\php.ini `来启动内置 `fastcgi` 管理器，还要保证`cmd`页面不关；然后要启动`nginx`服务，操作步骤繁琐，易出错；

**.bat 脚本制作**
利用一个`windows`插件工具[RunHiddenConsole.exe](http://owk2q4gs5.bkt.clouddn.com/RunHiddenConsole.exe)，下载；

**同目录下**，自制一个启动脚本`start_nginx.bat`与关闭脚本`stop_nginx.bat`；这样，通过点击启动脚本，就可以同时开启内置 `fastcgi` 管理器与`nginx`服务；

*start_nginx.bat文件*
```
@echo off
REM Windows 下无效
REM set PHP_FCGI_CHILDREN=5

REM 每个进程处理的最大请求数，或设置为 Windows 环境变量
set PHP_FCGI_MAX_REQUESTS=1000
 
echo Starting PHP FastCGI...
RunHiddenConsole D:/Nginx/php/php-cgi.exe -b 127.0.0.1:9000 -c D:/Nginx/php/php.ini
 
echo Starting nginx...
RunHiddenConsole D:/Nginx/nginx/nginx.exe -p D:/Nginx/nginx
```

*stop_nginx.bat文件*
```
@echo off
echo Stopping nginx...  
taskkill /F /IM nginx.exe > nul
echo Stopping PHP FastCGI...
taskkill /F /IM php-cgi.exe > nul
exit
```

# Mysql
[Mysql 官方下载地址](https://dev.mysql.com/downloads/windows/)

如果遇到 `(mysql-installer-web-community-5.7.19.0.msi)` 和 `(mysql-installer-community-5.7.19.0.msi)` 选择后一个；

安装简单，不记录；

