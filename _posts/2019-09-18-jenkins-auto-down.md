---
layout: post
title: jenkins启动一段时间后掉线
date: 2019-09-18 
tags: jenkins
---

## 问题
> 和gitlab使用的端口8080冲突，修改为了8088。然后每次jenkins启动过一段时间后会自动的down服务，
启动命令是service jenkins start。

使用lsof -i:8080查看端口使用情况：
![](http://img.zzhpeng.cn/FhgyPfOl3dr5UBcyGy0iMjavJ_3c)

## 解决路程
> 尝试用另外一种方法启动,但还是使用旧的端口问题。

![](http://img.zzhpeng.cn/Fiy7cWxbK29pDZZdUnqaosW5BDfC)

搜索了下，找到了可以直接在启动后面加参数，改使用端口。

```
java -jar jenkins.war --ajp13Port=-1 --httpPort=8088
```

挂在后台
```
nohup java -jar jenkins.war --ajp13Port=-1 --httpPort=8088 &
```

## 再次出现问题
> 两种启动的数据文件不是用同一个的，用挂在后台的是新的数据库，需要重新注册admin，重新配置。

解决~~待。。。。

## 后续
这个问题，最终是用加大linux物理内存,机器是8G的内存。默认是没有设置swap的，我设置了8G的，最终java相关的应用，观察了几个月，如Eleasticsearch、Jenkis都稳定了。

## 迭代
* 2019年09月18日 00:00:00 初稿
* 2019年12月27日 15:29:00 修改


## 参考
* 1、[Jenkins 安装Address already in use问题](https://blog.csdn.net/panruifang/article/details/14223323)
* 2、[Linux SWAP交换分区应该设置多大？](https://blog.csdn.net/sirchenhua/article/details/87861709)