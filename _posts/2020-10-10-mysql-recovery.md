---
layout: post
title: Mysql 逻辑数据恢复
date: 2020-10-10
tags: mysql mysql-logic-recovery
---

## 起因
有时候线上出了问题，而原本地数据不好操作，所以想把生产的数据弄一份到本地环境测试。操作步骤：1、阿里云RDS导出逻辑。2、导入本地mysql。

## 阿里云RDS生成逻辑
![Fh3k1F_AOs_QW_1tbIFDs3-f9R7M](http://img.chunpat.cn/Fh3k1F_AOs_QW_1tbIFDs3-f9R7M)

点击确认后，就会生成，然后下载即可。

## 数据库逻辑恢复

解压下载后压缩文件，然后选择对应要恢复的数据库。`注意：➜ source 是目录`
```
➜ source gunzip -c sima_datafull_202009291740_1601372414.sql.gz|mysql -uroot -p sima
Enter password: 
ERROR 1840 (HY000) at line 24: @@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_EXECUTED is empty.
```

`上面出现error`，度娘下，提示集群迁移到单实例的设置问题，操作：
```
➜  source mysql -uroot -p 
Enter password: 
mysql> reset master;
Query OK, 0 rows affected (0.01 sec)
```
然后重复上面操作即可。

## 迭代
* 2020年10月10日 14:14:40 初稿

## 参考：
* 1、[高性能mysql](https://book.douban.com/subject/23008813/)
* 2、[ERROR 1840 (HY000) at line 24: @@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_EXECUTED is empty - tonnytangy - 博客园](https://www.cnblogs.com/tonnytangy/p/7779164.html)
