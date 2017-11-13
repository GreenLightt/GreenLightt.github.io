---
title: Python 虚拟环境工具 virtualenv
date: 2017-11-13 15:19:41
description: Python 虚拟环境工具 virtualenv 相关命令及参考阅读
tags:
categories:
- Python
copyright: false
---

# 常用命令

## 创建 virtualenv
进行指定项目根目录，执行

```
virtualenv venv
```
然后，会在项目根目录下有一个`venv`文件夹。这个文件夹下，`bin`, `bin/python`存放在当前环境是使用的 `python` 解释器；所有安装的 `python` 库都会放在这个目录中的 `lib/pythonx.x/site-packages/` 下。

### 指定 python 版本
```
#创建python2.7虚拟环境
➜  Test git:(master) ✗ virtualenv -p /usr/bin/python2.7 ENV2.7
Running virtualenv with interpreter /usr/bin/python2.7
New python executable in ENV2.7/bin/python
Installing setuptools, pip...done.
```

```
#创建python3.4虚拟环境
➜  Test git:(master) ✗ virtualenv -p /usr/local/bin/python3.4 ENV3.4
Running virtualenv with interpreter /usr/local/bin/python3.4
Using base prefix '/Library/Frameworks/Python.framework/Versions/3.4'
New python executable in ENV3.4/bin/python3.4
Also creating executable in ENV3.4/bin/python
Installing setuptools, pip...done.
```

## 激活
`Windows`平台
```
venv\scripts\activate
```

`Linux`平台
```
. venv/bin/activate
```

## 关闭 virtualenv
```
deactivate
```


# 参考阅读

- [廖雪峰的官方网站 virtualenv](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432712108300322c61f256c74803b43bfd65c6f8d0d0000)




