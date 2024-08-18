---
layout: post 
title: gmk mini主机
date: 2024-08-04 
tags: gmk
---
## 各种问题


## pve 网卡竟然是关闭的

买了两个硬盘，一个装win，一个装pve，原先装了pve, 之后win装了驱动后，回到pve里面的时候，网口的指示灯是不亮的，
ip addr 显示网卡state down。

通过命令开启。
```angular2html
ip link set enp1s0 up
```

## 动手

## 迭代

* 2024年07月28日 14:50:54 初稿

## 参考

- 1、[GitHub Actions 还能这么玩？自动将发布的博客文章更新到 GitHub 个人主页](https://shenxianpeng.github.io/2021/11/special-repository/)
- 2、[https://github.com/gautamkrishnar/blog-post-workflow](https://github.com/gautamkrishnar/blog-post-workflow)
