---
layout: post
title: mysql5.7创建用户
date: 2018-08-30 
tags: mysql   
---


今天把自己服务器的mysql5.7用户误删除了root,所以整理了,如何添加用户。
### 1、mysql5.7加用户或者误删root
- 创建一个新账号
```sql
use mysql;
insert into user set user='root',ssl_cipher='',x509_issuer='',x509_subject='';
```
- 设置默认参数
```sql
update user set Host='localhost',select_priv='y', insert_priv='y',update_priv='y', Alter_priv='y',delete_priv='y',create_priv='y',drop_priv='y',reload_priv='y',shutdown_priv='y',Process_priv='y',file_priv='y',grant_priv='y',References_priv='y',index_priv='y',create_user_priv='y',show_db_priv='y',super_priv='y',create_tmp_table_priv='y',Lock_tables_priv='y',execute_priv='y',repl_slave_priv='y',repl_client_priv='y',create_view_priv='y',show_view_priv='y',create_routine_priv='y',alter_routine_priv='y',create_user_priv='y' where user='root';
```
- 设置密码
```sql
update user set authentication_string=password('root') where user='root'
flush privileges;
```

参考：[点滴记录——Linux Mysql数据库误删root用户 - CSDN博客](https://blog.csdn.net/cywosp/article/details/42145779)
### 2、扩展
mysql -uroot -p 默认是验证mysql.user的host 等于 localhost的而不是127.0.0.1
[不是你想像的简单！配置mysql连接方式的时候，localhost和127.0.0.1的区别。 - CSDN博客](https://blog.csdn.net/love2377/article/details/78249233?locationNum=10&fps=1)




