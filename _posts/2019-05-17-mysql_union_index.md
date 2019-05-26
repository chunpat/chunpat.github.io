---
layout: post
title: gitlab搭建和数据迁移
date: 2019-05-17 
tags: gitlab
---

## Ubuntu 搭建 gitlab

### 1.首先是安装一些依赖服务
```shell
sudo apt-get install curl openssh-server ca-certificates postfix

```

### 2.安装gitlab
> 注意：如果有旧gitlab要数据迁移的话，需要和旧gitlab的版本一致。

我下的是Ubuntu18
2.1、更新源
```shell
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

```
2.2、安装
```shell
sudo apt-get install gitlab-ce=11.7.0-ce.0
```

### 3.配置
打开/etc/gitlab/gitlab.rb,将external_url = 'http://git.example.com'修改为自己的域名地址：http://example.com，默认为80端口，如要使用其他端口后面加上端口号，如：http://127.0.0.1:8080。
更新配置
```shell
sudo gitlab-ctl reconfigure

```

### 4.操作
* sudo gitlab-ctl stop
* sudo gitlab-ctl start
* sudo gitlab-ctl restart


参考：https://www.cnblogs.com/m2ez/p/7063606.html


## gitlab搭建的数据迁移

### 1.查看版本
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

### 2.备份旧gitlab数据
2.1、备份原a服务器上的的数据
gitlab-rake gitlab:backup:create RAILS_ENV=production

PS: 备份后的文件一般是位于/var/opt/gitlab/backups下, 自动生成文件名文件名如1481529483_gitlab_backup.tar

### 3.旧gitlab数据转移到新
将步骤2生成的tar文件拷贝到b服务器上相应的backups目录下
可以利用scp或者用ftp进行直接拷贝.
scp username@src_ip:/var/opt/gitlab/backups/1481529483_gitlab_backup.tar /var/opt/gitlab/backups
PS: username为原服务器的用户名，src_ip原服务器IP地址

### 4. 在新服务器恢复数据
gitlab-rake gitlab:backup:restore RAILS_ENV=production BACKUP=1481529483
PS：BACKUP的时间点必须与原服务器备份后的文件名一致

参考：https://www.cnblogs.com/wenwei-blog/p/6362829.html
