---
layout: post
title: sql语句中的GROUP BY 和 HAVING的使用count()
date: 2017-06-08 
tags: mysql   
---


这个sql语句是我分析tpshop2.0.4的登录流程的时候，会检测购物车重复数据问题！

![](http://img.zzhpeng.cn/FuwyoA14FY-Cbc-mhA6qFr6t7EIu)
这条语句有点不懂，所以查了下某度。http://blog.163.com/hks_blog/blog/static/214926090201382225845920/

这条链接写的挺详细的，并且还有demo，值得参考。

简要总结下关键点。

<strong>1.group by 和having  </strong>
<ul>
 	<li>having是分组（group by）后的筛选条件，分组后的数据组内再筛选。</li>
 	<li>where则是在分组前筛选。</li>
</ul>
<strong>2.聚合函数</strong>
<ul>
 	<li>SUM, COUNT, MAX, AVG</li>
</ul>
&nbsp;

摘录-----------------------------------------------------------------------------------------------------------

&nbsp;

SQL实例：

一、显示每个地区的总人口数和总面积．
SELECT region, SUM(population), SUM(area)
FROM bbc# `&amp; e4 k' X* n1 v% ?+ |
GROUP BY region
先以region把返回记录分成多个组，这就是GROUP BY的字面含义。分完组后，然后用聚合函数对每组中的不同字段（一或多条记录）作运算。# B* i' z  `, }* S, E5 i

二、 显示每个地区的总人口数和总面积．仅显示那些面积超过1000000的地区。
SELECT region, SUM(population), SUM(area)7 ]; Z&amp; I! t% i
FROM bbc8 F4 w2 v( P- f
GROUP BY region
HAVING SUM(area)&gt;1000000# y" P  z. O7 D9 `# X
在这里，我们不能用where来筛选超过1000000的地区，因为表中不存在这样一条记录。
相反，HAVING子句可以让我们筛选成组后的各组数据

&nbsp;

三、查询CUSTOMER 和ORDER表中用户的订单数

select c.name, count(order_number) as count from orders o,customer c where c.id=o.customer_id group by customer_id;

+--------+-------+
| name   | count |
+--------+-------+
| d      |     9 |
| cc     |     6 |
| 菩提子 |     1 |
| cccccc |     2 |
+--------+-------+

增加HAVING过滤

select c.name, count(order_number) as count from orders o,customer c where c.id=o.customer_id group by customer_id having count(order_number)&gt;5;

+------+-------+
| name | count |
+------+-------+
| d    |     9 |
| cc   |     6 |
+------+-------+

&nbsp;

&nbsp;

四、我在多举一些例子

SQL&gt; select * from sc;

SNO PNO        GRADE
---------- ----- ----------
1 YW             95
1 SX              98
1 YY             90
2 YW            89
2 SX             91
2 YY             92
3 YW            85
3 SX             88
3 YY             96
4 YW            95
4 SX             89

SNO PNO        GRADE
---------- ----- ----------
4    YY            88

这个表所描述的是4个学生对应每科学习成绩的记录，其中SNO（学生号）、PNO（课程名）、GRADE（成绩）。

1、显示90分以上学生的课程名和成绩

//这是一个简单的查询，并没有使用分组查询

SQL&gt; select sno,pno,grade from sc where grade&gt;=90;

SNO PNO        GRADE
---------- ----- ----------
1 YW            95
1 SX             98
1 YY             90
2 SX             91
2 YY             92
3 YY             96
4 YW            95

已选择7行。

2、显示每个学生的成绩在90分以上的各有多少门

//进行分组显示，并且按照where条件之后计数

SQL&gt; select sno,count(*) from sc where grade&gt;=90 group by sno;

SNO   COUNT(*)
---------- ----------
1          3
2          2
4          1
3          1

3、这里我们并没有使用having语句，接下来如果我们要评选三好学生，条件是至少有两门课程在90分以上才能有资格，列出有资格的学生号及90分以上的课程数。

//进行分组显示，并且按照where条件之后计数，在根据having子句筛选分组

SQL&gt; select sno,count(*) from sc where grade&gt;=90 group by sno having count(*)&gt;=2；

SNO   COUNT(*)
---------- ----------
1          3
2          2

这个结果是我们想要的，它列出了具有评选三好学生资格的学生号，跟上一个例子比较之后，发现这是在分组后进行的子查询。

4、学校评选先进学生，要求平均成绩大于90分的学生都有资格，并且语文课必须在95分以上，请列出有资格的学生

//实际上，这个查询先把语文大于95分的学生号提取出来，之后求平均值，分组显示后根据having语句选出平均成绩大于90的

SQL&gt; select sno,avg(grade) from sc where SNO IN (SELECT SNO FROM SC WHERE GRADE&gt;=95 AND PNO='YW') group by sno having avg(grade)&gt;=90;

SNO AVG(GRADE)
---------- ----------
1    94.3333333
4    90.6666667

5、查询比平均成绩至少比学号是3的平均成绩高的学生学号以及平均分数

//having子句中可进行比较和子查询

SQL&gt; select sno,avg(grade) from sc
group by sno
having avg(grade) &gt; (select avg(grade) from sc where sno=3);







