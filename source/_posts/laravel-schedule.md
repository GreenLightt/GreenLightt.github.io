---
title: Laravel 用法之 任务调度 模块
date: 2017-12-27 18:51:41
description: 转载 Laravel 学院的任务调度相关文章并略加精简
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

本文转载[[ Laravel 5.1 文档 ] 服务 —— 任务调度](http://laravelacademy.org/post/235.html)

在以前，开发者需要为每一个需要调度的任务编写一个 `Cron` 条目，这是很让人头疼的事。你的任务调度不在源码控制中，你必须使用 `SSH` 登录到服务器然后添加这些 `Cron` 条目。`Laravel` 命令调度器允许你平滑而又富有表现力地在 `Laravel` 中定义命令调度，并且服务器上只需要一个 `Cron` 条目即可。

# 基本使用

任务调度定义在 `app/Console/Kernel.php` 文件的 `schedule` 方法中，该方法中已经包含了一个示例。你可以自由地添加你需要的调度任务到 `Schedule` 对象。

```
use DB;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;


class Kernel extends ConsoleKernel{

    // 定义应用的命令调度
    protected function schedule(Schedule $schedule) {
        $schedule->call(function () {
            DB::table('recent_users')->delete();
        })->daily();
    }
}
```

除了调度闭包调用外，还可以调度 `Artisan` 命令和操作系统命令。例如，可以使用 `command` 方法来调度一个 `Artisan` 命令：

```
$schedule->command('emails:send --force')->daily();
```

`exec` 命令可用于发送命令到操作系统：

```
$schedule->exec('node /home/forge/script.js')->daily();
```

## 调度常用方法

| 方法 | 描述 |
|: --- :|: --- :|
| `->cron('* * * * *');` | 在自定义Cron调度上运行任务 |
| `->everyMinute();` | 每分钟运行一次任务 |
| `->everyFiveMinutes();` | 每五分钟运行一次任务 |
| `->everyTenMinutes();` | 每十分钟运行一次任务 |
| `->everyThirtyMinutes();` | 每三十分钟运行一次任务 |
| `->hourly();` | 每小时运行一次任务 |
| `->daily();` | 每天凌晨零点运行任务 |
| `->dailyAt('13:00');` | 每天13:00运行任务 |
| `->twiceDaily(1, 13);` | 每天1:00 & 13:00运行任务 |
| `->weekly();` | 每周运行一次任务 |
| `->monthly();` | 每月运行一次任务 |

这些方法可以和额外的约束一起联合起来创建**一周**特定时间运行的更加细粒度的调度，例如，要每**周一**调度一个命令：

```
$schedule->call(function () {
    // 每周星期一13:00运行一次...
})->weekly()->mondays()->at('13:00');
```


| 方法 | 描述 |
|: --- :|: --- :|
| `->weekdays();` | 只在工作日运行任务 |
| `->sundays();` | 每个星期天运行任务 |
| `->mondays();` | 每个星期一运行任务 |
| `->tuesdays();` | 每个星期二运行任务 |
| `->wednesdays();` | 每个星期三运行任务 |
| `->thursdays();` | 每个星期四运行任务 |
| `->fridays();` | 每个星期五运行任务 |
| `->saturdays();` | 每个星期六运行任务 |
| `->when(Closure);` | 基于特定测试运行任务 |

**基于测试的约束条件**
`when` 方法用于限制任务在通过给定测试之后运行。换句话说，如果给定闭包返回 `true`，只要没有其它约束条件阻止任务运行，该任务就会执行：

```
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});
```

## 避免任务重叠
默认情况下，即使前一个任务仍然在运行调度任务也会运行，要避免这样的情况，可使用 `withoutOverlapping` 方法：

```
$schedule->command('emails:send')->withoutOverlapping();
```

在本例中，`Artisan` 命令 `emails:send` 每分钟都会运行，如果该命令没有在运行的话。如果你的任务在执行时经常大幅度的变化，那么 `withoutOverlapping` 方法就非常有用，你不必再去预测给定任务到底要消耗多长时间。

## 任务输出
`Laravel` 调度器为处理调度任务输出提供了多个方便的方法。首先，使用 `sendOutputTo` 方法，你可以发送输出到文件以便稍后检查：

```
$schedule->command('emails:send')->daily()->sendOutputTo($filePath);
```

使用 `emailOutputTo` 方法，你可以将输出发送到电子邮件，注意输出必须首先通过 `sendOutputTo` 方法发送到文件。还有，使用电子邮件发送任务输出之前，应该配置 `Laravel` 的电子邮件服务：

```
$schedule->command('foo')->daily()->sendOutputTo($filePath)->emailOutputTo('foo@example.com');
```

注意：`emailOutputTo` 和 `sendOutputTo` 方法只对 `command` 方法有效，不支持 `call` 方法。

## 任务钩子
使用 `before` 和 `after` 方法，你可以指定在调度任务完成之前和之后要执行的代码：

```
$schedule->command('emails:send')->daily()
         ->before(function () {
             // Task is about to start...
         })
         ->after(function () {
             // Task is complete...
         });
```

**ping Url**

使用`pingBefore` 和 `thenPing` 方法，调度器可以在任务完成之前和之后自动 `ping` 给定的 `URL`。该方法在通知外部服务时很有用，例如`Laravel Envoyer`，在调度任务开始或完成的时候：

```
$schedule->command('emails:send')->daily()->pingBefore($url)->thenPing($url);
```

使用 `pingBefore($url)` 或 `thenPing($url)` 特性需要安装 `HTTP` 库 `Guzzle`，可以在 `composer.json` 文件中添加如下行来安装 `Guzzle` 到项目：

```
"guzzlehttp/guzzle": "~5.3|~6.0"
```

## ServiceProvider 中注册
如果想在 `ServiceProvider` 中注册任务调度命令，需要利用 `ServiceProvider` 的 `boot` 回调函数执行注册；

```
use Illuminate\Foundation\Support\Providers\RouteServiceProvider

class ShelfServiceProvider extends RouteServiceProvider {

    public function boot() {
        $this->app->booted(function () {
        
            $schedule = $this->app->make(Schedule::class);
            
            $schedule->command('schedule:check_advertising_position_status')->dailyAt('04:00');
        });
    }
}
```












