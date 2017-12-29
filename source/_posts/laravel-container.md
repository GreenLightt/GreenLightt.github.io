---
title: Laravel 大将之 服务容器 模块
date: 2017-08-31 17:57:41
description: 介绍如何使用服务容器模块
tags:
- Laravel-5.4
categories:
- Laravel
copyright: false
---

# 简介
官方 `API` 地址 [https://laravel.com/api/5.4/Illuminate/Container/Container.html](https://laravel.com/api/5.4/Illuminate/Container/Container.html)

服务容器是一个用于管理类依赖和执行依赖注入的强大工具。是整个框架的核心；

几乎所有的服务容器绑定都是在服务提供者中完成。

# 框架调用分析
在框架直接生成服务容器的只有一处，在`bootstrap/app.php`，通过`require`引用会返回服务容器实例。通过`require`引用有两处，一处是`public/index.php`，服务器访问的入口；另一处是`tests/CreatesApplication.php`，是单元测试的入口；

如果想在项目各处中调用，可以调用`$app = Illuminate\Container\Container::getInstance()`或者全局帮助函数`app()`获取服务容器实例（也就是`Illuminate\Foundation/Application`实例）；

`Illuminate\Foundation/Application`是对`Illuminate\Container\Container`的又一层封装；

## `Application`初始化
那么实例化`Illuminate\Foundation/Application`时，做了什么呢？

**第一步**，设置应用的根目录，并同时注册核心目录到服务容器中；核心的目录有以下

- `path`：目录`app`的位置
- `path.base`：项目根目录的位置
- `path.lang`：目录`resources/lang`的位置
- `path.config`：目录`config`的位置
- `path.public`：目录`public`的位置
- `path.storage`：目录`storage`的位置
- `path.database`：目录`database`的位置
- `path.resources`：目录`resources`的位置
- `path.bootstrap`：目录`bootstrap`的位置

**第二步**，将当前`Illuminate\Foundation/Application`实例保存到`$instance`类变量，并同时绑定到服务容器作单例绑定，绑定名为`app`或`Container::class`；

**第三步**，顺序分别执行注册`Illuminate\Events\EventServiceProvider`、`Illuminate\Log\LogServiceProvider`和`Illuminate\Routing\RoutingServiceProvider`三个服务提供者；

注册服务提供者的顺序如下：

- 如果类变量`$serviceProviders`已经存在该服务提供者并且不需要强制重新注册，则返回服务提供者实例`$provider`；
- 未注册过当前服务提供者，则继续执行以下；
- 如果存在`register`方法，执行服务提供者的`register`方法；
- 将当前服务提供者`$provider`实例保存到类变量`$serviceProviders`数组中，同时标记类变量`$loadedProviders[get_class($provider)]`的值为`true`；
- 判断类变量`$booted`是否为`true`，如果是`true`，则执行服务提供者的`boot`方法；（类变量`$booted`应该是标志是否所有服务提供者均注册，框架是否启动）

**第四步**，注册核心类别名；
比如`\Illuminate\Foundation\Application::class`、`\Illuminate\Contracts\Container\Container::class`起别名为`app`；

## 单元测试`Application`的`bootstrap`启动分析
启动代码很简洁，
```
public function createApplication() {
    // require 初始化分析上面已经介绍了
    $app = require __DIR__.'/../bootstrap/app.php';
    // 生成`Illuminate\Foundation\Console\Kernel`实例，执行bootstrap方法，完成启动
    $app->make(Kernel::class)->bootstrap();
    return $app;
}
```

构造函数主要干了一件事，注册一个`booted`完成后的回调函数，函数执行的内容为“注册 `Schedule`实例到服务提供者，同时加载用户定义的`Schedule`任务清单”；

`bootstrap`方法的执行内容如下：

1. 加载`Illuminate/Foundation/Console/Kernel`中`$bootstrappers`变量数组中的类，执行它们的`bootstrap`方法；

 ```
 protected $bootstrappers = [
     // 加载 .env 文件
     \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
     // 加载 config 目录下的配置文件
     \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
     // 自定义错误报告，错误处理方法及呈现
     \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
     // 为 config/app.php 中的 aliases 数组注册类别名
     \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
     // 在服务容器中单例绑定一个 request 对象，控制台命令会用到
     \Illuminate\Foundation\Bootstrap\SetRequestForConsole::class,
     // 注册 config\app.php 中的 providers 服务提供者
     \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
     // 项目启动，执行每个 ServiceProvider 的 boot 方法，
     \Illuminate\Foundation\Bootstrap\BootProviders::class,
 ];
 ```
2. 加载延迟的服务提供者；

## Http访问`Application`的`bootstrap`启动分析
启动入口文件在`public\index.php`
```
$app = require_once __DIR__.'/../bootstrap/app.php';

// 实例化 Illuminate/Foundation/Http/Kernel 对象
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

// 中间件处理、业务逻辑处理
$response = $kernel->handle(
    // 根据 Symfony 的 request 对象封装出 Illuminate\Http\Request
    $request = Illuminate\Http\Request::capture() 
);

$response->send();

// 执行所有中间件的 terminate 方法，执行 Application 中的 terminatingCallbacks 回调函数
$kernel->terminate($request, $response);
```

# 重要的类变量数组

## `aliases`数组
维护 类与别名 的数组；键名为 类的全限定类名，键值为 数组，每一个元素都是该类的别名；

> 判断指定类是否有别名：`app()->isAlias($name)`；
  获取指定类的别名：`app()->getAlias($abstract)`；
  
## `abstractAliases`数组

维护 类与别名 的数组；键名为 别名，键值为 类的全限定类名；

## `instances`数组
维护 类与实例的数组；键名为 类的全限定类名，键值为该类的实例；

> 移除绑定类：`app()->forgetInstance($abstract);`
移除所有绑定类：`app()->forgetInstances();`

## `bindings`数组
通过 `bind` 方法实现 接口类与实现的绑定；

> 获取`bindings`数组中的内容：`app()->getBindings()`
## `resolved`数组
键名为 类的全限定类名，键值为布尔值类型（`true`表示已解析过，`false`表示未解析过）；

## `with`数组
在`resolved`过程中，会有一些参数；`with`数组就是参数栈，开始解析时将参数入栈，结束解析时参数出栈；

## `contextual`数组
上下文绑定数组；第一维数组键名为 场合类（比如某个`Controller`类的类名），第二维数组键名为 抽象类（需要实现的接口类），键值为 `Closure` 或 某个具体类的类名；

## `tags`数组
维护 标签与类 的数组；键名是 标签名，键值是 对应要绑定的类的名称；

如果调用`tagged`方法，会将键值数组中的类都`make`出来，并以数组形式返回；

## `extenders`数组
在`make`或`resolve`出对象的时候，会执行
```
foreach ($this->getExtenders($abstract) as $extender) {
    $object = $extender($object, $this);
}
```
能对解析出来的对象进行修饰；

## `methodBindings`数组
可以绑定类的一个方法到自定义的函数；用法可以参考[Laravel核心——Ioc服务容器 ](http://www.leoyang90.cn/2017/05/06/Laravel-container/)中的相关章节；


> 向容器绑定方法与及实现：`app()->bindMethod($method, $callback)`
  判断容器内是否有指定方法的实现：`app()->hasMethodBinding($method)`
  执行方法的实现：`app()->callMethodBinding($method, $instance)`或者`app()->call($method)`

## `buildStack`数组
调用`build`方法时维护的栈，栈中存放的是当前要`new`的类名；

## `reboundCallbacks`数组
当调用`rebound`函数时，会触发`rebound`中为此`$abstract`设置的回调函数；

> 注册入口：`app()->rebinding($abstract, Closure $callback);`

## `serviceProviders`数组
已在系统注册的服务提供者`ServiceProvider`；

数组内存放的是`loadedProviders`键名对应类的实例；

## `loadedProviders`数组
系统已加载的`ServiceProvider`的集合；键名为`ServiceProvider`的全限定类名，键值为布尔值（`true`表示已加载，`false`表示未加载）；

> 获取延迟加载对象：`app()->getLoadedProviders()`;

## `deferredServices`数组
有些服务提供者是会延迟加载的；这时候会将这些服务提供者声明的服务登录在`deferredServices`数组，键名为延迟加载对象名 ，键值为该延迟加载对象所在的`ServiceProvider`；

> 获取延迟加载对象：`app()->getDeferredServices()`;

## `bootingCallbacks`数组
项目启动前执行的回调函数；（项目启动是在执行`\Illuminate\Foundation\Bootstrap\BootProviders::class`的时候）

> 注册入口：`app()->booting($callback);`

## `bootedCallbacks`数组
项目启动后执行的回调函数；（项目启动是在执行`\Illuminate\Foundation\Bootstrap\BootProviders::class`的时候）

> 注册入口：`app()->booted($callback);`

## `resolvingCallbacks`数组
解析时回调函数集合；键名为 类名， 键值为 回调函数数组，每一个元素都是回调函数；

> 注册入口：`app()->resolving($abstract, $callback);`

## `afterResolvingCallbacks`数组
解析后回调函数集合；键名为 类名， 键值为 回调函数数组，每一个元素都是回调函数；

> 注册入口：`app()->afterResolving($abstract, $callback);`

## `globalResolvingCallbacks`数组
全局解析时回调函数集合；每一次`resolve`方法调用时都会执行的回调函数集合；

> 注册入口：`app()->resolving($callback);`

## `globalAfterResolvingCallbacks`数组
全局解析后回调函数集合；每一次`resolve`方法调用后都会执行的回调函数集合；

> 注册入口：`app()->afterResolving($callback);`

## `terminatingCallbacks`数组

系统在返回`response`之后，会执行`terminate`方法，来做应用结束前的扫尾处理；

这个数组就是执行`terminate`方法时会执行的回调函数集合；

> 注册入口：`app()->terminating(Closure $callback)`; 

# 常用方法的解析

## `bind`方法
```
public function bind($abstract, $concrete = null, $shared = false)
```
第一个参数是要注册的类名或接口名，第二个参数是返回类的实例的闭包（或类的实例类名），第三个参数是否是单例；

方法内部流程：

1. `unset` 掉 **`instances`** 和 **`aliases`** 数组中键值为 `$abstract` 的元素；
2. 如果 `$concrete` 值为 `null` ，将 `$abstract` 赋值给 `$concrete`；
3. 如果 `$concrete` 不是 `Closure` 对象，则封装成闭包；
4. 将 `$concrete` 和 `$shared` 通过 `compact`，添加进 **`bindings`** 数组，键名为 `$abstract`；
5. 判断 `$abstract` 在 **`resolved`** 和 **`instances`** 数组中是否存在，如果存在，则执行第 6 步；
6. 触发 `rebound`回调函数；如果 `reboundCallbacks` 数组中注册以 `$abstract` 为键名的回调函数，则执行这些回调函数；

> 涉及数组：`instances`和`aliases`（unset 操作）、`bindings`（add 操作）

## `singleton`方法
单例绑定；
```
public function singleton($abstract, $concrete = null)
    $this->bind($abstract, $concrete, true);
}
```

> 涉及数组：`instances`和`aliases`（unset 操作）、`bindings`（add 操作）

## `bindIf`方法
单例绑定；
```
public function bindIf($abstract, $concrete = null, $shared = false) {
    if (! $this->bound($abstract)) {
        $this->bind($abstract, $concrete, $shared);
    }
}
```

> 涉及数组：`instances`和`aliases`（unset 操作）、`bindings`（add 操作）

## `instance`方法
绑定实例；
```
public function instance($abstract, $instance)
```
方法内部流程：

1. 如果`$abstract`在`aliases`数组中存在，则从`abstractAliases`中所有的值数组中移除该类；
2. `unset` 掉 `aliases` 数组中键名为 `$abstract`的元素；
3. 赋值操作：`$this->instances[$abstract] = $instance;`
4. 判断 `$abstract` 在 **`resolved`** 和 **`instances`** 数组中是否存在，如果存在，则执行第 5 步；
5. 触发 `rebound`回调函数；如果 `reboundCallbacks` 数组中注册以 `$abstract` 为键名的回调函数，则执行这些回调函数；

> 涉及数组：`instances`（add 操作）、`aliases` 和 `abstractAliases`（unset 操作）

## `make`方法
```
public function make($abstract) {
    return $this->resolve($abstract);
}
```

## `alias`方法
给类起别名；
```
public function alias($abstract, $alias) {
    $this->aliases[$alias] = $abstract;
    
    $this->abstractAliases[$abstract][] = $alias;
}
```
> 涉及数组：`aliases`和`abstractAliases`（add 操作）

# 扩展阅读

- [leoyang90博客 - Laravel核心——Ioc服务容器](http://www.leoyang90.cn/2017/05/06/Laravel-container/)
