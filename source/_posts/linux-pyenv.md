---
title: Linux 下 python 版本管理工具 pyenv
date: 2017-10-09 14:35:41
description: 总结 pyenv 的使用
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

# 简介
`pyenv`是一个简单的`python`版本管理工具，前身为`pythonbrew`，`pyenv`允许你改变全局的`python`版本，安装多种不同的`python`版本，设置应用指定的`python`版本以及创建/管理虚拟的`python`环境；

# 安装
`Centos6.8`上安装`pyenv`，先安装依赖

```
yum -y install readline readline-devel readline-static openssl openssl-devel openssl-static sqlite-devel bzip2-devel bzip2-libs
```
安装`pyenv`:
```
curl -L https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash
```

配置环境变量，并添加 `pyenv` 初始化到你的`shell`，
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
```

# 常用命令

1. 查看当前设置默认的 `python` 版本

  ```
  pyenv version
  ```
2. 查看当前已安装的所有`python`版本
 
  ```
  pyenv versions
  ```
3. 查看当前可安装的`python`版本列表

  ```
  pyenv install -l | less
  ```
4. 安装指定版本的`python`

  ```
  pyenv install 2.7.1 
  ```
5. 设置全局`python`版本

  ```
  pyenv global <python版本>
  ```
6. 设置局部 `Python` 版本 
  设置之后可以在目录内外分别试下`which python`或`python --version`看看效果, 如果没变化的话可以`pyenv rehash`之后再试试

  ```
  pyenv local <python版本>
  ```

# `python`版本本地安装
以本地安装 `2.7.1` 版本为例，

1. 通过执行`pyenv install 2.7.1 -v`获取下载地址，然后通过浏览器下载文件，在目录`/tmp`准备`Python-2.7.1.tgz`文件；
2. 然后在 `/tmp`下启动一个简单的 `httpserver`
  
  ```
  cd /tmp
  python -m SimpleHTTPServer 4999  
  ```
3. 在执行`pyenv install 2.7.1`之前要先添加一个环境变量

  ```
  export PYTHON_BUILD_MIRROR_URL="http://127.0.0.1:4999/"
  pyenv install 2.7.1 -v
  ```

4. 从`httpserver`的打印`log`中发现收到的请求是一个字符串
"`HEAD /ca13e7b1860821494f70de017202283ad73b1fb7bd88586401c54ef958226ec8 HTTP/1.1`"，我们要把`Python-2.7.1.tgz`复制一份到这个字符串为名的文件，然后重启`httpserver`，最后用`pyenv`即可安装

  ```
  cp Python-2.7.1.tgz ca13e7b1860821494f70de017202283ad73b1fb7bd88586401c54ef958226ec8
  
  python -m SimpleHTTPServer 4999  
  
  pyenv install 2.7.1
  ```

# FAQ
## 支持 YouCompleteMe
需要在 `.bashrc` 加入下面命令：

```
export PYTHON_CONFIGURE_OPTS="--enable-shared"
```







