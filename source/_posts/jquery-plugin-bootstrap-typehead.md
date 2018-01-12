---
title: Jquery 文本自动补全插件 Bootstrap-Typehead
date: 2018-01-12 16:26:41
description: 整理关于 Bootstrap-Typehead 的常见问题
tags:
- Jquery-Plugin
categories:
copyright: false
---

`github` 仓库地址： [https://github.com/tcrosen/twitter-bootstrap-typeahead](https://github.com/tcrosen/twitter-bootstrap-typeahead)

# 介绍
文本输入框，根据已经键入的文本，模糊搜索，给出自动补全提示。

![此处输入图片的描述][1]

# FAQ

## 光标聚焦，自动弹出

```
$(function () {
    // 指定文本输入框 初始化 自动补全
    var typeahead_ele = $('input[name="xx"]').typeahead({..option..});

    // 光标聚焦注册事件
    $('input[name="xx"]').focus(function() {
        var typeahead = typeahead_ele.data('typeahead');
        typeahead.query = ''; 
        typeahead.ajaxExecute('');
    });

    // 光标聚焦
    $('input[name="xx"]')[0].focus();

});
```

  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180112160854.png
