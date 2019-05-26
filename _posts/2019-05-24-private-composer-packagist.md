---
layout: post
title: 搭建私有的Composer Packagist
date: 2019-05-24 
tags: PHP Composer
---

### 官方文档和官方源
官方文档：https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md

github地址：https://github.com/composer/satis

### 下载源

* 方法一

```shell
git clone https://github.com/composer/satis.git

cd satis && composer update
```

* 方法二

```shell
composer create-project composer/satis:dev-master
```

### 写satis.json配置
* 生成satis.json文件

```shell
touch satis.json
```

* 写入配置

```json
    {
      "name": "zzhpeng/onehourx",
      "homepage": "http://packages.example.org", 
      "repositories": [
        {
          "type": "git",
          "url": "https://gitlab.01hour.com/onehourx/backend/plugin/saas-rpc-client.git"
        },
        {
          "type": "composer",
          "url": "https://packagist.laravel-china.org"
        }
      ],
      "require": {
        "onehourx/saas-rpc-client": "dev-master",
        "topthink/framework": "5.1.*"
      }
    }
```

### 运行
* 生成静态页面和更新源

```shell
php bin/satis build satis.json ./public

```

执行命令后会生成public文件夹，下面有静态文件

### 部署站点
* php cli部署

```shell
php -S localhost:8866 -t ./public
```


* nginx 部署

略

### 额外
* linux crontab定时器实时更新

略

