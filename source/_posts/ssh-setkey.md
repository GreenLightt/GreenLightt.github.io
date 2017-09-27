---
title: ssh 公密钥设置标准步骤
date: 2017-09-26 22:58:41
description: 介绍如何规范化设置 ssh 公密钥
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

标题：

# 功能介绍
通过`ssh-keygen`命令来生成`ssh`公钥密钥，帮助我们可以免密进行`ssh`连接别的主机或者进行认证注册操作（比如`github`，这样后续一些需要权限的操作，可以不需要密码确认）；


# 应用：单向登录
以免密登录`A`服务器为例；

**第一步，执行以下命令，生成公密钥文件**

```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa_192.168.1.169
```
说明，`-t`参数是指定密钥类型，如果没有指定则默认生成用于`SSH-2`的`RSA`密钥；`-P`提供密语（我们希望是免密登录，所以置空）；`-f`参数指定密钥文件名（建议命名上采用`id_rsa_<对方的IP地址或HostName>`）；

**第二步，配置**
如果文件名不是采用默认的`id_rsa`，都需要在`~/.ssh/config`文件中进行配置；
```
Host 192.168.1.169
    HostName 192.168.1.169
    User root
    PubkeyAuthentication yes
    IdentityFile ~/.ssh/id_rsa_192.168.1.169
```

说明,

- `Host`：别名，默认是同`HostName`一致
- `HostName`：服务器的`IP`地址
- `User`：登录的用户名
- `PubkeyAuthentication`：是否允许客户端通过`public-key authentication`来登陆
- `IdentityFile`：密钥的路径

**第三步，将`id_rsa_192.168.1.169.pub`文件中的内容追加到`A`服务器上的`~/.ssh/authorized_keys`文件中**

先`scp`到`A`服务器的`tmp`目录下，再进行追加操作；
```
// 本地服务器上执行 scp
scp ~/.ssh/id_rsa_192.168.1.169.pub 192.168.1.169:/tmp

// A 服务器执行追加操作
cat 192.168.1.169:/tmp/id_rsa_192.168.1.169.pub >> ~/.ssh/authorized_keys
```

**第四步，验证**
```
ssh root@192.168.1.169
```
登录成功！


# 应用：认证
以认证`github`为例；

**第一步，执行以下命令，生成公密钥文件**

```
ssh-keygen -t rsa -P '' -C "your_email@example.com" -f ~/.ssh/id_rsa_github
```
说明，`-t`参数是指定密钥类型，如果没有指定则默认生成用于`SSH-2`的`RSA`密钥；`-P`提供密语（我们希望是免密登录，所以置空）；`-f`参数指定密钥文件名（建议命名上采用`id_rsa_<平台名>`）；`-C`参数

这时候会生成两个文件，`id_rsa_github`和`id_rsa_github.pub`；

**第二步，配置**
如果文件名不是采用默认的`id_rsa`，都需要在`~/.ssh/config`文件中进行配置；
```
Host github.com
    HostName github.com
    User git
    PubkeyAuthentication yes
    IdentityFile ~/.ssh/id_rsa_github
```
说明,

- `Host`：别名，默认是同`HostName`一致
- `HostName`：平台的域名
- `User`：登录的用户名，认证一般是`git`
- `PubkeyAuthentication`：是否允许客户端通过`public-key authentication`来登陆
- `IdentityFile`：密钥的路径

**第三步，将`id_rsa_coding.pub`中的文件内容，放到`github`平台上的`SSH and GPG keys`上**

用`cat`命令直接输出文件内容，方便复制粘贴
```
cat ~/.ssh/id_rsa_github.pub
```

**第四步，验证**

```
ssh -T git@github.com
```
说明，`-T`参数用于测试连接，并不真正登录；

![验证成功][1]


  [1]: http://owk2q4gs5.bkt.clouddn.com/sucess-test-ssh.png
