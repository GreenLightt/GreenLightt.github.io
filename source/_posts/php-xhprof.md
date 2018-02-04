---
title: PHP 性能分析工具 xhprof
date: 2017-11-15 00:02:38
description: 整理记录 xhprof
tags:
- xhprof
- PHP-Extensions
categories:
- PHP
---


`XHProf` 是 `facebook` 开发的一个 `php` 扩展，用于采集 `php` 程序中每个函数的性能开销。采集的数据包括：内存消耗、`CPU` 计算时间、函数执行时长等等。



# 安装
**下载**
```
sudo wget https://codeload.github.com/phacility/xhprof/zip/master -O xhprof.zip
```
你也可以从 [http://pecl.php.net/package/xhprof](http://pecl.php.net/package/xhprof) 这里下载。

> 注意：
php5.4 及以上版本不能在 pecl 中下载，不支持。需要在 github 上下载 https://github.com/facebook/xhprof。
另外 xhprof 已经很久没有更新过了，截至目前还不支持 php7。

**安装**
```
cd xhprof-master/
cd extension/
sudo /opt/bksite/php/bin/phpize
sudo ./configure --with-php-config=/opt/bksite/php/bin/php-config --enable-xhprof
sudo make && make install
```

**修改配置** `php.ini`

```
[xhprof]
extension=xhprof.so
xhprof.output_dir=/tmp/xhprof  定义输出文件的存放位置
```


# 使用
## 优雅地注入
目前大部分 `MVC` 框架都有唯一的入口文件，只需要在入口文件的开始处注入 `xhprof` 的逻辑

```
require_once "/tmp/xhprof-master/xhprof_lib/utils/xhprof_lib.php";
require_once "/tmp/xhprof-master/xhprof_lib/utils/xhprof_runs.php";

//开启xhprof
xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);

//注册一个函数，当程序执行结束的时候去执行它。
register_shutdown_function(function() {
    //stop profiler
    $xhprof_data = xhprof_disable();

    //冲刷(flush)所有响应的数据给客户端
    if (function_exists('fastcgi_finish_request')) {
        fastcgi_finish_request();
    }

    $xhprof_runs = new XHProfRuns_Default();

    //save the run under a namespace "xhprof_foo"
    $run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");
});
```
但是这样免不了要修改项目的源代码，其实 `php` 本身就提供了更好的注入方式，比如将上述逻辑保存为 `/opt/bitnami/php/etc/xhprof-inject.php`，然后修改 `php`配置文件 `php.ini `

```
auto_prepend_file = /opt/bitnami/php/etc/xhprof-inject.php
```
这样所有的 `php-fpm` 请求的 `php` 文件前都会自动注入 `/opt/bitnami/php/etc/xhprof-inject.php` 文件;

如果使用Nginx的话，还可以通过Nginx的配置文件设置，这样侵入性更小，并且可以实现基于站点的注入。

```
fastcgi_param PHP_VALUE "auto_prepend_file=/opt/bitnami/php/etc/xhprof-inject.php";
```

## Laravel 项目中局部使用
首先，将 `xhprof-master` 包中的 `xhprof_lib/utils/` 下的文件移到 `Laravel` 根目录下的 `utils` 文件夹中；

```
cp /tmp/xhprof-master/xhprof_lib/utils/* utils/
```
然后，配置文件的自动加载，在 `composer.json` 中修改：

```
"autoload": {
    "files":[
        ...,
        "utils/xhprof_lib.php",
        "utils/xhprof_runs.php"
    ],
    ...
}
```
执行 `composer dump-autoload`；


最后在业务代码附近加如下代码：
```
xhprof_enable(XHPROF_FLAGS_NO_BUILTINS | XHPROF_FLAGS_CPU | XHPROF_FLAGS_MEMORY);

// 业务代码

$xhprof_data = xhprof_disable();
$xhprof_runs = new \XHProfRuns_Default();
$run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_take_stock");
```

## Web 页面查看
`Nginx` 配置；（环境 bitnami）

```
server {
   listen       7777;
   server_name  localhost;

   root /tmp/xhprof-master/xhprof_html;
   index index.php index.html;  

   location ~* \.php$ {
       fastcgi_pass   unix:/opt/bitnami/php/var/run/www.sock;
       include fastcgi_params; 
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
       fastcgi_index index.php; 
   }                 
    
   include "/opt/bitnami/nginx/conf/bitnami/phpfastcgi.conf"; 
			    
   include "/opt/bitnami/nginx/conf/bitnami/bitnami-apps-prefix.conf";
}  
```

支持图表查看，得安装 `graphviz`插件；

```
yum install -y libpng
yum install -y graphviz
```

页面表格各列含义：

| 列名 | 描述 |
|: --- :|: --- :|
| `Function Name` | 函数名|
| `Calls` | 调用次数 |
| `Calls`% | 调用次数占比 |
| `Incl.Wall Time(microsec)` | 函数运行时间（包括子函数）|
| `IWall`% | 函数运行时间（包括子函数）占比 |
| `Excl.Wall Time(microsec)` | 函数运行时间（不包括子函数） |
| `EWall`% | 函数运行时间（不包括子函数）占比 |
| `Incl.CPU(microsec)` | 函数执行花费的 `CPU` 时间（包括子函数） |
| `ICpu`% | 函数执行花费的 `CPU` 时间（包括子函数）占比|
| `Excl.CPU(microsec)` | 函数执行花费的 `CPU` 时间（不包括子函数）|
| `ECpu`% | 函数执行花费的 `CPU` 时间（不包括子函数）占比|
| `Incl.MemUse(bytes)` | 函数执行占用的内存（包括子函数） |
| `IMemUse`% | 函数执行占用的内存（包括子函数）占比 |
| `Excl.MemUse(bytes)` | 函数执行占用的内存（不包括子函数）|
| `EMemUse`% | 函数执行占用的内存（不包括子函数）占比 |
| `Incl.PeakMemUse(bytes)` | `Incl.MemUse` 峰值 |
| `IPeakMemUse`% | `Incl.MemUse` 峰值占比 |
| `Excl.PeakMemUse(bytes)` | `Excl.MemUse` 峰值|
| `EPeakMemUse`% |`Excl.MemUse` 峰值占比 |

# xhprof 数据保存
注入代码后我们还需要实现保存 `xhprof` 数据以及展示数据的 `UI`，听起来似乎又是一大堆工作，有现成的轮子可以用吗？

经过搜索和比较，貌似比较好的选择有 [xhprof.io](https://github.com/gajus/xhprof.io) 以及 [xhpgui](https://github.com/perftools/xhgui)。

两个项目做得事情差不多，都提供了 `xhprof` 数据保存功能以及一套索引展示数据的 `UI`，下面是一些比较

`xhprof.io`

    ✗ 年久失修
    ✗ 保存xhprof数据到MySQL
    ✓ 支持域名、URI等多个维度的数据索引
    ✓ 函数调用记录完整，内核级别函数都能显示
    ✗ 无法针对个别URI开启
    ✗ 注入被分割成两个文件，如果程序被强制中断时xhprof数据将无法收集
    
`xhgui`

    ✓ 保存xhprof数据到MongoDB
    ✗ 不支持域名索引
    ✗ 函数调用记录不完整，部分内核级别函数（如扩展内）无法显示
    ✓ 有配置文件可以控制开启条件
    ✓ 注入只有一个文件
    ✓ 狂拽酷炫的基于D3.js的调用关系动态图






