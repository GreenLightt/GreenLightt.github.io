---
title: Laravel 大将之 加密 模块
date: 2017-06-03 17:53:00
description: 加密分两种`encrypt`加密和哈希加密
tags:
- Laravel-5.4
categories:
- Laravel
---

# 简介
加密模块包含两种加密方式，分别为`Encrypt`加密与`Hash`加密；

`Encrypt`加密将目标文本转换成具有不同长度的、可逆的密文；`Hash`加密将目标文本转换成具有相同长度的、不可逆的杂凑字符串；

在应用程序中使用哪一种加密方式取决于业务需求，基本原则如下：如果被保护数据仅仅用作比较验证，在以后不需要还原成明文形式，则使用`Hash`加密；如果被保护数据在以后需要被还原成明文，则需要使用`Encrypt`加密。

官方 `API` 地址 

- [https://laravel.com/api/5.4/Illuminate/Encryption.html](https://laravel.com/api/5.4/Illuminate/Encryption.html)
- [https://laravel.com/api/5.4/Illuminate/Hashing.html](https://laravel.com/api/5.4/Illuminate/Hashing.html)

# 使用
## `Encrypt`加密
### 创建实例
1. 最方便的方式，从服务容器取`Encrypt`加密对象；`Encrypt`加密对象的`key`密钥及`cipher`加密算法参数均读取`config/app.php`配置文件中的`key`及`cipher`值；

  ```
    $encrypter = app('encrypter');
  ```
2. 手动创建；`Encrypt`加密构造函数的第一个参数是密钥，第二个参数是加密算法类型，默认是'AES-128-CBC'；密钥的长度与加密算法类型有关联，'AES-128-CBC'算法的密钥长度是16位，'AES-256-CBC'算法的密钥长度是32位；

  ```
    $encrypter = new Illuminate\Encryption\Encrypter(random_bytes(16));
    $encrypter = new Illuminate\Encryption\Encrypter(random_bytes(16), 'AES-128-CBC');
    $encrypter = new Illuminate\Encryption\Encrypter(random_bytes(32), 'AES-256-CBC');
  ```

 
### 加密
调用`Encrypt`加密实例的`encrypt`方法对指定对象进行加密；`encrypt`方法有两个参数，第一个参数是加密的目标对象，第二个参数是布尔值，表示是否对第一个参数进行序列化操作；
```
$encrypter->encrypt('foo');
```
如果是加密字符串类型的对象，可以调用`encryptString`方法；`encryptString`方法只有一个参数，即字符串对象；
```
$encrypter->encryptString('foo');
```

### 解密
调用`Encrypt`加密实例的`decrypt`方法对指定对象进行解密；`decrypt`方法有两个参数，第一个参数是解密的目标对象，第二个参数是布尔值，表示是否对第一个参数进行序列化操作；
```
$encrypter->decrypt($encrypter->encrypt('foo'));
```
与`encryptString`方法对应的是`decryptString`方法；
```
$encrypter->decryptString($encrypter->encryptString('foo'));
```


## `Hash`加密
### 创建实例
1. 最方便的方式，从服务容器取`Hash`加密对象；

 ```
    $hasher = app('hash');
 ```
2. 手动创建；

 ```
    $hasher = new \Illuminate\Hashing\BcryptHasher;
 ```
### 加密
调用`Hash`加密对象的`make`方法；
```
    $hasher->make('password');
```
也可以调用全局帮助函数`bcrypt`
```
    bcrypt('password');
```

### 较验
因为哈希加密是不可逆的，所以要想判断是否值相等，可以调用`check`方法；`check`方法的本质是将传入值也进行哈希加密，判断加密后的字符串是否相同；
```
$hasher->check('password', $hasher->make('password'));
```
