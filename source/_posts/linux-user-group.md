---
title: Linux 账号管理
date: 2018-01-19 23:36:00
description: 总结 Linux 下用户与用户组相关知识点与命令
tags:
categories:
- Linux
copyright: false
---

# 概念

**用户标识符**

每个登录的用户至少都有两个 `ID`，一个是用户 `ID`（`UID`）， 另一个是用户组 `ID`（`GID`）;

用户 `ID` 信息存放在 `/etc/passwd` 文件中；

用户组 `ID` 信息存放在 `/etc/group` 文件中；

用户密码表存放在 `/etc/shadow` 文件中；

## /etc/passwd 文件结构

每一行使用 “：” 来分隔开，共有七个字段，分别是

**1. 账号名称**
就是账号，用来对应 `UID`；

**2. 密码**
早期 `UNIX` 系统的密码就是放在这字段上，后来这个字段的密码数据改放到 `/etc/shadow` 中了，所以这里会显示 `x`; 

**3. UID**
用户标识符。

| `ID`范围 | 该 `ID` 用户特性 |
|: --- :|: --- :|
| 0 (系统管理员) | 当 `uid` 是 0 时，代表这个账号是 “系统管理员”！所以当你要让其他的<br/>账号名称也具有 `root` 的权限时，将该账号的 `UID` 改为 0 即可。<br/>这也就是说一个系统上的系统管理员不见得只有 `root`。<br/>不过，不建议有多个账户的 `uid` 是 0 |
| 1 ~ 499 (系统账号) |  根据系统账号的由来，通常系统账号又被分为两种： <br/>1 ~ 99： 由 `distributions` 自行创建的系统账号 <br/> 100 ~ 499：若用户有系统账号需求时，可以使用的账号 `uid` |
| 500 ~ 65535 (系统管理员) | 给一般用户用的。事实上，目前的 Linux 内核（2.6.x）版已经可以支持到 （2^32 - 1）这么大的 `uid` 号码 |

**4. GID**
这个与 `/etc/group` 有关。用来规定组名与 `GID` 对应关系。

**5. 用户信息列说明**
这个字段基本上并没有什么重要用途，只是用来解释这个账号的意义而已。不过，如果你提供使用 `finger` 的功能时，这个字段可以提供很多的信息。

**6. 主文件夹**
这是用户的主文件夹。

**7. Shell**
当用户登录系统后就会取得一个 `Shell` 来与系统的内核通信以进行用户的操作任务。

## /etc/group 文件结构

**1. 用户组名称**

**2. 用户组密码**
密码，已经移到 `/etc/gshadow` 文件中

**3. 用户组ID**

**4. 此用户组支持的账号名称**

# 命令

## 用户

### 新增

```
# 新增普通用户, 会在/home 下创建同名文件夹，权限为 700，会创建同名的用户组
useradd vbird1
# 指定用户组，指定 uid
useradd -u 700 -g users vbird2
# 创建系统账户，不会在/home 下创建同名文件夹，会创建同名的用户组，UID/GID 在 500 以内
useradd -r vbird3
```

`useradd` 基本账号设置值参考文件在 `/etc/default/useradd` 文件中，但也可以由命令 `useradd -D` 调出； 关于 `uid` / `gid` ， 在 `/etc/login.defs`;


### 修改密码

```
# 修改 vbird2 用户的密码
passwd vbird2
# 修改自己账号密码
passwd

# 部分 linux 版本支持非手动修改密码，参考 man passwd 确认是否支持
echo "newpassword" | passwd --stdin vbird2

# 查看密码相关参数信息
passwd -S vbird2
# 或 change -l vbird2

# 锁住用户密码，让其无法登录
passwd -l vbird2
# 解锁
passwd -u vbird2
```

### 修改用户信息

```
# 修改用户 vbird2 的说明列
usermod -c "Vbird 's test" vbird2

# 用户 vbird2 密码在 2009/12/31 失效
usermod -e "2009-12-31" vbird2
```
### 删除

```
# 删除 username 用户
userdel username

# 删除 vbird2 用户，包含主文件夹
userdel -r vbird2
```

### 查询相关 UID/GID 信息

```
# 查询 vbird2 用户的 uid / gid 等信息
id vbird1

# 查询 root 自己的相关 id 信息
id
```

### 查看当前用户支持的用户组

```
groups
```

### 查看登录系统的用户信息

```
w
#或者
who
```

查看登录系统的用户日志

```
last
# 或者
lastlog
```

### 用户对谈

> 语法：write 用户账号 [用户所在终端接口]

```
# 比如 `root` 用户向 `vbird1` 用户发消息
write vbird1 pts/2
```

如果 `vbird1` 不想要接收任何人的消息，执行 `mesg n`, 当然  `mesg` 对 `root` 发来的消息没有抵抗力；如果想要解开的话 `mesg y`; 

如果要对所有人群发信息，用 `wall` 命令；

```
wall "I will shutdonw my linux server..."
```

## 用户组

### 切换有效用户组

```
newgrp 新用户组名
```
### 新增用户组

```
# 新增名为 group1 的普通用户组
groupadd group1
# 新增名为 group2 的系统用户组
groupadd -r group2
```

### 修改用户组信息

```
# 将 group1 改名为 mygroup
groupmod -n mygroup group1
```

### 删除用户组

```
# 删除用户组 mygroup
groupdel mygroup
```

### 用户组管理员

示例，

以下是 `root` 用户执行

```
# 新建用户组
groupadd testgroup
# 给用户组一个密码
gpasswd testgroup
# testgroup 用户组添加 vbird1 用户为组管理员
gpasswd -A vbird1 testgroup
# 查询 testgroup 用户组信息
grep testgroup /etc/group /etc/gshadow
```

`vbird1` 用户登录

```
# 查询用户信息
id
# 添加 vbird1、vbird2 进 testgroup 用户组
gpasswd -a vbird1 testgroup
gpasswd -a vbird2 testgroup
```

