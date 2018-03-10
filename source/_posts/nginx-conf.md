---
title: 部署后台环境中遇到的nginx配置问题
date: 2016-12-01 17:15:39
tags:
    - nginx
    - 学习笔记
---
<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510990107790&di=b499cf794a1a4cc6dc2eb2598442ebde&imgtype=0&src=http%3A%2F%2Fimg2.lvyou114.com%2Fsmall%2F2014%2F0805%2F14072340054177.jpg" />
其实，我在工作中很抵触nginx环境，实习的时候nginx环境配置的一塌糊涂。就在前几天配置nginx环境也还是晕头转向的，今天要把公司的一个项目部署起来，想部署其实很简单，就是clone下代码，然后配置下nginx环境就可以了，但是我是真的很抵触nginx,说白了就是理解不透彻，所以今天彻底的学习一下。在此记录下。
<!-- more -->

## nginx.conf模块
前提是本地要部署好LNMP环境，然后部署项目的时候我们要使用本地的nginx虚拟服务器来代理我们的项目，这就要用到nginx的配置文件了，废话不多说，直接来看nginx.conf的几个模块。

### 顶级配置

```
#nginx的用户和用户组
user  nobody;
#nginx的工作进程数 一般为电脑的cpu核数。
worker_processes  1;

#不同级别的错误日志的位置，后面是对应的级别
#级别有：debug、info、notice、warn、error、crit
error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;
#进程文件
pid        logs/nginx.pid;
```
### Event模块
```
events {
    #单个工作进程的最大并发连接数
    #总并发连接数等于 worker_processes * worker_connections
    worker_connections  1024;

    #告诉nginx收到一个新连接通知后接受尽可能多的连接
    multi_accept on;

    #设置用于复用客户端线程的轮询方法。如果你使用Linux 2.6+，
    #你应该使用epoll。如果你使用*BSD，你应该使用kqueue。
    use epoll;
}
```
### HTTP模块
```
http {
    #文件扩展名与文件映射表
    include       mime.types;

    #默认文件类型
    default_type  application/octet-stream;

    #设置日志格式，知道是干什么用的就行
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #定义访问日志，设置为 off 可以关闭日志，提高性能
    #access_log  logs/access.log  main;

    #开启高效文件传输模式，sendfile 指令指定 Nginx 是否调用sendfile
    #函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘
    #IO 重负载应用，可设置为 off，以平衡磁盘与网络 I/O 处理速度，降
    #低系统的负载。
    sendfile        on;
    #tcp_nopush     on;

    #连接超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;

    #开启 gzip 压缩
    #gzip  on;
}
```
### Server模块
```
server {
        #监听端口
        listen       8080;
        #访问项目的域名 多个域名空格分开
        server_name  local.bool.com;
        #指向部署项目的入口文件的路径
        root   /usr/local/var/www/matrix-book/www;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #默认请求
        location / {
            #定义首页访问的索引文件
            #index  index.html index.htm index.php;
            try_files $uri /index.php?$args;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #错误提示页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #php脚本的访问通过 fastCGI 进程管理器 在9000端口上处理请求
        location ~ \.php$ {
            #项目入口
            root           /usr/local/var/www/matrix-book/www
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            #下面这个参数是指定脚本名称的路径 默认值是fastcgi_script_name
            #我刚开始出现的错误是访问指定的项目不是运行index.php文件而是直接下载
            #index.php文件，所以要加上固定的路径或者$document_root/ 我觉得太麻烦
            #就直接include 了fastcgi.conf文件。
            #fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
            #include        fastcgi_params;
            include        fastcgi.conf;
            access_log  /usr/local/etc/nginx/logs/local.book.com-access.log ;
            error_log   /usr/local/etc/nginx/logs/local.book.com-error.log;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        # 禁止访问的.ht***文件
        #location ~ /\.ht {
        #    deny  all;
        #}
}
```
现在对nginx的配置文件不同模块总算有了一定的理解，基本上配置nginx文件只用到server模块，所以明白每个参数都是什么意思，就不是那么抵触了。

## nginx启动关闭的几个命令
1. **通过配置文件启动nignx**
```
sudo nginx -c /usr/local/etc/nginx/nginx.conf
```
2. **直接找到nginx的启动文件，通过./启动文件名 启动**
```
cd /usr/local/Cellar/nginx/1.10.1.2/bin/
sudo ./nginx
```
3. ** 关闭nginx**
```
sudo nginx -s stop
```
4. **重启nginx**
```
sudo nginx -s reload
```
当然修改完配置文件我们可以检查下配置文件是否修改正确，然后重新启动nginx

```
nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```
## fastcgi.conf 和 fastcgi_params的区别
为什么要说以下两者的区别呢？因为我配置环境的时候很大的一个坑就是这里，首先nginxnginx本身不能处理PHP，它只是个web服务器，当接收到请求后，如果是php请求，则发给php解释器处理，并把结果返回给客户端。

而nginx一般是把请求发fastcgi管理进程处理，fascgi管理进程选择cgi子进程处理结果并返回被nginx，所以在配置nginx文件的时候一般会用到fastcgi.params文件。而这个文件里面其实是没有全路径的SCRIPT_FILENAME，所以我们要把我们配置文件的入口的全路径写死放到$fastcgi_script_name
前面，否则会报找不到文件的错误，可以参考[杰大屌的博客 LNMP安装](http://bettercuicui.github.io/2016/04/28/php/%E5%AE%89%E8%A3%85lnmp%E7%8E%AF%E5%A2%83/)。

而我出现的问题是访问页面的时候不是运行项目的入口文件，而是直接下载这个index.php这个文件
这是因为 一般nginx默认配置中会是这个样子的。这里有一个SCRIPT_FILENAME变量，但是fastcgi_params这个文件中是不包含该变量的，该变量的定义实际上是在fastcgi.conf文件中。

**那两者的区别是什么？**
在很久之前向我们配置nginx文件的时候是这样的

```
#之前都是这样包含 fastcgi_params 文件的
include        fastcgi_params;
#然后在后面加上：
fastcgi_param  SCRIPT_FILENAME  /usr/local/var/www/matrix-book/www/$fastcgi_script_name;```
因为这个指令它是数组形态的,并不会说,同名的指令,后面会替换掉前面的。而nginx的开发者慢慢发现大家写死这个root有问题.或是不方便?于是给了一个方案,或是说,前面的时候,那块还不能写变量?里面是硬编码写死的。

后面可以了.但是估计很多人还是旧写法,如果直接把这句加入params这个文件的前面话,就会可能跟nginx.conf中同时出现,了二次.就会导致很多莫名的问题。

有可能某些地方会用前面一个指令的路径,而另一个地方会可能用到后面一个指令。
所以,作者保留params,新加一个文件叫fastcgi.conf.

我们可以这样看两者的区别
```
diff fastcgi.conf fastcgi_params
2d1
< fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
```
两者的区别就是多了这句话，所以在配置nginx文件的时候我直接包含的是astcgi.conf而不是fastcgi_params，我之前的问题就解决了。

### 感受
可能写得比较乱，但是谁又会看呢？是不是，自己明白原理后记录下来，也算是巩固了下基础了，哈哈哈。

### 参考资料
[fastcgi.cong 和 fastcgi_params的区别](http://blog.csdn.net/qidizi/article/details/41295661)。
[nginx PHP文件不能正常访问](https://zhidao.baidu.com/question/1765912484737485380.html)。
[Nginx 配置文件详解](https://segmentfault.com/a/1190000002789743)。




