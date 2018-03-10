---
title: PHP运行模式
date: 2017-06-15 18:20:29
tags:
	- php
	- 运行原理
---

了解php运行原理之前先认识下php的运行模式。
<img src="http://blog.zhangruipeng.me/hexo-theme-hueman/gallery/math.jpg" width="30%" height="25%" />

<!--more-->

## PHP的5中常见的运行模式

1. **cgi**
2. **fastcgi**
3. **cli**
4. **web模块模式**
5. **isapi**

> 备注：php5.3以后，不再有isapi模式，安装php之后也不会再有php5isapi.dll这个文件了。

### 一：cgi

cgi即通用网关接口（common gateway interface）,它是用来进行客户端（浏览器）和web服务器之间用来交互的程序。
这么说的话挺让人费解，是的我开始就没有理解cgi到底是用来干什么的？好的，我们先了解下从 客户端->web服务器->cgi
之间的数据是怎么传输的。

#### 什么是cgi

客户端点击某个操作之后向服务器发出请求，web服务器接收到这个请求要干什么？它先看自己能不能解析这些文件，如果是
能够自己解析的文件比如html文件，那么服务器会去查找到这个文件并返回给客户端。要是不能解析的话，那么服务器就会为
解析这个请求开启一个cgi程序，然后web服务器将客户端发来请求的数据交给这个cgi程序，OK,这个时候体现出cgi的作用了，
这个对应的cgi程序是要保证web服务器传过来数据要遵循某个标准的，就是说，你服务器要让我为你对应解析相应文件，那么你
给我传过来的数据就要符合对应解析这个文件的标准。这就是通过网关接口的其中一个作用：我们编写的程序和web服务器之间接口的标准。

标准？什么标准？好的，再举个例子：比如我们编写的程序是用php来写的，而且保存在某个php脚本中（php文件）要知道nginx解析不了php文件，那就为其开启一个cgi程序（php-cgi）用来解析php脚本。好了，解析器有了，那我要开始通过解析器来处理客户端发过来的请求了吧。那web服务器传给php-cgi的数据得遵循某些标准：url得有吧，post数据得有吧，header也不能少吧。

之后就是cgi程序处理完请求之后，（这其中可能使用外部程序，比如数据库）然后把请求的结果返回个web服务器，web服务器在把结果响应给客户端。我理解就是这个一个过程。

#### cgi工作原理

以上我们大概了解web服务器每处理一个需要cgi程序的请求就会开启一个cgi程序，也就是每次都fork一个进程，在cgi执行之前，web服务器要为cgi设置环境变量，初始化执行环境，比如php-cgi每次执行之前都会解析php.ini文件。每个被激活的cgi程序都有自己唯一的一组环境变量。事实上对于一个非常繁忙的Web服务器，可能同时会有同一个cgi程序的很多个进程在运行，这时每个cgi程序的进程都有自己的运行环境，互不影响。当一个cgi执行成功之后这些环境变量也会对应销毁。那么问题来了，每次fork一个进程都会去进行初始化环境。那么当web服务器的请求量比较大的时候，这个的模式是相当占用内存和cpu时间的，这时性能就会有问题了。

### 二：fastcgi

cgi（Fork-And-Execute模式）比较让人诟病,所以fastcgi像是cgi的升级版本，我私下里把fastcgi也理解成cgi的一种，但是显然不太妥当，我觉得fastcgi更像是一种协议。为什么这么说呢，百度百科说它像是一个常驻型的cgi，主要是因为，一旦服务器开启就会载入fastcgi，fastcgi相比于cgi不会每次来发来一个请求就去fork一个cgi进程，而是常驻存在，在开启的时候只初始化一次环境，（如只解析一次php.ini），然后他会开启多个work进程，如果请求发来，它就会对应的将相应的数据发给一个work进程（如php-cgi程序）然后进行处理请求。并返回结果给服务器。而且当work进程不够用的时候，fastcgi会预先开启出来一部分work进程，当空闲的时候，又会关掉一些多余的work进程，php手册上把fastcgi有叫做进程管理器，我觉得再合适不过了。

#### fastcgi工作原理 摘自百度百科

1. Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module).
2. FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
3. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
4. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。

#### fastcgi缺点

因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。

我操，这有个问题：CGI多线程 那么，cgi模式下，启动的cgi程序到底是进程还是线程？

#### php-fpm

PHP-FPM是一个PHPFastCGI管理器，是只用于PHP的。是一个实现了Fastcgi的程序，被PHP官方收了

>大家都知道，PHP的解释器是php-cgi。php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理（所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。好了PHP-FPM也是这么个东西，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨PHP-FPM稳定性太差的），也越来越流行。

网上有的说，fastcgi是一个协议，php-fpm实现了这个协议
>对

有的说，php-fpm是fastcgi进程的管理器，用来管理fastcgi进程的
>不太对，首先fastcgi就是一个进程管理器，它本身不是一个进程，而php-fpm管理的对象是php-cgi

所以总的来说 php-fpm是
>php-fpm实现了fastcgi，它用来管理php-cgi

注：
>对于PHP 5.3.3之前的php来说，是一个补丁包。旨在将FastCGI进程管理整合进PHP包中。如果你使用的是PHP5.3.3之前的PHP的话，就必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。从PHP 5.4 RC2开始，php-fpm已经转正了，不再被php团队标注为EXPERIMENTAL（实验性的东西）相对Spawn-FCGI，PHP-FPM在CPU和内存方面的控制都更胜一筹，而且前者很容易崩溃，必须用crontab进行监控，而PHP-FPM则没有这种烦恼，它可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多优点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。

#### fpm的相关命令

```
php-fpm{start|stop|quit|restart|reload|logrotate}
--start 启动php的fastcgi进程
--stop 强制终止php的fastcgi进程
--quit 平滑终止php的fastcgi进程
--restart 重启php的fastcgi进程
--reload 重新平滑加载php的php.ini
--logrotate 重新启用log文件
```

### 三：cli

附上原文:

>What is PHP CLI? PHP CLI is a short for PHP Command Line Interface. As the name implies, this is a way of using PHP in the system command line. Or by other words it is a way of running PHP Scripts that aren't on a web server (such as Apache web server or Microsoft IIS). People usually treat PHP as web development, server side tool. However, PHP CLI applies all advantages of PHP to shell scripting allowing to create either service side supporting scripts or system application even with GUI!
PHP CLI is available on all popular operating systems: Linux, Windows, OSX, Solaris. Popular Linux distributions (such as Ubuntu, Debian, Fedora Core, Suse and etc.) allow to install PHP CLI from package manager (e.g. Synaptic or similar) with couple of mouse clicks. This makes installation hassle free and you can start using it within a seconds!
PHP CLI SAPI was first released in PHP 4.2.0 as experimental, but as of version PHP 4.3.0 (including PHP5), it is fully supported and enabled by default. PHP CLI is just a new SAPI type (Server Application Programming Interface) that focused on developing shell (or desktop as well) applications with PHP. It's worth mentioning that PHP CLI and PHP CGI are different SAPI's although they do share many of the same behaviours.
If you have standard installation of PHP for Apache web server, then there are very high chances that you already have PHP CLI installed on your system. You chances are even higher if your system is running Linux. If you unlucky enough not to have it buy default, then you need to recompile your PHP with the --enable-cli flag or reinstall from the package that does have it. If you are running Windows, then you probably need to add php executable to your system path.

PHP CLI 是PHP Command Line Interface的简称，它是一种通过命令行运行php的一种方式。换句话说，它是一种区别于Web Server（例如Apache Web Server 或者 Microsoft IIS）的能够运行php脚本的方式。人们通常认为php是用来进行web开发的，然而，PHP CLI提供了所有的PHP的优势。PHP shell脚本允许创建要么支持脚本，要么支持应用程序，甚至是支持GUI的服务端。

PHP CLI 在PHP 4.2.0以前还是实验性的，但是在4.3.0以后默认的已经转正了，它是一种新类型的CLI SAPI(Server Application Programming Interface)，用来通过php专注于开发shell脚本。值得一提的是，PHP CLI和PHP CGI是不同的SAPI尽管他们之间拥有很多共同的行为。

#### 为什么要使用PHP CLI

因为他在通过命令行运行php代码有几个优势：
1. 不需要了解其他语言，比如perl、bash、awk。
2. 运行用php写的调度任务
3. 使GUI应用程序中使用PHP和GTK
4. 重复使用现有的组件
5. 使用的是多线程功能，能够编写非常健壮的脚本
6. 使用PHP访问系统STDIN、STDOUT STERR

### 四：模块模式（web模块） [COPY](http://www.cnblogs.com/xia520pi/p/3914964.html)

模块模式是以mod_php5模块的形式集成，此时mod_php5模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求，然后将处理后的结果返回给Apache。如果我们在Apache启动前在其配置文件中配置好了PHP模块（mod_php5）， PHP模块通过注册apache2的ap_hook_post_config挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。

除了这种启动时的加载方式，Apache的模块可以在运行的时候动态装载，这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。我们所需要做的仅仅是给服务器发送信号HUP或者AP_SIG_GRACEFUL通知服务器重新载入模块。但是在动态加载之前，我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。 Apache中对动态链接库的处理是通过模块mod_so来完成的，因此mod_so模块不能被动态加载，它只能被静态编译进Apache的核心。这意味着它是随着Apache一起启动的。

Apache是如何加载模块的呢？我们以前面提到的mod_php5模块为例。首先我们需要在Apache的配置文件httpd.conf中添加一行：

```
LoadModule php5_module modules/mod_php5.so
```
这里我们使用了LoadModule命令，该命令的第一个参数是模块的名称，名称可以在模块实现的源码中找到。第二个选项是该模块所处的路径。如果需要在服务器运行时加载模块，可以通过发送信号HUP或者AP_SIG_GRACEFUL给服务器，一旦接受到该信号，Apache将重新装载模块，而不需要重新启动服务器。

该运行模式是我们以前在windows环境下使用apache服务器经常使用的，而在模块化（DLL）中，PHP是与Web服务器一起启动并运行的。（它是apache在CGI的基础上进行的一种扩展，加快PHP的运行效率）。



### 五：isapi [COPY](http://www.cnblogs.com/xia520pi/p/3914964.html)

ISAPI（Internet Server Application Program Interface）是微软提供的一套面向Internet服务的API接口，一个ISAPI的DLL，可以在被用户请求激活后长驻内存，等待用户的另一个请求，还可以在一个DLL里设置多个用户请求处理函数，此外，ISAPI的DLL应用程序和WWW服务器处于同一个进程中，效率要显著高于CGI。（由于微软的排他性，只能运行于windows环境）

PHP作为Apache模块，Apache服务器在系统启动后，预先生成多个进程副本驻留在内存中，一旦有请求出现，就立即使用这些空余的子进程进行处理，这样就不存在生成子进程造成的延迟了。这些服务器副本在处理完一次HTTP请求之后并不立即退出，而是停留在计算机中等待下次请求。对于客户浏览器的请求反应更快，性能较高。