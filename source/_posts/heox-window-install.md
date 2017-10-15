---
title: Windows 上 hexo 安装
date: 2017-10-15 13:28:41
description: Windows 上 hexo 安装
tags:
- Hexo
copyright: false
---

# 环境准备
全局安装 `hexo`， 需要准备环境  `node.js`、 `git`；

## Node.js
从[Node.js 官网下载地址](https://nodejs.org/en/)，下载，双击安装；

进入 `cmd` 环境下，键入 `node -v` 查看是否安装成功；

## git
参考之前写的文章 [Windows 上 Git 安装部署](https://greenlightt.github.io/2017/10/14/git-window-install/)

# 开始安装
鼠标右击 `Git Bash Here`，进入 `Bash` 环境，
先修改安装源

```
npm config set registry https://registry.npm.taobao.org
npm info underscore
```

然后，键入

```
npm install -g hexo
```

![此处输入链接的描述][1]


  [1]: http://owk2q4gs5.bkt.clouddn.com/windows+hexo.png