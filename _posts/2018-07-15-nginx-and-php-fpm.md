---
layout: post
title: 关于nginx和php-fpm
date: 2018-07-15 
tags: php-fpm nginx 
---

### 1、网友一
Nginx 是非阻塞IO & IO复用模型，通过操作系统提供的类似 epoll 的功能，可以在一个线程里处理多个客户端的请求。
Nginx 的进程就是线程，即每个进程里只有一个线程，但这一个线程可以服务多个客户端。

PHP-FPM 是阻塞的单线程模型，pm.max_children 指定的是最大的进程数量，pm.max_requests 指定的是每个进程处理多少个请求后重启(因为 PHP 偶尔会有内存泄漏，所以需要重启).
PHP-FPM 的每个进程也只有一个线程，但是一个进程同时只能服务一个客户端。

大多数的 Linux 程序都倾向于使用进程而不是线程，因为 Linux 下相对来说创建进程的开销比较小，而 Linux 的线程功能又不是很强大。
### 2、网友二
php-fpm 是多进程的 fast-cgi 管理服务

nginx 主进程是单线程的， master 进程会 fork 出多个 worker 进程，每个 worker 进程也同样是单线程模式，worker 进程采用的是 epoll 模型，异步非阻塞/事件驱动/IO多路复用，所以 nginx 比较适用于处理高并发请求的场景，就像现在的 nodejs，nodejs 是单进程且单线程模式，master 进程只有一个线程，负责请求调度和轮询回调事件的处理工作。

在 nginx + php-fpm 的架构模式下

nginx 的 worker 服务进程会把请求转发给 php-fpm 服务，它们是通过 unix socket 或 tcp socket 进行通信的，worker 把 request_1 发送给 php-fpm 后并不会同步阻塞等待 php-fpm 返回数据，而是将此次请求放入事件队列，接着去处理新的请求 request_2，同时会监测有没有回调事件触发了，这里的回调事件便是 request_1 的请求处理好了并已经返回了数据，worker 只需要把数据返回给客户端即可。
php-fpm 相当于一个进程池，里面维系着一定数量的php服务进程，等待 nginx 的 worker 发送任务，处理完成后返回给 nginx 的 worker。
### 联想
### 如何查看进程数
1、ps -ef | wc -l
2、ps -ef | grep httpd | wc -l

### 如何查看某进程数的线程数
1、pstree -p 4761（pid）
2、cat /proc/4761（pid）/status
3、top -p 4761（pid），然后按H。







[1] https://segmentfault.com/q/1010000000326510.
[2] https://segmentfault.com/q/1010000004621209







