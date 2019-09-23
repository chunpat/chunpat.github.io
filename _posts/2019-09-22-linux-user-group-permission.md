---
layout: post
title: gitlab出现502问题
date: 2019-09-22
tags: gitlab,宝塔,nginx,linux
---
### 问题概述
上周，公司内部服务器重启，自搭建的gitlab仓库返回502。在这里先说明gitlab的搭建情况，用了宝塔管理，已经有了
nginx，所以不用gitlab自带的nginx服务，修改过程如下。

1、修改配置文件，禁用内嵌 nginx
```
sudo vim /etc/gitlab/gitlab.rb
```

修改一下配置

```
# 关闭nginx服务
nginx['enable'] = false
```

应用修改的配置
```
# 重新加载配置
sudo gitlab-ctl reconfigure
# 重新启动
sudo gitlab-ctl restart
```

2、设置宝塔的nginx代理转发,如下
```
# 添加upstream指向gitlab
upstream gitlab-workhorse {
  server unix:/var/opt/gitlab/gitlab-workhorse/socket  fail_timeout=0;
}


# 在server中添加
server
{

    ......

    location / {
            # serve static files from defined root folder;.
            # @gitlab-workhorse is a named location for the upstream fallback, see below
            try_files $uri $uri/index.html $uri.html @gitlab-workhorse;
    }

    location @gitlab-workhorse {
            # If you use https make sure you disable gzip compression 
            # to be safe against BREACH attack

            proxy_read_timeout 300; # Some requests take more than 30 seconds.
            proxy_connect_timeout 300; # Some requests take more than 30 seconds.
            proxy_redirect     off;

            proxy_set_header   X-Forwarded-Proto $scheme;
            proxy_set_header   Host              $http_host;
            proxy_set_header   X-Real-IP         $remote_addr;
            proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header   X-Frame-Options   SAMEORIGIN;

            proxy_pass http://gitlab-workhorse;
    }

    location ~ ^/(assets)/  {
            root /opt/gitlab/embedded/service/gitlab-rails/public;
            # gzip_static on; # to serve pre-gzipped version
            expires max;
            add_header Cache-Control public;
    }

    .....
}
```

然后报出了502，查看nginx的error日志，提示权限问题。
![](http://img.zzhpeng.cn/Fkn0XGjtnpSN-0bTrMNeFeWo3wxr)

### 解决过程
看到/etc/gitlab/gitlab.rb文件有个说明，当不开启内部的nginx的时候，需要加一个外部的用户。这里的外部用户
即是指宝塔的nginx开启的用户www。

1、查看nginx的开启用户
![](http://img.zzhpeng.cn/FiqxjiPd1NGzUVLRyb9XmszwI9TE)

2、修改/etc/gitlab/gitlab.rb
![](http://img.zzhpeng.cn/Fp5IQkW4PcGC4O_-kF_oelDWhQJF)

应用修改的配置
```
# 重新加载配置
sudo gitlab-ctl reconfigure
# 重新启动
sudo gitlab-ctl restart
```

**！！！但是，还是不行，还是502问题。你看，理论上没问题啊！**

这里涉及到了socket文件通讯的问题，一旦建立连接就会一直连接，原权限之类的还是原来的。

nginx代理流，文件socket服务转发
```
upstream gitlab-workhorse {
  server unix:/var/opt/gitlab/gitlab-workhorse/socket  fail_timeout=0;
}
```

所以，需要重启nginx,让它重新加载conf配置文件，重新建立连接


> 额外

如何查看系统组情况和修改
```
cat /etc/group
```