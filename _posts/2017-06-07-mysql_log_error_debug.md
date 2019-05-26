---
layout: post
title: 开启mysql日志排查错误
date: 2017-06-07 
tags: mysql   
---

系统的大半的错误是由于sql语句的错误，所以快速找到那条sql报错至关重要，这里可以通过查看sql来排错！我用的是mysql数据库，其

mysql的配置文件是my.ini。

添加这些代码。

[mysqld]

general_log=ON

general_log_file=D:/testlog

#log-raw=true            如果错误日志没记录，则开启这个

重启mysql生效。

本来想这样直接在打开日志文件看的，不过不方便。并且文件被占用，不能直接操作文件！所以用网页看吧！

工具链接：http://pan.baidu.com/s/1cxHQge

修改config.php即可，并且配置个站点。
![](https://img.zzhpeng.cn/FteLS27Ds9P0DoE9LWoLcNY1VC5R)







