---
title: PHP 生成器及协程
date: 2018-02-05 17:14:38
description: 整理 PHP 生成器及协程的知识点；
tags:
categories:
- PHP
---

# 生成器

生成器提供了一种更容易的方法来实现简单的对象迭代，相比较定义类实现 `Iterator` 接口的方式，性能开销和复杂性大大降低。

生成器允许你在 `foreach` 代码块中写代码来迭代一组数据而不需要在内存中创建一个数组,那会使你的内存达到上限，或者会占据可观的处理时间。相反，你可以写一个生成器函数，就像一个普通的自定义函数一样,和普通函数只返回一次不同的是, 生成器可以根据需要 `yield` 多次，以便生成需要迭代的值。

自 `PHP 5.5+` 支持生成器；

```
Generator implements Iterator {
    //返回当前产生的值
    public mixed current ( void )
    //返回当前产生的键
    public mixed key ( void )
    //生成器继续执行
    public void next ( void )
    //重置迭代器,如果迭代已经开始了，这里会抛出一个异常。
    public void rewind ( void )
    //向生成器中传入一个值，当前yield接收值，然后继续执行下一个yield
    public mixed send ( mixed $value )
    //向生成器中抛入一个异常
    public void throw ( Exception $exception )
    //检查迭代器是否被关闭,已被关闭返回 FALSE，否则返回 TRUE
    public bool valid ( void )
    //序列化回调
    public void __wakeup ( void )
    //返回generator函数的返回值，PHP version 7+
    public mixed getReturn ( void )
}
```

> 生成器函数的核心是 `yield` 关键字。它最简单的调用形式看起来像一个 `return` 申明，不同之处在于普通 `return` 会返回值并终止函数的执行，而 `yield` 会返回一个值给循环调用此生成器的代码并且只是暂停执行生成器函数。

> 一个生成器不可以返回值：这样做会产生一个编译错误。然而 `return ` 空是一个有效的语法并且它将会终止生成器继续执行。

示例

```
function xrange($start, $end, $step = 1) {  
    for ($i = $start; $i <= $end; $i += $step) {  
        yield $i;  
    }  
}  

foreach (xrange(1, 1000) as $num) {  
    echo $num, "\n";  
} 
```

如果了解过迭代器的朋友，就可以通过上面这一段代码看出 `Generators` 的运行流程

```
调用 `xrang()` 的时候，里面的代码并没有真正的执行，而是返回了一个生成器对象


Generators::rewind()  第一次执行迭代器 隐式调用 
Generators::valid() 检查迭代器是否被关闭
Generators::current() 返回当前产生的值


Generators::next() 生成器继续执行
Generators::valid() 
Generators::current() 


 ...
 Generators::next() 
 Generators::valid() 直到返回 false 迭代结束
```


# 交互 

除了实现 `Iterator` 的接口之外 `Generator` 还添加了 `send` 方法，用来向生成器传入一个值，并且当做 `yield` 表达式的结果，然后继续执行生成器，直到遇到下一个 `yield` 后会再次停住。

> `Generator::send` 如果当这个方法被调用时，生成器不在 `yield` 表达式，那么在传入值之前，它会先运行到第一个 `yield` 表达式。

通过示例，来了解 `Generator::send` 的用法

```

function printer() {
    $i = 1;
    while(true) {
        echo 'this is the yield ' . (yield $i) . "\n";
        $i++;
    }
}

$printer = printer();
var_dump($printer->send('first'));
var_dump($printer->send('second'));

/* 
 * output
 * 
 * this is the yield first
 * int(2)
 * this is the yield second
 * int(3)
 */  
```

1. `$printer = printer();`
  代码返回一个生成器对象
2. `$printer->send('first')`
  此时生成器还未开始运行；于是先调用 `rewind`,执行 `yield 返回`; 接着将 `send` 的参数赋值作为 `yield` 表达式的值，继续执行到 下一个 `yield 返回` 后停止；等待下一次输入；


不仅仅能够 `send`，`PHP` 还提供了一个 `throw`，允许我们返回一个异常给 `Generator`

```
$Generatorg = call_user_func(function(){
    $hello = (yield '[yield] say hello'.PHP_EOL);
    echo $hello.PHP_EOL;
    try{
        $jump = (yield '[yield] I jump,you jump'.PHP_EOL);
    }catch(Exception $e){
        echo '[Exception]'.$e->getMessage().PHP_EOL;
    }
});

$hello = $Generatorg->current();
echo $hello;
$jump = $Generatorg->send('[main] say hello');
echo $jump;
$Generatorg->throw(new Exception('[main] No,I can\'t jump'));

/*
 * output
 *
 * [yield] say hello
 * [main] say hello
 * [yield] I jump,you jump
 * [Exception][main] No,I can't jump
 */
```

# 协程

协程的支持是在迭代生成器的基础上， 增加了可以回送数据给生成器的功能(调用者发送数据给被调用的生成器函数)。 这就把生成器到调用者的单向通信转变为两者之间的**双向通信**。 上面的例子很好地进行了示范；

你也许会疑惑“为了双向通信我为什么要使用协程呢？我完全可以使用其他非协程方法实现同样的功能啊?”, 是的, 你是对的, 但上面的例子只是为了演示了基本用法, 这个例子其实并没有真正的展示出使用协程的优点。

正如上面介绍里提到的,协程是非常强大的概念,不过却应用的很稀少而且常常十分复杂。要给出一些简单而真实的例子很难。

在这篇文章里,我决定去做的是使用**协程实现多任务协作**。我们要解决的问题是你想并发地运行多任务(或者“程序”）。不过我们都知道 `CPU`  在一个时刻只能运行一个任务（不考虑多核的情况）。 因此处理器需要在不同的任务之间进行切换,而且总是让每个任务运行 “一小会儿”。

多任务协作这个术语中的“协作”很好的说明了如何进行这种切换的：它要求当前正在运行的任务自动把控制传回给调度器,这样就可以运行其他任务了. 这与“抢占”多任务相反, 抢占多任务是这样的：调度器可以中断运行了一段时间的任务, 不管它喜欢还是不喜欢. 协作多任务在 `Windows` 的早期版本( `windows95` )和 `Mac OS` 中有使用, 不过它们后来都切换到使用抢先多任务了。 理由相当明确：如果你依靠程序自动交出控制的话, 那么一些恶意的程序将很容易占用整个 `CPU`, 不与其他任务共享。

现在你应当明白协程和任务调度之间的关系：`yield` 指令提供了任务中断自身的一种方法, 然后把控制交回给任务调度器. 因此协程可以运行多个其他任务. 更进一步来说, `yield` 还可以用来在任务和调度器之间进行通信.

此处示例详见 [Laurance 在PHP中使用协程实现多任务调度](http://www.laruence.com/2015/05/28/3038.html)
