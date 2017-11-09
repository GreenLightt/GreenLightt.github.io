---
title: Mysql 用户与权限
date: 2017-11-11 00:34:41
description: 整理 Mysql 用户与权限
tags:
- Mysql
categories:
---

# 用户管理

**查看**服务器上的用户

```
USE mysql;

SELECT host, user FROM USER;
```
注： `host` 显示值： `::1` 是 `IPv6` 下的本地回环地址；`%` 是任意 `IP`；

**创建**
`identified by` 会将纯文本密码加密作为散列值存储
```
CREATE USER 【用户名】 IDENTIFIED BY '【密码】';
```
这里的【用户名】可以是  用户名（等价于 用户名@'%'） 或者 用户名@'`IP`'；

**重命名**用户名
```
RENAME USER 【原用户名】 TO 【新用户名】;
```
**修改**密码
```
UPDATE mysql.user
SET password=PASSWORD('【密码】')
WHERE user='【用户名】';
```
**删除**
```
DROP USER test_root;
```

# 权限管理
查看用户权限
```
SHOW GRANTS FOR 【用户名】;
```
> 设置权限时必须给出一下信息
1. 要授予的权限
2. 被授予访问权限的数据库或表
3. 用户名

赋予权限，比如将 `test` 数据库的所有操作权限给 `test_admin`；

```
GRANT SELECT, INSERT, UPDATE, DELETE ON test.* TO test_admin;
```

回收权限，比如 `test_admin` 用户撤销 `delete` 权限：

```
REVOKE DELETE ON test.* FROM test_admin;
```

注： 如果希望立即看到结果 `flush  privileges;`


更多的示例与 `demo`， 参考[mysql 用户管理和权限设置](https://www.cnblogs.com/fslnet/p/3143344.html)

# 附录
## 服务器支持远程访问
修改 `my.cnf` 文件的 `bind-address` 值为 `0.0.0.0`

```
bind-address=0.0.0.0
```




