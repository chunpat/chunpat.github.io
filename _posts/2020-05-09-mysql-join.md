---
layout: post
title: 重学 MySQL 的Left Join操作
date: 2020-05-12
tags: mysql
---
软件版本：mysql8.0.18

## 起因
今日有个需求，根据门店流水算出直营门店当月的使用天数。这需求大概只需要门店表和门店流水表即可，看上去挺简单，两张表left join，然后group by
就能解决，但是调整了下，结果却不一样了，感到迷惑。我大概只记得，以左边为主，遍历右边，on后面条件匹配，匹配失败的话右边字段弄成null。
但是，这不足以解答我的疑惑，所以重新学习下join的操作原理。

## 测试的Sql
```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for store
-- ----------------------------
DROP TABLE IF EXISTS `store`;
CREATE TABLE `store` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '门店名称',
  `business_type` tinyint(255) DEFAULT NULL COMMENT '门店类型:1直营2加盟',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ----------------------------
-- Records of store
-- ----------------------------
BEGIN;
INSERT INTO `store` VALUES (1, '门店A', 1);
INSERT INTO `store` VALUES (2, '门店B', 2);
INSERT INTO `store` VALUES (3, '门店C', 1);
INSERT INTO `store` VALUES (4, '门店D', 1);
COMMIT;

-- ----------------------------
-- Table structure for store_wallet_bill
-- ----------------------------
DROP TABLE IF EXISTS `store_wallet_bill`;
CREATE TABLE `store_wallet_bill` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `bill` decimal(8,2) DEFAULT NULL COMMENT '流水',
  `store_id` int(11) DEFAULT NULL COMMENT '门店id',
  `create_time` datetime DEFAULT NULL COMMENT '生成时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ----------------------------
-- Records of store_wallet_bill
-- ----------------------------
BEGIN;
INSERT INTO `store_wallet_bill` VALUES (1, 1.00, 1, '2020-05-10 21:29:49');
INSERT INTO `store_wallet_bill` VALUES (2, 1.00, 1, '2020-05-03 21:30:11');
INSERT INTO `store_wallet_bill` VALUES (3, 1.00, 1, '2020-05-07 21:30:18');
INSERT INTO `store_wallet_bill` VALUES (4, 1.00, 2, '2020-05-06 21:30:42');
INSERT INTO `store_wallet_bill` VALUES (5, 1.00, 2, '2020-05-08 21:30:54');
INSERT INTO `store_wallet_bill` VALUES (6, 2.00, 3, '2020-05-08 21:31:10');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

## 两条Sql语句为什么结果不同
> 1、第一条Sql

```sql
SELECT
	`store`.NAME,
	`store`.business_type,
	SUM(`store_wallet_bill`.bill) 'sum_bill',
	COUNT( DISTINCT LEFT ( `store_wallet_bill`.create_time, 10 ) ) 'use_day'
FROM
	`store`
	LEFT JOIN store_wallet_bill ON `store`.id = store_wallet_bill.store_id
where `store`.business_type = 1	
GROUP BY
	store_id 
ORDER BY
	`use_day` desc
```
显示:

name|business_type|sum_bill|use_day
---|---|---
门店A | 1 | 3.00 | 3
门店C | 1 | 2.00 | 1
门店D | 	1 |  (NULL)	| 0


> 2、第二条Sql，将第一条Sql的where后面的`store`.business_type = 1移到left join的on后面。

```sql
SELECT
	`store`.NAME,
	`store`.business_type,
	SUM(`store_wallet_bill`.bill) 'sum_bill',
	COUNT( DISTINCT LEFT ( `store_wallet_bill`.create_time, 10 ) ) 'use_day'
FROM
	`store`
	LEFT JOIN store_wallet_bill ON `store`.id = store_wallet_bill.store_id AND `store`.business_type = 1	
GROUP BY
	store_id 
ORDER BY
	`use_day` desc
```
显示:

name|business_type|sum_bill|use_day
---|---|---
门店A | 1 | 3.00 | 3
门店C | 1 | 2.00 | 1
门店B | 	2 | (NULL) | 0


## Mysql的Left Join解析
mysql 对于left join的采用类似嵌套循环的方式来进行从处理，以下面的语句为例：
```sql
SELECT * FROM LT LEFT JOIN RT ON P1(LT,RT)) WHERE P2(LT,RT)
```
其中P1是on过滤条件，缺失则认为是TRUE，P2是where过滤条件，缺失也认为是TRUE，该语句的执行逻辑伪代码大概如下：
```
FOR each row lt in LT {   // 遍历左表的每一行
  BOOL b = FALSE;
  FOR each row rt in RT such that P1(lt, rt) {// 遍历右表每一行，找到满足join条件的行 !!!!注意这里，这就是on后面加上`store`.business_type = 1但是还是会出现business_type=2的原因
    IF P2(lt, rt) {//满足 where 过滤条件
      t:=lt||rt;//合并行，输出该行
    }
    b=TRUE;// lt在RT中有对应的行
  }
  IF (!b) { // 遍历完RT，发现lt在RT中没有有对应的行，则尝试用null补一行  
    IF P2(lt,NULL) {// 补上null后满足 where 过滤条件
      t:=lt||NULL; // 输出lt和null补上的行
    }         
  }
}
```

**从上面的伪代码可以解析我上面的两条Sql为什么是这个结果了**

## 两条Sql疑惑解析

> 1、第一条Sql

从上面的伪代码可以看到，左边store表的name='门店D'的数据在右边匹配不到，所以第二个遍历都满足不了，而满足了IF (!b) ，所以右边是null。
那么name='门店B'怎么不见这一条呢？因为满足了第二个遍历，但是不满足 IF P2(lt, rt) 过滤条件而b=TRUE，所以这左边name='门店B'一条都没
组合到。

> 2、第二条Sql

从上面的伪代码可以看到，on后面加上`store`.business_type = 1 。左边store表的name='门店D' 满足第二个遍历条件，不满足IF P2(lt, rt) ，
所以一条都没。左边store表的name='门店B',不满足第二个遍历条件，所以右边是null。

## 豁然开朗

- 1、在left join中，不是左右遍历完了后再where的（我在此之前都以为是这样的），而是每条遍历都where。

## 注意

Mysql5.7+版本限制使用GROUP BY 的select 筛选元素，报错大概如下
```
SELECT list is not in GROUP BY clause and contains nonaggregated column 'userinfo.t_long.user_name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

> 1、临时性解决方案

```
select @@sql_mode; //会显示一些sql_mode属性
```

然后去掉 ONLY_FULL_GROUP_BY，重新设置
```
set global sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
set session sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
```

> 2、永久更改

修改my.ini，去掉ONLY_FULL_GROUP_BY，重启服务


## 迭代
* 2020年05月24日 00:17:00 初稿

## 参看：
* 1、[MySQL：left join 避坑指南 - 知乎](https://zhuanlan.zhihu.com/p/93445040)
