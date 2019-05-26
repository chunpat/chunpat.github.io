---
layout: post
title: 关于php的运行方式
date: 2018-07-15 
tags: php
---

好久好久前有个同事问我，同一台服务器怎么用两个php版本，当时解决了，只是简单的教他怎么切换。如今空闲觉得有必要整理下相关的知识。
## 常见的五大运行模式

* 1）CGI（通用网关接口/ Common Gateway Interface）
* 2）FastCGI（常驻型CGI / Long-Live CGI）
* 3）CLI（命令行运行 / Command Line Interface）
* 4）Web模块模式（Apache等Web服务器运行的模式）
* 5）ISAPI（Internet Server Application Program Interface）

> 备注：在PHP5.3以后，PHP不再有ISAPI模式，安装后也不再有php5isapi.dll这个文件。要在IIS6上使用高版本PHP，必须安装FastCGI 扩展，然后使IIS6支持FastCGI。

### 1）Cgi
CGI全称是“公共网关接口”(Common Gateway Interface)，是一个跨语言沟通的标准(协议)。例如，HTTP请求和WEB服务器沟通是没有问题的，但是请求带有动态文件的时候，WEB服务器是无法处理的。所以需要找合伙人（会处理动态文件的人），合作的话，双方需要达成协议。这个协议标准就是Cgi啦。

### 2）FastCGI
FastCGI是在CGI基础上加上了Fast，快速达成协议。CGI这种合作，是一笔短暂的交易。若想要长期合作就需要加强下协议的细节了。所以就有了FastCGI，它也是CGI的加强版。

### 3）CLI
命令行模式,在CLI直接运行文件。

### 4）Web模块模式
相当于合伙人驻场了，合作密切，工作作息时间一致了。在linux上，apache有如下这段代码。
httpd.conf文件

```shell
LoadModule alias_module modules/mod_alias.so
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule php5_module        modules/libphp5.so
```
windows下phpstudy
```shell
LoadFile "D:/phpstudy2/php/php-5.5.38/php5ts.dll"
LoadModule php5_module "D:/phpstudy2/php/php-5.5.38/php5apache2_4.dll"
<IfModule php5_module>
PHPIniDir "D:/phpstudy2/php/php-5.5.38/"
</IfModule>
LoadFile "D:/phpstudy2/php/php-5.5.38/libssh2.dll"
<FilesMatch "\.php$">
    SetHandler application/x-httpd-php
</FilesMatch>
```
 详细解说可以查看下面的参考文献

## 图说 php-cgi和 php-fpm
### php-cgi

![](https://s1.ax1x.com/2018/07/15/PMfbeH.png)

### php-fpm

![](https://s1.ax1x.com/2018/07/15/PMf7Oe.png)


## nginx与php-fpm通信的两种方式
在Linux中，nginx服务器和PHP-fpm可以通过tcp socket和unix socket两种方式实现。

unix socket是一种终端，可以使同一台操作系统上的两个或多个进程进行数据通信。这种方式需要再nginx配置文件中填写php-fpm的pid文件位置，效率要比tcp socket高。

tcp socket的优点是可以跨服务器，当nginx和php-fpm不在同一台机器上时，只能使用这种方式。

* windows系统只能使用tcp socket的通信方式

### 配置方法
#### tcp socket
>tcp socket通信方式，需要在nginx配置文件中填写php-fpm运行的ip地址和端口号。

```shell
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME /var/www/website$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
}
```
#### unix socket
>unix socket通信方式，需要在nginx配置文件中填写php-fpm运行的pid文件地址。

```shell
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME /var/www/website$fastcgi_script_name;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
}
```

php-fpm的运行端口号和pid文件的地址都是在php-fpm.conf中配置的。
php-fpm.conf文件在php安装文件的/etc目录下，比如你的php安装在/opt/php目录，则应该是/opt/php/php-fpm.conf。

## PHP的nts 和 zts(ts)
nts是non-thread-safe,ts是thread-safe
nts和zts用过windows的同学都有看见过。Linux上的PHP同样有NTS和TS版本的区别,默认是NTS版本,configure时加上--enable-maintainer-zts则编译为TS版本.什么时候需要TS版本呢?比如你要使用pthreads这个多线程的PECL扩展时,或者PHP以MOD_PHP嵌入多线程运行下的Apache,比如Apache在Linux上提供的Event MPM就是一个多进程多线程的工作模型,Windows上Apache采用的WinNT MPM也是一个多线程模型,这时都需要TS版本的PHP。
而如果以PHP-FPM(比如搭配Nginx或者Apache的mod_fastcgi)或者PHP-CGI(比如搭配Apache的mod_fcgid或者Win上的IIS)来运行PHP,则一般都不需要TS线程安全版本的PHP.

参考文献
[1] https://blog.csdn.net/xiaoxiaoqiye/article/details/52094004
[2] https://blog.csdn.net/koastal/article/details/52303316
[3] https://segmentfault.com/q/1010000002974864







