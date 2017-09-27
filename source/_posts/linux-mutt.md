---
title: Linux 下邮件发送工具 mutt
date: 2017-09-27 23:21:00
description: 总结 mutt 的使用
tags:
- Linux Tool
categories:
- Linux
copyright: false
---

# 

# 使用
**正文只包含文字**

`-s`参数指定邮件主题

```
echo "正文" | mutt -s "Linux测试发送邮件" xxx@qq.com
```

或者，通过读取文件内容作正文主体

```
cat xx.txt  | mutt -s "Linux测试发送邮件" xxx@qq.com
```

**正文是HTML内容**
```
mutt xxx@qq.com -s "Linux测试发送邮件" -e 'set content_type="text/html"'  < /tmp/hello.html
```

**带附件**

`-a`携带附件；

```
mutt xxx@qq.com -s "Linux测试发送邮件" -e 'set content_type="text/html"' -a /tmp/phpcs_phpexcel.log -a /tmp/phpcs2.log < /tmp/hello.html 
```

> 日志查看：`centos`虚拟机可以在`/var/log/maillog`文件中查看发送邮件日志信息；


# 语法
> mutt [-hnpRvxz][-a<文件>][-b<地址>][-c<地址>][-f<邮件文件>][-F<配置文件>][-H<邮件草稿>][-i<文件>][-m<类型>] [-s<主题>][邮件地址]

| 参数  | 描述 |
|: --- :|: --- :|
|`-a`　<文件> |　在邮件中加上附加文件。|
|`-b`　<地址> |　指定密件副本的收信人地址。 |
|`-c`　<地址> |　指定副本的收信人地址。 |
|`-f`　<邮件文件>　| 指定要载入的邮件文件。 |
|`-F`　<配置文件>　| 指定`mutt`程序的设置文件，而不读取预设的`.muttrc`文件。 |
|`-h`　| 显示帮助。 |
|`-H`　<邮件草稿>　| 将指定的邮件草稿送出。 |
|`-i`　<文件>　| 将指定文件插入邮件内文中。 |
|`-m` <类型>　| 指定预设的邮件信箱类型。 |
|`-n`　| 不要去读取程序培植文件(`/etc/Muttrc`)。 |
|`-p`　| 在`mutt`中编辑完邮件后，而不想将邮件立即送出，可将该邮件暂缓寄出。 |
|`-R`　| 以只读的方式开启邮件文件。 |
|`-s`　| <主题>　指定邮件的主题。 |
|`-v`　| 显示`mutt`的版本信息以及当初编译此文件时所给予的参数。 |
|`-x`　| 模拟`mailx`的编辑方式。 |
|`-z`　| 与`-f`参数一并使用时，若邮件文件中没有邮件即不启动`mutt`。 |

# 安装
```
yum -y install mutt
```

创建或编辑配置文件`/root/.muttrc`

```
# 如果你收到的邮件乱码，设置以下信息
set charset="utf-8"
set rfc2047_parameters=yes
# 如果你想自定义发件人信息，需要进行如下设置
#set envelope_from=yes
#set use_from=yes
#set from=root@itdhz.com
#set realname="itdhz"
```

# FAQ
1. 邮件发出去，但收不到；
 默认情况下`mutt`的发件人格式是`用户名@主机名.主机域名`，如`root@pr.net`，这种格式有可能被邮件服务器认为是垃圾邮件。可以通过修改发件人信息，来避免被误认为是垃圾邮件。


