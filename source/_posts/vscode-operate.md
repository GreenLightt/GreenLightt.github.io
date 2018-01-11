---
title: Vscode 日常操作
date: 2018-01-11 21:44:41
description: 整理 Vscode 日常操作
tags:
categories:
- Editor
copyright: false
---

# Python
## Python 文件首行编码
```
# -*- coding: UTF-8 -*-
```

## 安装自动格式化代码
在命令行上键入:
```
python -m pip install yapf  
```
`VSCode` “自动格式化代码”的快捷键是“`Alt + Shift + F`”。

## 安装静态代码检查工具 Flake8
在命令行上键入:
```
python -m pip install flake8
```

# 插件
`Ctrl` + `Shift` + `x` 打开扩展页面；

- 文件图标主题: `vscode-icons`
  激活位置: 文件 - 首选项 - 文件图标主题
- 缩进线插件: `Guides`

# 快捷键

## 打开
| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+K` + `Ctrl+S` | 打开快捷键面板  |
| `Ctrl+K` + `Ctrl+O` | 打开文件夹  |

## 屏幕移动

| 按键 | 描述 |
|: --- :|: --- :|
| `Home` / `End`  | 转到 行首 / 行尾  |
| `Ctrl+Home` / `Ctrl+End`  | 转到 文件开始 / 文件末尾  |
| `Alt+PgUp` / `Alt+PgDown` | 向 上/下 滚动页面 |

## 光标移动

| 按键 | 描述 |
|: --- :|: --- :|
| `UpArrow` / `DownArrow` / `LeftArrow` / `RightArrow`  | 向 上/下/左/右 移动  |
| `Ctrl+Shift+\` |  跳到匹配的括号 |

## 多光标和选择

| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+F2` | 选择当前字的所有出现 |
| `Ctrl+Shift+L` | 选择当前选择的所有出现 |
| `Shift+Alt+（拖动鼠标）` | 列（框）选择 |
| `Ctrl+Shift+Alt+（箭头键）` | 列（框）选择 |

## 行移动

| 按键 | 描述 |
|: --- :|: --- :|
| `Alt+UpArrow` / `Alt+DownArrow` | 向 上/下 移动行  |
| `Shift+Alt+UpArrow` / `Shift+Alt+DownArrow` | 向 上/下 复制行  |
| `Ctrl+Enter` / `Ctrl+Shift+Enter` | 在下面/上面 插入行 |
| `Ctrl+Shift+K` | 删除行  |

## 展开折叠

| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+K` + `Ctrl+[` / `Ctrl+K` + `Ctrl+]` | 局部折叠/展开  |
| `Ctrl+K` + `Ctrl+0` / `Ctrl+K` + `Ctrl+J` | 全部折叠/展开  |

## 注释

| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+/` | 切换行注释  |
| `Ctrl+K` + `Ctrl+C` / `Ctrl+K` + `Ctrl+U` | 选中区域 添加/删除 行注释符号  |
| `Shift+Alt+A` | 切换块注释 |

## 搜索与替换

| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+F` | 查找  |
| `Ctrl+H` | 替换  |
| `F3` / `Shift+F3` | 查找 下一个/上一个  |

## 复制 粘贴 撤销

| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+C` / `Ctrl+X` | 复制 / 剪切  |
| `Ctrl+V` | 粘贴  |
| `Ctrl+Z` / `Ctrl+Shift+Z` | 撤销 / 反撤销  |

## 格式化

| 按键 | 描述 |
|: --- :|: --- :|
| `Shift+Alt+F` | 格式化文档  |
| `Ctrl+K` + `Ctrl+F` | 格式化选定区域  |
| `Ctrl+K` + `Ctrl+X` | 修剪尾随空格  |

## 引用

| 按键 | 描述 |
|: --- :|: --- :|
| `F12` | 转到定义  |
| `Shift+F12` | 显示引用 |

## 调试

| 按键 | 描述 |
|: --- :|: --- :|
| `Ctrl+Shift+D` | 显示调试  |

# FAQ
## 系统上禁止运行脚本

执行命令
```
set-executionpolicy remotesigned
```
详细阅读参考[PowerShell因为在此系统中禁止执行脚本解决方法](https://www.cnblogs.com/zhaozhan/archive/2012/06/01/2529384.html)








