---
title: Laravel 用法之 Queue 模块
date: 2017-12-27 10:48:41
description: 转载 Laravel 学院的队列相关文章并略加精简
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---


本文转载[[ Laravel 5.1 文档 ] 服务 —— 队列](http://laravelacademy.org/post/222.html)

官方 `API` 地址 [https://laravel.com/api/5.1/Illuminate/Queue.html](https://laravel.com/api/5.1/Illuminate/Queue.html)

`Laravel`队列服务为各种不同的后台队列提供了统一的 `API`。队列允许你推迟耗时任务（例如发送邮件）的执行，从而大幅提高`web`请求速度。

队列配置文件存放在 `config/queue.php`。在该文件中你将会找到框架自带的每一个队列驱动的连接配置，包括数据库、`Beanstalkd`、 `IronMQ`、 `Amazon SQS`、 `Redis` 以及同步（本地使用）驱动。其中还包含了一个 `null` 队列驱动以拒绝队列任务。

# 任务类

任务类非常简单，正常情况下只包含一个当队列处理该任务时被执行的 `handle` 方法，让我们看一个任务类的例子：

```
use App\Jobs\Job;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Bus\SelfHandling;
use Illuminate\Contracts\Queue\ShouldQueue;


class SendReminderEmail extends Job implements SelfHandling, ShouldQueue
{
    use InteractsWithQueue, SerializesModels;

    protected $user;

    // 创建一个新的任务实例
    public function __construct(User $user) {
        $this->user = $user;
    }

    // 执行任务
    public function handle() {
    }
}
```

在本例中，注意我们能够直接将 `Eloquent` 模型传递到对列任务的构造函数中。由于该任务使用了`SerializesModels trait`，`Eloquent` 模型将会在任务被执行是优雅地序列化和反序列化。如果你的队列任务在构造函数中接收 `Eloquent` 模型，只有模型的主键会被序列化到队列，当任务真正被执行的时候，队列系统会自动从数据库中获取整个模型实例。这对应用而言是完全透明的，从而避免序列化整个 `Eloquent` 模型实例引起的问题。

`handle` 方法在任务被队列处理的时候被调用，注意我们可以在任务的 `handle` 方法中对依赖进行类型提示。`Laravel` 服务容器会自动注入这些依赖。

**出错**

如果任务被处理的时候抛出异常，则该任务将会被自动释放回队列以便再次尝试执行。任务会持续被释放直到尝试次数达到应用允许的最大次数。最大尝试次数通过 `Artisan` 任务 `queue:listen` 或 `queue:work` 上的 `--tries` 开关来定义。

**手动释放任务**
如果你想要手动释放任务，生成的任务类中自带的 `InteractsWithQueue trait` 提供了释放队列任务的 `release` 方法，该方法接收一个参数——同一个任务两次运行之间的等待时间：

```
public function handle(Mailer $mailer) {
    if (condition) {
        $this->release(10);
    }
}
```

**检查尝试运行次数**
正如上面提到的，如果在任务处理期间发生异常，任务会自动释放回队列中，你可以通过attempts方法来检查该任务已经尝试运行次数：

```
public function handle(Mailer $mailer) {
    if ($this->attempts() > 3) {
    }
}
```

# 推送任务到队列
## 基本使用

默认的 `Laravel` 控制器位于 `app/Http/Controllers/Controller.php` 并使用了 `DispatchesJobs trait`。该 `trait` 提供了一些允许你方便推送任务到队列的方法，例如 `dispatch` 方法：

```
use App\Jobs\SendReminderEmail;
use App\Http\Controllers\Controller;

class UserController extends Controller {

    // 发送提醒邮件到指定用户
    public function sendReminderEmail(Request $request, $id) {
        $user = User::findOrFail($id);
        $this->dispatch(new SendReminderEmail($user));
    }
    
    // 处理失败任务
    public function failed() {
        parent::failed();
        // Called when the job is failing...
    }
}
```

当然，有时候你想要从应用中路由或控制器之外的某些地方分发任务，因为这个原因，你可以在应用的任何类中包含`DispatchesJobs trait`，从而获取对分发方法的访问，举个例子，下面是使用该`trait`的示例类：

```
use Illuminate\Foundation\Bus\DispatchesJobs;

class ExampleClass{
    use DispatchesJobs;
}
```

**为任务指定队列**

```
use App\Jobs\SendReminderEmail;
use App\Http\Controllers\Controller;

class UserController extends Controller {

    // 发送提醒邮件到指定用户
    public function sendReminderEmail(Request $request, $id) {
        $user = User::findOrFail($id);
        $this->dispatch((new SendReminderEmail($user))->onQueue('emails'));
    }
}
```

**延迟任务**
有时候你可能想要延迟队列任务的执行。例如，你可能想要将一个注册15分钟后给消费者发送提醒邮件的任务放到队列中，可以通过使用任务类上的 `delay` 方法来实现，该方法由 `Illuminate\Bus\Queueable trait` 提供：

```
use App\Jobs\SendReminderEmail;
use App\Http\Controllers\Controller;

class UserController extends Controller {

    // 发送提醒邮件到指定用户
    public function sendReminderEmail(Request $request, $id) {
        $user = User::findOrFail($id);
        $this->dispatch((new SendReminderEmail($user))->delay(15 * 60));
    }
}
```

**从请求中分发任务**
映射`HTTP`请求变量到任务中很常见，`Laravel`提供了一些帮助函数让这种实现变得简单，而不用每次请求时手动执行映射。让我们看一下 `DispatchesJobs trait`上的`dispatchFrom`方法。默认情况下，该`trait`包含在`Laravel`控制器基类中：

```
public function processOrder(Request $request, $id) {
    // 处理请求...
    $this->dispatchFrom('App\Jobs\ProcessOrder', $request);
}
```

该方法检查给定任务类的构造函数并从`HTTP`请求（或者其它`ArrayAccess`对象）中解析变量来填充任务需要的构造函数参数。所以，如果我们的任务类在构造函数中接收一个`productId`变量，该任务将会尝试从`HTTP`请求中获取`productId`参数。

你还可以传递一个数组作为 `dispatchFrom` 方法的第三个参数。该数组用于填充所有请求中不存在的构造函数参数：

```
$this->dispatchFrom('App\Jobs\ProcessOrder', $request, [
    'taxPercentage' => 20,
]);
```



## 任务完成事件
`Queue::after` 方法允许你在队列任务执行成功后注册一个要执行的回调函数。在该回调中我们可以添加日志、统计数据。例如，我们可以在`Laravel`内置的`AppServiceProvider`中添加事件回调:

```
use Queue;

class AppServiceProvider extends ServiceProvider {

    public function boot() {
        Queue::after(function ($connection, $job, $data) {
        });
    }

}
```

# 运行队列监听器

**开启任务监听器**
`Laravel` 包含了一个 `Artisan` 命令用来运行被推送到队列的新任务。你可以使用 `queue:listen` 命令运行监听器， 还可以指定监听器使用哪个队列连接：

```
php artisan queue:listen

// php artisan queue:listen connection
```

注意一旦任务开始后，将会持续运行直到手动停止。你可以使用一个过程监视器如 `Supervisor` 来确保队列监听器没有停止运行。

**队列优先级**
你可以传递逗号分隔的队列连接列表到 `listen` 任务来设置队列优先级；在本例中，`high` 队列上的任务总是在从 `low` 队列移动任务之前被处理。

```
php artisan queue:listen --queue=high,low
```

**指定任务超时参数**
设置每个任务允许运行的最大时间（以秒为单位）：

```
php artisan queue:listen --timeout=60
```

**指定队列睡眠时间**
需要注意的是队列只会在队列上没有任务时“睡眠”，如果存在多个有效任务，该队列会持续运行，从不睡眠。可以指定轮询新任务之前的等待时间（以秒为单位）：

```
php artisan queue:listen --sleep=5
```

## `Supervisor` 配置
`Supervisor`为`Linux`操作系统提供的进程监视器，将会在失败时自动重启`queue:listen`或`queue:work`命令，要在`Ubuntu`上安装`Supervisor`，使用如下命令：

```
sudo apt-get install supervisor
```

`Supervisor`配置文件通常存放在`/etc/supervisor/conf.d`目录，在该目录中，可以创建多个配置文件指示`Supervisor`如何监视进程，例如，让我们创建一个开启并监视`queue:work`进程的`laravel-worker.conf`文件：

```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```

在本例中，`numprocs`指令让`Supervisor`运行8个`queue:work`进程并监视它们，如果失败的话自动重启。配置文件创建好了之后，可以使用如下命令更新`Supervisor`配置并开启进程：

```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

## 后台队列监听器
`Artisan` 命令 `queue:work` 包含一个 `--daemon` 选项来强制队列 `worker` 持续处理任务而不必重新启动框架。相较于 `queue:listen` 命令该命令对 `CPU` 的使用有明显降低：

```
php artisan queue:work connection --daemon
php artisan queue:work connection --daemon --sleep=3
php artisan queue:work connection --daemon --sleep=3 --tries=3
```

由于后台队列 `worker` 是常驻进程，不重启的话不会应用代码中的更改，所以，最简单的部署后台队列 `worker` 的方式是使用部署脚本重启所有 `worker`，你可以通过在部署脚本中包含如下命令重启所有 `worker`：

```
php artisan queue:restart
```
该命令会告诉所有队列 `worker` 在完成当前任务处理后重启以便没有任务被遗漏。



# 处理失败任务
由于事情并不总是按照计划发展，有时候你的队列任务会失败。别担心，它发生在我们大多数人身上！`Laravel` 包含了一个方便的方式来指定任务最大尝试执行次数，任务执行次数达到最大限制后，会被插入到 `failed_jobs` 表，失败任务的名字可以通过配置文件 `config/queue.php` 来配置。

要创建一个 `failed_jobs` 表的迁移，可以使用 `queue:failed-table` 命令：

```
php artisan queue:failed-table
```

运行队列监听器的时候，可以在 `queue:listen` 命令上使用 `--tries` 开关来指定任务最大可尝试执行次数：

```
php artisan queue:listen connection-name --tries=3
```


## 失败任务事件
如果你想要注册一个队列任务失败时被调用的事件，可以使用 `Queue::failing` 方法，该事件通过邮件或 `HipChat` 通知团队。举个例子，我么可以在 `Laravel` 自带的 `AppServiceProvider` 中附件一个回调到该事件：

```
class AppServiceProvider extends ServiceProvider {

    public function boot() {
        Queue::failing(function ($connection, $job, $data) {
            // Notify team of failing job...
        });
    }
}
```


## 重试失败任务
要查看已插入到 `failed_jobs` 数据表中的所有失败任务，可以使用 `Artisan` 命令 `queue:failed`：

```
php artisan queue:failed
```

该命令将会列出任务`ID`，连接，对列和失败时间，任务`ID`可用于重试失败任务，例如，要重试一个`ID`为5的失败任务，要用到下面的命令：

```
php artisan queue:retry 5
```

如果你要删除一个失败任务，可以使用 `queue:forget` 命令：

```
php artisan queue:forget 5
```

要删除所有失败任务，可以使用 `queue:flush` 命令：

```
php artisan queue:flush
```











