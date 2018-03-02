---
title: Vbox 共享文件夹
date: 2018-03-02 18:56:41
description: 记录 Vbox 共享文件夹的步骤
tags:
- VBox
categories:
copyright: false
---

# 安装增强工具

步骤：

1. 找到 `Locate VBoxGuestAdditions.iso` 在主机上的位置 （以我为例, 在 `C:\Program Files\Oracle\VirtualBox\VBoxGuestAdditions.iso`）
2. 将 `VBoxGuestAdditions.iso` 文件传送到虚拟机里 （以我为例，用的是 `XShell` 和 `lszrz`）
3. 在虚拟机终端上，挂载 `ISO` 文件

   ```
   sudo mkdir /media/GuestAdditionsISO
   sudo mount -o loop path/to/VBoxGuestAdditions.iso /media/GuestAdditionsISO
   ```
4. 此时，你可能得到一个消息：`ISO` 已经被挂载为只读；如果你改变目录到 `/media/GuestAdditionsISO`，你可以看到 `VBoxLinuxAdditions.run` 并且可被执行

    ```
    cd /media/GuestAdditionsISO
    ls -l
    ```
    
5. 现在执行 `VBoxLinuxAdditions.run` 文件:

    ```
    sudo ./VBoxLinuxAdditions.run
    ```
    
# 共享文件夹

![此处输入图片的描述][1]

![此处输入图片的描述][2]

切换到 `root` 用户输入挂载命令：

> 格式为： `sudo mount -t vboxsf` 共享文件夹名称（在设置页面设置的） 挂载的目录

```
mkdir /opt/workspace
sudo mount -t vboxsf vbox_connect_workspace /opt/workspace
```


  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180302152346.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180302152530.png
