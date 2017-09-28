---
title: PHP 依赖管理工具 Composer 入门
date: 2017-09-28 23:16:38
description: 粗粒度了解 Composer, 及快速安装；
tags:
- PHP-Plugin
categories:
- PHP
---

# 阅读链接
- [最详细的命令行教程](http://docs.phpcomposer.com/03-cli.html)
- [Composer入门文章，介绍 autoload 字段](http://www.cnblogs.com/XACOOL/p/5627444.html)

# 安装

安装前请务必确保已经正确安装了 `PHP`。打开命令行窗口并执行 `php -v` 查看是否正确输出版本号。

**第一步**，下载安装脚本 `composer-setup.php ` 到当前目录：
```
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
```

**第二步**，执行安装过程：
```
php composer-setup.php
```

**第三步**，删除安装脚本：
```
php -r "unlink('composer-setup.php');"
```

**第四步（可选）**，全局安装：
```
sudo mv composer.phar /usr/local/bin/composer
```

# 配置镜像源
## 中国全量镜像源
**全局使用**
```
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

可以在`~/.composer/composer.json`文件中看到`repositories`的值；

**项目单独使用**
如果仅限当前工程使用镜像，去掉 `-g` 即可，如下： 

```
composer config repo.packagist composer https://packagist.phpcomposer.com
```

**取消镜像**

```
composer config -g --unset repos.packagist
```

## Laravel China 镜像源
**全局使用**
```
composer config -g repo.packagist composer https://packagist.laravel-china.org
```

**项目单独使用**
如果仅限当前工程使用镜像，去掉 `-g` 即可，如下： 

```
composer config repo.packagist composer https://packagist.laravel-china.org
```

**取消镜像**

```
composer config -g --unset repos.packagist
```



