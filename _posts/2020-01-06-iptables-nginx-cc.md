---
layout: post
title: iptables和nginx配置防御cc学习
date: 2020-01-06
tags: iptables nginx
---
## 问题概述
上一篇文章配置了iptables两条命令，就觉得没问题了，但是我太天真了，归咎原因还是没有好好
学iptables的内在意思。

配置的iptables两条命令
```
sudo iptables -I INPUT -p tcp --dport 443 -m connlimit --connlimit-above 30 -j REJECT
sudo iptables -I INPUT -p tcp --dport 80 -m connlimit --connlimit-above 30 -j REJECT
```
解释下这两条命令，意思都是表示在同一时刻最大的tcp为30连接数限制，超过30连接数就会被reject拒绝。

`记住是同一时刻！！!`

同一时刻，即是tcp处于ESTABLISHED连接状态的。那这些怎么看呢？

> 案例


```
netstat -nat  | grep :443

Proto Recv-Q Send-Q Local-Address           Foreign-Address         State
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN     
tcp        0      0 172.18.245.246:443      113.67.9.211:14451      TIME_WAIT  
tcp        0      1 172.18.245.246:42776    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14442      TIME_WAIT  
tcp        0      0 172.18.245.246:42388    47.106.188.48:443       ESTABLISHED
tcp        0      1 172.18.245.246:42748    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14540      TIME_WAIT  
tcp        0      1 172.18.245.246:42690    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42682    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14559      TIME_WAIT  
tcp        0      1 172.18.245.246:42694    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42564    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42706    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:42380    47.106.188.48:443       ESTABLISHED
tcp        0      1 172.18.245.246:42396    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42674    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      47.106.188.48:42236     ESTABLISHED
tcp        0      1 172.18.245.246:42610    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42606    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42592    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:42294    47.106.188.48:443       ESTABLISHED
tcp        0      1 172.18.245.246:42342    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14498      TIME_WAIT  
tcp        0      0 172.18.245.246:443      47.106.188.48:42278     ESTABLISHED
tcp        0      1 172.18.245.246:42558    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      47.106.188.48:42272     ESTABLISHED
tcp        0      0 172.18.245.246:443      47.106.188.48:42252     ESTABLISHED
tcp        0      0 172.18.245.246:443      113.67.9.211:14543      TIME_WAIT  
tcp        0      1 172.18.245.246:42746    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42788    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14570      TIME_WAIT  
tcp        0      0 172.18.245.246:42270    47.106.188.48:443       ESTABLISHED
tcp        0      1 172.18.245.246:42792    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14493      TIME_WAIT  
tcp        0      0 172.18.245.246:42370    47.106.188.48:443       ESTABLISHED
tcp        0      0 172.18.245.246:443      113.67.9.211:14445      TIME_WAIT  
tcp        0      1 172.18.245.246:42688    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42570    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42328    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      47.106.188.48:42292     ESTABLISHED
tcp        0      0 172.18.245.246:42278    47.106.188.48:443       ESTABLISHED
tcp        0      1 172.18.245.246:42216    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14550      TIME_WAIT  
tcp        0      0 172.18.245.246:443      113.67.9.211:14487      TIME_WAIT  
tcp        0      1 172.18.245.246:42774    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14425      TIME_WAIT  
tcp        0      1 172.18.245.246:42726    47.106.188.48:443       SYN_SENT   
tcp        0      1 172.18.245.246:42390    47.106.188.48:443       SYN_SENT   
tcp        0      0 172.18.245.246:443      113.67.9.211:14578      TIME_WAIT  
tcp        0      0 172.18.245.246:443      113.67.9.211:14564      TIME_WAIT 
```
> 统计对外443端口连接数

```
netstat -nat|grep "47.106.188.48:443"|wc -l
```

> 统计对外443已连接上的，状态为ESTABLISHED

```
netstat -nat | grep ESTABLISHED | grep 47.106.188.48:443 | wc -l  
```

> 统计对外443等待连接的，状态为SYN_SENT

```
netstat -nat | grep SYN_SENT | grep 47.106.188.48:443 | wc -l  
```

iptables配置完后，过几天又出现了nginx的502情况，在nginx error查看当时的那个时间段的该ip的请求失败数。
```
grep '2020/01/01 19:02:31' xxx.com.error.log|wc -l   //结果显示62个请求失败数
```

``初步猜想``：nginx error显示62个请求失败数! 保持30个连接数量，但是请求却是一直在请求，动态语言php-cgi处理不过来，nginx
传过来的请求，就会直接返回错误。导致nginx直接将请求，记做错误写入error日志。

画个流程图：
![图](http://img.chunpat.cn/Fk7-M-U3c-LFEmPmzOe_bmNwuRNx)

## iptables学习与测试
本测试在测试环境测试，环境参数为4g内存2个核心1M带宽，系统环境是ubuntu16.4，该发行版本的默认管理防火墙的工具是ufw。

### 无任何防御测试
压测服务器ab命令 ab -c 20 -n 1000 https://dev.xxx.com/，全部通过
![ab命令](http://img.chunpat.cn/FrN6NF5LYoD3J9BZL37rzHsm0OVK)

服务器tcp连接数变化
![tcp连接数变化](http://img.chunpat.cn/FsCP9OnEiddWnKI1bMMQtKdTSEV0)

### iptables防御

> 查看iptables状态
 
```
sudo ufw status  //会显示 Status: active ，证明已经开启
```

> iptables 命令

添加，防御同一时刻20连接数限制
```
sudo iptables -I INPUT -p tcp --dport 443 -m connlimit --connlimit-above 20 -j REJECT
```

删除防御同一时刻20连接数限制
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
![ab命令](http://img.chunpat.cn/FjGLx-JepP-Q4JvMxYOOnNIMMQEW)

服务器tcp连接数变化,压测中的建立连接数不超过20
![tcp连接数变化](http://img.chunpat.cn/FhjpjR9Px6Y4sbo1l6f_d3ek0XIk)

加强压测，压测服务器ab命令 ab -c 25 -n 1000 https://dev.xxx.com/ 
![ab命令](http://img.chunpat.cn/FoJbn_IqOilb6IX1SYq-npWaMxmA)

服务器tcp连接数变化，压测中的建立连接数不超过20
![tcp连接数变化](http://img.chunpat.cn/FgjeblbI2m_2fElE8MwK1cs-9Y5M)


## nginx cc防御设置
即是是配置了iptables还是会出现502，查看大量请求的ip，这个ip是某个正常使用的门店，但是为什么会出现大量的请求呢？
推测F5,模拟测试。

> F5模拟测试

F5模拟测试,果然系统抛出502，我按出了浏览器显示差不多1200多个请求。
![tcp连接数变化](http://img.chunpat.cn/FkNU7r-CYTOdCTYzFylfeR4PCdFr)

然后，我在测试服务器，查看tcp变化数，限制到了20个连接数。自我推测：持续1200个请求下来，
存在了很多等待连接的。数据库返回数据慢了或者奔溃了，php-fpm响应超时重启出现502。

如图下，为F5请求产生502后，查看服务器tcp情况，还保持着20个连接数，和80个准备连接数。
![tcp连接数变化](http://img.chunpat.cn/FmQH6dfFWvqmgHrcTF_EcpYUjQPW)

`结论：iptable的限制20个连接数，能限制流量，但是F5攻击触及到数据库，就存在数据库那边的读的压力。
如果是静态或者缓存资源的的响应是没问题的，所以这里找到了nginx的cc防御。遇到一个时间段过多的流量可以抛出
个网络异常或503响应给浏览器，这样用户持续的F5就不会影响服务器了`

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
![宝塔设置](http://img.chunpat.cn/Fp8s437_mKXOffD-bKDyTub_vxg4)

> F5再次模拟测试

再次显示f5 65次就网络出错了，f5防御成功。
![F5再次模拟测试](http://img.chunpat.cn/Fubhh6oFbkaA5UBA9Et61YUcPFLp)



## 恍然大悟
1、前几次出现的f5攻击，只有上千条单个ip的error就停了，并且出现nginx 502。而最近f5攻击达到了上万条单个ip的error，这个就是
设置的iptables做了限制作用了，只允许30个连接数，数据库短暂压力没那么大，可以持续更长的时间。

2、iptables 设置了30个连接数限制，但是为什么单个ip会有上万个error错误呢？这里就牵着到了浏览器http2协议当中，一个tcp多路复用
可以传输多个http请求。

## 迭代
* 2020年01月06日 16:54:00 初稿

## 参考
* [UFW - Community Help Wiki](https://help.ubuntu.com/community/UFW)
* [IptablesHowTo - Community Help Wiki](https://help.ubuntu.com/community/IptablesHowTo)
* [Nginx限速遇到的问题 - MacoLee - 博客园](https://www.cnblogs.com/MacoLee/p/6023201.html)
* [服务器TIME_WAIT和CLOSE_WAIT分析和解决办法 - HttpClient 中文官网](http://www.httpclient.cn/archives/106.html)
* [一个 TCP 连接可以发多少个 HTTP 请求 - 割肉机 - 博客园](https://www.cnblogs.com/williamjie/p/11075565.html)
* [Nginx + CGI/FastCGI + C/Cpp - 吴秦 - 博客园](https://www.cnblogs.com/skynet/p/4173450.html)

