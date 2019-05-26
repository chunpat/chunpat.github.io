---
layout: post
title: linux的进程守护daemontools
date: 2018-05-16 
tags: linux   
---

>介绍

Daemontools是管理Unix服务的工具，它提供一组工具来管理一系列用户进程，当进程由于某些原因down掉之后，daemontools会自动重启进程。主要的功能是Supervisor。

>安装

```shell
mkdir  ~/tools
cd /tools
wget http://cr.yp.to/daemontools/daemontools-0.76.tar.gz
tar xvzf daemontools-0.76.tar.gz
cd admin/daemontools-0.76
package/install

```

### 如果安装出现错误

```shell
/usr/bin/ld: errno: TLS defini  tion in /lib/libc.so.6 section .tbss mismatches non-TLS 
```

### How to fix it 

将admin/daemontools-0.76/src/error.h中的extern int errno;替换为#include <errno.h>


安装完成之后，会创建 /service 和/command两个目录。


>使用daemontools

daemontools是一组service管理工具，其中svscanboot工具用来启动svscan工具。可以通过以下命令启动svscanboot。

```shell
nohup /command/svscanboot &
```
* nohup为终端断开还在后台运行

启动之后，查看进程，可以发现svscan做为svscanboot的子进程在运行。

![](http://www.zzhpeng.cn/wp-content/uploads/2018/05/ps_svs.png)


>实战

启动svscanboot之后，相应的svscan进程也启动起来，其中参数/service 就是管理配置文件的目录。

### 1、创建脚本目录
```shell
mkdir -p /opt/svc/nginx
```

### 2、在目录创建run脚本(名字必须是run而且权限是755)

```shell
touch /opt/svc/nginx/run  && chmod 755 /opt/svc/nginx/run
cat /opt/svc/nginx/run
#!/bin/sh
exec /usr/local/nginx/sbin/nginx   #启动进程命令

```

### 3、创建symbol link，映射到/service下

```shell
ln -s /opt/svc/nginx/  /service/
```

之后svscan会自动检测，并添加任务，每隔几秒执行一次该文件。
查看进程树形图
![](http://www.zzhpeng.cn/wp-content/uploads/2018/05/pstree.png)

从中可以看出来，svscanboot负责启动svscan服务，svscan管理supervise进程。而具体的客户进程，是通过supervise进程来统一管理的。

现在nginx被daemontool管理起来了，试试看杀掉nginx应用进程看看。


>扩展

1、启动被管理的进程 (配置完后无需执行svc命令)
svc -u /service/nginx/  (启动之后，如果nginx挂掉，daemontools会自动重启nginx)

2、关闭被管理的进程（不会关闭daemontools supervise进程）
svc -d /service/nginx/

3、查看service状态
svstat /service/nginx/

4、移除service
rm  /service/nginx   #移除软连接  
svc -dx /opt/svc/nginx/


>实战2 与tp5的think-queue结合
  
  run如下：
```shell
#!/bin/sh
exec /usr/local/php/bin/php /home/wwwroot/thinkphp/think queue:listen --queue helloJobQueue   #启动进程命令
```


>注意

* 1、被管理的进程不能以daemon形式运行，例如nginx.conf 必须关闭daemon， daemon off。
* 2、不要在/service/建任何目录， /service/只存放一些symbol link。

参考文献
http://kavy.iteye.com/blog/2119978







