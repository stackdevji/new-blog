---
title: B+Tree
date: 2018-01-17 09:48:31
tags:
	- 数据结构
---

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Bplustree.png/800px-Bplustree.png" width="474px" height="228px">

B+Tree是B-Tree的一种变种，也是一种为磁盘设计的多路平衡搜索树。
<!--more-->
### B+Tree与B树的区别
1. 非叶子节点只存储键值信息。
2. 所有叶子节点之间都有一个链指针。
3. 数据记录都存放在叶子节点中。
4. B+Tree非叶子节点的关键字都出现在叶子节点中。

### B+Tree的节点结构
在B+Tree中的节点通常被表示为一组有序的元素和子指针。如果此B+树的阶是m ，则除了根之外的每个节点都包含最少 [m/2] 个元素最多 m-1 个元素，对于任意的节点有最多 m 个子指针。对于所有内部节点，子指针的数目总是比元素的数目多一个。因为所有叶子都在相同的高度上，节点通常不包含确定它们是叶子还是内部节点的方式。

每个内部节点的元素充当分开它的子树的分离值。例如，如果内部节点有三个子节点（或子树）则它必须有两个分离值或元素 a1 和 a2。在最左子树中所有的值都小于等于 a1，在中间子树中所有的值都在 a1 和 a2 之间(a1，a2]，而在最右子树中所有的值都大于 a2。

> 网上有的博客说B+Tree的关键字个数和指针个数相等，在维基百科中和《Mysql技术与内幕中》B+Tree部分中都介绍到节点的关键字个数和指针个数的关系和B-Tree是一样的。这里以后者为主。

### B+Tree的查找
查找以典型的方式进行，类似于二叉查找树。起始于根节点，自顶向下遍历树，选择其分离值在要查找值的任意一边的子指针。在节点内部典型的使用是二分查找来确定这个位置。

### B+Tree的插入操作
由于B+Tree的非叶子节点中存放的是关键字，我们用Index Page来表示，叶子节点中存放的是关键字和数据，这里用Leaf Page来表示。
1. 如果Leaf Page空间满足，Index Page空间满足，直接将记录插入到叶子节点中Leaf Page.
2. 如果Leaf Page空间不足，Index Page空间满足，拆分Leaf Page,将中间值放入到Index Page中，小于中间值的记录放左边，大于或等于中间值的记录放右边。
3. 如果Leaf Page、Index Page空间都不足，拆分Leaf Page,将中间节点放入到Index Page中，小于中间节点的记录放左边，大于或等于中间节点的记录放右边。拆分Index Page, 小于中间节点的记录放左边，大于或等于中间节点的记录放右边。中间值放入到上一层Index Page中。

> 由于B+Tree要保持平衡，新插入的键值可能需要做大量的拆分页操作，因为B+Tree主要用于磁盘，所以要减少页的拆分操作，因此在Leaf Page空间满了的时候，但是其左右兄弟节点没有满的时候，B+Tree会通过旋转操作代替拆分操作，通常是将左兄弟节点进行旋转。
(a) 旋转左兄弟，Leaf Page最小关键字（同时存在上层Index Page中）移动到左兄弟中，Leaf Page第二小关键字放到Index Page中。
(a) 旋转右兄弟，Leaf Page最大关键字上移到上层Index Page中，Index Page中 用来分离Leaf Page和有兄弟的关键字下移到右兄弟中。

### B+Tree的删除操作
删除操作要考虑到节点的填充因子或者说是填充度，50%是填充因子可设的最小值。B+Tree的删除操作要和插入操作一样，要保证节点中关键字的顺序。
1. Leaf Page填充因子 > 50%，Index Page填充因子 > 50%，直接将记录在叶子节点中删掉，如果该记录还是Index Page中的节点,用该记录右面的记录替换。
2. Leaf Page填充因子 < 50%，Index Page填充因子 > 50%, 合并叶子节点和他的兄弟节点，同时更新Index Page。
3. Leaf Page填充因子 < 50%，Index Page填充因子 < 50%, 合并叶子节点和他的兄弟节点，同时更新Index Page, 合并Index Page。



