---
title: Chrome的调试利器Clockwork(二)
date: 2017-06-23 15:41:36
tags:
	- clockwork
---

<img src="http://blog.zhangruipeng.me/hexo-theme-hueman/gallery/salt-lake.jpg" width="60%" height="40%" />
本文简要介绍了clockwork结构体系结构，在实现对新框架的支持时非常有用。
<!--more-->

### Chrome扩展

Chrome扩展检查了X-Clock-Version和X-Clockwork-Id头的HTTP响应。X-Clock-Version包含了Clockwork服务器端组件的版本，它允许在数据协议更改时保持与旧版本兼容的扩展。当在响应头中发现X-Clock-Version时，HTTP GET ajax请求被制成/__clockwork/{id}，其中{id}是X-Clockwork-Id头的值。请求的JSON的数据以指定的id(输出 Clockwork\Request\Request::toJson)返回，并在扩展中显示。

### Server-side library（服务端库）

服务器端库提供了尽可能简单的为各种框架和应用程序添加支持。它由几个组件组成:
1. DataSource - 收集数据
2. Request - 表示收集的数据
3. Storage - 存储和检索请求
4. Support - 特定于框架的支持文件，如服务提供者，中间件等等

典型的用法如下:
1. Clockwork类被实例化，自动创建新的Request\Request实例
2. 通过$clockwork->addDataSource方法添加数据源
3. 通过$clockwork->setStorage设置storage实例
4. 应用程序运行。。。
5. $clockwork->resolveRequest被调用 - Request实例通过每个DataSource的reslove()方法传递，每个DataSource将相关数据添加到请求实例中
6. $clockwork->storeRequest被调用 - Request实例通过store()方法存储在Storage实例上
7. Clockwork::VERSION的值作为X-Clockwork-Version header被发送，$clockwork->getRequest()->id的值作为X-Clockwork-Id header被发送

当/__clockwork/{id}被请求的时候
1. Storage实例被创建
2. request通过$storage->retrieve($id)被检索
3. JSON格式的数据通过$request->toJson()方法被输出

收集的数据被存储在Request的对象中，仅仅通过公有属性被赋值的方法。在Request类中没有进行验证,DataSource只负责赋值而已。

可用的赋值：
1. id - 唯一的请求id，不应该手动设置，当创建新的请求实例时，由当前时间和随机数自动生成
2. time - 请求时间，期望在微秒内数值
3. method - 使用HTTP方法，期望字符串值
4. uri - 请求的uri，期望是字符串的值
5. headers - 请求的头，期望的值是数组形式，格式如下：
```php
[
   'string value of header name' => array('string value of header value', ...),
    ...
]
```
6. controller - 在MVC框架中使用的控制器名称，期望字符串值
7. getData - request GET数据，期望GET数据数组
8. postData - request POST数据，期望POST数据数组
9. sessionData - session数据，期望是session数组
10. cookies - 请求的cookies,期望是cookies数组
11. responseTime - 期望在发送响应时的微秒内数值，然后计算响应时间，以响应时间
12. responseStatus - 响应HTTP状态码，期望数值
13. databaseQueries - 数据库查询数据，包括durations，期望的数据格式如下：
```php
[
    [
        'query'   => 'string value of first sql query in executable form',
        'duration' => 'numeric value in miliseconds'
    ],
    ...
    [
    	...
    ]
]
```
14. timelineData - 时间轴选项卡的数据，期望数组形式为:
```php
[
    [
        'start' => 'numeric representation of events start time in microseconds',
        'end' => 'numeric representation of events end time in microseconds',
        'duration' => 'numeric representation of events duration in milliseconds',
        'description' => 'string value describing the event'
    ],
    ...
    [
    	...
    ]
]
```
15. 有一个用于创建时间轴数据的助手对象 -  Request\Timeline
16. log - 应用程序日志，期望数组形式为:
```php
[
    [
        'time' => 'numeric value of timestamp in seconds',
        'level' => 'numeric constant representing message type',
        'message' => 'string value of message'
    ],
    ...
    [
    	...
    ]

]
```
17. 可用的错误日志等级：1 - DEBUG, 2 - INFO, 3 - NOTICE, 4 - WARNING, 5 - ERROR
18. 有一个用于创建日志数据的对象 - Request\Log
19. routes - 应用程序的路由，期望数组的形式为：
```php
[
    [
        'uri' => 'string value of routes uri',
        'name' => 'string value of routes name',
        'action' => 'string value of routes callback',
        'before' => 'string value of route's before filters',
        'after' => 'string value of route's after filter'
    ],
    ...
    ...
    [
    	...
    ]
]
```

所有属性只能包含可以编码为JSON的数据(没有资源、对象实例和闭包)，因为这是用于存储和传输数据到扩展的格式。
注意，扩展不要求您使用这个库，您可以为您提供返回正确JSON数据的自定义实现，但是推荐使用该库提供一些稳定的API，允许数据格式进行更改。

正确的JSON响应示例:
```json
{
    "id":"1379711612.4843.251523074",
    "time":1379711611.8899,
    "method":"GET",
    "uri":"\/users",
    "headers": {
        "host":["base.daylight.home"],
        "x-real-ip":["10.10.10.103"],
        "connection":["close"],
        "accept":["*\/*"],
        "user-agent":["Mozilla\/5.0 (Macintosh; Intel Mac OS X 10_8_5) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/31.0.1637.0 Safari\/537.36"],
        "accept-encoding":["gzip,deflate,sdch"],
        "accept-language":["en-US,en;q=0.8,cs;q=0.6,sk;q=0.4"],
        "cookie":["laravel_session=s566k2i47igcam0v6nn8i0f4h5"]
    },
    "controller":"UsersController@index",
    "getData":[],
    "postData":[],
    "sessionData": {
        "_sf2_attributes": {
            "_token":"ZI3OJ0EliZieVvp9ol3hveu11NqMjSr6MXLe3tpF"
        }
    },
    "cookies": {
        "laravel_session":"s566k2i47igcam0v6nn8i0f4h5"
    },
    "responseTime":1379711612.5321,
    "responseStatus":200,
    "responseDuration":642.18783378601,
    "databaseQueries": [
        {
            "query":"SELECT count(*) as aggregate FROM `languages` WHERE `id` = 'en'",
            "duration":0.75
        },
        {
            "query":"SELECT * FROM `languages`",
            "duration":0.52
        }
    ],
    "databaseDuration":1.27,
    "timelineData": {
        "total": {
            "start":1379711611.8899,
            "end":1379711612.5323,
            "duration":642.45295524597,
            "description":"Total execution time."
        },
        "initialisation": {
            "start":1379711611.8899,
            "end":1379711612.4734,
            "duration":583.5108757019,
            "description":"Application initialisation."
        },
        "run": {
            "start":1379711612.4734,
            "end":1379711612.5323,
            "duration":58.935880661011,
            "description":"Framework running."
        },
        "boot": {
            "start":1379711612.4734,
            "end":1379711612.4743,
            "duration":0.86092948913574,
            "description":"Framework booting."
        },
        "dispatch": {
            "start":1379711612.4833,
            "end":1379711612.5318,
            "duration":48.506021499634,
            "description":"Router dispatch."
        }
    },
    "log":[],
    "routes":null,
    "userData":null
}
```

### 完结

原文地址:[https://github.com/itsgoingd/clockwork/wiki/Development-notes](https://github.com/itsgoingd/clockwork/wiki/Development-notes),本文对原文进行翻译，快翻译吐了。

