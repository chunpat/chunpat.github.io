---
layout: post
title: 使用Linux的cpu挖xmr币
date: 2020-04-09
tags: xmr 挖矿
---
系统版本：CentOS Linux release 7.6.1810 (Core)

## 1、创建一个专门的用户
```
useradd coin
passwd coin
visudo //给sudo权限，你懂得
su coin
```

## 2、编译和安装xmr-stak，cmake3命令后面的参数声明只使用CPU进行挖矿

> 安装依赖

```
sudo yum install -y centos-release-scl epel-release
sudo yum install -y cmake3 devtoolset-4-gcc* hwloc-devel libmicrohttpd-devel openssl-devel make git
//临时使用devtoolset-4
scl enable devtoolset-4 bash
//下载源
git clone https://github.com/fireice-uk/xmr-stak.git
```
> 切换git仓库分支版本

因为master分支去掉了Monero币，而且剩下的都是些无名币。
```
cd xmr-stak
git fetch origin xmr-stak-rx
```

> 编译安装，只使用CPU进行挖矿

```
mkdir build
cd build
sudo cmake3 -DCUDA_ENABLE=OFF -DOpenCL_ENABLE=OFF ..
sudo make install
```

## 3、启动程序，填写币钱包地址、矿池ip等
```
./bin/xmr-stak-rx //之后有提示填写钱包地址和矿池ip地址，其他信息都是次要的
```
**./bin/xmr-stak-rx 可以先看看配置是否正确，然后在nohup ./bin/xmr-stak-rx & 挂载在后台**

## 4、钱包的创建、和矿池ip查找

* 4.1、下载钱包链接，可生成钱包地址，支持多种OS，点击[Downloads](https://web.getmonero.org/downloads/)

* 4.2、官网可以找到矿池ip，点击[MoneroHash - Monero Mining Pool](https://monerohash.com/#)

## 5、后话
2台四核服务器收益图，跑了9个小时左右，收益有点低，当时xmr市场价格为57.84USD。
![](http://img.chunpat.cn/Fr9Zq23WuchKUcBHAZB7gwhCIZvR)

## 迭代
* 2020年04月09日 09:00:00 初稿

## 参考：
* 1、[MoneroHash - Monero Mining Pool](https://monerohash.com/#getting_started)
