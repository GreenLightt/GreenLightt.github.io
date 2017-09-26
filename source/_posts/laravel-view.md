---
title: Laravel 大将之 View 模块
date: 2017-09-17 17:57:41
description: 介绍如何使用 View 模块
tags:
- Laravel-5.4
categories:
- Laravel
copyright: false
---

本文是基于`Laravel 5.4`版本的`View`模块代码进行分析书写；

# 文件结构
`View`模块的文件格局及功能如下图所示：
![图片描述][1]

视图化呈现时的大概流程：

1. 通过`view()`方法的调用，开始视图的呈现；
2. 首先，查找视图文件；
    （1）依次遍历路径，如果文件名带命名空间（也就是`::`之前的部分），则采用命名空间对应注册的路径数组，否则采用全局路径数组（在`Illuminate\View\FileViewFinder`类中的`paths`变量）；
    （2）结合当前路径，文件名，后缀名（默认顺序是`blade.php`、`php`、`css`），判断文件是否存在；
    （3）如果文件不存在，报异常：对应的`view`文件不存在；如果文件存在，则根据后缀名调用对应的引擎进行解析；
3. 如果是`css`后缀，采用`file`引擎，核心调用方法是`file_get_contents`;
4. 如果是`php`后缀，采用`php`引擎，核心调用方法是

 ```
 ob_start();
 include $__path;
 ob_get_clean();
 ```
5. 如果是`blade.php`后缀，采用`blade`引擎；
    这个引擎会主动作缓存处理，如果缓存文件未过期，则直接调用缓存文件，否则重新编译，并通过`sha1`生成缓存文件（位于`storage/framework/views`目录下）；

# Blade 引擎编译
`Blade`引擎对文件的编译，是通过大量的正则匹配和替换实现的；

```
protected $compilers = [ 
    'Comments',   // 注释部分
    'Extensions', // 扩展部分
    'Statements', // 语句块 （@ 开头的指令）
    'Echos',      // 输出
];

protected function parseToken($token) {
    list($id, $content) = $token;
    
    if ($id == T_INLINE_HTML) {
        foreach ($this->compilers as $type) {
            $content = $this->{"compile{$type}"}($content);
        }
    }
}
```
在解析的过程中，`Blade`会先使用`token_get_all`函数获取视图文件中的被`PHP`解释器认为是`HTML`（`T_INLINE_HTML`）的部分，然后依次进行`Comments`、`Extensions`、`Statements` 和 `Echos`部分的正则替换；

**注释部分**
核心代码如下，将注释符号“{{-- --}}”包裹的代码替换为空字符串；
```
preg_replace("/{{--(.*?)--}}/s", '', $value);
```

**扩展部分**
通过`extend`方法向`BladeCompiler`添加自定义处理的回调函数，对模板内容进行自定义的文本匹配替换；

核心代码在`Illuminate\View\BladeCompiler`文件中，如下：
```
// 自定义的文本替换扩展 数组
protected $extensions = [];

protected function compileExtensions($value) {
    foreach ($this->extensions as $compiler) {
        $value = call_user_func($compiler, $value, $this);
    }
    
    return $value;
}
```

**指令替换**
这部分就是将类似`@if`这种框架自带的指令和通过`directive`方法注册的指令进行文本替换；

框架提供的指令有以下十部分：

- `View\Compilers\Concerns\CompilesAuthorizations`: 权限检查
  指令包括：`@can`、`@cannot`、`@elsecan`、`@elsecannot`、`@endcan`、`@endcannot`
- `Concerns\CompilesComponents`：与组件、插槽相关
  指令包括：`@component`、`@endcomponent`、`@slot`、`@endslot`
- `Concerns\CompilesConditionals`：与判断语句相关
  指令包括：`@if`、`@unless`、`@else`、`@elseif`、`@endif`、`@endunless`、`@isset`、`@endisset`、`@hassection`
- `Concerns\CompilesIncludes`：嵌入文件
  指令包括：`@each`、`@include`、`@includeif`、`@includewhen`
- `Concerns\CompilesInjections`：服务注入
  指令包括：`@inject`
- `Concerns\CompilesLayouts`：和布局相关
  指令包括：`@extends`、`@section`、`@parent`、`@yield`、`@show`、`@append`、`@overwrite`、`@stop`、`@endsection`
- `Concerns\CompilesLoops`：与循环相关
  指令包括：`@forelse`、`@empty`、`@endforelse`、`@endempty`、`@for`、`@foreach`、`@break`、`@continue`、`@endfor`、`@endforeach`、`@while`、`@endwhile`
- `Concerns\CompilesRawPhp`：与原生PHP语句相关
  指令包括：`@php`、 `@endphp`、 `@unset`
- `Concerns\CompilesStacks`：和堆栈相关
  指令包括：`@stack`、`@push`、`@endpush`、`@prepend`、`@endprepend`
- `Concerns\CompilesTranslations`：与本地化翻译相关
  指令包括：`@lang`、`@endlang`、`@choice`

**Echo 替换**
`echo`输出是针对`{!! !!}`、`{{ }}`、`{{{ }}}`三种括号进行正则替换；

- `{!! !!}`输出未转义字符，用于输出原生带`html`标签的值；
- `{{ }}`正常输出，支持三目运算符替换；
- `{{{ }}}`输出转义字符，支持三目运算符替换；

> 三目运算符替换是指：`{{ $a ?: "默认值" }}` (或者 `{{$a or "默认值"}}`) 换成 `{{ isset($a) ? $a : "默认值"}} `



# 参考文章
- [Laravel 模板引擎（Blade）原理简析](https://segmentfault.com/a/1190000003906422)
- [Laravel 5.4 文档  前端 —— Blade模板](http://laravelacademy.org/post/6780.html#toc_17)


  [1]: http://owk2q4gs5.bkt.clouddn.com/bVVcNa.png
