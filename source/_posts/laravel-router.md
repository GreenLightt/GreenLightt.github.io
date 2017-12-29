---
title: Laravel 大将之 路由 模块
date: 2017-08-01 17:57:41
description: 介绍如何使用路由模块
tags:
- Laravel-5.4
categories:
- Laravel
---

本文是基于`Laravel 5.4`版本的路由模块代码进行分析书写；

官方 `API` 地址 [https://laravel.com/api/5.4/Illuminate/Routing.html](https://laravel.com/api/5.4/Illuminate/Routing.html)

# 模块组成
下图展示了路由模块中各个文件的关系，并进行简要说明；
![图片描述][1]

# 剖析
## 服务提供者
看`Laravel`模块，首先找`ServiceProvider`文件，这是模块与`IOC`容器交互的入口，从这个文件，可以看出该模块提供向系统提供了哪些服务；

```
public function register() {
    // 注册路由管理，提供路由注册，路由匹配的功能
    $this->registerRouter();
    // 注册 Url 生成器实例
    $this->registerUrlGenerator();
    // 注册跳转器
    $this->registerRedirector();
    // 绑定 PSR-7 请求实现到 ServerRequestInterface 接口
    $this->registerPsrRequest();
    // 绑定 PSR-7 Response 实现到 ResponseInterface 接口
    $this->registerPsrResponse();
    // 注册 ReponseFactory，提供各式各样的 Response，比如视图响应、Json响应、Jsonp响应、文件下载等
    $this->registerResponseFactory();
}
```

## 路由管理
“路由管理”服务有以下元素需要了解：

- `Route`：路由；会记录 `Url`、`Http` 动作、`Action` (路由要执行的具体对象，可能是 `Closure`，也可以是某个 `Controller` 中的方法)，路由参数，路由参数的约束；
- `RouteCollection`：路由集，用来存储所有`Route`对象的“盒子”；
- `RouteGroup`：路由组；只有路由注册过程中会临时用到；存储一批路由公共的一些属性，属性包括`domain`、`prefix`、`as`、`middleware`、`namespace`、`where`；
- `Resource`：资源路由；资源路由是一套路由的统称，包含列表（`index`）、显示增加（`create`）、保存增加（`store`）、显示详情（`show`）、显示编辑详情（`edit`）、更新编辑（`update`）、删除详情（`destory`）；同时可以通过调用`only`或`except`方法或参数的形式只生成部分路由；
- `Action`：路由要执行的对象；有两种表现形式，一是`Closure`函数，二是类似`['uses' => 'FooController@method', 'as' => 'name']`这样的字符串；对于不同的表现形式，路由在执行时会调用不同的处理；

### 注册流程
在项目启动后，会执行所有`ServiceProvider`的`loadRoutes`方法，也就是调用`map`方法，一般情况下`map`方法如下
```
public function map(Router $router){
    require __DIR__.'/routes.php';
}
```
这时候，项目就会执行很多`Route::get`、`Route::post`、`Route::group`方法；

当遇到`Route::group`方法时，会实例化一个`RouteGroup`对象，`put`进`Router`管理类的路由组栈头部；而后当执行`get`、`post`这类具体的注册路由方法时，会把当前路由组栈中所有组的属性合并进新路由中，将新路由存储在`RouteCollection`这个大盒子里；当`Route::group`的`Closure`执行完毕时，会把头部的`RouteGroup`实例`pull`出去；

当执行`Route::resource`时，`Router`管理类会调用`ResourceRegister`类来完成批量注册路由；

> 对于 `Router::get`这类注册方法，`Illuminate\Foudation\helpers`提供了简写；
`Router::get`     简化成 `get`，
`Router::post`    简化成 `post`，
`Router::put`     简化成 `put`，
`Router::patch`   简化成 `patch`，
`Router::delete`  简化成 `delete`，
`Router::resource`简化成 `resource`，

至此，`RouteCollection`大盒子就存放了所有要注册的路由；

### request 请求匹配流程
首先，`request`请求会经过`Foundation/Http/Kernel`的`handle`方法，在这个方法中，请求会执行以下语句
```
$this->router->dispatch($request)
```
这里的`$this->router`，就是`Router`管理类；`dispatch`方法如下
```
public function dispatch(Request $request) {
    $this->currentRequest = $request;
    return $this->dispatchToRoute($request);
}

public function dispatchToRoute(Request $request) {
    // 根据请求的 url 找到匹配的路由
    $route = $this->findRoute($request);
    // 将路由绑定到请求上
    $request->setRouteResolver(function () use ($route) {
        return $route;
    }
    // 触发 RouteMatched 事件
    $this->events->dispatch(new Events\RouteMatched($route, $request));
    // 通过 Pipeline 流水线执行路由上绑定的中间件及对应的方法
    $response = $this->runRouteWithinStack($route, $request);
    // 根据 request 请求设置 response 的响应头
    return $this->prepareResponse($request, $response);
}
```

1. *根据请求找匹配的路由*
    `RouteCollection`根据请求的`http`动作缩小要匹配的路由范围；在筛选出来的这些路由中依次遍历，找出第一个符合验证的路由（需要进行较验的验证在`Route`中的`getValidators`方法中声明）；
2. *将路由绑定到请求上*
3. *触发`RouteMatched`事件*
    初始化的`Laravel`项目没有对`RouteMatched`路由匹配事件进行任何的监听器绑定，如有需要，可以自定义监听器，在模块的`EventServiceProvider`中注册该事件监听；这样一旦请求匹配上某个路由，就可以执行自定义方法了；
4. *通过 `Pipeline` 流水线执行路由上绑定的中间件及对应的方法*
    在`runRouteWithinStack`方法中，系统会判断是否需要执行中间件，如果`IOC`容器中设置了`middleware.disable`的值为`true`，则需要执行的中间件数组为空；否则会找到所有的中间件，并按照`middlewarePriority`对必要的一些中间件进行排序调整；然后执行`$route->run()`方法；
5. *根据 request 请求设置 response 的响应头*

> 项目中会用到的一些方法
1. 获取路由集合 `app('router')->getRoutes()`
2. 获取当前的请求 `$request = app('router')->getCurrentRequest()`
3. 获取当前请求所对应的路由 `$route = $request->route()` 或 `$route = app('router')->getCurrentRoute()`
4. 获取当前路由需要执行的中间件 `$middlewares = app('router')->gatherRouteMiddleware($route)`


## Url 生成器
Url 生成器是什么？
举个例子，
```
$url = new UrlGenerator(
    $routes = new RouteCollection,
    $request = Request::create('http://www.foo.com/')
);

$url->to('foo/bar'); // 输出 http://www.foo.com/foo/bar
```
像这种基于当前请求，生成指定路径的`Url`；

这部分功能由两个文件完成，一个是`UrlGenerator.php`，另一个是`RouteUrlGenerator.php`；`UrlGenerator.php`处理根据路径名生成`Url`，`RouteUrlGenerator.php`处理根据路由生成`Url`；

列一些常用的使用：

### 根据路径名生成
使用`to`方法，第一个参数为路径，第二个参数是数组，`implode`后会接着路径名，第三个参数决定用不用`https`
```
// 路径名是 foo/bar，当前请求的根路径为 http://www.foo.com，所以输出是 http://www.foo.com/foo/bar
$url->to('foo/bar')
// 路径名是 foo/bar，当前请求的根路径为 http://www.foo.com，第三个参数决定 scheme 是 https，所以输出是 https://www.foo.com/foo/bar
$url->to('foo/bar', [], true)
// 路径名是 foo/bar，第二个参数 是补充路径名，implode 后是 /baz/boom
// 第三个参数决定 scheme 是 https，所以输出是 https://www.foo.com/foo/bar/baz/boom
$url->to('foo/bar', ['baz', 'boom'], true)
// 路径名是 foo/bar，查询参数是 ?foo=bar ，补充路径是 /baz，所以输出是 https://www.foo.com/foo/bar/baz?foo=bar
$url->to('foo/bar?foo=bar', ['baz'], true)
```

### 根据路由的 as 名生成
使用`route`方法，第一个参数为指定路由的 `as` 名，第二个参数是参数数组，第三个参数决定是否显示根目录（默认为 true）
```
$route = new Route(['GET'], 'foo/bar', ['as' => 'foo']);
$routes->add($route);

// 输出 'http://www.foo.com/foo/bar
$url->route('foo');

// 第三个参数为 false，表示不显示根目录，于是输出 /foo/bar
$url->route('foo', [], false)

// 路由中的 url 本身不带参数，则第二参数中所有关联数组都将作为查询参数
// 输出 /foo/bar?foo=bar
$url->route('foo', ['foo' => 'bar'], false)
```
```
$route = new Route(['GET'], 'foo/bar/{baz}/breeze/{boom}', ['as' => 'bar']);
$routes->add($route);

// 路由上的 url 带参数，根据参数名找值；剩余多余的为查询参数；
// 输出 http://www.foo.com/foo/bar/otwell/breeze/taylor?fly=wall
$url->route('bar', ['boom' => 'taylor', 'baz' => 'otwell', 'fly' => 'wall']);

// 路由上的 url 带参数，找不到对应的参数值，则按顺序作值；剩余多余的为查询参数；
// 输出 http://www.foo.com/foo/bar/taylor/breeze/otwell?fly=wall
$url->route('bar', ['taylor', 'otwell', 'fly' => 'wall']);
```

### 根据路由的 action 名生成
使用`action`方法，第一个参数为指定路由的 `action` 名，第二个参数是参数数组，第三个参数决定是否显示根目录（默认为 true）
```
$route = new Route(['GET'], 'foo/bam', ['controller' => 'foo@bar']);
$routes->add($route);

// 输出 http://www.foo.com/foo/bam
$url->action('foo@bar');
```

```
$route = new Route(['GET'], 'foo/invoke', ['controller' => 'InvokableActionStub']);
$routes->add($route);

// 输出 http://www.foo.com/foo/invoke
$url->action('InvokableActionStub');
```

### 设置全局默认参数
```
$url->defaults(['locale' => 'en']);

$route = new Route(['GET'], 'foo', ['as' => 'defaults', 'domain' => '{locale}.example.com', function() {}]);

// 路由 url 有参数，但没有传参数值，则会找全局默认参数值；输出 http://en.example.com/foo
$url->route('defaults');
```

### 设置全局命名空间
这样调用的时候，不用在 action 上省略这部分命名空间
```
// 设置全局命名空间
$url->setRootControllerNamespace('namespace');

// 配置添加路由
$route = new Route(['GET'], 'foo/bar', ['controller' => 'namespace\foo@bar']);
$routes->add($route);
$route = new Route(['GET'], 'foo/invoke', ['controller' => 'namespace\InvokableActionStub']);
$routes->add($route);

// 输出 http://www.foo.com/foo/bar;  action 的值省略 namespace 这个命名空间
$url->action('foo@bar');
// 输出 http://www.foo.com/foo/invoke;  action 的值省略 namespace 这个命名空间
$url->action('InvokableActionStub');

// 配置添加路由
$route = new Route(['GET'], 'something/else', ['controller' => 'something\foo@bar']);
$routes->add($route);

// 输出  http://www.foo.com/something/else； action 的最前面加了 `\`，全局命名空间下调用
$url->action('\something\foo@bar')；
```

## 跳转器
跳转器内部提供了以下跳转；

**home**
通过调用`app('redirect')->home()`会跳转至根目录下`\`；
```
public function home($status = 302)
```

**back**
通过调用`app('redirect')->back()`会跳转至上一次访问页面；或者全局帮助函数`back()`也可以；
```
public function back($status = 302, $headers = [], $fallback = false)
```
第三个参数表示，如果没有前一次访问请求，访问哪个页面，具体源码如下：
```
if ($url) {
     return $url;
} elseif ($fallback) {
    return $this->to($fallback);
} else {
    return $this->to('/');
}
```

**refresh**
通过调用`app('redirect')->refresh()`会刷新当前访问页面；
```
public function refresh($status = 302, $headers = [])
```

**to**
通过调用`app('redirect')->to('path')`会跳转至指定路径页面；或者全局帮助函数`redirect('path')`也可以；

这里的 `path` 路径是不包含根目录的，例如（`foo/bar`）；

```
public function to($path, $status = 302, $headers = [], $secure = null)
```
第四个参数表示是否使用`https`；

**away**
通过调用`app('redirect')->away('path')`会跳转至指定路径页面；

这里的 `path` 路径是包含根目录的，例如（`http://xx.com/foo/bar`）；
```
public function away($path, $status = 302, $headers = [])
```

**secure**
通过调用`app('redirect')->secure('path')`会跳转至指定路径页面；这里的`path`路径是不包含根目录的；
```
public function secure($path, $status = 302, $headers = [])
```
其本质是调用了`to`方法
```
return $this->to($path, $status, $headers, true);
```


**route**
通过调用`app('redirect')->route('route_as_name')`，根据路由的`as`名会跳转至与路由一致的`url`路径页；
```
public function route($route, $parameters = [], $status = 302, $headers = [])
```

**action**
通过调用`app('redirect')->action('route_action')`，根据路由的`action`名会跳转至与路由一致的`url`路径页；
```
public function action($action, $parameters = [], $status = 302, $headers = [])
```

**guest**
跳到指定的路径页的同时，将当前`url`存放至`session`中，键名为`url.intended`;
```
public function guest($path, $status = 302, $headers = [], $secure = null)
```

**intended**
跳转至`session`中键名为`url.intended`的值所对应的`Url`；如果不存在，则跳转至第一个参数所传的值；
```
public function intended($default = '/', $status = 302, $headers = [], $secure = null)
```

## 响应工厂（ResponseFactory）
`ResponseFactory`文件提供了两部分 `API`，分别是与响应类型相关和与跳转相关；

### 响应
`response()`会返回`ResponseFactory`实例；

**视图响应**
```
response()->view('hello', $data, 200);
```

**`Jsop`响应**
```
response()->json(['name' => 'Abigail', 'state' => 'CA']);
```

**`Jsonp`响应**
```
response()->json(['name' => 'Abigail', 'state' => 'CA'])->withCallback($request->input('callback'));
```
**文件响应**
直接在浏览器显示文件，而不是下载，例如图片或`PDF`；`file`方法第一参数为文件路径，第二参数选填为头信息数组；
```
response()->file($pathToFile, $headers);
```
**文件下载**
`download`方法第一参数为文件路径，第二参数选填为文件名，第三参数选填为头信息数组；
```
return response()->download($pathToFile, $name, $headers);
```

### 跳转
这里的跳转方法，其实调用的还是跳转器中的方法，不过是在暴露更多的接口，方便调用与使用；

| 方法名 | 调用 | 实际调用的是跳转器中的哪个方法 |
|: --- :|: --- :|: --- :|
|`redirectTo` | response()->redirectTo(...)| `to`方法 |
|`redirectToRoute`| response()->redirectToRoute(...)| `route`方法|
|`redirectToAction`| response()->redirectToAction(...)|`action`方法 |
|`redirectGuest`| response()->redirectGuest(...)|`guest`方法 |
|`redirectToIntended`| response()->redirectToIntended(...)| `intended`方法|


  [1]: http://owk2q4gs5.bkt.clouddn.com/bVRYcN.png
