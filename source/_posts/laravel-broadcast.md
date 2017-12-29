---
title: Laravel 大将之 广播 模块
date: 2017-08-21 17:57:41
description: 介绍如何使用 广播 模块
tags:
- Laravel-5.4
categories:
- Laravel
copyright: false
---

本文是基于`Laravel 5.4`版本的广播模块代码进行分析书写；

官方 `API` 地址 [https://laravel.com/api/5.4/Illuminate/Broadcasting.html](https://laravel.com/api/5.4/Illuminate/Broadcasting.html)

# 简介
广播是指发送方发送一条消息，订阅频道的各个接收方都能及时收到消息；比如 `A`同学写了一篇文章，这时候 `B`同学在文章底下评论了，`A`同学在页面上是不用刷新就能收到提示`有文章被评论了`，这个本质上就是`A`同学收到了广播消息，这个广播消息是由`B`同学评论这个动作触发了发送广播消息；

在整个广播行为中，有一个重要的概念叫频道`channel`，频道的类型有

- 公共频道`public`
- 私有频道`private`
- 存在频道`presence`

移动端订阅了公共频道`public`，会直接提示成功；私有频道`private`和存在频道`presence`在进行订阅的过程中，会向服务器端发送权限验证，看是不是有权限可以订阅该频道；私有频道`private`和存在频道`presence`的区别在于，私有频道`private`能够接收其他成员发送的消息，而存在频道`presence`除此之外，还能够在用户的加入与离开时接收信息；

广播适合以下场景（此小部分摘自[基于 Pusher 驱动的 Laravel 事件广播（上）](https://segmentfault.com/a/1190000004997982)）：

- 通知（`Notification`） 或 信号（`Signal`）
 通知是最简单的示例，也最经常用到。信号也可看作是通知的一种展现形式，只不过信号没有`UI`而已。
- `Activity Streams`
 `Activity Streams(feeds)`是社交网络的核心。如微信朋友圈的点赞和评论，`A`可以实时看到`B`的点赞，`B`可以实时看到`A`的评论。
- 聊天
 聊天信息的实时显示


# 模块组成
![图片描述][1]

# Demo
## 日志驱动
### 配置
`.env`文件修改或添加一行：`BROADCAST_DRIVER=log`；

### 广播
**直接调用**
```php
 $manager = app(Illuminate\Broadcasting\BroadcastManager::class);
 $driver = $manager->connection();
 // 第一个参数是频道名，第二个参数是事件名，第三个参数是广播内容
 $driver->broadcast(['channel_1', 'channel_2'], 'login', ['message' => 'hello world']);
```
因为是日志驱动，所以广播内容会写到框架配置的日志文件中，输出消息如下所示
```
[2017-08-18 20:45:49] local.INFO: Broadcasting [login] on channels [channel_1, channel_2] with payload:
{
    "message": "hello world"
} 
```
**监听事件广播**
这种调用方式，是当实现`ShouldBroadcast`接口的事件被触发时，则会进行广播操作；（同时，还有一个接口叫`ShouldBroadcastNow`，与`ShouldBroadcast`接口的不同在于，将实现`ShouldBroadcastNow`接口的事件放入队列中时，会被放入叫`sync`的队列中）

举个例子，

第一步，`Illuminate\Auth\Events\Login`事件是用户登录成功后会触发的事件，略作改动，让其实现广播功能；
```
class Login implements ShouldBroadcast {
    ......
    
    // 定义事件被触发时，广播频道；此处定义名为 first-channel 的私有频道
    public function broadcastOn() {
        return [
            new PrivateChannel('first-channel'),
        ];
    }
    
    // 自定义广播名称；如果方法未定义，默认以类名为事件名，此处的默认值是 Illuminate\Auth\Events\Login
    public function broadcastAs() {
        return 'login';
    }
}
```

第二步，注册事件监听；在`app/Providers/EventServiceProvider.php`中修改：
```
protected $listen = [
   ......
   'Illuminate\Auth\Events\Login' => [
        'App\Listeners\UserLogin',
   ],
];
```
文件`app/Listeners/UserLogin.php`粗糙地实现了一下：
```
class UserLogin {
    public function __construct() {}
    
    public function handle(Login $event){
        \Log::info('Do UserLogin Listener: I was Login');
    }
}
```

第三步，触发事件，发送广播；有好几种触发广播方式：

1. 直接事件触发

 ```
 event(new Illuminate\Auth\Events\Login($user, true));
 ```
2. 帮助函数`broadcast`，间接触发事件

 ```
 broadcast(new Illuminate\Auth\Events\Login($user, true));
 ```
3. 广播管理类，间接触发事件，直接广播
 
 ```
 $manager = app(Illuminate\Broadcasting\BroadcastManager::class);
 $manager->event(new Illuminate\Auth\Events\Login($user, true));
 ```
4. 广播管理类，间接触发事件，放入队列

 ```
 $manager = app(Illuminate\Broadcasting\BroadcastManager::class);
 $manager->queue(new Illuminate\Auth\Events\Login($user, true));
 ```

## Pusher驱动
`Pusher`是一个第三方服务，服务器发送广播时，会向`Pusher`发送请求，再通过`Pusher`与浏览器或移动端保持的长连接进行数据交互；

### 配置
通过[`Pusher`官网](https://dashboard.pusher.com/)注册用户信息，获取属于自已的一套密钥信息，修改`.env`的配置文件；
```
BROADCAST_DRIVER=pusher
PUSHER_APP_ID=xxxxxxxxxxxxxxxxxxxxxx
PUSHER_APP_KEY=xxxxxxxxxxxxxxxxxxxxxx
PUSHER_APP_SECRET=xxxxxxxxxxxxxxxxxxxxxx
```

### 准备工作

#### 事件监听
后台的事件监听还是采用"日志驱动"部分的登录例子；

#### 前端
前端页面引入以下代码：
```
<script src="https://js.pusher.com/4.1/pusher.min.js"></script>

<script>
// 打开 Pusher 的调试日志
Pusher.logToConsole = true;

// 定义 Pusher 变量
var pusher = new Pusher('PUSHER_APP_KEY的值', {
    cluster: 'ap1',
    encrypted: true
});

// 定义频道，绑定事件
var channel = pusher.subscribe('private-first-channel');
channel.bind('login', function(data) {
    alert(data);
});
</script>
```
如果订阅的是公共频道，则不会向服务器端请求权限检查；如果是私有频道（频道名是以`private-`开头）或存在频道（频道名是以`presence-`开头），则会发出权限检查请求；对应的后端需要定义私有频道和存在频道的权限；

#### 频道权限定义
频道的权限定义是在`routes/channels.php`里；此处笔者为`first-channel`频道定义权限回调函数：
```
Broadcast::channel('first-channel', function ($user) {
    return (int) $user->id === 1;
});
```
有读者会疑问，前端页面订阅的频道不是`private-first-channel`吗？怎么后端只定义`first-channel`频道的权限呢？那是因为，后端定义的频道假设是`A`，那么在`Pusher`及浏览器端或移动端传递的私有频道名为`private-A`，存在频道则会是`presence-A`；

### 广播
**直接广播**
```
$manager = app(Illuminate\Broadcasting\BroadcastManager::class);
$driver = $manager->connection();
// socket 参数是广播私有频道时排除的 socket， 每个浏览器端或者移动端在建立 websocket 时都会被分配一个 socket_id
$driver->broadcast(['private-first-channel'], 'login', ['user' => ['name' => 'hello'], 'socket' => '5395.4377611']);
```
**间接广播**
参考“日志驱动”提及的间接广播方式；

如果要发送排我广播（也就是除了当前请求的这个客户端不收到广播消息），则需要以下条件：

1. 事件使用`Illuminate\Broadcasting\InteractsWithSockets` `trait`；
2. 前端发送过来的请求头部要携带`X-Socket-ID`信息；
3. 事件触发执行`broadcast(new Illuminate\Auth\Events\Login($user, true))->toOthers();`

## Redis驱动
### 配置
`.env`文件修改或添加一行：`BROADCAST_DRIVER=redis`；

### 广播
原理是同样在后端部署一个`Socket.IO`服务器，`Laravel`框架会发布消息到`Socket.IO`服务器上，由`Socket.IO`服务器同浏览器端或者移动端保持长连接；

这部分笔者尚未`demo`，网上入门资料还是挺多的，知道原理，这部分动作上手就容易多了；

# 附录

同类型的文章可参考以下，加深了解：

- [Laravel学院 事件广播基础知识](http://laravelacademy.org/post/6851.html#toc_9)
- [Pusher 的认识](https://segmentfault.com/a/1190000004997982)


  [1]: http://owk2q4gs5.bkt.clouddn.com/bVTjgh.png
