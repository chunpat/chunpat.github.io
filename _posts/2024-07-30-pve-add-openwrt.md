---
layout: post 
title: 搭建旁路由
date: 2024-08-04 
tags: OpenWrt
---
## 起因

查看指令：
```
root@chunpat:/var/lib/vz/template/iso# ./img2kvm -h
A utility that convert OpenWrt/LEDE firmware to disk image for KVM guest in Proxmox VE.
Copyright (C) 2017 everun.top
usage: img2kvm <img_name> <vm_id> <vmdisk_name> [storage]

  -h or --help       display this help.
  -V or --version    output img2kvm version informaton.

Command parameters:
  img_name           the name of OpenWrt/LEDE image file, e.g. 'lede-xxx-x86-kvm-combined-ext4.img'.
  vm_id              the ID of VM for OpenWrt/LEDE guest, e.g. '200'.
  vmdisk_name        the name of disk for OpenWrt/LEDE guest, e.g. 'vm-200-disk-1'.
  storage            Storage pool of Proxmox VE, default is 'local-lvm'.
```

执行操作
```
root@chunpat:/var/lib/vz/template/iso# ./img2kvm ./Openwrt.img 102 vm-102-disk-0 
Converting file system to qcow2 format ...
... Done.
Importing disk to the VM 102 ...
importing disk 'vm-102-disk-0.qcow2' to VM 102 ...
  Rounding up size to full physical extent 320.00 MiB
  Logical volume "vm-102-disk-1" created.
transferred 0.0 B of 316.5 MiB (0.00%)
transferred 5.3 MiB of 316.5 MiB (1.66%)
transferred 8.5 MiB of 316.5 MiB (2.67%)
transferred 13.3 MiB of 316.5 MiB (4.20%)
···
transferred 316.5 MiB of 316.5 MiB (100.00%)
transferred 316.5 MiB of 316.5 MiB (100.00%)
Successfully imported disk as 'unused0:local-lvm:vm-102-disk-1'
... Done.
```

https://post.smzdm.com/p/a60xqlrg/pic_14/

https://www.ryanchan.top/archives/openwrt-installation-on-pve

## 动手


## 迭代

* 2024年07月28日 14:50:54 初稿

## 参考

- 1、[GitHub Actions 还能这么玩？自动将发布的博客文章更新到 GitHub 个人主页](https://shenxianpeng.github.io/2021/11/special-repository/)
- 2、[https://github.com/gautamkrishnar/blog-post-workflow](https://github.com/gautamkrishnar/blog-post-workflow)
