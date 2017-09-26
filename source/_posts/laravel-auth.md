---
title: Laravel 大将之 Auth 模块
date: 2017-08-16 17:57:41
description: 介绍如何使用 Auth 模块
tags:
- Laravel-5.4
categories:
- Laravel
copyright: false
---

本文是基于`Laravel 5.4` 版本的`Auth`模块代码进行分析书写；

# 模块组成
`Auth`模块从功能上分为用户认证和权限管理两个部分；从文件组成上，`Illuminate\Auth\Passwords`目录下是密码重置或忘记密码处理的小模块，`Illuminate\Auth`是负责用户认证和权限管理的模块，`Illuminate\Foundation\Auth`提供了登录、修改密码、重置密码等一系统列具体逻辑实现；下图展示了`Auth`模块各个文件的关系，并进行简要说明；

![Laravel Auth 模块关系图][1]

# 用户认证
`HTTP`本身是无状态，通常在系统交互的过程中，使用账号或者`Token`标识来确定认证用户；

## 配置文件解读
```
return [
    'defaults' => [
        'guard' => 'web',
        ...
    ],
    'guards' => [  
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [    
            'driver' => 'token', 
            'provider' => 'users',
        ],
    ],
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ], 
    ],
], 
];
```
从下往上，理解；

`providers`是提供用户数据的接口，要标注驱动对象和目标对象；此处，键名`users`是一套`provider`的名字，采用`eloquent`驱动，`modal`是`App\User::class`；

`guards`部分针对认证管理部分进行配置；有两种认证方式，一种叫`web`，还有一种是`api`；`web`认证是基于`Session`交互，根据`sessionId`获取用户`id`，在`users`这个`provider`查询出此用户；`api`认证是基于`token`值交互，也采用`users`这个`provider`；

`defaults`项显示默认使用`web`认证；

## 认证
1. `Session`绑定认证信息：
 ```
 // $credentials数组存放认证条件，比如邮箱或者用户名、密码
 // $remember 表示是否要记住，生成 `remember_token`
 public function attempt(array $credentials = [], $remember = false) 
  
 public function login(AuthenticatableContract $user, $remember = false)
  
 public function loginUsingId($id, $remember = false)
 ```
 `HTTP`基本认证，认证信息放在请求头部；后面的请求访问通过`sessionId`；
 ```
 public function basic($field = 'email', $extraConditions = [])
 ```
2. 只在当前会话中认证，`session`中不记录认证信息：
 ```
 public function once(array $credentials = [])
 
 public function onceUsingId($id)
 
 public function onceBasic($field = 'email', $extraConditions = [])
 ```


认证过程中（包括注册、忘记密码），定义的事件有这些：

| 事件名 | 描述 |
|: --- :|: --- :|
| Attempting | 尝试验证事件 |
| Authenticated | 验证通过事件 |
| Failed | 验证失败事件 |
| Lockout | 失败次数超过限制，锁住该请求再次访问事件 |
| Logi | 通过‘remember_token’成功登录时，调用的事件 |
| Logout | 用户退出事件 |
| Registered | 用户注册事件 |

还有一些其他的认证方法：
1. 检查是否存在认证用户：`Auth::check()`
2. 获取当前认证用户：`Auth::user()`
3. 退出系统：`Auth::logout()`

# 密码处理
## 配置解读
```
return [
    'defaults' => [
        'passwords' => 'users',
        ...
    ],
    
    'passwords' => [
        'users' => [
            'provider' => 'users',
            'table' => 'password_resets',
            'expire' => 60,
        ],
    ],
]
```
从下往上，看配置；

`passwords`数组是重置密码的配置；`users`是配置方案的别名，包含三个元素：`provider`（提供用户的方案，是上面`providers`数组）、`table`（存放重置密码`token`的表）、`expire`（`token`过期时间）

`default` 项会设置默认的 `passwords` 重置方案；

## 重置密码的调用与实现
先看看`Laravel`的重置密码功能是怎么实现的：
```
public function reset(array $credentials, Closure $callback) {
    // 验证用户名、密码和 token 是否有效
    $user = $this->validateReset($credentials);

    if (! $user instanceof CanResetPasswordContract) {
         return $user;
    }	
    
    $password = $credentials['password'];
    // 回调函数执行修改密码，及持久化存储
    $callback($user, $password);
    // 删除重置密码时持久化存储保存的 token
    $this->tokens->delete($user);

    return static::PASSWORD_RESET;
}  
```

再看看`Foundation\Auth`模块封装的重置密码模块是怎么调用的：
```
// 暴露的重置密码 API
public function reset(Request $request)   {
    // 验证请求参数 token、email、password、password_confirmation
    $this->validate($request, $this->rules(), $this->validationErrorMessages());
    // 调用重置密码的方法，第二个参数是回调，做一些持久化存储工作
    $response = $this->broker()->reset(
        $this->credentials($request), function ($user, $password) {
	    $this->resetPassword($user, $password);
        }
    );
    // 封装 Response
    return $response == Password::PASSWORD_RESET
        ? $this->sendResetResponse($response)
        : $this->sendResetFailedResponse($request, $response);
}

// 获取重置密码时的请求参数
protected function credentials(Request $request)  {
    return $request->only(
        'email', 'password', 'password_confirmation', 'token'
    );
}

// 重置密码的真实性验证后，进行的持久化工作
protected function resetPassword($user, $password) {
    // 修改后的密码、重新生成 remember_token
    $user->forceFill([
        'password' => bcrypt($password),
        'remember_token' => Str::random(60),
    ])->save();
	// session 中的用户信息也进行重新赋值								     
    $this->guard()->login($user);
}   
```

“忘记密码 => 发邮件 => 重置密码” 的大体流程如下：
1. 点击“忘记密码”，通过路由配置，跳到“忘记密码”页面，页面上有“要发送的邮箱”这个字段要填写；
2. 验证“要发送的邮箱”是否是数据库中存在的，如果存在，即向该邮箱发送重置密码邮件；
3. 重置密码邮件中有一个链接（点击后会携带 token 到修改密码页面），同时数据库会保存这个 token 的哈希加密后的值；
4. 填写“邮箱”，“密码”，“确认密码”三个字段后，携带 `token` 访问重置密码`API`，首页判断邮箱、密码、确认密码这三个字段，然后验证 `token`是否有效；如果是，则重置成功；

# 权限管理
权限管理是依靠内存空间维护的一个数组变量`abilities`来维护，结构如下：
```
$abilities = array(
    '定义的动作名，比如以路由的 as 名(common.dashboard.list)' => function($user) {
        // 方法的参数，第一位是 $user, 当前 user, 后面的参数可以自行决定
        return true;  // 返回 true 意味有权限， false 意味没有权限
    },
    ......
);
```
但只用 `$abilities`，会使用定义的那部分代码集中在一起太烦索，所以有`policy`策略类的出现；

`policy`策略类定义一组实体及实体权限类的对应关系，比如以文章举例：

有一个 `Modal`实体类叫 `Post`，可以为这个实体类定义一个`PostPolicy`权限类，在这个权限类定义一些动作为方法名；
```
class PostPolicy {
    // update 权限，文章作者才可以修改
    public function update(User $user, Post $post) {
        return $user->id === $post->user_id;
    }
}
```
然后在`ServiceProvider`中注册，这样系统就知道，如果你要检查的类是`Post`对象，加上你给的动作名，系统会找到`PostPolicy`类的对应方法；
```
protected $policies = [
    Post::class => PostPolicy::class,
];
```

怎么调用呢？

**对于定义在`abilities`数组的权限**：

- 当前用户是否具备`common.dashboard.list`权限：`Gate::allows('common.dashboard.list')`
- 当前用户是否具备`common.dashboard.list`权限：`! Gate::denies('common.dashboard.list')`
- 当前用户是否具备`common.dashboard.list`权限：`$request->user()->can('common.dashboard.list')`
- 当前用户是否具备`common.dashboard.list`权限：`! $request->user()->cannot('common.dashboard.list')`
- 指定用户是否具备`common.dashboard.list`权限：`Gate::forUser($user)->allows('common.dashboard.list')`
 
**对于`policy`策略类调用的权限**：

- 当前用户是否可以修改文章（Gate 调用）：`Gate::allows('update', $post)`
- 当前用户是否可以修改文章（user 调用）：`$user->can('update', $post)`
- 当前用户是否可以修改文章（用帮助函数）：`policy($post)->update($user, $post)`
- 当前用户是否可以修改文章（Controller 类方法中调用）：`$this->authorize('update', $post);`
- 当前用户是否可以修改文章（Controller 类同名方法中调用）：`$this->authorize($post);`
- 指定用户是否可以修改文章（Controller 类方法中调用）：`$this->authorizeForUser($user, 'update', $post);`

## 有用的技巧
获取当前系统注册的权限，包括两部分`abilities`和`policies`数组内容，代码如下：
```
$gate = app(\Illuminate\Contracts\Auth\Access\Gate::class);
$reflection_gate = new ReflectionClass($gate);

$policies = $reflection_gate->getProperty('policies');
$policies->setAccessible(true);
// 获取当前注册的 policies 数组
dump($policies->getValue($gate));
                                                                                                        
$abilities = $reflection_gate->getProperty('abilities');                                       
$abilities->setAccessible(true);
// 获取当前注册的 abilities 数组
dump($abilities->getValue($gate));
```


  [1]: http://owk2q4gs5.bkt.clouddn.com/bVS3fX.png
