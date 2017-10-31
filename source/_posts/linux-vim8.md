---
title: Centos 安装 vim 8.0
date: 2017-10-31 17:20:00
description: 整理 Centos 安装 vim 8.0
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

# 编译安装如下

```
yum install ncurses-devel 
wget https://github.com/vim/vim/archive/master.zip 
unzip master.zip 
cd vim-master 
cd src/ 
./configure --enable-multibyte --enable-pythoninterp=yes
make 
sudo make install 
vim --version
```

# 检查 `python` 支持
检查 `vim` 是否有 `python` 支持，命令如下：

```
vim --version |grep python
```

重新编译的时候，想要支持 `python2.x`，则加入 `--enable-pythoninterp=yes` 参数。如果想开启 `Python3` 支持，则 `--enable-python3interp=yes`。



