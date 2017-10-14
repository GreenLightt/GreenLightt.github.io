---
title: Windows 上 Git 安装部署
date: 2017-10-14 16:10:41
description: 总结 Windows 上安装 Git
tags:
- Git
categories:
- Version Management
copyright: false
---

# 下载
打开 [http://git-scm.com/download/win](http://git-scm.com/download/win)，下载会自动开始。 

笔者安装的是 32 位的客户端，所以默认安装位置在 `C:\Program Files (x86)\Git`;

# 公密钥生成

## 生成
进入 `C:\Program Files (x86)\Git\bin`目录，双击 `bash` 命令；在打开的图形界面中键入

```
ssh-keygen -t rsa -C "your_email@example.com"
```

1. 提示你把新生成的 `id_rsa` 存放到哪里，此处默认会存放在c盘的用户名下的 `.ssh` 文件夹下并默认名为 `id_rsa`，注意防止命名重复导致覆盖；
2. 在输入了路径后，会提示你输入提交项目时输入的验证密码，不输则表示不用密码；

## 添加
默认 `SSH` 只会读取 `id_rsa`，所以为了让 `SSH` 识别新的私钥，需要将其添加到 `SSH agent`
```
ssh-add ~/.ssh/id_rsa_github
```

如果报错：`Could not open a connection to your authentication agent.` 即无法连接到`ssh agent`；
则执行 `ssh-agent bash` 命令后再执行 `ssh-add` 命令

## 配置

c盘的用户名下的 `.ssh` 文件夹下如果不存在 `config` 文件，即创建；

内容如下

```
Host github.com
    HostName github.com
    User git
    PubkeyAuthentication yes
    IdentityFile C:\Users\Administrator\.ssh\id_rsa_github

Host git.coding.net 
    HostName git.coding.net 
    User git
    PubkeyAuthentication yes
    IdentityFile C:\Users\Administrator\.ssh\id_rsa_codingnet
```

## 测试
使用命令：
```
ssh -T git@github.com
```
