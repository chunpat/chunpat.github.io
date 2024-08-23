---
layout: post 
title: 搭建旁路由+科学上网
date: 2024-08-04 
tags: OpenWrt
---
## 起因

旁路由可以不影响主路由的稳定，又可以解决多设备科学上网。

## 镜像安装

素材地址，[点击谷歌云盘下载](https://drive.google.com/file/d/1_PXwS0fft44T8TDA-LT-272MMcg0XvV6/view?usp=drive_link)

### 将文件镜像上传到PVE宿主机
```
scp -r -P 22 /Users/zzhpeng/Downloads/openwrt.zip  root@192.168.199.225:/tmp
```

### 登录系统
```
root@192.168.199.225
```

### 还原镜像
```cd
root@xn:~# cd /tmp/
root@xn:/tmp# unzip openwrt.zip 
Archive:  openwrt.zip
   creating: openwrt/
  inflating: __MACOSX/._openwrt      
  inflating: openwrt/Openwrt.img     
  inflating: __MACOSX/openwrt/._Openwrt.img  
  inflating: openwrt/img2kvm         
  inflating: __MACOSX/openwrt/._img2kvm 
  
root@xn:/tmp# cd openwrt/
root@xn:/tmp/openwrt# ls -la
total 324128
drwxr-xr-x  2 root root      4096 Jul 30 14:05 .
drwxrwxrwt 11 root root      4096 Aug 22 09:16 ..
-rw-rw-r--  1 root root     18608 Jul 20  2023 img2kvm
-rw-rw-r--  1 root root 331874304 Nov 16  2022 Openwrt.img
root@xn:/tmp/openwrt# chmod +x img2kvm
root@xn:/tmp/openwrt# ./img2kvm ./Openwrt.img 102 vm-102-disk-0     
Converting file system to qcow2 format ...
... Done.
Importing disk to the VM 102 ...
```

### 扩展磁盘，原来的镜像只有320M，扩展到2G。

这里引用别人的[blog 给OpenWrt软件空间扩容](https://www.wort.cloud/post/notes/%E7%BB%99openwrt%E8%BD%AF%E4%BB%B6%E7%A9%BA%E9%97%B4%E6%89%A9%E5%AE%B9/)。这里需要注意，操作前做好镜像克隆以便还原，
操作错了不用重新开始。

### 更换v2ray版本

我的机场节点是用v2ray，不知道什么版本，直接用原本的v2ray 4.26版本无法使用，然后尝试了4.0最新版本的4.45就解决了。

查看原来的v2ray版本：
```
root@OpenWrt:/usr/bin/v2ray# /usr/bin/v2ray/v2ray
V2Ray 4.26.0 (OpenWrt) Lean (go1.14.4 linux/amd64)
A unified platform for anti-censorship.
2024/08/23 09:08:19 Using config from STDIN
2024/08/23 09:08:19 [Info] v2ray.com/core/main/jsonem: Reading config: stdin:
```

源文件手动下载[https://github.com/v2fly/v2ray-core/releases](https://github.com/v2fly/v2ray-core/releases)，然后通过openwrt的文件
上传上去到/tmp/upload/，或者scp上传。

scp指令如下

```angular2html
scp -r -P 22 /Users/zzhpeng/Downloads/openwrt.zip  root@192.168.199.11:/tmp/upload/
```

更换v2ray 4.45版本
```
root@OpenWrt:/usr/bin/v2ray# uname -a
Linux OpenWrt 4.19.131 #0 SMP Thu Jul 30 00:46:16 2020 x86_64 GNU/Linux

//解压
root@OpenWrt:/usr/bin/v2ray# cd /tmp/upload/
root@OpenWrt:/tmp/upload# unzip -d v2ray4.45 v2ray-linux-64.zip
Archive:  v2ray-linux-64.zip
inflating: v2ray4.45/v2ray         
inflating: v2ray4.45/config.json   
inflating: v2ray4.45/vpoint_socks_vmess.json  
inflating: v2ray4.45/geoip-only-cn-private.dat  
inflating: v2ray4.45/vpoint_vmess_freedom.json  
inflating: v2ray4.45/geosite.dat   
inflating: v2ray4.45/geoip.dat     
creating: v2ray4.45/systemd/
creating: v2ray4.45/systemd/system/
inflating: v2ray4.45/systemd/system/v2ray@.service  
inflating: v2ray4.45/systemd/system/v2ray.service  
inflating: v2ray4.45/v2ctl         

//添加执行权限
root@OpenWrt:/tmp/upload# cd v2ray4.45/
root@OpenWrt:/tmp/upload/v2ray4.45# chmod +x v2ray

//测试
root@OpenWrt:/tmp/upload/v2ray4.45# /tmp/upload/v2ray4.45/v2ray
V2Ray 4.45.0 (V2Fly, a community-driven edition of V2Ray.) Custom (go1.18.1 linux/amd64)
A unified platform for anti-censorship.
2024/08/23 09:17:21 Using default config:  /tmp/upload/v2ray4.45/config.json
2024/08/23 09:17:21 [Info] main/jsonem: Reading config: /tmp/upload/v2ray4.45/config.json
2024/08/23 09:17:22 [Warning] V2Ray 4.45.0 started

//备份原来的v2ray
root@OpenWrt:/tmp/upload/v2ray4.45# mv /usr/bin/v2ray/v2ray  /usr/bin/v2ray/v2ray-bk

//将新的复制份过去
root@OpenWrt:/tmp/upload/v2ray4.45# cp /tmp/upload/v2ray4.45/ /tmp/upload/v2ray/v2ray
```

## 效果

![openwrt-passwall](img/2024-07-30-pve-add-openwrt/openwrt-passwall.png)


## 迭代

* 2024年07月30日 14:50:54 初稿

## 参考

- 1、[给OpenWrt软件空间扩容](https://www.wort.cloud/post/notes/%E7%BB%99openwrt%E8%BD%AF%E4%BB%B6%E7%A9%BA%E9%97%B4%E6%89%A9%E5%AE%B9)
