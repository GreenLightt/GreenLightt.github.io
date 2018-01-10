---
title: ThinkPHP 框架搭建
date: 2018-01-10 11:22:41
description: thinkphp5 框架搭建做记录
tags:
- ThinkPHP
categories:
copyright: false
---

# 本地部署及运行

严格来说，`ThinkPHP` 无需安装过程，这里所说的安装其实就是把 `ThinkPHP` 框架放入 `WEB` 运行环境；

所以只要将代码下载到本地，并配置服务器即可；

笔者此处采用 `Composer` 安装；


```
# tp5 即项目文件夹名称
composer create-project topthink/think=5.0.* tp5  --prefer-dist
```

在 `Nginx` 上配置：

```
    server {                                                                                        
        listen       9052;                                                   
        server_name  localhost;                                                         
    
        proxy_buffer_size 128k;                                                                
        proxy_buffers 8 256k;                                                      
        proxy_busy_buffers_size 256k;                                          
    
        set $root_path '/opt/tp5-cms';                                             
        root $root_path;                                             
    
        index  index.php index.html index.htm;                                      
    
        location / {                                              
            if (!-e $request_filename) {                                       
                rewrite  ^/(.*)$  /index.php/$1  last;                                   
                break;                                        
            }                                                 
        }                                                    
    
        location ~ \.php {                                        
            fastcgi_pass   unix:/opt/bksite/php/var/run/www.sock;                            
            fastcgi_split_path_info ^(.+\.php)(/.*)$;                                       
            include fastcgi_params;                                           
            fastcgi_param SCRIPT_FILENAME $root_path$fastcgi_script_name;                            
            fastcgi_param DOCUMENT_ROOT $root_path;                                 
            fastcgi_param SCRIPT_NAME $fastcgi_script_name;                                   
            fastcgi_param PATH_INFO $fastcgi_path_info;                             
        }                                         
    
        location ~ (css|img|js|flv|swf|download)/(.+)$ {                    
            root $root_path;                                  
        }                                           
    
        location ~ /\.ht {                         
            deny all;
        }
    }
```

成功页面如下：

![此处输入图片的描述][1]


[1]: http://owk2q4gs5.bkt.clouddn.com/2016-03-11_56e274a2376df.png


# 目录结构
最初的目录结构如下：

```
project  应用部署目录
├─application           应用目录（可设置）
    │  ├─common             公共模块目录（可更改）
│  ├─index              模块目录(可更改)
    │  │  ├─config.php      模块配置文件
    │  │  ├─common.php      模块函数文件
    │  │  ├─controller      控制器目录
    │  │  ├─model           模型目录
    │  │  ├─view            视图目录
    │  │  └─ ...            更多类库目录
    │  ├─command.php        命令行工具配置文件
    │  ├─common.php         应用公共（函数）文件
    │  ├─config.php         应用（公共）配置文件
    │  ├─database.php       数据库配置文件
    │  ├─tags.php           应用行为扩展定义文件
    │  └─route.php          路由配置文件
    ├─extend                扩展类库目录（可定义）
    ├─public                WEB 部署目录（对外访问目录）
│  ├─static             静态资源存放目录(css,js,image)
    │  ├─index.php          应用入口文件
    │  ├─router.php         快速测试文件
    │  └─.htaccess          用于 apache 的重写
    ├─runtime               应用的运行时目录（可写，可设置）
    ├─vendor                第三方类库目录（Composer）
    ├─thinkphp              框架系统目录
    │  ├─lang               语言包目录
    │  ├─library            框架核心类库目录
    │  │  ├─think           Think 类库包目录
    │  │  └─traits          系统 Traits 目录
    │  ├─tpl                系统模板目录
    │  ├─.htaccess          用于 apache 的重写
    │  ├─.travis.yml        CI 定义文件
    │  ├─base.php           基础定义文件
    │  ├─composer.json      composer 定义文件
    │  ├─console.php        控制台入口文件
    │  ├─convention.php     惯例配置文件
    │  ├─helper.php         助手函数文件（可选）
    │  ├─LICENSE.txt        授权说明文件
    │  ├─phpunit.xml        单元测试配置文件
    │  ├─README.md          README 文件
    │  └─start.php          框架引导文件
    ├─build.php             自动生成定义文件（参考）
    ├─composer.json         composer 定义文件
    ├─LICENSE.txt           授权说明文件
    ├─README.md             README 文件
    ├─think                 命令行入口文件
    ```
