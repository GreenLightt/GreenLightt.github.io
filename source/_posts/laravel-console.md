---
title: Laravel 用法之 Artisan 控制台 模块
date: 2017-12-29 18:11:41
description: 转载 Laravel 学院的 Artisan 控制台相关文章并略加精简
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

本文转载[[ Laravel 5.1 文档 ] 服务 —— Artisan 控制台
](http://laravelacademy.org/post/170.html)

官方 `API` 地址 [https://laravel.com/api/5.1/Illuminate/Console/Command.html](https://laravel.com/api/5.1/Illuminate/Console/Command.html);

`Artisan` 是 `Laravel` 自带的命令行接口名称，它为你在开发过程中提供了很多有用的命令。通过强大的 `Symfony Console` 组件驱动。想要查看所有可用的 `Artisan` 命令，可使用 `list` 命令：

```
php artisan list
```

# 编写命令
## 示例
```
namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;
use Illuminate\Foundation\Inspiring;

class Inspire extends Command {
    // 控制台命令名称
    protected $signature = 'email:send {user}';

    // 控制台命令描述
    protected $description = 'Send drip e-mails to a user';

    // 创建新的命令实例， 可以依赖注入实例
    public function __construct() {
        parent::__construct();
    }

    // 执行控制台命令
    public function handle() {
    }
}
```

## 命令IO
### 定义期望输入
编写控制台命令的时候，通常通过参数和选项收集用户输入，`Laravel` 使这项操作变得很方便：在命令中使用 `signature` 属性来定义我们期望的用户输入。`signature` 属性通过一个优雅的、路由风格的语法允许你定义命令的名称、参数以及选项。所有用户提供的参数和选项都包含在大括号里：

```
// 控制台命令名称
protected $signature = 'email:send {user}';

/**
 *  // 选项参数...
 *  email:send {user?}
 *  // 带默认值的选项参数...
 *  email:send {user=foo}
 *  // 选项， 如果 `--queue` 开关被传递，其值是 `true`，否则其值是 `false`：
 *  email:send {user} {--queue}
 *  // 选项值由用户来分配
 *  email:send {user} {--queue=}
 *  // 选项值有默认值
 *  email:send {user} {--queue=default}
 */
```


你可以通过`：`分隔参数和描述来分配描述给输入参数和选项

```
//  控制台命令名称
protected $signature = 'email:send
    {user : The ID of the user}
    {--queue= : Whether the job should be queued}';
```

### 获取输入
在命令被执行的时候，很明显，你需要访问命令获取的参数和选项的值。使用 `argument` 和 `option` 方法即可实现：

要获取参数的值，通过 `argument` 方法：

```
$userId = $this->argument('user');

// 获取所有参数值
$arguments = $this->argument();
```

选项值和参数值的获取一样简单，使用 `option` 方法，同 `argument` 一样如果要获取所有选项值，可以调用不带参数的 `option` 方法：

```
// 获取指定选项...
$queueName = $this->option('queue');
// 获取所有选项...
$options = $this->option();
```
如果参数或选项不存在，返回 `null`。

### 输入提示
**`secret` 和 `ask`命令**
除了显示输出之外，你可能还要在命令执行期间要用户提供输入。 `ask` 方法将会使用给定问题提示用户，接收输入，然后返回用户输入到命令：

```
$name = $this->ask('What is your name?');
```

`secret` 方法和 `ask` 方法类似，但用户输入在终端对他们而言是不可见的，这个方法在问用户一些敏感信息如密码时很有用：

```
$password = $this->secret('What is the password?');
```

**`confirm`命令**
如果你需要让用户确认信息，可以使用 `confirm` 方法，默认情况下，该方法返回 `false`，如果用户输入 `y`，则该方法返回`true`：

```
if ($this->confirm('Do you wish to continue? [y|N]')) {
}
```

**`anticipate` 和 `choice`命令**

`anticipate` 方法可用于为可能的选项提供自动完成功能，用户仍然可以选择答案，而不管这些选择：

```
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

如果你需要给用户预定义的选择，可以使用 `choice` 方法。用户选择答案的索引，但是返回给你的是答案的值。如果用户什么都没选的话你可以设置默认返回的值：

```
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], false);
```

### 编写输出
#### 信息
要将输出发送到控制台，使用 `info`, `comment`, `question` 和  `error` 方法，每个方法都会使用相应的 `ANSI` 颜色以作标识。

要显示一条信息消息给用户，使用 `info` 方法。通常，在终端显示为绿色：

```
$this->info('Display this on the screen');
```

要显示一条错误消息，使用 `error` 方法。错误消息文本通常是红色：

```
$this->error('Something went wrong!');
```

#### 表格布局
`table` 方法使输出多行/列格式的数据变得简单，只需要将头和行传递给该方法，宽度和高度将基于给定数据自动计算：

```
$headers = ['Name', 'Email'];
$users = App\User::all(['name', 'email'])->toArray();
$this->table($headers, $users);
```


#### 进度条
对需要较长时间运行的任务，显示进度指示器很有用，使用该输出对象，我们可以开始、前进以及停止该进度条。在开始进度时你必须定义步数，然后每走一步进度条前进一格：

```
$users = App\User::all();

$this->output->progressStart(count($users));

foreach ($users as $user) {
    $this->performTask($user);
    $this->output->progressAdvance();
}

$this->output->progressFinish();
```



# 注册命令

## 在 `Kernel` 中注册
命令编写完成后，需要注册到 `Artisan` 才可以使用，这可以在 `app/Console/Kernel.php` 文件中完成。

在该文件中，你会在 `commands` 属性中看到一个命令列表，要注册你的命令，只需将其加到该列表中即可。当 `Artisan` 启动的时候，该属性中列出的命令将会被服务容器解析被注册到 `Artisan`：

```
protected $commands = [
    'App\Console\Commands\SendEmails'
];
```

## 在 `ServiceProvider` 中注册

```
protected $commands = [
    ClearInvalidProcurementOrder::class,
]

public function register() {
    $this->commands($this->commands);
}
```


# 通过代码调用命令
有时候你可能希望在 `CLI` 之外执行 `Artisan` 命令，比如，你可能希望在路由或控制器中触发 `Artisan` 命令，你可以使用 `Artisan` 门面上的 `call` 方法来完成这个。`call` 方法接收被执行的命令名称作为第一个参数，命令参数数组作为第二个参数，退出代码被返回：

```
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
});
```

使用 `Artisan` 上的 `queue` 方法，你甚至可以将 `Artisan` 命令放到队列中，这样它们就可以通过后台的队列工作者来处理：

```
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
});
```

如果你需要指定不接收字符串的选项值，例如 `migrate:refresh` 命令上的 `--force` 标识，可以传递布尔值`true` 或 `false`：

```
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

有时候你希望从一个已存在的 `Artisan` 命令中调用其它命令。你可以通过使用 `call` 方法开实现这一目的。`call` 方法接收命令名称和数组形式的命令参数：

```
// 执行控制台命令
public function handle(){
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
}
```

如果你想要调用其它控制台命令并阻止其所有输出，可以使用 `callSilent` 方法。`callSilent` 方法和 `call` 方法用法一致：

```
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```








