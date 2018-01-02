---
title: Laravel 用法之 验证 模块
date: 2018-1-2 17:11:41
description: 转载 Laravel 学院的验证相关文章并略加精简
tags:
- Laravel-5.1
categories:
- Laravel
copyright: false
---

本文转载[[ Laravel 5.1 文档 ] 服务 —— 验证](http://laravelacademy.org/post/240.html)

官方API [https://laravel.com/api/5.1/Illuminate/Validation.html](https://laravel.com/api/5.1/Illuminate/Validation.html)

`Laravel` 提供了多种方法来验证应用输入数据。默认情况下，`Laravel` 的控制器基类使用 `ValidatesRequests trait`，该 `trait` 提供了便利的方法通过各种功能强大的验证规则来验证输入的 `HTTP` 请求。

# 快速入门
## 编写验证逻辑
现在我们准备用验证新博客文章输入的逻辑填充 `store` 方法。如果你检查应用的控制器基类（`App\Http\Controllers\Controller`），你会发现该类使用了 `ValidatesRequests trait`，这个 `trait` 在所有控制器中提供了一个便利的 `validate` 方法。

`validate` 方法接收一个 `HTTP` 请求输入数据和验证规则，如果验证规则通过，代码将会继续往下执行；然而，如果验证失败，将会抛出一个异常，相应的错误响应也会自动发送给用户。在一个传统的 `HTTP` 请求案例中，将会生成一个重定向响应，如果是 `AJAX` 请求则会返回一个 `JSON` 响应。

要更好的理解 `validate` 方法，让我们回到 `store` 方法：

```
/**
 * 存储博客文章
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request){
    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // 验证通过，存储到数据库...
}
```

正如你所看到的，我们只是传递输入的 `HTTP` 请求和期望的验证规则到 `validate` 方法，在强调一次，如果验证失败，相应的响应会自动生成。如果验证通过，控制器将会继续正常执行。

## 显示错误信息
那么，如果请求输入参数没有通过给定验证规则怎么办？正如前面所提到的，`Laravel` 将会自动将用户重定向回上一个位置。此外，所有验证错误信息会自动一次性存放到 `session`。

注意我们并没有在 `GET` 路由中明确绑定错误信息到视图。这是因为 `Laravel` 总是从 `session` 数据中检查错误信息，而且如果有的话会自动将其绑定到视图。所以，值得注意的是每次请求的所有视图中总是存在一个 `$errors` 变量，从而允许你在视图中方便而又安全地使用。`$errors` 变量是的一个 `Illuminate\Support\MessageBag` 实例。

所以，在我们的例子中，验证失败的话用户将会被重定向到控制器的 `create` 方法，从而允许我们在视图中显示错误信息：

```
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if (count($errors) > 0)
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

##  AJAX请求&验证

在这个例子中，我们使用传统的表单来发送数据到应用。然而，很多应用使用 `AJAX` 请求。在 `AJAX` 请求中使用 `validate` 方法时，`Laravel` 不会生成重定向响应。取而代之的，`Laravel` 生成一个包含验证错误信息的 `JSON` 响应。该 `JSON` 响应会带上一个 `HTTP` 状态码 `422`。


# 其它验证方法
## 手动创建验证器
如果你不想使用 `ValidatesRequests trait` 的 `validate` 方法，可以使用 `Validator` 门面手动创建一个验证器实例，该门面上的 `make` 方法用于生成一个新的验证器实例：

```
use Validator;

class PostController extends Controller{
    // 存储新的博客文章
    public function store(Request $request) {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // 存储博客文章...
    }
}
```

传递给 `make` 方法的第一个参数是需要验证的数据，第二个参数是要应用到数据上的验证规则。

检查请求是够通过验证后，可以使用 `withErrors` 方法将错误数据一次性存放到 `session`，使用该方法时，`$errors` 变量重定向后自动在视图间共享，从而允许你轻松将其显示给用户，`withErrors` 方法接收一个验证器、或者一个`MessageBag`，又或者一个 `PHP` 数组。

### 命令错误包
如果你在单个页面上有多个表单，可能需要命名 `MessageBag`，从而允许你为指定表单获取错误信息。只需要传递名称作为第二个参数给 `withErrors` 即可：

```
return redirect('register')->withErrors($validator, 'login');
```

然后你就可以从 `$errors` 变量中访问命名的 `MessageBag` 实例：

```
{{ $errors->login->first('email') }}
```

### 验证钩子之后

验证器允许你在验证完成后添加回调，这种机制允许你轻松执行更多验证，甚至添加更多错误信息到消息集合。使用验证器实例上的 `after` 方法即可：

```
$validator = Validator::make(...);

$validator->after(function($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

# 处理错误信息

调用 `Validator` 实例上的 `errors` 方法之后，将会获取一个 `Illuminate\Support\MessageBag` 实例，该实例中包含了多种处理错误信息的便利方法。

**获取某字段的第一条错误信息**

```
$messages = $validator->errors();
echo $messages->first('email');
```

**获取指定字段的所有错误信息**

```
foreach ($messages->get('email') as $message) {
    //
}
```

**获取所有字段的所有错误信息**

```
foreach ($messages->all() as $message) {
    //
}
```

**判断消息中是否存在某字段的错误信息**

```
if ($messages->has('email')) {
    //
}
```


**获取指定格式的错误信息**

```
echo $messages->first('email', '<p>:message</p>');
```

**获取指定格式的所有错误信息**

```
foreach ($messages->all('<li>:message</li>') as $message) {
    //
}
```

## 自定义错误消息

如果需要的话，你可以使用自定义错误信息替代默认的，有多种方法来指定自定义信息。首先，你可以传递自定义信息作为第三方参数给 `Validator::make` 方法：

```
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```

在本例中，`:attribute` 占位符将会被验证时实际的字段名替换，你还可以在验证消息中使用其他占位符，例如：

```
$messages = [
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute must be between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
];
```

### 为给定属性指定自定义信息
有时候你可能只想为特定字段指定自定义错误信息，可以通过”.”来实现，首先指定属性名，然后是规则：

```
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```


### 在语言文件中指定自定义消息
在很多案例中，你可能想要在语言文件中指定属性特定自定义消息而不是将它们直接传递给 `Validator`。要实现这个，添加消息到 `resources/lang/xx/validation.php` 语言文件的 `custom` 数组：

```
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```








