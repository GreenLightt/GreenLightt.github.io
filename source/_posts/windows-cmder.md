---
title: cmder 常用命令
date: 2017-10-16 21:24:41
description: Windows 下命令利器
tags:
categories:
- Windows Tool
copyright: false
---


`cmder` 是一个跨平台的命令行增强工具，可以集成`windows batch`, `power shell`, `git`, `linux bash`等多种命令行于一体；详情见[官网](http://cmder.net/)；

# 快捷键
`ctrl` + `t`：快速新建一个 `tab`框；
`ctrl` + `w`：快速关闭一个 `tab`框；
`alt` + `enter`：全屏；
`ctrl` + &#96;：快速打开或关闭 `cmder` 面板；
`shift` + 鼠标左键拖动：复制选择文本；
`win` + `alt` + `p` 键快速打开 `settings`页面；


# Settings

## 显示中文
如果当前目录下存在中文文件，`ls` 会显示乱码，解决的方法也简单，就是： 打开设置，在 `startup` -> `environment` 中输入： 

```
set LANG=zh_CN.UTF-8
```
## 中文文字乱码重叠
打开设置，在 `main` 中，取消勾选 `Monospace` 选项；

## 设置默认打开目录
打开设置，在 `startup` -> `tasks` 中，修改 `{cmd::Cmder}`项，把

```
cmd /k "%ConEmuDir%\..\init.bat"  -new_console:d:%USERPROFILE%
``` 
替换为 
```
cmd /k "%ConEmuDir%\..\init.bat"  -new_console:d:D:\vscode_workspace
```
笔者，将默认打开目录设置为 `D:\vscode_workspace`

## 关闭启动时自动更新
打开设置，在 `main` -> `update` 中，

`Do automatic check on`选项中取消 `startup` 选项；

## 修改背景透明色
打开设置，在 `features` -> `transparency` 中， 调整 `transparent` 指标的位置；