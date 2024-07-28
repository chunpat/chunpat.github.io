---
layout: post
title: 云服务器磁盘扩容
date: 2019-09-10 
tags: disk
---
# 操作前记得保存数据，镜像
数据丢失，概不负责！！

# 数据盘扩容
## 查看磁盘情况
> 查看已分区磁盘情况
```
df -h
```

> 查看磁盘情况
```
sudo fdisk -lu
```

## 格式化
```
sudo mkfs.ext3 /dev/vdb5  //扩展分区，然后分逻辑分区
```


## 挂载
创建目录
```
sudo mkdir data
```
挂载
```
sudo mount /dev/vdb5 /data
```


# 系统盘扩容

## 阿里云扩容
阿里云可以直接加大当前磁盘容量，然后用工具直接命令扩容，很容易。详细查看下面url
https://help.aliyun.com/document_detail/111738.html?spm=a2c4g.11186623.4.1.74dc7f67KyKnUd#section-gxq-3tw-dhb

## 腾讯云

重装系统扩容
![](http://img.chunpat.cn/FtGTBTeftcin55OCyU8g-J2p_Q4J)

内容地址：https://cloud.tencent.com/document/product/362/32539



# 知其然
fdisk [选项] –l <disk>  列出所有分区表

(1).选项
```
-b <size>  指定扇区大小（512，1024，2048或4096 B）
-c  关闭DOS兼容模式
-u <size>  以扇区编号取代柱面编号来表示每个分区的起始地址，一般与-l选项联合使用
-C <number>  指定柱面编号
-H <number>  指定磁头编号
-S <number>  指定磁道扇区编号
```


# 参考
* 1、[Linux命令之fdisk - 苦逼运维 - 博客园](https://www.cnblogs.com/diantong/p/8820779.html)

