---
title: Laravel 用法之 Database 模块  ORM
date: 2017-12-24 14:34:41
description: 参考 Laravel 学院的数据库相关文章整理 模型 ORM 的使用
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

# 参考阅读

- [[ Laravel 5.1 文档 ] Eloquent ORM —— 起步](http://laravelacademy.org/post/138.html)
- [[ Laravel 5.1 文档 ] Eloquent ORM —— 关联关系](http://laravelacademy.org/post/140.html)

# 定义模型

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model{

    // 如果不指定表名，默认规则是模型类名的复数作为与其对应的表名
    protected $table = 'my_flights'; 
    
    // 默认每张表的主键名为 id
    protected $primaryKey = 'flight_id';
    
    // 表明模型是否应该被打上时间戳
    public $timestamps = false;
    
    // 模型日期列的存储格式
    protected $dateFormat = 'U';
    
    // 应该被调整为日期的属性
    protected $dates = ['created_at', 'updated_at', 'disabled_at'];
}
```

注：时间戳格式可以参考 [ PHP Date()函数详细参数](https://www.cnblogs.com/glory-jzx/archive/2012/09/29/2708396.html)


## 访问器和修改器
访问器和修改器允许你在获取模型属性或设置其值时格式化 `Eloquent` 属性。例如，你可能想要使用 `Laravel` 加密器对存储在数据库中的数据进行加密，并且在 `Eloquent` 模型中访问时自动进行解密。

除了自定义访问器和修改器，`Eloquent` 还可以自动转换日期字段为 `Carbon` 实例甚至将文本转换为 `JSON`。

**定义访问器** 
比如，要访问 `ORM` 模型的 `first_name` 属性， 可以定义如下的访问器

```
// 方法名 使用驼峰式命名规则
public function getFirstNameAttribute($value) {
    return ucfirst($value);
}
```

**定义修改器**
比如，为 `first_name` 属性定义一个修改器，当我们为模型上的 `first_name` 赋值时该修改器会被自动调用：

```
public function setFirstNameAttribute($value) {
    $this->attributes['first_name'] = strtolower($value);
}
```

## 属性转换

模型中的 `$casts` 属性提供了便利方法转换属性到通用数据类型。`$casts` 属性是数组格式，其键是要被转换的属性名称，其值时你想要转换的类型。目前支持的转换类型包括：`integer`, `real`, `float`, `double`, `string`, `boolean`, `object` 和 `array`。

例如，让我们转换 `is_admin` 属性，将其由 `integer` 类型转换为 `boolean` 类型：

```
// 应该被转化为原生类型的属性
protected $casts = [
    'is_admin' => 'boolean',
];
```
现在，`is_admin` 属性在被**访问**时总是被转换为 `boolean`，即使底层存储在数据库中的值是 `integer`；

**数组转换**

`array` 类型转换在处理被存储为序列化 `JSON` 的字段是特别有用，例如，如果数据库有一个 `TEXT` 字段类型包含了序列化 `JSON`，添加 `array` 类型转换到该属性将会在 `Eloquent` 模型中访问其值时自动将其反序列化为 `PHP` 数组：

```
// 应该被转化为原生类型的属性
protected $casts = [
    'options' => 'array',
];
```

类型转换被定义后，就可以访问 `options` 属性，它将会自动从 `JSON` 反序列化为 `PHP` 数组，当你设置 `options` 属性的值时，给定数组将会自动转化为 `JSON` 以供**存储**：

```
$user = App\User::find(1);
$options = $user->options;
$options['key'] = 'value';
$user->options = $options;
$user->save();
```

## 序列化

当构建 `JSON API` 时，经常需要转化模型和关联关系为数组或 `JSON`。`Eloquent` 包含便捷方法实现这些转换，以及控制哪些属性被包含到序列化中。

`toArray()` 转化模型及其加载的关联关系为数组；
`toJson()`  转化模型及其加载的关联关系为 `JSON`；

你还可以转化模型或集合为字符串，这将会自动调用 `toJson` 方法：

```
$user = App\User::find(1);
return (string) $user;
```

**追加值到 `JSON`**
有时候，需要添加数据库中没有相应的字段到数组中，要实现这个，首先要定义一个访问器; 
定义好访问器后，添加字段名到模型的 `appends` 属性：
```
// 追加到模型数组表单的访问器
protected $appends = ['is_admin'];
```

字段被添加到 `appends` 列表之后，将会被包含到模型数组和 `JSON` 表单中，`appends` 数组中的字段还会遵循模型中的 `visible` 和 `hidden` 设置配置。

**在 `JSON` 中隐藏属性显示**

有时候你希望在模型数组或 `JSON` 显示中限制某些属性，比如密码，要实现这个，在定义模型的时候添加一个 `$hidden` 属性：

```
protected $hidden = ['password'];
```

此外，可以使用 `visible` 属性定义属性显示的白名单：

```
protected $visible = ['first_name', 'last_name'];
```

# 操作模型

## 事件
`Eloquent` 模型可以触发事件，允许你在模型生命周期中的多个时间点调用如下这些方法：`creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`。事件允许你在一个指定模型类每次保存或更新的时候执行代码。

```
class User extends Model{
    public static function boot() {
        parent::boot();
        
        // 注册 已创建事件
        static::created(function ($obj) {
        
        }
    }
}
```


## 插入
想要在数据库中插入新的记录，只需创建一个新的模型实例，设置模型的属性，然后调用  `save` 方法：

```
$flight = new Flight;
$flight->name = $request->name;
$flight->save();
```

**批量赋值**
还可以使用 `create` 方法保存一个新的模型。该方法返回被插入的模型实例。但是，在此之前，你需要指定模型的 `fillable` 或 `guarded` 属性，因为所有 `Eloquent` 模型都通过批量赋值（`Mass Assignment`）进行保护。

## 更新

`save` 方法还可以用于更新数据库中已存在的模型。要更新一个模型，应该先获取它，设置你想要更新的属性，然后调用 `save` 方法。同样，`updated_at` 时间戳会被自动更新，所以没必要手动设置其值：

```
$flight = App\Flight::find(1);
$flight->name = 'New Flight Name';
$flight->save();
```

更新操作还可以同时修改给定查询提供的多个模型实例，在本例中，所有有效且 `destination=San Diego` 的航班都被标记为延迟：

```
App\Flight::where('active', 1)->where('destination', 'San Diego')->update(['delayed' => 1]);
```

`update` 方法要求以数组形式传递键值对参数，代表着数据表中应该被更新的列。

## 删除
**实例调用 delete 方法**

```
$flight = App\Flight::find(1);
$flight->delete();
```
**主键删除**

```
App\Flight::destroy(1);
App\Flight::destroy([1, 2, 3]);
App\Flight::destroy(1, 2, 3);
```

**条件查询删除**

```
$deletedRows = App\Flight::where('active', 0)->delete();
```

### 软删除
除了从数据库删除记录外，`Eloquent` 还可以对模型进行“软删除”。当模型被软删除后，它们并没有真的从数据库删除，而是在模型上设置一个 `deleted_at` 属性并插入数据库，如果模型有一个非空 `deleted_at` 值，那么该模型已经被软删除了。要启用模型的软删除功能，可以使用模型上的`Illuminate\Database\Eloquent\SoftDeletestrait`并添加 `deleted_at` 列到 `$dates` 属性：

```
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model{
    use SoftDeletes;
    // 应该被调整为日期的属性
    protected $dates = ['deleted_at'];
}
```

现在，当调用模型的 `delete` 方法时，`deleted_at` 列将被设置为当前日期和时间，并且，当查询一个使用软删除的模型时，被软删除的模型将会自动从查询结果中排除。

判断给定模型实例是否被软删除，可以使用 `trashed` 方法：

```
if ($flight->trashed()) {
    //
}
```

软删除模型将会自动从查询结果中排除，但是，如果你想要软删除模型出现在查询结果中，可以使用 `withTrashed` 方法：

```
$flights = App\Flight::withTrashed()->where('account_id', 1)->get();
```

只获取软删除模型 `onlyTrashed`

```
$flights = App\Flight::onlyTrashed()->where('airline_id', 1)->get();
```

恢复软删除模型

```
$flight->restore();

App\Flight::withTrashed()->where('airline_id', 1)->restore();
```

永久删除模型

```
// 强制删除单个模型实例...
$flight->forceDelete();
```

## 查询
作用域允许你定义一个查询条件的通用集合，这样就可以在应用中方便地复用。例如，你需要频繁获取最受欢迎的用户，要定义一个作用域，只需要简单的在 `Eloquent` 模型方法前加上一个 `scope` 前缀：

```
public function scopeActive($query) {
    return $query->where('active', 1);
}
```

# 关联关系

## 一对一
比如，用户关联手机

```
class User extends Model {

    public function phone() {
        return $this->hasOne('App\Phone');
    }
    
}

class Phone extends Model{

    public function user() {
        return $this->belongsTo('App\User');
    }
    
}
```

## 一对多
比如，一篇博客文章拥有无数评论

```
class Post extends Model{
    
    public function comments() {
        return $this->hasMany('App\Comment');
    }
}

class Comment extends Model{

    public function post() {
        return $this->belongsTo('App\Post');
    }
}

```

## 多对多
比如，一个用户有多个角色，同时一个角色被多个用户共用；要定义这样的关联关系，需要三个数据表：`users`、`roles` 和 `role_user`，`role_user` 表按照关联模型名的字母顺序命名，并且包含 `user_id` 和 `role_id` 两个列。

```
class User extends Model { 

    public function roles() {
        // 如果中间表没有按照关联模型名的字母顺序命名
        // return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'role_id');
        return $this->belongsToMany('App\Role');
    }
    
}

class Role extends Model {

    public function users() {
        return $this->belongsToMany('App\User');
    }
    
}
```

### 中间表
**获取中间表的列**

```
$user = App\User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```
默认情况下，只有模型键才能用在 `pivot` 对象上，如果你的 `pivot` 表包含额外的属性，必须在定义关联关系时进行指定：

```
return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
```

如果你想要你的 `pivot` 表自动包含 `created_at` 和 `updated_at` 时间戳，在关联关系定义时使用 `withTimestamps` 方法：

```
return $this->belongsToMany('App\Role')->withTimestamps();
```

## 远层的一对多

例如，`Country` 模型通过中间的 `User` 模型可能拥有多个 `Post` 模型。在这个例子中，你可以轻易的聚合给定国家的所有文章，让我们看看定义这个关联关系需要哪些表：

```
countries
    id - integer
    name - string

users
    id - integer
    country_id - integer
    name - string

posts
    id - integer
    user_id - integer
    title - string
```

既然我们已经查看了该关联关系的数据表结构，接下来让我们在 `Country` 模型上进行定义：

```
class Country extends Model {
    public function posts() {
        // return $this->hasManyThrough('App\Post', 'App\User');
        return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
    }
}
```

## 多态关联
多态关联允许一个模型在单个关联下属于多个不同模型。例如，假如你想要为产品和职工存储照片，使用多态关联，你可以在这两种场景下使用单个 `photos` 表，首先，让我们看看构建这种关联关系需要的表结构：

```
staff
    id - integer
    name - string

products
    id - integer
    price - integer

photos
    id - integer
    path - string
    imageable_id - integer
    imageable_type - string
```

接下来，让我们看看构建这种关联关系需要在模型中定义什么：

```
class Photo extends Model{
    /**
     * 获取所有拥有的imageable模型
     */
    public function imageable()
    {
        return $this->morphTo();
    }
}

class Staff extends Model{
    /**
     * 获取所有职员照片
     */
    public function photos()
    {
        return $this->morphMany('App\Photo', 'imageable');
    }
}

class Product extends Model{
    /**
     * 获取所有产品照片
     */
    public function photos()
    {
        return $this->morphMany('App\Photo', 'imageable');
    }
}
```

**获取多态关联**
要访问一个职员的所有照片，可以通过使用 `photos` 的动态属性：
```
$staff = App\Staff::find(1);

foreach ($staff->photos as $photo) {
    //
}
```

你还可以通过访问调用`morphTo`方法名来从多态模型中获取多态关联的所属对象。在本例中，就是`Photo`模型中的`imageable`方法。因此，我们可以用动态属性的方式访问该方法：

```
$photo = App\Photo::find(1);
$imageable = $photo->imageable;
```
`Photo` 模型上的 `imageable` 关联返回 `Staff` 或 `Product` 实例，这取决于那个类型的模型拥有该照片。

## 多对多多态关联
除了传统的多态关联，还可以定义“多对多”的多态关联，例如，一个博客的 `Post` 和 `Video` 模型可能共享一个 `Tag` 模型的多态关联。使用对多对的多态关联允许你在博客文章和视频之间有唯一的标签列表。首先，让我们看看表结构：

```
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

接下来，我们准备在模型中定义该关联关系。

```

class Post extends Model{

    // 获取指定文章所有标签
    public function tags() {
        return $this->morphToMany('App\Tag', 'taggable');
    }
}

class Tag extends Model{

    // 获取所有分配该标签的文章
    public function posts() {
        return $this->morphedByMany('App\Post', 'taggable');
    }

    // 获取分配该标签的所有视频
    public function videos() {
        return $this->morphedByMany('App\Video', 'taggable');
    }
}
```

# 关联查询
## 查询已存在的关联关系
访问一个模型的记录的时候，你可能希望基于关联关系是否存在来限制查询结果的数目。例如，假设你想要获取所有至少有一个评论的博客文章，要实现这个，可以传递关联关系的名称到`has`方法：

```
// 获取所有至少有一条评论的文章...
$posts = App\Post::has('comments')->get();
// 获取所有至少有三条评论的文章...
$posts = Post::has('comments', '>=', 3)->get();
// 获取所有至少有一条评论获得投票的文章...
$posts = Post::has('comments.votes')->get();
```

如果你需要更强大的功能，可以使用 `whereHas` 和 `orWhereHas` 方法将 `where` 条件放到 `has` 查询上，这些方法允许你添加自定义条件约束到关联关系条件约束，例如检查一条评论的内容：

```
// 获取所有至少有一条评论包含foo字样的文章
$posts = Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

## 渴求式加载

```
$books = App\Book::with('author')->get();
// 渴求式加载多个关联关系
$books = App\Book::with('author', 'publisher')->get();
// 嵌套的渴求式加载
$books = App\Book::with('author.contacts')->get();
```

**带条件约束的渴求式加载**

```
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();

$users = App\User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

## 懒惰渴求式加载
有时候你需要在父模型已经被获取后渴求式加载一个关联关系。例如，这在你需要动态决定是否加载关联模型时可能很有用：

```
$books = App\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

如果你需要设置更多的查询条件到渴求式加载查询上，可以传递一个闭包到 `load` 方法：

```
$books->load(['author' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

# 插入关联模型
## 基本使用
### save 方法

`Eloquent` 提供了便利的方法来添加新模型到关联关系。例如，也许你需要插入新的 `Comment` 到 `Post` 模型，你可以从关联关系的 `save` 方法直接插入 `Comment`而不是手动设置`Comment`的`post_id`属性：

```
$comment = new App\Comment(['message' => 'A new comment.']);
$post = App\Post::find(1);
$comment = $post->comments()->save($comment);

$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);
```

当处理**多对多关联**的时候，`save` 方法以数组形式接收额外的中间表属性作为第二个参数：

```
App\User::find(1)->roles()->save($role, ['expires' => $expires]);
```

### create 方法

除了 `save` 和 `saveMany` 方法外，还可以使用 `create` 方法，该方法接收属性数组、创建模型、然后插入数据库。`save` 和 `create` 的不同之处在于 `save` 接收整个 `Eloquent` 模型实例而 `create` 接收原生 `PHP` 数组：

```
$post = App\Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```
使用 `create` 方法之前确保先浏览属性批量赋值文档。

### 更新“属于”关联
更新 `belongsTo` 关联的时候，可以使用 `associate` 方法，该方法会在子模型设置外键：

```
$account = App\Account::find(10);
$user->account()->associate($account);
$user->save();
```

移除 `belongsTo` 关联的时候，可以使用 `dissociate` 方法。该方法在子模型上取消外键和关联：

```
$user->account()->dissociate();
$user->save();
```




## 多对多关联
### 附加/分离
处理多对多关联的时候，`Eloquent` 提供了一些额外的帮助函数来使得处理关联模型变得更加方便。例如，让我们假定一个用户可能有多个角色同时一个角色属于多个用户，要通过在连接模型的中间表中插入记录附加角色到用户上，可以使用 `attach` 方法：

```
$user = App\User::find(1);
$user->roles()->attach($roleId);
```

附加关联关系到模型，还可以以数组形式传递额外被插入数据到中间表：

```
$user->roles()->attach($roleId, ['expires' => $expires]);
```
当然，有时候有必要从用户中移除角色，要移除一个多对多关联记录，使用 `detach` 方法。`detach` 方法将会从中间表中移除相应的记录；然而，两个模型在数据库中都保持不变：

```
// 从指定用户中移除角色...
$user->roles()->detach($roleId);
// 从指定用户移除所有角色...
$user->roles()->detach();
```

为了方便，`attach` 和 `detach` 还接收数组形式的 `ID` 作为输入：

```
$user = App\User::find(1);
$user->roles()->detach([1, 2, 3]);
$user->roles()->attach([1 => ['expires' => $expires], 2, 3]);
```



### 同步
你还可以使用 `sync` 方法构建多对多关联。`sync` 方法接收数组形式的 `ID` 并将其放置到中间表。任何不在该数组中的 `ID` 对应记录将会从中间表中移除。因此，该操作完成后，只有在数组中的 `ID` 对应记录还存在于中间表：
```
$user->roles()->sync([1, 2, 3]);
```
你还可以和 `ID` 一起传递额外的中间表值：
```
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

## 触发父级时间戳
当一个模型属于另外一个时，例如 `Comment` 属于 `Post`，子模型更新时父模型的时间戳也被更新将很有用，例如，当 `Comment` 模型被更新时，你可能想要“触发”创建其所属模型 `Post` 的 `updated_at` 时间戳。`Eloquent` 使得这项操作变得简单，只需要添加包含关联关系名称的 `touches` 属性到子模型中即可：

```
class Comment extends Model{
    // 要触发的所有关联关系
    protected $touches = ['post'];

    // 评论所属文章
    public function post() {
        return $this->belongsTo('App\Post');
    }
}
```
现在，当你更新 `Comment` 时，所属模型 `Post` 将也会更新其 `updated_at` 值：

```
$comment = App\Comment::find(1);
$comment->text = 'Edit to this comment!';
$comment->save();
```
