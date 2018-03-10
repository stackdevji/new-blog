---
title: 将Chrome Clockwork集成到你的Web-app中
date: 2017-06-22 16:32:05
tags:
	- php
	- 工具
	- clockwork
---

<img src="https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=3809377190,229980002&fm=27&gp=0.jpg" width="60%" height="40%" />
<!--more-->

原文地址：[Integrating Chrome’s Clockwork into your web-app](http://hotcashew.com/2013/12/integrating-chromes-clockwork-custom-web-app/)。本文对原文进行翻译，总结如何手动集成Clockwork。
Clockwork它能够与任何服务端平台、框架一起工作。下面我们将阐述如何将它集成到我们的php-app上。

### 安装

通过composer进行安装:
1. 添加“itsgoingd/clockwork”: “dev-master” 到你的composer.json文件中。[eg](https://packagist.org/packages/itsgoingd/clockwork)。
2. 执行composer update。

或者你也可以通过clone [https://github.com/itsgoingd/clockwork/](https://github.com/itsgoingd/clockwork/),可能需要翻墙。然后使用传统的加载库的方法来加载它。

### 集成Clockwork

它的最好的放置位置是在你的应用程序的根目录中，在控制器、路由的周围。header()命令需要在发送其他信息之前运行，它将告诉Chrome的Clockwork标签，从服务器请求什么文件来获取统计信息。

最后两行的 resolveRequest()和storeRequest() 需要在web应用程序完成所有伟大的工作之后运行。reslove()函数将在每个提供的DataSoure上被调用。这将解析并保存您的计时、日志、查询等。

```php
// 这将发送header,除非你是缓冲输出
// 这个应该放在程序开始的部分
$clockwork = new Clockwork();
header("X-Clockwork-Id: " . $clockwork->getRequest()->id);
header("X-Clockwork-Version: " . Clockwork::VERSION);
 
// spawn your own custom DataSource (explained below) 生成你自定义的DataSource
$clockwork->addDataSource(new YourCustomDataSource($yourAppContext));
 
// attach a sample datasource that comes with  附加一个示例数据源
// the Clockwork library (grabs session info, etc)
$clockwork->addDataSource(new PhpDataSource());
 
// we could write a custom APC, MemCached, etc here
// or just use the FileStorage that comes with Clockwork
$clockwork->setStorage(new FileStorage("some/path/"));
 
// run your web-app here
// ...
 
// once your app is done, tell Clockwork to resolve and store
// data in a file on your server; it will call the resolve()
// method on YourCustomDataSource and PhpDataSource
$clockwork->resolveRequest();
$clockwork->storeRequest();
```
之后会创建一个json文件(文件名字貌似是：x-Clockwork-id)到上面提到的  “some/path/”中，这个路径由我们自己创建。我项目中设置的路径是/data/data/clockwork。

```php
define('CLOCKWORK_LOG_PATH', '/data/data/clockwork');
if ( ! is_dir(CLOCKWORK_LOG_PATH)) {
	@mkdir(CLOCKWORK_LOG_PATH, 0777);
}
```

在页面加载后，Chrome的Clockwork扩展会收集X-Clockwork头信息，并向服务器发送一个新的请求，使用x-Clockwork-id来获取生成的文件。例如：如果X-Clockwork-Id 是1387208177.8923.1394938488的话，那么Chrome’s Clockwork 扩展将会发送一个请求到你的服务器上，他的url是： /__clockwork/1387208177.8923.1394938488。

可以通过监听“/ __clockwork /[*:id]”来处理这一请求，并进行如下操作:

```php
$storage = new FileStorage("/some/path/");
$data = $storage->retrieve($ctx->parameters->id);
$data->toJson();
```
像[Klein](https://github.com/klein/klein.php)或[AltoRouter](https://github.com/dannyvankooten/AltoRouter)这样的路由器可以采用上面的语法并返回请求id。或者您可以手动解析$ _SERVER[' REQUEST_URI ']来获取id。文件存储为您做剩下的工作

### 生成自定义DataSoure

教程上的代码如下：

```php
class YourCustomDataSource extends DataSource
{
	protected $context;
 
	/**
	 * YourAppContext should link to your main web-app
         * and fetch timings, queries, and logs
	 */
	function __construct(YourAppContext $context)
	{
		$this->context = $context;
	}
 
	/**
	 * the entry-point. called by Clockwork itself.
	 */
	function resolve(Request $request)
	{
		$timings = $this->context->getTimings();
 
		// optionally: pre-sort the timeline
		uasort($timeline, function($a, $b) {
			if($a['start'] > $b['start'])
				return 1;
 
			if($a['start'] == $b['start']) {
				if($a['end'] > $b['end'])
					return 1;
				elseif ($a['end'] < $b['end'])
					return -1;
 
				return 0;
			}
 
			return -1;
		});
 
		$queries = $this->context->getQueries();
 
		$request->timelineData = $timeline;
		$request->databaseQueries = $queries;
 
		return $request;
	}
}
```

我们项目中的代码：

```php
<?php
/**
 * ClockWork.php
 *
 * User: longfei.he 这是我大哥
 * Date: 2017/6/13
 * Time: 下午8:54
 */
class CollectData extends \Clockwork\DataSource\DataSource
{
	public static $sqls = [];
	public static $uri = '';
	public static $method = '';
	public static $controller = '';
	public static $getData = [];
	public static $postData = [];
	public static $responseTime = 0;
	public static $logs = [];
	public static $startTime = 0;
	public static $responseStatus = 200;
	public static $headers;
	public function resolve(\Clockwork\Request\Request $request)
	{
		$request->databaseQueries = self::$sqls;
		$request->uri = self::$uri;
		$request->method = self::$method;
		$request->time = self::$startTime;
		$request->responseStatus = self::$responseStatus;
		$request->responseTime = self::$responseTime;
		$request->controller = self::$controller;
		$request->log = self::$logs;
		$request->getData = self::$getData;
		$request->postData = self::$postData;
		$request->headers = self::$headers;
		return $request;
	}
}
```


