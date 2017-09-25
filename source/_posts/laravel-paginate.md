---
title: Laravel 大将之 分页 模块
date: 2017-06-13 18:22:12
description: 介绍 Laravel 框架中两种分页器的使用
tags:
- Laravel-5.4
categories:
- Laravel
---

<!-- toc -->

# 简介
分页模块的基本使用有两种：一种是基于查询构建器或`Eloquent`模型，调用`paginate`方法；另一种是手动创建分页器；

`Laravel`框架的分页器不仅实现了数据的分页，而且支持生成`Bootstrap`的分页框，如下图所示

![][1]

# 使用
## 基于查询构建器或`Eloquent`模型
从`User`表获取数据，每页16条，可以这样写
```
$users = DB::table('user')->paginate(16);
// 或
$users = User::paginate(16);
```
这时的`$user`是`Illuminate\Pagination\LengthAwarePaginator`实例；这里没有传递当前页的原因是，如果不传递，会从`$request`请求获取当前页；
`paginate`方法完整参数定义如下：
```
paginate($perPage = null, $columns = ['*'], $pageName = 'page', $page = null)
```
其中 `$perPage` 代表每页显示数目， `$columns` 代表查询字段， `$pageName` 代表页码名称， `$page` 代表第几页。 

同理，也可以获取另一种分页，简单分页
```
$users = DB::table('user')->simplePaginate(16);
// 或
$users = User::simplePaginate(16);
```
这时的`$user`是`Illuminate\Pagination\Paginator`实例；

要想在页面呈现分页器的小方块， 只要在`blade.php` 中书写
```
    {!! $users->render() !!}
```

## 手动创建
通过看看`Laravel`的`Database`是怎么实现创建分页器，更好地学会使用手动创建；
先看看`paginate`方法，
```
public function paginate($perPage = null, $columns = ['*'], $pageName = 'page', $page = null)   
{               
    // 获取当前页数
    $page = $page ?: Paginator::resolveCurrentPage($pageName);       
    // 获取每页的数量
    $perPage = $perPage ?: $this->model->getPerPage();               
    // Collection类，存放当前页的数据记录
    $results = ($total = $this->toBase()->getCountForPagination())                              
        ? $this->forPage($page, $perPage)->get($columns)                
        : $this->model->newCollection();                                
                                                                                                     
    return new LengthAwarePaginator($results, $total, $perPage, $page, [ 
        // 当前页面的 url
        'path' => Paginator::resolveCurrentPath(),                                              
        'pageName' => $pageName,                                                                
    ]);                                                                                         
}  
```
再看看`simplePaginate`方法
```
public function simplePaginate($perPage = null, $columns = ['*'], $pageName = 'page', $page = null)
{        
    // 获取当前页数
    $page = $page ?: Paginator::resolveCurrentPage($pageName);                                  
    // 获取每页的数量                      
    $perPage = $perPage ?: $this->model->getPerPage();                                          
                                                                                                
    // 调用 Database 模块当前类的方法，获取当前页的数据；
    // 下面这个语句只是设置查询条件，get 方法调用时才是真正去获取
    $this->skip(($page - 1) * $perPage)->take($perPage + 1);                                    
                                                                                                 
    return new Paginator($this->get($columns), $perPage, $page, [  
        // 当前页面的 url
        'path' => Paginator::resolveCurrentPath(),                                              
        'pageName' => $pageName,                                                                
    ]);                                                                                         
} 
```


# 看看源代码
## 服务提供者 ServiceProvider
`boot`方法
```
public function boot()
{
    // 注册包视图
    $this->loadViewsFrom(__DIR__.'/resources/views', 'pagination');

    // 如果程序是在命令行下运行，则将模块内的`resources/views`文件夹下的文件
    // 发布一份到项目的`views/vendor/pagination`文件夹；
    // publishes 方法的第二个参数是 group 组；
    if ($this->app->runningInConsole()) {
        $this->publishes([
            __DIR__.'/resources/views' => $this->app->resourcePath('views/vendor/pagination'),
        ], 'laravel-pagination');
    }
}
```
`register`方法
```
public function register()
{
    // 绑定 view 视图解析器
    Paginator::viewFactoryResolver(function () {
        return $this->app['view'];
    });

    // 绑定 url 路径解析器，返回当前 url
    Paginator::currentPathResolver(function () {
        return $this->app['request']->url();
    });

    // 绑定 当前页 解析器，返回当前页
    Paginator::currentPageResolver(function ($pageName = 'page') {
        $page = $this->app['request']->input($pageName);

        if (filter_var($page, FILTER_VALIDATE_INT) !== false && (int) $page >= 1) {
            return $page;
        }

        return 1;
    });
}
```
分页器类有两个`Paginator`和`LengthAwarePaginator`，都继承了父类`AbstractPaginator`;两者的主要区别主要在于`render`方法，也就是呈现的`bootstrap`风格的分页器不一样；

`Paginator`的分页器，只有上一页和下一页的图标
![][2]

`LengthAwarePaginator`的分页器
![][3]


  [1]: http://owk2q4gs5.bkt.clouddn.com/bVO72Q.png 
  [2]: http://owk2q4gs5.bkt.clouddn.com/bVO73d.png
  [3]: http://owk2q4gs5.bkt.clouddn.com/bVO72Q.png
