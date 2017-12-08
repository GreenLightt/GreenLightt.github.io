---
title: Laravel 大将之 Mail 模块
date: 2017-12-08 11:45:41
description: 稍微整理关于 Mail 模块的笔记
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

本文是基于 `Laravel 5.1` 版本的 `Mail` 模块代码进行分析书写；

`Laravel` 基于目前流行的 `SwiftMailer` 库提供了一套干净清爽的邮件 `API`。`Laravel` 为 `SMTP`、`Mailgun`、`Mandrill`、`Amazon SES`、`PHP` 的 `mail` 函数，以及`sendmail`提供了驱动，从而允许你快速通过本地或云服务发送邮件。

# 文件结构
`Mail` 模块的文件格局及功能如下图所示：

![此处输入链接的描述][1]

在 `MailServiceProvider` 文件中，此服务提供者的 `register` 方法依次做了以下的事：

1. 向服务容器注入驱动管理服务 `swift.transport`，对应调用的类是 `TransportManager`；
2. 向服务容器注入邮件管理服务 `swift.mailer`，对应调用的类是 `Swift_Mailer`；
3. 向服务容器注入 `Laravel` 自定义邮件管理服务 `mailer`， 对应的类是 `Illuminate\Mail\Mailer`；
  首先，实例化 `Mailer` 对象，参数为 `$app['view']`、`$app['swift.mailer']`、 `$app['events']`；
  其次，给 `Mailer` 实例绑定 容器`$app` 、日志器`$app->make('Psr\Log\LoggerInterface')`、队列`$app['queue.connection']`对象；
  然后，从`config/mail.php` 读取 `from` 和 `to` 参数，如果存在，即绑定到 `Mailer`对象中；
  最后，根据 `config/mail.php`配置文件的 `pretend` 参数，修改 `Mailer`对象的 `pretending` 私有变量，如果该私有变量为 `true`，则不实际发送邮件，而是写到日志中，便于本地开发调试；

# Demo
准备一个用于发送文件的视图，内容如下

```
<body>                                                                                              
    {!! $content !!}                                                                                    
    <br>                                                                                                
    <br>                                                                                                                                                                                           
### 系统邮件   请勿回复 ###         

</body> 
```

具体调用代码片段：

```
Mail::send('mail.email', ['content' => 'Hi！请查收补货单。'], function ($message) use ($email, $file) {
    @$message->to([$email]); // 发送给收件人              
    @$message->subject('主题');                                                           
    @$message->attach($file); // 绑定附件                                                      
}); 
```

# 参考

- [[ Laravel 5.1 文档 ] 服务 —— 邮件](http://laravelacademy.org/post/213.html)



  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171208111125.png
