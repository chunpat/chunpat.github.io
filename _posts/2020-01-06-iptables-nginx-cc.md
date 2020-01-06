---
layout: post
title: iptables和nginx配置防御cc学习
date: 2020-01-06
tags: iptables nginx
---
## 问题概述
上一篇文章配置了iptables两条命令，就觉得没问题了，但是我太天真了。

配置的iptables两条命令
```
sudo iptables -I INPUT -p tcp --dport 443 -m connlimit --connlimit-above 30 -j REJECT
sudo iptables -I INPUT -p tcp --dport 80 -m connlimit --connlimit-above 30 -j REJECT
```
解释下这两条命令，意思都是表示在同一时刻最大的tcp为30连接数限制，超过30就会reject拒绝。

`记住是同一时刻！！!`

但是配置完后，过几天又出现了nginx的502情况，在nginx error查看当时的那个时间段的该ip的请求数
```
grep '2020/01/01 19:02:31' xxx.com.error.log|wc -l   //结果显示62
```

结果显示62! ？？？不是设置了iptables限制了单个ip 30个并发吗？为什么会出现62个？这里上面有提示，同一时刻呀，即是tcp处于
ESTABLISHED建立状态的。那这些怎么看呢？


> 统计443端口连接数

```
netstat -nat|grep -i "443"|wc -l
```

> 统计已连接上的，状态为“established

```
netstat -nat | grep ESTABLISHED | grep :443 | wc -l  
```

## iptables学习与测试
本测试在测试环境测试，环境配偶4g内存2核心1M带宽，系统环境是ubuntu16.4,该发行版本的默认管理防火墙的工具是ufw。

### 无任何防御测试
压测服务器ab命令 ab -c 20 -n 1000 https://dev.xxx.com/，全部通过
![ab命令](http://img.zzhpeng.cn/FrN6NF5LYoD3J9BZL37rzHsm0OVK)

服务器tcp连接数变化
![tcp连接数变化](http://img.zzhpeng.cn/FsCP9OnEiddWnKI1bMMQtKdTSEV0)

### iptables防御

> 查看iptables状态
 
```
sudo ufw status  //会显示 Status: active ，证明已经开启
```

> iptables 命令

添加，防御20并发
```
sudo iptables -I INPUT -p tcp --dport 443 -m connlimit --connlimit-above 20 -j REJECT
```

删除防御20并发
```
sudo iptables -D INPUT -p tcp --dport 443 -m connlimit --connlimit-above 20 -j REJECT
```

查看是否生效
```
sudo iptables -nL
```

开机生效
```
自查
```

> iptables 测试

压测服务器ab命令 ab -c 20 -n 1000 https://dev.xxx.com/，少部分失败
![ab命令](http://img.zzhpeng.cn/FjGLx-JepP-Q4JvMxYOOnNIMMQEW)

服务器tcp连接数变化,压测中的建立连接数不超过20
![tcp连接数变化](http://img.zzhpeng.cn/FhjpjR9Px6Y4sbo1l6f_d3ek0XIk)

加强压测，压测服务器ab命令 ab -c 25 -n 1000 https://dev.xxx.com/ 
![ab命令](http://img.zzhpeng.cn/FoJbn_IqOilb6IX1SYq-npWaMxmA)

服务器tcp连接数变化，压测中的建立连接数不超过20
![tcp连接数变化](http://img.zzhpeng.cn/FgjeblbI2m_2fElE8MwK1cs-9Y5M)


## nginx cc防御设置
即是是配置了iptables还是会出现502，查看大量请求的ip，这个ip是某个正常使用的门店，但是为什么会出现大量的请求呢？
推测F5,模拟测试。

> F5模拟测试

F5模拟测试,果然系统抛出502，我按出了浏览器显示1300多个请求。
![tcp连接数变化](http://img.zzhpeng.cn/Fuc5PefZ1BirVjv4jdqN_gNIcyIT)

tcp变化数查看，拦截到了20个并发。自我推测：持续1300个请求下来，数据库返回数据慢了或者奔溃了，
php-fpm响应超时重启出现502。这里再次过滤，准确显示F5请求过来的ip。
![tcp连接数变化](http://img.zzhpeng.cn/FuVheog1dHsPBFdOejEXILWFk7-n)

`结论：单纯的iptable是防不住F5的，响应到数据库的都很危险，静态或者缓存资源的的响应是没问题的，所以这里找到了nginx的cc防御。`

> nginx.conf主配置

```
    limit_conn_zone $binary_remote_addr zone=perip:10m;//远端ip 限制 
    limit_conn_zone $server_name zone=perserver:10m;//服务器域名 限制
```

`容器共使用10M的内存来对于IP传输开销`

> server.conf某个网站

```
    limit_conn perserver 300;//远端服务器域名 300个并发限制
    limit_conn perip 25;//远端ip25个并发 限制
    limit_rate 1024k;
```
`limit_rate 512k： 对每个连接限速512k. 注意，这里是对连接限速，而不是对IP限速。如果一个IP允许两个并发连接，那么这个IP就是限速limit_rate×2。`

配置，这里我直接用了宝塔的UI配置，这里是在server外部配置的，全局。
![宝塔设置](http://img.zzhpeng.cn/Fp8s437_mKXOffD-bKDyTub_vxg4)

> F5再次模拟测试

再次显示f5 65次就网络出错了，f5防御成功。
![F5再次模拟测试](http://img.zzhpeng.cn/Fubhh6oFbkaA5UBA9Et61YUcPFLp)



## 恍然大悟
前几次出现的f5攻击，只有上千条就停了，并且出现nginx 502。而最近f5攻击达到了上万条，这个就是我的iptables做了限流作用了。

## 迭代
* 2020年01月06日 16:54:00 初稿

## 参考
* [UFW - Community Help Wiki](https://help.ubuntu.com/community/UFW)
* [IptablesHowTo - Community Help Wiki](https://help.ubuntu.com/community/IptablesHowTo)
* [Nginx限速遇到的问题 - MacoLee - 博客园](https://www.cnblogs.com/MacoLee/p/6023201.html)

