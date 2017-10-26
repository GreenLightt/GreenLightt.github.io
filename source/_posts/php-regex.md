---
title: PHP 正则表达式函数
date: 2017-10-26 14:01:38
description: 整理 PHP 正则表达式函数
tags:
categories:
- PHP
---

在`PHP`中有两套正则表达式函数库。一套是由`PCRE`（`Perl Compatible Regular Expression`）库提供的。`PCRE` 库使用和 `Perl` 相同的语法规则实现了正则表达式的模式匹配，其使用以“`preg_`”为前缀命名的函数。另一套是由`POSIX`（`Portable Operation System interface`）扩展库提供的。`POSIX`扩展的正则表达式由`POSIX 1003.2`定义，一般使用以“`ereg_`”为前缀命名的函数。两套函数库的功能相似，执行效率稍有不同。一般而言，实现相同的功能，使用`PCRE`库的效率略占优势。


# preg_match
> 函数原型：`int preg_match (string $pattern, string $content [, array $matches]) `

`preg_match()` 函数在`$content`字符串中搜索与`$pattern`给出的正则表达式相匹配的内容。如果提供了`$matches`，则将匹配结果放入其中。`$matches[0]`将包含与整个模式匹配的文本，`$matches[1]`将包含第一个捕获的与括号中的模式单元所匹配的内容，以此类推。该函数只作一次匹配，最终返回 0 或 1 的匹配结果数。

示例：

```
$content = "Current date and time is ".date("Y-m-d h:i a").", we are learning PHP together.";

if (preg_match ("/([\d-]{10}) ([\d:]{5} [ap]m)/", $content, $matches)) {
    print_r($matches);
}
```
![此处输入链接的描述][1]

# preg_match_all
> 函数原型：`int preg_match_all( string pattern, string subject, array matches [, int flags ] )`

用于进行正则表达式全局匹配，成功返回整个模式匹配的次数（可能为零），如果出错返回 `FALSE` 。

```
$str = "<pre>学习php是一件快乐的事。</pre><pre>所有的phper需要共同努力！</pre>";

preg_match_all("/<pre>([^<]*?)<\/pre>/i", $str, $matches);

print_r($matches);
```

![此处输入链接的描述][2]

# preg_grep
> 函数原型：`array preg_grep (string $pattern, array $input) `

`preg_grep()`函数返回一个数组，其中包括了`$input`数组中与给定的`$pattern`模式相匹配的单元。对于输入数组`$input`中的每个元素，`preg_grep()`也只进行一次匹配。

示例：

```
$subjects = array(
    "Mechanical Engineering",
    "Medicine",
    "Social Science",
    "Agriculture",
    "Commercial Science",
    "Politics"
); 

$alonewords = preg_grep("/^[a-z]*$/i", $subjects);

print_r($alonewords);
```

![此处输入链接的描述][3]

# 整理常用的正则表达式
## 邮箱

```
"/^[_a-z0-9-]+(\.[_a-z0-9-]+)*@[a-z0-9-]+(\.[a-z0-9-]+)*(\.[a-z]{2,})$/"
```

## 手机号
```
"/^1[34578]\d{9}$/"
```

## 中文
```
"/([\x{4e00}-\x{9fa5}])/u"
```

## 身份证
```
"/^\d{15}|\d{18}$/"
```



  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171026110817.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171026113018.png
  [3]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171026135108.png
