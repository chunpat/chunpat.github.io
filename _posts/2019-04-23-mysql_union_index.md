---
layout: post
title: mysql联合索引
date: 2019-04-23 
tags: mysql   
---

## mysql联合索引中
* 1、select * from table where a=1, b=2
* 2、select * from table where b=1, a=2
* 3、select * from table where b=1
以上哪一句不会触发索引？为什么？

答曰：3，因为联合索引的最左匹配原则，详细如下。

## Mysql中联合索引的最左匹配原则

  在Mysql建立多列索引（联合索引）有最左前缀的原则，即最左优先。如果我们建立了一个2列的联合索引(col1,col2),实际上已经建立了两个联合索引(col1)、(col1,col2）;如果有一个3列索引(col1,col2,col3)，实际上已经建立了三个联合索引(col1)、(col1,col2)、(col1,col2,col3)。并且最左优先原则是基于BTREE算法,HASH算法不行,因为简单地说，哈希索引就是采用一定的哈希算法，把键值换算成新的哈希值，检索时不需要类似B+树那样从根节点到叶子节点逐级查找，只需一次哈希算法即可立刻定位到相应的位置，速度非常快。具体原理看这篇[MySQL B+树索引和哈希索引的区别 - 梦中山河 - 博客园](https://www.cnblogs.com/heiming/p/5865101.html)

## 为什么上面例子3不会触发索引
  b+树的数据项是复合的数据结构，比如(name,age,sex)的时候，b+树是按照从左到右的顺序来建立搜索树的，比如当(张三,20,F)这样的数据来检索的时候，b+树会优先比较name来确定下一步的所搜方向，如果name相同再依次比较age和sex，最后得到检索的数据；但当(20,F)这样的没有name的数据来的时候，b+树就不知道第一步该查哪个节点，因为建立搜索树的时候name就是第一个比较因子，必须要先根据name来搜索才能知道下一步去哪里查询。详情查看：[Mysql中联合索引的最左匹配原则 - 王凯华 - 博客园](http://www.cnblogs.com/wangkaihua/p/10220462.html)
  
## 实践验证真理
### 建表
```
CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(10) DEFAULT NULL,
  `sex` tinyint(1) DEFAULT NULL,
  `age` tinyint(2) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `Index_user` (`name`,`age`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4;

```

### 测试

1) explain select * from user where name = 'zzhpeng'
![FkHS_EVmThc5fAnvhwtxR89MEKbV](http://img.zzhpeng.cn/FkHS_EVmThc5fAnvhwtxR89MEKbV)

2) explain select * from user where name = 'zzhpeng' and age = 18
![FhBY0qcm_j1pxEEqBW77-saLXQVh](http://img.zzhpeng.cn/FhBY0qcm_j1pxEEqBW77-saLXQVh)

3) explain select * from user where age = 18
![FqQgNo2OIMtFZ5N3er2aFYipS-7p](http://img.zzhpeng.cn/FqQgNo2OIMtFZ5N3er2aFYipS-7p)

4) explain select * from user where age = 18 and name = 'zzhpeng'
![FhEoYAWN1gVc7D3Y5UBi9Hqeqg0j](http://img.zzhpeng.cn/FhEoYAWN1gVc7D3Y5UBi9Hqeqg0j)