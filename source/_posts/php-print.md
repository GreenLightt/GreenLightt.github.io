---
title: PHP 常用的打印方法
date: 2017-10-19 23:16:38
description: 总结 PHP 的打印方法
tags:
categories:
- PHP
---


# PHP 常用的打印方法

打印方法用于输出变量信息，输出对象等，可以方便调试等；

# echo
`echo()` 实际上不是一个函数，是 `php` 语句，因此无需对其使用括号。不过，如果希望向 echo() 传递一个以上的参数，那么使用括号会发生解析错误。而且 `echo` 是返回 `void` 的，并不返回值，所以不能使用它来赋值。

示例：
```
// 输出 words
echo "words"; 

// 输出 words
echo("words"); 

// 不用括号的时候可以用逗号隔开多个值， 会输出 alicebillcartdaring  不管是否换行，最终显示都是为一行
echo "alice", "bill", "cart", "daring";

// 如果 $firstname = "alice", 则会输出 alice com.  
echo "$fistname com";

// 由于使用单引号，所以不会输出 $firstname 的值，而是输出 $firstname com
echo '$firstname com'; 

```

# print
`print()` 和 `echo()` 用法一样，但是 `echo` 速度会比 `print` 快一点点。实际上它也不是一个函数，因此无需对其使用括号。不过，如果希望向 `print()` 传递一个以上的参数，那么使用括号会发生解析错误。注意 `print` 总是返回 1 的，这个和 `echo` 不一样，也就是可以使用 `print` 来赋值，不过没有实际意义。

示例：
```

$a = print("alice"); // 这个是允许的  
echo $a; // $a的值是1 
```

# print_r
`print_r` 函数打印关于变量的易于理解的信息。

如果变量是 `string` , `integer` 或 `float` , 将会直接输出其值，如果变量是一个数组，则会输出一个格式化后的数组，便于阅读，也就是有 `key` 和 `value` 对应的那种格式。对于 `object` 对象类同。

示例：
```
$a = "alice";
print_r($a);  // 输出 "alice"
```

# printf
`printf` 函数返回一个格式化后的字符串。
语法：`printf(format, arg1, arg2, arg...)`

参数 `format` 是转换的格式，以百分比符号 (“`%`”) 开始到转换字符结束。下面是可能的 `format` 值

| `format`符号 | 呈现 |
|: --- :|: --- :|
| %% | 返回百分比符号 |
| %b | 二进制数 |
| %c | 依照 ASCII 值的字符 |
| %d | 带符号十进制数 |
| %e | 可续计数法（比如 1.5e+3） |
| %u | 无符号十进制数 |
| %f | 浮点数(local settings aware) |
| %F | 浮点数(not local settings aware) |
| %o | 八进制数 |
| %s | 字符串 |
| %x | 十六进制数（小写字母） |
| %X | 十六进制数（大写字母） |

示例：
```
printf("My name is %s %s。","alice", "com"); // 输出 My name is alice com。 
```

# sprintf
此函数使用方法和 `printf` 一样，唯一不同的就是该函数把格式化的字符串写写入一个变量中，而不是输出来。

示例：
```
//你会发现没有任何东西输出的。
sprintf("My name is %1\$s %1\$s","alice", "com");


$out = sprintf("My name is %1\$s %2\$s","alice", "com");
//输出 My name is alice com
echo $out;
```

# var_dump
功能: 输出变量的内容、类型或字符串的内容、类型、长度。常用来调试。

![此处输入链接的描述][1]

# var_export
此函数返回关于传递给该函数的变量的结构信息，它和 `var_dump()` 类似，不同的是其返回的表示是合法的 `PHP` 代码。

![此处输入链接的描述][2]


  [1]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171019230737.png
  [2]: http://owk2q4gs5.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20171019230724.png