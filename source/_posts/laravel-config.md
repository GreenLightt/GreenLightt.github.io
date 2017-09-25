---
title: Laravel 大将之 配置 模块
date: 2017-06-04 17:57:41
description: 介绍如何使用配置模块
tags:
- Laravel-5.4
categories:
- Laravel
---

# 简介
`Laravel`的配置管理模块统一管理项目所需的配置，包括插件所需的配置，从而为容器服务提供支持；

根据`src/Illuminate/Foundation/Bootstrap/LoadConfiguration.php`文件,可以看出`Lavavel`的配置加载流程如下：
1. 判断是否存在配置缓存文件（位于项目根目录下的`bootstrap/cache/config.php`）, 如果存在，则读取缓存文件内容，并生成`Illuminate\Config\Repository`实例，在服务容器绑定`config`服务；
2. 如果不存在缓存文件，则先生成`Illuminate\Config\Repository`实例，在服务容器绑定`config`服务，再遍历项目根目录下的`config`文件，保存其配置数据内容；

# 使用
1. 生成配置缓存文件；通过`artisan`命令，在项目根目录下执行

  ```
    php artisan config:cache
  ```
2. 清空配置缓存文件

  ```
    php artisan config:clear
  ```
3. 读取配置内容；比如读取`config/app.php`文件的`locale`参数;调用全局帮助函数，直接从`Illuminate\Config\Repository`实例中获取参数值；

  ```
    config('app.locale');
  ```
