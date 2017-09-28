---
title: HTTP响应可视化测试工具 httpstat
date: 2017-09-28 17:42:00
description: 体验 HTTP响应可视化测试工具 httpstat
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

`httpstat`是一款可以测试`http`状态的可视化工具，通过这个工具可以看出来`http`响应信息。包括`dns`解析、`tcp`连接等信息，`httpstat`一共有`golang`版本和`python`版本。安装及使用，移步到对应的`github`地址；

- `golang版本`：[https://github.com/davecheney/httpstat](https://github.com/davecheney/httpstat)
- `python版本`：[https://github.com/reorx/httpstat](https://github.com/reorx/httpstat)

> 说明，如果打算使用`python`版本的`httpstat`，请确保`python`环境至少是`2.7`，因为代码中使用了列表推导式，在`python2.7`之后才支持该语法；

# Demo
笔者拿该命令测试博客站点的响应，效果图很直观！

```
python httpstat.py greenlightt.github.io
```
![此处输入链接的描述][1]



  [1]: http://owk2q4gs5.bkt.clouddn.com/httpstat-blog.png
