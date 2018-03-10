---
title: Laravel 踩坑大集合
date: 2017-05-25 09:47:49
tags:
	- Laravel
	- php
---
<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510989934064&di=6673a22d4b54599073a05e795351795a&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fimgad%2Fpic%2Fitem%2F0df3d7ca7bcb0a466b331e276163f6246b60af9f.jpg" width="60%" height="40%" />
Laravel是一套简洁、优雅的PHP、Web开发框架。它可以让你从面条一样杂乱的代码中解脱出来；它可以帮你构建一个完美的网络APP，而且每行代码都可以简洁、富于表达力。公司的大多数php项目都是基于laravel框架的，而作为一个初学laravel的人，会有一堆坑等着我踩，到网上有搜不到满意的答案，所以每踩一次坑。就再次博文记录一笔。

<!--more-->

### 执行php artisan list 报错

1. **报错提示**
```php
[Symfony\Component\Debug\Exception\FatalThrowableError]
Class 'Memcached' not found
```
2. **解决办法**
> 将根目录下的 .env.exampel文件复制一份并命名为.env文件（或者重命名），提交的时候我们一般提交 .env.example文件，因为每个人可能在不同的环境需要配置也不同，这些配置我们都写到.env里面。
> 如果是刚刚git clone下来的项目，需要执行composer install，将php使用的packagist安装完毕，php所使用的composer扩展存放在vendor目录下。

### TODO
