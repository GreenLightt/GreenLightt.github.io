---
title: Laravel 大将之 本地化 模块
date: 2017-08-01 17:57:41
description: 介绍如何使用本地化模块
tags:
- Laravel-5.4
categories:
- Laravel
copyright: false
---

本文是基于`Laravel 5.4`版本的本地化模块代码进行分析书写；

# 模块组成
下图展示了本地化模块各个文件的关系，并进行简要说明；
![图片描述][1]

- `TranslationServiceProvider`
   本地化模块的服务提供者，既是一个模块的入口，也是与`IOC`容器交互的中心；注册翻译器实例`translation.loader`，注册翻译管理实例`translator`，并声明延迟加载服务；
- `Translator`
   翻译管理类；
- `MessageSelector`
   消息过滤器，通过判断复数值来选择合适的消息；比如消息内容是这样的`{0}没有|[1,19]一些|[20,*]很多`，我们传的数字是 `18`，那么最后选择的消息就是"一些"；
- `LoaderInterface`
   翻译器接口；声明了三个方法`load`，`addNamespace`，`namespaces`；
- `FileLoader`
   继承了`LoaderInterface`，从文件获取本地化资源数据；
- `ArrayLoader`
   继承了`LoaderInterface`，在内存用数组维护本地化资源数据；

# 配置说明
在`config`配置目录下和本模块有关的参数只有`app.php`文件中的`locale`和`fallback_locale`；

`locale`表示默认本地化语言是什么，这样会优先从该语言资源目录中获取翻译（转换）内容；
如果`locale`表示的语言不存在，则使用`fallback_locale`这个备用语言；

笔者的`locale`是`zh_CN`，`fallback_locale`是`en`；

# 功能介绍
全局的语言资源目录在项目的`resources/lang`下，每个子目录分别以语言为名，比如`en`、`zh_CN`等；

另外一些子目录是命名空间为名，是对第三方加载库资源文件的补充替换；

有可能还存在`en.json`、`zh_CN`这类`Json`文件，项目有时候会从`Json`文件读取数据，这些数据均来自于这个已存在的`Json`文件；

## 翻译全局语言资源
笔者的语言资源根目录`resources/lang`下有`zh_CN/validation.php`，内容如下
```
<?php
return [
    'accepted'             => ':attribute 必须接受。',
    'active_url'           => ':attribute 不是一个有效的网址。',
    'after'                => ':attribute 必须是一个在 :date 之后的日期。',
    ......
];
```

通过调用代码
```
app('translator')->trans('validation.accepted', ['attribute' => '用户名'])
```
或者全局帮助函数`trans`
```
trans('validation.accepted', ['attribute' => '用户名'])
```
输出 "用户名 必须接受。"；

调用过程如下：
1. **解析键名**：将键名进行解析成数组 `($namespace = '*', $group = 'validation', $item = 'accepted')`；`namespace`为`*`，表示在全局命名空间下；`group`，组，其实就是文件名，一个文件为一组；`item`是元素的意思；
2. **获取语言数组**： 这里的`$locale`为`null`，所以返回的是默认与备用语言组成的数组，也就是`['zh_CN', 'en']`；并进行`for`循环，进入语言资源目录中寻找需要的元素值，如果找到，即 `break`；
3. **加载资源**：因为命名空间为`*`，所以定位资源根目录为`resources/lang`；语言为`zh_CN`，所以子目录为`zh_CN`；`group`名为`validation`，这时就把`resources/lang/zh_CN/validation.php`文件中的所有内容都加载进内存中，并进行保存 `$this->loaded[$namespace][$group][$locale] = $lines;`
4. **获取资源，并替换参数**：通过`Arr::get`方法从`$this->loaded[$namespace][$group][$locale]`中获取元素值`:attribute 必须接受。`；此时，参数数组为不空，循环替换，得到结果"用户名 必须接受。"；

## 翻译带命名空间的语言资源
笔者在语言资源根目录`resource/lang`下，创建`vendor/Faker/Provider/zh_CN/Internet.php`文件，内容如下：
```
<?php
return [
    'message' => 'hello, Faker/Provider',
    ......
];
```
同时，手动在`Translator`中注册第三方插件（也就是带命名空间）的资源根目录位置；
```
app('translator')->addNamespace('Faker/Provider', base_path('vendor/xx/resource/lang'))
```

现在，获取带命名空间的资源；
```
trans('Faker/Provider::Internet.message');
```
输出 'hello, Faker/Provider'；

调用过程如下：
1. **解析键名**：将键名进行解析成数组 `($namespace = 'Faker/Provider', $group = 'Internet', $item = 'message')`；
2. **获取语言数组**： 这里的`$locale`为`null`，所以返回的是默认与备用语言组成的数组，也就是`['zh_CN', 'en']`；并进行`for`循环，进入语言资源目录中寻找需要的元素值，如果找到，即 `break`；
3. **加载资源**：因为命名空间为`Faker/Provider`，此时会分两步；第一步读取第三方插件资源库下的信息，这时读取命名空间注册的根目录为`base_path('vendor/xx/resource/lang')`，就读取`base_path('vendor/xx/resource/lang')/zh_CN/Internet.php`内容，文件不存在，返回空数组；第二步读取全局语言资源，进行补充，也就是读取`base_path('resource/lang/vendor/Faker/Provider')/zh_CN/Internet.php`; 最后进行保存 `$this->loaded[$namespace][$group][$locale] = $lines;`
4. **获取资源，并替换参数**：通过`Arr::get`方法从`$this->loaded[$namespace][$group][$locale]`中获取元素值" hello, Faker/Provider"；此时，参数数组为空，直接返回结果 "hello, Faker/Provider"；

## 翻译`Json`文件中的资源
笔者在语言资源根目录`resource/lang`下，创建`zh_CN.json`文件，内容如下：
```
{
    "name": "zh_CN.json",
    "place": "../resources/lang/zh_CN.json"
}  
```

现在，获取`Json`文件中的`name`值；
```
trans('*.name')
```
输出 "zh_CN.json"；

调用过程如下：
1. **解析键名**：将键名进行解析成数组 `($namespace = '*', $group = '*', $item = 'name')`；
2. **获取语言数组**： 这里的`$locale`为`null`，所以返回的是默认与备用语言组成的数组，也就是`['zh_CN', 'en']`；并进行`for`循环，进入语言资源目录中寻找需要的元素值，如果找到，即 `break`；
3. **加载资源**：因为命名空间为`*`，且组也为`*`，这时会读取语言根目录下，名字为语言值的`Json`文件；此时会读取`resource/lang/zh_CN.json`，将读取的内容，进行保存 `$this->loaded[$namespace][$group][$locale] = $lines;`
4. **获取资源，并替换参数**：通过`Arr::get`方法从`$this->loaded[$namespace][$group][$locale]`中获取元素值"zh_CN.json"；此时，参数数组为空，直接返回结果 "zh_CN.json"；

## 运行时绑定资源
资源的内容除了放在文件中，用到的时候在读取，也可以在项目运行时，存放；
以`resources/lang/zh_CN/validation.php`为例，现在想要在运行时，给这个组添加一个新的元素叫 `extra`，需要指定放在哪个语言下，可以这样写
```
app('translator')->addLines(array('validation.extra' => '测试添加额外数据'), 'zh_CN');
```

现在可以获取这个新添加的元素值
```
trans('validation.extra')
```

## 复数资源过滤
笔者通过 *运行时绑定资源* 添加一条翻译内容：
```
app('translator')->addLines(array('validation.extra' => '{0}没有|[1,19]一些|[20,*]很多'), 'zh_CN'); 
```
如果通过`trans('validation.extra')`，获取的就是整条翻译内容，不是我们所期望的；用`choice`方法：

`app('translator')->choice('validation.extra', 0)` 得到 `没有`；
`app('translator')->choice('validation.extra', 18)` 得到 `一些`；
`app('translator')->choice('validation.extra', 20)` 得到 `很多`；

可以将`app('translator')->choice(...)`简写成全局帮助函数`trans_choice(...)`；


  [1]: http://owk2q4gs5.bkt.clouddn.com/bVR1ec.png
