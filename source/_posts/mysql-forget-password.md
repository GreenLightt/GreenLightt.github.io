---
title: Mysql 服务器管理员忘记密码
date: 2017-11-11 01:01:41
description: Mysql 服务器管理员忘记密码
tags:
- Mysql
categories:
---

1. 先停止 `mysql` 服务

  ```
  service mysqld stop
  ```
2. 然后打开 `mysql` 配置文件 `/etc/my.cnf`, 在【`mysqld`】下面添加一行代码：`skip-grant-tables`。这行代码意思就是跳过跳过授权表，即是可以跳过密码验证直接进入数据库。
3. 重启 `mysql` 数据库。假如不重启的话，不会生效。并进入 `mysql`
  
  ```
  service mysqld restart
  mysql -uroot -p
  ```
4. 更改 `root` 密码。

  ```
  update user set password=password('123456') where user="root";
  ```
  用户选 `root` ,可以随便更改成任意密码，我这里设置的`123456`，`password()`是`mysql`密码加密的一个函数。
5. 刷新下密码，使更改的生效

  ```
  flush privileges; 
  ```
6. 退出数据库。

  ```
  exit
  ```
7. 退出数据库，重新登录

  ```
  mysql -uroot -p   // 回车输入刚刚更改的密码
  ```
  然后再次进入配置文件 `/etc/my.cnf` 把 `skip-grant-tables` 去掉。






