---
title: Laravel 用法之 Cache 模块
date: 2017-12-25 15:44:41
description: 转载 Laravel 学院的缓存相关文章并略加精简
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

本文转载 [[ Laravel 5.1 文档 ] 服务 —— 缓存](http://laravelacademy.org/post/176.html)；

官方 `API` 地址 [https://laravel.com/api/5.1/Illuminate/Cache.html](https://laravel.com/api/5.1/Illuminate/Cache.html)

# 配置
`Laravel` 为不同的缓存系统提供了统一的 `API`。缓存配置位于 `config/cache.php`。

## 数据库驱动
使用 `database` 缓存驱动时，你需要设置一张表包含缓存缓存项。下面是该表的 `Schema` 声明：

```
Schema::create('cache', function($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```

## Memcached 驱动
使用 `Memcached` 缓存要求安装了 `Memcached PECL` 包，即 `PHP Memcached` 扩展。

`Memcached::addServer` 默认配置使用 `TCP/IP` 协议：

```
'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],
```

你还可以设置 `host` 选项为 `UNIX socket` 路径，如果你这样做，`port` 选项应该置为 0：

```
'memcached' => [
    [
        'host' => '/var/run/memcached/memcached.sock',
        'port' => 0,
        'weight' => 100
    ],
],
```



## Redis 驱动
使用 `Laravel` 的 `Redis` 缓存之前，你需要通过 `Composer` 安装 `predis/predis` 包（~1.0）。


# 缓存使用

## 获取缓存实例

```
Cache::getDefaultDriver()

Cache::get('store_name')
```

**指定缓存存储器**
使用 `Cache` 门面，你可以使用 `store` 方法访问不同的缓存存储器，传入 `store` 方法的键就是 `cache` 配置文件中 `stores` 配置数组里列出的相应的存储器：

```
Cache::store('file')->get('foo')
```

##  获取数据
`Cache` 门面的 `get` 方法用于从缓存中获取缓存项，如果缓存项不存在，返回 `null` 。如果需要的话你可以传递第二个参数到 `get` 方法指定缓存项不存在时返回的自定义默认值：

```
$value = Cache::get('key');
$value = Cache::get('key', 'default');

$value = Cache::get('key', function() {
    return DB::table(...)->get();
});
```

**检查缓存项是否存在**
```
if (Cache::has('key')) {
    //
}
```

**数值增加/减少**

```
Cache::increment('key');
Cache::increment('key', $amount);

Cache::decrement('key');
Cache::decrement('key', $amount);
```

**获取或更新**
如果缓存项不存在，传递给`remember`方法的闭包被执行并且将结果存放到缓存中。

```
$value = Cache::remember('users', $minutes, function() {
    return DB::table('users')->get();});
    
$value = Cache::rememberForever('users', function() {
    return DB::table('users')->get();});
```

**获取并删除**
如果你需要从缓存中获取缓存项然后删除，你可以使用 `pull` 方法，和 `get` 方法一样，如果缓存项不存在的话返回 `null`：

```
$value = Cache::pull('key');
```

## 存储缓存
你可以使用 `Cache` 门面上的 `put` 方法在缓存中存储缓存项。当你在缓存中存储缓存项的时候，你需要指定数据被缓存的时间（分钟数）：

```
Cache::put('key', 'value', $minutes);
```

除了传递缓存项失效时间，你还可以传递一个代表缓存项有效时间的`PHP Datetime`实例：

```
$expiresAt = Carbon::now()->addMinutes(10);
Cache::put('key', 'value', $expiresAt);
```

`add` 方法只会在缓存项不存在的情况下添加缓存项到缓存，如果缓存项被添加到缓存返回`true`，否则，返回`false`：

```
Cache::add('key', 'value', $minutes);
```

`forever` 方法用于持久化存储缓存项到缓存，这些值必须通过 `forget` 方法手动从缓存中移除：

```
Cache::forever('key', 'value');
```



## 移除缓存
```
Cache::forget('key');
```

# 添加自定义缓存驱动
要使用自定义驱动扩展 `Laravel` 缓存，我们使用 `Cache` 门面的 `extend` 方法，该方法用于绑定定义驱动解析器到管理器，通常，这可以在服务提供者中完成。

例如，要注册一个新的命名为“ `mongo` ”的缓存驱动：

```
use App\Extensions\MongoStore;
use Illuminate\Support\ServiceProvider;


class CacheServiceProvider extends ServiceProvider{

    public function boot() {
        Cache::extend('mongo', function($app) {
            return Cache::repository(new MongoStore);
        });
    }
    
}
```

`App\Extensions\MongoStore` 文件内容如下：

```
<?php

namespace App\Extensions;

class MongoStore implements \Illuminate\Contracts\Cache\Store{
    public function get($key) {}
    public function put($key, $value, $minutes) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

扩展完成后，只需要更新配置文件 `config/cache.php` 的 `driver` 选项为你的扩展名称。



# 缓存标签
缓存标签不支持 `file` 或 `database` 缓存驱动，此外，在“永久”存储缓存中使用多个标签时，`memcached` 之类的驱动有着最佳性能，因为它可以自动清除过期的记录。

## 存储打上标签的缓存项
缓存标签允许你给相关的缓存项打上同一个标签，然后可以输出被分配同一个标签的所有缓存值。你可以通过传递一个有序的标签名数组来访问被打上标签的缓存。例如，让我们访问一个被打上标签的缓存并将其值放到缓存中：

```
Cache::tags(['people', 'artists'])->put('John', $john, $minutes);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);
```
然而，并不只限于使用 `put` 方法，你可以在处理标签时使用任何混存存储器提供的方法。

## 访问打上标签的缓存项
要获取被打上标签的缓存项，传递同样的有序标签名数组到 `tags` 方法：

```
$john = Cache::tags(['people', 'artists'])->get('John');
$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

通过上面的语句你可以输出所有分配了该标签或标签列表的缓存项，例如，下面这个语句将会移除被打上 `people`，`authors`标签的缓存，或者，`Anne` 和 `John` 都会从缓存中移除：

```
Cache::tags(['people', 'authors'])->flush();
```

相比之下，下面这个语句只会移除被打上 `authors` 标签的缓存，所以 `Anne` 会被移除，而 `John` 不会：

```
Cache::tags('authors')->flush();
```

# 缓存事件
要在每次缓存操作时执行代码，你可以监听缓存触发的事件，通常，你可以将这些缓存处理器代码放到 `EventServiceProvider` 的 `boot` 方法中：

```
public function boot(DispatcherContract $events){
    parent::boot($events);

    $events->listen('cache.hit', function ($key, $value) {
        //
    });

    $events->listen('cache.missed', function ($key) {
        //
    });

    $events->listen('cache.write', function ($key, $value, $minutes) {
        //
    });

    $events->listen('cache.delete', function ($key) {
        //
    });
}
```





