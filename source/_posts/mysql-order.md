---
title: MYSQL多个排序时出现的问题
date: 2017-03-08 14:12:09
tags:
	- 学习笔记
	- mysql
---
<img src="http://www.logw.jp/wp-content/uploads/2011/08/mysqllogo.png" width="40%" height="30%" />
<!--more-->
## 工作中无意中发现的坑
在统计人员绩效的时候有限按照发布日期，群组id进行排序，然后进行分页就发生了奇怪的问题。。。

### 查询第2页的sql

```
SELECT * FROM book_performance_item 
WHERE is_deleted = 'N' AND team_id = '23' AND job = 'edit' AND pdate >= '2017-03-05' AND pdate <= '2017-03-05'  
ORDER BY pdate DESC,`team_id`desc LIMIT 10 OFFSET 10
```
<!-- more -->

### 查询结果

![](https://stackdevji.github.io/pics/sql1.jpeg)

### 查询第3页的sql

```
SELECT * FROM book_performance_item 
WHERE is_deleted = 'N' AND team_id = '23' AND job = 'edit' AND pdate >= '2017-03-05' AND pdate <= '2017-03-05'  
ORDER BY pdate DESC,`team_id`desc LIMIT 10 OFFSET 20
```
### 查询结果

![](https://stackdevji.github.io/pics/sql1.jpeg)

我很奇怪明明查的是两页的不同数据，为什么返回的数据是一样的呢？

## 推测
经过查询资料和与同事之间探讨，发现这可能是多个order by 查询时，当排序值都一样的时候，会造成排序乱序，
那我们在经过分页查询的时候很可能数据跟我们想要的结果就不一样了，这时我们需要增加一个排序值，这个排序值是唯一的就能避免上述问题
的发生了。具体为什么mysql会给出这样的结果就得研究mysql排序算法是怎么实现的或者去问问DBA了，由于现在能力有限，暂时
研究不了mysql源码。。。哈哈。




