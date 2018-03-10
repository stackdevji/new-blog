---
title: Chrome的调试利器Clockwork
date: 2017-06-22 16:31:27
tags:
	- php
	- 工具
	- clockwork
---
<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510989803956&di=48ff944ac778ee661307f50d7906d8ae&imgtype=0&src=http%3A%2F%2Fimg.ivsky.com%2Fimg%2Ftupian%2Fpre%2F201208%2F13%2Fchengde_bishu_shanzhuang-034.jpg" width="60%" height="40%" />

Clockwork是PHP开发的一个Chrome的扩展工具，它提供了一个新的面板，提供了用于调试和剖析PHP应用程序的各种信息，包括关于request、header、get和post数据、cookie、session数据、数据库查询（database query）、路由和可视化应用程序运行时的信息。
<!--more-->

Clockwork存储库包含的服务器端组件，会收集所有的数据，并以json格式存储他们，并为他们提供在Chrome开发工具扩展中显示。

### 好处

安装Clockwork之后对我们程序运行的各种信息的可视化程度大大提高，我们可以看到页面加载之后调用接口的耗时时间、头信息等等。最方便的是方便我们查看程序的运行日志，和sql语句，可以很方便的帮助我们调试程序。

Clockwork提供了对Laravel、Slim 2和CodeIgniter 2.1框架的支持，您可以通过一个可扩展的API为任何其他或自定义框架添加支持。

### Laravel

安装Clockwork之后需要在config/app.php文件下注册Laravel服务提供者

```php
'providers' => [
	...
	Clockwork\Support\Laravel\ClockworkServiceProvider::class,
]
```
当你使用Laravel 5的时候，你需要在你的app/Http/Kernel.php文件中添加Clockwork中间件

```php
protected $middleware = [
	\Clockwork\Support\Laravel\ClockworkMiddleware::class,
	...
]
```
默认情况下，Clockwork只能在调试模式下被调用，您可以在配置文件中更改这个和其他设置。使用下面的Artisan命令将配置文件发布到配置目录中:

```php
$ php artisan vendor:publish --provider='Clockwork\Support\Laravel\ClockworkServiceProvider'
```

对于Laravel 4 你可以执行下面这个命令：

```php
$ php artisan config:publish itsgoingd/clockwork --path vendor/itsgoingd/clockwork/Clockwork/Support/Laravel/config/
```

Clockwork 提供了一个clock()辅助函数，它提供了一个比较简单的方法去记录Clockwork日志和事件.

```php
clock()->startEvent('event_name', 'Event description.'); // event called 'Event description.' appears in Clockwork timeline tab

clock('Message text.'); // 'Message text.' appears in Clockwork log tab
logger('Message text.'); // 'Message text.' appears in Clockwork log tab as well as application log file

clock(['hello' => 'world']); // logs json representation of the array
clock(new Object()); // logs string representation of the objects if the object implements __toString magic method, logs json representation of output of toArray method if the object implements it, if neither is the case, logs json representation of the object cast to array

clock()->endEvent('event_name');

```

如果你比较喜欢使用Facades,你需要将下面这句话添加到 app/config/app.php文件中：

```php
'aliases' => [
	...
	'Clockwork' => Clockwork\Support\Laravel\Facade::class,
]
```

### Lumen

一旦安装了Clockwork，您需要在bootstrap/app.php中注册Clockwork服务提供程序:

```php
$app->register(Clockwork\Support\Lumen\ClockworkServiceProvider::class);
```

你也可以添加Clockwork到上面提到的相同文件中：

```php
$app->middleware([
	...
	Clockwork\Support\Lumen\ClockworkMiddleware::class,
]);
```

### 其他框架

其他框架还支持 Slim 2 和 CodeIgniter 2.1，此文就不对其进行赘述，请参考[文档](https://github.com/itsgoingd/clockwork)
对于其他框架这里没有提及的可以进行手动集成Clockwork,请参考[将Chrome Clockwork集成到你的Web-app中](https://stackdevji.github.io/2017/06/22/Integer-clockwork/)

最后需要在Chrome中安装clockwork的扩展程序，可以在谷歌商店中进行下载，需要翻墙。
