---
layout: post 
title: gmk mini主机
date: 2024-08-04 
tags: gmk
---

## pve 网卡竟然不是开启的，需要手动开启

买了两个硬盘，一个装win，一个装pve，原先装了pve, 之后win装了驱动后，回到pve里面的时候，网口的指示灯是不亮的，
ip addr 显示网卡state down。

通过命令开启。
```angular2html
ip link set enp1s0 up
```

## 迭代

* 2024年08月18日 14:50:54 初稿

## 参考
