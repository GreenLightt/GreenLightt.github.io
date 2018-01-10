---
title: CodeIgniter 框架初次上手
date: 2018-01-10 10:22:41
description: ci 框架初次上手做记录
tags:
- CodeIgniter
categories:
copyright: false
---


# 本地部署及运行

**下载代码**
[CodeIgniter 中国 下载](http://codeigniter.org.cn/user_guide/installation/downloads.html)页面罗列了各个版本的 `CI` 源代码； 找到最新的版本并下载；

执行 `composer install` 初始化安装依赖；

在 `Nginx` 上配置：

```
     server {                                                                                           
         listen       9050;                        
         server_name  localhost;            
                                                                                                        
         proxy_buffer_size   128k;               
         proxy_buffers   8 256k;                          
         proxy_busy_buffers_size   256k;                           
         client_max_body_size 50M;                                 
         root /opt/ci;                                      
                                                                                                        
         location / {                                     
             if (-f $request_filename/index.html){                  
                     rewrite (.*) $1/index.html break;                        
             }                          
                                                                                                        
             if (-f $request_filename/index.php){                     
                     rewrite (.*) $1/index.php;              
             }                                   
                                                                                                        
             if (!-f $request_filename){    
                     rewrite (.*) /index.php;
             }
             index index.php index.html index.htm;
             try_files $uri $uri/index.php;
         }                                                                                              
         location ~* \.(eot|otf|ttf|woff|woff2)$ {
             add_header Access-Control-Allow-Origin *;
         }                                                                                              
         rewrite ^/(.*\.php)(/)(.*)$ /$1?file=/$3 last;
                                                                                                        
         include "/opt/bksite/nginx/conf/bitnami/phpfastcgi.conf";
         include "/opt/bksite/nginx/conf/bitnami/bitnami-apps-prefix.conf";
     }
```



成功页面如下：

![此处输入图片的描述][1]


  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180109182532.png
  
# 配置

**设置网站 `base_url`**

在文件 `application/config/config.php` 上；如果不设置，默认是服务器的 `IP` 地址，没有端口信息；

```
$config['base_url'] = 'http://172.16.0.43:9050';
```

**开启 Composer 自动加载**

在文件 `application/config/config.php` 上；

```
$config['composer_autoload'] = TRUE;
```


