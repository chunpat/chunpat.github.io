---
layout: post
title: sshd预防被暴力破解
date: 2018-05-29 
tags: hacker sshd
---

今天服务器出现了被攻击的现象，登录的时候很慢，登录成功后出现了有10w多的尝试登陆。

![](http://img.zzhpeng.cn/FpXkldTl5Jw2t77T_O3cKV4359kv)

查看失败登录情况
```shell
lastb | less
```
![](http://img.zzhpeng.cn/FjvMrfFakXiE2cgz0nzhvteJFqNp)

应对办法还是有的，我暂时选择了修改默认端口22。如果还有过多的尝试登录，日后在弄其他办法。
修改端口地方/etc/ssh/sshd_config，重启应用service sshd restart

参考链接 https://yq.aliyun.com/articles/195078。







