---
title: 如何从头开始写一个 PHP 项目
date: 2017-12-15 20:07:38
description: 整理不用框架的 PHP 项目的流程
tags:
categories:
- PHP
---

本文是基于一个从头开始写的 `PHP` 项目 `mg_food` 进行阅读学习，该项目未用到任何框架；通过阅读该[项目源码](https://github.com/hytcxuguojian/mg_food)，有利于了解纯 `PHP` 项目的流程；


**界面展示**
```
<?php
    // 引入简易数据库操作类
    include_once "ez_sql_core.php";
    include_once "ez_sql_mysql.php";
?>

<?php
    // 定义类似配置的一些常量，这里简约化了，可以引入 phpdotenv 第三方读取 env 文件形式进行配置
    define('DB_HOST', 'localhost');
    ...
?>

<?php
    // 启用会话
    session_start();
?>

<?php
    // 预置功能代码
    
    // 检查用户是否登录，否则跳转到登录页
    function checkLogin() {
        if (empty($_SESSION["user"])) {
            header("location:login.php?error=needlogin");
        }
    }
    
    // 退出登录
    function logOut() {
        $_SESSION["user"] = null;
    }
    
    ...... 其他功能性代码
?>

<!DOCTYPE html>
   ...
   ...
   ...
</html>
```

**`ajax` 请求的接口文件**

```
// 获取请求动作
$action = isset($_POST["action"]) ? $_POST["action"] : "";

// 登录
if ($action == "login") {
   ......
}

// 确认商品
if ($action == "ensure") {
   ......
   echo json_encode($data);
}
```

**总结**：
1. 没有使用框架的项目一般都是通过 `$_` 这种超全局变量做数据周转；
2. `response` 返回是跳转，则使用 `header` 方法；返回的是 `ajax` 的 `json` 数值，则通过 `echo json_encode($data)`  方法；





