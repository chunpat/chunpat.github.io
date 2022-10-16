---
layout: post 
title: Linux角色，目录权限(含实际案例)
date: 2022-10-15 
tags: Linux
---

## 起因

有时候程序写入内容、创建文件目录无权限问题导致程序异常。这大部分是没有搞清权限
和角色的问题。今天就结合实际案例去复习并整理下。

## 知识储备

### 基础

|权限| 解释|                       
|---|--------------------------------|
|读取（r）| 允许查看文件内容，显示目录列表，对应数字4|
|写入（w）| 允许修改文件内容，允许在目录中新建、删除、移动文件或者子目录，对应数字2|
|可执行（x）| 允许运行程序，切换目录，对应数字1|
|无权限（-）| 没有权限|


在某个项目下，操作如下：
```
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la
total 892
...//省略
-rw-r--r--  1 onehour onehour    146 Mar 10  2022 apidoc.json
...//省略
```

表格解析如下：

|-rw-r--r-- |  1   |  onehour onehour  |  146     |  Mar 10  2022    |
|---|--------------------------------|---|---|---|
|文件类型与权限 | 如果是目录，表示下面文件个数 | 归属，第一个onehour是属主，第二个是属主的分组 | 文件大小，默认普通文件才直接显示大小，单位字节B。 | 修改日期|

> 文件类型与权限

可以拆开四个组件 -、rw-、r--、r--, 对应 文件类型、属主权限 、属组权限 、其他人权限。

其中文件类型有常用的四种，如下表。

|文件类型符号| 代表|                   
|---|--------------------------------|
|- |    普通文件|
|  d |    代表目录|
|  c | 代表字符型文件|
|  l | 代表链接文件|


### 操作

> 权限修改 **chmod** 原本权限是644-》777

我是用权限对应的数字去修改， 如下
```shell
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la apidoc.json           
-rw-r--r-- 1 onehour onehour 146 Mar 10  2022 apidoc.json
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ sudo chmod -R 777 apidoc.json
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la apidoc.json 
-rwxrwxrwx 1 onehour onehour 146 Mar 10  2022 apidoc.json
```

操作解释

|chmod| -R   | 777 | apidoc.json |                         
|---|--------------------------------|---|----|
|权限修改| 下面有目录，继承 | 属主权限 、属组权限 、其他人权限 都是满权限 |  文件|


> 归属（所有权）**chown** 原本归属是onehour onehour -》www www

```shell
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la apidoc.json 
-rwxrwxrwx 1 onehour onehour 146 Mar 10  2022 apidoc.json
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ sudo chown -R www:www apidoc.json 
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la apidoc.json 
-rw-r--r-- 1 www www 146 Mar 10  2022 apidoc.json

```

## 实战

假设用的是宝塔管理服务器，用的nginx代理服务，执行的用户www:www，默认配置如下：

```
user  www www;  //执行用户
worker_processes auto;
error_log  /www/wwwlogs/nginx_error.log  crit;
pid        /www/server/nginx/logs/nginx.pid;
worker_rlimit_nofile 51200;
```

cgi用的是php-fpm，执行的用户www:www，php-fpm.conf配置如下：
```
[global]
pid = /www/server/php/72/var/run/php-fpm.pid
error_log = /www/server/php/72/var/log/php-fpm.log
log_level = notice

[www] //执行用户
listen = /tmp/php-cgi-72.sock
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www
pm = dynamic
pm.status_path = /phpfpm_72_status
pm.max_children = 100
pm.start_servers = 20
pm.min_spare_servers = 20
pm.max_spare_servers = 70
request_terminate_timeout = 100
request_slowlog_timeout = 30
slowlog = var/log/slow.log
```

如果我们登录linux服务器，用的是onehour这个账户管理与操作，那么执行git clone 下来的项目会是onehour onehour的用户组。
```
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot$ ls -lh 
drwxr-xr-x 20 onehour onehour 4.0K Oct  5 21:57 dev.xxx.com
```

**这样会存在什么问题呢？**

如果网站访问，用php-fpm写日志操作，假设写的目录是runtime，如下：
```
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la runtime/
total 28
drwxr-xr-x  5 onehour onehour  4096 Mar 11  2022 .
drwxr-xr-x 20 onehour onehour  4096 Oct  5 21:57 ..
drwxr-xr-x  3 onehour onehour  4096 Mar 10  2022 cache
drwxr-xr-x 10 onehour onehour  4096 Oct  1 00:30 log
drwxr-xr-x  2 onehour onehour 12288 Oct 10 11:53 temp
```

其他用户是r-x，是没有写入权限的。

解决办法，将runtime目录归于权限为www:www
```
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ sudo chown -R www:www runtime
onehour@iZwz9at7o13nmv1vs5ps94Z:/www/wwwroot/dev.xxx.com$ ls -la runtime/              
total 28
drwxr-xr-x  5 www     www      4096 Mar 11  2022 .
drwxr-xr-x 20 onehour onehour  4096 Oct  5 21:57 ..
drwxr-xr-x  3 www     www      4096 Mar 10  2022 cache
drwxr-xr-x 10 www     www      4096 Oct  1 00:30 log
drwxr-xr-x  2 www     www     12288 Oct 10 11:53 temp
```

额外，如果执行脚本，用crontab去跑，这里的执行用户就是登录的账号onehour了,那么写日志是没权限的，这个怎么办呢？

我这边的做法是将执行crontab的用户改为www去执行。
这里改的话，需要登录对应账号执行crontab -e编辑，或者在root账号下crontab -e -u www。

如果是脚本，像宝塔的计划任务用的是root执行，可以改写成下面这样:
```
su -s /bin/bash - www <<EOF
.......//todo执行的命令
EOF
```

## 迭代

* 2022年10月15日 21:00:00 初稿

## 参考

1、[一文带你彻底搞懂Linux 文件权限管理](https://segmentfault.com/a/1190000039202476)