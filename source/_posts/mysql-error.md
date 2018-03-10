---
title: 一次Mysql启动的问题
date: 2016-11-30 21:41:57
tags:
   - mysql
---
<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510990565602&di=37b1bf367448a2658f08562d8d2cab33&imgtype=0&src=http%3A%2F%2Fm.tuniucdn.com%2Ffb2%2Ft1%2FG2%2FM00%2F02%2F2B%2FCii-T1gphFyIQHX6ACwnIkU26YMAAEZ0gAAAAAALCc6522_w500_h280_c1_t0.jpg" />

## 场景
刚到学霸君的时候，终于开始上手项目了，本地环境搭建起来之后，东哥让我将一个文件导入本地数据库。
结果就出现问题了。
<!-- more -->
### 为什么我不启动Mysql?
我干了一件特别傻缺的事情就是不启动mysql,就直接链接数据库，哈哈。

```
mysql -u root -p
```

mysql肯定不答应啊！所以还是乖乖的启动mysql吧！
```
➜  mysql mysql -u root -p
Enter password:
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
➜  mysql mysql.server start
```
结果还是不行：
```
➜  mysql mysql.server start
Starting MySQL
.. ERROR! The server quit without updating PID file (/usr/local/var/mysql/GZIT000375deMacBook-Pro.local.pid).
```
上网查了可能原因有很多，参考链接[ centos7 mysql The server quit without updating PID file](http://blog.csdn.net/caiyaodeng/article/details/45937183)

我的问题就是
1. **权限不够**
2. **可能已经有进程在占用3306端口**

