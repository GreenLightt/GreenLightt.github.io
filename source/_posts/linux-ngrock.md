---
title: Linux 下内网映射工具 ngrock
date: 2017-10-17 18:12:00
description: 体验Linux 下内网映射工具 ngrock
tags:
- Linux Tool
categories:
- Linux
copyright: false
---


`ngrok` 是非常流行的反向代理服务，可以进行内网穿透，支持 80 端口以及自定义 `tcp` 端口转发。这样你就可以运行本地的程序，而让别人通过公网访问了。比如微信公众号开发的时候，需要接入一个外网的`IP`地址，由于我们在自己的电脑上需要开发，测试很不方便，不可能每次都把代码上传到服务器，测试一次。

按照[官网](https://ngrok.com/)下载并安装，如图所示：

![此处输入链接的描述][1]

运行时，指定`http`端口号，即可；对暴露的公网`IP`进行访问，相当于访问本机器的此端口应用；

![此处输入链接的描述][2]


  [1]: http://owk2q4gs5.bkt.clouddn.com/ngrock.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/ngrock-http8099.png