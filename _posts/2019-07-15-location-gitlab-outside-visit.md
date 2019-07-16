---
layout: post
title: 基于frp,gitlab的docker镜像实现本地gitlab外网访问(一)
date: 2019-07-15 
tags: frp,gitlab,docker
---
### 基础
* 会git的基本操作

* 会docker和docker-compose的基本操作


### 基于gitlab的docker镜像安装内网git仓库
> 1、这里docker-compose管理安装

![FgO-PtshY7SQ52npAmUL4en4WGCO](http://img.zzhpeng.cn/FgO-PtshY7SQ52npAmUL4en4WGCO)
然后在当前目录运行docker-compose up,执行时间可能会有点久，请耐性等待。
当执行到开始写日志的时候，就证明本地的gitlab安装成功了。如下图：
![](http://img.zzhpeng.cn/FrMofWRxKkWDy7Nn2Lb4JwjAesJz)

> 2、访问127.0.0.1:8686测试

### 内网穿透frp
> 1、在frp的release找到相应系统的压缩包，里面包含服务端frps和客户端frpc

* mac系统下载frp_0.27.1_darwin_amd64.tar.gz

* linux系统下载frp_0.27.1_linux_amd64.tar.gz

> 2、下载包

* 我在内网用的是mac测试，所以用了frp_0.27.1_darwin_amd64.tar.gz这个包，
外网用linux的frp_0.27.1_linux_amd64.tar.gz。

> 3、配置

* 本地客户端frpc配置

```
[common]
server_addr = *.*.*.* #填写自己的外网服务器的ip
server_port = 7000

[web] 
type = http
local_port = 8686
custom_domains = gitlab.zzhpeng.cn                               

```

* 外网frps配置

```
[common]
bind_port = 7000
vhost_http_port = 8080
subdomain_host = gitlab.zzhpeng.cn                            

```

> 4、启动

* client端启动

```
./frpc -c ./frpc.ini
```


* server端启动

```
./frps -c ./frps.ini
```



### 外网域名配置与测试
> 1、做域名解析，腾讯云云服务器如下配置

![](http://img.zzhpeng.cn/Fpud9LMVf-xDNvznnHnfWusl8N7W)

> 2、域名http://gitlab.zzhpeng.cn:8080访问测试

![](http://img.zzhpeng.cn/FuPWVEIWOixgp6wM2vmBfP84_emk)

### 外网域名访问优化--nginx转发
```

server{
    listen 80;
    server_name  gitlab.zzhpeng.cn;
    root /var/www/html/goShares/public; # 该项要修改为你准备存放相关网页的路径

    #access_log /dev/null;
    #access_log  /var/log/nginx/nginx.localhost.access.log  main;
    error_log  /var/log/nginx/wx.zzhpeng.cn.log  warn;

    location / {
        #resolver 127.0.0.1; #通过配置/etc/dnsmasq.conf，本地解析域名
        proxy_ssl_server_name on;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;
        proxy_pass http://gitlab.zzhpeng.cn:8080; #通过域名访问frp服务
    }

    location = /favicon.ico {
         log_not_found off;
         access_log off;
    }
}
```
效果图如下：
![](http://img.zzhpeng.cn/FiasTv3nEaiDaxUK_PcotivGkE9F)

### 其他优化Q&A
> Q:本地服务器是宝塔搭建的，已经有nginx环境。不用gitlab的docker镜像安装，用一键安装方法，又会帮你弄一个nginx环境，起冲突。该怎么办？

A:一键安装可以选择是否安装nginx，然后参考gitlab的nginx的转发。

> Q:本地开发与公司内部git仓库交流不用翻到外网再到公司内网，直接与内网地址连接，怎么做？

A:上面本地docker-compose搭建的环境，将映射出去的8686端口改为80，映射出来的config配置文件下有gitlab.rb，修改如下配置项并改公司路由器的host：
```
 # For HTTP
 external_url "http://gitlab.example.com:8929"

 or

 # For HTTPS (notice the https)
 external_url "https://gitlab.example.com:8929"
```





### 资源列表
* frp的github仓库：https://github.com/fatedier/frp

* gitlab的github仓库：https://github.com/gitlabhq/gitlabhq

* gitlab的docker安装指导文档：https://docs.gitlab.com/omnibus/docker/

