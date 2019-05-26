---
layout: post
title: docker 入门
date: 2019-03-06 
tags: docker   
---

# 简介
官网 [Enterprise Container Platform | Docker](https://www.docker.com/)
手册 [Docker Documentation | Docker Documentation](https://docs.docker.com/)

# 虚拟机与docker对比
[What is a Container | Docker](https://www.docker.com/resources/what-container)

![FlUbUkJUdU9WSBqD9IBYzHZ3gkSp](http://img.zzhpeng.cn/FlUbUkJUdU9WSBqD9IBYzHZ3gkSp)
# docker-ce 和 docker-ee
### About Docker CE（Community Edition）
Docker Community Edition (CE) is ideal for developers and small teams looking to get started with Docker and experimenting with container-based apps. Docker CE has three types of update channels, stable, test, and nightly:
* Stable gives you latest releases for general availability.
* Test gives pre-releases that are ready for testing before general availability.
* Nightly gives you latest builds of work in progress for the next major release.

For more information about Docker CE, see Docker [click me](https://www.docker.com/community-edition/).
    
Releases： 每六个月发布一个大版本

### About Docker EE（Enterprise Edition）
Docker Enterprise is designed for enterprise development as well as IT teams who build, ship, and run business-critical applications in production and at scale. Docker Enterprise is integrated, certified, and supported to provide enterprises with the most secure container platform in the industry. For more info about Docker Enterprise, including purchasing options, see[ Docker Enterprise](https://www.docker.com/enterprise-edition/).

### Docker CE 和 Docker EE 对比

![FrnYjbS3hXBqxik_zQFpF-SiH-c5](http://img.zzhpeng.cn/FrnYjbS3hXBqxik_zQFpF-SiH-c5)


# docker安装
原始镜像地址会慢，可以用阿里、163等等提供的，不过要注册阿里等账号。
### 1、docker : [Get Docker CE for Ubuntu | Docker Documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  有很多安装途径，实在找不到对应的系统可以用 Binaries 包安装


### 2、docker-compose : [Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)
docker-compose 镜像组合技术 组合成服务service

# dockerfile 镜像构建
[| Docker Documentation](https://docs.docker.com/engine/reference/builder/)
### 案例1，构建一小时nodejs镜像
* 1、docker build --rm --tag nodejs_test .  （基于官网node:11-alpine镜像，构建镜像）
* 2、docker run -it --rm --name node_test -v "$PWD/store_system/":/usr/src/app -w /usr/src/app nodejs_test npm run dev

![Frlszrq6fwzOO7WJjCdeIaEB80rG](http://img.zzhpeng.cn/Frlszrq6fwzOO7WJjCdeIaEB80rG)

### 案例2，构建一小时后端php&nginx开发环境
php 相关资料 如何加扩展 [Docker Hub](https://hub.docker.com/_/php) 

#创建镜像
#docker build --rm -t myphp:7.2-fpm .

#建立网络
#docker network create --subnet=172.23.0.0/16 net1

#启动容器
#docker run --rm -p 9000:9000 --name myphp -v $PWD/www/:/var/www/html/ -v $PWD/conf/php-fpm.conf:/usr/local/etc/php/php-fpm.d/php-fpm.conf -v $PWD/conf/php.ini:/usr/local/etc/php/php.ini --net=net1 --ip 172.23.0.2 -d myphp:7.2-fpm
#docker run --rm -p 8088:80 --name mynginx -v $PWD/www:/www -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf -v $PWD/conf/conf.d:/etc/nginx/conf.d/ -v $PWD/logs/:/var/log/nginx/ --net=net1 --ip 172.23.0.3 --link myphp:7.2-fpm -d nginx

### 案例3，构建python开发环境
scrapy爬虫项目 [codePool/python/autohome_spider at master · LittleLory/codePool · GitHub](https://github.com/LittleLory/codePool/tree/master/python/autohome_spider)
![Fg7kWIhRoHX5GSrOs_Jl1VDXWla8](http://img.zzhpeng.cn/Fg7kWIhRoHX5GSrOs_Jl1VDXWla8)



# docker-compose 镜像组建管理工具
Compose is a tool for defining and running complex applications with Docker. With Compose, you define a multi-container application in a single file, then spin your application up in a single command which does everything that needs to be done to get it running.
手册：[Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/)

项目地址：[GitHub - FromChinaBoy/dnmp: Docker LNMP (Nginx, PHP7/PHP5, MySQL, Redis)](https://github.com/FromChinaBoy/dnmp)
> 有关php72配置参数的解释
cap_add:
      - SYS_PTRACE

因为有用到xdebug打断点，需要获取sys的拦截系统能力。

参考：[[译] 玩转ptrace (一) - twoon - 博客园](https://www.cnblogs.com/catch/p/3476280.html)

# docker 网络
创建网段  
docker network create --subnet=172.18.0.0/16 net1

### docker的bridge网桥和虚拟机的网桥是不同的
[Use bridge networks | Docker Documentation](https://docs.docker.com/network/bridge/)

### docker-compose下的设置
![FgW7P1v9FCatiIQMo-4xmqOx26Gm](http://img.zzhpeng.cn/FgW7P1v9FCatiIQMo-4xmqOx26Gm)

参考文献
<https://github.com/buxiaomo/MarkdownBooks/blob/master/Docker%E5%85%A5%E9%97%A8%E7%BA%A7%E7%AE%80%E6%98%93%E6%89%8B%E5%86%8C.md>



