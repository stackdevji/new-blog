---
title: FILES数据ERROR
date: 2018-01-19 15:24:33
tags:
	- 工作笔记
	- php
---

>今天遇到php上传文件到FILES数组中失败，对其中的error错误码进行分析。
<!--more-->

### FILES结构
php接收上传文件时会将文件信息存放在文件上传变量中。也就是$__FILES数组，他的结构如下
```php
Array
(
	[file1] => Array(
	   [name] => MyFile.jpg // 文件在上传者机器上的地址
           [type] => image/jpeg
           [tmp_name] => /tmp/php/php6hst32 // 上传的服务器上文件保存的临时地址
           [error] => 0
           [size] => 98174
	)
)
```

### error对应的值

其值为 0，没有错误发生，文件上传成功。 
其值为 1，上传的文件超过了 php.ini 中 upload_max_filesize 选项限制的值。 
其值为 2，上传文件的大小超过了 HTML 表单中 MAX_FILE_SIZE 选项指定的值。 
其值为 3，文件只有部分被上传。 
其值为 4，没有文件被上传。 
其值为 6，找不到临时文件夹。PHP 4.3.10 和 PHP 5.0.3 引进。 
其值为 7，文件写入失败。PHP 5.1.0 引进。 
